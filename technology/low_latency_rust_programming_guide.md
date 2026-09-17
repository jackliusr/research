# Low-Latency Rust Programming: The Borrow Checker's Dividend

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Core Data Infrastructure / Architecture — Banking, Electronic Trading, Real-Time Systems, Systems Programming  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** September 2026

---

> A companion guide to the latency discipline of C/C++ — not a replacement for it. This document covers only the *Rust-specific delta*: which latency techniques Rust makes unnecessary, which it makes harder, the hazards that quietly blow a latency budget, the crate ecosystem layer by layer, the C/C++ interop seam where kernel-bypass and FPGA code still lives, and an honest comparison for the technical decision-maker.

**Audience**: Solution architects, low-latency engineers, and trading-technology leads who already understand the latency discipline and are evaluating Rust for a measured component. This guide assumes you have read the repo's C/C++ sibling. Where the sibling owns a topic, this document cross-references it rather than re-deriving it. All system facts (release numbers, crate versions, stability statuses) were verified at primary sources and date-stamped; anything that could not be established is stated as such in §14.

**Verification date for all version and stability claims: 17 September 2026.**

---

## Table of Contents

1. [Overview and the Boundary Declared](#1-overview-and-the-boundary-declared)
2. [What Rust Actually Guarantees for Latency](#2-what-rust-actually-guarantees-for-latency)
3. [The Rust Latency Hazards](#3-the-rust-latency-hazards)
4. [The Rust Toolbox, Layer by Layer](#4-the-rust-toolbox-layer-by-layer)
5. [The Async Question](#5-the-async-question)
6. [The Measurement Discipline in Rust](#6-the-measurement-discipline-in-rust)
7. [Interop: Where Rust Stops and C/C++ Begins](#7-interop-where-rust-stops-and-cc-begins)
8. [The Trading and Market-Data Application](#8-the-trading-and-market-data-application)
9. [The Honest Comparison with C++](#9-the-honest-comparison-with-c)
10. [The Build, Toolchain and Compiler Levers](#10-the-build-toolchain-and-compiler-levers)
11. [The Regulated-Institution Angle](#11-the-regulated-institution-angle)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Claims Audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [Cross-References and Further Reading](#16-cross-references-and-further-reading)
17. [Closing Summary](#17-closing-summary)

---

## 1. Overview and the Boundary Declared

Low-latency Rust is the practice of meeting a hard, measured latency budget in a language whose default allocation and safety model differs from C++ in ways that are sometimes a gift and sometimes a tax. This guide is about the delta — the parts of the problem where the language changes your answer.

### 1.1 The sibling relationship, stated explicitly

The repo already contains **[`technology/low_latency_cpp_development_guide.md`](low_latency_cpp_development_guide.md)** (1,099 lines). That document is the primary reference for the low-latency discipline itself. This document is its Rust companion. The two are meant to be read as a pair: read the sibling for the discipline, read this for the language-specific delta.

The table below is the contract. Where the sibling owns a topic, this guide cross-references it by path and section number and **does not re-derive it**.

| Topic | Owner | Sibling sections | This guide's treatment |
|---|---|---|---|
| The discipline: what low-latency development is, the latency hierarchy, latency vs throughput, jitter | Sibling | `low_latency_cpp_development_guide.md` §1 | Cross-reference only |
| The latency budget model, budget decomposition, SLOs, measurement infrastructure | Sibling | `…§2` | Cross-reference; this guide describes how a Rust binary *fits* a budget, not how to build one (§8.5, §8.6 add the hot/warm/cold split and the observability line item) |
| Hardware fundamentals — caches, cache lines, NUMA, the memory hierarchy | Sibling | `…§3` | Cross-reference; this guide covers only how Rust *exposes or hides* those effects (false sharing, `CachePadded`, struct-of-arrays) |
| C++ language fundamentals for latency | Sibling | `…§4` | Not applicable; replaced by §2 and §3 of this guide |
| Concurrency and lock-free programming — memory model, fences, lock-free queues | Sibling | `…§5` | Cross-reference; this guide covers the Rust memory model's inheritance from C++20, and the crate supply |
| The single-threaded event loop and I/O design; networking for low latency, kernel bypass, NICs, Onload/`ef_vi`/DPDK/VMA | Sibling | `…§6`, `…§7` | Cross-reference; §5 of this guide covers only whether an *async runtime* earns its place on the loop, §7 only the FFI seam where Rust must call into that C/C++ stack |
| Message parsing and market-data formats (ITCH/SBE/FAST); the trading system reference architecture | Sibling | `…§8`, `…§9` | Cross-reference; §4 and §8 of this guide cover only the Rust crates, the SIMD question and the *Rust expression* of the architecture |
| Shared memory and IPC; FPGA and hardware acceleration | Sibling | `…§10`, `…§11` | Cross-reference; §7 of this guide covers `memmap2` and the mapping seam, and notes only that vendor FPGA toolchains are C/C++ |
| Profiling, tracing and benchmarking practices; latency measurement — clock sources, `rdtsc`, histograms, percentiles, outlier analysis | Sibling | `…§12`, `…§13` | **Owned by the sibling.** §6 of this guide covers only the Rust toolchain that implements it (`hdrhistogram`, `quanta`) and the Rust-specific benchmarking traps |
| Build and tooling philosophy; the optimization workflow; common pitfalls; real-world numbers | Sibling | `…§14`–`…§17` | Cross-reference; §10 of this guide covers only Rust/Cargo levers, §3 is the *Rust-specific* hazard list, and no latency number is restated here that cannot be attributed |
| Low latency in banking — MiFID II RTS 6, MAS, the economics of microseconds; practical projects and exercises | Sibling | `…§18`, `…§19` | Cross-reference; §11 of this guide covers only the *Rust-specific* regulatory argument, and the exercises are not duplicated |
| Rust language semantics, the crate ecosystem, the C/C++ interop seam, the Rust-vs-C++ judgement | **This guide** | — | §2–§12 |

**Disclaimed overlap.** If a passage in this guide appears to restate a mechanism from the sibling — the latency hierarchy, cache behaviour, tail-latency statistics, the trading reference architecture — treat the sibling as authoritative and this guide as a Rust-flavoured annotation. Nothing here is intended to supersede it.

### 1.2 One-page orientation

If you read only this page, read this:

- **Rust's single biggest latency property is negative**: there is no garbage collector, so there is no collector-induced pause to bound. That is a genuine and structural advantage over any managed runtime and it is *not* a performance claim relative to C++ — C++ has the same property. Verified against the language's own documentation: Rust's memory management is scope-based and deterministic, not runtime-collected.
- **Rust's second property is that deterministic destruction is on the critical path.** Every `Drop` runs at a place the compiler decides. That is the same discipline C++ programmers call RAII, and in Rust it is mandatory rather than stylistic. The Rust-specific twist is *where* the compiler chooses to drop things — a topic §3 treats at length, and one the 2024 edition changed.
- **Rust's third property is that aliasing is a compile-time discipline.** You cannot have a `&mut` and a `&` to the same data alive at once. Trading systems want shared mutable state (an order book read by the risk check while the feed thread writes it). Rust forces that sharing to be *expressed* — via ownership splitting, message passing, `Arc<Mutex<_>>`, or `unsafe`. Each of those has a measurable cost, and the cost is where the latency conversation lives.
- **Rust's fourth property is the ecosystem's shape.** The crate supply for ultra-low-latency primitives is real but thinner than C++ at exactly the two layers that matter most — kernel bypass and FPGA vendor tooling — because those layers *are* C/C++ vendor SDKs. A production Rust trading system is almost always a hybrid. See §7.
- **Rust's fifth property is that the compiler is a co-author.** `cargo` unifies build, dependency resolution, tests, benchmarks and docs. That is a productivity gain that shows up in defect rates and onboarding, not in nanoseconds. Do not confuse the two.

### 1.3 Is Rust a latency language? The framing question, answered with reasoning

**Short answer: Rust is a latency *viable* language with a correctness dividend, and it is the right default only for a specific shape of team and workload.**

The longer answer has three parts.

**First, "latency language" is not a property of syntax.** The properties that actually determine low-latency capability are: (a) no implicit runtime pause; (b) predictable allocation; (c) control over memory layout and cache behaviour; (d) the ability to express lock-free synchronisation with a defined memory model; (e) an ecosystem of vendor interfaces at the hardware layer; and (f) an engineering culture that measures tails. Rust satisfies (a), (b), (c) and (d) — (d) because it inherits the C++20 memory model (verified: the Rustonomicon states Rust "pretty blatantly just inherits the memory model for atomics from C++20"). Rust **partially** satisfies (e): it reaches C/C++ libraries perfectly well through FFI, but it does not *replace* them. Rust satisfies (f) only to the extent the team imports the measurement discipline from the sibling guide — the language does not hand it to you.

**Second, the mechanism that people conflate with speed is really the mechanism for *tails*.** No-GC means no unbounded pause. Compile-time aliasing means data-race freedom by construction, which removes a class of intermittent latency spike that is very expensive to diagnose in production (a contended write that only shows up under a specific scheduling interleaving). The dividend is concentrated in p99.9 and in the number of production incidents, not in the p50.

**Third, for whom.** Rust is a reasonable default when *all* of the following hold: the hot path can be written without reaching for the kernel-bypass or FPGA layer directly; the team has or can build Rust systems expertise; the component's lifetime is long enough to amortise the learning curve (a five-to-fifteen-year maintenance horizon); and the organisation places real value on the class of defect Rust removes. It is a poor default when the component *is* the kernel-bypass path, when the team is a small group of C++ specialists with a proven codebase and a short delivery window, or when the vendor interface you must meet only exists as a C++ template header with no C ABI. §9 and §12 work through this against explicit criteria.

---

## 2. What Rust Actually Guarantees for Latency

This section is deliberately mechanism-level. Every claim is attributed, and the attribution is to language documentation or a documented crate behaviour rather than to a conference slide.

### 2.1 No garbage collector — and why that is the single most important latency property

**The mechanism.** Rust's memory is reclaimed by scope-based destruction: when a value goes out of scope, its destructor runs and its memory is released, at a point the compiler determines statically. The Rust Reference's destructor chapter defines this via drop scopes. There is no tracing collector, no compactor, no finaliser queue, and therefore no runtime component that can suspend your thread to reclaim memory.

**Why it matters for latency.** A tracing garbage collector's cost is not its throughput cost — a modern generational collector is often *cheaper overall* than manual management. Its cost is *when* it charges you. Collection happens at an allocation point chosen by the collector's heuristic, and the resulting pause is not bounded by your code. For a system with a 10 µs budget, any unbounded pause is disqualifying regardless of its average size, because averages do not matter when the business cost is asymmetrically bad at the tail.

**Honest comparison, without overclaiming.** A generational GC with a concurrent collector (the G1/ZGC/Shenandoah generation on the JVM) can be tuned to low single-digit-millisecond or even sub-millisecond pauses, and ZGC's documented design goal is pause times that do not grow with heap size. Those are engineering achievements, not marketing. The accurate statement is therefore narrow: *modern collectors have bounded the pause problem impressively, but they do it with a tuning and verification burden that Rust removes entirely by construction.* Rust wins the argument at the point where you would otherwise be spending a week proving to a risk committee that no GC pause can breach the SLO under load — not at the point where you are choosing a language for a 5 ms analytics job.

**A caution against the common overclaim.** "No GC therefore low latency" is false as stated. A Rust program that allocates on every message will have a latency profile dominated by allocator behaviour and page faults, and it will be worse than a well-tuned JVM in a steady state with a pre-touched heap. Rust gives you the *ability* to have a flat profile. It does not give you the flat profile.

### 2.2 The zero-cost-abstraction claim, examined precisely

The claim is best read as a *design principle with exceptions*, not a guarantee. The useful discipline is a three-way split.

**Genuinely free (compile to the same machine code you would hand-write):**

| Abstraction | Why it is free |
|---|---|
| Iterators and closures over slices | Monomorphised and inlined; the Rust Performance Book's bounds-check chapter documents iteration as one of the safe ways to *remove* bounds checks |
| Generic functions (`fn f<T: Trait>`) | Monomorphisation emits one specialised copy per concrete type; no vtable, no indirection |
| `Option<&T>` and `Option<NonZeroU64>` | Niches make them the same size as the pointer/integer |
| Newtypes, `struct` wrappers and `#[repr(transparent)]` | No runtime representation; layout is the wrapped type, and the transparent wrapper has the same ABI by construction |

**Not free — and you must count the cost:**

| Abstraction | The cost |
|---|---|
| `dyn Trait` (trait objects) | Vtable indirection plus a pointer-sized fat pointer; the call cannot be inlined at the call site in the general case |
| `Rc<T>` / `Arc<T>` | Heap allocation for the value plus an atomic (or non-atomic) reference count mutated on every clone and drop |
| `RefCell<T>` / `Cell<T>` | Runtime borrow-flag or `Copy` access discipline; `RefCell` will panic on a conflicting borrow, which is a latency event (§3.6) |
| `Mutex<T>` / `RwLock<T>` `std` types | A real lock; contention turns microseconds into milliseconds |
| `async` blocks compiled to futures | A state machine whose size is the sum of its suspension points, plus runtime bookkeeping per poll |

**How to tell which you have.** Do not argue about it — look. `cargo-show-asm` (the renamed `cargo-asm`) and `cargo-llvm-lines` (0.4.48, published 2026-08-19) show you the generated code and the monomorphisation bloat. Benchmarks with instruction counting (see §6) make the difference legible. If an abstraction appears in neither the profile nor the assembly, it was free for your case; if it appears in the assembly but not the profile, it is free in practice. The Rust Performance Book's inlining chapter makes the corresponding point from the other direction: inlining is what makes most abstractions free, and it is not guaranteed.

### 2.3 Ownership and borrowing as a compile-time aliasing discipline

**The mechanism.** Rust's borrow checker enforces, statically, that at any moment a value is either borrowed by any number of immutable references *or* by exactly one mutable reference, never both. Lifetimes ensure no reference outlives its referent. This is documented in the Rust Reference's chapter on references and validity, and the enforcement is the compiler's.

**Why this is a latency property, not just a safety property.** The shared-mutable-state pattern a trading system wants — a book structure updated by the feed handler and read by the strategy and the pre-trade risk check — is exactly the pattern that Rust forces you to make explicit. You have four honest options, each with a different latency profile:

| Approach | Latency profile | When it is correct |
|---|---|---|
| **Ownership split** — the writer owns the book; readers get a message or a snapshot | Lowest sustained cost; adds a copy or a handoff | Default choice for a hot path |
| **Single-threaded ownership** — one thread owns everything, no sharing at all | Lowest possible cost; matches the sibling's event-loop model (sibling §6) | When the loop can be single-threaded |
| **Interior mutability** — `Mutex`/`RwLock`/`Arc<Mutex<_>>` | Lock acquisition, contention, potential priority inversion; a real p99.9 risk | Cold or warm path only |
| **`unsafe` with your own synchronisation** — atomics, seqlock, ring buffer | You get C++'s profile and C++'s risk | When you have measured the need and can carry the audit burden |

The genuine Rust advantage is that options three and four are *visible*. In C++ the same sharing is equally possible and equally costly, but nothing in the type system marks it, so the cost is discovered in production. Rust's contribution is to make the aliasing decision a design decision rather than an accident. It does not make the decision cheaper.

**The honest limitation, verified at a primary source.** The borrow checker rejects some *safe* patterns because it is a model, not an oracle. The strongest available published example is from a firm that runs both languages in production: Databento's October 2025 engineering post on why they did **not** rewrite their market-data feed handler in Rust documents three concrete frictions — buffer reuse across a loop colliding with lifetimes; self-referential structs (an owning struct plus subcomponents holding references to its fields) requiring either reference-counting overhead or passing the cache through every method; and compile-time generic versioning of message structs requiring trait scaffolding where C++ templates need only `is_same_v`. Their conclusion was to keep Rust for the gateway, the encoding reference implementation and the client library, and write the new feed handler in C++23. They also note that as of their testing, the Polonius borrow-checker work did not solve either lifetime case. That is the most useful evidence in this guide precisely because it is a negative result from a Rust-positive source.

### 2.4 Monomorphisation and static dispatch

Generic code in Rust is instantiated per concrete type at compile time. The latency consequence is favourable: no dynamic indirection on the hot path, and the inliner sees through the abstraction boundary. The costs are the two the discipline already knows: **code size** (binary and instruction-cache pressure — `cargo-llvm-lines` 0.4.48 exists to measure the bloat) and **compile time**. On a hot loop, instruction-cache pressure is a real latency effect; a monomorphisation explosion that pushes the hot loop out of L1i is a measurable regression, which is why instruction-count benchmarking (§6) is the right tool rather than wall-clock.

### 2.5 `no_std` — what it buys and what it costs

`no_std` is the crate-level attribute that links `core` instead of `std`. The Embedded Rust Book's chapter on it is the authoritative table, verified at source:

| Feature | `no_std` | `std` |
|---|---|---|
| Heap (dynamic memory) | Only with the `alloc` crate and a suitable global allocator | ✓ |
| Collections (`Vec`, `BTreeMap`, …) | Only with `alloc` and a configured global allocator | ✓ |
| `HashMap` / `HashSet` | Not available (no secure random number generator) | ✓ |
| Stack overflow protection | ✘ | ✓ |
| Runs init code before `main` | ✘ | ✓ |
| `libstd` | ✘ | ✓ |
| `libcore` | ✓ | ✓ |
| Suitable for firmware / kernel / bootloader | ✓ | ✘ |

**The honest latency verdict: for a colocated Linux trading binary, `no_std` buys nothing and costs a lot.** The `std` runtime does not introduce latency spikes into a steady-state hot loop; the things that do are allocation, page faults, syscalls and interrupts — and those are covered by the sibling guide's hardware and OS sections. What `no_std` removes is the *convenience layer*: `HashSet` disappears, panics need a handler, and thread primitives and file I/O go with `std`. You would be re-adding most of `std` by hand and losing stack-overflow protection (which, in a recursive parser, is a crash-avoidance property with its own availability argument).

`no_std` is genuinely relevant for a different shape of problem: a bare-metal or firmware-adjacent component — a smart-NIC adjunct, an embedded latency monitor, a microcontroller in a hardware timestamping rig — where the sibling guide's FPGA/low-level hardware material (sibling §11) applies. If you are in that world, `no_std` plus a static allocator (e.g. `talc`, 5.1.1, published 2026-09-09) is the pattern; if you are on a colocated Linux host, keep `std` and stop worrying about it.
## 3. The Rust Latency Hazards

Each hazard below follows the same shape: **the mechanism**, **how it shows up in a profile**, **the fix**. Every mechanism is either documented at a primary source or directly observable with the tooling named.

### 3.1 Allocations hiding behind ordinary-looking code

**Mechanism.** Rust heap allocation is not signposted. The Rust Performance Book is explicit about the cost: "each allocation (and deallocation) typically involves acquiring a global lock, doing some non-trivial data structure manipulation, and possibly executing a system call," and "small allocations are not necessarily cheaper than large allocations." Allocation is triggered by `Vec::push` past capacity, `String` growth, `format!`, `.clone()`/`.to_owned()`/`.to_string()` on owned types, `Box::new`, collecting an iterator, `HashMap` growth past the load factor, and `BufRead::lines` (documented as allocating a `String` per line, the fix being a reused `String` and `read_line`).

**In the profile.** `malloc`/`free` (or the alternative allocator's entry points) high in a `perf` profile, and a **heavier right tail rather than a lifted median** — the book's own diagnostic is that "if a general-purpose profiler shows `malloc`, `free`, and related functions as hot, then it is likely worth trying to reduce the allocation rate and/or using an alternative allocator." The book's sensitivity datapoint from rustc's history: reducing allocation rates by roughly 10 allocations per million instructions produced measurable improvements of about 1%. Cost is *cumulative*, which is exactly why it lands on p99: the tail is where a capacity growth, a rehash or a page fault coincides.

**Fix, in order of preference.** (1) **Measure** with `dhat` (0.3.3, 2024-02-04), which "precisely identifies hot allocation sites and their allocation rates" — and use its heap-usage testing feature as a *test*, so an allocation regression fails CI. (2) **Pre-allocate and reuse**: `Vec::with_capacity`, a reused `String` with `clear()`, process-lifetime buffers. (3) **Remove the allocation**: `arrayvec` (0.7.8) for fixed-size stack buffers, `smallvec` (1.16.1) for inline small buffers, `bumpalo` (3.20.3) as per-message scratch, `object-pool` (0.6.0) or `slab` (0.4.12) for reuse. (4) **Change the allocator** (§4.1).

**The Rust-specific wrinkle.** Ownership makes step 2 harder in exactly the case Databento documented: reading into a reused buffer while handing out borrows of that buffer's contents trips lifetimes, because the borrow checker cannot prove the borrowed view dies before the next refill. Workarounds: copy the parsed view into a per-message type; index instead of borrow; or keep the parse output as offsets into the buffer and materialise only what the strategy needs.

### 3.2 Smart pointers and interior mutability

| Construct | Runtime cost | Where it belongs |
|---|---|---|
| `Box<T>` | One allocation, one pointer chase per access | Cold path, or deliberately to shrink a large enum variant |
| `Cell<T>` (`T: Copy`) | Free — no runtime check, pure load/store | Hot path, for counters and flags |
| `RefCell<T>` | A borrow flag checked at runtime; **panics** on conflict | Warm path only; the panic is a latency event (§3.5) |
| `Rc<T>` | Non-atomic refcount on clone/drop; not `Send` | Single-threaded cold path |
| `Arc<T>` | **Atomic** refcount on every clone and drop — an RMW that invalidates the cache line | Cold path or per-connection setup, never per message |
| `Mutex<T>` / `RwLock<T>` (`std`) | OS-backed exclusive access; contention unbounded | Cold path only |
| `parking_lot::Mutex` (0.12.5) | Spin-then-park; faster uncontended paths | Warm path, if measured |
| `UnsafeCell<T>` | Free — the legal basis for all interior mutability | Behind your own synchronisation, with an audit |

**In the profile.** `Arc` shows up as atomic RMW traffic and cache-line invalidation — the symptom is *high variance in a loop that does almost no work*. A contended `Mutex` shows up as a flat, wide plateau in the tail. `RefCell` shows up as a cheap branch or, catastrophically, as a panic.

**Fix.** The Perf Book's framing is accurate: `Rc`/`Arc` "allow value sharing, which can be an effective way to reduce memory usage," but used for rarely-shared values they "can increase allocation rates by heap allocating values that might otherwise not be heap-allocated." On a per-message path, the presence of `Arc` is a design smell, not a tuning opportunity — restructure to ownership splitting or a single-threaded owner (§2.3). For genuinely contended shared structures, reach for the concurrency crates (§4.5), not a `Mutex` over a `HashMap`. On `parking_lot`: treat the crate's own comparative benchmarks as crate claims and re-measure with your contention pattern.

### 3.3 Dynamic dispatch and trait objects

**Mechanism.** `dyn Trait` is a fat pointer (data + vtable); the call through it cannot in general be inlined, so the inliner cannot see through the abstraction. This is the same cost C++ pays for virtual calls (sibling §4); the Rust-specific point is that *generic* code is statically dispatched by default, so `dyn` is a deliberate choice rather than an inheritance default.

**In the profile.** An indirect call in the assembly, and a small function that appears as its own frame with no inlining and no constant propagation into it.

**Fix.** Use a generic parameter and let monomorphisation specialise the call site. Use an `enum` plus `match` where the variant set is closed — idiomatic Rust, producing a jump table with inlinable bodies rather than a vtable indirection. Reserve `dyn` for cold-path polymorphism (plugin points, venue configuration, serialisation backends). `cargo-llvm-lines` (0.4.48, 2026-08-19) shows the monomorphisation cost of going generic — trade the indirect call for code size deliberately.

### 3.4 Bounds checks

**Mechanism.** Indexing a slice or `Vec` performs a bounds check. The Perf Book's verdict cuts against the folklore: the checks "can affect performance, e.g. within hot loops, **though less often than you might expect**." Branch predictors handle a well-predicted compare almost for free, and LLVM frequently proves the bound unnecessary.

**In the profile.** Not the compare — the **lost inlining and vectorisation** around a panic-capable path. If a hot parser loop fails to vectorise, look for a bounds check that could not be elided, because the panic path (§3.5) bloat inhibits optimisation even when it never executes.

**Fix — the book's three safe techniques, in order.** (1) Replace direct element access in a loop with **iteration** (`for x in &v` gives the compiler a length it already knows). (2) **Take a slice before the loop** and index the slice, hoisting the length check out. (3) **Assert the index range** so the compiler discharges the check once instead of per iteration. (4) `get_unchecked`/`get_unchecked_mut` — labelled "as a last resort," and every use is an obligation carried into §7's auditability argument. **The highest-leverage change in a market-data parser is usually to establish the message length once** (after validating header and framing) and then operate on a slice whose length is proven, which lets the compiler elide all the per-field checks. Prefer a validated-cursor pattern (`let (head, rest) = buf.split_at(n)`) over an unchecked index.

### 3.5 The panic and unwind machinery

**Mechanism.** A panic is not an `if`. The default build (`panic = 'unwind'`, the verified release-profile default) generates landing pads, unwinding tables and a personality routine so the stack can be unwound. Those tables cost binary size and inhibit optimisation in every function containing a panic-capable operation. So a panic in a hot path is both **(a) a multi-microsecond event when it fires** and **(b) a permanent tax on the surrounding code even when it never does.** Panic-capable operations include out-of-bounds indexing, integer overflow *if* overflow checks are on, `unwrap`/`expect`, `RefCell` borrow conflicts, allocation failure, and `panic!`.

`catch_unwind` is not a cheap alternative. The standard library documentation states it "is **not** recommended… for a general try/catch mechanism" (`Result` is appropriate), that it "**might not catch all Rust panics**" because a panic may abort rather than unwind, and that dropping an `Err` result "may in turn panic." It also notes that functions called from foreign code that does not support unwinding should be `extern "C"`, which "ensures that if the Rust code panics, it is automatically caught and the process is aborted."

**In the profile.** Larger-than-expected binaries and code sections; functions that fail to inline; and when it fires, a step change in the tail histogram.

**Fix — the no-panic discipline.** Set `panic = "abort"` for the release profile (also the Perf Book's recommendation for maximum runtime speed), accepting that a panic becomes an outage rather than a degraded response. Then eliminate panic-capable operations *structurally*: `get` and `match` rather than indexing; explicit checked/wrapping arithmetic; `Result` carried through the parse and handled on the cold path (drop the message, log off-path); `TryFrom` rather than `as`. Add `#[cold]`/`#[inline(never)]` on the error branch. And make the policy a build failure rather than an aspiration: `no-panic` (0.1.37, 2026-07-18) makes the compiler error if a function can panic, with the caveat that it needs optimisations enabled and is a compiler analysis, not a proof. Reserve `catch_unwind` for the supervisor boundary that must survive a worker failing.

### 3.6 The destructor-timing problem

**Mechanism, and what differs from C++.** Both languages promise deterministic destruction; Rust adds a language-level subtlety — **the temporary scope rules have changed.** The Rust 2024 edition (verified: released in 1.85.0, and the current edition) altered the drop order of temporaries in tail expressions. Before 2024 a temporary in a block's tail expression could live longer than the block and be dropped *after* the block's locals; in 2024 temporaries "may now be dropped before local variables," and the edition guide is explicit that "this narrowing of the temporary scope may cause programs to fail to compile in 2024." The safety net is the `tail_expr_drop_order` lint, which fires when a temporary with a non-trivial `Drop` is generated in a tail expression and which the guide recommends enabling explicitly (`#![warn(tail_expr_drop_order)]`) for manual inspection.

**Why a latency engineer cares.** A `Drop` is arbitrary code, and the place the language chooses to run it may be on your hot path or inside a lock. The classic incident is a destructor that runs while a lock is held, or frees a buffer that must be re-allocated on the next message, or is deferred to a batch-boundary scope exit. Rust makes this *more* tractable than C++ (no destructor ordering across translation units, no `delete` on a partially-constructed base), but the drop-scope rules remain the programmer's responsibility.

**In the profile.** A `free` at a point in the loop unrelated to the work being freed; a lock held across an allocator call; and, worst, a latency spike whose location moves when you reorder unrelated lines — because reordering changed the drop scope.

**Fix — three patterns.** (1) **Take-and-drop-off-path**: `mem::take`/`Option::take` the value out of the hot structure, push it onto a cold disposal queue, and let a warm/cold thread or an end-of-batch step dispose of it, leaving a move rather than a `Drop` on the hot path. (2) **Deliberate leak for process-lifetime buffers**: `Box::leak` or `mem::forget` converts a `Drop` obligation into a documented, intentional leak — legitimate *only* when the lifetime genuinely is the process, and it must be commented as such. (3) **Make the drop site explicit and measurable**: put disposal behind a named function on a named cold path, and assert with `dhat` that the hot path neither allocates nor frees per message. **Related 2024-edition note:** `unsafe_op_in_unsafe_fn` now warns by default, requiring explicit `unsafe {}` blocks inside `unsafe fn` — an auditability improvement (§7.6), not a performance one, but relevant when reasoning about drop and safety claims around FFI.

### 3.7 Formatting and logging in the hot path

**Mechanism.** `format!` produces an owned `String` — an allocation per call. The `core::fmt` machinery routes through `Formatter` and a `Display`/`Debug` implementation: dynamic dispatch per field, then a write. `println!`/`eprintln!` additionally lock stdout and may block on a pipe. In async code, `tracing` (0.1.44, 2025-12-18) with `tracing-subscriber` (0.3.23, 2026-03-13) is idiomatic and excellent for observability, but each event carries field construction, span bookkeeping and subscriber dispatch, and the cost is not zero even when a filter discards the event.

**In the profile.** `core::fmt` frames and allocator traffic — and the tell-tale sign, a latency profile that *changes shape when the log level changes*, which proves the logging is on the hot path and proves it is not free.

**Fix.** Never format on the hot path: emit a structured fixed-size record (integers and indices) into a pre-allocated ring and let a cold thread or an out-of-process collector format it (§8.6). Guard the *call site* where the arguments are expensive — a macro whose arguments are constructed unconditionally pays the construction cost even when filtered out. Render numbers without `format!` using `itoa` (1.0.18, 2026-03-20) and `ryu` (1.0.23, 2026-02-08), which write into a caller-provided buffer with no allocation and no `core::fmt` dispatch. Write into a reused buffer with `write!` rather than building a `String`. Treat logging as a budget line item (sibling §2, §12): "debug level is off in production" is an unmeasured claim until you have shown the disabled cost is negligible. I could not verify a published, reproducible per-event nanosecond cost for `tracing` with a subscriber installed — treat any quoted figure as unverified (§14) and measure your own configuration.

### 3.8 The default hasher

**Mechanism — verified at source.** `HashMap`'s default hasher comes from `RandomState`, whose associated `Hasher` is `DefaultHasher`. The Perf Book states the algorithm: "The default hashing algorithm is not specified, but at the time of writing the default is an algorithm called **SipHash 1-3**." SipHash is cryptographic-quality and DoS-resistant — the right default for a general-purpose language, because otherwise a `HashMap` keyed on attacker-controlled input is a trivial algorithmic-complexity denial of service. It is also *slow, particularly for short keys such as integers*, which is exactly the key shape a trading system uses: instrument IDs, order IDs, venue IDs.

**In the profile.** `DefaultHasher`/SipHash frames hot in any `HashMap`-heavy path — one of the largest single-function wins available in Rust. The book reports that switching rustc from `fxhash` back to the default hasher caused slowdowns ranging from 4% to 84% (that is a compiler, not a trading system; read it as an order-of-magnitude indicator).

**Fix — choose by key shape and threat model.**

| Hasher | Version (published) | Good for | Caveat |
|---|---|---|---|
| `rustc-hash` (`FxHasher`) | 2.1.3 (2026-07-02) | Integer and small keys; per the book, "out-perform[s] all other hash algorithms within rustc" | Low quality; **not** HashDoS-resistant — never on attacker-controlled input |
| `fnv` | 1.0.7 (2020-05-14) | Small keys; higher quality than Fx, slightly slower | Stale (last release 2020); superseded in most use |
| `ahash` | 0.8.12 (2025-05-08) | General purpose with AES-NI; keyed by default | The book records rustc seeing 1–4% *slowdowns* switching Fx→AHash; fast ≠ faster for your case |
| `foldhash` | 0.2.0 (2025-08-23) | Modern fast general purpose; designed for hashbrown | Newer; verify your own workload |
| `nohash-hasher` | 0.2.0 (2020-02-17) | Newtype keys wrapping an already well-distributed integer | Removes hashing entirely, so a bad key distribution becomes a collision problem on *your* table |
| `hashbrown` (as the map) | 0.17.1 (2026-05-09) | The implementation behind `std`'s `HashMap`, with a configurable hasher and raw-entry APIs | 0.x semver churn is real |

**The decision, plainly.** For a pre-trade risk check keyed on an instrument ID validated against a fixed universe, use a fast non-cryptographic hasher *and* validate the key space at the boundary, so the attack surface the default protects against does not exist. That is a conscious security trade — make it because you reasoned about the input, not because a blog post said SipHash is slow. If any part of the key crosses your trust boundary, keep SipHash (or a keyed `ahash`) and pay for it. The book also flags byte-wise struct hashing via `zerocopy`/`bytemuck`'s `ByteHash` derive and `derive_hash_fast` (0.2.3, 2025-08-18), with the book's caution that effects are "highly dependent on the hash function and the exact structure of the types being hashed. Measure carefully."

### 3.9 The async future's size and polling cost

**Mechanism.** An `async` block compiles into a state machine implementing `Future`, whose single method is `poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>` (verified in the standard library documentation). The documentation describes the contract exactly: `poll` "does not block if the value is not ready. Instead, the current task is scheduled to be woken up when it's possible to make further progress," via the `Waker` in the `Context`. Three costs follow:

1. **Future size.** The state machine's size is roughly the sum of the values live across its suspension points, so a large struct or array held across an `.await` inflates it. **The future's size is your packet-size and stack problem**: a `.await` on a boxed future in a per-message path is an allocation, and an oversized future moved between threads is a large memcpy. The observable test is `std::mem::size_of_val` at each layer; a per-message future of hundreds of bytes means the runtime is moving hundreds of bytes per message.
2. **Poll overhead.** Every `.await` that is not ready returns `Poll::Pending` and re-schedules. Five `.await`s that are always ready are cheap; five that are not are five scheduling round-trips. A hot path that is *always* ready should not be an async pipeline at all.
3. **Waker churn.** Each registration and wake is a vtable call on the `Waker` plus whatever the runtime does to move the task between queues. Under load this turns a nanosecond-scale decision into a microsecond-scale one, and it is invisible in a flamegraph of user functions — you see runtime internals, not your code.

**Fix.** For a hot, always-ready path, do not use async: use a hand-written event loop and blocking reads (sibling §6). For genuinely concurrent paths (thousands of client connections — the gateway case), async is right and these costs amortise. Keep futures small and uniform, avoid a `.await` in a loop that must stay in cache, and prefer a current-thread runtime for a core-pinned workload (§5.2).
## 4. The Rust Toolbox, Layer by Layer

Every version and date below was read from the crates.io API on **17 September 2026**. Downloads are cumulative all-time and indicate adoption, not quality. Status vocabulary: **active** (release within ~12 months), **slow** (12–30 months), **stale** (>30 months — use with a maintenance plan), **nightly** (requires an unstable compiler feature).

### 4.1 Allocators

Two different things are easy to conflate. **`#[global_allocator]` with the `GlobalAlloc` trait is stable** — you can replace the process-wide allocator today, and that is the lever that matters for latency. **`allocator_api` is unstable** — the `Allocator` trait and per-collection allocator parameters remain behind a feature gate. The tracking issue [`rust-lang/rust#32838`](https://github.com/rust-lang/rust/issues/32838) is **still open** as of 17 September 2026 and carries the `B-unstable` label ("Implemented in the nightly compiler and unstable") plus `S-tracking-needs-summary`. There is no stabilisation date. The `allocator-api2` shim (0.4.0, 2025-12-11) provides the API on stable, understanding that it is a compatibility shim.

| Crate | Version (published) | Status | Notes |
|---|---|---|---|
| `mimalloc` | 0.1.52 (2026-05-22) | active | Microsoft's allocator; Rust bindings maintained by the same org; secure mode available; ~55M downloads |
| `tikv-jemallocator` | 0.7.0 (2026-05-25) | active | The maintained jemalloc binding (TiKV's fork); background-thread and arena configuration; ~96M downloads |
| `tikv-jemalloc-ctl` | 0.7.0 (2026-05-25) | active | Runtime introspection of jemalloc state — how you *observe* allocator behaviour in production |
| `jemallocator` | 0.5.4 (2023-07-27) | **stale** | The original binding; no release in over three years. Prefer the TiKV fork |
| `snmalloc-rs` | 0.7.5 (2026-08-12) | active | Build-support wrapper that compiles the snmalloc C++ library and links it. **There is no `snmalloc` crate** — a lookup for that exact name returns HTTP 404 (verified 2026-09-17) |
| `talc` | 5.1.1 (2026-09-09) | active | `no_std`-friendly allocator for the embedded case (§2.5) |

**How to choose.** Start with the system allocator and `dhat` (§3.1). If `malloc`/`free` is hot and you cannot remove the allocations, try one alternative allocator and measure — the Perf Book's own summary of build configurations lists "an alternative allocator" as one of four levers for maximum runtime speed. Do not adopt one because a benchmark table says it wins: allocator behaviour is workload-specific and the cost of being wrong is a tail that looks fine in the median. One point that is easy to miss — `tikv-jemalloc-ctl` is often why jemalloc is chosen in a latency-critical component even when it is not the fastest: reading allocated/resident bytes and arena statistics from a live process, and alerting on them, is worth more operationally than a marginal throughput difference.

### 4.2 Arenas, pools and reusable scratch space

| Crate | Version (published) | Status | Good for |
|---|---|---|---|
| `bumpalo` | 3.20.3 (2026-05-22) | active | Single-threaded bump arena; ~582M downloads; default choice for per-message/per-batch scratch reset wholesale |
| `typed-arena` | 2.0.2 (2023-01-09) | slow | Typed `&mut` arena allocation; no release since Jan 2023 (feature-complete but strictly unmaintained) |
| `object-pool` | 0.6.0 (2024-08-18) | slow | Reuse of objects across messages without re-allocation |
| `slab` | 0.4.12 (2026-01-31) | active | Index-addressed storage with stable keys; ~925M downloads; the primitive for slot-indexed order stores |

**The design note.** An arena's value is that deallocation is bulk and free — you reset the arena rather than running destructors per object. That interacts directly with §3.6: an arena sidesteps the destructor-timing problem *for types that need no destructor*, and will silently skip `Drop` for types that do. If your scratch types own heap memory, arena reset is not cleanup — it is a leak. This is the most common arena bug.

### 4.3 Cache-friendly collections and small-buffer types

| Crate | Version (published) | Status | Good for |
|---|---|---|---|
| `hashbrown` | 0.17.1 (2026-05-09) | active | The SwissTable implementation behind `std`'s `HashMap`; configurable hasher, raw-entry API; ~2.45bn downloads |
| `smallvec` | 1.16.1 (2026-09-11) | active | Inline small-buffer vector for "usually short, must not allocate" |
| `arrayvec` | 0.7.8 (2026-07-02) | active | Fixed-capacity, stack-only vector with no allocation path — the strongest guarantee of the three |
| `indexmap` | 2.14.2 (2026-09-05) | active | Insertion-ordered map; `swap_remove` gives deterministic iteration order for cache behaviour |

**The queue story.** `std::collections::VecDeque` is a ring buffer and is entirely adequate for a single-producer/single-consumer handoff when locking is external; it is under-used because engineers reach for a channel crate first. For a shared queue use `crossbeam-queue` (§4.5); for "SPSC, wait-free, bounded, no allocator" use a dedicated ring. The distinction that matters is **bounded vs unbounded**: an unbounded queue converts latency into memory consumption and hides backpressure, so a latency-critical path should prefer a bounded ring and decide explicitly what happens on full. **On struct-of-arrays:** the idiomatic Rust is hand-written `Vec<A>` plus `Vec<B>` for hot fields and a separate AoS array for cold ones; `soa_derive` automates it but had its last release in March 2025. SoA/AoSoA layout is the sibling guide's subject (sibling §3); in Rust the technique is identical but the ergonomics are worse, and that ergonomic cost is a real argument for keeping a layout-critical component in C++.

### 4.4 Hashing

Covered in depth in §3.8. Versions: `rustc-hash` 2.1.3 (2026-07-02), `fnv` 1.0.7 (2020-05-14, stale), `ahash` 0.8.12 (2025-05-08), `foldhash` 0.2.0 (2025-08-23), `gxhash` 3.5.0 (2025-03-11), `nohash-hasher` 0.2.0 (2020-02-17, stale), `derive_hash_fast` 0.2.3 (2025-08-18). `std`'s default remains SipHash 1-3 via `RandomState`/`DefaultHasher`.

### 4.5 Concurrency primitives

**Atomics and orderings.** `std::sync::atomic` is stable and exposes `Ordering::{Relaxed, Acquire, Release, AcqRel, SeqCst}`. The important verified fact is the model's provenance: the Rustonomicon states that "Rust pretty blatantly just inherits the memory model for atomics from C++20," deliberately, so existing tooling and research apply. Every ordering decision you would make in C++ (sibling §5) is identical here, with identical hardware consequences — and the Nomicon notes that on strongly ordered platforms "there's rarely a benefit in making an operation relaxed," while relaxed operations "can be cheaper on weakly-ordered platforms." Rust does not make atomics cheaper than C++. It makes them harder to get wrong at the type level and removes the *data-race* class by construction.

| Crate | Version (published) | Status | Good for |
|---|---|---|---|
| `crossbeam-channel` | 0.5.17 (2026-09-05) | active | MPMC channels; the general-purpose workhorse |
| `crossbeam-queue` | 0.3.14 (2026-09-05) | active | `ArrayQueue` (bounded MPMC ring), `SegQueue` (unbounded) |
| `crossbeam-epoch` | 0.9.21 (2026-09-05) | active | Epoch-based reclamation — what makes lock-free structures with deferred free sound |
| `crossbeam-utils` | 0.8.23 (2026-09-05) | active | `CachePadded`, backoff, scoped threads |
| `rtrb` | 0.4.0 (2026-08-18) | active | Real-time SPSC ring with one allocation at construction and wait-free `push`/`pop` — the closest Rust has to a hot-path-safe ring. **API inconsistency:** crates.io reports `newest_version` 0.3.5 and `max_stable_version` 0.4.0 on the same date; verify what resolves |
| `ringbuf` | 0.5.2 (2026-09-13) | active | Lock-free SPSC/MPSC/SPMC ring with sync and async APIs |
| `flume` | 0.12.0 (2025-12-08) | active | MPMC channels with blocking and async interfaces, no `unsafe` in the API |
| `parking_lot` | 0.12.5 (2025-10-03) | active | Mutex/RwLock/Condvar/Once replacements; the de-facto `std` sync upgrade |
| `arc-swap` | 1.9.2 (2026-06-28) | active | Atomic replacement of `Arc` contents without tearing — the "publish a new config/book version" primitive |

**The false-sharing story.** `crossbeam_utils::CachePadded` pads and aligns a value to the cache line, and its documentation states the reason exactly: "Updating an atomic value invalidates the whole cache line it belongs to, which makes the next access to the same cache line slower for other CPU cores." Two idioms do the job: `CachePadded<T>` (preferred — tested and documented) and `#[repr(align(64))]` (or 128 on the architectures above) on a wrapper struct. Note that the assumption is architecture-specific: on x86-64, aarch64 and powerpc64 the documented padding is 128 bytes. Diagnosis remains the sibling guide's (sibling §3, §13) — pad the counters that are *actually* contended and prove it with hardware counters, because padding everything wastes cache. **The safety chapter that comes with lock-free Rust:** `crossbeam-epoch` is not a convenience but the answer to the memory-reclamation problem that makes hand-rolled lock-free structures in C++ a career-defining exercise. Using it correctly is still hard and it uses `unsafe` internally with `loom`-based model checking. If your lock-free code is not exercised through a model checker, your confidence level equals C++'s — which is none.

### 4.6 Async runtimes

| Runtime | Version (published) | Status | Design centre | Latency posture |
|---|---|---|---|---|
| `tokio` | 1.53.1 (2026-07-20) | **active, dominant** | Work-stealing multi-threaded scheduler plus a current-thread flavour | The ecosystem default; highest coverage; the multi-threaded scheduler is the wrong shape for a core-pinned hot path |
| `smol` | 2.0.2 (2024-09-07) | slow | Small, composable executor | Recommended by the async-std project as its successor; no release in ~2 years |
| `async-std` | 1.13.2 (2025-08-15) | **discontinued** | — | The repo README now reads "`async-std` has been discontinued; use `smol` instead" (verified 2026-09-17). Do not start new work on it |
| `monoio` | 0.2.4 (2024-08-20) | slow | Thread-per-core, `io_uring`-first | Matches a pinned trading core; no release in ~2 years |
| `glommio` | 0.9.0 (2024-03-25) | **stale** | Thread-per-core (DataDog), `io_uring` | No release in ~2.5 years despite an active repository. At-risk for a long-lived estate |
| `compio` | 0.19.2 (2026-08-18) | active | Completion-based I/O (`io_uring`, IOCP, kqueue) | The most actively developed completion-based runtime in the 0.x space; 0.x means API churn |
| `embassy-executor` | 0.10.0 (2026-03-20) | active | Embedded, `no_std`, interrupt-driven | The embedded case (§2.5) |

**The runtime-cost question, honestly stated.** None of these crates publishes a latency number that means anything for your workload, and I could not verify a first-party "documented overhead profile" with reproducible methodology for any of them (§14). What *is* verifiable is the architecture, and the architecture determines the cost shape. A **multi-threaded work-stealing** runtime (Tokio's default) can migrate a task between cores, which for a pinned, warmed hot path destroys cache and TLB locality — the observable symptom is tail spikes correlated with CPU count and load rather than with your code. A **current-thread** runtime (Tokio's `current_thread`, `smol`, `monoio`/`glommio`'s thread-per-core model) pins the task graph to one core, which is the shape that matches core-pinning discipline and the reason a thread-per-core runtime is the more natural Rust answer for a market-data path. A **completion-based** runtime (`compio`, `monoio`, `glommio`) moves the submission boundary into the runtime and can batch submissions — the closest Rust gets to the sibling guide's kernel-bypass-era I/O model (sibling §7) without the bypass itself. One verified detail: Tokio's published dependency list includes an optional `io-uring` dependency (`^0.7.11`), the mechanism behind its `tokio_unstable`-gated uring support — so "Tokio or io_uring" is not either/or; the questions are whether the integration is stable and whether the runtime flavour suits your core layout.

### 4.7 `io_uring` and networking crates

| Crate | Version (published) | Status | Notes |
|---|---|---|---|
| `io-uring` | 0.7.15 (2026-09-07) | active | The low-level, unsafe, faithful kernel binding. If you are writing the ring yourself, this is the crate |
| `tokio-uring` | 0.5.0 (2024-05-27) | **stale** | No release in over two years; its README still says the project "is still very young" and describes "safe APIs for all io-uring compatible operations" as eventual. **Do not plan a long-lived estate around it without a maintenance answer** |
| `monoio` / `glommio` / `compio` | per §4.6 | slow / stale / active | Runtimes that also provide the io_uring I/O layer |
| `memmap2` | 0.9.11 (2026-06-22) | active | Memory-mapped files; the shared-memory primitive (§7.5) |
| `libc` | 0.2.189 (2026-07-21); 1.0.0-alpha.4 exists | active | The FFI base. Do not adopt a 1.0 alpha for a production latency component. For POSIX wrappers prefer `rustix` or a hand-written declaration over adding a crate per syscall |

**What could not be established, flagged rather than asserted.** I could not identify a maintained, popular Rust binding to **DPDK** at a primary source — DPDK is a C library and the practical Rust approach is an FFI wrapper written for your build. **AF_XDP** bindings exist in small numbers but I did not establish a maintained one with credible adoption. **Kernel-bypass libraries** (Onload/`ef_vi`, VMA, Exanic) are C/C++ vendor SDKs with C ABIs, and I found no first-party Rust SDK from any of them. **This is the single most important ecosystem gap for a Rust trading system.** The practical implication: the Rust ecosystem covers the *application* layers of a trading stack very well (parsing, collections, channels, async, serialisation) and the *hardware* layers poorly. Plan the architecture with that asymmetry as a given, not as a problem to be solved later.

### 4.8 SIMD

Two mechanisms, two stability stories. **`std::arch` is stable**: target-specific intrinsics (`_mm_*`, AVX-512, NEON) gated behind runtime feature detection or compile-time target features (§10.2) — this is the production path today. **`std::simd` (portable SIMD) is nightly-only. Verified 2026-09-17** at `doc.rust-lang.org/nightly/std/simd/`: the module carries "🔬 This is a nightly-only experimental API. (`portable_simd` [#86656])" and every type in it is marked Experimental. There is no stabilisation timeline I could establish. **Do not architect around portable SIMD for a production component.** The module's own documentation notes that operations "do not necessarily map to a single instruction" and that `std::arch` can be faster for specific cases.

| Crate | Version (published) | Status | Notes |
|---|---|---|---|
| `wide` | 1.7.1 (2026-09-14) | active | Portable SIMD types that work on stable Rust; the pragmatic stable-channel option; ~73M downloads |
| `simd-json` | 0.18.1 (2026-08-23) | active | A real SIMD-accelerated JSON parser built on `std::arch`; the strongest existence proof that stable-channel SIMD parsing works |
| `simdutf8` | 0.1.5 (2024-09-22) | slow | SIMD UTF-8 validation; ~215M downloads; no release in ~2 years but widely depended upon |

**The market-data caution.** ITCH and its family, and the binary encodings of the sibling guide's §8, are predominantly *fixed-width binary* with a variable-length symbol field, not text. SIMD helps a JSON or UTF-8 validator enormously because the work is *scanning for structure*; it helps a fixed-offset binary parser much less, because the parse is a handful of unaligned loads and byte-swaps and the win comes from avoiding branches and feeding the right cache lines, not from vector width. If your ingestion path is textual (FIX, ASCII ITCH variants), SIMD scanning is a real lever; if it is binary, measure before reaching for intrinsics, and treat "SIMD will make my parser faster" as an unverified assumption.

### 4.9 Serialisation and parsing crates

| Crate | Version (published) | Status | Role in a market-data path |
|---|---|---|---|
| `rkyv` | 0.8.18 (2026-08-05) | active | Zero-copy deserialisation via archived types — read a record directly out of a mapped buffer with no copy step; the closest thing to the classic "cast the packet" pattern with a soundness story |
| `bincode` | 3.0.0 (2025-12-16) | active | Compact serde-based binary codec; the 3.0 line is a significant rework. Convenient, not zero-copy |
| `zerocopy` | 0.8.57 (2026-09-08) | active | Safe conversions between byte slices and types (`FromBytes`/`AsBytes`/`Unaligned`); also provides the byte-wise hash derive of §3.8 |
| `bytemuck` | 1.25.2 (2026-07-19) | active | The older, widely used equivalent (`Pod`/`Zeroable`/`AnyBitPattern` derives) |
| `serde` / `serde_json` | 1.0.229 / 1.0.151 (2026-07-18 / 2026-07-20) | active | The universal serialisation layer. **Its cost is the point**: `serde` derive produces a visitor-based deserialiser with a `Visitor` dispatch per field, and `serde_json` has no zero-copy path for owned strings. A warm/cold-path tool, not a hot-path one |
| `itoa` / `ryu` | 1.0.18 / 1.0.23 (2026-03-20 / 2026-02-08) | active | Integer and float to ASCII into a caller buffer; no allocation, no `core::fmt` |
| `no-panic` | 0.1.37 (2026-07-18) | active | Compile-time assertion that a function cannot panic (§3.5) |

**What is missing, said plainly.** I found **no established, widely adopted Rust crate for parsing the actual exchange protocols** — ITCH, SBE or FAST — of the kind the sibling guide covers in its §8. That is not a criticism of the ecosystem: these parsers are commercially sensitive, venue-specific, and short enough that firms write their own. The realistic plan for a Rust feed handler is to hand-write the parser using `zerocopy`/`bytemuck` for the wire layout, `arrayvec`/`smallvec` for variable-length fields, `rkyv` or a hand-rolled record type for normalised output, and `simdutf8`/`simd-json` only if the protocol is textual. Budget to write it; do not budget to adopt.
## 5. The Async Question

This is where Rust latency debates actually happen, so it gets its own section. It is deliberately short, because the *design* of the loop is the sibling guide's subject — see `low_latency_cpp_development_guide.md` §6 (the single-threaded event loop and I/O) for the loop model, and §7 (networking) for the transport. What follows is only the Rust-specific question: does an async runtime earn its place on this path.

### 5.1 The costs, and where they land

An async runtime is a library that owns your thread, your task queue and your I/O readiness notifications. Four costs follow:

1. **Per-task bookkeeping.** Every spawned task is a future plus runtime metadata, allocated and tracked. Task spawn is not free, and a task per unit of work is a design smell on a hot path.
2. **Scheduling.** A multi-threaded runtime moves tasks between worker threads; a task can be polled on a different core than it was last polled on. For a cache-warm, core-pinned hot path, that is a locality loss with a tail-shaped symptom.
3. **Waker traffic.** Each wake is a vtable dispatch plus a queue operation. Under load this is the dominant term in a "cheap" async pipeline (see §3.9).
4. **Two I/O models with different semantics.** Readiness-based runtimes (Tokio, `smol`) need buffers to outlive the operation; completion-based runtimes (`compio`, and `io_uring` in general) hand buffers to the kernel and get them back. The completion model is closer to the kernel-bypass mental model, and it changes your buffer-ownership code substantially.

### 5.2 Single-threaded vs multi-threaded

| Choice | Right when | Wrong when |
|---|---|---|
| **Current-thread / thread-per-core** (`tokio` current-thread flavour, `smol`, `monoio`, `glommio`) | A pinned, cache-warm, latency-critical path; one core per work unit; deterministic placement | The workload is I/O-latency-bound across many independent connections and you need core utilisation |
| **Multi-threaded work-stealing** (Tokio default) | Many concurrent connections, bursty work, throughput matters more than a single core's determinism — the Databento gateway case in §7.1 | A single hot path that must not migrate |

**The rule.** For the latency-critical core, choose a current-thread or thread-per-core runtime and pin it (sibling §3 for the pinning mechanics). For the surrounding service layer — the API, the admin plane, the metrics exporter — the multi-threaded default is fine, and using it keeps you on the ecosystem's beaten path. This is exactly the split Databento describes: they use `monoio` with `io_uring` and per-core pinning for feed handlers, and Tokio for the API layer, on the reasoning that ecosystem support for middleware matters more than microseconds on the API path.

### 5.3 When a hand-written event loop beats async

Write the loop yourself — `epoll`/`io_uring` directly, or a blocking-read loop, in the sibling guide's §6 shape — when **all** of these hold:

- The number of file descriptors is small and known (one or two feed sockets, one order gateway connection).
- The work per event is short and always-ready (parse, update a book, evaluate, maybe send).
- There is no third-party library you need that only exposes an async API.

In that configuration, async buys you nothing you do not already have, and costs you the four items in §5.1 plus a substantial readability tax: `async` infects function signatures, lifetimes get harder, and `Send` bounds propagate through your types. Adding `async` to a single-connection hot path is a common and expensive mistake.

### 5.4 The task-per-connection trap

Spawning one task per connection is the natural way to write a server in async Rust and it is the wrong shape for a latency-critical gateway where a small number of connections matters. The costs compound: one task per connection multiplies per-task bookkeeping by the connection count; each task's future carries its per-connection state, so future size times connections is your memory footprint; and the scheduler's fairness heuristics now interleave unrelated connections on the hot path. The alternatives are the classic ones — a slab of connection state indexed by slot (`slab`, 0.4.12) with one poll loop, or a bounded pool of worker tasks with explicit work distribution — both of which are more code and less scheduler surprise. Choose deliberately.

### 5.5 The honest verdict

**Async is a productivity and scalability tool that must earn its place on a hot path.** It earns it when: there are many concurrent connections; the I/O is genuinely slow or unavailable relative to the work; the codebase is large enough that a hand-rolled state machine per protocol would be a maintenance liability; or a third-party library you depend on is async-only. It does not earn it when the hot path is one socket, always ready, on a pinned core, with a 10 µs budget. The decision procedure is the same one the sibling guide applies to any abstraction: state the requirement in nanoseconds, measure the candidate, and keep the candidate that meets the requirement with the least machinery.

## 6. The Measurement Discipline in Rust

**Methodology belongs to the sibling guide.** Clock sources, `rdtsc` and its pitfalls, histogram construction, percentile interpretation, coordinated omission, and outlier analysis are all in `low_latency_cpp_development_guide.md` §13, and the profiling/tracing workflow is its §12. Do not re-derive them here. This section covers only the Rust toolchain that implements that discipline, and the Rust-specific traps that invalidate measurements.

### 6.1 The benchmark harnesses

| Tool | Version (published) | Status | Use |
|---|---|---|---|
| Built-in `#[bench]` | ships with `rustc` | **nightly only** | Verified 2026-09-17: the Rust Unstable Book documents that "the internals of the `test` crate are unstable, behind the `test` flag" and requires `#![feature(test)]`. Usable on nightly with an intentional pin; not usable on stable |
| `criterion` | 0.8.2 (2026-02-04) | active | Statistical wall-clock benchmarking on stable; ~282M downloads; the default choice |
| `divan` | 0.1.21 (2025-04-10) | slow | Lighter-weight alternative with allocation-count measurement built in |
| `iai-callgrind` | 0.16.1 (2025-07-30) | slow | Instruction-count and cache-counter benchmarking via Valgrind's Callgrind: deterministic, low-noise, and the right tool for a change that is under wall-clock noise |

**Instruction counting deserves emphasis.** On a noisy shared host — or a laptop — wall-clock difference of 30 ns across 200 ns of work is invisible. Callgrind counts instructions, so a 15% instruction reduction is a 15% measurable number regardless of the CPU's frequency governor. For a hot-loop optimisation, that is the difference between "I think it's faster" and "it is fewer instructions." The caveat is that Callgrind does not model the CPU (no branch prediction accuracy, no cache timing beyond its simulation modes), so a reduction in instruction count is necessary but not sufficient; confirm with a real latency histogram on real hardware before claiming a win.

### 6.2 Profiling and allocation accounting

| Tool | Version (published) | Status | Notes |
|---|---|---|---|
| `flamegraph` (installs the `cargo-flamegraph` binary) | 0.6.14 (2026-08-12) | active | Wraps `perf` and the flamegraph scripts. **Note the naming trap:** `cargo install cargo-flamegraph` works, but the *crate* is named `flamegraph` — a `crates.io` lookup for `cargo-flamegraph` returns HTTP 404 (verified 2026-09-17). This is why `cargo install` sometimes appears to work with a name that does not exist as a crate |
| `pprof` | 0.14.1 newest, 0.15.0 listed as max stable (both 2025-07-23) | active | In-process profiling for a running service; the crates.io API reports an inconsistency between `newest_version` and `max_stable_version` for this crate — verify the version you resolve |
| `perf` | distribution package | — | The kernel profiler. Works on a Rust binary directly; see sibling §12 for methodology. A hand-written counting `GlobalAlloc` belongs beside it: implement `GlobalAlloc`, count allocations and bytes per message, and fail the build on a breach |
| `dhat` | 0.3.3 (2024-02-04) | slow | The Rust binding for Valgrind DHAT, plus heap-usage testing for CI assertions |
| `tikv-jemalloc-ctl` | 0.7.0 (2026-05-25) | active | Read jemalloc statistics from a live process — the production-side counterpart to the test-time measurement |
| `hdrhistogram` | 7.6.0 (2026-07-18) | active | HDR histograms in Rust; the data structure the sibling guide's §13 recommends for tails |
| `quanta` | 0.12.6 (2025-06-10) | slow | Cheap clock; supports TSC-based measurement where available. `coarsetime` (0.1.37, 2026-01-10) is the same trade for durations: coarse, monotonic, low overhead |

**The allocation budget as a test, not a habit.** The highest-value Rust-specific practice in this section is turning the sibling guide's latency-budget model into a *build-time assertion*: count allocations per message in a unit test and fail the build above zero (hot path) or above a fixed small number (warm path). `dhat`'s heap-usage testing exists for exactly this; a counting global allocator does the same job with less dependency. This is the Rust answer to §3.1's cumulative-allocation hazard — you cannot drift if the test will not let you.

### 6.3 The optimiser-deleted-the-work trap

This is the Rust-specific measurement hazard, and it is not theoretical: the Rust Unstable Book's own worked example shows a `#[bench]` reporting `0 ns/iter` because the compiler removed the entire computation, rising to `131 ns/iter` once `black_box` was applied. The same trap applies to Criterion and Divan benchmarks.

**The fix, and its limitation.** `std::hint::black_box` (stable since Rust 1.66) is "an identity function that hints to the compiler to be maximally pessimistic about what `black_box` could do," and the standard library documentation explicitly recommends it for benchmarks. Three things to know:

- **`black_box` affects the output, not the input.** The documentation's own example is exact: `black_box(5 * 10)` is seen by the compiler as `5 * 10` computed then passed in, "so it will likely simplify `5 * 10` to just `50`." To stop the multiplication being folded you must write `black_box(5) * black_box(10)`. Benchmarking code that feeds `black_box` a fully-known constant is a common and silent error.
- **It is best-effort.** The documentation is explicit that "`black_box` is only (and can only be) provided on a 'best-effort' basis," that the extent to which it blocks optimisations varies by platform and codegen backend, and that it "**must not be relied upon to control critical program behavior**."
- **It does not solve the setup/teardown problem.** If the input must be constructed destructively per iteration, use the harness's batching helpers — Criterion exposes batching facilities for separating setup from measurement; check the API names against the version you use, as they have changed across releases. Hand-rolling `iter_batched` semantics incorrectly is the second most common way to measure the wrong thing.

**Two further Rust-specific traps.**

- **Measuring a debug build.** Cargo's `dev` profile has `opt-level = 0` and the Rust Performance Book notes release builds are typically "10–100x speedups over dev builds." `cargo bench` uses the `bench` profile, which inherits `release` — so a benchmark run through Cargo is fine; a timing harness you wrote around `cargo build` (no `--release`) is not.
- **Measuring with `panic = "unwind"` when you ship `panic = "abort"`.** The panic strategy changes codegen and removes unwinding tables, so the binary you measured is not the binary you ship. Make the benchmark profile match the production profile — see §10.3.

## 7. Interop: Where Rust Stops and C/C++ Begins

The sibling guide's networking and hardware sections (`low_latency_cpp_development_guide.md` §7 and §11) describe a stack whose bottom two layers — kernel bypass and FPGA — exist as C and C++ vendor SDKs. That is not going to change because of Rust. This section is about meeting those layers correctly from Rust, and about the cost of doing so.

### 7.1 Why a Rust trading system is almost always a hybrid

| Layer | Language it exists in | Rust access |
|---|---|---|
| FPGA bitstream / HLS | AMD (Xilinx) Vitis, Alveo toolchains; Intel toolchains | None. Out of scope for Rust; the sibling guide's §11 owns this |
| Kernel bypass (Onload / `ef_vi`, DPDK, VMA, Exanic) | C and C++ SDKs with C ABIs | FFI wrapper you write and maintain (§7.4) |
| NIC and driver tuning | kernel + vendor C | Configuration, not code |
| Shared memory transport | C ABI conventions plus a byte layout you define | Direct: `memmap2` + `zerocopy`/`bytemuck` (§7.5) |
| Feed parsing, book building, risk, strategy, gateway sessions | **This is where Rust competes** | Native Rust |
| Enterprise integration (adapters, sequencing, order management) | C++, Java, or both | FFI, shared memory, or a message bus |

**The practical conclusion.** Do not plan a "rewrite the trading stack in Rust" programme. Plan a hybrid with an explicitly-named seam, exactly as the two firms that have published on this have done. Databento runs some services in Rust and some in C++ and states that "they call each other in production," with a shared C FFI boundary enabling component-by-component migration. That is the realistic architecture.

### 7.2 The C ABI: the only contract that matters

Rust's default layout is **unspecified** — the compiler may reorder fields. Every type that crosses the FFI boundary must have a defined layout and a defined calling convention:

- **`#[repr(C)]`** on every struct shared with C/C++. `#[repr(transparent)]` for newtypes over a single C type; `#[repr(u8)]` or similar for enums that must have a fixed discriminant. `#[repr(C)]` on an enum gives it C enum semantics where that is what you want.
- **`extern "C"`** on both sides of the boundary for the calling convention. Note the `"C-unwind"` ABI for a boundary that must permit an unwind to propagate (used with C++ exceptions); for a latency path you want neither, and `extern "C"` causes a Rust panic crossing the boundary to be caught and the process aborted — which is the behaviour you want in a `panic = "abort"` world.
- **Opaque types.** Pass `*mut c_void` or a pointer to a forward-declared `#[repr(C)]` struct with no fields; never expose a Rust type with a non-`repr(C)` layout.

| Tool | Version (published) | Status | Direction |
|---|---|---|---|
| `bindgen` | 0.73.2 (2026-09-08) | active | C/C++ headers → Rust `extern` declarations |
| `cbindgen` | 0.29.4 (2026-06-09) | active | Rust `extern "C"` declarations → C/C++ headers |
| `cc` | 1.4.6 (2026-09-13) | active | Compile C/C++ sources from `build.rs`; ~1.24bn downloads |
| `cmake` | 0.1.58 (2026-03-26) | active | Drive a CMake build from `build.rs` — the usual way to consume a vendor SDK |
| `meson` (via `build.rs`) | — | — | Direct equivalent; verify against the vendor's own documented build before assuming a wrapper exists |

**The build-integration reality.** Consuming a kernel-bypass SDK from Cargo means a `build.rs` that compiles or links the vendor library, sets the include and library paths, and passes linker flags. This works and is well-trodden. It is also where most of the intractable friction lives: vendor SDKs assume a specific toolchain, ship their own libc-adjacent assumptions, and are frequently shipped only as a static archive built with a different GCC/Clang version than the one the `cc` crate selects. Budget for this. A hybrid build's fragility is not in the FFI call; it is in the build script.

### 7.3 The cost of crossing the boundary

**The cost is not the call.** A C ABI call is a call: push arguments, jump, return. The costs that matter are structural:

1. **The compiler can no longer see through the call.** Everything on the far side is opaque: no inlining, no constant propagation, no dead-code elimination, no vectorisation across the boundary. A 5 ns C function called in a loop is not a 5 ns problem if the presence of the opaque call also forces the loop to be re-loaded and prevents the surrounding code from being scheduled well.
2. **Cross-language LTO is possible but not automatic.** The Cargo Book's `lto` documentation is explicit: cross-language LTO "is not yet supported natively in Cargo, but can be performed via `RUSTFLAGS`" (through `-C linker-plugin-lto`). Without it, the boundary is opaque by construction. With it, you get cross-language optimisation at the cost of a much more brittle build; see §10.4.
3. **Unwinding semantics differ.** Rust's `panic = 'unwind'` default and C++ exceptions crossing each other is a documented source of unspecified behaviour — `catch_unwind`'s own documentation says that catching a foreign exception "will have one of two behaviors, and it is unspecified which will occur." With `panic = "abort"` (the recommended latency setting) the question mostly disappears, because there are no Rust unwinds to propagate.
4. **The marshalling tax is yours to control.** If the boundary passes owned `String`/`Vec` values, you are allocating and copying per call. That is usually what makes an FFI call expensive in a Rust codebase, not the call itself. Pass borrowed slices (`*const u8`, `len`) and `#[repr(C)]` structs of scalars.

**The design rule.** Keep the boundary off the hot path. Call into the vendor C SDK at *connection setup* (create the ring, map the buffer, register the queue) and then poll the ring from Rust code that touches only memory. The hot path then contains zero FFI calls, and the boundary's cost is amortised to nothing. Where you cannot avoid a per-message call, measure it with instruction counting (§6.1) rather than guessing.

### 7.4 The kernel-bypass interfaces a reader would actually meet

Named factually, because these are the interfaces a real project meets — and note that the sibling guide owns their *behaviour* (sibling §7):

| Interface | Vendor / ecosystem | Form | Rust path |
|---|---|---|---|
| **Onload / `ef_vi`** | AMD (formerly Solarflare/Xilinx) | `libonload` user-space TCP shim; `ef_vi` for the low-level virtual interface | FFI link against the vendor library; `ef_vi` is the lower-level, higher-control API |
| **DPDK** | Linux Foundation DPDK project | C library, poll-mode drivers, hugepages | FFI; **I could not establish a maintained Rust binding** (§4.7) |
| **VMA** | NVIDIA (Mellanox) | C++/C messaging accelerator library | FFI |
| **Exanic / `exanic` APIs** | Cisco (formerly Exablaze) | C library for their NICs | FFI |

**The honest engineering statement.** All four are C/C++ and all four are outside Rust's safety model. When you call them, you are writing C with Rust syntax, and the safety argument stops at the `unsafe` block. That is not a reason to avoid Rust — it is a reason to keep the FFI surface as small as possible, to write the safe wrapper yourself with the same care you would spend on C, and to *test* it. §7.6 covers the auditability of that wrapper.

### 7.5 Shared memory and IPC

The sibling guide's §10 owns the shared-memory and IPC *design* (layout, cache-line discipline, the ring protocol, sequencing). The Rust-specific layer is small:

- **`memmap2` (0.9.11, 2026-06-22, active)** maps a file or an anonymous region into the address space; it is the mechanism, and it is a thin, well-maintained wrapper.
- **`zerocopy` / `bytemuck`** provide the safe(ish) byte-level reinterpretation for the shared layout. This is the correct Rust idiom for a shared record: define a `#[repr(C)]` type, derive the byte-view trait, and read directly out of the mapping — no copy, no `serde`.
- **The ring protocol is the same problem it always was.** Whether you write it yourself over atomics, use `rtrb` (0.4.0, 2026-08-18) for a single-process SPSC ring, or use `crossbeam-queue`'s `ArrayQueue` (0.3.14, 2026-09-05) for a bounded MPMC ring, the cache-line padding, the memory ordering and the false-sharing discipline are the sibling guide's material. Rust gives you the same tools with `CachePadded` and `Ordering` as the spelling.
- **Do not put a `Mutex` in shared memory.** A `std`/`parking_lot` mutex is a process-local object; it is meaningless across processes and its contents across a mapping are undefined. Cross-process synchronisation must be built on atomics with a documented protocol, exactly as in C++.

### 7.6 Rust as a library consumed by C++, and Rust calling C++

**Rust → C++.** Rust cannot call C++ directly. The options, in ascending order of pain: (a) ask the C++ side for a C API — by far the best outcome and the one to engineer for; (b) write a thin C shim by hand around the C++ class; (c) use an automated binding generator for C++ (the `cxx`/`autocxx` family) — I did not verify these crates' current versions in this pass, so treat them as *available in the ecosystem, unverified here* and check versions before adopting. Option (c) is attractive for a C++ library with a rich API and unattractive for a latency-critical call, because generated shims are hard to audit and add their own abstraction layer to the hot path.

**Rust → a shared library consumed by C++.** This is the pattern a hybrid trading system actually uses. Build a `cdylib`, expose a narrow `extern "C"` surface, and generate the header with `cbindgen` (0.29.4) so the C++ side cannot drift from the Rust declaration. Keep the surface to: create, destroy, submit, poll, and a small set of `#[repr(C)]` accessors. Do not expose Rust's collections, traits or errors — map errors onto an enum with a fixed representation, and never let a Rust `panic` cross the boundary (with `panic = "abort"` it cannot).

**Why the `unsafe` FFI blocks are the auditability story, not a loophole.** This is the strongest non-performance argument in Rust's favour for a supervised environment, and it is worth stating precisely:

- Unsafe code is *marked*. Every operation whose soundness the compiler cannot check sits inside an explicit `unsafe {}` block. That gives an auditor a countable, locatable set of review targets rather than "the whole codebase in C++ terms."
- The 2024 edition strengthened this. Verified at source: the `unsafe_op_in_unsafe_fn` lint "now warns by default," so an `unsafe fn` no longer implicitly permits unsafe operations in its body — each one needs its own explicit `unsafe {}` block. The edition guide explains the reasoning: the `unsafe` keyword had two roles, and the second one (permitting unsafe operations inside the function) "was determined to be too risky without explicit `unsafe` blocks."
- The boundary is therefore auditable *by construction*: the `unsafe` block carries a documented safety contract (what the caller must guarantee) that a reviewer can check, and everything outside it is compiler-verified.

**The tooling that makes that claim testable.** Rust has a development-time undefined-behaviour checker, Miri, which interprets Rust's MIR and detects many classes of UB in `unsafe` code during tests; and the compiler can be built against LLVM/Clang sanitizers. **I did not verify Miri's or the sanitizer integration's current status or version in this pass** — treat that as named-but-unverified, and confirm before making a compliance statement that depends on it. The point of naming them is that the "audit the `unsafe` surface" claim is only credible if it is backed by a runtime and static analysis story, which Rust has and C++ arguably has in a different form (sibling §12).
## 8. The Trading and Market-Data Application

**The architecture is owned by the sibling guide.** Feed handler, book builder, strategy, pre-trade risk, order gateway and their sequencing are `low_latency_cpp_development_guide.md` §9; the wire formats are its §8. This section builds only the *Rust expression* of that architecture.

### 8.1 The Rust expression, component by component

| Component | Path | Rust expression |
|---|---|---|
| Feed handler (decode, normalise) | hot | Hand-written parser over `zerocopy`/`bytemuck` for the wire layout; `arrayvec`/`smallvec` for variable-length fields; no allocation; `get`-based bounds handling; a reused read buffer |
| Book builder | hot | `Vec` of price levels plus a `slab`-indexed order store; hand-written SoA for hot fields; no hashing if the instrument set is fixed and indexable by a dense `u16` |
| Strategy / signal | hot | Plain structs, enums and `match`; no `dyn Trait`; no `Arc` on the message path |
| Pre-trade risk | hot, must not block | Dense array of per-instrument limits indexed by instrument slot; bounded ring for the handoff; zero locks |
| Order gateway session | hot transmit, warm management | Hand-rolled state machine per session; encoded into a pre-sized buffer; no `format!` |
| Persistence | warm | `serde`/`bincode` acceptable here (`bincode` 3.0.0, 2025-12-16) |
| Metrics, audit, drops; config and reference data | cold | Fixed-size records into a bounded ring, formatted by a cold thread; `itoa`/`ryu` for numbers; `HashMap` with any hasher is fine; `arc-swap` (1.9.2, 2026-06-28) for atomic publication of a new version |

**The shape is deliberately un-idiomatic Rust**: no `Arc`, no `dyn`, no `Mutex`, no `async`, minimal `Result` propagation into the inner loop (errors collected and handled at the batch boundary), and heavy use of indices rather than references to sidestep the lifetime friction of §3.1. Warm and cold paths are ordinary idiomatic Rust. Getting that split right is the design work; the language does not enforce it.

### 8.2 The feed handler and the SIMD question

**The wire format decides the parser.** If the protocol is fixed-width binary with a variable-length symbol field — the ITCH-family shape (sibling §8) — a Rust parser is unaligned loads and byte-swaps over a validated slice. The performance work is proving the message length once so bounds checks elide (§3.4), avoiding allocation (§3.1), and keeping the parse in registers. There is almost no SIMD surface: SIMD accelerates *scanning for structure*, and a fixed-offset binary parse has little to scan for.

**Where SIMD does pay:** textual protocols (FIX, ASCII ITCH variants, JSON on a REST path). `simdutf8` (0.1.5, 2024-09-22) for validation and `simd-json` (0.18.1, 2026-08-23) for parsing are real, stable-channel SIMD parsers. Note the constraint: both are built on `std::arch`, not portable SIMD, which remains nightly-only (§4.8).

**The claim to avoid.** I could not find a published, methodologically credible Rust-vs-C++ benchmark for market-data feed parsing specifically. Treat any such number in a blog post as unverified until you reproduce it with your own wire format, because the format dominates the result.

### 8.3 The order gateway and the session/risk path

The gateway has a different profile from the feed handler: it transmits, so it cares about the send path (in a kernel-bypass deployment, the vendor SDK — §7.4) and about session state. In Rust: use an `enum` of session states with `match` rather than trait objects, so transitions are one jump table in one place; give the risk check ownership of its limits and the gateway ownership of the session, and connect them with a bounded wait-free ring (`rtrb`, 0.4.0, 2026-08-18) — Rust forces that ownership question to be answered, which the sibling guide would want answered anyway; encode into a pre-sized `[u8; N]` or a reused `Vec` with `clear()`, never `format!`.

**This is where `async` competes with a hand-written loop.** Few sessions, always-ready transmit: hand-write the loop (sibling §6). Many sessions over a network with variable client latency: `async` earns its place, and Databento's published reasoning is the honest case — async I/O lets a slow client be served less often without punishing clients who keep up, and Rust's ownership model plus Tokio let them iterate faster without data-race anxiety.

### 8.4 Book-building data structures and cache behaviour

Two patterns work. **(1) A dense array of levels indexed by price tick**: the price range is known, so `(price − floor_tick)` indexes directly — no hashing, no search, cache-friendly by construction, bounded by the price range. **(2) A `Vec` of levels in price order plus a `slab` (0.4.12) of orders indexed by slot**: `slab` gives stable integer keys, so a cancel is an index lookup, and there is no per-order allocation after warm-up. This is the closest Rust gets to the array-indexed C++ design, and it is genuinely good.

**What Rust gives up.** The self-referential book structure — an owning book whose subcomponents hold references to its fields, the natural C++ design Databento demonstrated — does not compile. The workarounds (index-based references, threading the owned structure through every method, or `Rc`-ing the inner structure) change either the code shape or the memory traffic. For a book builder that is a real constraint: the `Rc` workaround adds refcount traffic to the hot path (§3.2).

### 8.5 The hot/warm/cold split

| Path | Budget posture | Rust practices |
|---|---|---|
| **Hot** (per message) | No allocation, no locks, no `format!`, no FFI, no panics, no syscalls | `arrayvec`/`smallvec`, dense indices, `Cell` counters, `#[inline]`, iteration over slices, an allocation-counting test that fails the build above zero |
| **Warm** (per batch/order/session event) | A small, bounded, counted number of allocations | `slab`, `object-pool`, a bounded `crossbeam-queue::ArrayQueue`, `parking_lot` locks, `serde` |
| **Cold** (per second/config change/audit flush) | Anything, but measured | `HashMap` with any hasher, `serde_json`, `tracing`, formatting, file I/O, `Arc<Mutex<_>>` |

Make it a type-level boundary where you can: a hot type with no `String` field cannot allocate by accident. That is a genuine Rust advantage — the compiler enforces what in C++ is a convention in a coding standard (sibling §16).

### 8.6 The observability overhead budget

Treat observability as a line item in the latency budget (sibling §2), not an afterthought. **Hot path:** increment pre-allocated `Cell<u64>` counters and write fixed-size records into a bounded ring — no formatting, no locks, no syscalls. **Warm path:** a drain thread reads the ring. **Cold path:** format, aggregate, ship. **Enforce it:** an allocation-count assertion on the hot path, and a latency measurement taken with logging at its *production* level enabled — a measurement taken with logging off does not tell you the cost of logging on.

## 9. The Honest Comparison with C++

### 9.1 Where Rust's guarantees genuinely buy latency or correctness

| Claim | Mechanism | Evidence |
|---|---|---|
| No GC pause to bound | Scope-based deterministic destruction; no tracing collector exists to suspend a thread | Strong — language design |
| Data races are a compile error | Ownership plus `Send`/`Sync`; the borrow checker forbids simultaneous mutable aliasing | Strong — the mechanism is the compiler's |
| Buffer overflows, use-after-free and null derefs are caught at compile time | Bounds checking, lifetimes, and no null (`Option` instead) | Strong — but only for *safe* code; every `unsafe` block and FFI call is an explicit exception (§7.6) |
| The unsafe surface is enumerable and auditable | `unsafe {}` blocks are syntactically marked, and 2024's `unsafe_op_in_unsafe_fn` warns by default so even `unsafe fn` bodies require explicit blocks | Strong — verified at the edition guide |
| Iteration and generics cost nothing | Monomorphisation plus inlining | Moderate — true when the profile agrees; verify with instruction counting |

Databento's published reasoning for choosing Rust for their live subscription gateway is the cleanest primary-source statement of these benefits in a trading context: memory safety for a network-exposed entry point, "not wanting a runtime with a garbage collector," the availability of Tokio (contrasted with C++, where "asynchronous programming is far from being a first-class citizen"), and thread safety. They state in the same breath that their own low-level data-sharing primitives "are mostly written in C" and integrated via Rust's FFI — the hybrid model (§7.1).

### 9.2 Where C++ retains the advantage

| Advantage | Evidence |
|---|---|
| **Mature ecosystem at the vendor layer** | Verified. Databento (Rust-positive) states "most exchanges provide native C++ APIs, and critical infrastructure components like FPGA interfaces and kernel bypass networking libraries target C++ first." §4.7 corroborates: no maintained Rust DPDK binding found, no vendor Rust SDKs |
| **Existing codebases and code reuse** | Verified by the strongest available case: Databento chose C++ for their feed-handler rewrite because it allowed "code reuse from our existing implementation, speeding up the rewrite" |
| **Team expertise** | Verified in the same case: "we have many more years of collective C++ experience," so everyone was productive immediately |
| **Control over resource sharing and memory layout** | Verified in the same case — the three frictions were buffer reuse across lifetimes, self-referential structs, and compile-time generic versioning boilerplate |
| **Toolchain depth and tuning lore** | Verified as a claim (GCC/Clang, ASAN, Valgrind, dedicated latency tooling). Rust now has real equivalents (§6), so this is depth and lore, not capability |
| **Talent pool** | **Flagged, not verified.** Asserted by Databento and consistent with ecosystem surveys that Rust experience is scarcer; no quantitative comparison was verified here |

**A nuance that cuts against the folklore.** Databento's C++ choice was made *without* a benchmark: their post states "This wasn't a decision driven by performance benchmarks — we didn't write the feed handler in both languages to compare them." The most-cited real-world C++-over-Rust decision in this domain was driven by ergonomics, code reuse and team familiarity, not measured latency. That is a legitimate basis for a decision, and it should be reported as what it is.

### 9.3 Where the two are equivalent

**The memory model** — Rust inherits C++20's atomics model deliberately (verified: Rustonomicon), so the same orderings, the same hardware consequences, the same reasoning. **Cache behaviour of a given layout** — cache lines, false sharing, prefetching and NUMA are properties of the machine (sibling §3); Rust neither helps nor hinders, it only changes how easily you express SoA and padding. **Peak achievable throughput on a well-written hot loop** — both are ahead-of-time through LLVM with the same codegen levers (§10 correlates to sibling §14); Databento's own assessment is that the two are "largely comparable in terms of performance." **The need for the measurement discipline** — neither language measures latency for you; sibling §12 and §13 apply unchanged.

### 9.4 Migration and interop cost

Component-by-component migration over a C FFI boundary is the practical path and is the pattern Databento runs in production ("Rust and C++ can interact through C FFI… We do this at Databento today"). The boundary's cost is **structural, not per-call** (§7.3): lost inlining across the seam, no cross-language LTO without `RUSTFLAGS`, and a build system that must link a vendor SDK built with a possibly-different toolchain. Budget most of the migration effort for the build and the C-shim layer, not the Rust. And note the seam is permanent until the last component moves — which in a fifteen-year estate may be never.

### 9.5 What the evidence base actually supports

**Supported.** Rust is in production in trading- and market-data-adjacent systems at a named, primary-source-documented firm. Databento states that more than half of its core infrastructure is written in Rust, that its DBN encoding reference implementation and real-time subscription gateway are Rust, and that its Rust client library performs comparably to its C++ client. Their operational figures — a feed handler processing 14 million messages per second with sub-100 µs requirements, and a client library comparable to a C++ client achieving historical order-book replay at 19 million events per second — are the vendor's own numbers about the vendor's own system: first-party claims, not independently audited, and to be read as such.

**Not supported, and to be rejected.** (a) That Rust has displaced C++ in the ultra-low-latency trading core — the same firm that uses Rust heavily chose C++23 for its feed-handler rewrite. (b) That Rust is *faster* than C++ here — no credible, reproducible, methodology-complete benchmark exists. (c) That "Rust is used by HFT firms" without a named firm and a primary account is a marketing statement; a large fraction of search results making that claim are SEO-optimised or AI-generated, with no named author, no methodology and no reproducible artifact. When this guide searched for primary accounts, most hits for "Rust for low-latency trading" were exactly that. **The honest summary: the correctness case for Rust is well-evidenced and the latency case is not — because in the cases studied, the latency difference was not the deciding factor.**

## 10. The Build, Toolchain and Compiler Levers

**Build philosophy is the sibling guide's** (`low_latency_cpp_development_guide.md` §14). This covers only Rust/Cargo levers, verified against the Cargo and rustc Books on 17 September 2026.

### 10.1 The release profile defaults — verified exactly

`release`: `opt-level = 3`, `debug = false`, `strip = "none"`, `debug-assertions = false`, `overflow-checks = false`, `lto = false`, `panic = 'unwind'`, `incremental = false`, `codegen-units = 16`, `rpath = false`. For contrast, `dev`: `opt-level = 0`, `debug = true`, `debug-assertions = true`, `overflow-checks = true`, `lto = false`, `panic = 'unwind'`, `incremental = true`, `codegen-units = 256`.

**The gotcha, stated precisely.** `cargo build --release` sets **both** `debug-assertions = false` *and* `overflow-checks = false`. There is no "release still enables overflow checks" — it does not. The trap runs the other way: enabling `overflow-checks = true` in the release profile "to be safe" adds a **panic** to every arithmetic operation on the hot path, and with it the unwind-inducing code paths §3.5 describes. The correct reading: release disables both, which is what you want for latency, at the cost of silent wrapping — so use explicit `checked_`/`wrapping_`/`saturating_` at the few places where intent matters.

### 10.2 The levers that matter, and their cost

| Lever | Effect | Cost |
|---|---|---|
| `opt-level = 3` (default) | Full optimisation | Compile time. The Cargo Book is explicit that higher is not always faster — "there may be surprising results, such as level `3` being slower than `2`." Measure |
| `codegen-units = 1` | More optimisation opportunity across the crate | Slower builds; the Perf Book notes it may improve runtime speed and reduce binary size at the cost of compile time |
| `lto = "thin"` / `"fat"` | Whole-program optimisation; the Perf Book reports "10–20% or more" possible (its general figure) | Much slower builds; `fat` may improve performance "but not always" |
| `panic = "abort"` | Removes unwinding tables; tighter codegen | A panic becomes an outage — combine with the no-panic discipline (§3.5) |
| `-C target-cpu=native` | Tune instruction selection and scheduling for the build machine | Binary is not portable; build and run must be the same CPU generation |
| `-C target-feature=+avx2,…` | Enable specific instruction sets | The rustc Book declares this unsafe: "Using this flag is unsafe and might result in undefined runtime behavior." Prefer `is_x86_feature_detected!` unless you control the fleet |
| PGO | Real execution profiles guide inlining, layout and register allocation | Four steps: `-Cprofile-generate=…`, run, `llvm-profdata merge`, `-Cprofile-use=…`. `llvm-profdata` comes from `rustup component add llvm-tools-preview` and is not added to `PATH` |
| `cargo-pgo` (0.3.0, 2026-01-25) | Wraps PGO: `cargo pgo build && cargo pgo optimize` | Small adoption (~69k downloads); the rustc Book endorses it as "an alternative… with an intuitive command-line API" |

**On BOLT.** The Perf Book recommends considering `cargo-pgo` "if it works with your distribution mechanism." **I did not verify a current, documented BOLT integration for Rust**, so treat "BOLT works with rustc" as flagged (§14).

### 10.3 Make the measured binary the shipped binary

The most common Rust build mistake in a latency context is measuring one profile and shipping another. `cargo bench` inherits `release`, so it is safe; a hand-rolled timing harness or a `dev` build is not. Two verified wrinkles: **"Tests, benchmarks, build scripts, and proc macros ignore the `panic` setting"** — the test harness requires unwinding, so you cannot validate a `panic = "abort"` build through `cargo test`; and with the `abort` strategy, building a test forces all dependencies to `unwind`, so the panic strategy in tests differs from production by construction. Define a greppable production profile and measure through it — a `lat` profile inheriting `release` with `lto = "fat"`, `codegen-units = 1`, `panic = "abort"`, plus a `lat-bench` variant that adds `debug = "line-tables-only"` so `perf` flamegraphs remain usable (§6.2). Document that the two differ only in debug info, and re-verify after any toolchain upgrade.

### 10.4 Inlining and cross-language LTO

`#[inline]`, `#[inline(always)]` and `#[cold]` are the language levers. `#[inline]` matters most for a small function in a *different crate* (within a crate LLVM already sees the body); `#[inline(always)]` should be justified by measurement, since forcing inlining in a cold path bloats the instruction cache — a latency cost, not a size cost (sibling §3). `#[cold]` on an error or panic path is the highest-value of the three and is usually free: it tells the optimiser to keep the hot path contiguous. Cargo's profile documentation also notes that at `opt-level` 2 or 3 "a crate will not use monomorphized generics from other crates," which interacts with attempts to optimise a dependency via profile overrides.

**Cross-language LTO** — what would let Rust and C++ optimise across the FFI seam (§7.3) — is verified as *not natively supported by Cargo*: "This is not yet supported natively in Cargo, but can be performed via `RUSTFLAGS`" (`-C linker-plugin-lto`). The implication for a hybrid system is serious: you cannot get a single optimised whole-program view of a Rust-plus-C++ hot path through Cargo alone. Budget for a custom build, or accept the opaque seam.

### 10.5 Pinning, reproducibility and build cost

**Pin the toolchain.** A `rust-toolchain.toml` naming an exact version is the difference between "it was fast when we benchmarked it" and "it is fast in production." Rust ships every six weeks and codegen changes between releases: a stable upgrade can help or hurt a hot loop, and a toolchain change must be treated as a performance change to be measured (sibling §15). **Lock the graph:** commit `Cargo.lock` and run an audit step in CI for §11.1's supply-chain argument; note that a lockfile pins versions, not code, so vendoring or a checksummed mirror is the stronger control. **Reproducibility** has a Rust wrinkle — builds can embed paths and toolchain metadata — answered by `--remap-path-prefix` and containerised builds; verify bit-for-bit reproducibility yourself rather than assuming it. **Build cost is the real operational tax:** `codegen-units = 1` plus `lto = "fat"` plus PGO turns a famously fast incremental build into a slow batch build. Plan CI for the *shipping* profile, not the default one.

## 11. The Regulated-Institution Angle

**Condensed and cross-referenced.** The regulatory framing, timestamping obligations and the economics of microseconds are the sibling guide's (`low_latency_cpp_development_guide.md` §18). The repo's resilience and governance content is at [`banking/operational_resilience_framework_guide.md`](../banking/operational_resilience_framework_guide.md) — read it there. This covers only the argument *specific to Rust*.

### 11.1 What a bank actually gets

| Benefit | Why it holds up in a supervised estate |
|---|---|
| **An enumerable safety argument** | The unsafe surface is syntactically marked, and 2024's `unsafe_op_in_unsafe_fn` makes even `unsafe fn` bodies mark each operation. An auditor can be given a *count* of unsafe operations and a review list, rather than being told "the codebase is C++." This is the strongest argument and it is structural |
| **A reduced class of exploitable defects** | Buffer overflow, use-after-free and data races are compile errors in safe code. For a network-facing gateway — Databento's stated reason for choosing Rust — this removes the category that memory-safety statistics repeatedly identify as dominant |
| **Concurrency correctness by construction** | `Send`/`Sync` plus the borrow checker mean a data race requires `unsafe` or a wrong `unsafe impl`. The eliminated class includes the worst production bug: the intermittent, load-dependent stall |
| **Tractable supply-chain vetting** | Cargo's single dependency graph, a committed lockfile and audit tooling make the dependency surface enumerable — a genuine advantage over a heterogeneous C++ build tree with vendored submodules (with the flip side in §11.2) |
| **Lower onboarding and review cost** | `cargo` unifies build, test, bench, docs and dependencies, and diagnostics are unusually good — Databento calls them "on another level" versus Clang/GCC. Where code is reviewed by people who did not write it, that is a governance benefit |

### 11.2 The counterweights, stated honestly

| Counterweight | Substance |
|---|---|
| **Hiring market** | Fewer candidates with Rust systems experience. Every mitigation costs something, and the cost lands in the first two years and then declines |
| **Toolchain and support maturity** | There is no commercial vendor support contract for the Rust toolchain itself in the way there is for a C++ compiler or a Linux distribution. Regulated estates often require a named support relationship for critical components; without one, the argument must be met with in-house capability and a pinned, vendored toolchain |
| **The vendor ecosystem gap** | Kernel bypass and FPGA are C/C++. This matters *more* in a supervised estate, because the vendor SDK is often the component with the support contract, the certified version matrix and the audit trail — and Rust sits outside all of it. A Rust trading system does not remove the C/C++ dependency; it adds a language boundary to it (§7.1) |
| **The fifteen-year horizon** | Rust is eleven years past 1.0 as of 2026, a real track record, but the *crate* ecosystem turns over faster than the language: several crates in §4 are already slow or stale (`jemallocator`, `async-std` discontinued, `tokio-uring`, `glommio`, `monoio`). A fifteen-year plan must assume you will own, fork or replace dependencies |
| **Assurance tooling** | C++ has a longer history of certified toolchains, static analysis suites and safety cases. Rust's equivalent (Miri, sanitizer integration, `no-panic`) is credible and improving but less mature, and its certification status was not verified here (§14) |

### 11.3 Which bank workloads justify a new language at all

Apply this filter before any language comparison. **Strongest case:** a new latency-critical component with a new team — marginal cost of the language choice is lowest and the correctness dividend accrues from day one. **Strong:** a new network-facing service with a security exposure, which is the case Databento articulates for their gateway. **Weak:** a component whose hot path *is* the kernel-bypass path — you will write FFI wrappers around most of the value and the language's guarantees stop at the boundary. **Usually not a case:** an existing C++ component with a working, tuned hot path; Databento's decision, made with Rust already in-house and a rewrite already justified on architectural grounds, was still to keep C++. **A correctness case, not a latency case:** a latency-tolerant component — justify Rust on maintainability and defect class instead.
## 12. The Cymbal Bank Worked Example

> **This section is illustrative.** Cymbal Bank is a fictional institution used as a persona in this repo. Every number below is a *planning assumption or a design target*, clearly labelled as such. **Nothing here is a measured result and no figure here should be cited as evidence of anything.** Where a real external figure appears it is attributed in §13.

### 12.1 The workload profile

**Scenario.** Cymbal Bank's electronic-trading and payments-latency initiative must choose an implementation language for a **new** latency-critical component: an order-gateway *session and pre-trade risk layer* sitting between the existing strategy engine and the venue connectivity stack. It is new, network-facing, and must deploy into an estate that is overwhelmingly C++ (venue connectivity, kernel bypass) and Java (order management, settlement).

| Property | Assumption | Consequence |
|---|---|---|
| Outbound order intents per second, peak | ~200,000 | Modest by low-latency standards; not an HFT core |
| Requirement | p99.9 intent-to-wire-write ≤ 80 µs *within this component* | A component budget; the end-to-end method is the sibling guide's (`low_latency_cpp_development_guide.md` §2) |
| Venue sessions | <20, persistent | Small, known fd set — favours a hand-written loop over async (§5.3) |
| **Hot-path calls into the kernel-bypass stack** | **Zero per message; all at setup** | **The decisive assumption (§7.3)** |
| Regulatory obligation | Every order intent recorded and timestamped; retained | Observability must be budgeted, not bolted on (§8.6) |

**The row that decides the question** is the fourth. If the session/risk layer never calls the C++ kernel-bypass SDK on the hot path, the language choice is confined to pure compute and state — where Rust competes well. If it must call the vendor stack per message, the ecosystem gap bites and the honest answer is to keep this component in C++ alongside its neighbours.

### 12.2 The team and the trade-off

**Team (illustrative).** Eight engineers: six with deep C++, two with strong Java and no Rust, two with Rust side projects. No production Rust experience. This is the realistic case for a bank of this size.

| Criterion | C++ | Rust | Weighting here |
|---|---|---|---|
| Meets 80 µs p99.9 | Yes (proven in the estate) | Likely — no GC, deterministic destruction, same memory model, comparable codegen | **Equal.** Evidence supports "comparable", not "faster" (§9.5) |
| Defect class removed | Baseline | Data races, use-after-free, buffer overflows are compile errors in safe code; unsafe surface is enumerable | **Rust.** The real prize for a network-facing, multi-threaded component |
| Ecosystem where this component touches it | Strong | Strong at the application layer — and this component *is* the application layer | **Equal**, but only because of §12.1's fourth row |
| Ecosystem at the kernel-bypass layer, and vendor SDK/support-contract coverage | Strong; inside existing vendor relationships, with a certified version matrix | **Absent** (no maintained DPDK binding; vendor SDKs are C/C++ and outside those relationships) | **C++**, decisive if the boundary moved onto the message path |
| Existing code reuse | High (validation, risk models, encoding) | Low | **C++** for 12–18 months |
| Time to first production release | Lower | Higher (learning curve plus new build integration) | **C++**, materially |
| 10-year maintenance cost | Familiar; the estate's defect rate persists | Higher early, lower if the defect class is genuinely gone | **Rust**, long-run |
| Hiring and succession | Deep pool | Scarcer; one to two external hires or a training programme | **C++**, first two years |

### 12.3 The interop plan

Adopt by component, not by rewrite, using a C ABI seam (§7.2, §9.4):

1. **Define the seam first** — a narrow `extern "C"` surface on the Rust side (create, destroy, submit-intent, poll-completion plus `#[repr(C)]` accessors), with the header generated by `cbindgen` (0.29.4) so it cannot drift.
2. **Java reaches the component through the existing transport**, not JNI. The order-management plane already talks to the trading estate over a message bus; keep it that way.
3. **C++ reaches Rust on cold and warm paths only.** If a hot-path C++→Rust call becomes necessary, re-examine the design rather than paying the seam cost (§7.3).
4. **No `Mutex` in shared memory; no shared synchronisation primitive across the language boundary** (§7.5). Cross-process handoff uses atomics with a documented protocol, as the C++ side already does.
5. **Both sides build in the same CI**, with the Rust profile pinned and the boundary contract tested in both directions. Budget most of the integration effort here.

### 12.4 The comparison — illustrative only

> **Read the label.** The table below is a *planning artefact* showing the **shape** of the decision. The values are illustrative assumptions authored for this example. They are **not measurements**, they are **not sourced**, and **no build was performed**. The real method for obtaining such numbers is the sibling guide's §2, §12 and §13, applied by the reader.

| Cost line | C++ path (illustrative) | Rust path (illustrative) | Where the Rust cost comes from |
|---|---|---|---|
| Validation / risk-model reuse | Mostly free | Rewrite or FFI-wrap | §9.2 — the verified code-reuse advantage |
| New build integration | Known | New `build.rs` work, link-time unknowns | §7.2 |
| Engineer ramp-up | None | Ownership model, lifetimes, no-panic discipline | §2.3 |
| Hot-path language work | Familiar idioms | Allocation-free discipline, hasher choice, bounds-check elision | §3 |

### 12.5 Risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Ramp-up exceeds estimate | Medium | Schedule slip | Four-week single-engineer prototype; gate the decision on the prototype, not the plan |
| Hot path drifts onto the FFI seam | Medium | Ecosystem-gap advantage evaporates | Make the seam rule explicit in design review; test that the boundary is off the message path |
| Toolchain upgrade changes hot-loop codegen | Medium | Silent regression after routine maintenance | Pin via `rust-toolchain.toml`; treat every bump as a measured performance change (§10.5) |
| A critical crate goes stale | Medium | Forced fork or replacement in year 4 | Vet release cadence at adoption; keep the hot-path dependency count small (§4) |
| No internal Rust depth after the sponsor leaves | Medium | Component becomes unmaintainable | Train two engineers, not one; write an internal safe/unsafe standard |

### 12.6 Recommendation, and what must be true

**Recommendation: build the new session and pre-trade risk layer in Rust, behind a C ABI seam, with the kernel-bypass stack kept strictly off the message path — and keep every existing C++ latency component in C++.**

The reasoning: the component is new (so C++'s code-reuse advantage does not apply), it is network-facing and multi-threaded (so the correctness dividend is at maximum), and it does not need to call the vendor SDK per message (§12.1). The ecosystem gap therefore does not bite; where it does bite — in the connectivity layer below — the estate stays in C++ behind a cold-path seam.

**This holds only if all of the following are true.** If any is false, choose C++ and revisit in two years:

1. The hot path can be designed with **zero per-message FFI calls** into the vendor stack, and that rule survives design review.
2. The bank accepts a **schedule premium** on the first release in exchange for a lower defect rate thereafter, and the sponsor will defend that trade when the release slips.
3. **Two engineers**, not one, are trained to competence, and a written internal standard exists for the safe/`unsafe` split.
4. The **toolchain is pinned**, and a toolchain upgrade is treated as a measured performance change.
5. Latency is verified with the **production build profile** and instruction counting, not a default `cargo bench`.
6. Someone owns the **decade-long dependency-maintenance plan**, with a list of crates that would force a fork if abandoned.

If conditions 1, 2 or 3 cannot be met, the correct answer is C++ — which is the same judgement a firm running Rust in production made about its feed handler in October 2025.

## 13. The Claims Audit

Every load-bearing claim, with source, source quality and date checked. All checks **17 September 2026** unless stated. Vocabulary: **primary** (the language's or crate's own documentation/repository), **first-party vendor** (a firm describing its own system), **flagged** (asserted but not substantiated), **rejected** (not supported).

| # | Claim | Verdict | Source | Quality | Checked |
|---|---|---|---|---|---|
| 1 | Stable Rust is 1.98.1; beta 1.99.0 on 1 Oct 2026, nightly 1.100.0 on 12 Nov 2026 | Verified | [releases.rs](https://releases.rs/) | Primary | 2026-09-17 |
| 2 | Current edition is Rust 2024, released in 1.85.0; it narrowed tail-expression temporary drop scope (`tail_expr_drop_order` lint) and made `unsafe_op_in_unsafe_fn` warn by default | Verified | [Edition Guide](https://doc.rust-lang.org/edition-guide/rust-2024/index.html), [tail-expr scope](https://doc.rust-lang.org/edition-guide/rust-2024/temporary-tail-expr-scope.html), [unsafe-op-in-unsafe-fn](https://doc.rust-lang.org/edition-guide/rust-2024/unsafe-op-in-unsafe-fn.html) | Primary | 2026-09-17 |
| 3 | `std::simd` is nightly-only behind `portable_simd`, no stabilisation timeline; `std::arch` is stable | Verified | [std::simd nightly](https://doc.rust-lang.org/nightly/std/simd/index.html) | Primary | 2026-09-17 |
| 4 | `allocator_api` / `Allocator` still unstable (issue #32838 open, `B-unstable`), while `#[global_allocator]` with `GlobalAlloc` is stable | Verified | [rust#32838](https://github.com/rust-lang/rust/issues/32838), std documentation | Primary | 2026-09-17 |
| 5 | `release` defaults `opt-level = 3`, `debug-assertions = false`, `overflow-checks = false`, `codegen-units = 16`, `lto = false`, `panic = 'unwind'`, `incremental = false`; tests, benches and build scripts ignore the `panic` setting (tests force `unwind`); cross-language LTO is not native to Cargo | Verified | [Cargo Book — Profiles](https://doc.rust-lang.org/cargo/reference/profiles.html) | Primary | 2026-09-17 |
| 6 | PGO workflow: `-Cprofile-generate` → run → `llvm-profdata merge` → `-Cprofile-use` | Verified | [rustc Book — PGO](https://doc.rust-lang.org/rustc/profile-guided-optimization.html) | Primary | 2026-09-17 |
| 7 | `HashMap`'s default hasher is SipHash 1-3 via `RandomState`/`DefaultHasher`; moving rustc away from `fxhash` cost 4–84% | Verified | [Perf Book — Hashing](https://nnethercote.github.io/perf-book/hashing.html), [RandomState](https://doc.rust-lang.org/std/collections/hash_map/struct.RandomState.html) | Primary (cited measurement) | 2026-09-17 |
| 8 | `black_box` is best-effort and affects output not input; `#[bench]` needs the nightly `test` feature; a naive bench can measure 0 ns/iter | Verified | [black_box](https://doc.rust-lang.org/std/hint/fn.black_box.html), [Unstable Book — test](https://doc.rust-lang.org/unstable-book/library-features/test.html) | Primary | 2026-09-17 |
| 9 | Rust inherits the C++20 atomics memory model deliberately; `CachePadded` pads to 128 B on x86-64/aarch64/powerpc64 | Verified (quote) | [Rustonomicon — Atomics](https://doc.rust-lang.org/nomicon/atomics.html), [docs.rs CachePadded](https://docs.rs/crossbeam-utils/latest/crossbeam_utils/struct.CachePadded.html) | Primary | 2026-09-17 |
| 10 | `catch_unwind` is not a general try/catch, may miss panics, and dropping its `Err` may panic | Verified | [catch_unwind](https://doc.rust-lang.org/std/panic/fn.catch_unwind.html) | Primary | 2026-09-17 |
| 11 | `no_std` loses heap, collections, `HashMap`, stack-overflow protection and pre-`main` init | Verified | [Embedded Book — no_std](https://docs.rust-embedded.org/book/intro/no-std.html) | Primary | 2026-09-17 |
| 12 | Rust memory is reclaimed by scope-based deterministic destruction; there is no tracing collector | Verified | Rust Reference — destructors and temporary scopes | Primary | 2026-09-17 |
| 13 | All crate versions in §4; no crate named `snmalloc` (404); the flamegraph crate is `flamegraph`, not `cargo-flamegraph` (404); crates.io reports `newest_version` ≠ `max_stable_version` for `rtrb` and `pprof` | Verified | crates.io API, per crate | Primary | 2026-09-17 |
| 14 | `async-std` is discontinued in favour of `smol`; `tokio-uring` is stale (0.5.0, May 2024) and self-describes as "still very young"; Tokio carries an optional `io-uring` dependency (`^0.7.11`) | Verified (README/repo/docs.rs) | [async-std repo](https://github.com/async-rs/async-std), [tokio-uring repo](https://github.com/tokio-rs/tokio-uring), [docs.rs tokio](https://docs.rs/tokio/latest/tokio/runtime/index.html) | Primary | 2026-09-17 |
| 15 | Databento wrote its live subscription gateway in Rust for memory safety, no GC, async ergonomics and thread safety; some low-level sharing primitives are C via FFI | Verified (first-party) | [Databento, Apr 2023](https://databento.com/blog/live-subscription-gateway-rust/) | First-party vendor | 2026-09-17 |
| 16 | Databento chose C++23 for its feed-handler rewrite, driven by buffer-reuse lifetimes, self-referential structs and generic-versioning boilerplate, plus code reuse and team expertise — explicitly *not* by a benchmark, and with Polonius noted as not resolving the lifetime cases | Verified (first-party, self-limiting) | [Databento, Oct 2025](https://databento.com/blog/why-we-didnt-rewrite-our-feed-handler-in-rust) | First-party vendor | 2026-09-17 |
| 17 | Databento: >half of core infrastructure in Rust; feed handler at 14M msg/s with sub-100 µs requirements; a client library comparable to their C++ client replaying 19M events/s | Verified as first-party claims, **not independently audited** | [Databento, Aug 2023](https://databento.com/blog/rust-client-library) | First-party vendor (self-reported) | 2026-09-17 |
| 18 | "Most exchanges provide native C++ APIs, and critical infrastructure components like FPGA interfaces and kernel bypass networking libraries target C++ first"; Rust and C++ "call each other in production" through C FFI | Verified as a first-party assessment | [Databento, Aug 2025](https://databento.com/blog/rust-vs-cpp) | First-party vendor | 2026-09-17 |
| 19 | Rust is faster than C++ for low-latency trading | **Rejected** — no methodology-complete reproducible benchmark found | — | — | 2026-09-17 |
| 20 | Rust has displaced C++ in ultra-low-latency trading cores | **Rejected** — contradicted by row 16 | — | — | 2026-09-17 |
| 21 | A specific microsecond gain from rewriting a latency component in Rust | **Rejected** — every instance found lacked methodology or a named author | — | — | 2026-09-17 |
| 22 | The async book is a stable, complete reference; per-event `tracing` cost with a subscriber; a maintained Rust DPDK/AF_XDP binding; BOLT integration with rustc; Miri/sanitizer certification status | **Flagged** — not established | [async book](https://rust-lang.github.io/async-book/) (self-describes as an incomplete rewrite); others: no source | Primary (self-describing) / — | 2026-09-17 |
| 23 | C++ has a deeper low-latency talent pool | **Flagged** — asserted first-party and plausible; no quantitative comparison verified | Databento, Aug 2025 (assertion) | Flagged | 2026-09-17 |
| 24 | Garbage collectors can be tuned to low or sub-millisecond pauses | **Flagged** — stated qualitatively; no pause figures verified | — | — | 2026-09-17 |
| 25 | "~10 allocations per million instructions ≈ 1% improvement" (rustc experience); LTO "may improve runtime speed by 10–20% or more" | Verified as the book's cited anecdote and general statement — not trading measurements | [Perf Book — Heap Allocations, Build Configuration](https://nnethercote.github.io/perf-book/) | Primary (cited) | 2026-09-17 |

## 14. What Could Not Be Verified

Explicit negative findings. Each is stated as what it is rather than filled with a plausible substitute.

**1. Documented production use of Rust in a latency-critical trading or market-data *core* at a primary source — a partial negative finding.** The strongest primary account found is Databento's, and it is substantial: more than half of core infrastructure in Rust, a Rust real-time gateway, a Rust encoding reference implementation, a Rust client library, with first-party figures. Two limits apply. The *feed handler* — the component with the tightest budget — was rewritten in C++23. And every Rust performance figure is the vendor's own about the vendor's own system, independently unaudited. I found **no** primary account from a bank, market maker, exchange or HFT firm describing a Rust implementation of a tick-to-trade or market-data hot path, with named engineers and a methodology. If such a deployment exists, it is not publicly documented at a source this guide could verify.

**2. No methodology-complete Rust-vs-C++ latency benchmark for market-data parsing.** Every candidate was undated, anonymously authored, methodologically silent, or a marketing page. The firms that have published on the choice explicitly did not benchmark.

**3. No maintained, credibly adopted Rust binding for DPDK or AF_XDP**, and **no first-party Rust SDK from any kernel-bypass or FPGA vendor.** Onload/`ef_vi`, DPDK, VMA, Exanic and the FPGA toolchains are C/C++. Rust reaches them; it does not replace them.

**4. No published, reproducible overhead profile for any Rust async runtime.** The architecture of each runtime is verifiable and is described in §4.6; a *number* with a methodology is not.

**5. No established Rust crate for parsing the actual exchange wire protocols** (ITCH, SBE, FAST). §4.9. The plan must be to write the parser.

**6. Per-event tracing cost.** §3.7. Not established; measure your own subscriber configuration.

**7. BOLT integration with the current Rust toolchain.** §10.2. Not established.

**8. Miri and sanitizer-integration status or certification story.** Named in §7.6 as what makes the unsafe-auditability claim testable; current status unverified.

**9. Quantitative comparison of C++ versus Rust talent availability**, and **certification of any Rust toolchain for a safety-critical assurance regime.** §9.2, §11.2. Both not established.

**10. GC pause figures.** §2.1 declines to quote specific pause numbers because none were verified at a primary source. Widely repeated claims that ZGC or G1 achieve a particular pause profile are not substantiated here.

**11. Specific venue or exchange SDK language support.** §9.2. Typically NDA-restricted; the pattern is asserted from a first-party engineering assessment, not enumerated.

**Standing caveat.** All crate versions and stability statuses are dated 17 September 2026 and are the fastest-decaying claims in this document — they go stale within weeks. The language-level and documentation-level claims are far more durable. Rows 26 (§13) show the registry API itself can be internally inconsistent: re-verify at the registry before writing any version into a manifest.

## 15. Glossary

| Term | Meaning |
|---|---|
| **Borrow checker** | The compiler pass enforcing Rust's ownership, lifetime and aliasing rules — it forbids two live references where one is mutable |
| **Coordinated omission / tail latency** | Measurement terms owned by the sibling guide (§13): the error where a harness fails to record the delay a stalled system would have suffered, and latency at a high percentile (p99, p99.9) respectively |
| **`dhat`** | Rust binding for Valgrind's DHAT heap profiler, plus an allocation-assertion API for tests |
| **`Drop` / drop scope** | Rust's destructor trait, and the statically determined region in which it runs; the 2024 edition changed where tail-expression temporaries drop |
| **False sharing** | Two cores invalidating each other's cache lines by writing different variables that share a line; fixed with `CachePadded` or `#[repr(align(N))]` |
| **FFI** | Foreign Function Interface: how Rust calls C ABI functions and is called from C/C++ |
| **Future / `Waker`** | A value representing an incomplete computation, polled by an executor; the `Waker` signals readiness |
| **Hot / warm / cold path** | Workload fractions with different budgets: per-message, per-batch or per-order, per-second or per-config |
| **io_uring / kernel bypass** | A Linux asynchronous I/O interface using shared submission and completion rings, and the family of techniques that move packet handling out of the kernel into user space (Onload, DPDK, VMA, Exanic). Sibling guide §7 |
| **ITCH / SBE / FAST** | Exchange market-data protocol families. The binary formats are the sibling guide's §8 |
| **Monomorphisation** | Generating a specialised copy of generic code per concrete type at compile time |
| **`no_std`** | Linking `core` instead of `std`; loses collections, `HashMap`, stack-overflow protection and the `std` runtime |
| **PGO** | Profile-Guided Optimisation: compiling with real execution profiles to guide inlining, layout and register allocation |
| **SipHash 1-3** | Rust's default `HashMap` hash algorithm: DoS-resistant and deliberately slow for short keys |
| **SoA / AoS** | Struct-of-arrays versus array-of-structs layout |
| **`tracing`** | Rust's structured, span-based instrumentation framework |
| **Zero-cost abstraction** | A Rust design principle: an abstraction compiling to what hand-written code would. True for generics and iterators; false for `dyn`, `Arc`, `RefCell` and other runtime-checked constructs |

## 16. Cross-References and Further Reading

**The sibling guide (read first).** [`low_latency_cpp_development_guide.md`](low_latency_cpp_development_guide.md) — owns the latency budget model (§2), hardware fundamentals (§3), concurrency (§5), the event loop (§6), networking and kernel bypass (§7), market-data formats (§8), the reference architecture (§9), shared memory and IPC (§10), FPGA (§11), profiling practice (§12), latency measurement (§13), build philosophy (§14), the optimization workflow (§15), pitfalls (§16), real-world numbers (§17), and the banking and regulatory framing (§18).

**Repo context.** [`banking/operational_resilience_framework_guide.md`](../banking/operational_resilience_framework_guide.md) for the operational-resilience and third-party-risk framing behind §11; [`technology/api_governance_guide.md`](api_governance_guide.md) for interface governance at the seam in §7.

**Rust documentation (primary).** [The Rust Performance Book](https://nnethercote.github.io/perf-book/) — allocations, hashing, bounds checks, inlining, type sizes, build configuration. [The Rustonomicon](https://doc.rust-lang.org/nomicon/) — unsafe Rust and atomics. [Cargo Book — Profiles](https://doc.rust-lang.org/cargo/reference/profiles.html). [rustc Book](https://doc.rust-lang.org/rustc/) and its [PGO chapter](https://doc.rust-lang.org/rustc/profile-guided-optimization.html). [Edition Guide](https://doc.rust-lang.org/edition-guide/). [Unstable Book](https://doc.rust-lang.org/unstable-book/). [Embedded Rust Book](https://docs.rust-embedded.org/book/) — `no_std`. [releases.rs](https://releases.rs/) and the [Rust blog](https://blog.rust-lang.org/). The [async book](https://rust-lang.github.io/async-book/) self-describes as an incomplete rewrite.

**Primary engineering accounts (read critically).** Databento: [why we wrote our real-time market gateway in Rust](https://databento.com/blog/live-subscription-gateway-rust/) (Apr 2023); [Rust vs C++ for trading systems](https://databento.com/blog/rust-vs-cpp) (Aug 2025); [why we didn't rewrite our feed handler in Rust](https://databento.com/blog/why-we-didnt-rewrite-our-feed-handler-in-rust) (Oct 2025).

**Crate documentation.** Read the version you resolve, not `latest`: [tokio](https://docs.rs/tokio), [compio](https://docs.rs/compio), [crossbeam](https://docs.rs/crossbeam), [rtrb](https://docs.rs/rtrb), [ringbuf](https://docs.rs/ringbuf), [hdrhistogram](https://docs.rs/hdrhistogram), [criterion](https://docs.rs/criterion), [divan](https://docs.rs/divan), [iai-callgrind](https://docs.rs/iai-callgrind), [dhat](https://docs.rs/dhat), [zerocopy](https://docs.rs/zerocopy), [bytemuck](https://docs.rs/bytemuck), [rkyv](https://docs.rs/rkyv), [no-panic](https://docs.rs/no-panic), [quanta](https://docs.rs/quanta), [coarsetime](https://docs.rs/coarsetime).

## 17. Closing Summary

The Rust-specific delta in low-latency work is narrower than the language's advocates suggest and more interesting than its critics allow.

**What Rust removes:** the possibility of a runtime garbage-collection pause; the possibility of a data race in safe code; and the possibility of a use-after-free or buffer overflow in safe code — the last two being compile errors rather than review comments. That is a genuine dividend, and it is concentrated in the tails and in the production incident count rather than in the median.

**What Rust taxes:** a borrow checker that is a model rather than an oracle, and therefore rejects some safe patterns; destructor timing that is statically chosen and was changed by the 2024 edition; an ecosystem excellent at the application layer and absent at the kernel-bypass and FPGA layers; and a production build that is slow, whose cross-language LTO is not native, and whose crate dependencies need a decade-long maintenance plan.

**What does not change when you switch:** the memory model (inherited from C++20, deliberately); the cache behaviour of a given layout; the codegen backend; the need for the sibling guide's measurement discipline; and — on the available evidence — the achievable latency.

The honest conclusion is the one the best available first-party account reached by choosing C++ for its feed handler while running Rust across the rest of its stack: this is a decision about the shape of the component, the shape of the team, and how much you value a smaller class of catastrophic defect — not a decision about nanoseconds. Choose Rust where the hot path is yours and the correctness dividend compounds; keep C++ where the hot path belongs to the vendor. And in either case, measure your own tail before you claim anything about anyone else's.

The borrow checker's dividend.
