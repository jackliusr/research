# Operational Resilience: The Resilient Bank — A Comprehensive Guide

*The dedicated deep-dive on operational resilience — the discipline that assumes disruption will happen and asks the bank to prove it can keep its most important services running inside defined tolerances anyway. From the regulator's framing (prevent, respond, recover, learn) to the UK's impact-tolerance machinery, from MAS's service-centric BCM Guidelines to the EU's DORA, from the business impact analysis to the disaster-recovery tier, ending in a worked operational-resilience programme for a Cymbal Bank.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore

**Series:** Banking & Financial Technology Guides — this is the **operational-resilience anchor** of the risk/regulatory cluster. The sibling guides own the neighbouring disciplines: [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) owns the enterprise risk framework (three lines, risk appetite) this guide plugs into; [MAS Regulations, Guidelines and Industry Expectations Guide](mas_regulations_guidelines_guide.md) owns the full MAS instrument map (notices, guidelines, circulars); [Vendor Management Guide](../management/vendor_management_guide.md) owns the third-party-risk half of resilience (including the DORA mention); [Cybersecurity Guide](../technology/cybersecurity_guide.md) owns the cyber half; and the technology guides own the availability mechanics (zones, failover, backup, DR drills). This guide cross-references all of them instead of re-deriving their content.

**Audience:** Solution architects, risk and BCM technologists at a global bank — the recurring persona is **Cymbal Bank**, a global bank with its **Singapore APAC hub**, supervised by MAS locally while the group navigates the UK/EU rulebooks. The architect's question this guide answers: *what does "operational resilience" mean as a set of requirements on the systems, data, suppliers, and governance I build and operate?* The answer runs from the regulatory instruments down to the BIA outputs, the impact-tolerance metrics, the BCP/DR designs, and the board dashboard that proves the bank stays inside its tolerances.

**How this guide is organised:** the operational-resilience overview (definition, why it is a board topic, how it relates to — but is distinct from — operational risk, BCM, and cyber resilience); the regulatory landscape (MAS, PRA/FCA, DORA, BCBS, with a comparative table); the governance and accountability (board ownership, three lines, resilience-by-design); the business impact analysis; the important business services and the impact tolerances (IBS, MTPD, MBCO, RTO, RPO); the scenario-based risk assessment; the business continuity planning (BCP lifecycle, ISO 22301); the disaster recovery (DR tiers, RTO/RPO mapping); the crisis management and the communication; the third-party resilience and the supply chain; the testing and the assurance; a worked operational-resilience programme for a Cymbal Bank; a one-page summary ("the resilient bank"); the verification notes and claims audit; and the glossary with the series cross-references. Every section ends with a reference table.

**Scope note on verification:** regulatory facts in this guide were verified against primary sources in September 2026 — the MAS BCM Guidelines page and the MAS site, the PRA's PS6/21 and SS1/21 documents on bankofengland.co.uk, the FCA's operational-resilience page and PS21/3, the EIOPA DORA page, the BIS pages for the BCBS operational-risk and operational-resilience principles and the Joint Forum business-continuity principles, and ISO's catalogue pages for ISO 22301:2019 and ISO/TS 22317:2021. Where a fact could not be fully confirmed — including several claims inherited from the task brief that turned out to be misattributed (flagged in §14) — it is explicitly marked **[verify]** / ⚠ rather than asserted. The worked example's numbers are pedagogical constructions, not figures from any bank's disclosures.

**Cross-references used throughout:** [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) (the ERM framework, three lines, risk appetite), [MAS Regulations, Guidelines and Industry Expectations Guide](mas_regulations_guidelines_guide.md) (the MAS instrument map), [Banks in Singapore Guide](banks_in_singapore_guide.md) (MAS expectations for the Cymbal Bank context), [Vendor Management Guide](../management/vendor_management_guide.md) (third-party risk, outsourcing, the DORA mention), [Cybersecurity Guide](../technology/cybersecurity_guide.md) (cyber resilience), [OpenShift Workload Availability Guide](../technology/openshift_workload_availability_guide.md) (HA/DR/BC mechanics, DR drills), [IBM Cloud Guide](../technology/ibm_cloud_guide.md) (regions, multizone regions), [Securing Red Hat OpenShift Guide](../technology/secure_red_hat_openshift_guide.md) (platform security and backup), [Workshop Methodology Guide](../management/workshop_methodology_guide.md) (tabletop facilitation).

### Reading paths

