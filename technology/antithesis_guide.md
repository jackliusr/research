# Antithesis — The Deterministic Simulation Vendor: FoundationDB's Lineage, the Determinator Hypervisor, and a Business Built by Selling the Simulated Environment

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — Reliability & Verification Engineering series (company deep-dive on a commercial testing platform; read alongside the deterministic-engineering and deterministic-simulation-testing guides)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary sources:** antithesis.com product page including its FAQ answers (retrieved September 2026); antithesis.com/docs "How Antithesis works" (retrieved September 2026); **Alex Pshenichkin, "So you think you want to write a deterministic hypervisor?"**, antithesis.com, 20 March 2024; antithesis.com/company/about (founding timeline and leadership list, retrieved September 2026); **PR Newswire, "Jane Street Leads Antithesis's $105M Series A…", Tysons Corner VA dateline, 3 December 2025** (vendor-issued release); antithesis.com/blog/ethereum_merge (13 February 2024); **antithesis.com/blog/mongo_bug — guest post by Dan Gottlieb, former Senior Staff Engineer at MongoDB (22 April 2024)**; **GitHub wiki, mongodb/mongo → "Testing MongoDB with Antithesis" (last edited 18 February 2022)**; **turso.tech/blog (January 2025)** as recorded in this repository's `turso_database_guide.md`; **github.com/pgraug/dhyve-src README (retrieved September 2026)**; TigerBeetle `docs/internals/vopr.md`; SE Radio episode 685, "Will Wilson on Deterministic Simulation Testing" (September 2025).
> **Last Updated:** September 2026

---

## Table of Contents

