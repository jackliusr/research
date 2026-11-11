# Building an AI Factory - Power First, and the Acceptance Test Is the Workload

> **Author:** Jack Liu Shurui, Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research - AI Infrastructure series (how-to; read alongside the data-centre, GPU-cloud and Singapore data-centre guides)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary sources:** NVIDIA Glossary "What Is an AI Factory?" (retrieved 11 November 2026); NVIDIA AI Factories solutions page (retrieved 11 November 2026); NVIDIA Newsroom, "NVIDIA DSX Gives Infrastructure Builders the Playbook for AI Factories", 31 May 2026; HPE datasheet "NVIDIA GB200 NVL72 by HPE", PSN1014890104NLEN, November 2026; NVIDIA GB200 NVL72 product page (retrieved 11 November 2026); ASHRAE Thermal Guidelines for Data Processing Environments, 5th edition, 2021, and ASHRAE TC 9.9 liquid-cooling white paper, 2021; ANSI/TIA-942-C (2024), via TIA white paper DCWP041724, May 2024; Uptime Institute Tier Standard: Topology (updates effective 1 October 2018); MLCommons MLPerf Inference: Datacenter v6.1 (2026); NVIDIA DGX SuperPOD storage architecture page (retrieved 11 November 2026); TechCrunch report of Jensen Huang's GTC keynote, 19 March 2024; WhiteFiber SEC filings as recorded in [WhiteFiber Guide](whitefiber_guide.md); IMDA policy record as recorded in [Singapore Data Centres Guide](singapore_data_centres_guide.md).
> **Last Updated:** November 2026

**Table of Contents**

