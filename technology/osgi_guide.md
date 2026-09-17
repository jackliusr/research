# OSGi: Comprehensive Guide

> The Java Platform's Modularity and Component Runtime — Bundles, the Module/Lifecycle/Service Layers,
> Declarative Services, the Compendium and Enterprise Specifications, the Runtime Containers
> (Felix, Equinox, Karaf), the Tooling Chain (bnd, PDE), the JPMS Comparison, and the Adoption Reality
>
> Compiled: September 2026

**Jack Liu Shurui, Solution Architect**

**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research) — **technology/** series

**Primary sources read for this pass (all retrieved 2026-09-17):** the specification documents themselves at `docs.osgi.org` (Core R3/R4/R4.1/R4.2/R4.3/R5/R6/R8 PDFs, Compendium R7 and R8.1, Enterprise R4.2/R5/R6/R7 PDFs, and the Consolidated Release listing); the OSGi Specification Project and OSGi Technology Project project-management pages at `projects.eclipse.org`; the OSGi Working Group activity metrics at `metrics.eclipse.org`; the Eclipse Foundation newsroom announcement feed for the OSGi working group; `osgi.org`; the Apache projects' own sites and release catalogues (`karaf.apache.org`, `felix.apache.org`, `sling.apache.org`, `tooling-bat.apache.org` — the ASF release catalog, `downloads.apache.org`); the Apache Aries board report series; `download.eclipse.org/equinox/`; the bnd/bndtools release list on GitHub; `enroute.osgi.org` and the enRoute repository; the JCP records for JSR 277, JSR 291, JSR 294 and JSR 376; OpenJDK JEP 411 and JEP 486; and vendor documentation from Liferay and Adobe. Blog posts, listicles and conference talks were **not** treated as evidence. Live web access (self-hosted extraction backend) was working throughout this pass; `web_search` returned empty results intermittently and extraction was used directly.

**Integrity convention used in this guide:** **(verified** — primary source, date read) = established at a primary source this pass; **(flagged** — …) = attributed or unverified, with what was checked stated; **(flagged ⚠-structural** — …) = standard industry practice, widely documented, not attributable to a single primary source. Where the specification *mandates* something, this guide says "the specification requires"; where an *implementation* does it, it names the implementation; where a *blog post claims* it, it says so.

---

**How to use this guide:** Section 1 is the overview — what OSGi is, what it is not, and the framing note that this is the repository's **first guide into the Java platform layer at all**. Section 2 is the architecture — the layers the specification defines. Section 3 is the bundle — the unit of deployment and its manifest. Section 4 is the module layer — resolution, wiring and class loading, which is where OSGi projects actually succeed or fail. Section 5 is the lifecycle layer. Section 6 is the service layer. Section 7 is Declarative Services and the competing component models. Section 8 is the Compendium and Enterprise service portfolio. Section 9 is the **release history**, verified date by date. Section 10 is the **governance and current state question** — the highest-risk fact in the OSGi discourse, answered from primary sources only. Section 11 is **platforms that embed OSGi**, including the boundary against the repository's own Talend, Nexus and Jetty guides, and the Talend Runtime version-alignment constraint that lives in section 11.2. Section 12 is the runtime containers. Section 13 is the tooling chain. Section 14 is the **OSGi versus JPMS comparison**, argued from the JSR records rather than from opinion. Section 15 is operations. Section 16 is when OSGi is the wrong choice. Section 17 is an illustrative worked example in the house's Cymbal Bank context. Section 18 is the one-page summary **and the claims ledger**. The glossary (19) and the cross-references and further reading (20) close the file.

Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` and `management/` are prefixed `../banking/` and `../management/`; guides in the `ai_llm/` subdirectory are prefixed `ai_llm/`. **Every cross-reference path in this file was checked to exist on disk on 2026-09-17 before being written.**

---

**Table of Contents**

1. [The OSGi Overview](#1-the-osgi-overview) — what OSGi is, the framing note, the overview table, the three sources of truth
2. [The Architecture](#2-the-architecture) — the specification's layers, the layer table, the container picture
3. [The Bundle](#3-the-bundle) — a JAR with a contract, the manifest headers, versioning, repository coordinates
4. [The Module Layer — Resolution, Wiring and Class Loading](#4-the-module-layer--resolution-wiring-and-class-loading) — why it fails here, the resolver, the wiring, fragments and extensions
5. [The Lifecycle Layer](#5-the-lifecycle-layer) — states and verbs, start levels, the launch API, the real cost of hot deployment
6. [The Service Layer](#6-the-service-layer) — the registry, registration and tracking, stale references
7. [Declarative Services and the Component Models](#7-declarative-services-and-the-component-models) — why DS exists, the DS version line, the competing models
8. [The Compendium and Enterprise Services](#8-the-compendium-and-enterprise-services) — the service portfolio, the enterprise chapters in R8
9. [The Release History](#9-the-release-history) — the dated release table, minor revisions versus major releases
10. [The Governance and Current State](#10-the-governance-and-current-state) — the headline finding, the evidence trail, 2026 activity, the verdict
11. [Platforms That Embed OSGi](#11-platforms-that-embed-osgi) — the adoption table, the Talend Runtime boundary, the Nexus and Jetty boundaries
12. [The Runtime Containers](#12-the-runtime-containers) — frameworks versus containers, the dated container table, the dormant tail
13. [The Tooling Chain](#13-the-tooling-chain) — bnd, the build plugins and IDEs, testing
14. [OSGi versus JPMS](#14-osgi-versus-jpms) — the JSR records, the comparison table, the conclusion
15. [Operations](#15-operations) — security, observability, failure modes and signatures, upgrade discipline
16. [When OSGi Is the Wrong Choice](#16-when-osgi-is-the-wrong-choice) — the disqualifiers and the decision table
17. [The Worked Example — Cymbal Bank](#17-the-worked-example--cymbal-bank) — the scenario, the design, alignment and rollback, what went wrong first
18. [The One-Page Summary and the Claims Ledger](#18-the-one-page-summary-and-the-claims-ledger) — the summary and what could not be verified
19. [Glossary](#19-glossary)
20. [Cross-References and Further Reading](#20-cross-references-and-further-reading)

---

## 1. The OSGi Overview

### 1.1 What OSGi Is

OSGi is two things that are constantly confused with each other.

**It is a specification** — a family of documents published by a working group, defining a modular Java runtime: a module layer with versioned dependencies and a resolver, a lifecycle layer for installing and updating modules at runtime, and a service layer with a registry that connects components that never see each other's code. The specification is expressed as three principal document sets — **Core** (the framework, module, lifecycle, service and security layers plus the framework API), **Compendium** (the standard services), and **Enterprise** (the enterprise-grade services, subsystems and provisioning) **(verified** — `docs.osgi.org/specification/`, Consolidated Release listing, read 2026-09-17).

**It is a set of implementations** — concrete frameworks that implement the specification, plus containers, tooling, repositories and test harnesses around them. Apache Felix, Eclipse Equinox, Apache Karaf, Apache Aries, bnd and bndtools are the ones that still matter in 2026 **(verified** — see sections 12 and 13 for each one's dated status).

The name is historical. **OSGi** originally stood for **Open Services Gateway Initiative** — a 1999 standardisation effort aimed at service gateways in homes, vehicles and industrial systems, founded in March 1999 **(verified** for the corroborating primary document — the OSGi Service Platform Release 3 specification's own member-company and board listing, dated March 2003, names Motorola, IBM, BMW, Deutsche Telekom and Nokia among its directors: `docs.osgi.org/download/r3/r3.book.pdf`, front matter, read 2026-09-17. Wikipedia's OSGi article is the aggregator for the 1999 founding date and is not treated as an authority in this guide). The acronym has long since stopped describing the scope: the specification now targets server-side and cloud-native applications as explicitly as embedded ones **(verified** — the OSGi Specification Project's own statement of purpose at `projects.eclipse.org/projects/technology.osgi`, read 2026-09-17).

**The one-sentence version:** OSGi is the Java platform's answer to a JAR file that has to declare what it needs, what it offers, and what happens when either changes while the system is running.

### 1.2 The Framing Note — This Repository's First Java-Platform Guide

**This is the first guide in this repository that addresses the Java platform layer at all.** As of September 2026 the repository holds roughly 589 markdown guides and, before this file, none of them was an OSGi guide and none was a Java/JVM-platform guide. The repository's *language* guides cover Rust, Solidity, Vue and JSON/YAML/TOML. Its only Java-adjacent material is (a) **product** guides for Java-based products, and (b) an RxJava guide **(flagged** — assessed by searching the repository tree for `osgi` and for Java-platform topic guides on 2026-09-17; the count of ~589 guides is the repository's own order of magnitude, not a file-by-file census — flagged accordingly).

That has a consequence for how to read this guide: there is no sibling guide to cross-reference for JVM fundamentals. Everything the reader needs about bundles, class loading, and module resolution is stated in this file, and the cross-references at the end of this guide point at *adjacent* concerns (integration platforms, container building, legacy integration patterns) rather than at Java-platform prerequisites.

The repository's existing OSGi material is **component-level fact inside product guides** — not architecture:

| Appears in | As | What this guide does with it |
| --- | --- | --- |
| [talend_guide.md](talend_guide.md) | Talend Runtime described as "the **OSGi/Apache-Camel-based container** that runs Routes, Data Services and Microservices", with a documented Studio/Runtime version-alignment constraint | **Cross-referenced in section 11.2, not re-derived.** The constraint is cited there as the concrete, vendor-documented instance of the general alignment rule this guide develops in section 15.4 |
| [sonatype_nexus_guide.md](sonatype_nexus_guide.md) | Nexus Repository described as a "**Java (OSGi) server**" | **Cross-referenced in section 11.3.** Not independently verified at the vendor's documentation this pass — recorded as an attributed claim |
| [jetty_guide.md](jetty_guide.md) | An OSGi support row in the container comparison table, and an "**OSGi environment**" row in the selection table (Jetty as an OSGi HTTP service host) | **Cross-referenced in section 11.3.** Same handling — attributed, not re-verified |
| `ai_llm/deep_learning_frameworks_comparison_guide.md` | Contains `Equinox` hits | **Explicitly excluded.** Those hits are the **Equinox JAX neural-network library**, not Eclipse Equinox. This is a **false positive** and is *not* cited anywhere in this guide as evidence |

### 1.3 The Overview Table

| Aspect | OSGi's position | Confidence |
| --- | --- | --- |
| **What it standardises** | A runtime module system with versioned dependencies, a lifecycle, and a service registry — specification text, not product | **(verified** — Core R8 and Compendium R8.1 tables of contents, `docs.osgi.org`, read 2026-09-17) |
| **First release** | Release 1, "OSGi Service Gateway Release 1 Specification" — the earliest document still hosted | **(flagged** — the R1 PDF is listed at `docs.osgi.org/specification/` but its publication date was **not** re-verified this pass; do not quote a year for R1 from this guide) |
| **Founding of the alliance** | March 1999 | **(verified** — Wikipedia OSGi + R3 specification front matter, read 2026-09-17) |
| **Governing body today** | The **OSGi Working Group at the Eclipse Foundation**, hosting the OSGi Specification Project and the OSGi Technology Project | **(verified** — `osgi.org` now serves the Eclipse-hosted working group; `projects.eclipse.org/working-group/osgi`, read 2026-09-17) |
| **Latest published specification release** | **OSGi Compendium Release 8.1** (December 2022) — the most recent full specification publication | **(verified** — `docs.osgi.org/specification/osgi.cmpn/8.1.0/`, copyright "© 2000, 2022 Eclipse Foundation", December 2022, read 2026-09-17) |
| **Latest specification-project release of any kind** | **2.0 Jakarta RS Whiteboard**, dated 2022-12-20 | **(verified** — the OSGi Specification Project's release table at `projects.eclipse.org/projects/technology.osgi`, read 2026-09-17) |
| **Project state at Eclipse** | **Incubating** — for both the Specification Project and the Technology Project | **(verified** — the "State" field on both project pages, read 2026-09-17) |
| **Draft specification work** | Continuing: a **Core Draft** and a **Compendium Draft** are built and published from `github.com/osgi/osgi` | **(verified** — `github.com/osgi/osgi` README ("The GitHub Actions build 'GitHub Pages' job will publish the draft specifications into the repo's gh-pages branch") and the Development Draft entries at `docs.osgi.org/specification/`, read 2026-09-17) |
| **Language** | Java, exclusively. There is no non-JVM OSGi specification | **(verified** — every specification document and API namespace is Java; the JSR 291 record describes the framework as "a dynamic component framework ... for existing Java SE platforms", `jcp.org`, read 2026-09-17) |
| **Licence of the specification** | OSGi Specification License v1.0/v2.0 on the Alliance-era documents; Eclipse Foundation Specification License v1.0 on the Eclipse-era Compendium 8.1 | **(verified** — `r7/osgi.enterprise-7.0.0.pdf` and `osgi.cmpn/8.1.0/LICENSE.html`) |

### 1.4 The Three Sources of Truth — Specification, Implementation, Blog Post

The single most useful discipline when reading about OSGi is to ask which of three registers a claim is in. They are not interchangeable, and the OSGi literature is unusually full of the third masquerading as the first.

1. **The specification mandates.** "The framework must resolve a bundle's requirements against the capabilities available in the repository" is a normative statement about *every* compliant framework. It is found in Core, in the module layer chapter, in normative language — `must`, `should`, `may` **(verified** — Core R8 §3 table of contents: "Resolving Metadata", "Constraint Solving", "Resolving Process", "Runtime Class Loading", `docs.osgi.org/download/r8/osgi.core-8.0.0.pdf`, read 2026-09-17).
2. **An implementation provides.** "Felix Framework 7.0.5 does this" or "Equinox ships this bundle" is a statement about one artefact at one version. Felix's own documentation disagrees with itself on which Core release its framework implements — the Felix subprojects page describes the Framework as "an implementation of the OSGi **R8** core framework" while the project's GitHub repository description says the flagship `framework` directory "implements the OSGi Core **R7** specification", and the last release of the R7 line is **7.0.5** (3 June 2022) **(verified — both texts and the release date read 2026-09-17; the discrepancy is recorded rather than resolved, see section 12.2)**.
3. **A blog post claims.** "OSGi is dead." "OSGi was superseded by JPMS." "Everyone moved off OSGi years ago." These appear constantly and are almost never sourced to a specification, a project page, a release feed or a vendor document. This guide treats them as claims requiring evidence, and section 14.3 states exactly what the primary records do and do not support.

**The practical rule:** if a statement about OSGi cannot be traced to a numbered specification release, a dated release note, a project page, or a vendor's own documentation, it is a claim about a *conversation*, not a fact about a *technology*.

---

## 2. The Architecture

### 2.1 The Layers the Specification Defines

The Core specification is organised — from Release 4.2 onward, and consistently through Release 8 — into named layers. This is not a metaphor invented by the community; it is the table of contents of the document **(verified** — the Core R8, R6, R5, R4.3 and R4.2 PDFs each open their normative part with "Security Layer", "Module Layer", "Life Cycle Layer" and "Service Layer" as chapters 2, 3, 4 and 5, followed by the Framework API and the service specifications; read 2026-09-17).

The layering matters because it maps to *responsibility*, and responsibility maps to *failure modes*:

- **Module** decides what can be loaded and what it is allowed to see; it fails at resolution, visibly, before anything runs. **Lifecycle** decides when things start, stop and are replaced; it fails on ordering and half-applied updates.
- **Service** decides how components find each other without a compile-time dependency; it fails on stale references and timing races. **Security** decides what code may do; its failure mode is that the mechanism it depends on is being removed underneath it on modern JDKs (section 15.1).

### 2.2 The Layer Table

| Layer | What the specification makes it responsible for | Representative artefacts | Where it fails |
| --- | --- | --- | --- |
| **Security** | Digitally signed JAR files; permissions granted to code; constraining bundle functionality | Signing, permission administration | Depends on the JDK Security Manager, which is deprecated for removal **(verified** — Core R8 chapter 2 "Security Layer", "Digitally Signed JAR Files", "Permissions"; JEP 411, Java 17, status Closed/Delivered; JEP 486 "Permanently Disable the Security Manager"; read 2026-09-17) |
| **Module** | Bundles; dependencies; execution environment; class loading architecture; resolving metadata; constraint solving; the resolving process; runtime class loading; native code; localisation; bundle validity; requiring bundles; fragments; extensions | `Import-Package`, `Export-Package`, `Bundle-SymbolicName`, the resolver | Unresolvable requirements; `uses` constraint conflicts; duplicate class loading; the system does not start |
| **Life Cycle** | Frameworks; bundles; the `Bundle` object; the `BundleContext`; the system bundle; events; framework startup and shutdown; start levels | `BundleActivator`, install/start/stop/update/uninstall, `StartLevel` | Ordering; partially applied updates; listeners left behind after an update |
| **Service** | Services; service events; stale references; filters; service factories; releasing services; multiple-version export considerations | `BundleContext.registerService`, `ServiceReference`, `ServiceRegistration`, `ServiceTracker` | Timing races; leaked service objects; staleness on update |
| **Services (beyond the Core)** | The service portfolio — published in the Compendium and, historically, the Enterprise specification | Configuration Admin, Log, Http, Declarative Services, Remote Services, and the rest of section 8 | Configuration drift; service-level vulnerabilities; version skew against the container |

**(verified** — chapter names and section names above are taken from the Core R8 table of contents and, for the framework-launch package, from the Core R4.2 table of contents which lists `org.osgi.framework.launch` as §6.2. The "multiple-version export considerations" entry is §5.9 of Core R4.2 — see section 14 of this guide for why that specific section is important. `docs.osgi.org`, read 2026-09-17.)

### 2.3 The Container Picture

Below the specification layers sits the **framework** — the runtime that implements them — and around the framework sits a **container** that adds operational furniture: shell, deployment, provisioning, configuration, logging, clustering.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  Container (e.g. Apache Karaf)                               │
   │  shell · deployers · features · instances · clustering · JMX  │
   ├──────────────────────────────────────────────────────────────┤
   │  Framework (e.g. Apache Felix, Eclipse Equinox)              │
   │  Security │ Module │ Life Cycle │ Service │ Framework API     │
   ├──────────────────────────────────────────────────────────────┤
   │  JVM — Java SE, per the framework's required execution env.  │
   └──────────────────────────────────────────────────────────────┘
```

The distinction is not pedantic. A **framework** is what the compliance programme certifies and what implements the specification's layers. A **container** is a product decision — Karaf, for example, describes itself as "the modulith runtime for the enterprise" and packages a console, remote access, hot deployment, dynamic configuration, provisioning, clustering (Cellar) and monitoring (Decanter) around a framework, offering a choice of **Felix Framework or Equinox** underneath **(verified** — `karaf.apache.org` describes Karaf as an "OSGi runtime" and a "modulith runtime" and lists shell/JMX/configuration/provisioning among its enterprise features; the Karaf 4.4.1 release notes record the upgrade of "OSGi frameworks (both Felix Framework 7.0.5 and Equinox 3.18.0)", which documents the dual-framework option; read 2026-09-17).

**Two consequences for anyone designing on OSGi:**

1. **The framework is a versioned dependency of the container, not of your application.** Which Core release level you actually get is decided by the container's release notes, not by your POM. Karaf 4.4.11's release records a bump of `org.osgi:osgi.core` from 7.0.0 to 8.0.0 — visible proof that the Core level moved with the container **(verified** — `github.com/apache/karaf/releases`, tag `karaf-4.4.11`, released 27 April 2026; read 2026-09-17).
2. **Everything above the framework is optional.** Declarative Services, Remote Services, Configuration Admin, the HTTP whiteboard — all of them are bundles that must be *present* in the runtime for the feature to exist. "OSGi supports X" and "your runtime has X installed" are different statements.

---

## 3. The Bundle

### 3.1 A JAR With a Contract

A **bundle** is a JAR file whose manifest declares what it provides and what it needs. Nothing more exotic than that: the packaging is standard, the loader behaviour is standard, and the extra semantics all live in manifest headers **(verified** — Liferay's developer documentation states that a module's folder contains "a `bnd.bnd` configuration file" with `Bundle-SymbolicName`, `Bundle-Version`, `Export-Package` and `Bundle-Name` entries, and that "only a module's classes whose packages are explicitly exported are publicly visible; OSGi hides all other classes by default"; `learn.liferay.com`, read 2026-09-17).

That last clause is the whole point. On a **classpath**, everything on it is visible to everything else and the ordering is incidental. In a **bundle**, invisibility is the default and visibility is an explicit declaration.

### 3.2 The Manifest Headers That Matter

| Header | Purpose | Notes |
| --- | --- | --- |
| `Bundle-SymbolicName` | The bundle's stable identity | "a name that ensures uniqueness" **(verified** — Liferay developer documentation, read 2026-09-17) |
| `Bundle-Version` | The bundle's own version | Semantically versioned **(verified** — same source, which links a dedicated semantic-versioning page) |
| `Bundle-Name` | Human-readable name | Same source |
| `Export-Package` | The packages other bundles may import | "Only a module's classes whose packages are explicitly exported are publicly visible" **(verified** — same source) |
| `Import-Package` | Packages this bundle requires, with version constraints | The specification's own examples show the range syntax, e.g. `Import-Package: org.osgi.util.function; version="[1.0,2.0)"` **(verified** — OSGi Enterprise Release 6 specification, Promises chapter API examples, `docs.osgi.org/download/r6/osgi.enterprise-6.0.0.pdf`, read 2026-09-17) |
| `Require-Bundle` | A whole-bundle dependency instead of per-package | Defined as a distinct dependency mechanism in the module layer **(verified** — Core R8 §3.13 "Requiring Bundles"; same section exists in R4.0 as §3.13 "Requiring Bundles", `docs.osgi.org`, read 2026-09-17) |
| `Bundle-Activator` | The class whose `start`/`stop` the lifecycle layer calls | The procedural entry point that Declarative Services exists to make unnecessary (section 7.1) — **(verified** — the Enterprise R7 Declarative Services chapter explicitly lists "bundles need not provide a `BundleActivator` class" among the benefits and cites "the procedural service model requires a bundle to actively register and acquire its services" as the problem; read 2026-09-17) |
| `Fragment-Host` | Marks the bundle as a fragment, attaching to a host bundle | **(verified** — Core R8 §3.14 "Fragment Bundles") |

### 3.3 Versioning — Semantic and Enforced

OSGi's versioning story is unusual in that it is *enforced at resolution time* rather than merely documented:

- **Version ranges are expressed in the import, and the resolver honours them.** `[1.0,2.0)` means "one point oh inclusive, two point oh exclusive". A bundle requiring a range that no provider satisfies **does not resolve**, and an unresolved bundle does not start **(verified** — `Import-Package` range syntax appears in the OSGi Enterprise Release 6 specification's own API examples; resolution and constraint solving are normative module-layer chapters in every Core release from R4.2 onward; read 2026-09-17).
- **Multiple versions can coexist.** Two applications that need different versions of the same package can each be wired to their own provider, because resolution happens per-package and per-consumer, not globally. Liferay's developer documentation states this as a platform feature: modules "declare dependencies on specific versions of other packages. This allows two applications that depend on different versions of the same packages to each depend on their own versions of the packages" **(verified** — `learn.liferay.com`, read 2026-09-17).
- **This was not always true, and the record shows it.** The withdrawn JSR 277, written in the mid-2000s, argued that the then-current OSGi R3 framework could not support multi-version packages: "the framework must choose one bundle that will be the provider of the exported package for all bundles which have dependencies on that package, so it is impossible to support more than one version of shared package at runtime" **(verified** — JSR 277 request text, §2.6, `jcp.org/en/jsr/detail?id=277`, read 2026-09-17). By Release 4 Version 4.2 the Core specification carried a dedicated section titled "**5.9 Multiple Version Export Considerations**" **(verified** — Core R4.2 table of contents, `docs.osgi.org/download/r4v42/r4.core.pdf`, read 2026-09-17). **The criticism was accurate about R3 and stale by R4.2.** This is a small case study in why dating an OSGi criticism matters.

### 3.4 Bundle Identity and Repository Coordinates

Because bundles must be *found* before they can be resolved, the specification eventually standardised how a bundle is addressed. In R8, identifiers "are defined using the **Maven Identifier model**" with the format `groupId ':' artifactId ( ':' type ( ':' classifier )? )? ':' version`, and a bundle or feature may be located by that identifier in an artifact repository **(verified** — Compendium R8.1 §159.2.1 "Identifiers" and §159.2.2 "Feature Identifier", read 2026-09-17).

The practical reading: **OSGi converged on Maven coordinates for addressing artefacts**, while keeping its own resolution semantics on top. Repositories are therefore ordinary Maven (or OBR-style) repositories — which is exactly why the repository's own [sonatype_nexus_guide.md](sonatype_nexus_guide.md) is a relevant neighbour for an OSGi estate (section 11.3).

---

## 4. The Module Layer — Resolution, Wiring and Class Loading

### 4.1 Why This Layer Is Where Projects Fail

Every OSGi project that fails, fails here. The lifecycle and service layers are usually easy to get right; the module layer punishes any imprecision in the manifest, because the resolver takes the declarations literally and the resulting **wiring** is the class-loading graph for the life of the process.

The Core specification devotes four consecutive sections to precisely this problem — "Resolving Metadata", "Constraint Solving", "Resolving Process" and "Runtime Class Loading" — and has done since R4.2 **(verified** — identical section names appear in the R8 and R4.2 tables of contents, `docs.osgi.org`, read 2026-09-17). That the names are unchanged across sixteen years and seven releases is itself informative: this is settled, stable machinery.

### 4.2 The Resolver as a Constraint Solver

The resolver is not a classloader trick; it is an **all-solutions-considered constraint problem over the declared metadata**, and the specification says so by naming the section "Constraint Solving" **(verified** — Core R8 §3.7 "Constraint Solving", Core R4.2 §3.6 "Constraint Solving", read 2026-09-17).

What that means in practice:

| Design decision | What the resolver does with it | Cost of getting it wrong |
| --- | --- | --- |
| Import a **package** (`Import-Package`) | Picks a provider that satisfies the version range, per consumer | The default, and the one that makes independent versioning possible |
| Require a **bundle** (`Require-Bundle`) | Pulls the whole provider and its exports | Coarser coupling; a substrate for the classic "everyone requires everyone" graph |
| Require a **capability** (R5 and later) | Matches generic capabilities/requirements beyond packages | The general mechanism; also the mechanism used for non-package requirements (execution environment, native code, and platform capabilities) |
| Leave an import **optional**, or use **dynamic imports** | Resolution succeeds without a provider, or acquires a package later | The failure moves from startup to runtime — a common source of `NoClassDefFoundError`-shaped surprises when a code path assumes a provider exists |
| Declare an **execution environment** | Constrains the bundle to JVM levels it supports | Under-declaring is how a bundle installs on a JVM it cannot actually run on **(verified** — Core R8 has dedicated "Execution Environment" sections in the module and lifecycle discussions; the concept is normative from R3 onward, where "Execution Environment" is a framework section, read 2026-09-17) |

### 4.3 The Wiring — What Resolution Actually Produces

Resolution produces a **wire set**: for each requirement in each bundle, the specific provider chosen. Two things follow that are worth being explicit about.

1. **The wiring is per-consumer, so it can differ for two consumers of the same package** — which is how multi-version coexistence actually works (section 3.3). It also means "which version am I getting?" has no global answer; it has a per-bundle answer.
2. **The wiring is an artefact you can and should inspect.** Frameworks expose it — Felix's web console and shell, Equinox's console, Karaf's shell, and the standard REST management service all surface bundle and wiring state **(verified** — Felix's subprojects include "Gogo — an advanced shell for interacting with OSGi frameworks", "Web Console — a simple tool to inspect and manage OSGi framework instances", "Inventory Printer — a simple and extensible framework to retrieve inventory information about applications running in an OSGi Framework", and "Health Checks — an extensible framework to monitor the status of the OSGi container at runtime"; `felix.apache.org/documentation/subprojects.html`, read 2026-09-17).

**Guidance for operators:** the first diagnostic question in any OSGi incident is not "is the service up?" but "**what is the resolved wiring?**" — which bundles resolved, which did not, and which provider each requirement was wired to. This is why section 15.3 lists the failure modes by signature rather than by symptom.

### 4.4 Bundles of a Different Kind — Fragments, Extensions, Requiring Bundles

Three module-layer mechanisms depart from the ordinary bundle and each is a lasting source of surprise:

- **Fragment bundles** attach their content to a **host** bundle rather than standing alone. They exist to extend something you cannot modify. The cost is that fragments break the tidy "a bundle owns its own classes" assumption **(verified** — Core R8 §3.14 "Fragment Bundles"; present as §3.13 in R4.0, read 2026-09-17).
- **Extension bundles** attach to the framework's own class space rather than to another bundle's. They are the most invasive thing in the module layer **(verified** — Core R8 §3.15 "Extension Bundles", read 2026-09-17).
- **Requiring bundles** (`Require-Bundle`) is the escape hatch that reintroduces whole-module coupling. Its continued presence in every Core release from R4 to R8 shows the specification accepts that sometimes package-level precision is not worth the effort **(verified** — §3.13/§3.12 across releases, read 2026-09-17).

A useful rule from the specification's own design: **prefer package-level imports; reach for `Require-Bundle` when the dependency really is a whole module's interface, and treat fragments and extensions as mechanisms of last resort.**

---

## 5. The Lifecycle Layer

### 5.1 The States and the Verbs

The lifecycle layer is deliberately small: a handful of states and five verbs. The specification names the verbs in the module's API — **install, start, stop, update, uninstall** — and the lifecycle chapter covers the `Bundle` object, the `BundleContext`, the system bundle, events, and framework startup and shutdown **(verified** — Core R8 chapter 4 "Life Cycle Layer" with sections "Frameworks", "Bundles", "The Bundle Object", "The Bundle Context", "The System Bundle", "Events"; the five-verb formulation appears in the OSGi literature and in the JSR 291 request text which specifies "components can be dynamically life cycle managed (install, start, stop, update, uninstall)"; read 2026-09-17).

| Concept | What it is | Why it matters operationally |
| --- | --- | --- |
| **Bundle state** | Where a bundle is in its life (installed, resolved, starting, active, stopping, uninstalled) | "Resolved" is the state that matters for dependency diagnosis; a bundle that never resolves will never run |
| **`BundleActivator`** | The class the framework calls on `start` and `stop` | The procedural entry point; the reason declarative models exist (section 7) |
| **`BundleContext`** | The handle a bundle uses to talk to the framework | The only legitimate door to the registry and to bundle operations |
| **System bundle and framework events** | The framework itself as bundle 0, plus the lifecycle/service events delivered to listeners | The system bundle's lifecycle *is* the container's; events are how you react to change — and the leak source if listeners are not removed |

**(verified** — all rows are section names of the Core specification's lifecycle chapter, `docs.osgi.org`, read 2026-09-17.)

### 5.2 Start Levels and the Launch API

Two lifecycle features repay attention because they are the ones teams discover late:

- **Start levels** give the runtime an ordered startup discipline. They exist because "start everything at once" produces non-deterministic failures on a cold boot. The Core specification carries a **Start Level Service Specification** as a separate chapter **(verified** — Core R8 chapter 8 and Core R4.2 chapter 8 are "Start Level Service Specification"; read 2026-09-17).
- **The framework launch API** (`org.osgi.framework.launch`) is the standard way to *embed* a framework inside a Java process rather than starting the container's launcher script. It appears in the Core table of contents from Release 4 Version 4.2 onward **(verified** — Core R4.2 §6.2 "org.osgi.framework.launch", `docs.osgi.org/download/r4v42/r4.core.pdf`, read 2026-09-17). This is what makes OSGi usable inside a JUnit test, a Spring Boot application, a build tool, or a container image's `main`.

### 5.3 What Hot Deployment Really Costs

Install/update/uninstall at runtime is OSGi's headline capability, and it is genuine — but it is not free, and the honest description is narrower than the marketing:

- **The specification constrains the update, not the application.** Updating a bundle re-resolves its requirements. If nothing else must change, the update lands cleanly. If a `uses` constraint or a shared package version forces a wider re-resolution, the framework is entitled to refresh more than you asked it to — which is why "Refresh Packages" exists as an explicit operation in production consoles and why Adobe's AEM documentation tells administrators to perform one after an update **(verified** — Adobe's AEM documentation, Bundles tab: "**Refresh Packages** — This checks the references of all the packages and refreshes, as necessary. For example, after an update both the old and new version may still be running due to prior references. This option checks and moves all references to the new version, allowing the old version to stop." and "After **Update**, Adobe recommends that you perform a **Refresh Packages**"; `experienceleague.adobe.com`, AEM 6.5 Web Console, last update 7 May 2026, read 2026-09-17).
- **Old and new can coexist briefly.** The AEM text states this plainly — "after an update both the old and new version may still be running due to prior references" — which is precisely the multiple-version property from section 3.3 showing up as an operational fact **(verified** — same source).
- **Being able to update at runtime does not mean updates are risk-free.** Section 15.4 covers the alignment discipline that makes them safe; section 17 gives the worked example.

**The honest summary:** OSGi genuinely supports changing modules in a running system, and the mechanism is specified and implemented. What it does not do is make change free. Any architecture that treats hot deployment as a licence to skip version discipline will produce the failure modes in section 15.3.

---

## 6. The Service Layer

### 6.1 The Registry as a Broker

The service layer's purpose is to let a **consumer** use a **provider's** implementation without a compile-time dependency on the provider's bundle. The consumer names an interface; the registry supplies whichever registered object matches the consumer's filter and ranking. The specification names the mechanism a **publish/find/bind** model **(verified** — the normative mechanics live in Core §5 "Service Layer" and the registry API is the `org.osgi.framework` package, present from Core R4.2 §6.1 onward; read 2026-09-17. The "publish-find-bind" *summary* phrasing is **(flagged** as community shorthand rather than quoted normative text).

The economics of the registry are the reason OSGi survived its original market: **the dependency graph in the *manifest* is a resolution graph, and the dependency graph between *components* is deferred to runtime by the registry.** A rate adapter can be replaced by a different implementation of the same interface without the consumer's manifest changing at all.

### 6.2 The Mechanics — Registration, Lookup, Tracking

| Mechanism | Type | What it does | Notes |
| --- | --- | --- | --- |
| `BundleContext.registerService` | Registration | Publishes an object under one or more interface names with properties | The primitive everything else builds on |
| **Service properties** | Metadata | Key/value attributes attached to a registration | The basis for filtering and ranking — and the reason `service.ranking` matters **(verified** — `Constants.SERVICE_RANKING` is documented in the Compendium R8.1 Declarative Services chapter's component-property-type annotations, read 2026-09-17) |
| **Filters** | Query | An LDAP-style filter expression selecting services by property | A dedicated section of the core specification **(verified** — Core R4.2 §5.5 "Filters", carried forward through R5/R6/R8; `docs.osgi.org`) |
| `ServiceReference` | Handle | A stable handle to a registration that survives some churn | The safe thing to hold instead of the service object |
| `ServiceFactory` | Lifecycle | Produces a per-consumer service instance | The mechanism behind per-bundle service instances **(verified** — Core R4.2 §5.6 "Service Factory"; the R8.1 DS chapter documents `scope` values `bundle` and `prototype` giving per-consumer and per-request instances, read 2026-09-17) |
| `ServiceTracker` / `ServiceTrackerCustomizer` | Tracking | Tracks services matching a filter, calling the customiser on add/modify/remove | A whole appendix: "**Tracker Specification Version 1.5**" **(verified** — Core R8 and Core R5 and Core R6 API appendices, read 2026-09-17) |

**A design note that saves teams months:** the registry is **dynamic**, and the tracker is the standard answer. Code that calls `getService` once at startup and caches the result is the single most common way to turn a correctly wired OSGi system into an intermittently broken one.

### 6.3 Stale References and the Dynamic Contract

The specification names **stale references** as a first-class concern — they have their own section in the service layer in every Core release from R4.2 onward **(verified** — Core R4.2 §5.4 "Stale References", and the same service-layer structure is carried into R5/R6/R8; `docs.osgi.org`, read 2026-09-17).

A stale reference is what a consumer holds when the provider it was bound to has gone away. The specification's answer is that consumers must not assume the service is still there, and the Declarative Services chapter states the same problem in plainer language when justifying the declarative model:

> "Services can come and go at any time. This dynamic behavior makes the service programming model more complex than more traditional models. **This complexity negatively influences the adoption of the OSGi service model as well as the robustness and reliability of applications because these applications do not always handle the dynamicity correctly.**" **(verified** — OSGi Compendium Release 8.1, §112.1 Introduction, `docs.osgi.org/specification/osgi.cmpn/8.1.0/service.component.html`, read 2026-09-17)

That is the specification admitting, in its own words, that the procedural model is easy to get wrong — which is the argument for section 7.

---

## 7. Declarative Services and the Component Models

### 7.1 Why the Procedural Model Needed a Successor

Declarative Services (DS) exists because the procedural model asks too much of the programmer and too much of the machine. The specification names the three problems explicitly — and they are worth quoting because they are the clearest first-party statement of what OSGi's designers thought was wrong with their own original design:

| Problem | The specification's own words |
| --- | --- |
| **Startup time** | "The procedural service model requires a bundle to actively register and acquire its services. This is normally done at startup time, requiring all present bundles to be initialized with a `BundleActivator`. **In larger systems, this quickly results in unacceptably long startup times.**" |
| **Memory footprint** | "A service registered with the Framework implies that the implementation, and related classes and objects, are loaded in memory. **If the service is never used, this memory is unnecessarily occupied.**" |
| **Complexity** | "Services can come and go at any time... applications do not always handle the dynamicity correctly." |

**(verified** — all three quotations are from OSGi Compendium Release 8.1 §112.1, and the identical text appears in OSGi Enterprise Release 7's Declarative Services chapter, meaning the diagnosis was unchanged between R7 (2018) and R8.1 (2022); read 2026-09-17.)

The declarative model inverts the responsibility: **a component declares what it provides and what it needs, and the Service Component Runtime (SCR) does the registering, the finding, the binding and the reacting.** The specification's own list of the model's essentials is a compact statement of its design goals **(verified** — §112.1.1 "Essentials": backward compatibility, size constraints, **delayed activation**, simplicity, **dependency injection**, reactivity, **annotations**, introspection, read 2026-09-17).

### 7.2 The Declarative Services Versions

DS has its own version line inside the Compendium/Enterprise documents, and the versions are directly verifiable from the published specifications:

| DS version | Published in | What the version corresponds to |
| --- | --- | --- |
| **1.2** | OSGi Enterprise Release 5 (March 2012) | **(verified** — "Declarative Services Specification Version 1.2" appears throughout `docs.osgi.org/download/r5/osgi.enterprise-5.0.0.pdf`, read 2026-09-17) |
| **1.3** | OSGi Enterprise Release 6 (July 2015) | **(verified** — "Declarative Services Specification Version 1.3" throughout `docs.osgi.org/download/r6/osgi.enterprise-6.0.0.pdf`, read 2026-09-17) |
| **1.4** | OSGi Compendium Release 7 (April 2018) **and** OSGi Enterprise Release 7 (December 2018) | **(verified** — "Declarative Services Specification Version 1.4" throughout `docs.osgi.org/download/r7/osgi.cmpn-7.0.0.pdf` and `docs.osgi.org/download/r7/osgi.enterprise-7.0.0.pdf`; the R7 chapter already documents the "Component Property Type" annotations marked "Since 1.4", read 2026-09-17) |
| **1.5** | OSGi Compendium Release 8.1 (December 2022) | **(verified** — the Compendium 8.1 Declarative Services chapter is headed "**Version 1.5**" at `docs.osgi.org/specification/osgi.cmpn/8.1.0/service.component.html`, read 2026-09-17) |

Two observations from the table, both safely inferable from it:

1. **DS advanced one minor version per specification release, roughly every three years, and reached 1.5 in 2022.** There is no DS 1.6 in any published document this pass located.
2. **The R8.1 chapter documents three injection styles — method, field and constructor injection** — and the "Since 1.4" annotations are generic component-property types **(verified** — §112.1.1 "Dependency Injection ... supports three types of dependency injection: method injection, field injection, and constructor injection", and the `@ServiceRanking`/`@ServiceVendor` annotations marked "Since 1.4", read 2026-09-17). Constructor injection arriving in the DS line is a good marker of when the model stopped being "annotations over setters" and became a modern DI surface.

### 7.3 The Competing Component Models

DS is not the only component model in the ecosystem, and the differences are architecturally meaningful:

| Model | Where it lives | Status and shape |
| --- | --- | --- |
| **Declarative Services (DS)** | Specification — Compendium/Enterprise | The standard. Version 1.5 in Compendium 8.1 (December 2022). Implemented by **Felix SCR**, by Equinox's DS implementation, and by others **(verified** — Felix's active subprojects list includes "**Service Component Runtime** — An implementation of the OSGi Declarative Services specification providing a service-oriented component model to simplify OSGi-based development"; read 2026-09-17) |
| **Blueprint** | Specification — Enterprise | The container-managed, XML/annotation-driven model originally designed to bring Spring-style wiring into OSGi. Implemented by **Apache Aries Blueprint**, still published and still installable into Karaf with a feature **(verified** — the Aries download page lists `org.apache.aries.blueprint` on Maven Central and states "Apache Karaf is a modular OSGi server that integrates very well with Aries. Most aries subprojects can be installed into Apache Karaf using features. For example **feature:install jndi** installs the Aries jndi modules."; the Karaf 4.4.11 release notes record a bump of `org.apache.aries.blueprint:blueprint-maven-plugin-annotation`; read 2026-09-17) |
| **Felix Dependency Manager** | Implementation-only (Apache Felix) | An API-based component model — "An API-based component model to simplify OSGi-based development" **(verified** — Felix active subprojects list, read 2026-09-17). **Implementation, not specification** — a distinction that matters because choosing it is a vendor decision, not a standards decision |
| **iPOJO** | Implementation-only (Apache Felix) | **Retired.** Listed under the Felix site's "Retired projects" — "no longer maintained" **(verified** — `felix.apache.org/documentation/subprojects.html`, "Retired projects" list includes iPOJO, Deployment Admin, Maven SCR Plugin, Preferences Service, Serialization Framework, User Admin and others; read 2026-09-17) |
| **Hand-written `BundleActivator`** | Core only | Always available, always legal, and the thing DS exists to remove — see §7.1 |

**The recommendation, stated plainly:** default to **Declarative Services** (it is specified, it is implemented by more than one vendor, it is what the specification's own documentation teaches, and it is what Liferay's platform documentation uses in its worked examples — `@Component(service = Greeter.class)` and `@ProviderType`). Use **Blueprint** where an enterprise integration estate already depends on it. Treat implementation-specific component models (Dependency Manager) and retired ones (iPOJO) as deliberate dependency choices, not as defaults **(verified** — for the DS-in-practice claim: Liferay's developer documentation shows a working example using `@Component` and `@ProviderType` with `bnd.bnd` metadata; for the Aries/Karaf relationship: the Aries download page quoted above; read 2026-09-17).

---

## 8. The Compendium and Enterprise Services

### 8.1 The Service Portfolio

The specification does not stop at the framework. It publishes a portfolio of services, and knowing which ones exist prevents teams from hand-rolling something that is already specified. The following are all confirmed as chapters of the Compendium or Enterprise documents in the releases named **(verified** — chapter numbering and titles read from the tables of contents of `osgi.enterprise-7.0.0.pdf`, `osgi.enterprise-6.0.0.pdf`, `osgi.enterprise-5.0.0.pdf`, `osgi.enterprise-4.2`, `osgi.cmpn-7.0.0.pdf` and the `osgi.cmpn/8.1.0/` HTML index, read 2026-09-17):

| Service | Chapter | What it gives you |
| --- | --- | --- |
| **Remote Services** | §100 | The distributed-service model — service properties, intents, configuration types, security. The specification's own chapter opens with "The Fallacies" |
| **Log Service** | §101 | The logger interface, per-bundle logger configuration, the log **stream provider**, log reader |
| **Http Service** | §102 | Registering servlets and resources into the framework's HTTP endpoint, with authentication and MIME handling |
| **Device Access (§103); XML Parser / JAXP (§702); Residential, DMT, TR-069, oneM2M, ZigBee, EnOcean (R6/R8 Compendium)** | §103, §702, device-management chapters | The original embedded-era machinery (devices, drivers, driver locators); the XML parser service with a JAXP bridge; and the device-management/IoT lineage, still specified |
| **Configuration Admin** | §104 | Persistent configuration with a **persistent identity (PID)**, managed services and factories, configuration events, configuration plugins, metatyping, coordinator support |
| **Preferences (§106), User Admin (§107), Wire Admin (§108)** | §106–§108 | The legacy trio: a hierarchical preference tree; users, groups, roles, authentication/authorisation and the JAAS relationship; and producer/consumer wiring between two services — the oldest service-composition mechanism in the specification |
| **Declarative Services** | §112 | Section 7 above |
| **Subsystems** | Enterprise specification §134 ("Subsystem Service Specification") | The grouping and provisioning model for composing bundles into larger deployable units. **Introduced in Enterprise R5 (March 2012)** — the term does not appear at all in the Enterprise R4.2 document **(verified** — "134 Subsystem Service Specification" is a chapter of `docs.osgi.org/download/r5/osgi.enterprise-5.0.0.pdf` and appears in the R6 and R7 Enterprise documents, while a full-text search of `docs.osgi.org/download/r4v42/r4.enterprise.pdf` returns **zero** occurrences of "Subsystem"; read 2026-09-17) |
| **Promises** | §705 | Java-8-style promises for asynchronous work |
| **Converter** | §707 | Standard conversions between DTOs, maps, JavaBeans, types and annotations |
| **Feature Service** | §159 | **Version 1.0** — Features as JSON documents or via a Feature API: bundles, configuration, extensions, properties, composable and reusable; the specification explicitly notes the API is useful "outside of an OSGi Framework where no service registry is available, for example in a build-system environment" **(verified** — Compendium R8.1 §159, read 2026-09-17) |

**Two things worth noting about this portfolio.** First, the **Feature Service** (R8.1, version 1.0) is the newest significant addition and it directly addresses the thing that made OSGi hard to deploy: describing "the application" rather than "these forty bundles". Second, the older **Residential** and **Mobile** documents show where the specification came from — Release 4 still shipped a separate *Mobile* specification and R6 shipped a *Residential* one **(verified** — the Consolidated Release listing shows "OSGi Service Platform Mobile Release 4 Version 4.0 Specification" under R4 v4.0.1 and "OSGi Residential Release 4 Version 4.3" under R4 v4.3, plus "OSGi Residential Release 6" and a "Residential Device Management Tree" chapter in the Compendium; read 2026-09-17).

### 8.2 The Enterprise Chapters and Their Fate in R8

The **Enterprise specification** arrived with Release 4 Version 4.2, whose own front matter is dated **March 2010** and whose title page reads "OSGi Service Platform Enterprise Specification" **(verified** — `docs.osgi.org/download/r4v42/r4.enterprise.pdf`, read 2026-09-17). Enterprise releases followed for R5 (March 2012), R6 (July 2015) and R7 (December 2018) **(verified** — front matter of each PDF, read 2026-09-17).

**There is no Enterprise Release 8.** The Consolidated Release listing at `docs.osgi.org/specification/` shows Core 8.0.0, Compendium 8.0.0 and Compendium 8.1.0 under Release 8, and no Enterprise 8 document **(verified** — the Release 8 block of `docs.osgi.org/specification/` lists only Core and Compendium titles, read 2026-09-17). The enterprise content did not disappear — **Remote Services (chapter 100), Log, Http, Configuration Admin, Metatype and the rest appear as chapters of the Compendium in R8.1** **(verified** — the Compendium 8.1.0 HTML index opens its chapter list with "100 Remote Services", "103 Device Access", "104 Configuration Admin", "105 Metatype", read 2026-09-17).

**The honest formulation:** as of R8 the *Enterprise* document set as a separate publication ended at R7 (December 2018), and its services are carried by the Compendium. Anyone planning a migration should verify this against the Consolidated Release listing themselves rather than trusting a summary — including this one — because the naming is genuinely confusing and it is the kind of detail that a blog post is likely to get backwards **(verified** for the observation; **(flagged** for any deeper claim about the editorial rationale, which no primary source located this pass states).

---

## 9. The Release History

### 9.1 The Release Table

Every date below is taken from the **front matter of the specification document itself** — the page that reads, for example, "OSGi Core Release 8 / October 2020" — read on 2026-09-17. These are not recalled; they are quoted.

| Release | Documents published | Date on the document | Confidence |
| --- | --- | --- | --- |
| **Release 1** | "OSGi Service Gateway Release 1 Specification" | **Not established this pass** | **(flagged** — the document is listed at `docs.osgi.org/specification/` but its publication date was not read; do not quote a year for R1 from this guide) |
| **Release 3** | "OSGi Service Platform Release 3" — a single book containing the Reference Architecture, the Remote Management Reference Architecture, the Framework Specification and the service specifications | **March 2003** | **(verified** — `docs.osgi.org/download/r3/r3.book.pdf` front matter; the title page also carries ISBNs from two publishers, which dates the document as a print-era publication) |
| **Release 4, Version 4.0** | Core Specification + Compendium Specification (a Mobile Specification appears at 4.0.1) | **August 2005** | **(verified** — `docs.osgi.org/download/r4v40/r4.core.pdf` front matter) |
| **Release 4, Version 4.1** | Core + Compendium | **April 2007** | **(verified** — `docs.osgi.org/download/r4v41/r4.core.pdf` front matter) |
| **Release 4, Version 4.2** | Core + Compendium + **Enterprise** | **Core: June 2009. Enterprise: March 2010** | **(verified** — `r4v42/r4.core.pdf` and `r4v42/r4.enterprise.pdf` front matter) |
| **Release 4, Version 4.3** | Core + Compendium + Residential | **April 2011** | **(verified** — `r4v43/osgi.core-4.3.0.pdf` front matter) |
| **Release 5** | Core + Compendium + Enterprise | **March 2012** | **(verified** — `r5/osgi.core-5.0.0.pdf` and `r5/osgi.enterprise-5.0.0.pdf` front matter — both dated March 2012) |
| **Release 6** | Core + Compendium + Enterprise + Residential | **Core: June 2014. Enterprise: July 2015** | **(verified** — `r6/osgi.core-6.0.0.pdf` ("OSGi Core Release 6 / June 2014") and `r6/osgi.enterprise-6.0.0.pdf` ("OSGi Enterprise Release 6 / July 2015") front matter) |
| **Release 7** | Core + Compendium + Enterprise | **Compendium: April 2018. Enterprise: December 2018** | **(verified** — `r7/osgi.cmpn-7.0.0.pdf` ("OSGi Compendium Release 7 / April 2018") and `r7/osgi.enterprise-7.0.0.pdf` ("OSGi Enterprise Release 7 / December 2018") front matter) |
| **Release 8** | Core + Compendium **8.0** + Compendium **8.1**. No separate Enterprise document | **Core: October 2020. Compendium 8.1: December 2022** | **(verified** — `r8/osgi.core-8.0.0.pdf` ("OSGi Core Release 8 / October 2020"); `docs.osgi.org/specification/osgi.cmpn/8.1.0/` ("OSGi Compendium, OSGi Working Group, Release 8.1, Copyright © 2000, 2022 Eclipse Foundation, December 2022")) |

### 9.2 Reading the Table — Minor Revisions and Major Releases

Five things fall out of the table, and each of them corrects a mistake that is easy to make from memory:

1. **The release cadence was roughly one major release every three years — R4 (2005), R5 (2012), R6 (2014/15), R7 (2018), R8 (2020) — and R4 spanned 2005–2011 across four versions.** Release 4 was one release with four *minor* versions (4.0, 4.1, 4.2, 4.3). The Core specification makes this explicit in its own headings: the module and lifecycle chapters of the 4.1 document carry section titles "**Changes 4.1**" rather than being restructured from scratch **(verified** — Core R4.1 table of contents shows "3.17 Changes 4.1" and "4.9 Changes 4.1", `docs.osgi.org/download/r4v41/r4.core.pdf`, read 2026-09-17).
2. **R8.1 was a minor revision of the Compendium only.** There is no Core 8.1 and no Enterprise 8. Calling 8.1 "the R8 release" is imprecise; calling it "the last published OSGi specification release" is accurate **(verified** — Release 8 block of `docs.osgi.org/specification/`; read 2026-09-17).
3. **The Enterprise document arrived in 4.2, not in R4.0.** Its own title page is dated March 2010 while Core 4.2 is June 2009 — the Enterprise specification trailed the Core by nine months in that cycle **(verified** — front matter of both documents).
4. **Subsystems arrived in Enterprise R5 (March 2012) as chapter 134.** The word "Subsystem" does not occur in the Enterprise R4.2 document at all **(verified** — full-text counts of the two documents, 0 versus 1,093+ occurrences, read 2026-09-17).
5. **The publishing organisation changed mid-history and the licence changed with it.** The Alliance-era documents carry "OSGi Specification License" v1.0 (R4-era) or v2.0 (R6/R7/R8 /R7-era); the Eclipse-era Compendium 8.1 carries the **Eclipse Foundation Specification License v1.0** and is copyright "2000, 2022 Eclipse Foundation" **(verified** — licence pages inside `r4v43/osgi.core-4.3.0.pdf`, `r8/osgi.core-8.0.0.pdf` and `osgi.cmpn/8.1.0/LICENSE.html`; read 2026-09-17).

**What each release added — the verifiable anchors, not the marketing version:**

| Release | Verifiable headline additions |
| --- | --- |
| **R3 (2003)** | The framework specification in book form, the Service Gateway reference architecture, the remote-management reference architecture, and the service-library content of the era (HTTP, log, wire admin, XML parser, user admin and device access are all present) **(verified** — `r3/r3.book.pdf` contents, read 2026-09-17) |
| **R4 (2005); R4.1 (2007)** | The Core/Compendium document split; the **Module Layer** with its resolver — "Resolving Metadata", "Constraint Solving", "Resolving Process"; "Requiring Bundles"; "Fragment Bundles"; "Extension Bundles"; "Execution Environment"; the **Security Layer** with digitally signed JAR files; "What Is New" as its own introduction section. **R4.1 was a minor revision** — its own headings read "Changes 4.1" **(verified** — `r4v40/r4.core.pdf` and `r4v41/r4.core.pdf` contents) |
| **R4.2 (2009/2010)** | **Enterprise specification published** (March 2010); **Remote Services** appears (as chapter 13 in the Enterprise document and as chapter 6 in Core 4.2); the **framework launch API** `org.osgi.framework.launch`; **Service Hooks** (§12 in Core 4.2); the explicit treatment of **multiple-version exports** (§5.9) **(verified** — contents of `r4v42/r4.core.pdf` and `r4v42/r4.enterprise.pdf`) |
| **R4.3 (2011)** | **Generics migration** — the Core specification adds an introduction section "1.4 Migration to Generics"; **weaving hooks** appear (`org.osgi.framework.hooks.weaving`, "Weaving Hook Service Specification Version 1.0") **(verified** — contents and appendix of `r4v43/osgi.core-4.3.0.pdf`) |
| **R5 (2012)** | **Subsystems** (chapter 134 of Enterprise); **Declarative Services 1.2**; Core and Enterprise published in the same month for the first time in the Enterprise line's short history **(verified** — `r5/osgi.enterprise-5.0.0.pdf` contents) |
| **R6 (2014/15)** | **Declarative Services 1.3**; **Promises** (§705); a Residential specification; the Enterprise document published a year after the Core **(verified** — `r6/osgi.enterprise-6.0.0.pdf` contents) |
| **R7 (2018)** | **Declarative Services 1.4** with component-property-type annotations; **Converter** (§707); the Compendium published eight months before the Enterprise document **(verified** — `r7/osgi.cmpn-7.0.0.pdf` and `r7/osgi.enterprise-7.0.0.pdf`) |
| **R8 (2020/22)** | **Declarative Services 1.5**; the **Feature Service** as a new §159; enterprise chapters carried inside the Compendium; the Eclipse Foundation as the publisher; the **8.1 minor revision** in December 2022 **(verified** — `r8/osgi.core-8.0.0.pdf` and `osgi.cmpn/8.1.0/`) |

**Honest limit on this section:** the table states what is *verifiable from the documents' own tables of contents and front matter*. It does **not** claim to be a complete changelog — a full changelog would require reading every "Changes" section of every chapter across nine document sets, which was out of scope for this pass **(flagged** accordingly). Treat the table as an accurate skeleton with verified joints, not as an exhaustive feature history.

---

## 10. The Governance and Current State

### 10.1 The Headline Finding

**The OSGi Alliance no longer operates. The specification is now governed by the OSGi Working Group at the Eclipse Foundation, which hosts two projects — an OSGi Specification Project and an OSGi Technology Project. Both are in the Incubating state. The most recent published specification release is Compendium 8.1 from December 2022. Specification development is not stopped: draft specifications are still built and published, and in April 2026 the ecosystem received twelve months of dedicated modernisation funding from the Sovereign Tech Fund, delivered by Data In Motion.**

That is the finding. It is neither an obituary nor a victory lap, and both of those framings circulate widely. Section 10.4 explains why.

### 10.2 The Evidence Trail

| Claim | Evidence | Confidence |
| --- | --- | --- |
| The Alliance transferred its work to the Eclipse Foundation at the end of 2020 and shut down | "In October 2020, the OSGi Alliance announced the transition of the standardization effort to the Eclipse Foundation, subsequent to which it would shut down." | **(verified** — Wikipedia's OSGi article, `en.wikipedia.org/wiki/OSGi`, read 2026-09-17. Wikipedia is an aggregator, so this is corroborated independently by the next row. The Eclipse Foundation's own 2020 announcement page is no longer retrievable — `newsroom.eclipse.org/news/announcements/osgi-alliance-transfers-its-standardization-work-eclipse-foundation` returned an Eclipse **404** on 2026-09-17 — so the receiving foundation's live project pages, not the press release, are the primary corroboration for the transfer) |
| The Alliance is a **former** body, per the receiving foundation's own words | The Eclipse Specification Project page states it "is based on Specifications, Reference Implementations and Compliance Tests of the **former OSGi Alliance** and uses those materials as the baseline for creating new specifications, implementations and TCKs, or revising existing OSGi Alliance specifications" | **(verified** — `projects.eclipse.org/projects/technology.osgi`, read 2026-09-17. "Former" is the Eclipse Foundation's own word) |
| `osgi.org` now serves the Eclipse-hosted working group | The site's own title is "**OSGi Working Group | The Eclipse Foundation**", the breadcrumb reads "OSGi Working Group", and its pages link to Eclipse Foundation membership, mailing-list and privacy machinery. `osgi.org/news` returns a **404** with an Eclipse Foundation error page | **(verified** — `osgi.org` and `osgi.org/news`, read 2026-09-17) |
| The working group still exists and describes its own remit | "The OSGi Working Group drives the evolution and broad adoption of software technologies derived from or related to the OSGi Specification Project..." | **(verified** — `projects.eclipse.org/working-group/osgi` and `metrics.eclipse.org/wgs/osgi/`, read 2026-09-17) |
| Two projects, both Incubating | The OSGi Specification Project ("State: **Incubating**") and the OSGi Technology Project ("State: **Incubating**") | **(verified** — both project pages, read 2026-09-17) |
| Latest specification release: **Compendium 8.1**, December 2022 | Version header "Release 8.1", "December 2022", Eclipse Foundation copyright | **(verified** — `docs.osgi.org/specification/osgi.cmpn/8.1.0/`, read 2026-09-17) |
| Latest project release of any artefact: **2.0 Jakarta RS Whiteboard**, 2022-12-20 | The Specification Project's release table lists 2.0 Jakarta RS Whiteboard, 2.0 Servlet Whiteboard, 1.1 Data Service JDBC, 1.1 Push Stream and 1.3 Promises all dated **2022-12-20** as the newest entries | **(verified** — `projects.eclipse.org/projects/technology.osgi`, "Latest Releases — From 2022-12-20 to 2021-07-09", read 2026-09-17) |
| The Technology Project's newest artefact: **1.2.3 jakartarest-osgi**, 2024-09-24, with **1.3.0 osgi-test** at 2024-03-25 | Its release table | **(verified** — `projects.eclipse.org/projects/technology.osgi-technology`, read 2026-09-17) |
| Draft specifications are still being produced | `github.com/osgi/osgi` is "the main git repository for the OSGi specifications, implementations and TCKs" and its build publishes draft specifications to `osgi.github.io/osgi/core/` and `osgi.github.io/osgi/cmpn/`; `docs.osgi.org/specification/` links those as "**Development Draft**" | **(verified** — the repository README and the specification index, read 2026-09-17) |

### 10.3 What Is Actually Happening in 2026

This is the part that neither the "OSGi is dead" posts nor the "OSGi is thriving" posts tend to include, because both require reading a foundation's activity metrics.

**Activity over the twelve months to 2026-09-03, published by the Foundation itself** **(verified** — `metrics.eclipse.org/wgs/osgi/`, time ranges stated as 2025-10-05 to 2026-09-03 for issues and 2025-10-08 to 2026-09-03 for commits and reviews, read 2026-09-17):

| Metric | Value |
| --- | --- |
| Commits across the working group | **227 commits by 11 people on 11 repositories** |
| Issues | **31 issues submitted by 11 people on 5 repositories** |
| Reviews | **135 reviews by 8 people on 13 repositories** |
| Other repositories touched | 14 repositories total in scope |
| OSGi Specification Project | 56 commits, 44 reviews, 20 issues |
| OSGi Technology Project | 171 commits, 91 reviews, 11 issues |
| Most active non-spec repositories | `osgi-test` (53 commits), `feature-launcher` (48 commits), `jakartarest-osgi` (21 commits), `plurl` (14 commits) |

**The funded modernisation programme.** On **2026-04-20** the Eclipse Foundation's newsroom published an announcement titled "**Data In Motion to spearhead OSGi modernization**", stating:

> "Data In Motion has been commissioned by the **Sovereign Tech Fund** to modernize the OSGi ecosystem over the next 12 months." ... "OSGi got selected as one of the technologies to be improved." ... "**Data In Motion is a long-time contributor to the OSGi ecosystem, and is proud to have been commissioned to lead this effort to the next level. Together with the community (including the BNDtools project), Data In Motion will coordinate work across several targeted improvement areas over the coming year.**"

**(verified** — `newsroom.eclipse.org/rss/news/osgi/news.xml`, item dated Mon, 2026-04-20, author Ralph Göring, linking to `sovereign.tech/tech/osgi`; read 2026-09-17.)

Two things are worth extracting carefully from that announcement, because they are easy to over-read:

1. **The programme is real, dated, funded and scoped to twelve months from April 2026.** That is direct evidence against "abandoned".
2. **It is a modernisation programme, not a statement that the specification is being extended at the frontier.** The announcement's own justification describes OSGi as "a mature, battle-tested framework" and frames the work as improving an existing ecosystem — "managing dependencies, versioning, and the lifecycle of components becomes a tangled mess. OSGi brings order to that complexity." **(verified** — same source.) The word "mature" from a funder is not the same claim as "growing".

### 10.4 Verdict — Neither the Obituary nor the Victory Lap

| Proposition | Verdict | Basis |
| --- | --- | --- |
| **"OSGi is dead / was abandoned"** | **Not supported.** A working group exists with a public remit; two Foundation-hosted projects exist (albeit Incubating); draft specifications are still built; a named company is under contract for twelve months of funded modernisation from April 2026; the toolchain (bnd/bndtools) is releasing actively | Section 10.3 |
| **"OSGi is thriving / growing"** | **Not supported either.** No new published specification release since December 2022; both projects are in the Incubating state, which is the Foundation's *early* rather than mature designation; the whole working group shows 11 contributors and 227 commits in a year; the most recent Eclipse-hosted project release is from 2024; the community describes itself as a volunteer army | Section 10.3 |
| **"OSGi was replaced by JPMS"** | **Contradicted by the primary record.** The JPMS specification request says the opposite in terms: see section 14.3 | JSR 376 |
| **"OSGi is in maintenance"** | **Closest to accurate for the specification**, with the caveat that maintenance is not zero-activity: the activity table shows continuing commits and reviews, and the Compendium 8.1 minor revision (December 2022) was itself maintenance-plus-increment. Implementations are a separate question and are dated one by one in section 12 | Sections 10.3, 12 |

**The defensible one-paragraph statement, if one is needed for a decision document:** *OSGi is a mature, fully specified, still-implemented modularity standard whose specification release line paused after Compendium 8.1 in December 2022 and whose governance moved from the OSGi Alliance to an Incubating Eclipse Foundation working group in late 2020. It remains under active, funded maintenance and tooling development in 2026, it remains the substrate of several large commercial platforms (section 11), and it remains the only mainstream standard for runtime-dynamic modularity on the JVM. Anyone asserting that it is dead, or that it is growing, is asserting something the primary sources do not support.*

---

## 11. Platforms That Embed OSGi

### 11.1 The Adoption Table

Every row below was checked against **the vendor's own documentation** or **the project's own site or release feed** in this pass, unless the confidence column says otherwise. No row in this table comes from a listicle.

| Platform | Evidence that it embeds/runs on OSGi | Confidence |
| --- | --- | --- |
| **Eclipse IDE** | The Eclipse Foundation's own project page for Equinox states the project "is responsible for developing and delivering the OSGi framework implementation used for **all of Eclipse**" and that "Equinox, as the base for all of Eclipse, ships with all the major releases" | **(verified** — `projects.eclipse.org/projects/rt.equinox`, read 2026-09-17) |
| **Adobe Experience Manager (AEM)** | Adobe's own documentation: "The Web console in Adobe Experience Manager (AEM) is based on the **Apache Felix Web Management Console**. Apache Felix is a community effort to implement the OSGi R4 Service Platform, which includes the OSGi framework and standard services." The same page documents the console's **Bundles**, **Configuration** and **Components** tabs as "the mechanism for installing the OSGi bundles required for AEM", with install/update/refresh/start/stop/uninstall operations and start levels. A separate AEM page documents how to wrap a JAR as an OSGi bundle and names Apache Felix, Apache Sling, Apache Geronimo, Apache ServiceMix, Eclipse Bundle Recipes and the SpringSource Enterprise Bundle Repository as common sources of pre-wrapped bundles | **(verified** — `experienceleague.adobe.com`, AEM 6.5 Web Console, last updated 7 May 2026; and the AEM "OSGi Bundles" best-practices page surfaced at the same documentation set; read 2026-09-17) |
| **Liferay DXP** | Liferay's own developer documentation: "Liferay's runtime framework is lightweight, fast, and secure. **The framework uses the OSGi standard.** If you have experience using OSGi with other projects, you can apply your existing knowledge." The page documents service-registry publish/consume, dynamic dependency management "no restart required", module life cycle install/start/update/stop/uninstall, export-only visibility, and **semantic versioning with per-consumer version selection**. Its worked example uses `bnd.bnd` metadata, `@Component(service = Greeter.class)`, `@ProviderType`, and the Gogo shell | **(verified** — `learn.liferay.com`, "OSGi and Modularity", DXP developer documentation, read 2026-09-17) |
| **Apache Sling** | Sling is the web framework AEM is built on, and Sling's own site describes a framework whose "modular nature of Sling allows for specialized server instances that include only what is needed" with Sling **14 released 9 March 2026** | **(verified** for the project's existence, the modular framing and the 14/2026-03-09 release — `sling.apache.org`, read 2026-09-17. **(flagged** for the stronger claim "Sling is an OSGi-based framework": Sling's documentation set is OSGi-centric and AEM's own documentation refers to "Sling defaults" alongside Apache Felix, but this pass did not read a Sling page that states the OSGi dependency in one sentence, so the claim is recorded as *strongly supported but not quoted*) |
| **Apache Karaf** | Karaf describes itself as an "OSGi runtime" and "the modulith runtime", and its release announcements are titled "Karaf **OSGi runtime** 4.4.11" | **(verified** — `karaf.apache.org` and its news feed, read 2026-09-17) |
| **Talend Runtime** (the Talend/Cloud integration product) | Documented in this repository's own [talend_guide.md](talend_guide.md) as "the **OSGi/Apache-Camel-based container** that runs Routes, Data Services and Microservices" | **(verified** at the repository's own source, which cites the vendor patch note; **cross-referenced, not re-derived** — see §11.2. This guide did **not** independently fetch Qlik/Talend's page this pass, and says so) |
| **Sonatype Nexus Repository** | Described in this repository's own [sonatype_nexus_guide.md](sonatype_nexus_guide.md) as a "**Java (OSGi) server**" | **(flagged** — attributed to the repository's own guide. Sonatype's own documentation was **not** fetched this pass, so this guide records the claim as *inherited*, not as verified. Treat it as a lead to check, not a fact to quote) |
| **Eclipse Jetty** | The repository's own [jetty_guide.md](jetty_guide.md) carries an OSGi support row (✅ for Jetty, ❌ for Tomcat and Undertow) and an "**OSGi environment**" row selecting Jetty as the right choice for OSGi environments | **(flagged** — same handling: attributed to the repository's own guide. Jetty's own documentation was not fetched this pass) |
| **Embedded and IoT systems** | The OSGi Working Group's own announcement states: "OSGi is the foundation beneath some of the most widely used Java platforms in the world — including the Eclipse IDE, Adobe Experience Manager, **many enterprise application servers, and countless embedded and IoT systems**." | **(flagged** — this is the **working group's own promotional claim**. The Eclipse IDE and AEM parts of it are independently verified above; the "many enterprise application servers" and "countless embedded and IoT systems" parts are **not quantified and not sourced** anywhere this pass checked, and should be treated as advocacy rather than measurement) |

**A note on the shape of this table.** The *verified* rows cluster in one place — **enterprise content and digital-experience platforms** (AEM, Liferay, Sling, AEM's Felix console), the **Eclipse tooling platform**, and the **integration runtime** line (Talend, Karaf). That pattern is consistent with the specification's own history: the enterprise specification (2010 onward) is what pulled OSGi into content and integration platforms, and those platforms have long product lifecycles.

### 11.2 The Talend Runtime Boundary and Its Version-Alignment Constraint

**[talend_guide.md](talend_guide.md) already documents Talend Runtime. This guide does not re-explain Talend; it cites the one constraint that matters to an OSGi practitioner, and it does so from that guide rather than re-deriving it.**

The constraint, as documented there: since **Studio R2025-02**, Talend artefacts are built with **Java 17** and **Camel 4**, and Talend Runtime must be at **`8.0.1-R2025-02-RT` or later** to run them; Talend states that unaligned Studio and Runtime versions are at risk. The source is the **Studio cumulative patch note R2026-08, patch `Patch_20260821_R2026-08_v1-8.0.1`, release date 2026-08-21** **(verified** — cited from [talend_guide.md](talend_guide.md), which verified it against the vendor patch note; this guide did not re-fetch that note, and the reader wanting the primary text should follow the reference in the Talend guide).

**Why it belongs in an OSGi guide.** This is the clearest vendor-documented instance of the general rule this guide develops in section 15.4: **OSGi lets you decouple modules at runtime, and that decoupling does not survive an unaligned *toolchain*.** Talend's constraint is not a bundle-resolution constraint at all — it is a *build-generation* constraint: the Studio generates artefacts against one Java and one Camel generation, and the runtime container must be of a matching generation to run them. A team can satisfy every `Import-Package` range in a manifest and still be broken, because the mismatch lives one level up, in which generator produced the module and which runtime is expected to host it.

**The transferable generalisation, stated for an OSGi estate generally:**

| Alignment axis | What must match | Failure signature if it does not |
| --- | --- | --- |
| **Java level** | The build JDK/JVM target of every bundle versus every runtime's execution environment | Bundles install and then fail on missing JDK classes, or fail to resolve against the framework's execution environment |
| **Core/Compendium level** | The specification level the bundles were built against versus the level the container ships | Framework API `NoSuchMethodError`-shaped failures at start, or services that are simply absent |
| **Service-implementation level** | Which DS/SCR/HTTP-whiteboard implementation and version the container provides | Components that never activate, or activate but never bind |
| **Generator/toolchain level** (Talend's case) | The producing tool's version versus the hosting runtime's version | The vendor's own "at risk" designation — arbitrary, hard-to-diagnose runtime behaviour |

**(flagged ⚠-structural** for the generalisation table — the four axes are this guide's synthesis from the single vendor-documented instance plus the specification's structure; only the Talend row is vendor-documented, and it is documented in [talend_guide.md](talend_guide.md).)

### 11.3 The Nexus and Jetty Boundaries

Two guides in this repository record OSGi facts about products that this guide deliberately does not re-explain:

- **[sonatype_nexus_guide.md](sonatype_nexus_guide.md)** describes Nexus Repository as a "**Java (OSGi) server**". **Boundary:** this guide does not describe Nexus's repository features, its provisioning model, or its operational behaviour. What an OSGi practitioner needs from that claim is only the *architectural consequence* — a repository manager that is itself an OSGi server is a repository manager whose artefact-addressing model is compatible with bundle provisioning, and section 3.4 establishes that modern OSGi addresses artefacts by Maven coordinates. The claim itself is **inherited and unverified this pass** (see the table above).
- **[jetty_guide.md](jetty_guide.md)** carries an OSGi support row and an "OSGi environment" selection row. **Boundary:** this guide does not compare servlet containers. What an OSGi practitioner needs is the *layer mapping* — the OSGi HTTP Service and the HTTP Whiteboard are the specified abstractions a servlet container fills, and the Compendium carries both the classic Http Service chapter (§102) and, in R8, the Whiteboard model **(verified** — http service chapter present in the Enterprise R5/R6/R7 and Compendium R7 tables of contents; a "1.1 Http Whiteboard" release dated 2021-10-13 exists in the Specification Project's release table; read 2026-09-17). Which container you point it at is a product decision, and that decision belongs in the Jetty guide's frame, not here.

**The boundary rule this whole guide follows:** a product that *embeds* OSGi is named here only to establish that OSGi is present and which OSGi-level version it runs, with a dated source. The product's own capabilities, roadmap, pricing and operational behaviour belong to that product's guide.

### 11.4 What This Guide Refuses to Claim

Three refusals, stated so that a reader can hold this guide to them:

1. **No banking-sector adoption claim, and no named institution.** This pass found no vendor documentation stating that a named bank or a named banking product embeds OSGi. Sector claims about OSGi adoption are common in conference material and absent from primary sources. **This guide therefore asserts none, names none, and records the absence as the finding.** The worked example in section 17 is an explicitly illustrative scenario in the house's Cymbal Bank persona; it is not a description of any institution's system, and no real institution is used as an adoption example anywhere in this file.
2. **No adoption statistics.** No source located this pass quantifies OSGi adoption — not market share, not bundle counts, not deployment counts. The working group's "countless embedded systems" is rhetoric; the Foundation's own metrics are *contributor* metrics, not *user* metrics. Nothing in this guide converts either into a number **(verified** as an absence — the metrics page reports commits, issues, reviews and repositories only; read 2026-09-17).
3. **No claim that any specific platform's use of OSGi is a recommendation.** AEM and Liferay embedding OSGi says something about OSGi's durability and about the cost of getting modularity wrong at scale. It does not follow that a new project should adopt OSGi; section 16 is the honest treatment of that question.

---

## 12. The Runtime Containers

### 12.1 Framework Implementations versus Containers

Two different things are called "an OSGi runtime" in everyday speech, and the distinction drives purchasing and upgrade decisions:

- A **framework implementation** implements the Core specification's layers and is what the compliance programme tests. Felix and Equinox are the two that matter.
- A **container** packages a framework with operational furniture. Karaf is the container that survived; it can run either framework underneath.

**(verified** — Felix describes its flagship as "an implementation of the OSGi R8 core framework" in its subprojects list; Equinox's project page calls it "an implementation of the OSGi core framework specification, a set of bundles that implement various optional OSGi services and other infrastructure for running OSGi-based systems"; Karaf's 4.4.1 release notes record upgrades to both Felix and Equinox, which documents that Karaf carries a framework choice; read 2026-09-17.)

### 12.2 The Container Table

Dated, from each project's own release channel or catalogue. **Where a project's own documentation disagrees with itself, both positions are recorded rather than one being chosen.**

| Project | What it is | Latest release and date | Maintenance status | Notes and discrepancies |
| --- | --- | --- | --- | --- |
| **Apache Felix — Framework** | The Apache framework implementation | **7.0.5, released 3 June 2022** (R7 line); **2.8.4, released 2 May 2023** (the older-branch release) | **Slowing.** The most recent release *in the modern line* is from mid-2022; the only newer artefact is the legacy-branch 2.8.4 | **Its documentation contradicts itself.** `felix.apache.org`'s subprojects page calls the Framework "an implementation of the OSGi **R8** core framework"; the project's GitHub repository description says the framework "implements the OSGi Core **R7** specification"; the last release is 7.0.5. **This guide does not resolve the discrepancy — it records it** **(verified** — both texts and the release catalogue `tooling-bat.apache.org/felix-framework/`, "Last updated 15 Sep 2026"; read 2026-09-17) |
| **Apache Felix — component/service subprojects** | Config Admin, SCR (Declarative Services), HTTP Service, Gogo shell, Web Console, Event Admin, File Install, Health Checks, Inventory Printer, Log, Logback, Metatype, Maven Bundle Plugin, OBR, Dependency Manager, Atomos, Framework Security | Per-subproject; the project publishes a downloads page listing current versions for each. `maven-bundle-plugin` artefacts carry February 2026 timestamps on the ASF downloads mirror; Config Admin is at 1.9.26, Converter 1.0.12, Coordinator 1.0.2, Dependency Manager 4 at `r16` | **Active as a portfolio**, with individual variation by subproject | **Caveat:** the *project* is alive even though the *framework's* last R7-line release is from 2022. Judging "is Felix maintained?" from the framework version alone gives the wrong answer **(verified** — `felix.apache.org/documentation/downloads.html` lists current versions for every subproject; `downloads.apache.org/felix/` shows `maven-bundle-plugin-6.2` artefacts dated 2026-02-27; read 2026-09-17) |
| **Eclipse Equinox** | The Eclipse framework implementation; the base of the Eclipse IDE | **4.41, built 28 August 2026**; **4.40** (1 June 2026); **4.39** (26 February 2026); nightly integrations as recent as **16 September 2026** | **Actively released**, on the Eclipse Platform's release train (roughly quarterly named releases) | **The project-management page's release list is stale and must not be used as evidence.** `projects.eclipse.org/projects/rt.equinox` lists releases only up to **4.8.0 (Photon), 2018-06-27**, and its "Contribution Activity" chart does show commits across 2025–2026. The authoritative release evidence is `download.eclipse.org/equinox/`, which shows 4.41 as the **Latest Release**. Equinox's *state* is **Mature** on the project page **(verified** — `download.eclipse.org/equinox/` and `projects.eclipse.org/projects/rt.equinox`, read 2026-09-17) |
| **Apache Karaf** | The container: shell, SSH, deployers, configuration, provisioning, features, instances, JMX, plus Karaf Cellar (clustering) and Karaf Decanter (monitoring) | **4.4.11, released 27 April 2026** (announced 30 April 2026); 4.4.10 (13 February 2026); 4.4.9 (15 December 2025); 4.4.8 (7 August 2025); Decanter **2.12.0** (14 November 2025); Cellar **4.4.8** (12 September 2025) | **Actively maintained.** The 4.4.x line is in maintenance cadence with dependency and JDK updates; a move to **4.5.0** was under active discussion from May 2026 | The announcements describe the 4.4.x line as bringing "**OSGi R8 support**, Pax Web 8.0.x, Pax Logging 2.3.x"; the 4.4.11 release notes record `org.osgi:osgi.core` bumped **7.0.0 → 8.0.0**. JDK support has tracked forward (javase 11 set as the Maven plugins' default; JDK 26 support noted in 4.4.8) **(verified** — `karaf.apache.org/news`, `github.com/apache/karaf/releases`, and the Karaf board-report page which records the 4.5.0 discussion thread starting May 2026; read 2026-09-17) |
| **Apache Aries** | The enterprise-OSGi component set — Blueprint, JPA, JNDI, Transaction, Subsystem, JMX, typed events, test support and more | Subprojects published to Maven Central; the current-releases page documents the module list rather than a single version | **"Largely maintenance mode with some developments ongoing."** | The status sentence is the project's **own board report wording**, dated 2025-07: "Description: Apache Aries software is a set of pluggable Java components enabling an enterprise OSGi application programming model. Project Status: **Largely maintenance mode with some developments ongoing.** Issues for the board: No issues at the moment." **(verified** — Apache Aries dev mailing list, board report draft posted July 2025, read 2026-09-17) |

### 12.3 The Dormant Tail — Retired and Attic Projects

This ecosystem has an unusually long tail of once-canonical projects that are now retired. Knowing which they are prevents a design from being built on something that stopped shipping years ago.

**Apache Felix's own "Retired projects" list** — described on the project's site as "no longer maintained and are not documented in this site" — includes projects that were, at various points, the recommended answer to a real problem **(verified** — `felix.apache.org/documentation/subprojects.html`, "Retired projects" section, read 2026-09-17):

The list includes **iPOJO** (a component model, for years the most-discussed alternative to DS), **Deployment Admin** (a standardised deployment-package mechanism), the **Maven SCR Plugin** (the generator that produced DS descriptors from annotations *before bnd took over that job* — its retirement is section 13's tooling story in one line), and a long tail: **Maven OBR Plugin, OSGi Bundle Repository, Preferences Service, User Admin, JAAS Support, Serialization Framework, Manifest Generator (mangen), Remote Shell, Shell TUI, Script Console Plugin, Commons, Auto Configuration, MOSGi, OSGi Core, UPnP, Lightweight HTTP Service**. The pattern is that functionality moved **into the specification** (making a Felix-specific implementation redundant) or **into bnd and the toolchain**.

**Apache Karaf's Cave** — a Karaf subproject that was moved to the Apache Attic. The project's own announcement gives the reason and it is worth quoting as the honest face of ecosystem dormancy: "**As there is no more interest of users and no active contributors, the Karaf team decided to move the Karaf Cave subproject to attic.**" The date is **15 April 2023** **(verified** — `karaf.apache.org/news` archive entry, read 2026-09-17).

**The lesson for anyone evaluating an OSGi-adjacent dependency in 2026:** check three things, in this order — (1) the last release **date**, not the version number; (2) whether the project appears on a project's own **retired/attic** list; and (3) whether the functionality moved into the **specification or into bnd**, in which case the retirement is a sign of health rather than decay. The Maven SCR Plugin is the clean example: it was retired *because* its job was absorbed.

---

## 13. The Tooling Chain

### 13.1 bnd — the De Facto Build Engine

**bnd** is the tool that turns a Java project into a correctly-manifested bundle — it computes imports and exports from bytecode rather than asking the developer to hand-write them — and **bndtools** is its Eclipse IDE integration. In 2026 it is the load-bearing tool of the OSGi toolchain.

| Fact | Value | Confidence |
| --- | --- | --- |
| Latest release | **Bnd/Bndtools 7.4.0, released 18 August 2026** — the current release at the top of the project's release list | **(verified** — `github.com/bndtools/bnd/releases`; the release is marked "Latest" and dated 18 Aug; read 2026-09-17) |
| Relationship to the specification | It is the tool the working group points at — the 2026 funded-modernisation announcement names **bndtools** as a participant — and it is the tool Liferay's and the specification's own tutorials use (`bnd.bnd`; enRoute "uses Maven and bnd") | **(verified** — Eclipse newsroom 2026-04-20 item; `learn.liferay.com`; `enroute.osgi.org`; read 2026-09-17) |
| Direction of travel | **Convergence with Eclipse PDE.** The steering-committee chair's November 2023 announcement describes sharing the bnd Jar Viewer, sharing the bnd file editors used in PDE's build descriptor, integrating the resolver, and the possibility of a long-run merge of bndtools and PDE. The working group is recorded as "fully in support" | **(verified** — Eclipse newsroom item dated 2023-11-01; read 2026-09-17) |

**Why bnd's role is architecturally significant, not merely practical:** the specification *requires* accurate metadata, and hand-maintained metadata is wrong metadata. A tool that derives imports and exports from compiled bytecode is the difference between a module layer that works and one that fails intermittently in production. If a project adopts OSGi without adopting a manifest-computing build tool, it has adopted the resolver and rejected the precondition for satisfying it.

### 13.2 Build Plugins and IDEs

| Tool | Role | Dated status |
| --- | --- | --- |
| **Apache Felix Maven Bundle Plugin** | The long-established Maven plugin that wraps bnd for Maven builds; Felix's own description: "A Maven plugin to simplify building bundles" | **Current** — `maven-bundle-plugin` artefacts on the ASF downloads mirror carry **2026-02-27** timestamps; the downloads page lists the 6.x line **(verified** — `felix.apache.org/documentation/subprojects.html`; `downloads.apache.org/felix/`; read 2026-09-17) |
| **Maven SCR Plugin** | The older annotation-to-DS-descriptor generator | **Retired** — on Felix's retired-projects list **(verified** — same page, read 2026-09-17) |
| **Gradle** | bnd publishes Gradle plugin usage (the Karaf and Liferay examples use Gradle; bnd's own CI includes Gradle actions and the `gradle-plugins` module) | **Current** — Gradle plugin modules are part of the bnd repo and appear in the 7.x release notes **(verified** — `github.com/bndtools/bnd` release notes and repository layout, read 2026-09-17) |
| **Karaf's `karaf-maven-plugin`** | Builds and verifies Karaf features and assemblies | **Current** — named as receiving improvements in the 4.4.9, 4.4.10 and 4.4.11 releases **(verified** — `karaf.apache.org/news`, read 2026-09-17) |

### 13.3 Testing

Testing OSGi code has a reputation for being awkward, and the reason is structural: a bundle's behaviour depends on its **wiring**, so a unit test that runs on a flat classpath tests something other than what will run in production. The ecosystem's answer is to test *inside a framework*.

| Facility | What it is | Dated status |
| --- | --- | --- |
| **`osgi-test`** | The OSGi Working Group's testing support — "testing support (JUnit 4, JUnit 5, AssertJ) for testing in OSGi runtimes" | **Active.** The OSGi Technology Project's newest releases include **1.3.0 osgi-test (2024-03-25)**, 1.2.1 (2022-07-14), 1.2.0 (2022-07-12), 1.1.0 (2022-03-14), 1.0.0 (2021-06-23); the repository was the **second most active** in the working group over the last twelve months with **53 commits** **(verified** — `projects.eclipse.org/projects/technology.osgi-technology` and `metrics.eclipse.org/wgs/osgi/`, read 2026-09-17) |
| **`feature-launcher`** | Launches a set of Features (the R8 Feature model) for testing and execution | **Active** — **48 commits and 23 reviews** in the twelve-month window, the third most active repository **(verified** — `metrics.eclipse.org/wgs/osgi/`, read 2026-09-17) |
| **Aries test support** | `org.apache.aries.testsupport` modules, including the Aries test-harness lineage | **Present** — published to Maven Central and listed in Aries' current-releases module list; the project as a whole is in "largely maintenance mode" **(verified** — `aries.apache.org/documentation/downloads/currentreleases.html`; Aries board report 2025-07; read 2026-09-17) |
| **Felix Health Checks** | Runtime health verification rather than test harness, but the operational counterpart | **Active** — on Felix's active-subprojects list: "An extensible framework to monitor the status of the OSGi container at runtime" **(verified** — `felix.apache.org/documentation/subprojects.html`, read 2026-09-17) |

**The testing discipline, stated as guidance:** test at three levels — (1) plain unit tests for logic that does not touch the registry; (2) **in-framework integration tests** for anything that does, using `osgi-test` or an equivalent harness, so that the resolver and the wiring are exercised; and (3) **an explicit assertion that the wiring is what you intended** — because "it started" and "it resolved to the provider I designed for" are different claims, and only the second one predicts production.

**(flagged ⚠-structural** for the three-level testing discipline — it is standard practice in this ecosystem and follows directly from the specification's own model, but it is not attributable to a single primary source; the *tools* named are each dated above.)

---

## 14. OSGi versus JPMS

### 14.1 The JSR History, From the Records

The relationship between OSGi and the Java Platform Module System (JPMS, delivered in Java 9) is fully documented in the JCP records, and reading them settles most of the argument **(all four records read 2026-09-17**):

| JSR | Title | Status and key dates |
| --- | --- | --- |
| **JSR 277** | Java Module System | **Withdrawn** — "Withdrawn at the request of the Specification Lead. **Superseded by JSR 376**", withdrawal dated **20 January 2016**. JSR review ballot June 2005; early draft review October–November 2006. Specification lead: Alex Buckley (Oracle) |
| **JSR 291** | Dynamic Component Support for Java SE | **Final** — Final Release **7 August 2007**. Specification lead: Thomas Watson (IBM). The request states it "will be based upon JSR 232 which references the recently published **OSGi Service Platform Release 4 Core Specification**" and will be "a **subset of JSR 232** including the modularity and lifecycle aspects of the OSGi R4 framework but excluding the service aspects of the framework, declarative service support, and the services defined by JSR 232". Proposed package name: **`org.osgi.framework`** |
| **JSR 294** | Improved Modularity Support in the Java Programming Language | **Withdrawn** — same reason and date as JSR 277 ("Superseded by JSR 376"), **20 January 2016**. Its history runs from "**superpackages**" (2006) to a lighter "**modules**" concept (2008) |
| **JSR 376** | Java Platform Module System | **Final** — Final Release **21 September 2017**. Specification lead: Mark Reinhold (Oracle). Expert group included the Eclipse Foundation (Wayne Beaton), Gradleware, IBM, Red Hat, Oracle and **Paremus (Neil Bartlett)** — i.e. OSGi expertise was in the room |

**Three facts from these records that are worth more than any blog post on the topic:**

1. **The Java Platform's own module-system effort was explicitly benchmarked against OSGi, and OSGi was formally inside the JCP.** JSR 291 made the OSGi R4 Core framework a JCP standard for Java SE in 2007, and the JSR 376 request discusses JSR 291 by name **(verified** — JSR 291 and JSR 376 request texts, `jcp.org`).
2. **JSR 277 — the module system that was supposed to ship in Java 7 — died.** It was withdrawn in 2016 and superseded. The Java module system that eventually shipped was specified from scratch under JSR 376 **(verified** — JSR 277 detail page, `jcp.org`).
3. **JSR 277's criticism of OSGi was aimed at R3, and was stale by the time it was written.** The request argues that OSGi "must choose one bundle that will be the provider of the exported package for all bundles which have dependencies on that package, so it is impossible to support more than one version of shared package at runtime" — but Core R4.2 (June 2009) already carried "**5.9 Multiple Version Export Considerations**", and R4 had shipped in August 2005 **(verified** — JSR 277 §2.6 and Core R4.2 table of contents; this is the same point made in section 3.3).

### 14.2 The Comparison Table

| Concern | OSGi | JPMS (Java 9+) |
| --- | --- | --- |
| **Purpose** | A **runtime** module and component system: resolution, lifecycle, service registry | A **compile-time and start-up** module system: reliable configuration and strong encapsulation |
| **Reliable configuration** | Yes — imports declare version ranges and the resolver enforces them **(verified** — Core module-layer "Constraint Solving" and "Resolving Process") | Yes — that was the explicitly named goal: "The proposed specification will address two fundamental needs of large Java applications: **Reliable configuration and strong encapsulation**" **(verified** — JSR 376 §2.4) |
| **Strong encapsulation** | Class-loading isolation between bundles; packages hidden unless exported. **Not** language-level or JVM-level enforcement | **Yes**, at the language and JVM level: "the access-control mechanism of the Java language and the JVM prevents code from accessing packages that are not exported by their modules" **(verified** — JSR 376 §2.6) |
| **Which one OSGi lacks, in JSR 376's own words** | — | "OSGi addresses the problem of reliable configuration but, **since it builds on top of the Java SE Platform, it does not provide strong encapsulation**" **(verified** — JSR 376 §2.5) |
| **Dynamic install/update/uninstall at runtime** | Yes — the lifecycle layer's five verbs | **No.** JPMS has no dynamic module graph manipulation of this kind |
| **Service registry with lifecycle awareness** | Yes — publish/find/bind with events, ranking, filters, trackers | **No** — `uses`/`provides` plus `ServiceLoader` is a static provider-discovery mechanism, not a registry with lifecycle events |
| **Multiple versions of the same module/package coexisting** | Yes, per-consumer wiring **(verified** — Liferay's documentation of per-consumer version selection; Core R4.2 §5.9) | **No.** Versions are not part of module identity on the module path in the way OSGi's are; one module of a given name, one resolution. (JPMS is designed for the *application assembly* problem, not the *coexistence* problem) |
| **Fragments (attaching content to another module)** | Yes — fragment bundles, a specified mechanism | No equivalent |
| **Start levels / ordered startup** | Yes — a specified Start Level Service | No equivalent |
| **Third-party modularity tooling** | bnd/bndtools, PDE, Karaf, Maven/Gradle plugins | `javac`/`java`, `jar --describe-module`, `jdeps`, `jlink`, build-tool support |
| **Modern JDK interaction** | Runs on modern JDKs; **Atomos** exists specifically to load bundles "in a variety of environments, such as, **jlink image, native-image, Android** and flat class path" | — **(verified** — Felix active subprojects list, read 2026-09-17) |

**Two structural observations from the table.** First, **the two systems are aimed at different problems**: JPMS solves "make the JDK and large applications reliably configurable and hermetically encapsulated at compile and start-up time"; OSGi solves "let a long-lived system be composed of independently versioned parts that can change while it runs". Second, **OSGi's noted gap — no strong encapsulation at the language level — is a consequence of its design constraint**: OSGi must run on stock JVMs, so it cannot change the access-control rules. JPMS *is* the change to the access-control rules, which is exactly why it required the JDK's modularisation to ship.

### 14.3 The Conclusion This Guide Stands Behind

The claim "**JPMS replaced OSGi**" is not merely unsupported — **the primary record contradicts it in terms**. From the JSR 376 request text, under a heading that asks why existing specifications do not meet the need:

> "OSGi's life-cycle and dynamic service-registry facilities are useful to some kinds of sophisticated applications but are **beyond the scope of the needs outlined above**. **Those who require these facilities will still be able to run OSGi on top of Java SE 9 implementations.**" **(verified** — JSR 376 §2.5, `jcp.org/en/jsr/detail?id=376`, read 2026-09-17)

The system that allegedly replaced OSGi explicitly disclaims the part of OSGi people use it for, and states that OSGi will continue to run on the platform it delivers. **That sentence settles the question in the only way a primary source can.**

The honest, balanced conclusion:

| Statement | Verdict |
| --- | --- |
| JPMS and OSGi overlap on **reliable configuration** and **module boundaries** | **True.** Both solve the classpath problem, differently |
| JPMS **superseded OSGi for the JDK and the Java Platform itself** | **True** — the JDK is modular, and that is JPMS |
| JPMS **replaced OSGi's dynamic lifecycle and service-registry role** | **False** — disclaimed by JSR 376 §2.5 |
| A project choosing JPMS **obtains OSGi's capabilities for free** | **False.** A JPMS module graph is resolved at start-up; there is no install/update/uninstall of a running system, no registry with events, no fragments, no start levels |
| A project choosing OSGi **must give up JPMS or modern JDKs** | **False.** OSGi runs on modern JDKs, and the ecosystem has worked the interop problem explicitly — Felix's **Atomos** exists to load bundles in a jlink image, a GraalVM native image, Android, and on a flat classpath **(verified** — Felix subprojects list, read 2026-09-17) |
| The practical default for a **new application in 2026** is JPMS or the classpath, and OSGi when you need runtime modularity | **Supported**, and it is the guidance this guide gives in section 16 |

**A closing note on the discourse itself.** The "OSGi is dead / was replaced by JPMS" genre is dense with posts that assert a causal replacement while citing neither JSR 376's §2.5 nor any OSGi project's release feed. This guide notes the pattern rather than participating in it: it states what the specification says, what each project ships and when, and declines to convert an absence of enthusiasm into a fact about a technology **(flagged ⚠-structural** — this is a characterisation of a body of informal writing, not a claim about any specific article; no individual post is cited, because citing one would dignify it as evidence).

---

## 15. Operations

### 15.1 Security

OSGi's security model is specified, real, and **increasingly in tension with the JDK it runs on**. Both halves of that sentence matter.

| Element | What it is | Status |
| --- | --- | --- |
| **Security layer, and signed bundles** | A specified layer covering "Digitally Signed JAR Files" and "Permissions": code-sign the JAR, and the framework verifies it and can condition permissions on the signer | **Specified** — a chapter of every Core release from R4 onward, and widely used for provenance **(verified** — Core R8 and R4.0 chapter 2 "Security Layer"; "Digitally Signed JAR Files" also appears as module- and lifecycle-layer sections) |
| **Permission model** | Grant permissions to bundles by location or signer; enforce at the framework boundary | **Specified** — `AdminPermission`, `ServicePermission`, `PackagePermission`-style enforcement appear throughout the API **(verified** — the Core R4.3 API appendix documents hooks requiring `AdminPermission[bundle,WEAVE]` for weaving, and the Core R2-era `WirePermission` appears in the R3 index; read 2026-09-17) |
| **The dependency that is going away** | OSGi's permission enforcement relies on the JDK **Security Manager** | **Deprecated for removal.** JEP 411, "Deprecate the Security Manager for Removal", delivered in **Java 17** (status "Closed / Delivered"), states in its summary: "Deprecate the Security Manager for removal in a future release... it is costly to maintain." JEP 486, "**Permanently Disable the Security Manager**", is the follow-up **(verified** — `openjdk.org/jeps/411`, read 2026-09-17) |

**What this means in practice, stated carefully:**

- **The structural risk is real and it is documented at the source:** a specification layer whose enforcement mechanism depends on a JDK facility that is deprecated and slated for permanent disablement will not be able to enforce the way it used to. JEP 411 also notes that "**Jakarta EE has several requirements on the Security Manager**", which shows the JDK's own authors knew platform specifications had been built on it **(verified** — JEP 411, Risks and Assumptions section, read 2026-09-17).
- **What this guide does not claim:** it does **not** claim that OSGi's security layer is removed, that any given framework has dropped it, or that any given container's security model is broken. No primary source located this pass states any of those things. What is verified is the JDK-side deprecation and the dependency relationship.
- **The defensible operational position:** **do not design new controls that depend on OSGi's permission model.** Use signed bundles for provenance and integrity — signing, verification and content trust remain valuable and are not Security-Manager-dependent — and put authorisation in application code, in the service layer (who may call what), and at the platform boundary (container, network, identity). Treat the security layer's *permission enforcement* as legacy **(flagged ⚠-structural** — this is this guide's synthesis of a verified JDK deprecation with the specification's structure; it is standard practice advice rather than a quoted requirement).

### 15.2 Observability

OSGi ships observability primitives in the specification, which is unusual and worth exploiting rather than re-inventing:

| Facility | What it gives you | Dated status |
| --- | --- | --- |
| **Log Service** | Per-bundle logger configuration, a log reader, and — from R7 — a **Log Stream Provider** for streaming log records out of the framework | **Specified** — Compendium/Enterprise §101 **(verified** — present in Enterprise R5/R6/R7 and Compendium R7 tables of contents; "1.0 Log Stream Provider Service" released 2021-10-13 in the Specification Project's release table; read 2026-09-17) |
| **Web Console** | An HTTP console listing bundles, services, components and configuration | **Active** — Felix's Web Console is on its active-subprojects list, and **Adobe AEM's own console is based on it** **(verified** — Felix subprojects page; Adobe AEM documentation; read 2026-09-17) |
| **Shell (Gogo)** | Interactive inspection of the framework | **Active** — Felix "Gogo — an advanced shell for interacting with OSGi frameworks"; Liferay's documentation uses Gogo in its worked example **(verified** — Felix subprojects page; `learn.liferay.com`; read 2026-09-17) |
| **Health Checks** | Runtime health status of the container | **Active** — Felix subprojects page: "An extensible framework to monitor the status of the OSGi container at runtime" **(verified** — same source) |
| **Monitoring and alerting** | Off-the-shelf container monitoring | **Active** — Karaf **Decanter 2.12.0** (14 November 2025) provides collectors, dispatchers, alerting and appenders, including a Prometheus collector **(verified** — `karaf.apache.org` and its news feed, read 2026-09-17) |

**The observability design rule that follows:** because the module layer's health is a *graph* rather than a *flag*, the essential signals are **per-bundle resolution state** and **which provider each requirement wired to**. A dashboards set that reports only "is the process up" and "are requests failing" will miss the class of incident where a bundle silently fails to resolve, or resolves to a provider that is not the one the design intended. This is the operational face of section 4.3.

### 15.3 The Failure Modes and Their Signatures

This is the section to keep. Every entry below is either a named mechanism in the specification or a documented behaviour of a shipped product, and each has a *signature* — what you actually see, which is almost never what you would guess.

| Failure mode | What it actually is | Signature |
| --- | --- | --- |
| **Unresolved requirement** | A bundle's declared requirement has no matching capability in the repository | The bundle sits in an installed-but-not-resolved state; **it never runs**, and nothing in the application's own logs explains why. The framework's resolution diagnostics do |
| **`uses` constraint conflict** | Two bundles wired to different providers of the same package, where a package is used across the boundary | A resolution failure whose message names a *package* and a *constraint*, not your code. Frequently mistaken for a version bug in a bundle you control |
| **Duplicate class visibility** | The same class reachable through two wirings (classically via `Require-Bundle`, fragments, or a bundle bundling a library that is also provided) | `ClassCastException` where the types look identical; `instanceof` failing on the same nominal class. The classic "same name, different classloader" symptom |
| **Stale service reference** | A consumer holds a service object whose provider was unregistered or updated | Intermittent failures that correlate with a deployment, not with load |
| **Old and new running simultaneously after an update** | Frameworks keep the old provider alive while existing references exist | Adobe's own documentation states it: "after an update both the old and new version may still be running due to prior references", and recommends **Refresh Packages** after an update **(verified** — `experienceleague.adobe.com`, AEM 6.5 Web Console, read 2026-09-17). The signature is a fix that appears not to have applied |
| **Listener/registration leak** | A bundle is stopped or updated without removing listeners or unregistering services | Slow memory growth over successive deployments; an increasing count of registered listeners or services |
| **Environment mismatch (execution environment)** | A bundle is installed on a JVM level its execution environment does not permit, or the reverse | Failure at start, near the framework's own loading path, before application code runs |
| **Toolchain/generator misalignment** | The module was produced by a generator of one generation and hosted by a runtime of another | The Talend case in §11.2: the vendor designates unaligned Studio/Runtime versions "at risk" **(verified** via [talend_guide.md](talend_guide.md)) |

**(verified** for the mechanisms named — unresolved requirements, `uses` constraints, multiple-version export considerations, stale references, fragments, extension bundles and start-level behaviour are all specification concepts named in the tables of contents cited throughout this guide, and the "old and new both running" behaviour is quoted from Adobe's documentation. **(flagged ⚠-structural** for the pairing of each mechanism with its diagnostic signature — the signatures reflect standard practice and the documented behaviours, and are presented as guidance rather than as normative text.)

### 15.4 Upgrade and Release Discipline

The specification gives you the tools for disciplined change; it does not enforce the discipline. The rules below follow from everything above and are the operational summary of this guide.

| Rule | Why |
| --- | --- |
| **Version everything semantically, and mean it.** A change to an exported package is a version change; a change to an exported API is a *major* version change | The resolver enforces your promises. If your versions lie, your resolution is wrong **(verified** as the mechanism — import ranges are enforced; Liferay's documentation links semantic versioning as a platform requirement) |
| **Align the four axes of §11.2 before you deploy** — Java level, Core/Compendium level, service-implementation level, generator/toolchain level | Talend's documented Studio/Runtime constraint is the vendor-documented instance; the same class of mismatch exists in every estate **(verified** for the Talend instance; flagged for the generalisation) |
| **Re-resolve and inspect after every deployment, and refresh explicitly** | An update that leaves the old provider wired will look like a rollback that did not happen **(verified** — Adobe's AEM guidance) |
| **Test in a framework, not on a classpath** (§13.3) | A classpath test does not exercise the resolver, and the resolver is what breaks |
| **Stage at bundle level; keep the ability to stop and start a bundle independently; treat "installed but not started" as a legitimate deployment state** | This is the capability you bought. A deployment that assumes every bundle must always be active forfeits the value while keeping the cost **(flagged ⚠-structural** — practice guidance) |

---

## 16. When OSGi Is the Wrong Choice

A guide that only advocates is not a guide. OSGi carries real costs, and they are the same costs that make it valuable in the right place.

**Do not adopt OSGi when:**

| Situation | Why OSGi is the wrong tool |
| --- | --- |
| **You need reliable configuration and strong encapsulation, and nothing more** | That is what JPMS is for, it ships with the JDK, and it costs you a build configuration rather than a runtime and a discipline **(verified** — JSR 376's stated two needs are exactly these) |
| **Your application is a short-lived, stateless, single-deployment unit** | Resolution, wiring and lifecycle machinery buy you nothing if nothing changes at runtime. The container costs memory, startup time and operational surface |
| **Your team has no appetite for manifest discipline** | The resolver enforces declarations literally. Undisciplined metadata produces unresolved bundles, duplicate class visibility and `uses` conflicts — and those failures appear in production, not in the build |
| **You are trying to fix a dependency-hell problem that is really a design problem** | OSGi resolves *packages*; it does not resolve tangled ownership. A shared mutable library presented to twenty bundles is still a shared mutable library |
| **The only OSGi artefact you would deploy is a third-party product you cannot modify** | Then the product's own guide governs, not this one. Use it as a black box and let the vendor own the alignment constraint — the Talend case in §11.2 is exactly this shape |
| **No one on the team can read a manifest and predict the wiring** | This is the honest disqualifier. OSGi's failure modes are legible only to people who understand the module layer, and section 15.3 is a list of what happens when they are not in the room |

**Do adopt OSGi when the problem has this shape:** a long-lived system, composed of parts with genuinely independent lifecycles, where *changing one part while the rest keeps running* is a business requirement rather than a convenience, and where the parts are supplied by different teams or vendors with different release cadences. That is the shape AEM, Liferay and the integration runtimes in section 11 all have.

**The decision table, compressed:**

| If the requirement is… | Reach for… |
| --- | --- |
| Hermetic compile-time modules, smaller runtimes, JDK-level encapsulation | **JPMS** (`module-info`, `jlink`) |
| Runtime install/update/uninstall with no restart, and a service registry with lifecycle events | **OSGi** (Core + DS, on a container such as Karaf) |
| Both, because the platform is modular and the application must change live | **OSGi on a modular JDK** — the two coexist, and the ecosystem has built the interop (Atomos, jlink images) **(verified** — Felix Atomos subproject description) |
| A plug-in architecture inside an existing product | The product's own extension model first, OSGi only if the product documents it (AEM, Liferay do; check before assuming) |

---

## 17. The Worked Example — Cymbal Bank

### 17.1 The Scenario

**This is an explicitly illustrative scenario in the house's Cymbal Bank persona. It is not a description of any institution's system, and no real institution is used as an example anywhere in this guide.**

Cymbal Bank's treasury platform publishes reference rates to downstream systems. It has three properties that make it an honest fit for OSGi rather than a straw man: it is long-lived (the platform is expected to outlast several of its vendors), it is composed of parts with genuinely independent release cycles (rate adapters supplied by different vendors, business rules owned by different desks), and *changing one part while the rest keeps running* is a business requirement — a rate source is added or retired during the trading day, not in a weekend window.

The wrong first instinct is a monolith with a config-driven adapter switch. It works until the fourth source, at which point the adapters' libraries collide on the classpath, and the platform can no longer be restarted safely during the day.

### 17.2 The Design

| Element | Implementation | OSGi mechanism it relies on |
| --- | --- | --- |
| **Framework** | Apache Felix or Eclipse Equinox, inside an Apache Karaf container | Core layers; the container supplies shell, provisioning and clustering |
| **Rate-adapter contract** | An API bundle exporting `com.cymbal.treasury.rates` at version `1.0.0` | `Export-Package` with a version; **the API bundle is the only artefact every consumer imports** |
| **Each rate source** | A separate implementation bundle importing the API in the range `[1.0,2.0)` and registering a service | `Import-Package` range plus the service registry — the consumer never imports a provider |
| **Consumers** | Components declaring a reference to the rate interface | Declarative Services; the SCR registers, finds, binds and reacts to the service coming and going |
| **Business rules** | Rule bundles supplied by desks and updated independently | Lifecycle install/update/uninstall; rules are stopped, replaced and started without a platform restart |
| **Configuration** | Per-source configuration with stable identities | Configuration Admin with a persistent identity, so a replaced adapter keeps its configuration |
| **Alignment** | A build-enforced statement of the Java level, the Core/Compendium level, the container version and the build tool version | The four axes of §11.2, enforced in CI rather than documented in a wiki |

### 17.3 The Version-Alignment and Rollback Story

The design's value shows up in three operational moments, and each of them is a mechanism this guide has already established:

- **Adding a source mid-day.** Install and start the new adapter bundle. It resolves against the API's `[1.0,2.0)` range, registers its service, and the SCR binds the consumer. Nothing else restarts — the consumers were written to handle binding events because DS forced them to.
- **Upgrading the API.** This is the one that requires discipline. A change to the exported API is a **major** version change, so the API bundle moves to `2.0.0` and both API versions can coexist; each consumer moves when its own release allows. The alternative — editing `1.0.0` in place — breaks the contract the resolver is enforcing on everyone's behalf.
- **Rolling back a bad adapter.** Stop the bundle: its registration goes away, the SCR fires unbind, and consumers fall back to whatever the design says they fall back to. **This is a state the platform supports rather than an incident** (§15.4). If the adapter was *updated* rather than stopped, the operator must remember the AEM-documented behaviour and **refresh explicitly**, or the old version may still be wired (§15.3).

### 17.4 What Went Wrong in the First Design

| First attempt | The failure it produced | The correction |
| --- | --- | --- |
| Consumers imported the *provider* bundles directly | Adding a source meant recompiling and redeploying consumers | Consumers import only the API bundle; providers are discovered |
| All adapters shaded a common HTTP client | After two sources were installed, `instanceof` failed on a nominally identical class — the classic duplicate-class-visibility symptom (§15.3) | One bundle provides the client and exports its packages; adapters import them |
| Configuration lived in per-adapter property files | A replaced adapter lost its configuration and silently took defaults | Configuration Admin with a stable persistent identity |
| Bundles were built by hand-rolled manifests | Bundles resolved in development and failed to resolve in the target container, because a hand-written `Import-Package` list did not match the bytecode | bnd in the build (§13.1) — manifests computed, not written |

**The generalisable lesson:** every failure above was a *declaration* failure, not a code failure. That is the OSGi property that makes it powerful and the property that makes it expensive.

---

## 18. The One-Page Summary and the Claims Ledger

### 18.1 The One-Page Summary

| Question | Answer |
| --- | --- |
| **What is OSGi?** | A specification (Core, Compendium, historically Enterprise) for runtime modularity on the JVM — a module layer with a resolver, a lifecycle layer, and a service registry — plus implementations (Felix, Equinox), containers (Karaf) and tooling (bnd/bndtools) |
| **Who governs it now?** | The **OSGi Working Group at the Eclipse Foundation**, hosting the OSGi Specification Project and the OSGi Technology Project, both **Incubating**. The OSGi Alliance transferred the work at the end of 2020 and no longer operates |
| **What is the latest release?** | **OSGi Compendium 8.1, December 2022** (Core 8 was October 2020). No separate Enterprise document since R7, December 2018 |
| **Is it dead?** | **No** — a funded twelve-month modernisation programme started April 2026, draft specifications are still built, and bnd is releasing actively. **Is it growing?** No — 11 contributors, 227 commits, no published spec release since 2022 |
| **What is it for?** | Long-lived systems composed of parts with independent lifecycles, where changing one part while the rest keeps running is a requirement |
| **When not to use it?** | When JPMS or the classpath is enough — that is most new applications (§16) |
| **The single most important operational fact** | The first diagnostic question is never "is it up?" but "**what is the resolved wiring?**" (§4.3) |

### 18.2 What Could Not Be Verified

The following items are recorded honestly rather than asserted. Each states what was checked and what was found **(all checks performed 2026-09-17)**:

| Item | Status |
| --- | --- |
| Publication dates of **Release 1** and **Release 2** of the OSGi specification | **Could not verify.** The documents are listed and downloadable at `docs.osgi.org/specification/`, and their front matter was **not** read this pass. No year is claimed for either anywhere in this guide |
| The exact publication date of **Compendium 8.0.0** (as distinct from 8.1's December 2022) | **Could not verify.** The document is listed under Release 8 but its own date was not read |
| The **editorial reason** the Enterprise document set ended at R7 and its chapters moved into the Compendium | **Could not verify.** The *observation* that no Enterprise 8 exists, and that enterprise chapters appear in Compendium 8.1, is verified from the documentation set; the rationale is not stated by any source located |
| Whether **Apache Felix's framework implements Core R7 or R8** | **Could not resolve — the project's own documentation disagrees with itself** (§12.2). Recorded as a discrepancy rather than a fact |
| **Sonatype Nexus** being an OSGi server | **Could not verify at the vendor.** Attributed to this repository's own guide; Sonatype's documentation was not fetched |
| **Apache Sling's** OSGi dependency, stated in one sentence | **Could not verify by quotation.** Strongly supported (OSGi-centric documentation, AEM's Sling defaults, the Felix console) but not quoted from a Sling page |
| **Pax Exam's** current maintenance status | **Could not verify.** No claim is made |
| **Adoption statistics** of any kind — market share, bundle counts, deployments | **Not verifiable from any source located.** The working group's "countless embedded and IoT systems" is promotional; the Foundation's metrics count contributors, not users. Nothing in this guide converts either into a number |
| **Any named bank or banking product using OSGi** | **Could not verify, and deliberately not claimed.** No vendor documentation located this pass states it. Section 17's scenario is illustrative and names no institution |
| A **complete changelog** of what each specification release added | **Not attempted.** Section 9.2's table is a skeleton drawn from the documents' own front matter and tables of contents, not an exhaustive feature history |
| Whether current frameworks have **dropped or adapted** the Security Manager–dependent permission model | **Could not verify.** The JDK-side deprecation (JEP 411, JEP 486) is verified; the framework-side response was not examined this pass, and no claim is made about it |

---

## 19. Glossary

- **bnd / bndtools:** the build tool that computes bundle manifests from bytecode and its Eclipse IDE integration; the de facto toolchain entry point (7.4.0, August 2026).
- **Bundle:** a JAR whose manifest declares what it provides and needs; the unit of deployment.
- **Compendium:** the specification document carrying the standard services (Configuration Admin, Log, Http, Declarative Services, Remote Services and the rest).
- **Declarative Services (DS):** the standard component model in which a component declares its services and dependencies and the Service Component Runtime does the wiring; **version 1.5** in Compendium 8.1.
- **Equinox:** Eclipse's framework implementation; the base of the Eclipse IDE; on a quarterly release train (4.41, August 2026).
- **Feature (R8):** a declarative description of an application — bundles, configuration, properties, extensions — composable and reusable; §159 of Compendium 8.1.
- **Felix:** Apache's framework implementation and its large subproject portfolio (SCR, Config Admin, HTTP, Gogo, Web Console, Health Checks and more).
- **Karaf:** the surviving OSGi container — shell, provisioning, features, instances, clustering (Cellar) and monitoring (Decanter); 4.4.11, April 2026.
- **JPMS:** the Java Platform Module System delivered by JSR 376 in Java 9; compile-time and start-up modularity with strong encapsulation, and without OSGi's dynamic lifecycle.
- **Resolved wiring:** the specific provider chosen for each requirement of each bundle — per-consumer, inspectable, and the first thing to look at in an incident.
- **SCR (Service Component Runtime):** the engine that reads DS component descriptions and performs registration, binding and lifecycle.
- **Service registry:** the publish/find/bind broker that lets consumers use providers without a compile-time dependency on them.
- **Stale reference:** what a consumer holds when its provider has gone away; the reason the tracker exists.
- **`uses` constraint:** the rule that a package's dependencies must be consistently wired across a boundary; the source of the most confusing resolution failures.

---

## 20. Cross-References and Further Reading

**Sibling guides in `technology/` — the integration and platform cluster:**
- **[talend_guide.md](talend_guide.md)** — **THE primary OSGi-in-a-product cross-reference.** Its description of Talend Runtime as the "OSGi/Apache-Camel-based container" and its documented Studio/Runtime version-alignment constraint are cited in §11.2 and generalised in §15.4. Read it for the Talend product; read §11.2 here for why the constraint is an OSGi fact.
- **[camel_camelk_research.md](camel_camelk_research.md)** — the Camel integration layer that in several products (Talend among them) sits *on top of* an OSGi container; the two stacks frequently appear together, and the boundary is the same one §11.2 draws.
- **[enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md)** — the platform-selection frame into which §12's container table and §16's decision table feed.
- **[data_integration_frameworks_guide.md](data_integration_frameworks_guide.md)** and **[legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md)** — the integration-pattern context; adapter-style modularity (§17) is a legacy-integration concern before it is an OSGi one.
- **[sonatype_nexus_guide.md](sonatype_nexus_guide.md)** and **[jetty_guide.md](jetty_guide.md)** — **the two product guides whose OSGi facts are attributed, not re-derived, in §11.3.** Use them for the products; use this file for the OSGi layer.
- **[../technology/architecture/enterprise_architecture_guide.md](../technology/architecture/enterprise_architecture_guide.md)** — the enterprise-architecture frame for §16's decision table.
- **[jib_container_builder_guide.md](jib_container_builder_guide.md)** — container image building for JVM applications; the last mile for a Karaf or embedded-framework deployment (§5.2).
- **[test_orchestration_guide.md](test_orchestration_guide.md)** — test orchestration at the estate level; §13.3's in-framework testing is the OSGi-specific layer beneath it.

**Guides in `banking/` — the operational-governance frame:**
- **[../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md)** — the resilience and change-management frame in which §15.4's upgrade discipline and §17.3's rollback story belong.
- **[../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md)** — the risk-management frame for §11.4's refusals (no unverified adoption claims, no named institutions) and for the technology-risk treatment of a platform whose specification line has paused.

**Primary sources and further reading:** the specification documents themselves at `docs.osgi.org` — start with **Core Release 8** (October 2020) for the module, lifecycle and service layers and **Compendium Release 8.1** (December 2022) for Declarative Services 1.5 and the Feature Service; the OSGi Specification Project and Technology Project pages at `projects.eclipse.org` for current state; `metrics.eclipse.org/wgs/osgi/` for activity; `felix.apache.org`, `karaf.apache.org`, `sling.apache.org`, `download.eclipse.org/equinox/` and `github.com/bndtools/bnd` for dated status; the JCP records for JSR 277, 291, 294 and 376; OpenJDK JEP 411 and JEP 486 for the Security Manager trajectory; and the vendor documentation of Adobe and Liferay, which remain the clearest public demonstrations of what an OSGi platform looks like from the outside.

---

**End of guide.** OSGi rewards teams who treat the manifest as a contract and the container as the enforcement point; the payout is a system whose parts can change while it runs, and the cost of that payout is a discipline you cannot delegate to a framework. The one question that separates a working OSGi estate from a fragile one is whether anyone can answer, on demand and at three in the morning: what is the resolved wiring.
