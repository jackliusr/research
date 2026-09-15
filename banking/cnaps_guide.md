# CNAPS: The China National Advanced Payment System

> **This guide is a deep-research reference on China's domestic interbank clearing and settlement backbone — the China National Advanced Payment System (CNAPS, 中国现代化支付系统) and its component systems: the High Value Payment System (HVPS, 大额支付系统), the Bulk Electronic Payment System (BEPS, 小额批量支付系统), and the Internet Banking Payment System (IBPS, 网上支付跨行清算系统).** It owns the *system itself*: how the components are constituted, how clearing and settlement mechanics actually work, who the participants are and what access requires, what messaging and standards apply, what the operating windows and settlement-account arrangements are, and — the practical question for a non-Chinese bank — by which corridors RMB reaches CNAPS settlement, including the cross-border gateway CIPS (人民币跨境支付系统). It is deliberately scoped to the PBOC-operated domestic infrastructure and its interfaces; the international *comparative* frame (how CNAPS compares with Fedwire, T2, CHAPS, MEPS+ and the like) lives in the umbrella guide, not here.
>
> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Payment system infrastructure — CNAPS components (HVPS/BEPS/IBPS), clearing and settlement mechanics, participant tiers, message standards, operating windows, and RMB corridor access for a non-Chinese bank  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Primary Sources:** People's Bank of China (PBOC, 中国人民银行) payment-system operation reports and 支付体系运行总体情况 disclosures; BIS Committee on Payments and Market Infrastructures (CPMI) Red Book chapter *Payment, clearing and settlement systems in China* (2012, and successor material); IMF/World Bank assessments; the CIPS operator (cips.com.cn); 移动支付网, Finextra, Reuters and other payment-industry press  
> **Last Updated:** 2026-09  
> **Companion guides:** [financial_infrastructure_guide.md](financial_infrastructure_guide.md) (umbrella; owns the international RTGS/CSD/CCP comparative frame), [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) (Chinese core-banking landscape), [chinese_core_banking_vendors_guide.md](chinese_core_banking_vendors_guide.md) (vendors and connector certification), [payment_rails_guide.md](payment_rails_guide.md) (rail taxonomy and architecture), [payments_hub_guide.md](payments_hub_guide.md) (hub architecture), [singapore_fintech_payments_guide.md](singapore_fintech_payments_guide.md) (SG market), [shanghai_huateng_software_guide.md](shanghai_huateng_software_guide.md) (支付清算 message-exchange vendor deep-dive)

---

## Table of Contents