1. [Overview, the Two-Sense Decoder and the Thesis](#1-overview-the-two-sense-decoder-and-the-thesis)
2. [Build, Rent or Colocate - the Decision Before the Design](#2-build-rent-or-colocate---the-decision-before-the-design)
3. [The Sizing Chain, Step by Step](#3-the-sizing-chain-step-by-step)
4. [Power - the Binding Constraint](#4-power---the-binding-constraint)
5. [Cooling - Why Air Is Largely Over at AI Density](#5-cooling---why-air-is-largely-over-at-ai-density)
6. [The Network Fabric](#6-the-network-fabric)
7. [Storage and the Data Feed](#7-storage-and-the-data-feed)
8. [The Physical Build](#8-the-physical-build)
9. [Commissioning and Acceptance](#9-commissioning-and-acceptance)
10. [Operations](#10-operations)
11. [The Economics, Honestly](#11-the-economics-honestly)
12. [The Regulated and Banking Angle](#12-the-regulated-and-banking-angle)
13. [The Failure Modes](#13-the-failure-modes)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example)
15. [The Claims Audit and What Could Not Be Verified](#15-the-claims-audit-and-what-could-not-be-verified)
16. [Glossary and Closing Summary](#16-glossary-and-closing-summary)

---

## 1. Overview, the Two-Sense Decoder and the Thesis

Two different things are called an "AI factory" in current usage, and they share almost nothing but a metaphor. Read the phrase wrongly and you will buy the wrong thing.

**Sense (a): the platform sense.** An internal, governed GenAI platform - models, guardrails, evaluation harnesses, data pipelines - feeding use cases across a business. It is a software and operating-model construct. It needs compute, but the compute is a line item, not the project. This is what a bank usually means when someone in a strategy deck says "we are building an AI factory".

**Sense (b): the physical sense.** A purpose-built facility for AI training and inference, designed as a whole rather than assembled from a general-purpose data centre. It is a power, cooling, networking and concrete project, and its output is measured in tokens. This is what NVIDIA means, and it is what this guide is about.

### 1.1 The repository already owns sense (a)

This repository documents the platform sense in nine files. Three anchors do the work, and this guide will not re-derive them:

- [DBS Bank Guide](../banking/dbs_bank_guide.md), line 671, section 11.2, "AI at Scale: The GenAI 'AI Factory'": DBS's internal "AI factory" described as "a governed GenAI platform (models, guardrails, evaluation, data pipelines) feeding use cases across sales, risk, ops, and engineering".
- [Standard Chartered Guide](../banking/standard_chartered_guide.md), line 793, section 13.2: the same pattern, named "the AI factory pattern" - GenAI services, guardrails and evaluation treated as core infrastructure rather than as projects.
- [DBS Software Systems Guide](../banking/dbs_software_systems_guide.md), lines 369-371 and line 604, section 7.3, "GenAI: The 'AI Factory' Question and the Copilots": the correction - "DBS has not publicly branded an 'AI factory'" - and the conclusion that the term describes an internal architectural pattern, not a DBS product name.

That last file is the repository at its best: it was asked to verify a brand that does not exist, and it said so with a warning mark instead of manufacturing a citation. It is the standard this guide is held to.

Six further files touch the platform sense and can be read for it: [GPU Optimization Guide](gpu_optimization_guide.md), [OpenShift AI Alternatives Guide](openshift_ai_alternatives_guide.md), [Nutanix Enterprise AI vs OpenShift AI Guide](nutanix_enterprise_ai_vs_openshift_ai_guide.md), [HTX NGINE Guide](htx_ngine_guide.md), [Synapxe HealthTech Guide](synapxe_healthtech_guide.md) and [Banks in Singapore Guide](../banking/banks_in_singapore_guide.md). None of them describes a building. That is the gap this guide fills.

### 1.2 Where sense (b) comes from

The physical sense is a vendor's term, and the vendor is explicit about it.

NVIDIA's glossary defines it as follows: "An AI factory is a specialized computing infrastructure designed to create value from data by managing the entire AI life cycle, from data ingestion to training, fine-tuning, and high-volume AI inference. The primary product is intelligence, measured by token throughput, which drives decisions, automation, and new AI solutions." (NVIDIA Glossary, "What Is an AI Factory?", retrieved 11 November 2026. The page carries no publication date; the date given is the retrieval date. Flagged in section 15.3.)

NVIDIA's solutions page sharpens the contrast with the general-purpose hall: "Unlike traditional data centers, AI factories are purpose-built to manufacture intelligence at scale, tightly integrating accelerated infrastructure and AI software to optimize token generation" (retrieved 11 November 2026). The same page gives the five-layer framing: "AI factories unify five critical layers - energy, chips, infrastructure, models, and applications".

Provenance of current usage: Jensen Huang used the phrase in his GTC 2024 keynote, delivered 18 March 2024 - the same date as the Blackwell launch press release. TechCrunch's report of 19 March 2024 quotes him: "There's a new Industrial Revolution happening in these [server] rooms: I call them AI factories." Treat that as a press report of a keynote, not a transcript; the quote is as the outlet rendered it. Earlier NVIDIA usage than March 2024 may well exist and was not traced in this pass.

### 1.3 No standards body defines the term

This matters for a bank, because "certified AI factory" is a phrase a procurement team may one day be shown. It does not exist.

- The **Uptime Institute Tier Standard: Topology** (updates effective 1 October 2018) classifies a site's topology by redundant capacity components and distribution paths. It says nothing about AI, and it states that it "is independent of the IT systems operating within the site".
- **ANSI/TIA-942-C** (2024) is a requirements standard for telecommunications infrastructure in a data centre: site location, architectural, electrical, mechanical, fire safety, telecommunications and security. It does not certify workload performance.
- **ASHRAE** publishes thermal envelopes and liquid-cooling guidance, not facility classes for AI.

Industry uses the phrase loosely in at least three ways: for any purpose-built GPU hall, for any internal AI platform, and as marketing for both. When a vendor says "AI factory", the first question is which of those three is on the invoice. NVIDIA does not own the English language, but it does own the definition most of the market is quoting, and section 15 grades that definition as a vendor definition, not a standard.

### 1.4 The thesis, and what follows from it

**An AI factory is not a data centre with GPUs in it; it is a power-and-cooling project whose only honest acceptance test is the workload it was built for.**

Two consequences are argued through this guide rather than asserted here.

**(i) Power is the binding constraint.** Section 3 computes the megawatts forward, from workload to utility contract. Section 4 explains why the programme runs backward: the electrical supply has the longest lead time, so it is contracted first, and when the delivered megawatts come in below the computed ones, it is the GPU count that moves. The arithmetic and the schedule point in opposite directions. That is not a contradiction to be smoothed over; it is the central planning problem, and section 4 handles it head-on.

**(ii) Commissioning ends when the workload runs, not when the racks light up.** Section 9 defines the acceptance test as tokens per second per megawatt, with the model, sequence length, concurrency and latency target named in the contract, measured at the wall under MLCommons discipline. A rack count proves a building. It does not prove a factory.

Where the evidence complicates the thesis, the guide says so: an inference factory is not a training factory (section 5.4), and liquid cooling is not a monotonic improvement (section 5.5). A guide that only confirms its own thesis is a sales document.

### 1.5 What this guide is not

It is not a facilities primer: [Data Center Guide](data_center_guide.md) owns the electrical chain, cooling families, rack standards, PUE and physical security, and this guide cross-references rather than repeats. It is not a rental-market survey: [GPU Cloud Providers Comparison Guide](gpu_cloud_providers_comparison_guide.md) section 9 owns rent-versus-own. It is not about robots ([Physical AI Guide](physical_ai_guide.md)). It is the missing link between them: the build programme.

---

## 2. Build, Rent or Colocate - the Decision Before the Design

The most expensive mistake in this guide is made before any engineering starts: designing a facility because a facility is what the word "factory" implies. Design follows the access decision, never the other way round. Once a site is shortlisted and an architect is engaged, the build has momentum that no business case will stop.

[GPU Cloud Providers Comparison Guide](gpu_cloud_providers_comparison_guide.md) section 9, "Rent Versus Own", owns this decision from the buyer's side. It sets out the route set in section 9.1 (rented GPU capacity, an owned workstation GPU, an owned node placed in a colocation cage, an on-premises cluster, and a managed inference API), and it works the GPU-hour break-even arithmetic in section 9.2 with its assumptions exposed. Read it before this section. This guide adds only the facility-side considerations the rental view cannot see: what a building does to your cost base when demand moves, and what it does to your organisation when it does not.

### 2.1 Four conditions under which you should not build

**1. Utilisation you cannot fill.** A purpose-built facility has a fixed cost at five per cent utilisation and at ninety-five per cent. Rent is variable; concrete is not. If your workload is episodic - a quarterly fine-tuning run, a regulatory model refresh, a pilot that may not survive its own steering committee - then a building converts a variable cost into a permanent one and does it at exactly the moment you are least sure of demand. The relevant question is not "what is the cost per GPU-hour at full load" but "what is the cost per GPU-hour at the load I can actually commit to for five years". Answer the second one.

**2. Generation turnover faster than your depreciation.** An accelerator's economic life is shorter than a building's. This is the two-clocks problem, and section 11.2 takes it apart. In short: a shell, a substation and a cooling plant are 20-to-30-year assets, while the silicon inside is a 3-to-5-year asset at best, and the building will outlive several GPU generations. You are therefore not building one factory; you are building a host for a sequence of factories, each of which will want different power and different cooling. If the shell cannot be re-fitted for the third generation, you built the wrong shell. Renting outsources that bet to someone with more tenants to spread it across.

**3. The absence of operational staff.** A facility runs 24/7 whether or not your workload does. That means an on-call rota, an escalation path, electrical and mechanical competence you cannot improvise, and vendor support contracts with response times you have tested. If your current operations model is "the cloud provider's status page is our monitoring", you do not have the run model, and the facility will find that out at 03:00 on a Saturday during a genset test. Section 10 is the honest inventory of what this costs in people.

**4. A power contract you cannot get.** If you cannot contract the megawatts, the build decision is already made for you. Section 4 is the argument; the Singapore case in section 4.2 is the demonstration. A bank that has not established, in writing, that its target site can be energised at the required load by the required date has not decided to build - it has decided to hope.

There is a fifth consideration that is not a condition but a warning. A build programme is a two-to-four-year commitment of the same senior people who are supposed to be delivering the AI use cases. The facility will consume the architect who was going to design the guardrails. Budget the attention, not just the capital.

### 2.2 The honest statement

For a single institution's own AI workload, renting is frequently correct. Not "acceptable" - correct. The workloads most banks actually have (inference at moderate volume, periodic fine-tuning, evaluation runs) often fit rented capacity better than they fit a dedicated hall, because rented capacity is elastic and the workloads are not flat. A build guide that does not say this plainly is a sales document with a bibliography.

Building becomes defensible when at least three of the following hold: a large, steady inference base load that you can forecast; a data-residency or model-confidentiality constraint that no rental route satisfies; a power position you can actually contract; and an operations organisation that already runs 24/7 infrastructure. Fewer than three, and the honest answer is colocate or rent, and revisit in eighteen months.

The route that gets skipped in most bank debates is colocation, because it sounds like a compromise. It is not. A dedicated cage in a provider's facility gives the bank control of the physical layer - its own racks, its own access list, its own audit - while the provider carries the power relationship, the cooling plant and the 24/7 staff. What the bank gives up is the ability to specify the density beyond what the provider's hall supports, and the ability to re-let the space if the workload changes. Those are real costs, and they are smaller than the cost of a building that never fills. For a first AI facility at a bank, colocation is the modal correct answer, and the burden of proof sits with the build case rather than against it.

### 2.3 The decision table

Qualitative on purpose. The verified rental arithmetic and rate tables live in the GPU-cloud guide; this table is about which questions decide the answer, not about which is cheaper this quarter.

| Condition | Build | Colocate | Rent |
|---|---|---|---|
| Steady inference base load, forecastable for 3+ years | Preferred | Workable | Overpaying for elasticity you do not need |
| Episodic or pilot workload, demand unproven | Rejected | Workable | Preferred |
| Hard data-residency or weights-confidentiality constraint | Preferred | Workable if the cage is dedicated and audited | Depends entirely on the provider's tenancy model - check, do not assume |
| No 24/7 facilities operations capability | Rejected | Preferred (the provider runs the building) | Preferred |
| Power cannot be contracted at the site | Rejected | Depends on the provider's own allocation | Preferred |
| Need capacity in under 12 months | Rejected | Workable | Preferred |
| Workload needs rack densities above what a general-purpose hall can cool | Preferred (purpose-built) | Only if the provider has a high-density hall | Only if the provider's SKU matches |
| Regulated resilience obligation requiring site-level control | Preferred | Workable with contractual controls | Hardest to evidence |

Read the table twice. The pattern is that "build" wins on constraints and loses on time and uncertainty. Most banks have more uncertainty than constraint, which is why most bank AI factories should begin as colocation with a power option, not as a groundbreaking.

---

## 3. The Sizing Chain, Step by Step

Nine steps take a business demand and produce a utility contract size. Run them in order, write each output down, and keep the document - because the outputs of steps 2, 3 and 5 become the acceptance criteria of section 9. If the chain is done verbally, the acceptance test will be invented by whoever writes the commissioning script, and it will test the building.

Every step has the same four fields: the inputs you need, the output you produce, the decision the output forces, and what you must obtain and from whom. The last field is the one that stops the chain being a thought experiment.

### 3.1 Step 1 - Workload and model

**Inputs:** a business demand forecast (what volume of what interaction, at what growth, over what horizon) and a model plan (which model families, which sizes, which are fine-tuned rather than prompted, which are trained in-house).

**Output:** a workload specification - one page, signed, that says: training, inference, or both; the model families and parameter classes; the context length distribution (not the maximum, the distribution); and the demand unit, either tokens per day or GPU-hours per run.

**Decision it forces:** which kind of factory you are building. This is the fork that changes everything downstream, and section 5.4 explains why: a training factory is throughput-bound and an inference factory is latency- and utilisation-bound, and they want different buildings.

**Obtain:** the demand forecast from the business owner, in writing, with the assumptions visible; the model plan from the ML lead. If the business owner will not commit to a number, you are in a pilot, and section 2 says rent.

### 3.2 Step 2 - Required throughput

**Inputs:** the workload specification from step 1, and the latency expectations of the application that consumes it.

**Output:** sustained tokens per second and a p95 latency target, or GPU-hours per epoch for a training workload. Both are the numbers that later become the acceptance test, so state the measurement point: at the wall, per IT megawatt, per facility megawatt (section 9.4).

**Decision it forces:** what "fast enough" means in contractual language. A target of "low latency" is not a number and cannot be tested. A target of "p95 first-token latency under a stated value at a stated concurrency, at a stated tokens-per-second aggregate" can be tested, failed, and remediated.

**Obtain:** from the application owner, and benchmarked on your stack - your model, your quantisation, your serving framework, your sequence lengths. A vendor chart is a hypothesis about your workload, not a measurement of it.

### 3.3 Step 3 - GPU count and generation

**Inputs:** measured throughput per GPU for your model and software stack; MLPerf closed-division results for the same model as an outside sanity check; and a stated efficiency assumption for how much of the theoretical throughput you will actually sustain.

**Output:** a GPU count, plus spares (section 10.2), plus the number of GPUs per node and per rack, because those determine the rack count more than the total does.

**Decision it forces:** current generation or wait for the next. Waiting has a cost that is easy to hide (the use case does not ship) and a benefit that is easy to overstate (the next part is always 2x in a keynote). Choose the generation that lets the workload start, and design the facility to accept a hotter successor.

**Obtain:** vendor quotes with delivery dates; your own benchmark, run on your data; MLPerf closed-division results for the same model as a calibration point. Where the three disagree, your benchmark wins and the disagreement is a finding.

The reason this step is arithmetic and not procurement: throughput per GPU is a property of the pair (hardware, software stack). Change the serving framework and the number moves. That is why the number you defend must be one you measured.

### 3.4 Step 4 - Rack density and power draw

**Inputs:** the rack-level power figure from the integrator's datasheet, and the GPUs per rack implied by the node design.

The reference point for a current flagship rack is the HPE datasheet for the NVIDIA GB200 NVL72 system (document PSN1014890104NLEN, November 2026): "Each rack consumes 132 kW of power, with 115 kW liquid cooled and 17 kW air cooled." The same datasheet gives 48U, 72 NVIDIA Blackwell B200 GPUs and 36 Grace CPUs, up to 13.5 TB of HBM3E at 576 TB/s, and 6-8 1U 33 kW DC power shelves feeding a 50 V DC bus bar.

Divide 132 kW by 72 GPUs and you get approximately 1.83 kW per GPU all-in (derived, this guide). That number includes CPUs, switches, fans and power-conversion losses. It is the number a facilities engineer can use. A per-GPU thermal design power is not, and using one is the single most common sizing error in AI facility plans.

**Output:** kW per rack, racks, and IT megawatts (racks x kW per rack / 1000).

**Decision it forces:** air or liquid, and therefore hall shape. A 132 kW rack is not an air-cooled object (section 5). The choice is made here, at the sizing step, not later by the mechanical engineer.

**Obtain:** the integrator's rack datasheet, with the liquid and air split stated. Not a conference slide, not a partner's blog, not a number from a neighbouring bank's programme.

### 3.5 Step 5 - Megawatts

**Inputs:** IT megawatts from step 4, and a PUE target you are prepared to be held to in a contract.

PUE is defined by The Green Grid (2007) and standardised as ISO/IEC 30134-2 (2016); [Data Center Guide](data_center_guide.md) section 10.1 owns the definition and the benchmarks, including the Uptime Institute 2024 survey average of 1.56 and new builds around 1.3. This guide will not re-derive it. Use one number, state it as a design target, and accept that the audited figure will differ - which is why section 9 tests at the wall and normalises by IT megawatts as well as facility megawatts.

**Output:** facility megawatts (IT MW x PUE) and the utility contract size.

**Decision it forces:** which site, which utility, and whether to build at all. This is the step where the programme either has a power path or does not.

**Obtain:** a capacity offer from the utility or the site owner, in writing, with an energisation date and, critically, a delivery obligation. Section 4.3 explains the difference between an obligation and an effort, using a documented case where the difference was worth millions.

### 3.6 Step 6 - Cooling and electrical design

**Inputs:** facility megawatts; the water class you will design to (section 5.5); the delta-T across the loop; and the redundancy topology (N, N+1, 2N - see [Data Center Guide](data_center_guide.md) section 5.4).

**Output:** CDU capacity, flow rates, heat rejection capacity, and UPS and genset sizing. Flow follows from the load and the delta-T by physics, not by vendor preference: flow in litres per second equals power in kW divided by (density x specific heat x delta-T). At a density of approx. 0.997 kg/L and a specific heat of 4.186 kJ/kg-K, 100 kW at delta-T 10 K needs 2.40 L/s (approx. 8.6 m3/h); 1 MW at delta-T 10 K needs 24.0 L/s (approx. 86 m3/h); a 132 kW rack at delta-T 15 K needs 2.11 L/s (approx. 7.6 m3/h). Every watt that enters the IT leaves as heat, so the plant must reject the IT load continuously, not a fraction of it - 1 MW is approx. 3.41 million BTU/h.

**Decision it forces:** direct-to-chip, immersion or a hybrid; and the water question, which is a siting question. Water availability and treatment are local constraints, and in some jurisdictions they are the schedule risk rather than the power.

**Obtain:** the CDU and chiller vendors' datasheets with the water class and approach temperatures stated; the water authority's constraints in writing.

### 3.7 Step 7 - Physical space

**Inputs:** rack count, rack pitch, floor loading, ceiling height, and pipe and cable routes.

**Output:** square metres of hall, and of building.

**Decision it forces:** a new shell or a fit-out of an existing one. The fit-out is usually cheaper and almost always harder, because the constraints you inherit (slab strength, clear height, riser space) are the ones that decide whether high-density rows are possible at all.

**Obtain:** the structural engineer's floor-loading report and the landlord's as-built drawings. [Data Center Guide](data_center_guide.md) sections 2 and 7 own the anatomy and layout conventions; this guide does not re-teach them.

### 3.8 Step 8 - Network fabric

**Inputs:** GPU count, rail topology, and an oversubscription target for the east-west fabric.

**Output:** switch count, optics, fibre count, and rack-to-rack cabling.

**Decision it forces:** InfiniBand or RoCE, and single-rail or multi-rail. Section 6 argues the decision; here it is enough to note that it changes the rack count, because a rail-optimised fabric allocates leaf switches per rail and the cable tray fills up before the port count does.

**Obtain:** the fabric vendor's reference architecture for your exact GPU generation. Not the previous generation's, and not a generic CLOS design - the port mapping is specific.

### 3.9 Step 9 - Storage and the data feed

**Inputs:** dataset size, checkpoint size, checkpoint interval, and step time.

**Output:** required read and write throughput in GB/s, capacity, and the feed rate the data pipeline can actually sustain.

**Decision it forces:** parallel file system or object store with GPU-direct, and the budget line everyone forgets. Section 7 gives the numbers; the decision here is that storage is sized from the checkpoint and feed requirements of step 1, not from a capacity-per-GPU rule of thumb.

**Obtain:** measured dataset sizes from the data owners (measured, not estimated - the difference is routinely an order of magnitude), and checkpoint sizes from the training team.

### 3.10 Why the chain runs forward but the programme runs backward

The chain above computes megawatts from workload. It is the right arithmetic and it produces the number you need.

The programme cannot run in that order, because the steps do not have equal lead times. A GPU order is a purchase order with a delivery date measured in months. A transformer, a switchgear lineup and a utility interconnection are measured in years, and in some markets the queue is the constraint rather than the equipment. So the megawatts get contracted first, on an estimate, and the estimate is refined while the electrical equipment is on order.

This means the chain is computed forward and executed backward, and at some point the two meet - usually at the moment the utility's capacity offer comes back lower than step 5's number. The rule for that moment is in section 4.1 and it is not negotiable: the GPU count is the variable, because it is the only variable that can still move. This is why step 4's kW-per-rack figure and step 5's PUE target must be honest rather than optimistic. An optimistic PUE does not buy you a smaller utility contract; it buys you a facility that cannot run its own racks.

---

## 4. Power - the Binding Constraint

### 4.1 The apparent contradiction, and why it is not one

Section 3 runs the arithmetic from workload to megawatts. Section 3.10 says the programme runs from megawatts to workload. A reader is entitled to ask which direction is correct. Both are, and the reason is that they are answering different questions.

The arithmetic answers: **how much power would this workload need if power were available?** That number is a property of the workload and the hardware, and it is what you take to a utility, a landlord or a board.

The programme answers: **how much workload can this site support given the power it will actually deliver, by the date it will actually deliver it?** That number is a property of the site and the grid, and it is what you build to.

So the sequence is: compute the requirement, take it to the power, and then adjust the workload to the answer. The GPU count is the adjusting variable for three reasons. It is the only input that can change late, because silicon has a short order-to-delivery time relative to electrical equipment. It is granular, so it can be reduced in units rather than in binary. And it is the input whose reduction degrades the business case gracefully - fewer GPUs means a smaller factory with the same design, whereas fewer megawatts than the cooling plant needs means a factory that cannot be commissioned.

What must not be the adjusting variable is the PUE assumption, the kW-per-rack figure or the cooling design. Adjusting those does not reduce the requirement; it hides it. A facility planned at PUE 1.3 that runs at 1.45 has not saved energy, it has under-provisioned its own electrical plant, and the shortfall appears as derated racks rather than as a smaller bill.

The strongest form of the argument is a one-liner a board can hold onto: **you can always buy fewer GPUs, and you cannot always buy more megawatts.**

### 4.2 Power as a gated, allocated resource: Singapore

The cleanest documented case that power is allocated rather than purchased is Singapore, and the repository already holds the regulatory record in [Singapore Data Centres Guide](singapore_data_centres_guide.md) sections 2 and 5.

- **May 2019:** IMDA imposed a moratorium on new data-centre construction. Not a price signal, not a planning delay - a stop.
- **January 2022:** construction resumed, but only for "green" data centres meeting PUE 1.3 or better.
- **December 2025:** DC-CFA2, the second call for application, offering "at least 200 MW" for AI-ready, low-carbon data centres, with PUE below 1.3 and at least 50 per cent green power in the energy mix.

Read that sequence as an architect rather than as a policy analyst. For six and a half years, the constraint on a Singapore data centre was not capital, land, customers or silicon. It was an allocation, released in tranches, against criteria that had nothing to do with the workload. A bank designing a Singapore factory is designing into a process where the megawatts are awarded, and the award conditions (PUE, green-power share) become design requirements before the first drawing exists.

This is the strongest available demonstration of the thesis's first consequence, because it is dated, sourced and local to the guide's primary audience. It is also a warning about planning assumptions: a facility business case that models power as a purchased input with a market price would have been wrong about Singapore for most of the last decade.

### 4.3 Contract or effort: the Duke Energy case

The second documented case is in [WhiteFiber Guide](whitefiber_guide.md), and it is the best illustration in the repository of "power contracted but not delivered".

Under a Capacity Agreement with Duke Energy dated 16 May 2025, Duke would use "commercially reasonable efforts" to achieve 24 MW (gross) of service by 1 September 2025, 40 MW (gross) by 1 April 2026, and 99 MW (gross) within four years of 16 May 2025. "Commercially reasonable efforts" is an effort standard, not a delivery obligation. The site purchase price was made contingent on delivery: an additional $8m if 99 MW arrived within two years, $5m within three, plus $200,000 per MW above 99 MW capped at $5m. As of 30 June 2026, no Electric Service Agreement of more than 99 MW existed, so no contingent payment was payable.

The company also carried the cost of waiting rather than a remedy for it: a minimum monthly bill of $8,754 irrespective of actual usage, $11,405 a month of facilities charges on approximately $1,137,975 of installed extra facilities, and a $3m surety bond. And $322.5 million of NC-1 development costs were capitalised in the six months to 30 June 2026 (Form 10-Q, 12 August 2026).

Three lessons, and none of them is "Duke Energy is a bad utility":

1. **An effort standard is the default, and it is not a commitment.** If your agreement's power milestone is qualified by "commercially reasonable efforts", you have an aspiration with a legal format. Ask for a date certain with a remedy attached, and expect to pay for it.
2. **The contingent purchase price is the market's estimate of delivery probability.** A seller who accepts a bonus for delivery, rather than a discount for non-delivery, is telling you what it thinks of the schedule.
3. **The cost of waiting is real and it is not capitalised as risk.** A minimum bill, facilities charges and a surety bond are carrying costs on a facility that may not be able to earn.

One thing this guide will not do: divide $322.5m by a megawatt figure to produce a capex per MW. The filings do not disclose cost per MW for that site, the guide that owns the case records the gap, and a derived number would be exactly the kind of fabricated precision section 15 exists to prevent. If you need a cost per MW for your own business case, it comes from a contractor's budget estimate for your site, not from a proxy built on someone else's filings.

### 4.4 What to ask the utility for

A capacity offer is not a contract, and the difference is where programmes die. Ask for, in writing:

- **A capacity figure, in MW, at a stated voltage and connection point.** Not "up to". Not a range unless the range has a floor you can build to.
- **An energisation date, with the milestone schedule behind it.** Which substation, which feeder, which upgrade, which approval, which date.
- **A delivery obligation with a remedy.** A date certain, a liquidated position if missed, or at minimum a termination right without penalty. Compare it explicitly against the "commercially reasonable efforts" standard of section 4.3.
- **The conditions precedent.** Utility offers routinely depend on permits, on network upgrades by third parties, or on the customer funding "extra facilities" - the WhiteFiber case records approximately $1.14m of such facilities. Get the list.
- **The tariff structure, including minimum bills and demand charges.** This is the number the business case needs and the number this guide will not invent; it comes from the utility's published tariff schedule.

The reason this list has to be closed early is a lead-time asymmetry that governs the whole programme. A transformer, a switchgear lineup and the utility's own network upgrade are long-lead items measured in years, and their specification is fixed by the electrical design. A GPU order is measured in months and can be placed late, changed late, or reduced late. So the electrical design freezes first, against an estimate of the load, and the GPU order follows - which is the reverse of the order in which the numbers are computed. Section 3.10 states the consequence: if the utility's offer comes back below the requirement, the electrical design is already committed, and the GPU count is what moves.

### 4.5 Behind-the-meter generation

On-site generation is the standard answer to a slow interconnection, and it changes the risk profile rather than removing it. You trade a utility schedule risk for a fuel supply risk, an emissions-permit risk, a noise-and-planning risk with the local authority, and an operations risk that you now own a small power station. The fuel storage requirement is not incidental: the Uptime Institute Tier Standard has carried on-site power production fuel storage requirements since they became effective on 1 May 2010.

For a bank, behind-the-meter generation also raises a governance question that a utility contract does not: the facility's resilience now depends on a fuel contract and a maintenance regime that the bank controls, which means it appears in the bank's own operational risk inventory rather than in a supplier's.

### 4.6 Redundancy, and the weakest-subsystem rule

The design question is not "what tier is this facility" but "which single failure can take the workload down". The Uptime Institute Tier Standard: Topology (updates effective 1 October 2018) is explicit that "the Tier topology rating for an entire site is constrained by the rating of the weakest subsystem that will impact site operation", that there are no partial or fractional Tier ratings, and that the Standard "is independent of the IT systems operating within the site".

The practical consequence is the one that catches AI facilities: a 2N UPS lineup feeding a hall with N cooling is not a Tier IV site, and it is not a resilient facility either. The cooling is the weakest subsystem, so the cooling sets the resilience. In a high-density liquid-cooled hall, the CDU and the heat-rejection loop are therefore as much a resilience decision as the UPS, and they are usually the ones specified last.

[Data Center Guide](data_center_guide.md) sections 4 and 5.4 own the tier taxonomy, the nines myth and the N / N+1 / 2N / 2(N+1) topologies. The only thing this guide adds is the AI-specific reading: at 132 kW per rack, the cooling plant is not a supporting system, it is a co-equal one, and it belongs in the same resilience conversation as the electrical spine.

### 4.7 What could not be verified here

Grid interconnection queue statistics - median time from request to energisation, by market - were not retrievable in this pass. Web search returned empty results and two primary pages failed to load. The guide therefore argues from the two documented cases above (Singapore's allocation regime and the Duke capacity agreement) rather than from an industry average, and it does not quote a queue duration. If your business case needs one, obtain it from the system operator's published queue data for your jurisdiction, with a date.

---

## 5. Cooling - Why Air Is Largely Over at AI Density

### 5.1 The density argument

Air cooling is not obsolete. It is out of its range, and the range it is out of is exactly the one an AI factory lives in.

The repository's density curve in [Data Center Guide](data_center_guide.md) section 7.2 gives the baseline: the Uptime Institute 2024 survey puts the modal deployed rack at 4-6 kW, the average at 8 kW, and records that most facilities have no rack above 30 kW at all. Against that baseline, a 132 kW rack (HPE datasheet PSN1014890104NLEN, November 2026) is not an increment. It is 22 to 33 times the modal 4-6 kW rack, and it is more than four times the 30 kW ceiling that most existing halls never cross.

TIA's own material records the direction of travel: in 2022, 25 per cent of enterprise data centres reported rack densities greater than 20 kW, and some hyperscale extreme-density racks had reached 80 kW or higher (TIA white paper DCWP041724, May 2024). The same paper's Figure 4 - sourced to ASUS, and flagged here as vendor-sourced rather than independent - places air cooling at around 50 kW per rack, single-phase direct-to-chip at 25-90 kW, and immersion at 50-100 kW.

Two sourced numbers are enough to make the point. The repo says air becomes impractical around 30-50 kW per rack. TIA's figure puts the air ceiling around 50 kW. A 132 kW rack is above both. Whatever the exact crossover, it is well below the rack you are trying to install.

The engineering reason is air's heat capacity per unit volume, which is low. Delivering 132 kW to a rack with air means a very large volumetric flow at a workable temperature difference, and the fan power and the aisle containment to support it consume the density advantage you were trying to gain. This is why the industry's answer is liquid: water carries far more heat per litre, so the same load moves in a pipe instead of a duct. The flow arithmetic is in section 3.6 - 132 kW at a 15 K delta-T is approx. 2.11 L/s, which is a pipe, not a wind tunnel.

### 5.2 The liquid families, and what each is for

- **Direct-to-chip.** A cold plate sits on the processor; the loop carries heat from the die to a CDU and out to the plant. It cools the components it touches and leaves the rest of the rack (memory in some designs, NICs, optics, power shelves) to air. This is why the HPE datasheet's 132 kW splits into 115 kW liquid and 17 kW air: even a fully liquid-cooled flagship rack still needs an air path for roughly 13 per cent of its load. Any design that assumes "liquid-cooled means no air handling" is wrong by that margin.
- **Rear-door heat exchangers.** A passive coil in the rack's rear door; a transitional technology that raises the practical ceiling of an air hall without re-plumbing the room. Useful, and limited - it moves heat into water at the row, but the rack still has to be fed and the door has to be openable.
- **Immersion.** The rack's components sit in a dielectric fluid, single-phase or two-phase. High heat capture, and a different operations model: maintenance means lifting hardware out of a tank, and the fluid is a supply chain of its own.

The TIA ranges above are the only per-rack numbers this guide will print for these families, and they are vendor-sourced. Do not use them to size a plant. Use the vendor datasheet for the equipment you are actually buying.

### 5.3 The retrofit problem is structural, not electrical

The tempting plan is to take an existing 4-6 kW hall and "upgrade" it. Three things make that a construction project rather than a procurement one.

First, the cabinets. ANSI/TIA-942-C (2024) "requires a minimum of 800mm-wide cabinets in functional areas that house switches (i.e., MDA, HDA, and IDA)". A hall built to an older, narrower standard cannot be brought to that requirement by rearranging racks - the floor grid, the aisle widths and the cable pathways were all set at build time.

Second, the routes. Liquid cooling needs supply and return pipework to every rack, with isolation, leak detection and drainage. In a finished hall, that pipework has to go somewhere: above the racks, in a raised floor that may not have the depth, or in a new riser. None of those is a change order; they are drawing revisions.

Third, the floor. High-density racks concentrate load, and the slab and the structure either take it or they do not. The number is the structural engineer's, and it is not negotiable by the IT team.

This is why section 3.7's "new shell or fit-out" decision is a real fork, and why the answer is often "fit-out for a moderate density now, new shell for the high-density hall later".

### 5.4 An inference factory is not a training factory

The thesis bites differently depending on which one you are building, and a guide that pretends otherwise is selling a single design.

A **training factory** is throughput-bound. Its economics are GPU-hours per run, its fabric is sized by the all-reduce, its storage is sized by the checkpoint, and its utilisation target is high and bursty. Throughput per megawatt is close to the right objective function, because the job either finishes sooner or later.

An **inference factory** is latency- and utilisation-bound. Its economics are tokens per second at a p95 latency target, at a concurrency the application actually produces. That target is frequently set by something outside the facility - a customer-facing interaction budget, a fraud decision that has to complete inside a transaction - and it does not care how efficiently the building is running. An inference hall optimised for throughput per megawatt can be the wrong building, because the configuration that maximises aggregate tokens (large batches, high queue depth) is the configuration that misses the latency target.

The practical consequence: **the building is similar, the acceptance test is not.** Section 9.4 requires the acceptance test to name the workload class, because "tokens per second per megawatt" measured on a saturating batch job tells you nothing about whether the inference service meets its p95. If you are building for both, you are building two acceptance tests and probably two halls.

### 5.5 Liquid cooling is not a monotonic improvement

The second honest complication is that warmer facility water, which is generally good for energy and for water use, costs cooling capability. This is not intuition; ASHRAE says it.

ASHRAE TC 9.9's 2021 white paper "Emergence and Expansion of Liquid Cooling in Mainstream Data Centers" records the water classes as "W17 (previously W1), W27 (W2), W32 (W3), W40 (new), W45 (W4), and W+ (W5)". The same paper states: "The ASHRAE water cooling class W27 can support up to 500 W for each CPU and up to 12 W per DIMM with single-sided DIMM cooling. On the other hand, ASHRAE water cooling class W45 reduces the cooling capability to a point that it may be comparable to a conventional air-cooled system with a low cold aisle temperature for this dense 1U half-width shadowed configuration."

Read that carefully, because it is the opposite of the marketing story. A warm-water loop (W45) can give you roughly the cooling performance of a cold-aisle air system for a dense node. The water class is a design decision with a performance cost, not a free upgrade. It affects which components you can cool, and therefore what the rack can hold.

The decision is therefore a trade: warmer water improves the plant's efficiency and its ability to use dry coolers rather than chillers, and it reduces the heat you can pull off the die. Choose the class against the silicon's requirements, not against the PUE target alone.

### 5.6 What the standards specify, and what they do not

The ASHRAE Thermal Guidelines for Data Processing Environments, 5th edition (2021) give a recommended server inlet range of 18-27 C for the common air classes. TIA's white paper records that the 5th edition added class H1, with a narrowed 18-22 C recommendation for high-density systems - a useful reminder that "run hotter to save energy" has a boundary set by the hardware, not by the energy model.

What ASHRAE does not do is certify a facility, approve a design, or tell you what your cooling plant must be. It publishes envelopes and guidance. A design that cites ASHRAE is citing a thermal requirement; it is not citing an approval.

For the cooling families themselves, and for CRAC versus CRAH, containment and economisation, [Data Center Guide](data_center_guide.md) section 6 owns the material. This section has added only what changes at AI density: the numbers that put air out of range, the liquid families, the retrofit constraints, the inference-versus-training distinction, and the W-class caveat.

---

## 6. The Network Fabric

### 6.1 The fabric is sized by the all-reduce, not by the users

In a general-purpose data centre, the network is sized by traffic between applications: request rates, east-west service calls, database chatter. In an AI factory, the number that sizes the fabric is the collective operation. A training step ends with a gradient all-reduce across every GPU in the job, and that operation is not a rounding error on the traffic - for large models it is most of it, in a burst, every step.

Two consequences follow, and both are design inputs rather than procurement details.

**The fabric's failure mode is not congestion, it is idle silicon.** A slow fabric does not make the application feel sluggish; it makes the GPUs wait, which converts directly into wasted megawatt-hours. This is why AI fabrics are specified for latency and for the absence of congestion rather than for average throughput.

**The fabric is part of the rack count.** A rail-optimised design allocates leaf switches per rail, so the switch count and the cable count scale with the GPU count and the rail count, not with the number of server ports. Section 3.8 puts the fabric in the sizing chain for this reason.

### 6.2 InfiniBand or RoCE

This is a design decision with an operational consequence, and it is frequently made on the wrong grounds - usually a preference for "the Ethernet team already knows Ethernet".

**InfiniBand** is a purpose-built fabric: lossless by design, with credit-based flow control, and with a mature management stack for large GPU clusters. It is the default in the reference architectures the GPU vendors publish for their own rack-scale systems, which is a strong argument: the vendor has already debugged it at your scale.

**RoCE** (RDMA over Converged Ethernet) runs remote direct memory access over Ethernet. It is genuinely attractive for a bank: one fabric technology, one team, one set of operational tooling, and a path to the existing network estate. But RoCE achieves lossless behaviour through configuration, not through the protocol's nature. It depends on priority flow control and on explicit congestion notification being correctly configured across every switch and every host, and on that configuration staying correct through every change.

That is the point to hold onto: **RoCE is an operational discipline, not a purchase.** A misconfigured priority flow-control setting or a PFC storm does not degrade a RoCE fabric gracefully; it can stall it. If the bank's network change process is not strong enough to protect a lossless configuration across years of changes, choose InfiniBand and remove the variable.

Neither choice is a standards question - both are deployed at scale. The authority is the fabric vendor's reference architecture for your exact GPU generation, and section 6.4 explains why "exact" is doing real work in that sentence.

### 6.3 Topology and oversubscription

The vocabulary is the repository's already. [L2 Design for Financial Data Centers Guide](l2_design_financial_data_centers_guide.md) section 3 owns leaf-spine and CLOS: why CLOS won (section 3.1), how oversubscription sizes a fabric (section 3.2), spine count and failure domains (section 3.3), the underlay/overlay split (section 3.4) and the physical-layer notes on optics, cables and ports (section 3.5). Read those; this guide will not re-teach them.

What changes for AI is the target. In an enterprise leaf-spine, a significant oversubscription ratio between access and spine is a normal cost optimisation, because the traffic is bursty and forgiving. In a training fabric, oversubscription on the GPU-to-GPU path converts to idle GPUs during the all-reduce, so the design target moves toward non-blocking for the GPU plane, with the oversubscription pushed out to the storage and management planes where it can be tolerated. The ratio you choose is a decision to be made with the fabric vendor's reference architecture for your rail count, not a rule lifted from the enterprise network.

Rail-optimised topology is the specific case worth naming: rather than every GPU reaching every leaf, each GPU in a node connects to the leaf that serves its rail, so a collective across N nodes traverses a predictable set of paths. The benefit is that the fabric scales with the GPU count without the spine becoming the bottleneck. The cost is that the cabling is specific to the node design and cannot be re-patched for a different GPU generation without re-planning the tray.

### 6.4 The physicality nobody budgets for

A current rack-scale GPU domain is a copper-and-fibre problem before it is a protocol problem, and the numbers are large.

The NVIDIA GB200 NVL72 product page (retrieved 11 November 2026) describes "36 Grace CPUs and 72 Blackwell GPUs in a rack-scale, liquid-cooled design", with a 72-GPU NVLink domain carrying 130 TB/s of GPU communications and 1.8 TB/s of GPU-to-GPU bandwidth. Those figures are the reason the cabling is hard: 72 GPUs in one domain means every GPU is cabled to a switch fabric inside or beside the rack, and the copper for the intra-rack links is short-reach but voluminous.

The practical budget lines that get missed:

- **Tray fill.** A tray filled to capacity at install cannot accept the next generation's cabling. Specify headroom as a percentage and put it in the drawing.
- **Bend radius.** Copper and fibre have minimum bend radii, and a densely packed rack with a tight cable manager violates them. The failure is intermittent, which means it appears as a training job that hangs rather than as a cable that is obviously broken.
- **Fibre count.** ANSI/TIA-942-C recommends at least two optical fibres for horizontal and backbone cabling - a minimum for general infrastructure, not a count for an AI fabric, where the GPU plane's requirement comes from the vendor's reference architecture for your rail count and is not bounded by the standard.
- **Weight and support.** A fully cabled high-density rack is heavy, and the cable tray has to be supported as structure.

None of this appears in a bill of materials as "cabling". It appears as a delay in commissioning, a re-work charge, and a fabric that passes its link test and fails its collective benchmark. Budget the physical layer as a first-class line item, with a witness test at install.

---

## 7. Storage and the Data Feed

### 7.1 The checkpoint problem

Training a large model means writing the model's state to storage at intervals, so that a failure does not cost the whole run. Those writes are not small and they are not asynchronous.

NVIDIA's DGX SuperPOD reference architecture, "Storage Architecture" page (retrieved 11 November 2026), states the problem directly: "The size of checkpoint files can be terabytes in size and while not written frequently are typically written synchronously that blocks forward progress of DL models." Two facts in one sentence, and both matter: terabytes, and synchronous.

Synchronous means the training job stops while the checkpoint is written. A checkpoint that takes longer than the interval between checkpoints is not a checkpoint policy, it is a permanent stall. The sizing question is therefore not "how much capacity" but "at what write throughput, so that the pause is a small fraction of the interval". Take the checkpoint size from the training team, the interval from the risk appetite for lost work, and divide.

### 7.2 The feed rate

The read side is sized by the dataset and by how fast the GPUs can consume it. The SuperPOD page gives tiered guidance that is the most useful published starting point this guide could verify:

| Scope | Metric | Good | Better | Best |
|---|---|---|---|---|
| Single node | Read (GB/s) | 4 | 8 | 40 |
| Single node | Write (GB/s) | 2 | 4 | 20 |
| Single scalable unit, aggregate | Read (GB/s) | 15 | 40 | 125 |
| Single scalable unit, aggregate | Write (GB/s) | 7 | 20 | 62 |
| Four scalable units, aggregate | Read (GB/s) | 60 | 160 | 500 |
| Four scalable units, aggregate | Write (GB/s) | 30 | 80 | 250 |

Two further statements from the same page set the floor and the ratio:

- "In these cases, 4 GBps per GPU for read performance is needed" - the case being large uncompressed image datasets, often over 30 TB. That is per GPU, which is the number that turns a storage design into an arithmetic problem rather than a preference.
- "Consider at least 1/2 of the read performance as recommended write performance for LLM and large model use cases." Write is not an afterthought; it is half the read figure.

The same page records that GPUDirect Storage on the SuperPOD storage fabric should read "over 27 GBps" directly into GPUs, against a per-node network maximum of 80 GBps. The gap between those two numbers is the reason GPU-direct matters: the path from storage to GPU memory bypasses the host, and the storage fabric becomes the limit rather than the host's PCIe budget.

### 7.3 Why storage is the line item that gets cut

Storage is underestimated for a structural reason: it is the only major component whose requirement is invisible until the workload runs. A GPU count is in the purchase order. A rack count is on the drawing. The storage throughput requirement is a number in a reference architecture that nobody has measured against their own job yet. So it is the line item that gets trimmed in the budget review, and it is the one that caps the training job six months later.

The failure is quiet and expensive. The job runs; it simply runs slower than the GPUs allow, because the data loader is waiting. The GPUs are still consuming their megawatts. This is the worst kind of under-provisioning in an AI factory: it does not fail a test, it degrades a number that nobody is measuring per-job, and it appears in the monthly bill as "training is taking longer than planned".

The defence is to size storage from step 9 of the chain - dataset size, checkpoint size, interval, step time - and to write the feed rate into the acceptance test, not into a capacity plan.

### 7.4 The acceptance test for storage

The acceptance test is a measured feed rate during a real job, not a vendor's IOPS number.

A vendor's IOPS figure is typically measured on a synthetic pattern, often cached, often at a queue depth your workload will not produce. It tells you the device can do it in the vendor's lab. The question is whether your training job, with your data loader and your file sizes, sustains the GB/s that step 9 requires.

So the test is: run the actual job, at the actual scale, with the actual dataset, and measure the bytes per second delivered to the GPUs while the job is running - and simultaneously measure GPU utilisation. If utilisation is high and the feed rate matches the plan, storage passes. If utilisation is low and the feed rate is below plan, storage fails, and no amount of capacity will fix it.

This is the same discipline section 9 applies to the whole facility, applied to one subsystem. It is also why section 9.4 insists that the acceptance test names the workload: a storage system that passes on a synthetic benchmark and fails on the real job has passed the wrong test.

For the parallel-file-system versus object-store decision, and for the vendor landscape, this guide has no verified comparison to offer and prints none. The design question is answerable from the numbers above: a parallel file system is the conventional answer for the checkpoint and feed pattern described here, and an object store with GPU-direct is a legitimate alternative when the data is already object-native and the access pattern is read-mostly. Choose against measured feed rate, not against a vendor's positioning.

---

## 8. The Physical Build

### 8.1 What is different about this building

The building is not exotic. It is a data hall with a heavier electrical service, a liquid loop and a bigger structural load, and the repository already owns the anatomy: [Data Center Guide](data_center_guide.md) section 2 covers the shell, the rows and the rooms, and section 7 covers layout and racks. This section adds only the four things that change when the tenant is a rack-scale GPU system.

### 8.2 The shell decision

Two routes, and the choice is made early and expensively revised later.

**A new shell** lets you set the floor loading, the clear height, the riser positions and the electrical rooms from the design of the hall you actually want. It takes longer, needs planning consent, and commits capital before you have a workload running - which is precisely the bet section 2 warns about.

**A fit-out** is faster and cheaper per square metre, and it inherits constraints that cannot be designed away: slab capacity, clear height, the position of existing risers, the rating of the incoming service. A fit-out that cannot take the pipe routes is not a fit-out of an AI hall; it is an air hall with ambitions.

The decision rule that survives contact with reality: fit out if the inherited constraints permit the density you need at the second GPU generation, not just the first. If they only support the first, you are building a facility with a known expiry date.

One vendor now sells the step before this decision. NVIDIA's DSX release of 31 May 2026 quotes Jensen Huang: "With the DSX platform, you can simulate the entire factory before you spend a dollar, validate performance before a single rack is installed". That is a vendor claim for a design tool, and treated as one it is useful: simulating the electrical, thermal and workload behaviour before committing a shell is cheaper than discovering the interaction afterwards. It is not a substitute for the sizing chain in section 3, because the simulation is only as good as the numbers put into it, and those numbers come from your benchmark, your utility and your integrator.

### 8.3 Floor loading and the high-density row

High-density racks concentrate weight as well as power. A rack full of liquid-cooled compute, filled with cold plates, manifolds and CDUs, is a different structural proposition from a rack of 1U servers, and the load is not evenly distributed across the hall - it is in rows.

Three consequences for the layout:

- **Rows, not a uniform grid.** The high-density rows carry the manifolds and the CDU connections, and they should be placed to keep pipe runs short and to keep the heat-rejection plant close.
- **The slab is a constraint, not a detail.** The number comes from the structural engineer, and it is a hard limit on where the heavy rows can go. It is also the single most common reason a fit-out fails at the design review.
- **Service access.** A CDU needs maintenance access, and a manifold needs isolation valves that a technician can reach without standing in a live aisle. This is a layout decision that is expensive to change after the racks are installed.

[Data Center Guide](data_center_guide.md) sections 2.4 and 7.3 own the raised-floor and layout conventions; the AI-specific addition is that the row is now a thermal and hydraulic unit as well as an electrical one.

### 8.4 Rack power distribution

The rack power chain is where the datasheet numbers become a procurement list. The HPE GB200 NVL72 datasheet (PSN1014890104NLEN, November 2026) describes 6-8 1U 33 kW DC power shelves feeding a 50 V DC bus bar, in a 48U rack drawing 132 kW with 115 kW liquid cooled and 17 kW air cooled.

Three design implications:

1. **The rack is fed at DC, inside the rack.** The AC-to-DC conversion happens in the power shelves, not in a room-level UPS output. That changes what the facilities team is maintaining: the UPS protects the AC feed, and the shelves are IT equipment with their own failure modes and their own spare parts.
2. **The power shelf is a redundancy unit.** With 6-8 shelves at 33 kW each for a 132 kW rack, the shelf count is the N+1 decision, and it is made by the integrator's configuration rather than by the facilities engineer. Ask for the configured shelf count and the resulting redundancy explicitly.
3. **The bus bar is the distribution.** A 50 V DC bus bar replaces the row-level PDU as the thing the rack connects to. The cabling is different, the metering is different, and the electrician's toolkit is different. [Data Center Guide](data_center_guide.md) section 5.6 covers the conventional PDU / RPP / busway chain; this is the variant that arrives with the GPU rack.

The procurement rule: get the rack's power architecture from the integrator's datasheet, and get the facilities side from the electrical contractor's design. Do not let either document be written from the other's summary.

### 8.5 Physical security, briefly

The layered model is the repository's ([Data Center Guide](data_center_guide.md) section 8: layers, monitoring and response, the meet-me room nuance, and security as an operational programme). The AI-specific change is not the model but the concentration: a purpose-built facility for one workload puts the weights, the training data and the training run in one hall. Section 12.3 takes that up as a banking question, because it is a governance question before it is a security one.

---

## 9. Commissioning and Acceptance

Commissioning is the point at which the project stops being a set of contracts and becomes a facility. It is also where the thesis is either proven or quietly abandoned, because the default commissioning test - count the racks, power them up, run a burn-in - is a test of the building.

### 9.1 Factory acceptance test (FAT)

The factory acceptance test happens at the vendor's works, before shipment, against a written test procedure. It is the last moment at which a defect is the vendor's logistics problem rather than your schedule problem.

Done properly, a FAT has four properties:

- **A written procedure, agreed before the equipment is built.** Not a datasheet, not a standard's summary - a document that says which parameter is measured, with which instrument, at which point, against which acceptance value.
- **A named witness.** Someone from your side, or an independent engineer acting for you, signs the result. If nobody witnesses it, you have a vendor's report, not a test.
- **Records that travel with the equipment.** The signed procedure, the readings, the instrument calibration certificates, and the serial numbers of the units tested. When a unit fails on site, the FAT record is what tells you whether it failed in transit or was never right.
- **A pass definition that includes the interfaces.** A CDU that meets its own flow and temperature specification but does not match the manifold it connects to has passed its FAT and failed the project.

For a GPU rack, the FAT is also where the liquid and air split gets verified - the HPE datasheet's 115 kW liquid and 17 kW air are design figures, and the test is whether the as-built rack behaves as designed.

### 9.2 Site acceptance test (SAT)

The site acceptance test repeats the FAT procedure on site, plus everything that only exists on site: the interfaces between the vendor's scope and everyone else's. Power quality at the rack, the actual flow and return temperatures from the real plant, the leak detection, the controls integration, and the alarms that must reach the building management system.

The SAT is where the "boundary" failures live. A CDU vendor's scope ends at the flange; the mechanical contractor's begins there; the controls contractor's begins at a signal that neither of them owns. SAT is the only test that exercises the seam, and the seam is where commissioning schedules slip.

### 9.3 Burn-in

Burn-in is a sustained load test at design density for a defined period. It has two phases and they are not interchangeable.

**Load-bank burn-in, before the IT arrives.** Resistive load banks at the rack position, at design density, for long enough to prove the electrical and cooling plant under sustained load - not a one-hour demonstration, but a test that includes a utility failure and a generator transfer, and a chiller or CDU failure if the redundancy design claims to survive one. This is the test that finds the plant defects while the hall is empty and the remedy is a warranty call.

**Workload burn-in, after the IT arrives.** The real workload, at the real scale, for a defined period, with the acceptance metrics being measured continuously. This is the phase that produces the acceptance result, and section 9.4 defines it.

A facility that does only the first has proved it can dissipate heat. A facility that does only the second has never tested its redundancy. Neither alone is commissioning.

### 9.4 The acceptance test is the workload

This is the thesis's second consequence, expressed as a contract term.

**The acceptance criterion is tokens per second per megawatt.** Not racks energised, not GPUs detected, not a benchmark score. And the unit has to be defined twice, because the two definitions answer different questions and a vendor will pick whichever is more flattering:

- **Tokens/s per IT MW** = sustained tokens per second divided by the power drawn by the IT equipment alone. This measures the compute efficiency of the design. It is the number an architect argues about.
- **Tokens/s per facility MW** = sustained tokens per second divided by the total facility power, including cooling, losses and everything the meter sees. This measures the factory. It is the number the bank pays for, and it is the one that should be in the contract, with the IT-MW figure reported alongside it.

The measurement discipline comes from MLCommons. MLPerf Inference: Datacenter (v6.1, 2026) measures power as the "measured average AC power (energy) consumed by the entire system for the duration of the performance measurements of a benchmark ... the AC power is measured at the wall". And it is explicit about what that excludes: "MLPerf Power is only capable of measuring and validating the full system power. Any other references to power in any description (e.g., a TDP configuration, a power supply rating) are not measured or validated by MLCommons."

That last sentence is the authority for refusing a TDP-based acceptance test. If a vendor offers to demonstrate efficiency by citing a thermal design power, the answer is that the number is not measured at the wall and therefore not a facility number. The same logic is why section 3.4 uses 132 kW / 72 GPUs rather than a per-GPU TDP.

A testable acceptance criterion therefore has five named parameters, and all five belong in the contract:

1. **The model** - by name and version, including quantisation.
2. **The sequence length** - input and output, since throughput varies enormously with context length.
3. **The concurrency** - the number of simultaneous requests, because a saturating batch and a realistic arrival rate are different tests.
4. **The latency target** - p95, at that concurrency, for the metric that matters (time to first token, or inter-token latency, or end-to-end).
5. **The power measurement point** - at the wall, per MLPerf discipline, with the duration of the measurement stated.

Without all five, "tokens per second per megawatt" is a slogan. With all five, it is a test that can fail, and a test that can fail is the only kind worth writing.

### 9.5 What a rack-count test proves

A rack-count acceptance test proves the building: that the racks arrived, that the power is present, that the cooling loop is closed, that the network links. All necessary, none sufficient.

It does not prove that the facility can do the job it was built for. A hall can pass every rack-count test and still deliver half the expected throughput, because the fabric is oversubscribed, or the storage cannot feed the job, or the cooling cannot hold the clock frequency at sustained load, or the scheduler is misconfigured. Every one of those is invisible to a rack count and fatal to the business case.

The commercial reason to insist on the workload test is that it is the only point at which the vendor, the integrator and the bank are all measuring the same thing. After handover, a throughput shortfall becomes an argument between three parties with three sets of numbers. At acceptance, it is a test result.

---

## 10. Operations

### 10.1 The run model

A facility needs a run model before it needs a building, because the run model determines whether the building can be operated at all. Four questions define it:

- **Who is on call, and for what?** The electrical and mechanical plant, the cooling loop, the network fabric and the GPU fleet have different failure signatures and different responders. A single rota that treats them as one is a rota that will escalate a CDU leak to a GPU engineer.
- **What is the escalation path, and has it been tested?** A path that exists on a slide is not an escalation path. Test it during commissioning, at night, with the vendor's actual support line.
- **What does each vendor support, and to what response time?** The GPU vendor, the integrator, the CDU vendor, the chiller vendor, the electrical contractor and the network vendor all have support contracts with different clocks. Those clocks are the inputs to your MTTR, and if they are not written down, your resilience design is aspirational.
- **Who can change what?** A change to the cooling loop's setpoints, to the fabric's lossless configuration, or to the scheduler's allocation can each take the workload down. Change control on a single-tenant facility has no other tenant to catch the error.

[Data Center Guide](data_center_guide.md) section 9 covers monitoring (BMS, EPMS, DCIM), capacity management, maintenance regimes and change control as facility disciplines. This section covers only what changes when the load is a training job.

### 10.2 Spares

The spares list for an AI factory is longer than for a general-purpose hall, and the reason is the density: more failure points per square metre, and a higher cost per hour of downtime.

| Spare | Why it matters | The trap |
|---|---|---|
| GPU modules and nodes | A failed GPU can stall an entire collective | A spare is only a spare if it is the same part number and the same firmware revision |
| Optics and transceivers | The most common fabric failure, and the cheapest to hold | Mixing vendors or firmware levels produces intermittent link errors that look like a fabric bug |
| Power shelves | A 33 kW shelf failing derates the rack | Shelf firmware must match the rack's configured redundancy model |
| CDUs and pumps | The cooling loop is a single point of failure if it is N, not N+1 | A spare pump is useless without the commissioning procedure to swap it under load |
| Manifolds, hoses, dripless connectors | Leak risk at every joint | Connector standards differ between vendors; a mixed fleet needs a mixed spares kit |
| Cables, of the exact lengths and types | Bend radius and length are design constraints | A "same" cable of the wrong length violates the bend radius it was specified for |

The rule that makes the table operational: **spares are defined by part number and firmware, not by description.** A spare GPU that is the right model but the wrong firmware revision is a compatibility incident waiting for a maintenance window.

### 10.3 MTTR is the number that decides whether resilience is real

Mean time to repair is the number the resilience design is actually promising. A 2N electrical topology buys you the ability to survive a failure; the MTTR determines how long you can survive it, and whether the surviving path is degraded while you repair.

Three MTTR components, each owned by a different party:

1. **Detection** - how long until someone knows. This is a monitoring question, and it is usually the shortest.
2. **Response** - how long until a competent person is on site or on the line. This is the staffing question, and it is usually the longest.
3. **Repair** - how long until the failed unit is replaced and the system is back to full redundancy. This is the spares and procedure question.

A facility with excellent detection, no resident staff and no spares has a detection time of minutes and an MTTR of days. That is not a resilient facility; it is a well-instrumented one.

### 10.4 The 24/7 staffing question

A facility with no resident staff is a facility whose first responder is a vendor's ticket queue. That is a defensible choice for some workloads - a batch training cluster can tolerate a slow response if the checkpoint interval is short - and it is indefensible for a production inference service with a latency commitment.

The honest options are three:

- **Resident staff.** Highest cost, shortest response, and the only model that supports a production commitment without a contractual fiction.
- **A managed service from the colocation provider or a facilities operator.** The response time is contractual, and the contract is only as good as the provider's own staffing at your site.
- **Remote monitoring with a vendor SLA.** Cheapest, and the SLA's response time is a target, not a guarantee. It works for training, and it is a gamble for inference.

A bank should decide which of these it is buying before it signs the construction contract, because the staffing model changes the building - a resident model needs offices, a workshop, a store and a second person on site at all times, and those are square metres that do not appear in a rack-count plan.

### 10.5 When the factory serves exactly one workload

A multi-tenant facility averages demand across tenants. An idle tenant is offset by a busy one, and the provider's business case survives a tenant's bad quarter. A single-tenant factory has no averaging: **utilisation is the entire business case**, and an idle hall is a total loss, because the fixed cost does not move.

This has an operational consequence that is easy to miss. In a colo, capacity management is a commercial function - the provider sells what is free. In a single-tenant factory, capacity management is the architecture: the scheduler's allocation policy, the queueing discipline and the job admission rules are how the bank converts megawatts into useful work. If the scheduler is a default configuration, the facility's utilisation is a default outcome.

The measurement to run from day one is therefore not uptime. It is **utilisation at the rack and at the GPU**, because that number is the only lever the operations team has on the economics. Section 11.1 takes it up.

---

## 11. The Economics, Honestly

This section contains no capex per megawatt, no construction cost per square metre, no electricity tariff and no staff cost. Every one of those is a number that must come from a document specific to your site: a contractor's budget estimate, the utility's tariff schedule, the integrator's quote. A guide that prints them from memory is the reason bad business cases get approved. What this section can do is name the structure and the four variables that decide the answer.

### 11.1 Utilisation is the first variable

A purpose-built facility converts a variable cost into a fixed one. That is the whole financial story, and it means the business case is a utilisation forecast wearing an engineering costume.

The arithmetic that matters is not the cost per GPU-hour at design load. It is the cost per GPU-hour at the load you can actually sustain, which is a different number and usually a worse one. Three questions expose it:

- **What is the committed base load?** Not the peak, not the ambition - the demand the business owner will sign for over the depreciation period. Section 3.1 asks for it in writing for this reason.
- **What is the shape of the peak?** A facility sized for a peak that occurs for three weeks a year has 49 weeks of under-utilisation built into its capital cost.
- **What fills the trough?** In a multi-tenant facility, other tenants. In a single-tenant factory, nothing - unless the bank deliberately schedules research, fine-tuning and evaluation work into the capacity the production service is not using.

The third question is where most single-tenant business cases quietly fail. The answer is usually "we will find work for it", which is a plan to run the facility at whatever utilisation the organisation happens to produce.

### 11.2 The two clocks

The building and the silicon depreciate on different schedules, and the gap between them is the central financial risk of building.

- **The building clock** is measured in decades. A shell, a substation, a switchgear lineup and a chiller plant are long-lived assets, and their cost is recovered over a period that assumes the building is useful for most of it.
- **The accelerator clock** is measured in a handful of years. The silicon inside the building will be superseded, probably more than once, before the building's cost is recovered.

[WhiteFiber Guide](whitefiber_guide.md) documents a case where these clocks visibly diverge: a company capitalising development costs for a facility whose power delivery was on a multi-year schedule, against hardware that would be a generation old before the power arrived. The filings record $322.5 million of NC-1 development costs capitalised in the six months to 30 June 2026 (Form 10-Q, 12 August 2026) and a power agreement whose 99 MW milestone sat four years out. Whatever the merits of that specific project, the structure is the warning: the asset being built has a longer life than the asset it is being built for.

The architectural consequence is the one section 5.3 and section 8.2 both point at: **design the shell for the third GPU generation, not the first.** That is not a sustainability argument. It is the only way to make the building clock and the silicon clock reconcile - by making the building's useful life depend on its ability to host successive generations rather than on the one it was designed around.

### 11.3 A single-tenant facility is a different financial animal

A colocation provider's building is a diversified asset. It has multiple tenants, a re-letting market, and a residual value based on its ability to earn rent from someone else. A purpose-built facility for one tenant's one workload has none of those.

- **No tenant diversification.** One customer, one demand curve, one credit.
- **No re-letting.** The hall is designed around a rack density and a cooling class. A different tenant with a different density is a refit, not a lease.
- **A residual value that is mostly scrap plus power.** The land and the electrical connection retain value; the fit-out largely does not. The connection is the most valuable thing the bank owns at the end, which is a strange sentence and a true one.

This is the honest answer to "why not just build it, we own it then": ownership of a single-tenant facility is ownership of a concentrated bet, and the balance sheet treats it as such whether or not the business case does.

### 11.4 The cost stack, as line items to be quoted

The business case needs these lines, each with a named source document. The guide provides the structure; the numbers come from the market.

| Line item | What it covers | Where the number comes from |
|---|---|---|
| Land or lease | Site acquisition or ground lease | Valuation report, landlord's term sheet |
| Shell and structure | Building, slab, envelope, structural works | Contractor's budget estimate |
| Electrical infrastructure | Substation, transformers, switchgear, UPS, gensets, fuel storage | Electrical contractor's design and quote; utility's connection offer |
| Cooling plant | CDUs, chillers or dry coolers, pumps, pipework, heat rejection | Mechanical contractor's quote; CDU vendor datasheets |
| Racks and power distribution | Racks, bus bars, power shelves, PDUs | Integrator's quote, per the rack datasheet |
| IT equipment | GPUs, CPUs, memory, local storage | Vendor quote with delivery dates |
| Network fabric | Switches, optics, cabling, trays | Fabric vendor's reference architecture and quote |
| Storage | Parallel file system or object store, capacity and throughput | Storage vendor quote, sized from section 7 |
| Commissioning | FAT and SAT witnesses, load banks, independent engineer | Commissioning agent's fee proposal |
| Operations | Staff, 24/7 cover, vendor support contracts, spares | Run-model design (section 10) and vendor support quotes |
| Energy | Consumption at the PUE you will actually achieve | Utility tariff schedule; your own metered PUE after a year |
| Contingency | Design development, scope growth, schedule slip | The contractor's and the bank's own risk allowance |

Two rules for using the table. First, the energy line is the one that is wrong most often, because it is computed from a design PUE rather than a measured one; after the first year, replace the design figure with the metered figure and re-run the case. Second, the contingency line is not padding - it is the line that absorbs the schedule risk section 4 describes, and a business case without it is a business case that assumes the utility delivers on the first date.

---

## 12. The Regulated and Banking Angle

A bank building one of these changes the facility's governance in four ways, and none of them is captured by the engineering design.

### 12.1 Resilience moves into your own recovery scope

A rented multi-region inference service has resilience you can describe in a contract and test by failing over. A single-site factory is a single point of failure for a production inference workload, and it is a single point of failure that the bank now owns end to end.

Three consequences:

- **The facility enters the recovery plan and the exit-test scope.** Whatever the bank's recovery objectives are for the workload, the factory's electrical topology, its fuel autonomy and its MTTR are now inputs to them. A 2N electrical design with a four-day MTTR does not meet a same-day recovery objective, however good the topology.
- **The recovery plan needs a second site or an honest statement that there is not one.** A factory in one hall has no failover. If the workload is production, the recovery plan is either a rented overflow capacity arrangement or a documented acceptance that the service stops.
- **Exit testing gets harder, not easier.** Testing a failover between a bank's own hall and a rental route is a test of two things that were designed separately, and it needs to be run before the workload goes live, not after.

The architectural question that follows: **should a production inference workload live in a single-site facility at all?** For a non-critical internal assistant, yes. For a customer-facing decision engine, the honest answer is usually that the factory hosts the training and the batch work, and the latency-critical serving stays on a multi-region platform. That is a decision about the workload, not about the building, and section 3.1 is where it should be made.

### 12.2 The third-party dependency inverts

In a colocation arrangement, the bank is one tenant among many and the provider has other customers to lose. That is a real forcing function: the provider's operational discipline is maintained partly because a failure costs it several customers, not one.

In its own factory, the bank is the only customer of its own facility. There is no external party whose revenue depends on the plant staying up, and no landlord whose other tenants would notice a cooling failure. The operational discipline has to be generated internally, by governance rather than by commercial pressure.

That is a genuine risk, and it is the one that most often distinguishes a well-run facility from a badly-run one with identical equipment. The countermeasures are unglamorous: an internal service level with the same rigour as an external one, an independent audit of the facility's operations, and a change-control board with the authority to say no to the workload team.

### 12.3 Physical security: the concentration problem

The layered model is the repository's, and it does not change ([Data Center Guide](data_center_guide.md) section 8). What changes is what is inside the layers.

A purpose-built facility for one workload concentrates the crown jewels in one hall: the model weights, the training data, the fine-tuning corpus, and the training run itself. In a colo, those are distributed across cages and providers; in a factory, they are behind one door with one access list.

Three implications:

- **The insider model matters more, not less.** A single hall with a small operations team means the number of people with physical access is small, and each of them has access to everything. Segregation of duties has to be designed into the access model, because the building will not do it for you.
- **The escort model has to be enforced, not assumed.** Contractors, vendor engineers and auditors pass through the same door. An escort policy that is waived for a familiar vendor is a policy that does not exist.
- **The weights are the asset, and they are portable.** A GPU can be replaced; a fine-tuned model exfiltrated on a removable drive is a permanent loss. Physical access control and data-at-rest protection are the same control in this building.

### 12.4 Concentration risk in a different shape

The fourth change is strategic rather than operational. A purpose-built facility for one workload is a bet that the workload persists. A bank that changes its model strategy - moving from self-hosted fine-tuning to vendor-hosted models, or from a large model to a small one, or from training to pure inference - has a building it cannot easily re-let.

Section 11.3 gives the financial form of this. The governance form is a question the board should ask before approval: **what is the plan if the workload this facility was built for is no longer the workload we run?** Acceptable answers include "the hall is reconfigurable for a different density" and "the power connection is the asset and we can lease the hall". Unacceptable answers include "we will not change our model strategy", because that is a statement about a market, not about a plan.

### 12.5 What goes in the risk register

Five entries, each with an owner, that this guide would expect to see in any bank's approval pack:

| Risk | Why it is a bank risk, not a project risk | Owner |
|---|---|---|
| Power not delivered by the required date | The workload's go-live is a business commitment, and the facility is the dependency | Infrastructure lead, with the utility relationship |
| Single-site failure of a production workload | No failover exists inside one hall; recovery objectives are not met by topology alone | Head of resilience |
| Concentration of weights and training data in one hall | Physical access becomes a data-protection control | CISO |
| Model-strategy change orphans the facility | Capital committed against a demand that may move | CFO and the business sponsor |
| Operational discipline decays without an external forcing function | No other tenant and no landlord to provide the pressure | COO or the facilities governance function |

No penalty figures, no regulatory clause numbers and no claims about a specific regulator's rules are asserted in this section. The Singapore regulatory facts used in section 4.2 are the ones the repository has already verified; anything beyond them would need its own sourcing.

---

## 13. The Failure Modes

Six ways an AI factory programme fails. Each is written as symptom, cause and guardrail, because the guardrail is the only part that changes a decision.

### 13.1 Power contracted but not delivered

**Symptom:** the building is complete, the racks are installed, and the energisation date slips by quarters. The facility earns nothing while the carrying costs continue.

**Cause:** an agreement whose power milestone is an effort standard rather than a delivery obligation, and a business case that treated the milestone date as a commitment. [WhiteFiber Guide](whitefiber_guide.md) records the documented case: a Capacity Agreement dated 16 May 2025 under which the utility would use "commercially reasonable efforts" to reach 24 MW (gross) by 1 September 2025, 40 MW (gross) by 1 April 2026 and 99 MW within four years, with no 99 MW Electric Service Agreement in place at 30 June 2026 - while the company paid a minimum monthly bill and carried a $3m surety bond.

**Guardrail:** contract for a date certain with a remedy, not an effort. Stage the capital so that the IT order follows the energisation milestone rather than the other way round, and put the schedule risk in the contingency line (section 11.4) rather than in an optimistic assumption.

### 13.2 A building sized for a GPU generation that was superseded during construction

**Symptom:** the facility is commissioned for a rack density that is one generation behind the equipment it is asked to host. The racks fit; the power and cooling do not.

**Cause:** a two-to-four-year build programme against an accelerator cadence that moves faster. The design freezes at the concept stage, the silicon does not.

**Guardrail:** design the shell, the electrical service and the cooling distribution for the density of the generation after next, and state that assumption in the design basis. The HPE figure of 132 kW per rack (November 2026) is not a ceiling - it is the current flagship, and section 5.1's point is that the direction of travel has been upward for a decade. Reserve floor space, electrical capacity and pipe routes for a hall you have not specified yet.

### 13.3 Cooling designed for a density never reached - or reached and then exceeded

**Symptom, first form:** an expensive liquid cooling plant serving racks that run at a fraction of design density, because the workload was smaller than planned. **Symptom, second form:** racks that hit the design density and then are run above it, with the CDU at its limit and the delta-T climbing.

**Cause:** in the first form, a sizing chain that used a peak instead of a base load. In the second, a design with no headroom and no interlock - density creeps up as teams add accelerators to existing racks.

**Guardrail:** design for a density band, not a single number, and instrument the delta-T and the flow at every rack so that the plant's margin is visible before it is exhausted. The flow arithmetic in section 3.6 is the tool: at a fixed load, a rising delta-T means falling flow, and falling flow is the first symptom of a cooling plant at its limit.

### 13.4 Storage that cannot feed the training job

**Symptom:** training runs take longer than planned, GPU utilisation is low, and nobody can point to a failed component. The GPUs are simply waiting.

**Cause:** storage sized on capacity and vendor IOPS rather than on the read and write throughput the job needs. Section 7.2's numbers are the exposure: 4 GB/s per GPU for read on large image datasets, with write at "at least 1/2 of the read performance", and checkpoints measured in terabytes written synchronously.

**Guardrail:** size storage from step 9 of the sizing chain, and put the measured feed rate into the acceptance test. The test is a real job at real scale with GPU utilisation measured alongside bytes per second - not a synthetic benchmark at a queue depth the workload will never produce.

### 13.5 An acceptance test that proves racks, not throughput

**Symptom:** the facility passes acceptance, and the workload underperforms within weeks. The vendor's numbers and the bank's numbers disagree, and there is no agreed test to settle it.

**Cause:** acceptance criteria written as counts and states - racks energised, GPUs detected, links up - rather than as the workload's performance at the wall. The commercial consequence is that the argument happens after handover, when three parties have three sets of numbers and no shared test.

**Guardrail:** section 9.4's five named parameters in the contract: model, sequence length, concurrency, latency target, and the power measurement point. Tokens per second per megawatt, measured at the wall under MLPerf discipline, with the IT-MW and facility-MW figures both reported. A rack count is a milestone, not an acceptance criterion.

### 13.6 A factory whose single customer changes its model

**Symptom:** the workload the facility was built for is replaced by a different one - a smaller model, a vendor-hosted model, a different modality - and the hall's density, cooling class and even its location are wrong for the successor.

**Cause:** a facility designed around one workload's specifics rather than around a class of workloads. This is the concentration risk of section 12.4 expressed as an engineering failure rather than a financial one.

**Guardrail:** design for reconfiguration and write down what reconfiguration means: which racks can be re-piped, which cooling class the plant can serve, what density the floor can take, whether the hall can be split. A factory that cannot answer those questions is a factory with a single-use design, and single-use designs are only safe when the workload is genuinely permanent.

### 13.7 The pattern across all six

Five of the six failures are procurement or contract failures wearing engineering clothes. The power was never committed, the design was frozen too early, the density band was never specified, the storage throughput was never tested, the acceptance criteria were never written as performance. Only the third is primarily a technical failure, and even that one is a sizing failure.

Which is the practical argument for the whole guide: the engineering is well understood by the people you hire to do it. What fails is the document set - the workload specification, the capacity offer, the acceptance criteria, the density band - and that document set is the architect's deliverable, not the contractor's.

---

## 14. The Cymbal Bank Worked Example

**This example is fictional and illustrative.** Cymbal Bank is a persona, not a client, and every workload number below is invented to make the arithmetic visible. Every technical constant used - the 132 kW rack, the PUE basis, the power-arithmetic, the Singapore regulatory dates - is sourced in section 15. Cymbal Bank is the only institution in this example.

### 14.1 The scenario

Cymbal Bank (Singapore) runs an internal GenAI estate: a customer-service assistant, a document-processing pipeline for trade finance, a developer copilot, and a risk-reporting summariser. The bank fine-tunes its own domain models quarterly and does evaluation runs continuously. It is considering whether to build a factory for its own estate.

The business sponsor's ask is not "build a factory". It is: "give us sovereign capacity for the fine-tuning and the inference, on our own floor, because the model weights and the training corpus cannot leave the bank's control". That is a constraint, and section 2 says a constraint is what justifies a build. The job of the sizing chain is to find out whether the constraint is affordable in megawatts.

### 14.2 Steps 1 to 3: workload, throughput, GPUs

| Parameter | Illustrative value | Note |
|---|---|---|
| Inference demand, year 1 | 300 million output tokens/day | Invented |
| Inference demand, year 3 | 600 million output tokens/day | Invented; 2x growth |
| Peak hour as share of daily volume | 15% | Invented |
| Sustained output tokens/s per GPU at the p95 target | 1,200 | Invented; latency-bound serving, not a saturation benchmark |
| Headroom for N+1 and seasonality | 40% | Invented |
| Fine-tuning campaigns per year | 8 | Invented |
| GPU-hours per campaign | 55,000 | Invented |
| Experimentation and evaluation multiplier | 2x | Invented |
| Practical utilisation of dedicated capacity | 65% | Invented |

The arithmetic, so a reader can check it:

- **Average load:** 600,000,000 tokens/day / 86,400 seconds = 6,944 output tokens/s.
- **Peak load:** 15% of the daily volume in one hour = 90,000,000 tokens / 3,600 seconds = 25,000 output tokens/s.
- **Inference GPUs:** 25,000 / 1,200 = 20.8 GPUs, plus 40% headroom = 29.2, so 30 GPUs.
- **Fine-tuning GPUs:** 8 campaigns x 55,000 GPU-hours x 2 = 880,000 GPU-hours/year. Divided by (8,760 hours x 0.65 utilisation) = 154.5 GPUs of continuous dedicated capacity.
- **Total:** 30 + 154.5 = 184.5, so 185 GPUs.

That total is the first finding, and it is not the one the sponsor expected. **The fine-tuning, not the inference, drives the facility.** Inference at bank volumes is a small, bursty load; the quarterly fine-tuning programme, multiplied by experimentation, is what fills a hall. That single observation reshapes the decision at section 14.6.

### 14.3 Steps 4 and 5: racks and megawatts

185 GPUs at 72 GPUs per rack is 2.6 racks, so the plan is 3 racks (216 GPUs, 31 spare). Using the HPE datasheet's 132 kW per rack (PSN1014890104NLEN, November 2026):

- **IT load:** 3 x 132 kW = 396 kW = 0.396 MW.
- **Facility load at the PUE the bank would commit to:** 396 kW x 1.3 = 514.8 kW, so approx. 515 kW. The 1.3 is not a guess at a good number; it is the design target the bank must commit to if it wants to build in Singapore at all (section 4.2), and it is consistent with the repository's note that new builds run around 1.3 ([Data Center Guide](data_center_guide.md) section 10.1).
- **The ask to the utility:** 600 kW, which covers the 515 kW requirement plus a margin for the fourth rack.

Three racks. Half a megawatt of IT, two-thirds of a megawatt at the meter, and a request for 600 kW. That is a small facility by industry standards, and it is the honest size of a single bank's own GenAI estate. It is also small enough that the economics of section 11 must be examined carefully, because a purpose-built shell for 0.4 MW is a great deal of building for a modest load.

### 14.4 The schedule test

The utility's response, illustrative and typical in shape: **400 kW firm by the date the business needs it, with 600 kW only after a substation upgrade in 2029.** The offer is a capacity figure without a delivery obligation for the balance - the "commercially reasonable efforts" pattern of section 4.3.

Apply section 4.1's rule. The GPU count is the variable:

- **IT power available:** 400 kW / 1.3 = 307.7 kW.
- **Racks that fit:** 307.7 / 132 = 2.33, so 2 racks (264 kW), leaving 43.7 kW of facility-equivalent headroom.
- **GPUs that fit:** 2 x 72 = 144, against the 216 planned. **72 GPUs come out of the plan**, and with them part of the fine-tuning programme.

This is the guide's first consequence happening to a named bank: the arithmetic said 185 GPUs and three racks, the power said 144 GPUs and two, and nothing about the workload changed. Note what did not get adjusted - the PUE assumption, the 132 kW rack figure, the cooling design. Adjusting any of those would have hidden the shortfall rather than resolved it.

### 14.5 The Singapore allocation gate

A 400 kW requirement is far below the scale at which Singapore's allocation rounds operate: DC-CFA2 (December 2025) offered "at least 200 MW" for AI-ready, low-carbon data centres at PUE below 1.3 and at least 50 per cent green power ([Singapore Data Centres Guide](singapore_data_centres_guide.md) section 5.3). Cymbal Bank cannot apply for a tranche.

That is not a dead end; it is a clarification. The bank's realistic route to allocated, compliant capacity is to become a tenant inside a facility that holds an award, where the PUE and green-power conditions are already satisfied by the provider and the bank's requirement is a cage, not a campus. The allocation regime, in other words, pushes a bank of this size toward colocation for reasons that have nothing to do with cost.

### 14.6 The decision that falls out

The staged decision, and it is the one this guide would defend:

**Stage 1, now: rent for the training need.** The fine-tuning programme is bursty, generation-sensitive and the part of the load that drives the facility's size. Renting it removes 154.5 GPUs of dedicated capacity from the requirement and takes the accelerator-turnover risk off the bank's balance sheet (section 11.2). [GPU Cloud Providers Comparison Guide](gpu_cloud_providers_comparison_guide.md) section 9 has the arithmetic; the point here is that the bursty load is the last load that should justify concrete.

**Stage 2, 12 to 24 months: colocate the inference base load.** 30 GPUs, two racks of headroom, inside an awarded Singapore facility at PUE below 1.3 with at least 50 per cent green power. This satisfies the sovereignty constraint through a dedicated, audited cage rather than through ownership, and it satisfies the resilience constraint better than a single-site build would (section 12.1), because the provider has other customers and its own forcing function.

**Stage 3, conditional: build only if the threshold is cleared.** The stated threshold: measured GPU utilisation above 70 per cent across the colocated footprint for four consecutive quarters, **and** a power contract for a third rack with a date certain rather than an effort standard. Both conditions, not either. If utilisation is at 70 per cent and the power has a date, the bank has earned the right to build and has the evidence a board needs. If either fails, the colocated footprint is the right answer and the build case was a hypothesis.

### 14.7 What the example teaches

Three things, in order of importance.

**The load that drives the facility is rarely the load the sponsor is thinking about.** The sponsor asked for sovereign inference; the fine-tuning programme was 84 per cent of the GPU requirement. A sizing chain that starts from the wrong workload produces a facility of the wrong size, and no amount of engineering skill recovers it.

**The power answer, not the workload answer, sets the size.** The plan was 185 GPUs and three racks; the deliverable power supported 144 GPUs and two. The difference was settled by a utility's capacity offer, and the only correct response was to move the GPU count.

**The acceptance test has to be the workload, because the building will pass every other test.** Cymbal Bank's two racks would pass a rack-count test whether the fabric is oversubscribed 4:1 or non-blocking, and whether the storage feeds the job or starves it. The only test that distinguishes the two outcomes is the one in section 9.4: a named model, a named sequence length, a named concurrency, a named latency target, and tokens per second per megawatt measured at the wall.

---

## 15. The Claims Audit and What Could Not Be Verified

### 15.1 The convention

Every material claim in this guide is graded, in the repository's style, on a three-mark scale:

- ✅ verified at a primary source, or verified verbatim as a quotation of that source.
- ⚠ flagged: unverified, promotional, vendor-sourced, or a claim whose truth this guide cannot establish even though its wording is accurately reported.
- ❌ rejected or contradicted, and not used.

The distinction that matters most: a vendor's performance claim can be **accurately quoted** and still be ⚠, because quotation accuracy is not truth. The clearest case in this guide is the 40 per cent figure in row 7.

### 15.2 The claims table

| # | Claim | Kind | Source and date | Grade | Note |
|---|---|---|---|---|---|
| 1 | "An AI factory is a specialized computing infrastructure designed to create value from data..." | VENDOR DEFINITION | NVIDIA Glossary, "What Is an AI Factory?"; page undated, retrieved 11 November 2026 | ✅ | The page carries no publication date; the date is retrieval. Not a standard |
| 2 | No standards body defines "AI factory" | DOCUMENTED ABSENCE | Checked against Uptime Tier Standard, ANSI/TIA-942-C, ASHRAE guidelines | ✅ | Absence of a definition in those three; not proof that no body is drafting one (15.3e) |
| 3 | "AI factories unify five critical layers - energy, chips, infrastructure, models, and applications" | VENDOR CLAIM | NVIDIA AI Factories solutions page, retrieved 11 November 2026 | ⚠ | Accurately quoted; a vendor's framing, not a taxonomy anyone else has adopted |
| 4 | "Unlike traditional data centers, AI factories are purpose-built to manufacture intelligence at scale" | VENDOR CLAIM | Same page, retrieved 11 November 2026 | ⚠ | Accurately quoted; definitional claim by the term's originator |
| 5 | "There's a new Industrial Revolution happening in these [server] rooms: I call them AI factories" | PRESS REPORT | TechCrunch, 19 March 2024, reporting the GTC keynote of 18 March 2024 | ⚠ | Press rendering of a keynote, not an NVIDIA transcript. Earlier usage not traced (15.3b) |
| 6 | GTC 2024 keynote date | DOCUMENTED FACT | NVIDIA Blackwell launch press release, 18 March 2024 | ✅ | Used to date the provenance of the phrase |
| 7 | "Combining 45-degrees-Celsius liquid cooling with in-rack technologies ... lets operators run up to 40% more GPUs" | VENDOR CLAIM | NVIDIA Newsroom, DSX release, 31 May 2026 | ⚠ | Vendor claim. Not adopted anywhere in this guide as a design input |
| 8 | DSX MaxLPS maximises "token performance per megawatt within a fixed power budget" | VENDOR CLAIM | Same release, 31 May 2026 | ⚠ | Promotional framing; the metric itself is the right one (section 9.4) |
| 9 | DSX Reference Design covers "power, cooling and controls, as well as civil, structural and architectural design" | VENDOR CLAIM | Same release, 31 May 2026 | ✅ | Quoted as a product-scope description |
| 10 | "Each rack consumes 132 kW of power, with 115 kW liquid cooled and 17 kW air cooled" | DOCUMENTED FACT | HPE datasheet "NVIDIA GB200 NVL72 by HPE", PSN1014890104NLEN, November 2026 | ✅ | Vendor datasheet, but a specification document rather than marketing |
| 11 | 48U rack; 72 B200 GPUs and 36 Grace CPUs; up to 13.5 TB HBM3E at 576 TB/s; 6-8 1U 33 kW power shelves; 50 V DC bus bar | DOCUMENTED FACT | Same datasheet, November 2026 | ✅ | As specified |
| 12 | 1.83 kW per GPU all-in | DERIVED | 132 kW / 72 GPUs, this guide | ✅ | Includes CPUs, switches, fans and conversion losses - which is why a per-GPU TDP is not a facility number |
| 13 | 72-GPU NVLink domain, 130 TB/s GPU communications, 1.8 TB/s GPU-to-GPU | DOCUMENTED FACT | NVIDIA GB200 NVL72 product page, retrieved 11 November 2026 | ✅ | Product page specification |
| 14 | Tier rating constrained by the weakest subsystem; no partial tiers; independent of the IT systems | STANDARD | Uptime Institute, Tier Standard: Topology, updates effective 1 October 2018 | ✅ | A topology standard, not an availability promise |
| 15 | Fuel storage requirements for on-site power production | STANDARD | Same Standard, requirements effective 1 May 2010 | ✅ | Cited for the behind-the-meter section |
| 16 | Minimum 800mm-wide cabinets in MDA/HDA/IDA; at least two optical fibres for horizontal and backbone cabling | STANDARD | ANSI/TIA-942-C (2024), via TIA white paper DCWP041724, May 2024 | ✅ | A requirements standard; certification to it is not a certification of availability or workload performance |
| 17 | In 2022, 25% of enterprise DCs reported racks above 20 kW; some hyperscale racks reached 80 kW+ | DOCUMENTED (SECONDARY) | TIA white paper DCWP041724, May 2024 | ⚠ | Industry survey reported by TIA; not independently re-verified here |
| 18 | Air cooling around 50 kW/rack; single-phase direct-to-chip 25-90 kW; immersion 50-100 kW | VENDOR-SOURCED | TIA white paper, Figure 4, sourced to ASUS, May 2024 | ⚠ | Provenance is a vendor's figure carried in a standards body's paper. Used as a range, not as a sizing input |
| 19 | Recommended server inlet 18-27 C for common air classes | STANDARD | ASHRAE Thermal Guidelines, 5th edition, 2021 | ✅ | An envelope, not an approval |
| 20 | 5th edition added class H1, 18-22 C for high-density systems | STANDARD (SECONDARY) | TIA white paper DCWP041724, May 2024, reporting ASHRAE | ✅ | Reported by TIA rather than read from ASHRAE directly |
| 21 | Water classes: W17, W27, W32, W40, W45, W+ | STANDARD | ASHRAE TC 9.9 white paper, "Emergence and Expansion of Liquid Cooling in Mainstream Data Centers", 2021 | ✅ | Renaming of W1-W5 plus a new W40 |
| 22 | W45 "may be comparable to a conventional air-cooled system" for a dense 1U half-width node | STANDARD | Same white paper, 2021 | ✅ | The honest complication in section 5.5; not a monotonic improvement |
| 23 | Power measured as average AC power at the wall for the benchmark duration | STANDARD / METHOD | MLCommons, MLPerf Inference: Datacenter, v6.1, 2026 | ✅ | The measurement discipline adopted for section 9.4 |
| 24 | MLCommons does not measure or validate TDP or PSU ratings | STANDARD / METHOD | Same, v6.1, 2026 | ✅ | The authority for refusing a TDP-based acceptance test |
| 25 | Storage read/write tiers; 4 GB/s per GPU for large image datasets; write at least 1/2 of read | DOCUMENTED (VENDOR RA) | NVIDIA DGX SuperPOD reference architecture, Storage Architecture page, retrieved 11 November 2026 | ✅ | Vendor-published guidance, quoted as guidance |
| 26 | Checkpoints are terabytes and written synchronously, blocking forward progress | DOCUMENTED (VENDOR RA) | Same page, retrieved 11 November 2026 | ✅ | The sentence that makes storage a latency problem |
| 27 | GPUDirect Storage "over 27 GBps" against a per-node network maximum of 80 GBps | DOCUMENTED (VENDOR RA) | Same page, retrieved 11 November 2026 | ✅ | As stated |
| 28 | 100 kW at delta-T 10 K needs 2.40 L/s (approx. 8.6 m3/h) | DERIVED | Physics: flow = P / (rho x cp x delta-T), rho approx. 0.997 kg/L, cp 4.186 kJ/kg-K | ✅ | Recomputed for this guide; formula shown in section 3.6 |
| 29 | 1 MW at delta-T 10 K needs 24.0 L/s (approx. 86 m3/h) | DERIVED | Same formula | ✅ | As above |
| 30 | 132 kW at delta-T 15 K needs 2.11 L/s (approx. 7.6 m3/h) | DERIVED | Same formula | ✅ | As above |
| 31 | 1 MW is approx. 3.41 million BTU/h | DERIVED | 1 kW = 3,412 BTU/h | ✅ | Arithmetic |
| 32 | PUE defined 2007; ISO/IEC 30134-2 (2016); Uptime 2024 average 1.56, new builds around 1.3 | DOCUMENTED (REPO) | [Data Center Guide](data_center_guide.md) section 10.1 | ✅ | Already verified in the repository; cross-referenced, not re-derived |
| 33 | Uptime 2024: modal rack 4-6 kW, average 8 kW, most facilities no rack above 30 kW | DOCUMENTED (REPO) | [Data Center Guide](data_center_guide.md) section 7.2 | ✅ | Cross-referenced |
| 34 | Singapore moratorium May 2019; January 2022 resumption at PUE 1.3 or better; DC-CFA2 December 2025 at least 200 MW, PUE below 1.3, at least 50% green power | DOCUMENTED (REPO) | [Singapore Data Centres Guide](singapore_data_centres_guide.md) sections 2 and 5 | ✅ | Verified regulatory facts; used in sections 1.4 and 4.2 |
| 35 | Duke Capacity Agreement 16 May 2025; commercially reasonable efforts; 24/40/99 MW milestones; contingent price; $8,754 minimum monthly bill; $3m surety bond; $322.5m capitalised | DOCUMENTED (REPO) | [WhiteFiber Guide](whitefiber_guide.md), from the 10-K of 26 March 2026 and 10-Q of 12 August 2026 | ✅ | No cost per MW disclosed; none computed |
| 36 | No capex per MW, cost per square metre, GPU price, land price, tariff or staff cost is printed | SELF-AUDIT | This guide, sections 2, 11 and 15 | ✅ | Deliberate omission, not an oversight |
| 37 | Grid interconnection queue durations | NOT VERIFIED | Search returned empty results; two primary pages failed to load | ❌ | Not printed (15.3c) |
| 38 | Cymbal Bank workload and cost figures in section 14 | ILLUSTRATIVE | Invented for the example | ⚠ | Every workload number is illustrative; every technical constant is sourced above |

### 15.3 What Could Not Be Verified

**(a) No publication date on NVIDIA's glossary definition.** The page carries no date, so the date given throughout is the retrieval date, 11 November 2026. A definition quoted from an undated page cannot be dated, and pretending otherwise would be the kind of false precision this guide refuses.

**(b) Earlier NVIDIA usage of the term than the GTC 2024 keynote was not traced.** The phrase may have appeared in earlier NVIDIA material, partner decks or conference talks. This guide dates the provenance to 18 March 2024 on the strength of the keynote coverage and states that earlier usage is possible and untraced. Claims that NVIDIA "invented" the term in 2024 should be read as "the earliest usage this pass could verify".

**(c) Grid interconnection queue statistics were not retrievable.** Median time from request to energisation, by market, is the number a schedule risk assessment most wants. Web search returned empty results in this pass and two primary sources failed to load. The guide therefore argues from two documented cases (Singapore's allocation regime and the Duke capacity agreement) and prints no queue average. For a real business case, obtain the system operator's published queue data with a date.

**(d) Capex per megawatt is not disclosed by any source consulted in a form this guide would print.** The WhiteFiber filings disclose $322.5m of capitalised NC-1 development costs without disclosing the MW those costs relate to on a comparable basis, and dividing one by the other would manufacture a number the filings do not support. The guide prints no capex per MW, no construction cost per square metre, and no GPU price. The documents to obtain instead are named in section 11.4.

**(e) Whether any standards body is currently drafting an "AI factory" definition was not established.** This is an absence of a finding, not proof of absence. The three bodies consulted (Uptime, TIA, ASHRAE) publish facility, infrastructure and thermal standards and none of them defines the term. A drafting effort elsewhere, in a working group or a national body, could exist and would not have appeared in this pass.

### 15.4 What the audit is for

The table is not decoration. It is the mechanism by which a reader can find the two rows they disagree with and go to the source, and by which a reviewer can see that the guide's strongest claims are the sourced ones and its weakest claims are labelled. A build guide without one is asking to be believed. This one is asking to be checked.

---

## 16. Glossary and Closing Summary

### 16.1 Glossary

**AI factory.** In this guide, the physical sense: a purpose-built facility for AI training and inference, designed as a whole rather than assembled from a general-purpose data centre. NVIDIA's definition - "a specialized computing infrastructure designed to create value from data by managing the entire AI life cycle" whose "primary product is intelligence, measured by token throughput" - is a vendor definition, not a standard. The platform sense (a governed internal GenAI platform) is documented elsewhere in this repository and is a different thing (section 1.1).

**PUE (Power Usage Effectiveness).** Total facility power divided by IT power. Defined by The Green Grid in 2007 and standardised as ISO/IEC 30134-2 (2016). A ratio, so it is dimensionless; 1.3 means 30 per cent overhead. It measures the building, not the workload, which is why section 9.4 pairs it with tokens per second.

**Rack density.** The power drawn by one rack, in kW. The unit that decides whether a hall can be air-cooled and whether an existing hall can be reused. The Uptime Institute 2024 survey puts the modal enterprise rack at 4-6 kW; a current flagship GPU rack is specified at 132 kW (HPE, November 2026).

**Direct-to-chip.** Liquid cooling in which a cold plate is attached to the processor and a loop carries heat to a CDU. Cools the components it touches; the rest of the rack is still air-cooled, which is why a 132 kW rack splits into 115 kW liquid and 17 kW air.

**InfiniBand.** A purpose-built lossless fabric with credit-based flow control and a mature management stack for large GPU clusters. The default in the GPU vendors' own reference architectures.

**RoCE (RDMA over Converged Ethernet).** Remote direct memory access over Ethernet. Attractive for standardisation on one network technology, but lossless behaviour depends on correct priority flow control and explicit congestion notification configuration - an operational discipline rather than a property of the protocol.

**FAT (Factory Acceptance Test).** The test at the vendor's works, before shipment, against a written procedure with a named witness and recorded results. The last point at which a defect is the vendor's logistics problem rather than the buyer's schedule problem.

**SAT (Site Acceptance Test).** The same procedure repeated on site, plus the interfaces that only exist on site: power quality at the rack, real flow and return temperatures, controls integration, alarms. Where boundary failures between vendors' scopes are found.

**Tokens per second per megawatt.** The acceptance metric of section 9.4. Defined twice, deliberately: per IT MW (the compute design's efficiency) and per facility MW (what the bank pays for). Meaningless without the model, sequence length, concurrency and latency target stated alongside it, and measured at the wall under MLCommons discipline rather than from a thermal design power.

**W class.** ASHRAE's liquid-cooling water classes, renamed in the 2021 TC 9.9 white paper: W17 (previously W1), W27 (W2), W32 (W3), W40 (new), W45 (W4) and W+ (W5). A higher number generally means warmer facility water, which is better for plant efficiency and worse for heat capture - the W45 caveat in section 5.5.

**Colocation.** Renting space, power and cooling in a facility operated by someone else, typically in a cage or a suite, with the tenant supplying its own IT. The middle route in section 2: the provider owns the building and the power relationship, the bank owns the workload.

### 16.2 The guide in one page

Six statements, each of which the preceding sections argued rather than assumed:

1. **Two senses, one phrase.** The repository documents the platform sense in nine files, with three anchors in the DBS and Standard Chartered guides. This guide is the physical sense, and the term belongs to a vendor because no standards body defines it (section 1).
2. **Decide before you design.** Renting is frequently correct for a single institution's own workload. Building is justified by constraints and a contractible power position, not by the word "factory" (section 2).
3. **The chain computes forward.** Nine steps take a demand forecast to a utility contract size, and each step produces a document the acceptance test will later depend on (section 3).
4. **The programme runs backward, and the GPU count is the variable.** You can always buy fewer GPUs; you cannot always buy more megawatts. Singapore's allocation regime and the Duke capacity agreement are the two documented proofs (section 4).
5. **Air is out of range, and liquid is not a free upgrade.** A 132 kW rack is 22 to 33 times the modal enterprise rack, and warmer water can cost cooling capability (section 5).
6. **The acceptance test is the workload.** Tokens per second per megawatt, at the wall, with the model, sequence length, concurrency and latency target named in the contract. A rack count proves the building (section 9).

### 16.3 What to do on Monday

The first four actions, in order, for an architect who has read this and has a sponsor waiting:

1. **Get the workload specification signed** (step 1). Training, inference or both; context length distribution; demand in tokens/day or GPU-hours per run. Without it, nothing downstream can be sized.
2. **Benchmark your own stack** (step 2). Your model, your quantisation, your serving framework, your sequence lengths. Not a vendor chart.
3. **Ask the utility or the site owner for a capacity offer in writing**, with an energisation date and a delivery obligation (section 4.4). Do this before the GPU quote, because it is the longer lead time and the harder answer.
4. **Write the acceptance test now**, while the design is still fluid: model, sequence length, concurrency, latency target, measurement point. The test written at commissioning time tests whatever the contractor finds easiest.

### 16.4 Closing

The industry's marketing has settled on "AI factory" because it captures something true: the facility is a production plant whose output is tokens, and a plant is designed around its product rather than around its building. What the marketing obscures is that the plant is a power and cooling project with a long lead time and a short tolerance for optimistic assumptions, and that the product is only proven when it is made.

Everything in this guide follows from taking that seriously: contract the power before you order the silicon, design the shell for a generation you have not bought yet, measure at the wall rather than at the datasheet, and refuse to call the facility finished until the workload it was built for actually runs at the performance it was built for.

An AI factory is not a data centre with GPUs in it; it is a power-and-cooling project whose only honest acceptance test is the workload it was built for.