- **Solution architects and engineers** — §1 (what resilience means as a design requirement), §5 (the metrics that become system requirements), §8 (DR tiers and RTO/RPO), §11 (testing), §12 (the worked programme). Pair with the OpenShift Workload Availability Guide for the mechanics.
- **Risk and BCM professionals** — §2 (the four regulators' expectations), §3 (governance), §4 (BIA), §7 (BCP lifecycle), §9 (crisis management), §14 (the claims audit).
- **Regulatory/compliance teams** — §2 (the instrument-by-instrument landscape and the comparative table), §14 (verification ledger with primary-source URLs).
- **General readers** — §1, §5, §13 (the one-page summary), and the glossary in §14.

Each section stands alone; the reference table at the end of each section is the quick-reference, and the cross-references let readers jump to the sibling guides without losing context.

---

## Table of Contents

1. The Operational Resilience Overview: Prevent, Respond, Recover, Learn
2. The Regulatory Landscape: MAS, PRA/FCA, DORA, BCBS
3. The Governance and the Accountability: Board Ownership and the Three Lines
4. The Business Impact Analysis: From Activities to Impact Tolerances
5. The Important Business Services and the Impact Tolerances
6. The Risk Assessment: Severe-but-Plausible Scenarios
7. The Business Continuity Planning: The BCP Lifecycle and ISO 22301
8. The Disaster Recovery: Tiers, RTO/RPO, and the Technology Estate
9. The Crisis Management and the Communication
10. The Third-Party Resilience and the Supply Chain
11. The Testing and the Assurance
12. The Worked Example: An Operational-Resilience Programme for a Cymbal Bank
13. The Summary: One Page — "The Resilient Bank"
14. Verification Notes and Sources, Glossary, and Cross-References in This Series

---

## 1. The Operational Resilience Overview: Prevent, Respond, Recover, Learn

### 1.1 What operational resilience is

**Operational resilience** is the capability of a firm — and of the financial system it sits in — to keep delivering its most important services through operational disruption. The canonical regulator's definition comes from the UK's Prudential Regulation Authority (PRA), in its Supervisory Statement SS1/21:

> "Operational resilience ... refers to the ability of firms, their groups, and the financial sector as a whole to **prevent, adapt to, respond to, recover from, and learn from operational disruptions**." (PRA SS1/21, March 2022, updating March 2021 — verified against the published SS1/21 document)

The FCA's formulation is the same family: the ability "to prevent, adapt and respond to, and recover and learn from operational disruption" (FCA, verified on the FCA operational-resilience page). The definition's load-bearing idea is the assumption behind it, which the PRA states explicitly: **disruptions will occur** — from time to time firms *will* be unable to operate as usual — so resilience is not about preventing every outage but about knowing which services matter most, deciding how long a disruption to each can be tolerated, and proving — through mapping, testing, and governance — that the firm can stay inside those tolerances. That is a fundamentally different posture from the traditional "we try not to fail" control mindset: it is a "**when we fail, we fail inside the tolerance**" engineering mindset.

### 1.2 Why operational resilience is the board-level risk topic

Operational resilience graduated from a BCM back-office discipline to a board-level topic for three converging reasons:

- **The regulator made it a board question.** The UK rules (effective 31 March 2022) require boards to select the firm's important business services and to sign off the impact tolerance for each — a judgement about how much disruption the *franchise* can bear, not a technical parameter (PRA PS6/21, §2). The PRA expects boards to be "clear" in their focus on operational resilience as the sector grows more dynamic, complex, and reliant on technology and third parties (SS1/21, ¶1.6).
- **The outage record made it a supervisory priority.** The 2023–24 bank-outage episode in Singapore (major retail-banking service disruptions) made technology risk a board-level issue for MAS-supervised institutions and fed MAS's operational-resilience focus area (see the MAS sibling guide §5.2 for the verified account). Globally, cloud outages, ransomware, and supplier failures have repeatedly taken down customer-facing services — each time, the question "which services, how long, and who decides?" lands on the board.
- **Interconnection made it systemic.** The Joint Forum's high-level principles for business continuity (August 2006) already framed continuity as a *financial-system* property: critical participants should target higher continuity standards because their failure propagates. The UK's 2018 Discussion Paper *Building the UK financial sector's operational resilience* and the 2021 final rules extended that to the sector level, and the EU's DORA (applied 17 January 2025) regulates digital operational resilience across 20 types of financial entities.

For the board, operational resilience is the risk topic that cannot be delegated downward: it sets the firm's answer to "which of our services must never fail for long, and how long is too long?" — a strategy-level judgement with capital, technology, and supplier consequences.

### 1.3 Operational resilience vs operational risk vs BCM vs cyber resilience

The four disciplines overlap and are frequently conflated; the distinctions matter because they have different owners, instruments, and metrics:

| Discipline | Core question | Primary instruments | Owner | Metric |
|---|---|---|---|---|
| **Operational risk** | "What can go wrong in our people, processes, systems, and external events — and how much capital does that require?" | Basel PSMOR; MAS Guidelines on Risk Management Practices – Operational Risk (2013); PRA op-risk framework | CRO / operational risk function | Losses, capital (SMA/AMA), KRIs |
| **Business continuity management (BCM)** | "How do we keep operating during and after a disruption?" | ISO 22301; MAS BCM Guidelines (2022); Joint Forum high-level principles (2006) | BCM owner | MTPD, MBCO, RTO/RPO, SRTO |
| **Cyber resilience** | "How do we withstand, respond to, and recover from cyber attacks specifically?" | MAS TRMG/FSM notices; NIST CSF; DORA ICT risk management | CISO / security function | Cyber KRIs, incident counts, recovery from cyber events |
| **Operational resilience** | "Which services are important, how long may each be disrupted, and can we prove we stay inside that?" | PRA/FCA impact-tolerance rules; DORA; MAS BCM+TRM+outsourcing stack; BCBS operational-resilience principles | Board / COO with the risk function | Impact tolerances per IBS, tolerance breaches |

The key relationship: **operational resilience is the outcome; BCM, DR, and cyber resilience are the capabilities that deliver it; operational risk is the risk family that frames and capitalises the underlying exposures.** The PRA made the boundary explicit: it deliberately refused to link a firm's ability to remain within impact tolerances to its operational-risk capital requirements (PS6/21, feedback chapter) — resilience is about *service delivery within tolerance*, while op-risk capital is about *expected and unexpected loss*. MAS's stack makes the same separation structurally: the BCM Guidelines own continuity, the TRM notices own technology reliability, the outsourcing notices own third parties, and the operational-resilience *expectation* is the supervisory theme that reads them together (MAS guide §5.2).

### 1.4 The lineage: how operational resilience became a discipline

Like ERM (see the Enterprise Risk Management Guide §1.4), operational resilience is a series of reactions to failure, each layer adding a requirement the previous one missed:

- **2006 — Joint Forum *High-level principles for business continuity*.** Seven principles (verified: published 29 August 2006 via BIS) that made business continuity a board responsibility (Principle 1), required planning for major disruptions (Principle 2), tied recovery objectives to the risk the participant poses to the financial system (Principle 3), stressed communication (Principles 4–5), periodic testing (Principle 6), and supervisory review (Principle 7).
- **2011–2021 — Basel operational risk and resilience principles.** The PSMOR (2003, revised 2011) embedded business-continuity expectations in the operational-risk canon; the final revisions of 31 March 2021 updated the principles for the Basel III op-risk framework; and on the same day the BCBS published its standalone *Principles for operational resilience* (both verified on bis.org).
- **2018–2025 — the UK builds the impact-tolerance machinery.** The BoE/PRA/FCA Discussion Paper (July 2018) → joint consultations (December 2019) → final rules (March 2021: PRA PS6/21, FCA PS21/3, BoE FMI policy) → in force 31 March 2022 → the "fully operational" testing milestone 31 March 2025.
- **2022–2025 — the EU makes digital resilience statutory.** DORA (Regulation (EU) 2022/2554) applied from 17 January 2025, harmonising ICT risk management, incident reporting, resilience testing, and ICT third-party oversight across the EU.
- **2022–2026 — MAS builds the service-centric stack.** The revised BCM Guidelines (6 June 2022) moved Singapore from process-centric to end-to-end service-centric continuity, layered under the TRM guidelines/notices, the 2023 outsourcing regime, and the 2025 incident-reporting circular (MAS guide §4.3, §5.2).

The lineage's lesson: each layer added *accountability* (board ownership), *measurement* (impact tolerances), or *proof* (testing, assurance) on top of the previous layer's *plans*.

### 1.5 The overview table

| Aspect | Description |
|---|---|
| **Definition** | PRA SS1/21: "the ability of firms, their groups, and the financial sector as a whole to prevent, adapt to, respond to, recover from, and learn from operational disruptions" |
| **The core assumption** | Disruptions will occur; resilience is the ability to keep important services inside defined tolerances, not the absence of outages |
| **The unit of analysis** | The **important business service (IBS)** and its **impact tolerance**, not the IT system |
| **The disciplines it consumes** | BCM (ISO 22301), DR (RTO/RPO), cyber resilience, third-party resilience — all subordinate to the IBS/tolerance view |
| **The regulators** | PRA/FCA (impact-tolerance rules), MAS (BCM/TRM/outsourcing stack), EU (DORA), BCBS (principles) — §2 |
| **The governance** | Board selects IBSs and sets tolerances; three lines of defence; resilience-by-design (§3) |
| **The evidence** | BIA (§4), IBS map (§5), scenarios (§6), BCP/DR designs (§7–§8), testing and assurance (§11) |

---

## 2. The Regulatory Landscape: MAS, PRA/FCA, DORA, BCBS

Four regulatory families shape a global bank's operational-resilience obligations. The MAS regime is covered in depth by the [MAS Regulations, Guidelines and Industry Expectations Guide](mas_regulations_guidelines_guide.md) (notices/guidelines naming, the instrument pyramid, the enforcement record) — this section cites the instruments relevant to resilience and verifies each against its primary source.

### 2.1 MAS (Singapore): the service-centric BCM stack

For a bank in Singapore, operational-resilience expectations sit in **four layered instruments** (verified against mas.gov.sg and the MAS sibling guide):

1. **The Guidelines on Business Continuity Management (BCM Guidelines), revised 6 June 2022** — verified directly on the MAS guidelines page (https://www.mas.gov.sg/regulation/guidelines/guidelines-on-business-continuity-management). They "set out the need for financial institutions (FIs) to take an **end-to-end service-centric view** in ensuring the continuous delivery of critical business services to their customers", and **supersede the June 2003 version and the January 2006 circular "Further Guidance on BCM"**. Implementation is disciplined: FIs "should meet the new Guidelines and establish a **BCM audit plan within 12 months** following its issuance; the **first BCM audit should be conducted within 24 months**". The guidelines apply across the full FI perimeter (banks, merchant banks, finance companies, insurers, CMS licensees, PSPs, trust companies). The revision process is documented on the page: first consultation March 2019, second consultation P015-2021 (Oct–Nov 2021, incorporating COVID-19 learnings), response 6 June 2022. The BCM Guidelines' headline expectations (per the published text as summarised in professional synopses **[verify]** for the full text): a **Service Recovery Time Objective (SRTO)** for each critical business service; **end-to-end dependency mapping** covering people, processes, technology, and third parties; **concentration-risk** awareness (people/technology/resources in the same zone, or several critical services outsourced to one provider); regular **testing**; **BCM audit**; incident and crisis management with an overall coordinator; and **board and senior-management responsibility** for business continuity.
2. **The technology-risk layer: TRM Guidelines (18 January 2021) + the FSM-series notices.** The *Guidelines on Risk Management Practices – Technology Risk* (published 18 January 2021, verified in the MAS sibling §4.2) carry the supervisory expectations on IT resilience and cyber resilience. The *binding* layer moved from sectoral notices — **MAS Notice 644, the banks' Technology Risk Management notice** (issued alongside sector siblings 830, 127, 506, 912, PSN05, CMG-N02, FAA-N18, TCA-N05, 1114, 644A), **all cancelled 10 May 2024** and replaced by the FSMA-era FSM-series (for a bank: **FSM-N05** for technology risk management, FSM-N22 for cyber hygiene) — verified in the MAS sibling §3.2/§4.2. Note the correction: **Notice 644 is a technology-risk-management notice, not a BCM notice**; the continuity expectations live in the BCM Guidelines. The TRM notice family's well-known availability benchmarks (critical systems: **no more than 4 hours of unscheduled downtime in any 12-month period**; **recovery time objective ≤ 4 hours**; **notification of relevant incidents to MAS within 1 hour** of discovery) are documented across secondary sources describing Notice 644 but were not re-extracted from the notice PDF in this pass **[verify]**.
3. **The outsourcing layer: Guidelines on Outsourcing (Banks), 11 December 2023, effective 11 December 2024, with Notices 658/1121** — the third-party/cloud resilience regime, treated in depth in [Vendor Management Guide](../management/vendor_management_guide.md) §8.8 and the MAS sibling §4.4.
4. **The incident-reporting layer: the Circular on Financial Institution Incident Reporting (16 December 2025)** — all reportable incidents submitted to MAS on the **MAS-Tx** platform with an updated template **from 1 February 2026** (verified in the MAS sibling §5.2).

MAS also names **Operational Resilience** as a supervisory focus area on its Regulation index (verified in the MAS sibling §5.2, which documents the post-2023 push following the bank-outage episode). **What this pass could not find: a standalone MAS "operational resilience" instrument** (notice or guidelines) — as of September 2026 the expectations are layered across the BCM, TRM, outsourcing, and incident-reporting instruments above **[verify] — monitor for a consolidated operational-resilience instrument**, especially given the direction of travel in the proposed third-party-risk management guidelines (consultation P004-2026, see §10).

### 2.2 The UK: PRA/FCA/BoE impact-tolerance rules

The UK regime is the intellectual heart of modern operational resilience. The verified timeline:

- **July 2018** — Discussion Paper *Building the UK financial sector's operational resilience* (BoE/PRA/FCA).
- **December 2019** — joint consultations: the FCA's **CP19/32** and the PRA's **CP29/19**, both titled *Operational resilience: Impact tolerances for important business services*. (Correction flagged: the consultation is not numbered "CP26/19" — that number could not be verified anywhere; the PRA's own PS6/21 identifies CP29/19 as its consultation, and the FCA identifies CP19/32.)
- **March 2021** — final policy published: **PRA PS6/21** *Operational resilience: Impact tolerances for important business services* (Policy Statement, March 2021, verified on bankofengland.co.uk), with **SS1/21** (Supervisory Statement) and a Statement of Policy; **FCA PS21/3** *Building operational resilience: Feedback to CP19/32 and final rules* (March 2021, verified on fca.org.uk); and the Bank of England's policy on operational resilience of financial market infrastructures. (Correction flagged: the FCA's operational-resilience statement is **PS21/3**, not PS21/15 — PS21/15 is the November 2021 *Regulation of funeral plans* statement, an unrelated instrument.)
- **31 March 2022** — the Operational Resilience Parts of the PRA Rulebook and SS1/21 became **effective** (verified in PS6/21: "The Operational Resilience Parts will be effective from Thursday 31 March 2022"); firms were required to have **mapped the people, processes, technology, and suppliers** supporting each important business service by this date; CRR consolidation entities implemented group provisions no later than 30 June 2022.
- **March 2022** — **PS2/22** *Operational Resilience and Operational Continuity in Resolution* extended the regime to financial and mixed-activity holding companies (verified in the SS1/21 annex).
- **31 March 2025** — the **testing milestone**: firms had until this date to complete the work needed to remain within their impact tolerances under severe-but-plausible scenarios; the FCA page states firms "had until 31 March 2025 to ensure they could operate their important business services within their impact tolerances" (verified on fca.org.uk/firms/operational-resilience).

The key obligations (verified from PS6/21 and SS1/21):

- **Identify important business services (IBSs)** — the short list of external-facing services whose disruption could pose a risk to the firm's safety and soundness or, for systemically relevant firms, UK financial stability; reviewed at least annually (SS1/21 ¶2.10).
- **Set an impact tolerance for each IBS** — the maximum tolerable level of disruption, typically time-based, set by boards and senior management.
- **Map resources** — people, processes, technology, and third parties supporting each IBS (by 31 March 2022).
- **Scenario test** — severe-but-plausible scenarios, testing the firm's ability to remain within impact tolerances (fully evidenced by 31 March 2025).
- **Govern, self-assess, and report** — board ownership, annual self-assessment against the rules, and lessons-learned documentation.

The PRA deliberately keeps operational resilience **separate from operational-risk capital**: it declined to link impact-tolerance performance to capital requirements (PS6/21 feedback).

### 2.3 The EU: DORA

**DORA — the Digital Operational Resilience Act, Regulation (EU) 2022/2554** — was adopted by the European Parliament and Council on **14 December 2022** and **entered into application on 17 January 2025** (verified on the EIOPA DORA page, https://www.eiopa.europa.eu/digital-operational-resilience-act-dora_en; the EUR-Lex full text could not be scraped in this pass, and the facts are corroborated by the repo's [Vendor Management Guide](../management/vendor_management_guide.md) §8.8). DORA harmonises, for **20 types of financial entities** plus ICT third-party service providers, five areas: (1) **ICT risk management** (a framework, not a silo); (2) **ICT-related incident reporting** (major incidents to the competent authority, with ESMA/EBA/EIOPA RTS setting classification and reporting); (3) **digital operational resilience testing**, including **threat-led penetration testing (TLPT)** for significant entities; (4) **ICT third-party risk**, including the **register of information** (a full inventory of ICT third-party arrangements) and contractual requirements; and (5) **information sharing** on cyber threats — plus an EU-level **oversight framework for critical ICT third-party providers (CTPPs)** to manage concentration risk. The repo's existing DORA treatment lives in [Vendor Management Guide](../management/vendor_management_guide.md) §8.8 (register of information, contractual clauses, oversight, and the non-EU contractual flow-down) — this guide does not re-derive it.

### 2.4 The BCBS: principles, not rules

The Basel Committee's contribution is principles-based and global:

- **Principles for the Sound Management of Operational Risk (PSMOR)** — first published 2003, revised June 2011, with the final **revisions published 31 March 2021** (verified on bis.org; the August 2020 consultative version is d508). PSMOR embeds business-continuity and resilience expectations in the operational-risk management canon: governance, risk management environment, and the role of disclosure.
- **Principles for operational resilience** — published **31 March 2021** (verified on bis.org; consultative version d509, August 2020). A standalone principles-based framework building on PSMOR and on earlier guidance on corporate governance, outsourcing, and business continuity, aimed at banks' ability to withstand events that could cause significant operational failures or wide-scale disruption (pandemics, cyber incidents, technology failures, natural disasters).
- **High-level principles for business continuity** — a **Joint Forum** paper (banking/securities/insurance supervisors) published **29 August 2006**, setting out **seven high-level principles** (verified on bis.org, document joint17). Correction flagged: this paper is **not** "BCBS 122" — BIS document BCBS 122 (February 2006) is *Enhancing corporate governance for banking organisations*, a different, superseded paper; the business-continuity principles are the Joint Forum publication.

### 2.5 The comparative table

| | **MAS (Singapore)** | **PRA/FCA/BoE (UK)** | **DORA (EU)** | **BCBS (global)** |
|---|---|---|---|---|
| **Instrument** | BCM Guidelines (2022); TRMG (2021) + FSM notices (2024); Outsourcing Guidelines + Notices 658/1121 (2023–24); Incident Reporting Circular (2025) | PRA PS6/21 + SS1/21 (March 2021); FCA PS21/3 (March 2021); BoE FMI policy; PS2/22 (2022) | Regulation (EU) 2022/2554 (14 Dec 2022), applied 17 Jan 2025; RTS/ITS/DR level-2 acts | PSMOR (2003/2011/2021); Principles for operational resilience (2021); Joint Forum BCP principles (2006) |
| **Scope** | All FIs in Singapore (banks, merchant banks, insurers, CMS licensees, PSPs, trust companies, ...) | UK banks, building societies, PRA-designated investment firms, insurers; FMIs; holding companies | 20 types of EU financial entities + ICT third-party providers | All internationally active banks |
| **Key obligation** | End-to-end service-centric BCM; SRTO per critical service; dependency mapping; BCM audit; tech-risk notices; incident notification | Identify IBSs; set impact tolerances; map resources; test severe-but-plausible scenarios; board ownership; self-assessment | ICT risk management framework; major-incident reporting; resilience testing incl. TLPT; register of information; ICT third-party oversight | Principles-based: governance, op-risk management, business continuity planning and testing |
| **Timeline** | BCM Guidelines effective from 6 June 2022 issuance; BCM audit within 24 months; FSM notices 10 May 2024; MAS-Tx incident reporting from 1 Feb 2026 | In force 31 Mar 2022; mapping by 31 Mar 2022; testing completed by 31 Mar 2025 | In force 16 Jan 2023; applied from 17 Jan 2025 | PSMOR revisions and resilience principles published 31 Mar 2021; implemented through national supervisors |

### 2.6 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| MAS BCM Guidelines | Revised 6 June 2022; supersede 2003 version + 2006 circular; audit plan ≤ 12 months, first audit ≤ 24 months | https://www.mas.gov.sg/regulation/guidelines/guidelines-on-business-continuity-management |
| MAS Notice 644 | Banks' Technology Risk Management notice — cancelled 10 May 2024, superseded by FSM-N05 | MAS guide (sibling) §3.2/§4.2 |
| PRA PS6/21 / SS1/21 | March 2021; effective 31 March 2022; mapping by 31 Mar 2022; testing by 31 Mar 2025 | bankofengland.co.uk PS6/21 PDF; SS1/21 PDF |
| FCA PS21/3 | *Building operational resilience* final rules, March 2021 (not PS21/15) | fca.org.uk PS21/3 |
| DORA | Regulation (EU) 2022/2554, 14 Dec 2022, applied 17 Jan 2025 | EIOPA DORA page; vendor guide §8.8 |
| BCBS PSMOR / resilience principles | Final revisions 31 March 2021 (both) | bis.org (d508/d509 consultations; 2021 final guidelines) |
| Joint Forum BCP principles | Seven principles, 29 Aug 2006 — not "BCBS 122" | https://www.bis.org/publ/joint17.htm |

---
## 3. The Governance and the Accountability: Board Ownership and the Three Lines

### 3.1 The board owns the tolerance, not just the policy

Every major instrument converges on the same governance answer: **operational resilience is owned at the top**. The Joint Forum's Principle 1 (2006) put ultimate responsibility for business continuity with boards and senior management; the PRA's rules make the board's judgement *the* mechanism — firms' boards select the important business services and set the impact tolerances, because "encouraging boards and senior management to make judgements in the selection of their important business services and the setting of impact tolerances will facilitate better decision-making" (PRA PS6/21); and MAS's BCM Guidelines state that the board and senior management are ultimately responsible for the FI's business continuity (MAS sibling §4.3). The accountability logic is simple: tolerance-setting is a **strategy decision** (how much disruption can the franchise, the customers, the regulators, and the system bear?), so it cannot be delegated to a BCM office that lacks the authority to trade off investment, risk, and service levels.

The board's operational-resilience duties in practice:

- **Approve the IBS list and each impact tolerance**, and re-approve at least annually or on significant change (PRA expectation, SS1/21 ¶2.10).
- **Approve the resilience strategy and the investment plan** — the BCP/DR designs, the testing calendar, the remediation programme for tolerance gaps.
- **Monitor tolerance breaches and near-breaches** through a standing board risk report (metrics, breach log, remediation status — see §11.5).
- **Own the "learn" loop** — post-incident and post-test lessons must reach the board, not stay in the operations silo.

### 3.2 The operational-resilience framework inside the risk framework

Operational resilience is not a parallel risk universe: it is a **capability requirement layered on the enterprise risk framework**. The [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) owns the machinery this guide plugs into — this section only draws the lines:

- **Three lines of defence (IIA model).** First line: the business and technology functions own and operate the services, the BCPs, and the DR designs. Second line: the risk function (CRO) and the resilience/BCM oversight function set the framework, challenge the BIA, review tolerance-setting, and monitor KRIs. Third line: internal audit provides independent assurance (including the MAS-required BCM audit — §11.4). The ERM guide §5 has the full treatment.
- **Risk appetite.** The enterprise risk appetite statement (ERM guide §6) should contain the operational-resilience lines: critical-service availability targets, maximum tolerable disruption per IBS, tolerance-breach escalation thresholds. The Cymbal Bank worked example (§12) shows how the RAS line "critical-system availability ≥ 99.9%" and the IBS tolerance table relate.
- **The risk register.** Resilience scenarios and tolerance gaps are risks like any other: they belong in the enterprise risk register with owners, dates, and escalation paths (§6.2).
- **ICAAP / capital.** Operational risk capital (SMA under Basel III, op-risk RWA) covers the *loss* dimension; operational resilience covers the *service-delivery* dimension. The PRA explicitly declined to link the two (PS6/21) — but the ICAAP stress-testing narrative should still discuss severe operational disruptions, because a tolerance breach can become a capital event (reputational, legal, and financial loss).

### 3.3 The role map

| Role | Operational-resilience accountability |
|---|---|
| **Board / board risk committee** | Approves IBS list, impact tolerances, resilience strategy, and the annual resilience report; monitors breaches |
| **CEO / group executive** | Owns the resilience outcome; chairs the crisis-management committee during major disruptions; signs the self-assessment |
| **CRO (second line)** | Sets the framework; challenges the BIA and tolerances; owns the tolerance-breach escalation and the KRI dashboard |
| **COO / head of operations** | Owns service delivery and the BCPs; first-line owner of business continuity for the operating businesses |
| **BCM owner (first/second line)** | Runs the BIA, the BCP lifecycle, the testing calendar, and the continuity culture |
| **Technology resilience owner (CTO/CIO side)** | Owns DR designs, RTO/RPO delivery, failover testing, and the technology-dependency maps |
| **Third-party risk owner** | Owns supplier mapping into IBS dependency maps, exit plans, concentration analysis (with the vendor-management function — §10) |
| **CISO** | Owns cyber-resilience response and recovery (with the Cybersecurity Guide) |
| **Internal audit (third line)** | Independent assurance: BCM audits, DR test observation, framework reviews |

### 3.4 The resilience-by-design principle

**Resilience-by-design** is the architectural discipline that mirrors security-by-design: resilience is not added after the fact (a DR plan bolted onto a finished system) but designed into the service from the start. For a solution architect this means:

- **Decomposition for tolerance.** Each IBS's chain of activities (per the PRA's framing: the chain from taking on an obligation to delivering the service) is decomposed so that each link can be assessed against the impact tolerance — you cannot design recovery for a service whose dependencies you cannot name (§4, §5).
- **No single points of failure in the critical path.** The dependency maps (people, process, technology, suppliers) are reviewed for single points of failure, and MAS's BCM Guidelines explicitly expect FIs to identify potential single points of failure early and eliminate them where possible.
- **Recovery objectives as first-class requirements.** RTO/RPO become non-functional requirements in the architecture and in procurement (every vendor contract carries the recovery and testing obligations — §10).
- **Testability.** A design that cannot be tested to the tolerance is not resilient; the design must include the test (failover, degraded mode, workaround) as a first-class artefact (§11).
- **Anti-fragile operations.** Runbooks, chaos-style exercises, and "game days" turn the design into muscle memory (cross-ref the OpenShift Workload Availability Guide §29 on DR drills and verification strategies).

### 3.5 The accountability map

The accountability map is the governance artefact that makes the role map auditable: a matrix that ties every IBS to its executive owner, its tolerance approver, its BCP/DR owners, and its assurance coverage. Regulators expect this to be *demonstrable* — MAS's BCM Guidelines expect BCM audit coverage of the framework (§11.4), and the PRA's self-assessment expectation (SS1/21 §8) effectively asks firms to document who decided what, with what evidence, and what lessons were learned. The Cymbal Bank worked example (§12.6) shows the three-lines governance in tabular form.

### 3.6 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| Board ownership | Boards select IBSs and set impact tolerances (PRA); board/senior management ultimately responsible for continuity (MAS BCM; Joint Forum P1) | PS6/21; MAS BCM Guidelines; Joint Forum 2006 |
| Three lines of defence | First line owns services/BCP; second line sets framework and challenges; third line assures | ERM guide §5 |
| Risk appetite linkage | Availability and tolerance lines in the RAS; tolerance breaches escalate as risk events | ERM guide §6 |
| Resilience-by-design | Decompose IBS chains; remove single points of failure; RTO/RPO as NFRs; testable designs | This guide §3.4; MAS BCM Guidelines |
| Role map | Board → CEO → CRO → COO → BCM owner → tech resilience owner → TPRM owner → CISO → audit | This guide §3.3 |
| Self-assessment | PRA expects documented self-assessment with lessons learned (SS1/21 §8) | SS1/21 |

---

## 4. The Business Impact Analysis: From Activities to Impact Tolerances

### 4.1 The purpose of the BIA

The **business impact analysis (BIA)** is the analytical engine of the whole discipline: it identifies the organisation's activities, quantifies the impact of their disruption **over time**, and produces the recovery priorities and objectives that every downstream artefact — the IBS list, the impact tolerances, the BCP, the DR design, the testing calendar — is built from. The international reference is **ISO/TS 22317:2021**, *Security and resilience — Business continuity management systems — Guidelines for business impact analysis* (verified on iso.org, standard 79000), which gives guidelines for implementing and maintaining a formal, documented BIA process without prescribing a uniform method.

The BIA answers three questions that nothing else in the framework answers:

1. **Which activities, if disrupted, would hurt us most — and how fast does the hurt grow?**
2. **What do those activities depend on** (people, processes, technology, data, third parties, facilities)?
3. **How long can each activity be down before the impact becomes unacceptable** — i.e., what is its maximum tolerable period of disruption?

Without a BIA, impact tolerances are guesses, BCPs are shelf documents, and the DR design is a technology wish-list.

### 4.2 The BIA steps

The ISO/TS 22317-shaped BIA process, in five steps:

1. **Scope and inventory the activities.** Enumerate the business activities (the unit of analysis is the *activity*, not the department): payments processing, trade settlement, client onboarding, loan disbursement, market-data distribution, regulatory reporting, and so on. The inventory should be service-oriented — MAS's BCM Guidelines push FIs to think in terms of **critical business services** delivered to customers, supported by **business functions** (a function may underpin many services, so its disruption amplifies across them).
2. **Identify the dependencies.** For each activity, map the end-to-end dependencies: people (roles, skills, single points of human failure), processes (workflows, handoffs, manual fallbacks), technology (applications, infrastructure, data, networks), and third parties (suppliers, market infrastructures, cloud providers). MAS's BCM Guidelines make end-to-end dependency mapping an explicit expectation, including dependencies that run through third parties.
3. **Quantify the impact over time.** For each activity, estimate the impact of disruption at successive time points (e.g., 1 hour, 4 hours, 1 day, 3 days, 1 week, 1 month) across the impact categories (§4.3). The *time profile* is the key output: most impacts are not linear — a payments service may absorb a 2-hour outage and suffer severely at 24 hours; a settlement service may be tolerable for a day and catastrophic at T+2.
4. **Derive the recovery objectives.** From the time profiles, derive for each activity: the **MTPD** (maximum tolerable period of disruption — how long the activity may be down at all), the **MBCO** (minimum business continuity objective — the minimum level of service that must be restored, and by when), and the supporting technology objectives **RTO/RPO** (§5.4). MAS's BCM Guidelines express the same idea as an **SRTO** — a Service Recovery Time Objective per critical business service.
5. **Validate and maintain.** BIA outputs are validated with the business owners (the tolerance is a *business* judgement, not an IT estimate), challenged by the second line, and reviewed at least annually or on significant change — the same cadence the PRA expects for the IBS list.

### 4.3 The impact categories

The BIA quantifies impact across four families — the same families the UK rules' "safety and soundness" test and MAS's "reputation, financial safety and soundness, and the proper functioning of the financial ecosystem" test decompose into:

| Impact category | What it captures | Typical metrics |
|---|---|---|
| **Financial** | Lost revenue, compensating payments, penalties, interest, liquidity strain | SGD/day of disruption, revenue at risk, compensation exposure |
| **Customer** | Inconvenience, harm, attrition, complaints, redress | Affected customers, churn, complaints, redress costs |
| **Regulatory** | Breach of obligations (settlement, reporting, notification), censure, licence conditions | Reportable incidents, notification deadlines missed, regulatory actions |
| **Reputational** | Franchise damage, media, counterparty confidence, funding access | Front-page risk, social-media sentiment, depositor/counterparty behaviour |

The four categories are weighted differently per service: for a retail payments service, customer and reputational impact dominate early; for a settlement service, regulatory and financial impact dominate at T+2; for a trading service, financial impact is immediate.

### 4.4 How the BIA feeds the IBS identification

The BIA is the feedstock for the important-business-services decision (§5): the activities that score highest on impact-over-time, once dependency-mapped, become candidates for IBS status; the MTPDs derived in the BIA become the raw material for the impact tolerances the board signs off. The PRA's framing makes the linkage explicit — important business services are the external-facing services whose disruption could pose a risk to safety and soundness or financial stability, and the BIA is the evidence base for that judgement. In the Cymbal Bank worked example (§12), the BIA output table (activities × impact over time × MTPD) is the direct input to the IBS map.

### 4.5 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| BIA standard | ISO/TS 22317:2021, *Guidelines for business impact analysis* | https://www.iso.org/standard/79000.html |
| BIA steps | Scope/inventory → dependencies → impact over time → recovery objectives → validate/maintain | This guide §4.2; ISO/TS 22317 |
| Impact categories | Financial, customer, regulatory, reputational | This guide §4.3 |
| Key outputs | MTPD, MBCO, RTO/RPO per activity; SRTO per critical service (MAS) | This guide §4.2, §5.4 |
| MAS linkage | End-to-end dependency mapping; service-centric view; SRTOs | MAS BCM Guidelines (2022) |
| IBS linkage | BIA outputs → IBS candidates → board-approved tolerances | This guide §4.4, §5 |

---
## 5. The Important Business Services and the Impact Tolerances

### 5.1 What an important business service is

The **important business service (IBS)** is the unit of analysis of the modern operational-resilience regime — the PRA/FCA framing defines it as the services a firm provides which, **if disrupted, could pose a risk to the firm's safety and soundness** or, for systemically relevant firms, **the financial stability of the UK** (and, for insurers, policyholder protection) (verified in PRA SS1/21 §2.2–2.3). Three properties of the definition do most of the work:

- **External-facing.** An IBS delivers a specific outcome to an *identifiable user external to the firm* — retail customers, business customers, other legal entities, market participants, supervisors. Internal services (HR, payroll) are not IBSs in their own right; they enter the picture only as dependencies inside an IBS's chain of activities (SS1/21 ¶2.7–2.8).
- **A short list.** The PRA expects a "relatively short list" of services for which the firm chooses to build high levels of resilience in anticipation of disruption (SS1/21 ¶2.3) — the list must be proportionate to the business and small enough that boards can make prioritisation and investment decisions (¶2.9).
- **A judgement, not a calculation.** Identification is a board-level judgement informed by analysis (the BIA, §4), not a scoring formula.

MAS's sibling concept is the **critical business service** in the BCM Guidelines — the services whose continuous delivery to customers the end-to-end, service-centric view is built around; the 2022 revision explicitly moved MAS's regime from process-centric to service-centric thinking.

### 5.2 How to identify and prioritise IBSs

The identification workflow, in the PRA's framing as refined by market practice:

1. **Start from the BIA.** Take the activities with the steepest impact-over-time profiles and the broadest dependency fan-out (§4.4).
2. **Apply the safety-and-soundness / financial-stability / policyholder-protection test.** For each candidate, ask: would disruption threaten the firm's viability, its P&L and reputation, legal or regulatory censure — or, at the system level, the functioning of the wider economy and knock-on effects on counterparties and market infrastructures (SS1/21 ¶2.5)?
3. **Check the users are identifiable.** An impact tolerance can only be applied to a service whose users — and therefore whose disruption impacts — are identifiable (SS1/21 ¶2.6).
4. **Keep the chain in view.** Identify the critical parts of the chain of activities that make up the service (e.g., trade execution includes the risk-management clearance step; a mortgage IBS includes onboarding through to servicing) — those critical parts are what must be operationally resilient (¶2.8).
5. **Prioritise.** Rank the IBSs by severity of impact and by how quickly the impact escalates; the ranking drives investment, testing depth, and the board's attention.
6. **Review annually, or on significant change** (¶2.10) — new products, acquisitions, outsourcing moves, and technology changes can all change the list.

### 5.3 The impact tolerance

An **impact tolerance** is the maximum level of disruption to an IBS that the firm is willing to accept — the level of disruption beyond which the firm's safety and soundness (or the wider system's stability) would be threatened. Under the UK rules, firms must specify for each IBS the **length of time (or point in time)**, in addition to any other relevant metrics, for which a disruption can be tolerated (verified in PS6/21 — the rules were amended to require the time dimension explicitly). Impact tolerances are:

- **Set by the board and senior management** — the judgement-based mechanism at the heart of the regime;
- **Typically time-based** — e.g., "customer payments may be disrupted for no more than X hours"; other metrics (e.g., number of affected customers) can be used in conjunction;
- **Set per IBS, with cross-IBS awareness** — firms should consider the impact of the failure of related IBSs when setting a single IBS's tolerance (PS6/21);
- **Testable** — the whole regime is built on the firm proving, under severe-but-plausible scenarios, that it can remain within each tolerance.

MAS's BCM Guidelines express the same concept as the **SRTO — Service Recovery Time Objective** for each critical business service: a time-based metric that provides clarity on expected recovery timelines, guides resource prioritisation, and supports decision-making and monitoring during a disruption (per the published guidelines as summarised in professional synopses **[verify]** for the full text).

### 5.4 The metric family: MTPD, MBCO, RTO, RPO

The recovery-metrics vocabulary comes from the ISO 22301 family (verified: ISO 22301:2019 is the current BCMS requirements standard — see §7.3):

| Metric | Definition | Where it comes from | Used for |
|---|---|---|---|
| **MTPD** — maximum tolerable period of disruption | The longest period an activity/service can be disrupted before the impact becomes unacceptable | ISO 22301 vocabulary | Setting the business-level tolerance; the ceiling the recovery design must respect |
| **MBCO** — minimum business continuity objective | The minimum level of service that must be achieved (and by when) during a disruption, to meet the MTPD | ISO 22301 / BS 25999 lineage | Defining the degraded-but-acceptable operating state (which services at what volume) |
| **RTO** — recovery time objective | The target time within which a system/process must be restored after a disruption | DR practice (ISO 22301-aligned) | Technology and supplier recovery targets; DR tier design |
| **RPO** — recovery point objective | The maximum acceptable data loss (the point in time to which data must be recovered) | DR practice (ISO 22301-aligned) | Backup frequency, replication design, data-loss exposure |
| **SRTO** — service recovery time objective (MAS) | The expected recovery timeline for a critical business service | MAS BCM Guidelines (2022) | MAS-aligned service recovery planning and monitoring |

The hierarchy that holds them together: **impact tolerance (board) ≥ MTPD (business) ≥ RTO (technology/supplier)**. The board sets the impact tolerance for the IBS; the BIA derives the MTPD per activity; the architecture translates MTPD into RTO/RPO for systems and contracts. If RTO > MTPD for any critical dependency, the design is *wrong* — the firm would breach its own tolerance by construction. (The exact clause references for MTPD/MBCO inside ISO 22301:2019 were not re-extracted in this pass **[verify]** — the definitions given match the standard's canonical vocabulary as used throughout BCM practice and the ISO 22301/ISO/TS 22317 documentation.)

### 5.5 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| IBS definition | External-facing services whose disruption could threaten safety and soundness / financial stability / policyholder protection; a short, board-approved list | PRA SS1/21 §2 |
| Impact tolerance | Maximum tolerable disruption per IBS, time-based (plus other metrics), board-set, testable | PRA PS6/21; SS1/21 §3 |
| Annual review | IBS list reviewed at least annually or on significant change | SS1/21 ¶2.10 |
| MTPD | Maximum tolerable period of disruption — the business-level ceiling | ISO 22301 vocabulary |
| MBCO | Minimum business continuity objective — the minimum acceptable service level during disruption | ISO 22301 vocabulary |
| RTO / RPO | Recovery time / recovery point objectives — the technology and supplier targets | DR practice, ISO 22301-aligned |
| SRTO | MAS's service recovery time objective per critical business service | MAS BCM Guidelines (2022) |
| BIA standard | ISO/TS 22317:2021 (BIA guidelines) | https://www.iso.org/standard/79000.html |

---

## 6. The Risk Assessment: Severe-but-Plausible Scenarios

### 6.1 Scenario-based assessment for operational resilience

The operational-resilience risk assessment is **scenario-based by regulatory design**: the PRA requires firms to test their ability to remain within impact tolerances under **severe but plausible scenarios** (PS6/21; SS1/21 §6), and MAS's BCM Guidelines expect BCPs to address "a range of severe and plausible disruption scenarios" (per the published text). The scenario method matters because the traditional risk-assessment question — "how likely is this, and how big is the expected loss?" — is the wrong instrument for resilience. Resilience is about *what would happen to our important services if a severe event occurred*, regardless of its probability, and about whether the recovery design closes the gap. Scenarios therefore replace likelihood with plausibility and replace loss with **tolerance breach**.

### 6.2 The scenario taxonomy and the link to the risk register

A resilience scenario library typically spans five families:

1. **Technology failure** — data-centre or cloud-region loss, core-system outage, network failure, database corruption, cascading failure across interdependent systems.
2. **Cyber events** — ransomware encrypting critical systems, destructive malware, denial-of-service against customer channels, supply-chain attack through a third party (cross-ref the Cybersecurity Guide for the threat-side treatment).
3. **People and facilities** — pandemic-scale absenteeism, loss of a key site (the COVID-19 episode that shaped MAS's 2022 BCM revision), loss of a critical team, industrial action.
4. **Third-party and market-infrastructure failure** — a critical supplier's outage (cloud provider, payment scheme, settlement infrastructure, market-data vendor), a supplier's insolvency, a market-wide disruption of a shared utility.
5. **Compound events** — the realistic ones: a cyber attack that disables both primary and backup sites (the classic "both copies deleted" ransomware pattern), a regional outage hitting people, power, and connectivity at once.

Each scenario is run against the **IBS map**: for each IBS in the scenario, does the firm remain inside its impact tolerance? The outputs — the tolerance gaps — are **risks like any other**: they go into the enterprise risk register with owners and remediation dates, assessed through the ERM process (cross-ref the Enterprise Risk Management Guide §7 for the register mechanics). The MAS sibling's "operational resilience" focus area and the 2023–24 outage episode show the supervisory expectation: scenario gaps must be visible in the risk framework, not parked in a BCM folder.

### 6.3 How resilience scenarios differ from standard operational-risk scenarios

| Dimension | Standard op-risk scenario (loss-focused) | Operational-resilience scenario (tolerance-focused) |
|---|---|---|
| **Unit of analysis** | The loss event (frequency × severity) | The important business service and its impact tolerance |
| **Question** | "How much would this cost?" | "Would we stay inside our tolerance — and if not, how far over, and why?" |
| **Probability** | Estimated (frequency) | Plausibility only ("severe but plausible") |
| **Output** | Loss distribution, capital, provisioning | Tolerance-breach map, remediation programme, investment cases |
| **Owner** | Operational-risk function / CRO | Board + COO + risk function jointly |
| **Evidence** | Loss data, RCSA | Mapping, testing results, dependency analysis |

The two views coexist: the same scenario (a cloud-region outage) produces a loss estimate for the ICAAP/op-risk capital process and a tolerance-breach analysis for the resilience programme. The PRA's separation of resilience from op-risk capital (PS6/21) is precisely the statement that these are two different outputs from the same scenario work.

### 6.4 The scenario-assessment workflow

1. **Select scenarios** from the library (severe but plausible, relevant to the firm's IBSs and threat profile) — typically 3–6 exercised in depth per year (§11.3).
2. **Define the scenario baseline** — the IBSs in scope, the assumed failure (duration, extent, which dependencies), the degraded operating conditions.
3. **Run the service-impact analysis** — for each IBS: can we deliver within tolerance? Via which recovery path (failover, fallback, workaround, manual process)? With what data loss (RPO) and at what recovery time (RTO)?
4. **Document the gaps** — every tolerance breach or near-breach becomes a registered risk with an owner and a remediation date.
5. **Learn and adapt** — lessons feed the BCP, the DR design, the supplier contracts, and the next BIA (the "learn" clause in the definition).

### 6.5 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| Scenario standard | Severe-but-plausible scenarios; test ability to remain within impact tolerances | PRA PS6/21; SS1/21 §6 |
| MAS expectation | BCPs address a range of severe and plausible disruption scenarios | MAS BCM Guidelines (2022) |
| Scenario families | Technology, cyber, people/facilities, third-party/FMI, compound | This guide §6.2 |
| Register linkage | Tolerance gaps → enterprise risk register → ERM process | ERM guide §7 |
| Vs op-risk scenarios | Tolerance-focused, not loss-focused; plausibility, not frequency | This guide §6.3 |

---
## 7. The Business Continuity Planning: The BCP Lifecycle and ISO 22301

### 7.1 The BCP lifecycle: plan, do, check, act

Business continuity management is a management system, and the management system runs on the **plan–do–check–act (PDCA)** cycle that ISO 22301 codifies. The four phases map onto the operational-resilience programme:

- **Plan** — establish the BCMS: policy, scope, governance (§3), the BIA (§4), the risk assessment (§6), the strategy (§7.4), and the documented plans. This is where the IBS/tolerance view is translated into continuity requirements.
- **Do** — implement: write the BCPs, crisis-management plans, and IT disaster-recovery plans; build the recovery capabilities (sites, people, suppliers, runbooks); train the responders; embed continuity in change management and procurement.
- **Check** — verify: testing and exercising (§11), BCM audits (the MAS-required audit — §11.4), KRI monitoring, and management review of the programme's effectiveness.
- **Act** — improve: lessons from incidents, tests, and audits feed back into the BIA, the plans, and the strategy — the "learn and adapt" clause of the resilience definition.

### 7.2 The BCP document set

The plan set has three layers, each with a different audience and trigger:

| Plan | Purpose | Trigger | Owner |
|---|---|---|---|
| **Business continuity plan (BCP)** | How each critical business service/function is recovered and maintained at the MBCO: teams, tasks, manual fallbacks, workarounds, supplier invocation, RTO/RPO per activity | Service disruption | Business + BCM owner |
| **Crisis-management plan (CMP)** | How the firm manages the *event*: incident command, decision rights, communications, regulatory notifications, stakeholder management | Major/crisis-level event | CEO office / crisis-management team (§9) |
| **IT disaster-recovery (DR) plan** | How the technology estate supporting the services is recovered: failover procedures, site activation, data restoration, RTO/RPO per system, test evidence | Technology disaster | Technology resilience owner (§8) |

The three must be **mutually consistent**: the BCP's MBCO figures must match the DR plan's RTO/RPO, and both must reference the same IBS/tolerance table — a classic failure mode is a BCP that assumes a 4-hour system recovery while the DR plan only promises 24 hours.

### 7.3 ISO 22301: the BCMS standard

**ISO 22301:2019**, *Security and resilience — Business continuity management systems — Requirements*, is the international standard for a BCMS (verified on iso.org, standard 75106): it provides "a framework for organizations to plan, establish, implement, operate, monitor, review, maintain, and continually improve a documented management system to protect against, reduce the likelihood of, and ensure recovery from disruptive incidents." Verified edition facts: **Edition 2, published October 2019** (the 2012 first edition is withdrawn), currently at the "International Standard to be revised" stage with **Amendment 1:2024** (climate action changes) published; the companion guidance standard is **ISO 22313:2020** (*Guidance on the use of ISO 22301*). The BIA guidelines live in **ISO/TS 22317:2021** (§4). For a bank, ISO 22301 certification of the BCMS is a common supervisory and client expectation — and it provides the auditable backbone the MAS BCM Guidelines' audit expectation and the PRA's self-assessment expectation can hook onto.

### 7.4 The business-continuity strategy options

The BIA produces the requirements; the strategy chooses *how* continuity is achieved. The strategy menu (from ISO 22301 practice and the repo's technology guides):

- **Redundancy** — duplicate the capability so a second instance carries the load: active-active sites, multi-AZ/multi-region deployment (cross-ref the IBM Cloud Guide §3.1 on regions and multizone regions; the OpenShift Workload Availability Guide §3 on HA/DR/BC concepts), resilient network paths, cross-trained staff.
- **Fallback** — a standby capability that takes over: warm standby environments, cold sites, alternate suppliers, a manual process that replaces a failed automated one.
- **Workarounds** — degraded-mode operation at the MBCO: service a queue manually, route customers to another channel, prioritise high-value transactions while volumes are reduced.
- **Acceptance** — for non-critical activities, accept the outage within a defined period (the MTPD is the acceptance boundary).

The strategy is chosen per activity *and per recovery time band*: the fastest-recovering services (minutes-to-hours) need redundancy; the next band (hours-to-days) can use fallback; the slowest band (days) can use workarounds and acceptance. The cost curve is steep — redundancy is expensive, so the tolerance table is what justifies the spend: only services with tight tolerances get active-active designs (§12.4 shows the Cymbal Bank strategy table).

### 7.5 The MBCO and the recovery priorities

The **MBCO** (minimum business continuity objective, §5.4) defines the degraded-but-acceptable operating state: which services continue, at what volume, by when. Recovery priorities sequence the restoration: highest priority goes to the services whose tolerance is tightest *and* whose failure cascades (a settlement service whose outage blocks other services ranks above a standalone reporting service, even with similar tolerances). MAS's BCM Guidelines make the prioritisation explicit through the SRTO per critical service, guiding "the prioritisation of resources during the planning and facilitating decision-making and monitoring of the recovery progress in a disruption". The recovery-priority table is a board-level artefact — it is the operational expression of the tolerance decisions.

### 7.6 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| BCP lifecycle | Plan–do–check–act (PDCA) | ISO 22301; this guide §7.1 |
| Plan set | BCP + crisis-management plan + IT DR plan, mutually consistent | This guide §7.2 |
| ISO 22301 | Current edition 2019 (Edition 2, 2019-10); Amd 1:2024; ISO 22313:2020 guidance | https://www.iso.org/standard/75106.html |
| Strategy options | Redundancy, fallback, workarounds, acceptance — per recovery-time band | This guide §7.4 |
| MBCO / priorities | Minimum service level during disruption; recovery-priority table | This guide §7.5 |
| MAS angle | SRTO per critical service; prioritisation during recovery | MAS BCM Guidelines (2022) |

---

## 8. The Disaster Recovery: Tiers, RTO/RPO, and the Technology Estate

### 8.1 The DR tiers

Disaster recovery is the technology half of continuity: restoring the systems and data that deliver the services. The classic DR tiering describes how ready the standby is:

| Tier | Description | Typical RTO | Typical RPO | Cost |
|---|---|---|---|---|
| **Cold site** | Bare facility (power, cooling, space); nothing pre-staged; you bring the hardware and restore from backups | Days | Up to the last backup (hours–days) | Low |
| **Warm site** | Facility with pre-staged infrastructure (servers, storage, network); data restored from backups or asynchronous replication | Hours–1 day | Minutes–hours (asynchronous replication) | Medium |
| **Hot site** | Fully equipped and configured standby, synchronised data, exercised regularly | Minutes–hours | Minutes (synchronous/async replication) | High |
| **Active-active** | Multiple live sites sharing production traffic; failure just re-routes load | Minutes (near-zero) | Near-zero (synchronous replication) | Highest |

The tier is chosen per service from the tolerance table: an IBS with a 2-hour impact tolerance cannot be served by a cold site; an active-active design is justified only where the tolerance is tight enough to demand it (§7.4). The repo's technology guides carry the mechanics: the OpenShift Workload Availability Guide §3 (HA, DR, BC, and resilience concepts), §24–§26 (DR maturity model, Red Hat Advanced Cluster Management for automated DR, DR for stateful workloads), and §29 (verification strategies and DR drills); the IBM Cloud Guide §3.1 (regions, multizone regions, and data centres — the building blocks of site strategy); and the Securing Red Hat OpenShift Guide §3 (etcd encryption, key management, and backup keys — the data-protection layer that makes restores possible at all).

### 8.2 RTO/RPO mapping to impact tolerances

The DR design is the tolerance table translated into technology:

- **RTO ≤ MTPD, with margin.** The system recovery time must be comfortably inside the business's maximum tolerable period of disruption — "comfortably" because the RTO is a target and the MTPD is a ceiling: a service with an MTPD of 24 hours and an RTO of 23 hours has a design, not a margin.
- **RPO from the tolerance.** The acceptable data loss derives from the impact analysis: a payments service may tolerate minutes of loss; a market-data archive may tolerate hours; a trade-capture system's RPO may be effectively zero (synchronous replication), because re-creating lost trades is worse than the cost of replication.
- **Per-service, not per-system.** RTO/RPO are set per IBS or per critical activity, then decomposed into per-system targets — the decomposition is where architects earn their keep, because shared infrastructure means one system's RTO drives several services' recovery paths.
- **Supplier RTO/RPO are contractual.** Every critical supplier's recovery commitments become contract terms with test evidence (§10).

### 8.3 The dependency on the technology estate

DR is only as good as the estate it runs on. Three architectural realities dominate:

1. **The single-point-of-failure hunt.** The dependency maps (§4.2) are scanned for single points of failure — a shared network segment, a single storage array, one cloud region, one supplier — and MAS's BCM Guidelines expect FIs to identify and eliminate them where possible. The availability themes of the repo's technology guides (availability zones, failover, backup — OpenShift Workload Availability Guide; IBM Cloud Guide) are the raw material for that hunt.
2. **Data is the hard part.** Applications fail over; data must be *consistent* when it arrives. RPO design, replication topology (synchronous vs asynchronous), and the "both copies deleted" ransomware scenario (§6.2) determine whether the DR site is actually usable — the reason DR drills (§11.3) test data restoration, not just site activation.
3. **Change management.** The DR design decays with every change: a new dependency, a new supplier, a decommissioned system. Recovery documentation must be versioned with the estate, and the testing calendar must re-validate the design on change — the "plan–do–check–act" loop applied to DR (§7.1).

### 8.4 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| DR tiers | Cold / warm / hot / active-active, with RTO/RPO bands | This guide §8.1 |
| RTO/RPO mapping | RTO ≤ MTPD with margin; RPO from the impact analysis; per-service decomposition | This guide §8.2 |
| Availability mechanics | Zones, failover, backup, DR drills | OpenShift Workload Availability Guide §3, §24–§26, §29; IBM Cloud Guide §3.1 |
| Data protection | etcd encryption, backup keys, restore capability | Securing Red Hat OpenShift Guide §3 |
| SPOF elimination | MAS expectation to identify and eliminate single points of failure | MAS BCM Guidelines (2022) |
| Supplier recovery | Contractual RTO/RPO with test evidence | This guide §8.2, §10 |

---
## 9. The Crisis Management and the Communication

### 9.1 The crisis-management plan

When a disruption crosses from "incident" to "crisis" — the tolerance is at risk, customers are affected, regulators may need to know — the BCP is not enough: the firm needs **crisis management**. The crisis-management plan (CMP, §7.2) governs the *event* rather than the service: it defines what triggers crisis mode, who is in the crisis-management team (CMT), where they sit (a physical or virtual crisis room), how decisions get made under time pressure, and how the firm communicates internally, externally, and with regulators. MAS's BCM Guidelines expect FIs to have robust incident and crisis-management processes to resume critical business services within the stipulated SRTOs/RTOs, including an overall coordinator where a service depends on multiple functions.

### 9.2 The incident command structure

The incident command model — adapted from emergency-management practice — gives the response a spine:

- **Incident commander** — one accountable owner of the response, with authority over resources and priorities; typically the COO or the head of the affected business for service incidents, escalating to the CEO for firm-level crises.
- **Functional cells** — service recovery (the BCP/DR teams), technology (the platform teams), communications (internal/external), customer care (front office and contact centre), regulatory affairs (notification drafting and submission), legal, HR (for people-related events), and finance (loss capture and compensation).
- **Liaison** — a single point of contact to the CMT, to the board, and to suppliers.
- **Decision rights** — pre-agreed: who may declare crisis mode, who may invoke the DR site, who may commit compensation, who signs off the regulatory notification, who speaks to the press. The PRA expects firms to have developed communication strategies in advance to minimise the impact of disruptions (PS6/21 feedback — the SS expectations for communication plans were published as proposed).

The structure must be **trained, not invented**: the first time the command structure is exercised should be a tabletop (§11.3), not a live outage.

### 9.3 The internal and external communication

Communication is where good recovery plans and good crisis management diverge most in practice. The Joint Forum's principles made communication a first-class continuity requirement (Principle 4: clear, regular communication is necessary to manage a crisis and maintain public confidence; Principle 5: cross-border communication protocols). The PRA's expectations (PS6/21, communication chapter) cover preparing communication strategies in advance, including with supervisors, third-party providers, and industry peers. The discipline:

- **Internal**: staff know what to tell customers and what not to promise; the frontline has a scripted, authorised status message; the CMT status updates are time-boxed and factual.
- **Customer**: service-status transparency (outage pages, in-app banners, call-tree messaging) reduces complaint volume and reputational damage; MAS's scam-aware consumer-protection agenda makes customer communication during disruptions a conduct issue as well as a service issue.
- **Regulatory**: the notification duties below are treated as a *submission discipline* with pre-drafted templates and named owners, not improvised on the day.
- **Market/counterparties**: for wholesale services, counterparties and market infrastructures need status and fallback instructions (the cross-border angle of Principle 5).

### 9.4 The regulatory notification duties

The notification map (as of 2026, verified through the MAS sibling and the repo's verified records):

- **MAS**: the *Instructions on Incident Notification and Reporting to MAS* (linked from the BCM Guidelines page) govern how incidents are reported under the various acts, notices, circulars, and guidelines; the **Circular on Financial Institution Incident Reporting (16 December 2025)** moved all reportable-incident submissions onto the **MAS-Tx platform with an updated template from 1 February 2026** (verified in the MAS sibling §5.2); the TRM notice family's 1-hour notification expectation for relevant incidents is widely documented **[verify]** (§2.1).
- **PRA/FCA**: firms notify their supervisors of operational incidents affecting important business services; the March 2025 milestone was accompanied by supervisory attention to incident reporting (including reporting of incidents meeting certain thresholds even before an impact tolerance is breached — flagged in law-firm commentary on the 2025 expectations **[verify]**).
- **DORA**: **major ICT-related incidents** must be reported to the competent authority (initial, intermediate, and final reports), with classification and reporting standards set in the level-2 RTS (verified via the EIOPA DORA page listing the RTS on ICT incidents classification and reporting).

The guiding principle across regimes: **when in doubt, notify** — a late or absent notification is almost always worse than the incident itself.

### 9.5 The post-incident review: learn and adapt

"Learn" is the last verb in the resilience definition, and the post-incident review (PIR) is its instrument. The PRA expects firms to identify lessons learned from scenario testing and document them in their self-assessments (PS6/21; SS1/21 §8.3) — the same discipline applies to live incidents. The PIR discipline:

1. **Timeline reconstruction** — what happened, when, with what decision points.
2. **Tolerance-breach analysis** — were we inside tolerance? If not, how far over, and at which link of the chain?
3. **Root-cause and contributing-factor analysis** — technology, process, people, supplier, or scenario-design causes (the ERM register linkage, §6.2).
4. **Action plan with owners and dates** — each lesson becomes a tracked remediation, reviewed at the next board risk report.
5. **Culture** — the review must be blameless in fact-finding and accountable in action-taking; a PIR that produces no owners is a ceremony, not a control.

### 9.6 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| Crisis plan | CMP governs the event: triggers, CMT, crisis room, decision rights | This guide §9.1; MAS BCM Guidelines |
| Command structure | Incident commander + functional cells + liaison + pre-agreed decision rights | This guide §9.2 |
| Communication | Internal, customer, regulatory, market; prepared in advance | Joint Forum P4–P5; PRA PS6/21 |
| MAS notification | Incident Reporting Circular (16 Dec 2025); MAS-Tx from 1 Feb 2026 | MAS sibling §5.2 |
| DORA notification | Major ICT incidents: initial/intermediate/final reports | EIOPA DORA page; RTS 2024/1772, 2025/301 |
| PIR | Lessons learned documented in self-assessment | SS1/21 §8.3; this guide §9.5 |

---

## 10. The Third-Party Resilience and the Supply Chain

*Condensed by design: the full third-party-risk machinery (lifecycle, contracts, monitoring, exit, DORA's ICT third-party regime) lives in the [Vendor Management Guide](../management/vendor_management_guide.md) §8 (and the MAS outsourcing framework in the [MAS Regulations, Guidelines and Industry Expectations Guide](mas_regulations_guidelines_guide.md) §4.4). This section covers only the operational-resilience angle.*

### 10.1 The resilience angle on third parties

From the operational-resilience perspective, a third party is not a procurement category — it is a **dependency inside an IBS's chain of activities**. The consequences:

- **Suppliers appear in the IBS maps.** Every critical supplier (cloud provider, payment scheme, settlement infrastructure, market-data vendor, outsourcer, software vendor) is mapped to the IBSs it supports, with its RTO/RPO commitments recorded against the service's tolerance (SS1/21 expects firms to map third parties supporting IBSs and to do assurance work on them — the PRA's expectations on third-party assurance were clarified in PS6/21).
- **End-to-end testing runs through the supplier chain.** The firm's testing calendar must include joint testing with critical suppliers — the vendor's BCP is only relevant to the extent it has been *tested with yours* (Vendor Management Guide §8.8: BCP aligned with the vendor's own, tested jointly, RTOs defined).
- **Exit plans are resilience plans.** A supplier's failure or insolvency is a disruption scenario; the exit plan (data return, transition to a successor, run-off) is the recovery design for that scenario, and DORA's exit-strategy requirements (data portability for critical ICT services) and MAS's outsourcing expectations (orderly exit) make it a regulatory requirement (Vendor Management Guide §9.2–9.3).
- **Concentration is a resilience risk, not just a commercial one.** MAS's BCM Guidelines explicitly warn that concentration arises when several critical services are outsourced to a single provider, or when people/technology/resources are concentrated in the same zone; DORA's oversight framework for critical ICT third-party providers exists precisely to manage systemic concentration (EIOPA DORA page); the Vendor Management Guide §2 covers the portfolio-level concentration metrics.

### 10.2 The MAS outsourcing framework in one paragraph

For Cymbal Bank's Singapore hub, the binding framework is the **Guidelines on Outsourcing (Banks) (11 December 2023, effective 11 December 2024)** read with **Notices 658 (banks) and 1121 (merchant banks)**: materiality assessment of outsourced relevant services, lifecycle risk management (due diligence, contracts, monitoring, exit), notification of material outsourcing, and the expectation that cloud and critical service providers are treated as outsourced relevant services regardless of label (verified in the MAS sibling §4.4). The direction of travel: the **proposed Guidelines on Third-Party Risk Management (consultation P004-2026, March 2026)** would broaden the regime from "outsourcing" to the full third-party lifecycle with board accountability and concentration risk front and centre (verified in the Vendor Management Guide §8.8) **[verify]** for finalisation.

### 10.3 The supplier-resilience checklist

- Critical suppliers identified and mapped to IBSs (with single points of failure flagged).
- Contractual RTO/RPO per critical service, consistent with the tolerance table (§8.2).
- Joint BCP alignment and joint testing on the annual calendar.
- Audit and assurance rights (SOC 2, ISO 27001, on-site audits) — and the regulator's right to inspect flowing down to the contract (MAS/DORA both require this; Vendor Management Guide §8.8).
- Concentration analysis: critical services per supplier, per zone, per region; Herfindahl-style metrics for the critical portfolio.
- Exit plans current for every critical supplier, with data-return mechanisms tested.
- Subcontractor visibility through the chain (DORA's subcontracting RTS; MAS expectations on sub-outsourcing).

### 10.4 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| Suppliers in IBS maps | Third parties mapped as dependencies; assurance work on them | SS1/21; PS6/21 |
| MAS outsourcing | Guidelines (11 Dec 2023, eff. 11 Dec 2024) + Notices 658/1121 | MAS sibling §4.4 |
| TPRMG direction | Consultation P004-2026 (March 2026) — lifecycle-wide TPRM **[verify]** | Vendor Management Guide §8.8 |
| DORA third-party | Register of information, contractual clauses, CTPP oversight | EIOPA DORA page; Vendor Management Guide §8.8 |
| Concentration | MAS BCM concentration warning; DORA CTPP oversight | MAS BCM Guidelines; EIOPA |
| Exit plans | Data portability, orderly exit, run-off | Vendor Management Guide §9 |

---

## 11. The Testing and the Assurance

### 11.1 The testing hierarchy

Testing is the "check" phase of the BCP lifecycle and the evidence base of the entire regime — the PRA's regime culminates in the *demonstration* that firms can remain within impact tolerances, and MAS's BCM Guidelines make regular, comprehensive testing an explicit expectation. The testing hierarchy runs from cheapest to most realistic:

| Level | What happens | What it proves | Frequency guide |
|---|---|---|---|
| **Desk check** | Plans reviewed for currency (contacts, versions, dependencies) | The plan is not stale | Quarterly–continuous |
| **Walkthrough** | Plan owners talk through the steps against the plan | The plan is coherent and complete | 6–12 monthly |
| **Tabletop** | Key players simulate a scenario in a room (or virtually), decisions and gaps surface | The *organisation* can respond — roles, decisions, handoffs | 6–12 monthly per IBS |
| **Simulation** | A realistic rehearsal with live data feeds and partial execution | Recovery procedures work end-to-end in degraded mode | Annually |
| **Live failover test** | Actual failover to the DR environment (or a parallel test environment), real data, real traffic | The DR design actually delivers its RTO/RPO | Annually (or per regulatory expectation) |

The hierarchy's logic: cheap tests run often and catch the cheap failures (stale plans, wrong contacts, missing dependencies); expensive tests run less often and catch the expensive failures (designs that don't deliver). A firm that only does live failover tests will discover plan staleness at the worst moment; a firm that only does desk checks will discover design failure at the worst moment.

### 11.2 The scenario-based testing

The PRA's model (SS1/21 §6) ties testing to the tolerance machinery: firms select severe-but-plausible scenarios (§6) and test, for each affected IBS, whether they remain within impact tolerance. The test output is not "pass/fail of the system" but **"inside/outside the tolerance, and what would it take to close the gap"**. Testing therefore produces investment cases, not just test reports — a firm that discovers a 30-hour recovery against a 24-hour tolerance has found a funded remediation programme, not a failed drill. The same scenario library (§6.2) drives both risk assessment and testing, so the assessment and the evidence stay aligned.

### 11.3 The tabletop exercises and the live failover tests

- **Tabletops** are the highest-value-per-dollar test: they exercise the *organisation* (decision rights, escalation, communication, the incident command structure of §9.2) rather than the technology. The facilitation craft — neutrality, managing dominant voices, drawing out the second-order questions — is covered in the [Workshop Methodology Guide](../management/workshop_methodology_guide.md) §7 (the facilitation craft) and §11 (the Cymbal Bank workshop series); a resilience tabletop is a workshop with a crisis scenario instead of a design agenda. Tabletops should be run per IBS or per scenario family, with observers and a structured debrief that feeds the lessons log (§9.5).
- **Live failover tests** prove the DR design: actual failover to the standby environment, actual data restoration against the RPO, actual service resumption against the RTO. The OpenShift Workload Availability Guide §29 (verification strategies and DR drills) carries the platform-level drill types and runbook templates; the firm-level discipline is to run them on a calendar that covers every critical system at least annually, with supplier joint tests (§10.3) on the same calendar. DORA's testing regime goes further for significant entities: **threat-led penetration testing (TLPT)** every three years (verified via the EIOPA DORA page and the RTS on TLPT, EU 2025/1190) — a red-team exercise of the resilience and security posture together.

### 11.4 The independent assurance

- **Internal audit** provides the third-line view: the MAS BCM Guidelines require FIs to establish a **BCM audit plan within 12 months** of the 2022 issuance and conduct the **first BCM audit within 24 months** (verified on the MAS page) — with the audit programme covering BCM preparedness based on the FI's operational-risk exposure. Internal audit also observes DR tests, reviews the tolerance-setting governance, and audits the IBS selection evidence.
- **Self-assessment** is the PRA's complementary instrument: firms document their compliance with the rules, the results of scenario testing, and lessons learned (SS1/21 §8) — a self-assessment is only as good as the evidence trail behind it.
- **External assurance**: ISO 22301 certification of the BCMS (§7.3), supplier SOC/ISO reports (§10.3), and — under DORA — the supervisory authorities' own review and the TLPT regime.

### 11.5 The metrics and the board reporting

The board reporting pack (see §12.5 for the Cymbal Bank version) turns the programme into numbers the board can govern:

- **Resilience metrics** — per IBS: recovery time achieved vs tolerance (test and live), RTO/RPO achievement rates, test completion vs calendar, supplier test completion.
- **Tolerance breach reporting** — every breach or near-breach of an impact tolerance, live or in testing, with root cause and remediation; this is the headline number, because the regime is defined by tolerances.
- **KRIs** — leading indicators: availability of critical systems, incident counts by severity, open remediation items over age, single-point-of-failure count, supplier concentration metrics, BCP documentation currency.
- **The board dashboard** — a one-page view: the IBS/tolerance table with RAG status per IBS, the breach log, the testing calendar status, the open-gap register with owners and dates, and the investment asks.

The reporting cadence is monthly (management) and quarterly (board risk committee), with the annual self-assessment and BCM audit report as the year-end artefacts.

### 11.6 The section reference table

| Term / fact | Value | Source / pointer |
|---|---|---|
| Testing hierarchy | Desk check → walkthrough → tabletop → simulation → live failover | This guide §11.1 |
| Scenario testing | Severe-but-plausible; output is inside/outside tolerance | SS1/21 §6; this guide §11.2 |
| Tabletop craft | Facilitation, neutrality, debrief | Workshop Methodology Guide §7, §11 |
| DR drills | Drill types, runbook templates | OpenShift Workload Availability Guide §29 |
| TLPT | Threat-led penetration testing every 3 years (significant entities) | EIOPA DORA page; RTS EU 2025/1190 |
| BCM audit | Audit plan ≤ 12 months, first audit ≤ 24 months (MAS) | MAS BCM Guidelines page |
| Board reporting | Resilience metrics, breach log, KRIs, dashboard | This guide §11.5, §12.5 |

---
## 12. The Worked Example: An Operational-Resilience Programme for a Cymbal Bank

*All numbers in this section are pedagogical constructions for teaching the framework — they are not figures from any bank's disclosures.*

### 12.1 The scenario

**Cymbal Bank** is a global corporate and investment bank — group headquarters in Paris, a major **APAC hub in Singapore**, a balance sheet of roughly €400bn, and four businesses: global markets (rates, FX, credit, commodities), structured finance, trade finance, and corporate banking (the familiar repo context from the Enterprise Risk Management Guide §10.1 and the MAS guide §7). The Singapore hub is MAS-supervised; the group parent sits in the EU (so DORA-aligned group standards apply at the parent level and flow down), and the group has a UK branch (PRA/FCA impact-tolerance rules apply to the group's UK entities). The board has mandated a rebuild of the operational-resilience programme to the 2026 standard: **IBS-led, tolerance-driven, tested, and board-reported** — replacing a 2010s-era stack of BCP binders.

The design below follows the sections of this guide in order: IBS map (§12.2) from the BIA outputs (§12.3), BCP/DR design (§12.4), testing calendar (§12.5), board reporting pack (§12.6), and three-lines governance (§12.7).

### 12.2 The IBS map

The board-approved list: **five important business services** for the Singapore hub, each with an impact tolerance (the level of disruption the board will accept), an MTPD derived from the BIA, and the technology/supplier objectives decomposed from the tolerance. The tolerance is set at group standard and localised to the MAS-supervised entity.

| # | Important business service | Users | Impact tolerance (board) | MTPD (BIA) | RTO (tech) | RPO (tech) |
|---|---|---|---|---|---|---|
| IBS-1 | **Customer payments & cash management** (retail and corporate payments, FAST/MEPS+ and cross-border) | Retail and corporate customers | No more than **4 hours** of disruption to customer payments | 4 hours | ≤ 2 hours | ≤ 5 minutes |
| IBS-2 | **Markets execution & trade capture** (rates, FX, credit, commodities) | Institutional clients, internal desks | No more than **2 hours** of disruption to execution and capture | 2 hours | ≤ 1 hour | ≤ 1 minute |
| IBS-3 | **Clearing & settlement** (payments settlement, securities settlement, nostro reconciliation) | Counterparties, market infrastructures (MEPS+, CSDs, correspondent banks) | No more than **1 business day** — but settlement obligations at T+2 must be met | 1 day (T+2 hard stop) | ≤ 6 hours | ≤ 15 minutes |
| IBS-4 | **Client onboarding & KYC/AML** (account opening, screening, transaction monitoring) | New and existing clients; regulators | No more than **1 business day** of full outage; screening backlog cleared within 2 days | 1 day | ≤ 8 hours | ≤ 1 hour |
| IBS-5 | **Regulatory reporting & MAS returns** (MAS 610-series returns, incident notifications, tax reporting) | MAS, group regulators | No **missed regulatory deadline**; reporting capability restored within **6 hours** | 6 hours | ≤ 4 hours | ≤ 30 minutes |

Notes on the map: IBS-1 and IBS-2 carry the tightest tolerances (customer-facing, revenue-critical, reputationally explosive — the categories of §4.3); IBS-3's tolerance is *event-driven* (the T+2 settlement cycle defines the real deadline); IBS-5's tolerance is *deadline-driven* (a report that can be filed late is a breach). The board re-approves the table annually (SS1/21 ¶2.10).

### 12.3 The BIA outputs

The BIA (ISO/TS 22317:2021 method, §4) produced the following evidence for the IBS decision — an abridged extract:

| Activity (within IBS) | Key dependencies | Impact at 1h / 4h / 1d / 3d | MTPD | Feeds |
|---|---|---|---|---|
| Retail payment instruction processing (IBS-1) | Payments hub, FAST/MEPS+ connectivity, fraud screening, contact centre | Low / High / Severe / Severe | 4 h | IBS-1 tolerance |
| Trade capture & booking (IBS-2) | Trading platform, market data, order management, risk clearance | High / Severe / Severe / Severe | 2 h | IBS-2 tolerance |
| Securities settlement instruction (IBS-3) | Settlement engine, CSD connectivity, nostro reconciliation, counterparty ops | Low / Medium / High / Severe (T+2 hard stop) | 1 d | IBS-3 tolerance |
| Sanctions screening (IBS-4) | Screening platform, watchlist data, compliance ops | Low / Medium / High / High | 1 d | IBS-4 tolerance |
| MAS return production (IBS-5) | Reporting data warehouse, regulatory reporting engine, data governance | Low / Medium / High / High (deadline breach) | 6 h | IBS-5 tolerance |

The BIA also surfaced three cross-cutting findings: (1) IBS-1 and IBS-3 share the same payments hub — a single point of failure that made both tolerances fragile; (2) the sanctions-screening platform is a single supplier with a 24-hour RTO in its contract — above IBS-4's MTPD; (3) market-data dependencies run through one vendor across IBS-2 and IBS-3. Each finding became a registered risk with an owner and a remediation date (§6.2).

### 12.4 The BCP/DR design

The strategy table (per §7.4 and §8) translates the tolerances into designs:

| IBS | Strategy | Site design | RTO / RPO | Supplier design |
|---|---|---|---|---|
| IBS-1 | **Redundancy** (active-active) | Payments hub active-active across two availability zones in the Singapore multizone region; warm standby in a second APAC region | 2 h / 5 min | Payments-scheme and cloud contracts carry 2-hour RTO, joint test annually |
| IBS-2 | **Redundancy** (active-active) + workarounds | Trading platform active-active in-region; market-data fallback via secondary vendor feed | 1 h / 1 min | Secondary market-data vendor contracted; failover exercised in simulation |
| IBS-3 | **Redundancy + fallback** | Settlement engine warm standby in second APAC region; manual fallback procedures for T-day processing | 6 h / 15 min | CSD and correspondent RTOs aligned to the T+2 hard stop |
| IBS-4 | **Fallback + workaround** | Screening platform warm standby; manual screening queue with priority triage as workaround | 8 h / 1 h | Supplier contract renegotiated to 8-hour RTO with penalty |
| IBS-5 | **Fallback** | Reporting engine warm standby; data warehouse replicated with 30-minute RPO | 4 h / 30 min | In-house; no critical supplier |

The DR tiering (§8.1) follows the tolerance bands: IBS-1 and IBS-2 are active-active (hot); IBS-3 and IBS-5 are warm; IBS-4 is warm-with-workaround. The single-point-of-failure finding on the payments hub drove a funded remediation: the shared hub is being split into independently recoverable segments, with the interim design documented as a known gap in the risk register.

### 12.5 The testing calendar

A 12-month calendar, mapped to the regulatory expectations:

| Month | Test | Scope | Regulatory mapping |
|---|---|---|---|
| Jan | Desk-check + walkthrough cycle | All five IBSs' BCP/DR plans | MAS BCM Guidelines (regular testing) |
| Feb | Tabletop: ransomware on the payments hub (IBS-1) | Crisis team, incident command, comms, supplier | PRA-style severe-but-plausible scenario |
| Mar | Tabletop: cloud-region loss (IBS-2, IBS-3) | Technology resilience team, business owners | PRA-style; group DORA-aligned |
| Apr | Simulation: settlement T+2 stress (IBS-3) | Settlement ops, CSD connectivity, manual fallback | MAS BCM; PRA scenario testing |
| May | Live failover test: payments hub to standby region (IBS-1) | Full failover, data restoration, RPO check | MAS BCM; DR drill (§11.3) |
| Jun | **BCM audit (first of the year)** | Framework, BIA, plans, test evidence | MAS BCM Guidelines (audit within 24 months) |
| Jul | Joint supplier test: cloud provider + scheme | Supplier BCP alignment, joint failover | MAS outsourcing; vendor guide §8.8 |
| Aug | Tabletop: market-data vendor failure (IBS-2) | Fallback feed, desk workarounds | PRA-style scenario |
| Sep | Live failover test: reporting engine (IBS-5) | Warm standby activation, return production | MAS BCM; regulatory reporting continuity |
| Oct | Group TLPT (DORA-aligned, EU parent) | Red-team exercise of security + resilience | DORA TLPT (RTS EU 2025/1190) |
| Nov | Full simulation: compound scenario — regional outage + cyber | Multiple IBSs, crisis management, MAS notification drill | MAS BCM; incident-reporting circular drill |
| Dec | Annual self-assessment + board resilience report | Tolerance evidence, lessons log, remediation status | PRA self-assessment (SS1/21 §8); MAS BCM |

Every test produces: an inside/outside-tolerance verdict per IBS, a lessons log entry (§9.5), and — where a gap is found — a registered risk with an owner and date (§6.2).

### 12.6 The board reporting pack

The quarterly board risk-committee pack, one page:

| IBS | Tolerance | Last test result | Live-incident record | Status |
|---|---|---|---|---|
| IBS-1 Payments | 4 h | 2 h 10 min (May failover) | 1 near-breach (37 min over at 4h37m, root cause: DNS) — remediated | 🟢 |
| IBS-2 Markets | 2 h | 55 min (simulation) | None | 🟢 |
| IBS-3 Settlement | 1 d | 5 h 40 min (Apr simulation) | None | 🟢 |
| IBS-4 Onboarding/KYC | 1 d | 9 h (walkthrough; live test due) | 1 breach (screening backlog, 26 h) — remediation on track | 🟡 |
| IBS-5 Reporting | 6 h | 3 h 50 min (Sep failover) | None | 🟢 |

**Breach log (quarter):** one live tolerance breach (IBS-4 screening backlog after a platform upgrade, 26 hours to clear, root cause: batch-job regression; remediation: job monitoring KRI added, supplier penalty clause invoked), one near-breach (IBS-1 DNS event), three test gaps closed within the quarter. **KRIs on the dashboard:** critical-system availability (99.96% YTD vs 99.9% appetite), open remediation items over 90 days (3), single points of failure (2, down from 5), supplier concentration index (within appetite), BCP documentation currency (100%). **Investment asks:** the payments-hub segmentation programme and the screening-platform supplier exit options — both traced to tolerance evidence.

### 12.7 The three-lines governance

| Line | Who | Operational-resilience accountabilities |
|---|---|---|
| **Governing body** | Board / board risk committee (SG hub: MAS-supervised entity board + group board) | Approves the IBS map and tolerances (§12.2), the testing calendar, the annual resilience report |
| **First line** | Business heads, COO, technology (CTO/CIO), vendor owners | Own and run the services, the BCPs, the DR designs, the joint supplier tests; execute the calendar |
| **Second line** | CRO office (operational risk + resilience oversight), BCM owner, TPRM | Set the framework, challenge the BIA/tolerances, run the KRI dashboard, escalate breaches, own the MAS notification process |
| **Third line** | Internal audit | BCM audits (MAS-timed), DR test observation, framework and governance reviews, self-assessment challenge |

The accountability map (§3.5) names the executive owner per IBS — for example, the COO owns IBS-1 and IBS-3, the head of global markets owns IBS-2, the head of client life-cycle management owns IBS-4, and the CFO owns IBS-5 — each with the tolerance approver (board) and the assurance coverage (audit) documented.

### 12.8 The lessons

1. **The tolerance table is the keystone.** Every other artefact — BIA, BCP, DR design, testing calendar, board pack — hangs off the board-approved IBS/tolerance table; an unowned table produces unowned resilience.
2. **The BIA finds the real problems.** The shared payments hub and the supplier RTO mismatches emerged from dependency mapping, not from technology reviews.
3. **Testing is an investment case, not a compliance exercise.** Every test that produced a gap also produced a funded remediation with an owner and a date.
4. **The board pack is the programme.** If the board cannot see tolerance status, breach log, and open gaps on one page, the programme is not yet governed.
5. **Regulatory alignment is layered, not duplicated.** MAS BCM testing, PRA-style scenarios, and the group's DORA-aligned TLPT share one calendar and one evidence base.

### 12.9 The section reference table

| Artefact | Where it lives | Key content |
|---|---|---|
| IBS map | §12.2 | Five IBSs with board-set impact tolerances, MTPDs, RTO/RPO (pedagogical numbers) |
| BIA outputs | §12.3 | Activities × dependencies × impact-over-time × MTPD; cross-cutting findings |
| BCP/DR design | §12.4 | Strategy per IBS (redundancy/fallback/workarounds); site design; supplier design |
| Testing calendar | §12.5 | 12-month calendar mapped to MAS BCM, PRA-style scenarios, DORA-aligned TLPT |
| Board reporting pack | §12.6 | Tolerance-status table, breach log, KRIs, investment asks |
| Three-lines governance | §12.7 | Governing body / first / second / third line with named owners |

---

## 13. The Summary: One Page — "The Resilient Bank"

Operational resilience is the discipline that assumes disruption will happen and proves the bank can keep its most important services inside defined tolerances anyway. The one-page version:

- **The definition** (PRA SS1/21): the ability of firms, their groups, and the financial sector to **prevent, adapt to, respond to, recover from, and learn from** operational disruptions. The core assumption: disruptions will occur; resilience is staying inside the tolerance, not avoiding the outage.
- **The unit of analysis**: the **important business service** — external-facing, board-selected, few in number — and its **impact tolerance**: the maximum disruption the firm will accept, set by the board, time-based, and testable.
- **The regulators**: the UK built the machinery (PS6/21/PS21/3, March 2021; in force 31 March 2022; testing by 31 March 2025); MAS layered the service-centric stack (BCM Guidelines 2022, TRM notices, outsourcing 2023–24, incident reporting 2025–26); the EU made digital resilience statutory (DORA, applied 17 January 2025); the BCBS set the global principles (PSMOR and operational-resilience principles, March 2021; Joint Forum BCP principles, 2006).
- **The machinery**: the BIA (ISO/TS 22317) derives MTPD, MBCO, RTO/RPO, and SRTO; the IBS/tolerance table is the keystone; severe-but-plausible scenarios test inside/outside tolerance; the BCP lifecycle (ISO 22301, PDCA) and the DR tiers (cold to active-active) deliver recovery; crisis management and communication govern the event; suppliers are dependencies inside the IBS maps, tested jointly; testing and assurance (desk check to live failover, BCM audits, self-assessment) provide the evidence; the board pack reports tolerance status, breach log, and KRIs.
- **The final word — the resilient bank**: resilience is not a plan on a shelf or a DR site that has never failed over — it is a set of board-approved judgements (which services, how long, who decides), translated into designs and contracts, proven by testing, and reported as numbers the board can govern. Every element of the framework exists to answer one question with evidence: *when the disruption comes — and it will — do we stay inside the tolerance?* The bank that can answer yes, with a board that owns the answer, is the resilient bank.

---
## 14. Verification Notes and Sources

**Method.** Facts in this guide were verified against primary sources using targeted web research conducted in **September 2026** — the MAS regulations and guidance pages (mas.gov.sg), the Bank of England/PRA publications (bankofengland.co.uk, including the PS6/21 and SS1/21 PDFs), the FCA's operational-resilience page and PS21/3, the EIOPA DORA page (europa.eu), the BIS/BCBS publication pages (bis.org), and the ISO catalogue pages (iso.org). Where a fact came from the repo's sibling guides (which carried their own verification ledgers), the sibling is named as the source. The repo's house honesty convention applies: anything that could not be confirmed is marked ⚠ or ❌ below and in the body text rather than asserted.

### 14.1 The claims audit

| # | Claim | Status | Source |
|---|---|---|---|
| 1 | PRA SS1/21 definition: "prevent, adapt to, respond to, recover from, and learn from operational disruptions" | ✅ verified | PRA SS1/21, March 2022 (updating March 2021), bankofengland.co.uk supervisory-statement PDF |
| 2 | PRA PS6/21 final policy published **March 2021**; Operational Resilience Parts effective **31 March 2022**; mapping of resources required by 31 March 2022 | ✅ verified | PS6/21 PDF (bankofengland.co.uk/-/media/boe/files/prudential-regulation/policy-statement/2021/march/ps621.pdf) |
| 3 | FCA final rules = **PS21/3** *Building operational resilience*, March 2021; consultation was **CP19/32** (Dec 2019) | ✅ verified | fca.org.uk PS21/3 page/PDF |
| 4 | Firms had until **31 March 2025** to operate important business services within impact tolerances (three-year transition) | ✅ verified | fca.org.uk/firms/operational-resilience |
| 5 | The joint consultation was numbered "**CP26/19**" | ❌ could not verify — the consultations were FCA **CP19/32** and PRA **CP29/19** (both December 2019); no CP26/19 exists for this policy | PRA PS6/21 (identifies CP29/19); FCA PS21/3 (identifies CP19/32) |
| 6 | The FCA operational-resilience statement was "**PS21/15**" | ❌ could not verify — PS21/15 is the November 2021 *Regulation of funeral plans* statement; operational resilience is PS21/3 | fca.org.uk publications index |
| 7 | MAS **BCM Guidelines revised 6 June 2022**; supersede the June 2003 version and the January 2006 BCM circular; BCM audit plan within 12 months, first BCM audit within 24 months; end-to-end service-centric view | ✅ verified | MAS page: mas.gov.sg/regulation/guidelines/guidelines-on-business-continuity-management |
| 8 | "MAS **Notice 644 on Business Continuity Management**" | ❌ could not verify — MAS Notice 644 is the banks' **Technology Risk Management** notice; the sectoral TRM notices (incl. 644) were cancelled 10 May 2024 and replaced by the FSM-series (banks: FSM-N05); the continuity expectations live in the BCM Guidelines | MAS sibling guide §3.2/§4.2 (verified from MAS catalogue) |
| 9 | MAS *Guidelines on Risk Management Practices – Operational Risk* (**1 March 2013**) exist as the operational-risk framework guidelines | ✅ verified | MAS sibling guide §4.8 (BCM page related list) |
| 10 | TRM notice family benchmarks: critical systems ≤ 4 hours unscheduled downtime per 12 months; RTO ≤ 4 hours; incident notification within 1 hour | ⚠ partially verified — consistent across secondary sources describing Notice 644, but the notice PDF was not re-extracted in this pass | Secondary sources (e.g., TRM summaries); MAS sibling guide §3.2 |
| 11 | **DORA** = Regulation (EU) 2022/2554, adopted 14 December 2022, **applied 17 January 2025**; five pillars incl. TLPT and CTPP oversight; 20 types of financial entities | ✅ verified | EIOPA DORA page (europa.eu); corroborated by Vendor Management Guide §8.8 (EUR-Lex direct scraping was blocked in this pass) |
| 12 | BCBS **PSMOR final revisions published 31 March 2021** (2003 original; 2011 revision superseded) | ✅ verified | bis.org (202103-guidelines-revisions-principles-sound-management-operational-risk; consultation d508) |
| 13 | BCBS **Principles for operational resilience** published **31 March 2021** | ✅ verified | bis.org (202103-guidelines-principles-operational-resilience; consultation d509) |
| 14 | "*High-level principles for business continuity*" = "**BCBS 122**" | ❌ could not verify — BIS document BCBS 122 (Feb 2006) is *Enhancing corporate governance for banking organisations*; the business-continuity principles are the **Joint Forum** paper (joint17), published **29 August 2006**, seven principles | bis.org (bcbs122.htm vs joint17.htm) |
| 15 | **ISO 22301:2019** is the current BCMS standard (Edition 2, published October 2019; Amd 1:2024 climate-action changes; ISO 22313:2020 guidance companion) | ✅ verified | iso.org/standard/75106.html |
| 16 | **ISO/TS 22317:2021** is the BIA guidelines standard | ✅ verified | iso.org/standard/79000.html |
| 17 | "Joint PRA/FCA/BoE policy published **March 2022**" | ⚠ partially verified — the final policy was published **March 2021** and became effective 31 March 2022; SS1/21 was updated in March 2022 | PS6/21; SS1/21 (March 2022 edition) |
| 18 | A standalone MAS "operational resilience" notice or guidelines instrument exists | ❌ could not verify — none identified as of September 2026; expectations are layered across BCM, TRM, outsourcing, and incident-reporting instruments; monitor for consolidation | MAS site; MAS sibling guide §5.2 |
| 19 | MTPD / MBCO definitions trace to ISO 22301 vocabulary | ⚠ partially verified — the definitions given match the standard's canonical vocabulary as used across ISO 22301 / ISO/TS 22317 documentation, but exact clause references were not re-extracted | ISO 22301:2019 documentation; BCM practice |
| 20 | MAS BCM Guidelines require an **SRTO** per critical business service and end-to-end dependency mapping | ⚠ partially verified — consistent with the published guidelines as summarised in professional synopses; the MAS page extraction did not reproduce the full guideline text | MAS BCM Guidelines page; professional synopses (GIEOM, PwC) |
| 21 | PRA expects assurance work on third parties supporting IBSs | ✅ verified | PS6/21 feedback chapter; SS1/21 |
| 22 | **PS2/22** *Operational Resilience and Operational Continuity in Resolution* (March 2022) extended the regime to financial/mixed-activity holding companies | ✅ verified | SS1/21 annex (March 2022 update note) |
| 23 | MAS **Circular on Financial Institution Incident Reporting (16 December 2025)**; MAS-Tx platform from 1 February 2026 | ✅ verified | MAS sibling guide §5.2 |
| 24 | Post-2025 FCA/PRA expectation to report incidents meeting thresholds even before an impact tolerance is breached | ⚠ partially verified — reported in law-firm commentary on the 2025 milestone; the primary supervisory text was not re-extracted in this pass | Sidley client alert (Jan 2025); FCA page |
| 25 | DORA TLPT cycle: significant entities test every three years | ✅ verified | EIOPA DORA page; RTS on TLPT (EU 2025/1190) |

### 14.2 What Could Not Be Verified

The following items could **not** be verified in this pass, and are flagged accordingly in the body text (⚠/❌) rather than asserted:

- **"CP26/19"** as the joint operational-resilience consultation — no such number could be found; the verified consultations are FCA CP19/32 and PRA CP29/19 (December 2019).
- **"FCA PS21/15"** as the operational-resilience final rules — PS21/15 is the funeral-plans statement; the verified instrument is FCA PS21/3 (March 2021).
- **"MAS Notice 644 on BCM"** — Notice 644 is the banks' Technology Risk Management notice (cancelled 10 May 2024, superseded by FSM-N05); there is no MAS BCM notice, the continuity regime is the BCM Guidelines.
- **"BCBS 122"** as the business-continuity principles — BCBS 122 is a corporate-governance paper; the seven high-level principles for business continuity are the Joint Forum paper (joint17, 29 August 2006).
- **A standalone MAS operational-resilience instrument** — none was found as of September 2026; the MAS expectations remain layered across the BCM, TRM, outsourcing, and incident-reporting instruments (with the proposed TPRM guidelines P004-2026 pending).
- **The exact clause references for MTPD/MBCO inside ISO 22301:2019** — the vocabulary is canonical and used as defined here, but the clause numbers were not re-extracted.
- **The full text of the MAS BCM Guidelines (June 2022)** — the MAS page (title, dates, audit expectations, applies-to) was verified directly; the SRTO/dependency-mapping/concentration expectations are quoted via professional synopses of the published text.
- **The TRM notice availability benchmarks (4 hours/12 months, RTO ≤ 4 hours, 1-hour notification)** — verified only through secondary sources describing Notice 644; the notice PDF itself was not re-extracted.
- **The EUR-Lex full text of DORA** — scraping was blocked; DORA facts were verified via the EIOPA page and the repo's Vendor Management Guide §8.8.

---

### 14.3 Glossary

| Term | Definition |
|---|---|
| **Operational resilience** | The ability of a firm, its group, and the financial sector to prevent, adapt to, respond to, recover from, and learn from operational disruptions (PRA SS1/21) |
| **Important business service (IBS)** | An external-facing service whose disruption could pose a risk to the firm's safety and soundness or, for systemically relevant firms, financial stability (or policyholder protection); board-selected, short-list |
| **Impact tolerance** | The maximum level of disruption to an IBS that the firm is willing to accept — time-based (plus other metrics), board-set, testable |
| **BIA** | Business impact analysis — the process (ISO/TS 22317:2021) that identifies activities, maps dependencies, and quantifies the impact of disruption over time |
| **MTPD** | Maximum tolerable period of disruption — the longest period an activity/service can be disrupted before impact becomes unacceptable (ISO 22301 vocabulary) |
| **MBCO** | Minimum business continuity objective — the minimum level of service that must be achieved, and by when, during a disruption |
| **RTO** | Recovery time objective — the target time within which a system/process is restored after a disruption |
| **RPO** | Recovery point objective — the maximum acceptable data loss (the point in time to which data must be recovered) |
| **SRTO** | Service recovery time objective — MAS's expected recovery timeline per critical business service (BCM Guidelines 2022) |
| **BCP** | Business continuity plan — how each critical service/function is recovered and maintained at the MBCO |
| **CMP** | Crisis-management plan — how the firm manages the event: command structure, decision rights, communications, notifications |
| **DR** | Disaster recovery — restoring the technology estate (tiers: cold/warm/hot/active-active) against RTO/RPO targets |
| **BCMS** | Business continuity management system — the management system ISO 22301:2019 specifies |
| **PDCA** | Plan–do–check–act — the improvement cycle the BCMS runs on |
| **Severe-but-plausible scenario** | The scenario standard for resilience testing (PRA); scenarios test inside/outside tolerance rather than loss |
| **Tolerance breach** | A live or tested disruption that exceeds an IBS's impact tolerance — the headline resilience metric |
| **TLPT** | Threat-led penetration testing — DORA's advanced resilience testing for significant entities (RTS EU 2025/1190) |
| **Resilience-by-design** | Designing services so recovery objectives are architectural requirements, single points of failure are removed, and designs are testable |
| **PSMOR** | Principles for the Sound Management of Operational Risk (BCBS, 2003/2011/2021) |
| **DORA** | Digital Operational Resilience Act — Regulation (EU) 2022/2554, applied 17 January 2025 |
| **CTPP** | Critical ICT third-party provider — subject to DORA's EU oversight framework |
| **Register of information** | DORA's required inventory of ICT third-party arrangements (entity and consolidated level) |
| **MAS-Tx** | MAS's reporting platform for FI incident submissions (from 1 February 2026) |

### 14.4 Cross-references in this series

| Guide | Why it matters here |
|---|---|
| [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) | The **risk framework** this guide plugs into — three lines of defence, risk appetite, the risk register (§3, §6) |
| [MAS Regulations, Guidelines and Industry Expectations Guide](mas_regulations_guidelines_guide.md) | The **MAS instrument map** — notices/guidelines naming, the BCM/TRM/outsourcing/incident-reporting instruments, the enforcement record (§2.1) |
| [Banks in Singapore Guide](banks_in_singapore_guide.md) | The **MAS expectations for Cymbal Bank** and the 2023–24 outage episode behind the operational-resilience push |
| [Vendor Management Guide](../management/vendor_management_guide.md) | **Third-party risk** — the TPRM lifecycle, MAS Notice 658/1121, the DORA ICT third-party regime and register of information (§10) |
| [Cybersecurity Guide](../technology/cybersecurity_guide.md) | **Cyber resilience** — the threat side of the resilience scenarios, security programmes (§1, §6.2) |
| [OpenShift Workload Availability Guide](../technology/openshift_workload_availability_guide.md) | **HA/DR/BC mechanics** — availability concepts, DR maturity, automated DR, DR drills (§8, §11.3) |
| [IBM Cloud Guide](../technology/ibm_cloud_guide.md) | **Site strategy building blocks** — regions, multizone regions, data centres (§8.1) |
| [Securing Red Hat OpenShift Guide](../technology/secure_red_hat_openshift_guide.md) | **Data protection layer** — etcd encryption, backup keys, restore capability (§8.3) |
| [Workshop Methodology Guide](../management/workshop_methodology_guide.md) | **Tabletop facilitation** — the craft of running the resilience exercises (§11.3) |

*End of guide. Companion reading: enterprise_risk_management_guide.md (the risk framework the resilience programme sits in), mas_regulations_guidelines_guide.md (the MAS instruments), vendor_management_guide.md §8 (the third-party half), openshift_workload_availability_guide.md §29 (the DR-drill mechanics) — and the discipline that keeps the lights on: the resilient bank.*
