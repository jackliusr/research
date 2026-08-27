# Airalo: The eSIM Store Without Borders

*A solution-architect's deep-dive on Airalo — the travel-eSIM marketplace that popularized "buy a digital SIM for your destination before you land": the company profile (founded 2019, Delaware-incorporated with a Singapore operational base, the world's-first-eSIM-store claim, the $287.3M funding arc from Antler/Sequoia seed to a CVC-led $220M Series C and a $1B+ "first eSIM unicorn" valuation), the eSIM category itself (eUICC hardware, the GSMA SGP.22/SGP.02/SGP.32 standards, SM-DP+ remote provisioning, eSIM vs physical SIM vs carrier roaming), the product surface (the iOS/Android app, local/regional/global plans, top-ups, instant activation), the pricing economics (wholesale carrier capacity vs retail packages), a head-to-head landscape table (Airalo vs Holafly vs Ubigi/Transatel-NTT vs Nomad vs Saily/Nord Security vs carrier roaming), the banking angle (why eSIMs are showing up in neobank and premium-card propositions), and a worked example — Cymbal Bank's premium-card travel-benefits program issuing eSIM data as a cardholder perk.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Connectivity / Travel-Tech (technology/)
> **Audience:** Solution architects, product/platform architects, travel-tech and fintech leads, and anyone designing digital services around global connectivity (eSIM provisioning, partner APIs, cross-border data services, cardholder benefits)
> **Last Updated:** August 2026