1. [Overview and Identity: What CNAPS Is, and What It Is Not](#1-overview-and-identity-what-cnaps-is-and-what-it-is-not)
   - 1.1 [Whose System It Is](#11-whose-system-it-is)
   - 1.2 [The Problem It Solves](#12-the-problem-it-solves)
   - 1.3 [What It Replaced](#13-what-it-replaced)
   - 1.4 [The Component Map: HVPS, BEPS, IBPS](#14-the-component-map-hvps-beps-ibps)
   - 1.5 [Inside the Umbrella vs Outside It: CICS, ECDS, CIPS, NUCC](#15-inside-the-umbrella-vs-outside-it-cics-ecds-cips-nucc)
2. [History and Generations](#2-history-and-generations)
   - 2.1 [The Pre-CNAPS Arrangement](#21-the-pre-cnaps-arrangement)
   - 2.2 [Generation One: Phased Rollout](#22-generation-one-phased-rollout)
   - 2.3 [Generation Two: What Actually Changed](#23-generation-two-what-actually-changed)
   - 2.4 [Capability Additions Since: IBPS and the Extended Windows](#24-capability-additions-since-ibps-and-the-extended-windows)
   - 2.5 [Dated Timeline Table](#25-dated-timeline-table)
   - 2.6 [Conflicting and Secondary Records](#26-conflicting-and-secondary-records)
3. [Architecture and Participants](#3-architecture-and-participants)
   - 3.1 [NPC and CCPC: The Two-Tier Topology](#31-npc-and-ccpc-the-two-tier-topology)
   - 3.2 [Participant Tiers: Direct, Indirect, Specially-Licensed](#32-participant-tiers-direct-indirect-specially-licensed)
   - 3.3 [Settlement Accounts at the Central Bank](#33-settlement-accounts-at-the-central-bank)
   - 3.4 [Access Modes and the Mechanics of Indirect Access](#34-access-modes-and-the-mechanics-of-indirect-access)
4. [HVPS in Operation](#4-hvps-in-operation)
   - 4.1 [RTGS Mechanics and Operating Windows](#41-rtgs-mechanics-and-operating-windows)
   - 4.2 [Queuing, Gridlock and Liquidity-Saving](#42-queuing-gridlock-and-liquidity-saving)
   - 4.3 [Intraday Liquidity and Collateralised Funding](#43-intraday-liquidity-and-collateralised-funding)
   - 4.4 [Value Thresholds, Fees, and Business Types](#44-value-thresholds-fees-and-business-types)
   - 4.5 [Message Types: What Is and Is Not Publicly Documented](#45-message-types-what-is-and-is-not-publicly-documented)
5. [BEPS in Operation](#5-beps-in-operation)
   - 5.1 [Netting and Batching Model](#51-netting-and-batching-model)
   - 5.2 [Cycles, Cutoffs and Business Types](#52-cycles-cutoffs-and-business-types)
   - 5.3 [How a Bank's Settlement Obligation Arises and Is Discharged](#53-how-a-banks-settlement-obligation-arises-and-is-discharged)
   - 5.4 [Settlement Finality: BEPS Contrasted with HVPS](#54-settlement-finality-beps-contrasted-with-hvps)
6. [IBPS and the Retail Layer](#6-ibps-and-the-retail-layer)
   - 6.1 [IBPS Launch and 24×7 Positioning](#61-ibps-launch-and-247-positioning)
   - 6.2 [What IBPS Enabled That BEPS Could Not](#62-what-ibps-enabled-that-beps-could-not)
   - 6.3 [NetsUnion (网联/NUCC): The Separate Retail Clearing House](#63-netsunion-网联nucc-the-separate-retail-clearing-house)
   - 6.4 [Interaction with the Mobile and Retail Payment Landscape](#64-interaction-with-the-mobile-and-retail-payment-landscape)
7. [Cross-Border Dimension](#7-cross-border-dimension)
   - 7.1 [CIPS: What It Is and What It Is Not](#71-cips-what-it-is-and-what-it-is-not)
   - 7.2 [The Domestic Leg: How CIPS Settles Inside CNAPS](#72-the-domestic-leg-how-cips-settles-inside-cnaps)
   - 7.3 [Participant Types, Standards, Operating Hours](#73-participant-types-standards-operating-hours)
   - 7.4 [The DVP / Securities Question](#74-the-dvp--securities-question)
   - 7.5 [Practical Corridor Routes for a Non-Chinese Bank](#75-practical-corridor-routes-for-a-non-chinese-bank)
8. [Technical Standards and Messaging](#8-technical-standards-and-messaging)
   - 8.1 [The PBOC Message Specifications](#81-the-pboc-message-specifications)
   - 8.2 [The ISO 20022 Question](#82-the-iso-20022-question)
   - 8.3 [Connectivity and Network Arrangements](#83-connectivity-and-network-arrangements)
   - 8.4 [Certification and Interfaces](#84-certification-and-interfaces)
9. [Scale, Volumes, International Comparison](#9-scale-volumes-international-comparison)
   - 9.1 [Latest Published Figures](#91-latest-published-figures)
   - 9.2 [Growth Trend](#92-growth-trend)
   - 9.3 [Comparison Discipline: Why the Numbers Do Not Line Up](#93-comparison-discipline-why-the-numbers-do-not-line-up)
10. [Operational and Regulatory Reality](#10-operational-and-regulatory-reality)
    - 10.1 [What Participation Requires](#101-what-participation-requires)
    - 10.2 [Liquidity and Funding Obligations](#102-liquidity-and-funding-obligations)
    - 10.3 [Contingency, BCP and Reporting](#103-contingency-bcp-and-reporting)
    - 10.4 [Operational Risk Register](#104-operational-risk-register)
11. [Cymbal Bank Worked Example](#11-cymbal-bank-worked-example)
    - 11.1 [Scenario and Scope](#111-scenario-and-scope)
    - 11.2 [Access-Route Decision](#112-access-route-decision)
    - 11.3 [Account and Liquidity Structure](#113-account-and-liquidity-structure)
    - 11.4 [Operating-Window and Cutoff Constraints](#114-operating-window-and-cutoff-constraints)
    - 11.5 [Integration, Messaging and Contingency](#115-integration-messaging-and-contingency)
    - 11.6 [Illustrative Cost Comparison](#116-illustrative-cost-comparison)
    - 11.7 [Risk Assessment and Open Items](#117-risk-assessment-and-open-items)
12. [Claims Audit](#12-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [Closing Summary](#16-closing-summary)

---

## 1. Overview and Identity: What CNAPS Is, and What It Is Not

**The China National Advanced Payment System (CNAPS, 中国现代化支付系统) is the People's Bank of China's (PBOC, 中国人民银行) national interbank clearing and settlement infrastructure — the domestic system of record through which Chinese banks settle obligations to one another in central-bank money.** It is not one system but a family of systems sharing one national platform: a large-value real-time gross settlement (RTGS) application, a bulk retail netting application, and a real-time retail application built on top of that netting layer. For a foreign practitioner, the single most useful framing is this: **CNAPS is the sole domestic final-settlement layer for RMB.** Every other domestic retail clearing arrangement — the bankcard network, the third-party payment clearing house, the online-banking channel — resolves into a net position that is ultimately discharged in CNAPS, and specifically through its RTGS component.

That framing matters because the name "CNAPS" is used loosely in trade press and vendor documentation to mean different things. Some sources treat CNAPS as exactly two systems (HVPS + BEPS). Others include IBPS. Others again conflate CNAPS with the entire modern-payment-system programme including cross-border and retail-adjacent systems that are legally and operationally *separate* from it. This guide takes the strict view — **CNAPS = the PBOC national platform and its three core domestic applications (HVPS, BEPS, IBPS)** — and then explicitly maps what sits outside it.

### 1.1 Whose System It Is

CNAPS is owned and operated by the **People's Bank of China**, the central bank. Operationally, the systems are run by the **China National Clearing Center (CNCC, 中国人民银行清算总中心)** — a non-profit public institution established in May 1990 and administered by the PBOC. The CNCC is the entity that operates CNAPS's core applications and related clearing utilities; the PBOC itself sets the rules of participation, holds the settlement accounts in which final settlement occurs, and regulates the system's access regime. In the CPMI taxonomy, CNAPS's HVPS is a **systemically important payment system (SIPS)** — a designation examined in the IMF's 2012 detailed assessment of China against the CPSS Core Principles.

The practical implication of central-bank ownership is that **settlement is in central-bank money, not commercial-bank money**. When a payment settles in CNAPS, it does so by debit and credit of the participants' settlement accounts (清算账户) held at the PBOC. That removes interbank credit risk from the settlement leg in a way that a commercial correspondent arrangement does not — the reason direct HVPS access is prized by banks with material RMB business.

### 1.2 The Problem It Solves

Before CNAPS, interbank RMB payments moved through a fragmented patchwork of local clearing houses, PBOC branch-level manual and semi-manual arrangements, and bilateral correspondent balances. Three problems followed from that:

- **No single national final-settlement point.** A payment between a bank in Guangdong and a bank in Heilongjiang had no national RTGS rail; value moved through intermediate arrangements with timing and credit exposure.
- **No unified message and processing standard.** Each locality or arrangement used its own paper-based or idiosyncratic electronic formats, so straight-through processing was limited and reconciliation was manual.
- **No central visibility.** The central bank could not observe, in aggregate and in real time, the settlement flows across the banking system, weakening its ability both to conduct monetary policy and to monitor systemic risk.

CNAPS solved those by creating one national platform: a two-tier topology (a national processing centre plus regional city centres) with a common message specification, real-time gross settlement for large and time-critical payments, and batch netting for the mass of small retail payments — all settling in PBOC accounts.

### 1.3 What It Replaced

The predecessor arrangements are described in the BIS/CPMI Red Book chapter on China (2012) and in PBOC historical material. In summary, pre-CNAPS interbank value moved through:

- **Local clearing houses (同城票据交换所)** — city-level clearing of paper instruments and local electronic items, run under PBOC branch auspices, with net positions settled locally.
- **The PBOC's own electronic interbank system (电子联行系统, EIS-equivalent)** — the earlier satellite-based national electronic interbank funds-transfer arrangement, which handled inter-city transfers but without true real-time RTGS settlement and with limited throughput.
- **Manual and semi-manual arrangements** for many payment types, including paper-based interbank settlement.

CNAPS subsumed these functions into a single platform. The local clearing houses did not vanish immediately — but the *final settlement* of their net positions migrated into CNAPS. Note: **the English name of the predecessor national electronic interbank arrangement is recorded inconsistently across sources**; this guide does not assert a definitive official English acronym for it (see Section 13).

### 1.4 The Component Map: HVPS, BEPS, IBPS

The three core CNAPS applications serve distinct payment segments:

| Component | Chinese name | English gloss | Model | Typical business | Launched |
|---|---|---|---|---|---|
| **HVPS** | 大额支付系统 | High Value Payment System / High-value real-time payment system | RTGS — each payment settled individually and irrevocably in central-bank money | Interbank lending, securities and FX settlement, large corporate transfers, monetary-policy operations, final net settlement of every other domestic system | June 2005 (first generation); upgraded under CNAPS II |
| **BEPS** | 小额批量支付系统 | Bulk Electronic Payment System (also rendered "small-amount batch payment system") | Deferred net settlement (DNS) — payments batched, multilaterally netted, net positions settled in HVPS | Payroll, recurring bill and utility payments, government benefits, pensions, direct debits, high-volume low-value transfers | June 2006 (first generation); upgraded under CNAPS II |
| **IBPS** | 网上支付跨行清算系统 | Internet Banking Payment System (colloquially 超级网银, "super online banking") | Real-time transmission and response for clearing, with net positions settled in batches through HVPS | Online/mobile banking transfers, cross-bank balance and account queries, online signing and fund aggregation | 30 August 2010 (first production system of CNAPS II); nationwide from 24 January 2011 |

Two observations that trip up outsiders:

1. **BEPS is often described as "embedded in" HVPS** — meaning BEPS does not hold its own settlement accounts; it calculates net positions and instructs settlement through HVPS. The two share the CNAPS platform and, historically, nearly identical participant counts.
2. **IBPS is sometimes mis-dated as a "CNAPS II addition" without its sub-generation nuance.** It launched in 2010 as the *first* production system delivered under the second-generation programme — ahead of the main CNAPS II cutover. Its operating model is real-time clearing with deferred net settlement, not RTGS; it is not "China's RTGS."

A fourth application is operated by the same centre and is often grouped with the above for convenience: the **China Foreign Exchange Payment System (CFXPS, 中国外汇交易支付系统)**, formerly rendered as the China Domestic Foreign Currency Payment System (CDFCPS/FCPS). Established in 2008, it is a dedicated RTGS system for domestic transactions denominated entirely in foreign currencies (USD, EUR, GBP, JPY, HKD, AUD, CAD, CHF), with proxy settlement banks named by the PBOC. The English transcription of its Chinese name changed from CDFCPS to CFXPS in the early 2020s. It is **not** part of the strict three-component CNAPS definition used here, but it shares the CNCC platform and settles alongside CNAPS — treat it as an adjacent, same-operator system.

### 1.5 Inside the Umbrella vs Outside It: CICS, ECDS, CIPS, NUCC

The phrase "modern payment system (现代化支付系统)" is an umbrella whose boundary is drawn differently by different authors. The practical distinctions that matter:

| System | Chinese name | Status relative to CNAPS | What it does |
|---|---|---|---|
| **CNAPS core** | 中国现代化支付系统 | The system itself | HVPS + BEPS + IBPS; the domestic final-settlement layer for RMB |
| **CFXPS** | 中国外汇交易支付系统 | Adjacent, same operator (CNCC), separate mandate | Domestic interbank foreign-currency clearing and settlement |
| **Cheque-image system — CICS** | 全国支票影像交换系统 | Separate, pre-dates CNAPS 2 thinking | Nationwide electronic exchange of cheque images; net settlement via CNAPS. Not a CNAPS component |
| **Commercial-paper system — ECDS** | 电子商业汇票系统 | Separate system | Electronic commercial drafts/bills (电子商业汇票) issuance, acceptance, discounting and clearing; settlement of funds legs flows through CNAPS. Operated in the PBOC/上海票据交易所 (Shanghai Commercial Paper Exchange) orbit, not as a CNAPS application |
| **Cross-border system — CIPS** | 人民币跨境支付系统 | Separate system, separate operator (CIPS Co. in Shanghai); the domestic leg settles through CNAPS | Cross-border RMB clearing and settlement (Section 7) |
| **Retail third-party clearing house — NetsUnion/NUCC** | 网联清算有限公司 | Separate private-sector utility, PBOC-mandated | Clears transactions between third-party payment institutions (Alipay, WeChat Pay and peers) and banks; net positions settle through CNAPS (Section 6.3) |
| **Bankcard interbank system — UnionPay** | 中国银联银行卡跨行交易清算系统 | Separate, China UnionPay-operated | Card transaction clearing; net positions settle through CNAPS |

The pattern to internalise: **CNAPS is the settlement layer; most of the surrounding named systems are clearing layers that resolve into it.** CIPS, NUCC, UnionPay, ECDS and CICS all ultimately produce net or gross obligations that are discharged through HVPS in PBOC money. That is why HVPS's transaction count is modest by Chinese standards while its value throughput makes it one of the largest RTGS systems on earth — it settles not only its own transfers but the final net positions of the entire domestic stack.

---

## 2. History and Generations

### 2.1 The Pre-CNAPS Arrangement

Chinese accounts of the country's payment-system development describe **three stages**: manual interbank clearing (银行手工联行), the electronic interbank system (电子联行), and the modern payment system (现代化支付系统). The first two stages were the problem CNAPS was built to remove.

Under the manual and early electronic regimes, interbank RMB value moved through:

- **City-level clearing houses (同城票据交换所)** handling same-city paper and local electronic items, netting them and settling locally under PBOC branch auspices.
- **A national electronic interbank arrangement** — the 电子联行 stage — which gave inter-city electronic transfer but without true real-time gross settlement, and with limited throughput against a fast-growing economy.
- **Bilateral correspondent and branch-account arrangements**, which left interbank credit exposure and inconsistent processing standards across regions.

The result was a fragmented national picture: no single final-settlement point, no common message specification, and no central real-time visibility of system-wide settlement flows. CNAPS consolidated these into one platform with a two-tier topology (Section 3), one message specification, an RTGS application for large and time-critical payments, and a batch netting application for the retail mass.

> ⚠ **Note on the predecessor's name.** Secondary sources render the pre-CNAPS national electronic interbank arrangement variously (电子联行系统 is the reliable Chinese term). This guide does not assert a definitive official English acronym — for example "EIS" — because the pairing could not be confirmed against a PBOC primary document in this pass. Treat any English acronym seen elsewhere for this predecessor as unverified.

### 2.2 Generation One: Phased Rollout

CNAPS was built as a national programme with a phased, province-by-province rollout — the pattern that explains why different sources give different "launch dates" for the same component.

| Milestone | Date | Status | What the date actually means |
|---|---|---|---|
| China National Clearing Center (CNCC, 中国人民银行清算总中心) established | May 1990 | ✅ (Wikipedia citing ECB 2010; broadly reported) | The operating institution pre-dates CNAPS by over a decade |
| HVPS enters production | October 2002 | ⚠ | Widely stated as 2002年10月投产; a Chinese source places nationwide completion at 1 December 2003 (甘肃 as the final province). Secondary sources |
| HVPS nationwide completion / "domestic RTGS launched" | June 2005 | ⚠ | Wikipedia/CPMI-drawn summary states China's domestic RTGS "launched in June 2005", citing the PBOC's 2007 payment-system development report. Best read as *completion of the national rollout*, not first go-live |
| BEPS (小额批量支付系统) launched | June 2006 | ✅ (Wikipedia citing PBOC 2007) | Retail batch application goes live |
| Cheque Imaging System (CIS/CICS, 全国支票影像交换系统) launched | June 2007 | ✅ (Wikipedia citing IMF 2018 FSAP technical note) | Separate from CNAPS; net settlement at HVPS |
| CFXPS (then CDFCPS, 境内外币支付系统) established | 2008 | ✅ (Wikipedia citing IMF 2012) | Dedicated domestic foreign-currency RTGS; adjacent to CNAPS, same operator |
| IBPS (网上支付跨行清算系统) production | 30 August 2010 | ✅ (CNCC official page) | First CNAPS-II system delivered, ahead of the main cutover |
| IBPS extended nationwide | 24 January 2011 | ✅ (CNCC official page) | Completion of national rollout |
| CNAPS second generation (第二代支付系统) fully in production | 8 October 2013 | ⚠ (conflicting) | Chinese encyclopaedia sources give 8 October 2013; one widely-cited source gives 6 October 2013. See 2.6 |

The **practical takeaway** is that "CNAPS launched in 2002/2005" and "CNAPS launched in 2010/2013" are both defensible statements about different things: the first-generation HVPS/BEPS platform versus the second-generation architecture that superseded it.

### 2.3 Generation Two: What Actually Changed

The second generation (CNAPS II / 第二代支付系统) was the PBOC's re-platforming of the whole national clearing service onto a single central-bank clearing platform, built on the first generation's operating experience. Verified and flagged changes:

- **A unified national central-bank clearing platform, delivered in phases.** The programme delivered systems sequentially, with **IBPS first (August 2010, nationwide January 2011)** and the main cutover to the second-generation platform in **October 2013**. ✅ for IBPS dates (CNCC); ⚠ for the October 2013 main cutover date.
- **Separation of message transmission from business processing.** Chinese technical documentation for CNAPS II describes the design goal as 实现报文传输和业务处理分离 ("separating message transmission from business processing"), to allow participants more flexible access and to simplify business-system logic. ⚠ (technical vendor/summary documentation, not a PBOC specification document).
- **Support for indirect ("间联") participants** with a participant-side business management subsystem, alongside direct access. ⚠ same source class.
- **IBPS as a core business subsystem of the second generation**, supporting cross-bank payment, cross-bank account-information queries, and online signing/distribution of agreements (在线签约) with real-time transmission and response. ✅ (CNCC).
- **Extended HVPS operating hours**, ultimately a consequence of the second-generation platform's capacity to run longer windows. See 2.4.

**What is *not* verified in this pass:** the full itemised CNAPS-II feature list, the precise technical re-platforming scope (hardware, database, networking), and any claim about specific new message standards introduced at the second-generation cutover. Where vendors claim CNAPS II "added ISO 20022 messaging," that claim is not supported by the primary material reviewed here (Section 8).

### 2.4 Capability Additions Since: IBPS and the Extended Windows

**IBPS (2010–2011).** IBPS is the single most consequential post-CNAPS-I capability addition for retail banking. It is a real-time *clearing* system (each payment gets an immediate response) with **deferred net settlement** — the net positions settle through HVPS. It handles cross-bank payments, cross-bank account-information queries, and online agreement signing, and it is positioned for 24×7 retail availability. ✅ (CNCC official page).

**HVPS extended operating windows — the verified sequence.**

| Date | Change | Status |
|---|---|---|
| 1990s-era baseline / pre-2018 | HVPS ran **5×12 hours** — business acceptance from 08:30 on the working day (T), business cutoff (clearing-window opening) 17:00 | ✅ (CNCC 2018 notice describes the outgoing 5×12 regime) |
| **22 January 2018 (Monday)** | HVPS moved to **5×21 hours**: business acceptance moved from 08:30 on T to **23:30 on T-1**; business cutoff (clearing-window opening) moved from 17:00 to **17:15**; **clearing window 17:15–20:30**. Service days remained statutory working days | ✅ (CNCC notice, published 19 January 2018) |
| Current schedule as documented in July 2026 | Described as **"5×21+12 hours"**: on statutory working days, business acceptance begins at **20:30 on T-1** and **day-end is 17:30 on T**; on weekends and statutory holidays, HVPS continues under **special working-day arrangements** (特殊工作日) | ✅ (CNCC notice, published 28 July 2026) |
| **From 17 October 2026 (Saturday)** | HVPS service time extended from **"5×21+12 hours" to "5×22.5+13.5 hours"** | ✅ (CNCC notice, 28 July 2026) |
| **From 19 October 2026 (Monday)** | On statutory working days, business acceptance moves from **20:30 on T-1 to 19:00 on T-1**; **day-end remains 17:30 on T** | ✅ (CNCC notice, 28 July 2026) |

Two things worth flagging from the primary notices:

1. **The 2018 extension was explicitly justified in part by CIPS.** The CNCC's own notice states the extension "will effectively support the operating-time requirements of the Renminbi Cross-Border Interbank Payment System (CIPS), preparing RMB cross-border clearing services to cover the world's major time zones." That is a dated, primary confirmation that CIPS's time-zone coverage depended on a prior HVPS window extension.
2. **The intermediate step is unverified.** Between the January 2018 regime (acceptance from T-1 23:30, clearing window to 20:30) and the schedule described in July 2026 (acceptance from T-1 20:30, day-end 17:30), the window evidently shifted roughly three hours earlier. **The notice effecting that intermediate change was not located in this pass.** ⚠ Treat any specific claim about *when* the move to a T-1 20:30 start and a 17:30 day-end happened as unverified.

**Reporting discipline:** cutoffs, window start times and day-end times on CNAPS change by CNCC notice, sometimes with only weeks' notice, and trade-press and vendor documentation lags badly. Any figure in this guide should be re-checked against the current CNCC 信息公告 before being used operationally.

### 2.5 Dated Timeline Table

| Year / date | Event | Source class | Marker |
|---|---|---|---|
| May 1990 | CNCC (清算总中心) established under the PBOC | Secondary (Wikipedia citing ECB) | ✅ |
| Oct 2002 | HVPS enters production (first generation) | Secondary (Chinese technical/press) | ⚠ |
| 1 Dec 2003 | HVPS rollout said to reach its final province (甘肃) | Secondary (Chinese encyclopaedia) | ⚠ |
| June 2005 | CNAPS first generation RTGS nationally established (per PBOC 2007 report framing) | Strong/secondary (PBOC report via Wikipedia) | ⚠ |
| June 2006 | BEPS launched | Strong (PBOC 2007 report via Wikipedia) | ✅ |
| June 2007 | Cheque Imaging System launched | Strong (IMF FSAP 2018 via Wikipedia) | ✅ |
| 2008 | CFXPS/CDFCPS established | Strong (IMF 2012 via Wikipedia) | ✅ |
| 30 Aug 2010 | IBPS in production | **Primary (CNCC official page)** | ✅ |
| 24 Jan 2011 | IBPS nationwide | **Primary (CNCC official page)** | ✅ |
| 8 Oct 2013 | CNAPS second generation fully in production | Secondary; conflicting (alt. 6 Oct 2013) | ⚠ |
| 8 Oct 2015 | CIPS Phase 1 launched — 19 direct / 176 indirect participants, 50 countries | Secondary (Wikipedia from CIPS material) | ✅ |
| 25 Mar 2016 | SWIFT–CIPS MoU; CIPS messaging based on ISO 20022 | Secondary (SWIFT press release) | ✅ |
| 31 Mar 2017 | NetsUnion trial operation begins | Secondary (Chinese trade press) | ⚠ |
| 29 Aug 2017 | NetsUnion Clearing Corporation (NUCC) incorporated | Secondary (Baidu Baike) | ⚠ |
| 22 Jan 2018 | HVPS moves to 5×21-hour operation | **Primary (CNCC notice)** | ✅ |
| 26 Mar 2018 | CIPS Phase 2 pilot, 10 direct participants | Secondary (Wikipedia from CIPS material) | ⚠ |
| 2 May 2018 | CIPS Phase 2 fully operational | Secondary | ⚠ |
| 9 Oct 2018 | CIPS Phase 2 introduces DVP; supports Bond Connect Northbound | Secondary | ⚠ |
| 30 Jun 2018 | PBOC deadline for third-party payment institutions to route clearing through NUCC ("断开直连", 断直连) | Secondary (trade press) | ⚠ |
| 28 Jul 2026 | CNCC announces HVPS extension to "5×22.5+13.5 hours" effective 17 October 2026 | **Primary (CNCC notice)** | ✅ |

### 2.6 Conflicting and Secondary Records

| Item | Conflict | Resolution adopted here |
|---|---|---|
| CNAPS II cutover date | 8 October 2013 (Chinese encyclopaedia entry) vs 6 October 2013 (another widely-cited Chinese wiki). Both fall on or near the National Day holiday week, which is consistent with a holiday-window cutover | Reported as **October 2013**, with 8 October as the more commonly cited date, marked ⚠ |
| "When CNAPS launched" | 2002 (HVPS production), 2005 (national RTGS completion), 2010 (first CNAPS-II system), 2013 (second generation) all appear | Resolved by component and phase — see 2.2 |
| HVPS operating hours | Sources variously give 08:30–17:00 (obsolete), 23:30 T-1 with 17:15–20:30 clearing window (the January 2018 regime), and T-1 20:30 with 17:30 day-end (the regime in force in 2026) | All dated; the **current** regime is the 2026-notice one, and it changes again on 19 October 2026 |
| Number of CCPCs | Commonly stated as 32 city processing centres; the count has changed as the network was consolidated | Marked ⚠ in Section 3 |
| Chinese name of CNAPS | 中国现代化支付系统 (mainland simplified) vs 中國現代化支付系統 (traditional) — same term, different scripts; note the acronym CNAPS expands from the *English* rendering, not the Chinese | Both scripts accepted; see Section 14 |

---

## 3. Architecture and Participants

### 3.1 NPC and CCPC: The Two-Tier Topology

CNAPS is a **two-tier national topology**:

- **National Processing Center (NPC, 国家处理中心)** — operated by the CNCC, this is the central node. It hosts the system of record: settlement accounts, the RTGS queue, the net-position calculations, and the messaging hub. It is the single point through which interbank settlement finality is achieved.
- **City Clearing Processing Centers (CCPC, 城市处理中心)** — regional nodes connecting local participants to the national centre. Historically the deployment was described as **32 city processing centres**; the number reflects China's administrative and PBOC branch geography and has been consolidated over time.

Participants connect to their local CCPC; CCPCs connect to the NPC. This is the classic two-tier design seen in large-country RTGS systems: it localises access and messaging for institutions that are only locally significant, while keeping a single national settlement point. It also concentrates continuity risk in the NPC — the single most important operational fact about CNAPS (Section 10.4).

> ⚠ **CCPC count.** "32 CCPCs" appears in secondary guides and is consistent with historical descriptions, but an authoritative current count was **not** verified from a PBOC/CNCC primary page in this pass. The direction of travel is consolidation. Do not quote "32" as current without re-checking.

### 3.2 Participant Tiers: Direct, Indirect, Specially-Licensed

CNAPS distinguishes participants by the way they access the system and settle:

| Tier | Chinese | Definition | Settlement |
|---|---|---|---|
| **Direct participant** | 直接参与者 | An institution with its own **settlement account (清算账户)** at the PBOC and a direct technical connection to the system (via its CCPC) | Settles its own obligations directly in central-bank money |
| **Indirect participant** | 间接参与者 | An institution that accesses the system *through* a direct participant, without its own PBOC settlement account; the direct participant settles for it | Settles through its direct participant's account — introducing that direct participant's credit and liquidity risk |
| **Specially-licensed participant** | 特许参与者 | Under PBOC rules, a non-bank entity admitted to participate in specified CNAPS business — the category used for market infrastructures and central clearing counterparties such as the clearing houses and exchanges that settle through CNAPS | Settles according to PBOC-specified arrangements; typically as a systemically important infrastructure rather than a bank |

The **specifically named participants** are instructive. The CNCC's own 2018 notice lists, as HVPS participants: the large state-owned commercial banks, but also **China UnionPay, the China Foreign Exchange Trade System (CFETS), China Central Depository & Clearing (CCDC), Shanghai Clearing House, the Shanghai Commercial Paper Exchange, and NetsUnion (网联公司)**. That list is the clearest public statement of who the specially-licensed/central-infrastructure participants are — and it is primary.

**Scale of the tiers, with dates:**

| Metric | End-2010 | End-2016 | Source |
|---|---|---|---|
| HVPS direct participants | 1,729 | 305 | CPMI 2012 / IMF 2018 (via Wikipedia) |
| HVPS indirect participants | 100,510 | 141,023 | CPMI 2012 / IMF 2018 (via Wikipedia) |
| BEPS direct participants | 1,730 | — | CPMI 2012 |
| BEPS indirect participants | 100,510 | — | CPMI 2012 |
| IBPS direct participants | — | 218 direct + 175 proxy-access (July 2020) | World Bank FPS case study (via Wikipedia) |

The collapse of direct participation between 2010 and 2016 (1,729 → 305) alongside explosive indirect growth is the single most important structural trend in CNAPS participation. It reflects a deliberate consolidation: the PBOC moved to a smaller set of well-capitalised direct participants, each settling for a large tail of indirect ones — concentrating liquidity and risk in fewer nodes. ✅ (the figures) / ⚠ (the interpretive claim about intent).

> ⚠ **Current taxonomy.** The three-tier direct/indirect/specially-licensed framing is well-attested historically. Whether the PBOC's *current* participation rulebook uses exactly this taxonomy, and whether additional sub-categories exist (for example separate classifications for clearing institutions, or for foreign-currency participants on CFXPS), was not verified against a current primary rule text in this pass. Treat the tier names as accurate but the *completeness* of the taxonomy as flagged.

### 3.3 Settlement Accounts at the Central Bank

Final settlement in CNAPS occurs by **debit and credit of settlement accounts (清算账户) held by direct participants at the PBOC**. Key structural points:

- **The account is the settlement asset.** A direct participant's ability to make payments is bounded by the balance and any intraday credit in that account, not by its balance sheet strength. This is why liquidity management at the PBOC account is the core operating discipline for a CNAPS direct participant.
- **BEPS and IBPS do not hold their own settlement accounts.** They compute net positions; the net debit/credit is applied to the participants' PBOC settlement accounts through HVPS. In this sense HVPS is the settlement engine for the entire domestic stack.
- **Indirect participants do not hold a PBOC settlement account** for CNAPS purposes; their positions are folded into their direct participant's obligation.
- **CFXPS** operates with its own proxy-settlement-bank arrangements for foreign currencies (the PBOC originally designated Bank of China, China Construction Bank, ICBC and Shanghai Pudong Development Bank as proxy settlement banks for foreign-currency RTGS).

### 3.4 Access Modes and the Mechanics of Indirect Access

An institution reaching CNAPS generally chooses among:

1. **Direct participation** — a PBOC settlement account, a certified connection to the local CCPC, and full operating and liquidity obligations. Highest cost and highest control.
2. **Indirect participation** — clearing through a direct participant (typically a large commercial bank, or a market infrastructure acting as a settlement agent). No PBOC settlement account. Lower cost, but the institution's payment capacity depends on its agent's willingness and capacity to fund and settle.
3. **Special licensed participation** — for designated infrastructures, on PBOC-specified terms.

**The risk that indirect access creates** deserves precision, because it is the crux of the corridor decision in Section 7.5:

- **Credit risk on the agent.** The indirect participant's payment is only final when the *direct participant* settles in central-bank money. Until then, the indirect participant has an exposure to its agent.
- **Liquidity risk.** If the agent's PBOC account is short, the agent may queue or delay the indirect participant's payments. The indirect participant cannot remedy this directly — it cannot fund the system's settlement account itself.
- **Operational dependency.** The agent's connection to the CCPC, its cutoffs, its message-mapping and its contingency arrangements all become constraints on the indirect participant.
- **Asymmetric visibility.** The indirect participant typically learns of settlement outcomes from the agent, not from the system of record.

The genuine benefit of indirect access is cost: no PBOC membership obligations, no direct CCPC connectivity, no direct liquidity-funding requirement beyond the agent's commercial terms. For banks with modest RMB flow, indirect access via a clearing agent is often the economically rational answer — and that is exactly the trade-off worked through in Section 11.

---

## 4. HVPS in Operation

The High Value Payment System (HVPS, 大额支付系统, also 大额实时支付系统 — "large-amount real-time payment system") is CNAPS's RTGS application and the settlement engine of the entire domestic stack.

### 4.1 RTGS Mechanics and Operating Windows

**Mechanics.** Each payment is transmitted to the system, checked against the sending direct participant's settlement account balance (and any intraday liquidity arrangements), and — if covered — settled individually and irrevocably by debit of the payer's PBOC settlement account and credit of the receiver's. There is no netting of the payment itself and no waiting for a clearing session: settlement is immediate and final on a per-transaction basis. If funds are not available, the payment is **queued** rather than rejected (4.2).

**Operating windows.** HVPS operates on statutory working days, with the window straddling the overnight period. The verified schedule history is in Section 2.4. The schedule in force as this guide is written:

| Element | Value | Source |
|---|---|---|
| Service days | Statutory working days (国家法定工作日), plus **special working-day arrangements (特殊工作日)** on weekends and statutory holidays | CNCC notice, 2026-07-28 |
| Business acceptance opens | **20:30 on T-1** (the preceding calendar day) | CNCC notice, 2026-07-28 |
| Day-end | **17:30 on T** | CNCC notice, 2026-07-28 |
| Descriptor | **"5×21+12 hours"** | CNCC notice, 2026-07-28 |
| Change announced | From **17 October 2026**: "5×22.5+13.5 hours". From **19 October 2026**: acceptance opens **19:00 on T-1**; day-end stays 17:30 | CNCC notice, 2026-07-28 |

Note the practical implication of a T-1 opening: **a bank's HVPS "day" begins the evening before the settlement date.** A payment dated T can be submitted from 20:30 on T-1. Foreign banks and their treasury teams routinely mis-model this, and it is one of the most common sources of cutoff surprises on RMB flows.

> ⚠ **Cutoff figures that are widely quoted but obsolete.** Several trade and vendor sources still describe HVPS as "08:30–17:00" or with a "17:15 business cutoff and 17:15–20:30 clearing window." Those were accurate for the January 2018 regime; the 2026 schedule differs. Always re-check the current CNCC 信息公告.

### 4.2 Queuing, Gridlock and Liquidity-Saving

Queuing in an RTGS system is normal, not a failure: it is how an RTGS avoids rejecting payments while leaving the sender's liquidity to arrive later in the day. CNAPS handles this with a set of mechanisms that are documented in PBOC's HVPS business-handling rules (大额支付系统业务处理办法) as reproduced by secondary sources:

- **Payment queue.** Payments that cannot be settled immediately for lack of funds enter a queue and are settled in sequence as funds arrive.
- **Queue reordering by level (级次).** A participant may send a message to change the order of its queued HVPS payments, but only *within the same level/priority tier* — the queue's tier structure cannot be crossed by participant instruction.
- **Cancellation.** A participant may request cancellation of its queued payments, subject to authorisation.
- **Clearing window (清算窗口).** The system designates a clearing window in which direct participants with insufficient account positions are expected to raise funds so that their queued payments — and, critically, *all queued same-city bill-exchange and other net amounts* — are fully settled before the window closes. The rule that **all queued net amounts must be cleared before the clearing window closes** is the formal backstop against end-of-day gridlock. ⚠ (PBOC rule text as reproduced by secondary sources)
- **Multilateral netting settlement for certain participants.** For specially-licensed participants (the clearing houses and infrastructures), HVPS is not purely RTGS: the settlement account system supports **real-time gross, bilateral netting and multilateral netting** mechanisms. Secondary technical descriptions describe specially-licensed participants receiving multilateral-netting settlement receipts (多边轧差清算回执) and the ability to apply to cancel queued business in a partial-settlement scenario. This is the mechanism by which UnionPay, NetsUnion, CFETS, CCDC and Shanghai Clearing House feed net positions into HVPS. ⚠ (source class: Chinese technical/summary writing, not a PBOC specification document)

**What could not be verified:** the *current* set of liquidity-saving features. In particular:

- Whether CNAPS HVPS offers a formal **multilateral liquidity-saving mechanism (LSM)** or offsetting algorithm for ordinary bank-to-bank payments, comparable to TARGET2's liquidity-saving features or CHAPS's offsetting. The netting mechanisms documented above appear to apply principally to specially-licensed/clearing participants, not to interbank payments generally. ❌ **Not verified — do not assert an LSM equivalent for bank payments.**
- Whether the system offers **bilateral-offset or queued-payment cycling** for ordinary participants. ❌ **Not verified.**
- Whether a **central-bank intraday credit / overdraft** facility exists for HVPS direct participants beyond the automatic pledge financing mechanism. ❌ **Not verified.**

### 4.3 Intraday Liquidity and Collateralised Funding

The one liquidity facility that is well documented, dated and specific is **automatic pledge financing (自动质押融资)** — the PBOC's facility allowing a deposit-taking institution whose settlement-account intraday position is insufficient to pledge bonds to the PBOC and obtain funds automatically to complete settlement.

| Feature | Detail | Date / source |
|---|---|---|
| Nature of the facility | A **clearing convenience arrangement**, not a liquidity tool: the great majority of financing is intraday, repaid at day-end, and has no aggregate effect on the monetary base | PBOC statements, 2017-12 (secondary reproduction) |
| Institutional start | Trial *Provisional Measures* issued 10 November 2005; business formally commenced **8 May 2006** with three banks (ICBC, Industrial Bank, Bank of Beijing); extended 5 September 2006 to nine banks including China Development Bank, Agricultural Bank of China and Bank of Communications | Secondary (encyclopaedic, citing PBOC/press) |
| Current rulebook | 《中国人民银行自动质押融资业务管理办法》 published 13 December 2017, **effective 29 January 2018**, repealing the 2005 provisional measures | ✅ rulebook and effective date; secondary reproduction |
| Eligibility | Members must be **HVPS direct participants** with interbank-market settlement qualification; deposit-taking institutions and other PBOC-approved financial institutions; branches require head-office authorisation and PBOC consent | Secondary |
| Financing limits (as % of paid-in capital) | Development/policy banks, state-owned commercial banks, Postal Savings Bank: **4%** (raised from 2%). National joint-stock commercial banks: **10%** (raised from 2%). City commercial banks and other financial institutions: **15%** (raised from 5%) | 2017 Measures, effective 2018-01-29 — secondary reproduction |
| Pricing | Both intraday and overnight financing at the prevailing **overnight Standing Lending Facility (SLF) rate**; intraday interest calculated **by the hour**, overnight by actual days of use | Same |
| Pledgeable collateral | Widened from government bonds, central bank bills and policy financial bonds to include **PBOC-approved local government bonds and other securities**; haircuts set by the PBOC, in principle **no higher than 90%** | Same |
| Repayment discipline | Financing and repayment are **transaction-by-transaction**; failure to fully repay overnight is treated as overdue with a **3 percentage-point interest surcharge**; overdue beyond three days is treated as **default** | Same |
| Trigger modes | **Automatic** and **manual** | Same |
| Technical rail | The automatic pledge financing system is a **supporting functional module of HVPS**; bond pledge and release are effected through the **CCDC (中央结算公司)** bond depository system | Same |

**Practical reading.** For a bank operating an HVPS settlement account, automatic pledge financing is the principal *system-native* way to avoid queued payments at the margin: it converts a collateral pool into intraday settlement capacity automatically. It is, however, restricted to direct participants — which means it is one of the material economic advantages of direct access, and one of the things an indirect participant simply cannot obtain for itself (Section 3.4).

### 4.4 Value Thresholds, Fees, and Business Types

**Business types.** PBOC's HVPS business-handling rules define the business HVPS processes. As reproduced in secondary sources, these include:

- **Interbank credit transfers above the PBOC-prescribed amount threshold (规定金额起点以上的跨行贷记支付业务).**
- **Urgent interbank credit transfers below that threshold (规定金额起点以下的紧急跨行贷记支付业务).**
- Businesses prescribed by the PBOC for specially-licensed participants — which in practice means the net/gross settlement legs of the securities, FX, card and retail clearing systems that resolve into HVPS.
- Related service messages: queries and replies (查询查复), cancellation requests (撤销申请), reversals (冲正), stop-payment requests (止付申请), return requests (退回申请) and similar.

**On the threshold question — an important correction.** HVPS is *not* threshold-free. The rule text carries an **amount threshold (金额起点)** concept: HVPS is the channel for interbank credit transfers *above* the prescribed threshold, and also for *urgent* credit transfers below it. ⚠ The **current numerical threshold was not verified** in this pass, and it has been adjusted over time; secondary guides that describe HVPS as carrying "any value" or as having "no threshold" are describing the practical outcome (banks route urgent and large payments through HVPS) rather than the rule. Do not quote a threshold figure without checking a current PBOC rule text.

**Fees.** ⚠ **Not verified.** The PBOC operates a fee schedule for CNAPS services, and CNCC's user platform publishes documentation for participants, but the current system-level fee schedule (per-transaction charges by business type, and any exemptions) was **not** obtainable from an authoritative public source in this pass. What banks charge their *customers* for RMB transfers is a separate, commercially and regulatorily constrained matter. Any cost model must be built on the bank's own contracted terms with its clearing agent or with the CNCC — not on published figures.

**Value composition.** HVPS's economics are dominated by value, not count. In 2023 it processed roughly 382 million transactions worth about RMB 8,481 trillion (Section 9). The average transaction is therefore on the order of RMB 20 million — a reminder that HVPS's transaction population is dominated by institutional, market and settlement flows, not by customer payments.

### 4.5 Message Types: What Is and Is Not Publicly Documented

CNAPS participants exchange messages using **PBOC's own message specification (报文标准 / 报文规范)**, distributed to participants and via the CNCC's document services. This is *not* a public, freely published interface specification in the way ISO 20022 message definitions are.

**This guide deliberately does not reproduce HVPS or BEPS message-type codes.** The reason is straightforward: the authoritative PBOC message catalogue could not be obtained in this pass, and message codes, field tags and business-type codes are exactly the class of detail that plausible-looking invention gets wrong in a way that is expensive and embarrassing. Where a vendor document or a consultancy deck lists CNAPS message codes, treat them as vendor-derived and verify against the actual CNCC specification before use.

What *can* be said with confidence:

- The design principle adopted in the second generation was **separation of message transmission from business processing** (报文传输和业务处理分离), which is why third-party "payment message transmission" components and participant access platforms exist alongside the business systems. ⚠
- The participant-side integration surface includes a **participant access/business management subsystem** for indirect access (间联). ⚠
- CNAPS business messages carry a **business type (业务种类)** field — this is stated in the CIPS rules for the cross-border system and is standard practice in the domestic systems too. ✅ for CIPS (Article 14 of the 2025 CIPS Rules); ⚠ for the extrapolation to HVPS/BEPS.
- The CNCC operates a **participant certification process** with published material templates — the user platform at ccsp.cncc.cn publishes 参与机构认证材料模板. ✅ (existence of the process and templates)

---

## 5. BEPS in Operation

The Bulk Electronic Payment System (BEPS, 小额批量支付系统, also 小额支付系统 — "small-amount batch payment system") is CNAPS's retail netting application. It is the workhorse for high-volume, low-value, non-urgent payments.

### 5.1 Netting and Batching Model

BEPS is a **deferred net settlement (DNS)** system, and its netting is more granular than the "end-of-day single net" that the label might suggest:

- Payment instructions are **transmitted in batches (批量发送)**; payment obligations are **settled on a net basis (轧差净额清算)**. ✅ (PBOC framing via secondary sources)
- Netting is performed **bilaterally in real time between the payer's clearing bank and the payee's clearing bank (以付款清算行和收款清算行为对象实时双边轧差)** — i.e. the offsetting object is the pair of clearing banks, not the whole membership. ⚠ (secondary, but specific and internally consistent)
- At the **daily cut-off (日切)**, the system performs real-time bilateral netting over the instructions in the unified session, computes each party's net settlement amount, and **submits the net amount in real time to SAPS** — the PBOC's settlement account management system (清算账户管理系统) — for settlement. ⚠
- **Net debit cap (净借记限额).** A BEPS payment is admitted to the netting set **only after passing a net debit limit check**; and once admitted to netting, **the payment's finality is established and it cannot be revoked.** ⚠
- BEPS also physically follows the two-tier structure — an NPC/CCPC split — and commercial banks may access BEPS **either directly through the NPC or through a CCPC**. UnionPay participates in BEPS as a special participant through the NPC for agency-acquiring services. ⚠

The **net debit cap** is the single most important risk feature of BEPS and the thing most often omitted from summaries. It is the standard DNS control: by capping how large a net debit any participant can accumulate before settlement, the system bounds the credit exposure that would otherwise build up between net settlement points. It is also the reason a bank can be **unable to send a BEPS payment even with ample customer balances** — the constraint is the system's cap, not the customer's funds.

### 5.2 Cycles, Cutoffs and Business Types

**Operating model.** BEPS runs **7×24 continuously**. Its working day is defined as the **previous calendar day 16:00 to the current calendar day 16:00**, and — critically — **fund settlement occurs during HVPS working hours.** In addition, BEPS operates a "full-time" (全时) service over statutory holidays to meet holiday payment needs. ⚠ (secondary but specific; consistent with the general design that BEPS has no settlement accounts of its own)

That split — 24×7 acceptance and processing, HVPS-hours settlement — is the key operational fact for foreign banks. **A BEPS payment accepted at 02:00 on a Sunday is not settled in central-bank money at 02:00 on a Sunday.** It is processed, netted and queued for settlement when HVPS is next running.

**Business types.** BEPS handles both credit and debit businesses, and the PBOC rule set defines a specific business menu:

| Business type | Chinese | Model |
|---|---|---|
| Ordinary credit transfer | 普通贷记业务 | Initiated by the payer's bank; sent to BEPS; netted; forwarded to the payee's bank |
| Regular/scheduled credit transfer | 定期贷记业务 | For payments made periodically — scheduled disbursements, payroll and salary transfers, public-benefit funds. The bank must have an agreement with the payer |
| Ordinary direct debit | 普通借记业务 | Initiated by the payee's bank; forwarded to the payer's bank, which returns a receipt within a prescribed time limit; then netted and forwarded back |
| Regular/scheduled direct debit | 定期借记业务 | For periodic collection — water, electricity, gas and similar utilities. Requires a pre-existing payer authorisation |
| Real-time direct debit | 实时借记业务 | For real-time collection by the payee — including collection under banker's cheques/notes (银行本票、银行汇票的委托收款) |
| Centralised agency collection/payment | 集中代收付业务 | Small-value, public-benefit, convenience-oriented collection and payment through **approved centralised agency collection/payment centres (集中代收付中心)**, restricted to utility and public-benefit institutions **within the centre's own PBOC-branch jurisdiction**; extending the service to other institution types is prohibited |
| Other PBOC-prescribed business | 人行规定的其他支付业务 | Queries and replies, cancellations, reversals, stop-payments, returns, etc. |

**Value limit.** BEPS is characterised as processing **credit payments under RMB 50,000 per transaction (每笔金额5万元以下的贷记支付业务)**, alongside same-city and remote voucher-truncated debit businesses. ⚠ **Flagged:** the RMB 50,000 figure is widely and consistently cited as a BEPS characteristic, but the *current* limit — and whether any business types sit outside it, or whether the limit has been adjusted — was **not** verified against a current PBOC rule text in this pass. Treat "RMB 50,000" as the documented historical figure, not as a live limit to build a payment routing rule on.

**Where the utility-collection use case actually lives.** Note the structure above: 定期借记 for utilities and 集中代收付 for public services are distinct business types with distinct authorisation and jurisdictional rules. A foreign bank building a direct-debit proposition in China is not just implementing a payment message — it is operating inside a business-type regime with its own permissioning and, for centralised collection, explicit geographic and sectoral restrictions.

### 5.3 How a Bank's Settlement Obligation Arises and Is Discharged

Putting 5.1 and 5.2 together, the sequence for a direct participant is:

1. **Instruction intake.** The bank submits or receives BEPS instructions; they are exchanged in batches through the NPC/CCPC.
2. **Netting admission.** Each instruction is checked against the participant's **net debit cap**. Only instructions that pass are admitted to the netting set. Admission establishes finality and the instruction becomes **irrevocable**. ⚠
3. **Real-time bilateral netting.** Between each pair of clearing banks, debits and credits are continuously offset.
4. **Net position determination at 日切.** The system computes each participant's net settlement amount for the session.
5. **Settlement through SAPS in HVPS hours.** The net amount is submitted to the settlement account management system and applied to the participants' PBOC settlement accounts. **This is where the bank's cash actually moves.** ⚠
6. **Day-end reconciliation.** Trial balance is performed on the accounting rule "where there is a debit there must be a credit, and debits and credits must be equal" (有借必有贷，借贷必相等); the bank downloads its statement (对账单), prepares local transactions, nets and reconciles. Where reconciliation shows a mismatch, the CCPC adjusts to the **NPC's data as authoritative (以国家处理中心的数据为准)**. ⚠

That last point — the NPC's record is the source of truth in a dispute — tells a foreign bank everything it needs to know about where evidence sits in the CNAPS architecture.

### 5.4 Settlement Finality: BEPS Contrasted with HVPS

| Dimension | HVPS | BEPS |
|---|---|---|
| Settlement model | Real-time gross settlement — each payment settled individually | Deferred net settlement — bilateral netting, session net positions settled in HVPS hours |
| Settlement timing | Immediate, real time | Net position settled at session cut-off, in HVPS working hours |
| Irrevocability point | On settlement (debit/credit of settlement accounts) | On **admission to the netting set** — before settlement of the net position. BEPS payments once netted are irrevocable ⚠ |
| Risk control | Settlement-account balance, queue, automatic pledge financing, clearing window | **Net debit cap** per participant; multilateral/bilateral exposure between net settlement points |
| Loss-sharing / unsettled-risk | Not applicable in the same way — no netting exposure | ❌ **Not verified:** whether BEPS has a formal loss-sharing or unsettled-obligation arrangement for the failure of a participant between netting and settlement. Do not assume one exists |
| Service window | Statutory working days (with special working-day arrangements), currently T-1 20:30 → T 17:30 | 7×24 acceptance and processing; settlement in HVPS hours |
| Value profile | Large-value/market/institutional; 2023 average ≈ RMB 20m per transaction | Retail; per-transaction credit limit documented at RMB 50,000 ⚠ |

The **finality asymmetry is the thing to hold onto.** A BEPS payment is irrevocable *before* it is settled in central-bank money — which is possible precisely because the net debit cap bounds the system's exposure. A HVPS payment is irrevocable *when* it is settled. For a bank managing its own exposure to a counterparty or agent, that difference is material: an "irrevocable" BEPS instruction does not yet mean settled funds (see [financial_infrastructure_guide.md](financial_infrastructure_guide.md) for the general treatment of settlement finality and its legal basis).

---

## 6. IBPS and the Retail Layer

### 6.1 IBPS Launch and 24×7 Positioning

The Internet Banking Payment System (IBPS, 网上支付跨行清算系统), colloquially 超级网银 ("super online banking"), was **put into production on 30 August 2010 and extended nationwide on 24 January 2011** — the first business system delivered under the second-generation programme. ✅ (CNCC official page)

Its operating model, verified from both CNCC material and the PBOC's own management measures for the system:

- **7×24 uninterrupted operation.** A PBOC office notice promulgating the IBPS management measures states plainly: 网上支付跨行清算系统实行7×24小时不间断运行 ("IBPS operates 24 hours a day, 7 days a week without interruption"), with **the system working day being the natural/calendar day (系统工作日为自然日)** and **fund settlement occurring during the working hours of the settlement account management system (资金清算时间为清算账户管理系统的工作时间)**. The PBOC may adjust operating and settlement times. ✅ (PBOC office notice text, as republished)
- **Real-time transmission and response (实时传输及回应机制).** A customer initiating a payment online receives a final processing result promptly — the result returned is terminal, not provisional. ✅ (CNCC)
- **Business scope:** cross-bank (and same-bank) payment transfers, **cross-bank account information queries (跨行账户信息查询)**, and **online signing (在线签约)** — the last being the mechanism that lets a customer authorise a bank to query or debit accounts at other banks, which is what made "super online banking" account aggregation possible. ✅ (CNCC)
- **Deferred net settlement.** IBPS is a **DNS system**, not RTGS: clearing is real-time, settlement of net positions is deferred into the settlement-account system. ✅ (World Bank FPS case study, via Wikipedia)

**The limit, flagged.** IBPS is characterised as handling online-banking payments **under RMB 50,000 per transaction** in its standard business, with participants and the PBOC able to set limits by transaction type. ⚠ As with the BEPS limit, the RMB 50,000 figure is widely cited and reflects the system's founding design; the **current** limit structure was not verified against a current rule text, and per-transaction limits in Chinese retail systems have been adjusted repeatedly. Do not hard-code it.

### 6.2 What IBPS Enabled That BEPS Could Not

BEPS was built for *batch, non-urgent, low-value* payments and settles net positions in HVPS hours. That model cannot deliver three things that the growth of e-commerce and online banking demanded:

1. **Immediate confirmation.** A BEPS payment's fate is known when the batch is processed and settled; an IBPS payment returns a result in the customer session. This is the difference between "transfer submitted" and "transfer done" — the difference between a workable online checkout and an unusable one.
2. **24×7 availability.** BEPS accepts 24×7 but settles in HVPS hours. IBPS gives consumers the *appearance and the substance* of continuous service: an online transfer works at 23:00 on a Sunday, with the interbank settlement netted and discharged at the next available settlement opportunity. This is a deferred-net real-time rail — structurally the same shape as a modern fast payment system, and why the World Bank's Fast Payments Toolkit treats IBPS as a case study.
3. **Cross-bank account information and authorisation.** BEPS has no equivalent of cross-bank account queries or online signing. IBPS's query and signing services are the rail on which China's retail fund-aggregation and "one app, all accounts" behaviours were built.

In short: **IBPS turned the banking internet from a batch channel into a real-time retail rail.** It is the direct ancestor of the instant-payment expectation Chinese consumers now hold (see [payment_rails_guide.md](payment_rails_guide.md) for the general instant-payment rail taxonomy).

### 6.3 NetsUnion (网联/NUCC): The Separate Retail Clearing House

**NetsUnion is not a CNAPS component.** It is a separate, PBOC-mandated clearing house for third-party payment institutions — but it is essential context because its positions settle through CNAPS, and because conflating it with CNAPS/IBPS is the single most common error in foreign practitioners' mental models of Chinese retail payments.

**What it is.** The **NetsUnion Clearing Corporation (NUCC, 网联清算有限公司)** operates the **Non-bank Payment Institution Network Payment Clearing Platform (非银行支付机构网络支付清算平台)**, commonly 网联. It was **incorporated on 29 August 2017** with the PBOC's approval as a licensed clearing institution, headquartered in Beijing, with the third-party payment providers holding board seats alongside central-bank-affiliated shareholders. ⚠ (secondary; the incorporation date and the shareholding structure are stated by encyclopaedic and press sources, and registered capital figures vary between sources — do not quote a capital figure)

**Why it exists — the 断直连 ("disconnecting direct connections") mandate.** Before NUCC, Alipay, WeChat Pay and their peers each held **direct bilateral connections to hundreds of banks**, clearing customer payments outside any central clearing house and outside the PBOC's direct observation. The PBOC's Payment and Settlement Department issued a notice — 《关于将非银行支付机构网络支付业务由直连模式迁移至网联平台处理的通知》 — requiring banks and payment institutions to complete preparation for NUCC connection and business migration by **15 October 2017**, and providing that **from 30 June 2018 all network payment business accepted by payment institutions that involves bank accounts must be processed through the NUCC platform**. ✅ (the 2018-06-30 mandate date and the 2017-10-15 preparation deadline; secondary/press and encyclopaedic sources, all consistent)

**How completely did the traffic move?** By NUCC's own disclosure at the start of 2019, **99% of cross-institution business had been migrated to the NUCC platform** ✅ (NUCC statement, via People's Daily's financial channel, January 2019). The residual was the tail of institutions completing migration.

**Scale and settlement.** In 2024 the NUCC platform processed **1.03 trillion transactions worth RMB 520.53 trillion**, up 15.59% and 4.54% year on year, averaging 28.27亿笔 (2.827 billion) and RMB 1.42 trillion per day. ✅ (PBOC 2024 payment system report — primary, and note this is PBOC's own reported figure for NUCC, sourced from NUCC). NUCC clears in real time but **nets positions for settlement through CNAPS/HVPS** — the same settlement pattern as UnionPay and the bankcard system. A reported peak of **11万笔/秒 (110,000 transactions per second) on Chinese New Year's Eve 2025** is a useful order-of-magnitude indicator of the platform's capacity requirements. ⚠ (secondary, NUCC statement)

**The precision that matters.** Which traffic moved to NUCC, and which did not:

| Flow | Cleared by | Settled via |
|---|---|---|
| Third-party wallet payment (Alipay, WeChat Pay) funded from or to a bank account | **NUCC** | CNAPS/HVPS net position |
| Payment between two payment-institution accounts (wallet-to-wallet, where not involving a bank account) | Payment institution's own books / NUCC per business type | Not a CNAPS flow |
| Consumer online-banking transfer between two banks | **IBPS** | CNAPS/HVPS net position |
| Card transaction at a merchant terminal | **China UnionPay** interbank system | CNAPS/HVPS net position |
| Bank-to-bank batch retail (payroll, utilities, direct debit) | **BEPS** | CNAPS/HVPS net position |
| Corporate large-value / interbank / market payment | **HVPS** | CNAPS/HVPS (RTGS, its own transactions) |

So the statement "China's mobile payments go through CNAPS" is **imprecise in a specific way**: they are *cleared* by NUCC (not by IBPS, and not by CNAPS itself) and only the resulting **net positions** are *settled* through CNAPS. Conversely, "IBPS is China's fast payment system" is directionally right for bank-channel transfers, but IBPS is not the clearing channel for wallet payments.

### 6.4 Interaction with the Mobile and Retail Payment Landscape

The practical layering for a foreign bank is:

- **Customer-facing retail rails** (IBPS for bank transfers, NUCC for wallets, UnionPay for cards) are the ones a bank's retail or merchant proposition must connect to — each with its own membership, certification and rules.
- **CNAPS is the settlement spine beneath all of them.** A bank that only cares about settlement liquidity and central-bank-money finality can work entirely at the HVPS layer; a bank that wants to *originate* retail payments in China must join the relevant clearing system as well.
- **Retail conduct, licensing and reserve regulation is a separate body of law** — the licensing of non-bank payment institutions, customer reserve fund rules, and wallet limits are their own topics, outside this guide's scope. This guide points at them rather than unwinding them. For the payment-rail architecture view, see [payment_rails_guide.md](payment_rails_guide.md) and [payments_hub_guide.md](payments_hub_guide.md).

---

## 7. Cross-Border Dimension

### 7.1 CIPS: What It Is and What It Is Not

The **Cross-Border Interbank Payment System (CIPS, 人民币跨境支付系统)** is China's dedicated cross-border RMB clearing and settlement system. It is **not** a CNAPS component: it is operated by a separate corporate entity — **Cross-border Interbank Payment Clearing (Shanghai) Co., Ltd. (跨境银行间支付清算有限责任公司)**, headquartered in Shanghai, referred to in the rules as the **operating institution (运营机构)** — and its domestic settlement leg runs through CNAPS. ✅ (PBOC, 《人民币跨境支付系统业务规则》, 银发〔2025〕248号)

Verified history:

| Milestone | Date | Marker |
|---|---|---|
| PBOC launched CIPS construction (Phase 1) | 2012 | ⚠ |
| **CIPS Phase 1 put into operation** — 19 direct and 176 indirect participants from 50 countries and regions | **8 October 2015** | ✅ |
| SWIFT–CIPS MoU signed; CIPS messaging based on ISO 20022 | 25 March 2016 | ✅ |
| CIPS membership data published via SWIFTRef | September 2017 | ⚠ |
| **CIPS Phase 2 pilot, 10 direct participants** | 26 March 2018 | ⚠ |
| **CIPS Phase 2 fully operational** | 2 May 2018 | ⚠ |
| CIPS Phase 2 extended operating hours from 5×12 to **5×24 + 4 hours**, covering all global time zones | with Phase 2 | ⚠ |
| CIPS Phase 2 introduces **DVP** and supports **Bond Connect Northbound** | 9 October 2018 | ⚠ |
| PBOC issues revised **CIPS Business Rules** (银发〔2025〕248号, dated 2025-12-19); supersedes 银发〔2018〕72号 | published in 国务院公报 2026年第5号; **effective 1 February 2026** | ✅ (primary) |

**Current scale of the network.** Per CIPS's own participant announcements: at end-October 2025, CIPS had **187 direct and 1,559 indirect participants**, with indirect participants distributed across Asia (1,143, including 566 in mainland China), Europe (261), Africa (64), North America (34), South America (33) and Oceania (24), across 122 countries and regions. ✅ (CIPS announcement #114, cips.com.cn). By end-June 2026 CIPS reported **210 direct and 1,619 indirect participants** — the first time direct participation passed 200. ✅ (CIPS announcement #118, via payment-industry press). Reach is often described as extending to more than 4,900 banking institutions in 187+ countries through the participant network. ⚠ (CIPS/press claim, not independently audited)

### 7.2 The Domestic Leg: How CIPS Settles Inside CNAPS

This is the mechanism that ties this section back to the rest of the guide, and it is now documented at primary level. The 2025 CIPS Business Rules establish:

- **CIPS accounts are zero-balance accounts (零余额账户).** The operating institution opens a CIPS account for each direct participant. The account **bears no interest, may not be overdrawn, and must have a zero balance at end of day (场终/日终余额为零).** Funds in the account belong to the direct participant, not to the operating institution. A direct participant may hold **only one** such account at CIPS. Indirect participants hold **no CIPS account.** ✅ (Article 8)
- **The settlement funds sit at the PBOC.** The operating institution may open a **clearing account at the People's Bank of China** to hold participants' CIPS settlement funds, kept in a segregated sub-ledger from the operator's own funds; that account may also not be overdrawn and must close at zero. Critically, the operator **may not open accounts at commercial banks to hold CIPS settlement funds and may not allow clearing funds to accumulate (不得形成清算资金沉淀)**. ✅ (Article 9)
- **Liquidity is managed through PBOC clearing accounts.** A direct participant manages its CIPS account liquidity **through the clearing account it — or its funds custodian bank — holds at the PBOC.** ✅ (Article 10)

Read together: **CIPS itself does not hold value overnight.** It is a real-time message-and-ledger layer over the participants' central-bank money. The actual settlement asset is the same PBOC settlement-account money that HVPS moves. That is why CIPS's operating calendar depends on the domestic system's (Section 2.4), and why a foreign bank's CIPS liquidity ultimately needs to be visible and fundable at the PBOC-account level.

### 7.3 Participant Types, Standards, Operating Hours

**Participant types.** The rules distinguish two, with an important refinement on the direct side ✅ (Articles 5, 7, 11, 12, 13):

- **Direct participant (直接参与者)** — holds a CIPS account and a **CIPS bank code (CIPS行号)** and conducts business directly through CIPS. The operator **must** open CIPS accounts for **bank-type** direct participants, and opens them for **financial-market-infrastructure-type** direct participants as business requires. Bank-type direct participants process payments per client instructions or per indirect participants' entrustment, **individually or in batches**; infrastructure-type direct participants act for trading, securities-settlement or CCP functions and **may not expand their service scope, objects or business scenarios without approval**. Bank-type direct participants must keep their systems logged in throughout CIPS processing hours; infrastructure-type ones must be logged in during overlapping business hours.
- **Indirect participant (间接参与者)** — has a **CIPS bank code** but **no CIPS account**, and entrusts a direct participant to process its business. **One indirect participant may establish business relationships with several direct participants** ✅ (rule text and ICBC's own service description agree). ICBC's published service terms illustrate the practical requirement: the client must open an RMB account with the direct participant for cross-border settlement and sign a CIPS participant entrustment agreement, after which clearing parameters are set and the arrangement becomes effective.
- **Funds custodian bank (资金托管行).** An **overseas institution** applying to become a direct participant **may entrust a qualifying direct participant as its funds custodian bank**, which handles the PBOC clearing account through which that direct participant manages CIPS liquidity. ✅ (Article 7, Article 10) This is the mechanism that lets an overseas direct participant exist without a PBOC relationship of its own.

Note the industry taxonomy that circulates alongside the rule: some sources (ICBC's, and Chinese explainers) split direct participants into **"class 1"** (holds accounts at both CNAPS/HVPS and CIPS, and uses HVPS to move the funds) and **"class 2"** (holds an account only at CIPS and completes settlement indirectly via a funds-custody arrangement). ⚠ **This two-class split is a market description, not the rule's own taxonomy** — the rule text speaks of bank-type and infrastructure-type direct participants, plus the funds custodian bank. Use the rule's categories; treat "class 1 / class 2" as trade shorthand.

**Message standard.** ⚠ **The rule does not mandate ISO 20022 by name.** Article 28 provides that **the operating institution formulates and publishes the CIPS message standard (报文标准)**, that the standards must implement anti-money-laundering requirements, and that direct participants must adapt their systems to the **latest published message standard**. Separately, and well-attested: CIPS **adopted ISO 20022-based messaging at launch**, with the SWIFT–CIPS MoU of March 2016 concerned partly with enabling Chinese-character content and richer data in that format. ✅ (ISO 20022 adoption, secondary — SWIFT press release and CIPS material) / ⚠ (the current precise alignment, version and any Chinese extensions to the standard)

**Operating hours.** CIPS supports a **hybrid settlement model**: **real-time gross settlement** for payments a direct participant initiates individually, and **timed net settlement (定时净额结算)** for payments it initiates in batches — with the operator able to adjust the netting sessions and times **with same-day effect**. ✅ (Article 17). On hours, CIPS's own current FAQ states that CIPS **operates 24 hours a day, five days a week, plus an additional four hours**, providing round-the-clock service on statutory working days, and supporting external service on domestic holidays and overseas working days, subject to published announcements; the operator must publish the CIPS business calendar and operating sequence. ✅ (CIPS FAQ; Article 15). This "5×24 + 4" schedule — an extension of the Phase 1 "5×12" day with daytime processing of 09:00–20:00 — is the reason the domestic HVPS window had to be extended first (Section 2.4).

**Service scope.** Per the same Article 17, CIPS supports RMB payments, **PvP (人民币对外币同步交收)**, **DvP (券款对付) settlement**, **CCP central clearing (中央对手集中清算)** and other cross-border RMB transaction settlement. ✅ **Note:** the same article, and Article 27, envisage **cross-border HKD and other foreign-currency payment business** through CIPS under separately issued operating guidelines — i.e. CIPS's mandate is no longer exclusively RMB. ✅

### 7.4 The DVP / Securities Question

The DVP question has a clean answer now, and it is a *yes* — defined in rule, not just in press:

- **DvP is an express supported settlement type** under Article 17 of the 2025 rules ✅ — and has been live since CIPS Phase 2, with implementation and Bond Connect Northbound support dated to 9 October 2018 ⚠ (secondary).
- **CCP central clearing is also an express supported type** ✅ (Article 17).
- **Infrastructure-type direct participants** are the channel: they organise or participate in fund settlement through CIPS for their trading, securities settlement or CCP functions, and may not broaden their mandate without approval ✅ (Article 11).

The practical caveat: CIPS settles the **cash leg** in central-bank RMB. The securities leg lives in the domestic CSDs — CCDC, CSDC and the Shanghai Clearing House — and the cross-border bond access channels (Bond Connect, and the various CIBM access routes) are the arrangements that connect foreign holders to them. CIPS is the money leg, not the securities depository. For the FMI taxonomy and the settlement-mechanics treatment, see [financial_infrastructure_guide.md](financial_infrastructure_guide.md).

### 7.5 Practical Corridor Routes for a Non-Chinese Bank

This is the question the rest of the guide has been building toward: **by what routes does a non-Chinese bank reach RMB clearing and CNAPS settlement?** Four routes are real and current. They are commonly described in Chinese industry material as 代理行模式、清算行模式、NRA账户模式, and 接入CIPS模式.

| Route | Chinese | Structure | Settlement locus | Cost / control |
|---|---|---|---|---|
| **1. Correspondent bank (agency) route** | 代理行模式 | The bank holds an RMB nostro account with a Chinese bank (or a foreign bank with China access) and instructs payments across it. The correspondent handles Chinese in-country clearing | The correspondent's HVPS/CNAPS connectivity | Lowest setup; the bank has no CNAPS relationship at all, and prices and cutoffs are the correspondent's |
| **2. RMB clearing bank route** | 清算行模式 | The bank uses an appointed **RMB clearing bank** in its jurisdiction — traditionally Bank of China (Hong Kong) and Bank of China (Macau) for those markets, per the CPMI record of them as HVPS direct participants and clearing agents — and the network of PBOC-designated RMB clearing banks in other financial centres | The clearing bank's CNAPS/HVPS access, plus CIPS where available | Deep local liquidity and a regulatorily recognised channel; the clearing bank is a single point of dependency |
| **3. RMB NRA route** | 人民币NRA账户模式 | A **Non-Resident Account (NRA, 境外机构人民币银行结算账户)** opened by an overseas institution with a bank in mainland China, used to settle RMB directly through the onshore clearing system | The onshore bank's CNAPS connectivity; funds sit in an onshore account | Direct onshore RMB account and access to domestic rails; the account is onshore and subject to onshore account rules |
| **4. CIPS participation** | 接入CIPS模式 | The bank becomes a **direct CIPS participant** (its own CIPS account and bank code, its own or a custodian's PBOC clearing account) or an **indirect CIPS participant** via a direct participant | CIPS message layer over PBOC central-bank money | Highest control and directness for cross-border RMB; highest cost and compliance burden |

**Verification status of each route:**

- **Route 1 (correspondent)** — ✅ real and current; it is the default and the only one requiring no Chinese membership of any kind.
- **Route 2 (clearing bank)** — ✅ real. Bank of China (Hong Kong) and Bank of China (Macau) were direct HVPS participants and its clearing agents in Hong Kong and Macau respectively, per the CPMI record ✅ (2012, via Wikipedia). The wider set of PBOC-designated RMB clearing banks evolved across major financial centres; the **current** designated list and each clearing bank's precise access mode should be confirmed with the PBOC and the bank concerned rather than taken from any guide. ⚠
- **Route 3 (NRA)** — ✅ real. NRA is the **RMB non-resident account (境外机构人民币银行结算账户)**, opened by an overseas institution with a domestic bank to settle RMB. The governing instrument is the PBOC's 《境外机构人民币银行结算账户管理办法》; note that sources differ on the document number and year (one secondary source cites 银发〔2010〕249号; another attributes a differently-numbered instrument), so **verify the current instrument** before citing one. ⚠ An NRA is an **onshore account** — the money sits inside the mainland system and is subject to onshore rules; that is both its advantage (direct domestic rail access) and its constraint.
- **Route 4 (CIPS)** — ✅ real and now defined at primary level. Direct participants need a CIPS bank code and a CIPS zero-balance account, and either their own PBOC clearing account or a **funds custodian bank**; indirect participants need a CIPS bank code, an RMB account with, and an entrustment agreement to, a direct participant, and may use more than one. ✅ (Articles 5, 7, 10; ICBC service terms)

**How they combine in practice.** A non-Chinese bank's typical path is layered rather than exclusive: start on Route 1 or 2 for corridor coverage; add Route 3 where it needs an onshore RMB account for corporate clients; graduate to Route 4 (first indirect, then possibly direct) when cross-border RMB volume and directness justify the cost. Note also that Routes 2 and 4 are not alternatives but complements: a clearing-bank relationship is often itself delivered as an indirect CIPS participation. The access-route decision is worked through with numbers in Section 11.2.

---

## 8. Technical Standards and Messaging

### 8.1 The PBOC Message Specifications

CNAPS participants exchange messages through **PBOC's own message specification (报文标准)** rather than a publicly documented open standard. What is verifiable:

- The specification exists, is versioned, and is distributed to participants; the CNCC publishes documentation and certification material for participants through its user platform (ccsp.cncc.cn — 参与机构认证材料模板, 文档下载). ✅ (existence and channel)
- The second generation's stated design principle was **separation of message transmission from business processing (报文传输和业务处理分离)**, which created a distinct participant-access and message-transmission layer alongside the business systems. ⚠ (Chinese technical/summary documentation)
- Business messages carry a **business type (业务种类)** designation. ✅ for CIPS (Article 14 of the 2025 rules); ⚠ as a generalisation to HVPS/BEPS.
- **This guide does not reproduce CNAPS message-type codes**, because the authoritative PBOC/CNCC message catalogue was not obtained in this pass (Section 4.5). Anyone who needs the codes must get them from the CNCC documentation or from a certified vendor — not from a guide.

**Cross-reference:** the vendor-side reality of message-exchange and CNAPS connector products — who builds them, what certification they carry, and how they are implemented — is the subject of [shanghai_huateng_software_guide.md](shanghai_huateng_software_guide.md) and [chinese_core_banking_vendors_guide.md](chinese_core_banking_vendors_guide.md). This section deliberately does not repeat that material.

### 8.2 The ISO 20022 Question

The honest answer is that this splits by system, and the distinction is routinely elided:

| System | ISO 20022 status | Marker |
|---|---|---|
| **CIPS** | **Adopted ISO 20022-based messaging at launch** (2015); the SWIFT–CIPS MoU of March 2016 covered enabling Chinese-character content and richer data in that format. The *rules*, however, say the **operating institution publishes the CIPS message standard** and participants must conform to the latest version — ISO 20022 is the practical foundation, not a rule-mandated name | ✅ adoption (secondary); ⚠ the exact current standard, version and Chinese extensions |
| **CNAPS domestic (HVPS / BEPS / IBPS)** | ❌ **Not verified.** No authoritative source reviewed in this pass states that the domestic CNAPS message specification is ISO 20022-based, aligned to it, or on a migration path to it. Vendor material that asserts "CNAPS uses ISO 20022" should be treated as **unverified** | ❌ |
| **PBOC statistics infrastructure** | The PBOC's payment statistics have used the 《支付业务统计指标》 financial-industry standard for disclosure since 2015 — an unrelated but relevant example of the PBOC standardising by *its own* industry standard rather than by an international one | ✅ (footnote of the 2024 payment report) |

**Why this matters practically.** If a bank's China payments proposition assumes ISO 20022 end-to-end because "CIPS uses ISO 20022," the assumption may hold for the CIPS leg and break on the domestic leg — where the mapping from PBOC message formats to ISO 20022 is the bank's or its vendor's problem. Treat the ISO 20022 question as a **per-hop** question: CIPS hop ✅ (ISO 20022-based, operator-published standard), domestic CNAPS hop ❌ (unverified). Note also that the global ISO 20022 migration programme — CBPR+, the November 2025 end of the MT/ISO 20022 coexistence window and the differently-timed Chinese, Indian, Japanese and US adoptions — is the comparative topic; it lives in [financial_infrastructure_guide.md](financial_infrastructure_guide.md), not here.

### 8.3 Connectivity and Network Arrangements

What a participant must implement, at the level that can be stated with confidence:

- **Direct access to a CCPC (or, for some participants, the NPC).** BEPS participants can connect directly through the NPC or through a CCPC; the same two-tier logic governs HVPS access. ⚠
- **A participant access / business-management subsystem**, particularly for **indirect (间联) access**, on the second-generation architecture. ⚠
- **Primary and backup systems.** CIPS rules require the operating institution and direct participants to **build CIPS-related backup systems and to conduct regular production-to-backup switchover drills (定期开展生产系统与备份系统的切换演练)** so that backup can be brought into service quickly after an incident, and to notify each other of failures. ✅ (Articles 25, 26). The same discipline is a general expectation of CNAPS direct participants.
- **Dedicated lines vs SWIFT.** Industry descriptions of CIPS access distinguish **dedicated line (专线)** for one class of direct participant from **SWIFT or dedicated line** for another — i.e. CIPS can be reached over closed SWIFT messaging as well as over a dedicated network. ⚠ (trade description; the rules require participants to maintain system login during processing hours, Article 13, but do not themselves name SWIFT as a channel in the text reviewed)
- **Message permissions.** Direct participants must **apply for message permissions (报文权限)** from the operator by business function; the net-settlement message permission additionally requires meeting the operator's settlement-risk management requirements. ✅ (Article 12)

### 8.4 Certification and Interfaces

- The CNCC maintains a **participant certification process with published material templates** (参与机构认证材料模板) on its user platform. ✅ (existence)
- CIPS rules require participants to **adapt their systems to the latest published CIPS message standard** — a standing change obligation, not a one-off integration. ✅ (Article 28)
- Operational interfaces a participant's systems must support include: message submission and receipt with business-type designation (Article 14); end-of-day **reconciliation data** — CIPS sends bank-type direct participants **payment business summary reconciliation information and fund adjustment reconciliation information**, and infrastructure-type direct participants **payment business reconciliation information and fund adjustment reconciliation information**, against which participants must reconcile their accounts. ✅ (Articles 20, 21). Participants may use their own local date as the customer value date (Article 20) — a small but real accounting-integration detail for a foreign bank.
- **Message codes, field mappings and certification test scripts are not reproduced here** — obtain them from the CNCC/CIPS operating institution or from a certified vendor. For the supplier landscape, see [chinese_core_banking_vendors_guide.md](chinese_core_banking_vendors_guide.md) and [shanghai_huateng_software_guide.md](shanghai_huateng_software_guide.md).

---

## 9. Scale, Volumes, International Comparison

### 9.1 Latest Published Figures

The authoritative source is the PBOC's annual **支付体系运行总体情况** (Payment System Operation Report), published in the first quarter following the reporting year. The 2024 report was published **15 February 2025** and is the latest complete year used here. ✅ (PBOC, primary; republished by CNCC)

**CNAPS core systems, 2024** (all figures year-on-year change in parentheses):

| System | Transactions | Value (RMB) | Daily average | Working days |
|---|---|---|---|---|
| **HVPS** 大额实时支付系统 | **392 million** (+2.52%) | **8,824.18 trillion** (+4.05%) | 1.5614 million / RMB 35.16 trillion | 251 |
| **BEPS** 小额批量支付系统 | **4.79 billion** (+3.82%) | **198.13 trillion** (+6.63%) | 13.0882 million / RMB 541.335 billion | ~366 |
| **IBPS** 网上支付跨行清算系统 | **16.651 billion** (−1.94%) | **290.24 trillion** (−3.56%) | 45.4932 million / RMB 793.016 billion | ~366 |
| CFXPS 境内外币支付系统 | 6.1327 million (+19.44%) | USD 4.82 trillion (≈ RMB 34.25 trillion) (+82.96%) | 24,400 / USD 1.9209 billion | 251 |

**Subtotal:** PBOC CNCC systems processed **21.839 billion transactions worth RMB 9,346.81 trillion** in 2024. ✅

**The wider domestic stack, 2024** — useful because it shows where the *count* lives:

| System / operator | Transactions | Value (RMB) | Marker |
|---|---|---|---|
| **NetsUnion (NUCC)** | **1.03 trillion** (+15.59%) | **520.53 trillion** (+4.54%) | ✅ (PBOC report, NUCC-sourced) |
| **China UnionPay interbank system** | **334.64 billion** (+7.97%) | **255.46 trillion** (−8.61%) | ✅ |
| Intra-bank systems (bank internal) | 23.343 billion (+8.11%) | 2,144.50 trillion (−1.12%) | ✅ |
| **CIPS** | **8.2169 million** (+24.25%) | **175.49 trillion** (+42.60%) | ✅ (PBOC report, CIPS-sourced) — 269 working days |
| City Bank Clearing (城银清算) | 51.977 million (+28.75%) | 5.21 trillion (+37.84%) | ✅ |
| Rural Credit Banks clearing (农信银) | 2.255 billion (−26.62%) | 2.61 trillion (−16.22%) | ✅ |
| **All payment systems combined** | **14.2 trillion** | **12,450.60 trillion** | ✅ |

**Comparisons that leap out:**

- **A single retail clearing house (NUCC) processed ~2,600× the transaction count of HVPS in 2024** (1.03 trillion vs 392 million), while HVPS processed **~17× the value** (RMB 8,824 trillion vs RMB 521 trillion). That ratio *is* the CNAPS architecture in one line: settled value concentrates in the RTGS layer, transaction count lives in the retail layer.
- **CIPS's value (RMB 175.49 trillion) is ~2% of HVPS's value and ~1.4% of CIPS transactions' worth relative to the domestic stack.** Cross-border RMB is large in absolute terms and small relative to the domestic system — a useful antidote to commentary that treats CIPS as rivalling the domestic infrastructure in scale.
- **CIPS in 2024 grew 42.60% by value** — an extraordinary rate, but from the base above.

### 9.2 Growth Trend

| Year | HVPS transactions | HVPS value (RMB tn) | BEPS value (RMB tn) | IBPS value (RMB tn) | Source |
|---|---|---|---|---|---|
| 2009 | 247 million | 760 | 11 | — | IMF 2012 (via Wikipedia) |
| 2023 | 382 million | 8,481 | 186 | 301 | PBOC 2023 report (via Wikipedia) |
| **2024** | **392 million** | **8,824.18** | **198.13** | **290.24** | **PBOC 2024 report (primary)** |

Two things to read carefully. **HVPS value grew ~11.6× between 2009 and 2024 while its transaction count grew ~1.6×** — the system's growth is a value phenomenon, driven by the expansion of China's economy, interbank markets and settlement volumes rather than by payment counts. And **IBPS value fell in 2024 (−3.56%) while BEPS grew (+6.63%)** — the first IBPS decline in the data reviewed, consistent with the migration of retail traffic toward wallet rails (NUCC) and away from bank-channel transfers. Treat the second as an observation, not an established trend; one year is not a trend. ⚠

### 9.3 Comparison Discipline: Why the Numbers Do Not Line Up

Three cautions before anyone builds a chart comparing CNAPS to Fedwire, T2 or CHAPS:

1. **Different reporting bases.** The PBOC publishes on a **calendar-year** basis, in **RMB**, with statistics governed by its own 《支付业务统计指标》 industry standard (since 2015), and reports systems by *number of actual operating days* (251 for HVPS, 269 for CIPS, 366 for the retail systems in 2024). European and US authorities publish on their own bases, in EUR/USD, often at different reporting frequencies and with different treatment of what counts as a "payment." **A direct volume-to-volume comparison is a category error unless the bases are reconciled first.**
2. **Different inclusion boundaries.** The PBOC's "payment systems" aggregate explicitly includes intra-bank systems, UnionPay, the city and rural clearing systems, CIPS and NUCC — a much wider perimeter than most central banks publish. Quoting the PBOC's RMB 12,450 trillion total against another country's RTGS turnover compares two different things.
3. **Who settles what.** HVPS carries not only its own RTGS payments but the **net settlement positions of the whole domestic stack**. Comparing HVPS with a foreign RTGS that settles only its own payments understates HVPS's role without any of the numbers being wrong.

The **comparative frame — how these systems sit against Fedwire, T2, CHAPS, MEPS+, CLS and the rest — belongs to [financial_infrastructure_guide.md](financial_infrastructure_guide.md)** and is not rebuilt here. What this guide owns is the *Chinese* numbers and the caveats on reading them.

---

## 10. Operational and Regulatory Reality

### 10.1 What Participation Requires

Consolidating Sections 3–8 into an obligation list:

| Obligation | Direct participant | Indirect participant |
|---|---|---|
| PBOC settlement account (清算账户) | **Yes** — required | No |
| Technical connection to CCPC/NPC | **Yes**, plus a certified participant access subsystem | No — connects to the direct participant |
| Liquidity funding of the settlement account | **Yes** — the binding constraint on payment capacity | No — funds via the agent's commercial terms |
| Automatic pledge financing eligibility | **Yes**, if eligible (HVPS direct participant + interbank settlement qualification) | **No** — cannot access it (Section 4.3) |
| Operating-hours discipline | Full window including the T-1 evening opening | Constrained by the agent's window and internal cutoffs |
| Backup systems and switchover drills | Expected, and required for CIPS direct participants (Articles 25–26) | Depends on the agent |
| Message-standard change management | Direct obligation to adopt the latest standard | Via the agent, on the agent's timetable |
| Reporting to the central bank | PBOC reporting and statistical obligations as a direct participant | Via the agent and its own licence, not the CNAPS channel |
| Membership/certification | CNCC participant certification (参与机构认证) | Onboarding with the agent plus any CIPS/clearing-system registration |

⚠ The **fee and membership economics** are not published in usable form (Section 4.4). The costs that matter — the price of direct participation, per-payment tariffs, and the terms of an agency arrangement — are commercial and bilateral. Do not build a business case on published tariffs, because there are none that are verified and current.

### 10.2 Liquidity and Funding Obligations

The reference cases for how demanding CNAPS liquidity management is come from the operators themselves. CNCC's 2018 notice moving HVPS to 5×21 hours warned participants directly that the change "places higher demands on system participants," requiring institutions **to manage account positions properly, handle overnight HVPS business, system maintenance and emergency management, and ensure timely overnight settlement (做好账户头寸管理…保证夜间资金及时清算)**. ✅ (primary — CNCC). That warning is the whole operating discipline in one sentence: a direct participant must staff and fund an overnight settlement operation.

Practical implications: intraday liquidity forecasting against the T-1 evening opening and the 17:30 day-end; monitoring the settlement-account position continuously during the window; queue management (and automatic pledge financing eligibility as the safety valve); and end-of-day reconciliation against the NPC's authoritative record (Section 5.3).

### 10.3 Contingency, BCP and Reporting

- **Backup systems and drills.** CIPS rules require the operating institution and direct participants to build backup systems and **regularly rehearse production-to-backup switchover**, so that backup can be activated quickly; both sides must notify each other of failures and cooperate to restore service. ✅ (Articles 25, 26). The same expectation applies to CNAPS direct participants, and a foreign bank should assume it must demonstrate rehearsal evidence to its clearing agent or the operator.
- **Incident information flow.** Participants and the operator collect, analyse and report system-abnormality and emergency information, and maintain an early-warning mechanism. ✅ (Article 24)
- **The CNCC runs 7×24 operational duty (7×24小时运行值守)** and maintains maintenance windows published in advance (维护窗口安排) on its website. ✅ (primary — CNCC notices). Planned maintenance windows are themselves a scheduling constraint on payment operations.
- **Reporting to the PBOC.** Direct participants are subject to PBOC reporting and statistical obligations; the PBOC's payment statistics are compiled from participants and operators (the 2024 report cites NUCC, UnionPay and CIPS as data sources). ✅ (methodology confirmed) ⚠ (the specific per-participant reporting templates were not reviewed).

### 10.4 Operational Risk Register

| Risk | Mechanism | Where it bites a foreign bank |
|---|---|---|
| **Settlement-account liquidity shortfall** | Account funded late; payments queue; clearing window pressure | Overnight staffing and funding; missed cutoffs chain into failed client SLAs |
| **Cutoff drift** | CNCC revises windows by notice; HVPS changes again on 19 October 2026 | Treasury and payment-processing cutoffs must track notices, not last year's documentation |
| **Indirect-access settlement risk** | Payment is final only when the agent settles; the indirect participant bears the agent's credit and liquidity risk | Cap exposure per agent; monitor agent queues; obtain settlement confirmations |
| **Central-node concentration** | The NPC is a single national settlement point | Continuity planning must treat NPC/CCPC disruption as a systemic scenario, not a bank-level one |
| **Message-standard change** | Operator publishes standards; participants must adapt their systems | Change-management cost and release cycles on the bank's own roadmap |
| **Operating-window mismatch** | CNAPS works Chinese working days and CIPS 5×24+4; a foreign bank's own cutoffs and holiday calendars differ | Cross-border payments needing a same-day domestic leg must respect the domestic calendar, not the bank's |
| **Statutory-holiday arrangements** | Special working-day (特殊工作日) arrangements apply on weekends/holidays | Holiday liquidity and staffing plans must follow the PBOC calendar |
| **Regulatory/geopolitical exposure** | Cross-border RMB channels are policy instruments | Corridor design should assume access terms can change |

For the condensed general treatment of operational resilience in payment infrastructure, see [financial_infrastructure_guide.md](financial_infrastructure_guide.md); for the bank-side operating model, see [operating-model guides in this repository] and [payments_hub_guide.md](payments_hub_guide.md).

---

## 11. Cymbal Bank Worked Example

> **This section is explicitly illustrative and fictional.** Cymbal Bank is a persona, not a real institution. Every number below is a *made-up planning figure* chosen to show the shape of the analysis — none is a market rate, a published tariff, or a verified cost. Where a real fact is being used (cutoffs, windows, limits), it is flagged as such via the guide's markers. Do not cite the illustrative figures as data.

### 11.1 Scenario and Scope

**Cymbal Bank** is a mid-size commercial bank headquartered in Singapore with corporate and institutional clients, and a live proposition for **China-corridor payments in RMB**. Its clients need two flow types:

1. **Trade settlement** — RMB payments to and from mainland suppliers and buyers, typically RMB 200,000 to RMB 50 million per payment, with documentary and trade-finance context.
2. **Treasury and liquidity flows** — RMB sweeps, entity-to-entity funding between Chinese subsidiaries and the regional treasury centre, and FX-leg settlement after conversions.

Planning parameters (**illustrative**): 1,800 RMB payments/month; average RMB 4.2 million; peak single payment design point RMB 250 million; 70% of value in trade settlement, 30% treasury; ~15% of payments requiring next-day value, the rest T+1 or later.

### 11.2 Access-Route Decision

Cymbal Bank's options map directly onto Section 7.5. Applying them:

| Criterion (illustrative weights) | Route 1: Correspondent | Route 2: RMB clearing bank | Route 3: RMB NRA | Route 4: CIPS indirect |
|---|---|---|---|---|
| Time to live | 2–3 months | 4–6 months | 4–6 months | 6–9 months |
| Annual fixed cost (illustrative, indexed) | 1.0× | 1.6× | 2.1× | 2.4× |
| Domestic-leg cutoffs controllable by Cymbal Bank | No | Partly | Partly | Partly (via agent) |
| Same-day RMB value for clients | Limited | Common | Common | Common |
| Onshore RMB account for client flows | No | Indirect | **Yes (NRA)** | No |
| Client-facing pricing flexibility | Low | Medium | Medium-High | High |
| Exit cost / lock-in | Low | Medium | Medium | Medium-High |
| Direct settlement finality visibility | None | None | None | None (agent-mediated) |

**Decision path.** Cymbal Bank runs **Route 1 for initial corridor coverage**, adds **Route 3 (NRA)** to give corporate clients an onshore RMB account for collections and disbursements, and takes **Route 4 as an indirect CIPS participant with a large Chinese direct participant as agent** for the cross-border leg. **Route 2 (a designated RMB clearing bank in Singapore)** is evaluated in parallel as a liquidity backstop and as the fallback if the CIPS relationship degrades. **Direct CIPS participation is explicitly deferred**: at the illustrative volume profile, the cost structure does not justify it, and the bank has no PBOC settlement account — direct participation would require either one (not available to it) or a **funds custodian bank** arrangement (available, but adds a dependency that removes much of the directness advantage). This is the honest conclusion: **direct participation is not the answer for this bank at this volume.**

### 11.3 Account and Liquidity Structure

- **Cross-border leg.** RMB nostro/liquidity account with the **CIPS direct participant (agent)**, plus a signed CIPS participant entrustment agreement; the agent holds the PBOC-side liquidity. Cymbal Bank must fund the agent's account ahead of the value window.
- **Onshore leg.** An **NRA (境外机构人民币银行结算账户)** with an onshore bank for client collections and disbursements inside China.
- **Treasury.** A standing RMB liquidity buffer sized to cover **the worst-case daily net outflow plus a stress margin** (illustrative: 1.5 days of peak net outflow), held in the agent account and in the NRA.
- **Value-dating discipline.** Client value dates must be mapped to the **CNAPS/HVPS window (T-1 20:30 → T 17:30, changing to T-1 19:00 from 19 October 2026) and the CIPS 5×24+4 schedule** — not to Singapore cutoffs.

### 11.4 Operating-Window and Cutoff Constraints

This is where the analysis earns its keep. Working backwards from the client promise:

| Constraint (verified fact) | Consequence for Cymbal Bank's cutoffs (illustrative) |
|---|---|
| HVPS day-end 17:30 Beijing (UTC+8); acceptance from 20:30 T-1, moving to 19:00 T-1 from 19 Oct 2026 | Internal release cutoff for same-day RMB must be set well before the domestic day-end, with buffer for agent processing |
| HVPS operates statutory working days, with special working-day arrangements on holidays | Holiday calendars must follow the PBOC, not Singapore; a Singapore business day is not automatically an RMB settlement day |
| BEPS settles in HVPS hours despite 7×24 acceptance | Low-value client payments submitted overnight settle later — SLAs must not promise otherwise |
| CIPS runs 5×24+4 over its published calendar | Cross-border leg is available far wider than the domestic leg; the *binding* constraint is the domestic one |
| No direct settlement-account visibility (indirect access) | Cutoffs must be set from the agent's confirmations, not from the system of record |

**Cymbal Bank's illustrative internal cutoffs:** client cut-off 11:00 SGT for same-day RMB value; agent cut-off 14:00 SGT; exception handling until 15:30 SGT; overnight window staffed for the T-1 leg once volumes justify it. Each of these is a *derived* figure, not a published one.

### 11.5 Integration, Messaging and Contingency

- **Messaging.** Cymbal Bank maps its internal ISO 20022 payment messages to the **CIPS message standard** (operator-published, ISO 20022-based) for the cross-border leg, and relies on the **NRA bank's** channel mapping for the onshore leg. The **domestic CNAPS hop's message format is not ISO 20022-verified** (Section 8.2), so the mapping risk sits with the onshore bank and the agent — and must be **contractually allocated**, not assumed away.
- **Reconciliation.** Client-facing reconciliation is built from the agent's and the NRA bank's confirmations, because Cymbal Bank has no direct feed from the system of record. Daily position reconciliation and a monthly attestation against the agent's statements.
- **Contingency.** Dual agents (primary CIPS direct participant plus the clearing-bank fallback), a documented manual fallback for high-value trade payments, and a defined escalation path with both providers. Cymbal Bank also assumes it must provide evidence of its own backup arrangement and test results to its providers.

### 11.6 Illustrative Cost Comparison

All figures **illustrative, indexed and fictional**. They show the *shape* of the trade-off, not market prices.

| Cost element (illustrative) | Route 1 | Route 1+3 | Route 1+3+4 |
|---|---|---|---|
| Setup / integration, one-off | $180k | $420k | $900k |
| Annual fixed (connectivity, memberships, assurance) | $120k | $260k | $560k |
| Per-payment all-in (incl. agent and onshore fees) | $14 | $11 | $7 |
| Annual variable at 21,600 payments/yr | $302k | $238k | $151k |
| **Illustrative annual total** | **~$422k** | **~$498k** | **~$711k** |
| Indicative break-even vs Route 1 | — | ~24 months | ~41 months at current volume |
| Client capability unlocked | Correspondent basics | Onshore RMB account | Full cross-border RMB control |

The point of the table is not the numbers. It is that **each additional layer of directness costs more up front and only pays back at volume**, and that the "cheapest" route is genuinely the cheapest at Cymbal Bank's current scale. A bank with 10× the volume reaches the opposite conclusion.

### 11.7 Risk Assessment and Open Items

**Risks carried (with the mitigation design):** agent credit and liquidity risk (cap exposure, dual agents, monitor agent queue behaviour); cutoff drift (a standing process to track CNCC notices and the 19 October 2026 change); holiday-calendar mismatch (PBOC calendar as the system of record for value dates); message-mapping failure (contractual allocation to the onshore bank/agent, plus end-to-end testing before go-live); reconciliation gap from indirect access (daily agent confirmations).

**What Cymbal Bank would still need to establish before committing — the honest list:**

1. **Contracted, current terms** from each candidate provider: pricing, cutoffs, SLA, failure-recovery commitments and the message standards each supports. Nothing published substitutes for this.
2. **The current PBOC rule texts** for the specific business it intends to transact — NRA account rules, any applicable cross-border RMB rules, and the current BEPS/IBPS limits if retail routing is involved. Several rule details in this guide are flagged ⚠ precisely because they are not verifiable from public sources.
3. **Confirmation of the current HVPS schedule** — including the 19 October 2026 change and any subsequent notice — before finalising cutoffs.
4. **The CIPS operating calendar and message standard version** in force, from the operating institution or the agent.
5. **Its own liquidity stress numbers** — settlement timing, peak outflow, and the cost of the buffer — modelled from its real payment profile, not the illustrative parameters above.
6. **Legal and regulatory sign-off** on the onshore account structure and cross-border RMB regulatory reporting.

---

## 12. Claims Audit

| # | Claim | Verdict | Source | Source quality | Date of source |
|---|---|---|---|---|---|
| 1 | CNCC established May 1990; operates CNAPS under the PBOC | ✅ | Wikipedia citing ECB 2010; consistent across sources | Secondary | 2010 / 2024 |
| 2 | HVPS is the RTGS component; BEPS the bulk retail component | ✅ | CPMI Red Book (2012); PBOC material | Strong | 2012 |
| 3 | HVPS first-generation production October 2002; nationwide completion dated 1 Dec 2003 in one source | ⚠ | Chinese encyclopaedia/technical sources | Secondary | undated |
| 4 | "China's domestic RTGS launched June 2005" | ⚠ | PBOC 2007 report as summarised by Wikipedia | Strong source, secondary transmission | 2007 |
| 5 | BEPS launched June 2006 | ✅ | PBOC 2007 report via Wikipedia | Strong source, secondary transmission | 2007 |
| 6 | IBPS in production 30 August 2010; nationwide 24 January 2011 | ✅ | **CNCC official page** | Strong (operator primary) | CNCC page 2016 |
| 7 | CNAPS second generation fully in production 8 October 2013 | ⚠ conflicting (6 Oct also cited) | Chinese encyclopaedia sources | Secondary | — |
| 8 | HVPS moved to 5×21 hours from 22 January 2018; acceptance T-1 23:30, cutoff 17:15, clearing window to 20:30 | ✅ | **CNCC notice** | Strong (operator primary) | 2018-01-19 |
| 9 | 2018 extension was explicitly tied to supporting CIPS operating-time needs | ✅ | **CNCC notice** | Strong (operator primary) | 2018-01-19 |
| 10 | Current HVPS schedule: "5×21+12 hours", acceptance T-1 20:30, day-end T 17:30; special working-day arrangements on holidays | ✅ | **CNCC notice** | Strong (operator primary) | 2026-07-28 |
| 11 | HVPS extended to "5×22.5+13.5 hours" from 17 Oct 2026; acceptance moves to T-1 19:00 from 19 Oct 2026 | ✅ | **CNCC notice** | Strong (operator primary) | 2026-07-28 |
| 12 | The intermediate step from the 2018 regime to the current T-1 20:30 / 17:30 regime | ❌ | Not located | — | — |
| 13 | Two-tier NPC/CCPC architecture; 32 city processing centres | ✅ architecture / ⚠ count | CPMI 2012; secondary guides | Strong / Secondary | 2012 / 2026 |
| 14 | BEPS runs 7×24 with working day 16:00 T-1 → 16:00 T and settlement in HVPS hours | ⚠ | Chinese product/technical writing | Secondary | 2022 |
| 15 | BEPS uses real-time bilateral netting with a net debit cap; netted payments are irrevocable | ⚠ | Chinese technical/press reproduction of PBOC rules | Secondary | 2022 |
| 16 | BEPS credit-payment limit RMB 50,000 per transaction | ⚠ (documented historically; current limit unverified) | Chinese encyclopaedia and technical sources | Secondary | 2023 |
| 17 | IBPS runs 7×24; working day is the calendar day; settlement in the settlement-account system's hours | ✅ | PBOC office notice promulgating IBPS measures, as republished | Strong (PBOC rule text) | — |
| 18 | IBPS RMB 50,000 per-transaction limit | ⚠ (unverified current) | Chinese encyclopaedia | Secondary | — |
| 19 | NUCC incorporated 29 August 2017; NUCC mandate for third-party network payments | ✅ mandate / ⚠ incorporation + no capital figure | PBOC department notice via press; encyclopaedic | Secondary | 2017 / 2019 |
| 20 | From 30 June 2018 all network payment business involving bank accounts to be processed through NUCC; preparation deadline 15 Oct 2017 | ✅ | PBOC Payment & Settlement Dept notice, reported consistently | Secondary (consistent, multiple) | 2017 |
| 21 | 99% of cross-institution business migrated to NUCC by January 2019 | ✅ | NUCC disclosure via People's Daily finance | Secondary | 2019-01 |
| 22 | CIPS Phase 1 launched 8 October 2015 with 19 direct / 176 indirect participants | ✅ | CIPS material via Wikipedia | Secondary | 2024 |
| 23 | CIPS Phase 2 pilot 26 March 2018; full operation 2 May 2018; DVP + Bond Connect Northbound 9 Oct 2018 | ⚠ | Wikipedia from CIPS material | Secondary | 2024 |
| 24 | CIPS operating hours 5×24+4; Phase 1 was 5×12 with 09:00–20:00 daytime processing | ✅ current hours (CIPS FAQ) / ⚠ Phase 1 detail | cips.com.cn FAQ; encyclopaedic | Strong / Secondary | 2026 / — |
| 25 | CIPS adopted ISO 20022-based messaging at launch; SWIFT–CIPS MoU March 2016 | ✅ | SWIFT press release; CIPS material | Secondary | 2016 |
| 26 | CIPS accounts are zero-balance, non-interest-bearing, non-overdrawable; operator's PBOC clearing account; no depositing at commercial banks | ✅ | **PBOC, CIPS Business Rules, Articles 8–10** | Strong (primary rule text) | 银发〔2025〕248号, 2025-12-19 |
| 27 | CIPS hybrid settlement: RTGS for single payments, timed net settlement for batches; two queues (urgent/normal) with priorities | ✅ | **PBOC, CIPS Business Rules, Articles 17–18** | Strong (primary) | 2025-12-19 |
| 28 | CIPS supports PvP, DvP, CCP central clearing; and non-RMB (e.g. HKD) business | ✅ | **PBOC, CIPS Business Rules, Articles 17, 27** | Strong (primary) | 2025-12-19 |
| 29 | Overseas direct participants may appoint a funds custodian bank (资金托管行) | ✅ | **PBOC, CIPS Business Rules, Articles 7, 10** | Strong (primary) | 2025-12-19 |
| 30 | CIPS participants: 187 direct / 1,559 indirect at end-Oct 2025; 210 / 1,619 at end-June 2026 | ✅ | CIPS participant announcements #114, #118 | Strong (operator) / Secondary transmission | 2025-11 / 2026-07 |
| 31 | Automatic pledge financing: 2006-05-08 launch; 2017 Measures effective 2018-01-29; limits 4%/10%/15%; SLF-rate pricing; ≤90% haircut; 3pp overdue surcharge | ⚠ (rulebook and dates credible; reproduced via encyclopaedic source citing PBOC/gov.cn) | Baidu Baike citing PBOC/gov.cn | Secondary (dense, internally consistent, cited) | 2017-12 / 2018-01 |
| 32 | HVPS carries an amount-threshold concept (above-threshold credit transfers + urgent below-threshold) | ⚠ (rule text via secondary; current threshold unknown) | Chinese reproduction of 大额支付系统业务处理办法 | Secondary | — |
| 33 | HVPS/BEPS/IBPS fee schedule | ❌ not obtained | — | — | — |
| 34 | CNAPS domestic message specification is ISO 20022-based | ❌ not verified | — | — | — |
| 35 | 2024 volumes: HVPS 392m tx / RMB 8,824.18tn; BEPS 4.79bn / 198.13tn; IBPS 16.651bn / 290.24tn; CIPS 8.2169m / 175.49tn; NUCC 1.03tn / 520.53tn; all systems 14.2tn / 12,450.60tn | ✅ | **PBOC 2024 payment system report** | Strong (primary) | published 2025-02-15 |
| 36 | HVPS ran 251 working days, CIPS 269, other systems 366 in 2024 | ✅ | **PBOC 2024 report, footnote** | Strong (primary) | 2025-02-15 |
| 37 | 2023 comparison figures (HVPS 382m / 8,481tn; BEPS 4.6bn / 186tn; IBPS 17bn / 301tn) | ✅ as 2023 | PBOC 2023 report via Wikipedia | Strong source, secondary transmission | 2024 |

---

## 13. What Could Not Be Verified

1. **The current HVPS amount threshold (金额起点)** and whether the concept remains operative. The rule text referencing it was available only via secondary reproduction.
2. **BEPS and IBPS current per-transaction limits.** The RMB 50,000 figure is widely cited and historically grounded but was not confirmed as current.
3. **The current CNAPS system-level fee schedule** — per-transaction tariffs by business type, and exemptions. Not available publicly in usable form.
4. **CNAPS HVPS/BEPS/IBPS message-type codes, field layouts and business-type code lists.** Deliberately not asserted; obtain from the CNCC specification or a certified vendor.
5. **Whether the domestic CNAPS message specification is ISO 20022-based or aligned.** No authoritative source found.
6. **Whether HVPS offers a multilateral liquidity-saving mechanism or bilateral offsetting for ordinary interbank payments.** The netting mechanisms found apply principally to specially-licensed/clearing participants.
7. **The failure-handling and loss-sharing arrangements for BEPS** between netting admission and settlement of the net position.
8. **The current CCPC count** (the "32" figure is historical and possibly outdated).
9. **The completeness of the current participant taxonomy** (direct / indirect / specially-licensed) against the current PBOC participation rulebook.
10. **The intermediate HVPS operating-window change** between the January 2018 regime and the July 2026 regime.
11. **The exact PBOC instrument number and year for the NRA rules** (《境外机构人民币银行结算账户管理办法》); sources disagree.
12. **The current PBOC-designated RMB clearing bank list** and each bank's exact CNAPS/CIPS access mode.
13. **Automatic pledge financing's current parameters** — the limits, haircut schedule and eligibility were reproduced from an encyclopaedic source citing PBOC/gov.cn material, not from the rule text itself.
14. **The English acronym for the pre-CNAPS national electronic interbank arrangement** (电子联行系统).
15. **2025 and partial-2026 PBOC payment statistics** — the 2024 report is the latest complete year obtained; figures circulating for 2025 (including one source giving CIPS 2025 full-year data that appears inconsistent with a part-year basis) were not usable without the underlying report.

Note on accessibility: several PBOC and CNCC Chinese-language primary pages were unreachable from the research host during this pass, and Chinese encyclopaedia pages failed to render. Where that happened, the guide cites the best available source and marks its quality rather than filling the gap with a plausible figure.

---

## 14. Glossary

**Chinese–English term pairs.** All pairings below were checked against the sources cited in this guide rather than transliterated.

| Chinese | English | Notes |
|---|---|---|
| 中国现代化支付系统 | China National Advanced Payment System (CNAPS) | The umbrella; the acronym derives from the English rendering, not the Chinese |
| 中国人民银行 | People's Bank of China (PBOC) | Owner and regulator |
| 中国人民银行清算总中心 | China National Clearing Center (CNCC) | Operator of CNAPS |
| 大额支付系统 / 大额实时支付系统 | High Value Payment System (HVPS) | RTGS |
| 小额批量支付系统 / 小额支付系统 | Bulk Electronic Payment System (BEPS) | Deferred net settlement, batch retail |
| 网上支付跨行清算系统 | Internet Banking Payment System (IBPS) | Also colloquially 超级网银 ("super online banking") |
| 境内外币支付系统 | China Foreign Exchange Payment System (CFXPS) | Formerly 中国境内外币支付系统 → CDFCPS/FCPS; domestic FX-currency RTGS |
| 全国支票影像交换系统 | Cheque Imaging System (CICS/CIS) | Separate system; not a CNAPS component |
| 电子商业汇票系统 | Electronic Commercial Draft System (ECDS) | Separate; operated in the Shanghai Commercial Paper Exchange orbit |
| 人民币跨境支付系统 | Cross-Border Interbank Payment System (CIPS) | Separate operator in Shanghai |
| 跨境银行间支付清算有限责任公司 | Cross-border Interbank Payment Clearing (Shanghai) Co., Ltd. | The CIPS operating institution (运营机构) |
| 网联清算有限公司 / 网联 | NetsUnion Clearing Corporation (NUCC) | Mandated third-party network-payment clearing house |
| 非银行支付机构网络支付清算平台 | Non-bank Payment Institution Network Payment Clearing Platform | NUCC's platform |
| 国家处理中心 | National Processing Center (NPC) | Central node |
| 城市处理中心 | City Clearing Processing Center (CCPC) | Regional nodes |
| 清算账户 | Settlement account | PBOC account of a direct participant |
| 清算账户管理系统 (SAPS) | Settlement Account Management System | Holds the settlement accounts; supports RTGS/bilateral/multilateral settlement |
| 直接参与者 | Direct participant | Holds a settlement account and direct access |
| 间接参与者 | Indirect participant | Accesses via a direct participant |
| 特许参与者 | Specially-licensed participant | PBOC-approved non-bank entity for specified business |
| 资金托管行 | Funds custodian bank | CIPS: overseas direct participant's PBOC-account custodian |
| 净借记限额 | Net debit cap | BEPS risk control |
| 轧差 | Netting | Bilateral (双边) or multilateral (多边) |
| 清算窗口 | Clearing window | Period for participants to raise funds before day-end |
| 自动质押融资 | Automatic pledge financing | PBOC collateralised intraday funding for HVPS direct participants |
| 日切 | Daily cut-off | The point at which the session's books are struck |
| 特殊工作日 | Special working day | Weekend/holiday HVPS operating arrangement |
| 报文标准 / 报文规范 | Message standard / specification | PBOC's own specification; the CIPS standard is operator-published |
| 业务种类 | Business type | Field designation carried in CIPS messages |
| 代理行模式 | Correspondent (agency) bank model | Cross-border RMB route 1 |
| 清算行模式 | Clearing bank model | Cross-border RMB route 2 |
| 境外机构人民币银行结算账户 / NRA | RMB Non-Resident Account (NRA) | Cross-border RMB route 3 |
| 断直连 | "Disconnecting direct connections" | The 2017–18 migration of third-party payments to NUCC |
| 零余额账户 | Zero-balance account | CIPS direct-participant account |
| 中央对手集中清算 | CCP central clearing | A CIPS-supported settlement type |
| 券款对付 (DvP) | Delivery versus payment | A CIPS-supported settlement type |
| 同步交收 (PvP) | Payment versus payment | A CIPS-supported settlement type |
| 定期净额结算 | Timed net settlement | CIPS batch settlement mechanism |

**English–Chinese operational terms used without a single canonical Chinese pairing in the sources reviewed:** gridlock, liquidity-saving mechanism, loss-sharing arrangement, straight-through processing. These are described in English in the body rather than given an unverified Chinese pairing.

---

## 15. Cross-References and Further Reading

**Sibling guides in this repository:**

- [financial_infrastructure_guide.md](financial_infrastructure_guide.md) — the umbrella guide. **Owns the international comparative frame** (Fedwire, T2, CHAPS, MEPS+, and the CPMI-IOSCO PFMI taxonomy), the general treatment of settlement finality, FMI risk management, and the ISO 20022 migration programme. This guide defers to it throughout.
- [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) — the Chinese core-banking landscape. Referenced for the bank-side systems that sit above the clearing layer; not re-derived here.
- [chinese_core_banking_vendors_guide.md](chinese_core_banking_vendors_guide.md) — **owns vendors and connector certification.** The supplier-side reality of CNAPS connectivity belongs there (Section 8 defers to it).
- [shanghai_huateng_software_guide.md](shanghai_huateng_software_guide.md) — the repository's 支付清算 / message-exchange vendor deep-dive.
- [payment_rails_guide.md](payment_rails_guide.md) — rail taxonomy and architecture, including instant-payment rails.
- [payments_hub_guide.md](payments_hub_guide.md) — payment-hub architecture for a bank orchestrating multiple rails.
- [singapore_fintech_payments_guide.md](singapore_fintech_payments_guide.md) — the Singapore market, relevant to the Cymbal Bank persona's home jurisdiction.

**Primary and authoritative external sources:**

- PBOC — 支付体系运行总体情况 (annual payment-system operation report). The 2024 edition (published 2025-02-15) is the volume source for Section 9. Available via pbc.gov.cn (支付结算司) and republished by cncc.cn under 业务服务 → 统计数据.
- PBOC — 《人民币跨境支付系统业务规则》, 银发〔2025〕248号, dated 2025-12-19, effective 2026-02-01. Published in 国务院公报 2026年第5号. **The single most useful primary document for the cross-border sections of this guide.**
- CNCC (cncc.cn) — 信息公告 for operating-time notices and maintenance windows; 统计数据 for the reports; 支付系统 pages for system descriptions; and the user platform (ccsp.cncc.cn) for participant certification material.
- CIPS operating institution (cips.com.cn) — participant announcements (参与者公告, numbered by period), operating-hours FAQ, and product pages (including the CIPS standard transceiver).
- BIS Committee on Payments and Market Infrastructures — *Payment, clearing and settlement systems in China* (Red Book country chapter, 2012), bis.org/cpmi/publ/d105_cn.pdf. **Note:** this PDF returned a 502 from the research host during this pass; the content is used here via its citation in secondary sources and should be retrieved directly for authoritative detail.
- IMF — *People's Republic of China: Detailed Assessment Report: CPSS Core Principles for Systemically Important Payment Systems* (April 2012); and *FSAP Technical Note: Systemic Oversight of Financial Market Infrastructures* (June 2018).
- World Bank — Fast Payments Toolkit case study: China (IBPS), 2021.
- SWIFT — press releases on the CIPS MoU (March 2016) and CIPS membership data in SWIFTRef (September 2017).

---

## 16. Closing Summary

CNAPS is best understood as one thing, not three: **a single national settlement layer in central-bank money, with three applications serving three payment segments.** HVPS settles individually and irrevocably and is the final settlement point for the entire domestic stack; BEPS batches, nets bilaterally against a net debit cap, and discharges net positions through HVPS in HVPS hours; IBPS clears retail transfers in real time, 24×7, and defers settlement into the same accounts. Everything else in the Chinese picture — the card network, the third-party clearing house, the cheque-image and commercial-paper systems, the cross-border gateway — produces obligations that resolve into HVPS.

For a foreign bank, the conclusions that survive the verification work in this guide are narrow and durable: **the binding constraints are the operating calendar and the settlement-account liquidity at the centre, not the messaging.** CIPS gives cross-border RMB a modern, ISO 20022-based, near-continuously-available message layer, and its accounts are zero-balance by rule: the value always ends up as PBOC central-bank money, which is why the domestic HVPS window had to be extended before CIPS could cover the world's time zones. Access is a ladder — correspondent, clearing bank, NRA, CIPS participation — and the correct rung is a function of volume, not of ambition. Direct participation buys control, automatic pledge financing eligibility and settlement finality visibility, and it costs an overnight liquidity operation that most banks outside China should not build.

The route to RMB settlement always ends in the same place: central-bank money, at a day-end that is not your day-end, through a window that opens the evening before. That path — from a client's payment instruction, through clearing, into PBOC settlement and back out — is the clearing corridor.
