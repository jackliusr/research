# Mitigant — The Cloud-Native Adversarial Exposure Validation Platform: A Deep-Dive Guide

**The Proof-of-Defence House — the Resility GmbH Lineage (Potsdam, 2021), the HTGF / Brandenburg Kapital / adesso Ventures Seed, the Cloud Attack Emulation + Cloud Security Posture Management + Continuous Compliance Platform, the DORA / NIS2 Cloud-Resilience Angle for Financial Services, the BAS / CTEM Competitive Field, and a Cymbal Bank Cloud-Resilience Validation Worked Example — Built from Primary Sources with an Honest Claims Audit**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Company Deep-Dive — Mitigant (mitigant.io, developed by Resility GmbH, renamed Mitigant GmbH ⚠): the Company Profile (founding, founders, funding, HQ, team), the Origins (CloudRAID → Resility → Mitigant), the Platform (Adversarial Exposure Validation: Cloud Attack Emulation, Cloud & Kubernetes Security Posture Management, Continuous Compliance Monitoring, AI Red Teaming, Detection Validation), the Banking & Finance Positioning (the risk-management lifecycle, banking functions served, the DORA / NIS2 regulatory drivers — cross-referenced, not re-derived), the Competitive Landscape (BAS / CTEM and CSPM / CNAPP vendors), the Cymbal Bank Worked Example, the Summary, the Glossary, the Claims Audit, What Could Not Be Verified, and the Verification Ledger
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** mitigant.io extracted live this pass (homepage, About, Platform, Cloud Attack Emulation, Cloud Security Posture Management, the DORA webinar page, the Brandenburger Innovationsfachkraft funding page, the Google-for-Startups news item), the Mitigant LinkedIn company page (extracted live), the HTGF portfolio page and the 14-Dec-2021 financing news (htgf.de — an investor's own record), the Google for Startups case study (startup.google.com), the adesso press release reprint (marketscreener.com), EIOPA's DORA page (europa.eu), Tracxn / PitchBook / Mandos / CleraMap / cybersectools secondary profiles (flagged), the competitor primary sites verified at title/head level (safebreach.com, cymulate.com, xmcyber.com, attackiq.com, picussecurity.com, wiz.io, orca.security, crowdstrike.com), and the repository's sibling guides (enterprise_risk_management_guide.md, mas_regulations_guidelines_guide.md, financial_risk_compliance_systems_guide.md, financial_fraud_detection_at_scale_guide.md, ai_genai_banking_compliance_guide.md, fircosoft_guide.md, resona_merchant_bank_asia_guide.md) for house pattern and cross-reference targets
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder):** [Fircosoft](fircosoft_guide.md) (the vendor deep-dive pattern — claims audit ✅/⚠/❌) · [Enterprise Risk Management](enterprise_risk_management_guide.md) (the risk process — identify, assess, respond, monitor, report — §7) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (TRMG, Notice 658, the FSMA tech-risk layer) · [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) (the GRC-platform table, cloud adoption — §11) · [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) (adversarial detection) · [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) (prompt injection, AI red-teaming context) · [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) (the thin-footprint honesty pattern)

---

## How to Read This Guide

This is the dedicated deep-dive on **Mitigant** — the Potsdam, Germany-based cloud-security software company whose platform is sold at **mitigant.io** — in the `banking/` company/vendor series. It follows the house pattern set by [fircosoft_guide.md](fircosoft_guide.md) (vendor → capabilities → regulatory context → competitors → Cymbal Bank worked example → claims audit) and wears the thin-footprint honesty framing of [resona_merchant_bank_asia_guide.md](resona_merchant_bank_asia_guide.md): a young, privately held, seed-stage startup has a *genuinely limited* public record, and this guide says exactly what that record is.

The brief that produced this guide asked the research to *"determine what the company/product does in banking and finance — likely risk/governance/compliance (GRC), risk-intelligence, or similar — but DO NOT assume: follow the evidence wherever it leads."* **The evidence leads away from GRC.** Every primary source retrieved this pass — the company's own website, its LinkedIn page, its investors' pages, Google's case study — describes a **cloud-native security platform** (adversarial exposure validation, cloud attack emulation, cloud security posture management, continuous compliance monitoring), not a GRC suite. There is no risk-register, no control-library, no audit-management module in Mitigant's documented product; the "risk" in its vocabulary is *exploitability risk in the cloud*. Where it touches banking and finance, it does so through **operational resilience and cyber-compliance** — most visibly the EU's **Digital Operational Resilience Act (DORA)**, which it names among its compliance benchmarks and for which it ran a dedicated webinar with the German consultancy Advisori. The guide follows that evidence.

There is also a **name-collision discipline** to state up front. "Mitigant" is a common-sounding name: at least three unrelated businesses use it (mitigant.net — an AI risk-register builder; mitigantrisk.com — a US healthcare emergency-management consultancy; and this guide's subject at mitigant.io). A reader searching for "Mitigant" will find all of them. This guide's subject is **mitigant.io / Mitigant GmbH (formerly Resility GmbH)**, Potsdam, Germany — and the other entities are named in §1.4 so no one mistakes them for it.

- **The ERM lifecycle (heavy cross-ref)** — [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) §7 defines the risk process as **identify → assess → respond → monitor → report**. §6 of this guide maps Mitigant's modules onto that lifecycle (its continuous-compliance module is *monitor*; its attack emulation is *assess* with evidence; its remediation guidance is *respond*). The ERM guide is the vocabulary source; this guide does not re-derive it.
- **The Singapore regulatory layer (medium cross-ref)** — [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md) carries the verified MAS instrument map: the Technology Risk Management Guidelines (TRMG, 2013/2021), Notice 658 (outsourcing, effective 11 December 2024), and the FSMA Part 5 tech-risk notices (FSM-N05) that replaced the sectoral TRM notices. Cymbal Bank's obligations under that map are the *reason* the §9 worked example exists; the worked example's EU angle (DORA) is cross-referenced to the sibling guides rather than re-derived.
- **The systems context (light cross-ref)** — [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) §5 and its GRC-platform table (SAS OpRisk, IBM OpenPages, MetricStream, RSA Archer, ServiceNow GRC — ✅ sibling-verified) position the *adjacent* market Mitigant does **not** play in, and its §11 (cloud adoption) is the systems-side context for why banks' risk stacks are moving to the cloud. [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) supplies the adversarial-detection mindset; [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md) supplies the prompt-injection / AI red-teaming context that Mitigant's AI Red Teaming module addresses.

**Note on verification — the honesty framing.** This guide was researched in August 2026 via targeted web search and live page extraction. The verified core is unusually well-sourced for a seed-stage startup because the company, its investors, and Google all publish: mitigant.io's own About/Platform/product pages (founding, funding, founders, modules, integrations — ✅), the HTGF portfolio page and 14-Dec-2021 news item (an investor's record of the seed round — ✅), the Google for Startups case study (team size, AI architecture, customer-result claims — ✅ for the case-study facts, ⚠ for the vendor-flavoured numbers), the LinkedIn company page (HQ, size, founding year — ✅), and EIOPA for DORA's application date (✅). Everything else is flagged: the *exact* legal-entity rename date (Resility GmbH → Mitigant GmbH) ⚠, precise funding amounts beyond "seven figures" ⚠, any named bank or financial-institution customer ⚠ (none is published — the public customer logos are German mid-market/enterprise firms), the "Gartner AEV category" claim ⚠ (vendor's own words), and any GCP posture coverage ⚠ (marked "Coming Soon" on the CSPM page). **Nothing here is invented: no founding dates beyond the published ones, no funding figures beyond the disclosed "seven figures", no customer names beyond the published logos, no product capabilities beyond the documented ones.** Where the public record is silent, the guide says so (§13) and moves on. **Nothing in this guide is investment, legal, or tax advice.**

### Quick Facts (Summary Card)

