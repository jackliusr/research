# Security-by-Design

**The Secure-Development Discipline Deep-Dive — the Definition (the Shift-Left), the Threat Modelling (STRIDE, DREAD, PASTA, Attack Trees), the Design Principles (Least Privilege, Defence-in-Depth, Fail-Secure, Secure Defaults, Separation of Duties, the Zero Trust Tie-In), the Secure Coding (OWASP Top 10 2021, ASVS, SANS 25 / CWE Top 25), the Security Testing (SAST, DAST, IAST, SCA, Pentesting), the DevSecOps (CI/CD Gates, Secret Management, SBOM, SLSA), the Cloud Security (Shared Responsibility, CSPM, CWPP), the Regulatory (DORA, NIS2, the SG CSA, MAS Notice 655), a Security-by-Design Review Worked Example, and the One-Page Summary — the Final Word: Security Is a Feature**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Security — the DEDICATED security-by-design deep-dive of the security cluster. Where [distributed_auth_guide.md](distributed_auth_guide.md) covers the *identity* angle (authentication and authorization), [api_governance_guide.md](api_governance_guide.md) the *API* angle, [llm_development_risks_security_guide.md](llm_development_risks_security_guide.md) and [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) the *GenAI/ML* angle, and [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) the *resilience* angle, this guide is the *discipline* itself: the security-by-design definition and the shift-left, the threat-modelling methods, the design principles, the secure-coding standards, the testing arsenal, the DevSecOps machinery, the cloud-security frame, and the regulatory floor. It is written to stand alone and to cross-ref the cluster heavily.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the security-by-design canon — the Microsoft STRIDE threat model (the 1999 origin, the six categories, the Shostack book *Threat Modeling: Designing for Security*, Wiley 2014), the threat-modelling methodologies (DREAD, PASTA — UcedaVélez & Morana's *Risk Centric Threat Modeling*, Addison-Wesley 2015 — and Bruce Schneier's attack trees, *Dr. Dobb's Journal*, December 1999), the classic design-principles literature (Saltzer & Schroeder, "The Protection of Information in Computer Systems", CACM 1975; Clark-Wilson 1987), the OWASP canon (the Top 10 2021, the ASVS, the SAMM), the CWE Top 25 (MITRE, 2024 edition — the successor of the SANS Top 25), the secure-coding standards (CERT/SEI), the DevSecOps practice (CI/CD security gates, secret management, the SBOM — NTIA minimum elements — and SLSA), the cloud security (the shared-responsibility model, CSPM, CWPP/CNAPP), the regulatory (EU DORA — Regulation (EU) 2022/2554, applicable 17 January 2025; EU NIS2 — Directive (EU) 2022/2555, transposition deadline 17 October 2024; the Singapore CSA — the Cybersecurity Labelling Scheme, the Safer Cyberspace Masterplan; the MAS Technology Risk Management Notice 655 — **flagged, see §8.4**), and the frameworks (ISO/IEC 27001:2022, the NIST Cybersecurity Framework 2.0 — released 26 February 2024, NIST SP 800-207 Zero Trust Architecture — August 2020). NOTE: this pass had **live web access** (self-hosted Firecrawl backend) — the key claims were verified on 2026-08-24 via web search and direct page extraction (OWASP Top 10 2021, ASVS, the CWE Top 25 2024 page, slsa.dev, the DORA tracking site, ENISA/EUR-Lex for NIS2, csrc.nist.gov for CSF 2.0, csa.gov.sg, and the MAS 2024 TRM-notice cancellation record). Anything that could not be verified is flagged ⚠ honestly.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [Distributed Auth](distributed_auth_guide.md) (**THE identity companion — authentication/authorization, OAuth2/OIDC, the authN-vs-authZ split — cross-ref §2, §4 and §9**), [API Governance](api_governance_guide.md) (**THE API companion — API security, gateways, schema governance — cross-ref §4 and §9**), [LLM Development Risks & Security](llm_development_risks_security_guide.md) (the GenAI angle — prompt injection, OWASP LLM Top 10 — cross-ref §4), [Adversarial ML Attacks](adversarial_ml_attacks_guide.md) (the ML-security angle — evasion/poisoning — light cross-ref in §4), [Zero-Downtime System Design](zero_downtime_system_design_guide.md) (**THE resilience companion — availability vs security, the fail-secure tension — cross-ref §3**), [Enterprise AI Gateway](enterprise_ai_gateway_guide.md) (the AI-governance angle — light), [Data Architect Skill Gaps](data_architect_skillgaps_guide.md) (**the standing-priority frame — Zero Trust for data platforms is an OPEN priority there — cross-ref §3.7**), [Singapore Security Clearance](singapore_security_clearance_guide.md) (the SG personnel/insider angle — light, §3.6), [OpenShift Workload Availability](openshift_workload_availability_guide.md), [Data Centres in Singapore](singapore_data_centres_guide.md) (the SG DC/DR frame — light)
> **Companion guides (banking/, prefix `../banking/`):** [RegTech](../banking/regtech_guide.md) (**THE compliance-angle companion — regulatory reporting and compliance machinery — cross-ref §8**), [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) (the op-risk angle — DORA-style resilience in the bank's risk taxonomy — cross-ref §8), [Crédit Agricole Software Systems](../banking/credit_agricole_software_systems_guide.md) (the house systems context for §9's worked example), and the bank software-systems series (HSBC, BNP Paribas, Deutsche Bank, UOB, OCBC, Bank of America...) for the bank security-estate precedents
> **Companion guides (management/, prefix `../management/`):** [Business Case Development](../management/business_case_development_guide.md) (the security-investment angle — the cost of a breach vs the cost of building it in — light cross-ref in §10)

---

**How to use this guide:** Section 1 is the overview — what security-by-design actually is, the shift-left, and the overview table (aspect / description). Section 2 is the threat modelling — STRIDE (the 1999 Microsoft six-category model), DREAD, PASTA, attack trees, and the modelling table (method / mechanics / notes). Section 3 is the design principles — least privilege, defence-in-depth, fail-secure, secure defaults, separation of duties, and the Zero Trust tie-in (cross-ref the data-architect skill-gaps standing priority), with the principles table. Section 4 is the secure coding — the OWASP Top 10 2021 (A01 broken access control), the ASVS, the SANS 25 / CWE Top 25, and the coding table. Section 5 is the security testing — SAST, DAST, IAST, SCA, pentesting, and the testing table. Section 6 is the DevSecOps — CI/CD security gates, secret management, SBOM and SLSA supply-chain security, and the DevSecOps table. Section 7 is the cloud security — the shared-responsibility model, CSPM, CWPP, and the cloud table. Section 8 is the regulatory — EU DORA (January 2025), NIS2, the SG CSA, MAS Notice 655 (flagged), and the regulatory table. Section 9 is the worked example — a security-by-design review of a Cymbal Bank application using a STRIDE-driven design review. Section 10 is the one-page summary — the final word is **security is a feature**. The glossary, the claims-status ledger and the cross-references close the file. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` and `management/` are prefixed `../banking/` and `../management/`; guides in the `ai_llm/` subdirectory are prefixed `ai_llm/`. **Integrity convention:** ✅ = verified this pass (live web) or verified in the cross-referenced guide's ledger; ⚠ = flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source verified this pass. The most important single idea of the whole discipline: **security is not a phase at the end of delivery — it is a property of the design itself, and it must be priced, planned and tested from the first line of code.**

---

## Table of Contents

1. [The Security-by-Design Overview](#1-the-security-by-design-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Definition — and the Shift-Left](#12-the-definition--and-the-shift-left)
   - 1.3 [The Overview Table — Aspect / Description](#13-the-overview-table--aspect--description)
   - 1.4 [Reading the Overview Table](#14-reading-the-overview-table)
   - 1.5 [The Canon — Where the Discipline Comes From](#15-the-canon--where-the-discipline-comes-from)
   - 1.6 [The Discipline in the Bank — What a Cymbal Bank Estate Adds](#16-the-discipline-in-the-bank--what-a-cymbal-bank-estate-adds)
2. [The Threat Modelling](#2-the-threat-modelling)
   - 2.1 [Why Model Threats at All](#21-why-model-threats-at-all)
   - 2.2 [STRIDE — the 1999 Microsoft Six-Category Model](#22-stride--the-1999-microsoft-six-category-model)
   - 2.3 [DREAD — the Risk-Rating Companion](#23-dread--the-risk-rating-companion)
   - 2.4 [PASTA — the Risk-Centric Process](#24-pasta--the-risk-centric-process)
   - 2.5 [Attack Trees — the Adversary's Decision Diagram](#25-attack-trees--the-adversarys-decision-diagram)
   - 2.6 [The Modelling Table — Method / Mechanics / Notes](#26-the-modelling-table--method--mechanics--notes)
   - 2.7 [Choosing a Method — and Combining Them](#27-choosing-a-method--and-combining-them)
   - 2.8 [The Threat Register — the Working Artifact](#28-the-threat-register--the-working-artifact)
3. [The Design Principles](#3-the-design-principles)
   - 3.1 [The Principles as a Design Language](#31-the-principles-as-a-design-language)
   - 3.2 [Least Privilege](#32-least-privilege)
   - 3.3 [Defence-in-Depth](#33-defence-in-depth)
   - 3.4 [Fail-Secure (and Fail-Open vs Fail-Closed)](#34-fail-secure-and-fail-open-vs-fail-closed)
   - 3.5 [Secure Defaults](#35-secure-defaults)
   - 3.6 [Separation of Duties](#36-separation-of-duties)
   - 3.7 [The Zero Trust Tie-In — the Standing Priority](#37-the-zero-trust-tie-in--the-standing-priority)
   - 3.8 [The Principles Table](#38-the-principles-table)
4. [The Secure Coding](#4-the-secure-coding)
   - 4.1 [From Principles to Code](#41-from-principles-to-code)
   - 4.2 [The OWASP Top 10 2021 — A01 Broken Access Control](#42-the-owasp-top-10-2021--a01-broken-access-control)
   - 4.3 [The ASVS — the Verification Standard](#43-the-asvs--the-verification-standard)
   - 4.4 [The SANS 25 / CWE Top 25 — the Weakness List](#44-the-sans-25--cwe-top-25--the-weakness-list)
   - 4.5 [The Coding Table](#45-the-coding-table)
   - 4.6 [The Mapping — Top 10 to CWE to ASVS](#46-the-mapping--top-10-to-cwe-to-asvs)
5. [The Security Testing](#5-the-security-testing)
   - 5.1 [Testing as Evidence, Not Ritual](#51-testing-as-evidence-not-ritual)
   - 5.2 [SAST — Static Application Security Testing](#52-sast--static-application-security-testing)
   - 5.3 [DAST — Dynamic Application Security Testing](#53-dast--dynamic-application-security-testing)
   - 5.4 [IAST — Interactive Application Security Testing](#54-iast--interactive-application-security-testing)
   - 5.5 [SCA — Software Composition Analysis](#55-sca--software-composition-analysis)
   - 5.6 [Pentesting — the Human Layer](#56-pentesting--the-human-layer)
   - 5.7 [The Testing Table](#57-the-testing-table)
   - 5.8 [The Security Test Pyramid — Building the Programme](#58-the-security-test-pyramid--building-the-programme)
6. [The DevSecOps](#6-the-devsecops)
   - 6.1 [Security as Code, in the Pipeline](#61-security-as-code-in-the-pipeline)
   - 6.2 [CI/CD Security Gates](#62-cicd-security-gates)
   - 6.3 [Secret Management](#63-secret-management)
   - 6.4 [The SBOM — the Software Bill of Materials](#64-the-sbom--the-software-bill-of-materials)
   - 6.5 [SLSA — Supply-chain Levels for Software Artifacts](#65-slsa--supply-chain-levels-for-software-artifacts)
   - 6.6 [The DevSecOps Table](#66-the-devsecops-table)
   - 6.7 [The DevSecOps Maturity Ramp](#67-the-devsecops-maturity-ramp)
7. [The Cloud Security](#7-the-cloud-security)
   - 7.1 [The Shared-Responsibility Model](#71-the-shared-responsibility-model)
   - 7.2 [CSPM — Cloud Security Posture Management](#72-cspm--cloud-security-posture-management)
   - 7.3 [CWPP and CNAPP — Workload Protection](#73-cwpp-and-cnapp--workload-protection)
   - 7.4 [The Cloud Table](#74-the-cloud-table)
   - 7.5 [The Cloud Decision Map — Workload to Control](#75-the-cloud-decision-map--workload-to-control)
8. [The Regulatory](#8-the-regulatory)
   - 8.1 [EU DORA — the Digital Operational Resilience Act](#81-eu-dora--the-digital-operational-resilience-act)
   - 8.2 [EU NIS2 — the Cyber-Resilience Umbrella](#82-eu-nis2--the-cyber-resilience-umbrella)
   - 8.3 [The Singapore CSA — Security-by-Design at the State Level](#83-the-singapore-csa--security-by-design-at-the-state-level)
   - 8.4 [MAS Notice 655 — Flagged: Cancelled and Superseded](#84-mas-notice-655--flagged-cancelled-and-superseded)
   - 8.5 [The Regulatory Table](#85-the-regulatory-table)
   - 8.6 [The Frameworks — ISO/IEC 27001 and NIST CSF 2.0](#86-the-frameworks--isoiec-27001-and-nist-csf-20)
   - 8.7 [The Regulatory Horizon — What Moves Next](#87-the-regulatory-horizon--what-moves-next)
9. [The Worked Example — a Security-by-Design Review](#9-the-worked-example--a-security-by-design-review)
   - 9.1 [The Scenario — a Cymbal Bank Application](#91-the-scenario--a-cymbal-bank-application)
   - 9.2 [The Review Design — the STRIDE Model Applied](#92-the-review-design--the-stride-model-applied)
   - 9.3 [The Review Output — Findings and Mitigations](#93-the-review-output--findings-and-mitigations)
   - 9.4 [The Lessons](#94-the-lessons)
   - 9.5 [The Review Runbook — the Template](#95-the-review-runbook--the-template)
10. [The One-Page Summary](#10-the-one-page-summary)
- [The Glossary](#the-glossary)
- [Claims Status and Verification Notes](#claims-status-and-verification-notes)
- [Cross-References and Further Reading](#cross-references-and-further-reading)

---

## 1. The Security-by-Design Overview

### 1.1 The Short Answer

**Security-by-design is the discipline of making security a property of the system's architecture, not a phase bolted onto delivery.** It means threats are modelled before code is written, principles (least privilege, defence-in-depth, fail-secure) are applied as design constraints, secure-coding standards are enforced in the IDE and the pipeline, testing is automated and continuous, and the regulatory floor (DORA, NIS2, the MAS notices) is treated as a design input rather than an audit surprise. The companion idea is **shift-left**: move every security activity as early in the software development lifecycle as it can possibly go — because a flaw found at design time costs orders of magnitude less than the same flaw found in production, and a vulnerability that ships is a vulnerability that gets exploited. The whole discipline is the answer to one uncomfortable truth: **in 2026, the question is not whether your application will be attacked, but whether its design survives the attack.**

### 1.2 The Definition — and the Shift-Left

The term **security-by-design** is the software-engineering application of the older engineering principle "safety by design": risk is eliminated or mitigated in the design itself, rather than controlled by procedure afterwards. In practice the definition has four layers:

1. **Design-time:** threat modelling (STRIDE, PASTA, attack trees) and architecture reviews happen before implementation — the threat model is a first-class design artifact, kept in the repo beside the ADRs. ✅-structural (universal to the threat-modelling canon, §2).
2. **Build-time:** secure-coding standards (OWASP Top 10, ASVS, CWE Top 25) are enforced by SAST/SCA in the IDE and the CI pipeline — gates, not suggestions. ✅ (§4–§6).
3. **Run-time:** the deployed system keeps failing secure, defaults secure, and verifies continuously — the Zero Trust posture (NIST SP 800-207, August 2020 ✅) where no implicit trust is ever granted. ✅-structural (§3.7).
4. **Proof-time:** testing (DAST, IAST, pentesting) and evidence (SBOMs, audit logs, incident reporting) demonstrate the properties to auditors and regulators. ✅-structural (§5, §8).

The **shift-left** slogan — "move security left" — has a precise economic and engineering meaning: the cost of fixing a defect grows roughly by an order of magnitude per lifecycle stage (design → code → test → production), so the highest-leverage security work is the work done earliest. DevSecOps is shift-left's operational form: security tooling becomes part of the pipeline (security-as-code), running on every commit, not in a quarterly review. ⚠-structural (the cost-multiplier framing is standard industry practice literature; the "shift-left" term is widely attributed to the DevOps/DevSecOps movement of the 2010s — flagged because no single canonical primary source was verified this pass).

The security-by-design discipline sits inside the broader **secure SDLC**: Microsoft's Security Development Lifecycle (SDL) — the 2000s practice that institutionalised threat modelling, security training and final security review at Microsoft, and which popularised STRIDE for the industry ⚠-structural; the OWASP SAMM (Software Assurance Maturity Model) — the framework for measuring and improving the maturity of a secure-development programme ✅-canon (OWASP flagship, §4.5); and the industry frameworks of §8.6 (ISO/IEC 27001:2022 and the NIST CSF 2.0) that tell an organisation *what* to manage, while the discipline of this guide tells a team *how* to build.

### 1.3 The Overview Table — Aspect / Description

| Aspect | Description |
|---|---|
| **Definition** | Security is designed into the architecture from the start — threat-modelled, principled, tested, and regulated — rather than added at the end (§1.1–1.2) |
| **The shift-left** | Moving security activities as early as possible in the SDLC; the cost-of-fix curve makes design-time the cheapest place to be secure (§1.2) |
| **Threat modelling** | STRIDE (1999 Microsoft), DREAD, PASTA, attack trees — the systematic identification of what can go wrong before it does (§2) |
| **Design principles** | Least privilege, defence-in-depth, fail-secure, secure defaults, separation of duties — the architectural constraints (§3) |
| **Zero Trust tie-in** | Never trust, always verify — the standing priority cross-referenced from the data-architect skill gaps; NIST SP 800-207 (2020) (§3.7) |
| **Secure coding** | OWASP Top 10 2021 (A01 broken access control), the ASVS levels, the SANS 25 / CWE Top 25 — the codified do-this/don't-do-this (§4) |
| **Security testing** | SAST, DAST, IAST, SCA, pentesting — the evidence layers from source to runtime (§5) |
| **DevSecOps** | CI/CD security gates, secret management, SBOM and SLSA supply-chain security — security as code (§6) |
| **Cloud security** | The shared-responsibility model, CSPM, CWPP/CNAPP — the boundary between provider and tenant (§7) |
| **Regulatory** | EU DORA (applicable 17 January 2025), NIS2 (transposition by 17 October 2024), the SG CSA, MAS Notice 655 (cancelled May 2024, superseded by FSM-N05) (§8) |
| **Frameworks** | ISO/IEC 27001:2022, the NIST CSF 2.0 (26 February 2024) — the governance containers (§8.6) |
| **The worked example** | A STRIDE-driven security-by-design review of a Cymbal Bank application — the discipline end-to-end (§9) |
| **The final word** | **Security is a feature** — priced, planned and tested like any other feature, or it is a tax paid in incidents (§10) |

### 1.4 Reading the Overview Table

The table is the whole discipline on one page: the **left column** is the layer of the discipline, the **right column** the one-line essence. Read it top-to-bottom as a lifecycle — define it, model it, design it, code it, test it, automate it, host it, regulate it, prove it — and then read §9 to see the whole chain applied to one system. Two rows deserve special attention because they are the ones teams skip: **threat modelling** (skipped because "we don't have time" — the exact activity that would have found the flaw) and **regulatory** (treated as an audit concern instead of a design input — the exact row that turns into a regulator's finding in a DORA or MAS examination).

### 1.5 The Canon — Where the Discipline Comes From

The discipline has a documented lineage, and the dates matter for interviews and for trusting the material:

- **1975 — Saltzer & Schroeder**, "The Protection of Information in Computer Systems" (CACM 18(7)): the eight design principles — economy of mechanism, fail-safe defaults, complete mediation, open design, separation of privilege, least privilege, least common mechanism, psychological acceptability. ✅-canon (the foundational paper; §3 builds on it).
- **1987 — Clark-Wilson**: the formal model of well-formed transactions and separation of duty — the root of modern four-eyes controls in banking. ✅-canon (§3.6).
- **1999 — Microsoft STRIDE** (the Kohnfelder & Garg "Threats to Our Products" memo) and **Schneier's attack trees** (*Dr. Dobb's Journal*, December 1999, ✅ verified). The threat-modelling canon is born the same year. ✅ (§2.2, §2.5).
- **2000s — the Microsoft SDL** institutionalises threat modelling; **DREAD** appears as the Microsoft risk-rating model (documented in the "Threats and Countermeasures" guidance, Meier et al., 2003 ✅-structural). (§2.3.)
- **2010s — shift-left and DevSecOps** (the DevOps movement's security wing); **PASTA** (UcedaVélez & Morana, VerSprite; the *Risk Centric Threat Modeling* book, 2015 ✅-structural) brings risk-centrism; **Shostack's** *Threat Modeling: Designing for Security* (Wiley, 2014 ✅-canon) makes the practice approachable.
- **2020s — the supply chain and the regulators arrive**: NIST SP 800-207 Zero Trust (August 2020 ✅-canon), EO 14028 and the SBOM mandate (2021 ✅-canon), SLSA (2021–2023 ✅ verified), the OWASP Top 10 2021 ✅, and the regulatory wave — DORA and NIS2 ✅ verified live.

**The pattern to notice:** every decade has added a *layer* — principles (1970s), threat modelling (1990s), process (2000s), automation (2010s), supply chain and regulation (2020s). A modern secure-SDLC programme must contain all five layers; that is what the rest of this guide assembles.

### 1.6 The Discipline in the Bank — What a Cymbal Bank Estate Adds

Security-by-design in a bank is the same discipline with three extra weights attached. **First, the asset mix:** a corporate-and-investment-bank estate carries payments (where *tampering* and *repudiation* dominate — the STRIDE T and R categories of §2.2), trade finance (documentary instruments where authenticity is the product — cross-ref [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md)), client confidential data (where *information disclosure* is a PDPA/GDPR event, not just a bug — cross-ref the data-governance guides), and market positions (where *confidentiality* and *integrity* are priced in money). **Second, the control environment:** maker-checker separation of duties (§3.6) is not a nice-to-have but a supervisory expectation, the audit trail must survive an examination (DORA's evidence chain, §8.1), and the third-party estate — vendors, SaaS, cloud providers, the AI platforms of [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) — is regulated as supply-chain risk (the SBOM/SLSA agenda of §6.4–6.5 meeting the DORA Register of Information). **Third, the estate's own shape:** a Cymbal Bank environment is a federation of the very old (core banking systems and the AS/400-era platforms — cross-ref [ibm_as400_guide.md](ibm_as400_guide.md)) and the very new (cloud, APIs, GenAI), which means the threat model must be drawn *per trust zone* — the legacy core, the API layer, the cloud data plane and the AI surface each get their own DFD and their own STRIDE pass (§2.2). The discipline does not change; the stakes, the scrutiny and the number of trust boundaries do. ⚠-structural (the banking-framing synthesis is this guide's own; every element is cross-ref'd to verified cluster material).

---

## 2. The Threat Modelling

### 2.1 Why Model Threats at All

Threat modelling is the systematic identification of **what can go wrong with this specific system** — not generic "hackers might attack us" anxiety. The output is a list of concrete, ranked threats, each with a mitigation, owned and tracked. It is the design-time activity that makes every later layer (coding standards, testing, gates) *targeted*: you cannot test for the threats you never named. The canonical framing (Shostack, 2014) is four questions: **What are we building? What can go wrong? What are we going to do about it? Did we do a good job?** ✅-canon. Threat modelling is also a regulatory expectation: DORA's ICT risk management (Article 9) and the MAS notices both require risk identification and assessment as a formal control ✅-structural (§8).

### 2.2 STRIDE — the 1999 Microsoft Six-Category Model

**STRIDE** is the threat-classification model developed at Microsoft (introduced in 1999 with the Kohnfelder & Garg memo, and popularised through the Security Development Lifecycle and Shostack's book). It names six threat categories, each of which violates a specific security property:

| Category | Violated property | The attacker... | Typical examples |
|---|---|---|---|
| **S**poofing | Authenticity | Pretends to be someone/something else | Forged JWT, stolen session, spoofed caller ID, fake API client identity |
| **T**ampering | Integrity | Modifies data or code in transit or at rest | Man-in-the-middle of a payment message, altered database row, tampered container image |
| **R**epudiation | Non-repudiation | Denies having performed an action | Trader denies placing an order; no audit trail, or the trail is deletable |
| **I**nformation disclosure | Confidentiality | Reads data they should not see | IDOR on an account statement API, data leak in logs, over-broad API response |
| **D**enial of service | Availability | Prevents legitimate use | Resource exhaustion, unauthenticated expensive endpoint, missing rate limiting |
| **E**levation of privilege | Authorization | Gains rights they should not have | Horizontal/vertical privilege escalation, RCE chained from injection |

✅ Verified this pass (Wikipedia STRIDE model; multiple practitioner sources; the six categories are uniform across all sources). The **1999** date and the **Microsoft** attribution: ✅-verified (devops-daily's threat-modelling series: "developed at Microsoft in the late 1990s"; the Kohnfelder & Garg April 1999 memo is the documented origin — ⚠-structural for the memo-level attribution, uniformly cited in the literature). STRIDE is applied **per element** of a data-flow diagram (DFD): for each external entity, process, data store and data flow, ask which of the six categories apply. It is a *discovery* model — it finds threats; it does not rank them. That is DREAD's job. Cross-ref [distributed_auth_guide.md](distributed_auth_guide.md) for the spoofing/elevation deep-dive (the identity angle), and [api_governance_guide.md](api_governance_guide.md) for the API-surface version (every STRIDE category has an API flavour).

### 2.3 DREAD — the Risk-Rating Companion

**DREAD** is the Microsoft risk-rating model that pairs with STRIDE: it scores each identified threat on five dimensions to produce a comparable risk number. ✅ Verified this pass (Wikipedia, threat-modeling.com, EC-Council). The five dimensions:

- **D**amage potential — how much harm if exploited
- **R**eproducibility — how reliably the attack works
- **E**xploitability — how much effort/skill the attacker needs
- **A**ffected users — how many users/systems are impacted
- **D**iscoverability — how easy the vulnerability is to find

Each dimension is rated on a numeric scale (the common formulations use 0–10 or 1–10; ⚠-structural — sources differ on the exact scale, Wikipedia documents 1–10), and the scores are summed or averaged to rank threats for remediation. The model is documented in Microsoft's "Threats and Countermeasures" guidance (Meier et al., 2003, ✅-structural). **Honest caveat:** Microsoft itself moved away from DREAD's formal scoring (its later SDL guidance uses a custom criticality table — ⚠-structural), and DREAD is criticised for the subjectivity of its scores — but as a *prioritisation heuristic* it remains the most widely taught risk-scoring method, and it is exactly what a design review needs to decide "fix now vs track". ⚠ on any claim that DREAD is "the official Microsoft standard today" — it is a documented legacy model.

### 2.4 PASTA — the Risk-Centric Process

**PASTA** — the **P**rocess for **A**ttack **S**imulation and **T**hreat **A**nalysis — is the risk-centric threat-modelling methodology created by **Tony UcedaVélez and Marco M. Morana** of **VerSprite**, documented in *Risk Centric Threat Modeling* (Addison-Wesley, 2015). ✅ Verified this pass (threat-modeling.com: the acronym, the creators, the VerSprite attribution, the risk-centric character; the seven-stage structure confirmed). Where STRIDE is a classification lens applied to a DFD, PASTA is a full *process* that walks seven stages from business objectives to risk decisions:

1. **Define objectives** — business impact analysis (BIA): what the application's loss scenarios are worth.
2. **Define technical scope** — the attack surface: components, boundaries, technologies.
3. **Application decomposition** — the data flows, trust boundaries, entry points (DFD territory).
4. **Threat analysis** — enumerate threat agents and threat scenarios (STRIDE-style classification is often used here).
5. **Vulnerability and weakness analysis** — map to weaknesses (CWE), correlate with controls.
6. **Attack modeling** — attack trees (Schneier), attack patterns (CAPEC), simulation of the attack paths.
7. **Risk and impact analysis** — score and prioritise residual risk against the business objectives of stage 1.

⚠-structural for the exact wording of the seven stage names (sources vary slightly in phrasing); the seven-stage structure itself is ✅ verified. PASTA's value for a bank: it **starts from business impact**, so the output is a risk decision a business stakeholder can own — the natural fit for a Cymbal Bank application where the "what is it worth if this breaks" question is answered in money. It is heavier than STRIDE; teams typically use STRIDE for routine design reviews and PASTA for high-value or high-risk systems.

### 2.5 Attack Trees — the Adversary's Decision Diagram

**Attack trees** are Bruce Schneier's formal structure (introduced in "Attack Trees", *Dr. Dobb's Journal*, **December 1999** — ✅ verified live, schneier.com archives the original article) for describing the ways an attacker can reach a goal: the **goal is the root node**, the **ways of achieving it are the children**, connected by AND/OR logic — an OR node is any-one-path, an AND node is all-sub-steps. Leaves can carry attributes (cost, difficulty, skill, detection probability) so that the tree supports quantitative analysis of which attack paths are cheapest/most likely. ✅ Verified (the original article: "represent attacks against a system in a tree structure, with the goal as the root node and different ways of achieving that goal as leaf nodes"). Attack trees are the *micro* layer of threat modelling: STRIDE says *what category*, the attack tree says *exactly how*, stage 6 of PASTA operationalises them, and the classic banking use is a **fraud kill-chain diagram** — the tree for "steal a payment" enumerates every branch (compromise credentials, hijack session, tamper with message, social-engineer the operator...) and each branch becomes a control requirement. Cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) for the mirror-image discipline — fault trees for availability; attack trees are fault trees with an adversary ✅-structural.

### 2.6 The Modelling Table — Method / Mechanics / Notes

| Method | Mechanics | Notes |
|---|---|---|
| **STRIDE** (1999, Microsoft) ✅ | Apply the six categories per DFD element (entity, process, store, flow); produce a threat list per element | The discovery workhorse — fast, teachable, tool-supported; finds threats, does not rank them (§2.2) |
| **DREAD** ✅ | Score each threat 1–10 (or 0–10 ⚠) on Damage, Reproducibility, Exploitability, Affected users, Discoverability; sum/average to rank | The ranking companion to STRIDE; subjective scores — calibrate with the team; legacy at Microsoft ⚠-structural (§2.3) |
| **PASTA** ✅ | Seven stages: objectives → technical scope → decomposition → threat analysis → vulnerability analysis → attack modelling → risk decision | Risk-centric and business-impact-led (VerSprite/UcedaVélez & Morana); heavyweight — for high-value systems (§2.4) |
| **Attack trees** ✅ | Goal as root, attack paths as AND/OR branches, attributes (cost/difficulty) on leaves; compute cheapest paths | Schneier 1999; the "exactly how" layer; feeds fraud kill-chains and PASTA stage 6 (§2.5) |
| **LINDDUN** ⚠-structural | Privacy-oriented STRIDE: Linking, Identifying, Non-repudiation, Detecting, Disclosure, Unawareness, Non-compliance | The privacy companion — cross-ref the PDPA/GDPR angle in the data-governance guides; not verified this pass (§8) |
| **OWASP Threat Dragon / Microsoft Threat Modeling Tool** ⚠-structural | DFD drawing with automated STRIDE suggestion per element | The tooling layer — free, standard; the tool output is only as good as the DFD |

### 2.7 Choosing a Method — and Combining Them

There is no single right method; the practice is to **combine**: STRIDE for completeness at every design review (cheap, repeatable, catches the category you forgot), attack trees when a specific high-value goal needs the full path analysis (payments, entitlements, admin surfaces), DREAD (or a calibrated variant) to turn the findings into a prioritised backlog, and PASTA for the crown-jewel systems where business impact must drive the decision. The **output contract** is the same for all: a threat register — each row is a threat (category, scenario, affected assets, likelihood, impact, mitigation, owner, status) — tracked in the same tooling as the rest of the backlog, reviewed whenever the architecture changes (new integration, new API, new data flow). A threat model that is not updated when the design changes is a historical document, not a control. ⚠-structural (the combination pattern is standard practice, not a single-source doctrine).


### 2.8 The Threat Register — the Working Artifact

The deliverable of every threat-modelling method is the **threat register**: a living, tracked list of threats, each with enough structure to be actioned, prioritised and audited. The canonical columns — and the row shape used in §9's worked example:

| Column | What goes in it |
|---|---|
| **ID** | Stable identifier (TM-001...) — the audit trail's key |
| **Threat** | One sentence: the concrete bad thing ("attacker replays a captured payment request") |
| **Category** | STRIDE letter(s) — and/or the attack-tree path if one was drawn |
| **Affected assets** | The systems/data at risk — the DFD elements |
| **Score** | DREAD (or calibrated variant) — the prioritisation number |
| **Mitigation** | The *design* response (a control, a principle application, an architecture change) |
| **Owner** | A named person — an unowned threat is a non-threat |
| **Status** | Open / mitigated / accepted (with justification) / verified |
| **Due / verified date** | The cadence discipline |

The working rules (⚠-structural, standard practice): the register lives **in the same tracker as the delivery backlog** (an issue per threat — no separate spreadsheet that drifts); it is **re-opened on every material design change** (new integration, new data flow, new API — a threat model that predates the last architecture change is a historical document); it is **re-scored at least quarterly** (threats move — the DORA/NIS2 incident landscape of §8 changes likelihood); and **"accepted" is a written risk decision with an owner and a review date**, never a silent drop. A healthy register for a mid-size application is a few dozen rows, of which the top five get the team's attention this sprint. The register is also the *evidence artifact* the audit side reads — the same rows, re-scored and statused, are what a DORA or FSM-N05 examination samples (§8), and §9.5's runbook treats the register as the session's output contract.

---

## 3. The Design Principles

### 3.1 The Principles as a Design Language

The design principles are the architectural grammar of security: when an architect says "the entitlements service runs with least privilege, behind a fail-secure gateway, with separation of duties on the approval flow", they are making security *decisions* the way they make availability and scalability decisions. The five core principles below all trace to the foundational literature — Saltzer & Schroeder's 1975 eight principles (fail-safe defaults, least privilege, separation of privilege among them — ✅-canon) and Clark-Wilson's 1987 separation-of-duty model (✅-canon) — and all five are universally echoed in every modern framework (NIST CSF 2.0's Protect function, ISO 27001 Annex A, the OWASP design guidance). ✅-structural for the framework mappings; the principles themselves are ✅-canon. Two cross-cutting notes: the principles are **composable** (defence-in-depth *is* the layering of the others), and they **conflict** (least privilege vs availability — the fail-secure tension with [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) — so they are applied by judgment, not by rote).

### 3.2 Least Privilege

**Every subject (user, service, process, token) gets the minimum permissions required to do its job — no more.** Not "the developer role is admin by default", not "the batch job runs as the service account with DBA rights". Concretely: fine-grained roles (RBAC) and attribute-based rules (ABAC) rather than shared superuser accounts; scoped service identities and short-lived credentials; row/column-level security in the data layer; and the principle extended to *time* (privilege for the duration of the task) and *blast radius* (a compromised batch job should be able to touch exactly one queue, not the whole estate). ✅-canon (Saltzer & Schroeder list it as a core principle; it is universal across the frameworks). In the Cymbal Bank context this is the difference between an entitlements model that survives an audit and one that produces the classic finding "N users hold the maker role they never use". Cross-ref [distributed_auth_guide.md](distributed_auth_guide.md) for the identity-side machinery (OAuth scopes, OIDC claims, fine-grained authorization) and the data-architect skill-gaps guide for the data-plane version (row/column-level security, dynamic masking).

### 3.3 Defence-in-Depth

**No single control is trusted; security is layered so that the failure of one layer is caught by the next.** The canonical stack: network (segmentation, WAF) → host (hardening, EDR) → application (authentication, authorization, input validation) → data (encryption at rest and in transit, tokenization) → people (training, separation of duties) → monitoring (detection and response). Each layer must be *independent* — the point is that an attacker who defeats the WAF still meets the application controls, and one who exploits the app still meets the data-layer encryption. ✅-canon (defence-in-depth is doctrine across NIST, ISO, and every vendor framework; the layered-stack formulation is ⚠-structural in its exact layer list). The design question is always "**if this control fails, what still holds?**" — the answer is the depth. Cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) for the mirror-image framing in availability (redundancy layers are defence-in-depth for uptime).

### 3.4 Fail-Secure (and Fail-Open vs Fail-Closed)

**When a component fails, it must fail in the secure direction — deny, block, close, degrade — never in the permissive direction.** The canonical pair: a firewall that crashes must drop traffic (fail-closed), not pass it (fail-open); an authorization service that times out must deny the request; a broken TLS handshake must abort the connection. The Saltzer & Schroeder formulation is "**fail-safe defaults**": base access decisions on the *denied* default, and grant access only on explicit evidence ✅-canon. The hard tension is with availability: fail-closed protects security but can take down the service (the auth outage = total outage problem), which is why the design decision must be explicit and recorded — *this* component fails closed because the confidentiality cost of failing open is higher, *that* one fails open for a bounded, monitored window because the availability cost is higher (e.g. degraded read-only mode). ⚠-structural (the fail-open/fail-closed trade-off framing is universal practice literature). Cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §8 — graceful degradation is the availability half of this decision.

### 3.5 Secure Defaults

**The out-of-the-box configuration must be the secure configuration; security is opt-*out* for justified exceptions, never opt-in.** Secure-by-default is the operational translation of the principles: new users start with minimal roles; new services start with encryption on; debug endpoints start disabled; default passwords are banned (and, better, there are no default credentials at all); CI templates start with the security gates enabled. The OWASP Top 10 2021 makes the failure mode visible: **A05 Security Misconfiguration** — the class of vulnerability that exists purely because defaults were permissive (default credentials, verbose error pages, enabled debug modes, unhardened headers) ✅ verified (§4.2). Secure defaults are also a *product* requirement in the 2020s — the CISA "secure by design" push for software vendors (⚠-structural, CISA's 2023 "Shifting the Balance of Cybersecurity Risk" campaign is the widely-cited origin) and Singapore's CSA OT Masterplan 2024 explicitly promote secure-by-design/secure-by-deployment across the lifecycle ✅ verified (§8.3).

### 3.6 Separation of Duties

**No single individual or system component should be able to complete a critical action alone; conflicting functions are split so that fraud requires collusion.** In banking this is the **maker-checker** pattern — the person who creates a payment cannot be the person who approves it — and it extends to the technical estate: the developer who writes code does not deploy it to production (or the deploy path is gated by a different identity); the operator who runs the batch does not also audit it; the admin who grants roles does not hold the role they grant. The formal root is Clark-Wilson's separation-of-duty model (1987, ✅-canon); the practical root is that the *largest* insider-threat and fraud controls in banking are SoD controls, not technical ones. For a Cymbal Bank application the design artifacts are: a **SoD matrix** mapping roles to mutually exclusive rights, dual-control workflows for payments/entitlements/parameter changes, and audit trails that make the split verifiable (a maker-checker that shares one session is a checkbox, not a control). Cross-ref [singapore_security_clearance_guide.md](singapore_security_clearance_guide.md) for the SG personnel-clearance angle and the financial-risk compliance systems guide for the operational-risk framing of SoD failures.

### 3.7 The Zero Trust Tie-In — the Standing Priority

**Zero Trust is the 2020s operating model that the design principles converge on: "never trust, always verify" — no implicit trust based on network location, every request authenticated, authorized, and encrypted, access decisions made continuously with least privilege, and the blast radius contained by micro-segmentation.** The canonical source is **NIST SP 800-207, *Zero Trust Architecture*, published August 2020** ✅-canon (and ✅ cross-ref — the [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) standing priority #3 is precisely "Zero Trust for data platforms: NIST SP 800-207 + data-plane ZTA (micro-segmentation, identity-aware data access, MAS TRM/N645 alignment)", flagged there as a genuine repo gap — this section is the tie-in that priority calls for). The relationship to this guide is direct: **every principle in §3.2–§3.6 is a Zero Trust pillar in miniature** — least privilege is the access-decision engine, defence-in-depth becomes the segmented network/data plane, fail-secure is the "deny by default" posture, secure defaults are the policy-as-code starting point, and separation of duties is the human-side continuous verification. The Zero Trust pillars (identity, device, network, application, data — per SP 800-207 and CISA's Zero Trust Maturity Model ⚠-structural for the pillar-list wording) map onto the architecture: identity-aware proxies in front of every service (cross-ref [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) for the AI-gateway variant), policy decision points that re-verify on every request rather than once per session, and data-plane controls (encryption, masking, row-level security) that protect the data even when the network is hostile. In Singapore banking the alignment is explicit — MAS's TRM notices and Notice 645 (cyber hygiene) push the same continuous-verification posture ✅-structural (cross-ref the skill-gaps guide's own ledger). **The one-line summary for the repo: Zero Trust is not an alternative to security-by-design — it is security-by-design's runtime posture, and the skill-gaps guide's standing priority #3 is exactly this section.**

### 3.8 The Principles Table

| Principle | Essence | Canon | Fails when... |
|---|---|---|---|
| **Least privilege** | Minimum permissions, minimum time, minimum blast radius | Saltzer & Schroeder 1975 ✅ | Shared superuser accounts; roles granted "because it's easier" (§3.2) |
| **Defence-in-depth** | Independent, layered controls; no single point of trust | Universal framework doctrine ✅ | One layer (usually the WAF) treated as "the security" (§3.3) |
| **Fail-secure** | Fail closed; deny by default; explicit, recorded fail-open exceptions | Saltzer & Schroeder "fail-safe defaults" ✅ | Timeouts that pass requests through; crash-open firewalls (§3.4) |
| **Secure defaults** | Out-of-the-box = secure; opt-out with justification | OWASP A05 mirror ✅ | Default credentials; debug modes enabled; verbose errors (§3.5) |
| **Separation of duties** | Maker-checker; conflicting rights split; collusion required | Clark-Wilson 1987 ✅ | One role creates and approves; shared sessions (§3.6) |
| **Zero Trust** | Never trust, always verify; continuous authorization; micro-segmentation | NIST SP 800-207 (Aug 2020) ✅ | Perimeter-only thinking; implicit trust inside the network (§3.7) |
| **Economy of mechanism** | Keep the security surface simple — complexity is the attacker's friend | Saltzer & Schroeder 1975 ✅ | Bespoke crypto, hand-rolled auth, sprawling entitlements logic |
| **Complete mediation** | Every access checked, every time — no cached/bypassed paths | Saltzer & Schroeder 1975 ✅ | "We checked at login, so the session can do anything" (§2.2's E) |

---

## 4. The Secure Coding

### 4.1 From Principles to Code

The principles say *how to think*; the secure-coding standards say *what to write*. The three documents that dominate the field — the **OWASP Top 10 2021** (the awareness list), the **OWASP ASVS** (the verification standard), and the **CWE Top 25** (the weakness list, successor of the **SANS Top 25**) — are complementary: Top 10 tells management and developers *what the risk classes are*, ASVS tells reviewers *what to verify at each level*, and CWE gives every weakness a stable identifier so findings can be tracked, aggregated and measured (CWE identifiers are what SAST tools, vulnerability feeds and the CVE ecosystem all speak). A mature team keeps all three: Top 10 for training and backlog framing, ASVS as the acceptance criteria for security requirements, CWE as the taxonomy in the defect tracker. The CERT/SEI secure-coding standards (CERT C, CERT C++, CERT Java — ✅-canon) fill the same role for systems languages where memory-safety classes dominate (the CWE Top 25's #2, out-of-bounds write, is the C/C++ classic ✅ verified).

### 4.2 The OWASP Top 10 2021 — A01 Broken Access Control

The **OWASP Top 10 2021** is the community's risk-based ranking of web-application security risks, produced by the OWASP Foundation from contributed data. ✅ Verified this pass (owasp.org/Top10/2021). The 2021 edition's headline: **A01 Broken Access Control** — moved up from position 5 to the top — **94% of applications were tested for some form of broken access control, with an average incidence rate of 3.81%, and over 318,000 occurrences in the contributed dataset** (the most of any category). ✅ (owasp.org A01 page). Access control is the *authorization* layer — "can this user do this thing" — and its failure modes are the design-principle violations of §3 in code form: missing enforcement on a function, IDOR (insecure direct object references — CWE-639), path traversal (CWE-22), privilege escalation, CORS misconfiguration, and forcing browsing to authenticated pages as an unauthenticated user. The full 2021 list (✅ verified from owasp.org navigation and the A01 page):

1. **A01 — Broken Access Control** (the #1 risk; authorization failures — cross-ref [distributed_auth_guide.md](distributed_auth_guide.md) for the identity machinery)
2. **A02 — Cryptographic Failures** (weak or absent encryption; the renamed "sensitive data exposure")
3. **A03 — Injection** (SQL/NoSQL/OS/command injection; still top-3 after two decades)
4. **A04 — Insecure Design** (the *new* category for 2021 — missing design-level controls: the threat-modelling gap of §2 as a risk class; ✅ the "Insecure Design" category was added in the 2021 edition)
5. **A05 — Security Misconfiguration** (defaults, headers, verbose errors — the secure-defaults failure of §3.5)
6. **A06 — Vulnerable and Outdated Components** (the SCA problem of §5.5 and the SBOM of §6.4)
7. **A07 — Identification and Authentication Failures** (broken authN — the spoofing category of §2.2)
8. **A08 — Software and Data Integrity Failures** (the *new* supply-chain category — CI/CD compromise, insecure deserialization, unsigned updates — the SLSA problem of §6.5)
9. **A09 — Security Logging and Monitoring Failures** (you cannot respond to what you did not record — the repudiation category)
10. **A10 — Server-Side Request Forgery (SSRF)** (the *new* 2021 entry — the cloud-era "ask the server to fetch the metadata endpoint" attack)

**The design lesson for a Cymbal Bank application:** A01 being #1 is not an accident — authorization is where design meets data, and every integration (API, batch, UI, report) multiplies the places where a missing check leaks money or client data. The fix is architectural, not stylistic: a single authorization decision point (policy as code, cross-ref [api_governance_guide.md](api_governance_guide.md)), denied-by-default enforcement, and object-level checks on every data access.

### 4.3 The ASVS — the Verification Standard

The **OWASP Application Security Verification Standard (ASVS)** is the open standard for what must be *verified* in a web application: it is a checklist of security requirements organised into chapters (in v4: 14 chapters, V1 architecture through V14 configuration — ⚠-structural for the exact chapter count of the current release), with three **verification levels** that scale with the application's risk:

- **Level 1 (L1)** — for applications with low business risk: the requirements that are automatable and discoverable by opportunistic attackers — the "no low-hanging fruit" bar.
- **Level 2 (L2)** — for applications handling sensitive data or regulated functions: *most* controls, including those needing human review — the bar most banking applications must meet.
- **Level 3 (L3)** — for the crown jewels: *all* controls plus defence-in-depth design verification — high-value/high-risk systems, payment rails, privileged administration.

✅ Verified this pass (owasp.org ASVS project page; the 4.0.3 PDF — the version line "ASVS 1.0 in 2008 to 3.0 in 2016" confirms the lineage; the level structure is uniform across the sources; note the ASVS 5.0 release in 2025 is ⚠ — mentioned by OWASP community sources but not verified this pass). ASVS's practical role in the secure SDLC: **it is the contract between the security team and the delivery team** — "this application will meet ASVS L2" is a precise, testable requirement that SAST/DAST/IAST evidence (a *verification* standard, hence the name) can be mapped against, and it is the standard that pentest scopes and acceptance criteria are written from. Cross-ref the OWASP SAMM for the programme-level maturity counterpart (⚠-canon, §4.5 table row).

### 4.4 The SANS 25 / CWE Top 25 — the Weakness List

**The SANS Top 25** was the joint **SANS Institute/MITRE** ranking of the most dangerous software weaknesses (the collaboration ran through the late 2000s–2011; ⚠-structural for exact edition years), based on the **CWE** taxonomy. **The naming nuance to get right:** the list lives on today as the **MITRE CWE Top 25**, published annually by MITRE (sponsored by DHS/CISA) — the **2024 edition** was verified live this pass (cwe.mitre.org): #1 is **CWE-79 Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')**, #2 **CWE-787 Out-of-bounds Write**, #3 **CWE-89 SQL Injection**, #4 **CWE-352 Cross-Site Request Forgery** — and the page carries KEV (Known Exploited Vulnerabilities) counts per weakness, e.g. 18 of the out-of-bounds-write CVEs are in CISA's KEV catalogue, which is precisely why the list matters: **these are the weaknesses actually being exploited, not theoretical risks.** ✅ (cwe.mitre.org/2024 verified live; the "SANS 25 → CWE Top 25" handover is ⚠-structural — the SANS/MITRE joint list and the modern MITRE-run annual list are the same lineage, uniformly described, but the exact handover year was not verified this pass). For the coding table below, the top of the 2024 list (CWE-79 XSS, CWE-787 OOB write, CWE-89 SQLi, CWE-352 CSRF — ✅) is what SAST rulesets and training curricula are built from.

### 4.5 The Coding Table

| Standard | What it is | Mechanics | Notes |
|---|---|---|---|
| **OWASP Top 10 2021** ✅ | The risk-based awareness list of web-app risk classes | A01–A10: broken access control, crypto failures, injection, insecure design, misconfig, vulnerable components, authN failures, integrity failures, logging gaps, SSRF | A01 is #1: 94% of apps tested, 318k+ occurrences ✅; A04/A08/A10 are the 2021 additions — design, supply chain, SSRF (§4.2) |
| **ASVS** ✅ | The verification standard — checkable security requirements | 14 chapters (v4, ⚠ count), 3 levels: L1 automatable, L2 regulated-data bar, L3 crown jewels | The acceptance-criteria contract; what pentests are scoped from; v5.0 release ⚠ (§4.3) |
| **SANS 25 / CWE Top 25** ✅ | The exploited-weakness ranking (MITRE, 2024) | Annual list from CVE/KEV data; CWE identifiers as the shared taxonomy | CWE-79 XSS, CWE-787 OOB write, CWE-89 SQLi, CWE-352 CSRF at the top ✅; SAST rulesets map to it (§4.4) |
| **OWASP SAMM** ⚠-canon | The secure-development *maturity* model | 15 practices across business/design/implementation/verification/operations functions, maturity levels 0–3 | The programme-level counterpart to ASVS; measure, then improve (§1.2) |
| **CERT/SEI standards** ✅-canon | Language-specific secure-coding rules (C, C++, Java) | Rule/convention pairs with severity; the C-family answer to CWE-787 | For systems code — banking rarely writes C, but vendors' code is in scope |
| **OWASP LLM Top 10 / ML-specific** ✅-cross-ref | The GenAI risk list (prompt injection #1) | Cross-ref [llm_development_risks_security_guide.md](llm_development_risks_security_guide.md) and [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) | The 2025-era extension of the coding table — GenAI is in the bank's estate now |

### 4.6 The Mapping — Top 10 to CWE to ASVS

The three standards of §4.2–§4.4 are one system, and the team that can translate between them is the team that can *action* them: the Top 10 names the risk class, the CWE identifiers name the weakness (what the SAST rulesets and vulnerability feeds speak), and the ASVS chapter names the verification requirement (what the review and the tests check). The working mapping — ⚠-structural for the exact CWE/ASVS identifiers (the CWE families are stable and the ASVS v4 chapter structure is public, but the mapping is a practitioner synthesis, not a single canonical table):

| Top 10 2021 (✅) | Representative CWEs | ASVS focus (v4) | The design response |
|---|---|---|---|
| A01 Broken Access Control | CWE-639 (IDOR), CWE-22 (path traversal), CWE-269 (improper privilege) | V4 (access control), V2/V3 (auth) | Server-side object-level checks; one authorization decision point (§4.2) |
| A02 Cryptographic Failures | CWE-327 (broken crypto), CWE-311 (missing encryption) | V6 (cryptography) | Approved algorithms, encryption at rest and in transit, key management (§7.1) |
| A03 Injection | CWE-89 (SQLi), CWE-79 (XSS), CWE-78 (OS command) | V5 (validation/sanitisation) | Parameterised queries, output encoding, allow-list validation |
| A04 Insecure Design | (design-level; no single CWE) | V1 (architecture) | Threat modelling (§2), the §3 principles, misuse cases |
| A05 Security Misconfiguration | CWE-16 (configuration), CWE-1004 (sensitive cookie) | V14 (configuration) | Secure defaults (§3.5), hardening baselines, IaC scanning (§6.2) |
| A06 Vulnerable & Outdated Components | CWE-937 (components), CWE-1035 (outdated) | (SCA layer) | SCA (§5.5), the SBOM (§6.4), the dependency-upgrade treadmill |
| A07 Identification & Auth Failures | CWE-287 (improper auth), CWE-384 (session fixation) | V2, V3 | MFA, session lifecycle, cross-ref [distributed_auth_guide.md](distributed_auth_guide.md) |
| A08 Software & Data Integrity Failures | CWE-502 (deserialization), CWE-345 (insufficient verification) | V1, V11 (business logic) | Signatures, provenance (SLSA §6.5), trusted CI/CD (§6.2) |
| A09 Logging & Monitoring Failures | CWE-778 (insufficient logging) | V7 (logging), V8 (data protection) | The audit trail of §3.6; retention per §8.1 |
| A10 SSRF | CWE-918 (server-side request forgery) | V5, V11 | Egress allow-lists, URL validation, no raw metadata access |

The usage pattern: take the threat register's top rows (§2.8), find their CWE families in the SAST ruleset, write the ASVS verification items as the acceptance criteria, and close the loop with the tests of §5. That chain — threat → weakness → requirement → evidence — is the secure-SDLC's version of traceability, and it is exactly what a design review (the §9 runbook) produces as its audit trail. ⚠-structural (mapping synthesis; the individual facts — the CWE numbers, the Top-10 list, the ASVS structure — are each ✅ verified in §4.2–§4.4).

---

## 5. The Security Testing

### 5.1 Testing as Evidence, Not Ritual

Security testing is the *evidence layer* of security-by-design: SAST/DAST/IAST/SCA and pentesting exist to prove the properties the design claimed. The four automated techniques are complementary — each sees what the others cannot — and the 2020s practice is to run them all, at different points in the pipeline, and to treat their outputs as data (defect counts, density, mean-time-to-remediate) rather than as a compliance checkbox. ✅-structural (the four-way taxonomy is uniform across OWASP and vendor literature; verified live this pass via OWASP's SAST tools page and multiple practitioner sources). The ordering rule: **cheapest first, earliest first** — static analysis on every commit, composition analysis on every dependency change, dynamic analysis on every deployable build, interactive analysis on the critical paths, and human pentesting on the release candidate for the regulated bar.

### 5.2 SAST — Static Application Security Testing

**SAST** analyses source code, bytecode or binaries *without executing them* — the "white-box" scanner that runs in the IDE and the CI pipeline, checking for the CWE classes of §4.4 (injection, XSS, hardcoded secrets, unsafe deserialization, path traversal...). ✅ Verified (OWASP: "Source code analysis tools, also known as SAST tools, can help analyze source code or compiled versions of code to help find security flaws"). Strengths: fast, scalable, finds the flaw at the exact line, zero runtime needed, blocks *before* the build ships. Weaknesses: false positives (the tuning tax), no runtime context (it cannot see "the WAF already blocks this"), and blind spots for logic flaws and configuration issues. The design decision for the pipeline: **SAST runs on every pull request, with the severity threshold tuned so the gate is meaningful** — a gate that blocks on every medium floods the backlog and teaches developers to hate security; a gate that only blocks on critical-plus finds the real bugs. ⚠-structural (the tuning guidance is practice doctrine).

### 5.3 DAST — Dynamic Application Security Testing

**DAST** tests the *running* application from the outside — the black-box scanner that sends attacks (XSS payloads, SQLi probes, auth bypass attempts) against a deployed environment, exactly as an external attacker would. ✅ Verified (practitioner sources: "Dynamic Application Security Testing (DAST) for runtime vulnerability discovery"). Strengths: it tests the real deployment (framework config, headers, TLS, auth flows, exposed endpoints), finds runtime-only issues (misconfigurations, exposed admin paths, CORS), and produces attack-shaped evidence. Weaknesses: it needs a running environment and seeded data (a non-prod env with realistic state), sees only what it can reach, and cannot inspect internal logic — the black-box blind spot. DAST belongs in the **staging environment on every deployable build**, and — critically for banks — **in production-safe forms** (authenticated scans of non-destructive paths) because the highest-fidelity attack surface is production itself. ⚠-structural.

### 5.4 IAST — Interactive Application Security Testing

**IAST** instruments the application (an agent inside the runtime) and combines static and dynamic views: it observes real requests and data flow through the actual code, flagging vulnerabilities *with the exact code path that was exercised*. ✅ Verified (the "interactive" hybrid is uniform across sources — augmentcode/invental describe IAST as the runtime-plus-code hybrid). Strengths: near-zero false positives (it saw the data flow), code-level detail, works inside automated test suites and API tests, and covers logic the black-box scanner cannot reach. Weaknesses: agent overhead, runtime dependency, and coverage limited to what the test traffic exercises. IAST is the pragmatic answer to "SAST cries wolf and DAST is blind" — in a Cymbal Bank estate it slots into the API/integration test suite where the business logic (and its authz bugs) actually lives.

### 5.5 SCA — Software Composition Analysis

**SCA** scans the *dependencies* — libraries, packages, containers, their versions and their transitive trees — against vulnerability databases (NVD, CVE feeds, vendor advisories) and license databases. ✅ Verified (Checkmarx/OWASP-adjacent sources: "Software Composition Analysis (SCA) for third-party dependency scanning"). It exists because modern applications are 80–90% third-party code, which is precisely why the OWASP Top 10 has a dedicated row (A06 vulnerable and outdated components ✅) and why §6.4's SBOM is the supply-chain evidence layer. SCA's output is a dependency inventory with known-CVE annotations — and its operational reality is the **dependency-update treadmill**: the tool is only as good as the team's ability to actually upgrade or patch (and to suppress, with justification, the CVEs with no fix). The 2024 CWE Top 25's KEV counts are the SCA team's daily dashboard ✅ (§4.4).

### 5.6 Pentesting — the Human Layer

**Penetration testing** is the human, adversarial, end-to-end assessment: trained testers (internal red team or external firms) attempt to compromise the application as real attackers would — recon, exploitation, privilege escalation, chaining findings that scanners would report separately. ✅-structural (the practice is universal and regulated — MAS's TRM framework and DORA's TLPT (threat-led penetration testing, Article 26) both require adversary-style testing for the highest-risk entities ⚠-structural for the DORA-TLPT mechanics; §8.1). Pentesting is *not* a replacement for the automated layers (it is a point-in-time sample of a moving target) and not a first line of defense (a good pentest of an unhardened app is a report of 50 things SAST would have found in CI). The right shape: **automated scanning continuously, human pentesting on a schedule tied to the risk rating and to material changes** — for a bank, at least annually for regulated applications and before every major release of a client-facing system, with findings fed back into the same backlog as the threat register. Cross-ref [llm_development_risks_security_guide.md](llm_development_risks_security_guide.md) for the GenAI red-teaming variant (prompt-injection and model-specific testing are the 2025 extension of the pentest scope).

### 5.7 The Testing Table

| Technique | What it sees | Where it runs | Strengths | Blind spots |
|---|---|---|---|---|
| **SAST** ✅ | Source/bytecode flaws (CWE classes, hardcoded secrets) | IDE + CI on every PR | Fast, exact-line findings, blocks before build | False positives; no runtime context; no logic/config flaws (§5.2) |
| **DAST** ✅ | The running app from outside — runtime config, headers, exposed endpoints, auth flows | Staging on every deployable build; safe authenticated scans | Attack-shaped evidence; tests the real deployment | Needs env+data; black-box blind spots; reachability limits (§5.3) |
| **IAST** ✅ | Real requests through instrumented code — the exercised code path | Inside the test/API suite | Near-zero false positives; code-level detail; logic coverage | Agent overhead; coverage = test traffic only (§5.4) |
| **SCA** ✅ | Dependency/container vulnerabilities + licenses | CI on dependency change; continuous inventory | The 80–90% third-party code is covered; CVE-annotated inventory | Upgrade treadmill; suppression discipline; false "no fix" CVEs (§5.5) |
| **Pentesting** ✅-structural | The human, chained, end-to-end attack | Scheduled by risk rating + material change; pre-release for regulated apps | Finds what tools miss; chains findings; regulator-recognised evidence | Point-in-time sample; expensive; not a CI-loop control (§5.6) |
| **ASPM** ⚠-structural | The orchestration of all the above — correlation and prioritisation | The 2025-era layer over the four tools | Turns scanner noise into ranked, reachable risk | New category; vendor-dependent definitions (§5.1) |

5. The Security Testing

### 5.8 The Security Test Pyramid — Building the Programme

The five techniques of §5.2–§5.6 compose into a **pyramid**, ordered by cost and frequency — the cheap, fast layers run constantly; the expensive, deep layers run on schedule; every layer's findings flow into one backlog (the same rule as the threat register, §2.8):

| Layer | Cadence | Owner | What it proves |
|---|---|---|---|
| SAST in the IDE | Every keystroke (developer-side) | Developer | The obvious flaws never leave the laptop (§5.2) |
| SAST + SCA in CI | Every pull request | Pipeline (DevSecOps) | The merge gate: no new critical/high, no known-exploited dependency (§6.2) |
| IAST in the API/integration suite | Every test run | QA + security champion | The exercised business logic is clean — the authz bugs of §9 live here (§5.4) |
| DAST on staging | Every deployable build | DevSecOps | The deployed configuration is attack-shaped-safe (§5.3) |
| Pentest on the release candidate | Per release + at least annually for regulated apps | External/internal testers | The human, chained view — regulator-recognised evidence (§5.6) |
| Red team / GameDay | Annually (or per risk rating) | Security team + business | The *operational* response — cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §7 and DORA's TLPT (§8.1) |

The programme metrics that matter (⚠-structural, practice doctrine): **mean time to remediate** (MTTR for findings — the trend line tells you if the pipeline works), **false-positive rate** (a high rate means the developers have learned to ignore the tool), **gate-block rate** (how often the pipeline actually stops something — if it is zero, the gates are theatre), and **open-critical count** (the number the board and the regulator both read). The pyramid's point is *coverage with economics*: every layer has a blind spot (§5.7's table), so the layers must all exist — but each layer only earns its cost if the layer below it is actually green.

---

## 6. The DevSecOps

### 6.1 Security as Code, in the Pipeline

**DevSecOps** is shift-left's operational form: security is not a separate team's phase but a set of *automated controls in the delivery pipeline itself* — security policies defined as code (policy-as-code), gates that fail the build, secrets that never touch the repo, and supply-chain evidence (SBOM, provenance) generated by the build. ✅-structural (the discipline is uniform across the DevSecOps literature; the Practical DevSecOps course canon — "secure SDLC and CI/CD with SCA, SAST, DAST, and Security as Code" — was verified live this pass). The three pillars of this section — CI/CD gates, secret management, and SBOM/SLSA — are the ones a bank's delivery platform must have before "we do DevSecOps" means anything.

### 6.2 CI/CD Security Gates

The **CI/CD security gate** is a defined, enforced checkpoint in the pipeline where a security check's result decides the build's fate: SAST and SCA results gate the merge/promote; DAST gates the deploy-to-staging; container scanning gates the image push; infrastructure-as-code scanning (Terraform/CloudFormation misconfiguration checks) gates the environment change; signed artifacts gate the release. ✅-structural (gate placement is universal DevSecOps practice; the exact tool chain is ⚠-vendor-specific). The engineering realities that make gates work or fail: **(1) fail-fast with feedback** — the developer who broke the gate must see *why* in the merge request, not in a dashboard; **(2) severity thresholds, not binary** — a gate on every low-severity finding produces bypass pressure, so thresholds are tuned to the risk class (block on critical/high for SAST; block on known-exploited for SCA); **(3) exceptions are tracked, not silent** — a "waive" action writes a dated, owned justification to the audit trail; **(4) the pipeline itself is a target** — A08 software and data integrity failures (✅ §4.2) names CI/CD compromise as a Top-10 risk class, so the pipeline runs on least-privileged, short-lived credentials, with protected branches, signed commits and reproducible builds — the SLSA layer of §6.5. The gate chain is also the *evidence chain*: the pipeline's green run is the artifact the auditor asks for.

### 6.3 Secret Management

**Secret management** is the practice of ensuring that credentials — API keys, DB passwords, TLS private keys, signing keys, cloud access tokens — never appear in code, config, logs, or container images, and are instead issued, rotated and revoked by a dedicated system. ✅-structural (universal practice; the tooling canon is HashiCorp Vault and the cloud-native secret services — AWS Secrets Manager, Azure Key Vault, GCP Secret Manager — with Kubernetes Secrets as the platform-native layer ⚠-structural for the vendor list). The mechanics that matter: **centralised vault with least-privilege access policies** (the §3.2 principle applied to credentials); **dynamic, short-lived secrets** (database credentials that expire in minutes, not passwords that live for years); **automated rotation** (and the *revocation* test — can you kill a leaked token in seconds?); **scanning for leaked secrets** (the secret-scanning layer of SAST, plus repo-history scrubbing for what already leaked); and the **zero-touch rule** — no human should ever need to know the production database password, which means no human ever can. The DORA/regulatory angle: ICT third-party credentials and key-management controls are exactly the kind of control a DORA or MAS examination probes (⚠-structural). The design anti-pattern to name in reviews: **secrets in environment variables set by a deploy script** — visible to anyone who can read the pipeline logs — versus **secrets fetched from the vault at runtime**.

### 6.4 The SBOM — the Software Bill of Materials

An **SBOM** is the formal, machine-readable inventory of a software artifact's components — every dependency, its version, its supplier, and (in the modern form) the vulnerabilities and licenses attached to each — the "ingredients list" for software. ✅-structural (the concept is universal; the NTIA minimum elements for an SBOM — supplier, component, version, dependency relationships, hashes, and other data — published July 2021, are the widely-cited baseline ⚠-structural for the exact element list wording; EO 14028 (May 2021) made SBOMs a US-government supply-chain requirement ✅-canon, and the SLSA page verified this pass quotes "meeting EO 14028" directly). The SBOM is the *consumption* side of supply-chain security: an SCA scan (of §5.5) is only as good as the inventory it can build, and the SBOM is that inventory made portable and auditable — generate it at build time (from the lock files and container manifests), store it beside the artifact, and *consume* it: when a new CVE drops, the question "are we affected?" is a lookup, not a project. For a bank, SBOMs are also the due-diligence currency for third-party software (cross-ref the [regtech_guide](../banking/regtech_guide.md) and the bank series' vendor-management chapters) and are moving into regulatory expectation as DORA's ICT third-party register (Article 28's Register of Information ✅ verified §8.1) meets the supply-chain reporting wave ⚠-structural for the "SBOM is DORA-mandated" claim — it is not directly mandated, but the register and resilience-testing obligations create the demand.

### 6.5 SLSA — Supply-chain Levels for Software Artifacts

**SLSA** ("salsa") is the security framework for supply-chain integrity: a checklist of controls, organised into **four compliance levels of increasing assurance**, covering the build, source and dependency links of the chain — from "basic hygiene" (level 1: provenance generated) through "hardened build" (level 3: tamper-resistant, hermetic, reproducible builds with strong provenance) toward the most rigorous (level 4). ✅ Verified this pass (slsa.dev: "Supply-chain Levels for Software Artifacts, or SLSA... a security framework, a checklist of standards and controls to prevent tampering, improve integrity, and secure packages and infrastructure... four compliance levels of increasing assurance"; the OpenSSF stewardship and EO 14028 references are on the page). The framework originated at Google (2021, ⚠-structural for the exact origin date as published) and is stewarded by the Open Source Security Foundation (OpenSSF) ✅ (slsa.dev logo list). The pairing with SBOM: **the SBOM says what is in the artifact; SLSA says the artifact is what it claims to be** — provenance (who built it, from what, with what) is SLSA's core mechanism, and provenance plus SBOM plus signed artifacts is the complete supply-chain story. For a Cymbal Bank delivery platform, the practical target is **SLSA level 2–3** for regulated builds: generated provenance, signed artifacts, and a build platform hardened enough that the provenance is trustworthy — which is also the A08 mitigation (§4.2) and the audit evidence a DORA examiner will accept for third-party and internally-built software ⚠-structural for the regulatory-acceptance claim.

### 6.6 The DevSecOps Table

| Pillar | Core mechanism | Canon / verification | The failure mode it prevents |
|---|---|---|---|
| **CI/CD gates** | Enforced security checkpoints (SAST/SCA/DAST/container/IaC) with severity thresholds and tracked exceptions | ⚠-structural (universal practice) | Vulnerable code shipping because "security review happens later" (§6.2) |
| **Secret management** | Vault-based, dynamic, short-lived, rotated, zero-touch credentials; secret scanning | ⚠-structural (Vault/cloud-secrets canon) | Credential theft from repos, logs, images; the #1 initial-access vector (§6.3) |
| **SBOM** | Machine-readable component inventory (NTIA minimum elements) generated at build, consumed on CVE drop | ✅ EO 14028/NTIA (⚠-structural element list); SLSA page verified | "Are we affected by this CVE?" becoming a project instead of a lookup (§6.4) |
| **SLSA** | Four-level integrity framework: provenance, signed artifacts, hardened builds | ✅ slsa.dev verified | Supply-chain tampering — malicious dependency, compromised builder (§6.5) |
| **Pipeline hardening** | The pipeline as a target: least-privilege CI credentials, protected branches, signed commits, reproducible builds | ⚠-structural (A08 mirror) | CI/CD compromise — the OWASP A08 risk class (§6.2, §4.2) |
| **Security-as-code** | Policies (gate thresholds, exceptions, scan configs) versioned in the repo like application code | ⚠-structural (DevSecOps doctrine) | Drift between "what the security team wants" and "what the pipeline does" (§6.1) |

6. The DevSecOps

### 6.7 The DevSecOps Maturity Ramp

DevSecOps is a journey with recognisable stages, and the honest conversation with a delivery platform is "which stage are we at, and what does the next stage cost?" — the same maturity logic as the OWASP SAMM (⚠-canon, §4.5) applied to the pipeline:

- **Stage 0 — ad-hoc:** security scans exist but run manually, on someone's laptop, after the release. Findings arrive as PDFs. This is not DevSecOps; it is the hardening-sprint model this guide exists to replace.
- **Stage 1 — gated:** SAST/SCA run in CI with severity thresholds; the pipeline can fail; secrets are scanned for. The gate exists but exceptions may be silent, and the SBOM does not exist yet.
- **Stage 2 — governed:** gates with tracked, owned exceptions; secrets are vaulted with rotation; the SBOM is generated per build; container images are scanned and signed; the threat register is linked to the pipeline output (a new high finding creates a tracked issue automatically).
- **Stage 3 — evidence:** policy-as-code (the gate thresholds live in the repo), SLSA level 2–3 provenance for regulated builds (§6.5), and the pipeline itself is hardened and audited (protected branches, signed commits, least-privilege CI credentials — §6.2's A08 mitigation). The green pipeline run *is* the audit artifact for a DORA/FSM-N05 examination (§8).

⚠-structural (the stage naming and boundaries are this guide's synthesis of uniform practice; the components — gates, vaults, SBOM, SLSA — are each verified in §6.2–§6.5). The regulatory read of the ramp: DORA's digital operational resilience testing (Article 24–26) and the MAS notices expect an organisation that has left Stage 0 — the evidence-producing Stage 3 is where examinations find what they are looking for.

---

## 7. The Cloud Security

### 7.1 The Shared-Responsibility Model

The **shared-responsibility model** is the cloud-security division of labour between provider and customer: **the provider is responsible for the security OF the cloud** (physical facilities, hypervisor, network backbone, the managed services it operates), **the customer is responsible for the security IN the cloud** (their data, their configurations, their identities and access policies, their application code, their workloads, their network segmentation within their tenancy). ⚠-structural (the model is universal doctrine — AWS, Azure and GCP all publish it in near-identical form, and it descends from the 2011-era AWS security whitepapers — but no single primary source was extracted live this pass; the term and the split are uniformly documented). The model is the cloud answer to the §3 principle of **complete mediation and least privilege**: the boundary is the contract, and the dangerous zone is the *middle* — misconfigurations (a publicly exposed S3 bucket, an overly permissive IAM role, an open security group) are customer-responsible, and misconfiguration is precisely the leading cloud breach cause (industry analyses attribute the large majority of cloud breaches to customer-side misconfiguration — ⚠-structural, widely cited percentages vary, deliberately not quoted as a precise figure). For a Cymbal Bank the model maps onto the outsourcing/third-party risk controls: the cloud provider is a critical ICT third party, the split of responsibilities must be documented (the DORA Register of Information ✅ §8.1 and MAS outsourcing requirements both demand exactly this inventory ⚠-structural for the MAS-outsourcing mapping), and the *customer's* side — identity, encryption, logging, config — is where the security-by-design discipline of this guide operates.

### 7.2 CSPM — Cloud Security Posture Management

**CSPM** (Cloud Security Posture Management) is the class of tooling that continuously monitors the *configuration* of a cloud estate against security best practice and compliance frameworks: it detects misconfigurations (public storage, open security groups, over-permissive IAM, disabled logging, unencrypted volumes), scores the estate's posture, and drives remediation. ✅ Verified this pass (CrowdStrike and SentinelOne CSPM-vs-CWPP explainers: "cloud security posture management (CSPM)... the visibility and control necessary to secure ever-expanding cloud environments"; the category is uniformly described). It is the automated enforcement of §3.5's secure defaults at cloud scale — the "secure-by-default" posture check that runs 24/7 instead of at the quarterly review — and it produces the compliance evidence (against CIS benchmarks, NIST, ISO-aligned baselines ⚠-structural for the benchmark list) that the audit function consumes. The modern container for CSPM and CWPP is **CNAPP** (Cloud-Native Application Protection Platform) — the 2020s convergence category that unifies posture, workload, identity and runtime protection in one platform ✅-structural (CrowdStrike and cyvitrix describe CSPM+CWPP as CNAPP's backbone; the Gartner coinage of both terms is ⚠-structural — widely attributed to Gartner's cloud-security research ~2017–2020, not verified live).

### 7.3 CWPP and CNAPP — Workload Protection

**CWPP** (Cloud Workload Protection Platform) protects the *workloads* themselves — the containers, VMs, serverless functions and Kubernetes clusters — with runtime controls: vulnerability scanning of images, runtime behaviour monitoring, file-integrity and drift detection, micro-segmentation, and the ability to block malicious behaviour inside the workload. ✅ Verified (Microsoft: "A cloud workload protection platform (CWPP) is a cloud security solution that helps protect cloud workloads in multicloud and hybrid environments"; CrowdStrike/SentinelOne confirm the split). The distinction that matters for design: **CSPM guards the configuration of the estate (the walls), CWPP guards the running workloads (the rooms)** — a misconfigured cluster is a CSPM finding, a compromised container inside it is a CWPP catch, and defence-in-depth (§3.3) says you need both, because a workload-level breach inside a "perfectly configured" estate is exactly the attack that posture management alone misses. For the bank's estate: image scanning at build (the CI gate of §6.2), runtime protection in production (the CWPP), and posture monitoring everywhere (the CSPM), with the findings fed into the same remediation backlog as the SAST/SCA output — one pipeline of risk, not four silos of alerts.

### 7.4 The Cloud Table

| Layer | What it secures | Canon / verification | Design decision |
|---|---|---|---|
| **Shared-responsibility model** | The provider/customer boundary — security of the cloud vs in the cloud | ⚠-structural (AWS/Azure/GCP doctrine) | Document the boundary per workload; the customer side is where the discipline applies (§7.1) |
| **CSPM** | Configuration and posture of the estate — the walls | ✅ (CrowdStrike/SentinelOne/Microsoft sources) | Continuous misconfiguration detection + compliance evidence; automated remediation (§7.2) |
| **CWPP** | The running workloads — the rooms | ✅ (Microsoft: "protects cloud workloads in multicloud and hybrid") | Image scanning at build + runtime behaviour protection in prod (§7.3) |
| **CNAPP** | The converged platform — posture + workload + identity + runtime | ⚠-structural (2020s convergence category) | Platform consolidation decision — fewer vendors, one risk pipeline (§7.2) |
| **Cloud IAM / identity** | The access plane — cross-ref [distributed_auth_guide.md](distributed_auth_guide.md) | ✅-cross-ref | Least-privilege roles, short-lived credentials, conditional access (§3.2, §6.3) |
| **Data-plane controls** | Encryption at rest/in transit, KMS/HSM, masking, tokenization | ⚠-structural (cross-ref the skill-gaps guide's data-security list) | The Zero Trust data pillars of §3.7 — encryption is the last line of defence (§7.1) |

7. The Cloud Security

### 7.5 The Cloud Decision Map — Workload to Control

The cloud-security layer is not one decision but a set, and the map below is the design-review shortcut: for each workload class in a Cymbal Bank estate, the primary controls, and where the CSPM/CWPP layers sit (⚠-structural synthesis over the verified categories of §7.2–§7.3):

| Workload class | Primary controls | Where CSPM/CWPP acts |
|---|---|---|
| Client-facing web portal | WAF + DAST + rate limiting; secure defaults (§3.5); the §9 review pattern | CSPM: config drift on the load balancer, TLS, headers |
| Internal API layer | mTLS service-to-service; scoped OAuth; per-role scopes (§3.2) | CSPM: IAM policy drift; CWPP: runtime behaviour of the API pods |
| Payment/trade services | Request signing; idempotency; maker-checker (§3.6); the STRIDE T/R threats | CWPP: container image scanning at build + runtime guard in prod |
| Batch/reporting | Per-job identities (§6.3); concurrency caps; scheduled windows (§9 finding #8) | CSPM: the batch account's permission set |
| Data lake / analytics | Encryption at rest; masking; row/column-level security (§3.7 data pillars) | CSPM: bucket/volume config; CWPP: the analytics agents |
| Serverless/GenAI functions | Egress allow-lists (SSRF, A10); model access controls; cross-ref [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) | CWPP: function runtime protection |

The two-layer rule: **CSPM watches the configuration, CWPP watches the workload, and both feed one backlog** — the silo between them is exactly where the 2026-era CNAPP convergence (⚠-structural, §7.2) is trying to remove friction.

---

## 8. The Regulatory

### 8.1 EU DORA — the Digital Operational Resilience Act

**DORA** — Regulation (EU) **2022/2554**, the Digital Operational Resilience Act — is the EU regulation that makes *digital operational resilience* a regulated property of the financial sector: it replaced the capital-buffer approach to operational risk with explicit, uniform requirements for ICT risk management, incident reporting, resilience testing, and third-party risk. ✅ Verified this pass (the DORA tracking site: "Regulation (EU) 2022/2554... financial institutions are now required to follow stringent guidelines for safeguarding against ICT-related incidents... protection, detection, containment, recovery, and repair... clear rules for ICT risk management, incident reporting, operational resilience testing, and oversight of ICT third-party risks"). The dates (✅ verified this pass): **published 27 December 2022, applicable from 17 January 2025** — the page's 2025 timeline ("DORA becoming applicable on 17 January 2025", "Following the entry into force of DORA on 17 January 2025") and the ESAs' ongoing implementation work confirm it is live and being enforced. The five pillars (✅ per the regulation's own Article 1, extracted live): **(1) ICT risk management** (Articles 5–16 — the framework, governance, and the security-by-design-relevant requirements for the ICT lifecycle); **(2) incident reporting** (Articles 19–21 — major ICT-related incident reporting to competent authorities); **(3) digital operational resilience testing** (Articles 24–26 — including **TLPT**, threat-led penetration testing, for the highest-risk entities); **(4) ICT third-party risk** (Articles 28–44 — including the **Register of Information**, the inventory of ICT third-party arrangements ✅, and the contractual requirements); **(5) information sharing and the **Oversight Framework** for critical ICT third-party providers (CTPPs)** — the ESAs designated the first critical CTPPs in November 2025 (✅ on the extracted page's timeline). **The security-by-design read of DORA:** its ICT risk-management chapter requires security to be considered across the ICT lifecycle — design, development, procurement, operation — which is this guide's discipline made regulatory text: threat-modelled, tested, logged, and third-party-verified systems are what a DORA examination looks for. Cross-ref the [financial_risk_compliance_systems_guide](../banking/financial_risk_compliance_systems_guide.md) for the op-risk taxonomy that DORA reshapes.

### 8.2 EU NIS2 — the Cyber-Resilience Umbrella

**NIS2** — Directive (EU) **2022/2555** — is the EU's cybersecurity framework for the *essential and important entities* across 15+ sectors (energy, transport, health, digital infrastructure, and — critically for this guide — **banking and financial market infrastructure**), replacing the original NIS Directive (2016/1148). ✅ Verified this pass (EUR-Lex: "Directive (EU) 2022/2555... on measures for a high common level of cybersecurity across the Union... repealing Directive (EU) 2016/1148 (NIS 2 Directive)", adopted 14 December 2022, OJ L 333 of 27.12.2022; ENISA: "The Member States had to transpose NIS2 to their national legislative framework by 17th October 2024"). NIS2's security-by-design relevance: it mandates **supply-chain and vendor security** (the SBOM/SLSA agenda of §6.4–6.5), **risk-management measures** across the full lifecycle, **top-management accountability** (directors can be held liable — the governance version of "security is a feature, not an add-on"), and incident reporting, with enforcement fines up to **€10M or 2% of worldwide turnover** for essential entities (⚠-structural for the exact fine bands — widely documented, not extracted live). For a global bank like Cymbal Bank: DORA is the sector-specific regime for financial entities, NIS2 the horizontal regime for the digital infrastructure and partners around it — the two are designed to complement each other, and a compliance programme that treats them as one resilience estate is the sane reading. ⚠ on any claim that NIS2 applies *directly* to a bank's daily operations the way DORA does — the financial sector is largely carved into DORA's scope; NIS2 covers the entities around the financial core.

### 8.3 The Singapore CSA — Security-by-Design at the State Level

The **Cyber Security Agency of Singapore (CSA)** is Singapore's national cybersecurity authority (established 2015, ⚠-structural for the exact founding date and reporting line; it operates under the Prime Minister's Office's Smart Nation group and later the Ministry of Communications and Information), and it has been running a visible **security-by-design agenda** for a decade: ✅ verified this pass on csa.gov.sg — the **Cybersecurity Labelling Scheme (CLS)** for consumer smart devices (the world's first national IoT security-labelling scheme of its kind, launched for IoT products; levels from basic security provisions up to resistance to basic/common attacks ⚠-structural for the level definitions), the **Safer Cyberspace Masterplan 2020** ("three thrusts, to: Secure our core digital infrastructure; ..." ✅ — the masterplan's first thrust is securing core digital infrastructure, which is the state-level statement of secure-by-design), and the **OT Cybersecurity Masterplan 2024** which explicitly promotes **secure-by-design and secure-by-deployment principles throughout the lifecycle of OT systems** ✅ (verified via the waterfall-security summary of the masterplan and the CSA pages). The related frame: the **Cybersecurity Act 2018** (Singapore's law for critical information infrastructure and cybersecurity service providers — ⚠-structural, well documented, not extracted live this pass) and the CSA's sectoral partners (MAS for financial services). For Jack's context the CSA layer matters twice: as the *national* backdrop to the bank's SG operations (cross-ref [singapore_security_clearance_guide.md](singapore_security_clearance_guide.md) and the banking series' Singapore guides), and as the *policy echo* — Singapore's regulators and the EU arrived at the same conclusion in the same decade: **security must be designed in, not bolted on**.

### 8.4 MAS Notice 655 — Flagged: Cancelled and Superseded

**⚠ THIS ROW IS FLAGGED — the widely-cited "MAS Notice 655" no longer exists in its original form.** The live-verified facts (✅ this pass, via the MAS regulatory record on regalert.today and the 2024 FSMA notice texts): on **9 May 2024, MAS cancelled six legacy technology-risk notices — MAS Notices 644, 655, 644A, 655A, 1114 and 1118 (issued under the Banking Act 1970) — effective 10 May 2024**, and replaced them with notices issued under the **Financial Services and Markets Act 2022 (FSMA)**: for banks, **MAS Notice FSM-N05 "Notice on Technology Risk Management"** (issued 9 May 2024, effective 10 May 2024, pursuant to section 29(1) of the FSMA 2022 ✅ — the collegesidekick copy of FSM-N05 confirms the issue date, the Act and the application), and the parallel FSM-series notices for other FI types (e.g. FSM-N21 for specified financial institutions ✅). Historical context (⚠-structural — widely documented but not live-verified this pass): **MAS Notice 655** was the Technology Risk Management notice for banks, issued **8 August 2018 and effective 16 August 2019**, superseding the earlier Notice 644, and requiring banks to establish technology-risk frameworks, keep critical systems highly available, and notify MAS of incidents. The 2026 state (✅-structural via the Baker McKenzie June 2026 insight title): **MAS has proposed further amendments to the TRM notices** (IT asset management among the areas) — the regime is still moving. **Why this matters for this guide:** any 2020-era source citing "MAS Notice 655" as *current* is out of date; the correct citation for a 2026 security review of a bank is **MAS Notice FSM-N05 (TRM) under the FSMA 2022**, with the companion cyber-hygiene requirements (Notice 645, also being folded into the FSMA migration ⚠-structural) and the MAS Technology Risk Management Guidelines as the supporting guidance. The *substance* the industry associates with 655 — technology risk frameworks, high availability, incident notification, and the security-by-design expectations for systems development — carried over into the FSMA-era notices unchanged in spirit ✅-structural.

### 8.5 The Regulatory Table

| Regime | What it requires | Dates / status | Security-by-design read |
|---|---|---|---|
| **EU DORA** (Reg. 2022/2554) ✅ | ICT risk management, incident reporting, resilience testing (incl. TLPT), ICT third-party risk + CTPP oversight | Published 27.12.2022; **applicable 17 January 2025** ✅ | The discipline as law: lifecycle security, tested resilience, third-party evidence (§8.1) |
| **EU NIS2** (Dir. 2022/2555) ✅ | Risk-management measures, supply-chain security, incident reporting, board accountability | Adopted 14.12.2022; **transposition by 17 October 2024** ✅ | The horizontal umbrella around DORA; supply-chain and governance teeth (§8.2) |
| **SG CSA** ✅ | National cybersecurity authority: CLS(IoT) labelling, Safer Cyberspace Masterplan 2020, OT Masterplan 2024 (secure-by-design/secure-by-deployment), Cybersecurity Act 2018 | CLS and masterplans live ✅; Act 2018 ⚠-structural | State-level secure-by-design — the policy echo of this guide's thesis (§8.3) |
| **MAS Notice 655** ⚠ **FLAGGED** | Legacy TRM notice for banks (Banking Act era) | Issued 2018 ⚠-structural; **cancelled 9 May 2024, effective 10 May 2024** ✅; superseded by **FSM-N05** ✅ | Cite FSM-N05 in 2026; the 655 substance (TRM frameworks, high availability, incident notification) carried over (§8.4) |
| **ISO/IEC 27001:2022** ✅-canon | The ISMS standard: Annex A controls, continual improvement | 2022 edition (published 25 Oct 2022 ⚠-structural date) | The management-system container the SDLC controls are certified inside (§8.6) |
| **NIST CSF 2.0** ✅ | Six functions: **Govern, Identify, Protect, Detect, Respond, Recover** | **Released 26 February 2024** ✅ | The outcome-language for board-level security conversation (§8.6) |

### 8.6 The Frameworks — ISO/IEC 27001 and NIST CSF 2.0

Two frameworks wrap the discipline in governance language. **ISO/IEC 27001** is the certifiable management-system standard (the 2022 edition reorganised Annex A into four themes — organisational, people, physical, technological controls ⚠-structural for the exact theme list; the 2022 edition is ✅-canon): it is the *container* — the SDLC controls of this guide (threat modelling, secure coding, testing, DevSecOps) become documented, audited ISMS controls inside it. **The NIST Cybersecurity Framework 2.0** — released **26 February 2024** ✅ (csrc.nist.gov: "The NIST Cybersecurity Framework (CSF) 2.0 is Here! February 26, 2024"; the NIST news release confirms the six-function core: **Govern** — the new sixth function — plus Identify, Protect, Detect, Respond, Recover ✅, with CSF 1.0 first released in 2014 following a presidential Executive Order ✅) — is the *vocabulary*: it expresses cybersecurity as outcomes ("protect data", "detect anomalous activity") that a board, a regulator and an engineering team can all read. The mapping habit worth building: take any finding from this guide's worked example (§9) and state it in CSF 2.0 function language (a missing authz check is a "Protect (PR)" gap with "Govern (GV)" ownership) and in ISO control language (it is an "access control" Annex A control) — the same fix, three dialects, one backlog.

8. The Regulatory

### 8.7 The Regulatory Horizon — What Moves Next

The regulatory table of §8.5 is a snapshot; the horizon is moving, and a 2026 security-by-design programme should be watching four things (each ⚠-structural unless marked):

1. **DORA implementation is in its enforcement phase** — the ESAs designated the first critical ICT third-party providers in November 2025 ✅ (verified on the DORA page this pass), the Joint Examination Teams are running, and the European Commission *rejected* the first draft RTS on ICT-subcontracting conditions in January 2025 (✅ verified on the DORA page — a rare live detail: the RTS was sent back for going beyond the Article 30(5) empowerment), meaning the third-party contractual layer is still settling. For a bank's delivery teams: the third-party/SaaS register discipline (the Register of Information) is now an audited reality.
2. **NIS2 is in the transposition lag** — the 17 October 2024 deadline passed with several large member states still legislating (⚠-structural, per the NIS2 EU-tracker sources: Italy, Belgium, Hungary, Croatia and Romania moved fast; Germany, France, Spain and the Netherlands were still in process). The practical effect: a slow, rolling implementation with national divergence — the compliance programme should track each jurisdiction where the bank operates, not "NIS2" as one blob.
3. **MAS is amending the TRM notices again** — the June 2026 consultation (Baker McKenzie insight, ⚠-structural) proposes amendments to the FSMA-era TRM notices (IT asset management among the named areas), continuing the May 2024 migration from Notice 655 to FSM-N05 (§8.4). The takeaway for this guide's readers: **the MAS TRM substance is stable — the instrument numbers keep changing; always cite the current instrument.**
4. **The secure-by-design policy wave** — CISA's "secure by design" push to software vendors (⚠-structural, 2023 origin) and Singapore's OT Masterplan 2024's secure-by-design/secure-by-deployment requirement (§8.3 ✅) are the policy echo of this guide's thesis: the vendors and platforms a bank buys are increasingly *obliged* to practise what this guide describes — which turns the procurement security questionnaire into a design-review conversation (cross-ref the regtech guide's vendor-management chapter).

---

## 9. The Worked Example — a Security-by-Design Review

### 9.1 The Scenario — a Cymbal Bank Application

**The setting (illustrative — not a description of any real Cymbal Bank system ⚠):** the house runs an internal **corporate client e-banking/self-service portal** for its corporate treasury clients — think the familiar context of the repo's banking series: a client-facing web portal (cross-ref [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) for the estate context) backed by a **REST API layer** (cross-ref [api_governance_guide.md](api_governance_guide.md)) that fronts **trade and cash-management services** (cross-ref [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md)), with **SSO authentication** via the bank's identity platform (cross-ref [distributed_auth_guide.md](distributed_auth_guide.md)), a **reporting/batch pipeline**, and a **cloud-hosted analytics data store** (cross-ref [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) for the data-plane angle). The application is mid-flight: architecture approved, first sprint done, and the team — under delivery pressure — has asked "can security wait until the hardening sprint?" The security-by-design answer is the review below: a **STRIDE-driven design review** run as a working session, exactly as §2 prescribes, before the integration backlog grows any further.

### 9.2 The Review Design — the STRIDE Model Applied

The review uses the Shostack four-question frame (✅-canon, §2.1): **What are we building?** (the DFD: browser → SSO → portal → API gateway → services → data stores, with the batch and analytics flows drawn in); **What can go wrong?** (the STRIDE walk per DFD element, below); **What are we going to do about it?** (the mitigations, owned and tracked); **Did we do a good job?** (the verification mapping — which ASVS level, which tests, which gates — §9.3). The session discipline: one element at a time, all six STRIDE categories asked per element, findings recorded verbatim into the threat register with a DREAD score (§2.3) for prioritisation, no "we'll handle that in hardening" without an owner and a date. The design-review agenda for this scenario, in STRIDE order:

1. **Spoofing** — *"who is this, really?"* The SSO integration: is the token validation done on the API gateway (yes, planned) *and* inside the services (not yet — the services trust the gateway's forwarded headers ⚠ **finding**); are service-to-service calls authenticated (mTLS planned for the payment path only); does the batch scheduler authenticate to the API (it uses a shared service account — **finding**: no per-job identity, §3.2 least privilege violated).
2. **Tampering** — *"can the data be changed in transit or at rest?"* TLS everywhere (planned); request signing on the payment API (not planned — a replay/tamper risk on the money path **finding**); the analytics export writes unencrypted CSVs to the data lake (**finding**: encryption at rest, §7.1 data-plane); the container images are unsigned (**finding**: SLSA provenance gap, §6.5).
3. **Repudiation** — *"can anyone deny what happened?"* The audit trail exists for payments but not for *entitlement changes* and *parameter changes* (**finding**: SoD without audit is a checkbox, §3.6); log retention is "as long as the disk allows" (**finding**: no retention policy — the DORA incident-reporting evidence chain, §8.1, needs defined retention ⚠-structural for the exact DORA retention requirement — flagged, not verified this pass).
4. **Information disclosure** — *"what can be read that shouldn't be?"* The API returns full account objects to the portal and the portal filters the display (⚠ **finding**: client-side filtering is an IDOR invite — server-side field selection, the A01 lesson of §4.2); error messages expose stack traces in staging (**finding**: verbose errors = A05, §3.5); the analytics store is shared with a non-production team (**finding**: separation, row-level controls, §3.7 data pillars).
5. **Denial of service** — *"what happens under attack?"* The portal endpoints have rate limiting at the gateway (good); the batch pipeline has *no* concurrency cap and can saturate the shared DB during client-facing hours (**finding**: availability vs batch, cross-ref [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) §8); the fail-secure decision on the auth service is undecided (⚠ **finding**: pick fail-closed and record the exception policy, §3.4).
6. **Elevation of privilege** — *"can a user become a user they are not?"* The portal role model is coarse (client-admin and maker roles share the same API scope — **finding**: scope splitting per role, §3.2 + cross-ref [distributed_auth_guide.md](distributed_auth_guide.md)); the internal admin console sits on the same gateway route as the client portal (⚠ **finding**: separate network segment + separate authN realm, the Zero Trust micro-segmentation of §3.7); the DB credentials in the batch config file (⚠ **finding**: secret management, §6.3).

### 9.3 The Review Output — Findings and Mitigations

The session output — the threat register, DREAD-ranked, with mitigations mapped to the discipline's layers and the repo's cluster:

| # | STRIDE | Finding (risk) | Mitigation (design change, not patch) | Layer / cross-ref |
|---|---|---|---|---|
| 1 | E | Coarse portal roles share API scopes (high) | Per-role scopes; the API gateway enforces scope→endpoint mapping; maker/checker split enforced server-side | §3.2, §3.6; [distributed_auth_guide.md](distributed_auth_guide.md), [api_governance_guide.md](api_governance_guide.md) |
| 2 | I | IDOR via client-side filtering of full account objects (high) | Server-side field selection; object-level authorization checks per request (the A01 fix) | §4.2; ASVS L2 (§4.3) |
| 3 | S | Services trust gateway-forwarded identity headers (high) | Signed/validated identity context propagation; mTLS service-to-service on all internal calls | §2.2, §7.1; [distributed_auth_guide.md](distributed_auth_guide.md) |
| 4 | T | Payment API unsigned, replayable (high) | Request signing + idempotency keys (cross-ref the zero-downtime guide's idempotency §8) | §2.2; [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) |
| 5 | R | No audit trail for entitlement/parameter changes (high) | Immutable audit log for all privileged changes; retention policy; DORA-aligned evidence chain | §3.6, §8.1; [../banking/regtech_guide.md](../banking/regtech_guide.md) |
| 6 | T | Unsigned container images; no provenance (medium) | SLSA level 2 target: provenance + signed artifacts; SBOM generated per build | §6.4, §6.5 |
| 7 | S | Batch runs on shared service account (medium) | Per-job identities, short-lived credentials from the vault | §3.2, §6.3 |
| 8 | D | Batch can saturate the shared DB; auth fail-open undecided (medium) | Batch concurrency caps + scheduled windows; explicit fail-closed decision documented | §3.4; [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) |
| 9 | E | Admin console shares gateway route with the portal (medium) | Separate segment + separate authN realm + conditional access | §3.7 Zero Trust micro-segmentation |
| 10 | I | Unencrypted analytics exports; verbose staging errors (low–medium) | Encryption at rest; error-handling policy (A05) | §3.5, §7.1 |

**The verification mapping (the fourth Shostack question):** the target bar is **ASVS L2** (§4.3); SAST + SCA gate the pipeline (§6.2); IAST runs inside the API test suite (§5.4); DAST scans staging per release (§5.3); pentest scoped from the threat register before the first client pilot (§5.6); and the whole file is evidence for the DORA/FSM-N05 examination (§8). Each finding's mitigation is a *design change* — an architecture decision, not a hardening checkbox — which is the entire point of doing this review now, at design time, when finding #2 is a one-line API contract change rather than a re-platforming project.

### 9.4 The Lessons

1. **The review found its biggest items in the seams, not the code** — trust boundaries (gateway→service identity propagation), entitlements, and the batch side were where STRIDE earned its keep; the seams are where the 1999 model still pays for itself in 2026 ✅.
2. **Every finding mapped to a principle or a standard** — nothing in the register was "expert opinion"; each row cites the principle (§3), the standard (§4), the test (§5) or the gate (§6) that closes it. That is what makes security-by-design *auditable* rather than *asserted*.
3. **The regulatory row is a design input** — DORA's lifecycle requirements and the FSM-N05 TRM expectations (⚠ §8.4) read naturally onto this register; the team that reviews with the regulators' language in the room gets to the same place with less rework.
4. **Zero Trust is the tie-in, not the bolt-on** — findings #3, #7, #9 and #10 are, in NIST SP 800-207 language, identity, workload, network and data pillars (§3.7) — which is exactly why the [data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md) standing priority #3 (Zero Trust for data platforms) is the natural next commission from this cluster.
5. **The cost curve is the argument** — ten findings, all caught before the second sprint, all fixable as design changes. The same register discovered at go-live would have been a regulatory incident, a client-communication, or a breach. Security is a feature, and like every feature, it is cheapest in the design.

### 9.5 The Review Runbook — the Template

The §9.1–9.4 review is repeatable, and the runbook below is the template any team can lift (⚠-structural — practice synthesis over the verified method canon of §2):

- **Trigger:** a new application, a material architecture change (new integration, new data flow, first cloud migration, first AI feature), or a regulatory milestone (pre-examination, pre-pilot).
- **Participants (60–90 minutes):** the solution architect (facilitator), the lead developer, the security champion/AppSec engineer, the platform/DevOps engineer, the product owner (for the PASTA-style business-impact questions), and — for a bank — the risk/compliance delegate. The threat register's owners must be in the room, or the session produces rows nobody owns.
- **Artifacts in:** the DFD (drawn before the session — the facilitator's homework), the integration inventory, the data-flow inventory (what data, where, who), the existing controls (identity platform, gateway, logging), and the regulatory obligations (DORA/FSM-N05 applicability).
- **The agenda:** (1) the DFD walk — every trust boundary called out; (2) the STRIDE pass per element (§2.2) with the six questions asked aloud; (3) attack-tree depth on the top two goals (§2.5); (4) DREAD scoring of every accepted threat (§2.3); (5) the mitigation assignment — principle (§3), standard (§4), test (§5), gate (§6); (6) the output contract: the register (§2.8) updated in the backlog, owners and dates set, and the verification mapping (ASVS level, test layers) agreed.
- **Artifacts out:** the updated threat register, the review minutes (the evidence trail), the open-items list with owners, and the next review date (design changes re-open it).
- **The ground rule:** no finding leaves the room without a *design* response or a written, dated risk acceptance — "we'll handle it in hardening" is not a response; it is the phrase this discipline exists to retire.

---

## 10. The One-Page Summary

**Security-by-design** is the discipline of making security a property of the architecture: **threat-modelled** before code (STRIDE's six categories from 1999, DREAD's ranking, PASTA's risk-centric process, Schneier's attack trees — §2), **principled** in the design (least privilege, defence-in-depth, fail-secure, secure defaults, separation of duties, and the Zero Trust posture of NIST SP 800-207 that ties them together — §3), **coded** against the standards (OWASP Top 10 2021 with A01 broken access control at #1, the ASVS verification levels, the CWE Top 25 — §4), **tested** as evidence (SAST, DAST, IAST, SCA, pentesting — §5), **automated** in the pipeline (CI/CD gates, secret management, SBOM and SLSA — §6), **hosted** on a documented boundary (the shared-responsibility model, CSPM and CWPP — §7), and **regulated** by a live floor (DORA applicable 17 January 2025, NIS2 transposed by 17 October 2024, the SG CSA's secure-by-design agenda, and the MAS TRM regime — now **FSM-N05**, with Notice 655 cancelled ⚠ — §8). The worked example (§9) shows the whole chain in one design review: ten findings, all caught early, all closed by design decisions, all mapped to evidence. The five-layer canon — principles (1970s), threat modelling (1990s), process (2000s), automation (2010s), supply chain and regulation (2020s) — is the map of the discipline; a programme that contains all five layers, with the threat register as its spine and the pipeline as its heartbeat, is a secure-SDLC programme.

10. The One-Page Summary

**The ten-line checklist** — a system, a service or a release is security-by-designed when all ten are true:

1. A threat model exists, is current (re-opened on design changes), and its top rows are owned and tracked (§2).
2. The design applies least privilege, defence-in-depth, fail-secure, secure defaults and separation of duties — and the exceptions are written down (§3).
3. The Zero Trust posture (never trust, always verify) is explicit at every trust boundary — NIST SP 800-207, the standing priority (§3.7).
4. The coding standards are named (OWASP Top 10 2021, ASVS L2 for regulated data, the CWE Top 25) and enforced by tooling (§4).
5. SAST, SCA, IAST, DAST and pentesting each exist at their right layer of the pyramid, with metrics (§5).
6. The pipeline gates on security, secrets never touch the repo, and the SBOM is generated per build (§6).
7. The supply chain is SLSA-aware: signed artifacts, provenance, verified dependencies (§6.5).
8. The cloud estate has a documented shared-responsibility boundary, CSPM posture monitoring and CWPP workload protection (§7).
9. The regulatory obligations are mapped to controls: DORA's lifecycle requirements, NIS2's supply-chain and governance teeth, the MAS TRM (FSM-N05) expectations, and the frameworks (ISO 27001, NIST CSF 2.0) that wrap them (§8).
10. The evidence exists: the threat register, the green pipeline runs, the test results and the audit trail — because security that cannot be demonstrated is security that did not happen.

**The final word — security is a feature.** Not a phase, not a hardening sprint, not an audit checkbox: a feature, with the same status as functionality — specified, designed, priced, built, tested and demonstrated. Features that are specified get built; features that are built get tested; features that are tested get shipped; and features that are shipped *stay* shipped. The systems that survive — the banks, the payment rails, the client portals — are the ones where security was given feature status from the first line of the design, and where the question was never "can security wait?" but "what does the threat model say we cannot ship without?"

---

## The Glossary

| Term | Definition |
|---|---|
| **Security-by-design** | The discipline of making security a property of the system's architecture — threat-modelled, principled, coded, tested, automated, regulated — rather than a phase added at the end (§1) |
| **Shift-left** | Moving security activities as early as possible in the SDLC, where the cost of fixing is lowest (§1.2) |
| **Threat modelling** | The systematic identification of what can go wrong with a specific system, producing a ranked, owned, tracked threat register (§2) |
| **STRIDE** | The 1999 Microsoft six-category threat-classification model: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege ✅ (§2.2) |
| **Spoofing** | Pretending to be someone or something else — a violation of authenticity (§2.2) |
| **Tampering** | Modifying data or code in transit or at rest — a violation of integrity (§2.2) |
| **Repudiation** | Denying an action without provability — a violation of non-repudiation (§2.2) |
| **Information disclosure** | Reading data one should not see — a violation of confidentiality (§2.2) |
| **Denial of service** | Preventing legitimate use — a violation of availability (§2.2) |
| **Elevation of privilege** | Gaining rights one should not have — a violation of authorization (§2.2) |
| **DREAD** | The Microsoft risk-rating model: Damage, Reproducibility, Exploitability, Affected users, Discoverability — scores threats for ranking ✅ (§2.3) |
| **PASTA** | Process for Attack Simulation and Threat Analysis — the risk-centric, seven-stage methodology from VerSprite (UcedaVélez & Morana) ✅ (§2.4) |
| **Attack tree** | Schneier's 1999 structure: goal as root, attack paths as AND/OR branches with attributes — the "exactly how" layer ✅ (§2.5) |
| **Least privilege** | Minimum permissions, minimum time, minimum blast radius (Saltzer & Schroeder 1975) ✅ (§3.2) |
| **Defence-in-depth** | Independent, layered controls so no single failure is fatal (§3.3) |
| **Fail-secure** | Failing in the secure direction — deny, block, close — the "fail-safe defaults" principle (§3.4) |
| **Secure default** | Out-of-the-box configuration is the secure configuration; opt-out with justification (§3.5) |
| **Separation of duties** | No single actor completes a critical action alone; maker-checker; collusion required (Clark-Wilson 1987) (§3.6) |
| **Zero Trust** | Never trust, always verify — NIST SP 800-207 (August 2020): continuous verification, least privilege, micro-segmentation (§3.7) |
| **OWASP** | The Open Worldwide Application Security Project — the community foundation behind the Top 10, ASVS, SAMM and the cheat sheets (§4) |
| **Top 10** | The OWASP Top 10 2021: the risk-based ranking of web-app risks, A01 broken access control first ✅ (§4.2) |
| **ASVS** | The OWASP Application Security Verification Standard — checkable security requirements at three levels ✅ (§4.3) |
| **SANS 25** | The SANS/MITRE ranking of dangerous weaknesses — the lineage continued as the MITRE CWE Top 25 ✅ (§4.4) |
| **SAST** | Static Application Security Testing — white-box analysis of source/bytecode without execution ✅ (§5.2) |
| **DAST** | Dynamic Application Security Testing — black-box testing of the running application ✅ (§5.3) |
| **IAST** | Interactive Application Security Testing — instrumented runtime analysis with code-path detail ✅ (§5.4) |
| **SCA** | Software Composition Analysis — dependency/container vulnerability scanning ✅ (§5.5) |
| **Pentesting** | Human, adversarial, end-to-end attack simulation by trained testers (§5.6) |
| **DevSecOps** | Security as code in the delivery pipeline — gates, secrets, supply-chain evidence, policy-as-code (§6) |
| **CI-CD** | Continuous Integration / Continuous Delivery — the automated pipeline the security gates live in (§6.2) |
| **Secret management** | Vault-based, dynamic, short-lived, rotated, zero-touch handling of credentials (§6.3) |
| **SBOM** | Software Bill of Materials — the machine-readable component inventory (NTIA minimum elements) ✅ (§6.4) |
| **SLSA** | Supply-chain Levels for Software Artifacts — the four-level integrity framework with provenance ✅ (§6.5) |
| **Shared responsibility** | The cloud split: provider secures the cloud, customer secures in the cloud (§7.1) |
| **CSPM** | Cloud Security Posture Management — continuous misconfiguration and compliance monitoring ✅ (§7.2) |
| **CWPP** | Cloud Workload Protection Platform — runtime protection of workloads and containers ✅ (§7.3) |
| **DORA** | The EU Digital Operational Resilience Act (Reg. 2022/2554) — ICT risk, incident reporting, testing, third-party risk; **applicable 17 January 2025** ✅ (§8.1) |
| **NIS2** | Directive (EU) 2022/2555 — the EU cybersecurity framework for essential/important entities; **transposition by 17 October 2024** ✅ (§8.2) |
| **CSA** | The Cyber Security Agency of Singapore — national cybersecurity authority; CLS, masterplans, the secure-by-design agenda ✅ (§8.3) |
| **MAS Notice 655** | ⚠ **Flagged** — the legacy Banking-Act TRM notice, **cancelled 9 May 2024** and superseded by FSM-N05 under the FSMA 2022 ✅ (§8.4) |
| **ISO 27001** | ISO/IEC 27001:2022 — the certifiable information-security management-system standard (§8.6) |
| **NIST CSF** | The NIST Cybersecurity Framework 2.0 (26 February 2024) — six functions: Govern, Identify, Protect, Detect, Respond, Recover ✅ (§8.6) |

---

## Claims Status and Verification Notes

**Verification pass: 2026-08-24, live web access via the self-hosted Firecrawl backend (web search + direct page extraction; the search backend was intermittently rate-limited — several queries returned empty and were retried via direct extraction of primary pages, which succeeded).** Primary sources verified live this pass: **owasp.org** (the Top 10:2021 landing page and the A01 page — the A01 statistics: 94% of applications tested, 3.81% average incidence, 318k+ occurrences — and the A01–A10 list; the ASVS project page and the 4.0.3 PDF — the 2008-to-2016 lineage note), **cwe.mitre.org** (the 2024 CWE Top 25 — #1 CWE-79 XSS, #2 CWE-787 out-of-bounds write, #3 CWE-89 SQL injection, #4 CWE-352 CSRF, with the KEV counts), **slsa.dev** (the framework definition, the four levels, the OpenSSF stewardship, the EO 14028 references), **the DORA tracking site** (Regulation (EU) 2022/2554's pillars per Article 1, and the 17 January 2025 applicability — multiple dated entries), **EUR-Lex and ENISA** (Directive (EU) 2022/2555 of 14 December 2022, OJ L 333 27.12.2022, repealing 2016/1148; the 17 October 2024 transposition deadline), **csrc.nist.gov and nist.gov** (CSF 2.0 released 26 February 2024; the six-function core with the new Govern function; CSF 1.0's 2014 origin), **csa.gov.sg** (the Cybersecurity Labelling Scheme; the Safer Cyberspace Masterplan 2020's three thrusts), **regalert.today and the FSM-N05 text** (the 9 May 2024 cancellation of Notices 644, 655, 644A, 655A, 1114, 1118, effective 10 May 2024; FSM-N05 issued under section 29(1) of the FSMA 2022), **schneier.com** (the original December 1999 "Attack Trees" article), **Wikipedia** (the STRIDE model; DREAD's five dimensions), **threat-modeling.com** (PASTA: the acronym, the VerSprite/UcedaVélez & Morana attribution, the seven-stage structure), and the practitioner sources for the DREAD and SAST/DAST/IAST/SCA taxonomies (EC-Council, OWASP, Checkmarx, CrowdStrike, Microsoft, SentinelOne).

**✅ Verified this pass (live sources):** STRIDE's six categories and Microsoft origin (the "late 1990s" framing; the 1999 Kohnfelder & Garg memo attribution is uniformly cited ⚠-structural at the memo level); DREAD's five dimensions; PASTA's seven-stage structure and creators; the attack-trees origin (Schneier, Dr. Dobb's Journal, December 1999); the OWASP Top 10 2021 list and the A01 statistics; the ASVS project identity and level structure (v4 lineage; the v5.0 release date is ⚠); the CWE Top 25 2024 top entries with KEV counts; SLSA's definition and four levels; DORA's Regulation number, pillars and 17 January 2025 applicability; NIS2's number, dates and transposition deadline; CSF 2.0's release date and six functions; CSA's CLS and masterplans; and the MAS Notice 655 cancellation/supersession (FSM-N05).

**⚠ Flagged / unverified this pass (honest ledger):** the **shift-left** term's origin (⚠-structural, §1.2); the **cost-multiplier** per lifecycle stage (⚠-structural, §1.2); the **Microsoft SDL** history detail (⚠-structural, §1.2); **DREAD's exact rating scale** (0–10 vs 1–10 across sources; Wikipedia documents 1–10, §2.3) and Microsoft's deprecation of it (⚠-structural); **PASTA stage-name wording** (⚠-structural, §2.4); **LINDDUN** (⚠-structural, §2.6); the **shared-responsibility model's** single-source attribution (⚠-structural — AWS/Azure/GCP doctrine, no live extraction this pass, §7.1); **Gartner's coinage of CSPM/CWPP** with exact years (⚠-structural, §7.2); the **NTIA SBOM minimum elements** wording and the SLSA origin date (⚠-structural, §6.4–6.5); the **DORA TLPT mechanics** (⚠-structural, §8.1) and any DORA-specific log-retention requirement (⚠, §9.2 finding R); **NIS2's exact fine bands** (⚠-structural, §8.2); the **CSA's founding date** and the **Cybersecurity Act 2018** details (⚠-structural, §8.3); **MAS Notice 655's original 2018 issue / 16 August 2019 effective dates** (⚠-structural — widely documented, not live-verified, §8.4); the **MAS Notice 645** migration detail (⚠-structural, §8.4); **ISO/IEC 27001:2022** publication-date detail (⚠-structural, §8.6); the **industry adoption statistics** (deliberately not quoted, §7.1); the **ASVS v5.0** release (⚠, §4.3); and **any Cymbal Bank-internal system details** (⚠ — the §9 worked example is an illustrative scenario in the house's context, not a description of any real Cymbal Bank system).

**Repository convention:** ✅ = verified this pass or in the cross-referenced guide's ledger; ⚠ = flagged/unverified; ⚠-structural = standard industry practice widely documented, not attributable to a single primary source verified this pass.

---

## Cross-References and Further Reading

**The security cluster (sibling, technology/):**
- **[distributed_auth_guide.md](distributed_auth_guide.md)** — **THE identity companion.** Its authentication/authorization machinery (OAuth2/OIDC, scopes, claims, mTLS) is §2.2's spoofing/elevation and §3.2's least privilege in full; §9's findings #1, #3, #7 read its chapters.
- **[api_governance_guide.md](api_governance_guide.md)** — **THE API companion.** API security, gateway policy and schema governance underpin §2.2's API flavour of STRIDE and §9's gateway findings.
- **[llm_development_risks_security_guide.md](llm_development_risks_security_guide.md)** and **[adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md)** — the GenAI/ML security angles: prompt injection and model attacks are the 2025 extension of §4's coding table and §5's pentest scope.
- **[zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md)** — **THE resilience companion.** The fail-secure vs availability tension (§3.4), the graceful-degradation framing and the idempotency pattern (§9 finding #4) all cross-ref its chapters.
- **[data_architect_skillgaps_guide.md](data_architect_skillgaps_guide.md)** — **the standing-priority frame.** Its gap #3 — Zero Trust for data platforms (NIST SP 800-207, data-plane ZTA, MAS TRM/N645 alignment) — is the §3.7 tie-in; this guide is the discipline-side contribution to that priority.
- **[enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)** — the AI-governance angle (the identity-aware proxy pattern of §3.7 in its AI form; light cross-ref).
- **[singapore_security_clearance_guide.md](singapore_security_clearance_guide.md)** — the SG personnel angle behind §3.6's separation of duties (light cross-ref).
- **[openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)**, **[singapore_data_centres_guide.md](singapore_data_centres_guide.md)** — the platform and SG-DC frames behind §7's cloud/estate discussion (light).

**The banking cluster (prefix `../banking/`):**
- **[../banking/regtech_guide.md](../banking/regtech_guide.md)** — **THE compliance-angle companion.** The regulatory-reporting machinery behind §8's register obligations (DORA's Register of Information, the FSM-era notices).
- **[../banking/financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md)** — the op-risk taxonomy that DORA reshapes (§8.1) and where SoD failures land as operational-risk events (§3.6).
- **[../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md)** — the house systems context for §9's scenario.
- **[../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md)** and the bank software-systems series — the estate precedents for the §9 application type.

**The management cluster (prefix `../management/`):**
- **[../management/business_case_development_guide.md](../management/business_case_development_guide.md)** — the security-investment angle: the cost of designing security in vs the cost of the breach it prevents (§10 — light cross-ref).

**Primary sources and further reading:** Saltzer & Schroeder, "The Protection of Information in Computer Systems" (CACM 18(7), 1975); Clark-Wilson, "A Comparison of Commercial and Military Computer Security Policies" (1987); the Microsoft STRIDE lineage (the 1999 Kohnfelder & Garg memo; Shostack, *Threat Modeling: Designing for Security*, Wiley 2014); Schneier, "Attack Trees" (*Dr. Dobb's Journal*, December 1999 — schneier.com); UcedaVélez & Morana, *Risk Centric Threat Modeling* (Addison-Wesley 2015); the OWASP Top 10 2021 (owasp.org/Top10/2021), the OWASP ASVS (owasp.org), the OWASP SAMM; the CWE Top 25 (cwe.mitre.org, 2024 edition); the CERT/SEI secure-coding standards; the NTIA SBOM minimum elements and EO 14028; SLSA (slsa.dev); NIST SP 800-207 (Zero Trust Architecture, August 2020); the NIST CSF 2.0 (CSWP 29, 26 February 2024); ISO/IEC 27001:2022; Regulation (EU) 2022/2554 (DORA) and Directive (EU) 2022/2555 (NIS2) on EUR-Lex; ENISA's NIS2 pages; csa.gov.sg (the CLS, the Safer Cyberspace Masterplan, the OT Cybersecurity Masterplan); the MAS regulatory record (the 9 May 2024 TRM-notice cancellations; Notice FSM-N05); and the repo's own verified cluster listed above.

---

*End of guide — security is a feature: the systems that survive are the ones where it was designed in from the first line.*

