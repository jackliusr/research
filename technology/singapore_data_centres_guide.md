# Data Centres in Singapore: The SG Data-Centre Landscape — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Technology Research — Singapore Infrastructure / Data Centre series; the dedicated deep-dive on the Singapore data-centre landscape: market, regulation, operators, hyperscalers, the AI-DC wave, energy, and a bank's DC strategy
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** IMDA (moratorium, Green Data Centre Roadmap, DC-CFA press releases); operator sites (Equinix, Digital Realty, ST Telemedia Global Data Centres, Keppel DC, GDS, AirTrunk, Princeton Digital Group); AWS / Azure / GCP region documentation; The Business Times, CNA, The Straits Times, Data Centre Dynamics (DCD)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Market Overview — The SG DC Market](#1-market-overview--the-sg-dc-market)
2. [The Regulatory Story — Moratorium to Green Criteria](#2-the-regulatory-story--moratorium-to-green-criteria)
3. [The Operators — Who Runs Singapore's DCs](#3-the-operators--who-runs-singapores-dcs)
4. [The Hyperscalers — AWS, Azure, GCP in Singapore](#4-the-hyperscalers--aws-azure-gcp-in-singapore)
5. [The AI-DC Wave — AI-Ready Facilities and the Roadmap](#5-the-ai-dc-wave--ai-ready-facilities-and-the-roadmap)
6. [The Energy Constraints — Grid, Gas and Green Power](#6-the-energy-constraints--grid-gas-and-green-power)
7. [Worked Example — A Bank's DC Strategy in Singapore](#7-worked-example--a-banks-dc-strategy-in-singapore)
8. [Summary — Singapore's Data Heart in One Page](#8-summary--singapores-data-heart-in-one-page)
9. [Glossary](#9-glossary)
10. [Claims Status, References and Further Reading](#10-claims-status-references-and-further-reading)

### How to Read This Guide

This is the **dedicated deep-dive on data centres in Singapore** — the market, the regulatory story, the operators, the hyperscalers, the AI wave, the energy reality, and what it all means for a bank running its APAC platform from Singapore — in the `technology/` Singapore-infrastructure series. Several sibling guides carry adjacent depth and are cross-referenced inline:

- **The sovereign-AI angle** — [htx_ngine_guide.md](htx_ngine_guide.md) covers HTX NGINE, Singapore's sovereign AI infrastructure for the Home Team. NGINE is the *public-sector* face of the same story told here: where Singapore's AI compute physically lives, how it is regulated, and why the government treats DC capacity as strategic national infrastructure. Read the two together for the full picture (sovereign DCs vs commercial DCs).
- **The cloud-regions angle** — [cloud_providers_guide.md](cloud_providers_guide.md) is the companion deep-dive on cloud regions and availability zones; §4 here summarises the three hyperscaler regions *in Singapore* (AWS `ap-southeast-1`, Azure Southeast Asia, GCP `asia-southeast1`) and defers the architecture detail there.
- **The on-prem DC angle** — [nutanix_products_guide.md](nutanix_products_guide.md) and [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) describe the hyperconverged software that typically runs *inside* the colocation cages this guide talks about; [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) covers running LLMs on-prem — the exact workload driving the AI-ready DC wave in §5.
- **The cost angle** — [finops_guide.md](finops_guide.md) covers cloud cost governance; the energy-cost pressure in §6 is the physical-economics twin of that story.
- **The AI-compute angle** — [physical_ai_guide.md](physical_ai_guide.md) (NVIDIA hardware ecosystem) and [gpu_optimization_guide.md](gpu_optimization_guide.md) (GPU optimisation) describe the silicon that AI-ready DCs are built around; §5 of this guide is about where that silicon lives.
- **The SG public-sector angle** — [ica_systems_guide.md](ica_systems_guide.md) and [nec_asia_pacific_guide.md](nec_asia_pacific_guide.md) touch the same government ecosystem; [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) is the enterprise-side AI plumbing that a bank's SG DC strategy (§7) would terminate into.
- **The bank angle** — the `banking/` series (e.g. [dbs_software_systems_guide.md](../banking/dbs_software_systems_guide.md), [ocbc_software_systems_guide.md](../banking/ocbc_software_systems_guide.md), [standard_chartered_guide.md](../banking/standard_chartered_guide.md)) shows how local banks run their own DC estates; §7 builds the same exercise for a global wholesale bank.

**A note on numbers.** Singapore DC statistics are notoriously methodology-dependent: different trackers count different things (operational facilities vs projects, colocation buildings vs hyperscale campuses, MW of critical IT load vs total power). Where sources disagree, this guide says so explicitly and flags the figure. Claims marked ***(flag)*** could not be fully verified against primary sources at writing time.

---

## 1. Market Overview — The SG DC Market

### 1.1 The scale: ~70+ data centres on 728 km²

Singapore is one of the most data-centre-dense geographies on earth. The most commonly cited figure is **70+ operational data centres**, with roughly **1.4 GW of total capacity** — the Economic Development Board (EDB) itself states that Singapore "hosts more than 70 data centres with around 1.4 gigawatts (GW) of total capacity" (EDB, 2026). Independent trackers paint a consistent picture with different lenses:

- **70+ operational DCs** — cited repeatedly since 2022 (Ashurst, 2022: "more than 70 operational data centres as of 2022"; Credence Research, Jan 2025: "over 70 operational data centers"; Research & Markets, Jun 2025: "more than 70 data centers").
- **57 facilities / 35 operators** — datacenters.world's map-based count (2026), which counts buildings.
- **66 facilities / 47 operators** — DataCenterMap's count, which splits campuses into buildings.
- **110 data centres / 49 providers** — Baxtel's count (2026), which is the most inclusive (it adds planned and under-construction projects).

***(flag)*** The "~70+" count is the consensus headline figure, but the true number is anywhere from ~57 to ~110 depending on whether you count buildings, campuses, or projects. What is *not* disputed: per square kilometre, Singapore has among the highest DC density in the world, and its ~1.4 GW of capacity is the largest concentration in Southeast Asia.

### 1.2 The position: the region's data hub

Singapore's market position is the same one it holds in finance and shipping: **the regional hub**. It is the de facto data heart of Southeast Asia for several interlocking reasons:

- **Connectivity.** Singapore is a landing point for dozens of subsea cable systems (including the SEA-ME-WE series, APG, and newer systems such as Echo and Bifrost serving the region), making it the natural place to interconnect the region's traffic. Cable density is why carriers and content providers put network nodes here.
- **Carrier-neutral ecosystem.** A deep market of colocation providers (Equinix, Digital Realty, STT GDC, Keppel, and others) means a tenant can cross-connect to carriers, cloud providers, and financial venues in the same building — the classic "meet-me room" model.
- **Financial centre gravity.** With SGX, MAS-regulated banks and brokers, and a huge FX/rates/trade-finance community, Singapore pulls in the latency-sensitive workloads that demand colocation near the venues. This is the same gravity that made Singapore the region's FX hub.
- **Stability and rule of law.** Political stability, strong property rights, and a reliable power grid make Singapore the low-risk location for regional HQ data estates — even though it is costlier than neighbours.
- **The cloud anchor.** All three hyperscalers (AWS, Azure, GCP) run major regions in Singapore (§4), which makes the market self-reinforcing: cloud demand pulls in DC capacity, DC capacity pulls in more cloud workloads.

***(flag)*** "Regional hub" is well supported by sources (Credence: "key data hub in Asia"; multiple analyst reports call Singapore "Asia's digital hub" / "Southeast Asia's leading data centre hub"), but precise market-share figures vary: the Singapore DC market was estimated at ~US$1.3 billion in 2023, projected to ~US$3 billion by 2032 (~9.5% CAGR) — a third-party analyst estimate, not an official statistic.

### 1.3 The market at a glance

| Aspect | Description |
|---|---|
| **Scale** | ~70+ operational DCs; ~1.4 GW total capacity ***(flag: count 57–110 by methodology)*** |
| **Density** | Among the highest DC density per km² globally, on 728 km² of land |
| **Position** | Southeast Asia's data hub: subsea cable landing point, carrier-neutral ecosystem, financial-centre gravity |
| **Cloud anchor** | AWS `ap-southeast-1` (2010), Azure Southeast Asia (2010), GCP `asia-southeast1` (2017) all in-country |
| **Key customers** | Hyperscalers, global banks, trading firms, content providers, government |
| **Market value** | ~US$1.3B (2023) → ~US$3B (2032E), ~9.5% CAGR ***(flag: analyst estimate)*** |
| **Key constraint** | Land + power scarcity; ~94–95% gas-fired grid (see §6) |
| **Regulatory regime** | IMDA/EDB capacity allocation since 2019 (see §2) |

### 1.4 The market's shape: segments and a short history

The Singapore DC market is really four sub-markets with different economics:

- **Retail colocation** — cabinets and cages for enterprises, trading firms, and network providers. This is the Equinix/Digital Realty/STT GDC/Keppel heartland: high interconnection value, higher price per kW, short-to-medium leases.
- **Wholesale colocation** — large floor plates with power measured in MW, leased to hyperscalers, banks, and big enterprises for 5–10+ years. AirTrunk and GDS play here; the 2023 pilot capacity (80 MW) was wholesale-class.
- **Hyperscale/self-built** — campuses the cloud providers build and run themselves (AWS, Azure, GCP all run in-country facilities). These are the largest power consumers and the reason ~1.4 GW of capacity exists at all.
- **Edge / network nodes** — small, connectivity-dense facilities near cable landing stations and exchanges; the "last mile" of the subsea-cable story.

A compressed history explains why the market looks the way it does:

1. **1990s–2000s — the financial era.** Early purpose-built facilities appear (Digital Realty's Jurong site was one of the first; Equinix opened SG1 in the early 2000s) to serve banks and telecoms. Singapore's FX/rates markets and SGX make low-latency colocation a banking necessity.
2. **2010s — the cloud era.** AWS (`ap-southeast-1`) and Azure (Southeast Asia) both launch Singapore regions in **2010**; GCP follows in **2017**. Hyperscale demand turns Singapore from a banking-DC town into a cloud-DC town, and capacity roughly doubles across the decade.
3. **2019–2022 — the freeze.** The IMDA moratorium halts new supply (§2); demand that cannot wait flows across the Causeway to Johor.
4. **2022–2026 — the green/AI era.** Capacity returns under green criteria (2022), the 80 MW pilot is awarded (2023), the Green Data Centre Roadmap lands (2024), and DC-CFA2 opens the AI era (2025).

### 1.5 The Johor spillover and the wider regional map

The moratorium did not reduce Southeast Asian DC demand — it *redirected* it. The most visible effect is the **Johor (Malaysia) spillover**: across the Causeway, land is cheaper, power is more abundant, and construction is faster, so a wave of hyperscale campuses (including major AirTrunk, GDS, and regional players' builds around Sedenak and Iskandar) absorbed the demand Singapore could not host. Batam/Bintan (Indonesia) play a similar overflow role, and Jakarta has grown into the region's second hub.

The result is a **"SG+" twin-hub pattern** that any DC strategy must internalise:

- **Singapore** — the control plane: financial data, low-latency venues, HQ estates, regulated workloads, and the region's interconnection core.
- **Johor + regional metros** — the capacity valve: bulk, cost-sensitive, and AI-training workloads that Singapore's power rationing cannot host.

***(flag)*** The specific Johor build-out (which operators, which parks, exact MW) was not re-verified in this session; the *pattern* of moratorium-driven spillover is well documented in DCD, Business Times, and EDB-adjacent coverage from 2019–2022.


### 1.6 Why Singapore wins — and what it costs

The market position is not an accident of history; it is a maintained trade-off. Singapore is deliberately the **premium** location in Southeast Asia, and enterprises pay for that premium in four currencies:

- **What you buy.** Political and legal stability, zero natural-disaster risk, the region's densest connectivity, a deep talent pool, MAS-grade regulatory predictability, and a timezone (UTC+8) that overlaps the whole APAC trading day.
- **What it costs.** The highest power prices in the region (gas-linked, §6), expensive land, expensive labour, and a regulator that rations capacity (§2). DC build and operating costs in Singapore are typically meaningfully higher than across the Causeway ***(flag: indicative industry ranges — Johor builds are commonly cited at 30–50% cheaper on land+power+labour; treat as directional)***.
- **The decision matrix.** Choose Singapore when: latency to financial venues matters, data residency/regulation dominates, or the workload is the "control plane" of the region. Choose Johor/regional metros when: cost dominates, the workload is bulk compute or AI training that can tolerate a few more milliseconds, or Singapore capacity is simply not available in the CFA cycle you need.
- **The self-reinforcing loop.** Because the region's most demanding workloads (finance, cloud control planes, interconnection) sit in Singapore, the market keeps attracting the infrastructure those workloads need — and the regulator keeps rationing exactly enough supply to preserve the premium.

***(flag)*** Cost comparisons are directional industry estimates, not audited figures; validate current pricing in any real business case.

---

## 2. The Regulatory Story — Moratorium to Green Criteria

No market on earth regulates data centres quite like Singapore. The story is a three-act arc: **freeze (2019) → green light (2022) → green *and* AI light (2024–2025)**. It matters to anyone building or buying DC capacity here, because the regulator, not the land market, decides how much capacity exists.

### 2.1 The moratorium (May 2019, IMDA)

In **May 2019**, the Infocomm Media Development Authority (**IMDA**) announced that Singapore would stop accepting applications for new data centre construction — a **moratorium** on new DC development and on the release of further land for DC use, initially framed as a two-year pause to review strategy.

**Why.** Singapore's grid runs ~94–95% on imported natural gas with no domestic hydro or nuclear, and DCs were consuming a growing share of national electricity. The government's concern was threefold: (1) power capacity, (2) land scarcity, and (3) carbon commitments (Singapore has pledged net zero by 2050). The moratorium effectively said: *we cannot keep letting the market decide how much power and land DCs consume.*

***(flag)*** The announcement date is widely cited as May 2019 and the moratorium as "in place since 2019" by IMDA/EDB materials and press (DCD, Business Times). The exact day is less consistently reported; what is solid is 2019, IMDA, ~three-year duration.

**The freeze in practice.** From 2019 to mid-2022, no new DC construction was approved. Existing facilities continued to operate and be retrofitted, and demand that could not be satisfied in Singapore leaked to neighbouring markets — notably **Johor, Malaysia** (a flood of "SG+" builds) and Batam/Bintan, Indonesia. This spillover created the modern Southeast Asia DC map: Singapore the regulated core, Johor the overflow valve.

### 2.2 The lifting (January 2022, green criteria)

The moratorium did not end with a return to business as usual — it ended with **conditions**. In **January 2022**, the government completed its review and announced that Singapore would resume DC development **only for "green" data centres**: new facilities must meet stringent energy-efficiency criteria, including a **Power Usage Effectiveness (PUE) of 1.3 or better** (EDB: "The review was completed in January this year. The new criteria include having a power usage effectiveness (PUE) of at least 1.3").

Then came the machinery of allocation, in three steps:

1. **July 2022 — the pilot.** IMDA and EDB launched the **pilot Data Centre – Call for Application (DC-CFA)**, inviting operators to bid for a limited pool of new capacity under the green criteria. This was Singapore's answer to the moratorium: *calibrated, sustainable growth* instead of open-market growth.
2. **July 2023 — the awards.** ~**80 MW** of new capacity was provisionally awarded to four winners: **Equinix, GDS, Microsoft, and an AirTrunk–ByteDance consortium** (DCD, Jul 2023). Notable: **Digital Realty, despite ~three existing SG sites, missed out** — proof that incumbency no longer guarantees expansion rights.
3. **December 2025 — DC-CFA2.** A second call for application was launched offering **at least 200 MW** for **AI-ready, low-carbon** data centres, with even tougher conditions: PUE below 1.3 and **at least 50% green power** in the energy mix (see §5).

### 2.3 The green standards (BCA Green Mark and beyond)

The "green criteria" rest on a stack of standards:

- **BCA Green Mark** — the Building and Construction Authority's national green-building certification, which has **data-centre-specific rating criteria** (the Green Mark for Data Centres / Green Mark 2021 DC criteria). New DCs winning capacity are expected to achieve top-tier Green Mark ratings; the scheme covers energy efficiency, water efficiency, and sustainable construction.
- **Green Data Centre Standard (GDCS)** — a joint BCA–IMDA standard for DC energy efficiency, benchmarked against international frameworks; the operational companion to Green Mark's building-level certification.
- **SS 715:2025** — a new Singapore Standard for sustainable data centres (2025), referenced alongside DC-CFA2 ***(flag: standard number verified via OpenGov coverage of DC-CFA2, Dec 2025)***.
- **Green Data Centre Roadmap (May 2024)** — the umbrella policy: **at least 300 MW of additional capacity** through green pathways, two thrusts (accelerate DC energy efficiency & green energy adoption; catalyse cross-ecosystem collaboration), and a push for a **"tropical data centre" standard** (running at higher operating temperatures to cut cooling energy). The roadmap also flagged plans for **~1.5 GW of additional renewable energy imports by 2035** ***(flag: import target widely reported with the May 2024 announcement; treat as policy direction rather than committed contract)***.

### 2.4 The regulatory timeline

| Event | Date | Impact |
|---|---|---|
| **Moratorium announced (IMDA)** | May 2019 ***(flag: 2019 confirmed; exact day less consistently reported)*** | No new DC construction approved; DC demand spills to Johor/Batam |
| **Review completed; green criteria announced** | January 2022 | Lifting signalled; new DCs must meet PUE ≤ 1.3 and green standards |
| **Pilot DC-CFA launched (IMDA + EDB)** | 20 July 2022 | Competitive bidding for ~80 MW of new capacity under green criteria |
| **Pilot awards (~80 MW)** | July 2023 | Equinix, GDS, Microsoft, AirTrunk–ByteDance consortium win; Digital Realty misses out |
| **Green Data Centre Roadmap** | 30 May 2024 | +300 MW via green pathways; tropical DC standard; ~1.5 GW renewables imports target ***(flag)*** |
| **DC-CFA2 launched** | December 2025 | At least +200 MW for AI-ready low-carbon DCs; PUE < 1.3; ≥50% green power; SS 715:2025 |

### 2.5 What the regulation actually means

The net effect of §2 is that **DC capacity in Singapore is now an administered, scarce resource** — allocated by IMDA/EDB through periodic calls for application, rationed by power, land, and carbon, and priced into every colocation contract. Three practical consequences for enterprises:

1. **Lead times are structural.** Winning capacity in a CFA round, then designing, building, and commissioning, takes years. If your bank needs DC capacity in Singapore, the clock starts at the *call for application*, not at the lease signature.
2. **Green is a licence to operate.** PUE targets, Green Mark ratings, and green-power obligations are now contractual and regulatory givens — not marketing.
3. **The scarcity is real and directional.** ~80 MW awarded in 2023, ~200 MW announced in 2025, against ~1.4 GW existing capacity and AI demand growing faster than both (see §5). Singapore will stay tight; the overflow will keep flowing to Johor and beyond.


### 2.6 The carbon and ESG overlay

The green criteria are not an isolated regulatory quirk — they sit inside Singapore's broader climate-policy stack, and that stack is tightening:

- **Net zero by 2050.** Singapore's national pledge (and its 2030 Nationally Determined Contribution) is the umbrella commitment behind the DC policy. Every CFA round since 2022 has been a mechanism for growing the digital economy *within* the carbon budget.
- **Carbon tax.** Singapore operates a carbon tax that has been rising deliberately: from S$5/tCO2e (2019–2023) to S$25 (2024–2025), S$45 (2026), with a stated trajectory toward S$50–80 by 2030 ***(flag: trajectory per government announcements; exact schedule dates drift with budget statements)***. DCs feel this directly on diesel genset testing/use and indirectly through electricity prices (gas-fired generation passes the tax through). It is a structural, predictable cost escalator for DC tenants.
- **MAS and bank ESG.** MAS's supervisory expectations on climate disclosure (and the banks' own net-zero commitments) make *green* colocation procurement a governance matter, not a marketing one: expect to justify the PUE and green-power mix of your chosen facilities in internal and external ESG reporting.
- **Green leases and attributes.** New DC leases in the CFA era increasingly carry green-power obligations (renewable energy certificates, imported green power) and efficiency clauses. DC-CFA2's ≥50% green-power requirement will normalise this for every tenant, not just the operators.

The strategic line to remember: **Singapore is not anti-DC — it is anti-waste-DC.** The government wants the digital economy to grow; it is simply insisting that growth come from efficiency and green energy rather than from an ever-growing share of a gas-fired grid.


### 2.7 How other hubs regulate — the international comparison

Singapore's model looks less exotic when set against its peers — and it pioneered the approach others now copy:

- **Hong Kong** — no formal moratorium, but chronic land and power constraints (and 2023–2024 political uncertainty) effectively ration DC growth; new supply has mostly gone to the New Territories and to nearby hubs.
- **Japan** — no moratorium; growth is constrained by grid connection queues in Tokyo/Osaka rather than by policy, with government incentives pushing green DCs (flag: incentive schemes evolve frequently).
- **Malaysia** — deliberately pro-growth post-2022: Johor's DC boom is government-encouraged, positioning Malaysia as the region's *capacity* answer to Singapore's rationing (cross-ref §1.5).
- **Indonesia** — open-door for hyperscale around Jakarta and Batam; like Malaysia, monetising Singapore's scarcity.
- **European Union** — the EU Energy Efficiency Directive (2023+) introduced mandatory energy reporting for large DCs, the first comparable regulatory regime that treats DC energy as a compliance matter rather than a market matter.
- **United States** — the counterpoint: local moratoriums on new DCs have appeared (e.g., New York-area and other municipal pauses in the mid-2020s ***(flag: US examples are local and volatile — treat as illustrative)***), showing the "freeze" instinct is no longer unique to Singapore.

The takeaway: Singapore was first to turn the moratorium into a *green rationing system*, and its CFA design is now the reference model for "how to grow DCs inside a carbon and power budget."

---
## 3. The Operators — Who Runs Singapore's DCs

Singapore's DC estate is run by a mix of global colocation majors, Temasek-linked national champions, hyperscale developers, and newer pan-Asian players. Seven names carry the bulk of the institutional story.

### 3.1 Equinix — the interconnection king

**Who.** The world's largest colocation operator by revenue, headquartered in California, operating "IBX" (International Business Exchange) data centres globally.

**Singapore footprint.** Equinix runs **8 IBX facilities in Singapore, SG1–SG8**, making it the largest carrier-neutral operator in the market. The early sites (SG1 in Kaki Bukit, SG2/SG3 in Tai Seng) anchor Singapore's interconnection ecosystem; SG2 was built out to ~1,700 cabinets and SG3 to a maximum of ~5,000 cabinets (Equinix announcements, 2015) ***(flag: cabinet figures are dated (2015) but illustrative of scale)***. Under the 2023 pilot award, Equinix won capacity for a new build — **SG8** — extending the portfolio into the post-moratorium era.

**Why it matters.** Equinix's Singapore cluster is the de facto **meet-me room for Southeast Asia**: the SG IBXs host more networks, clouds, and financial venues than any rival, and Platform Equinix interconnects SG to its other APAC metros (Tokyo, Hong Kong, Sydney, Osaka). For a bank needing low-latency cross-connects to venues and other banks, Equinix SG is usually the default answer.

### 3.2 Digital Realty — the density player

**Who.** The world's largest data centre REIT (Real Estate Investment Trust), US-headquartered, with a deep Singapore history — Digital Realty's Jurong facility was **one of the first purpose-built data centres in Singapore**.

**Singapore footprint.** Roughly **11 facilities (SIN1–SIN11)** across the island, including SIN1 in Jurong (a seven-storey facility at 29A International Business Park, powering multi-rack through 1 MW+ deployments) and SIN11 at 3 Loyang Way (a ~177,000 sq ft, Tier III-designed facility with ~13.2 MW of critical IT capacity) ***(flag: facility counts and SIN11 specs per DCD/DataCenters.com listings; counts vary slightly by source)***.

**Why it matters.** Digital Realty combines colocation with cloud on-ramps (ServiceFabric interconnects) and a large financial-services client base. Its 2023 pilot miss was a market event — it applied for ~60 MW and lost out to the four winners — but the operator has since announced a **S$4.3 billion AI-focused investment push** in Singapore (2026) ***(flag: investment figure per AI-Dominance SG coverage, 2026; unverified against Digital Realty's own release at writing time)***.

### 3.3 STT GDC (ST Telemedia Global Data Centres) — the Temasek champion

**Who.** Singapore-headquartered colocation operator backed by Temasek (through ST Telemedia), one of the fastest-growing DC platforms in Asia Pacific, with a global footprint spanning Southeast Asia, India, China, and beyond.

**Singapore footprint.** **STT Singapore 1–7** — seven facilities in the city-state ***(flag: "7 facilities" is the commonly cited count; STT GDC publishes its Singapore sites as STT Singapore 1–7)***. The Singapore cluster is the HQ showcase for its regional play, and STT GDC has declared **AI readiness across its Southeast Asia data centres** (TelecomTV, 2025), positioning the SG sites for GPU-era tenants.

**Why it matters.** STT GDC is the Singapore-establishment operator: Temasek backing, government-linked credibility, and deep telco lineage. Its trajectory is also a barometer for the market — in 2026, reports surfaced of a **Singtel/KKR-led consortium nearing a deal for STT GDC at a >US$13 billion valuation** (Straits Times, citing WSJ) ***(flag: reported, not yet closed at writing time)***.

### 3.4 Keppel — the conglomerate's DC arm

**Who.** Keppel Data Centres, the data centre arm of Singapore conglomerate Keppel Corporation, plus **Keppel DC REIT** — Asia's first pure-play data centre REIT (SGX-listed).

**Singapore footprint.** **Keppel DC Singapore 1–9 (SGP1–SGP9)** — a portfolio spanning the city-state, centred on the **Keppel Data Centre Campus at Genting Lane** (82 Genting Lane), where SGP9 is being built as a **planned, AI-ready hyperscale facility** targeting hyperscaler and cloud tenants (DataCenterMap listing). The numbered series (SGP1–SGP8 in operation, SGP9 planned) gives Keppel one of the widest local portfolios. ***(flag: SGP1–SGP9 numbering is consistent with PeeringDB/DataCenterMap/Baxtel listings; exact per-facility MW not verified)***.

**Why it matters.** Keppel is the pure-play local champion — its REIT structure means listed-market visibility into Singapore DC economics, and its Genting Lane campus is one of the few purpose-built DC *campuses* in the country.

### 3.5 GDS — the China-giant expansion

**Who.** GDS Holdings, the largest neutral data centre developer-operator in China, expanding aggressively across Southeast Asia.

**Singapore footprint.** GDS entered Singapore as part of its regional push and scored a major win in the 2023 pilot: it was **one of the four winners of the ~80 MW pilot allocation** (alongside Equinix, Microsoft, and AirTrunk–ByteDance). In October 2024 it **acquired a warehouse site in Singapore for redevelopment** into a data centre (DCD), signalling conversion of the pilot capacity into a real facility. ***(flag: GDS's exact Singapore MW allocation within the 80 MW pool was not published in the sources reviewed; treat GDS's SG build-out as in-progress)***.

**Why it matters.** GDS represents the Chinese DC supply chain entering Southeast Asia's premium market — and its pilot win shows Singapore is willing to allocate scarce capacity to new entrants who bring genuine demand and capital.

### 3.6 AirTrunk — the hyperscale developer

**Who.** Australian-founded hyperscale data centre developer, backed by global infrastructure capital (a Blackstone/CPPIB-led consortium acquired AirTrunk in 2024 ***(flag: acquisition widely reported; unverified in this session's sources)***).

**Singapore footprint.** **SGP1 at Loyang** — a hyperscale campus described by AirTrunk as **Singapore's largest neutral data centre by megawatts**, with a campus capacity of **over 78 MW across multiple buildings** (DataCenterMap). The first phase launched around 2020–2021 with **StarHub as anchor tenant** and a ~S$450 million price tag (Baxtel) ***(flag: phase-1 launch date and anchor tenant per Baxtel; AirTrunk's own site confirms SGP1 open at Loyang)***. In the 2023 pilot, **AirTrunk won additional capacity as part of the AirTrunk–ByteDance consortium** — ByteDance (TikTok) being one of the clearest signals of AI-era demand in Singapore.

**Why it matters.** AirTrunk is the pure hyperscale play: big campuses, cloud/AI tenants, and the construction economics that incumbents struggle to match inside Singapore's constraints.

### 3.7 PDG (Princeton Digital Group) — the pan-Asian upstart

**Who.** Princeton Digital Group (**PDG**), founded 2017, a fast-growing pan-Asian data centre operator with a regional platform spanning Singapore, Indonesia, India, China, Japan, and Malaysia.

**Singapore footprint.** **SGP1** — PDG's Singapore facility ***(flag: PDG lists SGP1 in Singapore; specific MW and address were not re-verified in this session — treat footprint as "one flagship SG facility, expanded in phases")***. Singapore is PDG's HQ market and the anchor of its regional story.

**Why it matters.** PDG is representative of the new wave of Asian DC operators building regional platforms out of Singapore — the market's role as *launchpad* for the wider Southeast Asia build-out, not just a destination in itself.

### 3.8 The operators at a glance

| Operator | SG footprint | Notes |
|---|---|---|
| **Equinix** | 8 IBX (SG1–SG8) | Largest carrier-neutral interconnection hub in SG; SG8 secured via 2023 pilot |
| **Digital Realty** | ~11 facilities (SIN1–SIN11), Jurong + Loyang | One of the first DCs in SG; missed 2023 pilot; S$4.3B AI push announced ***(flag)*** |
| **STT GDC** | STT Singapore 1–7 | Temasek-backed national champion; AI-ready across SEA; >US$13B reported deal talk ***(flag)*** |
| **Keppel** | Keppel DC Singapore 1–9, Genting Lane campus | Asia's first pure-play DC REIT; SGP9 planned AI-ready hyperscale |
| **GDS** | Pilot award + warehouse site (redeveloping) | China's largest neutral DC operator expanding into SG ***(flag: MW allocation unpublished)*** |
| **AirTrunk** | SGP1 campus, Loyang, 78+ MW | Largest neutral DC by MW in SG; pilot winner with ByteDance |
| **PDG** | SGP1 | Pan-Asian operator HQ'd in SG ***(flag: footprint details light)*** |


### 3.9 Choosing an operator — what a tenant should look at

Once the market structure is clear, the practical question is *which* facility your workloads land in. The checklist below is what a bank's DC due diligence actually examines (and what the §7 worked example uses):

1. **Interconnection.** Count the meet-me rooms, carriers, and IXPs on site, and the cloud on-ramps (AWS Direct Connect, Azure ExpressRoute, GCP Interconnect) available in the building. Interconnect revenue is the real moat of the Equinix model; a building with three carriers and a building with forty are different products.
2. **Power density headroom.** Ask for the maximum kW per rack the facility can actually deliver today (not the marketing number), and whether it is liquid-cooling-ready. A legacy 10 kW/rack building cannot host an AI rack without major retrofit.
3. **Energy economics.** How is electricity priced and passed through? Is there a PUE guarantee with penalties? Are green-power attributes (RECs/imports) available and costed? Under §6, energy is 50%+ of the total cost of ownership.
4. **Resilience credentials.** Tier certification (Uptime Institute), N+1/2N power and cooling, backup runtime at full density, generator fuel contracts, and the site's real outage history. For MAS-regulated workloads this is audit fodder.
5. **Location physics.** Distance to venues/cable stations, flood and climate risk, electrical feeder diversity (are the two "diverse" sites on the same substation?), and physical security posture.
6. **Commercial structure.** Lease term, power escalation formula, expansion rights, and renewal risk. In a scarcity market, the renewal is the dangerous moment — the operator knows your exit cost, and power prices have risen since your original deal.
7. **Ownership model.** REIT-owned (yield-driven, predictable), developer-owned (growth-driven, more flexible on new builds), or telco-linked (bundled connectivity) — each behaves differently on pricing and capex decisions.

The one-line summary: **in Singapore, you are not renting floor space — you are renting allocated power, interconnection, and resilience under a regulator-imposed scarcity regime. Evaluate all three explicitly.**


### 3.10 Where things are: notable SG facility locations

A mental map of Singapore's DC geography helps when evaluating sites (addresses per public facility listings; the pattern matters more than any single address):

| District | Notable facilities | Character |
|---|---|---|
| **Kaki Bukit** (east) | Equinix SG1 | One of the earliest DC estates; older stock, strong connectivity legacy |
| **Tai Seng** (central-east) | Equinix SG2/SG3 | The established carrier-neutral belt; dense interconnection |
| **Jurong / International Business Park** (west) | Digital Realty SIN1 (29A IBP) | Industrial-west campus zone; one of Singapore's first DCs |
| **Loyang / Pasir Ris** (east coast) | AirTrunk SGP1, Digital Realty SIN11 | The modern hyperscale campus zone; newer builds, larger floor plates |
| **Genting Lane** (central) | Keppel Data Centre Campus (SGP series) | Purpose-built DC campus; SGP9 is the AI-ready hyperscale build |
| **Tampines** (east) | Keppel DC Singapore 2 | East-side DC estate; mixed colo and enterprise stock |

Two structural observations: (1) the estate clusters in a few **DC belts** (Tai Seng, Jurong, Loyang) shaped by land availability and power infrastructure — "diverse" sites for DR purposes should sit in *different belts*, ideally on different electrical feeders; (2) the newest, largest builds (AirTrunk SGP1, Keppel SGP9, the GDS redevelopment) are the AI-era stock — density and liquid-cooling capability concentrate in the post-2020 facilities.

---

## 4. The Hyperscalers — AWS, Azure, GCP in Singapore

Singapore is one of the few markets where all three hyperscalers operate flagship regions *inside* the country. That is the demand engine behind everything else in this guide.

### 4.1 AWS — `ap-southeast-1` (Asia Pacific, Singapore)

- **Region code:** `ap-southeast-1`, the **first AWS region in Southeast Asia**, launched **2010** (AWS global infrastructure documentation; region-listing sites confirm "since 2010").
- **Availability Zones:** 3 (the region started with 2 and expanded; 3 AZs today).
- **Notes:** Singapore is one of AWS's longest-standing APAC regions (only Tokyo/N. Virginia-era regions are older in AWS's footprint) — the 2010 launch predates the DC moratorium and shaped Singapore's cloud ecosystem. `ap-southeast-1` hosts major financial-services, gaming, and SaaS workloads; AWS runs its APAC edge, Local Zones, and Outposts ecosystems around it. For data-residency purposes it is the default "Singapore region" in every AWS architecture.

### 4.2 Microsoft Azure — Southeast Asia (Singapore)

- **Region name:** **Southeast Asia** (code `southeastasia`), launched **2010** — one of Azure's first four international regions (alongside East Asia, North Europe, West Europe).
- **Availability Zones:** 3 (Azure Singapore supports AZs; part of the 2019+ global AZ expansion).
- **Notes:** The longest-serving Azure region in the region, heavily used by Singapore's enterprise and government-linked workload base (plus M365/data-residency needs). Azure pairs `southeastasia` with `eastasia` (Hong Kong) for regional DR patterns ***(flag: exact AZ count for `southeastasia` verified as "3 AZs supported" per Azure documentation; confirm per-workload at deployment time)***.

### 4.3 GCP — `asia-southeast1` (Singapore)

- **Region name:** **asia-southeast1** (Singapore), launched **2017** — Google's entry point into Southeast Asia, with 3 zones (a/b/c).
- **Notes:** The youngest of the three Singapore regions, and the anchor of GCP's Southeast Asia story (followed by `asia-southeast2`, Jakarta, in 2020). GCP's AI/ML services (Vertex AI, TPUs, GPUs) make `asia-southeast1` disproportionately relevant to the AI-DC wave in §5 — Google also runs its own DCs in Singapore and has been a vocal proponent of the tropical-DC operating standard.

### 4.4 The hyperscalers at a glance

| Provider | Region (code) | Launched | Notes |
|---|---|---|---|
| **AWS** | Asia Pacific (Singapore) — `ap-southeast-1` | 2010 | First AWS region in SEA; 3 AZs; dominant cloud share in SG |
| **Azure** | Southeast Asia — `southeastasia` | 2010 | One of Azure's first international regions; 3 AZs; strong enterprise/GLC base |
| **GCP** | Singapore — `asia-southeast1` | 2017 | 3 zones; AI/ML-heavy; anchor for Google's SEA presence |

*(For the mechanics of regions, AZs, and DR patterns, see [cloud_providers_guide.md](cloud_providers_guide.md).)*

### 4.5 Why the hyperscalers matter to the DC market

The three regions are not just tenants of Singapore's DCs — they are the *reason* the market exists at its current scale:

- **Cloud pulls DC capacity.** Each region needs multiple facilities (hyperscale campuses run by the cloud providers themselves, plus colocation for edge and partner capacity). Hyperscaler demand is the single largest driver of the ~1.4 GW estate.
- **Cloud demand is regulated too.** When hyperscalers wanted *more* Singapore capacity after 2019, they had to go through the same IMDA/EDB allocation process — which is why Microsoft showed up as a direct winner of the 2023 pilot (80 MW), and ByteDance partnered with AirTrunk for AI capacity.
- **The AI wave runs through them.** The three regions are rolling out GPU instance families (AWS P/G-series, Azure ND-series, GCP A3/TPU v5) in Singapore — the cloud-side of the AI-ready DC story in §5.


### 4.6 Resilience and the wider APAC map

The three Singapore regions give excellent *in-city* resilience, but they are still one city. The resilience playbook for APAC workloads anchored in Singapore:

- **In-Singapore redundancy.** All three regions offer 3 availability zones/zones — AWS `ap-southeast-1` (3 AZs), Azure `southeastasia` (3 AZs), GCP `asia-southeast1` (3 zones). Deploying across AZs covers most MAS-level expectations for material outsourced functions (cross-ref [cloud_providers_guide.md](cloud_providers_guide.md) for the AZ mechanics).
- **The residual risk is city-level.** A region is a single metro: subsea cable cuts, city-scale power events, or (hypothetically) physical disruption take down every AZ at once. MAS-grade DR for critical functions therefore wants a **second geography**, not just a second AZ.
- **Regional DR targets.** The standard APAC pattern is Singapore primary + one of: AWS `ap-southeast-3` (Jakarta, launched 2021), AWS `ap-southeast-2` (Sydney), Azure's newer Southeast Asia neighbours (Indonesia, Malaysia regions, early 2020s), or GCP `asia-southeast2` (Jakarta, 2020) ***(flag: region launch years per provider documentation; verify current region list in [cloud_providers_guide.md](cloud_providers_guide.md))***. The Johor build-out (§1.5) adds a low-latency, near-Singapore option that is *not* a cloud region but is increasingly used as a colocation DR site.
- **Edge does not equal DR.** AWS Local Zones, Azure Edge Zones, and GCP edge PoPs in the region are latency extensions, not recovery sites — they share the same metro's fate.
- **The governance triangle.** For a bank: (1) colocation pair inside Singapore (physically and electrically diverse), (2) cloud across 3 AZs in-country, (3) one documented regional DR geography. That triangle is the de facto standard response to MAS outsourcing and BCP expectations — and it is exactly the shape of the §7 strategy.


### 4.7 AI services and GPU availability in the three regions

For the AI-DC wave (§5), what the three regions *offer* matters as much as where they are:

- **AWS `ap-southeast-1`** — GPU instance families (P- and G-series, including A100/H100-class) available in Singapore, alongside SageMaker and Bedrock for model workflows ***(flag: instance availability is dynamic — confirm current families at procurement time)***.
- **Azure Southeast Asia** — ND-series GPU VMs (A100/H100-class) and Azure ML in the region; Azure OpenAI service availability varies by region ***(flag: verify per-service regional availability)***.
- **GCP `asia-southeast1`** — GPU and TPU capacity (Vertex AI, A3/H100-class, TPU v5-generation), making Singapore one of GCP's key AI regions in Asia ***(flag: TPU/GPU capacity in SG is allocated and can be constrained in peak periods)***.
- **The practical consequence.** Data-residency and regulatory requirements (PDPA, MAS expectations, bank AI governance) increasingly demand that AI workloads run *in-country* — which pushes AI compute into exactly the Singapore regions and AI-ready colo discussed here. The catch: GPU capacity in Singapore is a subset of global capacity, and AI demand spikes (new model launches) can create regional scarcity — plan burst capacity and lead times accordingly (cross-ref [physical_ai_guide.md](physical_ai_guide.md) and [gpu_optimization_guide.md](gpu_optimization_guide.md) for the workload side).

---

## 5. The AI-DC Wave — AI-Ready Facilities and the Roadmap

### 5.1 What "AI-ready" means for a DC

An "AI-ready" data centre is one engineered for the physics of GPU clusters, which are different from traditional enterprise workloads:

- **Power density.** Traditional racks run ~5–15 kW; AI racks run **30–100+ kW** (NVIDIA NVL72-class racks push beyond). This changes power distribution, backup (UPS/batteries), and cabling at every level.
- **Cooling.** Air cooling tops out around 20–25 kW/rack; AI densities demand **liquid cooling** (rear-door heat exchangers, direct-to-chip, immersion). This is a structural retrofit, not an incremental one.
- **Network spine.** GPU clusters need fat, low-latency fabrics (InfiniBand/400G Ethernet) inside the DC — the facility must be built around the network, not the other way around.
- **Power per site.** AI facilities are measured in tens of MW per building, with campuses in the 100+ MW class. Under Singapore's allocation regime, that scale is exactly what the regulator rations.

***(flag)*** Density ranges are industry-standard figures (NVIDIA/industry whitepapers), not Singapore-specific statistics.

### 5.2 The AI-ready build-out in Singapore

The AI wave is visible across the operator landscape:

- **Keppel SGP9** — explicitly "designed to be AI-ready" as a hyperscale facility at the Genting Lane campus (DataCenterMap listing).
- **STT GDC** — declared "AI readiness in its datacentres across Southeast Asia" (TelecomTV, 2025), with the Singapore cluster as the flagship.
- **The 2023 pilot winners** — Microsoft (directly) and ByteDance (via AirTrunk) are AI-native tenants; the pilot was effectively Singapore's first AI-capacity allocation.
- **Digital Realty's S$4.3B investment push** — positioned explicitly around AI inference workloads (2026) ***(flag)***.
- **Bridge Data Centres** — announced a S$3–5 billion Singapore investment for AI-ready capacity (2026) ***(flag: company press release; unverified independently)***.

The common thread: every operator's 2025-era build is AI-first, because that is where demand — and the regulator's 200 MW DC-CFA2 allocation — points.

### 5.3 The roadmap — Green Data Centre Roadmap (2024) and DC-CFA2 (2025)

The policy framework for the AI wave is the **Green Data Centre Roadmap** and its follow-ons:

- **30 May 2024 — Green Data Centre Roadmap (IMDA).** The umbrella policy, with two thrusts: (1) accelerate DC energy efficiency and green-energy adoption; (2) catalyse cross-ecosystem collaboration. Target: **at least 300 MW of additional capacity** via sustainable pathways, plus a **"tropical data centre" standard** (run DCs hotter, cut cooling energy — Google has been a key partner in testing this), and a **~1.5 GW additional renewable-energy-imports target by 2035** ***(flag: imports target widely reported with the roadmap; policy-direction status)***.
- **December 2025 — DC-CFA2.** The second call for application: **at least 200 MW** for **AI-ready, low-carbon data centres**, with conditions that make the AI era explicit — **PUE below 1.3** and **at least 50% green power**, alongside the new **SS 715:2025** sustainable-DC standard (OpenGov/Mallesons coverage, Dec 2025) ***(flag: DC-CFA2 terms verified via Dec 2025 coverage; award outcomes not yet announced at writing time)***.

***(flag — important)*** A standalone "Singapore AI data centre roadmap" announcement dated **2025** could not be verified in this research session. The verified 2025 policy event is **DC-CFA2 (December 2025)**; the umbrella policy is the **Green Data Centre Roadmap (May 2024)**. If you have seen a 2025-dated "AI data centre roadmap" headline, it most likely refers to DC-CFA2 coverage or to the AI-ready framing around the 2025 standard release.

### 5.4 The AI-DC picture at a glance

| Aspect | Description | Notes |
|---|---|---|
| **AI-ready DC definition** | High power density (30–100+ kW/rack), liquid cooling, GPU-scale network fabrics | Industry-standard ranges ***(flag)*** |
| **AI-ready builds in SG** | Keppel SGP9; STT GDC AI-ready SEA DCs; Microsoft/ByteDance pilot capacity | Operator announcements |
| **Policy umbrella** | Green Data Centre Roadmap (30 May 2024): +300 MW, efficiency + green energy, tropical DC standard | IMDA |
| **Capacity allocation** | DC-CFA2 (Dec 2025): ≥200 MW for AI-ready low-carbon DCs, PUE < 1.3, ≥50% green power | ***(flag: 2025 "AI roadmap" headline unverified; DC-CFA2 is the verified 2025 event)*** |
| **Standards** | BCA Green Mark (DC criteria), BCA–IMDA GDCS, SS 715:2025 | ***(flag: SS 715 number via Dec 2025 coverage)*** |
| **Strategic angle** | Sovereign AI compute (HTX NGINE) + commercial AI DCs share the same scarce power/land pool | Cross-ref [htx_ngine_guide.md](htx_ngine_guide.md) |


### 5.5 The technology underneath AI-ready

"AI-ready" is not a label — it is a set of concrete engineering choices. The useful mental model is a ladder:

- **Cooling.** Air cooling tops out around 20–25 kW/rack. Rear-door heat exchangers stretch to ~30–40 kW. **Direct-to-chip cold plates** handle 50–100+ kW. **Immersion** (single- or two-phase) goes beyond 100 kW. AI facilities in Singapore's constrained land therefore skip straight to liquid cooling, which also happens to be the most PUE-friendly option — aligned with the green criteria. The **tropical data centre standard** (running inlet temperatures higher, toward ~40°C, cutting compressor work) is being piloted in Singapore with Google ***(flag: the Google/IMDA tropical-DC partnership is well documented; exact temperature ranges vary by pilot phase)***.
- **Power.** Higher-density racks mean higher-voltage distribution (400/415 V trending toward 800 V DC architectures), UPS and battery systems sized for GPU load *spikes* (with power-capping software to flatten them), and backup generators or battery storage sized at full AI density — a 50 kW rack needs 5× the backup of a 10 kW rack. On-site solar is negligible in Singapore; the green share comes from imports and RECs (§6).
- **Networking.** GPU clusters need fat, lossless fabrics (InfiniBand NDR/XDR, 400/800 GbE) and CLOS topologies. The facility must be engineered network-first: cable paths, optical plant, and cooling for switch rows are as critical as the compute rows. Cross-ref [physical_ai_guide.md](physical_ai_guide.md) (the silicon) and [gpu_optimization_guide.md](gpu_optimization_guide.md) (the workload side).
- **The tenant checklist.** Before signing AI-era capacity, verify in writing: maximum deliverable kW/rack, liquid-cooling readiness (and at what density), PUE *at load* (not at idle), backup runtime at density, and power-capping tooling. Operators who cannot answer these four in writing are not AI-ready.


### 5.6 The sovereign-AI lane

Singapore treats AI compute as *national infrastructure*, and there is a parallel, government-owned lane beside the commercial DC market:

- **HTX NGINE** — the Home Team's (MHA family of agencies) first enterprise-grade sovereign AI infrastructure, the flagship of the public-sector AI push (deep-dive: [htx_ngine_guide.md](htx_ngine_guide.md)). NGINE-class systems run on NVIDIA's newest silicon (B200-class per that guide) in classified locations — the sovereign-DC angle of this series.
- **National AI Strategy 2.0** — the umbrella policy; secondary sources cite ~S$740 million allocated for sovereign AI compute within it ***(flag: S$740M figure per secondary coverage (Introl, 2026); verify against official NAS 2.0 documentation)***.
- **The relationship to this guide.** Sovereign AI and commercial AI DCs draw from the *same* power pool, land, and CFA cycles — the government's own compute needs are part of why DC-CFA2 is explicitly AI-first. For enterprises, the sovereign lane matters in three ways: (1) it signals policy direction (AI capacity will keep being prioritised), (2) it absorbs scarce resources (competition for power), and (3) it sets standards (the tropical-DC and efficiency agenda) that commercial facilities must then meet.
- **The boundary.** Sovereign AI infrastructure is not a service enterprises buy; it is the state's own estate. Your bank's AI compute lives in the commercial lane — cloud regions and AI-ready colo — governed by the same regulatory clock that the sovereign lane helped set.


### 5.7 Who is actually buying AI capacity in Singapore

The demand side of the AI-DC wave, in rough order of size:

1. **The hyperscalers themselves.** AWS, Azure, and GCP are rolling out GPU fleets in the three SG regions (§4.7) and consuming the largest share of new AI capacity — Microsoft's direct 2023 pilot win is the clearest evidence.
2. **AI-native content platforms.** ByteDance (TikTok) — via the AirTrunk consortium in the 2023 pilot — is the flagship case of AI-scale demand that could not wait for a CFA round and still entered through one.
3. **Regional enterprises going AI.** Banks, insurers, and telcos running in-country model inference (compliance, fraud, customer AI) — the §7 category; volumes are smaller but contracts are sticky and regulatory demands are high.
4. **The public sector.** The sovereign lane (§5.6) — NGINE-class national infrastructure plus GLC and smart-nation workloads.
5. **Emerging AI labs and GPU cloud providers.** A newer cohort renting AI-ready space wholesale, often as part of regional expansion from China and elsewhere.

The pattern to note: **Singapore's AI demand is inference- and control-plane-heavy, while training gravitates to where power is cheaper (Johor, regional metros, or the hyperscalers' global fleets).** That split — Singapore as the AI *brain* of Southeast Asia, neighbours as the AI *muscle* — is the demand-side twin of the §1.5 "SG+" supply pattern.

---
## 6. The Energy Constraints — Grid, Gas and Green Power

Every story in this guide — the moratorium, the green criteria, the AI wave — traces back to one physical fact: **Singapore has no energy of its own and no room to grow one.** Understanding the energy side is the difference between understanding the market and merely knowing its names.

### 6.1 The grid reality

- **Gas dependence.** Roughly **94–95% of Singapore's electricity is generated from imported natural gas** (pipelines from Malaysia/Indonesia, LNG terminals). There is no domestic hydro, no nuclear, negligible wind, and solar is a small but growing fraction (a few percent of generation) ***(flag: ~94–95% gas is the widely cited Energy Market Authority range; exact share drifts with solar additions)***.
- **Tiny grid, big consumers.** Singapore's system peak demand is in the single-digit-GW range (roughly 8 GW-class as of the mid-2020s) ***(flag: approximate; EMA publishes exact figures)***. Against that, the DC estate alone is ~1.4 GW of capacity — data centres consume an estimated **~7–10% of national electricity**, and that share is climbing with the AI wave ***(flag: 7% was the long-cited IMDA-era figure; more recent estimates run higher — treat as range)***.
- **Land = power ceiling.** At ~728 km², there is no "remote desert" option in Singapore; power must be generated (gas), imported (renewables via regional grids), or bought as carbon offsets. All three are constrained or contested.
- **The strategic answer.** The government's response has been three-pronged: (1) import green power (targeting ~1.5 GW additional renewable imports by 2035 — from Laos, Malaysia, Indonesia and beyond) ***(flag: policy direction)***; (2) make DCs radically more efficient (PUE < 1.3, tropical-DC standards, liquid cooling); (3) ration capacity through the CFA process (§2). Grid import capacity itself is finite — new cross-border interconnectors are years in the making — so efficiency and allocation carry the load in the near term.

### 6.2 What the constraints do to DC economics

Energy constraints are not abstract for an enterprise buying DC capacity in Singapore:

- **Energy cost is a first-order P&L line.** Singapore electricity is among the most expensive in Asia (no domestic fuel, market-priced gas). For an AI rack at 50+ kW, the electricity bill can exceed the rack's hardware cost within a few years — this is the physical twin of cloud FinOps (cross-ref [finops_guide.md](finops_guide.md)).
- **Power is the scarce allocation, not floor space.** Colocation contracts in Singapore are priced and rationed by *watts per square metre*, and CFA winners effectively receive a power quota. Your lease is a power allocation in disguise.
- **Green power is becoming mandatory.** DC-CFA2's ≥50% green-power requirement means tenants will increasingly pay for renewable attributes (RECs, imports) as a contractual given — budget for it.
- **Cooling design is a competitive weapon.** Tropical-DC standards and liquid cooling directly cut the energy bill; the operators who win the CFA rounds are the ones who can prove the lowest PUE.

### 6.3 The energy picture at a glance

| Constraint | Detail | Implication |
|---|---|---|
| **Gas dependence** | ~94–95% of electricity from imported natural gas ***(flag)*** | Price and carbon exposure; no cheap domestic power |
| **Grid scale** | System peak ~8 GW-class; DC estate ~1.4 GW ***(flag: approximate)*** | DCs are a large, visible share of national demand (~7–10%) |
| **Renewables** | Solar is single-digit %; no hydro/nuclear; imports (Lao PDR, Malaysia, Indonesia) ramping | Green-power targets (≥50% for new AI DCs) depend on cross-border infrastructure |
| **Land** | 728 km²; no remote-siting option | Capacity is rationed by the regulator, not built on demand |
| **Efficiency mandate** | PUE ≤ 1.3 (2022 criteria); < 1.3 + ≥50% green power (DC-CFA2) | Efficiency is the licence to operate; tropical-DC/liquid-cooling standards are now strategic |
| **Cost** | High per-kWh prices; AI racks multiply the bill | Energy is a first-order cost in DC decisions; FinOps-style governance applies to watts too |


### 6.4 Energy markets, carbon, and what to budget

Understanding the energy *market* is the final piece of the cost picture:

- **Market mechanics.** Singapore's electricity runs through the Energy Market Authority-regulated wholesale market (Uniform Singapore Energy Price, USEP), with retail contracts from licensed retailers; DCs typically contract firm capacity at negotiated tariffs. Because ~94–95% of generation is gas-fired ***(flag)***, wholesale prices track global LNG and gas prices — the 2021–2022 energy crisis flowed straight into Singapore DC economics.
- **The cost formula.** Total site energy = **IT load × PUE × hours × blended $/kWh**, plus demand charges and carbon-tax pass-through. Efficiency (PUE) is therefore a *multiplier* on your largest cost, which is why the regulator weaponises it.
- **Worked magnitudes (all ***(flag: indicative)***).** A 10 MW IT-load colocation at PUE 1.3 draws ~13 MW; at a blended ~S$0.15–0.25/kWh that is on the order of **S$17–28 million per year in electricity alone**. A single 50 kW AI rack at ~S$0.20/kWh costs ~**S$88,000/year in energy** — roughly the annual salary of an engineer, per rack, before the hardware.
- **Budgeting guidance.** (1) Demand PUE guarantees with penalty clauses in the lease; (2) negotiate the electricity pass-through structure (fixed vs index-linked) deliberately — in a gas-price spike, an unhedged formula is a P&L surprise; (3) price green-power attributes explicitly; (4) meter per tenant and run energy showback to business lines, exactly like FinOps for cloud (cross-ref [finops_guide.md](finops_guide.md)); (5) model the carbon-tax trajectory into every 5–10 year DC business case, because it is legislated to rise.


### 6.5 Signals to watch on the energy front

The energy picture is not static. Five developments are worth tracking because each would change DC economics in Singapore:

1. **Green power imports.** Cross-border renewable deals (Laos–Malaysia–Singapore, Indonesia, and beyond) are the government's headline answer to the ~1.5 GW imports target ***(flag: policy direction; individual projects are at various stages of development and contracting)***. Every GW of import that lands is a GW of DC headroom.
2. **LNG and gas prices.** Because the grid is gas-fired, global LNG markets set Singapore DC electricity prices. The 2021–2022 spike demonstrated the exposure; hedging and fixed-price contracts are the mitigation.
3. **Solar + storage.** Rooftop and floating solar continue to grow (a few percent of generation) and grid-scale batteries are appearing — modest but compounding contributions that shave peak pricing.
4. **Nuclear feasibility.** Singapore launched nuclear feasibility studies in 2024–2025 (SMR technologies) ***(flag: feasibility-study stage; no commitment)***. If ever realised, it would be the single biggest structural change to the country's energy ceiling — but it is a 2030s-at-earliest prospect.
5. **ASEAN grid integration.** The long-horizon ASEAN Power Grid would let Singapore buy regional renewables at scale; timelines are measured in decades, not years.

The through-line: **near-term headroom comes from efficiency and rationing; import-driven green power arrives mid-term; nuclear and grid integration are long bets.** Every DC plan should be built on that ordering.

---

## 7. Worked Example — A Bank's DC Strategy in Singapore

### 7.1 The scenario: a global wholesale bank with an APAC hub

**Familiar context.** Our bank is a global wholesale bank (think Crédit Agricole CIB — the author's house) running its APAC business out of Singapore: FX and rates trading, trade finance, corporate banking, and a regional shared-services estate. Regulators (MAS), the markets (SGX), and the clients (regional corporates and banks) all sit within a few kilometres of each other in Singapore's central business district. The bank currently runs a mix of on-prem infrastructure in Singapore (Temenos/core-banking and market-data stacks — see the `banking/` series), a growing cloud footprint (cross-ref [cloud_providers_guide.md](cloud_providers_guide.md)), and is under pressure from two directions: **MAS outsourcing and resilience expectations** (Notice 644, business-continuity requirements) and **AI demand** (model training/inference for credit, KYC, and market analytics — cross-ref [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) and [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)).

**The question.** Where should the bank's compute physically live in Singapore, and how should the estate be shaped — colocation, cloud, or both — given everything in §1–§6?

### 7.2 The strategy: colocation for the core, cloud for the elastic, sovereign for the sensitive

**Tier 0 — Core production: colocation.** The trading and payment platforms stay in carrier-neutral colocation, because latency to SGX and to counterparties, and control of the physical estate, still matter:

- **Primary:** a cage in an **Equinix SG IBX** — interconnection to SGX, market-data feeds, and peer banks in the same building; cross-connects instead of WAN hops.
- **Disaster recovery:** a second facility, **geographically and electrically diverse** within Singapore (e.g., an **STT GDC** or **Keppel** site), to satisfy MAS BCP expectations; the two sites are the classic active/standby pair.
- **On-prem software:** the cage runs the bank's own stacks (hyperconverged infra per [nutanix_products_guide.md](nutanix_products_guide.md), core banking per the `banking/` series) — the DC is a *location*, not a *service*, in this tier.

**Tier 1 — Elastic and analytics: cloud.** Everything that scales with demand, or that is born in the cloud, goes to the three Singapore regions, chosen by workload shape:

- **AWS `ap-southeast-1`** — analytics, data lakes, and the bulk of modern app workloads; the region's 3 AZs give the AZ-level resilience MAS expects for material outsourced functions.
- **Azure Southeast Asia (`southeastasia`)** — enterprise platform (M365, identity, ERP-adjacent) and the GLC/public-sector-adjacent workloads; its 2010 tenure means deep ISV and system-integrator ecosystems.
- **GCP `asia-southeast1`** — data and AI/ML workloads (Vertex AI, BigQuery); where the bank's model building lives in the cloud.

**Tier 2 — AI: hybrid, density-driven.** AI is the new decision axis:

- **Training / heavy inference:** either cloud GPU instances in the three regions (fast, elastic, expensive) or **AI-ready colocation** if the bank runs continuous, large-scale training (the density and cooling physics of §5 favour liquid-cooled AI-ready space — e.g., Keppel SGP9-class facilities ***(flag: as-built availability and pricing must be validated commercially)***).
- **Sovereign/sensitive AI:** workloads touching regulated or government-adjacent data stay in-country by definition (all three regions are in Singapore, so residency is satisfied) — but the *sovereign* play (national AI infrastructure like HTX NGINE) is a public-sector lane, not a bank lane; cross-ref [htx_ngine_guide.md](htx_ngine_guide.md) for the boundary.

**The decision rule.** *Latency- and control-critical → colocation; elastic and analytics → cloud; density- and physics-bound → AI-ready colo or cloud GPU; regulated data → any in-Singapore option, documented per MAS outsourcing rules.* Cloud-first for new builds, colocation-first for the regulated core.

### 7.3 The design in one table

| Workload class | Where it lives | Why |
|---|---|---|
| FX/rates trading, payments, market data | Equinix SG colo (primary) + STT/Keppel site (DR) | Latency to SGX/venues; MAS BCP; cross-connects |
| Core banking, trade finance apps | Colo cages (hyperconverged on-prem stacks) | Control, integration with legacy, regulator comfort |
| Analytics, data lakes, web/API layer | AWS `ap-southeast-1` | Elasticity, 3 AZs, ecosystem |
| Enterprise platform (M365/identity) | Azure Southeast Asia | Tenure, ISV ecosystem, enterprise controls |
| Data engineering & ML experimentation | GCP `asia-southeast1` | Vertex AI/BigQuery strength |
| AI training / heavy inference | Cloud GPU (all three) or AI-ready colo | Density/cooling physics; cost vs control trade-off |
| Sensitive/sovereign-adjacent | In-Singapore only; documented residency | PDPA, MAS Notice 644, residency by construction |

### 7.4 The lessons

1. **Singapore DC capacity is rationed, so plan 2–3 years out.** The CFA process (§2) means new colo supply arrives in regulatory waves. Lock in lease/power commitments early; treat "we'll expand when we need to" as the most expensive sentence in your infrastructure strategy.
2. **Buy watts, not square metres.** Power allocation is the scarce unit. Size your colo requirement in kW and negotiate PUE/SLA economics on that basis.
3. **Green is contractual now.** PUE targets and green-power obligations flow down from DC-CFA2 into every lease. Budget for renewable attributes and efficiency-driven pricing.
4. **Two sites, one city, must be truly diverse.** MAS-grade BCP within Singapore means physical and *electrical* diversity (different substations/grid feeders if possible) — check your DR site's power path, not just its address.
5. **Cloud regions are your elasticity valve, not your DR plan.** The three hyperscaler regions give you AZ-level resilience, but a regional DR story still needs a second geography (e.g., `ap-southeast-3` Jakarta or `ap-southeast-2` Sydney) — see [cloud_providers_guide.md](cloud_providers_guide.md).
6. **AI changes the physics.** If the bank's AI ambition is serious, AI-ready colo (liquid cooling, 30–100+ kW/rack) is a different product from legacy colo; qualify it early and price the energy honestly (cross-ref [physical_ai_guide.md](physical_ai_guide.md) for the hardware reality).
7. **Watch the energy bill like FinOps.** In Singapore, watts are a P&L line; the same governance you apply to cloud spend (cross-ref [finops_guide.md](finops_guide.md)) should apply to DC energy — metering, showback, and rightsizing density.


### 7.5 Sizing and procurement exercise

To make the strategy concrete, here is the sizing arithmetic a Solution Architect would run (all figures ***(flag: indicative, for method not precision)***):

**Sizing.** Trading/venue tier: two sites × 200 racks × ~10 kW average = **~4 MW of colocation**. Core banking tier: cages inside those sites (shared floor plates, no extra MW — density packing). Analytics tier: cloud, roughly 2,000 vCPU-equivalent on AWS + M365 on Azure — zero colo MW. AI tier: cloud GPUs for experimentation (burst) + **~1 MW of AI-ready colo** for continuous fine-tuning/inference that must not leave the country. **Total colocation requirement ≈ 5 MW, with density-headroom requirements up to 30–50 kW/rack** for the AI slice.

**Procurement.** RFP against the §3.9 checklist; shortlist three operators (e.g., Equinix for the venue-facing primary, STT GDC or Keppel for the diverse DR site, one AI-ready provider for the AI slice). Timeline: 3–6 months for procurement, 3–9 months for fit-out and commissioning, while cloud capacity is live in weeks. Budget order-of-magnitude: colocation in Singapore runs roughly **S$2,000–4,000/kW/year** plus energy ***(flag: wide industry range; negotiate hard — scarcity cuts both ways)***; cloud spend is governed by FinOps (cross-ref [finops_guide.md](finops_guide.md)).

### 7.6 The risk register

| Risk | Mitigation |
|---|---|
| **Capacity scarcity** (CFA cycles constrain supply) | Commit early; track IMDA/EDB call-for-application calendar; size for 2–3 years of growth |
| **Power price spikes** (gas-linked wholesale market) | Fixed or capped energy pass-through; efficiency SLAs; green-power contracts |
| **Carbon-tax escalation** (legislated upward path) | PUE discipline; REC/import procurement; bake tax trajectory into business case |
| **MAS outsourcing/BCP changes** | Documented controls; the colo + 3-AZ + regional-DR triangle (§4.6) |
| **AI density outgrowing facility** | Qualify max kW/rack and liquid-cooling readiness in every RFP |
| **Single-city concentration** (SG is one metro) | One regional DR geography (Jakarta/Sydney-class) for critical functions |
| **Cable cuts / climate events** | Diverse sites, diverse cable paths, tested DR runbooks |
| **Regulatory change** (new CFA rules, standards) | Contractual flexibility (assignment/termination); stay current on SS 715 and Green Mark updates |


### 7.7 The 12-month action plan

Bringing the whole guide down to a plan a Solution Architect can actually run:

**Months 1–3 — Assess.** Inventory the current estate: classify every workload as *core* (latency/control-critical), *elastic* (cloud-appropriate), or *AI* (density-bound). Measure actual power density and PUE at existing sites — most estates discover 20–30% headroom from consolidation. Gap-analysis against MAS outsourcing/BCP expectations and the bank's ESG commitments.

**Months 3–6 — Procure.** Run the colocation RFP against the §3.9 checklist for three lanes: venue-facing primary (Equinix-class), diverse DR site (STT GDC/Keppel-class), and an AI-ready slice (liquid-cooling-qualified). Negotiate power pass-through structure, PUE guarantees with penalties, and green-power attributes. In parallel, have cloud teams design the AZ-aware architecture for the three regions (§4) and start FinOps metering (cross-ref [finops_guide.md](finops_guide.md)).

**Months 6–9 — Commit.** Sign leases with expansion rights; begin fit-out and commissioning (power path testing, cooling validation at target density); stand up cloud landing zones and network cross-connects between colo and cloud on-ramps.

**Months 9–12 — Migrate and prove.** Sequence migrations non-critical-first; run DR failover tests between the two colo sites and to the regional DR geography (§4.6); document the ESG record (Green Mark procurement, PUE achieved); train operations on AI-era practices (liquid cooling, power capping).

**Running in parallel all year.** Track the IMDA/EDB call-for-application calendar (the next CFA round decides your 2028 capacity); model the carbon-tax trajectory into the business case; and re-qualify the AI density numbers with every operator at least annually — in this market, the plan that waits a year is the plan that pays 30% more.

---

## 8. Summary — Singapore's Data Heart in One Page

**The market.** Singapore runs ~70+ data centres and ~1.4 GW of capacity ***(flag: count by methodology)*** on 728 km² — the densest, most connected DC market in Southeast Asia, anchored by subsea cables, a carrier-neutral colocation ecosystem, and three hyperscaler regions. It is the region's data heart: the hub every other Southeast Asian market orbits.

**The regulation.** The story is a freeze-then-ration arc: the **2019 IMDA moratorium** (no new builds; demand spilled to Johor), the **January 2022 lifting on green criteria** (PUE ≤ 1.3), the **July 2022 pilot DC-CFA** (80 MW, awarded July 2023 to Equinix, GDS, Microsoft, AirTrunk–ByteDance), the **May 2024 Green Data Centre Roadmap** (+300 MW via green pathways, tropical-DC standard, ~1.5 GW renewables-imports target ***(flag)***), and the **December 2025 DC-CFA2** (≥200 MW for AI-ready low-carbon DCs, PUE < 1.3, ≥50% green power). Capacity in Singapore is now an administered, scarce resource.

**The players.** Seven operators dominate the institutional story — **Equinix** (8 IBX, the interconnection king), **Digital Realty** (~11 SIN sites, the pioneer), **STT GDC** (Temasek-backed national champion), **Keppel** (DC REIT + AI-ready Genting campus), **GDS** (China's giant, pilot winner), **AirTrunk** (hyperscale SGP1, pilot winner with ByteDance), **PDG** (pan-Asian upstart) — plus the hyperscalers: **AWS `ap-southeast-1`** (2010, first in SEA), **Azure Southeast Asia** (2010, first international wave), **GCP `asia-southeast1`** (2017, AI/ML anchor).

**The AI wave.** AI-ready DCs (30–100+ kW/rack, liquid cooling, GPU fabrics) are the only game in new builds; the 2025 allocation (DC-CFA2) is explicitly AI-first. The sovereign-AI lane runs in parallel via national infrastructure like HTX NGINE — the same scarce power pool, a different customer.

**The constraint.** Everything is bounded by energy: ~94–95% gas-fired power ***(flag)***, an ~8 GW-class grid ***(flag)***, no domestic fuel, no land to spare. Green efficiency is the licence to operate; power, not space, is what you buy.

**The play for a bank.** Colocate the regulated core (Equinix + a diverse DR site), cloud the elastic layer across the three regions, qualify AI-ready space early, buy watts not square metres, plan on CFA cycles, and treat energy as a FinOps line.

**The final word.** Singapore is not just a market for data centres — it is **the region's data heart**: the place where Southeast Asia's cables, capital, and compute converge, rationed by power, disciplined by regulation, and now beating faster under the AI wave. Whoever understands the energy math and the regulatory clock will find Singapore the most dependable — and the most demanding — data home in the region.


### 8.1 The cheat sheet

| Dimension | The one-liner |
|---|---|
| **Market** | ~70+ DCs, ~1.4 GW, densest and most connected in Southeast Asia ***(flag: count by methodology)*** |
| **Regulation** | 2019 moratorium → 2022 green criteria (PUE ≤ 1.3) → CFA allocations (80 MW 2023, ≥200 MW 2025) |
| **Operators** | Equinix (interconnect), Digital Realty (density), STT GDC (Temasek), Keppel (REIT + AI-ready), GDS/AirTrunk (pilot winners), PDG (pan-Asian) |
| **Hyperscalers** | AWS `ap-southeast-1` (2010), Azure Southeast Asia (2010), GCP `asia-southeast1` (2017) |
| **AI wave** | AI-ready = 30–100+ kW/rack + liquid cooling; DC-CFA2 is AI-first; sovereign lane runs in parallel |
| **Energy** | ~94–95% gas-fired ***(flag)***; efficiency is the licence to operate; green power mandatory by 2025 criteria |
| **The play** | Colo the regulated core, cloud the elastic layer, qualify AI-ready early, buy watts not square metres, plan on CFA cycles |

---

## 9. Glossary

- **Data centre (DC)** — A physical facility housing compute, storage, and networking equipment, with power, cooling, security, and connectivity engineered for continuous operation.
- **DC** — Abbreviation for data centre (also "datacentre"); in Singapore policy documents, "DCs" is the standard shorthand.
- **Colocation (colo)** — A service model where a tenant rents space, power, and connectivity in a shared, carrier-neutral facility and runs its own equipment in cages or cabinets.
- **Singapore** — The city-state at the heart of this guide: ~728 km², ~70+ DCs, ~1.4 GW DC capacity, the region's data hub.
- **IMDA** — Info-communications Media Development Authority; Singapore's regulator for info-communications and media; co-architect (with EDB) of the DC moratorium, green criteria, and DC-CFA process.
- **Moratorium** — The 2019–2022 pause on new DC construction and land release in Singapore, announced by IMDA in May 2019.
- **Green criteria** — The post-moratorium conditions for new DCs: PUE ≤ 1.3 (2022), tightened for DC-CFA2 to PUE < 1.3 and ≥50% green power.
- **Green Mark** — BCA's national green-building certification, with data-centre-specific rating criteria; a key licence-to-operate for new DCs.
- **BCA** — Building and Construction Authority; Singapore's building regulator and custodian of the Green Mark scheme (co-publisher of the BCA–IMDA Green Data Centre Standard).
- **Equinix** — Global colocation leader; 8 IBX data centres in Singapore (SG1–SG8); the market's interconnection hub.
- **Digital Realty** — Global data centre REIT; ~11 SIN facilities in Singapore, including one of the country's first DCs (Jurong).
- **STT GDC** — ST Telemedia Global Data Centres; Temasek-backed Singapore colocation operator (STT Singapore 1–7).
- **ST Telemedia** — Singapore's state-linked technology holding company; majority backer of STT GDC.
- **Keppel** — Singapore conglomerate whose Keppel Data Centres arm runs Keppel DC Singapore 1–9 (including the AI-ready SGP9) and Asia's first pure-play DC REIT.
- **GDS** — China's largest neutral DC developer-operator; 2023 pilot winner in Singapore, building its first local facility.
- **AirTrunk** — Australian-founded hyperscale DC developer; operates SGP1 (Loyang, 78+ MW), Singapore's largest neutral DC by MW; 2023 pilot winner with ByteDance.
- **PDG** — Princeton Digital Group; pan-Asian DC operator founded 2017, HQ'd in Singapore (SGP1).
- **Princeton Digital Group** — See PDG.
- **AWS** — Amazon Web Services; operates the `ap-southeast-1` (Singapore) region, launched 2010, first AWS region in Southeast Asia.
- **Azure** — Microsoft's cloud; operates the Southeast Asia (`southeastasia`) region in Singapore, launched 2010.
- **GCP** — Google Cloud Platform; operates the `asia-southeast1` (Singapore) region, launched 2017.
- **ap-southeast-1** — AWS region code for Asia Pacific (Singapore); 3 availability zones.
- **Southeast Asia** — Azure's region name for Singapore (code `southeastasia`).
- **asia-southeast1** — GCP region code for Singapore; 3 zones.
- **Hyperscaler** — A cloud provider operating at massive global scale (AWS, Azure, GCP); in the DC context, the largest demand driver and tenant class.
- **AI-ready** — Describes a DC engineered for GPU-era physics: 30–100+ kW/rack densities, liquid cooling, GPU-scale networking.
- **AI data centre roadmap** — In this guide, the policy framework for AI-era capacity: the Green Data Centre Roadmap (2024) and DC-CFA2 (Dec 2025) ***(flag: no separately-verified 2025-dated "AI roadmap" announcement found)***.
- **Energy** — The binding constraint on Singapore DCs: ~94–95% gas-fired grid, ~7–10% of national electricity consumed by DCs, green-power mandates rising.
- **Grid** — Singapore's ~8 GW-class national power grid ***(flag: approximate)***; the physical ceiling on DC growth.
- **Cloud** — On-demand compute/storage delivered over the network from hyperscaler regions; the elasticity valve of the bank's DC strategy in §7.

---

## 10. Claims Status, References and Further Reading

### 10.1 Claims status

| Claim | Status | Basis |
|---|---|---|
| Moratorium announced 2019 (widely cited May 2019), IMDA | Verified (year); May 2019 widely cited ***(flag: exact day varies by source)*** | IMDA/EDB materials; DCD; Business Times |
| Review completed Jan 2022; green criteria incl. PUE ≤ 1.3 | Verified | EDB insight article ("review was completed in January this year… PUE of at least 1.3") |
| Pilot DC-CFA launched 20 Jul 2022 (IMDA + EDB) | Verified | IMDA press release; DCD (20 Jul 2022) |
| ~80 MW pilot awarded Jul 2023 to Equinix, GDS, Microsoft, AirTrunk–ByteDance | Verified | DCD (14 Jul 2023); TheTechCapital (17 Jul 2023) |
| Green Data Centre Roadmap 30 May 2024; +300 MW; 1.5 GW imports target | Verified (roadmap, 300 MW); imports target ***(flag)*** | Linklaters/techinsights (30 May 2024); JLL Q2-2025 |
| DC-CFA2 Dec 2025: ≥200 MW, AI-ready low-carbon, PUE < 1.3, ≥50% green power, SS 715:2025 | Verified (Dec 2025 coverage) ***(flag: awards pending)*** | OpenGov; Mallesons (Dec 2025) |
| ~70+ DCs; ~1.4 GW capacity | Verified (consensus); count range 57–110 ***(flag)*** | EDB (2026); Ashurst (2022); Credence (2025); Research & Markets (2025); datacentermap; datacenters.world; Baxtel |
| Singapore = regional data hub | Verified (qualitative consensus) | Credence ("Asia's digital hub"); analyst reports |
| Equinix SG1–SG8 | Verified (8 IBX; SG8 pilot-era) | Equinix announcements; DCD; datacenters.world |
| Digital Realty ~SIN1–SIN11 incl. Jurong + Loyang SIN11 (177k sq ft, 13.2 MW) | Verified (facility listings) ***(flag: exact count varies)*** | Digital Realty; DataCenters.com; Baxtel; cloudscene |
| STT GDC STT Singapore 1–7; Temasek-backed; >US$13B deal talk | Verified (operator identity); 7 sites ***(flag)***; deal talk ***(flag)*** | STT GDC; Straits Times (2026, citing WSJ) |
| Keppel DC Singapore 1–9; SGP9 AI-ready at Genting Lane | Verified (listings) | DataCenterMap; PeeringDB; Woh Hup |
| GDS pilot winner; warehouse acquisition Oct 2024 | Verified | DCD (Jul 2023; Oct 2024) |
| AirTrunk SGP1 Loyang; 78+ MW campus; StarHub anchor; pilot win w/ ByteDance | Verified (78+ MW, Loyang, pilot win); anchor ***(flag)*** | AirTrunk; DataCenterMap; Baxtel; DCD |
| PDG SGP1 in Singapore | Verified (presence); details ***(flag)*** | datacenters.world; PDG |
| AWS `ap-southeast-1` launched 2010; first AWS SEA region; 3 AZs | Verified | AWS docs; region trackers (HostDir "since 2010") |
| Azure Southeast Asia launched 2010 | Verified | Azure region documentation (2010 first international wave) |
| GCP `asia-southeast1` launched 2017 | Verified | GCP region documentation |
| ~94–95% gas-fired grid; ~7–10% DC share; ~8 GW peak | ***(flag: ranges/approximate)*** | EMA-derived public figures; industry commentary |
| AI-ready density 30–100+ kW/rack; liquid cooling | ***(flag: industry-standard ranges, not SG-specific stats)*** | NVIDIA/industry whitepapers |

### 10.2 Primary and key sources

- **IMDA** — moratorium and DC-CFA press releases (imda.gov.sg): "EDB and IMDA Launch Pilot Data Centre – Call for Application" (20 Jul 2022); Green Data Centre Roadmap materials (30 May 2024).
- **EDB** — "Singapore pilots sustainable way to grow data centre capacity" (Sep 2022; Jan 2022 review completion, PUE 1.3); "Singapore: Where AI vision becomes reality" (2026; 70+ DCs, 1.4 GW).
- **Data Centre Dynamics (DCD)** — moratorium coverage; "Singapore selects Equinix, Microsoft, AirTrunk, and GDS for 80MW data center trial" (14 Jul 2023); "GDS acquires Singapore warehouse" (14 Oct 2024); pilot analysis (Aug 2022).
- **Operator sites** — Equinix (SG IBX portfolio); Digital Realty (Singapore data centres); ST Telemedia Global Data Centres (STT Singapore 1–7); Keppel Data Centres (Keppel DC Campus, SGP series); AirTrunk (SGP1 Singapore); Princeton Digital Group (SGP1); GDS announcements.
- **Hyperscaler region docs** — AWS global infrastructure (regions/AZs); Microsoft Azure regions (Southeast Asia); Google Cloud regions (asia-southeast1).
- **Press** — The Business Times (moratorium-lifting coverage, Dec 2021/Jan 2022); The Straits Times (STT GDC deal reports, 2026); CNA and regional tech press for DC-CFA2 (Dec 2025); OpenGov Asia (SS 715:2025, DC-CFA2, 2 Dec 2025); Mallesons (DC-CFA2, 9 Dec 2025).
- **Trackers** — DataCenterMap (66 facilities/47 operators); datacenters.world (57 facilities/35 operators); Baxtel (110 DCs/49 providers, 2026); JLL Data Centre Report Q2-2025 (Singapore); Credence Research (Jan 2025 market sizing).

### 10.3 Further reading in this repository

- [htx_ngine_guide.md](htx_ngine_guide.md) — Singapore's sovereign AI infrastructure (the public-sector DC story).
- [cloud_providers_guide.md](cloud_providers_guide.md) — cloud regions, AZs, and DR patterns (the architecture behind §4).
- [physical_ai_guide.md](physical_ai_guide.md) and [gpu_optimization_guide.md](gpu_optimization_guide.md) — the AI hardware that defines AI-ready DCs.
- [nutanix_products_guide.md](nutanix_products_guide.md) / [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) — what runs inside the colo cage.
- [finops_guide.md](finops_guide.md) — cost governance for the cloud half of the §7 strategy.
- [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) — running LLMs on-prem (the AI-DC workload).
- [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) — enterprise AI plumbing for the bank scenario.
- `banking/` series — how banks run their own estates (DBS, OCBC, Standard Chartered, and the core-banking/DC-relevant guides).