1. [Overview, Identity and Decoder](#1-overview-identity-and-decoder)
2. [The Company — The Identity Gate](#2-the-company--the-identity-gate)
3. [Origin and the Problem It Addresses](#3-origin-and-the-problem-it-addresses)
4. [What the Product Is](#4-what-the-product-is)
5. [How It Is Used End to End](#5-how-it-is-used-end-to-end)
6. [The Evidence](#6-the-evidence)
7. [The Commercial Model](#7-the-commercial-model)
8. [Competitive Landscape and Positioning](#8-competitive-landscape-and-positioning)
9. [The Technique in Brief](#9-the-technique-in-brief)
10. [Who This Is For and Who It Is Not For](#10-who-this-is-for-and-who-it-is-not-for)
11. [The Regulated-Enterprise and Banking Angle](#11-the-regulated-enterprise-and-banking-angle)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [Anti-Patterns](#13-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary, Cross-References and Closing Summary](#16-glossary-cross-references-and-closing-summary)

---

## 1. Overview, Identity and Decoder

### 1.1 What This Guide Is

This is the company-and-product deep-dive on **Antithesis** (antithesis.com), a United States commercial software vendor that sells **deterministic simulation testing (DST) as a service**. It is a per-company guide in the `technology/` series that already contains the technique-level and practice-level treatments of DST, and it is deliberately narrow: it covers **the company, the product, the evidence and the commercial model**, and it does not re-derive the engineering method. Everything the reader needs about *how* DST works is one cross-reference away (§1.5).

The company is easy to describe and hard to assess. It sells one thing — access to a simulated computer in which a customer's whole distributed system runs deterministically under injected faults — and it is, on the published record, the only vendor selling that as a product rather than practising it internally. It emerged from stealth in February 2024, announced $47M in seed funding in the same month, announced a $105M Series A led by **Jane Street** on 3 December 2025, and published no price, no licence terms and no service-level commitments at any point this guide examined. What follows separates, deliberately and throughout, three different grades of statement: **documented fact**, **company claim**, and **customer claim**.

### 1.2 The Thesis, In One Line

> **Antithesis sells the simulated environment — the one component deterministic simulation testing requires and almost nobody can build for themselves.**

The technique is not secret. It has been public since a FoundationDB engineer described it at a conference in 2014 (§9, and `deterministic_engineering_guide.md` §5). What has never been public is a **product** for it, because DST normally requires the team to own its runtime, scheduler, clock and network — which is why, historically, it has existed inside a handful of organisations that wrote their own database and their own concurrency language to get it. Antithesis's commercial insight is that the *environment* can be owned by a third party and rented, leaving the customer with only the two things it was always going to have to supply anyway: a containerised system and a set of invariants worth asserting. The whole company is an elaboration of that one bet. Whether the bet is well-aimed is the subject of §8 and §10; whether the evidence supports it is the subject of §6 and §14.

### 1.3 Two Category Distinctions, Stated Plainly

Two comparisons cause more confusion than any other part of this subject, and both are settled here before the product is described.

**(i) Antithesis versus chaos engineering.** Chaos engineering injects **real faults into a real, running environment** — it kills real processes, partitions real network links, and observes real systems. It is not deterministic: the fault schedule is not reproducible at the interleaving level, so a failure discovered by a chaos experiment is usually a failure you now know *exists* and still cannot reproduce on demand. Deterministic simulation runs **the real code against a simulated environment**: time, network, disk and scheduling are supplied by a simulator rather than by the OS, and because every input descends from a seed, a discovered failure is reproducible by re-running that seed. The practical difference is therefore not "simulated versus real" (both run real application code) but **determinism plus guided search** — the simulator searches the state space deliberately and can rewind to the exact instruction where the violation occurred. The category-level treatment is `chaos_engineering_guide.md`, which owns what chaos engineering actually is; this guide only records where Antithesis sits relative to it.

Note one framing fact a buyer will meet, because the vendor states it itself: Antithesis's own documentation describes running in Antithesis as **"an extreme version of chaos engineering, where *everything* is running in our fault-filled environment — not just your software, but your dependencies, your client, and your checkers as well."** ⚠ That is **the vendor's framing, not a definition**; read literally it is a rhetorical bridge, and the precise difference is the one above — chaos engineering is a real-environment discipline without reproducibility of interleavings, and DST is a simulated-environment discipline whose defining property is exactly that reproducibility. When the vendor's own FAQ asks "How is Antithesis different from chaos testing?", the answer it gives is the same distinction: unlike "chaos testing's blind fault injection, Antithesis intelligently guides fault injection and input data generation to maximize coverage" and "every bug found with Antithesis is perfectly reproducible". ⚠ (vendor FAQ, retrieved September 2026).

**(ii) Antithesis versus Jepsen-style black-box verification.** Jepsen interrogates **real deployed systems** with **real faults** — network partitions, process pauses, clock skew applied to actual machines — and searches for a violation of a stated consistency property, such as linearizability. It is a black-box, adversary-driven, end-to-end discipline applied to a real cluster, and its outputs are analyses and histories rather than a replayable seed. A deterministic simulator does the opposite in one crucial respect: it **reproduces a failure by seed**, because the entire run — every input, every fault, every scheduling decision — is a function of that seed and the binary. Both are rigorous; they are rigorous about different things. Jepsen-style verification tests the system you actually deploy, in the configuration you actually deploy, which a simulator by construction does not; a simulator can reach interleavings that no black-box test will hit within a human lifetime, and it can hand you the exact instruction at which the invariant broke. The repo names both; `deterministic_engineering_guide.md` §11 is where the property-based and model-based testing families are treated, and this guide does not re-derive either.

Also worth naming here so it is never confused with the platform: **generic fuzzing** (AFL/libFuzzer-style input fuzzing) explores a *parser's or single process's* input space and is not a whole-system environment; **formal methods** (TLA+, Coq, model checkers) prove properties of a *model*, and the vendor itself publishes a blog post titled "Finding bugs in Raft implementations: why formal verification is not enough to maintain consensus in real systems" (27 July 2026) — which is the honest positioning, and also a competitor's argument against the vendor run in reverse.

### 1.4 Decoder — The Vocabulary You Will Meet

| Term | What it means in this guide, and who is asserting it |
|---|---|
| **Deterministic simulation testing (DST)** | Running the real production code against a simulated, seeded, controllable environment in which time, network, disk and process faults are supplied by the simulator rather than the OS, so that a run is reproducible from a seed. The technique; defined and owned by `deterministic_engineering_guide.md` §5. Not a product. |
| **Deterministic hypervisor** | A hypervisor that enforces determinism on everything inside a virtual machine — instruction stream, time, interrupts, I/O — rather than merely virtualising hardware. Antithesis's is called **the Determinator** and is a fork of FreeBSD's **bhyve**. ⚠ (vendor engineering blog, 20 March 2024). |
| **The environment / "the Antithesis environment"** | The vendor's term for the simulated world in which the customer's containers run: one x86 virtual machine, all containers inside it, all timing and I/O virtualised. The unit of reproducibility is stated by the vendor as "the state of the entire system/experiment/workload as an interconnected whole, not any single process or server within the system." ⚠ (vendor, 20 March 2024). |
| **Fault injection** | The deliberate introduction of faults — network partitions, packet loss and latency, node kills, disk and storage faults, thread starvation, clock effects — as a first-class, seed-derived part of the run. The vendor states the simulator injects "clock, scheduling, network, and storage faults with zero configuration." ⚠ (product page, September 2026). |
| **Reproducible failure** | A failure that recurs on demand, because the run is a pure function of seed and binary. This is the product's central claim and the reason determinism matters commercially. |
| **Assertion / invariant** | The customer-written statement of how the system must behave — "this must never happen", "this must always hold after recovery". In Antithesis, assertions are written **in the customer's own test harness**, and the platform scans the run for deviations. **No invariants means nothing to find** (§4.5). |
| **Time-travel debugging** | Interactive debugging in which the run can be suspended, rewound and inspected, and destructive analysis performed, because the state is deterministic. Vendor term. ⚠ |
| **The workload / container model** | The customer's system, packaged as container images and orchestrated with Docker Compose or Kubernetes, **including its dependencies** (the vendor names Kafka, Postgres, Redis, NATS, Ceph), launched inside the environment. The container registry and credentials are supplied by the vendor after contact. ⚠ (vendor FAQ) |
| **The multiverse** | The vendor's name for "the collection of timelines produced in a single Antithesis test run" — the tree of branching execution paths created by fault injection and input randomisation, described as "tens (or hundreds) of thousands of alternate universes". ⚠ (vendor docs) |
| **Causality analysis** | The vendor's automated root-cause feature: pinpointing the events in the run that are causally related to a bug, and reporting the bug's probability over time across the multiverse. The mechanism was prototyped with MongoDB and turned into a product feature. ⚠/✅ (§6) |
| **Property-based testing (PBT)** | Asserting invariants over generated inputs rather than asserting outputs for hand-written examples; a failing case carries a seed. The technique is general; Antithesis implements a form of it, and separately publishes **Hegel** and **Bombadil**, open-source PBT tools. `deterministic_engineering_guide.md` §11.2 owns the technique. |

### 1.5 The Boundary — What This Guide Owns, and What It Does Not

Read this subsection before reading anything else in the file; it is the reason the rest of the guide is short in places where a reader might expect it to be long.

| Subject | Owner | This guide's relationship to it |
|---|---|---|
| **The technique** — the definition of DST, the "this is not mocking" distinction, the Will Wilson Strange Loop 2014 talk as canon, the FoundationDB documentation, the implementations table (FoundationDB, VOPR, Shadow, dhyve), the benefits, the five architectural requirements, and the comparison to chaos testing | **`deterministic_engineering_guide.md` §5, "Toolkit II: Deterministic Simulation Testing"** | Cited by name, never re-derived. §9 of this guide states DST in a few lines and hands off. |
| **The construction and practice** — how a team actually builds and runs deterministic simulation: harness design, stub boundaries, seed handling, fault models, workload design, the organisational cost | **`deterministic_simulation_testing_guide.md`** (sibling guide in this directory) | Cited by name. This guide describes what a *vendor* supplies and where the customer's engineering obligation still begins (§5). |
| **The category it is confused with** — chaos engineering: real faults, real environments, blast radius, game days, steady-state hypotheses | **`chaos_engineering_guide.md`** | Cited by name in §1.3 and §8. This guide records the vendor's framing of itself relative to that category and the precise technical difference; it does not teach chaos engineering. |
| **The other kind of rigorous testing** — Jepsen-style black-box verification of real deployed systems, and its relationship to property-based and model-based testing | **`deterministic_engineering_guide.md` §11** | Cited by name in §1.3 and §8. Not re-derived. |
| **The company and the product** — identity, origin story, what is actually licensed, deployment options, the evidence for customer claims, the commercial model, the pricing *absence*, the lock-in question, and the fit assessment for a regulated buyer | **This guide** | Owns it. |

So: if you have come to learn deterministic simulation testing, put this file down and read `deterministic_engineering_guide.md` §5 and `deterministic_simulation_testing_guide.md`. If you have come to decide whether to buy something, keep reading.

---

## 2. The Company — The Identity Gate

This section is deliberately narrow, and it follows this repository's standing discipline: establish who the corporate subject actually is, mark every statement with ✅ (verified at source), ⚠ (flagged, vendor claim, or third-party-derived) or ❌ (rejected), and state plainly what could not be established. For a vendor whose product is a *trust* product — you are asked to believe a simulation's result — the identity gate is not paperwork. It is the first diligence question.

### 2.1 What Is Established

| Item | Finding | Grade and source |
|---|---|---|
| **Legal entity name and jurisdiction** | ❌ **Not established.** No registered legal entity name, incorporation number, state of incorporation or corporate registry filing (e.g. a Virginia State Corporation Commission or Delaware Division of Corporations record) was located in this research pass. The press release's dateline is **TYSONS CORNER, Va.** and its boilerplate says the company "is based in Northern Virginia, was founded in 2018, and launched out of stealth in 2024"; some third-party aggregators place it in "Vienna, Virginia", which is also Northern Virginia. Both are locality statements, not an entity registration. | ❌ **Searches returned empty** for registry records; recorded as a tool and coverage limitation (§15), not as evidence that no entity exists. ⚠ locality from the vendor's own release, 3 December 2025. |
| **Founding year** | **January 2018** (the company's own About-page timeline: "January 2018 — Will Wilson and Dave Scherer start Antithesis"). Consistent with the release boilerplate's "founded in 2018". | ⚠ **Company claim** — self-published timeline. No registry filing read. |
| **Founders** | **Will Wilson** (CEO & Co-Founder) and **Dave Scherer** (Co-Founder). | ⚠ Company's own About page (retrieved September 2026). |
| **Founders' documented prior affiliation** | Both came from **FoundationDB**. The About page states 2009 as FoundationDB's founding by **Dave Scherer, Dave Rosenthal and Nick Lavezzo**; **April 2013** as "Will Wilson joins FoundationDB, starts working on deterministic simulation harness"; and **March 2015** as "Apple acquires FoundationDB". The Apple acquisition itself is independently and widely reported — **that specific fact is documented, the surrounding chronology is the company's own account.** | ⚠ company account, with ✅ for the Apple acquisition as a widely reported event. |
| **Headquarters** | **Tysons Corner, Virginia** — the dateline of the vendor-issued release; boilerplate: "based in Northern Virginia". ⚠ Some aggregators give "Vienna, Virginia". Both are consistent with Northern Virginia. | ⚠ vendor release, 3 December 2025; aggregator variants flagged. |
| **Funding history — seed** | **$47M seed, announced February 2024**, "led by Amplify Partners, Tamarack Global, and First in Ventures". The company also states it **emerged from stealth in February 2024** and that **customer no. 1 started in September 2019**. | ⚠ Company's own About-page timeline (self-published). |
| **Funding history — Series A** | **$105M Series A announced 3 December 2025**, **led by Jane Street**, "with additional investors **Amplify Venture Partners, Spark Capital, Tamarack Global, First In Ventures, Teamworthy Ventures, Hyperion Capital**, and individuals including **Patrick Collison, Dwarkesh Patel and Sholto Douglas**". | ⚠ Vendor-issued press release (PR Newswire, 3 December 2025). Note the discrepancy in the lead seed investor's name between the two sources ("Amplify Partners" in the 2024 timeline vs "Amplify Venture Partners" in the 2025 release) — recorded as published, not reconciled. |
| **Ownership and control** | ⚠ **Only the named investors are known.** There is **no shareholder register, no cap table, no ownership percentage, and no statement of who controls the company** in the record examined. The single control-relevant fact that *is* published is the **dual role of Jane Street: "Jane Street is both an Antithesis investor and customer."** (vendor release, 3 December 2025). Neither the board's composition nor any board seat attached to the round is disclosed. | ⚠ vendor release; ❌ control facts not established. |
| **Size** | ❌ **Not published.** The company publishes no headcount. Aggregator figures (e.g. a corporate-client count and revenue band on startup-data sites) were **not verified and are not repeated here**; per this repository's rule, aggregator data of that kind is noise. The only size-adjacent datum the company itself offers is a growth claim: "increased revenue by over 12x over the last two years" ⚠ (vendor release, 3 December 2025) (§14). | ❌ not established. |
| **Leadership as documented** | **Will Wilson** (CEO & Co-Founder), **Dave Scherer** (Co-Founder), **Nick Lavezzo** (Chief Operations Officer — himself a FoundationDB founder), **Randall Jackson** (Chief Revenue Officer), **Ben Collins** (Chief Technology Officer), **Jeremy King** (VP of Product), **Hao Guan** (VP of Sales), **McKenna Dullea** (Controller & VP of Finance), **Max Jacobson** (Field CTO EMEA), **Akshay Shah** (Field CTO North America). | ⚠ single source: the company's own leadership page (retrieved September 2026). |

### 2.2 A Documented Change of Title, Recorded Without Inventing a Reason

There is a **role-over-time nuance** a reader doing diligence will trip over, and it is worth putting on the record precisely:

- ⚠ The engineering blog post of **20 March 2024** refers in passing to "our CTO, **Dave Scherer**".
- ⚠ The current leadership page lists **Ben Collins as Chief Technology Officer** and **Dave Scherer as Co-Founder**, with no CTO title.

That is a **documented change of title between March 2024 and September 2026**. This guide records the change and **does not speculate about its cause**, and it found no company statement explaining it. It is not a red flag by itself — founder-CTO-to-co-founder transitions are ordinary — but for a buyer, *who technically owns the architecture* is a legitimate question, and the answer in the current record is: the CTO role is Ben Collins's, and the co-founder who "was the first person to work on the Determinator, alongside our CTO, Dave Scherer" (in that post's own words) is no longer titled CTO.

### 2.3 The Conflict-of-Interest Fact a Buyer Should Weigh

The December 2025 press release contains one sentence that belongs in every evaluation file, in bold:

> ⚠ **"Jane Street is both an Antithesis investor and customer."** (vendor-issued release, 3 December 2025)

Jane Street **led the Series A** and is simultaneously **a paying customer whose testimonial appears on the vendor's own site** and a quoted voice in the funding announcement itself ("Antithesis has helped us uncover issues that no other testing method could find" — Doug Patti, Engineer, Jane Street, in the same release). None of that is improper, and lead investors who are also customers are common in enterprise infrastructure. But it has three concrete consequences a buyer should price in:

1. **The most-cited customer testimonial is furnished by the lead investor** — the strongest available evidence of product value is not independent of the company's financing.
2. **Roadmap gravity.** A lead investor that is also a customer has a route to product direction that a non-investor customer does not.
3. **Competitive sensitivity.** For a firm in the same business as the lead investor, "our workload runs on the same platform as our competitor's" is a question worth asking explicitly and early. This guide does not know whether Antithesis's contracts restrict that; nothing in the public record addresses it (§15).

### 2.4 What the Company Says About Itself in One Sentence

⚠ **Company claim:** "Antithesis is a fundamentally new way to validate that software works correctly before it is released. It conducts a fully deterministic, massively parallel simulation that tests years of real-world production in a few hours." (vendor release, 3 December 2025.) Marked as a claim because "years of real-world production in a few hours" is a marketing construction with no stated baseline, denominator or measurement method — §14 audits it as such.

### 2.5 What Could Not Be Established About the Company Itself

Stated here in summary and expanded in §15: ❌ the registered legal entity name and jurisdiction; ❌ any corporate registry filing (incorporation number, officers, registered agent); ❌ ownership percentages or a shareholder list beyond named investors; ❌ board composition and board seats attached to the Series A; ❌ headcount; ❌ revenue, burn or any financial statement of any kind; ❌ whether the "12x revenue growth" claim has any corroboration; ❌ whether any customer contract contains exclusivity, most-favoured-nation or competitive-restriction terms; and ❌ any company statement explaining the CTO title change. The identity gate is, for this vendor, **substantially open** — which is normal for a private venture-backed company in 2026 and abnormal for an enterprise purchaser who expects a vendor's registration to be checkable before a contract is signed. It is checkable in practice (a vendor can simply supply its registration and a certificate of insurance); it is not checkable *from outside*, and this guide's finding is that it was not checkable from the public record it examined.

---

## 3. Origin and the Problem It Addresses

### 3.1 The Historical Trap: DST Lived Where the Runtime Lived

Deterministic simulation testing was not hidden; it was **unaffordable to adopt**. Its canonical home was a single distributed database, and the reason it stayed there is architectural rather than cultural. To simulate a system deterministically you must own the boundary through which it touches the world: the clock, the scheduler, the network, the disk, and randomness. A team building an ordinary application on a standard runtime owns none of those. Its code reads `System.currentTimeMillis()`, calls a socket library, blocks on a lock and receives OS scheduling decisions; there is no seam at which a simulator can be inserted without rewriting the system around one.

The organisations that solved this did it by **owning the runtime itself**. That is the FoundationDB lineage, and it is not re-explained here: `deterministic_engineering_guide.md` §5.2 owns the canon (Will Wilson's Strange Loop 2014 talk; the FoundationDB documentation's account of simulation as "enabled by and tightly integrated with Flow"); §5.5 owns the five architectural requirements the technique imposes; §5.3 owns the implementations table. What matters *for this guide* is the shape of the trap: **the technique was trapped inside the few organisations that owned their own runtime**, because the environment — the thing that must be deterministic — was the runtime.

### 3.2 The Commercial Insight

Antithesis's insight is a repackaging of that constraint. If the hard part is **owning the environment**, then the environment does not have to be owned by the customer. Put the customer's system inside somebody else's deterministic environment and the customer no longer needs to own its runtime, its scheduler, its clock or its network — it needs only to make its system launchable in containers and to say what must be true about it.

The vendor's own account of why the environment has to be a hypervisor rather than a library is unusually candid and worth quoting, because it is the clearest statement of the commercial thesis:

> ⚠ **Vendor claim:** "We thought about this and decided to just go all out and write a hypervisor which emulates a deterministic computer. Consequently, we can force anything inside it to be deterministic." ("So you think you want to write a deterministic hypervisor?", 20 March 2024, quoting the company's own 2024 launch post.)

And the blunt corollary, from the same post: building the environment is **"borderline-impossible stuff"** — the phrase the vendor uses in its own hiring call to action. The company's business model is, in effect, a wager that this remains true.

### 3.3 The Founding Narrative, Attributed

The founding story below is **the company's own account of itself**, published on its About page. It is set out here because it is the origin story a buyer will be told in the sales cycle, and because a diligence file should record which parts are corroborated and which are not.

> ⚠ **Company account** (antithesis.com/company/about, retrieved September 2026):
>
> "In 2009, we began building FoundationDB — a strongly-consistent, highly-available distributed database that most experts thought impossible. … So before writing the database itself, we invented a deterministic simulation framework that could find and perfectly reproduce even the subtlest bugs. **It worked! Despite our breakneck development speed, we only shipped one user-reported bug before Apple acquired FoundationDB in 2015.**"
>
> "After the acquisition, our team dispersed into many big tech companies — and we were shocked to find that nothing like our simulation framework existed anywhere else. The most sophisticated engineering organizations on Earth were losing months to production bugs that our approach would have caught in hours. Worse, their fear of outages made them unbearably slow."
>
> "**In 2018, Will Wilson and Dave Scherer reunited to found Antithesis.** Their mission was simple: to bring the superpower of deterministic simulation testing to every mission-critical software project."

**How to read that passage, in three notes.** First, **"only shipped one user-reported bug"** is a ⚠ **company claim**, unpublished elsewhere in a form this guide could check, and it is a marketing construction rather than a measurement: "user-reported bug" excludes bugs found by any other means, and no denominator (releases, customers, years, transaction volume) is offered. It is audited as such in §14 and must not be repeated as a fact. Second, the claim that **"nothing like our simulation framework existed anywhere else"** is contradicted by the company's own subsequent behaviour and by the record: the *technique* was public from 2014, was documented by FoundationDB itself, and TigerBeetle's documentation states its own simulator "was heavily inspired by the work of FoundationDB and Antithesis" — what did not exist was a *product*, which is the accurate version of the claim. Third, the mission statement — **"to bring the superpower of deterministic simulation testing to every mission-critical software project"** — is the sentence that answers §8's positioning question, and it is a claim of universality that the product's own preconditions (§5.3) do not support. Not every project can be brought.

### 3.4 The Timeline, as the Company Publishes It

| Date | Event | Grade |
|---|---|---|
| **2009** | Dave Scherer, Dave Rosenthal and Nick Lavezzo start FoundationDB | ⚠ company timeline |
| **April 2013** | Will Wilson joins FoundationDB, starts work on the deterministic simulation harness | ⚠ company timeline |
| **March 2015** | Apple acquires FoundationDB | ✅ widely reported independently; ⚠ for the "everyone moves to California" detail |
| **January 2018** | Will Wilson and Dave Scherer start Antithesis | ⚠ company timeline |
| **September 2019** | Customer no. 1 starts using Antithesis | ⚠ company timeline; customer not named on that page |
| **February 2024** | $47M seed announced, led by Amplify Partners, Tamarack Global and First in Ventures | ⚠ company timeline |
| **February 2024** | Emerges from stealth; "stays at #1 on HackerNews for 24 hours" | ⚠ company timeline (the HackerNews item exists; the 24-hour duration is the company's claim) |
| **December 2025** | $105M Series A, led by Jane Street | ⚠ vendor-issued release |

Five years elapse between founding (January 2018) and the stealth exit (February 2024), during which the company had a customer (from September 2019) and no public profile. That is a long, quiet build for a vendor whose product requires building a hypervisor — and it is a point in the company's favour as a diligence signal, since the first customer predates both funding rounds by approximately four and a half years.

---

## 4. What the Product Is

### 4.1 The Determinator — The Hypervisor Underneath

The foundation of the platform is a **custom deterministic hypervisor** which the vendor names **the Determinator**, built as a **fork of FreeBSD's bhyve** by stripping standard hypervisor functionality out and enforcing determinism on everything inside. The vendor's own engineering account (Alex Pshenichkin, Senior Engineer, 20 March 2024) is the primary source, and the details that matter commercially are these:

- **It runs the customer's whole system on a single CPU core per instance.** "Each instance of the deterministic hypervisor runs on just one physical CPU core"; a 48- or 96-core host runs that many separate VMs, each pinned to a core, "exploring different parts of the possible program state in parallel". The vendor states it deliberately optimises for **throughput over latency** — aggregate state-space coverage rather than any one VM's wall-clock speed. ⚠
- **It removes real time from the guest.** "Every attempt to access a time source from inside the guest — reading TSC, reading HPET, etc. — returns a virtual time value computed by the hypervisor." The vendor describes inventing workarounds for two limitations of using Intel performance-monitoring counters as the clock: an instructions-retired count miscounted "about one in a trillion instructions", and APIC-delivered threshold interrupts whose delivery overhead varies.
- **It needs a deterministic I/O channel, implemented via the x86 `VMCALL` instruction** — used so that software inside the guest can emit data (logs, artifacts) outward and ingest commands or RNG seeds inward. "The points in execution history where the guest ingests input from the Antithesis platform become possible branch points for future execution. Consequently, the external view of the exploration of a system is **an input tree**."
- **It is x86-only, and the vendor states it is designed around modern Intel server CPUs using VMX** with hardware-supported virtualisation; the vendor's claim is **"Instruction-level determinism works with any x86 binary."** ⚠

`deterministic_engineering_guide.md` §5.3 records the Determinator/bhyve fact in its implementations table; this guide does not re-derive the hypervisor and deliberately stops at the product-relevant line. The technique-level question — whether an instruction-level-deterministic hypervisor is the *right* way to build DST, versus the in-tree simulators that FoundationDB and TigerBeetle use — is §8's subject.

### 4.2 The Workload and Container Model

The unit placed inside the environment is **the whole distributed system**, not a service and not a stub:

- The customer deploys "your whole system to Antithesis — **including dependencies like Kafka and Postgres** — just like you'd deploy to production", orchestrated "with Docker or Kubernetes". ⚠ (product page, September 2026)
- The vendor's FAQ states plainly: **"Your software must support x86 hardware and run in containers orchestrated by Docker Compose or Kubernetes."** Containerised dependencies are explicitly welcome — "Postgres, Redis, Kafka, NATS, Ceph, and a long list of other software works well" — and where a dependency is **not available as a container, "you'll need to use a mock"**, with the vendor publishing a list of ready-to-use mocks for cloud and SaaS services. ⚠
- Multicast networking is claimed to work out of the box with Docker Compose manifests — the vendor's stated reason being that it "runs your whole system on a single physical machine", so multi-node CNI complexity disappears. ⚠ (vendor FAQ; the FAQ also says, unusually specifically, that "many of our capital markets customers use Antithesis to test systems that rely on multicast networking" — a **vendor claim about unnamed customers**, which this guide does not convert into a customer list.)
- **SDKs** exist for **C, C++, Go, Java, JavaScript, .NET, Python and Rust**, with "a low-level fallback API" for other languages. (`deterministic_engineering_guide.md` §5.3 records the latest Java SDK tag as **v1.7.0, dated 2026-09-02** — a verified-at-source maintenance signal.) ⚠/✅
- The customer receives **a container registry and credentials** from the vendor after contacting them — i.e. there is no self-serve path that this guide could find. ⚠ (vendor docs, "How Antithesis works")

### 4.3 The Environment and the Unit of Reproducibility

The environment is described by the vendor as **"one or more computers using a collection of containers, all running within a single virtual machine managed by our hypervisor"**, and its **unit of reproducibility** is stated precisely:

> ⚠ **Vendor claim:** "The unit of reproducibility is **the state of the entire system/experiment/workload as an interconnected whole, not any single process or server within the system**. This approach reduces the need for complicated, domain-specific mocks and test harnesses, since you can just run (for example) both your client and server software together in the same bubble of determinism and perfectly reproduce their synchronized state." (20 March 2024; also quoted in `deterministic_engineering_guide.md` §5.1.)

Architecturally, the vendor describes the hypervisor as "a big box for the whole end-to-end experimental setup to sit in", with everything inside seeing "a single linear history", while outside the box the platform tracks "every execution path ever visited – a tree of possible paths the experiment could take". Two structural consequences for a buyer:

1. **The customer's system must be a single-machine-shaped deployment.** Everything runs inside one VM. The vendor turns this into a selling point (no CNI, no multi-node multicast complexity, no cross-machine clock skew to model); the honest reading is that it also means the simulation is **not** a model of a real multi-host topology's infrastructure layer — a point §10 and §15 carry forward.
2. **The customer's system becomes the platform's input, not its dependency.** The environment is the vendor's; the system inside it is the customer's. That division is the entire product.

### 4.4 Fault Injection, Exploration and Guidance

Antithesis's fault model is **built in and on by default**, which is unusual and is the feature most often misread as chaos engineering:

- The vendor lists, as a product capability: **"Simulate clock, scheduling, network, and storage faults with zero configuration"**, and "Exercise hard-to-reach code paths like exception handling and leader election", and "Prioritize exploration of particularly critical code". ⚠ (product page, September 2026)
- The scheduling dimension is a deliberate fault channel: because the platform controls the guest's scheduler, it can inject **thread starvation** and similar scheduling pathologies. ⚠ (20 March 2024)
- Exploration is **guided rather than blind**. The vendor states the platform "intelligently guides fault injection and input data generation to maximize coverage" (FAQ) and describes the guidance component as follows: "we have an **intelligent guidance component (it uses RL, but you can tell your boss it's AI)** that drives the platform to seek out new and interesting states." ⚠ (vendor docs — the parenthetical is the vendor's own; it is quoted because it is the most concise statement of what the guidance is, and because a buyer should know the level of technical specificity the vendor offers publicly: an RL-based search, no architecture, no metric.)
- **MongoDB's own wiki page** (the customer's source, last edited 18 February 2022) enumerates the fault settings it was running, which is a rare public view of the fault knobs: packet latency (20 ms mean, σ=50); packet drop probability 0.05; network partition events roughly every 4 seconds into up to 2 groups for 0.3–3 s; network teardowns roughly every 0.3 s with reconnection in a different order, each service having probability 0.033 of inclusion; with event options randomly choosing between dropping, queuing or slowing packets by up to a mean of 300 ms at σ=300. ✅ **verified at the customer's own source** — and note what it says structurally: **the workload container is excluded from network fuzzing**, and the workload itself is the customer's test suite. See §6.

### 4.5 The Two Surfaces the Customer Must Fill: Assertions and the Harness

This is the most important paragraph in the section, and it is where most evaluations of this product go wrong.

**The platform searches for violations. A violation must be defined by the customer.** The vendor's own documentation says the platform "does a form of property-based testing. You **directly state how your system ought to behave** (e.g. 'this application should always recover after a single node dies'), and Antithesis **scans the multiverse to see if it ever deviates from those promises**", and that it "just gets you to express these implicit assumptions explicitly, by **writing assertions in your test harness**". ⚠ (vendor docs)

Two obligations follow, and neither can be outsourced to the vendor:

1. **A test harness the platform can drive.** MongoDB's wiki page is the clearest public example of what this means concretely: a **workload container** holding `resmoke` as "the entry point to run the test suite", and a command line the platform runs — `buildscripts/resmoke.py run --suite antithesis_concurrency_sharded_with_stepdowns_and_balancer --shellConnString "mongodb://mongos:27017"` — against a MongoDB deployment described in a `docker-compose.yml` that the platform uses to "spin up the system under test". ✅ (customer's own wiki.) If a team has no end-to-end suite that can be launched this way, the platform has nothing to drive.
2. **Invariants worth asserting.** The vendor supplies the search, the faults and the reproduction; the customer supplies the definition of wrong. This repository already states the consequence in the technique guide — "a simulator with no oracles is a random number generator" (`deterministic_engineering_guide.md` §11.2) — and Antithesis's own onboarding hard-codes it as step one: **"Outline your invariants."**

The vendor has moved toward softening this obligation by shipping **agent skills** (`github.com/antithesishq/antithesis-skills`) that "help with every step of onboarding", including writing "a precise specification of your system's expected behavior" and producing Docker Compose or Kubernetes manifests. ⚠ That is a real reduction in the *effort* of the obligation. It is not a reduction in the *obligation*: a generated specification is still a specification someone has to review and be accountable for (§13).

### 4.6 The Output: Artifacts, Time Travel, Causality

What the customer buys is not only "bugs found" but **bugs made tractable**. The vendor's claims, all ⚠ unless noted:

- **Unified, perfectly ordered traces of distributed execution** ("Get unified, perfectly ordered traces of distributed execution"), in the vendor's web application.
- **Custom artifact extraction** from the system: "Pull custom artifacts from your system, like data files or core dumps." This is not a minor feature; it is what turned MongoDB's investigation (§6) from stalled to solved — the customer asked for "core dumps for every 100 milliseconds within a two second range around the bug". ✅ (customer's own post, 22 April 2024.)
- **Time-travel debugging**: "Suspend execution, rewind time, and analyze your system as bugs occur", including **destructive analysis** — the vendor's engineer states his "favorite benefit of reproducibility is that it allows *destructive analysis* of a buggy system state: I can freely extract before-and-after crash dumps, modify program data to see how it changes the behavior, and generally perturb the system as much as I want without fear". ⚠
- **Causality analysis**, published as a product feature in a blog post dated 11 May 2026 ("When did the bug start?"), described on the product page as: "Causality analysis pinpoints events related to a bug."
- **Bug-report / probability-over-time analysis**: the vendor's Mongo guest post describes what became the bug-report feature as an analysis that "summarized the probability of a failure from any given checkpoint", so that "when the probability spikes between adjacent checkpoints, we know with confidence that the bug happened in that time window". ✅ **Verified in structure at the customer's own account** — the customer confirms the analysis existed, was useful, and that Antithesis "ended up turning it into a full-fledged product".

One vendor claim about the output deserves flagging for what it implies about workflow: "Our simulator is 100% deterministic and **automatically root causes every bug**. Rather than wasting time and tokens chasing flaky heisenbugs, your team can jump straight to a fix." ⚠ ("100% deterministic" and "automatically root causes every bug" are both absolutes, and the customer account in §6 shows the root-causing still took a customer engineer's expertise plus a hand-produced two-second core-dump window.)

---

## 5. How It Is Used End to End

### 5.1 The Vendor's Documented Four Steps

The vendor publishes onboarding as four numbered steps, and they are quoted here verbatim because the sequence itself is the most informative thing on the product page — it tells you exactly where the vendor's contribution starts and where the customer's does not end:

> ⚠ **The vendor's four steps** (product page, retrieved September 2026):
>
> **1. Outline your invariants** — "Decide how your system should behave – what it should do or never do. Our agent skills can help with this, or check out our property catalogs for different system types."
>
> **2. Send us your system** — "Deploy your whole system to Antithesis – including dependencies like Kafka and Postgres – just like you'd deploy to production. Orchestrate with Docker or Kubernetes."
>
> **3. We find your bugs** — "Aggressive fault injection and intelligent exploration inside a fully deterministic environment reveal edges you've never thought of. Our process turns up runtime issues that static review and type checkers simply cannot catch."
>
> **4. And root cause them** — "Our deterministic environment lets us provide you with some nifty debugging tools. Causality analysis pinpoints events related to a bug. Our time traveling debugger lets you time travel and do destructive analysis. We can also teach your agents to use them."

Read the sequence honestly and it becomes an obligation map. **Steps 1 and 2 are the customer's**, and they are the expensive ones: an invariant list and a containerised, drivable deployment. **Steps 3 and 4 are the vendor's**, and they are what is actually being purchased. The product does not remove steps 1 and 2; it is *conditional* on them. A buyer who reads the four steps as a turnkey sequence has misread the product.

**A precondition the vendor states but the four steps do not surface.** The FAQ's answer on supported stacks is the binding technical constraint: **"Your software must support x86 hardware and run in containers orchestrated by Docker Compose or Kubernetes."** ⚠ Where a dependency "isn't available as a container, you'll need to use a mock". So the honest precondition list is five items, not four: (1) x86-buildable software; (2) containerised, with Docker Compose or Kubernetes manifests; (3) non-containerised dependencies either containerised or mocked; (4) a workload/harness the platform can drive; (5) invariants.

### 5.2 The Honest Preconditions, and What Happens When They Are Absent

| Precondition | What it means concretely | What happens if it is absent |
|---|---|---|
| **A system that can be launched and driven in a container** | The whole system, plus dependencies, described as Compose/K8s manifests; the platform starts it and runs a workload against it | The evaluation cannot start. There is no hosted mode that runs your binary in a vacuum; the environment boots *your* topology. MongoDB's public example is exactly this: a `docker-compose.yml` describing a sharded cluster that the platform spins up (§6). |
| **A test harness / workload the platform can drive** | An existing integration or end-to-end suite, or a purpose-built workload container that exercises the system | Nothing drives the system. The vendor's FAQ is candid that onboarding comes in two steps — "first getting your system running in Antithesis, and then gradually adopting property-based testing" — which means a team with no end-to-end suite has to build one before step 3 of the four steps can begin. |
| **Invariants worth asserting** | Explicit statements of the form "must always / must never", expressed as assertions in the harness | **The platform finds nothing.** It searches for deviations from promises. No promises, no deviations. This is the single most common reason a DST engagement produces a green dashboard and no value, and §13 treats it as an anti-pattern. |
| **x86** | The binaries must be x86 (the Determinator uses Intel VMX) | ARM-only workloads are out of scope on the published record. ⚠ |
| **A working baseline of tests** | Some existing quality signal for the new tool to be compared against | Without a baseline, no one can tell whether the platform found more than the existing suite, and the engagement cannot be evaluated (§5.5). |

### 5.3 Onboarding Timelines — Vendor Claim Versus Reality

The vendor claims: **"As little as a day, especially when starting with a single critical sub-system."** And: "Getting up and running often takes just a few hours, especially when using our agent skills and starting with a working Docker Compose or Kubernetes manifest. Property-based testing can also start very simply, perhaps with an existing integration or end-to-end test. From there, your setup can gradually become more powerful — by expanding to include more related systems, by adding more sophisticated properties, or both." ⚠ (vendor FAQ, retrieved September 2026)

Two things to hold in tension. **The claim is scoped narrowly and is therefore credible on its own terms** — "starting with a single critical sub-system", "starting with a working manifest". Getting *something* running in a day is plausible for a team that already has a working Compose/K8s manifest and an existing E2E test. **The claim is not a claim about value.** Nothing in the vendor's material states how long it takes to write invariants good enough to find a real bug, and the vendor's own honesty elsewhere ("gradually become more powerful") concedes that the setup deepens over time. This guide could not verify the timeline claim from any independent source (§15); it is a **vendor claim with a date**, and it should be tested in a paid pilot rather than believed.

### 5.4 What the Customer Receives and What It Does Not

**Receives** (⚠ vendor, unless marked): a container registry and credentials after contacting the vendor; a web application with run dashboards, findings over time, log/trace exploration, bug reports and causality analysis; SDKs for eight languages plus a low-level fallback API; ready-to-use mocks for many cloud/SaaS dependencies; a property catalogue "for different system types"; agent skills; time-travel debugging and artifact extraction; and — per the vendor — support in packaging, workload creation, property definition and triage ("We can also teach your agents to use them").

**Does not receive, on the published record:** ❌ a price list or licence terms (§7); ❌ a published SLA; ❌ a fully on-premise deployment (§7); ❌ a self-serve or free tier (vendor FAQ: "Sadly, no"); ❌ any statement of what happens to your artifacts at contract end (§7, the artefact-lock-in question); ❌ a published description of the guidance algorithm beyond "it uses RL" (§4.4); ❌ any published accuracy or coverage metric for the simulator itself.

### 5.5 How to Run the Evaluation Honestly (Not a Vendor Process)

Neither the vendor nor this guide's sources describe a de-risked evaluation protocol, so here is the one this repository's discipline implies, stated as an evaluation design and **not** as a vendor claim:

1. **Baseline first.** Record, for a defined period before the pilot, the bugs found by existing means on the chosen component — the number, the class, and the mean time-to-diagnosis. Without a baseline, the pilot's output is unfalsifiable.
2. **Fix the invariant set and freeze it.** Write the invariants before the run and record them. The most persuasive possible result is "these N invariants that our own suite already asserts, and which we believed held, are violated under faults we did not write".
3. **Choose the component by §10's fit test**, not by what is easiest to containerise. A trivial component that is easy to containerise will produce a trivial result.
4. **Pre-agree the artifact handover.** Ask for the artifacts, the seed(s) and the reproduction procedure for the first N findings, and **reproduce them yourself**. If a finding cannot be reproduced by the customer from the customer's own environment, the central claim of the product is untested.
5. **Price the obligations, not the platform.** Budget the engineering days for steps 1 and 2 and for triage of the findings. Those are the costs that make or break the business case, and they are the ones the vendor's four-step framing does not quantify.
6. **Ask the §15 questions in writing** before signature: registry/entity, security model, data handling, exit and artifact ownership, roadmap commitment, and what happens if the vendor is acquired or fails.

---

## 6. The Evidence

This section is the guide's core evidentiary work. The repository's standing rule applies and is applied strictly: **a named party announcing the work is evidence; a vendor's customer-logo page is not.** Every entry below is graded ✅ (verified at the customer's own source), ⚠ (the vendor's claim), or ❌ (not located), with dates.

### 6.1 The Evidence Table

| Claimed party | What is claimed | Where the claim is published | Grade |
|---|---|---|---|
| **MongoDB** | Uses Antithesis to test "MongoDB's Storage Engine, Core Server, Cluster-to-Cluster Syncing Utility, and upgrade/downgrade compatibility"; "Antithesis is then integrated into MongoDB at the CI level, to be run as part of their release process"; "Since 2021, MongoDB has used Antithesis" | **Two independent customer-side sources**: (a) **MongoDB's own repo wiki**, "Testing MongoDB with Antithesis", last edited 18 February 2022, which states "Antithesis is a third party vendor with an environment that can perform superior network fuzzing compared to existing `resmoke` infrastructure", describes the workload container and the exact command, lists the fault settings, and reports that the platform "has surfaced novel failures that were not detected using the `ContinuousStepdown` hook"; (b) a **guest post by Dan Gottlieb, "Former Senior Staff Engineer, MongoDB"**, published on the vendor's site on 22 April 2024, describing a specific data-corruption investigation in the WiredTiger storage engine and a JIRA ticket (WT-9500) | ✅ **verified at the customer's own source** for the relationship and for the fault configuration (wiki, February 2022). ⚠ The guest post is on the vendor's domain — the *person* is a named former MongoDB engineer with a named ticket, which is materially stronger than a logo, but it is not a MongoDB-published document. |
| **Ethereum Foundation** | Used Antithesis to simulate the global network under extreme conditions ahead of The Merge, finding bugs that "could have jeopardized the historic Proof-of-Stake transition"; the December 2025 release says "Ethereum used Antithesis" | The **December 2025 vendor release**, and a **vendor case study dated 13 February 2024**, which states the EF "began working with Antithesis to test the Merge codebase under very demanding conditions" **about a year before the Merge** (i.e. from ~2021), that Antithesis "ran Ethereum's entire network (including the proposed Proof of Stake code) inside of the Antithesis environment", found "thirteen bugs within the category of 'Panics, Crashes, or Denials of Service'" and "twenty-two bugs that affected the overall performance of the Ethereum blockchain", the bugs spanning "all six programming languages used to implement the eight clients being tested", and quotes **Danny Ryan (Researcher)** and **Paritosh Jayanthi (DevOps Engineer)** in support | ⚠ **Vendor claim**, with named EF individuals quoted **on the vendor's own page**. ❌ **No Ethereum Foundation blog post, EF-published case study or EF talk confirming the engagement was located in this pass** — searches for an EF-side write-up returned empty result sets (recorded as a tool limitation, §15, not as evidence of absence). The bug counts (13 panics/crashes/DoS, 22 chain-operation bugs) are **vendor-authored numbers with a vendor date**; the vendor does link to several upstream pull requests (e.g. `sigp/lighthouse#3183`, `prysmaticlabs/prysm#10994`) as evidence, which any evaluator should open and verify independently. Do not convert "the Merge succeeded" into evidence that the platform found bugs — the vendor itself presents the causal link, and it cannot be separated from the ordinary testing the EF also ran. |
| **Jane Street** | "depends on Antithesis to validate complex distributed systems critical to the operation of its global trading business"; testimonial from Doug Patti, Software Engineer, Jane Street | **The vendor's product page** and **the vendor's own funding release**, which also discloses Jane Street as the **lead investor**. SE Radio episode 685 (September 2025) is a **Will Wilson** interview on the vendor's side of the story | ⚠ **Vendor claim + investor dual role** (§2.3). ❌ **No Jane Street–published engineering write-up on Antithesis was located.** The testimonial is quoted in a document whose issuer is the vendor. This is the weakest of the "big three" for evidentiary purposes despite being the most prominent, and the dual role should be disclosed in any internal evaluation paper that cites it. |
| **Turso** | Deterministic Simulation Testing "at the core" of its rewrite, "plus a partnership with Antithesis (a deterministic hypervisor) to find bugs, 'including bugs in the simulator itself'" | **turso.tech/blog, January 2025**, as recorded in this repository's `turso_database_guide.md` §4.2 (and repeated at §near-641), citing the customer's own blog. Corroborated at search level by two Turso posts: "We will rewrite SQLite. And we are going all-in" ("Through our partnership with Antithesis, we're using their Deterministic hypervisor to catch bugs that even our simulator might miss — including bugs in the simulator itself") and "Introducing Limbo: A complete rewrite of SQLite in Rust" ("we are partnering with Antithesis, a company that provides a system-level Deterministic Simulation Testing framework, and can simulate all sorts of hardware and software failures") | ✅ **verified at the customer's own source** (`turso_database_guide.md` is the repository's record; the Turso posts are the primary). This is the cleanest customer-side evidence in the file, because the customer published it on its own blog, unprompted by a funding announcement, and describes a *two-layer* arrangement (their own in-house simulator plus the vendor) that reads like engineering judgement rather than marketing. |
| **Tigris Data (Adil Ansari, Founding Engineer)** | "Antithesis helped us find three key bugs in Tigris… These are the bugs that CI can't cover because no one thinks to write tests for them" | Testimonial on the **vendor's product page**; plus a **vendor report, "Antithesis report: Tigris Data", dated 21 April 2026** | ⚠ **Vendor claim.** A vendor-authored "report" is a marketing artefact unless the customer countersigns it; ❌ no Tigris-published write-up located. |
| **Formance (Clément Salaün, CTO)** | "What would otherwise take six months of testing in production, running the software — we can get that in an hour." | **Vendor product page** testimonial | ⚠ **Vendor claim.** The quote is an order-of-magnitude comparison with no baseline, workload, or measurement method. It is the single most aggressive multiple on the vendor's site (a ~4,300× speed claim) and it should never be repeated as a fact (§14). |
| **ParadeDB (Philippe Noël, CEO)** | "I basically dropped the Antithesis logs into Claude and had it debug + fix it end-to-end"; a "6hrs in Antithesis covers more cases than 100 engineers" stat | **Vendor product page** testimonials | ⚠ **Vendor claim**, and notable for what it actually demonstrates: a *workflow* anecdote about log quality, plus a headline ratio ("6hrs" vs "100 engineers") that is not a measurement. |
| **PingThings (Sean Murphy, CEO)** | "There's been a transformation in the engineering group"; "1 day — Compressed 3 weeks of debugging into one day" | **Vendor product page** testimonials | ⚠ **Vendor claim.** "3 weeks" and "1 day" are customer-stated, vendor-published, with no method. |
| **Mysten Labs / Sui (Mark Logan, Tech Lead for Sui Core)** | "With Antithesis, you can make these pretty bold changes… and you can just kind of write the thing." | **Vendor product page** | ⚠ **Vendor claim**; ❌ no Mysten-published write-up located. |
| **Marek Siarkowicz, Kubernetes & etcd maintainer** | "Antithesis's deterministic simulation and active exploration removes the blindfold, enabling a systematic and reproducible search for bugs." | **Vendor product page** | ⚠ **Vendor-published testimonial from an individual**, not an organisation-level endorsement. It is attributed to his maintainer role, but it is not an etcd/Kubernetes project statement, and no CNCF-side document was located. |
| **Readyset** | "Catching a caching bug at Readyset" — a vendor customer story dated 19 February 2026 | **Vendor blog** | ⚠ **Vendor-authored customer story**; ❌ no Readyset-side publication located in this pass. |
| **"100+ critical bugs, 75% of which had evaded internal tests, 47% MTTR reduction, ~10x ROI"** | A strikingly specific MongoDB outcome set | A **third-party case-study aggregator page** discovered via search, not a MongoDB or Antithesis publication | ❌ **Rejected as evidence.** Numbers of this specificity appear nowhere in MongoDB's own wiki page or in the vendor's own MongoDB guest post, and no methodology, period or denominator is published. This guide records that such a page exists and declines to use its figures. If a buyer is shown these numbers in a sales cycle, the question is: *who measured that, over what period, and can I see the report?* |

### 6.2 Summary of the Evidence Position

- ✅ **One customer relationship is verified at the customer's own sources with technical specificity: MongoDB** — a customer-authored wiki page with fault parameters and an exact workload invocation (February 2022), plus a named former MongoDB engineer's first-person investigation account (April 2024, hosted by the vendor).
- ✅ **One customer is verified at its own published blog with a clear statement of use: Turso** (January 2025, via `turso_database_guide.md`).
- ⚠ **The three most prominent names — Jane Street, Ethereum, MongoDB — appear in a vendor-issued funding release**; of those, MongoDB is independently supported, Ethereum is vendor-only in this pass, and Jane Street is both vendor-only **and** the lead investor.
- ⚠ **Every quantitative outcome claim located is vendor-authored**, with one exception (the MongoDB wiki's qualitative "surfaced novel failures that were not detected using the ContinuousStepdown hook" and "bugs that have historically been difficult to reproduce have been reproduced consistently in Antithesis" ✅ 2022). No bug count, speed multiple, revenue multiple or ROI figure in this guide comes from an independent source, and §14 grades each accordingly.
- ❌ **No bank, and no financial institution of any kind, is named as a customer or user** anywhere in this guide. The vendor's own FAQ refers to "many of our capital markets customers" without names; that is a vendor claim about an unnamed population and is treated as such (§4.2). §11 discusses banking *use cases* as classes of system, never as customers.

### 6.3 The Customer's Own Account, Quoted — Because It Is the Best Evidence Available

The MongoDB investigation is worth reading in the customer's own framing, because it is more informative about what the platform does than any vendor page. From the guest post by Dan Gottlieb (22 April 2024), a former MongoDB Senior Staff Engineer:

- **The failure class**: an index inconsistency — "An index entry existed, but the document it referred to was missing" — on the `_id` index, in the `config.transaction` collection, "soon after a replication rollback", with data files that provided "no clues about the missing data. This bug did a good job hiding its tracks." ✅
- **What the platform contributed**: "Antithesis came prepared. They had rerun their simulations, and had not only reproduced the bug, but found the exact time at which it had occurred." ✅
- **The mechanism, described by the customer**: "It runs a simulation for some amount of time and takes a 'checkpoint'. It will then add entropy and explore multiple paths rooted at that checkpoint. If one path keeps leading to the same error state, it can record that as a problem and spend resources exploring other paths instead." ✅ — the checkpoint/branch model, in the customer's own words.
- **The limit of the analysis**: "This was useful but unfortunately, it still wasn't good enough… I still didn't know why the bug was occurring, even if we now were sure of when and where." ✅
- **The intervention that actually solved it**: a request for core dumps "for every 100 milliseconds within a two second range around the bug", which initially "had to be done by hand (and we were heading into the weekend)". The core dumps "were gold"; the final narrowing was to a window "no wider than 10ms". ✅
- **A constraint no marketing page mentions**: because Antithesis reproduces failures by reproducing "the exact (simulated) timing of every instruction the CPU processes… **Adding or removing instructions would have a butterfly effect on the outcome. Therefore, instrumenting the binary with diagnostics was off the table.**" ✅

That last point is the most valuable sentence in the entire public record on this product, and it is a *limitation*, published by the customer: **you cannot add logging to the binary you are debugging inside the deterministic environment, because that changes the run.** The workaround is artifact extraction (core dumps, data files) from a pre-built binary, and — as the customer describes — at the time of that engagement the extraction ran "by hand". A buyer should ask how that has changed; the vendor's current documentation describes artifact extraction as a platform feature, but this guide did not verify the workflow's current level of automation.

---

## 7. The Commercial Model

### 7.1 Licensing — Established, Including the Negative

| Question | Finding | Grade |
|---|---|---|
| Is there a free or open-source version? | **No.** The vendor's FAQ answers the question "Is there a free or open-source version of Antithesis?" with: **"Sadly, no. Antithesis is compute-intensive by nature, and the infrastructure has to get paid for somehow."** | ✅ **verified at the vendor's own FAQ** (retrieved September 2026), and recorded in `deterministic_engineering_guide.md` §5.3. **This is a verified negative fact and one of the few hard facts in the commercial model.** |
| Is there a published price? | **No.** See §7.5 — the absence is the finding. | ✅ absence verified across the product page, the FAQ, the docs and the funding release. |
| Is there a free tier, trial or self-serve signup? | **No self-serve path found.** The documentation's path to getting started is: "Contact us to request a container registry and credentials." The product page's calls to action are "Book a demo" and "Test with us". | ⚠ vendor docs, September 2026. |
| Is there any free-of-charge route at all? | ⚠ **One, historically: the "Antithesis Open Source Giveaway Program"** (announced 28 February 2024 by Will Wilson). It was **not a free tier** — it was a **nomination-and-poll process to donate work to one volunteer open-source project at a time**, with explicit rules: "We're reluctant to do a ton of free work for a project that has well-funded institutional sponsors who could be paying us. Please submit things that are **actual** volunteer/charitable efforts"; the program explicitly excluded parsers and HTML renderers ("you can already test those with Google's OSSFuzz… Let's focus instead on the other 99% of software that conventional fuzzers can't handle"). The post also contains the closest thing to price signalling the company has published: "we knew our pricing would be tough for small teams — it was always part of the Master Plan that we would start out with an offering that's great for enterprises, and gradually work our way down to something that's great for startups and even for individual developers." | ⚠ **vendor blog, 28 February 2024.** Useful as *evidence that a price exists and is enterprise-grade*, and as evidence of the company's own view that its pricing excludes small teams. **It is not a price list and no figure is derivable from it.** Whether the Giveaway Program is still running in 2026 was **not verified** (§15). |
| What is the licensing *form*? | ❌ **Not established.** No licence agreement, no term sheet, no unit of sale (per-run? per-seat? per-core-hour? subscription?) is published. The AWS Marketplace answer below implies at least a consumption or subscription construct, but this guide could not determine the unit. | ❌ not located. |

### 7.2 The Adjacent Open-Source Tools — and What They Are Not

The vendor's FAQ, in declining to offer a free version, points at three open-source things. It is important to be precise about what they are, because a reader may reasonably hope one of them is "Antithesis, free":

| Tool | What it is | What it is not |
|---|---|---|
| **Hegel** (hegel.dev) | A family of **property-based testing libraries** from the same team. The vendor blogged about it on 24 March 2026 ("Hypothesis, Antithesis, synthesis"). | Not a simulator, not a hypervisor, no fault injection, no whole-system environment. PBT is a *component* of the technique (`deterministic_engineering_guide.md` §11.2). |
| **Bombadil** (bombadil.bot) | A **property-based testing tool for web and terminal UIs**. | Not a system simulator. UI-level PBT, genuinely useful, categorically different. |
| **dhyve** (`github.com/pgraug/dhyve-src`) | **An open-source deterministic hypervisor built on bhyve** — verified at its own README in this pass: it "leverages Intel VMX to create a deterministic execution environment for existing software, without modifications", replaces "wall-clock time… by a virtual time model derived from a deterministic count of executed branch instructions", intercepts "hardware randomness, timer interrupts, and I/O", and ships with a "snapshot/branch/mutation harness for state-space exploration" called **Director**. **BSD-2-Clause**, 61 stars, 14 commits at retrieval (September 2026). Authored by **Peter Graugaard and Nicholas Kristiansen as a bachelor's project at the Technical University of Denmark**. | Not a product: no hosted environment, no platform, no SDK ecosystem, no support. The `deterministic_engineering_guide.md` §5.3 table records the vendor's description of dhyve as "inspired by Antithesis" and flags it as *"reported; not independently verified here"*. **This pass verified that description at dhyve's own README**, which says: "We were also very inspired by the people at Antithesis (especially Will Wilson and Alex Pshenichkin). Their blog posts, talks and podcast episodes about their closed-source deterministic hypervisor helped us a lot in designing our architecture and avoiding pitfalls." ✅ **So the "inspired by" claim now stands verified at the source — as a claim about inspiration, which is all it ever was.** Note also that the authors describe Antithesis's hypervisor as **closed-source**, which is consistent with §7.1. |

**The commercial reading of that table:** the vendor's open-source gifts are *upstream* of its product (PBT libraries you can use without a simulator, and a student-grade reference implementation of the environment). None of them is a substitute for the platform, and the one thing that would be a substitute — the deterministic environment itself — remains closed and commercial. That is a coherent business posture, not a bait-and-switch; but it does mean the standard open-source escape hatch is unavailable, and free-and-open-source projects should not expect to adopt this tool without either the Giveaway Program or a sponsor.

### 7.3 Deployment Options

| Option | Detail | Grade |
|---|---|---|
| **Vendor-hosted (SaaS)** | The default: the vendor's control plane and execution environments; the customer deploys a containerised system into it. | ⚠ vendor docs |
| **Run in your own AWS VPC** | The vendor's FAQ: "You can run Antithesis in your own AWS VPC, keeping proprietary binaries and source code fully under your control. In this architecture, **Antithesis's ephemeral execution environments and container image registry run exclusively in your VPC, while the control plane remains in our VPC** with customer-isolated, fully single-tenant compute and databases." | ⚠ **vendor FAQ**, retrieved September 2026. This is the most important paragraph in §7 for a regulated buyer (§11), and note its precise shape: **execution and images in your VPC; control plane, and therefore the orchestration metadata about your runs, in the vendor's VPC.** |
| **Fully on-premise** | **"We do not offer a fully on-premise option at this time."** | ✅ **verified negative fact** (vendor FAQ). |
| **AWS Marketplace** | "Yes! We often transact via **AWS Marketplace Private Offers**, so purchasing Antithesis usually draws down your organization's committed spend. As always, the details depend on the terms of your particular AWS Private Pricing Agreement." | ⚠ vendor FAQ. Commercially significant: it means the purchase may be an **existing cloud-commit drawdown** rather than a new line item — which is a real procurement advantage, and simultaneously a reason the price is invisible (it lands under an existing agreement). |

### 7.4 Support and Services

The published record on support is thin: the vendor offers "book a demo", publishes documentation and SDKs, offers agent skills for onboarding, and the MongoDB guest post attests to a high-touch relationship in the customer's own words — Antithesis "came to the MongoDB office and talked about their technology", "helped onboard more of our tests into their system", and the customer's summary is that "the other (arguably larger) half of what makes Antithesis great is the company's commitment to their customers' success". ⚠ (customer-authored, vendor-hosted, April 2024.) Beyond that: ❌ **no support tiers, no SLAs, no response-time commitments, and no professional-services catalogue are published** (§15). For a platform whose value depends on onboarding quality, "the vendor will work closely with you" is a real signal and a weak contract.

### 7.5 THE PRICING FINDING

> **The finding is the absence.** No price is published anywhere this guide examined — not on the product page, not in the FAQ, not in the documentation, not in the funding release, not in any marketing or comparison page reachable in this pass. **Accordingly, this guide prints no number, offers no band, and makes no estimate.** Any figure a buyer encounters is either inside a private negotiation or invented.

What *can* be said about price without asserting one:

1. ✅ **A price exists and is charged per engagement or subscription** — the FAQ's own explanation for having no free version is that "Antithesis is compute-intensive by nature, and the infrastructure has to get paid for somehow", and the AWS Marketplace answer describes transacting via Private Offers.
2. ⚠ **The company's own view is that the price is enterprise-grade and excludes small teams** — from the February 2024 Giveaway post: "we knew our pricing would be tough for small teams". That is the company's own characterisation, and it is the closest thing to price disclosure published.
3. ⚠ **The purchase may be structured as a cloud-commit drawdown** via AWS Marketplace Private Offers, which changes the *procurement* question (does it consume existing committed spend?) and the *visibility* question (it may not appear as a new vendor line item).
4. ⚠ **Compute-intensity is the cost driver** — the product runs many single-core VMs in parallel and "massively parallelize[s] runs". Any pricing model in this category must recover that compute, which means the honest expectation for a large workload is a **usage-shaped** cost, not a flat licence. This is inference from the vendor's own architecture, labelled as inference.

**Consequence for an evaluation:** because no price is public, a deterministic-simulation purchase **cannot be compared to alternatives by desk research**. The comparison in §12 is therefore built on illustrative figures with an explicit label, and the real comparison requires the vendor's quote.

### 7.6 What a Buyer Cannot Know From Outside

Four unknowns, in order of how much they should worry a procurement or risk function:

1. **The commercial terms.** ❌ Unit of sale, minimum commitment, term, renewal mechanics, overage treatment, indexation. For a compute-shaped product, the *variance* in cost is the risk, and nothing published addresses it.
2. **The roadmap commitment.** ❌ Nothing contractually published says a given SDK, language, deployment mode or fault type will continue to exist. The record does show active investment — an active Java SDK at v1.7.0 (2026-09-02), a steady blog cadence through 2026, causality analysis shipped as a feature in May 2026, agent skills in 2026 — but **activity is not a commitment**, and a Series-A company's roadmap is its board's business, not its customers'.
3. **The artifact-lock-in question.** ❌ **Unanswered in the public record, and the most under-asked question in this category.** The platform's differentiating output is *artifacts about your system*: run histories, the multiverse, traces, bug reports, causality analyses, extracted core dumps and data files, and the reproduction procedure. Three specific questions a buyer must ask: **(a) In what format are run artifacts exported, and can they be extracted in bulk without the platform?** **(b) Does a reproduction require the platform to run, i.e. is the seed + binary sufficient to rerun a finding independently?** From the customer's own account in §6.3, reproducing a failure required Antithesis to rerun simulations and hand over core dumps — which suggests reproduction is a *platform* capability, not a customer-side one. **(c) What happens to run history, artifacts and the ability to reproduce a previously-found bug on termination — and is there an escrow or export right?** An organisation that lets this platform become the system of record for *why its own bugs happened*, with no export path and no exit clause, has created a novel and avoidable concentration risk. This guide found **no vendor statement on any of (a), (b) or (c)**.
4. **The security and data-handling model.** ❌ No published security documentation, certification list (SOC 2, ISO 27001), penetration-test summary, sub-processor list or data-residency schedule was located (§15). For the VPC option the *shape* is disclosed (execution in your VPC, control plane in the vendor's), which is genuinely better than nothing, but the controls are not.

---

## 8. Competitive Landscape and Positioning

### 8.1 The Honest Map

The technique itself is deferred to `deterministic_engineering_guide.md` §5 (and its §5.3 implementations table, which this section cross-references rather than repeats). What follows is the *market* view: who else can find this class of bug, and what each alternative costs the buyer.

| Alternative | What it is | Coupling / constraint | When it beats Antithesis | When Antithesis beats it |
|---|---|---|---|---|
| **Build your own DST harness (FoundationDB-style)** | An in-tree simulator that owns the system's I/O boundary, clock, network and storage, driven by a seeded fault schedule. `deterministic_engineering_guide.md` §5.2–§5.5 | **The technique imposes an architectural constraint on the system** — one narrow I/O boundary, deterministic core code, injectable time/randomness/storage, seeded fault injection, statable invariants. FoundationDB's is "enabled by and tightly integrated with Flow", the company's own actor language. You must own the runtime | You are already writing the runtime, or the system is new enough that the boundary can be designed in. Then the simulator is far more faithful to *your* system than a hypervisor can be: it can model your specific storage engine, your specific protocol, your own failure modes | The system exists, is large, is in production, and cannot be rewritten around a single I/O boundary. Antithesis's promise is precisely that the customer does **not** have to own the runtime — it containerises what it has |
| **FoundationDB's own in-tree simulator** | First-party, in-tree, coupled to the Flow language; still active (docs at release 7.4.7, published 2026-09-01) | Not licensable as a product; coupled to a language | Never, for an outside buyer — it is not for sale. It exists as *evidence that the technique works at scale* (the project's own claim: ~10:1 real-to-simulated time, ~one trillion CPU-hours of simulation, tens of thousands of simulations nightly — **the project's own estimate, not independently measured**) | Always, for a buyer: it is not an option |
| **TigerBeetle's VOPR** | First-party, in-tree, coupled to that project (Zig, a stubbed-side-effects discipline); the project's name for its deterministic simulator | Not licensable; coupled to TigerBeetle's own codebase and language choices | Never for an outside buyer | Always, for a buyer — but note what it proves: a *small, disciplined team that wrote its own database* could build the environment itself. That is the counter-example to Antithesis's necessity claim (§8.3) |
| **dhyve** (+ Director) | Open-source deterministic hypervisor on bhyve, DTU bachelor's project, BSD-2-Clause, 61 stars, 14 commits; verified at its own README (§7.2) | Not a product: no hosted environment, no SDKs, no support, no platform. Very low maturity | If the goal is to *learn the technique* or to prototype an environment, and the team has hypervisor-grade engineers | Almost always, for an enterprise: there is no operational offering behind it |
| **Chaos-engineering platforms** (the category, whatever the vendor) | Fault injection into *real, running* environments — real process kills, real partitions, real latency. Category owned by `chaos_engineering_guide.md` | Not deterministic; a found interleaving-dependent failure is generally not reproducible on demand | Finding integration, configuration, dependency and infrastructure bugs that no one modelled in a simulator; validating that the *real* environment behaves as the fault model claims; building operational muscle (game days) | Finding rare interleaving bugs at all, and reproducing whatever it finds. Also: no blast radius, no production risk, no change-management approval to run it |
| **Jepsen-style black-box verification** | Adversary-driven testing of *real deployed systems* with real faults, searching for violations of a stated consistency property (§1.3; technique cross-referenced to `deterministic_engineering_guide.md` §11) | Requires a real deployed cluster and a stated property; produces analyses, not seeds | It tests the system as deployed, at the infrastructure level the simulator abstracts away — the ground-truth check on the simulator's fidelity | Reproducibility by seed, instruction-level root cause, and bug-finding rate on interleavings that black-box testing will not hit in a human lifetime |
| **Generic fuzzing** (AFL/libFuzzer/OSSFuzz-class) | Input-space fuzzing of a parser or single process | Single-process input space; no whole-system environment, no fault injection, no cross-service interleavings | Cheap, mature, excellent for parsers, decoders, memory-safety bugs. **The vendor itself concedes this**: its Giveaway post says "we would love to test your parser or HTML rendering engine, but you can already test those with Google's OSSFuzz. Let's focus instead on the other 99% of software that conventional fuzzers can't handle" | Whole-system, stateful, concurrent software — the class the vendor calls "the other 99%" |
| **Formal methods / model checking** (TLA+, Coq, model checkers) | Proof or exhaustive checking over a *model* | Proves properties of the model, not of the implementation; needs a specification and expertise | When a specification exists and the property is worth a proof — and note the vendor's own post, "Finding bugs in Raft implementations: why formal verification is not enough to maintain consensus in real systems" (27 July 2026), is an argument for running real code, which cuts both ways: it is an argument against formal-methods-only *and* a reminder that nobody proves anything about the shipped binary | When there is no specification, or when the model-to-code gap is the problem (§10's "a problem a specification fix would solve") |
| **AI code review / static analysis** | Inspection of source, by rules or by model | Operates on the code, not on the running system | Cheap; catches classes of bug that running never reveals (style, API misuse, some security patterns) | Everything that only exists at runtime. The vendor's FAQ puts it sharply: AI review "operates on the same information that agents used to write the code in the first place… Antithesis actually runs your whole distributed system" ⚠ |

### 8.2 The Positioning Question

**Which buyer has a problem only this category solves?** The buyer whose system is (a) concurrent, stateful and distributed; (b) already containerised or containerisable; (c) has invariants that are *already known and written down somewhere* — in a data model, a protocol specification, a ledger's double-entry rule, a replication guarantee; and (d) suffers from failures whose diagnosis, not whose existence, is the bottleneck. That buyer is bought-in on the technique before the conversation starts, and the vendor's pitch is nothing more than "you do not have to build the environment". MongoDB's public record is that buyer exactly: a storage engine, an internal test framework (`resmoke`), an existing fault-injection hook it wanted to do better, and a data-corruption class whose root cause took weeks (§6.3).

**Which buyer is being sold a category it does not need?** Several, and the vendor's own positioning language invites them:

- **The buyer with no invariants and no property tests.** If the team has never written "must always / must never" assertions, the platform's output is a green dashboard. The four-step workflow starts here ("Outline your invariants") but markets the product as if the invariants were the easy part.
- **The buyer whose failures are not reproducibility-limited.** If the incidents are configuration drift, dependency upgrades, capacity, operator error and bad deploys, a deterministic simulator addresses almost none of them. The appropriate purchase is chaos engineering, better staging, and operational discipline — different category, different budget line.
- **The buyer whose problem a specification fix would solve.** If the bug class is "we never wrote down what the consistency guarantee actually is", the fix costs one engineer and a document, and a simulator would only discover the ambiguity under load.
- **The buyer shopping for "AI-era verification" as a category.** The vendor's 2026 marketing leans hard into agentic coding ("Verify every Pull Request", "even when they're authored by agents", "our agent only declares victory once our platform has verified its output"). That is a genuine strategic bet and a genuine use case; it is also a *different* purchase from "we have a consensus bug we cannot reproduce", and buyers should not let the agent narrative decide the technical question.
- **The buyer reflexively comparing it to chaos engineering.** Recounted in §1.3; the categories do different jobs and the mature posture — `deterministic_engineering_guide.md` §5.7 — is **both, in this order**: simulation to search and reproduce; real-environment testing to keep the simulator honest.

### 8.3 The Counter-Example Worth Stating in the Vendor's Own Market

One honest note on the necessity claim. The thesis of this guide is that almost nobody can build the environment — and TigerBeetle is the standing exception: a project that wrote its own database and its own simulator (the VOPR), publishes the fact that a *seed plus a commit* reproduces any bug, claims arbitrary time compression ("one minute of VOPR time is much longer than one minute of real time"), and whose own documentation states that the approach "was heavily inspired by the work of FoundationDB and Antithesis". A buyer evaluating Antithesis should therefore ask the necessary question: *is my system a "write the runtime" system or a "containerise what I have" system?* TigerBeetle is the first; most banks and most product companies are the second. The vendor's business exists because the second population is enormous — and the vendor's own hypervisor engineering post is the most persuasive evidence that the environment is hard: it describes PMC counters miscounting "about one in a trillion instructions", a need to write a custom kernel logging subsystem to capture "50 GiB of output per 20-minute run", and work that was "pushing various features of the CPU just a bit past what they were really designed for". ⚠ (vendor's own account, 20 March 2024.) If that is what it takes, the "buy" answer is much stronger than the "build" answer for anyone not writing a storage engine as their product.

### 8.4 What the Landscape Does Not Contain

For completeness, and to prevent a reader hunting for something that does not exist: ❌ this guide located **no other commercial vendor selling a general-purpose deterministic simulation platform** as a product. The implementations table in `deterministic_engineering_guide.md` §5.3 lists FoundationDB's simulator, TigerBeetle's VOPR, Antithesis, Shadow, and dhyve — of which exactly one is sold. Whether that is a moat or a market that has not yet been proven to exist is the strategic question the Series A answered with capital rather than with evidence, and it is the question a buyer's own evaluation will not answer either. What a buyer *can* observe is that the alternatives to buying are: build in-house (rare), adopt the technique in a narrower in-tree form (only if you own the runtime), or decline the technique entirely and rely on chaos engineering plus black-box verification (the common answer, and frequently the right one).

---

## 9. The Technique in Brief

This section exists to stop a reader mistaking this file for the technique guide. It is deliberately short.

**Deterministic simulation testing, in five lines.** You run your **real production code** against a **simulated environment** rather than against mocks: the simulator supplies time, network, disk, scheduling and randomness, all derived from a **seed**. Because every input descends from the seed, a run is reproducible, and because the simulator owns time, simulated time can advance faster than real time. Faults are injected as first-class, seed-derived parameters, so the fault schedule is itself replayable. What you get that no other technique gives you is **rare interleavings, found deliberately, and reproducible on demand** — and what you pay for it is a permanent maintenance obligation on the simulated model. **It is not mocking**: mocking replaces the system under test, while simulation keeps the system and replaces its environment.

**Now read these, by name:**

- **The technique, its definition, its canon and its costs** → **`deterministic_engineering_guide.md` §5, "Toolkit II: Deterministic Simulation Testing"**. That section owns: the definition (§5.1); the "not mocking" distinction (§5.1); the **Will Wilson Strange Loop 2014 talk as the canonical articulation** and the FoundationDB documentation as the primary written source (§5.2); the **implementations table** — FoundationDB, VOPR, Antithesis, Shadow, dhyve, batch-invariant kernels — with statuses checked 17 September 2026 (§5.3); what DST uniquely buys (§5.4); **the five architectural requirements it imposes on the system under test** (§5.5: a single narrow I/O boundary; deterministic core code; injectable time, randomness and storage; fault injection as a first-class seeded parameter; invariants you can state); the costs, including that the simulator itself becomes a system to trust (§5.6); and the trade-off table against real-environment testing (§5.7).
- **The construction and the practice** → **`deterministic_simulation_testing_guide.md`** (sibling guide in this directory). That guide owns how a team builds and operates deterministic simulation: harness construction, the stub boundary, seed handling, workload design, fault models, and the organisational cost. If your question is "how would *we* do this?", that is the file.
- **The category it is confused with** → **`chaos_engineering_guide.md`**. Chaos engineering injects real faults into real environments; it is a different discipline with a different failure-tolerance objective, and the vendor's "extreme version of chaos engineering" framing (§1.3) is a marketing bridge, not a taxonomy.
- **The other kind of rigorous testing** → **`deterministic_engineering_guide.md` §11** for the property-based and model-based testing families and the "a failing test must print the reproduction handle" discipline; Jepsen-style black-box verification of real deployed systems is the other rigorous posture named in this repository (§1.3, §8).
- **The determinism taxonomy that explains *why* any of this is necessary** → `deterministic_engineering_guide.md` §3, which enumerates the sources of nondeterminism (clock, RNG, concurrency order, unordered collections, float reduction, ambient state, external dependencies) that a simulated environment has to own.

Everything in this file from §1.4 onward that touches the technique does so in **one clause or less**, by design. The remainder is the company and the product.

---

## 10. Who This Is For and Who It Is Not For

### 10.1 The Fit Test

A deterministic simulation platform is worth its cost when **all five** of the following are true. This is not a vendor's list; it is the list this repository's technique guide implies (§5.5) and this guide's evidence supports (§6).

| # | Condition | Why it is a hard gate |
|---|---|---|
| 1 | **The system's correctness is a first-class business property** — a bug means lost money, lost data, a corrupted ledger, or an outage, not a slow page | The cost is substantial and the value is proportional to the cost of being wrong |
| 2 | **The system is concurrent, stateful and distributed** — it has interleavings, replication, recovery paths, leader election, retries | These are the failure modes the technique is uniquely good at, because they are the modes production rarely exercises and never reproduces |
| 3 | **The system can be launched and driven in containers** | The platform boots your topology (§5.2); this is a hard precondition, not a preference |
| 4 | **The team has a harness** — an end-to-end or integration suite that can exercise the system, or the will to build one | The platform drives your workload; it does not invent one |
| 5 | **The team has invariants worth asserting** — properties it *already believes* and can write down | The platform searches for deviations from promises (§4.5). No promises, no findings |

### 10.2 Where the Answer Is Yes — By Class of System

Unsentimental, and expressed as classes rather than named customers:

- **Consensus and replication layers.** Raft/Paxos/Multi-Paxos implementations, replicated logs, membership changes, snapshotting, catch-up. This is the single best-fit class: the failure modes are interleaving-dependent, rare in production, and catastrophic. The vendor's own blog post of 27 July 2026 ("Finding bugs in Raft implementations") targets exactly this, and it is a genuine market signal that the class is well served.
- **Storage engines and database internals.** Write-ahead logs, MVCC, compaction, index consistency, crash recovery, backup/restore, replication rollback. MongoDB's public record (§6.3) is this class, and the specific bug it describes — an index entry pointing at a missing document after a replication rollback — is exactly the kind of defect that no amount of code review finds.
- **Ledgers and settlement engines.** Double-entry invariants, idempotency, exactly-once effects, end-of-day state, reconciliation. The invariant is *already written down by the business* — this is why the fit is so good (§11.1).
- **Databases and stateful infrastructure products** generally: the vendor's customer evidence clusters here (MongoDB, Turso, Tigris, Readyset, ParadeDB), and the cluster is not accidental — it is the population that both needs the technique and can containerise its system.
- **Protocol implementations with adversarial edge cases.** The Ethereum work is the vendor's flagship example, and regardless of how one grades the case study (§6.1), a consensus-protocol client suite is a textbook fit.
- **Systems where the bug-finding bottleneck is *diagnosis*, not detection.** A team that already knows it has rare bugs and cannot reproduce them gets the most from the reproduction machinery.

### 10.3 Where the Answer Is No

Also unsentimental:

- **The failures are not reproducibility-limited.** If incidents come from configuration drift, dependency upgrades, capacity, deploy errors, DNS, certificates, cloud quotas or operator mistakes, a deterministic simulator addresses none of it. Buy chaos engineering, better staging and better observability. (Note carefully: this is the *most common* answer in practice, and it is not an insult to the technique — it is a diagnosis of a different problem.)
- **The team has no property tests and no invariant culture.** There is a fixable version of this (write them first; §12) and an unfixable version (the organisation has never expressed correctness as a property, and will not start because a vendor was bought). Buying the platform before the invariants produces a recurring cost with no findings.
- **The problem would be solved by a specification fix.** If the real defect is "we never decided what the consistency guarantee is", write the document. Simulation does not discover ambiguity; it punishes it.
- **The system cannot be containerised.** A mainframe core, an appliance, a hardware-in-the-loop system, a vendor binary that requires its own certified stack — the platform's precondition is unconditional, and where a dependency is not containerisable the requirement degrades to "use a mock", which re-imports the fidelity problem the technique exists to avoid.
- **The correctness risk is in interaction with the real world, not inside the system.** Market data feeds, external counterparties, exchange gateways, third-party APIs, network-level behaviour of real infrastructure — the simulated environment abstracts exactly these. Simulation plus real-environment testing is the answer, in that order (`deterministic_engineering_guide.md` §5.7).
- **The team is one engineer deep on a component nobody else can maintain.** A simulator needs an owner; a found bug needs someone who understands it. Getting value from this platform is an *organisational* capability before it is a technical one.
- **Compliance, not correctness, is the driver.** If the requirement is "demonstrate testing to an auditor", a deterministic-simulation purchase is an expensive way to produce evidence that the auditor may not even accept (§11.2). Buy the evidence the framework asks for.

### 10.4 The Uncomfortable Middle

The most common real answer is neither. It is: **"we have one component that fits perfectly and a portfolio of twenty that do not."** In that case the right shape is a **single-component engagement with a frozen invariant set and a measured baseline** (§5.5), not a platform-wide adoption — and the decision to expand should be made on the pilot's results rather than on the vendor's case studies, because the vendor's case studies are drawn from the population that already fits (§6).

---

## 11. The Regulated-Enterprise and Banking Angle

### 11.1 Where This Would Genuinely Matter Inside a Bank

Named as **classes of system**, and never as an assertion about any real institution, including any institution this repository has written about: a bank's correctness-critical distributed software looks like this.

| Class of system | The invariant that matters | Why the technique fits |
|---|---|---|
| **Core ledger / general ledger posting engine** | Every posting balances; the sum of debits equals the sum of credits; no posting is lost or duplicated; the trial balance reconciles after any restart | The invariant is *already specified by the business* — double-entry is a property, not an opinion. The failure mode is data corruption under concurrency and crash recovery, which is precisely what is unfindable by CI and unreproducible in production |
| **Settlement and clearing engines** | Idempotency of settlement instructions; exactly-once netting; deterministic end-of-cycle state; correct behaviour on partial failure mid-cycle | Settlement runs are batch-shaped, concurrently processed, and failure-sensitive at specific points in the cycle — a natural interleaving-hunting problem |
| **Replication and consensus inside a payment or ledger store** | No divergence between replicas; no lost acknowledged write; correct leader election; correct recovery after rollback | This is the Raft/replication class (§10.2), and it is the class where the industry's incident history is longest |
| **Payments state machines** | Every transition is legal; no state is reachable that the specification forbids; retries never double-post; reversals always net to zero | State-machine invariants are expressible as properties, and the bugs are usually "a state we didn't know was reachable" — which is the platform's exact pitch |
| **Risk and limit engines under concurrency** | Limits are never breached even transiently; exposure accounting is consistent across positions and aggregates | Concurrency plus shared state plus a hard invariant — a strong fit in principle, though the "never even transiently" property is often what the business has not actually specified |
| **Market connectivity and order management** | Order lifecycle integrity, no orphaned orders, correct handling of partial fills and cancels under adverse timing | A good fit for the *internal* state machine; a poor fit for the real-message-latency parts, which belong to real-environment testing |

Cross-reference for the operational framing rather than re-deriving it here: `../banking/operational_resilience_framework_guide.md` owns important-business-service mapping and third-party dependency; `../banking/enterprise_risk_management_guide.md` owns how a bank structures concentration and control risk; `../banking/mas_regulations_guidelines_guide.md` owns the Singapore regulatory context for outsourcing, technology risk and operational resilience that any contract of this type must satisfy. This guide does not restate their content.

### 11.2 The Evidence Question — Would a Model-Risk or Audit Function Accept It?

This is a real and under-discussed obstacle, and it should be resolved **before** the technical evaluation, not after. The output of a third-party simulation platform is a *new kind of evidence*, and a validation function has to decide what it is evidence *of*. What would have to be true for it to be accepted:

1. **The tool is validated, not just used.** A model-risk discipline asks for the *tool's* own validation: what is the simulator's fidelity boundary, how was it tested, what is known not to be modelled (single-VM topology, `deterministic_engineering_guide.md` §5.6's point that a simulator's own faithfulness is the load-bearing assumption), and what is the vendor's process for simulator bugs? ❌ **Antithesis publishes no simulator-validation document, no fidelity statement and no coverage metric.** That is a direct answer to this question and it is currently negative. A buyer should ask for it, and should ask what the vendor does when its *own* environment produces a wrong result.
2. **The result is reproducible by the institution, not only by the vendor.** A validation function will want the same run to be re-runnable and the finding to be visible to its own engineers, with artifacts it can retain. From §6.3, reproduction appears to be a vendor-side capability (Antithesis reran the simulations); from §7.6, artifact export and post-termination access are undocumented. **Both are gaping holes for an audit file**, and both are contractual rather than technical questions.
3. **The invariant set is the institution's.** The one piece of the process an auditor can actually inspect is the property set: it should be version-controlled, reviewed, owned by the bank, and not generated entirely by the vendor's agent skills without human sign-off. The vendor's "agent skills can help" a specification must not become "the vendor's agent wrote the specification and nobody read it".
4. **The negative result has a stated meaning.** "No violation found" means "no violation was found for these invariants under this fault model, in this run" — not "the system is correct". A model-risk write-up that says "the system is correct" on the basis of a green run has mis-stated the evidence, and the vendor cannot be blamed for that.
5. **The methodology is documented before the run, not after.** Baseline, component scope, fault configuration, invariant set, and the definition of a finding — all frozen beforehand. This is what makes it evidence rather than a demonstration.

**Honest assessment:** in most institutions, a third-party simulation platform's output would be treated as **supporting engineering evidence**, not as validation evidence — useful for finding and fixing defects, weak as an assurance artefact, and therefore *not* a substitute for the model validation, testing evidence or control documentation the framework requires. Buying it as an assurance product will disappoint; buying it as a defect-finding product and using its outputs as supplementary evidence is defensible.

### 11.3 The Data and Exposure Question

Running a bank's system inside a third party's environment raises the question every outsourcing review asks, in a form that is unusually sharp here, because the vendor's product requires **running the real system with real configuration and extracting artifacts from inside it** (core dumps, data files, logs, traces).

State the exposure precisely, from what the vendor actually says (§7.3):

- **Hosted:** the customer's container images, the customer's system running, the workload, the fault configuration, and *the artifacts extracted from inside the customer's system* — i.e. potentially data files and core dumps containing production-shaped data — are processed in the vendor's environment, with a control plane in the vendor's VPC.
- **Run-in-your-own-AWS-VPC:** execution environments and the container image registry run **exclusively in the customer's VPC**, keeping "proprietary binaries and source code fully under your control" (vendor FAQ, verbatim), **while the control plane remains in the vendor's VPC with customer-isolated, fully single-tenant compute and databases.** So the code and the images stay; **the orchestration metadata and run telemetry do not**, and this guide found no enumeration of what that metadata contains.
- **On-premise:** **not offered** (§7.3). ✅

Practical implications a bank must resolve, in order: (1) does the workload use production data, synthetic data or anonymised data, and does that decision hold for extracted core dumps and data files as well as for the live run; (2) what is in the control-plane metadata that stays with the vendor, and is it acceptable under the institution's data classification; (3) is there a contractual right to have all artifacts deleted and to receive confirmation, and does that survive termination. For the third-party-risk and data-lineage framing of those questions, this repository's `../banking/operational_resilience_framework_guide.md` and `../banking/enterprise_risk_management_guide.md` own the discipline; **this guide's finding is narrow and negative: the vendor publishes no security documentation, no certification evidence, no sub-processor list and no data-residency schedule (§15), so the answer cannot be assembled from the public record.** It has to be assembled from the vendor, in writing, during procurement.

### 11.4 The Vendor-Risk Question

An early-stage, venture-backed company with a closed-source product, no on-premise option, no published SLA and no published exit terms is a **concentration-risk counterparty** for a bank, and the standard mitigations apply — with one addition specific to this product.

- **Standard mitigations** (owned elsewhere in the repo; named, not derived): an exit plan and data-portability commitment at signature; a step-in or escrow arrangement for anything the bank cannot operate without; an operational-resilience assessment treating the vendor as a critical third party; and a contractual kill-switch for unacceptable changes or an acquisition. See `../banking/operational_resilience_framework_guide.md`, `../management/vendor_management_guide.md` (if present in this repository's `management/` directory) and `../banking/enterprise_risk_management_guide.md`.
- **The addition specific to this product — artefact lock-in (§7.6).** Most SaaS lock-in is about data of record. This one is about **the explanation of your own defects**. If the multiverse, the traces, the bug reports and the reproduction procedure live only in the vendor's platform, then a decision to leave the platform means losing the ability to reproduce bugs the bank *already knows it had* — including the reproduction of a bug whose fix is in flight during a vendor transition. That is a novel and entirely avoidable exposure, and the mitigation is a contractual right to export run artifacts and reproduction data in a usable form, negotiated **before** the first run rather than after the third renewal.
- **The funding facts bear on this**, without proving anything: a $105M Series A in December 2025 led by a customer-investor is a strong financial position and a signal of confidence; it is also a signal that the company's roadmap will follow its investors' and largest customers' interests. For a bank, the honest reading is that vendor risk is *lower than it was in 2024 and not zero*, and that the mitigations should be contractual rather than optimistic.

### 11.5 The Honest Note: The Binding Constraint Is Usually Not the Tool

For a bank weighing this purchase, the finding that should end the conversation — or start it, differently — is this: **the limiting factor is almost never the availability of a deterministic simulation platform. It is whether the team has invariants and a harness at all.**

Antithesis's four-step workflow begins with "outline your invariants" and moves to "send us your system", and both are the customer's obligations (§5.1). In most large financial institutions, the systems that most need this technique are also the systems with the *least* property-test coverage: long-lived estates, partial containerisation, test suites built around example-based integration tests, and inconsistent specification of what "correct" means. In that environment, buying the platform is buying the third and fourth steps of a four-step process while planning to improvise the first two — which is exactly the anti-pattern §13 records as "a purchase that precedes the harness".

The order that works — and it is the recommendation §12 reaches independently for an illustrative bank — is: **write the invariants and build the harness first, using an in-house deterministic test harness or an existing end-to-end suite; then decide whether to buy the environment.** The invariants are useful without the platform. The platform is useless without them. That asymmetry, not the price, is the decision.

---

## 12. The Cymbal Bank Worked Example

> **This section is fictional and explicitly illustrative.** "Cymbal Bank" is this repository's invented persona — a mid-sized bank headquartered in Singapore. Nothing here is a claim about any real institution, and **no real bank, exchange, broker or financial institution is named as a customer or user of Antithesis anywhere in this guide.** Every monetary figure below is **illustrative and invented**, for two stated reasons: (a) **Antithesis publishes no price**, so no real figure exists to use (§7.5), and (b) an illustrative example that fabricated a realistic-looking price would be exactly the kind of invented evidence this repository forbids. Read the numbers as a *model of the decision*, not as a quote.

### 12.1 The Situation

Cymbal Bank's payments platform contains a **real-time settlement state machine** — the illustrative component name is the **Settlement Orchestrator (SO)** — which:

- processes settlement instructions concurrently against a shared ledger and a replicated store;
- holds **double-entry and idempotency invariants that the business has actually written down** in its settlement policy document — every instruction settles exactly once, debits equal credits, a reversal always nets to zero, and the end-of-cycle position reconciles against the ledger;
- has an existing integration suite and a Compose-based local environment;
- has a **known-bad history**: two incidents in eighteen months in which a rare partial-failure state produced a duplicate netting entry and a reconciliation break. Both were found after the fact. Neither was reproducible in the test environment, and in both cases the root cause took a team of three roughly three weeks to establish by log archaeology.

Cymbal's engineering leadership is weighing three options: **buy** a deterministic simulation platform (Antithesis is the only vendor considered, being the only product in the category — §8.4), **build** an in-house deterministic harness for the SO, or **skip** the technique and invest in chaos engineering and better observability.

### 12.2 The Fit Assessment, Against §10

| §10 fit condition | Cymbal's SO | Verdict |
|---|---|---|
| 1. Correctness is a first-class business property | Settlement breaks mean reconciliation failures, regulatory attention and customer impact | ✅ Yes |
| 2. Concurrent, stateful, distributed | Concurrent instruction processing, replication, partial-failure paths | ✅ Yes — the strongest fit |
| 3. Can be launched and driven in containers | Yes, Compose manifests exist for local development | ✅ Yes — a real precondition met |
| 4. A harness exists | Integration suite exists and can exercise a full cycle | ✅ Yes, with caveats (see §12.3) |
| 5. Invariants worth asserting | **Yes, and written by the business**, not by engineers | ✅ Yes — this is the decisive condition |

**Assessment: the fit is strong, and it is strong for a specific reason that most buyers cannot claim** — the invariants already exist in prose, in a settlement policy that a regulator-facing function has already reviewed. The four-condition version of this purchase (an invariant list to formalise, a system to containerise, a harness to extend, and a vendor to buy) becomes a two-condition purchase for Cymbal.

### 12.3 What Cymbal Must Supply Before the Platform Can Find Anything

This is the work that is *not* in the vendor's four steps' pricing, and it is the work that decides the business case:

1. **Formalise the settlement policy's invariants into assertions.** The policy says "exactly once". The harness must say `assert settled_count(instruction_id) == 1` at a defined point. Illustratively **10–25 assertions** covering idempotency, double-entry balance, reversal netting, end-of-cycle reconciliation, and no-illegal-state.
2. **Extend the integration suite into a drivable workload.** The existing suite probably covers happy paths. It needs a workload container that can be invoked by the platform, exercising the settlement cycle continuously under injected faults (MongoDB's public example is the shape to copy — §4.5).
3. **Harden the Compose/K8s manifests into a deployable topology**, including whatever the SO depends on. If a dependency is a vendor binary that cannot be containerised, it must be mocked, and **mocking it weakens the very fidelity the purchase is for** — so this must be resolved, not papered over.
4. **Choose the failure window deliberately.** The two historical incidents happened under partial failure mid-cycle. The workload must be designed to *drive the system into that window repeatedly*, not to run a generic load test.
5. **Build the baseline record.** Bugs found by existing means, and mean time-to-diagnosis, over the preceding quarter. Without this, the pilot result is unfalsifiable (§5.5).

**Realistic pre-work, stated as an illustrative estimate:** one senior engineer plus a domain expert for roughly a quarter, before the first platform run. The domain expert is the constraint, not the engineer — the invariants belong to settlement operations, not to the platform team.

### 12.4 The Cost Comparison — All Figures Illustrative

> ⚠ **Every number in this table is invented for illustration.** Antithesis's real pricing is **not public** (§7.5). If you are using this table for a real decision, replace the "Buy" column with the vendor's actual quote and leave the two self-built columns as estimates. Currency: illustrative SGD.

| Cost line | Buy (vendor platform) | Build (in-house deterministic harness) | Skip (chaos + observability) |
|---|---|---|---|
| Platform / licence | **Illustrative:** `$X` per year, unspecified by the vendor — **the single largest unknown in the decision** | Zero licence; **same two engineers as an ongoing headcount cost** | Chaos tooling licence: modest |
| Pre-work (invariants + workload) | **Illustrative:** ~1 engineer + domain expert × 1 quarter — **required for all three options if the invariants are to exist at all** | Same, plus the harness boundary work | Not required to the same depth; a much smaller invariant exercise |
| The environment itself | Zero build. It is what you are buying | **The hard part:** a deterministic environment means owning the clock, network, storage, scheduler and randomness — architecturally the SO would need a single narrow I/O boundary, which an existing settlement system does **not** have (`deterministic_engineering_guide.md` §5.5). Illustratively **2–4 engineers for 12–24 months**, with a real risk of never finishing | Zero |
| Fault injection and reproduction | Included; seeded and reproducible | **The part that consumes the 12–24 months**: fault scheduling, seed management, replay, time-travel debugging | Faults are real but unreproducible; incidents stay hard to diagnose |
| Root-cause tooling | Included (traces, artifacts, causality analysis, time travel) | Must be built; this is where in-house efforts typically stall | Logs, traces, dashboards; the three-week archaeology problem persists |
| Ongoing maintenance | Vendor's problem, contractually | **Yours, permanently** — the fault model and simulator drift from reality (`deterministic_engineering_guide.md` §5.6) | Operational maintenance only |
| Time to first finding | Illustratively **1–2 quarters** from signature (pre-work + onboarding) | Illustratively **4–8 quarters**, if ever | Immediate, but for a different bug class |
| Exit / lock-in | Transfer of an undocumented quantity: run history, multiverse, reproducibility. **The novel exposure in §7.6** | None (you own it) | None |

**The honest reading of that table:** the *build* column is a fiction for a settlement system that already exists. Reaching the five architectural requirements of `deterministic_engineering_guide.md` §5.5 in a live payments state machine is a rewrite, not a project — which is precisely the trap Antithesis's product exists to route around (§3.1). **Build is realistically available only to a team writing the component from scratch**, and Cymbal is not. That leaves buy versus skip, and the difference between them is not the platform cost — it is whether the invariant formalisation happens at all. And if it happens, it has value even if the platform is never bought (§12.6).

### 12.5 The Evidence and Vendor-Risk Questions From §11

Cymbal must resolve these **before** signing, and this guide's honest position is that several cannot be answered from the public record at all:

| # | Question | Where it stands |
|---|---|---|
| 1 | **Registered legal entity and jurisdiction** — required for any vendor-onboarding and procurement record | ❌ Not published; must be obtained from the vendor (§2.1, §15) |
| 2 | **Will the audit and model-validation functions accept this output as evidence?** | ⚠ Probably as supporting evidence only. The honest answer in §11.2 is that the vendor publishes no simulator-validation document, no fidelity statement and no coverage metric, and that output from a third-party simulator is unlikely to serve as validation evidence in its own right. **Ask the validation function, in writing, before the technical evaluation.** |
| 3 | **Where does the settlement data sit, and what comes back?** | The vendor's VPC option keeps execution and images in Cymbal's VPC but the **control plane in the vendor's** (§7.3). Cymbal must classify: workload data (synthetic? production-derived?), extracted artifacts (core dumps and data files — possibly containing ledger content), and control-plane metadata. ❌ No vendor security documentation, certification list or sub-processor list is published |
| 4 | **Artifact lock-in and exit** | The novel exposure in §7.6: can Cymbal export run artifacts and **reproduce a previously-found bug without the platform**? ❌ Undocumented; negotiate a contractual export right **before** the first run |
| 5 | **Are the vendor's case studies transferable to a settlement system?** | Mostly no. The strongest public evidence is a database storage engine (MongoDB) and a database rewrite (Turso) (§6). Settlement is a *close* neighbour — concurrent, stateful, invariant-rich — but the transfer is an inference, not evidence. One detail to check explicitly: in the MongoDB record, the **workload container was excluded from network fuzzing** (§4.4) — ask whether the settlement workload container would be, and what that means for the fault coverage of the harness itself |
| 6 | **Vendor risk** | A Series-A, closed-source vendor with no on-premise option and no published SLA. Mitigations are contractual (§11.4). The December 2025 round materially improves the position; the customer-investor dual role (§2.3) is a fact to disclose in Cymbal's own evaluation paper, not a disqualifier |

### 12.6 The Recommendation

**Recommendation: do not buy in year one. Write the invariants and the workload first; rehearse the decision next year with the invariant set in hand and the vendor's actual quote on the table.**

The reasoning, in the order that matters:

1. **Obligations one and two are Cymbal's, and they are the binding constraint (§11.5).** The platform's value is entirely conditional on the invariant set and the drivable workload. Cymbal has the rare advantage of invariants already written in business prose — spend the effort converting them into assertions.
2. **Those artefacts are valuable without the platform.** Formalised settlement invariants immediately improve the existing integration suite, sharpen incident triage (a violation is now a *named* property, not a suspicion), and make the two historical incidents analysable in the language the platform would use. **This is the rare pre-work that pays for itself if the purchase never happens.**
3. **The build option is not real and should be closed out explicitly.** For a live settlement system, reaching the five architectural requirements is a rewrite (`deterministic_engineering_guide.md` §5.5). Recording that conclusion now prevents the same debate recurring each budget cycle.
4. **The buy decision cannot be priced today, and should not be pretended otherwise (§7.5).** Obtain a real quote — including the workload scale, the VPC deployment option, the support tier and the artifact-export terms — and compare it against the measured cost of the two historical incidents. An illustrative table cannot make that call.
5. **Revisit with a defined trigger, not a date.** Buy in the second year if, and only if: **(a)** the invariant set is formalised, version-controlled and owned by an engineering team; **(b)** the workload container can drive a settlement cycle in CI; **(c)** the validation function has responded in writing on how it would treat the output; and **(d)** the vendor's quote, its artifact-export terms and its security documentation have been obtained and reviewed. If all four are true, the purchase is a technical formality. If (a) and (b) are not true after a year, the honest conclusion is that Cymbal does not have the precondition, and no vendor can supply it.

**What Cymbal should *not* do** is sign a platform contract in year one on the strength of the vendor's case studies while planning to build the invariants later. That is the anti-pattern in §13, and it is how this category of purchase fails: not because the platform is bad, but because it was bought before the customer could ask it a question.

---

## 13. Anti-Patterns

| Symptom | Cause | Guardrail |
|---|---|---|
| A platform subscription with no findings after two quarters | **Buying a category you do not need — no invariants to assert.** The platform searches for deviations from promises; there were no promises. The licence renews, the dashboard stays green | Make invariant formalisation a **gate before purchase**, not a workstream after it. If the team cannot list ten properties the system must satisfy, the answer is "not yet" (§10.3, §12.6) |
| "We do deterministic simulation testing now" — while fault schedules are run by hand against staging | **Treating the platform as chaos engineering.** Real faults in a real environment are not deterministic, and a determinism vendor does not give you a chaos programme | Separate the two purchases and the two teams' objectives. `chaos_engineering_guide.md` owns the category; the mature posture is both, in the order given in `deterministic_engineering_guide.md` §5.7 |
| Every pull request is gated on a simulation run; failures are triaged as flakiness | **Treating the platform as a test suite.** A simulation run is a *search*, not a pass/fail suite; its green state means "no violation for these invariants under this fault model", not "correct" | Define what a run's outcome means before wiring it into CI. Automatically retrying a failed simulation is the same destructive practice as retrying a flaky test (`deterministic_engineering_guide.md` §11.1) |
| The pilot found nothing surprising, and the vendor's MongoDB/Ethereum results were cited as the expectation | **Assuming the vendor's case studies transfer to your system.** The vendor's evidence clusters on databases, storage engines and protocol clients (§6) — systems that are concurrent, stateful, containerisable, and already full of written invariants | Ask the transferability question explicitly: is my component in the same class as the published evidence, and if not, what makes me think the result transfers? (§12.5, item 5) |
| High run volume, long runtimes, zero violations reported | **A workload with no properties to violate.** Load without invariants is a performance test. The fault injection is doing its job; nobody has told the platform what "wrong" looks like | Write the assertions first, and assert *hard things* — recovery after a single node dies, no duplicate netting, replicas converge, the ledger balances after replay — not just liveness. Liveness-only properties find nothing on a system that is merely slow |
| A signed contract with no answer to "what happens to our run artifacts at termination?" | **An evaluation that ignores the lock-in question.** The vendor's differentiating output is *explanations of your own defects*, and its exportability and post-termination durability are undocumented (§7.6) | Negotiate three things before the first run: bulk artifact export in a usable format, whether a seed + binary is sufficient to reproduce a finding independently, and deletion/retention terms that survive termination |
| A one-day PoC that is declared a success because the environment booted | **A purchase that precedes the harness.** The vendor's "as little as a day" onboarding claim is about getting the system *running*, not about finding bugs (§5.3) | Judge the pilot on findings against a frozen invariant set and a measured baseline (§5.5), never on boot success |
| The strongest internal testimonial cites Jane Street, Etherereum or MongoDB equally, as if all three were equally evidenced | **Conflating vendor claims with verified customer evidence.** Two of the three most prominent names rest on vendor-published material, and one of those is the lead investor | Use §6's grades verbatim in the internal paper. A logo is not a customer; a vendor-published case study is not a customer's own account; a customer-investor's testimonial is neither |
| The evaluation cites "6 months of testing in an hour" or "6 hours covers more than 100 engineers" as a buying rationale | **Converting a vendor's marketing multiple into a fact.** These are testimonials with no baseline, denominator or method (§6.1, §14) | Translate every multiple into a measurable pilot target before the pilot, and measure it |
| The vendor's agent skills generate the specification, the manifests and the triage, and nobody in the bank reads them | **Outsourcing the one artefact that is the customer's to own.** The invariant set is the only part of the process an auditor can inspect (§11.2) | Generated artefacts must be reviewed, version-controlled and owned by a named human in the bank, with the same sign-off as any other control document |
| A green simulation run is written up as "the system is correct" | **Mis-stating a negative result.** "No violation found" is bounded by the invariant set, the fault model, the workload and the run duration | State the bound every time: *no violation of these N invariants under this fault model in this run* |

---

## 14. The Claims Audit

The discipline of this table is the guide's central evidentiary rule: **separate COMPANY CLAIMS from CUSTOMER CLAIMS from DOCUMENTED FACT**, and date everything. ✅ = verified at source; ⚠ = flagged (vendor claim, marketing, or third-party-derived); ❌ = rejected.

| # | Claim | Type | Source and date | Grade | Quality / note |
|---|---|---|---|---|---|
| 1 | Antithesis is built on a custom deterministic hypervisor called **the Determinator**, a fork of FreeBSD's **bhyve** | **DOCUMENTED FACT** (vendor engineering disclosure, corroborated by the repository) | antithesis.com blog, 20 March 2024; recorded in `deterministic_engineering_guide.md` §5.3 | ✅ | The single most technically credible thing the company publishes. Specific enough to be falsifiable, and falsification would be public |
| 2 | There is **no free or open-source version** | **DOCUMENTED FACT** (vendor's own FAQ, negative) | vendor FAQ, retrieved September 2026 | ✅ | A verified *negative* — unusually valuable, and consistent with dhyve's authors describing Antithesis's hypervisor as "closed-source" |
| 3 | There is **no fully on-premise option** | **DOCUMENTED FACT** (vendor's own FAQ, negative) | vendor FAQ, retrieved September 2026 | ✅ | Qualified by the VPC option, whose control plane stays with the vendor |
| 4 | The platform runs "the state of the entire system/experiment/workload as an interconnected whole, not any single process or server" | **COMPANY CLAIM** (architectural) | 20 March 2024; quoted in `deterministic_engineering_guide.md` §5.1 | ✅ as a quotation, ⚠ as a capability | The quotation is verified; the *capability* is verified only by the vendor's own description plus one customer's account of checkpoint/branching (MongoDB, 2022/2024) |
| 5 | Simulates "clock, scheduling, network, and storage faults **with zero configuration**" | **COMPANY CLAIM** | product page, retrieved September 2026 | ⚠ | "Zero configuration" cannot be literally true — someone configures the workload, and MongoDB's own wiki enumerates an extensive fault-parameter set (packet latency, drop probability, partition cadence, teardown cadence). The claim means "defaults exist", not "nothing to configure" |
| 6 | "Our simulator is 100% deterministic and automatically root causes every bug" | **COMPANY CLAIM** with two absolutes | product page, retrieved September 2026 | ⚠ | Both absolutes are unsupported. The customer account shows root-causing required a customer engineer's domain expertise and a hand-produced two-second core-dump window (22 April 2024) |
| 7 | Guidance component "uses RL, but you can tell your boss it's AI" | **COMPANY CLAIM** (documentation) | vendor docs, retrieved September 2026 | ⚠ | The most honest line on the vendor's site, and the only public statement of the guidance mechanism. No architecture, no metric |
| 8 | Onboarding "as little as a day" | **COMPANY CLAIM** | vendor FAQ, retrieved September 2026 | ⚠ | Scoped to running the system, not to finding bugs. No independent verification located |
| 9 | "increased revenue by over 12x over the last two years" | **COMPANY CLAIM** | vendor-issued release, 3 December 2025 | ⚠ | **No baseline, no absolute figure, no denominator, no corroboration.** Do not convert into a fact; note that the growth is measured in a period that includes a category with zero commercial competitors (§8.4) |
| 10 | "We've built bug-free software… we only shipped one user-reported bug before Apple acquired FoundationDB in 2015" | **COMPANY CLAIM** | company About page, retrieved September 2026 | ⚠ | "User-reported bug" is a self-selected denominator. FoundationDB's quality is independently well regarded, but this specific metric is unauditable |
| 11 | "Nothing like our simulation framework existed anywhere else" | **COMPANY CLAIM, overstated** | company About page | ⚠ | The *technique* was public from Wilson's Strange Loop 2014 talk and documented by FoundationDB; TigerBeetle's VOPR documents itself as "heavily inspired by the work of FoundationDB and Antithesis". The accurate claim is that no **product** existed |
| 12 | Jane Street "depends on Antithesis to validate complex distributed systems critical to the operation of its global trading business" | **CUSTOMER CLAIM — but furnished to the vendor, by the vendor's lead investor** | vendor product page and vendor release, 3 December 2025 | ⚠ | Disclose the dual role (§2.3). ❌ No Jane Street–published write-up located |
| 13 | MongoDB uses Antithesis, integrated at CI level, since 2021, for storage engine, core server, cluster-to-cluster sync and upgrade/downgrade | **CUSTOMER CLAIM — customer's own source** | **mongodb/mongo wiki, last edited 18 February 2022** | ✅ | The strongest customer evidence in the file: the customer's own repository, with the workload invocation and fault parameters published |
| 14 | "Antithesis has surfaced novel failures that were not detected using the `ContinuousStepdown` hook… bugs that have historically been difficult to reproduce have been reproduced consistently" | **CUSTOMER CLAIM** | MongoDB wiki, 18 February 2022 | ✅ | Qualitative and specific. Runs *against* the vendor's own interests in one respect: it says the failures were "novel" *relative to an existing fault-injection hook*, not relative to testing in general |
| 15 | A WiredTiger index-inconsistency bug after replication rollback was reproduced and root-caused with Antithesis's artifacts, ticket WT-9500 | **CUSTOMER CLAIM** (first-person account by a named former MongoDB engineer) | vendor-hosted guest post, 22 April 2024 | ⚠/✅ | The *person* and the JIRA ticket are checkable; the hosting is the vendor's. It is the most useful single document in the public record because it documents a **limitation** (no instrumentation possible; artifacts extracted by hand) alongside the success |
| 16 | Ethereum used Antithesis for The Merge; 13 panic/crash/DoS bugs and 22 chain-operation bugs found; "dozens of serious bugs… resolved" | **COMPANY CLAIM** (byte vendor case study with EF individuals quoted) | vendor case study, 13 February 2024; repeated in the vendor release, 3 December 2025 | ⚠ | Named EF individuals are quoted **on the vendor's page**; ❌ no EF-side publication located. The vendor does link to upstream PRs, which a diligent buyer should open. **The bug counts are vendor-authored numbers with a vendor date** |
| 17 | Turso uses DST at its core plus "a partnership with Antithesis" to find bugs "including bugs in the simulator itself" | **CUSTOMER CLAIM — customer's own blog** | turso.tech/blog, January 2025, via `turso_database_guide.md` | ✅ | The cleanest independently published customer statement: on the customer's own blog, describing a two-layer arrangement with its own in-house simulator |
| 18 | Testimonials from Formance, Tigris, ParadeDB, PingThings, Mysten/Sui, Readyset and the etcd/Kubernetes maintainer | **CUSTOMER CLAIMS, vendor-published, some with headline multiples** | vendor product page and vendor blog, 2024–2026 | ⚠ | Treat as marketing until the party publishes its own account. The multiples ("six months… in an hour"; "6hrs… more than 100 engineers"; "3 weeks… one day") have **no baseline, denominator, period or method** |
| 19 | MongoDB outcome figures of "100+ critical bugs, 75% missed by internal tests, 47% MTTR reduction, ~10x ROI" | **THIRD-PARTY-DERIVED** | a third-party case-study aggregator page | ❌ **REJECTED** | Appears nowhere in MongoDB's or Antithesis's own material; no methodology, period or denominator |
| 20 | Antithesis "runs your whole system on a single CPU core" per instance, x86-only, Intel VMX, optimised for throughput over latency | **DOCUMENTED FACT** (engineering disclosure) | 20 March 2024; product page | ✅ | The most consequential architectural fact for a buyer, and it is disclosed plainly |
| 21 | "Instruction-level determinism works with any x86 binary" | **COMPANY CLAIM** | product page, retrieved September 2026 | ⚠ | Plausible given the hypervisor design but unverified; note that the same engineering post describes needing custom workarounds for CPU behaviours that were "just a bit past what they were really designed for" |
| 22 | $47M seed (Feb 2024) and $105M Series A led by Jane Street (3 Dec 2025), with named additional investors | **COMPANY CLAIM via a vendor-issued press release** | PR Newswire, 3 December 2025; company About page | ⚠ | The release is a vendor document distributed through a wire service. **Some of the additional investors are individuals** (Patrick Collison, Dwarkesh Patel, Sholto Douglas), which the release itself states |
| 23 | "Jane Street is both an Antithesis investor and customer" | **DOCUMENTED FACT** (self-disclosed) | vendor release, 3 December 2025 | ✅ | A genuinely important disclosure, made by the company itself |
| 24 | Antithesis is "based in Northern Virginia", founded 2018, launched out of stealth 2024 | **COMPANY CLAIM** | vendor release boilerplate, 3 December 2025 | ⚠ | ⚠ Dateline Tysons Corner, Va.; aggregator variants give Vienna, Va. ❌ No registry filing located (§2.1) |
| 25 | **No price is published** | **DOCUMENTED ABSENCE** | product page, FAQ, docs, release — all examined, retrieved September 2026 | ✅ | **The pricing finding.** Evidence that a price exists and is enterprise-grade: the vendor's own statement that "we knew our pricing would be tough for small teams" (28 February 2024) |
| 26 | Antithesis onboarded its first customer in **September 2019** | **COMPANY CLAIM** | company About page timeline | ⚠ | Not named on that page. If true, it is a strong signal (customer predates funding by 4.5 years) |
| 27 | The Determinator's CTO changed from Dave Scherer (March 2024) to Ben Collins (September 2026) | **DOCUMENTED CHANGE**, no stated reason | vendor blog 20 March 2024 vs vendor leadership page | ✅ as a fact, ❌ as to cause | Recorded; cause not inferred (§2.2) |

---

## 15. What Could Not Be Verified

A numbered list, and the honest boundary of this guide. Each item is a **gap**, not a finding that the answer is negative — except where marked "verified negative". Searches that returned empty result sets are recorded as **tool limitations**, not as evidence of absence.

### 15.1 Corporate and Legal

1. ❌ **The registered legal entity name, jurisdiction, incorporation number and registered agent.** No state corporate-registry filing (Virginia SCC, Delaware, or any other) was located. Searches for a registry record returned empty results — **tool/coverage limitation.** The only company-issued locality statements are "TYSONS CORNER, Va." (release dateline) and "based in Northern Virginia" (boilerplate).
2. ❌ **Ownership percentages, cap table, and any shareholder list beyond the named 2025 investors.**
3. ❌ **Board composition, board seats attached to the Series A, and the identity of any director beyond the executive leadership list.**
4. ❌ **Any company statement explaining the CTO title change** from Dave Scherer (March 2024) to Ben Collins (present).
5. ❌ **Headcount**, and any company-published size figure.
6. ❌ **Any financial statement, revenue figure, burn rate or runway of any kind.** The "12x revenue growth" claim has no corroboration and no baseline.
7. ❌ **Whether any customer contract contains exclusivity, most-favoured-nation, competitive-restriction or non-compete terms** — relevant given the customer-investor dual role (§2.3).

### 15.2 Commercial

8. ✅ **VERIFIED NEGATIVE — no published price, no licence form, no unit of sale.** Confirmed across the product page, FAQ, documentation and the funding release.
9. ❌ **Support tiers, SLAs, response times, uptime commitments, professional-services catalogue.**
10. ❌ **Exit assistance, data-portability commitments, escrow arrangements, artifact retention/deletion terms at termination** (§7.6 — the artefact-lock-in question).
11. ❌ **Whether the "Antithesis Open Source Giveaway Program" is still running** in 2026, and what it has produced since 2024.
12. ❌ **The commercial terms behind AWS Marketplace Private Offers** — whether this is subscription, consumption, or a committed-spend drawdown with overage.

### 15.3 Technical and Security

13. ❌ **Any security documentation**: no whitepaper, no SOC 2 / ISO 27001 or equivalent certification evidence, no penetration-test disclosure, no vulnerability-disclosure policy (one is referenced in the 2024 Giveaway post as existing but "not posted on our website yet"), no sub-processor list, no data-residency schedule.
14. ❌ **Enumeration of what the control plane holds** in the run-in-your-own-VPC architecture, where the control plane remains in the vendor's VPC.
15. ❌ **Any simulator-validation, fidelity or coverage document** — what the environment does not model, how the simulator itself is tested, and what the vendor does when its environment produces a wrong result (§11.2, item 1).
16. ❌ **The guidance algorithm beyond "it uses RL"**, and any exploration-efficiency metric.
17. ❌ **Whether the artifact-extraction workflow described as manual in the 2024 MongoDB account (core dumps "done by hand") is now automated**, and to what degree.
18. ❌ **Any published language/runtime support statement beyond the eight SDKs**; whether there is any support for non-x86.
19. ❌ **Whether the environment's single-VM topology is stated anywhere as a limitation** by the vendor — the honest statement of the fidelity boundary.
20. ❌ **Pricing of compute, limits on run size, and what happens under heavy usage.**

### 15.4 Evidence

21. ❌ **Any Ethereum Foundation–published confirmation** of the Antithesis engagement, bug counts or outcomes. Searches for an EF-side write-up returned empty result sets — **tool limitation**, not proof of absence.
22. ❌ **Any Jane Street–published engineering write-up**, blog post or talk about using Antithesis. This matters disproportionately because Jane Street leads the round (§2.3).
23. ❌ **Any published account by Tigris, Formance, ParadeDB, PingThings, Mysten/Sui or Readyset** on their own channels.
24. ❌ **Any bank or financial institution's own publication** naming Antithesis. The vendor's FAQ refers to unnamed "capital markets customers" using multicast networking — a **vendor claim about an unnamed population**, recorded as such and never converted into a customer list (§4.2).
25. ❌ **Any independent benchmark, bug-count verification or reproduction study** by a third party.
26. ✅ **Verified negative in one direction:** the "100+ critical bugs / 75% / 47% / 10x ROI" MongoDB figures could not be traced to MongoDB or Antithesis and are **rejected** (§14, #19).

### 15.5 Method and Tool Limitations (Recorded Honestly)

- ⚠ **Several `web_search` queries returned empty result sets** during this pass — notably the pricing searches ("Antithesis pricing", "Antithesis cost per run", combos with "per hour"/"license"), the search for an Ethereum Foundation write-up, the search for a Readyset-side publication, and the search for a Virginia corporate registry record. Per this repository's rule, **empty searches are recorded as tool limitations, not as evidence of absence.** The fallback was direct `web_extract` on primary URLs, which is how most verified material here was obtained.
- ⚠ **The company's own docs depth was sampled, not exhausted.** The docs site's product and reference sections were read at the "How Antithesis works" level plus the FAQ; the full reference documentation (fault configuration knobs, SDK reference, dependency-mock list, scoring/report definitions) was **not** read end to end in this pass. A serious evaluation should read it, because it is where the product's real technical commitments live.
- ⚠ **The vendor's blog archive was enumerated from its index but not read post by post.** The index (retrieved September 2026) shows a dense 2026 cadence — causality analysis (11 May 2026), Tigris report (21 April 2026), agent skills (25 March 2026), Hegel (24 March 2026), Raft implementations (27 July 2026), the WAL reset bug (12 August 2026), Bug Bash Europe (Copenhagen, 30 September 2026) — and customer stories (Readyset, 19 February 2026). Individual posts beyond those cited here were not read in full.
- ⚠ **No non-public source was used and no vendor contact was made.** Everything here is from public pages, the customer-authored pages identified, and this repository's own existing records. A serious diligence exercise would put the §15 questions to the vendor in writing; this guide could not, and says so.
- ⚠ **The repository's own §5 record was relied on for the technique-level facts** (the Determinator/bhyve identification, the FAQ's "Sadly, no.", the latest Java SDK tag v1.7.0 dated 2026-09-02, the Hegel/Bombadil/dhyve adjacency) rather than re-verified from scratch, per the task boundary. One of those — dhyve — **was** independently checked in this pass and the "inspired by Antithesis" description was verified at dhyve's own README (§7.2).

### 15.6 The Three Items That Matter Most

If a reader takes three things from this section: **(1) the corporate identity and ownership are not checkable from outside — no registry filing, no cap table, no board — which for an enterprise buyer is a procurement blocker, not a curiosity; (2) the commercial model is invisible — no price, no licence form, no SLA, no exit terms, and no answer to the artifact-lock-in question; and (3) the customer evidence is thinner than the logo wall suggests — exactly two parties (MongoDB, Turso) published their own accounts, while the most-cited names rest on vendor-published material and one of them is the lead investor.** Everything else in this list is a detail; those three are the diligence file.

---

## 16. Glossary, Cross-References and Closing Summary

### 16.1 Glossary

| Term | Meaning in this guide |
|---|---|
| **The Determinator** | Antithesis's custom deterministic hypervisor, a fork of FreeBSD's **bhyve**. Runs the customer's whole system on a single CPU core per instance; x86 and Intel VMX. §4.1 |
| **Deterministic hypervisor** | A hypervisor that enforces determinism on everything inside a VM — instruction stream, time, interrupts, I/O — rather than merely virtualising hardware. §1.4 |
| **Director** | The snapshot/branch/mutation harness accompanying **dhyve**. §7.2 |
| **dhyve** | Open-source deterministic hypervisor built on bhyve, BSD-2-Clause, a DTU bachelor's project by Peter Graugaard and Nicholas Kristiansen, explicitly inspired by Antithesis. §7.2, §8.1 |
| **DST** | Deterministic simulation testing — running real code against a simulated, seeded, controllable environment. The technique; owned by `deterministic_engineering_guide.md` §5. §9 |
| **The environment / the Antithesis environment** | The vendor's simulated world: one or more computers, as a collection of containers, inside a single hypervisor-managed VM. §4.3 |
| **Fault injection** | Seed-derived introduction of faults — network partitions, packet loss/latency, node kills, storage and disk faults, thread starvation, clock effects. §4.4 |
| **Hegel** (hegel.dev) | The team's open-source property-based testing libraries. Adjacent tooling, not a simulator. §7.2 |
| **Bombadil** (bombadil.bot) | The team's open-source property-based testing tool for web and terminal UIs. §7.2 |
| **The multiverse** | The vendor's name for the tree of branching timelines produced in a single test run — "tens (or hundreds) of thousands of alternate universes". §1.4, §4.3 |
| **Causality analysis** | The vendor's automated pinpointing of events causally related to a bug, plus failure-probability-over-time; prototyped with MongoDB, shipped as a feature (blogged 11 May 2026). §4.6, §6.3 |
| **Time-travel debugging** | Suspending, rewinding and interactively inspecting a deterministic run, including destructive analysis. §4.6 |
| **Invariant / assertion** | The customer-written statement of required behaviour, expressed as assertions in the customer's test harness. **The precondition for the platform finding anything.** §4.5 |
| **Workload container** | The container holding the customer's test suite / harness, which the platform drives; in MongoDB's published example, the container holding `resmoke`, excluded from network fuzzing. §4.5, §6.3 |
| **Reproducible failure** | A failure that recurs on demand from the same seed and binary. The product's central claim. §1.4 |
| **The four steps** | The vendor's onboarding sequence: outline your invariants → send us your system → we find your bugs → and root cause them. §5.1 |
| **VOPR** | TigerBeetle's in-house deterministic simulator (Viewstamped Operation Replicator) — the in-tree alternative to buying an environment. §8.1, §8.3 |
| **Flow** | FoundationDB's actor-concurrency language, to which its in-tree simulator is coupled. §8.1 |
| **Bhyve** | FreeBSD's open-source hypervisor; the base of both the Determinator and dhyve. §4.1 |
| **Artefact lock-in** | This guide's term for the unaddressed risk that run history, the multiverse, traces and the ability to reproduce a previously-found bug live only in the vendor's platform. §7.6, §11.4 |
| **VPC deployment option** | Antithesis run in the customer's own AWS VPC: execution environments and container registry in the customer's VPC, control plane in the vendor's, single-tenant compute and databases. §7.3 |
| **AWS Marketplace Private Offers** | The procurement route the vendor says it "often" uses, drawing down existing committed cloud spend. §7.3 |

### 16.2 Cross-References in This Repository

Read alongside this guide, and note which owns what — the boundary stated in §1.5:

- **[deterministic_engineering_guide.md](deterministic_engineering_guide.md)** — **the technique guide.** Its **§5 ("Toolkit II: Deterministic Simulation Testing")** owns the definition, the "not mocking" distinction, the Will Wilson Strange Loop 2014 talk as canon, the FoundationDB documentation, the implementations table (with Antithesis's row), the benefits, the five architectural requirements, the costs, and the comparison to chaos testing. Its **§11** owns property-based and model-based testing and the reproduction-handle discipline; its **§3** owns the taxonomy of nondeterminism; its **§5.7** owns the "both, in this order" posture for simulation versus real-environment testing.
- **[deterministic_simulation_testing_guide.md](deterministic_simulation_testing_guide.md)** — **the practice guide.** How a team constructs and operates deterministic simulation: harness design, the stub boundary, seed handling, workload design, fault models, organisational cost. Read it before attempting to satisfy this guide's §5.2 preconditions.
- **[chaos_engineering_guide.md](chaos_engineering_guide.md)** — **the category this product is confused with.** Chaos engineering injects real faults into real running environments and is not deterministic. §1.3 and §8.1 state the precise difference and the vendor's own "extreme version of chaos engineering" framing.
- **[turso_database_guide.md](turso_database_guide.md)** — the repository's record of a **customer's own published account** of using DST at the core of a database rewrite plus a partnership with Antithesis (January 2025). Cite it rather than re-deriving it. It is one of only two customer-side verifications in this guide (§6).
- **[../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md)** — important-business-service mapping and third-party dependency: the framing for §11.3 and §11.4.
- **[../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md)** and **[../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)** — how a bank structures concentration and control risk, and the Singapore outsourcing/technology-risk and operational-resilience expectations a contract of this type must satisfy (§11.3, §11.4).
- **[../management/vendor_management_guide.md](../management/vendor_management_guide.md)** (if present in this repository's `management/` directory) — the vendor-assessment discipline behind §11.4's mitigations.
- **[test_orchestration_guide.md](test_orchestration_guide.md)** — test strategy, levels and environments; the general testing frame that `deterministic_engineering_guide.md` §11 deliberately excludes and that this guide's §5 preconditions sit inside.

### 16.3 Closing Summary

Antithesis is a **private, venture-backed United States company** — founded **January 2018** by **Will Wilson** and **Dave Scherer**, both from FoundationDB, headquarters reported as Northern Virginia (Tysons Corner), **$47M seed announced February 2024** and a **$105M Series A led by Jane Street announced 3 December 2025** — that sells exactly one thing: access to a **deterministic simulated environment** in which a customer's whole distributed system runs, under injected faults, reproducibly. It is the **only commercial product in its category** that this guide could locate, and its technical foundation is asserted with unusual specificity: a custom deterministic hypervisor, **the Determinator**, forked from FreeBSD's **bhyve**, running x86 binaries on a single core per instance, with time, scheduling, network and storage all virtualised.

The evidence for its value is **real but thinner than its logo wall**. Exactly two customers are verified at their own published sources — **MongoDB**, whose own repository wiki documents the workload, the fault parameters and the qualitative result (February 2022), and **Turso**, whose blog documents a two-layer DST arrangement including a partnership with Antithesis (January 2025). The three most prominent names — **Jane Street, Ethereum and MongoDB** — appear in a vendor-issued release, and **Jane Street is simultaneously the lead investor and the customer**, a dual role the company discloses and any buyer must weigh. Every quantitative outcome claim located is vendor-authored: the 12x revenue growth, "one user-reported bug" before Apple acquired FoundationDB, "six months of testing in an hour", "6hrs covers more than 100 engineers". One widely circulated set of MongoDB outcome figures — 100+ critical bugs, 75% missed by internal tests, 47% MTTR reduction, 10x ROI — could not be traced to MongoDB or to Antithesis and is **rejected** here.

Commercially, the product is **opaque by design or by stage**: there is **no free or open-source version** ("Sadly, no"), **no fully on-premise option**, no published price, no published licence form, no SLA, and no answer to the question that matters most for a platform whose output is the explanation of your own defects — what happens to your run history, your artifacts and your ability to reproduce a discovered bug when the contract ends. The adjacent open-source tools (**Hegel**, **Bombadil**, and the DTU-built **dhyve**) are honest and useful and are not substitutes for the environment, which stays closed and commercial.

What the company has actually solved is the problem this guide's thesis names: deterministic simulation testing is not hard because it is secret, but because it requires **owning the runtime, the scheduler, the clock and the network** — which trapped it for two decades inside a handful of organisations that wrote their own database and their own concurrency language to get it. Antithesis's insight is that the environment can be owned by a third party and rented, leaving the customer only the two obligations it was always going to have: a containerised system it can launch and drive, and invariants worth asserting. **The first is a deployment problem. The second is the whole purchase.** A buyer with a containerised, concurrent, stateful system and written-down invariants has a genuine and well-evidenced fit — the database and storage-engine population the vendor's evidence actually covers. A buyer without them is being sold a category it cannot yet use, and the honest recommendation for that buyer is the one §12 reaches illustratively: **write the invariants first, keep them whether or not a platform is ever bought, and revisit the vendor with a real quote, a frozen property set and the security and exit terms in writing.**

That is the whole of the product judgement: the technique was never the scarce thing, the method was public from 2014, and the corpus of research on it is in this repository's other guides — what Antithesis sells, and all it sells, is **the environment as a product.**