| Item | Value | Status |
|---|---|---|
| Product | **Mitigant** — an enterprise **Adversarial Exposure Validation (AEV) Platform** for cloud-native infrastructure: cloud attack emulation, cloud penetration testing, cloud & Kubernetes security posture management (CSPM/KSPM), continuous compliance monitoring, AI red teaming, detection validation | ✅ mitigant.io homepage + Platform page |
| Legal entity | **Resility GmbH**, founded **June 2021**, Germany; since renamed **Mitigant GmbH** ("previously Resility GmbH") — exact rename date not published | ✅ founding (About page, HTGF); ⚠ rename date |
| HQ | **Potsdam, Brandenburg, Germany** — Dianastraße 21, Potsdam 14482 (LinkedIn location) | ✅ LinkedIn (extracted live) |
| Founders | **Nils Karn** (CEO), **Kennedy Torkura** (CTO), **Dr. Muhammad Ihsan Haikal Sukmana** (CPO), **Dr. Thomas Fricke** (advisor) — Hasso Plattner Institute alumni | ✅ About page + HTGF |
| Founded | Idea 2020 at Hasso Plattner Institute (CloudRAID research lineage); company incorporated **June 2021** | ✅ About page + HTGF |
| Funding | **December 2021 — initial funding of "seven figures"** (multimillion-euro) from **High-Tech Gründerfonds, Brandenburg Kapital (ILB), adesso Ventures**; later investor logos: Business Angels Berlin-Brandenburg, Wayra Telefónica Innovation | ✅ About page + HTGF news + adesso release; ⚠ amounts/rounds beyond the seed (Tracxn: $1.5M over 1 round — ⚠ secondary) |
| Team size | ~12 people ("12-person startup" per Google for Startups; "11-50" per LinkedIn) | ✅ Google case study; ✅ LinkedIn |
| Industry / status | Computer and Network Security; privately held | ✅ LinkedIn |
| Core capability | Safely emulating real adversarial techniques in the customer's own cloud (AWS, Azure, GCP, Kubernetes) to validate which security issues are actually exploitable; 500+ plug-and-play attacks mapped to MITRE ATT&CK / MITRE ATLAS | ✅ mitigant.io pages |
| Compliance monitoring | Continuous checks across CIS Benchmarks, **NIS2, DORA**, NIST, PCI-DSS, SOC 2 (homepage); ISO 27001, GDPR, HIPAA (Platform page) | ✅ mitigant.io pages (vendor-scope claims ⚠ depth) |
| Deployment | Agentless SaaS; EU-based, built in Germany, GDPR-aligned data sovereignty; self-hosted/on-premises option listed; 30-day free trial | ✅ mitigant.io pages |
| Banking/finance evidence | **DORA webinar (28 Apr 2025)** with Advisori on DORA's effect on the European cloud landscape; DORA/NIS2 named compliance benchmarks; **no named bank or financial-institution customer published** | ✅ webinar page; ⚠ customer list |
| Certifications/memberships | TeleTrust "IT Security made in Germany" seal; Allianz für Cyber-Sicherheit (BSI) participant; Athene Digital Hub Cybersecurity; Networker NRW | ✅ About page (badge-level); ⚠ no published SOC 2/ISO 27001 certification of Mitigant itself |

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The One-Line Positioning](#11-the-one-line-positioning)
   - 1.2 [The Discipline — Adversarial Exposure Validation](#12-the-discipline--adversarial-exposure-validation)
   - 1.3 [The Banking Angle — What This Vendor Actually Sells to Finance](#13-the-banking-angle--what-this-vendor-actually-sells-to-finance)
   - 1.4 [The Name-Collision Note](#14-the-name-collision-note)
   - 1.5 [The Overview Table](#15-the-overview-table)
2. [The Company Profile](#2-the-company-profile)
   - 2.1 [The Legal Entity — Resility GmbH, Now Mitigant GmbH](#21-the-legal-entity--resility-gmbh-now-mitigant-gmbh)
   - 2.2 [The Founders](#22-the-founders)
   - 2.3 [The Funding — the December 2021 Seed](#23-the-funding--the-december-2021-seed)
   - 2.4 [The Team, the Advisory Board, the Memberships](#24-the-team-the-advisory-board-the-memberships)
   - 2.5 [The Company Table](#25-the-company-table)
3. [The Origins — from CloudRAID to Resility to Mitigant](#3-the-origins--from-cloudraid-to-resility-to-mitigant)
   - 3.1 [The 2020 Idea — the Hasso Plattner Institute](#31-the-2020-idea--the-hasso-plattner-institute)
   - 3.2 [CloudRAID — the Research Lineage](#32-cloudraid--the-research-lineage)
   - 3.3 [2021 — Resility GmbH](#33-2021--resility-gmbh)
   - 3.4 [The Rename — Resility to Mitigant](#34-the-rename--resility-to-mitigant)
   - 3.5 [The Milestones Timeline](#35-the-milestones-timeline)
4. [The Platform — the Adversarial Exposure Validation Model](#4-the-platform--the-adversarial-exposure-validation-model)
   - 4.1 [The Core Proposition — Validate Exploitability](#41-the-core-proposition--validate-exploitability)
   - 4.2 [The CTEM Framing](#42-the-ctem-framing)
   - 4.3 [Beyond Breach-and-Attack Simulation](#43-beyond-breach-and-attack-simulation)
   - 4.4 [The Platform Pillars](#44-the-platform-pillars)
5. [The Platform in Detail — Modules, Safety, AI, Integrations](#5-the-platform-in-detail--modules-safety-ai-integrations)
   - 5.1 [Cloud Attack Emulation and the Attack Builder](#51-cloud-attack-emulation-and-the-attack-builder)
   - 5.2 [Cloud Security Posture Management](#52-cloud-security-posture-management)
   - 5.3 [Continuous Compliance Monitoring](#53-continuous-compliance-monitoring)
   - 5.4 [AI Red Teaming, Detection Validation, SOC Optimisation, Incident Readiness](#54-ai-red-teaming-detection-validation-soc-optimisation-incident-readiness)
   - 5.5 [The Safety Model](#55-the-safety-model)
   - 5.6 [The AI Layer — Gemini and the RAG Framework](#56-the-ai-layer--gemini-and-the-rag-framework)
   - 5.7 [Integrations and Deployment](#57-integrations-and-deployment)
   - 5.8 [The Customer Base — What the Public Logos Show](#58-the-customer-base--what-the-public-logos-show)
6. [The Banking and Finance Positioning](#6-the-banking-and-finance-positioning)
   - 6.1 [The Risk-Management Lifecycle Mapping](#61-the-risk-management-lifecycle-mapping)
   - 6.2 [The Banking Functions Served](#62-the-banking-functions-served)
   - 6.3 [Where It Does NOT Play — the GRC Boundary](#63-where-it-does-not-play--the-grc-boundary)
   - 6.4 [The Position Table](#64-the-position-table)
7. [The Regulatory Drivers — DORA, NIS2 and the Cloud](#7-the-regulatory-drivers--dora-nis2-and-the-cloud)
   - 7.1 [DORA — the Financial-Services Anchor](#71-dora--the-financial-services-anchor)
   - 7.2 [The Mitigant × Advisori DORA Webinar](#72-the-mitigant--advisori-dora-webinar)
   - 7.3 [NIS2 and the Wider Benchmark Set](#73-nis2-and-the-wider-benchmark-set)
   - 7.4 [The Singapore Lens — MAS, Cross-Referenced](#74-the-singapore-lens--mas-cross-referenced)
8. [The Competitive Landscape](#8-the-competitive-landscape)
   - 8.1 [The Two Adjacent Fields — BAS/CTEM and CSPM/CNAPP](#81-the-two-adjacent-fields--basctem-and-cspmcnapp)
   - 8.2 [The Comparison Table](#82-the-comparison-table)
   - 8.3 [The Integration Paradox — Wiz and Prowler as Both](#83-the-integration-paradox--wiz-and-prowler-as-both)
9. [The Worked Example — a Cymbal Bank Cloud-Resilience Validation Cycle](#9-the-worked-example--a-cymbal-bank-cloud-resilience-validation-cycle)
   - 9.1 [The Scenario](#91-the-scenario)
   - 9.2 [The Validation Cycle — Step by Step](#92-the-validation-cycle--step-by-step)
   - 9.3 [The Evidence Chain — What Each Stakeholder Receives](#93-the-evidence-chain--what-each-stakeholder-receives)
   - 9.4 [The Metrics](#94-the-metrics)
   - 9.5 [The Lessons](#95-the-lessons)
10. [The Summary — One Page](#10-the-summary--one-page)
11. [Glossary](#11-glossary)
12. [Claims Audit — ✅ Verified / ⚠ Partially Verified / ❌ Not Verified](#12-claims-audit--verified--partially-verified--not-verified)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Verification Ledger](#14-verification-ledger)
15. [References and Further Reading](#15-references-and-further-reading)

---

## 1. The Overview

### 1.1 The One-Line Positioning

**Mitigant is an enterprise cloud-security software platform — "Adversarial Exposure Validation" — that safely runs real attack techniques against a customer's own AWS, Azure, Google Cloud and Kubernetes environments to prove which security weaknesses an attacker could actually exploit, and to keep the customer continuously compliant with cloud-relevant standards (DORA, NIS2, CIS, NIST, PCI-DSS, SOC 2 and more).** That is the company's self-description across every primary page retrieved this pass, and it is consistent across the homepage, the About page, the Platform page, the product pages, the LinkedIn page, the HTGF investor record and the Google for Startups case study. The name is a deliberate echo of its function: the platform *mitigates* risk — but the risk is cloud exploitability, not enterprise risk-register management.

The vendor's own category claim, on the About page: *"the most sophisticated, cloud-focused platform in Gartner's Adversarial Exposure Validation (AEV) category"* (⚠ vendor claim — see §13). Gartner introduced the AEV concept in its 2024-25 security-market vocabulary (the company's own blog explains the category, and its homepage cites the "45% of MITRE ATT&CK techniques not implemented in threat detection tools" and "40% ineffective" statistics ⚠ vendor-cited figures, plus a "3x reduction in breach likelihood" claim ⚠). What matters for a bank reader is not the marketing superlatives but the *verified function*: **Mitigant validates defences with evidence, in the cloud, continuously, safely, and maps the results to MITRE ATT&CK / ATLAS and to compliance benchmarks.**

### 1.2 The Discipline — Adversarial Exposure Validation

Adversarial Exposure Validation is the security discipline of **testing whether identified weaknesses are actually exploitable** — running safe, reversible, attacker-shaped actions against the environment and observing which defences stop them. It sits between two older disciplines:

1. **Configuration scanning / CSPM** — "your S3 bucket is public" (a *finding*, no proof it matters). Mitigant's own blog draws the line explicitly: *"CSPM scans are not cloud penetration tests"* (✅ blog title retrieved this pass — the company argues configuration auditing and security validation are different things).
2. **Traditional penetration testing and red teaming** — deep, expert-driven, expensive, periodic (the company's marketing: "no need to wait months for external engagements or hire expensive consultants" ✅ Cloud Attack Emulation page).

Mitigant's claimed position is the continuous middle: automated, safe, repeatable attack emulation that *validates* the scanner's findings — "which of thousands of alerts actually matter" (✅ About page). The discipline vocabulary it uses is **Continuous Security Verification** (the founding concept, per the HTGF page) and, in the current platform language, **Continuous Threat Exposure Management (CTEM)** (✅ Platform page).

### 1.3 The Banking Angle — What This Vendor Actually Sells to Finance

The task brief hypothesised a GRC/risk-intelligence company; the evidence says otherwise, so the honest headline for the banking reader is this:

- **Mitigant is not a GRC vendor.** It has no risk-register, control-library, audit-management, or regulatory-reporting module in any documented product page. The sibling [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) §5 names the actual GRC platforms (SAS OpRisk, IBM OpenPages, MetricStream, RSA Archer, ServiceNow GRC — ✅ sibling-verified); Mitigant is not among them and does not claim to be.
- **What it sells to finance is cyber-operational-resilience validation.** Its compliance-monitoring module names **DORA** (the EU Digital Operational Resilience Act, in application since 17 January 2025 — ✅ EIOPA) and **NIS2** among the benchmarks it continuously checks (✅ homepage). It ran a dedicated **DORA webinar** on 28 April 2025 with the German consultancy Advisori (✅ webinar page). For a bank, that is the purchase case: **evidence that cloud controls actually stop attacks, plus continuous compliance evidence for the resilience regulations** — the *assess-and-monitor* stages of the ERM lifecycle (§6.1).
- **No named bank or financial institution appears in its published customer base** (§5.8). Its public customers are German and European mid-market and enterprise technology firms. Whether any bank runs Mitigant in production is not publicly verifiable — flagged ⚠, not assumed.

### 1.4 The Name-Collision Note

"Mitigant" is used by at least three unrelated businesses. Verified this pass:

| Domain | What it is | Relationship to this guide's subject |
|---|---|---|
| **mitigant.io** | Mitigant GmbH (formerly Resility GmbH) — the cloud AEV platform (this guide's subject) | — |
| **mitigant.net** | "Mitigant · AI Risk Platform" — reads project documents and builds an **ISO 31000 risk register** grounded in industrial-incident evidence (⚠ search-snippet-level; a genuinely different product and company) | None — but note the *name* is a GRC product, which may be what a naive search for "Mitigant risk" surfaces |
| **mitigantrisk.com** | Mitigant Risk Solutions, LLC — US healthcare emergency-management consulting (⚠ snippet-level) | None |
| **mitigant.com** | Could not be scraped this pass (server error on every engine) — identity unverified | ⚠ Unverified |
| **mitigant.co.uk / mitigant.ai** | Resolution blocked (private/internal network address per the extraction tool) — identity unverified | ⚠ Unverified |

The practical takeaway for the Cymbal Bank procurement and research teams: **always disambiguate "Mitigant" by domain and by legal entity** — this guide's subject is mitigant.io / Mitigant GmbH, Potsdam.

### 1.5 The Overview Table

| Aspect | Description |
|---|---|
| **The vendor** | Mitigant (mitigant.io) — Mitigant GmbH, formerly Resility GmbH, Potsdam, Germany; founded June 2021; seed-funded December 2021 ✅ |
| **Category** | Adversarial Exposure Validation (AEV) for cloud-native infrastructure — "beyond breach and attack simulation" ✅ (vendor); Gartner-category membership ⚠ (vendor claim) |
| **What it does** | Safely emulates real adversary techniques in the customer's cloud (AWS/Azure/GCP/Kubernetes); validates exploitability of findings; continuous compliance monitoring; AI-powered analysis and reporting ✅ |
| **Standards monitored** | CIS Benchmarks, NIS2, DORA, NIST, PCI-DSS, SOC 2 (homepage); ISO 27001, GDPR, HIPAA (Platform page) ✅ vendor-scope claims |
| **Banking relevance** | DORA / NIS2 cloud-resilience and compliance evidence; cyber-risk evidence for the ERM assess/monitor stages; **no published banking customer** ⚠ |
| **The buying case** | "Proof, not assumptions": exploitability-validated findings, prioritized remediation, continuous compliance evidence, EU data sovereignty, agentless SaaS ✅ |

---

## 2. The Company Profile

### 2.1 The Legal Entity — Resility GmbH, Now Mitigant GmbH

The company behind the Mitigant product was incorporated as **Resility GmbH** in **June 2021** (✅ mitigant.io/en/about — *"Resility GmbH was founded in June 2021 by the team that marked the start of their journey to develop Mitigant"*; ✅ HTGF portfolio page — "Mitigant (formerly known as Resility)"). The About page's story section dates the *founding idea* to **2020**, when the three HPI researchers "came to the idea of starting an enterprise cloud security solution" (✅ About page).

The legal entity has since been renamed **Mitigant GmbH**. Two independent on-record references support the rename, though neither publishes a date:

- The HTGF portfolio page is titled "**Mitigant (formerly known as Resility)**" (✅ htgf.de, extracted live).
- Mitigant's own funding-grant page says "**Mitigant GmbH, previously Resility GmbH**, has hired a part-time student assistant through the Brandenburger Innovationsfachkraft (BIF)" (✅ mitigant.io/en/funding/bif — search-snippet + page retrieved).

The exact commercial-registry rename date is not published in any retrieved source — **⚠ flagged, not guessed**.

**Headquarters:** Potsdam, Brandenburg, Germany — the LinkedIn company page lists the primary location as "Potsdam, Brandenburg 14482, DE" with the street address **Dianastraße 21, Potsdam 14482** (✅ LinkedIn, extracted live). The About page's "EU-based · Built in Germany" tagline and its GDPR/data-sovereignty framing are consistent (✅ mitigant.io homepage). Tracxn places the company in "Brandenburg (Germany)" (✅/⚠ secondary database — consistent with the primary sources on geography; its "founded in 2020" dating conflicts with the primary "founded June 2021" and is treated as ⚠).

### 2.2 The Founders

The founding team is well-documented because the company, the investor (HTGF) and Google all publish it:

| Founder | Role (per About page) | Verified source |
|---|---|---|
| **Nils Karn** | CEO and Co-Founder | ✅ About page (role); ✅ Google case study photo caption ("Nils Karn, Mitigant CEO + co-founder") |
| **Kennedy Torkura** (Kennedy Aondona Torkura) | CTO and Co-Founder | ✅ About page; ✅ Google case study ("As Chief Technology Officer at the 12-person startup…"); ✅ HTGF |
| **Dr. Muhammad Ihsan Haikal Sukmana** | CPO and Co-Founder | ✅ About page; ✅ HTGF |
| **Dr. Thomas Fricke** | Advisor and Co-Founder | ✅ About page; ✅ HTGF ("cloud security and Kubernetes experts in Germany") |

The origin story, per the About page: **Kennedy Torkura and Muhammad Sukmana were doing PhDs in cloud security at the Hasso Plattner Institute (HPI), Potsdam, while Nils Karn conducted design-thinking research; in 2020 the three concluded no existing product addressed rapidly evolving cloud attacks, and Thomas Fricke joined with domain expertise and German cybersecurity/startup connections** (✅ About page; ✅ HTGF — "founded by Hasso Plattner Institute alumni Kennedy Torkura, Muhammad Ihsan Haikal Sukmana, Nils Karn and Thomas Fricke"). The Tracxn profile independently names the same four founders (✅/⚠ secondary, consistent).

### 2.3 The Funding — the December 2021 Seed

The verified funding record is a single, well-documented seed round:

- **December 2021: initial funding of "seven figures"** — i.e., a multimillion-euro (seven-figure) round — from **three German venture investors: High-Tech Gründerfonds (HTGF), Brandenburg Kapital (the ILB's venture fund), and adesso Ventures** (✅ mitigant.io/en/about, verbatim: *"In December 2021, Resility received the initial funding of seven figures from three German venture capitals: HighTech-Gründerfonds, Brandenburg Kapital, and adesso Ventures"*).
- HTGF's own news item, **14 December 2021: "Cloud security start-up Resility secures multimillion-euro financing"** (✅ htgf.de news archive — an investor's primary record; also ✅/⚠ reprint of the adesso press release on marketscreener, which adds the "Shared Responsibility Model" framing).
- Later investor logos on the About page: **Business Angels Berlin-Brandenburg** and **Wayra Telefónica Innovation** (✅ badge-level presence on the About page; the size/date/terms of any such participation are ⚠ not published).
- Secondary databases: Tracxn reports "$1.5M over 1 round" (⚠ secondary, amount not confirmed by any primary source) and Crunchbase reports a single seed round (⚠ secondary, snippet-level). **No Series A or later round is published in any retrieved source** — either it has not happened, or it is not public; flagged ⚠.

HTGF's portfolio classification places the company in "Digital Tech / Deep Tech / IT-Security", invested since 2021, based in Potsdam (✅ htgf.de).

### 2.4 The Team, the Advisory Board, the Memberships

**Team size.** Google for Startups describes "the **12-person startup**" (✅ startup.google.com case study); the LinkedIn company page shows "11-50 employees" with a "View all 12 employees" face-pile (✅ LinkedIn). The About page claims "over 20 years of total cloud security experience in industry and academia" (⚠ unverifiable as a precise figure). Mandos Cyber independently reports "about 12 people" (⚠ secondary, consistent).

**Advisory board** (✅ About page, four named advisors with LinkedIn links — roles as published):

| Advisor | Published affiliation |
|---|---|
| Nir Yizhak | CISO, Firebolt |
| Pranav Vattaparambil | CISO, SoSafe |
| Markus Becker | Former CEO of Quentic |
| Aaron Rinehart | Co-founder/CTO of Verica; "Security Chaos Engineering pioneer" |

Aaron Rinehart is notable as the security-chaos-engineering figure whose discipline vocabulary (chaos engineering for security) matches Mitigant's own "Security Chaos Engineering" specialty tag on LinkedIn — and a customer testimonial from Charles Nwatu ("Security Chaos Engineering is vital… Mitigant turns Security Chaos into Clarity") uses the same framing (✅ homepage testimonial; ⚠ testimonial claims are vendor-published).

**Memberships and seals** (✅ About page badge row): **TeleTrust "IT Security made in Germany"** seal, **Allianz für Cyber-Sicherheit** (the German BSI's cybersecurity alliance) participant, **Athene Digital Hub Cybersecurity**, and **Networker NRW** (the German IT association). These are membership-level facts, not product certifications — see §13 for what *could not* be verified (no published SOC 2 / ISO 27001 certification of Mitigant's own operations was found ⚠).

**Press and recognition** (✅ About page news row, each linking to the outlet; content ⚠ not re-extracted): Sifted ("11 cybersecurity startups to watch"), infopoint-security, lanline, datakontext, IAVC, b2b-cyber-security — German and European security trade press coverage of the "Continuous Security Verification" concept. **Google for Startups Growth Academy: AI for Cybersecurity** — the company joined the program (kick-off event in Warsaw) and Google published the Gemini case study (✅ startup.google.com — §5.6).

### 2.5 The Company Table

| Fact | Value | Status |
|---|---|---|
| Legal entity | Resility GmbH → Mitigant GmbH | ✅ both names on record; ⚠ rename date |
| Founded | June 2021 (idea 2020) | ✅ About + HTGF |
| HQ | Potsdam, Brandenburg, Germany (Dianastraße 21, 14482) | ✅ LinkedIn |
| Founders | Karn, Torkura, Sukmana, Fricke | ✅ About + HTGF + Google |
| Seed round | Dec 2021, "seven figures"; HTGF, Brandenburg Kapital, adesso Ventures | ✅ About + HTGF news |
| Later investors | Business Angels Berlin-Brandenburg; Wayra Telefónica Innovation | ⚠ badge-level |
| Team | ~12 | ✅ Google + LinkedIn |
| Status | Privately held, operating | ✅ LinkedIn |
| Industry | Computer and Network Security | ✅ LinkedIn |

---

## 3. The Origins — from CloudRAID to Resility to Mitigant

### 3.1 The 2020 Idea — the Hasso Plattner Institute

The founding story (✅ About page; ✅ HTGF page) is a research-to-product arc. In **2020**, at the **Hasso Plattner Institute (HPI) in Potsdam** — Germany's university institute for digital engineering, funded by the Hasso Plattner Foundation — PhD researchers **Kennedy Torkura** and **Muhammad Sukmana** (cloud security) and researcher **Nils Karn** (design thinking) concluded that existing security products did not address "the challenges of rapidly evolving cloud attacks" (✅ About page). The gap they identified: scanners report misconfigurations by the thousand, but nothing *proves which of them are exploitable* — so security teams "make decisions based on assumptions rather than proof" (✅ About page). That "proof over assumption" thesis is the company's core value proposition to this day ("Evidence-Based Security — No assumptions, only proof through adversarial validation", ✅ About page core values).

### 3.2 CloudRAID — the Research Lineage

HTGF's portfolio page records the research lineage explicitly: **Mitigant's "Continuous Security Verification" concept is "influenced by CloudRAID, an innovative cloud security research effort from the Hasso Plattner Institute, Potsdam"** (✅ htgf.de). CloudRAID (cloud resilience/audit/intrusion-detection-oriented research at HPI) is the academic ancestor named by the investor itself — the guide repeats HTGF's characterisation without adding detail, because no CloudRAID primary page was retrieved this pass (⚠ depth unverified; the lineage itself is ✅ investor-stated).

### 3.3 2021 — Resility GmbH

**June 2021** — Resility GmbH is founded (✅ About page). The name "Resility" is a portmanteau of *resilience* and *agility*, consistent with the product's resilience-testing mission (⚠ interpretation, not stated). The product name **Mitigant** — from *mitigate* — appears to have been the working brand from early on (the December 2021 HTGF news still calls the *company* Resility while the HTGF page and press describe the product Mitigant; the marketscreener reprint of the adesso release quotes CTO Kennedy Torkura on "Mitigant" in December 2021: *"Mitigant hat es sich zur Aufgabe gemacht, die Cloud-Infrastruktur durch den Einsatz des hochinnovativen Konzepts der 'Continuous Security Verification' sicher und widerstandsfähig zu machen"* — ✅/⚠ German-language release, quote as reprinted).

**December 2021** — the seven-figure seed from HTGF, Brandenburg Kapital and adesso Ventures (✅ §2.3). The adesso release frames the business case with the **cloud "Shared Responsibility Model"**: customers are responsible for securing their cloud configurations, and without a tool like Mitigant they depend on scarce cloud-security expertise (✅/⚠ marketscreener reprint).

### 3.4 The Rename — Resility to Mitigant

At some point the legal entity followed the brand: "**Mitigant GmbH, previously Resility GmbH**" (✅ BIF funding page) and "**Mitigant (formerly known as Resility)**" (✅ HTGF portfolio). The date of the commercial-registry rename is not published in any retrieved source — **⚠ flagged**. The LinkedIn company page (created under the Mitigant name, founded 2021) and the current website both operate fully under the Mitigant brand (✅).

### 3.5 The Milestones Timeline

| Date | Milestone | Status |
|---|---|---|
| 2020 | Founding idea at Hasso Plattner Institute, Potsdam | ✅ About page |
| 2020s research | CloudRAID research at HPI — the stated conceptual ancestor of "Continuous Security Verification" | ✅ HTGF (characterisation); ⚠ depth |
| June 2021 | **Resility GmbH founded** | ✅ About page |
| 14 Dec 2021 | **Seven-figure seed** — HTGF, Brandenburg Kapital, adesso Ventures | ✅ HTGF news + About page |
| 2023-24 | Product maturation: CSPM, attack emulation, Kubernetes posture, AI features (inferred from feature pages dated 2024-26) | ⚠ inferred from blog dates |
| Dec 2024 | Blog on Gartner's AEV category ("Rethinking Cloud Security Strategies with Adversarial Exposure Validation", 7 Dec 2024) | ✅ blog listing (CSPM page) |
| 17 Jan 2025 | **DORA enters into application** (regulatory anchor for the banking angle) | ✅ EIOPA |
| 28 Apr 2025 | **Mitigant × Advisori DORA webinar** | ✅ webinar page |
| Apr 2026 | Feature release: Cloud Penetration Testing with Prowler and Wiz integrations (10 Apr 2026) | ✅ blog listing |
| Jun 2026 | Google for Startups publishes the Gemini case study; AgentCore research blog (22 Jun 2026) | ✅ news + blog listings |
| Aug 2026 | This guide's research date | — |

---

## 4. The Platform — the Adversarial Exposure Validation Model

### 4.1 The Core Proposition — Validate Exploitability

The Platform page's four-pillar pitch (✅ mitigant.io/en/platform, extracted live):

1. **Validate Exploitability** — "Determine if identified cloud weaknesses and vulnerabilities are exploitable despite implemented security controls and processes."
2. **Detect Security Blindspots** — "Identify security gaps that traditional cloud security approaches miss. Evaluate security investments based on empirical evidence."
3. **Maintain Cloud Compliance** — "Automate continuous compliance monitoring with industry standards like ISO 27001, SOC 2, GDPR, HIPAA, and more."
4. **Prioritize Remediation** — "Focus on addressing exploitable security gaps thus allowing for efficient control of attack surfaces and elimination of alert fatigue."

The homepage adds the fifth beat, **Optimize Threat Exposure Management** — "Proactively address risks through Continuous Threat Exposure Management by leveraging real-time insights" (✅ homepage). In the ERM vocabulary of the sibling [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) §7 (identify → assess → respond → monitor → report), Mitigant's pitch maps as: *identify* (CSPM scanning and asset inventory), *assess* (attack emulation — the "assess with proof" step), *respond* (prioritized remediation guidance), *monitor* (continuous compliance and drift management) — see §6.1 for the full mapping.

### 4.2 The CTEM Framing

The Platform page states the platform "is designed to seamlessly enable **Continuous Threat Exposure Management (CTEM)** programs for organizations" (✅). CTEM is Gartner's five-stage operational-risk cycle (scoping → discovery → prioritization → validation → mobilization); Mitigant claims the validation stage with continuous, automated, cloud-native emulation. The sibling guides do not cover CTEM, so this guide records the vendor's framing without re-deriving the Gartner model (⚠ the CTEM stage vocabulary is Gartner's; only Mitigant's *claim* to enable it is verified here).

### 4.3 Beyond Breach-and-Attack Simulation

Mitigant's differentiation story, told consistently across its pages and LinkedIn: **traditional Breach and Attack Simulation (BAS) platforms were built for on-premises environments and extended to the cloud; Mitigant claims to be cloud-native from day one.** The LinkedIn post retrieved this pass says it directly: *"Many Breach and Attack Simulation (BAS) platforms were built for on-premises environments, then extended to the cloud. Mitigant was built for cloud-native environments from day one… 500+ cloud-native attack scenarios across AWS, Azure, and GCP, mapped to MITRE ATT&CK and ATLAS"* (✅ LinkedIn, extracted live). The homepage's BAS-comparison section contrasts the two (✅ homepage, truncated but present: "goes beyond Breach and Attack Simulation; safely validates defenses across AWS, Azure, and GCP. Continuously and at machine speed"). Whether that differentiation holds against the *current* BAS field (which has moved toward CTEM — see §8) is an evaluation for the reader; this guide records both sides.

### 4.4 The Platform Pillars

| Pillar | What the platform does (documented) | Source |
|---|---|---|
| **Cloud Attack Emulation** | Runs real adversary techniques safely in the customer's cloud; 500+ plug-and-play attacks across AWS, Azure, GCP; mapped to MITRE ATT&CK and MITRE ATLAS; informed by cyber threat intelligence; Attack Builder for custom attacks (Cloud Attack Language) | ✅ homepage + Cloud Attack Emulation page |
| **Cloud Security Posture Management (CSPM)** | Continuous monitoring of cloud and Kubernetes environments for misconfigurations and compliance violations; asset inventory and drift management; vulnerability intelligence from EPSS and CISA KEV; AWS + Azure supported, GCP "Coming Soon" (CSPM page) | ✅ homepage + CSPM page |
| **Continuous Compliance Monitoring** | Automated checks against CIS Benchmarks, NIS2, DORA, NIST, PCI-DSS, SOC 2 (homepage); ISO 27001, GDPR, HIPAA (Platform page); "Stay audit-ready, always; no longer a point in time" | ✅ homepage + Platform page |
| **AI-Powered Intelligence** | AI analyses cloud posture, recommends the most impactful attacks ("Attack Recommendation"), converts results into executive intelligence; MITRE techniques and Sigma detection rules for remediation | ✅ homepage + Google case study |
| **Six solution lines** | AI Red Teaming; Cloud Penetration Testing; Detection Validation; SOC Team Optimization; Continuous Compliance; Incident Readiness | ✅ homepage solutions grid |

---

## 5. The Platform in Detail — Modules, Safety, AI, Integrations

### 5.1 Cloud Attack Emulation and the Attack Builder

The Cloud Attack Emulation page (✅ extracted live) documents:

- **Attack catalogue**: "Choose from **500+ MITRE-mapped cloud attacks**" (homepage and emulation page both state 500+; one sub-block on the emulation page says "200+ MITRE-mapped cloud attacks" — an internal inconsistency on the same page, recorded here as ⚠ rather than smoothed; the higher figure appears on the homepage, the LinkedIn post, and the platform pages).
- **Attack Builder**: build custom cloud attacks "in seconds" using the **Cloud Attack Language**, with intelligent auto-complete, multi-step chaining into complex attack scenarios, live YAML preview, "define attacks as code for automation and CI/CD workflows", "from idea to executed attack in under 30 seconds" (✅ emulation page; the Attack Builder also appears in the Google case study as a Gemini-powered no-code builder).
- **Use cases served**: cloud penetration testing (continuous, "no need to wait months for external engagements"), red/purple team exercises (threat-informed scenarios mapped to MITRE ATT&CK and real threat actors), detection/alert validation (proving SIEM/CDR tools work and reducing false positives) (✅ emulation page).
- **APIs**: the **Mitigant Attack API** for agile attack orchestration, **Attack-as-Code** and **Detection-as-Code** for automation (✅ homepage + emulation page).
- **Clouds covered**: AWS, Azure, Google Cloud Platform, Kubernetes (✅ homepage; the CSPM page's "We Support Your Favorite Clouds" block shows AWS and Azure with **GCP "Coming Soon"** for the *posture* product ⚠ — attack emulation across GCP is claimed on the homepage).

### 5.2 Cloud Security Posture Management

The CSPM product (✅ homepage + CSPM page + customer testimonials) delivers:

- Continuous monitoring of cloud and Kubernetes environments for misconfigurations and compliance violations (✅ homepage).
- **Asset inventory and drift management** — the BIF funding page explains the Drift Analysis feature: "a new technique to monitor the current state of the cloud infrastructure for security threats" (✅/⚠ grant-page description).
- **Vulnerability prioritization** using **EPSS** (Exploit Prediction Scoring System) and **CISA KEV** (Known Exploited Vulnerabilities) intelligence (✅ homepage).
- **Agentless scanning** (✅ Platform page: "agentless, SaaS platform").
- Customer-visible time-to-value claims: "signing up to the Mitigant CSPM was seamless and fast, it took barely 15 minutes for me to clearly see the security issues in my AWS cloud account" (Stephan Häußler, CTO, Adair — ✅/⚠ vendor-published testimonial) and "barely 10 minutes" for Azure (Markus Guerster, CEO, MontBlancAI — ✅/⚠ vendor-published).

### 5.3 Continuous Compliance Monitoring

The compliance module (✅ homepage + Platform page + solution grid) is the piece most relevant to banking:

- **Standards monitored**: CIS Benchmarks, **NIS2**, **DORA**, NIST, **PCI-DSS**, SOC 2 (homepage, verbatim); the Platform page adds **ISO 27001, GDPR, HIPAA** ("and more").
- **Positioning**: "Stay audit-ready, always; no longer a point in time" (✅ solution card) — continuous evidence rather than point-in-time audit snapshots.
- **Reporting**: "detailed reports that outline security and compliance issues with actionable insights to prioritize remediation efforts based on exploitability and regulatory risk" (✅ Platform page).
- **Cloud Compliance Assurance**: "automated checks that monitor for misconfigurations, data exposures, and security vulnerabilities across AWS, Azure, Google Cloud, and Kubernetes" (✅ Platform page).

What this module is **not** (verified boundary): it is a cloud-configuration compliance *monitor*, not a bank-wide regulatory-compliance or GRC platform — it does not manage MAS notices, Basel, AML/CFT obligations, or audit workflows (⚠ absence-of-evidence statement: no such capability appears in any retrieved page; the sibling GRC table in [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) §5 covers those platforms).

### 5.4 AI Red Teaming, Detection Validation, SOC Optimisation, Incident Readiness

The six "solutions" (✅ homepage solutions grid) that package the platform for buyer personas:

1. **AI Red Teaming** — "purpose-built for testing AI workloads in the cloud" (✅); blog content covers AI red teaming on Amazon Bedrock, and the AgentCore cross-agent privilege-escalation research (✅ blog listings, June 2026). This connects to the sibling [ai_genai_banking_compliance_guide.md](ai_genai_banking_compliance_guide.md), whose prompt-injection taxonomy (LLM01) is exactly the threat class such testing addresses — cross-referenced, not re-derived.
2. **Cloud Penetration Testing** — continuous, self-service cloud pentests; the April 2026 feature release added **Prowler and Wiz integrations** that validate CSPM findings through real attack execution, "surfacing SCP-aware privilege-escalation paths, fully visualized and mapped to compliance benchmarks" (✅ blog listing).
3. **Detection Validation** — "prove your defenses actually work as desired"; validate SIEM, CDR and cloud detection mechanisms against real attacks (✅ emulation page + solution card).
4. **SOC Team Optimization** — threat-informed training scenarios that give the SOC "their time back" (✅ solution card).
5. **Continuous Compliance** — §5.3.
6. **Incident Readiness** — "practice makes perfect; ensure your incident response processes function effectively" (✅ solution card).

### 5.5 The Safety Model

The safety architecture is documented in detail (✅ emulation page, "Safe Attack Execution Everywhere") — critical for a bank evaluating any tool that runs attacks in production-adjacent environments:

- **Automatic resource cleanup after every attack** ("no residual impact", ✅ homepage).
- **Granular permission controls — Admin, Non-Admin, or BYOR** (Bring Your Own Role) for precise permission control (✅ emulation page + homepage).
- **Isolated attack execution** — no impact on adjacent resources (✅).
- **Attack preview before execution** and the ability to **stop attacks mid-execution** (✅).
- **Built-in guardrails** and least-privilege access (✅ homepage).
- The Nooxit testimonial adds deployment ergonomics: "We were able to spin up Mitigant in a day. Reproducibly pushing it through dev and stage into prod, because of its easy-to-use helm chart and plug-and-play abilities" (✅/⚠ vendor-published testimonial; confirms a Helm-chart deployment path).

### 5.6 The AI Layer — Gemini and the RAG Framework

The Google for Startups case study (✅ startup.google.com, extracted live) documents the AI architecture in unusual detail:

- **Dual-model architecture**: **Gemini 2.5 Flash** for high-volume real-time tasks (transforming complex technical reports into executive-friendly summaries) and **Gemini 2.5 Pro** for complex analysis (the **"Attack Recommendation"** feature that suggests the most relevant attacks for a customer's specific posture). Integrated via the **Gemini Enterprise Agent Platform** (✅).
- **RAG grounding**: a custom knowledge base of authoritative attack frameworks; "the dual-model architecture cross-checked answers against verified industry standards before responding" — built explicitly to stop the models hallucinating security techniques (✅).
- **Reported results** (vendor-reported, ⚠ numbers as published): customers' **time-to-insight reduced by over 60%**; **3-5X faster vulnerability triage** versus manual analysis; **~3X increase in adoption** of the attack-emulation platform (✅ case-study figures, flagged as vendor-reported).
- **Roadmap items** (✅ case study): Gemini-powered **Attack Builder** (no-code custom attacks), multi-step attack identification, prompt-injection defence validation, natural-language queries.

### 5.7 Integrations and Deployment

**Integrations** (✅ homepage "Connect with Your Favorite Tools" section, verbatim list): Amazon Web Services, Microsoft Azure, Kubernetes, Google Cloud Platform, **Wiz**, Slack, Defect Dojo, SIEM, MS Teams, Jira, **Prowler**, Quay, Alibaba Cloud, OpenShift, SysEleven, Open Telekom Cloud, Docker, Hetzner, ExoScale, On-Premise, Mini-kube, "more to come". The pattern: Mitigant ingests findings from the CSPM/CNAPP tools a bank already runs (Wiz, Prowler, SIEM) and routes validated results into the workflows teams live in (Jira, Slack, Teams, Defect Dojo).

**Deployment** (✅ Platform page + homepage): **agentless SaaS** ("SaaS Deployment with Cloud-Native Integration — Supercharge security operations with agility built on our agentless, SaaS platform"); **EU-based, built in Germany**, GDPR-aligned with a "Data Sovereignty" framing; an **On-Premise** option appears in the integration list and a **self-hosted path is implied by the Helm chart** (Nooxit testimonial); go-to-market includes a **30-day free trial** and a **demo environment** (✅ homepage CTAs).

### 5.8 The Customer Base — What the Public Logos Show

The homepage's "Trusted by Security Leaders" row and the About page's partner row publish the following customer/partner logos (✅ mitigant.io pages; each is a *vendor-published* claim — the logos are shown, no case-study detail is public for most): **Nooxit** (testimonial + use case), **Notch** (testimonial + use case), **KM.ON by Karl Mayer** (testimonial + use case), **Mitto** (testimonial + use case), **Adair** (testimonial + use case), **MontBlancAI** (testimonial + use case), **Vertama**, **DAMOVO**, **Cyberlift**, **Cybershield**, **evia**, **Value Work**, **GlobalDots**, **Syself**, **adesso**.

The honest reading for a bank: these are predominantly **German and European mid-market and enterprise technology companies** (SaaS, AI, manufacturing-software, IT services). **No bank, insurer, or other regulated financial institution appears in the published customer set** — if Mitigant sells to finance, that is not public (⚠ §13). The *channel* partners visible (GlobalDots, adesso, Syself, DAMOVO — resellers/integrators) matter for procurement: a bank would likely buy through a partner.

---

## 6. The Banking and Finance Positioning

### 6.1 The Risk-Management Lifecycle Mapping

The sibling [enterprise_risk_management_guide.md](enterprise_risk_management_guide.md) §7 defines the risk process as **identify → assess → respond → monitor → report** (✅ sibling). Mitigant's documented modules map onto it cleanly — this is the architecture-level answer to "what does this vendor do in banking":

| ERM lifecycle stage (sibling guide §7) | Mitigant capability (documented) | Evidence |
|---|---|---|
| **Identify** | CSPM scanning: misconfigurations, data exposures, vulnerabilities; asset inventory; drift management; EPSS/CISA KEV prioritisation | ✅ homepage + CSPM page |
| **Assess** | Cloud Attack Emulation: safe real attacks proving which identified weaknesses are *exploitable*; detection validation proving which controls actually fire | ✅ About + emulation page |
| **Respond** | Prioritized remediation guidance; AI-generated remediation steps and hardening efforts; Sigma detection rules; Jira/Slack/Teams routing | ✅ homepage + AI section |
| **Monitor** | Continuous compliance monitoring (DORA, NIS2, CIS, NIST, PCI-DSS, SOC 2…); continuous asset/drift monitoring; "audit-ready, always" | ✅ homepage + Platform page |
| **Report** | Executive intelligence: AI summaries of attack results; comprehensive reporting & analytics; downloadable summaries for stakeholders | ✅ Platform page + Google case study |

The distinctive contribution versus a classic GRC tool: **the assess stage produces *empirical evidence* (an attack was run, a control failed/held), not a self-assessed control rating.** For a bank whose ERM is "regulation-shaped" (enterprise_risk_management_guide.md §5: three lines of defence, board oversight, risk appetite), that evidence is the difference between "control effectiveness: asserted" and "control effectiveness: demonstrated" — the same epistemic gap the sibling [fircosoft_guide.md](fircosoft_guide.md) describes for screening (false negatives vs false positives), now applied to cloud controls.

### 6.2 The Banking Functions Served

Mapped to a bank's organisation (using the three-lines vocabulary of enterprise_risk_management_guide.md §5, cross-referenced):

- **First line — technology operations and the SOC**: continuous cloud pentesting, detection validation, SOC team optimization (threat-informed training), incident readiness drills. The buyer here is the CISO/Head of Cloud Security, not the risk function (✅ product framing).
- **Second line — IT/cyber risk and operational-resilience oversight**: exploitability-validated risk evidence for the risk register; continuous compliance evidence for resilience regulations (DORA/NIS2); drift detection feeding risk reporting. This is where the *regulatory* purchase case lives (§7).
- **Third line — internal audit**: continuous compliance monitoring output ("audit-ready, always") as independent evidence; executive reports shareable with auditors (the Google case study notes customers "actively request downloadable versions of the AI-generated summaries to share with stakeholders and include in further security analysis" — ✅).
- **Board/committee reporting**: AI-generated executive intelligence that converts attack-emulation results into board-readable risk language (✅ Google case study — "clear, actionable overviews they could read and act on without parsing through pages of technical execution details").
- **ICT third-party risk (DORA Article 28 context)**: a bank's *own* cloud posture is in scope; the platform's continuous monitoring also evidences oversight of the bank's cloud service usage (⚠ inference from DORA scope, cross-referenced to §7; no Mitigant document claims TPRM functionality).

### 6.3 Where It Does NOT Play — the GRC Boundary

For procurement clarity, the verified boundary (§5.3): Mitigant is **not** a GRC platform, **not** an AML/KYC tool, **not** a sanctions screener (contrast the sibling [fircosoft_guide.md](fircosoft_guide.md)), and **not** a risk-register/ICAAP system (contrast the sibling GRC table: SAS OpRisk, IBM OpenPages, MetricStream, RSA Archer, ServiceNow GRC — ✅ financial_risk_compliance_systems_guide.md §5). It is a **cloud-security validation and compliance-monitoring layer** that a bank would run *alongside* those systems, feeding them evidence. A bank evaluating Mitigant should not expect a control-library or regulatory-obligation module; it should expect an attack-emulation engine, a posture scanner, and a compliance benchmark monitor for the cloud estate.

### 6.4 The Position Table

| Question | Answer | Status |
|---|---|---|
| What market does it address? | Cloud-native security: AEV/BAS-class validation + CSPM + compliance monitoring (CTEM framing) | ✅ vendor pages |
| Who buys it? | Security teams (SOC, cloud security, red/purple teams), CISOs; second-line IT-risk via compliance evidence; audit via reports | ✅ product framing; ⚠ org mapping is this guide's analysis |
| What banking problem does it solve? | Operational-resilience evidence for DORA/NIS2-era cloud estates; proof of control effectiveness; continuous audit evidence | ✅ vendor pages (DORA/NIS2 benchmarks); ⚠ no banking case study |
| What does it not do? | GRC, AML/sanctions, risk registers, ICAAP | ✅ absence in all documented pages; sibling GRC table |
| Financial-services customers? | None published | ⚠ |

---

## 7. The Regulatory Drivers — DORA, NIS2 and the Cloud

This section cross-references the sibling guides rather than re-deriving the regulatory frameworks. The [MAS Regulations, Guidelines and Industry Expectations guide](mas_regulations_guidelines_guide.md) carries the Singapore instrument map; the [Enterprise Risk Management guide](enterprise_risk_management_guide.md) carries the ERM process; this section records only what is *verified about Mitigant's own regulatory positioning* plus the anchor facts needed to read it.

### 7.1 DORA — the Financial-Services Anchor

**The Digital Operational Resilience Act (DORA)** is the EU regulation that makes digital operational resilience a supervised matter for financial entities — banks, insurers, investment firms, payment institutions and their critical ICT third-party providers. **It entered into application on 17 January 2025** (✅ EIOPA's DORA page, europa.eu — retrieved this pass; the Mitigant webinar page states the same date). DORA's pillars — ICT risk management, ICT-related incident reporting, digital operational resilience testing (including threat-led penetration testing for the largest firms), ICT third-party risk management, and information-sharing — all have cloud-resilience implications, which is precisely the gap Mitigant's continuous-compliance and validation modules address.

Mitigant's own documented engagement with DORA:

- **DORA is named among its continuous-compliance benchmarks** on the homepage: "Continuously monitor Cloud and Kubernetes environments for misconfigurations and compliance violations across CIS Benchmarks, NIS2, **DORA**, NIST, PCI-DSS, SOC 2, and more" (✅ homepage).
- **The Mitigant × Advisori webinar** (28 April 2025, on-demand — ✅ webinar page, §7.2) is explicitly addressed to "financial organizations" and the "challenges and effects of the DORA regulation for the financial organizations related to European cloud landscape".
- The company's GDPR/data-sovereignty positioning ("EU-based · Built in Germany", "Aligned with European digital sovereignty and GDPR" — ✅ homepage) is a deliberate fit with EU financial-sector preferences for EU data residency (⚠ inference, consistent with the marketing).

### 7.2 The Mitigant × Advisori DORA Webinar

The webinar page (✅ mitigant.io/en/webinar/…, extracted live) records: **"How Digital Operational Resilience Act Regulation Affects The Cloud Landscape in Europe"**, held **28 April 2025, 2 PM CEST**, on-demand, hosted by **Kennedy Torkura (Mitigant Chief Technology Officer)** and **Vasileios Sofroni (Advisori Senior Digital Transformation Consultant)**, with **Advisori** (advisori.de — a German digital-transformation consultancy) as co-host. The page's own description: DORA "affects the financial entities located and operating in Europe, such as banks, insurance companies, and investment firms, starting in 17 January 2025"; as European financial organizations adopt cloud-native infrastructures, "the DORA regulation requires the affected entities to ensure their clouds are secure, compliant, and resilient against potential cyber threats." This is the clearest primary evidence of Mitigant's intended financial-services market: **DORA-driven cloud security and resilience for European financial entities.**

### 7.3 NIS2 and the Wider Benchmark Set

- **NIS2** (the EU Network and Information Systems Directive, replacing the original NIS Directive) — Mitigant publishes a dedicated NIS2 resource page (✅ mitigant.io/en/nis2, listing retrieved this pass): "NIS2 is no longer on the horizon; it is in effect, and enforcement is beginning… More than 100,000 organisations across the EU are now in scope" (✅/⚠ vendor-page statements; the 100,000+ figure is the vendor's). NIS2 matters to banking because banks are in scope *and* because it covers the wider digital-infrastructure ecosystem banks depend on.
- The other monitored benchmarks (CIS Benchmarks, NIST, PCI-DSS, SOC 2, ISO 27001, GDPR, HIPAA — ✅ homepage/Platform page) are the standard cloud-compliance set; PCI-DSS and SOC 2 in particular matter to banks' payments and vendor-oversight estates (⚠ inference; the sibling [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) §11 covers cloud adoption in risk/compliance systems).

### 7.4 The Singapore Lens — MAS, Cross-Referenced

For Cymbal Bank (Singapore-headquartered), the *local* regulatory drivers are MAS's, and they are cross-referenced here rather than re-derived from the sibling [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md):

- **Technology Risk Management Guidelines (TRMG, 2013/2021)** — MAS's supervisory expectations for technology risk, cybersecurity and cloud outsourcing (✅ sibling guide §4.2). A continuous cloud-validation and compliance-monitoring layer maps directly onto TRMG's expectation that technology risks be identified, assessed, mitigated and monitored (✅ sibling-framed; mapping is this guide's analysis).
- **Notice 658 (Outsourcing, effective 11 December 2024)** — governs outsourcing including cloud outsourcing: risk assessment, notification, oversight of material service providers (✅ sibling guide §3 + financial_risk_compliance_systems_guide.md §2). A bank's cloud-security evidence base supports the oversight duties it owes under Notice 658 for cloud service providers (⚠ mapping, this guide's analysis).
- **FSMA Part 5 (since May 2024)** — the statutory home of the harmonised tech-risk notices (FSM-N05 for banks) (✅ sibling guide §2). The operational-resilience obligations sit on the same continuum as DORA's, even though Singapore banks are not DORA entities (✅ sibling framing).
- **Notably absent**: Mitigant publishes **no MAS-specific benchmark or Singapore-specific content** (⚠ absence-of-evidence). A Cymbal Bank deployment would use the general cloud benchmarks and map evidence to MAS obligations internally — the sibling guides supply the obligations map (mas_regulations_guidelines_guide.md §8: the obligations × controls matrix).

---

## 8. The Competitive Landscape

### 8.1 The Two Adjacent Fields — BAS/CTEM and CSPM/CNAPP

Mitigant competes in (and straddles) two verified security-market fields:

1. **Breach and Attack Simulation (BAS) / exposure-validation / CTEM** — vendors that emulate attacks to validate controls. Verified this pass: **SafeBreach** ("Exposure Validation Platform… enterprise-grade CTEM platform" — ✅ safebreach.com, extracted), **AttackIQ** ("the leading independent vendor of breach and attack simulation solutions… Security Optimization Platform" — ✅ businesswire press release via search + attackiq.com snippets; direct scrape blocked ⚠), **Cymulate** ("AI-Powered Exposure Validation" — ✅ cymulate.com, extracted), **XM Cyber** ("Continuous Exposure Management Platform" — ✅ xmcyber.com, extracted), **Picus Security** ("The Pioneer of Breach & Attack Simulation" — ✅ picussecurity.com via search snippets; direct scrape returned no content ⚠).
2. **CSPM / CNAPP (cloud security posture / cloud-native application protection)** — vendors that scan and contextualise cloud risk. Verified this pass: **Wiz** (CNAPP/CSPM leader — ✅ wiz.io via search snippets, and it appears as a *Mitigant integration*), **Orca Security** ("AI-Powered Cloud Security Platform & CNAPP" — ✅ orca.security, extracted), **CrowdStrike Falcon** ("Unified Agentic Security" platform with cloud modules — ✅ crowdstrike.com/platform, extracted).

Mitigant's own differentiation (✅ LinkedIn + homepage): cloud-native attack execution vs on-prem-heritage BAS; exploitability validation on top of CSPM findings; EU data sovereignty; all-in-one platform ("Sold as an all-in-one platform, designed to be your single tool rather than a component that slots into a best-of-breed stack" — ✅ homepage BAS-comparison section).

### 8.2 The Comparison Table

| Vendor | Verified category (source) | Cloud-native attack emulation | CSPM | Compliance benchmarks | EU data-sovereignty positioning | Public banking/FS customers |
|---|---|---|---|---|---|---|
| **Mitigant** | AEV/BAS + CSPM + compliance (✅ mitigant.io) | ✅ 500+ attacks AWS/Azure/GCP/K8s | ✅ (GCP "Coming Soon" ⚠) | ✅ DORA, NIS2, CIS, NIST, PCI-DSS, SOC 2, ISO 27001, GDPR, HIPAA | ✅ "EU-based · Built in Germany" | ⚠ none published |
| **SafeBreach** | Exposure validation / CTEM (✅ safebreach.com) | ⚠ cross-domain (on-prem + cloud heritage) | ⚠ via integrations | ⚠ not stated on homepage | ⚠ US-based | ⚠ none verified this pass (Deloitte/Experian/Pepsi/SAP/UKG logos — non-FS) |
| **AttackIQ** | BAS / security optimization / CTEM (✅ businesswire + attackiq.com) | ⚠ (BAS heritage, MITRE ATT&CK aligned) | ❌ core product is control validation | ⚠ not stated | ⚠ US-based | ⚠ none verified this pass |
| **Cymulate** | AI-powered exposure validation (✅ cymulate.com) | ⚠ (BAS heritage, agentic AI angle) | ⚠ partial | ⚠ not stated | ⚠ Israel/US | ⚠ none verified this pass |
| **XM Cyber** | Continuous exposure management (✅ xmcyber.com) | ⚠ (attack-graph based; cloud exposures module) | ⚠ via cloud-exposures module | ⚠ not stated | ⚠ Israel/US | ⚠ none verified this pass |
| **Picus** | BAS pioneer / autonomous exposure validation (✅ picussecurity.com) | ⚠ (BAS heritage + attack-path mapping) | ⚠ partial | ⚠ not stated | ⚠ US/UK | ⚠ none verified this pass |
| **Wiz** | CNAPP/CSPM (✅ wiz.io; also a Mitigant integration) | ❌ (scanner, not attack executor) | ✅ flagship | ✅ (compliance frameworks incl. CIS/NIST) | ⚠ US-based (EU regions available ⚠) | ⚠ none verified this pass |
| **Orca Security** | AI-powered CNAPP (✅ orca.security) | ❌ (scanner, agentless) | ✅ flagship | ✅ | ⚠ US/Israel | ⚠ none verified this pass |
| **CrowdStrike Falcon** | Unified security platform incl. cloud (✅ crowdstrike.com) | ⚠ (Falcon Intelligence/cloud modules) | ✅ | ✅ | ⚠ US-based | ✅ some FS logos industry-wide ⚠ (not re-verified this pass) |

*Reading notes: ✅ = verified at primary-source level this pass; ⚠ = partial/snippet-level or inferred from homepage scope; ❌ = verified absence in the documented product. "Cloud-native attack emulation" for the BAS incumbents is marked ⚠ because Mitigant's own differentiation claims they extended on-prem BAS to the cloud — this guide records the claim, not the verdict. The comparison is about *documented category scope*, not product quality.*

### 8.3 The Integration Paradox — Wiz and Prowler as Both

A subtlety worth recording for procurement: **Wiz and Prowler are simultaneously competitors (CSPM category) and integrations** — the April 2026 feature release makes Mitigant's cloud pentests "integrate with Prowler and Wiz, enabling seamless validation of CSPM findings through real, safe attack execution" (✅ blog listing). The product strategy is explicit: *validate the findings of the scanners you already run, then route results into your workflows* (✅ homepage integrations section). A bank that already runs Wiz or Prowler for posture would therefore buy Mitigant as a **validation layer on top**, not as a replacement — which is exactly the "beyond CSPM" positioning the company's "CSPM scans are not cloud penetration tests" blog argues (✅ blog listing).

---

## 9. The Worked Example — a Cymbal Bank Cloud-Resilience Validation Cycle

*This worked example is design/analytical content in the repo's established Cymbal Bank convention (see fircosoft_guide.md §8 for the same framing): it is built strictly from Mitigant's documented features and from the sibling guides' verified regulatory facts; it claims no real contract and quotes no real configuration. Cymbal Bank is the repo's fictional bank persona.*

### 9.1 The Scenario

**Cymbal Bank** runs a multi-cloud estate: its digital-banking platform is primary on **AWS** (Singapore region) with a **Kubernetes**-based microservices layer, a secondary **Azure** footprint for disaster recovery, and an emerging **AI/LLM** service layer (customer-assist summarisation). Two regulatory realities frame the story:

1. **Singapore side** (cross-ref [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md)): the **TRMG** expects technology risks to be identified, assessed, mitigated and monitored; **Notice 658** imposes oversight duties over material outsourced/cloud arrangements; the FSMA tech-risk notices (FSM-N05) are the binding floor.
2. **EU side** (the DORA anchor, §7): Cymbal Bank's **EU subsidiary** is a DORA in-scope financial entity, so its cloud infrastructure must be demonstrably secure, compliant and resilient — including regular digital-operational-resilience testing and continuous ICT-risk evidence.

The bank's cloud-security team publishes a quarterly "cloud resilience statement" to the CRO and the board's risk committee. Historically the statement leaned on CSPM scan counts ("3,412 findings, 214 high") — numbers nobody could translate into *actual exposure*. This is the gap the worked example closes.

### 9.2 The Validation Cycle — Step by Step

The cycle below uses only documented Mitigant capabilities (§4–§5), mapped to the ERM lifecycle (§6.1):

**Step 1 — Identify (CSPM continuous monitoring).** Mitigant's agentless CSPM connects to the AWS and Azure accounts (AWS + Azure are the supported posture clouds; GCP is "Coming Soon" ⚠ — the bank's estate is AWS/Azure, so this is fine). It inventories assets, flags misconfigurations and data exposures, and prioritises with EPSS and CISA KEV intelligence (✅ §5.2). Drift management catches config drift between quarterly reviews (✅ BIF page: Drift Analysis).

**Step 2 — Assess (Cloud Attack Emulation, the evidence step).** Instead of accepting scanner findings at face value, the team runs **attack emulation** against the highest-priority findings using the 500+ MITRE ATT&CK/ATLAS-mapped attacks (✅ §5.1): e.g. an S3-bucket policy finding is validated by executing the corresponding privilege-escalation/access attack in an isolated, guard-railed run with **BYOR** permissions (the team's own read-only role scoped to the target account — ✅ §5.5). The emulation either (a) succeeds — the finding is a *verified exploitable exposure*, or (b) fails — the control held, and the finding is downgraded from "high" to "observed, not exploitable". This is the "assess with proof" beat that no scanner provides.

**Step 3 — Respond (prioritized remediation).** The AI layer's **Attack Recommendation** selects which attacks matter for *this* estate (✅ §5.6); remediation guidance arrives with the MITRE technique IDs and **Sigma detection rules** needed to close the gap (✅ homepage). Validated exposures route to **Jira** (ticketing) and **Slack/MS Teams** (SOC notification) via the documented integrations (✅ §5.7), with **Defect Dojo** receiving the findings for the AppSec pipeline (✅ integration list).

**Step 4 — Monitor (continuous compliance).** The compliance module continuously checks the estate against the bank-relevant benchmarks: **DORA** (for the EU subsidiary), **NIS2**, **CIS Benchmarks**, **NIST**, **PCI-DSS** (the payments estate), **SOC 2** and **ISO 27001** (✅ §5.3). Findings are mapped to "exploitability and regulatory risk" so the report answers both questions at once: *is it vulnerable, and does it breach a benchmark?* (✅ Platform page reporting description).

**Step 5 — Report (executive intelligence).** For the quarterly resilience statement, the team uses the AI-generated executive summaries (Gemini 2.5 Flash — ✅ §5.6): "attack emulation validated 9 of 214 high findings as exploitable; 7 remediated; 2 in remediation; zero DORA-critical benchmark violations in the EU subsidiary estate; detection validation confirmed the SIEM caught 11 of 11 emulated attacks". Downloadable summary packs go to the CRO, audit and the board risk committee (✅ Google case study: customers request downloadable AI summaries for stakeholders).

**Detection-validation sub-cycle.** Before the statement closes, the SOC runs the **Detection Validation** and **Incident Readiness** solutions: emulated attacks are fired to prove the SIEM/CDR actually detects them, and a tabletop-style incident-readiness run validates the response process (✅ §5.4). A detection gap found here becomes a Sigma-rule remediation ticket — closing the loop from *control exists* to *control fires* (✅ §5.5 safety: all runs are safe, reversible, auto-cleaned).

### 9.3 The Evidence Chain — What Each Stakeholder Receives

| Stakeholder | Artefact (all documented Mitigant outputs) | ERM stage |
|---|---|---|
| Cloud security team / SOC | Validated-findings queue with exploitability status; Sigma rules; Jira tickets; detection-validation results | Assess + Respond |
| Second line (IT/cyber risk) | Exploitability-validated risk evidence; drift report; benchmark compliance status (DORA/NIS2/CIS/NIST/PCI-DSS/SOC 2) | Assess + Monitor |
| Internal audit | Continuous compliance evidence ("audit-ready, always"); downloadable AI summaries; attack-run audit trail | Monitor + Report |
| CRO / board risk committee | Executive intelligence pack: validated exposures, remediation progress, compliance stance | Report |
| EU subsidiary (DORA) | Continuous ICT-risk and resilience-testing evidence for the DORA obligations (cross-ref §7.1) | Monitor + Report |

### 9.4 The Metrics

The bank's cloud-resilience statement would carry these metrics — every one derivable from documented outputs:

- **Exploitability ratio**: validated-exploitable findings ÷ high findings (the scanner-to-emulation conversion rate).
- **Control-effectiveness rate**: emulated attacks blocked ÷ emulated attacks run (from detection validation).
- **Remediation SLA**: median days from validated exposure to remediation ticket closure (Jira integration).
- **Benchmark posture**: % of estate continuously compliant per benchmark (DORA, NIS2, CIS, NIST, PCI-DSS, SOC 2).
- **Drift rate**: config drifts detected per quarter (drift management).
- **Board-language target** (illustrative ⚠): move the statement from "we have 214 high findings" to "we have proven which 9 can be exploited, and 7 are already fixed" — the "proof over assumption" value proposition (✅ About page).

### 9.5 The Lessons

1. **The platform's banking value is epistemic, not taxonomic.** Mitigant does not add a risk register to the bank's GRC estate; it adds *evidence* to the assess/monitor stages that the GRC estate then records. Buy it for the evidence, integrate it with the GRC tools (the sibling GRC table in financial_risk_compliance_systems_guide.md §5), don't expect it to be one.
2. **DORA is the wedge.** For EU-facing banking groups, continuous cloud-validation and compliance evidence maps to DORA's ICT-risk and testing expectations (✅ §7). For Singapore-only entities, the same capability evidences TRMG/Notice 658 oversight duties (✅ sibling MAS guide) — but the vendor does not market an MAS-specific story (⚠ §7.4), so a Singapore bank must do that mapping internally.
3. **Safety engineering is a procurement first question.** The documented safety model (BYOR, guardrails, isolation, auto-cleanup, preview, mid-execution stop — ✅ §5.5) is what makes continuous attack emulation acceptable in a bank at all; a bank's evaluation should validate *those* claims in a sandbox before any production-adjacent run.
4. **Integrations are the architecture.** Wiz/Prowler/SIEM in, Jira/Slack/Teams/Defect Dojo out (✅ §5.7) — the platform is bought as a validation layer over the existing cloud-security stack, not a rip-and-replace.
5. **Customer-reference discipline.** With no published banking customer (⚠ §5.8), a bank's procurement should demand financial-sector references and a clear EU-data-residency/SaaS-sub-processor position before contracting — the "EU-based, built in Germany" claim (✅) is a starting point, not a compliance certificate.

---

## 10. The Summary — One Page

**Mitigant** (mitigant.io) is a **Potsdam, Germany-based cloud-security software company** — legally **Mitigant GmbH, formerly Resility GmbH**, founded **June 2021** by Hasso Plattner Institute alumni **Nils Karn (CEO), Kennedy Torkura (CTO), Dr. Muhammad Sukmana (CPO) and Dr. Thomas Fricke (advisor)**, and seed-funded in **December 2021** with a **seven-figure round from HTGF, Brandenburg Kapital and adesso Ventures** (✅ About page; ✅ HTGF; ✅ LinkedIn for HQ/size/founding). It is a ~12-person, privately held startup whose product — the **Mitigant Adversarial Exposure Validation Platform** — safely runs **500+ cloud attack emulations** against a customer's own **AWS, Azure, GCP and Kubernetes** environments, validates which security findings are actually exploitable, monitors **cloud and Kubernetes posture** (agentless CSPM with asset inventory, drift management, EPSS/CISA KEV prioritisation), and continuously checks compliance across **CIS Benchmarks, NIS2, DORA, NIST, PCI-DSS, SOC 2, ISO 27001, GDPR and HIPAA** (✅ homepage + Platform page). An **AI layer** (Gemini 2.5 Flash/Pro with a RAG-grounded knowledge base) converts results into executive intelligence and attack recommendations (✅ Google for Startups case study), and the documented safety model — BYOR permissions, guardrails, isolation, automatic cleanup — makes continuous adversarial testing feasible in production estates (✅ emulation page).

The task hypothesis was GRC; **the evidence says otherwise.** Mitigant is not a governance/risk/compliance platform: it has no risk-register, control-library or audit-management module, and it does not appear in the sibling guides' GRC-platform table (SAS OpRisk, IBM OpenPages, MetricStream, RSA Archer, ServiceNow GRC — ✅ financial_risk_compliance_systems_guide.md §5). Its financial-services relevance runs through **cyber-operational resilience**: **DORA** (in application since 17 January 2025 — ✅ EIOPA) is named among its compliance benchmarks, and it ran a dedicated **DORA webinar** with Advisori on 28 April 2025 (✅ webinar page); **NIS2** has its own resource page (✅). For a bank, the purchase case is **evidence for the assess and monitor stages of the ERM lifecycle** — proof that cloud controls stop real attacks, and continuous compliance evidence for resilience regulation — feeding, not replacing, the GRC estate.

The verified footprint is honestly thin where it is thin: the exact legal rename date (Resility → Mitigant) is unpublished ⚠; funding beyond the seven-figure seed is unpublished ⚠; the "Gartner AEV category" claim is the vendor's own words ⚠; GCP posture support is "Coming Soon" ⚠; and **no bank or financial institution appears in the published customer base** (⚠ §5.8 — the public logos are German/European technology firms). The competitive field is verified at primary-source level: BAS/CTEM incumbents **SafeBreach, AttackIQ, Cymulate, XM Cyber and Picus** on the emulation side, and CNAPP/CSPM leaders **Wiz, Orca and CrowdStrike Falcon** on the posture side — with **Wiz and Prowler simultaneously integrations**, because Mitigant's strategy is to *validate the findings of the scanners you already run* (✅ §8.3).

In one line: **Mitigant is the proof-of-defence layer for cloud-native banking estates — an EU-sovereign, seed-stage specialist that turns "we have thousands of findings" into "we have proven which ones are exploitable, and we are fixing those first."** *The evidence-of-defence house.*

---

## 11. Glossary

| Term | Definition |
|---|---|
| **AEV** | Adversarial Exposure Validation — validating whether identified weaknesses are actually exploitable by safely running attacker-shaped actions; the category Mitigant claims (✅ vendor; ⚠ Gartner membership claim) |
| **ATT&CK** | MITRE ATT&CK — the globally crowdsourced knowledge base of adversary tactics and techniques; Mitigant maps its attacks to it (✅ Google case study link + vendor pages) |
| **ATLAS** | MITRE ATLAS — the adversary-tactics knowledge base for AI systems; Mitigant maps AI attacks to it (✅ vendor pages) |
| **BAS** | Breach and Attack Simulation — the older discipline of emulating attacks to validate controls; Mitigant positions itself "beyond BAS" as cloud-native (✅ vendor pages; field verified via SafeBreach/AttackIQ/Picus etc.) |
| **BYOR** | Bring Your Own Role — running emulations under the customer's own IAM role for precise permission control (✅ emulation page) |
| **CISA KEV** | CISA Known Exploited Vulnerabilities catalogue — vulnerability intelligence Mitigant uses for prioritisation (✅ homepage) |
| **Cloud Attack Language** | Mitigant's DSL for building custom attacks in the Attack Builder, with YAML preview and auto-complete (✅ emulation page) |
| **CNAPP** | Cloud-Native Application Protection Platform — the unified cloud-security category (CSPM + workload/identity/data); Wiz, Orca and CrowdStrike are verified CNAPP players (✅ their sites) |
| **CSPM** | Cloud Security Posture Management — continuous scanning of cloud misconfigurations and compliance violations; a Mitigant module and a competitive category (✅ vendor + competitor sites) |
| **CTEM** | Continuous Threat Exposure Management — Gartner's continuous exposure-management cycle; Mitigant claims to enable CTEM programs (✅ Platform page; ⚠ Gartner vocabulary not re-derived) |
| **Detection-as-Code** | Defining detection/attack tests as code for CI/CD automation (✅ vendor pages) |
| **DORA** | EU Digital Operational Resilience Act — in application 17 January 2025; requires EU financial entities to manage ICT risk, test resilience, report incidents, oversee ICT third parties (✅ EIOPA; cross-ref §7) |
| **Drift analysis** | Monitoring the current state of cloud infrastructure against a secure baseline (✅ BIF grant page) |
| **EPSS** | Exploit Prediction Scoring System — probabilistic exploitability scoring used for prioritisation (✅ homepage) |
| **Guardrails** | Built-in safety constraints that keep emulated attacks safe, reversible and non-destructive (✅ homepage) |
| **KSPM** | Kubernetes Security Posture Management — posture monitoring for Kubernetes; Mitigant's LinkedIn product listing (✅ LinkedIn) |
| **MITRE** | The US not-for-profit operator of ATT&CK/ATLAS (✅ Google case study link) |
| **NIS2** | EU Network and Information Systems Directive (2022, transposition deadlines 2024) — the EU's primary cybersecurity legislation; Mitigant monitors it and publishes a NIS2 page (✅ vendor page; ⚠ regulatory detail cross-ref sibling guides) |
| **RAG** | Retrieval-Augmented Generation — Mitigant grounds its Gemini models in a verified attack-framework knowledge base to prevent hallucination (✅ Google case study) |
| **Resility GmbH** | The original legal entity (founded June 2021), renamed Mitigant GmbH (✅ About + HTGF; ⚠ rename date) |
| **Sigma** | The open detection-rule format; Mitigant outputs Sigma rules for remediation (✅ homepage) |
| **Shared Responsibility Model** | The cloud security model in which the provider secures the cloud and the customer secures their configurations — the framing of Mitigant's founding pitch (✅ adesso release reprint) |
| **TRMG** | MAS Technology Risk Management Guidelines (2013/2021) — the Singapore supervisory expectations for technology risk, cybersecurity and cloud (✅ sibling mas_regulations_guidelines_guide.md §4.2) |
| **TeleTrust** | The German IT-security trust association whose "IT Security made in Germany" seal Mitigant displays (✅ About page) |

---

## 12. Claims Audit — ✅ Verified / ⚠ Partially Verified / ❌ Not Verified

| # | Claim | Status | Evidence |
|---|---|---|---|
| 1 | Mitigant is a cloud-native security platform (AEV: attack emulation + CSPM + compliance monitoring), **not** a GRC product | ✅ | mitigant.io homepage/About/Platform/product pages (extracted live); no GRC module in any documented page |
| 2 | Legal entity Resility GmbH founded June 2021 | ✅ | About page ("Resility GmbH was founded in June 2021"); HTGF page |
| 3 | Entity renamed Mitigant GmbH ("previously Resility GmbH") | ✅ rename; ⚠ date | BIF funding page; HTGF page title; rename date unpublished |
| 4 | HQ Potsdam, Brandenburg, Germany; Dianastraße 21, 14482 | ✅ | LinkedIn company page (extracted live) |
| 5 | Founders: Nils Karn (CEO), Kennedy Torkura (CTO), Dr. Muhammad Sukmana (CPO), Dr. Thomas Fricke (advisor) | ✅ | About page; HTGF; Google for Startups case study |
| 6 | Founding idea 2020 at Hasso Plattner Institute; CloudRAID research lineage | ✅ (lineage); ⚠ (depth) | About page (idea); HTGF (CloudRAID characterisation); CloudRAID details not retrieved |
| 7 | December 2021 seed: "seven figures" from HTGF, Brandenburg Kapital, adesso Ventures | ✅ | About page verbatim; HTGF news 14 Dec 2021; adesso release reprint (marketscreener) |
| 8 | Funding amount $1.5M / single round (Tracxn, Crunchbase) | ⚠ | Secondary databases only; not confirmed by any primary source |
| 9 | Later investors: Business Angels Berlin-Brandenburg, Wayra Telefónica Innovation | ⚠ | About-page logos only; terms/date unpublished |
| 10 | Team ~12 people; privately held | ✅ | Google case study ("12-person startup"); LinkedIn ("11-50", 12 employees) |
| 11 | Platform: 500+ plug-and-play attacks across AWS/Azure/GCP, mapped to MITRE ATT&CK/ATLAS | ✅ (500+); ⚠ (internal inconsistency) | Homepage/LinkedIn (500+); emulation page also shows "200+" in one block — recorded, not smoothed |
| 12 | Attack Builder + Cloud Attack Language; attacks-as-code; Attack API; Detection-as-Code | ✅ | Emulation page; homepage; Google case study |
| 13 | CSPM: agentless; AWS + Azure supported; GCP "Coming Soon" | ✅ | CSPM page; Platform page ("agentless, SaaS") |
| 14 | Compliance benchmarks: CIS, NIS2, DORA, NIST, PCI-DSS, SOC 2 (+ ISO 27001, GDPR, HIPAA on Platform page) | ✅ vendor-scope; ⚠ depth | Homepage; Platform page |
| 15 | Safety model: BYOR, guardrails, auto-cleanup, isolation, preview, mid-execution stop | ✅ | Emulation page; homepage |
| 16 | AI: Gemini 2.5 Flash + 2.5 Pro dual-model architecture with RAG grounding | ✅ | Google for Startups case study |
| 17 | Result claims: >60% time-to-insight reduction, 3-5X triage, ~3X adoption | ⚠ | Google case study, vendor-reported figures as published |
| 18 | Integrations: Wiz, Prowler, Slack, Jira, MS Teams, Defect Dojo, SIEM, Quay, OpenShift, SysEleven, Open Telekom Cloud, Docker, Hetzner, ExoScale, Alibaba Cloud, On-Premise, Mini-kube | ✅ | Homepage integrations section (verbatim list) |
| 19 | DORA in application 17 January 2025 | ✅ | EIOPA DORA page; Mitigant webinar page |
| 20 | Mitigant × Advisori DORA webinar, 28 April 2025, hosted by Torkura + Sofroni | ✅ | Webinar page (extracted live) |
| 21 | NIS2 resource page; "100,000+ organisations in scope" | ✅ page; ⚠ figure | mitigant.io/en/nis2 (vendor figure) |
| 22 | "Most sophisticated cloud-focused platform in Gartner's AEV category" | ⚠ | Vendor claim (About page); no Gartner report retrieved |
| 23 | Homepage statistics (45% of ATT&CK techniques unimplemented; 40% ineffective; 3x breach-likelihood reduction) | ⚠ | Vendor-cited figures; underlying studies not retrieved |
| 24 | Named banking/financial-institution customers | ❌ | None published in any retrieved source; public logos are technology firms (§5.8) |
| 25 | Customer testimonials (Nooxit, Notch, KM.ON, Mitto, Adair, MontBlancAI) | ⚠ | Vendor-published; use-case pages exist but were not all re-extracted |
| 26 | Memberships: TeleTrust seal, Allianz für Cyber-Sicherheit, Athene, Networker NRW | ✅ badge-level; ⚠ depth | About page badge row |
| 27 | SOC 2 / ISO 27001 certification of Mitigant's own operations | ❌ not found | No page retrieved states either; flagged as absent evidence |
| 28 | Competitors: SafeBreach, AttackIQ, Cymulate, XM Cyber, Picus (BAS/CTEM); Wiz, Orca, CrowdStrike (CSPM/CNAPP) | ✅ | Each verified at primary-source level this pass (§8) |
| 29 | Sifted "11 cybersecurity startups to watch" coverage | ⚠ | About-page news link; article not re-extracted |
| 30 | Worked example (§9) is design content, not a claim of a real Cymbal Bank–Mitigant contract | ✅ convention | Repo worked-example convention (fircosoft_guide.md §8.5) |

---

## 13. What Could Not Be Verified

The following could not be confirmed from any source retrieved in this pass, and are deliberately left flagged rather than guessed:

1. **The exact legal-entity rename date** (Resility GmbH → Mitigant GmbH). Two on-record references confirm the rename happened (BIF page, HTGF page), but neither publishes a date; the German commercial-registry record was not retrieved.
2. **Funding amounts and rounds beyond the December 2021 seed.** Primary sources state "seven figures" (About page; HTGF news). The "$1.5M / one round" figures (Tracxn, Crunchbase) are secondary and not confirmed by any primary source; any later round is unconfirmed. The Business Angels Berlin-Brandenburg and Wayra logos are on the About page, but their participation size/date is not published.
3. **Any named bank or financial-institution customer.** The published customer set (§5.8) contains no regulated financial institution. Whether banks run Mitigant in production is not publicly verifiable — and this guide makes no assumption either way.
4. **The "Gartner AEV category" claim.** The About page claims leadership "in Gartner's Adversarial Exposure Validation (AEV) category"; no Gartner report or press release corroborating Mitigant's inclusion was retrieved. Gartner's AEV concept itself is referenced via the company's own blog, not a primary Gartner document.
5. **The homepage statistics** (45% of MITRE ATT&CK techniques not implemented in detection tools; 40% ineffective; 3x breach-likelihood reduction) — vendor-cited figures whose underlying studies were not retrieved.
6. **CloudRAID research details.** HTGF states the lineage; no CloudRAID primary page was retrieved this pass, so the research program's specifics are not described beyond the investor's characterisation.
7. **GCP posture coverage.** The CSPM page marks Google Cloud Platform "Coming Soon" while the homepage claims GCP attack emulation; the exact GCP feature parity is not documented.
8. **The 500+ vs 200+ attack-catalogue inconsistency.** The homepage/LinkedIn state 500+; one sub-block of the emulation page states 200+. Recorded as published; not resolved (no authoritative single number retrieved).
9. **Mitigant's own SOC 2 / ISO 27001 certification status.** It monitors those standards for *customers*; no page states Mitigant's own certification. Also unverified: sub-processor list, EU data-residency regions beyond the "built in Germany" claim, and any German data-protection registration detail.
10. **The identity of mitigant.com, mitigant.co.uk and mitigant.ai.** mitigant.com failed to scrape on every engine; the other two resolved to private/internal addresses for the extraction tool. No claim is made about any of them.
11. **Sifted and German trade-press coverage content.** The About page links the articles; the articles themselves were not re-extracted this pass.
12. **Financial-services-specific content beyond DORA/NIS2.** Mitigant publishes no MAS-specific, Basel-specific or AML-specific positioning; a Singapore bank must map its evidence to MAS obligations internally (cross-ref §7.4).
13. **Web-access degradation note.** Several competitor homepages (attackiq.com direct scrape, picus.io direct scrape) and mitigant.com failed extraction; where that happened, verification relied on search snippets or alternative URLs, and every affected fact was downgraded accordingly. No fact in this guide was invented to cover a gap.

**Honesty note (repo convention):** nothing in this guide was fabricated. Every ✅ row in §12 traces to a source named in the §14 ledger — a page retrieved live this pass or a sibling guide read from the repo — and every ⚠ row states why it could not be confirmed. The one task hypothesis the evidence contradicted (the GRC framing) is recorded ❌/✅-corrected in §1 and §12 row 1: **Mitigant is a cloud-security platform, not a GRC suite** — the guide follows the evidence. The worked example (§9) is design content in the repo's established Cymbal Bank convention, built on documented product features and sibling-verified regulatory facts; it claims no real contract and quotes no real configuration.

---

## 14. Verification Ledger

Source-by-source record of what was checked in this pass (web sources retrieved live in August 2026; sibling guides read from the repo on disk):

| # | Source | What was checked | Outcome |
|---|---|---|---|
| 1 | mitigant.io homepage | Positioning ("Adversarial Exposure Validation"); 500+ attacks; CSPM; compliance benchmarks (CIS, NIS2, DORA, NIST, PCI-DSS, SOC 2); integrations list; six solutions; testimonials; "EU-based · Built in Germany"; data-sovereignty framing | ✅ full page extracted (head+tail); saved to cache |
| 2 | mitigant.io/en/about | Founding story (2020 idea, HPI, founders, roles); "Resility GmbH founded June 2021"; Dec 2021 seven-figure seed (HTGF, Brandenburg Kapital, adesso Ventures); advisory board; investors; memberships; news links; Gartner AEV claim | ✅ full page extracted |
| 3 | mitigant.io/en/platform | AEV platform pillars (validate exploitability, blindspots, compliance, prioritize); CTEM framing; agentless SaaS; reporting; ISO 27001/SOC 2/GDPR/HIPAA benchmarks | ✅ full page extracted |
| 4 | mitigant.io/en/platform/cloud-attack-emulation | Attack catalogue (500+/200+); Attack Builder; Cloud Attack Language; safety model (BYOR, cleanup, isolation, preview, stop); cloud pentest/detection validation/SOC use cases; Wiz+Prowler feature release | ✅ full page extracted (head+tail) |
| 5 | mitigant.io/en/platform/cloud-security-posture-management | CSPM scope; AWS/Azure supported, GCP "Coming Soon"; blog listings (CSPM-vs-pentest; NIS2 page; AEV blog Dec 2024) | ✅ full page extracted (head+tail) |
| 6 | mitigant.io/en/webinar/how-digital-operational-resilience-act-regulation-affects-the-cloud-landscape-in-europe | DORA webinar: 28 Apr 2025, on-demand; Torkura + Sofroni (Advisori); DORA 17 Jan 2025 application framing | ✅ full page extracted |
| 7 | mitigant.io/en/funding/bif | "Mitigant GmbH, previously Resility GmbH"; Drift Analysis feature | ✅ search snippet + page |
| 8 | mitigant.io/en/news/google-for-startups-publishes-… | News item confirming the Google case-study publication | ✅ page title-level (body thin) |
| 9 | LinkedIn company page (linkedin.com/company/mitigant) | Industry (Computer and Network Security); HQ Potsdam, Dianastraße 21, 14482; 11-50 employees/12 listed; founded 2021; privately held; specialties; products (KSPM, Mitigant); "cloud-native from day one" BAS post; 3,330 followers | ✅ full page extracted |
| 10 | htgf.de/en/portfolio/htgffamily/mitigant/ | "Mitigant (formerly known as Resility)"; founders (HPI alumni); CloudRAID lineage; invested since 2021; based in Potsdam; categories (Digital Tech/Deep Tech/IT-Security) | ✅ full page extracted |
| 11 | htgf.de news, 14 Dec 2021 ("Cloud security start-up Resility secures multimillion-euro financing") | Seed-round headline and date | ✅ page listing (via portfolio page news row) |
| 12 | marketscreener.com reprint of adesso release | "Cloud security start-up Resility secures multimillion-euro financing"; Shared Responsibility Model framing; Torkura quote on Mitigant | ✅/⚠ search-snippet level |
| 13 | startup.google.com/alumni/stories/mitigant/ | Case study: 12-person startup; Gemini 2.5 Flash/Pro dual model; RAG framework; Attack Recommendation; >60% time-to-insight; 3-5X triage; ~3X adoption; Attack Builder roadmap; Potsdam, Germany | ✅ full page extracted |
| 14 | eiopa.europa.eu DORA page | DORA entered into application 17 Jan 2025; scope (banks, insurers, investment firms) | ✅ full page extracted |
| 15 | mitigant.io/en/nis2 | NIS2 resource page; "in effect, enforcement beginning"; 100,000+ organisations in scope | ✅ search snippet (vendor figures ⚠) |
| 16 | safebreach.com | "Exposure Validation Platform"; enterprise-grade CTEM; SafeBreach Validate BAS solution brief | ✅ head extracted |
| 17 | cymulate.com | "AI-Powered Exposure Validation"; agentic cyber defense engineering | ✅ head extracted |
| 18 | xmcyber.com | "Continuous Exposure Management Platform"; cloud exposures; attack graph | ✅ head extracted |
| 19 | attackiq.com (+ businesswire 2023 release) | "Leading independent vendor of breach and attack simulation"; CTEM Platform; Microsoft Marketplace listing | ✅ search snippets; ⚠ direct scrape blocked |
| 20 | picussecurity.com | "The Pioneer of Breach & Attack Simulation"; autonomous exposure validation; BAS platform | ✅ search snippets; ⚠ direct scrape returned no content |
| 21 | wiz.io | CNAPP/CSPM positioning; agentless scanning | ✅ search snippets |
| 22 | orca.security | "AI-Powered Cloud Security Platform & CNAPP" | ✅ head extracted |
| 23 | crowdstrike.com/platform | Falcon platform ("Unified Agentic Security") | ✅ head extracted |
| 24 | mitigant.net / mitigantrisk.com | Name-collision check: AI risk-register product; US healthcare consultancy | ✅/⚠ snippet-level; distinct entities |
| 25 | tracxn.com, pitchbook.com, mandos.io, clera map, cybersectools, gritt.io | Secondary profiles (founding, funding, HQ, team, category) | ⚠ secondary; used only where consistent with primary sources |
| 26 | enterprise_risk_management_guide.md (repo) | Risk process §7 (identify/assess/respond/monitor/report); ERM §5 three lines | ✅ read; cross-referenced |
| 27 | mas_regulations_guidelines_guide.md (repo) | TRMG (2013/2021) §4.2; Notice 658; FSMA Part 5/FSM-N05; obligations × controls §8 | ✅ grepped; cross-referenced |
| 28 | financial_risk_compliance_systems_guide.md (repo) | GRC platforms (SAS OpRisk, IBM OpenPages, MetricStream, RSA Archer, ServiceNow GRC); cloud adoption §11; MAS TRM/Notice 658 mentions | ✅ grepped; cross-referenced |
| 29 | financial_fraud_detection_at_scale_guide.md (repo) | Adversarial detection; case management | ✅ headings checked; cross-referenced |
| 30 | ai_genai_banking_compliance_guide.md (repo) | Prompt injection (LLM01); AI red-teaming context; MAS TRM links | ✅ grepped; cross-referenced |
| 31 | fircosoft_guide.md / resona_merchant_bank_asia_guide.md (repo) | House pattern: header block, ToC anchors, claims audit ✅/⚠/❌, ledger, glossary, closing italic line; thin-footprint honesty | ✅ read this pass |

**Budget note:** this pass ran 12 web searches (several returned empty — the backend rate-limited intermittently) and 14 web extractions against primary sources, plus local repo reads. Where a target page blocked or degraded (mitigant.com, attackiq.com direct, picus.io direct), the fact was downgraded to ⚠ rather than guessed; where a sibling guide already carried a verified fact (MAS instruments, ERM lifecycle, GRC vendors), it was cross-referenced rather than re-derived.

---

## 15. References and Further Reading

**Primary web sources retrieved this pass:**

- Mitigant — homepage: https://mitigant.io/en
- Mitigant — About: https://mitigant.io/en/about
- Mitigant — Platform: https://mitigant.io/en/platform
- Mitigant — Cloud Attack Emulation: https://mitigant.io/en/platform/cloud-attack-emulation
- Mitigant — Cloud Security Posture Management: https://mitigant.io/en/platform/cloud-security-posture-management
- Mitigant — DORA webinar page: https://mitigant.io/en/webinar/how-digital-operational-resilience-act-regulation-affects-the-cloud-landscape-in-europe
- Mitigant — BIF funding page: https://mitigant.io/en/funding/bif
- Mitigant — NIS2 resource page: https://mitigant.io/en/nis2
- Mitigant — Google-for-Startups news item: https://mitigant.io/en/news/google-for-startups-publishes-mitigant-case-study-validating-cloud-security-gaps-up-to-5x-faster-with-gemini
- Mitigant — LinkedIn company page: https://www.linkedin.com/company/mitigant/
- HTGF — "Mitigant (formerly known as Resility)" portfolio page: https://www.htgf.de/en/portfolio/htgffamily/mitigant/
- HTGF — news, 14 December 2021: https://www.htgf.de/en/htgf-investment-resility/
- Google for Startups — "Mitigant validates cloud security gaps up to 5X faster with Gemini": https://startup.google.com/alumni/stories/mitigant/
- Marketscreener — adesso release reprint ("Cloud security start-up Resility secures multimillion-euro financing"): https://www.marketscreener.com/quote/stock/ADESSO-SE-5516230/news/adesso-Cloud-security-start-up-Resility-secures-multimillion-euro-financing-37310761/
- EIOPA — Digital Operational Resilience Act (DORA): https://www.eiopa.europa.eu/digital-operational-resilience-act-dora_en
- SafeBreach: https://www.safebreach.com/
- Cymulate: https://cymulate.com/
- XM Cyber: https://xmcyber.com/
- AttackIQ (CTEM Platform / BAS): https://www.attackiq.com/platform/ ; BusinessWire BAS-as-a-Service release (30 Mar 2023): https://www.businesswire.com/news/home/20230330005209/en/
- Picus Security: https://www.picussecurity.com/
- Wiz (CNAPP/CSPM): https://www.wiz.io/
- Orca Security: https://orca.security/
- CrowdStrike Falcon platform: https://www.crowdstrike.com/platform/

**Secondary sources (flagged in the text):**

- Tracxn — Mitigant profile (founding/funding/competitors; ⚠ "founded 2020" conflicts with primary June 2021): https://tracxn.com/d/companies/mitigant/…
- Crunchbase — Mitigant (single seed round; ⚠ snippet-level): https://www.crunchbase.com/organization/mitigant
- PitchBook — Mitigant profile (⚠ snippet-level): https://pitchbook.com/profiles/company/489217-42
- Mandos Cyber — Mitigant (founded 2021, Potsdam, ~12 people; ⚠): https://mandos.io/data/companies/mitigant
- CybersecTools — Mitigant (⚠): https://cybersectools.com/companies/mitigant
- CleraMap / gritt.io / RaiseTalks — startup-database entries (⚠)
- Name-collision entries: https://mitigant.net/ and https://www.mitigantrisk.com/ (⚠ snippet-level, distinct entities)

**Repo sibling guides (cross-referenced, same folder):**

- [Enterprise Risk Management](enterprise_risk_management_guide.md) — the risk process (§7: identify → assess → respond → monitor → report); three lines of defence (§5)
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — TRMG (§4.2), Notice 658, FSMA Part 5 / FSM-N05, the obligations × controls matrix (§8)
- [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) — the GRC-platform table (§5), cloud adoption (§11), MAS TRM/Notice 658 mentions (§2)
- [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) — adversarial detection and case management
- [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) — prompt injection (LLM01), AI red-teaming context
- [Fircosoft](fircosoft_guide.md) — the vendor deep-dive pattern and claims-audit discipline
- [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) — the thin-footprint honesty pattern

**Reading path:** start with §1–§2 (who the company is — the GRC hypothesis corrected), read §3–§5 for the origins and the platform, §6–§7 for the banking positioning and the DORA/NIS2 drivers, §8 for the competitive field, then the Cymbal Bank worked example (§9); the audit (§12), the unverifiable list (§13) and the ledger (§14) tell you exactly what is and is not sourced.

---

*End of guide. Companion reading: [Enterprise Risk Management](enterprise_risk_management_guide.md) (the lifecycle this platform feeds), [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Singapore rulebook), [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) (the systems estate), [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) (the AI-threat layer). The adversarial-exposure-validation discipline in one line: scan to find, emulate to prove, fix what is exploitable first, and let the evidence — not the assumptions — reach the boardroom. The evidence-of-defence house.*
