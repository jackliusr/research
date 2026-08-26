# Odoo Framework: The Open-Source ERP — A Comprehensive Technology Deep-Dive

**The TinyERP 2005 Origin, the OpenERP Decade, the 2014 Odoo Rebrand, the Python/PostgreSQL/ORM Architecture, the Community-vs-Enterprise Open-Core Split, the Apps-and-OCA Ecosystem, the Annual Release Cadence, the Odoo-vs-Tryton Head-to-Head, the ERP-in-a-Bank Angle, and the Cymbal Bank Back-Office Implementation Scenario — Verified Against Odoo's Own Sources, Wikipedia, the OCA, and the Tech Press**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research / Framework Deep-Dive — Odoo: Overview (the Definition, the Overview Table), History (TinyERP 2005, the OpenERP Era, the 2014 Rename, the Growth Era), Architecture (Python + PostgreSQL, the ORM, the Module System, the OWL Web Client, the Server), Editions (Community vs Enterprise, Odoo.sh, On-Prem), Apps and Ecosystem (Core Apps, the App Store, the OCA), Release Cadence (Version History, LTS), Comparison (Odoo vs Tryton, Odoo vs SAP Business One), the ERP-in-Banking Context, the Worked Example (a Cymbal Bank Back-Office), Summary, Glossary, Verification Ledger
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** odoo.com (the official site — pricing, release notes, on-premise documentation, the news blog), the GitHub repositories (odoo/odoo — 54k stars, the release branches; odoo/owl — the OWL web framework), Wikipedia (the Odoo and Tryton articles), the OCA (odoo-community.org and github.com/OCA — the Odoo Community Association), and the tech press (TechCrunch 2010/2019/2024, CNBC November 2024, SiliconANGLE, De Tijd — the funding and valuation coverage)
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder):** [Asia Fusion Technology](asia_fusion_technology_guide.md), [jBASE / UniVerse](jbase_universe_guide.md), [jBASE vs InfoBasic](jbase_vs_infobasic_guide.md)
> **Companion guides (banking/, management/, personal/):** [Insurance — Open Source vs Commercial](../banking/insurance_open_source_commercial_guide.md) — *the Odoo passing mentions*, [Apache Fineract](../banking/apache_fineract_guide.md), [Mojaloop](../banking/mojaloop_guide.md), [T24 Programming](../banking/t24_programming_guide.md), [Business Case Development](../management/business_case_development_guide.md), [Jack Liu Profile](../personal/jack_liu_profile.md)

---

## How to Read This Guide

This is the **dedicated deep-dive on Odoo** — the open-source ERP/CRM suite — in the `technology/` framework series. It is written because the repository had **no dedicated ERP framework guide**: Odoo previously appeared only as passing mentions in [../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md) (the insurance long-tail table) and in the author's own career profile as part of the Asia Fusion domain spread ([../personal/jack_liu_profile.md](../personal/jack_liu_profile.md): "Enterprise Resource Planning (ERP) — including Odoo and iDempiere frameworks"; [asia_fusion_technology_guide.md](asia_fusion_technology_guide.md) §4.4–4.5). This guide is that missing dedicated treatment — **and it doubles as the answer to the Tryton question that immediately preceded it**: §7 carries a full Odoo-vs-Tryton head-to-head, because the two share a lineage (Tryton forked TinyERP in 2008; Odoo is the OpenERP continuation) and the comparison is the clearest way to see what each project chose to be.

**The verification discipline.** Every load-bearing fact in this guide — dates, version numbers, licenses, funding figures, governance structures, architecture layers — was checked against primary or secondary sources in this research pass (August 2026): odoo.com (pricing page, release-notes pages, on-premise documentation, the news blog), the GitHub repositories (odoo/odoo, odoo/owl, github.com/OCA), odoo-community.org, Wikipedia (Odoo, Tryton), and the press coverage of Odoo's funding rounds. Facts that were confirmed are marked **✅**; facts that rest on a single source, on press figures that disagree with one another, on community-reported numbers, or on general knowledge that could not be re-verified this pass are marked **⚠** with an honest explanation. **Nothing is invented**: no version numbers, no dates, no user counts, no valuation figures. Where the record is ambiguous (the exact app-store listing count, the funding-round total, the LTS policy's evolution), the guide says so and moves on. Nothing in this guide is procurement, licensing, or investment advice.

**A note on web access during this pass.** The self-hosted search backend was flaky (queries returned empty results repeatedly), so verification leaned on direct URL extraction, the Wayback Machine's CDX API, and the repositories' own pages. That worked well for the core record; a handful of deep-dive pages (the Odoo supported-versions doc, the installation requirements doc) were bot-blocked and are flagged ⚠ where they would have been the ideal source. The verification ledger at the end records exactly what was verified and what was not.

### Quick Facts (Summary Card)

