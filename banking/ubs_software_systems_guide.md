# UBS: The Software Systems Landscape — A Comprehensive Guide to the Technology the World's Largest Wealth Manager Runs

*A companion deep-dive to [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md), [UOB Software Systems Guide](uob_software_systems_guide.md), [SMBC Software Systems Guide](smbc_software_systems_guide.md), [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) and [BNP Paribas Software Systems Guide](bnp_paribas_software_systems_guide.md) (the pattern for this series) and [Standard Chartered Guide](standard_chartered_guide.md) (the structural model). This guide focuses on the **specific software and technology systems** behind UBS Group AG — Switzerland's largest bank, the world's largest private bank, and (since the 2023 Credit Suisse rescue) the operator of the biggest wealth-management technology consolidation in banking history: the core banking estate (proprietary, internally developed — with the Avaloq misconception corrected), the Credit Suisse IT integration (the defining systems event of the 2020s), the wealth-management platforms (the crown-jewel estate), the investment-bank systems (the UBS Neo client platform), digital banking (the Key4 digital line), AI and innovation (the genAI programme), cloud (the Microsoft Azure strategy), security and fraud, and the Singapore angle (the APAC wealth hub) — what is publicly known, what is vendor-verified, what is inferred from industry practice, and what UBS simply does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary/secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §13](#13-claims-status-and-verification-notes).**

**Research-method note: web_extract was degraded (search-only backend) for this pass, so all evidence comes from targeted search surfaces (14 searches) — ubs.com surfaces (the corporate site, the media/newsroom pages, the "Innovation and AI" technology page, the cloud pages, the client-logins page, the fraud-prevention page), UBS annual-report material (the Annual Report 2023 and 2025 disclosures surfaced via search), Microsoft's UBS customer story, and press coverage (Reuters/Business Insider for the March 2023 rescue, Citywire/InvestmentNews for the GWM reorganisation, The Business Times for the Singapore AI pilot, Netzwoche for the genAI usage numbers, finews for the integration milestones). Anything that could not be verified is flagged ⚠ honestly — no system name or vendor fact in this guide is fabricated.**

### Series Context: Where This Guide Sits

This is the **dedicated deep-dive on software systems in UBS** in the bank-software-systems series — the first guide on a *Swiss* bank, the largest wealth-management franchise in the world, and the series' clearest example of **merger-driven estate consolidation** (the 2023 Credit Suisse acquisition). How it relates to the siblings:

- **The pattern** — [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md), [UOB Software Systems Guide](uob_software_systems_guide.md), [SMBC Software Systems Guide](smbc_software_systems_guide.md), [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) and [BNP Paribas Software Systems Guide](bnp_paribas_software_systems_guide.md) set the format: ✅/⚠ verification flags, claims-status audit, worked customer journey, glossary. The Deutsche Bank guide is the closest cousin — it too covers a European universal bank absorbing a second estate (Postbank) — and the BNP guide covers a wealth-strong European group; this guide's *wealth-first* lens is the differentiator.
- **The contrast class** — the Singapore banks run packaged or home-grown cores over a *single-franchise* retail model ([OCBC Software Systems Guide](ocbc_software_systems_guide.md), [UOB Software Systems Guide](uob_software_systems_guide.md)); the Japanese megabank builds in-house with an external alliance ([SMBC Software Systems Guide](smbc_software_systems_guide.md)); Deutsche Bank migrated its retail core to SAP ([Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §2). **UBS is the double-estate consolidation case**: two complete, rival banking platforms (UBS + Credit Suisse) being collapsed into one over 2023–2026, on top of the world's largest private-banking client franchise.
- **The core umbrella** — [Core Banking Systems Guide](core_banking_systems_guide.md) is the vendor/platform taxonomy this guide's §2 slots into. The Temenos guides ([Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md)) are **not** UBS-adjacent (verified negative — see §2.1); the Swiss wealth-core class belongs to **Avaloq** (structural knowledge, flagged — see §2.1). The [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) is the treasury-platform reference for the capital-markets angle of §5.
- **The Singapore angle is this guide's differentiator** — like SMBC and Deutsche Bank, UBS's Singapore operation is the group's **Asia-Pacific wealth hub** (§10): the region's largest private bank, a 400,000 sq ft APAC headquarters at 9 Penang Road, and the launch market for the group's adviser-AI pilot. Unlike the Deutsche Bank SG section (wholesale-only), UBS Singapore is a *wealth-retail-plus-wholesale* hub — so the worked journey in §11 is a genuine private-banking client flow.
- **The ⚠ headline findings of this guide** — (a) the widely-circulated premise that UBS's core runs on **Avaloq** is **not verified** anywhere in this pass, and Avaloq's own origin story (founded 1985 as BZ Informatik, an IT subsidiary of **BZ Bank**, not UBS) contradicts the "Avaloq came from UBS" variant of the rumour (§2.1); UBS's core is best described as **proprietary and internally developed** (⚠ structural). (b) The **scale numbers of the Credit Suisse IT integration** (applications decommissioned, costs, savings) are partially verifiable and partially not — the verified anchors are ~85% of Swiss-booked CS client accounts migrated (Q4 2025 disclosure) and full decommissioning targeted by **December 2026** (§3). This guide says so plainly rather than repeating unverified figures.

---

## Table of Contents

1. [UBS Overview](#1-ubs-overview)
2. [Core Banking Systems](#2-core-banking-systems)
3. [The Credit Suisse Integration (IT Integration Programme)](#3-the-credit-suisse-integration-it-integration-programme)
4. [Wealth Management Systems](#4-wealth-management-systems)
5. [Investment Bank Systems](#5-investment-bank-systems)
6. [Digital Banking Systems](#6-digital-banking-systems)
7. [AI and Innovation](#7-ai-and-innovation)
8. [Cloud](#8-cloud)
9. [Security and Fraud Systems](#9-security-and-fraud-systems)
10. [The Singapore Angle](#10-the-singapore-angle)
11. [Worked Example: A UBS Wealth Client Journey](#11-worked-example-a-ubs-wealth-client-journey)
12. [Summary: The Swiss Wealth Giant's Modern Stack](#12-summary-the-swiss-wealth-giants-modern-stack)
13. [Claims Status and Verification Notes](#13-claims-status-and-verification-notes)
14. [Glossary](#14-glossary)
15. [References and Further Reading](#15-references-and-further-reading)

### Reading Map (How This Guide Connects to the Series)

For a reader coming from the sibling guides, the fastest orientation:

- **The pattern guides** answer "how does a modern bank digitise?" — this guide answers "how does the world's largest wealth manager absorb a rival bank's entire technology estate?" The verification convention (✅/⚠), the worked journey (§11) and the claims-status audit (§13) are identical to the pattern.
- **The umbrella guides** — [Core Banking Systems Guide](core_banking_systems_guide.md) (core taxonomy), [Payments Hub Guide](payments_hub_guide.md) (rails), [Wealth Management Guide](wealth_management_guide.md) (the wealth-platform class of §4), [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) (the treasury/capital-markets class of §5), [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) and [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) (the security/compliance class of §9).
- **The technology-tree guides** (`../technology/` prefix) — the AI governance and enterprise-platform material ([Enterprise AI Platforms Guide](../technology/ai_llm/enterprise_ai_platforms_guide.md), [AI Governance, Bias & Red-Teaming Guide](../technology/ai_llm/ai_governance_bias_redteaming_guide.md)) and the integration patterns ([Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md)) back the §3 and §7 analysis.
- **The honest flags to hold** — the Avaloq/UBS core link is unverified (§2.1); the integration scale numbers are partly unverified (§3.1); the wealth-platform detail is thin for the US digital offering (§4); the IB markets stack beyond Neo is opaque (§5); the genAI specifics are partly single-source (§7); the Singapore depth is a mix of verified and flagged (§10).

---

## 1. UBS Overview

### 1.1 The Scope: What This Guide Covers

This guide is the **software-systems deep-dive for UBS Group AG** — the dedicated member of the bank-software-systems series covering Switzerland's largest bank, the world's largest private bank and wealth manager, and the operator of the largest financial-services merger of the 2020s. The *bank* — history, business segments, strategy — is covered here at the level needed to anchor the systems map; the deep strategy and financial analysis belongs to the sibling [Universal Banking Model Guide](universal_banking_model_guide.md) (the group-structure reference) and the [DBS Bank Guide](dbs_bank_guide.md) (the comparative Asian-bank reference). The division of labour:

| Topic | Where it lives |
|---|---|
| The bank, history, divisions, financials, leadership | This guide §1 (systems-anchoring summary) + [Universal Banking Model Guide](universal_banking_model_guide.md) |
| **The specific software systems: core, CS integration, wealth, investment bank, digital, AI, cloud, security** | **This guide** (§2–§9) |
| The vendor/platform classes these systems belong to | [Core Banking Systems Guide](core_banking_systems_guide.md), [Payments Hub Guide](payments_hub_guide.md), [Wealth Management Guide](wealth_management_guide.md), [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md), [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) |
| The Temenos-side cores (T24, TAFJ) — the *contrast* case | [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md) — **UBS is NOT Temenos-adjacent** (see §2.1); the Swiss wealth-core class is Avaloq's ⚠ |
| Core-banking mechanics (interest, posting, processes) | [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md) |
| The Singapore angle (APAC wealth hub) | This guide §10 (dedicated section — the series convention since the SMBC guide) |
| The insurance-adjacent wealth/AM angle | [Insurance Software Systems Guide](insurance_software_systems_guide.md) (lightly — UBS Asset Management is not an insurer; cross-ref for the AM back-office class only) |

What is covered here, section by section: the **UBS overview** — the 1998 merger of Union Bank of Switzerland and Swiss Bank Corporation, the group, the four divisions (+ Non-core and Legacy), the 2023 Credit Suisse rescue, and the technology organization (§1); the **core banking estate** — proprietary and internally developed, with the Avaloq misconception corrected (§2); **the Credit Suisse integration** — the IT integration programme, its verified milestones and its flagged scale numbers (§3); **wealth management systems** — the crown-jewel estate of the world's largest wealth manager (§4); **investment bank systems** — the UBS Neo client platform and the markets estate (§5); **digital banking** — the Key4 digital line and the UBS Mobile Banking app (§6); **AI and innovation** — the genAI programme, the Copilot deployment and the adviser-AI pilot (§7); **cloud** — the Microsoft Azure strategy (§8); **security and fraud** — the fraud-prevention estate and the integration-era security risk (§9); the **Singapore angle** — UBS's APAC wealth hub (§10); a **worked customer journey** through the UBS wealth estate with a Credit Suisse-migration tail (§11); a **one-page summary** (§12); the honest **claims-status audit** (§13); a **glossary** (§14); and **references** (§15).

### 1.2 The History: Founded 1998 — the Merger of UBS and SBC

The founding fact an architect needs before anything else: **UBS was formed in 1998 by the merger of two of Switzerland's largest banks — the Union Bank of Switzerland and the Swiss Bank Corporation (SBC)** ✅ (universally documented — Britannica: "major bank formed in 1998 by the merger of two of Switzerland's largest banks, the Swiss Bank Corporation and the Union Bank of Switzerland"; Wikipedia and UBS's own corporate-history material, including the UBS Virtual Museum, agree). Two details matter for the systems story:

- **The name** ✅ — although "UBS" derives from "Union Bank of Switzerland", it is not an acronym; the merged bank is known solely as "UBS" and kept the **Swiss Bank Corporation's "three keys" icon** ✅ (the SBC brand heritage — Wikipedia). The 1998 entity was UBS AG, later re-holding as **UBS Group AG** (2014 reorganisation ⚠ structural).
- **The systems inheritance** ⚠ structural — the 1998 merger itself was a *systems* merger: two rival Swiss banking estates (SBC's — which included the investment-bank platform bought from Dillon, Read & Co. in 1997 ⚠ structural, and UBS's) had to be consolidated. The 2023 Credit Suisse acquisition (§1.4, §3) is therefore the *second* major estate consolidation in UBS's 25-year history — an architect modelling the current landscape should hold both mergers in mind, because the "keep the winner's platform, decommission the loser's" pattern of 1998 is being repeated at vastly larger scale in 2023–2026 (§3).

The deeper lineage of the two 1998 merger partners (⚠ structural history, flagged as not re-verified this pass):

- **Swiss Bank Corporation (SBC)** — founded **1854 in Basel** ⚠; the more internationally-minded of the two, with a strong investment-banking culture (the SBC Warburg era of the 1990s ⚠).
- **Union Bank of Switzerland** — formed **1912** from the merger of the Bank in Winterthur (1862) and the Toggenburger Bank (1863) ⚠; the larger Swiss retail and private-banking franchise ⚠.
- **The merged bank's home market** ✅ — UBS is headquartered in **Zurich and Basel** (the current corporate description of the group's HQ arrangement ✅ — Wikipedia/Britannica characterisation; the legal seat is Zurich with Basel as a second registered seat ⚠).

**The systems inheritance of 25 years (and of the two predecessor banks)** ⚠ structural — UBS's estate is the product of: a 1998 merger of two Swiss giants, a 1990s investment-bank build-out (SBC's Dillon Read acquisition, the SBC Warburg franchise ⚠), the 2008 financial crisis (the UBS subprime losses and the subsequent re-focus on wealth management ⚠ structural), the 2012 "strategic decision" to shrink the investment bank around wealth management ⚠, and the 2023 Credit Suisse rescue (§1.4). The consequence is an estate with **multiple core generations running side by side** — proprietary Swiss retail/wealth cores (§2), a global wealth-management platform estate (§4), an investment-bank client platform (Neo, §5), and a Credit Suisse estate being folded in and decommissioned (§3). UBS's modernisation problem is not "rip and replace"; it is **orchestrated coexistence of two full bank estates during a three-year collapse** ⚠.

### 1.3 UBS Today: The Group and the Divisions

UBS Group AG is **Switzerland's largest banking institution and the world's largest private bank** ✅ (structural — the standard characterisation in every reference consulted: Wikipedia describes it as "the largest Swiss banking institution and the world's largest private bank" managing "the largest amount of private wealth in the world"). Post-Credit Suisse, the group's invested assets are of the order of **USD 5+ trillion** ✅/⚠ (the March 2023 deal announcement itself claimed a combined business "with more than USD 5 trillion in total invested assets" ✅ — the deal-claim figure; the current figure moves with markets and is flagged). The group, verified at the level that matters for the systems map:

- **UBS Group AG** — the parent (Swiss bank holding company), headquartered Zurich/Basel ✅; the operating bank is UBS AG, which absorbed Credit Suisse AG in the integration (§3) ⚠ structural.
- **The divisions** — UBS reports **four business divisions plus a Non-core and Legacy unit** ✅ (verified this pass via the UBS Group Annual Report 2023 structure: "We operate through five business divisions: Global Wealth Management, Personal & Corporate Banking, Asset Management, the Investment Bank and Non-core and Legacy" — the four-division naming in the series brief is verified, with the Non-core and Legacy (NCL) unit as the fifth, which is itself a *systems* fact: NCL holds the legacy/run-off books and their systems):

| Division | What it is | Systems relevance |
|---|---|---|
| **Global Wealth Management (GWM)** | The world's largest wealth-management franchise: advice, discretionary mandates, brokerage, lending for HNW/UHNW clients worldwide | The crown-jewel estate: client platforms, advisory tooling, the AI-KYC tool (§4, §7); the largest client-migration target of the CS integration (§3) |
| **Personal & Corporate Banking (P&C)** | UBS's Swiss domestic bank: retail, SME and corporate clients in Switzerland | The Swiss retail core (§2), the Key4 digital line and UBS Mobile Banking app (§6), TWINT payments ⚠ |
| **Asset Management (AM)** | Investment funds, institutional mandates (UBS Asset Management) | Its own fund-administration estate ⚠; AM portfolios fully migrated from CS by end-2025 ✅ (§3) |
| **Investment Bank (IB)** | Global markets (FX, rates, credit, equities), advisory, financing — "a more targeted Investment Bank" than the pre-2012 era | The UBS Neo client platform (§5), the markets/execution estate (§5), the Murex-class treasury angle ⚠ |
| **Non-core and Legacy (NCL)** | Run-off/legacy books (incl. CS legacy exposures) | The run-down estate — systems being retired rather than migrated ⚠ structural |

The strategic posture to hold: UBS is the **wealth-first universal bank** — the one whose centre of gravity is Global Wealth Management (the group "whose center of gravity is wealth management", per the structure summaries reviewed this pass), with P&C as the Swiss domestic engine, AM as the institutional engine, and the IB "more targeted" than its pre-2012 self. The 2023 Credit Suisse acquisition changed the estate arithmetic: UBS roughly doubled in size overnight (⚠ structural — the combined group is ~120,000 employees ⚠, against ~72,000 for UBS alone pre-merger ⚠; exact figures flagged) and inherited a *second complete banking platform*. Every systems section below (§2–§9) is coloured by that doubling: the core story is consolidation (§2–§3), the wealth story is migration (§4), and the modernisation story (AI + cloud, §7–§8) is being delivered *while* the integration runs.

#### The Scale and Financial Context (⚠ Flagged)

The financial and operational context an architect needs before reading the systems map (flagged ⚠ — the exact figures move with each reporting period and were not re-verified this pass):

- **Balance-sheet scale** ⚠ — UBS is one of the largest banks in Europe by total assets (the combined group's balance sheet is of the order of USD 1.5–1.7 trillion ⚠); the precise figure is flagged. The scale matters because *every* system in §2–§9 is sized for a global balance sheet: the world's largest wealth-management client book, a top-tier markets franchise (§5), and a Swiss domestic bank serving millions of clients (§6).
- **The two-estate problem** ✅ — the defining feature of the current era: UBS + Credit Suisse = two complete technology estates being collapsed into one (§3). This is *the* difference between UBS and every other bank in this series: DBS, OCBC, UOB, SMBC, Deutsche Bank and BNP Paribas are modernising one estate; **UBS is decommissioning an entire rival estate in three years** ✅ (the December 2026 decommissioning target, §3.1).
- **The cost arithmetic** ⚠ — UBS has publicly guided multi-year integration costs (of the order of **USD 10 billion** pre-tax ⚠ — reported, not re-verified this pass) against gross savings targets of the order of **USD 13 billion by 2027** ⚠ (reported figures; flagged). The tension — *spend on the merge while cutting the combined cost base* — is the single best explanation for the shape of the 2023–2026 estate (§3).
- **Headcount and technology workforce** ⚠ — ~120,000 employees post-merger ⚠ (structural, flagged); the technology workforce is a meaningful share (the group CIO organisation plus partner capacity; exact numbers not public).

### 1.4 The Credit Suisse Acquisition (✅ Verified — the 2023 Rescue)

The defining corporate event of UBS's modern era — and the defining *systems* event of this guide — is the **March 2023 emergency acquisition of Credit Suisse**:

- **The announcement** ✅ — on **19 March 2023**, UBS announced it planned to acquire Credit Suisse ("UBS to acquire Credit Suisse", the UBS media release of that date ✅); the deal was presented as creating "a business with more than USD 5 trillion in total invested assets" ✅ (the UBS announcement's own claim).
- **The rescue nature** ✅ — this was a **state-sponsored rescue takeover**: "The state-sponsored rescue takeover of Credit Suisse had just one aim: to prevent an imminent collapse of the Swiss banking system" (Euromoney ✅); UBS itself confirmed the takeover discussions were **initiated by Swiss authorities** (Business Insider, quoting UBS's statement ✅). The deal was brokered by the Swiss regulator and central bank over the weekend of 18–19 March 2023, after Credit Suisse's liquidity crisis (the collapse of its share price following the Silicon Valley Bank shock and the collapse of its largest shareholder's plans ⚠ structural).
- **The terms** ✅/⚠ — an all-share acquisition of roughly **CHF 3 billion** (~USD 3.2 billion / £2.7 billion per the UK press ✅/⚠ — the Standard: "UBS agreed to pay around £2.7 billion for its former rival"); the exact per-share ratio and the accompanying **SNB liquidity support** (a CHF 100+ billion-class facility ⚠ structural) and the **CHF 16 billion-class AT1 writedown** ⚠ (the Credit Suisse Additional Tier 1 bonds written to zero — a widely reported fact, flagged as not re-verified this pass) are the financial context an architect needs for the *risk* side of the systems story.
- **The completion** ⚠ — the acquisition completed in **June 2023** ⚠ (widely reported; UBS took over Credit Suisse AG as a subsidiary); the City AM coverage reviewed this pass confirms UBS "acquired Credit Suisse in June" ✅/⚠ (the first full quarter of results after the June 2023 acquisition — City AM ✅).
- **The integration timeline** ✅/⚠ — UBS has guided: most Swiss client migrations and **all asset-management portfolios migrated by end-2025** ✅ (Rio Times coverage of UBS's guidance); **~85% of Swiss-booked Credit Suisse client accounts migrated** as of the Q4 2025 disclosure ✅; and **full decommissioning of remaining Credit Suisse systems by December 2026** ✅ (integration updates; multiple sources this pass). The system-by-system story is §3.

#### The Acquisition Timeline at a Glance

| Date | Event | Status |
|---|---|---|
| March 2023 | Credit Suisse liquidity crisis; the SVB-shock week; UBS discussions initiated by Swiss authorities | ✅ rescue narrative verified (Euromoney, Business Insider) |
| **19 March 2023** | **"UBS to acquire Credit Suisse" announced** — all-share ~CHF 3bn; "USD 5 trillion combined invested assets" claim | ✅ UBS media release |
| March–June 2023 | Regulatory approvals; the AT1 writedown (CHF 16bn-class ⚠); SNB liquidity support ⚠ | ⚠ structural (terms not re-verified) |
| **June 2023** | **Acquisition completed**; Credit Suisse AG becomes a UBS subsidiary | ✅/⚠ City AM ("acquired Credit Suisse in June") |
| 2023–2025 | Integration waves: client migrations, dual-run, decommissioning; UBS reports progress quarterly | ✅ milestones (§3.1) |
| End-2025 | Most Swiss client migrations + all AM portfolios targeted complete | ✅ UBS guidance (Rio Times) |
| Q4 2025 | ~85% of Swiss-booked CS accounts migrated | ✅ Q4 2025 disclosure |
| **Dec 2026** | **Full decommissioning of remaining CS systems** | ✅ integration updates |

The architect's read: the CS acquisition is **not a footnote in UBS's systems history — it is the current systems strategy**. Every modernisation decision UBS makes in 2023–2026 (which platform survives, which gets decommissioned, where the AI/cloud budget goes) is made against the backdrop of collapsing two estates. The rest of this guide is organised around exactly that.

### 1.5 The Overview Table: Aspect and Description

| Aspect | Description |
|---|---|
| **Legal identity** | UBS Group AG — formed **1998** by the merger of the Union Bank of Switzerland and the Swiss Bank Corporation ✅; HQ Zurich and Basel ✅; the world's largest private bank ✅ |
| **Group** | Four divisions: **Global Wealth Management, Personal & Corporate Banking, Asset Management, Investment Bank** ✅ (+ **Non-core and Legacy** ✅, Annual Report 2023); ~120,000 employees ⚠ post-CS |
| **The CS acquisition** | **Credit Suisse rescue takeover** announced 19 March 2023 ✅; brokered by Swiss authorities ✅; all-share ~CHF 3bn ✅/⚠; "USD 5 trillion combined invested assets" deal-claim ✅; completed June 2023 ⚠ |
| **Core banking** | **Proprietary, internally developed** Swiss retail/wealth cores ⚠ structural; **Avaloq-UBS link NOT verified** ⚠ (§2.1 — corrects the misconception); the CS estate being decommissioned by Dec 2026 ✅ |
| **IT integration** | The **Credit Suisse integration programme**: ~85% of Swiss-booked CS accounts migrated (Q4 2025) ✅; all AM portfolios by end-2025 ✅; full decommissioning by Dec 2026 ✅; cost/savings figures ⚠ (§3) |
| **Wealth systems** | The crown-jewel estate of the world's largest wealth manager: GWM client platforms, the UBS Mobile Banking app family, Key4 wealth ⚠, digital-advice offerings ⚠ (§4) |
| **Investment bank** | **UBS Neo** — the cross-asset IB client platform ✅ (built with Lab49; the "Investment Bank Client Portal" per ubs.com) (§5); the markets estate ⚠ |
| **Digital channels** | **UBS key4** — the purely digital product line launched 19 May 2022 ✅ (key4 banking / key4 wealth brands ⚠); the long-established UBS Mobile Banking app ✅ (§6) |
| **AI** | "AI enabled financial institution" ambition ✅ (ubs.com); Microsoft 365 Copilot at 50,000 licences ⚠; 280+ active AI use cases and 8M prompts in Q2 2025 ✅/⚠; Singapore adviser-AI pilot (The Business Times, Oct 2025) ✅ (§7) |
| **Cloud** | **Microsoft Azure as primary cloud** ✅; 2018 cloud strategy (one-third of apps to public cloud in four years — achieved Feb 2021 ✅); Oct 2022 expansion: >50% of applications incl. critical workloads on Azure ✅ (§8) |
| **Security** | Client-facing fraud protection (scams, cyber fraud, identity theft, check fraud) ✅ (ubs.com fraud-prevention page); integration-era security risk ⚠; FINMA supervision ⚠ (§9) |
| **Singapore** | **APAC wealth hub** ✅; 9 Penang Road office (~400,000 sq ft — UBS's largest in Asia Pacific) ✅; adviser-AI pilot market ✅; regional-HQ relocation reports ⚠ (§10) |

### 1.6 The Technology Organization

The human system that runs the software estate is more opaque than the Singapore banks' (contrast [DBS Software Systems Guide](dbs_software_systems_guide.md) §1.5 and [UOB Software Systems Guide](uob_software_systems_guide.md) §1) but more visible than SMBC's. The shape is clear:

- **CIO-led transformation** ⚠ — UBS runs a group Chief Information Officer organisation (the current Group CIO is Mike Dargan ⚠ — structural knowledge, flagged as not re-verified this pass); the CIO office is the centre of gravity for the CS integration (§3), the cloud strategy (§8) and the AI programme (§7). UBS does not publish a DBS-style named-technology-executive roster in the material reviewed this pass ⚠.
- **The CEO with a technology mandate** ✅ — **Sergio Ermotti** returned as Group CEO in **April 2023** ⚠ (widely reported, structural) specifically to run the Credit Suisse integration; his public statements on integration milestones (e.g., confidence on meeting integration goals, Rio Times ✅) make the integration a *CEO-level* systems programme — the same pattern seen in Deutsche Bank's CTO-led transformation ([Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §1.4) but with the CEO personally accountable.
- **The partner layer** ✅ — **Microsoft** is the verified strategic technology partner (cloud §8, Copilot/AI §7 ✅); the GWM co-presidents (Iqbal Khan and Rob Karofsky) and the 2024 "GWM Solutions" reorganisation (Citywire/InvestmentNews ✅) show the *business-side* technology agenda (bringing client solutions into a single unit) running alongside the CIO's estate consolidation.
- **The marketing of modernisation** ✅/⚠ — UBS publishes technology narratives on ubs.com (the "Innovation and AI" page ✅, the "Cloud as an opportunity" page ✅, the "Our history" virtual museum ✅) but does *not* publish the system-level detail the German press extracted from Deutsche Bank's Postbank crisis; the integration numbers come from investor disclosures and press coverage, not from a UBS technology blog ⚠.

The architect's read: UBS's technology organization is a **CIO-led consolidation machine with a CEO-level mandate** — Microsoft for the platform layer (cloud + AI), a proprietary core estate (§2), a captive engineering force ⚠ structural (UBS runs group engineering largely in-house with vendor augmentation), and a three-year decommissioning programme (§3) that is the largest the series has seen.

### 1.7 The Business Segments (⚠ Structural)

The segment map behind the systems estate follows the verified division structure of §1.3; the revenue-level detail is standard annual-report material ⚠ (flagged as not re-verified this pass):

| Segment | What it is | Systems relevance |
|---|---|---|
| **Global Wealth Management (GWM)** | The world's largest private bank: advice, mandates, brokerage, lending | The wealth platforms (§4), the AI-KYC tool (§7), the CS client migrations (§3) |
| **Personal & Corporate Banking (P&C)** | Swiss retail + SME/corporate banking | The Swiss retail core (§2), Key4 (§6), TWINT ⚠ |
| **Asset Management (AM)** | Funds and institutional mandates | AM portfolio migration (done end-2025 ✅); fund-administration estate ⚠ |
| **Investment Bank (IB)** | Markets + advisory + financing | UBS Neo (§5), markets estate (§5) |
| **Non-core and Legacy (NCL)** | Run-off/legacy books | Run-down systems; the CS legacy tail ⚠ |

The systems consequence is the same as every universal bank in this series (compare [OCBC Software Systems Guide](ocbc_software_systems_guide.md) §1.4 and [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §1.7): **each division runs its own application landscape on shared group infrastructure** — identity, data, cloud, security — with the verified difference that at UBS the *wealth* estate is the crown jewel (§4), the *domestic* estate is the digital showcase (Key4, §6), and the *integration* estate (CS, §3) touches everything.

---

## 2. Core Banking Systems

### 2.1 The Core Question: What Does UBS Run On? (⚠ The Vendor Is Flagged)

The single most important verification result of this guide is a **correction with an honest flag**: the premise circulating in parts of the industry coverage — that UBS's core banking runs on **Avaloq** (Switzerland's flagship wealth-management core vendor) — is **not supported by any source found in this research pass**, and the origin-story variant of the rumour is contradicted by Avaloq's own history:

- **What Avaloq actually is** ✅ — Avaloq was **founded in Zürich in 1985 under the name "BZ Informatik Aktiengesellschaft", functioning as the information technology subsidiary of BZ Bank** (Avaloq's own history page and Wikipedia ✅); the founder is Francisco Fernandez; a partial management buyout followed in 1991 ✅. Avaloq (renamed in 2006 ⚠ structural) is now "an industry-leading provider of wealth management technology and services" — the Avaloq Core Platform serves banks and wealth managers with "full coverage of over 100 financial products", available in the cloud (BPaaS/SaaS) ✅ (Avaloq platform pages). **The "Avaloq came out of UBS" version of the rumour is factually wrong** ✅ — Avaloq's documented parent was BZ Bank, not UBS.
- **Whether UBS *uses* Avaloq is NOT verified** ⚠ — targeted searches for UBS + Avaloq produced **no** primary or secondary source confirming UBS runs Avaloq as its core. Avaloq is verifiably the dominant *Swiss private-banking* core class (structural — the platform is used by a large share of Swiss private banks and international wealth managers ⚠), but an architect must **not** model UBS's core as Avaloq-based on the evidence of this pass. **Honest negative, stated plainly.**
- **What UBS's core actually is (best evidence)** ⚠ structural — UBS runs **proprietary, internally developed core banking systems** for its Swiss retail/wealth franchises: the UBS Switzerland banking platform, the wealth-management booking platforms, and the global wealth-management platform estate (§4) are UBS-built ⚠ (structural knowledge from the bank's history of in-house engineering — the 1998 merger consolidated SBC's and UBS's own proprietary platforms, and UBS has remained a primarily *build* bank, unlike Deutsche Bank's SAP buy (§2.1 of the Deutsche guide) or the packaged-core Temenos banks). The specific module names, the mainframe-versus-distributed split, and the database estate are **not publicly enumerated — flagged ⚠**.

The systems consequence: **UBS's core banking is the proprietary-estate consolidation story, not a packaged-core story** — which is why the 2020s modernisation is about *merging two proprietary estates* (UBS's + CS's, §3), *cloud migration of that estate* (Azure, §8) and *AI on top of it* (§7), not a greenfield core replacement. The contrast with the series: DBS and the Singapore banks run modern or modernising cores ([DBS Software Systems Guide](dbs_software_systems_guide.md) §2); Deutsche Bank migrated to SAP standard software ([Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §2.1); SMBC commissioned a next-generation in-house core ([SMBC Software Systems Guide](smbc_software_systems_guide.md) §2.1); the Temenos banks bought packaged cores ([Temenos Guide](temenos_guide.md)); and **UBS runs and consolidates its own proprietary platforms** ⚠ — the "build and merge" answer. Cross-ref the umbrella taxonomy in [Core Banking Systems Guide](core_banking_systems_guide.md).

### 2.2 The Core Landscape (✅/⚠ Verified + Flagged)

The verified core-landscape story of the 2020s is the **two-estate consolidation** — the systems event that dominates UBS's operational headlines:

- **Two complete estates pre-merger** ✅/⚠ — before March 2023 there were two rival Swiss banking platforms: the UBS estate (proprietary core, wealth platforms, markets estate) and the Credit Suisse estate (its own proprietary core, its own wealth platform, its own markets estate) ✅/⚠ (structural — the existence of two separate estates is a direct consequence of the two banks' separate histories ✅; the internal architecture of each is not public ⚠). Industry estimates put the combined application inventory in the **thousands** ⚠ (reported figures of ~4,000+ applications are flagged — see §3.1).
- **The migration programme** ✅ — UBS's public integration guidance is verified: **most Swiss client migrations and all asset-management portfolios by end-2025** ✅ (Rio Times coverage of CEO Ermotti's statements); **~85% of Swiss-booked Credit Suisse client accounts migrated** as of the Q4 2025 disclosure ✅ (execpartners summary of the Q4 2025 report); **Credit Suisse client assets outside Switzerland migrated onto UBS platforms** (UBS Annual Report 2025 ✅); and **full decommissioning of remaining Credit Suisse systems by December 2026** ✅ (multiple sources this pass).
- **The Swiss domestic core** ⚠ — UBS Switzerland's retail/wealth core (which serves the P&C division and the Swiss GWM book, and which Key4 sits on top of, §6) is the *surviving* Swiss platform: the migration direction is CS-clients-onto-UBS-platforms, not the reverse ✅ (verified by the "migrated onto UBS platforms" language of the disclosures ✅).
- **The Non-core and Legacy (NCL) estate** ⚠ structural — the run-off books (including CS legacy exposures) sit in NCL, and their systems are *run-down rather than migrated* ⚠ — the "retire in place" slice of the landscape.
- **The core estate map for an architect** ⚠ structural — the practical shape: (a) the **UBS proprietary Swiss core** (retail + Swiss wealth) ⚠; (b) the **UBS global wealth-management platform estate** (§4) ⚠; (c) the **UBS markets estate** for the IB (§5) ⚠; (d) the **Credit Suisse estate in migration** — accounts and assets moved wave-by-wave onto (a)–(c), applications decommissioned as the waves close ✅/⚠; (e) the **NCL run-down estate** ⚠. The consolidation programme is the effort to collapse (d) into (a)–(c) — and the December 2026 decommissioning deadline is the hard milestone ✅.

#### The Integration-Era Core Risk (What the Migrations Mean)

For an architect, the UBS core story is a textbook case of **dual-estate migration risk** (the same class as Deutsche Bank's Postbank migration — [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §2.2 — but at roughly double the client scale ⚠):

- **Wave-based client migration** ✅/⚠ — UBS migrates clients in waves (by geography, by client segment, by product set ⚠ — the wave design is not fully public); each wave is a data migration (accounts, positions, mandates, KYC records) plus a channel cut-over plus a reconciliation window ⚠ structural. The Q4 2025 "~85% of Swiss-booked accounts" figure ✅ is the public scoreboard of those waves.
- **Dual-run and decommissioning** ✅/⚠ — while a wave is open, both estates run (dual-run, reconciliation, cut-over), then the CS application is decommissioned; the **December 2026 full-decommissioning target** ✅ is the end-state. The decommissioning *count* (applications retired) is reported by UBS in integration updates ⚠ (figures of 1,000+ decommissioned were reported in 2024–2025 coverage ⚠ — flagged, §3.1).
- **The lesson for the series** ⚠ structural — UBS's core modernisation is *merger-driven*, not strategy-driven: the estate is being consolidated because two banks became one, not because UBS chose a new core. That is why the modernisation budget (integration costs ⚠ §3.1), the cloud migration (§8) and the AI programme (§7) are all running concurrently — the same "modernise while merging" pattern seen in Deutsche Bank's "consolidate first, modernise on top" ⚠ ([Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §2.2), but at a scale no other bank in the series has attempted.

### 2.3 The Core Table: System, Function, Notes

| System | Function | Notes |
|---|---|---|
| **UBS proprietary Swiss core** | Retail/wealth back-end for Switzerland (accounts, deposits, loans, client data) | ⚠ Structural — internally developed; the surviving platform of the CS integration; specific modules not public; what Key4 (§6) sits on ⚠ |
| **UBS global wealth platforms** | GWM booking, portfolio administration, advisory/mandate engines for the world's largest private bank | ⚠ Structural — the crown-jewel estate (§4); the CS client-migration target ✅ |
| **UBS markets estate** | IB trading, execution, risk systems | ⚠ Structural — the IB layer (§5); UBS Neo as the client-facing portal ✅ |
| **Credit Suisse estate (in migration)** | CS's full banking platform — core, wealth, markets, channels | ✅ Migration verified: ~85% of Swiss-booked CS accounts migrated (Q4 2025); all AM portfolios by end-2025; full decommissioning by Dec 2026 ✅ |
| **Non-core and Legacy estate** | Run-off/legacy books (incl. CS legacy) | ⚠ Structural — run down, not migrated |
| **Avaloq Core Platform** | Swiss wealth-management core (vendor class) | **⚠ NOT VERIFIED as UBS's core** — Avaloq is the Swiss private-banking core class ✅/⚠, but no source connects UBS to Avaloq; origin story = BZ Bank, not UBS (§2.1) |
| **Temenos T24/TAFJ** | Packaged cores (contrast class) | **⚠ NOT UBS-adjacent** — verified negative; cross-ref [Temenos Guide](temenos_guide.md) for the class |

The architect's read: UBS's core banking is a **proprietary double-estate consolidation** — internally developed UBS platforms absorbing an entire rival bank's platform over 2023–2026, with the December 2026 decommissioning deadline as the hard milestone ✅ and the vendor map (Avaloq, Temenos) honestly flagged as *not* UBS's core. Compare the greenfield/cloud-native cores in the series ([DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md)) and the packaged-core taxonomy in [Core Banking Systems Guide](core_banking_systems_guide.md).

#### The Core-Estate Comparison Across the Series

| Bank | Core approach | Verified anchor | UBS contrast |
|---|---|---|---|
| **UBS** | Proprietary, internally developed cores; CS estate being absorbed and decommissioned | ✅ Migrations/decommissioning milestones; ⚠ vendor map unverified (§2.1) | The "build and merge" answer |
| **Deutsche Bank** | SAP-standard-software migration (Magellan) + Postbank legacy | [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §2 | Buy-SAP vs build-own; both run merger-driven consolidations |
| **DBS** | Modern, largely in-house core with greenfield ambitions | [DBS Software Systems Guide](dbs_software_systems_guide.md) | Greenfield vs double-estate merge |
| **OCBC / UOB** | Packaged/modernised cores over single-franchise retail | [OCBC Software Systems Guide](ocbc_software_systems_guide.md), [UOB Software Systems Guide](uob_software_systems_guide.md) | Single-franchise vs two-estate |
| **SMBC** | In-house next-gen core with NEC as build partner | [SMBC Software Systems Guide](smbc_software_systems_guide.md) §2.1 | Build-with-partner vs build-and-merge |
| **Temenos banks** | Packaged T24/TAFJ cores | [Temenos Guide](temenos_guide.md), [T24 Programming Guide](t24_programming_guide.md), [TAFJ Guide](tafj_guide.md) | Verified NOT UBS-adjacent (§2.1) |

---

## 3. The Credit Suisse Integration (IT Integration Programme)

### 3.1 The IT Integration Programme: Scale and Numbers (✅ Anchors, ⚠ Flags)

The Credit Suisse IT integration is the **defining systems programme of UBS's 2020s** — the largest bank-platform consolidation of its era. Its scale numbers divide into **verified anchors** and **flagged figures**; this guide is explicit about which is which:

**Verified anchors (✅):**
- **~85% of Swiss-booked Credit Suisse client accounts migrated** as of the Q4 2025 disclosure ✅ (execpartners summary of UBS's Q4 2025 earnings report).
- **All Credit Suisse asset-management portfolios migrated by end-2025** ✅ (UBS guidance via Rio Times coverage of CEO Ermotti's statements).
- **Credit Suisse client assets outside Switzerland migrated onto UBS platforms** ✅ (UBS Annual Report 2025).
- **Full decommissioning of remaining Credit Suisse systems by December 2026** ✅ (UBS integration updates, multiple sources this pass).
- **Most Swiss client migrations targeted for completion by end-2025** ✅ (Rio Times: "UBS targets completion of most Swiss client migrations and all asset management portfolios by the end of 2025"; CEO confidence statement ✅).
- **The deal context** ✅ — announced 19 March 2023; combined "USD 5 trillion invested assets" claim; completion June 2023 (§1.4).

**Flagged figures (⚠ — reported but not re-verified this pass, or structural estimates):**
- **Applications decommissioned** ⚠ — coverage of UBS integration updates reported **1,000+ Credit Suisse applications decommissioned** by the mid-2020s (the LinkedIn enterprise-architecture analysis and integration commentary reviewed this pass reference the decommissioning programme; the exact cumulative count is flagged ⚠ — the December 2026 "shut down all remaining CS applications" target ✅ is the verified bookend).
- **Total application inventory** ⚠ — estimates of the combined estate's size (thousands of applications, order of ~4,000–5,000 across both banks ⚠) are structural estimates, not UBS disclosures; flagged.
- **Integration costs** ⚠ — reported guidance of order **USD 10 billion** of total integration costs and **USD 13 billion of gross savings by 2027** ⚠ (press-reported UBS targets; flagged — the City AM coverage of the first quarter after acquisition records "$2.2bn in expenses largely related to its integration" ✅/⚠ for Q2 2023 alone, which is consistent with a multi-year double-digit-billion programme ⚠).
- **Client counts** ⚠ — the number of Credit Suisse clients migrated runs into the millions ⚠ (structural estimate, flagged; UBS does not publish a running client-migration tally in the material reviewed).
- **US and APAC completion dates** ⚠ — the regional sequencing (which geographies closed first) is flagged; the verified anchor is the Swiss figure (~85% at Q4 2025 ✅).

The architect's read of the numbers: **the integration is a three-year programme (2023–2026) with a verified end-state (December 2026 decommissioning) and a verified scoreboard (85% Swiss accounts at Q4 2025)** ✅ — and a cost/savings arithmetic that is real but only partially public ⚠. Any figure beyond those anchors should be treated as reported-but-unverified.

### 3.2 The Integration Approach (How the Collapse Works)

The shape of the programme, from the verified anchors plus structural inference (⚠ where inferred):

- **UBS platforms as the target estate** ✅ — the verified migration direction is *onto UBS platforms* (the Annual Report 2025 language: CS client assets "migrated onto UBS platforms" ✅); the CS estate is the *decommissioned* estate (the December 2026 shutdown target ✅). UBS's own proprietary platforms (§2) and the wealth/markets estates (§4–§5) are the survivors — the 1998 pattern ("keep the winner's platform") repeated at scale ⚠ structural.
- **Wave-based client migration** ⚠ — clients move in waves by segment/geography/booking centre (structural inference from the milestones; the wave design is not fully public ⚠); each wave is data migration + channel cut-over + reconciliation (the [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md) mechanics, applied at estate scale).
- **Data migration as the risk core** ⚠ structural — client data (KYC, mandates, positions, tax standing, source-of-wealth records) must migrate *cleanly* — which is exactly why the adviser-AI tooling (§7) targets KYC review productivity in the integration era (The Business Times ✅).
- **Dual-run then decommission** ✅/⚠ — while waves are open, both estates run; applications are decommissioned once their last wave closes; the **December 2026** milestone is the "all remaining CS applications shut down" date ✅ (LinkedIn integration commentary: "the bank still needs to fully shut down all remaining Credit Suisse applications by December 2026" ✅/⚠ single-source-ish but consistent with UBS's public end-2026 guidance ⚠).
- **The US angle** ⚠ — the US wealth business (UBS Americas) was reported as reaching its migration milestones early in the programme ⚠ (structural coverage; flagged); the Swiss market is the long tail because of the account complexity (Rio Times: "UBS Tackles Complex Swiss Account Migration" ✅ — headline verified this pass).

### 3.3 The Integration Table: Milestone, Status, Notes

| Milestone / figure | Status | Notes |
|---|---|---|
| Deal announced 19 March 2023 | ✅ Verified | UBS media release; rescue brokered by Swiss authorities |
| Completion (June 2023) | ⚠ Flagged | City AM confirms June 2023 acquisition ✅/⚠ |
| ~85% of Swiss-booked CS accounts migrated | ✅ Verified | Q4 2025 disclosure (execpartners summary) |
| Most Swiss client migrations by end-2025 | ✅ Verified | UBS guidance (Rio Times) |
| All AM portfolios migrated by end-2025 | ✅ Verified | UBS guidance (Rio Times) |
| CS client assets outside Switzerland migrated | ✅ Verified | UBS Annual Report 2025 |
| Full CS system decommissioning by Dec 2026 | ✅ Verified | UBS integration updates; multiple sources |
| 1,000+ applications decommissioned (cumulative) | ⚠ Flagged | Reported in integration commentary; exact count not re-verified |
| Total combined application inventory (~4–5k) | ⚠ Flagged | Structural estimate; not a UBS disclosure |
| Integration costs (~USD 10bn) / savings (~USD 13bn by 2027) | ⚠ Flagged | Press-reported targets; Q2 2023 expenses $2.2bn ✅/⚠ |
| Clients migrated (millions) | ⚠ Flagged | Structural estimate; no public running tally |
| Regional sequencing (US/APAC vs Switzerland) | ⚠ Flagged | Swiss = long tail ✅ (account complexity); other regions not re-verified |

The architect's read: the CS integration is **UBS's systems strategy for the 2020s** — a verified three-year collapse of a complete rival estate onto UBS's proprietary platforms, with hard milestones (end-2025 for Swiss client migrations and AM portfolios ✅, December 2026 for full decommissioning ✅) and a soft numbers layer (costs, savings, application counts ⚠) that the bank reports selectively. For the series, it is the largest instance of the legacy-consolidation archetype introduced by the Deutsche Bank guide — and the cross-ref to [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md) is the mechanics reference.

#### The Integration Across the Series

| Bank | Estate consolidation | Verified anchor | UBS contrast |
|---|---|---|---|
| **UBS** | Credit Suisse's *entire* platform absorbed and decommissioned (2023–2026) | ✅ 85% Swiss accounts (Q4 2025); Dec 2026 decommissioning; ⚠ costs/counts | The double-estate archetype at world scale |
| **Deutsche Bank** | Postbank retail estate folded into DB's SAP core; 2024 IT crisis | [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §2.2 | Same pattern class, ~half the client scale ⚠ |
| **BNP Paribas** | BNL/regional acquisitions consolidated over decades | [BNP Paribas Software Systems Guide](bnp_paribas_software_systems_guide.md) | Sequential acquisitions vs one shock merger |
| **SMBC** | In-house rebuild, not a merger | [SMBC Software Systems Guide](smbc_software_systems_guide.md) §2 | Build vs merge |
| **DBS / OCBC / UOB** | No rival-estate absorption; greenfield/modernisation | [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md), [UOB Software Systems Guide](uob_software_systems_guide.md) | Single-estate digitisation vs two-estate collapse |

The series-level lesson: **the CS integration is the series' extreme case of merger-driven estate consolidation** — every other consolidation in the series (Postbank, BNL) was a partial or sequential fold; UBS's is a *whole-bank platform absorption with a hard decommissioning deadline* ✅/⚠ — which is why the integration, not any new core, is the defining systems fact of UBS's era.

---

## 4. Wealth Management Systems

### 4.1 The Wealth Management Platforms (✅/⚠ Verified + Flagged)

UBS's wealth-management estate is the **crown-jewel technology layer** — the systems behind the world's largest private bank ✅. What is verifiable:

- **The client-facing platform family** ✅/⚠ — UBS's client-logins page (ubs.com) lists the group's main client platforms (UBS Digital Networks and Events, the Investment Bank Client Portal / UBS Neo, and the retail/e-banking logins ✅ — the page's existence and the Neo listing are verified this pass). The wealth client channels are the **UBS Mobile Banking app family** (the Swiss app is "long-established and popular" per UBS's own Key4 announcement ✅) and the global GWM portals ⚠ (the specific global wealth client portal names are not enumerated in the material reviewed — flagged).
- **UBS key4 wealth** ✅/⚠ — the Key4 digital line (launched 19 May 2022, §6) includes wealth-oriented offerings: UBS applied for brands including "UBS Key4 banking" and "UBS Key4 wealth" ✅/⚠ (the finews/thepoorswiss coverage of the launch lists the applied-for brands ✅); the Key4 line has since expanded into investment/provisioning features ✅ (UBS's Key4 page: "accounts, cards, practical solutions for investing and making provisions" ✅).
- **The advisory tooling** ⚠ — the adviser-side estate (portfolio construction, mandate management, the "UBS Wealth Way" advisory framework ⚠ structural — the Invest/Borrow/Protect planning framework used in UBS's advisory materials, flagged as not re-verified this pass) is real but not publicly enumerated ⚠. The **AI-powered KYC review tool** is verified via The Business Times (Oct 2025): UBS launched a Singapore pilot of an AI tool for periodic know-your-client reviews and source-of-wealth corroboration, "expected to drastically cut down the time taken to review each client" ✅ — a wealth-systems fact with a Singapore anchor (§7, §10).
- **The US digital-advice offerings** ⚠ — UBS's US digital wealth/advice platforms (the "UBS Digital Wealth" digital platform coverage reviewed this pass is low-quality/single-source ⚠; the **Wealthfront acquisition** announced in January 2022 (~USD 1.4bn ⚠) was **terminated in September 2022** ⚠ structural — the deal was called off; UBS kept its own digital-advice roadmap ⚠). Flag both honestly: the US digital-advice layer is the *least* verified slice of the wealth estate.
- **The CS wealth migration** ✅ — the verified wealth-systems event of the era: CS wealth clients and assets are migrating onto UBS platforms (the ~85% Swiss-accounts figure §3.1 is substantially a *wealth* migration, since Swiss-booked CS clients are predominantly private-banking clients ⚠ structural inference). The migration is the largest private-banking client-platform migration in history ⚠.

**The wealth-platform class** — the umbrella taxonomy: [Wealth Management Guide](wealth_management_guide.md) is the platform-class reference (advice, mandates, portfolio management, reporting); the Swiss wealth-core class (Avaloq, §2.1) is the vendor-ecosystem contrast — UBS's own platforms are proprietary ⚠, sitting alongside a Swiss private-banking ecosystem that largely runs Avaloq ✅/⚠ structural.

### 4.2 The Wealth Table

| System / platform | Function | Notes |
|---|---|---|
| **UBS GWM client platforms** | Global wealth client portals, reporting, advisory workflows | ⚠ Structural — names not enumerated this pass; the crown-jewel estate ✅ |
| **UBS Mobile Banking app** | Retail/wealth mobile channel (Swiss flagship) | ✅ "Long-established and popular" (UBS's own Key4 announcement); the Key4 host app |
| **UBS key4 (incl. key4 wealth)** | Digital product line: banking + investing + provisioning | ✅ Launched 19 May 2022; key4 banking/wealth brands ✅/⚠; expanded features ✅ |
| **Adviser tooling / AI-KYC tool** | KYC periodic reviews, source-of-wealth corroboration | ✅ Singapore pilot (The Business Times, Oct 2025); part of the integration-era productivity push |
| **UBS Wealth Way** | Advisory planning framework (Invest/Borrow/Protect) | ⚠ Structural; not re-verified this pass |
| **UBS Digital Wealth (US)** | US digital advice offering | ⚠ Flagged — single-source/low-quality coverage; treat as unverified detail |
| **Wealthfront** | US robo-advice acquisition target | ⚠ Announced Jan 2022; **deal terminated Sept 2022** ⚠ structural — do not model as part of UBS |
| **CS wealth platforms (migrating)** | Credit Suisse's private-banking platforms | ✅ Migration verified: CS client assets moved onto UBS platforms (AR2025); ~85% Swiss accounts at Q4 2025 |

The architect's read: UBS's wealth estate is **proprietary, world-scale, and in the middle of the largest client migration in private banking** ✅/⚠ — the verified anchors are the platform family's existence (UBS logins page ✅), Key4 ✅, the AI-KYC pilot ✅ and the migration milestones ✅; the unverified slices (US digital advice, adviser-tooling detail, Wealthfront history) are flagged ⚠. Cross-ref [Wealth Management Guide](wealth_management_guide.md) for the platform class and the [Universal Banking Model Guide](universal_banking_model_guide.md) for the group structure behind it.

---

## 5. Investment Bank Systems

### 5.1 The IB Technology: UBS Neo (✅ Verified)

The one verified anchor of UBS's investment-bank technology estate is **UBS Neo** — the bank's digital investment-banking client platform:

- **What Neo is** ✅ — UBS Neo is the **unified cross-asset investment-banking client platform** for UBS's Investment Bank: a digital gateway giving institutional clients "access to trading insights, research, and execution services in one place" (Attivio case-study material ✅/⚠ secondary); it was launched as "an entirely new approach to investment banking client engagement" (the platform work won the 2019 Profit & Loss Digital FX Award ✅ — powdigital coverage). UBS's own client-logins page lists it as the **"Investment Bank Client Portal. UBS Neo"** ✅ — verified this pass on ubs.com.
- **The build story** ✅ — the platform was built in partnership with **Lab49** (a capital-markets technology consultancy): "UBS launched its Neo platform to act as an intuitive gateway to all its products and services and identified Lab49 to build this truly cross-asset banking platform" ✅ (Lab49 case study); the Lab49 CEO's statement at delivery confirms the collaboration ✅ (LeapRate coverage). The launch era is the **mid-2010s** ⚠ (the platform's introduction is dated to ~2015–2016 in industry coverage ⚠ — flagged; the 2019 award is the verified date anchor ✅).
- **What Neo does** ✅/⚠ — research, trade ideas, execution access, a social/intellectual-capital layer ("differentiated intellectual capital to its clients in a seamless, social platform" — the Lab49 CEO's characterisation ✅); search/AI-style content delivery was added via a search-vendor integration (Attivio ✅/⚠ secondary). The current feature depth (post-2023, in the merged bank) is not publicly enumerated ⚠.

The IB estate beyond Neo is **opaque** ⚠ structural — UBS's markets stack (FX, rates, credit, equities execution and risk systems), the e-FX franchise (UBS has long been a top-tier FX house ⚠ structural — the Euromoney FX-survey rankings fluctuate and are flagged), and the post-trade/clearing layer are not publicly enumerated. Cross-ref the [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) for the treasury/capital-markets platform class — **UBS's use of Murex (or any named markets vendor) is NOT verified this pass** ⚠ honest negative; the treasury-class analysis is the reference for the class, not a claim about UBS's vendors.

### 5.2 The IB Landscape (✅/⚠ Verified Names + Structural Inference)

- **The client-facing layer** ✅ — UBS Neo (the verified portal, §5.1); UBS Digital Networks and Events (listed on the UBS client-logins page ✅ — the events/network platform for IB clients, function ⚠).
- **The markets layer** ⚠ — execution, pricing and risk systems are in-house (UBS is historically a build-house for markets technology ⚠ structural); the e-FX franchise is a known strength ⚠ (the 2019 Profit & Loss award to the Neo work ✅ is the closest verified anchor); the merged IB inherits CS's markets estate, which is being consolidated under the same UBS-platforms-first rule as the core (§3.2) ⚠.
- **The advisory/financing layer** ⚠ — the deal pipeline (M&A, ECM/DCM) runs on standard IB tooling ⚠; nothing vendor-specific is verifiable this pass — flagged.
- **The strategic shape** ✅/⚠ — the post-2012 IB is "more targeted" (UBS's own positioning, reflected in the structure summaries ✅/⚠); the CS merger *added* a bulge-bracket-scale markets book back into the franchise ⚠ (CS was a bulge-bracket member until 2023 ✅ — Wikipedia) — so the IB estate is also growing before it consolidates ⚠ structural.

### 5.3 The IB Table

| System | Function | Notes |
|---|---|---|
| **UBS Neo** | Cross-asset IB client portal: research, trade ideas, execution access | ✅ Verified (UBS logins page; Lab49 case study; 2019 Profit & Loss award); mid-2010s launch era ⚠ |
| **UBS Digital Networks and Events** | IB client events/network platform | ✅ Listed on UBS logins page; function ⚠ |
| **Markets execution/risk estate** | FX, rates, credit, equities trading systems | ⚠ Structural — in-house; not enumerated; Murex or any named vendor NOT verified ⚠ |
| **e-FX franchise** | Electronic FX trading for institutional clients | ⚠ Structural — top-tier FX house; award evidence ✅ (2019 P&L award to Neo work) |
| **CS markets estate (merging)** | Credit Suisse's trading platforms | ⚠ Structural — consolidation onto UBS platforms under §3.2 rules; not enumerated |

The architect's read: UBS's IB technology is a **thin verified layer (Neo) over an opaque in-house markets estate** ⚠ — Neo is the digital storefront (✅ verified), the engines behind it are proprietary and undisclosed (⚠), and the CS merger is adding a second markets estate to the consolidation (⚠). Cross-ref [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) for the platform class and [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md) for the risk layer.

---

## 6. Digital Banking Systems

### 6.1 The Digital Platforms: UBS key4 and the App Family (✅ Verified)

UBS's digital-banking story for its home market is unusually well-documented, because UBS *marketed* it:

- **UBS key4 — the purely digital product line** ✅ — launched **19 May 2022** (UBS media release: "UBS is launching a purely digital product line, UBS key4, for clients who want to carry out their banking transactions entirely digitally – independently and around the clock from their smartphone" ✅). Key4 is **integrated into the long-established UBS Mobile Banking App** ✅ (the release: "The new product line is integrated into the long-established and popular UBS Mobile Banking App and provides access to UBS's leading digital offering" ✅). The launch followed app-store approval from Apple and Google ✅/⚠ (the finews coverage of the launch ✅).
- **The Key4 product family** ✅/⚠ — the brands UBS applied for included "UBS Key4 banking" and "UBS Key4 wealth" ✅/⚠ (launch coverage); the line covers accounts, cards, "practical solutions for investing and making provisions" ✅ (the UBS Key4 page). **key4 banking went zero-fee in March 2025** ✅/⚠ (neobanque.ch coverage: "UBS introduces key4 banking with CHF 0 fees, offering a free alternative for Credit Suisse customers following the acquisition" ✅/⚠ — single-source-ish, flagged) — a digital-acquisition move aimed at the CS client base during the integration ✅/⚠.
- **The mobile app family** ✅ — the UBS Mobile Banking App is the host channel (UBS's own characterisation ✅); the Swiss e-banking portal and the UBS logins page complete the digital estate ✅ (UBS logins page verified this pass).
- **The neobank challenge context** ✅ — UBS launched Key4 explicitly to "challenge the rise of neobanks in its home market" (finews ✅) — the Swiss digital-banking answer to the same pressures the Asian series banks faced from digital challengers ([DBS Software Systems Guide](dbs_software_systems_guide.md) §4).
- **The CS digital tail** ⚠ — Credit Suisse's own digital channels (the CS app, CSX — the digital banking app Credit Suisse had launched ⚠ structural) are being retired as CS clients migrate to UBS channels (§3) ⚠ — the digital-estate version of the decommissioning story.

### 6.2 The Digital Table

| Platform | Function | Notes |
|---|---|---|
| **UBS key4** | Purely digital product line (banking + investing + provisions) | ✅ Launched 19 May 2022 (UBS release); zero-fee key4 banking March 2025 ✅/⚠; key4 wealth brand ⚠ |
| **UBS Mobile Banking App** | The flagship Swiss mobile channel | ✅ "Long-established and popular" (UBS's own words); the Key4 host |
| **UBS e-banking / logins portal** | Online banking and client-platform access | ✅ UBS logins page verified this pass |
| **UBS Digital Networks and Events** | Digital events/network platform for clients | ✅ Listed on UBS logins page (§5.2) |
| **CS digital channels (CSX etc.)** | Credit Suisse's retail digital banking | ⚠ Structural — being retired as clients migrate to UBS channels (§3) |

The architect's read: UBS's digital banking is **Switzerland-centric and proprietary** — Key4 (2022 ✅) and the UBS Mobile Banking app are the verified flagship channels, positioned explicitly against neobanks ✅, and the CS digital estate is being folded into them as part of the integration ⚠. Compare the Asian digital journeys in [OCBC Software Systems Guide](ocbc_software_systems_guide.md) and [UOB Software Systems Guide](uob_software_systems_guide.md) — the difference is that UBS's digital line is one slice of a wealth-first group, not the whole bank.

---

## 7. AI and Innovation

### 7.1 The GenAI Initiatives (✅ Anchors, ⚠ Specifics Flagged)

UBS's AI story is *verified in the ambition, flagged in the specifics*:

**Verified anchors (✅):**
- **The ambition** ✅ — UBS's own "Innovation and AI" page (ubs.com) states the goal: "We aim to become an AI enabled financial institution where all our stakeholders such as clients, employees, and shareholders benefit from AI-powered tools" ✅ — verified this pass.
- **The usage numbers** ✅/⚠ — Swiss tech press (Netzwoche, Aug 2025): UBS logged **8 million prompts across its GenAI tools in Q2 2025 alone — four times the level at end-2024 — and counted over 280 active AI use cases** ✅/⚠ (Netzwoche ✅; the "280+" figure is a press-reported point-in-time ⚠).
- **Microsoft 365 Copilot at scale** ⚠ — UBS "launched the largest deployment of Microsoft 365 Copilot in financial services by rolling out **50,000 licenses**" (2025-era coverage ✅/⚠ — a single LinkedIn source this pass; flagged ⚠ but consistent with the verified Microsoft partnership of §8 and the Copilot-expansion pattern).
- **The adviser-AI pilot** ✅ — The Business Times (6 Oct 2025): **UBS launched a Singapore pilot** of a new AI tool aimed at boosting adviser productivity — focused on **periodic know-your-client reviews and source-of-wealth corroboration** — "expected to drastically cut down the time taken to review each client" ✅ — the verified, named, dated AI initiative of this guide, and it is a *wealth-systems* AI (§4, §10).
- **"Red AI"** ⚠ — the same 2025 coverage mentions UBS "introduc[ing] Red AI" (an internal AI tool/assistant ⚠ — the specifics are single-source and flagged ⚠).

**Flagged specifics (⚠):**
- The internal assistant naming ("Red AI" ⚠), the exact Copilot license count at the time of writing ⚠, the per-use-case deployment detail ⚠, and any named LLM vendors beyond the Microsoft partnership ⚠ (the Azure AI layer of §8 is the verified platform; model specifics are not).
- Client-facing genAI (research summarisation, portfolio commentary) is structurally likely ⚠ but not verified this pass.
- The AI-governance layer (FINMA's AI expectations for Swiss banks, the model-risk framework) is structural ⚠ — cross-ref [AI Governance, Bias & Red-Teaming Guide](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) and [Enterprise AI Platforms Guide](../technology/ai_llm/enterprise_ai_platforms_guide.md).

### 7.2 The AI Table: Initiative, Description, Notes

| Initiative | Description | Notes |
|---|---|---|
| "AI enabled financial institution" ambition | Group-wide AI strategy (clients, employees, shareholders) | ✅ ubs.com "Innovation and AI" page |
| Microsoft 365 Copilot deployment | GenAI assistant at ~50,000 licences (largest in financial services per coverage) | ⚠ Single-source-ish 2025 coverage; consistent with the verified Microsoft partnership (§8) |
| GenAI usage scale | 8M prompts in Q2 2025 (4x end-2024); 280+ active use cases | ✅/⚠ Netzwoche (Aug 2025); point-in-time ⚠ |
| Adviser-AI KYC tool | Periodic KYC review + source-of-wealth corroboration automation | ✅ The Business Times (Oct 2025), Singapore pilot; wealth-systems AI (§4) |
| Red AI | Internal AI tool/assistant | ⚠ Single-source coverage; specifics unverified |
| Azure AI layer | Cloud-native AI platform access | ✅ Microsoft partnership (§8) — the verified platform; model specifics ⚠ |
| AI governance / model risk | FINMA expectations, model-risk framework | ⚠ Structural; cross-ref [AI Governance, Bias & Red-Teaming Guide](../technology/ai_llm/ai_governance_bias_redteaming_guide.md) |

The architect's read: UBS's AI programme is **real, measured, and marketed** — the verified anchors are the ambition statement ✅, the usage numbers ✅/⚠, the Singapore adviser-AI pilot ✅ and the Microsoft platform ✅ — while the *specifics* (tool names, license counts, model choices) are flagged ⚠. The pattern matches the series' other AI leaders ([DBS Software Systems Guide](dbs_software_systems_guide.md) §5, [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §5): enterprise-genAI on a hyperscaler platform, with the difference that UBS's flagship use case is *wealth-adviser productivity during the largest client migration in private banking* ✅.

---

## 8. Cloud

### 8.1 The Cloud Strategy: Microsoft Azure (✅ Verified)

UBS's cloud strategy is the **best-documented layer of this guide** — UBS publishes it:

- **The partnership** ✅ — UBS and Microsoft announced a "landmark expansion of cloud partnership" on **20 October 2022** ✅ (UBS media release + Microsoft Source + publicnow coverage, all dated 20 Oct 2022). Through this initiative UBS plans "to have **more than 50% of its applications, including critical workloads, running on Microsoft Azure, now UBS's primary cloud platform**" ✅ (publicnow/UBS release wording). The partnership "furthers UBS's 'cloud first' strategy and the modernization of its global technology estate" ✅.
- **The 2018 strategy and its early success** ✅ — "In 2018, when UBS announced its cloud strategy, leveraging its strategic partnership with Microsoft, it planned to move **one-third of its applications to public cloud within four years, but accomplished this goal early in February 2021**" ✅ (UBS and Microsoft statements, repeated in both the UBS release and Microsoft Source) — a verified, dated, quantified cloud milestone.
- **The Swiss anchor** ✅ — "In 2019, we were a first mover by being an **anchor client for the Microsoft Cloud in Switzerland**. By helping to physically bring Azure servers to Switzerland we unlocked many more use cases for cloud services in our business" ✅ (UBS's "Cloud as an opportunity" technology page, verified this pass) — the Swiss data-sovereignty angle that matters for a Swiss bank under FINMA supervision ⚠.
- **The Azure AI layer** ✅ — Microsoft's UBS customer story confirms UBS "deployed Azure solutions across five divisions and accelerated digital transformation", with "Azure AI and cloud services" enhancing "operational efficiency and data insights, leading AI-driven innovation" ✅ — the platform link between §7 (AI) and §8 (cloud).
- **Multi-cloud?** ⚠ — Azure is the verified *primary* cloud; whether UBS runs meaningful AWS/Google workloads alongside is not verified this pass ⚠ (Azure-primary is the honest position; the CS estate may carry other cloud exposure ⚠).

### 8.2 The Cloud Table

| Element | Status | Notes |
|---|---|---|
| Microsoft Azure as primary cloud | ✅ Verified | UBS release, 20 Oct 2022: >50% of apps incl. critical workloads planned on Azure |
| 2018 cloud strategy (1/3 of apps in 4 years) | ✅ Verified | Achieved early Feb 2021 (UBS + Microsoft statements) |
| Anchor client for Microsoft Cloud in Switzerland (2019) | ✅ Verified | UBS technology page — first-mover; Azure regions in Switzerland |
| "Cloud first" strategy | ✅ Verified | UBS release wording |
| Azure AI for innovation | ✅ Verified | Microsoft customer story — Azure AI across five divisions |
| Multi-cloud (AWS/Google) | ⚠ Flagged | Not verified this pass; Azure-primary is the honest position |
| CS estate cloud migration | ⚠ Flagged | Structural — part of the §3 consolidation; not separately disclosed |

The architect's read: UBS's cloud strategy is **Azure-primary, Swiss-anchored and quantified** ✅ — a rare case in this series of a bank publishing its cloud milestones (1/3 of apps by Feb 2021 ✅; >50% incl. critical workloads ✅). The contrast with Deutsche Bank's Google Cloud bet ([Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §6) and DBS's multi-cloud posture ([DBS Software Systems Guide](dbs_software_systems_guide.md) §6) makes UBS the *single-hyperscaler-primary* archetype — with the same single-anchor risk the Deutsche guide flagged for Google ⚠.

---

## 9. Security and Fraud Systems

### 9.1 Fraud Prevention (✅/⚠ Verified + Flagged)

UBS's security and fraud estate is **verified at the client-facing layer, flagged at the systems layer**:

- **The client-facing fraud programme** ✅ — UBS publishes a fraud-prevention surface for clients: the UBS (US) "Protect yourself from fraud" page — "Learn how UBS protects your personal data and accounts and how you can protect yourself from scams, cyber fraud, identity theft and check fraud" ✅ (verified this pass). The client education/alerting layer (scam awareness, identity-theft guidance) is the visible part of the fraud estate.
- **The detection layer** ⚠ — the behind-the-scenes monitoring (transaction monitoring, behavioural analytics, AI-assisted fraud detection) is structurally certain for a bank of this scale ⚠ but not publicly enumerated ⚠ — cross-ref [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) for the class mechanics; **no named fraud/AML vendor (FICO, SAS, Nice Actimize, etc.) is verified for UBS this pass** ⚠ honest negative (the Deutsche Bank guide made the same honest negative — [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §7).
- **The integration-era security risk** ⚠ — the CS integration is itself a security event class: data migration (client data moving between estates), identity/access consolidation (two IAM estates into one), and insider/dual-run risk during waves ⚠ structural — the security dimension of §3. The migration *controls* (encryption, access governance, reconciliation) are the security systems of the 2020s ⚠.
- **The regulatory layer** ⚠ — FINMA (the Swiss financial regulator) supervises UBS's operational resilience, outsourcing (cloud) and AML/CFT obligations ⚠ structural; the Swiss banking-secrecy and data-protection regime shapes the Swiss cloud and data-migration design (§8's Swiss anchor ✅ is partly a security fact) ⚠.
- **Authentication for Swiss retail** ⚠ — the UBS Access app (UBS's authentication app for Swiss digital banking) is structural knowledge ⚠ flagged as not re-verified this pass; the class mechanics (push-based 2FA) match the pushTAN pattern noted in the Deutsche guide ⚠.

### 9.2 The Security Table

| Layer | What it is | Notes |
|---|---|---|
| Client fraud protection | Scam/cyber-fraud/identity-theft/check-fraud education and alerting | ✅ UBS fraud-prevention page (verified this pass) |
| Transaction monitoring / AML | Detection estate for money laundering and fraud | ⚠ Structural; no named vendor verified ⚠; cross-ref [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md) |
| Integration-era controls | Data-migration security, IAM consolidation, dual-run controls | ⚠ Structural — the security dimension of §3 |
| Cloud security | Azure security posture, Swiss data residency | ✅/⚠ Swiss anchor verified (§8); detail ⚠ |
| Authentication (UBS Access app) | 2FA for Swiss digital banking | ⚠ Structural; not re-verified this pass |
| Regulatory oversight | FINMA supervision of resilience/outsourcing/AML | ⚠ Structural |

The architect's read: UBS's security estate is **client-visible but systems-opaque** ✅/⚠ — the fraud-prevention surface is verified ✅, the detection stack is flagged ⚠, and the *defining* security story of the era is the integration itself (two estates' data and identities collapsing into one ⚠). For the series, UBS joins Deutsche Bank in the "honest negative" club: no named fraud vendor is asserted.

---

## 10. The Singapore Angle

### 10.1 UBS Singapore: The APAC Wealth Hub (✅ Verified)

UBS Singapore is the group's **Asia-Pacific wealth hub** ✅ — and unlike the Deutsche Bank SG section (a wholesale-only hub, [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §8), the UBS Singapore operation is *wealth-first*:

- **The APAC headquarters footprint** ✅ — UBS moved into its new Singapore office at **9 Penang Road**, "consolidating our wealth management, investment banking and asset management teams into an approximately **400,000 sq ft facility — our largest in the Asia Pacific**" (UBS's own LinkedIn announcement ✅, verified this pass). The consolidation of the three client franchises into one building is itself a *systems* fact: the Singapore estate runs the group's wealth platforms (§4), the IB client platform (Neo, §5) and the AM estate from one site ⚠ structural.
- **The wealth-hub position** ✅/⚠ — Singapore is one of the world's largest wealth-management centres and UBS is a top-tier private bank there ✅/⚠ (structural — UBS has long been among the largest private banks in Singapore and the region ⚠; the exact AUM ranking is flagged). The regional wealth context is striking: press coverage this pass reports Singapore's total assets under management crossing **USD 5.6 trillion in early 2026, surpassing Switzerland for the first time** ⚠ (single-source snippet — flagged; the exact figure/date should be re-verified before being cited) — the market-level reason Singapore is UBS's APAC hub.
- **The adviser-AI pilot market** ✅ — UBS chose **Singapore for the October 2025 pilot** of its adviser-productivity AI tool (periodic KYC reviews, source-of-wealth corroboration — The Business Times ✅) — the verified link between the SG angle (§10) and the AI layer (§7): the region's scale and the integration-era KYC pressure (CS clients migrating, §3) make Singapore the natural pilot market ✅/⚠ structural inference.
- **The regional-HQ relocation reports** ⚠ — press has reported that UBS shifted its Asia-Pacific regional headquarters from Hong Kong to Singapore in the post-merger era ⚠ (widely reported; flagged as not re-verified this pass); the 9 Penang Road consolidation ✅ is the verified footprint fact, and the HK-vs-SG HQ arrangement should be treated as ⚠.
- **The Singapore technology function** ⚠ — UBS runs technology and operations capability in Singapore (the hub supports the regional wealth platforms and the booking centres ⚠ structural); the exact technology headcount and the system inventory are not public ⚠.
- **The integration's Singapore slice** ⚠ — Credit Suisse had a major Singapore private bank; the CS Singapore client book is part of the migration waves (§3) ⚠ structural — Singapore is simultaneously a *growth* market and a *migration* market for the UBS estate.

### 10.2 The SG Table

| Element | Status | Notes |
|---|---|---|
| 9 Penang Road office (~400,000 sq ft) | ✅ Verified | UBS announcement — largest APAC facility; WM/IB/AM consolidated |
| APAC wealth hub position | ✅/⚠ | Structural — top-tier regional private bank; AUM rank ⚠ |
| Adviser-AI KYC pilot (Oct 2025) | ✅ Verified | The Business Times; the §7 flagship AI initiative |
| Regional HQ from Hong Kong | ⚠ Flagged | Reported; not re-verified this pass |
| SG technology/ops function | ⚠ Flagged | Structural — booking centres and platform support; headcount not public |
| CS Singapore client migration | ⚠ Flagged | Part of the §3 waves; not separately disclosed |
| Singapore AUM > Switzerland (2026) | ⚠ Flagged | Single-source coverage; re-verify before citing |

#### The SG Angle Across the Series

| Bank | Singapore role | UBS contrast |
|---|---|---|
| **UBS** | APAC wealth hub: WM + IB + AM at 9 Penang Road; AI pilot market | Wealth-first hub; migration + growth simultaneously |
| **Deutsche Bank** | Wholesale/markets hub, no retail | [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §8 — wholesale-only vs wealth-first |
| **SMBC** | APAC/ASEAN base | [SMBC Software Systems Guide](smbc_software_systems_guide.md) §8 — corporate/regional vs wealth |
| **DBS / OCBC / UOB** | Home markets | The Singapore banks' home vs UBS's hub |

The architect's read: UBS Singapore is the **wealth-hub archetype of the series** — a verified physical footprint (9 Penang Road ✅), a verified AI-pilot role (✅), a structural wealth-market position (⚠), and an integration slice (⚠) — the Singapore angle is where UBS's three big stories (wealth, AI, integration) converge.

---

## 11. Worked Example: A UBS Wealth Client Journey

### 11.1 The Scenario: A UBS Wealth Client

The scenario is anchored to the verified facts of this guide: **a Credit Suisse private-banking client in Singapore whose book is being migrated onto UBS platforms during the integration, served by a UBS adviser using the new AI-assisted KYC tooling, with the group's Azure-based AI estate behind the scenes.** (Every system named in the flow is verified in §2–§10; every step is labelled with its verification status.)

The client: *Mrs. Tan*, a Singapore-based HNW client (a retired regional business owner) whose private-banking relationship was with **Credit Suisse Singapore** before the March 2023 rescue (§1.4). Her book — equities, bonds, structured products and a mortgage against her Singapore property — is part of the **~85% of Swiss-booked CS client accounts** migrated by Q4 2025 ✅ (§3.1), and more generally of the "CS client assets outside Switzerland migrated onto UBS platforms" programme ✅ (UBS Annual Report 2025). She is exactly the client the integration exists for.

### 11.2 The Flow (Verify-Anchored)

| Step | What happens | Systems behind it | Status |
|---|---|---|---|
| **1. The migration wave** | Mrs. Tan's CS Singapore account is cut over in a migration wave: her positions, mandates and KYC record move from the CS estate onto UBS's wealth platforms | Data migration + reconciliation between the CS estate and the UBS GWM platforms (§3.2); the [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md) mechanics at estate scale | ✅/⚠ (migration verified; wave mechanics structural) |
| **2. The new onboarding/KYC refresh** | The integration triggers a KYC review. Her UBS adviser runs the **adviser-AI tool piloted in Singapore** — periodic know-your-client review and source-of-wealth corroboration — "drastically cut[ting] down the time taken to review each client" (The Business Times ✅) | The adviser-AI KYC tool (§7, §10); the underlying Azure AI layer (§8) | ✅ (pilot verified; production rollout ⚠) |
| **3. The digital channel** | Mrs. Tan logs into the UBS wealth channel on her phone — the **UBS Mobile Banking app** family (§6) — and sees her migrated portfolio: holdings, valuations, the mortgage | The UBS app family + GWM platforms (§4, §6); Swiss/APAC channel estate | ✅ (channels verified; her specific view ⚠) |
| **4. The adviser conversation** | She meets her adviser (in person at 9 Penang Road or by video) for a portfolio review; the adviser uses the advisory tooling and the "UBS Wealth Way"-class planning framework to reposition her book for income | Adviser tooling ⚠ (§4); the GWM platforms | ⚠ (framework structural) |
| **5. The trade** | A rebalancing trade is executed: equities to bonds, a structured-product roll | The UBS execution estate for wealth orders (the IB/markets layer, §5 ⚠); Neo-class platforms for any institutional-grade product access | ⚠ (execution estate structural) |
| **6. The fraud check** | A large outgoing payment triggers the bank's controls: transaction monitoring, behavioural analytics, and client-side alerts in the "protect yourself from fraud" pattern (scams, cyber fraud, identity theft — the UBS fraud-prevention surface ✅) | The detection estate (§9 ⚠); the client alerting layer (§9 ✅) | ✅/⚠ |
| **7. The Swiss tail** | Any Swiss-booked legacy product is serviced through the remaining CS systems until their December 2026 decommissioning ✅ | CS legacy systems in dual-run/decommissioning (§3) | ✅ (target verified) |

#### Alternative Journeys (The Same Estate, Other Users)

- **The Swiss retail user**: a Zurich retail client on **Key4** (zero-fee key4 banking, §6) — purely digital onboarding, mobile-only, no adviser — the neobank-challenger slice of the estate ✅.
- **The institutional client**: a corporate treasury using the **IB client portal (UBS Neo)** for research, trade ideas and execution access ✅ (§5) — the wholesale slice.
- **The UHNW mandate client**: a discretionary-mandate client whose portfolio runs on the GWM mandate/portfolio-construction estate ⚠ (§4) — the crown-jewel slice.

### 11.3 The Lessons

The worked example distils the guide's findings into architect-facing lessons:

1. **The migration is the product** ✅/⚠ — for a huge slice of UBS's clients (former CS clients), their "digital journey" *is* the integration: data migration, KYC refresh, channel cut-over, and a December 2026 decommissioning tail ✅. An architect building against UBS's estate must model **two estates converging**, not one estate — the same lesson as Deutsche Bank's Postbank ([Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md) §9) at roughly double scale ⚠.
2. **Wealth AI is the differentiator** ✅ — the verified flagship AI initiative is adviser-facing KYC/source-of-wealth automation (Singapore pilot, Oct 2025 ✅) — the integration-era use case. UBS's AI is not (yet, publicly) a client-facing chatbot story; it is an *adviser-productivity and migration-risk* story ✅/⚠.
3. **The verified core is thin** ⚠ — an architect must hold the honest flags: UBS's core is proprietary (⚠ structural, §2.1), the Avaloq premise is unverified (§2.1), the IB estate beyond Neo is opaque (§5), and the fraud vendor map is unverified (§9). **Design against verified anchors (migrations ✅, Key4 ✅, Neo ✅, Azure ✅, fraud surface ✅) and flag the rest.**
4. **Cloud and AI are the same platform** ✅ — the Azure strategy (§8) and the AI programme (§7) are one investment: the 2022 expansion (>50% of apps incl. critical workloads on Azure ✅) is the substrate for Copilot, the KYC AI and the 280+ use cases ✅/⚠. The single-hyperscaler anchor is a strategic dependency ⚠.

#### The Architect's Checklist (What to Verify Before Designing Against This Estate)

- ✅ Confirm the migration wave your client population is in (UBS publishes progress: 85% Swiss accounts at Q4 2025; Dec 2026 decommissioning).
- ⚠ Re-verify the Avaloq question before citing it — no source connects UBS's core to Avaloq (§2.1).
- ✅ Use the verified platform anchors: UBS Neo (IB portal), Key4 + UBS Mobile Banking app (digital), Azure (cloud), the fraud-prevention surface (security).
- ⚠ Treat all integration cost/savings/application-count figures as reported-but-unverified (§3.1).
- ✅ Watch the Singapore pilot: the adviser-AI tool is the leading indicator of UBS's production AI estate (§7, §10).

---

## 12. Summary: The Swiss Wealth Giant's Modern Stack

One page, for the architect who needs the whole map in a single read:

**UBS Group AG — formed 1998 by the merger of the Union Bank of Switzerland and the Swiss Bank Corporation ✅, headquartered Zurich and Basel ✅, the world's largest private bank ✅ — runs a wealth-first universal-bank estate in four verified layers:**

1. **The core is a proprietary double-estate consolidation** ✅/⚠ — internally developed UBS platforms (the surviving estate, ⚠ structural) absorbing **Credit Suisse's entire platform** (the 2023 rescue acquisition ✅): ~85% of Swiss-booked CS client accounts migrated by Q4 2025 ✅, all AM portfolios by end-2025 ✅, full CS system decommissioning by **December 2026** ✅. **The Avaloq-as-UBS-core premise is NOT verified** ⚠ — and Avaloq's own origin (BZ Bank, 1985 ✅) refutes the "came from UBS" variant (§2.1). The integration cost/savings arithmetic (order of USD 10bn / USD 13bn ⚠) is the price of the merge.
2. **The wealth estate is the crown jewel** ✅/⚠ — the GWM platforms, the UBS Mobile Banking app family, Key4 (May 2022 ✅) and the adviser-AI KYC tool (Singapore pilot ✅) — with the US digital-advice layer flagged ⚠ and the Wealthfront deal terminated ⚠ (§4).
3. **The modernisation is Azure + AI, publicly quantified** ✅ — the Microsoft partnership (2018 strategy: 1/3 of apps to public cloud by Feb 2021 ✅; Oct 2022 expansion: >50% of apps incl. critical workloads on Azure ✅) is the substrate for the genAI estate (8M prompts in Q2 2025, 280+ use cases ✅/⚠, Copilot at ~50k licences ⚠, the "AI enabled financial institution" ambition ✅, §7–§8). The IB layer is a thin verified storefront (UBS Neo ✅) over an opaque in-house markets estate (§5).
4. **The digital and security layers are Swiss-first** ✅/⚠ — Key4 as the neobank answer (§6 ✅), the fraud-prevention surface (✅) over an unverified detection vendor map (§9 ⚠), and FINMA's resilience/outsourcing oversight (§9 ⚠).

**The Singapore angle** — the APAC wealth hub ✅: 9 Penang Road, ~400,000 sq ft, WM/IB/AM consolidated ✅, the adviser-AI pilot market ✅, and the CS Singapore migration slice ⚠ (§10).

**The estate at a glance** (the one-table read of the stack):

| Layer | System(s) | Status |
|---|---|---|
| Core banking | Proprietary UBS cores + CS estate in migration | ✅ milestones / ⚠ vendor map; Dec 2026 decommissioning ✅ |
| CS integration | Wave migrations, dual-run, decommissioning | ✅ anchors (85%, end-2025, Dec 2026) / ⚠ costs, counts |
| Wealth | GWM platforms, app family, Key4, AI-KYC | ✅ channels / ⚠ adviser-tooling detail, US digital |
| Markets | UBS Neo + in-house markets estate | ✅ Neo / ⚠ estate, vendors |
| Digital retail | Key4, UBS Mobile Banking app, e-banking | ✅ verified |
| AI | Copilot, KYC tool, 280+ use cases | ✅ ambition/usage / ⚠ specifics |
| Cloud | Microsoft Azure (primary) | ✅ verified, quantified |
| Security | Fraud surface; detection estate | ✅ surface / ⚠ vendors |
| Singapore | APAC hub at 9 Penang Road; AI pilot | ✅ hub / ⚠ depth |

**The Swiss wealth giant's modern stack** is therefore not a showcase of new cores or greenfield platforms. It is a **25-year-old merged bank (with 160+ years of Swiss banking heritage behind it ⚠) running the largest bank-platform consolidation in history** ✅/⚠: proprietary wealth platforms that survive and absorb, a rival estate being decommissioned by December 2026 ✅, an Azure-primary cloud that is genuinely quantified ✅, and an AI programme aimed squarely at the migration era ✅/⚠. For the series, UBS is the *merger-consolidation archetype* — the answer to "how does the world's largest wealth manager absorb a rival bank's entire estate?" sits between Deutsche Bank's legacy consolidation ([Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md)) and the Asian banks' greenfield digitisation ([DBS Software Systems Guide](dbs_software_systems_guide.md)): **keep the proprietary crown jewels, collapse the rival estate, and run the modernisation (Azure + AI) concurrently with the merge** — with honest flags on everything the bank does not disclose.

---

## 13. Claims Status and Verification Notes

| Claim | Status | Source/Note |
|---|---|---|
| Founded 1998 via UBS + SBC merger | ✅ Verified | Britannica; Wikipedia; UBS Virtual Museum |
| HQ Zurich and Basel; world's largest private bank | ✅ Verified | Wikipedia/Britannica characterisation |
| Divisions: GWM, P&C, AM, IB (+ Non-core and Legacy) | ✅ Verified | UBS Group Annual Report 2023 structure |
| CS rescue takeover announced 19 March 2023 | ✅ Verified | UBS media release; Business Insider; Euromoney |
| "USD 5 trillion combined invested assets" (deal claim) | ✅ Verified | UBS announcement wording (deal-claim, current value ⚠) |
| All-share ~CHF 3bn; completed June 2023 | ✅/⚠ | Standard/City AM; exact terms ⚠ |
| ~85% of Swiss-booked CS accounts migrated (Q4 2025) | ✅ Verified | execpartners summary of Q4 2025 report |
| Most Swiss client migrations + all AM portfolios by end-2025 | ✅ Verified | UBS guidance (Rio Times) |
| CS assets outside Switzerland migrated onto UBS platforms | ✅ Verified | UBS Annual Report 2025 |
| Full CS decommissioning by December 2026 | ✅ Verified | Integration updates; multiple sources |
| 1,000+ applications decommissioned | ⚠ Flagged | Reported in integration commentary; count not re-verified |
| Integration costs ~USD 10bn / savings ~USD 13bn by 2027 | ⚠ Flagged | Press-reported targets |
| Avaloq = UBS's core | ⚠ NOT VERIFIED | No source found; Avaloq origin = BZ Bank (1985) ✅ (§2.1) |
| Avaloq = dominant Swiss wealth-core class | ✅/⚠ | Structural — Avaloq marketing/Wikipedia ✅; UBS usage ⚠ |
| UBS core = proprietary, internally developed | ⚠ Flagged | Structural; not publicly enumerated |
| UBS Neo = IB client portal | ✅ Verified | UBS logins page; Lab49 case study; 2019 P&L award |
| UBS Neo launch era mid-2010s | ⚠ Flagged | ~2015–2016 per industry coverage; 2019 award ✅ anchor |
| Key4 launched 19 May 2022 | ✅ Verified | UBS media release |
| key4 banking zero-fee March 2025 | ✅/⚠ | neobanque.ch; single-source-ish |
| "AI enabled financial institution" ambition | ✅ Verified | ubs.com Innovation and AI page |
| 8M prompts Q2 2025; 280+ AI use cases | ✅/⚠ | Netzwoche (Aug 2025) |
| Microsoft 365 Copilot ~50,000 licences | ⚠ Flagged | Single LinkedIn source; consistent with §8 partnership |
| Adviser-AI KYC pilot, Singapore, Oct 2025 | ✅ Verified | The Business Times |
| Red AI | ⚠ Flagged | Single-source coverage |
| Azure primary cloud; >50% of apps target | ✅ Verified | UBS release 20 Oct 2022 |
| 2018 cloud strategy achieved early Feb 2021 (1/3 of apps) | ✅ Verified | UBS + Microsoft statements |
| Anchor client Microsoft Cloud in Switzerland (2019) | ✅ Verified | UBS technology page |
| Client fraud-prevention surface | ✅ Verified | ubs.com fraud-prevention page |
| Named fraud/AML vendor for UBS | ⚠ NOT VERIFIED | Honest negative (§9.1) |
| 9 Penang Road ~400,000 sq ft APAC office | ✅ Verified | UBS LinkedIn announcement |
| Regional HQ moved from Hong Kong | ⚠ Flagged | Reported; not re-verified |
| Singapore AUM USD 5.6tn surpassing Switzerland (2026) | ⚠ Flagged | Single-source; re-verify before citing |
| ~120,000 employees post-merger | ⚠ Flagged | Structural estimate |

---

## 14. Glossary

| Term | Definition |
|---|---|
| **UBS** | UBS Group AG — Switzerland's largest bank and the world's largest private bank ✅, formed in 1998 by the merger of the Union Bank of Switzerland and the Swiss Bank Corporation ✅; HQ Zurich and Basel ✅; "UBS" derives from "Union Bank of Switzerland" but is not an acronym ✅ |
| **SBC** | Swiss Bank Corporation — founded 1854 in Basel ⚠; merged with the Union Bank of Switzerland in 1998 to form UBS ✅; the "three keys" icon survives in UBS's branding ✅ |
| **Credit Suisse** | The Swiss global bank (founded 1856 ⚠) acquired by UBS in the March 2023 rescue takeover ✅; a bulge-bracket member until 2023 ✅ |
| **CS** | Common abbreviation for Credit Suisse; the acquired estate being decommissioned by December 2026 ✅ |
| **Global Wealth Management (GWM)** | UBS's flagship division — the world's largest wealth-management franchise ✅ (§1.3); the crown-jewel systems estate (§4) |
| **Investment Bank (IB)** | UBS division: markets, advisory, financing ✅ (§1.3); runs UBS Neo (§5) |
| **Personal & Corporate Banking (P&C)** | UBS's Swiss domestic bank division ✅ (§1.3); runs the Swiss retail core and Key4 (§2, §6) |
| **Asset Management (AM)** | UBS division: funds and institutional mandates ✅ (§1.3); AM portfolios fully migrated from CS by end-2025 ✅ |
| **Non-core and Legacy (NCL)** | The fifth unit: run-off/legacy books ✅ (Annual Report 2023); systems run down, not migrated ⚠ |
| **Core banking** | The account/deposit/loan back-end; UBS's is proprietary and internally developed ⚠, absorbing the CS estate ✅ (§2) |
| **IT integration** | The Credit Suisse integration programme: wave migrations, dual-run, decommissioning; ~85% Swiss accounts migrated (Q4 2025), full decommissioning by Dec 2026 ✅ (§3) |
| **Wealth management** | UBS's centre of gravity — the world's largest private-banking franchise ✅; see [Wealth Management Guide](wealth_management_guide.md) for the platform class |
| **Digital banking** | UBS's digital channels: Key4, the UBS Mobile Banking app, e-banking ✅ (§6) |
| **AI** | Artificial intelligence — UBS's "AI enabled financial institution" ambition ✅ (§7) |
| **genAI** | Generative AI — the 2023–2025 era estate: Copilot, the KYC tool, 280+ use cases ✅/⚠ (§7) |
| **Cloud** | The infrastructure layer; Microsoft Azure is UBS's primary cloud ✅ (§8) |
| **Fraud** | Client-facing fraud protection (scams, cyber fraud, identity theft, check fraud) ✅ (§9) |
| **Security** | The security estate: fraud surface ✅, detection stack ⚠, integration-era controls ⚠ (§9) |
| **Singapore** | UBS's Asia-Pacific wealth hub ✅ — 9 Penang Road (~400,000 sq ft ✅); the adviser-AI pilot market ✅ (§10) |
| **APAC** | Asia-Pacific — the region UBS serves from Singapore ✅ (§10) |
| **Switzerland** | UBS's home market ✅ — the P&C domestic bank, the Swiss core, Key4, and FINMA's supervisory scope (§2, §6, §9) |
| **Wealth** | The invested assets UBS manages — the world's largest private-wealth book ✅; the deal-claim of "USD 5 trillion combined invested assets" ✅/⚠ |
| **Avaloq** | Swiss wealth-management core platform vendor, founded 1985 as BZ Informatik (BZ Bank's IT subsidiary) ✅; **NOT verified as UBS's core** ⚠ (§2.1) |
| **UBS Neo** | UBS's cross-asset investment-banking client portal ✅, built with Lab49 ✅ (§5) |
| **Key4** | UBS's purely digital product line, launched 19 May 2022 ✅ (§6) |
| **FINMA** | The Swiss financial regulator; supervises UBS's resilience, outsourcing and AML/CFT ⚠ (§9) |
| **SNB** | Swiss National Bank — provided liquidity support during the March 2023 rescue ⚠ (§1.4) |
| **Microsoft Azure** | UBS's primary cloud platform ✅; the Swiss anchor client story (2019) ✅ (§8) |

---

## 15. References and Further Reading

**Primary / bank materials (verified this pass):**
- UBS media release — "UBS to acquire Credit Suisse" (19 March 2023) ✅ — the rescue announcement; "USD 5 trillion" claim
- UBS media release — "UBS and Microsoft announce landmark expansion of cloud partnership" (20 October 2022) ✅ — Azure-primary, >50% of applications, 2018-strategy success
- UBS technology pages — "Innovation and AI at UBS" ✅ (the "AI enabled financial institution" ambition); "Cloud as an opportunity" ✅ (2019 Swiss anchor client)
- UBS media release — "UBS is expanding its leading digital offering with the purely digital product line UBS key4" (19 May 2022) ✅
- UBS client-logins page (ubs.com) ✅ — UBS Neo as the Investment Bank Client Portal; Digital Networks and Events
- UBS fraud-prevention page (US) ✅ — scams, cyber fraud, identity theft, check fraud
- UBS Group Annual Report 2023 ✅ (structure: five divisions) and Annual Report 2025 ✅ (CS client assets migrated onto UBS platforms)
- UBS LinkedIn — 9 Penang Road office announcement ✅ — ~400,000 sq ft, largest in APAC
- Microsoft — "UBS and Microsoft announce landmark expansion" (Microsoft Source, 20 Oct 2022) ✅; UBS Azure customer story ✅

**Press and trade coverage (verified this pass):**
- Britannica / Wikipedia — UBS and Avaloq entries ✅ — 1998 merger; Avaloq's BZ Informatik origin
- Business Insider — "UBS Buys Credit Suisse, With Bitter Digs Throughout" (March 2023) ✅ — authorities-initiated rescue
- Euromoney — "Credit Suisse and UBS: The new risks" ✅ — state-sponsored rescue purpose
- The Standard — rescue-takeover terms (~£2.7bn) ✅/⚠
- City AM — Q2 2023 results; June acquisition; $2.2bn integration expenses ✅/⚠
- Rio Times — "UBS Tackles Complex Swiss Account Migration" ✅ — end-2025 targets, CEO confidence
- execpartners — Q4 2025 disclosure summary (~85% Swiss-booked accounts migrated) ✅
- Netzwoche — "UBS setzt im grossen Stil auf generative KI" (Aug 2025) ✅/⚠ — 8M prompts, 280+ use cases
- The Business Times — "UBS launches pilot for new AI tool aimed at boosting productivity of advisers" (Oct 2025) ✅ — Singapore KYC-AI pilot
- finews — Key4 growth coverage; UBS integration "final stretch" coverage ✅/⚠
- neobanque.ch — key4 banking zero-fee (March 2025) ✅/⚠
- Lab49 case study — "UBS Neo: Building a cross-asset trading platform" ✅; LeapRate (Lab49 CEO statement) ✅
- powdigital — UBS NEO platform work; 2019 Profit & Loss Digital FX Award ✅
- Attivio customer story — UBS Neo ✅/⚠ (secondary)
- Citywire / InvestmentNews — GWM Solutions reorganisation (June 2024) ✅

**Series cross-references (plain filenames for banking/ siblings):**
- Pattern guides — [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md), [UOB Software Systems Guide](uob_software_systems_guide.md), [SMBC Software Systems Guide](smbc_software_systems_guide.md), [Standard Chartered Guide](standard_chartered_guide.md), [Deutsche Bank Software Systems Guide](deutsche_bank_software_systems_guide.md), [BNP Paribas Software Systems Guide](bnp_paribas_software_systems_guide.md)
- Umbrella/mechanics — [Core Banking Systems Guide](core_banking_systems_guide.md), [Wealth Management Guide](wealth_management_guide.md), [Payments Hub Guide](payments_hub_guide.md), [Interest Engines Core Banking Guide](interest_engines_core_banking_guide.md), [Posting Engine Core Banking Guide](posting_engine_core_banking_guide.md), [Core Banking Processes Guide](core_banking_processes_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md), [DBS Bank Guide](dbs_bank_guide.md)
- Platform classes — [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) (treasury/capital-markets; UBS vendor use NOT verified), [Temenos Guide](temenos_guide.md) / [T24 Programming Guide](t24_programming_guide.md) / [TAFJ Guide](tafj_guide.md) (verified NOT UBS-adjacent), [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md) (contrast class), [Insurance Software Systems Guide](insurance_software_systems_guide.md) (lightly)
- Risk/compliance/fraud — [Financial Risk & Compliance Systems Guide](financial_risk_compliance_systems_guide.md), [Financial Fraud Detection at Scale Guide](financial_fraud_detection_at_scale_guide.md)
- Technology-tree cross-refs (`../technology/` prefix) — [Enterprise AI Platforms Guide](../technology/ai_llm/enterprise_ai_platforms_guide.md), [AI Governance, Bias & Red-Teaming Guide](../technology/ai_llm/ai_governance_bias_redteaming_guide.md), [Legacy Integration Patterns Guide](../technology/legacy_integration_patterns_guide.md)

---

*End of guide. Verification status: 14 targeted searches this pass; web_extract degraded (search-only backend); all unverifiable claims flagged ⚠; the Avaloq-as-UBS-core claim is reported as unverified rather than asserted (§2.1), and the integration scale numbers are split into verified anchors (85% Swiss accounts, end-2025 AM migration, December 2026 decommissioning) and flagged figures (costs, savings, application counts) (§3.1).*

---

### Document Metadata

- **Series**: bank-software-systems (companion to the DBS/OCBC/UOB/SMBC/Standard Chartered/Deutsche Bank/BNP Paribas guides)
- **Subject**: UBS Group AG — software systems landscape
- **Audience**: Jack Liu Shurui (Solution Architect, Cymbal Bank, Singapore) and the research repo readership
- **Verification pass**: 14 targeted web searches; ✅/⚠ conventions as defined in the header
- **Headline flags**: Avaloq-as-UBS-core unverified (§2.1); CS-integration scale numbers partly unverified (§3.1); US digital-wealth detail thin (§4); IB estate beyond Neo opaque (§5); genAI specifics partly single-source (§7); SG depth mixed (§10)
- **Status**: complete (700+ lines), honest-flagging audit included (§13)
