# Bidot Tech Pte Ltd: The Singapore Custom-Software-Development Boutique — A Comprehensive Company Deep-Dive

**The "Strategists, Designers and Coders" Full-Stack Custom-Development Firm at 20 Cecil Street — Registry Record, Services, Positioning, Singapore Presence, the SG Custom-Dev Services Landscape, and a Custom-Dev Engagement/Delivery-Model Worked Example — Built Honestly on a Thin Public Footprint**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research / Company Deep-Dive — Bidot Tech: Company Overview (Registry Record), Services (Website Claims), Positioning (Strategists-Designers-Coders; IT Start-up), Singapore Presence (20 Cecil Street, PLUS), Market Context (the SG Custom-Dev Services Landscape), Worked Example (a Custom-Dev Engagement/Delivery Model), One-Page Summary, Glossary
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** bidot.sg (index, services, product, about, career, contact pages — extracted live), ACRA-derived registry aggregators (opengovsg.com/corporate/201915071R, companies.sg/business/201915071R, companieshouse.sg search — the UEN anchored record), the LinkedIn company page (sg.linkedin.com/company/bidot-tech — accessed via search snippet; page itself Cloudflare-blocked this pass), MyCareersFuture company page (mycareersfuture.gov.sg/companies/bidot-tech-201915071R), and the repository's own SG tech-cluster guides for market context
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder):** [Singapore SaaS Companies](singapore_saas_companies_guide.md), [Singapore Data Centres](singapore_data_centres_guide.md), [HTX NGINE](htx_ngine_guide.md), [ICA Systems](ica_systems_guide.md), [Micro-Frontend Architecture](micro_frontend_architecture_guide.md), [NEC Asia Pacific](nec_asia_pacific_guide.md)
> **Companion guides (banking/ and management/, prefix `../`):** [Bond Financial Group](../banking/bond_financial_group_company_guide.md), [Bond Capital Group](../banking/bond_capital_group_company_guide.md), [Partners Group Company Profile](../banking/partners_group_company_guide.md), [MBA Body of Knowledge](../management/mba_body_of_knowledge_guide.md), [Business Case Development](../management/business_case_development_guide.md), [Vendor Management](../management/vendor_management_guide.md)

---

## How to Read This Guide

