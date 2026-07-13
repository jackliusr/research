# China's Data Governance Frameworks and Regulations: A Comprehensive Guide

> **Author**: Jack Liu Shurui  
> **Repository**: [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated**: July 2026

---

## Table of Contents

1. [Overview: The Three Pillars](#1-overview-the-three-pillars)
2. [Cybersecurity Law (CSL) 2017](#2-cybersecurity-law-csl-2017)
3. [Data Security Law (DSL) 2021](#3-data-security-law-dsl-2021)
4. [Personal Information Protection Law (PIPL) 2021](#4-personal-information-protection-law-pipl-2021)
5. [Cross-Border Data Transfer Rules](#5-cross-border-data-transfer-rules)
6. [Data Classification and Grading System](#6-data-classification-and-grading-system)
7. [Data Localisation Requirements by Industry](#7-data-localisation-requirements-by-industry)
8. [Enforcement and Penalties](#8-enforcement-and-penalties)
9. [Sector-Specific Regulations](#9-sector-specific-regulations)
10. [Algorithmic Regulation](#10-algorithmic-regulation)
11. [Extraterritorial Reach](#11-extraterritorial-reach)
12. [Comparison: PIPL vs GDPR vs Singapore PDPA](#12-comparison-pipl-vs-gdpr-vs-singapore-pdpa)
13. [Practical Compliance Recommendations](#13-practical-compliance-recommendations)
14. [Future Trends](#14-future-trends)

---

## 1. Overview: The Three Pillars

China's data governance framework rests on three foundational statutes enacted between 2017 and 2021, collectively known as the **"Three Pillars"**:

| Law | Effective | Primary Focus |
|-----|-----------|---------------|
| Cybersecurity Law (CSL) | June 2017 | Network security, CII, data localisation |
| Data Security Law (DSL) | Sept 2021 | Data classification, security obligations, export controls |
| Personal Information Protection Law (PIPL) | Nov 2021 | PI protection, consent, cross-border transfers, individual rights |

These are supplemented by dozens of implementing regulations, national standards (GB), and sector-specific rules issued by the Cyberspace Administration of China (CAC), MIIT, PBOC, and other regulators.

**Core Regulatory Philosophy**: China's approach differs fundamentally from Western privacy frameworks. Data governance is framed as a matter of **national security first**, with the state asserting **data sovereignty** over data generated within its territory. The government directly classifies high-value datasets (important data, core data) rather than leaving classification solely to data controllers.

**Key Regulatory Bodies**: CAC (lead data regulator), MIIT (telecom/industrial data), Ministry of Public Security (criminal enforcement), PBOC (financial data), National Health Commission (healthcare data), Ministry of Science and Technology (human genetic resources).

---

## 2. Cybersecurity Law (CSL) 2017

### 2.1 Overview

Effective 1 June 2017, China's first comprehensive cybersecurity legislation. Applies to all "network operators" (any entity owning or operating networks) with heightened requirements for **Critical Information Infrastructure (CII)** operators.

### 2.2 Critical Information Infrastructure (CII)

CII refers to network facilities and systems whose compromise could seriously endanger national security, the economy, or public welfare. Designated sectors include: public communications, energy, transportation, finance, water conservancy, e-government, and national defence. Operators in these sectors may be formally designated as CII operators by sector regulators.

### 2.3 Key Requirements

- **Data Localisation**: CII operators must store "important data" and "personal information" collected in China on domestic servers. Cross-border transfers require CAC security assessment.
- **National Security Reviews**: CII operators must undergo review when procuring network products/services that may affect national security (Cybersecurity Review Measures, 2022/2024).
- **Multi-Level Protection Scheme (MLPS/等级保护 2.0)**: All network operators must classify information systems into five security protection levels and implement corresponding safeguards.
- **Incident Reporting**: Immediate reporting of cybersecurity incidents to regulators with remedial action.

### 2.4 Penalties for Non-Compliance

- General violations: Fines up to RMB 1,000,000 for entities, RMB 100,000 for individuals; warnings; rectification orders.
- Serious violations: Fines up to RMB 1,000,000, suspension of operations, revocation of licenses.
- CII-specific violations: Enhanced penalties including fines of RMB 100,000–1,000,000 and potential business suspension.

---

## 3. Data Security Law (DSL) 2021

### 3.1 Overview

Effective 1 September 2021, the DSL is a horizontal law covering all data processing activities. It establishes a state-led data classification system, security assessment obligations, and data export controls.

### 3.2 Data Classification System

Data is categorised into three tiers based on harm to national security and public interests:

| Category | Impact if Compromised | Examples |
|----------|----------------------|----------|
| **Ordinary Data** | Minor; no national security impact | General business records |
| **Important Data** | Harm to national security, economy, or public order | Health data, financial market data, sector-specific catalogues |
| **Core Data** | Severe harm to national security or critical state functions | National defence data, state secrets, national core data |

China is the first major jurisdiction to make "important data" a defined legal concept. Sector-specific authorities publish **Important Data Catalogues** — as of 2026, catalogues exist for finance, automotive, telecom, healthcare, and transportation.

### 3.3 Security Assessment Obligations

Data processors must:
1. Conduct data security assessments for processing activities affecting national security or public interests.
2. Establish security management systems with designated data security officers.
3. Perform annual risk assessments with results reported to regulators.
4. Implement safeguards appropriate to data classification level.

### 3.4 Data Export Controls

Data that could impact national security may be subject to export licensing. Important/core data exports require security assessment. Unauthorised export can result in fines up to RMB 10 million and criminal liability.

### 3.5 Government Data Access

The DSL explicitly permits government agencies to request data for national security or criminal investigations. Processors must comply — a provision creating tension with PIPL consent requirements.

### 3.6 Penalties

| Violation | Entity Fine | Individual Fine |
|-----------|-------------|-----------------|
| General security violations | Up to RMB 500,000 | Up to RMB 100,000 |
| Core data breaches | RMB 500,000–10,000,000 | RMB 50,000–200,000 |
| Unauthorised data export | RMB 1,000,000–10,000,000 | RMB 100,000–1,000,000 |

Additional sanctions: suspension of operations, revocation of permits, criminal prosecution.

---

## 4. Personal Information Protection Law (PIPL) 2021

### 4.1 Overview

Effective 1 November 2021, China's first comprehensive privacy law modelled on the GDPR. Governs collection, storage, use, processing, transfer, and disclosure of personal information of individuals in China.

**Key Definitions**: Personal Information (PI) = any information related to an identified/identifiable natural person. Sensitive PI = biometrics, religion, identity, health, finance, location, minors under 14. PI Handler = entity determining purposes and means of processing. Entrusted Processor = third-party processor.

### 4.2 Consent Requirements

- **Informed, Voluntary, Explicit Consent** is the primary legal basis (opt-in required; no pre-checked boxes).
- **Separate Consent (单独同意)**: Required for sensitive PI, cross-border transfers, disclosure to other handlers, and public data processing. Blanket consent is insufficient.
- **Right to Withdraw**: Individuals can withdraw consent at any time; handlers cannot withhold services as a condition.
- **Parental Consent**: Required for minors under 14.
- **No "Legitimate Interests" Basis**: Unlike GDPR, PIPL does not include legitimate interests as a legal basis — a critical difference.

### 4.3 Rights of Data Subjects

PIPL grants 11+ data subject rights, broader than GDPR in several respects:

| Right | Description | Article |
|-------|-------------|---------|
| **Know** | Be informed of processing purposes, methods, scope, retention, and handler's contact | 44, 47 |
| **Decide** | Autonomously decide whether to allow processing | 44 |
| **Restrict** | Restrict processing in certain circumstances | 44 |
| **Refuse** | Refuse processing for automated decision-making, marketing, push notifications | 24, 44 |
| **Access** | Access and obtain a copy of their PI | 45 |
| **Portability** | Request transfer to another designated handler (where technically feasible) | 45 |
| **Rectify** | Correct inaccurate or incomplete PI | 46 |
| **Delete** | Request deletion when purposes achieved, consent withdrawn, or processing unlawful | 47 |
| **Withdraw Consent** | Withdraw consent without affecting prior processing legality | 15 |
| **Explanation** | Request explanation of processing rules and automated decision-making logic | 48 |
| **Deceased (Right to Die)** | Next of kin may exercise access/rectification/deletion for their own lawful interests | 49 |

### 4.4 Sensitive Personal Information

Enhanced requirements: (1) specific purpose limitation, (2) separate consent, (3) personal information protection impact assessment (PIPIA), (4) tighter technical safeguards, (5) strict necessity principle. Minors under 14 are treated as sensitive PI.

### 4.5 Penalties

| Level | Entity Fine | Individual Fine | Additional |
|-------|-------------|-----------------|------------|
| General | Up to RMB 5,000,000 | RMB 10,000–100,000 | Rectification order |
| Serious | **Up to RMB 50,000,000 OR 5% of annual revenue** | RMB 100,000–1,000,000; industry ban | Suspension, license revocation, social credit blacklisting |

### 4.6 DPO Requirement

Handlers processing large volumes of PI must appoint a **personal information protection officer** responsible for supervising processing activities and serving as regulatory contact.

---

## 5. Cross-Border Data Transfer Rules

### 5.1 Three Transfer Mechanisms

| Mechanism | Trigger | Description |
|-----------|---------|-------------|
| **CAC Security Assessment** | CII operators; handlers transferring important data; processing PI of ≥1M individuals or sensitive PI of ≥10K individuals (cumulative) | Full regulatory review; approval required pre-transfer |
| **Standard Contractual Clauses (SCCs)** | Below security assessment threshold | CAC-prescribed contract; filed with provincial CAC within 10 working days |
| **Certification** | Alternative to SCCs (especially intra-group) | Third-party certification from accredited body; finalised Nov 2025 |

### 5.2 2024 Reforms & Relaxations

The March 2024 *Provisions on Promoting and Regulating Cross-border Data Flows* introduced:
- Exemptions for transfers necessary for international trade, cross-border e-commerce, certain HR activities.
- Exemption from security assessment for transfers of <100,000 individuals' PI over 12 months.
- "Negative list" approach in Free Trade Zones (FTZs) allowing certain transfers without assessment.
- Elimination of mandatory data localisation for most non-CII operators.

### 5.3 PIPIA Requirement

Regardless of mechanism, a Personal Information Protection Impact Assessment must be conducted before any cross-border transfer, addressing: necessity, individual rights impact, overseas recipient safeguards, and recipient's Chinese law compliance.

### 5.4 Practical Challenges

1. Threshold uncertainty (calculation of cumulative PI volumes across group entities).
2. Incomplete important data catalogues (many sectors still drafting).
3. CAC assessment delays (6–18 months for security assessments).
4. Human Genetic Resources (HGR) requiring separate MoST approval.
5. Overseas recipient enforcement risk (blacklisting, contractual obligations under Chinese law).
6. Certification route still maturing (accredited bodies and detailed procedures evolving).

---

## 6. Data Classification and Grading System

### 6.1 National Framework (MLPS 2.0 — Levels 1–5)

| Level | Designation | Impact of Breach | Security Requirements |
|-------|-------------|------------------|----------------------|
| L1 | Independent | Minor individual/organisational impact | Basic controls |
| L2 | Guided Protection | Moderate individual/minor public harm | Basic controls + guided supervision |
| L3 | Supervised Protection | Serious individual/moderate public harm | Mandatory assessment, enhanced controls |
| L4 | Mandatory Protection | Major public interest/national security | Strictest controls, mandatory encryption |
| L5 | Ultra-High Protection | Extremely grave national security harm | State-level controls, network isolation |

### 6.2 Industry-Specific Standards

| Sector | Standard | Key Provisions |
|--------|----------|----------------|
| **Finance** | PBOC Financial Data Security Guidelines | 5 levels; L4+ data restricted from cross-border transfer |
| **Automotive** | Automotive Data Security Provisions (2022) | 6 categories; important data includes geospatial data exceeding volume thresholds |
| **Telecom/IT** | MIIT Data Security Measures (2023) | Annual reporting; classification by economic/national security impact |
| **Healthcare** | Healthcare Data Security Guidelines | Medical records and HGR as important data; strict access controls |
| **Transportation** | Transportation Data Classification Measures | Real-time traffic data as important data |

---

## 7. Data Localisation Requirements by Industry

| Industry | Requirement | Legal Basis |
|----------|-------------|-------------|
| **Finance** | All financial/transaction/credit data stored in China; offshoring requires PBOC approval | PBOC Financial Data Security Guidelines; Credit Information Measures |
| **Healthcare** | Medical records, health data, HGR stored domestically; HGR transfer requires MoST approval | Healthcare Data Guidelines; HGR Regulations |
| **Transportation** | Real-time traffic, logistics, operational data stored in China | Transportation Data Management Measures |
| **Telecom** | Network data and user information stored domestically | Telecom User Information Protection Provisions |
| **Automotive** | Vehicle operational, location, sensor data stored in China; limited cross-border for research/after-sales | Automotive Data Security Provisions |
| **Mapping/Geospatial** | All surveying/mapping data stored and processed domestically | Surveying and Mapping Law; Geospatial Data Regulations |

CII operators across all sectors face mandatory localisation for personal information and important data.

---

## 8. Enforcement and Penalties

### 8.1 Major Enforcement Actions

| Company | Year | Penalty | Violation | Regulator |
|---------|------|---------|-----------|-----------|
| **Didi Global** | 2022 | RMB 8.026B (~USD 1.2B) | CII violation; collected 64.7B pieces of PI; cross-border transfer without security assessment | CAC |
| **CNKI (知网)** | 2022 | RMB 50M | Illegal PI collection; consent failure; algorithmic non-compliance | CAC |
| **Mysteel** | 2024 | RMB 33M | Unauthorised cross-border transfer of important data (commodity prices) | CAC Shanghai |
| **25 ride-hailing apps** | 2021 | App store removals | PI collection beyond stated scope; portability denial | CAC, MIIT |
| **Multiple fintech firms** | 2023–2025 | Various enforcement actions | Inadequate consent; failure to conduct data security assessments | PBOC, CAC |

### 8.2 Enforcement Trends (2024–2026)

1. **Systematic Sector Inspections**: Enforcement has shifted from high-profile individual cases (Didi) to sector-wide audits in finance, healthcare, and ride-hailing.
2. **Provincial CAC Activity**: Local regulators increasingly conducting on-site audits and handling routine cross-border filings.
3. **Criminal Prosecution Rising**: More individuals facing prison sentences for data breaches and unauthorised data sales.
4. **Social Credit Integration**: Data violations now trigger blacklisting, affecting business operations and government procurement eligibility.
5. **Cross-Regulator Coordination**: Joint actions by CAC, MIIT, PBOC, and SAMR becoming standard.

---

## 9. Sector-Specific Regulations

### 9.1 Finance

Most mature sector-specific regime. PBOC mandates five-level financial data classification; L4+ data cannot be transferred abroad. Credit data must go through licensed reference agencies. Payment data must be stored domestically. Big tech financial services face additional requirements including data localisation and separation of financial data from other business data.

### 9.2 Healthcare

Medical/health data classified into five levels under Healthcare Data Security Guidelines. **Human Genetic Resource (HGR) Regulations** impose a separate approval system under MoST — collection and cross-border transfer of DNA samples and genetic sequences require specific approvals and risk-benefit assessments. Electronic medical records must be stored in China for 15 years (outpatient) or 30 years (inpatient).

### 9.3 Automotive

Automotive Data Security Management Provisions (2022) define six categories of vehicle data. Important data includes geospatial data covering >1 km² at >10m accuracy, vehicle trajectory data >100 km, and biometric data (face, voice). Vehicle data must be stored domestically; cross-border transfer limited to research, certification, and after-sales. Foreign companies restricted from collecting high-precision map data directly.

### 9.4 Telecommunications

MIIT Data Security Measures (2023) cover telecom operators, data centres, cloud providers, and internet platforms. Data security assessments must be submitted annually. Foreign cloud providers must operate through Chinese joint ventures.

---

## 10. Algorithmic Regulation

### 10.1 Algorithmic Recommendation Management Provisions (March 2022)

- **Scope**: Recommendation algorithms used in internet information services (content feeds, search, product recommendations, rankings).
- **Filing**: Algorithms "with public opinion attributes" must be filed with CAC including algorithm descriptions and self-assessments.
- **User Control**: Users can turn off personalised recommendations, delete interest tags, and obtain explanations.
- **Prohibitions**: No price discrimination via user profiling; no dissemination of illegal content; no undesirable values.
- **Labour Protection**: Platform algorithms for gig workers must comply with labour protection standards.

### 10.2 Deep Synthesis Provisions (January 2023)

- **Scope**: AI-generated/synthetic content (deepfakes, generative AI — text, audio, image, video).
- **Labelling**: All AI content must be conspicuously labelled as synthetic.
- **Consent**: Real individuals' likenesses require explicit consent for synthetic content.
- **Training Data**: Must not infringe IP or personal rights.
- **Content Audit**: Providers must prevent generation of illegal content.

### 10.3 Generative AI Interim Measures (August 2023)

- Content must align with socialist core values.
- Training data must be from legitimate sources, IP-compliant, and non-discriminatory.
- Providers must improve content accuracy and reliability.
- Users must provide real identity information for registration.

---

## 11. Extraterritorial Reach

### 11.1 When PIPL Applies to Foreign Companies

PIPL applies to any entity processing PI of **individuals in China** in connection with:
1. Providing products or services to individuals in China.
2. Analysing or evaluating behaviour of individuals in China (profiling, behavioural advertising).
3. Other circumstances specified by law.

**Practical Indicators**: Chinese-language websites/apps, RMB payment methods, .cn domains, China-targeted marketing, local subsidiaries, processing data about Chinese nationals regardless of where processing occurs.

### 11.2 Obligations for Foreign Entities

Foreign companies subject to PIPL must:
- Designate a **representative in China** (subsidiary, third-party, or established office).
- Comply with cross-border transfer mechanisms.
- Provide same data subject rights as domestic handlers.
- Cooperate with CAC investigations.

### 11.3 Comparison with GDPR Extraterritorial Reach

| Dimension | PIPL | GDPR |
|-----------|------|------|
| Trigger | Processing PI of individuals "in China" (territorial + behavioural) | Targeting/monitoring EU data subjects |
| Representative | Mandatory local representative | Mandatory local representative (Art. 27) |
| Same obligations | Yes | Yes |
| HGR regulation | Applies to any entity collecting HGR from Chinese donors | No equivalent |

---

## 12. Comparison: PIPL vs GDPR vs Singapore PDPA

### 12.1 High-Level Comparison

| Dimension | China PIPL | EU GDPR | Singapore PDPA |
|-----------|-----------|---------|----------------|
| **Effective** | 1 Nov 2021 | 25 May 2018 | 2014 (amended 2020) |
| **Regulator** | CAC (primary), PBOC, MIIT (sectoral) | National DPAs (varies) | PDPC |
| **Max Penalty** | 5% annual revenue OR RMB 50M | 4% global turnover OR €20M | 10% annual turnover OR SGD 1M |
| **Consent Model** | Strict opt-in; separate consent for sensitive | Opt-in; explicit for special categories | Opt-out for marketing (DNC); opt-in generally |
| **Legitimate Interests Basis** | No | Yes | Limited |
| **Data Localisation** | Yes (CII mandatory; sector-specific) | No | No |
| **Cross-Border Transfer** | Security Assessment / SCCs / Certification | Adequacy Decision / SCCs / BCRs | Comparable protection standard required |
| **Breach Notification** | Mandatory (timeframe not explicitly defined) | Mandatory (72 hours to DPA) | Mandatory (as soon as practicable) |

### 12.2 Detailed Comparison (10+ Dimensions)

**1. Personal Information / Data Definition**: PIPL broadest — includes re-identifiable anonymised data. GDPR excludes truly anonymised data and deceased persons' data. PDPA excludes deceased individuals (limited 10-year post-death protection via recent amendments).

**2. Legal Bases**: PIPL = consent (primary), contract, legal obligation, public health, research, news media — **no legitimate interests**. GDPR = 6 bases including flexible legitimate interests. PDPA = consent (primary), deemed consent, legitimate interests exception, business improvement exception.

**3. Sensitive Data**: PIPL has specific "sensitive PI" category with separate consent. GDPR has "special categories" with explicit consent. PDPA — no separate sensitive data category in primary legislation (advisory guidelines exist for health, NRIC, biometrics).

**4. Data Subject Rights**: PIPL = 11+ rights including right to restrict, refuse, and deceased data rights (unique). GDPR = 8 rights (access, rectification, erasure, restriction, portability, object, automated decisions, information). PDPA = 5 primary rights (access, correction, consent withdrawal, portability (limited), information) — no erasure right.

**5. Consent Requirements**: PIPL most stringent — separate consent for sensitive PI, cross-border, and disclosure. Cannot condition service on unrelated consent. GDPR — consent must be freely given, specific, informed. PDPA — deemed consent permitted; notification obligation where consent is impractical.

**6. Cross-Border Transfers**: PIPL — government pre-approval required (security assessment). GDPR — no pre-approval for SCCs (Schrems II introduced transfer impact assessments). PDPA — least restrictive; comparable protection standard required, no pre-approval.

**7. DPO Requirement**: PIPL — mandatory for large-volume handlers. GDPR — mandatory for public bodies, large-scale monitoring, large-scale special categories. PDPA — no mandatory DPO (must designate one compliance individual).

**8. Breach Notification**: PIPL — mandatory (no defined timeframe; best practice 24–48 hrs). GDPR — 72 hours to DPA, individuals if high risk. PDPA — as soon as practicable; to PDPC and individuals if significant harm.

**9. Automated Decision-Making**: PIPL — most comprehensive: right to refuse, right to explanation, algorithmic filing, price discrimination prohibition. GDPR — Article 22 (solely automated decisions producing legal effects), right to human intervention. PDPA — no specific provisions (proposed AI governance amendments pending).

**10. Enforcement Style**: PIPL — aggressive, multi-agency, social credit integration, operational suspensions. GDPR — decentralised, increasingly aggressive fines but varies by member state. PDPA — advisory and conciliatory historically (increasing fines 2022–2025); no criminal sanctions.

**11. Regulatory Certainty**: PIPL — low (regulations evolving; catalogues incomplete). GDPR — moderate (core stable; Schrems II created SCC uncertainty). PDPA — moderate-high (mature regime with established PDPC guidance).

**12. Deceased Persons Data**: PIPL — yes (right of the deceased; next of kin rights). GDPR — no (excluded by definition). PDPA — no (limited post-death protection added 2020).

---

## 13. Practical Compliance Recommendations

### 13.1 Data Mapping and Inventory
1. Conduct comprehensive data mapping covering all PI, important data, and core data in China operations.
2. Document data flows including third-party sharing, cross-border transfers, and intra-group sharing.
3. Classify data using DSL framework and sector-specific catalogues.
4. Establish a cross-functional data governance committee (legal, compliance, IT, business).

### 13.2 DPO and Local Entity
1. Appoint a China-based Personal Information Protection Officer with sufficient authority and resources.
2. Establish a local legal entity (WFOE) as the primary PI handler in China.
3. Register the DPO as accessible to CAC.
4. Foreign entities without China presence must designate a local representative.

### 13.3 Cross-Border Transfer Strategy
1. Determine whether CAC security assessment, SCCs, or certification applies based on PI volumes and data sensitivity.
2. Evaluate certification route (finalised 2025) for intra-group operational transfers.
3. Leverage FTZ negative lists if operations can be channelled through free trade zones.
4. Implement CAC-prescribed SCCs with all overseas recipients.
5. Conduct PIPIAs before each cross-border transfer, documenting necessity and safeguards.
6. Monitor regulatory developments — the transfer regime continues to evolve rapidly.

### 13.4 Data Classification Implementation
1. Develop internal policies aligned with national and sector-specific classification standards.
2. Implement technical controls per level: L1–2 (basic controls, encryption) → L3 (enhanced access, audit logging, mandatory encryption) → L4–5 (government-grade protection, network segmentation).
3. Label data assets with classification tags across databases, filesystems, and applications.
4. Train employees on classification requirements and handling procedures.
5. Conduct regular classification audits, especially when new data types or processing activities emerge.

### 13.5 Consent and Rights Management
1. Implement granular consent mechanisms for separate consent requirements.
2. Provide clear Simplified Chinese privacy notices covering all statutory elements.
3. Build automated systems for handling rights requests within statutory timeframes.
4. Enable easy consent withdrawal, opt-out of personalised recommendations, data access/portability, and deletion requests.
5. Maintain documented consent records for regulatory audits.

### 13.6 Vendor Management
1. Conduct due diligence on all vendors processing PI or important data.
2. Contractually obligate vendors to comply with CSL, DSL, PIPL, and sector-specific requirements.
3. Include audit rights in vendor contracts.
4. Assess subcontractors for additional due diligence requirements.
5. Restrict foreign vendors from directly accessing important data or core data.

### 13.7 Incident Response
1. Establish a China-specific data breach response plan with:
   - Internal escalation within 1 hour of detection.
   - Regulatory notification to CAC (best practice: 24–48 hours).
   - Individual notification for breaches causing potential personal or property harm.
2. Conduct tabletop exercises simulating cross-border data breaches involving sensitive PI.
3. Retain China-based legal counsel and incident response firms.
4. Prepare crisis communications templates in Simplified Chinese.

### 13.8 Ongoing Compliance
1. Conduct annual self-assessments as required by DSL and PIPL.
2. Commission third-party audits for high-risk processing activities.
3. Maintain records ready for regulatory inspection at any time.
4. Assign a team member to monitor new regulations, CAC guidance, and enforcement actions.

---

## 14. Future Trends

### 14.1 Data Security Review (DSR)
The expanded Cybersecurity Review Measures (2024) include DSR for: (1) cross-border data processing affecting national security, (2) platform operators with >1M PI users seeking foreign listings, (3) CII operators procuring network products. DSR is expected to become more prominent as a regulatory tool.

### 14.2 Data Factor Market (数据要素市场)
China is treating data as a production factor akin to labour and capital. The "20 Articles on Data" (2022) advocate data property rights systems. Over 50 data exchanges are being formalised with standardised trading rules. The Ministry of Finance issued data asset accounting standards (2024). Companies may monetise data products through exchanges, subject to privacy and security compliance.

### 14.3 Privacy Computing Mandates
Privacy-enhancing technologies (federated learning, secure multi-party computation, TEE, differential privacy) are increasingly mandated as the technical solution to the "privacy-security-utility" trilemma. Multiple GB standards on privacy computing are under development. The PBOC explicitly encourages privacy computing for financial data sharing.

### 14.4 AI Regulation Evolution
China is building a comprehensive AI governance framework alongside data governance: further regulation of foundation models, training data disclosure, and content watermarking expected. Potential legislation on AI civil liability. China promoting its governance model through the Global AI Governance Initiative.

### 14.5 Enforcement Trajectory
Systemic sector-wide inspections will continue. Criminal liability for individuals will increase. Social credit integration for data compliance will expand. Cross-regulator joint enforcement (CAC + MIIT + PBOC + SAMR) will become standard.

### 14.6 Opening vs Security Tension
Despite 2024's practical liberalisation (exemptions for small-volume transfers, FTZ negative lists), the fundamental data sovereignty posture is unlikely to change. Lingang FTZ's permissive experiments may expand nationally if successful. China's engagement in RCEP and DEPA may influence future approaches.

### 14.7 Cross-Border Transfer Reform Outlook
Expect: higher PI thresholds for security assessment, expanded FTZ negative lists, wider adoption of the certification route (finalised 2025), and potential bilateral/sectoral data transfer agreements (e.g., cross-border clinical trials, automotive certification).

---

## Visual Overview

```
┌──────────────────────────────────────────────────────────────┐
│                 CHINA'S DATA GOVERNANCE                       │
│                      FRAMEWORK                                │
│   ┌──────────────────────────────────────────────────┐      │
│   │              THE THREE PILLARS                    │      │
│   ├─────────┬──────────────┬────────────────────────┤      │
│   │  CSL    │     DSL      │         PIPL            │      │
│   │ 2017    │    2021      │        2021              │      │
│   ├─────────┴──────────────┴────────────────────────┤      │
│   │Network Security│ Data Security │ PI Protection    │      │
│   └──────────────────────────────────────────────────┘      │
│                                                              │
│   ┌──────────────────────────────────────────────────┐      │
│   │           CROSS-BORDER DATA TRANSFER              │      │
│   │  ┌─────────┬──────────┬────────────────┐         │      │
│   │  │Security │   SCC    │ Certification   │         │      │
│   │  │Assessmt │  Route   │    Route         │         │      │
│   │  └─────────┴──────────┴────────────────┘         │      │
│   └──────────────────────────────────────────────────┘      │
│                                                              │
│   ┌──────────────────────────────────────────────────┐      │
│   │       SECTOR-SPECIFIC + AI REGULATION             │      │
│   │  Finance│Healthcare│Auto│Telecom│Algo│DeepSynth   │      │
│   └──────────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────────┘
```
