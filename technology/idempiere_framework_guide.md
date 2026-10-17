# iDempiere: The Dictionary Is the Application — One Entry, Three Renderings
**The 1999 Compiere Origin, the September 2006 ADempiere Fork, the 2008–2011 Modular Turn and the 2011 iDempiere Name, the Java/OSGi/Jetty/ZK/PostgreSQL Stack, the Application Dictionary in Depth (the Six-Step Window Workflow, the Create/Complete Table and Synchronize Column Processes, the Window/Tab/Field Generation), the 2Pack Configuration Packages, the Plug-in and Extension-Point Model (Callouts vs Event Handlers vs Model Validators), the Toolchain and the Deployment Shape, the Structural Comparison with Compiere/ADempiere and the Sibling Odoo, the ERP-in-Banking Context, Where the Dictionary Metaphor Breaks, the Failure Modes, a Cymbal Bank Worked Example, the Anti-Patterns, and a Claims Audit and Verification Ledger — Verified Against the Project's Own Sources, Its Documentation Set, GitHub, and Wikipedia**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Research / Framework Deep-Dive — iDempiere: Overview (the Decoder, the Thesis, the Boundary, the Java Counterpart, Why This Guide Exists), History (Compiere 1999, the ADempiere Fork 2006, the Modular Turn, the iDempiere Name 2011, the Version Trail, the Governance and the Licence), Architecture (Tiers, the Renderer, the Databases, Multi-Tenancy, Where OSGi Sits, the Repository Layout, the Request Lifecycle), the Application Dictionary in Depth (the Six-Step Workflow, Create/Complete Table, Synchronize Column, Create Window/Tab/Field, the Thesis Test), the Extension Model (2Packs, Plug-Ins, Callouts vs Event Handlers vs Model Validators, the GardenWorld Reference Instance), the Developer Toolchain, Deployment and the Operating Shape, the Comparison (the Parent Line, the Sibling Odoo, the Wider Field), the ERP-in-Banking Context, Where the Dictionary Metaphor Breaks, the Failure Modes, the Cymbal Bank Worked Example, the Anti-Patterns, the Claims Audit, What Could Not Be Verified, Glossary, Cross-References, Closing Summary and Verification Ledger
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** www.idempiere.org (home, Features, Releases archive, About, Source Code, Test Sites — read 2026-10-17); docs.idempiere.org (the Docusaurus developer documentation set — Creating a New Window, What is a 2Pack, the Pack In/Pack Out guide, Developing Plug-ins Without Affecting the Core, Distributing and Installing Plug-ins, the Callout/Event-Handler/Model-Validator/Model-Factory guides, the Maven/Tycho build guide, Install Development Prerequisites, Setting up Eclipse, Download the Code, the v13 technical migration notes, and the Upgrade Guide via a Wayback capture dated 2026-06-10); github.com/idempiere/idempiere (repository root listing and README, read 2026-10-17); Wikipedia (the iDempiere, ADempiere and Compiere articles); and the project infrastructure hosts idempiere.atlassian.net (JIRA), jenkins.idempiere.org (Jenkins), mattermost.idempiere.org and the project wiki wiki.idempiere.org (the latter blocked by a Cloudflare challenge this pass)
> **Last Updated:** October 2026
> **Companion guides (technology/, same folder):** [Odoo Framework](odoo_framework_guide.md), [OSGi](osgi_guide.md), [Asia Fusion Technology](asia_fusion_technology_guide.md), [Jetty](jetty_guide.md), [RxJava](rxjava_guide.md), [Low-Latency Java Programming](low_latency_java_programming_guide.md), [Talend](talend_guide.md), [jBASE / UniVerse](jbase_universe_guide.md)
> **Companion guides (banking/, management/, personal/):** [T24 Programming](../banking/t24_programming_guide.md), [Insurance — Open Source vs Commercial](../banking/insurance_open_source_commercial_guide.md), [Apache Fineract](../banking/apache_fineract_guide.md), [Business Case Development](../management/business_case_development_guide.md), [Jack Liu Profile](../personal/jack_liu_profile.md)

---
### How to Read This Guide
This is the **dedicated deep-dive on iDempiere** — the Java, OSGi-based, metadata-driven ERP/CRM/SCM suite — in the `technology/` framework series. It is the second half of a deliberate pair. The sibling guide [odoo_framework_guide.md](odoo_framework_guide.md) covers the **Python + PostgreSQL** open-source ERP and says so in its own §3.5: *"The difference from iDempiere (a Java/ADempiere descendant) is the language and the weight: Python + PostgreSQL instead of Java + PostgreSQL/Oracle."* That sentence is a promise this guide keeps. Where the sibling analyses the ORM-and-apps world of Odoo, this guide analyses the **dictionary-and-plug-ins** world of iDempiere — and the two are read together, not merged.

**The verification discipline.** Every load-bearing fact in this guide — years, version numbers, licence identifiers, dates, module names, process names, command names — was checked in this research pass (October 2026) against the project's own website, its `docs.idempiere.org` documentation set, the GitHub repository's file listing, or Wikipedia. Facts confirmed this pass are marked **✅**; facts that rest on a single source, on secondary (encyclopaedic) material, on sources that disagree with one another, or on knowledge that could not be re-verified this pass are marked **⚠** with an honest explanation. **Nothing is invented.** Where the record is ambiguous — the release date of version 13, the Java and PostgreSQL version floors — this guide prints the disagreement rather than smoothing it.

**A note on tool limitations during this pass.** Three kinds of source were not reachable and are flagged rather than filled in. First, the project's own **wiki** (`wiki.idempiere.org`) — including its *Plugin* page and its Application Dictionary pages — sits behind a Cloudflare interstitial ("Just a moment…") and could not be read; Wikipedia cites several of those wiki pages, and this guide therefore treats the wiki as *known to exist* but *not read this pass*. Second, several documentation pages referenced from the fetched set (the v9 and v14 new-feature pages, the "Running Plug-ins Locally" page) were referenced by the fetched pages but not themselves in the cache, so their claims are attributed to the page that cites them. Third, the site's own version-history pages disagree on several dates, and this guide records each value rather than choosing one.

**No figures, no benchmarks, no rankings.** This guide is a **structural** comparison throughout. It prints no adoption number, no market share, no performance benchmark, and no ranking of any vendor or product against another. Where a source offers such a figure (the early SourceForge popularity of Compiere, the listicle placements of iDempiere), the guide states the *kind* of claim and declines to reproduce the number as evidence of anything. The community's own conference attendee counts are reported as *attendance records*, never converted into popularity claims.

**Nothing here is procurement, licensing, or investment advice.**
### Quick Facts (Summary Card)

| Item | Value |
|---|---|
| What it is | Open-source ERP + CRM + SCM suite; the project calls itself a "powerful, Tier II, open-source ERP/CRM/SCM system supported by a skilful community" — **✅ Verified** (idempiere.org introduction page, read 2026-10-17) |
| Also known as | "Community Powered Enterprise", or "OSGi + ADempiere" — **✅ Verified** (Wikipedia *iDempiere*; the GitHub repository tagline) |
| Lineage | Compiere (1999) → ADempiere (fork, September 2006) → iDempiere (2011) — **✅ Verified** (Wikipedia *iDempiere*, *ADempiere*, *Compiere*) |
| Founded / named | Project states "Founded in 2011"; Wikipedia records the 2011 name decision after ADempiere community disagreements — **✅ both** (project site + Wikipedia) |
| First official release | 1.0.a "Halloween Edition", **31 October 2012** (alpha) — **✅ Verified** (Wikipedia infobox; idempiere.org Releases archive listing "v1.0a Installers (Halloween Release) 2012-10-31") |
| Current stable | **13 "Orion"** — the project's Releases page recommends "Daily 13 Orion (released 2026-03-06, stable and maintained)"; Wikipedia's table says 9 March 2026; the project's blog post is dated 2026-03-12 — **⚠ three values recorded, not smoothed** (§2.5) |
| Next | **14** — a "Dev Build" from `master` on the project's test sites — **✅ Verified** (idempiere.org introduction page; Wikipedia lists v14 as the preview release) |
| Licence | **GPL-2.0-or-later** — project's own words: "The built iDempiere source is licensed under the GNU General Public License version 2 (or later)" — **✅ Verified** (idempiere.org/source-code); GitHub's classifier reports the repository's `LICENSE.md` as "Other"/NOASSERTION — **✅ noted** |
| Licence model | **No open-core split, no enterprise edition, no per-user licence** — **✅ Verified** by absence: the project publishes one source tree and one licence (contrast the sibling [odoo_framework_guide.md](odoo_framework_guide.md) §1.3) |
| Stack | "Java, ZK, OSGi, Oracle, PostgreSQL, Maven" — **✅ Verified** (idempiere.org home page); Java server on a Jetty container with a ZK web GUI, OSGi for modularisation — **✅ Verified** (Wikipedia *Platform*) |
| Platforms | Windows, Linux, macOS, Solaris — **✅ Verified** (Wikipedia infobox) |
| Databases | PostgreSQL and Oracle — **✅ Verified**; the version floors **differ between the project's own pages by vintage** — **⚠** (§3.3) |
| Extensibility | Application Dictionary (metadata), 2Packs (configuration packages), and OSGi plug-ins — **✅ Verified** (Wikipedia *Key features*; docs.idempiere.org) |
| Governance | Community-led, "no single person or company ruling the project"; council-of-contributors model inherited from ADempiere — **✅ Verified** (Wikipedia *Community-powered*; Wikipedia *ADempiere*, Project structure) |
| Community channels | JIRA (idempiere.atlassian.net), Jenkins (jenkins.idempiere.org), Mattermost (mattermost.idempiere.org), a Google Groups forum, docs.idempiere.org, wiki.idempiere.org, a donations page — **✅ Verified** (GitHub README, read 2026-10-17) |
| Repository | github.com/idempiere/idempiere, default branch `master`, **16,318 commits**, **666 stars**, **482 forks**, **70 open issues**, created on GitHub **2018-02-26**, last push **2026-10-14** — **✅ repository statistics, not a measure of adoption** (GitHub, read 2026-10-17) |
| Reference instance | The **GardenWorld** demonstration client on the project's public test sites — **✅ Verified** (docs.idempiere.org worked examples; idempiere.org introduction page) |
| Series context | The author's ERP-domain career background (Odoo, iDempiere) — see [../personal/jack_liu_profile.md](../personal/jack_liu_profile.md) — **✅ as career context** |

