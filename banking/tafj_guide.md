# TAFJ: The Temenos Application Framework for Java — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Core Banking / Temenos — the JVM-based runtime engine that executes T24/Transact application code — the Java alternative to the jBASE (TAFC) engine
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Scope and How This Guide Fits the Series

This guide is the **dedicated deep-dive on TAFJ** — the Temenos Application Framework for
Java — the runtime engine that executes T24/Transact application code on the Java Virtual
Machine. It is deliberately *not* a re-hash of the rest of the Temenos series; where a
topic already has a dedicated guide, this document cross-references it by filename:

- Temenos the company, product portfolio, Transact architecture, deployment models → `temenos_guide.md`
- The T24/Transact data model — jBASE/MultiValue records, `$NAU`/`$HIS` companion files, AA arrangements → `temenos_data_model_guide.md`
- The jBASE engine itself (hashed files, jBC, the MultiValue database family) → `../technology/jbase_universe_guide.md`
- The vendor landscape and core-banking eras → `core_banking_systems_guide.md`
- Posting/accounting flow (OFS → FUNDS.TRANSFER → STMT.ENTRY) → `posting_engine_core_banking_guide.md`
- Interest engines (Temenos AA INTEREST property) → `interest_engines_core_banking_guide.md`
- COB/EOD batch processing → `core_banking_processes_guide.md`
- A real Temenos-on-cloud deployment (GLDB on Huawei Cloud) → `green_link_digital_bank_guide.md`
- Banking microservices context (for the "TAFJ + modern stack" discussion) → `oracle_banking_microservices_architecture_guide.md`

**Verification note — read this first.** TAFJ is an *internal Temenos product* with
**severely limited public documentation** — Temenos publishes no full TAFJ architecture
spec. What is known comes from: (1) Temenos training/installation decks that leak into the
public domain (Scribd/PDFCOFFEE/SlideShare copies of "TAFJ Installation Guide",
"Evolution and Architecture of T24", "TAFJ Design Studio User Guide"); (2) the Temenos
partner/community ecosystem (Base Camp forums, t24ALL, Stack Overflow, consultant LinkedIn
posts); (3) job adverts and implementation case studies that name TAFJ explicitly. Where a
fact is verified against such a source it is stated plainly; where it is inferred, hearsay,
or marketing it is **flagged**. Anything flagged "flag" is directionally true but not
independently confirmable.

---

## Table of Contents

