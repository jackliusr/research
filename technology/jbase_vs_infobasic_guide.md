# jBASE vs InfoBasic: The Two BASIC Dialects of the MultiValue World — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Database / Data Engineering — the language-level comparison of the MultiValue BASIC family: **jBC** (jBASE BASIC) vs **InfoBasic** (UniVerse/UniData BASIC) — heritage, language differences, compatibility and porting, tooling, the T24 context (Data Engineering series)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Scope and How This Guide Fits the Series

This guide is the **dedicated deep-dive on the two main BASIC dialects of the MultiValue world** — the languages, not the databases. It deliberately does not re-hash the database-level material; where a topic already has a dedicated guide, this document cross-references it by filename:

- The **database family** — MultiValue data model, PICK heritage, jBASE deep-dive, UniVerse deep-dive, UniData, the ownership chains, current versions → `jbase_universe_guide.md` (same directory; the umbrella for this guide)
- The **T24/Transact application model** — FBNK files, versions, OFS, AA → `../banking/temenos_data_model_guide.md`
- The **T24/Transact programming** — jBC in the T24 context, routines, the Development Framework → `../banking/t24_programming_guide.md`
- The **TAFJ engine** — the Java runtime that compiles jBC to Java bytecode (the alternative to the TAFC C engine) → `../banking/tafj_guide.md`
- The relational contrast the MV family is measured against → `nosql_data_modelling_guide.md`

**Verification note — read this first.** The authoritative sources for both dialects are **Rocket Software's documentation** (the jBASE BASIC manuals, the UniVerse BASIC User Guide and Commands Reference, the UniData BASIC docs) and the **practitioner community** (Pickwiki, International Spectrum, the Rocket MV Developers community, consultant blogs). Temenos's T24 documentation is proprietary (Base Camp, training decks — see `../banking/t24_programming_guide.md` §1.4). Where a fact is verified against a public source it is stated plainly; where it is inferred, hearsay, or version-dependent it is **flagged**. The single most important honest framing: **jBC and UniVerse BASIC are not two languages — they are two dialects of one language (PICK BASIC), and porting between them is real work, not a recompile.** Claims of "drop-in" compatibility found in vendor material should be treated as marketing until demonstrated (§4).

---

## Table of Contents

