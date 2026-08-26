# TCO Modeling: The Total-Cost-of-Ownership Discipline — A Comprehensive Guide

*A deep dive into TCO modeling — the cost-estimation discipline that answers "what does this system really cost, all-in, over its lifetime?" — covering the overview (the definition, the overview table, the cost-only lens), the history (the Gartner 1987 origin, the Forrester TEI, the history table), the cost taxonomy (acquisition CapEx, operating OpEx, the hidden costs — end-user, downtime, training, decommissioning/EOL, the taxonomy table), the models (on-prem-vs-cloud, perpetual-vs-subscription, mainframe-vs-cloud, the models table), the methodologies (Gartner TCO, Forrester TEI, IDC, the methodology table), the modeling practice (the assumptions register, sensitivity/tornado, discounting, the horizons, the practice table), the banking angle (mainframe-vs-cloud TCO, the data-centre TCO, the banking table), the worked example (a Cymbal Bank on-prem-vs-cloud 5-year TCO with worked numbers, the lessons), the one-page summary — the final word is 'the true cost' — and the glossary.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore

> **Primary sources:** the TCO canon — the UK Cabinet Office / HMG ICT Strategy *Total Cost of Ownership: Things to Consider* (V1.0, 2011; the acquisition/operation/end-of-life taxonomy and the "developed by Gartner in 1987" attribution — extracted directly this pass), the Wikipedia *Total cost of ownership* article and the sources it carries (the Gartner "About Gartner TCO" archive, the UK government definition), the US EPA *Total Cost of Ownership Modeling for Electronics* paper (the "popularized by Gartner Research in 1987" attribution), Forrester's own Total Economic Impact™ methodology pages (the four components — cost, benefits, flexibility, risk; the 20+ year history), the AWS pricing/TCO documentation (the AWS Pricing Calculator and TCO tools), the on-prem-vs-cloud TCO literature (the 3–5-year horizon convention, the cloud TCO calculators), and the sensitivity-analysis / tornado-chart practice sources. **NOTE: this pass had live web access (self-hosted Firecrawl backend) — key claims verified on 2026-08-24 via web search and direct page extraction. Anything that could not be pinned to a primary source is flagged ⚠ honestly.**