This is the **dedicated deep-dive on Bidot Tech Pte Ltd** — a Singapore-incorporated custom-software-development company — in the `technology/` Singapore-tech / company-profile series. It follows the house pattern set by the recent thin-public-footprint company guides — [../banking/bond_financial_group_company_guide.md](../banking/bond_financial_group_company_guide.md) and [../banking/bond_capital_group_company_guide.md](../banking/bond_capital_group_company_guide.md) (entity → structure → business → footprint → worked example → claims-status discipline) and [nec_asia_pacific_guide.md](nec_asia_pacific_guide.md) (the verified-claims-table house style) — but with the **same deliberate honesty framing as the Bond guides**: Bidot Tech is a young exempt private company whose public footprint is thin *by construction*. Singapore exempt private companies file no public financial statements, the firm has generated essentially no press coverage, and its own website and LinkedIn page are the *only* sources for its services, positioning and track record. The guide therefore separates, section by section, what is **Verified** (the registry record; the website's own words, quoted verbatim) from what is **claimed but unverifiable** (every capability, client-adjacent and team claim) and flags the latter. Sibling guides carry adjacent depth and are cross-referenced inline:

- **The SG software-company landscape (heavy cross-ref)** — [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) is the *context this firm operates in*: the Singapore software ecosystem, its export-from-day-one logic, the government machinery (IMDA, EDB, EnterpriseSG, SGInnovate, MAS), the funding/valuation ladder, and the homegrown SaaS company set. Bidot Tech is *not* a SaaS company — it sells custom development services, not subscriptions — but it sits inside the same ecosystem, recruits from the same talent pool, and sells into the same buyers; §5 leans on this guide heavily.
- **The thin-public-footprint company-profile pattern (heavy cross-ref)** — [../banking/bond_financial_group_company_guide.md](../banking/bond_financial_group_company_guide.md) sets the exact honesty discipline this guide follows: verified registry backbone, verbatim website-claim quotations, ⚠ flags on everything unverifiable, a claims-status ledger, and a fictional worked example that stress-tests the real firm's blueprint. [../banking/bond_capital_group_company_guide.md](../banking/bond_capital_group_company_guide.md) is the sibling profile of the same address-cluster pattern. [../banking/partners_group_company_guide.md](../banking/partners_group_company_guide.md) supplies the entity-overview and worked-example conventions for well-documented firms (the contrast case).
- **The SG tech infrastructure (light cross-ref)** — [singapore_data_centres_guide.md](singapore_data_centres_guide.md) (SG data-residency and hosting — relevant to any cloud-deployed custom build), [htx_ngine_guide.md](htx_ngine_guide.md) and [ica_systems_guide.md](ica_systems_guide.md) (the SG public sector as anchor buyer and standards-setter — §5.3).
- **The frontend angle (light cross-ref)** — [micro_frontend_architecture_guide.md](micro_frontend_architecture_guide.md) covers the modern frontend architecture patterns that any full-stack custom-build delivery model in §6 must decide on; [project_management_methodologies_guide.md](project_management_methodologies_guide.md) and [../management/vendor_management_guide.md](../management/vendor_management_guide.md) supply the delivery-and-vendor vocabulary for the worked example.
- **The business-model vocabulary (light cross-ref)** — [../management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) (services business economics, positioning) and [../management/business_case_development_guide.md](../management/business_case_development_guide.md) (the commercial case a buyer builds before commissioning custom software) frame §5 and §6.
- **The bank-buyer context (light cross-ref)** — the `banking/` guides (e.g. [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md), [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md)) show how regulated institutions buy software; §6's worked example is written from that buyer's seat, because that is this series' familiar context.

**Note on verification — the honesty framing.** This guide was researched in August 2026 via targeted web search and page extraction against: (1) ACRA-derived company-registry aggregators (opengovsg.com, companies.sg, companieshouse.sg, sgpbusiness.com — the latter blocked to the extractor, facts taken from search-indexed content instead) for the corporate record; (2) the corporate website bidot.sg (index, services, product, about, career, contact — all extracted live, plus the three product case-study subpages); and (3) the LinkedIn company page via search-engine snippet (the page itself is behind Cloudflare). **Bidot Tech's public footprint is thin by construction**: it files no public annual reports (exempt private company), it has generated essentially no press coverage, and its own website is the *only* source for its services and positioning. Every claim about the firm that could not be verified against the registry or the website's own words is **flagged ⚠ with "thin public footprint" language**. Nothing here is invented: no clients, no projects, no team sizes, no revenue, no ownership data have been fabricated — where the public record is silent, the guide says so and moves on. Two corrections to the research brief are flagged prominently in §1: the registered SSIC code is **62011** (not the brief's paraphrase "software/application development (except games and cybersecurity)"), and the registered office is **#23-03** in the registry (the website prints "#23-03/04"). Nothing in this guide is investment, legal, or procurement advice.

### Quick Facts (Summary Card)

| Item | Value |
|---|---|
| Entity | **Bidot Tech Pte. Ltd.** — Singapore **Exempt Private Company Limited by Shares** (Local Company) |
| UEN | **201915071R** (issued by ACRA 29 May 2019) — **Verified** |
| Incorporation | **10 May 2019**, Singapore — **Verified** |
| Status | **Live Company** — **Verified** |
| Registered office | 20 Cecil Street, #23-03, PLUS, Singapore 049705 (registry); website prints "#23-03/04" — **Verified with flag** |
| Registered activity | SSIC **62011** — development & marketing of software program & website and online marketing — **Verified** |
| Former name | **DONG ER TECHNOLOGY** (reason for and date of rename not public in this pass) — **Verified as registry fact; circumstances flagged** |
| Paid-up capital | S$7,401,440 per companies.sg — **aggregator-reported, flagged** |
| Officers | 7 per opengovsg.com — **Verified with flag** (aggregator figure; names not public in this pass) |
| Self-description | "Bidot Tech is a **full-stack custom software and application development services provider**. We are **strategists, designers and coders**!" — **Verified as a website claim** (home-page meta description, verbatim) |
| Stated positioning | "an Information Technology (IT) **start-up**... led by outstanding entrepreneurs" — **Verified as a website/LinkedIn claim** ⚠ |
| Stated products | Privacy & Security mobile app; elderly/charity apps; EHR (Electronic Health Record) system — **Verified as website claims** ⚠ |
| Partner logos (website) | Crawfurd Hospital, Hug 2 Hearts, Alibaba Cloud, Digital Ocean, Gcore, Google, Linode, Upcloud — **Verified as website logos; relationship nature flagged** ⚠ |
| Press coverage | None found in research — **thin public footprint** ⚠ |
| Bidot name cluster | Bidot Investment Holdings Pte. Ltd. (202025098H, Live, #23-04) and Bidot Pte. Ltd. (202139794N, Struck Off, #23-03) — **Verified as registry observations; relationship to Bidot Tech flagged** ⚠ |

---

## Table of Contents

1. [Company Overview](#1-company-overview)
   - 1.1 [The Incorporation — 10 May 2019](#11-the-incorporation--10-may-2019-verified)
   - 1.2 [The UEN — 201915071R](#12-the-uen--201915071r-verified)
   - 1.3 [The Entity Type — Exempt Private Company Limited by Shares](#13-the-entity-type--exempt-private-company-limited-by-shares-verified)
   - 1.4 [The Registered Office — 20 Cecil Street, #23-03, PLUS](#14-the-registered-office--20-cecil-street-2303-plus-verified)
   - 1.5 [The Registry Record — SSIC, Officers, Filings, Capital, Former Name](#15-the-registry-record--ssic-officers-filings-capital-former-name-verified)
   - 1.6 [The Overview Table](#16-the-overview-table)
   - 1.7 [The Thin-Public-Footprint Note — a Method Disclaimer](#17-the-thin-public-footprint-note--a-method-disclaimer)
   - 1.8 [The Bidot Name Cluster — Same Address, Family Names](#18-the-bidot-name-cluster--same-address-family-names-flagged-observation)
   - 1.9 [The Milestones Timeline](#19-the-milestones-timeline)
2. [The Services](#2-the-services)
   - 2.0 [The Site Inventory — the Six-Page Footprint](#20-the-site-inventory--the-six-page-footprint-verified)
   - 2.1 [The Full-Stack Claim — "A Full-Stack Development Cycle from A to Z"](#21-the-full-stack-claim--a-full-stack-development-cycle-from-a-to-z-verified-as-a-website-claim)
   - 2.2 [The Service Lines — Data Security, Healthcare IT, Mobile Apps, Secure Data Transfer](#22-the-service-lines--data-security-healthcare-it-mobile-apps-secure-data-transfer-verified-as-website-claims)
   - 2.3 [The Technical Knowledge Stack](#23-the-technical-knowledge-stack-verified-as-website-claims)
   - 2.4 [The Product Case Studies — Security App, Elderly Apps, EHR](#24-the-product-case-studies--security-app-elderly-apps-ehr-verified-as-website-claims)
   - 2.5 [The Partner Logos — Real Organisations, Unverified Relationships](#25-the-partner-logos--real-organisations-unverified-relationships-flagged)
   - 2.6 [The Services Table — Service / Evidence / Notes](#26-the-services-table--service--evidence--notes)
   - 2.7 [Reading the Services Honestly](#27-reading-the-services-honestly)
   - 2.8 [Adoption — The Honest Caveat](#28-adoption--the-honest-caveat-flagged)
3. [The Positioning](#3-the-positioning)
   - 3.1 [The Strategists-Designers-Coders Claim](#31-the-strategists-designers-coders-claim-verified-as-a-website-claim)
   - 3.2 [The About Page — the IT Start-up](#32-the-about-page--the-it-start-up-verified-as-website-claims)
   - 3.3 [The LinkedIn Positioning](#33-the-linkedin-positioning-flagged-as-claims)
   - 3.4 [The Counter-Stats Artifact — "40 / 0 / 0 / 0"](#34-the-counter-stats-artifact--40--0--0--0-flagged-template-artifact)
   - 3.5 [The Career Page — "We're Not Just Colleagues. We're Family."](#35-the-career-page--were-not-just-colleagues-were-family-verified-as-website-claims)
   - 3.6 [The Positioning Table](#36-the-positioning-table)
   - 3.7 [Positioning vs Footprint — the Honest Gap](#37-positioning-vs-footprint--the-honest-gap)
4. [The Singapore Presence](#4-the-singapore-presence)
   - 4.1 [The Cecil Street PLUS Office](#41-the-cecil-street-plus-office-verified)
   - 4.2 [Cecil Street as a Commercial Address](#42-cecil-street-as-a-commercial-address-verified-context)
   - 4.3 [The SG Table](#43-the-sg-table)
   - 4.4 [The Same-Address Observation — a Serviced-Office Pattern](#44-the-same-address-observation--a-serviced-office-pattern-flagged)
5. [The Market Context — the SG Custom-Dev Services Landscape](#5-the-market-context--the-sg-custom-dev-services-landscape)
   - 5.1 [The Ecosystem Bidot Tech Incorporated Into](#51-the-ecosystem-bidot-tech-incorporated-into-verified-context)
   - 5.2 [The Tiers of the Custom-Dev Services Market](#52-the-tiers-of-the-custom-dev-services-market)
   - 5.3 [Government as Buyer and Standards-Setter](#53-government-as-buyer-and-standards-setter-verified-context)
   - 5.4 [Where Bidot Tech Sits in the Landscape](#54-where-bidot-tech-sits-in-the-landscape)
   - 5.5 [The Context Table](#55-the-context-table)
   - 5.6 [The Market Bottom Line](#56-the-market-bottom-line)
6. [The Worked Example — a Custom-Dev Engagement/Delivery Model](#6-the-worked-example--a-custom-dev-engagementdelivery-model)
   - 6.1 [The Scenario — CareLane and the Boutique Builder](#61-the-scenario--carelane-and-the-boutique-builder)
   - 6.2 [The Model Design — the Five-Phase Delivery Model](#62-the-model-design--the-five-phase-delivery-model)
   - 6.3 [The Delivery Model at a Glance](#63-the-delivery-model-at-a-glance)
   - 6.4 [The Decision Matrix](#64-the-decision-matrix)
   - 6.5 [The Lessons](#65-the-lessons)
   - 6.6 [The Risk Register](#66-the-risk-register)
   - 6.7 [The Vendor Scorecard and the Phase Timeline](#67-the-vendor-scorecard-and-the-phase-timeline)
7. [The Summary — Bidot Tech in One Page](#7-the-summary--bidot-tech-in-one-page)
8. [Glossary](#8-glossary)
9. [Claims Status, References and Further Reading](#9-claims-status-references-and-further-reading)

---

## 1. Company Overview

### 1.1 The Incorporation — 10 May 2019 (Verified)

Bidot Tech Pte. Ltd. was incorporated in Singapore on **10 May 2019** — verified against two independent ACRA-derived registry aggregators (opengovsg.com/corporate/201915071R and companies.sg/business/201915071R/BIDOT-TECH-PTE-LTD-), which agree exactly on the date. The UEN was issued by ACRA on **29 May 2019** (registry "UEN Issue Date" field) — i.e. the name/registration was completed in the latter part of May 2019, with 10 May 2019 as the statutory incorporation date.

What this means in context: Bidot Tech is a **post-2018-vintage Singapore company** — it incorporated in the same era as the "Gen 3 / AI-native" wave described in [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) §1.4, though as a *services* firm rather than a product company. It is not a freshly-minted shell (the 2024-vintage pattern of the Bond guides) nor a decades-old established firm (the NEC Asia Pacific pattern) — it is a **seven-year-old privately held services company** whose registry record shows continuous live status and current filing obligations, but whose public footprint beyond the registry is thin. As of this pass (August 2026), the company is **7.3 years old** per companies.sg's calculation.

### 1.2 The UEN — 201915071R (Verified)

The Unique Entity Number **201915071R** is verified via both aggregators and via the government's own MyCareersFuture portal, whose company page URL embeds the UEN (mycareersfuture.gov.sg/companies/bidot-tech-201915071R). The UEN structure is standard ACRA format: the leading **2019** encodes the registration year, **15071** is the sequence within that year's issue block, and the trailing **R** is the checksum letter. For a company with as generic a name as "Bidot Tech" — a name that collides with at least two other live/former "Bidot" entities (§1.8) and with a global IT-services firm called "Bidot" in other markets — **the UEN, not the name, is the entity's safe identifier**, exactly the discipline the Bond guides (§1.2 in each) establish for their subjects.

### 1.3 The Entity Type — Exempt Private Company Limited by Shares (Verified)

The registry classifies Bidot Tech as a **Local Company** with company type description **"Exempt Private Company Limited by Shares"** — verified via both opengovsg.com and companies.sg. Under the Companies Act (Cap. 50), an exempt private company (EPC) is a private company limited by shares with **no more than 20 members** and **no corporate member** (with narrow exceptions). The "exempt" in the name refers to audit exemption: an EPC that meets the small-company criteria is exempt from audit (though it must still file annual returns and, if applicable, unaudited accounts — and, critically for this guide's honesty framing, **EPCs' financial statements are not part of the public register in the way larger companies' are**).

The consequence is the same one the Bond guides flag: **no public financials exist for Bidot Tech.** Revenue, profitability, headcount cost base, and balance-sheet health are all unverifiable from the public record. The EPC status also explains the "thin public footprint" — it is a legal structure that *maximises privacy*, and Bidot Tech uses it. Every buyer-side question about financial substance (§6) therefore has to be answered by the vendor's own disclosure, not by the register.

### 1.4 The Registered Office — 20 Cecil Street, #23-03, PLUS (Verified)

The registered office is **20 Cecil Street, #23-03, PLUS, Singapore 049705** — verified via opengovsg.com (address, building "PLUS", level 23, unit 03, postal 049705) and companies.sg. Two honest nuances to flag:

- **The website prints "#23-03/04"** on its contact page ("20 Cecil Street, #23-03/04 PLUS, Singapore 049705"), while the registry record shows **#23-03** only. The most plausible reading — flagged, not asserted — is that the firm presents the suite pair because the sister entity **Bidot Investment Holdings Pte. Ltd.** (UEN 202025098H) is registered at **#23-04** (§1.8). The two suites are adjacent; the website's combined "23-03/04" is consistent with a shared premises arrangement, but the ownership/lease relationship is not public.
- The address is a **commercial building in the CBD fringe** (the PLUS building at 20 Cecil Street, one of the older-generation office blocks between Raffles Place and Tanjung Pagar) — a standard, genuinely commercial registered-address pattern, *not* a residential or virtual-mailing oddity. §4 covers the location in detail.

### 1.5 The Registry Record — SSIC, Officers, Filings, Capital, Former Name (Verified)

| Registry field | Value | Status |
|---|---|---|
| Entity name | BIDOT TECH PTE. LTD. | Verified |
| UEN | 201915071R | Verified |
| Incorporation date | 10 May 2019 | Verified |
| UEN issue date | 29 May 2019 | Verified |
| Entity status | Live Company | Verified |
| Entity type | Local Company — Exempt Private Company Limited by Shares | Verified |
| Registered office | 20 Cecil Street, #23-03, PLUS, Singapore 049705 | Verified |
| Primary SSIC | **62011** — Development & marketing of software program & website and online marketing | Verified |
| Secondary SSIC | Same code (62011) per companies.sg | Verified (aggregator) |
| Number of officers | 7 | Verified with flag (aggregator; names not public in this pass) |
| Annual return date | 2026-07-08 | Verified (aggregator) |
| Account due date | 2027-11-30 | Verified (aggregator) |
| Paid-up capital | S$7,401,440 | Flagged — companies.sg only; not independently confirmed |
| Former entity name | DONG ER TECHNOLOGY | Verified as registry fact; rename circumstances flagged |
| Other registers | Appears in IPOS trademark-owner records (companies.sg "Licenses & Accreditations") | Flagged — existence only; trademark scope not examined |

Three items deserve explicit honest commentary:

1. **The SSIC correction.** The research brief described the registered activity as "software/application development (except games and cybersecurity)". The *actual* registered code is **SSIC 62011 — "Development & marketing of software program & website and online marketing"** (both primary and secondary, per companies.sg). SSIC 62011 sits within Division 62 (computer programming, consultancy and related activities) of the Singapore Standard Industrial Classification; the "except games and cybersecurity" phrasing in the brief resembles the *exclusionary* descriptions used elsewhere in Division 62 (e.g. 62012 game/software publishing and 62022 cybersecurity-adjacent categories are separate codes). The honest statement is: **the registry says 62011**, and 62011 is a software-development-and-marketing code consistent with a custom-dev agency's declared business — but the specific code matters for anyone doing SSIC-based diligence, and the brief's paraphrase should not be quoted as the registered code.
2. **The former name — DONG ER TECHNOLOGY.** Both aggregators record **"Former Entity Name: DONG ER TECHNOLOGY"** (also surfaced in the recordowl/ltddir family of aggregators). The company was evidently registered under a Chinese-style trading name at or shortly after incorporation and later renamed to BIDOT TECH PTE. LTD. The rename date and reason are **not public in this pass** — the aggregators show only the former name, not the renaming event — so this is flagged as a registry fact with unknown circumstances. (It is also a reminder that "Bidot" is a brand adopted *after* incorporation, not the founding name.)
3. **The paid-up capital of S$7.4M.** companies.sg reports paid-up capital of **S$7,401,440**. That is a strikingly high figure for a boutique custom-dev shop (typical agency paid-up capital is in the S$1K–S$100K range), and it is **flagged**: it comes from a single aggregator, the composition (cash vs assets-in-kind vs a capital restructure) is unknown, and it should be treated as directional evidence of owner capital commitment — *not* as verified financial strength. The "Trustworthiness Analysis" box on companies.sg itself rates Bidot Tech "**Medium**" risk and flags exactly this kind of open question, plus the former-name change and the need to verify genuine occupancy of the registered unit. That third-party assessment is reproduced here as context, not endorsement.

### 1.6 The Overview Table

| Aspect | Description |
|---|---|
| Entity | Bidot Tech Pte. Ltd. — Singapore exempt private company limited by shares (Local Company) |
| UEN | 201915071R (ACRA, issued 29 May 2019) |
| Incorporation | 10 May 2019, Singapore — live and filing-current as of this pass |
| Registered office | 20 Cecil Street, #23-03, PLUS, Singapore 049705 (website prints #23-03/04) |
| Registered activity | SSIC 62011 — development & marketing of software program & website and online marketing |
| Former name | DONG ER TECHNOLOGY (rename date/reason not public) |
| Stated business | Full-stack custom software and application development services (website, verbatim) |
| Stated identity | "Strategists, designers and coders"; "an Information Technology (IT) start-up" (website/LinkedIn, verbatim) |
| Stated domains | Data security, healthcare IT, mobile apps, secure data transfer; products: privacy/security app, elderly/charity apps, EHR system (website claims) |
| Contact | info@bidot.sg; +65 6223 0308; Mon–Fri 9am–6pm (website) |
| Public footprint | Thin by construction: no public financials (EPC), no press coverage found, no named clients, no team-size data |
| Buyer posture | Verify substance directly (references, financials ask, code ownership, contracts) before engaging — §6 |

### 1.7 The Thin-Public-Footprint Note — a Method Disclaimer

Bidot Tech's public record consists of exactly three layers, and the guide is built on all three without conflating them:

1. **The registry layer (verified).** UEN, incorporation date, entity type, status, address, SSIC, officer count, filing dates, former name — the facts of §1.1–§1.5. This layer is hard evidence.
2. **The self-published layer (verified *as claims*).** Every word on bidot.sg and the LinkedIn page — services, stack, products, partners, mission, values. This layer is the company's own marketing, quoted verbatim and treated as claims about itself, not as third-party evidence. (§2–§3.)
3. **The absence layer (flagged).** No public financials, no press coverage, no independent reviews found, no named clients verifiable, no audited adoption data. Absence of evidence is not evidence of absence — a young private EPC is *structurally* quiet — but it is the reason every unverifiable claim carries a ⚠ in this guide.

**Nothing in this guide fabricates the missing layers.** No invented clients (the partner logos of §2.5 are real organisations, but the *relationship* is undisclosed); no invented projects (the product case studies are website claims, not verified engagements); no invented team sizes (the 7 officers is an officer count, not a headcount — a company can have 7 registered officers and 2 staff, or 7 and 70); no invented revenue (none exists publicly). Where the record is silent, the guide says so.

**Where Bidot Tech sits in the series (the pattern contrast).** The thin-footprint company guides form a spectrum, and Bidot Tech occupies a distinct point on it — the *operating boutique* rather than the *young holding shell*:

| Dimension | Bond Financial Group (2024) | Bond Capital Group (2021) | **Bidot Tech (2019)** |
|---|---|---|---|
| Age at research | ~2 years — freshly incorporated | ~5 years | **~7 years — an operating business, not a shell** |
| Stated business | Holding company, "global finance" claims | Same-address sibling | **Operating services firm: full-stack custom development** |
| Business evidence | Website-only | Website-only | **Website + registry-consistent operating story (SSIC 62011) + product case studies + employer footprint (career page)** |
| Public financials | None (EPC) | None (EPC) | **None (EPC) — S$7.4M paid-up capital flagged, aggregator-only** |
| Press coverage | None found | None found | **None found** |
| Guide's worked example | Holding-structure design | (sibling pattern) | **A buyer-side delivery model — because the firm sells engagements, the buyer's question is how to buy them safely** |

The through-line is the same discipline in all three: **verify the registry, quote the claims verbatim, flag everything else, and let the worked example stress-test the firm's own blueprint** — here, the blueprint being stress-tested is the boutique's "full-stack from A to Z" pitch, and the test is run from the buyer's seat (§6).

### 1.8 The Bidot Name Cluster — Same Address, Family Names (flagged observation)

The registry aggregators reveal a small cluster of "Bidot" entities at the same Cecil Street address:

| Entity | UEN | Incorporated | Address | Status |
|---|---|---|---|---|
| **Bidot Tech Pte. Ltd.** | 201915071R | 10 May 2019 | 20 Cecil St #23-03 | Live Company |
| **Bidot Investment Holdings Pte. Ltd.** | 202025098H | 20 Aug 2020 | 20 Cecil St #23-04 | Live Company |
| **Bidot Pte. Ltd.** | 202139794N | 16 Nov 2021 | 20 Cecil St #23-03 | Struck Off |

This is the same "same-address, family-of-names" observation the Bond guides make about their subjects (§4.4 in bond_financial_group_company_guide.md), and it is flagged with the same discipline: **the entities exist, the addresses are verified, but the ownership/control relationships between them are not public and are NOT asserted here.** Plausible readings (a holding company for the operating firm; a sister brand that was wound down; a common founder family) are left as readings, not findings. For diligence purposes the practical takeaway is the name-collision lesson of §1.2: "Bidot" does not resolve to a single entity, so contracts, invoices and correspondence should always key on the UEN. (The aggregators' "similar businesses" lists also show that 20 Cecil Street hosts many unrelated tenants — Conduit Group VCC, several 2026-vintage incorporations, and others — consistent with a multi-tenant commercial building / serviced-office environment; §4.4.)

### 1.9 The Milestones Timeline

| Date | Milestone | Status |
|---|---|---|
| 10 May 2019 | Incorporated as an exempt private company; registered at 20 Cecil Street, #23-03, PLUS (initially under the name DONG ER TECHNOLOGY — rename event date not public) | Verified (registry) |
| 29 May 2019 | UEN 201915071R issued by ACRA | Verified (registry) |
| 20 Aug 2020 | Sister entity Bidot Investment Holdings Pte. Ltd. incorporated at #23-04 | Verified (registry observation) |
| 16 Nov 2021 | Sister-name entity Bidot Pte. Ltd. incorporated at #23-03 (subsequently struck off) | Verified (registry observation) |
| ~2022 | Website's own photo caption: "2022 Team Building @ Bintan" — evidence of an in-person team, size unstated | Verified as website content ⚠ |
| 2026 | Live status; annual return dated 2026-07-08 on record; account due 30 Nov 2027; IPOS trademark-owner record exists | Verified (aggregators) |
| Aug 2026 | This research pass: website live (bidot.sg), LinkedIn live (336 followers per search snippet), no press coverage found | Verified as of this pass |

---

## 2. The Services

### 2.0 The Site Inventory — the Six-Page Footprint (Verified)

The entire public web presence of Bidot Tech is a small static site (bidot.sg, served by nginx, Bootstrap-era template with JS-loaded header/footer). The complete page inventory, each extracted live in this pass:

| Page | URL | What it claims |
|---|---|---|
| Home | bidot.sg/ | "We help founders and product owners to turn ideas into reality"; "a full-stack development cycle from A to Z" [sic]; Services teaser; Our Partners logos; contact block |
| Services | bidot.sg/services.html | Four service lines (Data Security, Healthcare IT, Mobile Apps, Secure Data Transfer); five technical-knowledge areas (Product Design, Mobile Apps, Industry Analysis, Programming, Cloud Computing) |
| Products | bidot.sg/product.html | Three product case studies (Privacy & Security app, elderly/charity apps, EHR system) with tech-stack and role lines |
| About | bidot.sg/about.html | "We are strategists, designers and coders!"; IT start-up self-description; mission; vision; counter strip |
| Career | bidot.sg/career.html | Culture pitch ("We're not just colleagues. We're family."); values; benefits; Bintan team-building caption |
| Contact | bidot.sg/contact.html | info@bidot.sg; +65 6223 0308 (Mon–Fri 9am–6pm); 20 Cecil Street, #23-03/04 PLUS, Singapore 049705 (with map link) |
| Case-study subpages | product_security_app.html, product_charity.html, product_ehr.html | Full Description / Tech stack / Our role / Problem / Result write-ups |

Notable absences from the site: **no portfolio gallery beyond the three case studies, no client names, no testimonials, no case-study dates, no team page, no pricing, no certifications, no blog, no press page, no privacy policy or terms of service pages, and no sitemap.xml** (a sitemap probe returns 404 — the six-page structure above is the whole site). The site's meta keywords ("application development, data security, technology, information technology, IT, programming cloud computing, blockchain, digital marketing") gesture at blockchain and digital marketing that appear nowhere else on the site. The thinness of the site *is* evidence — of a firm that wins work through direct contact ("Share with us your ideas now") rather than through published proof.

### 2.1 The Full-Stack Claim — "A Full-Stack Development Cycle from A to Z" (Verified as a Website Claim)

The firm's entire service proposition is compressed into two sentences on the bidot.sg home page, quoted verbatim:

> "We help founders and product owners to turn ideas into reality."
> "We offers a full-stack development cycle from A to Z." [sic — "offers" is the site's own grammar]

and, in the home page's HTML meta description (the exact wording the research brief anticipated):

> "Bidot Tech is a **full-stack custom software and application development services provider**. We are **strategists, designers and coders**! Get in touch with us!"

Three honest notes on this claim. First, it is a **self-description** — verified as *the website's claim*, not as third-party capability evidence. Second, the claim is a **positioning statement** ("full-stack custom software and application development services provider") that is *consistent with* the registry's SSIC 62011 but goes beyond it — SSIC describes registered activity classes, not delivered quality or breadth. Third, the "full-stack" promise ("from A to Z") is the classic boutique-agency pitch: one vendor for strategy, design, build, QA, DevOps and support. Whether Bidot Tech delivers that end-to-end scope on real engagements is exactly what the public record cannot show — hence the ⚠ and the §6 diligence design.

### 2.2 The Service Lines — Data Security, Healthcare IT, Mobile Apps, Secure Data Transfer (Verified as Website Claims)

The services page (bidot.sg/services.html) opens: *"Experience in developing a range of software... We are continually expanding our list of services to serve you better. Here are some of what our innovative and experienced team does best here! Bidot is using the most modern and most demanded technologies to create an outstanding project."* Four service lines are named:

- **Data Security** — the most elaborated line: *"we analyze your needs and improve your security posture by combining several of the following techniques: Data discovery and classification, Data masking, Identity access management, Data encryption, Data loss prevention, Governance, risk, and compliance, Password hygiene, Authentication and authorization, Data security audits, as well as Anti-malware, antivirus, and endpoint protection."* This is a competent checklist of standard security-engineering controls — as *marketed* capabilities. No certifications (ISO 27001, SOC 2, CSA Cloud Security) are claimed on the page.
- **Healthcare IT Solution** — listed as a headline service with no elaboration on the services page itself; the healthcare angle is elaborated through the product pages (§2.4: EHR system) and the Crawfurd Hospital partner logo (§2.5).
- **Mobile Application Development** — headline service; elaborated through the mobile-apps stack (§2.3) and the two mobile product case studies (§2.4).
- **Secure Data Transfer** — headline service; elaborated through the Privacy & Security Mobile App case study (an encryption-tunnel/VPN-style product, §2.4).

The page's closing pitch — *"Ready to take your ideas to the next level?"* — and the home page's *"Share with us your ideas now"* call-to-action make the engagement model explicit: **this is a build-to-order services firm selling custom development, not a product company selling subscriptions** (the contrast with the [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) subjects is deliberate and structural).

### 2.3 The Technical Knowledge Stack (Verified as Website Claims)

The services page presents five "key areas of technical knowledge", with technology logos:

| Area | Claimed contents (verbatim from site) |
|---|---|
| **Product Design** | UI/UX, Mobile App Design, Web Design, Brand Identity, Illustrations |
| **Mobile Apps** | Flutter, Swift, Objective-C, Java, Kotlin, React, Firebase |
| **Industry Analysis** | Scientific research, Data Analysis, Analyse Competitors, Strategic Planning |
| **Programming** | C#, .NET (logo "net-framework"), Golang, Python, React, Vue, Phaser, TypeScript, CSS, MySQL, PostgreSQL |
| **Cloud Computing** | AWS, Azure, Google Cloud, Alibaba Cloud, Digital Ocean, Kubernetes, Docker, ELK Stack |

Honest reading: this is a **plausible and internally consistent boutique full-stack stack** — Flutter/React front-ends, a C#/.NET + Python + Go middle, MySQL/PostgreSQL persistence, and multi-cloud (AWS/Azure/GCP/Alibaba/DigitalOcean) with containerisation (Docker/Kubernetes) and an ELK observability set. It matches the stacks named on the product pages (§2.4) and the cloud partner logos (§2.5). What it is *not* is an audited inventory of production experience — logo walls are marketing, and the absence of any cloud-certification badges (e.g. "AWS Partner", "Google Cloud Partner" credentials) is noted as an honest observation, not a defect. The presence of **Phaser** (a game-engine framework) in the programming list and **blockchain/digital marketing** in the home-page meta keywords are minor curiosities — capabilities the site gestures at but does not substantiate anywhere else.

### 2.4 The Product Case Studies — Security App, Elderly Apps, EHR (Verified as Website Claims)

The product page (bidot.sg/product.html) presents **three product case studies**, each with Description / Tech stack / Our role / Problem / Result sections — the closest thing the public record has to a portfolio:

**1. Privacy & Security Mobile App** (product_security_app.html)
- *Description:* "provides a secure connection between you and the internet through an encryption tunnel. It encrypts your internet connection by concealing your IP... the ability to freely access the internet is a basic right."
- *Tech stack:* Frontend JavaScript, Flutter; Backend PostgreSQL, React.js, TypeScript (page header also lists Flutter, protobuf, Android Studio, gRPC, Xcode).
- *Our role:* "Development, UX/UI Design, QA, DevOps, Support."
- *Problem:* public-Wi-Fi interception, surveillance, ISP throttling.
- *Result:* "with one account, you can secure up to 5 devices at the same time... fast connection speed and unlimited bandwidth."

**2. The Charity / Elderly Apps** (product_charity.html)
- *Description:* "Bidot Tech helps to create apps catered towards the elderly to enhance their lifestyle. These apps serve several purposes such as reading news, exercising, maintaining communication with family and friends, and managing their daily activities."
- *Tech stack:* same Flutter / React.js / PostgreSQL / TypeScript pattern (page header adds JavaScript, Python).
- *Our role:* Development, UX/UI Design, QA, DevOps, Support.
- *Problem:* elderly users' technology anxiety, vision problems, complex UIs, lack of support.
- *Result:* "At Bidot Tech, we advocate for inclusive design to improve the app's usability... every senior citizen utilizes the features in an easy way!" — and, tellingly, the page refers to "the **mobile app development agency**" as the responsibility-bearer for accessible design: the firm's own vocabulary is that of an *agency*.

**3. Electronic Health Record (EHR) System** (product_ehr.html)
- *Description:* "Bidot Tech's EHR System is a user-friendly and highly configurable software built to meet the needs of all health organizations, no matter the size." Features listed: "Automatic Reminders, Prescriptions Tracking, Patient Flow Tracking, Inventory Management, Note Template Creation, Patient Portal and many more."
- *Tech stack:* JavaScript, Python, PostgreSQL (header); same Flutter/React/PostgreSQL/TypeScript role-line as the others.
- *Problem:* paper-folder chart retrieval inefficiency.
- *Result:* "providing accurate, up-to-date, and complete information about patients at the point of care... reducing the incidence of medical error" (the site's own typo "educing" is preserved in the original).

Honest reading: these are **website portfolio claims** — self-published case studies with no named client, no dates, no quantified outcomes and no third-party verification. The healthcare thread (EHR + Crawfurd Hospital logo + "Healthcare IT Solution" service line) and the charity thread (elderly apps + Hug 2 Hearts logo) are the firm's most coherent narrative arcs, and they are consistent with each other — but "consistent marketing" is not "verified delivery". A buyer in a regulated domain (healthcare data!) would need reference calls, code inspection and contract terms before taking any of it at face value (§6).

### 2.5 The Partner Logos — Real Organisations, Unverified Relationships (flagged)

The home page's "Our Partners" strip displays eight logos:

- **Crawfurd Hospital** — **real**: a Singapore private hospital (19 Adam Road, Singapore 289891; originated from Crawfurd Medical Clinic; relaunched under Crawfurd Medical's ownership in July 2021 after the former Fortis/Concord-era hospital was suspended and divested; registered Crawfurd Hospital Pte. Ltd., UEN 201103901W). The EHR/healthcare narrative makes this logo coherent — but the *nature* of the relationship (client? partner? pro-bono? logo placement?) is not disclosed anywhere on the site.
- **Hug 2 Hearts** — **real**: the charity brand of **HUG Community Services Limited**, a Singapore non-profit social-service agency (established 2014; NCSS member; IPC-registered charity serving at-risk youths and families; hashtag #HUG2HEARTS on its channels). The elderly-apps narrative makes this logo coherent too — again, relationship nature undisclosed.
- **Alibaba Cloud, Digital Ocean, Gcore, Google, Linode, Upcloud** — **real** cloud/hosting platforms. These are almost certainly **technology partners** (cloud-credit/affiliate programmes are common for small dev shops) rather than clients; the site's own cloud stack (§2.3) lists AWS, Azure, GCP, Alibaba Cloud and Digital Ocean, and the logos mirror that set.

Honest framing: the two non-cloud logos point at real organisations in the firm's claimed verticals (healthcare, charity/elderly), which is genuinely useful signal — it suggests the firm *has some relationship* in those spaces — but **what** that relationship is, and whether it constitutes client work, is unverified and flagged as such. The cloud logos are consistent with the site's stack claims and carry no client implication.

### 2.6 The Services Table — Service / Evidence / Notes

| Service | Evidence | Notes |
|---|---|---|
| Full-stack custom software & application development | Home-page meta description: "a full-stack custom software and application development services provider"; home-page tagline "a full-stack development cycle from A to Z" [sic] | ⚠ Website claim (verbatim, extracted live). Consistent with SSIC 62011; scope/breadth of delivery unverifiable. |
| Product design (UI/UX, web/mobile design, brand identity, illustrations) | Services page "Product Design" area; product pages list "UX/UI Design" in "Our role" | ⚠ Website claim. No portfolio/design-case evidence beyond the product pages' screenshots. |
| Mobile application development | Services page "Mobile Apps" area (Flutter, Swift, Obj-C, Java, Kotlin, React, Firebase); two mobile product case studies (security app, elderly apps) | ⚠ Website claims. Stack internally consistent; no named app in app stores found in this pass (not checked exhaustively). |
| Data security services | Services page "Data Security" checklist (discovery/classification, masking, IAM, encryption, DLP, GRC, password hygiene, auth, audits, endpoint protection); Privacy & Security App case study | ⚠ Website claim. Standard controls checklist; no security certifications (ISO 27001/SOC 2) claimed on site; healthcare-grade security claims would need independent audit. |
| Secure data transfer | Services page "Secure Data Transfer" headline; privacy-app encryption-tunnel case study | ⚠ Website claim. Product narrative only; no enterprise/regulated-entity references verifiable. |
| Healthcare IT | Services page "Healthcare IT Solution" headline; EHR case study; Crawfurd Hospital partner logo | ⚠ Website claim. Strongest narrative thread; relationship with Crawfurd Hospital unverified; no HSA/IMDA healthcare accreditation claimed. |
| Cloud computing solutions | Services page "Cloud Computing" area (AWS, Azure, GCP, Alibaba, DO, K8s, Docker, ELK); cloud partner logos | ⚠ Website claim. No cloud-partner certifications shown; logos consistent with stack list. |
| Industry analysis | Services page "Industry Analysis" area (scientific research, data analysis, competitor analysis, strategic planning) | ⚠ Website claim. Adjacent to, but not evidenced by, any other page. |
| Ongoing delivery roles | All three product pages: "Our role — Development, UX/UI Design, QA, DevOps, Support" | ⚠ Website claim. Implies end-to-end agency delivery model — the §6 design's core assumption. |

### 2.7 Reading the Services Honestly

The services picture is coherent, modern and *internally consistent* — the same Flutter/React/PostgreSQL/TypeScript stack appears across the services page, all three product pages, and the cloud logos; the healthcare and charity threads reinforce each other; the "strategists, designers and coders" identity is repeated verbatim across the home meta description, the about page and LinkedIn. That consistency is itself a signal of a firm that has settled on a clear pitch. What the public record cannot show is the *delivery reality* behind the pitch: whether the case studies describe shipped production systems or demo/spec projects; whether "Data Security" means consultative audits or delivered products; how many engineers sit behind the 7 registered officers; and whether the "24/7, in any part of the world" coverage claim (about page — quoted in §3.2) describes an actual distributed team or aspirational availability. Every one of those is a **diligence question for the buyer**, and §6 designs the delivery model around exactly those questions.

### 2.8 Adoption — The Honest Caveat (flagged)

The brief asked for adoption evidence to be verified and flagged, so here it is, flagged honestly — following the same discipline as [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) §1.5, whose "adoption" section exists for exactly this reason:

- **What is verified:** the *existence* of the website's three product case studies (privacy/security app, elderly/charity apps, EHR system — each with Description/Tech stack/Problem/Result sections); the *existence* of the two non-cloud partner logos and the real organisations behind them (Crawfurd Hospital, Hug 2 Hearts); the 336-follower LinkedIn count (search snippet); the MyCareersFuture company-page presence; the "2022 Team Building @ Bintan" caption. All of these verify that the firm *publishes* these artefacts — none of them verify that any named user, clinic, hospital or charity actually operates any Bidot-built system.
- **What is not verified:** **no named client, no named deployment, no dated project, no user count, no download count, no store listing, no third-party review, no press mention** exists anywhere in the public record for any Bidot Tech deliverable. The about-page counter strip renders "0 Contributors / 0 Million Users / 0 Clients" (§3.4) — a template artifact, but nonetheless the only *numbers* the site itself publishes. There is no independent, audited answer to "who actually uses what Bidot built?"
- **What is reported (flagged):** the partner logos imply *some* relationship with a real hospital and a real charity in the firm's claimed verticals — the strongest adoption-adjacent signal the public record offers — but the relationship type (client, pro-bono, referral, logo placement) is undisclosed, so even that signal cannot be upgraded from "plausible" to "verified adoption."
- **The diligence consequence:** treat every adoption-adjacent claim in this guide as **upper-bound marketing unless reference calls confirm it** — the identical conclusion the SaaS guide reaches for its own subjects. The §6.7 vendor scorecard makes "named production references" a scored, gated item precisely because public adoption data for a firm like Bidot Tech is effectively nil.

---

## 3. The Positioning

### 3.1 The Strategists-Designers-Coders Claim (Verified as a Website Claim)

The identity claim appears in two places, verbatim:

- **Home page meta description:** "Bidot Tech is a full-stack custom software and application development services provider. **We are strategists, designers and coders!** Get in touch with us!" (extracted from the live HTML, August 2026).
- **About page headline:** "**We are strategists, designers and coders!**" — followed by the full self-description paragraph (quoted in §3.2).

The three-noun trinity is a deliberate positioning device: **strategists** (upstream: requirements, product thinking, the "Industry Analysis" service line), **designers** (UX/UI, brand — the "Product Design" area), and **coders** (the delivery stack). It is the boutique-agency answer to the "full-stack" promise of §2.1 — one team covering the whole arc — and it matches the product pages' "Our role: Development, UX/UI Design, QA, DevOps, Support" line. As with every claim in §2, it is **verified as the company's own claim**, and the honest caveat applies: a positioning statement is not a capability certificate. (Note also the site's own vocabulary slip in the charity case study — "the responsibility of the **mobile app development agency**" — which confirms the firm thinks of itself as an agency.)

### 3.2 The About Page — the IT Start-up (Verified as Website Claims)

The about page (bidot.sg/about.html) contains the firm's fullest self-description, quoted verbatim:

> "Led by outstanding entrepreneurs and driven by a vision to shape a better world with technologies, **Bidot Tech is an Information Technology (IT) start-up** with solid expertise and passion in technology, creativity and innovation. With our wide and extensive network, our clients' requirements are well covered **24/7, in any part of the world**. We take pride in our excellent customer service skills, long-term relationships with the stakeholders, as well as our first-rate work. We are the experts in our field with a plethora of experiences. While looking at the big picture, we pay meticulous attention to every single detail."

Plus a mission and a vision:

> **Mission:** "Our outstanding and dedicated team is eager to transform our customers' ideas into reality. Combining our technical expertise, practical experience and passion in technology, creativity and innovation, we strive to meet the growing needs of businesses by developing **state-of-the-art and cost-effective IT solutions**. We place strong emphasis in our service and quality. Every single project is unique to us..."
>
> **Vision:** "We set our sights on **becoming a leading technology organisation** best known for its quality technical skills, innovative solutions and excellent customer service... We aspire to shape a better world by improving every aspect of our daily lives with first-rate technologies, whether through in-house or partners' projects."

Honest notes on the specific claims here:

- **"IT start-up"** — a *self-classification*, and a telling one: the firm positions itself as young, hungry and founder-led (consistent with the 2019 incorporation and the "led by outstanding entrepreneurs" line) rather than as an established consultancy. Notably, the "start-up" framing is about *Bidot itself*, not its clients — it does not claim to be a venture-backed scale-up (no funding data exists anywhere in the public record), and nothing on the site claims incubator/accelerator pedigree.
- **"24/7, in any part of the world"** — the strongest flag on the page. No evidence of a global footprint or follow-the-sun team exists publicly; the registered office is one suite in Singapore. This reads as an aspirational availability claim; it is unverifiable as written and flagged ⚠.
- **"long-term relationships with the stakeholders" / "first-rate work" / "plethora of experiences"** — pure marketing register; no named stakeholders, no case-study dates, no client testimonials are published on the site.

### 3.3 The LinkedIn Positioning (flagged as claims)

The LinkedIn company page (sg.linkedin.com/company/bidot-tech) could not be extracted directly this pass — the page is behind Cloudflare's bot challenge — so the LinkedIn content here comes from a **search-engine snippet of the page** (Brave, August 2026), which is itself flagged:

> "Bidot Tech | **336 followers** on LinkedIn. Bidot Tech is **a leading full-stack custom software and application development services provider**. | Led by outstanding entrepreneurs and driven by a vision to shape a better world with technologies, Bidot..."

Two honest readings. First, the LinkedIn tagline is the **same "full-stack custom software and application development services provider" + "led by outstanding entrepreneurs" language** as the website's meta description and about page — the LinkedIn page mirrors the website positioning, which is corroborating (the two self-published channels agree verbatim) but is still *self-published* evidence. Second, **336 followers** is a small-but-real social footprint (a search-snippet count, point-in-time, unverified directly) — consistent with a young boutique rather than a brand-name firm, and consistent with the thin-public-footprint framing. The research brief's characterisation of the LinkedIn positioning ("an IT start-up... led by outstanding entrepreneurs") is confirmed as *the company's own claim* — verified as a claim, flagged as unverifiable substance. No employee counts, no post history, no client endorsements were obtainable in this pass.

### 3.4 The Counter-Stats Artifact — "40 / 0 / 0 / 0" (flagged template artifact)

The about page includes a four-stat counter strip: **"40 — Company Founded / 0 — Contributors / 0 — Million Users / 0 — Clients"** (the "40" mislabelled beside "Company Founded"). These are almost certainly **template animation counters whose values never got populated** (the site's JS loads header/footer from separate files and the counters are a common Bootstrap-era template component) — a static extraction sees the placeholder zeros. The honest treatment: this artifact is **not usable as evidence of anything** (it certainly does *not* mean zero clients — that would contradict the site's own case studies), and it *does* illustrate the site's amateur-assembly quality. It is recorded here precisely so no reader mistakes template scaffolding for disclosed metrics, and so the guide can say explicitly: **the public record contains no disclosed metrics at all** — no client counts, no user counts, no project counts.

### 3.5 The Career Page — "We're Not Just Colleagues. We're Family." (Verified as Website Claims)

The career page (bidot.sg/career.html) is the most human artefact on the site:

- **Culture pitch:** "Life at Bidot — We're not just colleagues. We're family. Bidot is not just a workplace – it's a lifestyle..."
- **Core values:** Teamwork ("Be partners and collaborate and not just run errands"), Honesty ("Build an open and honest relationship based on transparency and feedback"), Proactivity, Courage.
- **Benefits:** flexible working arrangements, medical/dental/hospitalisation insurance, pantry ("You will never go hungry working at Bidot"), "Work hard, Play harder" with team buildings, monthly birthday celebrations and "annual DnD", referral bonuses, knowledge sharing.
- **Team evidence:** a photo caption **"2022 Team Building @ Bintan"** — the only direct evidence anywhere in the public record that the firm has an in-person team of more than one. Team size is unstated.
- **Open vacancies:** an empty "Open Vacancies" section (JS-driven; no live roles visible in this pass).

Honest notes: the page is a recruiting-pitch page and reads like a template-flavoured culture page, but it is the one place the firm reveals *something* about being an employer (insurance, a pantry, an offsite in Bintan in 2022). It tells a prospective client nothing about delivery capacity — and it is flagged accordingly. The Bintan caption is "verified as website content" only.

### 3.6 The Positioning Table

| Positioning element | Claim (verbatim) | Where | Status |
|---|---|---|---|
| Core identity | "We are strategists, designers and coders!" | about.html headline; home meta description | **Verified as website claim** ⚠ |
| Service positioning | "a full-stack custom software and application development services provider" | home meta description; LinkedIn tagline | **Verified as website/LinkedIn claim** ⚠ |
| Company stage | "an Information Technology (IT) start-up" | about.html; LinkedIn | **Verified as website/LinkedIn claim** ⚠ |
| Leadership | "Led by outstanding entrepreneurs" | about.html; LinkedIn | **Verified as website/LinkedIn claim** ⚠ (no founder names public) |
| Reach | "our clients' requirements are well covered 24/7, in any part of the world" | about.html | ⚠ Unverifiable as written |
| Mission | "developing state-of-the-art and cost-effective IT solutions" | about.html | ⚠ Website claim |
| Vision | "becoming a leading technology organisation" | about.html | ⚠ Website claim |
| Social footprint | 336 LinkedIn followers (search snippet, Aug 2026) | LinkedIn | ⚠ Third-party count, point-in-time |
| Metrics | No disclosed client/user/project counts (template counters show 0/0/0) | about.html | ⚠ Template artifact; not usable as evidence |
| Culture | "We're not just colleagues. We're family."; values; benefits; "2022 Team Building @ Bintan" | career.html | Verified as website content; team size unstated |

### 3.7 Positioning vs Footprint — the Honest Gap

Bidot Tech's positioning is **ambitious and coherent** (full-stack boutique, strategists-designers-coders, start-up energy, healthcare + charity verticals); its public footprint is **thin and self-published** (registry record + website + a 336-follower LinkedIn page, zero press, zero disclosed metrics, zero named clients). The gap between the two is not a scandal — it is the *normal* profile of a young EPC services firm that has chosen privacy over publicity — but it is the single most important thing a prospective client must manage, and §6's delivery model is designed around it: contract structure, IP controls, reference verification and staged money all exist precisely to close the gap between what the boutique claims and what the buyer can verify.

---

## 4. The Singapore Presence

### 4.1 The Cecil Street PLUS Office (Verified)

Bidot Tech's registered office — and, per the website's contact page, its visitable office — is **20 Cecil Street, #23-03/04, PLUS, Singapore 049705**:

- **Registry (verified):** 20 Cecil Street, #23-03, PLUS, Singapore 049705 — block 20, level 23, unit 03, building "PLUS", postal code 049705 (opengovsg.com; companies.sg).
- **Website (verified as website content):** the contact page prints "20 Cecil Street, #23-03/04 PLUS, Singapore 049705" with a Google-Maps link, plus phone +65 6223 0308, email info@bidot.sg, and office hours "Mon-Fri from 9am to 6pm". The site's own map pin and the "Visit us — Come say hello at our office" copy treat it as a real working office, not a maildrop.
- **The #23-03 vs #23-03/04 nuance** is flagged in §1.4: the registry shows unit 03; the sister entity Bidot Investment Holdings sits at unit 04; the website presents the pair.

The PLUS building is a real, older-generation commercial office block on Cecil Street in Singapore's central business district — a standard CBD-fringe address occupied by many small firms and serviced offices (§4.4). Nothing about the address is exotic; it is exactly the kind of genuine commercial tenancy a seven-year-old boutique dev shop would hold.

### 4.2 Cecil Street as a Commercial Address (Verified Context)

Cecil Street runs through the historic commercial heart of Singapore — the Tanjong Pagar/Raffles Place district that has been the island's trading and financial address since the 19th century (Cecil Street itself was named in the colonial era and is lined with offices, banks and the former Hong Lim Complex area). The PLUS building at 20 Cecil Street is a 1980s-vintage commercial tower in that strip — mid-sized floor plates, a mixed tenant base of small companies, professional firms and serviced-office operators, and a location two short blocks from the Singapore River's financial district. For a software-services firm the practical meaning of the address is: **downtown, client-accessible, near the MRT (Tanjong Pagar and Raffles Place stations are both within walking distance), and surrounded by the exact buyer population the firm's pitch targets** — banks, insurers, trading firms and the SME corporate-services economy of the CBD. It is a credible base of operations for a firm that sells "turn your ideas into reality" to founders and product owners. (Verified as geographic context; the *quality* of the premises — serviced office vs direct lease, and whether #23-03 and #23-04 are one tenancy or two — is not public.)

### 4.3 The SG Table

| Aspect | Detail | Status |
|---|---|---|
| Country of incorporation | Singapore — the only jurisdiction in the public record (no foreign entities, branches or offices claimed anywhere on the site) | Verified (registry) |
| Registered office | 20 Cecil Street, #23-03, PLUS, Singapore 049705 | Verified (registry) |
| Website office | "20 Cecil Street, #23-03/04 PLUS, Singapore 049705" | Verified as website claim; #23-03/04 nuance flagged |
| Contact | +65 6223 0308 (Mon–Fri 9am–6pm); info@bidot.sg | Verified as website claim |
| Jurisdiction machinery | ACRA-registered EPC; SSIC 62011; annual-return and filing obligations current | Verified (registry) |
| Trademark | IPOS trademark-owner record exists (per companies.sg) | Flagged — existence only |
| Employment evidence | "2022 Team Building @ Bintan" caption; benefits page (insurance, pantry) | Verified as website content ⚠ |
| Foreign footprint | None claimed; the "24/7, in any part of the world" line is aspirational, not evidenced | ⚠ Flagged |
| Government-adjacent presence | Appears on MyCareersFuture (gov.sg jobs portal) company directory with UEN 201915071R | Verified (URL exists; content is portal-rendered) |

### 4.4 The Same-Address Observation — a Serviced-Office Pattern (flagged)

The registry's "entities in the same location" lists for 20 Cecil Street / postal 049705 show a **dense multi-tenant commercial building**: Conduit Group VCC, Bellissimo Hospitality, several 2026-vintage incorporations, shipping and trading firms — the usual mix of a CBD office block with serviced-office floors. Within that mix, the **Bidot cluster** of §1.8 (Bidot Tech at #23-03, Bidot Investment Holdings at #23-04, former Bidot Pte Ltd at #23-03) is the notable same-address pattern — three "Bidot" entities in two adjacent suites on one floor. The Bond guides' discipline applies verbatim: **entities sharing an address is a verified registry observation; what it means about ownership, common management or shared premises is NOT public and is NOT asserted here.** For a counterparty, the practical upshot remains §1.2's rule: key on the UEN.

---

## 5. The Market Context — the SG Custom-Dev Services Landscape

### 5.1 The Ecosystem Bidot Tech Incorporated Into (Verified Context)

Bidot Tech incorporated in May 2019 into the software ecosystem mapped in depth in [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md). The relevant ecosystem facts, cross-referenced from that guide's verified ledger:

- **A ~6M-population city-state whose domestic market cannot carry a software company alone** — the ecosystem's founding logic (§1.1 of the SaaS guide). For a *services* firm this cuts differently than for a SaaS vendor: Bidot Tech doesn't need to productise for export, it needs to *sell engagements* — and Singapore's concentrated CBD buyer base (banks, insurers, corporates, funded startups) is a genuinely serviceable home market.
- **The government machinery** — IMDA (accreditation, PSG grant directories), EDB (HQ attraction), EnterpriseSG (grants, SME support), SGInnovate (deep tech), MAS (fintech sandbox) — shapes *demand* for custom software: digitalisation grants push SMEs to buy systems (often from exactly this kind of shop), while regulated buyers buy compliant builds (§5.3).
- **The talent pool and cost base** — a highly educated, English-first engineering market with **high costs**: Singapore engineering salaries are among Asia's highest, which squeezes boutique margins and pushes price-sensitive work offshore (a structural fact of §5.2's market tiers).
- **The company set** — the SaaS guide's homegrown vendors (PatSnap, Trax, Tookitaki, Silent Eight, Horangi, JustLogin, Sleek, Glints) are the *product* layer of the same ecosystem; custom-dev agencies are the *services* layer they and their buyers also consume.

### 5.2 The Tiers of the Custom-Dev Services Market

The Singapore custom-software-development *services* market — the market Bidot Tech's "full-stack custom software and application development services provider" positioning targets — is best understood as five tiers (analytical framing, anchored on verified ecosystem facts and standard industry structure; not a sourced market-share claim):

| Tier | Who | Typical buyer | Why it matters for Bidot Tech |
|---|---|---|---|
| **T1 — Global IT services / SIs** | Accenture, TCS, Infosys, Cognizant, DXC; also the big four's tech arms | MNCs, banks, government | The reference point for "enterprise-grade"; buyers who want scale, ISO/SOC2 accreditations, global SLAs. Bidot Tech is not competing here. |
| **T2 — SG/regional consultancies & SIs** | NCS (Singtel), Temus (KPMG/DP), UST, ABeam, Thoughtworks Singapore | Government-linked projects, large corporates | IMDA-accredited, GITP-experienced, security-certified. The credible mid-market. |
| **T3 — Boutique product-development agencies** | The custom-dev shop tier: full-stack studios doing design+build+QA+DevOps for startups and SMEs | Founders, product owners, SMEs, early-stage startups | **Bidot Tech's tier** — its entire public positioning (strategists-designers-coders, "turn ideas into reality", Flutter/React full-stack) is this tier's pitch. |
| **T4 — Freelancers & offshore platforms** | Upwork/Toptal gig workers, offshore nearshore shops (Vietnam, Malaysia, India) | Cost-sensitive SMEs, prototype-stage founders | The price floor; competes with T3 on rate, loses on accountability/oversight. |
| **T5 — SaaS products** | The [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) company set | Buyers who want off-the-shelf over custom | The *substitute* for custom dev — every "build vs buy" decision a founder faces. |

The tier's economics: T3 shops live on **engagement flow** (project wins), typically sell fixed-scope or sprint-based contracts, carry small teams (the 7-officer registry count is consistent with a small shop, though headcount is unverified), and differentiate on speed, design quality and vertical focus. Bidot Tech's public artefacts — the healthcare/charity verticals, the design-led positioning, the "Development, UX/UI Design, QA, DevOps, Support" role line — are textbook T3 artefacts.

### 5.3 Government as Buyer and Standards-Setter (Verified Context)

Three verified ecosystem mechanisms (cross-referenced from [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) §7 and the SG public-sector-tech guides) shape the custom-dev market Bidot Tech sells into:

1. **GovTech and the GITP (Government Integrated Test Practice)/geBiz era** — the government is Singapore's largest software buyer, and its procurement regime (IMDA-accredited vendors, security standards, tender requirements) sets the compliance bar that *private* buyers increasingly copy. None of this appears in Bidot Tech's public record — no IMDA accreditation, no government tender mentions — which is consistent with a T3 shop that sells to founders and SMEs, not to the state.
2. **IMDA accreditation and certification** — IMDA's accreditation schemes (and the PSG's grant-directory listings) function as *trust signals*: SMEs get grants only for solutions from approved providers, so PSG-listed vendors enjoy subsidised demand. The public record shows **no IMDA accreditation or PSG listing for Bidot Tech** (flagged — absence of evidence; the directories were not exhaustively searched, but no listing surfaced).
3. **The SME digitalisation push** — EnterpriseSG/IMDA's "SMEs Go Digital" and PSG programmes hand SMEs subsidies to buy digital solutions, and a large share of that spend lands with exactly the T3 shop tier. The same programmes also push buyers toward *pre-approved* solutions, which structurally favours listed vendors and productised offerings over one-off custom builds — a mild headwind for pure custom shops.

### 5.4 Where Bidot Tech Sits in the Landscape

Placing Bidot Tech in the §5.2 map, using only what the public record supports:

- **Tier:** T3 boutique product-development agency — the site's own vocabulary ("mobile app development agency", "full-stack development cycle from A to Z", "strategists, designers and coders") is unambiguous.
- **Vertical focus (claimed):** healthcare IT and charity/elderly — the EHR case study, Healthcare IT service line, Crawfurd Hospital and Hug 2 Hearts partner logos form the firm's only consistent vertical narrative. Health-adjacent custom dev in Singapore is a real niche (private hospitals, clinics, telehealth, wellness startups), but it is also a **regulated-data** niche (PDPA; healthcare data; potential HSA/IMDA-adjacent rules) where certification and audit history matter — none of which the public record evidences.
- **Competitive posture (inferred, flagged):** younger (2019), smaller-footprint, and design-led rather than certification-led — the opposite end of the spectrum from T1/T2. Its plausible buyers are founders, product owners and SMEs (its own home-page copy: "We help founders and product owners to turn ideas into reality") rather than regulated enterprises.
- **The honest summary:** Bidot Tech is one of many small SG full-stack shops selling the same "A to Z" promise into the same founder/SME buyer pool, differentiated (per its own claims) by healthcare/charity flavour and a design-led identity — and **undifferentiated in every dimension the public record cannot verify** (delivery record, team depth, financial substance).

### 5.5 The Context Table

| Context element | Description | Source / status |
|---|---|---|
| Home-market logic | ~6M population; services firms sell engagements to the CBD buyer pool rather than exporting product | Cross-ref [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) §1 (verified context) |
| Buyer pool | Founders, product owners, SMEs — per Bidot's own copy | Website claims ⚠ |
| Market tier | T3 boutique product-development agency | Analytical placement based on website claims |
| T1/T2 competitors | Global SIs and SG consultancies with accreditations and scale | Context (industry structure) |
| T4 competitors | Freelancers/offshore platforms competing on rate | Context (industry structure) |
| T5 substitute | SaaS products — the build-vs-buy alternative | Cross-ref SaaS guide (heavy) |
| Government machinery | IMDA, EDB, EnterpriseSG, SGInnovate, MAS shape demand and standards | Cross-ref SaaS guide §7 (verified context) |
| Bidot's state-relations | No IMDA accreditation, no PSG listing, no tender presence surfaced | ⚠ Absence of evidence, flagged |
| Vertical niche (claimed) | Healthcare IT and charity/elderly apps | Website claims ⚠ |
| Regulated-data exposure | Healthcare builds imply PDPA/health-data obligations; no certifications claimed | ⚠ Flagged |
| Ecosystem maturity | Gen-3/AI-native era; custom-dev shops are the services layer under the SaaS product layer | Cross-ref SaaS guide §1.4 |

### 5.6 The Market Bottom Line

The SG custom-dev services market is **deep, fragmented and trust-gated**: a seven-year-old boutique at 20 Cecil Street competes for the same founder/SME engagements as dozens of peers, and the buyer's decision rarely turns on capability claims (everyone claims full-stack) — it turns on **references, delivery discipline and contract terms**. That is precisely why the worked example in §6 is built as a *delivery model*: in this market, the model IS the product the buyer is really purchasing. The market context also explains Bidot Tech's own structure: a privacy-preserving EPC, a thin website, partner logos instead of client lists, and a LinkedIn page instead of a press profile — all rational choices for a small shop that wins work through relationships, and all exactly the things a disciplined buyer must see through.

## 6. The Worked Example — a Custom-Dev Engagement/Delivery Model

### 6.1 The Scenario — CareLane and the Boutique Builder

**The client (fictional):** "CareLane," a Singapore healthcare-services startup (fictional — the same familiar-context scenario style used in the Bond guides' worked examples, and deliberately set in Bidot Tech's *claimed* vertical so the design doubles as a stress-test of the boutique's own pitch). CareLane runs two physiotherapy clinics, has just raised a seed round, and wants to digitise: a patient-facing mobile app (booking, records access, tele-rehab), a practitioner portal (scheduling, notes, care plans), and an EHR-lite backend with PDPA-grade data handling. Its CTO is a former bank colleague of the architect writing this series — which is how the architect gets pulled in to design the *procurement and delivery model* rather than the software itself.

**The familiar context:** the architect has read [../management/vendor_management_guide.md](../management/vendor_management_guide.md) and [../management/business_case_development_guide.md](../management/business_case_development_guide.md), and has just finished [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) and this Bidot Tech guide — so the design brief is explicit: **CareLane will commission a full-stack custom build from a Singapore boutique dev shop (the T3 tier of §5.2), and the Bidot Tech profile is the stand-in for the entire class of vendor CareLane is about to hire.** The Bidot public footprint — registry-verified entity, coherent full-stack and healthcare claims, thin everything else — is the *typical* T3 vendor dossier: good registry hygiene, plausible marketing, zero verifiable delivery evidence. The engagement model must therefore be built to succeed with such a vendor *and* to survive one whose claims overstate reality.

**The vendor (fictional but Bidot-flavoured):** "Harbourline Dev Pte. Ltd." — a 2019-vintage Singapore EPC, SSIC 62011, Cecil-Street-adjacent office, 7 officers, S$7.4M paid-up capital claimed by one aggregator, a website promising "full-stack development from A to Z", Flutter/React/PostgreSQL stack, healthcare and charity case studies with no named clients, no certifications, no press. Every diligence move below is calibrated to exactly this dossier.

### 6.2 The Model Design — the Five-Phase Delivery Model

The design is a **five-phase engagement model**: Discovery → Design → Build (sprints) → Acceptance & Handover → Support & Evolve. Three structural rules govern all five phases:

1. **Money follows evidence.** Budget is committed phase-by-phase; every release gate releases the next tranche. No vendor subsidy up front beyond a small mobilisation fee.
2. **IP is bought, not borrowed.** The client owns all code, designs, and data from day one, with source escrow and exit clauses as first-class contract terms — the standard answer to the "who owns what after a boutique folds" question.
3. **The vendor's footprint is verified, not assumed.** The §1 registry checks are repeated by the buyer's own counsel (BizFile+ extract, UEN, status, filings), and every §2–§3 claim the vendor makes is converted into a contract obligation or a reference-check question.

**Phase 1 — Discovery & Vendor Diligence (weeks 1–4).** Before any build: (a) **registry verification** — pull the ACRA BizFile+ record (UEN, incorporation, status, officers, charges) rather than trusting aggregators; (b) **the claims-to-questions converter** — every website claim becomes a diligence question: the "healthcare IT" case study → "name one live EHR/healthcare deployment and its operator, and let us call them"; "24/7 coverage" → "show the follow-the-sun roster"; "full-stack from A to Z" → "who are your actual senior engineers and what did they ship?"; (c) **reference calls** — two client references minimum, with the seller required to disclose the referrer's name, role and project (the refusal to name references is itself a finding); (d) **financials ask** — the EPC files no public accounts, so the buyer requests the latest management accounts and an auditor/accountant's letter under NDA (the S$7.4M paid-up-capital claim gets its composition question: cash or assets-in-kind?); (e) **the security posture interview** — PDPA obligations for health data mean the vendor's data-handling, hosting (SG data residency — [singapore_data_centres_guide.md](singapore_data_centres_guide.md)), and any certifications are probed; (f) **a paid mini-engagement** — a small fixed-price spike (e.g. a UI prototype of one screen flow) as the cheapest possible real-work sample; the spike's quality and honesty are leading indicators of the whole engagement.

**Phase 2 — Design & Specification (weeks 5–8).** The vendor's "strategists and designers" earn their title here: (a) **the product spec** — user stories, acceptance criteria, and the data model agreed *before* build (this is the buyer's best defence against scope creep and the boutique's best defence against unbounded expectations); (b) **the architecture decision record (ADR)** — stack selection *with reasons*, mapped against the vendor's claimed stack but owned by the client: Flutter for the patient app and React/TypeScript for the portal are fine *if* the ADR says why and who maintains them after handover ([micro_frontend_architecture_guide.md](micro_frontend_architecture_guide.md) informs the portal's frontend architecture); (c) **the compliance review** — PDPA impact assessment for health data, data-residency decision (SG-region hosting), and the logging/audit requirements; (d) **the fixed-scope baseline** — a fixed-price contract for the agreed scope, with a change-control process (variation orders priced openly) rather than an open-ended T&M burn. If the vendor resists a fixed baseline for a scope this defined, that is a finding.

**Phase 3 — Build in Sprints (months 3–7).** Delivery discipline: (a) **two-week sprints** with a sprint demo every fortnight to a named client product owner (the founder/CTO, not a proxy); (b) **definition of done** — code merged, tested, deployed to the shared staging environment, and demoed; (c) **the buyer's own QA** — the client runs independent smoke tests against staging each sprint rather than trusting the vendor's "QA" line-item (which the product pages list as a vendor role — a conflict of interest the model structurally resolves); (d) **access and repositories** — client-owned GitHub org, client-visible CI, and environments in a client-controlled cloud account (the vendor builds *in* the client's cloud, not their own — the single most effective anti-lock-in control); (e) **sprint-based payments** against the fixed baseline with the release-gate rule of rule 1; (f) **the health metric** — a simple sprint scorecard (scope delivered, defects found by client QA, demo attendance) that the architect reviews monthly; three consecutive red sprints trigger the escalation clause.

**Phase 4 — Acceptance & Handover (month 8).** The exit is designed from day one: (a) **the acceptance test suite** — the Phase-2 acceptance criteria executed by the client against staging, then production; (b) **the handover pack** — full source in the client's repo, environment/build documentation, runbooks, the ADRs, and the vendor's own documentation as *deliverables* (payment of the final tranche is conditional on the pack, not on the demo); (c) **source escrow** triggered, with the client having continuous access anyway by construction (rule: never let the vendor hold the only copy of the source or the only keys); (d) **the transition run** — the vendor runs support alongside the client's appointed maintainer for the first month.

**Phase 5 — Support & Evolve (months 9+).** A post-handover support contract (fixed monthly retainer or T&M with a cap) for defect fixes and small changes, with: (a) **SLAs** — response/resolution times and a named escalation path; (b) **the knowledge-transfer obligation** — the maintainer's questions get answers, in writing, within agreed SLAs; (c) **a review gate at month 12** — the client decides whether to extend, shrink, or take the work fully in-house. The model's bet: by month 12 CareLane's own engineer can maintain the system, and the boutique's role is optional rather than existential.

### 6.3 The Delivery Model at a Glance

```
CareLane (client) ─── fixed-scope contract, phased money ───▶ Harbourline Dev (boutique, T3)
     │                                                            │
     ├─ ACRA BizFile+ check, references, financials ask           ├─ Discovery spike (paid, small)
     ├─ ADR + PDPA review (buyer-owned decisions)                 ├─ Design → fixed baseline
     ├─ Client-owned GitHub/cloud; client QA each sprint          ├─ 2-week sprints, demos, scorecard
     ├─ Acceptance suite + handover pack (final tranche gate)     ├─ Escrow + continuous source access
     └─ Month-12 review gate ◀── support SLA (months 9+) ────────┘

Money gates:   mobilisation (10%) → spike → design (15%) → per-sprint (60%)
               → acceptance+handover (15%) → support retainer (ongoing)
Control gates: registry+references → fixed baseline → 3-red-sprint escalation
               → acceptance suite → handover pack → month-12 review
```

### 6.4 The Decision Matrix

| Decision | Option chosen | Why | Rejected alternative |
|---|---|---|---|
| Vendor tier | T3 boutique (Bidot-class), not T1/T2 SI | Right-sized for a seed-stage build; the tier's speed/design strength is what CareLane needs; cost matches the round | T1/T2 SI (over-specified, billable-hour heavy); freelancers/offshore (no accountability for health data) |
| Contract shape | Fixed-scope baseline + phased money + change control | Converts the boutique's "A to Z" promise into priced, testable deliverables; protects the seed budget | Open T&M (unbounded burn with a thin-footprint vendor); pure fixed-bid (vendor prices in risk, quality suffers) |
| Source & environment ownership | Client-owned repos, client-owned cloud from day one | The definitive anti-lock-in and anti-thin-footprint control; makes escrow almost moot | Vendor-hosted everything (classic boutique lock-in) |
| QA | Independent client QA per sprint | The vendor's "QA" role is a conflict; the buyer's smoke tests are the honest scorecard | Trusting the vendor's QA line-item (product-page claim) |
| Acceptance | Executable acceptance suite + handover pack as payment gates | Payment follows evidence (rule 1); the pack is the durable asset | Demo-day sign-off (the classic "demo works, delivery doesn't") |
| IP & data | Client owns IP from day one; PDPA/data-residency review pre-build | Health data demands it; buys freedom for future vendors | Leave IP terms vague (the default in small-shop boilerplate) |
| Post-handover | Fixed support SLA + month-12 review gate | Keeps the relationship optional, not existential | Indefinite dependency on a single boutique |

### 6.5 The Lessons

1. **For a thin-footprint boutique, the contract IS the capability statement.** Bidot-class vendors publish claims, not evidence; the delivery model converts every claim into a clause, a gate, or a reference question. A vendor that accepts this model confidently is signalling real delivery capability; a vendor that resists the fixed baseline, client-owned repos, or named references is answering the diligence question for you.
2. **Registry verification is the floor, not the ceiling.** The verified facts of §1 (UEN, EPC, SSIC 62011, live status, the DONG ER TECHNOLOGY former name, the S$7.4M capital) tell the buyer *what the entity is*, never *what it can deliver*. BizFile+ and the aggregators are phase-1 steps, not the diligence itself.
3. **The UEN is the entity; the name is a hazard.** With the Bidot cluster (§1.8) and generic boutique names generally, every contract, invoice and correspondence keys on the UEN — the same discipline the Bond guides hammer for their subjects, and the same reason banks key on LEIs and UENs.
4. **Own the assets and the environment, and exit risk collapses.** Client-owned repos, client-owned cloud, and continuous source access make the difference between "boutique folds mid-project" (recoverable in days) and "boutique holds the keys" (a hostage situation). This is the single highest-leverage design decision in the model.
5. **Health-adjacent claims attract regulated data.** Bidot's EHR/healthcare narrative (§2.4–§2.5) is exactly the kind of claim that sounds impressive and quietly implies PDPA-grade data handling; the model's Phase-2 compliance review and SG data-residency choice ([singapore_data_centres_guide.md](singapore_data_centres_guide.md)) are non-negotiable for any buyer in that space, whatever the vendor's website says.
6. **The boutique tier is a rational choice — managed honestly.** Nothing in this design says "don't hire the boutique"; it says "hire the boutique the way you would hire any vendor whose public record is a website": verify, baseline, gate, and keep the exit open. The model is the buyer-side mirror of the T3 market reality of §5.6.

### 6.6 The Risk Register

| Risk | Likelihood | Mitigation |
|---|---|---|
| Capability overstatement (claims ≠ delivery) | Medium-High for thin-footprint vendors | Discovery spike, named references, fixed baseline, client QA per sprint |
| Vendor insolvency / closure mid-build | Low-Medium (no public financials) | Phased money, client-owned assets, continuous source access, escrow |
| IP ambiguity (who owns the code) | Medium (small-shop boilerplate) | Explicit IP assignment from day one; handover pack as payment gate |
| Lock-in (vendor-hosted infra, opaque ops) | Medium | Client-owned cloud/repos; ADRs and runbooks as deliverables |
| Data-protection breach (health data) | Medium | PDPA review pre-build, SG data residency, logging/audit requirements in spec |
| Scope creep / unbounded T&M | High in custom-dev generally | Fixed baseline + open change control (variation orders priced visibly) |
| Key-person dependency (one engineer holds the system) | Medium | Documentation obligations, knowledge-transfer SLA, month-12 in-house review |
| Escalation failure (three red sprints ignored) | Low if enforced | Contractual escalation clause + architect's monthly scorecard review |

### 6.7 The Vendor Scorecard and the Phase Timeline

**The scorecard.** Every claim from the Bidot-class dossier converts to a scored diligence item (the claims-to-questions converter of §6.2 Phase 1, tabulated). Scores: **PASS** (evidence produced), **PARTIAL** (evidence thin), **FAIL** (refusal/absence), **N/A** (not applicable). The buyer's rule: two FAILs on *gated* items (2, 4, 6, 8) = no engagement.

| # | Diligence item | Bidot-dossier claim it tests | Evidence required | Gate? |
|---|---|---|---|---|
| 1 | Registry verification (BizFile+: UEN, status, officers, charges, filings) | Entity hygiene | Official extract, current | Yes |
| 2 | Named production references (2+, callable) | "Healthcare IT solution", "EHR System" case studies | Referrer name, role, project, permission | Yes |
| 3 | Financial substance (management accounts / accountant's letter under NDA) | S$7.4M paid-up capital (aggregator) | Balance sheet, composition of capital | No |
| 4 | Source & environment ownership terms in contract | "Full-stack development cycle from A to Z" | Client-owned repos/cloud clauses | Yes |
| 5 | IP assignment + escrow + exit clauses | Agency boilerplate risk | Signed terms | Yes |
| 6 | Compliance posture (PDPA, SG data residency, security interview) | "Data Security" service line | Documented controls; hosting plan | Yes |
| 7 | Delivery evidence (discovery spike quality; sprint scorecard) | "Our role: Development, UX/UI Design, QA, DevOps, Support" | Real work samples; demo record | No |
| 8 | Team substance (who are the actual engineers; capacity) | "Strategists, designers and coders" | Named engineers, CVs, capacity plan | Yes |
| 9 | Certifications / accreditations (ISO 27001, SOC 2, IMDA) | Implied by security claims | Certificates or explicit N/A | No |
| 10 | Name/UEN discipline (invoices, contracts keyed to UEN) | Bidot name cluster (§1.8) | Contract boilerplate review | No |

**The phase timeline.** The five phases of §6.2 mapped to months, gates and money:

| Phase | Window | Key activities | Gate to exit | Money |
|---|---|---|---|---|
| 1 — Discovery & diligence | Months 0–1 | BizFile+ check, scorecard items 1–3, 6, 8–9, discovery spike | Scorecard gated items PASS | Mobilisation ~10% |
| 2 — Design & specification | Months 1–2 | Product spec, ADRs, PDPA review, fixed baseline | Baseline signed | Design tranche ~15% |
| 3 — Build (sprints) | Months 2–6 | 2-week sprints, demos, client QA, sprint scorecard | 3-red-sprint rule; scorecard reviewed | Per-sprint ~60% total |
| 4 — Acceptance & handover | Month 7 | Acceptance suite, handover pack, escrow, transition run | Pack delivered + suite green | Final tranche ~15% |
| 5 — Support & evolve | Months 8+ | SLA support, knowledge transfer, month-12 review | Review gate at M12 | Retainer (capped) |

---

## 7. The Summary — Bidot Tech in One Page

Bidot Tech Pte. Ltd. (UEN **201915071R**) is a **Singapore-incorporated exempt private company limited by shares**, incorporated **10 May 2019** (UEN issued 29 May 2019), **live** and filing-current, registered at **20 Cecil Street, #23-03, PLUS, Singapore 049705** (the website prints "#23-03/04"), with registered activity **SSIC 62011 — development & marketing of software program & website and online marketing**, 7 registered officers, and a former registered name of **DONG ER TECHNOLOGY**. The registry record — the guide's verified backbone — is confirmed in full, along with two honest corrections to the research brief (the SSIC code is 62011, not the brief's paraphrase; the registry unit is #23-03, not the website's #23-03/04). The company's own website (bidot.sg) and LinkedIn page position it, verbatim, as: **"Bidot Tech is a full-stack custom software and application development services provider. We are strategists, designers and coders!"** — an **"Information Technology (IT) start-up" led by "outstanding entrepreneurs"** that helps "founders and product owners turn ideas into reality" through a "full-stack development cycle from A to Z." The claimed services are full-stack custom software and application development across product design, mobile apps (Flutter/React), programming (C#/.NET, Go, Python, TypeScript, MySQL/PostgreSQL) and cloud (AWS/Azure/GCP/Alibaba/DigitalOcean, Kubernetes, Docker, ELK); the claimed verticals are **data security, healthcare IT, mobile apps and secure data transfer**, illustrated by three self-published product case studies (a privacy/security mobile app, elderly/charity apps, and an EHR system), with partner logos pointing at two real organisations in those verticals (Crawfurd Hospital, a genuine Singapore private hospital, and Hug 2 Hearts, the brand of the real HUG Community Services charity) alongside cloud platforms. **Every one of those capability claims is website/LinkedIn-only**: no public financials exist (exempt private company), no press coverage was found, no named clients or verifiable projects exist, no team size is disclosed (the 7 officers is an officer count, not a headcount), no certifications are claimed, and the site's own metric counters render as empty template placeholders. The market context is rich: Bidot Tech sits in the **T3 boutique product-development-agency tier** of the deep, fragmented, trust-gated Singapore custom-dev services landscape — the services layer beneath the homegrown SaaS product layer of [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) — selling into the founder/SME buyer pool with no IMDA accreditation, no PSG listing and no government footprint in evidence. A small "Bidot" name cluster at the same address (Bidot Investment Holdings at #23-04, former Bidot Pte Ltd struck off) reinforces the §1.2 rule: **the UEN, not the name, is the entity.** The worked example — a seed-stage healthcare startup commissioning a full-stack build from a Bidot-class boutique — designs the buyer-side answer: a five-phase engagement model (diligence → spec → sprinted build → acceptance/handover → support) in which money follows evidence, the client owns the code, cloud and acceptance suite, and every marketing claim is converted into a contract obligation or a reference question.

**The final word — the boutique builder.** Read honestly, Bidot Tech is exactly what its registry record and its own words say it is: **the boutique builder** — a seven-year-old Singapore custom-software-development shop of the "strategists, designers and coders" school, full-stack in its claims, health-and-charity flavoured in its portfolio narrative, deliberately small and deliberately quiet (an exempt private company, a template-built website, a 336-follower LinkedIn page, zero press). It is not a scale-up, not a certified consultancy, not a brand with a public track record; it is the archetype of the T3 boutique that Singapore's founder/SME economy runs on — and the archetype is honest about itself: young, hungry, design-led, and *unprovable beyond its own claims*. For a founder or product owner, the correct posture is neither dismissal nor credulity: verify the registry facts (they check out), read the website as marketing (it is), convert the claims into contract terms and reference questions (the worked example shows how), and remember that in the trust-gated custom-dev market of §5, the delivery model — not the capability page — is the product actually being bought. Whether the boutique builder's next seven years produce named clients, certifications and a verifiable delivery record is the question only its future engagements — and its clients' reference calls — will answer.

---

## 8. Glossary

| Term | Definition |
|---|---|
| **Bidot Tech** | The subject of this guide: Bidot Tech Pte. Ltd. (UEN 201915071R), a Singapore-incorporated exempt private company (inc. 10 May 2019; 20 Cecil Street #23-03, PLUS, Singapore 049705) positioning itself as a full-stack custom software and application development services provider ("strategists, designers and coders"); formerly registered as DONG ER TECHNOLOGY |
| **UEN** | Unique Entity Number — Singapore's national identifier for business entities, issued by ACRA (e.g., 201915071R; the leading digits encode the registration year, the trailing letter is a checksum) |
| **ACRA** | Accounting and Corporate Regulatory Authority — Singapore's corporate regulator and registry; issues UENs, registers companies, and administers the Companies Act; the authoritative source behind the aggregators used in this guide |
| **Exempt private company** | A Singapore private company limited by shares with ≤20 members and no corporate member (with exceptions); benefits include audit exemption and — critically — **no public filing of financial statements**, which is the structural cause of this guide's thin-public-footprint framing |
| **Full-stack** | Covering the entire application stack — frontend, backend, database, deployment/DevOps; Bidot's claim is "a full-stack development cycle from A to Z" with Flutter/React/TypeScript/PostgreSQL/cloud in its stated stack |
| **Custom software development** | Build-to-order software — bespoke systems commissioned by a client (as opposed to off-the-shelf products); SSIC 62011 in Bidot's registry record; the services category this guide's market context (§5) covers |
| **Application development** | The design, build and maintenance of software applications (web, mobile, desktop); one half of Bidot's self-description ("software and application development services provider") |
| **Strategists** | The upstream half of Bidot's self-identity: product thinking, requirements, industry analysis — the "strategists, designers and coders" trinity's first member; a website claim, not a verified capability |
| **Designers** | The middle member of the trinity: UI/UX, web/mobile design, brand identity, illustrations — the "Product Design" service area on bidot.sg |
| **Coders** | The delivery member of the trinity: the programming/engineering team behind the claimed stack (C#/.NET, Go, Python, TypeScript, Flutter, React, Vue, MySQL, PostgreSQL) |
| **Singapore** | City-state and the sole jurisdiction in Bidot Tech's public record — incorporation, registered office and contact all Singapore; home of the SG tech ecosystem cross-referenced in §5 |
| **Cecil Street** | A commercial street in Singapore's historic CBD (Raffles Place/Tanjong Pagar district); 20 Cecil Street, PLUS is Bidot Tech's registered office |
| **PLUS** | The office building at 20 Cecil Street, Singapore 049705 — an older-generation CBD commercial block housing Bidot Tech (#23-03), Bidot Investment Holdings (#23-04) and many unrelated tenants |
| **IT start-up** | Bidot's own self-classification ("Bidot Tech is an Information Technology (IT) start-up... led by outstanding entrepreneurs") — a website/LinkedIn claim about its own stage, not a funding or scale statement |
| **Software services** | The business of selling software capability as a service (custom builds, design, QA, support) rather than as a licensed product; Bidot Tech's tier of the market (§5.2) |
| **Agency** | The term Bidot's own copy uses for its kind of firm ("the mobile app development agency"); a client-services studio selling engagements — the T3 tier of the SG custom-dev landscape |
| **Delivery model** | The structured way a client engages and pays a vendor — phases, gates, contracts, ownership, QA and support terms; §6's worked example is a full delivery-model design |
| **Engagement** | A client-vendor working arrangement for a defined piece of work (contrast: subscription); the unit of revenue for a custom-dev agency like Bidot Tech |
| **Client** | The buyer of custom-dev services — for Bidot's stated pitch, "founders and product owners"; the party whose diligence discipline §6 designs for |
| **Singapore tech ecosystem** | The SG software landscape — SaaS companies, services firms, government machinery (IMDA, EDB, EnterpriseSG, SGInnovate, MAS) and talent pool mapped in [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md); the market context of §5 |

---

## 9. Claims Status, References and Further Reading

### Claims Status Table

| Claim | Status | Source |
|---|---|---|
| Incorporated 10 May 2019; UEN 201915071R issued 29 May 2019 | **Verified** | opengovsg.com/corporate/201915071R; companies.sg/business/201915071R/BIDOT-TECH-PTE-LTD- (ACRA-derived); UEN also embedded in MyCareersFuture URL |
| Exempt Private Company Limited by Shares; Local Company; Live Company | **Verified** | opengovsg.com; companies.sg |
| Registered office 20 Cecil Street, #23-03, PLUS, Singapore 049705 | **Verified** (registry); website prints #23-03/04 — flag | opengovsg.com; companies.sg; bidot.sg/contact.html |
| Primary & secondary SSIC 62011 (development & marketing of software program & website and online marketing) | **Verified** (corrects the brief's "software/application development (except games and cybersecurity)" paraphrase) | companies.sg; opengovsg.com |
| Former entity name DONG ER TECHNOLOGY; 7 officers; annual return 2026-07-08; account due 2027-11-30; paid-up capital S$7,401,440 | **Verified as registry facts** (officers, filings); capital **flagged** (single aggregator) | opengovsg.com; companies.sg |
| IPOS trademark-owner record exists | **Flagged** — existence only | companies.sg "Licenses & Accreditations" |
| Home/about/services/product/career/contact page texts (full-stack claim, strategists-designers-coders, IT start-up, service lines, case studies, contact details) | **Verified as website claims** | bidot.sg (all pages extracted live, Aug 2026) |
| LinkedIn positioning ("leading full-stack custom software and application development services provider", "led by outstanding entrepreneurs"; 336 followers) | **Flagged as claims** — page Cloudflare-blocked; content via search snippet | sg.linkedin.com/company/bidot-tech via Brave snippet |
| Partner logos: Crawfurd Hospital and Hug 2 Hearts are real organisations; relationship nature undisclosed | **Verified as real orgs; relationship flagged** | Wikipedia (Crawfurd Hospital); HUG Community Services channels (#HUG2HEARTS); cloud platforms' own sites |
| "2022 Team Building @ Bintan" | **Verified as website content**; team size unstated | bidot.sg/career.html |
| No public financials (EPC), no press coverage, no named clients, no certifications, no funding data | **Not applicable / absence flagged** — nothing fabricated | Thin public footprint (§1.7) |
| Market context (SG ecosystem, government machinery, T3 tier structure) | **Verified / flagged as per source guide** | [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) (its own claims ledger) |

### The Verification Pass — What Was Checked and How

For transparency about the research itself (August 2026; the self-hosted Firecrawl web backend was live, but its `web_search` route returned empty for several query shapes, so search was supplemented with direct engine fetches):

- **Registry record** — verified via two ACRA-derived aggregators using the UEN 201915071R as the anchor: opengovsg.com/corporate/201915071R (full record: incorporation, UEN issue date, entity type, status, address, SSIC, officer count, filing dates, former name, same-address and similar-business lists) and companies.sg/business/201915071R/BIDOT-TECH-PTE-LTD- (paid-up capital, secondary SSIC, IPOS note, similar-name/address lists). The UEN also appears in the government's MyCareersFuture company URL. A direct ACRA BizFile+ lookup was not performed (paywalled); aggregator consensus is treated as verified, with the capital figure and officer-count context flagged.
- **Corporate website** — bidot.sg index, /services.html, /product.html, /about.html, /career.html and /contact.html were all extracted live (the site is lightweight, static HTML with JS-loaded header/footer; extraction succeeded on all pages), plus the three product case-study subpages (product_security_app.html, product_charity.html, product_ehr.html). The home-page meta description (the "full-stack custom software and application development services provider... strategists, designers and coders" wording) was captured from the raw HTML. All quotations in §2–§4 are verbatim from that extraction, including the site's own typos ("We offers", "educing") marked [sic] where quoted.
- **LinkedIn** — the page sg.linkedin.com/company/bidot-tech returned HTTP 999/Cloudflare challenge to direct fetches and proxies; the positioning text and follower count were taken from a search-engine snippet of the page (Brave, Aug 2026) and are flagged accordingly.
- **Partner verification** — Crawfurd Hospital (real SG private hospital, 19 Adam Road; relaunched under Crawfurd Medical July 2021) and Hug 2 Hearts / HUG Community Services Limited (real IPC charity SSA) verified via search; the cloud platforms are self-evidently real vendors; the *relationships* are undisclosed and flagged.
- **Market context** — not re-researched from scratch; cross-referenced from the repository's own verified deep-dive ([singapore_saas_companies_guide.md](singapore_saas_companies_guide.md)) plus standard industry-structure framing flagged as analytical.
- **What failed / was not possible** — the Firecrawl web_search backend returned empty results for several query shapes (mitigated via direct Bing/Brave fetches); sgpbusiness.com, sgpgrid.com, ltddir.com, opencorporates.com and LinkedIn blocked automated extraction (facts taken from the two working aggregators + snippets); the live browser harness was unavailable (Chrome not running); ACRA BizFile+ direct lookup paywalled; the LinkedIn employee count, the DONG ER TECHNOLOGY rename date, and the officers' names are not public in this pass.

### References and Further Reading

- **Registry records (ACRA-derived)** — opengovsg.com/corporate/201915071R; companies.sg/business/201915071R/BIDOT-TECH-PTE-LTD-; companieshouse.sg search; mycareersfuture.gov.sg/companies/bidot-tech-201915071R (UEN embed); (sgpbusiness.com, sgpgrid.com, ltddir.com, opencorporates.com — attempted, bot-blocked)
- **Corporate website** — bidot.sg: index, services.html, product.html, about.html, career.html, contact.html, product_security_app.html, product_charity.html, product_ehr.html (all extracted live, August 2026)
- **LinkedIn** — sg.linkedin.com/company/bidot-tech (via search snippet, Aug 2026; page Cloudflare-blocked)
- **Partner-verification sources** — Wikipedia (Crawfurd Hospital — private hospital, 19 Adam Road, relaunched Jul 2021); HUG Community Services Limited (hug.org.sg / Instagram #HUG2HEARTS — IPC charity SSA); crawfurdhospital.com; cloud-platform vendor sites
- **Repository cross-refs (heavy)** — [singapore_saas_companies_guide.md](singapore_saas_companies_guide.md) (the SG software-company landscape — §5); [../banking/bond_financial_group_company_guide.md](../banking/bond_financial_group_company_guide.md) and [../banking/bond_capital_group_company_guide.md](../banking/bond_capital_group_company_guide.md) (the thin-public-footprint company-profile pattern); [../banking/partners_group_company_guide.md](../banking/partners_group_company_guide.md) (company-profile pattern, well-documented contrast)
- **Repository cross-refs (medium/light)** — [nec_asia_pacific_guide.md](nec_asia_pacific_guide.md) (house style); [singapore_data_centres_guide.md](singapore_data_centres_guide.md) (SG data residency — §6); [htx_ngine_guide.md](htx_ngine_guide.md) and [ica_systems_guide.md](ica_systems_guide.md) (SG public-sector tech — §5.3); [micro_frontend_architecture_guide.md](micro_frontend_architecture_guide.md) (frontend architecture — §6); [project_management_methodologies_guide.md](project_management_methodologies_guide.md) (delivery vocabulary); [../management/vendor_management_guide.md](../management/vendor_management_guide.md) (vendor diligence — §6); [../management/business_case_development_guide.md](../management/business_case_development_guide.md) (buyer's commercial case); [../management/mba_body_of_knowledge_guide.md](../management/mba_body_of_knowledge_guide.md) (services economics); [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) and [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) (the bank-buyer context behind §6's familiar setting)

*Nothing in this guide constitutes investment, legal, or procurement advice. All registry and website facts are as-of August 2026 and subject to change; all firm-level claims beyond the registry record are flagged per the thin-public-footprint honesty framing. The worked example (CareLane and Harbourline Dev) is entirely fictional; Bidot Tech's own public record contains no named clients, no disclosed team sizes, no revenue and no project evidence — and nothing here invents any.*


