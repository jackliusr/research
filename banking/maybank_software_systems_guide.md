# Maybank: The Software Systems Landscape — A Comprehensive Guide to the Technology Maybank Runs

*A companion deep-dive in the per-bank software-systems series of the [jackliusr/research](https://github.com/jackliusr/research) repository — the Maybank entry alongside [DBS](dbs_software_systems_guide.md), [OCBC](ocbc_software_systems_guide.md) and [Citibank](citibank_software_systems_guide.md). This guide focuses on the **specific software and technology systems** behind Maybank (Malayan Banking Berhad): the core-banking estate, the payments/transaction-banking stack, cards and wealth platforms, the digital channels (Maybank2u, MAE), data & AI, Islamic-finance systems, the risk & regulatory context — and the Singapore franchise, which is the group's second home market. It documents what is publicly verifiable, what is inferred from industry practice, and what Maybank does not disclose.*

**Verification convention used throughout: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural inference); ❌ = disputed (the record contradicts the claim); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-claims-status-and-verification-notes), and the non-public specifics are collected in [§12](#12-what-could-not-be-verified).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Software-Systems Focus — the technology estate of Maybank (Malayan Banking Berhad, Bursa Malaysia: 1155): core banking, payments/TTS, cards, wealth, digital channels, Islamic finance, data & AI, risk context, the Singapore franchise
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides:** [Citibank Software Systems Guide](citibank_software_systems_guide.md) (structural template), [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md) (the SG-bank genre), [Core Banking Systems Guide](core_banking_systems_guide.md), [Payment Rails Guide](payment_rails_guide.md), [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Banks in Singapore Guide](banks_in_singapore_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md), [Private Banking Guide](private_banking_guide.md), [Nets Singapore Guide](nets_singapore_guide.md), [DBS Bank Guide](dbs_bank_guide.md)

---

### Table of Contents

1. [Bank Profile: The Tiger from Kuala Lumpur](#1-bank-profile-the-tiger-from-kuala-lumpur)
2. [Core Banking and the Vendor Question](#2-core-banking-and-the-vendor-question)
3. [Payments and Transaction Banking](#3-payments-and-transaction-banking)
4. [Cards and Wealth](#4-cards-and-wealth)
5. [Digital Channels: Maybank2u and MAE](#5-digital-channels-maybank2u-and-mae)
6. [Data and AI](#6-data-and-ai)
7. [Islamic Finance Systems: Maybank Islamic](#7-islamic-finance-systems-maybank-islamic)
8. [Risk and Regulatory Context](#8-risk-and-regulatory-context)
9. [Singapore Angle: The Second Home Market](#9-singapore-angle-the-second-home-market)
10. [Worked Example: Cymbal Bank × Maybank — A Regional Correspondent Relationship](#10-worked-example-cymbal-bank--maybank--a-regional-correspondent-relationship)
11. [Claims Status and Verification Notes](#11-claims-status-and-verification-notes)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [References and Further Reading](#14-references-and-further-reading)

---

## 1. Bank Profile: The Tiger from Kuala Lumpur

### 1.1 Scope and Verification Convention

This guide is the **software-systems deep-dive for Maybank** — the Malaysian mirror of the [DBS](dbs_software_systems_guide.md) and [OCBC](ocbc_software_systems_guide.md) systems guides, and the sibling of the [Citibank guide](citibank_software_systems_guide.md) that is this series' structural template. Because the repository has no separate Maybank *bank* guide, this entry carries both the verified bank profile (history, group structure, footprint — §1) and the systems landscape (§2–§7), followed by the risk & regulatory context (§8), the Singapore angle (§9), a worked Cymbal Bank example (§10), and the honest claims audit (§11–§12).

The verification discipline is the same one this series applies to every bank: **✅ verified** means the claim was confirmed in this research pass against a primary source (Maybank's own history and milestones pages and press releases, Bursa Malaysia filings, MAS notices and register, Bank Negara Malaysia releases) or a strong secondary source (The Edge Malaysia, The Star, Straits Times, Reuters, Bloomberg). **⚠ flagged** means the claim is inferred, approximate, single-source, or structurally reconstructed. **❌ disputed** marks claims where the public record contradicts the common telling (there is at least one such claim in this guide — see §11.1). The [What Could Not Be Verified section](#12-what-could-not-be-verified) collects every materially non-public item.

The bank's own mascot — the tiger — runs through Maybank's branding (the logo's tiger, and the Maybank Tigers football club), which gives this guide its closing metaphor: the systems map of the tiger, ending at the tiger's ledger.

### 1.2 What Is Public: The Maybank Disclosure Reality

Maybank sits between the two disclosure extremes this series has documented. It is **more transparent than DBS about its vendor estate** — Maybank's core-banking vendor relationship (Silverlake Axis) is documented in vendor announcements and press, and Maybank names many of its platform partners publicly. But Maybank is **not Standard Chartered or OCBC**: there is no Maybank equivalent of OCBC's clean public statement that its Malaysian retail core is Silverlake SIBS, and no Maybank architecture narrative equivalent to SC's "Atlas on AWS". The consequence, enforced rigorously throughout: the **product layer** of the Maybank stack (Maybank2u, MAE, the group's app family) is mostly ✅-verifiable; the **engine layer** (which entity runs which core, the payments hubs, the data platform, the AML platform) is mostly ⚠-inferred, with specific vendor datapoints (Silverlake, Avaloq, Google Cloud, Temenos-era heritage) verifiable where third parties published them.

### 1.3 The Verified History: 1960 → 2026

The founding dates of Maybank are among the best-documented facts in Malaysian banking, and they verify cleanly ✅:

- **31 May 1960** — **Maybank (as Malayan Banking Limited) is incorporated** ✅ (Maybank's own history and corporate-milestones pages: "Maybank is incorporated on 31 May"; Wikipedia and CompaniesHistory corroborate). The founders were **Singaporean tycoon Tan Sri Khoo Teck Puat** (a former OCBC general manager, later known as "Singapore's richest man") **and Oei Tjong Ie** ✅/⚠ (Wikipedia, secondary; the Khoo Teck Puat founding role is corroborated by The Star's 2004 obituary coverage, flagged single-source-strong).
- **12 September 1960** — Maybank **begins operations** in Kuala Lumpur, occupying a corner shophouse at Jalan HS Lee (High Street) ✅ (Maybank history page). This is the founding-era date Maybank still celebrates — the bank's "since 1960" positioning.
- **1960 (year one)** — Maybank opens its **first overseas branch in Brunei (28 November 1960)** ✅ (Maybank milestones; corroborated by the 1960 Pelita Brunei government gazette report via Wikipedia's citation) **and branches in Singapore, the first on South Bridge Road, in December** ✅/⚠ (Maybank milestones page snippet; the month of December is from Maybank's own milestones chronology). Singapore was then still part of the same monetary area as Malaya — the Singapore network predates separation.
- **1962** — listed on the Kuala Lumpur Stock Exchange; Hong Kong and London branches opened ✅ (Maybank milestones via Wikipedia/CompaniesHistory, secondary).
- **1973** — forms **Aseambankers Malaysia Berhad** as its investment-banking arm ✅ (Wikipedia timeline; secondary) — the corporate ancestor of today's Maybank Investment Bank (§1.5).
- **1984** — New York branch opened ✅ (Wikipedia timeline; secondary).
- **2000** — **Maybank2u.com launches — Malaysia's first internet banking platform** ✅/⚠ (Maybank's own milestones page per search snippet; CompaniesHistory corroborates; widely reported) — the ancestor of today's Maybank2u channel (§5).
- **2008** — the group's transformative expansion year: acquisition of stakes in **Bank Internasional Indonesia (BII)** (Indonesia), **An Binh Bank** (Vietnam) and **MCB Bank** (Pakistan), and the establishment of **Maybank Islamic Berhad** ✅ (Wikipedia timeline; secondary — each deal was heavily reported at the time).
- **2011** — acquisition of **Kim Eng Holdings Limited**, the Singapore-founded regional brokerage, creating **Maybank Kim Eng** ✅ (Wikipedia timeline; heavily reported at the time) — the Singapore equities platform that §4 and §9 exercise.
- **2012** — first branch in Laos completes the group's footprint in **all ten ASEAN nations** ✅ (Wikipedia timeline; secondary).
- **1 May 2022** — **Dato' Khairussaleh Ramli** appointed Group President & CEO ✅/⚠ (Maybank announcement 27 Jan 2022 per Wikipedia citation; the effective date of 1 May 2022 is Wikipedia's — the appointment announcement itself is primary ✅, the effective-date wording ⚠ single-source). He is the group CEO as of this guide's date.

**The Kwong Yik Bank lineage — a correction to the common telling** ⚠/❌ — the belief that Maybank's ancestry runs through "Kwong Yik Bank, the oldest local bank" needs careful unpicking, because there are *two* institutions called Kwong Yik and Maybank's connection to the Malaysian one is *partial and temporary*:

- The **1903 Kwong Yik Banking Corporation** of Singapore (founded 16 December 1903 by Cantonese towkays led by Wong Ah Fook, the first local bank in Singapore and British Malaya) **failed in 1913** after a bank run and was voluntarily liquidated ✅ (Wikipedia, with scholarly sources) — it has no Maybank lineage at all, and its failure actually triggered tighter colonial bank regulation.
- The **Kwong Yik Bank Berhad of Kuala Lumpur, established 1913**, was Malaya's oldest continuously operating local bank into the modern era ✅ (RHB Bank's own history via Wikipedia: Kwong Yik Bank Berhad was "established in 1913 in Kuala Lumpur, making it Malaya's first..." — secondary).
- Maybank's relationship with that Malaysian Kwong Yik Bank was as an **owner-seller, not a founder**: Maybank came to control Kwong Yik Bank in the consolidation era of the 1980s ⚠ (acquisition year not pinned in this pass — flagged), and in **December 1996 sold a 75% stake to Abdul Rashid Hussain for RM2.16 billion**, after which Kwong Yik Bank was **merged with DCB Bank to form RHB Bank in 1997** ✅/⚠ (The Capital Journal, secondary single-source for the price and stake; Wikipedia's Maybank timeline records "Kwong Yik Bank sold to Rashid Hussain Berhad in December"; the 1997 RHB Bank merger is in RHB Bank's own history ✅/⚠ secondary).
- **Conclusion for the lineage question:** Maybank's own 1960 founding is its true origin; the "Kwong Yik oldest-bank lineage" belongs to **RHB** (via the 1997 merger), not Maybank. Anyone attributing a Kwong Yik ancestry to Maybank is reading the ownership interlude (1980s–1996) as lineage. This guide records the interlude as a ⚠ item in §11 and moves on — the tiger's own founding in 1960 is the load-bearing date.

### 1.4 Group Structure: The Universal-Bank Shape

Maybank is a **universal banking group** in the [Universal Banking Model Guide](universal_banking_model_guide.md) sense — commercial banking, Islamic finance, investment banking, insurance and asset management under one listed holding company — and its public structure verifies cleanly ✅/⚠ (the entity names are public; the legal-entity *diagram* is in Maybank's annual reports, not re-verified here):

| Group business | Public entity (verified) | Notes |
|---|---|---|
| **Commercial banking (home market)** | Malayan Banking Berhad (Maybank) — the licensed bank | Bursa Malaysia: 1155; the listed holding company and its principal banking subsidiary in Malaysia |
| **Islamic finance** | **Maybank Islamic Berhad** | Established 2008; claims to be the largest Islamic bank in Malaysia and ASEAN by assets ✅/⚠ (Wikipedia/Maybank claims — vendor-of-record figures; §7) |
| **Investment banking** | **Maybank Investment Bank Berhad** | Renamed from Aseambankers in 2009 ✅ (Wikipedia; secondary); regional equities under the Maybank Kim Eng → Maybank Securities brand family (§9) |
| **Insurance & takaful** | **Etiqa** (Etiqa Insurance & Takaful under the group's insurance holding) | Etiqa brand launched 2007 consolidating Mayban Fortis-era businesses; the group's bancassurance JV with Ageas was consolidated under Maybank Ageas Holdings in the 2010s era ✅/⚠ (§8/§1 cross-ref; exact JV percentages flagged) |
| **International banking** | PT Bank Maybank Indonesia (ex-BII), Maybank Philippines, Maybank Cambodia, Maybank Singapore Limited, MCB Bank (Pakistan), Maybank Vietnam (ex-An Binh) | Country subsidiaries/branches under Maybank International Holdings / Maybank group ✅ (Wikipedia subsidiary list; secondary) |

**Scale markers (dated, with the year of each figure):** the group's total assets crossed into the **RM1-trillion era in the 2020s** ✅/⚠ — reported total assets of ~RM888 billion at FY2021 (Wikipedia, citing Maybank's 2021 Annual Report) and above RM1.0 trillion by the FY2024 reporting era ⚠ (secondary aggregator figures — see §11 for the exact dated figure this pass could verify); market capitalisation of roughly RM120 billion / US$30 billion in the 2025 era ⚠ (aggregator, flagged); **43,000+ employees** ✅/⚠ (Maybank's own boilerplate "over 43,000"; Wikipedia); a worldwide network of **2,600+ branches across 18 countries** ✅/⚠ (Wikipedia/aggregator, flagged as approximate and dated — Maybank's own "18 countries" framing is the verified anchor, §1.6).

### 1.5 The International Footprint

- **~18 countries** ✅/⚠ — Maybank's public language is that the group operates in 18 countries ⚠ (aggregator/secondary in this pass — Maybank's own "about us" pages state the count with slightly varying wording by year; the 18-country figure with presence in all ten ASEAN nations is consistent across sources). Key markets beyond Malaysia: **Singapore** (the second home market, §9), **Indonesia** (Maybank Indonesia), the Philippines, Vietnam, Cambodia, Laos, Brunei, Myanmar ⚠ (network composition by country flagged), plus China (Shanghai branch since 2000, Beijing representative office since 1993), Hong Kong, London, New York (branch since 1984), Bahrain (since 2002), Pakistan (MCB), Saudi Arabia ⚠, UAE ⚠, and Uzbekistan (Uzbek Leasing) ⚠ — several of these are flagged as reported-but-not-verified-in-this-pass.
- **Home-market leadership** ✅ — Maybank is **Malaysia's largest bank by market capitalisation and total assets** (Wikipedia; consistent with The Banker rankings and Brand Finance reports through the 2020s).
- The retail branch network of **~2,600 branches worldwide** ⚠ includes roughly 400 domestic Malaysian branches ⚠ (figure dated and flagged — Maybank's branch counts have been in gradual decline with digital migration; exact current counts are in annual-report footnotes, not re-verified here).

### 1.6 Key Milestones Timeline

| Year | Milestone | Status |
|---|---|---|
| 1960 | Incorporated 31 May as Malayan Banking Limited; operations begin 12 September in KL; first overseas branch Brunei (Nov); Singapore branches opened (first, South Bridge Road, Dec) | ✅ (incorporation/operations); ✅/⚠ (SG branch month) |
| 1962 | Listed on KLSE; Hong Kong and London branches | ✅/⚠ (secondary) |
| 1973 | Aseambankers Malaysia Berhad formed (investment banking) | ✅/⚠ (secondary) |
| 1984 | New York branch opened | ✅/⚠ (secondary) |
| 1996 | Kwong Yik Bank (75%) sold to Rashid Hussain (Dec) | ✅/⚠ (secondary) |
| 2000 | Maybank2u.com launched — Malaysia's first internet banking | ✅/⚠ |
| 2006 | Amex card business in Malaysia acquired | ✅/⚠ (secondary) |
| 2007 | Etiqa brand launched for insurance/takaful businesses | ✅/⚠ (secondary) |
| 2008 | BII (Indonesia), An Binh (Vietnam), MCB (Pakistan); Maybank Islamic Berhad established | ✅/⚠ (secondary) |
| 2009 | Aseambankers renamed Maybank Investment Bank; RM6 billion rights issue | ✅/⚠ (secondary) |
| 2011 | Kim Eng Holdings acquired — Maybank Kim Eng created | ✅/⚠ (secondary) |
| 2012 | Laos branch completes ASEAN-10 footprint | ✅/⚠ (secondary) |
| 2022 | Khairussaleh Ramli appointed Group President & CEO | ✅/⚠ |
| 2020s | Group total assets cross RM1.0 trillion era | ⚠ (see §11) |

The timeline reads in three eras. **Founding era (1960–1973):** the incorporation, the first branches across Malaya/Singapore/Brunei, the listing, and the investment-bank seed. **Expansion era (1984–2012):** the New York branch, the finance-company and insurance builds, Maybank2u, the 2008 regional acquisitions, and the ASEAN-10 completion. **Regional-digital era (2011–2026):** Kim Eng, Maybank Islamic scale-up, the RM1-trillion balance sheet, the app-first retail channels, and the AI/data agenda (§5–§6). The systems-relevant rows — 2000 (Maybank2u), 2008 (Islamic), 2011 (Kim Eng) — are the anchors §2–§7 hang their verified claims on.

### 1.7 The Strategy Layer: M25+ and the Digital Agenda

Every systems claim in this guide hangs off the group's public strategy frame, **M25+** — Maybank's medium-term plan to 2025, launched in effect from **Q4 2022**, with FY2025 as its "final leg" (per the CEO's FY2024 results statement) ✅ (The Asian Banker release). The M25+ verified facts that matter for the systems map:

- **"Supergrowth" segments** — wealth management, bancassurance and motor insurance, SME and cash management grew income **21–23% on average** since Q4 2022 ✅; these are the businesses whose platforms §4 exercises.
- **Digital-enablement outcomes** — transaction-banking proposition +16 percentage points; digitally opened/activated Malaysian CASA accounts **doubled**, contributing 10.7% of overall deposit growth ✅ (§5).
- **Analytics outcomes** — 31% uplift in insurance/takaful policy sales and 57% uplift in daily auto-renewal premiums attributed to "enhanced analytics and algorithm" ✅ (§6).
- **Regional growth** — Singapore investment income +66% YoY; SME loan origination +36% (SG) and +27% (Indonesia); Singapore deposits +18.7% ✅ (§9).
- **Successor framing** — the CEO's FY2024 statement points beyond M25+ to deeper ASEAN integration, "hyper-personalisation," and "technological innovation for Islamic financial services" ✅ (§7) — the strategic lane the next systems era will occupy.

The M25+ disclosures are the closest Maybank comes to publishing an architecture narrative: they verify *what the estate achieves* while remaining silent on *what the estate is* — which is precisely the disclosure posture §1.2 and §12 document.

---

## 2. Core Banking and the Vendor Question

This section answers the question every guide in this series must answer for its bank: **what ledger engine actually runs the deposits, loans and accounts?** For Maybank the honest answer has two layers: the *product layer* is public (Maybank2u, MAE, the brands customers touch), and the *engine layer* is the domain of one dominant regional vendor — Silverlake Axis — whose specific contracts with Maybank are documented only partially and indirectly. What follows separates the verified from the structural.

### 2.1 The Vendor Question: Silverlake Axis

- **The vendor's public facts verify cleanly ✅.** Silverlake Axis Ltd (SGX-listed) sells the **Silverlake Integrated Banking Solution (SIBS)** and its flagship successor **SIBS II**. Silverlake's own product page describes SIBS II as "our flagship core system, ... built on 30 years of experience, designed to modernise financial institutions with customer-centric and digital-first business models," with "a modular, service-oriented architecture that integrates seamlessly with existing systems" (silverlakeaxis.com/services/Core-Banking — extracted in this pass ✅). SGX vendor announcements confirm the *shape* of the business: SIBS contract wins routinely bundle the vendor's digital subsidiaries (Cyber Village, Silverlake Digitale, Silverlake One Paradigm) alongside the core — i.e., Silverlake sells core-plus-digital as one estate (SGX filing via links.sgx.com ✅ for the vendor's deal structure; the specific customer in that filing is an unnamed financial institution, not Maybank).
- **The Maybank connection is real but indirect ⚠.** The partial guide's §1.2 framing — that the Silverlake–Maybank relationship "is documented in vendor announcements and press" — holds as a *general* statement: Maybank group entities have appeared in Silverlake's public contract announcements over the years, and industry reporting has long associated Maybank's Malaysian domestic retail/commercial estate with the SIBS family ⚠ (no specific Maybank–Silverlake contract announcement was re-verified in this research pass — every such claim below this line is flagged). The Apps Run The World SIBS customer database lists Maybank among SIBS customers ⚠ (paywalled aggregator database; membership is inferred from vendor/industry sources, not from Maybank).
- **The honest structural position:** for a Malaysian universal bank of Maybank's vintage (domestic retail core built out through the 1990s–2000s, Maybank2u launched 2000), the *era-typical* domestic platform is a SIBS-family packaged core — Malaysia's banking-software industry consolidated around Silverlake precisely because the domestic banks standardised on it. That is structural/industry knowledge (unmarked here as such), **not** a verified Maybank fact. What *is* verified: Maybank is not OCBC — it has never published a clean "our retail core is SIBS" statement, and the search trail for one returns vendor pages, not Maybank statements (§12 item 1).
- **The 2006-era systems context** ⚠ — Maybank's domestic core estate of the 2000s would have been a classic packaged-core deployment: a central ledger for deposits (savings/current/fixed), a loans origination-and-servicing layer, and channel integration buses feeding Maybank2u, ATMs and branches. None of the internal application names are public; the architecture class is certain for a bank of this size.

### 2.2 The Maybank2u-Era Architecture (2000 → 2020s)

- **2000: Maybank2u.com launches as Malaysia's first internet banking platform** ✅ (§1.3) — the channel event that forced the domestic estate into always-on, integrated-channel mode. The Maybank2u-era stack, reconstructed structurally ⚠: the packaged core of record (SIBS-class, §2.1) at the centre; a channel-integration/middleware layer (online banking, later mobile, ATMs, branches, phone); payments adapters into the national rails (MEPS/PayNet, §3); and per-product engines (cards, loans, trade) around the core. Maybank's own milestones confirm the *product* milestones (Maybank2u launch 2000, mobile banking later in the 2000s-2010s ✅/⚠ — product names public, dates partially flagged); the *middleware identities* are not public (§12 item 2).
- **Regional subsidiaries ran their own estates** ⚠ — each country bank (Indonesia, Vietnam, Pakistan, Philippines, Cambodia, Singapore) would have its own core of record, whether a local packaged core, a home-grown system, or a shared-group platform; the only honest statement is that the identities are not public (§12 item 3). What the FY2024 results *do* verify is the group-wide **digital-enablement acceleration**: "digitally opened and activated CASA accounts in Malaysia by two-folds, contributing 10.7% in the overall deposit growth," and transaction-banking proposition improvement of 16 percentage points (The Asian Banker's re-dissemination of Maybank's FY2024 results ✅) — group-level outcomes, not engine details.
- **The core-banking family cross-ref, condensed from the [Core Banking Systems Guide](core_banking_systems_guide.md):** the guide's vendor classes are (1) legacy in-house/mainframe estates, (2) packaged regional cores (the SIBS class), (3) the Temenos T24/Transact class, and (4) modern cloud-native cores (Thought Machine, Mambu, and the "coreless" digital-bank pattern). Maybank's domestic estate sits structurally in class (2) ⚠; its newer digital propositions (MAE-era, §5) sit architecturally in a class-(2)-plus-digital-front configuration ⚠, with class-(4) exploration unverified (§12 item 4). The Temenos-era heritage named in §1.2 refers to the regional pattern in which T24-class systems appear across Malaysian banking histories ⚠ — no Maybank–Temenos contract was verified this pass.

### 2.3 Public vs Not-Public: The Maybank Core-Banking Ledger

| Layer | Publicly verifiable | Status |
|---|---|---|
| Core vendor *product* (SIBS / SIBS II) | Silverlake product pages, SGX announcements | ✅ |
| Core vendor *relationship* with Maybank | Industry reporting, vendor-customer databases | ⚠ (indirect; not re-verified this pass) |
| Maybank's own core-platform statements | None found (no OCBC-style disclosure) | ⚠ (absence of evidence) |
| Which entity runs which core (Malaysia, Singapore, Indonesia, etc.) | Not public | ⚠ / not public |
| Channel layer (Maybank2u, MAE) | Product brands, launches, user figures | ✅ (product); ⚠ (hosting/architecture) |
| Core-modernisation roadmap | Not public (M25+ outcomes are public; the platform plan is not) | ⚠ / not public |

The working rule for the rest of this guide: **when Maybank's own disclosures name a platform partner, the claim is ✅-grade; when the identity of an engine is inferred from the vendor class, it is ⚠-grade and lands in §12.**

### 2.4 A Domestic Retail Transaction Walk-Through

To make the abstract vendor question concrete, here is the *classic* Maybank domestic retail transaction as the systems estate must serve it (structural reconstruction ⚠ — every step is industry-standard for a Malaysian universal bank; none of the internal system names are verified):

1. **Onboarding** — a new customer opens a savings account in-branch or digitally (digitally opened CASA doubled in the M25+ era, §1.7 ✅). KYC data is captured (the AML/KYC estate, §8.1), the customer is risk-rated, and the account is created on the domestic core of record (§2.1) with a bank/state/branch/serial account number in the standard Malaysian format.
2. **The daily cycle** — deposits, withdrawals, transfers and card transactions post to the core; the core computes profit/interest, maintains balances, and feeds the general ledger. End-of-day batch processing produces the books; intraday, the core serves balances to the channels (Maybank2u/MAE/ATMs/branches).
3. **A payment leaves the bank** — a customer's DuitNow transfer to another bank exits via the payments layer (§3.1): the channel hands off to the payment hub, which formats the instruction for PayNet's rail, debits the customer's account on the core, and settles net through the central-bank settlement account.
4. **A loan is serviced** — instalment deductions run against the core's loans module; arrears and impairment feeds the risk/collections estate (FY2024 GIL 1.23% and loan-loss coverage 126.9% are the observable outputs of that estate ✅, §8.3).
5. **Reconciliation and reporting** — the core's trial balances reconcile to the GL; regulatory returns (BNM) and group reporting (the data platform, §6) consume the same ledgers.

Every step is ledger-true on the core estate; nothing in steps 1–5 requires the reader to know a vendor name, which is exactly the point of §2.3's public/not-public table.

### 2.5 Cross-Reference Digest: What Each Sibling Guide Contributes

- **[Core Banking Systems Guide](core_banking_systems_guide.md)** — the vendor classes (legacy in-house, SIBS-class packaged, Temenos-class, cloud-native) and the Islamic-module question; Maybank's domestic estate sits in class (2) structurally ⚠.
- **[Payment Rails Guide](payment_rails_guide.md)** — rail mechanics for §3: MEPS/PayNet clearing, FAST, SWIFT/ISO 20022, QR interoperability.
- **[Universal Banking Model Guide](universal_banking_model_guide.md)** — the holding-company shape (§1.4) and how a listed parent houses the licensed bank, the Islamic subsidiary, the broker and the insurer.
- **[Private Banking Guide](private_banking_guide.md)** — the wealth-platform layers §4.2 maps Maybank onto.
- **[Enterprise Risk Management Guide](enterprise_risk_management_guide.md)** — the risk-data and capital-estate context for §8.3's metrics (CET1, LCR, GIL).
- **[Citibank guide](citibank_software_systems_guide.md)** — the structural template and the worked-example canon §10 extends.

### 2.6 Where Maybank Sits on the Disclosure Spectrum

This repository's per-bank guides now span the full disclosure spectrum, which makes Maybank's position legible by comparison:

| Bank guide | Core-platform disclosure posture | Resulting guide posture |
|---|---|---|
| [DBS](dbs_software_systems_guide.md) | Most secretive — strategy public, stack is the moat | Product-layer guide with heavy ⚠ inference |
| **Maybank (this guide)** | **Partially open — vendor ecosystem public (Silverlake), specific contracts not** | **Product/results ✅; engines ⚠; balanced guide** |
| [OCBC](ocbc_software_systems_guide.md) | Clean public statement that the Malaysian-era retail core is Silverlake SIBS | Engine claims verifiable |
| [Standard Chartered](standard_chartered_guide.md) | Publishes architecture narratives ("Atlas on AWS") | Deep engine-layer verification |

The spectrum table explains this guide's tone: Maybank is the *middle case* — open enough that its vendor ecosystem is known (Silverlake's regional dominance is partly *because* of Malaysian banks like Maybank), closed enough that no contract-level claim can be ✅. Guides that read Maybank as either extreme (fully opaque like DBS, or fully disclosed like OCBC) would mislead; this one reads it as the middle case and flags accordingly.

---

## 3. Payments and Transaction Banking

Maybank's payments story is a national-rails story (Malaysia), a regional-rails story (ASEAN subsidiaries), and a Singapore-rails story (§9). The [Payment Rails Guide](payment_rails_guide.md) supplies the rail mechanics; this section maps Maybank onto them. Verified anchor first: **Maybank reports more than RM4 trillion in digital transactions over the five years since 2020, in its own words "reaffirm[ing] its commitment to support the nation's digital banking adoption"** ✅ (maybank.com newsroom, 29 Oct 2025 — "Maybank facilitates over RM4 trillion in digital transactions since 2020"; the URL slug's "double Malaysia GDP" framing and secondary coverage ⚠ put the RM4-trillion figure at roughly twice Malaysia's annual GDP). The same release ranks Maybank as **"the top bank used for services such as DuitNow transfers and FPX"** ✅/⚠ (Maybank's own claim, echoed by secondary coverage).

### 3.1 The Malaysian National Rails: MEPS and PayNet

- **MEPS (Malaysian Electronic Payment System)** — the domestic interbank network for shared ATMs and interbank funds transfers, the oldest layer of Malaysia's retail-payments fabric. Maybank is a founding-class MEPS participant (structural ✅/⚠; participant lists are on the operator's pages, not re-verified here). MEPS-era instruments (interbank GIRO, IBFT instant transfers) were the backbone Maybank2u plugged into from 2000 (§2.2).
- **PayNet (Payments Network Malaysia Sdn Bhd)** — the national payments *operator* that consolidated Malaysia's retail-payments infrastructure (the MEPS-era clearing houses and the PayNet entity were merged/consolidated in the late-2010s era ⚠ structural; the entity name and its product family are public ✅). PayNet operates the rails that define Malaysian digital payments today:
  - **DuitNow** — the real-time, proxy-ID retail transfer rail (mobile number / NRIC / passport-based addressing). Maybank's claim to be the **top bank for DuitNow transfers** ✅/⚠ (Maybank release) makes it the largest *participant* on the rail, which implies deep integration into the MAE/Maybank2u apps and the core of record (§2).
  - **FPX (Financial Process Exchange)** — the online-payments rail (direct-debit style e-payments from bank accounts to merchants/billers/government). Same Maybank top-bank claim applies ✅/⚠.
  - **DuitNow QR** — the interoperable QR rail (Malaysia's answer to SGQR), which cross-border-links with regional QR schemes (the ASEAN QR interoperability layer: Thai QR PromptPay linkage first, then the regional network) ⚠ structural — Maybank as a top QR acquirer/issuer is consistent with its DuitNow positioning ⚠.
  - **JomPAY** (bill presentment/payment) and **MyDebit** (domestic debit scheme) — the other PayNet products Maybank issues/accepts structurally ⚠.
- **The Maybank2u/MAE integration pattern** (structural ⚠): retail payment initiation lives in the apps; the payment hub (whoever runs it — §12 item 5) orchestrates rail selection (DuitNow for proxy-ID P2P, FPX for merchant e-payments, MEPS IBFT for account-number transfers, FAST in Singapore); settlement is on Bank Negara Malaysia's RPP (Real-time Retail Payments Platform) settlement systems ⚠ structural. Maybank2u's own history — first internet banking in Malaysia (2000) — makes it the country's longest-running digital payments front-end ✅.

### 3.2 Transaction Banking: Cash Management and Trade

- **Maybank2E / Maybank2u Biz** ⚠/✅ — the corporate/business channel family (product names public; specific versioning ⚠). Maybank's FY2024 M25+ reporting verifies the strategy: **"Acceleration of digital enablement increased transaction banking proposition by 16 percentage points"** ✅ (Asian Banker release), and SME funding access expanded (average monthly loan origination for SMEs **up 36% in Singapore** and 27% in Indonesia in the M25+ period ✅ — Asian Banker release), which quantifies the cash-management/SME digital push even where the underlying platform names are not public.
- **SWIFT/correspondent layer** ✅/⚠ — a group with branches/subsidiaries across 18 countries and a Singapore franchise of Maybank's size is a SWIFT member and maintains a correspondent network for MYR, SGD, USD and regional-currency clearing (structural; Maybank's own treasury/global-markets disclosures are the anchor ✅/⚠; the correspondent-bank list is not public). ISO 20022 migration applies to Maybank as it does to all SWIFT users ⚠ structural (see the [Payment Rails Guide](payment_rails_guide.md) for the MX migration mechanics).
- **Regional rails by country** ⚠ structural — Maybank Indonesia participates in Indonesia's BI-FAST/QRIS rails; Maybank Philippines, Maybank Cambodia, Maybank Vietnam (NAPAS-class rails), and MCB Pakistan (Raast-class rails) each sit on their home country's payment infrastructure. These are ⚠ because Maybank's country-level rail memberships were not individually verified this pass — but a licensed bank in each market is by definition a participant in that market's rails.
- **The RM4-trillion anchor again** ✅ — the 2025 release frames the group's digital-transaction volume as the summary statistic of the entire payments estate: five years, RM4+ trillion, DuitNow/FPX leadership, MAE at the centre (§5). That is the tiger's payments story in one number.

### 3.3 The Rail Map, Tabulated

| Rail / network | Operator | Instrument class | Maybank's role | Status this pass |
|---|---|---|---|---|
| MEPS (shared ATM + interbank transfer layer) | Payments network infrastructure (PayNet-operated era) | ATM sharing; IBFT/GIRO interbank transfers | Participant since the pre-PayNet era | ⚠ structural (participant lists operator-side) |
| DuitNow | PayNet | Real-time proxy-ID P2P transfers | Top bank for DuitNow transfers (Maybank's claim) | ✅/⚠ (claim in Maybank release) |
| FPX | PayNet | Online e-payments (bank account → merchant/biller) | Top bank for FPX (Maybank's claim) | ✅/⚠ |
| DuitNow QR | PayNet | Interoperable QR payments | Issuer/acquirer via MAE/merchant estate | ⚠ structural |
| MyDebit | PayNet | Domestic debit scheme | Issuer of MyDebit cards | ⚠ structural |
| JomPAY | PayNet | Bill presentment & payment | Participating biller bank | ⚠ structural |
| SWIFT | SWIFT (cooperative) | Cross-border correspondent messages (MT/MX) | Member; MYR/SGD/USD correspondent clearing | ✅/⚠ structural (membership implied by scale; ISO 20022 migration applies) |
| FAST / PayNow | Singapore (operator-led) | Real-time SGD transfers; proxy-ID layer | Maybank Singapore participant | ⚠ structural (see §9.2) |
| BI-FAST / QRIS | Indonesia | Real-time IDR; QR | Via Maybank Indonesia | ⚠ structural (country-level) |
| RPP settlement | Bank Negara Malaysia | Retail payment settlement | Settling member via PayNet | ⚠ structural |

The rail map summarises the honest posture: the *national rail names* are public and verified ✅/⚠; Maybank's *exact membership and certification status* on each is operator-side data not re-verified this pass ⚠.

### 3.4 Trade Finance and Treasury Systems Context

- **Trade finance** ⚠ structural — a universal bank of Maybank's scale runs the trade-finance system family: documentary credits (issuance/advice/confirmation, MT700s), collections, guarantees and standby LCs (SBLCs), and supply-chain finance. The [Trade Finance Guides](trade_finance_guide.md) in this repository document the instrument mechanics; Maybank's trade platform identities are not public (§12 item 5 extension — flagged with the payments hub). What is verified is the *corridor logic*: the ASEAN-10 network (§1.5) exists to serve intra-ASEAN trade, and §10's worked example exercises the LC corridor.
- **Treasury and global markets** ✅/⚠ — the FY2024 results verify the business (NOII driven by "income from ... global markets" ✅) and the Singapore hub role (NOII +57.9% in Singapore, §9.1 ✅); the treasury-system estate (FX trading, money markets, liquidity, the nostro management of §10) is not public (§12 item 16).
- **Cash management for corporates** — Maybank2E-class portals plus host-to-host/API connectivity for corporates ⚠ structural; the M25+ verified outcome ("transaction banking proposition +16 percentage points," §1.7 ✅) is the measurable signature of the cash-management build-out.

### 3.5 FX, Remittance and Corridor Notes

- **MYR is a non-internationalised currency** (structural, unmarked) — ringgit accounts and payments outside Malaysia are restricted by BNM's exchange-control rules, which is *why* the §10 correspondent pattern (a MYR nostro inside Malaysia, held by an offshore bank) is the standard way non-Malaysian banks serve MYR needs. Maybank's own cross-border MYR business is shaped by the same rules; its Singapore franchise handles SGD/MYR conversion at scale for the corridor (FY2024: SGD net income MYR-linked flows ⚠ structural).
- **Remittance corridors** ✅/⚠ — Malaysia is a major remittance-receiving market (Indonesian and Bangladeshi workers, among others) and Maybank's network (Maybank Indonesia, MCB Pakistan) sits on both ends of classic corridors ⚠ structural; Maybank's own remittance products are public at product level but the underlying correspondent/aggregation arrangements are not (§12).
- **The ASEAN-10 settlement argument** — the group's presence in all ten ASEAN nations (§1.5 ✅) makes it one of the few banks able to offer in-network settlement across the region's local-currency rails without always crossing USD — the strategic value §10.5 summarises and the intra-ASEAN trade agenda of §3.4 exercises.

---

## 4. Cards and Wealth

### 4.1 Cards: The Plastic Layer

- **Timeline anchors** ✅/⚠ — Maybank's card history is public at the product level: the 2006 acquisition of the **Amex card business in Malaysia** (§1.6 row — secondary-source verified) made Maybank the local issuer of American Express products, layered on top of its long-standing **Visa and Mastercard** issuing. Maybank's own milestones page covers the card-era product launches ✅/⚠ (dates of specific Visa/MC product milestones flagged; the Amex row is the verified systems-relevant anchor).
- **The domestic debit layer** — Maybank issues **MyDebit** (PayNet's national debit scheme) and is a top DuitNow/FPX bank (§3.1) ✅/⚠; the debit/ATM switching historically rides MEPS ✅/⚠ structural.
- **The cards engine** ⚠ — the identities of Maybank's card management, authorisation/switch, and acquiring platforms are not public (§12 item 6). What is structurally certain for a Malaysian issuer of Maybank's scale: a card-management system of record (limits, billing, rewards), scheme connectivity to Visa/Mastercard/Amex networks, an acquiring estate for merchants, and 3-D Secure / tokenisation layers in the digital era (structural, unmarked).
- **FY2024 card/consumer context** ✅ — Maybank's results release shows the M25+ "Supergrowth" segments (wealth management, bancassurance and motor insurance, SME and cash management) growing income **21–23% on average** since M25+ began in Q4 2022 — the consumer/affluent engine behind the cards and wealth businesses is verifiably the group's growth priority (Asian Banker release).

### 4.2 Wealth: Private Wealth, Kim Eng, and the Advisory Stack

- **Maybank Private Wealth** ✅/⚠ — the group's high-net-worth brand in Malaysia and Singapore (brand public; entity structure ⚠). FY2024 verified: **non-interest income rose 22.6% group-wide, "led by ... higher fees from wealth management and investment banking"** ✅ (Asian Banker release), and Singapore investment income rose **66% YoY** in the M25+ period ✅ — the wealth businesses are quantitatively the group's fastest-growing income stream.
- **Maybank Kim Eng / Maybank Securities** ✅/⚠ — Kim Eng Holdings acquired 2011 (§1.3), the Singapore-founded regional brokerage rebranded through the Maybank Kim Eng → Maybank Securities family. In Singapore the equities arm operates as **Maybank Securities Pte. Ltd.** (SGX member) ✅/⚠ (structure secondary; §9). The brokerage technology (trading platform, order routing to SGX/Bursa, client-accounting) is not public (§12 item 7) — structural: a regional broker of Kim Eng's vintage runs a DMA/agency-equities stack with online dealing and research distribution ⚠.
- **The private-banking cross-ref, condensed from the [Private Banking Guide](private_banking_guide.md):** the repo's private-banking canon holds that an Asian private bank of Maybank's tier runs (a) a core banking platform for the booking entity, (b) a wealth-management/advisory front (portfolio, advisory, discretionary), (c) an execution layer to the brokerage, (d) onshore/offshore booking centres (Singapore being the regional hub), and (e) the AML/KYC estate (§8). Which *products* Maybank uses in layers (b)–(e) is not public (§12 items 7–8); the guide's generic architecture is the honest template.
- **Maybank's Singapore wealth centre** ✅/⚠ — Maybank Singapore's FY2024 results (PBT SGD 702.20 million, +16.6%; net income SGD 1.29 billion, +14.0%, with NOII +57.9% on wealth/global-markets fees — Asian Banker release ✅) evidence a scaled wealth franchise in the second home market (§9). The specific wealth-centre locations and the platform behind them are ⚠ (§12).
- **Etiqa in the wealth story** ✅ — the insurance/takaful arm (Etiqa) is part of the bancassurance wealth engine: FY2024 PBT **MYR 1.52 billion, +54.7%**, general-insurance/takaful market share **16.6% (No. 1 in Malaysia)**, life/family new business third at 14.0% (Asian Banker release ✅). The policy-administration and distribution systems behind Etiqa are not public (§12 item 9).

### 4.3 The Card and Wealth Portfolio at a Glance

| Product family | Public facts | System layer | Status |
|---|---|---|---|
| Visa / Mastercard credit cards | Long-standing issuing franchise; product names public | Card management, scheme connectivity, authorisation | ✅ (products); ⚠ (engines, §12 item 6) |
| Amex cards (Malaysia) | 2006 acquisition of the Amex business (§1.6) | Amex network connectivity, cobrand servicing | ✅/⚠ (deal); ⚠ (platform) |
| MyDebit / ATM cards | National debit scheme issuance (§3.1) | Debit switch, MEPS ATM sharing | ⚠ structural |
| Maybank Private Wealth | HNW brand in MY & SG; NOII +22.6% group-wide on wealth fees ✅ | Booking, advisory, portfolio platforms | ⚠ (§12 item 8) |
| Maybank Kim Eng / Maybank Securities | Regional brokerage since 2011; SG equities arm | Trading, order routing, client accounting | ⚠ (§12 item 7) |
| Etiqa / Etiqa Takaful | No. 1 Malaysian general insurer/takaful ✅ | Policy admin, claims, bancassurance | ⚠ (§12 item 9) |
| Bancassurance & unit-linked wealth | M25+ Supergrowth (+21–23% income ✅) | Distribution middleware, fund links | ⚠ structural |

The portfolio table compresses the section's discipline: the *products* are public and mostly ✅/⚠-verified; the *platforms* under every row are §12 items. The wealth-management segment structure (which businesses count as "wealth management" in the FY24 numbers) is Maybank's internal segmentation ⚠.

### 4.4 Advisory Evolution and Digital Wealth

- **The hybrid advisory pattern** (structural ⚠) — Asian private banks of Maybank's tier are migrating from relationship-manager-led advice to hybrid models: digital onboarding, e-account-opening for wealth clients, robo/guided-investment propositions for the mass-affluent, and advisor workbenches for the HNW tier. Maybank's M25+ verified outcomes (wealth income +21–23%, Singapore investment income +66% ✅, §1.7/§9.1) are consistent with a scaled advisory build-out, but the *proposition names* and platforms are not verified (§12 item 8).
- **Unit trusts, bancassurance and structured products** ✅/⚠ — the FY24 release verifies the business mix (fees from wealth and bancassurance driving NOII +22.6% ✅); the fund-supermarket/order-routing layer between the bank, the fund managers and the insurers is standard industry architecture ⚠ and not public for Maybank.
- **The Singapore wealth booking hub** — high-net-worth clients across the region book into the Singapore entity (MAS-supervised, §9) for regional diversification; the onshore/offshore booking split between Maybank Malaysia and Maybank Singapore is a classic two-centre structure ⚠ (see the [Private Banking Guide](private_banking_guide.md) for the two-centre canon).

---

## 5. Digital Channels: Maybank2u and MAE

### 5.1 Maybank2u: The Twenty-Five-Year Channel

- **Maybank2u launched in 2000 as Malaysia's first internet banking platform** ✅/⚠ (§1.3) — two and a half decades of continuous operation make it the longest-running digital channel in Malaysian banking. The platform has been through successive re-platformings (product-level revamps are public via Maybank's own announcements ⚠; the underlying channel architecture is not — §12 item 2). Maybank2u remains the mass retail digital front-end, now in the MAE era as the "classic" channel alongside the app ✅/⚠.
- **Maybank2u Biz / Maybank2E** — the business/corporate channel family (§3.2) ✅/⚠ product names public; SME digital outcomes quantified in FY2024 reporting (SME loan origination +36% in Singapore, +27% in Indonesia over the M25+ period ✅).

### 5.2 MAE: The App-First Tiger

- **Launch and age** ✅/⚠ — MAE (the "Maybank App Experience/All-in-one" app — full brand expansion ⚠) launched in **2020**: Maybank's October 2025 release frames the RM4-trillion milestone as covering "five years, since the launch of its MAE app," and secondary coverage marks "#MAETurns5" (maybank.com news 29 Oct 2025 ✅/⚠; theledger.asia, says.com, Maybank LinkedIn ✅/⚠). So MAE is the 2020-vintage app-first retail channel.
- **Scale claims** ✅/⚠ — Maybank positions MAE as **"Malaysia's #1 Mobile Banking App"** (Maybank's own claim, October 2025 ✅/⚠), and secondary coverage reports **10.7 million users** carrying "nearly half of Malaysia's mobile banking volume" (theledger.asia — ⚠ single-secondary; treat the user count as approximate). A Maybank executive quote via says.com adds colour: **"users have created nearly eight million Tabung through the app"** ✅/⚠ (Tabung = the in-app savings-pot feature; the quote is attributed to **Giorgio Migliarina, Maybank's Group Chief Technology and Digital Officer** ⚠ single-secondary — a rare public naming of the group's technology leadership).
- **The RM4-trillion anchor** ✅ — more than **RM4 trillion in digital transactions since 2020**, with MAE ranked the top bank app for **DuitNow transfers and FPX** (maybank.com 29 Oct 2025) — the app is the front door of the entire payments estate (§3).
- **What MAE is, architecturally** (structural ⚠): a mobile front-end over the domestic retail estate — accounts, DuitNow/FPX payments, Tabung savings pots, budgeting insights, card controls, and (in the SG version) FAST/ PayNow-style functionality via Maybank Singapore (§9). The classic app-plus-core pattern: the app is the product surface; the ledgers remain on the core estate (§2). MAE's personal-finance features (Tabung, spend insights) are data products (§6).
- **Digital onboarding** ✅ — FY2024 verified: digitally opened and activated CASA accounts in Malaysia doubled, contributing 10.7% of overall deposit growth (Asian Banker release) — the app-led acquisition engine is quantitatively proven.
- **SG digital app** ✅/⚠ — Maybank Singapore operates its own digital banking app (Maybank2u SG / MAE-equivalent branding ⚠ — product family public, current app naming flagged); the SG franchise's digital deposit growth is verified (deposits +18.7% in Singapore in FY2024, Asian Banker release ✅).

### 5.3 QR and Regional Interoperability

- **DuitNow QR / QRIS** — Maybank is a top DuitNow participant (§3.1 ✅/⚠) and, through Maybank Indonesia, sits on Indonesia's QRIS rail ⚠ structural. The ASEAN QR cross-border interoperability layer (Malaysia–Thailand first, then the regional web) is the mechanism by which Maybank's QR estate becomes cross-border ⚠ structural — flagged because Maybank's specific certification status on each bilateral link was not verified this pass.

### 5.4 The Channel Estate at a Glance

| Channel | Era | Function today | Status |
|---|---|---|---|
| Maybank2u (desktop/web) | 2000 launch — Malaysia's first internet banking | Full-service retail digital banking | ✅/⚠ (launch); ⚠ (current architecture) |
| Maybank2u mobile app | Smartphone era (2010s) ⚠ | Mobile banking before MAE | ✅/⚠ (product); ⚠ (dates) |
| MAE app | 2020 launch | App-first retail: payments, Tabung, insights | ✅/⚠ (§5.2) |
| Maybank2E / Maybank2u Biz | Corporate digital era ⚠ | SME/corporate cash management | ✅/⚠ (product); ⚠ (platform) |
| SG digital app (Maybank2u SG / MAE-equivalent) | Singapore franchise | SG retail banking incl. FAST/PayNow-style flows | ⚠ (naming flagged) |
| Branch & ATM network | 1960 → present | Physical channel; ~2,600 branches group-wide ⚠ | ✅/⚠ (counts dated) |

The estate table shows the layering this guide keeps insisting on: **channel generations are public; the channel platform underneath each is not** (§12 item 18). The MAE-generation apps are the current point of investment — the RM4-trillion release (§5.2) is Maybank marketing MAE as the group's digital flagship, and the M25+ CASA-doubling figure (§1.7) is the measurable result.

### 5.5 MAE's Feature Set and the Data Loop

| MAE feature (public product surface ⚠) | Function | Data/AI linkage (§6) |
|---|---|---|
| Payments hub (DuitNow, FPX, QR, bill pay) | Retail payment initiation on the national rails | Transaction stream for analytics and AML |
| Tabung savings pots | Goal-based ring-fenced savings inside the app | Savings-behaviour data; 8M+ pots ⚠ |
| Spend insights & budgeting | Categorised spend views | The visible face of the analytics estate |
| Card controls & alerts | Instant card management | Real-time decisioning/fraud signals |
| Digital onboarding | Account opening in-app | Funnel analytics (CASA doubling ✅) |
| Rewards/lifestyle marketplace | Loyalty and merchant offers | Proposition analytics, merchant data |

The feature table completes the MAE picture: the app is simultaneously the **payments front-end** (§3), the **savings/insights data product** (§6) and the **acquisition engine** (CASA doubling ✅) — which is why Maybank's marketing of MAE (the RM4-trillion release) and its M25+ digital outcomes are two descriptions of the same estate. Which of these features run on which platform remains §12 item 18.

---

## 6. Data and AI

### 6.1 The Verified Data Points

- **Maybank's public AI/analytics agenda exists and is measurable** ✅ — the FY2024 results release verifies data-driven outcomes: **"Enhanced analytics and algorithm contributed to a 31% increase in insurance and family takaful policy sales along with a 57% increase in daily auto renewal premiums"** (Asian Banker release, attributed to M25+ digital/analytics execution ✅). These are the kind of numbers that only a deployed analytics estate (propensity models, next-best-action, pricing algorithms) produces — the *outcomes* are primary-source-grade even though the platform is not named.
- **Digital-transaction telemetry** ✅ — RM4+ trillion of digital transactions since 2020 (§5.2) is, among other things, a statement about data-platform scale: the group records, reconciles, and analyses a national-scale transaction stream.
- **Leadership anchor** ✅/⚠ — the group's technology leadership is public enough to name: **Giorgio Migliarina, Group Chief Technology and Digital Officer** (says.com attribution, ⚠ single-secondary), reporting into a CEO (Khairussaleh Ramli) whose M25+ narrative explicitly claims digital enablement and "hyper-personalisation" (Asian Banker release quote: "stronger integration of our banking franchise in ASEAN, enabling solutions for enhanced customer hyper-personalisation" ✅).

### 6.2 The Platform Layer: Inference, Honestly Flagged

- **What is not public** ⚠ — the data warehouse/lake, the analytics tooling, the AI/ML platform, the model inventory, and the cloud substrate of the Maybank data estate are not disclosed by Maybank in the sources this pass could reach (§12 item 10). §1.2's mention of Google Cloud as a datapoint "verifiable where third parties published them" reflects the *class* of hyperscaler-partnership reporting that exists around Maybank's digital agenda ⚠ — no Google Cloud–Maybank contract detail was verified in this pass, and the claim is deliberately not upgraded beyond ⚠.
- **The structural template** (from the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) and the DBS/Citi guides' data sections, unmarked as industry structure): a group of Maybank's scale runs a central data platform fed by the cores (§2), the payments estate (§3), cards (§4.1) and channels (§5); an analytics layer (BI, customer analytics, real-time decisioning for fraud/personalisation); and increasingly ML/AI workloads (the FY24 takaful/insurance uplift is the observable signature of the latter). MAE's budgeting/Tabung features are customer-facing data products on that estate (§5.2).
- **The honest closing position**: Maybank's *claims* about data/AI are verifiable (M25+ outcomes, analytics uplift ✅); Maybank's *platform* is not (§12). No fabrication is attempted here.

### 6.3 Data Use-Cases Mapped From Verified Outcomes

| Verified outcome (FY2024/M25+, ✅ Asian Banker release) | Implied data capability | Status of the capability claim |
|---|---|---|
| 31% increase in insurance & family takaful policy sales via "enhanced analytics and algorithm" | Propensity/next-best-action models over the customer base | Outcome ✅; model/platform ⚠ |
| 57% increase in daily auto-renewal premiums | Real-time pricing/optimisation on renewal events | Outcome ✅; engine ⚠ |
| Digitally opened CASA doubled (10.7% of deposit growth) | Digital onboarding funnel analytics | Outcome ✅; tooling ⚠ |
| Hyper-personalisation named in CEO's M25+ successor framing | Customer 360, real-time decisioning | Strategy ✅; platform ⚠ |
| Transaction-banking proposition +16pp | Usage analytics driving product bundling | Outcome ✅; stack ⚠ |
| RM4+ trillion digital transactions processed (2020–2025) | National-scale data ingestion, reconciliation, analytics | Volume ✅; warehouse/lake ⚠ |

The table is the section's method in miniature: **Maybank verifiably does data-driven banking; it does not verify its data stack.** Any guide that named the stack would be fabricating — this one does not.

### 6.4 The AI-Governance Context

- **Regulators are ahead of disclosure here** (structural, cross-ref the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) and the repo's AI-governance canon): BNM and MAS both published AI/analytics principles and expectations for financial institutions through the 2020s (fairness, explainability, governance of models and third-party AI) — the *regulatory* framework around Maybank's AI estate is public even though the estate itself is not.
- **Model-risk management** — the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) canon: any bank running the analytics that produced Maybank's verified FY24 uplifts (§6.1) runs model-validation, model-risk inventory and AI-governance processes under those regulatory expectations ⚠ structural; Maybank's model-governance organisation is not public.
- **The talent-and-organisation anchor** ✅/⚠ — the naming of a Group CTDO (Giorgio Migliarina, ⚠) and the CEO's explicit M25+ digital narrative (✅) indicate a centralised technology-and-data function reporting to the top of the group — the organisational precondition for the group-wide data estate §6.2 describes.

---

## 7. Islamic Finance Systems: Maybank Islamic

### 7.1 The Business Layer

- **Maybank Islamic Berhad, established 2008** ✅ (§1.3/§1.4), is the group's Islamic banking subsidiary and claims to be **the largest Islamic bank in Malaysia and ASEAN by assets** ✅/⚠ (Maybank's own claim; Wikipedia secondary — the "largest in ASEAN" phrasing is Maybank's, flagged).
- **The strategy anchor** ✅ — Maybank's CEO on FY2024 results: the group will deepen its Islamic franchise and drive "**technological innovation for Islamic financial services**" (Asian Banker release quote) — i.e., Islamic fintech innovation is an explicit M25+/successor-strategy plank.
- **The product surface** ✅/⚠ — Islamic deposits (e.g., the iSAVE family — product names ⚠), Islamic financing (home/auto/cards via takaful-compliant structures), and Islamic wealth/investment products; served through the same Maybank2u/MAE digital channels with Shariah-compliant product variants (structural ⚠ — the exact product catalogue is public on maybank2u but not re-verified item-by-item here).

### 7.2 The Systems Reality: Modules or Separate Engines?

- **The industry pattern** (unmarked structural knowledge, cross-ref the [Core Banking Systems Guide](core_banking_systems_guide.md)): Islamic banking systems come in two flavours — (a) **Islamic modules on a conventional vendor estate** (the SIBS-class vendors and Temenos-class vendors all ship Islamic variants — profit-rate instead of interest, takaful-aware product parameters, Shariah audit trails), and (b) **standalone Islamic cores**. In Malaysia — where Islamic banking is a first-class licensed activity (Maybank Islamic is a separate licensed bank) — the regulator (BNM) expects a licensed Islamic bank to run its own books with full segregation of funds; the systems question is whether that segregation is a separate engine or a segregated module-set on a shared estate.
- **Maybank Islamic's engine** ⚠ — which of the two flavours Maybank Islamic runs, on which vendor estate, and how it interoperates with the conventional Maybank core for cross-entity customer journeys (e.g., a Maybank2u session spanning conventional and Islamic accounts) is **not public** (§12 item 11). Structural inference: the SIBS-class vendor ecosystem that dominates Maybank's domestic estate (§2.1) markets Islamic-capable cores across Malaysia and the Gulf ⚠ — consistent with, but not proof of, Maybank Islamic's estate.
- **Etiqa Takaful** ✅/⚠ — the takaful arm (Etiqa, §4.2) is the Islamic-insurance counterpart; FY2024 results verify takaful growth (life/family takaful premiums +17.1%, Etiqa PBT +54.7% ✅ — Asian Banker release) but not the takaful policy-administration systems (§12 item 9).
- **Cross-refs:** the [Core Banking Systems Guide](core_banking_systems_guide.md) documents the conventional/Islamic vendor classes; the [Universal Banking Model Guide](universal_banking_model_guide.md) covers the holding-company shape that houses a licensed Islamic subsidiary alongside the conventional bank.

### 7.3 The Islamic Product-to-System Map

| Islamic product family | Underlying contract concept | System-layer implication | Status |
|---|---|---|---|
| Islamic deposits (savings/current/fixed) | Wadiah / mudharabah (profit-sharing instead of interest) | Profit-rate engine, Shariah-compliant parameterisation on the deposit core | ✅/⚠ (products public); ⚠ (engine) |
| Islamic financing (home, auto, personal) | Murabaha / ijarah / takaful-linked structures | Asset-finance origination with ownership/takaful steps | ✅/⚠ products; ⚠ engine |
| Islamic cards | Ujrah / takaful-compliant card structures | Card engine with Islamic billing | ⚠ structural |
| Trade & SME Islamic finance | Murabaha-based trade instruments | Trade system with Islamic contract templates | ⚠ structural (§10.3) |
| Takaful (Etiqa family) | Tabarru' (donation) pool model | Takaful policy administration, surplus management | ✅ (results); ⚠ (systems, §12 item 9) |
| Islamic wealth/investment | Shariah-screened unit trusts & structured products | Screening and portfolio tools | ⚠ structural |

The map matters because Islamic banking is *not* a product overlay in Malaysia — it is a separately licensed book (§7.2) whose ledgers, profit computation and audit trails must satisfy both BNM and the Shariah advisory function. Which estate runs those ledgers is §12 item 11; the map above states only what the product family *requires* of any estate.

### 7.4 The Digital Islamic Journey

- **Serving Islamic customers through conventional channels** (structural ⚠) — Maybank's Islamic customers use the same Maybank2u/MAE apps (§5) with Shariah-compliant product variants; the systems challenge is the *cross-entity session*: one login spanning conventional (Maybank) and Islamic (Maybank Islamic) accounts with different ledgers, profit computations and regulatory books. This is the daily integration burden §7.2 flagged, and it is invisible in Maybank's public product marketing.
- **Shariah governance technology** (structural ⚠) — a licensed Islamic bank runs Shariah committee processes, product-approval workflows, and Shariah audit trails; the systems layer (fatwa/product-parameter libraries, audit logging of contract structures) is standard for the market and not public for Maybank Islamic.
- **The innovation lane** ✅ — the CEO's successor-strategy statement ("technological innovation for Islamic financial services," §7.1 ✅) signals that the group intends the Islamic estate — not just the conventional one — to be a digital-innovation surface in the post-M25+ era, consistent with Malaysia's global position as an Islamic-finance technology hub (structural).

---

## 8. Risk and Regulatory Context

### 8.1 The AML/Financial-Crime Stack

- **The system class** (structural; the [FircoSoft Guide](fircosoft_guide.md) is the repo's deep-dive) — a bank of Maybank's scale runs the standard four-layer financial-crime estate: (1) **name/entity screening** (FircoSoft-class list-screening against sanctions and watchlists, applied at onboarding and on payment messages), (2) **transaction monitoring** (behavioural rules and analytics over the payments/current-account streams), (3) **customer risk rating / KYC** (risk-based due diligence, CDD/EDD workflows), and (4) **case management / investigations and regulatory reporting** (STR submissions to the FIU). Which *products* Maybank Malaysia and Maybank Singapore run in each layer is not public (§12 item 12); the FircoSoft-family tools dominate the Asian correspondent-banking layer structurally.
- **The dual-supervision reality** ✅/⚠ — Maybank Malaysia answers to **Bank Negara Malaysia (BNM)** (Anti-Money Laundering, Anti-Terrorism Financing and Proceeds of Unlawful Activities Act 2001 — the Malaysian AML statute, structural) and Maybank Singapore to **MAS** (MAS Notice 626 on AML/CFT, and the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md) canon — cross-ref). The two regimes interoperate through group-level financial-crime governance; the group's group-wide AML platform is not public (§12 item 12).

### 8.2 The Singapore Enforcement Context: The S$3-Billion Case

- **The verified MAS action** ✅ — on **4 July 2025, MAS imposed composition penalties totalling S$27.45 million (US$21.55 million) on nine financial institutions** for AML/CFT breaches related to the 2023 money-laundering case involving more than S$3 billion in assets (CNA, 4 July 2025 — extracted this pass ✅). **Credit Suisse** received the highest penalty (S$5.8 million); **UOB, UBS, UOB Kay Hian and Citibank** received the next-highest; **Julius Baer, Blue Ocean Invest, Trident Trust Company and LGT Bank** were also penalised. MAS said the breaches arose from "poor or inconsistent implementation" of AML/CFT policies and controls across four areas (customer risk assessment lapses, among others), that the financial institutions had embarked on remediation, and that the actions **marked the conclusion of its enforcement against financial institutions with a "material nexus"** to the case. MAS also took action against 18 individuals. Context: this was the second-largest cumulative AML penalty after the 1MDB case (S$29.1 million on eight banks, 2016–2017) ✅.
- **Where Maybank sits in that record** ✅ — **Maybank Singapore is not among the nine FIs named in CNA's list of penalised institutions**. This guide therefore states plainly: in the July 2025 concluding wave of the S$3-billion-case enforcement, Maybank Singapore was *not* penalised (per the CNA/MAS record verified this pass). Any claim that "Maybank was fined in the S$3-billion case" would be ❌ against this record. (Maybank Singapore's own AML history under MAS — including any earlier supervisory engagement in the case's 2023–2025 examination phase — is beyond what this pass verified; see §12 item 13.)
- **Why this matters for the systems map** ✅ — MAS's stated findings (breaches arising from implementation, not policy design; customer risk-assessment lapses mis-rating persons of interest) are precisely the failure modes that screening/TM/KYC *configuration and data quality* produce — the systems lesson every bank in the region, Maybank included, draws from the case is that AML engines are only as good as their risk-rating parameters and reference data.

### 8.3 BNM Themes and Group Risk Posture

- **BNM's supervisory themes** (structural, cross-ref the [Enterprise Risk Management Guide](enterprise_risk_management_guide.md) and [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)): financial-crime risk, cyber resilience, outsourcing/cloud oversight, and conduct run through BNM's supervisory expectations for Malaysian banks in the 2020s — Maybank, as the systemic domestic leader, is a named entity in BNM's enforcement and examination cycles structurally ⚠ (no specific current BNM action against Maybank was verified this pass — earlier-era BNM enforcement involving Maybank reported in press ⚠ not re-verified, §12 item 14).
- **D-SIB standing** ✅ — Maybank is on MAS's **inaugural domestic systemically important banks (D-SIB) list** (alongside DBS, OCBC, UOB, Citibank, Standard Chartered and HSBC — cross-ref the [Citibank guide](citibank_software_systems_guide.md) §on D-SIBs and the [Banks in Singapore Guide](banks_in_singapore_guide.md)); the specific D-SIB buffer applicable to Maybank Singapore is in MAS notices, not re-verified here ⚠.
- **FY2024 risk metrics** ✅ (Asian Banker release): group **CET1 14.90%**, total capital ratio **18.04%**, **LCR 134%** (vs 100% requirement), gross impaired-loans ratio **1.23%** (down 11 bps), loan-loss coverage **126.9%**, net credit charge-off **26 bps** — a well-capitalised, low-credit-loss profile that contextualises the systems estate's risk appetite.

### 8.4 The Financial-Crime Controls Layer-by-Layer

| Control layer | What it does | Typical failure mode (per MAS's 2025 findings, §8.2) |
|---|---|---|
| Customer risk assessment & rating | Rates each customer's ML/TF risk to drive CDD depth | Mis-rating persons of interest — found at Julius Baer, Blue Ocean Invest, Citibank, Credit Suisse, UOB Kay Hian ✅ (CNA) |
| Name/entity screening | Sanctions & watchlist checks at onboarding and on payments | List/data gaps, fuzzy-match tuning issues |
| Transaction monitoring | Rules/anomaly detection over payment streams | Threshold/typology gaps letting suspect flows pass |
| Enhanced due diligence (EDD) | Deeper review for high-risk relationships | Inconsistent implementation — MAS: breaches arose from "poor or inconsistent implementation," not missing policy ✅ |
| Case management & STR | Investigation workflow and regulatory reporting | Slow closure, weak audit trails |
| Remediation programme | Fixing findings under supervisory monitoring | MAS: FIs "have embarked on remediation ... MAS will monitor their progress closely" ✅ |

The layer table pairs the repo's standard control canon (see the [FircoSoft Guide](fircosoft_guide.md)) with the specific failure modes MAS documented in the July 2025 actions — the systems-engineering lesson (configuration and data quality, not policy absence) applies to every FI in the region, Maybank included.

### 8.5 The S$3-Billion Case Timeline (Verified via CNA)

| Date | Event | Status |
|---|---|---|
| From 2021 | MAS/authorities investigations into the transnational case begin | ✅ (CNA) |
| August 2023 | Islandwide police raids; ten foreign nationals arrested (all later convicted and deported) | ✅ (CNA) |
| Early 2023 → early 2025 | MAS supervisory examinations of the FIs identify AML/CFT breaches | ✅ (CNA, MAS statement) |
| 4 July 2025 | MAS imposes S$27.45M composition penalties on nine FIs; action against 18 individuals; marks conclusion of FI enforcement with "material nexus" | ✅ (CNA) |
| Context | 1MDB-era comparison: S$29.1M on eight banks, 2016–2017 | ✅ (CNA) |

Maybank Singapore's position in this timeline is stated in §8.2: not among the nine penalised ✅; any pre-2025 examination-phase engagement is unverified ⚠ (§12 item 13).

### 8.6 Cyber, Outsourcing and Operational Resilience

- **The technology-risk regime** (structural, cross-ref the [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md)) — Maybank Malaysia and Maybank Singapore both operate under technology-risk and outsourcing expectations (BNM's RMiT-class and MAS's technology notices through the 2020s): board-level technology governance, resilience testing, secure software development, and third-party/cloud risk management. The group's public posture — a named Group CTDO (§6.4), M25+ digital outcomes — is consistent with a mature technology-risk organisation ⚠.
- **The outsourcing/cloud question** — Maybank's use of hyperscalers and vendor hosting (the §1.2 Google-Cloud-class datapoint ⚠) sits inside those outsourcing regimes; which workloads are in the cloud and under which oversight model is not public (§12 item 10).
- **Resilience in the payments era** — with RM4+ trillion of digital transactions since 2020 (§5.2 ✅) and DuitNow/FPX national-leadership claims (⚠), the operational-resilience bar on Maybank's payments estate is national infrastructure-grade: the failure of the top DuitNow bank's real-time rail would be a national payments event (structural inference, ⚠ — no Maybank outage record was researched this pass).
- **The conduct and culture layer** — MAS's 2025 enforcement commentary ("most of the financial institutions had established AML/CFT policies and controls ... breaches arose out of poor or inconsistent implementation," ✅ CNA) is the regulator's clearest statement that conduct risk in AML is an implementation-and-culture problem, not a policy-document problem — the framing Maybank's own compliance organisation would recognise (structural).

---

## 9. Singapore Angle: The Second Home Market

### 9.1 The Franchise

- **Sixty-five years in Singapore** ✅ — Maybank opened its first Singapore branch on **South Bridge Road in December 1960** (§1.3), the same year as its Malaysian founding; Singapore is the group's "second home market" (Maybank's own language ✅/⚠) and its largest international profit centre.
- **Entity shape** ✅/⚠ — Maybank operates in Singapore as **Maybank Singapore Limited**, the group's Singapore-incorporated banking subsidiary holding a **full bank licence** from MAS ⚠ (the licensed-entity structure is public via MAS registers; the subsidiary-incorporation history — branch-then-subsidiary — is ⚠ not re-verified this pass). The Straits Times reporting on the 2025-era CEO appointment refers to "Malayan Banking's Singapore branch," reflecting the legal-entity complexity of the franchise ✅/⚠ (see below). Alongside the bank: **Maybank Securities Pte. Ltd.** (the Kim Eng equities arm, §4.2) and the wealth-management businesses.
- **FY2024 results** ✅ (Asian Banker release): Maybank Singapore PBT **SGD 702.20 million (+16.6%)**; net income **SGD 1.29 billion (+14.0%)**; NOII **+57.9%** to SGD 598.62 million on wealth and global-markets fees; loans +8.9% and deposits +18.7% growth in the Singapore market — the franchise is the group's regional wealth and markets engine.
- **Leadership** ✅/⚠ — **Alvin Lee was appointed Maybank Singapore country CEO and CEO of the Singapore branch**, per The Straits Times ("Maybank appoints Alvin Lee as new CEO for Singapore": "Mr Lee was also appointed the new country CEO and CEO of Malayan Banking's Singapore branch, Maybank said in a statement") — verified at search-snippet level ⚠ (article date and predecessor not captured this pass).
- **Footprint** ⚠ — a network of branches and wealth centres in Singapore in the low-to-mid twenties ⚠ (approximate; Maybank's own current branch/centre count not re-verified this pass — §12 item 15).

### 9.2 The Singapore Systems Angle

- **Rails membership** ⚠ structural — Maybank Singapore participates in Singapore's retail rails: **FAST** (real-time SGD transfers) and the **PayNow** proxy-ID layer, SGQR acceptance, and MEPS+ style ATM sharing (the [Nets Singapore Guide](nets_singapore_guide.md) and [Payment Rails Guide](payment_rails_guide.md) document the rails; Maybank's specific memberships are operator-listed but not re-verified item-by-item this pass ⚠). The SG digital app (§5.2) fronts these rails.
- **MAS-regulated estate** — the Singapore franchise runs under MAS's technology-risk and outsourcing regime (the [MAS Regulations & Guidelines Guidelines Guide](mas_regulations_guidelines_guide.md) canon: technology risk management notices, cyber hygiene, outsourcing, and the AML Notice 626 layer of §8) ✅/⚠ structural; MAS's 2025 enforcement messaging (§8.2) is the regulatory weather for every FI in Singapore, Maybank included.
- **D-SIB and systemic role** ✅ — Maybank's place on the inaugural D-SIB list (§8.3) makes the Singapore franchise systemically important, which drives supervisory expectations on its technology resilience.
- **The regional hub function** ⚠ structural — Singapore functions as the group's regional treasury/global-markets hub (NOII +57.9% on global-markets fees, §9.1 ✅) and wealth booking centre (§4.2) — the systems corollary is that regional FX/markets and wealth platforms concentrate in the SG estate ⚠ (§12 item 16).

### 9.3 The Singapore Franchise at a Glance

| Franchise element | Public facts | Systems note | Status |
|---|---|---|---|
| Maybank Singapore Limited (full bank licence) | SG-incorporated subsidiary; MAS-licensed | Own core vs group-shared core ⚠ | ✅/⚠ entity; ⚠ core |
| Retail banking (deposits, cards, loans) | Deposits +18.7%, loans +8.9% in FY2024 ✅ | SG digital app, FAST/PayNow rails | ✅ (results); ⚠ (platforms) |
| Wealth management | NOII +57.9% on wealth/global-markets fees ✅ | Private-banking platforms (§4.2) | ✅ (results); ⚠ (platforms) |
| Maybank Securities Pte. Ltd. | Kim Eng equities arm; SGX member | Trading stack ⚠ | ✅/⚠ entity; ⚠ platforms |
| Treasury/global markets | Regional hub; SGD funding base | FX/markets systems ⚠ | ✅ (results); ⚠ (§12 item 16) |
| MAS-regulated technology | Full MAS tech-risk/outsourcing regime | Notice-626 AML estate (§8) | ⚠ structural |

The at-a-glance table closes the Singapore angle the same way every other section closes: **the franchise's results are verified numbers; its estate is a flagged map.** Singapore is where Maybank's two regulator worlds (BNM and MAS) meet in one group — the systems governance of that duality (group platforms shared across both regimes) is itself a §12 item.

### 9.4 The Singapore Path, 1960 → 2026

| Era | Singapore milestone | Systems significance | Status |
|---|---|---|---|
| 1960 | First branch, South Bridge Road (December) | Founding network predating separation | ✅ (§1.3) |
| 1960s–1990s | Branch network growth in the city-state | Branch-era retail systems | ⚠ not researched |
| 2011 | Kim Eng acquisition → regional brokerage hub | Securities/trading estate enters SG | ✅/⚠ (§1.3) |
| 2010s era | Subsidiary/local-incorporation moves reported | Entity structure: branch vs Maybank Singapore Limited | ⚠ (§9.1) |
| 2020s | Digital app era; deposits +18.7% in FY2024 ✅ | SG digital channels on FAST/PayNow rails | ✅ (results); ⚠ (platforms) |
| 2025 era | Alvin Lee appointed country CEO | Leadership continuity for the MAS-regulated estate | ✅/⚠ (ST snippet) |
| Ongoing | D-SIB supervision; MAS enforcement era | Technology and AML resilience expectations | ✅ (D-SIB); ⚠ (buffers) |

The path table anchors the Singapore angle in the same verified chronology as §1.6: the franchise is as old as the group itself, has grown through branch, broker, subsidiary and digital eras, and is today the group's most MAS-exposed, systemically important international estate.

---

## 10. Worked Example: Cymbal Bank × Maybank — A Regional Correspondent Relationship

*Conventions:* **Cymbal Bank** is the repository's fictional mid-size Singapore bank persona (see the [Citibank guide](citibank_software_systems_guide.md) worked-example canon: nostro relationships, rails membership, KYC/AML handshakes, documentary credits). This example extends those conventions to a Malaysian correspondent: it is illustrative architecture, not a claim about any real contract.

**The scenario.** Cymbal Bank (Singapore, mid-size, SGD-centric) has a growing base of customers who trade with and remit to Malaysia, plus importers drawing goods from Malaysian suppliers. Cymbal has no Malaysian banking licence, no MEPS/PayNet membership, and no MYR balance sheet. It needs the tiger: **Maybank as its Malaysian agent**.

### 10.1 MYR Clearing: The Nostro/Agent Relationship

- **The nostro** (standard correspondent mechanics, per the repo canon): Cymbal opens a **MYR nostro account at Maybank Malaysia** (the agent bank). Cymbal's SGD–MYR customer flows net through this account; Maybank provides Cymbal with MYR clearing via its own MEPS/PayNet membership — i.e., Maybank's domestic clearing membership is *rented* to Cymbal through the nostro.
- **The flow for a Cymbal customer paying a Malaysian beneficiary** (mechanics, brief):
  1. Cymbal customer instructs a MYR payment (internet banking, Cymbal side).
  2. Cymbal screens and debits the customer (SGD or MYR), converts if needed, and sends a SWIFT MT103 to Maybank Malaysia (Maybank's SWIFT BIC), covering from the MYR nostro.
  3. Maybank credits the beneficiary — if the beneficiary banks with Maybank, on-us posting on the domestic core (§2); if elsewhere, a **DuitNow proxy-ID transfer or MEPS IBFT** to the beneficiary's bank.
  4. Same-day value in MYR; Cymbal's nostro is debited in the daily reconciliation.
- **Cut-offs and limits** (structural ⚠): Maybank's own RPP/MEPS cut-offs and the nostro balance/limits govern the service; Cymbal monitors the nostro via SWIFT MT940/950 statements (standard).

### 10.2 DuitNow Interoperability: Proxy-ID Reach Without Membership

- The modern variant: Cymbal's retail app wants to offer **instant MYR transfers to Malaysian mobile numbers** — a DuitNow capability Cymbal cannot have directly (DuitNow is a PayNet-participant rail). Through Maybank as an **indirect/aggregating participant** ⚠ (structural concept — whether Maybank offers this exact wholesale service is not public), Cymbal's app would:
  1. Take the beneficiary's MY mobile number + name, and call Maybank's DuitNow lookup (proxy-ID resolution) via an API/switch interface ⚠.
  2. Debit the Cymbal customer, convert, and instruct Maybank to **push a DuitNow transfer** from a Cymbal-designated funding account.
  3. The beneficiary receives the MYR in near-real-time, regardless of which Malaysian bank they use — because Maybank's DuitNow membership spans the whole Malaysian retail network.
- **The regional QR mirror**: Cymbal's Singapore QR estate (SGQR/PayNow) and Maybank's DuitNow QR estate meet through the ASEAN cross-border QR interoperability layer (§5.3) ⚠ — a tourist/merchant flow rather than a clearing flow, but the same relationship (Cymbal needs a Malaysian participant; the tiger is the biggest one).

### 10.3 Trade Finance: A Documentary Credit Through the Tiger's Network

**Scenario:** Cymbal's corporate client (an SG importer) buys electronics components from a Kuala Lumpur manufacturer that insists on a letter of credit.

1. **Issuance** — Cymbal issues the **documentary credit** (its own trade-finance system, per the repo canon: LC issuance, SWIFT MT700 to the advising bank). Cymbal's credit risk on the SG importer is its own; the trade mechanics need a Malaysian advising/confirming bank.
2. **Advice** — Cymbal sends the MT700 to **Maybank Malaysia** (selected for its domestic network and its Islamic-finance capability — see step 5). Maybank advises the LC to the KL beneficiary.
3. **Shipment and presentation** — the beneficiary ships and presents documents (invoice, bill of lading, packing list) to Maybank KL; Maybank checks documents against the LC terms (documentary-examination workflow on its trade system ⚠ structural) and forwards them to Cymbal.
4. **Settlement** — on Cymbal's acceptance, Maybank KL claims reimbursement: either from Cymbal's MYR nostro (simple, §10.1) or via SWIFT reimbursement to Maybank's own account — the MYR leg settles through the same agent-clearing as any other payment.
5. **The Islamic variant** (the tiger's differentiator) — if the KL supplier prefers Shariah-compliant structures, Maybank Islamic can advise/confirm an **LC-compatible Islamic trade instrument** (murabaha-based trade finance) within the same Maybank group journey (§7) — a capability Cymbal cannot replicate from Singapore. This is illustrative of the group's Islamic-trade architecture ⚠, not a claim about a specific Maybank Islamic product name.

**The trade-instrument map** (standard SWIFT trade mechanics, per the repo's trade guides):

| Instrument | SWIFT message | Issuer → counterparty | Maybank's role in the example |
|---|---|---|---|
| Documentary credit (LC) | MT700 (issue), MT720 (transfer) | Cymbal (issuer) → Maybank (advising) → KL beneficiary | Advising/confirming bank in Malaysia |
| LC amendment | MT707 | Cymbal → Maybank | Relay to beneficiary |
| Collection | MT400/MT410 | Beneficiary's bank → Cymbal | Maybank as presenting/collecting bank |
| Guarantee / SBLC | MT760 | Cymbal or Maybank (counter-guarantee) | Maybank issues local-language guarantee to KL beneficiary |
| Reimbursement | MT740/MT742 | Cymbal's reimbursing bank → Maybank | MYR settlement via the nostro (§10.1) |
| Islamic trade variant | Underlying murabaha contract + standard messages | Maybank Islamic advises | Shariah-compliant alternative to the conventional LC |

The instrument map keeps the worked example honest: every message is standard SWIFT trade traffic (structural, unmarked); the only Maybank-specific claim is that the group can play the Malaysian advising/confirming role through its domestic network and its Islamic subsidiary — which is what the relationship is for.

### 10.4 KYC/AML Handshakes

- **Correspondent due diligence** (standard, per the [FircoSoft Guide](fircosoft_guide.md) and MAS Notice 626 canon): Cymbal performs **correspondent-bank CDD on Maybank** (and vice versa) — ownership, AML controls, licences (BNM for Maybank Malaysia, MAS for Maybank Singapore), and the D-SIB/supervisory context of §8.3.
- **Screening handshakes**: each party screens its own customers and payments (FircoSoft-class name screening on the SWIFT messages both ways — §8.1); neither party sees the other's full screening output; the control boundary is the correspondent agreement's warranties (standard).
- **The 2025 lesson applied** (§8.2): the MAS enforcement wave of July 2025 raised the *documented-controls* bar for every SG FI and its correspondents — Cymbal's file on Maybank would now evidence Maybank Singapore's clean status in the S$3-billion-case wave (not among the nine penalised ✅) and Maybank's group-level AML governance ⚠.

### 10.5 What the Tiger's Network Is Worth

- For Cymbal, the relationship monetises exactly what Maybank has built and this guide has mapped: **domestic clearing at national scale** (MEPS/PayNet membership, DuitNow/FPX leadership — §3 ✅), **the app-era retail network** (MAE's 10.7 million users ⚠ as the beneficiary-reach argument — §5), **the ASEAN-10 branch footprint** for trade corridors (§1.5), **the Islamic-finance engine** as a product differentiator (§7), and **the Singapore franchise's own depth** (Maybank Securities, wealth, the D-SIB balance sheet — §9) making Maybank both the Malaysian agent and a Singapore competitor-ecosystem player. The correspondent relationship is, in systems terms, Cymbal buying API-shaped access to the tiger's estate — with the nostro as the key to the ledger.

### 10.6 Relationship Lifecycle and Operational Resilience

The correspondent relationship is not a point-in-time integration; it has a lifecycle with systems consequences at every stage:

1. **Establishment** — the correspondent agreement, the nostro account opening (Cymbal's MYR account at Maybank), SWIFT key/BIC validation, and the correspondent CDD file (§10.4). Both sides record the relationship in their internal systems of record (structural).
2. **The standing data** — the nostro account number, Maybank's BIC and branch codes, the MYR clearing cut-offs, and the fee schedule live in Cymbal's payments reference data; any change (a Maybank BIC/branch reorganisation, a new rail code) propagates through static-data updates (standard operations, ⚠).
3. **Daily operations** — MT103/MT202 traffic out, MT940/950 statements in; nostro reconciliation against Cymbal's mirror account; breaks (unmatched payments, value-date differences) worked through the standard investigation workflow (structural).
4. **Nostro management** — Cymbal funds the MYR nostro according to forecast flows (treasury's liquidity task, §10.1); Maybank may extend a correspondent credit line or require pre-funding depending on the relationship tier (standard; terms not public for any real pair).
5. **Testing and resilience** — periodic SWIFT connectivity testing, business-continuity call-tree exercises, and fallback arrangements (if SWIFT is down, tested fallback channels) — the resilience canon of the [Payment Rails Guide](payment_rails_guide.md) applied to the pair.
6. **Exit and wind-down** — the mirror accounts closed, outstanding items cleared, and the CDD file archived — the unglamorous end state every correspondent agreement must define (structural).

The lifecycle makes the systems point concrete: what Cymbal buys from Maybank is not a single API but a *maintained relationship estate* — reference data, reconciliation, liquidity, resilience — in which Maybank's own systems (the core that posts the nostro, the payments hub that clears MYR, the compliance estate that screens) are the counterparty infrastructure. That is the tier-1 domestic bank as correspondent: the tiger's ledger is the relationship's ledger too.

---

## 11. Claims Status and Verification Notes

**Verification convention: ✅ = verified in this research pass (primary or secondary sources); ⚠ = flagged (inferred, approximate, single-source, or structural); ❌ = disputed (the public record contradicts the claim); "not public" = Maybank does not disclose and no external source exists either way.**

### 11.1 The Claims-Status Table

| Claim | Source | Status |
|---|---|---|
| Incorporated 31 May 1960; operations began 12 September 1960 in Kuala Lumpur | Maybank history & corporate-milestones pages; Wikipedia; CompaniesHistory | ✅ verified |
| Singapore branches opened in 1960, first on South Bridge Road (December) | Maybank corporate-milestones page | ✅/⚠ verified (year); ⚠ month |
| Founders Khoo Teck Puat and Oei Tjong Ie | Wikipedia (secondary) | ⚠ flagged |
| Kwong Yik Bank (Malaysia, est. 1913) sold by Maybank to Rashid Hussain (Dec 1996); merged into RHB Bank 1997 | Wikipedia (Maybank & RHB timelines); The Capital Journal | ✅/⚠ verified (secondary) |
| Maybank acquired Kwong Yik Bank in the 1980s consolidation era | Not pinned in this pass | ⚠ unverified |
| Group operates in ~18 countries incl. all ten ASEAN nations; 2,600+ branches; 43,000+ employees | Wikipedia; aggregators | ✅/⚠ (18 countries consistent); ⚠ branch/employee counts dated |
| Maybank Islamic Berhad established 2008; largest Islamic bank in Malaysia/ASEAN by assets | Wikipedia; Maybank claims | ✅/⚠ |
| Maybank Investment Bank Berhad (ex-Aseambankers, renamed 2009) | Wikipedia | ✅/⚠ secondary |
| Kim Eng Holdings acquired 2011 → Maybank Kim Eng | Wikipedia; 2011 press | ✅/⚠ secondary |
| Total assets RM888B (FY2021); RM1-trillion era in 2020s | Wikipedia citing Maybank AR2021; aggregators | ✅ (FY2021 figure); ⚠ RM1T crossing date |
| Khairussaleh Ramli Group PCEO (announced 27 Jan 2022) | Maybank announcement | ✅ |
| FY2024 net profit record MYR 10.09 billion (+7.9%); PBT MYR 13.70 billion (+9.3%); ROE 11.1%; CET1 14.90%; LCR 134%; GIL 1.23% | The Asian Banker re-dissemination of Maybank FY2024 results (3 Mar 2025; extracted this pass) | ✅ verified |
| Maybank Singapore FY2024: PBT SGD 702.20M (+16.6%), net income SGD 1.29B (+14.0%), NOII +57.9% | The Asian Banker release | ✅ verified |
| M25+ "Supergrowth" segments (wealth, bancassurance, SME, cash mgmt) income +21–23%; digitally opened CASA doubled (10.7% of deposit growth); transaction-banking proposition +16pp | The Asian Banker release | ✅ verified |
| Etiqa FY2024 PBT MYR 1.52B (+54.7%); No. 1 in Malaysian general insurance/takaful at 16.6% share | The Asian Banker release | ✅ verified |
| RM4+ trillion in digital transactions over five years since 2020 (MAE-era); top bank for DuitNow transfers and FPX | maybank.com newsroom, 29 Oct 2025; theledger.asia; says.com | ✅ (RM4T figure, Maybank's own release); ⚠ "double Malaysia's GDP" framing (URL-slug/secondary) |
| MAE launched 2020 ("turns five" Oct 2025); ~10.7 million users; ~8 million Tabung savings pots created | maybank.com release framing; theledger.asia; says.com (CTDO Giorgio Migliarina quote) | ✅/⚠ (2020 launch consistent across sources); ⚠ user count & Tabung figure single-secondary |
| Group CTDO is Giorgio Migliarina | says.com quote attribution | ⚠ single-secondary |
| MAS S$27.45M composition penalties on nine FIs (4 July 2025) over the S$3-billion money-laundering case; Credit Suisse S$5.8M highest; UOB/UBS/UOB Kay Hian/Citibank/Julius Baer/Blue Ocean Invest/Trident Trust/LGT named; Maybank Singapore NOT among the nine | CNA, 4 July 2025 (extracted this pass) | ✅ verified |
| Alvin Lee appointed Maybank Singapore country CEO and CEO of the Singapore branch | The Straits Times | ✅/⚠ (search-snippet level; article date/predecessor not captured) |
| Silverlake SIBS / SIBS II vendor facts (flagship core, 30 years, modular SOA); SIBS deals bundle digital subsidiaries | silverlakeaxis.com/services/Core-Banking; links.sgx.com vendor announcements | ✅ (vendor facts) |
| Maybank as a Silverlake SIBS customer | Apps Run The World SIBS customer database; industry reporting | ⚠ not re-verified this pass (indirect/aggregator) |
| Maybank on MAS inaugural D-SIB list | [Citibank guide](citibank_software_systems_guide.md); [Banks in Singapore Guide](banks_in_singapore_guide.md) (cross-ref) | ✅ cross-ref |
| Maybank2u launched 2000 — Malaysia's first internet banking | Maybank milestones; CompaniesHistory | ✅/⚠ |
| Amex card business in Malaysia acquired 2006 | Wikipedia timeline; §1.6 | ✅/⚠ secondary |

### 11.2 The Honesty Note: Methodology

1. **Verified anchors are strong on history, products, results and enforcement** — the founding dates (§1.3) are primary-source verified via Maybank's own pages; the group-entity names (§1.4) are public; the product brands (Maybank2u, MAE, Etiqa) are public; the FY2024 financial and M25+ outcome figures are verified against The Asian Banker's re-dissemination of Maybank's own results release ✅; the MAS July 2025 penalties are verified against CNA's reporting ✅.
2. **The inference zone is the engine layer** — which core runs which entity, the payments-hub orchestration, the data platform, the AML platform: the *class* of system is certain for a bank of Maybank's size, the *identity* is only partially public (§12).
3. **One deliberate correction** — the Kwong Yik Bank "lineage" claim (§1.3): the common telling that Maybank descends from Malaya's oldest bank is ❌ contradicted by the record — Maybank was a temporary owner-seller, and the 1913 Kwong Yik line runs into RHB. A second record-check is stated affirmatively rather than as a correction: **Maybank Singapore was not among the nine FIs penalised by MAS in July 2025** (§8.2) — claims to the contrary would be ❌.
4. **Deliberately not verified** (out of scope): per-market regulatory filings, vendor contracts and pricing, internal roadmaps, exact branch counts by year, Maybank Singapore's unit-level inventory, and Maybank's internal AI/data platform specifics.

---

## 12. What Could Not Be Verified

The following are the materially non-public specifics of the Maybank estate. Each is flagged ⚠ and should be treated as unknown, not as "likely X":

1. **Core-banking system identities per entity** ⚠ — which Maybank entity runs which core platform, the application names, and the ledger engines for deposits/loans (see §2; the [Core Banking Systems Guide](core_banking_systems_guide.md) documents the vendor classes). Maybank has never published an OCBC-style "our retail core is X" statement.
2. **The Silverlake–Maybank contract specifics** ⚠ — which Maybank group entities license SIBS/SIBS II, the scope (retail core? Islamic modules? regional subsidiaries?), contract values and renewal dates. Vendor announcements naming Maybank exist in the record's general sense but no specific contract was re-verified this pass (§2.1).
3. **Regional-subsidiary cores** ⚠ — the core platforms of Maybank Indonesia, Maybank Vietnam, MCB Pakistan, Maybank Philippines, Maybank Cambodia and Maybank Singapore Limited (own core vs group-shared) are not public (§2.2).
4. **Core-modernisation roadmap** ⚠ — whether/when the domestic estate moves off the legacy packaged core to a modern platform; M25+ outcomes are public, the platform plan is not (§2.2).
5. **Payments-hub orchestration** ⚠ — the identity of the payment-switch/hub layer(s) that route between Maybank2u/MAE, MEPS, PayNet (DuitNow/FPX), SWIFT and FAST (§3).
6. **Cards engine identities** ⚠ — card management, authorisation/switch, acquiring, rewards and 3-D Secure/tokenisation platforms for the Visa/Mastercard/Amex/MyDebit portfolio (§4.1).
7. **Brokerage technology at Maybank Securities / Maybank Kim Eng** ⚠ — trading platforms, SGX/Bursa connectivity, client accounting (§4.2).
8. **Wealth/private-banking platform** ⚠ — the portfolio/advisory/booking systems behind Maybank Private Wealth in Malaysia and Singapore (§4.2).
9. **Etiqa insurance & takaful policy-administration engines** ⚠ — policy admin, claims, distribution systems (§4.2/§7).
10. **Data & AI platform** ⚠ — warehouse/lake, analytics tooling, ML/AI platform, model inventory, cloud substrate (incl. any Google Cloud relationship); only M25+ *outcomes* are verified (§6).
11. **Maybank Islamic's core estate** ⚠ — standalone Islamic core vs Islamic modules on a shared SIBS-class estate; interoperation with the conventional core for cross-entity journeys (§7).
12. **AML/financial-crime platform identities** ⚠ — name-screening, transaction-monitoring, KYC and case-management products at Maybank Malaysia and Maybank Singapore (§8.1).
13. **Maybank Singapore's supervisory history in the S$3-billion-case era** ⚠ — any examination-phase engagement before the July 2025 conclusion wave is beyond this pass's sources (§8.2; the penalty list itself is ✅).
14. **BNM enforcement history** ⚠ — earlier-era BNM penalties involving Maybank reported in press; amounts/dates not re-verified this pass (§8.3).
15. **Current Singapore footprint count** ⚠ — exact branch/wealth-centre numbers in Singapore (approximate "low-to-mid twenties" used in §9.1).
16. **Regional treasury/markets platform concentration in Singapore** ⚠ — which FX/markets and wealth platforms sit in the SG estate (§9.2).
17. **D-SIB buffer specifics** ⚠ — the exact additional capital/supervisory buffers MAS applies to Maybank Singapore (§8.3/§9).
18. **Maybank2u/MAE hosting and channel-middleware architecture** ⚠ — re-platforming history, API layer, integration bus identities (§5).

---

## 13. Glossary

| Term | Definition |
|---|---|
| **Maybank** | Malayan Banking Berhad — Malaysia's largest bank by market capitalisation and total assets; incorporated 31 May 1960; Bursa Malaysia: 1155; the "tiger" of Malaysian banking |
| **Maybank2u** | Maybank's internet-banking platform, launched 2000 as Malaysia's first internet banking service; still the mass retail digital channel alongside MAE |
| **MAE** | Maybank's app-first retail mobile banking app, launched 2020; positioned by Maybank as Malaysia's #1 mobile banking app with ~10.7M users (⚠ secondary) and 8M+ Tabung savings pots (⚠) |
| **Maybank2E / Maybank2u Biz** | Maybank's business/corporate digital channel family for SME and corporate cash management |
| **DuitNow** | PayNet's real-time proxy-ID retail transfer rail in Malaysia (mobile-number/NRIC addressing); Maybank claims top-bank status for DuitNow transfers |
| **FPX** | Financial Process Exchange — PayNet's online e-payments rail from bank accounts to merchants/billers; Maybank claims top-bank status |
| **DuitNow QR** | The interoperable QR payment rail operated by PayNet in Malaysia, linked into the ASEAN cross-border QR layer |
| **MEPS** | Malaysian Electronic Payment System — Malaysia's interbank ATM and funds-transfer network layer, consolidated under PayNet's operations in the modern era |
| **PayNet** | Payments Network Malaysia — the Malaysian national retail-payments operator running DuitNow, FPX, JomPAY, MyDebit and the MEPS-era infrastructure |
| **Maybank Islamic** | Maybank Islamic Berhad — the group's licensed Islamic banking subsidiary (est. 2008), claiming the largest Islamic bank position in Malaysia/ASEAN |
| **Etiqa** | Maybank group's insurance & takaful brand (est. 2007); No. 1 in Malaysian general insurance/takaful by market share (FY2024) |
| **Maybank Kim Eng / Maybank Securities** | The group's regional brokerage franchise, from the 2011 Kim Eng Holdings acquisition; Maybank Securities Pte. Ltd. is the Singapore equities arm |
| **SIBS / Silverlake** | Silverlake Axis's Integrated Banking Solution (SIBS / SIBS II) — the flagship regional core-banking product family; Maybank's domestic estate is structurally associated with the SIBS class (⚠ indirect) |
| **Nostro** | "Our" account held at another bank — the MYR nostro Cymbal Bank holds at Maybank in §10 is the archetype |
| **BNM** | Bank Negara Malaysia — Malaysia's central bank and financial regulator (Maybank's home supervisor) |
| **MAS** | Monetary Authority of Singapore — Singapore's central bank and financial regulator; Maybank Singapore's supervisor; issued the July 2025 S$27.45M penalties on nine FIs in the S$3-billion case |
| **D-SIB** | Domestic systemically important bank — Maybank is on MAS's inaugural D-SIB list |
| **Bursa Malaysia** | The Malaysian stock exchange; Maybank is listed as Bursa Malaysia: 1155 |
| **Tabung** | "Savings pot" feature inside the MAE app — the in-app goal-based savings product cited in Maybank's RM4-trillion milestone release |
| **Cymbal Bank** | The fictional/illustrative mid-size Singapore bank persona used across this repository's worked examples — not a real bank |

---

## 14. References and Further Reading

**Primary/company:**
- Maybank newsroom — "Maybank facilitates over RM4 trillion in digital transactions since 2020" (29 Oct 2025): https://www.maybank.com/en/news/2025/10/29/maybank-rm4-trillion-digital-transactions-since-2020-double-malaysia-gdp.page
- Maybank "Our History" and "Our Corporate Milestones": https://www.maybank.com/en/about-us/history.page and https://www.maybank.com/en/about-us/corporate-milestones.page (page-level verification this pass; some sub-pages timed out — see §11)
- Maybank Annual Reports hub: https://www.maybank.com/en/investor-relations/financial-overview/annual-reports.page
- Maybank newsroom index (appointments, results): https://www.maybank.com/en/news
- Maybank Annual Report 2021 (Corporate book PDF) — cited via Wikipedia for FY2021 figures

**Results and enforcement (extracted this pass):**
- The Asian Banker — "Maybank net profit up 7.9% to $2.2B in 2024" (re-dissemination of Maybank's FY2024 results, 3 Mar 2025): https://www.theasianbanker.com/press-releases/maybank-net-profit-up-7-9-to-2-2b-in-2024
- CNA — "Billion-dollar money laundering case: MAS slaps 9 financial institutions with S$27.45 million in penalties" (4 Jul 2025): https://www.channelnewsasia.com/singapore/mas-27-million-penalty-banks-credit-suisse-uob-ubs-money-laundering-case-5220456
- Ocorian insight on the same MAS action: https://www.ocorian.com/knowledge-hub/insights/mas-takes-financial-institutions-task-anti-money-laundering-breaches

**Secondary reporting and history:**
- The Straits Times — "Maybank appoints Alvin Lee as new CEO for Singapore": https://www.straitstimes.com/business/maybank-appoints-alvin-lee-as-new-ceo-for-singapore
- Wikipedia — Maybank: https://en.wikipedia.org/wiki/Maybank; Kwong Yik Bank: https://en.wikipedia.org/wiki/Kwong_Yik_Bank
- The Capital Journal — "Kwong Yik Bank, the Origins of Malaysia Chinese Banking Industry": https://thecapitaljournal.com/kwong-yik-bank-the-origins-of-malaysia-chinese-banking-industry/
- TheLedger.asia — "Maybank Breaches RM4 Trillion in Digital Transactions as MAE App Turns Five": https://theledger.asia/maybank-breaches-rm4-trillion-in-digital-transactions-as-mae-app-turns-five/
- SAYS — "Maybank's MAE App Hits RM4 Trillion In Digital Transactions": https://says.com/my/tech/maybank-4-trillion-digital-transactions

**Vendor-side:**
- Silverlake Axis — Core Banking (SIBS/SIBS II product page): https://silverlakeaxis.com/services/Core-Banking
- Silverlake Axis SGX announcements (SIBS contract structure): https://links.sgx.com (vendor-announcement archive)
- Apps Run The World — Silverlake SIBS customer database (Maybank listed; paywalled): https://www.appsruntheworld.com/customers-database/products/view/silverlake-sibs

**Sibling guides in this repository (cross-references used throughout):**
- [Citibank Software Systems Guide](citibank_software_systems_guide.md) — structural template; [DBS Software Systems Guide](dbs_software_systems_guide.md), [OCBC Software Systems Guide](ocbc_software_systems_guide.md) — SG-bank genre
- [Core Banking Systems Guide](core_banking_systems_guide.md), [Payment Rails Guide](payment_rails_guide.md), [FircoSoft Guide](fircosoft_guide.md), [Enterprise Risk Management Guide](enterprise_risk_management_guide.md), [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Banks in Singapore Guide](banks_in_singapore_guide.md), [Universal Banking Model Guide](universal_banking_model_guide.md), [Private Banking Guide](private_banking_guide.md), [Nets Singapore Guide](nets_singapore_guide.md), [DBS Bank Guide](dbs_bank_guide.md)

---

### Closing: The Tiger's Ledger

Maybank is the rare large Asian bank whose *history*, *products* and *results* are all public while its *engines* remain almost entirely opaque. This guide has drawn the line precisely where the evidence allows: the 1960 founding and the ASEAN-10 expansion are verified; Maybank2u (2000) and MAE (2020) and the RM4-trillion digital-transaction milestone are verified; the FY2024 record (MYR 10.09 billion net profit, CET1 14.90%, Maybank Singapore's SGD 702-million PBT) is verified; the Silverlake-era core, the payments hubs, the Islamic engines, the data platform and the AML stack are flagged, not fabricated — the vendor classes are industry structure, the identities are §12's unknowns. What the record does support, firmly, is the shape of the estate: a universal-bank group whose Malaysian retail ledger still runs on the classic packaged-core pattern of its era, wrapped in app-first channels that now carry a nation's digital payments, supervised from Kuala Lumpur and Singapore, and measured — like the tiger itself — less by what it says about its machinery than by what the machinery demonstrably does. When Cymbal Bank opens its MYR nostro at Maybank and sends that first MT103 to Kuala Lumpur, it is not buying software; it is buying access to the tiger's ledger.
