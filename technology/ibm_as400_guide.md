# IBM AS/400: The Midrange Platform Deep-Dive

*A comprehensive deep-dive on the IBM AS/400 — the midrange platform that refused to die: the 1988 launch and the Silverlake codename, the System/38 and System/36 lineage, the architecture (TIMI, single-level storage, object-based design), the operating system (OS/400 → i5/OS → IBM i), the hardware (CISC 1988 → RISC 1995 → POWER → Power10), the languages and the database (RPG IV, COBOL, CL, SQL, DB2 for i), the ecosystem (the ISVs, the banking Silverlake pattern), the modernization paths (PASE, open source on IBM i, Power Virtual Server), the enterprise reality (the banking cores, the flagged numbers), and a worked example — a SIBS-style banking core on the AS/400 and its modernization path.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Legacy-Systems / Enterprise-Hardware (technology/)
> **Audience:** Solution architects, integration architects, platform engineers, technical leads confronting AS/400-era estates in banking and beyond
> **Last Updated:** August 2026

**Cross-references:** [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) (the OCBC case — **Silverlake SIBS on IBM AS/400 (Power Systems), vendor-verified** — the flagship instance of the banking-core pattern in this series), [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) (the UOB case — **Silverlake-family SIBS cores across ASEAN, and UOB's 2025 acquisition of the vendor itself** — the other anchor instance of the same pattern), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (the **AS/400-as-legacy** angle — strangler fig, anti-corruption layer, batch-window reality), [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md) (the **cores** — where AS/400-class platforms sit in the vendor landscape), [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md) (the **AS/400 cores in China** — the regional extent of the pattern), [banking/tafj_guide.md](banking/tafj_guide.md) and [banking/temenos_guide.md](banking/temenos_guide.md) (the **alternative legacy platforms** — the JBase/TAFJ and T24 estates the AS/400 competes with in core banking), [technology/jbase_universe_guide.md](technology/jbase_universe_guide.md) (the **Pick-style alternative** — the other survivor of the midrange wars), [technology/oracle_database_guide.md](technology/oracle_database_guide.md) (the **database-systems** companion — cross-ref the DB2-for-i angle), [technology/data_integration_frameworks_guide.md](technology/data_integration_frameworks_guide.md) (lightly — the integration tooling that talks to the i), [technology/axway_transfer_cft_guide.md](technology/axway_transfer_cft_guide.md) (the **file-transfer** reality — how modern systems still move data to and from the AS/400), [technology/event_stream_processing_guide.md](technology/event_stream_processing_guide.md) (the **event-driven** path around the batch core).

**How to read this guide.** This is the *platform* deep-dive of the series: the sibling guides document what *runs on* the platform (the OCBC and UOB banking cores), the legacy guide documents *how to integrate* it, and this guide documents *what the platform itself is*. A fast path: read **§1** (what the AS/400 is and where it came from), **§2** (the architecture — TIMI and single-level storage, the reasons it survived), **§5** (the languages and the DB2-for-i database that everything is written in), and **§9** (the worked example that ties it to a real modernization decision); treat **§3** and **§4** as the reference history, **§6** as the market map, **§7** as the modernization menu, and **§8** as the honest reality check. Verification status of every factual claim is in **§11**; every term is in **§12**. Cross-references point at the series' sibling guides throughout.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The Architecture](#2-the-architecture)
3. [The Operating System](#3-the-operating-system)
4. [The Hardware](#4-the-hardware)
5. [The Languages and the DB](#5-the-languages-and-the-db)
6. [The Ecosystem](#6-the-ecosystem)
7. [The Modernization](#7-the-modernization)
8. [The Enterprise Reality](#8-the-enterprise-reality)
9. [The Worked Example: A Banking Core on the AS/400](#9-the-worked-example-a-banking-core-on-the-as400)
10. [The Summary: The Platform That Refuses to Die](#10-the-summary-the-platform-that-refuses-to-die)
11. [Verification and Claims-Status](#11-verification-and-claims-status)
12. [Glossary](#12-glossary)
13. [References](#13-references)

---

## 1. The Overview

### 1.1 The AS/400: The 1988 Launch

The **IBM AS/400** (Application System/400) is a family of midrange computers announced by IBM in **June 1988** (the commonly cited launch date is 21 June 1988) and shipped from **August 1988** ✅. It was positioned between the PC and the mainframe: a "midrange" system for the departmental and small-to-medium enterprise market — but with an architecture and an operating system that punched far above that positioning. It was the machine that ran a small company's entire back office, and the machine that ran a large bank's regional book: both from the same hardware line.

The launch was the largest product introduction in IBM's history to that point, replacing two product lines at once (see §1.2). The AS/400 shipped with a new operating system, **OS/400** (§3), an integrated relational database (now **DB2 for i**, §5), and a hardware/software boundary — the **TIMI** (§2) — that made it structurally immune to processor obsolescence. That last property is why the machine is still with us: the 1988 announcement date is real, and so is the fact that AS/400-era application binaries were still running — un-recompiled — on POWER hardware forty years later.

The names since 1988 are a tax on the memory:

| Year | Name | What happened |
|---|---|---|
| 1988 | **IBM AS/400** | Launched with OS/400 (announced June, shipped August) ✅ |
| 1995 | **AS/400e** | 64-bit RISC PowerPC models ("e" for the internet era) ✅ |
| 2000 | **eServer iSeries** | Rebranded into IBM's eServer line ✅ |
| 2004 | **eServer i5 / System i5** | 2004–2006: POWER5-based, OS renamed **i5/OS** ✅ |
| 2006 | **System i** | Dropped the "5" ✅ |
| 2008 | **IBM Power Systems** | System i and System p merged into one hardware line; OS renamed **IBM i** ✅ |
| Present | **IBM i on Power Systems** | POWER9/POWER10 servers, Power Virtual Server on IBM Cloud ✅ |

The terminology rule that matters: **the AS/400 is the 1988 machine, the platform it became is IBM i on Power Systems, and "AS/400" survives as the industry's habitual name for all of it** — including in this guide, where "the AS/400" is used for the platform line and "IBM i" for the current operating system.

### 1.2 The Lineage: The System/38 and the System/36

The AS/400 did not appear from nowhere: it was the deliberate successor to two IBM midrange families, and it absorbed both customer bases:

- **The System/38 (S/38)** — announced **24 October 1978**, first delivered **July 1980** ✅. The S/38 was the architectural ancestor: it pioneered **single-level storage** (a single flat address space spanning memory and disk), **48-bit addressing**, an **integrated relational database** (built into the machine, not bolted on), and an **object-based** programming model with machine-interface compilation — the direct predecessors of the AS/400's defining features (§2). The S/38 was ahead of its time and commercially difficult (expensive, proprietary), but its architecture became the AS/400's.
- **The System/36 (S/36)** — announced **16 May 1983** ✅. The S/36 was the *popular* one: a small-business workhorse, easy to program (RPG II, the SSP operating system), with a 17-year product lifespan. It was the volume seller IBM did not want to lose.
- **The merge.** The AS/400 brought the S/38 and S/36 lines together: it was **mostly based on the System/38 architecture** ✅, but OS/400 shipped with **compatibility environments** that ran System/36 and System/38 applications — three complete runtime environments (native AS/400, S/38, and S/36) managed by one OS ✅. Customers migrated by *restoring a backup* — IBM's famous "save and restore" migration path, which was possible because of the TIMI (§2).

The lineage claim in one line: **the AS/400 is the System/38's architecture wearing the System/36's market** — and that combination, plus binary-level migration, is why neither customer base was lost at the 1988 transition.

### 1.3 The Silverlake Codename

The AS/400 and OS/400 development program was known internally as **Project Silverlake** ✅. The name is documented in IBM's own history and in the OS/400 literature (the French Wikipedia's OS/400 article states directly that "le projet OS/400 (nom de code Silverlake)" was led by **Frank Soltis**, the IBM Fellow widely credited as the architect of the AS/400) ✅. Soltis's 1997 book *Inside the AS/400* (and its later editions, *Fortress Rochester*) is the canonical account of the project.

"Silverlake" matters beyond trivia for two reasons. First, it is the name the industry actually uses: in Southeast Asian banking, "the Silverlake" is commonly shorthand for the *Silverlake Axis* vendor and its SIBS core (see §6) — a different Silverlake, but the two are frequently conflated in conversation, and this guide's §6 keeps them distinct. Second, it anchors the design intent: Silverlake was a *clean-slate* program that still had to carry two existing customer bases (§1.2) — the constraint that produced the TIMI.

### 1.4 The Overview Table

| Aspect | Description |
|---|---|
| **What it is** | IBM's midrange computer family: between PC and mainframe; a self-contained business-computing platform (hardware + OS + database + languages as one integrated product) ✅ |
| **Launched** | Announced June 1988 (21 June per the commonly cited date), shipped August 1988 ✅ |
| **Codename** | Project Silverlake, led by Frank Soltis ✅ |
| **Lineage** | Successor to the System/38 (1978, architectural ancestor) and the System/36 (1983, market base); mostly S/38 architecture with S/36/S/38 compatibility ✅ |
| **Operating system** | OS/400 (1988) → i5/OS (2004) → IBM i (2008); the "i" stands for integrated ✅ |
| **Hardware** | CISC 48-bit processors (1988–1995) → 64-bit RISC PowerPC AS (1995) → POWER4/POWER5/POWER6/POWER7/POWER8/POWER9/POWER10 (2001–present) ✅ |
| **Defining innovations** | TIMI (hardware independence), single-level storage, object-based architecture, integrated relational database ✅ |
| **Primary languages** | RPG (RPG II/III/400 → RPG IV/ILE), COBOL, CL, SQL; now also Java, PHP, Python, Node.js ✅ |
| **Current status** | Still sold as IBM i on Power Systems; still the backbone of tens of thousands of businesses ⚠ (install-base numbers flagged in §8) |
| **Legacy reputation** | "The computer that refuses to die" — 1988 architecture still running core banking, ERP, and retail workloads ✅/⚠ (see §8) |

### 1.5 Why the Platform Matters to This Series

The AS/400 appears throughout this repository as the *runtime* of the Southeast Asian banking pattern — Silverlake SIBS on IBM AS/400 (Power Systems) at OCBC (vendor-verified in [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2.1) and at UOB's ASEAN franchises ([banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2.1), with the Chinese banking estates in [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md). Those guides treat the platform as a *fact about the bank*. This guide is the *platform* itself: what the machine is, why it was built that way, why it is still there, and what "modernizing" it actually means.

### 1.6 The Operating Experience: Green Screens, Commands, and ACS

To understand why the AS/400 estate persists, it helps to understand what *operating* one is like — because it is unlike anything in the modern Linux/cloud world:

- **The 5250 interface.** The platform's user and programmer interface for its first three decades was the **5250 green screen**: fixed 80×24 (or 132×27) text displays, driven by a terminal protocol (TN5250) over twinaxial cabling, later over TCP/IP. Operators and programmers spent their days at screens full of *WRKACTJOB* (work with active jobs), *DSPMSG* (display messages), and *WRKSPLF* (work with spooled files) — the command-driven, menu-driven world that the "green screen" stereotype refers to. Modern access uses **IBM i Access Client Solutions (ACS)** and TN5250 emulators (Windows, macOS, Linux) — the green screens are emulated now, but the *experience* is the same, and tens of thousands of operators still live in it ✅.
- **Everything is a command.** The AS/400's operator model is built on **CL commands** (§5.3): typed commands (*CRTLIB*, *SBMJOB*, *WRKUSRPRF*), command *menus*, and F-key navigation. There is no Unix-style free-form shell as the primary interface (though *STRQSH* and PASE shells exist); there is a structured command language with online help (*GO HELP*, F4 prompt) ✅.
- **Message queues, not logs.** The system communicates through **message queues** (system, job, user queues) — a *push* model where the operator answers messages ("job ended abnormally", "device needs varying on"), rather than grepping log files. The discipline of "answering the message queue" is the AS/400 operator's daily rhythm ✅.
- **The support model.** Because the whole stack (hardware, OS, DB, backup) is one product, an AS/400 shop runs with astonishingly small teams: one or two administrators per machine, with IBM's service infrastructure (hardware support, PTF — *Program Temporary Fix* — maintenance) handling the rest. For a modern architect the honest comparison: **an IBM i LPAR is closer to a managed appliance than to a server you assemble** — which is exactly why replacing it with "a Kubernetes cluster" is not the obvious win it sounds like.

The operating experience matters to modernization because it defines the *users'* reality: the estate is not "old screens nobody wants" — it is a working, disciplined operational culture with its own vocabulary, its own documentation, and its own experts. Any modernization program that ignores the 5250/CL/message-queue culture (e.g., by assuming "just give them a web UI") misunderstands what it is replacing.

---

## 2. The Architecture

### 2.1 The TIMI: The Technology Independent Machine Interface

The single most important thing about the AS/400 is a layer that users never see: the **Technology Independent Machine Interface (TIMI)** ✅. The TIMI is a **high-level instruction set** that sits between application programs and the machine's real hardware:

- Applications are **compiled to TIMI instructions**, not to machine code. When you compile an RPG or COBOL program on the AS/400, the compiler emits TIMI — a virtual, architecture-neutral instruction set defined by IBM, not by the silicon.
- Beneath the TIMI, the **Licensed Internal Code (LIC)** — IBM's microcode/OS kernel layer — *implements* the TIMI on whatever processor the machine actually contains. The LIC translates TIMI instructions to the real CPU and manages the machine's resources.
- The result: **application code is decoupled from the hardware**. When IBM changed the processor underneath (48-bit CISC → 64-bit RISC in 1995, then POWER4 through POWER10), the LIC was rewritten for the new silicon, and applications ran on the new hardware **without recompilation** — the famous 1995 migration was a *save, restore, and keep running* exercise, not a rewrite ✅.

The TIMI is why the platform survived. Every other machine of the 1988 era died when its processor family died, because applications were welded to the instruction set. The AS/400's applications were welded to the TIMI, which IBM controlled and kept stable for forty years. It is, in effect, a *binary-compatibility contract that outlived every physical processor it ran on* — the software industry's longest-running example of "compile once, run anywhere," decades before Java borrowed the idea.

A practical corollary for the modern architect: **the reason the AS/400 estate is still running is the same reason it is hard to migrate**. The application code (RPG, COBOL) is not portable to anything else — there is no TIMI on a Linux box. The platform's strength (hardware independence) was bought with the platform's weakness (software dependence): the *only* machine that can run TIMI-compiled programs is an IBM i machine. Modernization therefore means recompiling/porting (losing the TIMI benefit) or keeping the i (keeping the runtime).

### 2.2 The Single-Level Storage

The AS/400 implements **single-level storage** ✅: the machine presents **one flat virtual address space** that covers both main memory and disk, with no distinction visible to programs. The System/38 pioneered this with **48-bit addressing**; the 64-bit RISC AS/400 (1995) moved to a **64-bit address space** (about 18.4 *exabytes* of addressable space) ✅.

What this means in practice:

- **Programs and data live in the same address space.** A program refers to an object by its address; the system decides whether that address currently lives in RAM or on disk. There is no explicit "open the file / load the segment" dance in the traditional sense — the OS pages objects in and out transparently.
- **Addresses are permanent and never reused.** An object gets an address when it is created and keeps it for life. This is what makes the AS/400's pointer-based programming model safe across process lifetimes, and it underpins both the object model (§2.3) and the machine's famous resilience (no dangling references, no address reuse aliasing).
- **The database is part of the storage.** Because DB2 for i's tables are objects in the single-level store, the database is not a separate subsystem with its own buffer pools and file handling — it *is* the storage layer. The OS and the DB share one page-cache model. This integration is a large part of why the AS/400's transactional performance and recovery behavior are so distinctive, and why "the database is free and always there" (every IBM i ships with DB2 for i enabled — §5).

For the architect, the honest translation: **the AS/400's storage model is not like a Unix box's** (filesystem + RAM + swap, manually reconciled). It is closer to a modern *persistent-memory* ideal that most of the industry is still trying to reach — implemented in 1978 and kept ever since. The cost is that nothing else speaks the protocol, and "migrating the data off" means choosing a real export path (SQL, files, CDC — see [technology/data_integration_frameworks_guide.md](technology/data_integration_frameworks_guide.md) and [technology/axway_transfer_cft_guide.md](technology/axway_transfer_cft_guide.md)).

### 2.3 The Object-Based Architecture

Everything on the AS/400 is an **object** ✅: programs, files (tables), data areas, queues, user profiles, device descriptions — all are typed, self-describing objects managed by the OS. Each object has:

- **A type** (e.g., *PGM*, *FILE*, *USRPRF*, *DTAQ*, *OUTQ*), which defines what operations are legal on it;
- **Attributes** (size, authority, creation data, etc.);
- **An authority model** — access control is *on the object* (who can read/update/run/delete it), enforced by the OS at every reference, with the object's owner and the security officer (*QSECOFR*) governing it.

Two consequences define the platform's character:

1. **System security is object security.** There is no "root and everyone else" split as on Unix; there is a fine-grained, object-by-object authority model integrated into the OS. This is one of the reasons IBM i carries an unmatched **unbroken CVE record** — the operating system has never had a remote *security advisory* published in its history ⚠ (this claim, repeated in IBM's marketing, is widely repeated by practitioners; treat the "never" as the vendor's claim — flagged in §11).
2. **The object model + TIMI + single-level storage are one design.** Programs are objects compiled to TIMI instructions; they reference other objects by address in the single-level store; the OS mediates every access through the authority model. You cannot remove one pillar without the whole thing collapsing — which is precisely why the platform is monolithic by modern standards and why "lift the app off the i" is a rewrite, not a port.

### 2.4 The Architecture Table

| Component | Description | Significance |
|---|---|---|
| **TIMI (Technology Independent Machine Interface)** | High-level, architecture-neutral instruction set; applications compile to TIMI, not machine code; the Licensed Internal Code (LIC) implements TIMI on the real silicon ✅ | Hardware independence: the 1995 CISC→RISC and subsequent POWER transitions ran existing applications without recompilation — the platform's survival mechanism ✅ |
| **Single-level storage** | One flat 48-bit (S/38, CISC AS/400) or 64-bit (RISC, ~18 exabytes) address space spanning RAM and disk; permanent, never-reused object addresses ✅ | Transparent paging; integrated database; pointer-safe model; the reason "the DB is the storage" ✅ |
| **Object-based architecture** | Everything is a typed object (*PGM*, *FILE*, *USRPRF*...) with attributes and object-level authority enforced by the OS ✅ | Integrated security model; self-describing system; the reason backup/restore migrates whole machines cleanly ✅ |
| **Licensed Internal Code (LIC)** | IBM's microcode/OS layer beneath TIMI; implements TIMI, manages hardware, I/O, paging ✅ | The physical half of the TIMI contract; rewritten per processor generation so applications never notice ✅ |
| **Integrated relational database** | DB2 for i as a core OS component (in the single-level store), not an add-on ✅ | No separate DBMS to install/back up; DB2 for i is included with every system — the "free database" that anchors the platform's value ✅ |
| **Compatibility environments** | OS/400 shipped S/36 and S/38 runtimes alongside native AS/400 ✅ | Carried both customer bases through the 1988 transition; the migration path was restore-a-backup ✅ |

### 2.5 The Architecture Verdict

The 1988 design was, by the standards of its time, an act of extraordinary foresight — and it is the *same* design that makes the platform a legacy burden today. **The TIMI saved the platform from every hardware transition; it also sealed it inside its own instruction set.** Single-level storage made the database indestructible; it also made "the data" inseparable from "the machine" in a way that no other platform replicates. Modernization of an AS/400 estate is therefore never a port and always a *re-platforming or a rewrite* — the worked example in §9 walks through the realistic menu.

### 2.6 The Security Model in Practice

The object-based architecture (§2.3) produces a security model that is genuinely different from Unix/Windows:

- **Authority is per-object, per-user.** Each object carries an authority list (who can *read*, *update*, *add*, *delete*, *execute*, *manage* it), and users are *user profiles* (*USRPRF*) with their own authorities and group memberships. The **security officer** (*QSECOFR*) and the object owner administer it — there is no universal "root"; even the security officer works within the model ✅.
- **No open ports by default.** The system's services (and its famous default-deny posture) are administered centrally: network services, *inetd*-style daemons, and remote access are controlled per-service, per-interface. The platform's marketing claim — **"IBM i has never had a remote security advisory published"** — is repeated across the industry but is vendor-sourced ⚠ (flagged in §11); what is *structurally* true is that the integrated, object-level model eliminates whole classes of OS-level privilege-escalation bugs that plague general-purpose OSes.
- **The compliance angle.** For banking estates the model is a genuine asset: object-level audit journals, profile-based access, and integrated encryption (and, on modern releases, *Db2 for i* column/field-level encryption and external security managers such as RACF-family integration) map cleanly onto MAS/regulatory access-control and audit expectations ✅/⚠ (feature existence verified; per-bank compliance claims not).

The honest corollary: **the same integration that makes the i secure makes it hard to extend.** You cannot bolt a modern identity provider onto object authority without either the IBM tooling (Enterprise Identity Mapping, Kerberos/LDAP integration — which exists ✅) or a translation layer in front of the estate.

### 2.7 Backup, Restore, and the Save/Restore Culture

One operational consequence of single-level storage deserves its own section: **backup on the AS/400 is a system property, not an application chore.** *SAVSYS*, *SAVLIB*, *SAVOBJ* — the save commands — capture objects (including the database and the OS objects) in their consistent, address-stable form; *RSTLIB*/*RST* restore them. Because the object model is self-describing and addresses are permanent, **a full-system restore reconstructs a working machine** — including the OS, the configuration, the security model, and the data — to a known-good state, and "save/restore" is the platform's migration and DR mechanism of choice ✅. The 1995 CISC→RISC transition was, operationally, a save-and-restore event (§4.2); the same mechanics underpin PowerVS re-platforming (§7).

For the architect this is a double-edged inheritance: the save/restore culture is why AS/400 shops' DR plans are often *better* than the surrounding estate's (a full-system restore is scriptable and tested), and it is also why the estate's data has historically been treated as *one backup unit* — the granular, application-level data portability that modern integration wants comes from SQL and CDC (§5.5, §7), not from save/restore.

---

## 3. The Operating System

### 3.1 OS/400 (1988)

The AS/400's original operating system, **OS/400**, shipped with the machine in 1988 ✅. It was the sole operating system of the AS/400 line, and it inherited the System/38's **CPF** (Control Program Facility) heritage — IBM i's Wikipedia lineage states it explicitly as "an evolution of the System/38 CPF operating system" ✅. OS/400 was not a Unix variant and not a mainframe OS: it was a proprietary, object-based, database-integrated business OS, delivered with:

- The **Integrated Language Environment (ILE)** — introduced with OS/400 V2R3 (1993) — a binding model that lets programs written in different languages (RPG, COBOL, CL, C) be compiled to modules and bound into a single *program object*, with procedure calls across languages ✅. ILE is the reason mixed-language AS/400 codebases work at all.
- The **command language CL** (§5) and the **5250 terminal interface** — the green screens that defined the platform's user experience for decades.
- Batch job management (job queues, output queues, *SBMJOB*), spooling, message queues, and the **system journaling** that made the integrated DB transactional ✅.

### 3.2 i5/OS (2004)

In 2004, with the introduction of the **eServer i5** (POWER5-based) hardware, IBM renamed OS/400 to **i5/OS** ✅. The 2004 press release branding — "IBM i5/OS V5R3 — the next generation of OS/400" — is on the public record. The rename accompanied a positioning shift: the System i5 was pitched as an "all-in-one" server (compute, DB, and now optionally *other* operating systems — AIX and Linux in logical partitions, which the POWER5 generation supported well) ✅. i5/OS was OS/400 in substance — same objects, same TIMI, same DB — with the name catching up to the "i" branding.

### 3.3 IBM i (2008)

In 2008, IBM renamed the operating system a second time, to **IBM i** (the "i" standing for *integrated*) ✅. The rename accompanied the merger of the System i and System p hardware lines into **IBM Power Systems** (2008) — one server line running AIX, IBM i, and Linux side by side in logical partitions ✅. The naming transition is documented in the SAP documentation corpus ("As of operating system version IBM i 5.4, the operating system has been renamed from 'IBM i5/OS' to 'IBM i'") — the V5R4/V6R1 releases carried the change, with the first full "IBM i 6.1" release in 2008 ✅.

Since 2008 the platform has continued under the IBM i name through V7.1 (2010), 7.2 (2014), 7.3 (2016), 7.4 (2019), 7.5 (2022), and 7.6 (2025) — each release adding SQL and open-source capabilities while preserving the object/TIMI core (§7).

### 3.4 The OS Table

| OS | Era | Notes |
|---|---|---|
| **OS/400** | 1988–2004 | The original AS/400 OS; sole OS of the line; CPF (System/38) heritage; ILE introduced V2R3 (1993); RPG IV with V3R1 (1994); PASE AIX runtime added (V4R2 era, 1997 ⚠ date approximate) ✅ |
| **i5/OS** | 2004–2008 | Renamed with the eServer i5/POWER5 launch (2004); same OS substance; "all-in-one server" positioning with AIX/Linux LPARs ✅ |
| **IBM i** | 2008–present | Renamed with the Power Systems merger (2008, V5R4/6.1); current releases 7.1→7.6; modernized via SQL, REST, and open-source runtimes while keeping TIMI/object compatibility ✅ |
| **(The heritage line)** | 1978–1988 | System/38's CPF — the direct ancestor; single-level storage and object model born there ✅ |

### 3.5 The OS Character: One Integrated Stack

The defining property of the whole OS lineage is **integration**: OS, database, security, and object storage are one product with one backup, one upgrade, and one support contract. There is no "install the database" step, no "configure the filesystem" step, no "patch the kernel and pray the DB survives" step. This integration is why AS/400 shops run with tiny teams (a single administrator can run a machine serving a whole bank branch network), and why the platform's total cost of ownership has always been its killer argument — and, from a 2026 vantage, why it looks like a monolith: every modern architecture instinct (separate services, separate data stores, polyglot persistence) is the *opposite* of what the i is.

### 3.6 The Runtime Model: Subsystems, Jobs, and Batch

The AS/400's execution model is the other half of "how the OS works," and it explains the platform's batch character — the property that dominates every integration conversation about it (see [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §1.2):

- **Subsystems.** Work is organized into **subsystems** (*SBS*): managed pools of jobs with defined routing and priorities. The classic estate runs a *QBATCH* subsystem for batch jobs, *QINTER* for interactive (5250) sessions, and *QSPL* for spooling — each with its own job queues, memory pools, and activity rules ✅. *WRKACTJOB* (work with active jobs) is the operator's window into all of it.
- **Jobs and job queues.** Work is submitted as **jobs** to **job queues** (*SBMJOB*, *JOBQ*), where they wait for a subsystem to pick them up, run, and post results — with **spooled files** (*SPLF*, the printer/print-image queue) carrying reports and extracts. The whole estate runs on this queue-driven scheduling model, not on Unix-style cron daemons (though cron-style scheduling via *JOBSCD* and the modern *SBMJOB* + *WTR* patterns exists) ✅.
- **The batch window.** A banking core's day is a **batch cycle**: daytime online (enquiry, input capture) against a *journaled* database, then the **end-of-day (EOD)** run — interest accrual, posting, statement generation, report spooling, regulatory outputs — all as scheduled batch jobs with dependencies. This is the "batch window" that integration design must respect (§9); it is not a defect of the platform, it is its operating model ✅.
- **Journaling and commitment control.** DB2 for i's **journaling** (*DTAQ*-adjacent *JRN* objects) records every change to journaled files for recovery and, on modern systems, for **CDC** (change-data-capture feeds — the modern estate's event door, §7); **commitment control** (*COMMIT*, *ROLLBACK*) gives transaction atomicity across the DB ✅.

The runtime model is why "the AS/400" and "batch" are inseparable in the industry's mind: **the platform is a batch machine by design**, with online enquiry bolted on top — and every modernization conversation starts from that fact, not around it.

---

## 4. The Hardware

### 4.1 The CISC Era (1988)

The original AS/400 (1988) used **proprietary 48-bit CISC processors** ✅ — the machine's own CPU family, descended from the System/38's design lineage, with an instruction set (the *IMPI* lineage, "Internal Micro-Programming Interface") that the LIC hid behind the TIMI. The first models — the B-series (B10, B20, B30, B40, B50, B60, B70) — covered the range from small-office machines to data-center-class midranges, all running the same OS/400 and the same TIMI-compiled application binaries. Over the CISC era (roughly 1988–1995) the family progressed through the C, D, E, and F model series, each generation faster, none requiring application changes — the TIMI working as designed from day one.

### 4.2 The RISC Era (1995): The POWER, the AS/400e

In **June 1995** IBM announced the first **AS/400e** models based on **64-bit RISC PowerPC AS processors** ✅ — the machine's own POWER-derived RISC chips (the "AS" suffix = AS/400's variant of PowerPC). This was the transition the TIMI had been built for: the hardware was completely new (64-bit, RISC), the **Licensed Internal Code was rewritten** for it, and the *applications* simply did a save/restore and kept running ✅. The 1995 line (models 150, 170, 200, 400, 500, 530, 720, 730, 740) introduced the 64-bit address space (§2.2) and set the platform up for the internet era — hence the "e."

From 1995 onward the hardware tracked the POWER roadmap:

| Generation | Year | Notes |
|---|---|---|
| **PowerPC AS (AS/400e)** | 1995 | First 64-bit RISC AS/400; LIC rewritten; applications untouched ✅ |
| **POWER4 (iSeries)** | 2001 | AS/400 rebranded **eServer iSeries** (2000); POWER4-based iSeries servers ✅ |
| **POWER5 (i5)** | 2004 | The **eServer i5**; OS renamed i5/OS; first strong LPAR story for AIX/Linux alongside i5/OS ✅ |
| **POWER6** | 2007 | System i (2006 branding); decimal floating point in silicon ✅ |
| **POWER7** | 2010 | IBM i 7.1; huge memory bandwidth; the "smart" power management era ✅ |
| **POWER8** | 2014 | CAPI (coherent accelerator) era; IBM i 7.2 ✅ |
| **POWER9** | 2017 | IBM i 7.3/7.4; OpenCAPI/NVLink era ✅ |
| **POWER10** | 2020 | Current on-prem generation; IBM i 7.5; strong per-core multithreading (SMT8) ✅ |
| **Power Virtual Server (PowerVS)** | 2018+ | IBM i (and AIX/Linux) as a service on IBM Cloud — the "cloud IBM i" ✅ |

The pattern to internalize: **every hardware generation since 1988 has been a drop-in replacement for the applications, because the TIMI absorbed the change.** The machines the industry calls "AS/400s" today are POWER9/POWER10 Power Systems servers running IBM i — the 1988 architecture, the 1995 64-bit RISC base, the 2001+ POWER silicon, and (since 2008) a merged Power line that also runs AIX and Linux in partitions.

### 4.3 The Hardware Table

| Era | Processor | Notes |
|---|---|---|
| **CISC (1988–1995)** | 48-bit CISC (IMPI-lineage), B→F model series | Original AS/400 CPUs; TIMI + LIC from day one; every app compiled to TIMI ✅ |
| **RISC (1995–2000)** | 64-bit PowerPC AS | June 1995 AS/400e launch; LIC rewritten, apps save/restored; 64-bit addressing ✅ |
| **POWER (2001–present)** | POWER4 → POWER10 | iSeries (2000) → i5 (2004) → System i (2006) → Power Systems (2008); IBM i 7.1–7.6; current on-prem POWER10, cloud via PowerVS ✅ |
| **(The pattern)** | Any of the above | Applications compiled to TIMI run across all of them without recompilation — the platform's defining hardware fact ✅ |

### 4.4 The Hardware Reality Check

For the enterprise architect, the hardware table has one operative consequence: **an AS/400 estate is not "old iron" in the way a 1980s mainframe is.** The current machines are modern POWER servers with modern I/O (NVMe, 100GbE, CAPI-attached accelerators) — the "old" part is the software architecture and the application code, not the silicon. This is why "just retire the hardware" is a non-argument in AS/400 modernization conversations: the hardware is fine; the question is what to do with the RPG/COBOL estate that lives on top of it (§7, §9).

### 4.5 Form Factors and Models: From the B10 to the Power10

The hardware story, in the model numbers that practitioners actually recognize:

- **1988–1995 (CISC):** the B-series (B10 small office → B70 data-center class), then C, D, E, F series — rack and tower midranges, all running the same OS/400 and TIMI binaries. These are the machines of the platform's legend: unremarkable beige boxes that never went down.
- **1995–2000 (RISC):** the AS/400e line — models 150, 170, 200, 400, 500, 530, 720, 730, 740 — the 64-bit generation, from branch-office boxes to big symmetric-multiprocessing midranges ✅.
- **2000–2004 (iSeries):** the eServer iSeries — 270/800/810/820/830/840 — POWER4-based, with the "two boxes in one" pitch (integrated server + DB appliance) ✅.
- **2004–2008 (i5/System i):** the eServer i5 and System i — 520/550/570 and the big 595 — POWER5/POWER6, with LPARs running i5/OS, AIX, and Linux on one box; the "all-in-one server" era ✅.
- **2008–present (Power Systems):** the merged line — Power 520/550/720/740/750/770/780/795, then POWER8 (S8xx), POWER9 (S9xx: S914/S922/S924), and POWER10 (E1080/E980, S1014/S1022/S1024) — running IBM i, AIX, and Linux in LPARs, with PowerVM virtualization and, since 2018, the same LPAR images offered as **PowerVS** on IBM Cloud ✅.

The architectural constant across every model: **one OS image, one object store, one TIMI** — a machine is a machine, whether it is a B10 in a pharmacy or an E1080 in a bank's data center. LPARs (logical partitions, IBM's PowerVM hypervisor) let several "machines" (IBM i, AIX, Linux) share one box — which is how modern AS/400 estates co-locate the legacy core and the modern integration tier on the same physical hardware ✅.

---

## 5. The Languages and the DB

### 5.1 RPG (and RPG IV)

**RPG** — *Report Program Generator* — is the AS/400's signature language. Created by IBM in 1959–1960 for the punched-card era, it evolved through **RPG II** (System/3, System/36 — the volume language of the S/36 base), **RPG III** (System/38), **RPG/400** (the 1988 AS/400 compiler), and — the version that matters today — **RPG IV** ✅, introduced with **OS/400 V3R1 (1994)** as part of the **ILE** framework (ILE itself arriving with V2R3 in 1993) ✅. ILE RPG added free-format expressions, subprocedures, and modern program structure; the free-format RPG that IBM i 7.1+ (2010) promoted made RPG look almost like a modern language.

The honest description: **RPG is a domain-specific language for business data processing** — columnar output, record-oriented file (table) access, arithmetic on packed-decimal money fields, and (in its modern ILE form) proper modular procedures. It is not a language anyone learns in university, and it is *the* language of the AS/400 estate: the majority of business applications on the platform are written in it. For the migration architect it is the central problem: RPG exists on the IBM i and essentially nowhere else (there are RPG-to-Java/C# translation tools — e.g., the LANSA and ARCAD tooling families, and the *RPG-to-PowerBuilder* and *RPG-to-.NET* translators from various vendors ⚠ — but "translate RPG" is a rewrite with a compiler-assisted head start, never a port).

### 5.2 COBOL

**COBOL** (1959) is the AS/400's second language, shipped as COBOL/400 and then **ILE COBOL** with the ILE framework ✅. Banks and insurers in particular mixed RPG and COBOL on the same machine (and in the same bound program, thanks to ILE). The COBOL-on-the-i story is the same as COBOL everywhere (see [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §1.2): battle-tested business logic, decimal arithmetic, record structures, and a retiring skills base — with the i-specific twist that ILE lets COBOL modules share program objects with RPG and CL, so a "COBOL core" on the AS/400 is usually a mixed-language core.

### 5.3 CL (Control Language)

**CL** is the AS/400's command language and shell: system commands (*CRTPGM*, *SBMJOB*, *DSPLIB*...), batch job control, and full **CL programs** — ILE CL being a real compiled language with variables, logic, and file I/O ✅. CL is the glue of the estate: the nightly batch jobs, the startup sequences, the integration programs that call RPG/COBOL modules and drive the system. Any modernization exercise touches CL somewhere, because the *operating rhythm* of an AS/400 shop — the job queues, the output queues, the spool files, the end-of-day sequences — is expressed in CL. Modern IBM i adds PASE shells and *STRQSH* (a Unix-like shell) for scripting, but the CL job stream remains the skeleton of production.

### 5.4 SQL

**SQL** came to the platform as **SQL/400** in the late 1980s and is now, on modern IBM i, a first-class citizen: embedded SQL in RPG and COBOL, interactive SQL (*STRSQL*), ODBC/JDBC drivers, stored procedures, triggers, and (from 7.1 onward) SQL-centric development with JSON support ✅. The strategic point: **IBM's direction for IBM i is "SQL is the way in."** Every modern access path to the i — replication, CDC, analytics, REST — is SQL-shaped (see [technology/data_integration_frameworks_guide.md](technology/data_integration_frameworks_guide.md), [technology/oracle_database_guide.md](technology/oracle_database_guide.md) for the database-systems companion). A team that can write SQL can start modernizing an AS/400 estate *without touching RPG at all* — data access, reporting, and feeds can all be built on SQL against DB2 for i.

### 5.5 DB2 for i

**DB2 for i** is the platform's integrated relational database — the direct descendant of the System/38's built-in database and the lineage DB2/400 → **DB2 Universal Database for iSeries** → **DB2 for i** ✅. Its defining properties:

- **It is the OS.** DB2 for i lives in the single-level store (§2.2); tables are objects; the database cannot be "down" while the OS is up, and there is nothing to install, license separately, or back up independently ✅.
- **It is included and always on.** Every IBM i system ships with DB2 for i enabled — the "free database" that anchors the platform's cost story ✅.
- **It is genuinely capable.** DB2 for i supports full SQL, referential integrity, triggers, stored procedures, user-defined functions, journaling with forward recovery, and (modern releases) SQL windowing, JSON, and Db2 Mirror for high availability ✅. Its query engine routinely outperforms expectations on the platform's I/O model ⚠ (performance claims are workload-specific — flagged).
- **It is the migration surface.** Because DB2 for i speaks standard SQL, it is the *portable* part of the estate: data can be extracted, replicated (CDC), or queried in place from anywhere. "The data is trapped on the AS/400" is, on modern systems, a myth — the *code* is trapped; the data has a door (SQL), and that door is DB2 for i.

### 5.6 The Languages Table

| Language | Role | Notes |
|---|---|---|
| **RPG (RPG II/III/400 → RPG IV)** | The platform's signature business language; majority of the application estate | RPG IV + ILE since V3R1 (1994); free-format since 7.1; compiles to TIMI; essentially i-only — the central migration problem ✅ |
| **COBOL (COBOL/400, ILE COBOL)** | Second language; banks/insurers; mixed with RPG via ILE | Same legacy profile as COBOL everywhere; shares program objects with RPG/CL under ILE ✅ |
| **CL (Control Language)** | Command language, batch control, integration glue | CL programs are compiled ILE members; the estate's operating rhythm (job queues, EOD) is CL ✅ |
| **SQL (SQL/400 → DB2 for i SQL)** | The modern access path; embedded SQL, ODBC/JDBC, stored procedures, JSON | IBM's direction: SQL as the way in; the data-side migration door ✅ |
| **DB2 for i** | Integrated relational database — OS component, not add-on | DB2/400 → DB2 UDB for iSeries → DB2 for i; free, always on, single-level-store resident; CDC/SQL export surface ✅ |
| **(Modern additions)** | Java, PHP, Python, Node.js, C/C++ | ILE and PASE runtimes bring the modern languages to the i (see §7) ✅ |

### 5.7 The 5250 Interface: The User Experience That Defines the Estate

No languages section is complete without the interface those languages talk to: the **5250 display protocol**. In the AS/400 world, "the application" is very often *a set of 5250 screens* — the green screens of §1.6 — generated by DDS (Data Description Specifications) display files or by the modern *display-file-free* SQL/JSON interfaces. The estate's business logic is reachable through these screens, and the *integration* reality follows from that: for decades, the only way to "call" an AS/400 program from outside was to drive its 5250 screens (screen scraping) or exchange files (§6 of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md)). Modernization tooling (Profound Logic, LANSA, Fresche-class products ⚠) converts 5250 DDS to web UIs — a large sub-industry of the ecosystem (§6.3) exists precisely to modernize the screen layer without touching the RPG behind it.

### 5.8 A Taste of the Code: RPG IV, CL, and SQL

For the architect who has never seen the platform's code, three tiny examples convey more than prose:

**ILE RPG IV** — free-format, procedure-based, reading a customer record from DB2 for i (the *CUST* physical file/table):

```rpg
**free
ctl-opt dftactgrp(*no) actgrp('CORE') bnddir('CORE');

dcl-proc GetBalance;
  dcl-pi *n packed(15:2);
    pCustId char(10) const;
  end-pi;
  dcl-f CUST keyed;
  dcl-s balance packed(15:2);

  chain pCustId CUST;
  if %found;
    balance = CUSTBAL;          // packed-decimal money field
    return balance;
  endif;
  return 0;
end-proc;
```

**CL** — the nightly job that runs the end-of-day batch (the estate's operating rhythm, §3.6):

```cl
PGM        PARM(&P_DATE)
DCL        VAR(&P_DATE) TYPE(*CHAR) LEN(10)
CHKOBJ     OBJ(LEDGER) OBJTYPE(*FILE)
MONMSG     MSGID(CPF0000) EXEC(GOTO CMDLBL(ERROR))
OVRDBF     FILE(LEDGER) TOFILE(APP/LEDGER) +
           MBR(*LAST)
CALL       PGM(APP/ACCUM) PARM(&P_DATE)
CALL       PGM(APP/POSTIT) PARM(&P_DATE)
SNDMSG     MSG('EOD posting complete for ' *CAT &P_DATE) +
           TOUSR(*SYSOPR)
GOTO       CMDLBL(EXIT)
ERROR:     SNDPGMMSG  MSGID(CPF9898) MSGF(QCPFMSG) +
                       MSGDTA('EOD failed for ' *CAT &P_DATE) +
                       MSGTYPE(*ESCAPE)
EXIT:      ENDPGM
```

**SQL against DB2 for i** — the data door (§5.5) that modern integration actually uses:

```sql
-- the ledger cache feed: everything that moved today
SELECT CUST_ID, ACCT_ID, POST_DATE, AMOUNT
FROM   LEDGER
WHERE  POST_DATE = CURRENT DATE
  AND  JOURNAL_ENTRY > :LAST_SEEN_SEQ
ORDER BY JOURNAL_ENTRY;
```

What these three snippets encode: **RPG is the business logic, CL is the schedule, SQL is the door.** A modernization team that understands those three roles can build the whole §7/§9 strategy; the code itself is the legacy, and it is portable only through SQL (data) or translation (logic).

---

## 6. The Ecosystem

### 6.1 The ISV Pattern: The Platform's Real Business

The AS/400 was never primarily a *hardware* business — it was an **ISV (Independent Software Vendor)** business ✅. IBM built the platform and the OS, and an enormous industry of software vendors built the actual applications: ERP, distribution, retail, manufacturing, healthcare, and — critically for this series — **core banking**. The pattern that made the platform dominant in the midrange market was *vertical software on a rock-solid box*: the customer bought the vendor's package, the vendor certified it on the AS/400, and IBM's hardware/OS reliability did the rest. This is exactly the shape of the banking pattern in this series: **Silverlake Axis's SIBS on IBM AS/400 (Power Systems)** is a packaged core running on the platform — see [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2.1 (vendor-verified at OCBC) and [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2.1 (UOB's ASEAN SIBS estates, and UOB's 2025 acquisition of Silverlake Axis itself — a bank that bought its core vendor to control its roadmap).

### 6.2 The Banking Segment: Silverlake Axis and the SIBS Pattern

**Silverlake Axis** is the Malaysian-headquartered, SGX-listed core-banking vendor whose **SIBS (Silverlake Integrated Banking Solution)** is the canonical AS/400 banking core of Southeast Asia (the "Silverlake" of the banking world — distinct from the 1988 project codename of §1.3, and worth keeping apart in conversation). The SIBS-on-AS/400 pattern, documented in the sibling guides:

- **OCBC** — "Silverlake Axis Integrated Banking Solution (SIBS) on an IBM AS400 (Power Systems) Platform" — vendor-published case study (excerpted from an IDC case study, September 2013) ✅ (see [OCBC guide](banking/ocbc_software_systems_guide.md) §2.1; the per-franchise scope — which OCBC entities run SIBS — is flagged ⚠ there).
- **UOB** — Silverlake-family SIBS across the ASEAN franchises at industry-consensus level (⚠ not vendor-case-study level); and since 2025, UOB **owns** Silverlake Axis ✅ (see [UOB guide](banking/uob_software_systems_guide.md) §2.1).
- **The class itself** — SIBS is a batch-oriented, mainframe-class core: deposits (CASA), loans, accounts, transactions; end-of-day posting; RPG/COBOL-class code; tight coupling to the platform. The mechanics (interest accrual, posting, batch windows) are generic to the core class and documented in [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md) and the mechanics guides.
- **China** — the same platform pattern reached the Chinese banking estate; see [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md) for the AS/400 cores in China.

The honest cross-ref: **the OCBC guide's §2 is the cleanest public instance of "the AS/400 in banking"** — vendor-verified, IDC-sourced, and explicitly framed as the hard-to-modernize estate class. This guide's §9 builds its worked example on exactly that shape.

### 6.3 The Wider Ecosystem Table

| Segment | Players | Notes |
|---|---|---|
| **Core banking (SE Asia)** | Silverlake Axis (SIBS); also Temenos, FLEXCUBE on other platforms | SIBS on IBM AS/400 (Power Systems) is the regional workhorse; OCBC ✅ vendor-verified, UOB ✅ consensus-level + vendor acquisition; cross-ref [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md), [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md), [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md) |
| **ERP / manufacturing / distribution** | SAP (R/3 on AS/400 historically), JD Edwards (World, OneWorld), BPCS, JBA, MAPICS, Infor (LX, XA), VAI, Ramsey | The classic ISV base that made the platform a volume business; many still run on IBM i ⚠ (per-vendor install bases not verified here) |
| **Retail / POS / wholesale** | JDA-style distribution suites, bespoke 5250 estates | AS/400 as the retail back office — inventory, pricing, store systems ✅ (platform class verified; specific vendors flagged ⚠) |
| **Healthcare / insurance** | Policy administration and claims packages | See [banking/insurance_software_systems_guide.md](banking/insurance_software_systems_guide.md) for the domain's platform classes |
| **Tools / modernization vendors** | LANSA, ARCAD, Fresche Solutions, Profound Logic, Rocket Software, HelpSystems/Fortra, BCD, IBM (Rational, IWS) | The ecosystem that keeps the estate alive: 5250 modernization, RPG translation, HA/DR (PowerHA, iCluster, MIMIX), DevOps for IBM i ✅ (class-level, vendor names flagged ⚠) |
| **IBM itself** | Power Systems, IBM i, DB2 for i, PowerVS, IBM i Services, ACS (Access Client Solutions) | The platform owner: hardware + OS + DB + cloud + modern access tooling ✅ |

### 6.4 The Ecosystem Verdict

The AS/400 ecosystem is the *reason the platform survived*: not the hardware, not the OS, but **the installed application base of ISV packages and bespoke RPG/COBOL systems that represent decades of accumulated business logic**. Every ecosystem segment above is, in 2026, a *maintenance and modernization* market more than a growth market ⚠ (flag: platform growth claims are vendor-sourced; the honest statement is that the estate persists at scale — §8 — and the ecosystem's revenue is in keeping it running and moving it forward).

### 6.5 How the Ecosystem Formed: The ISV History

The ecosystem's shape is a product of the 1980s–1990s midrange market, and it explains both the platform's persistence and its banking footprint:

- **The packaged-application revolution.** Before the AS/400, midrange software was mostly custom. The AS/400's combination of an integrated DB (no DBA needed), a single OS (no porting across flavors), and cheap hardware created the first mass market for *packaged* vertical software: **JD Edwards (World), SAP (R/3 was certified on AS/400), MAPICS, BPCS (SSA), JBA, and the distribution/retail suites** turned the platform into the ERP workhorse of mid-market manufacturing and distribution ✅/⚠ (platform relationships verified historically; current install-base per vendor flagged). A distinctive cultural trait: AS/400 packages often shipped **with source code included** — the customer bought the *code*, not just the binaries, and customized freely. That is why the platform's estates are so deeply bespoke: decades of local modification on top of packages ✅ (the "source included" practice is documented across the AS/400 ISV literature ⚠).
- **The banking lineage.** The core-banking packages that dominate Southeast Asia have midrange roots: **Silverlake Axis's SIBS** evolved from the Jack Henry family of midrange banking systems (per the KGI initiation report's lineage note ⚠ — flagged in [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2.1), i.e., the same *package + platform* recipe applied to banking. The OCBC and UOB estates are the surviving, heavily-customized descendants of that 1990s wave ✅/⚠.
- **The modernization industry.** Because the estate persists, an entire vendor layer exists to keep it modern: **LANSA and ARCAD** (development/DevOps tooling and RPG modernization), **Fresche Solutions, Rocket Software, Profound Logic** (5250-to-web, data movement), **Fortra (HelpSystems) and BCD** (ops, security, HA), plus IBM's own **Rational** and **Integrated Web Services** tooling ⚠ (vendor class verified; individual claims not audited). In 2026, this layer — not new ISV applications — is the ecosystem's growth business.

The history lesson: **the AS/400 ecosystem was built on package-plus-platform and source-included customization; it persists because the customized packages became the customer's business; and it now monetizes modernization rather than new sales** — a lifecycle that exactly mirrors what happened in core banking (see [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md) for the vendor-landscape companion).

---

## 7. The Modernization

### 7.1 IBM i on the Power: The Platform Is Still Sold

The first modernization fact is counterintuitive: **the AS/400 platform is still a current product.** IBM sells IBM i on POWER9/POWER10 Power Systems (and, since 2018, as **IBM Power Virtual Server (PowerVS)** — IBM i as a service on IBM Cloud) ✅, with IBM i releases through 7.6 (2025) ✅. The platform is not in maintenance mode; IBM continues to invest in it — SQL, open source, security, and cloud. "Modernizing the AS/400" therefore does *not* mean "get off the platform" as a default; for many workloads the rational path is *modernize on the platform* — and IBM's own roadmap is built around that.

### 7.2 The Modern Paths: PASE and the Open Source on the i

**PASE — the Portable Application Solutions Environment.** Introduced in the OS/400 V4R2 era (1997) ⚠ (date approximate, flagged), PASE is an **AIX runtime environment integrated into the OS**: it provides **binary compatibility for user-mode AIX executables** (32-bit and 64-bit AIX ABIs) without emulation or interpretation ✅. In plain terms: a *lot* of Unix software can be compiled for AIX and run on the i as-is, in a PASE shell, alongside native IBM i objects. PASE is the door through which the modern software world entered the platform: it underpins the open-source story below, and it lets shops run AIX-flavored middleware (WebSphere-class app servers, Perl, and the like) on the same box as the RPG core.

**The open source on the i (IBM i OSS).** Since ~2018 IBM has shipped an **open-source stack for IBM i via yum/RPM** (the 5733-OPS option, superseded by the base 5770-OSS/OSS1 offering) ✅, making **Node.js, Python, PHP, Git, nginx, and a large package catalogue** installable with a package manager on the i ✅. The practical consequences for an estate:

- **Node.js and Python on the i** mean modern services can run *on the platform*, beside the RPG core, using standard drivers (ODBC/JDBC, the IBM_DB module) to reach DB2 for i — the pattern for building REST/API layers directly on the machine.
- **Git on the i** (with the ILE code via the IBM i "Git for IBM i" tooling) means the RPG/CL estate can finally enter version control and CI/CD — often the single highest-leverage modernization step available.
- PHP has been on the i since the early 2000s (Zend Server era) and is a major web-fronting language for AS/400 applications ✅.

The honest framing: **PASE and OSS give the i a modern application tier without leaving the machine.** The modernization menu below is built on that.

### 7.3 The Modernization Table

| Path | Description | Use case |
|---|---|---|
| **Modernize on-platform (SQL + APIs)** | Expose the estate via SQL (DB2 for i), REST (IBM i Integrated Web Services / IWS, REST APIs), and data access (ODBC/JDBC/JSON); build new capabilities in Node/Python/PHP beside the core | The default path: keep the RPG core, make it a service provider; lowest risk; immediate value ✅ |
| **PASE/AIX applications** | Run AIX binaries (and AIX-flavored middleware) in the PASE environment on the i | Bringing Unix-oriented middleware/tooling onto the box without porting ✅ |
| **Open-source tier (IBM i OSS)** | Node.js, Python, Git, nginx via yum/RPM; CI/CD for ILE code; web front ends | Modern services, DevOps, and web layers *on* the platform ✅ |
| **Re-platform to PowerVS** | Move the IBM i partition (LPAR) to IBM Power Virtual Server on IBM Cloud — same OS, same TIMI programs, same DB, cloud billing | Data-center exit / cloud strategy without a rewrite; DR and capacity flexibility ✅ |
| **HA/DR hardening** | PowerHA for i, Db2 Mirror, third-party replication (iCluster, MIMIX) | Availability modernization; the estate's availability requirements usually outgrow single-box setups ⚠ (vendor names flagged) |
| **Strangler / wrap** | Build the new architecture in front of the core — APIs, event streams (CDC from DB2 for i journals), channel layer — and retire core functions incrementally | The banking pattern (OCBC/UOB digital layers in front of SIBS-class cores; see [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2.4, [technology/legacy_integration_patterns_guide.md](technology/legacy_integration_patterns_guide.md) §6) ✅ |
| **Re-engineer / replace** | Translate RPG/COBOL to Java/C#/.NET (LANSA, ARCAD, Fresche-class tooling) or replace the package (SIBS → T24 → modern core) | When the core itself must leave the platform; the expensive path — a rewrite in practice ⚠ (tooling class verified; outcomes workload-dependent) |
| **Hybrid** | Keep the i as system of record; move everything else off; connect via file transfer (see [technology/axway_transfer_cft_guide.md](technology/axway_transfer_cft_guide.md)) and events (see [technology/event_stream_processing_guide.md](technology/event_stream_processing_guide.md)) | The realistic 2026 posture for most large estates ✅ |

### 7.4 The Modernization Verdict

The honest modernization message: **the i is not a dead-end you must escape; it is a monolith you must open.** The platform itself provides the opening tools (SQL, PASE, OSS, REST, PowerVS) — what it does *not* provide is a way to make RPG code portable. Every modernization strategy for an AS/400 estate therefore reduces to: *open the data (SQL/CDC), wrap the code (APIs), modernize the operations (OSS/DevOps/HA), and decide per-capability whether the RPG core is re-engineered or retained.* §9 turns this into a concrete design.

### 7.5 The Modern Access Stack: The Tools the i Gives You

Concretely, the modern IBM i estate is *reachable* through a stack that a mainstream architect will recognize:

- **SQL everywhere.** *IBM i Access Client Solutions* (ACS) ships a full SQL workbench (*Run SQL Scripts*); ODBC/JDBC/.NET drivers are standard; DB2 for i supports SQL procedures, triggers, UDFs, JSON, and *IBM i Services* — system-management data (jobs, objects, authorities, performance) exposed as SQL tables/views, so even *system administration* can be done in SQL ✅.
- **REST and web services.** *IBM i Integrated Web Services (IWS)* turns RPG/COBOL programs and SQL routines into SOAP/REST web services without code changes; modern releases add native REST and JSON support, and Node.js/Python tiers (IBM i OSS, §7.2) provide the framework-native API layer ✅.
- **Data movement and events.** DB2 for i **journal-based CDC** feeds Kafka-class event platforms (the pattern in [technology/event_stream_processing_guide.md](technology/event_stream_processing_guide.md)); file transfer remains the workhorse for batch estates ([technology/axway_transfer_cft_guide.md](technology/axway_transfer_cft_guide.md) — PeSIT/SFTP into the i's IFS — the *Integrated File System*, the i's POSIX-style directory tree that also holds the modern files ✅).
- **Security and identity.** SSH (PASE), SSL/TLS everywhere, Kerberos/LDAP/Enterprise Identity Mapping integration, externalized authority via *IBM i Access* security and (on Power) the **PowerSC** compliance tooling ✅/⚠ (feature class verified; specifics flagged).
- **Cloud.** **PowerVS** (IBM Power Virtual Server on IBM Cloud) provisions IBM i LPARs as cloud VMs — the same OS, the same TIMI programs, the same DB2 for i, with cloud networking/billing — the "lift and shift that actually works" story of the platform ✅.

The point for the modernizer: **almost nothing about "modern" access to the i requires leaving it.** The bottleneck is the application code and the organizational culture around it — not the connectivity.

---

## 8. The Enterprise Reality

### 8.1 The AS/400 in the Enterprises: Still There, Still Critical

The uncomfortable fact of enterprise IT is that **the AS/400 never went away** — it just stopped being news. Four decades after launch, IBM i estates still run core banking books, ERP installations, retail operations, distribution networks, healthcare back offices, and government systems around the world. The platform's persistence is not nostalgia: it is the accumulated weight of (a) business logic that works and is paid for, (b) an availability/reliability record that modern distributed systems struggle to match, (c) an integrated DB + OS that runs with tiny teams, and (d) the TIMI, which means the *hardware* is current even where the *software* is old (see §4.4).

The **banking cores** are the anchor case in this series: Silverlake SIBS on IBM AS/400 (Power Systems) at OCBC (vendor-verified) and across UOB's ASEAN franchises (consensus-level, with UOB's 2025 acquisition of the vendor) — plus the Chinese banking estate's AS/400 cores (see [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md)) — see §6.2 and the sibling guides for the evidence trail. Where the AS/400 runs a bank's core, it is not a peripheral system: it is the system of record for deposits, loans, and accounts, and the batch window it owns is the bank's operating rhythm.

### 8.2 The Numbers: Flagged Honestly

The install-base statistics circulating for the platform are **marketing-sourced and should be flagged**:

- **"More than 100,000 companies worldwide still use AS/400 systems"** — attributed to Fortra's IBM i marketing content and repeated widely ⚠. This is a vendor-ecosystem figure, not an audited census; treat as "on the order of 100,000+ customer organizations," directionally credible, not precisely verifiable.
- **Community estimates of 150,000–300,000+ IBM i systems (LPARs/boxes) in production** circulate in the IBM i press (IT Jungle and similar) ⚠ — consistent with the 100k+ customer figure but unverified.
- **"The majority of the world's..." claims** (e.g., "X% of retail transactions touch IBM i," "IBM i runs the backbone of Fortune 500 supply chains") — **do not verify** in this pass ⚠; treat any such percentage as marketing unless a primary source is produced.
- **What *is* verifiable**: the platform is still a current, sold, supported IBM product (POWER10, IBM i 7.6, PowerVS) ✅; the banking pattern is documented in vendor/IDC case studies (OCBC ✅); and the 1988 launch, the TIMI, and the CISC→RISC transition are all documented history ✅.

The discipline of this series: **numbers flagged, patterns verified.** The *shape* of the AS/400 reality — large, persistent, banking-critical, hard to replace — is verifiable from the case-study record; the *magnitude* claims are vendor-sourced and marked accordingly.

### 8.3 The Reality Table

| Domain | AS/400 role | Notes |
|---|---|---|
| **Banking cores (SE Asia)** | Silverlake SIBS on IBM AS/400 (Power Systems): deposits, loans, accounts, EOD batch | OCBC ✅ vendor-verified; UOB ✅ consensus + 2025 vendor acquisition; cross-ref [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2, [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2 |
| **Banking cores (China)** | AS/400-class cores in the Chinese banking estate | Cross-ref [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md) ⚠ (per-bank detail flagged there) |
| **ERP / manufacturing / distribution** | SAP, JD Edwards, BPCS/JBA/MAPICS/Infor-class ERP and distribution suites | The classic ISV volume base; still substantial ⚠ (install-base per vendor not verified) |
| **Retail / wholesale** | Back-office systems of record: inventory, pricing, store operations | Platform class verified; the "AS/400 in retail" anecdote base is large ⚠ (systematic stats unverified) |
| **Healthcare / insurance / public sector** | Policy, claims, and administrative systems | Domain platform classes in [banking/insurance_software_systems_guide.md](banking/insurance_software_systems_guide.md) ⚠ |
| **The integration boundary** | The batch/fidelity reality: file transfer, EOD windows, 5250 screens | The legacy-integration angle: [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §1; file movement via [technology/axway_transfer_cft_guide.md](technology/axway_transfer_cft_guide.md) |
| **The skills reality** | RPG/COBOL/CL expertise concentrated in retiring generations; new grads don't learn it | The fundamental constraint; drives both the "wrap it" strategy and the vendor-consolidation wave (UOB buying Silverlake) ✅/⚠ |

### 8.4 The Enterprise Verdict

For the solution architect, the enterprise reality of the AS/400 is best summarized as **a criticality/visibility asymmetry**: the platform carries some of the most important workloads in the world's banks and businesses, while being almost invisible to the modern engineering discourse. That asymmetry is precisely why it keeps appearing in integration and modernization programs — and why this guide exists: the platform deserves a proper understanding before anyone draws a retirement arrow through it.

### 8.5 The Southeast Asian View: Why This Guide Lives in This Series

For the Singapore-based reader, the AS/400 reality has a distinctly regional face:

- **The SIBS heartland.** Southeast Asia is the global heartland of the SIBS-on-AS/400 banking pattern: the OCBC (vendor-verified) and UOB (consensus + 2025 vendor acquisition) cases, plus the wider Silverlake client footprint the KGI corpus describes ("prominent banks such as OCBC, UOB and CIMB are using Silverlake's CBS" ⚠ per [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2.1). When a regional bank talks about "the core," it is often talking about an IBM i box that predates its own mobile app by thirty years.
- **The acquisition signal.** UOB buying Silverlake Axis (2025) is the market's clearest statement about the platform's future: **the banks are not exiting the AS/400; they are buying the roadmaps that run on it** (see [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2). Modernization in this region is overwhelmingly *wrap-and-extend* rather than replace — the OCBC posture of "technological transformation and refresh" of the SIBS estate rather than a vendor swap ✅ (per the Silverlake/IDC case study framing in [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2.4).
- **The skills market.** The region's RPG/COBOL/CL skills sit with the generation that built the 1990s cores; the MAS-supervised banks run these estates under the same regulatory scrutiny as their cloud stacks. The practical consequence: **integration and data-access skills (SQL, CDC, APIs, file transfer) are the scarce, hireable skills of AS/400 modernization** — the RPG rewrites are neither cheap nor abundant, which is precisely why the §7/§9 strategy deliberately avoids them.

---

## 9. The Worked Example: A Banking Core on the AS/400

### 9.1 The Scenario: A SIBS-Style Core

*Merlion Bank* (fictional, but shaped exactly like the documented pattern — see [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2 and [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2) runs its retail book on **Silverlake Axis SIBS on IBM i (Power Systems)**: a POWER9 server hosting the SIBS core — deposits (CASA, term deposits), loans, accounts, and transaction posting — with end-of-day batch as the posting rhythm, RPG/COBOL/CL application code compiled to TIMI, DB2 for i as the database, and a small operations team running it with the platform's traditional efficiency. Around it: a mobile app and internet banking (2020s stack), a payments hub, a data warehouse, and regulatory reporting — all *reaching into* the core via an integration layer.

The situation the architecture team faces:

- The core works and is the system of record. There is **no business case to replace it** (the SIBS estate was refreshed, not swapped — the OCBC case study's own framing).
- But the estate needs: **real-time balances** in the app (the core is batch-posted), **event feeds** for fraud and analytics, **API access** for partners, **DevOps** for the RPG/CL estate (currently maintained with traditional tools), **regulatory reporting** (MAS-style) that wants data *now*, not at end of day, and a **cloud/datacenter-exit** question.
- The classic constraints apply: the batch window is sacred, the RPG code is not portable, the DB2 for i data is portable (SQL), and the skills to change the core are scarce and expensive.

### 9.2 The Platform Modernization Path: The Design

The design follows the modernization table of §7.3, applied in order — **open, wrap, extend, then selectively re-engineer**:

**Phase 1 — Open the data (no code changes to the core).**
- DB2 for i is the door: enable **journaling-based CDC** on the core tables (deposits, loans, accounts) and stream changes to the data/event platform (Kafka-class; see [technology/event_stream_processing_guide.md](technology/event_stream_processing_guide.md)) via standard CDC tooling — this is exactly the "event-fed balances" pattern the OCBC guide's §2.4 describes for AS/400 cores ✅/⚠ (pattern verified; Merlion's specifics fictional).
- Point regulatory reporting and the warehouse at DB2 for i via **SQL replication / extract** (see [technology/data_integration_frameworks_guide.md](technology/data_integration_frameworks_guide.md)), replacing overnight file pulls (the [technology/axway_transfer_cft_guide.md](technology/axway_transfer_cft_guide.md) world shrinks but doesn't vanish).
- Result: real-time balances *outside* the core (ledger cache fed by CDC), zero change to SIBS/RPG.

**Phase 2 — Wrap the core (APIs on the machine).**
- Expose enquiry and transaction APIs using **IBM i Integrated Web Services / REST on the i**, backed by SQL stored procedures — or run a **Node.js API tier on the i** (IBM i OSS) calling DB2 for i via the IBM_DB driver, co-located with the core in the same LPAR (see [technology/api_governance_guide.md](technology/api_governance_guide.md) for the API layer governance).
- The mobile app, payments hub, and partners now call *APIs*, not 5250 screens or file extracts. This is the anti-corruption layer of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6, built on-platform.

**Phase 3 — Modernize operations (DevOps for the ILE estate).**
- **Git for IBM i** + a CI/CD pipeline for RPG/CL/COBOL source (the ARCAD/LANSA-class tooling or IBM's own); automated build/package of ILE programs.
- HA/DR hardening (PowerHA for i or Db2 Mirror; third-party replication as an alternative ⚠ vendor names flagged) so the core's availability posture matches the bank's digital ambition.

**Phase 4 — Re-platform decisions.**
- **Re-platform the LPAR to PowerVS (IBM Power Virtual Server on IBM Cloud)** if the datacenter-exit/cloud strategy demands it — same OS, same TIMI programs, same DB, same batch windows, cloud billing ✅ (this is precisely PowerVS's documented value proposition). Alternatively stay on-prem POWER10. The TIMI makes this a *hardware* decision, not a software one — the rare legacy system where "lift and shift" is genuinely trivial.

**Phase 5 — Selective re-engineering (the long game).**
- Identify the core functions that genuinely need to leave the platform (e.g., the accounts/payments engine that must go real-time-native) and re-engineer *those* against the modern ledger/payments stack, strangler-style — the new engine takes over one product line at a time while SIBS keeps the rest.
- Everything that is *fine* stays on the i. The honest rule: **the i keeps what the i is best at** (batched, transactional system-of-record processing with integrated DB2 for i); the modern estate takes what the modern estate is best at (real-time, event-driven, elastic).

### 9.3 The Lessons

1. **The TIMI is the gift that keeps on giving — and the lock-in that never expires.** Merlion's apps ran unchanged across POWER generations; that is the platform's survival mechanism, and it is also why "migrate off" means "rewrite." Design accordingly: *the data leaves via SQL; the code leaves only by being rewritten.*
2. **DB2 for i is the door.** Almost every modernization benefit (real-time, analytics, reporting, events) is achievable through the database without touching RPG. Open the data before you discuss the code.
3. **The batch window is a business constraint, not a technical one.** The core posts at EOD; no amount of API wrapping changes that. Real-time *enquiry* (ledger cache) and real-time *posting* are different problems — modernize the first, re-engineer the second only where the business case demands it.
4. **Wrap before you replace.** The OCBC/UOB pattern (digital layers in front of SIBS-class cores) is the verified industry approach; big-bang core replacement is the exception, not the rule (see [banking/standard_chartered_guide.md](banking/standard_chartered_guide.md) for the rare Atlas-style exception and its scope).
5. **The platform is a current product — treat it as one.** IBM i on POWER10/PowerVS, IBM i 7.6, OSS, and REST tooling mean "modernize on-platform" is a legitimate strategy, not a holding action. The vendor-consolidation wave (UOB buying Silverlake) shows the market's own answer to platform dependence: buy the roadmap.
6. **Skills, not software, are the scarcest resource.** The plan above deliberately minimizes the amount of RPG change required *because* RPG expertise is retiring. Every step that keeps the core code untouched is a step that can actually be executed.

---

### 9.4 The Target Architecture Sketch

The end state of Merlion's modernization path, in one diagram and one table:

```
 Channels (mobile, internet, partners)
        │ REST/JSON                     │ events (CDC)
        ▼                               ▼
┌───────────────────────┐   ┌──────────────────────────┐
│  API / Integration    │   │  Event / Data platform   │
│  tier (Node/Python    │   │  (ledger cache, fraud,   │
│  on the i + gateway)  │   │  analytics, reporting)   │
└──────────┬────────────┘   └────────────┬─────────────┘
           │ SQL / stored procs          │ journal CDC
           ▼                             ▲
┌───────────────────────────────────────────────────────┐
│  IBM i LPAR — SIBS core (RPG/COBOL/CL, DB2 for i)     │
│  batch engine: EOD posting, accrual, statements        │
│  PowerVM → on-prem POWER10 or PowerVS (IBM Cloud)      │
└───────────────────────────────────────────────────────┘
```

| Layer | What it is | What stays / moves |
|---|---|---|
| **Channels** | Mobile, internet banking, partner APIs | Modern estate — unchanged by the core's platform ✅ |
| **API tier** | Gateway + Node/Python services on the i or beside it; IWS-wrapped core programs | New — built on the i's modern stack (§7.5) ✅ |
| **Event/data platform** | Kafka-class events from DB2 for i journals; ledger cache; warehouse; regulatory reporting | New — fed by CDC; the "real-time balances" answer ✅ |
| **IBM i core** | SIBS on DB2 for i; RPG/COBOL/CL; batch EOD | **Stays** — the system of record; modernized via SQL/API/OSS/HA, re-platformed to PowerVS if the cloud strategy demands ✅ |
| **File transfer** | Remaining batch interfaces with counterparties | Shrinks but persists (see [technology/axway_transfer_cft_guide.md](technology/axway_transfer_cft_guide.md)) ✅ |

The design principle in one line: **everything that must be real-time lives outside the core and is fed by it; everything that is batch by nature stays inside it; and the two are connected by SQL, CDC, and APIs — not by screen scraping and overnight dumps** — the same topology the OCBC guide's §2.4 describes for its own SIBS estate ✅.

---

## 10. The Summary: The Platform That Refuses to Die

One page, for the architect who needs the whole argument:

**What it is.** The IBM AS/400 (announced June 1988, codename Silverlake, successor to the System/38 and System/36) is a midrange business-computing platform: hardware, operating system (OS/400 → i5/OS → IBM i), integrated database (DB2 for i), and languages (RPG IV, COBOL, CL, SQL) sold and run as one integrated product ✅.

**Why it was built that way.** Three architectural bets, all verified: the **TIMI** (Technology Independent Machine Interface) — applications compile to a virtual instruction set, so hardware can change underneath them; **single-level storage** — one flat 64-bit address space for memory and disk, with the database inside it; and an **object-based** model with object-level security ✅. The bets paid off: the CISC→RISC (1995) and every POWER transition since ran existing applications without recompilation ✅.

**Where it is now.** Still a current IBM product (POWER10, IBM i 7.6, Power Virtual Server on IBM Cloud) ✅ — and still the runtime of serious enterprise workloads: the Silverlake SIBS banking cores at OCBC (vendor-verified) and across UOB's ASEAN franchises (with UOB's 2025 acquisition of the vendor), the AS/400 cores of the Chinese banking estate, and ERP/retail/distribution estates worldwide. Install-base magnitudes ("100,000+ companies") are marketing-sourced and flagged ⚠; the pattern is verified even where the numbers are not.

**Why it refuses to die.** (1) The TIMI kept it current — old code, new silicon. (2) The integrated DB2 for i made the data indestructible and cheap to operate. (3) The ISV ecosystem (Silverlake, SAP, JD Edwards, the ERP families) welded vertical applications to the box. (4) It runs with a tiny team and an unmatched availability record ⚠ (the "never a remote CVE" claim is vendor-sourced). (5) Replacing it means rewriting RPG/COBOL — a business decision no one makes lightly.

**What to do about it.** Not "retire it," and not "keep it frozen." **Open it** (SQL, CDC, APIs), **wrap it** (REST, anti-corruption layers), **modernize on it** (PASE, Node/Python/Git on the i, HA/DR), **re-platform it** (PowerVS) when the data center strategy demands, and **re-engineer selectively** (strangler) only where the business case is real. The worked example in §9 is the template: the i keeps what the i is best at; the modern estate takes everything else; the bridge between them is DB2 for i, not a forklift.

**The final word.** Forty years after Silverlake, the machine that every obituary wrote off is still posting ledgers, still running the region's banks, still refusing to be replaced. The AS/400 is not a legacy system waiting to die — **it is the platform that refuses to die**, and the architect's job is not to fight it, but to integrate it, open it, and let it carry the estate into its next forty years.

---

## 11. Verification and Claims-Status

Every factual claim in this guide is marked inline **✅** (verified in this pass, sources below) or **⚠** (flagged: vendor-sourced, approximate, or unverifiable). Summary:

| Claim | Status | Source / note |
|---|---|---|
| AS/400 announced June 1988 (21 June commonly cited), shipped August 1988 | ✅ | Wikipedia "IBM AS/400" (announced June 1988, released August 1988); IBM history corpus (21 June) |
| Silverlake codename; Frank Soltis led the OS/400 project | ✅ | OS/400 literature (fr.wikipedia "OS/400": "projet OS/400 (nom de code Silverlake)… mené par Franck Soltis"); Soltis's *Inside the AS/400* |
| System/38 announced 24 Oct 1978, delivered July 1980; 48-bit addressing; integrated DB | ✅ | Wikipedia "IBM System/38" |
| System/36 announced 16 May 1983 | ✅ | Wikipedia "IBM System/36" |
| AS/400 merged S/38 + S/36; mostly S/38 architecture; S/36 + S/38 runtime environments in OS/400 | ✅ | ACS heritage article; IT Jungle interview corpus ("three complete runtime environments") |
| TIMI = Technology Independent Machine Interface; high-level instruction set; LIC beneath | ✅ | Wikipedia "IBM AS/400"; IBM System Handbook (GA19-5486) corpus; multisoftsystems architecture write-up |
| Single-level storage: 48-bit on S/38 & CISC AS/400; 64-bit on PowerPC; ~18 exabytes; addresses never reused | ✅ | Wikipedia "IBM AS/400"; IBM i literature |
| Object-based architecture; object authority model | ✅ | IBM i architecture corpus (multisoftsystems, srinsofttech write-ups) |
| OS/400 1988; i5/OS 2004; IBM i 2008 ("i" = integrated); CPF/System-38 heritage | ✅ | Wikipedia "IBM i" ("renamed to i5/OS in 2004, before being renamed a second time to IBM i in 2008"); SAP documentation corpus (V5R4 rename note); Fortra timeline; IBM 2004 press branding ("IBM i5/OS V5R3 — the next generation of OS/400") |
| CISC 48-bit era 1988; RISC transition June 1995, AS/400e, 64-bit PowerPC AS; LIC rewritten, apps save/restored | ✅ | Wikipedia "IBM AS/400"; AS/400 System Handbook (GA19-5486-20): "first AS/400e models based on the 64-bit RISC PowerPC AS processors were announced in June 1995"; ctorst.net HAR2009 slides (IMPI 48-bit CISC → PowerPC 64-bit RISC, 1995, save/restore) |
| ILE introduced with OS/400 V2R3 (1993); RPG IV with V3R1 (1994) | ✅ | O'Reilly "Moving to Integrated Language Environment for RPG IV" (ILE with V2R3, RPG IV in V3R1); IBM RPG docs |
| DB2 for i lineage (DB2/400 → DB2 UDB for iSeries → DB2 for i); integrated, free with OS | ✅ | IBM/SAP documentation corpus; thefillmoregroup/iSeries DB2 commentary ("every iSeries lpar has DB2 by default") |
| PASE = Portable Application Solutions Environment; AIX user-mode binary compatibility; not emulation; V4R2-era introduction | ✅/⚠ | Wikipedia "IBM i" (binary compatibility, 32/64-bit AIX ABIs); IBM "OS/400 PASE" documentation; introduction date (1997, V4R2) is approximate ⚠ |
| IBM i OSS: yum/RPM, Node.js, Python, Git, PHP, nginx; 5733-OPS/5770-OSS | ✅ | IBM "IBM i Open Source Resources" (ibm.github.io/ibmi-oss-resources); ibmi-oss-docs; MC Press Node-on-i corpus |
| Power Systems merger (2008); POWER4 2001 → POWER10 2020; PowerVS on IBM Cloud | ✅ | Fortra/Integrative Systems timelines; IBM Power Systems material (PowerVS slide deck) |
| OCBC runs Silverlake SIBS on IBM AS/400 (Power Systems) | ✅ | Silverlake Axis case study ("OCBC: A Truly World-Class Core Banking System"), excerpted from IDC case study Sept 2013 — see [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2.1 |
| UOB runs Silverlake-family SIBS; UOB acquired Silverlake Axis in 2025 | ✅/⚠ | Consensus-level (KGI initiation report July 2020, investor coverage); acquisition ✅ — see [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2.1; per-entity scope ⚠ |
| Install base "100,000+ companies"; 150k–300k systems estimates | ⚠ | Fortra/IBM-ecosystem marketing figures; IT Jungle community estimates — flag as vendor-sourced, directionally credible |
| "IBM i has never had a remote CVE" | ⚠ | IBM marketing claim, widely repeated — treat as vendor claim; not independently audited in this pass |
| IBM i 7.1 (2010) → 7.6 (2025) release cadence | ✅ | IBM i documentation (ibm.com/docs/en/i — 7.6.0 current) |
| RPG is the majority language of the estate; 5250 screens; ACS/TN5250 access | ✅ | IBM/industry corpus (itmanagement101, IBM docs) — the 5250/ACS access pattern is routine |
| Modernization tooling class (LANSA, ARCAD, Fresche, Profound, Fortra/HelpSystems, Rocket; HA via PowerHA/Db2 Mirror/iCluster/MIMIX) | ⚠ | Class-level verified; specific vendor claims not audited in this pass |

**Method note.** This guide was researched with a bounded web pass (~12 searches + primary-page snippets; no paywalled access). Facts marked ✅ trace to the sources above; facts marked ⚠ are flagged at the point of use. Where the sibling guides carry deeper evidence (OCBC/UOB vendor case studies), the cross-reference is given rather than re-argued. Fictional example in §9 is explicitly fictional but shaped on the verified OCBC/UOB pattern.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **AS/400** | IBM's midrange computer family, announced June 1988 (codename Silverlake); the platform whose architecture (TIMI, single-level storage, object model) survived as IBM i on Power Systems. The name persists as the industry's habitual term for the whole platform line ✅ |
| **IBM i** | The current (2008–) name of the operating system; "i" = integrated; the OS of IBM Power Systems running the AS/400 heritage architecture ✅ |
| **OS/400** | The original operating system of the AS/400 (1988–2004); object-based, DB-integrated, proprietary ✅ |
| **i5/OS** | The OS name 2004–2008, introduced with the POWER5-based eServer i5 ✅ |
| **System/38** | The AS/400's architectural ancestor (announced Oct 1978): single-level storage, 48-bit addressing, integrated relational DB, CPF OS ✅ |
| **System/36** | The AS/400's market ancestor (announced May 1983): small-business workhorse, RPG II, SSP OS; absorbed into the AS/400 with a compatibility environment ✅ |
| **Silverlake** | The 1988 project codename for the AS/400/OS-400 program, led by Frank Soltis. Distinct from Silverlake Axis, the banking-software vendor (§6) ✅ |
| **TIMI** | Technology Independent Machine Interface: the AS/400's high-level, architecture-neutral instruction set; applications compile to TIMI, the Licensed Internal Code implements it on real silicon ✅ |
| **Technology Independent Machine Interface** | The full name of TIMI (above) ✅ |
| **Single-level storage** | One flat virtual address space (48-bit on S/38/CISC, 64-bit on RISC — ~18 exabytes) spanning memory and disk; permanent, never-reused object addresses ✅ |
| **Object-based** | The AS/400 model in which everything (programs, files, user profiles, queues) is a typed object with attributes and OS-enforced object-level authority ✅ |
| **CISC** | The original AS/400 processor generation (1988–1995): proprietary 48-bit CISC chips hidden behind TIMI ✅ |
| **RISC** | The 1995+ processor generation: 64-bit PowerPC AS RISC; the transition TIMI was built for ✅ |
| **POWER** | IBM's RISC processor family (POWER4 2001 → POWER10 2020) that has powered the platform since the iSeries era ✅ |
| **AS/400e** | The 1995 64-bit RISC branding of the AS/400 line ✅ |
| **RPG** | Report Program Generator: IBM's business-data-processing language, the AS/400's signature language, from punched cards (1959) to ILE ✅ |
| **RPG IV** | The modern RPG (OS/400 V3R1, 1994) in the ILE framework; free-format since IBM i 7.1 ✅ |
| **COBOL** | The AS/400's second business language (COBOL/400, ILE COBOL); mixed with RPG under ILE ✅ |
| **CL** | Control Language: the AS/400's command language and compiled batch/glue language; the estate's operating rhythm ✅ |
| **SQL** | The platform's modern access path (SQL/400 → DB2 for i SQL): embedded SQL, ODBC/JDBC, stored procedures, JSON ✅ |
| **DB2 for i** | The integrated relational database of the platform (DB2/400 → DB2 UDB for iSeries → DB2 for i); an OS component, free and always on ✅ |
| **ISV** | Independent Software Vendor — the ecosystem (Silverlake Axis, SAP, JD Edwards, the ERP families) whose vertical applications made the platform a volume business ✅ |
| **PASE** | Portable Application Solutions Environment: the integrated AIX runtime providing binary compatibility for user-mode AIX executables on IBM i ✅ |
| **Open source** | The open-source software (Node.js, Python, PHP, Git, nginx, and more) available on IBM i via yum/RPM ✅ |
| **IBM i OSS** | IBM's open-source offering for IBM i (5733-OPS → 5770-OSS/OSS1); the package-managed modern runtime tier on the platform ✅ |
| **Modernization** | The menu of strategies for AS/400 estates: open (SQL/CDC), wrap (REST/APIs), modernize on-platform (OSS/DevOps/HA), re-platform (PowerVS), re-engineer (strangler) ✅/⚠ |
| **Power Systems** | IBM's merged server line (2008) running AIX, IBM i, and Linux in LPARs; the current home of the AS/400 heritage ✅ |

---

## 13. References

1. Wikipedia — *IBM AS/400* (announcement June 1988 / release August 1988; TIMI; single-level storage; 48-bit CISC vs 64-bit PowerPC addressing). https://en.wikipedia.org/wiki/IBM_AS/400
2. Wikipedia — *IBM i* (OS/400 1988 → i5/OS 2004 → IBM i 2008; "i" = integrated; CPF heritage; PASE binary compatibility). https://en.wikipedia.org/wiki/IBM_i
3. Wikipedia — *IBM System/38* (announced 24 October 1978, delivered July 1980; 48-bit addressing; integrated database). https://en.wikipedia.org/wiki/IBM_System/38
4. Wikipedia — *IBM System/36* (announced 16 May 1983; 17-year lifespan). https://en.wikipedia.org/wiki/IBM_System/36
5. Wikipedia (fr) — *OS/400* (Silverlake codename; Frank Soltis; System/36 application compatibility constraint). https://fr.wikipedia.org/wiki/OS/400
6. ACS Heritage Project, Chapter 37 — *HP and IBM rule the 80s* (AS/400 brought System/36 and System/38 lines together, mostly based on System/38). https://ia.acs.org.au/article/2017/acs-heritage-project--chapter-37.html
7. IBM — *AS/400 System Handbook* (GA19-5486-20, archive.org corpus): "first AS/400e models based on the 64-bit RISC PowerPC AS processors were announced in June 1995"; TIMI definition. https://archive.org/stream/bitsavers_ibmas400GAHandbookV4R5_8299629/GA19-5486-20_AS400_System_Handbook_V4R5_djvu.txt
8. T. Van Looy — *The IBM AS/400: A Technical Introduction* (HAR2009 slides; IMPI 48-bit CISC → PowerPC 64-bit RISC, 1995, save/restore). https://ctors.net/pub/har2009_as400_slides.pdf
9. Multisoft Systems — *A Deep Dive into OS400 Architecture and Core System Concepts* (TIMI as virtual machine layer; LIC beneath; object-based model). https://www.multisoftsystems.com/blog/a-deep-dive-into-os400-architecture-and-core-system-concepts
10. IT Jungle — *The AS/400's Grandfather Talks Past, Present, and Future* (three runtime environments in OS/400: S/38, S/36, native). https://www.itjungle.com/2008/06/23/tfh062308-story01/
11. O'Reilly / IBM redbook corpus — *Moving to Integrated Language Environment for RPG IV* (ILE with OS/400 V2R3; RPG IV with V3R1). https://www.oreilly.com/library/view/moving-to-integrated/0738408131/xhtml/ch01.html
12. SAP documentation — *Updating SAP Dual-Stack Systems on IBM i: IBM Db2 for i* (V5R4 rename note: i5/OS → IBM i). https://help.sap.com/doc/c785a2676c1210149efc13dd4e204223/sum10.28/en-US/sum10_abjv_ibmi_db4.pdf
13. Fortra (power.fortra.com) — *IBM i: History and Timeline* (iSeries 2000, System i 2006, IBM i 2008; 100k+ companies figure ⚠). https://power.fortra.com/blog/ibm-i-history-and-timeline
14. IBM — *IBM i Open Source Resources* (yum/RPM; Node.js, Python, Git, PHP). https://ibm.github.io/ibmi-oss-resources/
15. IBM — *OS/400 PASE* documentation (AIX application porting environment). https://public.dhe.ibm.com/systems/power/docs/systemi/v5r3/en_US/rzalf.pdf
16. IBM — *IBM i documentation* (current releases through 7.6.0). https://www.ibm.com/docs/en/i
17. Silverlake Axis — *OCBC: A Truly World-Class Core Banking System* (case study, excerpted from IDC, September 2013): "Silverlake Axis Integrated Banking Solution (SIBS) on an IBM AS400 (Power Systems) Platform" — via [banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md) §2.1.
18. KGI (July 2020 initiation report on Silverlake Axis) — UOB among flagship SIBS clients — via [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md) §2.1.
19. Integrative Systems — *IBM i (AS/400) History* (Power Systems merger 2008; IBM i 6.1). https://www.integrativesystems.com/ibmi-as400-history/
20. Seasoft — *IBM i: A History in Numbers* (rebranding timeline; ⚠ numbers flagged). https://seasoft.com/blog/ibm-i/ibm-i-a-history-in-numbers/

---

*End of guide. Series context: the AS/400 platform deep-dive sits alongside the banking-core guides ([banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md), [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md), [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md), [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md)) that document the workloads running on it, and the integration guides ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md), [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md), [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [event_stream_processing_guide.md](event_stream_processing_guide.md)) that document how the modern estate talks to it.*