1. [What Is TAFJ?](#1-what-is-tafj)
2. [The Architecture](#2-the-architecture)
3. [TAFJ vs jBASE (TAFC): The Engine Comparison](#3-tafj-vs-jbase-tafc-the-engine-comparison)
4. [Components and Development](#4-components-and-development)
5. [Deployment and Operations](#5-deployment-and-operations)
6. [The Architect's Perspective](#6-the-architects-perspective)
7. [Worked Example: Transact on TAFJ](#7-worked-example-transact-on-tafj)
8. [The Future: 2026 and Beyond](#8-the-future-2026-and-beyond)
9. [Glossary](#9-glossary)
10. [References and Verification Trail](#10-references-and-verification-trail)

---

## 1. What Is TAFJ?

### 1.1 The One-Paragraph Answer

**TAFJ — the Temenos Application Framework for Java — is the Java runtime engine for the
Temenos core-banking application.** It is a Pick-BASIC (jBC/InfoBasic) *runtime and
compiler written in 100% Java*: it takes the jBC source code that makes up the T24/Transact
application, parses it, compiles it into equivalent **Java classes**, and executes those
classes on a standard **JVM**. TAFJ is the *alternative engine* to **TAFC** (the Temenos
Application Framework in C, historically the default), which runs the same application code
on the **jBASE** MultiValue database engine. Same application, two engines — this is the
core fact to internalise:

```
                 The Temenos core-banking stack
   ┌─────────────────────────────────────────────────────────┐
   │  TEMENOS TRANSACT (the application — formerly T24)       │
   │  ~4,000+ application modules written in jBC (T24 BASIC)  │
   └──────────────────────────┬──────────────────────────────┘
                              │ jBC source is the SAME for both engines
              ┌───────────────┴────────────────┐
              ▼                                ▼
   ┌─────────────────────┐          ┌──────────────────────────┐
   │ TAFC engine (C)     │          │ TAFJ engine (Java)       │
   │ jBC interpreted     │          │ jBC compiled → Java      │
   │ natively on jBASE   │          │ bytecode, run on the JVM │
   └──────────┬──────────┘          └───────────┬──────────────┘
              ▼                                 ▼
   ┌─────────────────────┐          ┌──────────────────────────┐
   │ jBASE MultiValue DB │          │ Relational DB via JDBC   │
   │ (hashed files)      │          │ (Oracle, MySQL, SQL …)   │
   └─────────────────────┘          └──────────────────────────┘
```

The official expansion — **Temenos Application Framework for "Java"** (Temenos itself
writes it with the quotes, e.g. in the "Temenos Application Framework for JAVA — TAFJ"
training deck) — is consistent across every source that can be checked. The short name
**TAFJ** is what the entire ecosystem (job ads, forums, support tickets, consultant CVs)
uses.

### 1.2 Positioning: Application vs Engine

A persistent source of confusion in the Temenos world is the distinction between the
*application* and the *engine*. The tidy way to think about it:

| Layer | What it is | Examples |
|---|---|---|
| **The application** | The banking product logic — the code | T24 → Temenos Transact (CUSTOMER, ACCOUNT, FUNDS.TRANSFER, the AA modules…) |
| **The engine / framework** | The runtime that executes that code | **TAFC** (on jBASE) or **TAFJ** (on the JVM) |
| **The database** | Where records physically live | jBASE hashed files (TAFC), or Oracle/MySQL/SQL Server (TAFJ) |

So "the bank runs Temenos" is incomplete — the interesting question for an architect is
*which engine*. The jBC source is (almost) engine-agnostic: the same `FUNDS.TRANSFER`
subroutine compiles and runs on both. That is the entire point of the framework design —
Temenos wrote one application and two engines, and has spent the last decade-plus
steering the industry from the C engine to the Java engine.

### 1.3 Naming: T24 vs Transact vs TAFJ

- **T24** — the application's original name. Officially "Temenos 24", where 24 denoted
  *24-hour non-stop processing* (per the "Evolution and Architecture of T24" R10.01
  training deck). It is also glossed as "Temenos for the 24th century" in some Temenos
  marketing; the 24-hour reading is the one in the technical decks.
- **Transact** — the application's modern brand name, adopted in the **2018** rebrand
  ("T24 became Temenos Transact"). Functionally the same product line; the release train
  now runs as Transact R20, R21, R22, R23, R24… (see `temenos_guide.md` §4 for the product
  story). Old hands still say "T24"; the docs increasingly say "Transact". This guide uses
  "T24/Transact" or "Transact" to mean the application, and **TAFJ/TAFC** to mean the engine.
- **TAFJ** — the engine, not a product feature. Nobody "buys TAFJ" standalone: it ships as
  part of the Transact delivery (the TAFJ runtime JARs, the compiler, the Design Studio
  tooling). It is the *Java execution environment*: "The T24 application server with the
  Java runtime environment is known as TAFJ" (T24 R10.01 architecture deck — verified).

### 1.4 History: How TAFJ Came to Be

The timeline below is reconstructed from Temenos training decks, release archaeology, and
case studies. Dates before ~2011 are the least well documented publicly — flagged.

| Year | Event | Source confidence |
|---|---|---|
| 1993 | Temenos founded in Geneva; T24 built and deployed through the 1990s | Verified (see `temenos_guide.md`) |
| ~2001–2005 | T24 architecture evolves: dumb-terminal client/server → GUI desktop client → **R04** web-browser multi-tier (web servers, T24 application servers, DB servers) | Verified (T24 architecture deck) |
| **2007** | Temenos **acquires jBASE**, bringing the underlying MultiValue database in-house — the engine becomes strategic IP, not a third-party dependency | Verified (see `temenos_guide.md` §2) |
| **2008–2010** | **The R10/R10.01 era.** The Java runtime appears: "the T24 application server with the java runtime environment is known as TAFJ". Temenos' Java strategy ("T24 on Java") goes from experiment to roadmap. Early TAFJ builds use JBoss and H2 — open-source Java stack — and JDBC connectivity to Oracle, SQL Server, DB2, Derby. *Flag: the exact first release that shipped TAFJ production-ready is not publicly pinned; R10/R11 (2008–2010) is the commonly cited window in partner materials.* | Flag — directionally verified |
| 2011+ | TAFJ matures through R11/R12; Temenos pushes "Temenos on Java" as the strategic direction; TAFC remains the default for most installs through the early 2010s because most banks and the SI ecosystem had jBASE skills | Flag |
| **2016** | **Jordan Ahli Bank** goes live on T24 with **AA + TAFJ** — a named, verifiable TAFJ production deployment (FinTech Futures, Nov 2016) | Verified |
| 2018 | "T24" rebranded **Temenos Transact**; TAFJ continues as the runtime inside it | Verified (see `temenos_guide.md`) |
| 2018–2020 | TAFJ R14/R16/R18 releases documented publicly (installation guides, Design Studio guides leak into Scribd/PDFCOFFEE). ModelBank R14 ships with a TAFJ layout (`T24/Common/Jars`, compiled classes under `classes/com/temenos/t24`) | Verified (leaked docs) |
| 2021–2023 | Temenos Banking Cloud SaaS (700+ banks) and cloud-native Transact on Kubernetes; new-build deployments overwhelmingly TAFJ | Verified (see `temenos_guide.md` §10, §13) |
| 2024–2026 | Transact R22/R23/R24+ — **TAFJ is the strategic engine**; the Temenos community consensus (t24guru, partner material) is that TAFC is legacy and no longer the investment focus | Flag — community consensus, see §8 |

**The strategic reading (verified direction):** Temenos acquired jBASE in 2007 partly to
*control* the engine while it built the Java replacement — you cannot rebuild the runtime
underneath a live core-banking product unless you own the original. TAFJ was Temenos'
answer to the strategic problems of the 2000s: jBASE skills were scarce and aging,
MultiValue hashed files did not fit bank data-warehouse/reporting estates, and the industry
was consolidating on J2EE application servers. "T24 on Java" was the 2008-era slogan;
TAFJ is the productised result.

### 1.5 Adoption: Who Actually Runs TAFJ

Public, named TAFJ deployments are rare in the press because engine choice is an
implementation detail, not a headline. Verified or strongly documented instances:

- **Jordan Ahli Bank** (Jordan, 2016) — T24 with AA architecture on TAFJ (FinTech Futures). Verified.
- **Green Link Digital Bank (GLDB)** (Singapore, 2022) — Temenos core on **Huawei Cloud** with
  **RDS for MySQL** and Kubernetes (CCE/CCI). The engine is *not* named in any public case
  study, but a MySQL-backed, containerised, cloud-native Transact build is TAFJ by
  construction — TAFC/jBASE cannot use MySQL as its store. *Flag: strong inference, not a
  stated fact* (see `green_link_digital_bank_guide.md` §2.5).
- **Lesha Bank** (Qatar, 2024) — "first in MEA" on **Transact R23**; engine not named, but
  R23-class deployments are TAFJ by default. Flag (see `temenos_guide.md`).
- **The broad middle:** the majority of *new* Transact deals since ~2019 and essentially
  all Banking Cloud SaaS tenants run TAFJ. Legacy TAFC installs (thousands of banks from
  the 2000s–2010s) are being migrated engine-by-engine, or stay on jBASE because the
  cost/benefit of an engine swap has not closed — the "dual-engine reality" of §3.6.

### 1.6 TAFJ in the Documentation: What Exists Publicly

Honest inventory of the public documentation situation:

- **Temenos public site:** near-zero. The developer portal covers APIs/events and the
  extensibility layer; TAFJ internals are not published there.
- **Temenos Base Camp / partner portal:** the real documentation lives here (Design Studio,
  TAFJ installation/logging, deployment guides) — behind login, for licensed partners and
  customers. This is the primary gap an architect hits: **you cannot self-serve TAFJ
  documentation without a Temenos relationship.**
- **Leaked/secondary copies:** TAFJ Installation Guides (R14/R16/R18), "TAFJ Guide for
  Eclipse Users", "TAFJ Logging Documentation R18", "Evolution and Architecture of T24"
  decks circulate on Scribd/PDFCOFFEE/StudyDump — the main public window into the internals.
- **Community:** t24ALL, Stack Overflow (`temenos-tafj`), consultant LinkedIn posts
  (Mathisi Digital's TAFJ training videos are a surprisingly rich public source), and job
  adverts that enumerate TAFJ skills.

**Operational implication (flag, but widely reported):** teams without a Temenos
maintenance/partner contract operate TAFJ with substantially less documentation than
comparable platforms, concentrating knowledge in a thin layer of experienced TAFJ
consultants — see §6.5.

### 1.7 Why TAFJ? The Rationale (and What to Verify)

The documented drivers for Temenos building — and banks choosing — TAFJ:

1. **The Java ecosystem.** One app server, one JVM, one toolchain; integration with the
   J2EE world (EJB/MDB era, then Spring-era services) instead of a proprietary MultiValue
   world. The R04-era architecture already ran the presentation tier on J2EE servers; TAFJ
   completed the Java-isation by bringing the *application server tier* itself onto the JVM.
2. **Relational databases.** Banks want core data in Oracle/MySQL/SQL Server — for
   reporting, BI, regulatory extraction, and DBA skills — not in hashed files only jBASE
   tools can read. TAFJ's JDBC file-system layer delivers that (§2.3).
3. **Cloud-readiness.** A JVM process is containerisable; jBASE's process-per-agent model
   (§1.8) is not. TAFJ is the engine underneath cloud-native/containerised Transact and
   Banking Cloud SaaS.
4. **Skills.** Java developers outnumber jBC developers by orders of magnitude, and Java
   skills are cheaper and more portable. TAFJ lets banks hire Java developers for extension
   work (the "L3 Java" model — §4.3) while core logic remains jBC.
5. **Performance claims.** Temenos has claimed scalability/throughput advantages for TAFJ
   (threads inside one JVM vs one process per agent in TAFC; compiled Java vs interpreted
   jBC). **Flag: no independent, public, apples-to-apples TAFC-vs-TAFJ benchmark exists.**
   The thread-vs-process concurrency model is factual (verified — the "Comparative Analysis
   of Core Banking System — Temenos Application Frameworks" paper), but "TAFJ is faster"
   as a blanket statement is marketing-grade and contested in the field — see §3.4, §5.5.

### 1.8 The Thread vs Process Fact (Verified)

One of the few TAFJ facts with a solid public paper trail (the Comparative Analysis paper
on Commercial Bank of Ethiopia, plus the TAFJ installation decks):

- **TAFC:** each T24 *agent* (a unit of concurrency serving a session/batch job) is a
  **separate OS process**, with jBASE's process-based locking and memory model. 20 agents =
  20 processes.
- **TAFJ:** 20 agents run as **20 threads inside a single JVM** (one JVM = one OS process).
  This is the root cause of most of TAFJ's operational differences — cheaper concurrency,
  shared heap, but also shared-JVM failure modes and GC pauses that TAFC never had (§5.6).

---

## 2. The Architecture

### 2.1 The JVM Runtime

TAFJ is a Java application. At the bottom of the stack:

```
  Transact application code (jBC) ──┐
                                   ▼
  ┌───────────────────────────────────────────────────┐
  │ TAFJ                                               │
  │  ┌──────────────┐   ┌───────────────────────────┐  │
  │  │ TAFJ compiler│──▶│ Generated Java classes    │  │
  │  │ (jBC parser  │   │ (com.temenos.tafj.*,      │  │
  │  │  + javac)    │   │  com.temenos.t24.*)       │  │
  │  └──────────────┘   └───────────┬───────────────┘  │
  │  ┌──────────────────────────────▼────────────────┐  │
  │  │ TAFJ runtime services (jBC API, file systems, │  │
  │  │  locking, transactions, sessions, OFS)        │  │
  │  └──────────────────────────────┬────────────────┘  │
  └─────────────────────────────────┼───────────────────┘
                                    │ JDBC
                                    ▼
  ┌──────────────────────────────────────────────────────┐
  │ JVM (heap, GC, threads) — one JVM hosts many agents  │
  └──────────────────────────────────────────────────────┘
                                    │
                                    ▼
  Relational database (Oracle / MySQL / SQL Server / DB2 / H2)
```

Verified mechanics (TAFJ installation deck "1.TAFJ-Install-Day1Sn1" and the TAFJ Guide for
Eclipse Users):

- **"jBC code is parsed by the TAFJ parser and compiled by the javac compiler and
  translated into equivalent Java classes."** So the pipeline is: jBC source → TAFJ parser
  (produces an intermediate representation) → `javac` (the standard Java compiler) → `.class`
  bytecode → JVM. TAFJ therefore does not need a custom bytecode emitter — it leans on
  javac, which keeps it aligned with whatever JDK Temenos certifies.
- Compiled routines land under `classes/com/temenos/t24/...` (the `com.temenos.t24` package
  tree mirrors the application structure), are packaged into **JARs** (e.g. exported to
  `T24/Common/Jars`), and deployed to the application server. The TAFJ runtime itself lives
  in `com.temenos.tafj.*` packages — e.g. the well-known
  `com.temenos.tafj.common.exception.StopRuntimeException` (TAFJERR-1005 "STOP state")
  surfaces in virtually every TAFJ support forum thread.
- **Runtime, not just compiler:** TAFJ also *runs* the compiled code — it provides the jBC
  runtime semantics (dynamic arrays, `@ID`, `READ/READU/WRITE`, `CALL`, `MATPARSE`, the
  `@`-system variables, etc.) as Java libraries. The 100%-Java claim ("a Pick BASIC
  runtime and compiler, written in 100% java" — the TAFJ overview deck) refers to the
  framework itself, not the generated code.

### 2.2 The "jBC → Java" Translation

The translation layer is the heart of TAFJ. What it does, precisely:

- **The language:** jBC (jBASE BASIC / InfoBasic / "T24 BASIC") — a PICK-family BASIC
  dialect; the same language documented in `../technology/jbase_universe_guide.md` (§3) and
  `temenos_data_model_guide.md` (§2). TAFJ supports the language *as T24 uses it* — Temenos
  states TAFJ was written specifically for T24 needs and all proofs of concept were run on
  T24, so it is not a general-purpose jBC implementation (e.g. it does not target generic
  jBASE application portfolios; it targets Transact).
- **Dynamic arrays are first-class.** The MultiValue record — one string with attribute
  marks (0xFE), value marks (0xFD), sub-value marks (0xFC) — is handled natively by the
  TAFJ runtime library; generated Java code manipulates records through the TAFJ dynamic
  array API rather than raw strings. The MV/SV semantics documented in
  `../technology/jbase_universe_guide.md` §1.3 carry over unchanged, because the *language
  semantics* are preserved even though the *storage* changes.
- **What carries over 1:1:** `READ/READU/WRITE/RELEASE`, `OPEN`, `CALL`, `GOSUB`, `EXECUTE`,
  `SELECT`, file handles, `@`-variables, `CONVERT/CHANGE`, `FIELD`, `MATBUILD/MATPARSE`,
  `CATALOG`/`BASIC` equivalents (in TAFJ the compile step), error trapping (`ON ERROR`),
  and the record-status semantics (unauthorised `$NAU`, history `$HIS` — see
  `temenos_data_model_guide.md` §2.4). Application source that sticks to standard jBC
  compiles on TAFJ with little or no change.
- **What does *not* carry over cleanly (flag):** jBASE-specific system calls that have no
  Java analogue (direct jBASE file handles, jBASE `PERFORM`/shell escapes, `SPOOLER`-level
  constructs, code that pokes at the jBASE environment through `SYSTEM()`/`@`-variables with
  jBASE-specific meanings). Real migrations hit a small tail of these per bank; the
  migration effort discussion is in §3.5.
- **Compile units and packaging:** the TAFJ compiler produces a `.class` per routine, and
  routines are grouped into JARs per module ("BCON packs" in the old BUILD.CONTROL model).
  Deployment granularity is therefore JAR-level: replace a JAR, restart the app server (or
  hot-deploy), rather than re-catalog a routine against jBASE's `BP` source files.

### 2.3 The File-System Layer: MultiValue on Relational Storage

This is the single most important architectural fact about TAFJ:

> **TAFJ's "file systems" are relational databases.** A TAFJ file system is a Java/JDBC
> implementation of the jBASE file interface — `OPEN`, `READ`, `WRITE`, `SELECT`, `DELETE`
> — backed by tables in a relational DB instead of hashed files on disk.

Verified (TAFJ installation/overview decks): TAFJ manages connectivity to
**JDBC-compliant databases, including jBASE (via its JDBC driver), Oracle, Microsoft SQL
Server, DB2, and Derby**, and ships with H2 as the embedded/development database; the
community-standard production mix is **Oracle or MySQL**, with H2 for dev/test and
standalone mode. The decks also mention an **embedded export tool for migrating Pick data
to Oracle** — the ancestor of the migration tooling discussed in §3.5.

**How the mapping works (verified at the level of the TAFJ file-system design, flagged at
the physical-layout level):**

- Each T24 *file* (e.g. `FBNK.CUSTOMER`) maps to a set of relational tables (one table per
  file is the conceptual model; physical layouts — column names, index strategies — vary by
  release and are not publicly documented in detail. *Flag: verify against the installed
  release's DDL.*).
- A MultiValue record is stored as one row keyed by `@ID`; the attribute/value/sub-value
  structure is preserved *inside* the row — either as delimited blobs (the classic "one
  VARCHAR per attribute" or "one CLOB per record" designs) or as explicit MV child tables
  in the more evolved layouts. The **dictionary (DICT/]D) is itself data** — dictionary
  records live in the same relational schema, exactly as they live in `]D` files on jBASE.
- Because the *record semantics* (not the physical format) are what the application sees,
  the same `READ` on `FBNK.CUSTOMER` returns the same dynamic array whether the backend is
  jBASE hashed files (TAFC) or Oracle (TAFJ). The `$NAU`/`$HIS` companion-file logic also
  maps to tables (unauthorised/history versions of records — see
  `temenos_data_model_guide.md` §2.4).
- **Consequence for the data estate:** bank data that used to live in opaque hashed files
  now lives in the bank's relational DB estate — readable by SQL, extractable by the bank's
  BI/regulatory tooling, and manageable by mainstream DBAs. This, more than anything else,
  is what TAFJ sells to bank architects. The flip side — SQL access bypassing application
  semantics is dangerous — is discussed in §6.4.

### 2.4 Transactions and Locking

- **ACID via JDBC/JTA (verified direction, flagged detail):** TAFJ transaction management
  is built on the underlying relational database's transaction semantics, surfaced through
  JDBC (and JTA where the application server provides it — JBoss/WebLogic deployments).
  T24's own transaction model — the "commit at the end of a version/FT run, rollback on
  error" semantics, `TRANSACTION.START`/`TRANSACTION.END`-style jBC verbs, and the OFS
  single-request transaction boundary — is implemented *on top of* DB transactions by the
  TAFJ runtime. *Flag: the exact mapping (savepoints, isolation levels, when TAFJ spans
  multiple DB transactions for one logical T24 transaction) is not publicly documented.*
- **Record locking (verified behaviour, flagged internals):** TAFJ implements jBASE-
  compatible locking — `READU` (read-for-update), `WRITEU`, `RELEASE`, `MATREADU` — but the
  locks live in the **database** (lock rows/records in the DB, or DB row locks on the file
  tables) rather than in jBASE's in-memory/per-process lock tables. The observable contract
  — a second session blocks on a locked record until release or timeout — is preserved;
  the mechanics differ. Lock timeouts and lock-monitoring tooling (the Design Studio lock
  management view) exist precisely because DB-backed locking behaves differently under
  contention (see §5.6).
- **Multi-threaded concurrency:** because agents are threads in one JVM (§1.8), TAFJ
  serialises record access through its own thread-safety layer plus DB row locks; TAFC's
  process-level isolation is replaced by JVM-level coordination. This is the area where
  production TAFJ systems most often need tuning (deadlocks under contention, lock-wait
  timeouts) — see §5.5/§5.6.

### 2.5 Component Map

| Component | Role | Verified? |
|---|---|---|
| **TAFJ Compiler** | Parses jBC, emits Java source/classes via javac; `BASIC`/`COMPILE` equivalents; invoked from Design Studio or command line (`tCompile`/similar) | Verified (decks) |
| **TAFJ Runtime** | jBC runtime semantics in Java: dynamic arrays, file I/O, locking, transactions, sessions, `@`-variables, error model (`TAFJERR-xxxx` codes) | Verified (decks, forums) |
| **TAFJ File Systems** | JDBC storage adapters: Oracle, MySQL, SQL Server, DB2, Derby, H2 (embedded), jBASE (via JDBC driver) | Verified (decks) |
| **TAFJ Security** | User/session authentication, profile handling (the T24 sign-on model), authorisation of records (`$NAU` workflow) — surfaced through the runtime and the app server's security | Partially verified; details not public |
| **TAFJ Services** | OFS (Open Financial Service) listener/processor, web-services exposure (the T24/Transact integration surface — see `posting_engine_core_banking_guide.md` §7.1), event emission for the Integration Framework (e.g. Fiorano T24Inbound connects "to TAFJ to get the events" — verified) | Verified |
| **Design Studio** | The developer IDE (Temenos-customised Eclipse): edit, compile, debug (local/remote), JED record editor, JQL/SQL/OFS editors, lock view, packaging | Verified (Design Studio guides) |
| **TAFJ Ext / L3 Java** | The extension mechanism: custom Java JARs dropped into the TAFJ Ext folder, registered with the app server, callable from jBC | Verified (consultant material) |

### 2.6 The "SAB" Question — Flagged

The task brief asked to verify "SAB — the System Abstraction Layer" as a Temenos/TAFJ
component. **This term could not be verified in any public Temenos or TAFJ source.** Public
Temenos material does not document a component called SAB. The closest real things are:

- TAFJ's **file-system abstraction** (the JDBC-based storage layer described in §2.3) —
  which *is* the layer that abstracts storage away from the application;
- generic "system abstraction layer" literature that has nothing to do with Temenos.

**Conclusion (flag): treat "SAB" as either (a) an internal Temenos term not present in
public documentation, or (b) a mis-remembering of the TAFJ file-system/database abstraction
layer.** If it appears in a Temenos document you hold, it will be in partner-only material;
do not rely on it in architecture conversations without confirming the acronym against the
installed release's docs.

---

## 3. TAFJ vs jBASE (TAFC): The Engine Comparison

### 3.1 The Two Engines, Side by Side

The honest framing: **jBASE (TAFC) is the engine the application was born on; TAFJ is the
engine Temenos is steering the world to.** Both execute the same Transact application code;
they differ in every layer underneath it. The jBASE engine itself is deep-dived in
`../technology/jbase_universe_guide.md`; this section is the head-to-head.

### 3.2 Comparison Table

| Dimension | TAFC (jBASE engine) | TAFJ (Java engine) |
|---|---|---|
| **Runtime** | Native C implementation of the jBC interpreter; jBC is *interpreted* | jBC *compiled* to Java classes via javac, executed on a JVM |
| **Storage** | jBASE MultiValue **hashed files** on the OS filesystem | **Relational DB via JDBC** — Oracle, MySQL, SQL Server, DB2, H2; records mapped to tables (MV structure preserved inside rows) |
| **Concurrency model** | One OS **process per agent**; 20 agents = 20 processes | One **JVM**, agents as **threads**; 20 agents = 20 threads in one process |
| **Locking** | jBASE process-level record locks (in-memory/lock files) | DB-backed record locks (row locks / lock tables) via the file-system layer |
| **Language** | jBC (T24 BASIC) interpreted | jBC compiled to Java bytecode; plus **native Java extensions** (L3 Java, TAFJ Ext) callable from jBC |
| **Platform** | Native binaries per OS — Linux, AIX, Windows builds | Any platform with a certified **JVM** — "platform independence" is a documented TAFJ selling point |
| **Ecosystem** | MultiValue tools (jED, jQL, jBASE BASIC), PICK heritage | The **Java ecosystem**: app servers (JBoss, WebLogic, Tomcat), Maven, Eclipse, JVM monitoring, Spring-era integration |
| **Skills** | jBASE/jBC developers — a shrinking, expensive pool | Java developers — a deep, cheap pool; TAFJ-specific knowledge still scarce (see §5.7) |
| **Cloud-readiness** | Poor: process-per-agent, OS-level state, hard to containerise; jBASE does run on Linux VMs (see `../technology/jbase_universe_guide.md`) | Good: JVM in a container, stateless-ish app tier, Kubernetes-friendly; this is the engine under cloud-native Transact and Banking Cloud SaaS |
| **Reporting/data access** | Data in hashed files — needs jBASE tools or extracts to leave the core | Data in the bank's relational DB — SQL views, BI tooling, DW extracts directly |
| **Performance** | Mature, predictable for the classic model; low overhead per process but heavy process spawn | Claimed better scalability (threads, compiled code); GC pauses and JVM tuning are the new tax; **no public independent benchmark** — see §3.4 |
| **Licensing** | Part of the Temenos licence; jBASE is Temenos-owned since 2007 | Part of the Temenos licence (no separate Java licence); certified JVM/DB versions come from Temenos' support matrix |
| **Maturity** | 30+ years, extremely battle-tested; the installed base | ~15+ years in production (R10-era onwards); strategic, actively developed |
| **Temenos direction** | Legacy/maintenance; community consensus is new investment goes to TAFJ (see §8.1) | Strategic; all new features, cloud work, and performance investment |

### 3.3 Same Application, Different Physics

The practical differences are not cosmetic — they change the operating model (this is the
t24guru point, and it matches every consultant account):

- **Deployment:** TAFC ships/catalogs routines into jBASE source/BINARY files and reloads;
  TAFJ ships **JARs** to the app server (via `tRun T24PackageInstaller` /
  `packageDataInstaller` in the modern toolchain). Release mechanics, rollback, and version
  control change accordingly (Maven/Git for jars vs jBASE source control).
- **Debugging:** TAFC debugging is jBASE `DEBUG`/`JBC`-style; TAFJ debugging is Eclipse
  (Design Studio) breakpoints on generated Java, local or remote (`tDebug`), which behaves
  differently — you debug bytecode that *corresponds to* jBC, not the jBC directly.
- **Monitoring:** TAFC monitoring is process-level (ps, jBASE agent status); TAFJ monitoring
  is JVM-level (heap, GC, thread dumps) plus DB-level (connections, locks). The skill set
  shifts from "jBASE admin" to "Java + DB ops" (see §5.7).
- **Failure modes:** a TAFC agent crash kills one process; a TAFJ JVM crash (or a
  long GC pause) takes out *all* agents on that JVM at once. Availability architecture must
  account for this (multiple JVMs, careful heap sizing).

### 3.4 Performance: The Honest Position

- **The claims:** Temenos marketing and TAFJ training material claim better resource
  utilisation (threads vs processes), faster execution (compiled vs interpreted), and
  better scalability for large online workloads.
- **What is verifiable:** the thread-vs-process concurrency model (§1.8);
  compiled-Java-vs-interpreted-jBC as an architectural difference (§2.1); and general
  Java-platform advantages (JIT, mature GC, huge ecosystem).
- **What is not verifiable:** **no independent, public, apples-to-apples TAFC-vs-TAFJ
  benchmark exists** — no Temenos-published benchmark with methodology, no third-party lab
  test, no bank-published numbers. The Comparative Analysis paper (Ethiopia CBE) discusses
  the architectures but reports no comparative throughput figures. Field reports are mixed:
  TAFJ is generally considered competitive and better at *large multi-core scale-out*,
  while TAFC installs are often cited as having lower per-transaction overhead on
  small/medium loads and zero GC risk.
- **The honest verdict (flag):** treat "TAFJ is faster" as *directionally plausible,
  workload-dependent, and unproven publicly*. Architect the performance conversation around
  the bank's own load tests, not vendor claims.

### 3.5 Migration: jBASE → TAFJ

**The migration path** (verified as a real, recurring project type — "TAFC-to-TAFJ
migration" is a named specialism in job adverts, and the "How to transition from TAFC to
TAFJ" training videos exist):

1. **Code translation:** the *same jBC source* is recompiled — the compiler is TAFJ, not
   jBASE. In practice: fix the compatibility tail (jBASE-specific system calls, direct
   environment access, spooler/file-handle constructs), recompile all routines, package
   into JARs, redeploy. Most banks report 90%+ of routines compile with no change; the
   tail is where the schedule risk lives. *Flag: the 90% figure is consultant folklore,
   not a published Temenos number.*
2. **Data migration:** the hard part. MultiValue data must be exported from jBASE hashed
   files and loaded into the relational schema — record by record, preserving `@ID`,
   attribute/value/sub-value structure, `$NAU`/`$HIS` versions, and the dictionary. Temenos
   provides tooling (the embedded Pick→Oracle export in early TAFJ; `tRun`-family loaders
   later; TAFC-era data utilities for extract/transform), but every bank ends up writing
   validation and reconciliation scripts. The relational schema itself is
   Temenos-defined per release — banks do not design it.
3. **Parallel runs and cutover:** the classic core-replacement pattern (see
   `core_banking_processes_guide.md` for the batch/EOD cutover mechanics): run TAFC and
   TAFJ in parallel during user-acceptance testing, reconcile balances, then cut over at an
   EOD boundary.
4. **Migration effort (flag honestly):** Temenos does not publish effort figures. Industry
   accounts put a TAFC→TAFJ engine migration for a mid-size bank in the *multi-quarter to
   multi-year* range, driven mainly by data migration, customisation re-validation, and
   retraining — not by the code recompile. For banks on heavily customised jBASE code, the
   compatibility tail and re-testing dominate. For vanilla banks, the engine swap is mostly
   a data-and-ops project.

### 3.6 The Dual-Engine Reality

**Verified direction:** Temenos has run both engines in parallel for ~15 years and still
does. The reality on the ground:

- **New deals and SaaS:** TAFJ (all cloud/SaaS tenants, nearly all new-build on-prem since
  ~2019 — flag: strong industry consensus, no published ratio).
- **The legacy base:** thousands of banks still on TAFC/jBASE, many with no migration
  plan; Temenos continues to support and patch TAFC for them (it is licensed, documented,
  and maintained — it is "legacy", not "dead").
- **Mixed estates:** some banks run TAFC and TAFJ *in the same environment* (e.g. TAFJ for
  online, TAFC for batch, or one country/entity on each) during migration windows. Temenos
  supports this in the sense that both engines ship in the same release; cross-engine data
  sharing is a migration-phase pattern, not a long-term architecture.

### 3.7 The Engine Decision

For a bank choosing today, the decision factors (framework in §6.2):

| Factor | Leans TAFJ | Leans TAFC/jBASE |
|---|---|---|
| DB strategy | Oracle/MySQL/SQL Server central to the estate | jBASE acceptable as the store; no relational requirement |
| Skills | Java-heavy teams, scarce jBC | Existing jBASE team, no Java capacity |
| Cloud | Cloud/SaaS/K8s target | On-prem VM estate, no containerisation plans |
| Temenos roadmap | New features, new releases, SaaS | Staying on a locked legacy release |
| Risk appetite | Willing to manage JVM/DB ops complexity | Zero-change culture, proven platform preferred |
| Customisation depth | Vanilla-ish, or Java-extensible future | Deep jBC customisation, no appetite to re-test |

The industry-default answer in 2026 is TAFJ; the honest answer for a specific bank depends
on that table.

---

## 4. Components and Development

### 4.1 The TAFJ Compiler

- **What it is:** the jBC→Java compiler (§2.1). Invoked from Design Studio (compile a
  routine/file), from the T24 command line (the `BASIC`-equivalent, historically `tCompile`
  / the compiler component named in config like `temn.tafj.compiler.*`), or as part of the
  build pipeline (Maven `t24-packager-tafj` project).
- **What it produces:** one `.class` per routine under `classes/com/temenos/t24/...`,
  packaged into module JARs. Compile-time error messages surface as TAFJ/Java errors with
  line mappings back to the jBC source (Design Studio shows both).
- **Compile-time options** include debug info (for remote debugging), optimisation flags,
  and the generation of Java *source* (visible in early versions' "generate .java then
  javac" flow — verified in the TAFJ Eclipse guide).
- **Relationship to jBASE compilation (cross-ref):** on TAFC, `BASIC` compiles jBC to
  jBASE's own object format stored in BINARY files; on TAFJ, the same verb compiles to Java
  classes. `CATALOG`/`BUILD.CONTROL` on TAFC becomes JAR packaging/deployment on TAFJ
  (BUILD.CONTROL still exists but the modern toolchain is Maven + `tRun` installers —
  verified via the Stack Overflow automation thread).

### 4.2 The TAFJ Runtime

The runtime provides, in Java:

- **jBC language semantics** — dynamic arrays, string verbs, `@`-variables, error handling
  (`ON ERROR`, `StopRuntimeException`/`TAFJERR-xxxx` codes surface in logs — verified via
  t24ALL forum threads), `CALL`/subroutine dispatch across JARs.
- **File I/O** — `OPEN/READ/READU/WRITE/WRITEU/DELETE/SELECT` against the JDBC file systems
  (§2.3), including the `$NAU`/`$HIS` companion-file behaviour.
- **Locking** — jBASE-compatible `READU` semantics over DB locks (§2.4); lock timeout
  configuration; exposed in Design Studio's lock-management view.
- **Transactions** — the T24 transaction boundary mapped onto DB/JTA transactions (§2.4).
- **Sessions** — the T24 sign-on/session model (user profiles, `T24`-session state,
  terminal/session context per agent thread).
- **OFS and services** — the OFS engine (inbound request processing, outbound responses),
  web-services and Integration-Framework hooks (see `posting_engine_core_banking_guide.md`
  §7.1 for the OFS posting path; Fiorano's T24Inbound is a verified external component that
  "connects to TAFJ to get the events").

### 4.3 Development Models: jBC, L3 Java, and TAFJ Ext

Three ways code gets written on a TAFJ system:

1. **jBC (the mainstream).** Same language as always, compiled to Java. The bulk of
   Transact customisation (routines, subroutines, versions, enquiries, `INPUT.ROUTINE`s)
   still happens in jBC. "You can develop T24 in Java, but you don't have to" — the jBC
   path is the compatibility story that makes TAFJ a drop-in engine.
2. **L3 Java (verified as a real, named model).** "L3" is the Temenos term (used in the
   Extensibility Framework documentation and consultant material) for *writing application
   logic directly in Java* against the TAFJ runtime API — e.g. a Java `INPUT.ROUTINE` or a
   Java subroutine that manipulates records through the runtime's dynamic-array API, then
   gets called from jBC or OFS like any routine. LinkedIn consultant posts explicitly
   discuss "T24 TAFJ L3 Java" and "multi-value field handling in L3 Java" — this is the
   model for banks that want Java developers on the core.
3. **TAFJ Ext (verified).** The mechanism for *external* Java integration: write a Java
   class, compile, drop the JAR into the **TAFJ Ext folder**, register it with the
   application server, and call its methods from InfoBasic (jBC) — the documented pattern
   for "T24 has the ability to talk with Java" (consultant walkthroughs on LinkedIn).
   Typical uses: JSON parsing, calling external REST/SOAP services, complex formatting,
   anything the jBC ecosystem lacks.

**Temenos Extensibility Framework (cross-ref):** the modern, API-first extensibility story
(developer.temenos.com, Design Framework) sits *on top of* TAFJ — banks can also build
extensions as external services talking to Transact via APIs/OFS rather than inside the
runtime. See `temenos_guide.md` §12 and the `oracle_banking_microservices_architecture_guide.md`
context for where that fits.

### 4.4 Tooling

| Tool | What it is | Verified? |
|---|---|---|
| **Design Studio** | The TAFJ IDE — Temenos' customised **Eclipse** distribution (verified: "Temenos Customized Eclipse IDE"; the TAFJ Guide for Eclipse Users; plugins shipped under `tafj/eclipse/plugins`). Edit/compile/debug jBC and Java; local and remote debugging; JED record editor (edit records in the DB), lock management view, JQL/SQL/OFS editors | Verified |
| **tRun / tShow / tDebug (t-commands)** | Run/show/debug commands for TAFJ routines from the command line | Verified (TAFJ Eclipse guide) |
| **BUILD.CONTROL** | Legacy packaging/deployment application (compile + package modules) — still present; the modern path is Maven + installers | Verified (SO thread) |
| **Maven `t24-packager-tafj`** | The build project template in Design Studio for packaging customisations into deployable JARs | Verified (SO thread) |
| **`tRun T24PackageInstaller` / `tRun packageDataInstaller`** | The modern deployment utilities — install packaged JARs into the running TAFJ environment | Verified (SO thread) |
| **JED wizard / JQL-SQL-OFS editor** | Data editing (JED), and a combined query editor (jBASE-style JQL, SQL against the relational schema, OFS messages) | Verified (Design Studio docs) |
| **UTF (Temenos Unit Testing Framework)** | Unit testing framework for T24/TAFJ code (training videos exist; details partner-only) | Flag — existence verified, depth not |
| **Temenos Workbench** | The newer full-stack IDE (Eclipse-based) that supersedes/augments Design Studio for Transact development in current releases | Flag — seen in job ads/docs; verify against installed release |

### 4.5 Certification and Supported Platforms

The certified matrix is Temenos partner material (not public). What can be verified from
public sources, with flags:

- **OS:** Linux (RHEL/OEL-family is the common production baseline — the "T24 Reference
  Architecture on Oracle OEL" deck is public) and AIX historically; Windows for dev.
  *Flag: exact certified versions are in the Temenos support matrix, not public.*
- **Databases:** Oracle and Microsoft SQL Server (verified in the TAFJ overview deck's JDBC
  list), DB2 and Derby (same source), **MySQL** (the de-facto standard for newer/cloud
  builds — GLDB runs RDS for MySQL; flag: not in the older public decks), **H2** (embedded,
  for standalone/dev — verified). jBASE itself also has a JDBC driver and can serve as a
  TAFJ backend (verified mention).
- **JVM:** a certified JDK (OpenJDK/Oracle JDK depending on release era; the modern
  Transact releases certify current LTS JDKs). *Flag: exact JDK versions are in the
  release-specific installation guides (partner-only).*
- **Application servers:** **JBoss** (the classic TAFJ app server through R14–R18 era —
  verified across installation guides and the "TAFJ R18 on JBoss" Stack Overflow threads),
  **Oracle WebLogic** (verified in the T24 Reference Architecture on Oracle/OEL deck —
  "All the above components can be deployed on Oracle WebLogic Application Server"),
  and **standalone mode** (TAFJ-Standalone with embedded H2 — verified config files
  exist: `temn.tafj.compiler.tafc.component.dir` etc.). Tomcat and WebSphere are mentioned
  in consultant material for specific eras/regions but could not be verified against a
  primary source — *flag: verify against the release's certification matrix.* In the
  containerised/cloud era the app server recedes: Transact on Kubernetes runs the TAFJ
  runtime inside containers (see §5.2).

---

## 5. Deployment and Operations

### 5.1 Deployment Models

- **Classic on-prem (J2EE era, R10–R18):** JBoss or WebLogic application server hosting the
  TAFJ runtime; Oracle/SQL Server database; T24 Browser / Design Studio clients; COB
  scheduler for batch. This is the architecture in the public T24 Reference Architecture
  (Oracle/OEL) deck.
- **Modern on-prem / private cloud:** Transact R20+ on Linux VMs or containers, TAFJ
  runtime in the app tier, Oracle or MySQL, batch orchestration via the standard COB
  scheduler or cloud schedulers. See `temenos_guide.md` §13 for the deployment-model
  taxonomy.
- **Public cloud / SaaS:** Temenos Banking Cloud (700+ banks — verified figure in
  `temenos_guide.md`) runs Transact on TAFJ in the cloud; the GLDB pattern
  (`green_link_digital_bank_guide.md` §2.5) is the public reference for
  Temenos-on-hyperscaler: ECS + Kubernetes (CCE/CCI) + RDS MySQL, intra-city active-active.
- **Containers/Kubernetes:** the cloud-native Transact stack packages the TAFJ runtime in
  containers (stateful app tier, DB outside the pod or managed). This is where TAFJ's
  thread-in-JVM model pays off — one JVM per pod, horizontal scale-out by adding pods.

### 5.2 Deployment Components

```
  ┌──────────────┐   ┌───────────────────────────────┐   ┌──────────────┐
  │ Transact Web │──▶│ TAFJ application server       │──▶│ RDBMS        │
  │ (T24 Browser  │   │ (JBoss/WebLogic or container │   │ Oracle/MySQL │
  │  → modern     │   │  hosting the TAFJ runtime,   │   │ SQL Server   │
  │  Transact UI) │   │  agent threads, OFS, COB)    │   │              │
  └──────────────┘   └───────────────┬───────────────┘   └──────────────┘
  External systems ── OFS / APIs ────┘ (JDBC pools)
```

Web layer: the **T24 Browser** (classic HTML UI) evolved into the modern **Transact UI**
(and the newer Transact web/API-first channels); the integration surface is OFS + REST/SOAP
APIs. All of it terminates in the TAFJ runtime, which owns the business logic and the DB.

### 5.3 Operations: JVM and DB Tuning

- **JVM tuning (the new ops discipline TAFJ introduced):** heap sizing (the TAFJ JVM hosts
  all agents — heap too small → OOM; too large → long GC pauses), GC selection (G1 is the
  default-era choice; ZGC/Shenandoah for large-heap low-pause environments), thread pools
  (agent threads vs app-server worker threads), and `-X` flags per the Temenos sizing guide
  (partner material). A GC pause freezes *every* agent on the JVM — this is TAFJ's
  signature operational risk.
- **DB tuning:** connection pools (JDBC pool exhaustion is a classic TAFJ incident —
  see §5.6), index strategy on the TAFJ-generated schema, lock contention on hot files
  (e.g. `FBNK.STMT.ENTRY` at EOD), transaction log sizing, and the read-replica pattern
  (GLDB runs primary + up to 5 read replicas — `green_link_digital_bank_guide.md` §2.5).
- **Monitoring:** TAFJ logs (the R18 logging configuration is one of the few public TAFJ
  docs), JVM metrics (heap, GC, threads — any APM), DB metrics (connections, locks, slow
  SQL), plus the T24-level monitoring (user sessions, OFS queues, batch jobs). There is no
  public "TAFJ metrics API" spec — monitoring is built from JVM + DB + app logs (flag).

### 5.4 Batch: COB and EOD on TAFJ

The COB/EOD mechanics (COB = Close Of Business; the EOD batch chain: pre-COB, COB, post-COB,
data extraction, report generation) are documented in `core_banking_processes_guide.md`.
TAFJ-specific notes:

- Batch jobs are jBC (compiled to Java) run by the COB scheduler as TAFJ agents — i.e.
  threads in the batch JVM(s). Batch and online are typically **separate JVMs** so an EOD
  GC pause or OOM does not take online down.
- Batch performance on TAFJ is dominated by DB throughput (SQL writes per STMT.ENTRY,
  index maintenance) — the classic TAFC batch-timing analysis shifts from "hashed file I/O"
  to "DB write amplification". Flag: banks report batch windows on TAFJ being DB-bound;
  this matches the architecture but no public benchmark exists.

### 5.5 Production Performance Reality (Flag)

- Throughput/latency claims: no public benchmarks (§3.4). Field consensus (flag): TAFJ
  handles high online concurrency well when JVM and DB are tuned; the first production
  incidents are almost always GC pauses, pool exhaustion, or lock contention rather than
  application defects.
- The marketing-vs-reality gap is real: Temenos' "cloud-native, scalable" messaging is
  directionally true (containerised TAFJ scales horizontally), but the *core engine remains
  a shared, stateful application* — it is not magically stateless microservices (this is
  the `temenos_guide.md` §12 caveat).

### 5.6 Troubleshooting: The Common Issues

| Symptom | Root cause (typical) | Fix direction |
|---|---|---|
| `OutOfMemoryError` / JVM crash | Heap sized too small for the agent thread count; leak in custom Java (L3/Ext) | Heap audit, thread-count review, dump analysis |
| Long pauses / timeouts at peak | GC pauses on a large shared heap | GC tuning (G1/ZGC), split JVMs, reduce heap |
| `Connection pool exhausted` | JDBC pool smaller than agent concurrency × per-transaction connections | Pool sizing, connection-leak audit in custom code |
| `TAFJERR-1005`/StopRuntimeException | Routine hitting a STOP/abend path (normal T24 error model surfacing as Java exception) | Read the OFS/error context; it is the *expected* error model, not a crash |
| Lock timeouts / deadlocks | DB row locks under contention; long transactions holding locks | Transaction boundary review, lock timeout config, DB lock monitoring |
| Slow batch at EOD | Index/maintenance on hot tables; lock contention with online JVM | Batch window isolation, index tuning, read replicas |
| `ClassNotFoundException` on deploy | JAR not installed/registered with app server (TAFJ Ext rules) | Re-run installer (`tRun T24PackageInstaller`), check JAR layout |

### 5.7 The Operational Reality: Who Runs TAFJ

- **The ops team is Java + DB, not jBASE.** Production support needs JVM tuning, thread
  dumps, GC logs, JDBC pool management, SQL — a different profile from the TAFC-era
  jBASE admin (job adverts for "T24 Support Engineer" now list TAFJ + Java + JBoss/application
  servers explicitly — verified).
- **Temenos skills still dominate the application layer:** knowing T24/Transact (versions,
  OFS, AA, COB) is the core skill; TAFJ knowledge is a *second* specialism layered on top.
  The scarce profile is "T24 functional/devel knowledge + JVM/DB ops" — that combination is
  what migration and production-support projects pay for.
- **Documentation dependency:** without partner-portal access, TAFJ ops is largely
  tribal knowledge + leaked docs + the community (see §1.6). Budget for a Temenos
  maintenance/partner contract or experienced TAFJ consultants on the team.

---

## 6. The Architect's Perspective

### 6.1 TAFJ as the Core Runtime

In the bank landscape (`core_banking_systems_guide.md` for the vendor table), TAFJ is the
execution engine inside the Temenos core system of record. Placement consequences:

- **Statefulness:** the core engine is shared and stateful; TAFJ does not change that. It
  changes the *shape* of the runtime (JVM, relational store) but not the architectural
  contract: the core owns accounts/ledger, everything else integrates around it.
- **The DB is now part of the bank's estate:** the TAFJ relational store is bank-owned data
  in bank-managed DBs — a governance change (who can run SQL against the core? what is
  read-only vs write path?) that TAFC's hashed files never raised.

### 6.2 The TAFJ Decision Framework

For new deployments or engine migrations, the decision factors (detailed table in §3.7):

1. **DB strategy** — relational-first estate → TAFJ; jBASE-acceptable estate → TAFC.
2. **Skills** — Java pool available for L3/Ext/ops → TAFJ; deep jBC bench with no Java →
   TAFC or a managed migration.
3. **Cloud** — any cloud/SaaS/container target → TAFJ (non-negotiable in practice).
4. **Temenos roadmap** — new releases, new features, SaaS → TAFJ; frozen legacy release →
   TAFC stays viable.
5. **Customisation load** — heavy jBC customisation means migration cost is re-testing and
   the compatibility tail; vanilla means it is mostly a data/ops project.

### 6.3 Integration Architecture

- **OFS** remains the transactional integration protocol (see
  `posting_engine_core_banking_guide.md` §7.1 for the OFS → FUNDS.TRANSFER → STMT.ENTRY
  path); on TAFJ it is implemented in Java inside the runtime, exposed over the app server.
- **APIs:** the modern Transact API/Integration Framework (developer.temenos.com) exposes
  REST services over the same business logic; event emission (Temenos Integration
  Framework, consumed by e.g. Fiorano T24Inbound) feeds event-streaming estates — see the
  streaming guides in the series for the surrounding pattern.
- **Patterns:** channel → API gateway/OFS → TAFJ runtime → DB, with the DB also feeding
  analytics via read replicas (the GLDB pattern). Event-driven integration around the core
  (not inside it) is the sane architecture — see
  `oracle_banking_microservices_architecture_guide.md` for the banking microservices
  context and why the core stays monolithic-ish.

### 6.4 TAFJ + the Modern Stack; Data Access

- **Java ecosystem:** Spring-era services, Kafka/event streaming, APM, container platforms
  all integrate with a JVM core far more naturally than with a MultiValue engine — this is
  TAFJ's strategic win. The *application layer* (Transact) still runs on its own runtime,
  not Spring — the Java ecosystem is what surrounds it.
- **Data access:** TAFJ data is SQL-accessible — build reporting/DW/regulatory extraction
  on SQL views over the TAFJ schema (Temenos Data Hub does exactly this; see
  `temenos_data_model_guide.md` and the data guides). **Danger:** writing to core tables
  via SQL bypasses application semantics (record statuses, `$NAU`/`$HIS`, AA postings) —
  reads yes, writes through the application.

### 6.5 TAFJ Risks — the Honest List

1. **Vendor dependency:** the engine, the schema, the tooling, and the documentation are
   all Temenos-only; TAFJ skills are not portable to other cores. Lock-in is total — but
   that was equally true of TAFC.
2. **Limited documentation:** public docs are thin (§1.6); you operate on partner-portal
   access and community knowledge. Budget for the partner contract and experienced staff.
3. **Skills concentration:** the T24 + JVM/DB combination is scarce; key-person risk on
   migration and support is real.
4. **Performance claims unproven publicly** (§3.4) — validate with your own load tests.
5. **Stateful core in a cloud-native world:** containerising TAFJ is easy; making the core
   stateless/elastic is not. Expectations must be managed against the marketing.

### 6.6 Architect's Checklist

- [ ] Engine decision made explicitly (TAFJ vs TAFC) with the §6.2 factors, not by default
- [ ] DB platform chosen and certified against the Temenos release's support matrix
- [ ] JDK/JVM version and GC strategy validated with a proof-of-concept load test
- [ ] Data migration path designed (extract, transform, load, reconcile) — jBASE → relational
- [ ] Customisation compatibility audit (jBC tail, L3/Ext Java) completed
- [ ] Ops readiness: JVM/DB monitoring, pool sizing, lock timeout config, batch JVM isolation
- [ ] Documentation access confirmed (partner portal) and key-person coverage for TAFJ skills
- [ ] Reporting/DW access strategy defined (SQL views via read replicas; writes via application)

---

## 7. Worked Example: Transact on TAFJ

### 7.1 The Environment

A representative mid-size bank running **Temenos Transact R22 on TAFJ**:

- **App tier:** two Linux (RHEL) servers running the TAFJ runtime on JBoss (or containers
  on Kubernetes); online JVM + separate batch JVM.
- **DB:** Oracle 19c (or RDS MySQL) — one schema holding the TAFJ file tables, dictionary
  records, lock tables; read replicas for reporting.
- **Web:** Transact UI (browser) + OFS/REST API endpoints for channels.
- **Client tooling:** Design Studio (Eclipse) for development; Maven `t24-packager-tafj`
  for packaging; `tRun` installers for deployment.

### 7.2 The Transaction Flow (Online)

```
  User login ──▶ Transact UI ──▶ app server ──▶ TAFJ runtime (agent thread)
     │                                              │
     │   FUNDS.TRANSFER version submitted            ▼
     │   (or OFS: FT,AMOUNT=1000,...)          jBC routine (compiled Java class)
     │                                              │
     └──────────────────────────────────────────────▼
        Oracle: read FBNK.ACCOUNT/FBNK.CUSTOMER, lock (READU), write $NAU,
        post STMT.ENTRY rows, COMMIT
```

1. Login authenticates the session (TAFJ security/user profile) on the JVM.
2. The version/FT processing runs as a compiled Java class — dynamic arrays built from DB
   rows, business rules in jBC semantics, `READU` taking DB row locks.
3. On success the transaction COMMITs (one DB transaction boundary per logical T24
   transaction); the `$NAU` (unauthorised) record is written for the authorisation step
   (see `temenos_data_model_guide.md` for record-status semantics).
4. The UI/OFS caller gets the standard response; the Integration Framework may emit an
   event for downstream systems.

### 7.3 The EOD "Day in the Life"

- Pre-COB: freeze inputs (OFS queues drained, channels closed per cut-off).
- COB: the batch JVM runs the EOD chain (interest accrual/postings, AA activity, fee runs)
   — all compiled jBC, writing via JDBC; STMT.ENTRY and account files are the hot tables.
- Post-COB: data extraction to the DW (SQL against read replicas), reports, regulatory
   extracts.
- Operational watchpoints: batch window (DB-bound), lock contention with the (separate)
   online JVM, GC pauses during the heavy phases.

### 7.4 A jBASE→TAFJ Migration Sketch

1. **Assess:** inventory customisation; audit the jBC compatibility tail; size data.
2. **Prototype:** stand up TAFJ + Oracle; compile the standard release (no change); compile
   customisation; fix the tail.
3. **Data migration:** export jBASE records (including `$NAU`/`$HIS`, dictionaries), load
   into the TAFJ relational schema, reconcile counts and balances.
4. **Parallel run:** run TAFC (production) and TAFJ (UAT) in parallel; replay transactions;
   reconcile.
5. **Cutover:** at an EOD boundary, switch channels/OFS to TAFJ; keep TAFC available for
   rollback until stability is proven.
6. **Risks (honest):** the compatibility tail surprises; data migration reconciliation
   drags; JVM/DB tuning takes longer than expected; retraining the ops team. Mitigate with
   the §6.6 checklist and Temenos/partner support engaged early.

---

## 8. The Future: 2026 and Beyond

### 8.1 Temenos Engine Strategy (Verified Direction, Flagged Details)

- **TAFJ is the primary engine.** Community and partner consensus is unambiguous: new
  investment — features, performance, cloud — goes to TAFJ; TAFC is maintained, not
  advanced (t24guru's "TAFC is not where Temenos is investing" reflects the field's view).
- **jBASE phase-out?** Temenos has announced no end-of-life date for TAFC/jBASE support
  (flag: none found in public sources), and the installed base is huge — support will
  continue for years. But the *strategic* direction is "Transact on TAFJ", and new/SaaS
  deals are TAFJ by default. See `temenos_guide.md` §22 for the product-level outlook.

### 8.2 Cloud-Native and SaaS

- TAFJ is the engine inside containerised Transact and Banking Cloud SaaS (700+ banks).
  Expect deeper Kubernetes integration, managed DB backends, and platform-as-a-service
  ops (auto-scaling of JVM pods, managed MySQL/Oracle).
- The stateful-core constraint stays: cloud-native packaging, not microservices
  decomposition, is the honest trajectory (see `temenos_guide.md` §12, and
  `oracle_banking_microservices_architecture_guide.md` for the contrast).

### 8.3 AI and the Engine

- TAFJ's Java substrate is what lets Temenos bolt AI onto Transact (the AI/ML features in
  current releases call Java services around the core). Expect: AI-assisted development
  tooling in Design Studio/Workbench, AI over core data via the SQL-accessible store, and
  the Integration Framework feeding event streams to AI/analytics — see the AI guides in
  the series for the surrounding stack.

### 8.4 Trends Summary

| Trend | Direction |
|---|---|
| Engine mix | TAFJ everywhere new; TAFC a long, supported tail |
| Ops model | JVM+DB ops replaces jBASE admin; SRE-style monitoring |
| Deployment | Containers/K8s/SaaS standard; on-prem J2EE legacy-era shrinking |
| Data | Core data fully relational; SQL/DW access normalised |
| Skills | TAFJ + Java + T24 hybrid profile becomes the standard hiring spec |

---

## 9. Glossary

- **TAFJ** — Temenos Application Framework for Java: the JVM-based runtime+compiler that executes T24/Transact jBC code as compiled Java classes; the alternative to the TAFC/jBASE engine.
- **Temenos Application Framework for Java** — the official (long) name of TAFJ.
- **TAFC** — Temenos Application Framework (in C): the classic engine running jBC interpreted on jBASE.
- **Transact** — the modern brand name of the T24 core-banking application (rebranded 2018).
- **T24** — the application's classic name ("Temenos 24"; 24-hour processing); still used interchangeably with Transact.
- **jBASE** — the MultiValue database engine (Temenos-owned since 2007) that hosts TAFC (see `../technology/jbase_universe_guide.md`).
- **jBC** — jBASE BASIC / "T24 BASIC": the PICK-family language the Transact application is written in.
- **BASIC** — the language family; the T24 compile verb ("compile the BASIC").
- **MultiValue (MV)** — the data model: records as dynamic arrays with attribute/value/sub-value delimiters (see `temenos_data_model_guide.md`, `../technology/jbase_universe_guide.md`).
- **JVM** — Java Virtual Machine: executes the compiled TAFJ classes; one JVM hosts many agent threads.
- **Java / bytecode** — the language and the `.class` instruction format TAFJ compiles jBC into (via javac).
- **OFS** — Open Financial Service(s): Temenos' message-based transaction/integration protocol (see `posting_engine_core_banking_guide.md`).
- **SAB / System Abstraction Layer** — term from the brief; **unverifiable in public Temenos sources** — likely refers to TAFJ's JDBC file-system/storage abstraction (§2.6).
- **MySQL / Oracle / SQL Server / DB2 / H2** — relational backends supported by the TAFJ JDBC file systems (Oracle/MySQL production-standard; H2 embedded/dev).
- **JTA / JDBC** — Java transaction API / Java database connectivity: the transaction and DB-access mechanisms TAFJ's ACID handling is built on.
- **ACID** — atomicity/consistency/isolation/durability: guaranteed by the underlying DB transactions in TAFJ.
- **Record locking** — jBASE-compatible `READU` semantics implemented over DB locks in TAFJ.
- **Migration** — moving an engine/install from TAFC (jBASE) to TAFJ: code recompile + data migration + parallel run + cutover.
- **Compiler / runtime** — the TAFJ compiler (jBC→Java) and the TAFJ runtime (jBC semantics + services in Java).
- **Application server** — JBoss/WebLogic/standalone (Tomcat/WebSphere unverified): hosts the TAFJ runtime.
- **Cloud-native / Docker / Kubernetes** — the containerised deployment model for TAFJ and Transact.
- **GLDB** — Green Link Digital Bank: Singapore DWB running Temenos on Huawei Cloud (MySQL + K8s; TAFJ implied) — `green_link_digital_bank_guide.md`.
- **COB / EOD / batch** — Close of Business / end-of-day batch processing on the TAFJ batch JVM — `core_banking_processes_guide.md`.
- **JVM tuning / heap / GC** — heap sizing and garbage-collection tuning: TAFJ's signature operational discipline (GC pauses freeze all agents on the JVM).

---

## 10. References and Verification Trail

Primary-adjacent public sources used (all cross-checked, none is Temenos-official full
documentation):

1. "Evolution and Architecture of T24" (R10.01) training deck — T24 naming, TAFJ-as-Java-app-server definition, R04 web architecture evolution.
2. "TAFJ Installation Guide" / "1. TAFJ Install Day1" deck — jBC parsed by TAFJ parser + compiled by javac; JDBC databases (jBASE, Oracle, SQL Server, DB2, Derby); H2; Pick→Oracle export tool.
3. "TAFJ Guide for Eclipse Users" / "Design Studio" guides — Eclipse-based IDE, tRun/tShow/tDebug, JED, lock view, JAR packaging, `classes/com/temenos/t24`.
4. "Comparative Analysis of Core Banking System — Temenos Application Frameworks" (Academia.edu) — thread-vs-process concurrency (20 agents = 20 threads vs 20 processes).
5. FinTech Futures (Nov 2016) — Jordan Ahli Bank live on T24 with AA + TAFJ.
6. t24guru.com — "Why T24 Became Temenos Transact": TAFC legacy vs TAFJ investment; operational-impact framing.
7. Stack Overflow / t24ALL — TAFJ deployment automation (tRun installers, Maven t24-packager-tafj), TAFJERR-1005 exception evidence, JBoss R18.
8. T24 Reference Architecture (Oracle/OEL) deck — WebLogic deployment; TOFC (EE).
9. green_link_digital_bank_guide.md — GLDB stack (Huawei Cloud, RDS MySQL, CCE/CCI); engine inference flagged.
10. LinkedIn/consultant material — L3 Java, TAFJ Ext folder, "TAFJ talks to Java" patterns, TAFJ job-advert skill lists.

Sibling guides cross-referenced: `temenos_guide.md`, `temenos_data_model_guide.md`,
`../technology/jbase_universe_guide.md`, `core_banking_systems_guide.md`,
`core_banking_processes_guide.md`, `posting_engine_core_banking_guide.md`,
`interest_engines_core_banking_guide.md`, `green_link_digital_bank_guide.md`,
`oracle_banking_microservices_architecture_guide.md`.

**Flagged items summary:** TAFJ's first production release date (R10/R11 window);
TAFJ-vs-TAFC performance benchmarks (none public); "SAB" as a Temenos component (unverified);
GLDB's engine (TAFJ by inference); certified platform matrix details (partner-only);
Tomcat/WebSphere certification (unverified); the 90%-compiles-no-change migration figure
(consultant folklore); TAFC end-of-life (none announced).