---
### Table of Contents
1. [The Overview](#1-the-overview) — 1.1 the decoder · 1.2 the thesis · 1.3 the boundary declared by name · 1.4 the Java counterpart of the pair · 1.5 why this guide exists
2. [The History](#2-the-history) — 2.1 Compiere 1999 · 2.2 the 2006 fork · 2.3 the modular turn · 2.4 the iDempiere name · 2.5 the version trail · 2.6 the dates that disagree · 2.7 governance and the licence · 2.8 recognition and gatherings
3. [The Architecture](#3-the-architecture) — 3.1 the tiers · 3.2 the ZK renderer · 3.3 the databases and their version floors · 3.4 the multi-tenant model · 3.5 where OSGi sits · 3.6 the repository layout · 3.7 the request lifecycle · 3.8 the architecture table
4. [The Application Dictionary in Depth](#4-the-application-dictionary-in-depth) — 4.1 what the dictionary is · 4.2–4.5 the six documented steps · 4.6 the thesis test · 4.7 the best practices
5. [The Extension Model](#5-the-extension-model) — 5.1–5.3 2Packs, Pack Out/In and Automatic External Pack In · 5.4 OSGi plug-ins · 5.5 distribution (p2, JAR, Gogo, Felix, registry) · 5.6 the extension points: callouts vs event handlers vs model validators · 5.7 how a customisation is built and moved · 5.8 the GardenWorld reference instance
6. [The Developer Toolchain](#6-the-developer-toolchain) — 6.1 the IDE and build path · 6.2 source access and branching · 6.3 the prerequisites, with versions and dates · 6.4 testing and CI
7. [The Deployment and the Operating Shape](#7-the-deployment-and-the-operating-shape) — 7.1 the installation paths · 7.2 the runtime and the three ports · 7.3 the p2 update mechanism · 7.4 multi-tenant and the cluster picture
8. [The Comparison](#8-the-comparison) — 8.1 against its own parent line · 8.2 against the sibling Odoo · 8.3 against the wider open-source ERP field
9. [The ERP-in-Banking Context](#9-the-erp-in-banking-context) — 9.1 back-office, not the ledger · 9.2 the banking fit table · 9.3 cross-references to the bank series
10. [Where the Dictionary Metaphor Breaks](#10-where-the-dictionary-metaphor-breaks) — 10.1 the callout scope limit · 10.2 what it means for upgrades · 10.3 what it means for testing · 10.4 why the illusion fails exactly there
11. [The Failure Modes](#11-the-failure-modes) — 11.1 upgrade pain · 11.2 extensions that bypass the dictionary · 11.3 a community project's support reality · 11.4 the assumption that configuration is free · 11.5 the failure-mode table
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example) — 12.1 the scenario · 12.2 the selection · 12.3 the deployment design · 12.4 the extension design · 12.5 the decision the iDempiere-ness changed · 12.6 the lessons
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [The Glossary, the Cross-References, the Closing Summary, and a Verification Ledger](#16-the-glossary-the-cross-references-the-closing-summary-and-a-verification-ledger) — 16.1 the glossary · 16.2 the cross-references · 16.3 the closing summary · 16.4 the verification ledger

---
## 1. The Overview
### 1.1 The Decoder — the Terms This Guide Uses
iDempiere is dense with its own vocabulary, and almost every term means something slightly narrower than its generic reading. This is the decoder; each term is defined once here and used consistently after.

| Term | What it means in iDempiere | Source |
|---|---|---|
| **iDempiere** | The product: an open-source ERP/CRM/SCM suite written in Java, built on the OSGi module system, descended from ADempiere and Compiere. The project's own one-line definition is "a powerful, Tier II, open-source ERP/CRM/SCM system supported by a skilful community" — **✅** | idempiere.org introduction page; Wikipedia |
| **Compiere** | The 1999 ancestor by Jorg Janke; the source of the data-dictionary idea and of the `org.compiere.*` Java package names that survive in iDempiere's module names — **✅** | Wikipedia *Compiere*; GitHub module listing |
| **ADempiere** | The September 2006 fork of Compiere; it inherited the data dictionary and extended it, and it is iDempiere's immediate parent — **✅** | Wikipedia *ADempiere* |
| **Application Dictionary** | The metadata layer that describes the application to itself: tables, columns, windows, tabs, fields, references, validations, menus, roles. Wikipedia calls it an "Active Data Dictionary" — **✅** | Wikipedia *Key features* |
| **Model-driven architecture** | The consequence of the dictionary: entities, validation rules, windows and formats are declared as metadata, not written as Java, so "customisation can be done without new coding" — **✅** | Wikipedia *ADempiere*, Architecture |
| **OSGi bundle** | The unit of modularity: a JAR with an OSGi manifest that the runtime can install, start and stop independently. In iDempiere the core itself is a set of bundles — **✅** | Wikipedia *Platform*; GitHub module listing |
| **Plug-in** | A bundle that adds or changes ERP/CRM/SCM functionality; in the documentation a plug-in is written in Eclipse as a plug-in project and built against the Equinox target platform — **✅** | Wikipedia *Plug-ins*; docs.idempiere.org |
| **Tenant / client** | A tenancy unit carried on every dictionary row as `AD_Client_ID`. "Client" is the iDempiere term; "tenant" is the same thing described from the operations side — **✅** | docs.idempiere.org (Creating a New Window); project documentation-page titles |
| **Extension point** | A named hook the documentation lists as a place where plug-in code may attach: callouts, events, model factories, validators, processes, factories, forms, themes, and the newer extension registry — **✅** | docs.idempiere.org documentation page list |
| **2Pack** | A `.zip` package of dictionary metadata and configuration used to move customisations between environments — the project describes it as a "configuration snapshot" — **✅** | docs.idempiere.org, *What is a 2Pack* |
| **GardenWorld** | The demonstration client shipped with iDempiere, and the client the developer documentation's worked examples use — **✅** | docs.idempiere.org; idempiere.org |

Two names from the lineage survive visibly in the code. The `org.adempiere.*` package prefix persists in module names such as `org.adempiere.base` and `org.adempiere.pipo`, and the `org.compiere.*` prefix persists in the database-provider bundles (`org.compiere.db.postgresql.provider`, `org.compiere.db.oracle.provider`) — **✅ verified from the GitHub repository root listing**. Reading the repository is therefore a small lesson in its own history: the newest names (the `org.idempiere.*` bundles) sit beside the inherited ones.
### 1.2 The Thesis — One Dictionary Entry, Three Renderings
**The spine of this guide is a single claim: in iDempiere, the model, the screen and the database table are three renderings of one dictionary entry.**

The claim is not decorative; it is a description of a mechanism the documentation exposes directly. When a developer defines a table in the Application Dictionary and runs the documented processes, one set of dictionary rows produces *both* a physical database table *and* a window/tab/field set that produces a screen *and* the menu and role-access rows that decide who may see it (§4). The three are not kept in sync by a build step or a generator run outside the system — they are three views of the same rows, rendered at different times by different layers.

The claim is worth testing rather than asserting, because it is the reason iDempiere is interesting to a developer at all. Wikipedia states the design intent plainly: iDempiere "extends the concept of a data dictionary to an 'Active Data Dictionary' that lets it manage entities, validation rules, windows, tables, formats, and other customizations of the application without new Java code. So iDempiere can be seen not only as an ERP but also as a platform to build database driven applications" — **✅ verified** (Wikipedia *iDempiere*, Application dictionary and customization). The project's own features page makes the same promise operationally: "Thanks to its flexibility and Dictionary, iDempiere is widely used as a development platform. You can create new windows with all the functionality within minutes" — **✅ verified** (idempiere.org Features).

This guide tests that thesis (§4.6, §10) and says plainly where it breaks down. The short version, stated up front so the reader can hold it while reading: **the thesis holds exactly as far as the dictionary can describe the behaviour, and it fails the moment behaviour cannot be expressed as metadata.** The single best-documented failure is the callout scope limit (§5.6, §10.1): a piece of dictionary-attached behaviour that the documentation says executes *only* when data arrives through the UI or the CSV importer, and *not* when it arrives through the REST API, a background process, a direct database operation, or Java code calling `PO.save()`. That is a metadata-driven behaviour with a source-of-entry condition the metadata cannot express — and it is where the illusion of "the dictionary is the whole application" stops being true.

iDempiere also arrives with a second, orthogonal layer: the **OSGi plug-in model**. The dictionary renders the model; the plug-ins place behaviour around it. The boundary between the two — what goes in metadata, what goes in a bundle — is the central design decision in any iDempiere customisation, and it is the subject of §5, §10 and §11.
### 1.3 The Boundary Declared by Name
Two companion guides own adjacent territory **in full**, and this guide deliberately does not duplicate them. Stating the boundary by name is the house rule, so:

- **[odoo_framework_guide.md](odoo_framework_guide.md)** owns the **sibling ERP**: its history, editions, apps and release cadence (§1–§6 there), its **§7 comparison** including the Tryton head-to-head and the open-source-vs-commercial frame, its **§8 ERP-in-banking framing**, and its **§9 Cymbal Bank back-office worked example**. This guide **cites** those sections and does not re-derive the genre, does not redo the Tryton comparison, and does not re-argue the open-source-vs-commercial case. The one thing this guide *does* add is the structural contrast that belongs to the pair: Odoo is open-core (§1.3 there); iDempiere is not (§2.7 here).

- **[osgi_guide.md](osgi_guide.md)** owns the **OSGi runtime in full**: architecture, bundle, module and class-loading, lifecycle, the service layer, declarative services, the compendium, release history, and governance. Its **§11 "Platforms That Embed OSGi"** carries an adoption table covering Eclipse, AEM, Liferay, Sling, Karaf, Talend Runtime, Nexus and Jetty. This guide may describe **only how iDempiere uses OSGi** — the bundles, the console, the web console, the Equinox runtime it embeds — and treats the runtime itself as [osgi_guide.md](osgi_guide.md)'s subject.

*One honest observation of my own, with its caveat stated:* **iDempiere does not appear in [osgi_guide.md](osgi_guide.md)'s §11 platform table** — a check I ran this pass, not a claim of that guide's. The table there lists Eclipse, AEM, Liferay, Sling, Karaf, Talend Runtime, Nexus and Jetty, and iDempiere — which is an OSGi-based application as thoroughly as any of them — is absent. That is a gap in the sibling guide, recorded here rather than fixed there, because this guide does not edit its companions.

Also cross-referenced and not re-explained: [rxjava_guide.md](rxjava_guide.md), [low_latency_java_programming_guide.md](low_latency_java_programming_guide.md), [jetty_guide.md](jetty_guide.md), [talend_guide.md](talend_guide.md) and [jbase_universe_guide.md](jbase_universe_guide.md) — the Java and database material generally. Where this guide names Jetty as iDempiere's container, the Jetty guide owns the server; where it names Java, the low-latency and RxJava guides own the language and the reactive idiom.

The **career context** is cited, not re-told. The author's career profile records his ERP background as *"ERP frameworks (Odoo, iDempiere) — metadata-driven development, DSLs, application dictionaries"* — **✅ verified** ([../personal/jack_liu_profile.md](../personal/jack_liu_profile.md)); the Asia Fusion guide ([asia_fusion_technology_guide.md](asia_fusion_technology_guide.md) §4.4–4.5) carries the employer-domain attribution, and following the sibling Odoo guide's convention, that employer-domain attribution is **flagged as such** wherever it appears.
### 1.4 The Java Counterpart of the Pair
This guide exists because iDempiere is the **one remaining unbuilt member of a pair**. The sibling Odoo guide, at 715 lines, is the Python + PostgreSQL member; this is the Java + PostgreSQL/Oracle member, and the two together are the repository's account of the metadata-driven open-source ERP. The pairing is not a filing convenience — it is a controlled comparison. Both projects:

- descend from a small business-management application and generalised into platforms; both are open source and metadata-driven, with the dictionary/ORM as their centre of gravity; both ship an ERP and are used as development platforms; and both support PostgreSQL as a first-class database.

And they differ in ways that are structural rather than cosmetic. Odoo's metadata is **code-adjacent** — Python model classes and XML view definitions, versioned in a module (`__manifest__.py`), interpreted by the ORM at load time. iDempiere's metadata is **data** — rows in `AD_*` tables in the same database the business data lives in, editable at runtime through windows designed for the purpose. Odoo is open-core with a commercial company at the centre; iDempiere has no such split and no company at the centre. Odoo's web client is a bespoke TypeScript library (OWL); iDempiere's is a ZK application. The pairing makes each easier to see: read the two §4s side by side and the difference between "metadata as code" and "metadata as data" becomes concrete.
### 1.5 Why This Guide Exists — the Repository Gap and the Career Context
**The repository gap was real and countable.** Before this guide, iDempiere appeared in **exactly three files** in the repository, all incidentally — **✅ verified this pass** by a full-text search for the string across the repository's Markdown files:

1. **[../personal/jack_liu_profile.md](../personal/jack_liu_profile.md)** — as part of the author's ERP-domain career background ("ERP frameworks (Odoo, iDempiere) — metadata-driven development, DSLs, application dictionaries").
2. **[asia_fusion_technology_guide.md](asia_fusion_technology_guide.md)** — in the employer-domain spread at §4.4–4.5 (flagged as employer-domain attribution).
3. **[odoo_framework_guide.md](odoo_framework_guide.md)** — in a single sentence at §3.5, defining iDempiere only by contrast with Odoo ("a Java/ADempiere descendant").

Three mentions, none of them a treatment. iDempiere had never been given a dedicated guide, and — more to the point — the *promise* made in the sibling's §3.5 had never been kept. This guide keeps it: it gives iDempiere the same architecture-first, verify-everything treatment the sibling gave Odoo, and it does so explicitly as the counterpart, cross-referencing the sibling's §7, §8 and §9 rather than repeating them.

The **career context** is the reason the gap was visible to begin with, and it is cited as context only. The author's ERP background includes both Odoo and iDempiere, described as *"metadata-driven development, DSLs, application dictionaries"* — **✅** ([../personal/jack_liu_profile.md](../personal/jack_liu_profile.md)) — and the Asia Fusion guide carries the **employer-domain attribution** ([asia_fusion_technology_guide.md](asia_fusion_technology_guide.md) §4.4–4.5, **flagged as employer-domain**). Where the author's own experience and a primary source disagree, the primary source wins in this guide; the career context only explains why the questions are asked.

---
## 2. The History
### 2.1 Compiere — the 1999 Origin
**In 1999, Jorg Janke created Compiere, the predecessor of iDempiere. He had previously worked at Oracle in Germany and the USA** — **✅ verified** (Wikipedia *iDempiere*, History). The name comes from the Italian *"kom-pyeh-reh"*, meaning *to accomplish, to complete, to fulfil* — **✅ verified** (Wikipedia *Compiere*).

The Compiere era is where the ideas this guide is about were first assembled. Compiere shipped a **data dictionary** and a set of business processes, and the dictionary was the thing that made it extensible without touching the Java. Two later facts matter for the lineage:

- **2002 onward, Compiere was a leading SourceForge ERP project.** The encyclopaedic record describes it as among the most prominent SourceForge projects for several years and cites 2006-era figures for downloads and partners. **This guide does not print those numbers**: they are a 2006-era project claim carried by a secondary source, and the house rule is that no adoption or scale figure is reproduced. The honest statement: **Compiere was a leading open-source ERP project in its early years — ⚠ secondary source** (Wikipedia *Compiere*). *(The siblings' scale framing is deliberately different in kind: [odoo_framework_guide.md](odoo_framework_guide.md) §2.4 deals with funding figures; this guide prints none.)*
- **2006: Compiere, Inc. raised venture capital from New Enterprise Associates, stating the goal of transforming the open-source project into a commercial open-source business, and the product line split into Professional/Enterprise/Cloud editions alongside a Community edition** — **✅ verified** (Wikipedia *Compiere*). This decision — commercial editions beside a community edition — is the direct trigger of everything that follows, and it is the same strategic move the sibling documents for Odoo in a later decade ([odoo_framework_guide.md](odoo_framework_guide.md) §1.3).

- **24 March 2009: Compiere 3.3**, the last stable release listed by Wikipedia — **✅** — with the infobox listing the licence as *"GPL or Proprietary"* — **✅** (Wikipedia *Compiere*).
- **16 June 2010: Consona acquired Compiere, Inc., terms undisclosed** — **✅ verified** (Wikipedia *Compiere*). For the open-source community around the code, the acquisition closed the question of who controlled the upstream project.
### 2.2 The 2006 Fork — ADempiere
**ADempiere was created in September 2006 as a fork of Compiere, after disagreement between the open-source developer community and the project's corporate sponsor** — **✅ verified** (Wikipedia *ADempiere*, History). The name comes from the Italian verb *adempiere* — **✅** (Wikipedia *ADempiere*). Wikipedia's infobox lists a release date of **12 October 2006** and names **Jorg Janke as original author** — **✅**; the infobox licence is **GPLv2** — **✅** (Wikipedia *ADempiere*).

ADempiere is where the thesis of this guide was born, and the encyclopaedic record says so in exactly the terms the thesis uses. ADempiere **inherited the data dictionary from Compiere, and that architecture extends the data-dictionary concept into the application**, so that "the application's entities, validation rules and screen layout can be controlled from within the application itself — in practice customisation can be done without new coding"; a workflow engine based on **Workflow Management Coalition** and **Object Management Group** standards provides the BPM capability — **✅ verified** (Wikipedia *ADempiere*, Architecture). *That is the conceptual root of "one dictionary entry, three renderings": the dictionary controls the entities (the model), the screen layout (the screen) and the validation rules, all from inside the running application.*

The ADempiere technology profile, for the record — **✅ verified** (Wikipedia *ADempiere*): **Java EE, Apache Tomcat and JBoss**; database support **restricted to PostgreSQL and Oracle** (the constraint iDempiere inherits intact); and a stable release listed as **3.9.4, January 2023**.

**ADempiere's governance** was set in this era and iDempiere inherits its shape: "all community members have a say in the forums; for practical purposes the project is governed by a council of contributors, with a leader nominated from the council acting as overall project manager; the council supports the leader's decisions, accepts contributions, defines the roadmap, reviews and approves specifications, votes on new functionality and approves changes to core" — **✅ verified** (Wikipedia *ADempiere*, Project structure). Note the deliberate mirroring: a single accountable role, embedded in a council that can override — a structure designed to avoid a repeat of the sponsor-versus-community rupture that caused the fork.

**ADempiere today** — **✅ verified this pass** from the repository (github.com/adempiere/adempiere): default branch `develop`, **16,989 commits**, **888 stars**, **796 forks**, licence **GPL-2.0**. The parent line remains an active project, not a historical artefact — a fact that matters for §8.1, where iDempiere is compared to a living sibling rather than a dead ancestor.
### 2.3 The Modular Turn — 2008 to 2011
The turn that made iDempiere *iDempiere* was the move to a modular, OSGi-based architecture, and it happened **inside ADempiere before the name iDempiere existed**:

- **2008 to 2010: inside the ADempiere community, some users worked on a modular ERP design proposal, with OSGi implementations such as Apache Felix and Equinox** — **✅ verified** (Wikipedia *iDempiere*, History; it cites ADempiere wiki pages titled "ADempiere/OSGi Integration" in a revision of 30 July 2008 and "Equinox Integration"). Read that carefully: the modularisation was a *community proposal*, prototyped with two different OSGi frameworks, for roughly two years before it became a product. It was not a vendor roadmap item.
- **2010: "Adempiere 3.6.0 LTS" and the "Branch GlobalQSS Adempiere361" are launched** — **✅ verified** (Wikipedia *iDempiere*, History). The branch name is the one iDempiere is later described as forking from (§2.4), and the LTS label signals that the community was already thinking about maintained lifetimes.

The technology the proposal chose is the technology iDempiere still runs: the **OSGi framework for modularisation**, an application assembled from bundles rather than a monolith. [osgi_guide.md](osgi_guide.md) owns what OSGi *is*; this guide's interest is only that iDempiere's modularity was a community design decision taken two years before the product shipped, which is why plug-ins (§5.4) are a first-class feature rather than a bolted-on extension mechanism.
### 2.4 The iDempiere Name — 2011
**In 2011, due to disagreements in the former ADempiere developer community, the name iDempiere was adopted for the code path with the new OSGi architecture**, so that, in the encyclopaedic record's own framing, "iDempiere can be seen as the next generation of ADempiere or as a fork of the 'Branch GlobalQSS Adempiere361'", and **"the majority of the active developer community started to work on iDempiere"** — **✅ verified** (Wikipedia *iDempiere*, History).

Independently, the project's **own site** states: *"Founded in 2011 iDempiere is a powerful, Tier II, open-source ERP/CRM/SCM system supported by a skilful community. The project focuses on high-quality software, a philosophy of openness and its collaborative community that includes subject matter specialists, integrators, developers and end-users"* — **✅ verified** (www.idempiere.org, read 2026-10-17).

Two records, one year, **cited separately** because they are different kinds of source: the encyclopaedia records the *cause* (disagreement, a name change, a code path), the project records the *self-description* (a founding year, a tier, the participants). Neither is a substitute for the other, and this guide does not merge them.

**The first official release came a year later.** **31 October 2012: iDempiere 1.0.a "Halloween Edition"**, the first official version, an alpha — **✅ verified twice over** (Wikipedia infobox/changelog; the project's own Releases archive listing "v1.0a Installers (Halloween Release) 2012-10-31"). The Halloween date is not a one-off: the 31 October rhythm recurs in the version trail below, and from release 9 onward it settles onto **24 December** — a release cadence that is itself part of the project's character (§2.5).
### 2.5 The Version Trail
The trail below is drawn from the project's own Releases archive and corroborated, where it overlaps, by Wikipedia's version table — **✅ project-primary with Wikipedia corroboration** (idempiere.org/releases; Wikipedia *iDempiere*).

| Version | Name | Date | Notes |
|---|---|---|---|
| 1.0a | Halloween Edition | **2012-10-31** | First official version; alpha — **✅** |
| 1.0b | Devina | **2013-04-24** | Beta — **✅** |
| 1.0c | Devina LTS | **2013-06-27** | The first LTS line — **✅** |
| 2.0 | — | **2013-10-31** | Anniversary-day release — **✅** |
| 2.1 | — | **2014-10-31** | Last 31 October release for four years — **✅** |
| 3.1 | Maitreyi | **2015-10-31** | Tomcat → Jetty (see below) — **✅** |
| 4.1 | — | **2016-10-31** | — **✅** |
| 5.1 | — | **2017-10-31** | ZK 8.0.2.2; Oracle 12c tested — **✅** |
| 6.1 | — | **2018-11-04** | OpenJDK 8 → 10; Buckminster → Maven/Tycho — **✅** |
| 6.2 | — | **project page 2019-01-06 / Wikipedia 2019-01-04** | **Discrepancy recorded — ⚠** |
| 7.1 | — | **2019-11-22** | PostgreSQL/OpenJDK 11; code moved to GitHub — **✅** |
| 8.1 | — | **2020-11-03** | Release candidate — **✅** |
| 8.2 | Phong | **2020-12-20** | — **✅** |
| 9 | Horizon | **2021-12-24** | The 24 December rhythm begins — **✅** |
| 10 | Peace | **2022-12-24** | — **✅** |
| 11 | Synergy | **2023-12-24** | OpenJDK 17 — **✅** |
| 12 | Kudos | **2024-12-24** | — **✅** |
| 13 | Orion | **⚠ see §2.6** | Current stable — **✅ as a release; date disputed** |

**What the platform changes tell you, version by version** — **✅ verified** (Wikipedia *iDempiere*, History — *secondary source, flagged as such*): **1.0** replaced JBoss with **Apache Tomcat** (credited with improving performance and memory footprint), adopted an automatic build system based on **Eclipse Buckminster**, and upgraded **ZK from 3.6 to ZK 6**, which "allowed redesign of the web GUI"; **v3.1 (2015)** replaced Tomcat with **Jetty**; **v5.1 (2017)** moved **ZK to 8.0.2.2** and tested **Oracle 12c**; **v6.1 (2018)** moved **OpenJDK 8 → 10** and replaced Buckminster with **Maven Tycho**; **v6.2 (2019)** moved to **OpenJDK 11** and migrated the code from **Bitbucket/Mercurial to GitHub/Git** about a month before 7.1, with **Mattermost replacing IRC**; **v11 (2023)** moved to **OpenJDK 17**.

The **GitHub migration date is corroborated from the other side**: the repository at github.com/idempiere/idempiere was **created on GitHub on 2018-02-26** — **✅ verified this pass** — which is consistent with the documented "one month before 7.1" migration window, and the repository's last push was **2026-10-14** — **✅**. That last date matters: it is evidence that the project was actively shipping at the time of this pass.
### 2.6 The Dates That Disagree (Flagged)
**Release 13 "Orion" has three conflicting date values, and this guide records all three rather than choosing one** — **⚠ flagged**:

| Source | Value for 13 "Orion" |
|---|---|
| The project's **Releases** page | "Daily **13 Orion** (released **2026-03-06**, stable and maintained)" — **✅ as the project's own words** (idempiere.org/releases, read 2026-10-17) |
| The project's **blog post announcing it** | dated **2026-03-12** — **⚠ as reported by the fact sheet this pass** |
| **Wikipedia's** version table | **9 March 2026** — **✅ as Wikipedia's value** |

There is no way to reconcile these from the sources read this pass, and the honest move is to print the disagreement: the *release* of 13 "Orion" happened in **March 2026** on every account, but the *day* — 6, 9 or 12 — depends entirely on which project artefact you read. This is a small, clean example of the guide's whole method: **where the record is ambiguous, say so and move on.**

Two further date disagreements are recorded in their proper places rather than here: the **6.2** value (project page 2019-01-06 versus Wikipedia 2019-01-04 — §2.5), and the **Java/PostgreSQL version floors** that differ between the project's own pages by vintage (§3.3). And one count disagreement belongs to the overview: Wikipedia's infobox says the interface is available in **16 languages**, while its Overview text says **29 localizations** — **⚠ both recorded** (Wikipedia *iDempiere*; §3.4).
### 2.7 The Governance and the Licence at Source
**The licence — stated exactly, because the identifier matters.** The project's own words are decisive: *"The built iDempiere source is licensed under the GNU General Public License version 2 (or later)"* — **✅ verified** (www.idempiere.org/source-code, read 2026-10-17). So the identifier this guide prints is **GPL-2.0-or-later**, **attributed to the project's source-code page**.

Three corroborating and complicating notes, all recorded:

- **Wikipedia's infobox** says **GPLv2** — **✅** (Wikipedia *iDempiere*). Consistent, but stated without the "or later".
- **The GitHub repository** carries a `LICENSE.md` and a `license.html`, and its README badge reads "license GPL-2.0" — **✅ verified** (GitHub, read 2026-10-17). The **GitHub licence classifier**, however, reports the repository's licence as **"Other"/NOASSERTION** — **✅ verified** — i.e. GitHub does not assert a standard SPDX identifier for it. **The project's own statement is therefore the authority**, and the classifier is a caveat on how the file is machine-read, not a contradiction of the licence.
- The repository's `license.html` exists alongside `LICENSE.md` — **✅ verified from the file listing** — which is why a classifier can fail to settle on one identifier.

**There is no open-core split, no enterprise edition, and no per-user licence for iDempiere.** This is a structural contrast with the sibling, and it is stated here as a *difference in kind*: where [odoo_framework_guide.md](odoo_framework_guide.md) §1.3, §4.1 and §4.4 document an open-core line (Community LGPLv3 versus a proprietary Enterprise subscription, priced per user per month), iDempiere publishes **one source tree under one licence**, with no proprietary layer and no company to sell one. The project's About page frames the intent in values rather than in a business model: a mission — *"To help companies around the world to grow with freedom"* — a vision — *"a slim core surrounded by a range of plugins"* — and **ten project values**: fairness, respect, diversity, collaboration, inclusivity, transparency, quality, freedom, attribution, and think of the greater good — **✅ verified** (idempiere.org/about).

*Read the vision sentence again, because it is the licence and the architecture in five words: **"a slim core surrounded by a range of plugins."*** The slim core is GPL and genuinely open; the plugins are the community's growth surface; and it is a different answer to the same question Odoo's open-core is an answer to — how does an open-source ERP fund and steer its own development? iDempiere's answer is *it doesn't sell one*.

**Governance** is the community side of that answer. Wikipedia records: **"There is no single person or company ruling the project"**, and the project is organised as a decentralised bazaar, with local communities active in **Japan, China, USA, Germany, Indonesia, Brasil, Malaysia, Colombia, India, Russia** and elsewhere; communication runs through a multi-language project wiki, Google groups and Mattermost channels; wider reception comes through open-source conferences (Wikipedia names OpenRheinRuhr) and universities — **✅ verified** (Wikipedia *iDempiere*, Community-powered). The detailed contributor machinery is public in the repository itself: **CONTRIBUTING.md, CODE_OF_CONDUCT.md, a security policy, plugin guidelines, a git workflow guide, a pull-request review guide and a new-feature-suggestion guide** — **✅ verified** (§2.8; GitHub README; and the project's contributor documents). The project infrastructure that governance runs on — **JIRA at idempiere.atlassian.net, Jenkins at jenkins.idempiere.org, Mattermost at mattermost.idempiere.org, the Google Groups forum, docs.idempiere.org and wiki.idempiere.org, a donations page, Hacktoberfest participation and an "iDempiere Hero of the Month" community blog series** — is **✅ verified this pass** (GitHub README and the project's own links). §11.3 asks the uncomfortable question about what "community governance" and "community support" actually buy you; here the point is only that both are documented rather than asserted.
### 2.8 Recognition and Community Gatherings
**Two categories, deliberately separated.** First, **recognition** — attributed to the awarding bodies, never turned into a ranking:

- **InfoWorld's Bossie award**, which the awarding body titled for best open-source software application, in **2015 and 2016** — **✅ verified, attributed to the awarding body** (Wikipedia *iDempiere*, Awards and recognition).
- **SourceForge "Community Choice" Project of the Month, in July 2016 and February 2018** — **✅ verified, attributed to SourceForge** (Wikipedia).
- Wikipedia also lists a Russian Computer Review piece (**5 November 2015**), a **PCQuest** listicle (**20 May 2016**) and a **Capterra Logistics Technology Blog** listicle (**26 July 2017**) — **✅ as to their existence and dates**. **No ranking language is reproduced from them.** The honest framing: *listicle placements exist and are not evidence of position*; this guide refuses to rank, so it records that the listicles exist and nothing more.

Second, **community gatherings** — recorded as **attendance records, not adoption metrics** (Wikipedia *iDempiere*, World conferences — **⚠ secondary source**):

| Year / date | Place | Attendance as recorded |
|---|---|---|
| 2013 May 16–17 | **Krefeld, Germany** | about **30 core developers** from **4 continents** |
| 2014 March 7 | **Bonn, Germany** | the informal German iDempiere conference |
| 2015 Sep 3–4 | **Krefeld, Germany** | about **38 core developers** from **5 continents** |
| 2019 Oct 31–Nov 1 | **Lyon, France** | about **52 core developers** from **23 countries, 5 continents** |
| 2023 February 22–23 | **Manama, Bahrain** | about **30** core developers/implementors/integrators from **18 countries, 5 continents** |

**These are attendee counts at the community's own conferences.** They are the number of people who turned up, which is a fact about the community's events; they are *not* a measure of installations, users, market position or anything else, and this guide does not convert them into one. The pattern worth noticing is structural rather than numerical: the gatherings are geographically spread across three continents and roughly five years apart in the later period — a community whose physical meetings are rare, deliberate and far-flung.

---
## 3. The Architecture
### 3.1 The Tiers
iDempiere is a **three-tier web application assembled from OSGi bundles**. The project's own features page states the architecture in one line: *"iDempiere is based on original Compiere/Adempiere plus a new architecture to use state-of-the-art technologies like OSGi, jetty, ZK, maven"* — **✅ verified** (idempiere.org Features). The home page lists the same stack as a shopping list: *"Java, ZK, OSGi, Oracle, PostgreSQL, Maven"* — **✅ verified** (idempiere.org home).

The encyclopaedic description of the platform, useful because it names the tiers explicitly — **✅ verified, secondary source flagged ⚠**:

So the tier structure is:

- **Presentation** (the **ZK** web GUI, rendered in a standard browser) → **application** (the **Java server** in a **Jetty** container, hosting the OSGi runtime and every bundle — the dictionary engine, the document engine, the accounting engine, the UI bundle, the report bundles) → **data** (**PostgreSQL or Oracle**, holding both the business data *and* the dictionary that describes it, §4.6).

The **client platforms** are Windows, Linux, macOS and Solaris — **✅ verified** (Wikipedia infobox) — but those describe where the *server* may run, not where the user works, since the user works in a browser. This distinction is worth holding: installing iDempiere means installing a Java server; *using* it means opening a browser.
### 3.2 The Renderer — ZK
**The web GUI is built on ZK**, an Ajax framework for Java — **✅ verified** (Wikipedia infobox lists ZK under Platform; the Features page names ZK explicitly). The renderer matters to this guide for three reasons and [jetty_guide.md](jetty_guide.md) covers the container separately:

1. **ZK is server-side.** The UI is composed of Java components whose state lives on the server, and the browser is driven by the framework — which is why the dictionary's window/tab/field rows can render a live screen without a separate compile step (§4.4).
2. **ZK has been upgraded repeatedly, and each upgrade has UI consequences.** The 1.0 platform change is recorded as a **ZK 3.6 → ZK 6** upgrade that "allowed redesign of the web GUI" — **✅**; the **5.1** release updated ZK to **8.0.2.2** — **✅**; and Wikipedia's current Platform section names **ZK 9.6** — **✅/⚠ secondary**.
3. **Server push has been an unstable parameter.** The project's own v13 technical migration notes record that **release 13 initially shipped WebSocket as the default push server, was rolled back to Atmosphere after release-13-2026-04-13, and release 14 changed the default to WebSocket again**, with the workaround documented as a Java option `-Dorg.idempiere.ui.zk.serverpush=websocket` — **✅ verified** (docs.idempiere.org, v13 technical migration notes, read 2026-10-17). That is a rare, honest, first-party admission that a UI transport flip-flopped across a release line, and it is exactly the kind of operational detail the guide prints rather than hides.
### 3.3 The Databases and Their Version Floors
**PostgreSQL and Oracle, and only those two** — the constraint iDempiere inherited intact from ADempiere, whose database support Wikipedia records as "restricted to PostgreSQL and Oracle" — **✅ verified** (Wikipedia *ADempiere*). That restriction is structural, not incidental: the migration scripts, the `db/postgresql` and `db/oracle` subtrees in the repository, and the two database-provider bundles all encode the two-dialect assumption.

**The version floors are the single most disagreeable fact in the project's own documentation, and this guide prints a dated table rather than one number** — **⚠ flagged**:

| Source (vintage) | Java floor | PostgreSQL floor | Oracle floor |
|---|---|---|---|
| The project's **Features** page | OpenJDK **>= 11** | **>= 10** | **>= 12c** |
| The **development documentation for release 13** (prerequisites page) | **OpenJDK 17** | **17** in the guide ("you can use any postgres **>=14**") | "Oracle **23ai and later**" |
| Wikipedia's **Platform** section (current) | (not stated) | **>= 14** | **>= 23ai** |
| The **live test instances** (see §5.8) | **openjdk 17** (13 LTS demo); **oracle jdk 17** (Oracle dev) | **postgresql 18** | **Oracle 18c XE** (dev instance) |

All four rows are **✅ verified as recorded this pass** (idempiere.org Features; docs.idempiere.org prerequisites page; Wikipedia *Platform*; idempiere.org test sites / introduction page) — and they do not agree. **The guide does not smooth them into one claim.** The honest reading:

The honest reading, and the guide prints it rather than smoothing it: the **Features page** is simply older than the release-13 documentation (its floor was set at an earlier vintage); the **release-13 documentation** is the newest first-party statement — **Ubuntu 24.04, PostgreSQL 17 (any `>=14` acceptable), OpenJDK 17**, with **Oracle 23ai and later** also supported; Wikipedia has picked up the PostgreSQL/Oracle half of that pair but not the Java floor; and the **live test instances run *ahead*** of the documented floors (**postgresql 18** on the 13 LTS demo), with one genuine inconsistency — the Oracle dev instance runs **Oracle 18c XE**, *older* than the documented Oracle 23ai floor — recorded here without being resolved.

**Practical guidance stated as guidance, not as fact:** if you are standing up a release-13 instance today, the *documented* target is the release-13 prerequisite page (Ubuntu 24.04, PostgreSQL 17, OpenJDK 17, Eclipse 2025-12) — not the Features page, and not Wikipedia. The live test sites show the project itself running newer PostgreSQL and a different Oracle than its own floor. The reader should treat all four rows as the actual record and re-check the prerequisite page for their target release before provisioning.
### 3.4 The Multi-Tenant Model
**Every dictionary table carries two scoping columns: `AD_Client_ID` (the client, i.e. the tenant) and `AD_Org_ID` (the organisation).** This is stated in the documentation's own list of standard columns for a new table — **✅ verified** (docs.idempiere.org, Creating a New Window: "Every iDempiere table must include these standard columns… `AD_Client_ID`: multi-tenancy support… `AD_Org_ID`: Organisation"). The consequence is structural: **multi-tenancy is not a deployment mode bolted onto the side; it is a column on every row**, and the dictionary rows themselves are scoped that way, which is why a 2Pack has to be installed *into a named client* (§5.3) — the documentation warns "make sure you're logged into the correct client (e.g., System vs. GardenWorld)". Multi-tenancy is built into the *shape of the data*, not into a routing tier.

The documented capabilities, cited as **documentation-page titles that exist and were verified as existing** rather than as individually-read deep content — **✅ as page titles** (docs.idempiere.org sitemap; the project's new-features pages for v6.2, v9, v10 and v12):

- **Tenant administrator roles · specifying the tenant at login · per-tenant SMTP configuration · a per-tenant time zone for the cron scheduler · copying or moving a tenant.**

Wikipedia's Overview lists what the tenancy supports: **multiple tenants, multiple organisations (branches, plants, etc.), multiple languages, multiple currencies, multiple account schemas, and multi-user** — **✅ verified** (Wikipedia *iDempiere*, Overview; citation is to a Datamation slideshow, so **secondary**). Two of these deserve a note:

- **"Multiple account schemas"** is the finance-grade version of multi-tenancy: more than one set of books inside one login — a capability the project's own "Why iDempiere" list also advertises ("It supports multiple entities (sets of books) inside a single login") — **✅ verified** (idempiere.org introduction page). For §9, this is the feature that matters most to a bank.
- **Languages: the count disagrees.** Wikipedia's infobox says **16 languages**; Wikipedia's Overview text says **29 localizations (mixed access in a single installation is possible)** — **⚠ both recorded, not reconciled** (Wikipedia *iDempiere*). This is the third entry in this guide's catalogue of sources that disagree (§2.6, §3.3, §3.4) and it is the same lesson: print the disagreement.
### 3.5 Where OSGi Sits
**iDempiere *uses* OSGi; the runtime itself is [osgi_guide.md](osgi_guide.md)'s subject.** Stated once, cleanly, so the boundary is unambiguous:

- **The feature:** iDempiere's modularity is OSGi-based — the OSGi framework is the mechanism by which the application is assembled from bundles and by which plug-ins attach to it. Wikipedia: *"It uses the OSGi framework for modularisation"* — **✅**; and the plug-in feature is described as *"With the OSGi specification individual bundles of code (plug-ins) connect together in a dynamic way"* — **✅** (Wikipedia *iDempiere*, Platform and Plug-ins).
- **The runtime iDempiere embeds:** **Equinox**, the Eclipse OSGi implementation — **✅ verified** (Wikipedia *Development* names Equinox; the plug-in documentation instructs developers to set the plug-in target platform to **Equinox** — §5.4). The repository contains Equinox and Felix bundles side by side: `org.adempiere.eclipse.equinox.http.servlet` (an Equinox HTTP service inside the server), `org.idempiere.jetty.osgi.boot.fragment`, `org.idempiere.felix.webconsole` — **✅ verified** (GitHub repository root listing).
- **What this guide describes about OSGi usage, and only this:** the **Apache Felix Gogo console** on telnet **port 12612** (§5.5) and the **Apache Felix Web Console** at `https://<server>:8443/osgi/system/console/bundles` (§5.5), plus the bundle-level view of the core (§3.6).
- **What this guide does not describe:** OSGi's architecture, bundle lifecycle semantics, the service layer, declarative services, the compendium, or the platform-comparison table. Those belong to [osgi_guide.md](osgi_guide.md), and — as noted in §1.3 — iDempiere does not appear in that guide's §11 platform table, an omission recorded here and not repaired there.

The reason the OSGi question is not merely architectural for iDempiere is that **plug-ins are the project's stated growth mechanism** — the "range of plugins" around the "slim core" (§2.7). The dictionary renders the model; *plug-ins put behaviour around it*. The two extension layers (metadata and bundles) are the subject of §5.
### 3.6 The Repository Layout
The repository root at github.com/idempiere/idempiere is not just a file listing — it is a map of the architecture, and the module names are the evidence. All of the following were **✅ verified this pass** from the repository root (read 2026-10-17):

**Repository modules — the root listing is an architecture map** — **✅ all verified this pass** (read 2026-10-17)

| Module(s) | What it is / why it matters |
|---|---|
| `org.adempiere.base` | The core bundle: dictionary engine, PO model, services, event admin — what plug-ins `Require-Bundle` — **✅** |
| `org.adempiere.base.callout`, `org.adempiere.base.process` | Callouts and processes as **first-class bundles** — the strongest repository evidence that they are pluggable extension points, not conventions — **✅** |
| `org.adempiere.pipo`, `org.adempiere.pipo.handlers`, `org.adempiere.pipo-feature` | The **Pack-In/Pack-Out (2Pack)** engine; custom pack types are added by implementing handlers in `org.adempiere.pipo.handlers` — **✅** |
| `org.adempiere.plugin.utils` | Plug-in utilities — `AdempiereActivator` and `Incremental2PackActivator` (§5.3, §5.4) — **✅** |
| `org.adempiere.install`, `org.adempiere.server`, `org.adempiere.server-feature`, `org.adempiere.eclipse.equinox.http.servlet` | The installer; the Jetty-hosted application server; the Equinox HTTP service inside the servlet container — **✅** |
| `org.adempiere.payment.processor`, `org.adempiere.replication*`, `org.adempiere.report.jasper` (+ `-feature`, `.library`) | Payment processors; a replication subsystem; the **JasperReports** integration Wikipedia names — **✅** |
| `org.compiere.db.postgresql.provider`, `org.compiere.db.oracle.provider`, `org.compiere.db.provider-feature` | PostgreSQL and Oracle as **pluggable providers**, not hard-coded — **✅** |
| `org.adempiere.ui.zk`, `org.adempiere.ui.zk-feature`, `org.adempiere.ui`; `org.idempiere.zk.*` (themes, datatable, pivot, billboard, extra) | The **ZK web UI layer**, themes and widgets shipped as bundles — **✅** |
| `org.idempiere.acct`, `org.idempiere.felix.webconsole` (+ feature), `org.idempiere.jetty.osgi.boot.fragment`, `org.idempiere.extension.manager` | The accounting engine; the **Felix Web Console** (§5.5); Jetty's OSGi boot fragment; the newer extension manager (§5.4) — **✅** |
| `org.idempiere.p2*`, `org.idempiere.maven.to.p2`, `org.idempiere.p2.targetplatform`, `org.idempiere.equinox.p2.director-feature` | The **Maven/Tycho → p2** build machinery — why p2 is the distribution mechanism (§5.5, §7.3) — **✅** |
| `org.idempiere.webservices.client-feature`, `org.idempiere.ui.sso.oidc`, `org.idempiere.test`, `org.idempiere.test-feature`, `event.test`, `coverage-report` | Web-services client; UI OIDC SSO; the test and coverage modules (§6.4) — **✅** |
| `org.idempiere.tablepartition`, `org.idempiere.printformat.editor`, `org.idempiere.keikai`, `org.idempiere.hazelcast.service`, `org.idempiere.redis.service` | Table partitioning; print-format editor; a spreadsheet integration; **Hazelcast** and **Redis** service bundles — **✅** |
| `javax.websocket-api.fragment`, `lib`, `doc`, `db` (`db/postgresql`, `db/oracle`), `migration`, `migration-historic`, `utils_dev`, `.mvn/wrapper` | The websocket API fragment; libraries; docs; both database trees; current and historic migration scripts; utilities; the Maven wrapper — **✅** |

Two structural readings of that map, both **analysis grounded in the listing**: (1) **the extension points are bundles, not conventions** — `org.adempiere.base.callout` and `org.adempiere.base.process` existing as their own bundles is the strongest single piece of repository evidence that callouts and processes are first-class extension surfaces rather than a naming habit (§5.6); and (2) **the build machinery is itself visible** — `org.idempiere.p2*`, `org.idempiere.maven.to.p2`, `org.idempiere.p2.targetplatform` and `.mvn/wrapper` show a Maven/Tycho build that produces a **p2 repository**, which is why p2 is the recommended plug-in distribution mechanism (§5.5, §7.3).
### 3.7 The Request Lifecycle — How a Click Becomes a Row
Following the sibling's method ([odoo_framework_guide.md](odoo_framework_guide.md) §3.7), here is one request traced through the stack. The scenario: an accounts clerk edits a **Business Partner's credit limit** in a window and tabs out of the field, then saves.

1. **The browser** renders the ZK web client (§3.2). The user changes the value in a field and tabs away.
2. **The ZK layer** sends the change to the server. Because the UI is server-side, the framework has a live Java component graph for that window — the window, tab and field objects that the *dictionary* generated when the window was defined (§4.4).
3. **The dictionary resolves the field.** The field is not a hand-written input; it is an `AD_Field` row rendered by the dictionary, backed by an `AD_Column` row and its `AD_Reference` (data type) — **✅** (docs.idempiere.org, Creating a New Window).
4. **The callout, if one is attached to that column, fires here** — during a field-value change in the UI, in the browser session. This is the documented execution condition of a callout: **"during field value changes in UI"** — **✅** (docs.idempiere.org, Event Handlers vs Callouts comparison table). A callout attached to the credit-limit column could, say, validate the value or auto-fill another field.
5. **The user saves.** The server builds the persistent object (a `PO`) and persists it. **Before the database write**, any registered event handler for that table fires on the `PO_BEFORE_CHANGE` topic, and **after** the write, on `PO_AFTER_CHANGE` — **✅** (docs.idempiere.org, Event Handler guide: event handlers execute "before/after database operations"). This is the execution point a callout *cannot* reach: it runs at step 4, on the UI path only.
6. **The database write** lands on the physical table that the dictionary created in §4.2, in PostgreSQL or Oracle. **The table the row is written to, the window the field appeared in, and the dictionary entry that described the field are three renderings of the same `AD_Table`/`AD_Column` rows** — the thesis, in operation.
7. **The client refreshes**, and — if audit trail is enabled — the change may be recorded, since the project records "it is also possible to configure audit trails for the updated fields" — **✅** (Wikipedia *System admin*).

The architectural moral, and the one that runs through §10: **steps 1–3 and 6 are dictionary-rendered, but steps 4 and 5 are where behaviour attaches — and the two attach at different points in the lifecycle.** A callout attaches to the *UI field-change* (step 4, UI only); an event handler attaches to the *persistence operation* (step 5, all entry points). The choice between them is not stylistic — it decides which entry points the behaviour can see. That is the seam the next sections pull on.
### 3.8 The Architecture Table

| Layer | Technology | Role | Marker |
|---|---|---|---|
| **Client** | A standard browser (Firefox, Chrome, Opera, Edge) | Runs the ZK-driven web UI; no install on the client | **✅** (Wikipedia *Platform*) |
| **Renderer** | **ZK** (Ajax framework for Java; ZK 9.6 in Wikipedia's current Platform section) | Server-side UI component graph that renders the dictionary's window/tab/field rows | **✅/⚠** (Wikipedia names ZK 9.6; secondary) |
| **Application server** | **Java server on a Jetty container** | Hosts the OSGi runtime and every bundle | **✅** (Wikipedia *Platform*) |
| **Runtime / modularity** | **OSGi**, embedded as **Equinox**; Felix Web Console present | Assembles the application from bundles; the plug-in attach point | **✅** (Wikipedia; repo listing) — runtime itself owned by [osgi_guide.md](osgi_guide.md) |
| **Model / metadata** | The **Application Dictionary** (`AD_*` tables) | Describes tables, columns, windows, tabs, fields, references, validations, menus, roles | **✅** (Wikipedia; docs.idempiere.org) |
| **Data** | **PostgreSQL** or **Oracle** only | Holds business data *and* the dictionary | **✅** (Wikipedia *ADempiere*; repo `db/` subtrees) |
| **Reporting** | **JasperReports** integration | Complex reports beyond the built-in format | **✅** (Wikipedia *Reports*; `org.adempiere.report.jasper` bundle) |
| **Workflow** | A **WfMC/OMG-standards-based** workflow engine | Document workflow and BPM | **✅** (Wikipedia *Workflows*; Wikipedia *ADempiere*, Architecture) |
| **Integration** | **Web services** (XML formats; input/output templates for any table, process or workflow) | Machine-to-machine calls into the server | **✅** (Wikipedia *Web services*) |
| **Configuration packages** | **2Pack** (`.zip`) | Moves dictionary metadata and configuration between environments | **✅** (docs.idempiere.org) |
| **Build** | **Maven + Tycho** (Buckminster before 6.1) | Produces the p2 repository and the product | **✅** (Wikipedia; docs.idempiere.org; `org.idempiere.p2*`) |

---
## 4. The Application Dictionary in Depth
*This is the core section of the guide. It renders the project's own six-step window-creation workflow faithfully, then tests the thesis against it. The workflow below is drawn from the project's documentation page "Creating a New Window in iDempiere" (docs.idempiere.org, read 2026-10-17) — **✅ verified** unless noted.*
### 4.1 What the Dictionary Is
**The Application Dictionary is the metadata layer that describes the application to itself.** Wikipedia's definition is the thesis in one sentence: iDempiere *"extends the concept of a data dictionary to an 'Active Data Dictionary' that lets it manage entities, validation rules, windows, tables, formats, and other customizations of the application without new Java code. So iDempiere can be seen not only as an ERP but also as a platform to build database driven applications"* — **✅ verified** (Wikipedia *iDempiere*, Key features).

The word **"Active"** is the load-bearing one. A passive data dictionary *documents* a database; an active data dictionary *is consulted by the running application* to decide what a table is, what a field looks like, what a reference means and who may see it. The dictionary is not documentation *about* the application — it is one of the application's runtime inputs. And, as §4.6 establishes, it lives in the **same database** as the business data.

The dictionary is edited through windows that are themselves dictionary-driven — the same mechanism, applied to itself. The entry point for the workflow is **System Administrator → Application Dictionary → Table and Column** — **✅** (docs.idempiere.org). Everything in §4.2–§4.5 happens inside that window and the Window, Tab & Field window.
### 4.2 Step 1 — Create the Table
**Navigate to: System Administrator → Application Dictionary → Table and Column.** Then:

1. Click **New Record**.
2. Fill in the required fields — **✅** (docs.idempiere.org):
   - **DB Table Name** (the documentation's example is `my_customtable`);
   - **Name** (a human title, e.g. "My Custom Table");
   - **Description** (a brief description of the table's purpose);
   - **Entity Type** — and the documentation's instruction here is a rule, not a suggestion: use **your custom entity type (e.g. `U` for user customisation)**, never `Dictionary`;
   - **Data Access Level** — one of **System / Organization / Client+Org / All**;
   - **Window** — leave blank for now.
3. **Save** the record.

**Then run the "Create/Complete Table" process.** The documentation is specific: *"We recommend clicking the gear icon button and running the 'Create/Complete Table' process. This process lets you select common columns that an iDempiere table typically needs — you can choose the ones that fit your use case (e.g., if you're creating a document or transaction table)"* — **✅** (docs.idempiere.org).

**What every iDempiere table must include** — the documentation states this as a hard list, and it is the list that makes the multi-tenancy and audit behaviour work everywhere (**✅ verified**, docs.idempiere.org):

| Column | Purpose |
|---|---|
| `TableName_ID` | Primary key |
| `TableName_UU` | UUID |
| `AD_Client_ID` | Multi-tenancy (the tenant) |
| `AD_Org_ID` | Organisation |
| `IsActive` | Active flag |
| `Created` | Creation timestamp |
| `CreatedBy` | User who created |
| `Updated` | Last update timestamp |
| `UpdatedBy` | User who updated |

*That list is why the thesis works at scale.* Because **every** table carries `AD_Client_ID`/`AD_Org_ID`, every table is automatically tenant-scoped, and because every table carries the `Created`/`Updated` audit columns, every table is automatically auditable. The uniformity is not a convention developers are asked to honour — the **Create/Complete Table** process exists precisely to stamp it onto every new table. A dictionary-driven system whose dictionaries were *not* uniform would not scale; iDempiere makes the uniformity structural.
### 4.3 Step 2 — Configure the Columns
**Navigate to the Column tab in the Table and Column window.** The *Create/Complete Table* process has already created the common columns, so this step is about the columns specific to the table — **✅** (docs.idempiere.org).

For each column, set:

| Property | Meaning (per the documentation) |
|---|---|
| **System Element** | Defines the values and metadata for the column |
| **Name** | User-friendly name; **becomes the field label** (auto-populated from the element) |
| **Description** | Purpose of the column; **appears in the field's help tooltip** (auto-populated from the element) |
| **Reference** | Data type — String, Integer, List, Table, etc. |
| **Length** | Length |
| **Mandatory** | Whether a value is required |
| **Updatable** | Whether users can modify the value |
| **Always Updatable** | Editable **even if the record is processed or inactive** |
| **Identifier** | Marks the column as a key identifier |

**Two special configurations, called out by the documentation — ✅:**

- **Foreign-key columns:** use the **Search**, **Table** or **Table Direct** reference, with **Reference Key** selecting the referenced table, and **Mandatory** set per requirement.
- **List columns:** use the **List** reference plus a **Reference List** (created or selected).

**A detail that is easy to miss and worth naming:** the documentation shows how to create a **System Element inline** — *right-click on the System Element field → New → fill in the element details (Name, Column Name, Print Name, Description, Help) → Save* — so a developer never has to leave the record. The point of the **System Element** indirection is that a column's label and help text are *separately stored metadata* shared across every column that uses the same element; the label is not a string on the column.

**Then run "Synchronize column" to create the columns in the database** — **✅** (docs.idempiere.org). This is the first of the two processes that make the dictionary *active*: the metadata rows just created are turned into physical DDL. Note the asymmetry the two processes establish — **Create/Complete Table** helps you *populate* the dictionary rows; **Synchronize column** *applies* them to the physical schema. Metadata first, physical schema second, and the physical schema is derived.
### 4.4 Step 3 — Create the Window, Tab and Field
**Navigate back to the Table record**, click the **Process** button (gear icon), select **Create Window, Tab and Field from Table**, configure the parameters, and click **OK** — **✅** (docs.idempiere.org).

**The process automatically creates:**

- a **new Window record** (if "New Window" is checked);
- a **Tab** linked to your table;
- **all Fields corresponding to your table columns**;
- the **menu record** for your window.

*This is the moment the thesis becomes literal in the UI.* A single process reads the `AD_Table` and `AD_Column` rows created in §4.2–§4.3 and generates the `AD_Window`/`AD_Tab`/`AD_Field` rows that render a screen — plus the menu record that makes it reachable. The screen is *derived from* the same rows the physical table was derived from. This is what "three renderings of one dictionary entry" means in practice: the table row renders as a physical table (§4.3), as a screen (§4.4) and as a menu-and-access pair (§4.5).

**The new-window-versus-detail-tab decision — the documentation draws the line clearly — ✅:**

| Choose | When | Detail |
|---|---|---|
| **Create a new window** (check "New Window") | The table is master data or a main entity; users access this data **independently**; the table is not primarily a child of another entity | — |
| **Add as detail tab** (uncheck "New Window") | The table is a **child/detail** of an existing entity; data should always be viewed **in context of a parent record** | Set **Tab Level** to **1 or higher** (e.g. Order → Order Lines) |

**After running the process**, the documentation says you can *"further customize the window, tab, and fields in the Window, Tab & Field window to adjust layouts, add display logic, and fine-tune the user interface"* — **✅**. So generation is a *scaffold*, not a cage: the process produces a working screen, and the Window, Tab & Field window is where you refine it — all still metadata.

**Parent-child tabs** follow the same pattern (**✅**, docs.idempiere.org): create a second tab with **Tab Level = 1**, set the **Parent Column** link to the parent tab, and configure the **Link Column** (the foreign key back to the parent).
### 4.5 Steps 4 to 6 — Menu, Role Access, Test
**Step 4 — Add to the menu.** Navigate to **System Administrator → Application Dictionary → Menu**; if the *Create Window, Tab and Field* process did not already create the menu entry, create it manually here — find the folder, **New Record**, configure, **Save** — **✅** (docs.idempiere.org). *Note that step 3 usually creates it: the process's outputs include "the menu record for your window". The manual step exists for the case where it was not generated.*

**Step 5 — Assign role access.** Navigate to **System Administrator → Security → Role**, open the role, go to the **Window Access** tab, find the new window, enable it, and set the access level (**Read Only / Read Write**); repeat for other roles — **✅** (docs.idempiere.org).

*This is the third rendering completing itself.* A window that exists but is not granted to a role is invisible to every user. The visibility of the screen is **also metadata** — role-and-window-access rows in the dictionary. And because the same Role window also carries the role's data-access scope, the *who-sees-what* question is answered in the dictionary the same way the *what-is-it* question is.

**Step 6 — Test the window.** **Log out and log back in as a different role**, navigate to the new menu entry, and test: create records, edit records, verify field validations, check mandatory fields, test foreign-key lookups — **✅** (docs.idempiere.org). *The instruction to log out and back in is not incidental: role access is resolved at login, so a role change is only observable after a fresh session.*
### 4.6 The Thesis Test — the Dictionary Is Rows in the Same Database
The thesis is that the model, the screen and the database table are three renderings of one dictionary entry. §4.2–§4.5 show it holding on the way *out* (one `AD_Table` row → physical table, screen, menu, access). This subsection tests it on the way *in*: **what are the dictionary entries themselves?**

**The dictionary is itself rows in the same database, and the `AD_*` tables are themselves described by dictionary entries.** `AD_Table` is a table; it has `AD_Table` rows describing it and `AD_Column` rows for its columns; `AD_Window`, `AD_Tab` and `AD_Field` are tables too, described the same way. The dictionary is not a separate file, a separate schema, or a build artefact — it is data, in the same PostgreSQL or Oracle database the business data lives in, obeying the same standard columns (`AD_Client_ID`, `AD_Org_ID`, the audit columns of §4.2). **This is the structural fact that makes the thesis possible**, and it is **analysis grounded in the documented behaviour** of §4.2–§4.5 plus the standard-column list, marked as analysis rather than as a quoted fact.

The consequences of "dictionary as data" are far-reaching and are the reason this guide keeps returning to it:

- **The dictionary is editable at runtime, through the application itself.** You do not stop the server, edit a model file and redeploy — you open a window and change an entry. This is the "without new Java code" claim made literal.
- **The dictionary has the same entry points as business data** — which is precisely why the callout scope limit (§5.6, §10.1) is such a sharp failure: a table change made through the UI and the same change made through the REST API are indistinguishable to the dictionary but not to a callout.
- **The dictionary travels in the same package format as everything else.** A 2Pack (§5.1) exports dictionary rows alongside business rows, because they are the same kind of thing. "Configuration" and "data" are not separate categories in iDempiere — they are rows.
- **Tenancy and audit are dictionary properties too.** `AD_Client_ID` on the dictionary tables means the dictionary itself is tenant-scoped, which is why a 2Pack must be installed into a named client (§5.3).

**Where the thesis already strains, at the end of the core section.** The dictionary renders the model, the screen and the table. What it does **not** render is **behaviour** — the calculations, validations and side-effects that are the difference between a data-entry form and a working ERP. Behaviour is code (a callout, an event handler, a process — §5.6), attached to dictionary entries but not expressed *in* them. So the precise form of the thesis, at the end of §4, is:

> **One dictionary entry renders as a physical table, a screen, and an access rule — and it carries named hook points to which behaviour, defined in code, may attach.**

That is a *stronger* and more honest statement than "the dictionary is the whole application", and §10 tests exactly this formulation against the hardest case the documentation offers (the callout scope limit). Note also what the thesis does *not* claim: it says nothing about *how well* the dictionary scales, or whether dictionary-driven customisation is cheaper than code — those are §11's questions, and they have uncomfortable answers.
### 4.7 The Documented Best Practices
The documentation closes the window-creation guide with a best-practice list, and every item is worth having because each is a rule the project itself learned — **✅ verified** (docs.idempiere.org):

- **Naming conventions:** primary keys as `tablename_ID`; names clear, descriptive and user-friendly.
- **Entity types:** use a **custom entity type, not `Dictionary`** (repeated from §4.2 — it matters enough to appear twice).
- **Performance:** create appropriate database indexes; **prefer the `Search` reference for foreign keys** when possible (the documentation also calls this out in the foreign-key configuration itself).
- **Documentation:** **always fill the Description and Help fields**; document business purpose and usage; include examples in the help text. *This is not decoration:* because **Name** becomes the field label and **Description** becomes the field's tooltip (§4.3), the "documentation" of a field is its user-facing UI text — filling it is filling in the screen.

---
## 5. The Extension Model
iDempiere has **three extension layers**, and keeping them apart is the key to reading any iDempiere customisation:

1. **The dictionary** (§4) — metadata, edited live, rendered into model, screen and table. No code.
2. **2Packs** (§5.1–§5.3) — `.zip` packages that move dictionary metadata and configuration between environments. Still no code, but *transportable*.
3. **OSGi plug-ins** (§5.4–§5.7) — bundles of Java that attach behaviour at named extension points. Code.

The layers compose: a plug-in can *carry* a 2Pack, and a 2Pack can *configure* what a plug-in needs. §5.7 traces that composition end to end.
### 5.1 2Packs — the Configuration Package
**A 2Pack is iDempiere's packaging mechanism for application metadata and configuration.** The documentation's own definition: *"A 2Pack allows you to export and import application metadata and configuration (such as windows, processes, tables, reports, and even data) between different iDempiere environments"* — described as a **"configuration snapshot"** — **✅ verified** (docs.idempiere.org, *What is a 2Pack*, read 2026-10-17).

**Its three documented purposes — ✅:**

- Transfer configurations between **development, test and production** environments;
- Distribute customisations as part of **plug-ins**;
- Automatically **initialise or update** application components when a plug-in is deployed.

**What a 2Pack can carry — ✅** (docs.idempiere.org): windows/tabs/fields; processes/reports (custom processes or Jasper reports); tables/columns; roles/menus/messages; **SQL scripts** (executed automatically during Pack In); dynamic validations; print formats/references; and **arbitrary data** — any row from any table, via SQL.

*Read that table against §4.6 and the thesis closes the loop.* A 2Pack's contents are the dictionary rows §4 creates — windows, tabs, fields, tables, columns — plus roles, menus and messages, plus raw SQL and arbitrary table data. In iDempiere, "configuration" **is** rows, so the package format for configuration is a package of rows. That is why the documentation's benefits list includes *"Cleanly separates code from configuration"* — **✅** — as a claim worth stating: the separation is real, and it is between *bundles* (code) and *2Pack rows* (configuration).
### 5.2 Pack Out and Pack In
**Pack Out** exports configuration and/or data into a **`.zip`**; **Pack In** imports it. The `.zip` contains **an XML manifest plus associated data files** — **✅** (docs.idempiere.org).

**Pack Out — the documented workflow — ✅** (docs.idempiere.org, *Using 2Pack to Transfer Data and Automate Plug-in Setup*):

1. Log in as **System Administrator** for system-level configurations, and open **Application Dictionary > Application Packaging > Pack Out**. 2. Create an entry: **Name** (per the file-naming recommendations), **Package Version** (match the plug-in version from `MANIFEST.MF`), optional **Description/Instructions**, **Date From**, and **Export Dictionary Entity** (include entity-type `D` entries). 3. Add **Package Details** in the *Package Detail* tab: *Application or Module*; *Data* (custom SQL); *File / Code Snippet*; *Process / Report / Form / Window / Table / Workflow*; *Role / Message / Import Format / Entity Type / Model Validator*; *Print Format / Dynamic Validation / Reference / Report View / SQL Statement*. 4. For a *Data* detail, select the target table, write an SQL filter, and click **Export Package** in the **Export Package** tab.

The result is a `2Pack.zip` in the packout folder, downloadable through the Web UI — **✅**.

**A detail that shows the architecture's depth:** *"You can add custom types by implementing new PIPO Handlers in the `org.adempiere.pipo.handlers` bundle"* — **✅** (docs.idempiere.org). The PIPO handlers are a real bundle in the repository (§3.6), so **the 2Pack type system is itself extensible by a plug-in** — a fourth-order extension point (a bundle that teaches the package format a new kind of row).

**Packing out related records** uses a structured SQL notation — **✅**: `SELECT * FROM C_BPartner WHERE [condition];AD_User>R_ContactInterest;C_BPartner_Location`, where `;` separates reference paths and `>` defines nesting/depth. For implementation, the documentation points to `GenericPOElementHandler.exportDetail()` / `.create()` in the repository.

**Pack In — the documented workflow — ✅:** go to the **Pack In** window, create a new record, **upload the `.zip` as an attachment**, and click **Pack In**. The warning is explicit: **"Make sure you're logged into the correct client (e.g., System vs. GardenWorld)"** — because the dictionary is tenant-scoped (§4.6), a 2Pack lands in a *client*, and packing into the wrong one is a real mistake. Installed packages are visible via **Packages Installed**, and **Package Maintenance** performs rollback (select the package and run the **Pack Roll** process); the documentation notes that a `Data`-type detail "may not appear in rollback details but still rolls back" — **✅**.
### 5.3 Automatic External Pack In
The most operationally interesting 2Pack feature is **Automatic External Pack In**: *"iDempiere automatically scans a configured folder for `.zip` files, installs them automatically on startup"* — **✅** (docs.idempiere.org). The documentation recommends it explicitly **for CI/CD pipelines and Docker deployments**, to achieve "zero-touch configuration updates" — **✅**.

**Its filename convention is mandatory**, because the folder scan parses the name — **✅**: `[Timestamp]_[ClientValue]_[AdditionalInformation].zip`, where **Timestamp** is `YYYYMMDDHHMM`, **ClientValue** is the client identifier (the documentation names **`SYSTEM`** and **`GardenWorld`**), and **AdditionalInformation** describes the pack. The documented example is **`201803161725_SYSTEM_CreateNewColumn.zip`**, and best practice is a descriptive `AdditionalInformation` so multiple packs are identifiable — **✅**.

**The two activator mechanisms** that install 2Packs from a plug-in — **✅** (docs.idempiere.org):

- **`AdempiereActivator`** — *basic activation*. Set it as the plug-in's activator in `MANIFEST.MF`, add `org.adempiere.plugin.utils` as a dependency, and place the `2Pack.zip` inside the `META-INF` folder; when the plug-in starts, the 2Pack is installed if not already present. The rule: **update the plug-in and 2Pack version when making changes — only newer versions will be installed.**
- **`Incremental2PackActivator`** — *advanced activation with version control*. Handles **multiple** 2Packs named with a versioning pattern: `2Pack_<Name>_<version>.zip` (examples `2Pack_Tables_1.0.0.zip`, `2Pack_Processes_1.0.1.zip`), loaded and installed in order. Newer iDempiere versions (**after IDEMPIERE-4105**) also accept `2Pack_1.0.0_Tables.zip`, where **only the portion after the version may contain underscores** — **✅**.

*The distinction between the two activators is the whole difference between "ship a configuration" and "ship a versioned migration sequence".* The Incremental activator's ordering and versioning is what makes a 2Pack set behave like migrations rather than like a one-shot snapshot — and it is the reason §11.1's upgrade discipline (review migration notes per version jump) has a code-level counterpart.
### 5.4 OSGi Plug-Ins — the Code Extension
**Where the dictionary runs out — behaviour — a plug-in begins.** The documentation develops plug-ins **outside the core repository**, deliberately — the page is titled *"Developing Plug-ins Without Affecting the Core"* ("Trunk" in the documentation's internal cross-references) and states the goal as *"ensuring clean separation between your customizations and the official core codebase"* and *"enabling clean merges of iDempiere updates without interference from plug-in code"* — **✅ verified** (docs.idempiere.org).

**The documented development workflow — ✅** (docs.idempiere.org):

1. In Eclipse: **File → New → Other… → Plug-in Project**, give it a **custom project name** (e.g. `com.yourcompany.pluginname`), and **uncheck "Use default location"** so the project lives **outside the iDempiere core repository** (e.g. `idempiere-plugins/your-plugin-name`).
2. Set the plug-in **target platform to Equinox** and choose a compatible **Java execution environment (JavaSE-17 or later)**; **disable options like "Generate an activator"**.
3. **Share the project to version control** (right-click → Team → Share Project → Git).

The documentation also offers a **community scaffolding tool** — the **iDempiere Plugin Scaffold** — as an alternative to starting from scratch — **✅**. (The tool is third-party, hosted at `github.com/saulojg/idempiere-plugin-scaffold/`, and is named here as a community resource, not a project guarantee.)

**The steps that make it a *plug-in* rather than an Eclipse project — ✅:**

- **Target platform Equinox** (step 4) gives the project the OSGi/Eclipse bundles to compile against — the same Equinox the server embeds (§3.5); **JavaSE-17 or later** (step 5) matches the release-13 toolchain (§6.3).
- **"Generate an activator" disabled** (step 6) is a deliberate default: most plug-ins register their services through **OSGi declarative components** (`@Component`) rather than through a bundle activator's `start()` (§5.6). **Outside the core repository** (step 3) is the organisational counterpart of the technical boundary — the reason an upstream update merges cleanly, and the reason the repository root carries `.mvn/wrapper` while plug-ins live elsewhere (§6.1).

**The Maven/Tycho build path for plug-ins** is documented separately and is the modern approach — **✅** (docs.idempiere.org, *Building iDempiere Plug-ins with Maven*, contributed by Luis Amesty): **from iDempiere 6.1 onward, Maven with Tycho replaced Buckminster as the build system** (matching §2.5). A Maven-based plug-in set has this layout (**✅**): `org.myplugin.feature/` (feature), `org.myplugin.editors/` (plugin), `org.myplugin.themes/` (fragment), `org.myplugin.p2.site/` (p2 repository), `org.myplugin.p2.targetplatform/` (target platform), and a root `pom.xml`.

The documented rules of the layout (**✅**): the **feature** project uses packaging `eclipse-feature`; the **plugin** and **fragment** projects both use packaging `eclipse-plugin` (a fragment is distinguished by its `Fragment-Host` entry in `MANIFEST.MF`, **not** by a distinct packaging type); the **p2 site** project uses `eclipse-repository` and assembles the deployable p2 repository; the **target platform** project uses `eclipse-target-definition` and points at the local iDempiere p2 repository so Maven can resolve dependencies. The build command is `mvn verify -Didempiere.target=org.myplugin.p2.targetplatform`, and the generated **plugin and feature JARs land under `org.myplugin.p2.site/target/repository/`** — **✅**. The documentation's own advice, repeated across pages: **"Keep the plug-in directory alongside (not inside) the iDempiere source tree so it can be built independently and upgraded without merge conflicts"** — **✅**. That sentence is the plug-in model's whole philosophy in one line.
### 5.5 Distribution — p2, JAR, Gogo, Felix Web Console
iDempiere documents **three ways to get a bundle onto a server**, and their existence is itself an architectural fact — you can install a bundle by *feature repository*, by *file*, or by *live console* — **✅ verified** (docs.idempiere.org, *Distributing and Installing Plug-ins*):

**1. Via a p2 repository (recommended).** From the server directory: `cd /opt/idempiere-server; bash update-prd.sh <repository_url> <feature_id>`. The documentation's example installs the **REST API plug-in** from the project's Jenkins: `bash update-prd.sh https://jenkins.idempiere.org/job/idempiere-rest/ws/com.trekglobal.idempiere.extensions.p2/target/repository/ com.trekglobal.idempiere.rest.api`.

There is a **state-checking variant, `compare-update-prd.sh`**, which "does not uninstall when the plug-in is not installed, does not reinstall when the same version is already present, and uninstalls/reinstalls only when a different version is detected" — and the upgrade guide's tip is to **prefer compare-update for routine maintenance** to avoid unnecessary restarts and noisy harmless errors — **✅** (docs.idempiere.org *Distributing…* and the Wayback-captured **Upgrade Guide**, capture dated 2026-06-10). The documentation also warns that these commands must be written on **a single line** — **✅**.

**2. As a deployable JAR.** In Eclipse: right-click the project → **Export > Deployable plug-ins and fragments** → choose a directory → in the **Options** tab select **Package plug-ins as individual JAR archives** → Finish. The result is an installable JAR — **✅**.

**3. Via the Apache Felix Gogo console (if no p2 repository).** Connect by telnet: `telnet localhost 12612`.

The documented commands — **✅**: `ss` (short status; e.g. `ss rest`), `install` (from URL or local file, e.g. `install file:///tmp/….jar`), `uninstall`, `setbsl` (start level; e.g. `setbsl 5 com.trekglobal.idempiere.rest.api`), `sta`/`sto` (start/stop), and `help`.

**The Apache Felix Web Console** is the graphical alternative — **✅**: visit `https://<your-server>:8443/osgi/system/console/bundles`, log in as **SuperUser** (or another System Admin), use **Install/Update…**, browse for the `.jar`, and set **Start Bundle ✅**, **Refresh Packages ✅**, **Start Level `5`**; click *Install or Update*, then verify the bundle is active (using the "play" button to start it manually if needed).

*Note the two port numbers here and hold them for §7.2: **12612** is the Gogo telnet console and **8443** is the HTTPS/Felix Web Console port.* Both being open on a running server is a security fact, not a convenience (§5.8, §11.3).

**The newer distribution route — the Extension Registry (iDempiere 14+).** The project's documentation set adds an **Extension Registry** whose stated purpose is to distribute plug-ins through the official registry "so any iDempiere 14+ instance can find and install it from the **Extension Management** form" — **✅ verified** (docs.idempiere.org, *Contributing an Extension to the Registry*, read 2026-10-17). The mechanics are unusually strict and worth recording precisely, because they are a supply-chain control: the registry lives at **`github.com/idempiere/idempiere-extension-repository`**, that URL is the default value of the environment variable **`IDEMPIERE_EXTENSION_REPOSITORY`** baked into the server scripts and product files, and **"you do not upload a jar anywhere"** — the contributor publishes *source* and sends a pull request adding a few text files (`info.md`, a `metadata.json` per version, an optional `CHANGELOG.md`), and **the project builds the binary from the contributor's source itself** — **✅**. The repository contains validation and index-generation GitHub workflows (`validate-extensions.yml`, `generate-index.yml`) and a `skills/idempiere-review-extension/` directory, and the documentation states the acceptance policy exists because *"a catalog that installs code into production ERP systems is a supply-chain surface"* — **✅**. **This is a 14-and-later feature and is flagged as such**; release 13 (the current stable) distributes plug-ins by the p2/JAR/Gogo routes above. The repository's `org.idempiere.extension.manager` bundle is the corresponding runtime piece — **✅** (repo listing).
### 5.6 The Documented Extension Points — Callouts vs Event Handlers vs Model Validators
The documentation set lists the extension points **by name**, and the list is worth reproducing because it *is* the answer to "what can a plug-in attach to?" — **✅ as the project's own developer-documentation page list** (docs.idempiere.org):

> **callouts · event handlers · model factory (`IModelFactory` / custom models) · model validators (`ModelValidator` / `IModelValidatorFactory`) · processes · document factory · editor factory · credit manager factory · tax provider factory · forms · dashboard panels · user panel menu · WebUI themes · server push** — plus **"annotations instead of XML"** and the newer **extension registry** mechanism with its own contributing and worked-example pages.

Read that list structurally: the extension points cluster into **field-level** (callouts), **record-level** (event handlers, model validators), **model-level** (model factory, custom models), **process-level** (processes, document factory), **factory-level** (editor, credit-manager, tax-provider factories), **UI-level** (forms, dashboard panels, user panel menu, themes, server push) and **packaging-level** (annotations, the extension registry). In other words, **there is a hook at nearly every layer** — which is the strongest evidence that iDempiere is a platform, not just an ERP.

**The three behaviour mechanisms a developer actually chooses between** — callouts, event handlers and model validators — are distinguished by the documentation with unusual precision, and this is **the single most important technical passage in the whole guide** — **✅ verified** (docs.idempiere.org, *Developing Plug-in Callouts*, *Developing Plug-in Event Handlers*, *Developing Plug-in Model Validators*, all read 2026-10-17):

**Callouts — field-level, UI-only.**

- A callout is *"a piece of code that executes automatically when a user changes the value of a field in a window"* — commonly used to validate input, auto-fill related fields, perform calculations, show warnings, dynamically enable/disable fields, and load data from related records.
- **The scope limit, verbatim:** *"Callouts **only work** when data is entered through: the iDempiere UI, the CSV Importer."* And *"Callouts do **NOT** execute when records are created or modified via: REST API calls, background processes, direct database operations, Java code using `PO.save()`."*
- **The documented remedy:** *"If you need validation or logic that works across all entry points, use an Event Handler instead, which intercepts all record changes regardless of the source."*
- **Implementation approaches, with version markers:** the **Mapped Column Callout Factory (iDempiere 9+)** — three registration methods (direct registration via bundle activator, OSGi component with service binding, factory subclass); **Annotation-Based Callouts (iDempiere 9+, recommended)** using the `@Callout` annotation; and the **Legacy `CalloutFactory` (pre-iDempiere 9)** using the `IColumnCalloutFactory` interface. The documentation notes a trap: `Core.getMappedColumnCalloutFactory()` *"can return null if your bundle activates before the `IMappedColumnCalloutFactory` service"*, so the safe registrations are the OSGi-component ones.

**Event handlers — record-level, all entry points.**

- An event handler is *"a component that listens to and responds to events triggered by changes to data models (tables)"*, executed *"whenever records are created, updated, or deleted, regardless of how the change was initiated"*.
- The system *"is built on top of the **OSGi Event Admin Framework**"*, and the **`IEventManager`** service (provided by the `org.adempiere.base` bundle) registers handlers that *"replace the legacy `AD_ModelValidator` table and `ModelValidator` interface approach"*.
- Handlers extend **`AbstractEventHandler`**, override **`initialize()`** (to register table events) and **`doHandleEvent(Event event)`** (the logic), and register against topics from **`IEventTopics`**: **model (PO) events** (`PO_BEFORE_NEW`, `PO_AFTER_NEW`, `PO_BEFORE_CHANGE`, `PO_AFTER_CHANGE`, `PO_BEFORE_DELETE`, `PO_AFTER_DELETE`, the asynchronous `PO_POST_*` variants, `PO_ALL`), **document events** (`DOC_BEFORE_*` / `DOC_AFTER_*` for prepare, complete, void, close, reactivate, reversecorrect, reverseaccrual, post), **import events** (`IMPORT_BEFORE_VALIDATE`, `IMPORT_AFTER_VALIDATE`, `IMPORT_BEFORE_IMPORT`, `IMPORT_AFTER_IMPORT`), **process events** (`BEFORE_PROCESS`, `AFTER_PROCESS`, `POST_PROCESS`) and **system events** (`AFTER_LOGIN`, `ACCT_FACTS_VALIDATE`, `PREF_AFTER_LOAD`, `DOCACTION`, `BROADCAST_MESSAGE`, `REQUEST_SEND_EMAIL`, `REPORT_SEND_EMAIL`).
- The annotation era: *"Starting with **iDempiere 9**, you can use annotations to register event handlers. This is the simplest and recommended approach"* — **`@Component(service = org.osgi.service.event.EventHandler.class)`** with a `@Reference` binding of the `IEventManager`.

**Model validators — record-level, the legacy registration path.**

- A model validator *"intercepts and validates model (record) operations"* and is *"similar to event handlers but uses a different registration mechanism"*: the **`ModelValidator`** interface plus **`IModelValidatorFactory`**, versus event handlers' OSGi Event Admin plus `IEventManager`.
- Model validators are configured in a **Model Validator record** (*System Admin > General Rules > Model Validator*) naming the validation class, and the documentation states plainly they are **"legacy code"** versus event handlers' **"new development"**, recommending event handlers for new work *"as they provide better OSGi integration and more flexibility"*.

**The comparison the documentation draws — reproduced faithfully — ✅:**

| Feature | Event Handler | Callout |
|---|---|---|
| **Scope** | All operations (UI, API, processes, imports) | **UI and CSV Importer only** |
| **Execution Point** | **Before/after database operations** | **During field value changes in UI** |
| **Use Case** | System-wide business rules and validations | UI-specific field interactions |

The model-validator-versus-event-handler distinction is a **registration** difference, not a capability one: validators register through **`IModelValidatorFactory`** against the **`ModelValidator`** interface, event handlers through **OSGi Event Admin** against **`EventHandler`/`AbstractEventHandler`**. The documentation marks validators as *legacy code* and event handlers as *new development*, *"more flexible with topics"*, and **the modern OSGi approach** — **✅** (docs.idempiere.org).

*The whole of §10 rests on that "Scope" row.* A callout is a dictionary-attached behaviour with an entry-point restriction the dictionary cannot express — and that is where the metadata illusion fails. Everything else in §5 is the machinery around that seam.
### 5.7 How a Real Customisation Is Built and Moved
Putting §5.1–§5.6 together, here is the documented end-to-end path from idea to production — **✅ verified** as the composition of the documented workflows, marked **analysis** where it is a synthesis rather than a quoted sequence:

1. **Define the metadata (§4).** Create the table, columns, window, tab, fields, menu and role access in the dictionary. Run **Create/Complete Table**, **Synchronize column**, and **Create Window, Tab and Field from Table**.
2. **Test it in the development instance.** Log in as a different role and exercise the new window — the dictionary's Step 6.
3. **Write the behaviour, if any, as a plug-in (§5.4).** In Eclipse, create a plug-in project **outside the core repository**, targeting **Equinox**, at **JavaSE-17 or later**, with **"generate an activator"** disabled.
4. **Attach the behaviour at the right extension point (§5.6).** If the behaviour is a field-level, UI-only convenience, a **callout** (annotated `@Callout`, iDempiere 9+) is right. If it must hold across REST, processes, imports and `PO.save()` — **the usual case for a real business rule** — it must be an **event handler**. Choose once, correctly, because the choice determines which entry points see the rule.
5. **Package the metadata as a 2Pack (§5.1–§5.2).** Pack Out the dictionary rows (and any seed data) into a `.zip`. Name it per the conventions — a general `2Pack_1.0.0_MyPlugin.zip`, or the incremental `2Pack_<Name>_<version>.zip` form if it will be installed by `Incremental2PackActivator`.
6. **Make the plug-in carry the 2Pack (§5.3).** Place the `.zip` in `META-INF` and set **`AdempiereActivator`** (basic) or **`Incremental2PackActivator`** (version-controlled) as the bundle activator; add `org.adempiere.plugin.utils` as a dependency. **Bump the version when either changes** — only newer versions install.
7. **Build and publish (§5.4–§5.5).** Build with Maven/Tycho (`mvn verify -Didempiere.target=…`) to produce a **p2 repository**; or export a deployable JAR from Eclipse; or — on iDempiere 14+ — publish to the **Extension Registry**. Install with `update-prd.sh` (or `compare-update-prd.sh`), or over Gogo/Felix Web Console.
8. **Move it between environments (§5.3, §7.3).** Development → test → production, with **`update.sh`** for core patches, the **p2 update** for bundles, **Automatic External Pack In** for metadata in CI/CD and Docker, and — for major versions — the **fresh-instance** approach of the Upgrade Guide (§11.1) rather than an in-place upgrade.

The shape of that path is the answer to the guide's practical question: **metadata first, behaviour second, package both, and transport them as *rows* and *bundles* respectively.** The seam between the two — step 4 — is where iDempiere customisation is won or lost.
### 5.8 The GardenWorld Reference Instance
**GardenWorld is the demonstration client shipped with iDempiere**, and it is the environment the developer documentation's worked examples run in: the callout guide says *"Log in as **GardenAdmin** to **GardenWorld**"*, and the Pack-In guide says *"make sure you're logged into the correct client (e.g., System vs. GardenWorld)"* — **✅ verified** (docs.idempiere.org). It is the closest thing iDempiere has to a shared playground: everyone's first window, everyone's first callout, is tested there.

**The project runs public test sites that carry GardenWorld, and they are the reference instances practitioners learn on — ✅ verified this pass** (idempiere.org test sites, read 2026-10-17):

| Site | Build | Runtime as recorded |
|---|---|---|
| **demo.globalqss.com** | **iDempiere 13 Orion LTS** with the **Colombia localisation and REST**, **deployed continuously** | openjdk 17, postgresql 18, Ubuntu 22.04 |
| **test.idempiere.org** | **14 dev build** from `master` | openjdk 17, postgresql 18 |
| **test-oracle.idempiere.org** | **14 dev build** on Oracle | oracle jdk 17, **Oracle 18c XE**, Debian 9 |

Three things about that table are worth holding. It maintains both a **stable site (13 LTS)** and a **development site (14 from `master`)**; the Oracle site exists because one of the two supported databases is not the default. The runtime is **ahead of some documented floors** (PostgreSQL 18 on the demo against a documented PostgreSQL 17 and a Features-page floor of ≥10 — §3.3). And **the project warns that the test sites are deliberately unsecured**, hold **no sensitive data**, and that an **advanced role is effectively operating-system access** — **✅ verified** (idempiere.org test sites). **Take that literally:** the Gogo console on **12612** (§5.5) and the Felix Web Console on **8443** can install bundles, and an advanced role can reach both — a demonstration environment and nothing else, never a place for real data, never a template for a production security posture. §9.2 and §11.3 carry the production-side of this warning.

---
## 6. The Developer Toolchain
### 6.1 The IDE and Build Path
**The project's development toolchain is Eclipse + Maven/Tycho + Git + Equinox**, stated in the documentation and corroborated by Wikipedia — **✅ verified** (Wikipedia *iDempiere*, Development: *"The development environment (IDE) used for iDempiere is **Eclipse** with **maven/Tycho**, Git, **Equinox** (OSGi) and **Apache Felix Web Console**"*).

The documented build-and-workspace path — **✅ verified** (docs.idempiere.org, *Setting up Eclipse*):

1. **Materialise the target platform and build** from the clone root — `cd $HOME/sources/idempiere && ./mvnw verify`. The documentation notes this **downloads all libraries on the Bundle-ClassPath, builds all projects and outputs binaries to `org.idempiere.p2/target/products`**, and warns it **needs around 5 GB of free disk space**.
2. **Point an Eclipse workspace at the clone**, turn **Build automatically off**, set **Text file encoding → UTF-8** and **New text file line delimiter → Unix**; add the JRE (`java-17-openjdk`) and set **Compiler compliance level to 17**; then import via **File > Import > Maven > Existing Maven Projects**.
3. **Set the target platform** from `org.idempiere.p2.targetplatform.mirror.target` (bundled libraries) or `org.idempiere.p2.targetplatform.target` (downloaded from a remote URL — slower, and consuming roughly **287 MB more** disk), then **Set as Active Target Platform**, re-enable **Build automatically**, and **close the target editor** when done (keeping it open "may cause erratic behavior in Eclipse").
4. **Two version notes from the same page:** the **minimum Eclipse is 2024-03** (specifically *Eclipse IDE for Enterprise Java and Web Developers*); and if you use *Eclipse IDE for Java Developers* instead you will hit compile problems until you install the **m2e PDE** plugin — **✅**.

Two version notes from the same page: the **minimum Eclipse is 2024-03** (specifically *Eclipse IDE for Enterprise Java and Web Developers*), and if you use *Eclipse IDE for Java Developers* instead you will hit compile problems until you install the **m2e PDE** plugin — **✅**.
### 6.2 Source Access and Branching
**The source is cloned from GitHub — `github.com/idempiere/idempiere`** (**✅**, docs.idempiere.org, *Download the Code*):

```bash
git clone https://github.com/idempiere/idempiere.git
```

The documentation's constraints (**✅**): the clone needs **at least 3 GB**, and **the folder name must not contain spaces**. **Branch selection is the release selection** — **✅**:

- **`master`** — the development version, "a.k.a **14**";
- **`release-13`** — iDempiere **13**.

Keeping current is a plain `git pull`. *That the branch names carry the whole version story is worth noting:* `master` is already labelled as 14 while 13 "Orion" is the current stable, so the repository's default branch is one major version ahead of the recommended release (§2.5, §5.8). A developer reading `master` is reading the next release, not the current one.
### 6.3 The Prerequisites, With Versions and Dates
**The documented prerequisites, and their vintage, matter — this is the release-13 documentation set — ✅ verified** (docs.idempiere.org, *Install Development Prerequisites*, read 2026-10-17; the page states its prerequisites "apply both for release 13 and for development"):

| Prerequisite | Documented value | Notes |
|---|---|---|
| **Operating system** | **Ubuntu 24.04** 64-bit | The guide is written for Ubuntu; other systems (including Windows) are possible with corresponding packages |
| **Database** | **PostgreSQL 17** | "you can use any postgres **>=14**"; **Oracle 23ai and later** also supported |
| **Version control** | the **git client** | On Windows, the git-bash command runs the same scripts |
| **Java** | **OpenJDK 17** | Install via `apt-get install openjdk-17-jdk` |
| **IDE** | **Eclipse IDE for Enterprise Java and Web Developers 2025-12** | "older versions are also supported"; the Eclipse setup page sets the minimum at **2024-03** |

A **security detail the documentation calls out explicitly**, and worth repeating because it is the kind of thing that gets hand-waved: the guide instructs changing `/etc/postgresql/17/main/pg_hba.conf` from **`peer`** to **`scram-sha-256`** for local connections, and warns that *"Some guides suggest configuring `trust` instead… but that creates a security issue on your postgres"* — **✅ verified**. The warning is the project telling you, unprompted, not to do the convenient unsafe thing.

The documentation set also includes pages not read in full this pass but verified to exist as page titles — **✅ as titles**: *installing prerequisites*, *setting up Eclipse*, *configuring iDempiere within Eclipse*, *running iDempiere within Eclipse*, *building iDempiere without Eclipse*, *importing the database seed manually*, and *applying additional migration scripts*, plus a **"Fast Linux Development Setup" maintained by Heng Sin** as an alternative — **✅** (docs.idempiere.org sitemap and the prerequisite page's cross-links). The "Fast Linux" page and the "without Eclipse" page are the two escape hatches for developers who do not want the full Eclipse rig.
### 6.4 Testing and CI
**All continuous integration and automated testing are managed with Jenkins CI**, at **jenkins.idempiere.org** — **✅ verified** (GitHub README, verbatim: *"All continuous integration and automated testing are managed with **Jenkins CI**"*). The repository carries the machinery in plain sight — **✅** (repo listing): the **`org.idempiere.test`** and **`org.idempiere.test-feature`** modules, an **`event.test`** module (a test harness for the event system of §5.6), and a **`coverage-report`** module. The project also documents a **`RUN_SyncDBDev.sh`** development script and a **`RUN_ImportIdempiereDev.sh`** seed-import script at the repository root — **✅** (repo listing) — so a developer's first database is reproducible from the repository.

Two honest limits. First, the *content* of the Jenkins pipelines was not read this pass — the claim here is that the project states Jenkins runs its CI, corroborated by the repository's test modules, not a description of what each job does. Second, **[osgi_guide.md](osgi_guide.md) owns the OSGi test-harness idioms**; this guide records only that iDempiere's tests are bundle-hosted and Jenkins-run. For a **dictionary-driven** system, the deeper testing question — *how do you unit-test configuration that is rows rather than code?* — has no compiler and no test runner behind it, and that is the subject of §11.4.

---
## 7. The Deployment and the Operating Shape
### 7.1 The Three Installation Paths
**The project documents three ways to install, and it orders them for you — ✅ verified** (docs.idempiere.org introduction page and the installation guide; idempiere.org download page):

1. **Docker — "the easiest way".** The Docker page documents an **`idempiereofficial/idempiere:12-release`** image with a **PostgreSQL 16 container** and a **docker-compose stack exposing ports 8080, 8443 and 12612**, plus a separate **`idempiere-docker` repository** for advanced topics — **✅**. *(Note the image tag names 12 while the current stable is 13 — a documentation-vintage detail recorded as read.)* **2.** An **installation script for Ubuntu 24.04** — a scripted setup for the project's own documented OS. **3. Manual steps** — the full manual path, which the "Install Development Prerequisites" page of §6.3 expands.

Alongside those, **installers for Linux, Windows and macOS, plus a Debian `.deb`, are published on SourceForge**, and a **community development-environment setup script is linked from the project's download page** — **✅ verified** (idempiere.org/releases — the archive lists Linux, Debian, Windows and macOS artefacts per version). **The Docker route being offered first is a deliberate signal**: the project's own "Why iDempiere" list opens with *"It is easy to install"* and *"It runs on a small server"* — **✅** (idempiere.org introduction page) — and the container path is how it makes that true for a first-time evaluator.
### 7.2 The Runtime the Server Runs On
**The production shape is a Java server on a Jetty container, hosting the OSGi runtime, behind three ports — ✅ verified** (Wikipedia *Platform*; docs.idempiere.org; the Docker stack's exposed ports):

| Port | Service | Source |
|---|---|---|
| **8080** | The web UI (HTTP) | Docker stack — **✅** |
| **8443** | HTTPS, including the **Apache Felix Web Console** at `/osgi/system/console/bundles` | docs.idempiere.org; Docker stack — **✅** |
| **12612** | The **Apache Felix Gogo** telnet console | docs.idempiere.org; Docker stack; Upgrade Guide — **✅** |

The operating prerequisites are the ones §6.3 lists: **Ubuntu 24.04 64-bit, PostgreSQL 17 (or any `>=14`), OpenJDK 17** as the documented release-13 target, with **Oracle 23ai and later** supported — **✅**. **The server needs no desktop**: the client is a browser, and the server is Java-on-Jetty; the Swing client that once existed in the 2.1 era (the releases archive still lists "Download for Swing client" under the *Daily v2.1* section) is gone from the modern distribution — **✅** (idempiere.org/releases). That is a genuine simplification and it is fully documented: iDempiere today is a web application.

**Two operational cautions, both the project's own.** First, **Docker and CI/CD deployments are the documented use case for Automatic External Pack In** (§5.3) — the folder-scan feature exists precisely for containerised, zero-touch configuration. Second, **the Gogo console on 12612 and the Felix Web Console on 8443 are bundle-installation surfaces** (§5.5): anything that can reach them can install code, so a production deployment's network policy is the control that matters, not the console's convenience. The project's own warning about the unsecured public test sites (§5.8) — "an advanced role is effectively operating-system access" — is the same point from the other direction.
### 7.3 The p2 Update Mechanism
**iDempiere's update mechanism is the Eclipse p2 provisioning system, driven by shell scripts — ✅ verified** (docs.idempiere.org, upgrade and distribution guides):

- **`./update.sh`** — *"downloads and applies the latest patches from the official repository"* for the core system.
- **`update-prd.sh <repository_url> <feature_id>`** — installs or updates a plug-in from a **p2 repository** (§5.5).
- **`compare-update-prd.sh`** — the **state-checking** variant that avoids needless uninstall/reinstall (§5.5).
- **Non-p2 plug-ins** are applied over the **telnet console on 12612** (§5.5).

The mechanism's shape follows directly from the architecture: because iDempiere *is* an OSGi application built by **Maven/Tycho into a p2 repository** (§3.6, §6.1), extending it is a matter of pointing the standard p2 provisioning at another p2 repository. **The build output and the update input are the same artefact type** — a p2 repository in, a p2 repository out — which is why plug-in distribution needed no bespoke installer. The repository even carries `org.idempiere.equinox.p2.director-feature` and `org.idempiere.p2.profile` to drive it — **✅** (repo listing).
### 7.4 Multi-Tenant and the Cluster Picture
**What the project documents about running multiple tenants is in §3.4, and it is a data-model fact, not a deployment-mode fact**: every dictionary and business row carries `AD_Client_ID` and `AD_Org_ID`, and the documented capabilities are per-tenant administration (tenant admin roles, tenant-at-login, per-tenant SMTP, a per-tenant cron time zone, and copy/move a tenant) — **✅ as documentation-page titles** (docs.idempiere.org sitemap; v6.2/v9/v10/v12 new-features pages).

**On clustering, this guide does not invent capabilities the project did not document this pass.** What *can* be stated from verified evidence, and no more:

- The repository contains **`org.idempiere.hazelcast.service`** and **`org.idempiere.redis.service`** bundles (plus their `-feature` companions) — **✅ verified** (GitHub repository root listing, read 2026-10-17). A distributed cache and a Redis service bundle are present in the build.
- **What those bundles implement, and whether the project documents a supported cluster topology, could not be verified this pass** — the relevant documentation pages were not among the sources read, and **this guide will not guess at a failover model, a session-replication scheme, or a supported node count.** That is a deliberate omission: **⚠ not verified this pass**, with the honest note that the presence of Hazelcast and Redis service bundles is *evidence that a distributed-runtime design exists*, not evidence of what it does or of how it is supported. A reader planning a clustered deployment must read the project's current clustering documentation directly rather than rely on this guide.

---
## 8. The Comparison
*Structural and unranked throughout. This section names no vendor's superiority, prints no figure, and reproduces no ranking. Where it compares, it compares **shapes** — how a system is built, governed, licensed and extended — not performance or popularity.*
### 8.1 Against Its Own Parent Line — Compiere and ADempiere
iDempiere's nearest relatives are its own ancestors, and comparing it against them is comparing it against **living** projects, not a dead line.

| Dimension | Compiere | ADempiere | iDempiere |
|---|---|---|---|
| **Origin** | 1999, by Jorg Janke — **✅** | **September 2006 fork of Compiere**, after community/sponsor disagreement — **✅** | **2011**, the OSGi code path named after ADempiere-community disagreements — **✅** |
| **Licence** | Wikipedia infobox (3.3, 2009): *"GPL or Proprietary"* — **✅** | **GPLv2** (Wikipedia infobox) — **✅** | **GPL-2.0-or-later** (project's own source-code page) — **✅** |
| **Commercial layer** | **Yes** — 2006 venture funding (New Enterprise Associates); Professional/Enterprise/Cloud editions beside a Community edition; acquired by Consona in June 2010 — **✅** | **No** — community-governed; council-of-contributors model — **✅** | **No** — no open-core split, no enterprise edition, no per-user licence — **✅** |
| **Architecture** | The Compiere data dictionary (the ancestor of everything here) — **✅** | **Java EE, Tomcat and JBoss**; database support **restricted to PostgreSQL and Oracle**; the dictionary extended into the application; a **WfMC/OMG** workflow engine — **✅** | **Java + OSGi + Jetty + ZK**; the same two databases; the dictionary plus a plug-in model — **✅** |
| **Modularity** | Monolithic application — **analysis** | Monolithic application — **analysis** | **Bundles from the ground up** — the plug-in model is the reason the 2011 code path exists — **✅** |
| **Latest as of this pass** | Wikipedia's last listed stable: **3.3, 24 March 2009** — **✅** | Wikipedia's listed stable: **3.9.4, January 2023** — **✅**; repository default branch `develop`, **16,989 commits**, GPL-2.0 — **✅** | Current stable **13 "Orion"**; preview **14**; repository `master`, **16,318 commits** — **✅** |

**The structural reading, unstained by ranking:**

- **Compiere is the ancestor that took the commercial path.** It raised venture capital, split into editions and was acquired. Every fork question — ADempiere's, iDempiere's — descends from that choice.
- **ADempiere is the parent that kept the dictionary and stayed community-governed.** iDempiere inherited *both*: the dictionary (§4.6 traces it to ADempiere's Architecture section explicitly) and the governance model (the council-of-contributors structure of §2.2).
- **iDempiere is the child that changed the *platform*, not the *idea*.** The big difference at iDempiere 1.0 versus ADempiere, per Wikipedia, was **"the platform technology upgrade to the OSGi framework that allows iDempiere to have plug-ins"**, and *"a lot of specific Code made on Adempiere have been updated as plug-ins in iDempiere"* — **✅ verified**. Read that carefully: the *application logic* survived the re-architecture by becoming plug-ins. The dictionary stayed; the housing changed; the extension mechanism went from "modify the monolith" to "attach a bundle".

So the three form a clean sequence of *decisions*, each visible in the code: Compiere chose a commercial edition; ADempiere rejected that and kept the dictionary; iDempiere kept the dictionary and rebuilt the housing around a plug-in model. **And ADempiere is not a museum piece** — Wikipedia lists its last stable as January 2023 and its repository shows active commits — which is why this is a comparison between siblings, not between a system and its fossil.
### 8.2 Against the Sibling Odoo
**The full comparison lives in [odoo_framework_guide.md](odoo_framework_guide.md) §7, and this guide does not repeat it.** What belongs *here*, in the guide that completes the pair, is only the **structural contrast** the pairing makes visible. Four differences in kind, each of which the sibling documents on its own side and this guide documents on iDempiere's:

| Dimension | Odoo (see [odoo_framework_guide.md](odoo_framework_guide.md)) | iDempiere (this guide) |
|---|---|---|
| **Licence model** | **Open-core**: Community LGPLv3 + proprietary Enterprise apps and support (sibling §1.3, §4.1) — **✅** | **Single GPL-2.0-or-later tree**, no open-core, no enterprise edition, no per-user licence (§2.7) — **✅** |
| **Where metadata lives** | **In code**: Python model classes + XML views, versioned in a module with a `__manifest__.py` (sibling §3.2–§3.3) — **✅** | **In data**: dictionary rows in the same database, editable at runtime through the application itself (§4.6) — **✅** |
| **The extension unit** | The **module/addon** — a directory with manifest, Python, XML, security files (sibling §3.3) — **✅** | Three units: **dictionary entry**, **2Pack `.zip`**, **OSGi plug-in bundle** (§5) — **✅** |
| **The web client** | A bespoke TypeScript library — **OWL** (sibling §3.4) — **✅** | A server-side **ZK** application (§3.2) — **✅/⚠** |
| **Steward** | A company at the centre (Odoo S.A.) + a non-profit community (the OCA) (sibling §5.3) — **✅** | A community only — "no single person or company ruling the project" (§2.7) — **✅** |
| **Language & weight** | Python + PostgreSQL (sibling §3.5) — **✅** | Java + OSGi, PostgreSQL or Oracle (§3.1) — **✅** |

**The single sentence that carries the contrast**, and the reason the two guides are a pair: **Odoo's metadata is code that the framework interprets; iDempiere's metadata is data that the application reads.** In Odoo, changing a model means editing a Python file in a module and upgrading — the change travels through version control and a deployment. In iDempiere, changing a table's shape means editing rows through a window, and the same change travels through a 2Pack as a `.zip` of rows. Neither is "better": one puts metadata under a compiler and a review gate; the other lets a functional consultant change the model in a running system. The consequence each project accepted is the mirror of the other's — and §11.4 is where the "metadata as data" consequence is examined without flinching.

**Everything else about the comparison — the Tryton head-to-head, the SAP Business One framing, the decision tree — is the sibling's, at [odoo_framework_guide.md](odoo_framework_guide.md) §7.1–§7.9.** This guide cites it and stops.
### 8.3 Against the Wider Open-Source ERP Field
**Structurally, and with no ranking.** The wider open-source ERP field clusters along two axes this guide can describe without printing a single figure: **what the system's centre of gravity is**, and **who steers it**.

- **The Compiere-descended cluster.** Wikipedia's "See also" on the iDempiere article names **metasfresh** as being *"in the Compiere source code family"* — **✅ verified** — and the same family obviously contains **Compiere → ADempiere → iDempiere** (§8.1). These systems share the **data dictionary** at their centre and the **Java** tradition in various degrees; what distinguishes iDempiere within the cluster is the OSGi plug-in housing (§8.1).
- **The Python-descended cluster** — Odoo and its relatives, whose centre of gravity is the **ORM and the app ecosystem** ([odoo_framework_guide.md](odoo_framework_guide.md) §3, §7).
- **The licence axis cuts across both.** Within the Compiere family: **Compiere** carried a *"GPL or Proprietary"* dual answer and then a commercial layer; **ADempiere** and **iDempiere** are pure **GPL**; and in the Python family, **Odoo** took the **open-core** route while its 2008 fork **Tryton** stayed **GPL-3.0-or-later** and federation-governed ([odoo_framework_guide.md](odoo_framework_guide.md) §7.3). **The pattern worth noticing is not "open versus closed" but the split *within* each family**: the same codebase that forked over governance also forked over licence posture, twice, in two different languages.
- **The governance axis.** The field contains **company-centred** projects (a vendor ships the core and sells the edges), **foundation-or-federation-governed** projects (a non-profit or a federation of companies), and **community-only** projects. **iDempiere is squarely in the last category** (§2.7), and its 2Pack/plug-in split is the *technical* expression of a governance posture: configuration and extensions are meant to be things the community exchanges, not things a vendor controls.

**What this guide will not do:** name which cluster is superior, print which project is "leading", reproduce any listicle's ordering, or compare any two systems on speed, cost or scale. The listicle placements that exist for iDempiere (PCQuest, Capterra, a Russian Computer Review piece — §2.8) are recorded as *existing* and are explicitly **not** treated as evidence of position. **Structural comparison only; no ranking.** *(For the open-source-vs-commercial framing of this field — the genre the two ERP guides share — see [odoo_framework_guide.md](odoo_framework_guide.md) §7 and [../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md).)*

---
## 9. The ERP-in-Banking Context
*Following the sibling's §8 framing ([odoo_framework_guide.md](odoo_framework_guide.md) §8), cross-referenced rather than re-derived.*
### 9.1 The ERP in a Bank — Back-Office, Not the Ledger
**The honest first principle: an ERP is not a core-banking system.** The repository's banking series is explicit about what core banking is — the transaction ledger, the product catalogue, the customer accounts, the regulatory reporting — in systems like the commercial core-banking suite in [../banking/t24_programming_guide.md](../banking/t24_programming_guide.md) and the open-source microfinance core in [../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md). **iDempiere is not in that family, and the guide says so before saying anything else.** Like the sibling ERP, iDempiere's place in a bank is **the back office**: the finance, HR, procurement and operations layer that runs *around* the core, not the ledger itself.

**What a bank actually runs an ERP for** — the bank's own general ledger (AP/AR, fixed assets, expense management, intercompany postings, management accounting, statutory packages), HR and payroll-adjacent processes, procurement and vendor management, operations support, and the integration boundary to the core-banking estate — is the sibling's list, **cross-referenced rather than re-derived** ([odoo_framework_guide.md](odoo_framework_guide.md) §8.1); **analysis grounded in the banking series, flagged as such**.

**Where iDempiere differs from the sibling ERP in this context is two structural features, both verified, and neither of them a claim about capability:**

1. **Multiple account schemas.** iDempiere advertises support for **multiple entities (sets of books) inside a single login** — **✅ verified** (idempiere.org introduction page; Wikipedia's Overview lists "multiple account schemas"). For a bank group with several legal entities and reporting bases, this is a *data-model* answer to a back-office requirement — books are a first-class concept, not a per-entity installation.
2. **The dictionary as a configuration medium.** Because finance and procurement rules in a bank are the exact kind of thing that changes and must be *reviewable*, the dictionary/2Pack split (§5) gives a bank a documentable path: metadata changes move as **rows in a versionable `.zip`**, and behaviour moves as **bundles in a p2 repository** — two artefact types, two review gates. §12 turns this into a design.

**The strategic angle from the series' own finding** — recorded in the sibling's §8 and in [../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md) — is that open source in financial services dominates in **infrastructure** and survives in applications as long-tail and niche. **iDempiere is an open-source *application* in a *back-office* domain**, which is precisely the domain where a bank has no regulatory need to be proprietary and every commercial reason to be cheap and inspectable. That is the whole case, and it rests on structure, not on any figure.
### 9.2 The Banking Fit Table

| Bank domain | iDempiere fit | Where it does **NOT** fit | Marker |
|---|---|---|---|
| **Back-office finance** (the bank's own GL, AP/AR, fixed assets, expenses) | Strong fit — core ERP territory; **multiple account schemas** supported | **Customer** accounts, deposits, loans — that is the core-banking ledger | **✅/⚠ analysis + series cross-ref** |
| **Multi-entity books** | Strong fit — multiple entities (sets of books) in a single login | Statutory reporting *for the whole group* still sits in the bank's regulatory platform | **✅/⚠** |
| **HR / expenses / timesheets** | Reasonable fit — ERP HR breadth; payroll stays specialised | Core HRIS/HCM at bank scale; banks rarely move that to an ERP | **✅/⚠** |
| **Procurement & vendor management** | Strong fit | Trading/supply-chain finance products (a banking line, not a back office) | **✅/⚠** |
| **Regulatory reporting** | No — a *source* of data at best | The bank's regulatory data platforms | **✅/⚠ genre boundary** |
| **Core banking / payments** | **No — out of scope** | [../banking/t24_programming_guide.md](../banking/t24_programming_guide.md), [../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md) | **✅ genre boundary** |
| **The 12612 / 8443 console surfaces** | **A deployment risk, not a fit** | A production bank network must restrict bundle-installation ports; the project's own test-site warning (§5.8) applies with force here | **✅** |
| **Vendor escalation** | **A genuine gap** — there is no vendor to escalate to (§11.3) | A bank's production support model typically expects an accountable vendor; iDempiere offers a community instead | **✅/⚠** |
| **Clustered / HA topology** | **Not verified this pass** (§7.4) | This guide will not claim a failover model, session-replication scheme or supported node count | **⚠ not verified** |
| **Long-lifetime deployment risk** | Dictionary-driven upgrades are hard (§11.1) — the project documents a fresh-instance rule for majors | A bank that cannot absorb a fresh-instance major upgrade every few years should not choose a dictionary-driven ERP for a system of record | **✅/⚠ analysis** |

### 9.3 Cross-References to the Bank Series
- **[../banking/t24_programming_guide.md](../banking/t24_programming_guide.md)** and **[../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md)** — the core-banking boundary §9.1 draws: the ledger is T24-class (or Fineract-class), not an ERP.
- **[../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md)** — the open-source-vs-commercial survey and its structural finding, the frame for §9.1 (the genre itself is the sibling's territory, [odoo_framework_guide.md](odoo_framework_guide.md) §8). **[../management/business_case_development_guide.md](../management/business_case_development_guide.md)** — the TCO/business-case lens for §12: with **no licence fee and no per-user fee at all** (§2.7), the TCO reduces to infrastructure, implementation, integration and — critically — the community-support and upgrade-labour lines that §11 prices honestly.

---
## 10. Where the Dictionary Metaphor Breaks
*The thesis stated in §1.2 is that the model, the screen and the database table are three renderings of one dictionary entry. §4 showed it holding and then sharpened it: one entry renders a table, a screen and an access rule, and carries hook points to which code may attach. This section finds the exact place the metaphor fails, and explains why it fails there.*
### 10.1 The Callout Scope Limit — the Best Documented Example
**The single best-documented place where the dictionary illusion breaks is the callout scope limit**, and it breaks because *a piece of dictionary-attached behaviour has an entry-point condition the dictionary cannot express.* The documentation says it without hedging — **✅ verified, verbatim** (docs.idempiere.org, *Developing Plug-in Callouts*):

> **Callouts only work when data is entered through: the iDempiere UI; the CSV Importer.**
> Callouts do **NOT** execute when records are created or modified via: **REST API calls; background processes; direct database operations; Java code using `PO.save()`.**

Read that against §4.6. **The dictionary is rows in the same database as the business data.** A business record created through the UI and the *same* business record created through the REST API are, as far as the dictionary is concerned, **the same row** — same table, same columns, same references, same tenant scoping. The dictionary cannot tell them apart, because the dictionary knows *what the data is*, not *how it arrived*. **A callout, however, does distinguish them — and it is attached to a field, which is a dictionary thing.** So the dictionary carries an attachment whose behaviour depends on a fact (the entry point) that the dictionary does not model. **That is the seam.** The metadata is complete as a description of the data and incomplete as a description of the behaviour attached to it.

**The documented remedy is not a patch on the dictionary — it is a move out of it.** The documentation's own instruction: *"If you need validation or logic that works across all entry points, use an **Event Handler** instead, which intercepts all record changes regardless of the source."* And the comparison table draws the line exactly (§5.6):

So the correct reading of the thesis, forced by this limit, is: **the dictionary renders *structure* at every entry point, but it can only *host behaviour* that fires on the UI path.** Any behaviour that must hold wherever the data comes from has to leave the dictionary and live in a bundle (§5.6) — the code layer. **The metaphor breaks exactly at the boundary between "what the data is" and "what may happen to the data, and from where".** The dictionary owns the first; code owns the second; and the seam between them is where a callout lives and dies.
### 10.2 What the Limit Means for Upgrades
The callout limit is not an isolated wart — it is the visible tip of the upgrade consequence, and the project documents the consequence at length (§11.1). Three connections:

- **Behaviour that lives in code is behaviour that must be recompiled at every major version.** The v13 technical notes record that **classes and methods deprecated for more than two major releases were removed** (IDEMPIERE-6735), so *"custom plug-ins and customizations that still call these classes and methods will no longer compile and must be updated"* — **✅ verified** (docs.idempiere.org, v13 technical migration notes). The event handler you *chose* because a callout could not do the job is, by construction, code — and code carries the recompile burden the dictionary was supposed to spare you.
- **The dictionary half of the upgrade is documented as *harder*, not easier.** Dictionary-driven systems have a specific upgrade failure mode — *"Duplicate Column Errors"* when custom columns collide with new core columns, *"Duplicate ID Conflicts"* when custom dictionary entries use IDs now claimed by core, and *"View Overwrites"* when core updates replace custom views — all documented with their remedies (**Move ID/UUID process; rename with a custom prefix; define views in the Application Dictionary rather than as raw SQL**) — **✅ verified** (Upgrade Guide). **A dictionary that holds customisations is a dictionary that can collide with core.**
- **The migration itself is documented in numeric-jump detail.** The Upgrade Guide's tip is explicit: jumping across multiple majors means reviewing migration notes **in sequence (for example, 10 → 11 → 12), not only the final target**, because *"Skipping migration notes is one of the most common causes of post-upgrade regressions"* — **✅ verified**.

**The consequence, stated plainly:** the "no Java code required" promise holds at the *first* release of a customisation and weakens at every upgrade after it — because the parts that stayed metadata must be *migrated*, and the parts that became code must be *recompiled*, and the seam between them (§10.1) means there is always some of each.
### 10.3 What the Limit Means for Testing
The callout limit has a testing consequence that is worse than the upgrade one, because it is invisible until production:

- **A callout is tested on the UI path and skipped everywhere else.** If you write a business rule as a callout and test it by clicking through the window — the *only* path a callout fires on — the test passes. Then a nightly integration job posts the same data through the REST API or a direct `PO.save()`, the callout does not run, and the rule silently does not apply. **The rule existed, was tested, was green, and never fired.** That is the exact failure mode the documentation warns about, and it is a *test-coverage* failure masquerading as a *code* failure: the test suite tested the UI path and the production path was a different path.
- **The dictionary has no compiler and no test runner behind it.** §6.4 records the project's Jenkins CI and its test modules — `org.idempiere.test`, `event.test`, `coverage-report` — and those test *code*. But a dictionary change (a new column, a changed reference, a new mandatory flag) has **no unit test, no type check, and no build step** that would catch it: it is applied to a database, and the first thing to notice a mistake is the next request. §11.4 develops this.
- **The remedy the documentation implies is a discipline, not a tool.** Choosing an **event handler** over a callout because the rule must hold everywhere (§5.6) is the testing decision: it moves the rule to the execution point that *all* entry points share, so *one* test path covers *all* entry points. **The extension-point choice is a testability choice.**
### 10.4 Why the Illusion Fails Exactly There
Step back, because the failure is precise and worth naming exactly.

The dictionary is a **declarative** layer: it says *what exists* — tables, columns, references, windows, fields, menus, access. Declarative layers are wonderfully complete at describing structure, because structure is exactly what a declaration can carry. The dictionary even carries **hook point declarations** — an `AD_Column` row can say that a callout is attached, and the framework will invoke it. **That part works.**

What the dictionary *cannot* declare is a **condition on the invocation environment** — "run this only when the change came through the UI". **That is a runtime fact about the request, not a structural fact about the data**, and the dictionary's vocabulary is structure. So the dictionary's attachment is unconditional from its own point of view, and the *behaviour* is conditional from the code's — and the two conditions are expressed in different worlds. The behaviour that could have been a declaration ("when this field changes, validate it") is only *sound* if it is a declaration in a world where all changes are field changes. iDempiere's world has five entry points (UI, CSV importer, REST, processes, direct persistence), and the declaration only fires on two of them.

**So the illusion fails exactly where a dictionary-attached behaviour needs to know *how* the data arrived, and the dictionary cannot tell it.** Everything before that point — the model, the screen, the table, the access — is genuinely three renderings of one entry, and the thesis stands. Everything after that point is code, and the code layer is where the ERP actually runs. **The dictionary is not the application; it is the part of the application that can be written down.** The two layers meet at a seam, the seam is a named extension point, and the craft of iDempiere development is knowing which side of the seam each piece of behaviour belongs on (§5.7, step 4).

---
## 11. The Failure Modes
### 11.1 Upgrade Pain in a Dictionary-Driven System
**iDempiere's upgrade discipline is documented in unusual detail, and the detail is itself the warning** — **✅ verified** (docs.idempiere.org, *Best Practices for Migrating to New iDempiere Versions*, based on Carlos Ruiz's conference presentation; read via a Wayback capture dated **2026-06-10**).

**The fresh-instance rule for major versions is the headline.** The documentation states that **major version migrations "usually follow a fresh installation approach"**, and gives **two reasons not to upgrade in place** — **✅**:

> * **If the process goes badly, you can lose your current production instance.**
> * **Each new iDempiere release can introduce newer technology requirements (Java, PostgreSQL, and related dependencies)** — and a new instance gives you multiple attempts to identify and solve issues before users are impacted.

That is the project itself telling you that the *major-version* path is a new instance plus a database migration, not a patch-in-place — and the second reason is the one §2.5 and §3.3 keep illustrating: the Java and PostgreSQL floors move between releases, so an in-place major upgrade would have to move the runtime underneath a live instance.

**The documented procedure, in the order the guide gives it — ✅:**

1. **Prepare:** read the **release notes for the target version**, the **migration notes for every version jump** (review them **in sequence**, e.g. 10 → 11 → 12), and the **installation guide** for the target.
2. **Assess:** build a **system inventory** — baseline version and build number; all custom source code; database customisations (tables, columns, indexes, constraints); plug-in dependencies and exact versions; integration points. **Keep the inventory in version control** — *"it will be your checklist during each migration iteration"*.
3. **Back up:** a **full PostgreSQL dump** with schema and data; a **snapshot of the server folder** including configuration files; the **custom code repository committed and tagged**; and a **documentation snapshot**. Again the honesty: *"You will likely iterate through the migration process multiple times. Reliable restore points are essential for safe retries."*
4. **Minor updates:** test on **a staging server that mirrors production** (same database version and configuration, identical customisations and plug-ins, representative data, similar load); run **`./update.sh`**; **update plug-ins from a p2 repository** (`update-prd.sh`, or `compare-update-prd.sh`); install non-p2 plug-ins **over the telnet console**; then run the **Post Migration Helper** process.
5. **The Post Migration Helper — the dictionary-specific upgrade tool.** It identifies **customisations overwritten by the update**, **field-order changes in windows and tabs**, **changelog entries affecting custom components**, and **issues needing manual review** — **✅**. *This tool exists because a dictionary-driven system has exactly those failure modes and no compiler to catch them.*
6. **Verify:** run the **Verify Migration** process, whose checks are **changed customisations, overwritten views, field-order changes, and missing references** — **✅**.
7. **Then the rest:** performance tuning (slow-query logs, index review, PostgreSQL version, cache configuration), business-process testing (order-to-cash, procure-to-pay, inventory, financial reporting, custom workflows), training, and troubleshooting.

**And the failure modes are pre-catalogued — ✅**, from the same guide: **migration-script failures** (the script stops on an `ERROR`; resolve, re-run, repeat until clean, "otherwise only a subset of scripts will be applied"), **performance degradation**, **custom-code compatibility** (plug-ins fail to load — recompile against new libraries), **duplicate column** and **duplicate ID** conflicts with documented remedies, and **UI rendering issues** from theme changes.

**The single most useful operational fact in this whole guide:** *the migration scripts stop at the first error.* `./RUN_SyncDB.sh` **stops when it encounters an `ERROR`**, pending changes must be resolved manually or by custom scripts, and you **re-run until it finishes without errors** — **✅**. A migration is therefore an *iterative* process by design, and any plan that assumes one pass is wrong.
### 11.2 Extensions That Bypass the Dictionary
The second failure mode is the inverse: a customisation that **looks** like an extension but has escaped the dictionary's reach, and is consequently invisible to every dictionary-aware tool the project provides.

**What "bypassing the dictionary" looks like, and why the tools miss it:**

- **Raw SQL views instead of dictionary-defined views.** The Upgrade Guide names this one directly: to prevent core updates from overwriting custom views, *"Define custom views in Application Dictionary instead of managing them only as raw SQL"* — **✅**. A view defined only in SQL is not in the dictionary, so the Post Migration Helper's "overwritten views" check and the dictionary's own model do not know it exists — until an update replaces it.
- **Custom columns without a custom prefix.** The remedy for duplicate-column collisions is to **rename custom columns using a custom prefix (for example, `LIT_User1_ID`)** and the general rule is to **prefix custom objects (`XX_`, `LIT_`, `ORG_`)** — **✅**. A custom column named as though it were core is a column that will collide on the next upgrade, and its collision will be a *migration script failure* (§11.1).
- **Data written by direct database operations.** A record inserted by a direct database operation bypasses not only callouts (§10.1) but also the model layer, the event system, validation, and — depending on the path — the audit trail. The dictionary knows the row exists afterward; it never saw it arrive.
- **Customisations contributed to a fork instead of to the core or a plug-in.** The Upgrade Guide's "Contributing to Core" section recommends contributing valuable customisations back, on the stated grounds that it **"reduces future migration costs by eliminating long-lived custom code"** — **✅**. A customisation that lives in neither a plug-in nor the core is one that must be re-applied by hand at every upgrade.

**The correction is uniform and it is the guide's thesis applied to maintenance:** keep the *structure* in the dictionary, keep the *behaviour* in a bundle, and keep *every* custom object **named, prefixed, inventoried and version-controlled** so the dictionary-aware tools (Post Migration Helper, Verify Migration, the ID/UUID migration utilities) can see it. **An extension the dictionary cannot see is an extension the project's own upgrade tools cannot protect.**
### 11.3 A Community Project's Support Reality
**The uncomfortable section, stated honestly.** iDempiere has **no vendor to escalate to.** That is not a criticism and it is not praise — it is the direct consequence of §2.7: there is no company at the centre, no commercial edition, no support contract, and therefore no accountable party.

**What the community *does* provide, and where it lives — ✅ verified this pass** (GitHub README; the project's own links):

- **A JIRA tracker** at **idempiere.atlassian.net** for bugs and feature requests — and the README's own instruction is *"Before opening an issue, please discuss your idea or problem in our forums or Mattermost channels. Once confirmed, submit your ticket to our JIRA Tracker"* — **✅**.
- **Jenkins CI** at **jenkins.idempiere.org** for all continuous integration and automated testing — **✅**.
- **Mattermost** at **mattermost.idempiere.org**, a **Google Groups forum**, a Docusaurus **documentation** site at **docs.idempiere.org** (its sitemap lists **540 documentation pages** — **✅**) and a multi-language **wiki** at **wiki.idempiere.org** — **✅**.
- **Contribution machinery and governance:** CONTRIBUTING.md, CODE_OF_CONDUCT.md, a security policy, plugin guidelines, a git workflow guide, a pull-request review guide and a new-feature-suggestion guide; the council-of-contributors model inherited from ADempiere (§2.2); and a donations page — **✅**.

**What "community support" is, and honestly is not** — **analysis, stated so a reader can price it**:

- **It is** a large, documented, geographically spread community (§2.8) with a real issue tracker, real CI, real chat, real documentation, and a governance structure that has survived since 2006. It is *responsive in aggregate* — a question in Mattermost or the forum is likely to be seen by someone competent.
- **It is not** a support contract. There is **no SLA, no guaranteed response time, no accountable vendor, and no one contractually obliged to fix your problem.** A bug that matters to one bank is a bug that matters to whoever in the community cares. **A production incident's resolution time is a function of who is awake and interested, not of a service-level agreement.**
- **It is not** vendor escalation. There is no escalation path, because there is no vendor. That is either the point (freedom, no dependency) or the disqualifier (no accountability), depending entirely on what the adopting organisation requires — and a bank considering iDempiere for a system of record must answer that question *before* the business case ([../management/business_case_development_guide.md](../management/business_case_development_guide.md)), not after.
- **The practical mitigation** is the one the ecosystem has always used: **pay a community integrator.** Consultancies exist in the community (the documentation credits contributors from several — e.g. the prerequisite and Eclipse pages are brought to you by Carlos Ruiz of BX Service GmbH; the Maven guide by Luis Amesty of Amerpsoft; the Extension Registry worked example follows BX Service's Document Status Validator) — **✅ verified as attributions on the documentation pages**. That converts "community support" into "a commercial relationship with a community member" — which is a real, purchasable option — but it is a *third-party* relationship, unlike the sibling ERP's **vendor** subscription ([odoo_framework_guide.md](odoo_framework_guide.md) §4.4). **The honest line: with iDempiere you can buy support from someone in the community, but there is no company that owns the product to hold responsible for it.** For an organisation that needs a name on a contract, that is the deal-breaker; for one that values the absence of a vendor, it is the whole point.
### 11.4 The Assumption That Configuration Is Free
**The fourth failure mode is the subtlest, and it is the one that catches teams after go-live: the assumption that because configuration is not code, it is free.**

The reasoning is seductive and wrong. A dictionary change is not code, so it needs no developer, no compile, no release — *therefore it needs no test, no review and no change control*. Every premise is true and the conclusion is false, because **configuration has no unit tests, no compiler, and no review gate unless you build one.** §4.6 and §10.3 established why: the dictionary is rows, rows are applied to a live database, and the first thing that notices a mistake is the next request. There is no build step to fail and no type checker to object.

**What this costs, concretely, and every item is grounded in something verified above:**

- **Metadata changes are unversioned by default.** A column renamed in a window leaves no diff, no commit, no pull request — *unless* the developer routes the change through the dictionary's own transport, a **2Pack** (§5.1), and version-controls the `.zip`. The documentation's benefits list for 2Pack includes *"Works with version control and CI/CD"* — **✅** — precisely because *without* 2Pack, dictionary changes do **not** work with either.
- **Metadata changes are unreviewed by default.** The 2Pack's pull-request review is the review gate, and it exists only if you make the 2Pack the *only* path to production — a rule nothing in the system enforces.
- **Metadata changes are unrecorded by default.** The documentation's best-practice list says **"Document all changes: maintain records of customizations and rationale"** and **"Version control: keep all custom code in Git or a similar system"** — **✅** — and, tellingly, the Upgrade Guide's **system inventory** (baseline, custom source, custom **tables/columns/indexes/constraints**, plug-in versions, integration points) is *the artefact you need so that the next upgrade can see what the last one did*. `AD_ChangeLog` exists (§3.7 mentions configurable audit trails) but is not a substitute for a reviewed, versioned history.
- **The bill arrives at upgrade time.** Every unversioned, unprefixed, unreviewed dictionary change is a **migration script failure or a duplicate-ID conflict** waiting for the next major version (§11.1). The Upgrade Guide's *"One of the most common causes of post-upgrade regressions"* is skipping notes; a close second is a dictionary full of changes nobody recorded.

**The guardrail, stated as three rules:** (1) **make the 2Pack the only path to production** — the dictionary's own transport, versioned and reviewed; (2) **put a review gate on it** — a 2Pack pull request is the code review of metadata, and treating it as anything less is pretending configuration is free; (3) **maintain the inventory** (§11.1) so the upgrade tools can see every custom object. **Configuration is not free; it is merely *cheap to change* — and cheap-to-change without a gate is the most expensive habit a dictionary-driven project can form.**
### 11.5 The Failure-Mode Table

| Symptom | Cause | Guardrail | Marker |
|---|---|---|---|
| A business rule "stops working" when data arrives by integration/API | The rule was a **callout**, which fires only on the UI/CSV-importer paths | Use an **event handler** for anything that must hold across all entry points (§5.6, §10.1) | **✅** |
| Migration fails at the first script error and stops | `RUN_SyncDB.sh` **halts on the first `ERROR`**; pending changes need manual/custom fixes | Re-run after each fix until clean; expect **iterations**, keep restore points (§11.1) | **✅** |
| Customisation silently overwritten by an update | Custom views managed **only as raw SQL**; customisations not in the dictionary | Define views **in the Application Dictionary**; run the **Post Migration Helper** and **Verify Migration** (§11.1, §11.2) | **✅** |
| Custom plug-in fails to load after an upgrade | Core classes/methods deprecated **more than two majors ago were removed** (IDEMPIERE-6735) | Recompile against the new libraries; replace removed APIs (§10.2, §11.1) | **✅** |
| Custom column collides with a new core column | Unprefixed custom names; custom IDs in core ranges | Prefix custom objects (`XX_`, `LIT_`, `ORG_`); run the **Move ID/UUID** process (§11.1, §11.2) | **✅** |
| Integration/job posts data and validation is skipped | **Direct database operations** or `PO.save()` bypass callouts and the UI path | Route writes through the model layer; put rules in event handlers (§10.1, §11.2) | **✅** |
| Production incident waits on an unknown responder | There is **no vendor** — community support has no SLA, no escalation | Pre-arrange a **community integrator**; make the support model an explicit business-case line (§11.3) | **✅/⚠ analysis** |
| A dictionary change surprises the team in production | Configuration has **no compiler, no unit test, no review gate** by default | Route all changes through **2Packs under version control and review**; maintain the inventory (§11.4) | **✅/⚠ analysis** |
| Console-reachable server becomes an attack surface | Gogo on **12612** and Felix Web Console on **8443** can install bundles | Network-restrict both ports in production; never expose a test instance (§5.5, §5.8, §7.2) | **✅** |
| A requirement assumed clustering/HA the project may not document | The bundles (`org.idempiere.hazelcast.service`, `org.idempiere.redis.service`) exist, but the topology was **not verified** | Read the project's clustering documentation directly — do not assume (§7.4) | **⚠ not verified** |

---
## 12. The Cymbal Bank Worked Example
*Explicitly illustrative. This is a fictional design exercise set in the familiar context of the author's employer, **Cymbal Bank** — not a record of any actual project, and not a claim about any real deployment. It follows the sibling's §9 pattern ([odoo_framework_guide.md](odoo_framework_guide.md) §9): scenario, selection, deployment, extension design, lessons.*
### 12.1 The Scenario
Cymbal Bank's operations division runs its back office (the ERP domain of §9.1) on a patchwork: a spreadsheet-era expense process, a procurement workflow spread across email, HR records in three tools, and a general-ledger feeder held together by scheduled scripts. The core-banking estate (the T24-class ledger of [../banking/t24_programming_guide.md](../banking/t24_programming_guide.md)) is healthy and **explicitly out of scope** — this project touches nothing in the customer ledger.

The mandate, driven by the business case framework of [../management/business_case_development_guide.md](../management/business_case_development_guide.md): **replace the back-office patchwork with one system — the bank's own GL, procurement, HR-light and expense processing — under a hard budget, without a licence line, and with the option to run it entirely on-premise behind the firewall.** The business case compares candidates on TCO; because iDempiere carries **no licence fee and no per-user fee** (§2.7), its TCO reduces to infrastructure, implementation, integration and the support/upgrade labour §11 prices honestly. The bank selects iDempiere — **with the community-support reality (§11.3) written into the business case as an explicit, costed line rather than glossed over.**
### 12.2 The Selection
The design principle, stated once: **start with what the dictionary can express, add a plug-in only where behaviour must live in code, and make the 2Pack the only path from development to production.**

| Back-office need | iDempiere mechanism | Layer | Rationale |
|---|---|---|---|
| The bank's own GL, AP/AR, fixed assets | Core accounting + the bank's own accounting schema | **Dictionary + core** | Multiple account schemas are a first-class feature (§9.1); several legal entities in one login |
| Multi-entity books | Multiple entities (sets of books) in a single login | **Dictionary** | Advertised capability, verified (§3.4, §9.1) |
| Procurement, vendor master, PO→invoice matching | Core purchase + custom windows for the bank's approval structure | **Dictionary** | Approval routing expressed as dictionary windows/tabs, built with the six-step workflow (§4) |
| HR-light: employee records, time off, expenses | Core windows + a custom expense window | **Dictionary** | Deliberately **not** payroll — payroll stays in the bank's specialised system (§9.2) |
| Role-based access for bank staff | Roles + **Window Access** rows | **Dictionary** | Access is metadata; the same Role window carries scope (§4.5) |
| Bank-specific validations (e.g. cost-centre rules on a requisition) | **Event handler**, not a callout | **Code** | The rule must hold across UI, REST and batch imports — see §12.5 |
| Integration to the core-banking estate (GL feeds, reconciliation) | Server **web services** (XML formats; input/output templates for any table, process or workflow) | **Core + code** | Wikipedia-verified web-services capability (§3.8) |
| Distribution of everything | **2Pack** for dictionary content; **p2 repository** for bundles | **Transport** | §5.1, §5.5 |

**One selection decision is deliberately conservative and worth naming:** the design **does not adopt the Extension Registry route (§5.5)**, because it is an **iDempiere 14+** feature and the current stable is **13 "Orion"**. The bank distributes its bundle by **p2 repository** (`update-prd.sh`, or the state-checking `compare-update-prd.sh`), which is the release-13 route the documentation recommends. The registry is noted as the upgrade-path improvement to adopt when the bank moves to 14.
### 12.3 The Deployment Design
- **Hosting:** **on-premise behind the firewall** (the scenario's hard constraint), on the documented release-13 target — **Ubuntu 24.04 64-bit, PostgreSQL 17, OpenJDK 17** (§6.3, §3.3). *The bank ignores the Features page's older "PostgreSQL >= 10" floor and the live test sites' "postgresql 18", and pins to what the release-13 prerequisites document.*
- **Network posture — the deployment's most important single control.** The server's **Gogo console on 12612** and the **Felix Web Console on 8443** (§5.5, §7.2) are bundle-**installation** surfaces. In production they are **bound to the local management network only**, never to the application network, and the application's **8080/8443** are behind the bank's reverse proxy. The project's own test-site warning — *"an advanced role is effectively operating-system access"* (§5.8) — is treated as a design constraint, not a curiosity.
- **Anti-pattern avoidance from day one:** every custom dictionary object is prefixed (`XX_`-style), the **system inventory** (§11.1) is created at project start and kept in Git, and **every dictionary change goes out as a 2Pack under review** (§11.4 guardrail #1).
- **Release strategy:** pin to **13 "Orion"** at go-live; treat the **fresh-instance** rule for major versions (§11.1) as the plan for a future 14 move, not as a surprise; keep restore points and expect the migration to **iterate** (the script stops at the first error).
- **Integration:** one integration layer using the server's **web services** in both directions — GL postings out to the bank's financial data platform, statement data in for reconciliation — with the **core-banking ledger explicitly out of scope** (§9.1).
- **Support model:** a **community integrator** engaged for escalation (§11.3), because there is no vendor to escalate to — written into the business case as a recurring line.
### 12.4 The Extension Design
The scenario's customisation splits cleanly across the two layers of §5.7, and the split *is* the design:

**In the dictionary (no code), transported as 2Packs:**

- The bank's custom windows — expense, requisition approval, cost-centre master — built with the documented six-step workflow (§4.2–§4.5): table, columns (+ Create/Complete Table, Synchronize column), window/tab/field (+ Create Window, Tab and Field from Table), menu, role access.
- The roles and their **Window Access** rows, scoped per bank function.
- The reference lists the bank needs (cost-centre categories, approval tiers as **List** references).

**In the plug-in (code), distributed by p2:**

- The **bank-specific validations** — as **event handlers**, registered on the relevant tables via `@Component(service = EventHandler.class)` and bound to **`IEventManager`**, subscribing to the `PO_BEFORE_*` / `PO_AFTER_*` topics (§5.6).
- The **integration adapters** for the GL feed and the reconciliation pull.

**The composition — the whole point of §5.7, made concrete:** the **plug-in carries the 2Pack**. The dictionary content is packed out to a `.zip`, placed in the plug-in's `META-INF`, and installed by **`Incremental2PackActivator`** — chosen over `AdempiereActivator` *specifically* because the bank expects several versioned 2Packs (`2Pack_Tables_1.0.0.zip`, `2Pack_Processes_1.0.1.zip`, …) that must load and install **in order**, and because only newer versions install (§5.3). The bundle goes out over a **p2 repository**; the metadata rides inside it; **and the same 2Pack can also be dropped into the Automatic External Pack In folder** for the bank's CI/CD pipeline, named to the mandatory convention `[Timestamp:YYYYMMDDHHMM]_[ClientValue]_[AdditionalInformation].zip` — e.g. `202610170900_SYSTEM_CymbalProcurementApproval.zip` — so a test environment configures itself zero-touch (§5.3). *One artefact set, three transports, all documented.*
### 12.5 The Decision That the iDempiere-ness Changed
**This is the decision the guide exists to make explicit, and it is labelled as such: it was decided differently *because* the platform is iDempiere, and it would have been decided differently again in Odoo or in a proprietary suite.**

**The decision: the bank's cost-centre validation was placed in an *event handler* rather than in a *callout* — and the reason is a documented limitation of the dictionary, not a style preference.**

**What a naive iDempiere design would have done.** The obvious, documentation-blessed way to add "a requisition's cost centre must belong to the requesting department" is a **callout** — the guide's own §5.6, the "modern, recommended" iDempiere 9+ mechanism, `@Callout(tableName="XX_Requisition", columnName="XX_CostCentre_ID")` on a dictionary column. It is the *idiomatic* answer: a field-level rule attached to a field, auto-filling and validating as the user types. A first-pass design would stop there.

**Why the iDempiere-ness forced otherwise.** The bank's requisitions do **not** only arrive through the window. They arrive **in batch, through the REST API**, from an upstream departmental system, and through a nightly **process**. The callout documentation says, verbatim, that callouts **only** work on the UI and CSV-importer paths and do **NOT** execute for **REST API calls, background processes, direct database operations, or Java code using `PO.save()`** (§10.1). **So the callout would validate the requisitions a clerk types and silently skip every requisition the system posts.** The rule would be green in testing (a human clicked through the window) and absent in production (the API posted the row). **The platform's own documented execution-point semantics — not a preference — decided the mechanism: the rule had to be an event handler**, registered on the `PO_BEFORE_NEW`/`PO_BEFORE_CHANGE` topics, because that is the execution point **all** entry points share (§5.6).

**How this would have been decided differently elsewhere, and why:**

- **In Odoo**, the same requirement is written as a **constraint on the model** (`@api.constrains`) or an override of `create()`/`write()` in a Python model class — and **the constraint fires on every path by construction**, because Odoo's rules live *in the model* and every entry point goes *through the model*. There is no UI-only/multi-entry-point split to get wrong. So an Odoo team would not have faced this decision at all: the analogous mechanism (a model constraint) is *everywhere by default*, and the failure mode iDempiere forced the bank to design around **does not exist in the same form** ([odoo_framework_guide.md](odoo_framework_guide.md) §3.2, §3.5). **The iDempiere-ness created a decision Odoo does not have.**
- **In a proprietary suite**, the same requirement is typically met by a **module or an extension point defined and supported by the vendor** — the vendor's rule engine, workflow, or a supported customisation framework — and the *scope question* ("which entry points does this rule cover?") is answered by the vendor's documentation for that extension point, or is a support ticket. There is no equivalent of the callout/event-handler split *laid out in the public documentation as a trap and a remedy*; the platform decides where customisation goes and the adopter chooses among the vendor's sanctioned hooks.

**The point, stated cleanly:** **in iDempiere, the extension-point choice is load-bearing, and the platform *documents* the trap rather than hiding it.** The cost-centre rule had to be an **event handler** because the callout's documented scope could not cover the API path — a decision that **would have been a non-decision in Odoo** (where model constraints are universal) and a **vendor-specified** decision in a proprietary suite. Choosing the right side of the dictionary/code seam (§5.7, step 4) *is* the craft of iDempiere development, and the bank's design encodes it as policy: **any rule that must hold for data from more than one entry point is an event handler, without exception.**
### 12.6 The Lessons
1. **The dictionary is the first tool and the code is the second, and the seam between them is the design.** Everything structural — windows, tabs, fields, access, references — belongs in metadata and travels as a 2Pack; everything behavioural — rules, calculations, side-effects — belongs in a bundle. The scenario's split (§12.4) is the whole method.
2. **The extension-point choice is a coverage decision, not a style decision.** A callout covers the UI; an event handler covers every entry point (§5.6, §10.1). Get it wrong and the rule is green in test and absent in production.
3. **Configuration is not free.** The bank budgets the 2Pack review gate, the prefix discipline, and the system inventory as *work* — because configuration without a gate is the most expensive habit a dictionary-driven project forms (§11.4).
4. **Price the support model honestly.** No vendor, no SLA, no escalation — mitigated by a community integrator, costed in the business case (§11.3, §12.3).
5. **Plan the upgrade as a project, not an event** — fresh instance for majors, migration notes per jump, restore points, and the expectation that scripts stop at the first error and iterate (§11.1) — **and treat the console ports as a security boundary**: 12612 and 8443 install code, so in a bank they are management-network-only (§5.8, §7.2, §12.3).

**And the close, on the thesis.** The Cymbal back office is, in the end, **one dictionary** — the bank's tables, windows, fields, references and access rules, written down as rows in the same PostgreSQL database the business data lives in — plus **one plug-in** that holds the behaviour the dictionary cannot write down. The model, the screen and the table really are three renderings of one dictionary entry (§4.6); the cost-centre rule really did have to leave the dictionary for a bundle because the dictionary could not express an entry-point condition (§10.1); and holding both of those facts at once — the dictionary *is* the structure, and the dictionary *is not* the behaviour — is exactly what it means to develop on iDempiere. The bank's design is the thesis, its sharpening, and its limit, all three, in one system.

---
## 13. The Anti-Patterns
Each anti-pattern: the symptom that reveals it, and the correction. All are grounded in the verified material above.

1. **The UI-only rule (a callout where an event handler is required).** *Symptom:* a validation "works" when a clerk types a record and does nothing when the system posts one; the rule is green in testing and absent in production. *Correction:* any rule that must hold across more than one entry point is an **event handler**; reserve callouts for genuinely UI-only field interactions. (§5.6, §10.1)
2. **The unprefixed custom column.** *Symptom:* a migration script fails with a duplicate-column error because core added a column with the same name. *Correction:* **prefix every custom object** (`XX_`, `LIT_`, `ORG_`); use a **custom entity type**, never `Dictionary`. (§4.2, §4.7, §11.1)
3. **The SQL-only custom view.** *Symptom:* a custom view vanishes after an update, or the migration stops on it. *Correction:* **define views in the Application Dictionary**, not as raw SQL; run **Post Migration Helper** and **Verify Migration**. (§11.1, §11.2)
4. **The raw-SQL write.** *Symptom:* records inserted by a script or a direct database operation have no validation, no audit trail, no events — and business rules "randomly" skip them. *Correction:* route writes through the **model layer** (`PO`); never write business tables with direct DML. (§10.1, §11.2)
5. **The unversioned dictionary change.** *Symptom:* "who changed this column, and when?" has no answer; the change is untraceable in Git. *Correction:* **route every dictionary change through a 2Pack under version control**; the 2Pack is the diff. (§11.4)
6. **The unreviewed metadata change.** *Symptom:* a production field is mandatory (or a reference is wrong) and no one reviewed it, because "it isn't code". *Correction:* put a **review gate on the 2Pack**; treat a metadata pull request as seriously as a code one. (§11.4)
7. **The missing inventory.** *Symptom:* the upgrade team discovers custom columns, views and plug-in versions *during* the upgrade. *Correction:* build the **system inventory** at project start, keep it in version control, and use it as the migration checklist. (§11.1)
8. **The skip-the-notes upgrade.** *Symptom:* an upgrade "succeeds" and then regressions appear; the team read only the target version's notes. *Correction:* review migration notes for **every version jump in sequence** (10 → 11 → 12), not just the destination. (§11.1)
9. **The in-place major upgrade.** *Symptom:* a failed process corrupts the production instance; the runtime floors move underneath a live server. *Correction:* **fresh instance for major versions**, with a database migration and multiple restore points; in-place only for minors, on a staging mirror. (§11.1)
10. **The assumed cluster.** *Symptom:* an architecture diagram asserts failover and session replication that no one verified against the project's documentation. *Correction:* read the project's clustering documentation directly; the presence of the Hazelcast/Redis bundles is **not** a topology. (§7.4, §11.5)
11. **The exposed console.** *Symptom:* iDempiere's Gogo telnet port (12612) or Felix Web Console (8443) is reachable from the application network, so "a role is effectively OS access". *Correction:* bind both to a management network only; never expose a test instance. (§5.5, §5.8, §7.2)
12. **The bloated core modification.** *Symptom:* customisations were made by editing the core source tree, so every upstream update is a merge conflict. *Correction:* develop plug-ins **outside the core repository**, target **Equinox**, and contribute genuinely general improvements back to core. (§5.4, §6.1, §11.2)
13. **The 2Pack-into-the-wrong-client.** *Symptom:* a package installs but nothing changes, because it landed in `System` when the data lives in `GardenWorld` (or vice versa). *Correction:* the dictionary is **tenant-scoped**; always install into the intended **client**. (§4.6, §5.2)
14. **The unbumped 2Pack.** *Symptom:* an updated plug-in deploys and the configuration silently does not change. *Correction:* **bump the plug-in and 2Pack versions together** — only newer versions install. (§5.3, §5.7)
15. **The thesis overclaim.** *Symptom:* the team believes "the dictionary is the whole application" and plans no code, no tests and no review. *Correction:* the dictionary renders structure at every entry point but hosts behaviour only on the UI path; plan for **both** layers and the seam between them. (§1.2, §4.6, §10.4)

---
## 14. The Claims Audit
Every load-bearing claim, with its source, the date it was read, and its kind. **Kind legend:** *primary-project* = the project's own site, documentation, repository or hosts; *secondary* = Wikipedia or another third party; *analysis* = this guide's own reasoning over verified facts.

| # | Claim | Source | Date | Kind |
|---|---|---|---|---|
| 1 | Definition ("Tier II… skilful community"; "Founded in 2011") and stack ("Java, ZK, OSGi, Oracle, PostgreSQL, Maven"; "based on original Compiere/Adempiere plus a new architecture…") | www.idempiere.org (intro, home, Features) | 2026-10-17 | primary-project |
| 2 | Licence: "The built iDempiere source is licensed under the GNU General Public License version 2 (or later)" → **GPL-2.0-or-later**; GitHub classifier reports "Other"/NOASSERTION | idempiere.org/source-code; github.com/idempiere/idempiere | 2026-10-17 | primary-project |
| 3 | Repository statistics (`master`, 16,318 commits, 666 stars, 482 forks, 70 open issues, created 2018-02-26, last push 2026-10-14) and the full module listing | github.com/idempiere/idempiere | 2026-10-17 | primary-project |
| 4 | The complete Application Dictionary workflow: six steps, Create/Complete Table, Synchronize column, Create Window/Tab/Field from Table, mandatory standard columns, new-window-vs-detail-tab rule, best practices | docs.idempiere.org, "Creating a New Window in iDempiere" | 2026-10-17 | primary-project |
| 5 | Extension points: callouts (UI + CSV importer only); event handlers (all entry points, via OSGi Event Admin / `IEventManager`, with the `IEventTopics` list and the Event-Handler-vs-Callout table); model validators (`ModelValidator`/`IModelValidatorFactory`, legacy) | docs.idempiere.org, Callout / Event Handler / Model Validator guides | 2026-10-17 | primary-project |
| 6 | 2Pack: definition, contents, Pack Out/Pack In, Automatic External Pack In with filename convention and `201803161725_SYSTEM_CreateNewColumn.zip`, `AdempiereActivator` / `Incremental2PackActivator`, PIPO handler extension | docs.idempiere.org, "What is a 2Pack" and the Pack In/Pack Out guide | 2026-10-17 | primary-project |
| 7 | Plug-in development (outside core, Equinox, JavaSE-17), Maven/Tycho layout from 6.1, distribution (`update-prd.sh`/`compare-update-prd.sh`, Gogo on 12612, Felix Web Console on 8443 at start level 5), and the 14+ Extension Registry | docs.idempiere.org, plug-in and registry guides | 2026-10-17 | primary-project |
| 8 | Prerequisites (Ubuntu 24.04, PostgreSQL 17 / any ≥14, git, OpenJDK 17, Eclipse 2025-12, Oracle 23ai+); `scram-sha-256` and the `trust` warning; `./mvnw verify` ~5 GB; Eclipse minimum 2024-03; clone ≥3 GB; branches `master`=14 and `release-13` | docs.idempiere.org prerequisites / Eclipse / code pages | 2026-10-17 | primary-project |
| 9 | v13 technical migration notes: UUID native type and UUIDv7 (IDEMPIERE-6650), `stringtype=unspecified`, deprecated `X_` getters (IDEMPIERE-6642), removal of >2-major-old APIs (IDEMPIERE-6735), CSV Qty changes (IDEMPIERE-7010), `AD_Attachment.Title` list (IDEMPIERE-6640), ZK push flip-flop | docs.idempiere.org, v13 technical migration notes | 2026-10-17 | primary-project |
| 10 | Upgrade guide: fresh-instance rule and its reasons; inventory; migration notes per jump; `update.sh`; Post Migration Helper outputs; `RUN_SyncDB.sh` stops on first error; Verify Migration checks; duplicate-column/ID and view-overwrite remedies | docs.idempiere.org upgrade guide (Wayback capture) | 2026-06-10 | primary-project |
| 11 | Version trail (1.0a … 12); "Daily 13 Orion (released 2026-03-06)"; the 6.2 date discrepancy (2019-01-06 vs 2019-01-04); the 13 "Orion" date trio (2026-03-06 / 2026-03-12 / 9 March 2026) | idempiere.org/releases; project blog; Wikipedia | 2026-10-17 | primary-project + secondary |
| 12 | Lineage and history: Compiere 1999 (Jorg Janke, ex-Oracle); the 2006 VC (New Enterprise Associates) and editions; 3.3 on 24 Mar 2009; the Consona acquisition 16 Jun 2010; ADempiere fork Sept 2006 (infobox 12 Oct 2006, GPLv2, Jorg Janke as author), Java EE/Tomcat/JBoss, PostgreSQL+Oracle only, 3.9.4 Jan 2023, the council governance; the 2008–2010 Felix/Equinox modular proposal; 2010 LTS and Branch GlobalQSS; the 2011 iDempiere name; the platform changes per version | Wikipedia *Compiere*, *ADempiere*, *iDempiere* | 2026-10-17 | secondary |
| 13 | Wikipedia *Platform* (Java server on Jetty, ZK 9.6 GUI, OSGi modularisation, Oracle ≥23ai / PostgreSQL ≥14, browsers) and *Overview* (tenants/orgs/languages/currencies/account schemas/users; **16 languages vs 29 localizations**) | Wikipedia *iDempiere* | 2026-10-17 | secondary |
| 14 | Community: "no single person or company ruling the project"; decentralised bazaar; local communities; channels; conference **attendance** records; the InfoWorld Bossie (2015, 2016) and SourceForge Community Choice (Jul 2016, Feb 2018) awards; the listicle placements recorded as existing only (Computer Review 2015-11-05; PCQuest 2016-05-20; Capterra 2017-07-26) | Wikipedia *iDempiere* | 2026-10-17 | secondary |
| 15 | ADempiere repository today: `develop`, 16,989 commits, 888 stars, 796 forks, GPL-2.0 | github.com/adempiere/adempiere | 2026-10-17 | primary-project (the other project) |
| 16 | Project infrastructure (JIRA, Jenkins, Mattermost, Google Groups, docs.idempiere.org — 540 pages in sitemap — wiki, donations, Hacktoberfest, "Hero of the Month"), the contributor documents, and the documentation credits (Carlos Ruiz / BX Service GmbH; Luis Amesty / Amerpsoft; BX Service's Document Status Validator) | github.com/idempiere/idempiere README; docs.idempiere.org | 2026-10-17 | primary-project |
| 17 | Test sites (demo.globalqss.com = 13 Orion LTS, Colombia localisation + REST, openjdk 17/postgresql 18/Ubuntu 22.04, continuously deployed; test.idempiere.org = 14 dev; test-oracle.idempiere.org = 14 dev, Oracle 18c XE, Debian 9; unsecured warning) and the installation routes (Docker first — `idempiereofficial/idempiere:12-release` + PostgreSQL 16, ports 8080/8443/12612 — then script, then manual; installers on SourceForge) | idempiere.org; docs.idempiere.org | 2026-10-17 | primary-project |
| 18 | The repository-gap count: before this guide, iDempiere appeared in exactly **three** repository files (jack_liu_profile.md, asia_fusion_technology_guide.md, odoo_framework_guide.md) | full-text search of /home/ubuntu/research | 2026-10-17 | primary-project (this repo) |
| 19 | The thesis and its sharpening ("one dictionary entry renders a physical table, a screen and an access rule — and carries hook points to which behaviour, defined in code, may attach"); the seam analysis of §10; the anti-pattern list; the Cymbal Bank design | this guide, over the verified facts above | 2026-10-17 | analysis |
| 20 | **No adoption figure, no market share, no performance benchmark and no ranking of any vendor or product against another is printed in this guide.** Deliberately: (a) the house rule for the framework series forbids them; (b) the sources offering such figures for iDempiere are secondary or promotional (the 2006-era Compiere popularity claim, the listicle placements), so reproducing them would print unverifiable scale claims as fact; (c) a structural guide's value is in *how* the systems are built, not in a popularity contest. Repository statistics (commits, stars, forks, issue counts) and community conference **attendance** counts are printed because they are *descriptive facts about artefacts and events*, not adoption metrics, and each is labelled as such (§1.5, §2.8). | this guide — a deliberate omission, recorded as a claim | 2026-10-17 | analysis |

---
## 15. What Could Not Be Verified
*Honesty section. For a community-led project whose primary record lives partly on a Cloudflare-protected wiki, this list is substantial. Each entry states what could not be confirmed and why.*

- **The project wiki could not be read at all this pass.** `wiki.idempiere.org` — including its **"Plugin" page** and its **Application Dictionary pages** — served a **Cloudflare challenge ("Just a moment…")** and could not be extracted. Wikipedia cites several of those wiki pages (the migration-notes page, the New Features category, the AvailablePlugins category, the conference pages, the JasperReports redesign page, the NF5.1 Platform Upgrade page), so their existence is corroborated, but **their content was not read this pass**. This is recorded as a *tool limitation*, not as absence of material — the wiki demonstrably exists and is actively cited.
- **Several referenced documentation pages were not in the fetched set.** Pages that the cached documents cross-reference — the **"Running Plug-ins Locally"** guide, the **v9 "new column callout factory"** and **"new process factory"** pages, the **v14 "extension management"** page, **removed-deprecated-apis.md**, and the **clustering** documentation — were cited by fetched pages but could not themselves be read. Claims that depend on them are attributed to the page that cites them and marked accordingly.
- **Release 13 "Orion" has three conflicting release dates** — the project's Releases page says **2026-03-06**, the project's blog post is dated **2026-03-12**, and Wikipedia says **9 March 2026** — and this guide records all three rather than choosing (§2.6). It could not determine which value the project itself would consider canonical.
- **The 6.2 release date disagrees** between the project's installer page (**2019-01-06**) and Wikipedia (**2019-01-04**) (§2.5). Recorded, not resolved.
- **The Java and PostgreSQL version floors differ between the project's own pages by vintage** (§3.3) — the Features page (OpenJDK ≥11 / PostgreSQL ≥10 / Oracle ≥12c), the release-13 documentation (OpenJDK 17 / PostgreSQL 17, any ≥14 / Oracle 23ai+), Wikipedia (PostgreSQL ≥14 / Oracle ≥23ai), and the live test sites (postgresql 18; Oracle 18c XE). **No single authoritative floor could be established**; the guide presents the dated table and advises re-checking for a target release.
- **The language count disagrees within Wikipedia itself** — the infobox says **16 languages**, the Overview text says **29 localizations** (§3.4). Recorded, not reconciled.
- **The clustering / high-availability topology was not verified.** The repository contains `org.idempiere.hazelcast.service` and `org.idempiere.redis.service` bundles, but **what they implement and whether a supported cluster topology is documented could not be confirmed this pass** (§7.4). The guide deliberately refuses to state a failover model, a session-replication scheme or a supported node count. **⚠ not verified this pass.**
- **The project's 2026-era change process was only partially verified.** The presence of **`.agents/skills`**, **`.claude`**, **`AGENTS.md`**, **`CLAUDE.md`**, **`GEMINI.md`**, **`QWEN.md`**, **`.coderabbit.yaml`** and **`skills/idempiere-review-extension/`** in the repository is **✅ verified from the listings**; the content of the **"using AI agents" contributor page** was noted to exist but **not read**, so the specifics of the project's AI-agent contribution workflow are **⚠ not verified**.
- **The 2006-era Compiere popularity figures were deliberately not reproduced.** Wikipedia carries a 2006-era claim about Compiere's SourceForge prominence, downloads and partners; it is **secondary**, it is a **scale figure**, and the house rule forbids printing it as evidence. The guide states "a leading SourceForge ERP project in its early years" and marks it **⚠ secondary** (§2.1). Similarly, the listicle placements (Computer Review, PCQuest, Capterra) are recorded as **existing, with dates**, and **no ranking language is reproduced** (§2.8).
- **The Docker image tag names a previous release.** The documentation's Docker example is `idempiereofficial/idempiere:12-release` while the current stable is 13 (§7.1) — read as a documentation-vintage detail; whether a `13`-tagged image is published was **not verified this pass**.
- **The contents of the Jenkins pipelines were not read.** The claim that iDempiere runs its CI on Jenkins is verified from the project's own README and corroborated by the repository's test modules (§6.4); **what each job builds or runs was not verified.**
- **The worked example (§12) is fictional.** It is an explicitly framed illustrative scenario in the **Cymbal Bank** context — **not** a record of any actual project, and no figure, timeline or outcome in it should be read as a real datum.
- **"Tier II" in the project's own strapline** ("a powerful, Tier II, open-source ERP/CRM/SCM system") is quoted verbatim from the project's site; **the project does not define what "Tier II" means on that page, and this guide does not gloss it.**

---
## 16. The Glossary, the Cross-References, the Closing Summary, and a Verification Ledger
### 16.1 The Glossary

| Term | Definition | Marker |
|---|---|---|
| **iDempiere** | The open-source ERP/CRM/SCM suite (Java, OSGi); "Community Powered Enterprise"; strapline: "a powerful, Tier II, open-source ERP/CRM/SCM system supported by a skilful community" | **✅** |
| **Compiere / ADempiere** | The 1999 ancestor by Jorg Janke; the September 2006 fork of Compiere that extended the dictionary into the application (GPLv2) and is iDempiere's parent | **✅** |
| **Application Dictionary** | The metadata layer describing tables, columns, windows, tabs, fields, references, validations, menus and roles; Wikipedia's "Active Data Dictionary" — consulted by the running application, "lets it manage entities, validation rules, windows, tables, formats… without new Java code" | **✅** |
| **OSGi / Equinox / bundle** | The Java module system used for modularisation; the Eclipse OSGi implementation iDempiere embeds and developers target; the unit of modularity (the core and all plug-ins are bundles). The runtime itself is [osgi_guide.md](osgi_guide.md)'s subject | **✅** |
| **Plug-in / extension point** | A bundle that adds or changes ERP/CRM/SCM functionality, developed outside the core repository, attaching at a named hook — callouts, event handlers, model factories, validators, processes, factories, forms, themes, registry | **✅** |
| **Callout / event handler / model validator** | Field-level behaviour that runs **only** on the UI and CSV-importer paths / record-level behaviour that runs **before/after database operations** on **all** entry points, via OSGi Event Admin and `IEventManager` / record-level behaviour via `ModelValidator`/`IModelValidatorFactory`, the documented legacy alternative | **✅** |
| **2Pack / Pack Out / Pack In / PIPO** | A `.zip` "configuration snapshot" of dictionary metadata and configuration; its export and import operations (manual, activator-driven, or Automatic External Pack In); the engine, whose custom package types are added by implementing handlers in `org.adempiere.pipo.handlers` | **✅** |
| **Tenant / client / organisation** | The tenancy unit on every row as `AD_Client_ID` (the dictionary is tenant-scoped, which is why a 2Pack installs *into a client*); the sub-unit as `AD_Org_ID` | **✅** |
| **GardenWorld** | The demonstration client shipped with iDempiere; the environment the developer worked examples use | **✅** |
| **Gogo console / Felix Web Console / p2 / Maven-Tycho** | The Apache Felix Gogo telnet console on **12612**; the bundle-management web console at `https://<server>:8443/osgi/system/console/bundles` (recommended start level 5); Eclipse's provisioning system, the recommended plug-in distribution mechanism (`update-prd.sh`, `compare-update-prd.sh`); the build system from 6.1 onward (replaced Buckminster) that produces the p2 repository | **✅** |
| **ZK / Jetty / WfMC-OMG** | The Ajax framework behind the web GUI (Wikipedia's current Platform section names **ZK 9.6**); the servlet container the server runs in; the standards bodies behind the workflow/BPM engine | **✅/⚠** |
| **Migration Notes / Post Migration Helper / Verify Migration** | The per-version breaking-change notes read for **every** version jump; the post-update process reporting overwritten customisations, field-order changes, changelog hits and manual-review items; the post-migration check process (changed customisations, overwritten views, field-order changes, missing references) | **✅** |
| **Extension Registry / Tier II** | The iDempiere **14+** official distribution of plug-ins from source, via `github.com/idempiere/idempiere-extension-repository`; the project's own strapline classification, **not defined by the project on the page read** | **✅ / ⚠** |

### 16.2 The Cross-References
- **[odoo_framework_guide.md](odoo_framework_guide.md)** — the sibling ERP and the Python member of the pair; its **§3.5** is the sentence this guide answers, its **§7** owns the full comparison (including Tryton and the open-source-vs-commercial frame), its **§8** owns the ERP-in-banking framing, its **§9** owns the Cymbal back-office worked example, and its **§1.3/§4** own the open-core model this guide contrasts structurally. **Cited, not duplicated.**
- **[osgi_guide.md](osgi_guide.md)** — the OSGi runtime in full; this guide describes only *how iDempiere uses* it. Its **§11 platform table** does **not** list iDempiere — an observation made here, unrepaired there.
- **[asia_fusion_technology_guide.md](asia_fusion_technology_guide.md)** — **§4.4–4.5**, the author's ERP-domain career context (**employer-domain, flagged as such**).
- **[jetty_guide.md](jetty_guide.md)**, **[rxjava_guide.md](rxjava_guide.md)**, **[low_latency_java_programming_guide.md](low_latency_java_programming_guide.md)**, **[talend_guide.md](talend_guide.md)** and **[jbase_universe_guide.md](jbase_universe_guide.md)** — the servlet container iDempiere runs in; the Java language and reactive material; adjacent data-integration and legacy-database material. (The low-latency guide is already named in the line above; this line adds the rest.)
- **[../banking/t24_programming_guide.md](../banking/t24_programming_guide.md)** and **[../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md)** — the core-banking boundary of §9.1.
- **[../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md)** — the open-source-vs-commercial survey and its structural finding.
- **[../management/business_case_development_guide.md](../management/business_case_development_guide.md)** — the TCO/business-case lens for §12.
- **[../personal/jack_liu_profile.md](../personal/jack_liu_profile.md)** — the career context: "ERP frameworks (Odoo, iDempiere) — metadata-driven development, DSLs, application dictionaries".
### 16.3 The Closing Summary
**iDempiere is the dictionary made into a product.** It descends from Compiere (1999) through the ADempiere fork (September 2006) to the 2011 code path that took the community's modular-ERP proposal and rebuilt the application around OSGi — and in doing so it did something unusual: it kept the *idea* (the Active Data Dictionary that controls entities, validations and screen layouts from inside the application) and replaced the *housing* (bundles, plug-ins, p2, Equinox). The dictionary renders the model, the screen and the database table from one entry; the plug-in layer puts behaviour around what the dictionary cannot write down.

The facts that make it what it is, all verified this pass: **GPL-2.0-or-later, with no open-core split, no enterprise edition and no per-user licence**; a **community, not a company**, at the centre — "no single person or company ruling the project"; a stack of **Java on Jetty with a ZK web GUI over PostgreSQL or Oracle**; a **roughly annual major cadence** that settled onto 24 December; a **current stable of 13 "Orion"** whose exact date three sources disagree on; and a **documentation set that is remarkably candid** — it tells you callouts do not fire on the API path, that migration scripts stop at the first error, that `trust` auth is a security hole, that the public test sites are unsecured, and that major upgrades should be fresh instances rather than in-place.

**For a developer**, the guide's practical content reduces to a seam. The dictionary is the first tool and the code is the second; **callouts cover the UI only, event handlers cover every entry point**; the extension-point choice is a coverage decision; configuration travels as **2Packs** and behaviour travels as **bundles**; and the project's own upgrade discipline — inventory, migration notes per jump, fresh instance for majors, Post Migration Helper after — is the maintenance regime that keeps a dictionary from colliding with its own core. **The craft is knowing which side of the seam each piece of behaviour belongs on.**

**For an architect**, the guide's content reduces to an honest trade. On one side: no licence, inspectable source, multiple account schemas, a dictionary that a functional team can drive, and a governance model that refuses a vendor. On the other: **no vendor to escalate to, no SLA, hard dictionary-driven upgrades, and console ports that install code.** The trade is not hidden — it is in the project's own documentation and in this guide's §11 — and an organisation should price it before it chooses, not after.

**The limits of the metaphor, stated once more, because the whole guide turns on them.** The thesis holds and is sharpened (§4.6): *one dictionary entry renders a physical table, a screen and an access rule — and carries hook points to which behaviour, defined in code, may attach.* It breaks at a precise place (§10.4): *the dictionary can describe structure at every entry point, but it can host behaviour only where the dictionary can see how the data arrived — and the dictionary does not model that.* The model, the screen and the table are three renderings of one dictionary entry; the rule that governs them is not, and never was.
### 16.4 The Verification Ledger
### Verified this pass (✅)
- **Definition and stack**: "Tier II open-source ERP/CRM/SCM"; "Founded in 2011"; stack "Java, ZK, OSGi, Oracle, PostgreSQL, Maven"; "based on original Compiere/Adempiere plus a new architecture…" (idempiere.org, 2026-10-17).
- **Licence**: GPL-2.0-or-later, from the project's own source-code page; GitHub classifier "Other"/NOASSERTION noted.
- **Repository**: `master`, 16,318 commits, 666 stars, 482 forks, 70 open issues, created 2018-02-26, last push 2026-10-14, module listing (github.com/idempiere/idempiere, 2026-10-17).
- **The Application Dictionary workflow**: the full six steps, the Create/Complete Table process, Synchronize column, Create Window/Tab/Field from Table, the mandatory standard columns, the new-window-vs-detail-tab rule, the best practices (docs.idempiere.org, 2026-10-17).
- **Extension points**: callout scope limit (UI + CSV importer only); event handlers via OSGi Event Admin / `IEventManager` with the full `IEventTopics` list and the Event-Handler-vs-Callout table; model validators via `ModelValidator`/`IModelValidatorFactory` and the recommendation to prefer event handlers; the 9+ annotation era and the pre-9 legacy factory (docs.idempiere.org, 2026-10-17).
- **2Packs**: definition, contents, Pack Out/Pack In, Automatic External Pack In with its filename convention and `201803161725_SYSTEM_CreateNewColumn.zip` example, `AdempiereActivator` and `Incremental2PackActivator`, the PIPO handler extension (docs.idempiere.org, 2026-10-17).
- **Plug-ins and distribution**: the Eclipse workflow, Equinox target, JavaSE-17, "generate an activator" disabled, the Maven/Tycho layout from 6.1, `update-prd.sh`/`compare-update-prd.sh`, the `ss`/`install`/`uninstall`/`setbsl`/`sta`/`sto`/`help` commands on telnet 12612, the Felix Web Console on 8443 with start level 5, and the 14+ Extension Registry (docs.idempiere.org, 2026-10-17).
- **Toolchain**: Ubuntu 24.04, PostgreSQL 17 (any ≥14), git, OpenJDK 17, Eclipse 2025-12; `scram-sha-256` and the `trust` warning; `./mvnw verify`, ~5 GB; Eclipse setup specifics and minimum 2024-03; clone ≥3 GB; branches `master`=14 and `release-13` (docs.idempiere.org, 2026-10-17).
- **v13 technical migration notes**: UUID native type and UUIDv7 (IDEMPIERE-6650), `stringtype=unspecified`, deprecated `X_` getters (IDEMPIERE-6642), removal of >2-major-old APIs (IDEMPIERE-6735), CSV Qty field changes (IDEMPIERE-7010), `AD_Attachment.Title` (IDEMPIERE-6640), ZK push flip-flop (docs.idempiere.org, 2026-10-17).
- **Upgrade guide**: fresh-instance rule and reasons, inventory, migration notes per jump, `update.sh`, Post Migration Helper outputs, `RUN_SyncDB.sh` stopping on the first error, Verify Migration's four checks, duplicate-column/ID and view-overwrite remedies (Wayback capture, 2026-06-10).
- **Version trail**: the full archive from 1.0a (2012-10-31) to 12 "Kudos" (2024-12-24), and "Daily 13 Orion (released 2026-03-06)"; the 6.2 date discrepancy (idempiere.org/releases, 2026-10-17).
- **Lineage and history**: Compiere 1999 (Jorg Janke, ex-Oracle); the 2006 VC and editions; 3.3 (2009); the Consona acquisition (2010); ADempiere fork September 2006, 12 Oct 2006 infobox, GPLv2, dictionary architecture, Java EE/Tomcat/JBoss, PostgreSQL+Oracle only, 3.9.4 Jan 2023, the council governance; the 2008–2010 Felix/Equinox modular proposal; 2010 LTS and Branch GlobalQSS; 2011 the iDempiere name; the platform changes per version (Wikipedia, 2026-10-17).
- **Governance and community**: "no single person or company ruling the project"; the local communities and channels; the conference attendance records; the awards and the existence/dates of the listicles (Wikipedia, 2026-10-17).
- **Test sites and runtime**: demo.globalqss.com (13 LTS, Colombia localisation + REST, openjdk 17/postgresql 18/Ubuntu 22.04), test.idempiere.org (14 dev), test-oracle.idempiere.org (14 dev, Oracle 18c XE) and the unsecured-sites warning (idempiere.org, 2026-10-17).
- **The repository-gap count**: exactly three files before this guide (this repo, 2026-10-17).
- **The document's own structure**: 16 numbered sections covering the required ground, a glossary, a claims audit and a verification ledger, with the required closing line as the final line of the file.
### Flagged / not fully verified (⚠)
- **The project wiki is Cloudflare-blocked** ("Just a moment…") — including the Plugin page and the Application Dictionary pages; recorded as a tool limitation, not as absence of material.
- **Several cross-referenced documentation pages were not in the fetched set** (Running Plug-ins Locally; the v9 new-feature pages; the v14 extension-management page; removed-deprecated-apis; the clustering docs) — claims depending on them are attributed to the citing page.
- **Three conflicting dates for 13 "Orion"** (2026-03-06 / 2026-03-12 / 9 March 2026) — all three recorded.
- **The 6.2 date disagrees** (2019-01-06 vs 2019-01-04) — recorded, not resolved.
- **The Java/PostgreSQL/Oracle version floors differ between the project's own pages by vintage** — presented as a dated table; no single authority established.
- **The language count disagrees within Wikipedia** (16 vs 29 localizations).
- **The clustering/HA topology was not verified** — the Hazelcast and Redis service bundles exist; what they implement is not claimed.
- **The AI-agent contribution workflow** (`.agents/skills`, `.claude`, `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `QWEN.md`, `.coderabbit.yaml`) exists per the listing; the content of the "using AI agents" page was not read.
- **The Jenkins pipeline contents** were not read; the claim is that the project states Jenkins runs its CI.
- **The Docker `12-release` tag** was read as a documentation-vintage detail; whether a 13-tagged image is published was not verified.
- **"Tier II"** is quoted from the project's strapline; the project does not define it on the page read.
- **The 2006-era Compiere popularity figures and every listicle ranking are deliberately not printed** — flagged as secondary/promotional and excluded per the house rule.
- **The worked example (§12) is fictional** — an illustrative scenario, not a record of any project.
- **No adoption figure, market share, performance benchmark or ranking is printed anywhere in this guide** — a deliberate omission, recorded as claim #20 of §14.

---

*End of guide — the dictionary is the structure, the plug-in is the behaviour, and the seam between them is the craft. Last updated October 2026. Companion guides: [Odoo Framework](odoo_framework_guide.md), [OSGi](osgi_guide.md), [Asia Fusion Technology](asia_fusion_technology_guide.md), [T24 Programming](../banking/t24_programming_guide.md), [Business Case Development](../management/business_case_development_guide.md).*

in iDempiere, the model, the screen and the database table are three renderings of one dictionary entry.
