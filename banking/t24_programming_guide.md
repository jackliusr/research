# T24 / Transact Programming: The Temenos Development Deep-Dive — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Core Banking / Temenos — how T24/Transact is actually *programmed*: the jBC language in the T24 context, the application types and file structure, the Development Framework, the routine patterns, the integration programming (OFS/APIs), and the development lifecycle
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Scope and How This Guide Fits the Series

This guide is the **dedicated deep-dive on T24/Transact *programming*** — the developer's view of the Temenos core-banking platform. It deliberately does *not* re-hash the rest of the Temenos series; where a topic already has a dedicated guide, this document cross-references it by filename:

- The vendor, the product portfolio, the Transact architecture, releases and the Model Bank → `temenos_guide.md`
- The T24/Transact data model — jBASE/MultiValue records, `$NAU`/`$HIS` companion files, the FBNK tables, the AA arrangement architecture → `temenos_data_model_guide.md`
- The **TAFJ** engine — the Java runtime that compiles jBC to Java bytecode (the alternative to the C engine) → `tafj_guide.md`
- The **jBASE** engine and the **jBC** language family (PICK BASIC, dynamic arrays, jED, catalog) → `../technology/jbase_universe_guide.md`
- Posting and accounting: OFS → FUNDS.TRANSFER → STMT.ENTRY, the entry lifecycle → `posting_engine_core_banking_guide.md`
- Interest engines (the AA INTEREST property) → `interest_engines_core_banking_guide.md`
- The transaction lifecycle and Close of Business (COB/EOD) batch processing → `core_banking_processes_guide.md`
- The core-banking vendor landscape and the "eras" of core systems → `core_banking_systems_guide.md`

**Verification note — read this first.** T24 development documentation is **proprietary**: the authoritative sources are Temenos' own training decks ("T24 Application Development", "Application Structure and Files", "Template Programming", "T24 Technical Kick-Start"), the partner-facing **Knowledge Base** and **Base Camp** community, and `developer.temenos.com` — most requiring a Temenos/partner login. Copies of the training decks leak into the public domain (Scribd/PDFCOFFEE), and the practitioner ecosystem (t24ALL, t24guru, consultant blogs, Base Camp threads) documents the day-to-day reality. Where a fact is verified against such a source it is stated plainly; where it is inferred, hearsay, or marketing it is **flagged**. Anything flagged "flag" is directionally true but not independently confirmable against Temenos-official documentation.

---

## Table of Contents