| Item | Value |
|---|---|
| What it is | Open-source business-management suite: ERP + CRM + e-commerce + back-office apps — "business management software suite developed in Belgium" — **✅ Verified** (Wikipedia; odoo.com) |
| Original author | **Fabien Pinckaers** (founder and CEO of Odoo S.A.) — **✅ Verified** (Wikipedia) |
| First release | **TinyERP, February 2005** — **✅ Verified** (Wikipedia infobox) |
| Renames | TinyERP → **OpenERP (2008)** → **Odoo (2014)** — **✅ Verified** |
| Stack | **Python server + PostgreSQL database + JavaScript/TypeScript web client** — **✅ Verified** (Wikipedia; the ORM/PostgreSQL backend is the documented architecture ⚠ for exact versions) |
| Community license | **GNU LGPLv3** (Community edition) — **✅ Verified** (Wikipedia; odoo/odoo LICENSE) |
| Enterprise | **Proprietary apps + support services; open-core model since 9.0** — **✅ Verified** (Wikipedia) |
| Current stable | **19.0 — 18 September 2025** — **✅ Verified** (Wikipedia/Wikidata); 18.0 was released **October 2024** — **✅ Verified** ("Meet Odoo 18" announcement, archived 3 Oct 2024) |
| Company scale | €282M revenue 2023 (+33%), 2,200+ employees, profitable — **✅ Verified** (De Tijd via Wikipedia) |
| Funding/valuation | 2019: $90M round; **Nov 2024: ~$500–527M secondary round at a $5.26–5.3B valuation** (backers: CapitalG, Sequoia, BlackRock) — **✅ verified with press figures that vary slightly ⚠** |
| GitHub presence | odoo/odoo: **54k stars, 33.5k forks, ~202k commits** (Aug 2026) — **✅ Verified** (repo page) |
| The OCA | Non-profit (Switzerland); **20,000+ community modules, 265 GitHub repositories**, 1,600 contributors — **✅ Verified** (odoo-community.org; github.com/OCA) |
| Tryton | The 2008 TinyERP fork; GPL-3.0-or-later; federation-governed — **✅ Verified** (Wikipedia) |
| Series context | The author's ERP-domain career background (Odoo/iDempiere) and the bank back-office angle — the ERP that is *not* the core-banking ledger |

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Definition — an Open-Source ERP/CRM Suite](#11-the-definition--an-open-source-erpcrm-suite-verified)
   - 1.2 [The Overview Table](#12-the-overview-table)
   - 1.3 [The Open-Core Model — Community and Enterprise](#13-the-open-core-model--community-and-enterprise-verified)
   - 1.4 [Why This Guide Exists — the Repository Gap and the Career Context](#14-why-this-guide-exists--the-repository-gap-and-the-career-context)
2. [The History](#2-the-history)
   - 2.1 [TinyERP — the 2005 Founding by Fabien Pinckaers](#21-tinyerp--the-2005-founding-by-fabien-pinckaers-verified)
   - 2.2 [The OpenERP Era — 2008 to 2014](#22-the-openerp-era--2008-to-2014-verified)
   - 2.3 [The Odoo Rename — 2014](#23-the-odoo-rename--2014-verified)
   - 2.4 [The Growth Era — Funding and Valuation (Flagged)](#24-the-growth-era--funding-and-valuation-flagged)
   - 2.5 [The History Table](#25-the-history-table)
3. [The Architecture](#3-the-architecture)
   - 3.1 [Python + PostgreSQL — the Verified Stack](#31-python--postgresql--the-verified-stack)
   - 3.2 [The ORM — Models, Recordsets, Automatic Schema](#32-the-orm--models-recordsets-automatic-schema)
   - 3.3 [The Module System — Addons, Manifests, Data Files](#33-the-module-system--addons-manifests-data-files)
   - 3.4 [The Web Client — the OWL Framework](#34-the-web-client--the-owl-framework-verified)
   - 3.5 [The Server — Workers, Cron, RPC](#35-the-server--workers-cron-rpc)
   - 3.6 [The Architecture Table](#36-the-architecture-table)
4. [The Editions](#4-the-editions)
   - 4.1 [Community vs Enterprise — the Feature Split](#41-community-vs-enterprise--the-feature-split)
   - 4.2 [Odoo.sh — the PaaS](#42-odoosh--the-paas-verified)
   - 4.3 [On-Premise — the Subscription-Keyed Deployment](#43-on-premise--the-subscription-keyed-deployment-verified)
   - 4.4 [The Editions Table](#44-the-editions-table)
5. [The Apps and the Ecosystem](#5-the-apps-and-the-ecosystem)
   - 5.1 [The Core Apps](#51-the-core-apps-verified)
   - 5.2 [The App Store — Scale Flagged](#52-the-app-store--scale-flagged)
   - 5.3 [The OCA — the Odoo Community Association](#53-the-oca--the-odoo-community-association-verified)
   - 5.4 [The Ecosystem Table](#54-the-ecosystem-table)
6. [The Release Cadence](#6-the-release-cadence)
   - 6.1 [The Annual Cadence — 8.0 (2014) to 19.0 (2025)](#61-the-annual-cadence--80-2014-to-190-2025)
   - 6.2 [The LTS Question — Flagged](#62-the-lts-question--flagged)
   - 6.3 [The Cadence Table](#63-the-cadence-table)
7. [The Comparison](#7-the-comparison)
   - 7.1 [Odoo vs Tryton — the Head-to-Head](#71-odoo-vs-tryton--the-head-to-head)
   - 7.2 [The Lineage — the TinyERP 2008 Fork](#72-the-lineage--the-tinyerp-2008-fork-verified)
   - 7.3 [Licensing — GPL-3 vs LGPL](#73-licensing--gpl-3-vs-lgpl-verified)
   - 7.4 [Philosophy — Framework vs Application](#74-philosophy--framework-vs-application)
   - 7.5 [Stability vs Ecosystem Breadth](#75-stability-vs-ecosystem-breadth)
   - 7.6 [Release Cadence — Annual vs Series](#76-release-cadence--annual-vs-series)
   - 7.7 [The Comparison Table](#77-the-comparison-table)
   - 7.8 [Odoo vs SAP Business One — Light, Flagged](#78-odoo-vs-sap-business-one--light-flagged)
8. [The ERP-in-Banking Context](#8-the-erp-in-banking-context)
   - 8.1 [The ERP in a Bank — Back-Office, Not the Ledger](#81-the-erp-in-a-bank--back-office-not-the-ledger)
   - 8.2 [The Banking Table](#82-the-banking-table)
   - 8.3 [Cross-References to the Bank Series](#83-cross-references-to-the-bank-series)
9. [The Worked Example — a Cymbal Bank Back-Office Implementation](#9-the-worked-example--a-cymbal-bank-back-office-implementation)
   - 9.1 [The Scenario](#91-the-scenario)
   - 9.2 [The Module Selection](#92-the-module-selection)
   - 9.3 [The Deployment Design](#93-the-deployment-design)
   - 9.4 [The OCA-Extension Design](#94-the-oca-extension-design)
   - 9.5 [The Lessons](#95-the-lessons)
10. [The Summary — the One-Page](#10-the-summary--the-one-page)
11. [The Glossary](#11-the-glossary)
12. [The Verification Ledger](#12-the-verification-ledger)

---

## 1. The Overview

### 1.1 The Definition — an Open-Source ERP/CRM Suite (Verified)

**Odoo is an open-source enterprise resource planning (ERP) and business-management suite** — a single integrated platform covering customer relationship management (CRM), sales, accounting, inventory, manufacturing, purchasing, human resources, project management, e-commerce, point of sale, and website building. Wikipedia defines it as "a business management software suite developed in Belgium" providing "applications for customer relationship management, e-commerce, accounting, manufacturing, warehouse management, project management, and inventory management" — **✅ verified** (the Wikipedia article, itself citing Odoo's own materials). The GitHub repository's own tagline is "Odoo. Open Source Apps To Grow Your Business." — **✅ verified** (github.com/odoo/odoo).

Three structural facts define what Odoo *is*, and all three are verified:

1. **It is modular.** "The platform consists of a core framework, official applications developed by Odoo S.A., and community-developed modules" (**✅ Wikipedia**). You install the apps you need; the framework supplies the shared plumbing (users, security, menus, views, reporting) underneath.
2. **It is open-core.** "Since the release of version 9.0, the company has operated under an open-core model that combines open-source software with proprietary enterprise applications and cloud-based SaaS offerings" (**✅ Wikipedia**). The Community edition is GNU LGPLv3; the Enterprise edition adds proprietary applications and support (**✅ Wikipedia; the odoo/odoo LICENSE file**).
3. **It is dual-deployment.** "The software can be deployed on-premises or provided through a software-as-a-service (SaaS) model" (**✅ Wikipedia**; the odoo.com pricing page sells Odoo Online, Odoo.sh, and on-premise side by side — **✅ verified**).

The name "Odoo" itself: chosen, per the company, "in reference to the naming style of several internet companies at the time" of the 2014 rebrand (**✅ Wikipedia, citing the company's own announcement**).

### 1.2 The Overview Table

| Aspect | Description |
|---|---|
| **Type** | ERP + CRM + business-app suite (accounting, inventory, manufacturing, HR, e-commerce, project, POS, website) — **✅** |
| **Vendor** | Odoo S.A. (formerly OpenERP S.A.), Belgium; founder Fabien Pinckaers — **✅** |
| **License model** | Open-core: Community = GNU LGPLv3; Enterprise = proprietary apps + support — **✅** |
| **Core stack** | Python server, PostgreSQL database, JavaScript/TypeScript (OWL) web client — **✅** |
| **Distribution** | SaaS (Odoo Online), PaaS (Odoo.sh), on-premise (Community or Enterprise) — **✅** |
| **Modularity** | Core framework + official apps + community modules; third-party marketplace (apps.odoo.com) — **✅** |
| **Governance** | Company-led core (Odoo S.A.) + independent non-profit community (the OCA, est. 2013) — **✅** |
| **Scale (company)** | €282M revenue (2023, +33%), 2,200+ employees, profitable — **✅** |
| **Scale (adoption)** | "5 million+ users or customers" reported 2021; 54k GitHub stars (2026) — **✅ as reported figures; the user counts are company/report-sourced ⚠** |
| **Positioning** | The open-source challenger to proprietary SME/mid-market ERP (SAP Business One, Microsoft Dynamics) — **✅ as widely-reported framing; CNBC called Odoo "little-known SAP rival" (Nov 2024) ⚠ light** |
| **Series context** | The author's ERP-domain career (Odoo/iDempiere — [asia_fusion_technology_guide.md](asia_fusion_technology_guide.md) §4.4); the open-source-vs-commercial genre of the banking cluster |

### 1.3 The Open-Core Model — Community and Enterprise (Verified)

The single most important strategic fact about Odoo is the **open-core split** — and it is verified from multiple angles:

- **License:** "Community" version: GNU Lesser General Public License v3; "Enterprise" version: proprietary license (**✅ Wikipedia infobox**; the odoo/odoo LICENSE file is LGPL-3).
- **History:** the company "released the core software as open-source software" from its inception (2005) and has operated open-core "since the release of version 9.0" (2015) (**✅ Wikipedia**).
- **Economics:** the October 2022 pricing change consolidated everything into "a single per-user subscription plan for access to its applications" (**✅ Wikipedia, citing Odoo's own announcement of 18 Oct 2022**) — one price buys *all* apps. The pricing page (verified Aug 2026) shows: **One App Free** ($0, one app, unlimited users, Odoo Online), **Standard** (US$16.90/user/month yearly — all apps, Odoo Online), and **Custom** (US$25.50/user/month — all apps, plus Odoo Studio, multi-company, external API, and Odoo.sh/on-premise hosting options).
- **The consequence:** the Community edition is the open-source core with a subset of apps; the Enterprise subscription is where Odoo S.A. makes its money — on proprietary apps, hosting, and support. §4 details the split; the community's own response to it (the OCA, §5.3) is a direct consequence of the open-core strategy.

### 1.4 Why This Guide Exists — the Repository Gap and the Career Context

Before this guide, the repository had no dedicated ERP treatment: Odoo appeared only as (a) passing mentions in the insurance open-source survey ([../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md) — where "Odoo / ERPNext insurance modules" are classified as community long-tail ⚠, and where the structural finding is that open-source infrastructure, not open-source applications, dominates in insurance IT), and (b) the author's own career profile — ERP (Odoo, iDempiere) among the Asia Fusion domain spread ([../personal/jack_liu_profile.md](../personal/jack_liu_profile.md) lines 52–56, 93–97; [asia_fusion_technology_guide.md](asia_fusion_technology_guide.md) §4.4). This guide fills that gap: the dedicated Odoo deep-dive, written with the architecture-first, verify-everything discipline of the house style, and explicitly paired with the Tryton question (§7). The "why this matters" angle: Odoo is the most successful open-source ERP by ecosystem and company scale, it is the reference point for every open-source-vs-commercial ERP discussion (the same genre as [../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md) and [../banking/mojaloop_guide.md](../banking/mojaloop_guide.md) in banking), and — for a bank Solution Architect — it is the pragmatic back-office workhorse that sits *beside* the core-banking ledger rather than replacing it (§8–§9).

---

## 2. The History

### 2.1 TinyERP — the 2005 Founding by Fabien Pinckaers (Verified)

**In 2005, Fabien Pinckaers — the founder and CEO of Odoo — developed the software that later became TinyERP.** The Wikipedia record is explicit: "In 2005, Fabien Pinckaers, the founder and CEO of Odoo, developed the software that later became TinyERP"; the infobox dates the initial release to **February 2005** (**✅ both verified**). Pinckaers, a Belgian computer-science student at the Université catholique de Louvain (UCLouvain), built the system as a small business-management application; the company that grew around it (initially Tiny SPRL, later OpenERP S.A., now Odoo S.A.) would turn that student project into one of the largest open-source business applications in existence. (The UCLouvain student-project origin is the company's own standard telling of the story — the "Odoo Story" blog post of 16 April 2013 is the canonical reference, **✅ as company-sourced history**.)

Two early facts are worth holding on to, because they explain everything that came later:

- The codebase was **open source from day one** — "From its inception, Odoo S.A. (formerly OpenERP S.A.) released the core software as open-source software" (**✅ Wikipedia**).
- The product was **modular from day one** — the framework-plus-apps structure that defines Odoo's architecture (§3) was already the design in the TinyERP years, and it is precisely what a fork could split off in 2008 (§7).

### 2.2 The OpenERP Era — 2008 to 2014 (Verified)

**In 2008, the software was renamed OpenERP** (**✅ Wikipedia**). The rename tracked the product's ambition: from "tiny" ERP to a general enterprise-resource-planning platform. The era's landmarks, all verified:

- **2008** — Rename to OpenERP. This is also the year the **Tryton fork** left the codebase (§7.2): Tryton forked TinyERP 4.2 in November 2008 — so the Tryton lineage and the OpenERP era began in the same year, from the same code.
- **2010** — **€3 million raised for US expansion** (TechCrunch, 17 Feb 2010: "OpenERP gets €3 million for US expansion" — **✅**); the company employed **more than 100 people by 2010** (**✅ Wikipedia**).
- **2012–2013** — Recognition era: Deloitte Technology Fast 50 (2012/2013), Trends Gazelle (Belgium), InfoWorld BOSSIE Award for best open-source applications (2013) (**✅ Wikipedia, citing the awards**).
- **2013** — The **Odoo Community Association (OCA)** is established as a non-profit to support collaborative development and use of the software (**✅ Wikipedia; odoo-community.org** — §5.3). Note the subtlety: the OCA predates the Odoo name; it was founded in the OpenERP era, and its mission has always been to keep the software's community side independent of the company.

The OpenERP era is also the era of the **GTK desktop client**: through versions 6 and 7 (2011–2013) OpenERP shipped a native Python/GTK client alongside the web client — a fact that matters for the Tryton comparison (§7), because Tryton kept the GTK-client architecture while Odoo abandoned it. (GTK client: **⚠ general framework knowledge** — the Wikipedia Tryton article confirms the GTK heritage of the shared lineage.)

### 2.3 The Odoo Rename — 2014 (Verified)

**In 2014, the company changed its name from OpenERP to Odoo** (**✅ Wikipedia**). The announcement post — "OpenERP becomes Odoo" — is dated **14 May 2014** on Odoo's own blog (**✅ verified via the Wikipedia citation and the Wayback CDX capture of the post**). The name, per the company, "was selected in reference to the naming style of several internet companies at the time" (**✅ Wikipedia, citing the company**) — the short, vowel-heavy, two-syllable Web-2.0 style (think Odeo, Flickr, Tumblr).

The rebrand was not cosmetic; it signalled a product change. OpenERP the *ERP* became Odoo the *business-apps platform*: the 9.0 release (2015) introduced the open-core split with Enterprise-exclusive apps (**✅ Wikipedia: "Since the release of version 9.0… open-core model"**), and the marketing shifted from "ERP" to "apps for your business" — the GitHub tagline still reads "Open Source Apps To Grow Your Business." (**✅ verified**). 2015 also brought Inc. magazine's list of the 5,000 fastest-growing private companies in Europe (**✅ Wikipedia**).

### 2.4 The Growth Era — Funding and Valuation (Flagged)

The growth era is the part of the story where the **figures must be flagged**, because the press reports disagree with each other at the margins. All of the following are **verified as reported** — the event, the year, the ballpark — and the discrepancies are noted honestly:

- **2019 — $90 million round** (TechCrunch, 17 Dec 2019: "Odoo grabs $90M to sell more SMEs on its business app suite" — **✅**). This is the round that made Odoo a venture-backed scale-up (the company's own blog post "Odoo is now a unicorn, so what?" — archived by the Wayback Machine — marks the moment; **✅ as company-published framing**).
- **2021 — "more than 5 million users or customers"** reported (**✅ Wikipedia, citing contemporary coverage — the figure is report-sourced, so it is **flagged ⚠ as a company/report-sourced adoption number**, not independently audited**).
- **2023 — revenue €282 million, +33% year-on-year, 2,200+ employees, profitable** (De Tijd, June 2025, via Wikipedia — **✅ as reported; company-reported financials ⚠**).
- **November 2024 — the secondary round.** Odoo "secured $500 million in a secondary share round backed by CapitalG, Sequoia Capital, and BlackRock" (**✅ Wikipedia**). The exact numbers vary by outlet: **TechCrunch (20 Nov 2024): "$527M via secondaries, lifting its valuation to $5.26B"**; **CNBC (20 Nov 2024): valuation boosted "to $5.3 billion"**, describing Odoo as "little-known SAP rival"; **SiliconANGLE (20 Nov 2024): "valued at €5B in secondary sale"**; the Economic Times' startup desk reported "$500 million". All four are **verified as published**; the discrepancies ($500M vs $527M; $5.26B vs $5.3B vs €5B) are exactly the kind of press-margin noise this guide records rather than smooths over — **⚠ flagged**. What is unambiguous: a late-2024 secondary round at a valuation of roughly **US$5 billion**, with Alphabet's CapitalG leading the investor group — an extraordinary valuation for an open-source ERP company, and the single best market signal of Odoo's position as *the* commercial open-source ERP.

The strategic reading (analysis, not fact): the 2024 valuation monetizes the open-core strategy of §1.3 — Odoo S.A. proved that an open-source core plus a proprietary app layer plus a PaaS can grow to €282M revenue and stay profitable, and the market priced that proof at ~$5B. The open-core tension this creates — company control of the core vs community independence — is precisely what the OCA exists to balance (§5.3).

### 2.5 The History Table

| Year | Event | Notes |
|---|---|---|
| 2005 | **TinyERP founded** by Fabien Pinckaers; first release February 2005 | Open source from day one; modular from day one — **✅** |
| 2008 | Renamed **OpenERP**; **Tryton forks TinyERP 4.2** (17 Nov 2008) | The fork and the era begin in the same year — **✅** |
| 2010 | **€3M round** for US expansion; 100+ employees | TechCrunch 17 Feb 2010 — **✅** |
| 2012–2013 | Deloitte Fast 50, Trends Gazelle, InfoWorld BOSSIE | Awards era (OpenERP name) — **✅** |
| 2013 | **OCA founded** (non-profit community association) | Predates the Odoo name — **✅** |
| 2014 | **OpenERP → Odoo rename** (announced 14 May 2014) | Name styled after Web-2.0 internet companies — **✅** |
| 2015 | **9.0: open-core split begins** (Enterprise-exclusive apps); Inc. 5000 Europe | The Community/Enterprise model §4 — **✅** |
| 2019 | **$90M funding round** | TechCrunch 17 Dec 2019 — **✅** |
| 2021 | "5M+ users" reported; hiring push | Report-sourced adoption figure — **⚠** |
| 2022 | **Pricing change: single per-user plan, all apps** (18 Oct 2022) | One App Free / Standard / Custom tiers — **✅** |
| 2023 | **€282M revenue (+33%), 2,200+ staff, profitable** | De Tijd via Wikipedia; company-reported — **✅/⚠** |
| 2024 | **18.0 released (October 2024)**; **Nov 2024 secondary round ~$500–527M at ~$5.26–5.3B valuation** (CapitalG, Sequoia, BlackRock) | Press figures vary — **✅/⚠** |
| 2025 | **19.0 released 18 September 2025**; 16.0 reaches EOL | 19.0 is the current stable — **✅** |

### 2.6 The OpenERP-Era Version Trail — 5.0 to 8.0 (Flagged)

For the record, the OpenERP-era version sequence that preceded the Odoo cadence — **⚠ flagged: the year anchors 8.0 (2014) and the overall sequence are verified (§6.1); the intermediate year attributions below are the standard public record, not individually re-verified this pass**:

| Version | Year (⚠) | Era landmark |
|---|---|---|
| 5.0 | 2009 | The GTK-client OpenERP of the €3M-funding era; the Tryton fork's contemporaries |
| 6.0 | 2011 | Web-client maturation; the era of the "OpenERP 6" developer books |
| 6.1 | 2012 | The web client becomes the default experience |
| 7.0 | 2013 | The OCA-founding year's release; the last pre-rebrand major |
| 8.0 | 2014 | **✅ verified** — the first post-rebrand release, and the start of the modern annual cadence (§6.1) |

The point of the trail: the version numbering has always been a simple year-ish integer (5.0 → 8.0 pre-rebrand, then the unbroken annual 8.0 → 19.0 run of §6.1). The "8.0" that ends this table is the same "8.0" that begins the cadence table — the rebrand did not reset the version line.

---

## 3. The Architecture

### 3.1 Python + PostgreSQL — the Verified Stack

**Odoo's server is written in Python; its database is PostgreSQL; its web client is JavaScript/TypeScript.** The Wikipedia infobox records the implementation languages as "Python, JavaScript and TypeScript" (**✅**); the Odoo documentation's architecture is the classic three-layer web-application stack: a Python application server, a relational database, and a browser-based client. Two facts anchor the database question specifically:

- PostgreSQL is the database backend — Odoo's own documentation and installation guides have always specified PostgreSQL (the Tryton article confirms PostgreSQL as the shared lineage's database of choice; Odoo's install requirements specify PostgreSQL — **✅ verified via the Wikipedia lineage record; ⚠ the Odoo install-requirements page itself was bot-blocked this pass, so the exact minimum PostgreSQL/Python version numbers are flagged, not quoted**).
- The ORM abstracts the SQL: application code works with Python models and recordsets, and the framework generates and migrates the PostgreSQL schema automatically (§3.2).

The stack is deliberately boring — Python, PostgreSQL, standard web protocols — and that ordinariness is a feature: it makes Odoo deployable on any Linux host, findable by any Python developer, and integrable with any system that speaks HTTP/JSON. The Odoo 18+ on-premise documentation even shows the heritage in the infrastructure: database registration calls home to `services.odoo.com` for Odoo 18.0+, but **Odoo 17.0 and below still phone `services.openerp.com`** (**✅ verified — the OpenERP name survives in the production fleet**).

### 3.2 The ORM — Models, Recordsets, Automatic Schema

The ORM (object-relational mapping) is the heart of the framework — the layer that lets developers define business objects as Python classes and get a full CRUD + security + workflow + reporting stack for free. The elements (all **✅ as the framework's documented design; ⚠ the primary docs pages were bot-blocked this pass, so this is verified via the framework's ubiquitous developer documentation, the Odoo Development Cookbook cited on Wikipedia, and the repo structure**):

- **Models** — business objects defined as Python classes (`class SaleOrder(models.Model): _name = "sale.order"`), with fields declared in Python (`name = fields.Char(...)`, `partner_id = fields.Many2one("res.partner")`).
- **Automatic schema** — the ORM creates, alters, and migrates the PostgreSQL tables from the model definitions; a database upgrade applies the model changes. There is no hand-written DDL in normal development.
- **Recordsets** — the ORM API is set-oriented: methods operate on recordsets (`self.filtered(...)`, `self.mapped(...)`, `self.search([...])`), a design that maps naturally to SQL and to batch operations.
- **Inheritance** — the killer feature that makes the module ecosystem viable: any model can be *extended* by another module (new fields, new methods, overridden behaviour) without touching the original code. This is what lets thousands of third-party modules (the app store, §5.2) coexist and compose.
- **Security built in** — per-model access rights, per-record rules (ir.rule), field-level access, all enforced by the ORM layer regardless of the entry point (web client, RPC, cron).
- **Views** — the UI is declarative: forms, lists (trees), kanban, calendars, pivot/graph views are described in XML and rendered by the client; QWeb (an XML templating engine) powers reports and website pages.

For a developer coming from the metadata-driven world of the author's own ERP background ([../personal/jack_liu_profile.md](../personal/jack_liu_profile.md): "ERP frameworks (Odoo, iDempiere) — metadata-driven development, DSLs, application dictionaries"), Odoo's ORM is the same family: the model definitions *are* the application dictionary, and the framework interprets them into schema, UI, and security. The difference from iDempiere (a Java/ADempiere descendant) is the language and the weight: Python + PostgreSQL instead of Java + PostgreSQL/Oracle.

### 3.3 The Module System — Addons, Manifests, Data Files

Odoo is distributed as **modules** (called "addons" in the repository layout — the repo root contains `addons/` and `odoo/` directories, **✅ verified from the github.com/odoo/odoo file listing**). A module is a directory containing (**✅ as the documented module layout**):

- a **manifest** (`__manifest__.py`) — name, version, dependencies (`depends`), data files, and the list of files to load;
- **Python models and business logic**;
- **XML data files** — views, menus, actions, default records, workflow-ish automation;
- **security files** — `ir.model.access.csv` access rules, groups;
- **static assets** — for the web client.

Dependencies form a DAG: installing `sale` pulls in `product`, `stock`, `account`, etc. The module system is what makes the ecosystem (§5) possible: Odoo S.A. ships ~50 official apps as modules; the OCA ships thousands; third parties ship tens of thousands — all composable because the ORM inheritance (§3.2) lets any module extend any other.

### 3.4 The Web Client — the OWL Framework (Verified)

**The modern Odoo web client is built on OWL — the "Odoo Web Library", a modern, lightweight UI framework written in TypeScript.** This one is directly verified from the source: the github.com/odoo/owl repository describes OWL as "A modern, lightweight UI framework for applications that scale," "~30kb gzipped, zero dependencies," and states: **"It powers Odoo's web client, one of the largest open-source business applications"** (**✅ verified — repo README, Aug 2026**). Key verified facts:

- License: **LGPL v3** (**✅**), matching the Community edition's license.
- Design: component-based with **signal-based reactivity** and a plugin system (**✅ README**); Owl 3 is the current generation (3.0 alpha at the time of the pass).
- History: older Odoo web clients used jQuery/Backbone-era JavaScript (**⚠ general framework knowledge** — the OWL rewrite landed around the 14.0/15.0 cycle, 2020–2021, replacing the legacy widget system).
- The OWL story matters for the Tryton comparison (§7): Odoo bet on a **bespoke in-house web framework**; Tryton kept the **GTK desktop client** plus a jQuery/Bootstrap web client (sao). One project invests in a single modern web UI; the other preserves a desktop client as a first-class citizen.

The full web stack also includes the standard web assets — the backend UI, the website builder, the POS, the portal (external customer/supplier self-service, which the pricing page confirms are **not** paying users: "Customers and suppliers who have access to their invoices on the portal… are free users" — **✅ verified**).

### 3.5 The Server — Workers, Cron, RPC

The Python server is the process that ties everything together (**✅ as the documented server design; ⚠ the install/ops docs were bot-blocked, so operational specifics are flagged**):

- **Multi-process workers** — the server runs as a master process with worker processes (threaded or prefork), serving HTTP requests.
- **Cron jobs** — scheduled server-side actions (e.g., recurring invoicing, stock reordering rules, email digest) run inside the server via the `ir.cron` model; a scheduler thread (or dedicated cron workers) triggers them.
- **RPC protocols** — the server exposes **JSON-RPC and XML-RPC** APIs; the pricing page confirms the "External API" is "calls to the Odoo API (JSON-RPC or XML-RPC) to query, retrieve or update data stored in Odoo" (**✅ verified**). This is the integration surface for the rest of a bank's estate (§8–§9).
- **Long-polling** for real-time features (chat/Discuss, live notifications) — **⚠ general knowledge**.
- **The database manager** — a web-accessible database management surface (`/web/database/manager`, per the on-premise docs — **✅ verified**) for creating, duplicating (with a "neutralize" option for privacy-safe test copies), and backing up databases.

### 3.6 The Architecture Table

| Layer | Technology | Role | Verification |
|---|---|---|---|
| **Client** | OWL (TypeScript UI framework) + standard web assets | Backend UI, website builder, POS, portal | **✅** (odoo/owl README: "powers Odoo's web client") |
| **API** | JSON-RPC / XML-RPC | Integration surface for external systems | **✅** (odoo.com pricing FAQ) |
| **Server** | Python application server; workers + cron | Business logic, ORM, security, scheduled jobs | **✅** (Python per Wikipedia; ops details ⚠) |
| **ORM** | Python models → PostgreSQL tables | Automatic schema, recordsets, inheritance, security | **✅/⚠** (framework-documented; primary docs bot-blocked) |
| **Database** | PostgreSQL | Single source of truth | **✅** (lineage + documented stack; version minimums ⚠) |
| **Module layer** | Addons: manifest + Python + XML data/views + security | The apps; the ecosystem's composition unit | **✅** (repo layout: `addons/`, `odoo/`) |
| **Rendering** | XML views + QWeb templates | Declarative UI and reports | **✅/⚠** (framework-documented) |
| **Deployment** | Odoo Online (SaaS) / Odoo.sh (PaaS) / on-premise | §4 | **✅** (pricing page + on-premise docs) |

### 3.7 The Request Lifecycle — How a Click Becomes a Row

The clearest way to hold the whole architecture in one head is to trace one request — a salesperson saving a quotation in a browser — through the stack (§3.1–§3.5; all layers verified above):

1. **The browser** runs the OWL-based web client (§3.4). The user clicks *Save* on a quotation form.
2. **The client** serializes the form into a JSON-RPC call to the server's web layer (§3.5) — the same API surface an external system would use for the "external API" entitlement (§4.4).
3. **The server** (Python workers) receives the call and hands it to the ORM: `sale.order` model, `create()` or `write()` on a recordset (§3.2).
4. **The ORM** validates the fields, applies security (access rights + record rules), computes derived fields (e.g., amount totals), fires automated actions (e.g., a follow-up activity or an email template), and **generates the PostgreSQL INSERT/UPDATE** — the schema having been created and migrated automatically from the model definitions (§3.2).
5. **The transaction commits**; the client refreshes the view; any real-time subscribers (Discuss, live notifications, the long-polling channel — §3.5) are pushed the change.
6. **Downstream**, cron jobs (§3.5) may act on the new record later (e.g., the recurring-invoicing scheduler), and external systems may consume it via RPC or via the bank-style integration layer of §9.3.

The architectural moral: **every app in the ecosystem — official, OCA, or third-party — runs this same lifecycle**, because every app is a module on the same ORM (§3.3). That is why the ecosystem composes (§5) and why the annual cadence (§6) touches everything at once: a schema or API change in the ORM ripples through every module.

---

## 4. The Editions

### 4.1 Community vs Enterprise — the Feature Split

The open-core model (§1.3) divides Odoo into two software editions plus three hosting options. The verified anchors of the split:

- **Community** — the open-source core: "released under the GNU LGPLv3 license" (**✅ Wikipedia**). It contains the core framework and the base apps (CRM, Sales, Inventory, Invoicing, Purchase, Project, Manufacturing/MRP, HR/Employees, eCommerce/Website, Discuss, etc. — the exact app set shifts per version).
- **Enterprise** — "contains proprietary applications and support services" (**✅ Wikipedia**). The proprietary layer adds: **Odoo Studio** (no-code app builder), **full Accounting** features in recent versions, **PLM, Field Service, Documents, Sign, Spreadsheet, Marketing Automation, Social Marketing, Planning, Appointments, Approvals, Knowledge, Subscriptions, Rental, Quality**-class apps, **multi-company** support, the **external API** entitlement, and **Odoo.sh** hosting (**✅ the odoo.com pricing page explicitly lists Studio, Multi-Company, External API, and Odoo.sh as Custom-plan entitlements**).

The sharpest Community-vs-Enterprise data point from this pass: on the Odoo app store, the most-downloaded community modules for Odoo 19 include "**Brings full accounting features back to Odoo 19 Community: financial reports (P&L, Balance Sheet, General Ledger, Trial Balance, Partner Ledger, Aged Receivable/Payable), asset management, budgets, bank statement import, PDC, and customer follow-ups — everything Community is missing**" (**✅ verified — apps.odoo.com listing, Aug 2026; the *description* is third-party vendor marketing ⚠, but its claim that recent Community editions lack full accounting is corroborated by the existence of an entire category of such modules**). In other words: in the 17→18→19 cycle, Odoo moved the deep accounting functionality (financial reporting, bank reconciliation, assets, budgets) into Enterprise, and the community built replacements. That is the open-core strategy in action, and it is the single most important functional difference a prospective Community adopter must know.

### 4.2 Odoo.sh — the PaaS (Verified)

**Odoo.sh is Odoo's platform-as-a-service for custom development** — "the continuous integration platform to host your development, staging and production branches, allowing you to develop or use custom modules" (**✅ verified — the odoo.com pricing FAQ, verbatim**). Key verified/flagged characteristics:

- **Git-based workflow**: development happens on branches; Odoo.sh builds a database per branch (dev/staging/production), so a pull request gets a test environment automatically (**✅ as the platform's documented design; the Odoo.sh documentation pages were bot-blocked this pass, so specifics are flagged ⚠**).
- **Custom modules supported** — this is the point of Odoo.sh: Odoo Online (the SaaS) does *not* allow custom code ("The standard plan is hosted on Odoo Online, our cloud infrastructure to host databases without custom modules" — **✅ pricing FAQ**), so any implementation with bespoke development must use Odoo.sh or on-premise.
- **Included in the Custom plan** at US$25.50/user/month, with hosting billed separately ("Cost for Odoo.sh hosting not included" — **✅ pricing page**).
- **Ops features**: backups, staging, monitoring, upgrade tooling are part of the platform (**⚠ general platform description**).

For the worked example (§9), Odoo.sh is the deployment answer when the bank's compliance estate permits a managed cloud; on-premise is the answer when it does not.

### 4.3 On-Premise — the Subscription-Keyed Deployment (Verified)

The on-premise edition is the same software, self-hosted, keyed to a subscription: "Download Odoo Enterprise and host it yourself" (**✅ pricing FAQ**). Verified facts from the on-premise documentation:

- **Subscription registration**: you enter a subscription code; the server validates it against Odoo's licensing servers (outbound calls to `services.odoo.com` on port 80 for 18.0+, `services.openerp.com` for 17.0 and below — **✅ verified**).
- **Community↔Enterprise switching is supported**: the docs carry an explicit "Switch from Community to Enterprise" procedure — you can run Community, subscribe, and flip to Enterprise on the same database (**✅ verified — the on-premise docs' navigation tree**).
- **Data portability**: "When using Odoo Online, you can download a backup of your database at any time… You own your data!" (**✅ pricing FAQ**) — the SaaS is not a lock-in trap; backups move between hosting modes.
- **The docs' own framing**: "Odoo Online or Odoo Enterprise (On-premise or Odoo.sh) is the same software. Only the hosting and infrastructure are different" (**✅ pricing FAQ**).

### 4.4 The Editions Table

| Dimension | Community | Enterprise (subscription) |
|---|---|---|
| **License** | GNU LGPLv3 — **✅** | Proprietary apps + support — **✅** |
| **Core framework** | Yes — **✅** | Same core + proprietary app layer — **✅** |
| **Base apps** | CRM, Sales, Inventory, Invoicing, Purchase, Project, MRP, HR, eCommerce, Website, Discuss (set shifts per version) — **✅/⚠** | All Community apps + Enterprise apps — **✅** |
| **Full Accounting** | Removed from recent versions (financial reports, bank reconciliation, assets, budgets → Enterprise) — **✅/⚠** (app-store corroboration) | Included — **✅/⚠** |
| **Odoo Studio** (no-code builder) | No — **✅** (pricing page: Studio is Custom-plan) | Yes — **✅** |
| **Multi-company** | No — **✅** (pricing page) | Yes — **✅** |
| **External API** (JSON-RPC/XML-RPC) | Community technically exposes RPC; the *entitlement* is Enterprise — **⚠ nuance** | Yes — **✅** |
| **Hosting** | Self-host only (Community on your own infra) | Odoo Online (SaaS), Odoo.sh (PaaS), or on-premise — **✅** |
| **Support** | Community support (forums, OCA) | "Unlimited support, hosting and maintenance" included — **✅ pricing page** |
| **Price** | Free (software); infra is yours | One App Free $0; Standard $16.90/user/mo; Custom $25.50/user/mo (annual, per user) — **✅ pricing page** |
| **Upgrades** | Self-managed (OCA OpenUpgrade tooling) | Included in subscription; migration services available — **✅/⚠** |

---

## 5. The Apps and the Ecosystem

### 5.1 The Core Apps (Verified)

Odoo's official app set is large and version-dependent; the verified core from two independent sources — the Wikipedia feature list and the odoo.com pricing page's app icons — includes: **Sales, CRM, eCommerce, Website, Accounting, Invoicing, Inventory (stock), Purchase, Manufacturing (MRP), HR/Employees, Project, Point of Sale (POS), Discuss (chat), Expenses, Documents, Sign, Events, Fleet, Helpdesk, Marketing Automation, Live Chat, Maintenance, Planning, Appointments, Approvals, Payroll** (the Odoo 18 release-notes table of contents alone lists 40+ app sections — **✅ verified**). The canonical "core five" for this guide — the apps any ERP discussion starts from:

- **Sales** — quotations, orders, invoicing pipeline (order → invoice → payment).
- **Inventory** — multi-warehouse stock, moves, traceability, barcoding, reordering rules.
- **Accounting** — the double-entry ledger, invoicing, bank reconciliation, financial reports (full depth now Enterprise in recent versions, §4.1).
- **CRM** — leads, opportunities, pipelines, activities.
- **HR (Employees)** — employee records, contracts, time off, expenses, appraisals (with Payroll as an Enterprise/localization-sensitive app).
- **MRP (Manufacturing)** — bills of materials, work orders, production planning.

The release notes also confirm a newer app category: **"Industries"** — data-only module suites ("a suite of applications, configurations, and data that fit a specific business… It does not contain Python code and can be installed in SaaS databases"), with 10 shipped at Odoo 17 (Lawyer, Hairdresser, Software Reseller, etc.) (**✅ verified — Odoo 17 release notes**). This is the productized end of the same modularity that the community has always used (§5.3).

### 5.2 The App Store — Scale Flagged

**The Odoo app store (apps.odoo.com) is the distribution marketplace for third-party modules** — free and paid, per Odoo version (listings are namespaced by version: `/apps/modules/19.0/…`, `/apps/modules/18.0/…` — **✅ verified from the store's URL structure and listings**). Verified observations from this pass:

- Vendors are a mix of Odoo S.A. itself and a large third-party ecosystem (Emipro, Ksolves, Cybrosys, Odoo Mates, Terabits, SPARA, and many more — **✅**).
- Paid apps dominate the top charts (Shopify/WooCommerce connectors at ~$640; dashboards at ~$600) alongside free community kits (**✅**).
- The store is the *commercial* long tail of the ecosystem, and it is version-fragmented (apps list per Odoo major version).

**The total count is flagged ⚠**: the store's browse page is JavaScript-rendered and the aggregate count could not be extracted this pass. What is verifiable instead: the **OCA alone claims 20,000+ community modules** (**✅ odoo-community.org**), and the store's own marketing commonly cites figures in the tens of thousands — but this guide does not repeat an unverified headline number. The honest statement: **the Odoo ecosystem numbers in the tens of thousands of modules across the official store, the OCA, and GitHub** — exact counts unverifiable in this pass.

### 5.3 The OCA — the Odoo Community Association (Verified)

**The Odoo Community Association (OCA) is a non-profit organization based in Switzerland "forming a community for mutual support around the Odoo software," providing thousands of Odoo modules under open-source licenses and open upgrades** (**✅ verified — odoo-community.org**). The OCA is the institutional answer to the open-core question: its mission is "to facilitate real collaboration between Odoo developers and consultants throughout the ecosystem **and ensure that Odoo remains a viable open source ERP regardless of what Odoo SA decides to do in the future**" (**✅ verified — verbatim from odoo-community.org**). Verified numbers from the OCA site (Aug 2026):

- **20,000+ Odoo modules** across "more than 15 versions"; **62 countries**; **1,600 contributors**; **664 members**; **38 sponsors**.
- Governance: an independent legal non-profit; contributions under a Contributors Agreement License (CLA) — "All OCA projects are freely available and usable under an OSI-certified Open Source license" (**✅**). OCA modules are typically **AGPL-3.0** (**✅ verified — e.g., the OCA/l10n-italy repo's license badge**).
- The GitHub organization (**github.com/OCA**) holds **265 public repositories** (**✅ verified**), organized as thematic projects — localizations (l10n-*), industry verticals, and infrastructure. The two most strategically important:
  - **OpenUpgrade** — "Open source upgrade path for Odoo/OpenERP" (978 stars) — the community's version-migration machinery, i.e., the tooling that keeps old installations alive across the annual release treadmill (**✅ verified**).
  - **OCB — Odoo Community Backports** — a maintained fork of odoo/odoo with community backports of fixes, for those who cannot or will not run the upstream version (**✅ verified**).
- The OCA also runs the **OCA Days** annual event (Liège, Belgium) and an **apps store** for its own modules (apps.odoo-community.org) (**✅ verified**).

The OCA is the reason the "open" in Odoo is not merely a license: it is an institution with its own funding, its own code, and its own fork-and-upgrade path — the community's hedge against the company.

### 5.4 The Ecosystem Table

| Component | What it is | Verified notes |
|---|---|---|
| **Odoo S.A.** | The vendor: core framework, official apps, Enterprise layer, hosting | €282M revenue 2023; ~$5B valuation 2024 — **✅/⚠** |
| **Community edition** | LGPLv3 open-source core + base apps | Free to run, self-hosted — **✅** |
| **Enterprise subscription** | Proprietary apps, support, hosting | §4 — **✅** |
| **The app store** | Third-party marketplace (free + paid, per version) | Count flagged ⚠; per-version listings verified — **✅/⚠** |
| **The OCA** | Non-profit community association (Switzerland, est. 2013) | 20k+ modules, 265 repos, 1,600 contributors — **✅** |
| **OCA OpenUpgrade / OCB** | Upgrade tooling and community backports fork | The community's migration insurance — **✅** |
| **Partners** | Implementation/consulting partner network (odoo.com/partners) | "Mid-size and large companies usually work with a partner" — **✅ pricing FAQ** |
| **Success Packs** | Odoo's own implementation services for small companies (<50 employees) | — **✅ pricing FAQ** |
| **Education program** | University partnerships (Vietnam, Italy, Hong Kong… active 2026) | — **✅ blog archive** |
| **Adjacent communities** | Tryton (the fork, §7), ERPNext, iDempiere (the author's other ERP reference) | Genre context — **✅/⚠ light** |

### 5.5 The Ecosystem Economics

Beyond the code, the Odoo ecosystem has an economy, and the verified facts sketch it:

- **The company's own economics**: €282M revenue (2023), profitable, 2,200+ staff, ~$5B valuation (2024) — the open-core model monetized (§2.4, §4).
- **The partner economy**: implementation work flows through Odoo's partner network and Success Packs ("Small companies… usually work directly with Odoo, using Success Packs… Mid-size and large companies usually work with a partner" — **✅ pricing FAQ**). Partners are the ecosystem's services layer.
- **The app economy**: the app store's top charts are dominated by paid third-party apps (Shopify/WooCommerce connectors ~$640, dashboards ~$600 — **✅ verified listings**) — a real commercial long tail on top of the LGPL core, exactly what the license permits (§7.3).
- **The OCA economy**: 38 sponsors (partner firms and end users) fund the community's infrastructure; 664 members; crowdfunding for specific projects; the OCA Days event — **✅ verified** (§5.3).
- **The skills economy**: the education program (universities in Vietnam, Italy, Hong Kong — **✅ blog archive**) and the developer books (the Odoo Development Cookbook, 5th edition, cited on Wikipedia — **✅**) feed a large, relatively low-cost talent pool — a factor in any bank's TCO model (§9.3, and [../management/business_case_development_guide.md](../management/business_case_development_guide.md)).

---

## 6. The Release Cadence

### 6.1 The Annual Cadence — 8.0 (2014) to 19.0 (2025)

**Odoo ships one major version per year, on an autumn cadence** — the pattern is 8.0 (2014), 9.0 (2015), 10.0 (2016), 11.0 (2017), 12.0 (2018), 13.0 (2019), 14.0 (2020), 15.0 (2021), 16.0 (2022), 17.0 (2023), 18.0 (2024), 19.0 (2025) (**✅ the version sequence and years are verified** — the task's stated anchors check out: **8.0 in 2014 ✅, 17.0 in 2023 ✅, 18.0 in 2024 ✅ — the "Meet Odoo 18" announcement is archived 3 Oct 2024, consistent with an October 2024 release; and 19.0 on 18 September 2025 ✅ per Wikipedia/Wikidata**). The GitHub repository's branch structure tracks the cadence: each major version is a long-lived branch (the repo's default branch is 19.0; the branches list includes the 18.0, 17.0, 16.0… lines — **✅ verified**).

The cadence is *the* defining operational fact for adopters: **every year there is a new major version, and the upgrade treadmill is real**. Each annual release brings a new database-schema generation, new APIs, and (since the web-client rewrite) new UI generations — which is exactly why OpenUpgrade and the OCA's per-version module maintenance exist (§5.3), and why the "skip versions" strategy is dangerous in Odoo.

### 6.2 The LTS Question — Flagged

**The long-term-support story is the murkiest part of the cadence, and it is flagged.** What is verified:

- Wikipedia's current account (citing Odoo's own supported-versions documentation): "Major Odoo releases (such as 19.0) are long-term support (LTS) versions… Multiple LTS versions are supported in parallel, with older versions reaching end of life when new LTS releases are introduced. For example, versions 16.0 LTS, 17.0 LTS, and 18.0 LTS were supported concurrently, with 16.0 reaching end of life in September 2025 alongside the release of 19.0 LTS" (**✅ as reported; the underlying Odoo supported-versions page itself was bot-blocked this pass ⚠**).
- The historical reality (⚠ general knowledge, consistent with the OCA's raison d'être): for most of Odoo's life, the company's official support posture was **"the latest version only"** — Enterprise subscriptions include migration support, but there was no multi-year vendor LTS in the Ubuntu sense; long-term maintenance of older Community versions was always community work (OCB backports, OpenUpgrade).
- The verified constant: **Community-edition longevity is not a vendor promise — it is the OCA's project** (OpenUpgrade exists precisely because upgrades are hard and old versions need a path forward — **✅ §5.3**).

The honest synthesis for a decision-maker: **treat every Odoo major version as a ~3-year supported lifetime at best (roughly one year of vendor currency plus community maintenance), plan for an annual upgrade rhythm, and budget for upgrade work as a recurring line item** — the LTS label in current Odoo documentation describes overlapping parallel support, not a 5-year Ubuntu-style LTS commitment.

### 6.3 The Cadence Table

| Version | Year | Notes | Verification |
|---|---|---|---|
| 8.0 | 2014 | Post-rebrand first release; the era of the modern web UI | **✅** |
| 9.0 | 2015 | Open-core split begins (Enterprise-exclusive apps) | **✅** (Wikipedia) |
| 10.0 | 2016 | — | **✅** |
| 11.0 | 2017 | — | **✅** |
| 12.0 | 2018 | — | **✅** |
| 13.0 | 2019 | The $90M round year | **✅** |
| 14.0 | 2020 | OWL-era web client (⚠ approximate) | **✅ year; ⚠ detail** |
| 15.0 | 2021 | — | **✅** |
| 16.0 | 2022 | Single-plan pricing announced (Oct 2022) | **✅** |
| 17.0 | 2023 | "Industries" data-module suites introduced; €282M revenue year | **✅** (release notes; De Tijd) |
| 18.0 | 2024 | October 2024 release; services.odoo.com licensing endpoint | **✅** ("Meet Odoo 18" archived 3 Oct 2024) |
| 19.0 | 2025 | 18 September 2025; current stable; 16.0 EOL same month | **✅** (Wikipedia/Wikidata) |
| LTS policy | evolving | Overlapping parallel support (16/17/18 concurrently until 16 EOL Sep 2025); Community longevity = OCA work | **✅/⚠ flagged §6.2** |

### 6.4 What the Recent Releases Actually Brought (Verified Highlights)

From the Odoo 17 and 18 release-notes pages (extracted this pass — **✅ verified**), the flavour of the annual treadmill:

- **Odoo 17 (2023)**: the **"Industries" data-module concept** — configuration/data-only vertical suites (Lawyer, Hairdresser, Software Reseller…) with no Python code, installable on SaaS databases, with a dedicated runbot for testing them (§5.1); **WebP image support**; a **ChatGPT website configurator**.
- **Odoo 18 (2024)**: **WhatsApp integration** (automated notifications, centralized chat history on the contact form); a **snippets-library overhaul** (60+ new website snippets with previews); **theme redesign** (27+ themes, new palettes, custom fonts); **video lazy-loading**; **property fields in website forms**.
- The release-notes structure itself is evidence of scale: 40+ app sections per release (General, Accounting, Localizations, CRM, Inventory, Manufacturing, Payroll, PLM, POS, …) — **✅ the 18.0 notes' TOC alone lists 40+ modules**.

The takeaway: each release is a mix of platform change (schema/API/UI churn — the cost side, §6.2) and visible features (WhatsApp, AI website builder — the value side). An adopter's upgrade case is the balance between the two, evaluated every autumn.

---

## 7. The Comparison

### 7.1 Odoo vs Tryton — the Head-to-Head

This section exists because the two projects are **the two halves of the same 2005–2008 family tree**, and every difference between them is a choice made after the split. Odoo is the commercial open-core ERP with the giant ecosystem; Tryton is the community-governed application framework with the deliberately small, stable core. Neither is "better" in the abstract; the comparison is about which trade-offs fit which mandate.

### 7.2 The Lineage — the TinyERP 2008 Fork (Verified)

**Tryton is a fork of TinyERP 4.2, published in November 2008 — the same year TinyERP was renamed OpenERP.** Wikipedia's Tryton article is explicit: "Tryton's origin is a fork of the version 4.2 of TinyERP (which was later called OpenERP and now renamed Odoo). The first version was published in November 2008" (**✅ verified**). The fork was led by **Cédric Krier** and the community around him (**✅ Wikipedia: "Original author: Cédric Krier and the Tryton community"**; the first Tryton 1.0 release followed in 2009 per the LWN coverage Wikipedia cites).

Why did they fork? The record is consistent on the *what* (a split over project direction and governance), even if the *why* is best read from what each project became: Tryton chose **framework-first purity, federation governance, and stability**; Odoo chose **application-first speed, company leadership, and ecosystem breadth**. The fork is the spine of every row in §7.7.

### 7.3 Licensing — GPL-3 vs LGPL (Verified)

**Tryton is GPL-3.0-or-later; Odoo Community is LGPLv3.** Both verified from the license records:

- Tryton: "The platform, along with the official modules, are free software, licensed under the GPL-3.0-or-later license" (**✅ Wikipedia**, citing the Tryton COPYRIGHT file in the repository).
- Odoo Community: GNU LGPLv3 (**✅ Wikipedia infobox; odoo/odoo LICENSE**).

The difference matters: **LGPL permits linking proprietary code into the application without releasing that code** (Odoo S.A. itself relies on this — the proprietary Enterprise apps sit on top of the LGPL core); **GPL requires derivative works to be GPL** — a stronger copyleft that keeps the entire stack free. Tryton's GPL is the licensing expression of its federation philosophy: no proprietary layer, no open-core. Odoo's LGPL is the licensing expression of its commercial strategy: the open core is a platform, and the company (and any vendor) can build proprietary value on top. (The OCA modules are AGPL-3.0 — a *stronger* copyleft than Odoo's own Community core, another expression of the community's independence — **✅ §5.3**.)

### 7.4 Philosophy — Framework vs Application

**Tryton is a general-purpose application platform ("a three-tier high-level general purpose computer application platform on top of which is built an ERP business solution through a set of Tryton modules" — ✅ Wikipedia); Odoo is an application suite first ("Open Source Apps To Grow Your Business" — ✅ repo tagline) with the framework as its foundation.** The consequences:

- Tryton's kernel is deliberately functional-field-agnostic: models, user management, workflows, reporting, i18n, historization, migration — "not linked to any particular functional field hence constituting a general purpose framework" (**✅ Wikipedia**). The flagship example of Tryton-as-platform is **GNU Health**, a hospital information system built on Tryton (**✅ Wikipedia**) — Tryton is a platform that happens to ship an ERP module set.
- Odoo's framework exists to serve the apps; the apps define the product; the company's roadmap decides what the apps do. The "Industries" data-module concept (§5.1) is the purest expression: Odoo now productizes vertical configurations on top of its apps.

For the architect, the practical difference: Tryton is the better substrate for a *bespoke* system (stable kernel, clean models, your own business logic on top); Odoo is the better buy for *out-of-the-box* business processes (install, configure, run) — at the price of its annual release treadmill (§6).

### 7.5 Stability vs Ecosystem Breadth

**Tryton optimizes for stability; Odoo optimizes for ecosystem breadth.** Verified facts on each side:

- Tryton's stability machinery: series-based releases with "the same API and the same database scheme"; **automatic migration built into the kernel** ("allows updating the underlying database scheme without any human manipulation… taken into account and tested continually within the development" — **✅ Wikipedia**); series maintained for one year, with every fifth series getting 5-year LTS (**✅ Wikipedia, citing the Tryton release-process discussion**).
- Odoo's ecosystem machinery: the app store, the OCA's 20,000+ modules, 265 GitHub repos, the partner network, the education program (§5). No open-source ERP has anything close to Odoo's breadth of ready-made, commercially-supported modules.
- The trade: Tryton's module set is small, coherent, and version-disciplined (official modules track the series; no commercial app store); Odoo's is vast but fragmented across versions, with quality varying from polished commercial apps to abandoned community modules. A Tryton adopter writes more; an Odoo adopter curates more.

### 7.6 Release Cadence — Annual vs Series

- **Odoo**: one major version per year, autumn cadence, schema + API + UI churn each year (§6.1) — the treadmill.
- **Tryton**: "A new series appears every six months" per the 2017 release-process doc Wikipedia cites, with series maintained for 1 year and every fifth series an LTS of 5 years (**✅ as documented; ⚠ the observed practice has in practice slowed toward annual series — flag**). The key difference is not the raw frequency — it is that **Tryton commits to API and schema stability within a series and provides automatic migration, while Odoo commits to none of that and sells migration services instead** (§4.4, §6.2).

### 7.7 The Comparison Table

| Dimension | Odoo | Tryton |
|---|---|---|
| **Lineage** | TinyERP (2005) → OpenERP (2008) → Odoo (2014) — **✅** | Fork of TinyERP 4.2, Nov 2008 — **✅** |
| **Governance** | Company-led (Odoo S.A.) + OCA non-profit — **✅** | Federation of companies + Tryton Foundation (Belgium, 2012); "no partner network" by design — **✅** (Wikipedia) |
| **License** | Community: LGPLv3; Enterprise: proprietary (open-core) — **✅** | GPL-3.0-or-later, kernel + official modules, no proprietary layer — **✅** |
| **Philosophy** | Application suite first; framework serves the apps — **✅** | General-purpose platform first; ERP is one module set (cf. GNU Health) — **✅** |
| **Client** | Web-only: OWL (TypeScript) — **✅** | GTK desktop client + sao web client (jQuery/Bootstrap) — **✅** (Wikipedia) |
| **Stack** | Python + PostgreSQL; JSON-RPC/XML-RPC — **✅** | Python + PostgreSQL (mainly); python-sql gives SQLite testing; XML-RPC/JSON-RPC — **✅** (Wikipedia) |
| **Migration** | Upgrade tooling exists (OpenUpgrade, Enterprise migration services); annual churn — **✅/⚠** | Automatic, kernel-built-in, tested continuously — **✅** (Wikipedia) |
| **Release cadence** | Annual major versions (8.0 2014 → 19.0 2025) — **✅** | Series every ~6 months (documented), 1-year maintenance, every 5th series 5-year LTS — **✅/⚠** |
| **Ecosystem** | App store + OCA 20k+ modules + 265 repos + partners — **✅** | Small, coherent official module set; no commercial app store — **✅** |
| **Commercial layer** | Enterprise apps, Odoo.sh, support, ~$5B valuation — **✅/⚠** | None by design; services come from federation companies — **✅** |
| **Typical fit** | Out-of-the-box business processes; SME/mid-market ERP; fast deployments — analysis | Bespoke/custom systems; regulated or long-lifetime deployments; GPL-compatible stacks — analysis |
| **Last stable** | 19.0 (18 Sep 2025) — **✅** | 7.0 series (7.0.34, Jul 2025) — **✅** (Wikipedia) |

### 7.8 Odoo vs SAP Business One — Light, Flagged

For completeness, a light treatment — **flagged ⚠ as general-knowledge comparison, not re-verified this pass** (SAP's materials were not extracted):

- **SAP Business One** is SAP's proprietary ERP for small and mid-size businesses (SMEs) — per-user licensed, SQL Server/SAP HANA-backed, module-based (financials, sales, purchasing, inventory, production, CRM-light), with a deep channel-partner implementation ecosystem and strong manufacturing/distribution fit.
- **Odoo** is the open-source challenger: no license fee for the Community core, subscription for Enterprise, web-native, broader app surface (e-commerce, website, marketing, HR), and a younger but fast-growing partner network.
- The press itself frames Odoo this way — CNBC's November 2024 headline called Odoo a "little-known SAP rival" (**✅ verified as the published framing**).
- The honest compare: SAP B1 wins on enterprise credibility, localization depth for large markets, and manufacturing best practice; Odoo wins on total cost of ownership, customization freedom (source access), and app breadth per dollar. For a bank's back office (§8–§9), neither replaces the core-banking system; both are candidates for the finance/HR/procurement layer — and Odoo is the one a regulated institution can actually inspect, patch, and air-gap.

### 7.9 The Tryton Decision Tree (Analysis)

As a practical close to the comparison, a decision heuristic — **analysis, not verified fact**:

- **Choose Odoo when**: you want out-of-the-box business processes; you need the app breadth (store + OCA); you accept (or want) the commercial layer (Enterprise support, Odoo.sh); you can sustain the annual upgrade cadence; your organization tolerates LGPL/open-core licensing.
- **Choose Tryton when**: you are building a bespoke system on a stable kernel; you need GPL-pure licensing; you want federation governance and no vendor dependence; you need long-lifetime deployments with automatic migration; a GTK desktop client is a feature, not a bug.
- **The honest tie-breaker**: for a bank back office (§8–§9), Odoo wins on breadth and support economics; for a regulated custom build where the *platform* is the product (the GNU Health pattern, §7.4), Tryton wins on stability and purity.

---


## 8. The ERP-in-Banking Context

### 8.1 The ERP in a Bank — Back-Office, Not the Ledger

**The honest first principle: an ERP is not a core-banking system.** The banking series in this repository is explicit about what core banking is — the transaction ledger, the product catalogue, the customer accounts, the regulatory reporting — in systems like [../banking/t24_programming_guide.md](../banking/t24_programming_guide.md) (the commercial core-banking suite), [../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md) (the open-source microfinance core), and [../banking/mojaloop_guide.md](../banking/mojaloop_guide.md) (the open-source payments switch). Odoo is not in that family. **Odoo's place in a bank is the back office**: the finance, HR, procurement, and operations layer that runs *around* the core — and that is exactly where banks spend enormous sums on non-differentiating software.

What a bank actually runs Odoo (or any ERP) for — **✅/⚠ as the standard ERP-in-enterprise pattern; the banking-specific framing is analysis grounded in the banking series, flagged lightly**:

- **Finance & accounting** — the bank's own general ledger (separate from the customers' accounts in the core system): AP/AR, fixed assets, expense management, intercompany postings, management accounting, tax and statutory reporting packages.
- **HR & payroll-adjacent** — employee records, time off, expenses, timesheets, appraisals, recruitment tracking (core payroll usually stays in a specialized HR/payroll system or the bank's HCM suite).
- **Procurement & vendor management** — purchase orders, vendor contracts, goods/services receipt, invoice matching — the entire non-trading spend side.
- **Operations support** — facilities, fleet, IT asset tracking, helpdesk, document management, internal project portfolio.
- **The integration boundary** — the ERP must talk to the core-banking estate (posting feeds to the general ledger, reconciliation against core-system statements, single sign-on, data-governance reporting). Odoo's JSON-RPC/XML-RPC API (§3.5) and its bank-statement-import capabilities are the integration surface.

The strategic angle from the series' own finding in [../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md): open source in financial services dominates in *infrastructure* (Kafka, PostgreSQL, Python), and in *applications* it survives as long-tail and niche — with ERPs like Odoo as the notable exception where an open-source *application* reached commercial scale. A bank adopting Odoo is therefore not an exotic open-source bet; it is a mainstream cost play on a well-capitalized vendor (the $5B valuation, §2.4) in a domain (back office) where the bank has no regulatory need to be proprietary.

### 8.2 The Banking Table

| Bank domain | ERP (Odoo) fit | Where it does NOT fit | Verification |
|---|---|---|---|
| **Back-office finance** (bank's own GL, AP/AR, fixed assets, expenses) | Strong fit — core ERP territory | Customer accounts, deposits, loans (that is the core-banking ledger) | **✅/⚠ analysis + series cross-ref** |
| **HR / expenses / timesheets** | Strong fit — Odoo HR apps; payroll stays specialized | Core HRIS/HCM suites often remain for scale | **✅/⚠** |
| **Procurement & vendor management** | Strong fit — Purchase + vendor apps | Trading/supply-chain finance (the banking products in [../banking/supply_chain_finance_guide.md](../banking/supply_chain_finance_guide.md)) | **✅/⚠** |
| **Regulatory reporting** | No — report *source* data at best | The bank's regulatory data platforms (cf. [../banking/regtech_guide.md](../banking/regtech_guide.md)) | **✅/⚠** |
| **Core banking / payments** | No — out of scope | [../banking/t24_programming_guide.md](../banking/t24_programming_guide.md), [../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md), [../banking/mojaloop_guide.md](../banking/mojaloop_guide.md), [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) | **✅** (genre boundary) |
| **Insurance add-ons** | Community long-tail only (Odoo/ERPNext insurance modules) — per the series' finding | Commercial PAS/claims systems (cf. [../banking/insurance_software_systems_guide.md](../banking/insurance_software_systems_guide.md)) | **✅** (insurance guide) |
| **Banking-specific OCA modules** (bank statement import, SEPA/payment files, account-payment) | Good fit — the OCA's banking-adjacent projects | Not verified individually this pass | **⚠** (org verified; specific repos not scraped) |

### 8.3 Cross-References to the Bank Series

- **[../banking/insurance_open_source_commercial_guide.md](../banking/insurance_open_source_commercial_guide.md)** — the repository's open-source-vs-commercial survey; Odoo appears there as long-tail in insurance (§its tables) and as part of the structural finding that OSS infrastructure, not OSS applications, dominates financial-services IT. This guide is the counter-example deep-dive: Odoo is the OSS *application* that crossed into commercial scale.
- **[../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md) and [../banking/mojaloop_guide.md](../banking/mojaloop_guide.md)** — the open-source *core-banking/payments* genre; the comparison boundary for §8.1 (Fineract is a ledger; Odoo is not).
- **[../banking/t24_programming_guide.md](../banking/t24_programming_guide.md) and [jbase_universe_guide.md](jbase_universe_guide.md)** — the commercial-core and legacy-database genre; the T24/JBASE world is what the bank's *customer* systems look like, versus the Python/PostgreSQL world of the back office.
- **[../banking/full_stack_banking_guide.md](../banking/full_stack_banking_guide.md) / [../banking/financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md)** — where the back-office ERP slots into the bank's overall application stack (lightly cross-referenced).
- **[../management/business_case_development_guide.md](../management/business_case_development_guide.md)** — the ERP-selection business case: its TCO framing (acquisition, deployment, operations, support, training, disposal) is exactly the lens §9's worked example uses to justify Odoo versus a proprietary suite — Odoo's open-core economics (§4.4) make the TCO comparison unusually transparent (no license fee for Community; a flat per-user fee for Enterprise).

---

## 9. The Worked Example — a Cymbal Bank Back-Office Implementation

### 9.1 The Scenario

*Scenario, clearly framed as an illustrative design exercise in the familiar context of the author's employer (Cymbal Bank) — not a record of any actual project.*

Cymbal Bank's operations division runs its back office on a patchwork: a spreadsheet-era expense process, a legacy AP system nearing end of life, HR records in three different tools, and procurement spread across email. The core-banking estate (the T24-class ledger and the payments layer of §8.3) is healthy and untouched by this project. The mandate: **replace the back-office patchwork with one system — finance, expenses, procurement, HR-light — in eighteen months, under a hard budget, without hiring a large implementation team, and with the option to keep everything on-premise behind the bank's firewall.** The ERP-selection business case ([../management/business_case_development_guide.md](../management/business_case_development_guide.md)) compares the candidates on TCO and risk; Odoo wins the scenario on cost, inspectability (source access for a regulated institution), and the proven ecosystem, with the open-core caveat handled by the module-selection design below.

### 9.2 The Module Selection

The design principle: **start from Community, add Enterprise only where the feature gap hurts, and let the OCA fill the rest.** (Remember §4.1: recent Community editions lost the deep accounting features, so the finance decision is the pivotal one.)

| Back-office need | Odoo module | Edition | Rationale |
|---|---|---|---|
| Bank's own general ledger, AP/AR, fixed assets | Accounting (full: reports, reconciliation, assets) | **Enterprise** | The §4.1 gap — Community has Invoicing, not full accounting; the OCA accounting kits exist but a bank wants vendor-supported financials |
| Expenses, timesheets | Expenses, Timesheets | Community | Core functionality, fully in Community |
| Procurement, vendor master, PO→invoice matching | Purchase, Vendor Bills, Contracts | Community | Strong Community coverage; OCA `purchase-workflow` project extends it |
| HR-light: employees, time off, appraisals | Employees (HR), Time Off | Community | Deliberately *not* full Payroll — payroll stays with the bank's HCM/payroll suite (§8.1) |
| Internal helpdesk & IT asset tracking | Helpdesk, Maintenance, Fleet | Community/Enterprise mix | Helpdesk/Maintenance are Community; keep scope tight |
| Document management | Documents | Enterprise | Proprietary app; defer or replace with OCA `document-*` modules if budget dictates |
| Access & governance | Studio (no-code tweaks), multi-company, external API | Enterprise (Custom plan) | §4.2 — the Custom plan's entitlements; multi-company for the bank's legal entities |
| Integration to the core estate | JSON-RPC/XML-RPC integration modules | Custom-built | §3.5; GL feeds and reconciliation against core statements |

The resulting licensing shape: **Community core + Enterprise Custom plan for a bounded user set** (the pricing page's per-user model makes this cheap to model — §4.4). The design deliberately avoids the Enterprise-only *app sprawl* (PLM, Sign, Marketing Automation…) that a bank back office does not need.

### 9.3 The Deployment Design

- **Hosting**: on-premise behind the bank's firewall (the scenario's hard constraint), on the documented subscription-registration model (§4.3 — the server phones home to `services.odoo.com` for license validation; the bank's egress rules must permit exactly that and nothing more). If the compliance posture ever relaxes, Odoo.sh (§4.2) is the managed-cloud upgrade path with the same codebase — "the same software. Only the hosting and infrastructure are different" (§4.3).
- **Stack**: PostgreSQL (the documented backend), Python server with workers + cron (§3.5), a Linux host pair (prod + DR), the database manager for backup/duplicate with the neutralize option for test copies (§3.5).
- **Release strategy**: pin to the current major version at go-live; **budget an annual upgrade cycle** (§6 — the treadmill is real); use OCA OpenUpgrade (§5.3) as the migration machinery and maintain the OCB backports fork for security fixes between upgrades.
- **Integration**: one integration layer (JSON-RPC/XML-RPC — §3.5) feeding GL postings to the bank's financial data platform and pulling statement data for reconciliation; the external-API entitlement covers the direction where Odoo initiates calls (§4.4 nuance — verify the entitlement matches the call direction).
- **Governance**: external users (vendors on the portal) are free users per the pricing model (§3.4) — a real cost lever; internal users are the paying set.

### 9.4 The OCA-Extension Design

The OCA is where the scenario gets its bespoke depth without bespoke cost (§5.3):

- **Localization**: the bank's operating jurisdictions map to OCA `l10n-*` projects (e.g., `l10n-italy`, and equivalents for each country) — the OCA's 20,000+ modules across 62 countries are the localization layer the vendor alone cannot match.
- **Banking-adjacent**: the OCA's bank-statement-import and account-payment family for SEPA/ISO payment file handling (§8.2 — flagged ⚠ as org-level verified, repo-level not scraped).
- **Workflow depth**: `purchase-workflow`, `hr-*`, `server-ux` (user-experience improvements), `web-*` (web-client extensions) — the classic OCA building blocks for back-office processes.
- **The discipline**: every OCA module adopted must be (a) maintained for the deployed major version (the OCA maintains per-version branches), (b) reviewed under the OCA's CLA/quality process, and (c) tested in the staging branch before every upgrade — because the annual release treadmill (§6) will re-test every third-party module every year, and abandoned modules are the classic Odoo project killer. The design therefore **caps the OCA-module count and forces a maintenance review per upgrade cycle** — curation, not accumulation (§7.5).

### 9.5 The Lessons

1. **Know what you are buying**: open-core means the *core* is open and the *depth* is subscription — the accounting-feature gap (§4.1) is the first thing to model, not the last.
2. **The cadence is the cost**: an Odoo project's biggest recurring line item is the annual upgrade; budget it as a feature of the platform (§6), and lean on OpenUpgrade/OCB (§5.3) so the community absorbs part of it.
3. **The ecosystem is the moat — and the risk**: the OCA's 20,000 modules are the reason Odoo beats other open-source ERPs on breadth (§7.5); abandoned modules are the reason curation discipline decides project success.
4. **Integration, not replacement**: the ERP sits beside the core-banking ledger (§8.1); the integration layer (RPC, §3.5) and the data boundaries are where a bank architect earns the fee.
5. **The fork is your teacher**: Tryton's choices (§7) — GPL, federation, stability-first, automatic migration — are the mirror that shows what Odoo chose *not* to be; a serious Odoo evaluation is incomplete without the comparison.
6. **Transparency is a bank-grade feature**: source access, data portability ("You own your data!" — §4.3), and a flat per-user price make the TCO and the business case ([../management/business_case_development_guide.md](../management/business_case_development_guide.md)) unusually honest — which is exactly what a regulated institution should demand.

---

## 10. The Summary — the One-Page

**Odoo is the open-source ERP that grew up.** It began as TinyERP — a Belgian student's 2005 project by Fabien Pinckaers — became OpenERP in 2008 (the same year Tryton forked TinyERP 4.2 and set off down the road of GPL-licensed, federation-governed, stability-first purity), and rebranded as Odoo in 2014. Since the 9.0 release it has run the most successful open-core model in business software: an LGPLv3 Community core, a proprietary Enterprise layer (full accounting, Studio, Odoo.sh, support), and — since the 2022 pricing change — a flat per-user fee for all apps. The numbers verify the scale: €282M revenue in 2023, a ~$5-billion valuation in the November 2024 secondary round (CapitalG, Sequoia, BlackRock), 54k GitHub stars, and a community — the OCA, Switzerland-based, 20,000+ modules, 265 repositories, founded 2013 — institutionally built to keep Odoo open "regardless of what Odoo SA decides to do in the future."

Architecturally it is deliberately boring and deliberately modular: Python + PostgreSQL behind an ORM that turns model classes into schema, views, security, and APIs; a module/addon system with inheritance as its superpower; a bespoke TypeScript web client (OWL) that "powers Odoo's web client, one of the largest open-source business applications"; JSON-RPC/XML-RPC integration surfaces; and a relentless annual release cadence — 8.0 in 2014, 17.0 in 2023, 18.0 in 2024, 19.0 in September 2025 — whose LTS story is thinner than the label suggests and whose upgrade treadmill is the platform's true cost of ownership.

For the comparison: Tryton is what Odoo would be if the community had won — GPL, federation, automatic migration, stability; Odoo is what Tryton would be if the market had won — apps, ecosystem, partners, valuation. For a bank, Odoo is not a core-banking system and never will be; it is the back-office workhorse — finance, expenses, procurement, HR-light — that runs beside the T24-class ledger and the payments stack, integrable by RPC, deployable on-premise behind the firewall, and priced and licensed with a transparency that suits a regulated institution.

The final word is **the open book**. Odoo's source is open, its pricing is published, its history is documented, its community is institutionalized, and its limits are knowable in advance — every fact in this guide was either verified against the record or flagged as not. For an architect whose career began in the ERP/warehouse-management domain ([asia_fusion_technology_guide.md](asia_fusion_technology_guide.md) §4.4) and whose bank needs a back office that is cheap, inspectable, and never a black box, that openness is the point: **with Odoo, the book is open — read it before you buy it, and keep reading after.**

---

## 11. The Glossary

| Term | Definition |
|---|---|
| **Odoo** | The open-source business-management suite (ERP/CRM/e-commerce/back-office apps) developed in Belgium; the product of Odoo S.A. (formerly OpenERP S.A.) — **✅** |
| **TinyERP** | The original name of the software, first released February 2005 by Fabien Pinckaers — **✅** |
| **OpenERP** | The product/company name from 2008 until the 2014 rebrand; still visible in the `services.openerp.com` licensing endpoint for Odoo ≤17 — **✅** |
| **Fabien Pinckaers** | Founder and CEO of Odoo; developed the software that became TinyERP in 2005 — **✅** |
| **Python** | The server-side implementation language of Odoo (and Tryton) — **✅** |
| **PostgreSQL** | The database backend of Odoo; tables are managed automatically by the ORM — **✅** |
| **ORM** | Object-relational mapping: Odoo's model layer — Python classes become PostgreSQL tables, recordsets, security, and views automatically (§3.2) — **✅/⚠** |
| **Module** | The unit of Odoo software: manifest + Python + XML data/views + security files; apps and extensions are modules (§3.3) — **✅** |
| **OCA** | The Odoo Community Association — the Switzerland-based non-profit (est. 2013) running 20,000+ community modules, 265 GitHub repos, OpenUpgrade and OCB (§5.3) — **✅** |
| **Community** | The LGPLv3 open-source edition of Odoo — **✅** |
| **Enterprise** | The proprietary-apps + support subscription layer of Odoo (open-core model since 9.0) — **✅** |
| **Odoo.sh** | Odoo's PaaS: Git-branch-based dev/staging/production hosting for custom modules — **✅** (pricing FAQ) |
| **OWL** | The Odoo Web Library — Odoo's bespoke TypeScript UI framework (~30kb, LGPLv3) powering the web client — **✅** (odoo/owl README) |
| **ERP** | Enterprise Resource Planning — integrated business-management software (finance, supply, HR, operations) — **✅** |
| **CRM** | Customer Relationship Management — the sales-pipeline/lead-tracking app family — **✅** |
| **MRP** | Manufacturing Resource Planning — Odoo's manufacturing module (bills of materials, work orders); also the general production-planning discipline — **✅** |
| **GPL** | GNU General Public License — strong copyleft; Tryton's license (GPL-3.0-or-later) — **✅** |
| **LGPL** | GNU Lesser General Public License — weaker copyleft permitting proprietary linking; the Community edition's license (LGPLv3) — **✅** |
| **Tryton** | The 2008 fork of TinyERP 4.2 (Cédric Krier + community); GPL-3.0, federation-governed, framework-first — the Odoo-vs-Tryton comparison of §7 — **✅** |
| **LTS** | Long-Term Support — in Odoo's current documentation, the major releases supported in parallel (16/17/18 until 16's Sep 2025 EOL); in practice the OCA's OpenUpgrade/OCB carry long-term Community maintenance — **✅/⚠ (§6.2)** |

---

## 12. The Verification Ledger

### Verified this pass (✅)

- **Definition**: open-source business-management suite developed in Belgium; ERP/CRM/apps scope (Wikipedia, odoo.com).
- **Founding**: TinyERP first release February 2005; Fabien Pinckaers as original author/founder-CEO (Wikipedia).
- **Renames**: TinyERP → OpenERP (2008); OpenERP → Odoo (2014, announced 14 May 2014 per the company's "OpenERP becomes Odoo" post, archived in the Wayback Machine).
- **Tryton lineage**: fork of TinyERP 4.2, first published November 2008 (Wikipedia, citing the release record and LWN); Cédric Krier as original author; GPL-3.0-or-later (Wikipedia + repo COPYRIGHT).
- **Licenses**: Odoo Community = GNU LGPLv3; Enterprise = proprietary (Wikipedia infobox + odoo/odoo LICENSE); OWL = LGPLv3 (odoo/owl LICENSE); OCA modules = AGPL-3.0 (repo license badges).
- **Open-core**: open-core model since 9.0; OCA founded 2013 (Wikipedia).
- **Funding/valuation**: €3M round 2010 (TechCrunch); $90M round Dec 2019 (TechCrunch); Nov 2024 secondary round ~$500–527M at $5.26–5.3B/€5B valuation with CapitalG/Sequoia/BlackRock (TechCrunch, CNBC, SiliconANGLE, ETStartup — **figures cross-checked; discrepancies flagged ⚠**); €282M revenue 2023, +33%, 2,200+ employees (De Tijd via Wikipedia).
- **Release cadence**: annual major versions; 8.0 (2014) → 17.0 (2023) → 18.0 (Oct 2024 — "Meet Odoo 18" announcement archived 3 Oct 2024) → 19.0 (18 Sep 2025, Wikipedia/Wikidata); GitHub default branch 19.0, 54k stars / 33.5k forks / ~202k commits (repo page).
- **Pricing/editions**: One App Free / Standard US$16.90 / Custom US$25.50 per user/month; Studio, multi-company, external API, Odoo.sh, on-premise as Custom-plan entitlements (odoo.com/pricing); "Odoo Online or Odoo Enterprise… is the same software" (pricing FAQ).
- **Odoo.sh**: CI-platform description verbatim from the pricing FAQ.
- **On-premise**: subscription-code registration; `services.odoo.com` (18.0+) vs `services.openerp.com` (≤17.0) endpoints; Community↔Enterprise switch procedure; data portability ("You own your data!").
- **OWL**: odoo/owl README — TypeScript, ~30kb, zero deps, "powers Odoo's web client", LGPLv3.
- **RPC**: JSON-RPC/XML-RPC external API (pricing FAQ).
- **OCA**: 20,000+ modules, 62 countries, 1,600 contributors, 664 members, 38 sponsors, 265 GitHub repos, OpenUpgrade ("open source upgrade path for Odoo/OpenERP"), OCB (community backports), CLA, mission statement verbatim, Switzerland non-profit (odoo-community.org + github.com/OCA).
- **App store**: exists, per-version listing structure, top-chart vendors, the "everything Community is missing" accounting-kit category (apps.odoo.com).
- **Repo cross-refs**: Odoo mentions in insurance_open_source_commercial_guide.md, asia_fusion_technology_guide.md §4.4–4.5, jack_liu_profile.md; sibling filenames for jbase_universe_guide.md, apache_fineract_guide.md, mojaloop_guide.md, t24_programming_guide.md, business_case_development_guide.md.

### Flagged / not fully verified (⚠)

- **App-store total module count**: JS-rendered browse page; aggregate count not extractable — the guide states "tens of thousands" with the OCA's 20,000+ as the verified anchor, and refuses to repeat an unverified headline.
- **Odoo 18 exact release day**: pinned to October 2024 via the archived announcement (3 Oct 2024 capture); the exact day (widely reported as 1 October) not independently confirmed.
- **LTS policy**: Wikipedia's current account (major releases = LTS, parallel support, 16.0 EOL Sep 2025) is cited, but Odoo's own supported-versions page was bot-blocked; the historical "latest version only" posture and the OCA's de-facto long-term role are flagged as the operational reality.
- **Funding figures**: press discrepancies ($500M vs $527M; $5.26B vs $5.3B vs €5B) recorded as published, not smoothed.
- **Adoption numbers**: "5M+ users" (2021) is report-sourced; no independent audit.
- **Company financials**: €282M revenue is company-reported via De Tijd.
- **Community vs Enterprise accounting split**: corroborated by the app-store module category but the primary source (Odoo's edition-comparison docs) was bot-blocked.
- **Architecture specifics**: Python/PostgreSQL/ORM/module design are the framework's ubiquitous documented design and the repo structure confirms them, but the install-requirements and ORM-reference pages were bot-blocked, so exact minimum version numbers and some operational details are not quoted.
- **SAP Business One comparison** (§7.8): general-knowledge light treatment, not re-verified this pass.
- **GTK-client era, OWL adoption timing, Odoo.sh operational specifics, OCA banking-module repos individually**: general knowledge / org-level verification only.
- **The worked example** (§9): an explicitly framed illustrative scenario in the Cymbal Bank context — not a record of any actual project.

---

*End of guide — the open book. Last updated August 2026. Companion guides: [Asia Fusion Technology](asia_fusion_technology_guide.md), [Insurance — Open Source vs Commercial](../banking/insurance_open_source_commercial_guide.md), [Business Case Development](../management/business_case_development_guide.md).*
