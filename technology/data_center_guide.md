# The Data Centre: The Facilities and the Physical-Infrastructure Umbrella — A Comprehensive Guide

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Research — Data Centre series; the umbrella guide to the data centre as a *physical facility*: power, cooling, tiers, layout, physical security, operations, metrics and sustainability — the building and its infrastructure, not the network or the compute inside it
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** Uptime Institute (Tier Classification System, Tier Certification, Global Data Center Survey 2024); The Green Grid / ISO-IEC 30134 standards; ASHRAE thermal guidelines; vendor engineering documentation (Schneider Electric, Eaton, Vertiv, Rittal, Open Compute Project); operator disclosures (Equinix, Digital Realty, CyrusOne); Data Centre Dynamics (DCD) and Data Center Knowledge (DCK) industry reporting
> **Last Updated:** September 2026

---

## Table of Contents

1. [The Data-Centre Umbrella — What This Guide Covers](#1-the-data-centre-umbrella--what-this-guide-covers)
2. [Facility Anatomy — the Building, the Rows, the Rooms](#2-facility-anatomy--the-building-the-rows-the-rooms)
3. [The Taxonomy of Data Centres — Enterprise, Colocation, Hyperscale, Edge, Modular](#3-the-taxonomy-of-data-centres--enterprise-colocation-hyperscale-edge-modular)
4. [The Uptime Institute Tier Classification — and the 'Two Nines' Myth](#4-the-uptime-institute-tier-classification--and-the-two-nines-myth)
5. [Power — the Electrical Spine of the Data Centre](#5-power--the-electrical-spine-of-the-data-centre)
6. [Cooling — the Thermal Problem](#6-cooling--the-thermal-problem)
7. [Layout and Racks — from 19-Inch to the AI-Density Row](#7-layout-and-racks--from-19-inch-to-the-ai-density-row)
8. [Physical Security — the Layered Model](#8-physical-security--the-layered-model)
9. [Certification and Operations — Tier Certification, M&O, Monitoring, Maintenance](#9-certification-and-operations--tier-certification-mo-monitoring-maintenance)
10. [Metrics — PUE, DCiE, CUE, WUE and the Nines Table](#10-metrics--pue-dcie-cue-wue-and-the-nines-table)
11. [Sustainability — Energy, Water and Carbon (Condensed)](#11-sustainability--energy-water-and-carbon-condensed)
12. [Colocation Economics — Models, Providers and the Interconnection Story](#12-colocation-economics--models-providers-and-the-interconnection-story)
13. [Worked Example — Cymbal Bank's Data-Centre Strategy Review](#13-worked-example--cymbal-banks-data-centre-strategy-review)
14. [Glossary, Claims Status and Further Reading](#14-glossary-claims-status-and-further-reading)

### How to Read This Guide

This is the **facilities and physical-infrastructure umbrella** in the `technology/` data-centre series: it explains what a data centre *is as a building and a set of engineered systems* — power, cooling, tiered resilience, layout, physical security, operations, metrics and sustainability — and the economics of renting that building from a colocation provider. It deliberately stops at the white space where the IT begins:

- **The network layer is a sibling guide.** [l2_design_financial_data_centers_guide.md](l2_design_financial_data_centers_guide.md) is the companion deep-dive on L2/VXLAN/EVPN/multicast design inside and between data centres (the leaf-spine fabric, the meet-me-room handoffs, the dark fibre between sites). This guide only covers the physical and structural enablers the network plugs into.
- **The compute layer is condensed here.** [devstack_openstack_guide.md](devstack_openstack_guide.md) and the Kubernetes/OpenShift guides cover servers, hypervisors, containers and orchestration; the middleware angle lives in [kafka_virtualization_guide.md](kafka_virtualization_guide.md) and [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md). This guide treats racks, power feeds and cooling as the *supporting cast* for that compute.
- **DR/BCP depth lives in the banking series.** [operational_resilience_framework_guide.md](../banking/operational_resilience_guide.md) carries the recovery-time/recovery-point objectives, failover runbooks and regulatory resilience expectations; this guide only introduces the *pairing of facilities* concept in the worked example (§13).
- **The Singapore market deep-dive is a sibling.** [singapore_data_centres_guide.md](singapore_data_centres_guide.md) covers the SG market, regulation and the AI-DC wave in depth; where this guide needs a market example it cross-refs rather than re-derives.

**A note on numbers.** Data-centre statistics are methodology-dependent: different trackers count different things (buildings vs critical MW, retail vs wholesale space, certified vs operational capacity). Where sources disagree, this guide says so and flags the figure. Claims marked ***(flag)*** could not be fully verified against primary sources at writing time; the ✅/⚠/❌ audit in §14 is the index of exactly that.

---

## 1. The Data-Centre Umbrella — What This Guide Covers

Ask five engineers what a "data centre" is and you will get five answers: one points at a VMware cluster, one at a leaf-spine fabric, one at a PUE spreadsheet, one at a concrete bunker with generators, and one at a cloud region. All five are partially right, and the reason is that **the data centre is an umbrella term covering several very different layers of engineering**, of which the *facility and its physical infrastructure* is the layer this guide owns.

The mental model used throughout this guide is a simple one. A data centre is:

1. **A building** — a shell engineered for a purpose no ordinary office building serves: it must hold many tonnes of electronics, dissipate their waste heat continuously, survive grid power failure without blinking, and keep unauthorised humans out.
2. **A power system** — electricity enters from the utility, is transformed, conditioned, backed up by batteries and generators, and distributed to thousands of sockets in a way that tolerates the failure of almost any single component.
3. **A cooling system** — every watt of IT power becomes a watt of heat; the cooling plant exists to move that heat from chip to room to outside air, continuously, in every season.
4. **A security and operations envelope** — layered physical barriers, monitoring, and disciplined operational processes (maintenance, change, incident response) that keep the engineered resilience real rather than theoretical.
5. **A business arrangement** — someone owns the building (an enterprise, a colocation provider, a hyperscaler) and someone pays for the power, space and connectivity inside it.

The IT that the business actually cares about — servers, storage, network, applications — sits on top of this physical umbrella, and the umbrella's *only* job is to keep that IT powered, cool, connected and safe. A helpful way to frame the boundary: **everything between the utility substation and the IT equipment's power inlet, and everything between the outside air and the server intake, is the physical-infrastructure umbrella.** Above the inlet and the intake, the sibling guides take over.

### 1.1 Why the Facilities Layer Deserves Its Own Guide

Three forces have pushed facilities engineering from a back-office concern to a board-level one:

- **The AI/GPU power wave.** AI training clusters have rack power demands an order of magnitude above the traditional enterprise rack, which has forced new cooling architectures (liquid cooling), new power delivery designs (higher-voltage distribution, more backup), and new questions about grid capacity — see §6 and §11.
- **Regulatory attention.** Data centres are now treated as strategic infrastructure and significant energy consumers in multiple jurisdictions; Singapore's moratorium-then-green-criteria story ([singapore_data_centres_guide.md](singapore_data_centres_guide.md)) is the clearest example.
- **The economics of failure.** Outages are measured in millions per hour for a financial firm, so the design choices in §4 (Tiers), §5 (power) and §8 (security) are risk decisions with a price tag, not purely engineering ones.

### 1.2 What This Guide Verifies — and What It Assumes

Everything structural about the *facility layer* is researched against primary sources: the Uptime Institute's own Tier definitions (§4), the Green Grid/ISO metrics (§10), and industry-engineering documentation for power and cooling practice (§5, §6). The **network**, **compute** and **DR/BCP** layers are assumed known and cross-referenced, not re-derived. The worked example in §13 uses Cymbal Bank, a fictional global bank, to keep the strategy discussion concrete without misrepresenting any real institution's estate.

### 1.3 The Facility Life Cycle

Every data centre passes through the same phases, and knowing which phase a facility is in explains most of its behaviour — and most of the difference between what a provider *markets* and what a tenant *experiences*:

1. **Design.** The topology is specified (Tier objective, power and cooling architecture, density assumptions) and validated — for certified facilities, through the Uptime design-document review (§9.1). This is where the density assumptions of §7.2 get frozen into concrete; a hall designed for 4 kW/rack in 2010 cannot be re-designed by retrofit.
2. **Build and commission.** Construction, then **commissioning** — the exhaustive testing of every subsystem (generators under load, UPS transfers, cooling sequences, fire alarms) before IT moves in. Commissioning is where design errors are caught; skipping or compressing it is a classic false economy.
3. **Operate.** The decades-long phase covered by §9: monitoring, maintenance, change, capacity management, and the certification/operational assessments that keep the facility honest.
4. **Retrofit / upgrade.** As §6.4 and §7.2 showed, today's retrofit wave is density-driven: containment retrofits, higher-voltage distribution, liquid-cooling pipe runs, upgraded switchgear. Retrofit is almost always harder and more expensive than building it right initially — the reason 29% of operators are upgrading halls (2024 survey) and the reason old shells sometimes get retired instead.
5. **Retire / repurpose.** Facilities eventually end: leases expire, shells age out of density capability, and the workloads move to the cloud or a newer hall. The exit (decommissioning, data destruction, environmental remediation) is itself a risk event — a retired data centre that still contains storage media is a data-leak waiting to happen, which is why secure disposal belongs in the facilities programme.

---

## 2. Facility Anatomy — the Building, the Rows, the Rooms

Before the taxonomy of *kinds* of data centre (§3) and the deep dives on each engineering system (§5–§8), it helps to walk through the physical anatomy that nearly every data centre shares, in some form. The names differ by era and vendor, but the organs are the same.

### 2.1 The Building Shell

A data-centre building is a large, usually windowless, strongly constructed structure — often tilt-up concrete, structural steel or precast panel — with very high floor-loading capacity (raised-floor systems are typically rated for 600–1,200 kg/m² of live load, versus roughly 250–500 kg/m² for an ordinary office floor ***(flag: exact ratings vary by code and vendor)***) and a flat roof or yard space carrying the bulk of the cooling plant: chillers, dry coolers, cooling towers and generator exhaust paths. The shell is usually built to a higher fire-rating and seismic/storm standard than surrounding commercial buildings because the cost of a compromised shell is the loss of the whole IT contents.

Key architectural facts to know:

- **Slab-on-grade vs raised floor.** Traditional data centres use a **raised floor** — a grid of pedestals holding removable floor panels (commonly 600 mm × 600 mm) with a plenum underneath (typically 450–900 mm deep) that serves as a cold-air supply duct and a cable route. Modern high-density and hyperscale facilities increasingly use **slab-on-grade ("slab") construction with overhead cooling and cable trays**, because the underfloor plenum struggles to move enough air for dense rows and obstructs liquid-cooling pipework. Both coexist in the industry today; "raised floor vs slab" is a genuine design fork, not a right answer.
- **The white space (data hall).** The secure interior where IT racks live: the "data hall" or "computer room". Its floor plan is organised into **rows** of racks with aisles between them (see §7 for the hot/cold-aisle logic).
- **Ancillary rooms.** Around the data hall sit the **electrical rooms** (switchgear, UPS, batteries), the **generator yard** or plant room, the **mechanical rooms** (chillers, pumps, CRAC/CRAH units), the **security control room** / NOC, and loading bays. In a colocation building, additional customer-facing spaces exist (see 2.3).

### 2.2 The Three Rooms That Matter for Connectivity: MMR, Meet-Me Room, Cross-Connect

Connectivity inside a facility is organised around a small number of rooms that recur in every provider's floor plan, and which the network-layer sibling guide terminates into:

- **The meet-me room (MMR).** The demarcation point of a multi-tenant facility: a physically secure room where carriers, ISPs and cloud on-ramps bring their fibre and hand it off to the facility's internal cabling. Tenants cross-connect from their cage/suite to the MMR to buy circuits from whichever providers are present. The density of carriers in the MMR is a core commercial selling point of a colocation facility (§12).
- **Main distribution frame (MDF) / main cross-connect (MC).** The building's internal cabling hub, typically adjacent to or synonymous with the MMR, from which structured cabling fans out to each data hall and row (horizontal distribution areas).
- **Entrance room / POP room.** Where external carriers' cables enter the building (often from multiple diverse routes for resilience) and terminate. The *diversity of physical cable entries* is a resilience feature that operators advertise: two separate manhole routes into the building, so a single dig cannot sever both.

### 2.3 Customer Spaces in a Multi-Tenant Building

A colocation facility (and increasingly an enterprise facility subletting space) carves the white space into customer-delimited units. From Wikipedia's colocation overview and standard industry practice, the progression of privacy/space granularity is:

- **Cabinet / rack.** A single locking 19-inch enclosure (or half/quarter cabinet) rented to one customer, sharing the hall's power and cooling with everyone else.
- **Cage.** A fenced, lockable area of raised floor (mesh walls to the ceiling, or solid partitions) holding several racks, still sharing the hall's cooling and shared power distribution but with its own access control and its own cross-connect panels.
- **Suite.** A dedicated, fully enclosed room within the facility, with its own doors, often its own power distribution and sometimes dedicated cooling — closer to "a data centre inside the data centre".
- **Modular / POD.** A self-contained unit (steel-walled room, or containerised module) with its own power and cooling plant, deployed inside or alongside a building to add capacity in standardised increments (§3.5).

### 2.4 The Raised-Floor Details That Bite

Two under-floor details matter to anyone planning space:

- **Plenum depth and airflow.** Shallow plenums (<450 mm) restrict underfloor cold-air delivery and make it hard to place perforated tiles where they are needed; deep plenums cost building height. This is a live constraint when retrofitting dense racks into an old raised-floor hall — one reason high-density AI retrofits often go to slab halls or new buildings instead.
- **Cable vs air conflict.** Underfloor cable trays compete with cold air for plenum volume. Well-run facilities route power and fibre in overhead trays or in segregated underfloor lanes precisely so the cold-air path stays clean. When a hall is "cable-dense", its cooling effectiveness quietly degrades — a classic facilities-vs-IT tension resolved by disciplined cable management.

**Verified framing.** Facility anatomy is well-covered by primary and industry documentation (Uptime Institute Tier materials reference the physical components in §4; colocation configuration terms — cabinet/cage/suite/MMR — are standard across operator literature and are captured in the Wikipedia colocation survey above). The specific live-load ratings are code- and design-dependent, hence flagged rather than asserted.

### 2.5 Fire Detection and Suppression

Fire is the data centre's nightmare scenario — not because fires are common, but because the *response* to a fire (water, or the evacuation of power) can destroy what the fire spared. Standard practice, consistent across insurance and engineering literature:

- **Detection is aspirational about speed.** Modern facilities use **aspirating (very early warning) smoke detection** — the VESDA-style systems that continuously sample air through a pipe network and detect the first particles of an incipient fire, often while it is still a smouldering component rather than a flame. The goal is to detect *before* suppression is needed, so the incident can be handled with investigation and a fire extinguisher.
- **Suppression is designed not to destroy the load.** Where water sprinklers are the code fallback (and in many jurisdictions they are mandatory), facilities add **pre-action** systems (water is held back until both a smoke signal and a sprinkler head release) and — for the space itself — **clean-agent gas systems** (agents such as FM-200/Novec 1230 or inert-gas blends) that extinguish by interrupting combustion or displacing oxygen without leaving water damage across the IT. The trade is real: clean agents are effective and non-destructive but expensive to replenish, so they protect the data hall while pre-action water protects the structure around it.
- **The operations angle.** Fire-safety is a *regime*, not a fit-out item: compartmentation (fire walls and doors dividing halls), suppression-system testing and certification, evacuation plans, and the notorious risk that **fire-drill or false-alarm suppression discharges** cause their own outages. Uptime's operational literature lists fire suppression among the systems whose maintenance and testing carry unexpected risk — another reason the operational disciplines of §9 matter as much as the hardware.

---

## 3. The Taxonomy of Data Centres — Enterprise, Colocation, Hyperscale, Edge, Modular

The industry sorts data centres by *who owns them, who they serve, and how they are built*. The boundaries blur (a hyperscaler leases wholesale space from a colocation provider; an enterprise builds a mini-hyperscale hall), but the following working definitions are the ones used in contracts and market statistics.

### 3.1 Enterprise / Corporate (On-Premises)

An **enterprise data centre** is owned and operated by the organisation whose workloads it runs — a bank's own facility, a manufacturer's server room, a government agency's data hall. The organisation pays the full capital cost of building and the full operating cost of running it, and in exchange keeps complete control of design, security, operations and data residency. The Uptime Institute's 2024 survey still finds that more than half of workloads (55%) now run off-premises, yet "many continue to maintain their own data centres" — enterprise-owned facilities remain a large share of the world's footprint, and almost half (47%) of survey respondents work primarily in a facility more than 11 years old, which matters enormously for what can and cannot be retrofitted (§7).

### 3.2 Colocation — Retail vs Wholesale

A **colocation centre** (colo) rents space, power, cooling and physical security for customer-owned IT equipment, and connects customers to carriers and clouds (Wikipedia's colocation survey; the "carrier hotel" heritage). The customer owns the servers and typically manages them remotely or via "remote hands"; the provider owns the building and the infrastructure. Two commercial sub-models dominate:

- **Retail colocation.** Customers rent by the cabinet/cage (or fraction of one) with power bundled per circuit or per kW, plus optional managed services, cross-connects and remote hands. Contracts are shorter (months to a few years), pricing is higher per kW, and the provider carries more of the operational burden. This is the classic Equinix/Telehouse-style model and the one a bank most often uses for a handful of cabinets or a small cage.
- **Wholesale colocation.** Customers lease large dedicated footprints — private suites, entire data halls, sometimes whole buildings — often on multi-year (5–15 year) contracts, taking more control over power and cooling configuration and paying lower rates per kW. Data Centre Knowledge's retail-vs-wholesale explainer and provider glossaries (Stream, Flexential) agree on the shape: wholesale = dedicated environment at campus scale with significant customer control; retail = shared environment sold in small increments.

The dividing line is fuzzy in practice (some providers treat anything below a megawatt as retail), which is why colocation market statistics are methodology-sensitive — see the note on numbers in the preamble.

### 3.3 Hyperscale

**Hyperscale data centres** are the enormous, standardised facilities built by cloud and internet giants (AWS, Microsoft Azure, Google Cloud, Meta and their peers) to house tens of thousands of servers. They are distinguished less by any single size threshold than by *extreme standardisation, automation and scale economics*: uniform hardware (often Open Compute Project designs, §7), software-defined operations, and capacity added in repeating modular units. Lawrence Berkeley National Laboratory's 2024 US Data Center Energy Usage Report estimated that hyperscale and colocation centres held 74% of computer servers in 2023 — a striking measure of how concentrated the world's compute has become. Uptime Institute's 2024 survey notes that "hyperscale and IT services data center operators are the dominant buyers of GPUs" — the AI build-out is happening at this end of the market.

### 3.4 Edge

**Edge data centres** are smaller, more distributed facilities placed physically close to users, devices or data sources to cut latency and backhaul — the canonical definition used across the industry (Wikipedia's data-centre and edge-computing surveys; Gartner's edge-computing market literature). An edge node may be as small as a few racks in a telecom exchange, a retail store's micro-room, or a modular unit at a cell site; it typically has simpler redundancy than a core facility and is operated remotely with automated monitoring, because there is no staff on site. Edge is a *deployment pattern* as much as a building type — the same servers could run in a core facility, but latency budgets (5G, industrial control, trading proximity) push them outward.

### 3.5 Modular / Containerised

**Modular (or containerised) data centres** package IT capacity into standardised, factory-built units that ship to site and plug in — either full ISO shipping containers with their own cooling (the Cisco/IBM/Sun portable-modular form factor) or prefabricated building modules assembled into halls. The value proposition, per the industry literature: standardised components, faster deployment, incremental capacity that tracks demand, and factory quality control instead of site-built variation. Modularity has become a mainstream delivery mechanism rather than a niche: Uptime Institute runs a dedicated "Tier-Ready" modular certification track, hyperscalers build repeating module halls, and colocation providers deploy prefabricated "data centre as a service" units. In the AI era, containerised and prefabricated plants are a common way to add liquid-cooled capacity fast (see §6).

**Taxonomy caution.** These five labels answer different questions (ownership, tenancy granularity, scale, location, construction method), so a single physical site can carry several labels at once — a wholesale colocation campus on the edge of a metro, built from modular halls, hosting hyperscale tenants, is a perfectly normal 2026 object. Uptime's 2024 survey finds colocation growth "accelerates faster than other market segments" and that "nearly two-thirds of colocation providers host hyperscale tenants" — the categories are converging.

---

## 4. The Uptime Institute Tier Classification — and the 'Two Nines' Myth

No topic in facilities engineering is more quoted and more misquoted than the Uptime Institute **Tier Classification System**. Because it is the language of colocation contracts, lease negotiations and risk committees, it is worth getting precise — including where the marketing has outrun the standard.

### 4.1 What the Standard Actually Says (Primary Source)

Uptime Institute created the Tier levels in the mid-1990s (the organisation itself says "over 30 years ago") as a performance-based way to compare site infrastructure, and it remains the only organisation licensed to certify against it. Its own summary of the four Tiers, from [uptimeinstitute.com/tiers](https://uptimeinstitute.com/tiers), is:

| Tier | Name | Core topology requirements |
|------|------|----------------------------|
| **Tier I** | Basic Capacity | A UPS for power sags/outages/spikes, an area for IT, dedicated cooling that runs outside office hours, and an engine generator for outages. Protects against disruption from human error but not against unexpected failure; the facility must shut down completely for preventive maintenance. |
| **Tier II** | Redundant Capacity Components | Adds redundant capacity components (engine generators, energy storage, chillers, cooling units, UPS modules, pumps, heat rejection, fuel). Distribution is still a single path: components can be removed without shutting the critical environment down, but an unexpected shutdown still affects the system. |
| **Tier III** | Concurrently Maintainable | Redundant capacity components **plus redundant distribution paths**. Any capacity component or distribution path can be taken out of service on a planned basis without impacting IT — no shutdowns for maintenance or replacement. |
| **Tier IV** | Fault Tolerant | Several independent, **physically isolated** systems acting as redundant capacity components and distribution paths, so an event in one cannot compromise the other. Adds fault tolerance: a single unplanned failure or distribution-path interruption does not affect IT operations. All IT equipment must have a fault-tolerant power design, and cooling must be continuous. |

Three properties of the standard, per Uptime's own materials, are essential to reading it correctly:

1. **It is performance-based, not a component checklist.** Uptime's *Tier Standard: Topology* defines outcomes (redundant capacity components; concurrent maintainability; fault tolerance) and deliberately does not prescribe specific technologies. Its own "myths and misconceptions" article is blunt: *"increasing the component count does not determine or guarantee achievement of any specific Tier level… it is possible to achieve Tier IV with just N+1 components, depending on how they are configured and connected to redundant distribution pathways."*
2. **Each Tier incorporates the requirements of the lower Tiers** — the levels are progressive, but Uptime stresses that *"this progression does not mean that a Tier IV data center is better than a Tier II"*; higher Tiers fit different business risk profiles and cost more.
3. **Topology is only half the story.** Uptime's Tier classification pairs design topology with *operational sustainability* — the behaviours, staffing and processes that determine whether the engineered resilience survives contact with reality (§9).

### 4.2 The 'Two Nines / Three Nines' Myth — What Uptime Does NOT Certify

Here is the part that every procurement deck gets wrong. The industry constantly quotes availability percentages for the Tiers — Tier I ≈ 99.671%, Tier II ≈ 99.741%, Tier III ≈ 99.982%, Tier IV ≈ 99.995% (≈28.8 hours, ≈22 hours, ≈1.6 hours and ≈26 minutes of expected downtime per year) — and many vendors and brokers present them as "Uptime SLA" guarantees. The Uptime Institute's own statement could not be clearer (its eJournal article "Myths and Misconceptions Regarding the Uptime Institute's Tier Certification System"):

> **"Tier levels specify an estimated downtime per year. False.** Uptime Institute removed references to 'expected downtime per year' from the Tier Standard in 2009, but they were never a part of the Tier definitions."

The Tier Standard is built on *performance outcomes* — redundant components, concurrent maintainability, fault tolerance — not on a promised percentage. Availability is an operational outcome (arithmetic: time available over total time, driven by the number, frequency and duration of real disruptions), and *"even a Tier IV data center, which is Fault Tolerant, may experience IT outages if it is not operated and managed effectively."* So:

- ⚠ **"'Tier IV = 99.995% availability' is a category error.**" Those percentages circulate widely (they trace to legacy industry material and to the parallel TIA-942 standard's annexes, which *do* tabulate availability ratings); Uptime does not certify percentages. Treat any "guaranteed nines" tier claim as marketing shorthand, and read a colocation SLA's *actual contractual uptime credit terms* instead (§12).
- ⚠ **"Tier III plus", "Tier IV ready", "four-nines certified"** have no basis in the Uptime standard; Uptime's own staff call such invented shorthand "objectionable" and warn that it confuses procurement.
- **Certification is what makes a Tier claim real.** Uptime publishes a searchable list of certified facilities; a provider whose facility is not on that list is making a self-assessment (see §9 for the three certification phases). History shows why this matters: in 2014–2015 Uptime imposed a two-year expiry on design certifications and dropped standalone design certification for commercial providers in North America after cases where facilities marketed a Tier IV label their constructed infrastructure did not fully earn.

### 4.3 Where Each Tier Is Actually Used

Uptime's guidance frames Tiers I–II as *tactical* solutions driven by first cost and time-to-market, and Tiers III–IV as *strategic* solutions for organisations whose revenue depends on real-time delivery. The practical mapping used across the industry (consistent with Uptime's framing and market practice):

- **Tier I–II**: development/test environments, internal tools, facilities where planned maintenance shutdowns are tolerable.
- **Tier III**: the mainstream for enterprise production and commercial colocation — concurrent maintainability means planned maintenance never takes the business down, which is usually the requirement that actually matters.
- **Tier IV**: genuinely mission-critical, zero-tolerance workloads — and even then, only where application-level redundancy (a second facility, §13) cannot cover the gap, because Tier IV costs substantially more to build and run.

The Cymbal worked example in §13 applies this mapping to workload classes rather than to "the data centre" as a monolith — the correct way to use Tiers.

### 4.4 TIA-942 and Other Rating Systems — Don't Conflate

Uptime's Tiers are so dominant that buyers forget there are *other* four-level rating systems, and the most common conflation is with **TIA-942** (the Telecommunications Industry Association's *Telecommunications Infrastructure Standard for Data Centers*). Both use four levels — TIA calls them **Ratings 1–4** — and TIA-942's structure is descended from the same thinking as Uptime's Tiers, but they are different instruments run by different bodies:

- **Uptime Institute Tier** (topology + operational sustainability) is a *performance-based* evaluation of the infrastructure's configuration and its operation. Uptime publishes its Tier Standard documents (the *Tier Standard: Topology* defines the performance outcomes), and Uptime Institute itself is the only organisation permitted to certify facilities against them — certification is administered by the standard-setter, and third parties cannot issue "Uptime Tier" certificates.
- **TIA-942** is a published *design standard* (a document, regularly revised) that specifies required and recommended telecommunications, cabling, architectural and electrical attributes for each Rating, including structured-cabling details the Uptime Tiers deliberately do not cover (Uptime's topology standard explicitly excludes building codes, security and property factors). Availability percentages associated with the levels appear in TIA-942-derived material, which is one of the reasons those percentages get wrongly attached to Uptime Tiers (§4.2).

⚠ **The practical rule:** when a provider says "Tier IV", ask *which framework* — Uptime certification (check the registry) or TIA-942 conformance (check the documentation) — and note that the two are not interchangeable. A facility can be TIA-942 Rating 4 *designed* while holding no Uptime certification at all, and vice versa. Neither framework's level is a guarantee of measured uptime; both are design/operations capability statements, and both should be read together with the SLA's actual credit terms (§10.3). Other schemes exist (EN 50600 in Europe, country-specific green- and security-rating regimes) and are worth knowing by name, but Uptime and TIA-942 are the two a bank's procurement team will actually encounter.

---

## 5. Power — the Electrical Spine of the Data Centre

Power is the first thing a data centre actually *is*: an electrical distribution system with a building wrapped around it. Every downstream engineering decision — cooling capacity, rack density, redundancy tier — is bounded by the power chain. The canonical chain, from the street to the server, is: **utility feed → transformers/switchgear → UPS → distribution (PDUs/RPPs) → rack power → IT equipment**, with **generators** standing in parallel as the long-duration backstop and **batteries/flywheels** bridging the gap between utility loss and generator stabilisation.

### 5.1 Utility Feed, Transformers and Switchgear

Utility power arrives at medium voltage (11 kV, 22 kV or similar depending on grid and market), is stepped down by transformers to low voltage (400/480 V three-phase is the norm in modern facilities, with higher-voltage distribution increasingly used for AI halls), and is carved up by switchgear into the facility's electrical zones. Resilience starts *before* the building: genuinely resilient sites take **two (or more) diverse utility feeds** — ideally from different substations or at least different routes — so a single grid event or a single dig cannot take both down. The switchgear is where the A/B separation that the Tiers demand (§4) physically begins; from the switchboards onward, power paths must be physically segregated to be truly redundant.

### 5.2 Backup Generators

When the utility fails, **diesel generator sets** (gensets) take over — but not instantly: a diesel genset needs time to start, synchronise and accept load. The classic sequence on utility loss is: UPS carries the load from stored energy → generators start automatically → when stable, the facility transfers to generator power → generators run for as long as the outage lasts, bounded only by **fuel supply**. Engineering practice, verified across vendor and operator literature:

- Generators are deployed in redundant configurations themselves (N+1 gensets is the standard resilience posture: enough sets to power the full load plus one spare) and are regularly *tested under load* — a generator that never runs fails when finally called on, so monthly or quarterly load-bank testing is a fixture of DC operations.
- **Fuel storage is sized to a runtime policy**, commonly 24–72 hours of continuous full-load operation on site, with refuelling contracts as the backstop for longer outages. Exact hours are design/business decisions, not a universal standard — flagged rather than asserted.
- A generator's real-world reliability depends as much on its maintenance regime and its fuel quality management (diesel degrades, tanks need polishing) as on its nameplate rating — an operations point that Uptime's outage research reinforces (human error and operational practice, not hardware failure, dominate outage causes; §9).

### 5.3 UPS — the Bridge Across the Gap

The **uninterruptible power supply (UPS)** exists to deliver seamless, conditioned power across the milliseconds-to-minutes gap between a utility disturbance and generator takeover, and to condition utility power continuously. The topology that dominates critical facilities is the **online double-conversion UPS**: incoming AC is rectified to DC, then inverted back to clean AC, so the load is always powered from the inverter and the battery is always in circuit. This is the **VFI (voltage and frequency independent) class** under the IEC 62040-3 standard — the load is fully isolated from utility disturbances. Alternatives (standby/offline and line-interactive units, which switch rather than continuously convert) exist for non-critical gear; critical halls run double-conversion.

- **Battery runtime is deliberately short.** Traditional design sizes UPS batteries to cover only the generator-start window — commonly in the 5–30 minute range at full load — because carrying the facility on batteries for hours would be enormously expensive. Longer "extended runtime" battery strings exist (and lithium-ion is displacing valve-regulated lead-acid in new builds, with faster recharge and smaller footprint), but the economic design point is: batteries bridge, generators carry.
- **Flywheels are the no-chemical alternative.** Kinetic (flywheel) UPS modules store energy in a spinning rotor and provide ride-through measured in seconds to a few minutes — enough to bridge to a generator or through short sags, in a smaller footprint and without battery thermal risk, at the cost of shorter total ride-through. Both battery and flywheel designs are mainstream; the choice is a site-specific engineering trade.
- **Eco-mode caution.** Many double-conversion UPSs offer an "eco-mode" that bypasses the inverter to save the ~3–5% conversion loss. Operators of critical halls treat eco-mode sceptically — it re-exposes the load to utility disturbances and lengthens transfer times — a classic efficiency-vs-resilience trade that belongs in the ⚠ column of any marketing comparison.

### 5.4 Power Redundancy Topologies — N, N+1, 2N, 2(N+1)

Redundancy shorthand describes *how many components exist relative to how many are needed* (N = the minimum number of components required to serve the full load). The definitions below follow the vendor-engineering consensus (Fuji Electric's explainer is representative, and the same definitions appear across Schneider, Vertiv and Uptime material):

| Topology | What it means | Failure behaviour | Where it is used |
|----------|---------------|-------------------|------------------|
| **N** | Exactly enough components, no spare | Any single failure can take the system down | Tier I designs; non-critical plant |
| **N+1** | Full capacity plus one spare component | Survives any *single* component failure (e.g., one UPS module or one genset); simultaneous failures can still bring it down | Tier II/III capacity side — the workhorse of commercial DCs |
| **2N** | Two completely independent, full-capacity systems, each able to carry 100% of the load | Survives failure of an entire system (not just a component); maintenance can proceed on one system while the other carries the load | Tier IV power paths; mission-critical halls |
| **2(N+1)** | Two independent systems, each itself N+1 | Highest redundancy: survives multiple simultaneous failures and full-system maintenance without impact | The very top of the market — large financial/cloud critical halls |

Reading the table correctly requires two ⚠ cautions:

- ⚠ **"2N" is heavily marketed.** A facility may advertise "2N UPS" while its *cooling* is N+1, or its switchgear/distribution is single-path, or its generators are N — and the Tier outcome depends on the whole chain. When a colocation provider quotes a topology, ask for it per-subsystem (UPS, generator, cooling, distribution path, fuel) and ask what a Tier certification (not a marketing page) says (§4.2, §9).
- **Topology ≠ Tier.** As Uptime stresses, N+1/2N/2(N+1) describe component counts, while the Tier also evaluates distribution paths and isolation. A cleverly configured N+1 system can meet Tier IV outcomes; a blunt "2N everywhere" build can fail Tier IV if the two paths share a common point of failure.

### 5.5 A/B Feeds to the Rack — Dual-Corded IT

The power chain ends at the rack with the concept that makes rack-level redundancy work: **A/B (or A and B) power feeds.** Resilient racks receive two independent power feeds — each from a separate UPS/distribution path (ideally the two physically isolated paths a Tier IV demands) — and the IT equipment inside has **two power supplies (dual-corded)**, each plugged into one feed. If feed A's UPS fails, the server draws its full load from feed B without interruption; either feed can be de-energised for maintenance while the rack stays up.

The discipline this imposes is absolute and is a constant source of audit findings:

- **Every dual-corded device must actually have both cords live** and on *separate* feeds. Racks full of "dual-corded" servers with both plugs in the same PDU, or a PDU fed from a single breaker, are a classic illusion of redundancy.
- **Single-corded devices are the enemy of fault tolerance.** Anything with one power supply (some appliances, older switches, console servers) is a single point of failure on whichever feed it hangs from; the standard mitigations are a **static transfer switch (STS)** that can flip a single-corded load between feeds, or simply not allowing single-corded gear in the critical path. This is precisely why Uptime notes that Tier IV requires *all* IT equipment to have a fault-tolerant power design — the building cannot be more fault-tolerant than the devices plugged into it.

Why does the industry obsess over power redundancy? Because Uptime Institute's 2024 global survey still finds **power to be the leading cause of impactful outages** — the electrical spine is where the money and the risk concentrate.

### 5.6 Inside the Rack Power Chain — PDU, RPP and Busway

Between the UPS and the server's two power supplies sits the last mile of the electrical spine, and its anatomy decides how much of the theoretical redundancy actually reaches the IT:

- **RPP (rack power distribution / remote power panel)** — a floor-level or overhead panel that takes a large feeder from the switchgear/UPS distribution and fans it out to multiple racks or rows.
- **PDU (power distribution unit)** — the unit inside or beside the rack that steps the feed down to the outlet level. Two families exist: the basic "dumb" PDU (a strip of outlets) and the **intelligent/switched PDU** that reports per-outlet current draw, temperature and on/off control to the DCIM layer (§9.2). Intelligent PDUs are effectively mandatory in modern facilities because per-rack power *measurement* is what makes capacity management and PUE accounting honest.
- **Busway (busbar track)** — overhead aluminium/copper busbars running along the row, with plug-in tap-off boxes per rack. Busway has displaced underfloor whips in many modern halls because it makes reconfiguring rack power trivial (unplug and re-tap) and carries higher currents — a quiet enabler of the density churn in §7.2.
- **Voltage evolution.** Legacy enterprise racks ran 208 V single-phase (US) or 230 V; modern high-density halls distribute **three-phase 400/480 V** to the row and let the rack-level equipment convert — fewer amps, thinner cables, lower losses. Some hyperscale designs push higher voltages into the rack itself (the OCP Open Rack's 48 V DC bus is the prominent example, §7.1).

The design discipline that binds §5.5 to §5.6: **the A and B feeds must stay separate all the way to the outlet** — separate RPPs, separate busways (or separate ends of the same busway, with a mid-point isolator), separate PDUs per feed. The moment feed A's cable shares a tray, a conduit or a PDU with feed B, the "redundancy" is a fiction — and that fiction is exactly what Uptime consultants find when they inspect constructed facilities (§9.1).

---

## 6. Cooling — the Thermal Problem

Every watt of electricity that enters a server leaves as heat. Cooling is therefore not an accessory to the power system but its mirror image: a 10 MW data hall must continuously move ~10 MW of heat from chip to outside air, 24/7, in every season, or the IT throttles and fails. The 2024 Uptime survey finds cooling and power density the twin constraints on the AI build-out — this section covers the air-based mainstream, the efficiency levers, and the liquid-based systems that the GPU era is forcing into the mainstream.

### 6.1 CRAC vs CRAH — the Two Air-Cooling Workhorses

Room-level air cooling comes in two flavours (definitions consistent across HVAC and data-centre engineering literature):

- **CRAC — Computer Room Air Conditioner.** A self-contained unit that cools air with a **direct-expansion (DX) refrigerant cycle**, compressor and all, like a giant precision air conditioner. It needs no central chilled-water plant and is common in smaller rooms and older facilities; its efficiency is generally worse at scale because every unit runs its own compressor.
- **CRAH — Computer Room Air Handler.** A unit with **no compressor**: it blows room air over coils carrying **chilled water** produced centrally by the chiller plant. Because cooling is generated in one efficient central plant and distributed as water, CRAH-based designs scale better and are the dominant architecture for larger facilities — and they make *water-side economisation* possible (§6.3).

A useful shorthand from the engineering literature: *a CRAC is a complete cooling machine; a CRAH is the air-side terminal of a chilled-water system.*

### 6.2 Hot Aisle / Cold Aisle and Containment

Racks are arranged in rows with alternating aisles, and air is managed so that **cold supply air and hot exhaust air never mix**. In the classic layout, rack fronts face each other across a **cold aisle** (supply air is delivered there, from perforated floor tiles in raised-floor halls or overhead diffusers in slab halls) and rack rears face each other across a **hot aisle** (where exhaust air is collected and returned to the cooling units). Left unmanaged, exhaust air recirculates into intakes and supply air bypasses the racks, collapsing cooling efficiency. The fix is **containment**:

- **Cold-aisle containment (CAC)**: the cold aisle is enclosed (doors/roof) and pressurised with supply air; racks draw from the sealed cold envelope and exhaust into the open room.
- **Hot-aisle containment (HAC)**: the hot aisle is enclosed and ducted back to the cooling units; the room itself stays cool.

Containment, along with blanking panels (filling empty rack U-space so air cannot short-circuit through the rack) and variable-speed fans, is one of the "relatively easy and cost-effective measures" that Uptime credits for most of the industry's PUE gains — and one of the first things an efficiency audit checks.

### 6.3 Temperatures, Economisation and Free Cooling

- **The ASHRAE envelope.** The authoritative thermal guideline for IT equipment is ASHRAE TC 9.9's *Thermal Guidelines for Data Processing Environments* (5th edition, 2021). It sets a **recommended server-inlet range of 18–27°C** for the most common equipment classes, with wider *allowable* classes (A1–A4) defining the extremes at which equipment must still function (and warranty terms apply), plus separate liquid-cooling classes. The single most impactful efficiency lever of the past two decades has been *raising set points within that envelope*: older facilities ran "keep it cold" at 18–20°C; modern facilities push supply temperatures toward the top of the recommended band, cutting chiller work.
- **Economisation / free cooling.** When the outdoor environment is cool enough, the facility can stop (or reduce) mechanical refrigeration and use the outside air directly: **air-side economisers** bring filtered outside air into the hall; **water-side economisers** use a cooling tower or dry cooler to reject chiller heat without running the compressors. In temperate climates this delivers large chunks of the year "free" of compressor energy — the mechanism behind sub-1.2 PUE claims from cool-climate facilities. The corollary: **climate is destiny for cooling efficiency.** A facility in a cool, dry climate can free-cool much of the year; a tropical facility (Singapore included) needs mechanical cooling essentially year-round, which is why the SG energy story is fundamentally different — cross-referenced to the [singapore_data_centres_guide.md](singapore_data_centres_guide.md) rather than re-derived here.

### 6.4 Liquid Cooling — and Why the AI/GPU Era Drives It

Air cooling hits a wall around 20–30 kW per rack: moving that much heat with air requires enormous airflow, and beyond roughly 30–50 kW per rack, air-based designs become impractical. Liquid carries heat far more effectively than air (water has roughly 3,500× the volumetric heat capacity of air in practical systems), which is why the industry's liquid-cooling family has moved from exotic to essential:

- **Direct-to-chip (cold plates).** Coolant flows through cold plates mounted directly on the processors/GPUs, removing heat at the source; the rest of the server is still air-cooled. This is the mainstream liquid architecture of the current AI generation.
- **Rear-door heat exchangers (RDHx).** A heat exchanger coil mounted on the rack's rear door removes heat from exhaust air using facility water — a retrofit-friendly way to lift air-cooled halls' density ceiling without touching the servers.
- **Immersion.** Servers are submerged in dielectric fluid (single-phase or two-phase) that absorbs heat directly — the most aggressive density play, with its own maintenance and logistics implications.

The trend claim is now vendor-verifiable at the primary source. NVIDIA's flagship **GB200 NVL72** — a rack-scale system of 36 Grace CPUs and 72 Blackwell GPUs — is explicitly a *"rack-scale, liquid-cooled design,"* and NVIDIA positions liquid cooling as the enabler of its density and efficiency targets. Industry commentary (flagged ⚠, not a primary source) puts current AI training racks "commonly exceeding 100 kW" with flagship systems above 120 kW per rack. The installed base tells the same story from the other direction: Uptime's 2024 survey finds average modal rack density still at **4–6 kW**, an *average* typical density of **8 kW** (7.1 kW excluding a handful of >50 kW outliers), most facilities without *any* rack above 30 kW, and highest densities climbing in the 15–29 kW range — while a third of operators are building new capacity specifically for high-density cabinets and 29% are upgrading existing halls to cope. The industry consensus, as Uptime frames it: the density curve is a *climb, not yet a cliff* — but newer facilities are being designed around denser IT "often using direct liquid cooling," and the GPU-server generation (≈1 kW per rack unit and above) is what bends the curve. Cooling architecture choice is therefore now a *strategic* decision: a facility built air-only in 2025 will struggle to host the AI racks of 2027 without retrofit or liquid plumbing (see §7).

---

## 7. Layout and Racks — from 19-Inch to the AI-Density Row

The data hall's floor plan, rack standards and power-per-rack economics decide what can physically be deployed where. This section covers the physical substrate of §6's density numbers.

### 7.1 The Rack Standards — 19-Inch, EIA-310 and the Open Rack

- **The 19-inch rack.** The overwhelming majority of IT equipment mounts in a **19-inch rack** — the mounting width is 482.6 mm (19 in), standardised by **EIA-310** (the EIA-310-E revision is current). Vertical space is counted in **rack units (U)**: 1 U = 44.45 mm (1.75 inches). Enclosures come in standard heights — 42U and 48U are the workhorses, with 45U/52U variants depending on vendor and market — and standard widths/depths (600 mm wide/1,000–1,200 mm deep cabinets are typical for modern enterprise gear). Telecom gear has historically used wider 21-inch/23-inch frames, which is why colocation halls still carry a mix of mounting standards.
- **The Open Rack (OCP).** The **Open Compute Project (OCP)** — launched by Facebook in 2011 to open-source hyperscale hardware designs — defined the **Open Rack** standard as an alternative to EIA-310, and it is now the de-facto physical platform of hyperscale and much AI infrastructure. Per the OCP specifications and the Open Rack documentation: equipment bays are **21 inches (537 mm) wide**, vertical units (**OpenU**) are **48 mm** tall, power is delivered as **48 V DC** via busbars in the rack (rather than individual AC power supplies), and everything is front-serviceable and hot-pluggable. The wider bay gives roughly 20% more frontal area — more airflow for denser, hotter gear — and the current **ORv3** generation explicitly combines the 21-inch/OpenU format with EIA-310 compatibility so both equipment families can share a rack. (OCP's 2026 "Open Rack Wide" announcement extends the family to very large-format AI gear racks.)
- **Why this matters to a bank.** Buying compute is no longer only about the server's spec sheet — the *physical form factor* decides which facilities can host it. A dense GPU node in an Open Rack 21-inch format will not fit a legacy 19-inch cage without a rack swap; conversely, most enterprise software gear remains 19-inch. The rack standard is now a procurement and facilities-planning decision, not a footnote.

### 7.2 Power Density per Rack — the Curve

Rack power density is the single most useful number for sizing a facility, and it has moved more in the last five years than in the previous twenty:

- **The historical norm.** For most of the 2000s–2010s, enterprise racks ran at roughly **2–5 kW** — a figure driven by a few hundred watts per volume server. Facility designs and "per-rack" marketing from that era (and many colocation contracts still written that way) assume densities in this band.
- **The 2024 reality (Uptime Institute survey).** The modal deployed rack is still **4–6 kW**; the average of respondents' typical rack densities is **8 kW** (7.1 kW excluding a handful of above-50 kW outlier sites); 7–9 kW racks are the fastest-growing band; and *most facilities have no rack above 30 kW at all*, though highest-deployed densities in the 15–29 kW range are becoming common, with a few sites reporting **100 kW or more**.
- **The AI band.** GPU training racks sit in a different world: industry reporting (⚠, secondary sources) puts current AI training racks commonly in the **20–100+ kW** range, with flagship rack-scale systems (NVIDIA's liquid-cooled GB200 NVL72 family) at **~120 kW and above** per rack. Uptime expects this to accelerate as "dense GPU-servers (1 kW per rack unit or above)" ship in volume — a single such rack needs as much power and cooling as an entire legacy row.

The engineering consequences ripple through everything in this guide: a 100 kW rack needs liquid cooling (§6.4), high-voltage power distribution, and a floor slab that can take the weight; an existing 4–6 kW hall cannot be "upgraded" to it with a few bigger PDUs. The industry's current retrofit reality, per the 2024 survey, is that 29% of operators are upgrading existing halls to support high-density IT and a third are building new capacity specifically for high-density cabinets.

### 7.3 Layout Principles

- **Rows, aisles, and the hot/cold discipline of §6.2** drive the floor plan: racks face into contained cold aisles; power and fibre are segregated (power overhead or underfloor in dedicated lanes; fibre in separate trays); and every row ends at a **horizontal distribution area (HDA)** where the building cabling fans out.
- **The meet-me room and cross-connect topology of §2.2** anchor the plan: the MMR sits central to the cabling spine so cross-connect distances stay short and orderly.
- **Future-proofing is now a layout question.** Modern halls are designed with deeper rack pitches, higher floor loading, liquid-cooling pipe runs (or the space to add them), and higher power headroom per row, precisely because the density curve of §7.2 is climbing. Facilities that skipped this (older halls, "raised floor everywhere, 600 mm pitch, 2 kW/rack air-only") are the ones hitting the AI retrofit wall.

---

## 8. Physical Security — the Layered Model

Physical security for a data centre is classically described as **defence in depth**: concentric layers of control, each independently capable of slowing or stopping an intruder, so that a failure or bypass at one layer does not compromise the asset. The layers, from the street to the silicon, are:

### 8.1 The Layers

1. **Perimeter.** The site boundary: security fencing, controlled vehicle gates, **vehicle barriers/bollards** (anti-ram protection where the site is exposed), lighting, and often crash-rated gates for high-security sites. The perimeter's job is delay and denial: force any approach to be visible and to pass through a controlled point.
2. **Building.** The shell itself is a barrier (reinforced construction, minimal windows, locked plant doors), and entry is through a controlled lobby with a **mantrap** — an interlocking two-door airlock that allows only one authenticated person through at a time and physically prevents tailgating. Loading docks and plant-room entries are separately controlled, and delivery/visitor flows are segregated from staff flows.
3. **Data hall / suite / cage.** Inside the building, the white space is zoned by access level: the customer areas of a colocation facility (§2.3) each have their own access control — biometric or card readers, cage doors with individual credentials, visitor escort rules. Access to the data hall floor is restricted to people with a business reason, and the audit trail records who entered which zone when.
4. **Rack / cabinet.** The innermost layer: individual cabinet locks (electronic, keyed, or both), so that even within a shared cage, access to a specific rack is separately controlled.

### 8.2 Monitoring and Response

- **CCTV** covers the perimeter, entrances, aisles and cages, with retention policies measured in months; modern sites add analytics (motion, intrusion, loitering). Coverage of *every* rack face is standard in colocation because tenants expect video evidence for incident investigation.
- **Access control systems** (badge + PIN/biometrics, with mantrap anti-passback) generate the audit trail that both the operator and its tenants rely on for compliance (and that regulators ask about in outsourcing reviews).
- **Alarm response and guarding:** intrusion, duress and environmental alarms feed a security operations capability — on-site guards at larger facilities, a security control room that also hosts the NOC (§9), and documented response procedures (who is called, how fast, what they may do). Many colocation contracts specify response-time commitments.

### 8.3 The Meet-Me Room Nuance

The MMR (§2.2) is the one place where the "your space is sealed" model deliberately opens up, and it deserves its own security paragraph. The MMR is **shared by design**: every tenant and every carrier terminates cross-connects there, so many different parties' technicians legitimately enter it. The security model therefore shifts from *exclusion* to *containment and verification*:

- Access to the MMR is tightly logged and often requires provider-escorted entry or smart-hands service — tenants rarely touch the MMR directly; they order cross-connects and the provider's staff patch them.
- The physical demarcation between "the carrier's fibre" and "your cross-connect" is where responsibility transfers, so MMR patch panels and fibre trays are labelled, locked where possible, and audited — a tampered cross-connect is a quiet, high-impact attack vector that no amount of cage security prevents.
- For a financial tenant, the practical rule is: *treat the MMR as a semi-public space, encrypt everything that crosses it, and verify physical patching against the connectivity inventory* — the network-layer sibling guide ([l2_design_financial_data_centers_guide.md](l2_design_financial_data_centers_guide.md)) carries the logical-side controls that pair with this physical one.

### 8.4 Security as an Operational Programme

Physical security is not a set of hardware. The layered model only holds if the *processes* hold: visitor management and escort policy, contractor vetting, key/credential lifecycle (revocation on departure), periodic access reviews, red-team or walkthrough testing, and — critically for multi-tenant facilities — **clear separation between tenants' zones and shared zones** with documented evidence for each tenant's auditors. Uptime Institute's operational-sustainability assessments treat security among the behaviours that determine whether a facility's design resilience survives (§9); and for banks, physical controls at outsourcing providers are part of the third-party risk review that the operational-resilience framework ([operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md)) governs.

---

## 9. Certification and Operations — Tier Certification, M&O, Monitoring, Maintenance

A well-designed facility fails when it is badly operated; Uptime Institute's analyses attribute the majority of outages to human error and operational practice rather than to hardware failure. This section covers the certification machinery that validates both design *and* operation, and the operational disciplines (monitoring, capacity, maintenance, change) that keep the physics honest. DR/BCP depth is deliberately left to the [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md); what follows is the *facilities* side.

### 9.1 The Uptime Institute Certification Programme

Uptime Institute is the only organisation licensed to certify against its Tier Standard, and as of the organisation's 2026 materials it has issued **over 4,300 Tier certifications in 120+ countries**. Certification has three distinct phases, and each answers a different question:

1. **Tier Certification of Design Documents (TCDD)** — "is the *design* capable of the Tier?" Uptime consultants review the 100% design documents for weak links. Since January 2014 these awards **expire after two years**, precisely because a design review says nothing about what eventually gets built.
2. **Tier Certification of Constructed Facility (TCCF)** — "was the *building* built to that design?" Consultants visit the site, compare drawings to installed equipment, and witness tests and demonstrations. Uptime reports that in almost every site visit they find post-design changes that would have broken Tier compliance — the constructed-facility check is the one that makes a Tier claim real.
3. **Tier Certification of Operational Sustainability (TCOS)** — "is the facility *operated* well enough to deserve its Tier?" TCOS evaluates staffing, maintenance, management and planning behaviours against the Tier's operational demands, and awards **Bronze, Silver or Gold** levels for sites that exceed the minimum. For sites that are *not* Tier-certified, the equivalent assessment is the **Management and Operations (M&O) Stamp of Approval** — the same operational lens applied to a facility without the design-certification history.

The M&O stamp and TCOS exist because of a hard-earned industry lesson: **"even the most robustly designed and constructed facilities may experience outages without a well-developed comprehensive management and operation program."** Uptime's operational-sustainability materials put numbers on it: human error is blamed for more than 70% of outages in its analyses, and bad operational practices are roughly **5× more likely to harm a data centre than any other cause**.

### 9.2 Monitoring — BMS, EPMS, DCIM and Sensors

Running a facility blind is running it dangerously. The monitoring stack has four layers:

- **BMS (Building Management System)** — the mechanical side: chillers, CRAC/CRAH units, pumps, temperatures, humidity, leak detection, and the building's environmental alarms.
- **EPMS (Electrical Power Monitoring System)** — the electrical side: switchgear state, transformer and UPS readings, generator status, feeder loads and power quality.
- **DCIM (Data Centre Infrastructure Management)** — the layer that ties the physical plant to the IT it serves: real-time visibility of power draw, cooling, temperature and **capacity** (space, power, cooling headroom) mapped against racks, cages and tenants. The industry definition (Cisco and the DCIM vendor literature are representative): DCIM is the category of tools used to monitor, measure and manage the physical infrastructure and power consumption of the data centre — and, as AI racks push densities toward 100 kW, DCIM has become the tooling for managing the thermal envelope row by row.
- **Environmental and security sensors** — temperature/humidity at the row and rack level (spot cooling failures long before room-level alarms), water leak detection under floors, smoke/aspirating fire detection, and the CCTV/access systems of §8.

The operator's control room (NOC/security) consumes all four layers; modern practice adds automated alerting, trend analytics and, increasingly, AI-assisted operations (the 2024 survey shows most operators recognise AI's potential in operations even as *trust* in it has declined three years running — a ⚠-worthy operational caution).

Two operational realities deserve emphasis:

- **Alarm hygiene.** A monitoring stack that alarms on everything alarms on nothing. Mature operations tune thresholds, deduplicate and correlate (a chiller trip, a rising supply temperature and a DCIM hot-spot alert are usually *one incident*, not three), and route alarms by severity to the right human. Alarm floods during genuine incidents are a known failure mode — operators literally miss the critical alarm in a storm of trivial ones.
- **Remote hands and the unstaffed hall.** In retail colocation, the tenant's "eyes and hands" are the provider's **remote-hands / smart-hands** service: vetted technicians who perform physical tasks (cable a server, check a status LED, reseat a module, escort a visitor) under the tenant's instruction, with photo/video evidence. The quality and accountability of remote-hands service is a real selection criterion for distributed estates — it is the human interface of the facility for a bank whose staff are hundreds of kilometres away, and it is one more reason the provider's operational maturity (M&O/TCOS, §9.1) belongs in the due-diligence checklist (§13.5).

### 9.3 Capacity Management

Capacity management is the discipline of knowing, at every moment, how much *usable* headroom remains across the four scarce resources — **space, power, cooling and connectivity** — and of forecasting when each runs out. Key practices, and the survey facts behind them:

- **Power headroom is not nameplate.** A facility's "10 MW" is the designed critical load; usable capacity is bounded by UPS capacity, generator capacity, fuel, cooling rejection and utility contract. Uptime's 2024 survey notes that one in four data centres typically runs **under 40% utilised against available UPS capacity** — an enormous hidden inefficiency that also drags PUE up (lightly loaded UPS and cooling plant run inefficiently).
- **Density-aware planning.** As §7.2 showed, a hall full of 4 kW racks has capacity that a hall of 40 kW AI racks burns through in days; capacity plans must model *per-row* power and cooling, not hall averages.
- **Forecasting.** Survey respondents consistently rank "forecasting future capacity requirements" among their top concerns; in a market where colocation space is scarce and equipment lead times long, capacity decisions are made 12–36 months ahead.

### 9.4 Maintenance Regimes

- **Preventive maintenance (PM).** Critical plant (generators, UPS, switchgear, chillers, cooling units, batteries, fire suppression) runs on manufacturer-recommended PM schedules: oil and filter changes, battery load tests, chiller PMs, calibration. Two facts define the discipline: *tested equipment fails less* (generators are load-banked, batteries are discharge-tested) and *maintenance itself is a risk event* — every PM is an opportunity for human error, which is why it is executed against documented **Method of Procedures (MOPs)** with supervision.
- **Tier and maintenance are linked.** A Tier III facility is, by definition, one where any component can be maintained on a planned basis *without* impacting IT (§4). That property only has value if the operations team actually exploits it — scheduling maintenance against the redundant paths rather than letting spares degrade. Tier I/II sites, by contrast, must plan whole-facility shutdowns for major maintenance, and the business must schedule them.
- **Outage reality check.** Uptime's 2024 survey finds outage frequency and severity roughly unchanged from 2023 (small improvements at best), with **one in five impactful outages costing more than US$1 million** and power still the leading cause. The operational message: resilience is a programme of small disciplines, not a capital project.

### 9.5 Change Control (Condensed)

Changes to facilities and to the IT inside them share one failure mode — unmanaged change is a top outage cause — so both run through formal change control: classification (standard/normal/emergency), risk assessment, approvals, MOP-based execution, rollback plans and post-change review, with maintenance and capacity activities inside the same change calendar. The full change-management, incident and problem-management machinery is the operational-resilience sibling's territory ([operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md)); here it is enough to note that facilities change control and IT change control must be one process, because the riskiest changes in a data centre are precisely the ones where an IT change (a new rack, a rebalance) collides with a facilities state (a UPS in bypass for maintenance).

---

## 10. Metrics — PUE, DCiE, CUE, WUE and the Nines Table

If the facility is the machine, metrics are its instrument panel. This section defines the efficiency and availability metrics that appear in every sustainability report, lease and SLA — with the caveats that matter when reading them.

### 10.1 PUE — Power Usage Effectiveness

**PUE = Total Facility Energy ÷ IT Equipment Energy.** It was first defined by **The Green Grid** in 2007 and is standardised internationally as **ISO/IEC 30134-2** (published 2016). An ideal facility has PUE = 1.0 (every joule goes to IT); everything above 1.0 is the overhead of power conversion, cooling and the rest of the plant. Its inverse, **DCiE (Data Center Infrastructure Efficiency) = 1 ÷ PUE**, expresses the same thing as the percentage of facility energy that reaches the IT.

A quick worked example fixes the intuition: a facility whose IT draws **1.0 MW** and whose total draw (IT + cooling + power losses + lighting) is **1.5 MW** has PUE = 1.5 and DCiE = 67% — one-third of everything the facility buys is consumed before it reaches a server. Cutting that overhead from 0.5 MW to 0.3 MW moves PUE to 1.3 and, at prevailing industrial electricity prices, saves on the order of six figures annually per megawatt of IT — which is why containment, economisation and set-point management pay for themselves, and why PUE appears in leases and sustainability reports at all. The same example shows the metric's blind spot: if the 1.0 MW of IT is half-idle servers, PUE says nothing about the waste inside the 1.0 MW.

Verified benchmark facts (Uptime Institute Global Data Center Survey, which has collected average annual PUE since 2007):

- The industry-average PUE fell rapidly between 2007 and the mid-2010s — from roughly **2.5 in 2007** to a plateau around 1.5–1.6 — and has been **flat for five consecutive years**.
- The **2024 survey average is 1.56** (n=526): the "global average ~1.5×" figure, year-verified.
- Recent new-build facilities **consistently achieve ~1.3 or better**; the average is held up by the huge installed base of older facilities (47% of respondents work primarily in a facility over 11 years old).

PUE caveats worth their own ⚠ column:

- **PUE says nothing about IT efficiency.** It divides by IT energy but does not ask whether the IT is doing useful work — a facility full of idle servers can have a great PUE. Its limitations "will only grow as some future facilities specialize in denser IT architectures" (Uptime).
- **It can be gamed.** Excluding lighting or other loads, or measuring at high utilisation only, flatters the number; methodologies vary, which is why ISO/IEC 30134-2 exists and why contracts should specify the measurement standard.
- **Climate makes cross-facility comparison unfair.** A warm-climate facility legitimately spends more on cooling than a cool-climate one; PUE is a self-improvement metric first, a comparison metric second.

### 10.2 CUE and WUE — Carbon and Water

The Green Grid extended the PUE family to the other two sustainability dimensions:

- **CUE — Carbon Usage Effectiveness** = total carbon emissions from the facility's energy ÷ IT equipment energy, expressed in kg CO₂-equivalent per kWh of IT energy (Green Grid metric family; ***(flag)*** the ISO/IEC 30134 series part numbering for CUE could not be verified precisely at writing time). CUE makes the *energy source* visible: two facilities with identical PUE can have very different CUE if one buys coal-grid power and the other renewables.
- **WUE — Water Usage Effectiveness** = total water used by the facility ÷ IT equipment energy, in L/kWh (created by The Green Grid in 2011; standardised as **ISO/IEC 30134-9:2022**). Typical values span from **~0 L/kWh** for air-cooled facilities to **~2.5 L/kWh** for evaporatively cooled ones — Equinix, for example, reports a 2025 global WUE of 0.91 L/kWh overall and 1.41 L/kWh for its evaporative-cooling sites. Note the PUE/WUE tension: evaporative cooling lowers PUE but raises WUE; air cooling does the reverse — a water-scarcity decision, not just an efficiency one.

### 10.3 The Nines Table and Availability

"Availability" in the data-centre world is usually quoted as **nines** — the percentage of time a system is available over a year. The arithmetic is unforgiving:

| Availability | Downtime per year (365.25 days) |
|--------------|----------------------------------|
| 99% ("two nines") | 87.7 hours (≈3.65 days) |
| 99.9% ("three nines") | 8.77 hours |
| 99.99% ("four nines") | 52.6 minutes |
| 99.999% ("five nines") | 5.26 minutes |
| 99.9999% ("six nines") | 31.5 seconds |

Two readings of this table matter:

- **Measured vs claimed.** Availability is *measured* — time available ÷ total time — and a "four-nines" claim is only meaningful if someone can point to the measurement window and the incidents inside it. An SLA credit for "99.99% availability" usually compensates contractual damages at some multiple of the monthly fee; it does not compensate the business impact of an outage, which is why Uptime's finding that one in five impactful outages costs over US$1 million is the more honest number.
- **The Tier connection (recall §4.2).** The nines table is where the Tier availability myth lives. Industry shorthand associates Tiers with percentages (Tier III ≈ 99.982%, Tier IV ≈ 99.995%), but Uptime removed expected-downtime references from the Tier Standard in 2009 and does not certify availability percentages — a Tier is a *design and operations* outcome statement, not a nines promise. When someone says "Tier IV = 99.995%", the correct response is: *which certification, and what does the SLA actually credit?*

### 10.4 What Counts as Downtime — SLA Measurement Realities

Availability is only as meaningful as its measurement definition, and SLA measurement conventions shape behaviour on both sides of a colocation contract:

- **What the clock excludes.** Facility SLAs typically measure availability of the *infrastructure* (power and cooling to the space) and exclude from the calculation: scheduled maintenance windows (the very thing Tier III exists to make safe, §4), tenant-caused outages, force majeure, and often *downstream* effects — a facility that loses utility power but transfers cleanly to generators has, in SLA terms, suffered no downtime at all, even if a tenant's single-corded device blinked (the tenant's problem, per §5.5's discipline).
- **Credit mechanics.** The standard remedy is a service credit — a percentage of the monthly recurring fee, often scaled by the duration of the outage (e.g., credits once monthly availability drops below the threshold, escalating with consecutive hours down). Credits rarely approach the tenant's actual loss (recall: one in five impactful outages costs more than US$1M, §9.4), so an SLA is a *governance* mechanism, not an insurance policy — the tenant's real protection is the facility's design and operations, which is why due diligence (§13.5) matters more than the credit schedule.
- **Measurement disputes.** The parties must agree on what instrument measures availability (the provider's EPMS/DCIM data, §9.2), who can access it, and how incidents are classified — otherwise every outage becomes a dispute about whether it "counts". Banks typically negotiate **audit and reporting rights**: monthly availability reports, incident notifications within defined timeframes, and access to the monitoring data behind the numbers.

The unifying lesson of §10: *metrics are only useful when their definitions are fixed in advance.* PUE without a measurement methodology, WUE without a water boundary, availability without an incident-classification rule — each is a number that means whatever the reporter wants it to mean.

---

## 11. Sustainability — Energy, Water and Carbon (Condensed)

The physical-infrastructure umbrella is where the data centre's environmental footprint is made or reduced: the efficiency decisions of §6 and §10 *are* the sustainability programme at the facility level. This section is condensed by design — the regulatory and market depth for the Singapore market lives in [singapore_data_centres_guide.md](singapore_data_centres_guide.md), and this section only establishes the global facts a facilities guide must carry.

### 11.1 The Energy Demand Curve

Verified global context, as compiled in the industry-standard energy analyses:

- The **International Energy Agency (IEA)** estimated data-centre electricity consumption at around **415 TWh in 2024 — roughly 1.5% of global electricity** — after ~12% annual growth over the previous five years, and projects it could **roughly double to ~945 TWh by 2030** (≈15% annual growth). The same analysis identifies AI and data centres as a major driver of US electricity-demand growth to 2030.
- The **2024 US Data Center Energy Usage Report (Lawrence Berkeley National Laboratory)** is the reference US dataset; its finding that hyperscale and colocation centres held 74% of computer servers in 2023 is cited in §3. ***(flag)*** LBNL's headline US share-of-electricity figure is not quoted here because the underlying report text was not re-verified at writing time.

Three facility-level consequences follow. First, **new capacity is being built where power is** — grid interconnection and substation capacity have become the binding constraint on data-centre construction in many markets (a story the SG guide carries for Singapore specifically). Second, **efficiency gains are plateauing**: Uptime's PUE average has been flat at ~1.5× for five years, and the 2024 survey's top management concern is *cost* — energy is now a first-order operating expense, not an environmental afterthought. Third, **reporting is becoming mandatory**: the EU Energy Efficiency Directive is requiring operators to report renewable-energy consumption and water usage from 2024/2025 — data that, per Uptime's 2024 survey, fewer than half of operators currently collect (see 11.3).

### 11.2 Energy, Water and Carbon at the Facility Level

- **Renewable procurement.** The dominant mechanisms are **power purchase agreements (PPAs)** — long-term contracts buying wind/solar output (often off-site, via the grid) — and **energy attribute certificates (EACs)** (RECs, GOs and the like) that let an operator claim the carbon attributes of renewable generation without a physical link. The 2024 survey flags the live controversy: EAC use, quality and acceptability are being debated, and regulators are tightening what counts. Digital Realty's public reporting (1 GW of wind/solar under contract across Texas, Illinois, North Carolina, Oregon, Arizona and Virginia, per its disclosures) and Equinix's renewable programmes are representative of the industry's PPA-led approach.
- **Water.** Cooling water is the second environmental axis: evaporative cooling towers consume water (raising WUE, §10.2) while air-cooled or closed-loop designs consume far less. Equinix's reported 2025 global WUE of 0.91 L/kWh (1.41 L/kWh for evaporative sites) shows the spread in one portfolio. Water-scarcity regulation is increasingly constraining cooling choices in arid regions — a design input, not an afterthought.
- **Carbon accounting.** Under the GHG Protocol, facility emissions split into **Scope 1** (direct: mostly the diesel burned testing and running backup generators), **Scope 2** (embedded in purchased electricity — the big one for data centres) and **Scope 3** (supply chain and embodied emissions in equipment). Uptime's 2024 survey finds reporting still patchy — only about a third of operators report Scope 1, Scope 2 reporting is rising fast, and Scope 3 sits at 18% — which matters because "the majority of operators do not have the data to either make these submissions or back up their corporate net-zero goals."
- **Industry self-regulation.** The **Climate Neutral Data Centre Pact (CNDCP)** — a self-regulatory initiative with the European Data Centre Association and CISPE — commits signatories to climate neutrality by 2030 across energy, water, circularity and efficiency; Digital Realty is a signatory. It is an industry pledge, not law — worth reading as intent, not verification.

### 11.3 Grid Constraints and the AI Squeeze

The 2024–2026 story that ties §11 back to §6 is the **power constraint on AI**: high-density GPU halls multiply demand per square metre, utilities' interconnection queues are years long in hot markets, and some jurisdictions have imposed moratoria or green criteria on new builds (Singapore's moratorium-then-grey/green-criteria sequence is the canonical case — see [singapore_data_centres_guide.md](singapore_data_centres_guide.md)). The facility-level responses are the efficiency levers already covered — containment, economisation, liquid cooling, higher set points — plus siting where renewable generation and grid capacity actually are. For a bank's sustainability reporting, the practical takeaways are: demand PUE *and* WUE *and* carbon-data capability from any colocation provider, and put energy terms in the contract, because the EU-style reporting obligations are coming to more jurisdictions regardless of the operator's enthusiasm.

---

## 12. Colocation Economics — Models, Providers and the Interconnection Story

A bank does not need to *build* a data centre to *use* one. Colocation — renting space, power and connectivity in a carrier-neutral facility — is the default route for most financial workloads outside the hyperscale cloud (§3.2). This section covers the money mechanics, the major providers' verified public facts, and the interconnection economics that separate the "real-estate" business from the "digital-infrastructure" business.

### 12.1 How Colocation Is Priced

Colocation pricing has three recurring components, and the *structure* matters as much as the headline number:

- **Space.** Billed per cabinet (retail) or per square metre / per kW of reserved capacity (wholesale).
- **Power.** Two structures dominate: **inclusive** (power bundled into the monthly fee up to a contracted ceiling) and **pass-through** (the tenant pays the provider's measured consumption at the utility rate plus a small administration margin, with the base rent covering space and infrastructure). Pass-through is increasingly standard for larger deals because it aligns the tenant's conservation incentives with its bill — but it transfers *energy price risk* to the tenant, so the contract must define what happens when utility tariffs spike (a live issue given 2021–2026 energy inflation).
- **Connectivity.** Cross-connects to carriers and clouds are charged per circuit per month, and they are where the facility's ecosystem becomes a product (12.3).

⚠ **On quoted prices.** Market-rate figures for colocation (e.g., per-kW monthly lease rates, construction cost per MW) vary wildly by market, facility age, tier and contract size, and most published numbers are broker marketing. This guide deliberately does not quote per-kW rates as fact; where an indicative figure is needed for a build-vs-buy comparison (§13), it must come from a current market survey for the specific metro, not from this guide.

Beyond the three core components, the contract economics that decide the *total cost of tenancy* include:

- **What is included vs passed through (gross vs net).** "All-inclusive" retail deals bundle power, connectivity and management into one recurring fee; wholesale deals are usually "net" — base rent plus separately metered power, plus pass-throughs for security, maintenance and any shared services. The pass-through list is where surprises hide (a "facility maintenance charge" that grows, a power rate that tracks utility tariffs), so a lease review must enumerate every pass-through and its escalation basis.
- **Escalation and term.** Rent escalators (fixed percentages or index-linked) compound over 5–15 year terms; exit is governed by break options, termination fees and sublease rights. A deal that looks cheaper per kW can cost more over the term if its escalator and exit terms are worse — total-cost-of-tenancy, not unit price, is the number that matters.
- **Power metering and billing accuracy.** With pass-through power, the meter is the contract: tenants should verify that billing matches independently read sub-meter data and that the "power factor" or loss-allocation methodology is defined. Disputes over phantom kW are common enough that audit rights over the provider's metering belong in the lease.

### 12.2 The Major Providers — Verified Public Facts

| Provider | Verified public facts (company disclosures / filings, as compiled at writing time) |
|----------|----------------------------------------------------------------------------------------|
| **Equinix** | HQ Redwood City, California; **Nasdaq: EQIX**; converted to a **REIT in January 2015**; founded 1998 by two former Digital Equipment facilities managers; operates ~260 data centres ("International Business Exchanges", IBXs) across ~33 countries and five continents (company-reported, 2025); revenue ≈ **US$9.2B (FY2025)**; re-branded from "colocation" to "digital infrastructure company" in 2020; positions interconnection — not square metres — as its core product (12.3). |
| **Digital Realty** | HQ Austin, Texas; **NYSE: DLR**; **REIT**; formed in 2004 by GI Partners and listed November 2004; reports 300+ facilities in 25+ countries on six continents (June 2023); grew by acquisition — **Telx (2015, US$1.9B, interconnection)**, DuPont Fabros (2017), **Interxion (announced October 2019, ~US$8.4B — at the time the largest data-centre transaction ever)**, 55% of Teraco (2022) — and markets its platform as "PlatformDIGITAL". |
| **CyrusOne** | HQ Dallas, Texas; founded July 2000; **Nasdaq IPO in January 2013**; **taken private in March 2022 by KKR and Global Infrastructure Partners for US$15B**; operates 40+ carrier-neutral data centres across North America, Europe and Asia (company-reported); a leading *enterprise/wholesale* colocation provider with hyperscale-scale campuses in the US (e.g., Chandler AZ, Allen TX, Northern Virginia) and Europe (London, Frankfurt). |

The pattern to notice: all three are **carrier-neutral, interconnection-aware platforms**, and all three have consolidated hard — Digital Realty's Telx and Interxion acquisitions and Equinix's Infomart Dallas purchase (US$800M, 2018, for a famous interconnection hub) show that the industry's centre of gravity moved from "build sheds" to "own the places where networks meet".

### 12.3 The Interconnection Value Story

Equinix is the canonical case, and its founding logic is the cleanest statement of the interconnection thesis: two former DEC facilities managers started the company in 1998 as a *neutral place where competing networks could connect and exchange traffic*, betting that **each new customer (carrier, enterprise, cloud) makes the platform more valuable to every other customer** — the network effect. The product evolution is visible in the company's own history: from carrier-neutral colocation, to "International Business Exchange" data centres, to **software-defined interconnection** (customers provisioning private, low-latency links between themselves, clouds and partners programmatically rather than with physical cross-connect tickets), to the 2020 re-branding as a "digital infrastructure company".

Why the distinction matters economically:

- **Pure real-estate colocation** monetises space and power at real-estate-like margins, and its revenue is bounded by the building's capacity.
- **Interconnection-led colocation** sells the *connections between tenants*: cross-connects, cloud on-ramps, private fabrics, and ecosystem membership. Interconnection revenue is widely analysed as higher-margin and faster-growing than space/power revenue, and it compounds — a building with 300 carriers and every major cloud on-ramp in its MMR is worth more *per kW* than an identical building in a connectivity desert. ***(flag)*** Precise interconnection-revenue percentages for Equinix are not quoted here because they were not re-verified against the latest filings at writing time; the *direction* of the claim (interconnection as the premium product) is consistent across the company's own investor materials and industry analyses.
- The industry-wide evidence for the same thesis: Digital Realty bought its way into interconnection (Telx, then Interxion's European carrier base); CyrusOne markets peering and carrier-neutrality as core services; and Uptime's 2024 survey shows colocation as the fastest-growing segment, with **nearly two-thirds of colocation providers hosting hyperscale tenants** — the ecosystem, not the shed, is what scales.

For a bank doing facilities strategy, the practical consequence: **choose colocation locations for their ecosystems** (carrier density in the MMR, cloud on-ramps, proximity to the markets and liquidity venues the bank trades with), and treat the cross-connect bill as a feature, not a fee.

---

## 13. Worked Example — Cymbal Bank's Data-Centre Strategy Review

*Cymbal Bank is a fictional global bank used throughout this research series to make strategy concrete; its estate below is illustrative and does not represent any real institution's facilities. Where this example needs regulatory or market specifics, it cross-refs the dedicated guides rather than re-deriving them.*

### 13.1 The Starting Estate

Cymbal Bank's platform runs on an estate typical of a mid-size global bank that grew by acquisition:

- **Two owned enterprise data centres** ("DC-A" and "DC-B") built in the late 2000s — nominally Tier III designs, both on 20-year-old raised floors, 600 mm rack pitches, air-cooled, designed for ~3–5 kW/rack, and now operating near power and space capacity. Neither has ever been through full Uptime constructed-facility certification; both carry self-assessed "Tier III+" labels that would not survive the §4.2 scrutiny.
- **A scattering of retail colocation cages** in five metros, acquired with regional acquisitions, on legacy contracts of varying quality.
- **A DR site** ("DC-C") in a different flood zone from DC-A, active/passive, used for batch and recovery workloads.
- An **AI/GPU pilot** just approved by the board — 100 racks of accelerators at 40–100+ kW/rack (§7.2) — with *no current facility* that can host it. This is the forcing function for the whole review.

The bank's resilience obligations (RTO/RPO by workload, regulatory expectations, testing cadence) are set out in the [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) and are assumed here, not restated.

### 13.2 Decision 1 — Which Tier for Which Workload Class

The first discipline from §4 is to stop asking "what Tier is Cymbal's data centre?" and ask "what Tier does *each workload class* need?" A working classification:

| Workload class | Example | Availability need | Facility strategy |
|----------------|---------|-------------------|-------------------|
| Real-time market infrastructure | Trading engines, market-data, payments gateways | Seconds-class failover; zero tolerance for planned outage | **Tier IV** capability *or* Tier III at two concurrently-active sites with application-level failover — choose the latter, which is far cheaper and matches how the apps are actually built |
| Core processing | Core banking, risk, CRM, middleware | RTO in hours; no *unplanned* maintenance downtime | **Tier III** (concurrent maintainability) — the mainstream choice |
| Batch / analytics | Overnight runs, reporting, model training (non-GPU) | Restartable; tolerant of maintenance windows | Tier II–III colocation |
| Dev / test | Lower environments, sandboxes | Outage-tolerant | Tier II retail colocation or cloud — do not pay for Tier IV |
| AI training (new) | GPU clusters, 40–100+ kW racks | Restartable with checkpoints; *density* is the constraint, not nines | High-density facility with liquid cooling — a *different axis* than Tier (⚠: a Tier IV air-only hall can still be useless for this workload, §6.4) |

The strategic output: Cymbal does **not** need Tier IV anywhere if its real-time applications can fail over between two Tier III sites in seconds — and they can, because that is how they are already designed for the DR pair. Tier IV's fault tolerance is for workloads that cannot fail *anywhere*; a two-site active-active pattern is the bank's actual answer, and it costs a fraction of a Tier IV build.

### 13.3 Decision 2 — Owned vs Colocation (Build vs Buy)

Cymbal's owned DC-A/DC-B are 15+ years old, at capacity, and would need full refurbishment (new power distribution, liquid-cooling plumbing, containment) to host anything modern. The board-level trade-offs, applied to the workload classes above:

| Dimension | Owned (build/refurbish) | Wholesale colocation | Retail colocation |
|-----------|------------------------|----------------------|-------------------|
| Capital | Full capex, 10–20 year asset | Low capex; long-term (5–15 yr) lease commitments | Opex-only, short commitments |
| Control | Complete (design, security, ops) | High (dedicated suites/halls, customer-configurable power/cooling) | Low (provider standards) |
| Speed | Slowest (years to power) | Months (if space exists) | Fastest (weeks) |
| Exit flexibility | Worst (sunk asset) | Medium (lease expiry; sublease risk) | Best |
| Operational burden | Full DC operations team | Provider runs plant; bank runs IT | Provider does almost everything |
| Ecosystem | None built-in | Provider-dependent | MMR density is the point (§12.3) |

For Cymbal the recommendation shape is a **hybrid**:
- **Consolidate and exit the old owned halls** (or convert DC-A to a non-production role) rather than sinking refurbishment capex into 2000s-vintage shells — the retrofit economics of §7.2 argue against it.
- **Take wholesale colocation** for the two production sites (Tier III, dedicated suites, liquid-cooling-ready), with lease terms long enough to amortise fit-out but short enough to keep exit optionality.
- **Keep retail colocation** for regional presence and MMR access where the bank's connectivity ecosystem lives (§12.3).
- **Build or lease-to-build one dedicated high-density hall** for the AI pilot — this is the one workload where "buy" may beat "lease", because high-density liquid-cooled capacity at 100 kW/rack is scarce and providers price it at a premium (⚠: scarcity pricing is market- and time-specific; validate with a current RFI).
- **Buy SG-specific context**: the build-vs-buy decision in Singapore runs through the local regulatory and energy regime (moratorium history, green criteria, DC-CFA operators) — see [singapore_data_centres_guide.md](singapore_data_centres_guide.md) §7 for that market's worked logic.

### 13.4 Decision 3 — DR Pairing Across Two Facilities

The physical-infrastructure contribution to DR is the *pairing of facilities* — the logical failover machinery is the operational-resilience sibling's subject. Cymbal's two production sites (call them Site 1 and Site 2, one owned/wholesale in each of two metros) must satisfy:

- **Active/passive with clean failover.** Site 1 runs production; Site 2 runs the recovery copy, and the pair must be independently operable — separate power grids/utility zones, separate flood/earthquake risk profiles, no shared single points (a shared substation or a shared carrier duct makes the pair a single facility in disguise — §5.1 and §2.2's diverse-entrance logic apply at the *pair* level too).
- **Distance vs synchronous replication.** The RPO=0 workload (real-time trading state) wants **synchronous replication** between sites, and synchronous replication is bounded by physics: every write waits on the round trip to the peer, so usable distance is limited to roughly the range where round-trip latency stays within the application's write budget — commonly cited as ~100 km of fibre or less for aggressive synchronous designs, with metro-scale pairs (tens of km) the norm for true RPO=0. ***(flag)*** Exact distance limits are workload- and design-dependent; the point is the trade: *the closer the pair, the tighter the RPO and the weaker the disaster independence.* Beyond synchronous range, async replication gives RPO in seconds-to-minutes and unlocks cross-region pairs — which is where the DR/BCP guide takes over.
- **One pair is not enough for a global bank.** Cymbal's board-level standard (cross-ref [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md)) needs regional pairs, because a regional event (SG-wide, EU-wide) can take out both members of an in-region pair — the classic reason regulators expect *two independent regions* for critical services.
- **Test the pair, not just the sites.** The facilities review must verify what the sites can actually do under failover load — power headroom at Site 2 (a DR site that has never run production does not know it is short of cooling until it tries), and the generator/UPS behaviour under a full failover transient. Uptime's operational-sustainability lessons (§9.1) apply to the pair as much as to a single building.

### 13.5 Decision 4 — The Provider Due-Diligence Checklist

Whichever facilities Cymbal selects, the same verification checklist applies to every candidate colocation provider. It operationalises everything this guide has established:

- **Tier claims are verified, not believed.** Check the facility against Uptime Institute's published certification registry (§4.2); ask which certification phase was achieved (design, constructed facility, operational sustainability) and when; treat "Tier III+" marketing with the §4.2 scepticism it deserves. An uncertified facility is not automatically bad — but the burden of proof shifts to its operational evidence (M&O-style reviews, outage history, audit reports).
- **The power deal is understood end to end.** Inclusive vs pass-through (§12.1), escalation basis, the per-subsystem redundancy actually delivered (UPS/generator/cooling/distribution/fuel — not a blanket "2N", §5.4), and the A/B path separation all the way to the outlet (§5.6).
- **Density and cooling headroom match the roadmap.** The candidate hall must host not just today's 8 kW racks but the 40–100 kW AI racks of the pilot (§7.2): liquid-cooling readiness (pipe runs, or space and policy to add them), floor loading, and per-row power limits written into the contract.
- **Efficiency and sustainability terms are contractual.** PUE and WUE reporting (methodology specified, §10.1–10.2), energy attribute certificates and their quality, and audit rights — because Cymbal's own carbon reporting depends on the provider's data (EU EED-style obligations are coming to more markets, §11.3).
- **The ecosystem is priced and tested.** Carrier density in the MMR, cloud on-ramps present or absent, cross-connect costs, and whether the interconnection story of §12.3 actually serves Cymbal's trading and connectivity needs.
- **Operations and security survive inspection.** M&O/TCOS standing (§9.1), maintenance and change processes, remote-hands quality (§9.2), physical-security layers and audit trails (§8), fire-suppression regime (§2.5) — and the provider's incident history, including how it reports and learns from outages.
- **Exit is designed, not assumed.** Term, break options, data egress, decommissioning and the portability of cross-connects — the mirror image of §1.3's retire phase, negotiated at entry rather than discovered at exit.

### 13.6 The Output

The review's deliverable is a workload-to-facility map (13.2), a build/buy position per metro (13.3), and a pairing standard for the estate (13.4) — with the AI pilot as the forcing function that finally retires the 2000s-vintage self-assessed "Tier III+" estate in favour of certified, density-capable, ecosystem-rich facilities. The through-line is the theme of this entire guide: **tiers are about configuration, not marketing percentages; facilities are about physics, not labels; and the best facility strategy is the one that matches each workload's actual risk and density profile** — with the DR/BCP depth, network design and market specifics in the sibling guides this one cross-references throughout.

---

## 14. Glossary, Claims Status and Further Reading

### 14.1 Glossary

- **A/B feed** — two independent power feeds to a rack, each from a separate distribution path; pairs with dual-corded IT (§5.5).
- **ASHRAE TC 9.9** — the ASHRAE technical committee whose *Thermal Guidelines for Data Processing Environments* (5th ed., 2021) set the recommended 18–27°C server-inlet envelope and the allowable equipment classes (§6.3).
- **Availability** — measured fraction of time a system is available: uptime ÷ (uptime + downtime), often quoted as "nines" (§10.3).
- **BMS / EPMS** — building management system (mechanical plant) and electrical power monitoring system (§9.2).
- **Cage / suite / cabinet** — customer-space units in a multi-tenant facility, from a locking rack up to a fully enclosed room (§2.3).
- **CRAH / CRAC** — computer room air handler (chilled-water coil, no compressor) vs computer room air conditioner (self-contained DX refrigerant) (§6.1).
- **Concurrent maintainability** — the Tier III property: any capacity component or distribution path can be serviced on a planned basis without impacting IT (§4).
- **DCiE** — data centre infrastructure efficiency, 1 ÷ PUE (§10.1).
- **DCIM** — data centre infrastructure management software; monitors and manages power, cooling, space and assets (§9.2).
- **Direct-to-chip / immersion / rear-door heat exchanger** — the three liquid-cooling architectures (§6.4).
- **Fault tolerance** — the Tier IV property: a single unplanned infrastructure failure does not affect IT (§4).
- **MMR (meet-me room)** — the facility's carrier/cloud demarcation room where tenants cross-connect (§2.2, §8.3).
- **M&O Stamp / TCOS** — Uptime Institute's operational-sustainability assessment; the Management & Operations Stamp is the version for non-Tier-certified sites (§9.1).
- **N, N+1, 2N, 2(N+1)** — power/cooling redundancy topologies: minimum count, one spare, two full systems, two full systems each with a spare (§5.4).
- **PUE / WUE / CUE** — power, water and carbon usage effectiveness: the Green Grid efficiency metrics, total facility resource ÷ IT energy (§10).
- **Raised floor vs slab** — underfloor-plenum construction vs slab-on-grade with overhead services (§2.1).
- **Tier I–IV** — Uptime Institute's classification of site-infrastructure topology: basic capacity, redundant capacity components, concurrently maintainable, fault tolerant (§4).
- **U (rack unit)** — 44.45 mm (1.75 in) of rack height; 42U/48U are standard enclosure heights; the OCP Open Rack uses 48 mm OpenU units (§7.1).

### 14.2 Claims Status — What Is Verified, What Is Marketing, What Is Not

**✅ Verified at primary sources**

1. Uptime Institute Tier I–IV definitions and properties (basic capacity → redundant capacity components → concurrently maintainable → fault tolerant) — [uptimeinstitute.com/tiers](https://uptimeinstitute.com/tiers).
2. The Tier availability myth: Uptime removed "expected downtime per year" from the Tier Standard in 2009, and percentages were never part of the Tier definitions — Uptime Institute eJournal, "Myths and Misconceptions Regarding the Uptime Institute's Tier Certification System".
3. Certification phases (Design / Constructed Facility / Operational Sustainability), two-year design-cert expiry since 2014, and the M&O Stamp — Uptime Institute certification and operations pages.
4. PUE definition and Green Grid origin (2007); ISO/IEC 30134-2 standardisation (2016); industry-average PUE 1.56 in the 2024 global survey (n=526), flat five years, new builds ~1.3 — Uptime Institute Global Data Center Survey 2024 report (July 2024).
5. WUE definition, Green Grid (2011), ISO/IEC 30134-9:2022, typical 0–2.5 L/kWh range — Green Grid / ISO references as compiled.
6. Modal rack density 4–6 kW (2024), average typical density 8 kW (7.1 kW ex-outliers), most facilities with no rack above 30 kW, highest densities rising in the 15–29 kW band; 29% of operators retrofitting halls; power the leading cause of impactful outages; 1-in-5 impactful outages >US$1M — Uptime Institute Global Data Center Survey 2024.
7. ASHRAE TC 9.9 recommended server-inlet envelope 18–27°C (5th ed., 2021) — ASHRAE thermal-guidelines documentation.
8. Provider public facts: Equinix (Redwood City HQ; Nasdaq EQIX; REIT since 2015; ~260 IBX sites in ~33 countries; revenue ≈US$9.2B FY2025), Digital Realty (Austin HQ; NYSE DLR; REIT; 300+ facilities/25+ countries; Telx/Interxion acquisitions), CyrusOne (Dallas HQ; taken private by KKR/GIP for US$15B, 2022) — company filings and disclosures as compiled.
9. NVIDIA GB200 NVL72 is a rack-scale, liquid-cooled system (36 Grace CPUs, 72 Blackwell GPUs) — NVIDIA product documentation.
10. IEA 2024 estimate: ~415 TWh (~1.5% of global electricity), potential doubling to ~945 TWh by 2030 — IEA analysis as compiled.
11. Open Rack standard: 21-inch bays, 48 mm OpenU, 48 V DC busbars, front serviceability; ORv3 combining OpenU with EIA-310 compatibility — OCP / Open Rack documentation.
12. Human error in >70% of outages; bad operational practices ~5× more likely than other causes to harm a DC — Uptime Institute operational-sustainability materials.

**⚠ Verified with cautions (marketing-vs-standard, or secondary-source figures)**

1. Tier availability percentages (Tier III ≈ 99.982%, Tier IV ≈ 99.995%, etc.) — widely circulated, traceable to legacy/TIA-942-style material, **not** certified by Uptime; treat as shorthand (§4.2).
2. AI rack power "20–100+ kW, flagship >120 kW" — consistent across industry commentary but not a single primary measurement; vendor-adjacent figures only (§6.4, §7.2).
3. "Air cooling impractical beyond ~30–50 kW/rack" and "water carries ~3,500× more heat than air" — engineering consensus figures, presented as design guidance rather than measured standards.
4. Generator fuel-storage "24–72 hours" and UPS battery "5–30 minute" bridge runtimes — typical design practice, not universal standards; site-specific (§5).
5. Per-kW colocation pricing and construction $/MW figures in broker literature — market/marketing data, deliberately not quoted as fact (§12.1).
6. Interconnection revenue as higher-margin than space/power — consistent with company investor materials and industry analyses; precise percentages not re-verified at writing time (§12.3).
7. Equinix global WUE 0.91 L/kWh (0.91 overall, 1.41 evaporative, 2025) — company sustainability report as compiled.

**❌ Could not be verified (see 14.3 for details)** — CUE's ISO/IEC 30134 part number; LBNL's headline US electricity-share figure; the synchronous-replication distance limit as a single number; OCP founding date; DCIM's claimed revenue/projection figures appearing in vendor blogs; various per-market cost benchmarks.

### 14.3 What Could Not Be Verified

- **CUE standardisation detail.** CUE's definition (carbon emissions per unit IT energy) is consistent Green Grid practice, but the precise ISO/IEC 30134 series part number for CUE could not be confirmed against the standards catalogue at writing time; treat any specific part number quoted elsewhere as unverified.
- **LBNL headline share-of-electricity figure.** The 2024 US Data Center Energy Usage Report's server-share figure (74%) is cited here, but its headline US electricity-consumption percentage was not re-read at source and is therefore not quoted.
- **Synchronous-replication distance limits.** "~100 km or less" is engineering folklore with real physical logic (round-trip latency bounds write latency), but there is no single authoritative number — the correct answer is workload- and design-specific, so this guide flags it rather than asserts it.
- **AI rack power numbers beyond the vendor.** "100+ kW racks, flagship >120 kW" are widely repeated but could not be pinned to an independent measurement; treat as indicative of the current flagship generation.
- **OCP founding detail.** "Launched by Facebook in 2011" is the standard account found in OCP materials but was not re-verified against a primary press release in this research pass.
- **Per-market colocation cost benchmarks** (US$/kW/month, construction US$/MW): intentionally not verified; broker-published figures are market-specific and volatile.

### 14.4 Primary Sources and Further Reading

- Uptime Institute — [Tier Classification System](https://uptimeinstitute.com/tiers); [Tier Certification of Operational Sustainability](https://uptimeinstitute.com/tier-certification/operations); [Global Data Center Survey Results 2024](https://uptimeinstitute.com/resources/research-and-reports/uptime-institute-global-data-center-survey-results-2024); eJournal: ["Myths and Misconceptions Regarding the Uptime Institute's Tier System"](https://journal.uptimeinstitute.com/myths-and-misconceptions-regarding-the-uptime-institutes-tier-certification-system/)
- The Green Grid / ISO-IEC — PUE and the 30134 series; [ISO/IEC 30134-9:2022 (WUE)](https://www.iso.org/standard/30134)
- ASHRAE — *Thermal Guidelines for Data Processing Environments* (TC 9.9, 5th edition 2021)
- Open Compute Project — [Open Rack specifications and designs](https://www.opencompute.org/projects/rack-and-power/)
- NVIDIA — [GB200 NVL72](https://www.nvidia.com/en-us/data-center/gb200-nvl72/)
- Equinix, Digital Realty, CyrusOne — company disclosures, filings and sustainability reports
- International Energy Agency — electricity-demand and data-centre analyses (2024–2025)
- Sibling guides: [l2_design_financial_data_centers_guide.md](l2_design_financial_data_centers_guide.md) (network layer), [singapore_data_centres_guide.md](singapore_data_centres_guide.md) (SG market/regulation), [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (DR/BCP), [devstack_openstack_guide.md](devstack_openstack_guide.md) and the Kubernetes/OpenShift guides (compute layer)

---

*A building, a power plant, a cooling plant, a security envelope and an operational discipline — and none of it matters until the applications run. That is the facilities umbrella: everything that keeps the bits alive below the network, the compute and the code. Welcome to the house of compute.*