1. [The T24 Programming Landscape](#1-the-t24-programming-landscape)
2. [The jBC Language](#2-the-jbc-language)
3. [The T24 Application Structure](#3-the-t24-application-structure)
4. [The T24 Development Framework](#4-the-t24-development-framework)
5. [The T24 Programming Patterns](#5-the-t24-programming-patterns)
6. [The Development Lifecycle](#6-the-development-lifecycle)
7. [Worked Example: A Simple T24 Customisation](#7-worked-example-a-simple-t24-customisation)
8. [The Future: 2026 and Beyond](#8-the-future-2026-and-beyond)
9. [Glossary](#9-glossary)
10. [References and Verification Trail](#10-references-and-verification-trail)

---

## 1. The T24 Programming Landscape

### 1.1 The Platform: Application + Engine

The first thing a T24 developer must internalise is the **application/engine split**. "T24" (rebranded **Temenos Transact** in 2018 — see `temenos_guide.md` §4) is the *banking application*: thousands of application modules and routines written in **jBC** (jBASE BASIC, "T24 BASIC"). That code runs on one of two engines:

```
        TEMENOS TRANSACT (the application — formerly T24)
        ~4,000+ application modules written in jBC
        ┌─────────────┬─────────────┐
        ▼             ▼             ▼
   TAFC engine   TAFJ engine   (the engines — see tafj_guide.md)
   (C, on        (Java, on
   jBASE)        the JVM)
        ▼             ▼
   jBASE MV DB   Relational DB via JDBC
```

- **TAFC** — the Temenos Application Framework in C: jBC interpreted natively on the jBASE MultiValue database (the "classic" engine).
- **TAFJ** — the Temenos Application Framework for Java: jBC *compiled to Java bytecode*, executed on the JVM against a relational backend (Oracle, SQL Server, MySQL…).

The same jBC source compiles and runs on both — that is the point of the framework design. See `tafj_guide.md` §3 for the full engine comparison, and `temenos_data_model_guide.md` for the data side.

### 1.2 The Programming Model: Configuration-First + Code

T24 development is **configuration-first with a code escape hatch**. The overwhelming majority of a bank's "development" on T24 is done *without writing code*:

- **Versions** — customised input screens over existing applications (which fields appear, which are hidden, which default, which are authorised). No code.
- **Enquiries** — query/report definitions (selection criteria, display fields). No code.
- **Templates / applications** — table definitions; the framework auto-generates the input program, the file, and the validation skeleton from a template definition.
- **Local reference fields** (`LOCAL.REF`) — bank-specific fields bolted onto standard applications without touching core code.
- **Standard selection / data dictionary** — field lists and lookup rules.

The Temenos training material is explicit about this: *"T24 has an extensive infrastructure in place that enables the rapid development of business components. The creation and maintenance of T24 applications is based on a series of templates."* (verified — T24 Application Development Overview). This is the **no-code/low-code core** of the platform.

**The code layer** is **jBC routines** — the "traditional" T24 development: input routines, validation routines, pre/post processing, batch programs, enquiry routines, OFS handler routines. When configuration cannot express the business rule, you write a routine, attach it to a field, a version, or an application, and compile/catalog it. Since the TAFJ era (R10+), an additional code layer exists: **Java extensibility** — Java hook routines callable *from* jBC via `CALLJ` (see `tafj_guide.md` and §5.3 below).

> **The 80/20 rule of T24 development.** Practitioner consensus (flag — not a Temenos statistic): roughly 80% of a typical implementation's "customisation" is configuration (versions, enquiries, local fields, product parameters) and only ~20% is actual jBC/Java code. The ratio inverts for complex treasury/payments integrations, which is where the routine skills pay.

### 1.3 The "T24 Developer": Roles and Skills

| Role | What they do | Tools |
|---|---|---|
| **T24 (technical) developer** | Writes jBC routines, compiles/catalogs, builds versions and enquiries, OFS integration, batch programs | jED / Design Studio, TAFJ debugger, jBASE commands |
| **Functional analyst / configurator** | Configures products, versions, fields, local references, parameters; understands banking semantics deeply | T24 Browser / Data Entry screens, version builder, product parameters |
| **Technical / solution architect** | Environments, engines (TAFC vs TAFJ), deployment, performance, Java extensibility, integration architecture (OFS, IRIS/TWS, APIs), upgrades | Everything, plus Java/J2EE, middleware |

**The skills stack** (the "full-stack T24 developer" profile, in rough order of value):

1. **jBC** (the language) — non-negotiable for any code-level work.
2. **T24 application knowledge** — the data model: which application holds what, record statuses (unauthorised/live/history), the `$NAU`/`$HIS` companion files, the AA architecture (`temenos_data_model_guide.md`).
3. **The configuration suite** — versions, enquiries, local fields, templates, standard selection.
4. **OFS** — the integration protocol; every external system integration ends in an OFS message (`posting_engine_core_banking_guide.md`).
5. **TAFC/TAFJ operational knowledge** — compilation, catalog, runtime, batch.
6. **Java** — increasingly required for TAFJ-era extensibility (`tafj_guide.md` §4, §8).

### 1.4 The Documentation Reality

There is **no public, complete T24 developer manual**. The documentation landscape:

- **Temenos Knowledge Base / Base Camp** — the partner/customer support portal; contains the real documentation, release notes, and the developer community (Q&A, articles). Requires a Temenos account (bank/partner login).
- **developer.temenos.com** — the public developer community site; free to register, hosts articles and Q&A, and links to the cloud sandbox developer workflows.
- **Training decks** — "T24 Application Development", "Application Structure and Files" (R17), "Template Programming", "T24 Technical Kick-Start", "T24 Extensibility in Java" — circulate as leaked PDFs on Scribd/PDFCOFFEE and are the de-facto public reference.
- **The practitioner ecosystem** — t24ALL, t24guru, consultant blogs (performetriks, wikibanking), Stack Overflow, LinkedIn posts.

**Flag:** anything you read publicly about T24 internals is either a leaked Temenos deck, a consultant's summary, or community knowledge — treat precision with caution, and verify against the installed release where it matters.

---

## 2. The jBC Language

### 2.1 jBC: The Language

**jBC** — jBASE BASIC, marketed in the T24 world as **"T24 BASIC"** — is the programming language of the T24/Transact application. It is a member of the **PICK BASIC** family (the BASIC dialect of the MultiValue database world), extended by jBASE with dynamic arrays, file verbs, and the T24 runtime's conventions. The language itself is covered in depth in `../technology/jbase_universe_guide.md`; this section covers it *in the T24 context*.

Key properties relevant to T24 work:

- **No variable declaration** — variables spring into existence on first assignment; `$`-prefixed variables are common (e.g. `$INSTRUCTION`, `$FILE.NAME` in T24 routines).
- **Dynamic arrays** — the single most important concept: a variable can hold a multi-level delimited list (see §2.2). The entire T24 record model is built on this.
- **Case-insensitive** keywords; `*` starts a comment; `REM` also works.
- **The language is the application** — T24 is ~4,000+ modules of jBC. Reading jBC is how you learn what T24 actually does; writing jBC is how you change it.

### 2.2 Language Basics: Statements, Variables, Dynamic Arrays

A jBC program is a sequence of statements (`IF...THEN...ELSE`, `FOR...NEXT`, `LOOP...WHILE...REPEAT`, `BEGIN CASE...END CASE`, `GOSUB`), file verbs, and string/array functions (`FIELD()`, `CHANGE()`, `TRIM()`, `CONVERT()`, `ICONV()`/`OCONV()` for internal/external conversion of dates and amounts).

**Dynamic arrays and the MultiValue delimiters.** Every jBC variable can be a *dynamic array*: a single string with three levels of delimiter. T24 defines the system constants:

| Constant | Character | Name | Level |
|---|---|---|---|
| `@FM` | char 254 | Field mark | attribute (row) level |
| `@VM` | char 253 | Value mark | multi-value (MV) level |
| `@SM` | char 252 | Sub-value mark | sub-value (SV) level |

The T24 record model maps directly onto this: each *attribute* of a T24 record is one `@FM`-delimited field; repeating data inside a field is `@VM`-delimited (e.g. the many beneficiaries on a multi-payment FT); further structure is `@SM`-delimited. See `temenos_data_model_guide.md` §2 for the data-model view. T24 also provides `@AM` (attribute mark, synonym of `@FM`) for readability.

### 2.3 File I/O: OPEN, READ, WRITE, READU

File access in jBC is verb-based (no SQL). T24 routines do this constantly:

```jbc
OPEN "F.CUSTOMER" TO F.CUSTOMER ELSE
   CALL OFS.ERROR ("UNABLE TO OPEN F.CUSTOMER")
   STOP
END
```

- **`OPEN`** — opens a file (T24's `F.`-prefixed files are the physical hashed files; the jBASE `VOC` resolves the name). In TAFJ, the same verb maps to the JDBC file system (`tafj_guide.md` §2.6).
- **`READ`** — reads a record: `READ R.REC FROM F.FILE, ID ELSE ...`; on failure the record variable becomes `""`.
- **`WRITE`** — writes a record: `WRITE R.REC TO F.FILE, ID` (in T24, most *business* writes go through the OFS/input framework so that audit fields, `CURR.NO`, and the `$NAU`/`$HIS` lifecycle are maintained — see §6.2; raw `WRITE` is for system files and utilities).
- **`READU`** — read *with record lock* (the MultiValue concurrency model is record-level): `READU R.REC FROM F.FILE, ID LOCKED ... THEN ... ELSE ...` followed by `RELEASE` or `WRITE` (write releases the lock). TAFJ implements the same semantics over database locks (`tafj_guide.md` §2.7 — record locking).

Other verbs in daily use: `DELETE`, `READV`/`WRITEV` (read/write a single attribute), `MATREAD`/`MATWRITE` (map a record to a dimensioned array), `F.READ`/`F.WRITE` (jBASE "file variable" forms), and `COMMON` for shared variables.

### 2.4 The T24 Routine: Structure and Program Types

A T24 "routine" is a jBC program of one of three types:

| Type | Keyword | Purpose | Entry |
|---|---|---|---|
| Program | `PROGRAM` | Standalone executable (batch job, utility, report) | executed directly |
| Subroutine | `SUBROUTINE` | Callable routine (`CALL name(args...)`) — the overwhelming majority of T24 code | called, not run |
| Function | `FUNCTION` | Returns a value (`value = FN.NAME(args)`) | called in expressions |

Conventions every T24 codebase follows (verified against training material and practitioner code):

- Routine names are UPPERCASE, dot-separated (e.g. `FT.VALIDATE.DUPLICATES`, `EB.CLEAR.FILE`). A routine's filename must equal the routine name declared in the source.
- The T24 **common block** is the contract between the framework and your code: named common variables such as `V` (the number of fields in the record — *"should be set to Z + 9 for input applications and Z for display-only applications"*, per the Template Programming guide), `Z`, `R.RECORD`/`R.NEW` (the in-memory record), `APPLICATION`, `ID`, `COMI`, `TEXT`, and `E`/`ETEXT` (the error mechanism — set `E` and `ETEXT` to reject input). Getting these right is 80% of writing correct T24 routines.
- Every application has a **field-definition subroutine** — *"Every application MUST have a corresponding field definitions subroutine. The name of this subroutine is the full name of the application with the suffix `.FIELDS`, e.g. `CR.CAMPAIGN.OPPORTUNITY.FIELDS`"* (verified — T24 Application Development guide).

### 2.5 Compile and Catalog: The "COB" Confusion

The build pipeline for a jBC routine is:

1. **Edit the source** (in `BP` — the BASIC Program source file — or a `T24.BP`-style folder, via jED or Design Studio).
2. **Compile** — the `BASIC` verb (jBASE `jcompile`, which also invokes `jlink` for standalone programs). This produces object code (`.so`/`.dll` artifacts; jBASE refers to the object format as `$`-files).
3. **Catalog** — the `CATALOG` verb makes the compiled routine *callable by name* from any program or the T24 framework: `CATALOG T24.BP FT.VALIDATE.DUPLICATES`. The runtime resolves catalogued routines through the `JBCOBJECTLIST` environment variable; the catalog build area is controlled by `JBCDEV_LIB` (verified — Base Camp thread on duplicate catalog entries). De-catalog via `DECATALOG`.

**The "COB" terminology — verify with care.** This throws up a genuine ambiguity:

- **"COB" in T24 overwhelmingly means Close Of Business** — the end-of-day batch process (`core_banking_processes_guide.md`). A "COB job" is an EOD process, not a compiled routine.
- **"Compiled routine" is "catalogued code", not "COB"** — the artifacts are the jBASE catalog (shared libraries) and the `BP`-side object files; T24's own terminology for a code unit is *routine*, *compiled routine*, or *catalogued routine*.
- **Flag:** some older T24/TAFC documentation and install layouts use `COB` as the name of the *compiled-object storage file* (alongside `BP` for source, `LIB` for the catalog libraries). I could not verify this against a Temenos-official source — treat "COB = compiled object file" as unconfirmed practitioner lore; in a T24 context, default "COB" to *Close of Business*.

**TAFJ note:** on TAFJ the same jBC compiles to Java classes (`tafj_guide.md` §2) — the `BASIC`/`CATALOG` verbs are replaced by the TAFJ compiler tooling (Maven-based `t24-packager`), but the *source* and the *call semantics* are identical.

### 2.6 CALL and CALLJ

- **`CALL routine(args)`** — the standard jBC subroutine call; the workhorse of T24 (e.g. `CALL EB.READLIST(...)`, `CALL OFS.BULK.MANAGER(...)`).
- **`CALLJ`** — the jBC→Java bridge on TAFJ: invokes a **Java hook routine** (a class implementing Temenos' extensibility interfaces) from jBC source, passing T-type objects. See `tafj_guide.md` §4 and §5.3 below.

### 2.7 The Runtime: Online and Batch

T24 routines execute in two runtime contexts:

- **Online** — interactive: a user on the T24 Browser/Data Entry screen (or an external system via OFS/IRIS/TWS) drives a version; the input template fires field-level routines per field-exit, then record-level validation, then (if authorised) the write to the live file. Response-time-sensitive.
- **Batch (COB)** — the end-of-day Close of Business and other scheduled runs: `PROGRAM`-type jobs executed by the batch scheduler (jBASE jobs / the T24 `COB` control jobs: `BATCH`, `MULTITHREAD`). Same jBC, different lifecycle and performance profile (`core_banking_processes_guide.md`).

### 2.8 A jBC Routine Sketch

A small, realistic sketch — an FT version validation routine that checks a custom "payment purpose code" against a local parameter file, using `READU` and `WRITE`:

```jbc
SUBROUTINE FT.CUSTOM.VALIDATE
* Reads FT.CUSTOM.PARAM, checks the LOCAL.REF field
* LOCAL.PURPOSE. Error path: set E + ETEXT to reject input.
    $INSERT I_COMMON
    $INSERT I_EQUATE
    $INSERT I_F.VERSION

    FN.FT.CUSTOM.PARAM = "F.FT.CUSTOM.PARAM"
    F.FT.CUSTOM.PARAM = ""
    CALL OPF(FN.FT.CUSTOM.PARAM, F.FT.CUSTOM.PARAM)

    PURPOSE.CODE = R.NEW(FT.LOCAL.REF)<1, 1>   ;* local ref value
    IF PURPOSE.CODE EQ "" THEN RETURN            ;* nothing to check

    READU R.PARAM FROM F.FT.CUSTOM.PARAM, "SYSTEM" THEN
       VALID.CODES = R.PARAM<1>                  ;* @VM list of codes
       LOCATE PURPOSE.CODE IN VALID.CODES SETTING POS ELSE
          E = "FT.CUSTOM.INVALID.PURPOSE"
          ETEXT = "Purpose code ":PURPOSE.CODE:" is not valid"
          RETURN
       END
       R.PARAM<2, -1> = TODAY                    ;* log last use
       WRITE R.PARAM TO F.FT.CUSTOM.PARAM, "SYSTEM"   ;* releases lock
    END ELSE
       E = "FT.CUSTOM.PARAM.MISSING"
       ETEXT = "FT.CUSTOM.PARAM not found for SYSTEM"
    END
RETURN
END
```

This sketch is deliberately simplified — a real routine would use the T24 `I_F.VERSION`/`I_COMMON` include conventions and the standard `CALL`-based error handling. The `LOCAL.REF` mechanics are covered in §3.6 and the full worked example in §7.

---

## 3. The T24 Application Structure

### 3.1 The "Application" Concept

In T24, an **application** is a business entity plus its complete processing scaffold: not just a data file but the definition of its fields, its validation, its input program, and its behaviour. Every business entity is an application — `CUSTOMER`, `ACCOUNT`, `FUNDS.TRANSFER` (the payment/transfer), `STMT.ENTRY`, `AA.ARRANGEMENT`, plus thousands of parameter applications (`ACCOUNT.PARAMETER`, `FT.COMMISSION.TYPE`, …). See `temenos_data_model_guide.md` §2 for the data-model anatomy.

An application's runtime anatomy (verified — "Application Structure and Files" R17 deck and the data-model guide):

1. a **PGM.FILE** entry (the program/file definition — which template program drives it),
2. a **FILE.CONTROL** entry (multi-company behaviour, file type: live/`$NAU`/`$HIS`),
3. a **STANDARD.SELECTION** entry (the field list / data dictionary — §3.5),
4. the **I_** input program layer (the template-generated input/validation program, e.g. `I_FUNDS.TRANSFER`),
5. the **TEMPLATE** definition (the record layout — §3.4),
6. the **.FIELDS** subroutine (field definitions — §2.4).

Physically, the data lives in **`F.`-prefixed files** (`F.CUSTOMER`; multi-company installs use `FBNK.CUSTOMER` for company `BNK`) with the companion `$NAU` (unauthorised), `$HIS` (history), and `$DEL` (deleted) files (`temenos_data_model_guide.md` §2–3).

### 3.2 Application Types (the C/V/E/I/R/P/S Classification)

T24 classifies applications by **type**, and the type drives how the platform treats them. The standard classification taught in Temenos training (flag — verify against the installed release's `FILE.CONTROL`/`PGM.FILE` values; the letter prefixes below are the widely-used convention):

| Type | Letter | What it is | Examples |
|---|---|---|---|
| **Core / table** | `C` | The main entity applications; data tables with full input/validation | `CUSTOMER`, `ACCOUNT`, `FUNDS.TRANSFER` |
| **Version** | `V` | A customised *view* of a core application — a screen definition | `V.FT.NEW`, `V.FT.CUSTOM` (records live in the `VERSION` application) |
| **Enquiry** | `E` | Query/report definitions over one or more files | `E.ACCOUNT.STMT`, records in `ENQUIRY` |
| **Interface** | `I` | Message/interface definitions (external formats) | `I.FUNDS.TRANSFER` (message layouts) |
| **Report** | `R` | Report definitions (usually batch-generated) | `R.ACCOUNT.STATEMENT` |
| **Process** | `P` | Batch processes / COB jobs | `P.BATCH.DEBIT`, the `BATCH`/`BATCH.DETAIL` jobs |
| **Service** | `S` | Service definitions (SOA/web-service-visible operations) | `S.SERVICE.NAME` (TAFJ/IRIS-era) |

Practical consequence: a "V" application is not a new table — it is a record in the `VERSION` application pointing at a core application with screen rules. An "E" application is a record in the `ENQUIRY` application. The distinction between *entity* (C) and *view* (V/E/R) is the single most useful mental model for navigating a T24 codebase.

### 3.3 Application Classes: The Core Apps

The applications every T24 developer must know cold (deep-dived in `temenos_data_model_guide.md`):

- **`CUSTOMER`** — the party master (individuals, companies; the AA-era model adds the party as an arrangement component).
- **`ACCOUNT`** — the account master (currency, category, customer link, balance types). In the modern AA architecture accounts are `AA.ARRANGEMENT`s with property records.
- **`FUNDS.TRANSFER` (FT)** — the transfer/payment application; the posting engine's entry point (`posting_engine_core_banking_guide.md` §7.1). OFS messages addressed to `FUNDS.TRANSFER` drive debit/credit postings.
- **`STMT.ENTRY` / `CATEG.ENTRY`** — the accounting entry records generated from FT.
- **`AA.ARRANGEMENT`** — the modern product/agreement record.
- **Parameter apps** — `COMPANY`, `ACCOUNT.PARAMETER`, `LIMIT.PARAMETER`, `FT.COMMISSION.TYPE`, `LOCAL.TABLE` (the local-field definition application, §3.6).

### 3.4 TEMPLATE: The Table Definition

The **TEMPLATE** is the design-time definition of an application's record layout — the "application template" from which the framework generates the file, the input program, and the validation skeleton. Training material is explicit that T24 application creation is *template-driven* (§1.2). A template definition specifies, per field:

- the **field name** and **number** (position in the record — T24 fields are positional: `DEBIT.ACCT.NO` is attribute 2 of `FUNDS.TRANSFER`, etc.),
- the **field type / input type** (see §3.5),
- whether it is **single or multi-valued** (the "S/M" validation flag — verified: *"S - data field is single valued; M - data field is multi-valued"*),
- the **validation** to apply (a check routine, a lookup table, a format rule).

**Flag:** the *exact* template mechanics (which fields of the `TEMPLATE` application hold what) vary across releases — R14-era decks describe "THE.TEMPLATE"/"TEMPLATE" code records; later releases use the "application designer" tooling (§4). The concept — *define the layout once, the framework generates the rest* — is stable.

### 3.5 Fields and the STANDARD.SELECTION (SS)

**STANDARD.SELECTION** (colloquially "standard selection", "SS", or the *data dictionary*) is the application's field catalogue: the list of fields on an application, their types, and how enquiries and versions may use them. Verified from the T24 STANDARD.SELECTION data-dictionary material, each field definition carries a **field type**:

| Type | Meaning |
|---|---|
| `D` | Data field — an actual stored attribute |
| `C` | Concat(enated) file — assembled from another file |
| `I` | Dictionary I-type — a calculated/virtual field |
| `R` | Called routine — the field's value is derived by a routine |
| `J` | Field join — joined from a related record |
| `S` | Enquiry prompt — an enquiry-only selection field |
| `V` | Virtual field — system-maintained, no manual input |

Two STANDARD.SELECTION features every developer uses:

- **`SYS.VAL`** — the validation routine called to *modify user input before comparison* in enquiries. T24 ships the `IN2` family: *"A standard 'T24' IN2 routine can be specified here (to convert customer mnemonics etc.) by specifying `IN2xxxx` where xxxx is a valid IN2 suffix, i.e. CUS, AMT"* (verified). `IN2CUS` resolves a mnemonic to a customer id; `IN2AMT` normalises amount entry; etc.
- The SS record is also where **local subroutines can be used as selection items** in the enquiry system (verified — T24 Application Program Interfaces guide).

**Field attributes** you will meet constantly in versions and templates: `NOINPUT` (field displayed but not inputtable), `CONCAT` (field value concatenated from other fields), `MULTI` (multi-valued), `DEFAULT` (default value), `INPUT` (the field-level input routine), `NOCHG`, `AUTONEW`, `DISPLAY.NAME`… — **flag:** the exact attribute names/positions differ across releases; the semantics are stable.

### 3.6 Local Reference Fields (LOCAL.REF)

Banks need their own fields on standard applications without touching Temenos core code. The mechanism is **local reference fields**:

- The bank defines the field once in the **`LOCAL.TABLE`** application (name, type, validation, help text — and optionally a **vetting table** restricting values; the local-reference mechanism can do "simple fields, or a more complex decision-based type" — verified from the Local Reference & Alternate Keys training material).
- The field is then **attached to applications** by updating the application's `LOCAL.REF` field (a special multi-value field present on every T24 application record). Local fields are referenced **by position** in `LOCAL.REF` (a numeric code, max 4 digits) — e.g. `R.NEW(XX.LOCAL.REF)<1,1>` reads the *first* local field attached to the app.
- Each application's fields subroutine (`.FIELDS`, §2.4) exposes the local fields to versions and enquiries; a **`LOCAL.REF` field** on a version (e.g. `FT.LOCAL.REF`) lets the version screen include them.

This is *the* standard way to add a custom field (the worked example in §7 uses it).

### 3.7 Versions

A **version** is a customised screen over a core application — a "V" application. A version record (in the `VERSION` application) declares:

- the **parent application** (e.g. `FUNDS.TRANSFER`) and the version name → `V.FT.NEW` is "the NEW version of FT" (versions are named `V.<APP>.<NAME>`);
- which fields appear, in what order, which are **`NOINPUT`** (hidden from input), which have **`DEFAULT`** values;
- **authorisation rules** — `NO.OF.AUTH` (how many authorisers), authorisation levels (`AUTH.LEVEL`/`FIELD.NO`), which fields trigger re-authorisation;
- **routines** — field-level input/validation routines, version-level validation routines (`VALIDATION.FLD` / `VALIDATION.RTN` — fields 58/59 of the VERSION application: *"once field X is filled, trigger routine Y"* — verified, wikibanking + T24 extensibility material), and record routines (`ROUTINE` field — flag: exact field name varies);
- the **`FUNCTION`** of the version (I=Input, A=Authorise, R=Reverse, D=Delete — a version can be built purely for authorising, reversing, or deleting records).

The version lifecycle — unauthorised record → authorisation → live record — is the heart of T24's control framework; see `core_banking_processes_guide.md` for the process view and `temenos_data_model_guide.md` §2 for the `$NAU`/`$HIS` record-status mechanics. Most bank "screen customisation" work is version work: create `V.FT.CUSTOM`, hide irrelevant fields, default the rest, add validation.

### 3.8 Enquiries

An **enquiry** ("E" application) is a query/report definition. The `ENQUIRY` application record declares:

- **`ENQUIRY.SELECT`** — the selection criteria: which file(s) to search, which fields are selectable, and the selection routines (incl. `SYS.VAL`/`IN2` conversions);
- **`ENQUIRY.DISPLAY`** — the display fields and their order (any field in the target application's STANDARD.SELECTION, plus I-type/virtual fields);
- output options (drilldowns, multi-value expansion, totals).

Two flavours:

- **File enquiries** — based on a real file via its STANDARD.SELECTION.
- **NoFile enquiries** (from R09 onwards — verified) — driven entirely by a routine (the `ENQ.` prefix convention, e.g. `ENQ.CUSTOM.REPORT`); the routine builds the result set and the enquiry record defines only the display. This is how most complex bank reports are exposed in the browser.

Enquiries are the standard answer to "we need a new screen/report" before anyone writes a batch program.

### 3.9 Reports

**Reports** ("R" applications) are batch-generated outputs: the report builder defines selection, layout, and totals; a **COB report job** runs at end-of-day (or on demand) and produces spooled output (printer files, CSV/Excel via `EXTRACT`, PDF). Report generation is typically: select records → format via the report template or a custom routine → output via the jBASE spooler / `EB.REPORT` utilities. Cross-ref `core_banking_processes_guide.md` for the COB context; for the BI layer (Temenos Insight/Data Hub) see §4.5.

### 3.10 Processes

**Processes** ("P" applications) are the batch programs themselves — the COB/EOD jobs (`BATCH` and `BATCH.DETAIL` records drive the scheduler: job name, service, routine, frequency, dependencies). A "process" can be a Temenos-supplied job (e.g. the interest-run processes of the AA engine — `interest_engines_core_banking_guide.md`) or a bank-written `PROGRAM`-type routine registered as a `BATCH.DETAIL` record so it runs under COB with the standard job-control, logging, and restart semantics. See `core_banking_processes_guide.md` for the EOD sequence.

---

## 4. The T24 Development Framework

### 4.1 TDF: The Temenos Development Framework

The umbrella name for the design-time tooling is the **Temenos Development Framework** (TDF) — the model-driven layer that lets developers *design* applications (fields, templates, versions, enquiries) and *generate* the executable T24 artefacts (files, input programs, routines) from the design. **Flag:** Temenos does not publish a single "TDF" reference manual; the concept is reconstructed from the training decks (template programming, application development), the Design Studio user guides, and the `developer.temenos.com` development guides. The stable, verifiable core is:

- **Design Studio** — the IDE (§4.2);
- **the cloud development workflow** — Design Studio + local T24 libraries + TAFJ + a Temenos **sandbox** + Git, per the "Develop T24 Packages, Plugins and Code" guide on `documentation.temenos.cloud` (verified — that guide names exactly these components and the import/export of applications, enquiries and versions);
- **code generation** — the framework generating the boilerplate (I_ input programs, file definitions, `.FIELDS` subroutines) from template definitions (§4.3).

### 4.2 T24 Design Studio

**Design Studio** is the Eclipse-based IDE for T24 development (verified — the "TAFJ Guide for Eclipse Users"/"Design Studio" user guides, and the cloud developer guide which lists "Design Studio (locally)" as a prerequisite). It provides:

- jBC editing with syntax awareness, and the TAFJ run/debug tooling (`tRun`, `tShow`, `tDebug` — see `tafj_guide.md` §4);
- the lock view and source management for the `BP`/library files;
- integration with the local T24/TAFJ installation for compile/catalog;
- the "plugin" model — modern Temenos cloud development packages applications and plugins into deployable units (the "T24 packages" workflow on `documentation.temenos.cloud`).

**Flag:** older releases used the jBASE-native tooling (jED + command line) almost exclusively; Design Studio became the standard IDE in the TAFJ/R14+ era. Both are still in the field.

### 4.3 Code Generation and the Model Bank

- **Code generation.** The template-driven framework *generates* the executable scaffold from the design: define the template/fields, and T24 generates the input program (`I_<APP>`), the file, the `STANDARD.SELECTION`, and the `.FIELDS` subroutine skeleton. The developer then fills in the validation routines. This generation step is why T24 applications can be stood up in days rather than months.
- **The Model Bank.** The **Model Bank** is Temenos' pre-built reference bank — a complete, configured Transact instance (companies, chart of accounts, products, parameters, sample data, reference configuration) shipped with the product and used for demos, training, *and as the starting point for implementations and upgrades* (see `temenos_guide.md`; also §6.4). "Model Bank testing" = validating a change against the pristine reference bank before applying it to the bank's own configuration. **Flag:** the Model Bank's exact contents vary by release; the concept is stable and widely referenced.

### 4.4 IRIS: The Interface Layer (Correction)

**IRIS** is frequently mis-described in the wild as a *reporting* tool. **Verified correction:** IRIS is Temenos' **integration platform / interface layer** — *"IRIS serves as Temenos' integration platform, facilitating access to T24 services such as inquiries and version calls through multiple protocols including HTTP, SOAP, and REST. When implemented within WebSphere or WebLogic, IRIS functions as a middleware layer that manages the routing of requests to the T24 core system"* (performetriks / T24 reference-architecture material). Alongside it:

- **TWS (Temenos Web Services)** — exposes T24 as SOAP web services;
- **IRIS** — the HTTP/interface server (REST and SOAP-style access to versions and enquiries — the modern "T24 API" surface);
- **Integration Framework adapters** — expose logic to/from an ESB (XML-based) (verified — Base Camp thread on interface components).

So: **IRIS = API gateway / interface server; reporting = enquiries + COB reports + Temenos Insight/Data Hub** (BI). Getting this wrong is a classic interview tell.

### 4.5 Insight (BI/Analytics)

**Temenos Insight** is the BI/analytics solution for Temenos data (verified — e.g. Soneri Bank's 2018 go-live *"has also implemented Temenos' analytics solution, Insight BI"*). Insight consumes Transact extracts (via Temenos Data Hub / reporting databases) for dashboards and analytics — it is *not* part of the T24 development toolchain but sits on its output. See `temenos_guide.md` §9 (Analytics) and §7 (Data Hub).

### 4.6 Extensibility: The Java API

The **Transact Extensibility** framework is Temenos' Java customisation surface (TAFJ-era; see `tafj_guide.md` §4): instead of (or alongside) jBC, developers write **Java hook routines** that plug into the T24 processing flow. Verified facts from the "T24 Extensibility in Java" training material:

- Extensibility routine types include **version routines** (input/validation routines performing validation and processing at commit), **check-record routines** (default values or alter field attributes after record ID entry), and similar hooks;
- the Java API surface is the **`T` types** in `TAFJClient.jar`: `TStructure`, `TField`, `TValidationResponse`, `TString`, `TNumber`, `TBoolean`, `TDate` — `TField` offers getters/setters for enrichment, errors, and field values;
- jBC invokes them via **`CALLJ`** (§2.6); Java can also call back into jBC.

This is the "modern" T24 developer skill — see `tafj_guide.md` §8 for the roadmap discussion.

### 4.7 The Daily Tools

The working toolkit of a T24 developer:

- **T24 Browser** — the online client (the browser-based front end) where versions are exercised and enquiries run;
- **DE (Data Entry)** — the classic T24 online input screens (the term survives from the terminal era; "DE" is still used for the online input environment, e.g. in version-testing instructions);
- **jED** — the jBASE full-screen editor for jBC source (`../technology/jbase_universe_guide.md`);
- **The jBASE command line** — `BASIC`, `CATALOG`, `jcompile`, `jShow`, `jDebug`/`jbase_dbug` (the jBASE debugger), file tools (`jdiag`, `jrf`, `jQL`/SQL);
- **TAFJ tooling** — `tRun`/`tShow`/`tDebug` under Design Studio, Maven packaging (`tafj_guide.md` §4);
- **Temenos Sandbox** — the cloud dev/test environment used with the package-based workflow (verified — the "Develop T24 Packages" guide).

---

## 5. The T24 Programming Patterns

### 5.1 The Routine Landscape

T24 code follows a set of named patterns. The names are load-bearing — job ads, forum answers, and Temenos decks all use them:

| Pattern | Where it hooks in | Typical prefix / convention |
|---|---|---|
| Input routines (field-level) | Run when a field is entered/exited in a version | `I_`-family, `<FIELD>.INPUT` — flag: exact hook names vary by release |
| Validation routines (record-level) | Run when the record is committed / validated | `VALIDATION.RTN` on a version; `E`/`ETEXT` error protocol |
| Pre / post routines | Run before/after the main processing (input, authorisation, COB) | `<APP>.<ACTION>.PRE/POST` — flag |
| Check-record routines | Run after record-ID entry, before the rest of the screen | Java extensibility hook (§4.6) |
| Subroutines (shared) | Callable utilities | `EB.*`, `ENQ.*`, `I_*`, `F_*` standard library |
| Enquiry routines | Drive NoFile enquiries / selection processing | `ENQ.*` |
| Report routines | Format/select for batch reports | `R.*`/report programs |
| OFS routines | Handle OFS messages / map external formats | `OFS.*`, `I_`-style handlers — flag |
| Online routines | Interactive (online) processing hooks | — flag: "online routine" is a loose term for any routine in the online path |
| Java hook routines | TAFJ extensibility | Java classes, `CALLJ` from jBC |

### 5.2 The Core jBC Patterns

**Field-level input routines.** Attached per field (in the version or the application template), they fire as the user enters the field: they validate the input, derive values into other fields, or default the field. The error protocol is universal: set the common variable **`E`** (error flag) and **`ETEXT`** (message text) and return — the framework rejects the input and displays the message. Deriving values means writing into `R.NEW` (the record being built).

**Record-level validation.** The `VALIDATE` pattern: a routine attached via the version's `VALIDATION.FLD`/`VALIDATION.RTN` (or the application's own validation fields) that runs once the record is complete and can check cross-field consistency and reject the whole record.

**Pre/post routines.** Wrappers around the standard processing steps — e.g. a routine that runs *before* authorisation to enrich the record, or *after* input to trigger a downstream action. **Flag:** T24's exact pre/post hook fields differ across releases (some are version fields, some are application template fields); the pattern itself is everywhere in T24 code.

**Version routines.** Any routine attached at the version level rather than the application level — the per-version customisation pattern: the same application can have ten versions with ten different validation routines. This is deliberate: banks give different channels (branch vs call centre vs back office) different versions of the same transaction, each with its own rules.

**Subroutines (shared/common routines).** The T24 standard library is a large set of `EB.*` (system/utility) and `ENQ.*` (enquiry) subroutines — e.g. `EB.CLEAR.FILE` ("clear a file or delete selected records from the specified file" — verified from the T24 Subroutine Guide). New shared logic should be written as `SUBROUTINE`s (not `PROGRAM`s) so every consumer can `CALL` it.

**Enquiry routines.** `ENQ.`-prefixed routines power **NoFile enquiries** (§3.8): the routine performs the selection/build of the result list; the `ENQUIRY` record defines display. Selection routines are also hookable per field (the `SYS.VAL`/`IN2` conversions of §3.5).

**Report routines.** For complex batch reports, a custom `PROGRAM`/`SUBROUTINE` does the selection and formatting, feeding the standard report-output utilities (spooler, `EXTRACT` for CSV/Excel) — registered as a process (§3.10) to run in COB.

**OFS routines.** Routines in the OFS path — handling OFS messages for an application, converting external formats into the application's fields, or post-processing after an OFS input. See `posting_engine_core_banking_guide.md` §7 for the OFS mechanics.

### 5.3 Callbacks: The T24 Hook Conventions

"Callback" in T24 means: **a routine the framework calls back at a defined point in the processing flow.** Conventions to know:

- **jBC callbacks** — the input/validation/pre/post/check-record hooks above; they are *named in configuration* (a version field, a template field, a `BATCH.DETAIL` record) and the framework invokes them by name.
- **`CALLJ` / Java callbacks (TAFJ)** — Java hook routines implementing the extensibility interfaces, invoked from jBC via `CALLJ` with `T`-type arguments (§4.6). The "T24 Extensibility in Java" material calls these *"Java hook routines"* and documents version-routine and check-record-routine variants.
- **"J" routines — flag:** there is no verified public convention of a "J routine" as a distinct T24 artefact; the plausible reading is the *jBC routine that bridges to Java* (the `CALLJ`-enabled pattern). Treat "J routine" as practitioner shorthand, not a documented type.

### 5.4 Integration Patterns

**OFS — Open Financial Service(s).** The message-based transaction protocol that is T24's primary integration surface (deep-dive: `posting_engine_core_banking_guide.md` §7.1). An OFS message addresses an application with comma-delimited field/value pairs:

```text
FUNDS.TRANSFER,DEBIT.ACCT.NO=1002345,DEBIT.AMOUNT=1000,DEBIT.VALUE.DATE=20260812,CREDIT.ACCT.NO=1006789,CREDIT.AMOUNT=1000,PAYMENT.DETAILS=TEST,...
```

- **`OFS.INPUT`** — the entry point: external systems (or the bank's own code) submit OFS messages; T24 runs the standard input/validation/authorisation pipeline and returns an OFS response (with `CURR.NO` and the resulting record ID).
- OFS supports transactions, versions (`VERSION` can be named in the message), enquiries, and routine invocation — *"OFS is the standard module and message format used for integration"* (verified — Temenos OFS training material).
- Practically: every interface from a channel, a payments hub, or a middleware queue terminates in an OFS call (see `payments_hub_guide.md`, `iso_20022_core_processes_guide.md`).

**Transact APIs (REST/SOAP).** The modern API surface sits *in front of* OFS:

- **IRIS** — the HTTP/interface server exposing versions and enquiries via REST/SOAP (§4.4, verified);
- **TWS** — SOAP web services (older but still deployed);
- **Transact API Framework** — the current security/governance layer for the API surface (verified — the "Transact API Framework Security" learning path in Temenos' TLC Online brochure);
- **Extensibility APIs** — the Java `T`-type API for custom logic (§4.6).

**Events.** Event-driven integration is a growing surface: **Transact Data Event Streaming (DES)** — verified as a named capability in the same TLC Online brochure — streams Transact data events out of the core for downstream consumers. **Flag:** the detailed event-framework mechanics (event catalog, subscription configuration) are partner-documented; treat specifics as release-dependent. Conceptually: external systems can now be *pushed* data changes rather than polling OFS/enquiries.

**The Integration Framework (IF).** The **Temenos Integration Framework** is the adapter-based layer that exposes T24 logic to an ESB — *"Integration Framework Adapters... expose logic to be consumed/used to/from ESB (based on XML)"* (verified — Base Camp). **Flag:** Temenos also sells a broader "Temenos Integration Framework" product family; in a T24 context the phrase usually means the in-product adapter framework. XML in, OFS/API out — the adapter translates between the bank's integration standard and T24's native protocol.

**Data extraction.** For reporting/analytics the pattern is *extract, don't integrate*: jBASE SQL/JQL over the MultiValue files (TAFC) or the JDBC relational view (TAFJ), the `EXTRACT`/CSV utilities, Temenos Data Hub extracts, and the SQL views described in `temenos_data_model_guide.md` §8. Reading data is cheap; writing is governed by the input/authorisation pipeline.

### 5.5 Pattern Selection: When to Use What

A decision aid used by T24 architects:

| Need | First choice | Then |
|---|---|---|
| New screen over existing app | Version (hide/default/reorder fields) | Add validation routine if rules are complex |
| New query/report screen | Enquiry (file or NoFile) | Custom `ENQ.` routine if logic is heavy |
| Custom field on a standard app | Local reference field (`LOCAL.TABLE` + `LOCAL.REF`) | Vetting table or input routine |
| Cross-field rule | Version validation routine | Application-level validation if all versions need it |
| External system integration | OFS message | IRIS/TWS/API if a service contract is needed |
| Batch output | COB report (R application) | Custom report program + `BATCH.DETAIL` |
| Complex shared logic | `SUBROUTINE` + catalog | Java hook if TAFJ/Java skills in-house |
| Real-time push of changes | DES events | API/notification layer |

---

## 6. The Development Lifecycle

### 6.1 Environments: The Landscape

A standard Temenos implementation runs four-plus environments:

| Environment | Purpose | Notes |
|---|---|---|
| **DEV** | Developer sandboxes; per-developer or shared | Modern: Temenos cloud sandboxes + local Design Studio/TAFJ (verified workflow, §4.1) |
| **SIT** | System integration testing — OFS/API/interface testing | Often multiple (SIT1/2) for parallel streams |
| **UAT** | User acceptance — business users, version/enquiry walkthroughs | Configuration changes mostly |
| **PROD** | Production (+ DR) | Changes arrive only via controlled promotion |

Environment *data* is as important as code: T24 environments carry Model Bank data, bank-parameter data, and test data; the "copy up" (DEV→SIT→UAT) of configuration records is a discipline of its own (flag: tooling varies — manual OFS/`EXTRACT`-based moves, or the package-based deployment in newer releases).

### 6.2 Promotion and Versioning

**Code promotion.** jBC routines and configuration records move environment-to-environment as *packages*: the modern workflow (verified — "Develop T24 Packages" on `documentation.temenos.cloud`) packages applications/enquiries/versions/routines, imports/exports them, and deploys to the target environment (with Git as the source control layer; older shops used jBASE `FILE`/`SAVE`-style moves or `EXTRACT`-load).

**Record versioning — `CURR.NO`.** Every T24 record carries a **`CURR.NO`** ("current number") — *"Curr No contains the current number of overrides held on the record"* (verified — T24 API user guide). Mechanics (see `temenos_data_model_guide.md` §2):

- A new record is `CURR.NO = 0` in the unauthorised file (`$NAU`);
- each authorisation/change bumps `CURR.NO` and pushes the previous version to the history file (`$HIS`) as `<ID>;1`, `<ID>;2`, … — the audit trail;
- **never** `WRITE` a record with a hand-maintained `CURR.NO` — practitioner warning (verified — performetriks): *"updating the audit fields with the WRITE command and incrementing the curr.no field... will result in no audit field trace for those current numbers. This should be avoided."* Use OFS/input programs so the framework maintains the audit fields.

### 6.3 Delivery: Releases and Upgrades

**The release train.** T24 releases were numbered **R10, R11, R12** (R12 was the Windows/SQL Server-optimised release — verified), through R13–R17; the product rebranded to **Transact** in 2018 and releases became **R18…R24** (annual), with the latest naming and cadence covered in `temenos_guide.md` §4. Each release ships the application code, the Model Bank, and the upgrade tooling.

**Upgrades.** Upgrading T24 = applying a *delta*: the difference between the bank's customised system and the new release. The discipline:

1. **Delta analysis** — what Temenos changed vs what the bank changed (the upgrade tooling compares against the Model Bank baseline);
2. **Regression testing** — re-verify all customised versions/enquiries/routines against the new release (see §6.4);
3. **Parallel run** — new release alongside old, then cutover (see `tafj_guide.md` §7 for an engine-swap variant; `temenos_guide.md` §18 for cost/timeline reality).

Upgrade skill is one of the most valuable T24 market skills precisely because every bank upgrades roughly annually and the delta work is bespoke.

### 6.4 Testing

- **Regression testing** — the core discipline: the bank's customisations must be re-verified on every release. The **Model Bank** (§4.3) is the reference: test the change against the pristine Model Bank first, then against the bank's configuration — isolating whether a failure is the bank's customisation or the release. **Flag:** "Model Bank testing" as a named formal methodology is practitioner usage; the concept is real (Model Bank ships with the product), the formal process name is not independently verifiable.
- **Test tools.** **Flag:** Temenos' current test-automation product names shift (historically "Temenos Test Automation"/"T24 test harness", now integrated with the cloud/CI tooling); verified stable facts: testing is predominantly *OFS-driven* (automated OFS message replay is the universal T24 regression technique) plus UI walkthroughs in the T24 Browser; the cloud developer workflow (Git + sandbox + packages) enables CI-style build/test pipelines. Exact product names: verify against the current partner docs.
- **Stress/performance testing** — practitioner-flagged: T24 (especially TAFJ) has distinctive characteristics (GC pauses, record-lock contention) that make load testing non-trivial (`tafj_guide.md` §6).

### 6.5 Development Governance

- **Change control** — standard ITIL-ish gating: code review, environment promotion approvals, release windows. T24-specific: *authorisation discipline* — the system's own two-person rule (input + authorise) applies to *configuration data* (versions, parameters) exactly as to transactions: configuration records live in the `$NAU`/live lifecycle too.
- **The "technical account".** **Flag:** the phrase "T24 technical account" appears in practitioner material with two readings: (a) the dedicated T24 *user/operator id* used for system maintenance and batch operations (the technical user), or (b) the Temenos **Technical Account** = the bank's licence/entitlement view of who may access Knowledge Base/downloads. Both are plausible; neither is cleanly verifiable in public Temenos docs. In practice: banks maintain a small set of privileged technical operator ids (`INPUTT`/`ADMIN`-class) for COB and maintenance — flag.

---

## 7. Worked Example: A Simple T24 Customisation

### 7.1 The Scenario

The bank needs every outgoing **FUNDS.TRANSFER** (FT) to carry a **payment purpose code** chosen from an approved list, and the code must be validated before the payment can be authorised. Standard Temenos FT has no such field. Plan:

1. Define the purpose codes as a **local reference field** (no core-code change);
2. Create a **version** `V.FT.CUSTOM` exposing the new field on a simplified screen;
3. Write a **jBC validation routine** enforcing the approved list;
4. Test via **OFS** (external-system path) and **online** (browser path).

### 7.2 Step 1 — Define the Local Field

In **`LOCAL.TABLE`**, define the local field (name, type, help text, and a **vetting table** — the approved purpose codes — so even the screen-level help validates): `LOCAL.PURPOSE`, with a vetting table record holding the allowed codes (`TRADE`, `SALARY`, `SUPPLIER`, `TAX`…). See §3.6 for the mechanics.

### 7.3 Step 2 — Create the Version

Create the version record **`V.FT.CUSTOM`** in the `VERSION` application:

| Version field | Value | Effect |
|---|---|---|
| `APPLICATION` | `FUNDS.TRANSFER` | Parent app |
| `VERSION.FUNCTION` | `I` | Input version (others: A/R/D) |
| `DESCRIPTION` | `FT with purpose code` | — |
| `NOINPUT` fields | `DEBIT.THEIR.REF`, `CREDIT.THEIR.REF`, … | Hide the fields the branch doesn't use |
| `DEFAULT` fields | `DEBIT.CURRENCY=EUR`, `CREDIT.CURRENCY=EUR` | Default values |
| `FT.LOCAL.REF` | include `LOCAL.PURPOSE` | Expose the local field on the screen |
| `NO.OF.AUTH` | `1` | One authoriser (or 2 for high-value) |
| `VALIDATION.FLD` | `FT.LOCAL.REF` | Trigger validation when the local-ref field is populated |
| `VALIDATION.RTN` | `FT.CUSTOM.VALIDATE` | The routine below |

No code has been written yet — this is the configuration-first layer.

### 7.4 Step 3 — The Validation Routine

The jBC routine (extending the §2.8 sketch):

```jbc
SUBROUTINE FT.CUSTOM.VALIDATE
* V.FT.CUSTOM validation: LOCAL.PURPOSE must be on the
* approved list in FT.CUSTOM.PARAM (a bank parameter app).
* Runs at record-validate time via VALIDATION.FLD/RTN.
    $INSERT I_COMMON
    $INSERT I_EQUATE
    $INSERT I_F.VERSION

    FN.FT.CUSTOM.PARAM = "F.FT.CUSTOM.PARAM"
    F.FT.CUSTOM.PARAM = ""
    CALL OPF(FN.FT.CUSTOM.PARAM, F.FT.CUSTOM.PARAM)

* The LOCAL.REF field is multi-valued; position 1 = LOCAL.PURPOSE
    PURPOSE.CODE = R.NEW(FT.LOCAL.REF)<1,1>
    IF PURPOSE.CODE EQ "" THEN
       E = "FT.PURPOSE.REQUIRED"
       ETEXT = "Payment purpose code is mandatory on V.FT.CUSTOM"
       RETURN
    END

    READ R.PARAM FROM F.FT.CUSTOM.PARAM, "PURPOSE.LIST" THEN
       LOCATE PURPOSE.CODE IN R.PARAM<1> SETTING POS ELSE
          E = "FT.PURPOSE.INVALID"
          ETEXT = "Purpose code ":PURPOSE.CODE:" not on approved list"
       END
    END ELSE
       E = "FT.PARAM.MISSING"
       ETEXT = "FT.CUSTOM.PARAM PURPOSE.LIST record missing"
    END
RETURN
END
```

Compile and catalog: `BASIC T24.BP FT.CUSTOM.VALIDATE` then `CATALOG T24.BP FT.CUSTOM.VALIDATE` (§2.5) — or use Design Studio's build on TAFJ.

### 7.5 Step 4 — Test

**OFS path** (external-system integration test — this is how a channel would call it):

```text
OFS.INPUT
FT.CUSTOM,V.FT.CUSTOM/I/PROCESS
DEBIT.ACCT.NO=1002345,DEBIT.AMOUNT=1000,CREDIT.ACCT.NO=1006789,
CREDIT.AMOUNT=1000,FT.LOCAL.REF=NOTALLOWED,...
```

Expected: the OFS response returns the error (`FT.PURPOSE.INVALID`) and no record is written. Re-submit with `FT.LOCAL.REF=TRADE` → the payment goes in as unauthorised (`CURR.NO=0`, `$NAU`), then authorise via `V.FT.CUSTOM/A` → live with `CURR.NO=1` and the `$HIS` audit copy.

**Online path** — open `V.FT.CUSTOM` in the T24 Browser/DE, enter the fields, and confirm the purpose field validates per keystroke (field-exit input routine behaviour) and at commit.

### 7.6 The Day in the Life

A typical T24 developer's workflow for this change:

```
design → version → routine → test → promote
  │         │         │        │        └─ package to SIT → UAT → PROD (§6.2)
  │         │         │        └─ OFS test + online walkthrough (§7.5)
  │         │         └─ write/compile/catalog FT.CUSTOM.VALIDATE (§7.4)
  │         └─ build V.FT.CUSTOM in the VERSION application (§7.3)
  └─ LOCAL.TABLE local field + vetting table (§7.2)
```

Reality check: the version + local field take minutes; the routine an hour; the *testing* (all the FT field combinations, the authorisation path, the OFS error handling, regression against standard FT) takes days. T24 development is dominated by testing and release governance, not by coding — same as any core-banking platform.

---

## 8. The Future: 2026 and Beyond

### 8.1 Where T24 Development Is Heading

| Trend | What it means for developers | Status (2026) |
|---|---|---|
| **Cloud / SaaS** | Transact on **Temenos Banking Cloud**; the dev workflow is already cloud-sandbox + Git + packages (verified, §4.1) | Live — see `temenos_guide.md` §10 |
| **Low-code / no-code** | More business logic pushed into configuration: versions, enquiries, local fields, product parameters, decision rules | Accelerating; the "code" share of implementations shrinks |
| **Java extensibility** | TAFJ-era: Java hook routines, `CALLJ`, T-types, packaged Java plugins | Mainstream on TAFJ installs — see `tafj_guide.md` §8 |
| **API-first integration** | IRIS/Transact API framework, DES event streaming replace raw OFS for new interfaces | Live (verified capability names) |
| **AI-assisted development** | Code generation for jBC/Java boilerplate, natural-language enquiry/version building, automated regression from OFS scripts | Emerging — see the AI guides in `../technology/ai_llm/` |
| **Engine consolidation** | TAFC → TAFJ migration continues; jBC stays, runtime becomes Java | Directional — `tafj_guide.md` §3, §7 |

### 8.2 The Trends Summary

The jBC language is not going away — 4,000+ modules of it are the product — but the *centre of gravity* of T24 development is moving: configuration and Java around a shrinking jBC core, delivered through cloud tooling and consumed through APIs. The durable skills for a T24 developer in 2026+: jBC (still the substrate), the configuration suite (the 80%), TAFJ/Java (the growth area), OFS + IRIS/API integration (the interface reality), and upgrade/regression discipline (the recurring work).

---

## 9. Glossary

- **T24** — the Temenos core-banking application ("Temenos 24"; 24-hour processing); rebranded **Transact** in 2018 (see `temenos_guide.md`).
- **Transact** — the modern name of T24; the application layer, distinct from the engine.
- **jBC** — jBASE BASIC / "T24 BASIC": the PICK-family language the T24 application is written in (`../technology/jbase_universe_guide.md`).
- **jBASE BASIC** — the jBASE dialect of PICK BASIC; jBC's formal family name.
- **BASIC** — the language family; also the jBASE compile verb (`BASIC sourcefile routine`).
- **Catalog** — the jBASE verb/process that makes a compiled routine callable by name (builds the runtime library; `JBCOBJECTLIST`/`JBCDEV_LIB`).
- **COB** — (1) **Close Of Business** — the end-of-day batch (the dominant T24 meaning, `core_banking_processes_guide.md`); (2) flag: unverified lore that legacy T24 used "COB" for the compiled-object file. In T24, default to meaning (1).
- **Compile** — the `BASIC`/`jcompile` step producing object code from jBC source.
- **PROGRAM** — jBC program type: standalone executable (batch jobs, utilities).
- **SUBROUTINE** — jBC program type: callable routine (`CALL name(args)`); the dominant T24 code unit.
- **FUNCTION** — jBC program type: returns a value.
- **CALL** — the jBC statement invoking a subroutine.
- **CALLJ** — the jBC→Java bridge on TAFJ, invoking Java hook routines with T-type objects.
- **Dynamic array** — the jBC variable model: one string with @FM/@VM/@SM levels; the basis of the T24 record model.
- **@FM / @VM / @SM** — field/value/sub-value marks (chars 254/253/252), the MultiValue delimiters (`temenos_data_model_guide.md`).
- **OPEN / READ / WRITE / READU** — jBC file verbs; `READU` = read with record lock (write or `RELEASE` unlocks).
- **C application** — core/table application (e.g. `FUNDS.TRANSFER`) — the entity layer (flag: letter classification per training material).
- **V version** — the version application type; a customised screen over a core app (records in `VERSION`).
- **E enquiry** — the enquiry application type (records in `ENQUIRY`).
- **I interface** — the interface application type (message/format definitions).
- **R report** — the report application type (batch report definitions).
- **P process** — the process application type (COB/batch job definitions).
- **S service** — the service application type (SOA/web-service definitions; TAFJ/IRIS-era).
- **TEMPLATE** — the application template / table definition from which T24 generates files, input programs and validation.
- **F. prefix** — the physical file prefix: `F.CUSTOMER`; multi-company `FBNK.CUSTOMER` (`temenos_data_model_guide.md`).
- **Standard selection (SS)** — `STANDARD.SELECTION`: the application's field catalogue / data dictionary (field types D/C/I/R/J/S/V, `SYS.VAL`, `IN2` routines).
- **Field** — a named attribute of a T24 record; positional (field number = attribute).
- **Attribute** — the `@FM`-delimited position in a record (the MV term for "field").
- **NOINPUT** — field attribute: displayed but not inputtable.
- **CONCAT** — field attribute: value concatenated from other fields.
- **MULTI** — field attribute: multi-valued (`@VM` list).
- **LOCAL.REF** — the special multi-value field on every application that attaches bank-defined local fields (by position).
- **Local field** — a bank-defined field attached via `LOCAL.TABLE` + `LOCAL.REF` without touching core code.
- **Version** — customised input/authorise/reverse/delete screen over an application (`V.<APP>.<NAME>`).
- **Enquiry** — query/report definition (`ENQUIRY` records; `ENQUIRY.SELECT`/`ENQUIRY.DISPLAY`; NoFile enquiries via `ENQ.` routines).
- **Report** — batch output definition (R applications; spooler/`EXTRACT` output; COB report jobs).
- **Process** — batch job definition (P applications; `BATCH`/`BATCH.DETAIL` records).
- **TDF** — Temenos Development Framework: the design-time, model-driven layer (design → generate) — flag: no single public manual.
- **Design Studio** — the Eclipse-based T24 IDE (TAFJ-era standard; jBC editing, tRun/tShow/tDebug, package builds).
- **Model Bank** — Temenos' pre-built reference bank shipped with the product; baseline for implementations, demos and upgrades.
- **IRIS** — Temenos' integration platform / HTTP interface server (REST/SOAP access to versions and enquiries) — *not* a reporting tool (verified correction).
- **Insight** — Temenos BI/analytics product consuming Transact data (via Data Hub) — not part of the dev toolchain.
- **Extensibility** — the Transact Java customisation framework (Java hook routines, T-types in `TAFJClient.jar`, invoked via `CALLJ`).
- **Integration Framework (IF)** — adapter layer exposing T24 logic to ESBs (XML); flag: product naming overlaps with a broader Temenos product family.
- **OFS** — Open Financial Service(s): the message-based transaction/integration protocol (`OFS.INPUT`); the universal integration surface (`posting_engine_core_banking_guide.md`).
- **REST API** — the modern HTTP API surface to T24 (IRIS/Transact API framework).
- **Event** — data/event streaming out of Transact (DES — Data Event Streaming) for push integration; flag: detailed mechanics partner-documented.
- **Promotion** — moving code/config between environments (DEV → SIT → UAT → PROD) as packages.
- **Deployment** — installing/releasing packages to an environment (package-based workflow; Git + sandbox on the cloud path).
- **CURR.NO** — the record's current number: bump on each change; prior versions go to `$HIS` (`<ID>;n`); the audit-trail mechanism — never hand-maintain it.
- **DEV / SIT / UAT / PROD** — the environment chain: development sandboxes, system-integration test, user-acceptance test, production (+DR).

---

## 10. References and Verification Trail

**Primary-adjacent public sources used (none is Temenos-official full documentation; all cross-checked):**

1. "T24 Application Development" / "T24 Application Development R17" (Scribd/PDFCOFFEE) — application anatomy, template-driven development, `.FIELDS` subroutine convention, application types.
2. "Application Structure and Files" (R17 deck) — PGM.FILE/FILE.CONTROL/STANDARD.SELECTION, live/`$NAU`/`$HIS` files.
3. "Template Programming" (T24 User Guide) — template concepts, the `V`/`Z` common variables, application vs subroutine program types.
4. "T24 Technical Kick-Start Training" — routine types, commands, enquiries (file + NoFile), sample programs.
5. "T24 STANDARD.SELECTION Data Dictionary Overview" — field types (C/D/I/R/J/S/V), `SYS.VAL`, `IN2` routines.
6. "T24 Application Program Interfaces: User Guide" — `CURR.NO`, OFS/API surface, standard selection subroutines in enquiries.
7. "T24 Extensibility in Java" (T24-111-4) — Java hook routines, version/check-record routines, T-types, `TAFJClient.jar`.
8. "3. Local Reference and Alternate Keys" — LOCAL.TABLE/LOCAL.REF mechanics, vetting, 4-digit position codes.
9. wikibanking.info + Base Camp — VERSION `VALIDATION.FLD`/`VALIDATION.RTN` (fields 58/59).
10. documentation.temenos.cloud "Develop T24 Packages, Plugins and Code" — Design Studio + TAFJ + sandbox + Git + Tortoise workflow.
11. performetriks.com — IRIS as integration platform (HTTP/SOAP/REST middleware); CURR.NO audit-discipline warning.
12. Temenos TLC Online Pathways brochure (Dec 2024) — "Transact API Framework Security", "Transact Data Event Streaming (DES)".
13. FinTech Futures (Apr 2018) — Soneri Bank: T24 + "Insight BI".
14. t24subroutines.blogspot.com — the EB.* subroutine guide (e.g. `EB.CLEAR.FILE`).
15. jBASE documentation (vladimirk2.github.io, Base Camp catalog thread) — `BASIC`/`CATALOG`, `JBCOBJECTLIST`, `JBCDEV_LIB`, `$`-object files, TAFC directory layout.

**Sibling guides cross-referenced:** `temenos_guide.md`, `temenos_data_model_guide.md`, `tafj_guide.md`, `../technology/jbase_universe_guide.md`, `posting_engine_core_banking_guide.md`, `interest_engines_core_banking_guide.md`, `core_banking_processes_guide.md`, `core_banking_systems_guide.md`, `green_link_digital_bank_guide.md`, `oracle_banking_microservices_architecture_guide.md`.

**Flagged items summary:** the formal C/V/E/I/R/P/S application-type table (training material, verify per release); "COB = compiled object file" (unverified lore — COB means Close of Business); TDF as a single named framework (reconstructed from tooling decks); template-field-level attribute names; "J routine" as a documented type; the "online routine" term; OFS routine naming conventions; Model Bank testing as a formal methodology; Temenos test-automation product names; the "T24 technical account" (two plausible readings); event-framework mechanics; Integration Framework product naming.

---
*End of guide. T24 programming is a 90% discipline/10% syntax game: the language is small, the framework is the product, and the release train is the rhythm.*
