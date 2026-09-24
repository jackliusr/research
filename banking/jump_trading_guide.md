# Jump Trading: A Comprehensive Guide

**The Identity, History, Business, Arms, Technology, Latency Posture, Regulatory Record, Capital Silence, Singapore Footprint and Bank Interface of the Chicago Proprietary Trading Firm Jump Trading LLC — with the December 2024 SEC Settlement of Its Digital-Asset Affiliate Tai Mo Shan Limited, the 2018 NYSE Chicago Net-Capital Matter, the Reported 2024 CFTC Probe, the Terra Litigation Record, and a Cymbal Bank Counterparty Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Institutional Investment & Capital Markets — the proprietary trading firm as a distinct firm type, the no-clients business model, the latency arms race and co-location, the digital-asset and venture arms, the 2024 SEC settlement and the 2018 NYSE Chicago order, the reported CFTC probe, the Terra litigation record, the Asia/Singapore footprint, and the Cymbal Bank institutional-counterparty lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder):** [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) (its **§4.4 owns the Jump Trading entry** — founded / products / Singapore presence — and its §5.2 owns QCP Capital; cross-ref §11 and §12 here — do not re-derive either) · [Hudson River Trading](hudson_river_trading_guide.md) (the prop-firm archetype in this repository: **§5** technology, **§6** talent, **§7/§8** regulatory and market structure, **§9** Singapore/Asia, **§10** the Cymbal Bank worked example, **§11** the claims audit — cross-referenced throughout, not re-derived) · [Citadel LLC](citadel_llc_guide.md) (the archetype firm guide; **its §7 owns the technology-of-the-firm-type material**) · [ExodusPoint](exoduspoint_guide.md) (the most recent firm guide; this guide adopts its identity discipline, its capital discipline and its convention for recording a settled claim — **its §2 and §6 are the pattern for §2 and §7 here**) · [Tokenized Assets](tokenized_assets_guide.md) (its line 239 records Jump Trading's participation in Partior's Series B — cross-ref §7.6) · [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) (the fund-management contrast — cross-ref §12.3) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) (the Cymbal Bank persona and worked-example conventions)
> **Companion guides (technology/, prefix `../technology/`):** [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md), [Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md) and [Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md) (the latency discipline itself — cross-ref §8) · [Smart Contracts](../technology/smart_contracts_guide.md) (**its §9 owns the Wormhole bridge exploit record** — cross-ref §6.7, do not re-derive) · [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) (the quant-research and engineering skill stack — cross-ref §9) · [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md) (the platform architecture and market-access structure — cross-ref §13)

---

**How to use this guide:** Section 1 is the overview — the one-line thesis, the paragraph on the firm, the vocabulary decoder, and the boundary declaration naming the guides this one defers to. Section 2 is the identity and the entities — the legal entities from the firm's own disclosures page, the founding record, the registered footprint, and a blunt statement of what the entity map does **not** establish. Section 3 is the history. Section 4 is the business — the firm's own asset-class list and strategy spectrum, and the structural point that a proprietary firm has no clients and discloses no revenue composition. Section 5 is the arms — Jump Crypto and Jump Capital, each with its current status dated, plus the Jump Capital founder reconciliation. Section 6 is the highest-risk section: **the incidents and the regulatory matters, as a dated record with authority + instrument + date + outcome, and nothing else.** Section 7 is the capital and the revenue, where the finding is absence. Section 8 is the technology and the latency posture, built on the firm's own unusually available statements. Section 9 is talent and culture. Section 10 is the regulatory and market-structure context. Section 11 is the Asia and Singapore angle, including the two errors in the dispatch brief this guide corrects. Section 12 is the peer group and positioning. Section 13 is the bank interface at mechanism level. Section 14 is the Cymbal Bank worked example — fictional, illustrative, and the **only** bank persona in this guide. Section 15 is the anti-patterns and the claims audit. Section 16 collects What Could Not Be Verified, the glossary, the cross-references and the closing summary. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`. **Integrity convention:** ✅ = verified this pass against a primary source or a named, dated source; ⚠ = flagged — reported, single-sourced, contested, or not re-verified live; ❌ = refuted or rejected. Where the firm discloses nothing, this guide says so rather than inferring, and every gap named in the brief is written into §16.1. Nothing in this guide was invented: no case number, penalty, charge, settlement term, founder, entity, capital figure, technology attribution or date appears here that is not in the source named beside it.

---

## Table of Contents

