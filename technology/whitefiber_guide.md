# WhiteFiber - Two Clocks on One Balance Sheet: A Building, an Accelerator, and the Contract Between Them

> **Author:** Jack Liu Shurui - Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research - AI Infrastructure series (company deep-dive; read alongside the GPU-cloud and data-centre guides)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary sources:** **WhiteFiber, Inc., Form 10-K for the fiscal year ended 31 December 2025, filed 26 March 2026, SEC accession 0001213900-26-034341**; **Form 10-Q for the quarter ended 31 March 2026, filed 14 May 2026, accession 0001213900-26-056212**; **Form 10-Q for the quarter ended 30 June 2026, filed 12 August 2026, accession 0001213900-26-088026** (the most recent disclosed financials at the time of writing); SEC EDGAR submissions metadata for CIK 0002042022 (retrieved November 2026); whitefiber.com home page, whitefiber.com/cloud/compute and whitefiber.com/data-center/ai-hpc-colocation (marketing pages, retrieved November 2026).
> **Last Updated:** November 2026

---

**Table of Contents**

1. [Overview, Decoder and the Verified Identity](#1-overview-decoder-and-the-verified-identity)
2. [The Two Segments: Landlord and Lessor](#2-the-two-segments-landlord-and-lessor)
3. [The Colocation Business: Landlord Economics](#3-the-colocation-business-landlord-economics)
4. [The Cloud Business: Accelerators by the Month](#4-the-cloud-business-accelerators-by-the-month)
5. [The Two Clocks: Depreciation and Obsolescence](#5-the-two-clocks-depreciation-and-obsolescence)
6. [The Contracts: Term Against Asset Life](#6-the-contracts-term-against-asset-life)
7. [Customer Concentration: The Pause as Test](#7-customer-concentration-the-pause-as-test)
8. [Power and the Physical Constraint](#8-power-and-the-physical-constraint)
9. [Capital Structure and Capex Commitments](#9-capital-structure-and-capex-commitments)
10. [The Banking Angle: Underwriting the Contract](#10-the-banking-angle-underwriting-the-contract)
11. [Five Failure Modes for This Class of Company](#11-five-failure-modes-for-this-class-of-company)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [Competitive and Category Context](#13-competitive-and-category-context)
14. [The State of Practice](#14-the-state-of-practice)
15. [The Claims Audit](#15-the-claims-audit)
16. [Glossary and Closing Summary](#16-glossary-and-closing-summary)

---

## 1. Overview, Decoder and the Verified Identity

### 1.1 What This Guide Is

This is the company deep-dive on **WhiteFiber, Inc.** (Nasdaq: WYFI), an AI-infrastructure company that does two things on one balance sheet: it owns and leases data-centre buildings, and it rents out NVIDIA accelerators. It listed on Nasdaq on 8 August 2025 and files with the SEC as a domestic issuer with two reportable segments, **cloud services** and **colocation services**. It is small, newly public, and financially simple enough to read end to end, which makes it unusually good material for the question this series keeps returning to: what happens when a business sells time on an asset whose useful life is measured in years, using a contract whose term is also measured in years, and a building that outlives both.

The guide owns **the company**: identity, segments, contracts, capital structure, the depreciation question, and what a bank should and should not conclude from the disclosures. It does not own the facility physics or the buyer's cost model, and it does not re-derive either.

### 1.2 Decoder - The Vocabulary This Guide Uses

| Term | What it means here, and who is asserting it |
|---|---|
| **Colocation** | Renting physical space, power and cooling inside a data centre to a customer who installs its own hardware. The filing's own definition: "a data center facility that rents out rack space to third parties for their servers or other network equipment" (10-K, 26 March 2026). Landlord economics: the customer owns the depreciating asset, the landlord owns the building. |
| **GPU-as-a-service (GPUaaS)** | Renting access to accelerators rather than to floor space. The customer's workload runs on hardware the provider owns; the unit of sale is a GPU-hour, a GPU-month, or a reserved block of GPUs for a fixed term. WhiteFiber's cloud segment. |
| **Neocloud** | The filing's own term ("Neo-Cloud provider" - "a Cloud provider specializing in AI infrastructure as a service", 10-K, 26 March 2026). Structurally: a GPU-cloud operator that is not a hyperscaler, does not own a general-purpose cloud estate, and sells AI capacity directly. |
| **Reserved capacity** | A commitment to take a defined quantity of capacity for a defined period at a defined price, as distinct from on-demand hourly purchase. The commitment is what makes the provider's capital expenditure financeable. |
| **Take-or-pay** | A contract in which the customer pays for the committed quantity whether or not it consumes it. Rarely seen in pure form; the practical version is a minimum purchase commitment plus a termination fee. |
| **Useful life** | The period over which an asset is depreciated. An accounting estimate, not a physical fact - and, in this company's case, an estimate that changed. (Section 5.) |
| **Tier III** | A design-topology claim about concurrent maintainability: every capacity component and distribution path can be removed for planned maintenance without affecting operations (the filing's definition, 10-K, 26 March 2026). What it does and does not certify is owned by `data_center_guide.md` sections 4 and 9. |
| **Gross MW vs IT MW** | Gross load is total facility demand including cooling and losses; IT load is what reaches the customer's equipment. The filing defines both and states that "unless otherwise specified for a particular contract, all MW references are to gross capacity" (10-K, 26 March 2026). The gap between them is the efficiency of the building, and it is where capacity claims quietly inflate. |

### 1.3 The Verified Identity

| Item | Finding | Source and date |
|---|---|---|
| Legal name and jurisdiction | **WhiteFiber, Inc.**, a **Cayman Islands exempted company** | 10-K, 26 March 2026 (cover and "Certain Definitions") |
| Listing | Ordinary shares, par value **$0.01**, on **Nasdaq** under **WYFI** | 10-K, 26 March 2026 |
| SEC CIK | **0002042022** | EDGAR submissions metadata, retrieved November 2026 |
| Former name on record | EDGAR's submissions metadata records a former name **"White Fiber, Inc."**, current from **6 January 2025 to 11 July 2025** | EDGAR submissions metadata, retrieved November 2026 |
| Name history inside the filing | Incorporated as **Celer, Inc.** on 15 August 2024; renamed **WhiteFiber, Inc.** on **17 October 2024** | 10-K, 26 March 2026 |
| Headcount | **83 full-time employees** as of 31 December 2025 | 10-K, 26 March 2026 |
| Control | **Bit Digital, Inc. (BTBT)** held approximately **70.5%** of issued and outstanding ordinary shares as of 31 December 2025; the company is a **controlled company** under Nasdaq rules | 10-K, 26 March 2026 |

The two name records do not contradict each other and this guide does not force them to. The 10-K records the Celer, Inc. to WhiteFiber, Inc. change on 17 October 2024; EDGAR's metadata records the spaced rendering "White Fiber, Inc." as current for a six-month window in 2025. Both are corporate-registry-adjacent facts reported by their own systems. What is not established is why the spacing changed, and this guide does not speculate.

### 1.4 The Corporate History, as Verified

- **19 October 2023** - **WhiteFiber AI, Inc.** incorporated in Delaware under the name **Bit Digital AI, Inc.** (later renamed). Source: 10-K, 26 March 2026.
- **15 August 2024** - **WhiteFiber, Inc.** incorporated by Bit Digital as a Cayman Islands exempted company under the name **Celer, Inc.**, as a holding company for Bit Digital's HPC business. Renamed WhiteFiber, Inc. on **17 October 2024**. Source: 10-K, 26 March 2026.
- **11 October 2024** - **Enovum Data Centers Corp.** acquired, bringing the Montreal data-centre business including the leased **MTL-1** facility (4 MW gross, Tier-3, fully operational and fully leased at acquisition). Source: 10-K, 26 March 2026.
- **27 December 2024** - real estate and building acquired for the **MTL-2** build-to-suit expansion near Montreal, for CAD 33.5 million (approximately $23.3 million). Source: 10-K, 26 March 2026.
- **30 July 2025** - **Section 351 Contribution Agreement** signed with Bit Digital; **Transition Services Agreement** signed the same day. Source: 10-K, 26 March 2026.
- **6 August 2025** - the Contribution became effective on the declaration of effectiveness of the Form S-1 (File No. 333-288650). Bit Digital contributed 100% of the capital shares of WhiteFiber AI, Inc. and its subsidiaries in exchange for **27,043,749 ordinary shares**; WhiteFiber AI became a wholly-owned subsidiary of WhiteFiber, Inc. and Bit Digital became the direct shareholder. Source: 10-K, 26 March 2026.
- **8 August 2025** - IPO completed: **9,375,000 ordinary shares at $17.00**, gross proceeds **$159,375,000**, before underwriting discounts of $11,156,250 and expenses. On **2 September 2025** the underwriters exercised the over-allotment in full for a further **1,406,250 shares**, adding gross proceeds of **$23,906,250**. Source: 10-K, 26 March 2026.
- **31 December 2025** - Bit Digital holds approximately **70.5%**; the filing states that immediately after the IPO and full over-allotment it held approximately **71.5%** (one section of the same filing says 71%), so the stake has drifted down slightly. Source: 10-K, 26 March 2026.

One accounting consequence belongs in the same breath, because it governs every comparison in this guide: **for periods before 6 August 2025 the financial statements are presented on a combined basis**, reflecting common control by Bit Digital rather than an acquired group. The FY2024 comparatives are therefore not the prior-year results of an independent company. They are the carved-out HPC business of a parent, and the FY2025 income statement spans two different ownership regimes.

### 1.5 The Thesis

> **WhiteFiber is two businesses on one balance sheet - a building that depreciates over decades and an accelerator that depreciates in years - and the contract between them is the whole investment case.**

Three propositions follow, and the rest of this guide tests them rather than assumes them:

1. **Colocation is landlord economics on long leases.** A 30-year building, a 10 to 15-year mechanical plant, a 10-year customer contract, and a rent that escalates. The asset outlives the contract, and the contract outlives the debt.
2. **Cloud is a lessor of a fast-depreciating asset whose useful life is shorter than or comparable to the contract that pays for it.** Five-year book life, 12 to 24-month typical contract term (Section 6). The asset is still 60% undepreciated when the first customer's commitment ends.
3. **The risk sits in the gap between those clocks** - and, crucially, in which clock the financing is sized against.

### 1.6 The Boundary Against the Two Parent Guides

| Subject | Owner | This guide's relationship to it |
|---|---|---|
| Access models, cost decomposition, provider line-up, availability, tenancy, performance-per-dollar, rent-versus-own arithmetic | [gpu_cloud_providers_comparison_guide.md](gpu_cloud_providers_comparison_guide.md) (sections 1-9) | Cited, never re-derived. This guide assumes the reader knows what a GPU-hour costs to deliver and asks instead who owns the metal and for how long. |
| Facility anatomy, Uptime Institute tiers, electrical topology, cooling, density limits, certification, PUE and the nines table, colocation pricing mechanics | [data_center_guide.md](data_center_guide.md) (sections 2-12) | Cited, never re-derived. Section 3 of this guide interprets one company's Tier III claim; it does not restate the standard. |
| The company, its segments, contracts, capital structure, depreciation policy and the banking consequences | **This guide** | Owns it. |

---

## 2. The Two Segments: Landlord and Lessor

### 2.1 What Each Segment Sells

The two segments sell different things to different people, and the difference is not a matter of degree.

**Colocation** sells **space, power and cooling** in a building WhiteFiber controls. The customer installs its own hardware. WhiteFiber's capital is in the shell, the electrical spine and the cooling plant; its revenue is a rent per kilowatt of committed capacity, usually with annual escalators, on a contract the 10-Q describes as "generally one to five years for data center colocation customers" (Form 10-Q, 12 August 2026). The customer's asset depreciates; WhiteFiber's does not, or not quickly.

**Cloud** sells **access to GPUs** that WhiteFiber owns or leases. The customer runs its workload on WhiteFiber's metal. WhiteFiber's capital is in the accelerators, the networking and the orchestration software; its revenue is a rate per GPU per unit time, on a contract that is frequently shorter than the asset's book life.

### 2.2 The FY2025 Segment Economics, as Filed

All figures in dollars, from Note 15 of the FY2025 10-K (filed 26 March 2026):

| Line | Cloud services | Colocation services | Total |
|---|---|---|---|
| Revenue from external customers, FY2025 | 68,753,609 | 8,913,816 | 77,667,425 |
| Other revenue (equipment leasing), not in segment gross profit | - | - | 1,496,827 |
| **Total consolidated revenue, FY2025** | | | **79,164,252** |
| Electricity costs | 2,450,710 | 1,438,218 | 3,888,928 |
| Datacenter lease expense | 5,410,230 | 1,025,851 | 6,436,081 |
| GPU lease expense | 14,741,928 | - | 14,741,928 |
| Wage expense | - | 406,787 | 406,787 |
| Customer support contractor fees | 1,124,902 | - | 1,124,902 |
| Other segment items | 2,858,593 | 579,679 | 3,438,272 |
| **Segment gross profit, FY2025** | **42,167,246** | **5,463,281** | **47,630,527** |
| Revenue from external customers, FY2024 | 45,727,736 | 1,361,241 | 47,088,977 |
| Total consolidated revenue, FY2024 | | | 47,639,237 |
| Segment gross profit, FY2024 | 26,002,406 | 870,740 | 26,873,146 |

Three facts sit in that table and most readers walk past all three.

**First, cloud is 87% of revenue and 89% of gross profit.** Colocation, the segment with the durable asset and the long contract, is 11% of revenue. The thesis is not yet a description of the company's P&L; it is a description of where the company is going. The direction of travel is unmistakable - the colocation backlog is 12 times the cloud backlog (Section 6) - but the FY2025 income statement is still a GPU-rental business with a data-centre hobby attached.

**Second, depreciation is excluded from cost of revenue by policy, and stated separately.** The 10-K says so twice ("Cost of revenue excludes depreciation expenses, which are separately stated"; "excluding depreciation and amortization" in the segment policy). FY2025 depreciation and amortisation was **$23,440,884** against segment gross profit of $47,630,527. Charge the depreciation the segments exclude back against the margin they report and the picture changes materially: consolidated revenue of $79,164,252 less cost of revenue of $30,036,898 less $23,440,884 of depreciation leaves **$25,686,470**, or 32.4% of revenue, before a single dollar of the $52,507,246 general and administrative expense. The headline 60% segment margin is a pre-depreciation margin, and this is the single most important presentational fact in the filings.

**Third, the cloud segment is partly a lessee, not purely an owner.** $14,741,928 of GPU lease expense - 21.4% of cloud revenue - is an operating cost inside the cloud segment. Note 15 discloses the corresponding debt-like obligation only indirectly, through lease liabilities and the sale-leaseback history in the business section: 96 servers (768 GPUs) sold and leased back for three years in the first transaction, 128 servers (1,024 GPUs) in the second (10-K, 26 March 2026). So the "accelerator that depreciates in years" is, for a meaningful slice of the fleet, an accelerator WhiteFiber does not own and cannot depreciate at all.

### 2.3 Why the Economics Differ - and Where the Thesis Is Complicated

**The complication, stated plainly.** The naive version of the thesis says: colocation earns a modest return on a durable asset, cloud earns a high return on a perishable one, and the high return is compensation for the perishability. The filings do not support the second half of that sentence cleanly. The cloud segment's 61.3% gross margin is high partly **because the segment is asset-light at the margin**: it rents a fifth of its cost base (GPU leases) and books none of the depreciation on the rest. A true like-for-like comparison would charge cloud for the depreciation on its owned fleet and treat the GPU lease as the financing cost it is. On the disclosed numbers that comparison cannot be completed, because the company does not split depreciation by segment - it discloses $95.8 million of cloud service equipment net book value including construction in progress at 30 June 2026 against $556.3 million of colocation equipment (Form 10-Q, 12 August 2026), and no segment depreciation charge.

**Where the thesis holds without qualification.** Colocation's economics are landlord economics in the strict sense: the revenue is a rent on committed kilowatts, the cost is electricity (passed through in the Nscale contract, per the 10-K), the lease cost of the building, and wages. The segment carries $556.3 million of net book value and produced $8.9 million of FY2025 revenue. That is a capital-intensive, low-turn asset with a long tail - exactly what the thesis says it is. What the thesis gets right is not the margin comparison; it is the **clock mismatch**, and the clock mismatch is a balance-sheet fact, not a P&L fact.

> **Judgement:** the two segments are not two versions of the same business at different margins. They are two different asset classes sharing a treasury function. Reading the consolidated gross margin as though it described one business is the most common error available to a reader of these filings.

---

## 3. The Colocation Business: Landlord Economics

### 3.1 The Estate, Facility by Facility

| Facility | Location | Capacity | Status and contract | Source |
|---|---|---|---|---|
| **MTL-1** | Montreal, Quebec | 4 MW gross, Tier-3 | Leased building; fully operational and fully leased at acquisition (11 October 2024) | 10-K, 26 March 2026 |
| **MTL-2** | Pointe-Claire, Quebec | 5 MW gross, Tier-3 | Real estate bought 27 December 2024 for CAD 33.5m (~$23.3m); ~$23.6m to develop; **build deprioritised** - "we have prioritized other builds and preserved capital for more time sensitive projects" | 10-K, 26 March 2026 |
| **MTL-3** | Saint-Jerome, Quebec | 7 MW gross, Tier-3 | 202,000 sq ft on 7.7 acres; lease-to-own with CAD 24.2m fixed-price option exercised 14 January 2026, purchase closed 8 May 2026; operational November 2025; **Cerebras at 5 MW IT load**, billing CAD 1.4m (~$979k) monthly for a **five-year** contract from 1 November 2025 | 10-K, 26 March 2026; 10-Q, 12 August 2026 |
| **NC-1** | Madison, North Carolina | 24 MW contracted initially, targeting far more | Building and land bought 20 May 2025 for $45m cash; **Nscale** services agreement signed November 2025: approximately **$865 million of total contracted revenue over a 10-year term**, inclusive of escalators and non-recurring installation charges, with electricity and certain operating costs as pass-throughs; billing for the 40 MW phase expected to commence June 2026 | 10-K, 26 March 2026 |
| **Blonduos Campus** | Iceland | 6 MW gross, 45 kW rack density, 5.5 MW IT contracted | **Leased**, not owned; hosts the cloud fleet; 100% renewable supply | 10-K, 26 March 2026 |
| **Atlanta, Georgia** | Atlanta | 6 MW gross; contracts executed for 5.5 MW IT load | **Leased** capacity for cloud services; leases commenced February 2026 | 10-K, 26 March 2026 |
| **Sydney** | Australia | Not disclosed | Lease of data-centre space signed 30 July 2026; **59 months**; AUD 488k (~$344k) monthly; commences Q4 2026 | 10-Q, 12 August 2026 |

Two structural observations follow immediately from that table, and both matter more than the capacity totals.

**The company owns the buildings it is building and leases the buildings it is filling.** NC-1 and MTL-3 are owned (MTL-3 via an exercised purchase option); MTL-1, Blonduos, Atlanta and Sydney are leases. That is a deliberate split: the owned estate is the colocation product, where the landlord needs to control the asset for decades; the leased estate is the cloud product, where the company needs speed and does not want the capital commitment. The 10-K states the logic for equipment directly - "we anticipate utilizing equipment leasing facilities to limit our equity capital commitments" - and the same logic is visible in the property choices.

**The leased estate carries the same clock as the contracts that fill it.** A 59-month Sydney lease against 12-month cloud service orders (Section 6) is a matched book only if the orders renew. A lease is a fixed obligation; a service order is not. This is the colocation-segment version of the two-clock problem, and it runs in the opposite direction from the GPU version: here the *building* obligation is short and the *customer* commitment is shorter.

### 3.2 The Contract Structure That Makes It Landlord Economics

The Nscale agreement is the clearest specimen and the company's own description is precise: approximately **$865 million of total contracted revenue over a 10-year term, inclusive of contractual annual rate escalators and non-recurring installation services, with electricity and certain other operating costs structured as pass-through charges** (10-K, 26 March 2026). Four features make that landlord economics rather than a services contract:

1. **Term exceeds the debt.** A 10-year contract can support project finance. The Syndicated RBC facility signed 6 July 2026 has a three-year term amortising on a 15-year schedule (10-Q, 12 August 2026) - short tenor, long amortisation, which is what a lender does when it believes the contract outlives the loan.
2. **Escalators are contractual, not negotiated.** Revenue rises without additional capital.
3. **Power is a pass-through.** The landlord is not short electricity. This is the single most important risk allocation in a data-centre lease and the company has it right at NC-1.
4. **Installation revenue is non-recurring and recognised over the term.** The 10-Q confirms non-recurring installation fees "are deferred and recognized ratably over the contract term" (12 August 2026) - conservative, and it means the $865 million headline is not front-loaded.

### 3.3 What a Tier III Claim Actually Means

The company states that its "operational data centers **meet the requirements of the Tier-3 standard**, including N+1 redundancy architecture, concurrent maintainability, uninterruptible power supply, advanced and highly reliable cooling systems, strict monitoring and management systems, 99.982% uptime and no more than 1.6 hours of downtime annually, service organization control, SOC 2 Type 2..." (10-K, 26 March 2026; repeated verbatim in the 10-Q of 12 August 2026). The marketing site is more careful and less strong: "**Tier III designed** facilities" (whitefiber.com, retrieved November 2026).

That distinction is the whole point. Per `data_center_guide.md` section 4.1 and section 9.1, Tier III is a **design topology** claim - concurrent maintainability - and the Uptime Institute's certification is a separate, paid, site-specific programme with a certificate number. "Meets the requirements of the standard" and "Tier III designed" both describe intent and topology; neither is a certification, and neither is a measurement of delivered availability. The 99.982% figure is the availability a concurrently maintainable topology is *designed* to deliver, not a figure the company reports having achieved. The arithmetic is at least internally consistent: 8,760 hours x (1 - 0.99982) = 1.58 hours, which is the "no more than 1.6 hours" in the same sentence. But a design target and a measured uptime are different instruments, and no filing in this set reports measured availability for any facility.

Two further qualifications, both from the disclosures themselves. First, the density claim is inconsistent across channels: the 10-K describes enterprise customers benefiting from "high-density solutions, reaching upwards of **50kW per cabinet**"; the marketing site advertises "Accommodating up to **150 kW per cabinet**" for the same colocation product (retrieved November 2026). Density is the binding constraint on what a colocation building can sell per square metre, so a 3x gap between the filing and the brochure is not cosmetic - it is the difference between a building that can host current-generation racks and one that cannot. Second, the filing's own risk that "colocation sold against density the building cannot cool" is not disclosed as a risk at all; section 11 treats it as a failure mode precisely because the filings are silent.

> **Judgement:** treat Tier III as a topology statement with an SOC 2 Type 2 audit attached to the *controls*, not to the *availability*. The only availability numbers in the record are a design target (99.982%), a contractual floor for the cloud product ("> 99.5%"), and a marketing average (99.95%). None of the three is a measurement.

---

## 4. The Cloud Business: Accelerators by the Month

### 4.1 What Is Rented, and at What Granularity

The cloud segment rents NVIDIA accelerators, principally H200, B200 and GB200 generations at the time of the FY2025 filing, in configurations the company describes as H200, B200 and GB200 servers, sourced from Quanta Cloud Technology, Dell (via Advania in Iceland), Supermicro and HPE, under NVIDIA partner programmes (10-K, 26 March 2026). The first GB200 NVL72 shipment arrived in April 2025 from Quanta.

The unit of sale is not uniform, and the variation is the segment's most under-appreciated feature. Across the contracts disclosed in the 10-K of 26 March 2026, WhiteFiber sold:

- **by the week** - three service orders for 64 B200 GPUs each, terminable by not renewing with mutual written agreement; one renewed for a second week;
- **by the month, auto-renewing** - 16 B200 GPUs renewing monthly on seven days' notice (not renewed after the initial term); 128 B200 GPUs on a 12-month initial term then auto-renewing monthly; 256 GPUs on a 12-month initial term then monthly; 72 GB200 GPUs on a 12-month initial term then monthly;
- **by the multi-month block** - 16 H200s for six months (~$160,000); 32 H200s for six months (~$300,000); 24 H200s for 12 months (~$450,000); 40 H200s for 12 months (~$750,000, later cut to 8 GPUs and extended to May 2027);
- **by the multi-year reservation** - 2,048 GPUs over three years (the "Initial Customer"); 576 H200s over 25 months (DNA Fund); 256 H100 servers over 24 months with a 12-month renewal option (~$50.2 million, signed March 2026); 384 B200s over 24 months (~$18.1 million, signed February 2026).

The marketing site sells the last of those categories and only the last: "**Reserved GPU Cloud Instances** ... multi-year reservation commitments on Blackwell and Grace Blackwell 300 class GPUs" (whitefiber.com, retrieved November 2026). The filings show a book in which week-to-week and month-to-month orders are a material part of the activity.

### 4.2 The SLA, and the Three Different Numbers

The 10-K states: "We provide a high-standard service lease with an **Uptime percentage > 99.5%**" (26 March 2026). The marketing site states: "**AVERAGE 99.95% Uptime**" (whitefiber.com/cloud/compute, retrieved November 2026). The colocation disclosure states a design target of **99.982%** (10-K, 26 March 2026).

Converting all three to permitted downtime per year, which is the only form in which they can be compared:

| Figure | Where it appears | Downtime permitted per year |
|---|---|---|
| 99.982% | Filing, colocation design target | 1.58 hours |
| 99.95% | Marketing, cloud compute page | 4.38 hours |
| > 99.5% | Filing, cloud service lease floor | 43.8 hours |

The marketing number is **2.8 times more permissive** than the colocation design figure and **ten times tighter** than the contractual cloud floor. None of the three is a measurement; all three are commitments or aspirations of different kinds. A buyer who reads "99.95%" as a service level and a buyer who reads "> 99.5%" as one are buying different products, and the difference is 39 hours a year.

There is a second-order problem with all three, and it is the one that matters operationally: **uptime of what?** An accelerator cluster can be "up" - the host responds, the scheduler places jobs, the fabric is reachable - while the customer's training job fails on a correctable memory error, a thermal throttle, or a silently degraded GPU. `data_center_guide.md` section 10.4 owns the general problem of what counts as downtime; `gpu_cloud_providers_comparison_guide.md` section 5 owns the availability question for this category. For present purposes the point is narrower: the filings disclose a percentage and a definition of the *lease*, not a definition of the *measurement*, and no service-credit mechanics are disclosed in any of the three filings.

### 4.3 Granularity as an Economic Fact

Granularity is not a product decision; it is a statement about who carries the utilisation risk. When WhiteFiber sells 64 B200s for a week, the customer carries almost nothing and WhiteFiber carries everything: the asset, the depreciation, the power, and the re-letting risk that begins in seven days. When it sells 2,048 GPUs for three years, the reverse is true. The disclosed contract set spans both extremes, and the revenue line aggregates them. That is why the segment gross margin is not a stable number: it is a blend of a rental business and a reservation business, and the mix moves quarter to quarter with no disclosure of the mix.

The Q2 2026 numbers show what happens when the blend moves. Cloud revenue rose 43.5% year on year to $23.8 million, but $12.3 million of that was a **termination fee** recognised as revenue in the quarter (Form 10-Q, 12 August 2026). Strip the fee and cloud revenue was roughly flat to down against a fleet that grew. The company says so itself: the decrease from the Initial Customer termination "was substantially offset by $12.3 million of termination fee revenue recorded, and therefore was not a significant driver of the change in revenue." That sentence is doing a lot of work, and Section 7 returns to it.

### 4.4 Where the Cloud Revenue Is Actually Produced

One structural fact deserves its own heading because it is easy to miss between the segment tables and the property narrative: **the cloud segment's revenue is produced almost entirely in one leased building in one country.**

The geographic disclosure in the FY2025 10-K shows revenue by location of service: **Iceland $69,167,349**, Canada $8,913,816, other countries $1,083,087, against total revenue of $79,164,252 (26 March 2026). Iceland is where the cloud fleet sits, and Iceland is the **Blonduos Campus, which WhiteFiber leases rather than owns** (10-K, 26 March 2026: "our initial data center partnership through which we lease capacity"). The 10-Q confirms the position is essentially unchanged into 2026: "The Company's cloud services revenue has been generated from Iceland", with an immaterial amount in Canada from March 2026 (12 August 2026).

Set that next to the long-lived asset disclosure from the same filing: United States **$124,522,633**, Iceland **$113,516,239**, Canada **$122,776,181**, total **$360,815,053** (10-K, 26 March 2026). So the segment that produces 87% of revenue holds, at most, the Iceland long-lived assets - and much of what sits in Iceland may be the landlord's building rather than WhiteFiber's equipment.

The consequences are three, and none of them is disclosed as a risk in these terms:

1. **The cloud business is a tenant.** Its entire revenue stream depends on a lease it does not control, in a jurisdiction where it is a foreign subsidiary classified as a controlled foreign corporation for US tax purposes (10-K, 26 March 2026). A landlord dispute, a lease expiry, or a power allocation decision in Iceland is a revenue event for the whole company.
2. **The diversification story is geographic, not operational.** Adding Atlanta (leases commenced February 2026) and Sydney (signed 30 July 2026) reduces single-site dependence, and both are leases as well. The company is diversifying its tenancies, not its ownership.
3. **The colocation segment's growth is what converts this.** $556.3 million of colocation equipment at 30 June 2026 against $95.8 million of cloud equipment (10-Q, 12 August 2026) is the company moving its capital into buildings it owns and can rent for a decade, away from a business whose revenue is produced in someone else's building on 12-month orders. Read that way, the NC-1 build is not diversification for its own sake; it is the company buying the clock it wants to be on.

---

## 5. The Two Clocks: Depreciation and Obsolescence

### 5.1 The Stated Lives

From Note 2 of the FY2025 10-K (filed 26 March 2026), verbatim in structure:

| Asset category | Estimated useful life |
|---|---|
| **Cloud service equipment** | **5 years** |
| **Colocation service equipment** | **10 to 15 years** |
| **Building** | **30 years** |
| **Leasehold improvements** | **15 years** |
| **Purchased and internally developed software** | **1 to 5 years** |
| **Other property and equipment** | **20% to 30%** (declining-balance basis) |
| Customer relationships (Enovum acquisition) | **19 years**, $13.5 million |

### 5.2 The Disclosure That Matters Most

> **"Effective January 1, 2025, we changed our estimate of the useful lives for our cloud service equipment from three to five years. The change was made to better reflect the expected usage patterns and economic benefits of the assets."**
>
> - Note 2, Summary of Significant Accounting Policies, FY2025 Form 10-K, filed 26 March 2026. Repeated in MD&A at the same filing.

This is a **67% extension of the depreciation period on the company's central revenue-producing asset**, effective the first day of the fiscal year in which the company listed. The filing quantifies the effect precisely: "a reduction in depreciation and amortization expense of **$10.0 million** and a benefit to net loss of **$7.9 million**, or **0.25 per basic share and 0.25 per diluted share** for the year ended December 31, 2025" (Note 2, 10-K, 26 March 2026).

Three things should be said about that, in order of importance.

**It is not improper.** Useful-life estimates are management judgements, reviewed annually, and a change is accounted for prospectively as a change in estimate. The disclosure is complete, quantified, and placed in the accounting policies where a diligent reader will find it. Nothing in the record suggests anything other than a genuine reassessment.

**It is load-bearing.** $7.9 million of benefit to net loss, in a year in which the net loss before taxes was $25,768,370, is roughly **31% of the reported pre-tax loss**. Had the company retained a three-year life, the FY2025 pre-tax loss would have been in the region of $33.7 million on the same revenue. The company's headline loss is materially a function of an estimate, and the estimate moved in the year of the IPO.

**It moves the clock, not the asset.** A GPU does not last five years because it is depreciated over five. It lasts as long as somebody will pay for it. The accounting change aligns the book life with management's expectation of economic benefit; it does not create that benefit. Every subsequent discussion of "book value" in this guide should be read with that sentence in mind.

### 5.3 The Counter-Case: A Fully Depreciated GPU Still Earns Revenue

The strongest argument against treating the depreciation schedule as the risk is that **it is not a forecast of the asset's death, it is a schedule for reaching zero book value - and zero book value is the best place for an asset to be.** A GPU written down to nil that still rents produces revenue with no depreciation charge against it. On that view, the aggressive-looking five-year life is conservative: it front-loads the cost, gets the asset to a fully depreciated state inside the first contract term, and any revenue after that is close to pure margin. The company's own behaviour is consistent with this: it deployed "the Company's inventory of B200 GPUs" to an 18-month Initial Customer contract in August 2025, and after the customer paused, it "redeployed the GPUs previously allocated to the Initial Customer to three other customers" (10-K, 26 March 2026). Redeployment is the mechanism by which a depreciated asset keeps earning, and the company has demonstrated it at least once.

There is also direct evidence that resale value has recently been **better** than book. In the first half of 2026 the company sold H200 servers "with carrying value of $24.3 million" and recorded a **net gain on disposal of $1.8 million** (Form 10-Q, 12 August 2026, non-GAAP reconciliation and PP&E note). A gain on disposal of a depreciating asset is the cleanest possible refutation of the claim that the book life is too long - at least for that SKU, at that moment.

### 5.4 The Sceptical Case: Resale, Generations, and the Shape of the Curve

The sceptical case does not deny the counter-case; it denies its generality.

**Generation-over-generation jumps compress the window in which an old SKU is worth renting.** The company's own contract history is a catalogue of customers upgrading mid-contract. In mid-2024 the Initial Customer asked to "temporarily delay the purchase order so the customer could evaluate an upgrade to newer generation Nvidia GPUs" (10-K, 26 March 2026). The 464-GPU replacement agreement was served from B200 inventory rather than the H100s originally contemplated. A customer who upgrades every 18 months is a customer for whom a five-year asset is a two-generation asset.

**The sale-and-leaseback structure reveals what a third party thought the residual was worth.** WhiteFiber sold 96 servers (768 GPUs) and leased them back over three years, then sold 128 servers (1,024 GPUs) and leased them back over three years (10-K, 26 March 2026). A three-year sale-leaseback is a market transaction in which the lessor takes residual-value risk for exactly three years. That is a revealed preference about the horizon over which a professional counterparty will underwrite GPU value, and it is shorter than the five-year book life.

**Obsolescence is already being realised, just not on the GPUs.** In the first half of 2026 the company recorded a **$5,006,000 impairment of capitalised software assets**, described as "the one-time write-off due to the discontinuation of the internally developed software operations" (Form 10-Q, 12 August 2026). The software category carries a 1 to 5-year life and was written off inside two years. The mechanism that the sceptic fears - an asset class whose economic value collapses faster than its schedule - has already happened once on this balance sheet, in the adjacent category.

**The Initial Customer settlement shows what a contract is worth when the counterparty stops performing.** The company believed an early termination fee "would be equal to 40% of the fees that would have accrued for services during the remainder of the term of the MSA and applicable purchase orders" (10-K, 26 March 2026). What it actually collected, per the Q2 2026 filing, was a **fixed termination fee of $12.3 million, recognised as revenue in the quarter, subsequently amended to $15.7 million and still outstanding at the date of the filing**, against $12.5 million of preserved receivables that were collected and a **$2.2 million bad debt expense** on the unpreserved balance (Form 10-Q, 12 August 2026). The contractual protection was real; it was also renegotiable, partial, and paid late.

### 5.5 The Argument, Stated Without a Sensitivity Table

This guide will not print a depreciation sensitivity table, because every input that would make one interesting - resale value by SKU and vintage, utilisation by month, re-letting price, the date of the next generation - is undisclosed. A table built on invented inputs would be a machine for producing false precision.

The argument can be made in one paragraph instead. The company depreciates accelerators over five years, or 20% a year straight-line, and it depreciates buildings over thirty. Its disclosed cloud contracts have a modal initial term of 12 to 24 months (Section 6), and its disclosed third-party financing of accelerators runs three years. So the *contract* recovers between 20% and 40% of the asset's cost, the *book* recovers 20% a year for five years, and the *market* - as revealed by the sale-leaseback counterparties - is willing to underwrite residual value for three. Two of those three clocks are inside the third. The question is not whether the five-year life is right; it is **which clock the business is actually running on**, and the answer differs by contract: a 24-month reservation on a five-year asset is a bet that the customer renews or that the asset re-lets at a price that covers the remaining 60% of book. That bet has been won once (redeployment to three customers, gain on H200 disposal) and lost once (the Initial Customer pause, a $2.2 million bad debt, and a termination fee that had to be renegotiated upward and is still uncollected).

> **Judgement:** the five-year life is defensible and the extension was disclosed properly; but it is an estimate made in the year of the IPO, it is worth 31% of the reported pre-tax loss, and the disclosed contract terms do not support it without an assumption of renewal that the filings never state explicitly.

---

## 6. The Contracts: Term Against Asset Life

### 6.1 The Cloud Book, Contract by Contract

Assembled from the business section of the FY2025 10-K (filed 26 March 2026), which describes each agreement it "consider[s] to be material or representative":

| Counterparty / type | Scale | Initial term | Contracted value as stated | Outcome in the record |
|---|---|---|---|---|
| Initial Customer (MSA, 12 December 2023, as amended) | 2,048 GPUs | **3 years** | "more than $50 million of annualized revenue" | Paused; terminated Q2 2026; fee renegotiated |
| Initial Customer (January 2025 replacement) | 464 GPUs | **18 months** | ~$15m annualised | Deployed 20 August 2025 from B200 inventory |
| DNA Fund (30 December 2024) | 576 H200s | **25 months** | Not stated; 130 servers ordered at ~$30m | Terminated November 2025; $7.3m receivable, $2.1m collected at report date |
| DNA Fund (April 2025, first) | 104 H200s | **23 months** | Not stated | Deployed May 2025 |
| DNA Fund (April 2025, second) | 512 H200s | **24 months** | Not stated | Deployed July 2025 |
| Customer MSA (6 November 2024) | 16 H200s | **6 months minimum** | ~$160,000 | Concluded May 2025 |
| Customer MSA (6 January 2025) | 32 H200s | **6 months minimum** | ~$300,000 | Concluded April 2025; early termination penalty paid |
| Customer MSA (January 2025) | 24 H200s | **12 months minimum** | ~$450,000 | Concluded March 2025; customer ceased operations |
| Customer MSA (30 January 2025) | 40 H200s | **12 months minimum** | ~$750,000 | Amended down to 8 GPUs, extended to May 2027 |
| Customer service order (March 2026) | 256 H100 servers | **24 months** + 12-month option | ~$50.2 million | Deployment expected Q2 2026 |
| Customer service order (February 2026) | 384 B200s | **24 months**, then monthly auto-renew | ~$18.1 million | Commenced 27 January 2026 |
| Customer service order (November 2025) | 128 B200s | **12 months**, then monthly auto-renew | ~$3.0 million | Commenced 1 December 2025 |
| Customer service order (February 2026) | 256 GPUs | **12 months**, then monthly auto-renew | Not stated | Commenced 1 February 2026 |
| Customer service order (March 2026) | 72 GB200s | **12 months**, then monthly auto-renew | Not stated | Commenced 7 March 2026 |
| Boosteroid (MSA, 9 October 2024) | 489 GPUs initially | **through November 2029** | ~$7.9m aggregate | Plus 302+120+279 GPUs, ~$10.4m over five years |
| Boosteroid expansion option | Up to 50,000 servers | Five-year term | "potential contract value of approximately $700 million" **if** Boosteroid uses full capacity | Discretionary; "Boosteroid has full discretion" |

### 6.2 What the Table Says

**The modal initial term is 12 to 24 months.** Of the sixteen disclosed cloud agreements, three run longer than three years (the Initial Customer's original 3-year MSA, Boosteroid's five-year MSA, and one 36-month order that "was terminated in December 2025" after two months). Everything else is 6 to 25 months, and the most recent additions cluster at 12 and 24 months.

**The largest headline number is an option, not a contract.** Boosteroid's "approximately $700 million over the five-year term" is explicitly conditional - "assuming Boosteroid utilizes the GPUs and services at full capacity for the duration of the contract" - and the expansion "depends upon the internal development roadmap of Boosteroid", which has "full discretion". It is disclosed honestly and it should never be aggregated into a backlog. This guide does not.

**The book is short and the renewals are monthly.** Six of the disclosed agreements renew automatically for successive one-month periods unless terminated. A one-month renewal is not a contract term; it is a notice period. The practical duration of the segment's revenue is therefore closer to the notice period than to the initial term.

**The one long contract in the cloud book is small and unusual.** Boosteroid is a cloud-gaming provider, not an AI lab, and its five-year MSA includes a right of first refusal over the next 5,000 servers Boosteroid leases. It is a good contract. It is not the shape of the book.

### 6.3 The Colocation Book, for Contrast

| Counterparty | Term | Contracted value | Structure |
|---|---|---|---|
| **Nscale** (NC-1) | **10 years** | **~$865 million** total contracted revenue, inclusive of escalators and non-recurring installation charges | Electricity and certain operating costs as pass-throughs; billing for the 40 MW phase expected from June 2026 |
| **Cerebras** (MTL-3) | **5 years** | CAD 1.4m (~$979k) per month | Commenced 1 November 2025 |
| Other colocation tenants | "generally one to five years" | Not individually disclosed | Fixed monthly fees on committed kW; recurring billed monthly, recognised ratably |

The contrast is the thesis in two rows. **The colocation book has one 10-year contract worth $865 million; the cloud book has sixteen contracts, most under two years, worth tens of millions each.** Remaining performance obligations at 30 June 2026 make the asymmetry explicit (Form 10-Q, 12 August 2026, in thousands):

| | 2026 | 2027 | 2028 | 2029 | 2030 | Thereafter | Total |
|---|---|---|---|---|---|---|---|
| **Colocation services** | 36,053 | 93,465 | 94,835 | 95,285 | 94,021 | 519,236 | **932,895** |
| **Cloud services** | 21,627 | 43,254 | 10,265 | - | - | - | **75,146** |
| **Total RPO** | 57,680 | 136,719 | 105,100 | 95,285 | 94,021 | 519,236 | **1,008,041** |

Cloud RPO is **7.4% of total** and **exhausts entirely by 2028**. Colocation RPO is $932.9 million, of which $519.2 million falls after 2030. The filing notes that these amounts "exclude variable consideration allocated entirely to wholly unsatisfied performance obligations" - so cloud, which is priced per GPU per day, is systematically under-represented in this table. That caveat is real and cuts against the comparison. But it cuts against the cloud number's *completeness*, not its *shape*: a business whose contracted, fixed, disclosed backlog ends in 2028 is a business that must re-win its revenue every year, and there is no disclosure that lets a reader model the re-win.

### 6.4 Renewal Risk and What Happens When the Generation Turns

Three renewal risks are visible in the record, and one is not.

**Visible: the customer upgrades and the contract restarts at a lower price.** The Initial Customer's 2024 request to delay a purchase order to "evaluate an upgrade to newer generation Nvidia GPUs" converted a 3-year, 2,048-GPU agreement into an 18-month, 464-GPU agreement at ~$15 million annualised. Same customer, same relationship, materially smaller and shorter commitment. That is what a generation turn looks like from the seller's side.

**Visible: the customer's ownership changes and the contract ends.** One six-month MSA "concluded in April 2025 following a change in the customer's ownership, and the customer paid the remaining contract value as an early termination penalty" (10-K, 26 March 2026). Contractual protection worked here, in the small.

**Visible: the customer ceases operations.** Another concluded in March 2025 "after the customer ceased operations" - no penalty mentioned, and the counterparty presumably had nothing to pay with.

**Not visible: what the re-let price is.** The filings disclose revenue in aggregate and RPO in aggregate. They do not disclose price per GPU-hour, utilisation, or the price achieved when capacity is redeployed from a paused customer to three others. Without that, renewal risk can be described and cannot be quantified from outside.

> **Judgement:** the contract term in the cloud segment is a notice period dressed as a commitment, and the segment's economics depend on a renewal assumption that the filings neither state nor support with data. The colocation segment is the opposite: the contract is genuinely long, the counterparty is identifiable, and the power is a pass-through. The two books are not comparable, and the consolidated RPO figure hides that they are not.

---

## 7. Customer Concentration: The Pause as Test

### 7.1 What the Filings Disclose, Printed Exactly

The FY2025 10-K (filed 26 March 2026) states, in the business section and again in the risk factors:

> **"Our Initial Customer accounted for approximately 70.7% of our revenue during the 12 months ended December 31, 2025 and 96.6% of our revenues through December 31, 2024."**

> **"DNA Fund accounted for approximately 11.5% of our revenue during the 12 months ended December 31, 2025. We did not generate any revenue from DNA Fund in 2024."**

> **"During 2025, our HPC data center in Iceland had contracts with twenty one customers."**

On the colocation side, the same filing states that as of 31 December 2025 the leased MTL-1 and MTL-3 facilities "served fifteen customers. **No one customer accounted for in excess of 50% of data center revenue** in the 12 months ended December 31, 2025 or 2024."

By the Q2 2026 filing (12 August 2026) the picture had changed again:

> **"As of the date of this Form 10-Q, we have seven existing customers. Our largest customer accounted for approximately 63% of our revenue during the six months ended June 30, 2026. During the period we had discontinuation of three customer orders. The discontinued orders resulted in approximately $5.1M impact to revenue during the six months ended June 30, 2026. However, there were new customer orders contracted in the six months ended June 30, 2026 and through the date of this Form 10-Q for total contracted revenue of $635.8M over a six months to three-year period."**

### 7.2 The Credit Event, as Disclosed

The Initial Customer did not simply leave. The sequence, with dates and sources:

1. **October 2025** - the existing parent guarantee was due to expire. From November 2025 the customer was to provide a service deposit "funded through fifteen consecutive monthly payments of approximately $0.24 million each, totaling $3.6 million, payable from November 2025 through January 2027" as security for performance (10-K, 26 March 2026). A guarantee being replaced by a slow-paying deposit is a deterioration in credit support, disclosed in plain language.
2. **Year-end 2025 reporting date** - "the Company and the Initial Customer are engaged in discussions regarding a potential resolution of the existing service agreements following the **agreed pause of services**. No definitive termination or settlement agreement has been executed" (10-K, 26 March 2026). The parties were negotiating "the treatment of the remaining non-refundable prepayment, service deposit, outstanding receivables, and a potential early termination fee, which the Company believes would be equal to 40% of the fees that would have accrued for services during the remainder of the term."
3. **Already by the 10-K** - "Following the service pause, the Company has redeployed the GPUs previously allocated to the Initial Customer to three other customers."
4. **Q2 2026** - "the Company executed a termination agreement with the Initial Customer. The Termination Agreement **preserved $12.5 million of previously invoiced, unpaid trade receivables**. This preserved balance was fully collected as of June 30, 2026. Prepayment and service deposit balances were applied against other outstanding receivables and the Company recognized a **bad debt expense of approximately $2.2 million** for the unpreserved remaining receivable balance... the Initial Customer is obligated to pay the Company a **fixed termination fee of $12.3 million that was recognized as revenue during the second quarter of 2026**. Subsequently, after quarter-end, the termination fee was **amended to $15.7 million**. The amended amount of $15.7 million **remains outstanding** as of the date of this Form 10-Q" (Form 10-Q, 12 August 2026).

The customer is never named in any of the three filings. Media and market commentary have speculated; this guide does not, and Section 15 records the omission as a verification gap rather than filling it.

### 7.3 Reading the Pause Against the Thesis

This is the guide's central empirical test, and the evidence runs in both directions. Both directions should be stated.

**Evidence that supports the thesis.** A single customer at 70.7% of revenue on a 3-year contract against 5-year assets is precisely the concentration the two-clock view predicts will hurt. When that customer paused, the immediate consequence was a receivable problem ($2.2 million written off), a revenue problem (the fee had to be recognised as revenue to hold the cloud line flat, per the company's own Q2 explanation), and a **collection** problem (the amended $15.7 million fee is still outstanding). The assets did not disappear; they were redeployed. The revenue did not disappear; it was replaced by a one-off fee. That is what an asset-heavy, contract-light business looks like when its anchor tenant leaves: the P&L can be held together for a quarter by a settlement, while the underlying run-rate resets.

**Evidence that complicates the thesis, and this is the harder point.** The redeployment worked. The GPUs went to three other customers. The company sold H200 servers at a **gain** over carrying value. Cloud revenue in Q2 2026 was still up 43.5% year on year. If the thesis were simply "short contracts on long assets destroy value", this episode should have produced an impairment; it produced a gain on disposal and a termination fee. The five-year life was not tested and found wanting - it was tested and, on this occasion, survived, because there was a secondary market and a queue of replacement tenants. **The risk the thesis identifies is real but it is a liquidity risk, not an accounting one: the asset retains value, and the question is how long it takes to re-let and at what price.** The filings do not disclose the re-let price, which is exactly why the survival cannot be graded.

**And the concentration has not gone away - it has changed clocks.** At 30 June 2026 the largest customer was 63% of revenue. That customer is almost certainly a colocation tenant, because the Nscale agreement (announced November 2025, ~$865 million over 10 years, billing from June 2026) is the only contract in the book large enough to produce that share. So the company has replaced a 70.7% cloud customer with what will be a very large colocation customer. **Same concentration, different clock, better collateral.** That is the thesis working as designed - and it is also a warning: the concentration risk did not diversify, it migrated to the segment where the contract is long enough to survive it.

---

## 8. Power and the Physical Constraint

### 8.1 Own, Lease or Buy - the Three Different Answers

WhiteFiber's power position is not one position. It is three, and they carry different risks.

**Quebec (MTL-1, MTL-2, MTL-3): bought from a crown corporation at regulated rates.** "In the province of Quebec where MTL-1, MTL-2, and MTL-3 are located, all of the hydroelectric power is provided by a crown corporation, Hydro Quebec, which has predetermined rates depending on the customers' industry and based on the power demand" (10-K, 26 March 2026). This is the best power position in the portfolio: regulated, hydroelectric, and priced by tariff rather than by negotiation. It is also the position with the least growth headroom, because a crown utility's allocation to a new industrial customer is a political and engineering decision, not a commercial one.

**North Carolina (NC-1): contracted, but not delivered.** This is where the thesis meets its hardest physical fact. The Capacity Agreement with Duke Energy, dated 16 May 2025, has Duke "use **commercially reasonable efforts** to achieve 24 MW (gross) of service to NC-1 by September 1, 2025, 40 MW (gross) by April 1, 2026, and 99 MW (gross) within four years of May 16, 2025" (10-K, 26 March 2026). "Commercially reasonable efforts" is not a delivery obligation. And the purchase price of the site is **contingent on delivery**: an extra **$8 million** if Duke provides, or provides an Electric Services Agreement for, at least 99 MW within two years; **$5 million** if within three; plus **$200,000 per MW** above 99 MW up to a maximum of **$5 million** within four years (10-K, 26 March 2026; restated in the 10-Q of 12 August 2026).

That contingency is the most informative disclosure in the entire filing set, and it should be read as what it is: **a contractual admission, negotiated with the seller, that the power may not arrive.** A company that believed it had 99 MW would not have accepted a price mechanism that pays the seller more only if the power shows up. And at 30 June 2026: "the Company has **not received an Electric Service Agreement of more than 99 MW**. Thus, no contingent payment is payable as of the reporting date" (Form 10-Q, 12 August 2026). Two years into a contract whose first milestone was 24 MW by 1 September 2025, the 99 MW agreement does not exist.

The power that *does* exist at NC-1 is a legacy industrial service assignment from the previous owner, and its terms are disclosed in unflattering detail: "The ESA establishes a **minimum monthly bill** for electric service, based on Duke Energy's Rate of $8,754, **irrespective of actual usage levels**"; Duke has installed "Extra Facilities (including overhead lines, substations, transformers, breakers, and metering equipment)" costing approximately **$1,137,975**, for which the company pays **$11,405 a month**; the company "is obligated to pay the minimum monthly charges regardless of operational activity"; and on **15 July 2026** it posted a **$3 million surety bond** from Great American Insurance Company in Duke's favour "as security for the Company's payment obligations" (Form 10-Q, 12 August 2026). A take-or-pay obligation to a utility, secured by a bond, at a site whose anchor customer has not yet been billed, is the physical-constraint risk in its purest form.

**Iceland, Atlanta, Sydney: capacity leased from third-party operators.** Blonduos (6 MW gross, 45 kW rack density, 5.5 MW IT contracted, 100% renewable from the Blanda hydro station) is "our initial data center partnership through which we lease capacity" (10-K, 26 March 2026). Atlanta capacity leases commenced February 2026. Sydney was signed 30 July 2026 for 59 months at AUD 488k (~$344k) monthly. None of these is owned; all of them put a third party between WhiteFiber and its customer.

### 8.2 The Pipeline Claim and the Number That Moved

The 10-K (26 March 2026) states an intention "to achieve an estimated **76 MW (gross)** of total data center capacity by the end of the fourth quarter of 2026, a target that is underpinned by assets including our MTL-2, MTL-3, and NC-1 facilities", against "approximately **1,500 MW (gross)** under management review" as of 31 December 2025.

The 10-Q of 12 August 2026 states the same intention at **70 MW (gross)** by the end of Q4 2026, with the same supporting assets and the same 1,500 MW under review. **The target moved down by 6 MW in five and a half months, in the same period in which MTL-2 was disclosed as deprioritised.** That is not a scandal; it is what a pipeline looks like when one project slips and the arithmetic is redone. But it is the only forward-looking capacity number the company publishes, and it moved in one direction.

The marketing site is less restrained: "scalability through our **1.3GW energy pipeline**" (whitefiber.com/data-center/ai-hpc-colocation, retrieved November 2026). 1.3 GW is 1,300 MW, which is 87% of the 1,500 MW the filing describes as "potential projects **under management review**" - a review, not a pipeline, and with no commitment attached to any of it. A reader should treat 1.3 GW as a marketing restatement of a screening list.

### 8.3 What the Physical Constraint Means for Growth

Growth in this business has a sequence, and the sequence is not parallelisable: **site, then power, then building, then customer, then revenue.** The filing describes a retrofit model designed to compress the middle of that sequence - "average build time for retrofits is intended to be approximately six months from commencement of construction, which we believe is approximately one-third to one-half of the industry average development timeline for greenfield projects" (10-K, 26 March 2026) - and the MTL-3 evidence supports it: lease signed 11 April 2025, operational November 2025, seven months. That is a genuine capability and it is the company's real competitive asset.

But the sequence begins with power, and power in North Carolina is the constraint the company cannot compress. The NC-1 economics are, in effect, a bet that a utility will deliver 99 MW to a former manufacturing site in Madison, North Carolina, on a schedule that the utility has not committed to, while the company pays a minimum monthly bill and has posted a surety bond. The 40 MW phase is expected to bill from June 2026. Whether it does is the single largest swing factor in the FY2026 numbers, and it is not within the company's control.

### 8.4 The Three Sources, Ranked by Risk

Stated as a ranking of risk rather than of size, because that is the useful ordering for a reader deciding what to worry about:

| Source | Jurisdiction | Contractual character | Who bears the risk of non-delivery |
|---|---|---|---|
| **Hydro Quebec** (MTL-1, MTL-2, MTL-3) | Quebec | Regulated tariff; allocation is an administrative decision | The customer, in the sense that no power means no revenue, but the tariff protects against price shock. Lowest price risk, highest allocation risk. |
| **Duke Energy** (NC-1) | North Carolina | "Commercially reasonable efforts" to reach milestones; a legacy industrial ESA with a minimum monthly bill and a $3m surety bond | **WhiteFiber bears it twice**: it pays the minimum bill whether or not it uses the power, and it pays the seller more only if the power arrives. The contingent purchase price is the market's own estimate of delivery probability, and it is a bonus, not a discount. |
| **Third-party landlords** (Blonduos, Atlanta, Sydney) | Iceland, Georgia, Australia | Leases; the landlord holds the utility relationship | The landlord, contractually, and WhiteFiber commercially - because its cloud revenue depends on a facility it does not control and cannot replace quickly. |

The ordering matters because it inverts the usual intuition. The site with the *most* contracted megawatts (NC-1) has the *weakest* power instrument; the site with the *cleanest* power (Quebec, crown corporation, regulated tariff) has the least headroom to grow. A reader who assumed that contracted capacity equalled secured capacity would get this exactly backwards.

> **Judgement:** the company owns buildings, buys power, and leases cloud capacity. The building is the part it controls; the power is the part it does not; and the leases are where it has chosen speed over control. That ordering - control the shell, rent the metal, beg for the megawatts - is a rational strategy for a small developer and an uncomfortable one for a lender.

---

## 9. Capital Structure and Capex Commitments

### 9.1 What Was Raised

| Instrument | Date | Amount | Terms as disclosed | Source |
|---|---|---|---|---|
| IPO, ordinary shares | 8 August 2025 | **$159,375,000 gross** (9,375,000 x $17.00), before $11,156,250 of underwriting discounts and expenses | Ordinary shares, $0.01 par | 10-K, 26 March 2026 |
| Over-allotment | 2 September 2025 | **$23,906,250 gross** (1,406,250 x $17.00) | Full exercise | 10-K, 26 March 2026 |
| 4.500% Convertible Senior Notes due 2031 | 26 January 2026 | **$230,000,000 principal**, including the full $20m purchasers' option | General senior unsecured; interest payable 1 February and 1 August from 1 August 2026; conversion at an initial 49.2126 shares per $1,000, i.e. ~$25.91 per share; up to 11,318,898 shares issuable | 10-K, 26 March 2026; 10-Q, 12 August 2026 |
| Zero-strike call option (capped call) | 26 January 2026 | **$120,000,000 premium paid** to Barclays Bank PLC for the right to receive 5,905,511 shares | Reduces net proceeds; an economic hedge against dilution, not a source of funds | 10-K, 26 March 2026 |
| Landsbankinn hf. term loan (WhiteFiber Iceland ehf.) | 25 March 2026 | Up to **$20,000,000**; **$18,000,000 drawn** 24 April 2026 | SOFR + 4.25%; two-year initial term extendable to four; secured on the Iceland subsidiary's shares and designated GPU/CPU server assets; **guaranteed by WhiteFiber, Inc. and WhiteFiber AI, Inc.**; effective rate 10.64% at 30 June 2026 | 10-Q, 12 August 2026 |
| RBC amended facility | 27 April 2026 | CAD 28,000,000 (~$20,000,000); **CAD 24.7m (~$17.3m) drawn** 8 May 2026 | Funded the MTL-3 purchase; six-month term; repaid in full 15 July 2026 | 10-Q, 12 August 2026 |
| Syndicated RBC facility | 6 July 2026 | **CAD 115,000,000 (~$80,800,000)** plus a CAD 25m accordion; **CAD 36.8m (~$26.2m) drawn** 15 July 2026 | Non-revolving; three-year term; interest-only to the first full quarter then 15-year amortisation; first-ranking security over substantially all assets and mortgages on MTL-2 and MTL-3 | 10-Q, 12 August 2026 |
| Bit Digital Capital delayed draw term loan (NC-1) | 20 May 2026 | **$100,000,000** available, may increase to **$150,000,000**; **$50m drawn** 26 May 2026 at 3% OID; a further **$30m drawn** 27 and 31 July 2026 | 9.5% initial rate with a step-down on NC-1 milestones; **MOIC 1.1x** at maturity; secured on 100% of Enovum NC-1 Topco; **effective rate 50.6%** at 30 June 2026 | 10-Q, 12 August 2026 |
| B. Riley facility | 26 May 2026 | **$20,000,000** | The Bit Digital note assigned to B. Riley; 90-day maturity; effective rate 50.6% | 10-Q, 12 August 2026 |

### 9.2 What Is Owed

At 30 June 2026 (Form 10-Q, 12 August 2026, in thousands):

| Item | Carrying amount |
|---|---|
| Convertible notes, net of $7,406 issuance costs and discount | **222,594** |
| Delayed Draw Term Loan Facility (short-term) | 29,306 |
| B. Riley Facility (short-term) | 19,436 |
| Iceland Facility - current | 9,000 |
| Iceland Facility - non-current | 8,630 |
| Royal Bank of Canada Facility | 16,834 |
| **Total term debt, net** | **83,206** |
| Total liabilities | **543,506** |
| Total assets | **880,892** |

Two figures in that table deserve emphasis. The **$230 million convertible raised $102.1 million of net cash** once the $120 million capped-call premium, discounts and expenses are deducted (Form 10-Q, 12 August 2026) - the company paid more than half the gross proceeds for a hedge, and the money available to build with is less than half the headline. And the **effective interest rate on the NC-1 construction loan is 50.6%**, not 9.5%: the 10-Q explains that this "exceeded the contractual interest rate due to the inclusion of the contractual MOIC payment" and "the short-term nature of the facility". A 90-day loan with a 1.1x MOIC is bridge financing at a bridge price, and it is being provided by the controlling shareholder's own subsidiary.

### 9.3 What Is Committed

- **NC-1 construction**: $322.5 million of development costs capitalised in the six months to 30 June 2026 alone, within a $343.7 million increase in colocation equipment net book value (10-Q, 12 August 2026).
- **Total investing outflow**: $318,630,000 in the six months to 30 June 2026, against operating inflow of $89,105,000 and financing inflow of $171,927,000. Cash fell from $114.4 million to $56.1 million.
- **Purchase commitments**: the filings disclose "multi-quarter purchase commitments for critical hardware such as GPUs, power distribution units, and networking equipment" (10-K, 26 March 2026) and "purchase orders with leading hardware manufacturers that include extended delivery schedules spanning several months". **No aggregate dollar amount for these commitments is disclosed in any of the three filings.** Section 15 records this as a gap. The nearest quantitative proxy is $33.4 million of deposits for property, plant and equipment at 30 June 2026 (down from $52.7 million at 31 December 2025), and $27.4 million of unpaid construction in progress sitting in other payables at 31 December 2025 (10-K, 26 March 2026).
- **Utility take-or-pay**: a minimum monthly Duke Energy bill at Rate $8,754 "irrespective of actual usage levels", plus $11,405 a month of facilities charges on $1,137,975 of installed extra facilities, secured by a $3 million surety bond (10-Q, 12 August 2026).

### 9.4 The Funding Gap

The company describes it better than any external analyst could, and the sentence should be read twice:

> **"Our development pipeline is capital intensive, and depending on construction costs, leasing pace, and financing market conditions, our existing capital resources may not be sufficient to fund both this pipeline and our near-term debt maturities without accessing additional capital, whether through indebtedness, equity or equity-linked securities, project-level financing, or otherwise."**
>
> - Form 10-Q, 12 August 2026, Liquidity and capital resources.

The supporting facts: **working capital moved from an $85.2 million surplus at 31 December 2025 to a $28.6 million deficit at 30 June 2026**; cash is $56.1 million; **$57.7 million** of debt (the $29.3 million net DDTL, the $19.4 million B. Riley note and the $9.0 million current Iceland tranche) matures inside twelve months; the company has "entered into exclusivity with a consortium of lenders in connection with a proposed secured financing for its NC-1 project" but "there can be no assurance that the financing will be completed on favorable terms or at all" (same filing).

Two mitigants are real and should be stated. **Operating cash flow was positive $89.1 million** in the half-year, and **adjusted EBITDA was $8.5 million** (Q2: $5.5 million). But the operating cash flow is flattered by a **$63.6 million increase in deferred revenue** - customer prepayments, which are cash today and revenue later, and which reverse as the services are delivered. Strip the prepayments and operating cash flow is roughly $25 million against $318.6 million of investing outflow. The business is not yet self-funding its build, and it says so.

> **Judgement:** this is a company funding a ten-year asset with a mixture of permanent equity, convertible debt at a 4.5% coupon and a 50.6% effective bridge, related-party construction finance, and customer prepayments. The capital structure is not distressed, but it is *stacked* - each layer shorter and more expensive than the one below it - and the bottom layer is a building that will not produce cash until a utility delivers power.

---

## 10. The Banking Angle: Underwriting the Contract

### 10.1 The Principle

A lender to this company is not underwriting accelerators. It is underwriting **a contract and a counterparty**, with the accelerator as a secondary, deteriorating, and hard-to-perfect source of recovery. The reason is not that GPUs are worthless; it is that their value is a function of a market that can move faster than any enforcement action. A mortgage on a building can be enforced in eighteen months and the building will still be there. A lien on 2,048 H200s can be enforced in eighteen months and the H200s will be worth what the market says in eighteen months, which is a different number and possibly a much smaller one.

The company's own financings show that the market already prices this. The Iceland facility is secured on GPU servers *and* the subsidiary's shares *and* is **guaranteed by the parent**. The RBC facilities are real-estate instruments - a three-year USD 18.5 million lease facility for equipment and building improvements, a USD 19.6 million real-estate term loan to refinance MTL-2, and then a syndicated facility with first-ranking mortgages over MTL-2 and MTL-3 (10-K, 26 March 2026; 10-Q, 12 August 2026). The NC-1 construction loan is secured on **the equity of the project company**, not on the GPUs. In every case, the lender reached for something other than the accelerator.

### 10.2 Four Exposure Routes

**Route 1 - Borrower whose collateral depreciates on a schedule.** Cymbal lends to WhiteFiber or a subsidiary. The collateral includes cloud service equipment carried at a five-year life, which means the bank's loan-to-value ratio *improves* on paper every month while the asset's market value may be falling faster. A borrowing base certificate that marks GPUs at book value is therefore close to useless as a risk measure: it will show headroom growing as the loan amortises, on an asset whose realisable value is set by a secondary market the bank cannot observe. The mitigant is not a lower advance rate; it is a **shorter tenor than the shortest contract**, or a cash-sweep tied to contract receipts.

**Route 2 - Counterparty on a take-or-pay contract.** Two versions. (a) Cymbal *buys* reserved capacity from WhiteFiber on a multi-year commitment: the bank has prepaid or committed to pay for capacity that must be delivered on a schedule, and the provider's ability to deliver depends on power, hardware and staff that the bank has not diligenced. (b) Cymbal *lends to a customer* who has signed such a contract: the bank's borrower now has a fixed, escalating, take-or-pay obligation, and the bank should treat it as debt in the borrower's cash-flow model whether or not it is on the balance sheet. Both versions turn on the same question: **what happens to the commitment if the capacity is not delivered, and what happens to the money if the provider fails?**

**Route 3 - Supplier to a regulated institution.** WhiteFiber as a vendor to a bank - private AI cloud, colo for a regulated workload. This is ordinary third-party risk with two unusual features: the service is **material** to whatever business process runs on it (a risk model retrained weekly is not a discretionary workload), and the provider is **single-site dependent** for the relevant region, with concentration in a small number of facilities and a small number of customers. The operational-resilience expectations that apply to a regulated institution's material service providers are owned by [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md); the concentration and fourth-party questions by [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md); the onboarding and ongoing-monitoring mechanics by [vendor_management_guide.md](../management/vendor_management_guide.md).

**Route 4 - Vendor in a concentration-risk register.** Even a bank that never lends to WhiteFiber and never buys from it may hold it in a concentration register, because the bank's *clients* are increasingly dependent on a small set of GPU providers. If three of the bank's corporate borrowers all train on the same provider's capacity in the same region, the bank has a correlated operational exposure that no single credit file shows. That is a portfolio question, and the register is the only place it can be asked.

### 10.3 What a Bank Cannot Get Comfort On

Stated plainly, because a credit committee needs the list rather than a reassurance:

1. **Residual value of a named GPU SKU at a named future date.** No filing, no rating agency and no exchange publishes a forward curve for H200s. The one disclosed disposal realised a gain; that is a single observation, not a curve.
2. **Utilisation.** The company discloses revenue and fleet descriptions, never utilisation. A 60% gross margin is consistent with 95% utilisation at a low price or 60% utilisation at a high one, and the two have completely different implications for a lender.
3. **The enforceability of a termination fee against a distressed counterparty.** The Initial Customer episode is the evidence: a 40% fee was contemplated, a fixed $12.3 million was agreed, it was amended to $15.7 million, and it is **still outstanding**. Contractual protection is a claim, not a receipt.
4. **Whether the power will be delivered.** "Commercially reasonable efforts" from a utility is not a covenant, and the company's own purchase-price contingency concedes the point.
5. **Whether Tier III is certified.** The filings say "meet the requirements of the standard"; the website says "Tier III designed". Neither is a certificate, and neither is a measured availability figure.
6. **Who the largest customer is.** A 63% concentration in an unnamed counterparty cannot be diligenced by a lender at all. This is not a criticism of the disclosure - issuers routinely withhold customer names - but it is a hard limit on what a third party can underwrite.
7. **The aggregate hardware purchase commitments.** Disclosed as existing, not disclosed as an amount.

### 10.4 Six Questions to Put in Writing Before Any Term Sheet

Each is answerable, or explicitly unanswerable, from the record examined here. The value of asking is that three of the six have no answer, and a credit file that records the absence is more honest than one that assumes a default.

1. **What is the remaining term of the assigned contract, and what is the notice period inside it?** A 24-month agreement renewing monthly has a 30-day floor, not a 24-month floor.
2. **Who is the counterparty, and can it be diligenced?** If the concentration is 63% in an unnamed customer, the answer is no, and the loan should be sized as if the concentration were 100% in an unrated name.
3. **Is the collateral book value or liquidation value, and who produced the liquidation estimate?** If the answer is book, the collateral schedule is an accounting document, not a credit document.
4. **Has the power been delivered, and what does the off-taker pay if it has not?** At NC-1 the answer is a minimum monthly bill, a facilities charge and a surety bond - a cost of waiting, not a remedy for it.
5. **Is the Tier III claim certified, and is the availability figure measured?** If neither, the SLA is a design intention and should be priced as one.
6. **What is the aggregate hardware commitment, and is any of it take-or-pay?** Undisclosed. Treat the absence as a reason to require a covenant rather than a reason to relax.

### 10.5 The One Thing a Bank Should Take From This File

**Size to the contract term and the customer's credit; treat the accelerator as a rounding error.** A loan to a colocation landlord with a 10-year Nscale contract and pass-through power is a different instrument from a loan to a GPU lessor with 12-month orders, even when both are made to the same legal entity on the same day. The consolidated balance sheet does not distinguish them. The segment note does. The segment note is where the credit analysis begins.

---

## 11. Five Failure Modes for This Class of Company

### 11.1 The Five Modes

Each of these is a structural failure mode for GPU-cloud-plus-colocation businesses generally, not a prediction about WhiteFiber specifically. Where the filings contain evidence bearing on one, it is cited. Where they do not, that is said.

| # | Failure mode | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **Contract shorter than asset life** | Revenue is renewed every quarter rather than contracted; gross margin depends on a utilisation number nobody publishes; a single non-renewal takes a double-digit percentage off revenue with no impairment to explain it | The asset is sold on a 3 to 5-year schedule and rented on a 6 to 24-month term; the gap is bridged by an assumption of renewal that is never written down | Match tenor to the shortest material contract, not the longest; require a cash sweep on contract receipts; disclose renewal rates, not just backlog |
| 2 | **Concentration that reads as a pipeline** | One customer is 60-70% of revenue and also the entire growth story; the backlog looks healthy because it is one contract; the equity story and the credit story depend on the same counterparty | Customer concentration is normal in a young market and rational for the seller, because a single anchor tenant justifies the build. The filing discloses the percentage but not the identity, so the market cannot price it | Cap single-counterparty revenue at a board-approved limit; require contracted backlog to be disclosed with counterparty credit grades internally even when not publicly; test the model against the loss of the top customer this year, not next |
| 3 | **Power contracted but not delivered** | A capacity target is announced, then revised down; a site is built before the utility confirms; the company pays a minimum bill and posts a bond for power it is not using | Utilities commit to "commercially reasonable efforts"; the developer needs the site to exist before the utility will study it; the sequence (site, power, building, customer) cannot be reordered | Treat power as a condition precedent to capex, not a parallel workstream; disclose the delivery milestone and the consequence of missing it; never let a purchase price or a lease start date assume uncommitted power |
| 4 | **Colocation sold against density the building cannot cool** | A density figure appears in marketing that is higher than the density in the engineering design; tenants arrive with racks the cooling loop cannot serve; the SLA is met on paper and the customer's workload throttles | Density is the product, and marketing writes the aspiration while engineering writes the constraint. Air-cooled retrofit shells have a hard thermal ceiling; direct-to-chip helps but needs a facility water loop that older buildings lack | Publish the supported density per hall, not per portfolio; commission a thermal test at the sold density before the lease starts; align the SLA to sustained performance, not to power availability |
| 5 | **An uptime SLA measured on something other than what the customer cares about** | The dashboard reports 99.99% and the customer's training job still fails; credits are calculated on facility availability while the loss is a week of GPU time | Availability is easy to measure at the facility boundary (power present, network reachable) and hard to measure at the workload boundary (job completed, checkpoint intact). The provider measures what it controls | Define the SLA at the workload: job completion rate, time-to-first-token under load, replacement-GPU time. Measure what the customer's finance team already counts |

### 11.2 Each Mode, Against This Issuer's Own Numbers

**On mode 1.** The five-year life is disclosed; the contract terms are disclosed; the ratio is an exercise in arithmetic that Section 5.5 already performed. What makes the mode live rather than theoretical is that the *financing* on the accelerators also runs three years - the two sale-leaseback arrangements sold and leased back 96 and 128 servers for three years respectively (10-K, 26 March 2026). Three parties to the same asset have set three different horizons: the book at five years, the customer at 12 to 24 months, the lessor at three years. Nothing in the filings reconciles them, and the shortest of the three is the one that has been tested.

**On mode 2.** The issuer has now run this experiment twice inside eighteen months. In FY2024, 96.6% of revenue came from one customer. In FY2025, 70.7%. By the first half of 2026, 63% from a *different* largest customer. That is not a diversification trend; it is the same structural condition with different names attached, and the movement from 96.6% to 63% reads as improvement only if you do not notice that the denominator changed with the colocation segment arriving.

**On mode 3.** The capacity target is the cleanest symptom in the record: 76 MW (gross) by the end of Q4 2026 in the March 2026 filing, 70 MW in the August 2026 filing, with the same three supporting assets named and one of them - MTL-2 - disclosed as deprioritised. A target that moves down while the capital budget moves up is the signature of this mode, and the countervailing evidence is genuinely strong: MTL-3 went from lease signature (11 April 2025) to operational (November 2025) in seven months, which is a real capability, not a claim.

**On mode 4.** This is the one mode where the record contains a contradiction rather than a gap. The 10-K describes enterprise customers served at "upwards of 50kW per cabinet" and its Iceland cloud site at "45 kW rack density"; the marketing page advertises "up to 150 kW per cabinet". Two of those three numbers describe what has been built; the third describes what is being sold. A tenant who contracts on the third and is provisioned on the first has bought a different product, and the dispute will be about cooling capacity rather than about uptime - which is why the failure surfaces in the contract, not in the SLA.

**On mode 5.** The three availability numbers in this record (99.982%, 99.95%, > 99.5%) are all facility-side or lease-side figures. None of them is a statement about whether a training job completes. That is not a criticism peculiar to this issuer: the entire category measures availability at the boundary it controls. It is, however, the reason a bank buying reserved capacity should write its own acceptance test rather than accept the provider's availability metric, because the provider's metric will be green on the day the bank's workload fails.

Two observations on the table as a whole.

**Failure modes 1 and 2 are the same failure seen from different angles.** Concentration is dangerous precisely because the contract is short: if the contract were ten years, a 70% customer would be a strength. The company's move from a 70.7% cloud customer to an $865 million ten-year colocation customer is a move from a bad version of concentration to a good one, and the distinction is entirely about term.

**Failure modes 3, 4 and 5 all live in the gap between what is sold and what is engineered.** They are not financial failures; they are failures of the sales-to-engineering handoff, and they show up first in customer disputes and only later in the accounts. For a bank, they matter because they are the mechanisms by which a contracted revenue stream becomes a contested one - and a contested receivable is worth less than a performing one, as the $15.7 million outstanding termination fee demonstrates.

---

## 12. The Cymbal Bank Worked Example

**This section is fictional and illustrative.** Cymbal Bank is the only institution in it, the counterparty is a composite, and no figure in it is a real term sheet. It exists to show what the two-clock view changes in a live decision.

### 12.1 The Situation

Cymbal Bank's wholesale credit team is asked to approve two things in the same week, from the same prospective client - a regional AI-services company that has signed a 24-month reserved-capacity agreement with a GPU provider and needs to finance the associated commitments.

- **Facility A**: a $40 million three-year term loan, secured by a first-ranking charge over 2,048 current-generation accelerators that the borrower will place with the provider, plus an assignment of the service contract.
- **Facility B**: a $12 million revolving facility to fund working capital against the provider's monthly invoices.

The relationship manager's memo argues that Facility A is well secured: the accelerators cost $96 million, so a $40 million loan is 42% loan-to-value, and the equipment is "current generation". The memo attaches the provider's marketing page, which states an average 99.95% uptime.

### 12.2 What the Two-Clock View Changes

**The collateral analysis collapses on inspection.** The 42% LTV is computed against cost, on an asset with a five-year book life and a 24-month contract. At the end of the contract the asset is 60% undepreciated on the borrower's books and worth whatever the secondary market says - and the provider's own disclosed disposal is the only data point available, which is one transaction, not a curve. Worse, the "first-ranking charge" is over equipment sitting in a third party's building, which the bank cannot inspect without the provider's cooperation and cannot remove without the provider's consent. The security is, in practice, a contractual right against a company the bank has not diligenced.

**The contract analysis is the real analysis.** The 24-month reserved-capacity agreement has an initial term of 24 months and then renews monthly. The borrower's ability to service a three-year loan therefore depends on a renewal that occurs **twelve months after the loan matures its first amortisation step**, and on a provider whose own filings show that its largest customer was 70.7% of revenue and then paused, and that the termination fee it negotiated was renegotiated and is unpaid.

### 12.3 The Decision

**Facility A is declined as structured.** The credit committee's reasons, in its own words:

1. The bank will not take accelerators as collateral at carrying value. It will lend against a **liquidation estimate**, and the only defensible liquidation estimate available is bounded by the shortest contract in the structure - which, on a 24-month agreement renewing monthly, is close to zero.
2. The bank will size to **the contract term and the customer's credit**, not to the asset. A 24-month contract supports a 24-month loan with a balloon, not a three-year amortising facility.
3. The bank requires the provider's **counterparty risk** to be diligenced as a credit exposure in its own right, because the borrower's revenue depends on the provider continuing to operate the facility, and the provider's own filings disclose a working capital deficit, a 50.6% effective rate on its construction bridge, and a stated doubt about funding its pipeline and its near-term maturities.

**Facility B is approved with conditions.** The revolver is approved at $8 million against assigned invoices, with a 60-day dilution reserve, a requirement that the borrower's contract include a **termination-for-convenience right and a step-in right** in the bank's favour on provider default, and a covenant requiring the borrower to hold no more than 50% of its committed capacity with any single provider.

### 12.4 The One Decision the Two-Clocks View Changed

**The bank refused to lend against the accelerator at carrying value, and repriced the entire facility onto the contract and the counterparty.**

Before the two-clock analysis, the deal looked like secured equipment finance: 42% LTV, current-generation hardware, a named provider. After it, the deal looked like what it was - an unsecured loan to a services company, collateralised by a contractual position in a counterparty the bank had not diligenced, with a maturity mismatched to the only contract that paid it. The loan was not declined because the collateral was bad. It was declined because **the collateral was on a different clock from the loan**, and no advance rate fixes a maturity mismatch.

The residual point is the one the committee wrote into the file: *if we would not lend to the provider on the strength of its own balance sheet, we should not lend to its customer on the strength of its equipment.*

---

## 13. Competitive and Category Context

### 13.1 The Four Categories, Structurally

This section describes the structure of the market WhiteFiber operates in. It contains **no market share, no ranking, and no revenue comparison against any named company.** The categories are distinguished by what they own and what they sell, which is the only distinction that matters for the thesis.

| Category | Owns the building? | Owns the silicon? | Sells | Clock it runs on |
|---|---|---|---|---|
| **Hyperscalers** | Yes | Yes | A general cloud estate plus AI capacity inside it | Building 20-30 years, silicon 3-6 years, but the silicon is a small fraction of a much larger depreciable estate |
| **Specialist GPU clouds (neoclouds)** | Sometimes; often leases | Yes, or leases the metal | GPU-hours, reserved clusters, bare metal | Silicon 3-6 years; contracts 1-3 years; the mismatch is the business model |
| **Colocation REITs and wholesale developers** | Yes | Never | Space, power, cooling, interconnection | Building 30+ years; contracts 5-15 years; the asset outlives the contract comfortably |
| **Crypto-to-HPC converters** | Often yes, retrofitted | Sometimes | Converted capacity, power, and increasingly GPU capacity | Legacy electrical infrastructure, new thermal envelope, contract terms inherited from a different industry |

WhiteFiber is a **hybrid of categories two and three**: it owns buildings and sells colocation (category three), and it owns and leases accelerators and sells GPU capacity (category two). That is the structural reason the thesis applies to it more sharply than to either pure play. A colocation REIT never has to answer the obsolescence question; a pure neocloud never has a building to fall back on. WhiteFiber has to answer both, on one balance sheet, with one treasury, and - critically - with a single consolidated gross margin that mixes the two.

### 13.2 Where the Hybrid Helps

Three structural advantages are visible in the filings and are not marketing.

**It removes a layer of margin from the cloud business.** The 10-K makes the argument directly: "our vertically integrated solution allows us to capture additional margin for both our data center and cloud services businesses, avoiding expenses that would otherwise be due to third-party providers" (26 March 2026). The cloud segment's $5,410,230 of datacenter lease expense is 7.9% of cloud revenue; a pure neocloud leasing all its capacity would carry a larger line. That is a real cost advantage, and it is the reason the segment can report a 61% pre-depreciation margin.

**It shortens time to revenue on a new site.** A retrofit in six months means a building can be earning rent before a greenfield competitor has finished permitting. In a market where the constraint is power and time rather than demand, that is the operative advantage.

**It gives the cloud business somewhere to go when a contract ends.** The redeployment of the Initial Customer's GPUs to three other customers is easier for a company that also operates buildings, because it can place capacity without renegotiating a third-party lease.

### 13.3 Where the Hybrid Hurts

**It puts a low-return asset and a high-return asset in the same capital allocation queue.** $556.3 million of colocation equipment against $95.8 million of cloud equipment at 30 June 2026 (10-Q, 12 August 2026) is a statement about where the capital went. The colocation build is what consumes the cash; the cloud business is what produces the current revenue. A company with one treasury must choose between them every quarter, and the 10-Q discloses that it has: MTL-2, a cloud-adjacent build, was deprioritised in favour of "more time sensitive projects" - which is to say NC-1.

**It creates a conflict of interest inside the P&L.** When the cloud segment leases capacity from the colocation segment, the consolidated accounts eliminate the intercompany rent (the Q2 2026 segment note shows $26 thousand of intersegment revenue and cost of revenue, eliminated). That is correct accounting and it obscures transfer pricing. A reader cannot tell whether the cloud segment is paying a market rent for the space it uses inside WhiteFiber's own buildings, and therefore cannot tell whether either segment is genuinely profitable at arm's length.

**It makes the consolidated margin uninformative.** As Section 2 argued, a 60% blended gross margin that mixes landlord economics with lessor economics, excludes depreciation, and includes $14.7 million of GPU lease expense describes no single business.

### 13.4 The Competitive Facts the Filing Discloses, Without Comparison

The 10-K names its competitors: Digital Realty, Equinix, NTT, CyrusOne, STACK Infrastructure, Aligned Data Centers and Iron Mountain in data centres, and CoreWeave, Crusoe Energy, Nebius and Lambda Labs in cloud services (26 March 2026). It then states the competitive disadvantage in its own words: competitors may have "greater name recognition, longer operating histories, pre-existing relationships with global developers, utilities and local authorities, which may give them an advantage in securing real estate or power for new sites, particularly in constrained regions", and "more significant marketing budgets and other financial and operational resources" (same filing).

That is the honest structural position: a small entrant in a capital-intensive market where the binding constraints are relationships with utilities and access to capital, competing against firms that have more of both. The filing says so. This guide adds no comparison, because a comparison would require market data that is not in the record and would not change the structural conclusion.

> **Judgement:** the category structure explains both why the hybrid exists (margin capture, speed, redeployment optionality) and why it is fragile (one treasury, two clocks, a blended margin that conceals which business is paying for which).

---

## 14. The State of Practice

### 14.1 What Is Disclosed, and Well

This filing set is better than average for a company of this size, and the credit belongs in the record before the criticism.

- **Segment economics are disclosed at the line-item level.** Note 15 gives electricity, datacenter lease, GPU lease, wages, support contractors and other items, by segment, for two years. That is more granular than most issuers provide and it is what makes Section 2 of this guide possible.
- **The depreciation change is disclosed, quantified, and explained.** $10.0 million of expense, $7.9 million of net benefit, $0.25 per share, in the accounting policies and again in MD&A.
- **Customer concentration is disclosed as a percentage in every filing**, including the awkward ones - 96.6% in FY2024, 70.7% in FY2025, 63% in H1 2026.
- **The Initial Customer's deterioration is narrated in sequence**, including the guarantee being replaced by a deposit, the pause, the negotiation, the termination, the bad debt, and the unpaid fee. A company wanting to obscure this could have written two sentences. It wrote a page.
- **The purchase-price contingency on power is disclosed** with its three thresholds and its per-MW escalator.
- **The funding doubt is stated in the company's own words** rather than left to inference.
- **Remaining performance obligations are tabulated by year and segment**, with the ASC 606 exclusion noted.
- **Contingent liabilities and the Duke Energy take-or-pay are itemised**, down to the monthly facilities charge.

### 14.2 What Is Claimed

- **Tier III compliance** - "meet the requirements of the Tier-3 standard" in the filings, "Tier III designed" on the website. Neither is a certification and neither is a measurement.
- **99.982% uptime and no more than 1.6 hours of downtime annually** - stated as a property of the facilities, not as a reported result. No measurement period, no methodology, no third-party attestation is disclosed.
- **"> 99.5%" service lease uptime** for cloud, and **"AVERAGE 99.95%"** on the marketing page. Three different numbers, no reconciliation.
- **"Among the first service providers to offer H200, B200, and GB200 servers"** - a claim about a race whose result is not verifiable from outside and does not matter commercially.
- **"1.3GW energy pipeline"** on the website against "approximately 1,500 MW (gross) of potential data center projects under management review" in the filing. The website converts a screening list into a pipeline.
- **"Up to 150 kW per cabinet"** on the website against "upwards of 50kW per cabinet" in the 10-K.
- **"Latest generation of NVIDIA GPUs, including B300, GB300, and VR200"** on the website in November 2026; the FY2025 10-K names H200, B200 and GB200 as of March 2026. Both may be true at their respective dates; the marketing page is the only source for the newer SKUs.

### 14.3 What Is Not Knowable From Outside

The honest boundary, itemised:

1. **Price per GPU-hour, by SKU and by contract type.** Never disclosed. Revenue is aggregate.
2. **Utilisation.** Never disclosed, in any filing. This is the single largest gap: a GPU business's margin is a utilisation function, and the reader cannot see the input.
3. **Segment depreciation.** Total D&A is disclosed; the split between cloud and colocation is not. It is therefore impossible to compute a true segment operating margin, only a pre-depreciation one.
4. **Residual values, by vintage.** The H200 disposal is one data point; there is no schedule of fleet by age, and no disclosure of what the redeployed Initial Customer GPUs are earning.
5. **The identity of the largest customer**, at 63% of H1 2026 revenue. The Nscale agreement is the only contract large enough to plausibly account for it, but the filings do not connect the two, and this guide does not assert the connection as fact.
6. **The aggregate hardware purchase commitments.** Disclosed as existing; no amount.
7. **The commercial terms of the Nscale contract beyond the headline** - the escalator rate, the pass-through definition, the termination provisions, the credit support.
8. **Whether any facility holds a Uptime Institute certification**, as opposed to a design claim.
9. **The terms of the Transition Services Agreement** with Bit Digital, beyond its existence and its date.
10. **The price at which capacity was re-let** after the Initial Customer pause, and therefore whether redeployment was value-preserving or merely revenue-preserving.

### 14.4 Where the Practice Sits

Against the mature end of this category - colocation REITs that publish lease-expiry schedules and tenant concentration by grade, and utilities that publish contracted capacity and delivery dates - this is an early-stage disclosure set: rich on segment cost lines and narrative, thin on the operational metrics that would let a reader model the business. Against the emerging end - neoclouds that disclose revenue and nothing else - it is unusually forthcoming.

The gap that matters most is not a disclosure failure. It is structural: **the metrics that decide whether this business works (utilisation, price per GPU-hour, residual value, re-let price) are the metrics no GPU-cloud issuer discloses, because disclosing them would reveal the pricing to competitors and the utilisation to customers.** A reader should expect that gap to persist, and should price it rather than wait for it to close.

---

## 15. The Claims Audit

Every row carries a source, a date, and a kind. **DOCUMENTED FACT** = stated in a filing with a date. **COMPANY CLAIM** = the company's own assertion about a capability or an outcome, not independently verified. **MARKETING** = a promotional statement on a company web property. **THIRD-PARTY** = derived from a source other than the company. Grades: ✅ verified at primary source; ⚠ flagged, unverified, or promotional; ❌ rejected or contradicted by the record.

| # | Claim | Kind | Source and date | Grade | Note |
|---|---|---|---|---|---|
| 1 | WhiteFiber, Inc. is a Cayman Islands exempted company; ordinary shares $0.01 par; Nasdaq: WYFI; CIK 0002042022 | DOCUMENTED FACT | 10-K, 26 March 2026; EDGAR metadata, retrieved November 2026 | ✅ | Identity established. |
| 2 | Incorporated 19 October 2023 in Delaware as Bit Digital AI, Inc. (later WhiteFiber AI, Inc.) | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Operating subsidiary, not the listed parent. |
| 3 | Listed parent incorporated 15 August 2024 as Celer, Inc. (Cayman); renamed WhiteFiber, Inc. 17 October 2024 | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | The two incorporation facts are often conflated; they are different entities. |
| 4 | Former name "White Fiber, Inc." current 6 January 2025 to 11 July 2025 | DOCUMENTED FACT | EDGAR submissions metadata, retrieved November 2026 | ✅ | Not in the filing text; recorded as registry metadata. Reason for the spacing change not established. |
| 5 | Enovum acquired 11 October 2024, bringing MTL-1 (4 MW gross, Tier-3, fully leased) | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Basis of the colocation segment. |
| 6 | Contribution Agreement effective 6 August 2025; 27,043,749 shares issued to Bit Digital; S-1 File No. 333-288650 | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Pre-6 August 2025 statements are on a combined basis. |
| 7 | IPO completed 8 August 2025: 9,375,000 shares at $17.00, gross $159,375,000 | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Over-allotment of 1,406,250 shares on 2 September 2025 added $23,906,250. |
| 8 | Bit Digital holds approximately 70.5% at 31 December 2025; controlled company under Nasdaq rules | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | 71.5% (and 71% in one passage) immediately post-IPO. |
| 9 | FY2025 revenue $79,164,252 (cloud $68,753,609; colocation $8,913,816; other $1,496,827) | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Segment note. |
| 10 | FY2025 segment gross profit $47,630,527 (cloud $42,167,246; colocation $5,463,281) | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Excludes depreciation by policy. |
| 11 | FY2025 GPU lease expense $14,741,928, all in the cloud segment | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | 21.4% of cloud revenue. The segment is partly a lessee. |
| 12 | FY2025 depreciation and amortisation $23,440,884 (FY2024 $16,511,406) | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Stated separately from cost of revenue. |
| 13 | Cloud service equipment depreciated over 5 years; colocation equipment 10-15; building 30; leasehold improvements 15; software 1-5 | DOCUMENTED FACT | 10-K (Note 2), 26 March 2026 | ✅ | The analytical core of Section 5. |
| 14 | Useful life for cloud service equipment changed from 3 to 5 years effective 1 January 2025 | DOCUMENTED FACT | 10-K (Note 2), 26 March 2026 | ✅ | Quoted verbatim in Section 5.2. |
| 15 | The change reduced D&A by $10.0m and benefited net loss by $7.9m, or $0.25 per share, in FY2025 | DOCUMENTED FACT | 10-K (Note 2), 26 March 2026 | ✅ | Roughly 31% of the $25,768,370 pre-tax loss. |
| 16 | Initial Customer = 70.7% of FY2025 revenue and 96.6% of FY2024 revenue | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Customer unnamed. |
| 17 | DNA Fund = 11.5% of FY2025 revenue; nil in FY2024 | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | MSA terminated November 2025; $7.3m receivable, $2.1m collected. |
| 18 | Initial Customer under "agreed pause of services"; no definitive agreement at the 10-K date; GPUs redeployed to three other customers | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | The guide's central empirical test (Section 7). |
| 19 | Initial Customer termination in Q2 2026: $12.5m receivables preserved and collected; $2.2m bad debt; $12.3m termination fee recognised as revenue, amended to $15.7m and outstanding | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | Contractual protection is a claim, not a receipt. |
| 20 | Seven existing customers at the 10-Q date; largest = 63% of H1 2026 revenue; discontinued orders cost ~$5.1m; new orders of $635.8m contracted | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | Concentration migrated, not diversified. |
| 21 | Colocation: 15 customers at 31 December 2025; no customer above 50% of data centre revenue | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Contrast with the cloud segment. |
| 22 | Nscale agreement: ~$865m total contracted revenue over 10 years, inclusive of escalators and NRCs; power as pass-through; 40 MW phase billing from June 2026 | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Not yet revenue at the FY2025 year end. |
| 23 | Cerebras at MTL-3: 5 MW IT load; CAD 1.4m (~$979k) monthly; five-year contract from 1 November 2025 | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | MTL-3 purchase closed 8 May 2026. |
| 24 | Remaining performance obligations at 30 June 2026: colocation $932,895k; cloud $75,146k; total $1,008,041k | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | In thousands. Cloud RPO ends in 2028. Variable consideration excluded. |
| 25 | Pipeline target 76 MW gross by end Q4 2026 (10-K) revised to 70 MW (10-Q); 1,500 MW under management review | DOCUMENTED FACT | 10-K, 26 March 2026; 10-Q, 12 August 2026 | ✅ | The only forward capacity number, and it moved down. |
| 26 | NC-1 purchase price contingent: +$8m / +$5m for 99 MW within two / three years, plus $200k per MW above 99 MW to a $5m cap; Duke Capacity Agreement for 24 MW by 1 September 2025, 40 MW by 1 April 2026, 99 MW within four years of 16 May 2025 | DOCUMENTED FACT | 10-K, 26 March 2026; 10-Q, 12 August 2026 | ✅ | No 99 MW ESA received as of 30 June 2026. |
| 27 | Duke Energy ESA: minimum monthly bill at Rate $8,754 irrespective of usage; extra facilities $1,137,975 with an $11,405 monthly charge; $3m surety bond posted 15 July 2026 | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | A take-or-pay utility obligation at a site not yet at load. |
| 28 | 4.500% Convertible Senior Notes: $230.0m principal, due 2031, net proceeds $221.5m before a $120.0m capped-call premium; $102.1m net to the company | DOCUMENTED FACT | 10-K, 26 March 2026; 10-Q, 12 August 2026 | ✅ | Annual cash interest ~$10.4m. |
| 29 | Bit Digital Capital delayed draw facility: up to $100m (to $150m), 9.5%, MOIC 1.1x, effective rate 50.6% at 30 June 2026 | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | Related-party construction finance. |
| 30 | Working capital deficit $28.6m at 30 June 2026, from a $85.2m surplus at 31 December 2025; cash $56.1m | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | Company states existing resources "may not be sufficient". |
| 31 | H1 2026: operating inflow $89.1m, investing outflow $318.6m, financing inflow $171.9m | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | Operating inflow includes $63.6m of deferred revenue. |
| 32 | H200 servers with $24.3m carrying value sold in H1 2026, with a $1.8m net gain on disposal | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | Complicates the sceptical case on residual value. |
| 33 | $5,006,000 impairment of capitalised software assets in Q2 2026 on discontinuation of internally developed software operations | DOCUMENTED FACT | 10-Q, 12 August 2026 | ✅ | Realised obsolescence in the adjacent asset class. |
| 34 | Operational data centres "meet the requirements of the Tier-3 standard", including 99.982% uptime and no more than 1.6 hours of downtime annually | COMPANY CLAIM | 10-K, 26 March 2026; repeated 10-Q, 12 August 2026 | ⚠ | A design-topology assertion plus a design target. No certification, no measurement, no methodology. |
| 35 | Cloud service lease carries "Uptime percentage > 99.5%" | COMPANY CLAIM | 10-K, 26 March 2026 | ⚠ | Implies 43.8 hours of permitted downtime a year. No credit mechanics disclosed. |
| 36 | "AVERAGE 99.95% Uptime" | MARKETING | whitefiber.com/cloud/compute, retrieved November 2026 | ⚠ | 4.38 hours a year. 2.8x more permissive than the 99.982% design figure in the filing. |
| 37 | "Enterprise-grade SLAs" | MARKETING | whitefiber.com home page, retrieved November 2026 | ⚠ | Undefined. The filings disclose no SLA terms, no credit schedule, no remedy. |
| 38 | "Tier III designed facilities" / "Tier III designed data centers in North America" | MARKETING | whitefiber.com and /data-center/ai-hpc-colocation, retrieved November 2026 | ⚠ | Weaker than the filing's "meet the requirements". Both are design claims. |
| 39 | "Latest generation of NVIDIA GPUs, including B300, GB300, and VR200" | MARKETING | whitefiber.com/cloud/compute, retrieved November 2026 | ⚠ | The FY2025 10-K names H200, B200 and GB200 only. Different dates; newer SKUs are unevidenced in the filings. |
| 40 | "Multi-year reserved GPU capacity" / "multi-year reservation commitments on Blackwell and Grace Blackwell 300 class GPUs" | MARKETING | whitefiber.com home page, retrieved November 2026 | ⚠ | The disclosed book has a modal term of 12-24 months and several week-to-week orders. |
| 41 | "Accommodating up to 150 kW per cabinet" | MARKETING | whitefiber.com/data-center/ai-hpc-colocation, retrieved November 2026 | ⚠ | The 10-K describes "upwards of 50kW per cabinet" and 45 kW rack density at Blonduos. 3x discrepancy on the binding constraint. |
| 42 | "1.3GW energy pipeline" | MARKETING | whitefiber.com/data-center/ai-hpc-colocation, retrieved November 2026 | ❌ | The filing says ~1,500 MW "under management review" - a screening list, not a pipeline. |
| 43 | "NVIDIA Preferred Partner" | MARKETING | whitefiber.com, retrieved November 2026 | ⚠ | The 10-K describes an "authorized NVIDIA Preferred Partner through the NPN" relationship. Consistent; unverifiable from outside. |
| 44 | 83 full-time employees at 31 December 2025 | DOCUMENTED FACT | 10-K, 26 March 2026 | ✅ | Against $880.9m of total assets at 30 June 2026. |

### 15.2 What Could Not Be Verified

1. **The identity of the Initial Customer, and of the customer that was 63% of H1 2026 revenue.** No filing in the set names either. The Nscale agreement is the only contract large enough to plausibly account for a 63% share, but the filings never connect the two, and this guide does not assert the connection. A lender cannot diligence a counterparty it cannot name; that is a hard limit, not an oversight.
2. **Any measured uptime figure.** The record contains a design target (99.982%), a contractual floor ("> 99.5%") and a marketing average (99.95%). No filing reports availability achieved over any period, for any facility, by any method. Whether any WhiteFiber site holds an Uptime Institute certification - as distinct from a design claim - is not disclosed and could not be established.
3. **The aggregate value of hardware purchase commitments.** The 10-K states that "multi-quarter purchase commitments for critical hardware such as GPUs, power distribution units, and networking equipment" exist (26 March 2026) and that purchase orders carry "extended delivery schedules spanning several months". No dollar amount, no maturity profile, and no take-or-pay characterisation is given in any of the three filings. The nearest proxies - $33.4m of PP&E deposits and $27.4m of unpaid construction in progress - are not the same thing.
4. **Price per GPU-hour and utilisation, by SKU or in aggregate.** Never disclosed in any filing. This is the largest single hole in the analysis: the cloud segment's margin is a function of utilisation, and the input is invisible.
5. **Segment-level depreciation.** Total D&A is disclosed ($23.4m FY2025; $13.0m H1 2026) but not split between cloud and colocation, so no true segment operating margin can be computed by an outsider. Section 2's arithmetic is therefore a consolidated charge-back, not a segment allocation.
6. **Residual values by vintage, and the price achieved on redeployment.** The H200 disposal produced a $1.8m gain on $24.3m of carrying value - one transaction. What the redeployed Initial Customer GPUs earn, and at what rate, is not disclosed.
7. **The commercial terms of the Nscale agreement beyond the headline** - escalator rate, pass-through definition, termination provisions, credit support, and whether the $865m is inclusive of the 40 MW phase only or the full contracted capacity.
8. **The terms of the Transition Services Agreement with Bit Digital** (dated 30 July 2025), beyond its existence, its date, and the fact that consulting costs are charged to WhiteFiber under it.
9. **The 1,500 MW pipeline's composition.** No site list, no stage gates, no power status per site, and no capital cost estimate. The 76-to-70 MW revision is the only evidence that the pipeline is being marked to reality.
10. **Any third-party verification of the Tier III claim, the SOC 2 Type 2 report, or the 99.982% figure.** The SOC 2 report is referenced; it is not public, and its scope (controls, not availability) is not described.

---

## 16. Glossary and Closing Summary

### 16.1 Glossary

| Term | Definition as used in this guide |
|---|---|
| **Bare metal** | A whole physical server rented to one customer, with no hypervisor between the customer and the hardware. The highest-performance, lowest-multi-tenancy unit of GPU rental. |
| **Capped call / zero-strike call** | An option structure bought by an issuer alongside a convertible note to offset dilution above a strike. WhiteFiber paid $120.0m for one; it reduces net proceeds and does not reduce the debt. |
| **Committed capacity** | Capacity a customer has contracted to take, usually measured in kW or MW, as distinct from capacity consumed. The basis of colocation billing. |
| **Concurrent maintainability** | The property that any capacity component or distribution path can be removed for planned maintenance without affecting the load. The substance of a Tier III claim. |
| **Conversion rate** | The number of shares issuable per $1,000 of convertible principal. WhiteFiber's is 49.2126, implying ~$25.91 per share. |
| **Deferred revenue** | Cash received from customers for services not yet delivered. A liability that funds the business today and reduces revenue later. $143.1m of contract liabilities at 30 June 2026. |
| **Density (kW per rack / cabinet)** | The electrical load a rack position can support. The binding constraint on how much a colocation building can sell, and the number most often inflated in marketing. |
| **Gross MW vs IT MW** | Total facility demand including cooling and losses, versus the load that reaches customer equipment. The ratio between them is the facility's efficiency. |
| **MOIC** | Multiple on invested capital. A return floor paid to a lender at maturity on top of interest. The NC-1 bridge carries a 1.1x MOIC. |
| **Non-recourse** | A loan secured only on a project's assets, with no claim on the parent. The original RBC facility was described as non-recourse to the Company. |
| **Pass-through charge** | A cost, usually electricity, recovered from the customer at cost rather than embedded in the rent. It removes commodity risk from the landlord and from the landlord's lender. |
| **Remaining performance obligation (RPO)** | Contracted revenue not yet recognised. The closest thing to a backlog. Excludes variable consideration, which is why the cloud figure understates the cloud book's true contracted value. |
| **Reserved capacity** | A commitment to take a defined quantity of capacity for a defined period at a defined price. What makes capital expenditure financeable. |
| **Retrofit** | Converting an existing industrial building with in-place power into a data centre, rather than building greenfield. WhiteFiber's stated model, with a target of six months from construction start. |
| **Sale-leaseback** | Selling an asset and leasing it back. Used by WhiteFiber for GPU servers, converting owned metal into an operating lease expense and cash. A market signal about residual value: the lessor takes the residual risk for the lease term. |
| **Take-or-pay** | An obligation to pay for a committed quantity whether or not it is consumed. Seen at NC-1 as a utility minimum monthly bill, and in cloud as minimum purchase commitments plus termination fees. |
| **Tier III** | A design topology with concurrent maintainability and N+1 redundancy. Not a certification, not a measurement, and not a synonym for "reliable". |
| **Useful life** | The period over which an asset is depreciated. An estimate, reviewed annually, and - at WhiteFiber - changed from three to five years for cloud equipment effective 1 January 2025. |

### 16.2 The Guide in One Page

**The company.** WhiteFiber is a Cayman-incorporated, Nasdaq-listed AI-infrastructure company, 70.5% owned by Bit Digital, with 83 employees, $79.2 million of FY2025 revenue, and $880.9 million of assets at 30 June 2026. It reached the public market by a Section 351 contribution in August 2025 rather than by a conventional IPO of an independent company, which means its pre-2025 comparatives are combined, not consolidated.

**The two segments.** Cloud services (87% of FY2025 revenue, 61% pre-depreciation gross margin) rents accelerators. Colocation (11% of revenue) rents space, power and cooling. The margin comparison is misleading: cloud excludes depreciation by policy, includes $14.7 million of GPU lease expense, and is partly asset-light. The two segments are different asset classes sharing one treasury.

**The two clocks.** Accelerators are depreciated over five years - extended from three in the year of the IPO, worth $10.0 million of expense and $7.9 million of pre-tax loss. Buildings are depreciated over thirty. Contracts sit between them: cloud at a modal 12 to 24 months, colocation at 5 and 10 years. The cloud RPO of $75.1 million ends in 2028. The colocation RPO of $932.9 million runs past 2030.

**The test.** The Initial Customer was 70.7% of FY2025 revenue and 96.6% of FY2024 revenue. It paused, then terminated. The GPUs were redeployed to three other customers, H200s were sold at a gain over book, a $2.2 million bad debt was taken, and a $15.7 million termination fee remains outstanding. The asset survived; the revenue did not, cleanly. The concentration then migrated to the colocation segment as an $865 million ten-year contract - same concentration, better clock.

**The constraint.** The company owns buildings, buys power, and leases cloud capacity. The power is the part it does not control: the NC-1 purchase price is contingent on 99 MW arriving, the Duke agreement is "commercially reasonable efforts", and no 99 MW agreement existed at 30 June 2026 while a minimum monthly bill and a $3 million surety bond were already in place.

**The capital.** $159.4 million of IPO gross, $23.9 million of over-allotment, $102.1 million of net cash from a $230 million convertible after a $120 million capped call, $83.2 million of term debt at 30 June 2026, and a 50.6% effective rate on related-party construction bridge finance. Working capital moved from an $85.2 million surplus to a $28.6 million deficit in six months. The company states that its resources "may not be sufficient" to fund the pipeline and near-term maturities together.

**The banking conclusion.** Underwrite the contract and the customer. Do not underwrite the accelerator at carrying value, because its value is a function of a market that moves faster than enforcement, and because a five-year book life against a 24-month contract means the collateral is only as good as the renewal. The one decision the two-clock view changes is the advance rate: it is not a number, it is a maturity.

**The honest boundary.** Utilisation, price per GPU-hour, segment depreciation, residual values by vintage, the identity of the largest customer, and the aggregate hardware commitments are all undisclosed. A reader can establish the shape of this business and cannot model its margin. That is the state of practice for the category, not a failing peculiar to this issuer.

### 16.3 Closing

The evidence supports the thesis with one important qualification. Colocation is landlord economics, and the filings prove it: a ten-year contract with pass-through power, a building on a thirty-year life, and a lender amortising on a fifteen-year schedule against a three-year loan. Cloud is a lessor of a fast-depreciating asset, and the filings prove that too: sixteen contracts, most under two years, against a five-year book life that was extended in the year of the listing and is worth nearly a third of the reported loss.

The qualification is that the cloud segment is not a pure owner. It leases a fifth of its cost base, it excludes depreciation from its margin, and it has demonstrated - once, at a gain - that a redeployed accelerator retains value. That makes the five-year life defensible in a way a pure-owner model would not. What it does not change is the clock. A fully depreciated GPU that still earns is a good outcome; it is also an asset whose earning power is renegotiated every twelve months, and a business that must re-win its revenue annually cannot be underwritten on the same terms as one that has already won it for a decade.

The colocation backlog is now twelve times the cloud backlog, and the concentration has followed the contracts into the segment where the clock is long. That is the company voting with its own balance sheet on which business it believes in. Whether it can fund the vote is a separate question, and the answer at 30 June 2026 was that it would need more capital, on terms it had not yet obtained, against power it had not yet been promised.

WhiteFiber is two businesses on one balance sheet - a building that depreciates over decades and an accelerator that depreciates in years - and the contract between them is the whole investment case.
