# Solidity: The Language of the Ledger

**The Language-and-Compiler Deep-Dive — from Gavin Wood's 2014 Proposal and the 2015 First Release, through the Type and Storage Model, the solc Pipeline, the 0.5/0.6/0.7/0.8 Breaking-Change Boundaries, and the Migration of Stewardship to the Argot Collective — with a Cymbal Bank Tokenized-Fund Vault Reviewed at Source Level**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking/Technology Domain · Deep-Dive — the smart-contract *language* and its compiler: the type and storage model, the solc pipeline, the version history, the language-level safety mechanics, the analysis ecosystem, the alternatives, and the AI-generation angle
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (technology/, same folder):** [smart_contracts_guide.md](smart_contracts_guide.md) — owns the *standards and security-class* layer: Szabo's concept (§2), the platforms (§3), the EIP/ERC standards process (§4), the token standards ERC-20/721/1155/4626 (§5), the development lifecycle and frameworks (§6), upgradeability and the proxy patterns (§7), **the security deep-dive: reentrancy and the vulnerability classes, SWC (§8)**, the incident record (§9), and the tokenized-fund worked example (§11). This guide references all of those and **re-derives none of them** · [ethereum_guide.md](ethereum_guide.md) — owns the chain: accounts, gas, the EVM, clients, consensus history, the roadmap eras, EIP-4844, L2s, tokenization · [blockchain_technology_guide.md](blockchain_technology_guide.md) — the ledger fundamentals · [web3_technologies_guide.md](web3_technologies_guide.md) — the web3 application layer
> **Companion guides (ai_llm/, prefix `ai_llm/`):** the `ai_llm/` cluster — the LLM-generation, benchmark-contamination, and AI-assisted-audit context that §8 cross-references

**How to use this guide.** §1 overview, provenance and stewardship · §2 the language model · §3 the compiler · §4 the version history · §5 the language-level security *mechanics* (not the vulnerability classes) · §6 the tooling ecosystem, with each tool's status verified · §7 the alternatives · §8 the AI-era angle · §9 the Cymbal Bank worked example · §10–§14 the claims audit, what could not be verified, the glossary, cross-references, and the closing summary.

**Integrity convention.** Every factual claim carries one of three marks: **✅** verified this pass against a primary or authoritative source (named in the §10 claims audit); **⚠** flagged — reported, approximate, single-sourced, fast-moving, or contested; **❌** refuted or not found. Unmarked statements are domain-stable technical knowledge (what `delegatecall` *is*, what an ABI *is*) rather than research claims. Version numbers, dates, breaking changes, compiler flags and advisory identifiers are asserted **only** where a primary source was read; everything else is in §11.


## Table of Contents