**Cross-references:** [technology/ibm_cloud_guide.md](technology/ibm_cloud_guide.md) (the **style exemplar and platform companion** — the hybrid-cloud guide whose verification conventions this guide follows), [technology/super_app_guide.md](technology/super_app_guide.md) (the **super-app context** — its §1 landscape table already flags eSIM as a feature super-apps and neobanks bolt on, e.g. Revolut; this guide is the deep-dive on the eSIM layer itself), [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) and [banking/insurance_products_processes_compliance_guide.md](banking/insurance_products_processes_compliance_guide.md) (the **Singapore data-protection and consumer-product compliance themes** — PDPA obligations and the personal-data-minimization discipline that §7's worked example cross-references instead of re-deriving), [banking/banks_in_singapore_guide.md](banking/banks_in_singapore_guide.md) (the **Cymbal Bank persona** — the repository's only bank: the author's firm, a European wholesale banking group with a Singapore branch, and the market context its premium-card program lives in).

**How to read this guide.** This is the *travel-connectivity* deep-dive of the series: the super-app guide documents the platform pattern that absorbs eSIM as a feature, the banking guides document the regulatory and product context a bank adds eSIM benefits into, and this guide documents the *connectivity layer itself*. A fast path: read **§1** (what Airalo is), **§2** (the history — founding and the funding rounds that matter), **§3** (the eSIM category and GSMA standards), **§4** (the product and technology), **§5** (the pricing economics), **§6** (the competitive landscape table), **§7** (the worked example that ties it to a real bank architecture), and **§8** (the banking–telco intersection); treat **§9** as the summary, **§10**–**§11** as the honest verification record, **§12** as the glossary, and **§13** as the source list. Every factual claim carries a ✅ (verified from a source fetched in this pass), ⚠ (could not be confirmed from primary sources), or ❌ (refuted) marker; the unverified items are consolidated in **§11**.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The History: Founding & Funding](#2-the-history-founding--funding)
3. [The eSIM Category: Standards and Architecture](#3-the-esim-category-standards-and-architecture)
4. [The Product & Technology](#4-the-product--technology)
5. [The Pricing Economics](#5-the-pricing-economics)
6. [The Competitive Landscape](#6-the-competitive-landscape)
7. [The Worked Example: Cymbal Bank](#7-the-worked-example-cymbal-bank)
8. [The Banking–Telco Intersection](#8-the-bankingtelco-intersection)
9. [The Summary: The Connection Without Borders](#9-the-summary-the-connection-without-borders)
10. [Verification and Claims-Status](#10-verification-and-claims-status)
11. [What Could Not Be Verified](#11-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [References](#13-references)

---

## 1. The Overview

### 1.1 What Airalo Is

**Airalo** is a travel-eSIM marketplace: a digital store where travelers buy prepaid mobile-data packages — delivered as eSIMs (embedded SIMs, software profiles downloaded into a phone's eUICC chip) — for 200+ countries and regions, priced below typical carrier roaming ✅ (airalo.com and airalo.com/about-us/about-airalo, fetched). The company's own positioning, repeated verbatim across its site and press materials: **"the world's first eSIM store"** ✅ (airalo.com/about-us/about-airalo, fetched) and **"the world's first and largest eSIM marketplace"** ✅ (Series B press release via PRNewswire, fetched — company claim).

The one-line summary this guide unpacks: **Airalo is a marketplace that sits between hundreds of mobile carriers and millions of travelers** — it brokers wholesale data capacity and carrier roaming plans, repackages them into small, cheap, destination-specific eSIM bundles, and sells them through its own app and website, plus a B2B2C channel (Airalo Partners) that lets other companies resell or white-label the same connectivity ✅/⚠ (the wholesale mechanics are documented by TechCrunch's July 2023 reporting, fetched; the margin structure itself is not public — §5 and §11).

### 1.2 The Company Profile at a Glance

| Aspect | Detail | Status |
|---|---|---|
| **Founded** | 2019 (company communications; operating entity AirGSM Pte. Ltd. incorporated in Singapore 25 October 2018) | ✅ (Wikipedia; sgpbusiness.com, fetched) |
| **Founders** | Ahmet Bahadir Özdemir (CEO) and Abraham Burak (COO); third co-founder Duran Akcaylier left in 2020 | ✅ (Wikipedia, fetched) |
| **HQ / incorporation** | Delaware, USA (registered agent address in Lewes, Del.); operational base in Singapore; offices in Toronto, Istanbul, Singapore + remote team | ✅ (PRNewswire dateline; Wikipedia; TechCrunch; Series A release, fetched) |
| **Legal entity** | AirGSM Pte. Ltd. (UEN 201836421Z, Singapore), a GSMA member | ✅ (sgpbusiness.com; gsma.com membership page, fetched) |
| **Mission** | "To provide global data connectivity for all people" | ✅ (airalo.com/about-us/about-airalo, fetched — company claim) |
| **Users** | 20M+ (July 2025); 30M+ (current site "fast facts", 2026) | ✅ (CVC release, Wikipedia; about page — company claim) |
| **Coverage** | 200+ countries and regions; 689 eSIM plan combinations (July 2023) | ✅ (airalo.com; TechCrunch, fetched) |
| **Total funding** | US$287.3M across seed, Series A, Series B, Series C | ✅ (arithmetic from verified rounds; consistent with third-party trackers) |
| **Valuation** | Over US$1B post-Series-C — billed as the industry's "first eSIM unicorn" | ✅ (CVC release, Reuters via Wikipedia, fetched) |
| **Category role** | The category-defining pure-play travel-eSIM marketplace | ✅/⚠ (leadership claim is Airalo's own; competitor data in §6) |

### 1.3 Why Airalo Matters Beyond Travel

Three reasons this company earns a technology-series guide rather than a consumer-review page:

- **It is the demand-side proof point for the eSIM standard.** Airalo's growth — from 5.1M customers in July 2023 ✅ (TechCrunch, fetched) to 20M+ in July 2025 ✅ (CVC release, fetched) — tracks the industry shift Apple accelerated when the US iPhone 14 shipped without a SIM tray in 2022 ✅ (Wikipedia eSIM article, fetched). The standard's fate and Airalo's fate are correlated.
- **It is a distribution-layer business, not an infrastructure business.** Airalo owns no spectrum and no towers; it buys capacity wholesale from carriers and resells it ✅ (TechCrunch, fetched). That is an architecture pattern — a thin aggregation layer over a fragmented supply market — that banks and super-apps are now copying to add connectivity as a feature (see §7 and §8).
- **It is the first "unicorn" of its category.** The CVC-led $220M Series C at a $1B+ valuation (July 2025) ✅ (CVC release, fetched) signals that investors believe the travel-eSIM aggregation layer is a durable, scalable business — which is exactly the claim a bank's vendor-selection committee needs to stress-test (see §10's audit and §11's unverified list).

### 1.4 Scope and Honesty Note

This guide was researched in a single bounded pass (August 2026). Where a figure comes from Airalo's own marketing or press materials, it is marked ⚠ (company claim; not independently confirmed) even when fetched directly — a company-reported user count is not the same as an audited one. Where a figure could not be confirmed from any fetched source, it is flagged ⚠ and consolidated in §11. One task hypothesis was refuted outright (a "$4.3M Series A in 2020" — no such round exists; the verified Series A is $5.4M in October 2021, §2.3) and one competitor attribution was corrected (Saily is a Nord Security product, not a Proton product, §6.2) — both marked ❌ at their point of use.

### 1.5 What Airalo Is Not

Precision about what Airalo is *not* keeps the architecture honest:

- **Not a mobile network operator.** It holds no spectrum, operates no radio access network, and is not the carrier of record for the data it sells ✅ (TechCrunch's July 2023 reporting describes the wholesale-capacity and roaming-resale model — fetched). The underlying connectivity is delivered by partner MNOs/MVNOs under their own network identities ✅ (TechCrunch, fetched).
- **Not a hardware company.** It sells profiles, not devices; its device rules exist because it depends on the eSIM capabilities built by Apple, Google, Samsung, and the chip vendors ✅ (airalo.com/how-airalo-works; Wikipedia eSIM — both fetched).
- **Not a marketplace of eSIM *providers*.** The "marketplace" framing refers to the bazaar of *carriers and carrier plans* underpinning the catalog — Airalo is the only seller on its own storefront ✅ (TechCrunch, fetched).
- **Not an infrastructure-standard-setter.** It is a GSMA member and a consumer of the GSMA SGP.22 ecosystem, not a contributor to the standards themselves ✅ (gsma.com membership page, fetched — membership verified; the standard-setting role is occupied by GSMA and its carrier/ecosystem members, §3.2).

These four negatives are exactly what make Airalo's position replicable in principle and defensible in practice: the moat is the carrier-agreement network, the catalog economics, and the brand — all softer than owning a network, which is why §7.7's risk register treats supplier concentration seriously.

---

## 2. The History: Founding & Funding

### 2.1 The Verified Timeline

Every date and amount below was verified against a source fetched during this pass (Airalo's own press releases, PRNewswire, TechCrunch, CVC, Wikipedia's Airalo article, and Singapore company records); anything less certain is marked ⚠.

| Year | Event | Amount | Status |
|---|---|---|---|
| 25 Oct 2018 | AirGSM Pte. Ltd. incorporated in Singapore (UEN 201836421Z), Marina Square address | — | ✅ (sgpbusiness.com, fetched) |
| 2019 | Airalo publicly launched; positioned from the start as "the world's first eSIM store" | — | ✅ (Wikipedia; Series A release; Silicon Canals via Wikipedia) |
| Oct 2019 | Seed round from Antler and Sequoia Capital India's Surge program | US$1.9M | ✅ (Airalo Series A release PDF, fetched; Wikipedia) |
| Oct 2021 | Series A led by Rakuten Ventures (with Surge, Antler, Singtel Innov8, Wayra/Telefónica, LG Technology Ventures, GO Ventures, Ground Control, Plug and Play, I2BF) | US$5.4M | ✅ (Airalo Series A release PDF, fetched; Tech in Asia; PhocusWire) |
| Jul/Aug 2023 | Series B led by e& Capital (the venture arm of e& / Etisalat); total funding to $67.3M; ~$280M post-money; 5.1M customers at announcement | US$60M | ✅ (PRNewswire, fetched; TechCrunch, fetched) |
| 10 Jul 2025 | Series C led by CVC (CVC Asia Fund VI, $185M of the round), with Peak XV and Antler Elevate; valuation over $1B — "the first eSIM unicorn"; 20M+ travelers | US$220M | ✅ (CVC release, fetched; BetaKit, fetched; Reuters via Wikipedia) |

### 2.2 The Founders and the Origin Story

Airalo was founded in 2019 by **Ahmet Bahadir Özdemir** (CEO) and **Abraham Burak** (COO), with a third co-founder, web application designer/developer **Duran Akcaylier**, who left the company in 2020 ✅ (Wikipedia's Airalo article, fetched). Wikipedia describes Özdemir as a serial entrepreneur and Burak as a Canadian businessman ✅ (Wikipedia, fetched). BetaKit adds color: Burak is Toronto-based, studied law in China, Switzerland, and the Netherlands, and dropped out of a law program at the University of Ottawa to build Airalo ✅ (BetaKit, fetched).

The origin story Airalo tells on its about page is simple: "Huge roaming bills, losing phone service, desperately searching for free WiFi — we didn't like that stuff, either. That's why we started Airalo" ✅ (airalo.com/about-us/about-airalo, fetched). The company's mission statement, as published: **"Airalo's mission is to provide global data connectivity for all people. We're here to liberate you from the roaming limits and fix what's been broken for years"** ✅ (same page, fetched — company claim).

The founders' own framing in the Series A release: "Our mission is very clear: We want to create the gateway to instant travel connectivity worldwide… eSIMs enable a smooth and seamless solution where people do not need legacy roaming systems or physical SIM cards anymore" ✅ (Airalo Series A press release PDF, fetched).

Two structural facts worth an architect's attention:

- **The Delaware/Singapore dual structure.** Airalo is "founded in Singapore, officially HQ'd in Delaware with offices in many other places, including Toronto" ✅ (TechCrunch, fetched); the PRNewswire Series B dateline reads "LEWES, Del." ✅ (PRNewswire, fetched); the operating entity AirGSM Pte. Ltd. is Singapore-registered and a GSMA member ✅ (sgpbusiness.com; gsma.com, fetched); and Reuters/Forbes describe it as a "Singaporean tech company" ✅ (via Wikipedia citations). For a bank's vendor due diligence this matters: the contracting entity, the operating entity, and the marketing "HQ" are three different things (see §7.4).
- **The timing nuance.** AirGSM Pte. Ltd. was incorporated on 25 October 2018 ✅ (sgpbusiness.com, fetched) while Airalo itself dates its founding to 2019 ✅ (all Airalo materials; Wikipedia) — incorporation preceded the public launch by roughly a year. Both statements are true; they answer different questions ("when was the entity formed" vs "when did the company launch").

### 2.3 The Funding Rounds in Detail

**Seed — October 2019, US$1.9M.** From Antler and Sequoia Capital India's Surge rapid-scale-up program ✅ (Airalo Series A release PDF, fetched; Wikipedia). Airalo was part of the second Surge cohort (announced October 2019) ✅ (Tech in Asia, via Wikipedia). Antler remained a lifelong backer — its follow-on vehicle Antler Elevate participated in both the Series B and the Series C ✅ (PRNewswire; CVC release, fetched).

**Series A — October 2021, US$5.4M.** Led by **Rakuten Ventures**, with Sequoia Capital India's Surge, Antler, Singtel Innov8, Wayra (Telefónica), LG Technology Ventures, GO Ventures (GO p.l.c. of Malta), Ground Control, Plug and Play, and I2BF Global Ventures ✅ (Airalo Series A release PDF, fetched; Tech in Asia; PhocusWire). Rakuten Ventures' Adit Swarup joined the board ✅ (same release). Note on the numbers: Wikipedia's article says "US$5 million," the company's own release and Tech in Asia both say US$5.4M — the $5.4M figure is the primary-source one ✅, and it reconciles exactly with TechCrunch's later statement that Airalo had "previously raised just $7.3 million" (1.9 + 5.4 = 7.3) ✅ (TechCrunch, fetched). **The frequently-circulated hypothesis of a "~$4.3M Series A around 2020" is refuted: ❌ no such round exists in any fetched source; the verified early rounds are the $1.9M seed (Oct 2019) and the $5.4M Series A (Oct 2021).** GO Ventures and LG Technology Ventures are described by TechCrunch as "two other previous backers that were in its Series A" ✅ (TechCrunch, fetched).

**Series B — announced 31 July/1 August 2023, US$60M.** Led by **e& Capital**, the investment arm of e& (the UAE-based group formerly branded Etisalat) ✅ (PRNewswire, fetched; TechCrunch, fetched). Full participant list from the press release: Antler Elevate, Liberty Global, Orange, T.Capital (the venture arm of Deutsche Telekom), Rakuten Capital, Singtel Innov8, Telefónica Ventures, Sequoia Capital India and SEA's Surge (now Peak XV Partners), KPN Ventures, and I2BF Global Ventures ✅ (PRNewswire, fetched). The round was all-equity at a **~$280M post-money valuation** ✅ (TechCrunch, fetched), brought total funding to **$67.3M** ✅ (PRNewswire, fetched), and at announcement Airalo reported **5.1M customers** ✅ (TechCrunch, fetched) and **~1M app downloads per month with 20% month-on-month revenue growth** ⚠ (company-reported figures, relayed by TechCrunch — not independently audited). TechCrunch also reported that Airalo was already fielding acquisition interest from the carrier world and that follow-on discussions were happening "at a considerably higher valuation of between $800 million and $1 billion" ✅ (TechCrunch, fetched) — a preview of the Series C.

**Series C — announced 10 July 2025, US$220M.** Led by **CVC** through its **CVC Asia Fund VI** (which contributed $185M of the round), with participation from existing investors **Peak XV** and **Antler Elevate** ✅ (CVC release, fetched; BetaKit, fetched). The round valued Airalo at **over US$1 billion**, making it "the industry's first eSIM unicorn" ✅ (CVC release, fetched; Reuters and Forbes coverage cited by Wikipedia). At announcement Airalo reported **over 20 million travelers served across 200+ destinations** ✅ (CVC release, fetched).

**Totals.** The four verified rounds sum to 1.9 + 5.4 + 60 + 220 = **US$287.3M** ✅ (arithmetic on verified figures; third-party trackers such as The Company Check list the same total). A note for the audit trail: BetaKit additionally lists **Bell Ventures** (the venture arm of Canadian telecom BCE) among Airalo's backers ⚠ (BetaKit, fetched — not corroborated by Airalo's own Series B release or Wikipedia, so flagged rather than asserted).

### 2.4 Traction Over Time

| Date | Reported metric | Source | Status |
|---|---|---|---|
| Oct 2019 (post-seed) | "190+ countries and regions," 15× customer growth since initial funding | Airalo Series A release | ⚠ company claim |
| Jul 2023 | 5.1M customers; 689 plan combinations; ~1M downloads/month; 20% MoM revenue growth | TechCrunch interview-based reporting | ✅ figures reported; ⚠ growth rates are company-stated |
| Aug 2023 | 250+ staff across 44 countries; app in 22 languages; 4.7★ App Store / 4.6★ Play; #1 travel app in several countries | Airalo Series B release (PRNewswire) | ⚠ company claim |
| 2024 (exact date not confirmed in this pass) | 10 million users milestone | Tech in Asia, via Wikipedia's reference list | ⚠ secondary source; date unconfirmed |
| Jul 2025 | 20M+ travelers; 200+ destinations; remote team across 55 countries | CVC release; BetaKit | ✅ (press-sourced; counts are company-reported) |
| Aug 2025 | 5,000+ Airalo Partners | Wikipedia (citing Tech in Asia / AsiaTechDaily) | ⚠ secondary source |
| 2026 (current) | 30M+ users on the about page "fast facts" | airalo.com/about-us/about-airalo | ⚠ company claim |

The growth story has a pandemic-shaped pause in it: COO Abraham Burak told TechCrunch that growth "basically stalled" for two years during COVID-19 and picked up momentum in 2022 ✅ (TechCrunch, fetched). The July 2025 CVC release frames the post-pandemic period as "surging global demand" with Airalo "scaling faster than ever" ✅ (CVC release, fetched — company framing).

### 2.5 The Investor Base and Its Strategic Logic

The verified cap table splits cleanly into two camps ✅ (PRNewswire Series B list and CVC Series C release, both fetched):

- **Strategic telco capital** — e& Capital (UAE), Orange, Telefónica Ventures (and Wayra in the Series A), Singtel Innov8 (Singapore), KPN Ventures (Netherlands), T.Capital (Deutsche Telekom), Rakuten Capital (Japan), plus Liberty Global — carriers investing in the aggregator that undercuts their own roaming rates. TechCrunch's explanation: better to earn a small wholesale margin on a traveler you would lose anyway than to earn nothing ✅ (TechCrunch, fetched), and e&'s interest is unsurprising given the UAE's heavily traveling expat population ✅ (TechCrunch, fetched).
- **Financial/growth capital** — CVC (via CVC Asia Fund VI), Peak XV (formerly Sequoia Capital India/SEA), Antler Elevate (from the 2019 Antler residency), LG Technology Ventures, GO Ventures, I2BF, Plug and Play, Ground Control ✅ (Series A PDF; PRNewswire; CVC release, all fetched).

The strategic-logic read for a due-diligence reader: **the telco investors are simultaneously customers, suppliers, and competitors of Airalo** — they sell it wholesale capacity (supplier), they could distribute eSIMs themselves (competitor), and they validate the category (investor). That tri-fold relationship is a governance question any bank should explore before relying on Airalo as a single source of supply (§7.5).

---

## 3. The eSIM Category: Standards and Architecture

### 3.1 What an eSIM Is

An **eSIM** (embedded subscriber identity module) is a SIM that is not a removable plastic card: it is software installed onto an **eUICC** (embedded Universal Integrated Circuit Card) chip soldered into the device ✅ (Wikipedia eSIM article, fetched). The eUICC uses the same electrical interface as a physical SIM (ISO/IEC 7816) in a small 6mm × 5mm form factor commonly designated MFF2 (Machine-to-Machine Form Factor 2) ✅ (Wikipedia eSIM article, fetched). Functionally an installed eSIM profile behaves like a physical SIM — it carries its own ICCID and network authentication key — but it can be re-programmed over the air ✅ (Wikipedia eSIM article, fetched). Every eUICC is factory-programmed with a permanent **EID** (eUICC Identifier) that the provisioning service uses to associate the device with a subscription and negotiate a secure channel ✅ (Wikipedia eSIM article, fetched).

The GSMA's own framing: "the SIM may be securely downloaded into a 'Secure Element' that can be permanently embedded inside any type of device," supported by "an ecosystem of trusted platforms and players… facilitated by the GSMA," offering "an equivalent level of security and protection to that provided by the removable SIM card" ✅ (gsma.com/solutions-and-impact/technologies/esim/, fetched).

Two user-facing properties matter for the travel use case:

- A device can hold many eSIM profiles (upwards of 20) with one or two active at a time ✅ (Wikipedia eSIM article, fetched) — so a traveler can keep the home carrier profile and add a destination profile alongside it.
- Many travel eSIM plans are **data-only** (no phone number) to keep cost down ✅ (Wikipedia eSIM article, fetched; Airalo's own blog covers making calls over data-only eSIMs).

### 3.2 The GSMA Standards: SGP.22, SGP.02, SGP.32

The GSMA maintains three eSIM specifications ✅ (Wikipedia eSIM article, fetched; GSMA eSIM site, fetched):

| Specification | Scope | Notes | Status |
|---|---|---|---|
| **SGP.22** | Consumer eSIM remote provisioning (RSP) — smartphones, tablets, wearables | The standard behind Airalo's consumer product; defines the RSP architecture, the SM-DP+ role, and the LPA (Local Profile Assistant) | ✅ (Wikipedia eSIM; GSMA SGP.22 spec reference) |
| **SGP.02** | Machine-to-machine (M2M) eSIM management | The older, IoT-oriented remote provisioning architecture for connected machines | ✅ (Wikipedia eSIM; GSMA eSIM site) |
| **SGP.32** | eSIM IoT specification, published May 2023 | Server-driven remote profile management "suited to unattended, large-scale IoT deployments"; built on the consumer spec with differences for constrained devices | ✅ (Wikipedia eSIM, fetched; GSMA eSIM IoT page) |

**SM-DP+** — Subscription Manager – Data Preparation (the GSMA spec's own name; widely expanded as "Data Preparation **Plus**" in vendor literature) — is the server-side platform that prepares, securely stores, and delivers eSIM profiles to devices ✅ (GSMA SGP.22 RSP Technical Specification, v3.1, referenced from GSMA; vendor documentation corroborating the "Plus" expansion). In plain terms: when a user scans a QR code or taps "install" in an app, the SM-DP+ (operated by the carrier or by an eSIM provider's carrier partners) authenticates the device's EID and pushes an encrypted profile to it. GSMA's own explainer frames the whole flow as "Remote SIM Provisioning" — the journey from removable to embedded SIM ✅ (GSMA "Remote SIM Provisioning: How it Works" page, fetched).

### 3.3 The Device-Timeline Context

The eSIM's adoption curve explains the travel-eSIM market's timing. The full device history, per the Wikipedia eSIM article (fetched):

| Year | Milestone |
|---|---|
| Nov 2010 | GSMA begins discussing a software-based SIM |
| Mar 2012 | ETSI meeting: Motorola sees eUICC as industrial; Apple foresees consumer eSIMs |
| Mar 2016 | The eSIM is released; Feb 2016 saw the Samsung Gear S2 Classic 3G smartwatch, the first eSIM device |
| Mar 2017 | Qualcomm demonstrates eSIM in Snapdragon hardware at Mobile World Congress |
| Sep 2017 | Apple Watch Series 3 — first Apple device with eSIM |
| Oct 2017 | Google Pixel 2 — first phone with eSIM (via Google Fi); Microsoft Surface Pro LTE ships in December |
| 2018 | iPhone XS/XR and iPad Pro (3rd gen) add eSIM; Windows 10 gains eSIM support |
| 2019 | Pixel 3a enables dual-SIM (physical + eSIM); Airalo launches the same year |
| 2020 | Motorola Razr (2020) is eSIM-only |
| Nov 2021 | Samsung Galaxy S21/S20 eSIM enabled via One UI 4 in North America |
| 2022 | **US-sold iPhone 14 / 14 Pro ship without a SIM tray — eSIM-only** — the single strongest demand-side catalyst for travel eSIM providers |
| 2024 | iPad Air (6th gen), iPad Pro (7th gen), iPad Mini (7th gen) are eSIM-only; Pixel 7 generation supports dual eSIM |
| Sep 2025 | iPhone 17 / 17 Pro eSIM-only in several countries (with larger batteries in that configuration); the iPhone Air is Apple's first device with no physical SIM capability anywhere; US Pixel 10 series (except the Fold) are eSIM-only |

Every row above is ✅ (Wikipedia eSIM article, fetched). The strategic read for this guide: **Airalo launched in 2019 at the start of the consumer ramp and rode the eSIM-only handset wave from 2022 onward** — the category's tailwind is manufactured by Apple, Google, and Samsung, not by the travel-eSIM providers themselves.

### 3.4 eSIM vs Physical SIM vs Carrier Roaming

| Dimension | Physical SIM | eSIM (travel provider) | Carrier roaming (home plan) |
|---|---|---|---|
| Acquisition | Buy/collect a card abroad or ship it | Download profile before or during trip; QR code or in-app install | Nothing to buy — automatic on the home network |
| Activation | Insert card | Minutes; needs an internet connection for download ✅ (airalo.com how-it-works) | Automatic; arrives as bill shock later |
| Cost | Local-rate data, but per-destination cards | Small packages at near-local rates (e.g., BetaKit's example: ~US$9 for 2 weeks/2GB in the UK ⚠) | Premium daily rates (e.g., CA$16/day in the BetaKit example ⚠) |
| Friction | Multiple cards to juggle, swap, and lose | Multiple profiles stored on one device, 1–2 active ✅ | None — but pricey and inflexible |
| Coverage risk | Per-country availability | Provider's carrier agreements (200+ for Airalo) | Home carrier's roaming agreements |

The trade-offs are the standard ones of the category ✅ (Wikipedia eSIM article, fetched, for the technical mechanics; BetaKit, fetched, for the price comparison — the specific Bell CA$16/day vs Airalo US$9 two-week-2GB UK figures are a single journalist's example, flagged ⚠).

### 3.5 The Travel-eSIM Market Size

Market-size claims for this category are all over the map, and none of them is auditable in a bounded research pass — they are included here with an explicit ⚠:

- CVC, at the July 2025 Series C: "the digital travel eSIM market, whilst already worth US$1 billion, is at the very early stages of becoming the main method by which consumers receive the highest quality experience abroad" ⚠ (CVC release, fetched — a private-equity investor's claim, not an independent market study).
- ResearchAndMarkets (via BusinessWire, November 2025): global **travel SIMs & eSIMs** spending to reach **US$8.7B by 2030** ⚠ (market-research vendor, paywalled methodology).
- GII Research: a narrower "global travel eSIM market" of **US$364.88M in 2024 growing to US$734.16M by 2030 (12.36% CAGR)** ⚠ — an order of magnitude below the ResearchAndMarkets number, which shows how definition-dependent these figures are.
- NextMSC: the whole **eSIM market** (not travel-specific) at **US$8.75B in 2023 → US$16.10B by 2030** ⚠.

The honest takeaway for a due-diligence reader: **the category is real and growing, but its precise size is unverifiable from public sources, and vendors disagree by an order of magnitude on scope** (§11, items 10–12).

### 3.6 The Provisioning Architecture, Read as an Architect

For a solution architect, the eSIM stack is best understood as three cooperating planes ✅ (Wikipedia eSIM article and GSMA materials, fetched):

- **The secure-element plane** — the eUICC chip in the device: factory-programmed with a permanent EID, hosting multiple profiles, exposing the ISO/IEC 7816 interface ✅. The EID is what the provisioning service uses "to associate the device with an existing carrier subscription as well as to negotiate a secure channel for programming" ✅ (Wikipedia eSIM, fetched).
- **The provisioning plane** — the SM-DP+ platform(s) operated by carriers or their enablers: prepare profiles, bind them to an EID, deliver them over the secure channel (SGP.22 for consumer devices) ✅ (§3.2).
- **The distribution plane** — the layer companies like Airalo actually build: the app, the catalog, the QR codes and deep links, the payment and top-up logic, and the carrier-agreement management ✅ (§4.1, §4.4).

The architectural consequence for anyone integrating eSIM into a product (the §7 worked example): **your integration surface is the distribution plane, not the provisioning plane.** You do not need to operate an SM-DP+ or negotiate with carriers to offer eSIM data — you need a supply-side partner (Airalo Partners, Nomad Enterprise, or a carrier) that already operates the lower two planes, and you consume their catalog/issuance API ✅/⚠ (the API capability is documented at partners.airalo.com, fetched; the underlying provisioning is Airalo's carrier-side concern, not publicly detailed).

### 3.7 The Security and Lifecycle Model

What the fetched sources actually support on eSIM security:

- **Re-programmability is the design center.** An eUICC-compatible eSIM "can be re-programmed with new SIM information" — profile deletion, re-download, and top-up are all over-the-air operations ✅ (Wikipedia eSIM, fetched). This is what makes Airalo's top-up model possible (§4.1).
- **Identity binding is cryptographic.** The EID anchors the device's identity; profile delivery happens over a GSMA-defined secure channel, with the ecosystem offering "an equivalent level of security and protection to that provided by the removable SIM card" per the GSMA ✅ (GSMA eSIM page, fetched).
- **The consumer trade-off is convenience for lock-in.** A profile can be wiped remotely and the device can be carrier-locked in ways that constrain which eSIMs can be added — Wikipedia notes that a carrier-locked phone can only add eSIMs from the locking carrier "even after a carrier unlock" ✅ (Wikipedia eSIM, fetched). Airalo's own device rules (unlocked, not jailbroken/rooted) exist precisely because the provisioning flow depends on clean device state ✅ (airalo.com/how-airalo-works, fetched).
- **Humanitarian use demonstrates the model's resilience.** QR-code-activated eSIMs were used to keep civilians connected during Gaza communication blackouts ✅ (Wikipedia's Nomad article, fetched, citing Quartz and Al Jazeera) — remote provisioning works even when retail supply chains do not.

The security posture that a bank would care about (who holds which credential, where profile data sits, how deletion is handled) is **contract-level, not public** ⚠ (§11, item 3) — the technical mechanism itself is GSMA-standard and documented ✅.

---

## 4. The Product & Technology

### 4.1 The App and the Purchase Flow

Airalo's consumer surface is the **Airalo app** (iOS and Android) plus the airalo.com website ✅ (airalo.com/how-airalo-works, fetched). The flow the company documents is a three-step loop ✅ (same page, fetched):

1. **Choose a location** — coverage for over 200 locations, selectable as a single country, a region, or the whole globe.
2. **Select a package** — every package is an amount of data valid for a period of time; some packages are unlimited-data; some include calls and texts.
3. **Install and connect** — "installing takes a few minutes and requires internet connection."

Device requirements are explicit ✅ (airalo.com/how-airalo-works, fetched): the device must support eSIMs, must not be carrier- or network-locked, and must not be jailbroken (iOS) or rooted (Android). The app handles purchase, installation, management, and **top-up** — when a package runs out or expires, the same eSIM can be topped up rather than re-purchased ✅ (same page, fetched).

App-store health (as claimed in the Series B release, August 2023): 4.7★ on the App Store and 4.6★ on Google Play, "the #1 travel app in mobile app stores in multiple countries," and the app is offered in 22 languages with a plan to reach 53 ⚠ (company claim — PRNewswire release, fetched). A referral/loyalty layer — **Airmoney** — pays S$4.00 per successful referral ✅ (airalo.com blog page, fetched).

### 4.2 Plan Taxonomy: Local, Regional, Global

Airalo's catalog is structured as ✅ (airalo.com, fetched — site navigation and how-it-works page):

- **Local eSIMs** — single-country packages (e.g., Singapore, China, Indonesia, Japan, South Korea) at the cheapest per-GB rates.
- **Regional eSIMs** — multi-country packages (Africa, Asia, Caribbean, Europe, EU + UK, Latin America, MENA, North America, Oceania).
- **Global eSIMs** — worldwide packages for multi-continent itineraries.
- **Unlimited-data packages** — a separate catalog line ✅ (site navigation; CVC release notes "30-day unlimited data bundles," fetched).
- **Calls-and-texts packages** — "dedicated data, text, and voice packages in select destinations" announced alongside the Series C ✅ (CVC release, fetched); the default remains data-only, matching the category norm (§3.1).

At the Series B (July 2023) the catalog was described as 689 distinct combinations of country, region, duration, and data size ✅ (TechCrunch, fetched) — the "marketplace" framing in action: Airalo calls its matrix a marketplace because it aggregates "a large bazaar of carriers and carrier plans underpinning the deals," not because multiple eSIM providers sell on it ✅ (TechCrunch, fetched).

### 4.3 Activation and Provisioning Mechanics

Two activation paths dominate the category ✅ (Nomad's Wikipedia article, fetched, documents both; Airalo's own flow is app-first per §4.1):

- **QR-code activation** — the provider emails or displays a QR code carrying the SM-DP+ address and activation code; the phone's settings scan it and download the profile.
- **Instant in-app activation** — the app talks to the provisioning backend directly and pushes the profile with one tap ("one-click in-app activation").

Underneath both sits the GSMA RSP machinery from §3.2: the profile is prepared by an SM-DP+ operated by the underlying carrier (or its MVNE), authenticated against the device's EID, and delivered over a secure channel. Airalo's own site describes the user-visible version: install takes a few minutes and needs an internet connection ✅ (airalo.com/how-airalo-works, fetched). Notably, eSIMs have also proven their worth in humanitarian settings — QR-code-activated eSIMs were used to keep civilians connected during communication blackouts in Gaza ✅ (Wikipedia's Nomad article, fetched; Quartz and Al Jazeera coverage cited there) — a reminder that this provisioning layer is genuinely infrastructure, not just travel convenience.

### 4.4 The Coverage Model: How Airalo Actually Delivers Connectivity

Airalo owns no radio spectrum and no network. TechCrunch's July 2023 reporting documents the supply model ✅ (TechCrunch, fetched):

- **Wholesale capacity purchase** — direct agreements with mobile network operators for bulk data at wholesale rates, repackaged into retail eSIM bundles.
- **Roaming-plan resale** — reselling international roaming plans that individual carriers have already structured, at a smaller margin.
- **Demand/cost measurement technology** — Airalo built tooling that "measures demand and corresponding costs and pricing for these different eSIM packages," i.e., dynamic catalog economics.

Why would a carrier sell through a competitor-to-roaming intermediary? TechCrunch's analysis: if the traveler is already shopping elsewhere for connectivity before the trip, the home carrier has likely lost that customer anyway; a wholesale deal with Airalo lets the carrier win *some* margin on a customer it would otherwise lose ✅ (TechCrunch, fetched). The company's coverage claim is **200+ countries and regions** ✅ (airalo.com and press materials, fetched), with Wikipedia noting service is "compatible with mobile networks in most countries" ✅ (Wikipedia, fetched). **5G availability in specific destinations could not be verified from a fetched primary page in this pass** ⚠ (§11, item 6).

### 4.5 The B2B2C Layer: Airalo Partners

Airalo Partners (partners.airalo.com) is the B2B/B2B2C extension of the marketplace ✅ (Wikipedia, fetched; partners.airalo.com, fetched). The partnership product set, from Airalo's own materials ✅ (partners.airalo.com; CVC release, fetched):

- **Airalo for Business** — a platform for organizations to manage employee connectivity: assign eSIMs, control budgets, track usage, top up. The company claims businesses can "reduce roaming costs by up to 90%" ⚠ (CVC release, fetched — company claim, no methodology).
- **White Label** — a fully branded eSIM store the partner can launch "in minutes," served from the partner's own domain ✅ (partners.airalo.com whitelabel integration guide, fetched).
- **API integrations, reseller tools, voucher programs, and affiliate partnerships** ✅ (partners.airalo.com; CVC release, fetched).
- **Scale:** 5,000+ partners as of August 2025 ⚠ (Wikipedia, citing Tech in Asia — secondary source).

No *named* flagship airline, bank, or loyalty-program deal was verifiable in this pass ⚠ (§11, item 5) — the public footprint describes the *capability* (white-label stores, vouchers, APIs) without naming marquee customers, which is itself a due-diligence data point for §7.

### 4.6 Trust, Privacy, and Compliance Surface

Airalo operates a Trust Center (trust.airalo.com), privacy policy, and legal center ✅ (site footer, fetched). As a GSMA member ✅ (gsma.com membership page, fetched) it participates in the standard's ecosystem. For a bank's vendor risk team the relevant surface is: it is a data processor holding travelers' personal data (identity, device EID, purchase history, location-adjacent usage), it operates in multiple jurisdictions, and its Singapore legal entity brings it within Singapore's PDPA regime — the data-protection angle is developed in §7.4 and cross-referenced to the repo's MAS/PDPA guides rather than re-derived here.

### 4.7 The Device-Compatibility Landscape

The addressable market for any travel-eSIM provider is defined by device support. The compatibility picture, from the Wikipedia eSIM article and Airalo's own device rules (both fetched) ✅:

| Device class | eSIM support status | Implication for the travel-eSIM market |
|---|---|---|
| iPhone (XS onward, 2018+) | Native; US iPhone 14/14 Pro eSIM-only since 2022; iPhone 17 family eSIM-only in several countries since Sep 2025 | The flagship travel-eSIM demographic is fully addressable |
| iPad (2018+) | Native; 2024 iPad Air/Pro/Mini generations are eSIM-only | Tablets add a second eSIM per traveler |
| Google Pixel (2, 2017 onward) | Native; US Pixel 10 series eSIM-only | Android addressability growing |
| Samsung Galaxy (S21+, One UI 4, Nov 2021) | Enabled in North America from late 2021 | The largest Android base came late but arrived |
| Apple Watch / Gear S2 (2016–2017) | eSIM since the earliest devices | Wearables are a future secondary market |
| Windows / Surface (2017–2018) | Windows 10 eSIM support; Surface Pro LTE | Laptop eSIM is a niche but real segment |

Constraints that shrink the addressable market: carrier-locked devices (eSIM additions restricted to the locking carrier ✅), regional model variance (Airalo notes "some regional models may not support eSIMs" ✅), and Airalo's own rules excluding jailbroken/rooted devices ✅ (airalo.com/how-airalo-works, fetched). The net effect: **the travel-eSIM TAM is effectively "recent, unlocked smartphones"** — large, growing, and skewed toward exactly the demographic premium-card programs target (§7).

---

## 5. The Pricing Economics

### 5.1 The Wholesale-to-Retail Model

Airalo's economics are those of a **thin aggregation layer**: buy data wholesale from carriers (direct capacity deals or structured roaming plans), repackage into small destination-specific bundles, and sell at retail through the app ✅ (TechCrunch, fetched). The margin structure — wholesale rates, retail markups, revenue share with partners — is **not public** ⚠ (§11, item 3). What is knowable:

- The catalog is deliberately granular: 689 combinations at Series B time, from small short-duration bundles (the Series C release cites "1 GB plans ideal for quick trips and layovers") to "30-day unlimited data bundles" ✅ (TechCrunch; CVC release, fetched).
- Prices are benchmarked against carrier roaming, not against each other: BetaKit's worked example shows a two-week 2GB UK plan at ~CA$12 on Airalo vs CA$16 **per day** for Bell roaming ⚠ (BetaKit, fetched — single example, journalist-computed).
- The travel-eSIM retail price band is roughly US$4.50–US$30 per package depending on destination, size, and validity ⚠ (market observation from competitor pricing pages and reviews, not an Airalo primary source).

### 5.2 Where the Money Is Made

| Revenue lever | Mechanism | Status |
|---|---|---|
| Retail margin | Wholesale capacity bought low, sold per-package | ⚠ margins not disclosed |
| Volume | ~1M app downloads/month (2023); 20M+ travelers (2025) | ✅ (TechCrunch; CVC release) |
| B2B2C distribution | White-label stores, APIs, vouchers, affiliate | ✅ (partners.airalo.com) |
| Loyalty/retention | Airmoney referral credits; top-up economics (customer returns to the same eSIM) | ✅ (airalo.com, fetched) |
| Dynamic pricing | Proprietary demand-and-cost measurement tech | ✅ (TechCrunch, fetched) |

### 5.3 The Strategic Question for a Would-Be Partner

For an enterprise considering the Airalo Partners channel (see §7), the pricing questions a procurement team should press on are: wholesale unit economics per destination, volume-tiered discounts, settlement frequency and netting (see §7.3), revenue-share mechanics for white-label stores, and whether the "up to 90% roaming-cost reduction" claim ⚠ has a defensible methodology behind it. None of these is publicly answerable from Airalo's marketing surface — which is normal for a private company at this stage, and exactly why §7 frames the worked example around contract-level due diligence rather than published rates.

### 5.4 Example Price Points (Illustrative, ⚠)

The following price examples are **journalistic or third-party snapshots, not Airalo's current published rate card** (Airalo's live per-destination pricing was not scraped in this pass) — included to give the reader an order-of-magnitude sense, and flagged ⚠ throughout:

| Example | Figure | Source | Status |
|---|---|---|---|
| UK, 2 weeks, 2GB (Airalo) | ~CA$12 (~US$9) | BetaKit, fetched | ⚠ single journalist example, dated |
| Bell Canada roaming, same trip | CA$16 **per day** | BetaKit, fetched | ⚠ single example, dated |
| Entry-level Airalo packages (market observation) | "from US$4.50" | third-party review sites | ⚠ not verified against live catalog |
| Nomad entry pricing (competitor) | "from USD 4.50" | nomadesim.com (search result) | ⚠ not fetched directly |

The qualitative conclusion is robust even though the figures are soft: **travel-eSIM retail pricing sits in the single-digit-to-low-tens-of-US-dollars band per package, one to two orders of magnitude below per-day carrier roaming rates** ⚠ (derived from the BetaKit example and market observation).

### 5.5 What the Economics Imply for the Category

Three structural observations follow from the verified economics:

1. **The aggregator's margin is a spread, so scale is the strategy.** Fixed costs (app, integrations, carrier-relations team) amortize over volume; TechCrunch's reporting of ~1M downloads/month in 2023 and the 20M+ traveler base of 2025 ✅ (TechCrunch; CVC release, both fetched) shows the volume engine that makes thin per-package spreads viable.
2. **The carrier relationship is the true asset.** Because Airalo buys wholesale capacity and resells carrier roaming plans ✅ (TechCrunch, fetched), its unit costs are negotiated, not discovered — which is why margins are opaque (§11, item 1) and why the telco-investor camp (§2.5) is strategically positioned on both sides of the negotiation table.
3. **B2B2C shifts the spread's location.** White-label and API partners (§4.5) don't buy retail packages; they buy at partner rates and add their own margin — which is the mechanism the §7 worked example depends on, and the reason partner-tier pricing is the single most important commercial unknown for a bank evaluating the channel ⚠ (§11, item 3).

---

## 6. The Competitive Landscape

### 6.1 The Head-to-Head Table

| Dimension | **Airalo** | **Holafly** | **Ubigi (Transatel/NTT)** | **Nomad** | **Saily** | **Carrier roaming** |
|---|---|---|---|---|---|---|
| **Founded** | 2019 ✅ (Wikipedia) | 2017 ✅ (Wikipedia) | Ubigi 2017; Transatel 2000 ✅ (transatel.com) | 2020 ✅ (Wikipedia) | 2024 ✅ (Nord Security) | — |
| **HQ** | Delaware (US) + Singapore ops ✅ | Murcia, Spain ✅ (Wikipedia) | Paris, France ✅ (Wikipedia) | Santa Clara, California ✅ (Wikipedia) | Lithuania (Nord Security) ⚠ | — |
| **Parent / ownership** | Independent (VC-backed, unicorn) ✅ | Independent (VC/accelerator-backed) ⚠ | NTT Group subsidiary since 2019 ✅ (Wikipedia) | Business line of LotusFlare Inc. ✅ (Wikipedia) | Nord Security (NordVPN/NordPass) ✅ (nordsecurity.com) | The telco itself |
| **Funding** | $287.3M total; CVC-led $220M Series C (2025) ✅ | Not publicly confirmed in this pass ⚠ | Corporate (NTT) ✅ | Not independently confirmed ⚠ | Corporate (Nord Security) ✅ | — |
| **Coverage claim** | 200+ countries/regions ✅ | ~200 countries ⚠ | 200+ destinations ✅ (transatel.com) | 200+ destinations ✅ (Wikipedia) | 150–200+ countries ⚠ (review sites disagree) | Home carrier's roaming footprint |
| **Plan style** | Data + unlimited + select talk/text ✅ | Unlimited-data focus ⚠ (marketing knowledge) | Data plans, 4G/5G, device-agnostic ✅ (transatel.com) | Data-only ✅ (Wikipedia) | Data-only ✅ (Nord Security) | Full service at daily rates |
| **Activation** | App + QR/instant ✅ | App + QR ✅ (category norm) | App + QR ✅ (category norm) | QR + one-click in-app ✅ (Wikipedia) | App-based ✅ (category norm) | Automatic |
| **B2B offering** | Airalo Partners: white label, API, vouchers, Airalo for Business ✅ | Travel-agent/affiliate channel ⚠ | Strong IoT/M2M + enterprise heritage ✅ (Wikipedia) | Nomad eSIM Enterprise (2024) ✅ (Wikipedia) | Consumer-focused ⚠ | Wholesale roaming agreements |
| **Distinctive angle** | Category leader, "first eSIM unicorn" ✅ | Unlimited-data plans, Spanish market roots ✅ | NTT-grade network engineering, IoT roots ✅ | Marketplace aggregation, tech-savvy UX ✅ | Privacy/security brand (NordVPN trust) ✅ | Zero-setup, familiar billing |

Every ✅ cell above was verified from a source fetched in this pass (Wikipedia articles for Airalo, Holafly, Nomad, Transatel, and eSIM; transatel.com; nordsecurity.com; CVC/BetaKit). Every ⚠ cell is either unconfirmed from primary sources or resting on marketing-level knowledge — consolidated in §11. **Correction of a common misattribution: Saily is a Nord Security product (the NordVPN/NordPass team), not a Proton product** ❌ (task hypothesis refuted; verified at nordsecurity.com's Saily launch materials — Proton has no travel-eSIM product in any fetched source).

### 6.2 Competitor Notes

- **Holafly** — founded 2017 in Spain by Pedro Máiquez and Yingyan Hu; began selling physical travel SIMs before shifting to eSIM; joined the Lanzadera accelerator in 2019 ✅ (Wikipedia, fetched). Positioning centers on unlimited-data plans; funding is not publicly confirmable in this pass ⚠.
- **Ubigi (Transatel/NTT)** — Transatel, founded 2000 in Paris by Jacques Bonifay and Bertrand Salomon, became an NTT Group subsidiary in 2019; Ubigi, launched 2017, delivers "global LTE/5G connectivity across 200+ destinations" for travelers and remote workers, with a strong IoT/M2M enterprise heritage ✅ (Wikipedia Transatel article and transatel.com, fetched). The most infrastructure-heavy competitor on this list.
- **Nomad** — launched 2020 as a business line of LotusFlare Inc. (Santa Clara), a telecom-software company founded by ex-Facebook and ex-Microsoft engineers; data-only marketplace plans, QR and in-app activation, 200+ destinations; expanded to enterprise in 2024 ✅ (Wikipedia, fetched).
- **Saily** — launched 2024 by Nord Security as a data-only consumer eSIM with a privacy/security brand halo and 150–200+ country coverage ✅ (nordsecurity.com, fetched; coverage figure ⚠ varies by review source).
- **Carrier roaming** — the incumbent alternative: zero setup, but premium daily pricing (the Bell CA$16/day example in §5.1) and per-carrier coverage; the eSIM-only handset wave is steadily shrinking the physical-SIM fallback ✅ (Wikipedia eSIM article, fetched).
- **The wider set** — Wikipedia's Airalo article cross-links **GigSky** and **Yesim** as further travel-eSIM players ✅ (Wikipedia Airalo article, fetched). Neither is profiled in this guide; their presence shows the category's long tail beyond the five named competitors.

### 6.3 The Landscape Read

The travel-eSIM market in 2026 has one clear category leader (Airalo, on funding, users, and mindshare ✅), one infrastructure-backed challenger (Ubigi ✅), one unlimited-data brand (Holafly ✅), one marketplace-technology challenger (Nomad ✅), one privacy-brand entrant (Saily ✅), and the carriers themselves as the ever-present incumbent. Barriers to entry are real but not prohibitive — the hard assets are carrier agreements and provisioning integrations (the SM-DP+ ecosystem of §3.2), which is why the category has consolidated around a handful of scale players rather than hundreds of startups ✅/⚠ (industry observation grounded in the verified player set).

### 6.4 How the Competitors Differ Architecturally

The six columns of §6.1 are really four distinct architectures:

- **Pure marketplace (Airalo, Nomad)** — no network assets; aggregate other carriers' capacity; the business is the catalog, the app, and the supply agreements ✅ (TechCrunch for Airalo; Wikipedia for Nomad, both fetched). Airalo's differentiation is scale and the B2B2C channel (§4.5); Nomad's is its LotusFlare software lineage ✅.
- **Network-backed enabler (Ubigi/Transatel)** — Transatel's MVNE/IoT heritage means Ubigi sits closer to the provisioning plane (§3.6); it can offer 5G and device-agnostic connectivity (SD-WAN routers, hotspots) that pure marketplaces handle through carrier partners ✅ (transatel.com, fetched).
- **Brand-led consumer play (Holafly, Saily)** — differentiation is brand and plan simplicity (unlimited-data for Holafly; privacy heritage for Saily) rather than marketplace breadth ✅ (Wikipedia; Nord Security, fetched).
- **Incumbent (carrier roaming)** — the home operator's own roaming product; zero-setup but premium-priced and per-carrier in coverage ✅ (§3.4).

For a bank selecting a supply partner, the architectural distinction that matters most is the first vs. the second: **a marketplace partner is only as good as its carrier agreements, and those agreements are exactly what is not public** ⚠ (§11, item 1) — whereas a network-backed partner's capability is more inspectable but comes with less marketplace price competition ✅/⚠ (industry observation).

---

## 7. The Worked Example: Cymbal Bank

*This section is the repository's worked-example convention applied to eSIM: it is a target-architecture proposal in the author's firm — Cymbal Bank, a European wholesale banking group with a Singapore branch (see [banking/banks_in_singapore_guide.md](banking/banks_in_singapore_guide.md) for the persona and market context). Design content is marked as such; the Airalo-side facts it rests on are marked ✅/⚠ per the series convention.*

### 7.1 The Proposition

Cymbal Bank's Singapore branch runs a premium-card travel-benefits program for its high-net-worth and corporate clients. Today the marquee perks are lounge access, travel insurance, and concierge — all cross-referenced in the repo's insurance guide ([banking/insurance_products_processes_compliance_guide.md](banking/insurance_products_processes_compliance_guide.md)). **The proposal: add travel eSIM data as a cardholder perk** — each eligible cardholder receives a data allowance of eSIM connectivity per trip, redeemed through the card benefits portal, fulfilled through Airalo Partners' white-label/API channel ✅ (the channel's existence is verified: partners.airalo.com documents white-label stores, API integrations, reseller tools, and voucher programs, fetched; the specific commercial terms are ⚠ contract-level).

Why this perk works for a bank: eSIM data is digital (no logistics), globally consistent (one fulfillment partner, 200+ destinations), low-cost at wholesale (Airalo's model, §5), and highly valued by exactly the traveler segment premium cards court. It also mirrors what the wider industry is already doing — BetaKit documents neobanks and fintechs (Nubank, Revolut, Koho, Wealthsimple) launching eSIM offerings since early 2024 ✅ (BetaKit, fetched; the super-app pattern is documented in [technology/super_app_guide.md](technology/super_app_guide.md)).

### 7.2 The Redemption Flow

```
Cardholder (premium card, travelling)
        │
        ▼
Card benefits portal (Cymbal Bank digital channel)
        │  authenticates cardholder + entitlement check
        ▼
Benefits orchestration layer (bank backend)
        │  selects perk: "2 GB travel eSIM, destination = cardholder's trip"
        │  raises an issuance request (API call) with minimal attributes:
        │  destination country, data size, validity window, redemption token
        ▼
Airalo Partners API / White Label store (branded "Cymbal Travel Data")
        │  issues eSIM package; returns activation QR / deep link
        ▼
Cardholder's phone (eSIM-capable, unlocked — §4.1 device rules)
        │  scans QR or taps install (SGP.22 remote provisioning, §3.2)
        ▼
Data live on arrival — no SIM swap, no roaming bill
```

Architecture notes:

- **Entitlement check happens in the bank, fulfillment in Airalo's layer.** The bank never touches the carrier network; it consumes Airalo's catalog API and white-label surface ✅ (partners.airalo.com, fetched).
- **The white-label option is the brand-safe choice**: the cardholder sees "Cymbal Travel Data," not a third-party marketplace ✅ (Airalo's white-label guide explicitly supports serving the store from the partner's own domain, fetched).
- **Vouchers are the simple fallback**: for one-off campaigns, Airalo's voucher programs let the bank issue redeemable codes without building an API integration ✅ (CVC release, fetched).
- **Redemption analytics**: the bank records redemption, delivery, and (optionally) activation events for program ROI — which brings in the data-protection constraints of §7.4.

### 7.3 Partner Settlement

The settlement mechanics between a bank and an eSIM aggregator are not published by Airalo ⚠ (§11, item 3), so the worked example states the standard pattern the repo's payments guides document (see [banking/payment_rails_guide.md](banking/payment_rails_guide.md) for the rails context) and marks it as design:

- **Wholesale purchase model (recommended):** Cymbal Bank pre-purchases a data allowance pool (e.g., 10,000 × 2GB packages per quarter) at wholesale; each redemption decrements the pool. Settlement is periodic (monthly netting of usage against the pool) rather than per-redemption, which suits a predictable cardholder base.
- **Pay-per-use model (alternative):** issuance is metered and billed monthly against an agreed unit price per package/destination tier.
- **Reconciliation:** redemption records from the bank's orchestration layer are matched against the partner's usage/invoice reports; the eSIM's activation events (profile downloaded, data consumed) are the audit trail — mirroring how the repo's payment guides treat message-level reconciliation.
- **Currency and FX:** Airalo prices in USD at retail ✅ (site convention; e.g., S$ referral amounts appear in local currency, but catalog pricing is USD-based ⚠); a Singapore-incorporated entity with USD settlement adds FX line items to the TCO — flagged for the program's finance team.

### 7.4 Data Protection: PDPA and Minimization

This is where the worked example deliberately cross-references rather than re-derives: the repo's [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) and [banking/insurance_products_processes_compliance_guide.md](banking/insurance_products_processes_compliance_guide.md) already document Singapore's PDPA obligations and the personal-data-minimization discipline a bank must apply to any customer-data flow. Applying that discipline to the eSIM flow yields three concrete rules:

1. **Minimize what leaves the bank.** The issuance API should carry only what fulfillment needs — destination, data size, validity, and a redemption token — not the cardholder's full profile, card number, or travel history ✅ (design rule; the repo's PDPA guides are the authority on the legal framing).
2. **Classify Airalo (AirGSM Pte. Ltd.) as a data processor.** The Singapore legal entity brings the relationship inside the PDPA regime ✅ (AirGSM Pte. Ltd. is Singapore-registered — sgpbusiness.com, fetched; GSMA member — gsma.com, fetched), and the bank's vendor-onboarding must confirm sub-processor flows (which underlying carrier SM-DP+ touches which data) — contract-level ⚠.
3. **Keep the bank's data and the partner's data separable.** Redemption analytics should be aggregable without retaining trip-level location data longer than the program needs; the repo's data-governance and privacy guides (banking series) are the reference for retention limits.

### 7.5 Vendor-Due-Diligence Checklist (Condensed)

For Cymbal Bank's vendor risk team, the public record supports these checks ✅/⚠ (each mapped to the guide's sections):

| Check | Evidence in this guide | Status |
|---|---|---|
| Corporate substance | Delaware incorporation, Singapore entity (AirGSM Pte. Ltd.), Toronto/Istanbul/Singapore offices, GSMA membership | ✅ §1.2, §2.2 |
| Financial durability | $287.3M raised; $1B+ valuation; CVC, Peak XV, Antler, e& Capital, Liberty Global, Orange, Telefónica, Singtel Innov8, Rakuten, KPN, Deutsche Telekom (T.Capital) on the cap table | ✅ §2.3 |
| Product/platform maturity | Catalog API, white-label store, vouchers, Airalo for Business | ✅ §4.5 |
| Track record at bank-grade scale | 20M+ users; 5,000+ partners (secondary source) | ✅/⚠ §2.4 |
| Named marquee B2B2C customers | **None publicly verifiable** — capability described, customers unnamed | ⚠ §11 item 5 |
| Margin/methodology transparency | Wholesale economics, "90% savings" methodology, settlement terms not public | ⚠ §5, §11 item 3 |
| 5G and network-grade SLAs per destination | Not verifiable from public pages | ⚠ §11 item 6 |

The honest conclusion of the worked example: **the capability is real and well-funded, but a production contract needs the contract-level numbers this guide cannot verify from the public footprint** — which is the normal state of affairs for a private unicorn, and precisely why §11 exists.

### 7.6 The Integration Pattern in Technical Detail

The following is target-architecture design (marked as such), built on the verified capability surface of Airalo Partners (white-label storefronts on the partner's own domain, API integrations, voucher programs — partners.airalo.com, fetched):

- **Catalog sync.** The bank's benefits orchestration layer syncs Airalo's product catalog (destination → package → price tier) into an internal entitlement table, so the benefits portal can render "2GB Japan" without calling the partner per page-view. Sync cadence daily; catalog drift handled by the partner's API ✅/⚠ (API existence verified; sync specifics are design).
- **Entitlement token.** On redemption, the bank issues a single-use, short-lived token carrying destination, data size, validity window, and a bank-side redemption ID — no PII (per the minimization rule in §7.4). The token is what crosses the trust boundary.
- **Idempotent issuance.** The issuance call must be idempotent (retry-safe) on the bank-side redemption ID, so a timeout or duplicate cardholder tap cannot double-spend an entitlement — standard integration hygiene from the repo's API-governance conventions ([technology/api_governance_guide.md](technology/api_governance_guide.md)).
- **Delivery.** The partner returns either a QR payload or a deep link into its white-label storefront; the cardholder's phone completes the SGP.22 provisioning (§3.2). For the white-label path the activation happens on the bank's branded domain ✅ (partners.airalo.com whitelabel guide, fetched).
- **Event feedback.** Activation and usage events (profile downloaded, data consumed, expiry) feed the bank's redemption analytics — subject to the retention limits of §7.4. Event granularity and latency are contract-level ⚠.

### 7.7 The Risk Register (Condensed)

| Risk | Severity | Mitigation | Status of evidence |
|---|---|---|---|
| Single-supplier concentration (Airalo as sole fulfillment) | High | Contract in a second supplier (Nomad Enterprise, Ubigi) from day one; keep the integration supplier-agnostic | ⚠ industry-standard practice; no multi-supplier benchmark fetched |
| Wholesale margin opacity (§5.1) | Medium | Benchmark wholesale pool pricing against retail catalog; audit via reconciliation (§7.3) | ⚠ margins not public |
| Data-processing exposure (§7.4) | Medium | PDPA-compliant DPA; sub-processor disclosure; minimization by design | ✅ regime known (repo banking guides); ⚠ Airalo's sub-processor map not public |
| Coverage/quality variance per destination | Medium | Per-destination SLA review; fall back to voucher-based fulfillment for weak destinations | ⚠ no public per-destination SLAs |
| Vendor longevity | Low–Medium | $287.3M raised, $1B+ valuation, CVC-backed (§2.3) — but private-company risk remains | ✅ funding verified |
| FX and settlement friction (§7.3) | Low | USD pool with FX line in TCO; monthly netting | ⚠ contract-level |

### 7.8 Program Economics: An Illustrative Model

A rough order-of-magnitude model (design, ⚠ — every input is an assumption, clearly labeled, because Airalo's partner-tier pricing is not public):

| Input | Assumption | Basis |
|---|---|---|
| Cardholder base (premium cards, Singapore branch) | e.g., 50,000 eligible cardholders | bank-internal assumption |
| Redemption rate per trip | 15–25% of eligible cardholders per quarter | benefits-program benchmark ⚠ |
| Wholesale cost per 2GB package | single-digit USD ⚠ (retail band per §5.4) | derived from the US$4.50+ retail band |
| Per-cardholder annual cost | low tens of USD ⚠ | arithmetic on the above |
| Program cost vs. lounge-access budget | materially lower per redemption ⚠ | internal comparison — eSIM is one of the cheapest premium perks to fulfill |

The model's purpose is structural, not predictive: it shows that **an eSIM perk is a volume-elastic, low-marginal-cost benefit** — the marginal cardholder costs almost nothing to serve once the wholesale pool and the integration exist — which is precisely the property that makes it attractive to a benefits committee and easy to under-forecast ⚠ (analysis, not verified figures).

---

## 8. The Banking–Telco Intersection

### 8.1 Why Banks Are Getting Into eSIM

The banking industry is quietly becoming a distribution channel for connectivity ✅ (BetaKit, fetched): Brazilian neobank Nubank, UK fintech Revolut, and Canadian startups Koho and Wealthsimple all launched eSIM offerings in 2024 — a pattern the repo's super-app guide already captures for Revolut ([technology/super_app_guide.md](technology/super_app_guide.md), which lists eSIM among Revolut's non-financial features). The strategic logic:

- **eSIM is a low-cost, high-frequency engagement feature** — it puts the bank's app in the traveler's hands abroad, where the card is also being used.
- **It rides the super-app arc** — connectivity is the latest non-financial service folded into banking apps, exactly as the super-app guide documents for travel, insurance, and crypto ✅ (super_app_guide.md cross-reference).
- **The supply side is aggregator-ready** — Airalo Partners' white-label/API channel (and Nomad's enterprise line, §6.2) exist precisely to let non-telcos launch connectivity features without carrier negotiations ✅ (partners.airalo.com; Wikipedia).

### 8.2 The Precedent Set: Telco-Infrastructure Players and Banks

The intersection is not new — TechCrunch's Series B reporting notes that Truphone, a now-sold eSIM pioneer, had "services to banks" among its marquee deals before its Russian-owner sanctions saga and £1 sale ✅ (TechCrunch, fetched). The cautionary tale is structural, not categorical: eSIM provisioning is infrastructure, and infrastructure businesses live or die on capital discipline and trust — two things a bank's due diligence is built to probe (§7.5).

### 8.3 What This Means for the Series

For the repository, this guide closes a loop: the super-app guide documents the *pattern* (eSIM as a bolted-on feature), the banking guides document the *regulatory envelope* (MAS, PDPA), and this guide documents the *supply side* (who Airalo is, how the category works, what the contract-level unknowns are). A bank reading all three has the full picture: the feature, the rules, and the vendor.

The observed industry motion supports the thesis: BetaKit notes eSIMs "have been gaining popularity, especially among neobanks supplementing their core services," listing Nubank, Revolut, Koho, and Wealthsimple as 2024 entrants ✅ (BetaKit, fetched), and the super-app guide's landscape table independently lists eSIM among Revolut's non-financial features ✅ ([technology/super_app_guide.md](technology/super_app_guide.md)). The pattern is consistent enough to treat as a trend rather than an experiment: **banks are becoming connectivity distributors, and aggregators like Airalo are the wholesale layer that makes it operationally trivial** ✅/⚠ (trend verified; its durability is a judgment call).

### 8.4 The Singapore Regulatory Angle

The worked example lives in Cymbal Bank's Singapore branch, and Airalo's operational base is Singapore — so the regulatory overlay is Singapore's, and this guide deliberately does not re-derive it:

- **PDPA / data minimization** — the redemption flow's data-sharing rules are governed by Singapore's Personal Data Protection Act; the discipline of §7.4 (minimize what crosses the trust boundary, classify the vendor as a data processor, separate bank data from partner data) is the applied form of what the repo's [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) and [banking/insurance_products_processes_compliance_guide.md](banking/insurance_products_processes_compliance_guide.md) document in full.
- **MAS outsourcing discipline** — if the eSIM-benefits program is operated through a third-party platform as a material outsourced service, the bank's Notice 658 / Outsourcing Guidelines obligations (cross-referenced from the MAS guide) would shape the contract: service-level commitments, audit rights, data-residency and business-continuity clauses, and sub-processor oversight over Airalo's carrier-side provisioning chain ⚠ (applicability is scope-dependent — flagged, not asserted; see the MAS guide for the instrument details).
- **The jurisdiction mirror** — the deal structure (Delaware-incorporated group, Singapore-incorporated operating entity, global remote workforce, §2.2) means the bank's contracting counterparty, the data processor, and the service operator could be three different legal persons; the contract must name them and allocate obligations explicitly ✅/⚠ (corporate structure verified; contractual mapping is design).

---

## 9. The Summary: The Connection Without Borders

Airalo is a marketplace company in the purest sense: it owns no network, sells no hardware, and manufactures nothing — it aggregates wholesale data capacity from hundreds of carriers and resells it as small, cheap, destination-specific eSIM packages ✅ (TechCrunch, fetched). Founded in 2019 by Bahadir Özdemir and Abraham Burak with a $1.9M Antler/Sequoia seed, it raised $5.4M (Series A, Rakuten Ventures, 2021), $60M (Series B, e& Capital, 2023), and $220M (Series C, CVC, 2025) — $287.3M in total, at a $1B+ valuation that made it the category's first unicorn ✅ (§2). Its 20M+ users (30M+ per its current site) and 200+ destination footprint make it the reference point for the travel-eSIM market ✅/⚠ (§2.4).

The category itself is now standardized and boring — which is the compliment infrastructure deserves: GSMA SGP.22 for consumer remote provisioning, SGP.02 for M2M, SGP.32 for IoT, with SM-DP+ platforms doing the profile delivery ✅ (§3). The competitive set — Holafly, Ubigi, Nomad, Saily, and the carriers themselves — is well-defined, and Airalo's lead is measurable but not unassailable ✅/⚠ (§6).

For a bank like Cymbal Bank, the worked example shows a credible, low-friction path from "cardholder flying to Tokyo" to "2GB of branded eSIM data live on arrival" — provided the contract-level economics, settlement terms, and data-processing map are nailed down in diligence ✅/⚠ (§7). The eSIM is not a flashy technology; it is a quiet layer that turned global connectivity into a purchasable, embeddable digital good — and that is exactly the kind of layer banks, super-apps, and travel platforms will keep building on.

Three takeaways survive the audit: **the company is real and well-capitalized** (every funding round verified at primary or near-primary sources, §2) ✅; **the category is standardized** (GSMA SGP.22/SGP.32 and the SM-DP+ ecosystem are documented and stable, §3) ✅; and **the commercial unknowns are concentrated in the contract layer** (margins, partner pricing, settlement, named customers — all ⚠ in §11) — which is precisely where a bank's own procurement process, not a research guide, must close the gap ⚠.

---

## 10. Verification and Claims-Status

The series convention: ✅ = verified from a source fetched in this pass; ⚠ = could not be confirmed from primary sources (company claim, secondary source, or industry knowledge); ❌ = refuted.

| Claim | Status | Source (fetched this pass) |
|---|---|---|
| Founded 2019 by Özdemir and Burak (Akcaylier co-founder, left 2020) | ✅ | en.wikipedia.org/wiki/Airalo |
| AirGSM Pte. Ltd. incorporated in Singapore 25 Oct 2018 | ✅ | sgpbusiness.com |
| Delaware incorporation; Singapore operational base; Toronto/Istanbul/Singapore offices | ✅ | Wikipedia; TechCrunch; PRNewswire; Airalo Series A PDF |
| "World's first eSIM store" positioning | ✅ (as company claim) | airalo.com/about-us/about-airalo; Series A PDF; gsma.com member page |
| Mission: "global data connectivity for all people" | ✅ (as company claim) | airalo.com/about-us/about-airalo |
| Seed: $1.9M, Oct 2019, Antler + Sequoia Surge | ✅ | Airalo Series A PDF; Wikipedia |
| Series A: $5.4M, Oct 2021, Rakuten Ventures lead | ✅ | Airalo Series A PDF; Tech in Asia; PhocusWire |
| Hypothesis "Series A ~$4.3M in 2020" | ❌ refuted — no such round in any fetched source | §2.3 |
| Series B: $60M, Jul/Aug 2023, e& Capital lead; total $67.3M; ~$280M post-money | ✅ | PRNewswire; TechCrunch |
| Series B investor list (Antler Elevate, Liberty Global, Orange, T.Capital, Rakuten Capital, Singtel Innov8, Telefónica Ventures, Surge/Peak XV, KPN, I2BF) | ✅ | PRNewswire |
| Series C: $220M, 10 Jul 2025, CVC/CVC Asia VI ($185M), Peak XV + Antler Elevate; >$1B valuation; "first eSIM unicorn" | ✅ | cvc.com release; BetaKit; Reuters via Wikipedia |
| Total funding $287.3M | ✅ | arithmetic on verified rounds; consistent with trackers |
| Bell Ventures among backers | ⚠ | BetaKit only; not in Airalo/Wikipedia lists |
| 5.1M customers (Jul 2023); 689 plan combinations | ✅ | TechCrunch |
| 10M users milestone (2024) | ⚠ | Tech in Asia via Wikipedia reference list; date unconfirmed |
| 20M+ travelers (Jul 2025); 30M+ users (2026 site) | ✅ / ⚠ (company-reported) | CVC release; airalo.com about page |
| 200+ countries/regions coverage | ✅ (company claim; consistent across sources) | airalo.com; PRNewswire; CVC |
| 5,000+ Airalo Partners (Aug 2025) | ⚠ | Wikipedia citing Tech in Asia |
| GSMA standards: SGP.22 (consumer), SGP.02 (M2M), SGP.32 (IoT, May 2023) | ✅ | Wikipedia eSIM; gsma.com |
| SM-DP+ = Subscription Manager Data Preparation (Plus) | ✅ | GSMA SGP.22 spec; vendor docs |
| eUICC/EID/ICCID mechanics; ISO/IEC 7816; MFF2 | ✅ | Wikipedia eSIM |
| eSIM device history (Gear S2 2016 → iPhone 17 era 2025; US iPhone 14 eSIM-only 2022) | ✅ | Wikipedia eSIM |
| eSIM re-programmability; carrier-lock constraints; secure-channel provisioning | ✅ | Wikipedia eSIM; GSMA |
| Investor split: strategic telco capital vs financial/growth capital | ✅ | PRNewswire; CVC release; Series A PDF |
| Ubigi 5G and device-agnostic connectivity (SD-WAN routers, hotspots) | ✅ | transatel.com |
| Airalo supply model (wholesale + roaming resale + pricing tech) | ✅ | TechCrunch |
| App flow (choose → package → install); top-up; device rules | ✅ | airalo.com/how-airalo-works |
| Airalo Partners: white label, API, vouchers, Airalo for Business; "90% roaming cost savings" | ✅ / ⚠ (claim) | partners.airalo.com; CVC release |
| Travel-eSIM market size (US$1B per CVC; US$8.7B by 2030 per ResearchAndMarkets; US$365M→$734M per GII) | ⚠ divergent vendor estimates | CVC release; BusinessWire; GII |
| Holafly: 2017, Murcia, Máiquez/Hu | ✅ | Wikipedia |
| Ubigi: Transatel 2000, Paris, NTT since 2019, 200+ destinations | ✅ | Wikipedia; transatel.com |
| Nomad: 2020, LotusFlare, Santa Clara, data-only | ✅ | Wikipedia |
| Saily: 2024, Nord Security (NOT Proton) | ✅ / ❌ (task hypothesis refuted) | nordsecurity.com |
| Neobank eSIM wave (Nubank, Revolut, Koho, Wealthsimple) | ✅ | BetaKit |
| Named Airalo airline/loyalty marquee deals | ⚠ none verifiable | §11 item 5 |
| Airalo 5G availability per destination | ⚠ | §11 item 6 |

---

## 11. What Could Not Be Verified

The following claims could not be confirmed from primary sources fetched during this pass; they are flagged ⚠ at their point of use and listed here per the series convention:

1. **Airalo's wholesale/retail margin structure and unit economics** — TechCrunch documents the *mechanics* (wholesale capacity, roaming resale) but no margin figures are public; the "up to 90% roaming-cost reduction" claim (CVC release) has no published methodology.
2. **Bell Ventures as an Airalo backer** — asserted by BetaKit; absent from Airalo's own Series B release and Wikipedia's investor lists.
3. **Settlement and contract terms for the Airalo Partners channel** (pricing tiers, netting, FX, SLAs) — contract-level; the worked example's settlement design (§7.3) is explicitly a proposal, not a documented Airalo offering.
4. **Named marquee B2B2C customers** — Airalo's public footprint describes white-label/API/voucher *capabilities* but names no flagship airline, bank, or loyalty program deal.
5. **The "5,000+ partners" figure (August 2025)** — rests on Wikipedia's citation of Tech in Asia; no primary Airalo source fetched in this pass states it.
6. **Airalo 5G availability per destination** — the 5G-capable destinations claim could not be verified from a fetched Airalo page (Ubigi's 5G claim was verified; Airalo's was not).
7. **The current 30M+ user figure** — appears on Airalo's live about page ("fast facts") but is a company claim without independent confirmation; 20M+ (July 2025) is the last press-corroborated count.
8. **Holafly's funding history and current valuation** — only paywalled trackers (Tracxn/PitchBook/Crunchbase) surfaced; no public announcement was fetched.
9. **Nomad's funding history** — same situation; the Wikipedia article documents the product and parent (LotusFlare) but no round details were verified.
10. **Travel-eSIM market-size figures** — CVC's "US$1 billion" claim, ResearchAndMarkets' "US$8.7B by 2030," and GII's "US$365M (2024) → US$734M (2030)" disagree by an order of magnitude; none is independently auditable here.
11. **Saily's exact coverage count** — review sites variously say 150+, 190+, or 200+ countries; Nord Security's own materials fetched in this pass did not state a single authoritative number.
12. **"World's first eSIM store" precedence** — the phrase is Airalo's own claim (verified as *their claim*); whether Airalo was literally first among all eSIM stores (vs. e.g. earlier players like GigSky or Truphone's consumer products) is not adjudicated here.
13. **App-store ratings and "#1 travel app" status** — 4.7★/4.6★ and "#1 in multiple countries" come from Airalo's August 2023 release; not re-verified against live store listings in this pass.
14. **The pandemic "stalled growth" characterization and the 20% MoM revenue-growth / ~1M monthly-downloads figures** — reported by TechCrunch from company statements; not independently audited.
15. **The BetaKit price example** (CA$12 two-week 2GB UK plan vs CA$16/day Bell roaming) — a single journalist's example at a point in time; Airalo's current price list was not scraped in this pass.
16. **The 10-million-users milestone** — referenced by Wikipedia via a Tech in Asia article that was not fetched in this pass; the milestone's exact date and figure could not be independently confirmed (§2.4).
17. **GigSky and Yesim** — cross-linked by Wikipedia's Airalo article as further travel-eSIM players; neither was profiled or source-verified in this pass (§6.2).
18. **MAS outsourcing applicability to the §8.4 program** — whether the eSIM-benefits operation qualifies as a material outsourced service under Notice 658 is scope-dependent regulatory analysis, deferred to the MAS guide; not asserted here.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **eSIM** | Embedded subscriber identity module — a SIM implemented as software on an embedded chip rather than a removable card (✅ §3.1) |
| **eUICC** | Embedded Universal Integrated Circuit Card — the soldered secure element that hosts eSIM profiles; 6mm × 5mm MFF2 form factor, ISO/IEC 7816 electrical interface (✅ §3.1) |
| **EID** | eUICC Identifier — the permanent factory-programmed identity of the chip, used by provisioning services (✅ §3.1) |
| **ICCID / IMSI** | The SIM identifiers carried by an installed profile (✅ §3.1) |
| **RSP** | Remote SIM Provisioning — the GSMA-defined process of downloading a profile to a device over the air (✅ §3.2) |
| **SGP.22** | GSMA consumer eSIM remote-provisioning specification (smartphones, tablets, wearables) (✅ §3.2) |
| **SGP.02** | GSMA M2M eSIM specification — the older machine-to-machine architecture (✅ §3.2) |
| **SGP.32** | GSMA eSIM IoT specification, published May 2023 — server-driven profile management for constrained IoT devices (✅ §3.2) |
| **SM-DP+** | Subscription Manager – Data Preparation (Plus) — the server platform that prepares, stores, and delivers eSIM profiles (✅ §3.2) |
| **LPA** | Local Profile Assistant — the device-side component that manages profiles and talks to the SM-DP+ (✅ §3.2, referenced in GSMA spec) |
| **MNO / MVNO / MVNE** | Mobile network operator / mobile virtual network operator / mobile virtual network enabler — the carrier layer Airalo buys from (✅ §4.4, TechCrunch) |
| **Wholesale capacity** | Bulk data purchased directly from carriers at wholesale rates and repackaged into retail bundles (✅ §4.4) |
| **Local / regional / global eSIM** | Airalo's catalog tiers: single-country, multi-country region, worldwide (✅ §4.2) |
| **Top-up** | Replenishing data on an existing eSIM rather than buying a new profile (✅ §4.1) |
| **Airmoney** | Airalo's loyalty/referral credit system (✅ §4.1) |
| **Airalo Partners** | The B2B/B2B2C arm: white-label stores, API integrations, reseller tools, vouchers, Airalo for Business (✅ §4.5) |
| **White Label** | A partner-branded eSIM storefront served from the partner's own domain (✅ §4.5) |
| **Airalo for Business** | The enterprise platform for assigning and budgeting employee eSIMs (✅ §4.5) |
| **QR-code activation** | Installing an eSIM by scanning a QR code carrying the SM-DP+ address and activation code (✅ §4.3) |
| **Instant/in-app activation** | One-tap profile installation from inside the provider's app (✅ §4.3) |
| **Carrier roaming** | Using one's home-network plan abroad at premium daily rates — the incumbent alternative eSIMs undercut (✅ §3.4) |
| **First eSIM unicorn** | Airalo's billing after the $220M CVC-led Series C valued it above $1B (✅ §2.3) |
| **GigSky / Yesim** | Additional travel-eSIM players cross-linked by Wikipedia's Airalo article; not profiled in this guide (✅ existence; ⚠ details — §6.2) |
| **Notice 658** | MAS Notice on outsourcing — the Singapore instrument that would shape the §8.4 program contract if the eSIM-benefits operation is a material outsourced service (✅ via [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md)) |
| **Cymbal Bank** | The repository's bank persona — the author's firm, a European wholesale banking group with a Singapore branch (✅ repo convention, §7) |

---

## 13. References

All URLs below were fetched (or, where marked, referenced from a fetched page) during this pass. Reuters and Forbes items are marked as referenced via Wikipedia because their pages blocked automated retrieval; both are cited in Wikipedia's Airalo article for the Series C facts.

1. Airalo — *About Airalo* ("world's first eSIM store"; mission; fast facts 2019 / 30M+ users / 200+ countries; impact programs). https://www.airalo.com/about-us/about-airalo
2. Airalo — *How Airalo works* (three-step flow; install times; device requirements; top-up). https://www.airalo.com/how-airalo-works
3. Airalo — *Press Release: Airalo Raises $60M in Series B Financing* (full investor list; team; app ratings; Airmoney S$4 referral). https://www.airalo.com/blog/press-release:-airalo-raises-$60m-in-series-b-financing
4. PRNewswire / Airalo — *Airalo, World's Largest eSIM Marketplace, Raises $60 Million in Series B Financing* (1 Aug 2023; LEWES, Del. dateline; investor list; $67.3M total). https://www.prnewswire.com/news-releases/airalo-worlds-largest-esim-marketplace-raises-60-million-in-series-b-financing-301890455.html
5. Airalo — *Airalo, World's First eSIM Store, raises $5.4 million in Series A Funding* (press-release PDF: Rakuten Ventures lead; seed $1.9M Oct 2019; offices Toronto/Istanbul/Singapore; mission statement). https://www.airalo.com/assets/press-releases/Airalo_Series_A_Funding_Announcement_Main_Press_Release.pdf
6. TechCrunch (Ingrid Lunden) — *Airalo locks in $60M to expand its eSIM-based global roaming 'marketplace'* (31 Jul 2023; $280M post-money; 5.1M customers; 689 combos; wholesale model; Truphone; "previously raised just $7.3M"). https://techcrunch.com/2023/07/31/airalo-locks-in-60m-to-expand-its-esim-based-global-roaming-marketplace/
7. CVC — *Airalo becomes the first eSIM Unicorn with an investment round of $220m* (10 Jul 2025; CVC Asia Fund VI $185M; Peak XV + Antler Elevate; >$1B valuation; 20M+ travelers; Airalo for Business; white label; travel-eSIM market "already worth US$1 billion"). https://www.cvc.com/media/news/2025/airalo-becomes-the-first-esim-unicorn-with-an-investment-round-of-220m/
8. BetaKit — *Canadian co-founded eSIM provider Airalo hits unicorn status with $220-million USD Series C round* (founder background; Bell Ventures mention; neobank eSIM wave; price example). https://betakit.com/canadian-co-founded-esim-provider-airalo-hits-unicorn-status-with-220-million-usd-series-c-round/
9. Wikipedia — *Airalo* (founding; founders incl. Akcaylier; Delaware/Singapore; seed/Series A/Series B/Series C; 20M users July 2025; Airalo Partners 5,000+; AirGSM Pte. Ltd.). https://en.wikipedia.org/wiki/Airalo
10. Reuters — *CVC invests in Singaporean tech company Airalo at $1 billion-plus valuation* (10 Jul 2025; referenced via Wikipedia — Reuters page not directly fetchable in this pass). https://www.reuters.com/technology/cvc-invests-singaporean-tech-company-airalo-1-billion-plus-valuation-2025-07-10/
11. Forbes (Yessar Rosendar) — *Singapore's Airalo Becomes Unicorn After Raising $220 Million In CVC-Led Round* (11 Jul 2025; referenced via Wikipedia — page not directly fetchable in this pass). https://www.forbes.com/sites/yessarrosendar/2025/07/11/singapores-airalo-becomes-unicorn-after-raising-220-million-in-cvc-led-round/
12. Wikipedia — *eSIM* (definition; eUICC/EID/ICCID; SGP.22/SGP.02/SGP.32; device history: Gear S2, Watch Series 3, Pixel 2, iPhone XS, US iPhone 14 eSIM-only, eSIM-only iPads and iPhone 17 era). https://en.wikipedia.org/wiki/ESIM
13. GSMA — *eSIM* (consumer/M2M/IoT architectures; secure-element download; ecosystem framing). https://www.gsma.com/solutions-and-impact/technologies/esim/
14. GSMA — *Remote SIM Provisioning: How it Works* (the RSP explainer). https://www.gsma.com/solutions-and-impact/technologies/esim/gsma_resources/remote-sim-provisioning-how-it-works/
15. GSMA — *SGP.22 RSP Technical Specification v3.1* (names "Subscription Manager Data Preparation (SM-DP+)" in the interoperability framing). https://www.gsma.com/solutions-and-impact/technologies/esim/wp-content/uploads/2023/12/SGP.22-v3.1.docx
16. GSMA — *Airalo (AirGSM Pte. Ltd.) — Membership* ("the world's first eSIM store for travelers"; GSMA member). https://www.gsma.com/get-involved/gsma-membership/gsma_orgs/airgsm-pte-ltd/
17. SGP Business — *AIRGSM PTE. LTD. (201836421Z)* (incorporated 25 Oct 2018; Marina Square, Singapore). https://www.sgpbusiness.com/company/Airgsm-Pte-Ltd
18. Airalo Partners — *Partner with the world's leading eSIM provider* (resellers, API partners, affiliates; white-label platform). https://partners.airalo.com/
19. Airalo Partners — *Whitelabel eSIM platform integration guide* (partner-domain storefront). https://developers.partners.airalo.com/whitelabel-esim-platform-integration-guide-2188917m0
20. Wikipedia — *Holafly* (founded 2017, Spain, Máiquez and Hu; Lanzadera 2019). https://en.wikipedia.org/wiki/Holafly
21. Wikipedia — *Transatel* (founded 2000, Paris; Bonifay/Salomon; NTT Group subsidiary since 2019). https://en.wikipedia.org/wiki/Transatel
22. Transatel — *Ubigi enabled by Transatel* (launched 2017; global LTE/5G; 200+ destinations). https://www.transatel.com/ubigi/
23. Wikipedia — *Nomad (eSIM)* (2020; LotusFlare; Santa Clara; data-only; QR/in-app activation; Nomad eSIM Enterprise 2024; Gaza usage). https://en.wikipedia.org/wiki/Nomad_(eSIM)
24. Nord Security — *Stress-free travel with Nord Security's new eSIM — Saily* (Saily is a Nord Security product, launched 2024). https://nordsecurity.com/blog/stress-free-travel-esim-saily
25. Nord Security press area — *Browse the world with Saily — an eSIM solution designed especially for travelers*. https://nordsecurity.com/press-area/browse-the-world-with-saily-an-esim-solution-designed-especially-for-travelers
26. Tech in Asia — *Antler, Sequoia join Airalo's $5.4m round* (Oct 2021; Rakuten Ventures lead). https://www.techinasia.com/rakuten-antler-sequoia-singtel-innov8-pour-54m-traveltech-firm
27. PhocusWire — *Airalo raises $5.4M for digital SIM marketplace*. https://www.phocuswire.com/esim-marketplace-airalo-raises-5-4-million-series-a
28. BusinessWire / ResearchAndMarkets — *Travel SIMs & eSIMs Market Forecast Report 2025-2030* ("global market spending to reach $8.7 billion by 2030"). https://www.businesswire.com/news/home/20251114715758/en/
29. GII Research — *Global Travel eSIM Market Insights, Forecast to 2030* ($364.88M 2024 → $734.16M 2030, 12.36% CAGR). https://www.giiresearch.com/report/qyr1607565-global-travel-esim-market-insights-forecast.html
30. Repo cross-references (verified in earlier guides of this series): [technology/ibm_cloud_guide.md](technology/ibm_cloud_guide.md) (style exemplar; verification conventions), [technology/super_app_guide.md](technology/super_app_guide.md) (eSIM as a super-app feature — Revolut), [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) and [banking/insurance_products_processes_compliance_guide.md](banking/insurance_products_processes_compliance_guide.md) (PDPA / data-minimization), [banking/banks_in_singapore_guide.md](banking/banks_in_singapore_guide.md) (Cymbal Bank persona; market context).

---

*End of guide. Series context: the connectivity deep-dive of the research repository — read it beside the super-app guide (which documents eSIM as a feature) and the banking guides (which document the regulatory envelope), and treat the ⚠ items in §11 as the honest boundary of what a bounded research pass can confirm. Airalo is not a network; it is the marketplace that made global data a purchasable, embeddable digital good — the connection without borders.*
