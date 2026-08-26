# Core Banking Systems Used by Chinese Banks: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Core Banking / Banking Architecture — the Chinese banking sector, the mainframe-to-distributed history, 数据大集中, domestic vendors (Sunline, DCITS, GienTech, Yusys), the 自主研发 self-development phenomenon, 去IOE and distributed cores, digital banks (WeBank, MYbank), 信创 (xinchuang) and the domestic technology stack, China vs. West, Singapore/Asia relevance
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [The Chinese Banking Sector: Tiers, Scale, and Constraints](#1-the-chinese-banking-sector-tiers-scale-and-constraints)
   - 1.1 [The Bank Tiers](#11-the-bank-tiers)
   - 1.2 [Scale: The World's Largest Banking Workloads](#12-scale-the-worlds-largest-banking-workloads)
   - 1.3 [Peak-Season Loads: Spring Festival and 双11](#13-peak-season-loads-spring-festival-and-双11)
   - 1.4 [Core System Requirements at Chinese Scale](#14-core-system-requirements-at-chinese-scale)
   - 1.5 [The Regulatory Environment: PBOC, NFRA, and the Data Laws](#15-the-regulatory-environment-pboc-nfra-and-the-data-laws)
2. [The Historical Core Landscape](#2-the-historical-core-landscape)
   - 2.1 [Era 1: The Mainframe Era (1980s–2000s)](#21-era-1-the-mainframe-era-1980s2000s)
   - 2.2 [The Data Concentration Wave: 数据大集中 (1990s–2000s)](#22-the-data-concentration-wave-数据大集中-1990s2000s)
   - 2.3 [Era 2: The Client-Server Era](#23-era-2-the-client-server-era)
   - 2.4 [Era 3: The Distributed Era (2010s–Present)](#24-era-3-the-distributed-era-2010spresent)
   - 2.5 [International Vendors in China: IBM, Oracle, SAP, Temenos](#25-international-vendors-in-china-ibm-oracle-sap-temenos)
   - 2.6 [The Reality: Limited International Core Penetration](#26-the-reality-limited-international-core-penetration)
3. [The Domestic Vendor Landscape](#3-the-domestic-vendor-landscape)
   - 3.1 [The Vendor Map at a Glance](#31-the-vendor-map-at-a-glance)
   - 3.2 [Sunline / Changliang Technology (长亮科技)](#32-sunline--changliang-technology-长亮科技)
   - 3.3 [DCITS / Digital China Information Service (神州信息, 神州数码融信)](#33-dcits--digital-china-information-service-神州信息-神州数码融信)
   - 3.4 [GienTech (中电金信)](#34-gientech-中电金信)
   - 3.5 [Yusys Technologies (宇信科技)](#35-yusys-technologies-宇信科技)
   - 3.6 [Teamax (天阳科技) and the Mid-Tier IT Vendors](#36-teamax-天阳科技-and-the-mid-tier-it-vendors)
   - 3.7 [Huawei and ZTE: The Platform Giants](#37-huawei-and-zte-the-platform-giants)
   - 3.8 [The Securities-Channel Vendors: Hundsun, Kelan, ApexSoft](#38-the-securities-channel-vendors-hundsun-kelan-apexsoft)
   - 3.9 [Domestic Vendor Characteristics](#39-domestic-vendor-characteristics)
4. [The Self-Development Phenomenon: 自主研发](#4-the-self-development-phenomenon-自主研发)
   - 4.1 [The Big-Bank Pattern](#41-the-big-bank-pattern)
   - 4.2 [ICBC: NOVA, FOVA, and the Distributed Future](#42-icbc-nova-fova-and-the-distributed-future)
   - 4.3 [CCB, ABC, BOC: The New-Generation Cores](#43-ccb-abc-boc-the-new-generation-cores)
   - 4.4 [CMB: The Self-Reliant Pioneer](#44-cmb-the-self-reliant-pioneer)
   - 4.5 [Why Self-Develop? The Four Drivers](#45-why-self-develop-the-four-drivers)
   - 4.6 [The Self-Development Pattern](#46-the-self-development-pattern)
   - 4.7 [Self-Developed vs. Vendor: The Trade-Offs](#47-self-developed-vs-vendor-the-trade-offs)
5. [去IOE and the Distributed Core Transformation](#5-去ioe-and-the-distributed-core-transformation)
   - 5.1 [The De-IOE Movement: Origins and Meaning](#51-the-de-ioe-movement-origins-and-meaning)
   - 5.2 [Banking 去IOE: From Alibaba to the Big Four](#52-banking-去ioe-from-alibaba-to-the-big-four)
   - 5.3 [Distributed Core Architecture Patterns](#53-distributed-core-architecture-patterns)
   - 5.4 [The Flagship Transformation Projects](#54-the-flagship-transformation-projects)
   - 5.5 [Timelines and the State of Play by 2026](#55-timelines-and-the-state-of-play-by-2026)
6. [The Digital Banks' Cores](#6-the-digital-banks-cores)
   - 6.1 [WeBank (微众银行): The Cloud-Native Showcase](#61-webank-微众银行-the-cloud-native-showcase)
   - 6.2 [MYbank (网商银行): The 310 Model at Scale](#62-mybanks-网商银行-the-310-model-at-scale)
   - 6.3 [aiBank (百信银行) and XWBank (新网银行)](#63-aibank-百信银行-and-xwbank-新网银行)
   - 6.4 [The Digital Bank Core Characteristics](#64-the-digital-bank-core-characteristics)
   - 6.5 [Yillion and Suning: The Second Wave of Digital Banks](#65-yillion-and-suning-the-second-wave-of-digital-banks)
7. [The Technology Stack: Databases, Middleware, Chips, OS, and 信创](#7-the-technology-stack-databases-middleware-chips-os-and-信创)
   - 7.1 [The Database Landscape](#71-the-database-landscape)
   - 7.2 [Middleware and Application Servers](#72-middleware-and-application-servers)
   - 7.3 [Chips and Servers: x86, Kunpeng, Hygon, Phytium](#73-chips-and-servers-x86-kunpeng-hygon-phytium)
   - 7.4 [Operating Systems: Kylin, UOS, openEuler](#74-operating-systems-kylin-uos-openeuler)
   - 7.5 [信创 (Xinchuang): The Domestic Substitution Framework](#75-信创-xinchuang-the-domestic-substitution-framework)
8. [China vs. the West: Comparison and Implications](#8-china-vs-the-west-comparison-and-implications)
   - 8.1 [The Comparison Table](#81-the-comparison-table)
   - 8.2 [What Western Banks Can Learn](#82-what-western-banks-can-learn)
   - 8.3 [Foreign Banks Operating in China](#83-foreign-banks-operating-in-china)
9. [Singapore and Asia Relevance](#9-singapore-and-asia-relevance)
   - 9.1 [GLDB and the Singapore–China Corridor](#91-gldb-and-the-singaporechina-corridor)
   - 9.2 [Chinese Banks in Singapore](#92-chinese-banks-in-singapore)
   - 9.3 [Chinese Fintech in Singapore: Ant, Tencent, Airwallex](#93-chinese-fintech-in-singapore-ant-tencent-airwallex)
   - 9.4 [The MAS Digital Bank Context](#94-the-mas-digital-bank-context)
   - 9.5 [Cross-Border: RMB Internationalization and CIPS](#95-cross-border-rmb-internationalization-and-cips)
   - 9.6 [Learning Points for Singapore and Global Banks](#96-learning-points-for-singapore-and-global-banks)
10. [The Future: 2026 and Beyond](#10-the-future-2026-and-beyond)
    - 10.1 [Xinchuang 2.0: The Full Domestic Stack Mandate](#101-xinchuang-20-the-full-domestic-stack-mandate)
    - 10.2 [AI in the Chinese Core](#102-ai-in-the-chinese-core)
    - 10.3 [Cloud-Native Maturation](#103-cloud-native-maturation)
    - 10.4 [Internationalization of Chinese Banking Technology](#104-internationalization-of-chinese-banking-technology)
    - 10.5 [Trends Summary](#105-trends-summary)
11. [Glossary](#11-glossary)
12. [Claims-Status Table](#12-claims-status-table)
13. [References](#13-references)

---

### How to Read This Guide

This is the dedicated deep-dive on **core banking systems used by Chinese banks** — the companion to the umbrella guide [core_banking_systems_guide.md](core_banking_systems_guide.md) (which covers the global core-banking discipline: what a core is, the vendor landscape, modernization approaches). It assumes the reader knows the basics from that guide and focuses on what is *different* about China: the scale, the state-driven technology policy, the self-development tradition, and the distributed/cloud-native core transformation wave. Related sibling guides in this series that this guide cross-references: [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md) (FLEXCUBE/OBMA, historically strong in China), [temenos_guide.md](temenos_guide.md), [apache_fineract_guide.md](apache_fineract_guide.md), [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) (the Singapore–China corridor digital bank), [financial_infrastructure_guide.md](financial_infrastructure_guide.md) (CNAPS, CIPS, China's payment rails), and [banking_limits_domain_guide.md](banking_limits_domain_guide.md).

Suggested reading paths:

- **New to the domain** — [core_banking_systems_guide.md](core_banking_systems_guide.md) Sections 1–2 first, then this guide's [Section 1](#1-the-chinese-banking-sector-tiers-scale-and-constraints).
- **Sizing a Chinese modernization program** — [Sections 4](#4-the-self-development-phenomenon-自主研发) and [5](#5-去ioe-and-the-distributed-core-transformation).
- **Vendor/product research** — [Section 3](#3-the-domestic-vendor-landscape) and the [claims-status table](#12-claims-status-table).
- **Singapore/Asia angle** — [Section 9](#9-singapore-and-asia-relevance) and [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md).

A note on verification: this guide was researched with web searches in August 2026. China-specific facts that were **verified** from retrievable sources are marked as such; items that are widely reported in industry literature but could **not** be independently verified in this session (e.g., some internal system names) are explicitly flagged, and everything China-specific is summarised in the [claims-status table](#12-claims-status-table) at the end.

---

## 1. The Chinese Banking Sector: Tiers, Scale, and Constraints

### 1.1 The Bank Tiers

China's banking system is the world's largest by assets and is structured in a distinct hierarchy of tiers, each with its own core-banking technology economics. Understanding the tiers is essential because **core system strategy follows tier**: the Big Four self-develop; the joint-stock banks mix self-development with domestic vendors; city and rural banks buy domestic vendor cores; and the digital banks build cloud-native cores from scratch.

| Tier | Examples (Chinese name) | Approx. count | Core-system profile |
|---|---|---|---|
| **State-owned commercial banks** ("Big Four"/"Big Five") | ICBC 工商银行, CCB 建设银行, ABC 农业银行, BOC 中国银行 (+ BoCom 交通银行 when counting five) | 5–6 (incl. PSBC 邮储银行) | Self-developed cores; mainframe legacy; world's largest workloads |
| **Policy banks** (政策性银行) | CDB 国家开发银行 (China Development Bank), ADBC 中国农业发展银行 (Agricultural Development Bank of China), EximBank 中国进出口银行 (China Export-Import Bank) | 3 | Wholesale/policy lending, bond issuance; no retail networks; in-house + vendor systems |
| **Joint-stock commercial banks** (股份制商业银行) | CMB 招商银行, SPDB 浦发银行, CITIC 中信银行, CMBC 民生银行, CIB 兴业银行, CEB 光大银行, Ping An Bank 平安银行, Huaxia 华夏银行, CGB 广发银行, Zheshang 浙商银行, Bohai 渤海银行, Hengfeng 恒丰银行 | 12 | The "second tier": self-developed where strong (CMB), FLEXCUBE legacy (Ping An), distributed cores with GoldenDB/GaussDB (CITIC), etc. |
| **City commercial banks** (城市商业银行) | Bank of Beijing 北京银行, Bank of Shanghai 上海银行, Bank of Jiangsu 江苏银行, Bank of Ningbo 宁波银行, Bank of Nanjing 南京银行, Bank of Dongguan 东莞银行 | ~125 | Mostly domestic-vendor cores (Sunline, DCITS, GienTech, Yusys); the "distributed core" mid-market |
| **Rural commercial banks / credit cooperatives** (农村商业银行/农信社) | Changshu RCB 常熟农商行, hundreds of provincial-union RCCs, ~1,600 village banks 村镇银行 | ~3,000+ institutions | Small scale; vendor cores, increasingly "banking-as-a-service" on vendors' distributed platforms |
| **Digital/online banks** (互联网银行/直销银行) | WeBank 微众银行 (Tencent), MYbank 网商银行 (Ant), XWBank 新网银行 (New Hope + Xiaomi), aiBank 百信银行 (CITIC + Baidu), Yillion 亿联银行, Suning Bank 苏宁银行 | ~6 | Greenfield cloud-native cores; fully distributed; API-first |

Notes and clarifications:

- **The "Big Four" vs "Big Five" vs "Big Six".** The four state-owned commercial banks (ICBC, CCB, ABC, BOC) are the classic "Big Four". Bank of Communications (BoCom 交通银行) is often added to make the "Big Five", and Postal Savings Bank of China (PSBC 邮储银行, the world's largest by branch count with ~40,000 outlets) is sometimes included as the sixth. The Big Four are all classified by the PBOC/NFRA in the top group of **systemically important banks** (China published its first list of 21 domestic systemically important banks in December 2020, with ICBC, BOC, CCB and ABC in the highest group).
- **Policy banks** do not take retail deposits and have no retail branches; their cores support project finance, policy lending, bond issuance and treasury operations — a different (wholesale) core profile.
- **The rural tier is enormous in count but tiny per-institution.** Thousands of rural commercial banks, rural credit unions (农信社) and village banks mean the *volume* of core installations in China is dominated by the long tail — and this is exactly where domestic vendors make most of their money, often running the same core for dozens of small banks (the "banking-as-a-service" model, see [3.9](#39-domestic-vendor-characteristics)).
- **Digital banks** (Section 6) are a separate regulatory category of internet-only banks without physical branches, each backed by a major technology or industrial group.

### 1.2 Scale: The World's Largest Banking Workloads

The scale of Chinese banking is unlike anything in the West, and it is the single most important factor shaping core-system architecture. Key facts:

- **The Big Four alone serve over a billion customers.** ICBC reports on the order of **700 million personal customers** (roughly 7.3亿) plus millions of corporate accounts; ABC is comparable or larger. Together the Big Four count their retail relationships in the **billions** — more people than the population of Europe. (Customer counts are approximations from annual reports/industry reporting; see claims table.)
- **Trillions in assets.** ICBC alone holds assets in the tens of trillions of RMB (on the order of US$6 trillion in recent years) — routinely cited as the world's largest bank by assets. The Big Four combined exceed US$20 trillion.
- **Transaction volumes measured in the hundreds of millions per day** for the largest banks. ICBC's payment and account transaction volumes have long been described as the largest of any bank in the world; core systems at this scale process **billions of transactions per day across the group** when counting every channel (ATMs, POS, mobile, online, interbank).
- **Mobile-first ubiquity.** China's mobile-payment penetration means a very large share of bank transactions originate from mobile apps — with WeChat Pay and Alipay in front of the bank cores, plus the banks' own super-apps. The core is hit not only by direct account transactions but by the settlement of hundreds of millions of daily third-party-payment transactions.
- **Trillions in assets.** ICBC alone holds assets in the tens of trillions of RMB (on the order of US$6 trillion in recent years) — routinely cited as the world's largest bank by assets. The Big Four combined exceed US$20 trillion.

A snapshot of the giants (approximate, from annual-report-era reporting — treat as indicative):

| Bank | Head office | Scale indicators (approx.) | Core-relevant notes |
|---|---|---|---|
| ICBC 工商银行 | Beijing | ~US$6T assets; ~700M personal customers; ~450k employees; tens of thousands of branches/outlets | World's largest bank by assets; largest mainframe legacy; most advanced migration program |
| CCB 建设银行 | Beijing | ~US$5T assets; ~700M personal customers | New-generation core in-house (2017-era); GaussDB core scenarios |
| ABC 农业银行 | Beijing | ~US$5T assets; ~900M personal customers (largest customer base among the Big Four, incl. rural) | Huge rural/micro-finance product set; GaussDB scenarios |
| BOC 中国银行 | Beijing | ~US$4T assets; ~600M personal customers | Largest overseas network among Chinese banks; group-developed overseas platforms |
| PSBC 邮储银行 | Beijing | ~40,000 outlets — the world's largest branch network | Distributed core fully live on Huawei stack (verified) |

**Why this matters for core banking:** a core platform that comfortably handles 5,000 TPS for a European retail bank is a toy at ICBC scale. The Chinese core problem is not "can it do the business logic" but "can it do the business logic for a billion customers, at 10,000–100,000+ peak TPS, with five-nines expectations, without ever losing a posting." That is why every serious Chinese core discussion ends up at distributed architecture.

### 1.3 Peak-Season Loads: Spring Festival and 双11

Chinese cores are designed against **seasonal demand spikes** that are themselves world records:

- **Spring Festival (春节):** the world's largest annual human migration. Hundreds of millions of people travel, and — critically for banks — red envelopes (红包) and remittances spike. WeChat and Alipay red-packet campaigns have historically driven **billions of transactions in a single day** (e.g., reported figures of over 1 billion red packets handled in a single evening in peak years). The underlying bank settlement and account systems must absorb the post-festival clearing surge without degradation.
- **双11 (Singles' Day, 11 November):** Alibaba's shopping festival is the world's largest online-sales event, and Alipay processes the payments. The widely reported peak figures: **256,000 payments/second in 2017** (a then-world record for payment transactions per second) and on the order of **583,000 TPS (58.3万笔/秒) in 2020** (reported by Alibaba/Ant; widely quoted, see claims table). While Alipay's own systems absorb most of that, the *bank* side — MYbank lending, bank-card authorisation through UnionPay/NetsUnion, bank settlement accounts — all sees order-of-magnitude spikes.
- **Other peaks:** 618 (mid-year shopping festival), New Year's Eve, salary days, tax-deadline days, and policy-driven windows (e.g., savings-bond issuance days).

The design consequence: Chinese cores are engineered for **peak, not average**. Distributed cores scale horizontally before the event; capacity is provisioned for the spike; batch windows are compressed because the system must be effectively 24/7.

### 1.4 Core System Requirements at Chinese Scale

The requirements profile for a Chinese bank core (particularly the big banks) is distinctive:

- **Extreme concurrency.** Peak TPS at the core layer itself (postings, queries, authorisations) is measured in tens of thousands to over one hundred thousand for the largest payment chains. A single logical account database cannot do this; the data must be sharded (see [5.3](#53-distributed-core-architecture-patterns)).
- **Availability expectations.** Chinese bank mobile apps operate 24/7/365, and the regulatory expectation (and public expectation) is effectively five-nines for core functions. Outages at big banks are front-page news in China (and have happened — several high-profile core outages in the 2010s–2020s triggered regulatory attention and pushed the industry toward 双活/dual-active designs).
- **Batch windows compressed.** The classic end-of-day batch (interest accrual, fees, statements) that could run for hours on a mainframe must now complete in a fraction of the night, because the "night" barely exists when customers transact at 2 a.m. Distributed cores spread batch across shards in parallel.
- **Data at PB scale.** A big bank's core and data platform hold petabytes; account data alone runs into the tens of billions of records. Sharding, archiving and tiered storage are mandatory.
- **Full auditability and reconciliation.** Every posting must reconcile to the fen (cent) across the general ledger, the payment rails, and the channels; regulatory reporting (to PBOC, NFRA, CSRC, SAFE) is generated from core data continuously.
- **Regulatory and policy compliance** as a first-class requirement: 信创 (domestic-stack substitution, [7.5](#75-信创-xinchuang-the-domestic-substitution-framework)), data-localisation, and systemically-important-bank resilience standards (the PBOC/NFRA have issued business-continuity and disaster-recovery requirements for systemically important banks, including RTO/RPO targets for core systems).

### 1.5 The Regulatory Environment: PBOC, NFRA, and the Data Laws

The regulatory map every core-banking architect in China must navigate:

- **The People's Bank of China (PBOC 中国人民银行)** — the central bank. Beyond monetary policy, it operates the payment and clearing infrastructure (CNAPS, CIPS), licenses payment institutions, and drives the **金融科技发展规划** (FinTech Development Plans) that set the technology direction for the industry. PBOC's 科技司 (technology department) has been a direct force behind distributed-core adoption and 信创.
- **The National Financial Regulatory Administration (NFRA 国家金融监督管理总局)** — the prudential regulator for banks and insurers, created in **2023** by absorbing the CBIRC. The CBIRC (银保监会) itself was formed in 2018 from the merger of the CBRC (banking) and CIRC (insurance). NFRA sets IT-risk, business-continuity, and outsourcing rules for banks.
- **CSRC** (securities regulator) and **SAFE** (foreign exchange) matter for the securities and FX-adjacent systems.
- **Systemically important bank designation** — the PBOC/NFRA list (first published December 2020, 21 banks, updated periodically) imposes higher loss-absorbency and — in practice — higher technology-resilience expectations on the largest banks, including the Big Four at the top.
- **The data laws** — the legal frame that now governs every Chinese core's data handling:
  - **网络安全法** (Cybersecurity Law, 2017) — network security obligations for critical information infrastructure (CII) operators; banks are CII operators.
  - **数据安全法** (Data Security Law, effective **1 September 2021**) — establishes data classification (重要数据/important data), national-security review for data processing, and **data localisation** obligations for important data, including cross-border transfer restrictions.
  - **个人信息保护法** (Personal Information Protection Law, PIPL, effective **1 November 2021**) — China's GDPR analogue; governs personal-information processing, cross-border transfers (security assessments via the CAC), and consent — with direct implications for foreign banks and any core that touches Chinese customer data.
  - **金融数据分级分类** (financial-data classification rules) issued by the PBOC operationalise these laws for banks.
- **The 信创 policy machinery** — the IT-application-innovation (信息技术应用创新) programme, detailed in [7.5](#75-信创-xinchuang-the-domestic-substitution-framework), which now shapes *what* technology banks may buy for new core builds.

For foreign banks operating in China (Section 8.3), these laws are the compliance perimeter: data residency, cross-border transfer approval, and now xinchuang expectations for locally licensed entities.
---

## 2. The Historical Core Landscape

### 2.1 Era 1: The Mainframe Era (1980s–2000s)

Chinese banking computerisation began in earnest in the 1980s, with the state-owned banks rolling out branch-level automation, then regional systems. The defining technology of the first era was the **IBM mainframe**:

- From the late 1980s through the 1990s, the big banks standardised on **IBM z/OS mainframes (System/370 → ES/9000 → zSeries)** running COBOL applications under CICS, with **DB2** (and, in many mid-tier systems, Oracle) as the database. ICBC in particular built one of the **largest mainframe estates in the world** — its data centres ran more mainframe capacity than most countries' entire banking systems.
- Core applications were written in **COBOL**, organised as monolithic batch + online programs: overnight batch for interest, statements and clearing; online transactions via CICS for tellers, ATMs and early electronic channels.
- The architecture was **branch-centralised**: each branch or province ran its own systems and its own data, and inter-branch transactions were settled through the head office in batch. A customer of a branch was, in effect, a customer of that branch — the classic pre-"core banking" model.

The stack details are worth stating precisely because they define what 去IOE later had to dismantle: **z/OS + CICS + COBOL + DB2** on IBM mainframes, with hierarchical/VSAM files in the earliest years, batch windows at night, and green-screen branch terminals (and later PC-based front-ends) driving CICS transactions. The mainframe operations teams were elite units: mainframe MIPS capacity, storage and batch-window management were core competencies, and the data centres were among the largest commercial computer installations in the world. Around the same era (late 1990s–2000s), the first electronic channels arrived — ATM networks, telephone banking, and then internet banking (CMB's 一网通 launched in the late 1990s was China's first widely used internet banking) — all hanging off the batch-centric mainframe core as satellite systems.

### 2.2 The Data Concentration Wave: 数据大集中 (1990s–2000s)

The pivotal event of Chinese core-banking history is the **数据大集中 (data centralisation/concentration) wave** of roughly 1995–2006. The logic: as the big banks grew, dozens of provincial data centres meant duplicated infrastructure, inconsistent products, and no bank-wide view of a customer. The PBOC and the banks themselves pushed for consolidation.

- Each of the Big Four consolidated from **dozens of provincial computer centres down to a small number of national data centres** (typically two or three, with disaster recovery), so that a single core installation served the entire bank.
- The enabling technology was the mainframe scaled up: ICBC, for example, concentrated onto very large z/OS environments — at the time among the largest, if not the largest, commercial mainframe workloads in the world. (Specific consolidation timelines vary by bank — commonly cited windows: ICBC completed its major concentration around 2002–2004, BOC and the others through the early-to-mid 2000s. These specifics are reported in industry literature rather than verified in this session.)
- The consequence: **Chinese banks became "core-banking centralised" a decade before many Western banks**, but at the cost of extreme dependence on a single monolithic technology stack — exactly the dependency that 去IOE would later dismantle.
- "Core banking" (in the centralised sense explained in the umbrella guide) arrived in China through this wave, driven by the mainframe, not by packaged COTS cores.

### 2.3 Era 2: The Client-Server Era

As the mainframe estates aged, the 2000s brought client-server architectures into Chinese banking — but with a Chinese twist:

- **Unix servers (IBM pSeries/AIX, HP, Sun) + Oracle databases** became the standard for new systems — channels (internet banking, call centres), back-office, risk, and a growing share of account-processing systems. Oracle became the de facto database standard of the Chinese banking industry outside the mainframe core.
- Mid-sized banks (city commercial banks, rural banks) that could never afford mainframes built their first "real" core banking systems in this era on client-server platforms — often packaged cores (FLEXCUBE or domestic products) on Oracle.
- The big banks ran a **two-world architecture**: mainframe core (accounts, GL) + open-systems satellite (channels, CRM, risk, payments). This split — closed core, open periphery — is still visible in some banks today.

This is also the era in which the **domestic vendor industry was born**. The client-server wave created the demand that founded (or transformed) today's core vendors: **长亮科技 (Sunline) was founded in 2002** in Shenzhen; **神州数码融信 (Digital China Financial Information)** grew out of Digital China's financial services business in the late 1990s–2000s; **宇信科技 (Yusys)** and **文思海辉 (Pactera, now GienTech)** built their banking delivery franchises in this period. Their first products were client-server cores and channel systems for city and rural banks — the tier that could not afford mainframes and that Oracle/FLEXCUBE-style packaged software served imperfectly. The vendor-customer relationships formed then (a city bank's first core vendor) have largely persisted into the distributed era — an important lock-in dynamic.

### 2.4 Era 3: The Distributed Era (2010s–Present)

The distributed era is where China diverges from the West and becomes a technology exporter of architectural patterns. Triggered by Alibaba's 去IOE experience ([5.1](#51-the-de-ioe-movement-origins-and-meaning)) and accelerated by national policy, the 2010s saw:

- **WeBank (2014) and MYbank (2015)** build the first fully distributed, cloud-native bank cores from scratch — proving the pattern at national scale ([Section 6](#6-the-digital-banks-cores)).
- **The joint-stock banks** launch distributed-core projects: CITIC Bank with ZTE's GoldenDB ([5.4](#54-the-flagship-transformation-projects)), CMB and Ping An Bank moving to distributed platforms.
- **The big banks** begin mainframe-to-open distributed migration programs (2019 onward), running new accounts and new products on distributed cores while legacy runs on mainframes — the 双轨运行 (dual-core parallel operation) model ([5.5](#55-timelines-and-the-state-of-play-by-2026)).
- **Domestic databases mature** (OceanBase, GaussDB, GoldenDB, TiDB) to the point of running bank core workloads, and 信创 policy ([7.5](#75-信创-xinchuang-the-domestic-substitution-framework)) makes domestic stacks mandatory for new builds.

By 2026 the distributed core is the default answer for *new* core investment in China; the question is no longer "should we distribute" but "how far along is the migration."

### 2.5 International Vendors in China: IBM, Oracle, SAP, Temenos

The international core-banking vendors' story in China is one of **early hardware dominance, mid-period database dominance, and persistent marginality in core applications**.

- **IBM** — the historical king: mainframes, CICS, DB2, and the entire data-centre paradigm of the big banks (2.1–2.2). IBM's influence in China peaked with the mainframe era; its China banking business has since shifted to selling Power servers, storage, and (increasingly limited) mainframe maintenance as the banks migrate off. IBM is also the explicit target of 去IOE.
- **Oracle** — dominant in the **database** layer across Chinese banking (the client-server standard, 2.3) and the vendor with the most credible **core application** footprint: **Oracle FLEXCUBE**. Verified FLEXCUBE installations in China include **Ping An Bank** (selected March 2010 after a six-month evaluation against Chinese and global cores; went live with FLEXCUBE core banking, with Ping An Technology as the group's system integrator) and **China Dasheng Bank** (implementation reported in vendor/partner materials). See [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md) for FLEXCUBE/OBMA architecture. FLEXCUBE's Chinese story is otherwise mostly **foreign bank branches and smaller entrants**, not the mainstream domestic banks — and notably, Ping An Bank has since moved toward in-house/distributed systems, illustrating the churn.
- **SAP** — **SAP for Banking** had only **limited adoption in China**. Chinese banks generally did not take SAP's core; SAP's Chinese banking footprint has been in adjacent modules (some banks use SAP for finance/HR/ERP, and there have been niche banking deployments) rather than the retail core. This is consistent with SAP's global position (strong in Europe, weak in Asia retail cores).
- **Temenos (T24)** — a **China presence but no mainstream domestic install base**. Verified touchpoints: Temenos has a Shanghai entity ("Temenos Software (Shanghai) Co., Ltd", still recruiting as of 2026) and T24 implementations at **China Merchants Bank's New York subsidiary** (chosen over Misys and Fiserv for a rapid, low-risk implementation with bilingual support) and at foreign banks' China branches (e.g., **Swedbank's Shanghai office** going live on T24 in 2007 to meet local regulatory requirements). The Chinese banks using T24 are overwhelmingly **overseas subsidiaries and foreign-branch operations**, not mainland cores. (One vendor-directory source lists Agricultural Bank of China among T24 users — plausibly for an overseas subsidiary — but this is **unverified**; see claims table.) The GLDB case in Singapore (Temenos on Huawei Cloud) shows Temenos' relevance to Chinese-owned entities *outside* the mainland; see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md).

### 2.6 The Reality: Limited International Core Penetration

The honest summary of the international vendors' place in Chinese core banking:

- **They won the platform layer (hardware, databases, OS) for 30 years** — IBM mainframes, Oracle databases, EMC storage — and that is exactly what 去IOE targets.
- **They lost the core-application layer.** FLEXCUBE, T24, and SAP for Banking never became the standard core of a single major Chinese bank. The reasons:
  1. **Scale.** No packaged core was ever built for billion-customer, hundred-thousand-TPS workloads. The Big Four had to self-develop (Section 4).
  2. **Chinese product complexity.** Chinese deposit/loan products, the 存贷合一 current accounts, interest conventions (e.g., 积数-based interest accrual), the regulatory reporting formats, and the Chinese-language/charset requirements made COTS parameterisation a poor fit.
  3. **Price and support.** License + per-account pricing at Chinese scale was uneconomic; local support in Chinese for 24/7 operations was thin.
  4. **The state factor.** From the 2000s (after the banking-sector IT-security push) and especially post-2018, policy has actively steered away from foreign core software ([7.5](#75-信创-xinchuang-the-domestic-substitution-framework)).
- **The result:** a market where **domestic vendors serve the mid/low tier** and **the big banks self-develop**, with international vendors confined to foreign-bank branches, overseas subsidiaries of Chinese banks, and the platform layer — a structure that has no parallel in any other large banking market.

---

## 3. The Domestic Vendor Landscape

### 3.1 The Vendor Map at a Glance

The domestic core-banking vendor market is consolidated around a handful of public companies, all of which also act as system integrators (SI) — the Chinese model is "product + project" (产品+项目), where the vendor both licenses its core and delivers it bank-by-bank.

| Vendor (Chinese) | English name / ticker | Core-banking role | Representative customers (verified or widely reported) |
|---|---|---|---|
| 长亮科技 | **Sunline** (Shenzhen Sunline Tech Co., Ltd; pinyin Changliang), SZSE 300348 | The leading domestic **core-banking product** vendor; distributed core + GaussDB joint solution | Bank of Nanjing, Bank of Dongguan, many city/rural banks; CITIC Bank credit-card domestic-DB project |
| 神州信息 / 神州数码融信 | **DCITS** (Digital China Information Service Group, SZSE 000555) | Full banking suite; **Sm@rtOneBank** integrated/distributed banking platform | Mid-size city banks; long track record as bank SI |
| 中电金信 | **GienTech** (CEC-affiliated, formerly Pactera 文思海辉) | Full-stack fintech: core, channels, data; **ORIGIEN** financial-grade digital platform | Claims 95%+ of Chinese banks as clients (legacy Pactera claim); big-bank SI heritage |
| 宇信科技 | **Yusys Technologies** (SZSE 300674) | End-to-end banking IT: core-adjacent, channels, financial cloud | Joint-stock and city banks |
| 天阳科技 | **Teamax** (SZSE 300872) | Banking IT services: credit-card, risk, core projects | Mid-tier banks (name verification: see claims table) |
| 华为 | **Huawei** (GaussDB, Kunpeng/Taishan, 华为云) | Platform giant: database, chips, servers, cloud, distribution framework | ICBC (GaussDB), PSBC (distributed core), CCB/ABC/BOC (GaussDB scenarios), joint solutions with Sunline |
| 中兴通讯 | **ZTE** (GoldenDB) | Distributed database vendor | CITIC Bank (core "heart transplant"), multiple banks |
| 恒生电子 | **Hundsun** (SHSE 600570) | Securities/funds/wealth systems; banking wealth-management & channel products (not core) | Securities firms, banks' wealth side |
| 科蓝软件 | **Kelan** (SZSE 300663) | Internet/mobile banking channel systems, open banking | City banks' e-banking |
| 顶点软件 | **ApexSoft** (SHSE 603383) | Securities + niche banking systems | Securities firms mainly |

### 3.2 Sunline / Changliang Technology (长亮科技)

**Background.** Founded 2002, headquartered in Shenzhen; listed on the Shenzhen Stock Exchange (300348). It is the closest thing China has to a dedicated **core-banking product house** (as opposed to an SI that also sells software). An important naming correction: 长亮科技 is frequently mistranslated as "Longshine" — that name actually belongs to a **different** company (LongShine Technology Group 朗新科技, 300682, an energy/utility software firm in Wuxi). The verified English names for 长亮科技 are **"Changliang Technology"** (pinyin) and its international brand **"Sunline"** (Shenzhen Sunline Tech Co., Ltd).

**Products.**
- **Sunline Core Banking System** — a parameterised (参数化) core: business processes, business elements and financial products are configured rather than coded, so a bank can launch products quickly. This has been the workhorse for hundreds of city/rural-bank core implementations.
- **Distributed core** — the modern product line: microservices-based, cloud-native core for banks migrating off Oracle/mainframe cores, including a **joint distributed-core solution with Huawei** released in September 2021 based on **Huawei GaussDB + Taishan (泰山) servers** — a flagship 信创 stack pairing (domestic app vendor + domestic DB + domestic hardware).
- Sunline also operates an international arm selling core banking and digital-banking solutions in Southeast Asia and other markets.

**Customers.** Verified/widely reported: **Bank of Nanjing, Bank of Dongguan**, and many regional banks where "Sunline's core has often been a centerpiece of modernization efforts" (market-analysis source, April 2025); plus project work at **CITIC Bank** (credit-card core adopting domestic databases). Sunline publishes an influential industry report, *Sunline's Observation: 10-Year History of Core Changes in China's Banking Industry* — itself evidence of the vendor's role as the chronicler of Chinese core migrations.

**Technology profile.** Traditional parameterised core for the mid-market → distributed/microservices core for the migration wave; strong domestic-stack partnerships (Huawei GaussDB) → squarely positioned for 信创-era deals.

### 3.3 DCITS / Digital China Information Service (神州信息, 神州数码融信)

**Background.** Digital China Information Service Group Co., Ltd. (DCITS, 神州信息, SZSE 000555) is the information-services arm of the Digital China (神州数码) ecosystem; its banking-software lineage runs through **神州数码融信 (Digital China Financial Information)**, the financial-IT unit that historically built banking core systems for Chinese banks. Founded 1984 lineage; ~20,700 employees; 300+ financial software products (company disclosures).

**Products.** The flagship is **Sm@rtOneBank**, an integrated banking solution (white-papered by the company as its "one-stop banking overall solution") spanning core banking, channels, payments and data, with distributed-core capabilities for the modernization wave. DCITS also has deep data/BI and agricultural-finance systems (strong with the rural tier).

**Customers.** Historically strong with mid-size city commercial banks and the rural tier; one of the two or three vendors (with Sunline) that a mid-tier Chinese bank's CIO would shortlist for a core rebuild.

**Technology profile.** Traditional core + distributed "next-generation" core; heavy SI culture; strong data platforms.

### 3.4 GienTech (中电金信)

**Background.** GienTech (中电金信) is the financial-technology platform of **China Electronics Corporation (CEC 中国电子信息产业集团)** — the state-owned electronics conglomerate — launched as a brand in **January 2021**, absorbing the financial-services business of **Pactera (文思海辉)** (which itself traced back to 1995 and the Haihui/HiSoft heritage). CEC GienTech is positioned as "the core focus of China Electronics Corporation's efforts in the digitalization of the financial industry" — i.e., the state's own financial-software champion, a natural beneficiary of 信创.

**Products.** Full-stack fintech: core banking, channels, data, cloud — packaged as the **ORIGIEN Financial-Grade Digital Platform**, debuted internationally in Hong Kong in July 2023 as part of a push into Asia-Pacific markets.

**Customers.** Pactera's legacy claim of serving "over 95% of banks in China" (as an SI/outsourcing partner across consulting, application development, testing and operations) is the basis of its reach; the GienTech-era focus is full-platform delivery and international expansion. The 95% figure is a vendor claim (see claims table).

**Technology profile.** SI-heavy, full-stack, state-affiliated; strong in large-bank delivery projects; the "system integrator of record" for 信创-era megaprojects.

### 3.5 Yusys Technologies (宇信科技)

**Background.** Yusys Technologies Co., Ltd. (宇信科技, SZSE 300674) is a banking-focused IT company providing end-to-end solutions: consulting, software products, implementation, operations and system integration.

**Products.** Core banking systems, channel systems (internet/mobile banking), financial cloud services, and credit/risk systems; its core-banking and financial-cloud offerings are the flagships (CB Insights categorises it as "IT solutions for the financial industry, specifically core banking systems and financial cloud services"). Product configuration is parameterised and rule-based — e.g., designing new loan products and tailoring workflows within parameterisation and rule frameworks (market-analysis source).

**Customers.** Joint-stock and city commercial banks; strong in the mid-market alongside Sunline and DCITS.

### 3.6 Teamax (天阳科技) and the Mid-Tier IT Vendors

**Teamax** (天阳科技, full corporate name 天阳宏业科技股份有限公司, SZSE 300872) is a banking-IT services and products company active in core-banking implementation, credit-card systems and risk platforms for mid-tier banks. The English name "Teamax" is reported in company/aggregator profiles; the Chinese name 天阳科技 is unambiguous. Also in this mid-tier: **高伟达 (banking IT services)**, **润和软件**, and numerous smaller regional SIs that implement and localise the top vendors' cores. These firms matter because Chinese core projects are delivered as long-term on-site engagements — the SI layer is as important as the product layer.

### 3.7 Huawei and ZTE: The Platform Giants

Neither Huawei nor ZTE sells a packaged core banking application in the Temenos sense, but both are now **structural players** in Chinese core banking:

- **Huawei (华为).** The full-stack platform: **GaussDB** distributed database (PostgreSQL-based; open-sourced as openGauss), **Kunpeng (鲲鹏) ARM server chips** and **Taishan (泰山) servers**, **华为云 (Huawei Cloud)** with financial-industry zones, and distribution/middleware components. Verified banking wins: **ICBC migrated 200+ services from Oracle to Huawei Cloud GaussDB, deployed 3,000+ nodes, and cut RTO ~10x**; GaussDB supports core-business scenarios (credit, payment, clearing) at ICBC, PSBC, CCB and ABC per Huawei/Baidu-Baike materials; **PSBC's distributed core system** went fully live on the Huawei stack (Huawei Tech magazine); ICBC–GaussDB joint work won an industry award (2023); and Huawei Cloud co-authored the *GaussDB Maintenance White Paper for the Financial Sector* with ICBC, PSBC, BOC, BoCom and Huaxia (2025). Huawei also partners with app vendors (Sunline's GaussDB-based distributed core, 3.2).
- **ZTE (中兴通讯).** The distributed database **GoldenDB**: shared-nothing OLTP, jointly developed with **CITIC Bank beginning 2014**, and used for CITIC Bank's core "heart transplant" (换心) — replacing the legacy database under the core. GoldenDB is one of the most proven domestic databases *inside a major bank core*.

### 3.8 The Securities-Channel Vendors: Hundsun, Kelan, ApexSoft

Three vendors often appear in "Chinese banking IT" lists but are **not core-banking vendors** in the account-processing sense:

- **Hundsun (恒生电子, 600570)** — the dominant Chinese securities/funds/wealth-management systems house; its banking business covers wealth-management, treasury and channel products for banks, not the retail core.
- **Kelan (科蓝软件, 300663)** — internet banking and mobile-banking **channel** systems, open-banking APIs, and e-banking infrastructure for city banks.
- **ApexSoft (顶点软件, 603383)** — primarily securities trading systems, with niche banking work.

They matter to the core story only as the channel layer that hangs off the core — and as proof that the Chinese market is segmented (core vs channel vs securities) far more cleanly than in the West.

### 3.9 Domestic Vendor Characteristics

Why the domestic vendors won the Chinese mid-market and now feed the distributed wave:

- **Price advantage.** License and delivery costs are a fraction of international packaged cores — essential when selling to a 300-employee rural commercial bank.
- **Localisation.** Native Chinese-language support, Chinese accounting/interest conventions, PBOC/NFRA regulatory-reporting formats built in, and local 7x24 support — the "3,000 km of documentation in English" problem of international cores is absent.
- **"Banking-as-a-service" to the small rural tier.** The thousands of rural banks cannot each afford a core team. Vendors run shared/multi-tenant distributed platforms where a small bank gets a full core (often hosted on the vendor's or a partner's cloud) for a fraction of the cost of ownership — the Chinese version of BaaS, driven by domestic vendors rather than Western core-as-a-service vendors.
- **Distributed-core expertise.** Because the domestic vendors built distributed cores first (for the 2017+ wave), they now hold the reference implementations (Sunline's GaussDB-based core, DCITS Sm@rtOneBank distributed edition) that 信创-era deals demand — a moat against any international return.
- **The 信创 tailwind.** With policy pushing domestic stacks (7.5), domestic vendors are not just competitive but *mandated*: a city bank's 2024+ core RFP may simply exclude non-domestic candidates.

---

## 4. The Self-Development Phenomenon: 自主研发

### 4.1 The Big-Bank Pattern

The most important structural fact of Chinese core banking: **the Big Four and the strongest joint-stock banks do not buy cores — they build them.** The 自主研发 (independent R&D / self-development) tradition at ICBC, CCB, ABC, BOC and CMB is decades old and is the reason no packaged vendor ever became dominant in China (2.6). Chinese big-bank cores are in-house products maintained by in-house teams of thousands of engineers — a model that exists nowhere else at this scale (the nearest analogues are global banks' heavily customised cores, which are still vendor-derived).

### 4.2 ICBC: NOVA, FOVA, and the Distributed Future

ICBC is the archetype. Its core-banking systems are widely referenced in Chinese industry literature under the names:

- **NOVA** — ICBC's new-generation core banking system, built in-house in the early 2000s as the software backbone of the 数据大集中 era: the "New-generation Open Versatile Architecture" that replaced the batch of scattered provincial systems with a centralised, bank-wide core. (The name and full expansion are reported in industry literature; **not independently verified** in this session — see claims table.)
- **FOVA** — the corresponding core used by ICBC's **overseas** operations (the acronym is widely used in ICBC-related technology discussions for the offshore/foreign branch platform, built to globalised multi-currency, multi-language requirements rather than the domestic NOVA). Same verification caveat.
- **The distributed future.** Since ~2019 ICBC has run major migration programs off the mainframe and Oracle: verified moves include **OceanBase for important business systems (announced September 2020)** and **200+ Oracle services to Huawei GaussDB (3,000+ nodes, RTO cut ~10x)**. The trajectory: mainframe NOVA → dual-core parallel with distributed systems → progressive decommissioning of the legacy core. ICBC's engineering organisation (thousands of developers in its software development centres) makes this the largest in-house core program in the world.

ICBC's technology organisation is itself a datapoint: it maintains multiple **software development centres (研发中心)** (Beijing, Shanghai, Hangzhou, Guangzhou, Chengdu and others over the years) plus a central data centre and disaster-recovery sites, and its technology workforce (developers + operations + security) is commonly reported in the tens of thousands — larger than most countries' entire fintech workforces. ICBC treats IT as a strategic function: its annual technology budgets and its pattern of publishing banking-technology research (and, notably, its willingness to be an early production customer of OceanBase and GaussDB) make it both the anchor customer and the de facto quality gate for the entire domestic stack ecosystem.

### 4.3 CCB, ABC, BOC: The New-Generation Cores

- **CCB (建设银行).** Built its **新一代核心系统 (New-Generation Core System)** in-house (delivered around 2017) on an open-systems architecture, with a multi-year, multi-billion-RMB program that is a standard case study in Chinese banking IT. CCB also runs major domestic-database adoption (GaussDB core scenarios verified).
- **ABC (农业银行).** Similarly self-developed its new-generation core, with a huge rural/micro-finance product set; GaussDB is verified in ABC core scenarios (credit, payment, clearing).
- **BOC (中国银行).** Self-developed new-generation core banking system with the strongest international dimension (BOC's overseas network is the largest among Chinese banks, so its core must handle multi-currency, multi-jurisdiction operations; its overseas branches typically run group-developed platforms).
- **PSBC (邮储银行).** The sixth state-owned giant built its **distributed core system on the Huawei stack** — verified go-live (Huawei Tech magazine) — a landmark because PSBC has ~40,000 outlets and the largest branch network in the world, proving distributed cores at the very top of the scale curve.

Specific internal system names for CCB/ABC/BOC cores are not consistently public; what is public and verified is the pattern: in-house development, open-systems migration, domestic databases.

### 4.4 CMB: The Self-Reliant Pioneer

**China Merchants Bank (招商银行)** is famous in Chinese banking as the most IT-self-reliant major bank. Founded 1987 as the first share-holding commercial bank wholly owned by corporate legal entities, CMB built its technology edge early: it never became as mainframe-dependent as the Big Four, developed its own core and channel systems in-house from the 1990s, and its "一网通" internet banking was China's first. Its retail-banking success is widely attributed to this technology autonomy. CMB has since moved to a distributed, cloud-native platform for its new-generation core (announced in the late 2010s) and is a reference for both self-development *and* cloud migration among joint-stock banks. (The specific CMB core product names are not publicly standardised; treat as reported, not verified — claims table.)

### 4.5 Why Self-Develop? The Four Drivers

1. **Scale.** The commercial off-the-shelf cores — Temenos, FLEXCUBE, SAP, even the domestic products — were not built for ICBC's 700 million customers and hundred-thousand-TPS peaks. No vendor could demonstrate it; ICBC's engineers had to build it. At Big-Four scale, "buy" was never a real option.
2. **Strategic autonomy (自主可控).** After the US-China technology-export-control escalation (entity-list actions from 2019, sanctions risk, and the general decoupling climate), the cost of dependence on US vendors moved from commercial to strategic. Self-development is the ultimate form of 自主可控 (independent and controllable): the bank owns the source, the roadmap and the supply chain.
3. **Regulatory push.** PBOC/CBIRC-NFRA and the state's 信创 policy have explicitly encouraged 自主可控 in core systems; the regulator's own FinTech development plans steer banks toward independent control of critical systems. For systemically important banks, self-development is close to an expectation.
4. **Cost at scale.** Licensing a packaged core at per-account/per-installation pricing across a billion-account bank is uneconomic; in-house development amortises over the largest possible install base within the bank, and the big banks treat their IT centres as strategic cost centres with thousands of engineers already paid.

### 4.6 The Self-Development Pattern

The big-bank self-development model has a consistent shape:

- **In-house teams of thousands.** ICBC/CCB/ABC/BOC each run software development centres (研发中心) with thousands of developers (ICBC's combined technology workforce is commonly reported in the tens of thousands including operations). They hire from the same pool as Huawei/Tencent — the big banks are serious tech employers, not IT cost centres.
- **From mainframe to open systems.** The 2010s–2020s pattern: new systems on open-source-adjacent stacks (Java/microservices on Linux), legacy on mainframe, with progressive strangler migrations.
- **Domestic databases.** The big banks now standardise new builds on domestic distributed databases — verified examples: ICBC on OceanBase and GaussDB, CCB/ABC on GaussDB scenarios, CITIC on GoldenDB. (TiDB and Dameng also have bank deployments; see 7.1.)
- **Distributed architecture.** New cores are microservices on sharded databases with 单元化 (cell-based) deployment (5.3) — the design language the banks learned from the internet giants.
- **Vendors as workforces, not owners.** Even "self-developed" big-bank cores are delivered with armies of SI/vendor staff (GienTech, DCITS, Yusys, Teamax all run thousands of engineers inside big-bank programs) — but the *architecture, ownership and roadmap* stay with the bank. This is "self-development with outsourced capacity."

### 4.7 Self-Developed vs. Vendor: The Trade-Offs

| Dimension | Self-developed (Big Four, CMB) | Domestic vendor core (city/rural banks) |
|---|---|---|
| **Fit to scale** | Built for the actual workload — the only option at billion-customer scale | Parameterised for the mid-market; fine below ~10–50M customers |
| **Time to market for new products** | Slow in legacy years; fast now with product-factory frameworks on distributed cores | Fast — parameterisation is the product's core value |
| **Cost profile** | Huge fixed cost (thousands of engineers); low marginal cost per account | Low fixed cost; per-bank license + delivery fees; hosted BaaS cheapest |
| **Talent dependency** | Must retain deep in-house engineering capability | Vendor owns the product expertise; bank's dependency is contractual |
| **Compliance/信创** | Maximum 自主可控 — the strongest possible position | Domestic vendors are 信创-compliant by construction; fine |
| **Risk** | Program risk is the bank's own; schedule slippage at Big-Four scale is measured in years and billions | Vendor viability, product roadmap and (historically) quality vary |
| **Innovation** | Cradle of the distributed-core patterns China is now exporting | Rapid adoption of vendor-developed patterns (e.g., GaussDB-based cores) |
---

## 5. 去IOE and the Distributed Core Transformation

### 5.1 The De-IOE Movement: Origins and Meaning

**去IOE ("go-de-IOE", literally "remove IBM, Oracle, EMC")** is the Chinese industry term for replacing the three foreign platform giants — **IBM** (servers/mainframes), **Oracle** (databases), **EMC** (storage) — with domestic or open alternatives. The movement's origin is **Alibaba**, not the banks:

- Around **2009–2010**, Alibaba's core e-commerce systems hit the scalability ceiling of its Oracle + IBM Power + EMC stack, and — in parallel with a strategic desire for independence — Alibaba began replacing Oracle with a self-built distributed database that became **OceanBase**, and moving compute onto commodity x86 servers (the Apsara platform). Alibaba's "去IOE" became the canonical Chinese tech story of the 2010s: by ~2013 Alibaba publicly claimed to have substantially completed de-IOE for its core systems, and the term entered the national vocabulary as shorthand for technological self-reliance.
- **OceanBase** then proved itself in the hardest possible production environment (Alipay's payment core), and later set world records in the TPC-C OLTP benchmark (first to surpass Oracle's record in 2019; extended since) — turning "domestic database" from a slogan into a benchmark champion. OceanBase was spun out as an Ant Group company and now sells to banks globally (7.1).

For banking, 去IOE arrived as a double movement: **technical** (the scale ceiling of centralised stacks — the 数据大集中 mainframes were the most centralised systems in the world, and their batch windows and vertical-scaling costs were becoming unmanageable) and **political** (自主可控, accelerating sharply after the 2018–2019 US-China technology confrontation).

### 5.2 Banking 去IOE: From Alibaba to the Big Four

The banking phase of 去IOE is really the **distributed core transformation**:

- **2014–2017:** the digital banks (WeBank 2014, MYbank 2015) build cores with zero foreign platform dependence — the proof-of-concept for banking 去IOE at scale.
- **2017–2020:** the joint-stock banks go first among incumbents — **CITIC Bank** with ZTE's GoldenDB ("heart transplant" of the core, GoldenDB co-developed from 2014), CMB and Ping An Bank on distributed paths. **PSBC** builds its distributed core (Huawei stack, verified).
- **2019–2026:** the Big Four. ICBC's verified moves (OceanBase 2020; 200+ services to GaussDB; mainframe reduction) typify the approach: **migrate incrementally, keep the legacy core running beside the new systems, and let the new account/product volume flow to distributed systems** — the 双轨运行 (dual-core parallel run) model (5.5).
- **The database is the heart of banking 去IOE.** Replacing the mainframe CPU is relatively mechanical; replacing the *core database* is the risky, years-long "换心" (heart transplant) that vendors and banks describe with surgical metaphors. The domestic database landscape (7.1) exists because of this.

### 5.3 Distributed Core Architecture Patterns

The Chinese distributed core has converged on a well-defined set of patterns — the same playbook Alibaba/Tencent/Ant developed for internet scale, applied to banking:

- **Shared-nothing (无共享) architecture.** Data is partitioned across many independent database nodes; each node owns its slice (accounts by customer-ID hash, by branch, by account-number range) and there is no shared storage. GoldenDB, OceanBase and GaussDB are all shared-nothing engines; the core application is stateless so any node can serve any shard.
- **Microservices.** The monolithic core is decomposed into services — customer, account, deposit, loan, limit, interest, ledger, payments — each with its own data domain and API contract, communicating over the bank's service bus (often built on domestic middleware or open source; see 7.2). This mirrors the composable-core thinking in the West ([core_banking_systems_guide.md](core_banking_systems_guide.md) Section 6) but is pursued with far more vertical integration.
- **单元化 (cell-based / "unit-isation") architecture.** The Alibaba-derived pattern in which the system is divided into **cells (单元)** — each cell is a complete, self-contained slice of the application + data (a "mini-bank" that can serve any request for the customers whose data it holds). Traffic is routed to the cell owning the data (routing by customer ID); cells are deployed across data centres, enabling **异地多活 (geo-distributed multi-active)**: multiple data centres active simultaneously, with cells migrating between sites for DR. This is the pattern that lets a Chinese bank run two active data centres instead of the West's active-passive model.
- **Sharding (分库分表).** Horizontal partitioning of account tables across databases — the data-level expression of shared-nothing; sharding frameworks and the distributed databases' native sharding (including table-group/partition-group optimisations in OceanBase/GoldenDB that keep related rows co-located to avoid cross-shard transactions) are core engineering.
- **Event-driven integration.** Account events (postings, status changes) are published as events/streams to feed the ledger, risk, reporting and channels asynchronously — decoupling the hot path from analytics. (This connects to the event-driven architecture discussed in the repo's event-streaming guides.)
- **双活 / dual-active data centres.** RTO/RPO targets at the minutes/seconds level; the 单元化 design makes dual-active feasible for the core itself, not just the channels.
- **Resilience engineering.** Chaos engineering (Netflix-Chaos-Monkey-style drills, widely adopted in Chinese banks as 混沌工程), cell-level fault isolation, graceful degradation — because a hundred-thousand-TPS system cannot fail as a monolith.

The result is a core whose *scaling unit is the cell* and whose *availability unit is the data centre* — a design language that most Western cores (still centralised or COTS) have not adopted at the same depth.

### 5.4 The Flagship Transformation Projects

Verified or widely reported flagship projects, roughly in scale order:

- **ICBC** — OceanBase adoption (2020, verified announcement); 200+ Oracle services → GaussDB, 3,000+ nodes, ~10x RTO reduction (verified vendor statement); ongoing mainframe-offload. The world's largest core-migration program by any measure.
- **PSBC (邮储银行)** — distributed core fully live on the Huawei stack (Huawei Tech magazine; verified via source snippet). Landmark: largest branch network in the world running a distributed core.
- **CITIC Bank** — GoldenDB (ZTE) co-development from 2014; core "heart transplant" completed; also early domestic-DB adoption in its credit-card core (with Sunline's involvement). CITIC's new-generation distributed core is widely referred to as 凌云 (Lingyun) in industry reporting (name not independently verified).
- **CCB / ABC / BOC** — new-generation in-house cores (Section 4.3) with verified GaussDB usage in core scenarios (credit, payment, clearing).
- **CMB** — self-developed distributed, cloud-native new core (Section 4.4).
- **Ping An Bank** — the FLEXCUBE-era core (2010) has been progressively supplemented by in-house distributed systems under the Ping An Group's technology umbrella.
- **City-bank distributed cores** — the mid-market wave: Bank of Nanjing, Bank of Dongguan and many others via **Sunline's distributed core** (GaussDB joint solution, 2021); Hubei Bank's new core went live on **Dameng (达梦)** in May 2019 (verified via industry reprint); and a stream of city/rural banks adopting GoldenDB, TiDB and TDSQL-based cores. (The term "Yunxiang" sometimes encountered in this context appears to be a conflation/variant and is not used here; the verified mid-market products are Sunline's, DCITS's and the database vendors' distributed cores.)
- **Huawei's role** — as platform provider across the flagship list: GaussDB, Kunpeng/Taishan servers, 华为云 financial zones, and joint solutions with Sunline and other app vendors. Huawei is effectively the national champion of the distributed-core *platform* layer.

### 5.5 Timelines and the State of Play by 2026

- **The transformation wave runs ~2014–2030.** Digital banks first (2014–2015), joint-stock banks (2017–2020), Big Four (2019–2026+), and the rural tier via vendor BaaS (2021+).
- **双轨运行 (dual-core parallel run).** The standard cutover model: the legacy core (mainframe/NOVA-era or Oracle-based) and the new distributed core **run in parallel**, with accounts migrated in waves and the two reconciled continuously, until the legacy can be decommissioned. Big-bank dual-run periods last years; some systems (e.g., legacy deposits at a Big Four bank) may run dual-track well into the late 2020s.
- **By 2026:** every major Chinese bank has a distributed core in production for at least its new-generation products and a defined migration path for legacy; the mainframe estates are shrinking (ICBC/CCB have publicly signalled mainframe reduction); the database layer of new builds is domestic; and the industry has moved from "should we" to "how fast can we." The constraint is no longer technology but the sheer size of the legacy migration — which is why dual-run, not big-bang, is the universal pattern.

---

## 6. The Digital Banks' Cores

### 6.1 WeBank (微众银行): The Cloud-Native Showcase

**Profile.** China's first digital bank, founded **2014** in Shenzhen, backed by Tencent (with other shareholders including Baiye/百业源 and Li Ye/立业); began operations in 2015. No physical branches; customers served entirely through the WeChat ecosystem (WeBank's flagship product, 微粒贷 consumer lending, lives inside WeChat) and its own apps.

**Technology.** WeBank is the reference implementation of the fully distributed, cloud-native bank core:

- **"World's first next-generation core banking system based on a fully distributed architecture, designed in-house using commodity servers and open-source technologies"** (verified vendor/bank statement, 2024). No mainframe, no Oracle, no foreign commercial stack — the entire bank runs on x86 commodity hardware and open-source software, with a self-developed distributed core.
- **The "ABCD" strategy** — AI, Blockchain, Cloud, Big Data — the bank's own framing of its technology pillars (showcased internationally, e.g., at Singapore's FinTech Festival 2019).
- **Open source as product.** WeBank open-sourced major platforms: **FATE (Federated AI Technology Enabler)** — China's first enterprise-grade federated-learning platform, developed in-house from **2018** (verified); **Apache Linkis** (data-middleware, top-level Apache project); **Apache InLong** (data integration); plus DSS and others under the WeBankFinTech GitHub organisation (verified). WeBank's open-source portfolio is a deliberate soft-power strategy: the bank exports its architecture by exporting code.
- **Scale and economics.** WeBank serves **400+ million customers** (widely cited; verified via third-party summaries) with a famously tiny headcount (~3,000) — the cost-to-income and cost-per-customer ratios that make Western digital banks envious. Peak processing capacity is reported in the region of 100,000+ TPS (reported; see claims table).

**The showcase effect.** WeBank is China's (and arguably the world's) most-cited proof that a top-tier bank can run on a fully distributed, open-source, commodity-stack core — the model every 去IOE and 信创 program cites. It also exports its platform: **Fusion Bank** (Hong Kong's Tencent-backed virtual bank) launched a **new-generation core powered by WeBank technology in October 2024** (verified: product-development time halved, projected 53% reduction in non-human IT costs over three years) — WeBank's core as a commercial product outside the mainland.

### 6.2 MYbank (网商银行): The 310 Model at Scale

**Profile.** Founded **2015** in Hangzhou, wholly owned by **Ant Group**; China's second digital bank, focused on micro, small and medium enterprises (MSMEs) and rural customers. No branches; runs on Ant's cloud (Alibaba Cloud/Ant technology).

**Technology.** MYbank's core is built on **Ant Group's full stack: OceanBase database, SOFAStack middleware, and the distributed architecture Alibaba/Ant developed for Alipay**. The core is a distributed, cloud-native design where the *credit decisioning* — not just account processing — is the crown jewel:

- **The "310 model" (310模式):** a business loan takes **3 minutes to apply** (mobile), **1 second to approve** (automated), with **0 human intervention** (verified across multiple sources, including MYbank's own releases and The Asian Banker). This is the canonical statement of an AI-native credit core: application, risk-scoring, pricing and disbursement are fully automated, with hundreds of thousands of SME loans originated with near-zero manual touch.
- **Scale.** Tens of millions of MSME customers; by 2022 MYbank reported US$2.3 billion in revenue from SMEs with ~19% ROE (The Asian Banker; verified via source). Its AI-based risk models score businesses using transaction data, supply-chain data and alternative data — the "big-tech lending model" that Western regulators and banks study.
- **OceanBase is the shared DNA.** MYbank and Ant run on OceanBase; the same database is now sold to banks (ICBC, Bank of Nanjing, Zheshang Bank and others) — so Ant's banking technology competes with and supplies the incumbent banks simultaneously.

### 6.3 aiBank (百信银行) and XWBank (新网银行)

- **aiBank (百信银行).** China's first **direct-bank joint venture** between a bank and an internet company — **CITIC Bank + Baidu**, approved in **2017**. A "direct bank" (直销银行) subsidiary model: aiBank is a digital-only entity majority-owned by CITIC Bank, using Baidu's traffic and AI. Its core is a cloud-native distributed build (CITIC Bank's technology lineage, including the GoldenDB ecosystem) serving consumer and SME digital products.
- **XWBank (新网银行).** Founded **2016** in Chengdu, backed by **New Hope Group (新希望)** and **Xiaomi (小米)** among others. China's first digital bank backed primarily by industrial/mobile-ecosystem capital; focuses on consumer credit and SME lending at scale, with a distributed, cloud-native core and heavy reliance on AI risk models. XWBank is known for operating with a very lean cost base and for its "technology as a service" partnerships with smaller financial institutions.

### 6.4 The Digital Bank Core Characteristics

The digital banks' cores define the blueprint the incumbents are now copying:

- **Cloud-native by birth.** Everything runs on elastic cloud infrastructure (private or vendor cloud); no data centre of their own in the mainframe sense; scale-out is the only scaling mode.
- **API-first / open banking.** Products are exposed as APIs (WeBank's open platform, aiBank's API banking); partners and embedded-finance channels are first-class, not afterthoughts.
- **Fully automated operations.** 310-style automation extends to account opening, KYC (e-verification), lending and collections; operating costs per account are an order of magnitude below branch banks.
- **SME/consumer niche focus.** The digital banks do not try to be universal banks; they concentrate on the underserved mass-market consumer and MSME segments — which is exactly why they could start from a blank-slate core without the legacy product set.
- **Data- and AI-driven cores.** The core and the data platform are inseparable: risk models, product rules and even interest-rate setting are computed, not configured — the "AI-native core" (10.2) in embryonic form.
- **Lower cost-to-income.** WeBank and MYbank consistently post cost-to-income ratios that are a fraction of the incumbents' — the business case for the whole distributed-core movement. WeBank's cost-per-customer is famously reported in single-digit RMB (versus double-digit-to-hundreds of RMB for branch banks) and its headcount-to-customer ratio (a few thousand staff serving hundreds of millions of customers) is the most cited metric in Chinese digital-bank presentations.

### 6.5 Yillion and Suning: The Second Wave of Digital Banks

Two further digital banks round out the category (the task brief names four; these two complete the picture):

- **Yillion Bank (亿联银行)** — founded **2017** in Changchun, backed by Zhongfa Group (中发集团) and **Meituan** (美团的 holding came later); a "digital bank + ecosystem partner" model, serving consumers and SMEs through platform channels (Meituan's merchant ecosystem being the flagship integration). Its core is a distributed, cloud-native build — the pattern set by WeBank/MYbank applied to a smaller, partnership-driven institution.
- **Suning Bank (苏宁银行, now 苏商银行)** — founded **2017** in Nanjing, originally backed by the Suning retail group; serves consumer and micro-business customers through e-commerce and open-banking channels. (Suning Bank renamed to 苏商 Bank in the early 2020s after the Suning group's restructuring; treat the rename detail as reported.)

The second-wave lesson: once the WeBank/MYbank template existed, the marginal cost of launching a digital bank in China dropped dramatically — a new entrant could assemble a distributed core from domestic components (vendors, databases, cloud) in well under a year, which is precisely the pattern GLDB later repeated in Singapore (Section 9).

---

## 7. The Technology Stack: Databases, Middleware, Chips, OS, and 信创

### 7.1 The Database Landscape

The database is the strategic battleground of Chinese core banking: the layer where Oracle's 30-year dominance is being dismantled and where the 信创 mandate is most concrete. The landscape:

| Database | Owner / origin | Type | Banking footprint (verified or widely reported) |
|---|---|---|---|
| **Oracle** | US | Commercial RDBMS | The legacy standard across Chinese banking (non-mainframe systems); the explicit target of 去IOE; being replaced in new builds |
| **OceanBase** | Ant Group (蚂蚁集团) | Distributed, MySQL-compatible, shared-nothing; TPC-C world records | MYbank and all Ant core systems (verified); **ICBC** (2020, verified); Bank of Nanjing, Zheshang Bank, Changshu RCB, Hongta Bank (verified via OceanBase materials) |
| **GaussDB** | Huawei (华为) | Distributed, PostgreSQL-based; openGauss open-source sibling | **ICBC** (200+ services, 3,000+ nodes, verified); CCB, ABC, PSBC, BOC, BoCom, Huaxia scenarios (verified via Huawei/Baike materials); PSBC distributed core (verified) |
| **GoldenDB** | ZTE (中兴) | Distributed, shared-nothing, MySQL-compatible | **CITIC Bank** core "heart transplant" (verified); other banks' core migrations |
| **TiDB** | PingCAP (平凯星辰) | NewSQL, MySQL-compatible, open source | Bank deployments (scale-out OLTP in finance); strong in fintech/regional banks (widely reported) |
| **DM / 达梦** | Dameng (武汉达梦) | Commercial RDBMS, Oracle-compatible | **Hubei Bank** new core (May 2019, verified via industry reprint); state/enterprise installations |
| **openGauss** | Huawei (open source, 2020) | PostgreSQL-derived open-source DB | The open-source base of GaussDB deployments; adopted by banks for 信创 builds |
| **KingbaseES** | 人大金仓 (Renmin University-affiliated) | PostgreSQL-compatible | Government/state-sector and banking 信创 installations (widely reported) |
| **TDSQL** | Tencent (腾讯) | Distributed, MySQL-compatible | Reported at WeBank/Tencent ecosystem and several banks (secondary sources; see claims table) |

Selection logic in the 2020s: new core builds pair a **domestic distributed OLTP database** with the core application; the "GaussDB + Sunline core" and "GoldenDB + CITIC-style in-house core" are the two canonical 信创 stacks. Banks also run heterogeneous estates (different DBs per domain) to avoid single-vendor risk — a deliberate strategy given the stakes.

### 7.2 Middleware and Application Servers

- **TongWeb (东方通)** — the leading domestic J2EE application server (TongTech/东方通), the standard "TongWeb on Kunpeng on Kylin" stack element in 信创 deployments; widely used in bank channel and mid-tier systems.
- **Domestic middleware alternatives** — 中创中间件 (InforSuite), 金蝶天燕 (Apusic, Kingdee's application server), plus the internet giants' middleware stacks: **Ant SOFAStack** (service mesh, distributed transaction, registry — the middleware under MYbank and sold to banks), Huawei's microservice frameworks, and Tencent's equivalents.
- **Open source** — Spring Cloud/Dubbo/rocketmq-era stacks are pervasive; Chinese banks run massive Kafka/RocketMQ event pipelines and Kubernetes (or domestic distros of it). The pattern: **open-source foundations, domestic-hardened distributions** — the same relationship as openGauss/GaussDB.
- **Distributed transaction handling** — the hard problem in sharded cores (cross-shard transfers, account-to-GL postings) is solved with TCC/SAGA frameworks (SOFAStack's Seata lineage), local-message tables and eventual-consistency patterns — engineering the Western cores often still do with a single centralised database.

### 7.3 Chips and Servers: x86, Kunpeng, Hygon, Phytium

The hardware layer of 信创:

- **x86** — the incumbent commodity baseline (Intel/AMD); Alibaba's 去IOE moved *to* x86, and much of the distributed core estate still runs x86.
- **Kunpeng (鲲鹏)** — Huawei's ARM-based server CPU (Kunpeng 920 series); the flagship domestic server chip, paired with **Taishan (泰山) servers** and GaussDB in the canonical Huawei financial stack (verified in the Sunline joint solution and PSBC build).
- **Hygon (海光)** — Hygon Information (海光信息, SHSE 688041), x86-compatible server CPUs (originally derived from AMD IP); popular in banking/state-cloud deployments for software compatibility.
- **Phytium (飞腾)** — Phytium Technology (天津飞腾), ARM-based CPUs, CEC-affiliated (same parent as GienTech); strong in state and financial 信创 builds.
- **The server vendors** — Inspur (浪潮), H3C (新华三), Lenovo (联想) and Huawei's Taishan dominate the domestic server supply, all offering 信创 lines.

Practical note: the chip layer is where banking software must be *portable* — core vendors now test on x86, Kunpeng and Phytium simultaneously, and the 信创 stack assumes a "write once, run on domestic chips" requirement that shapes Java/container-based architecture choices.

### 7.4 Operating Systems: Kylin, UOS, openEuler

- **Kylin (银河麒麟)** — the NeoKylin/KylinSoft (麒麟软件, CEC-affiliated) Linux distribution; the default domestic server OS in financial 信创 builds (often cited as the single most widely deployed domestic OS in banks' core-adjacent servers).
- **UOS (统信UOS)** — UnionTech (统信软件) desktop/server OS (Debian-derived); the domestic desktop standard, increasingly used in bank office/back-office estates.
- **openEuler (欧拉)** — Huawei's open-source server OS; the base for many domestic cloud and GaussDB deployments.
- The pattern mirrors middleware: **Linux-kernel-based domestic distributions, hardened for finance**, replacing Red Hat/Windows in 信创 estates.

### 7.5 信创 (Xinchuang): The Domestic Substitution Framework

**信创 (信息技术应用创新, "IT application innovation")** is the national programme for replacing foreign IT in China's critical industries with domestic technology — the policy umbrella under which 去IOE, domestic databases, Kunpeng chips, Kylin OS and the domestic vendor wave all operate. Key features:

- **The "2+8" industry scope.** The standard framing: **2 = party and government organs (党政)** plus **8 key industries** — **finance (金融)**, telecom, energy, power, transportation, education, medical/healthcare, and aviation/aerospace (formulations vary slightly by source). **Finance is among the first wave of the 8** — banks are priority targets because they are both critical infrastructure and large IT spenders. (The "2+8" formulation is standard industry framing; see claims table.)
- **What banks must do.** Replace, in a phased manner, foreign **chips, servers, OS, middleware, databases and application software** in their IT estates with domestic alternatives; **new builds must be 信创-compliant**, and legacy systems must be migrated. Practical quotas and timelines are set by the regulator and the 金融信创 ecosystem (the FinTech Innovation labs/生态实验室 that coordinate bank pilots and certify stacks); the widely reported trajectory: pilots at major banks from ~2020, expansion through 2023–2025, and **full domestic substitution targeted by ~2027** for key industries (reported target; see claims table). Foreign-invested banks and JVs are subject to the same frameworks for their China entities (8.3).
- **Impact on core systems.** 信创 is the reason the distributed-core transformation and the database substitution are the *same* project: a bank cannot rebuild its core on Oracle in 2025. It also guarantees the domestic vendors' market (3.9), creates the "GaussDB+Sunline", "GoldenDB+in-house", "Kunpeng+Kylin+TongWeb" canonical stacks, and adds a certification/compliance dimension (产品名录/product catalogues, 测评/certification) to every core RFP.
- **For Western observers:** 信创 is not a rumour or a slogan — it is a procurement framework with catalogues, certification bodies, pilots and deadlines (the AmCham China 2026 White Paper discusses the xinchuang product-list management regime; verified). Any foreign technology vendor selling to Chinese banks must assume a hard ceiling on new deployments, and any foreign bank's China technology strategy must assume domestic stacks for new builds.
---

## 8. China vs. the West: Comparison and Implications

### 8.1 The Comparison Table

| Dimension | China | West (Europe/US/Singapore mainstream) |
|---|---|---|
| **Scale** | Billions of customers (Big Four alone); hundreds of millions of daily transactions; 10k–100k+ peak TPS | Millions per bank; thousands to low-tens-of-thousands TPS |
| **Core architecture** | Distributed, sharded, cell-based (单元化); cloud-native for new builds; dual-active data centres | Centralised mainframe or COTS cores; active-passive DR; SaaS cores emerging for challengers |
| **Vendor landscape** | Domestic product vendors (Sunline, DCITS, GienTech, Yusys) + in-house development at the top | Temenos, FLEXCUBE, Thought Machine, Mambu, Avaloq, Finacle, BaNCS (international COTS/SaaS) |
| **Build vs. buy** | Buy for mid/low tier; **self-develop (自主研发) at the top** | Buy or SaaS at nearly every tier; build only in bespoke niches |
| **Databases** | Oracle legacy → OceanBase/GaussDB/GoldenDB/TiDB/DM domestic wave | Oracle/DB2/SQL Server legacy; PostgreSQL/cloud-native for new builds |
| **Cloud** | Private/domestic cloud (Huawei Cloud, Alibaba Cloud, Tencent Cloud; 金融云 zones); foreign public cloud effectively barred for core | AWS/Azure/GCP, often with regulatory scrutiny; cloud-native cores (Thought Machine on AWS) |
| **Regulatory driver** | **信创/自主可控**: domestic substitution with deadlines; PBOC/NFRA tech plans; data localisation (数据安全法, PIPL) | MAS TRM/FEAT, ECB/SSM outsourcing & cloud guidance, DORA, BCBS 239 — resilience and risk-focused, not domestic-stack mandates |
| **Modernisation driver** | Self-reliance + scale ceiling + policy mandate | Digital transformation, cost, legacy risk; market-driven |
| **Distributed-core maturity** | World-leading: the reference implementations are Chinese (WeBank, CITIC/GoldenDB, PSBC/Huawei) | Maturing via SaaS cores and API banking; fewer at the extreme scale |
| **Peak event** | 双11/Spring Festival spikes engineered for | Black Friday/end-of-year; far smaller |

### 8.2 What Western Banks Can Learn

The Chinese experience is the world's largest laboratory for distributed core banking. Transferable lessons:

1. **Distributed cores are proven at extreme scale.** The "you can't shard a core" argument is empirically dead: WeBank (400M+ customers), PSBC (~40,000 outlets), ICBC (hundreds of millions of daily transactions) all run distributed cores in production. A Western bank's reluctance is about risk appetite, not feasibility.
2. **The cell/单元化 pattern is the key enabler** of both scale and 双活 availability — the Chinese answer to the active-passive DR problem. Banks with 24/7 mobile expectations (i.e., all of them) can adopt cell-based routing without adopting the full Chinese stack.
3. **The database is the migration's critical path.** The Chinese "换心" (heart-transplant) experience shows that core modernisation is a database-migration problem first, an application problem second; staged, shard-by-shard migration with dual-run (双轨运行) beats big-bang.
4. **Cloud-native economics are real** — WeBank/MYbank cost-to-income and cost-per-customer ratios are the business case for greenfield cores, relevant to any digital-bank build in Singapore or Europe.
5. **Open source as a strategic asset** — WeBank's FATE/Linkis/InLong portfolio shows a bank can be a major open-source contributor and exporter of architecture; Western banks are consumers, not producers, of core-adjacent open source.
6. **Compliance-by-design stacks** — the Chinese 信创 stack discipline (certified combinations of chip/OS/DB/middleware) is a model for banks that must meet outsourcing and supply-chain scrutiny (DORA, MAS TRM), even if the specifics differ.

### 8.3 Foreign Banks Operating in China

For a foreign bank (e.g., a European or US bank with a locally incorporated China entity, or a Singapore bank expanding into China), the Chinese core landscape imposes specific constraints:

- **Data residency (数据本地化).** Under the 数据安全法 and PIPL, important data and personal information of Chinese customers must be stored in China; cross-border transfer requires security assessments/standard contracts via the CAC process. A foreign bank's China core and data platform must be physically and legally local — which usually means a locally licensed entity running its own China instance, not a hub-and-spoke off a regional core.
- **Xinchuang compliance for the China entity.** The 信创 framework applies to all banking institutions in China, including foreign-invested banks (in practice with phasing and scope negotiated with the regulator, but the direction is unambiguous). New systems in China should assume domestic stack components; foreign banks typically maintain grandfathered legacy (Oracle, etc.) but cannot assume approval for new foreign-stack core builds.
- **Joint-venture banks.** The JV route (e.g., aiBank = CITIC + Baidu; earlier bank-insurance and bank-bank JVs) is how foreign groups access the Chinese market with local partners; the technology in these vehicles is overwhelmingly the local partner's stack.
- **The foreign-branch niche.** Foreign banks' China branches and overseas subsidiaries of Chinese banks are where international cores (Temenos T24, FLEXCUBE) still appear — the legacy of the pre-信创 era. A Western group's China entity is likely to run a global core instance (e.g., the group's own platform) for its corporate business while integrating with CNAPS/CIPS and UnionPay for local clearing.
- **Practical guidance:** treat the China entity as a separate technology jurisdiction; budget for local data centres/domestic cloud, Chinese regulatory reporting (PBOC/NFRA/SAFE) built into the core, and a 信创-compatible roadmap; and expect the local regulator to review core-system resilience and outsourcing with a specificity the home regulator may not match.

---

## 9. Singapore and Asia Relevance

### 9.1 GLDB and the Singapore–China Corridor

The most direct Singapore link in this guide's ecosystem is **Green Link Digital Bank (GLDB)** — the MAS-licensed digital wholesale bank built by the **Greenland Group–Linklogis consortium** (Greenland Financial Holdings + Linklogis HK + Beijing Co-operative Equity Investment Fund Management), one of the four winners of MAS's December 2020 digital bank round. GLDB's stack is the China-Singapore synthesis: **Temenos core banking (core, payments, data hub) on Huawei Cloud**, implemented by Greenland Digitech in ~11 months, live June 2022, focused on MSME supply-chain finance across the China–Singapore trade corridor. GLDB shows the Chinese stack entering Singapore *through* the corridor: Huawei Cloud as the regulated cloud, Temenos as the packaged core, Chinese capital and supply-chain fintech (Linklogis) as the business engine. Full detail in [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md).

### 9.2 Chinese Banks in Singapore

Singapore is the ASEAN hub for Chinese banking:

- **BOC, ICBC, CMB and the rest of the Chinese majors operate substantial Singapore branches/subsidiaries** (BOC has a century-old Singapore presence; ICBC Singapore is its regional booking hub). These entities run **parent-group core systems** — the group's overseas core platforms (e.g., the FOVA-style overseas platforms of the big banks) — with Singapore-specific regulatory reporting to MAS and integration to MAS's payment rails (FAST, PayNow, MEPS+) and to CIPS/SWIFT for RMB flows.
- The technology consequence: Singapore branches of Chinese banks are *extensions of Chinese cores*, which means MAS-regulated operations sit on systems whose architecture, resilience and data-residency posture were designed for the Chinese parent's model — an interesting supervisory juxtaposition with MAS's TRM and data-residency expectations.
- Singapore is also where Chinese banking technology is **tested for export**: WeBank's tech-powered Fusion Bank is Hong Kong, but Singapore's digital-bank wave (GXS, MariBank, ANEXT, GLDB, Trust) includes Ant-backed **ANEXT Bank**, which runs on Ant Group's stack (OceanBase lineage) — the Chinese distributed core inside a MAS-licensed bank.

### 9.3 Chinese Fintech in Singapore: Ant, Tencent, Airwallex

- **Ant Group** — ANEXT Bank (2020 DWB licence winner, wholly owned by Ant) and the regional payments/merchant business; Ant's OceanBase and SOFAStack are sold globally, with Singapore as the regional base.
- **Tencent** — WeChat Pay cross-border and Tencent Cloud financial services; WeBank's technology arm promotes the distributed-core platform regionally (Fusion Bank is the flagship reference).
- **Airwallex** — the Australia/HK-founded, Chinese-co-founded cross-border payments fintech with Singapore as a key hub (see [airwallex_guide.md](airwallex_guide.md)); it connects China-origin trade flows to global banking rails without being a licensed bank itself.
- The pattern: Chinese fintech/banking tech treats Singapore as both a market and a **compliance-friendly launchpad** for the rest of Asia — MAS licensing is respected internationally, and the Chinese stacks (OceanBase, WeBank platform, Huawei Cloud) gain legitimacy by running inside MAS-regulated entities (GLDB, ANEXT).

### 9.4 The MAS Digital Bank Context

MAS's digital bank framework (2019–2020, two digital full bank + three digital wholesale bank licences, four awarded December 2020 plus Trust with a full licence) created the region's most visible testbed for new cores. The winners' stacks are a spectrum: **GXS** (Grab+Singtel) and **MariBank** (Sea) on cloud-native builds; **ANEXT** on Ant's distributed stack; **GLDB** on Temenos-on-Huawei-Cloud. For the purposes of this guide, the MAS round is the first time the Chinese distributed-core philosophy and the international packaged-core philosophy competed head-to-head in a single regulated market — see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) for the full treatment, including the honest caveat that none of the five digital banks has proven profitability yet.

### 9.5 Cross-Border: RMB Internationalization and CIPS

Chinese cores increasingly face cross-border requirements as the RMB internationalises:

- **CIPS (Cross-Border Interbank Payment System, 人民币跨境支付系统)** — launched **October 2015** by the PBOC (Phase 2 in 2018), China's own cross-border RMB clearing and settlement system, running alongside (and partially parallel to) SWIFT. Direct and indirect participants span Asia, Europe and beyond; CIPS is the payment-rail expression of RMB internationalisation. Banks with RMB ambitions — including Singapore banks and the Chinese banks' Singapore branches — integrate their payment systems to CIPS. See [financial_infrastructure_guide.md](financial_infrastructure_guide.md) for the FMI-level detail.
- **The digital-currency layer** — the e-CNY (digital yuan) pilot has progressed through 2020–2026, with banks integrating e-CNY wallets into their cores' settlement architecture (a topic for a future guide).
- **For core architects:** cross-border RMB flows mean multi-currency cores, CIPS connectivity, and compliance with both Chinese (data laws) and foreign (MAS) regimes — the same duality that shapes the Singapore branches in 9.2.

### 9.6 Learning Points for Singapore and Global Banks

1. **The Chinese distributed-core playbook is importable** — cell-based scaling, shared-nothing DBs, dual-active DCs — and MAS's TRM/BCBS-style regulation does not prohibit it; GLDB (Temenos+Huawei) and ANEXT (Ant stack) show both ends running under MAS.
2. **Data residency cuts both ways.** Singapore's data-residency posture and China's 数据安全法 create a two-sided compliance wall for corridor banks; the core must support **jurisdiction-local data domains** (the same sharding/cell logic that serves scale also serves data residency).
3. **Chinese vendors are now regional vendors.** Huawei Cloud (GLDB), Ant (ANEXT/OceanBase), WeBank technology (Fusion Bank) and Sunline (Southeast Asia core deals) are selling in Singapore and ASEAN; a regional bank's vendor shortlist now includes Chinese platforms with real reference implementations.
4. **The corridor is the business case.** China–ASEAN trade flows are the demand driver for corridor banks; the core systems that win are the ones that bridge CIPS/CNAPS and FAST/PayNet/MEPS+ natively.

---

## 10. The Future: 2026 and Beyond

### 10.1 Xinchuang 2.0: The Full Domestic Stack Mandate

The first 信创 wave (2020–2025) proved the domestic stack in production; the second wave is about **completeness and depth**:

- From "domestic databases for new builds" to **full-stack domestic: chips (Kunpeng/Hygon/Phytium) + OS (Kylin/UOS/openEuler) + middleware (TongWeb) + database (OceanBase/GaussDB/GoldenDB) + application** — the "full-stack 信创" requirement tightening through 2026–2027 (reported targets for key industries).
- **Legacy mainframe retirement deadlines** — the big banks' mainframe-offload programs (ICBC/CCB publicly committed to reducing mainframe dependence) will reach their endgame; the remaining hard cases (the largest legacy deposit/GL systems) will be the last to migrate, with dual-run periods extending into the late 2020s.
- **Certification and catalogues** harden: procurement through 信创 product lists, third-party evaluation of domestic stacks, and regulator-visible migration roadmaps become the norm (the AmCham 2026 white paper's discussion of 信创清单 management confirms the machinery is real).
- Expect **信创 2.0 to be judged on operational metrics** — RTO/RPO, TPS at peak, defect rates — not just "runs on domestic stack." The era of the pilot is over.

### 10.2 AI in the Chinese Core

The Chinese cores are becoming **AI-native**:

- **The digital banks already are.** MYbank's 310 model is an AI credit core; WeBank's FATE federated learning and ABCD strategy put AI in the core's bloodstream. The next step is **LLM-native banking**: Chinese banks are deploying LLMs (domestic models — DeepSeek, Qwen, GLM, and bank-tuned variants) for customer service, contract review, risk analysis, code generation and internal ops at a scale and speed that Western banks are also pursuing.
- **LLM integration into core workflows** — e.g., natural-language product configuration, AI-assisted core testing, anomaly detection over core event streams, and "digital employees" in operations — is the 2026+ theme; the repo's LLM guides (see the ai_llm series) cover the underlying technology.
- **The AI-native core vision:** core systems where rules, risk models and even product definitions are model-driven rather than code-driven — the direction MYbank's credit core already exemplifies, and where the Chinese banks' combined advantage (data scale + domestic AI models + policy support for AI adoption) is strongest.

### 10.3 Cloud-Native Maturation

- **Financial clouds mature into "core-grade."** Huawei Cloud (with financial industry zones), Alibaba Cloud/Ant's financial cloud, and Tencent Cloud all now host core-adjacent and increasingly core workloads; the regulatory 金融云 framework (PBOC-supervised) has normalised banks running regulated workloads on domestic clouds.
- **Distributed cores get platformised.** The vendor products (Sunline distributed core, DCITS Sm@rtOneBank, GienTech ORIGIEN) converge on a common shape: microservices + domestic DB + cell deployment + DevOps toolchains, delivered as repeatable platforms rather than one-off projects — dropping the cost of the mid-market transformation.
- **The remaining frontier** is the **very top of the scale curve**: running the biggest legacy book (hundreds of millions of accounts on one mainframe) through a distributed cutover without a glitch — the technical drama of 2026–2030.

### 10.4 Internationalization of Chinese Banking Technology

China's core-banking technology is becoming an export:

- **OceanBase** — global expansion as a standalone database company (400+ customers globally per its own materials, TPC-C records as marketing, Singapore as a regional base) — the most mature exporter.
- **WeBank technology** — the Fusion Bank core (2024) is the reference for selling the WeBank distributed-core platform to overseas banks; expect more regional virtual banks to buy it.
- **Huawei** — GaussDB/openGauss and the financial stack sold across Asia, Africa and the Middle East (Huawei's financial-services business is already global); GLDB's Huawei Cloud (Singapore) is the regulated-market reference.
- **Ant Group** — ANEXT (Singapore), OceanBase, and the broader "tech out" strategy under Ant International.
- **The pattern to watch:** Chinese stacks are winning where (a) scale is the problem (emerging markets' fast-growing banks), (b) price matters (vs. Western COTS), and (c) the buyer has no regulatory objection to Chinese technology. Where those three hold — much of ASEAN, the Middle East, Africa — the 2030s could see the Chinese core as a first-class global option, the mirror image of how Western cores once entered China.

### 10.5 Trends Summary

| Trend | Direction |
|---|---|
| 去IOE → 信创 | From "remove IBM/Oracle/EMC" to full-stack domestic substitution with deadlines and certification |
| Distributed cores | From digital-bank novelty (2014) to default architecture (2026); migration wave at the Big Four continues through 2030 |
| Databases | Oracle legacy shrinking; OceanBase/GaussDB/GoldenDB/TiDB/DM split the new-build market |
| AI | From AI-assisted ops to AI-native core workflows and model-driven products |
| Cloud | Regulated domestic financial clouds become core-grade |
| Export | OceanBase, WeBank platform, Huawei stack and Ant sell abroad; Singapore is the launchpad |
| Global learning | Western/Singapore banks adopt the cell/双活/shared-nothing playbook on their own terms |

---

## 11. Glossary

- **Big Four** — the four state-owned commercial banks: ICBC 工商银行, CCB 建设银行, ABC 农业银行, BOC 中国银行; often extended to the "Big Five" with BoCom 交通银行 and to six with PSBC 邮储银行.
- **Joint-stock commercial bank (股份制商业银行)** — the twelve nationally licensed share-holding banks (CMB, SPDB, CITIC, CEB, CMBC, CIB, Ping An, Huaxia, CGB, Zheshang, Bohai, Hengfeng).
- **City commercial bank (城市商业银行)** — ~125 locally headquartered banks (Bank of Beijing, Bank of Shanghai, Bank of Jiangsu, Bank of Ningbo, Bank of Nanjing...).
- **Rural commercial bank / rural credit cooperative (农村商业银行/农信社)** — thousands of small rural institutions; the long tail of Chinese banking.
- **Digital/online bank (互联网银行)** — branchless internet-only banks: WeBank, MYbank, XWBank, aiBank, Yillion 亿联, Suning.
- **WeBank (微众银行)** — Tencent-backed Shenzhen digital bank (2014); the reference fully distributed, open-source, cloud-native core.
- **MYbank (网商银行)** — Ant Group's Hangzhou digital bank (2015); the "310 model" SME lender on the OceanBase/SOFAStack stack.
- **aiBank (百信银行)** — CITIC Bank + Baidu direct-bank JV (2017).
- **XWBank (新网银行)** — New Hope + Xiaomi-backed Chengdu digital bank (2016).
- **去IOE (de-IOE)** — "remove IBM, Oracle, EMC": the movement to replace foreign platform technology with domestic/open alternatives; coined in Alibaba's 2010s transformation.
- **分布式核心 (distributed core)** — a core banking system on shared-nothing, sharded, microservices architecture; China's default new-build pattern.
- **自主研发 (self-development / independent R&D)** — the big banks' practice of building and owning their cores in-house (自主可控, "independent and controllable").
- **信创 (xinchuang)** — 信息技术应用创新, "IT application innovation": the national domestic-substitution programme.
- **"2+8"** — the 信创 scope: party/government (2) plus eight key industries including finance.
- **OceanBase** — Ant Group's distributed MySQL-compatible database; TPC-C record holder; used at MYbank, ICBC, Bank of Nanjing, Zheshang Bank.
- **GaussDB** — Huawei's distributed PostgreSQL-based database; openGauss is its open-source sibling; used at ICBC, CCB, ABC, PSBC, BOC, BoCom, Huaxia.
- **GoldenDB** — ZTE's distributed shared-nothing database, co-developed with CITIC Bank; powers CITIC's core.
- **TiDB** — PingCAP's open-source NewSQL (MySQL-compatible) distributed database.
- **DM (达梦)** — Dameng, the Wuhan commercial RDBMS (Oracle-compatible); Hubei Bank's new core runs on it.
- **openGauss** — Huawei's open-source PostgreSQL-derived database (2020).
- **KingbaseES** — 人大金仓's PostgreSQL-compatible database (Renmin University lineage).
- **Kunpeng (鲲鹏)** — Huawei's ARM server CPU; with Taishan 泰山 servers, the flagship domestic hardware.
- **Hygon (海光)** — x86-compatible domestic server CPU maker (SHSE 688041).
- **Phytium (飞腾)** — CEC-affiliated ARM CPU maker.
- **Kylin (麒麟)** — CEC-affiliated domestic Linux (银河麒麟); the default 信创 server OS.
- **UOS (统信)** — UnionTech's Debian-derived domestic OS (desktop/server).
- **数据大集中 (data concentration)** — the 1990s–2000s centralisation of Chinese bank data from provincial centres into national data centres.
- **Dual-active (双活)** — two (or more) data centres active simultaneously; enabled by 单元化 cell architecture.
- **单元化 (cell-based/unit-isation)** — the Alibaba-derived pattern of dividing a distributed system into self-contained cells owning slices of data and traffic, deployable across data centres.
- **NOVA / FOVA** — widely cited names for ICBC's domestic new-generation core (NOVA) and its overseas core (FOVA); reported in industry literature, not independently verified in this guide.
- **双轨运行 (dual-core parallel run)** — the migration pattern where legacy and new cores run in parallel with reconciled data until cutover completes.
- **CIPS** — Cross-Border Interbank Payment System, the PBOC's RMB cross-border clearing system (2015).
- **Data localization (数据本地化)** — the 数据安全法/PIPL requirement that important data and Chinese personal information be stored in China and cross-border transfers be authorised.
- **CNAPS** — China National Advanced Payment System, the PBOC's domestic payment rails (HVPS, BEPS); see financial_infrastructure_guide.md.
- **NFRA / CBIRC** — National Financial Regulatory Administration (2023, absorbing the CBIRC 银保监会), China's banking prudential regulator.
- **PIPL** — Personal Information Protection Law 个人信息保护法 (2021).
- **310 model (310模式)** — MYbank's "3 minutes to apply, 1 second to approve, 0 human intervention" SME lending model.

---

## 12. Claims-Status Table

Per repo convention (see [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) Section 5), China-specific facts are graded below. **Verified** = retrieved from a source in this session (web search results); **Reported** = widely cited in industry literature but not independently verified here; **Vendor claim** = company marketing; **Unverified** = could not be checked.

| Claim | Status | Notes / source type |
|---|---|---|
| Ping An Bank selected Oracle FLEXCUBE (March 2010, six-month evaluation; Ping An Technology as SI) and went live | **Verified** | Press release (przoom, Oracle FLEXCUBE) retrieved via search |
| China Dasheng Bank FLEXCUBE implementation | **Verified (secondary)** | LinkedIn partner (Wecodee) post |
| FLEXCUBE was not the mainstream core of Chinese banks; international cores limited to foreign branches/overseas subsidiaries | **Reported** (assessment) | Industry understanding; consistent with verified installs |
| Temenos T24 at CMB New York subsidiary (chosen over Misys/Fiserv) | **Verified** | Finextra press article |
| Temenos T24 at Swedbank Shanghai office (2007) | **Verified** | Global Custodian article |
| Temenos Shanghai entity active 2026 | **Verified** | Liepin company listing |
| Agricultural Bank of China listed as T24 user | **Unverified** | AppsRunTheWorld directory entry only (possibly overseas subsidiary) |
| SAP for Banking limited adoption in China | **Reported** | No significant verified installs found; consistent with market knowledge |
| ICBC core names NOVA (domestic new-gen core) and FOVA (overseas) | **Unverified** | Widely referenced in industry literature; not retrievable via this session's search backend; treat as reported |
| ICBC data concentration completed ~2002–2004 | **Reported** | Industry literature; specific years vary by account |
| 长亮科技 English names: Changliang (pinyin) / Sunline (international brand); "Longshine" is a different company (朗新科技) | **Verified** | ZoomInfo (Changliang), Sunline investor/company pages (300348); EMIS/Devex for LongShine (300682) |
| Sunline core at Bank of Nanjing, Bank of Dongguan and many regional banks | **Verified (secondary)** | Market-analysis article (victorleungtw.com, Apr 2025) |
| Sunline–Huawei distributed core joint solution on GaussDB + Taishan (Sept 2021) | **Verified** | EqualOcean briefing |
| Sunline involvement in CITIC Bank credit-card domestic-DB adoption | **Verified (secondary)** | Sunline company article |
| DCITS (神州信息, 000555): Sm@rtOneBank, ~20,700 staff, 300+ products | **Verified** | en.dcits.com |
| GienTech = CEC brand launched Jan 2021, formerly Pactera; ORIGIEN platform | **Verified** | China Daily (Jan 2021), GienTech press/HK launch |
| Pactera "serves 95%+ of banks in China" | **Vendor claim** | Pactera website |
| Yusys (宇信科技) banking core + financial cloud | **Verified (secondary)** | CB Insights, company site, market article |
| Teamax = 天阳科技 English name | **Reported** | Aggregator profile; Chinese name unambiguous |
| ZTE GoldenDB co-developed with CITIC Bank from 2014; CITIC core "heart transplant" | **Verified** | Industry article (min.news) + ZTE solution pages |
| Hubei Bank new core on Dameng DB (May 2019) | **Verified (secondary)** | Industry reprint (codetd) |
| ICBC adopts OceanBase (announced Sept 2020) | **Verified** | CnTechPost + OceanBase customer story |
| ICBC migrates 200+ Oracle services to GaussDB, 3,000+ nodes, RTO ~10x better | **Verified** | Huawei Cloud LinkedIn/vendor statement |
| GaussDB core scenarios at ICBC, PSBC, CCB, ABC, BOC, BoCom, Huaxia | **Verified** | Huawei/Baidu-Baike materials; white paper (2025) |
| PSBC distributed core fully live on Huawei stack | **Verified** | Huawei Tech magazine (via search snippet) |
| OceanBase customers: MYbank, Ant core, Bank of Nanjing, Zheshang, Changshu RCB, Hongta Bank | **Verified** | OceanBase/Alibaba Cloud materials |
| WeBank: first fully distributed next-gen core, in-house, commodity servers + open source | **Verified** | PRNewswire/company statement (2024) |
| WeBank FATE (2018, first enterprise-grade federated-learning platform) | **Verified** | Tokyo FinTech/Medium + company statements |
| WeBank open source: Linkis, InLong, DSS (WeBankFinTech org) | **Verified** | GitHub org + PRNewswire |
| WeBank ~400M customers | **Verified (secondary)** | Third-party summaries (yellow.com) |
| Fusion Bank (HK) new-gen core powered by WeBank tech (launched Oct 2024; dev time halved; IT cost -53% projected) | **Verified** | PRNewswire (Nov 2024) + China Daily (Feb 2025) |
| MYbank 2015; "310 model" | **Verified** | Businesswire, The Asian Banker, Taiwan News |
| MYbank 2022: $2.3bn SME revenue, ~19% ROE | **Verified** | The Asian Banker |
| aiBank: CITIC + Baidu JV (2017); XWBank: New Hope + Xiaomi (2016) | **Reported** | Well-documented corporate facts; not re-verified this session |
| 双11 Alipay 256,000 TPS (2017 world record) | **Verified (secondary)** | Blog citation of Alibaba's record |
| 双11 Alipay ~583,000 TPS (2020) | **Reported** | Widely quoted Alibaba/Ant figure; not directly verified |
| WeBank peak capacity ~100k+ TPS | **Reported** | Company/industry reporting |
| ICBC ~700M personal customers; Big Four serve billions combined | **Reported** | Annual-report-era figures, approximations |
| China's 21 systemically important banks (Dec 2020); Big Four in top group | **Verified (secondary)** | LinkedIn summary of PBOC/NFRA announcement |
| 信创 "2+8" industries with finance among the 8 | **Reported** | Standard industry framing; formulations vary |
| 信创 full substitution ~2027 target for key industries | **Reported** | Widely reported policy trajectory |
| 信创 product-list/catalogue machinery (relevant to foreign banks) | **Verified** | AmCham China 2026 White Paper (PDF, via search) |
| 数据安全法 (Sept 2021) / PIPL (Nov 2021) data localisation | **Verified** | Well-established law facts (statute dates are public record) |
| CIPS launched Oct 2015, Phase 2 2018 | **Reported** | See financial_infrastructure_guide.md for the FMI treatment |
| GLDB: Temenos core on Huawei Cloud, live June 2022, Greenland–Linklogis consortium | **Verified** | See green_link_digital_bank_guide.md (fully verified guide) |
| ANEXT Bank (Ant, DWB licence Dec 2020) runs Ant stack | **Reported** | Parent-company lineage; consistent with MAS award |
| CITIC new core "凌云 (Lingyun)" name | **Unverified** | Industry references; not confirmed this session |
| OceanBase TPC-C world records (2019+; 400+ global customers) | **Verified (secondary)** | OceanBase site; widely reported |
| "Yunxiang" as a distributed-core product name | **Not used** | Could not be verified; excluded from this guide |

---

## 13. References

Primary/retrievable sources used in this session (via web search):

1. Oracle — FLEXCUBE for Core Banking (oracle.com/cn and global pages); "China's Ping An Bank Goes Live on Oracle FLEXCUBE" press release (2010, przoom).
2. WeBank / WeBankFinTech — PRNewswire (Oct 2019 "WeBank: The World's Leading Digital Bank Decoded"; Nov 2024 Fusion Bank core launch); github.com/WeBankFinTech; webank.com.
3. MYbank — Businesswire (Oct 2020, 310 model); The Asian Banker (Jiang Hao interview); Taiwan News (Star Plan).
4. Temenos — Finextra (CMB New York T24); Global Custodian (Swedbank Shanghai T24, 2007); temenos.com/offices; Liepin (Temenos Shanghai).
5. Sunline/长亮科技 — sunline-i.com; ZoomInfo (Changliang Technology); moomoo/Gelonghui (300348); EqualOcean (Sunline–Huawei GaussDB distributed core, Sept 2021); Caishen (Sunline international).
6. DCITS — en.dcits.com (about, Sm@rtOneBank white paper).
7. GienTech — en.gientech.com (history, ORIGIEN launch); China Daily (CEC launches GienTech, Jan 2021); PRNewswire (GienTech APAC debut, Jul 2023).
8. Yusys — yusys.com.cn/en; CB Insights; LinkedIn.
9. Huawei — Huawei Cloud GaussDB statements (ICBC 200+ services, 3,000+ nodes); Huawei Central (GaussDB–ICBC award, Jan 2023); Huawei Cloud news (GaussDB maintenance white paper with ICBC/PSBC/BOC/BoCom/Huaxia, Sept 2025); Huawei Tech magazine (PSBC distributed core).
10. ZTE — zte.com.cn GoldenDB solution pages; industry articles on GoldenDB–CITIC core "heart transplant".
11. OceanBase — en.oceanbase.com (customer stories: ICBC; overview; TPC-C records); Alibaba Cloud ApsaraDB documentation (external customers incl. Bank of Nanjing, Zheshang); CnTechPost (ICBC–OceanBase, Sept 2020).
12. Industry analyses — "Core Banking Systems Market in China — Vendors, SIs, and Trends" (victorleungtw.com, Apr 2025, snippet via search); AmCham China 2026 White Paper (PDF, xinchuang list regime).
13. Repo sibling guides — [core_banking_systems_guide.md](core_banking_systems_guide.md), [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md), [temenos_guide.md](temenos_guide.md), [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md), [financial_infrastructure_guide.md](financial_infrastructure_guide.md), [airwallex_guide.md](airwallex_guide.md).

Where sources disagreed or a claim could not be verified, the guide says so inline and in the [claims-status table](#12-claims-status-table).

---

*End of guide. Companion reading: the umbrella [core_banking_systems_guide.md](core_banking_systems_guide.md) for the global discipline; [green_link_digital_bank_guide.md](green_link_digital_bank_guide.md) for the Singapore–China corridor case; [financial_infrastructure_guide.md](financial_infrastructure_guide.md) for CNAPS/CIPS and the payment rails.*
