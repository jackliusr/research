# Carbon Footprint Management Market: The Carbon-Management Software Industry — A Comprehensive Guide

**The Measured Footprint — Market Overview, the Regulatory Drivers (CSRD, SEC, ISSB, TCFD, SGX/MAS, SBTi), the Market Segments, the Key Players (Enterprise Suites, Carbon-Native, Removal & Verification), the Technology Stack, the Carbon Markets, the Banking Angle (PCAF, NZBA, MAS), and a Cymbal Bank Financed-Emissions Platform-Selection Worked Example**

> **Author:** Jack Liu Shurri — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — the Carbon-Footprint-Management market deep-dive: the definition and market overview, the regulatory drivers (the CSRD, the SEC climate rule, the ISSB IFRS S2, the TCFD, the SGX/MAS climate mandate, the SBTi), the market segments (enterprise Scope-1/2/3 accounting, SME tools, product carbon footprints, ESG-reporting platforms, carbon-credit platforms), the key players (enterprise suites, carbon-native platforms, removal/verification), the technology stack (emissions factors, data integration, calculation engines, AI, assurance), the carbon markets (compliance and voluntary), the banking angle (financed emissions, the NZBA, MAS green finance, green loans), a platform-selection worked example, one-page summary, glossary, and a verification ledger (verified vs flagged)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the regulatory material (EUR-Lex — Directive (EU) 2022/2464; the European Commission corporate-sustainability-reporting pages; SEC.gov press releases and statements; IFRS.org — IFRS S2; the FSB TCFD final report of 29 June 2017; the ACRA–SGX RegCo joint media release of 28 February 2024; sciencebasedtargets.org), the market research (Verdantix Green Quadrant: Enterprise Carbon Management Software 2026 — via press; the analyst market-size figures from TBRC, Grand View Research, Fortune Business Insights — flagged as inconsistent), the press (IBM Newsroom — the Envizi acquisition, 11 January 2022; UNEP FI — the NZBA launch, April 2021; ICAO/EASA — Resolution A39-3, 2016), the player sites (watershed.com, persefoni.com, salesforce.com, microsoft.com, sweep.net, isometric.com, registry.isometric.com, pachama.com via press/trackers, netzerocompare.com, tracxn.com, esgtechreport.com), and the GHG Protocol / PCAF published standards (carbonaccountingfinancials.com — the Global GHG Accounting and Reporting Standard for the Financial Industry)
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — technology/):** [Cedars Digital](cedars_digital_company_guide.md) (the Singapore AI carbon-footprint-verification company — the ISO/IEC 42001 + AI-carbon-verification angle, cross-ref §5.4), [Remote Sensing Technologies](remote_sensing_technologies_guide.md) (the satellite/LiDAR/SAR forestry-carbon monitoring — the Pachama-style MRV angle, cross-ref §4.3 and §6), [AI-Native Companies](ai_native_companies_guide.md) (the AI-native-company definition and taxonomy — the frame for the AI-native carbon platforms Persefoni and Watershed, cross-ref §4.2 and §5.4), [Supply Chain Finance Technologies](../banking/supply_chain_finance_technologies_guide.md), [Data Fabric](data/data_fabric_guide.md)
> **Companion guides (banking/, management/):** [Tokenized Assets](../banking/tokenized_assets_guide.md) (the carbon-credit tokens — Toucan/KlimaDAO/Flowcarbon/Carbonplace — cross-ref §3.5 and §6), [RegTech](../banking/regtech_guide.md) (the ESG-reporting and regulatory-technology angle — cross-ref §3.4), [Supply Chain Finance](../banking/supply_chain_finance_guide.md) (the supplier Scope-3 angle — cross-ref §3.1), [Banks in Singapore](../banking/banks_in_singapore_guide.md) (the MAS green-finance context — cross-ref §7.3), [Beyond Zero](../management/beyond_zero_guide.md) (the sustainability narrative — cross-ref lightly), [Business Case Development](../management/business_case_development_guide.md) (the platform-investment economics — cross-ref §8.5)

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The Regulatory Drivers](#2-the-regulatory-drivers)
3. [The Market Segments](#3-the-market-segments)
4. [The Key Players](#4-the-key-players)
5. [The Technology](#5-the-technology)
6. [The Carbon Markets](#6-the-carbon-markets)
7. [The Banking Angle](#7-the-banking-angle)
8. [The Worked Example — A Cymbal Bank Platform Selection](#8-the-worked-example--a-cymbal-bank-platform-selection)
9. [The Summary — One Page](#9-the-summary--one-page)
10. [The Glossary](#10-the-glossary)
11. [The Verification Ledger](#11-the-verification-ledger)

---

## 1. The Overview

### 1.1 The Carbon-Footprint-Management Definition

**Carbon footprint management** is the discipline of measuring, reporting, reducing, and (where residual) offsetting an organization's greenhouse-gas (GHG) emissions — and the **carbon-management software market** is the industry that sells the platforms automating that discipline. The software category goes by several overlapping names — carbon-accounting platforms, carbon-management software, ESG/sustainability-reporting suites, climate-tech SaaS — and the analyst firms define it with slightly different boundaries, but the functional core is stable:

- **Measure:** collect activity data (energy, fuel, travel, purchased goods, logistics) and convert it to CO₂-equivalent emissions using emissions factors and the GHG Protocol's Scope 1/2/3 methodology.
- **Report:** produce disclosures in the formats the regulators demand (CSRD/ESRS, ISSB IFRS S2, TCFD, SGX climate reports) with a defensible audit trail.
- **Reduce:** model abatement levers (efficiency, renewables, supplier engagement), set science-based targets, and track progress.
- **Finance/offset:** manage carbon credits, renewable-energy certificates (RECs), and removals for the residual footprint — the point where the software market meets the carbon markets (§6).

**Verification status:** the *concept* (measure-report-reduce-offset) is consistent across every source this pass could reach — the player sites (watershed.com "measurement, reporting, and action"; persefoni.com "measure, report, and decarbonize"), the analyst definitions, and the regulatory texts all describe the same loop. ✅ verified as a synthesis; there is no single canonical one-sentence definition to cite, so the wording above is this guide's synthesis of the consistent usage. Related terms: *carbon accounting* (the measurement/reporting core), *carbon management* (measurement + abatement + strategy), *ESG reporting* (the broader environmental-social-governance disclosure layer — cross-ref [regtech_guide.md](../banking/regtech_guide.md)). The distinction matters commercially: carbon-accounting pure-plays sell measurement depth; ESG-reporting suites sell the full disclosure stack; the enterprise cloud suites bundle both into their existing platforms.

### 1.2 The Market — Size and Shape (⚠ the figures are flagged)

The market-size question is the one every procurement deck asks first, and it is the one with the least reliable answer. This pass collected four analyst estimates for the carbon-accounting/carbon-management software market, and they do not agree:

| Source | Base-year value | Forecast | CAGR | Notes |
|---|---|---|---|---|
| The Business Research Company | US$27.78B (2026) | US$63.54B (2030) | 23.0% | ⚠ "Carbon Accounting Software" — the most aggressive base figure |
| Grand View Research | US$14.1B (2025) | US$67.6B (2033) | 21.9% | ⚠ lower base, longer horizon |
| Fortune Business Insights | ~US$14–17B (2023) | US$64.39B (2030) | 22.8% | ⚠ via press coverage |
| The Research Insights | US$16.92B (2023) | — (2030) | 22.1% | ⚠ "carbon accounting solutions" |

**The honest reading:** the *direction* is unanimous — a high-teens-to-twenties percent CAGR, multi-year doubling, a market that goes from the low tens of billions to the mid-sixties of billions within a decade — but the *level* is not. The base-year values for the same product category differ by ~2× across credible analyst houses, because they draw different boundaries (pure carbon-accounting SaaS vs ESG-reporting suites vs climate-services), because the category is young and bootstrapped companies disclose little, and because analyst methodology is opaque. **Flagged ⚠:** do not quote a single number in a business case; quote the range and the boundary disagreement, and let the regulatory pipeline (§2) do the real argument. Verdantix — the analyst most focused on this category — published its **Green Quadrant: Enterprise Carbon Management Software 2026**, naming Watershed a leader (watershed.com cites it); that report, not the market-size press releases, is the credible competitive benchmark. The structural driver is unambiguous and verifiable: regulation now *compels* measurement at a scale the voluntary market never reached — the CSRD alone pulls in roughly 50,000 EU companies, and the SGX mandate pulls in every Singapore-listed issuer. Whatever the exact dollar figure, the demand side is legislated, not discretionary.

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **Definition** | Software platforms automating the measure → report → reduce → offset loop for corporate GHG emissions, built on the GHG Protocol's Scope 1/2/3 methodology (§1.1). |
| **Market size** | ⚠ Directionally: tens of billions USD today, mid-60s by ~2030, 21–23% CAGR per four analyst houses — but base-year figures disagree ~2× and are flagged; treat as a range, not a number (§1.2). |
| **Demand driver** | Regulation, not voluntarism: CSRD (~50,000 EU companies), ISSB IFRS S2, the SEC rule (stayed/rescission-pending), SGX FY2025 climate mandate, TCFD-derived disclosure norms (§2). |
| **Segments** | Enterprise Scope-1/2/3 accounting; SME tools; product carbon footprints (PCF, ISO 14067); ESG-reporting platforms; carbon-credit platforms (§3). |
| **Player structure** | Enterprise cloud suites (Salesforce, Microsoft, IBM, SAP) vs carbon-native pure-plays (Persefoni, Watershed, Sweep, Greenly, Normative, Plan A) vs removal/verification specialists (Isometric, Pachama) (§4). |
| **Technology core** | Emissions-factor databases (Ecoinvent, EPA), data integration, GHG-Protocol calculation engines, AI-assisted data ingestion and factor matching, ISO 14064/14065 assurance (§5). |
| **Market linkage** | Sits between the compliance carbon markets (EU ETS, CORSIA) and the voluntary markets (Verra, Gold Standard) — the software is where credits get booked, retired, and disclosed (§6). |
| **Banking angle** | Financed emissions (PCAF), the NZBA 2030/2050 targets, MAS green finance, green-loan products — banks are both buyers of the software and the disclosure machinery for their clients' Scope 3 (§7). |
| **The bottom line** | The carbon-management platform is becoming the financial system's "carbon ledger" — the system of record that turns a legislated obligation into auditable numbers (§9). |

---

## 2. The Regulatory Drivers

Regulation is the engine of this market. Every segment, player, and technology decision in this guide traces back to one of the disclosure regimes below. The table at the end of this section is the summary; the subsections are the verification.

### 2.1 The CSRD — EU Corporate Sustainability Reporting Directive (✅ verified)

The **Corporate Sustainability Reporting Directive** — **Directive (EU) 2022/2464**, adopted by the European Parliament and Council on **14 December 2022**, amending the accounting directive 2013/34/EU and related instruments — replaced the old Non-Financial Reporting Directive (NFRD) and is the single largest demand-side event in the carbon-management software market's history. Verified against EUR-Lex (eur-lex.europa.eu/eli/dir/2022/2464) and the European Commission's corporate-sustainability-reporting pages:

- **Scope — the ~50,000 companies:** the Commission's own framing and the industry consensus put the CSRD's scope at roughly **50,000 companies** (the NFRD covered ~11,700). The scope pulls in all large EU companies (exceeding two of: 250+ employees, €50M+ turnover, €25M+ balance-sheet total) plus listed SMEs, plus non-EU parent groups with significant EU turnover. The "~50,000" figure is the Commission-estimated order of magnitude — the standard citation, ✅ consistent across sources; treat it as an estimate, not an exact headcount.
- **First-wave timing:** the first companies (those already under the NFRD) applied the rules for the **2024 financial year**, with reports published in **2025**; the wider large-company wave follows for FY2025 (reports in 2026), listed SMEs later. Verified against the Commission's own page: "The first companies subject to the CSRD have to apply the new rules for the first time in the 2024 financial year, for reports published in 2025."
- **The standards:** reporting is to the **European Sustainability Reporting Standards (ESRS)** — including ESRS E1 on climate, which mandates GHG Protocol-aligned Scope 1/2/3 disclosure — with **mandatory limited assurance**, moving toward reasonable assurance.
- **What it means for the market:** ~50,000 in-scope companies needing auditable carbon data is the structural reason the market exists; the CSRD is the anchor reference in every carbon-platform's sales pitch and the reason the enterprise suites (Salesforce, Microsoft, SAP — §4.1) built native ESRS templates.

### 2.2 The SEC Climate-Disclosure Rule (✅ adopted 2024 — ⚠ current status: effectively withdrawn)

The **SEC** adopted its **Climate-Related Disclosure Rules on 6 March 2024** — the "In the Matter of The Enhancement and Standardization of Climate-Related Disclosures" final rules, which would have required registrants to disclose material climate risks, Scope 1 and (material) Scope 2 GHG emissions, and (for large accelerated filers) Scope 3 where material, with attestation for large filers. The timeline that followed is verified against SEC.gov press releases and statements:

- **4 April 2024:** the SEC **stayed** the rules pending consolidated litigation in the U.S. Court of Appeals for the Eighth Circuit (the rules were challenged by multiple states and private parties, and the cases were consolidated there).
- **27 March 2025:** the Commission **voted to end its defense** of the final rules in the litigation.
- **2026 (SEC press release 2026-49):** the Commission **proposed rescission** of the climate-disclosure rules.

**Current status — flagged ⚠ with confidence:** as of this pass (August 2026), the SEC climate rule is *not in effect* — it was stayed days after adoption, the agency withdrew its defense, and rescission has been formally proposed. The litigation tail may not be fully closed, and the proposed rescission must run its own rulemaking process, so the one-line status is: **stayed since 2024; defense withdrawn 2025; rescission proposed 2026 — do not design a platform around SEC climate filings.** The counterpoint for the market: the *demand* the rule represented did not evaporate — it migrated to California (SB 253/261 climate bills, in force for large companies doing business in California), to the ISSB/TCFD track (§2.3–2.4), and to the private-market expectations of investors. The carbon-software vendors barely noticed the difference; their US pipeline moved from "SEC compliance" to "California + ISSB + voluntary". This guide's banking angle (§7) inherits the same dynamic: US banks' financed-emissions work is driven by the NZBA and PCAF, not by the SEC.

### 2.3 The ISSB and IFRS S2 (✅ verified)

The **International Sustainability Standards Board (ISSB)**, established by the IFRS Foundation, issued its two inaugural standards on **26 June 2023**: **IFRS S1** (General Requirements for Disclosure of Sustainability-related Financial Information) and **IFRS S2** (Climate-related Disclosures). Verified against IFRS.org: "IFRS S2 Climate-related Disclosures was issued by the ISSB on 26 June 2023 and has an effective date of 1 January 2024." IFRS S2 integrates and builds on the TCFD recommendations (the ISSB explicitly took over the TCFD's monitoring role when the TCFD disbanded in 2023 — see §2.4). S2 requires Scope 1/2/3 GHG disclosure under the GHG Protocol, climate-risk scenario analysis, and transition-plan disclosure. Its significance is jurisdictional gravity: the ISSB standards are the designated baseline that dozens of jurisdictions (including Singapore — §2.5) are folding into their own rules, which is what makes S2 the de facto global lingua franca of climate disclosure.

### 2.4 The TCFD (✅ verified)

The **Task Force on Climate-related Financial Disclosures** was established by the **Financial Stability Board** in 2015 and published its final recommendations on **29 June 2017** — verified against the FSB's own record of the report. The TCFD framework is **11 recommendations across four pillars**: Governance, Strategy, Risk Management, and Metrics & Targets (including GHG emissions metrics). The TCFD is the ancestor of nearly everything else in this section: IFRS S2 builds directly on it, the CSRD's ESRS climate content echoes it, and the SGX mandate uses the ISSB standards which embed it. The TCFD formally completed its work and **disbanded in 2023**, handing monitoring to the ISSB — the transfer is the neatest possible illustration of how the voluntary disclosure movement became the regulatory one.

### 2.5 The SGX/MAS Singapore Climate-Reporting Mandate (✅ verified — FY2025 effective)

On **28 February 2024**, the **Accounting and Corporate Regulatory Authority (ACRA)** and **Singapore Exchange Regulation (SGX RegCo)** jointly announced mandatory climate-related disclosure for Singapore (verified against the joint media release and the SGX/ACRA announcement record):

- **Listed issuers on the SGX-ST report from FY2025** — the first wave of climate reports landed in 2026 covering FY2025, aligned with the **ISSB/IFRS S2** standards (SGX published its implementation of the IFRS Sustainability Disclosure Standards into the listing rules).
- **Large non-listed companies follow from FY2027**, with a phased-in scope (revenue/asset thresholds).
- The regime imports the ISSB baseline (Scope 1/2/3, TCFD-aligned governance/strategy/risk disclosures) and is enforced through the existing listing-rule machinery. A 2025 SGX review noted extended timelines for certain Scope-3 elements ⚠ (the phasing details evolve; the FY2025 effective date for listed issuers is the verified anchor).

**The MAS angle:** the Monetary Authority of Singapore sits behind this as policy sponsor of the green-finance agenda — its **Green Finance Action Plan** (2019) and the banks-in-Singapore sustainability expectations (cross-ref [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) §7.3) push the banks to demand climate data from borrowers, which is precisely the demand that makes Singapore a live market for the platforms in §4. For a Singapore-based bank like Cymbal Bank, the SGX mandate is not an abstract rule: every listed corporate client will be producing ISSB-format carbon numbers from FY2025 onward, and the bank's own disclosures sit under the same expectations.

### 2.6 The SBTi (✅ verified)

The **Science Based Targets initiative (SBTi)** is the partnership (CDP, UN Global Compact, WRI, WWF) that validates corporate decarbonization targets against climate-science pathways. Its **Corporate Net-Zero Standard** — published **2021**, verified against sciencebasedtargets.org ("the first global science-based standard for companies to set net-zero targets, published in 2021") — is the reference methodology for the "reduce" half of the carbon-management loop: companies commit to near-term science-based targets (typically 1.5°C-aligned, 2030-ish) plus long-term net-zero-by-2050 targets, with the residual neutralized through removals. The SBTi's importance to the software market: validated targets create the *reporting cadence* (annual progress disclosure against validated pathways) and the *abatement-planning* requirement that the platforms monetize — SBTi-validated companies are the platforms' most reliable customers, and most enterprise carbon platforms ship SBTi-aligned target-tracking modules.

### 2.7 The Drivers Table

| Driver | What it is | Verified facts | Market effect |
|---|---|---|---|
| **CSRD** (EU 2022/2464) | Mandatory sustainability reporting for ~50,000 EU companies to ESRS, with assurance | ✅ Adopted 14 Dec 2022; first wave FY2024, reports 2025; ~50,000-company scope (Commission-estimated) | The anchor demand: every in-scope firm needs auditable carbon data |
| **SEC climate rule** | US listed-company climate disclosure | ✅ Adopted 6 Mar 2024; stayed 4 Apr 2024; defense ended 27 Mar 2025; ⚠ rescission proposed 2026 | Demand migrated to California/ISSB/private markets; do not design around it |
| **ISSB IFRS S2** | Global baseline climate-disclosure standard | ✅ Issued 26 Jun 2023; effective 1 Jan 2024; builds on TCFD | The de facto global lingua franca; jurisdictions adopt it wholesale |
| **TCFD** | 11 recommendations, four pillars | ✅ Final recommendations 29 Jun 2017; FSB-established 2015; disbanded 2023 → ISSB | The ancestral framework; every later regime embeds it |
| **SGX/MAS mandate** | Singapore climate reporting | ✅ Announced 28 Feb 2024 by ACRA + SGX RegCo; listed issuers from FY2025; large non-listed from FY2027; ISSB-aligned | Live local demand; banks must collect client climate data |
| **SBTi** | Science-based target validation | ✅ Corporate Net-Zero Standard published 2021; SBTi founded 2015 | Creates reporting cadence + abatement-planning demand |

---
## 3. The Market Segments

The carbon-management software market is not one market — it is five, sold to different buyers with different pain points, regulatory clocks, and price points. The boundaries blur at the edges (the enterprise suites sell into every segment), but the segmentation explains who buys what and why.

### 3.1 Enterprise Scope-1/2/3 Accounting (✅ verified as a category)

The flagship segment: platforms for large enterprises to measure their full GHG inventory — **Scope 1** (direct emissions from owned/controlled sources), **Scope 2** (indirect from purchased energy), and **Scope 3** (all other value-chain emissions — purchased goods, logistics, business travel, use of sold products, and for banks, financed emissions). The methodology is the **GHG Protocol Corporate Accounting and Reporting Standard** (WRI/WBCSD), whose Scope 1/2/3 definitions are the universal reference; the category is verified by the existence and positioning of its named players (Persefoni, Watershed, Salesforce Net Zero Cloud, IBM Envizi — §4) and by the analyst coverage (Verdantix's Enterprise Carbon Management Green Quadrant). Buyers are CSRD/ISSB/SGX-in-scope corporates and the consultancies serving them. Scope 3 is where the pain lives: it is typically 70–90% of a company's footprint but depends on supplier data, which is why this segment shades into supplier-engagement modules and into supply-chain finance (cross-ref [supply_chain_finance_guide.md](../banking/supply_chain_finance_guide.md) — the bank's trade-finance data is a Scope-3 data source for corporate clients).

### 3.2 SME Tools (⚠ flagged)

The SME layer: lightweight, template-driven tools for small and mid-size companies that need a defensible carbon number (for a supply-chain questionnaire, a tender, a bank loan application) without an enterprise implementation. The category is real — Greenly (France, founded 2019) explicitly targets SMEs with a self-serve, card-and-receipts data-collection model, and the CSRD's listed-SME wave and the Singapore large-non-listed FY2027 wave both pull mid-market demand forward. **Flagged ⚠:** the segment boundary is fuzzy (every enterprise vendor also claims SME editions), few analysts size it separately, and the SME price points are thin — treat "SME carbon tools" as a real but loosely documented tier. The CSRD phasing (listed SMEs report later) means the SME wave lands after the large-enterprise wave — a timing point worth making in any SG go-to-market discussion.

### 3.3 Product Carbon Footprints — PCF and ISO 14067 (✅ verified standard)

A distinct sub-discipline: measuring the carbon footprint of a *product* across its life cycle (raw materials → manufacturing → use → end of life), rather than a company's annual inventory. The standard is **ISO 14067** (Greenhouse gases — Carbon footprint of products — Requirements and guidelines for quantification and communication), which builds on the life-cycle-assessment (LCA) standard ISO 14040/14044. The category is verified: ISO 14067 is a published, current ISO standard (2018 edition), and PCF data-exchange initiatives (the Catena-X automotive data space, the EU's Digital Product Passport under the Ecodesign for Sustainable Products Regulation) are documented drivers. Software for PCF is either a specialist LCA/PCF module (SimaPro, GaBi — the engineering-grade tools ⚠) or an enterprise-platform add-on; the Cedars Digital company guide ([cedars_digital_company_guide.md](cedars_digital_company_guide.md)) shows the SG market context for exactly this productized PCF angle. Why banks care: PCFs are becoming the unit of trade in green procurement and in supply-chain-finance pricing — the cleaner the PCF, the better the financing terms (cross-ref §7.4).

### 3.4 ESG-Reporting Platforms (✅ verified — cross-ref RegTech)

The broader disclosure layer: platforms that manage the full ESG data stack (environment, social, governance — climate being one pillar among many) and produce the regulatory filings (CSRD/ESRS, ISSB, GRI, DJSI/CSA questionnaires, ratings-agency responses). The category is verified by the named players — Salesforce Net Zero Cloud sits inside the broader Salesforce ESG stack; Microsoft Cloud for Sustainability spans ESG; specialist ESG suites (Workiva, Diligent/Accuvio ⚠, Envizi before IBM) — and by the regulatory-technology framing in the repo: **cross-ref [regtech_guide.md](../banking/regtech_guide.md)** for the ESG-reporting/regulatory-technology angle, and [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) for the adjacent asset-level angle. The commercial relationship: carbon-accounting platforms feed emissions data *into* ESG-reporting platforms; the reporting suites win the "single system of record for the annual report" budget, the carbon platforms win the "granular GHG inventory" budget. Banks see both: the reporting suite for the bank's own disclosures, the carbon platform for financed-emissions analytics.

### 3.5 Carbon-Credit Platforms (✅ verified — cross-ref Tokenized Assets)

The market-facing segment: platforms that source, vet, retire, and account for carbon credits and RECs — the "offset" side of the loop. This is where the software market touches the carbon markets (§6): credit registries (Verra, Gold Standard), credit marketplaces (Pachama's marketplace for nature-based credits), and the tokenization layer (Toucan Protocol bridging Verra credits, KlimaDAO, Flowcarbon, and the bank-backed Carbonplace settlement utility — **cross-ref [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §6.4**, which covers the tokenized-carbon-credit landscape in depth and flags the registry-integrity disputes). Verified: Verra operates the VCS (Verified Carbon Standard) program and piloted digital forest-carbon measurement with Pachama (verra.org); Isometric runs a removal-only registry with per-certificate provenance (registry.isometric.com). The segment's growth is hostage to credit quality and greenwashing risk (§6.3), which is precisely why the verification specialists in §4.3 exist.

### 3.6 The Segments Table

| Segment | Buyer | Core job | Standards | Verified/flagged |
|---|---|---|---|---|
| **Enterprise Scope-1/2/3 accounting** | Large corporates, CSRD/ISSB/SGX-in-scope | Full GHG inventory, reduction planning, disclosure-ready numbers | GHG Protocol (Scope 1/2/3), ISSB S2 | ✅ category and players verified |
| **SME tools** | Small/mid-size firms, listed SMEs, supply-chain respondents | Cheap defensible carbon number, questionnaire responses | GHG Protocol-lite, CSRD listed-SME phasing | ⚠ segment real but boundaries/ sizing flagged |
| **Product carbon footprints** | Manufacturers, exporters, EU Digital Product Passport | Product-level LCA footprint | ISO 14067, ISO 14040/44, PCF data spaces | ✅ ISO 14067 verified; tool tier ⚠ |
| **ESG-reporting platforms** | Corporates, banks' own disclosure teams | Full ESG disclosure stack and filings | CSRD/ESRS, GRI, ISSB, TCFD | ✅ cross-ref regtech_guide.md |
| **Carbon-credit platforms** | Corporates offsetting residuals, traders, banks | Credit sourcing, vetting, retirement, accounting | Verra VCS, Gold Standard, Isometric | ✅ cross-ref tokenized_assets_guide.md; quality risk flagged |

---

## 4. The Key Players

Three tiers: the enterprise cloud suites (incumbents that built or bought their way in), the carbon-native pure-plays (startups born for this market), and the removal/verification specialists (the trust layer). Plus the Singapore angle.

### 4.1 The Enterprise Suites

- **Salesforce Net Zero Cloud** (✅ verified — salesforce.com/net-zero/cloud): Salesforce's sustainability module on its CRM platform. Verified positioning: "robust environmental capabilities, spanning scope 1, 2, and 3 carbon emissions, plus supplier engagement, carbon conversions, and waste and water management" — with **Einstein AI** used to auto-categorize emissions factors ("Use Einstein AI to identify which greenhouse gas categories are associated with different emissions factors and categorize them"). Its edge is the CRM data graph: supplier-engagement workflows live where the supplier relationships already live, and the Salesforce ecosystem's consulting base makes it the safe enterprise choice. Launched 2021 (as Sustainability Cloud).
- **Microsoft Cloud for Sustainability** (✅ verified — announced July 2021, public preview October 2021): Microsoft's sustainability platform on Azure/Power Platform/Dynamics. Verified positioning: record, report, and reduce emissions ("completely extendable on both front and back ends" — data connectors and analytics via the Microsoft data stack). Its edge is the data-engineering story: the Microsoft Cloud for Sustainability **data model** (the common data model for ESG) and Fabric integration make it the choice for organizations whose emissions data is a data-platform problem, not a workflow problem.
- **IBM Envizi** (✅ verified — with a date correction): Envizi was an Australian ESG data-management and analytics software firm (Accel-KKR portfolio company) that IBM **acquired — announced 11 January 2022** (verified against IBM Newsroom: "2022-01-11-IBM-Acquires-Envizi"). **Flag/correction: the acquisition is frequently dated 2021 in press round-ups; the IBM announcement is January 2022.** Envizi's positioning is data-first environmental-performance management (utility bills, energy, ESG data consolidation across 100+ data types), which IBM folded into its sustainability software alongside Maximo, Sterling, and the Environmental Intelligence Suite. Its edge: the most "systems-of-record for energy data" DNA of the big-suite options.
- **SAP** (⚠ partially flagged): SAP's sustainability story runs through SAP Sustainability Control Tower / the 2023-rebranded SAP Sustainability Portfolio (Green Ledger concept — carbon alongside finance in S/4HANA). The *intent* is verified (SAP publicly positions carbon accounting as a ledger concept and ships sustainability modules in S/4HANA); the specific product-line naming and GA dates shifted through 2023–2025 and were not re-verified in this pass ⚠. SAP's edge for the banking angle: the SAP-installed base *is* the corporate client base — a bank's financed-emissions data for SAP-run corporates often exists inside their own systems already.

### 4.2 The Carbon-Native Platforms

- **Persefoni** (✅ verified — founded 2020): Persefoni AI, Inc. — founded **2020**, co-founded by Kentaro Kawamori, Jason Offerman, and Kim Stroh (verified via tracker profiles; the founders' background is enterprise software — Kawamori was CEO of Clutch Holdings; the firm was a major 2021–2022 venture darling before the climate-tech funding reset ⚠ on the funding specifics). Verified positioning on persefoni.com: "Measure, report, and decarbonize with **AI-native** sustainability software" — Persefoni is the platform that most explicitly claims the AI-native label, which makes it the case study for the repo's AI-native-company frame: **cross-ref [ai_native_companies_guide.md](ai_native_companies_guide.md)** — the model-is-product, inference-led economics, and data-flywheel properties apply directly to Persefoni's AI-assisted data ingestion and factor matching. Its edge: PCAF-aware financial-sector depth (it sells a financed-emissions workbench) and audit-grade methodology documentation.
- **Watershed** (✅ verified 2019 founding — ⚠ scale claims flagged): founded **2019** in San Francisco by ex-Stripe product leaders (Taylor Francis, Avi Itskovich — the Stripe pedigree is press-documented), backed by Stripe itself, Kleiner Perkins, and others. Verified via trackers/press: total funding ~**US$185M** across four rounds (Tracxn); the company reports **500+ enterprise customers and ~1 gigaton of CO₂e managed** (esgtechreport — company-reported, ⚠ flagged as unverified scale claims); named a **leader in Verdantix's 2026 Green Quadrant: Enterprise Carbon Management Software**. Its edge: enterprise-grade measurement depth (its own "climate database" of emissions factors), a credit marketplace, and a 100%-audit-pass-rate claim ⚠. For banks, Watershed's banking/financed-emissions modules (used by publicized bank clients ⚠) make it a Persefoni alternative in shortlists.
- **Sweep** (⚠ founded 2020, press-verified): French-founded enterprise carbon-management platform built for the whole value chain (the "system of record for carbon data" positioning); press reports ~US$100M raised in its first year and a "3% of global emissions data managed" claim (esgtechreport — company-reported ⚠). Its edge: value-chain/supplier collaboration workflows, strong CSRD fit.
- **Greenly** (⚠ founded 2019): French platform, the leading SME-tier carbon-accounting tool — self-serve data collection, card-linked emissions tracking for spend-based footprints, and a strong European SMB base; funding and customer figures not re-verified this pass ⚠.
- **Normative** (⚠ founded 2014): Swedish carbon-accounting platform — one of the oldest carbon-native vendors; known for the "carbon engine" (an emissions-factor knowledge base) and science-based-target tooling; Sweden/UK-centric footprint ⚠.
- **Plan A** (⚠ founded 2017): Berlin-based corporate carbon-accounting and ESG platform (Plan A GmbH); known for data-science-heavy carbon accounting and its decarbonization-software positioning; used by European corporates ⚠. Note: not to be confused with the UK climate-charity "Plan A" (a different entity — this guide covers the Berlin software firm; the name collision is worth a due-diligence footnote).

### 4.3 The Removal and Verification Specialists

- **Isometric** (⚠ founded 2022 — press-verified; positioning ✅): the carbon-**removal** registry — verified positioning (registry.isometric.com): certificates issued against the Isometric Standard, each certificate = **1 tonne of CO₂ removed** with full provenance data; backed by Lowercarbon Capital; founded by Eamon Jubbawy (Onfido co-founder) ⚠ on the details. Distinctive: it lists **only removal** credits (not avoidance/reduction offsets), publishes the science behind each protocol, and has pivoted its brand to "**agentic certification for the industrial economy**" — AI agents working alongside human verifiers on every data point (isometric.com/about). That AI-verifier model is the direct real-world analogue of the AI-carbon-verification theme in [cedars_digital_company_guide.md](cedars_digital_company_guide.md). For banks: the registry's science-first, removal-only stance is the "high-quality credit" answer to §6.3's greenwashing problem.
- **Pachama** (✅ positioning verified — ⚠ details): forest-carbon technology company using **satellite imagery, LiDAR, and AI to measure and monitor forest carbon** for reforestation/conservation projects; verified via Verra's own announcement that the world's largest carbon program **piloted digital forest-carbon measurement with Pachama** (verra.org), and via tracker profiles (netzerocompare.com) describing the platform's continuous forest-change detection. **Cross-ref [remote_sensing_technologies_guide.md](remote_sensing_technologies_guide.md)** for the underlying technology (GEDI/LiDAR biomass, SAR, NDVI/GPP calibration — the guide's §8.5 climate-and-carbon monitoring section is the technical substrate for Pachama-class MRV). ⚠ Funding and customer specifics not re-verified this pass.

### 4.4 The Singapore Angle (⚠ flagged)

The local layer of the market — relevant because the SGX FY2025 mandate (§2.5) creates domestic demand and because this guide's worked example (§8) is a Singapore bank's selection:

- **Global carbon platforms with SG presence:** Persefoni, Watershed, Salesforce, Microsoft, IBM all sell into Singapore via regional offices and partners; for a Singapore bank the "local" choice is usually a global platform with an APAC implementation partner, not a local product.
- **Home-grown carbon/ESG software:** the repo's SG-tech guides document the local ecosystem — **Cedars Digital** ([cedars_digital_company_guide.md](cedars_digital_company_guide.md)) positions as the SG AI carbon-footprint-verification firm ("Carbon Footprint Verification with AI", ISO/IEC 42001 claims, Deloitte/SGS partnership claims — all company-reported and flagged in that guide); the broader SG sustainability-software set (local ESG-reporting and carbon-consulting tooling) is documented piecemeal across the repo's Singapore guides ⚠ — no single authoritative directory of SG carbon-software vendors was found this pass, and the local pure-play market is thin relative to the global vendors' APAC push. **Flagged ⚠:** treat any "SG carbon-software leaderboard" claim with suspicion; the verified facts are the global platforms' SG presence and the ACRA/SGX demand side.

### 4.5 The Players Table

| Player | Focus | Notes (verified unless flagged) |
|---|---|---|
| **Salesforce Net Zero Cloud** | Enterprise carbon + ESG on CRM | ✅ Scope 1/2/3, supplier engagement, Einstein AI factor matching; launched 2021 |
| **Microsoft Cloud for Sustainability** | Enterprise carbon + ESG on Azure/Power Platform | ✅ Announced Jul 2021, preview Oct 2021; data-model + Fabric integration edge |
| **IBM Envizi** | ESG/energy data consolidation | ✅ IBM acquisition announced **11 Jan 2022** (⚠ often mis-dated 2021); data-first DNA |
| **SAP** | Carbon ledger inside S/4HANA | ⚠ Product-line naming shifted 2023–25; "Green Ledger" intent verified, specifics flagged |
| **Persefoni** | AI-native carbon accounting, financial-sector depth | ✅ Founded 2020 (Kawamori/Offerman/Stroh); "AI-native" self-positioning verified; PCAF workbench |
| **Watershed** | Enterprise measurement/reporting/marketplace | ✅ Founded 2019, ~$185M raised; ⚠ 500+ customers/1 Gt managed claims company-reported; Verdantix 2026 leader |
| **Sweep** | Value-chain carbon management, CSRD | ⚠ Founded 2020 (press); "3% of global emissions" claim company-reported |
| **Greenly** | SME carbon tools | ⚠ Founded 2019 (press); self-serve SME model |
| **Normative** | Carbon engine + science-based targets | ⚠ Founded 2014 (press); one of the oldest carbon-natives |
| **Plan A** | Corporate carbon/ESG, data-science-heavy | ⚠ Founded 2017 (press); Berlin; name-collision footnote |
| **Isometric** | Carbon-**removal** registry, science-first | ✅ Registry + 1-certificate-per-tonne verified; ⚠ founded 2022 (press); agentic-certification AI-verifier model |
| **Pachama** | Forest-carbon MRV (satellite + AI) | ✅ Verra digital-MRV pilot verified; ⚠ commercial details; cross-ref remote_sensing guide |

---

## 5. The Technology

The platform stack underneath every vendor above. Five layers: where the numbers come from (emissions factors), how they get in (integration), how they're computed (calculation engines), how AI accelerates them, and how they're made trustworthy (assurance).

### 5.1 The Emissions-Factors Databases (✅ verified institutions — ⚠ catalogue details)

The conversion layer: activity data (kWh, litres, tonnes-km, spend) × **emissions factor** (kg CO₂e per unit) = emissions. The two reference sources:

- **Ecoinvent** — the Swiss life-cycle-inventory database (ecoinvent v3.x), the de facto standard for LCA/PCF work (ISO 14067 quantification relies on exactly this class of background data); academically maintained, industry-licensed. ✅ Ecoinvent is a real, current, widely licensed LCI database; ⚠ specific version numbers and licence terms not re-verified this pass.
- **The US EPA** — the EPA's GHG Emission Factors Hub and eGRID (electricity-grid emission factors by subregion) plus the Center for Corporate Climate Leadership's guidance; the US corporate-reporting reference. ✅ EPA factor resources are real and current; ⚠ catalogue specifics flagged.

The platforms wrap these (plus commercial factor libraries and their own curated sets — Watershed's "climate database", Normative's "carbon engine") behind a lookup service. For banks, the electricity-factor question (market-based vs location-based, per GHG Protocol Scope 2 guidance) is the most consequential factor decision in financed-emissions calculations.

### 5.2 The Data Integration (✅ verified as the core engineering problem)

Every serious evaluation of carbon software concludes the same thing: the *data collection* is 80% of the project and the calculation is 20%. Verified by the vendors' own architectures (Microsoft's connector model "completely extendable on both front and back ends"; Salesforce's supplier-engagement data capture; the audit-grade ingestion pipelines the carbon-natives advertise). The integration surface for an enterprise: ERP/energy-management data (utility bills, fuel, fleet), HR systems (commuting/remote work), procurement systems (purchased goods), logistics/TMS (freight), and for banks, the loan book, trade-finance systems, and treasury data (cross-ref [supply_chain_finance_guide.md](../banking/supply_chain_finance_guide.md) — the trade data the bank already holds is the raw material of its clients' Scope 3 and its own financed emissions). The integration architecture pattern is the repo's standard data-platform play: connectors/APIs → ingestion → validation → a canonical emissions data model → the calculation engine → the disclosure output. **Cross-ref the data-fabric discipline** ([data/data_fabric_guide.md](data/data_fabric_guide.md)) for the platform pattern.

### 5.3 The Calculation Engines — GHG Protocol Scope 1/2/3 (✅ verified)

The methodological core: the **GHG Protocol Corporate Accounting and Reporting Standard** (WRI/WBCSD) — the definitions of **Scope 1** (direct), **Scope 2** (purchased energy — with market/location-based dual reporting), and **Scope 3** (15 categories of value-chain emissions, from purchased goods to investments) are the universal calculation grammar, embedded in every platform and every regime in §2 (CSRD/ESRS E1, ISSB S2, SGX all point at GHG-Protocol-aligned inventories). ✅ Verified: the GHG Protocol is the reference methodology — confirmed this pass via the GHG-Protocol-hosted PCAF standard (ghgprotocol.org hosts the financial-industry GHG accounting standard, which is itself built as an extension of the Corporate Standard — the GHG Protocol–PCAF link is direct). The calculation engine turns the normalized data model into Scope 1/2/3 totals with the emissions-factor lookups, and — critically for audit — preserves the *calculation provenance* (which factor, which formula version, which data vintage) behind every tonne.

### 5.4 The AI Layer (✅ verified as a live feature set — cross-ref the AI-native frame)

AI is no longer speculative in this market; it is shipped. Verified examples from this pass: **Salesforce Einstein** auto-categorizing emissions factors; **Persefoni's** AI-native data ingestion and factor matching; **Isometric's** agentic certification (AI agents reviewing data points alongside human verifiers); **Pachama's** satellite-imagery AI for forest-carbon measurement (cross-ref [remote_sensing_technologies_guide.md](remote_sensing_technologies_guide.md) for the geospatial substrate); and the SG angle — Cedars Digital's AI carbon-footprint-verification positioning ([cedars_digital_company_guide.md](cedars_digital_company_guide.md), company-claimed). The AI-native-company frame applies most cleanly to Persefoni and Watershed: **cross-ref [ai_native_companies_guide.md](ai_native_companies_guide.md)** for the definition (model-is-product, inference-led economics, data flywheel) and the governance discipline ([ai_trust_assessments_guide.md](ai_trust_assessments_guide.md), [responsible_ai_frameworks_guide.md](responsible_ai_frameworks_guide.md)) — in carbon accounting the AI governance question is acute because a hallucinated factor or a silently mis-mapped data feed corrupts a *regulated, audited, financially consequential* number. The pattern the market has converged on: **AI for ingestion and classification speed, humans for verification and sign-off** — the same human-in-the-loop principle the repo's AI guides recommend everywhere.

### 5.5 The Assurance Layer — ISO 14064/14065 (✅ verified standards)

The trust layer that turns software output into a report someone can sign:

- **ISO 14064** — the GHG accounting/verification standard family: Part 1 (organization-level quantification and reporting), Part 2 (project-level), Part 3 (verification/validation requirements for GHG assertions). Current editions: 2018 for Parts 1–2, 2019 for Part 3 ⚠ (edition years are stable standard facts; flagged as not re-verified this pass).
- **ISO 14065** — requirements for bodies that validate/verify GHG assertions (accreditation of the verifiers; 2020 edition ⚠).
- The ESRS/CSRD regime adds the statutory layer: mandatory **limited assurance** on sustainability reports (moving toward reasonable) — which means the platform's audit trail must survive an external verifier's inspection. This is the commercial moat of the enterprise-grade platforms: the data provenance model (§5.3) exists to make ISO 14064-3-style verification cheap. For the SG market, the same assurance logic applies under the SGX mandate.

### 5.6 The Technology Table

| Layer | What it does | Key references | Verified/flagged |
|---|---|---|---|
| **Emissions factors** | Activity data × factor = emissions | Ecoinvent, EPA GHG Hub/eGRID, vendor factor libraries | ✅ institutions; ⚠ catalogue specifics |
| **Data integration** | Connectors → ingestion → canonical data model | ERP/energy/HR/procurement/TMS/loan-book feeds | ✅ the dominant engineering effort |
| **Calculation engine** | Scope 1/2/3 computation with provenance | GHG Protocol Corporate Standard; PCAF for financials | ✅ GHG Protocol verified; PCAF link confirmed |
| **AI** | Factor matching, ingestion, verification, MRV | Einstein, Persefoni AI, Isometric agents, Pachama, Cedars | ✅ shipped features; cross-ref AI guides |
| **Assurance** | Third-party verification of the numbers | ISO 14064 (1–3), ISO 14065; CSRD limited assurance | ✅ standards verified; ⚠ edition years flagged |

---
## 6. The Carbon Markets

The carbon-management software market's demand side is regulation (§2); its supply side touches the carbon markets — where credits and allowances are created, priced, and retired. Two markets, one controversy.

### 6.1 The Compliance Markets (✅ verified)

- **The EU Emissions Trading System (EU ETS)** — the world's first and largest carbon market, **launched 2005** (Phase I pilot 2005–2007, Phase IV now running), a cap-and-trade system covering ~40% of EU emissions (power, industry, aviation since 2012, shipping and buildings/road-transport extension under ETS2 phased in from 2027 ⚠ on the precise phasing). ✅ The 2005 launch is standard, stable history; the extension phasing is flagged. The EU ETS is the compliance benchmark every other carbon market copies, and the price signal it produces (€60–100/t in recent years ⚠ price level flagged — volatile) is what makes carbon a *financial* quantity.
- **CORSIA** — the **Carbon Offsetting and Reduction Scheme for International Aviation**, adopted by the **39th ICAO Assembly in 2016** (Resolution A39-3), the first global sectoral market-based measure: airlines offset growth in international-flight emissions above a baseline with eligible credits. ✅ Verified via ICAO/EASA documentation of the 2016 Resolution A39-3 adoption. The EU's implementation rules aligning ETS aviation with CORSIA were adopted in 2025 (verified — climate.ec.europa.eu, June 2025).

For the software market: compliance regimes create *allowance-accounting* needs (the ETS MRV — monitoring, reporting, verification — requirements are a regulated software category of their own) and the data that feeds them is the same GHG inventory the platforms compute.

### 6.2 The Voluntary Markets (✅ verified institutions)

- **Verra** — the non-profit that runs the **Verified Carbon Standard (VCS)**, the largest voluntary carbon-credit program, plus the SD VISta social-impact and Plastic programs. ✅ Verified: Verra is the dominant voluntary registry (the "world's largest carbon program" per its own materials — verra.org, confirmed via the Pachama pilot announcement).
- **Gold Standard** — the registry founded in 2003 by WWF and other NGOs (originally under the CDM architecture), known for higher social-co-benefit thresholds. ⚠ founding detail flagged (2003 origin is standard history; not re-verified this pass).
- The voluntary market's mechanics: project developer → third-party validation → registry issuance (one verified credit = 1 tCO₂e) → corporate buyer → **retirement** (permanent cancellation) → disclosure in the carbon platform. The retirement step is the point where the credit platforms (§3.5) and the tokenization layer ([tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §6.4) attach — and the point where double-counting risk lives (the same credit must not be claimed by two footprints; registries and retirement ledgers exist to prevent it).

### 6.3 The Offset Controversies (⚠ flagged)

The credibility problem is structural and well documented:

- **Greenwashing findings:** the 2023 Guardian investigation alleging that a large share of Verra-certified rainforest-offset credits did not represent real, additional emission reductions (the "phantom credits" findings) — press-reported ⚠ (the underlying journalism is real; the contested findings remain disputed between the journalists, the researchers, and Verra).
- **Quality dimensions:** additionality (would the reduction have happened anyway?), permanence (fires, logging, and development can reverse forest-carbon gains), leakage (deforestation moves next door), and baseline inflation — the standard critique set.
- **The market's response** — verified this pass: (1) the shift to **removal** credits with strict provenance — Isometric's registry is built entirely around this thesis; (2) **digital MRV** — Verra piloting satellite/AI measurement with Pachama (verra.org); (3) corporate buyers demanding retirements with serial numbers and disclosure trails; (4) the tokenization experiments — but registry-integrity disputes remain the documented brake on that segment ([tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) §6.4 flags exactly this).

For banks and platform buyers the operating rule is: **treat offset quality as a first-class risk dimension, not a checkbox** — which is why the verification specialists (§4.3) and the registry layer are where the market's trust budget is being spent.

### 6.4 The Markets Table

| Market | What it is | Verified facts | Software-market linkage |
|---|---|---|---|
| **EU ETS (compliance)** | Cap-and-trade, largest carbon market | ✅ Launched 2005; aviation in 2012; ⚠ ETS2 extension phasing flagged | Allowance/MRV accounting; the price signal that makes carbon financial |
| **CORSIA (compliance)** | Aviation offsetting scheme | ✅ 39th ICAO Assembly, 2016, Resolution A39-3; EU implementing rules 2025 | Airline offset accounting; credit demand |
| **Verra VCS (voluntary)** | Largest voluntary credit program | ✅ "World's largest carbon program"; digital-MRV pilot with Pachama | Credit issuance/retirement accounting; the greenwashing risk surface |
| **Gold Standard (voluntary)** | NGO-founded registry, co-benefit focus | ⚠ 2003 founding (standard history, flagged) | Premium-credit sourcing |
| **Isometric (removal)** | Removal-only registry with provenance | ✅ 1 certificate = 1 tCO₂e removed; science-first | The "high-quality credit" answer to controversies |
| **Offset controversies** | Additionality/permanence/leakage disputes | ⚠ Guardian 2023 Verra findings press-reported; responses verified (removals, digital MRV, tokenization) | Drives demand for verification technology (§4.3, §5.4) |

---

## 7. The Banking Angle

Banks are double customers of this market: they must report their own footprint, and — more consequentially — they must report the emissions **financed by their lending and investment books**, which is the single largest accounting surface the market has ever faced. This section is the reason a bank's solution architect reads this guide.

### 7.1 The Financed Emissions — PCAF (✅ verified)

**Financed emissions** = the share of a borrower's or investee's GHG emissions attributable to a financial institution's lending/investment, computed by attribution factor (the bank's exposure ÷ the borrower's enterprise value, or similar). The methodology standard is **PCAF — the Partnership for Carbon Accounting Financials**: launched globally in **September 2019** (initiated by Dutch financial institutions from 2015), and publisher of the **Global GHG Accounting and Reporting Standard for the Financial Industry** (published **November 2020**), which extends the GHG Protocol into the financial sector — covering six asset classes (listed equity/corporate bonds, business loans and unlisted equity, project finance, commercial real estate, mortgages, and sovereign debt ⚠ on the exact current asset-class list) and defining **data-quality scores** (a 1–5 scale grading how close the underlying data is to actual measured emissions vs estimated). ✅ Verified this pass: the September 2019 global launch, the GHG-Protocol-hosted standard, and the measurement/reporting focus — all confirmed via carbonaccountingfinancials.com. PCAF data-quality scores are the single most important technical concept in the banking angle: a bank's financed-emissions number is only as good as its data-quality score, and improving the score (from estimated sector averages toward client-reported actuals) is exactly what the carbon platforms' data-collection machinery does.

### 7.2 The Net-Zero Banking Alliance (✅ verified — with a 2025–26 reality check)

The **Net-Zero Banking Alliance (NZBA)** — convened by **UNEP FI** in **April 2021** (launched with **43 founding banks** as part of the Glasgow Financial Alliance for Net Zero / GFANZ), committing members to align lending and investment portfolios with net-zero-by-2050 pathways, with 2030 intermediate targets. ✅ Verified: the April 2021 launch and 43 founding banks (unepfi.org). **The reality check — flagged ⚠ with confidence:** beginning late 2025, a wave of major US banks (and some non-US) **withdrew from the NZBA**, ahead of US political pressure on climate-finance coalitions (press-documented departures of large US money-center banks from NZBA and GFANZ through 2025–2026; the ScienceDirect paper found in this pass describes "the rise and demise of the Net-Zero Banking Alliance" and measures the market reaction — an independent academic confirmation that the exodus is real). The significance for this guide: the *voluntary* commitment has frayed, but the *regulatory and market* machinery it built (PCAF-based measurement, financed-emissions disclosure, transition-plan expectations) has not unwound — the CSRD/ISSB/SGX regimes and investor pressure carry the same requirements forward. A bank's platform decision should be built on PCAF/regulatory requirements, not on alliance membership.

### 7.3 The MAS Green-Finance Context (✅ verified — cross-ref)

Singapore's regulator is one of the most active green-finance policymakers in Asia: the MAS **Green Finance Action Plan** (2019), the **Green Finance Industry Taskforce** taxonomy work, the **ESG data and disclosure** push (including the Project Greenprint data-infrastructure initiative ⚠ on current programme names), and the sustainability-reporting expectations for financial institutions that sit alongside the ACRA/SGX regime (§2.5). **Cross-ref [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md)** for the full MAS regime context; the carbon-platform relevance is that MAS's data-and-disclosure agenda is precisely the demand side the global platforms' APAC teams sell into, and a Singapore bank's financed-emissions platform must speak PCAF (global), ISSB (the SGX baseline), and MAS's expectations simultaneously.

### 7.4 The Green-Loan Products (⚠ flagged)

The product side: **green loans** (proceeds ring-fenced for eligible green purposes), **sustainability-linked loans (SLLs)** (pricing tied to the borrower's ESG KPIs — often carbon-intensity targets verified by an external reviewer), and the emerging **transition-finance** variants. The category is real and documented (the LMA/APLMA Green Loan Principles and Sustainability-Linked Loan Principles are the market standards ⚠ on the details — established, not re-verified this pass). **Flagged ⚠:** the *software* angle — "green-loan products inside carbon platforms" is a thin, fast-moving niche: the platforms provide the KPI/emissions data that SLL pricing mechanics reference (the borrower's platform output feeds the external-verifier opinion that triggers the margin ratchet), and banks increasingly require borrower emissions data as a credit-underwriting input, but there is no mature, named software category for green-loan administration as of this pass. For a bank architect the practical reading: the carbon platform is the *data* layer under the green-loan product, not the loan system itself — the loan books to the core banking system, the carbon data comes from the platform.

### 7.5 The Banking Table

| Element | What it is | Verified facts | Platform implication |
|---|---|---|---|
| **Financed emissions (PCAF)** | Bank's attributable share of client emissions | ✅ PCAF launched globally Sep 2019; Global GHG Standard Nov 2020; GHG-Protocol-extension; data-quality scores | The core calculation the platform must support; data-quality scores are the metric |
| **NZBA** | Voluntary net-zero commitment for banks | ✅ Launched Apr 2021, 43 founding banks, UNEP FI/GFANZ; ⚠ major withdrawals 2025–26 (press + academic) | Do not anchor the business case on membership; anchor on PCAF/regulation |
| **MAS green finance** | SG regulatory push on green data/disclosure | ✅ Green Finance Action Plan 2019; cross-ref banks_in_singapore_guide.md | Platform must serve MAS/ISSB/SGX simultaneously |
| **Green loans / SLLs** | Proceeds-linked and KPI-linked lending | ⚠ Principles exist (LMA/APLMA); software niche immature | Carbon data feeds KPI verification; loan admin stays in core banking |

---

## 8. The Worked Example — A Cymbal Bank Platform Selection

The scenario: **Cymbal Bank, Singapore** — a commercial bank with a corporate loan book, trade-finance operations, and a growing green-finance product line — must stand up a **financed-emissions platform** to meet its PCAF-based measurement commitments, its ISSB-aligned disclosure obligations (and the SGX climate-reporting expectations its listed clients face), and its green-loan KPI-verification needs. This section is the selection design, in the house pattern of the repo's worked examples.

### 8.1 The Scenario and the Requirements

The bank's context (the familiar corpus setting — a Singapore solution architect's world): the loan book spans listed corporates (SGX-listed clients now producing ISSB-format climate reports from FY2025 — §2.5), mid-market borrowers (no climate data at all), trade-finance clients (whose invoice/freight data the bank already holds), and real-estate exposure (mortgages and commercial property). The requirements the selection must satisfy:

1. **PCAF alignment** — the platform must implement the PCAF Global GHG Standard across the asset classes Cymbal holds (business loans, listed equity/corporate bonds, commercial real estate, mortgages), with **data-quality scores** tracked per portfolio segment and a clear score-improvement roadmap (§7.1).
2. **Attribution-factor machinery** — enterprise-value-based attribution for listed exposures, and the PCAF-prescribed approaches for unlisted loans (which default to sector-average intensity × exposure — the low-data-quality starting point most portfolios face).
3. **Disclosure outputs** — ISSB IFRS S2 / TCFD-aligned financed-emissions tables for the bank's own annual reporting, and CSRD-ready exports for the bank's EU counterparties that ask for the bank's data in ESRS format.
4. **Data integration** — feeds from the core loan system, trade-finance systems, treasury, and the client-onboarding data; plus a client-data-collection channel (the platform's supplier-style portal, pointed at borrowers) to lift data-quality scores from estimated to reported.
5. **Assurance-readiness** — a provenance-preserving audit trail (every tonne traceable to factor, formula version, and data vintage — §5.3) so the external verifier's ISO 14064-3 / CSRD limited-assurance work is cheap.
6. **The AI guardrail** — AI-assisted ingestion and factor matching permitted, but **human verification and sign-off mandatory** for anything that reaches the disclosure (the §5.4 governance pattern).
7. **Singapore fit** — MAS expectations (§7.3), local data-residency and hosting posture, and an APAC implementation partner.

### 8.2 The PCAF-Aligned Selection Design

The shortlist — three of the §4 players plus one enterprise option, scored against the requirements:

| Criterion (weight) | Persefoni | Watershed | Salesforce Net Zero Cloud | IBM Envizi |
|---|---|---|---|---|
| PCAF asset-class coverage & data-quality scores (25%) | ✅ native financed-emissions workbench, PCAF-first | ✅ banking modules, ⚠ depth varies by asset class | ⚠ via ecosystem partners | ⚠ data model strong, financed-emissions templates thinner |
| Attribution & calculation transparency (15%) | ✅ audit-grade methodology docs | ✅ strong factor DB | ⚠ configurable | ✅ strong |
| Disclosure outputs ISSB/TCFD/CSRD (15%) | ✅ | ✅ | ✅ native ESG stack | ✅ |
| Data integration to loan/trade systems (20%) | ⚠ good APIs | ✅ strong connectors | ✅ CRM-native but loan-system feeds need work | ✅✅ the data-consolidation specialist |
| Assurance-readiness / audit trail (15%) | ✅ | ✅ (100%-audit-pass claim ⚠) | ✅ | ✅ |
| AI assistance with human sign-off (5%) | ✅ AI-native | ✅ | ✅ Einstein | ⚠ conservative |
| APAC/SG delivery posture (5%) | ✅ | ✅ | ✅✅ | ✅ |

**The design outcome (a reasoned recommendation, not a procurement verdict):** for a bank whose first problem is **PCAF-correct financed-emissions numbers with defensible data-quality scores**, the carbon-native pair (Persefoni, with Watershed as the runner-up) leads on the two highest-weighted criteria; the enterprise suites lead on integration into the bank's existing vendor estate. The pragmatic Cymbal design is a **two-layer architecture**: the carbon-native platform as the financed-emissions calculation and disclosure engine, and the enterprise data platform (the bank's existing integration layer, not a second carbon product) as the ingestion backbone — with the AI features enabled only under the human-verification guardrail. The PCAF data-quality scoreboard becomes the bank's management KPI: the platform's real job is to move the portfolio from Score 5 (sector estimates) toward Score 1–2 (audited client-reported data) segment by segment, starting with the largest exposures.

### 8.3 The Lessons

1. **PCAF alignment is table stakes, not a differentiator** — every credible vendor claims it; the differentiator is asset-class depth and data-quality-score mechanics.
2. **The data-quality score is the product** — the platform is a machine for converting estimated emissions into reported, verifiable emissions; the selection should be judged on which vendor moves the score fastest.
3. **Integration is the project** — 80% of the effort is feeding loan-book, trade, and client data in (§5.2); the vendor with the best connector story for the bank's actual systems wins more than the vendor with the best demo.
4. **AI speeds ingestion, humans sign the numbers** — the §5.4 guardrail is not a compromise; it is the assurance requirement in software form.
5. **Anchor on regulation, not coalitions** — the NZBA exodus (§7.2) does not change the requirement set; CSRD/ISSB/SGX/PCAF do.
6. **The platform is the ledger, not the strategy** — the carbon platform records what the bank and its clients do; the business case discipline lives in [business_case_development_guide.md](../management/business_case_development_guide.md) and the narrative in [beyond_zero_guide.md](../management/beyond_zero_guide.md).

---

## 9. The Summary — One Page

- **The market:** carbon-footprint-management software — the platforms that automate measure → report → reduce → offset for corporate GHG emissions. Directionally a tens-of-billions market growing at ~22% CAGR toward the mid-60s by ~2030; the precise figures are analyst-dependent and flagged (§1).
- **The demand is legislated:** the CSRD (~50,000 EU companies, first wave FY2024), ISSB IFRS S2 (June 2023, the global baseline), the TCFD (2017, the ancestor), the SGX/ACRA mandate (listed issuers from FY2025, ISSB-aligned), and the SBTi (validated targets) — with the SEC climate rule (adopted March 2024, stayed, defense withdrawn, rescission proposed 2026) the cautionary tale of regulatory volatility (§2).
- **Five segments, three player tiers:** enterprise Scope-1/2/3 accounting, SME tools, product carbon footprints (ISO 14067), ESG-reporting platforms, carbon-credit platforms — served by the enterprise suites (Salesforce Net Zero Cloud, Microsoft Cloud for Sustainability, IBM Envizi, SAP), the carbon-natives (Persefoni, Watershed, Sweep, Greenly, Normative, Plan A), and the trust layer (Isometric, Pachama) (§3–§4).
- **The stack is standard:** emissions factors (Ecoinvent, EPA), data integration, GHG-Protocol calculation engines with provenance, AI-assisted ingestion under human sign-off, and ISO 14064/14065 assurance (§5).
- **The markets are two-sided:** the compliance markets (EU ETS since 2005, CORSIA since 2016) and the voluntary markets (Verra, Gold Standard) — with offset-quality controversies driving the shift to removal credits and digital MRV (§6).
- **The banking angle is the biggest surface:** PCAF financed emissions with data-quality scores, the NZBA (launched April 2021, since frayed — but the measurement machinery endures), MAS green finance, and green loans whose KPI mechanics run on platform data (§7).
- **The final word — the measured footprint.** Every regime in this guide converges on one demand: an auditable, defensible number. The carbon-management platform's enduring role is to be the **system of record that turns a legislated obligation into a measured footprint** — and for a bank, the measured footprint of what it finances is the most consequential number its systems will ever be asked to produce. The platforms will change names and owners; the requirement will not.

---

## 10. The Glossary

| Term | Definition |
|---|---|
| **Carbon footprint** | The total GHG emissions attributable to an organization, product, or activity, expressed in tonnes of CO₂-equivalent (tCO₂e). |
| **Carbon accounting** | The measurement and quantification of GHG emissions against a defined methodology (GHG Protocol, PCAF), producing an emissions inventory. |
| **Carbon management** | Carbon accounting plus the strategy layer: reduction planning, target setting, offsetting, and disclosure. |
| **Scope 1** | Direct GHG emissions from sources owned or controlled by the reporting organization (combustion, fleet, process emissions). |
| **Scope 2** | Indirect emissions from purchased electricity, heat, steam, or cooling (reported market-based and location-based). |
| **Scope 3** | All other value-chain emissions: 15 categories from purchased goods and logistics to use of sold products and investments (the largest, hardest category). |
| **GHG Protocol** | The WRI/WBCSD accounting standards — the universal methodology for corporate GHG inventories (Scope 1/2/3). |
| **CSRD** | EU Corporate Sustainability Reporting Directive (EU 2022/2464) — mandatory sustainability reporting for ~50,000 companies to ESRS, with assurance. |
| **SEC** | US Securities and Exchange Commission — adopted climate-disclosure rules March 2024; stayed, defense withdrawn, rescission proposed (not in effect). |
| **ISSB** | International Sustainability Standards Board (IFRS Foundation) — issuer of IFRS S1/S2 (June 2023). |
| **IFRS S2** | Climate-related Disclosures standard — the global baseline climate-disclosure standard, effective 1 January 2024. |
| **TCFD** | Task Force on Climate-related Financial Disclosures (FSB) — 2017: 11 recommendations across governance, strategy, risk management, metrics & targets. |
| **SBTi** | Science Based Targets initiative — validates corporate targets against climate science; Corporate Net-Zero Standard (2021). |
| **SGX** | Singapore Exchange — with ACRA, mandates ISSB-aligned climate reporting for listed issuers from FY2025 (large non-listed from FY2027). |
| **PCF** | Product carbon footprint — the life-cycle GHG emissions of a product, quantified per ISO 14067. |
| **ISO 14067** | The standard for product carbon-footprint quantification and communication (2018 edition). |
| **ISO 14064** | The GHG accounting/verification standard family — Part 1 organization-level, Part 2 project-level, Part 3 verification. |
| **ISO 14065** | Requirements for bodies that validate and verify GHG assertions (the verifier-accreditation standard). |
| **Ecoinvent** | The Swiss life-cycle-inventory database — the de facto reference for LCA/PCF background data. |
| **EPA** | US Environmental Protection Agency — publishes the GHG Emission Factors Hub and eGRID electricity factors. |
| **ETS** | Emissions Trading System — cap-and-trade; the EU ETS launched 2005, the world's first and largest carbon market. |
| **CORSIA** | Carbon Offsetting and Reduction Scheme for International Aviation — ICAO's 2016 (Resolution A39-3) sectoral offsetting scheme. |
| **Verra** | Non-profit running the VCS — the world's largest voluntary carbon-credit program. |
| **Gold Standard** | NGO-founded voluntary registry (2003) with strong social-co-benefit criteria. |
| **Offset** | A verified credit representing 1 tCO₂e of reduction or removal, retired against a footprint — quality varies (additionality, permanence, leakage risks). |
| **PCAF** | Partnership for Carbon Accounting Financials — the Global GHG Accounting and Reporting Standard for financial institutions (2019 launch, 2020 standard) with data-quality scores. |
| **NZBA** | Net-Zero Banking Alliance — UNEP FI-convened (April 2021, 43 founding banks); voluntary portfolio net-zero commitment (withdrawals from 2025). |
| **Financed emissions** | The share of borrowers'/investees' emissions attributable to a bank's exposure, via PCAF attribution factors. |
| **Net zero** | Balancing residual emissions with removals; corporate framing via SBTi (2030 near-term + 2050 long-term targets). |
| **Persefoni** | AI-native carbon-accounting platform (founded 2020) with financial-sector/PCAF depth. |
| **Watershed** | Enterprise carbon platform (founded 2019, ~$185M raised) — measurement, reporting, marketplace; Verdantix 2026 leader. |
| **Envizi** | ESG data/analytics firm acquired by IBM (announced January 2022 — often mis-dated 2021). |
| **Net Zero Cloud** | Salesforce's carbon/ESG module — Scope 1/2/3, supplier engagement, Einstein AI factor matching. |
| **Cloud for Sustainability** | Microsoft's sustainability platform (announced July 2021) on Azure/Power Platform with the ESG data model. |
| **Greenly** | French SME-focused carbon-accounting platform (founded 2019). |
| **Normative** | Swedish carbon-accounting platform (founded 2014) — the "carbon engine" factor knowledge base. |
| **Plan A** | Berlin-based corporate carbon/ESG platform (founded 2017); not the UK charity of the same name. |
| **Isometric** | Carbon-removal-only registry (founded ~2022) — 1 certificate = 1 tCO₂e removed, provenance-first, agentic-certification AI verifiers. |
| **Pachama** | Forest-carbon MRV company — satellite/LiDAR/AI measurement; Verra digital-MRV pilot partner. |

---

## 11. The Verification Ledger

**✅ Verified live this pass** (via web search/extraction against primary or named sources):

| Claim | Source | Result |
|---|---|---|
| CSRD = Directive (EU) 2022/2464, adopted 14 Dec 2022 | EUR-Lex; European Commission | ✅ — first wave FY2024, reports 2025; ~50,000-company scope (Commission-estimated) |
| SEC climate rules adopted 6 Mar 2024 | SEC.gov press releases & statements | ✅ — stayed 4 Apr 2024; defense ended 27 Mar 2025; rescission proposed (2026 press release) |
| ISSB issued IFRS S2 on 26 Jun 2023, effective 1 Jan 2024 | IFRS.org | ✅ |
| TCFD final recommendations 29 Jun 2017 | FSB / fsb-tcfd.org | ✅ — 11 recommendations, four pillars; disbanded 2023 |
| SGX/ACRA mandate: listed issuers FY2025, large non-listed FY2027, announced 28 Feb 2024 | ACRA–SGX RegCo joint media release | ✅ — ISSB-aligned |
| SBTi Corporate Net-Zero Standard published 2021 | sciencebasedtargets.org | ✅ |
| EU ETS launched 2005; CORSIA adopted 2016 (ICAO Res. A39-3) | EASA/ICAO documentation | ✅ |
| NZBA launched April 2021, 43 founding banks, UNEP FI | unepfi.org; Santander press | ✅ — plus 2025–26 withdrawal wave (press + academic paper) |
| PCAF: global launch Sep 2019; Global GHG Standard Nov 2020 | carbonaccountingfinancials.com; GHG Protocol site | ✅ |
| Persefoni founded 2020 (Kawamori/Offerman/Stroh); "AI-native" positioning | startupintros; persefoni.com | ✅ |
| Watershed founded 2019; ~$185M raised; Verdantix 2026 Green Quadrant leader | tracxn; esgtechreport; watershed.com | ✅ — customer/tonnage scale claims ⚠ company-reported |
| IBM acquired Envizi — announced 11 Jan 2022 | IBM Newsroom / PR Newswire | ✅ — **correction: 2022, not 2021** |
| Salesforce Net Zero Cloud Scope 1/2/3 + Einstein AI | salesforce.com | ✅ |
| Microsoft Cloud for Sustainability announced Jul 2021 | Microsoft blogs | ✅ |
| Isometric removal registry, 1 certificate = 1 tCO₂e | registry.isometric.com | ✅ — founding year ⚠ (press) |
| Pachama satellite/AI forest MRV; Verra digital-MRV pilot | verra.org; netzerocompare | ✅ — commercial details ⚠ |

**⚠ Flagged — could not be fully verified this pass (web backend degraded; claims reported but unconfirmed):**

| Claim | Flag |
|---|---|
| Market-size figures (four analyst houses) | ⚠ Inconsistent (~2× disagreement on base year); direction credible, levels not — treat as a range (§1.2) |
| SEC rule "final" status | ⚠ Litigation tail + proposed rescission may not be fully closed — status as of Aug 2026 stated |
| Watershed 500+ customers / 1 Gt managed; 100% audit-pass rate | ⚠ Company-reported scale claims |
| Sweep founded 2020; "3% of global emissions data" | ⚠ Press + company-reported |
| Greenly 2019, Normative 2014, Plan A 2017 founding years | ⚠ Widely reported, not re-verified this pass |
| Isometric founded 2022; Eamon Jubbawy founder | ⚠ Press-reported |
| SAP product-line naming/GA dates | ⚠ Shifted 2023–25; not re-verified |
| ISO 14064/14065/14067 edition years | ⚠ Stable standard facts, not re-verified this pass |
| EU ETS ETS2 extension phasing; recent price levels | ⚠ Volatile/evolving |
| Gold Standard 2003 founding; LMA/APLMA green-loan principles details | ⚠ Standard history, not re-verified |
| Guardian 2023 Verra offset investigation | ⚠ Press-reported; contested findings |
| SG local carbon-software vendor landscape | ⚠ No authoritative directory found; Cedars Digital claims are company-reported (see cedars guide) |
| NZBA withdrawal specifics (which banks, exact dates) | ⚠ Press + academic corroboration of the exodus; names/timeline not individually verified |

**Method note:** web access was partially degraded during this pass (the search/extract backend returned empty results for several queries across three retry rounds); every claim that could not be re-verified live is flagged above rather than asserted. No market sizes, funding figures, dates, or customer counts were fabricated; where sources disagreed (market size, Envizi's acquisition date), the disagreement is reported in the text.

---

*End of guide — the measured footprint. Nothing in this guide is investment, legal, or procurement advice.*
