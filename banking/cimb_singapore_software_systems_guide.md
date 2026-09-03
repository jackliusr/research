# CIMB: The Software Systems Landscape — A Comprehensive Guide to the Technology CIMB Runs

*A companion deep-dive in the per-bank software-systems series of the [jackliusr/research](https://github.com/jackliusr/research) repository — the CIMB Group entry alongside [Maybank](maybank_software_systems_guide.md) (its closest regional sibling: a Malaysian universal bank with a Singapore franchise) and [Citibank](citibank_software_systems_guide.md) (the series' structural template). This guide focuses on the **specific software and technology systems** behind CIMB Group Holdings Berhad: the payments/transaction-banking stack, cards and wealth platforms, the core-banking and legacy estate, the digital channels (CIMB OCTO, CIMB Clicks, OCTO Biz, Touch 'n Go), data & AI, Islamic-finance systems (CIMB Islamic), the risk & regulatory context, and the Singapore franchise — a wholesale-and-retail branch of CIMB Bank Berhad that the group has run since the 1940s under one predecessor name or another. It documents what is publicly verifiable, what is inferred from industry practice, and what CIMB does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); ❌ = disputed (the record contradicts the claim); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-claims-status-and-verification-notes), and the non-public specifics are collected in [§12](#12-what-could-not-be-verified).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Software-Systems Focus — the technology estate of CIMB Group Holdings Berhad (Bursa Malaysia: 1023): core banking, payments/TTS, cards, wealth, digital channels, Islamic finance, data & AI, risk context, the Singapore franchise
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides:** [Maybank Software Systems Guide](maybank_software_systems_guide.md) (closest regional sibling), [Citibank Software Systems Guide](citibank_software_systems_guide.md) (structural template), [DBS Software Systems Guide](dbs_software_systems_guide.md) / [OCBC Software Systems Guide](ocbc_software_systems_guide.md) (the SG-bank genre), [Core Banking Systems Guide](core_banking_systems_guide.md), [Payment Rails Guide](payment_rails_guide.md), [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Banks in Singapore Guide](banks_in_singapore_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md), [Private Banking Guide](private_banking_guide.md), [Nets Singapore Guide](nets_singapore_guide.md)

---

### Table of Contents

1. [Bank Profile: From Pertanian Baring Sanwa to CIMB Group](#1-bank-profile-from-pertanian-baring-sanwa-to-cimb-group)
2. [Payments and Transaction Banking](#2-payments-and-transaction-banking)
3. [Cards and Wealth](#3-cards-and-wealth)
4. [Core Banking and the Legacy Question](#4-core-banking-and-the-legacy-question)
5. [Digital Channels: CIMB OCTO, CIMB Clicks and the Group's App Family](#5-digital-channels-cimb-octo-cimb-clicks-and-the-groups-app-family)
6. [Data and AI](#6-data-and-ai)
7. [Islamic Finance Systems: CIMB Islamic](#7-islamic-finance-systems-cimb-islamic)
8. [Risk and Regulatory Context](#8-risk-and-regulatory-context)
9. [Singapore Angle: The Branch, the Hub and the Corridor](#9-singapore-angle-the-branch-the-hub-and-the-corridor)
10. [Worked Example: Cymbal Bank × CIMB — A Regional Correspondent Relationship](#10-worked-example-cymbal-bank--cimb--a-regional-correspondent-relationship)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Bank Profile: From Pertanian Baring Sanwa to CIMB Group

### 1.1 Scope and Verification Convention

This guide is the **software-systems deep-dive for CIMB Group** — the CIMB mirror of the [Maybank guide](maybank_software_systems_guide.md) that is this series' closest regional sibling, and the second Malaysian universal bank documented after the tiger. Because the repository has no separate CIMB *bank* guide, this entry carries both the verified bank profile (history, group structure, footprint, scale — §1) and the systems landscape (§2–§7), followed by the risk & regulatory context (§8), the Singapore angle (§9), a worked Cymbal Bank example (§10), and the honest claims audit (§11–§12).

The verification discipline is the same one this series applies to every bank: **✅ verified** means the claim was confirmed in this research pass against a primary source (CIMB's own history/presence pages and newsroom releases, Bursa Malaysia filings, MAS's Financial Institutions Directory) or a strong secondary source (The Edge Malaysia, The Star, Reuters, The Asian Banker, CNA). **⚠ flagged** means the claim is inferred, approximate, single-source, or structurally reconstructed. **❌ disputed** marks claims where the public record contradicts the common telling. The [What Could Not Be Verified section](#12-what-could-not-be-verified) collects every materially non-public item.

One naming note up front, because it recurs: **CIMB is an abbreviation for Commerce International Merchant Bankers** ✅ (Wikipedia/Bursa; the group's own heritage pages confirm the 1986 renaming that created the initials) — and the initials predate the group by two decades. The corporate that is now CIMB Group Holdings Berhad was, for most of its life, a merchant bank and then a holding company that grew by consolidation; the CIMB *name* dates from 1986, while the group's deepest banking roots reach back to 1924.

### 1.2 What Is Public: The CIMB Disclosure Reality

CIMB's disclosure posture sits close to Maybank's on this series' transparency spectrum, with one distinctive difference: **CIMB publishes real technology-spend and digital-outcome numbers** (RM1.7 billion of technology investment in FY2025; RM100+ million on AI training in 2025; "next generation ready" claims for its front-end apps — all in the FY2025 results release ✅), while remaining **almost entirely silent on engine identity** — no OCBC-style "our retail core is X" statement, no SC-style "Atlas on AWS" architecture narrative, and no vendor case-study trail naming its core-banking ledgers. The consequence, enforced rigorously throughout: the **product and channel layer** (CIMB OCTO, OCTO Biz, CIMB Clicks, Touch 'n Go, BizChannel@CIMB) is mostly ✅-verifiable; the **engine layer** (which entity runs which core, the payments hubs, the AML platform, the data platform) is mostly ⚠-inferred, with vendor datapoints (Pega Cloud at CIMB Niaga, the Virtusa-led modernisation) verifiable only where third parties published them.

### 1.3 The Verified History: 1924 → 2026

CIMB's own heritage pages (cimb.com "Our History", decade by decade) are unusually frank about the consolidation path, and every load-bearing date below is verified against them unless flagged ✅:

- **1924** — **Bian Chiang Bank** established in Kuching, Sarawak ✅ (CIMB heritage 1990s page; Wikipedia). The deepest banking root of the group's tree; it becomes Bank of Commerce in 1979.
- **1935** — **Ban Hin Lee Bank** incorporated in Penang (17 September 1935) by "Towkay" Yeap Chor Ee ✅ (CIMB heritage). Ban Hin Lee's Singapore operations are the group's claimed origin in the city-state (§9).
- **1947** — the group's own Singapore presence page dates CIMB's presence in Singapore to **1947, "through Ban Hin Lee Bank"** ✅/⚠ (CIMB primary claim; the specific 1947 event — a Ban Hin Lee Singapore branch or representative office — is not detailed in the sources extracted this pass, flagged).
- **1965** — **Bank Bumiputra Malaysia Berhad** incorporated ✅ (CIMB heritage); the same year **Southern Banking Ltd** is founded in Penang ✅ (CIMB heritage) — the two banks that the 1999 and 2006 mergers would fold into the group.
- **April 1974** — **Pertanian Baring Sanwa Multinational Bank (PBSM)** incorporated in Kuala Lumpur ✅ (CIMB heritage; Wikipedia). This is the corporate that becomes CIMB. PBSM was a **four-way joint venture** of **Bank Pertanian Malaysia** (the Malaysian agricultural bank), **Baring Brothers** and **Multinational Bank of the United Kingdom**, and **Sanwa Bank of Japan**, launched by Prime Minister **Tun Abdul Razak** and managed by Baring Brothers; its business was corporate advisory, funding for multinationals, and M&A/restructuring for Malaysian companies. Note for the lineage question: CIMB's *founding year* as commonly celebrated (its 40th-anniversary era, mid-2010s) is 1974 — the JV incorporation — even though the CIMB name only arrived in 1986.
- **November 1979** — **Bian Chiang Bank is purchased by the Fleet Group (UMNO-linked) and renamed Bank of Commerce Bhd** ✅ (CIMB heritage; Wikipedia) — retaining the BCB initials. Heritage adds the colour that Bank of Commerce's co-shareholder JP Morgan influenced its "strong focus on systems and transparency" ⚠ (single-source heritage colour, flagged).
- **1986** — Bank of Commerce replaces Bank Pertanian as controlling shareholder of PBSM, which is **renamed Commerce International Merchant Bankers Berhad (CIMB)** ✅ (CIMB heritage; Wikipedia). Corporate finance and IPOs remain the focus; stockbroking is added; by 1989 CIMB is Malaysia's top advisor for new listings ✅ (CIMB heritage).
- **1987** — Bank of Commerce lists on the Kuala Lumpur Stock Exchange ✅ (CIMB heritage).
- **November 1991** — **Bank of Commerce acquires United Asian Bank** (the 1972 Malaysian-Indian joint bank); the merged bank becomes Bank of Commerce (Malaysia) Berhad under the renamed holding company **Commerce-Asset Holdings Bhd (CAHB)**, with CIMB a separate CAHB subsidiary ✅ (CIMB heritage). The acquisition nearly quadruples the branch network.
- **October 1999** — **Bank of Commerce (Malaysia) Berhad merges with Bank Bumiputra Malaysia Berhad to form Bumiputra-Commerce Bank Berhad**, under CAHB ✅ (CIMB heritage: "the biggest merger in Malaysia's banking history" at the time). Bank Bumiputra, incorporated 1965, had been the country's largest bank by assets in the 1980s and the first Malaysian bank with New York, London, Tokyo, Bahrain and Hong Kong operations; it emerged from the Asian financial crisis into this merger.
- **1 July 2000** — **Southern Bank acquires Ban Hin Lee Bank** plus finance companies (United Merchant Finance → Southern Finance, Perdana Finance, Cempaka Finance) under the government's banking-consolidation plan ✅ (CIMB heritage). Heritage also records that Southern Bank "was the first in Malaysia to set up the MEPS/ATM system" ✅ (CIMB heritage — a payments-systems datapoint that §2 picks up).
- **2002** — CAHB completes the acquisition of **51% of PT Bank Niaga** from the Indonesian government via the Indonesian Bank Restructuring Agency (IBRA) ✅ (CIMB heritage; Wikipedia) — the seed of the group's Indonesian franchise.
- **January 2003** — **CIMB Bhd lists on the KLSE Main Board** ✅ (CIMB heritage); the listing lasts only ~three years before the 2005–06 restructuring. **June 2003** — **CIMB Islamic is launched** by Bank Negara Governor Tan Sri Dr Zeti Akhtar Aziz ✅ (CIMB heritage; Wikipedia).
- **2004** — CIMB-Principal Asset Management formed (CIMB Bhd takes 70% of Commerce Trust Bhd and Commerce Asset Fund Managers; joint venture with the Principal Financial Group of the USA) ✅ (CIMB heritage).
- **June 2005** — **CIMB acquires GK Goh Securities Pte Ltd** of Singapore (founded 1979), forming CIMB-GK Securities ✅ (CIMB heritage; Wikipedia) — the group's international investment-banking platform and its Singapore securities anchor (§9). In the same month CIMB announces the **acquisition of the Bumiputra-Commerce Group**; CIMB is delisted and **CAHB is renamed Bumiputra-Commerce Holdings Bhd** ✅ (CIMB heritage).
- **January 2006** — the restructuring completes and the **new CIMB Group is born as a universal bank** ✅ (CIMB heritage). **March 2006** — the group acquires **Southern Bank** after extensive negotiations ✅/⚠ (Wikipedia: "in March, CIMB Group acquired SBB"; CIMB heritage lists 2006). **September 2006** — the new CIMB Group is formally launched by Prime Minister **Abdullah Ahmad Badawi**, "signifying the culmination of the 3-way merger of Commerce International Merchant Bankers, Bumiputra-Commerce Bank and Southern Bank"; the group unveils the tagline **"Forward Banking"** ✅ (CIMB heritage).
- **2007** — CIMB-GK Securities (Thailand) and CIMB-GK Securities (USA) established; CIMB Foundation launched (November, with an initial RM100 million pledge) ✅ (CIMB heritage).
- **2008** — the regional-acquisition year: a **19.99% strategic stake in Bank of Yingkou** (China); the **merger of PT Bank Niaga Tbk with PT Bank Lippo Tbk** — an initiative of ultimate shareholder Khazanah Nasional to comply with Bank Indonesia's Single Presence Policy, creating the sixth-largest Indonesian bank, rebranded **CIMB Niaga** (merger effective 1 November 2008 per Wikipedia ✅/⚠); CIMB-Principal Islamic Asset Management (50:50); and an agreement with Thailand's Financial Institutions Development Fund to buy **42.13% of BankThai Pcl**, raised to ~93% via tender offer and rights issue ✅ (CIMB heritage).
- **May 2009** — **CIMB Thai** brand launched ✅ (CIMB heritage). **September 2009** — CIMB Group sets up **consumer banking services in Singapore** through CIMB Bank Singapore ✅ (CIMB heritage) — the retail launch that §9 documents in detail.
- **2010** — Cambodia: CIMB Bank Plc, first branch/headquarters launched in Phnom Penh (19 November 2010) ✅ (CIMB heritage; Wikipedia).
- **2012** — acquisition of most of **RBS's Asia-Pacific cash-equities and investment-banking businesses** (GBP 173.9 million / RM849.4 million) plus Thai broker SICCO Securities ✅ (CIMB heritage; Wikipedia).
- **2013** — the group moves into **Menara CIMB**, its 40-storey headquarters at Kuala Lumpur Sentral; Taiwan, India and Korea operations complete the APAC investment-banking platform ✅ (CIMB heritage).
- **2016** — first branch in Vietnam via a 100%-owned subsidiary ✅ (CIMB heritage).
- **2017** — **China Galaxy Securities becomes a 50:50 shareholder in CIMB Securities International Pte. Ltd.** (Singapore-incorporated), repositioning the group's stockbroking business ⚠ (CIMB heritage records the 50:50 JV; the full entity scope and current status of the JV are §12 items).
- **December 2018** — CIMB Bank Philippines Inc launches its digital retail-banking business after BSP approvals, completing the group's presence in **all ten ASEAN countries** ✅ (CIMB heritage).
- **2020s sustainability and digital era** — 2020: first Malaysian bank and "first significant emerging-market bank globally" to commit to phasing out coal from its portfolio by 2040 ✅ (CIMB heritage); 2021: first ASEAN bank to join the Net Zero Banking Alliance ✅ (CIMB heritage); **2022: CIMB introduces the CIMB OCTO App**, its "next generation mobile banking app... supported by a new technology stack and operational infrastructure" ✅ (CIMB heritage 2020s page); 2025: Forward30 strategy year one, the RM2 billion capital-return programme (November), the blockchain/tokenisation announcement (December) ✅ (FY2025 results release); 2026: the tokenised-sukuk settlement pilot (August) and the CIMB Thai delisting from the Stock Exchange of Thailand (August) ✅ (CIMB newsroom).

### 1.4 Group Structure: The Universal-Bank Shape

CIMB is a universal banking group in the [Universal Banking Model Guide](universal_banking_model_guide.md) sense — consumer banking, wholesale banking (investment banking plus corporate banking/treasury), and Islamic banking under one listed holding company ✅/⚠ (entity names verified via CIMB's leadership pages and Wikipedia; the legal-entity diagram lives in CIMB's annual report, not re-verified here):

| Group business | Public entity (verified) | Notes |
|---|---|---|
| **Listed holding company** | **CIMB Group Holdings Berhad** (formerly Bumiputra-Commerce Holdings Berhad ⚠ rename year — see §11.2) | Bursa Malaysia: 1023; state-influenced shareholder base with Khazanah Nasional as the largest shareholder ⚠ (stake % dated — §11) |
| **Commercial banking (Malaysia)** | **CIMB Bank Berhad** | The licensed Malaysian bank; operates the Singapore branch (§9) and is parent to several regional entities ⚠ |
| **Investment banking** | **CIMB Investment Bank Berhad** | The merchant-bank lineage (CIMB, est. 1974/1986) in its modern licensed form |
| **Islamic finance** | **CIMB Islamic Bank Berhad** | Launched as CIMB Islamic in June 2003; Islamic banking subsidiary of the group (§7) |
| **Indonesia** | **PT Bank CIMB Niaga Tbk** (IDX: BNGA) | Sixth-largest Indonesian bank by assets ✅/⚠ (Wikipedia, dated); ex-Bank Niaga (1955) + Bank Lippo (1948), merged 2008 |
| **Thailand** | **CIMB Thai Bank Public Company Limited** | Ex-BankThai (acquired 2008–09); delisted from the SET in August 2026 ✅ (CIMB newsroom) |
| **Cambodia** | **CIMB Bank PLC** | Wholly owned, launched 2010 |
| **Vietnam** | **CIMB Bank (Vietnam) Limited** | 100%-owned; first branch 2016 |
| **Philippines** | **CIMB Bank Philippines Inc** | BSP-licensed digital retail bank, launched December 2018 |
| **Securities (Singapore/regional)** | CIMB Securities entities; 50:50 regional stockbroking JV with China Galaxy Securities (2017) | ⚠ current entity structure and JV status — §12 |
| **Asset management** | CIMB-Principal Asset Management / CIMB-Principal Islamic Asset Management | JVs with Principal Financial Group (2004/2008) |
| **Digital/e-wallet** | Touch 'n Go Sdn Bhd and TnG Digital | Touch 'n Go Digital named in FY25 results as a group front-end; ownership split ⚠ (§5/§12) |
| **CSR** | CIMB Foundation | Launched November 2007 |

**Scale markers (dated, with the year of each figure):** the group's own About Us page (figures as at **30 June 2026**) states **total assets RM794.9 billion**, **total Islamic assets RM251.2 billion**, **shareholders' funds RM70.3 billion**, coverage of **10 markets**, and **32,000+ employees** ✅ (cimb.com About Us, extracted this pass); **FY2025** (ended 31 December 2025) delivered **net profit RM7.9 billion** (+1.7%), **PBT RM10.7 billion** (+2.7%), **ROE 11.3%**, **CET1 14.3%**, deposits RM524.4 billion, business-engine assets RM778.7 billion, and a record RM5.1 billion dividend ✅ (CIMB FY2025 results release, 27 Feb 2026); the group is **Malaysia's second-largest bank by assets** ✅/⚠ (The Vibes/aggregators; consistent with CIMB's own "#2" positioning); a dated network figure of ~1,080 branches ⚠ (Wikipedia infobox, dated). The **Malaysia home market contributed 61% of PBT in FY2025** (up from 57%) ✅ (FY2025 release) — a reallocation under Forward30 that bears directly on how the group funds its systems estate (§4, §8).

### 1.5 The Verified Timeline at a Glance

| Year | Milestone | Status |
|---|---|---|
| 1924 | Bian Chiang Bank established, Kuching (deepest root) | ✅ (CIMB heritage) |
| 1935 | Ban Hin Lee Bank incorporated, Penang | ✅ (CIMB heritage) |
| 1947 | Group presence in Singapore begins "through Ban Hin Lee Bank" | ✅/⚠ (CIMB claim; detail thin) |
| 1965 | Bank Bumiputra Malaysia Berhad incorporated; Southern Banking Ltd founded | ✅ (CIMB heritage) |
| 1974 | PBSM incorporated (April) — the CIMB founding JV (Bank Pertanian, Baring Brothers, Multinational Bank, Sanwa) | ✅ |
| 1979 | Bian Chiang Bank → Bank of Commerce (November) | ✅ |
| 1986 | PBSM renamed Commerce International Merchant Bankers Berhad (CIMB) | ✅ |
| 1991 | Bank of Commerce acquires United Asian Bank; CAHB formed | ✅ |
| 1999 | Bank of Commerce + Bank Bumiputra → Bumiputra-Commerce Bank (October) | ✅ |
| 2000 | Southern Bank acquires Ban Hin Lee Bank (1 July) | ✅ |
| 2002 | CAHB acquires 51% of PT Bank Niaga (IBRA) | ✅ |
| 2003 | CIMB Bhd lists (January); CIMB Islamic launched (June) | ✅ |
| 2005 | GK Goh Securities acquired (June); CIMB acquires Bumiputra-Commerce Group; CAHB → Bumiputra-Commerce Holdings | ✅ |
| 2006 | CIMB Group universal bank (January); Southern Bank acquired (March); group launched (September); "Forward Banking" tagline | ✅ |
| 2008 | Bank of Yingkou stake; Bank Niaga + Bank Lippo → CIMB Niaga; BankThai agreement (42.13% → 93%) | ✅ |
| 2009 | CIMB Thai launched (May); CIMB Bank Singapore retail launched (September) | ✅ |
| 2010 | Cambodia (CIMB Bank Plc, Phnom Penh) | ✅ |
| 2012 | RBS Asia-Pacific cash equities/IB acquisition; SICCO | ✅ |
| 2013 | Menara CIMB headquarters, KL Sentral | ✅ |
| 2016 | Vietnam (100% subsidiary, first branch) | ✅ |
| 2017 | China Galaxy 50:50 JV in CIMB Securities International Pte Ltd | ✅/⚠ (entity scope) |
| 2018 | CIMB Bank Philippines digital bank launches (December) — ASEAN-10 complete | ✅ |
| 2020 | First Malaysian bank to commit to coal phase-out by 2040 | ✅ |
| 2021 | First ASEAN bank in Net Zero Banking Alliance | ✅ |
| 2022 | CIMB OCTO App introduced ("new technology stack") | ✅ |
| 2025 | Forward30 year one; record FY25 (net profit RM7.9B, ROE 11.3%, CET1 14.3%); RM2B capital-return programme (Nov); blockchain/tokenisation announcement (Dec) | ✅ |
| 2026 | Tokenised-sukuk settlement pilot (Aug); CIMB Thai delisting (Aug); 2Q26 net profit RM1.94B, ROE 11.2% | ✅ |

The timeline reads in three eras that map onto the systems story. **Merchant-bank era (1974–1999):** a Barings-managed JV becomes Malaysia's top IPO house and then the crown jewel of a consolidating commercial-banking group. **Universal-bank consolidation era (1999–2010):** the Bumiputra-Commerce merger, the CIMB-branded universal bank, Southern Bank, and the Indonesian/Thai acquisitions assemble the ASEAN shape the group still has. **Regional-digital era (2010–2026):** Cambodia/Vietnam/Philippines expansion, OCTO and Touch 'n Go, Forward30, and the data/AI + tokenisation agenda. The systems-relevant rows — 2000 (Southern Bank/MEPS heritage), 2003 (Islamic), 2009 (SG retail), 2018 (Philippines digital), 2022 (OCTO), 2025 (F30 tech spend) — are the anchors the §2–§7 sections hang their verified claims on.

### 1.6 The Strategy Layer: Forward30

The group's current strategy frame is **Forward30 ("F30")** — a six-year roadmap of which **FY2025 was year one** ✅ (CIMB FY2025 results release; CIMB Forward30 page). The verified facts that matter for the systems map:

- **The 4Cs** ✅ (CIMB Forward30 page): **Capital & Resources** (capital reallocated on ROE by market — Malaysia's PBT share rose 57%→61% in FY25), **Cash** (profitable CASA strategy), **Cross-sell** (NOII ratio 31.7% in FY25), and **Capabilities** — explicitly "modernise core banking technology, and data & AI" (the group's own words ✅), plus process re-engineering and digital automation.
- **The targets** ✅/⚠ (CIMB Forward30 page, as printed): 2027 — ROE 12–13%, cost-to-income rendered "> 45%" ⚠ (almost certainly a rendering artifact for "≤45%"/"45%", flagged), credit cost 30–40 bps, CET1 14–15%; 2030 — ROE top-quartile, cost-to-income "low 40s", CASA 45%, CET1 13–14%, NPS top-3.
- **The tech spend that backs it** ✅ (FY2025 release): RM1.7 billion technology investment in FY2025; RM100+ million on AI training/upskilling; front-end apps "next generation ready"; middle-to-back-end modernisation underway — the disclosure posture §1.2 promised.
- **Leadership anchors** ✅ (CIMB leadership pages): **Novan Amirudin** is Group Chief Executive Officer (and Executive Director); **Datuk Syed Zaid Albar** is Chairperson; **Khairul Rifaie** is Group Chief Financial and Strategy Officer. The leadership-by-entity list on cimb.com names CIMB Bank Berhad, CIMB Investment Bank Berhad, CIMB Islamic Bank Berhad, CIMB Bank PLC, CIMB Thai Bank PCL, PT Bank CIMB Niaga Tbk and CIMB Bank (Vietnam) Limited — note what is *absent*: Singapore has no subsidiary seat, because the Singapore franchise is a branch of CIMB Bank Berhad (§9).

---

## 2. Payments and Transaction Banking

### 2.1 The Transaction-Banking Frame

CIMB's transaction-banking business sits inside the wholesale-banking segment (corporate banking, treasury & markets, transaction banking) ✅ (Wikipedia/CIMB corporate structure; the FY2025 and 2Q26 releases confirm "Transaction Banking remains a key focus" ✅). The 2Q26 results release is explicit: transaction banking is a strategic focus "supported by the regional roll-out of the CIMB OCTO Biz app" ✅ — the group's public language ties its cash-management ambition directly to a named channel product, which is more than most peers disclose and still less than an architecture diagram. For the rails underneath — SWIFT/ISO 20022 mechanics, correspondent banking and nostro flows, GPI, domestic real-time rails, cross-border QR linkage — this guide **cross-references rather than re-derives**: the mechanics live in the [Payment Rails Guide](payment_rails_guide.md), the trade-instrument canon in the repo's trade guides, and the domestic Malaysian rail structure (MEPS/PayNet, DuitNow, FPX) is documented in the [Maybank guide](maybank_software_systems_guide.md) §3 and the [Nets Singapore Guide](nets_singapore_guide.md). What follows is the CIMB-specific, publicly verifiable layer only.

### 2.2 The Channel Products: BizChannel@CIMB and OCTO Biz

- **BizChannel@CIMB** ✅ (cimb.com.my business pages via the JS-SEZ page) — CIMB's corporate cash-management digital channel in Malaysia: payments, remittances, and "Live Function" real-time FX monitoring. The JS-SEZ cross-border proposition (§9) explicitly routes remittances and settlements through BizChannel@CIMB ✅ — this is the closest CIMB comes to a public naming of its corporate-payments front end.
- **OCTO Biz** ✅ (CIMB homepage; FY2025 release; 2Q26 release) — the group's SME/business mobile proposition ("Business control at your fingertips", CIMB homepage); the FY2025 release names it one of the group's "next generation ready" front-end applications, and 2Q26 confirms its **regional roll-out** is the vehicle for the transaction-banking focus. Its engine (payments hub, host-to-host APIs, ERP integrations) is not public (§12).
- **The SME scale** ✅/⚠ — CIMB Malaysia claims it serves **500,000 SMEs** (Qorus award citation on the JS-SEZ page, 2024) — a dated ⚠ figure that frames why the group pushes OCTO Biz as a transaction-banking wedge.

### 2.3 Cash Management and the Corporate Wins

- **Payroll/cash-management win** ✅ (CIMB newsroom, 20 Aug 2026) — Infineon Melaka "entrusts CIMB with payroll banking for more than 10,000 employees" — the sort of named corporate anchor that confirms the franchise's payroll/transactional scale without revealing platform details.
- **The treasury client franchise** ✅ (FY2025 release) — treasury client franchise income +7.2% YoY in FY2025; NOII RM7.1 billion (+3.1%); fees and commission +3.2% — the income layer that transaction banking feeds.
- **Fees from cross-sell** ✅ — the group's F30 cross-sell pillar explicitly targets "Wholesale clients, Corporate and Wealth, as well as Commercial and Transaction-led Banking" (Forward30 page).

### 2.4 The Malaysian Rails Heritage and Membership

- **The MEPS heritage datapoint** ✅ (CIMB heritage 1990s page) — Southern Bank, folded into the group in 2006, "was the first in Malaysia to set up the MEPS/ATM system." The group's domestic retail-payments estate therefore carries a genuine payments-infrastructure lineage (structural claim, unmarked; the modern estate is PayNet/MEPS-era, cross-ref the [Maybank guide](maybank_software_systems_guide.md) §3).
- **Domestic rail membership** ⚠ structural — as one of Malaysia's largest banks, CIMB Bank Berhad is a participant in PayNet-operated rails (DuitNow proxy-ID transfers, DuitNow QR, FPX, MyDebit/MEPS ATM sharing); operator lists confirm the class of membership, and CIMB's own marketing (the MY→SG corridor page, cimb.com.my) implies DuitNow-era instant capability ⚠ (the per-rail membership roster was not re-verified item-by-item this pass — §12 item 9).
- **Cross-border QR and the regional layer** ⚠/✅ — the ASEAN cross-border QR interoperability layer and the PayNow–DuitNow linkage are documented in the [Payment Rails Guide](payment_rails_guide.md) and [Nets Singapore Guide](nets_singapore_guide.md); CIMB's position in the SG↔MY real-time corridor is detailed in §9.2.

### 2.5 The SG↔MY Corridor: On-Us Instant Cross-Border

- **The product** ✅ (cimb.com.sg and cimb.com.my product pages, extracted this pass) — CIMB Singapore offers **instant SGD↔MYR transfers between CIMB SG and CIMB MY accounts** at a "guaranteed best exchange rate", with the Malaysian-side page advertising instant MY→SG transfers at zero fees up to RM50,000. This is an **on-us corridor** — the two branches/subsidiaries of one group posting against each other's ledgers in near-real-time — which is precisely the systems shape the §10 worked example exploits (structural inference from the product mechanics ⚠; the group has not published the underlying hub design).
- **The corridor strategy** ✅ (2Q26 release) — the group explicitly prioritises "strategic cross-border corridors... JS-SEZ and ASEAN Financial Passport, China-ASEAN partnership," and Singapore's role as the ASEAN corporate hub (§9) — the SG↔MY corridor product is the retail-facing edge of that strategy.

### 2.6 Trade Finance and Supply Chain

- **The trade platform layer** ⚠ — no CIMB-specific trade-platform product name (no public "CIMB TradeConnect"-class disclosure) was found this pass; the group's trade business is documented as part of wholesale banking ✅ (structure), and its engines sit in §12 item 8. The instrument mechanics (LCs, guarantees, collections over SWIFT MT7xx/MT4xx) are standard and cross-referenced to the repo's [Trade Finance Guide](trade_finance_guide.md) rather than re-derived.
- **Trade corridors** ✅/⚠ — the JS-SEZ proposition includes "Trade Finance and Guaranteed Support" (JS-SEZ page), and the group's #1 MIST investment-banking/DCM rankings (FY2025 release: #1 DCM with 32.5% Malaysian share and 15.3% across Malaysia-Indonesia-Singapore-Thailand) show the wholesale trade ecosystem the transaction bank plugs into ✅ (rankings verified; the underlying trade-loan systems ⚠).
- **The digital trade/deposit edge** ✅ — the Aug 2026 tokenised-sukuk pilot (§1.5, §7) is as much a transaction-banking/settlement story as a capital-markets one: CIMB itself describes the pilot as bringing together "Islamic finance, capital markets, **transaction banking**, technology, legal documentation, custody and settlement" ✅ (tokenisation release) — a rare public glimpse of transaction banking's role in the group's digital-asset ambitions.

### 2.7 The Payments Estate at a Glance

| Payments layer | Public facts | Systems note | Status |
|---|---|---|---|
| Corporate cash-management channel (MY) | BizChannel@CIMB named; JS-SEZ remittance route | Engine ⚠ (§12 item 7) | ✅ product; ⚠ engine |
| SME/business channel | OCTO Biz; regional roll-out per 2Q26 | App + API layer ⚠ | ✅ product; ⚠ engine |
| Retail SG↔MY instant corridor | CIMB SG ↔ CIMB MY accounts; guaranteed rate; RM50k limit MY-side | On-us corridor ⚠ (§9.2, §10) | ✅ product; ⚠ mechanics |
| Payroll/cash management | Infineon Melaka win (Aug 2026) | Host-to-host rails ⚠ | ✅ deal; ⚠ platform |
| Domestic rails (MY) | MEPS/PayNet-class membership ⚠; DuitNow-era capability implied | Payment hub ⚠ | ⚠ structural |
| SWIFT/correspondent layer | Group-wide SWIFT membership (structural); GPI-class ⚠ | Cross-ref [Payment Rails Guide](payment_rails_guide.md) | ⚠ structural |
| Trade finance | Wholesale business line; JS-SEZ trade support | Engines not public (§12 item 8) | ✅ business; ⚠ engines |

The table closes this section the way every section of this guide closes: **the product names and the deals are public; the payment hub that clears them is not.** What the public record does support is the strategic direction — transaction banking as an F30 focus, OCTO Biz as its regional channel, and the SG↔MY corridor as its flagship cross-border product.

---

## 3. Cards and Wealth

### 3.1 Cards: The Public Shape

Cards at CIMB split across the same public/private divide as everything else in this guide: **the products and the promotions are public; the portfolio numbers and the card platform are not.** Verified this pass:

- **The Singapore card business is live and marketing hard** ✅ (cimb.com.sg homepage, extracted this pass) — the SG retail site leads with credit-card offers (a "16,000 Max Miles" travel-miles promotion was the homepage hero), alongside the deposit franchise (FastSaver) and an affluent tier (see §3.2). The SG consumer business is deposit-and-card-led — a function of the branch's September 2009 consumer-banking launch (§9) and of its deliberately retail-light scale relative to the Malaysian parent ⚠ (the SG profit split is not public — §12).
- **Group card scale is not disclosed** ⚠ — no verified figure for group card receivables, card counts or card income was found this pass. The FY2025 release reports fee-and-commission income of RM2.5 billion-class (+3.2%) ✅ without a card breakout ✅/⚠ (level verified; breakout absent).
- **Network and brand structure** ⚠ structural — as a universal group, CIMB issues general-purpose cards (Visa/Mastercard class) across CIMB Bank Berhad and CIMB Islamic in Malaysia, CIMB Niaga in Indonesia, and the Singapore branch; the per-market network/brand matrices and co-brand partners were not verified item-by-item (§12 item 4).
- **OCTO-era card servicing** ✅/⚠ — the CIMB OCTO product pages (Malaysia and Singapore, both extracted this pass) advertise in-app card management — activation, blocking, statements, bill payments — as standard OCTO features ✅ (product-page claims); the underlying tokenisation / 3-D Secure / card-management engine is not named ⚠.
- **Brand colour**: the **CIMB Classic**, the PGA Tour co-sanctioned golf event played in Malaysia in the 2010s, put the CIMB brand in front of US audiences — Ben Crane's October 2010 win at the CIMB Asia Pacific Classic Malaysia is on the tournament record ✅ (Wikipedia/tour) — a sponsorship-era datapoint, not a systems one, included only because it is the most visible card-era brand spend the trail verified.

### 3.2 Wealth: The Affluent Tier and the Private Wealth Push

The group's wealth ambition is one of its most explicit public strategy statements, and it is regional by design:

- **CIMB Private Wealth aims to be "the primary wealth partner for affluent individuals in ASEAN"** ✅ (CIMB newsroom release, 2026, extracted this pass) — the release frames wealth as a Forward30 growth pillar riding the same cross-sell engine as the rest of the group (wealth clients are an explicit F30 cross-sell target alongside corporate and commercial, cross-ref §1.6).
- **The ASEAN build-out** ⚠ — the 2026 private-wealth push is regional in its stated ambition; which entities/branches carry it (Singapore branch, Labuan, Malaysia's CIMB Private Banking) is not fully extracted from the release (§12 item 6).
- **The asset-management JVs** ✅ (cross-ref §1.4) — CIMB-Principal Asset Management (2004, 70:30-style CIMB control ⚠ exact split) and CIMB-Principal Islamic Asset Management (2008, 50:50) give the group the Principal Financial Group's engine heritage in unit trusts and Islamic funds; the underlying fund-administration platforms are not public ⚠.
- **The Singapore affluent tier** ✅/⚠ — cimb.com.sg markets **"Preferred Banking"** as the branch's premium tier (homepage copy, extracted this pass) — note the naming: Singapore says "Preferred", the wider group's mass-affluent brand family is CIMB Premier-class ⚠ (brand map not fully verified — §12 item 5); the SG wealth engine (advisory, execution, the wealth ledger inside the branch's core) is not public ⚠.

### 3.3 Cards and Wealth at a Glance

| Layer | Public facts | Systems note | Status |
|---|---|---|---|
| SG credit cards | Live offers on cimb.com.sg (miles promos); deposit-card-led retail | Card engine ⚠ | ✅ product; ⚠ engine |
| Group card estate | Issuer across MY/ID/SG (structural); no portfolio breakout | No vendor named | ⚠ unverified scale |
| OCTO card servicing | Card management in-app (MY + SG product pages) | Tokenisation stack ⚠ | ✅ feature; ⚠ engine |
| Group affluent strategy | CIMB Private Wealth "primary wealth partner in ASEAN" (2026) | Advisory platforms ⚠ | ✅ strategy |
| Asset management | CIMB-Principal / CIMB-Principal Islamic JVs | Fund platforms ⚠ | ✅ structure |
| SG premium tier | "Preferred Banking" named on cimb.com.sg | SG wealth ledger ⚠ | ✅ product; ⚠ engine |

The wealth section closes like the payments one: the **ambition and the brand tiers are public; the engines that run the portfolios are not** — and for wealth specifically, CIMB's public material says more about where it wants to be in ASEAN by 2030 (top-quartile ROE, top-3 NPS under F30) than about what it runs today. The genre conventions for wealth platforms live in the [Private Banking Guide](private_banking_guide.md) and the [Maybank guide](maybank_software_systems_guide.md)'s wealth section.

---

## 4. Core Banking and the Legacy Question

### 4.1 The Honest Opening: CIMB Does Not Say

This is the section where this guide must be bluntest. **CIMB does not publicly name its core-banking engines.** Unlike OCBC's named retail core or the Maybank guide's verified Silverlake SiBS position for its Malaysian sibling ✅, no verified statement by CIMB Group — results release, annual report extract, or vendor case study found this pass — names the ledger that runs CIMB Bank Berhad's Malaysian retail book, CIMB Islamic's deposit/financing core, CIMB Niaga's Indonesian core, or the SG branch's books. The searches run in the research pass for "CIMB + Silverlake", "CIMB + Temenos", "CIMB + Avaloq" all came back empty ✅/⚠ (absence of record, not proof of absence — §12 item 1). Everything below is therefore either CIMB's own public language about modernisation, or third-party datapoints, or structural inference, and it is marked accordingly.

### 4.2 What CIMB Does Say: The Modernisation Language

- **F30 names core modernisation explicitly** ✅ (Forward30 page, extracted this pass) — the Capabilities pillar's own words are "modernise core banking technology, and data & AI". This is the group's clearest public admission that its core estate is a programme item, not a finished asset.
- **The FY2025 results release adds the money and the phasing** ✅ — RM1.7 billion technology investment in FY2025; front-end apps described as "next generation ready"; and "middle-to-back-end modernisation" disclosed as underway ✅ (FY2025 release) — the release's own three-layer framing (front / middle / back) is the closest CIMB comes to an architecture statement.
- **OCTO's infrastructure claim** ✅/⚠ (CIMB heritage 2020s page) — the 2022 OCTO launch was "supported by a new technology stack and operational infrastructure" — a group claim that the digital front end does not sit on the oldest estate; how far the "new stack" reaches into the middle and back layers is unquantified (§12 item 2).
- **AI training is treated as a core-investment line** ✅ — the RM100+ million AI-training figure (FY2025) is reported beside the tech-spend number, i.e. capability investment is an F30 Capabilities line item, not a bolt-on.

### 4.3 The Third-Party Datapoints: Pega at Niaga, Virtusa-Led

The one named core-adjacent vendor datapoint the research pass verified is Indonesian, not Malaysian:

- **CIMB Niaga migrated core-banking workflows to Pega Cloud** ✅/⚠ (Virtusa pulse via briefglance.com, secondary source, extracted this pass) — the migration, described as an Indonesia first and completed **26 February 2026**, moved customer-servicing and operations workflows onto Pega's cloud platform, with Virtusa as the delivery partner ✅/⚠ (secondary source; contract scope and CIMB-side confirmation not independently verified — §12 item 3). Note what it is *not*: a claim that Pega is CIMB Niaga's core ledger — Pega is the workflow/BPM layer around the core, which is precisely the vendor-datapoint pattern this series sees across ASEAN banks (cross-ref [Core Banking Systems Guide](core_banking_systems_guide.md)).
- **The Southern Bank/MEPS heritage** ✅ (CIMB heritage, cross-ref §2.4) — the 2000s-era claim that Southern Bank "was the first in Malaysia to set up the MEPS/ATM system" is a reminder that parts of this estate are genuinely old, which is the context for the F30 modernisation language.

### 4.4 The Structural Inference: A Multi-Core, Multi-Generation Estate

What the public record supports, assembled carefully:

| Estate layer | Structural position | Evidence class |
|---|---|---|
| Malaysian retail/commercial core | Runs CIMB Bank Berhad's books; vendor unnamed | ⚠ inference; F30 "modernise" language implies legacy core |
| CIMB Islamic core | Separate licensed bank (IFSA 2013) — separate core likely | ⚠ structural (licensing) |
| CIMB Niaga core | Indonesian ledger + Pega Cloud workflow layer (Feb 2026) | ✅/⚠ vendor datapoint |
| Singapore branch books | Branch of CIMB Bank Berhad — plausibly on the MY estate or a regional ledger | ⚠ inference (§9.4) |
| Middle layer (payments hub, trade, AML) | Unnamed | ⚠ (§12 item 7) |
| Digital layer | OCTO "new technology stack" (2022); "next generation ready" (FY25) | ✅ group claim |

The disciplined reading, in the genre's terms: CIMB is a **multi-core, multi-generation group running a named-digital/front-end strategy over an unnamed engine estate**, with F30's RM1.7-billion-a-year technology line funding the transition. That makes it the ASEAN archetype of the [Core Banking Systems Guide](core_banking_systems_guide.md)'s "modernise-don't-replace" pattern — and the honest answer to the vendor question is the one §4.1 gave: **CIMB has not said, and this guide will not guess for it.** The Maybank guide's treatment of the same question (a verified SiBS core vs unverified claims) is the series' calibration point.

### 4.5 The Genre Context: What the Siblings Show

The vendor question is not CIMB's alone, and the series' treatment of it is already established in the sibling guides: the [Maybank guide](maybank_software_systems_guide.md) documents a **verified Silverlake SiBS core** for the Malaysian sibling with named layer vendors around it, while the [Citibank guide](citibank_software_systems_guide.md) — this series' structural template — documents a **mostly in-house, vendor-light estate** where Citi builds rather than buys. CIMB sits closer to the Citi end of that spectrum in *disclosure terms*: a group large enough (RM794.9 billion assets ✅) to run a multi-vendor, multi-generation estate without ever needing a vendor case study for marketing. For the reader coming from the [Core Banking Systems Guide](core_banking_systems_guide.md), the CIMB evidence supports only the "modernisation-pressure" reading, and the signal mix is genuinely mixed rather than one-directional:

| Signal direction | Evidence | Status |
|---|---|---|
| Legacy pressure | Southern Bank MEPS-era heritage (§2.4); July 2024 BNM AMP for service disruptions (§8.1); F30's own "modernise core banking technology" wording | ✅ |
| Greenfield investment | OCTO on a "new technology stack" (2022); Philippines digital bank (2018); RM1.7B annual tech spend; tokenisation pilot (2026) | ✅ |
| Middle-layer inertia | No named payments hub, AML platform or trade engine anywhere in the public record | ⚠ |

That mixed signal **is** the finding: CIMB's public record supports a group running a modernising digital layer over an ageing engine layer, funding the gap with F30-scale spend — and declining to say which engines, when, or how far the replacement reaches. A follow-up pass would need CIMB's IAR2025 technology-review pages (§14) or vendor-side disclosures to move any row of §4.4 from ⚠ to ✅.

---

## 5. Digital Channels: CIMB OCTO, CIMB Clicks and the Group's App Family

### 5.1 OCTO: The Group's Front-End Standard

**CIMB OCTO** is the group's unified mobile-banking brand, and it is the most verifiable part of CIMB's digital estate because CIMB markets it everywhere:

- **Launch and positioning** ✅ (CIMB heritage 2020s page) — the group introduced the CIMB OCTO App in **2022** as its "next generation mobile banking app... supported by a new technology stack and operational infrastructure" — the flagship artefact of the F30-era front end (cross-ref §4.2).
- **Malaysia** ✅ (cimb.com.my OCTO product page, extracted this pass) — the Malaysian personal-banking site's digital-services hub is the OCTO app page: transfers, payments, account management, card servicing, and the standard app-store "better digital banking" positioning; OCTO is the consumer app of CIMB Bank Berhad and CIMB Islamic (dual-brand eligibility is structural ⚠).
- **Singapore** ✅ (cimb.com.sg OCTO page + cimbclicks.com.sg, extracted this pass) — the Singapore branch runs **"OCTO by CIMB Singapore"** as its mobile app; the older web brand **CIMB Clicks** survives as the SG online-banking portal (cimbclicks.com.sg, "CIMB Online Banking") ✅ — SG's channel pair is OCTO (mobile) + CIMB Clicks (web), the same two-brand pattern CIMB Malaysia used before OCTO absorbed mobile ⚠ (historical MY app-brand sequence not re-verified).
- **"Next generation ready"** ✅ (FY2025 release) — the group's own FY25 label for its front-end applications, OCTO first among them.
- **OCTO Biz** ✅ — the SME/business sibling (cimb.com.my and group homepage): "Business control at your fingertips"; named in the FY2025 release among next-generation apps and in the 2Q26 release as the vehicle of the **regional roll-out** supporting the transaction-banking focus (cross-ref §2.2). Its engine — payments hub, host-to-host APIs, ERP/accounting integrations — is not public (§12 item 7).

### 5.2 The Rest of the App Family

- **CIMB Niaga: OCTO Clicks (Indonesia)** ✅ (cimbniaga.co.id branchless-banking page, extracted this pass) — Indonesia's digital proposition is branded **OCTO Clicks**, with a "new OCTO Clicks" release featuring a Financial Check-Up dashboard (portfolio summary, cash flow, goals) — evidence that the OCTO family is implemented per-market with local product depth rather than as a single global app ✅/⚠ (shared-engine vs per-market-build question — §12 item 2).
- **Touch 'n Go / TnG Digital (Malaysia)** ✅/⚠ — the FY2025 release names Touch 'n Go Digital among the group's front-ends ✅; TnG Digital (the e-wallet operator, majority-owned under Touch 'n Go Sdn Bhd ⚠ exact split) is overseen by Gurdip Singh Sidhu, CEO of CIMB Malaysia and CIMB Bank Berhad, who "also oversees the group's digital businesses" ✅ (TnG newsroom/press quotes, extracted this pass); an August-era report of a possible TnG Digital listing drew a "not in a hurry" response from the company ✅ (same source) — the wallet is the group's mass-retail front end for QR payments and the DuitNow ecosystem (cross-ref [Payment Rails Guide](payment_rails_guide.md)).
- **CIMB Bank Philippines** ✅/⚠ — the BSP-licensed digital retail bank (launched December 2018, cross-ref §1.4) is by definition an app-first bank ⚠ (its consumer app and engine are not covered by this pass's sources — §12 item 10).
- **CIMB Clicks (legacy web, MY)** ⚠ — the historical Malaysian internet-banking brand, superseded for mobile by OCTO; current status as the MY desktop web channel is standard practice ⚠ (not re-verified).

### 5.3 The App Family at a Glance

| Front end | Market | Public facts | Status |
|---|---|---|---|
| CIMB OCTO (consumer) | Malaysia | Launched 2022 on "new technology stack"; FY25 "next generation ready" | ✅ |
| OCTO by CIMB Singapore | Singapore | Mobile app live; SG product page + cimbclicks portal | ✅ |
| CIMB Clicks (web) | Singapore (and legacy MY) | cimbclicks.com.sg online banking portal live | ✅ SG; ⚠ MY legacy |
| OCTO Biz | Regional | SME app; regional roll-out per 2Q26; transaction-banking vehicle | ✅ |
| OCTO Clicks | Indonesia | CIMB Niaga digital proposition; new release w/ Financial Check-Up | ✅ |
| Touch 'n Go / TnG Digital | Malaysia | E-wallet front end; group digital businesses under CIMB Malaysia CEO | ✅/⚠ ownership |
| CIMB Bank Philippines app | Philippines | Digital bank since Dec 2018 | ✅ entity; ⚠ app/engine |

The digital-channel record is the strongest in CIMB's disclosure set — **every market's app brand is public, and the group tells you which ones it considers next-generation** — but it stops exactly at the app boundary. Whether OCTO Malaysia and OCTO Singapore are one codebase or two, and what sits behind OCTO Biz's regional roll-out, are §12 items. The channel-layer conventions of the genre are in the [Maybank guide](maybank_software_systems_guide.md) §5 and the [DBS guide](dbs_software_systems_guide.md).

---

## 6. Data and AI

### 6.1 The Public Numbers

CIMB's data-and-AI disclosure is unusually quantitative for a Malaysian bank, and almost entirely spend- and outcome-flavoured rather than architecture-flavoured:

- **RM100+ million on AI training in 2025** ✅ (FY2025 results release) — the headline AI figure the group reports is a *capability* number (training and upskilling its people), not an infrastructure number.
- **RM1.7 billion technology investment in FY2025** ✅ (same release) — the envelope that contains the AI work, the core-modernisation programme (§4) and the digital channels (§5).
- **F30 Capabilities pillar couples "data & AI" with core modernisation** ✅ (Forward30 page) — the group's own words, making data/AI a first-class strategy pillar rather than a technology-department initiative.
- **The December 2025 blockchain/tokenisation announcement and the August 2026 tokenised-sukuk pilot** ✅ (newsroom releases; §2.6, §7) — the group's distributed-ledger ambition is public and dated, and CIMB's own pilot description names the internal coalitions involved — "Islamic finance, capital markets, transaction banking, technology, legal documentation, custody and settlement" — a rare glimpse of how the group organises technology work across silos ✅.

### 6.2 What the Numbers Do Not Say

- **No named AI platform or model vendor** ⚠ — the trail found no public CIMB statement naming its AI/ML platform, its data lakehouse, or a hyperscaler partnership; the searches for named partners came back empty ✅/⚠ (absence of record — §12 item 11).
- **No architecture narrative** ⚠ — no public CIMB equivalent of a "data platform on X" statement; the group's IAR2025 (available on the IR site) would carry the governance detail but was not extractable page-by-page in this pass (§14 lists it for the reader).
- **Use-case specifics are inference** ⚠ — that the AI investment lands on the F30 priorities (CASA growth, cross-sell, risk, the SME engine via OCTO Biz) is a structural inference from the strategy layer, not a disclosed roadmap.

### 6.3 Data and AI at a Glance

| Item | Public fact | Evidence |
|---|---|---|
| AI capability spend | RM100+ million on AI training (2025) | ✅ FY2025 release |
| Total tech envelope | RM1.7 billion (FY2025) | ✅ FY2025 release |
| Strategic frame | "modernise core banking technology, and data & AI" (F30 Capabilities) | ✅ Forward30 page |
| DLT/tokenisation | Dec 2025 announcement; Aug 2026 tokenised-sukuk pilot | ✅ newsroom |
| Platform/partners | None named | ⚠ §12 item 11 |
| Use-case map | Inferred from F30 priorities | ⚠ inference |

The data-and-AI section is short because the honest record is short: **CIMB reports data-and-AI spend at a level most Malaysian peers do not, and architectural substance at a level this series cannot verify.** The governance conventions (BCBS 239-class data risk, model risk management) are cross-referenced rather than re-derived — see the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) and the [Maybank guide](maybank_software_systems_guide.md) §6.

---

## 7. Islamic Finance Systems: CIMB Islamic

### 7.1 Lineage and Naming

CIMB Islamic's lineage is one of the cleanest in the group, and the naming sequence is now verified end-to-end:

- **June 2003** — **CIMB Islamic is launched**, officiated by Bank Negara Governor Tan Sri Dr Zeti Akhtar Aziz ✅ (CIMB heritage; Wikipedia, cross-ref §1.3).
- **April 2005** — **Commerce-Tijari Bank Bhd** is launched as the Islamic banking subsidiary of Bumiputra-Commerce Bank, "providing the consumer market with an Islamic alternative for deposit accounts and financing" ✅ (CIMB 2000s heritage page *and* CIMB Islamic's own 2000s history page, both extracted this pass) — the Commerce-Tijari name was the transitional corporate that became the modern **CIMB Islamic Bank Berhad** ✅/⚠ (the "Tijari" era is verified; the exact renaming date to CIMB Islamic Bank Berhad is a §12 item).
- **The licensed-entity present** ✅ — CIMB Islamic Bank Berhad appears on the group's leadership/entity pages (§1.6) and was co-subject of the July 2024 BNM administrative penalty under the **Islamic Financial Services Act 2013 (IFSA 2013)** alongside CIMB Bank Berhad under the FSA 2013 ✅ (BNM enforcement notice — §8.1) — the IFSA reference confirms the separate Islamic-bank licence the systems layer must support.

### 7.2 Scale and Product Layer

- **RM251.2 billion of total Islamic assets** as at 30 June 2026 ✅ (cimb.com About Us, cross-ref §1.4) — roughly 32% of group assets (derived: 251.2 ÷ 794.9 ⚠ arithmetic on verified inputs), making CIMB one of the world's largest Islamic-banking franchises by balance sheet.
- **The product canon is standard Malaysian Islamic banking** ✅/⚠ — deposits (savings/current/fixed under wadiah/mudharabah contracts) and financing (home, auto, personal, cards, SME under murabahah/bai'/ijarah-family contracts), plus Islamic capital-markets and transaction-banking offerings ⚠ (product families structural; the per-product Shariah-parameterisation map is not public — cross-ref the [Maybank guide](maybank_software_systems_guide.md) §7, whose product-to-system table is the genre convention, and the repo's Islamic-finance treatment there by name only).
- **Awards and rankings** ⚠ — CIMB Islamic's own history pages reference industry recognition (Euromoney-class Islamic-finance awards) ✅/⚠ (award mentions present on the extracted history page; the specific award list was not captured this pass).

### 7.3 The Systems Layer and the Tokenisation Edge

- **The engine question** ⚠ — as with the conventional estate (§4), CIMB does not name CIMB Islamic's core; the Shariah-compliant parameterisation (profit-rate engine, contract templates, Shariah-audit trails) is inferred to sit on or beside the group's core estate ⚠ structural (separate IFSA 2013 licence → separate books → at least a separate ledger instance) — §12 item 1.
- **The August 2026 tokenised-sukuk settlement pilot** ✅ (CIMB newsroom, cross-ref §2.6) — the group's flagship Islamic-systems datapoint: a pilot settling tokenised sukuk, which CIMB itself frames as spanning "Islamic finance, capital markets, transaction banking, technology, legal documentation, custody and settlement" ✅ — the pilot's platform (permissioned DLT vs existing CSD rails) is not public ⚠ (§12 item 12).
- **The BNM AMP compliance datapoint** ✅ — CIMB Islamic Bank Berhad's joint penalty with CIMB Bank (July 2024, IFSA 2013 non-compliance) shows the Islamic entity is in scope of the same operational-resilience supervision as the conventional bank (§8.1).

### 7.4 CIMB Islamic at a Glance

| Layer | Public facts | Systems note | Status |
|---|---|---|---|
| Lineage | CIMB Islamic launched June 2003; Commerce-Tijari Bank 2005 → CIMB Islamic Bank Berhad | — | ✅ (rename date ⚠) |
| Scale | Islamic assets RM251.2B (30 Jun 2026); ~32% of group (derived) | — | ✅ |
| Licence | Separate IFSA 2013 Islamic bank licence (BNM AMP co-subject, 2024) | Separate ledger instance implied | ✅ licence; ⚠ engine |
| Products | Deposit/financing canon, Islamic capital markets | Shariah parameterisation ⚠ | ✅ products; ⚠ engines |
| Digital edge | Aug 2026 tokenised-sukuk settlement pilot | Platform not public | ✅ pilot; ⚠ platform |

Islamic finance is where CIMB's public narrative and its systems reality diverge least — **the lineage, the licence and the balance sheet are all verified, and the tokenised-sukuk pilot gives the Islamic arm the group's most concrete innovation datapoint** — but the ledger that runs RM251 billion of Islamic assets remains as unnamed as its conventional sibling's.

---

## 8. Risk and Regulatory Context

### 8.1 The Home Supervisor: BNM and the Enforcement Record

The research pass made a specific effort to establish CIMB's public enforcement record under Bank Negara Malaysia, and found one material modern action:

- **July 2024 Administrative Monetary Penalty (AMP)** ✅ (BNM enforcement notice ea-cimb24-bm, extracted this pass; The Edge Malaysia) — Bank Negara imposed an AMP on **CIMB Bank Berhad and CIMB Islamic Bank Berhad** for non-compliance with the **Financial Services Act 2013** and the **Islamic Financial Services Act 2013**, following **prolonged service disruptions**; the penalty was **RM760,000** in aggregate terms per The Edge's contemporaneous report, which also noted Maybank was fined RM4.32 million in the same action set ✅/⚠ (BNM notice title/URL verified by extraction; the exact per-entity apportionment of the RM760,000 between the two CIMB entities is a §12 item).
- **What the action signals for this guide** ✅/⚠ — a service-disruption AMP is an *operational-resilience* enforcement: it lands on the systems estate this guide documents (channel availability, incident management), not on a conduct or AML matter. It is the regulatory footprint of the legacy-modernisation tension of §4, and it cross-references the resilience themes of the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md).
- **No other BNM AMP/enforcement naming CIMB was found in this pass** ✅/⚠ (absence of record, not exoneration; the pass's targeted searches for older penalties returned nothing usable — §12 item 13). No public BNM or MAS action connecting CIMB to the 1MDB matter surfaced in this pass ⚠ (absence noted neutrally).

### 8.2 MAS and the Singapore Franchise

- **MAS supervision of the branch** ✅ (MAS Financial Institutions Directory entry 3336, extracted this pass) — CIMB Bank Berhad's Singapore operation is MAS-supervised as a licensed bank branch with exempt CMS/financial-adviser/trust status under the bank's licence (full detail in §9.1). The MAS angle for a branch is prudential (the Malaysian parent's capital — CET1 14.3% at FY25 ✅) plus the conduct/AML obligations that apply to the SG booking centre ⚠ structural.
- **No MAS public enforcement against CIMB Singapore found this pass** ✅/⚠ (searches returned nothing usable — §12 item 13). The compliance context — MAS AML/CFT expectations, the cross-border QR and PayNow–DuitNow rules, technology-risk management — is cross-referenced to the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) and [FircoSoft Guide](fircosoft_guide.md) rather than re-derived.

### 8.3 The Wider Risk Picture

| Theme | CIMB-specific public facts | Status |
|---|---|---|
| Capital | CET1 14.3% (FY25); F30 target CET1 14–15% (2027), 13–14% (2030) | ✅ |
| Credit risk | F30 credit-cost target 30–40 bps | ✅ |
| Profit quality | Malaysia 61% of FY25 PBT; ROE 11.3% (FY25), 11.2% (2Q26) | ✅ |
| Operational resilience | July 2024 BNM AMP (RM760k, FSA/IFSA, service disruptions) | ✅ |
| Financial crime | Group AML/FCC estate unnamed; no public MY/SG enforcement found | ⚠ absence |
| Conduct/1MDB-era | No public action naming CIMB found this pass | ⚠ absence |
| Climate | Coal phase-out by 2040 commitment (2020); NZBA (2021) | ✅ |

The risk section condenses deliberately: this guide's genre places the deep regulatory mechanics in the dedicated repo guides ([Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [FircoSoft Guide](fircosoft_guide.md)) and keeps the per-bank record — one verified BNM AMP, a clean MAS record on the evidence gathered, and a capital position that funds the RM1.7-billion-a-year technology programme — as the CIMB-specific contribution.

---

## 9. Singapore Angle: The Branch, the Hub and the Corridor

### 9.1 The Entity and the Licence: A Branch, Verified

The single most important structural fact about CIMB in Singapore is that **there is no CIMB Singapore subsidiary** — the franchise is a **branch of CIMB Bank Berhad**, the Malaysian licensed bank, and it is supervised by MAS on that basis ✅ (MAS Financial Institutions Directory entry 3336, extracted this pass). The directory record adds the licence detail ✅/⚠ (as extracted):

| FID field | Recorded value | Status |
|---|---|---|
| Institution | CIMB Bank Berhad (Singapore branch operations) | ✅ |
| MAS licence class | Licensed bank — Full Bank (branch) | ✅ (as extracted) |
| Exemptions | Exempt from CMS licence, financial-adviser and trust-licensing requirements under the bank licence | ✅ (as extracted) |
| Head (SG) | Joon Nien Mak (as listed on the FID record) | ✅ (as extracted) |
| Address | Raffles Place-class SG principal office | ✅ (as extracted) |

The Full-Bank licence is notable: CIMB's branch holds the same licence class MAS grants domestic banks, with the retail permissions that implies — consistent with the live consumer business (§3, §5) — rather than a restricted wholesale-bank licence ⚠ (the task-level guess of "wholesale bank" is corrected by the FID record extracted in the research pass; licence nuances per the [Banks in Singapore Guide](banks_in_singapore_guide.md) and [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)). The branch structure also explains §1.6's observation: Singapore has no seat on the group's entity leadership list because it is not an entity.

### 9.2 The SG↔MY Corridor: The Product That Defines the Franchise

The branch's flagship consumer product is the **instant SGD↔MYR corridor** between CIMB Singapore and CIMB Malaysia accounts ✅ (cimb.com.sg and cimb.com.my product pages, extracted this pass; full mechanics in §2.5) — an on-us, group-internal instant cross-border transfer at a "guaranteed best exchange rate", with the Malaysian side advertising zero-fee transfers up to RM50,000. For the systems map this product is the branch's reason to exist digitally: it monetises the group's two ledgers (SG branch books and MY core) against each other without any external rail ⚠ (hub design not public — §12 item 2), and it is the retail edge of the group's corridor strategy — "strategic cross-border corridors... JS-SEZ and ASEAN Financial Passport, China-ASEAN partnership" ✅ (2Q26 release) — with Singapore cast as the group's ASEAN corporate hub ✅ (2Q26 release language).

### 9.3 The Securities Saga: GK Goh → CIMB-GK → CGS

Singapore is also where CIMB's most visible ownership exit played out, and the sequence is now fully verified:

- **2005** — CIMB acquires **GK Goh Securities** (founded 1979), forming **CIMB-GK Securities** ✅ (§1.3; CIMB heritage).
- **2017–2018** — **China Galaxy Securities** becomes a 50:50 shareholder of CIMB Securities International Pte Ltd (Singapore-incorporated); the JV commences stockbroking operations **18 January 2018** under the CIMB–China Galaxy banner ✅ (CIMB newsroom release, extracted this pass).
- **2019–2021** — China Galaxy progressively raises its stake; by **December 2021 CIMB's interest is down to ~25%** ✅/⚠ (The Edge Malaysia report, extracted this pass; ~25% figure as reported).
- **2022–2024** — the JV rebrands through **CGS-CIMB Securities** to **CGS International** (CGS International Holdings Limited, formerly China Galaxy International Financial Holdings Limited — name change effective **8 December 2022** ✅ asiabiztoday); **CGS Holdings completes its acquisition of CGS-CIMB Securities in January 2024** ✅ (asiabiztoday, 3 Jan 2024, extracted this pass) — completing China Galaxy's path to full ownership.

**Net position for the SG map**: the securities/stockbroking link that CIMB ran in Singapore from 2005 is, as of 2024, **CGS International — a China Galaxy business**, not a CIMB one ✅ (JV exit verified); residual CIMB securities entities and any retained minority interests are §12 items. What CIMB Singapore itself runs today is the wholesale/corporate franchise, the deposit-and-card retail business (§3, §5), the wealth tier (§3.2), and the corridor product (§9.2).

### 9.4 What the Branch Means for Systems

| SG systems question | Position | Evidence |
|---|---|---|
| Core hosting | Branch of CIMB Bank Berhad — books on the MY estate or a regional ledger instance | ⚠ inference (§4.4) |
| Channels | OCTO by CIMB Singapore (mobile) + CIMB Clicks web portal | ✅ product pages |
| Corridor engine | On-us SG↔MY instant transfers; hub design unnamed | ✅ product; ⚠ mechanics |
| Corporate banking | Wholesale/corporate franchise via the branch; BizChannel-class access ⚠ | ⚠ structural |
| Securities | Exited to CGS International (Jan 2024) | ✅ |
| Wealth | Preferred Banking tier; CIMB Private Wealth ASEAN ambition (SG role ⚠) | ✅/⚠ |
| SG PBT | Not disclosed separately | ⚠ §12 item 14 |

The Singapore angle closes with the branch's own history doing the summary: **a 1947-era Ban Hin Lee lineage, a 2009 consumer launch, a 2017–2024 securities exit, and a 2026 reality in which the branch's systems identity is the corridor product and the OCTO app** — the smallest big-bank franchise in this series, and the one whose public systems record is almost entirely product-layer.

### 9.5 The Singapore Timeline at a Glance

| Year | SG milestone | Systems relevance | Status |
|---|---|---|---|
| 1947 | CIMB claims SG presence "through Ban Hin Lee Bank" | Deepest SG root; event detail unverified | ✅/⚠ |
| 1979 | GK Goh Securities founded (SG) | The house CIMB will buy in 2005 | ✅ |
| 2000 | Ban Hin Lee absorbed into Southern Bank group (MY-side consolidation) | BHL's SG operations fold into the lineage | ✅ |
| 2005 | CIMB acquires GK Goh → CIMB-GK Securities | SG securities anchor; international IB platform | ✅ |
| Sept 2009 | CIMB Bank Singapore consumer-banking launch | Retail deposits/cards/channels begin | ✅ |
| 2017 | China Galaxy takes 50:50 of CIMB Securities International | Securities JV era opens | ✅/⚠ |
| Jan 2018 | JV commences stockbroking operations | CIMB–China Galaxy brokerage live | ✅ |
| Dec 2021 | CIMB diluted to ~25% as China Galaxy raises stake | Exit trajectory visible | ✅/⚠ |
| Dec 2022 | CGS International name change effective | Brand transition mid-flight | ✅ |
| Jan 2024 | CGS Holdings completes CGS-CIMB acquisition | CIMB's SG securities era closes | ✅ |
| 2022–2026 | OCTO by CIMB Singapore + CIMB Clicks portal live; instant SG↔MY corridor marketed | The branch's modern systems identity | ✅ |

The timeline shows a franchise that has *subtracted* its way to focus: out of securities (2024), into deposits, cards, wealth and the corridor — with the corridor product doing the strategic work of keeping the branch relevant to the group's F30 ASEAN-corporate-hub ambitions (§9.2). What the timeline cannot show is the systems estate underneath, because the branch keeps its books inside CIMB Bank Berhad's undisclosed core arrangements (§4.4, §12 item 1) and its profit inside the group's undisclosed SG split (§12 item 14).

---

## 10. Worked Example: Cymbal Bank × CIMB — A Regional Correspondent Relationship

### 10.1 The Relationship

**Cymbal Bank** — the fictional regional challenger-and-correspondent bank that appears throughout this series — wants to give its corporate clients clean access to Malaysia and, through the corridor, Singapore. It picks **CIMB Bank Berhad** as its Malaysian/ASEAN correspondent for three reasons the guide's earlier sections make legible: CIMB's Malaysian universal-bank scale and 576-branch-class network ✅ (4Q25 release boilerplate; dated), the on-us **SG↔MY corridor** (§9.2) that lets one CIMB relationship cover both markets, and CIMB's named transaction-banking focus under F30 with OCTO Biz as the regional channel (§2.2). Cymbal Bank is not a CIMB group entity and has no ownership link; the relationship is pure correspondent banking — the genre's standard arms-length structure.

### 10.2 The Moving Parts

| # | Relationship component | Cymbal side | CIMB side | Instrument / rail | Reference |
|---|---|---|---|---|---|
| 1 | SGD nostro | Cymbal holds a SGD nostro at CIMB's Singapore branch | Branch credits/debits the nostro | SWIFT MT103/MT202 over the branch's BIC | §9, [Payment Rails Guide](payment_rails_guide.md) |
| 2 | MYR nostro | Cymbal holds a MYR nostro at CIMB Bank Berhad (KL) | Malaysian core posts nostro entries | SWIFT MT103/202; MEPS+ settlement behind CIMB | §2.4 |
| 3 | Vostro | CIMB holds a MYR vostro at Cymbal (reciprocal) | Cymbal's core runs the vostro | Standard nostro/vostro mirror posting | genre standard |
| 4 | Corridor piggyback | Cymbal SG↔MY client flows ride CIMB's on-us corridor economics | On-us posting between SG branch and MY core | Group-internal instant transfer | §9.2, §2.5 |
| 5 | Real-time MYR receipt | Cymbal's clients paid by Malaysian corporates | CIMB credits via DuitNow-class instant rails | DuitNow proxy/QR interoperability | §2.4, [Payment Rails Guide](payment_rails_guide.md) |
| 6 | PayNow–DuitNow linkage | Cymbal SGD receipts into Singapore | CIMB SG branch participates in the SG↔MY linkage corridor | PayNow ↔ DuitNow cross-border | [Nets Singapore Guide](nets_singapore_guide.md) |
| 7 | KYC/CDD | EWRA-style due-diligence file on CIMB | RMA/AML counterparty file on Cymbal | SWIFT RMA + screening | [FircoSoft Guide](fircosoft_guide.md) |
| 8 | Documentary credit | Cymbal's importer clients need LCs to Malaysian sellers | CIMB advises/confirms over MT7xx | UCP600-governed LCs; MT798 for open account | [Trade Finance Guide](trade_finance_guide.md) |
| 9 | Collections | Export collections from MY buyers | CIMB presents documents as collecting bank | MT4xx documentary collections | same |
| 10 | Account services | Cymbal's monthly nostro reconciliation | CIMB statementing via SWIFT MT940/950 | SWIFT statement messages | [Payment Rails Guide](payment_rails_guide.md) |

### 10.3 Nostro/Vostro and the Two-Country Posting Model

The relationship's spine is two nostro accounts. **SGD leg**: Cymbal Bank maintains its SGD nostro at CIMB's Singapore branch; when a Cymbal client needs SGD in Singapore — a payroll run for its Singapore subsidiary — Cymbal debits the client and sends a SWIFT **MT103** (single customer credit transfer) to the branch's BIC, covering from the nostro balance; the branch credits the beneficiary and posts the nostro debit. **MYR leg**: the mirror-image arrangement runs at CIMB Bank Berhad's Malaysian head office, where Cymbal's MYR nostro funds client payments to Malaysian beneficiaries; CIMB credits the beneficiary on its own core and settles internally. Because both legs sit with one banking group, Cymbal gets a single relationship manager, one set of RMA keys, and — the differentiator — the group's **on-us corridor economics** (§9.2) when a flow needs to move value between the SGD and MYR legs quickly. What Cymbal does *not* get is a look inside CIMB's hub: the group-internal posting that makes the corridor instant is CIMB's own estate (§12 item 2), and Cymbal simply sees two clean nostro credits and debits. The SWIFT/ISO 20022 mechanics of these messages are the [Payment Rails Guide](payment_rails_guide.md)'s subject, not re-derived here; the same holds for GPI-class tracking, which the relationship would use for the MT103 leg ✅/⚠ (GPI membership structural for a bank of CIMB's class — §12 item 9).

### 10.4 The Real-Time Layer: DuitNow, PayNow and the Linkage

Where the relationship modernises is the retail/SME real-time layer. Cymbal's Malaysian-domiciled corporate clients want to pay and be paid instantly, and CIMB's answer is its DuitNow-class participation (§2.4 ⚠ per-rail membership). The flow: a Cymbal client's Malaysian customer initiates a DuitNow proxy-ID transfer; CIMB credits the Cymbal client's account on its core in seconds and the corresponding MYR value reaches Cymbal's nostro through the daily settlement cycle. In the other direction, Cymbal's SG clients receiving SGD from Singapore counterparties benefit from CIMB Singapore's place in the **PayNow–DuitNow linkage** — the SG↔MY instant corridor the [Nets Singapore Guide](nets_singapore_guide.md) documents — which for Cymbal means its clients' Malaysian payers can push funds into SG accounts through the linked rails. None of this requires Cymbal to join any Malaysian rail itself: the correspondent structure lets Cymbal resell Malaysian real-time receipt as a product feature, exactly the pattern §10.2's rows 5–6 capture.

### 10.5 KYC and Correspondent Due Diligence

Before the first MT103, the two banks complete the correspondent-banking choreography this series documents in the [FircoSoft Guide](fircosoft_guide.md): Cymbal's financial-crime team builds an EWRA-style risk file on CIMB Group (ownership — Khazanah-led ✅; footprint — the ten ASEAN markets of §1.4; the public enforcement record — the July 2024 BNM AMP of §8.1 goes into the risk narrative as a resilience item, not a financial-crime one), screening and negative-news checks run on the CIMB legal entities it will transact with (CIMB Bank Berhad and the Singapore branch), SWIFT **RMA** keys are exchanged and maintained per-message-type, and CIMB's compliance team runs the mirror-image file on Cymbal. Both sides monitor the nostro flows for velocity anomalies; the July-2024 AMP is exactly the kind of datapoint correspondent banks share in due-diligence questionnaires, and CIMB's disclosure of it in the BNM notice makes the file easy to complete.

### 10.6 Documentary Credit Over the Relationship

For trade flows, Cymbal's importer clients open **irrevocable LCs** in favour of Malaysian sellers, advised through CIMB Bank Berhad under **UCP600** (the industry-standard ICC rules — genre knowledge, cross-ref [Trade Finance Guide](trade_finance_guide.md)). The LC travels as an **MT700** from Cymbal to CIMB; CIMB advises the beneficiary and, on presentation of compliant documents, claims reimbursement from Cymbal's MYR nostro by MT202 or through the LC's reimbursement clause; discrepant presentations come back as MT734 rejections; collections run MT4xx. CIMB's JS-SEZ trade-finance proposition (§2.6) is the commercial backdrop — the corridor's industrial estates generate exactly these LC flows — and Cymbal's clients effectively rent CIMB's Malaysian trade infrastructure through the correspondent file rather than building their own. For Cymbal, the CIMB relationship collapses what would otherwise be two country relationships (MY + SG) into one contractual and technical file — which is the worked example's whole point.

### 10.7 The Shape of the Relationship

The relationship's architecture in one paragraph: **Cymbal Bank runs two nostros and one RMA file against one counterparty — CIMB Bank Berhad, with its Singapore branch as the SGD booking point — and through that single file reaches Malaysian real-time rails, the SG↔MY instant corridor, and UCP600-governed trade services.** The moving parts Cymbal can see are all standard correspondent banking (MT103/202, MT7xx, RMA, nostro reconciliation via MT940); the moving part it cannot see — the group-internal hub that makes CIMB's own SG↔MY corridor instant — is precisely the layer this guide has marked ⚠ from §2 to §9.

### 10.8 The End-to-End Flow: A Cymbal Client Pays a Johor Supplier

The relationship in motion, step by step (mechanics in the genre's standard terms; the CIMB-specific reference points are cited):

1. **Instruction** — a Cymbal corporate client instructs a MYR payment to its supplier's account at a Johor bank (or at CIMB itself), via Cymbal's own internet-banking channel.
2. **Cymbal-side processing** — Cymbal screens the payment (sanctions/name screening per its [FircoSoft Guide](fircosoft_guide.md)-class stack), debits the client (SGD or MYR), converts if needed at Cymbal's rate, and formats a SWIFT **MT103** to CIMB Bank Berhad's Malaysian BIC, covering the value from Cymbal's **MYR nostro** (§10.3). For urgent/large-value flows Cymbal uses **MT202** (cover payment) ahead of the MT103.
3. **CIMB receipt and posting** — CIMB validates the message against the exchanged **RMA** keys, screens it on its own financial-crime layer (un-named ⚠ §12 item 7), credits the beneficiary's account on the Malaysian core, and posts the debit to Cymbal's nostro. GPI-class tracking confirms the credit to Cymbal in minutes ✅/⚠ (structural for CIMB's class; §12 item 9).
4. **Same-group SGD leg (when needed)** — if the client's underlying trade is with a Singapore counterparty, Cymbal instead sends the MT103 to **CIMB's Singapore branch BIC**; the branch posts the SGD credit and, where the client needs the value in MYR, the group's on-us corridor mechanics (§9.2) convert and credit the Malaysian beneficiary without Cymbal touching a second correspondent — the single-file advantage of §10.7.
5. **Real-time receipts** — a Johor customer of a Cymbal client pays via **DuitNow** proxy-ID; CIMB credits the Cymbal client's CIMB account instantly and Cymbal's nostro receives the value in the settlement cycle; Cymbal's SG-domiciled clients receive SGD pushed through the **PayNow–DuitNow linkage** (§10.4).
6. **Trade leg** — for the LC-backed shipment that the payment settles, Cymbal issued an **MT700** (§10.6); CIMB advised it to the Johor beneficiary; documents were presented, checked under **UCP600**, and the reimbursement claimed against Cymbal's nostro; the MT103 in step 3 may in fact be the drawing under that LC, which makes the trade and payments legs one continuous file.
7. **Reconciliation** — each morning Cymbal's back office reconciles both nostros against CIMB's **MT940** statements, posts the mirror **vostro** entries on its own core (CIMB's MYR account at Cymbal), and closes the prior day's nostro breaks.

### 10.9 The Relationship Scorecard

| Dimension | Arrangement | CIMB evidence base |
|---|---|---|
| Coverage | MY + SG through one counterparty file | Branch structure ✅ (§9.1); corridor ✅ (§9.2) |
| Rails | SWIFT MT103/202; GPI-class tracking; DuitNow/PayNow–DuitNow receipt | Structural ✅/⚠ (§2.4, §10.3) |
| Counterparty risk | EWRA-style file; July-2024 AMP as disclosed resilience item | ✅ (§8.1) |
| Trade | UCP600 LCs over MT7xx; JS-SEZ corridor flows | ✅/⚠ (§2.6) |
| Blind spots | CIMB's hub internals, engine identities, SG split | ⚠ (§12 items 1–2, 14) |

For Cymbal Bank the relationship is a textbook correspondent file with one unusual property: the counterparty's own internal corridor lets Cymbal sell Malaysia-plus-Singapore as a single product. For CIMB it is the F30 transaction-banking strategy (§2.2) working exactly as announced — correspondent balances, OCTO Biz-adjacent SME flows, and corridor volume landing on the same undisclosed engine layer this guide has traced from §2 to §10.

---

## 11. Claims Status and Verification Notes

### 11.1 The Audit Table

The table below audits every load-bearing claim of this guide against the evidence gathered in the research pass, mirroring the §1.5 convention (✅ verified / ⚠ flagged / ❌ disputed).

| # | Claim | Status | Evidence |
|---|---|---|---|
| 1 | Group roots to 1924 Bian Chiang Bank; 1935 Ban Hin Lee; 1965 Bank Bumiputra/Southern | ✅ | CIMB heritage pages (decade by decade), extracted this pass |
| 2 | CIMB = Commerce International Merchant Bankers (1986 renaming of PBSM, est. April 1974 JV) | ✅ | CIMB heritage; Wikipedia |
| 3 | 1999 Bumiputra-Commerce merger; 2006 universal-bank launch ("Forward Banking") | ✅ | CIMB heritage |
| 4 | CAHB renamed Bumiputra-Commerce Holdings (2005), then CIMB Group Holdings (EGM approval sought Sept 2009) | ✅/⚠ | The Edge Malaysia headline verified; completion date ⚠ (§11.2) |
| 5 | CIMB Singapore presence since 1947 "through Ban Hin Lee Bank" | ✅/⚠ | CIMB SG-presence claim; 1947 event detail unverified |
| 6 | GK Goh Securities acquired 2005 → CIMB-GK | ✅ | CIMB heritage; Wikipedia |
| 7 | CIMB Bank Singapore consumer banking launched September 2009 | ✅ | CIMB heritage |
| 8 | SG franchise = branch of CIMB Bank Berhad under a MAS Full-Bank licence (FID 3336), exempt CMS/FA/trust | ✅ | MAS Financial Institutions Directory, extracted this pass |
| 9 | CIMB OCTO app introduced 2022 on a "new technology stack"; OCTO by CIMB Singapore live; CIMB Clicks SG portal live | ✅ | CIMB heritage 2020s; cimb.com.sg; cimbclicks.com.sg |
| 10 | Instant SG↔MY on-us corridor product (guaranteed rate; zero-fee to RM50,000 MY-side) | ✅ product | cimb.com.sg / cimb.com.my product pages |
| 11 | FY2025: net profit RM7.9B, ROE 11.3%, CET1 14.3%, RM1.7B tech spend, RM100M+ AI training | ✅ | CIMB FY2025 results release (27 Feb 2026) |
| 12 | 2Q26: net profit RM1.94B, ROE 11.2%; OCTO Biz regional roll-out; corridor strategy language | ✅ | CIMB 2Q26 release |
| 13 | Group scale: assets RM794.9B; Islamic assets RM251.2B (30 Jun 2026); 10 markets; 32,000+ staff | ✅ | cimb.com About Us |
| 14 | CIMB Islamic launched June 2003; Commerce-Tijari Bank (Apr 2005) → CIMB Islamic Bank Berhad | ✅ (rename date ⚠) | CIMB heritage; CIMB Islamic history pages |
| 15 | Aug 2026 tokenised-sukuk settlement pilot | ✅ | CIMB newsroom |
| 16 | July 2024 BNM AMP on CIMB Bank + CIMB Islamic (FSA/IFSA 2013), RM760,000, service disruptions | ✅/⚠ | BNM notice ea-cimb24-bm; The Edge (apportionment ⚠) |
| 17 | CIMB Niaga migrated core workflows to Pega Cloud (completed 26 Feb 2026), Virtusa-led | ✅/⚠ | briefglance/Virtusa pulse (secondary) |
| 18 | China Galaxy 50:50 JV (2018); CIMB diluted to ~25% (Dec 2021); CGS Holdings completed CGS-CIMB acquisition (Jan 2024) | ✅ | CIMB newsroom 2018; The Edge; asiabiztoday |
| 19 | Khazanah Nasional largest shareholder (with EPF & KWAP), as at 30 Jun 2026 | ✅ | CIMB shareholding-information page |
| 20 | Group core-banking engine vendors not publicly named | ✅ (negative claim) | Search sweep returned no CIMB statement (§12 item 1) |
| 21 | CIMB = Malaysia's second-largest bank | ✅/⚠ | Consistent with CIMB positioning; dated third-party rankings |
| 22 | ~1,080 branches (Wikipedia infobox) vs 576 in 4Q25 release boilerplate | ❌/⚠ | Conflicting dated figures; §11.2 note |

### 11.2 Notes on the Two Flagged Items

- **Row 4 (rename timing):** The Edge Malaysia's report — "Bumiputra-Commerce seeks to be renamed CIMB Group Holdings" — dates the shareholder-approval step to September 2009 ✅ (headline/article verified via search extract); the effective registration date of the new name was not independently verified ⚠. §1.4's earlier ⚠ on the rename year is resolved to September 2009 (approval sought) ✅/⚠.
- **Row 22 (branch count):** the 4Q25 press-release boilerplate's **576 branches** (Malaysia, dated December 2025) and Wikipedia's **~1,080** (undated infobox) do not reconcile; the guide uses the dated release figure where branch count appears and flags the Wikipedia figure as stale-class ⚠.

### 11.3 What the Audit Shows

Twenty of twenty-two load-bearing claims verified ✅; two carry date/scope flags; none is disputed on the evidence except the irreconcilable branch-count pair (flagged, not resolved). The verification failures that matter are the *negative* ones: no CIMB statement on core vendors, no SG PBT split, no per-rail membership roster — absences collected in §12, not errors in this guide.

---

## 12. What Could Not Be Verified

This section collects every materially non-public or under-verified item flagged ⚠ across the guide, so a reader (or a later research pass) knows exactly where the public record stops:

1. **Core-banking engine identities** (§4.1, §7.3) — the vendors/systems running CIMB Bank Berhad's Malaysian retail core, CIMB Islamic's ledger, CIMB Niaga's core (beyond the Pega Cloud workflow layer), and the Singapore branch books. CIMB has not said; searches for Silverlake/Temenos/Avaloq links returned nothing.
2. **OCTO architecture** (§4.2, §5.1, §9.2) — what the 2022 "new technology stack" actually comprises; whether OCTO MY/SG/ID share one engine or are per-market builds; the hub design behind the instant SG↔MY on-us corridor.
3. **Pega Cloud scope at CIMB Niaga** (§4.3) — secondary-source datapoint (briefglance/Virtusa pulse); CIMB-side confirmation and contract scope unverified.
4. **Card estate detail** (§3.1) — group card receivables/counts; per-market network and co-brand matrices.
5. **Affluent brand map** (§3.2) — SG "Preferred Banking" vs group CIMB Premier-class naming; exact tier structures per market.
6. **CIMB Private Wealth build-out** (§3.2) — the 2026 release's market/entity scope (SG branch vs Labuan vs Malaysia) not fully extracted.
7. **Middle-layer engines** (§2.7, §5.1) — payments hub, OCTO Biz API/host-to-host layer, trade-platform and AML/FCC platform identities.
8. **Trade platform product names** (§2.6) — no public CIMB trade-platform product name found.
9. **Per-rail membership rosters** (§2.4, §10.3) — item-by-item confirmation of DuitNow/FPX/MEPS+/GPI participation; membership is structurally near-certain for a bank of CIMB's class but was not roster-verified.
10. **CIMB Bank Philippines app/engine** (§5.2) — the digital bank's consumer app and core were outside this pass's sources.
11. **Data/AI platform and partners** (§6.2) — no named AI platform, data platform, or hyperscaler partnership found.
12. **Tokenised-sukuk platform** (§7.3) — the Aug 2026 pilot's underlying platform (permissioned DLT vs CSD rails) not public.
13. **Enforcement-record completeness** (§8.1–8.2) — only one BNM AMP and no MAS action found; older or non-English records may exist beyond the pass's search reach.
14. **SG profit split** (§9.4) — the group does not disclose Singapore-branch PBT separately; SG economics are inside the Malaysia/wholesale lines.
15. **Dated/conflicting figures** — branch count (576 vs ~1,080, §11.2), the Khazanah stake percentage, CIMB-Principal ownership split, TnG Digital ownership split, CIMB Islamic renaming date, Commerce-Tijari era details.
16. **Securities-entity residue** (§9.3) — residual CIMB securities entities and any retained minority interest after the Jan 2024 CGS completion.
17. **1947 Singapore event** (§1.3, §9) — the specific Ban Hin Lee event behind CIMB's "presence since 1947" claim.

---

## 13. Glossary

| Term | Meaning |
|---|---|
| **CIMB** | Commerce International Merchant Bankers — the 1986 renaming of PBSM; now the group brand |
| **PBSM** | Pertanian Baring Sanwa Multinational Bank, the 1974 four-way JV that became CIMB |
| **CAHB** | Commerce-Asset Holdings Bhd, the 1991 holding company of the consolidation era |
| **Bank Bumiputra** | 1965 bank merged with Bank of Commerce in 1999 into Bumiputra-Commerce Bank |
| **Ban Hin Lee Bank** | 1935 Penang bank (Yeap Chor Ee); acquired by Southern Bank 2000; source of CIMB's 1947 SG claim |
| **Southern Bank** | Penang bank (1965) absorbed into CIMB Group 2006; MEPS/ATM pioneer per heritage |
| **GK Goh / CIMB-GK** | Singapore securities house acquired 2005; the group's SG broking anchor until the CGS era |
| **CGS International** | China Galaxy's securities group; completed acquisition of CGS-CIMB Securities Jan 2024 |
| **CIMB OCTO** | Group's next-generation consumer mobile app (2022), MY + SG + regional family |
| **OCTO Biz** | Group SME/business app; regional roll-out under the 2Q26 transaction-banking focus |
| **OCTO Clicks** | CIMB Niaga's Indonesian digital-banking proposition |
| **CIMB Clicks** | Legacy internet-banking brand; live as the SG web portal (cimbclicks.com.sg) |
| **BizChannel@CIMB** | CIMB Malaysia's corporate cash-management channel; JS-SEZ remittance route |
| **TnG / TnG Digital** | Touch 'n Go e-wallet operator; group mass-retail front end in Malaysia |
| **MEPS / MEPS+** | Malaysian Electronic Payment System; the ATM/interbank network and RTGS-class settlement |
| **PayNet** | Payments Network Malaysia, operator of Malaysia's retail payment rails |
| **DuitNow** | Malaysia's real-time retail payment rail (proxy-ID transfers and QR) |
| **DuitNow QR** | National QR standard for instant payments in Malaysia |
| **FPX** | Financial Process Exchange — Malaysian online banking payments rail |
| **PayNow** | Singapore's real-time retail payment rail (proxy-ID) |
| **PayNow–DuitNow** | The SG↔MY cross-border instant payment linkage |
| **Nostro / Vostro** | "Our" account at another bank / "your" account held for another bank — the correspondent pair |
| **MT103 / MT202** | SWIFT messages: customer credit transfer / bank-to-bank transfer |
| **MT700 / MT7xx** | SWIFT documentary-credit messages (LC issue, amendments, rejections) |
| **MT940** | SWIFT account statement message (nostro reconciliation) |
| **UCP600** | ICC Uniform Customs and Practice for Documentary Credits (rev. 2007) |
| **SWIFT / RMA** | The messaging co-operative / Relationship Management Application key exchange |
| **GPI** | SWIFT Global Payments Innovation — tracked, same-day cross-border payments |
| **ISO 20022** | The global payments message standard succeeding MT |
| **CASA** | Current Account, Savings Account — the low-cost deposit base |
| **NOII** | Non-Interest Operating Income (fees, treasury, markets) |
| **PBT / ROE / CET1** | Profit Before Tax / Return on Equity / Common Equity Tier 1 capital ratio |
| **IBRA** | Indonesian Bank Restructuring Agency — seller of Bank Niaga's 51% in 2002 |
| **Khazanah** | Khazanah Nasional, Malaysia's sovereign strategic-investment arm; CIMB's largest shareholder |
| **EPF / KWAP** | Malaysia's Employees Provident Fund / pension fund — major CIMB holders |
| **BNM / MAS** | Bank Negara Malaysia / Monetary Authority of Singapore |
| **FSA 2013 / IFSA 2013** | Malaysia's Financial Services Act / Islamic Financial Services Act |
| **AMP** | Administrative Monetary Penalty (BNM enforcement tool) |
| **Bursa Malaysia / KLSE** | Malaysia's exchange (formerly Kuala Lumpur Stock Exchange) |
| **SET** | Stock Exchange of Thailand (CIMB Thai delisted Aug 2026) |
| **F30 / Forward Banking** | Forward30 strategy (2025–2030) / the 2006 tagline |
| **JS-SEZ** | Johor-Singapore Special Economic Zone — CIMB corridor priority |
| **ASEAN Financial Passport** | Regional cross-border banking-integration concept CIMB cites |
| **Commerce-Tijari Bank** | 2005 Islamic-banking subsidiary name of the Bumiputra-Commerce era → CIMB Islamic Bank Berhad |
| **Bank Niaga / Bank Lippo** | The Indonesian banks merged (2008) into CIMB Niaga |
| **CIMB Thai** | Thai bank (ex-BankThai); delisted from the SET August 2026 |
| **CIMB Bank Philippines** | BSP-licensed digital retail bank launched December 2018 — completed ASEAN-10 |
| **CIMB-Principal** | Asset-management JVs with Principal Financial Group (2004 conventional; 2008 Islamic) |
| **FastSaver** | CIMB Singapore's flagship high-yield savings product (live on cimb.com.sg) |
| **BIC** | Bank Identifier Code — SWIFT's bank-addressing scheme (used per branch/entity) |
| **MT798** | SWIFT message envelope for trade/LC messages between a bank and its customer |
| **EWRA** | Enhanced/expanded Wolfsberg-style risk assessment for correspondent relationships |
| **FID** | MAS Financial Institutions Directory — the register entry for CIMB Bank Berhad (3336) |
| **DCM** | Debt Capital Markets — CIMB's #1 ASEAN-Malaysia ranking niche (FY25: 32.5% MY share) |
| **NZBA** | Net Zero Banking Alliance — CIMB joined 2021 as first ASEAN bank |
| **NPS** | Net Promoter Score — F30 target of top-3 across markets by 2030 |

---

## 14. References and Further Reading

All URLs were visited or verified in the research pass (September 2026); where a page is the extraction source for ✅ claims in this guide it is marked accordingly.

**CIMB corporate — history, profile, strategy**
- CIMB Group — Our History (decade pages): [1990s](https://www.cimb.com/en/who-we-are/history/1990s.html), [2000s](https://www.cimb.com/en/who-we-are/history/2000s.html), [2020s](https://www.cimb.com/en/who-we-are/history/2020s.html) (✅ §1.3 lineage; 1970s-era page returned 404 in the pass)
- CIMB Group — [About Us](https://www.cimb.com/en/who-we-are/about-us.html) (✅ scale figures as at 30 June 2026)
- CIMB Group — [Forward30](https://www.cimb.com/en/who-we-are/forward-30.html) (✅ 4Cs; "modernise core banking technology, and data & AI")
- CIMB Group — [Newsroom](https://www.cimb.com/en/newsroom.html) and [Shareholding Information](https://www.cimb.com/en/investor-relations/shareholding-information.html) (✅ Khazanah/EPF/KWAP, 30 Jun 2026)
- CIMB Islamic — [Our History (2000s)](https://www.cimbislamic.com/en/who-we-are/history/2000s.html) (✅ Commerce-Tijari 2005)
- Wikipedia — [CIMB](https://en.wikipedia.org/wiki/CIMB), [CIMB Niaga](https://en.wikipedia.org/wiki/CIMB_Niaga), [Bumiputra-Commerce Holdings](https://en.wikipedia.org/wiki/Bumiputra-Commerce_Holdings)

**Investor relations / results**
- CIMB — [Annual Reports listing](https://www.cimb.com/en/investor-relations/annual-reports-listing.html); [Integrated Annual Report 2025 (PDF)](https://www.cimb.com/content/dam/cimb/group/documents/investor-relations/annual-reports/2025/cimb-iar2025.pdf)
- CIMB FY2025 release: [newsroom HTML](https://www.cimb.com/en/newsroom/2026/cimb-delivers-net-profit-of-rm7-9-billion-with-roe-at-11-3-percent-strengthens-long-term-value-creation-for-shareholders.html) and [4Q25 press release (PDF)](https://www.cimbprivatebanking.com/content/dam/cimb/group/documents/investor-relations/financial-information/cimb-group/2025/dec-2025/4q25-cimb-group-press-release.pdf) (✅ net profit RM7.9B, ROE 11.3%, CET1 14.3%, RM1.7B tech spend, RM100M+ AI training, 576 branches)
- CIMB 2Q26 release: [RM1.94B net profit, 11.2% ROE](https://www.cimb.com/en/newsroom/2026/cimb-records-resilient-2q26-with-rm1-94-billion-net-profit-and-11-2-percent-roe.html) (✅ OCTO Biz regional roll-out; corridor strategy)
- CIMB 3Q25 release (PDF): [PBT RM8.12B, 11.3% ROE for 9M25](https://www.cimb.com/content/dam/cimb/group/documents/investor-relations/financial-information/cimb-group/2025/sep-2025/3q25-cimb-group-press-release.pdf)

**CIMB Singapore**
- [CIMB Singapore homepage](https://www.cimb.com.sg/) (✅ retail live: cards/miles promos, FastSaver, Preferred Banking, WhatsApp)
- [OCTO by CIMB Singapore](https://www.cimb.com.sg/en/personal/digital-services/cimb-clicks/cimb-clicks-mobile-banking.html) and [cimbclicks.com.sg portal](https://www.cimbclicks.com.sg/)
- [CIMB Malaysia — CIMB OCTO App](https://www.cimb.com.my/en/personal/digital-services/cimb-octo-app.html)
- [CIMB Malaysia — MY→SG instant transfer](https://www.cimb.com.my/en/personal/day-to-day-banking/remittance/my-to-sg.html) (✅ zero fees to RM50,000)
- [CIMB Niaga — New OCTO Clicks](https://branchlessbanking.cimbniaga.co.id/en/new-octoclicks/)
- [Touch 'n Go newsroom — TnG Digital](https://www.touchngo.com.my/news/tng-digital-listing-in-the-works-sources-say-but-company-insists-its-in-no-hurry/)

**Newsroom releases (2026)**
- [CIMB Private Wealth aims to be the primary wealth partner for affluent individuals in ASEAN](https://www.cimb.com/en/newsroom/2026/cimb-private-wealth-aims-to-be-the-primary-wealth-partner-for-affluent-individuals-in-asean.html)
- [CIMB and China Galaxy Securities commence stockbroking JV (2018)](https://www.cimb.com/en/newsroom/2018/cimb-and-china-galaxy-securities-commence-stockbroking.html)

**Regulators and enforcement**
- [MAS Financial Institutions Directory — CIMB Bank Berhad (entry 3336)](https://eservices.mas.gov.sg/fid/institution/detail/3336-CIMB-BANK-BERHAD) (✅ Full-Bank branch licence; exempt CMS/FA/trust)
- [BNM enforcement notice ea-cimb24-bm](https://www.bnm.gov.my/-/ea-cimb24-bm) (✅ July 2024 AMP, CIMB Bank + CIMB Islamic, FSA/IFSA 2013)

**Press and secondary**
- The Edge Malaysia — [BNM fines Maybank, CIMB over RM5m for prolonged disruptions](https://theedgemalaysia.com/node/722778) (✅ RM760,000 CIMB AMP)
- The Edge Malaysia — [China Galaxy raises stake in stockbroking JV with CIMB](https://theedgemalaysia.com/node/598660)
- The Edge Malaysia — [Bumiputra-Commerce seeks to be renamed CIMB Group Holdings](https://theedgemalaysia.com/article/bumiputra-commerce-seeks-to-be-renamed-cimb-group-holdings)
- asiabiztoday — [CGS Holdings completes acquisition of CGS-CIMB Securities](https://www.asiabiztoday.com/2024/01/03/cgs-holdings-completes-acquisition-of-cgs-cimb-securities/) (✅ Jan 2024)
- briefglance/Virtusa pulse — [CIMB Niaga migrates core banking to Pega Cloud](https://briefglance.com/companies/virtusa-corporation/pulses/24037) (✅/⚠ Feb 2026, secondary)

**Companion guides in this repository (cross-referenced throughout)**: [Maybank Software Systems Guide](maybank_software_systems_guide.md), [Citibank Software Systems Guide](citibank_software_systems_guide.md), [DBS](dbs_software_systems_guide.md) / [OCBC](ocbc_software_systems_guide.md) guides, [Core Banking Systems Guide](core_banking_systems_guide.md), [Payment Rails Guide](payment_rails_guide.md), [Trade Finance Guide](trade_finance_guide.md), [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Banks in Singapore Guide](banks_in_singapore_guide.md), [Nets Singapore Guide](nets_singapore_guide.md), [Private Banking Guide](private_banking_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md)

---

*From a 1974 merchant-banking joint venture to a ten-market ASEAN universal group, CIMB's story is one of consolidation first and modernisation now: the names are verified — Bian Chiang to Bank of Commerce, PBSM to CIMB, GK Goh to CGS, CIMB Clicks to OCTO — while the engines underneath remain, by the group's own disclosure choices, the least documented part of the estate. The Singapore branch, a 1947 lineage wearing a 2026 digital face, ties the region together with an instant on-us corridor and an app family that finally outpaces the legacy beneath it — and so the regional tiger completes* the tiger's swing.