1. [The Two Dialects](#1-the-two-dialects)
2. [The Heritage: Two Branches, One Root](#2-the-heritage-two-branches-one-root)
3. [The Language Comparison](#3-the-language-comparison)
4. [Compatibility and Porting](#4-compatibility-and-porting)
5. [The Tooling](#5-the-tooling)
6. [The T24 Context](#6-the-t24-context)
7. [Worked Example: Porting a Routine](#7-worked-example-porting-a-routine)
8. [The Future: 2026 and Beyond](#8-the-future-2026-and-beyond)
9. [Glossary](#9-glossary)
10. [References and Verification Trail](#10-references-and-verification-trail)

---

## 1. The Two Dialects

### 1.1 The One-Paragraph Answer

Every MultiValue database speaks a dialect of the same BASIC family, inherited from the PICK operating system of the 1970s. The two dialects that matter commercially are:

- **jBC** (jBASE BASIC) — the language of **jBASE**, Rocket Software's OS-native MultiValue database and the engine embedded in Temenos Transact. jBC compiles via C to native machine code.
- **InfoBasic** — the family label for the BASIC of Rocket's **U2** products: **UniVerse BASIC** (UniBasic) and **UniData BASIC** (UDBASIC). The name descends from Prime INFORMATION's **INFO/BASIC** of the 1980s; Rocket's current documentation calls the languages "UniVerse BASIC" and "UniData BASIC", and the unified tooling "Rocket MV BASIC" (§1.3).

They share the same dynamic-array data model (`record<attr, mv, sv>`), the same file verbs (`OPEN`/`READ`/`WRITE`), the same conversion functions (`OCONV`/`ICONV`/`FMT`), and the same @-system variables. They differ in syntax details, function inventories, error handling, tooling, and — most importantly for anyone planning a migration — in the application frameworks that surround them (T24 on jBC; SB+/vertical apps on UniBasic).

### 1.2 jBC: The jBASE BASIC

**jBC** (the official Rocket name is **jBASE BASIC**; "jBC" is the shorthand used throughout the jBASE manuals) is the application programming language of the jBASE database. Key facts:

- **Compiler model.** jBC is compiled by the `basic` command into C, then by the host C compiler into native executables — "jBASE is unique in that it compiles applications to native machine code form, rather than to an intermediate byte code" (Pick OS literature; verified). This is why jBC programs run at near-C speed, can link C libraries directly, and why the T24 community calls the C-based runtime **TAFC**.
- **Family DNA.** It is a PICK-BASIC descendant: dynamic-array strings with attribute/value/sub-value marks, hashed-file verbs, `GOSUB`/`RETURN` structure, `@`-prefixed system variables.
- **Modern extensions.** jBASE has pushed the dialect furthest toward modernity: dynamic objects (JSON-like structures, jBASE 5.x), object-oriented class syntax (jBASE 6.x), `CALLJ` for Java interop, sockets, threading, and the `EXECUTE` verb for running jQL.
- **It is also a scripting language.** The jBASE shell (`jsh`) can execute jBC statements interactively and run programs; `jsh -i` gives an interactive BASIC-like shell, which makes jBC double as the jBASE scripting language (like `ksh` for the data layer).
- **The database connection.** jBC is the language of the jBASE database — files are opened, read, locked, and written from jBC; jQL (the query language) is invoked from jBC via `EXECUTE`/`SELECT`. The database-level picture (architecture, jEDI, transaction journal) is in `jbase_universe_guide.md` §3.

The T24 variant of jBC (T24 BASIC, with the `I_` validation layer, versions, and enquiries) is covered in `../banking/t24_programming_guide.md` §2 and `../banking/temenos_data_model_guide.md` §2; the Java runtime that replaces the C one is in `../banking/tafj_guide.md`.

### 1.3 InfoBasic: The UniVerse/UniData BASIC

**InfoBasic** is the historical family name for the BASIC of the Rocket U2 databases. The naming has three layers that are worth getting straight:

1. **INFO/BASIC** (1980) — the original: the BASIC of Prime Computer's **INFORMATION** database. Verified against the primary source: "INFO/BASIC programming language reference manual for the PRIME INFORMATION computer system" (Prime Computer, Inc., Framingham MA, ©1980, PDR 3903; WorldCat OCLC 223321054). Prime INFORMATION was a licensed re-implementation of the Pick model that "deliberately left out some features and added others" (Wikipedia "Prime Computer").
2. **UniVerse BASIC / UniData BASIC** — the modern Rocket product names. Rocket's documentation library ships "UniVerse BASIC User Guide" (11.x and 14.x editions), "UniVerse BASIC Commands Reference", and the UniData BASIC equivalents. Practitioners call UniVerse BASIC "UniBasic" and UniData BASIC "UDBASIC".
3. **Rocket MV BASIC** — the current *tooling* umbrella: Rocket's MV BASIC for Visual Studio Code extension (MVVS) speaks to UniVerse, UniData, *and* jBASE servers from one IDE (§5.4).

*Flag:* "InfoBasic" as a single product name does not exist in Rocket's current catalogue — it is the practitioner/historical family label (from INFO/BASIC via the U2 manuals' shared heritage). UniVerse BASIC and UniData BASIC are **separate compilers with a shared core** (UniData's has a stronger SQL-mapping flavour; see `jbase_universe_guide.md` §5.2). For the rest of this guide, "InfoBasic" means the U2 BASIC family (UniVerse BASIC primary, UniData BASIC noted where it differs).

### 1.4 One Language, Two Dialects

The correct mental model is **dialects of one language**:

- **The common core** (what "PICK BASIC" means in practice — §4.1): dynamic-array string handling, the three delimiters, `OPEN`/`READ`/`WRITE`/`DELETE` file verbs, `IF-THEN-ELSE`/`BEGIN CASE` structure, `FOR-NEXT` and `LOOP...REPEAT`, `GOSUB`/`RETURN`, `OCONV`/`ICONV`/`FMT` conversions, and the `@` system variables. Code written to this core compiles on both engines after cosmetic fixes.
- **The dialect layer** (what differs — §3): function inventories (each dialect has functions the other lacks), file-I/O semantics (locking verbs, `STATUS` handling, select lists), error handling, `EXECUTE` behaviour, system-variable names, and the toolchains.
- **The framework layer** (what really differs in the field): T24's `I_` routines, versions, and OFS on jBC have no UniBasic equivalent; SB+/SystemBuilder screens on UniVerse have no jBC equivalent. A T24 jBC developer and a UniVerse SB+ developer both write "MV BASIC", but neither can sit down at the other's terminal and be productive without retraining.

### 1.5 The Context: Engines and Versions

The dialect comparison sits on top of the engine comparison, which is covered in depth in `jbase_universe_guide.md` §2–§5. The version facts you need for this guide (verified there against Rocket's Product Lifecycle Handbook, Nov 2025, and release notes):

| Product | Current GA (Aug 2026) | Notes |
|---|---|---|
| **jBASE** | **6.2.2** (~Q4 2025) | 5.9.0 (May 2023) still the version many T24 estates run; 6.x is the modern line |
| **UniVerse** | **14.2.1** (Nov 2025) | 11.x/12.x still widely deployed in the installed base |
| **UniData** | **8.4.1** (~Q2 2026) | The SQL-mapping flavour of U2 |

*Correction to a common belief:* the "jBASE 5.x vs UniVerse 11.x" pairing in older write-ups reflects the 2015–2023 era; as of 2026 the current pairing is **jBASE 6.2.x vs UniVerse 14.2.x**. Rocket's lifecycle commitment runs to 2029–2033 across the MV lines — the portfolio is being maintained, not wound down (`jbase_universe_guide.md` §10).

### 1.6 Who Uses What: The Ecosystems

- **jBASE** — the engine under **Temenos Transact (T24)**, the most widely deployed core-banking platform (1,000+ banks, 150+ countries; see `../banking/temenos_guide.md`). Outside banking: jBASE ISVs, modern MV shops (containers, REST, jEDI external stores), and the Zumasys-era cloud MV practice.
- **UniVerse / UniData** — the historical workhorse of the MV vertical-application market: distribution, logistics, retail, education, healthcare, and a deep **financial-services installed base** (insurance policy administration, legacy banking applications, wealth platforms built in the 1980s–2000s that were never migrated). IBM's InfoSphere DataStage even embedded a UniVerse-flavoured BASIC as its stage language — a quiet reminder of how far the dialect spread.
- **The consequence for skills:** an MV BASIC developer's core skills transfer across both dialects; their *frameworks* (T24 vs SB+/custom) do not. This guide exists to make the transfer explicit.

---

## 2. The Heritage: Two Branches, One Root

### 2.1 The Common Ancestor: PICK and PICK BASIC

The full timeline (GIRLS 1965 → Pick OS 1973 → the 1990s OS-to-database pivot) is in `jbase_universe_guide.md` §1.2 and §2.1. The language-relevant summary:

- **1973** — Dick Pick ships the **Pick operating system**: OS + database + **PICK BASIC** + the ENGLISH query language in one box. PICK BASIC is the ancestor of *every* dialect in this guide: dynamic-array records, `@`-variables, `OCONV`/`ICONV`, `READ`/`WRITE`, `GOSUB`.
- **1975–1984** — the licensed-clone era: Microdata Reality, **Prime INFORMATION**, Ultimate, General Automation. Each implementation shipped its own BASIC flavour, all mutually intelligible, all source-compatible with the PICK core.
- **1984+** — the OS dies, the model survives as a database on Unix/Windows: **UniVerse (1984)**, **UniData (1985)**, **jBASE (1991)**, D3, OpenQM. Two of those — UniVerse and jBASE — are the subjects of this guide.

### 2.2 The InfoBasic Lineage: Prime INFORMATION → UniVerse

- **Prime INFORMATION (1979–80s).** Prime Computer (Framingham, MA) licensed the Pick model and built the **INFORMATION** database on its PRIMOS operating system — "a re-implementation which deliberately left out some features and added others" (Wikipedia "Prime Computer"), marketed for rapid 4GL-style application development. Its programming language was **INFO/BASIC** (verified: the 1980 Prime reference manual). Prime also shipped **INFO/ACCESS** (the ENGLISH-style query) and **INFO/REPORT** alongside it.
- **INFO/BASIC's character.** The dialect carried the classic Pick core plus Prime's own syntax flavour: `@`-system variables, `MAT` matrix verbs, `GOSUB`-structured programs, the `E`-command editor lineage. This is the "InfoBasic" heritage that the task framing refers to, and it is why UniVerse BASIC's syntax (and its manual's organisation) feels INFO/BASIC-shaped to this day.
- **VMark and UniVerse (1984).** VMark Software built **UniVerse** as a Unix-based implementation of the Pick model ("extended relational": MultiValue storage *plus* an SQL engine). The lineage chain, verified in `jbase_universe_guide.md` §2.3: **VMark (1984) → Ardent (1997) → Informix (2000) → IBM (2001) → Rocket (2009)**. UniData ran the parallel chain from Unidata Corporation (1985), merging into the same ownership stream in 1997.
- *Flag:* the exact personnel/technology transfer from Prime INFORMATION to VMark is **not documented in public sources this guide could verify** — what is verifiable is that both are Pick-family, and that UniVerse BASIC's `@`-variable and `MAT` syntax closely matches INFO/BASIC's. Do not repeat the stronger "UniVerse was written by ex-Prime engineers" claim without a primary source.

### 2.3 The jBASE Lineage: JAC → Temenos → Rocket

The verified chain from `jbase_universe_guide.md` §3.2:

- **~1989–91** — **James Anthony Consultants (JAC)**, a UK firm, builds **jBASE** (the "j" = James Anthony). Designed from the start as a *database-independent* MV platform: jBC applications run against jBASE files or, via the **jEDI** external-device interface, against Oracle, SQL Server, etc.
- **1999** — **Temenos** — a jBASE VAR that had built T24 on UniVerse and was migrating — acquires jBASE outright. The jBASE corporate history dates the acquisition to 1999 (some secondary sources say 2001; this guide follows the verified primary source). T24's engine relationship is covered in §6.
- **2002** — Mpower1 International takes worldwide distribution (non-banking), creating **jBASE International**; Temenos keeps the IP and the core team.
- **2015** — **Zumasys** acquires jBASE (Temenos retains a perpetual licence to ship jBASE inside T24/Transact).
- **2021** — **Rocket Software** acquires jBASE (with AccuTerm, MVConnect, OpenQM) — uniting jBASE and the U2 family under one owner for the first time.

### 2.4 The Two Branches

```
                     PICK OS (1973) — PICK BASIC
                              │
        ┌─────────────────────┴──────────────────────┐
   Licensed clones (1975–80s)                Independent re-implementations
        │                                       │
   Prime INFORMATION (1979)                jBASE / JAC (1991)
        │  INFO/BASIC                            │
   VMark UniVerse (1984)                    Temenos (1999)
        │                                       │
   Ardent (1997) → Informix (2000)          Mpower1 (2002)
        → IBM (2001) → Rocket (2009)            │
        │                                   Zumasys (2015)
   UniVerse 14.x / UniData 8.x              Rocket (2021)
        │                                       │
        └─────────── Rocket Software (2021) ─────┘
```

The two branches diverged for ~40 years (1984→2021) — enough time for genuinely different dialects to evolve — then rejoined under Rocket. The 2021 reunion is the reason the "dialect" question is live again: one vendor, two languages, one ecosystem (§5, §8).
## 3. The Language Comparison

This section is the core of the guide: feature-by-feature, what jBC and InfoBasic (UniVerse BASIC primary; UniData noted where it differs) do the same, and where they diverge. The authoritative references are the Rocket **jBASE BASIC Programmer's Guide / Commands Reference** and the **UniVerse BASIC User Guide / Commands Reference**; everything below is drawn from those families of manuals plus the practitioner community, with flags where the details are version-sensitive.

### 3.1 Statement Syntax

| Feature | jBC (jBASE BASIC) | InfoBasic (UniVerse BASIC) | Verdict |
|---|---|---|---|
| Terminal output | `PRINT` (to terminal), `CRT` (to screen, bypasses print queue), `PRINTER ON/OFF/CLOSE` (to print queue) | Same trio: `PRINT`, `CRT`, `PRINTER ON/OFF` | Same |
| Cursor/format codes | `@(-1)` (clear), `@(col,row)` positioning | Same `@()` convention | Same |
| Termination | `STOP` (with optional message/code); `HALT` (jBC: terminate with return status) | `STOP` (with optional code); `ABORT`; `HALT` exists mainly as a command-level verb | *Mostly same* — `HALT` semantics differ (flag: check per version; in jBC `HALT` exits with a status, in UniVerse `HALT` is primarily a TCL/shell verb) |
| Comments | `*` or `REM` at start of statement | Same (`*`/`REM`) | Same |
| Continuation | jBASE 5.2+: trailing `\` or `&`-style continuation; long statements via multiple physical lines | UniVerse: statement continuation is limited; long lines typical | jBC is more permissive (flag detail) |
| `IF-THEN-ELSE` | Single-line `IF expr THEN stmt ELSE stmt`; block form `IF expr THEN ... END ELSE ... END` (also `ELSE` block after `THEN` block) | Same single-line and block forms (`IF ... THEN ... ELSE ... END`) | Same |
| `CASE` | `BEGIN CASE / CASE expr / CASE ELSE / END CASE` | Same | Same |
| Subroutines | `GOSUB label` / `RETURN`; labelled subroutines; also `CALL` to external subroutines | `GOSUB` / `RETURN`; `CALL` to catalogued subroutines | Same core |
| `GOSUB` vs `GO SUB` | `GOSUB` is the keyword; "GO SUB" is not documented as an alternative in jBC | `GOSUB` is the keyword in UniVerse BASIC as well | *Settled:* the `{GO SUB | GOSUB}` dual spelling exists in other BASIC lineages (e.g. VSI BASIC on OpenVMS documents both), but in the MV world both dialects use `GOSUB` — not a real jBC/InfoBasic difference (flag: some PICK-emulation modes of jBASE tolerate legacy spellings, unverified) |
| Loops | `FOR var = a TO b STEP c ... NEXT var`; `LOOP ... REPEAT` with `WHILE`/`UNTIL` anywhere inside; statement-level `WHILE`/`UNTIL` on verbs (e.g. `READNEXT key FROM list UNTIL key = ""`) | Same: `FOR ... NEXT`, and `LOOP ... REPEAT` with `WHILE`/`UNTIL` **anywhere** in the loop body (documented; UniVerse's LOOP is famously flexible — the UNTIL can be the first or last statement) | Same — *corrects a common myth:* `LOOP...REPEAT` is not jBASE-only; both dialects have it |
| `CALL` indirection | `CALL routine` (compiled-in), `CALL @variable` (dynamic), `CALLJ` (Java) | `CALL routine`, `CALL @variable` (dynamic) | Same core; `CALLJ` is jBC-only |
| `EXECUTE` | `EXECUTE "jQL command"`; returns status in `@SYSTEM.RETURN_CODE` | `EXECUTE "UniQuery command"` with `CAPTURING`/`RETURNING` clauses; status handling differs | Same concept, different mechanics — a real porting point |

### 3.2 File I/O

Both dialects speak the same PICK file-verb family, but the *details* of locking, status reporting, and select lists differ in ways that bite during porting.

| Verb | jBC | InfoBasic (UniVerse) | Notes |
|---|---|---|---|
| `OPEN` | `OPEN filevar TO var THEN ... ELSE ...` (also `OPEN 'DICT'`) | Same shape | Same |
| `READ` | `READ rec FROM var, key SETTING err THEN/ELSE` | `READ rec FROM var, key THEN/ELSE`; `READL`/`READU` variants for locked reads | Same core; jBC's `SETTING err` clause is the modern status idiom (older `STATUS`/`STATUS1` still work in both) |
| `READU` (record lock) | `READU rec FROM var, key LOCKED stmt THEN/ELSE` | `READU` with `LOCKED` clause (and `READL` for read-with-lock) | Same concept; lock-wait semantics and `LOCKED` handling differ in edge cases (flag) |
| `READV` / `WRITEV` | `READV val FROM var, key, attr` / `WRITEV val ON var, key, attr` | Same | Same |
| `WRITE` / `DELETE` | `WRITE rec ON var, key` / `DELETE var, key` | Same | Same |
| `RELEASE` | `RELEASE var, key` (unlock) | Same | Same |
| `MATREAD` / `MATWRITE` | Supported (matrix I/O) | Canonical — a signature UniVerse verb family | Present in both; jBC practitioners lean on `READ` into `@RECORD`-style arrays more |
| `READNEXT` | `READNEXT key FROM list THEN/ELSE` (also `READNEXT` with `BY`/`WITH` on the list) | `READNEXT key FROM list THEN/ELSE` | Same — *both* dialects use `READNEXT` (the task framing "READNEXT — jBASE — verify" is resolved: UniVerse has it too) |
| `SELECT` / `SSELECT` | `SELECT file {WITH criteria}` builds a numbered select list; `SSELECT` = sorted; list consumed by `READNEXT`; `@SELECTED` holds the count | `SELECT file {WITH criteria}` / `SSELECT` (sorted); same `READNEXT` consumption; `@SELECTED` count | Same. *Flag:* "`SELECT *`" is **SQL** syntax (UniVerse SQL / jBASE SQL engine), not MV-BASIC — you will see it in SQL-bridge contexts, never in BASIC source |
| `GET-LIST`/`SAVE-LIST`/`DELETE-LIST` | Named select lists | Same | Same |
| `EXECUTE` of queries | `EXECUTE "SELECT F.CUST WITH ..."` then read the list | `EXECUTE` UniQuery, or `SELECT` at TCL and consume `@LISTS` | Same pattern |

**The one real I/O difference that matters for porting:** *status reporting*. jBC's modern idiom threads the error code through `SETTING err` on `READ`/`OPEN`/`WRITE`; UniVerse code historically uses `STATUS()` / `STATUS1()` after the verb (jBASE supports these too for compatibility). When porting, the `SETTING`-clause code must be rewritten to `STATUS()` calls, and vice versa — a mechanical but pervasive change.

### 3.3 Dynamic Arrays

- **Delimiters.** Identical characters in both dialects: attribute mark 0xFE (`^`), value mark 0xFD (`]`), sub-value mark 0xFC (`]]`). The system-variable names differ cosmetically: jBC's canonical names are `@FM`/`@VM`/`@SM`; UniVerse's canonical names are `@AM`/`@VM`/`@SM` (`@FM` is accepted in UniVerse contexts as the historical "field mark" synonym). Porting: a global `@FM`→`@AM` sweep on UniVerse-bound code is cosmetic but must not be skipped — the two names are the same character.
- **The `<n>` notation.** Identical: `record<attr>`, `record<attr, mv>`, `record<attr, mv, sv>` for read; `record<attr> = value` for write (dynamic assignment). Both support `record<attr, mv, sv; start:length>` slicing in modern versions.
- **`@RECORD` / `@ID`.** Both provide `@ID` (current record key). jBC exposes the full current record as `@RECORD`; UniVerse has `@RECORD` as well (with `@RECORD<attr>` access). Same concept, check version support.
- **Delimiter functions.** `FIELD`, `DCOUNT`, `INDEX`, `CONVERT`, `CHANGE`, `REPLACE`, `INSERT`, `DELETE` (string function), `TRIM` — present in both.

### 3.4 Functions

**Common core (same in both):**

| Function | Purpose | Notes |
|---|---|---|
| `OCONV(value, conv)` | Output conversion (formatting for display) | Identical semantics and conversion codes (`MD2`, `D4`, `G1`, …) |
| `ICONV(value, conv)` | Input conversion (parsing user input) | Same |
| `FMT(value, fmt)` | Format according to dictionary-style masks | Same |
| `DCOUNT(string, delim)` | Count delimited elements | Same |
| `FIELD(string, delim, n)` | Extract nth element | Same |
| `INDEX(string, substr, n)` | Position of nth occurrence | Same |
| `TRIM`, `STR`, `SPACE`, `SEQ`, `CHAR`, `NUM`, `LEN`, `SUBROUTINES`, `TIME()`, `DATE()` | String/utility functions | Same |
| `MATPARSE` / `MATBUILD` | Split a dynamic array into a matrix / rebuild | **Both** have them (verified in the UniVerse Commands Reference and jBASE manuals) — the task's "matrix functions — verify" resolves: they are not jBC-only |
| `STATUS()` / `STATUS1()` | Post-verb file status | Both (jBC adds the `SETTING` clause as the modern idiom) |

**Where they differ:**

| Function | jBC | InfoBasic (UniVerse) |
|---|---|---|
| Regex replace | `EREPLACE(string, pattern, repl)` — long-standing jBC function | `EREPLACE` **also exists** (listed in the UniVerse BASIC Commands Reference) — *resolves the task's "EREPLACE (jBASE — verify)"*: it is in both dialects today, though UniVerse's pattern syntax nuances differ (flag) |
| Regex match | `REGEXP` / `OCONV(x,"M...")` | UniVerse uses `MATCHES` / `MATCH` operators and `OCONV` pattern codes | Different idioms for the same job |
| Object/JSON | Dynamic objects (`{...}`, `->` navigation, `$`-prefixed object functions), `JSON` serialisation | No native object type; JSON handled via `EXECUTE` bridges or `@`-string parsing | jBC-only — a major dialect gap |
| Sorting | `SORT`, `COLLATE` (internal) | `SORT` similar; locale/`COLLATE` differences | Same concept |
| Misc jBC extras | `JGETENV`, `JSETENV`, `SYSTEM(100)`, `@USERNAME`, `@LOGNAME`, `ASSIGNED`, `FILEINFO`, `FDELETE` (jBASE 5.x+), `FREAD`/`FWRITE` (OS file access), `PERFORM` | UniVerse equivalents: `SYSTEM(100)` differs, OS file access via `OPEN` on `&SAVEDLISTS&`-style paths or `CALL SYS`-type subroutines | The function *inventories* overlap ~80% but the tail is genuinely different |

**System variables (`@` variables):**

| Variable | jBC | UniVerse | Notes |
|---|---|---|---|
| `@DATE` / `@TIME` | Yes | Yes | Same format (PICK internal date, seconds since midnight) |
| `@FM`/`@VM`/`@SM` (jBC) vs `@AM`/`@VM`/`@SM` (UniVerse) | `@FM` canonical | `@AM` canonical | Same characters (§3.3) |
| `@ID` | Current record key | Same | Same |
| `@RECORD` | Current record | Same | Same |
| `@USERNAME` / `@USERNO` | `@USERNAME` (jBASE 5.x+; `@LOGNAME` too) | `@USERNAME`, `@USERNO` | Same names, slightly different semantics (flag) |
| `@SELECTED` | Select-list count | Same | Same |
| `@SYSTEM.RETURN_CODE` | `EXECUTE` status | UniVerse uses `@SYSTEM.RETURN_CODE` too (and `@STATUS` in places) | Mostly same |
| `@FILE.NAME`, `@PATH`, `@WHO`, `@SENTENCE` | Yes | Partial overlap | Version-dependent (flag) |

### 3.5 Dialect Extensions

**jBC extensions** (no UniVerse equivalent):

- **Dynamic objects** — JSON-like maps/lists with `->` member access (`obj->name`), `$`-prefixed functions (`$set`, `$get`, `$list`…), `JSON` conversion. This is the biggest single language gap between the dialects: jBC has a modern data structure; UniBasic does not.
- **OO class syntax (jBASE 6.x)** — `CLASS`, `METHOD`, `PROPERTY`, inheritance — real OOP in MV BASIC, used in modern jBASE applications.
- **`CALLJ`** — direct Java method invocation from jBC (jBASE 5.6+), the bridge TAFJ-style Java integration leans on.
- **Threading and sockets** — `THREADCREATE`/`THREADJOIN`, socket verbs (`SOCKET` open/read/write/close); UniVerse has no equivalent in-language.
- **jCL and PROC.** jBASE supports the PICK-style **PROC** procedure language (via emulation) and its own **jCL** (jBASE Command Language — the `&`-macro command layer). *Flag:* jBASE's PROC support is emulation-dependent; treat "jBASE procedures" as jCL-first, PROC-for-PICK-compat.
- **Error handling.** jBC: `ON ERROR GOSUB` (statement-level trapping), `SETTING err` clauses, and — in the 6.x line — structured exception handling in the class syntax. *Flag:* the exact jBASE 6 TRY/CATCH surface is not documented in public sources this guide could verify; the *verified* jBC idioms are `ON ERROR` + `SETTING` + `STATUS()`.

**InfoBasic extensions** (no jBC equivalent, or different in jBC):

- **`MAT` matrix verbs** — `MATREAD`/`MATWRITE`/`MATPARSE`/`MATBUILD`/`MAT` dimensioning; jBC has them too, but UniVerse treats them as first-class (their zero-element convention for `MATPARSE` is documented in the UniVerse User Guide).
- **`EXECUTE ... CAPTURING`/`RETURNING`** — UniVerse's rich capture of command output into variables; jBC's `EXECUTE` returns status but output capture goes through `@` variables / temporary files.
- **`ON ERROR` at program level, `BEGIN ... END` blocks** — UniVerse's structured `BEGIN`/`END` (e.g. `BEGIN CASE` is shared, but UniVerse also has `BEGIN`/`END` for `CRT`/`PRINT` redirection groups) — details differ (flag).
- **The `UNI*` / `INFO*` family.** *Flag — resolves the task's "UNI functions / INFO functions" question:* there is **no standard `UNI*` or `INFO*` function family inside UniVerse BASIC**. The "INFO"/"UNI" branding lives in the *products around* the language: Prime's INFO/ACCESS + INFO/REPORT (the 1980s siblings of INFO/BASIC), and Rocket's **UniQuery**, **UniObjects**, **UniVerse SQL**, **U2 ODBC/JDBC**. Inside the language, UniVerse BASIC's distinctive calls are `CALL` to catalogued subroutines, `MAT` verbs, and `EXECUTE` — not `INFO()`-style built-ins.
- **PROC and PARAGRAPH.** UniVerse supports the PICK **PROC** procedure processor (UVPROC) and **PARAGRAPH** (stored command lists) at the shell level. *Flag:* the exact PARAGRAPH feature set varies by UniVerse version; both are shell-level, not BASIC-language, constructs.

### 3.6 The Differences Table (Summary)

| Feature | jBC (jBASE BASIC) | InfoBasic (UniVerse BASIC) |
|---|---|---|
| Compiler model | `basic` → C → native executable | `basic` → compiled host executable (own code generator) |
| Delimiter variables | `@FM`/`@VM`/`@SM` canonical | `@AM`/`@VM`/`@SM` canonical |
| Record access | `rec<1,2,3>` — same | `rec<1,2,3>` — same |
| File verbs | `OPEN/READ/READV/READU/WRITE/WRITEV/DELETE` | Same + `READL`, `MATREAD`/`MATWRITE` first-class |
| Error status | `SETTING err` clauses (modern); `STATUS()` legacy | `STATUS()`/`STATUS1()`; `ON ERROR` |
| Select lists | `SELECT`/`SSELECT`/`READNEXT` + named lists | Same |
| Locking | `READU`/`WRITEU`/`RELEASE` | `READU`/`READL`/`WRITEU`/`RELEASE` |
| Regex | `EREPLACE`, `REGEXP` | `EREPLACE`, `MATCHES`/`MATCH` |
| Objects/JSON | Dynamic objects, classes (6.x), `CALLJ` | None in-language |
| Threads/sockets | Yes | No |
| Query integration | `EXECUTE` jQL | `EXECUTE` UniQuery + `CAPTURING` |
| Matrix verbs | Supported | Canonical |
| Shell/editor | `jsh`, jED | `uv` shell, ED editor |
| Framework layer | T24/Transact (TAFC/TAFJ) | SB+/SystemBuilder, custom verticals |
| SQL surface | jBASE SQL engine, JDBC/ODBC | UniVerse SQL, U2 ODBC/JDBC |
| Typical installed base | Temenos Transact banking; modern MV | Vertical apps: distribution, retail, insurance, legacy finance |

### 3.7 Quick Reference: The Statement Cheat Sheet

A compact side-by-side of the statements a developer actually types, for the two dialects. Read it as "same" unless a delta is shown:

| Statement | jBC | InfoBasic (UniVerse) | Delta |
|---|---|---|---|
| `OPEN` | `OPEN 'FILE' TO F ELSE ...` | identical | none |
| `READ` | `READ R FROM F, K SETTING E THEN/ELSE` | `READ R FROM F, K THEN/ELSE` | status idiom |
| Locked read | `READU R FROM F, K LOCKED L THEN/ELSE` | `READU`/`READL` + `LOCKED` | minor |
| Read attr | `READV V FROM F, K, A` | identical | none |
| Write | `WRITE R ON F, K` | identical | none |
| Write attr | `WRITEV V ON F, K, A` | identical | none |
| Delete | `DELETE F, K` | identical | none |
| Unlock | `RELEASE F, K` | identical | none |
| Select | `SELECT F WITH C` / `SSELECT F` | identical | none |
| Next key | `READNEXT K FROM L ELSE ...` | identical | none |
| Named list | `SAVE.LIST`/`GET.LIST` | `SAVE.LIST`/`GET.LIST` | none |
| If | `IF e THEN s ELSE s` / block `IF ... END` | identical | none |
| Case | `BEGIN CASE/CASE/CASE ELSE/END CASE` | identical | none |
| Loop | `LOOP ... REPEAT` / `FOR ... NEXT` | identical | none |
| Sub call | `GOSUB L` / `CALL sub` | identical | none |
| Convert | `OCONV`/`ICONV`/`FMT` | identical | none |
| Replace | `REPLACE`/`CHANGE`/`EREPLACE` | `REPLACE`/`CHANGE`/`EREPLACE` | none (regex nuances) |
| Delim vars | `@FM`/`@VM`/`@SM` | `@AM`/`@VM`/`@SM` | **rename** |
| Attr access | `R<1,2,3>` | identical | none |
| Matrix | `MATPARSE`/`MATBUILD`/`MATREAD` | identical (first-class) | none |
| Command | `EXECUTE 'cmd'` | `EXECUTE 'cmd' CAPTURING O` | **capture** |
| Terminate | `STOP` / `HALT n` | `STOP` / `ABORT` | minor |
| Objects | `obj->name`, `{...}`, `CALLJ` | — | **no equivalent** |
| Comment | `*` / `REM` | identical | none |

The cheat sheet condenses the entire §3 argument into one row per construct: **~20 of 24 rows are "same or cosmetic"; the load-bearing deltas are the delimiter-variable names, `EXECUTE` capture, status idioms, and jBC's object/Java extensions.** A port that stays inside the "same" rows is the portable PICK.BASIC subset (§4.1).

---

## 4. Compatibility and Porting

### 4.1 "PICK.BASIC": The Portable Common Subset

The honest concept behind "PICK.BASIC" is **the common core of §3.3–3.4** — the R83-era PICK BASIC subset that both dialects compile with cosmetic fixes:

```
OPEN, READ, WRITE, DELETE, READV, WRITEV
IF ... THEN ... ELSE ..., BEGIN CASE ... END CASE
FOR ... NEXT, LOOP ... REPEAT (WHILE/UNTIL)
GOSUB ... RETURN, CALL
OCONV, ICONV, FMT, DCOUNT, FIELD, INDEX, TRIM, STR, CONVERT
@DATE, @TIME, @ID, @FM/@AM, @VM, @SM
record<attr, mv, sv> notation
```

Code written to this subset — no `SETTING` clauses, no dialect-only functions, no `EXECUTE` capture tricks, no objects — **compiles on both engines after a `@FM`↔`@AM` sweep**. This is the pragmatic definition used by the migration community. It is *not* the same as "runs unchanged": even the subset has semantic edges (error behaviour after a failed `READ`, `LOCKED` handling, numeric precision) that only testing surfaces.

### 4.2 Porting Effort: The Honest Assessment

*Flag — general statement:* there are **no public, measured studies** of jBC↔UniBasic porting cost; the figures below are the practitioner consensus (consultant write-ups, community threads, e.g. the "converting from jBASE to Universe" discussions on Stack Overflow / Pickwiki) and should be treated as directional.

- **The 80/20 rule.** For typical *statement-level* business code (record reads, loops, string manipulation, screen I/O), roughly 80% of lines port with mechanical changes (delimiter-variable names, status idioms, `PRINT`/`CRT` normalisation). The 20% that hurts: file-I/O error semantics, locking behaviour, select-list edge cases, `EXECUTE` output capture, dialect-only functions, and — above all — **the frameworks** (T24's `I_`/version layer vs SB+ screens vs custom utility libraries).
- **jBASE → UniVerse** (the rarer direction in banking — jBASE shops moving an app to U2): mechanical fixes are `@FM`→`@AM`, `SETTING err` → `STATUS()` checks, `CALLJ`/objects/threads → rewrite in UniBasic (no equivalent), jQL `EXECUTE` → UniQuery `EXECUTE ... CAPTURING`. Real effort: small for core business logic, large for anything using jBASE-only extensions.
- **UniVerse → jBASE** (the common direction — U2 apps modernising onto jBASE, or T24 migrations pulling UniVerse data/logic across): mechanical fixes are `@AM`→`@FM`, `STATUS()` → `SETTING` or retained `STATUS()`, `EXECUTE ... CAPTURING` → jBC capture idioms, `MATREAD` stays (jBC supports it). UniVerse-specific constructs without jBC equivalents are rare in the core language — jBC is the superset — so this direction is generally the easier of the two, *except* where the UniVerse app leans on UniVerse SQL (`SELECT * FROM` inside programs) or SB+, which have no jBC analogue (SB+ paragraphs must be rewritten; there is a historical jBASE path via **OSMOSiS**, the Mpower1-era 4GL converter — flag: legacy, no longer marketed).

### 4.3 Conversion Tools

- **Automated converters.** *Flag — verified as "no mature public tool":* this guide found **no widely-adopted automated jBC↔UniBasic source converter** in the public domain. The realistic tooling is:
  - **Scripted transformations** — `sed`/`perl` sweeps for the mechanical deltas (`@FM`↔`@AM`, `SETTING`→`STATUS()`, `CAPTURING` clauses), which handle the 80% and must be followed by manual review of the 20%. Practitioner communities share such scripts (Pickwiki, GitHub) but nothing is standardised.
  - **Rocket professional services** — Rocket's MV migration practice (and historically Zumasys) performs cross-MV conversions as consulting engagements; no public converter product is offered for jBC↔UniBasic.
  - **Third-party cross-dialect tooling** — e.g. **BP Forms** (BluePinc), an MV form/report generator that ships pre-built source for jBASE, UniData, and other dialects from one definition — evidence that cross-dialect code *generation* exists commercially even if *translation* does not.
- **The compiler-as-tester.** Both dialects' `basic` compilers are strict about undefined functions, so a fast "compile → fix errors → compile" loop is the de-facto migration tool. Compile-clean is the first milestone; behaviour-clean is the second.
- **Data migration** (the other half): records are plain ASCII with the same three delimiters in both engines, so bulk data moves with `jbackup`/`uvbackup` extracts + key/record copies; the hard part is dictionary and application migration, not bytes (see `jbase_universe_guide.md` §7.4).

### 4.4 Manual Porting: The Checklist

A practical, order-of-operations checklist used by practitioners:

1. **Inventory the surface:** count programs, catalogued subroutines, paragraphs/PROCs, `EXECUTE`d queries, embedded SQL, SB+/screen definitions.
2. **Normalise the delimiters:** `@FM`↔`@AM`, `@VM`/`@SM` (same both sides).
3. **Normalise status handling:** `SETTING err` → `STATUS()`/`STATUS1()` (or keep both working where the target supports them).
4. **Rewrite `EXECUTE` output capture** (jBC `@`-variable capture vs UniVerse `CAPTURING`/`RETURNING`).
5. **Map the function tail:** replace dialect-only functions (jBC objects/`CALLJ`/threads; UniVerse `MATCHES` vs jBC `REGEXP`, etc.).
6. **Locking pass:** review every `READU`/`READL`/`WRITEU`/`RELEASE` for `LOCKED`-clause and timeout semantics in the target engine.
7. **System variables:** reconcile `@USERNAME`/`@USERNO`/`@LOGNAME`, `@SYSTEM.RETURN_CODE` semantics.
8. **Query integration:** rewrite jQL↔UniQuery `EXECUTE` strings and any embedded UniVerse SQL (`SELECT * FROM …`) into the target's SQL surface or jQL.
9. **Framework translation:** T24 routines (`I_` validation, versions) ↔ UniVerse subroutines/catalogued programs; SB+ ↔ jBASE 4GL equivalents — this is the project-sized part.
10. **Test equivalence:** build a golden-data regression set; compare outputs record-by-record (conversion formatting via `OCONV` is the usual silent-breaker).

### 4.5 Compatibility Modes

- **jBASE's `EMULATE` (verified mechanism).** jBASE emulates other MV platforms through the **`Config_EMULATE`** file in `$JBCGLOBALDIR/config` (jBASE 6.x documentation: "Emulation support is provided by the Config_EMULATE file in the config subdirectory of the jBASE release directory"; the modern doc page is "Emulation (revised)" in the jBASE 6.1 library). The `EMULATE` command/`JBASE_EMULATE` environment variable selects the active emulation; the file contains per-emulation option lines (e.g. the **UV-compat** family: `conv_mct_uv_compat`, `Universe compatible capitalisation after non-alpha characters` — verified in the Zumasys-era manual). Known emulation targets include the classic PICK lines — **PICK, PRIME, D3, R83, QM, mvBase** — and the general claim in Pick OS literature that "jBASE emulates all implementations of the system to some degree" (verified). *Flag:* whether a full **"UniVerse" emulation label** ships in current `Config_EMULATE` could not be verified against the 6.x file; what is verified is that UV-compatibility *options* exist and that jBASE compiles PICK-family source under its emulation modes. Treat "EMULATE = UniVerse" as "partial, option-driven" until demonstrated on the exact jBASE release.
- **UniVerse compatibility modes.** UniVerse has **no `EMULATE` equivalent** — it is a fixed dialect. Its flexibility is at the *compile* level: the UniVerse `basic` compiler options and the `UNIVERSE`/compatibility settings affect dialect strictness (e.g. accepting some PICK idioms), and Rocket's MV BASIC tooling presents a common front-end to both dialects (§5.4). *Flag:* per-option details are version-specific; the honest statement is "UniVerse compiles UniVerse BASIC; cross-dialect behaviour comes from jBASE's side via EMULATE, not the reverse."
- **Mixed environments (the reality).** jBASE and UniVerse **do coexist** in many estates — the typical shape being a U2 legacy application (or T24 on jBASE) plus a second engine acquired through merger. The practical integration surfaces, all covered in `jbase_universe_guide.md` §6: **record-level exchange** (same ASCII format — extracts and loads), **SQL bridges** (jBASE SQL / UniVerse SQL, U2 ODBC/JDBC), **REST** (MVIS/MVConnect), and **UniObjects** for programmatic access. What you *cannot* do is share a live file between engines or run one engine's compiled program on the other — the dialects compile to different executables.
## 5. The Tooling

### 5.1 Compilers

- **jBASE.** `basic` compiles jBC source to C, then the host C compiler produces a native executable (verified: jBASE "compiles applications to native machine code form, rather than to an intermediate byte code"). `catalog` installs the compiled program into a library (CATALOG file) so it can be called by name from `jsh` or from other programs; `decat` removes it. Compile flags: `basic -J` (object listing), `basic -D`-style debug variants, `-v` versions — see `jbase_universe_guide.md` §3.6 for the compile→catalog workflow and `../banking/t24_programming_guide.md` §2.5 for the T24 "COB" confusion (in T24, "COB" = catalog + the compile step, unrelated to COBOL).
- **UniVerse.** `basic` compiles UniVerse BASIC source into native host executables (UniVerse's own code generator; *flag:* the C-intermediary detail is vendor-internal — verified fact is "compiled objects run as host executables", `jbase_universe_guide.md` §4.7). `catalog` performs the same installation role; UniVerse also has `basic -d`-type debug compile options (the community references UniData's `basic -D` switch for debug builds — verified via BP Forms docs; UniVerse's flag set is similar but version-specific, flag).
- **UniData.** `basic` with the UDBASIC compiler; the `-D` debug switch is the documented community convention (verified via BP Forms documentation).
- **Compiled vs interpreted — settled.** Both dialects are **compiled** languages; neither is interpreted. jBC → C → native; UniVerse BASIC → native executables. The "interpreted" feel comes from the shells (`jsh`, `uv`) which execute commands and can run quick statements, and from `EXECUTE`-style dynamic calls — but production programs in both worlds are compiled and catalogued. (The TAFJ twist — jBC *recompiled to Java bytecode* — is the third model; see `../banking/tafj_guide.md` §2.)

### 5.2 Editors

- **jED — the jBASE editor (verified).** "The jED editor is a full screen, context sensitive, screen editor. It has been specifically designed so that users will find it easy to learn and use and is the preferred editing tool for the jBASE-operating environment" (jBASE manuals). jED is the day-to-day editor for jBC source in jBASE shops; its command set is PICK-editor-flavoured (line commands, `I`/`D`/`R`/`C` line ops, full-screen mode).
- **ED — the UniVerse editor (verified).** UniVerse ships the **UniVerse Editor** (invoked as `ED`), documented in the "Guide to the UniVerse Editor" ("allows users to modify UniVerse files or programs"; reference pages for all editor commands). It is the same editor family as PICK's `ED` and Prime's editor — the shared lineage is visible in the command set. *Flag on the task's "the 'E' (the UniVerse editor — the 'UV' editor — verify)":* the canonical invocation is **`ED`**; `E` is at most a site-defined synonym (VOC aliases can map `E` → `ED`, but no verified default `E` command exists — flag).
- **Modern editing.** Rocket **MV BASIC for VS Code** (MVVS) is the current cross-dialect editor: syntax highlighting, compilation, and debugging for UniVerse, UniData, and jBASE from one IDE (verified: "The Rocket MV BASIC extension can also connect to a U2 or jBASE server"; feature announcements cite UniData 8.2.4+, UniVerse 11.3.4/12.1.1-era support). For T24 projects, Temenos's **T24 Design Studio** (Eclipse-based) is the framework-level IDE (see `../banking/t24_programming_guide.md` §4.2).

### 5.3 Debuggers

- **jBASE.** jBC programs can be run under the **jBASE debugger**, which supports breakpoints, single-stepping, variable inspection, and dynamic-array drilling. *Flag:* the invocation surface is version-dependent — the documented entry points are the `DEBUG` statement/verb and the `JBC_DEBUG` environment variable (jBASE 5.x/6.x manuals); the exact flags (`basic -J`, debugger commands `S`/`G`/`P`…) vary, so verify against the release's debugger chapter.
- **UniVerse.** UniVerse BASIC has its own **debugger** (compile with debug options, then break into it from a running program); the modern path is the **MV BASIC VS Code debugger**, which attaches to U2 servers (verified: MVVS "Step-in may terminate debugging session" caveat for U2 in the v1.7.0 release notes — a real-world reminder that U2 debugging is less polished than jBASE's in VS Code). UniData debug builds use the `-D` compile switch (verified via BP Forms).
- **T24 note.** In the T24 world, jBC debugging on TAFC happens through jBASE's debugger plus Temenos's `DEBUG` routines (`DEBUG.INFO`, `DEBUG.SELECT` — see `../banking/t24_programming_guide.md` §5); on TAFJ it is JVM-level (Eclipse/IntelliJ attach).

### 5.4 The Development Environment: Rocket MV Tools and "MVDev"

- **Rocket MV BASIC for VS Code (MVVS)** — the unified editor/compiler/debugger front-end for all three dialects (verified; free from the VS Code marketplace). Its connection settings explicitly target "a U2 or jBASE server" — the practical answer to "one IDE for both dialects".
- **"MVDev" — flag.** The task's "Rocket MV Developers? ('MVDev')" resolves as follows: Rocket's developer community is the **Rocket MV Developers** community (community.rocketsoftware.com, the "multivalue-tools" forum where MVVS releases are announced), and "MVDev" as a standalone product name could not be verified — treat it as shorthand for the Rocket MV developer community/tooling, not a specific IDE.
- **The wider Rocket MV toolset** (from `jbase_universe_guide.md` §2.4): **MVIS** (free REST/JSON integration server), **MVConnect** (REST/graphQL middleware), **AccuTerm** and **wIntegrate** (terminal emulation), **U2 Web DE** (web dev environment), **SB+/SystemBuilder** (the U2 4GL), **MVDashboard** (dashboards), **MVX Performance** (monitoring). The jBASE-specific set: `jsh`, `jED`, `jlogadmin`, `jbackup`, `jrestore`, `jshow`, `jlicense`, `jbase_env`, `jdiag`, `jkeyauto`.

### 5.5 Administration

- **jBASE admin.** jBASE runs as ordinary OS processes — no database server daemon — so "administration" is a mix of OS-level and jBASE-level tasks: `jbase_env` (environment/account setup), `jlicense` (licensing), `jshow` (licence/version status), `jlogadmin` (transaction journal admin), `jbackup`/`jrestore`, `CREATE.FILE`/`RESIZE` (file sizing), and the service layer (`jAgent`, `jREST`) managed like any OS service. Licensing is per-server/per-user (verified; see `jbase_universe_guide.md` §3.2).
- **UniVerse admin.** The **`uv` command** is the UniVerse shell (verified — `uv` is how you enter the environment from Unix); administration happens in the **`sysadmin` account** (the UniVerse administrative login) with `CREATE.FILE`, `RESIZE`, `uvbackup`, and the U2 admin utilities (see `jbase_universe_guide.md` §4.10). *Flag on the task's "uvadmin":* **`uvadmin` is not a standard UniVerse command** — the verified names are `uv` (shell) and the `sysadmin` account; "uvadmin" appears in third-party tooling names (e.g. some ODBC/DataStage admin utilities), not in UniVerse's own command set.
- **Cross-dialect admin reality:** two engines in one estate means two admin models (process-based jBASE vs account-based UniVerse), two backup regimes (TJ + `jbackup` vs `uvbackup`), and two licensing schemes — a real operational cost of mixed environments (§4.5).

---

## 6. The T24 Context

### 6.1 jBC Inside T24/Transact

T24/Transact — the world's most deployed core-banking platform — is **written in jBC**. The application code (routines, subroutines, `I_` validations, versions, enquiries) is jBC compiled by **TAFC** (the Temenos Application Framework in C, running on jBASE) or executed by **TAFJ** (the Java framework — see `../banking/tafj_guide.md`). The T24-specific dialect surface — the `I_` naming, `V$`/`R$` record templates, `EB.` utilities, the version/`ID.NEW`/`AUTHORISATION` machinery — is the subject of `../banking/t24_programming_guide.md` (its §2 covers the jBC language in T24; §2.5 the compile/catalog/COB workflow) and `../banking/temenos_data_model_guide.md` (the FBNK file model). For this guide, the point is: **every T24 developer is a jBC developer first** — and the jBC they write is plain jBC plus the T24 framework layer, compilable in principle (though never in practice) outside T24.

### 6.2 Why jBASE? (The 1999 Acquisition)

The engine history is the answer: **T24 was originally built on UniVerse**. Temenos — itself a jBASE VAR — migrated T24 to jBASE and then bought jBASE (1999) so the platform would run on an engine it owned. jBASE's own corporate history states it plainly: "Since replacing UniVerse and embedding jBASE in their T24 banking application…" (verified quote, cited in `jbase_universe_guide.md` §3.12). The strategic logic, per the verified history: (1) **control** — an engine Temenos could evolve for banking workloads; (2) **open connectivity** — jEDI's external-device interface let T24 run on Oracle/DB2/SQL Server as well as jBASE files, a differentiation UniVerse's closed model didn't offer; (3) **performance/cost** — native compilation and a licensing model suited to embedding. The 2015 Zumasys deal preserved Temenos's perpetual right to ship jBASE inside Transact, which is why the jBASE-in-T24 relationship survives Rocket's ownership (§2.3).

### 6.3 InfoBasic in the Banks

While T24 owns the *productised* banking headline, the U2 family holds a vast **custom-installed base** in financial services: insurance policy administration systems, legacy retail-banking applications, wealth/private-banking platforms, and treasury back-offices written in UniVerse/UniData BASIC through the 1980s–2000s and never migrated. These systems are the classic "zombie core" profile: mission-critical, stable, running on 11.x/12.x-era UniVerse, maintained by a shrinking pool of UniBasic skills, and fed into modern stacks via U2 ODBC/JDBC, UniVerse SQL, or MVIS REST bridges (`jbase_universe_guide.md` §8). For a bank architect, the practical consequence: **both dialects are in your estate's past** — jBC via T24, UniBasic via legacy U2 applications — and the integration patterns (SQL bridges, REST, extracts) are the same for both.

### 6.4 T24 on jBC vs Legacy UniVerse Applications: The Comparison

| Dimension | T24/Transact on jBC | Legacy U2 app on UniBasic |
|---|---|---|
| Language | jBC + T24 framework (`I_`, versions, OFS, AA) | UniVerse BASIC (+ optionally SB+) |
| Engine | TAFC (C/jBASE) or TAFJ (Java) | UniVerse 11.x/12.x/14.x |
| Data model | FBNK files, `$NAU`/`$HIS`, concat files | Custom vertical file layouts |
| Development | T24 Design Studio, Model Bank, version-controlled framework | `ED`/VS Code MV BASIC, catalogued subroutines |
| Modernisation | TAFJ, Banking Cloud, REST via T24 APIs | MVIS/MVConnect, UniVerse SQL/ODBC |
| Skills market | Large (T24 SI ecosystem) but Temenos-centric | Small and ageing (UniBasic generalists) |
| Vendor | Temenos (application) + Rocket (engine) | Rocket (both) |

The strategic lesson for a solution architect: the **language** is the least interesting difference between these two worlds; the **framework and the data model** are the moat. Porting a T24 routine to UniBasic is pointless (T24 *is* the framework); porting a UniVerse app to jBC is a real but bounded project (§4.2); and the 2020s conversation is mostly about *bridging* both to REST/SQL/cloud rather than translating either (`jbase_universe_guide.md` §7–§8).

---

## 7. Worked Example: Porting a Routine

### 7.1 The Source: A Small InfoBasic Routine

A deliberately small UniVerse BASIC program: open a CUSTOMER file, select a subset of keys, read each record, print a formatted line, and trap a read error. Classic UniBasic idioms used: `@AM` delimiter variables (UniVerse also accepts `@FM` as the historical synonym, but the canonical name is `@AM` — we use `@AM`), `STATUS()` after `READ`, `EXECUTE ... CAPTURING` for the select, `MATREAD` for matrix access, and `OCONV`/`FMT` for output.

```basic
* UV.CUST.REPORT — UniVerse BASIC (InfoBasic) version
    OPEN 'CUSTOMER' TO F.CUST ELSE
        CRT 'Cannot open CUSTOMER'
        STOP
    END
    EXECUTE 'SELECT CUSTOMER WITH STATUS = "ACTIVE"' CAPTURING OUT
    SELECTLIST = OUT
    CRT 'Keys selected: ' : DCOUNT(SELECTLIST, @AM)
    LOOP
        READNEXT KEY FROM SELECTLIST ELSE EXIT
        READ REC FROM F.CUST, KEY THEN
            MATREAD MAT.ROW FROM F.CUST, KEY ELSE
                CRT 'MATREAD failed for ' : KEY
                CONTINUE
            END
            BALANCE = REC<40>
            CRT KEY : ' | ' : REC<2> : ' | ' : FMT(OCONV(BALANCE,'MD2'),'R,#20')
        END ELSE
            CRT 'READ failed: ' : KEY : ' status=' : STATUS()
        END
    REPEAT
    CRT 'Done. Rows=' : @SELECTED
    STOP
```

### 7.2 The Identified Dialect Differences

1. `EXECUTE ... CAPTURING` — UniVerse-specific output capture; jBC captures differently (§3.1).
2. `@AM` vs `@FM` — same character, different canonical name.
3. `STATUS()` post-`READ` — works in jBC, but jBC's modern idiom is `SETTING err`.
4. `MATREAD ... MAT.ROW` — jBC supports `MATREAD`, but the zero-element matrix convention and `MAT` dimensioning must be checked.
5. `READNEXT ... FROM LIST ELSE EXIT` — `EXIT` inside `LOOP`/`REPEAT` is standard in both.
6. `@SELECTED` — both; count semantics identical here.

### 7.3 The Ported jBC Version

```basic
* JBC.CUST.REPORT — jBASE BASIC (jBC) version
    OPEN 'CUSTOMER' TO F.CUST ELSE
        CRT 'Cannot open CUSTOMER'
        STOP
    END
    EXECUTE 'SELECT CUSTOMER WITH STATUS = "ACTIVE"'
    SELECTLIST = @SELECTED       ;* jBC: select-list id, or use SAVELIST
    * jBC capture idiom: the selected keys come back via the list,
    * not via CAPTURING; count is in @SELECTED
    CRT 'Keys selected: ' : @SELECTED
    LOOP
        READNEXT KEY FROM SELECTLIST ELSE EXIT
        READ REC FROM F.CUST, KEY SETTING ERR THEN
            MATREAD MAT.ROW FROM F.CUST, KEY ELSE
                CRT 'MATREAD failed for ' : KEY
                CONTINUE
            END
            BALANCE = REC<40>
            CRT KEY : ' | ' : REC<2> : ' | ' : FMT(OCONV(BALANCE,'MD2'),'R,#20')
        END ELSE
            CRT 'READ failed: ' : KEY : ' status=' : ERR
        END
    REPEAT
    CRT 'Done. Rows=' : @SELECTED
    STOP
```

The port is ~5 changed lines out of 22: `CAPTURING` handling, `@FM` (used implicitly here — jBC's canonical), `SETTING ERR` in place of `STATUS()`. This is the honest 80%: mechanical, reviewable, low-risk. The 20% would appear in a bigger routine: locking (`READU`/`LOCKED` semantics), `EXECUTE` of UniQuery vs jQL syntax differences, `@USERNAME` handling, and any embedded UniVerse SQL.

### 7.4 The Reverse: jBASE → UniVerse (Sketch)

```basic
* UV.CUST.REPORT — reverse sketch: jBC idioms needing UniVerse fixes
    EXECUTE 'SELECT CUSTOMER WITH STATUS = "ACTIVE"'   ;* jBC-style
    * → UV: EXECUTE 'SELECT ...' CAPTURING OUT
    READ REC FROM F.CUST, KEY SETTING ERR THEN         ;* jBC-style
    * → UV: READ ... THEN ... ELSE  + STATUS() check
    IF obj->balance GT 1000 THEN                      ;* jBC dynamic object
    * → UV: no objects — rewrite with a dynamic array or MAT
```

Directionally: the jBC→UniVerse direction is harder whenever the source uses jBC-only extensions (objects, `CALLJ`, threads); for plain PICK-core code it is the same mechanical exercise in reverse.

### 7.5 The "PICK.BASIC" Subset: Code That Runs on Both

Written to the common core (§4.1), the routine compiles on both engines with zero dialect-only constructs:

```basic
* PICK.BASIC — the portable subset (compiles on jBASE AND UniVerse)
    OPEN 'CUSTOMER' TO F.CUST ELSE
        CRT 'Cannot open CUSTOMER'
        STOP
    END
    EXECUTE 'SELECT CUSTOMER WITH STATUS = "ACTIVE"'
    LOOP
        READNEXT KEY FROM @LISTS ELSE EXIT        ;* @LISTS: both
        READ REC FROM F.CUST, KEY THEN
            BALANCE = REC<40>
            CRT KEY : ' | ' : REC<2> : ' | ' : FMT(OCONV(BALANCE,'MD2'),'R,#20')
        END ELSE
            CRT 'READ failed: ' : KEY
        END
    REPEAT
    STOP
```

Caveats kept deliberately out of the "portable" version: the `@LISTS` select-list handle (both dialects expose a default active list, but the *named-list* APIs differ), status reporting (omitted — both sides tolerate omission here), and `EXECUTE` capture (omitted — the selected keys are consumed via the list, not captured text). This is the realistic shape of portable MV BASIC: **it exists, and it is deliberately modest.**

---

## 8. The Future: 2026 and Beyond

### 8.1 The Rocket Consolidation

Since 2021 both dialects have had one vendor. The observable convergence signals, verified in `jbase_universe_guide.md` §2.4/§10: **MV BASIC for VS Code** speaks to all three engines from one IDE; **MVIS** exposes U2 and jBASE data through one REST layer; the **product lifecycle** commitment runs to 2029–2033 across the lines. But *convergence of tooling is not convergence of languages*: as of 2026, Rocket shows **no sign of merging the compilers** — jBC (C-compiled, objects, classes) and UniVerse BASIC (its own compiler, `MAT`-canonical, no objects) remain separate products with separate manuals, and the company's stated roadmap is maintenance-plus-modernisation, not unification. *Flag:* any "Rocket will merge jBC and UniBasic" claim is speculation; the verified posture is one portfolio, two dialects, shared tooling.

### 8.2 The TAFJ / Java Era

The biggest single change in the jBC world is **jBC → Java**: TAFJ compiles T24's jBC to Java bytecode on a JVM, and Temenos's strategic direction (R23+, Banking Cloud) is TAFJ-first, with TAFC/jBASE as the legacy-but-huge option (`../banking/tafj_guide.md` §1.4–1.5, §8). The dialect-level significance: **the jBC language outlives the jBASE engine** — the same T24 jBC source runs on jBASE (TAFC) or the JVM (TAFJ), which is the strongest possible proof that jBC is a language with multiple runtimes. UniBasic has no analogous second runtime; UniVerse is UniVerse.

### 8.3 The Declining BASIC Skills Pool

The honest trend, from `jbase_universe_guide.md` §10: MV BASIC skills are **ageing and not being replaced at scale** — new development in MV BASIC is near zero, and the pool of T24 jBC and UniBasic developers is dominated by 20–30-year veterans. The countervailing signals: (1) T24's scale (1,000+ banks) keeps jBC demand alive through the SI ecosystem; (2) **LLM-assisted comprehension** is genuinely changing the economics of legacy MV code (a model can read `^`-delimited records and BASIC dialect code fluently — this guide series is itself evidence of the appetite); (3) Rocket's tooling investment (MVVS, MVIS) is aimed at making the existing base serviceable, not at growing the pool.

### 8.4 Trends Summary

| Trend | Direction | Signal |
|---|---|---|
| Rocket consolidates ownership | ↑ | jBASE + U2 + OpenQM under one vendor since 2021 |
| Language convergence | → | Shared tooling (MVVS, MVIS), but two compilers, no merger announced |
| jBC → Java (TAFJ) | ↑ | TAFJ-first Transact; jBC outlives the jBASE engine |
| UniVerse/UniData maintenance | → | 14.x/8.x lines supported to 2029–2033; no strategic rewrite |
| MV BASIC skills pool | ↓ | Ageing; LLM-assisted comprehension emerging |
| Cross-dialect porting demand | → | Driven by consolidations and T24 migrations, not new build |
| Bridge-first integration (REST/SQL) | ↑ | MVIS, U2 ODBC/JDBC, jBASE SQL — the money is in the bridges |

The verdict this guide has argued throughout: **jBC and InfoBasic are two dialects of one language, sharing a common ancestor, a common data model, and now a common vendor — but with different futures.** jBC's future is Temenos-shaped (Java, cloud, TAFJ); InfoBasic's future is installed-base-shaped (maintenance, bridges, eventual migration). If you must pick one to learn in 2026, learn the common core first — it is portable by construction — then specialise by ecosystem: jBC for the T24 world, UniBasic for the U2 installed base.

---

## 9. Glossary

| Term | Definition |
|---|---|
| **jBC** | jBASE BASIC — the compiled programming language of the jBASE database; compiles via C to native code. |
| **jBASE BASIC** | The official Rocket name for jBC. |
| **InfoBasic** | The family label for the U2 BASICs (from Prime INFORMATION's INFO/BASIC); UniVerse BASIC + UniData BASIC. |
| **UniVerse BASIC (UniBasic)** | The BASIC dialect of UniVerse; VMark/IBM/Rocket lineage. |
| **UniData BASIC (UDBASIC)** | The BASIC dialect of UniData; the SQL-mapping flavour of U2. |
| **PICK BASIC** | The common ancestor dialect, from the Pick operating system (1973); the portable core both dialects share. |
| **PICK** | Dick Pick's operating system + database (1973); the root of the MultiValue world. |
| **Prime INFORMATION** | Prime Computer's MultiValue database (1979–80s, on PRIMOS); home of INFO/BASIC. |
| **INFO/BASIC** | The BASIC of Prime INFORMATION (1980 manual verified). |
| **VMark** | The company that created UniVerse (1984). |
| **Informix** | Owner of the U2 line 2000–2001 (via Ardent 1997). |
| **IBM** | Owner of U2 2001–2009 (the "IBM U2" era). |
| **Rocket** | Rocket Software — owner of U2 since 2009 and jBASE since 2021. |
| **@FM / @AM** | Field/attribute mark 0xFE — jBC's `@FM` vs UniVerse's `@AM` (same character). |
| **@VM** | Value mark 0xFD — repeating value separator (same in both). |
| **@SM** | Sub-value mark 0xFC — sub-value separator (same in both). |
| **Dynamic array** | An MV string: one record as a delimiter-separated array (`record<1,2,3>`). |
| **OCONV / ICONV** | Output/input conversion functions — formatting and parsing (same in both). |
| **FMT** | Format function using dictionary-style masks. |
| **DCOUNT** | Count delimited elements. |
| **FIELD / INDEX** | Extract nth element / find nth occurrence. |
| **EREPLACE** | Regex-based replace — present in both dialects (verified). |
| **MATPARSE / MATBUILD** | Split a dynamic array into a matrix / rebuild it — both dialects. |
| **READ / WRITE** | The core record I/O verbs (same in both). |
| **READU / READV / WRITEV** | Locked read / read one attribute / write one attribute. |
| **READNEXT** | Consume the next key from a select list — both dialects. |
| **SELECT / SSELECT** | Build a (sorted) select list of keys. |
| **OPEN** | Open a file to a file variable. |
| **GOSUB / GO SUB** | Subroutine call — `GOSUB` is the keyword in both MV dialects; the dual spelling exists in other BASIC lineages only. |
| **LOOP / REPEAT** | The flexible loop construct (WHILE/UNTIL anywhere) — both dialects. |
| **BEGIN CASE / END CASE** | The CASE construct — same in both. |
| **jED** | The jBASE full-screen editor (verified). |
| **ED / UE** | The UniVerse editor (`ED`; "UE" is the UniVerse Editor in Rocket docs). |
| **EMULATE** | jBASE's cross-dialect compatibility mode (`Config_EMULATE`, `JBASE_EMULATE`). |
| **Catalog** | Install a compiled program into a library so it can be called by name. |
| **Compile** | Turn BASIC source into an executable (`basic` in both dialects). |
| **TAFC** | Temenos Application Framework in C — T24's jBC runtime on jBASE. |
| **TAFJ** | Temenos Application Framework in Java — T24's jBC→Java runtime. |
| **T24 / Transact** | Temenos's core-banking platform, written in jBC. |
| **MVDev** | *Flag:* practitioner shorthand for Rocket's MV developer community/tooling; not a verified standalone product. |

---

## 10. References and Verification Trail

**Sibling guides in this repo:**

- [jbase_universe_guide.md](jbase_universe_guide.md) — the database-family umbrella: MV data model, PICK heritage, jBASE/UniVerse/UniData deep-dives, ownership chains (verified), current versions, ecosystem (§2.3–2.4, §3, §4, §6, §10) — the primary cross-reference for this guide
- [t24_programming_guide.md](../banking/t24_programming_guide.md) — jBC in the T24 context (§2), routines, the Development Framework, the "COB" compile/catalog workflow (§2.5)
- [temenos_data_model_guide.md](../banking/temenos_data_model_guide.md) — the T24 application model (FBNK files, versions, OFS, AA)
- [tafj_guide.md](../banking/tafj_guide.md) — the jBC→Java runtime, TAFJ vs TAFC (§1–§2, §8)
- [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) — the modern schema-on-read context (§1, §9)

**Primary sources used for verification (Aug 2026):** Prime Computer, *INFO/BASIC Programming Language Reference Manual for the PRIME INFORMATION Computer System* (PDR 3903, ©1980; WorldCat OCLC 223321054); Wikipedia "Prime Computer" and "Pick operating system"; Rocket jBASE 6.x documentation (including "Emulation (revised)", Config_EMULATE, jED editor manual); Zumasys jBASE 3.0-era manuals (emulation options incl. `conv_mct_uv_compat`); Rocket UniVerse BASIC User Guide (11.x PDF edition) and UniVerse BASIC Commands Reference (14.1.1; EREPLACE, MAT verbs, LOOP…REPEAT); Rocket MV BASIC for VS Code release announcements (v1.7.0, v1.10.1) and connection documentation; BluePinc BP Forms documentation (cross-dialect compile switches); mannyneira.com UniVerse BASIC tutorials (LOOP/REPEAT); Manny Neira "The Editor — ED"; the Rocket Software Product Lifecycle Handbook (Nov 2025) as cited in `jbase_universe_guide.md` §2.4/§3.3; jBASE corporate history and Temenos acquisition coverage as cited in `jbase_universe_guide.md` §3.2.

> **Honesty footer:** Version numbers (jBASE 6.2.2, UniVerse 14.2.1, UniData 8.4.1), ownership dates, and lifecycle horizons are as verified in August 2026 against the sources above and `jbase_universe_guide.md`; they change frequently — verify against rocketsoftware.com for the exact release before architecture or procurement decisions. Items flagged in-line: the Prime→VMark personnel transfer; the exact jBASE 6.x emulation label list and TRY/CATCH surface; jBASE's debugger invocation details; UniVerse compile-flag specifics; "MVDev" as a product name; porting-effort percentages (practitioner consensus, not measured); and any "dialect merger" claims (none found). The honest bottom line: **two dialects of one language, one data model, one vendor since 2021 — and porting between them is real, bounded, mostly-mechanical work that this guide exists to make cheaper.**
