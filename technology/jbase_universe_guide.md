# jBASE and UniVerse: The MultiValue Database Family — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Database / Data Engineering — MultiValue (PICK) data model, jBASE, UniVerse, UniData, Rocket Software U2 portfolio, Temenos Transact engine (Data Engineering series)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [The MultiValue Data Model](#1-the-multivalue-data-model)
2. [The PICK Heritage and the MultiValue Industry](#2-the-pick-heritage-and-the-multivalue-industry)
3. [jBASE Deep-Dive](#3-jbase-deep-dive)
4. [UniVerse Deep-Dive](#4-universe-deep-dive)
5. [UniData: The Third Rocket MV Platform](#5-unidata-the-third-rocket-mv-platform)
6. [MultiValue Development and Ecosystem](#6-multivalue-development-and-ecosystem)
7. [MultiValue in the Modern Landscape](#7-multivalue-in-the-modern-landscape)
8. [The Banking Context: MultiValue in Financial Services](#8-the-banking-context-multivalue-in-financial-services)
9. [Worked Example: FBNK.ACCOUNT on jBASE and UniVerse](#9-worked-example-fbnkaccount-on-jbase-and-universe)
10. [The Future Outlook](#10-the-future-outlook)
11. [Glossary](#11-glossary)
12. [References](#12-references)

---

## 1. The MultiValue Data Model

### 1.1 What MultiValue Is

**MultiValue (MV)** is a database paradigm — the direct descendant of the PICK operating system — in which a "record" is a single variable-length string of delimiter-separated fields, and repeating data lives *inside* the record rather than in child tables. The three defining properties:

- **Records are dynamic arrays.** A record is one ASCII string; fields are separated by attribute marks, with two further levels of in-field repetition (multi-values and sub-values). There are no fixed column widths and no empty-cell overhead — a field occupies only as many bytes as its value needs.
- **The schema is advisory, not enforced.** Files can be read and written with no definition at all; a separate *dictionary* optionally describes fields, formats, and computed values. This is **schema-on-read**, applied decades before the term existed.
- **Access is by record key first.** Every record has a unique key (`@ID`); hashed-file storage makes single-record lookup O(1)-ish — one physical read for an entire entity that a relational design would scatter across several normalized tables.

The MV model is the **original NoSQL**: flexible schema, denormalized aggregates, no joins — all before "NoSQL" was a term. The comparison to modern document stores is developed in §7.2 and in [nosql_data_modelling_guide.md §1](nosql_data_modelling_guide.md) (the query-driven philosophy is shared; the storage format and query language are not).

### 1.2 The PICK Heritage: From GIRLS to the Pick Operating System

MultiValue's core ideas were born in the 1960s:

- **1965 — GIRLS.** Don Nelson, working at TRW on a US Navy logistics project, designed the **General Information Retrieval Language System** — a file-based retrieval system whose data structures (attribute-delimited variable-length records, a simple English-like query language) are the direct ancestors of everything in this guide. GIRLS was public-domain and deliberately simple: one key per record, no fixed schema.
- **1973 — the Pick operating system.** **Dick Pick** (who had worked on the GIRLS project at TRW) turned the concept into a complete operating system + database: the **Pick OS**, licensed to Microdata and shipped as the **Reality** machine in 1975. Pick OS was a self-contained multi-user environment — OS, DBMS, BASIC language, and the **ENGLISH** query language in one box. It was famously easy to develop on: business users could build applications in days.
- **1984 — Pick Systems.** Dick Pick founded Pick Systems to sell the OS itself; **Advanced Pick** and later **D3** descend from this line.
- **1980s — the clone boom.** Because Pick was ported rather than copied, an ecosystem of licensed implementations grew: **Prime INFORMATION**, **Ultimate**, **General Automation**, and **McDonnell Douglas Reality** (a Unix re-implementation). Each ran Pick applications with near-identical data files.
- **1990s — from OS to database.** Pick-as-OS died (it could not compete with Unix/Windows on hardware and networking), but the *data model* survived by being re-implemented as an application layer on mainstream operating systems: **UniVerse (1984), UniData (1985), jBASE (1991), D3, Revelation, OpenQM (1993)**. That shift — from "Pick OS" to "MultiValue database on Unix/Windows" — is the pivot that created the modern industry (§2).

### 1.3 Records as Dynamic Arrays: The Encoding

An MV record is a string. Three delimiter characters create the hierarchy:

| Delimiter | Name | Hex | Displayed as | Role |
|---|---|---|---|---|
| Attribute mark | AM / field mark | 0xFE | `^` | Separates fields (attributes) within a record |
| Value mark | VM | 0xFD | `]` | Separates repeated values within one attribute (multi-values, MV) |
| Sub-value mark | SVM | 0xFC | `]]` | Separates repeated sub-values within one value (sub-values, SV) |

Example — a CUSTOMER record (keys and field numbers are conventional; the `^` notation is standard PICK display syntax):

```
@ID: 100001
1:  JONES^JOHN SMITH^15 MARINA BOULEVARD]18 RAFFLES PLACE]1 TEMASEK AVENUE^SINGAPORE]SINGAPORE]SINGAPORE^^S888888^+65 6123 4567^...
```

Here attribute 1 holds the name fields, attribute 2 (say) holds **three addresses as multi-values**, each with **street, city, postcode as sub-values** — one attribute, nine logical values, in a single string. A relational model would need three tables and two foreign keys to express the same entity; the MV record holds it in one read.

**The two levels of repetition are what the name "MultiValue" means.** MV fields (repeating values) replace child tables for one-to-many data; SV fields add a second dimension (the columns of each repeated row) without adding a table. This is the model's signature trade: massive read efficiency for the whole entity, at the cost of awkward relational-style reporting (see §1.6).

### 1.4 The Vocabulary

| Term | Meaning |
|---|---|
| **File** | The MV equivalent of a table — a named collection of records. A file = a data file + a dictionary file (see §1.5). |
| **Record** | The MV equivalent of a row — one dynamic-array string under one key. |
| **Attribute / field** | One delimiter-separated element of a record, addressed by number (attribute 1, 2, …). The physical position *is* the identity — there is no column name stored with the data. |
| **Multi-value (MV)** | A repeated value within an attribute (value marks). |
| **Sub-value (SV)** | A repeated value within a multi-value (sub-value marks). |
| **@ID** | The record key — the only mandatory, indexed element. Every MV file access begins with a key lookup. |
| **Dictionary / DICT / `]D`** | The optional metadata file accompanying every data file (§1.5). |
| **Account** | A login environment (a directory on the host OS) containing a VOC (vocabulary) file and the user's files — the MV analogue of a schema/namespace. |
| **VOC** | The account's vocabulary file: command names, file pointers, aliases, verbs — the "command table" of the MV environment. |
| **Hashed file** | The default MV storage: records distributed across groups by a hash of the key for O(1) lookup (§3.5). |

### 1.5 Dictionaries: Schema-on-Read

Every MV data file may have a companion **dictionary** (`DICT` in UniVerse, `]D` in jBASE/T24). Dictionary entries are themselves records that *describe* the data file:

- **D-descriptors (data definitions):** bind an attribute number to a name, type, and display format (e.g. `CUSTOMER = A;1` — "attribute 1, alphanumeric", `WORKING.BALANCE = R;10` — "attribute 10, right-justified numeric"). The dictionary gives the file its *readable* schema; without it, a file is just numbered strings.
- **I-descriptors (computed fields):** define derived attributes evaluated at query time — a concatenation (`"MR. " & CUSTOMER`), a transformation, a look-up into another file. I-descriptors are the MV answer to both views and (partly) joins: they compute on read, so the stored record never carries the derived value.
- **S-descriptors (symbolic/synonyms)** map alternate names to the same attribute; **A-descriptors** in some dialects cover both data and computed cases.

**"No schema" is a half-truth.** MV files *can* be written with no dictionary at all (the schema is then implicit in the application code — classic schema-on-read), but production systems — T24 above all — run on rich dictionaries: the T24 application definition (`PGM.FILE`, `FILE.CONTROL`, `STANDARD.SELECTION`) is effectively the schema, enforced by the application layer rather than the storage engine. The honest formulation: **MV storage enforces nothing; the dictionary and the application enforce whatever the designers chose.** This is exactly the schema-on-read/schema-on-write spectrum analysed in [nosql_data_modelling_guide.md §9.1](nosql_data_modelling_guide.md).

### 1.6 MultiValue vs Relational

| Dimension | MultiValue | Relational (SQL) |
|---|---|---|
| Unit of storage | File of dynamic-array records | Table of fixed-shape rows |
| Schema | Dictionary-driven, advisory, schema-on-read | DDL-enforced, schema-on-write |
| Repeating data | MV/SV fields inside the record (denormalized by design) | Normalized child tables + FKs |
| Relationships / joins | Cross-reference files (indexes into other files, e.g. `CUSTOMER.ACCOUNT`) maintained by the application; I-descriptors for lookups | Declarative SQL JOINs |
| Query language | ENGLISH-like (jQL, UniQuery, Retrieve) — verb + file + criteria, no joins | SQL — set algebra, arbitrary joins |
| Transactions | Record-level locking; explicit BEGIN/COMMIT/ROLLBACK (TRANSTART/TRANSEND/TRANSABORT in jBASE); transaction journaling for recovery | ACID transactions, row/table locks, MVCC |
| Single-record access | One hashed read returns the whole entity | Multiple indexed reads + joins |
| Ad-hoc reporting | Weak — query language is limited; needs extracts/SQL bridge | Strong — the core competency |
| Concurrency | File/record-level; high contention on hot records in older engines | Fine-grained locking, better OLTP concurrency at scale |
| Scaling | Vertical (bigger box); horizontal scale-out is hard | Vertical + horizontal (sharding, cloud-native) |
| Development | Rapid: no DDL, no migrations, one language (BASIC) end-to-end | Slower schema discipline; rich tooling/ecosystem |
| Skills | Scarce, specialized (PICK/MV BASIC) | Mainstream, abundant |

### 1.7 Strengths and Weaknesses

**Strengths.** Fast single-record access (one read per entity); schema flexibility that makes change cheap — adding a field is a dictionary/application change, not a migration; rapid development in one integrated BASIC language; small footprint and low resource cost; data locality (an entity is physically contiguous); decades of battle-tested vertical applications (distribution, retail, banking).

**Weaknesses.** Reporting and ad-hoc SQL access are hard (mitigated by SQL bridges — jBASE SQL engine, UniVerse SQL, ODBC/JDBC drivers — but always second-class); no real joins — cross-file relationships must be pre-built and maintained; concurrency and hot-record contention under heavy OLTP; skills scarcity (MV BASIC developers are a shrinking pool); the "legacy" perception that depresses budgets and recruitment; and horizontal scaling that lags modern distributed systems.

## 2. The PICK Heritage and the MultiValue Industry

### 2.1 Timeline

| Year | Event |
|---|---|
| 1965 | Don Nelson develops **GIRLS** at TRW (US Navy logistics); the conceptual ancestor of MV. |
| 1973 | **Dick Pick** completes the **Pick operating system**; licensed to Microdata (shipped as **Reality**, 1975). |
| 1984 | **Pick Systems** founded; **UniVerse** development starts at VMark Software (Unix-based Pick implementation). |
| 1985 | **UniData** development starts at Unidata Corporation. |
| 1980s | Pick clones spread: **Prime INFORMATION, Ultimate, General Automation, McDonnell Douglas Reality**. |
| 1991 | **jBASE** released by James Anthony Consultants (JAC), UK. |
| 1993 | **OpenQM** development starts (Ladybridge Systems, Martin Phillips). |
| 1990s | Pick-as-OS declines; MV databases on Unix/Windows become the industry; **D3** created by Pick Systems (successor to Advanced Pick); **Revelation/OpenInsight** carve the PC niche. |
| 1997 | VMark and Unidata merge → **Ardent Software**. |
| 1999 | **Temenos** (originally a jBASE VAR) acquires 100% of jBASE Software. |
| 2000 | Informix acquires Ardent. |
| 2001 | **IBM** acquires Informix's database business → UniVerse/UniData become the IBM U2 family; Pick Systems renamed **Raining Data**. |
| 2002 | Mpower1 creates **jBASE International** as worldwide distributor (Temenos retains IP and R&D). |
| 2008 | Raining Data renamed **TigerLogic**. |
| 2009 | **Rocket Software** acquires the U2 portfolio (UniVerse, UniData, SB+, U2 Web DE, wIntegrate) from IBM. |
| 2013 | **Rocket** acquires the MultiValue business of TigerLogic: **D3, mvBase, mvEnterprise** (announced 18 Nov 2013, completed 2014). |
| 2015 | **Zumasys** acquires jBASE from Temenos (5 Jan); becomes OpenQM's worldwide distributor. |
| 2019 | **Zumasys** purchases OpenQM from Ladybridge Systems (1 Jan). |
| 2021 | **Rocket** acquires jBASE, AccuTerm, MVConnect, MVDashboard, and OpenQM from Zumasys (14 Oct). Rocket now owns the entire mainstream MV portfolio. |

### 2.2 The 1990s Shift: From Operating System to Database

The PICK OS era ended because the OS itself could not compete with Unix/Windows on hardware support, networking, and price. The model survived by a strategic retreat: re-implement Pick's data model as an application layer on top of commodity OSes. UniVerse was the pioneer (1984, VMark); UniData, jBASE, and OpenQM followed. Each kept the same record format (attribute/value/sub-value marks), the same BASIC dialect family, and the same ENGLISH-style query verbs — which is why MV skills and MV data are largely portable across the family to this day. This is the industry's defining fact: **one data model, many engines, six decades of continuity.**

### 2.3 The Consolidation: Ownership Chains (Verified)

The MV market consolidated into Rocket Software through three distinct chains. **Correction to a commonly repeated version:** Rocket did *not* acquire jBASE from Temenos in 2013 (Zumasys intervened), and the U2 products left IBM in 2009, not 2004.

| Product | Chain (verified) | Key dates |
|---|---|---|
| **jBASE** | James Anthony Consultants (UK) → Temenos → Mpower1/jBASE International (distribution) → Zumasys → Rocket | 1991 (product) · 1999 (Temenos) · 2002 (Mpower1) · 2015 (Zumasys) · 2021 (Rocket) |
| **UniVerse** | VMark Software → Ardent → Informix → IBM U2 → Rocket | 1984 (start) · 1997 (Ardent) · 2000 (Informix) · 2001 (IBM) · 2009 (Rocket) |
| **UniData** | Unidata Corporation → Ardent → Informix → IBM U2 → Rocket | 1985 (start) · same chain as UniVerse from 1997 |
| **D3 / mvBase / mvEnterprise** | Pick Systems → Raining Data → TigerLogic → Rocket | 1990s (D3) · 2001 (Raining Data) · 2008 (TigerLogic) · 2013–14 (Rocket) |
| **OpenQM** | Ladybridge Systems → Zumasys → Rocket | 1993 (start) · 2019 (Zumasys) · 2021 (Rocket) |

Sources: jBASE International's own corporate-history page (via the Internet Archive); Rocket Software and Zumasys acquisition announcements (2015, 2021); Wikipedia "Rocket U2"; TigerLogic acquisition announcement. The 1999 Temenos acquisition date is from jBASE's own history page; some secondary sources cite 2007 — the primary source is used here.

### 2.4 The Current Landscape: Rocket's MultiValue Portfolio

Rocket Software's MultiValue portfolio (2026) spans five database engines plus a tooling layer:

| Product | Lineage | Current version (GA) | Notes |
|---|---|---|---|
| **jBASE** | PICK-derived, OS-native | **6.2.2** (~Q4 2025) | The modern-MV flagship; engine under Temenos Transact (§3) |
| **UniVerse** | VMark/IBM U2 | **14.2.1** (Nov 2025) | The U2 workhorse (§4) |
| **UniData** | Unidata Corp/IBM U2 | **8.4.1** (~Q2 2026) | SQL-mapping flavour of U2 (§5) |
| **D3** | Pick Systems direct line | **10.4.1** (~Q1 2026) | The "pure PICK" engine; enhancements now 10.4-only |
| **mvBase / mvEnterprise** | Pick Systems | 3.3.2 (Apr 2021) | Small-footprint/entry tier; legacy support |
| **OpenQM** | Ladybridge Systems | 4.0.9.1 (Jan 2025) | Free community-licensed MV; popular for education/migration |

Tooling: **AccuTerm** (terminal emulation), **MVConnect** (REST/graphQL middleware), **MVDashboard** (dashboards), **MVIS** (MultiValue Integration Server, free REST/JSON integration, v2.3.0 Oct 2025), **U2 Web DE** (web dev environment), **SB+/SystemBuilder** (4GL), **wIntegrate** (terminal emulation), **MV BASIC for VS Code** (VS Code extension), **MVX Performance** (monitoring). Rocket's published product-lifecycle handbook (Nov 2025) commits support for the current MV lines to 2029–2033 — a deliberate signal that the portfolio is not being wind-downed (§10.1).

## 3. jBASE Deep-Dive

### 3.1 The Product

**jBASE** is Rocket Software's MultiValue database and application development platform — a PICK-model database that runs *natively on the host operating system* (no emulation layer, no Pick OS underneath). Its defining architectural bet is that the OS does the OS's job (files, processes, memory, networking) while jBASE provides the MV data layer: hashed files, dynamic arrays, jBASE BASIC (jBC), the jQL query language, and a transaction journal. It is simultaneously the flagship for modern MV development and the engine embedded in **Temenos Transact** — which makes jBASE the most consequential MV database in the world by banking footprint (§3.12, §8).

### 3.2 History (Verified)

- **~1989–91** — Founded in the UK as **James Anthony Consultants (JAC)** — the "j" in jBASE — releasing the jBASE product in 1991. Co-founder **Clive Ketteridge** (later "manager of jBASE" at Temenos). jBASE was designed from the start as a *database-independent* MV platform: applications written in jBC run against jBASE files or, via the **jEDI** (jBASE External Device Interface), against external stores.
- **1999** — **Temenos**, originally a jBASE VAR and early adopter (it had built T24 on UniVerse and was migrating to jBASE), acquires 100% of jBASE Software and funds the product's R&D. jBASE becomes the T24 engine (§3.12).
- **2002** — Mpower1 International takes worldwide distribution/support rights (non-banking) and creates **jBASE International** as a distributor subsidiary; Temenos keeps the IP and the core development team.
- **2015** — **Zumasys** acquires jBASE from Temenos (5 Jan). Temenos retains a perpetual licence to keep shipping jBASE inside T24/Transact.
- **2021** — **Rocket Software** acquires jBASE (with AccuTerm, MVConnect, MVDashboard, OpenQM) from Zumasys (14 Oct), uniting jBASE with the U2 portfolio under one owner for the first time.

**Licensing model (verified):** jBASE is proprietary commercial software — per-server/per-user licensing, not open source. Rocket maintains an official GitHub organisation (`github.com/RocketSoftware`) with jBASE-adjacent material — samples, a jBASE Docker-cluster template, the MVIS integration server, MV BASIC tooling — but **the jBASE engine source is not published**. Claims that jBASE became "source-available" in 2024 could not be verified and are not supported by Rocket's GitHub or docs; treat them as unconfirmed. A notable consequence of the 2015 deal: Temenos is both the largest jBASE *user* (via Transact) and not its vendor — jBASE upgrades inside Transact follow Temenos's certified version matrix (§3.12).

### 3.3 Current Versions and Platforms

From Rocket's MultiValue Product Lifecycle Handbook (Nov 2025) and release notes:

| Version | GA date | Support horizon | Notes |
|---|---|---|---|
| **6.2.2** | ~Q4 2025 | EOS 2029 / EOLS 2031 | Current GA |
| 6.2.1 | Oct 2024 | EOS Oct 2028 | Security-focused release line |
| 6.1.1 / 6.1.2 | Feb/Apr 2024 | EOS Feb/Apr 2028 | "300+ mitigated risks", new security features |
| 5.9.0 | May 2023 | EOS May 2027 | AIX 7.2/7.3 certification; the version many T24 sites still run |
| 5.8.x | 2021–22 | EOS 2026 | Legacy |

Platforms: **Linux** (RHEL/SUSE/Ubuntu-family), **Windows**, **AIX** (5.9+ certified on 7.2/7.3); the 6.x line continues that coverage. jBASE runs as ordinary OS processes (no separate database server daemon required — a "database-in-process" model that makes it easy to embed and containerise).

### 3.4 Architecture

```
┌─────────────────────────────────────────────────────────┐
│ Application layer: jBC (compiled BASIC), jQL, C/C++/Java│
│ .NET/Python/REST clients via jAgent, jREST, ODBC/JDBC   │
├─────────────────────────────────────────────────────────┤
│ jBASE runtime: dynamic arrays, file drivers, dictionary │
│ services, transaction manager, jEDI plug-in layer       │
├─────────────────────────────────────────────────────────┤
│ Storage drivers (jEDI): hashed files (native) │ external│
│ (Oracle, SQL Server, PostgreSQL, MySQL via jEDI)        │
├─────────────────────────────────────────────────────────┤
│ Host OS: Linux / Windows / AIX — files, processes, MMU  │
└─────────────────────────────────────────────────────────┘
```

Key architectural facts:

- **OS-native.** jBASE is a library/runtime, not an OS or a server daemon. Records map to OS files; processes are OS processes; this is why jBASE feels "native" and why it was the industry's first MV platform that could store data in SQL Server or Oracle through jEDI.
- **jEDI** (jBASE External Device Interface) lets a jBASE application address external DBMSes as if they were MV files. This is the mechanism behind T24's historic "open database" option (Oracle, DB2, SQL Server, or jBASE) — see §3.12.
- **jBASE 6** modernised the runtime: 64-bit, enhanced security (encryption, auditing), container-friendly deployment, and continued jBC language growth (dynamic objects, OO-style classes).

### 3.5 Files, Dictionaries, and Accounts

- **Hashed files.** A jBASE data file is a directory of "groups"; a hash of the record key assigns each record to a group, giving direct (one-disk-read) lookup. jBASE supports dynamic resizing of files (`RESIZE`, `CREATE.FILE` with modulo/separator parameters).
- **Dictionary files.** Each data file has a dictionary (the `]D` file in jBASE/T24 naming): entries are D-descriptors (data), I-descriptors (computed), and synonyms. Query output formatting comes from the dictionary.
- **Accounts and VOC.** A jBASE account is a directory with an environment (`.env`/`jbase_env.sh`) and a **VOC** file listing commands, file pointers (a file pointer maps a logical name like `FBNK.ACCOUNT` to the physical hashed file), and user-defined verbs. `jbase_env.sh`/`.profile` setup and `JBCRELEASEDIR` (the jBASE install root) define the runtime environment.
- **File types.** Beyond hashed files, jBASE supports directories (for source code), and legacy/foreign formats for migration (e.g., reading UniVerse/UniData/D3 files directly — a real migration advantage).

### 3.6 jBC: The jBASE BASIC Compiler

**jBC** (jBASE BASIC) is the MV-BASIC dialect compiled by `basic` into C, then compiled by the host C compiler into native executables — which is why jBASE programs run at near-C speed and can link C libraries directly. Dialect essentials (see also [temenos_data_model_guide.md §2](../banking/temenos_data_model_guide.md) for the T24 variant):

- **Dynamic-array string handling:** `field = record<attr>` / `record<attr> = value`; MV/SV access via `record<attr, mv>` and `record<attr, mv, sv>`.
- **File verbs:** `OPEN`, `READ`, `WRITE`, `DELETE`, `READNEXT` (with `SELECT`/`READLIST` for keys).
- **Transaction verbs:** `TRANSTART`, `TRANSEND`, `TRANSABORT` (§3.8).
- **Modern extensions:** dynamic objects (JSON-like), `$`-prefixed system functions, threading, sockets, Java/.NET interop (`CALLJ`), and the **jBASE 6** class syntax. T24's dialect is historically called **InfoBasic** in some older Temenos documentation; the modern term is jBASE BASIC / T24 BASIC — same compiler family.

### 3.7 jQL: The Query Language

**jQL** (jBASE Query Language) is the ENGLISH-descendant query language: `VERB file {criteria} {output}`. Core verbs:

| Verb | Effect |
|---|---|
| `LIST` | List records matching criteria |
| `SORT` | List with a sort key (column of the sort field on the left) |
| `SELECT` / `SSELECT` | Put matching record keys into a select list (for BASIC loops) |
| `COUNT` | Count matching records |
| `SUM` | Column totals |
| `STAT` | File statistics |

Example: `SORT FBNK.ACCOUNT BY CATEGORY WITH WORKING.BALANCE GT 10000 CUSTOMER ACCOUNT.TITLE.1 WORKING.BALANCE`. jQL reads the dictionary for display formats, supports I-descriptors as computed columns, `BY-DSND` (descending), `BREAK-ON`, `TOTAL`, and `HEADING`. It has no joins — cross-file lookups go through dictionary descriptors or application code (see §9 for worked examples).

### 3.8 Transactions and the Transaction Journal (TJ)

jBASE transactions are explicit and file-scoped:

- **`TRANSTART`** begins a transaction; **`TRANSEND`** commits; **`TRANSABORT`** rolls back. Between START and END, writes are held so that other users see the pre-transaction state (locking is record-level, with `WRITEU`/`READU` for explicit record locks).
- **Transaction journaling (TJ)** records all updates to log files so the system can roll forward/back after a crash. Logsets (active/standby) are switched at backup/Close-of-Business points; `jlogadmin` administers the journal. This is the mechanism that gives T24 its recoverability story — a jBASE crash mid-Close-of-Business is rolled back to the last consistent checkpoint, then re-run.
- Concurrency reality: record-level locking works well for the single-record-at-a-time workload MV was built for; hot records (e.g., a heavily-updated GL summary record) become serialisation points — a known T24 performance topic.

### 3.9 Security

- **Login and accounts:** OS-level users map to jBASE accounts; `PASSWORD`/`LOGIN` commands manage MV-level passwords.
- **File permissions:** read/write/delete/execute rights per file and per user class; the VOC and dictionary can restrict what users see.
- **jBASE 6.1+:** hardened defaults, data-at-rest encryption support (the jEDI encryption layer descends from Mpower1's DREM module), audit hooks, and TLS for the jAgent/jREST services. (Historically, MV security was application-level — the T24 maker-checker/record-status regime in [temenos_data_model_guide.md §2.5](../banking/temenos_data_model_guide.md) is the banking-grade example.)

### 3.10 Tools

- **jShell** (command interpreter), **jEditor** (vi-style editor), `jsh`/`jsh -i` interactive sessions.
- **Admin utilities:** `CREATE.FILE`, `RESIZE`, `jlogadmin`, `jbackup`/`backup_jbase`, `jrestore`, `jshow`, `jlicense`, `jbase_env`, `jdiag`, `jkeyauto`.
- **jAgent / jREST:** the service layer exposing jBC routines and MV data over TCP/HTTP — the bridge for modern frontends (§6.6).
- **JDBC/SQL:** the jBASE SQL engine + `jbasejdbc.jar` expose MV files as read-only tables — the standard reporting path (§8.6, §9).
- **IDEs:** Rocket's **MV BASIC for VS Code** extension covers the MV BASIC family; jBASE work historically happens in jEditor/jShell or the T24 Workbench/Designer for Temenos projects.

### 3.11 Performance Characteristics

- Single-record access is the model's superpower: one hashed read for the whole entity; jBC compiles to native code.
- Indexes are optional and explicit (`CREATE.INDEX`); unindexed criteria scans read whole groups — the classic MV query performance trap, mitigated by select lists and well-chosen keys.
- The journal and record locking add write-path overhead; tune logset sizing and checkpoint frequency.
- Scale is vertical: jBASE performs extremely well on a big, well-tuned box (many T24 banks run hundreds of thousands of transactions/day on modest hardware); horizontal partitioning is manual (split files by key ranges/companies).
- In the T24 context, database performance tuning is dominated by file sizing (modulo/separator), journal configuration, and the EOD batch window — see the Temenos-specific operational guidance in [temenos_guide.md §12](../banking/temenos_guide.md).

### 3.12 jBASE and Temenos Transact (T24)

- **The engine relationship.** T24 was originally built on **UniVerse**; Temenos replaced UniVerse with jBASE after acquiring it ("Since replacing UniVerse and embedding jBASE in their T24 banking application…" — jBASE corporate history, 2011). Today **jBASE is the native database engine of Temenos Transact**, the world's most widely deployed core banking platform (1,000+ banks across 150+ countries — Euromoney 2025, per [temenos_guide.md §1](../banking/temenos_guide.md)).
- **File structure.** T24 applications live in files named `F<MNE>.<APPLICATION>{$<SUFFIX>}` — e.g. `FBNK.ACCOUNT`, `FBNK.ACCOUNT$NAU` (unauthorised), `FBNK.ACCOUNT$HIS` (history), `FBNK.ACCOUNT]D` (dictionary), with concat files like `FBNK.CUSTOMER.ACCOUNT` for customer→account lookups. Full application anatomy (PGM.FILE, FILE.CONTROL, STANDARD.SELECTION, I_, TEMPLATE) is covered in [temenos_data_model_guide.md §2.2](../banking/temenos_data_model_guide.md) — that guide is the T24 application-model companion to this database-level guide.
- **Version coupling.** Transact certifies specific jBASE releases; banks run the jBASE version shipped inside their Transact release (the "T24/jBASE licensing mechanism" is documented in Temenos Base Camp). Upgrading jBASE independently of Transact is not supported — a key operational constraint.
- **Not jBASE-only anymore.** jBASE's own history page states that T24's prerequisite was "jBASE open connectivity technology… coupled with an open database such as Oracle, IBM's DB2, Microsoft's SQL Server or jBASE". Modern Transact (R23+) runs on the **TAFJ** Java runtime and is certified with **SQL Server 2019, Oracle 19c, and PostgreSQL 15** alongside jBASE (R23 Stack-1 runbooks; PostgreSQL became a first-class option in R23+, per [temenos_guide.md §12.1](../banking/temenos_guide.md)). The MV data model persists in all of them — the relational backends are physical storage behind the same application/dictionary layer. **Verify the certified database matrix for the exact Transact release before planning** — it changes by version.

## 4. UniVerse Deep-Dive

### 4.1 The Product and the U2 Family

**UniVerse** is Rocket Software's MultiValue database and application development environment — the "other" flagship, and historically the most widely deployed MV database in the vertical-application market (distribution, logistics, retail, education, healthcare software). It is the senior product of the **Rocket U2** family: UniVerse + UniData, plus SB+/SystemBuilder (4GL), U2 Web DE, UniObjects (API), and wIntegrate (terminal emulation). UniVerse is a **derivative of the Pick operating system**, run as an application layer on Unix/Linux/Windows.

### 4.2 History (Verified)

- **1984** — VMark Software begins UniVerse as a Unix-based implementation of the Pick model; the product is marketed as an "extended relational" DBMS — MultiValue storage *plus* an SQL engine, a duality that remains its signature.
- **1997** — VMark merges with Unidata Corporation → **Ardent Software** (UniVerse and UniData stay separate products).
- **2000** — Informix acquires Ardent.
- **2001** — IBM acquires Informix's database business; UniVerse/UniData become the **IBM U2 family** (positioned under IBM's Information Management group).
- **2009 (1 Oct)** — **Rocket Software purchases the entire U2 portfolio from IBM**, establishing what became Rocket's MultiValue database business. (The 2004 acquisition date sometimes quoted is incorrect — the U2 purchase closed in 2009.)

### 4.3 Current Version and Platforms

From Rocket's Product Lifecycle Handbook (Nov 2025):

| Version | GA date | Support horizon | Notes |
|---|---|---|---|
| **14.2.1** | Nov 2025 | EOS Nov 2029 / EOLS Nov 2031 | Current GA |
| 14.1.1 | Jun 2024 | EOS Jun 2028 | 14.x = modern line |
| 12.2.1 | Oct 2022 | EOS Oct 2026 | Previous major line |
| 11.4.1 | Aug 2023 | EOS Aug 2027 | 11.x still supported |

Platforms: **Linux, Windows, and Unix** (AIX; historically HP-UX and Solaris). The 14.x line is actively developed — a direct rebuttal to the "abandonware" assumption some architects carry about U2.

### 4.4 Architecture

- **Accounts.** A UniVerse system is organised into accounts (directories on the host OS). Each account contains a **VOC** (vocabulary) file — the command table mapping every command, file name, keyword, alias, and script — plus the user's files. The account is the MV analogue of a database/schema namespace.
- **Files = data file + DICT.** Every UniVerse file has two parts: the data file (records) and the **dictionary** (`DICT`), which may contain metadata describing or formatting the data. Files are either **hashed** (records distributed by hash of the record ID for O(1) lookup) or **non-hashed** (directory-style storage for unstructured content like source code).
- **Records as ASCII.** UniVerse stores records (including the record ID) as pure ASCII strings — no binary blobs. Fields are separated by field marks (hex FE); values by value marks (FD); sub-values by sub-value marks (FC). The famous UniVerse storage property: a record occupies exactly its characters plus delimiters — variable-length with zero waste.
- **"Extended relational".** UniVerse's SQL engine coexists with the native model: tables are *views* over files (an MV field becomes a repeating column; dictionaries map to relational metadata). This was the industry's earliest serious MV↔SQL bridge and is the ancestor of today's U2 ODBC/JDBC and MVIS integration layers.

### 4.5 The UniVerse Data Model

Identical in shape to §1: files, records, `@ID` keys, attributes, MV, SV. UniVerse-specific nuances:

- The dictionary supports D/I/S descriptors plus **A-descriptors**; I-descriptors can perform transforms and cross-file lookups at query time.
- **Cross-reference files** (e.g. an invoice file with a customer cross-reference) implement the relationships that SQL would express as joins; UniVerse provides `CREATE.INDEX` for alternate keys and `XREF` maintenance patterns.
- Data typing is display-oriented (A = alphanumeric, R = right-justified numeric, etc.) — types describe formatting and collation, not storage.

### 4.6 UniQuery

**UniQuery** is UniVerse's ENGLISH-descendant query language:

| Verb | Effect |
|---|---|
| `LIST` / `LIST.ONLY` | Display records |
| `SORT` | Sort + display |
| `SELECT` / `SSELECT` | Build a select list of keys |
| `COUNT` | Count records |
| `SUM` / `TOTAL` | Aggregate a column |

Modifiers: `BY`/`BY-DSND` (sort), `WITH`/`WITHIN` (criteria), `BREAK-ON` (grouping), `HEADING`, `FOOTING`, `GRAND-TOTAL`, `ID-SUPP`, `DET-SUPP`. Sample: `SORT CUSTOMER BY CUSTOMER.NAME WITH BALANCE GT 1000 CUSTOMER.NAME BALANCE` — the same verb+file+criteria+output shape as jQL (§3.7) because both descend from the same 1960s ENGLISH design. UniQuery is read-oriented; writes go through BASIC or the UniVerse editor.

### 4.7 UniVerse BASIC

**UniVerse BASIC** (U2 BASIC, a.k.a. UniBasic) is the MV-BASIC dialect with `OPEN/READ/WRITE`, `MATREAD`, dynamic-array attribute syntax (`record<attr, mv, sv>`), `EXECUTE` for running UniQuery, and `CALL` for subroutines. It shares the family DNA with jBC but has its own compiler (`basic`), its own system variables (`@USERNAME`, `@DATE`), and dialect differences that matter when porting code (see §6.1). Program source is stored in non-hashed directories; compiled objects run as host executables.

### 4.8 Development Tools

- **SB+/SystemBuilder (SB/XA):** the U2 4GL for screen and menu-driven applications — a generation of vertical applications (order entry, inventory, accounting) was built on SB+ over UniVerse data.
- **U2 Web DE:** web development environment for building browser front-ends over U2 data.
- **UniObjects:** the object API for connecting Java/.NET/Python programs to UniVerse/UniData.
- **wIntegrate:** terminal emulation and application client (the modern replacement for the classic UniVerse character UI).
- **Rocket MV BASIC for VS Code:** syntax/compilation support for U2 BASIC in VS Code.

### 4.9 Integration with Modern Stacks

- **UniVerse SQL:** the embedded SQL engine exposes files as tables (read-only for most purposes) — `SELECT * FROM ACCOUNT WHERE ...` against MV data. This is the closest thing MV has to native SQL and is the basis of most UniVerse reporting.
- **ODBC/JDBC:** Rocket's **U2 ODBC/JDBC drivers** are the standard integration path to BI tools (Power BI, Tableau, Crystal Reports) and ETL (SSIS, Informatica).
- **REST/JSON:** **Rocket MVIS** (MultiValue Integration Server, free) exposes U2 (and other MV) data as REST/JSON endpoints — the modern "headless MV" bridge (§6.6). MVConnect (from the Zumasys line) provides similar REST/graphQL capabilities.
- **.NET:** U2.Data.Client for .NET; classic COM/ActiveX bridges for older integrations.

### 4.10 Security and Administration

- Accounts and passwords at the MV layer; OS-level file permissions underneath; file- and command-level access control through the VOC and file headers.
- Administration via `uv` (the UniVerse command shell), `sysadmin` account tasks, `CREATE.FILE`, `RESIZE`, `uvbackup`, and the U2 admin utilities; Rocket provides certified upgrade/support lifecycles (14.x through 2031+).

## 5. UniData: The Third Rocket MV Platform

### 5.1 What UniData Is

**UniData** is Rocket Software's other U2 MultiValue database — UniVerse's sibling, born from the separate **Unidata Corporation** line (1985) and unified with UniVerse under Ardent (1997), Informix (2000), IBM (2001), and Rocket (2009). It is a full MV platform — hashed files, dictionaries, BASIC, ENGLISH-style query — with one distinctive emphasis: **a strong SQL mapping layer and "meta-database" features** that make it the U2 product most comfortable in mixed relational environments. UniData is common in distribution, retail, and supply-chain vertical applications, and is the DB of choice for MV shops that also run Oracle/SQL Server alongside.

### 5.2 UniData vs UniVerse

| Dimension | UniVerse | UniData |
|---|---|---|
| Origin | VMark Software (1984), Unix-first | Unidata Corporation (1985) |
| SQL surface | Embedded SQL engine, "extended relational" positioning | SQL *mapping* layer + meta-database features; strong ODBC story |
| BASIC dialect | UniVerse BASIC (UniBasic) | UniData BASIC (UDBASIC) — distinct compiler, similar family |
| Query language | UniQuery (LIST/SORT/SELECT/…) | Retrieve + SQL (RETURN/EXECUTE SQL) |
| Storage | Hashed/non-hashed files, ASCII records | Hashed files; typed/schema-mapped files supported for SQL visibility |
| Dictionaries | DICT with D/I/S/A descriptors | Dictionaries + data-type mapping to relational metadata |
| Typical positioning | General-purpose MV app platform, biggest U2 install base | MV with the smoothest relational coexistence (SQL, ODBC, meta-data) |
| Current version | 14.2.1 (Nov 2025) | 8.4.1 (~Q2 2026) |

Both share the same record format, so MV data and much MV code move between them (and to jBASE/D3) with modest dialect work — a practical point for migrations (§7.4). UniData's "meta-database" capability — describing MV files to external SQL tools as if they were relational schemas — is the cleanest of the U2 pair for feeding BI/ETL.

### 5.3 UniData's Niche

UniData's niche is **MV applications that must look relational to the outside world**: ISVs embedding UniData behind their products while exposing SQL/ODBC to customer BI stacks; retailers integrating MV inventory/order systems with relational ERP; and migration projects that want to keep the MV application but report through standard SQL tooling. It is also the U2 product most often chosen when an organisation standardises on the Rocket U2 ODBC/JDBC drivers for enterprise reporting.

## 6. MultiValue Development and Ecosystem

### 6.1 The MV BASIC Dialects

All MV engines speak dialects of the same BASIC family — dynamic-array string handling (`record<attr, mv, sv>`), file verbs, and the same philosophical simplicity — but each dialect has its own compiler, syntax quirks, and system functions. Porting is real work, not a recompile:

| Dialect | Engine | Distinctive marks |
|---|---|---|
| **jBC / jBASE BASIC** | jBASE | Compiles via C to native code; dynamic objects; `CALLJ`; class syntax (6.x) |
| **T24 BASIC** | Transact on jBASE | jBC + TAFC/TAFJ runtime, `I_` validation layer, version/enquiry frameworks — see [temenos_data_model_guide.md §2](../banking/temenos_data_model_guide.md) |
| **UniVerse BASIC (UniBasic)** | UniVerse | `MATREAD`, `EXECUTE`, @-system variables, SB+ integration |
| **UniData BASIC** | UniData | SQL interop verbs, meta-database calls |
| **D3/mvBase BASIC** | D3 | The most "classic PICK" dialect; PROC paragraphs |
| **OpenQM BASIC** | OpenQM | Close to UniVerse BASIC; good documentation |

### 6.2 Query Languages: One Family, Three Flavours

jQL (jBASE), UniQuery (UniVerse), and Retrieve (UniData) are all descendants of PICK **ENGLISH** — the same verb-first grammar (`LIST file WITH criteria fields`), same dictionary-driven formatting, same absence of joins. Differences are cosmetic at the syntax level but real in features (jQL's `SELECT`-list integration with jBC is the tightest; UniQuery's `BREAK-ON`/`TOTAL` reporting is the richest). None of them replaces SQL for analytics — that role belongs to the SQL bridges (§6.5).

### 6.3 Application Frameworks and 4GLs

- **T24/Transact** — the largest MV application framework in existence: versions, enquiries, OFS, the AA (Arrangement Architecture) product model. Database-level view in this guide; application-level view in [temenos_data_model_guide.md §3](../banking/temenos_data_model_guide.md).
- **SB+/SystemBuilder (SB/XA)** — U2's 4GL for screens/menus/reports; still supported and maintained by Rocket.
- **OpenInsight (Revelation)** — the Windows-centric MV development environment (the Revelation line, now part of Rocket's wider portfolio through acquisitions).
- **OSMOSiS** — the Mpower1-era 4GL for jBASE (converts SB+ paragraphs into BASIC).
- **Modern hybrids** — jBC + jREST + web front-ends; MVConnect/MVIS for API-first builds; Rocket's "MV modernization" practice (jBASE + containers + cloud) — see §6.6 and §7.4.

### 6.4 IDEs and Development Environments

- **MV BASIC for VS Code (MVVS)** — Rocket's VS Code extension for the MV BASIC family (jBASE/UniVerse/UniData/D3) — the most important recent IDE development for the community.
- **jShell/jEditor, uv/EDIT** — the traditional editors; still how most production MV work gets done.
- **Temenos Workbench/Designer** — the Eclipse-based IDE for Transact development (TAFJ/TAFC) — see [temenos_guide.md §12](../banking/temenos_guide.md).
- **AccuTerm** (terminal emulation, Windows/Mobile) and **wIntegrate** remain the UI clients of record for character-mode MV apps.

### 6.5 Tools: Reporting, Migration, and Integration

- **Reporting:** U2 ODBC/JDBC drivers and jBASE JDBC feed Power BI/Tableau/Crystal Reports; Rocket **MVX Performance** monitors MV systems; classic **mv.ODBC/mv.SSIS/mv.RSDC** (Mpower1 line) plug MV into SQL Server BI/SSIS. For banking, extracts to the warehouse remain the dominant pattern (§8.6).
- **Migration:** Rocket professional services run the official MV→MV and MV→modern paths; the community **jbase-discovery** tool (GitHub) assesses jBASE estates for migration readiness; jBASE's ability to read UniVerse/UniData/D3 files directly makes cross-engine migration unusually tractable. MV→relational extraction (attributes→columns, MV→child tables) is the hard part — see §7.4.
- **Integration:** MVIS (REST/JSON, free), MVConnect (REST/graphQL), jAgent/jREST (jBASE), U2 ODBC/JDBC, UniObjects, U2.Data.Client (.NET), and classic SOAP/XML bridges. CDC-style change capture is not built in — it is achieved via transaction-journal parsing or extract-and-compare — a key constraint for event-driven architectures (§8.6).

### 6.6 "Headless MV": MV as Backend for Modern Frontends

The dominant modern pattern is to leave the MV application untouched and expose it: MVIS/MVConnect REST endpoints or jAgent services in front, a modern SPA/mobile app in front of that, and the MV database still the system of record. This is how most UniVerse ISVs and many T24 banks are modernising without rewriting: the MV core keeps running batch, integrity rules, and transactional logic; the new UI talks JSON. The data-integration and event-streaming context is in [event_stream_processing_guide.md §8](event_stream_processing_guide.md).

### 6.7 The Community and Skills

- **International Spectrum** — the MultiValue industry's magazine and conference (the annual MV conference and the MV "Database of Databases" visibility); the community's news organ for acquisitions and releases.
- **Rocket community** (community.rocketsoftware.com) — vendor-run forums for jBASE/D3/U2; **Zumasys** remains a major services/cloud partner; **Temenos Base Camp** — the T24 developer community.
- **Skills reality:** MV BASIC developers are scarce and ageing; demand is sustained by the installed base (T24 banks alone need thousands of T24 BASIC/TAFC developers — see [temenos_guide.md §20](../banking/temenos_guide.md) and [temenos_data_model_guide.md §11](../banking/temenos_data_model_guide.md)). Rates for senior MV engineers are high; the risk is concentration in a small, retiring pool — the key human-risk input to §8.5's assessment.

## 7. MultiValue in the Modern Landscape

### 7.1 MV in 2026: The Hidden Installed Base

MultiValue in 2026 is *invisible but everywhere*:

- **Temenos Transact** is the largest MV footprint on the planet: 1,000+ banks across 150+ countries (Euromoney 2025, per [temenos_guide.md §1](../banking/temenos_guide.md)) run on jBASE (or Transact's relational backends, which preserve the MV model — §3.12). That single application dwarfs every other MV estate combined.
- **The "hidden" MV:** thousands of vertical applications — distribution/wholesale (the classic PICK home), retail point-of-sale, education administration, healthcare records, logistics, government licensing — still run on UniVerse, UniData, D3, and jBASE behind the scenes. They are not in any "legacy database" survey because their names don't appear in the stack lists; the applications are known, the database is just "the system".
- **OpenQM** provides a free entry path (education, small ISVs), keeping the skill pool from collapsing entirely.

The honest frame: MV is legacy in the sense of *vintage* — but it is a legacy of working, debt-free systems that process real business every day, and its largest user (Temenos) is a growth company. "Legacy" ≠ "dying"; it means "long-lived and needing deliberate strategy" (§10.5).

### 7.2 MV vs NoSQL: The Original NoSQL

MV is the original NoSQL — schema-flexible, denormalized aggregates, key-first access — and the comparison to document stores is genuinely instructive:

| Dimension | MultiValue (jBASE/UniVerse) | Document stores (MongoDB et al.) |
|---|---|---|
| Storage unit | Dynamic-array record (delimiter string) | JSON/BSON document |
| Schema | Dictionary-driven, schema-on-read | Schema-on-read (`$jsonSchema` optional) |
| Repeating data | MV/SV inside the record | Embedded arrays/objects |
| Key | @ID (any string) | _id / partition key |
| Query | ENGLISH-style verbs (jQL/UniQuery), no joins | JSON query language, aggregation pipelines, indexes |
| Transactions | Explicit TRANSTART/END, record locks | Single-document atomicity, multi-doc transactions (recent) |
| Ecosystem | Proprietary, small community, BASIC-centric | Huge open ecosystem, drivers for everything |
| Scaling | Vertical | Horizontal by design |

Both models answer the same question — *"how do I store a whole entity in one retrievable unit?"* — with different answers to *"how do I then find things?"*. MongoDB's query engine and ecosystem win that second half decisively; MV's advantage is the existing application base, the deterministic performance of one-hashed-read, and the transaction journal. The deeper modelling comparison (query-driven design, denormalization, aggregate identity) is in [nosql_data_modelling_guide.md §1–2](nosql_data_modelling_guide.md); the schema-evolution discussion in §9 there maps almost one-to-one onto MV dictionary evolution.

### 7.3 MV vs Relational

Against PostgreSQL/MySQL, the decision is workload-shaped, not ideology-shaped:

- **Choose/keep MV when:** the application is record-centric (one entity, one read), the schema churns, the team is MV-skilled, or the application *is* the database (T24, vertical ISV products). MV's total cost of ownership on an existing estate is usually far lower than a rewrite.
- **Choose relational when:** ad-hoc analytics, joins, or multi-system integration dominate; the team is mainstream-SQL; horizontal scale-out is a hard requirement; or the data must participate in a modern data platform (warehouse, lakehouse, feature stores). The relational design canon is in [data_model_resource_book_guide.md §3](data_model_resource_book_guide.md).
- **The middle path** (increasingly the norm): keep MV as system of record, replicate/extract to relational for analytics — §7.4's "integrate" path.

### 7.4 Modernization Paths

| Path | What it is | When it fits | Effort |
|---|---|---|---|
| **1. Leave (run as-is)** | Maintain on current versions (Rocket supports MV lines to 2029–2033; §2.4) | Stable, small, no business driver for change | Low |
| **2. Lift-and-shift (cloud MV)** | Run jBASE/UniVerse on AWS/Azure VMs or containers; managed ops, backup, DR in the cloud | Cost/compliance drivers; no functional change needed | Low–medium |
| **3. Integrate (MV + modern stack)** | Expose via REST (MVIS/MVConnect/jREST), ODBC/JDBC to BI, ETL extracts, event feeds | Analytics, digital front-ends, APIs required | Medium |
| **4. Replace (migrate off MV)** | Extract MV data to relational/lakehouse; rewrite or retire the application | End-of-life apps, strategic platform consolidation | High |

**Cloud MV specifics:** jBASE runs natively as OS processes — trivially containerised (Rocket publishes Docker/cluster templates); UniVerse/UniData run on cloud Linux/Windows images; Zumasys and Rocket both operate MV cloud practices. The AWS/Azure lift keeps the application binary-compatible while buying managed infrastructure — the pragmatic first step for most MV estates.

**Migration challenges (path 4):** MV→relational extraction is where projects bleed:

- **Attributes → columns:** field numbers become column names via the dictionary — but the dictionary is often incomplete, so columns are defined from application code archaeology.
- **MV/SV → child tables:** repeating groups become child tables (or JSON columns — the pragmatic middle ground); choosing which changes the query model downstream.
- **@ID semantics:** composite/smart keys (e.g. T24 ids with check digits and `;n` history suffixes) need re-keying or surrogate keys.
- **Validation logic:** MV applications enforce rules in code (and T24 in `I_` layers); the relational schema must re-encode them as constraints.
- **History files:** T24's `$HIS`/`$NAU` companion files and record-status lifecycle (§2.5 of [temenos_data_model_guide.md](../banking/temenos_data_model_guide.md)) have no relational analogue — audit trails must be redesigned.

The Temenos-specific variant — moving a bank off Transact itself — is covered in [core_banking_systems_guide.md §8](../banking/core_banking_systems_guide.md) and [chinese_bank_core_systems_guide.md §5](../banking/chinese_bank_core_systems_guide.md) (the 去IOE/distributed-core context).

## 8. The Banking Context: MultiValue in Financial Services

### 8.1 The Temenos Banks

The banking story *is* the jBASE story at scale. Temenos Transact — jBASE-native, MV-modelled — runs the core of 1,000+ banks (retail, corporate, Islamic, microfinance; [temenos_guide.md §1](../banking/temenos_guide.md)). For a bank architect this means: **your core system of record is a MultiValue database**, whether the storage underneath is jBASE or a relational backend, because the application model (files, records, attributes, MV/SV, dictionaries, versions, enquiries) is MV all the way down. The T24 application-model details — `F<MNE>.<APP>` files, ACCOUNT/AA, OFS, STMT.ENTRY — are the subject of [temenos_data_model_guide.md §2–5](../banking/temenos_data_model_guide.md); this section is about what that means for banking IT strategy.

### 8.2 Reporting and Data Extraction

- **The core is not a reporting database.** T24 reporting runs through enquiries (query definitions), extracts, and the Data Hub layer — Temenos Transact Data Hub (which itself runs on relational platforms: Azure SQL, SQL Server, Postgres, Oracle; [temenos_guide.md §7](../banking/temenos_guide.md)).
- **The standard pattern:** batch extracts from jBASE (via jQL selects, jBC extract programs, or jBASE JDBC) into the warehouse/lakehouse, then governance-grade reporting. BCBS 239 risk-data lineage starts at the extract definition — see [data_governance_guide.md §10](data_governance_guide.md) and the worked BCBS 239 program in §12 there.
- **Read-only SQL access** via jBASE JDBC is the sanctioned reporting path; write access stays in the application (OFS/versions) — the read/write split is architectural, not accidental (§8.6).

### 8.3 Integration: OFS and APIs

- **OFS** (Open Financial Service) is Transact's message interface — `<APP>,<VERSION>,<FUNCTION>,<ID>/<FIELD>:<VALUE>` — the workhorse for real-time inbound (channels, payments) and outbound integration; full treatment in [temenos_data_model_guide.md §5](../banking/temenos_data_model_guide.md) and [temenos_guide.md §19](../banking/temenos_guide.md).
- **APIs:** modern Transact exposes 700+ Open APIs (REST/JSON) over the same application layer; the MV data model is invisible behind them.
- **Event-driven:** the core's transaction log (jBASE TJ) and extracts are the raw material for CDC-style feeds into streaming platforms; because MV lacks native CDC, banks build it from OFS outbound messages, journal parsing, or poll-and-diff — see [event_stream_processing_guide.md §9](event_stream_processing_guide.md) for the banking patterns.

### 8.4 MV Skills in Banking IT

Banking IT carries a parallel skills stack: T24 BASIC (jBC) development, TAFC (C-based) vs TAFJ (Java) runtime knowledge, version/enquiry configuration, and jBASE administration. These are well-paid but narrow, and the pool skews to markets with large Temenos footprints (EMEA, Middle East, Africa, Asia-Pacific). Succession planning is a real board-level topic at T24 banks — cross-ref [temenos_guide.md §20](../banking/temenos_guide.md) for the skills market and [temenos_data_model_guide.md §11](../banking/temenos_data_model_guide.md) for the T24 developer profile.

### 8.5 The Risk View

- **Skills concentration:** a retiring MV/T24 workforce in a market where new entrants learn SQL-first — the single biggest operational risk.
- **Legacy risk:** aging jBASE 5.x estates (still supported to 2027+) need version hygiene; unsupported versions are an audit finding.
- **Modernization drivers:** cloud strategy (run Transact on AWS/Azure/GCP or SaaS via Temenos Banking Cloud), real-time expectations (the MV batch/EOD model vs 24×7), data-platform convergence (the bank's lakehouse wants the core's data), and — in China — 信创/xinchuang domestic-stack mandates that exclude foreign MV engines entirely ([chinese_bank_core_systems_guide.md §7](../banking/chinese_bank_core_systems_guide.md)). Note that Chinese banks run domestic cores (or Oracle-based), not Temenos; the MV-skills concentration risk there is nil, which is precisely why xinchuang accelerates MV displacement in that market.

### 8.6 The Architect's View

- **MV as system of record:** accept the MV core as the authoritative ledger and design around its strengths (fast record access, strong batch, maker-checker audit) and its boundaries (no joins, weak analytics, record-level locks).
- **Integration patterns that work:**
  - *Transactional in:* OFS / Open APIs (write path stays in the application).
  - *Transactional out:* OFS outbound, API notifications.
  - *Analytics out:* scheduled extracts + jBASE JDBC read-only into the warehouse ([data_governance_guide.md §7](data_governance_guide.md) metadata/lineage, §10 BCBS 239).
  - *Event out:* journal/extract-fed streaming for real-time use cases ([event_stream_processing_guide.md §8](event_stream_processing_guide.md)).
  - *Feature data for ML:* extracts → feature store ([feature_store_guide.md §12](feature_store_guide.md) — banking features are dominated by core-derived balances and transaction history).
- **Assessment framework for a Transact bank:** (1) version hygiene audit (jBASE + Transact release, support status); (2) data-access inventory (what reads the core, through which bridge); (3) extract lineage map (feeds → warehouse → reports → regulators); (4) skills heatmap (who can still change the jBC); (5) cloud readiness (containerisation, DR); (6) then choose: leave / lift / integrate / replace (§7.4). The replace option — moving off Transact entirely — is a core-banking programme, not a database project ([core_banking_systems_guide.md §8](../banking/core_banking_systems_guide.md)).

## 9. Worked Example: FBNK.ACCOUNT on jBASE and UniVerse

### 9.1 The Record

T24 stores accounts in `FBNK.ACCOUNT` (Model Bank company mnemonic `BNK`). Field numbers below are the widely documented T24 ACCOUNT layout — **field numbers are release- and bank-dependent, so verify against the installed dictionary (`FBNK.ACCOUNT]D`) before relying on them**:

| Attr | Field name | Sample value | Notes |
|---|---|---|---|
| @ID | Account number | `1001234` | T24 account ids carry a check digit (ACCT.CHECKDIG.TYPE in COMPANY) |
| 1 | CUSTOMER | `100001` | Customer master key (join via CUSTOMER.ACCOUNT concat) |
| 2 | ACCOUNT.TITLE.1 | `JOHN SMITH` | |
| 3 | ACCOUNT.TITLE.2 | `SAVINGS` | |
| 5 | CURRENCY | `USD` | |
| 6 | CATEGORY | `1001` | The category *is* the GL account (chart of accounts) |
| 10 | WORKING.BALANCE | `12500.50` | The LIMITS-check balance |
| 11 | CLEARED.BALANCE | `12500.50` | |
| 12 | AVAILABLE.BALANCE | `12500.50` | Available-funds ladder |

A raw jBASE dump of such a record (field marks shown as `^`):

```
1001234^100001^JOHN SMITH^SAVINGS^^USD^1001^^^^^^^^^12500.50^12500.50^12500.50^...
```

And the MV/SV mechanics with a CUSTOMER record (attribute 2 = three addresses as MV, each with street^city^postcode as SV):

```
100001^JONES^JOHN SMITH^15 MARINA BOULEVARD]18 RAFFLES PLACE]1 TEMASEK AVENUE^SINGAPORE]SINGAPORE]SINGAPORE^018981]039594]039192^^S888888^+65 6123 4567^...
```

In jBC, attribute 2, value 2, sub-value 1 = `rec<2,2,1>` → `18 RAFFLES PLACE`. The same address data in a relational model needs a table, an FK, and a join; here it is one attribute of one record.

### 9.2 jQL Queries

```
LIST FBNK.ACCOUNT WITH CATEGORY EQ 1001 CUSTOMER ACCOUNT.TITLE.1 WORKING.BALANCE
SORT FBNK.ACCOUNT BY WORKING.BALANCE WITH CURRENCY EQ "USD" CUSTOMER ACCOUNT.TITLE.1 WORKING.BALANCE
COUNT FBNK.ACCOUNT WITH WORKING.BALANCE GT 10000
SELECT FBNK.ACCOUNT WITH CATEGORY EQ 1001
```

The `SELECT` puts matching keys into a select list consumable by jBC's `READNEXT` (§9.3) — the classic MV "query then process" pattern, since jQL itself cannot join or update.

### 9.3 jBC Snippet

```
PROGRAM LIST.ACCOUNTS
OPEN "FBNK.ACCOUNT" TO F.ACCOUNT ELSE ABORT 201, "FBNK.ACCOUNT"
EXECUTE "SELECT FBNK.ACCOUNT WITH CATEGORY EQ 1001" CAPTURING SEL
LOOP
  READNEXT ID ELSE EXIT
  READ R.ACCT FROM F.ACCOUNT, ID ELSE CONTINUE
  CUSTOMER     = R.ACCT<1>
  CURRENCY     = R.ACCT<5>
  CATEGORY     = R.ACCT<6>
  WORK.BAL     = R.ACCT<10>
  CRT ID, CUSTOMER, CURRENCY, CATEGORY, WORK.BAL
REPEAT
```

Compiled with `basic` (to C) then `jbc`/`c` to a native executable. This is the shape of a T24 extract routine — minus the OFS/audit ceremony — and is the pattern banks use to feed warehouses (§8.2).

### 9.4 SQL Access via jBASE JDBC

The jBASE SQL engine exposes files as read-only tables (dots become underscores):

```sql
SELECT id, customer, account_title_1, currency, category, working_balance
FROM   fbnk_account
WHERE  category = '1001' AND working_balance > 10000;
```

Accessed from Java via `jbasejdbc.jar` (`jdbc:jbase:...`); MV fields map to the first value unless a relational mapping is defined in the dictionary. This is the sanctioned read-only reporting path into T24 data — never the write path (§8.2, [temenos_data_model_guide.md §5](../banking/temenos_data_model_guide.md)).

### 9.5 The UniVerse Equivalent

Same model, sibling dialect — UniQuery:

```
LIST ACCOUNT WITH CATEGORY EQ "1001" CUSTOMER ACCOUNT.TITLE.1 WORKING.BALANCE
SORT ACCOUNT BY WORKING.BALANCE
COUNT ACCOUNT WITH WORKING.BALANCE GT 10000
SELECT ACCOUNT WITH CATEGORY EQ "1001"
```

UniVerse BASIC equivalent loop: `OPEN "ACCOUNT" TO F.ACCT`, `SELECT F.ACCT TO LIST`, `READNEXT`, `READ R FROM F.ACCT, ID` — same verbs, `MATREAD`/`EXECUTE` dialect differences (§6.1). In a UniVerse shop the reporting equivalent of §9.4 is UniVerse SQL or the U2 ODBC/JDBC driver into Power BI.

### 9.6 Lesson of the Exercise

The same business object — an account with a customer, a currency, a category, and balances — is *one record, one read* in MV, and *four tables plus joins* in relational form. That is the entire MV value proposition in miniature: superb for the transactional application that owns the record, weak for the analyst who wants to slice it — which is why every serious MV estate builds the extract bridge (§8.6) rather than fighting the model.

## 10. The Future Outlook

### 10.1 Rocket's MV Strategy (Verified Signals)

Rocket's actions say "investing, not harvesting": jBASE 6.2.2 and UniVerse 14.2.1 shipped within weeks of each other (late 2025); UniData 8.4.1 and D3 10.4.1 followed (2026); the Product Lifecycle Handbook (Nov 2025) commits current-line support to 2029–2033; MVIS is given away free to make REST integration universal; MV BASIC for VS Code modernises the developer experience; and Zumasys was anointed Rocket's "first MV Modernization Partner" (2021). The commercial logic is clear: MV is a cash-generating installed base with a captive upgrade path (25% legacy-version surcharges push version hygiene), and Rocket can monetise modernisation (cloud, integration, tooling) on top.

### 10.2 MV and AI/ML

- **MV data for ML** flows through the same extract bridges: core data → warehouse/lakehouse → feature store → models. For banks, the highest-value features (balances, velocity, history) are core-derived — see [feature_store_guide.md §12](feature_store_guide.md).
- **LLM-assisted MV development** is nascent but real: LLMs trained on MV BASIC can generate jBC/T24 routines, explain legacy code, and translate dialects (jBC ↔ UniBasic) — the practical use is *legacy code comprehension* (the biggest MV cost) before generation. **Flag: no verified production-grade MV-specific LLM tooling as of mid-2026** — treat copilot claims as vendor marketing until evaluated.
- **AI agents + MV operations** (auto-diagnosing T24 EOD failures, jBASE journal analysis) are emerging in the managed-services space — consistent with the broader trend in [data_governance_guide.md §13](data_governance_guide.md).

### 10.3 MV and the Cloud

Cloud MV adoption is real and accelerating along two lines: **lift** (jBASE/UniVerse on cloud VMs/containers — Rocket publishes container templates; Zumasys runs a MV cloud practice) and **SaaS** (Temenos Banking Cloud runs Transact — jBASE-backed — as managed SaaS for 700+ institutions, per [temenos_guide.md §10](../banking/temenos_guide.md)). The constraint is licensing and skills, not technology: MV runs fine in the cloud; the market is learning to buy it there.

### 10.4 The Temenos Trajectory and jBASE's Role

- Transact's database future is plural: **jBASE remains the native engine and the default for the installed base**; relational backends (SQL Server, Oracle, PostgreSQL from R23+) serve cloud-native and large-scale deployments while preserving the MV application model (§3.12). Temenos sells database choice as a cloud-agnosticism feature (the "distributed relational database" line from the Transact rebranding blog).
- For the architect this means: **jBASE skills stay relevant for the life of the Transact installed base — measured in decades, not years** — while new deployments increasingly ask about the relational option. The T24 data model (MV) survives on either substrate; only the physical layer changes.
- Temenos's own roadmap pushes cloud-native, API-first, AI-enabled Transact ([temenos_guide.md §12](../banking/temenos_guide.md)); jBASE's role within it is engine, not identity.

### 10.5 Decline vs Persistence: The Honest View

MultiValue is in a state of **managed persistence, not decline**:

- The T24 installed base alone (1,000+ banks) guarantees jBASE relevance for decades — core banking migrations take 3–10 years and banks replace cores once or twice a generation.
- The vertical-ISV MV base (UniVerse/UniData/D3) is slowly shrinking as ISVs modernise or retire, but the decay rate is measured in single-digit percentages per year — these are profitable, working systems.
- The skill pool is the real constraint: MV will persist longer than the people who currently maintain it unless banks invest in knowledge transfer and modern tooling (VS Code, REST, cloud) to make the work attractive.
- The honest verdict: **MV is legacy but huge; it will not grow, but it will not disappear on any roadmap horizon that matters.** The professional posture is the one this guide has taken throughout: understand the model, respect the installed base, and engineer the bridges — because the data will outlive the engines' fashionability.

### 10.6 Trends Summary

| Trend | Direction | Signal |
|---|---|---|
| Rocket investment | ↑ | 6.2.2 / 14.2.1 / 8.4.1 / D3 10.4; PLC to 2033 |
| Cloud MV | ↑ | Containers, Zumasys/Rocket cloud, Temenos SaaS |
| REST/API modernisation | ↑ | MVIS free, MVConnect, jREST |
| Transact database pluralism | ↑ | PostgreSQL/Oracle/SQL Server certified alongside jBASE |
| MV skills pool | ↓ | Ageing workforce; LLM-assisted comprehension emerging |
| MV as a % of new development | ↓ | Near zero — but installed base is the market |

## 11. Glossary

| Term | Definition |
|---|---|
| **MultiValue (MV)** | The PICK-descendant data paradigm: records as attribute-delimited dynamic arrays with repeating values and sub-values. |
| **MV (value)** | A repeated value within an attribute (value mark 0xFD). |
| **SV (sub-value)** | A repeated sub-value within a value (sub-value mark 0xFC). |
| **Attribute / field** | A delimiter-separated element of a record, addressed by number. |
| **Record** | One dynamic-array string stored under one key. |
| **File** | A named collection of records; = data file + dictionary. |
| **@ID** | The record key. |
| **Dictionary / DICT / `]D`** | The metadata file describing fields and formats. |
| **D-descriptor** | Dictionary entry binding an attribute number to a name/type/format. |
| **I-descriptor** | Dictionary entry computing a derived field at query time. |
| **Schema-on-read** | Schema applied at read time by dictionary/application; storage enforces nothing. |
| **PICK** | The 1970s operating system + database from which MV descends (Dick Pick; GIRLS heritage from 1965). |
| **GIRLS** | General Information Retrieval Language System — Don Nelson's 1965 TRW system; the MV ancestor. |
| **ENGLISH** | The original PICK query language; ancestor of jQL, UniQuery, Retrieve. |
| **Hashed file** | MV storage distributing records by hash of the key for O(1) lookup. |
| **VOC** | The account's vocabulary file (commands, file pointers, aliases). |
| **Account** | An MV login environment (directory + VOC). |
| **jBASE** | Rocket's OS-native MV database; engine under Temenos Transact. |
| **jBC** | jBASE BASIC — the jBASE programming language (compiled via C). |
| **jQL** | jBASE Query Language (LIST/SORT/SELECT/COUNT…). |
| **jEDI** | jBASE External Device Interface — storage abstraction to external DBMSes. |
| **TJ** | jBASE Transaction Journal — the recovery log. |
| **TRANSTART/TRANSEND/TRANSABORT** | jBASE transaction control verbs. |
| **UniVerse** | Rocket's U2 MV database (VMark/IBM lineage; current 14.x). |
| **UniQuery** | UniVerse's ENGLISH-style query language. |
| **UniData** | Rocket's U2 MV database with strong SQL mapping (current 8.x). |
| **U2** | Rocket's UniVerse + UniData product family. |
| **D3 / mvBase / mvEnterprise** | The Pick-Systems-lineage MV engines (via Raining Data/TigerLogic). |
| **OpenQM** | Free-licensed MV database (Ladybridge Systems; now Rocket). |
| **SB+ / SystemBuilder** | The U2 4GL application framework. |
| **MVIS** | Rocket MultiValue Integration Server — free REST/JSON bridge. |
| **MVConnect** | REST/graphQL middleware from the Zumasys line. |
| **AccuTerm** | MV terminal emulation client. |
| **T24** | Temenos's core banking platform (predecessor of Transact). |
| **Transact** | Temenos's core banking product (T24 evolution); jBASE-native. |
| **FBNK** | Model Bank company mnemonic; T24 files are `F<MNE>.<APP>`. |
| **OFS** | Open Financial Service — Transact's message integration interface. |
| **TAFC / TAFJ** | Transact's C-based / Java-based runtimes. |
| **ODBC / JDBC** | Standard DB access APIs; Rocket ships U2 and jBASE drivers (read-only SQL surface). |
| **CDC** | Change Data Capture — not native to MV; built from journals/extracts. |

## 12. References

**Sibling guides in this repo** (cross-referenced above; the Temenos data-model companion is [temenos_data_model_guide.md](../banking/temenos_data_model_guide.md), the T24 application-level view is [temenos_guide.md](../banking/temenos_guide.md)):

- [temenos_data_model_guide.md](../banking/temenos_data_model_guide.md) — T24/Transact application model, ACCOUNT/AA, OFS, jBASE hashed-file storage (§2), record lifecycle (§2.5), SQL access (§5)
- [temenos_guide.md](../banking/temenos_guide.md) — Temenos company/product context (§1), Transact architecture incl. the jBASE heritage (§12), integration (§19), skills (§20)
- [nosql_data_modelling_guide.md](nosql_data_modelling_guide.md) — the NoSQL context MV predates (§1–2, §9 schema evolution, §10 banking)
- [event_stream_processing_guide.md](event_stream_processing_guide.md) — event-driven integration around MV cores (§8–9)
- [data_governance_guide.md](data_governance_guide.md) — extract lineage, BCBS 239, metadata for MV-sourced data (§7, §10, §12)
- [feature_store_guide.md](feature_store_guide.md) — core-derived features for ML (§12)
- [data_model_resource_book_guide.md](data_model_resource_book_guide.md) — the relational design canon MV is contrasted with (§3)
- [core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) — core modernization (§8)
- [chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md) — 去IOE/xinchuang context (§5, §7)
- [oracle_flexcube_data_model_guide.md](../banking/oracle_flexcube_data_model_guide.md) — the relational-core counterpart

**Primary sources used for verification (Aug 2026):** jBASE International corporate history (via Internet Archive, "jBASE, TEMENOS and Mpower1"); Zumasys acquisition announcements (Jan 2015 jBASE; Oct 2021 sale to Rocket); Wikipedia "Rocket U2"; Rocket Software Product Lifecycle Handbook v1 (Nov 2025); Rocket jBASE 5.9.0/6.x release notes and docs; Rocket U2 documentation; dbdb.io (jBASE, UniVerse, OpenQM, D3); Temenos "T24 is now Temenos Transact" blog; Temenos R23 Stack-1 runbooks; International Spectrum coverage; TigerLogic acquisition announcement.

> **Honesty footer:** Product facts (versions, GA dates, support horizons, ownership dates, certifications) are as verified in August 2026 against the sources above. Version numbers and the Transact database-certification matrix change frequently — verify against rocketsoftware.com and the Temenos certification matrix for the exact release before architecture or procurement decisions. Items that could not be verified are flagged in-line (notably: the "jBASE source-available" claim; LLM-assisted MV tooling; exact T24 ACCOUNT field numbers, which are release-dependent).