1. [The Overview](#1-the-overview)
2. [The Language Model](#2-the-language-model)
3. [The Compiler (solc)](#3-the-compiler-solc)
4. [The Version History and the Breaking Changes](#4-the-version-history-and-the-breaking-changes)
5. [The Language-Level Security Mechanics](#5-the-language-level-security-mechanics)
6. [The Tooling and Analysis Ecosystem](#6-the-tooling-and-analysis-ecosystem)
7. [The Language Alternatives and the Comparison](#7-the-language-alternatives-and-the-comparison)
8. [The AI-Era Angle](#8-the-ai-era-angle)
9. [The Cymbal Bank Worked Example](#9-the-cymbal-bank-worked-example)
10. [The Claims Audit](#10-the-claims-audit)
11. [What Could Not Be Verified](#11-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [Cross-References and Further Reading](#13-cross-references-and-further-reading)
14. [Closing Summary](#14-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Solidity** is a statically typed, contract-oriented, high-level language whose only compilation target is the **Ethereum Virtual Machine (EVM)**. It is the dominant language of the smart-contract ecosystem: most deployed value-bearing contracts on Ethereum and its EVM-compatible chains are Solidity, compiled by **`solc`**, its reference compiler.

It looks like JavaScript or C++ with curly braces, but *storage*, *message calls* and *reverts* are first-class constructs rather than library abstractions. That is the whole design story: in an ordinary language "write to the database" is an API call to something outside the language; in Solidity `myStateVar = 1;` is a direct, priced, permanent write to a globally replicated store. The type system, the data-location keywords and the entire safety posture exist to make that directness survivable.

**Solidity is not the EVM, and the EVM is not Solidity.** The EVM is a stack machine with no native types above 256-bit words; Solidity is one of several source languages compiling to it, and the front end readers actually read. The chain, gas model and EVM are owned by [ethereum_guide.md](ethereum_guide.md); the token standards and vulnerability classes by [smart_contracts_guide.md](smart_contracts_guide.md). **This guide owns the middle layer — the language and the compiler.**

### 1.2 The Key-Facts Table

| Fact | Value | Status |
|---|---|---|
| Language class | Statically typed, contract-oriented, curly-braces; single target: the EVM | — |
| Proposed | **August 2014, by Gavin Wood** | ⚠ §1.5 (secondary sources only) |
| Early development lead | The Ethereum project's Solidity team, led by **Christian Reitwiessner** | ⚠ §1.5 |
| First tagged release | **0.1.0, 10 July 2015** | ✅ §1.5 (`Changelog.md`) |
| Reference compiler | **`solc`** (C++); **`solc-js`** is the emscripten build of the same compiler | ✅ §3 |
| Current released version | **0.8.37 (2026-09-10)**; 0.8.38 *unreleased* at read time | ✅ §4.1 |
| Current steward | **Argot Collective** — independent non-profit formed **October 2024** | ✅ §1.6 |
| Latest breaking boundary | **0.8.0 (2020-12-16)** — checked arithmetic by default | ✅ §4.2 |
| Critical arithmetic change | Overflow/underflow **revert by default** from 0.8.0; `unchecked { }` restores wrapping | ✅ §4.2 |
| Panic signalling | `Panic(uint256)`, selector **`0x4e487b71`**; regular errors `Error(string)`, `0x08c379a0` | ✅ §4.2 |
| Custom errors (`error X(...)`) | Introduced in **0.8.4** (April 2021) | ✅ §2.4 |
| ABI specification home | The **Solidity documentation** ABI spec | ✅ §2.7 |
| Data locations | `storage`, `memory`, `calldata`; explicit since **0.5.0** | ✅ §2.3 |
| Default optimiser `runs` | **200** | ✅ §3.3 |
| Experimental IR backend | `--via-ir`; experimental **SSA-CFG** (`--via-ssa-cfg`) opt-in in **0.8.35** | ✅ §3.2 |
| 1.0 status | Not released; **no published date**; deprecations point at a 0.9.0 | ✅ §4.3 |
| Recommended production pin | An exact `0.8.x` — *not* a floating range | §4.4 |

### 1.3 The Design Goals

Solidity's docs describe it as a "statically-typed curly-braces programming language" targeting the EVM with on-chain state as a first-class concept. The docs' "by example" framing is itself a design statement: learnable quickly by a competent JavaScript or C++ developer, auditable by a reviewer who is not a compiler engineer.

- **Contract-oriented, not object-oriented.** A `contract` is a deployable, addressable unit with its own storage; `this` is not an object reference but an external call.
- **Statically typed.** Every expression has a compile-time type; no `any`, no runtime reflection. Types are cheap because they are erased into gas-relevant operations.
- **Compile-to-EVM and nothing else.** No WASM, native or JVM target; no C++-style standard library, no general heap, no threads — but precompiles and opcodes.
- **Safety over silent surprise — where the ecosystem accepts it.** A goal *revised* over time: the 0.5/0.6/0.7/0.8 releases each removed a footgun (§4.1).
- **Gas is a design input.** `runs` (§3.3), storage packing (§5.2), `calldata` vs `memory` (§2.3) and custom errors over revert strings (§2.4) are all justified by gas.

Candidly: the most valuable safety features arrived **late and under pressure**. Checked arithmetic only became the default in 0.8.0 — December 2020, five years after the first release and four years after the canonical reentrancy incident.

### 1.4 Where Solidity Sits

```
   ┌─ SOURCE LANGUAGES ─────── ← THIS GUIDE's layer
   │  Solidity · Vyper · Huff · Fe · Yul · (Core Solidity)
   └────────┬──────────── compile (solc, vyper, huffc, …)
   ┌────────▼────────────
   │  IR + OPTIMISER      Yul → Yul-IR → SSA-CFG (experimental)
   └────────┬──────────── codegen
   ┌────────▼────────────
   │  EVM BYTECODE        creation code + runtime code + metadata tail
   └────────┬──────────── deploy
   ┌────────▼────────────
   │  EVM / THE CHAIN     ← ethereum_guide.md (gas, accounts, state)
   └─────────────────────
```

Three separations to hold firmly. **Language ≠ bytecode:** two compilers, or two settings of one compiler, produce different-but-equivalent bytecode from the same source — which is why verification (§3.4) needs the *settings*, not just the source. **Language ≠ protocol:** Solidity cannot change what gas costs or what an opcode does; it only chooses which opcodes to emit. **Language ≠ security model:** a contract's security is its language mechanics (§5), its design (sibling §8) and its operational controls; no compiler makes a bad design safe.

### 1.5 Provenance: Wood's 2014 Proposal

The accepted account: **Solidity was proposed in August 2014 by Gavin Wood**, and developed by the Ethereum project's Solidity team, **led by Christian Reitwiessner**.

**Status: ⚠ flagged.** Every source located this pass for the authorship and the August 2014 date is **secondary** — Wikipedia carries a `[non-primary source needed]` tag against exactly that sentence, and the Solidity docs and `Changelog.md` (both read) are silent on it. The established account is reported here, **not a first-hand verified fact** (§11).

Verifiable from the compiler repository's own changelog is the **release chronology**:

| Release | Date (`Changelog.md`) | What it introduced |
|---|---|---|
| 0.1.0 | **2015-07-10** | First tagged release (changelog entry empty) |
| 0.1.1 | 2015-08-04 | Strings as mapping keys; clone contracts |
| 0.1.3 | 2015-09-25 | The `throw` statement; linker stage for library addresses |
| 0.1.6 | 2015-10-16 | `.push()` for dynamic storage arrays; tuples; destructuring |
| 0.1.7 | 2015-11-17 | "Proof-of-concept transcompilation to why3 for formal verification of contracts" |

Two observations. First, **the first release tag predates Ethereum's Frontier mainnet launch of 30 July 2015** (cross-ref [ethereum_guide.md](ethereum_guide.md)) — language and chain were built in parallel. Second, **formal verification was on the roadmap in the language's fourth month** (the 0.1.7 why3 entry), making the SMT-checker story (§5.7) a long thread rather than a recent bolt-on. Early contributors beyond Reitwiessner are **not verified** (§11).

A second provenance thread: **the language was never gate-kept by a standards body.** There is no "Solidity EIP." The language evolves by the compiler team's own release process — changelog, blog, docs. Hence the weight of the *breaking-release discipline* in §4, and the split whereby the **EIP process** (sibling §4) governs *contract interfaces* while the compiler team governs *the language*.

### 1.6 Stewardship: The Argot Collective, Verified

The most important governance finding of this pass, and a **change** to the arrangement most existing material describes.

**Historical arrangement.** Solidity was for most of its life an **Ethereum Foundation** project — EF's in-house Solidity team, `ethereum/solidity` on GitHub, docs at `docs.soliditylang.org`, blog at `soliditylang.org/blog`. Any guide written before late 2024 saying "the Ethereum Foundation maintains Solidity" was accurate then.

**Present arrangement.** Stewardship has moved to the **Argot Collective** — verbatim from argot.org (read this pass): "a non-profit, independent research and development group sustaining Ethereum's core programming languages and tooling, most notably Solidity." Evidence this is live, not proposed:

- The canonical repository has moved to **`github.com/argotorg/solidity`**. Fetching the old `ethereum/solidity/blob/develop/CHANGELOG.md` redirected to `argotorg/solidity` and returned a 404 *because the file has been renamed* (`Changelog.md`, capital C, at repository root).
- The Solidity team's roadmap is published by Argot: the July 2026 update describes "Argot is a self-governed collective of engineers and researchers that maintains Solidity and the open-source compiler tooling the Ethereum ecosystem depends on."
- Argot's project roster (argot.org, read this pass): **Act** (formal specification language for EVM programs), **Ethdebug** (smart-contract debugging data format), **Fe** (statically typed language for the EVM), **Hevm** (symbolic execution engine), **Solidity** (the compiler and language), **Sourcify** (open-source, decentralised source-code verification).

**The transition, dated.** Argot was **introduced in October 2024**, simultaneously with an announced Ethereum Foundation funding commitment specified as **five years of operational runway at an ETH price of USD 2,592**, with a mandate to maintain critical infrastructure "under a neutral and independent umbrella." The **first three years were paid in full**; the **remaining two were finalised on 30 June 2026**, placed in a **2-of-3 multi-signature wallet** controlled by Argot, a newly established **Fail-Safe Committee** and the Ethereum Foundation, unlocking **1 July 2026** and **1 July 2027** (approximately **4,938 stETH**, 50 % released at each date). The Fail-Safe Committee comprises **five independent members plus one Argot member and one EF member** — a sounding board by default, an independent arbiter in a dispute.

**The honest reading, and the flag.** The transition is **not ambiguous** — compiler, repository, roadmap and funding structure have all moved — but it is **not a severance**: the EF remains a co-signer of the reserved funds and a member of the Fail-Safe Committee, and Argot itself notes that "the EF's treasury is finite," framing the arrangement as a **bridge to diversified funding**. Two consequences:

- **Governance risk is now a real category for a language dependency.** A compiler-supply-chain review must name Argot — and the EF, and the Fail-Safe Committee — not merely "Ethereum." **Flag: Argot's funding durability beyond the committed runway is unverified and is a live risk, not a settled one.**
- **The scope of "Solidity" is expanding.** The same organisation houses **Fe**, **Act**, **hevm**, **Sourcify**, **ethdebug** and a new language project, **Core Solidity** (`solcore`) — see §7.3.

Not resolved: the **exact date in October 2024** of Argot's announcement, and whether any **formal legal transfer** of repository ownership, trademarks or the `soliditylang.org` domain accompanied the practical move (§11).

### 1.7 Why a Bank Should Care: The Cymbal Bank Lens

The sibling guide's §1.3 argues why a bank should care about smart contracts as a *concept*. This guide answers the narrower question: why should technology-risk care about the **language and compiler** specifically?

| Concern | Why it lands on the language/compiler | Where |
|---|---|---|
| **Reproducible builds** | Bytecode verification can only succeed if the exact compiler version and settings are known; a floating `pragma` makes the artefact unreproducible | §3.4, §4.4 |
| **Upgrade safety** | Storage-layout collisions are a *source-level* property: the compiler's declared layout, not the proxy pattern | §5.2 (pattern owned by sibling §7) |
| **Supply-chain integrity** | The compiler is third-party software that has shipped *high-severity* codegen bugs; version pinning and advisory monitoring are mandatory | §4.4, §5.7 |
| **Audit determinism** | Auditors review source; the deployed artefact is bytecode produced with settings. If the two disagree, the audit is void | §3.4, §9.4 |
| **Tooling claims** | Every analyser in §6 makes accuracy claims; trusting them without measuring is relying on marketing | §6.3, §8.2 |
| **AI-generated code** | The compiler team reports an **order-of-magnitude increase in vulnerability reports driven by better LLMs** | §8.4 |

The through-line: **the compiler is part of the audit boundary.** A bank that treats `solc` as an invisible implementation detail cannot make a reproducible-build, verification or supply-chain attestation that holds up.

---

## 2. The Language Model

### 2.1 The Value Types

| Value type | Definition | Notes verified |
|---|---|---|
| `bool` | `true` / `false` | ABI-equivalent to `uint8` restricted to 0/1 |
| `uint<M>` / `int<M>` | Unsigned / two's-complement signed integer, `8 ≤ M ≤ 256`, `M % 8 == 0` | `uint`/`int` are synonyms for the 256-bit forms; selector uses `uint256`/`int256` |
| `address` | 20-byte (160-bit) address | ABI-equivalent to `uint160`; the ABI type is `address` |
| `address payable` | Address with `transfer`/`send` | Distinct from `address` since **0.5.0** |
| `bytes1` … `bytes32` | Fixed-size byte sequence | The `byte` alias was **disallowed in 0.8.0** |
| `enum` | User-defined ordered set, at most **256** members | >256 disallowed since **0.8.0**; ABI type `uint8` |
| Contract types | `MyContract` as a type | ABI type `address` |
| Function types | `function (uint) external returns (bool)` (`internal` variant too) | ABI encodes an external function as address + selector, i.e. `bytes24` |
| User-defined value types | `type MyUint is uint256;` | ABI maps to the underlying type |

Two operational points. **The ABI-visible type is not always the Solidity type** — an `enum` is `uint8` on the wire, a `contract` is an `address`, a `struct` is a `tuple`. **Width is semantic, not cosmetic** — `uint8 x = 255; x++;` reverts under checked arithmetic even though 256 fits an EVM word, because the EVM has no native sub-word arithmetic and the compiler adds the checks.

### 2.2 The Reference Types

| Reference type | Fixed or dynamic | Storage rules |
|---|---|---|
| `T[k]` | Fixed-size array | Index bounds enforced at runtime |
| `T[]` | Dynamic array | `.push()`, `.pop()`, `.length`; `.length` is **read-only since 0.6.0** |
| `struct` | Fixed shape | Member access; ABI type `tuple` |
| `mapping(K => V)` | Effectively infinite, sparse | **Storage only**; no `.length`, no iteration |

A `mapping` is a **hash-table-shaped declaration, not an allocated table**: `mapping(address => uint256) balances;` allocates nothing at declaration, and the value for key `k` lives at `keccak256(h(k) . p)` where `p` is the mapping's slot. Consequences that are language-level and unavoidable: **there is no way to enumerate a mapping**, so any registry, cap or aggregate must be maintained in a parallel array or counter; **mappings cannot be in `memory` or `calldata`** (and neither can a struct containing one — restrictions tightened in 0.7.0); and zeroing a mapping element does **not** free the slot.

### 2.3 The Data-Location Model

| Location | Lifetime | Where it lives | Mutability |
|---|---|---|---|
| `storage` | Permanent, chain state | The contract's persistent trie | Mutable unless `immutable`/`constant` |
| `memory` | One external call | The EVM's per-call linear memory | Mutable, cheap |
| `calldata` | One external call | The call's input data area | **Read-only** |

- **Explicit data locations have been mandatory since 0.5.0.** The release notes make it a breaking change: "Require explicit data location for all variables, including function parameters." Before that the compiler inferred locations — and got the wrong one often enough to become a headline break.
- **`calldata` was introduced as a keyword in 0.5.0** and is required for `external` reference-type parameters. Before it, external parameters were copied into memory by default — paying to copy data that would never change.
- **`storage → memory` and `memory → storage` copy.** Assigning a storage reference into `memory` copies; assigning `memory` into storage writes each element. Loop over the wrong location and the gas cost is effectively quadratic.
- **`memory → memory` and `storage → storage` are references, not copies.** Two `memory` variables of the same type can alias — the SMTChecker documents this explicitly as the "aliasing" problem (§5.7).
- **`calldata` reads avoid the copy entirely** — reading directly and writing only what is needed is the standard optimisation, expressed by the keyword rather than a compiler flag.
- **Storage reads are transcribed index-by-index.** `SLOAD`/`SSTORE` pricing is chain-level ([ethereum_guide.md](ethereum_guide.md)); the language's contribution is that a naive `for` over a storage array re-reads.

The review rule (§9.5): **every reference-type declaration states its location, and every location is the cheapest one that still expresses the intent.**

### 2.4 The Contract Model

- **State variables** — stored in `storage`. Qualifiers: `constant` (compile-time value), `immutable` (set once in the constructor, embedded in **code rather than storage**), and `transient` (cleared at transaction end, alongside the Cancun `TSTORE`/`TLOAD` opcodes — the exact introducing release is **not pinned this pass**, §11).
- **Functions** — visibility (`public`/`external`/`internal`/`private`), mutability (`pure`/`view`/`payable`/non-payable) and overloading. **Visibility must be stated explicitly — mandatory since 0.5.0**, which is why pre-0.5 code is dangerous to read: an omitted visibility defaulted to `public`.
- **Modifiers** — `modifier onlyOwner() { require(msg.sender == owner); _; }`. The `_` placeholder splices the function body; modifiers can run code after `_`.
- **Events** — `event Transfer(address indexed from, address indexed to, uint256 value);`, emitted with `emit`. Up to three `indexed` parameters become log topics (four with `anonymous`); the rest is ABI-encoded in the log data. Dynamic indexed parameters are stored as the **Keccak-256 hash** of their encoding, which the ABI spec warns is ambiguous for structs with more than one dynamic array.
- **Custom errors** — `error InsufficientBalance(uint256 available, uint256 required);` thrown with `revert InsufficientBalance(a, b);`. **Introduced in 0.8.4** (April 2021), verified from the Solidity blog's "Custom Errors in Solidity" post and the 0.8.4 release announcement. Their purpose is gas: a custom error compiles to a 4-byte selector plus arguments, where a revert *string* embeds the whole string in the deployed bytecode.
- **`receive()` and `fallback()`** — two separate entry points since **0.6.0**, splitting the old single unnamed fallback into a bare-ETH receiver and a generic catch-all.
- **The constructor** — the `constructor` keyword since **0.5.0**; `payable` since **0.4.0** (the release that made receiving ether explicit). **Since 0.7.0, visibility on a constructor is deprecated** and should not be written.
- **File-level declarations** — global `enum`s and `struct`s since 0.6.0; file-level constants since 0.7.4; free functions.
- **`try`/`catch` and `virtual`/`override` both arrived in 0.6.0.** Together they are what make a Solidity contract expressible as a *specification* rather than merely an implementation.

### 2.5 Inheritance and C3 Linearisation

Solidity supports multiple inheritance via `contract A is B, C { ... }`, resolved by **C3 linearisation** — the algorithm Python uses — producing a total, deterministic **method resolution order (MRO)**.

- **`virtual`** marks a function overridable; **`override`** marks the override. Both arrived in **0.6.0**, and `private` cannot be combined with `virtual`.
- **`super` does not mean "the parent class."** It means **the next contract in the linearised order**. In a diamond, `super.f()` walks the chain rather than jumping to the base — what makes cooperative multiple inheritance work, and what makes it misreadable. `super` is **not available in libraries** (a 0.7.6 note records turning that into an error).
- **`is`-list order is semantic, not stylistic.** Because the MRO derives from the order of base contracts, reordering the `is` list silently changes what `super` resolves to and which override wins.
- **State variable shadowing is disallowed since 0.6.0** — the language-level half of the storage-layout problem (§5.2). **Events with the same name and parameter types in a hierarchy were disallowed in 0.7.0**, as was overriding a `pure` function with a public state variable.
- **Super constructor calls** can use member notation (`M.C(123)`) since **0.8.0**; base constructor arguments may not be specified twice in one hierarchy (0.5.0).

To know what a function *does*, you must know the linearised order, not the file order. Because the C3 order derives from the `is` list, **the `is` list is part of the contract's public semantics** and should be treated as immutable in a review.

### 2.6 Interfaces and Libraries

**Interfaces** declare an external contract's shape with no implementation: all functions `external` (enforced since 0.5.0), no state variables, no constructors; `enum`s and `struct`s allowed since 0.5.0. The critical property: **an interface is a compile-time assertion about a deployed address.** Casting with `IMyContract(addr)` performs **no runtime check** that the code at `addr` implements that interface. The resulting mis-call is caught only by `try`/`catch` (0.6.0) or an explicit `supportsInterface`-style probe — a recurring integration error and a checklist item.

**Libraries** are stateless helpers with `internal` functions (inlined, no deployment or linking) and `public`/`external` functions (deployed separately and linked by address). Two consequences:

- **Library function selectors differ from contract selectors** — the ABI spec notes that "the ABI specification for libraries is slightly different."
- **Library linking is a build artefact, not a runtime property.** Bytecode contains a placeholder of the form `__$53aea86b7d70b31448b230b20ae141a537$__` (format changed in 0.5.0). The docs warn explicitly that **manual post-hoc linking is discouraged because it does not update the contract metadata**, producing a binary that will not match at verification time (§3.4). Pass libraries to `solc` via `--libraries` or the standard-JSON `libraries` key.
- **`using A for B;` is scoped.** Since **0.7.0** it "only affects the contract it is mentioned in and not all derived contracts" — a change that silently altered inheritance behaviour.

### 2.7 The ABI as the Language's External Contract

The **Application Binary Interface** is the ecosystem-wide encoding convention, specified in the Solidity documentation.

**Where the specification lives — verified.**

1. **The canonical, complete specification is in the Solidity docs**: `docs.soliditylang.org/en/latest/abi-spec.html`, titled *Contract ABI Specification* (read this pass; it renders "Solidity 0.8.38-develop documentation", confirming both the spec's home and the current development version). It defines the selector, argument encoding, the type list, the Solidity→ABI type mapping, the formal `enc` definition, indexed-event encoding and the packed-encoding warning.
2. **A shorter "contract ABI" description is carried by the Ethereum execution-layer docs**, on the JSON-RPC page (`ethereum.org/en/developers/docs/apis/json-rpc/`, read this pass) within the worked `eth_sendTransaction` example. That page points to the **execution-APIs repository** (`github.com/ethereum/execution-apis`) as the authoritative API specification, and its encoding rules match the Solidity spec.

**The answer: the Solidity documentation, with the execution-layer docs carrying a summary and the execution-APIs repo governing the transport.** They are consistent; the Solidity page is the fuller one.

Load-bearing details:

- **Selector derivation.** "The first four bytes of the call data … is the first (left, high-order in big-endian) four bytes of the Keccak-256 hash of the signature of the function." The signature is "the canonical expression of the basic prototype without data location specifier" — name, parenthesis, comma-separated types, **no spaces**. Data-location keywords are excluded, and **the return type is not part of the selector** because "in Solidity's function overloading return types are not considered," keeping call resolution context-independent. So `transfer(address,uint256)` always hashes to the same 4 bytes regardless of `memory` annotations or return values.
- **Static/dynamic split.** `bytes`, `string`, `T[]`, `T[k]` where `T` is dynamic, and tuples containing a dynamic member are **dynamic**; everything else is **static**. Static types are encoded in place; dynamic types are encoded after the head block, with the head holding a byte offset.
- **`abi.encode` vs `abi.encodePacked`.** The spec warns explicitly that with both arguments dynamic "it is easy to craft collisions in the hash value" — `abi.encodePacked("a","bc") == abi.encodePacked("ab","c")` — concluding "Unless there is a compelling reason, `abi.encode` should be preferred." Because this sits in the *specification*, it is a language-level fact rather than folklore.
- **The ABI JSON description** carries both inputs and outputs (unlike the selector) and is emitted as the `abi` artefact.

Two consequences. **The ABI is the language's fragility surface** — adding a function or changing a parameter type changes the selector, so off-chain integrations keyed on the old selector silently fail; ABI artefacts must be versioned alongside deployments. And **the ABI is what makes cross-language interoperability work at all**: the same encoding lets a Vyper contract call a Solidity contract, or a TypeScript client call either (§7).

---

## 3. The Compiler (solc)

### 3.1 The Pipeline

`solc` is the reference compiler, written in C++ — and **`solc-js` is the emscripten build of the same compiler**, not a reimplementation. The docs are explicit that the command-line section "does not apply to `solcjs`, not even if it is used in commandline mode."

```
  .sol sources
    │ 1. scanner/lexer → tokens
    │ 2. parser → AST (--ast-compact-json)
    │ 3. name resolution + type checking → typed AST; errors
    │ 4. semantic analysis: control-flow (unassigned returns), pure/view
    │      enforcement, override/linearisation checks, Yul analysis
    │ 5. codegen — one of: (a) legacy evmasm (historic default);
    │      (b) IR: Solidity → Yul → Yul-IR → optimise → EVM (--via-ir);
    │      (c) experimental SSA-CFG (--via-ssa-cfg, 0.8.35+)
    │ 6. optimiser (Yul optimiser on the IR path; evmasm optimiser on legacy)
    │ 7. assembly + linking → creation bytecode, runtime bytecode
    │ 8. metadata → CBOR blob appended to the artefact
  artefacts: bin, bin-runtime, abi, ast, asm, storageLayout, metadata, …
```

- **The compiler is deliberately strict.** Explicit conversions changing more than one of sign, width or kind are rejected (0.8.0): `uint16(int16(int8(-1)))` yields `0xffff` while `uint16(uint8(int8(-1)))` yields `0xff`, and the compiler refuses to choose.
- **Diagnostics are graded.** Documented categories include `SyntaxError`, `DeclarationError`, `TypeError`, `UnimplementedFeatureError`, `InternalCompilerError`, `YulException`, `FatalError`, plus `Warning` and `Info`.
- **An `InternalCompilerError` is a security-relevant event**, not a nuisance: the compiler hit a state it believed unreachable, and bugs of that flavour have shipped as *high-severity* codegen defects (§5.7).

### 3.2 Legacy Codegen versus via-IR

| | Legacy (`evmasm`) | via-IR |
|---|---|---|
| Path | AST → **direct** EVM assembly | AST → **Yul** → Yul-IR → optimise → EVM |
| Status | Historic default | Opt-in; `--experimental-via-ir` arrived in **0.7.5** |
| Optimiser | evmasm optimiser (peephole/block) | **Yul optimiser** over the IR (more powerful, much slower) |
| Stack handling | Works around the 16-slot limit opportunistically | IR optimiser + spilling; classic failure is **"stack too deep"** |
| Risk profile | Long service history | Newer; the **transient-storage clearing-helper collision** affected `--via-ir` only (§5.7) |

0.6.0 recorded a milestone: "Enable Yul optimizer as part of standard optimization." The `settings.viaIR` key was added in **0.7.5** alongside `--experimental-via-ir`.

**Current direction — verified from Argot's July 2026 roadmap.** H1 2026 went into a third path, an **SSA-CFG backend**: "a from-scratch single static assignment (SSA) based Yul codegen path … wired into the CLI and Standard JSON as `--via-ssa-cfg` / `settings.viaSSACFG`, shipped as an experimental opt-in in **0.8.35**." The motivation is to fix the two chronic via-IR complaints — **"stack-too-deep" errors** and **long compile times**: stack-to-memory spilling moves into the stack-scheduling phase "opposed to viaIR's memory spilling in the optimizer phase," and the new optimiser pipeline "aims to fix viaIR's long compile times." H2 2026 is "stabilizing the SSA-CFG pipeline and moving it out of `--experimental`."

Two further codegen facts:

- **The experimental EOF backend was deleted in 0.8.36** ("Remove support for the experimental EOF (EVM Object Format) backend"), confirmed by the roadmap as "a strategic removal … across the entire codebase."
- **The `--experimental` gating framework arrived in 0.8.35.** A defined feature set (`ast-import`, `lsp`, `evmasm-import`, `generic-solidity`, `ir-ast`, `eof`, `evm`, `ethdebug`, `ssa-cfg`) is reachable only behind `--experimental` / `settings.experimental`, and **the mode's use is recorded in the metadata** — which matters for verification (§3.4). Also in 0.8.37, the LSP mode and the `pragma experimental solidity` prototype were **removed**.

### 3.3 The Optimiser: What `runs` Trades

The optimiser is enabled with `--optimize` / `settings.optimizer.enabled`. The central parameter is **`runs`**, and the docs give the clearest statement of the trade:

> "By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime (more specifically, it assumes each opcode is executed around 200 times). If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to `--optimize-runs=1`. If you expect many transactions and do not care for higher deployment cost and output size, set `--optimize-runs` to a high number."

The docs also state the current effects, with the caveat that "this might change in the future": **the size of the binary search in the function dispatch routine**, and **the way constants like large numbers or strings are stored**.

| `runs` | Lean toward | Consequence |
|---|---|---|
| `1` | Minimum **deployment** cost | Larger runtime; higher per-call cost |
| `200` (default) | Balanced | The compiler's default assumption; safe when the call count is unknown |
| Very high | Minimum **runtime** cost | Largest deployment cost; rational only for hot contracts such as a token or AMM core |

Three cautions. **The optimiser is not a security tool** — the high-severity IR bug of §5.7 was a codegen bug, invisible to a source review. **Optimiser settings must be part of the verification record** (they change bytecode). And **`--optimize-runs` now accepts a very large interval** — as of 0.8.36 it "also accepts values from the interval `[INT64_MAX, UINT64_MAX]`," accompanied by a standard-JSON serialisation fix in the same release.

### 3.4 The Metadata Hash and the Verification Link

Every compilation appends a **metadata blob** to the deployed bytecode — the mechanism behind source verification, and the cause of a whole class of "bytecode doesn't match the explorer" tickets.

- A **CBOR-encoded** structure (with a JSON equivalent) containing the compiler version, optimiser settings, EVM version, the sources and their **IPFS (or Swarm) hashes**, the ABI, NatSpec docs, and — since 0.8.35 — whether experimental mode was used. The changelog records the duality: "Metadata: Store the state of the experimental mode in JSON and CBOR metadata. In CBOR this broadens the meaning of the existing `experimental` field."
- Appended at the **end of the runtime bytecode**, the final two bytes encoding its length — how tools find it.
- **The hash method is selectable**: 0.6.0 added "option to disable or choose hash method between IPFS and Swarm for the bytecode metadata," in both CLI and standard JSON. Since sources are content-hashed, **a one-character source change changes the metadata, the bytecode, and (under deterministic deployment) the address.**

> **Verification is a claim that "this source, compiled with these settings by this compiler version, produces this bytecode."** Source alone is insufficient: compiler version, `runs`, `viaIR`/SSA-CFG selection, `--evm-version`, `--metadata-hash` and `--revert-strings` all change the output.

Consequences: **source-verify on deployment and archive the inputs** — the artefact to keep is the *complete standard-JSON input* (§3.6), not merely the `.sol` files. **The metadata hash is a build identifier**, so "we changed nothing but the comment" still changes the artefact under CREATE2. And **the metadata tail is a fingerprinting surface**, advertising compiler version and settings to anyone reading the chain; `--metadata-hash none` removes it at the cost of verifiability — a policy decision, not a technical one.

### 3.5 The Flags That Matter

| Flag | Standard-JSON key | What it does |
|---|---|---|
| `--optimize` | `settings.optimizer.enabled` | Turns on the optimiser |
| `--optimize-runs N` | `settings.optimizer.runs` | Deployment-vs-runtime trade; default **200** |
| `--via-ir` | `settings.viaIR` | Compile through the Yul IR pipeline (§3.2) |
| `--via-ssa-cfg` | `settings.viaSSACFG` | Experimental SSA-CFG backend (0.8.35+; requires `--experimental`) |
| `--evm-version <V>` | `settings.evmVersion` | Selects the target fork (§3.7) |
| `--metadata-hash <ipfs\|swarm\|none>` | `settings.metadata` | Chooses or removes the metadata hash (§3.4) |
| `--revert-strings <default\|strip\|debug\|verboseDebugString>` | `settings.debug.revertStrings` | Controls embedded revert strings; added **0.6.0** |
| `--standard-json` | — | The machine interface (§3.6) |
| `--libraries` | `settings.libraries` | Links libraries at compile time (§2.6) |
| `--bin`, `--bin-runtime`, `--abi`, `--ast-compact-json`, `--asm` | output selection | Artefact selection; legacy `--ast-json` **removed in 0.8.0** |
| `--base-path`, `--include-path`, `--allow-paths` | paths/remappings | Import resolution and filesystem sandbox |
| `--experimental` | `settings.experimental` | Enables the gated feature set (0.8.35+); never in production |
| `--model-checker-engine <all\|bmc\|chc\|none>` | `settings.modelChecker.engine` | SMTChecker engine selection; **default `none`** (§5.7) |
| `--model-checker-targets`, `--model-checker-timeout` | `settings.modelChecker` | Verification targets and per-query timeout |

Two notes. **`--revert-strings` interacts with custom errors** — a 0.8.35 bugfix was "Preserve custom error argument of `require` when stripping of revert strings is selected via `--revert-strings strip`," so `strip` and custom errors have not always been independent: test the exact settings combination you deploy. And **the filesystem restrictions are a real sandbox** — the compiler rejects by default any directory other than the source paths and remapping targets, with `--allow-paths`/`--base-path` as the escapes.

### 3.6 The Standard-JSON Interface

For anything automated, **standard JSON is the recommended interface** — the docs say so directly: "This is the recommended interface for more complex and especially automated uses."

**It is a total-function contract.** The process "will always terminate in a 'success' state and report any errors via the JSON output." A failed compilation is a JSON document with an `errors` array, not a non-zero exit — so a CI script checking only the exit code is not checking the compilation.

```json
{ "language": "Solidity",
  "sources": { "Vault.sol": { "content": "// SPDX-License-Identifier: MIT\npragma solidity 0.8.37; ..." } },
  "settings": {
    "optimizer": { "enabled": true, "runs": 200 },
    "evmVersion": "osaka", "viaIR": false,
    "metadata": { "bytecodeHash": "ipfs" },
    "outputSelection": { "*": { "*": ["abi", "evm.bytecode", "evm.deployedBytecode",
                                     "metadata", "storageLayout"] } } } }
```

Interface hygiene, made concrete by the changelog:

- **Artefacts are sub-objects, not strings.** Since **0.8.0**, "JSON fields `abi`, `devdoc`, `userdoc` and `storage-layout` are now sub-objects rather than strings" — pre-0.8 tooling silently reads the wrong shape.
- **`legacyAST` was removed in 0.8.0**; `--ast-compact-json` is the supported form and the older `--ast-json` flag was removed in the same release.
- **Output selection actually filters** — "Standard-Json: Properly filter the requested output artifacts" (0.7.6).
- **`storageLayout` is a first-class artefact** since 0.5.13 ("Output the storage layout of a contract when artifact `storageLayout` is requested") — the machine-readable input to the collision check in §5.2 and §9.3.

The reason to prefer standard JSON over shell invocations is not tidiness: **the standard-JSON input is the reproducibility record.**

### 3.7 EVM-Version Targeting

`--evm-version` / `settings.evmVersion` selects a fork target. It changes which opcodes the compiler will emit and how it models gas.

| Target | Compiler-relevant change (per the docs) |
|---|---|
| `homestead` | Oldest; **support deprecated** |
| `tangerineWhistle` | Gas for other-account access raised; all gas sent by default for external calls; **deprecated** |
| `spuriousDragon` | `exp` gas raised; **deprecated** |
| `byzantium` | `returndatacopy`/`returndatasize`/`staticcall` in assembly; `staticcall` for non-library view/pure calls; `revert` introduced; **deprecated** |
| `constantinople` | `create2`, `extcodehash`, `shl`, `shr`, `sar`; cheaper shifts |
| `petersburg` | Behaves as `constantinople` |
| `istanbul` | `chainid`, `selfbalance` in assembly |
| `berlin` | Cold-gas costs assumed for `SLOAD`, `*CALL`, `BALANCE`, `EXT*`, `SELFDESTRUCT` |
| `london` | `block.basefee` (EIP-3198 / EIP-1559) |
| `paris`, `shanghai`, `cancun`, `prague`, `osaka` | Later forks; `cancun` brings transient storage; `osaka` is the **current default** as of 0.8.31 |

The rules that matter:

- **The default moves forward with the compiler.** 0.5.14 set it to Istanbul; 0.8.30 to **`prague`**; **0.8.31 to `osaka`**. A pinned compiler pins a default target — another reason a floating `pragma` plus floating compiler is an unreproducible build.
- **Support is being withdrawn on a schedule.** As of **0.8.37**: "Deprecate support for `constantinople`, `petersburg`, `istanbul` and `berlin` EVM versions."
- **Experimental targets exist and are gated.** 0.8.36 added "Support for the EVM version 'Amsterdam'"; 0.8.35 introduced the experimental `@future`. In the 0.8.35 gating table the "Non-mainnet EVMs" feature (`evm`) *affects bytecode* — which is why `--evm-version` is a first-class member of the reproducibility record.
- **The docs' own warning is the review rule**: "Compiling for the wrong EVM version can result in wrong, strange and failing behavior." A permissioned deployment on a chain that has not activated a fork, compiled for a target that assumes it, is a compatibility failure no source review will catch.
- **New opcode families arrive through the target.** 0.8.37 added `block.slotnum` and the Yul builtin `slotnum()` for `SLOTNUM`, "available since the Amsterdam EVM version (EIP-7843)"; 0.8.31 introduced Yul `clz(x)`.

---

## 4. The Version History and the Breaking Changes

### 4.1 The Breaking-Change Boundaries

A `.0` minor bump is a break, and the changelog names what broke. Verified from `Changelog.md` (argotorg/solidity) and the release announcements.

| Boundary | Date | What it broke |
|---|---|---|
| **0.1.0** | **2015-07-10** | First tagged release (changelog entry empty) |
| **0.4.0** | **2016-09-08** | "Deliberately breaks backwards compatibility mostly to enforce some safety features. The most important change is that you have to explicitly specify if functions can receive ether via the `payable` keyword." |
| **0.5.0** | **2018-11-13** | **Function visibility mandatory** ("Strictly require visibility specifier for functions"); **explicit data location mandatory**; `var`, `throw`, the `sha3`/`suicide` aliases and `callcode` **disallowed**; `constructor` and `calldata` keywords introduced; `address payable` distinguished; `emit` becomes a keyword; C99 scoping enforced; loose-assembly syntax disallowed entirely; the `std` directory removed; `keccak256(a,b,c)` → `keccak256(abi.encodePacked(a,b,c))`; `.call()` → `.call("")` |
| **0.6.0** | **2019-12-17** | Fallback **split** into `fallback()` and `receive()`; **`virtual`/`override` introduced**; **`try`/`catch` introduced**; **state variable shadowing disallowed**; abstract contracts must be marked `abstract`; `length` read-only for storage arrays; explicit conversions from external function types to `address` disallowed; **Yul optimizer enabled as part of standard optimization**; `--revert-strings` added |
| **0.7.0** | **2020-07-28** | `now` **removed** (use `block.timestamp`); `finney`/`szabo` **removed**; `using A for B` **scoped to the declaring contract only**; constructor visibility deprecated; shifts by signed types disallowed; assignments to state variables containing nested mappings disallowed |
| **0.8.0** | **2020-12-16** | **All arithmetic checked by default**; `unchecked { }` introduced; **ABI coder v2 by default**; failing assertions revert with `Panic(uint256)` instead of the invalid opcode; explicit conversions restricted to a single change of sign/width/kind; the `byte` type **disallowed**; enums capped at 256 members; `address(...)` returns non-payable; `tx.origin`/`msg.sender` retyped to `address` |
| **0.8.37** | **2026-09-10** | Latest tagged release at read time; 0.8.38 *unreleased*. Deprecations rather than breaks: EVM support for `constantinople`/`petersburg`/`istanbul`/`berlin` deprecated; the LSP mode and `pragma experimental solidity` removed |

**The pattern is the important artefact:** each boundary moved the same direction — **making previously implicit behaviour explicit**. Visibility, data location, payability, mutability, `virtual`/`override` and arithmetic checking were all once implicit. That is a language maturing toward reviewability, and it is why a bank's checklist is largely a list of these decisions (§9.5).

### 4.2 The 0.8 Arithmetic Change and Its Consequence

The most consequential change in the language's history for anyone reading code written before December 2020. Verified from the 0.8.0 release announcement (16 December 2020), three sub-features shipped together as "Checked Arithmetic":

1. **Arithmetic is checked by default.** "By default, all arithmetic operations will perform overflow and underflow checks … In case of an underflow or overflow, a `Panic(0x11)` error will be thrown and the call will revert." The checks follow the **declared type**, not the word: `uint8 x = 255; x++;` panics.
2. **Internal errors revert rather than executing the invalid opcode.** "Previously, internal errors like division by zero, failing assertions, array access out of bounds, etc., would result in an invalid opcode being executed." Because the invalid opcode *consumes all remaining gas*, this was "very expensive."
3. **`unchecked { }` restores wrapping arithmetic.**

**The error-data contract — a durable interface.** Regular errors use no data or `Error(string)` (selector **`0x08c379a0`**); critical errors use `Panic(uint256)` (selector **`0x4e487b71`**). Documented panic codes:

| Code | Condition |
|---|---|
| `0x01` | `assert` with a false argument |
| `0x11` | Arithmetic underflow or overflow **outside** an `unchecked` block |
| `0x12` | Division or modulo by zero |
| `0x21` | Conversion of an out-of-range or negative value into an `enum` |
| `0x22` | Access to an incorrectly encoded storage byte array |
| `0x31` | `.pop()` on an empty array |
| `0x32` | Out-of-bounds or negative array / `bytesN` / slice index |
| `0x41` | Excessive memory allocation or an oversized array |
| `0x51` | Calling a zero-initialised internal function pointer |

The announcement notes the list "can be extended in the future" and that `try ... catch Panic(uint _code)` was not yet usable — "planned for the near future."

**Consequences beyond the feature.** **Pre-0.8 code has different arithmetic semantics from post-0.8 code, silently**: source written against `^0.7.0` and compiled then wraps on overflow; the *same source* recompiled under 0.8.x reverts. An inventory must therefore record the *compiler version*, because the source does not encode which semantics apply. **`SafeMath` became largely redundant**, so its presence in a 0.8.x contract signals mechanical migration rather than re-review. **The announcement is explicit about the remaining gap**: "We did not introduce checks for explicit type conversions from larger to smaller types, because we think that such checks might be unexpected" — so `uint128(bigValue)` still truncates silently (§5.5). And **`unchecked` is narrower than it looks**: "If there is a function call inside the `unchecked` block, the function will not inherit the setting" — lexical, not transitive — and in a modifier `_;` cannot appear inside `unchecked`, and `unchecked` cannot replace a block.

### 4.3 The SemVer Policy and the Long-Deferred 1.0

**The policy.** Versions use `0.x.y` where **`y` is a non-breaking fix and `x` is a breaking change**. Because the major number is still `0`, the standard SemVer reading applies: *every* `0.x` minor bump may break. There is no promise that 0.9.0 will not break 0.8.x — and the changelog carries the evidence: as of 0.8.31 the compiler emits **deprecation warnings for `send`/`transfer` on `address`, ABI coder v1, virtual modifiers, comparisons between contract-type variables, and the `memory-safe-assembly` comment**. Argot's roadmap labels these collectively **"Deprecation warnings ahead of 0.9.0"** and adds warnings about "identifiers selected for future promotion to Solidity or Yul keywords (`at`, `error`, `layout`, `transient`, `this`, and a Yul reserved-identifier set)." **0.9.0's direction is visible even though it has not shipped.**

**The 1.0 status — verified, and it is not a date.** 1.0 has **not** been released. The latest tagged release read this pass is **0.8.37 (2026-09-10)** and the docs render as **"Solidity 0.8.38-develop documentation"**; there is no `1.0.0` release. A GitHub issue titled "Release version 1.0.0" (**#11690** on `argotorg/solidity`) proposes releasing "the same exact code as the latest release. Just renamed as version 1.0.0" — a *proposal in an issue tracker*, not a roadmap commitment.

**Why 1.0 has been deferred — a flag rather than a claim.** The team has continued to find breaking changes worth making (0.5, 0.6, 0.7, 0.8, and the deprecation set pointing at 0.9), while reaching 1.0 is conventionally a compatibility promise. **This guide found no primary statement of a 1.0 plan or date** (§11). Do not plan around a 1.0.

### 4.4 Version-Pinning Discipline

Pinning is a control, not a preference — §3.4 establishes that verification needs the compiler version and settings, and §4.2 that the same source means different things under 0.7 and 0.8.

| Practice | Why |
|---|---|
| **Pin an exact compiler version**, `pragma solidity 0.8.37;` — not `^0.8.0` or `>=0.8.0` | `^0.8.0` admits every future 0.8.x, whose default EVM target, deprecations and codegen can differ (§3.7). The pragma is a compatibility *claim*, not a build instruction |
| **Pin the compiler in the build separately** — `solc`/`solc-js` version or container digest | The pragma constrains; the toolchain decides. Reproducibility needs the toolchain pinned *and* recorded |
| **Archive the full standard-JSON input** | The only artefact containing sources **and** settings (§3.6) |
| **Record `runs`, `viaIR`, `evmVersion`, `metadata.bytecodeHash`, `revertStrings`** | Each changes bytecode and can break verification (§3.5) |
| **Monitor the supported-versions policy** | `SECURITY.md`: **"As a general rule, only the latest release gets security updates."** Exceptions only when the current breaking release is under about three months old. A bank pinned to an older 0.8.x is, by that policy, unpatched |
| **Watch the advisories, not the changelog alone** | Compiler bugs are announced as security blog posts and recorded in `docs/bugs.html` / `docs/bugs_by_version.json` (§5.7), not in the release notes' feature list |

---

## 5. The Language-Level Security Mechanics

This section covers the **mechanisms** — what the language *permits*, *prevents* and *implies*. It deliberately does not re-explain the vulnerability classes: the attack taxonomy, the reentrancy class, MEV and signature attacks are owned by [smart_contracts_guide.md](smart_contracts_guide.md) §8, and the incident record by its §9.

### 5.1 Default Visibilities and the Language's Own Safety Features

The most important historical fact in Solidity's security posture is that **the dangerous default was once the default**. Before 0.5.0 a function's visibility could be omitted and the compiler chose, with `public` a common outcome. Since **0.5.0** the language requires it: *"Strictly require visibility specifier for functions."*

The safety features the language itself provides, all enforced by the compiler:

- **State-mutability enforcement.** `view`/`pure` are checked by a dedicated "View Pure Checker"; since 0.5.0 state mutability is *strictly enforced* and `msg.value` is disallowed in a non-payable function even indirectly via a modifier. Since **0.5.0**, `view`/`pure` external calls compile to `STATICCALL`, preventing state modification *at the EVM level*.
- **Type-system strictness.** Conversions changing more than one of sign, width or kind are rejected; enums are capped at 256 members; literal-to-enum conversions must fit.
- **Declaration hygiene.** State variable shadowing (0.6.0), empty structs (0.5.0), uninitialised storage pointers (0.5.0) and unassigned named return values (control-flow warning since 0.7.4) are compiler-checked.
- **Alignment with secure defaults over time.** The deprecation set ahead of 0.9.0 (§4.3) continues the story: `send`/`transfer` because their fixed-gas stipend is a footgun; ABI coder v1 because it performs fewer input checks; contract-type comparisons because they invite address confusion.

The review heuristic: **every omission of a safety keyword is a finding** — an omitted visibility, data location, `override`, or `immutable`/`constant` is a place where the language would have helped and was not asked to.

### 5.2 Storage Layout and the Layout-Collision Implication

- **Declaration order determines slot order.** Variables are assigned slots from **slot 0** in declaration order.
- **Packing.** Consecutive value types share a slot if their combined size is at most 32 bytes. Two `uint128`s share; a `uint256` never shares.
- **Structs and fixed arrays.** A struct occupies whole slots with members packed within them; `T[k]` occupies `ceil(k × sizeof(T) / 32)` consecutive slots from its own slot.
- **Dynamic arrays and `bytes`/`string`.** The declaring slot stores the **length** (`bytes`/`string` under 32 bytes store the data in the slot with `length × 2` in the lowest byte; longer values store `length × 2 + 1` and the data at `keccak256(slot) + i`). Dynamic array elements live at `keccak256(p) + i`.
- **Mappings.** The mapping's slot is **empty**; the value for key `k` lives at `keccak256(h(k) . p)`.
- **Inheritance determines the prefix.** A contract stores base-contract variables **first**, in linearisation order (§2.5), then its own — so changing the `is` list order is *also* a storage-layout change.
- **`constant` and `immutable` consume no storage** — they live in the code.
- **The layout is machine-readable.** The `storageLayout` artefact (since 0.5.13) reports slots and offsets — the input to an automated collision check.
- **Layouts are becoming configurable.** 0.8.29 added "Allow relocating contract storage to an arbitrary location"; **0.8.35** added "a builtin that computes the base slot of a storage namespace using the `erc7201` formula from ERC-7201." ERC-7201 is a *standard emitted by the compiler* — a new coupling between language and standards layer (sibling §4).

**The layout-collision rule for upgradeable contracts** (the proxy *patterns* are sibling §7):

> In an upgradeable contract, the storage layout is append-only. A new version may **add** variables at the end. It must not **reorder**, **remove**, **change the type of**, **change the packing of**, or **re-parent** existing state variables. Any of those silently reinterprets existing on-chain data — no error, no revert, only wrong values read as if they were correct.

This is a *source-level* property, which is why it belongs here: the compiler's layout is a function of the declarations, and the reviewer's job is to diff two `storageLayout` artefacts and confirm the new one is the old one's prefix. Two corollaries: **inheritance changes are layout changes**, and **a storage gap is a placeholder, not a guarantee** — reserved-slot patterns rely on hand-maintained accounting the language does not enforce.

### 5.3 delegatecall and the Trust Model It Creates

`delegatecall` executes *another contract's code* against *the caller's own storage, address and balance*. That is its purpose and its risk.

- **`address(this)` inside the delegated code is the caller**, and `msg.sender`/`msg.value` are preserved — the delegated code's notion of "who am I" and "who is calling" both refer to the caller's world.
- **`immutable` and `constant` live in the *code*, not storage** — so a `delegatecall` target reading an `immutable` reads *its own* value, not the caller's. Context assumptions that hold in the caller do not automatically hold in the callee.
- **Public library functions are invoked by `delegatecall`** — which is why libraries have their own selector rules (§2.6) and why linking a library is a trust decision about code that will run in the caller's storage context.
- **The trust model: whoever controls the delegated-to address controls the caller's storage.** There is no per-function sandbox, no capability check, and no runtime assertion that the target implements the expected interface (§2.6). The language hands the developer an unchecked primitive and a compiler warning.
- **The discipline is procedural, not language-enforced:** the target must be immutable or governed by a trust-minimised upgrade path; caller and callee storage layouts must be *deliberately* aligned; and the call must be a purpose-built, audited entry point rather than a generic dispatch.

### 5.4 tx.origin and the Authorisation Anti-Pattern

`tx.origin` is the **externally-owned account that originated the transaction**; `msg.sender` is the **immediate caller**, which may be a contract. Using `tx.origin` for authorisation is a language-level anti-pattern because the value survives every intermediate call: if a user is induced to call an attacker's contract, that contract's call to the victim arrives with `tx.origin == the user`, and the check passes.

`tx.origin` is a *global of type `address`* (retyped from `address payable` in 0.8.0, alongside `msg.sender`), **read-only**, and **not** the same as `msg.sender`. The language provides no check against misuse — `tx.origin` remains available because legitimate uses exist, notably classifying an EOA versus a contract. The review rule: **`tx.origin` may be used for classification, never for authorisation.** The taxonomy entry is SWC-115, owned by sibling §8.2.

### 5.5 The unchecked/checked Boundary

- **Default: checked.** Overflow/underflow reverts with `Panic(0x11)` (§4.2), following the **declared type** — so narrow types revert earlier than the word size suggests.
- **`unchecked { }`: wrapping.** Used for genuinely modular arithmetic, for loop counters with a provable bound, and for gas-sensitive paths. The justification must be *stated*, because a safety property is being traded for gas.
- **The boundary does not propagate.** A function called from inside `unchecked` is checked unless it declares its own block. `_;` cannot be placed inside `unchecked` in a modifier, and `unchecked` cannot replace a block.
- **Explicit downcasts are outside the boundary entirely.** `uint128(x)` on a larger value truncates **silently**, in checked or unchecked context alike — the 0.8.0 announcement's own statement that narrower conversions were deliberately left unchecked. This is the surviving arithmetic hazard, hence the checklist item "every narrowing conversion is preceded by an explicit bounds check."
- **Inline assembly is always raw.** Yul operates on 256-bit words with no compiler-inserted checks, so arithmetic written in assembly re-opts out of the entire 0.8.0 safety net. Since **0.6.0** only *strict* inline assembly is permitted, and since **0.8.0** reserved identifiers (including EVM instruction names) are disallowed in Yul — both reduce, but do not remove, the "assembly is where the checks end" property.

### 5.6 Reentrancy Guards at the Language Level

The reentrancy *attack class* is sibling §8.1. Only the **language idioms** are covered here.

- **Interaction ordering** — perform state writes *before* external calls, so a re-entrant call observes consistent state. Permitted by the language; not enforced by it.
- **The mutex modifier** — a `bool` (or `uint256`) state variable set on entry and cleared on exit, checked at the top of every external entry point:

```solidity
contract Guarded {
    bool private _locked;

    modifier noReentry() {
        require(!_locked, "reentrant call");
        _locked = true;
        _;
        _locked = false;
    }

    function withdraw(uint256 amount) external noReentry {
        // state writes first, external call last
    }
}
```

  The guard must be set **before** `_` and cleared **after**; it must be applied to *every* externally reachable state-changing function; and a revert anywhere in the body rolls back the lock automatically, so no manual `try`/`finally` is needed.
- **The transient-storage variant.** Since the Cancun-era `TSTORE`/`TLOAD` opcodes a guard can live in **transient storage**, persisting for the transaction only and cheaper to write. The language exposes this through the `transient` qualifier. The correctness caveat is the one that produced the §5.7 bug: **`delete` on transient storage and clearing of persistent storage must not be conflated.**
- **What the language does not offer.** No built-in reentrancy guard exists — it is a convention, or in practice an imported library such as OpenZeppelin's `ReentrancyGuard`. And **a guard is not a substitute for correct ordering**: it defends only the functions it decorates, and read-only reentrancy is not blocked by a mutex on state-changing functions.

### 5.7 Compiler-Emitted Safety Nets

The compiler ships **one** built-in formal-verification facility: the **SMTChecker** — precise about what it is, because the name overstates it.

**What it does (documentation, v0.8.35 page, read this pass):**

> "The SMTChecker module automatically tries to prove that the code satisfies the specification given by `require` and `assert` statements. That is, it considers `require` statements as assumptions and tries to prove that the conditions inside `assert` statements are always true."

Documented targets, checked at compile time: arithmetic underflow and overflow (**"not … by default for Solidity >=0.8.7"**, since checked arithmetic already reverts); division by zero; trivial conditions and unreachable code; popping an empty array; out-of-bounds index access; insufficient funds for a transfer.

**How it is enabled.** "To enable the SMTChecker, you must select which engine should run, where **the default is no engine**." The engines are **BMC** and **CHC**; the CLI selector is `--model-checker-engine <all|bmc|chc|none>`, with `settings.modelChecker.engine` in standard JSON. The historic `pragma experimental SMTChecker;` is **deprecated** — "it still enables the SMTChecker for backwards compatibility, **but it will be removed in Solidity 0.9.0**." As of **0.8.37**, "SMTChecker: Emit a deprecation warning for the BMC engine" — i.e. **BMC is being retired and CHC is the surviving engine**.

**The soundness caveats are the docs' own:** a lack of warnings "represents an undisputed mathematical proof of correctness, **assuming no bugs in the SMTChecker and the underlying solver**"; but "these problems are *very hard* and sometimes *impossible* to solve automatically in the general case," so "several properties might not be solved or might lead to false positives for large contracts." A warning can mean two different things: "`<failing property> happens here`" (proved to fail, with a counterexample that "may also be a false positive in certain cases") versus "`<failing property> might happen here`" (undecided within the timeout, reported "for soundness"). The checker **requires an SMT solver** (historically Z3, optionally CVC4) and reasons under **real-world assumptions** the docs state explicitly — e.g. that a dynamic array's length cannot reach `2^256 − 1` and that an address balance cannot overflow.

**What else the compiler has shipped — and has not.** **0.1.7 (2015)** documents "Proof-of-concept transcompilation to why3," and the **`--formal` CLI option was removed in 0.5.0**. The why3 path is gone, and **the SMTChecker is the only solver-backed verification the compiler itself ships.** Anything stronger is external: `hevm` (symbolic execution, now Argot, whose engine Echidna consumes), `act` (Argot's formal specification language), and commercial tools (§6.3–§6.4). The framework walk-through is sibling §6.

**The compiler-bug process — verified.** `SECURITY.md` (argotorg/solidity, read this pass):

- **Scope:** "Only bugs that have a demonstrable security impact on smart contracts are in scope. For example, a Solidity program whose optimization is incorrect (e.g., leads to an incorrect output) qualifies as a security bug." Dependency bugs are out of scope "unless they result in a Solidity specific bug."
- **Supported versions:** "As a general rule, only the latest release gets security updates," with exceptions when the current breaking release is under about three months old.
- **Bounty:** reporting runs through the **Ethereum bounty programme** (`bounty.ethereum.org`), whose rules "have precedence over" the compiler's own policy.
- **Where advisories actually live:** the team maintains **"JSON-formatted lists of patched security vulnerabilities"** — `docs.soliditylang.org/en/develop/bugs.html` and **`docs/bugs_by_version.json`**. Notably, **the repository's GitHub `security/advisories` page shows "There aren't any published security advisories"** — advisories are *not* published through GitHub's advisory mechanism. Monitoring must watch the **docs `bugs.html` / `bugs_by_version.json` files and the blog's `security-alerts` category**, not a GitHub advisories feed.

**The worked example — the transient storage clearing helper collision bug.** Codegen bugs are why language-level review cannot substitute for compiler-version discipline.

| Attribute | Verified detail |
|---|---|
| Reported | **2026-02-11**, by **Hexens** |
| Severity | **High** on the team's internal scale |
| Affected | Compiler versions **0.8.28–0.8.33**, **only with the IR pipeline (`--via-ir`)** |
| Fixed in | **0.8.34**, tagged **2026-02-18** (shipped within a week) |
| Mechanism | The IR pipeline generates reusable Yul helpers deduplicated **by name**, and the storage-clearing helper's name derived from the **type** only — not the storage kind. So `storage_set_to_zero_t_address` was generated once and reused for both persistent (`sstore`) and transient (`tstore`) clearing; whichever operation the compiler met first fixed the opcode and the second used the wrong one. The fix adds the storage location to the name |
| Trigger | `delete` on a **transient** state variable in a compilation unit that also clears persistent storage of a **matching value type** (nested anywhere, including as an array element or struct member) |
| Blast radius | "Only three deployed contracts matching the affected pattern have been identified across all EVM-compatible chains." Identification involved **SEAL 911**, **Dedaub** and Etherscan data |
| Why it matters | Invisible at source level; triggers only under a specific codegen path; the effect is either *overwriting persistent state* or *failing to clear transient state* |

And the AI-era pointer: Argot's July 2026 roadmap records the team analysed **"Around 30 security reports"** in H1 2026 and that **"LLMs getting better resulted in an order of magnitude increase in the amount of vulnerability reports that the team had to investigate and react to,"** triggering "a general overhaul of the internal security procedures." That is a primary-source finding about *incoming reports*, not generated contracts — §8.4 returns to it.

---

## 6. The Tooling and Analysis Ecosystem

### 6.1 Compilers as a Library

| Binding | What it is | Notes |
|---|---|---|
| **`solc`** | The C++ compiler, release binaries | The reference implementation |
| **`solc-js`** | The **emscripten build of the same compiler** | The docs are explicit that the command-line section "does not apply to `solcjs`, not even if it is used in commandline mode" |
| **`libsolc`** | Embeddable C API | Changed in **0.6.0**: `solidity_free` renamed to `solidity_reset`; `solidity_alloc`/`solidity_free` added; `solidity_compile` returns a string that must be freed; the callback gained a two-parameter (kind, data) signature |
| **Standard JSON** | Language-agnostic input/output contract | The portable interface — Python (`py-solc-x`), JavaScript, Rust and Go bindings all drive `solc` through it or invoke the binary |

For automation, use standard JSON (§3.6): version-stable, language-independent, and it doubles as the reproducibility record.

### 6.2 Formatters and Linters

**The compiler does not ship a formatter.** The "Using the Compiler" page reviewed this pass covers the optimiser, path resolution, library linking, EVM-version selection, the error taxonomy, the experimental-mode gating table and the standard-JSON interface — there is **no formatting mode or style option** in it, and `solc` has no `--format`-style flag. Formatting comes from external tools, principally **`forge fmt`** (Foundry) and **`prettier-plugin-solidity`**.

**Linting** splits similarly. `solc` is a strict *static checker* — it enforces types, visibility, mutability, linearisation and the checks of §5.1, and emits graded warnings — but it is not a configurable style linter. Rule-based linting (naming, ordering, NatSpec coverage, gas hints) comes from external tools such as **Solhint**. The practical point: **"compiles without warnings" is not "meets the house standard,"** and a gate should require both the compiler's warning-clean state *and* the linter's rule set at a pinned version.

### 6.3 Static Analysers, with Maintenance Status Verified

| Tool | Approach | Maintenance status (verified this pass) |
|---|---|---|
| **Slither** | Python static-analysis framework; a suite of detectors plus an API for custom analyses. Works on **Solidity and Vyper** | **Actively maintained ✅.** `crytic/slither` (Trail of Bits / Crytic), AGPL-3.0, ~5,200 commits, ~6.4k stars, with a `CITATIONS.cff` and a research-prize programme |
| **Mythril** | **Symbolic execution on EVM bytecode**; produces transaction-sequence counterexamples | **Repository present, release cadence not verified ⚠.** `ConsenSysDiligence/mythril`, MIT, ~4,900 commits, ~4.3k stars. Treat as *available but re-verify before relying on it* |
| **MythX** | Commercial SaaS: Mythril's engine plus proprietary layers behind a paid API | **SUNSET ✅ (exact date ⚠).** The vendor's page (`mythx.io/sunset`): *"we have made the difficult decision to sunset the MythX suite, effective March 31st"*, redirecting users to *"Diligence Fuzzing."* Secondary write-ups date it to **31 March 2026**; the vendor page prints only day and month, so the **year is flagged** |
| **Securify** | Academic static analyser (ETH Zurich spin-out), first-generation | **Status not verified ⚠.** Do not cite as current |
| **Oyente** | The earliest public symbolic analyser (2016) | **Effectively historical ⚠.** Named because it appears in older literature |
| **SMTChecker** | The compiler's built-in verification targets (§5.7) | **Shipped with `solc` ✅**; BMC engine deprecation warning as of 0.8.37 ✅; default is no engine |

**On the tools' own accuracy claims.** Slither is a **detector suite**, so its usefulness depends on which detectors are enabled and their false-positive rate on *your* code. The only externally-measured number surfaced this pass is SmartEval's **79.4 % agreement** between an LLM auditor and Slither (§8.2) — and **agreement is not accuracy**. MythX's sunset is itself the strongest evidence against treating vendor detection claims as durable. And the sharpest warning comes from **Sourcify's own FAQ** (§6.6): *"Verification does not mean it's safe to interact with a contract. Verifiers do not check the contents of the contracts."* The same applies to analysers: a clean run is evidence of *absence of detected findings*, not of safety.

### 6.4 Test, Fuzz, and Invariant Tooling

The frameworks themselves — Foundry, Hardhat and their testing models — are sibling §6 and are not walked through again. The **language-level** points:

- **Property-based and invariant testing** suit Solidity's failure modes, because the invariants are usually statements about *storage* and *accounting* across arbitrary call sequences — what the storage model makes expressible.
- **`hevm`** (Argot) is "a symbolic execution engine and testing framework," and the roadmap notes it "is increasingly positioned as a foundational execution engine on which other tools can be built, **with Echidna demonstrating this model in practice**." It ships Forge cheatcodes (`expectRevert`, the `assertApprox*` family, `etch`) and is used by the Solidity team "to differentially fuzz new codegen pipelines in development (SSA-CFG pipeline)."
- **`act`** (Argot) is a formal specification language for EVM programs, with **EquiVM** ("proving equivalence between bytecode and high-level specification languages defined in Lean") and a game-theoretic reasoning backend in progress. **Research-grade**: the roadmap lists soundness proofs as "in progress." Do not put `act` on a deployment gate.
- The team maintains **`solc-bench`**, a statistical benchmarking harness with "Welch t-test significance gating" tracking instructions, cycles, CPU/wall time, peak memory, bytecode size and gas "for regression gating."

### 6.5 Debuggers and Source Maps

- **Source maps** are a compiler artefact. Since **0.6.0** the format carries **a fifth field, "modifier depth"** — a map producer and consumer disagreeing on field count is a silent mis-mapping.
- **`ethdebug`** (Argot) is a "smart contract debugging data format that aims to make the EVM readable." **0.8.35** replaced `--ethdebug`/`--ethdebug-runtime` with the more granular **`--ethdebug-resources`, `--ethdebug-compilation`, `--ethdebug-program`, `--ethdebug-program-runtime`** and decoupled the schema artefacts from bytecode generation. The roadmap calls it "ethdebug foundation," with "dedicated testing tools in the isoltest framework" and a "revamp of debug info storage in SSA CFG." **Status: experimental and gated (`--experimental`) — watch, do not depend on.**
- **Interactive debuggers** (Remix's, Foundry's traces) are the day-to-day tools; their fidelity rests on the source maps.

### 6.6 Verification Services

**Sourcify** — verified from the project's own site (read this pass): "an open-source and decentralized source-code verification service" whose method is "we take the Solidity/Vyper code provided by the developer or anyone, compile it, and compare the compiled bytecode with the onchain bytecode at that address … If they match, the contract is verified."

- **Provenance:** "It started under the Ethereum Foundation and **now is part of the Argot Collective**." Public good, non-profit, no revenue model.
- **Exact-match semantics:** Sourcify "supports 'exact match'es, which means the verified source-code is cryptographically guaranteed to be the same as the original one, **even whitespaces, comments etc.**" — a stronger claim than a plain bytecode match, and what makes verification usable as an audit artefact.
- **Open data and self-hosting:** the verified-contract dataset is published "in modern data formats," verified contracts are stored **on IPFS** so they are addressable "directly through the IPFS hash in the bytecode," and the service is self-hostable.
- **Ecosystem role:** "block explorers can use Sourcify as their verification backend, which is what **Blockscout** does"; Sourcify participates in the **Verifier Alliance** and the Clear Signing working group.
- **The limits, in Sourcify's own words:** verification proves the source matches the bytecode; it says nothing about safety or correctness. It is a *provenance* control, not a *correctness* control.

**Block explorers** (Etherscan and equivalents) provide the commercial verification experience. The operational distinction: an explorer's verification is a claim about **reproducibility under the exact compiler settings recorded in the metadata** (§3.4). If the settings are not archived, the explorer may still verify heuristically while a third party cannot reproduce the claim — which is why Sourcify's "exact match" and the archived standard-JSON input are the two artefacts a bank should insist on.

---

## 7. The Language Alternatives and the Comparison

### 7.1 Vyper

**What it is.** A "Pythonic smart contract language for the EVM" whose self-description is "a relentless focus on security, simplicity, and readability." Its philosophy is subtractive: where Solidity adds features and then safety mechanisms around them, Vyper omits the features.

**Current status — verified.** The repository (`vyperlang/vyper`) is **actively developed**: this pass read a master branch whose latest commit is dated **11 September 2026**, with ~6,250 commits and ~5.2k stars. Work in flight includes a **module system with `@abstract` and `@override`** decorators and an `initializes` relationship — Vyper acquiring a controlled form of composition where it previously had none. The docs' release notes describe **Venom**, an IR-based codegen path (`--venom` / `--experimental-codegen`) with a standalone `venom` binary.

**Stewardship and funding — flagged.** Vyper is maintained by the `vyperlang` team with **grant-dependent** funding. A Vyper team post states: *"We are sad to see ENS shutting down its public goods grants, but also incredibly thankful that Vyper was able to benefit from it last year. The grant allowed us to improve the compiler, start work on formal verification and unlock matching funding from @ethereumfndn."* Verified: Vyper **received** ENS public-goods grants plus matching EF funding, and **that channel has closed**. The *current* funding arrangement and any formal stewardship structure were **not** established (§11). There is also a reported community debate about relative funding between Vyper and Argot's **Fe** (a secondary source cites roughly **$219k** to Fe in late 2025) — **⚠ single-sourced and contested**.

**A name-collision trap, explicitly rejected.** A search for Vyper funding surfaces a claim that *"Vyper was acquired by pump.fun"* on **6 February 2026**. That is an **unrelated project sharing the name** (the listing's sector tags include meme/monitoring, not compilers). It is **not** the Vyper language, and it is recorded in §11 as a rejected claim.

### 7.2 Huff

**What it is.** A low-level assembly language for the EVM using **macros** — "blocks of bytecode that can be rigorously tested and evaluated" — rather than functions. It was **created by the Aztec Protocol team to write Weierstrudel**, "an on-chain elliptical curve arithmetic library that requires incredibly optimized code that neither Solidity nor Yul could provide."

**Current status — verified, and it is archived.**

- The Rust compiler **`huff-language/huff-rs` was archived by its owner on 20 October 2024** and is now read-only. Its README: **"`huff-rs` (aka legacy huff) is deprecated and no longer maintained, go see [huff2] for the renewed compiler."**
- The same README carries a warning worth quoting: *"This is experimental software … This code is not designed for safety. There are untested invariants in the code that may break. **You can easily shoot yourself in the foot if you're not careful.**"*
- A successor, **`huff2`**, exists as the "renewed compiler." Its maturity, maintenance and audit status were **not verified** (§11). The original implementation lives at `AztecProtocol/huff`.

**Verdict for a bank:** a specialist optimisation instrument with an archived primary toolchain. Writing a regulated production contract in Huff is not defensible on tooling-maturity grounds.

### 7.3 Fe, Act, and Core Solidity

Three projects sharing a steward (**Argot**), representing three bets on Solidity's future.

- **Fe** — "Statically-typed language for the EVM." Argot's July 2026 roadmap records that **"act and Fe each delivered their first releases following extensive rewrites, marking important milestones for both projects."** So Fe is **pre-1.0, post-rewrite, first release in H1 2026** — real, but far from a production alternative, with a nascent ecosystem.
- **Act** — a formal specification language for EVM programs, with the Lean-based EquiVM direction and CheckMate integration (§6.4). Research-grade.
- **Core Solidity** (`argotorg/solcore`) — the most consequential for anyone with a Solidity estate. Per its repository: *"This is a prototype implementation of **Solidity's new type system**. It is functional and can produce executable EVM code."* Its pipeline is `.sol → sol-core → .core → yule → .yul → solc → .hex` — **Core Solidity deliberately lowers to Yul and reuses the existing `solc` backend** rather than replacing it. The README is unambiguous: *"The compiler contains bugs. **You should under no circumstances be using this in a production setting.**"* Argot's roadmap says Core Solidity "has reached the point where smart contracts can be written and compiled end-to-end," that "essential contracts have been successfully ported (e.g., the beacon chain deposit contract, wETH, a multisig, a Uniswap v2 vault)," and that H2 2026 focuses on "preparing and starting the production implementation of the language." A design overview was published on the Solidity blog in **November 2025** ("Core Solidity Deep Dive").

**The strategic reading.** Argot is simultaneously maintaining **Classic Solidity** (the language in production), building **Core Solidity**, shipping **Fe**, and researching **Act/EquiVM**. For a bank: *Classic Solidity is the only production language*; **Core Solidity is the thing to watch**, because it is explicitly designed for eventual migration ("This ensures that it will be as easy as possible for existing developers to migrate to Core Solidity"); and the coupling of Classic Solidity and Core Solidity through the **SSA-CFG backend** ("it will use the same backend") is how Classic Solidity gets faster compiles.

### 7.4 Yul, Yul+, and the Assembly Layer

**Yul** is the intermediate language (§3.2) *and* a standalone language: no Solidity types, the target of the IR pipeline. It is used three ways — as inline assembly inside Solidity, as a standalone target (`--strict-assembly`), and as the optimiser's input on the via-IR path. Since **0.6.0** only **strict** inline assembly is permitted; since **0.8.0** reserved identifiers (including EVM instruction names) are disallowed in Yul even when unavailable in the target dialect. Yul gained a `leave` statement in 0.6.0 and continues to gain fork-tied builtins (`clz` in 0.8.31, `slotnum` in 0.8.37). **Yul+** is a community extension whose maintenance status was **not verified**. The language-level rule is the one from §5.5: **inside Yul, every compiler-inserted safety check is absent.**

### 7.5 The Non-EVM Families

The comparison that matters at design level is **which safety philosophy each family chose** — the tooling and audit markets follow the philosophy.

| Family | Chain(s) | Safety philosophy (design-level) | Status verified this pass? |
|---|---|---|---|
| **Solidity / Vyper / Fe / Huff** | EVM chains | **Imperative + progressively-enforced discipline.** Safety added incrementally by the language (§4.1) and enforced by external analysis | ✅ for the projects above; language claims are domain-stable |
| **Rust** (with Anchor) | Solana | **Ownership and borrowing enforced by the type system**; memory-safety guarantees the EVM family does not attempt | ⚠ Not verified — §11 |
| **Move** | Sui, Aptos | **Resource types / linearity.** Assets are first-class resources that cannot be duplicated or implicitly discarded, making double-spend-style bugs a *type error* rather than a runtime exploit | ⚠ Not verified — §11 |
| **Cairo** | StarkNet | **Prover-friendly typed language**; the STARK-provable execution model shapes language and cost model | ⚠ Not verified — §11 |
| **Clarity** | Stacks | **Decidable and deliberately non-Turing-complete** (no unbounded loops), so behaviour is analyzable in advance | ⚠ Not verified — §11 |
| **Michelson** | Tezos | **Formal-verification-first stack language**, designed for on-chain formal reasoning | ⚠ Not verified — §11 |

**Honest framing.** This pass **verified** the Solidity-family statuses and read primary sources for the EVM-adjacent projects. It did **not** verify the current state of Solana/Anchor, Sui/Aptos/Move, StarkNet/Cairo, Stacks/Clarity or Tezos/Michelson — those entries state *design-level* characteristics stable for years, and every claim about their *present* maturity, maintenance or adoption is **explicitly flagged as unverified** in §11.

### 7.6 The Comparison Table

Scored on the axes a bank's technology-risk function actually uses. **Solidity's row is verified; the alternatives' rows are directional and flagged.**

| Axis | Solidity | Vyper | Huff | Fe | Core Solidity | Yul |
|---|---|---|---|---|---|---|
| **Safety philosophy** | Imperative, checks added by breaking release; external analysis expected | Subtractive: omit features, then no checks needed | None claimed — "not designed for safety" | Static types, modern design | New type system, reuses existing backend | None — raw |
| **Tooling maturity** | **Highest** — compiler, analysers, formatters, debuggers, verifiers all mature | Good and improving; new IR (Venom) in flight | **Archived** primary toolchain | First release only (H1 2026) | Prototype; **"under no circumstances … production"** | Mature as an IR; primitive as a hand-written language |
| **Audit familiarity** | **Highest** — the default auditor skillset | Moderate — auditors exist, fewer | Low — specialist niche | Very low | Essentially none yet | Low |
| **Ecosystem adoption** | **Dominant** on EVM chains | Niche but real (notably security-critical protocols) | Small; used where gas is paramount | Negligible | None in production | Pervasive as a *compilation target*, not a source choice |
| **Interoperability** | Baseline (defines the ABI) | Full — same ABI (§2.7) | Full — compiles to bytecode | Full — same target | None (may eventually inherit) | Not applicable |
| **Recommended for production?** | **Yes**, with §3.4/§4.4 discipline | Possibly, with a smaller auditor pool | **No** | **No** | **No** | Only inside Solidity/Vyper output |

**The interoperability row decides most real questions.** Because all of these languages compile to the same EVM and speak the same ABI (§2.7), choosing Vyper over Solidity does **not** isolate a bank from the Solidity ecosystem — the contracts call each other. Language choice on the EVM is therefore a *tooling and audit-market* decision, not an interoperability one — which is why "audit familiarity" carries the most weight and the least glamour.

---

## 8. The AI-Era Angle

Scoped to what is **documented**. It cross-references the repository's `ai_llm/` cluster rather than re-deriving LLM fundamentals, and separates measured findings from speculation (§8.5).

### 8.1 What Is Documented: LLM Generation Benchmarks

The substantive published artefact located this pass is **SmartEval** (arXiv:**2605.09610**, submitted **10 May 2026**, by Abhinav Goel, Agostino Capponi, Alfio Gliozzo and Chaitya Shah — an academic group with Columbia University affiliations). Its abstract, read directly, states:

- **Corpus:** "9,000 generated contracts paired with expert-written ground-truth implementations drawn from the FSMSCG dataset."
- **Rubric:** "a five-dimensional evaluation rubric covering **functional completeness, variable fidelity, state-machine correctness, business-logic fidelity, and code quality**."
- **Three independent validation studies:** a "five-condition ablation study (N=300 per condition)"; a "human expert evaluation by three Columbia University PhD researchers confirming automated scores align with expert judgment to within **0.34 points**"; and "external security analysis via the **Slither** static analyzer confirming **79.4 % agreement** between the LLM auditor and a non-LLM rule-based tool."
- **Failure modes across the 9,000 contracts:** "**logic omissions at 35.3 %**, **state transition errors at 23.4 %**, and **complexity-driven degradation**."
- **A counter-intuitive headline:** "a **+8.29 composite-score advantage of generated contracts over ground-truth implementations**, attributable to LLMs' literal specification-following behavior."

**Methodology assessment — and the contamination risk.**

- **Strong:** three validation studies including an independent human-expert check and an external tool cross-check, with dataset, rubric and pipeline published.
- **Sceptical:** the **+8.29 advantage over expert-written ground truth** demands scrutiny before repetition. The paper's own explanation — LLMs' "literal specification-following behaviour" — implies the ground truth optimises for something the rubric does not reward (engineering judgement, defensive coding, future-proofing) while the LLM optimises exactly for the specification as written. A rubric can prefer the literal to the correct. This guide reports the number because it is published and attributed; it does **not** endorse reading it as "AI writes better contracts than experts."
- **Benchmark contamination risk:** where ground truth comes from a **public dataset** and the generator is a model trained on public code, the model may have seen the reference implementations. The design mitigates this partially by generating from *natural-language specifications*, but the risk is structural and cannot be ruled out from the abstract. Assume contamination and require **held-out, internal** evaluation.
- **A naming discrepancy to flag:** the abstract says **"FSMSCG dataset"**; the project's own dataset record and one artefact description render it **"FSM-SCG"** (§11).

### 8.2 AI-Assisted Audit Tools and Their Measured Effectiveness

**The measured numbers are thin, and one is widely misread.**

| Claim | What it actually measures | Verdict |
|---|---|---|
| "79.4 % agreement between the LLM auditor and a non-LLM rule-based tool" (SmartEval, via Slither) | **Agreement** — the two found the same things; *not* accuracy, recall against known bugs, or precision | ⚠ **Frequently miscited as a detection rate.** Two auditors agreeing is not two auditors being right; correlated blind spots are invisible to this metric |
| "Human expert evaluation … automated scores align with expert judgment to within 0.34 points" | Alignment between rubric scores and human scoring on a 0.34-point tolerance | ⚠ Evidence the *rubric* is sound; says nothing about the *security* of the generated contracts |
| Vendor claims of AI auditors with "impressive detection rates" and quick turnarounds | **Vendor marketing**, surfaced in the same search as the MythX sunset coverage | ❌ **Not substantiated.** No methodology, dataset or independent evaluation. Excluded as evidence |

**The structural caution.** AI-assisted audit tools are entering an ecosystem that just lost a major commercial service (§6.3) — the MythX sunset created a gap, and a gap attracts vendors whose claims outrun their evidence. An evaluation protocol for any AI audit tool must require the **dataset and rubric** used, the **false-positive and false-negative rates on the bank's own codebase**, the **model version** (since model updates invalidate past results), and an explicit statement of **whether the tool was evaluated against known-bug corpora or against agreement with another tool** — entirely different properties, and §6.3 shows how easily "agreement" is read as "accuracy."

### 8.3 Reported Failure Modes of Generated Contract Code

The documented failure modes from SmartEval, in order of frequency: **logic omissions (35.3 %)**, **state transition errors (23.4 %)** and **complexity-driven degradation**. Notably, these are **not** the classic Solidity vulnerability classes:

- **Logic omission** is a *completeness* failure — the specification was partially implemented. Standard static analysis does not detect it, because the code that would fail any given detector's pattern is **absent rather than present**.
- **State transition error** is a *state-machine* failure — an invalid transition permitted, or a valid one blocked. Invisible to pattern-based analysis; visible to invariant testing (§6.4) and to the SMTChecker's assertion targets (§5.7), if the assertions are written.
- **Complexity-driven degradation** means quality correlates inversely with contract size — generated code is least trustworthy exactly where the money is, in large multi-contract systems.

To these, this guide adds one **domain-stable** observation rather than a benchmark finding: because much public Solidity predates 0.8.0 (§4.2), models trained on that corpus reproduce **pre-0.8 idioms** — `SafeMath` wrappers, unjustified `unchecked`, revert strings instead of custom errors (a 0.8.4 feature), and older visibility/location conventions. A reviewer of generated code should check **which language era the code was written in**.

### 8.4 The Security Implication at Scale

Two distinct pressures, which must not be conflated:

1. **The compiler team's own reported experience — primary source.** Argot's July 2026 roadmap records that the Solidity team handled **"around 30 security reports"** in H1 2026 and that **"LLMs getting better resulted in an order of magnitude increase in the amount of vulnerability reports that the team had to investigate and react to,"** triggering "a general overhaul of the internal security procedures to deal with the increased load." **The honest reading:** "an order of magnitude increase in vulnerability reports" does **not** mean an order of magnitude more *bugs* — it means more *reports*, and LLM-assisted reporting plausibly generates many plausible-but-spurious findings. The verified fact is the **load**, not a defect rate. The follow-on effect *is* unambiguously good: a **high-severity IR codegen bug was found and fixed in a week** (§5.7), so the same scrutiny that generates noise also generates real finds.
2. **The production-volume pressure — inference, flagged as such.** If a growing share of deployed contracts is AI-generated, the population acquires the failure modes of §8.3 at scale. The genuine concern is therefore **not** that generated code is obviously insecure — it is that generated code is *plausibly secure-looking* and fails in ways pattern-based tooling does not catch. The controls that address this do not depend on pattern-matching: **invariant testing** (§6.4), **assertion-based formal verification** (§5.7), **storage-layout diffing** (§5.2) and **compiler-version discipline** (§4.4).

### 8.5 Documented Findings versus Speculation

| Statement | Status |
|---|---|
| SmartEval's corpus size, rubric dimensions, validation studies, failure-mode percentages, and the +8.29 result | **Documented** — arXiv:2605.09610 abstract, read this pass |
| Slither's 79.4 % *agreement* with an LLM auditor | **Documented**, but **commonly misread as accuracy** |
| The Solidity team's ~30 reports and order-of-magnitude report increase attributed to better LLMs | **Documented** — Argot roadmap, read this pass |
| A high-severity IR codegen bug was found and fixed within a week in 2026 | **Documented** — Solidity blog + changelog |
| Vendor claims of AI auditors' detection rates | **Marketing; not substantiated** |
| "AI-generated contracts are more insecure than human-written ones" | **Speculation.** SmartEval's composite score points the *other* way; neither claim is established for security specifically |
| "Corpus contamination invalidates LLM benchmark results" | **Speculation / risk** — structural, real, unquantified; requires held-out internal evaluation |
| "A large body of AI-written contracts in production will produce a wave of losses" | **Speculation.** Plausible given §8.3; **no measured evidence located this pass** |
| "LLM-generated security reports are mostly spurious" | **Speculation**, *suggested* by the volume/quality asymmetry in §8.4 but not stated by any source read |

---

## 9. The Cymbal Bank Worked Example

### 9.1 The Design-Fiction Frame

[smart_contracts_guide.md](smart_contracts_guide.md) §11 designs a **Cymbal Bank tokenized money-market fund vault** and works through its architecture, standards choices (ERC-20 / ERC-4626) and SWC-mapped threat model. This section does **not** repeat any of that. It takes the same vault — call the derivative under review **CYV-2**, a revised deployment of the same fictional product — and reviews it at the **language and compiler** level only: the layer the sibling guide explicitly does not own.

**Everything numeric below is illustrative and fictional.** Addresses, the version chosen for the example, and organisational controls are invented for the walkthrough. The *technical properties* examined (what a pragma means, what a layout rule is, what `solc` emits) are the verified mechanics of §2–§6.

### 9.2 Version and Pragma Discipline

The highest-value language-level review of any contract is the first five lines. For CYV-2:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.37;

import {ERC4626} from "@openzeppelin/contracts/token/ERC20/extensions/ERC4626.sol";
```

| Check | Finding against CYV-2 | Rule |
|---|---|---|
| Is the pragma **exact**? | `0.8.37`, not `^0.8.0` | §4.4 — an exact pin removes the future-0.8.x degree of freedom |
| Is the **build toolchain** pinned to match? | CI's `solc`/`solc-js` version and container digest must match the pragma | §4.4 — the pragma constrains; the toolchain decides |
| Is there an **SPDX identifier**? | Required; omitting it produces a warning and a licensing gap | Compiler hygiene |
| Is the version **after** the last breaking boundary? | `0.8.37` is post-0.8.0, so **checked arithmetic applies** and custom errors are available | §4.1, §4.2 |
| Is the code **free of pre-0.8 idioms**? | Flag any `SafeMath` import, any `revert("string")` where a custom error belongs, any `now`/`suicide`/`throw` | §4.2, §4.3 |
| Is the **dependency version** pinned? | OpenZeppelin pinned by tag or commit — a floating `^5.x` dependency is a floating compiler-input change | §3.4 |
| Are there **deprecation warnings**? | The build must be warning-clean; deprecations ahead of 0.9.0 (`send`/`transfer`, ABI coder v1) are findings, not noise | §4.3 |

The principle: **a contract's version declaration is a statement about how it will behave, and it is unverifiable without the toolchain record.** A reviewer who accepts `^0.8.0` has accepted that the contract's arithmetic semantics, default EVM target and available builtins are unspecified.

### 9.3 Storage-Layout and Upgrade-Safety Review

CYV-2 is an upgradeable vault, so the layout is the risk. This is a **source-level** procedure producing an artefact; the proxy *pattern* is sibling §7.

1. **Extract both layouts mechanically.** Compile CYV-1 (deployed) and CYV-2 with `storageLayout` in the standard-JSON `outputSelection` (§3.6), yielding slot, byte offset and type for every variable.
2. **Diff them** as a variable → slot → offset → type table.
3. **Apply the append-only rule** (§5.2): CYV-2's layout must be a **prefix-preserving extension** of CYV-1's. New variables only at the end.

| Pattern | Why it breaks | Verdict |
|---|---|---|
| Reordering two existing `uint128` members | Changes packing: one slot becomes two, shifting everything after | ❌ Blocking |
| Removing a deprecated variable | The slot is reused by the next declaration; old data is read as the new variable | ❌ Blocking |
| Changing `mapping(address => uint256)` to `mapping(address => uint128)` | Changes packing inside the mapping's slots; every entry is reinterpreted | ❌ Blocking |
| Adding a **new base contract** to the `is` list | Base variables occupy the **prefix** in linearisation order (§2.5) | ❌ Blocking |
| Reordering the existing `is` list | The C3 order changes, so the inherited prefix changes | ❌ Blocking |
| Adding a new variable **after all existing ones** | Append-only extension | ✅ Allowed |
| Changing `constant`/`immutable` declarations | These occupy **no storage**, so they cannot collide | ✅ Allowed, but check the value is intentional |
| Introducing an **ERC-7201 namespaced** layout | Namespaced slots derive from a hashed base and do not collide with the legacy sequential layout | ⚠ Allowed, but the design must *prove* the namespace is disjoint and document the transition |

**What this cannot establish:** it verifies that the *declarations* are compatible. It does **not** verify that the *initialiser* correctly handles existing storage — a perfect prefix can still be broken by an `initialize()` that overwrites a live slot. That check is behavioural and requires a state-transition test against a fork of the live chain.

### 9.4 Compiler Settings and Verification Posture

The compiler settings are part of the contract's identity (§3.4), so the review records them as a table that must match across build, deployment and verification.

| Setting | CYV-2 value (illustrative) | Why |
|---|---|---|
| Compiler version | `0.8.37` (pinned) | Matches the pragma; post-advisory-clean for §5.7 |
| Optimiser | enabled | Production standard |
| `runs` | `200` | The default; CYV-2 is not a hot-path AMM core (§3.3) |
| `viaIR` | `false` **at launch** | The IR pipeline is where the §5.7 high-severity bug occurred. Adopting IR must be justified by a gas benefit large enough to offset the added codegen-path risk — and must be recorded |
| `evmVersion` | pinned explicitly, not left to the compiler default | The default moved Istanbul → Prague → Osaka across releases (§3.7). A default is not a decision |
| `metadata.bytecodeHash` | `ipfs` | Verification is a required control; `none` forfeits it (§3.4) |
| `revertStrings` | `debug` (or `default`) | Confirms custom-error arguments survive the chosen stripping mode — a 0.8.35 bugfix area (§3.5) |
| `--experimental` | **not used** | SSA-CFG, EOF and the other gated features are explicitly not production (§3.2) |

**Verification posture.** The release must publish and archive immutably: (1) the pinned source tree with dependency versions; (2) the **complete standard-JSON input**; (3) the compiler binary hash or `solc-js` version; (4) the resulting bytecode and metadata; and (5) the **Sourcify exact-match** record and explorer verification link. An explorer "match" without the archived inputs is not reproducible by a third party, which is the whole point (§6.6).

### 9.5 The Language-Level Review Checklist

Derived directly from §2–§5. Deliberately *linguistic*: it asks whether the language was used in the way that makes the code reviewable, and leaves the vulnerability classes to the sibling guide.

| # | Check | Language rule |
|---|---|---|
| 1 | Every function declares **visibility** | Mandatory since 0.5.0 (§5.1) |
| 2 | Every reference-type declaration declares a **data location** | Mandatory since 0.5.0 (§2.3) |
| 3 | Every input that is only read is declared **`calldata`**, not `memory` | §2.3 |
| 4 | Every state-mutating external entry point is non-payable or explicitly `payable` — none incidental | `payable` required since 0.4.0 (§2.4) |
| 5 | Every variable that never changes after construction is **`immutable`** or **`constant`** | §5.2 — these occupy no slot |
| 6 | No `tx.origin` used for **authorisation** | §5.4 |
| 7 | Every `unchecked { }` carries a **stated justification** | §5.5 |
| 8 | Every **narrowing conversion** is preceded by an explicit bounds check | §5.5 — downcasts are unchecked by design |
| 9 | Every external call is **state-after-call-free**: writes precede calls, or a guard is present | §5.6 |
| 10 | The reentrancy guard is applied to **every** externally reachable state-changing function | §5.6 |
| 11 | **No `delegatecall`** except through an audited, trust-minimised path | §5.3 |
| 12 | Every interface cast (`IX(addr)`) is accompanied by a check of what is deployed at `addr` | §2.6 — the language performs no such check |
| 13 | The **`is` list and its order** are frozen and reviewed for storage implications | §2.5, §5.2 |
| 14 | Overrides annotated `override`; overridable functions `virtual` | Since 0.6.0 (§2.5) |
| 15 | **Custom errors** used instead of revert strings on all revert paths | Since 0.8.4 (§2.4) |
| 16 | Events emit **all** state transitions a downstream ledger needs, with correct `indexed` choices | §2.4, §2.7 |
| 17 | All hashing/signature construction uses **`abi.encode`** unless `abi.encodePacked` is provably collision-free | ABI spec's own warning (§2.7) |
| 18 | No raw `keccak256` over packed dynamic values | Same |
| 19 | Inline assembly is **minimal, documented and reviewed separately** | §5.5, §7.4 |
| 20 | The storage layout is **append-only** versus the previous version | §5.2, §9.3 |

### 9.6 Analyser and Audit Gates

| Gate | Tool | What it establishes | What it does not |
|---|---|---|---|
| Compile, warning-clean | `solc` pinned | No type, visibility, mutability or linearisation errors; no deprecation warnings | Anything about design correctness |
| Storage-layout diff | `storageLayout` artefact | Layout is append-only | That initialisers preserve existing state (§9.3) |
| Static analysis | **Slither**, pinned version and detector set | No findings from the enabled detectors | Absence of undetected classes |
| Symbolic / property testing | Invariant tests (+ `hevm` as engine) | Invariants hold across arbitrary call sequences, within the harness's model | Anything the harness does not express |
| Assertion verification | **SMTChecker**, CHC engine, explicit targets | The assertions **written** hold — "an undisputed mathematical proof … assuming no bugs in the SMTChecker and the underlying solver" | Anything not asserted; anything the solver cannot decide (reported as "might happen") |
| Manual audit | Human reviewers using §9.5 | Design-level and economic-logic review | Compiler-level defects (the §5.7 bug was found by a security firm, not a source review) |
| Build integrity | Advisory monitoring on `docs/bugs_by_version.json` + the blog's security-alerts category | Awareness of newly disclosed compiler bugs affecting the pinned version | Nothing preventive — it is a monitoring gate |
| Verification | **Sourcify exact match** + explorer | The deployed bytecode corresponds to the reviewed source under the recorded settings | That the contract is safe — Sourcify's own FAQ says so (§6.6) |

Two gate-design rules. **No single gate is a safety claim** — a clean Slither run and a passing SMTChecker run cover disjoint properties, and neither covers the design. And **every gate must be version-pinned**, because an analyser's detector set and an SMT solver's behaviour are themselves dependencies.

### 9.7 Sign-Off Criteria and the Limits of a Language-Level Review

**What a regulated institution should require before CYV-2 goes live:**

1. **Reproducible build demonstrated by a third party** — an independent party recompiles the archived standard-JSON input with the pinned compiler and obtains byte-identical output. (§3.4, §3.6, §9.4)
2. **Layout compatibility proven structurally and behaviourally** — the append-only diff passes *and* a state-transition test against a fork of the live chain shows existing balances and shares preserved. (§5.2, §9.3)
3. **Compiler-version risk accepted in writing** — `SECURITY.md` supports only the latest release (§4.4); the residual risk of the pinned version is documented with an owner and a review date.
4. **Advisory-monitoring process live** — designated owners watching `bugs_by_version.json` and the security-alerts blog, with a defined response path. (§5.7)
5. **Every checklist item in §9.5 evidenced, not asserted** — with the reviewer's justification for each deliberate deviation.
6. **Analyser and verification gates run at pinned versions, results archived** alongside the build artefacts, with the **limits of each tool recorded** (§9.6).
7. **A single accountable engineering owner** for the contract's language-level posture, distinct from the product owner.

**And the honest statement of what this review can and cannot establish.**

A language-level review of CYV-2 **can** establish: that the source is version-pinned and free of deprecated constructs; that the arithmetic, visibility, data-location and mutation semantics are the intended ones; that the storage layout is compatible with the deployed version; that the compiler settings are recorded and the build is reproducible; that the code follows the reviewable conventions the language now forces; and that the standard analyser and verification gates have run at known versions with archived results.

It **cannot** establish: that the design is correct (that is the sibling guide's §11 threat model and the auditors' economic review); that the compiler is free of undisclosed bugs (the §5.7 bug was found by an external security firm after release, and was invisible at source level); that a clean analyser or SMTChecker run means safety (both have documented failure modes, and the SMTChecker documents false positives); that the *initialiser* correctly handles pre-existing storage (§9.3); that the off-chain components — the price oracle, the middleware, the operational key management — behave correctly; or that the contract is legally or regulatorily sound. **The compiler is inside the audit boundary; it is not the whole boundary.**

CYV-2's review therefore produces a bounded, evidenced claim — "this source, compiled these ways, deployed this bytecode, with these tools and these recorded limits" — which is exactly the claim a regulated institution can defend, and exactly the claim a review conducted only at the design level cannot make.

---

## 10. The Claims Audit

Every factual research claim in this guide, with its verdict (**✅ verified** against a primary/authoritative source read this pass · **⚠ flagged** reported/approximate/single-sourced/contested · **❌ rejected**) and the source named.

| # | Claim | Verdict | Source named |
|---|---|---|---|
| 1 | Solidity was proposed in **August 2014 by Gavin Wood** | ⚠ | Secondary (Wikipedia + mirrors); Wikipedia's own `[non-primary source needed]` tag. Absent from the docs and changelog |
| 2 | Early development led by **Christian Reitwiessner** | ⚠ | Secondary sources only |
| 3 | First tagged release **0.1.0, 2015-07-10** | ✅ | `argotorg/solidity` → `Changelog.md` (final line) |
| 4 | Formal verification surfaced early: 0.1.7 (2015-11-17) documents why3 transcompilation | ✅ | `Changelog.md` |
| 5 | **Stewardship moved to the Argot Collective** (independent non-profit, "most notably Solidity") | ✅ | `argot.org` homepage |
| 6 | Argot **introduced October 2024**; EF commitment = **5 years of runway at ETH USD 2,592** | ✅ | `argot.org/blog/2026-06-30-ef-funding-final-part` |
| 7 | Final funding part completed **30 June 2026**; ~**4,938 stETH** in a **2-of-3 multi-sig** (Argot / Fail-Safe Committee / EF); unlocks 1 July 2026 and 1 July 2027, 50 % each | ✅ | Same post |
| 8 | Fail-Safe Committee = 5 independent members + 1 Argot + 1 EF (members named in the post) | ✅ | Same post |
| 9 | Canonical repo is **`github.com/argotorg/solidity`**; `ethereum/solidity` redirects there | ✅ | GitHub fetch of both paths |
| 10 | The changelog file was **renamed** (`CHANGELOG.md` → root `Changelog.md`); the old path 404s | ✅ | GitHub 404 + successful fetch of `Changelog.md` |
| 11 | Latest tagged release **0.8.37, 2026-09-10**; 0.8.38 marked *unreleased* | ✅ | `Changelog.md` |
| 12 | Docs render as **"Solidity 0.8.38-develop documentation"** | ✅ | `docs.soliditylang.org` ABI spec + v0.8.35 pages |
| 13 | Breaking boundaries **0.4.0 (2016-09-08) / 0.5.0 (2018-11-13) / 0.6.0 (2019-12-17) / 0.7.0 (2020-07-28) / 0.8.0 (2020-12-16)** | ✅ | `Changelog.md` headings |
| 14 | **0.8.0 made arithmetic checked by default**; `unchecked { }` added | ✅ | 0.8.0 release announcement (16 Dec 2020) + `Changelog.md` |
| 15 | **`Panic(uint256)` selector `0x4e487b71`**; `Error(string)` selector `0x08c379a0`; panic codes `0x01/0x11/0x12/0x21/0x22/0x31/0x32/0x41/0x51` | ✅ | 0.8.0 release announcement |
| 16 | **Downcasts were deliberately left unchecked** in 0.8.0 | ✅ | 0.8.0 release announcement |
| 17 | **Custom errors (`error X(...)`) introduced in 0.8.4** (April 2021) | ✅ | soliditylang.org "Custom Errors in Solidity" + 0.8.4 announcement |
| 18 | Visibility and explicit data location became **mandatory in 0.5.0**; `var`/`throw`/`sha3`/`suicide` disallowed | ✅ | `Changelog.md` 0.5.0 |
| 19 | `virtual`/`override`, `try`/`catch`, the `receive()`/`fallback()` split, and the state-variable-shadowing ban all landed in **0.6.0** | ✅ | `Changelog.md` 0.6.0 |
| 20 | `now` and `finney`/`szabo` removed in **0.7.0**; `using A for B` re-scoped; constructor visibility deprecated | ✅ | `Changelog.md` 0.7.0 |
| 21 | **`--experimental-via-ir` and `settings.viaIR` added in 0.7.5** | ✅ | `Changelog.md` 0.7.5 |
| 22 | **Experimental SSA-CFG codegen shipped opt-in in 0.8.35** (`--via-ssa-cfg` / `settings.viaSSACFG`) | ✅ | `Changelog.md` 0.8.35 + Argot roadmap 2026-07-01 |
| 23 | **EOF backend removed in 0.8.36**; LSP mode and `pragma experimental solidity` removed in 0.8.37 | ✅ | `Changelog.md` |
| 24 | **`--experimental` gating framework introduced in 0.8.35**; experimental state recorded in JSON + CBOR metadata | ✅ | `Changelog.md` + docs `using-the-compiler` |
| 25 | Default optimiser **`runs` = 200**; the deployment-vs-runtime trade quoted verbatim from the docs | ✅ | docs `using-the-compiler` |
| 26 | Metadata hash method selectable **IPFS vs Swarm since 0.6.0** | ✅ | `Changelog.md` 0.6.0 |
| 27 | `abi`/`devdoc`/`userdoc`/`storage-layout` became **sub-objects in 0.8.0**; `legacyAST` removed | ✅ | `Changelog.md` 0.8.0 |
| 28 | **`storageLayout` artefact available since 0.5.13** | ✅ | `Changelog.md` 0.5.13 |
| 29 | Source maps gained a **5th field ("modifier depth") in 0.6.0** | ✅ | `Changelog.md` 0.6.0 |
| 30 | **ERC-7201 base-slot builtin added in 0.8.35** | ✅ | `Changelog.md` 0.8.35 + Argot roadmap |
| 31 | Default EVM version: **Istanbul (0.5.14) → Prague (0.8.30) → Osaka (0.8.31)** | ✅ | `Changelog.md` |
| 32 | EVM versions deprecated in **0.8.37**: constantinople, petersburg, istanbul, berlin | ✅ | `Changelog.md` 0.8.37 |
| 33 | **The ABI specification lives in the Solidity docs**, with a summary in the Ethereum JSON-RPC docs | ✅ | Both pages read this pass |
| 34 | Selector = first 4 bytes of Keccak-256 of the canonical signature; **return type excluded** | ✅ | Solidity ABI spec |
| 35 | The `abi.encodePacked` collision warning sits in the **specification** | ✅ | Solidity ABI spec |
| 36 | **`tx.origin`/`msg.sender` retyped to `address` in 0.8.0** | ✅ | `Changelog.md` 0.8.0 |
| 37 | **`solc` ships no formatter**; formatting is external (`forge fmt`, prettier-plugin-solidity) | ⚠ | Absence of any formatter in the docs' "Using the Compiler" page — negative evidence, not a positive statement |
| 38 | **SMTChecker default = no engine**; engines BMC + CHC; `pragma experimental SMTChecker` deprecated and **to be removed in 0.9.0** | ✅ | docs `smtchecker` (v0.8.35) |
| 39 | SMTChecker targets (underflow/overflow not default for ≥0.8.7; div-by-zero; trivial conditions; empty pop; OOB index; insufficient funds), soundness caveats, real-world assumptions | ✅ | docs `smtchecker` |
| 40 | **BMC engine deprecation warning as of 0.8.37** | ✅ | `Changelog.md` 0.8.37 |
| 41 | **`--formal` CLI option removed in 0.5.0** (the end of the why3 path) | ✅ | `Changelog.md` 0.5.0 |
| 42 | `SECURITY.md`: **only the latest release gets security updates**; scope requires demonstrable smart-contract impact; reporting via `bounty.ethereum.org` | ✅ | `argotorg/solidity` `SECURITY.md` |
| 43 | Advisories live in **`docs/bugs.html` + `docs/bugs_by_version.json`**, not GitHub advisories | ✅ | `SECURITY.md` + the repo advisories page showing "There aren't any published security advisories" |
| 44 | **Transient storage clearing helper collision bug**: reported 2026-02-11 by **Hexens**; affects **0.8.28–0.8.33 under `--via-ir`**; **high** severity; fixed in **0.8.34 (2026-02-18)**; **3** affected deployed contracts; root cause = helper name derived from type, not storage kind | ✅ | soliditylang.org security post, 18 Feb 2026 |
| 45 | Argot roadmap: **~30 security reports** in H1 2026 and an **"order of magnitude increase"** attributed to better LLMs; procedures overhauled | ✅ | `argot.org/blog/2026-07-01-argot-roadmap-update-2026-2` |
| 46 | The team's stated H1 2026 priority = fixing **stack-too-deep** via SSA-CFG; H2 2026 = stabilise and un-gate it | ✅ | Same roadmap |
| 47 | **Slither** actively maintained (Solidity **and Vyper**, Python, AGPL-3.0, ~5.2k commits) | ✅ | `crytic/slither` README |
| 48 | **Mythril** exists as a ConsenSys Diligence symbolic-execution tool (MIT, ~4.9k commits) | ✅ existence; ⚠ release cadence |
| 49 | **MythX sunset** — vendor page: "sunset the MythX suite, effective March 31st", redirecting to "Diligence Fuzzing" | ✅ on the sunset; ⚠ on the **year** (vendor page prints day/month only; secondary sources say 2026) |
| 50 | **Huff's `huff-rs` archived 20 Oct 2024**; README: "deprecated and no longer maintained," pointing to `huff2`; "not designed for safety" | ✅ | `huff-language/huff-rs` archive banner + README |
| 51 | Huff was originally built by **Aztec Protocol** to write **Weierstrudel** | ✅ | `huff-rs` README |
| 52 | **Vyper** actively developed (latest commit **11 Sep 2026**); module system with `@abstract`/`@override`; **Venom** IR codegen (`--venom`) | ✅ | `vyperlang/vyper` master + `docs.vyperlang.org` release notes |
| 53 | Vyper received **ENS public-goods grants + matching EF funding**, and that grant channel has closed | ✅ | Vyper team post (quoted) |
| 54 | "Vyper was acquired by pump.fun, Feb 2026" | ❌ **Rejected** | Name collision with an unrelated project; not the Vyper language |
| 55 | **Fe** shipped a first release in H1 2026 after an extensive rewrite | ✅ | Argot roadmap 2026-07-01 |
| 56 | **Core Solidity** (`argotorg/solcore`) is a prototype of "Solidity's new type system"; pipeline `.sol → sol-core → .core → yule → .yul → solc`; README: "under no circumstances… production" | ✅ | `argotorg/solcore` README |
| 57 | Core Solidity ported the beacon chain deposit contract, wETH, a multisig and a Uniswap v2 vault; H2 2026 begins "production implementation" | ✅ | Argot roadmap |
| 58 | **Sourcify** is now part of Argot; supports "exact match"; stores verified contracts on IPFS; Blockscout uses it as backend; participates in the Verifier Alliance | ✅ | `sourcify.dev` |
| 59 | "Verification does not mean it's safe to interact with a contract" | ✅ | Sourcify FAQ |
| 60 | **SmartEval** (arXiv:2605.09610, 10 May 2026): 9,000 contracts; five-dimensional rubric; three validation studies; 0.34-point expert alignment; 79.4 % Slither agreement; logic omissions 35.3 %; state-transition errors 23.4 %; +8.29 composite advantage | ✅ | arXiv abstract page |
| 61 | Argot project roster = Act, Ethdebug, Fe, Hevm, Solidity, Sourcify | ✅ | `argot.org` homepage |
| 62 | `hevm` is the symbolic-execution engine **used by Echidna**; the Solidity team uses it to differentially fuzz SSA-CFG | ✅ | Argot roadmap |
| 63 | `--optimize-runs` accepts up to `UINT64_MAX` as of 0.8.36 | ✅ | `Changelog.md` 0.8.36 |
| 64 | GitHub issue **#11690** is titled "Release version 1.0.0" and proposes renaming the current release | ✅ existence; the issue is a proposal, **not** a roadmap commitment |

---

## 11. What Could Not Be Verified

**Nothing in this guide asserts any of the following as fact.**

**Provenance and governance**

- **Gavin Wood's authorship and the August 2014 date.** Every source located is secondary, and Wikipedia tags the sentence `[non-primary source needed]`. The Solidity docs and `Changelog.md` — both read — are silent on it. §1.5 reports it as "the accepted account," not as verified.
- **The early contributors** other than Reitwiessner. Named nowhere in the sources read.
- **The exact date in October 2024** of Argot's formation/announcement. The consolidated post read this pass is dated 30 June 2026; the "hello world"/manifesto post referenced by argot.org was not read directly.
- **Whether any formal legal transfer** of repository ownership, trademark or the `soliditylang.org` domain accompanied the practical move to Argot. The GitHub org move and the redirect are verified; the legal instruments are not.
- **Argot's funding beyond the committed runway.** Argot states the EF treasury is finite and that it is working toward diversified funding. Whether that succeeds is open, and it is a live dependency risk.
- **The 1.0 release plan, or any date for it.** No primary statement of a plan was found. Verified: 1.0 has not shipped, 0.8.38 was unreleased at read time, and deprecation warnings point at a 0.9.0. Issue #11690 is a proposal only.
- **Whether `solcjs` and `solc` are always behaviourally identical** across every release. The docs describe `solc-js` as the emscripten build, but no equivalence guarantee was located.

**Tooling statuses**

- **Mythril's current release cadence.** Repository and history verified; a recent release was not confirmed.
- **Securify's current status.** Not verified. Do not cite as current.
- **Oyente's** status beyond "historical." Not verified.
- **`huff2`'s maturity, maintenance and audit status.** Existence verified; nothing else.
- **Yul+'s current maintenance status.** Not verified.
- **Vyper's *present* funding arrangement and formal stewardship structure.** The historical ENS + EF-matching grant is verified; what funds it *now* is not. The reported ~$219k allocation comparison to Fe is **single-sourced and contested**.
- **Solhint's and `prettier-plugin-solidity`'s current maintenance.** Named as a category; individual statuses not verified.
- **The SmartEval ground-truth dataset's name** — "FSMSCG" in the abstract versus "FSM-SCG" elsewhere. Treated as the same dataset; the discrepancy is unresolved.
- **Whether any bank-scale, independent evaluation of an AI audit tool exists.** None located. Every AI-auditor effectiveness number in §8.2 comes from a single academic study or from vendor marketing.

**Non-EVM language families**

- The current status — maturity, maintenance, adoption, tooling — of **Solana/Rust+Anchor, Sui and Aptos/Move, StarkNet/Cairo, Stacks/Clarity, and Tezos/Michelson**. Their §7.5 entries describe *design-level* characteristics only, and every claim about their present state is flagged. No primary source for those chains was read this pass.

**Compiler and language details**

- **Docs pages that failed to fetch this pass** — notably `internals/layout_in_storage.html`, `security-considerations.html` and the docs index. The storage-layout rules in §5.2 are stated from long-stable documented behaviour and cross-checked against changelog entries that *were* read (`storageLayout` artefact, configurable storage layout, ERC-7201 builtin), but the layout page itself was **not re-read**. Treat §5.2 as ⚠ on sourcing, ✅ on substance.
- **The exact release that introduced `transient` state variables.** Narrowed to "Cancun-era, present by 0.8.28" via the affect range of the 2026 bug (0.8.28–0.8.33), but the introducing release was not pinned.
- **The changelog's full middle** (roughly 0.8.1–0.8.29) was paged rather than read exhaustively; individual patch-release features in that range may be under-reported here.
- **Whether any *other* high-severity compiler bug besides the transient-storage collision affected 2026 0.8.x releases.** Only that one security post was read; `bugs.html` and `bugs_by_version.json` were identified but **not fetched**, so the full advisory list is not reflected here.
- **The Etherscan-specific verification procedure.** Not read; the guide describes the general reproducibility requirement that applies to it.

**Repository/process items named but not fetched**

- `docs.soliditylang.org/en/develop/bugs.html` and `docs/bugs_by_version.json` — identified as the advisory home, **not read**.
- The "Core Solidity Deep Dive" blog post (Nov 2025) referenced by the `solcore` repo — **not read**.
- The EIPs referenced by compiler features (EIP-7843 `SLOTNUM`, ERC-7201) — identified from source text, **not read** at `eips.ethereum.org`.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **ABI (Application Binary Interface)** | The encoding defining how calls and data cross the contract boundary: a 4-byte selector plus ABI-encoded arguments. Specified in the Solidity docs. |
| **`abi.encode` / `abi.encodePacked`** | Standard (padded, unambiguous) versus packed (unpadded, collision-prone with multiple dynamic arguments) encoding. The spec warns against packed for hashing/signatures. |
| **`address` / `address payable`** | A 20-byte address; `address payable` additionally supports `transfer`/`send`. Distinct since 0.5.0. |
| **Argot Collective** | The independent non-profit that now stewards Solidity, Sourcify, hevm, ethdebug, Fe, Act and the Core Solidity research project. Formed October 2024. |
| **`assert` / `require`** | `require` validates inputs and reverts with `Error(string)` or a custom error; since 0.8.0 a failing `assert` reverts with `Panic(0x01)` instead of using the invalid opcode. |
| **C3 linearisation** | The algorithm turning multiple inheritance into a deterministic method-resolution order; the basis of `super` semantics. |
| **`calldata`** | Read-only data location holding an external call's input; canonical for `external` reference-type parameters since 0.5.0. |
| **Core Solidity** | An Argot prototype of "Solidity's new type system," lowering via Yul to the existing `solc` backend. Explicitly not production-ready. |
| **Custom error** | `error Foo(uint256 x);` with `revert Foo(1);` — a gas-efficient, typed revert introduced in **0.8.4**. |
| **Data location** | `storage`, `memory` or `calldata`; mandatory for reference types since 0.5.0. |
| **`delegatecall`** | Executes another contract's code against the **caller's** storage, address and balance; the mechanism behind library calls and proxy patterns. |
| **Echidna** | A property-based fuzzer for contracts; now running on the `hevm` engine. |
| **EOA** | Externally Owned Account — an account with a key, as opposed to a contract. `tx.origin` is always an EOA. |
| **ERC-7201** | A standard for storage-namespaced layouts; `solc` 0.8.35 added a builtin computing an ERC-7201 base slot. |
| **`ethdebug`** | An Argot project defining a smart-contract debugging data format; emitted by `solc` behind the experimental gate. |
| **EVM version / fork target** | The fork the compiler targets (`--evm-version`), determining available opcodes and gas assumptions. |
| **Fe** | An Argot statically typed language for the EVM; first release after a rewrite in H1 2026. |
| **`hevm`** | An Argot symbolic execution engine and testing framework; the engine behind Echidna, also used to differentially fuzz new `solc` codegen paths. |
| **Huff** | A macro-based EVM assembly language from Aztec Protocol, built for Weierstrudel. Its Rust compiler was archived in October 2024. |
| **Immutable** | A state variable assigned once in the constructor and stored **in code**, not storage; consumes no slot. |
| **Interface** | A declaration of an external contract's functions with no implementation. An interface cast performs **no** runtime check on the target. |
| **Layout collision** | The failure in which an upgrade's storage-layout change reinterprets existing data. Prevented by the append-only rule. |
| **`memory`** | A data location for the duration of one external call; deallocated when the call returns. |
| **Metadata** | The CBOR (and JSON) blob appended to the deployed bytecode: compiler version, settings, source hashes. The basis of source-code verification. |
| **Modifier** | A reusable function wrapper using `_;` as the place the body is spliced. |
| **`Panic(uint256)`** | The revert encoding for critical errors since 0.8.0, selector **`0x4e487b71`**, with documented codes such as `0x11` (arithmetic) and `0x32` (out-of-bounds). |
| **Pragma** | The source-level version directive, e.g. `pragma solidity 0.8.37;`. A compatibility claim, not a build instruction. |
| **Reproducible build** | A build whose output can be regenerated byte-for-byte from archived inputs: pinned source, standard-JSON settings and compiler version. |
| **`runs`** | The optimiser's assumed execution count; default **200**. Low `runs` favours cheap deployment, high `runs` cheap execution. |
| **Selector** | The first 4 bytes of `keccak256` of a function's canonical signature. |
| **Slither** | A Python static-analysis framework for Solidity and Vyper, maintained by Trail of Bits / Crytic; a detector suite plus an API. |
| **SMTChecker** | `solc`'s built-in SMT/Horn-based verifier, proving `assert` conditions from `require` assumptions. Engines BMC (being deprecated) and CHC; default disabled. |
| **`solc` / `solc-js`** | The C++ reference compiler and its emscripten build. |
| **Sourcify** | An open-source, non-profit EVM source-verification service, now part of Argot; supports "exact match" and stores artefacts on IPFS. |
| **SSA-CFG** | The experimental single-static-assignment control-flow-graph codegen backend (0.8.35+), aimed at stack-too-deep errors and compile time. Gated behind `--experimental`. |
| **`storage`** | The permanent, priced, slot-based data location; the basis of the layout rules and upgrade-safety discipline. |
| **Storage layout** | The mapping from declared state variables to slot numbers and byte offsets; emitted as the `storageLayout` artefact. |
| **Standard JSON** | `solc --standard-json`: the recommended machine interface, and the reproducibility record. |
| **`super`** | The next contract in the C3 linearisation order — not "the parent class." |
| **SWC / SWC Registry** | The Smart Contract Weakness Classification taxonomy; owned by [smart_contracts_guide.md](smart_contracts_guide.md) §8. |
| **Transient storage** | Storage persisting only for one transaction (Cancun-era `TSTORE`/`TLOAD`), exposed via the `transient` qualifier. |
| **`tx.origin`** | The transaction's originating EOA. Usable for classification; unsafe for authorisation. |
| **`unchecked { }`** | A lexical block restoring wrapping arithmetic; introduced in 0.8.0. Does not propagate into called functions. |
| **Via-IR** | The compilation path Solidity → Yul → Yul-IR → optimised IR → EVM; enabled by `--via-ir` / `settings.viaIR`. |
| **Vyper** | A Pythonic, deliberately minimal EVM language with a subtractive safety philosophy. |
| **Yul** | The low-level intermediate language: inline assembly in Solidity, the via-IR pipeline's input, and a standalone target. |

---

## 13. Cross-References and Further Reading

**Within this repository**

| Guide | Relationship to this guide |
|---|---|
| [smart_contracts_guide.md](smart_contracts_guide.md) | **The primary complement.** Owns: Szabo's concept (§2), the platforms (§3), the EIP-1 standards process (§4), the token standards ERC-20/721/1155/4626 (§5), the development lifecycle and frameworks (§6), **upgradeability and the proxy patterns (§7)**, **the security deep-dive — reentrancy and the vulnerability classes, SWC (§8)**, the incident record (§9), and the tokenized-fund worked example (§11). This guide references all of these and re-derives none |
| [ethereum_guide.md](ethereum_guide.md) | Owns the chain: accounts, gas, the EVM, clients, consensus history, the roadmap eras, EIP-4844, L2s, tokenization. This guide treats gas *pricing* as external and covers only the compiler's gas *modelling* |
| [blockchain_technology_guide.md](blockchain_technology_guide.md) | The ledger fundamentals underneath both |
| [web3_technologies_guide.md](web3_technologies_guide.md) | The web3 application layer above the ABI |
| `ai_llm/` cluster | The LLM-generation, benchmark-contamination and AI-audit context that §8 cross-references |

**Primary sources used (and where to monitor them)**

- **Compiler and language:** `github.com/argotorg/solidity` — in particular **`Changelog.md`** (root, capital C), `SECURITY.md`, `docs/bugs_by_version.json`, and the release tags.
- **Specification:** `docs.soliditylang.org/en/latest/` — the ABI spec (`abi-spec.html`), `using-the-compiler.html` (optimiser, flags, EVM versions, experimental mode), `smtchecker.html`, `types.html`, `contracts.html`, `internals/layout_in_storage.html`.
- **Announcements and advisories:** `soliditylang.org/blog` (releases and the **`security-alerts`** category).
- **Stewardship and roadmap:** `argot.org` and `argot.org/blog` (bi-annual roadmap updates).
- **Verification:** `sourcify.dev` and `docs.sourcify.dev`; `verifieralliance.org`.
- **Tooling:** `github.com/crytic/slither` (Slither), `github.com/ConsenSysDiligence/mythril` (Mythril), `github.com/huff-language/huff-rs` and `huff2` (Huff), `github.com/vyperlang/vyper` (Vyper), `github.com/argotorg/solcore` (Core Solidity), `github.com/argotorg/hevm` (hevm).
- **Research:** arXiv **2605.09610** (SmartEval, 10 May 2026).

---

## 14. Closing Summary

Solidity is a small language in a hard place. Its syntax is unremarkable — curly braces, static types, a familiar shape — but it is the front end to a machine where a state write is permanent, a call is a trust boundary, and a bug is an irreversible transfer. Everything distinctive follows from that: the data-location keywords that price copying, the mapping type that cannot be enumerated, the storage layout that makes an upgrade either safe or silently destructive, the linearisation order that decides what `super` means, and an ABI that has to be right because there is no second chance after deployment.

What this guide established, by reading primary sources, is that this middle layer — the language and its compiler — is exactly where a bank's assumptions break down most quietly. The language has spent a decade moving in one direction: **making the implicit explicit.** Visibility became mandatory in 0.5.0, and so did data locations. `virtual`/`override` and `try`/`catch` arrived in 0.6.0, along with the split of the fallback function. `now` and the old denominations died in 0.7.0. And in 0.8.0 the arithmetic itself became safe by default, with overflow reverting under a documented `Panic(uint256)` protocol and `unchecked` as the explicit opt-out. Each boundary retired a footgun, and the deprecation warnings now pointing at a 0.9.0 show the process is not finished. A bank reviewing a contract that predates those boundaries is not reviewing the same language.

The stewardship finding is the one most likely to have moved since the last guide was written, and it is verified rather than rumoured: **Solidity is no longer an Ethereum Foundation project in the ordinary sense.** The repository is `argotorg/solidity`; the compiler team's roadmap is published by an **independent non-profit, the Argot Collective**, formed in **October 2024** under a five-year Ethereum Foundation commitment specified at an ETH price of USD 2,592. The first three years were paid; the final two sit in a **2-of-3 multi-signature** arrangement with Argot, a **Fail-Safe Committee** and the EF, unlocking in July 2026 and July 2027. That is a stable present and an open future — Argot is candid that the EF's treasury is finite, and its independence is exactly what a long-lived compiler dependency needs a governance answer for.

The compiler itself is not a neutral translator. It emits the metadata that makes verification possible or impossible; it selects a default EVM fork that moves with each release; it offers a legacy path, an IR path and now an experimental SSA-CFG path with materially different risk profiles; and it has shipped **high-severity codegen bugs** — the transient-storage clearing-helper collision that affected `0.8.28`–`0.8.33` under `--via-ir`, invisible at source level, found by an external security firm, fixed within a week in `0.8.34`. Its own policy is blunt: **only the latest release gets security updates.** A pinned older compiler is, by the team's own standard, an accepted risk.

The ecosystem around it is consolidating and thinning at once. **Slither** is healthy. **MythX is sunset.** **Huff's** primary compiler was archived in October 2024. **Vyper** is actively developed and funded precariously. And the steward that took on Solidity is now also building **Core Solidity** — a new type system for the same language, reusing the same backend — which is the development most worth watching for anyone with a Solidity estate.

Then the AI era, where this guide drew a hard line between what is documented and what is merely said. Documented: a 9,000-contract benchmark with three validation studies, whose reported failure modes are **logic omissions and state-transition errors** rather than the classic exploit classes — failures pattern-based analysis does not catch — and a compiler team that handled **around thirty security reports in a single half-year**, with **"an order of magnitude increase"** in report volume attributed to better LLMs. Not evidence: a 79.4 % figure that measures agreement rather than accuracy, and a market of AI auditors whose detection claims carry no methodology. The honest conclusion is that the risk is not obviously-insecure generated code; it is **plausibly secure-looking** generated code that fails where pattern-matching cannot see.

Which is why the discipline in this guide is boring on purpose. Pin the compiler exactly. Record the standard-JSON input. Diff the storage layout and refuse anything but an append. State every data location and every visibility. Justify every `unchecked` and every narrowing cast. Watch `bugs_by_version.json` rather than a GitHub advisories feed. Run the analysers at pinned versions and record their limits. Archive the verification. And know, precisely, what those controls do and do not establish — because the compiler sits inside the audit boundary, and a review that treats it as invisible produces a claim nobody can reproduce.

Read the source, then read the compiler, then read the changelog. That is how a language gets trusted by an institution — and Solidity, for all its footguns, can be trusted exactly to the extent that its compiler, its version and its layout are understood. That is what it means to treat Solidity as **the language of the ledger.**