1. [The Overview](#1-the-overview) — the one-line thesis ("a proprietary trading firm has no clients, so nothing stands between its engineering and its P&L"), what the firm is and is not, the key-facts table, the vocabulary decoder, and the boundary declaration naming the guides this one defers to (§1.1–§1.5)
2. [The Identity and the Entities](#2-the-identity-and-the-entities) — the entity table from the firm's own disclosures page, the founding and the founders, the registered footprint at FINRA BrokerCheck, and a blunt statement of what the entity map does **not** establish (§2.1–§2.4)
3. [The History](#3-the-history) — 1999 and the open-outcry context, the 2 November 2010 CFTC meeting, the electronic transition and the 2013 tower and 2014 subpoena, the exchange footprint from 2000, the arms in 2012 and 2021, and 2024 to the present (§3.1–§3.6)
4. [The Business](#4-the-business) — the structural point that a proprietary firm has no clients, the firm's own asset-class list and strategy spectrum, and what a business description cannot contain (§4.1–§4.4)
5. [The Arms and the Affiliates](#5-the-arms-and-the-affiliates) — Jump Crypto and Jump Capital with current status dated, the Jump Capital founder reconciliation, and why the trading firm must be distinguished from its arms (§5.1–§5.4)
6. [The Incidents and the Regulatory Matters](#6-the-incidents-and-the-regulatory-matters) — the dated record: the SEC order against Tai Mo Shan Limited and its Fair Fund, the NYSE Chicago order against Jump Trading, LLC, the CFTC meeting and reported probe, the civil matters as allegations, Wormhole, and what remains unresolved (§6.1–§6.10)
7. [The Capital and the Revenue](#7-the-capital-and-the-revenue) — no group financials, what the disclosures page does publish, the Pillar III obligation, the absence as the finding, the QCP mis-attribution reconciled, and the Partior datum (§7.1–§7.6)
8. [The Technology and the Latency Posture](#8-the-technology-and-the-latency-posture) — the class of infrastructure cross-referenced, the firm's own statements verbatim and attributed, the CTO's words, dated 2026 firm statements, and what is not documented (§8.1–§8.5)
9. [Talent and Culture](#9-talent-and-culture) — the resourcing model cross-referenced, what the firm says publicly, its fellowship and research programmes, its ML/AI positioning, and the people record (§9.1–§9.5)
10. [The Regulatory and Market-Structure Context](#10-the-regulatory-and-market-structure-context) — where a proprietary firm sits in the debate, the designation and registration questions, the Principal Traders Group, spoofing in 2010, and conduct over statements (§10.1–§10.5)
11. [The Asia and Singapore Angle](#11-the-asia-and-singapore-angle) — what the sibling guide's §4.4 already owns, what this guide adds, the two dispatch-brief errors reconciled, and what is not established (§11.1–§11.4)
12. [The Peer Group and Positioning](#12-the-peer-group-and-positioning) — the peer set with its dated repo sources, the contrast with a bank's markets business and with a multi-manager hedge fund, and this firm's relative position (§12.1–§12.4)
13. [The Bank Interface](#13-the-bank-interface) — the firm as counterparty, as liquidity provider and as a clearing client; the artefacts a bank can actually obtain; the counterparty-credit questions; and what this guide will not name (§13.1–§13.6)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example) — the fictional Lattice Trading scenario: entity identification, the credit assessment without disclosure, the market-structure risk, and the recommendation with its limit and condition (§14.1–§14.5)
15. [The Anti-Patterns and the Claims Audit](#15-the-anti-patterns-and-the-claims-audit) — seven anti-patterns with symptom, cause and guardrail; the verified, flagged and rejected claims tables with sources and dates; and the reconciled corrections (§15.1–§15.5)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary) — every unverified item, the glossary, the cross-references and sources, and the closing summary (§16.1–§16.4)

---

## 1. The Overview

### 1.1 The Short Answer — The Thesis in One Line

**A proprietary trading firm has no clients, so nothing stands between its engineering and its P&L — which makes it a technology firm that happens to trade, and makes every latency problem a physics problem.**

That sentence is the whole guide in miniature, and it is worth unpacking because it explains everything else about a firm of this type. A bank's markets business has a client franchise between its engineers and its revenue: the flow, the relationship, the mandate. A hedge fund has investors in that position. An exchange has members. A proprietary trading firm has **neither clients nor investors** — it trades its own capital, for its own account, and the only thing that converts a research idea into money is the code and the infrastructure that carry it to the order book ✅ (the NYSE Chicago order states the position in the regulator's own words: "The Firm trades solely in a proprietary capacity"). Two consequences follow immediately. First, **technology spend at a firm of this kind is not a support function; it is the P&L**. Second, and less obviously, **the residual constraint on every one of those engineering decisions is physical** — the speed of light in fibre, the propagation delay across an ocean, the nanoseconds added by a switch hop — which is why firms of this class buy microwave towers, build custom silicon, and put compute inside the exchange's own building.

**Jump Trading LLC** is an American proprietary trading firm founded in **1999**, headquartered in **Chicago, Illinois**, and founded by two former **Chicago Mercantile Exchange pit traders, Bill DiSomma and Paul Gurinas** ✅ (Wikipedia, citing Forbes 2006 and Bloomberg; the firm's own site states "25 Years trading, building, evolving", consistent with a 1999 founding). It trades **equities, options, credit, commodities, digital assets and futures**, with strategies spanning — in its own words — "high-frequency to stat arb to discretionary macro" ✅ (jumptrading.com/trading, accessed 24 September 2026). It is a **registered broker-dealer** in the United States through **Jump Trading, LLC**, which has held a **NYSE Chicago participation since 22 December 2000** ✅ (NYSE Chicago order). It runs two publicly named arms: **Jump Crypto**, launched as a brand on **14 September 2021**, and **Jump Capital**, a venture arm the firm says began in **2012** ✅/⚠ (Businesswire, 14 September 2021; Wikipedia for the 2012 date — see §5.3). Its own office list names **thirteen cities across the United States, Europe and Asia-Pacific, including Singapore** ✅ (jumptrading.com, accessed 24 September 2026).

It is also, in the two years before this guide was written, the subject of **the largest single regulatory settlement in the proprietary-trading sector's recent history** — a **US$123,095,287** SEC order against its digital-asset affiliate **Tai Mo Shan Limited**, entered **20 December 2024** ✅ (SEC Press Release 2024-212; SEC Admin. Proc. File No. 3-22382, Release No. 33-11349) — and of a **reported, unresolved CFTC probe of its crypto activities**, reported **20 June 2024** ⚠ (Fortune, carried by Reuters), for which **no enforcement action has been identified** (§6.5). Both belong in a bank's file on the firm, and both are treated in §6 as what they are: a settled order on the one hand, a dated press report on the other.

### 1.2 What Jump Trading Is, and What It Is Not

Because this firm is routinely described in second-hand write-ups in ways that fold its trading business, its crypto arm and its venture arm into one undifferentiated thing, the distinctions need stating first:

- **It is a proprietary trading firm.** It trades its own capital as principal. It has no clients, publishes no client assets, runs no external fund and reports to no investor base. The NYSE Chicago order records the position directly: the firm "engages in algorithmic trading of a variety of asset classes on numerous securities venues" and "trades solely in a proprietary capacity" ✅.
- **It is a technology firm in the sense that matters commercially.** Its own technology page says its engineers "build our own hardware, run large-scale AI workloads, and move faster than most tech companies" and describes "custom ASIC, FPGA, and platform development for low-latency and compute-intensive workloads" ✅ (§8.2). At a firm with no clients, that capability *is* the product: there is no distribution channel to hide behind.
- **It is a registered broker-dealer in part, not a bank and not an exchange.** **Jump Trading, LLC** is a FINRA-registered broker-dealer, CRD 106124, SEC file 8-52989, showing as ACTIVE with a disclosure flag ✅ (FINRA BrokerCheck, the firm search index, queried 24 September 2026). **Jump Execution, LLC** — also carrying the other name **"Jump Liquidity"** — is separately registered, CRD 313060, SEC file 8-70668, ACTIVE, with **no** disclosure flag ✅ (FINRA BrokerCheck, same query). See §2.3 for what those registers do and do not show.
- **It is not the same firm as Jump Crypto or Jump Capital.** Jump Crypto is the digital-asset arm; Jump Capital is the venture arm; **Tai Mo Shan Limited**, the respondent in the SEC matter, is described by the SEC as **a wholly owned subsidiary of Jump Crypto Holdings LLC** ✅. That is a subsidiary of an arm, not of the trading firm's broker-dealer. The public record about one is frequently misapplied to the others (§5.4, §15.5).
- **It is a policy participant.** The CFTC's own published record of a **2 November 2010** meeting lists three Jump Trading attendees — **Matt Schrecengost, Bill DiSomma and Paul Gurinas** — meeting then-Chairman **Gary Gensler** and **Rosemary Hollinger** to discuss "the definition of spoofing as a disruptive trade practice" ✅ (§10.4). The firm is a member of the **FIA Principal Traders Group** ✅/⚠ (the FIA PTG is the policy group for firms trading their own capital; the membership is documented for the peer set in the repository and is carried here as a class fact — see §10.3).
- **It is a firm that discloses almost nothing about its finances.** No revenue, no profit, no capital, no AUM, no valuation, no headcount figure that its own site will let you read. That absence is the finding, and it is §7's subject.

### 1.3 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Full name | Jump Trading LLC (the brand "Jump Trading Group" is used by the firm for some disclosures) | ✅ firm site / disclosures page |
| Founded | 1999 | ✅ (Wikipedia, citing Forbes 2006 and Bloomberg) |
| Founders | Bill DiSomma and Paul Gurinas, former CME pit traders who met in the Deutsche Mark pit | ✅ (Wikipedia) / ⚠ for the biographical detail |
| Headquarters | Chicago, Illinois | ✅ Wikipedia; ✅ NYSE Chicago order ("principal office located in Chicago, Illinois") |
| Registered office (broker-dealer) | 600 West Chicago Avenue, Suite 600, Chicago, IL 60654 | ✅ FINRA BrokerCheck (search index), 24 Sep 2026 |
| Business | Proprietary algorithmic trading as principal; market making and positional strategies | ✅ NYSE Chicago order ("trades solely in a proprietary capacity") |
| Asset classes (firm's own list) | Equities · Options · Credit · Commodities · Digital Assets · Futures | ✅ jumptrading.com/trading, 24 Sep 2026 |
| Strategy spectrum (firm's own words) | "high-frequency to stat arb to discretionary macro" | ✅ jumptrading.com/trading, 24 Sep 2026 |
| Broker-dealer entities | Jump Trading, LLC (CRD 106124 / SEC 8-52989, ACTIVE, disclosure flag Y); Jump Execution, LLC, other name "Jump Liquidity" (CRD 313060 / SEC 8-70668, ACTIVE, no disclosure flag) | ✅ FINRA BrokerCheck firm search index, 24 Sep 2026 |
| Exchange participation | NYSE Chicago Participant from 22 December 2000 | ✅ NYSE Chicago order, Proceeding No. 2018-11-00017 |
| Named arms | Jump Crypto (brand launched 14 Sep 2021); Jump Capital (venture arm; 2012 per Wikipedia, founders disputed — §5.3) | ✅/⚠ |
| Offices (firm's own list) | Chicago, New York, Austin, London, Singapore, Shanghai, Bristol, Mumbai, GIFT City, Sydney, Amsterdam, Hong Kong, Paris (13) | ✅ jumptrading.com, 24 Sep 2026 |
| Group financials | None disclosed. No revenue, profit, capital, AUM or valuation figure established | ✅ verified absence (§7) |
| Largest regulatory matter | SEC order against Tai Mo Shan Limited (Jump Crypto Holdings LLC subsidiary), 20 Dec 2024, US$123,095,287 total | ✅ SEC PR 2024-212; Admin. Proc. File No. 3-22382 |
| Broker-dealer regulatory matter | NYSE Chicago censure + US$250,000 fine, self-reported net capital violation of 4 May 2018, 674 unintended executions | ✅ NYSE Chicago order |
| Reported investigation | CFTC probe of Jump Crypto's trading and investment activities, reported 20 June 2024; no action identified | ⚠ Fortune/Reuters, dated |
| Singapore entity | Jump Trading Pacific Pte. Ltd., incorporated 15 March 2011, Asia Square Tower 1, 8 Marina View | ✅ ACRA via opengovsg.com / sgpbusiness.com (owned by §4.4 of the sibling guide) |

### 1.4 The Vocabulary Decoder

Six terms carry this guide, and each means something narrower than it sounds:

- **The proprietary trading firm.** A firm that trades **its own capital as principal** — not client money, not investor capital. The regulatory consequence is that it sits **outside the prudential perimeter**: no Basel capital regime applies to the group, no deposit funding, no liquidity coverage ratio, no resolution plan, and no supervisor charged with its safety and soundness. Its broker-dealer arms *are* regulated — market access, net capital, supervision — but that is conduct-and-prudential regulation of a dealer, not prudential regulation of a bank-like entity. For a bank assessing one, this is the single most important structural fact: **the usual substitute for disclosure — a prudential regulator's published capital and liquidity data — does not exist here.** What exists instead is the enforcement record (§6) and whatever the firm chooses to publish (§7.2).
- **Market making versus positional trading.** Market making is quoting two-sided prices and earning the spread while keeping inventory near zero; it is a high-volume, low-margin, latency-sensitive business. **Positional trading** (which the firm's own phrase "discretionary macro" gestures at) holds risk for hours, days or longer and earns from direction, carry or basis. The two have opposite funding needs: market making needs speed and inventory-financing, positional trading needs balance sheet, borrow and time. The firm's own strategy sentence — "high-frequency to stat arb to discretionary macro" — claims **both ends of that spectrum**, which is a much broader position than a pure speed shop ✅ (jumptrading.com/trading). The repository treats the spectrum question for the peer set in [Hudson River Trading](hudson_river_trading_guide.md) §3.3, and the mid-frequency/high-frequency distinction is that guide's to own.
- **The latency arms race.** The competition to reduce the time between a market-data event and the resulting order (tick-to-trade latency), and between an exchange's matching event and the firm's knowledge of it. At the limit it is not an engineering competition at all but a **physics** one: signal propagation is bounded by the speed of light in the medium, so the remaining gains come from **shortening the path** (microwave and millimetre-wave links, which travel faster through air than light through glass), **moving the compute** (co-location inside the venue, then inside the venue's own rack), and **removing software** (FPGA and ASIC replacing general-purpose CPUs). The repository's three low-latency guides own the engineering discipline; this guide references them and does not re-derive them (§8.1).
- **Co-location and the compute-sharing question.** Co-location is renting rack space in an exchange's data centre so that the firm's compute sits metres from the matching engine ✅ (a standard industry arrangement; the repository's sibling guides document it for the peer set). The **compute-sharing question** is the live market-structure controversy that follows: as top-tier firms build large research compute estates and AI workloads, the debate turns on whether the same physical and network infrastructure that serves trading also serves shared or outsourced compute, and how venue, cloud and firm compute estates interact. The firm's own site answers part of this for itself — three purpose-built research data centres plus "over 100 co-located compute environments near global exchanges" ✅ (§8.2) — and says nothing about the rest. This guide asserts no compute-sharing arrangement the firm has not stated.
- **The digital-asset arm.** For this firm, **Jump Crypto**, a brand launched **14 September 2021** ✅/⚠ (Businesswire, 14 September 2021), whose self-description now leads with **building** blockchain infrastructure rather than with proprietary crypto trading, while retaining a **"Trade"** pillar ✅ (jumpcrypto.com, accessed 24 September 2026). Its regulatory record is the SEC matter of §6.2, and the entity that settled it was a subsidiary of **Jump Crypto Holdings LLC**.
- **The venture arm.** **Jump Capital**, on the firm's own footer "Network" list alongside Jump Crypto ✅ (jumptrading.com), positioning itself as backing early-stage founders "chasing the leapfrog moments AI is creating across industries" ✅ (jumpcap.com, accessed 24 September 2026). It is a venture investor, not the trading business; a Jump Capital portfolio company is not a Jump Trading client and not evidence of anything about the trading firm.

### 1.5 The Boundary Declaration

This guide is deliberately bounded, and it names the guides that own the material it does not re-derive. **The prop-firm archetype, the market-making franchise, the class-level technology treatment, the talent model, the regulatory and market-structure debate, and the Cymbal Bank worked-example pattern for a firm of this type all belong to [Hudson River Trading](hudson_river_trading_guide.md)** — its §5 (technology), §6 (talent), §7 and §8 (regulatory and market structure) and §10 (the worked example) are cross-referenced here by section and are not restated. **The Singapore market-making landscape and Jump Trading's Singapore facts belong to [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.4** (§5.2 of that guide owns the home-grown tier, including QCP Capital) — this guide cross-references §4.4 by name, adds only what is independently verified, and does not re-derive the Singapore peer landscape. **The technology-of-the-firm-type material belongs to [Citadel LLC](citadel_llc_guide.md) §7**, cross-referenced rather than repeated. **The latency discipline itself belongs to the three low-latency guides** — [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md), [Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md) and [Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md) — which own kernels, NICs, lock-free data structures and the engineering practice; this guide treats only what **this firm itself says** about its infrastructure and stops. Two further delegations: the **Wormhole bridge exploit record is owned by [Smart Contracts](../technology/smart_contracts_guide.md) §9** and is cross-referenced in §6.7 rather than re-derived; and the **Singapore regulatory overlay and the Cymbal Bank persona conventions belong to [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)** and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md). What remains here, and what makes this guide worth writing, is the part of the record that is genuinely this firm's own: its entity map, its dated regulatory file, its capital silence, and the unusually large volume of **first-party technology and culture statements** it publishes — which is the opposite of the usual problem in this genre.

---

## 2. The Identity and the Entities

*This section follows the identity discipline of [ExodusPoint](exoduspoint_guide.md) §2 and [Crane Capital](crane_capital_guide.md) §2: establish the legal entities at the firm's own disclosures and at the regulators' own registers, dated, before any other claim — and then state plainly what the entity map does not establish.*

### 2.1 The Entity Table — From the Firm's Own Disclosures Page

The single most useful first-party document about this group's legal shape is its own **disclosures page**, which groups entities by the disclosure each one publishes. Reproduced as it stands, with what each entry establishes:

| Entity as the firm writes it | What the page attaches | What that establishes | Status |
| --- | --- | --- | --- |
| **Jump Trading, LLC / Jump Execution, LLC** | "Broker-dealer disclosures" | **Two US broker-dealer entities** in the group, named together as the ones with BD disclosure obligations | ✅ jumptrading.com/disclosures, 24 Sep 2026 |
| **Jump Trading Futures, LLC** | "FX Global Code"; "Algo Due Diligence Template"; "FX Global Code Liquidity Provider Disclosure Cover Sheet" | A **futures entity** that adheres to the FX Global Code and that publishes two customer-facing due-diligence artefacts — the most bank-relevant documents the group puts in public | ✅ same page |
| **Jump Trading Europe B.V.** | "A copy of our Pillar III disclosure can be obtained by written request to compliance@jumptrading.com." | A **Dutch (B.V.) entity subject to Pillar III disclosure** — i.e. a prudentially regulated EEA entity inside the group | ✅ same page |
| **Jump Trading International, Ltd.** | UK Gender Pay Gap Disclosure 2023, 2024 and 2025; Modern Slavery Statement | A **UK entity** (a company limited by shares — hence "Ltd.") with UK employment-law and modern-slavery reporting obligations | ✅ same page |
| **Jump Trading Group** | Eightfold AI Recruiting Tool Bias Audit Summary; Healthcare Disclosures | The **brand name** the group uses for certain corporate-level disclosures; whether it is itself a legal entity is **not established** | ⚠ same page / §2.4 |
| **Jump Capital** and **Jump Crypto** | (Named on the site's "Network" footer, not on the disclosures page) | The firm's **own framing of its arms** as network entities rather than as trading entities | ✅ jumptrading.com footer |

Two observations follow directly from the table, and each has bank consequences. **The group is multi-jurisdictional by disclosure obligation, not by marketing**: UK gender-pay-gap reporting and a Modern Slavery Statement establish a UK employer; a B.V. form plus a Pillar III disclosure obligation establishes an EEA entity in scope of CRR/CRD prudential disclosure; an FX Global Code adherence plus a dedicated "Liquidity Provider Disclosure Cover Sheet" establishes a futures/FX entity that **expects to be assessed by counterparties**; and two named US broker-dealers establish the dealer perimeter. Second, **a Singapore entity does not appear on the disclosures list** — which is not evidence of absence, because a Singapore company would not file UK gender-pay-gap, US BD or CRR disclosures. The Singapore entity's existence rests on the **ACRA** record captured in the sibling guide (§11 of this guide; [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.4).

The entity vocabulary is also worth setting down precisely, because the names are confusable: **Jump Trading, LLC** (the principal US broker-dealer and the respondent in the NYSE Chicago matter); **Jump Execution, LLC**, which BrokerCheck also shows under the other name **"Jump Liquidity"** (the execution-side BD); **Jump Trading Futures, LLC**; **Jump Trading Europe B.V.** (Netherlands); **Jump Trading International, Ltd.** (UK); **Jump Trading Group** (the brand); **Jump Trading Pacific Pte. Ltd.** (Singapore, per ACRA); **Jump Crypto Holdings LLC** (the parent of Tai Mo Shan Limited, per the SEC); **Tai Mo Shan Limited** (the SEC respondent); **Jump Crypto** and **Jump Capital** (the arms).

### 2.2 The Founding and the Founders

- **Founded 1999. Headquarters Chicago, Illinois.** ✅ (Wikipedia, citing Forbes, "It's Not The Pits", 2006, and Bloomberg.)
- **Founders: Bill DiSomma and Paul Gurinas**, described as **two former pit traders who met in the Deutsche Mark pit at the Chicago Mercantile Exchange (CME)**, with the firm beginning in the **open-outcry pits** and now trading **almost entirely electronically** ✅/⚠ (the 1999 founding and the founder names are consistent across sources; the biographical detail is Wikipedia's, citing Forbes and Bloomberg).
- **Documented prior occupation: CME pit traders, Deutsche Mark pit.** ✅ (Wikipedia.) **No other prior employer is asserted anywhere in this guide** — the founder narratives in this genre are the most repeated and least re-verified claims in it, and the repository's convention is to name only what a dated source supports.
- **The founders' own documented footprint is unusually good here**, because a **primary regulator record** names them: the CFTC's published **External Meetings** record for **2 November 2010** lists the visitors as **Matt Schrecengost (Jump Trading), Bill DiSomma (Jump Trading) and Paul Gurinas (Jump Trading)** ✅ (§6.5, §10.4). That record confirms DiSomma and Gurinas as senior Jump personnel in 2010 at the firm's own identification — it does not, on its own, confirm their role titles (the record carries no titles) and it does not confirm the DE Mark pit detail.
- **Not established:** the month or day of the 1999 founding; the state of incorporation of any group entity other than the two US broker-dealers (**those two are Delaware LLCs** ✅ — §2.3); any second or third founder; any funder; the founders' current roles or shareholdings; and whether they still hold governance positions. **The task of naming DiSomma and Gurinas as the only founders is a source-limited claim, not an exhaustive one** ⚠.

### 2.3 The Registered Footprint — What the Registers Show

Beyond the firm's own disclosures page, two regulator-side records were established this pass, and both are worth recording with their exact provenance because one of them is a **search-index** result rather than a full firm record:

**FINRA BrokerCheck (the US broker-dealer register), queried 24 September 2026** ✅:

| Entity | CRD | SEC file no. | Status | Branches | Other names | Disclosure flag |
| --- | --- | --- | --- | --- | --- | --- |
| **JUMP TRADING, LLC** | 106124 | 8-52989 | ACTIVE | 6 | "AKAMAI TRADING LLC", "JUMP TRADING, LLC" | **Y** (disclosures on file) |
| **JUMP EXECUTION, LLC** | 313060 | 8-70668 | ACTIVE | 5 | "JUMP EXECUTION, LLC", **"JUMP LIQUIDITY"** | N |

Both list the same office address — **600 West Chicago Avenue, Suite 600, Chicago, IL 60654**, telephone 312-205-8900 ✅ — and both show FINRA approval dates in the record (2024-07-29 for Jump Trading, LLC; 2022-01-20 for Jump Execution, LLC) ✅. Three readings matter. First, **the group's US broker-dealer perimeter is two entities, and the firm's own disclosures page names exactly those two together** — the disclosures page and the register agree, which is a rare and useful alignment. Second, **the "Jump Liquidity" other name** on the execution entity is the identity artefact a bank should carry into name-screening: a liquidity-provision relationship with this group may be papered with an entity whose registered trading name is **not** "Jump Trading". Third, the disclosure flag of **Y** on Jump Trading, LLC corresponds to the **NYSE Chicago matter** of §6.4 — the register is where a bank would first see that a disclosure exists, and the order is where it reads what it was. **The BrokerCheck firm record** returned **HTTP 403 to a first request and answered on a second, with browser-style request headers**, and what it serves carries fields the search index does not ✅ (queried 24 September 2026). Both entities are **Limited Liability Companies**, with size bands Medium and Small respectively in the record. **The disclosure-event list behind that "Y" was still not enumerated** — the item-level endpoint requires authentication (§16.1) ⚠.

**What the BrokerCheck firm record adds** ✅ (queried 24 September 2026):

| Field | **JUMP TRADING, LLC** | **JUMP EXECUTION, LLC** |
| --- | --- | --- |
| State of formation | **Delaware** | **Delaware** |
| Formation date | **31 October 2008** | **2 July 2020** |
| Status date in the record | 21 December 2000 | 20 January 2022 |
| Business-disclosure flag | **Y** | N |
| Items typed "**Regulatory Event**" on the record | **15** | **0 (none)** |
| Non-registered-affiliate disclosures | 1 | 1 |

Three readings follow. First, **the present US dealer vehicle is not the 1999 firm.** *Jump Trading, LLC* is a **Delaware LLC formed on 31 October 2008** — so "founded 1999" (the business) and "formed 2008" (this entity) are both true of different things, and a bank's corporate-tree file should carry both rather than reconcile them into a single date. Second, **the record's status date for Jump Trading, LLC (21 December 2000) sits within a day of the NYSE Chicago participation date of 22 December 2000 recorded in the order (§6.4)** — the register and the SRO document corroborate each other rather than conflict. Third, and most carefully: **the "Y" flag resolves to a count of 15 items typed "Regulatory Event" on the face of the record, and to nothing further.** The endpoint that serves the item text requires authentication and returned **HTTP 403, "Missing Authentication Token"** ⚠, so this guide records the **count and the type** and asserts **nothing** about any individual item or about what the 15 comprise. **"15" is not a count of enforcement actions** — the BrokerCheck "Regulatory Event" category spans the full range of regulatory items a firm's record can carry, and none of the 15 is characterised here.

**The firm's own trading-page and site facts about the wider registered footprint** — membership of **CME Group** and the **New York Stock Exchange**, membership of most European exchanges including **Eurex** and the **London Stock Exchange**, a **Nasdaq Stockholm derivatives membership held as "Jump Trading Europe B.V."** (GlobeNewswire, 17 December 2018) and an **LME Category 3 membership** ✅/⚠ — are carried in the dossier as Wikipedia's citation chain to those primary outlets, and are labelled here as **second-hand unless the outlet is named inline**. The Nasdaq Stockholm membership is the one with a direct entity-name payoff: it is **firm-adjacent corroboration that "Jump Trading Europe B.V." is a real, exchange-facing legal entity** with the name the disclosures page uses (§2.1).

### 2.4 What the Entity Map Does Not Establish

The repository's convention is that an entity section names its own gaps rather than filling them, and this one has real gaps. **None of the following was established this pass, and none is asserted anywhere in this guide:**

- **The ultimate holding company and the ownership chain.** Whether a top-level holding company exists, where it is incorporated, and how **Jump Crypto Holdings LLC**, **Jump Trading, LLC**, **Jump Trading Futures, LLC**, **Jump Trading Europe B.V.**, **Jump Trading International, Ltd.** and **Jump Trading Pacific Pte. Ltd.** relate to one another in a corporate tree. The SEC establishes one edge only: Tai Mo Shan Limited is a wholly owned subsidiary of **Jump Crypto Holdings LLC** ✅.
- **Whether "Jump Trading Group" is a legal entity or a brand.** It appears on the firm's own disclosures page as the heading under which two corporate disclosures sit ✅, and in the firm's own scare-alert copy ("Beware of scams impersonating Jump Trading Group") ✅ — but the disclosures page does not say it is a company, and no register was found that identifies it as one ⚠. **Treat it as a brand until a register says otherwise.**
- **Tai Mo Shan Limited's jurisdiction of incorporation.** The SEC order describes the entity as a wholly owned subsidiary of Jump Crypto Holdings LLC; **the SEC order text captured does not state Tai Mo Shan's jurisdiction of incorporation**, and this guide therefore does not assert a Hong Kong, BVI, Cayman or any other jurisdiction for it despite the name ⚠ (§16.1).
- **Whether Jump Trading Pacific Pte. Ltd. remains an active Singapore entity today.** The ACRA record captured in the sibling guide establishes incorporation on 15 March 2011 and an address ✅; the firm's own current office list confirms **Singapore as a location** ✅; **nothing re-verified the Singapore company's live status as at this pass** ⚠ — the two facts are different facts (§11.4).
- **The number of legal entities in the group**, and which entity employs staff in each of the thirteen named office cities. No register-level mapping of office to entity exists in the sources examined.
- **The state of incorporation of the group's entities other than the two US broker-dealers.** Those two are now established — **Jump Trading, LLC is a Delaware LLC formed 31 October 2008, and Jump Execution, LLC a Delaware LLC formed 2 July 2020** ✅ (§2.3) — but no register was found that states the domicile of **Jump Trading Futures, LLC**, **Jump Crypto Holdings LLC**, **Tai Mo Shan Limited** (which the SEC order captured does not place in any jurisdiction), or any holding entity above them, and **Jump Trading International, Ltd.**'s Companies House particulars were not read.

---

## 3. The History

### 3.1 1999 and the Open-Outcry Context

Firms of this type came out of the **pits**, and the reason is institutional rather than romantic. In 1999 the open-outcry floor was where **price discovery and immediacy of execution** lived, and a trader who understood order flow and the mechanics of a contract had an edge that was in the market's design rather than in a model. The founders of this firm — **Bill DiSomma and Paul Gurinas**, who met in the **Deutsche Mark pit at the CME** ✅/⚠ — founded **Jump Trading** in **1999** in **Chicago**, the city whose exchanges were the centre of that world ✅ (Wikipedia, citing Forbes 2006 and Bloomberg). The firm "began in the open-outcry pits and now trades almost entirely electronically" ✅/⚠ in the same source. **The open-outcry origin matters for three reasons a bank should carry:** the firm's founding competence was in **market microstructure and no-inventory risk-taking**, not in client relationships; its culture began as a **competitor's culture** rather than a service culture; and the transition away from the floor — which removed the human counterparties and made speed the differentiator — is the origin story of the entire latency arms race this guide's thesis is about.

### 3.2 The 2010 CFTC Meeting — A Policy Marker

The earliest **primary** document about the firm's market-structure posture is a **CFTC External Meetings record dated 2 November 2010** ✅. It records a meeting **at Jump Trading**, at which CFTC staff — **Gary Gensler** (then Chairman) and **Rosemary Hollinger** — met **Matt Schrecengost, Bill DiSomma and Paul Gurinas**, all of Jump Trading. The record states: "After a discussion of high frequency trading and the events of May 6, the conversation concerned **the definition of spoofing as a disruptive trade practice** and **transparency and access to SEFs**." The rulemakings listed against the meeting are **XIII. SEF Registration** and **XXIV. Disruptive Trading Practices** ✅ (cftc.gov, External Meetings record).

Three careful readings. First, **this is a meeting record, not an enforcement action** — the distinction is the whole of §6.5's discipline, and it is worth stating here as well as there. Second, it dates the firm's **early and direct engagement with the market-structure rulemaking that followed the 6 May 2010 flash crash**, and it does so in the regulator's own publication. Third, it puts the founders and a third named executive in the room on the **definitional question that would later become the spoofing prohibition** — the firm was engaging the regulator on what spoofing *is*, which is a materially different posture from being the subject of a spoofing case (and the record shows no case).

### 3.3 The Electronic Transition, the Tower, and the 2014 Subpoena

- **The transition to nearly-all-electronic trading**, described as complete by the sources describing the firm today, with the strategy set spanning high-frequency through to positional macro ✅/⚠ (§4.2).
- **2013: a UK affiliate's purchase of a Belgian microwave tower**, reported by **Bloomberg in 2014** ⚠. The dossier carries this as a reported Bloomberg item. It is included here because it is the earliest widely-reported artefact of the firm's **physical-latency** thinking — buying a tower is what a firm does when the remaining latency is the speed of light across a path and the only way to shorten it is to own the path. **This guide asserts no route, no counterparty site and no vendor for that tower** (§8.5, §13.6).
- **April 2014: a New York Attorney General subpoena** issued to this firm alongside **five other high-speed firms** ⚠ (reported; the dossier carries it as part of the 2014 policy wave). The repository's class treatment of that wave and its disposition is in [Hudson River Trading](hudson_river_trading_guide.md) §7.4 and §8.1, and **this guide does not re-derive it**: what is documented for *this* firm is the receipt of a subpoena, not any finding, sanction or resolution against it. **No disposition of the 2014 NY AG inquiry as it concerned Jump Trading was established this pass** ⚠ (§16.1).

### 3.4 The Exchange Footprint From 2000

The NYSE Chicago order supplies an entity fact that no press source does, and it is the earliest **dated, primary** footprint the guide has for the firm: "**Jump is a registered broker-dealer with its principal office located in Chicago, Illinois. Jump became a Participant of NYSE Chicago on December 22, 2000**" ✅ (NYSE Chicago order, Proceeding No. 2018-11-00017). Two points follow. First, the firm's **exchange-facing broker-dealer status is at least as old as 2000** — one year after the 1999 founding — which is the structural fact that distinguishes a proprietary trading firm with dealer status from a purely private trading vehicle. Second, the order states the business model in the regulator's words — the firm "engages in algorithmic trading of a variety of asset classes on numerous securities venues" and "**trades solely in a proprietary capacity**" ✅ — which is the single best primary description of what this firm is, and the sentence §1's thesis rests on.

### 3.5 The Arms — 2012 and 2021

- **2012 (or June 2012): the venture arm.** Wikipedia states that "Gurinas and DiSomma also founded a venture capital firm, Jump Capital, in **June 2012**" ⚠, and adds that by **January 2016** the firm had invested in **30 companies**, citing Crain's Chicago Business, January 2016 ⚠. **Jump Capital's own team page, however, lists Sach Chitnis and Mike McMahon as Co-Founders** ✅ — the reconciliation is set out in §5.3 and is not resolved here.
- **14 September 2021: the digital-asset arm.** **Jump Crypto** was announced as a new brand on 14 September 2021 ✅/⚠ (Businesswire, "Jump Trading Group Launches Jump Crypto"), with **Kanav Kariya** named as its head ✅/⚠ (Bloomberg, 14 September 2021 — the piece is titled "Jump Names 25-Year-Old Former Intern as Crypto Unit Head"). Note the entity name in the Businesswire headline: **"Jump Trading Group"** — the brand, used by the firm itself in a press release, which is the best evidence available that the brand is used corporately even though its legal status is unestablished (§2.4).
- **May 2019: a documented venture investment.** Jump Capital invested **US$5 million** in **The Small Exchange** in May 2019 ✅/⚠ (Wikipedia, citing the May 2019 Businesswire release in which the headline names **Citadel Securities** and **Jump Capital** in a US$10m round). This is a **venture-arm participation in a funding round** — it is not a trading-firm fact, not a client, and not a capital figure for the group (§5.4).

### 3.6 2024 and After

The period a bank's file will actually be built from:

- **20 June 2024: the reported CFTC probe.** **Fortune** reported, carried by **Reuters**, that the **CFTC was probing Jump Trading's involvement in the cryptocurrency space**, examining Jump Crypto's trading and investment activities; the reports said scrutiny does not imply misconduct ⚠ (§6.5). **No enforcement action has been identified** ✅ as a finding about the search performed.
- **June 2024: the departure of the crypto arm's president.** **Kanav Kariya**, president of Jump Crypto, announced his departure in June 2024, reported by multiple outlets (e.g. Cointelegraph, 24 June 2024) ⚠. The relationship to the probe is **press characterisation**, not a sourced fact (§6.8).
- **20 December 2024: the SEC settlement.** The order against **Tai Mo Shan Limited** — a wholly owned subsidiary of **Jump Crypto Holdings LLC** — with total sanctions of **US$123,095,287** ✅ (§6.2). This is the most consequential dated event in the guide's subject matter.
- **2025–2026: the follow-on and the firm's continued activity.** The SEC's Fair Fund process continued through the **20 August 2026** notice of a proposed plan of distribution ✅ (§6.3); the firm's Signals page carries dated 2026 publications and events ✅ (§8.4, §9.3); and the firm's crypto arm is publishing dated research through April 2026 ✅ (§5.1). Two further **reported** items sit in the press record and are flagged: a **Business Insider** report (April 2026) that two long-tenured researchers — **Yiming Zhang** (joined 2009) and **Darko Kirovski** (joined 2011) — had left ⚠, and crypto-press reports (late 2025/2026) that the firm had **doubled a prediction-markets team** and holds equity in prediction-market venues ⚠. **Neither is verified at a primary source and neither is asserted as fact** (§9.5, §12.4).

---

## 4. The Business

### 4.1 The Structural Point — No Clients

A proprietary trading firm has **no clients**, and that single fact reorganises everything about how its business must be described. There is no client-revenue composition to disclose, because there are no clients; no assets under management, because no money is managed for anyone else; no fee schedule, no mandate, no client-asset mix, no flow-quality statistic. The firm's own regulator record puts the position plainly — the firm "**trades solely in a proprietary capacity**" ✅ (NYSE Chicago order, Proceeding No. 2018-11-00017) — and the practical consequence for anyone researching the firm is that **the revenue-composition analysis a bank would write about a broker, an asset manager or an exchange cannot be written about this firm at all**. There is no public document that states where its revenue comes from, in what proportion, at what margin, or with what volatility.

That has a knock-on effect on this section's honesty: **the business description here must be structural rather than financial.** It can say what the firm trades and what strategies it says it runs. It cannot say what any of it earns.

### 4.2 The Firm's Own Asset-Class List and Strategy Spectrum

The firm's own trading page is the primary source, and it is used in preference to any encyclopaedic list:

- **Asset classes the firm says it trades** ✅ (jumptrading.com/trading, accessed 24 September 2026): **Equities · Options · Credit · Commodities · Digital Assets · Futures.** Six classes.
- **Strategy spectrum, in the firm's own words** ✅ (same page): strategies "span the full spectrum of modern trading" from "**high-frequency to stat arb to discretionary macro**".
- **Frame, in the firm's own words** ✅ (same page): "We don't just trade. We build the machine that powers it." … "We're not just deploying strategies; we architect the entire machine." … "If there's signal, we chase it. If there's edge, we scale it."
- **The home page's own summary** ✅ (jumptrading.com, same access): "Every asset class. Every time horizon. No boundaries." and "We don't chase what's working. We build what works next." — with the firm's self-description as "a global trading firm where smart people build remarkable things", trading across "**Every asset class. Every time horizon.**"

Two things are notable about this self-description. First, **it is a self-description** — this guide tags it as such and does not treat the asset-class list as a disclosure of exposure, size or concentration, because it is not one. Second, **the strategy claim is unusually broad for a firm of this type.** "High-frequency to stat arb to discretionary macro" spans sub-second speed-driven trading, statistical arbitrage (typically minutes-to-days holding periods, relative-value and factor-driven), and **discretionary macro** (positional, judgement-driven, held over long horizons across rates, FX and commodities). A firm that runs all three is running three different businesses with three different cost, funding and risk profiles — which is itself a structural statement about the firm's capital position, in the same way the repository says of the peer set that a mid-frequency expansion is a move from the speed business into the capital-and-signal business ([Hudson River Trading](hudson_river_trading_guide.md) §3.3).

### 4.3 Reading the Spectrum — High-Frequency, Stat Arb, Discretionary Macro

The three bands demand different things, and the firm's own material says nothing about how it weights them: **high-frequency** needs co-location and custom silicon (which the firm's technology page documents ✅); **statistical arbitrage** needs research compute, data, borrow and financing (the firm's ML and research-data-centre statements ✅); and **discretionary macro** needs balance sheet, duration and tolerance for drawdown — named on the firm's trading page and **otherwise undisclosed** ✅. Any table of that shape has a thin right-hand column for the same reason: the firm publishes **what it trades and how it labels its strategies, and nothing about the size, mix or performance of any of them** ✅ — a finding worth stating rather than filling in with a plausible guess.

### 4.4 What the Business Description Cannot Contain

- **Revenue composition.** Not disclosed. There is no segment reporting anywhere, because there is no reporting anywhere (§7).
- **Notional volumes, market share or venue rankings.** None established for this firm this pass. **No claim of "x% of volume" of any kind appears in this guide**, for the reason the repository gives elsewhere: a market-share figure needs a dated, named source, and none was found for this firm.
- **Client or counterparty identities.** The firm has no clients (trading-wise), and **no counterparty, clearing member, prime broker or exchange member is named in this guide as a relationship** (§13.6).
- **The economic shape of the business.** How much of the P&L comes from market making versus positional risk is not merely undisclosed — for a private principal trading firm it is the most closely held fact there is.

---

## 5. The Arms and the Affiliates

*This section's job is to state each arm's **current status, dated**, and then to insist on a distinction the public record routinely drops: the trading firm, the digital-asset arm and the venture arm are three different things, and evidence about one is not evidence about the others.*

### 5.1 Jump Crypto — Current Status

**Status as at this pass: live and publishing** ✅ (jumpcrypto.com, accessed 24 September 2026).

| Aspect | What was verified | Status |
| --- | --- | --- |
| Site | **jumpcrypto.com is live** and served content on 24 Sep 2026 | ✅ |
| Self-description | "Builders of blockchain technology revolutionizing open, community-driven networks." | ✅ |
| Strategy in its own words | "identify real world constraints and build decentralized consortiums to convert systemic bottlenecks into shared breakthroughs. We do it by contributing time, capital, and code." | ✅ |
| Structure | Three titled pillars: **Trade / Build / Invest** | ✅ |
| Dated publications on its own site | "Dual Flow Batch Auction" — 21 August 2025; "PropAMMs and the Next Chapter of Permissionless Market Structure" — 15 April 2026; "Quantum Migration Paths for Solana" — 27 April 2026 | ✅ |
| Framing on the site | A scam warning: "Beware of scams impersonating Jump Trading Group. We only communicate through our official accounts." | ✅ |

**The honest nuance, stated rather than smoothed:** the arm's self-description now **leads with building blockchain infrastructure** rather than with proprietary crypto trading, while a **"Trade"** pillar remains and is described in its own words as "Identify real-world constraints & deep market insights through **active market participation**" ✅. So the documented position is: **the arm is active, its public emphasis is on infrastructure building, and the scale of its current trading book is not disclosed.** Two failure modes are avoided here: **this guide does not characterise the arm as wound down** (the site is publishing new research in 2026 ✅), and **it does not present a 2021–2023 description of the arm as current** (the framing has moved).

**Launch and leadership, dated:** Jump Crypto was announced as a new brand on **14 September 2021** ✅/⚠ (Businesswire, "Jump Trading Group Launches Jump Crypto"), with **Kanav Kariya** named as its head ✅/⚠ (Bloomberg, 14 September 2021, "Jump Names 25-Year-Old Former Intern as Crypto Unit Head"). Kariya **departed in June 2024** ⚠ (§6.8). **The arm's current head was not established this pass** ⚠ (§16.1).

**Regulatory record:** the **SEC order of 20 December 2024** against **Tai Mo Shan Limited**, a wholly owned subsidiary of **Jump Crypto Holdings LLC**, is this arm's regulatory record and is set out in full in §6.2. The **reported CFTC probe** of 20 June 2024 concerns this arm's trading and investment activities and is a **reported investigation with no action identified** (§6.5) ⚠.

**Documented event association, cross-referenced and not re-derived:** the **Wormhole bridge exploit of February 2022** (~120,000 wETH, reported at approximately US$320 million), in which the repository records that **the parent company Jump restored the funds** and a **US$10 million white-hat bounty** was offered — the record is owned by [Smart Contracts](../technology/smart_contracts_guide.md) §9 (and appears also in [DeFi](../technology/defi_guide.md), [Blockchain Technology](../technology/blockchain_technology_guide.md) and [Ethereum](../technology/ethereum_guide.md)) ✅/⚠ for the dollar figure, which is **reported**. See §6.7.

### 5.2 Jump Capital — Current Status

**Status as at this pass: live** ✅ (jumpcap.com, accessed 24 September 2026).

| Aspect | What was verified | Status |
| --- | --- | --- |
| Site | **jumpcap.com is live** | ✅ |
| Positioning | The firm "backs early-stage founders chasing the leapfrog moments **AI** is creating across industries" | ✅ |
| Co-founders named on its own team page | **Sach Chitnis — Co-Founder and Partner**; **Mike McMahon — Co-Founder and Partner** | ✅ |
| Other named team members | Saaya Pal (Partner), Will Walker (Senior Associate), Yelena Shkolnik (Partner), Aqil Pasha (Vice President), Jason Felger (Partner), Ola Jewusiak (VP of Marketing), Miles Scheffler (Data & Operations Analyst), Robb Hutchins (Partner, Portfolio Growth), Pete Carroll (Operating Partner), Kasey Mitria (Operations Manager), Eliza Chamberlain (Senior Associate) | ✅ |
| Site testimonials | Multiple portfolio-founder quotes on the team/history pages (Amos Stern, Matt Kunkel, Brian Barnes, Carter Malloy, Kian Katanforoosh, Or Eshed, Farhad Massoudi) | ✅ as published |

**What was not established: any fund size, any AUM, and any vintage year for Jump Capital.** None was found this pass, and none is stated ✅ (verified absence). The only dated capital-adjacent facts about the arm in the sources examined are Wikipedia's 2012 founding year ⚠ and its statement that by **January 2016** the arm had invested in **30 companies** (citing Crain's Chicago Business, January 2016) ⚠, plus the **US$5 million** May 2019 investment in **The Small Exchange** ⚠ (§3.5). **A portfolio company of this arm is not a Jump Trading client, and a Jump Capital investment is not a Jump Trading financial** (§5.4).

### 5.3 The Jump Capital Founder Reconciliation

This is a genuine documentary conflict, and the repository's convention is to record both sides rather than silently pick one:

| Account | Statement | Source |
| --- | --- | --- |
| The encyclopaedic account | "**Gurinas and DiSomma also founded a venture capital firm, Jump Capital, in June 2012**" | ⚠ Wikipedia (Jump Trading article) |
| The firm's own account | Team page lists **Sach Chitnis — Co-Founder and Partner** and **Mike McMahon — Co-Founder and Partner** | ✅ jumpcap.com team page, accessed 24 Sep 2026 |

**Both are recorded here, neither is resolved.** The two are not logically incompatible — a venture arm can be *initiated* with capital and sponsorship from the trading firm's founders while its *operating co-founders* are the investment professionals who run it, and "founded" then means two different things in the two sentences. But **the sources examined do not establish which reading is correct**, and this guide does not choose. What can be said with confidence is narrower and still useful: **the arm's own primary source names Chitnis and McMahon as co-founders** ✅, and **any statement that the trading firm's founders personally founded Jump Capital rests on the secondary source** ⚠. A bank documenting the group's control structure should treat the venture arm's ownership and control as **unestablished** (§16.1).

### 5.4 Why the Distinction Matters

Four distinctions, each of which prevents a specific error:

1. **The trading firm is not the digital-asset arm.** The SEC matter of §6.2 names **Tai Mo Shan Limited**, a subsidiary of **Jump Crypto Holdings LLC** — **not Jump Trading, LLC**. A bank onboarding Jump Trading, LLC for equities or futures clearing is not onboarding the respondent in the SEC order, and **must not describe the SEC order as a Jump Trading, LLC matter** ⚠ (the mis-application runs both ways and appears in the press).
2. **The trading firm is not the venture arm.** Jump Capital's portfolio, its team and its investment decisions are the venture business. A Jump Capital-backed company is not a Jump Trading client or counterparty, and — the direction that matters most for capital analysis — **a Jump Capital fundraise or portfolio valuation is not a Jump Trading financial figure**.
3. **The arms' own sites are the primary sources for the arms' own status** — the disclosures page and the footer "Network" list are how the firm itself frames them ✅ — and **neither arm publishes financials**, so nothing from either can be used to size the trading firm (§7).
4. **The group's brand is used loosely, including by the firm.** The 2021 Jump Crypto launch release is headlined to **"Jump Trading Group"** ✅, and the firm's scare-alert copy refers to impersonation of "Jump Trading Group" ✅. **"Jump Trading Group" is therefore how the group is named publicly in the firm's own material, and it is still not established as a legal entity** (§2.4) ⚠. Do not onboard a brand.

---

## 6. The Incidents and the Regulatory Matters

### 6.1 How to Read This Section

This is the guide's highest-risk section, and it is built as a **dated record, not a narrative of blame**. Every item below carries, in the order a file would need them: **authority · instrument · date · respondent · findings as the authority states them · outcome.** Three rules govern the whole section and are stated once here rather than repeated:

1. **Nothing is reconstructed from memory.** Every regulatory matter below is taken from the authority's own document (the SEC press release and its harmed-investor page; the NYSE Chicago order) or from a **dated, named** press report, and the instrument and release numbers are given because the repository's convention is that a regulatory matter without its instrument and date is not a fact.
2. **The distinction between an action, a meeting, a report and an allegation is maintained everywhere.** A settled order is not a complaint; a reported probe is not a charge; an allegation in a complaint is not a finding. The labels in the table of §6.9 carry that discipline.
3. **No finding is softened and no unverified detail is added.** Where the authority said "without admitting or denying", this guide says so **because the authority said so**. Where the source does not name a party (the clearing broker in the NYSE Chicago order), **this guide does not name it either** (§13.6).

### 6.2 The SEC Matter — Tai Mo Shan Limited, 20 December 2024

| Element | Detail | Status |
| --- | --- | --- |
| **Authority** | U.S. Securities and Exchange Commission | ✅ |
| **Instrument** | Order instituting and simultaneously settling cease-and-desist proceedings. **Admin. Proc. File No. 3-22382**; the Order is **Release No. 33-11349**; press release **2024-212** | ✅ |
| **Date** | **20 December 2024** | ✅ |
| **Respondent** | **Tai Mo Shan Limited**, described by the SEC as **a wholly owned subsidiary of Jump Crypto Holdings LLC** | ✅ |
| **Finding 1** | From at least **January 2021 to May 2022**, Tai Mo Shan **acted as a statutory underwriter** with respect to certain offers and sales of **LUNA** — a crypto asset issued by Terraform Labs PTE Ltd. and "offered and sold as a security" — offering and selling securities in unregistered transactions → violated **Securities Act Sections 5(a) and 5(c)** | ✅ |
| **Finding 2** | In **May 2021**, when UST devalued from its $1 peg, Tai Mo Shan and Terraform entered an agreement that incentivised Tai Mo Shan to purchase UST in exchange for Terraform "vesting" Tai Mo Shan's existing option to purchase LUNA at a discount; Tai Mo Shan bought **more than $20 million of UST** to restore the peg. In light of Terraform's prior statements that its algorithmic mechanism would maintain the peg, the SEC found Tai Mo Shan **acted negligently** by trading UST in a manner that **deceived the market** into believing the algorithmic mechanism was working → violated **Securities Act Section 17(a)(3)** | ✅ |
| **Sanction** | Disgorgement **$73,452,756** + prejudgment interest **$12,916,153** + civil money penalty **$36,726,378** = **$123,095,287** total, to the Commission; **cease and desist** | ✅ |
| **The admitting clause** | The settlement was entered **"without admitting or denying the SEC's findings"** — **the SEC's own words; say so because the authority said so** | ✅ |
| **Payment** | "**The Respondent has paid in full.**" | ✅ SEC harmed-investor page |
| **Context the SEC itself supplies** | Terraform and its founder **Do Kwon** were found liable for **fraud and unregistered securities offerings** by a federal district court in **April 2024**, and agreed to pay **$4.5 billion** to harmed investors | ✅ SEC PR 2024-212 |
| **The SEC Chair's own framing** | "This case reminds us that, too many times in the crypto markets, we've seen significant investor losses due to fraud. … Regardless of the labels, crypto market participants should comply with the securities laws where applicable and not deceive the public." (Gary Gensler) | ✅ SEC PR 2024-212 |
| **Sources** | sec.gov/newsroom/press-releases/2024-212 · sec.gov/enforcement-litigation/distributions-harmed-investors/tai-mo-shan-limited | ✅ |

Two things this guide deliberately does **not** do with this matter. It does not describe it as a matter against **Jump Trading, LLC** — the respondent is a **subsidiary of the crypto arm's holding company** ✅. And it does not describe the finding as **fraud** — the SEC's findings are **statutory-underwriter registration violations** under §5 and **negligent** misleading conduct under §17(a)(3) ✅, and the negligence standard is the one the SEC itself applied to this respondent (the district court's fraud finding was against **Terraform and Do Kwon**, not against Tai Mo Shan) ✅.

### 6.3 The Fair Fund Follow-On

The order did not end the matter; it began a distribution process, and every step is primary:

| Date | Instrument | What it did | Status |
| --- | --- | --- | --- |
| **20 December 2024** | Release No. 33-11349 | Order instituting and settling the proceedings; sanctions imposed; respondent paid in full | ✅ |
| **31 January 2025** | Harmed-investor page first published (page reviewed/updated 24 Aug 2026) | SEC's investor-facing record of the matter | ✅ |
| **13 March 2025** | **Release No. 34-102662** | Order appointing **Miller Kaplan Arase LLP** as **Tax Administrator** of the Fair Fund | ✅ |
| **20 August 2026** | **Release No. 34-106168** (and the Proposed Plan) | Commission published the **notice of the proposed plan of distribution and opportunity for comment** — a **30-day comment period** | ✅ |

The Fair Fund was created under **Sarbanes-Oxley §308(a)** so that the penalty, disgorgement and interest collected can be distributed to harmed investors; it has been deposited in a Commission-designated account at the **U.S. Department of the Treasury**, with accrued interest added to it ✅ (SEC harmed-investor page). The **claim period runs 23 May 2021 through 8 May 2022** — from the date **Tai Mo Shan entered into the agreement to purchase $20 million of UST** to **the eve of the Terraform ecosystem collapse**, with the page recording that on **9 May 2022** UST's value fell ✅. Distributions go to investors harmed on **UST** purchases in that window, per the Plan's allocation methodology ✅. **What is unresolved: as at this pass the Proposed Plan was in its comment period; no distribution had been made and no final distribution date is established** ⚠ (§6.10).

### 6.4 The NYSE Chicago Matter — Jump Trading, LLC

| Element | Detail | Status |
| --- | --- | --- |
| **Authority** | **NYSE Chicago, Inc.** — an **exchange SRO**, **NOT the SEC** | ✅ |
| **Instrument** | "Order Instituting Proceedings, Accepting Settlement, Making Findings, and Imposing Sanctions." **Proceeding No. 2018-11-00017**. Respondent **Jump Trading, LLC**, **CRD No. 106124** | ✅ |
| **Settlement posture** | The respondent submitted an Offer of Settlement accepted by NYSE Chicago; **"without admitting or denying the findings herein," except as to the Exchange's jurisdiction** over the firm and the subject matter, **which was admitted solely for the proceeding** | ✅ |
| **Origin** | An investigation by **NYSE Chicago's Member Regulation Division** following a **self-reported net capital violation by the firm on 4 May 2018** | ✅ |
| **Facts as found** | A team lead edited the code of an end-of-day netting algorithm ("**Algo 1**") on 3 May 2018 to improve execution percentage; the change passed the team's **historical regression test suite** but contained an error. On **4 May 2018** the algorithm **increased** the team's positions instead of reducing them — **674 unintended executions, totalling hundreds of millions of dollars** — accumulating significant short positions and leaving the firm out of compliance with applicable net capital requirements **for several hours**. The firm began discussions with **its clearing broker** to exit the positions and executed a **block trade** with it the same day, returning to compliance | ✅ |
| **The order's own explanation of why testing missed it** | The regression suite "in conjunction with Jump's trading platform, did not accurately simulate the conditions that would cause the modified algorithm to malfunction" | ✅ |
| **Risk-control deficiencies found** | (1) Team A's pre-trade single-order control limited orders to 200,000 shares / $40m gross notional, but **did not block** erroneous orders — it only **reduced** their size to below the limit; (2) the capital limit was based **only on executed orders** and ignored **placed-but-unexecuted** limit-on-close orders, so by the time it bound, it was "too late to prevent the majority of the erroneous orders from executing"; (3) the firm "**generally failed to sufficiently document how it determined its market access controls**" | ✅ |
| **Findings** | Violations of **Exchange Act Rule 15c3-5(b)** and **15c3-5(c)(1)** (market-access risk controls); **Exchange Act Rule 15c3-1** (net capital); **NYSE Chicago Article 6, Rule 5** (supervision); and **Article 7, Rule 3(a)(1)(A)** | ✅ |
| **Sanction** | **Censure and a $250,000 fine**, payable within **15 days** of the order, **plus an undertaking** to certify within **90 days** that the risk-control deficiencies were addressed | ✅ |
| **Disciplinary history** | "The Firm does not have any relevant disciplinary history." | ✅ |
| **Entity facts stated** | Jump Trading, LLC is a registered broker-dealer with its principal office in **Chicago, Illinois**; it **became a Participant of NYSE Chicago on 22 December 2000** | ✅ |
| **Source** | nyse.com/publicdocs/nyse/markets/nyse-chicago/disciplinary-actions/**2019**/Jump%20Trading%20Order.pdf | ✅ |

Three readings, all of which matter to a bank (§13.1):

- **This is an operational-risk matter, not a market-integrity one.** The findings are about **pre-trade controls, net capital and supervision** — the failure mode was a software regression that the firm's own testing did not catch, in a firm the exchange itself records as having **no relevant disciplinary history** ✅.
- **The two control lessons are exactly the two a bank builds into market-access due diligence:** a **size cap that reduces rather than blocks** is not a block, and a **capital limit that counts only executed orders** does not see the limit-on-close risk that is queued but not yet filled ✅. This is why the matter is the concrete evidence of why a bank cares about a counterparty's pre-trade controls (§13.1).
- **The clearing broker is not named, and this guide does not name it.** The order says "**its clearing broker**" ✅ and nothing more; **naming a bank, broker or exchange as Jump's clearing broker is prohibited in this guide** and would also be an unsupported inference from a document that chose not to say (§13.6).
- **Issuance date caution:** the PDF sits in NYSE's **2019** disciplinary-actions folder, i.e. the order **post-dates the May 2018 incident**; the signature/date line in the PDF's text layer extracted garbled and **the exact issuance date was not read cleanly** ⚠. The guide states the **incident date (4 May 2018) as verified** and describes the order as issued in the **2019** folder ✅/⚠, rather than asserting a day it could not read (§16.1).

### 6.5 The CFTC — A Meeting and a Report, Not an Action

Three distinct things exist in the CFTC record about this firm, and conflating any two of them is the error this subsection exists to prevent.

**(i) A primary meeting record — a meeting, NOT an action.** The CFTC's published **External Meetings** record for a meeting **at Jump Trading on 2 November 2010**: CFTC staff **Gary Gensler** (then Chairman) and **Rosemary Hollinger**; visitors **Matt Schrecengost, Bill DiSomma and Paul Gurinas** (all Jump Trading); subject: after a discussion of high-frequency trading and the events of **May 6** (the 2010 flash crash), the conversation concerned **the definition of spoofing as a disruptive trade practice** and **transparency and access to SEFs**; rulemakings listed: **XIII. SEF Registration** and **XXIV. Disruptive Trading Practices** ✅ (cftc.gov/node/171661). **This is a meeting record.**

**(ii) The absence of a CFTC enforcement action.** **No CFTC enforcement action — order, complaint or settlement — naming Jump Trading or its affiliates was found** in this pass or the dossier pass, having searched the CFTC's enforcement listings ✅ as a finding about the search performed. **Record the absence, and record its limits:** an empty search in a listing is evidence about a search, **not** evidence that no such action exists — the repository's convention is to write that limitation down rather than convert it into a negative assertion (§16.1).

**(iii) A dated press report of a probe.** On **20 June 2024**, **Fortune** reported, carried by **Reuters**, that the **CFTC was probing Jump Trading's involvement in the cryptocurrency space**, examining **Jump Crypto's trading and investment activities**; the reports stated that scrutiny does not imply misconduct ⚠. **This is a reported investigation, dated, with no CFTC action identified.** This guide does **not** state or imply any allegation the CFTC has not made, and it does not treat the report as an action.

**Whether any CFTC action has followed the reported 2024 probe could not be established this pass** ⚠ and is written into §16.1.

### 6.6 The 2023 Class Action — Allegations Only

| Element | Detail | Status |
| --- | --- | --- |
| **Filed** | **9 May 2023** | ✅ as filed (complaint) |
| **Caption (per the filed, file-stamped complaint)** | **Taewoo Kim, individually and on behalf of all others similarly situated, v. Jump Trading, LLC and Kanav Kariya** | ✅ complaint caption |
| **Claims** | Violations of the **Commodity Exchange Act**, **CFTC regulations** and **unjust enrichment**; alleging Jump participated in manipulation of UST and aided and abetted Do Kwon | ✅ complaint (allegations) |
| **Alleged quantum** | **over $1.28 billion** in profits from selling discounted LUNA received in exchange for propping up UST | ⚠ **an allegation from the complaint** — not a finding, not a figure established by any authority |
| **That these are allegations** | **These are allegations in a complaint. No finding.** | ✅ |
| **Later docket development** | Reported: the complaint was **amended in August 2024** to add plaintiffs (**Kashyap Patel, Kerry Woolley, Ken Worsham**) and defendants (**Jump Crypto Holdings LLC, William DiSomma**), and on **Friday 9 May 2025** Judge **Georgia Nick Alexakis** of the **U.S. District Court for the Northern District of Illinois** **denied** the defendants' motion to compel arbitration, keeping the case in federal court | ⚠ **single secondary source** (BlockTribune) — reported, not verified at the docket |
| **Current status / outcome** | **Not established.** No judgment, no dismissal, no settlement identified | ⚠ §16.1 |

**A separate, later civil matter — also reported, also allegations only.** Crypto-press reports (December 2025) state that **Todd R. Snyder**, the court-appointed **plan administrator** of Terraform Labs (appointed by the U.S. Bankruptcy Court on **1 October 2024**), filed a **US$4 billion lawsuit** in the **U.S. District Court for the Northern District of Illinois** on **18 December 2025** (some reports say 19 December) against **Jump Trading**, **William DiSomma** and **Kanav Kariya**, alleging secret token deals, market manipulation and undisclosed support for UST's peg ⚠. Multiple outlets carry it; the date varies by a day between them; **no primary filing was read this pass**, and **none of it is asserted as fact** ⚠. One outlet reports the firm's response characterising the suit as a "desperate attempt" to transfer responsibility — **press characterisation of a response**, flagged ⚠. **This matter, like the 2023 class action, is recorded here as a dated, reported claim, not as an outcome** (§16.1).

### 6.7 The Wormhole Exploit — Cross-Referenced

The **Wormhole bridge exploit of 2 February 2022** — a signature-verification bypass on the Solana side of the bridge, **~120,000 wETH ≈ US$320 million (reported)** — is a matter in which the repository records that **the parent company Jump restored the funds** and a **US$10 million white-hat bounty** was offered. **This record is owned by [Smart Contracts](../technology/smart_contracts_guide.md) §9 (the incident table at its line 632, with the bounty noted in its section on bug bounties)**, and appears also in [DeFi](../technology/defi_guide.md), [Blockchain Technology](../technology/blockchain_technology_guide.md) and [Ethereum](../technology/ethereum_guide.md). **This guide cross-references it and does not re-derive it.** Two cautions carried over with it: the dollar figure is **reported** ⚠, and the restoration of funds by "the parent company Jump" is recorded in the repository in those terms — **the exact entity that restored the funds, and its jurisdiction, were not established**.

### 6.8 The 2024 Departure

**Kanav Kariya**, president of Jump Crypto, **announced his departure in June 2024** ⚠ (his own social-media announcement, reported by multiple outlets, e.g. Cointelegraph, 24 June 2024). It is **contemporaneous** with the reported CFTC-probe coverage of 20 June 2024 ⚠. **The causal linkage between the two is press characterisation and is labelled as such** — this guide does not assert that the departure was caused by, or connected to, the reported probe, because no source establishes that. The arm's current leadership after the departure was **not established this pass** ⚠ (§16.1).

### 6.9 The Dated Record

Every regulatory and legal item in one table, in the form a compliance file needs — authority, instrument, date, outcome — with the item's **kind** made explicit so that a screening system cannot read across categories:

| Date | Kind | Authority | Instrument | Respondent / party | Outcome |
| --- | --- | --- | --- | --- | --- |
| 2 Nov 2010 | **Meeting record** (not an action) | CFTC | External Meetings record | Jump Trading (Schrecengost, DiSomma, Gurinas) | Discussion of the definition of spoofing; SEF registration and disruptive trading practices listed as rulemakings |
| Apr 2014 | **Subpoena** (reported) | New York Attorney General | Subpoena, reported | Jump Trading, among six high-speed firms | **No disposition established** ⚠ |
| 4 May 2018 | **Incident (self-reported)** | — | Self-report to NYSE Chicago | Jump Trading, LLC | Net capital violation for several hours; 674 unintended executions |
| Issued 2019 (exact date not read) | **SRO settled order** | NYSE Chicago, Inc. | Order Instituting Proceedings, Accepting Settlement, Making Findings, and Imposing Sanctions; Proceeding No. **2018-11-00017** | **Jump Trading, LLC** (CRD 106124) | **Censure + US$250,000 fine + 90-day certification undertaking.** Settled **without admitting or denying** the findings. Violations of Exchange Act Rules 15c3-5(b), 15c3-5(c)(1), 15c3-1 and NYSE Chicago Art. 6 R.5, Art. 7 R.3(a)(1)(A) |
| 9 May 2023 | **Civil complaint** (allegations) | N.D. Ill. (filed by private plaintiffs) | Class action complaint — *Taewoo Kim v. Jump Trading, LLC and Kanav Kariya* | Jump Trading, LLC; Kanav Kariya | **No finding; status not established** ⚠ |
| 20 Jun 2024 | **Reported investigation** | CFTC (as reported) | None — press report | Jump Crypto's trading and investment activities | **No action identified** ⚠ |
| Jun 2024 | **Personnel change** (reported) | — | — | Kanav Kariya | Departure announced; linkage to the probe is press characterisation ⚠ |
| 20 Dec 2024 | **Settled regulatory order** | **SEC** | Order instituting and simultaneously settling cease-and-desist proceedings; **Admin. Proc. File No. 3-22382**; **Release No. 33-11349**; PR 2024-212 | **Tai Mo Shan Limited** (wholly owned subsidiary of **Jump Crypto Holdings LLC**) | **US$123,095,287** (disgorgement $73,452,756 + prejudgment interest $12,916,153 + penalty $36,726,378); cease and desist; **without admitting or denying**; paid in full |
| 13 Mar 2025 | **Fair Fund administration** | SEC | **Release No. 34-102662** | — | Miller Kaplan Arase LLP appointed Tax Administrator |
| 9 May 2025 | **Docket development** (reported) | N.D. Ill. | Order denying motion to compel arbitration | Jump Trading, LLC et al. | Motion denied; case remains in federal court ⚠ single secondary source |
| 18/19 Dec 2025 | **Civil complaint** (reported; allegations) | N.D. Ill. | Terraform Labs plan administrator's complaint, reported | Jump Trading; William DiSomma; Kanav Kariya | **No finding; reported only** ⚠ |
| 20 Aug 2026 | **Notice and proposed plan** | SEC | **Release No. 34-106168** | — | Proposed Fair Fund plan of distribution published; 30-day comment period |

### 6.10 What Remains Unresolved

Stated plainly, so a reader can carry the uncertainty rather than resolve it by inference:

- **Whether the 2023 class action is still live, dismissed or settled** — not established ⚠.
- **Whether any CFTC or other US regulator action has followed the reported June 2024 probe** — none identified; **an empty search, not a negative proof** ⚠.
- **Whether the 2025 Terraform plan-administrator suit has been served, answered, moved against or consolidated** — not established; the filing's existence itself rests on press reports ⚠.
- **The disposition of the April 2014 NY AG inquiry as it concerned this firm** — not established ⚠.
- **Tai Mo Shan Limited's jurisdiction of incorporation** — not stated in the SEC text captured ⚠.
- **What sits behind the "Y" flag on Jump Trading, LLC's BrokerCheck record.** The record carries **15 items typed "Regulatory Event"** ✅, but the endpoint that serves the item text requires authentication (returned **HTTP 403, "Missing Authentication Token"**) ⚠ — so the **count** is recorded and **nothing is asserted about the items**, individually or in aggregate. **"15" is not a count of enforcement actions.**
- **Whether the Fair Fund distribution has been made** — as at the 20 August 2026 notice it had not; the comment period was open ⚠.
- **Whether any of the private civil matters has produced a finding against any Jump entity** — none identified ✅ as a finding about the record examined.

---

## 7. The Capital and the Revenue

*This section follows the capital discipline of [ExodusPoint](exoduspoint_guide.md) §6: state what is published, state what is not, give every figure its kind and its date, and treat the absence itself as the finding rather than as an invitation to estimate.*

### 7.1 No Group Financials

**Jump Trading publishes no financial statements.** There are no group accounts, no revenue figure, no profit figure, no capital figure, no AUM, no valuation, and no headcount figure that can be read from its own site ✅ (verified absence, jumptrading.com and its disclosures page, accessed 24 September 2026). This is not an oversight or a gap in a search: **a private principal trading firm has no obligation to publish any of it**, no listed securities through which a market would price it, no fund vehicle through which a regulator would require it, and no investor base to whom it would owe it.

There is one narrow exception in the group, and it is instructive rather than useful for sizing: **Jump Trading Europe B.V.** is subject to a **Pillar III** disclosure obligation ✅ (§7.3), which is a prudential-disclosure regime and therefore publishes *some* capital and risk information — but only on **written request** from the firm, only about **that entity**, and only under the CRR's own definitions (§7.3). **No Pillar III disclosure for the group, or for the trading parent, exists.**

### 7.2 What the Disclosures Page Does Publish

The firm's disclosures page is the whole of its voluntary corporate disclosure, and it is worth listing in full because what a firm chooses to publish is itself evidence about what it expects to be asked:

| Category | Item published | What it signals |
| --- | --- | --- |
| **Broker-dealer** | Disclosures for **Jump Trading, LLC / Jump Execution, LLC** | The group accepts the dealer-disclosure regime for two entities ✅ |
| **FX / futures** | **FX Global Code** materials; an **Algo Due Diligence Template**; an **FX Global Code Liquidity Provider Disclosure Cover Sheet** | The group expects **counterparties to perform algorithmic due diligence on it** and has built the artefacts for that process ✅ |
| **Prudential (EEA)** | **Pillar III disclosure** for **Jump Trading Europe B.V.**, obtainable **by written request to compliance@jumptrading.com** | A prudentially regulated EEA entity exists in the group; the disclosure is **not** published openly ✅ |
| **UK employment/corporate** | **UK Gender Pay Gap Disclosure 2023, 2024 and 2025**; **Modern Slavery Statement** for **Jump Trading International, Ltd.** | A UK employer with three consecutive years of gender-pay-gap reporting ✅ |
| **Group-level corporate** | **Eightfold AI Recruiting Tool Bias Audit Summary**; **Healthcare Disclosures** for **Jump Trading Group** | An AI-hiring tooling bias audit is a notable, unusual disclosure for a trading firm ✅ |

Three observations. First, **the FX Global Code adherence plus the Algo Due Diligence Template is a deliberate, counterparty-facing posture** — a proprietary trading firm that publishes an algo due-diligence template is telling the market it wants to be traded with, and that it will submit to the questions that go with it. That is directly relevant to §13.4. Second, **the "Bias Audit Summary" and "Healthcare Disclosures" being filed under "Jump Trading Group" is the strongest first-party evidence that the brand is used as a corporate umbrella**, even though its legal status remains unestablished (§2.4) ✅/⚠. Third, **the absence of any financial item from this list is itself the answer to the capital question.**

### 7.3 What the Pillar III Obligation Implies

**Pillar III** is the **disclosure pillar of the CRR/CRD prudential framework** — the Basel-derived regime that applies to credit institutions and to certain investment firms in the EU/EEA. It requires disclosure of capital resources, capital requirements, risk exposures, governance and remuneration, on a published or on-request basis, at the level of the **regulated entity**.

Three things follow from "the firm's own disclosures page says Jump Trading Europe B.V.'s Pillar III disclosure is available on written request" ✅:

1. **There is a prudentially regulated EEA entity in the group.** A Pillar III obligation does not attach to a commercial company; it attaches to an entity inside a prudential regime. That is a materially different corporate fact from "the group has a Dutch office".
2. **The group's closest thing to a capital disclosure is entity-level and not publicly posted.** A bank that needs it must **write to compliance@jumptrading.com** and obtain it — which means the document a counterparty-credit team most wants (an EEA entity's own capital position) is obtainable **only on request, only for that entity, and only under CRR definitions**.
3. **Counterparty classification consequences.** Whether Cymbal Bank's exposure runs to a **prudentially regulated EEA entity**, to a **US registered broker-dealer**, or to a **corporate principal** changes the reporting basis and the risk treatment (§13.5, §14.2). The Pillar III point is what makes the EEA entity legible at all — and what makes "which entity is this?" the first question rather than the last.

**What the Pillar III point does not give you:** any figure for the group, any figure for the trading entity, any revenue, and any statement of the group's equity, leverage or liquidity. **The obligation discloses EU-entity capital adequacy, not group economics** ✅.

### 7.4 The Absence Is the Finding

**No revenue, profit, capital, AUM or valuation figure for Jump Trading was verified this pass** ✅ — and this is the finding, stated as a finding. Two disciplines follow from it, and both are load-bearing for the rest of this guide:

- **A reported figure must carry its outlet and its date, and be labelled reported.** Where a number about a firm of this type appears in circulation and cannot be traced to a named outlet on a named date, it does not enter a file. **No such figure appears in this guide**, because none was established. That is deliberate, and it is consistent with the repository's treatment of the last firm guide in this cluster ([ExodusPoint](exoduspoint_guide.md) §6).
- **The absence is not an inference of trouble.** It is the normal disclosure posture of a private principal trading firm, and the correct analytic response is to find **substitute inputs** (§13.4, §14.3) rather than to read silence as risk. But it *is* a real limit on what a bank can conclude, and pretending otherwise — by adopting a third-party estimate as if it were a disclosure — is the anti-pattern of §15.1(i)–(iv).

### 7.5 The QCP Mis-Attribution — Reconciled

This reconciliation exists because the dispatch brief for this guide asserted something about the repository that is not true, and the error is exactly the kind this repository's claims audits are for. **The brief attributed to [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.4 a Jump Trading scale claim — "over US$1.5 billion in assets deployed (per a 2023 profile)" — and a CEO "Melvin Deng (joined 2021)". Neither is in §4.4.**

| Asserted of Jump Trading by the dispatch brief | What the repository actually says | Where the fact actually lives |
| --- | --- | --- |
| "Over US$1.5 billion in assets deployed, per a 2023 profile" | §4.4 contains **only** founded / products / Singapore presence (lines 286–290) | **§5.2, the QCP Capital entry** — the scale bullet at **line 338**: "over US$1.5 billion in assets deployed (per a 2023 profile)" ✅ |
| CEO "Melvin Deng (joined 2021)" | §4.4 names **no** CEO and no officer of any Jump entity | **§5.2 QCP Capital**, same bullet, line 338 ✅ |
| Anything about Jump's capital | §4.4 states none | **Nowhere in the repository for Jump** ✅ |

**The consequence is a clean finding, and it is stronger than a correction:** *there is no "US$1.5 billion assets deployed" Jump Trading figure in the repository to date or to label.* The figure belongs to **QCP Capital**, a Singapore home-grown digital-asset firm, and so does the CEO attribution. It has almost certainly migrated into circulation about Jump Trading because both are mentioned in the same paragraph of the same guide (the QCP entry itself cross-references "Jump Crypto" at line 338) — **which is precisely how a figure migrates between firms**, and precisely the anti-pattern §15.1(iv) is about. **Do not repeat it as a Jump fact, in either direction.**

### 7.6 The Only Adjacent Repo Datum — Partior

One Jump-adjacent financial datum exists in the repository, and its shape must be stated carefully:

> **Partior's Series B** — **US$60 million first close, July 2024**, led by **Peak XV Partners** with **Valor Capital** and **Jump Trading**; later closing at **US$80 million** with **Deutsche Bank** joining (November 2024). Recorded in [Tokenized Assets](tokenized_assets_guide.md) **line 239** ✅ as published in that guide.

What this is and is not:

- **It is a fundraise participation by Jump Trading.** A named investor in a Series B round. That is a real, dated, reconcilable repository fact ✅.
- **It is not a Jump financial.** A participation in someone else's funding round says nothing about the participant's revenue, capital, AUM or valuation — a firm can invest any amount it likes, disclosed nowhere, into such a round.
- **It is on no account a client, counterparty or clearing-member claim.** Being a co-investor alongside other institutions in a company's share round is not a trading relationship, not a brokerage relationship and not a counterparty relationship. **This guide does not name any bank, broker or exchange as Jump's client, counterparty or clearing member**, and a fundraise participant is the limit of what the Partior source supports ✅ (§13.6).

---

## 8. The Technology and the Latency Posture

*This section is unusually well-sourced by the standards of this genre, and the reason is structural: the firm's own site publishes a technology page, and its CTO is quoted on it. What follows is therefore **first-party, attributed, and dated** — and where the firm has said nothing, this guide says the firm has said nothing, rather than importing a peer's architecture or a vendor's case study.*

### 8.1 The Class of Infrastructure (Cross-Referenced)

A top-tier proprietary trading firm of this type runs a stack that the repository already documents at class level, and this guide does not re-derive it: **the technology-of-the-firm-type treatment is owned by [Citadel LLC](citadel_llc_guide.md) §7**, and **the prop-firm technology treatment — the research/engineering split, low-latency practice, hardware acceleration, compute and machine learning, and the disclosure contrast with peers that publish — is owned by [Hudson River Trading](hudson_river_trading_guide.md) §5**. The **latency discipline itself** — kernel bypass, lock-free data structures, cache-aware layout, NIC and FPGA offload, the measurement practice and the physics of propagation — belongs to the three low-latency guides: [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md), [Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md) and [Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md). **The engineering skill stack behind the roles is owned by [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md)**, and the platform architecture the trading estate belongs to is in [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md).

What is left for this section is the part only this firm can supply: **what it says about itself.**

### 8.2 The Firm's Own Statements — Verbatim and Attributed

**All of the following is from jumptrading.com/technology, accessed 24 September 2026, and every item is the firm's own claim about itself** ✅:

| The firm's own words | What it establishes |
| --- | --- |
| "Jump's technology organization creates the platforms and infrastructure that connect our research, trading, and data efforts around the world. The work spans hardware, software, and distributed systems, from the lowest levels of networking to large-scale compute and simulation environments, as well as the quantitative research platforms that power our trading businesses." | The scope: hardware → software → distributed systems → research platforms |
| "**Three large, purpose-built research data centers and over 100 co-located compute environments near global exchanges.**" | The scale claim — **the firm's own**, and the single most quotable infrastructure fact it publishes |
| "**Custom ASIC, FPGA, and platform development for low-latency and compute-intensive workloads.**" | Custom silicon, named by the firm |
| "**Proprietary network infrastructure optimized for bandwidth, reliability, and efficiency.**" | Proprietary network — the firm's phrase, with **no** route, vendor or technology named |
| "**High-performance software frameworks written in C++, Python, Go, and Rust.**" | The published language set: **four** languages, named |
| "**Storage, simulation, and analytics platforms that power both trading and AI research.**" | Shared platforms across trading and AI research |
| "We're always evolving. Engineers work across a mix of mature and emerging systems with the ability to reimagine how things should be built, not just how they've always been done." | Iteration as a stated posture |

Two readings a bank should take from this table. **First, this is a firm that publishes its own technology story — the opposite of the usual silence in the genre** (§8.5). **Second, and this is the discipline that makes the section usable: the firm names no vendor, no route, no venue product, no latency figure and no system name.** "Custom ASIC, FPGA" is a capability claim; it is **not** a statement about which FPGA family, which NIC, which exchange, which protocol or which latency in nanoseconds ✅. **This guide attributes to the firm only what the firm has stated, and attributes nothing to it that it has not** — no named microwave route, no named network vendor, no specific exchange-connectivity product, no colocation provider, no hardware supplier (§8.5).

The phrase "**Three large, purpose-built research data centers**" is the one worth marking as the firm's own framing rather than a verified count: the firm says three, **it does not say where they are, what is in them, or who operates them** ✅/⚠.

### 8.3 The CTO's Own Words

The technology page carries a quotation attributed to **Alex Davies, CTO** ✅ (jumptrading.com/technology, accessed 24 September 2026):

> "Working in finance isn't what most people imagine. People are often surprised by the engineering we do here. **We build our own hardware, run large-scale AI workloads, and move faster than most tech companies.** You can design something on Monday and see it in production by Friday. That speed and trust are rare."

Three things this quotation does to the guide's thesis. **It is a first-party statement that the firm builds its own hardware** — a claim about vertical integration into silicon, from the firm, via its CTO ✅. **It states that the firm runs large-scale AI workloads** — first-party, no scale given ✅. And **"you can design something on Monday and see it in production by Friday"** is the deployment-cadence claim that, at a firm with no clients, is directly a P&L claim: the loop from idea to live trading is short because nothing sits between engineering and revenue. **The CTO's title and name are the firm's own attribution**, and this guide repeats them as such.

Note also what the page says about the engineering talent market — "You don't have to come from a trading background. Many of our engineers join us from research labs, AI companies, or large-scale software environments" ✅ — which is a hiring-positioning statement carried in §9.2.

### 8.4 Dated Firm Statements After 2024

The firm's own **Signals** page carries dated 2026 items, and two bear directly on its compute posture. **Both are the firm's own publications and are attributed as such** ✅ (jumptrading.com/signals, accessed 24 September 2026):

- **17 March 2026** — "**Jump Trading Becomes One of the First Financial Services Firms to Deploy NVIDIA Vera Rubin Platform in Deep Learning Financial Research Data Center Expansion**", sub-headed "Initiative Builds on Over 15 Years of Collaboration Between Jump and NVIDIA" ✅. This is a **firm-published** statement, dated, naming a **vendor relationship the firm itself discloses**. It matters because it is the one place the firm puts a named technology partner on the record — and the discipline is that **this is attributable precisely because the firm said it**, whereas a named vendor chosen by a journalist or a vendor's own case study would not be (§8.5). This guide does not extend the claim beyond the firm's own words: **no chip count, no cluster size, no performance figure and no vendor part number is asserted.**
- **16 March 2026** — "**Jump Trading Partners with UCL's Gatsby Computational Neuroscience Unit to Fund Four PhD Scholarships**", sub-headed "Supporting Intelligence at Its Foundations" ✅ — a research-funding commitment, carried into §9.3.
- Also on the same page: **ICLR 2026** (the AI/ML team exhibiting at Booth 601, Rio de Janeiro, 23–27 April 2026) ✅; **SREcon Americas 2026** (the infrastructure team, Booth 108, Seattle, 24–26 March 2026) ✅; **NVIDIA GTC 2026** (the AI/ML team, Booth 189 in the FSI Pavilion, San Jose, 16–19 March 2026) ✅. The conference footprint is itself a first-party signal: **the firm sends both its AI/ML team and its infrastructure team to their respective professional conferences, and says so** ✅.

### 8.5 What Is Not Documented

Stated as a list, because a list is what a due-diligence file needs:

- **No latency figure of any kind.** No tick-to-trade, no wire-to-wire, no colocation-to-match number. **The firm has never published one** ✅ (verified absence).
- **No system or platform names.** No order gateway, no matching-adjacent component, no risk engine, no internal platform brand appears in the firm's public material ✅.
- **No network route, no venue product, no named colocation provider.** The firm says "proprietary network infrastructure" ✅ and stops there. **The 2013 Belgian microwave tower reported by Bloomberg (2014) is the only physical-infrastructure item in the record and it is a press report, not a firm statement** ⚠ (§3.3). **This guide asserts no route, no path, no counterparty site and no vendor for it.**
- **No hardware vendor except the NVIDIA relationship the firm itself announced** ✅. Where a vendor publishes a case study about a firm of this type, **the vendor's material is marketing until the firm corroborates it**, and this guide does not treat any vendor page as evidence about this firm.
- **No published engineering blog, no open-source release and no architecture talk identified** ✅/⚠. Where the peer set in the repository publishes engineering writing, **this firm's public technical output is its own careers-and-capability copy** — which is more than some peers publish and less than an engineering blog. **The absence of an open-source release was a single-pass finding, and absence of evidence in one pass is not proof of absence** ⚠ (§16.1).
- **No compute-sharing arrangement disclosed.** The firm describes research data centres and co-located compute (§8.2) and says nothing about shared, cloud or outsourced compute at scale, beyond the AI-workload statement. **This guide asserts no cloud provider and no compute-sharing arrangement.**

The right summary of §8 is that **this firm's technology posture is unusually legible at the level of intent and completely opaque at the level of implementation** — and that the legibility is a recruiting decision, not a disclosure decision. A firm with no clients publishes its engineering story because it is competing for engineers, not for mandates.

---

## 9. Talent and Culture

### 9.1 The Resourcing Model (Cross-Referenced)

The resourcing model for a firm of this class — the role taxonomy (quantitative researcher, algorithm developer, low-latency engineer, FPGA/silicon engineer, trading operations), the recruiting funnel (internships and new-grad pipelines, university relationships, the compensation reputation, the retention and attrition picture), and the culture signals — is owned by [Hudson River Trading](hudson_river_trading_guide.md) §6 and is **not re-derived here**. The engineering skill stack those roles require is owned by [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md). What this section adds is what **this firm itself says publicly**, which is more than most of its peers say.

### 9.2 What the Firm Says Publicly

All of the following is the firm's own language, accessed 24 September 2026 ✅:

| The firm's own words | Where |
| --- | --- |
| "Jump is where **traders, engineers, and researchers** solve hard problems. **Every asset class. Every time horizon. No boundaries.**" | jumptrading.com home |
| "**We go all in on talent.** We build teams with purpose, keep the bar high, and the focus sharp. **Cross-disciplinary teams** work from first principles to develop models, strategies, and systems that stand up to real-world conditions." | jumptrading.com home |
| "Our culture is built on **autonomy, speed, and trust**. Engineers here have **ownership from day one**. You design, test, and deploy solutions in production and see the results almost instantly." | jumptrading.com/technology |
| "**Those who succeed here are builders at heart.** People who like to dig into complex systems, learn how they work, and make them better will thrive in our environment that rewards **precision, curiosity, and independence**." | jumptrading.com/technology |
| "Engineers at Jump have a level of **visibility across the entire technical stack** that's rare in most large-scale organizations. … **curiosity doesn't clock out.**" | jumptrading.com/technology |
| "You don't have to come from a trading background. Many of our engineers join us from **research labs, AI companies, or large-scale software environments**." | jumptrading.com/technology |

Three signals a reader should take from this: **cross-disciplinary teams and full-stack visibility are stated as the differentiator** (a claim that makes sense at a firm with no client-facing division to silo against); **ownership-from-day-one and immediate production deployment are stated as the operating model** (consistent with the CTO's "Monday → Friday" claim, §8.3); and **the firm explicitly recruits from outside trading** — research labs, AI companies, large-scale software — which is a recruiting posture that tracks its ML/AI positioning (§9.4).

The firm's careers page also frames two entry paths — **"Experienced Candidates"** ("Explore our full-time roles and how we support high-agency professionals") and **"Students & New Grads"** (internships and early-career roles) — plus a **"Talent Community"** for speculative applications ✅ (jumptrading.com/careers, accessed 24 Sep 2026).

### 9.3 The Programmes

This firm runs an unusual number of publicly named research programmes for a proprietary trading firm, all first-party and dated ✅ (jumptrading.com and its Signals page, accessed 24 September 2026):

| Programme | What it is | Date on the firm's own page |
| --- | --- | --- |
| **Fellowship Program** | A named fellowship programme on the firm's own "Research & Programs" navigation | ✅ listed |
| **Conference Travel Grants** | Grants supporting conference attendance, listed alongside the fellowship | ✅ listed |
| **2026–2027 Jump Trading Fellows** | Announcement — the firm's own page says "**We meant to select six fellows this year. We selected eight.**" | **23 April 2026** ✅ |
| **UCL Gatsby Computational Neuroscience Unit partnership** | Funding **four PhD scholarships**, under the heading "Supporting Intelligence at Its Foundations" | **16 March 2026** ✅ |
| **Jump Trading Probability Cup** | "A forecasting competition for people who think in probabilities" | **10 June 2026** ✅ |

Two readings. **The programmes are a recruiting and brand instrument, and they are concentrated on AI/ML and quantitative reasoning** — the fellowship, the computational-neuroscience PhD funding and the forecasting competition are all upstream investments in the same talent pool the firm hires from. And **"We meant to select six fellows this year. We selected eight"** is a first-party statement of programme **growth**, which is the closest thing to a talent-investment datum the firm publishes ✅ (it is not a headcount figure, and this guide does not treat it as one).

### 9.4 The ML/AI Positioning

The firm positions AI/ML as **infrastructure** rather than as a research project, which is a distinction worth stating precisely:

- The home page's own section header: "**Built to learn. Designed to trade.** We build where infrastructure meets intelligence. **Our ML stack powers live inference and fast iteration**, delivering seamless edge when it matters most." ✅
- The technology page describes "**applied AI/ML**" as one of the things technology powers, and describes platforms that "power both **trading and AI research**" ✅.
- The CTO states the firm "**run[s] large-scale AI workloads**" ✅ (§8.3).
- The firm's 2026 conference footprint includes **ICLR** and **NVIDIA GTC** with the AI/ML team named ✅ (§8.4).

**"Live inference"** is the load-bearing phrase: it is the claim that ML models are **serving in production trading**, not only in research — which, at a firm with no clients, is the whole point (§1.1). **No model, no scale, no infrastructure size and no performance figure is claimed by this guide**, because the firm claims none ✅.

### 9.5 The People Record — What Is Verified and What Is Press

- **Verified, first-party:** **Alex Davies, CTO** ✅ (quoted on the firm's technology page, accessed 24 Sep 2026); **Sach Chitnis** and **Mike McMahon**, Co-Founders and Partners of Jump Capital ✅ (jumpcap.com team page); eleven further Jump Capital team members by name and title ✅ (§5.2); **Bill DiSomma**, **Paul Gurinas** and **Matt Schrecengost** as Jump Trading attendees at the CFTC meeting of 2 November 2010 ✅; **Kanav Kariya** as Jump Crypto's named head from the 2021 launch ✅/⚠ and as the departing president in June 2024 ⚠.
- **Press, flagged:** the reported **April 2026** Business Insider item that two long-tenured researchers, **Yiming Zhang** (joined 2009) and **Darko Kirovski** (joined 2011), had left the firm ⚠. **Recorded here as reported; not verified at a primary source; not used as a basis for any conclusion about the firm's research capability.**
- **Not established at all:** the firm's **headcount** (the firm's own site renders counters as animated digits that do not extract ✅ as a technical limitation; Wikipedia's "over 2,000 employees" is ⚠ secondary ⚠); compensation structure, equity or profit-sharing arrangements; attrition or tenure data; the size of any office team; and the current head of Jump Crypto ⚠ (§16.1). **No compensation figure is quoted anywhere in this guide**, because none was established.

---

## 10. The Regulatory and Market-Structure Context

### 10.1 Where a Proprietary Firm Sits in the Debate (Cross-Referenced)

The class-level treatment of the market-structure debate as it applies to firms of this type — the post-2010 policy wave, the industry counter-position, the question of what non-bank liquidity provision does in stress, and the specifics of market-access and pre-trade-risk rules — is owned by **[Hudson River Trading](hudson_river_trading_guide.md) §7 and §8**, and the exchange and market-maker incentive programmes are owned by its **§7.3**. **This section does not re-derive any of it.** What it does is state where **this** firm sits, and the honest answer has a shape worth naming: **this firm's public posture in the debate is almost entirely a matter of its conduct and its filings rather than of its public statements.** It is not a co-founder of a lobbying group of the kind the repository documents for a peer; it publishes no position papers; and the one primary document in which it engages the substance of market structure is a **2010 meeting record about the definition of spoofing** (§10.4). Beyond that, the evidence about its posture is the conduct record of §6.

### 10.2 The Designation and Registration Questions

A firm of this type raises a specific set of regulatory classification questions, and the repository's convention is to name them rather than to assume an answer:

| Question | What applies to a firm of this type | This firm's documented position |
| --- | --- | --- |
| **Market access and pre-trade risk** | A broker-dealer with market access must maintain risk-management controls and supervisory procedures reasonably designed to prevent erroneous orders and orders exceeding pre-set capital thresholds (**Exchange Act Rule 15c3-5**), with a control-person certification | ✅ This firm has been **found in violation of Rules 15c3-5(b) and 15c3-5(c)(1)** — the NYSE Chicago order of §6.4 is the concrete record, and it is the single most useful document in this guide for a bank's market-access due diligence |
| **Net capital** | **Exchange Act Rule 15c3-1** applies to the registered broker-dealer | ✅ The same order found a violation of 15c3-1 (and NYSE Chicago Art. 7 R.3(a)(1)(A)) — a self-reported violation lasting several hours on 4 May 2018 |
| **SEF registration** | Swap execution facilities are registered venues; a firm of this type may be a **participant** on them or a member of venue programmes | ✅ The firm engaged the CFTC in 2010 on "**SEF Registration**" and "**transparency and access to SEFs**" as listed rulemakings ⚠ — engagement, not a registration claim; **this guide establishes no SEF registration, membership or participation for this firm** |
| **DMM-type programmes** | Exchange designated-market-maker schemes carry quoting obligations in exchange for incentives; the repository documents the SGX DMM regime for the Singapore market in the sibling guide §3 | **Not established for this firm.** No DMM or equivalent designation was identified, and **none is claimed** ⚠ |
| **The principal-trader category** | Firms trading their own capital are treated as a distinct class in policy discussion, and the FIA's **Principal Traders Group** is the industry vehicle for them | ✅/⚠ See §10.3 |
| **EEA prudential perimeter** | Certain investment firms fall inside a CRR/CRD prudential regime, with **Pillar III** disclosure | ✅ **Jump Trading Europe B.V.** publishes a Pillar III disclosure on written request — the group's one prudential-perimeter presence (§7.3) |
| **UK employer and corporate obligations** | Gender-pay-gap reporting; Modern Slavery statements | ✅ **Jump Trading International, Ltd.** files both, three years of the former (§7.2) |
| **US broker-dealer registration** | FINRA registration with CRD and SEC numbers | ✅ Two entities: Jump Trading, LLC and Jump Execution, LLC (§2.3) |

The pattern the table shows is the standard one for this firm type and it is worth naming: **a proprietary trading firm is not unregulated — it is regulated on the conduct and dealer side, and absent from the prudential side**, except where a group entity sits inside a prudential regime (here, one Dutch B.V.). That is the classification fact a bank's own regulatory-reporting consequences turn on (§13.5, §14.2).

### 10.3 The Principal Traders Group

The **FIA Principal Traders Group (PTG)** is the Futures Industry Association's policy group for firms that trade their own capital — the industry vehicle through which firms of this class engage rulemaking on market structure, market access, and the treatment of proprietary traders. The repository records membership of the PTG as a class fact for the peer set ✅ ([Hudson River Trading](hudson_river_trading_guide.md) §1.2 and §7.1, where the FIA PTG membership list names the peer firm). **This guide carries this firm's PTG membership as a class fact flagged ⚠**, because **the membership list was not re-read this pass** and a membership claim about a named firm should rest on the list itself, not on the class pattern. **If the claim is to be used in a file, it should be verified against the FIA's own list first** ⚠ (§16.1).

What is safe to say without the list: **this firm belongs to the same policy class as the peer set in [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4 and §5**, and the peer guides' treatment of the principal-trader policy position applies to it by type. **This guide asserts no specific policy position, filing, comment letter or lobbying activity by this firm**, because none was identified ✅ (verified absence).

### 10.4 Spoofing, 2010

The firm's **only primary document of substantive market-structure engagement** is the CFTC record of **2 November 2010**, when CFTC staff (**Gary Gensler** and **Rosemary Hollinger**) met **Matt Schrecengost, Bill DiSomma and Paul Gurinas** at the firm's offices and discussed, after high-frequency trading and the events of 6 May 2010, "**the definition of spoofing as a disruptive trade practice**" and **transparency and access to SEFs**, against the listed rulemakings **XIII. SEF Registration** and **XXIV. Disruptive Trading Practices** ✅ (CFTC External Meetings record).

Three things this dates and documents, stated precisely because the distinction is easy to blur:

- **The firm engaged the regulator on the definition of a practice, not as its subject.** The record is a **meeting**, and **the record shows no enforcement action**. Any suggestion that this firm was investigated for spoofing in 2010 would be a fabrication, and this guide makes no such suggestion ✅.
- **The engagement was early and substantive** — contemporaneous with the rulemaking that followed the flash crash, on the two questions that would define the post-2010 market-structure settlement (what counts as a disruptive trading practice, and who gets to access swap venues). That is a fact about the firm's posture worth recording, and it is the strongest basis for describing the firm as a policy participant from 2010 ✅.
- **It is a matter concerning the trading firm.** The three attendees are identified in the record as "Jump Trading" ✅ — this is **not** a crypto-arm matter and must not be folded into the digital-asset narrative.

### 10.5 Conduct, Not Statements

The honest conclusion of this section, and the reason it is short: **this firm's position in the market-structure debate is legible from what it does, not from what it says.** It does not publish position papers, it is not identified as the co-founder of a trading-industry lobbying group, and it made no public comment identified in this pass on the market-structure questions that concern firms of its class ✅ (verified absence). What the record shows instead is: **a self-reported control failure and the sanction that followed** (§6.4); **a US$123 million settlement by its crypto affiliate** (§6.2); **a 2010 meeting at which its principals engaged the regulator on the definition of spoofing** (§10.4); and **a registered broker-dealer with no relevant disciplinary history as the exchange recorded it in the NYSE Chicago order** ✅. That is a conduct record, and it is a materially more useful thing for a bank to hold than a policy position — which is why §13.1 treats the conduct record as the substitute for the prudential one.

---

## 11. The Asia and Singapore Angle

### 11.1 What the Sibling Guide Already Owns

**[Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.4 owns the Jump Trading Singapore record**, and this guide cross-references it by name rather than restating it. §4.4's three bullets are: **Founded** — 1999, Chicago, by former CME pit traders Paul Gurinas and Bill DiSomma (who met in the Deutsche Mark pit), HQ Chicago ✅ (Wikipedia); **Products** — HFT and algorithmic trading across futures, options, equities and digital assets (Jump Crypto); heavy investment in low-latency infrastructure including microwave links ✅ (Wikipedia; tradermath.org) — note that the microwave-link element there rests on a secondary source and is **flagged** in this guide accordingly (§8.5); and **Singapore presence** — **Jump Trading Pacific Pte. Ltd. incorporated 15 March 2011**, at **Asia Square Tower 1, 8 Marina View** ✅ (ACRA via opengovsg.com and sgpbusiness.com). The same guide's §6 comparison table carries a Jump Trading row: founded **1999**, **Chicago**, products futures/options/equities/crypto, Singapore footprint **"Jump Trading Pacific Pte Ltd, incorporated 15 Mar 2011"**, notable **Jump Crypto; microwave/laser low-latency infrastructure** ✅/⚠. A second repository location records a unit-number detail: [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) line 319 records **"Jump Trading Pacific at #38-01"** of Asia Square Tower 1, alongside **DRW Singapore at #32-05** ✅ as published in that guide.

**This guide does not re-derive the Singapore market-making landscape** — the SGX DMM schemes, the MAS CMS overlay, the CDP/SGX-DC infrastructure and the Singapore peer set all belong to the sibling guide's §3, §7 and §8 — and it does not restate the §4 landscape.

### 11.2 What This Guide Adds

Only what is independently verified, and it is a short list:

1. **Singapore is on the firm's own current office list** ✅ (jumptrading.com, accessed 24 September 2026) — which is **primary, first-party evidence that Singapore exists as a location for the firm today**, in the firm's own framing. The full named list is **Chicago, New York, Austin, London, Singapore, Shanghai, Bristol, Mumbai, GIFT City, Sydney, Amsterdam, Hong Kong, Paris** (thirteen).
2. **The Asia-Pacific set on the firm's own list** ✅: **Singapore, Shanghai, Mumbai, GIFT City (Gujarat International Finance Tec-City), Hong Kong** — plus **Sydney** among the Asia-Pacific locations. This is a wider Asia footprint than most of the peer set documented in the sibling guide's §4 and §5 carries, and it is **the firm's own list, not a third-party compilation**.
3. **The firm's own site does not state the Singapore office's establishment year** ✅ (verified absence). **The 2011 incorporation date rests entirely on ACRA**, and the two facts — an office that exists today and a company incorporated in 2011 — are **different kinds of fact from different sources** and should never be collapsed into one sentence.
4. **The firm's own site says nothing about Singapore's function.** No Singapore-specific hiring page, no Singapore-desk description, no MAS licence, no Singapore entity name appears in the firm's own material examined ✅/⚠. **This guide therefore claims no MAS licence for any Jump entity**, establishes no Singapore-regulated entity, and does not assert what the Singapore office does beyond the fact that it is on the office list ⚠ (§16.1).

### 11.3 The Two Errors in the Dispatch Brief — Reconciled

The dispatch brief for this guide attributed two facts to **Jump Trading** by way of [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.4. **Both attributions are errors, and both are corrected here in the terms a filing needs.**

| The brief's claim about Jump Trading | The repository's actual content | The correct attribution |
| --- | --- | --- |
| A scale claim of **"over US$1.5 billion in assets deployed (per a 2023 profile)"** | §4.4 contains **only** founded / products / Singapore presence, at lines 286–290. It states **no scale, no AUM and no capital figure of any kind** | **§5.2, the QCP Capital entry, line 338** — a **Singapore home-grown digital-asset firm**, not Jump Trading |
| A CEO **"Melvin Deng (joined 2021)"** | §4.4 names **no officer of any Jump entity** | **§5.2 QCP Capital, line 338** — Melvin Deng is **QCP Capital's** CEO, not Jump's |
| Anything about Jump's Singapore capital or leadership | §4.4 has no such content | **Nothing** — and this guide establishes no Jump Singapore capital figure and no Jump Singapore officer |

**The finding, stated positively:** **§4.4 contains only founded / products / Singapore presence.** There is **no Jump Trading scale figure in the repository to date, label or reconcile**, and the $1.5 billion figure and the CEO attribution belong to a different firm altogether. The mechanism of the error is worth naming in a guide that has a §15 anti-pattern for it: **the QCP entry at §5.2 explicitly cross-references "Jump Crypto" in its own text**, so the two firms sit in the same paragraph of the same guide, and a figure lifted out of that paragraph acquires the wrong subject. **Nothing in this guide repeats either claim about Jump Trading, and the dispatch brief's version of §4.4 should be treated as erroneous.** ✅

### 11.4 What Is Not Established

- **Whether Jump Trading Pacific Pte. Ltd. remains a live Singapore company today** — the ACRA incorporation record of 15 March 2011 is in the repository ✅; **no current-status check of the Singapore company was performed this pass** ⚠. **The firm's own office list confirms Singapore as a location, not the company's status** ✅ — and those are not the same claim (§2.4).
- **Whether any Jump entity holds a MAS licence** — not established; **no MAS Financial Institutions Directory entry for any Jump entity was located this pass** ⚠. This is an **open item, not a negative finding**.
- **What the Singapore office does** — not disclosed by the firm ✅/⚠.
- **The Singapore office headcount** — not disclosed ✅.
- **Whether the address on the ACRA record (Asia Square Tower 1, 8 Marina View) is the firm's current Singapore address** — the ACRA record and the sibling guide's unit detail are the only sources; the firm's own site names **Singapore** without an address examined this pass ✅/⚠.
- **The establishment year of any other Asia office** — the firm names the locations and no dates ✅.

---

## 12. The Peer Group and Positioning

### 12.1 The Peer Set

The peer set for this firm is documented **with dates and sources** in [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) **§4 and §5**, and this guide names the firms and cross-references rather than re-deriving the entries: **Jane Street** (incorporated 31 August 1999, New York; its Singapore office's establishment year is flagged as not disclosed in that guide), **Susquehanna (SIG)**, **Tower Research Capital**, **Hudson River Trading** (2002, New York), **XTX Markets** (2015, London; a foreign-exchange pricing engine set up in Singapore in conjunction with MAS in 2018), **DRW** (1992, Chicago; crypto subsidiary **Cumberland**), **IMC**, **Optiver** (the sibling guide's reference firm), **Flow Traders**, plus the repository's Asia names — **AlphaGrep**, **QCP Capital**, **Cumberland**. The hedge-fund cluster is named separately for contrast in §12.3.

What the peer set shares, and what this firm shares with it: **principal trading of the firms' own capital, a technology-first operating model, a low public disclosure profile, and a market-making and liquidity-provision franchise.** What differentiates within the set is **strategy breadth and horizon** — and on that axis this firm's own self-description ("high-frequency to stat arb to discretionary macro") places it, by its own claim, among the **broader-horizon** members of the group rather than the pure speed shops ✅/⚠ (a self-description, not a verified book).

### 12.2 Against a Bank's Markets Business

The comparison a bank's own staff most often get wrong, because the two firms can look superficially similar — both quote prices in size, both run multi-asset books, both employ quants:

| Dimension | A bank's markets business | A firm of this type |
| --- | --- | --- |
| **Whose capital** | The bank's capital, on top of a deposit and wholesale-funding base | **Its own capital**, funded from its own equity and retained earnings ✅ |
| **Client flow** | The franchise: client orders are the business, and market making is often an accommodation to them | **No clients at all.** The firm "trades solely in a proprietary capacity" ✅ (NYSE Chicago order) |
| **Regulatory perimeter** | Prudential (capital, liquidity, leverage, resolution), conduct, depositor protection, supervisor-appointed safety-and-soundness oversight | **Conduct and dealer regulation only** (Rule 15c3-5, 15c3-1, supervision), plus an EEA prudential entity where one exists in the group ✅ — **no group-level prudential regime** |
| **Disclosure** | Audited financial statements, prudential returns, Pillar III, published risk disclosures | **None of it.** No group financials at all ✅ (§7) |
| **Failure mode that concerns a counterparty** | Balance-sheet, funding and capital adequacy | **Operational**: a control failure, a software incident, a supervisory lapse — precisely the class the NYSE Chicago order documents ✅ |

The practical point is not that one is safer than the other; it is that **the two require different underwriting instruments**, and a file that treats a proprietary trading firm as if it were a bank-shaped counterparty with missing data will ask for the wrong documents and draw the wrong conclusions (§14.3).

### 12.3 Against a Multi-Manager Hedge Fund

The second comparison, and the one that catches out onboarding teams, is with the **multi-manager hedge-fund platform** — the firm type owned by the hedge-fund cluster in this repository: **[Hedge Funds in Singapore](hedge_funds_singapore_guide.md)** (the Singapore manager landscape and the RFMC/LFMC/CMS regime), **[Hedge Fund Guide](hedge_fund_guide.md)** (the platform model, fund economics and banking relationships) and **[ExodusPoint](exoduspoint_guide.md)** (the worked multi-manager case, whose §6 documents the capital-reporting problem for a private manager and whose §13 is its own Cymbal Bank worked example). **The contrast is worth stating because the two firm types converge in their disclosure posture and diverge in every structural dimension that matters:**

| Dimension | A multi-manager hedge-fund platform | A firm of this type |
| --- | --- | --- |
| **Whose capital** | **Investors'** capital, in fund vehicles, with fees, hurdles and redemption terms | **Its own** capital, with no investors and no redemption right |
| **Regulatory disclosure the vehicle produces** | A Form ADV reports **regulatory AUM, employee counts, fund lists and jurisdictions** — which is why the ExodusPoint guide can quote a US$159.4bn regulatory figure and 707 employees ✅ | **Nothing of the kind.** No ADV (the entities are broker-dealers, not advisers), no fund vehicles, no reported AUM ✅ |
| **How a bank sizes it** | Net investor capital from fund documents, plus the regulatory figure as a portfolio gross measure — kept deliberately apart ([ExodusPoint](exoduspoint_guide.md) §6.2) | **It cannot be sized from public sources at all** ✅ (§7.4) |
| **Performance** | Reported by press from investor documents; the firm publishes none ([ExodusPoint](exoduspoint_guide.md) §7) | **No performance record exists publicly**, and there is no investor document to leak it ✅ |
| **The relationship a bank has** | Prime brokerage, financing, custody, FX — a **client** relationship to a fund | **Counterparty, liquidity provider, and clearing client of a dealer**, never a fund client (§13) |

**The single sentence worth carrying:** a multi-manager fund is opaque about performance but *legible in structure* (filing-mandated AUM and headcount); a proprietary trading firm is **opaque in both** — which is why §13's substitute inputs matter so much and why §7's absence is the section most likely to be got wrong.

### 12.4 This Firm's Relative Position

Stated only as far as sources support, which is the repository's rule for this genre and is especially binding here:

- **Documented and firm-sourced:** founded **1999**; **13 named office locations** across the United States, Europe and Asia-Pacific ✅; six traded asset classes and a high-frequency-to-discretionary-macro strategy span, in the firm's own words ✅; a **2021-vintage digital-asset arm** that is publishing research in 2026 ✅; custom ASIC and FPGA development and **three purpose-built research data centres with over 100 co-located compute environments** per the firm's own technology page ✅.
- **Documented from regulators or the firm:** two US broker-dealer registrations, both ACTIVE ✅; a **NYSE Chicago participation since 2000** ✅; an EEA entity with a Pillar III obligation and a UK entity with three years of gender-pay-gap filings ✅; and the settled matters of §6 — the largest of which is a **US$123,095,287** SEC order against its crypto affiliate ✅.
- **Not established, and therefore not asserted:** the firm's **size in any financial dimension** ✅; its **headcount** ⚠; any **market-share or volume statistic** ✅ (none exist for this firm in the sources examined); its **ranking against any named peer** — the repository does not rank firms without a common, dated measure, and none exists here; and any **profitability or capital comparison** to Jane Street, Citadel Securities, HRT or any of the names in §12.1, which the sibling guides document with revenue figures this guide has **no equivalent for**. **Where the peer set has a revenue figure and this firm has none, the honest comparison is not a comparison — it is a statement of what is not known.**
- **One reported item, flagged and not relied upon:** crypto-press reports in late 2025/2026 that the firm had **doubled a prediction-markets team to about 20 people** and holds equity in prediction-market venues ⚠. Recorded as reported. **This guide draws no positioning conclusion from it** (§16.1).

---

## 13. The Bank Interface

*This section is the shape of [Hudson River Trading](hudson_river_trading_guide.md) §10 at mechanism level, cross-referenced: that guide's §10.2 (prime brokerage and clearing), §10.5 (margin, collateral and limit mechanics) and §10.7 (regulatory-reporting consequences) carry the mechanics for a firm of this type, and the Singapore clearing and settlement infrastructure is owned by [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §7. What follows is what is specific to **this** firm: its entity structure, its own due-diligence artefacts, and the conduct record that substitutes for the prudential one.*

### 13.1 The Firm as Counterparty

A bank's exposure to a firm of this type runs through **clearing and margin**, and the governing rules are the ones that apply to the firm's **broker-dealer arms** rather than to the group:

- **Clearing and margin mechanics** are not re-derived here (see the cross-references above). What matters for this firm is the **entity**: an exposure to **Jump Trading, LLC** is an exposure to a **US registered broker-dealer** with a **NYSE Chicago participation since 2000** and a **disclosure flag on its BrokerCheck record** ✅ (§2.3); an exposure to **Jump Execution, LLC** is an exposure to a **separate registered broker-dealer carrying the other name "Jump Liquidity"** ✅; an exposure to **Jump Trading Europe B.V.** is an exposure to an **EEA entity with a Pillar III obligation** ✅; an exposure to **Jump Trading Futures, LLC** is an exposure to the **FX Global Code-adhering futures entity** ✅. **Which entity bears the risk is the first control, and the names are confusable by design** (§2.1).
- **The net-capital and market-access rules that govern the broker-dealer arm are the rules that produce the bank's own obligations.** Where a bank extends market access or acts as clearing member to a firm of this type, the **pre-trade risk controls, erroneous-order limits and control-person certification** of Exchange Act Rule 15c3-5 flow through the relationship, and the SEC's market-access rule is not a formality: it is the rule **this firm was found in violation of** ✅.
- **The NYSE Chicago order is the concrete evidence of why a bank cares about a counterparty's pre-trade controls.** The two control failures the exchange found are the exact two a bank's own market-access due-diligence checklist tests for: **a limit that reduces order size rather than blocking it**, and **a capital limit that counts executed rather than placed orders** ✅ (§6.4). A bank drafting pre-trade risk requirements for a counterparty of this type can cite the regulator's own findings as the specification — that is the single most practically useful sentence in this guide.
- **The conduct record as the governance proxy.** Because there is no prudential supervisor publishing safety-and-soundness assessments of this firm, the **enforcement record is the substitute**, and its shape is informative: one **self-reported** control incident with a **US$250,000** fine and a **90-day certification undertaking**, in a firm the exchange recorded as having **no relevant disciplinary history** ✅; and, on the **digital-asset affiliate** rather than the trading firm, a **US$123,095,287** settled order entered **without admitting or denying** ✅. A bank should read those two as **different in kind and different in entity** (§5.4) — operational-control in one case, registration-and-negligent-conduct in the other, and **neither of them a market-integrity finding against Jump Trading, LLC** ✅.

### 13.2 The Firm as Liquidity Provider

Where a bank sources external liquidity — for its own execution, for a client-facing platform, or for a product that streams prices — a firm of this type can appear as a **non-bank liquidity provider**, and the bank's controls shift accordingly:

- **The bank's price is being shown to a professional.** A firm whose own stated capability includes "high-frequency" and custom silicon ✅ is a counterparty built to detect stale pricing. The practical controls are the ones the repository documents for the counterparty class: **last-look versus firm pricing, fill ratios in stress, rejection rates, and mark-out analysis by counterparty** — so that the cost of interacting with a specific liquidity provider is a **measured input** rather than an anecdote (the pattern is set out at [Hudson River Trading](hudson_river_trading_guide.md) §10.3–§10.4 and is not re-derived here).
- **The firm's own liquidity-provision posture is documented by the firm — unusually.** **Jump Trading Futures, LLC** publishes both an **Algo Due Diligence Template** and an **FX Global Code Liquidity Provider Disclosure Cover Sheet** ✅ (§7.2). Those two artefacts are what a bank's electronic-trading risk function would otherwise have to extract by negotiation; here the firm publishes the frame.
- **Concentration is the risk that is easy to miss.** If a bank's streaming-liquidity panel includes one firm of this type at a material share, the panel's depth is a function of that firm's own risk appetite — which is **unobservable** ✅, and which is why the limit architecture in §14.5 treats panel concentration as a distinct limit from counterparty credit.
- **Stress behaviour is not contracted.** Nothing obliges a proprietary firm trading its own capital to keep quoting when its own limits bind; the class treatment of that point is at [Hudson River Trading](hudson_river_trading_guide.md) §8.3 and it applies here directly ✅.

### 13.3 The Firm as a Client of Prime and Clearing Services

The direction banks model least well: a firm of this type buys **clearing, financing, FX and asset-servicing** services, and it buys them **as a principal trading firm rather than as a fund**. The product set is the one documented at [Hudson River Trading](hudson_river_trading_guide.md) §10.2, and the firm-specific points are these:

- **The requirement is driven by inventory and by multi-venue membership, not by client flow.** A firm with exchange memberships of its own still needs intermediation elsewhere, plus give-up and allocation flows across venues ✅/⚠ — the registered-footprint evidence is at §2.3.
- **The firm has no investor base to report to**, so there is no fund administrator, no NAV, no subscription/redemption machinery and no investor reporting — which removes a whole category of the documentation a bank would expect from a fund client ✅.
- **The firm's own site's "Fraud & Scam Alert" page is operationally relevant to this relationship.** A group whose name is impersonated publishes a warning about it ✅; for a bank, that is a prompt to run **identity verification on the contracting entity** rather than on the brand — a control that also addresses the "AKAMAI TRADING LLC" and "Jump Liquidity" other names on the registers ✅ (§2.3).
- **Financing appetite is unobservable, and therefore cannot be planned for from public sources.** With no financial statements, a bank cannot size the firm's financing need from the outside — it must be obtained from the client (§13.5).

### 13.4 The Artefacts a Bank Can Actually Obtain

This is the section that makes the relationship workable, because **this firm publishes more counterparty-facing documentation than is typical for the genre**. The verifiable list, all first-party ✅ (jumptrading.com/disclosures, accessed 24 September 2026):

| Artefact | Why a bank wants it | How to get it |
| --- | --- | --- |
| **Algo Due Diligence Template** (Jump Trading Futures, LLC) | The framework for the algorithmic due-diligence questionnaire the bank must complete for any algo counterparty — the firm publishes the frame the bank would otherwise draft | Published on the firm's disclosures page ✅ |
| **FX Global Code Liquidity Provider Disclosure Cover Sheet** (Jump Trading Futures, LLC) | The FX Global Code's own liquidity-provider disclosure cover sheet — the standardised document that commits a provider to the Code's disclosure expectations, and therefore a **conduct commitment a bank can point to** | Published on the firm's disclosures page ✅ |
| **FX Global Code adherence** (Jump Trading Futures, LLC) | Adherence to the Global Foreign Exchange Committee's Code of Conduct — a public commitment on FX execution conduct, last look, and information handling | Published on the firm's disclosures page ✅ |
| **Pillar III disclosure** (Jump Trading Europe B.V.) | The **only** capital and risk disclosure the group makes — entity-level, CRR-defined, and the closest thing to a capital document available | **By written request to compliance@jumptrading.com** ✅ |
| **Broker-dealer disclosures** (Jump Trading, LLC / Jump Execution, LLC) | The dealer-side disclosures, and the register entry behind them (BrokerCheck CRD 106124 / 313060) | Published on the disclosures page ✅; register at FINRA BrokerCheck ✅ |
| **UK Gender Pay Gap Disclosure 2023, 2024 and 2025; Modern Slavery Statement** (Jump Trading International, Ltd.) | Corporate-conduct and governance evidence, dated across three reporting years — useful as a **corporate-governance datapoint for an entity the bank may contract with in the UK** | Published on the disclosures page ✅ |
| **Eightfold AI Recruiting Tool Bias Audit Summary** (Jump Trading Group) | An unusual governance artefact: an independent bias audit of an AI hiring tool, published. Relevant to the bank's own AI-governance file if it treats counterparty AI governance as in scope | Published on the disclosures page ✅ |
| **The enforcement record itself** (SEC; NYSE Chicago) | The **substitute** for a prudential assessment: the conduct record as governance evidence, with authority, instrument and date | **Public**: SEC PR 2024-212 and Admin. Proc. File No. 3-22382; NYSE Chicago Proceeding No. 2018-11-00017 ✅ |
| **The entity registers** (FINRA BrokerCheck) | Entity identification, other names, status | Public ✅ (§2.3) |

**What a bank cannot obtain by any of these routes:** group financial statements, group revenue or profit, group capital, headcount, the ownership chain, or any statement of the trading book's size ✅ (§7.4). Those must come from the client directly, and a bank that needs them must say so in the negotiation.

### 13.5 The Counterparty-Credit Questions

The questions a bank asks about a firm that discloses no financials, in the order a credit file needs them, with the honest state of each answer:

| # | The question | The substitute input available | Status |
| --- | --- | --- | --- |
| 1 | **Which legal entity is the counterparty, and what owns it?** | The entity table of §2.1 plus the registers of §2.3 — **but the ownership chain is not established** | Partial ⚠ |
| 2 | **How is the entity capitalised?** | **Pillar III for the EEA entity only**, on written request ✅; nothing for the group or the US entities | Partial ⚠ |
| 3 | **What does it earn?** | Nothing. No revenue, profit or earnings history exists publicly | ❌ |
| 4 | **What is the book, and how volatile is it?** | Nothing disclosed; the asset-class list and strategy spectrum are self-descriptions only | ❌ |
| 5 | **Is it supervised for safety and soundness?** | No. Conduct-and-dealer regulation only, except the EEA entity's prudential regime | ❌ / Partial |
| 6 | **How has it behaved under failure?** | **The enforcement record** — a self-reported control failure and its remediation undertaking; the disclosure flag on the register | ✅ |
| 7 | **What are its conduct commitments?** | **FX Global Code adherence**, the **Liquidity Provider Disclosure Cover Sheet**, and the **Algo Due Diligence Template** | ✅ |
| 8 | **Is it a going concern with continuity?** | Dated first-party evidence of current activity (technology statements, 2026 publications and programmes, senior hires named) ✅; the register statuses ✅ | ✅ as to activity |
| 9 | **What is the concentration and correlation of my exposure?** | Derivable: exposure by entity, by product, by clearing member relationship, and by liquidity-panel share | Bank-internal |
| 10 | **What happens on a wind-down?** | No resolution plan, no prudential regulator and no depositor or investor protection in the ordinary sense — the group is outside the resolution regime, and the bank's protection is **collateral, margining and short-tenor exposure** | Structural |

**The single most important sentence in this table is row 6.** For this firm type, **the enforcement record is the governance evidence**, and it is the only external assessment of how the firm behaves when a control fails. Its shape here is favourable on the whole — **self-reported, remediated by undertaking, no relevant disciplinary history** on the trading entity's own SRO record ✅ — and it is materially better evidence than nothing, which is what the prudential route provides.

### 13.6 What This Guide Will Not Name

Stated as an explicit list, because the repository's hazard rules require it and because each omission is a control rather than a gap:

- **No bank, broker, dealer or exchange is named in this guide as Jump Trading's counterparty, client, prime broker, clearing member or clearing broker.**
- **The clearing broker in the NYSE Chicago matter is not named** — the order says only "its clearing broker" ✅ and this guide does not fill that in.
- **No exchange is named as the venue for a specific Jump trading activity**, beyond the firm's own NYSE Chicago participation and the memberships its own sources state (§2.3). **No DMM, SEF, market-maker programme or venue designation is asserted for this firm.**
- **No vendor, colocation provider, network carrier, microwave route or technology supplier is named for this firm**, except the **NVIDIA** relationship the firm itself announced on its own site ✅ (§8.4).
- **No client of Jump Capital and no counterparty of Jump Crypto is named.**
- **No figure for the firm's size, revenue, capital or headcount appears anywhere in this guide** ✅ (§7.4), and the one figure that circulates in the repository under the firm's name belongs to **QCP Capital** ✅ (§7.5, §11.3).
- **The Partior Series B participation is stated as a fundraise participation and nothing more** ✅ (§7.6) — **no institution named in that round is described anywhere in this guide as Jump's client, counterparty or clearing member.**

---

## 14. The Cymbal Bank Worked Example

> **Illustrative and fictional.** The scenario, entity names, limits, ratios and currency amounts in this section are **invented for illustration**. They are consistent with the public facts established in §1–§13, but **no number here is a disclosure by Jump Trading, by Cymbal Bank, or by any real institution**. The persona and worked-example conventions follow **[Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md)** and **[MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)**; the worked-example format for a firm of this type follows **[Hudson River Trading](hudson_river_trading_guide.md)** §10. **Cymbal Bank is the only bank persona in this guide.**

### 14.1 The Scenario

**Cymbal Bank** (Singapore) has an existing multi-product relationship with a large global proprietary trading firm — call it **"Lattice Trading"** in this worked example, which is Jump-shaped in every dimension that matters: **Chicago-headquartered, founded in 1999, a US registered broker-dealer among its entities, a Dutch entity with a Pillar III obligation, a British entity filing UK gender-pay-gap reports, an FX/futures entity adhering to the FX Global Code, a Singapore office on its own office list, a digital-asset arm with its own settled regulatory matter, and no published financial statements of any kind.**

The relationship has four components, and Cymbal Bank's file must handle each on its own terms: **liquidity provision** to Cymbal's client-facing FX and equities execution; **clearing and margin** where Cymbal acts as clearing member for part of Lattice's Singapore-booked activity; **an FX prime-brokerage facility** under which Cymbal novates and takes credit risk on Lattice's FX trades with third parties; and **a secured financing line** supporting Lattice's inventory. The onboarding question in front of the bank is a **new US$120 million secured financing limit** and a **renewal of the liquidity-provider panel slot**. All numbers invented.

### 14.2 Entity Identification — Which Entity Contracts

The first control, and the one this guide's §2 exists to serve. Cymbal Bank's onboarding team has already made the wrong first pass: it has a naming convention that reads "Lattice Trading" as one counterparty, and a name-screening run on the brand returned no adverse media against the *contracting* name. **The corrected analysis:**

| Relationship component | The entity Cymbal should contract with | Why, on the public record |
| --- | --- | --- |
| **FX/futures liquidity provision** | The **futures entity** (Lattice's equivalent of **Jump Trading Futures, LLC**) | It is the entity on the disclosures page that adheres to the **FX Global Code** and publishes the **Algo Due Diligence Template** and the **Liquidity Provider Disclosure Cover Sheet** ✅ — the artefacts that make the relationship assessable |
| **Equities/derivatives clearing, Singapore-booked** | Confirm whether the contracting entity is the **US broker-dealer**, the **EEA B.V.**, or a **Singapore entity** | The two US broker-dealers are register-visible ✅; the EEA entity is Pillar III-visible ✅; **the Singapore entity's identity is not on the disclosures page and its current status was not established** ⚠ — this is the gap the onboarding team must close **with the client**, not by inference |
| **Secured financing** | The **corporate principal** if the security is group-level, or the entity holding the financed inventory | Financing follows the asset and the obligor; the group's ownership chain is **not public** ✅, so the obligor and guarantor structure must come from the client |
| **The digital-asset business** | **Not part of this relationship, and a separate obligor group** | The arm's settled matter is against a **subsidiary of the crypto arm's holding company**, not the trading firm ✅ (§6.2) — the file must not aggregate the two, and **Cymbal must not describe the SEC order as a Lattice trading-entity matter** |

**Two controls fall out of the table, and both are cheap:** screen **every** name on the contracting entity's register entry, including any **other names** — this guide's real-world parallel is that the execution entity also appears as **"Jump Liquidity"** and the principal entity as **"AKAMAI TRADING LLC"** ✅ — and record **which entity is the obligor** in the facility documentation, because the group's legal structure is not public and a guarantee cannot be inferred from a brand.

### 14.3 The Counterparty-Credit Assessment Without Disclosure

The credit committee has no financial statements, no audited accounts, no rating and no supervisor's assessment. **The four substitute inputs of §13.5, worked through, with every number invented:**

| The missing input | The substitute Cymbal uses | What it produces in the file |
| --- | --- | --- |
| **Financial statements** | Request the **Pillar III disclosure** for the EEA entity (written request to compliance@) and obtain **audited accounts for the contracting entity directly from the client** as a condition of the limit. If the client declines the accounts, that refusal is itself a data point recorded in the file | A capital position for one entity, or a documented refusal |
| **A prudential regulator's view** | **The enforcement record as a governance proxy**: a **self-reported** control failure with a **US$250,000** fine and a **90-day certification undertaking**, in an entity the SRO recorded as having **no relevant disciplinary history**; and, on the **affiliate** rather than the trading entity, a **US$123,095,287** settled order **without admitting or denying** | A governance assessment scored on remediation behaviour and disclosure posture |
| **Conduct commitments** | **FX Global Code adherence**, the **Liquidity Provider Disclosure Cover Sheet** and the **Algo Due Diligence Template** — all first-party and published ✅ | Contractual hooks: the cover sheet becomes a schedule to the liquidity agreement |
| **An earnings history** | **None exists, and the file says so.** The only dated continuity evidence is first-party activity (technology and programme publications through 2026, register statuses, named senior personnel) ✅ | A **limit sized to collateral and tenor rather than to an earnings multiple** — because an earnings multiple cannot be computed |

**The assessment conclusion Cymbal reaches (invented):** a **"medium-high" internal credit grade**, driven not by any indication of weakness but by **structural unobservability** — no earnings history, no group capital, no prudential supervisor — combined with a **positive conduct-and-remediation record** on the trading entity and a **clean register status**. The grade is **capped** at that level regardless of the size of the firm, because **the input that would justify a higher grade does not exist**. That cap is the whole lesson of the worked example, and it is not specific to this firm: **for any proprietary trading firm, an unobserved counterparty cannot be a high-grade counterparty** — a point worth stating to a credit committee before it is tested by a client's excellent reputation.

### 14.4 The Market-Structure Risk the Relationship Carries

Two risk scenarios, both drawn from the record this guide documents rather than invented from nothing:

**Scenario A — dependence on one non-bank LP during stress (invented numbers).** Cymbal's client-facing FX stream takes **38%** of its executable depth from Lattice's panel slot during London hours. In a volatility event, Lattice's own risk limits bind and it widens or withdraws. Cymbal's clients see spreads triple and fill ratios halve. **The lesson the record supplies:** nothing obliges a proprietary firm trading its own capital to keep quoting, and the firm's own risk appetite is **unobservable from outside** ✅ — so **panel concentration is a limit in its own right**, distinct from counterparty credit, and should be sized by **depth share** rather than by relationship value. Cymbal's mitigation (invented): **a 25% per-provider depth-share cap**, two qualified alternative providers on warm standby, and a **quarterly mark-out analysis by provider** so that the cost of the concentration is priced rather than assumed.

**Scenario B — an algo-malfunction / erroneous-order event (invented numbers).** During a closing auction, a change to a Lattice end-of-day netting routine passes its historical regression suite, is deployed, and malfunctions — **increasing** positions instead of reducing them. Cymbal's clearing relationship means the erroneous executions flow through Cymbal's clearing member exposure before they are unwound. **This is the NYSE Chicago fact pattern and it is not hypothetical: the regulator's order documents 674 unintended executions totalling hundreds of millions of dollars, a net-capital violation lasting several hours, and an unwind executed by block trade with the firm's clearing broker** ✅ (§6.4). Cymbal's mitigations (invented, and each mapped to a documented control failure):

- **A blocking pre-trade limit, not a reducing one** — because a size cap that only shrinks an order to the maximum is exactly the deficiency the order identifies ✅.
- **A capital-utilisation limit computed on placed orders, not only executed ones** — because the order's second deficiency was a limit that could not see queued but unfilled limit-on-close orders ✅.
- **Documented control rationale** — because the order records that the firm "generally failed to sufficiently document how it determined its market access controls" ✅, and a bank's own market-access obligations run the same way.
- **A same-day escalation path and a pre-agreed unwind mechanism**, because the record shows the firm's own recovery ran through a block trade with its clearing broker ✅ — a mechanism Cymbal should have agreed in writing **before** it is needed.

### 14.5 The Recommendation, the Limit and the Condition

**Cymbal Bank's recommendation (invented, and every figure in it invented):**

| Element | Cymbal's decision | The reason, tied to a documented fact |
| --- | --- | --- |
| **Decision** | **Approve, with a limit and a condition** | The conduct record is favourable and the artefacts are available; the limit is sized to what is observable |
| **Secured financing limit** | **US$120 million**, fully collateralised, **tenor capped at 30 days** | No earnings history exists ✅, so the limit is sized to **collateral and not to earnings**; short tenor bounds the unobservable-risk window |
| **Unsecured exposure** | **US$10 million intraday only**, no overnight | The group has no prudential capital regime and no resolution plan ✅ (§13.5 row 10); overnight unobservable exposure is the line Cymbal declines to take |
| **Panel depth share** | **Maximum 25% per provider**, reviewed quarterly with mark-out analysis | Liquidity is optional in stress and the firm's risk appetite is unobservable ✅ |
| **Entity scope** | Facility papered **only** with the identified contracting entities; **no group guarantee assumed**; the digital-asset arm expressly out of scope | The ownership chain is not established ✅ (§2.4) and the affiliate's regulatory matter belongs to a different entity ✅ |
| **The condition Cymbal attaches** | **Delivery of the Pillar III disclosure for the EEA entity (via written request to the firm's compliance address) and audited accounts for the contracting entity, within 60 days of first drawdown; failure to deliver caps the facility at the fully-collateralised amount already approved** | **The reason, stated plainly:** every other substitute input Cymbal has is **behavioural** — a conduct record, a code adherence, a disclosure cover sheet ✅ — and none of them is **financial**. The condition is the mechanism that converts a public-disclosure posture into a private one, and it is only fair to the client that it is **stated as a condition rather than assumed as a right**. If the accounts are not delivered, the facility still works: it just works at the collateralised size, which is what the bank can actually underwrite |

**Why this is the right shape of recommendation for this firm type, in one paragraph.** Cymbal does not decline the relationship, because there is nothing in the record to decline it on: the trading entity's only regulatory matter is **self-reported, remediated and free of any relevant disciplinary history** ✅, and the firm publishes more counterparty-facing documentation than most of its peers ✅. Cymbal does not underwrite it as if it were a bank-shaped counterparty either, because the prudential inputs do not exist. **What Cymbal does is convert the unobservable into the bounded** — collateral instead of earnings, tenor instead of capital, depth caps instead of relationship goodwill, and a stated condition instead of an assumed disclosure. That is the entire art of onboarding a firm whose engineering is its P&L and whose financials are nobody's business: **where the balance sheet is dark, the limit is the light.** And it is the same conclusion the repository reaches for the peer firm type at [Hudson River Trading](hudson_river_trading_guide.md) §10.6, from an entirely different evidential base — which is itself the strongest evidence that the conclusion is structural rather than firm-specific.

---

## 15. The Anti-Patterns and the Claims Audit

### 15.1 The Anti-Patterns — Symptom, Cause, Guardrail

Seven failure modes recur whenever a firm of this type is written up, onboarded or credit-assessed, and each is stated as symptom, cause and guardrail because the guardrail is the only part that survives contact with a real file.

| # | Anti-pattern | Symptom in the file | Cause | Guardrail |
| --- | --- | --- | --- | --- |
| 1 | **Asserting a technical architecture from a blog post rather than the firm's own statement** | "Uses FPGA-based feed handlers and kernel bypass on X hardware" — a sentence with no source, in a capability assessment | Third-party write-ups, vendor case studies and peer architecture get read as descriptions of the firm; the genre's secondary literature is largely inferred | **Attribute only what the firm itself states, and mark the boundary.** This firm's own technology page supports "custom ASIC, FPGA, and platform development", "proprietary network infrastructure", four named languages, and the data-centre and co-location counts ✅ — and **nothing else**. Everything beyond that is someone else's inference (§8.2, §8.5) |
| 2 | **Repeating a regulatory matter from memory without the instrument and date** | "Jump was fined by the SEC over a net capital issue" or "Jump paid $123m for fraud" — a sentence a screening system cannot verify | Regulatory matters are the most retold and least re-read claims in the genre; the telling drifts from the instrument | **Every regulatory matter carries authority + instrument + date + outcome, or it does not go in the file** (§6.9). For this firm: SEC Admin. Proc. File No. 3-22382 / Release No. 33-11349, 20 Dec 2024, against **Tai Mo Shan Limited**; NYSE Chicago Proceeding No. 2018-11-00017, against **Jump Trading, LLC**. **Neither was a fraud finding against the trading firm** ✅ |
| 3 | **Conflating the trading firm with its digital-asset or venture arms** | The SEC order cited as a Jump Trading, LLC matter; a Jump Capital portfolio company described as a Jump client; the group treated as one obligor | The firm brands itself as one group ("Jump Trading Group", "Network: Jump Crypto, Jump Capital") and the press follows the brand; the entities are separate | **Separate the entities at the top of every section, and keep them separate in the file** (§5.4). The SEC respondent is a **subsidiary of Jump Crypto Holdings LLC** ✅; the venture arm's team and portfolio are its own ✅. **Screen and contract by legal entity, not by name cluster** |
| 4 | **Treating a reported capital figure as current** | "Client AUM/capital: US$1.5bn" — a number with no date, no outlet and no basis, in a field that looks like a fact | Reported figures migrate between firms, especially when two firms are named in the same source; and a date-free number reads as current by default | **Every capital figure carries outlet + date + basis, or it does not go in the field — and check whose figure it is before you use it.** **The worked example is this repository's own: the "over US$1.5 billion in assets deployed" claim attributed to Jump Trading belongs to QCP Capital, at §5.2 line 338 of the sibling guide, and the CEO attributed with it is QCP's** ✅ (§7.5, §11.3). **The migration mechanism is the paragraph** — two firms named in one bullet, one figure lifted out, subject changed. There is **no Jump Trading capital figure in the repository at all**, so the only correct file entry is "none established" (§7.4) |
| 5 | **Assuming a subsidiary's existence from a domain name** | "Jump Crypto Holdings Ltd (Cayman)" or "Tai Mo Shan Limited (Hong Kong)" written from the shape of a name or the registration of a website | Domain names and entity names are chosen for marketing and tax-legibility, not to declare jurisdiction; a website proves a brand, not a company | **Entity status comes from a register or an authority, never from a name or a URL** (§2.4). This firm's known edges are: **Tai Mo Shan Limited is a wholly owned subsidiary of Jump Crypto Holdings LLC** per the SEC ✅, and the four disclosure-bearing entities plus two BDs per the firm's own page ✅. **Tai Mo Shan's jurisdiction of incorporation is not stated in the source captured and is therefore not asserted** ⚠; **"Jump Trading Group" is not established as a legal entity and is treated as a brand** ⚠ |
| 6 | **Onboarding a different entity than the one that bears the risk** | The facility papered with the brand, the group, or the wrong affiliate; a guarantee assumed from a shared name | The names are confusable (two "Jump Trading"-adjacent BDs, plus other names "**Jump Liquidity**" and "**AKAMAI TRADING LLC**"), and the group's ownership chain is not public | **Establish the contracting entity, its register entry and its other names first, and record the obligor in the documentation** (§2.1, §2.3, §14.2). **Screen every register name**, because a liquidity relationship may be papered with an entity whose registered trading name is not "Jump Trading" ✅ |
| 7 | **Treating a reported investigation as an action** | "Jump is under CFTC investigation for manipulation" — an enforcement-flavoured sentence built on a news item | A reported probe and a settled order are both "regulatory news", and compression erases the distinction | **Label the item's kind explicitly** — action / meeting / subpoena / reported probe / allegation / personnel change — **as §6.9's table does.** For this firm: the CFTC item of **2 Nov 2010 is a meeting record** ✅; the **20 June 2024 Fortune/Reuters report of a CFTC probe is a reported investigation with no action identified** ⚠; the **2023 and 2025 civil matters are allegations in complaints** ⚠; and **there is no finding against any Jump entity in either** ✅ |

The unifying failure is a **provenance collapse**: a number, a name or a relationship migrating from a dated, attributed source into a file as an unattributed fact. Every guardrail above is the same instruction in different clothes — **keep the outlet, keep the date, keep the instrument, keep the entity, and where a private firm discloses nothing, write that down and stop.**

### 15.2 The Verified Claims (✅)

| Claim | Source | Date |
| --- | --- | --- |
| Founded 1999, Chicago; founders **Bill DiSomma** and **Paul Gurinas**, former CME pit traders who met in the Deutsche Mark pit | Wikipedia, citing Forbes ("It's Not The Pits", 2006) and Bloomberg | — (secondary, consistent) |
| Firm "trades **solely in a proprietary capacity**"; "engages in algorithmic trading of a variety of asset classes on numerous securities venues" | **NYSE Chicago order**, Proceeding No. 2018-11-00017 | incident 4 May 2018; order in the 2019 folder |
| Jump Trading, LLC became a **Participant of NYSE Chicago on 22 December 2000**; principal office Chicago, Illinois | **NYSE Chicago order** | same |
| **Jump Trading, LLC** — CRD **106124**, SEC file **8-52989**, ACTIVE, 6 branches, disclosure flag **Y**, **Delaware LLC formed 31 Oct 2008**, 15 items typed "Regulatory Event"; **Jump Execution, LLC** (other name "**Jump Liquidity**") — CRD **313060**, SEC file **8-70668**, ACTIVE, 5 branches, no disclosure flag, **Delaware LLC formed 2 Jul 2020**; both at 600 West Chicago Avenue, Suite 600, Chicago IL 60654 | **FINRA BrokerCheck** — firm search index and firm records | queried 24 Sep 2026 |
| **SEC order**: Admin. Proc. File No. **3-22382**, Release No. **33-11349**, PR **2024-212**; respondent **Tai Mo Shan Limited**, a wholly owned subsidiary of **Jump Crypto Holdings LLC**; findings under **Securities Act §§5(a), 5(c)** and **§17(a)(3)**; **$73,452,756** disgorgement + **$12,916,153** prejudgment interest + **$36,726,378** penalty = **$123,095,287**; cease and desist; **"without admitting or denying"**; respondent **paid in full** | **SEC** press release and harmed-investor page | **20 Dec 2024** |
| **Fair Fund**: created under Sarbanes-Oxley §308(a); **Miller Kaplan Arase LLP** appointed Tax Administrator; proposed plan of distribution published with a 30-day comment period; claim period **23 May 2021 – 8 May 2022** | **SEC** Releases No. **34-102662** and **34-106168** | 13 Mar 2025; 20 Aug 2026 |
| **NYSE Chicago order** against Jump Trading, LLC: censure + **$250,000** fine + **90-day** certification undertaking; violations of **Exchange Act Rules 15c3-5(b), 15c3-5(c)(1), 15c3-1** and **NYSE Chicago Art. 6 R.5, Art. 7 R.3(a)(1)(A)**; settled **without admitting or denying**; "does not have any relevant disciplinary history"; **674 unintended executions**; unwind via **block trade with its clearing broker** (unnamed) | **NYSE Chicago, Inc.**, Proceeding No. **2018-11-00017** | order issued 2019 folder (exact date not read) |
| **CFTC External Meetings record** for a meeting at Jump Trading: **Gary Gensler** and **Rosemary Hollinger** with **Matt Schrecengost, Bill DiSomma, Paul Gurinas**; subject the **definition of spoofing** and **SEF access**; rulemakings **XIII SEF Registration**, **XXIV Disruptive Trading Practices** | **CFTC**, cftc.gov/node/171661 | **2 Nov 2010** |
| Firm's own asset classes: **Equities · Options · Credit · Commodities · Digital Assets · Futures**; strategies from "**high-frequency to stat arb to discretionary macro**" | **jumptrading.com/trading** | accessed 24 Sep 2026 |
| Firm's own technology statements: "**Three large, purpose-built research data centers and over 100 co-located compute environments near global exchanges**"; "**custom ASIC, FPGA, and platform development**"; "**proprietary network infrastructure**"; "**C++, Python, Go, and Rust**"; storage/simulation/analytics platforms | **jumptrading.com/technology** | accessed 24 Sep 2026 |
| **Alex Davies, CTO**: "We build our own hardware, run large-scale AI workloads, and move faster than most tech companies." | **jumptrading.com/technology** | accessed 24 Sep 2026 |
| Disclosures page entity list: **Jump Trading, LLC / Jump Execution, LLC** (BD disclosures); **Jump Trading Futures, LLC** (FX Global Code; Algo Due Diligence Template; Liquidity Provider Disclosure Cover Sheet); **Jump Trading Europe B.V.** (Pillar III on written request to compliance@jumptrading.com); **Jump Trading International, Ltd.** (UK Gender Pay Gap 2023/2024/2025; Modern Slavery Statement); **Jump Trading Group** (Eightfold AI recruiting-tool bias audit; Healthcare Disclosures) | **jumptrading.com/disclosures** | accessed 24 Sep 2026 |
| Office list (13): **Chicago, New York, Austin, London, Singapore, Shanghai, Bristol, Mumbai, GIFT City, Sydney, Amsterdam, Hong Kong, Paris** | **jumptrading.com** | accessed 24 Sep 2026 |
| Footer "Network" list: **Jump Crypto**, **Jump Capital**; a **Fraud & Scam Alert** page | **jumptrading.com** | accessed 24 Sep 2026 |
| **Jump Crypto** live; self-description "Builders of blockchain technology revolutionizing open, community-driven networks."; pillars **Trade / Build / Invest**; publications dated **21 Aug 2025**, **15 Apr 2026**, **27 Apr 2026** | **jumpcrypto.com** | accessed 24 Sep 2026 |
| **Jump Capital** live; positioning on early-stage founders and AI; team page names **Sach Chitnis** and **Mike McMahon** as **Co-Founder and Partner**, plus eleven further named team members | **jumpcap.com** | accessed 24 Sep 2026 |
| Firm-published 2026 items: **NVIDIA Vera Rubin** deployment (17 Mar); **UCL Gatsby** four PhD scholarships (16 Mar); **2026–2027 Fellows**, "we selected eight" (23 Apr); **Probability Cup** (10 Jun); conference presence at **ICLR 2026**, **SREcon Americas 2026**, **NVIDIA GTC 2026** | **jumptrading.com/signals** | accessed 24 Sep 2026 |
| **Singapore** on the firm's own current office list | **jumptrading.com** | accessed 24 Sep 2026 |
| **Jump Trading Pacific Pte. Ltd.** incorporated **15 March 2011**, Asia Square Tower 1, 8 Marina View | **ACRA** via opengovsg.com / sgpbusiness.com — recorded in the sibling guide §4.4 | recorded in repo |
| **Partior Series B** — US$60m first close July 2024, led by Peak XV with Valor Capital and **Jump Trading** | [Tokenized Assets](tokenized_assets_guide.md) line 239 | recorded in repo |
| **Wormhole** exploit 2 Feb 2022 (~120,000 wETH ≈ US$320m reported); parent restored funds; US$10m white-hat bounty | [Smart Contracts](../technology/smart_contracts_guide.md) §9 (line 632) | recorded in repo |
| **Terraform / Do Kwon** found liable for fraud and unregistered securities offerings; agreed to pay **$4.5bn** | **SEC PR 2024-212** (context the SEC supplies) | April 2024 |

### 15.3 The Flagged Claims (⚠)

| Claim | Why flagged | Source and date |
| --- | --- | --- |
| Founder biographical detail (met in the **Deutsche Mark pit** at the CME; firm began in open outcry) | Secondary/encyclopaedic, resting on cited press not re-read this pass | Wikipedia citing Forbes 2006 and Bloomberg |
| **Jump Capital founded June 2012 by Gurinas and DiSomma** | Contradicted in part by Jump Capital's own team page, which names **Chitnis and McMahon** as Co-Founders. **Both recorded; neither resolved** (§5.3) | Wikipedia vs jumpcap.com, 24 Sep 2026 |
| **2013 purchase of a Belgian microwave tower by a UK affiliate** | Reported, single outlet; **no route, site, vendor or counterparty named** and none asserted here | Bloomberg, 2014 |
| **April 2014 NY AG subpoena**, alongside five other high-speed firms | Reported; **no disposition established** | reported |
| **CFTC probe of Jump Crypto**, reported 20 June 2024 | Reported investigation; **no action identified**; "reported, not an action" applied throughout | Fortune, carried by Reuters, 20 Jun 2024 |
| **Kanav Kariya's June 2024 departure** and its linkage to the probe | Departure is reported; **the linkage is press characterisation** and is labelled as such | reported, e.g. Cointelegraph, 24 Jun 2024 |
| **Class action docket development**: amended Aug 2024 (adding **Patel, Woolley, Worsham** as plaintiffs and **Jump Crypto Holdings LLC, DiSomma** as defendants); motion to compel arbitration **denied 9 May 2025** by Judge **Georgia Nick Alexakis** (N.D. Ill.) | **Single secondary source**; not verified at the docket; the complaint's status and outcome remain unestablished | BlockTribune, reported |
| **$4bn Terraform plan-administrator suit** against Jump Trading, DiSomma and Kariya, filed **18 Dec 2025** (some reports 19 Dec) by **Todd R. Snyder** in N.D. Ill. | Reported by multiple crypto outlets; **date varies by a day**; no primary filing read; **allegations only** | crypto press, Dec 2025 |
| The alleged **>$1.28bn** in the 2023 complaint | **An allegation from a complaint**, not a finding, and not a box-office figure for the firm | filed complaint |
| **FIA Principal Traders Group membership** | Carried as a class fact; **the membership list was not re-read this pass** and a named-firm claim should rest on the list | class fact, FIA PTG |
| **Reported April 2026 departures** of researchers **Yiming Zhang** (joined 2009) and **Darko Kirovski** (joined 2011) | Press only; not verified at a primary source; no conclusion drawn | Business Insider, Apr 2026 |
| **Prediction-markets team doubled to ~20**, equity in prediction-market venues | Crypto-press reports only; **no positioning conclusion drawn** | crypto press, 2025–2026 |
| **Microwave-link low-latency infrastructure** (as stated in the sibling guide §4.4) | Carried in that guide from Wikipedia and tradermath.org — secondary; **this guide asserts no route or vendor** | sibling guide §4.4 ⚠ |
| **Nasdaq Stockholm derivatives membership as "Jump Trading Europe B.V."**; CME/NYSE/Eurex/LSE/LME memberships | Outlet named (GlobeNewswire) but not re-read this pass; second-hand through the citation chain | GlobeNewswire, 17 Dec 2018; lme.com — via Wikipedia |
| **Jump Capital invested US$5m in The Small Exchange (May 2019)** | Secondary; the release's headline also names **Citadel Securities** in the same round — **the co-investor relationship is not asserted here** | Wikipedia citing Businesswire, May 2019 |
| The **exact issuance date of the NYSE Chicago order** | The PDF's signature/date line extracted garbled; only the **2019** folder location is reliable | NYSE Chicago order |

### 15.4 The Rejected or Not-Found Claims (❌)

| Claim | Status | Evidence |
| --- | --- | --- |
| **Any revenue, profit, capital, AUM or valuation figure for Jump Trading** | ❌ **Not found and not asserted.** The absence is the finding | jumptrading.com and its disclosures page contain no financial item (accessed 24 Sep 2026) (§7.4) |
| **"Jump Trading deployed over US$1.5 billion in assets"** | ❌ **Rejected — mis-attributed.** The figure belongs to **QCP Capital** (§5.2 line 338 of the sibling guide) | §7.5, §11.3 |
| **Jump Trading CEO "Melvin Deng"** | ❌ **Rejected — mis-attributed.** Melvin Deng is **QCP Capital's** CEO | same |
| **"Jump was fined $250,000 by the SEC in May 2018"** | ❌ **Rejected.** The fine is **NYSE Chicago's**, and May 2018 is the **incident** date, not the order date (§15.5) | Wikipedia's rendering vs the NYSE Chicago order |
| **A finding of fraud against Jump Trading, LLC** | ❌ **Not found.** The SEC findings are **statutory-underwriter registration violations** and **negligent** conduct, against **Tai Mo Shan Limited**; the district court's fraud finding was against **Terraform and Do Kwon** | SEC PR 2024-212; harmed-investor page |
| **Any CFTC enforcement action naming Jump Trading or its affiliates** | ❌ **Not found** (a search result, not a negative proof) | CFTC enforcement listings searched (§6.5) |
| **A SEF registration, a DMM designation, or any market-maker programme membership for this firm** | ❌ **Not found and not asserted** | no source identified (§10.2) |
| **A MAS licence or Singapore-regulated entity for any Jump entity** | ❌ **Not found** — an **open item, not a negative finding** | no MAS FID entry located (§11.4) |
| **Ownership chain, ultimate holding company, ultimate beneficial owners** | ❌ **Not established for any entity except the one SEC-stated edge** | §2.4 |
| **Any named counterparty, client, prime broker or clearing broker of the firm** | ❌ **Not asserted anywhere in this guide, by rule** | §13.6 |
| **Any named vendor, colocation provider, network carrier or microwave route** | ❌ **Not asserted**, except the **NVIDIA** relationship the firm itself announced | §8.4, §8.5 |
| **The firm's technology stack in any implementation particular** (latency figures, system names, silicon family, NIC, protocol) | ❌ **Not publicly documented by the firm** | §8.5 |
| **A Cymbal Bank relationship of any kind with this firm** | ❌ | The §14 worked example is explicitly hypothetical and uses an invented counterparty name |

### 15.5 The Reconciled Corrections

Two reconciliations belong in this guide's audit, and both are corrections of **attribution** rather than of underlying fact — which is the harder kind to catch, because each underlying document is real.

**(a) The Wikipedia / NYSE-Chicago mis-attribution.** Wikipedia renders the 2018 matter as *"In May 2018, Jump was fined $250,000 by the U.S. Securities and Exchange Commission (SEC)"*, citing **the NYSE Chicago order**. Every element of that sentence is wrong in a way that matters:

| Element | Wikipedia's rendering | The document itself |
| --- | --- | --- |
| **Authority** | U.S. Securities and Exchange Commission | **NYSE Chicago, Inc.** — an exchange SRO ✅ |
| **Date** | May 2018 | **4 May 2018 is the incident**; the **order sits in the 2019 folder** ✅ |
| **Instrument** | (none given) | **Order Instituting Proceedings, Accepting Settlement, Making Findings, and Imposing Sanctions; Proceeding No. 2018-11-00017** ✅ |
| **The fine** | $250,000 | **$250,000 — correct as a figure, wrong as an authority's** ✅ |

**Consequence for a file:** a screening system that reads "SEC fine" flags a **securities-fraud-adjacent** item; the correct reading is an **SRO operational-control sanction with a self-report and a remediation undertaking** — a materially different governance signal. **Do not repeat the SEC attribution.**

**(b) The QCP mis-attribution.** The dispatch brief for this guide attributed to Jump Trading a scale claim ("over US$1.5 billion in assets deployed (per a 2023 profile)") and a CEO ("Melvin Deng, joined 2021") by way of the sibling guide's §4.4. **Both belong to QCP Capital**, at **§5.2 line 338** of the same guide, and **§4.4 contains only founded / products / Singapore presence** ✅. The mechanism is instructive and is the worked example of anti-pattern 4: the QCP entry's own bullet **names "Jump Crypto"** in the same sentence as the figure, so a reader lifting the figure out of that paragraph acquires the wrong subject. **The repository contains no Jump Trading capital figure at all, in any guide** ✅ — so the correct file entry is **"none established"**, and the figure should be recorded against **QCP Capital** wherever it is used. **A third, smaller reconciliation, recorded for completeness:** the sibling guide's §4.4 and §6 carry **"microwave links"** for this firm, sourced to **Wikipedia and tradermath.org** ⚠, while the firm's own material **states no microwave or laser link anywhere examined** ✅ — the two are not in conflict (a firm's silence about a route is not a denial), but **the infrastructure claim is secondary-sourced and is labelled as such in this guide**, and **no route, path or counterparty site is asserted** (§8.5, §13.6).

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

This subsection collects every item this pass could not confirm against a primary or reliable source, so a reader can distinguish verified fact from honest uncertainty. **Nothing in this list is asserted anywhere in this guide.**

1. **The ultimate holding company and the ownership chain.** Whether a top-level holding company exists, its jurisdiction, and how **Jump Crypto Holdings LLC**, **Jump Trading, LLC**, **Jump Trading Futures, LLC**, **Jump Trading Europe B.V.**, **Jump Trading International, Ltd.** and **Jump Trading Pacific Pte. Ltd.** relate to one another. **One edge only is established:** Tai Mo Shan Limited is a wholly owned subsidiary of Jump Crypto Holdings LLC (SEC).
2. **How "Jump Trading Group" relates to the legal entities** — whether it is a company, a brand, a trading name, or a label on disclosures. No register identifies it as an entity; the firm uses it corporately in a 2021 press release and on its own disclosures page.
3. **Tai Mo Shan Limited's jurisdiction of incorporation.** **Not stated in the SEC text captured**, and therefore not asserted despite the name.
4. **Three Singapore facts, none of them established.** (i) **Whether Jump Trading Pacific Pte. Ltd. remains an active Singapore entity today** — the ACRA incorporation date (15 March 2011) is in the repository, but **no current-status check was performed this pass**; the firm's own office list confirms **Singapore as a location**, which is a different fact. (ii) **The Singapore office's establishment year at the firm's own hand** — the firm names Singapore as a location and **states no year**, so the 2011 date rests entirely on ACRA. (iii) **Whether any Jump entity holds a MAS licence, what the Singapore office does, and its headcount** — no MAS Financial Institutions Directory entry for any Jump entity was located this pass. **An open item, not a negative finding.**
5. **Any financial figure whatsoever** — revenue, profit, capital, AUM, valuation, or the size of any trading book or strategy. **None exists publicly.** The closest is the **Pillar III disclosure of Jump Trading Europe B.V.**, available only on written request and only for that entity.
6. **The current status of the 2023 class action.** No judgment, dismissal or settlement identified. A reported 9 May 2025 arbitration ruling rests on **one secondary source**.
7. **Whether any CFTC action has followed the reported June 2024 probe.** None identified; **an empty search, not a negative proof** — recorded as a limitation of the search.
8. **The firm's headcount.** The firm's own site renders employee/office/country counters as animated digits that **do not extract from the page source**; Wikipedia's "over 2,000 employees" is secondary and undated in the material examined. **No headcount is stated in this guide.**
9. **The exact issuance date of the NYSE Chicago order.** The PDF's signature/date line extracted garbled; only the **2019** folder placement is reliable.
10. **What sits behind the "Y" flag on Jump Trading, LLC's BrokerCheck record.** The record states **15 items typed "Regulatory Event"** ✅; the item-level endpoint requires authentication (**HTTP 403, "Missing Authentication Token"**) ⚠, so the count is recorded and the items are not characterised. **The domicile and formation dates of the two US broker-dealers were established** (Delaware; 31 Oct 2008 and 2 Jul 2020) ✅ — but no other group entity's domicile was (§2.4).
11. **The disposition of the April 2014 NY AG inquiry as it concerned this firm.** No finding, sanction or resolution identified.
12. **The outcome of the December 2025 Terraform plan-administrator suit** — reported filings, no primary document read, **allegations only**.
13. **The current head of Jump Crypto, and the current status of the arm's trading book.** The arm is publishing in 2026; **its current leadership after the June 2024 departure and the scale of its trading activity are not established**.
14. **Whether the Fair Fund distribution has been made.** As at the 20 August 2026 notice of the proposed plan, the comment period was open.
15. **Any quantitative particular of the firm's performance or its technology** — no **market-share, volume or venue-ranking statistic** in any source examined, and no **latency figure, system name, silicon family, NIC, protocol, colocation provider or network carrier**. **The firm has published none of these**; the secondary-sourced microwave-link claim is flagged and attributed as secondary.
16. **Compensation, equity or profit-sharing arrangements, and attrition or tenure data** — nothing verifiable; **no compensation figure is quoted anywhere in this guide**.
17. **Tool limitations encountered this pass, recorded as limitations and never as evidence of absence:** **three `web_search` queries returned empty result sets** (the class-action status, the CFTC enforcement question, and a Jump Trading 2026 news query in one attempt); the **FINRA BrokerCheck item-level disclosure endpoint requires authentication (HTTP 403, "Missing Authentication Token")** while the search index and the firm records answered; the **CFTC's own site search did not return a usable page** to the scraper (the enforcement-listing page itself was retrievable but carried no Jump entry in the portion read); and the firm's own site renders certain counters as animated digits that cannot be read from the page source. **An empty search is a tool limitation here, not a finding.**

### 16.2 Glossary

| Term | Meaning |
| --- | --- |
| **Proprietary trading (prop)** | Trading a firm's own capital as principal, rather than managing clients' money or acting as agent. The defining structural fact about this firm: the NYSE Chicago order records that it "trades solely in a proprietary capacity" |
| **ASIC / FPGA** | Application-Specific Integrated Circuit and Field-Programmable Gate Array — custom and reconfigurable silicon for low-latency and compute-intensive workloads; both are named on the firm's own technology page |
| **Rule 15c3-5** | The SEC's **Market Access Rule** — pre-trade risk controls and supervisory procedures for broker-dealers with market access, plus a control-person certification. **This firm was found in violation of 15c3-5(b) and (c)(1)** |
| **Rule 15c3-1** | The SEC's **net capital rule** for broker-dealers. **The subject of the 4 May 2018 self-reported violation** |
| **Net capital** | The US broker-dealer capital requirement reported privately to FINRA; relevant to any exposure to a broker-dealer entity of the group |
| **NYSE Chicago** | The exchange SRO that issued the 2019 order in this guide's §6.4. **Not the SEC** — the distinction that the Wikipedia reconciliation (§15.5) is about |
| **Admin. Proc. File No. / Release No.** | The SEC's docket and release identifiers for an administrative proceeding. **3-22382** and **33-11349** identify this guide's SEC matter; **34-102662** and **34-106168** identify its Fair Fund steps |
| **Statutory underwriter** | A person or firm that purchases securities from an issuer with a view to distribution, and is therefore subject to the Securities Act §5 registration requirements. **The SEC's first finding against Tai Mo Shan Limited** |
| **Fair Fund** | A fund created under Sarbanes-Oxley §308(a) so that penalties, disgorgement and interest collected by the SEC can be distributed to harmed investors |
| **"Without admitting or denying"** | The settlement formulation in both matters here — the respondent neither admits nor denies the findings, while consenting to the order. **Say so because the authority said so** |
| **Pillar III** | The disclosure pillar of the **CRR/CRD** prudential framework for EU/EEA credit institutions and certain investment firms — capital, risk and governance disclosure at the regulated-entity level. **Jump Trading Europe B.V.** publishes one on written request |
| **FX Global Code** | The Global Foreign Exchange Committee's code of conduct for the FX market. **Jump Trading Futures, LLC** adheres, and publishes a **Liquidity Provider Disclosure Cover Sheet** |
| **Algo Due Diligence Template** | The published framework for algorithmic due-diligence questionnaires — a counterparty-facing artefact this firm publishes |
| **FIA Principal Traders Group (PTG)** | The Futures Industry Association's policy group for firms trading their own capital — the industry vehicle for this firm class |
| **SEF** | Swap Execution Facility — a registered trading venue for swaps; discussed in the firm's 2 November 2010 CFTC meeting |
| **Spoofing** | A disruptive trading practice involving orders placed without intent to execute; **the subject of the firm's 2010 meeting with CFTC staff, and not the subject of any action against it** |
| **BrokerCheck / CRD / SEC file number** | FINRA's public broker-dealer registry, keyed by CRD and SEC 8-XXXXX numbers. **Jump Trading, LLC**: CRD 106124 / SEC 8-52989, **Delaware LLC formed 31 Oct 2008**. **Jump Execution, LLC** ("Jump Liquidity"): CRD 313060 / SEC 8-70668, **Delaware LLC formed 2 Jul 2020** |
| **ACRA / UEN** | The Accounting and Corporate Regulatory Authority of Singapore, and the Unique Entity Number it issues; the register behind the **Jump Trading Pacific Pte. Ltd.** record of 15 March 2011 |
| **CMS licence** | Capital Markets Services licence — the MAS licence for dealing in capital-markets products in Singapore. **No Jump entity's CMS licence was established this pass** |
| **DMM** | Designated Market Maker — an exchange-appointed liquidity provider with quoted obligations. **No DMM designation for this firm was established** |
| **Prudential perimeter** | The bank-style regulatory regime (capital, liquidity, leverage, resolution, supervision of safety and soundness) that a proprietary trading firm **sits outside of**, except where a group entity is inside a prudential regime (here, the Dutch B.V.) |
| **Partior** | The interbank clearing and settlement network in which **Jump Trading** participated as an investor in the **Series B** (US$60m first close, July 2024, per [Tokenized Assets](tokenized_assets_guide.md) line 239). A **fundraise participation**, not a Jump financial and not a counterparty relationship |
| **Tai Mo Shan Limited** | The SEC respondent of 20 December 2024, described by the SEC as a **wholly owned subsidiary of Jump Crypto Holdings LLC**. **Not** a Jump Trading, LLC matter |
| **Wormhole** | The cross-chain bridge exploited in February 2022 (~120,000 wETH ≈ US$320m reported); the parent restored the funds and a US$10m white-hat bounty was offered. **Owned by [Smart Contracts](../technology/smart_contracts_guide.md) §9** |

### 16.3 Cross-References

**Repository guides (banking siblings — plain filenames):**
- [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) — **§4.4 owns the Jump Trading entry** (founded / products / Singapore presence) and **§5.2 owns QCP Capital**; §3 the SGX DMM schemes; §6 the comparison table; §7 the SGX/CDP/SGX-DC infrastructure; §8 the MAS overlay (§1.5, §7.5, §11.1, §11.3, §12.1, §13)
- [Hudson River Trading](hudson_river_trading_guide.md) — the prop-firm archetype: §3.3 mid-frequency/HFT, §5 technology, §6 talent, §7/§8 regulatory and market structure, §9 Singapore/Asia, §10 the Cymbal Bank worked example, §11 the claims audit (§1.5, §4.2, §7.4, §8.1, §9.1, §10.1, §13, §14.5)
- [Citadel LLC](citadel_llc_guide.md) — the archetype firm guide; **its §7 owns the technology-of-the-firm-type material** (§1.5, §8.1)
- [ExodusPoint](exoduspoint_guide.md) — the identity and capital discipline this guide adopts, and its §13 multi-manager worked example (§2, §7, §12.3)
- [Tokenized Assets](tokenized_assets_guide.md) — line 239, the Partior Series B (§7.6, §15.2)
- [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) and [Hedge Fund Guide](hedge_fund_guide.md) — the fund-management contrast (§12.3)
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore regulatory overlay and the Cymbal Bank persona conventions (§1.5, §14)
- [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) — the Cymbal Bank worked-example conventions; its line 319 records the Asia Square Tower 1 unit detail (§11.1, §14)
- [Crane Capital](crane_capital_guide.md) — the identity-gate and capital-honesty pattern followed in §2 and §7

**Repository guides (technology — prefix `../technology/`):**
- [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md) · [Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md) · [Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md) — the latency discipline itself (§1.4, §8.1)
- [Smart Contracts](../technology/smart_contracts_guide.md) — **§9, line 632, owns the Wormhole exploit record** (§5.1, §6.7, §15.2)
- [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) — the quant-research and engineering skill stack (§8.1, §9.1)
- [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md) — the platform architecture and market-access structure (§8.1, §13)

**Primary sources used this pass (all accessed 24 September 2026 unless dated):**
- **SEC** — Press Release **2024-212** (20 Dec 2024); the harmed-investor page for *In the Matter of Tai Mo Shan Limited* (Admin. Proc. File No. 3-22382; page reviewed/updated 24 Aug 2026); Release Nos. **34-102662** and **34-106168**
- **NYSE Chicago, Inc.** — Order Instituting Proceedings, Accepting Settlement, Making Findings, and Imposing Sanctions, Proceeding No. **2018-11-00017** (PDF, nyse.com, 2019 disciplinary-actions folder)
- **CFTC** — External Meetings record, cftc.gov/node/171661 (2 Nov 2010)
- **jumptrading.com** — the home page, the **technology** page, the **trading** page, the **disclosures** page, the **signals** page, the careers pages, the Fraud & Scam Alert page, the office/footer list
- **jumpcrypto.com** and **jumpcap.com** — the arms' own sites, including the Jump Capital team page
- **FINRA BrokerCheck** — the firm search index **and the firm records** for CRD **106124** (Jump Trading, LLC) and CRD **313060** (Jump Execution, LLC), queried 24 Sep 2026: both **Delaware** LLCs, formed **31 Oct 2008** and **2 Jul 2020**, disclosure flag **Y** (15 items typed "Regulatory Event") and **N**. **The item-level disclosure endpoint requires authentication (HTTP 403, "Missing Authentication Token")**, so no item text was read
- **Wikipedia (Jump Trading)** — used for the 1999 founding, the founder record, the Jump Capital founding claim and the 2014 subpoena, with its citation chain to Forbes (2006), Bloomberg, Crain's Chicago Business (2016), Businesswire (May 2019) and GlobeNewswire (Dec 2018), all flagged ⚠ where not re-read at the outlet

### 16.4 Closing Summary

Jump Trading is a useful case study precisely because it is the **opposite** of the usual problem in this genre. Where most private trading firms publish nothing, **this firm publishes a great deal — about its engineering, its programmes and its culture — and nothing at all about its money.** The record assembled here has three findings a reader will not get from the firm's Wikipedia page.

**First, the firm's technology posture is first-party and legible, and it stops exactly where the firm stops.** Its own technology page gives "three large, purpose-built research data centers and over 100 co-located compute environments near global exchanges", "custom ASIC, FPGA, and platform development", "proprietary network infrastructure", four named languages, and a CTO stating that the firm builds its own hardware and runs large-scale AI workloads ✅. That is more than most peers publish and it is a **recruiting** posture rather than a disclosure posture: at a firm with no clients, the engineering story *is* the hiring pitch, and the deployment cadence is the P&L. What the firm does not publish is any latency figure, system name or route, and **a bank's architecture document should therefore contain only what the firm itself has said** (§8.2, §8.5, §15.1(i)).

**Second, the regulatory record is two matters in two different firms and one meeting — and the distinction is the whole finding.** The **US$123,095,287** SEC order of 20 December 2024 is against **Tai Mo Shan Limited**, a wholly owned subsidiary of **Jump Crypto Holdings LLC**, on findings of **statutory-underwriter** registration violations under Securities Act §5 and **negligent** misleading conduct under **§17(a)(3)**, entered **without admitting or denying** the SEC's findings and **paid in full** ✅. The **US$250,000** NYSE Chicago censure is against **Jump Trading, LLC** and arises from a **self-reported net capital violation** on 4 May 2018 that produced **674 unintended executions** — an **operational-control** matter with a **90-day certification undertaking**, in a firm the exchange recorded as having **no relevant disciplinary history** ✅. The **CFTC** item is a **meeting record from 2 November 2010**, not an action ✅; the 2024 CFTC matter is a **reported probe with no action identified** ⚠; and the civil matters are **allegations in complaints** ⚠. **A file that merges any two of those six items is wrong, and the Wikipedia rendering of the NYSE Chicago fine as an SEC fine (§15.5) shows how easily it happens.**

**Third, the capital question has no answer, and the repository's own version of it was wrong.** There is **no revenue, profit, capital, AUM, valuation or headcount figure for this firm anywhere in the public record examined** ✅. The "over US$1.5 billion in assets deployed" claim attributed to Jump Trading in the dispatch brief belongs to **QCP Capital** at §5.2 line 338 of the sibling guide, and so does the CEO attributed with it ✅ — §4.4 of that guide contains only founded / products / Singapore presence, and the repository holds **no Jump financial figure at all**. The group's nearest thing to a capital document is the **Pillar III disclosure of its Dutch entity**, obtainable only on written request ✅. **For a bank, that absence is not an obstacle to be worked around with estimates; it is the input that determines the shape of the relationship** — collateral instead of earnings, short tenor instead of capital assessment, depth caps instead of relationship goodwill, and a stated disclosure condition instead of an assumed one (§14.5).

For **Cymbal Bank**, the operating conclusions are four, and each follows from the evidence rather than from the reputation. **Get the entity right first** — the names are confusable, the register carries "**Jump Liquidity**" and "**AKAMAI TRADING LLC**" as other names, the ownership chain is not public, and the difference between a US registered broker-dealer, an EEA Pillar III entity and a corporate principal is the difference between three reporting and capital outcomes (§2, §13.1, §14.2). **Use the firm's own artefacts as the relationship's hooks** — the **Algo Due Diligence Template**, the **FX Global Code Liquidity Provider Disclosure Cover Sheet** and the FX Global Code adherence are published, first-party, and make the conduct dimension assessable in a way most peers' do not (§13.4). **Read the conduct record as the governance evidence**, because no prudential supervisor supplies one — and read it as favourable on the whole: self-reported, remediated, no relevant disciplinary history (§13.5). And **treat the firm's liquidity as optional in stress and its risk appetite as unobservable**, which makes panel concentration a limit in its own right rather than a by-product of the credit line (§13.2, §14.4).

Underneath all of it sits the thesis this guide opened with, and the record supports it more directly than a general argument could. A firm with **no clients** has nothing between its engineering and its money — which is why its own CTO describes designs reaching production in a week ✅, why its own technology page leads with silicon and data centres ✅, and why the one physical-infrastructure story in its press record is a **microwave tower** ⚠. When a firm removes every intermediary between a research idea and a filled order, what is left as the constraint is not organisational. It is distance, and the speed limit on the medium that covers it. For a firm like this one, latency is a physics problem.