> **Integrity convention (repo standard):** ✅ = verified this pass (live web or a cross-referenced guide's own ledger); ⚠ = flagged — commonly cited but not pinned to a single authoritative public source this pass; ⚠-structural = industry-standard practice, widely documented, but not attributable to one primary source. **Never fabricated:** all cost figures in this guide are either sourced or explicitly labelled as illustrative planning figures with the arithmetic shown.

---

## Cross-References (this guide maps into the repo's cost/investment cluster)

This guide is the **dedicated TCO-modeling treatment** in the cost/investment cluster. The siblings use TCO as a *component*; this guide is the *discipline* behind the component. Cross-ref heavily:

- [business_case_development_guide.md](../management/business_case_development_guide.md) — **the decision machinery this guide feeds** (34 TCO mentions): the §4 financials (TCO → NPV/IRR/payback/ROI), the §4.6 financial-model design discipline (named assumptions, inputs sheet, frozen version — this guide's §6 practice), and the §8 Cymbal Bank worked example (the S$8.0M capex trade-finance STP platform case, S$6.9M NPV at a 10% hurdle, 3.1-year payback) whose TCO layer §8 of THIS guide models in detail. **This guide does NOT re-derive the NPV/IRR/payback or the Five Case machinery — cross-ref that guide for them.** TCO answers "what does it cost, all-in"; the business case answers "is it worth it".
- [finops_guide.md](finops_guide.md) — **the cloud ongoing-cost management** (just extended): unit economics, showback/chargeback, commitments, §11 FinOps in Banking (BCBS 239 cost attribution, landing-zone cost controls). FinOps is the *post-decision* operating half of the cloud TCO; this guide is the *pre-decision* estimation half.
- [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) — **the priority this guide closes**: the skill-gap scan flagged "business case & TCO modeling for data initiatives" as gap #2 / a standing priority, with `finops_guide.md` covering only cloud operating cost. The business-case guide plus this guide close that priority.
- [mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) — the quant vocabulary (§6: DCF/NPV/WACC/CAPM) behind §6's discounting discussion; cross-ref lightly.
- [singapore_data_centres_guide.md](singapore_data_centres_guide.md) — **the DC-cost reality** (power, cooling, PUE, the moratorium → green-criteria regulatory story) behind §7's data-centre TCO angle.
- [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) — **just added** — the downtime-cost angle: the nines math (§2.4) and the "cost of an extra nine must be justified by the business value of the avoided downtime" framing (§2.3) behind §3's hidden-cost catalogue.
- [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) — **just added** — the vendor-TCO pattern: its §9/§10 3-year TCO sketch (licence line vs ops burden; figures flagged) is a worked mini-example of this guide's discipline.
- [cockroachdb_guide.md](cockroachdb_guide.md) — **just added** — the licensing-cost angle (§8.4 the Apache→BSL→CCL journey, §8.6 pricing) behind §4's perpetual-vs-subscription model.
- [cloud_providers_guide.md](cloud_providers_guide.md) — §14 (Cloud Cost Management / FinOps): the provider-side cost-management layer; cross-ref with finops_guide.
- The banking cluster ([../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md), [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md), the bank series) — the mainframe estates behind §7's banking angle: the "fully amortized cost base vs high and rising platform costs" tension.

**Link convention (repo standard):** siblings in `technology/` are plain filenames; `ai_llm/` subdir guides are prefixed `ai_llm/`; `banking/` and `management/` guides are prefixed `../banking/` and `../management/`.

---

## How to Use This Guide

Section 1 is the overview — what TCO is, the definition, the cost-only lens, and the overview table (aspect / description). Section 2 is the history — the Gartner 1987 origin (verified, with the flag), the Forrester TEI, the IDC angle, and the history table (year / event / notes). Section 3 is the cost taxonomy — acquisition CapEx, operating OpEx, the hidden costs (end-user, downtime, training, decommissioning/EOL, migration, compliance), and the taxonomy table (category / components / notes). Section 4 is the models — on-prem-vs-cloud (the 3–5-year horizon), perpetual-vs-subscription, mainframe-vs-cloud, and the models table (model / framing / costs captured / notes). Section 5 is the methodologies — Gartner TCO (flagged on detail), Forrester TEI (verified), IDC (flagged), and the methodology table (methodology / origin / components / notes). Section 6 is the modeling practice — the assumptions register, sensitivity/tornado, discounting (the NPV connection), the horizons, and the practice table (practice / what / why it matters / notes). Section 7 is the banking angle — mainframe-vs-cloud TCO, the data-centre TCO, and the banking table. Section 8 is the worked example — a Cymbal Bank on-prem-vs-cloud 5-year TCO model with worked numbers and the lessons. Section 9 is the one-page summary — the final word is **the true cost**. The glossary, the claims-status ledger and the cross-references close the file.

---

## Table of Contents

1. [The TCO Overview](#1-the-tco-overview) — the definition, the cost-only lens, the overview table
2. [The History](#2-the-history) — the Gartner 1987 origin, the Forrester TEI, the history table
3. [The Cost Taxonomy](#3-the-cost-taxonomy) — acquisition CapEx, operating OpEx, the hidden costs, the taxonomy table
4. [The Models](#4-the-models) — on-prem-vs-cloud, perpetual-vs-subscription, mainframe-vs-cloud, the models table
5. [The Methodologies](#5-the-methodologies) — Gartner TCO, Forrester TEI, IDC, the methodology table
6. [The Modeling Practice](#6-the-modeling-practice) — the assumptions register, sensitivity/tornado, discounting, the horizons, the practice table
7. [The Banking Angle](#7-the-banking-angle) — mainframe-vs-cloud TCO, the data-centre TCO, the banking table
8. [The Worked Example](#8-the-worked-example) — the Cymbal Bank scenario, the 5-year model with worked numbers, the lessons
9. [The Summary](#9-the-summary) — the one-page, 'the true cost'
10. [The Glossary](#the-glossary)
11. [The Claims-Status Ledger](#the-claims-status-ledger)

---

# 1. The TCO Overview

## 1.1 The Definition (verified)

**Total cost of ownership (TCO) is a financial estimate intended to help buyers and owners determine the direct and indirect costs of a product or service over its full life cycle.** (✅ Verified — the Wikipedia definition, which the UK government's own TCO guidance matches in substance: "TCO allows departments to look at both the short and long term costs of any particular solution" — UK Cabinet Office, *Total Cost of Ownership: Things to Consider*, V1.0, HMG ICT Strategy.)

The discipline rests on one empirical claim, stated in the UK government's own guidance and in the Gartner lineage since 1987:

> "Traditionally cost has been associated with the purchase price alone or purchase price plus support costs, however **the initial procurement cost is typically only a relatively small percentage of the total cost of owning and operating most IT products**." (✅ UK Cabinet Office, verbatim)

The canonical illustration is the HMG "puppy" analogy: a rescued puppy may be free, but the cost of keeping it for its lifetime most certainly is not — food, shelter, vaccinations — and "in caring for the animal over its lifetime you will have spent a substantial amount of money, despite it costing you nothing to actually acquire the puppy at the outset." (✅ UK Cabinet Office.) The IT translation: a platform that is "cheap to buy" but expensive to run, staff, power, patch, migrate and retire is the expensive platform — and the whole point of TCO modeling is to make that visible *before* the purchase decision.

Formally, a TCO analysis includes:

$$TCO = \text{acquisition cost} + \text{operating cost (cumulative over the horizon)} + \text{end-of-life cost} + \text{hidden/indirect costs}$$

where the components are estimated per year, summed over the appraisal horizon (typically 3–5 years), and — when the model feeds a business case — discounted to present value (this guide's §6.3; the discounting machinery lives in [business_case_development_guide.md](../management/business_case_development_guide.md) §4.2).

## 1.2 What TCO Is — and What It Is Not

The UK government guidance is unusually explicit about the boundary, and it is worth quoting as the discipline's own self-description:

- **TCO is cost-only.** "TCO is only concerned with the financial cost of any solution independent of any other benefits." It does **not** assess how well a solution meets requirements, does **not** assess risk or strategic fit, and does **not** necessarily track environmental or social costs and benefits. (✅ UK Cabinet Office.)
- **TCO is therefore a component of options analysis, not the whole of it.** "TCO modelling... should form a part of a bigger options analysis exercise which evaluates the proposed solutions based on more than just cost i.e. suitability, reliability, support, appropriateness." (✅ UK Cabinet Office.) The cheapest option is not always the better option — "if software is cheaper but does not actually do everything you require it to, forcing you to go back out to market at a later date and incurring new costs." (✅.)
- **TCO is a comparison discipline, not an absolute number.** Its value comes from comparing options — including do-nothing and including the *vendor's* option vs the *internal* option — on the same basis with the same assumptions. (✅ in substance; the business-case guide's §1.4 "comparative, not absolute" rule is the same rule.)
- **TCO is the cost-estimation layer of the business case.** The business case asks "is this worth it?" via NPV/IRR/payback; TCO supplies the cost stream those metrics discount. This guide is the dedicated treatment of that layer; [business_case_development_guide.md](../management/business_case_development_guide.md) is the decision machinery it feeds — do not re-derive the NPV/IRR/Five-Case content here.

**The placement in the cluster:** TCO estimation happens *before* the funding decision (this guide); FinOps manages the cloud cost *after* the decision ([finops_guide.md](finops_guide.md)); the business case wraps both in the decision framework ([business_case_development_guide.md](../management/business_case_development_guide.md) §4); and the skill-gap scan that commissioned all of it is [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) (gap #2 — business case & TCO modeling — now closed by the pair of guides).

## 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **Definition** | A financial estimate of the **direct and indirect costs** of a product or service over its full life cycle — acquisition, operation, maintenance, and end-of-life (✅ Wikipedia; UK Cabinet Office in substance) |
| **Core claim** | The purchase/acquisition price is **a small fraction** of lifetime cost — Gartner's PC-era finding put it at ~20% of the 5-year cost (⚠-structural, the Stratrix/Gartner-lineage figure; the UK government states the principle, not the percentage) |
| **Scope** | Full life cycle: **cost to create, cost to operate, cost to change, cost to end** (✅ the UK Cabinet Office diagram: acquisition, licenses, design, build, training, customisation, maintenance, upgrade, change, decommission, replace, data migration, carbon footprint) |
| **Lens** | **Cost-only** — explicitly does not assess benefits, risk, or strategic fit (✅ UK Cabinet Office); feeds a wider options analysis |
| **Horizon** | Typically **3–5 years** for IT infrastructure decisions (✅ cloud-TCO literature, verified §2/§4); longer for facilities, mainframes, data centres |
| **Decision role** | Supplies the **cost stream** to the business case's NPV/IRR/payback machinery — TCO answers "what does it cost, all-in"; NPV answers "is it worth it" (cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md) §4) |
| **Companions** | FinOps = the **post-decision** cloud-cost management ([finops_guide.md](finops_guide.md)); the business case = the **decision** wrapper; the skill-gap scan = the **motivation** ([data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md)) |
| **Origin** | Popularized by **Gartner in 1987** for IT costs (✅ EPA; ✅ UK Cabinet Office; ⚠ the original 1987 research note itself is not publicly accessible — §2.1) |
| **Failure mode** | A TCO model whose assumptions are hidden, whose horizon is picked to flatter one option, or whose "hidden" costs were left out — the model then *decides* instead of *informing* |

## 1.4 Why the Discipline Exists

TCO exists because procurement incentives push buyers toward the wrong number. The Gartner lineage's observation — restated in Gartner's current TCO research ("TCO principles are complex and hard to implement, incentivizing continued emphasis on unit price versus total cost") — is that organizations gravitate to the sticker price because the full cost is hard to see: it is spread across IT, facilities, procurement, and business-unit budgets; it lands in different fiscal years; and much of it (end-user time, downtime, decommissioning) is never on any invoice. The discipline's job is to make the invisible visible and the challengeable explicit. Section 3 is the catalogue of what "full cost" means; §6 is the practice of building a model that survives finance's adversarial pre-read.

---

# 2. The History

## 2.1 The Gartner 1987 Origin (verified — with the flag)

**TCO analysis was popularized by Gartner (then the Gartner Group) in 1987, in the context of desktop/PC computing costs.** The attribution is verified across multiple independent sources this pass:

- ✅ **US EPA**, *Total Cost of Ownership Modeling for Electronics*: "TCO models for IT investments were popularized by Gartner Research in 1987 and are now widely used."
- ✅ **UK Cabinet Office**, *Total Cost of Ownership: Things to Consider* (HMG ICT Strategy, 2011): "TCO models were initially developed by Gartner in 1987 and are now widely used to assist decision making relating to the selection or procurement of an Information Technology product."
- ✅ **Wikipedia**, citing Gartner's own archived "About Gartner TCO" document (web.archive.org, 2006 capture of gartner.com): "TCO analysis was popularized by the Gartner Group in 1987."

**The flag:** the *specific* 1987 Gartner research note is not publicly accessible (Gartner research is paywalled; the archive link is a summary page, and even its exact title/author is not pinned this pass). The 1987 date is therefore **verified as the standard attribution across government and industry sources, with the primary note itself unverifiable**. The commonly told backstory — that Gartner analyst **Bill Kirwin** led the early TCO work and demonstrated that a PC's purchase price was only ~20% of its five-year cost of ownership (the rest being deployment, support, administration, and end-user time) — is ⚠-structural: consistent across the Gartner-lineage literature but not pinned to a single primary document this pass.

**Why 1987 was the moment:** the PC era had just decentralized computing out of the data centre onto every desk. Purchase price was visible; the support burden (help desks, administration, end-user self-support, training, downtime) was diffuse and hidden. TCO was the analytical answer to "what does this desktop actually cost the firm?" — the same move this guide's §3 makes for platforms and workloads. The concept's roots go further back (the Wikipedia article notes the *concept* dates at least to the early 20th century as a management-accounting idea; facilities TCO was pioneered independently at Brigham Young University in the 1980s ⚠), but the **IT TCO discipline as we practice it is the 1987 Gartner lineage.**

## 2.2 The Forrester TEI (verified)

**Forrester's Total Economic Impact™ (TEI) methodology is the other pillar of the canon** — a complementary framework that extends cost into value. Verified directly from Forrester's own methodology page this pass:

- TEI is Forrester's "proven methodology to justify, quantify, and articulate the value of an investment," developed by the TEI consulting practice, "used for over 20 years by technology consumers and technology organizations."
- It evaluates investment value across **four components: cost, benefits, flexibility, and risk** (✅ Forrester, verbatim). Where classic TCO models *costs*, TEI models *economic impact*: the benefit stream, the option value of flexibility (what the investment enables later), and the risk-adjusted view.
- TEI studies are built on **due diligence, independent customer interviews and primary research**, with a financial model framework; they are commissioned (clearly marked), Forrester maintains editorial control, and positive results are not guaranteed (✅ Forrester).
- The exact year TEI launched is ⚠ (not pinned to a public source this pass; Forrester states "over 20 years," placing its origins in the early 2000s).

**TEI vs TCO in one line:** Gartner's TCO asks "what does it cost, all-in?"; Forrester's TEI asks "what is the whole economic impact — costs, benefits, flexibility, risk — of owning it?" A TCO model is the cost half of a TEI. Both are decision-support frames for the same investment question, and both feed the [business-case](business_case_development_guide.md)-style NPV machinery — cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md) §4 for how the cost stream becomes NPV.

## 2.3 The IDC Angle (flagged)

**IDC (International Data Corporation, founded 1964) is the third named methodology in the canon, and the one that verifies least well publicly** ⚠. IDC is a major IT market-research firm that publishes TCO studies and "Business Value" research (quantifying the value of technology investments, often in partnership with vendors), and its name appears in the TCO literature as a standard reference. What could NOT be pinned this pass: a single public, authoritative IDC document describing a canonical "IDC TCO methodology" — the firm's methodology documents sit behind its paywall, and the public artifacts are vendor-facing study summaries. **Flagged honestly:** the IDC methodology is real and widely referenced (its TCO/cost-per-unit studies inform storage, server, and cloud comparisons), but this guide treats it as ⚠ — referenced, not re-verified. The IDC contribution that does come through clearly in the literature is the **cost-per-unit / unit-economics angle**: expressing TCO per unit of work (per GB, per VM, per transaction) to make comparison portable — the habit §6 recommends for every model.

## 2.4 The Cloud Era and the Calculators

The third act of the history is the cloud: from 2006 (AWS S3/EC2) onward, the on-prem-vs-cloud TCO comparison became the canonical use of the discipline — and the vendors industrialised it. Verified this pass:

- **AWS** publishes pricing/TCO tooling: the **AWS Pricing Calculator** and TCO tools, documented in the official *How AWS Pricing Works* whitepaper (✅ docs.aws.amazon.com: "If the workload details and services to be used are identified, AWS Pricing Calculator can help with calculating the total cost of ownership").
- The other hyperscalers ship equivalents (Azure TCO Calculator, Google Cloud pricing calculator) ⚠-structural (well documented in the cloud literature; not individually re-verified this pass).
- The vendor calculators are **comparison engines with an agenda**: they encode the vendor's assumptions (utilization rates, staff costs, power costs, hardware refresh cycles) and are biased toward the vendor's outcome. The discipline's answer is not to refuse them but to treat them as *inputs to challenge* — the assumptions register (§6.1) exists precisely to surface and override them.
- The FinOps movement (FinOps Foundation, ~2019 ⚠-structural) then added the *operating* half: what to do about cloud cost once it is real and recurring — cross-ref [finops_guide.md](finops_guide.md) and [cloud_providers_guide.md](cloud_providers_guide.md) §14.

## 2.5 The History Table

| Year | Event | Notes |
|---|---|---|
| 1987 | **Gartner popularizes TCO for IT** (✅ EPA, UK Cabinet Office, Wikipedia citing Gartner; ⚠ the original research note is not public) | PC-era: purchase price ≈ ~20% of 5-year cost (⚠-structural, Bill Kirwin lineage); the rest is support, administration, end-user time |
| 1980s | Facilities TCO pioneered at BYU (⚠) | Independent parallel lineage for buildings; became ANSI standard APPA 1000-1 in 2017 (✅ Wikipedia) |
| ~2000s | **Forrester launches TEI** (✅ "used for over 20 years"; ⚠ exact year) | Extends cost into value: cost, benefits, flexibility, risk (✅ Forrester) |
| 2006 | Cloud era begins (AWS S3/EC2) | TCO becomes the standard frame for on-prem-vs-cloud; vendor TCO calculators appear |
| 2006–present | Vendor TCO tools (✅ AWS Pricing Calculator + TCO tools; ⚠-structural for Azure/GCP) | Comparison engines with vendor agendas — challenge their assumptions (§6.1) |
| 2019 | FinOps Foundation (⚠-structural) | The post-decision operating discipline for cloud cost — cross-ref [finops_guide.md](finops_guide.md) |
| Today | TCO is standard procurement practice | UK government policy: decisions on "total lifetime cost of ownership... including exit and transition costs" (✅ Cabinet Office); the discipline extends to AI/GPU estates and data-centre power economics (§7) |

---

# 3. The Cost Taxonomy

The taxonomy below is anchored on the **UK Cabinet Office TCO checklist** (✅ extracted directly from the HMG document this pass) and the **Wikipedia TCO component list** (✅), organised into the three canonical buckets — acquisition (CapEx), operating (OpEx), and hidden costs — with end-of-life as its own category. The Cabinet Office structure is worth knowing verbatim: **Acquisition and procurement** (selection, upfront evaluation, purchase price, licenses, hardware, integration); **Operation and management** (migration of data and users, use, maintenance, upgrades, support services, training, software scaling, customisation/change, development/modification, carbon footprint); **End of life management** (retirement, disposal, migration of data and users); plus **additional indirect costs** (SLA-assurance costs incurred with third parties, unplanned costs such as compliance auditing and under-licensing). The government's own diagram compresses this to four phases: **cost to create / cost to operate / cost to change / cost to end** (✅).

## 3.1 Acquisition Cost — CapEx (verified)

**Acquisition cost is the one-time, capitalizable expenditure to obtain and stand up the asset** — the CapEx of the decision. From the Cabinet Office checklist (✅): selection and upfront evaluation (the procurement effort itself — commonly forgotten), purchase price, licenses, hardware, and integration. In an IT-platform TCO this is: servers/storage/network hardware; software licences (perpetual licenses — §4.2); initial implementation/integration services; the migration of data and users *in*; facilities build-out (racks, power drops, cooling for a new DC footprint); and initial training to make the asset usable.

The classic TCO trap is treating acquisition as *the* cost — "the initial procurement cost is typically only a relatively small percentage of the total cost of owning and operating most IT products" (✅ Cabinet Office). The §8 worked example makes this concrete: acquisition is 28% of the on-prem 5-year nominal TCO — and 34% of the *discounted* TCO, because it is paid in year zero with no discounting relief. A TCO model that stops at acquisition systematically understates the true cost by the largest factor in the discipline.

## 3.2 Operating Cost — OpEx (verified)

**Operating cost is the recurring, consumption/run expenditure over the asset's life** — the OpEx. From the Cabinet Office checklist (✅): use, maintenance, upgrades, support services, training, software scaling, customisation/change, development/modification, and the carbon footprint of running the estate. In IT-platform terms: staff (FTEs for administration, operations, patching, DBA, platform engineering — usually the single largest line), hardware maintenance contracts, software maintenance/support renewals, power and cooling, floor space/rack space (or colocation fees), network and telecoms, backup and DR operations, and — for cloud — the consumption bill (compute, storage, network/egress, managed services) plus support plans. The Wikipedia component list adds the operational classics: infrastructure/floor space, electricity, testing, insurance, IT personnel, corporate-management time, backup/recovery process, audit (✅).

**The OpEx note for banks:** a surprising share of operating cost is *change* — "cost of customisation (change)" and "development, modification" are explicit Cabinet Office lines (✅). Legacy estates spend their OpEx on change; greenfield estates defer it; a TCO that ignores change cost systematically flatters the status quo. And for cloud, operating cost is *elastic* — it scales with usage, which is why the growth assumption (§6.1, §8) is a first-order sensitivity, and why [finops_guide.md](finops_guide.md) exists to manage it after the decision.

## 3.3 The Hidden Costs (verified)

The "hidden" costs are the ones no invoice announces, which is exactly why the discipline exists. The taxonomy table below catalogues them; the canonical set:

- **End-user cost** — the time users spend learning, waiting, self-supporting, and working around the system. Wikipedia lists it operationally: "diminished performance (i.e. users having to wait, diminished money-making ability)" and "technology/user training" (✅). The Gartner PC-era finding — support and end-user time dwarfing the purchase price — is this line (⚠-structural, the ~80% figure). At bank scale, end-user time is usually the largest single *unbudgeted* cost of a system. (Cross-ref: the business-case guide's §8 treats productivity/benefits on the benefit side; here the end-user cost sits on the cost side of the TCO.)
- **Downtime** — outage, failure, and diminished-performance costs, including lost revenue, SLA penalties, remediation labour, and reputational damage (✅ Wikipedia: "downtime, outage and failure expenses"). The quantitative frame lives in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md): the nines math (§2.4 — 99.9% ≈ 8.77 h/yr downtime, 99.99% ≈ 52.6 min/yr) and the discipline's core rule that "the cost of an extra nine must be justified by the business value of the avoided downtime" (§2.3, the SRE book's framing). A TCO model should put a number on the availability target it prices — this is where the reliability architecture (§3–§6 of the zero-downtime guide) meets the money.
- **Training** — both the initial training to stand the system up and the recurring training as staff and features turn over (✅ Cabinet Office: training appears under acquisition-adjacent and operation; ✅ Wikipedia).
- **Decommissioning / EOL (end of life)** — retirement, disposal, and the migration of data and users *out* to the replacement (✅ Cabinet Office: "retirement, disposal, migration (data and users)" under End of Life Management). Note the UK government's definition explicitly includes retirement/disposal/migration but *not* replacement — the replacement system's cost is a new TCO. Decommissioning is the most frequently zeroed line in vendor models and one of the first a bank's finance team checks: exit cost is a contractual and data-sovereignty reality (MAS outsourcing guidance, data residency — cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) §2.6 and the regtech cluster).
- **Migration (data and users)** — appears on *both* sides of the lifecycle (✅ Cabinet Office: under Operation and Management and under End of Life Management): migrating *into* the platform at acquisition and *out of* it at EOL. Migration is where the "cheap" option's hidden cost lives — the [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) §9 pattern (zero licence, full migration/ops burden) is this line in miniature.
- **Compliance and under-licensing** — "unplanned costs, for example the possibility of unanticipated expenditure through compliance auditing and under-licensing" (✅ Cabinet Office, the additional-indirect-costs list). Audit, licence-tracking, and true-up exposure.
- **SLA-assurance costs** — "costs incurred with another party to ensure the ability to meet Service Level Agreement targets for business critical solutions" (✅ Cabinet Office) — the extra redundancy, escrow, or third-party assurance bought to *guarantee* the SLA.
- **Carbon/environmental** — the carbon footprint is an explicit Cabinet Office line (✅) — and in Singapore's DC market it has become a *capacity* constraint, not just an ESG line (cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) §2.3–2.6: BCA Green Mark, the green criteria that lifted the moratorium).

## 3.4 The Taxonomy Table

| Category | Components | Notes |
|---|---|---|
| **Acquisition (CapEx)** | Selection/evaluation effort, purchase price, hardware (servers/storage/network), software licences (perpetual), integration/implementation services, migration-in (data + users), facilities build-out, initial training | "Typically only a relatively small percentage of the total cost" (✅ Cabinet Office); the trap is stopping here — §3.1 |
| **Operating (OpEx)** | Staff/FTEs, hardware maintenance, software maintenance/support, power + cooling, floor space/colo, network/telecoms, backup/DR operations, cloud consumption (compute/storage/network/egress/managed services), **change/customisation/development**, training (recurring), carbon footprint | Staff and change are usually the two largest lines (✅ Cabinet Office checklist; ⚠-structural on relative size); cloud OpEx is elastic — growth is a first-order sensitivity (§6, §8) |
| **Hidden — end-user** | User time: waiting, self-support, workarounds, learning; diminished performance/productivity | The Gartner PC-era finding (~80% of PC TCO in support/end-user, ⚠-structural); usually the largest unbudgeted line |
| **Hidden — downtime** | Outage/failure costs: lost revenue, SLA penalties, remediation labour, reputational damage | Price the availability target you actually need — the nines math and "cost of an extra nine" rule in [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §2.3–2.4 |
| **Hidden — training** | Initial + recurring training as staff/features turn over | Explicit Cabinet Office line (✅) |
| **Hidden — decommissioning / EOL** | Retirement, disposal, data/user migration *out*, exit costs, data-sovereignty compliance | UK definition includes retirement/disposal/migration but not replacement (✅); the most commonly zeroed line — exit cost is contract reality |
| **Hidden — migration** | Data + user migration in (acquisition) and out (EOL) | Migration is where "cheap" options hide cost — the [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) §9 pattern |
| **Hidden — compliance** | Audit, licence tracking, under-licensing true-up, unplanned compliance expenditure | Explicit Cabinet Office indirect cost (✅) |
| **Hidden — SLA assurance** | Third-party costs to guarantee SLA targets for business-critical systems | Explicit Cabinet Office indirect cost (✅) |
| **EOL — data centre** | Power decommissioning, DC space release, equipment disposal, carbon/Green Mark obligations | Singapore angle: DC capacity and green criteria — [singapore_data_centres_guide.md](singapore_data_centres_guide.md) §2 |

---

# 4. The Models

The taxonomy (§3) is the *what*; the models are the *comparisons* the discipline is actually deployed on. Three canonical models dominate the literature and the bank's decision room: on-prem-vs-cloud, perpetual-vs-subscription, and mainframe-vs-cloud.

## 4.1 On-Prem vs Cloud (verified — the 3–5-year horizon)

**The canonical TCO model: compare running a workload on owned infrastructure (or colocation) against renting it from a hyperscaler, over a 3–5-year horizon.** The horizon is verified: the on-prem-vs-cloud TCO literature consistently uses "a defined period, typically 3 to 5 years" (✅ professioncalculators.com; ✅ the TCOIQ/vicinity/servnet calculators all frame 3–5 years; ✅ the opsolute AWS-TCO guide's step 1 explicitly amortises server hardware "over 3-5 years"). Five years is the standard because it covers a hardware refresh cycle, long enough for cloud's recurring bill to compound and for on-prem's capex to amortise, and short enough that forecasting stays credible.

**The shape of the comparison:**

- **On-prem pays early, cloud pays always.** On-prem concentrates cost in year zero (CapEx) and then runs at a steady, largely *fixed* OpEx; cloud front-loads a small migration cost and then pays a *variable* OpEx that grows with usage. The cross-over point — where cumulative cloud spend passes cumulative on-prem spend — is the single most informative number in the model (the vendor calculators all surface it).
- **Utilisation is the swing factor.** On-prem hardware is paid for whether it runs at 10% or 90%; cloud bills only what runs. A steady, high-utilisation workload can be cheaper on-prem; a variable, spiky, or low-utilisation workload is usually cheaper in cloud. Every serious comparison makes utilisation the headline sensitivity (§6.2, §8).
- **The vendor calculators encode vendor agendas.** The AWS Pricing Calculator / TCO tools (✅ AWS docs) and their Azure/GCP equivalents (⚠-structural) are useful for the cloud side of the ledger and systematically optimistic about the on-prem side (utilization, staff loading, power). The discipline: build your own model, use the calculators as input sources, and challenge every number they supply — §6.1.
- **The post-decision half is FinOps.** Once the cloud bill is real, [finops_guide.md](finops_guide.md) (commitments, right-sizing, showback/chargeback, §11 for banking) governs it; the TCO model is the pre-decision estimate that FinOps will hold the estate against.
- **The §8 worked example** is this model, worked with numbers, in a Cymbal Bank setting: at face value cloud wins the 5-year comparison; under the marginal-cost (sunk facilities, marginal staff) view on-prem wins. Same arithmetic, different assumptions — which is the whole lesson.

## 4.2 Perpetual vs Subscription (verified — structure; figures flagged)

**The licensing-shape model: buy the software once (perpetual licence) and pay annual maintenance, versus rent it (subscription/SaaS) year after year.** This model is ⚠-structural in its details — the underlying structure is universally documented, but the specific percentages (typical maintenance is commonly cited at ~18–22% of licence fee per year ⚠) could not be pinned to a single authoritative public source this pass, and vendor licensing has fragmented into consumption/cloud-commitment models that blur the old binary.

**The shape of the comparison:**

- **Perpetual:** a large upfront licence (CapEx, capitalised), then an annual maintenance/support stream (~18–22% of list, ⚠-structural) that buys upgrades and support *forever*. Over a long horizon with a stable user base, the licence amortises and the TCO flattens — ownership gets cheaper per year as it ages.
- **Subscription:** a recurring fee (OpEx) covering licence, support, and usually upgrades and often the managed platform. Total spend is linear and predictable; there is no amortisation hump; the cost continues forever and rises with seats/usage.
- **The crossover:** over a short horizon (1–3 years) subscription usually wins (no big upfront); over a long horizon (5+ years) perpetual usually wins *if* the software is stable and the maintenance stream doesn't climb — and the vendor's pricing power (maintenance escalation, forced upgrades) is the risk that flips it. This is why horizon (§6.4) is a decision-shaping assumption, not a technicality.
- **The modern twist:** cloud-commitment pricing (reserved instances, savings plans, enterprise agreements) is subscription with a prepayment — and FinOps manages exactly that trade-off ([finops_guide.md](finops_guide.md) §6: RIs vs Savings Plans). The database-licensing angle is worked in [cockroachdb_guide.md](cockroachdb_guide.md) §8.4 (the Apache→BSL→CCL journey — the vendor moving the goalposts mid-ownership is the classic perpetual-licence risk) and the registry-licensing angle in [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) §9 (zero licence + full ops burden vs paid licence + managed burden — the TCO is not the licence line).
- **The banking rule:** perpetual buys *optionality and a known ceiling*; subscription buys *cash-flow flexibility and vendor alignment*. The TCO model's job is to price the ceiling and the flexibility, not to pick a favourite.

## 4.3 Mainframe vs Cloud (verified in structure)

**The estate-scale model: compare keeping a workload on the mainframe against migrating it to cloud (or to distributed on-prem).** This is the banking model — the mainframe estate is the industry's largest single technology cost and its largest single TCO decision (cross-ref [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) §3.2 and §8: mainframe cores run "a startling share of the world's banking," with "extreme reliability... and a fully amortized cost base" on the plus side and "hardware, MIPS licensing, specialist staff — high and rising" platform costs on the minus).

**The shape of the comparison:**

- **The run-rate leg:** mainframe TCO is priced per MIPS/capacity: hardware rental or owned cost, IBM software licensing (the infamous MIPS-based pricing — the more you use, the more you pay), maintenance, and scarce COBOL/z-OS staff. It is a *rising* run-rate — "the law of diminishing returns in mainframe tuning and enhancement" (✅ Accenture, *Reframe Your Mainframe*: the benefits of cloud plus diminishing returns in mainframe tuning "are pointing to the need for an inevitable migration away from the mainframe for core functionality").
- **The migration leg:** migration has a large one-off cost (replatforming COBOL, data migration, integration, testing, parallel run) and a *multi-year* profile (a big-bang cutover is a classic failure mode; phased migration spreads cost and risk). Vendor-published figures are ⚠ — one 2026 vendor survey cites mainframe-to-cloud migration costs of $3M–$45M per programme and "287% budget overruns" (⚠ entrans.ai vendor blog — flagged; the order of magnitude is plausible, the precision is not independently verifiable).
- **The honest finding (⚠-structural, consistent across the banking literature):** the TCO comparison rarely shows cloud "cheaper" for a *steady, fully-loaded, high-throughput* mainframe workload at face value — the mainframe's cost base is fully amortised. It shows cloud/moderisation cheaper on the *trajectory*: when the estate is growing (MIPS pricing punishes growth), when the workload mix changes (API/real-time vs batch), when staff costs are rising, and when the innovation backlog (what the mainframe *cannot* do) is priced as opportunity cost. This is exactly why the [business_case_development_guide.md](../management/business_case_development_guide.md) §8 discipline — compare trajectories, not snapshots — is the correct frame.
- **The bank-series context:** Crédit Agricole's group runs a mainframe-class estate (⚠-structural, per [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) — the mutualized IT subsidiary, the Euro Information heritage); the capital-markets side ([../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md)) carries the same legacy in its trading/back-office stack; Deutsche's legacy-consolidation case (bank series) is the same model with a different flag. §7 works the banking angle in depth.

## 4.4 The Models Table

| Model | Framing | Costs captured | Notes |
|---|---|---|---|
| **On-prem vs cloud** | Owned/colo infrastructure vs hyperscaler rental, **3–5-year horizon** (✅) | CapEx hardware + all on-prem OpEx (staff, power, floor, maintenance) vs cloud consumption + migration + support | Cross-over point is the headline number; **utilisation is the swing factor**; vendor calculators need their assumptions challenged (§6.1); §8 works it with numbers |
| **Perpetual vs subscription** | Buy + maintain vs rent per year | Upfront licence + annual maintenance vs annual recurring fee; upgrade/escalation risk | ⚠-structural on the ~18–22% maintenance figure; crossover depends on horizon; cloud commitments blur the binary — [finops_guide.md](finops_guide.md) §6; licensing-pivot risk in [cockroachdb_guide.md](cockroachdb_guide.md) §8.4 |
| **Mainframe vs cloud** | Estate-scale: keep vs modernise/migrate | MIPS-based run-rate (hardware/software/staff) vs migration one-off + distributed run-rate | Run-rate leg is rising; migration leg is large and multi-year; vendor migration figures ⚠; trajectory (not snapshot) comparison — cross-ref the banking cluster (§7) |
| **Build vs buy vs outsource** | The business-case guide's frame | The §8 case's three options, of which the buy option's TCO includes licence + support stream + integration + exit cost | Cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md) §4.1: the build option had the *lowest* operating cost but the *highest* TCO — the TCO discipline is what makes the comparison honest |
| **Vendor vs self-hosted** | The huggingface_vs_csghub pattern | Licence line vs ops burden + migration + second-registry cost | 3-year TCO sketch, figures flagged — [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) §9–10 |

---
# 5. The Methodologies

Three named methodologies dominate the canon: Gartner's TCO (the cost framework), Forrester's TEI (the value framework), and IDC's TCO/Business Value studies (the unit-economics framework). They are complements, not rivals — and all three feed the [business-case](business_case_development_guide.md) machinery rather than replacing it.

## 5.1 Gartner TCO (verified in origin; flagged on detail)

**Gartner's TCO is the originator and the de-facto industry standard for IT cost analysis** — Gartner's own materials describe it as "the industry-standard method for the financial analysis of IT and other enterprise costs" (✅ the "Defining Gartner Total Cost of Ownership" PDF, Gartner 2005-era, retrieved from the barsand archive this pass). What is verified:

- The **1987 origin** and the PC-era framing (§2.1) — ✅ EPA, UK Cabinet Office, Wikipedia/Gartner archive.
- The **core principle**: direct + indirect costs across the full life cycle, with the purchase price a minority share of the total (✅ Cabinet Office restates it; ⚠-structural on the ~20% figure).
- The **current research position**: Gartner's ongoing TCO research notes that TCO "principles are complex and hard to implement, incentivizing continued emphasis on unit price versus total cost" and that TCO is "resource-intensive, discouraging continuous adoption" (✅ Gartner's own public abstract, gartner.com document 5505795, extracted this pass).

**The flag:** the *detailed* Gartner TCO framework — its canonical category structure (the direct-costs/indirect-costs split, the specific cost categories and the published category weights) — is **not verifiable this pass**: Gartner's detailed methodology sits behind the paywall, and the secondary sources (including the archived Gartner PDF) describe the framework without a fully enumerable public category tree. This guide therefore treats Gartner TCO as: **origin verified, principle verified, category detail flagged** — and uses the UK Cabinet Office taxonomy (§3) as the concrete, publicly verifiable instantiation of the same idea (the Cabinet Office checklist is effectively the Gartner framework in government clothing, ✅ in substance).

## 5.2 Forrester TEI (verified)

**Forrester's Total Economic Impact™ (TEI) is the value-side methodology: cost, benefits, flexibility, and risk** — verified directly from Forrester's own methodology page this pass (✅):

- **Cost** — the TCO component (Forrester's cost analysis maps to this guide's §3 taxonomy).
- **Benefits** — the quantified business value: cost savings, revenue, productivity, risk avoidance — modelled over a multi-year horizon with a present-value calculation (the NPV machinery — cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md) §4.2).
- **Flexibility** — the option value: what the investment enables *later* (new capabilities, scaling, avoiding lock-in). This is TEI's distinctive contribution — classic TCO has no line for "what does this buy us the right to do next?"
- **Risk** — the range around the estimate: TEI applies confidence ranges to each benefit/cost line and models the downside (the sensitivity/tornado habit of §6.2, formalised).

**The TEI process (✅ Forrester):** commissioned studies built on due diligence, independent customer interviews and primary research, with a financial model; Forrester maintains editorial control and "positive results are not guaranteed." The methodological discipline — third-party interviews, explicit ranges, no bought conclusions — is the template for what a *bank-internal* TCO/TEI should look like when it wants to survive the CFO's pre-read.

**TEI vs TCO vs business case:** TCO prices the cost stream; TEI prices the whole economic impact (cost + benefit + flexibility + risk); the business case wraps either into the funding decision (NPV/IRR/payback against a hurdle rate). A TEI is essentially a business case with the option-value and risk-range machinery made explicit — which is why [business_case_development_guide.md](../management/business_case_development_guide.md) §6 (risk analysis) and §4 (financials) are the natural partners.

## 5.3 IDC (flagged)

**IDC's TCO work is the third pillar in name and the least verifiable in practice — flagged ⚠.** As established in §2.3: IDC publishes TCO and Business Value studies (frequently vendor-sponsored, measuring the business value and TCO of specific platforms), and its name is a standard reference in the TCO literature; but no single public, authoritative IDC methodology document could be extracted this pass — the methodology sits behind IDC's paywall, and the public artifacts are study summaries rather than a framework description. **Honest status: referenced, not re-verified.** The IDC contribution the literature does consistently show — and which this guide adopts as practice — is **unit economics**: expressing TCO as cost per unit of work (per transaction, per GB, per VM, per user) so that comparisons survive scale differences. §6.3/§8 use per-unit framing throughout.

## 5.4 The Methodology Table

| Methodology | Origin | Components | Notes |
|---|---|---|---|
| **Gartner TCO** | **Gartner, 1987** (✅ EPA/Cabinet Office/Wikipedia; ⚠ original note not public) | Direct + indirect costs across the life cycle; the purchase price is a minority share of total | Origin and principle ✅; **category detail flagged** (paywalled) — use the UK Cabinet Office taxonomy (§3) as the verifiable instantiation; Gartner's own research: TCO is "resource-intensive," adoption lags (✅ abstract) |
| **Forrester TEI** | Forrester, ~20+ years (✅; ⚠ exact year) | **Cost, benefits, flexibility, risk** (✅ Forrester, verbatim) | The value framework: TCO + benefit stream + option value + risk ranges; commissioned studies with independent interviews, editorial control (✅); the template for bank-internal rigor |
| **IDC TCO / Business Value** | IDC (1964–), ongoing studies (⚠ methodology not publicly pinned) | TCO studies, Business Value research, **cost-per-unit economics** | ⚠ Referenced, not re-verified; adopt the unit-economics habit (§6.3), not the paywalled framework |
| **Vendor calculators** | AWS Pricing Calculator + TCO tools (✅ AWS docs); Azure/GCP equivalents (⚠-structural) | Cloud-side cost estimation, on-prem baselining | Comparison engines with vendor agendas — input to challenge, not authority (§6.1); the §8 model's cloud side is built on this pattern |

---

# 6. The Modeling Practice

The methodology is the frame; the practice is the craft. Five habits separate a TCO model that decides from one that informs: the assumptions register, sensitivity/tornado analysis, discounting, an explicit horizon, and the auditability that makes all of the above survive finance's pre-read. (The business-case guide's §4.6 "financial-model design discipline" — named and sourced assumptions, a clearly marked inputs sheet, incremental flows, one discount rate, a frozen version — is the same list from the business-case side; cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md) §4.6.)

## 6.1 The Assumptions Register (verified in substance)

**Every input to the model is named, sourced, and versioned in an assumptions register — the model's inputs sheet, kept separate from the calculations.** This is ⚠-structural as a named practice (the consultant and finance literature is unanimous; the business-case guide's §4.6 is the repo's own statement of it, verified in that guide) and it is the single most important habit in the discipline, because:

- **TCO comparisons are decided by assumptions, not arithmetic.** The §8 worked example shows the same platform, the same horizon, the same numbers — and two different answers depending on whether facilities are costed fully-loaded or at marginal cost. The assumption, not the math, made the decision. An assumptions register makes that visible and challengeable instead of hidden.
- **It is the defence against vendor calculators.** The AWS/Azure/GCP calculators arrive with built-in assumptions (utilisation, staff cost, power, refresh cycle) that flatter the vendor. The register is where those defaults get overridden with *your* numbers — and where the override is documented.
- **It is the audit trail.** The CFO's team will ask "where does this number come from?" The register answers in one screen: assumption, value, source, owner, date. (The business-case guide's §4.6: "If finance cannot take the model over and audit it in an afternoon, it is not decision-grade yet.")

**Register entries that matter most in a platform TCO:** workload profile (transactions, growth rate, peak-to-average), utilisation assumptions, FTE loading (fully-loaded vs marginal), power/cooling cost per kW, hardware refresh cycle, licence escalation, discount rate, and the horizon (§6.4). Every one of these is a *decision* dressed as a number.

## 6.2 Sensitivity and Tornado (verified)

**Sensitivity analysis asks "how much does the answer move when an assumption moves?"; the tornado chart is its canonical visual — a ranked, horizontal bar chart of the output's swing across each input's range, most-impactful at the top.** Verified this pass: a tornado chart "displays the results of a one-way sensitivity analysis... which input variables have the greatest impact on your model's output, sorted from most to least impactful" (✅ riskpublishing.com); the same framing appears across the project-controls and decision-analysis literature (✅ somaprojectcontrols.com, decisionframeworks.com — "identify key uncertainties, evaluate risk drivers"). Two practice rules from that literature, worth quoting: the tornado's ranking is "only as good as the ranges chosen for its inputs, which makes the range assumptions the first thing worth reviewing" (✅ projectmanagementpathways.com) — the ranges are assumptions too, and belong in the register.

**In a TCO model the tornado almost always ranks the same suspects at the top:** utilisation/growth (cloud-side), staff loading (on-prem-side), the discount rate, and the horizon. When the tornado shows the decision flipping within plausible ranges (the §8 worked example does exactly this), the honest output is not a number but a *decision rule*: "cloud wins if utilisation stays under X%; on-prem wins above it." The business-case guide's §6 risk analysis is the same habit at the case level — cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md) §6, and note its §8 discipline: show the downside case as a headline number ("downside case remains positive at S$3.1M NPV").

## 6.3 Discounting (verified — the NPV connection)

**A TCO model that feeds a business case must discount its future cost streams to present value — the same machinery as NPV, applied to costs.** Verified (standard corporate finance; the repo's statement is [business_case_development_guide.md](../management/business_case_development_guide.md) §4.2–4.5 and [mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) §6):

$$PV = \sum_{t=0}^{T} \frac{C_t}{(1+r)^t}$$

where $C_t$ is the cost in year $t$ (a negative cash flow), $r$ is the discount rate, and $T$ is the horizon. The business-case guide's rules carry over verbatim: use the corporate discount rate agreed with finance (often WACC; the §8 case there uses a 10% hurdle — ✅ that guide's worked example), apply it **consistently across all options**, and show the undiscounted and discounted views side by side.

**What discounting does to a TCO comparison — the non-obvious part:** it systematically favours the option that *pays later*. On-prem pays its CapEx in year zero (discounted at 1.0 — full weight); cloud pays an annual bill that shrinks in present-value terms each year. In the §8 worked example, discounting deepens cloud's nominal advantage for exactly this reason (and the §8 model shows the arithmetic). Conversely, the *undiscounted* view matters too: a bank's affordability question ("can we pay for this from this year's budget?") is answered in nominal, undiscounted terms — the financial case cares about both. And a subtle trap: the discount rate is itself the highest-leverage assumption in the model after utilisation — the business-case guide's §4.5 warning ("a lower rate flatters long-dated benefits; a higher rate punishes them") applies to costs with the sign reversed. Green Book practice (3.5% social time preference, ⚠-framed per the business-case guide's own ledger) is the public-sector contrast to a bank's WACC/hurdle.

## 6.4 The Horizons (verified)

**The standard IT-infrastructure horizon is 3–5 years** (✅ verified §4.1: "a defined period, typically 3 to 5 years," across the cloud-vs-on-prem literature). The practice rules:

- **Match the horizon to the decision's cost profile, and say why.** 5 years covers a hardware refresh cycle and lets cloud's recurring bill compound — the two forces the comparison is actually about. 3 years flatters subscription options (less time for the perpetual licence to amortise); 7–10 years flatters ownership (more time to amortise) and is appropriate for data-centre, mainframe, and facilities decisions whose assets live that long (cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) — DC assets are 10–15-year assets).
- **The horizon is an assumption like any other** — it belongs in the register (§6.1), it gets a tornado bar (§6.2), and it must not be chosen to flatter one option. The honest move is to present the crossover year ("cumulative cloud spend passes cumulative on-prem in year 4") alongside the 5-year total, so the committee sees where the horizon choice bites.
- **Longer horizons degrade estimate quality** — forecast error compounds; the discipline's answer is to discount (which shrinks far-future error) and to state that beyond the horizon the comparison is qualitative (the TEI "flexibility" component — §5.2 — is where the beyond-horizon story lives).

## 6.5 The Practice Table

| Practice | What | Why it matters | Notes |
|---|---|---|---|
| **Assumptions register** | Named, sourced, versioned inputs sheet, separate from calculations | TCO decisions are made by assumptions, not arithmetic; the register is the audit trail finance pre-reads | ⚠-structural as a named practice; the business-case guide's §4.6 is the repo's statement; vendor-calculator defaults get overridden *here* |
| **Sensitivity / tornado** | One-way sensitivity across each input's range, ranked bars | Shows which assumptions decide the answer; the decision rule ("cloud wins below X% utilisation") replaces the false-precision number | ✅ verified (riskpublishing, somaprojectcontrols, decisionframeworks); ranges are assumptions too — register them; cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md) §6 |
| **Discounting** | Present-value the cost stream at the corporate rate (WACC/hurdle) | The NPV connection: discounted TCO feeds the NPV machinery; discounting favours the pay-later option — the on-prem capex gets full year-zero weight | ✅ standard corporate finance; business-case guide §4.2/4.5 (10% hurdle in its §8 case); Green Book 3.5% as the public-sector contrast (⚠-framed); show undiscounted view too — affordability is nominal |
| **Horizons** | Explicit 3–5-year horizon for IT decisions; longer for infra/DC/mainframe | The horizon decides which option's cost profile dominates; it is a decision, not a technicality | ✅ 3–5-year convention verified; state the crossover year; longer horizons → worse forecasts → discount + qualitative tail (TEI flexibility) |
| **Auditability** | Model finance can take over and audit in an afternoon | The survival criterion: a TCO that can't be pre-read is a TCO that won't be believed | Business-case guide §4.6, verbatim in substance |

---

# 7. The Banking Angle

The bank is where TCO modeling stops being an academic exercise: the estate is mainframe-scale, the data centres are Singapore-priced, and the decisions are board-level. Two angles matter most — mainframe-vs-cloud TCO and the data-centre TCO — plus the regulatory overlay that makes the model's assumptions (residency, exit cost, availability) non-negotiable.

## 7.1 The Mainframe-vs-Cloud TCO (verified — cross-ref the bank series)

**For a tier-1 bank, the largest TCO decision in the estate is the mainframe question** — the §4.3 model applied at estate scale. The repo's banking cluster supplies the context:

- [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) §3.2 (Era 1: Legacy Mainframe Cores): mainframe cores "still run a startling share of the world's banking"; the plus side is "extreme reliability... and a fully amortized cost base"; the minus side is "hardware, MIPS licensing, specialist staff — high and rising." The TCO translation: the mainframe's *current* cost is low and amortised, its *trajectory* is rising (MIPS pricing scales with usage; staff scarcity grows; change is slow and expensive).
- [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md): the Crédit Agricole group runs "a mainframe-class estate" via its mutualized IT subsidiary (⚠-structural — the hardware inventory is not published); Cymbal Bank's own markets estate sits alongside the group backbone. For the bank's own decision room, the mainframe TCO is not hypothetical — it is the standing estate question.
- The bank series generally: Deutsche's legacy-consolidation case, BNP's integration case, the Singapore banks' modernization programs — each is the same model with a different estate shape (cross-ref the series; [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) for the markets-side stack).

**The banking TCO structure for a mainframe-vs-cloud decision:**

1. **The run-rate leg (keep):** MIPS-based capacity cost, IBM software licence/maintenance, hardware rental or owned amortisation, specialist staff, facilities. Rising with usage and with the estate's age.
2. **The migration leg (move):** one-off replatforming (COBOL→distributed), data migration, integration, testing, parallel run, cutover — large, multi-year, and the source of the vendor-survey horror stories (⚠ vendor figures, §4.3: $3M–$45M programmes, "287% budget overruns" — order of magnitude plausible, precision not verifiable).
3. **The post-migration run-rate:** distributed/cloud cost for the same workload — usually lower per unit at variable utilisation, but *elastic* (it grows with the business), which is why the growth assumption is the first tornado bar.
4. **The regulatory overlay:** MAS outsourcing and data-residency requirements price into the *exit and decommissioning* lines (the hidden costs of §3.3) — the model must include the cost of getting data back and the compliance cost of the destination (cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) §2.6 and the regtech cluster; the [business_case_development_guide.md](../management/business_case_development_guide.md) §8 discipline of trajectory-vs-snapshot applies).

**The honest banking finding (⚠-structural, consistent across the industry literature and the Accenture *Reframe Your Mainframe* paper ✅):** the decision rarely turns on the *current* TCO of the mainframe — it turns on the *trajectory* (rising run-rate, shrinking skills, innovation backlog) versus the *one-off* migration cost and the *elastic* post-migration bill. A TCO model that answers only "is cloud cheaper today?" has asked the wrong question.

## 7.2 The Data-Centre TCO (verified — cross-ref singapore_data_centres_guide.md)

**The data-centre TCO is the facilities layer under every on-prem and hybrid option — and in Singapore it is a *capacity and regulatory* question as much as a cost question.** The repo's own DC guide is the context ([singapore_data_centres_guide.md](singapore_data_centres_guide.md) — cross-ref heavily):

- **The cost structure:** DC cost is dominated by power and cooling (the PUE multiplier), land/shell cost (Singapore: ~70+ DCs on 728 km², some of the world's most expensive land and power), connectivity (the island's interconnection advantage), staffing, and compliance. The guide's §1.6 ("Why Singapore wins — and what it costs") is the one-paragraph version of this TCO: the connectivity and regulatory advantages are real, and the power/land price is the bill.
- **The regulatory overlay:** the IMDA moratorium (May 2019) → lifting (Jan 2022, green criteria) → BCA Green Mark standards mean a DC TCO must price *green* capacity (water efficiency, renewable energy, carbon) and the *risk* of capacity rationing — the moratorium years showed that "you cannot build it" is the ultimate cost line. The green criteria also make the carbon line (§3.3, an explicit Cabinet Office TCO line) a *capacity* line, not just an ESG line.
- **The TCO model for a DC decision:** horizon 10–15 years (DC assets outlive every IT horizon — §6.4); the discount rate matters enormously over that span; the tornado bars are power price, PUE, and utilisation; the exit line is decommissioning and green-compliance. For a bank choosing between building, colocating, or hyperscaler-region residency, this is the facilities TCO that the on-prem leg of §8's model summarises into a single "facilities" line.

## 7.3 The Banking Table

| Angle | What it captures | Notes |
|---|---|---|
| **Mainframe-vs-cloud TCO** | The estate-scale §4.3 model: run-rate leg (MIPS, IBM licensing, specialist staff — rising) vs migration leg (large, multi-year, ⚠ vendor figures) vs post-migration run-rate (elastic) | The decision turns on trajectory, not snapshot (⚠-structural; ✅ Accenture's framing); cross-ref [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) §3.2/§8, [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md), the bank series |
| **Data-centre TCO** | Facilities layer: power/cooling (PUE), land/shell, connectivity, staffing, compliance; 10–15-year horizon | Singapore: power/land price + green criteria + the moratorium's lesson that capacity is a cost line — [singapore_data_centres_guide.md](singapore_data_centres_guide.md) §1.6, §2 |
| **Regulatory overlay** | MAS outsourcing, data residency, exit obligations priced into the hidden-cost lines (decommissioning, migration-out, SLA assurance) | The regulatory overlay is not a cost category; it is a *multiplier* on the hidden-cost categories (§3.3) — cross-ref the regtech cluster and the FinOps banking section ([finops_guide.md](finops_guide.md) §11: BCBS 239 cost attribution, chargeback across legal entities) |
| **FinOps in banking** | Post-decision: attribution to legal entity/business line, auditable allocation, landing-zone cost controls | The operating half of the cloud TCO once it is real — [finops_guide.md](finops_guide.md) §11; the pre-decision model's growth assumptions become FinOps' budgets |

---
# 8. The Worked Example

This section builds a complete TCO model with worked numbers: an on-prem-vs-cloud
5-year comparison in the house's own context. **All figures are illustrative
planning estimates (flagged ⚠) — they are internally consistent, built from the
§3 taxonomy, discounted per §6.3, and stress-tested per §6.2 — but they are NOT
quoted from any vendor price list or benchmark, and real numbers must come from
the bank's own rates.** The arithmetic is shown in full so the model can be
rebuilt, challenged, and re-run — which is the point.

## 8.1 The Scenario — a Cymbal Bank Decision

The bank must host a **trade-finance STP platform** (straight-through processing
of documentary trade transactions: ~2,500 transactions/day at peak, a nightly
EOD batch, a 12 TB database estate, a 99.95% availability target, MAS-grade
residency). Two hosting options reach the decision room:

- **Option A — On-prem:** run the platform on owned hardware in the bank's
  Singapore data centre (or colocation), with a second-site DR footprint.
- **Option B — Cloud:** run the platform on hyperscaler infrastructure in
  `ap-southeast-1` (Singapore region), multi-AZ, with managed services.

**Framing note:** this is a *sibling* of the [business_case_development_guide.md](../management/business_case_development_guide.md)
§8 worked example (the S$8.0M-capex trade-finance STP platform case), not the
same case: that case compares buy-vs-build-vs-outsource at the *platform
procurement* level; this case compares *infrastructure hosting* for the same
family of workload. The TCO model here produces the cost stream that a
business-case NPV would discount — the two exercises are the cost-estimation
layer and the decision layer of the same discipline (§1.2).

## 8.2 The Assumptions Register (illustrative — flagged ⚠)

| Assumption | Value | Basis / note |
|---|---|---|
| Horizon | 5 years (Y0–Y5) | The standard IT-infrastructure horizon (§4.1, §6.4); covers a hardware refresh cycle |
| Discount rate | 10% | Matches the business-case guide's §8 hurdle rate (✅ that guide); the corporate WACC would be the real input |
| Workload | 100 vCPU-equivalent, 12 TB usable storage, +5%/yr growth | Steady-state trade-finance workload with modest growth; growth is a tornado bar (§8.6) |
| On-prem hardware | 20 servers @ S$45K, SAN S$250K, network S$80K | S$1.23M hardware estate; refresh at Y3 (S$800K) |
| Maintenance | 15% of hardware + 20% of software per year | ⚠-structural industry norms, not vendor quotes |
| Staffing | 2 FTE @ S$150K loaded (ops + platform) | Fully-loaded view; the marginal view (1 FTE, sunk) is the §8.6 sensitivity |
| Facilities | S$150K/yr (power, cooling, floor) | Singapore power/land pricing — cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) §1.6 |
| Cloud rates | Compute S$420K/yr, managed DB S$180K/yr, storage S$90K/yr, network/egress S$60K/yr, enterprise support S$60K/yr, DR S$150K/yr | Aggregate of a vendor-calculator build (the AWS Pricing Calculator pattern, ✅ AWS docs) with RI/commitment discounts assumed |
| Migration | On-prem: S$400K integration; Cloud: S$350K migration | One-off; the cloud migration includes data + user migration (§3.3) |
| EOL | On-prem decommissioning S$50K at Y5 | The most commonly zeroed line (§3.3); includes data/user migration-out and disposal |

## 8.3 Option A — On-Prem, the 5-Year Model (S$K)

| Cost line | Y0 | Y1 | Y2 | Y3 | Y4 | Y5 | Total |
|---|---|---|---|---|---|---|---|
| Hardware (20 servers, SAN, network) | 1,230 | — | — | — | — | — | 1,230 |
| DB + middleware licences (perpetual) | 350 | — | — | — | — | — | 350 |
| Integration/implementation | 400 | — | — | — | — | — | 400 |
| DC setup (racks, power drops) | 120 | — | — | — | — | — | 120 |
| Maintenance (HW 15% + SW 20%) | — | 255 | 255 | 255 | 255 | 255 | 1,275 |
| Facilities (power, cooling, floor) | — | 150 | 150 | 150 | 150 | 150 | 750 |
| Staff (2 FTE, loaded) | — | 300 | 300 | 300 | 300 | 300 | 1,500 |
| Network/telecom | — | 60 | 60 | 60 | 60 | 60 | 300 |
| DR second site | — | 120 | 120 | 120 | 120 | 120 | 600 |
| Training (initial) | — | 40 | — | — | — | — | 40 |
| Mid-life refresh (Y3) | — | — | — | 800 | — | — | 800 |
| Decommissioning/EOL (Y5) | — | — | — | — | — | 50 | 50 |
| **Total** | **2,100** | **925** | **885** | **1,685** | **885** | **935** | **7,415** |

**Read of the table:** acquisition is S$2.1M — **28.3% of the nominal 5-year
total, and 34.3% of the discounted total** (year-zero spend carries no
discounting relief). The three largest lines over five years are staff
(S$1.5M), maintenance (S$1.28M), and facilities (S$750K) — none of which
appears on the procurement PO. This is the §1.1 claim, in the bank's own
numbers: **the purchase price is not the price.**

## 8.4 Option B — Cloud, the 5-Year Model (S$K)

| Cost line | Y0 | Y1 | Y2 | Y3 | Y4 | Y5 | Total |
|---|---|---|---|---|---|---|---|
| Migration (one-off) | 350 | — | — | — | — | — | 350 |
| Compute (100 vCPU-eq, RI-discounted) | — | 420 | 441 | 463 | 486 | 511 | 2,321 |
| Managed DB | — | 180 | 189 | 198 | 208 | 219 | 994 |
| Storage (12 TB + growth) | — | 90 | 94 | 99 | 104 | 109 | 496 |
| Network/egress | — | 60 | 63 | 66 | 69 | 73 | 331 |
| Enterprise support | — | 60 | 63 | 66 | 69 | 73 | 331 |
| DR (multi-AZ standby) | — | 150 | 158 | 165 | 174 | 182 | 829 |
| Training (initial) | — | 30 | — | — | — | — | 30 |
| **Total** | **350** | **990** | **1,008** | **1,057** | **1,110** | **1,167** | **5,682** |

(Note: consumption lines grow ~5%/yr from Y2; per-line rounding accounts for the
small differences from a straight 5% on the Y1 total. The full check is in
§8.5.)

**Read of the table:** year zero is only S$350K — **5.9% of the nominal total**.
The cloud option pays almost nothing upfront and an ever-present bill after;
its total (S$5.68M) is below on-prem's (S$7.42M) because the on-prem estate
carries staff, facilities, maintenance, refresh and DR that the cloud option
buys as managed services. The cloud bill *grows with the business* — which is
the assumption the whole comparison rides on (§8.6).

## 8.5 The Comparison — Nominal vs Discounted (S$K)

| Metric | Option A (on-prem) | Option B (cloud) | Delta (A − B) |
|---|---|---|---|
| 5-year nominal total | 7,415 | 5,682 | **+1,733 (cloud cheaper)** |
| PV at 10% | 6,123 | 4,360 | **+1,763 (cloud cheaper)** |
| Year-0 share of nominal | 28.3% | 5.9% | — |
| Year-0 share of PV | 34.3% | 8.0% | — |
| Cumulative position | Cloud leads from Y0 (cum A−cum B: 1,750 → 1,685 → 1,562 → 2,190 → 1,965 → 1,733) | No crossover — cloud never trails | — |

**The discounting effect (§6.3), demonstrated:** discounting *widens* cloud's
lead (1,733 → 1,763) even though cloud pays a bigger annual bill. Why? On-prem
pays S$2.1M in year zero at full weight; cloud's year-zero is S$350K and its
recurring bill shrinks in present-value terms each year. **Discounting
systematically favours the pay-later option** — the bank's affordability
question ("can we fund Y0?") is answered by the *nominal* column; the value
question ("what does this cost in today's money?") by the PV column. Both are
shown because both are asked.

## 8.6 The Sensitivity — the Tornado, Worked

| Assumption | Low case | Base | High case | Effect on the decision |
|---|---|---|---|---|
| Cloud consumption growth | 0%/yr → cloud total 5,300 | 5%/yr → 5,682 | 10%/yr → ~7,000 | At ~10% growth the cloud option's nominal total approaches on-prem's (7,000 vs 7,415); growth is the first tornado bar |
| Staff loading (on-prem) | Marginal: 1 FTE (total 6,815) | 2 FTE (7,415) | 3 FTE (8,015) | Each FTE moves on-prem by S$750K over 5 years; the fully-loaded-vs-marginal question is a *decision*, not a detail |
| Facilities (on-prem) | Sunk/marginal: S$80K/yr (view below) | S$150K/yr | New DC build (>> 150K) | Sunk facilities are the heart of the marginal-cost view — cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md) for what real SG facilities cost |
| Refresh cycle | No refresh in horizon (6,615) | Refresh at Y3 (7,415) | Refresh at Y2+Y4 | The refresh assumption alone is worth S$800K — vendors' on-prem models routinely hide or postpone it |
| Discount rate | 0% → A 7,415 / B 5,682 | 10% → A 6,123 / B 4,360 | 12% → A 5,924 / B 4,158 | Rate changes the *gap* (1,733 → 1,763 → 1,766), never the ranking, on the fully-loaded view |
| **The marginal view (combined)** | **On-prem at sunk facilities + 1 FTE + no refresh: 5,515** | — | — | **This is the case where on-prem wins the *nominal* comparison (5,515 < 5,682) — and still loses the PV comparison (4,688 vs 4,360), because year-zero capex carries full weight** |

**The tornado's verdict:** on the fully-loaded view, cloud wins across every
plausible range — the 5-year comparison is robust. The on-prem option only
wins when the bank believes the *marginal-cost* view (facilities and staff are
sunk — "we pay them anyway"). That belief is the single most consequential
assumption in the model, it is an *economic philosophy* not a number, and the
TCO model's job is to surface it — which is exactly what the assumptions
register (§6.1) is for. (And note the deep irony, worth stating in any
committee: the same marginal-cost logic that makes on-prem look cheap is what
makes mainframe estates look cheap — §7.1.)

## 8.7 From TCO to the Business Case

The TCO model produces the cost stream; the business case does the decision.
In the [business_case_development_guide.md](../management/business_case_development_guide.md) §8
worked example, the equivalent case was presented as: **"We recommend funding
Option 2 — the acquisition of a commercial trade-finance STP platform — at
S$8.0M capex, generating S$6.9M NPV over five years at a 10% hurdle rate, with
payback at 3.1 years."** The TCO discipline from this guide is what makes that
case's cost side honest: the S$8.0M capex is one line of a full TCO; the
support stream, the integration cost, the exit cost and the hosting option's
own TCO (§8.3–§8.4) must sit in the same model before the NPV is trusted. And
the business-case guide's own lesson — the *build* option had the lowest
operating cost but the highest TCO (§4.1 of that guide) — is this guide's §3
taxonomy doing its job: operating cost is not total cost.

## 8.8 The Lessons

1. **Assumptions decide, arithmetic reports.** The same platform, the same
   horizon, the same tables — and the ranking flips between the fully-loaded
   and marginal views. A TCO model is an argument about assumptions made
   visible; hide the assumptions and the model becomes propaganda.
2. **The purchase price is not the price.** 28% of on-prem's nominal 5-year
   cost is acquisition; the other 72% is maintenance, staff, facilities,
   refresh, DR and decommissioning — none of it on the PO.
3. **Discounting favours the pay-later option.** Cloud's nominal lead widens
   under PV because on-prem's year-zero capex carries full weight. Show both
   columns; they answer different questions (affordability vs value).
4. **The horizon is a decision.** Five years flatters neither option much
   here — but a 3-year horizon would flatter cloud and a 10-year horizon
   would flatter on-prem ownership. State the crossover logic, don't bury it.
5. **The hidden costs are the battleground.** Downtime pricing (the 99.95%
   target), training, migration-out, and decommissioning are where honest and
   dishonest models part company — the §3.3 catalogue is the checklist.
6. **Vendor calculators are inputs, not authorities.** The cloud side of
   §8.4 is a vendor-calculator build with the bank's own rates overlaid; the
   on-prem side had to be rebuilt from scratch because the calculators'
   on-prem baselines (utilisation, staff, refresh) flatter the vendor.
7. **The marginal-cost view is legitimate — and must be argued.** Sunk
   facilities and staff are a real economic position for a bank that already
   runs a data centre. What is not legitimate is holding it silently: it
   belongs in the register, with a tornado bar, and with the mainframe
   parallel (§7.1) acknowledged.
8. **TCO feeds the case; it is not the case.** The NPV, the hurdle rate, the
   options appraisal and the risk analysis live in
   [business_case_development_guide.md](../management/business_case_development_guide.md).
   This guide's discipline makes the cost stream those metrics discount
   defensible — which is the entire job.

---

# 9. The Summary — the One-Page

**TCO modeling is the discipline of pricing the whole life of a system —
acquisition, operation, change, and end — so that the decision that spends the
money is made on the true cost, not the sticker price.** In one page:

1. **TCO is a financial estimate of the direct and indirect costs of a
   product or service over its full life cycle** (✅ verified) — the purchase
   price is typically a small fraction of the total (✅ UK Cabinet Office).
2. **The discipline was popularized by Gartner in 1987** for PC costs (✅
   EPA/Cabinet Office/Wikipedia; ⚠ the original note is not public) and grew
   into the IT-platform standard; Forrester's TEI (✅) extended cost into
   value (cost, benefits, flexibility, risk); IDC's TCO studies are the third
   pillar, flagged (⚠).
3. **The taxonomy is three buckets plus the hidden costs:** acquisition
   CapEx, operating OpEx, and the hidden set — end-user time, downtime,
   training, migration, decommissioning/EOL, compliance, SLA assurance, and
   carbon (✅ UK Cabinet Office checklist, §3).
4. **The canonical models:** on-prem-vs-cloud over 3–5 years (✅), with
   utilisation and growth as the swing factors; perpetual-vs-subscription
   (⚠-structural figures), where the horizon picks the winner; and
   mainframe-vs-cloud at estate scale, where the *trajectory* — not the
   snapshot — is the honest comparison (§4, §7).
5. **The methodologies:** Gartner's TCO (origin ✅, detail flagged), Forrester's
   TEI (✅), IDC (⚠), and the vendor calculators (✅ AWS; ⚠-structural others)
   — complements, not rivals, all feeding the business-case machinery
   (cross-ref [business_case_development_guide.md](../management/business_case_development_guide.md)).
6. **The practice is five habits:** a named-and-sourced assumptions register;
   sensitivity with a tornado chart (the ranges are assumptions too); NPV-style
   discounting at the corporate rate; an explicit, defended horizon; and a
   model finance can audit in an afternoon (§6).
7. **In banking, TCO is estate-scale and regulation-priced:** the mainframe
   run-rate versus the migration one-off (cross-ref the bank series and
   [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)),
   and the data-centre bill that Singapore's power, land and green criteria
   make real (cross-ref [singapore_data_centres_guide.md](singapore_data_centres_guide.md)).
8. **The worked example (§8) shows the discipline working:** same platform,
   same horizon, two answers — cloud wins the fully-loaded 5-year comparison
   (S$5.68M vs S$7.42M nominal; S$4.36M vs S$6.12M PV), and on-prem wins the
   *nominal* comparison only under the marginal-cost view (S$5.52M). The
   assumptions, not the arithmetic, made the difference.
9. **The honest model ends with a decision rule, not a number:** "cloud wins
   if utilisation stays under X%; on-prem wins above it; the marginal-cost
   view is the argument to have out loud."
10. **TCO is cost-only** (✅ UK Cabinet Office): it does not judge benefits,
    risk, or strategy — it supplies the cost stream that the business case's
    NPV/IRR/payback machinery turns into a decision
    ([business_case_development_guide.md](../management/business_case_development_guide.md) §4).

**The final word — 'the true cost':** every system has two prices: the one on
the purchase order, and the true cost — everything spent from acquisition to
decommissioning, in staff hours and downtime and training and migration and
power and the quiet years of maintenance that outlive every project plan. The
cheapest option is rarely the one that costs least; it is the one whose true
cost is *visible*. TCO modeling exists to make the true cost visible — and
challengeable — before the money moves. **The purchase price is not the price.
The true cost is.**

---

# The Glossary

- **TCO** — total cost of ownership; the discipline and the metric for the
  full life-cycle cost of an asset or service (§1).
- **Total cost of ownership** — a financial estimate of the direct and
  indirect costs of a product or service over its full life cycle (✅ §1.1).
- **CapEx** — capital expenditure: the one-time, capitalisable acquisition
  costs (hardware, licences, integration) (§3.1).
- **OpEx** — operating expenditure: the recurring run costs (staff,
  maintenance, power, consumption) (§3.2).
- **Acquisition cost** — the CapEx to obtain and stand up an asset: purchase
  price, licences, hardware, integration, migration-in, initial training
  (✅ Cabinet Office checklist, §3.1).
- **Operating cost** — the recurring OpEx to run an asset: staff, maintenance,
  support, power, floor space, change/customisation, cloud consumption
  (✅ Cabinet Office, §3.2).
- **Hidden cost** — the cost no invoice announces: end-user time, downtime,
  training, migration, decommissioning, compliance, SLA assurance, carbon
  (§3.3).
- **End-user cost** — user time spent learning, waiting, self-supporting and
  working around a system (✅ Wikipedia; §3.3).
- **Downtime** — outage and failure cost: lost revenue, SLA penalties,
  remediation, reputation; priced against the availability target
  (cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md)).
- **Decommissioning** — the EOL retirement, disposal, and data/user
  migration-out of an asset (✅ Cabinet Office; §3.3).
- **EOL** — end of life: the final phase of the cost life cycle; the most
  commonly zeroed TCO line (§3.3).
- **Gartner** — the research firm that popularized TCO for IT in 1987
  (✅ §2.1); its detailed framework is paywalled (⚠ §5.1).
- **Forrester** — the research firm behind Total Economic Impact™ (✅ §2.2).
- **TEI** — Total Economic Impact: Forrester's methodology — cost, benefits,
  flexibility, risk (✅ §5.2).
- **Total economic impact** — the full value frame (cost + benefit + option
  value + risk) of an investment, per Forrester (§5.2).
- **IDC** — International Data Corporation; publisher of TCO/Business Value
  studies; methodology not publicly pinned this pass (⚠ §2.3, §5.3).
- **On-prem** — on-premises: infrastructure owned and operated by the
  organisation in its own (or colocated) facilities (§4.1).
- **Cloud** — hyperscaler infrastructure and services rented on consumption
  (or commitment) terms (§4.1; FinOps half in [finops_guide.md](finops_guide.md)).
- **Mainframe** — the IBM z/OS-class estate; the banking legacy platform whose
  TCO decision is estate-scale (§4.3, §7.1).
- **Perpetual licence** — buy-once software with annual maintenance; the
  ownership model (§4.2; licensing risk in [cockroachdb_guide.md](cockroachdb_guide.md) §8.4).
- **Subscription** — recurring-fee software/SaaS; the rental model (§4.2).
- **Assumptions register** — the named, sourced, versioned inputs sheet of a
  TCO model (§6.1).
- **Sensitivity** — the practice of moving one assumption at a time to see the
  output move; the honest answer to false precision (§6.2).
- **Tornado** — the ranked bar chart of a one-way sensitivity analysis; the
  canonical TCO visual (✅ §6.2).
- **Discounting** — converting future costs to present value at the corporate
  rate; the TCO-to-NPV connection (§6.3).
- **NPV** — net present value: the business-case master decision rule that
  TCO cost streams feed; cross-ref
  [business_case_development_guide.md](../management/business_case_development_guide.md) §4.2.
- **Horizon** — the appraisal period, typically 3–5 years for IT (✅ §6.4);
  longer for DC/mainframe/facilities assets.
- **Data centre** — the facilities layer of on-prem TCO; in Singapore, a
  power/land/green-criteria constraint (cross-ref
  [singapore_data_centres_guide.md](singapore_data_centres_guide.md)).
- **Workload** — the unit of TCO comparison: the application/transaction set
  whose resource consumption the model prices (§8.2).
- **Migration** — moving data and users into (acquisition) or out of (EOL) a
  platform; where cheap options hide their cost (✅ Cabinet Office; §3.3).

---

# The Claims-Status Ledger

| Claim | Status | Source / note |
|---|---|---|
| TCO definition: direct + indirect life-cycle cost estimate | ✅ | Wikipedia; UK Cabinet Office in substance |
| Purchase price is a small fraction of lifetime cost | ✅ | UK Cabinet Office, verbatim ("typically only a relatively small percentage") |
| ~20% purchase-price share of PC TCO (Bill Kirwin/Gartner) | ⚠-structural | Gartner-lineage literature (Stratrix et al.); principle verified, percentage not pinned |
| Gartner popularized TCO in 1987 | ✅ (with flag) | EPA; UK Cabinet Office; Wikipedia citing Gartner's archive — the original 1987 note is not public |
| UK Cabinet Office taxonomy (acquisition/operation/EOL + indirect) | ✅ | HMG *Total Cost of Ownership: Things to Consider* V1.0, extracted this pass |
| Hidden costs: end-user, downtime, training, decommissioning/EOL, migration, compliance, SLA assurance, carbon | ✅ | Cabinet Office checklist + Wikipedia component list |
| Forrester TEI: four components (cost, benefits, flexibility, risk); 20+ years | ✅ | Forrester's own methodology page, extracted this pass |
| TEI launch year | ⚠ | Forrester says "over 20 years"; exact year not pinned |
| IDC TCO methodology | ⚠ | No single public methodology document pinned this pass; studies widely referenced |
| On-prem-vs-cloud horizon of 3–5 years | ✅ | Multiple cloud-TCO sources (professioncalculators, opsolute, vicnity, TCOIQ) |
| AWS Pricing Calculator / TCO tools | ✅ | docs.aws.amazon.com (*How AWS Pricing Works*) |
| Azure/GCP TCO calculators | ⚠-structural | Widely documented; not individually re-verified |
| Perpetual maintenance ~18–22%/yr | ⚠-structural | Common industry figure; not pinned to one source |
| Mainframe migration cost ranges ($3M–$45M; "287% overruns") | ⚠ | Vendor blog figures (entrans.ai); order of magnitude plausible, precision unverifiable |
| Mainframe run-rate rising; "fully amortized cost base" tension | ✅-in-substance | Accenture *Reframe Your Mainframe* + [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) §3.2 |
| Tornado = ranked one-way sensitivity bars | ✅ | riskpublishing.com, somaprojectcontrols.com, decisionframeworks.com |
| Discounting favours the pay-later option; WACC/hurdle practice | ✅ | Standard corporate finance; business-case guide §4.2/4.5 (10% hurdle in its §8 case) |
| Green Book 3.5% social time preference | ⚠-framed | Per the business-case guide's own ledger |
| §8 worked-example figures | ⚠ illustrative | Internally consistent planning estimates; arithmetic shown; NOT vendor quotes |
| DC cost structure and SG regulatory story | ✅ | [singapore_data_centres_guide.md](singapore_data_centres_guide.md) (its own ledger) |
| Crédit Agricole mainframe-class estate | ⚠-structural | [credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) |

*End of guide. The final word: **the true cost.***
