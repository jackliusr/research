# Tower Research Capital: A Comprehensive Guide

**The Identity, Entities, History, Business, Regulatory Record, Technology Posture, Talent Model, Asia Footprint, Capital Silence, Peer Positioning and Bank Interface of the New York Proprietary Trading Firm Tower Research Capital LLC — with the 2019 Deferred Prosecution Agreement and the Parallel CFTC Settlement for E-Mini Spoofing, the Individual Trader Prosecutions Kept Strictly Separate, the Latour Trading Net-Capital Matter as a Flagged Secondary Claim, and a Cymbal Bank Counterparty Worked Example**

> **Author:** Jack Liu Shurui, Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Institutional Investment & Capital Markets — the proprietary trading firm as a distinct firm type, the no-clients business model, the registered-entity and systematic-internaliser perimeter, co-location and the latency posture, the commodities-fraud and spoofing enforcement record and what it does and does not establish, the crypto team, the ventures arm, the Singapore/Asia footprint, and the Cymbal Bank institutional-counterparty lens
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder):** [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) (**its §4.6 owns the Singapore facts about this firm and is cross-referenced throughout — §9.1 here reproduces its ⚠ and §9.5 reconciles against it; do not re-derive**; its §6 comparison table carries the Tower row) · [Hudson River Trading](hudson_river_trading_guide.md) (the prop-firm archetype in this repository: **§3** the business and the franchise explanation, **§5** technology, **§6** talent, **§7/§8** regulatory and market-structure context, **§10** the Cymbal Bank worked example, **§11** the claims audit — cross-referenced, not re-derived) · [Jump Trading](jump_trading_guide.md) (the immediately preceding firm guide: **§6** the incidents and regulatory matters, **§7** capital where the finding is absence, **§11** the Asia/Singapore reconciliation, **§14** the worked example, **§15** the anti-patterns and claims audit — this guide adopts that discipline) · [Citadel LLC](citadel_llc_guide.md) (**its §7 owns the technology-of-the-firm-type material**) · [ExodusPoint](exoduspoint_guide.md) (the identity and settled-claim recording conventions this guide adopts) · [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) (the fund-management contrast) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) (the Cymbal Bank persona and the worked-example conventions)
> **Companion guides (technology/, prefix `../technology/`):** [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md), [Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md) and [Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md) (the latency discipline itself — cross-ref §6) · [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md) (the platform architecture and market-access structure — cross-ref §6 and §12) · [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) (the quant-research and engineering skill stack — cross-ref §7)

---

**How to use this guide:** Section 1 is the overview — the one-line thesis, what Tower Research Capital is and is not, the key-facts table, the vocabulary decoder applied to this firm, and the boundary declaration naming the guides this one defers to. Section 2 is the identity and the entities — the entity map assembled from the firm's own pages and the DOJ releases, the founding and the founders with their documented prior occupations, the calibration of what this firm does publish against what it does not, and a blunt statement of what the entity map does **not** establish. Section 3 is the history — February 1998 and its context, the reported growth, the dated milestones, and the founder's separately documented public activity including LimeWire, kept exactly distinct from Tower. Section 4 is the business — what a firm of this kind does, the firm's own asset-class and product statements, the crypto team, and the structural point that a proprietary firm discloses no revenue composition. Section 5 is the highest-risk section: **the regulatory record as a dated record of authority, instrument, date, parties and outcome, with the company-level matter and the individual-level matters kept exactly separate.** Section 6 is the technology and the latency posture — the class of infrastructure cross-referenced, then what this firm itself documents and what it does not. Section 7 is talent and culture. Section 8 is the regulatory and market-structure context, cross-referenced to [Hudson River Trading](hudson_river_trading_guide.md) §7 and §8 rather than re-derived. Section 9 is the Asia and Singapore angle, opening with the sibling guide's ownership of the Singapore facts and closing with an explicit reconciliation. Section 10 is the capital and the revenue, where the finding is absence. Section 11 is the peer group and positioning. Section 12 is the bank interface at mechanism level, ending with the list of relationships this guide refuses to name. Section 13 is the Cymbal Bank worked example — fictional, illustrative, and the **only** bank persona in this guide. Section 14 is the anti-patterns. Section 15 is the claims audit (✅/⚠/❌), with §15.4 collecting what could not be verified. Section 16 collects What Could Not Be Verified, the glossary, the cross-references and the closing summary. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`. **Integrity convention:** ✅ = verified this pass against a primary source or a named, dated source; ⚠ = flagged — reported, single-sourced, contested, or not re-verified live; ❌ = refuted or rejected. Nothing in this guide was invented. Where the public record does not disclose something, this guide says so and stops.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [What Tower Research Capital Is, and What It Is Not](#12-what-tower-research-capital-is-and-what-it-is-not)
   - 1.3 [The Key-Facts Table](#13-the-key-facts-table)
   - 1.4 [The Vocabulary Decoder](#14-the-vocabulary-decoder)
   - 1.5 [The Boundary Declaration](#15-the-boundary-declaration)
2. [The Identity, the Entities and the Silence](#2-the-identity-the-entities-and-the-silence)
   - 2.1 [The Entity Table](#21-the-entity-table)
   - 2.2 [The Founding and the Founders](#22-the-founding-and-the-founders)
   - 2.3 [The Opacity Finding, Calibrated](#23-the-opacity-finding-calibrated)
   - 2.4 [The Leaders](#24-the-leaders)
   - 2.5 [What the Entity Map Does Not Establish](#25-what-the-entity-map-does-not-establish)
3. [The History](#3-the-history)
   - 3.1 [February 1998 and Its Context](#31-february-1998-and-its-context)
   - 3.2 [Growth as Reported](#32-growth-as-reported)
   - 3.3 [The Milestones Table](#33-the-milestones-table)
   - 3.4 [The Founder's Separately Documented Public Activity — LimeWire and After](#34-the-founders-separately-documented-public-activity--limewire-and-after)
4. [The Business](#4-the-business)
   - 4.1 [What a Firm of This Kind Does](#41-what-a-firm-of-this-kind-does)
   - 4.2 [The Firm's Own Asset-Class and Product Statements](#42-the-firms-own-asset-class-and-product-statements)
   - 4.3 [The Trading-Team Structure](#43-the-trading-team-structure)
   - 4.4 [The Crypto Team — Limestone Trading](#44-the-crypto-team--limestone-trading)
   - 4.5 [The Ventures Arm](#45-the-ventures-arm)
   - 4.6 [What a Proprietary Firm's Business Description Cannot Contain](#46-what-a-proprietary-firms-business-description-cannot-contain)
5. [The Regulatory Record](#5-the-regulatory-record)
   - 5.1 [How to Read This Section](#51-how-to-read-this-section)
   - 5.2 [The Firm-Level Matter — the DOJ Deferred Prosecution Agreement, 7 November 2019](#52-the-firm-level-matter--the-doj-deferred-prosecution-agreement-7-november-2019)
   - 5.3 [The Parallel CFTC Settlement](#53-the-parallel-cftc-settlement)
   - 5.4 [The 2018 Charging Release — Three Individuals, an Unnamed Firm](#54-the-2018-charging-release--three-individuals-an-unnamed-firm)
   - 5.5 [The Individual-Level Record](#55-the-individual-level-record)
   - 5.6 [The Latour Trading Net-Capital Matter, 2014 — Secondary-Sourced](#56-the-latour-trading-net-capital-matter-2014--secondary-sourced)
   - 5.7 [The Civil Litigation](#57-the-civil-litigation)
   - 5.8 [Registrations and the Asia Regulatory Position](#58-registrations-and-the-asia-regulatory-position)
   - 5.9 [The Dated Record](#59-the-dated-record)
   - 5.10 [What Remains Unresolved or Unverified](#510-what-remains-unresolved-or-unverified)
6. [The Technology and the Latency Posture](#6-the-technology-and-the-latency-posture)
   - 6.1 [The Class of Infrastructure](#61-the-class-of-infrastructure)
   - 6.2 [What This Firm Itself Documents](#62-what-this-firm-itself-documents)
   - 6.3 [The One Documented Vendor Relationship — Torstone, Post-Trade](#63-the-one-documented-vendor-relationship--torstone-post-trade)
   - 6.4 [What Is Not Documented](#64-what-is-not-documented)
7. [Talent and Culture](#7-talent-and-culture)
   - 7.1 [The Class Model](#71-the-class-model)
   - 7.2 [What This Firm Publishes — Roles and Structure](#72-what-this-firm-publishes--roles-and-structure)
   - 7.3 [What This Firm Publishes — Benefits, Values and Hiring Philosophy](#73-what-this-firm-publishes--benefits-values-and-hiring-philosophy)
   - 7.4 [What Is Not Published About the People Model](#74-what-is-not-published-about-the-people-model)
8. [The Regulatory and Market-Structure Context](#8-the-regulatory-and-market-structure-context)
   - 8.1 [Where a Proprietary Trading Firm Sits](#81-where-a-proprietary-trading-firm-sits)
   - 8.2 [The Registration and Designation Questions](#82-the-registration-and-designation-questions)
   - 8.3 [Spoofing as a Market-Conduct Offence](#83-spoofing-as-a-market-conduct-offence)
   - 8.4 [The Record, Not the Statements](#84-the-record-not-the-statements)
9. [The Asia and Singapore Angle](#9-the-asia-and-singapore-angle)
   - 9.1 [What the Sibling Guide Already Owns](#91-what-the-sibling-guide-already-owns)
   - 9.2 [What This Guide Independently Verifies](#92-what-this-guide-independently-verifies)
   - 9.3 [The Wider Asian Footprint](#93-the-wider-asian-footprint)
   - 9.4 [The Regulatory Position in Asia](#94-the-regulatory-position-in-asia)
   - 9.5 [The Reconciliation](#95-the-reconciliation)
10. [The Capital and the Revenue](#10-the-capital-and-the-revenue)
    - 10.1 [The Discipline](#101-the-discipline)
    - 10.2 [What Is Reported, with Outlet and Date](#102-what-is-reported-with-outlet-and-date)
    - 10.3 [The Headcount Figure and Its Discrepancy](#103-the-headcount-figure-and-its-discrepancy)
    - 10.4 [What the Firm's Own Structure Implies, and What It Does Not](#104-what-the-firms-own-structure-implies-and-what-it-does-not)
    - 10.5 [The Absence Is the Finding](#105-the-absence-is-the-finding)
11. [The Peer Group and Positioning](#11-the-peer-group-and-positioning)
    - 11.1 [The Peer Set with Dated Sources](#111-the-peer-set-with-dated-sources)
    - 11.2 [The Older Latency-Led Firms Against the Newer Entrants](#112-the-older-latency-led-firms-against-the-newer-entrants)
    - 11.3 [This Firm's Relative Position, Only as Far as Sources Support](#113-this-firms-relative-position-only-as-far-as-sources-support)
12. [The Bank Interface](#12-the-bank-interface)
    - 12.1 [The Firm as Counterparty — Clearing and Margin](#121-the-firm-as-counterparty--clearing-and-margin)
    - 12.2 [The Firm as a Liquidity Provider to a Bank's Clients](#122-the-firm-as-a-liquidity-provider-to-a-banks-clients)
    - 12.3 [The Firm as a Client of Clearing and Prime Services](#123-the-firm-as-a-client-of-clearing-and-prime-services)
    - 12.4 [The Counterparty-Credit Questions a Bank Asks](#124-the-counterparty-credit-questions-a-bank-asks)
    - 12.5 [The Artefacts a Bank Can Actually Obtain](#125-the-artefacts-a-bank-can-actually-obtain)
    - 12.6 [What This Guide Will Not Name](#126-what-this-guide-will-not-name)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
    - 13.1 [The Scenario](#131-the-scenario)
    - 13.2 [Entity Identification — Which Entity Contracts](#132-entity-identification--which-entity-contracts)
    - 13.3 [The Counterparty-Credit Approach Given Non-Disclosure](#133-the-counterparty-credit-approach-given-non-disclosure)
    - 13.4 [How the Documented Conduct Record Informs the Assessment](#134-how-the-documented-conduct-record-informs-the-assessment)
    - 13.5 [The Recommendation and the Condition](#135-the-recommendation-and-the-condition)
    - 13.6 [Regulatory-Reporting Consequences](#136-regulatory-reporting-consequences)
14. [The Anti-Patterns](#14-the-anti-patterns)
    - 14.1 [The Table — Symptom, Cause, Guardrail](#141-the-table--symptom-cause-guardrail)
    - 14.2 [The Unifying Failure](#142-the-unifying-failure)
15. [The Claims Audit](#15-the-claims-audit)
    - 15.1 [The Verified Claims (✅)](#151-the-verified-claims-)
    - 15.2 [The Flagged Claims (⚠)](#152-the-flagged-claims-)
    - 15.3 [The Rejected or Not-Found Claims (❌)](#153-the-rejected-or-not-found-claims-)
    - 15.4 [What Could Not Be Verified](#154-what-could-not-be-verified)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)
    - 16.1 [What Could Not Be Verified](#161-what-could-not-be-verified)
    - 16.2 [Glossary](#162-glossary)
    - 16.3 [Cross-References](#163-cross-references)
    - 16.4 [Closing Summary](#164-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**For a firm that never explains itself, the record is the regulator's — which is both how this guide knows anything and the reason its limits are the interesting part.**

That sentence is the whole guide in miniature. Tower Research Capital publishes a marketing website of unusual quality and a financial position of exactly nothing: no accounts, no revenue, no capital, no valuation, no ownership percentages, no headcount by entity, no venue list, no latency numbers. What it does publish — offices with street addresses, engineering claims with numbers attached, a liquidity-provision page that names two European legal entities, a ventures portfolio with dated press releases, a leaders page — is enough to establish **what kind of firm this is** and **how it is organised at the edges**. What it is not enough to establish is **how big it is, how it is capitalised, or how it behaves when nobody is watching**. For the parts where the firm is silent, the only first-party document that speaks about Tower Research Capital under its own name, with dates and instruments, is the enforcement record — which is why this guide leans on the United States Department of Justice, and why §5 is both its highest-risk section and its evidential centre of gravity. The limits of that substitution are stated, not smoothed over: a deferred prosecution agreement is a record of conduct and remediation, **not** a financial statement, and it is not a statement about the firm today.

### 1.2 What Tower Research Capital Is, and What It Is Not

Because firms of this class are routinely collapsed by secondary write-ups into "high-frequency trading", the distinctions need stating first.

- **It is a proprietary trading firm.** It trades its own capital, as principal, with no clients, no external fund and no investor base to report to. Its own liquidity-provision page offers "Access Our Liquidity" to counterparties, which makes it a **market-facing counterparty**, not a fiduciary ✅ (tower-research.com/liquidity-provision, retrieved 27 Sep 2026). A counterparty relationship is not a client relationship, and the distinction is the whole of §12.
- **It is an electronic market maker and a market-taker.** The same page describes "on-exchange market making" and "bilateral, disclosed liquidity in the world's most active markets" ✅. The firm's engineering page claims proprietary technology "for every aspect of trading: market access, data management, quantitative research, compute infrastructure, compliance, support, and beyond" ✅ (tower-research.com/engineering, retrieved 27 Sep 2026).
- **It is a New York LLC, not a bank and not an exchange.** Wikipedia's infobox records Tower Research Capital LLC as an LLC in financial services, headquartered in New York City ✅/⚠ (Wikipedia, Tower Research Capital, retrieved 27 Sep 2026 — secondary, and the "LLC" form is also used by DOJ in the 2019 release, which is primary). DOJ's own release names the entity **Tower Research Capital LLC (Tower)** and describes it as "a New York, New York-based financial services firm" ✅ (DOJ Office of Public Affairs Release No. 19-1,208, 7 Nov 2019).
- **It is a group with at least two European regulated-facing entities, and one identifier the firm does not explain.** The liquidity-provision page names "Two Systematic Internalisers: **Tower Research Capital Europe Limited (Tower UK)** and **Tower Research Capital Europe B.V.**" ✅ — and separately lists "**TRCX (US Equities and ETFs)**" without stating what TRCX legally is ⚠ (§2.5).
- **It is a firm with one company-level enforcement matter and three individual-level prosecutions arising from the same conduct window.** The firm-level matter is a **deferred prosecution agreement** announced by DOJ on **7 November 2019**; the individuals are **Kamaldeep Gandhi**, **Krishna Mohan** and **Yuchun "Bruce" Mao** ✅ (DOJ Releases No. 18-1328 and No. 19-1,208). §5 keeps the firm and the individuals in separate tables, because that is the distinction most often lost.
- **It is not a hedge fund.** Wikipedia's article on founder Mark Gorton describes Tower Research Capital LLC as "a hedge fund" ✅/⚠ — that is an error of classification: the firm trades its own capital as a proprietary trading firm ✅ (Wikipedia's own Tower Research Capital article states it "is a proprietary trading firm that develops and operates automated quantitative trading strategies across global financial markets"), and the repository treats it as a proprietary trading firm throughout. The two Wikipedia articles contradict each other; the proprietary-trading classification is the one this guide uses ❌ for the hedge-fund label.

### 1.3 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Full name | Tower Research Capital LLC ("Tower Research") | ✅ DOJ Release No. 19-1,208 (7 Nov 2019); Wikipedia infobox |
| Firm type | Proprietary trading firm / electronic market maker — no clients, no external fund | ✅ firm's own pages; ✅ Wikipedia |
| Founded | February 1998 | ✅ Wikipedia infobox ("February 1998; 28 years ago"), citing CNBC and Business Insider |
| Founders | Mark Gorton and Alistair Brown | ✅ Wikipedia, citing CNBC and Business Insider |
| HQ | New York City; **120 Broadway, 38th Floor, NY 10271** and **377 Broadway, NY 10013** both listed | ✅ tower-research.com/offices (27 Sep 2026) |
| CEO | **Albert An** (succeeded the founder in August 2019; joined the firm 2016 as technology lead) | ✅ firm's own leaders page; ✅/⚠ Wikipedia citing Bloomberg (Abelson & Leising, 1 Aug 2019) |
| Chairman | **Mark Gorton** (founder; stepped down as CEO August 2019) | ✅ firm's own leaders page; ✅/⚠ Wikipedia/Bloomberg |
| Entity set | Tower Research Capital LLC (US); Latour Trading (US subsidiary per Wikipedia) ⚠; Tower Research Capital Europe Limited (Tower UK) and Tower Research Capital Europe B.V. (European SIs) ✅; TRCX (unexplained identifier) ⚠; Tower Research Ventures ✅; a Canada site (tower-research.ca) ⚠; a reported Jersey subsidiary ⚠ | mixed — see §2.1 |
| Employees | "more than 1,100 people worldwide" as of 2025 (firm's About Us, via Wikipedia) vs "c. 1,400+ (2026)" (Wikipedia infobox) | ⚠ discrepancy flagged, §10.3 |
| Offices | Live offices page lists **14 city entries**; Wikipedia says **11 locations**; the firm's engineering page says "**12+ global offices**" | ✅ live page is best evidence; ⚠ count discrepancy, §9 |
| Regulatory registrations | No registration identified at any register this pass; the two European SIs are the firm's own statement ⚠ | see §5.8 |
| Company-level enforcement | DOJ Release No. 19-1,208 (7 Nov 2019): deferred prosecution agreement; US$67.4m combined; parallel CFTC settlement ≈US$67.4m incl. US$24.4m civil monetary penalty | ✅ DOJ |
| Individual-level enforcement | Gandhi and Mohan guilty pleas (Nov 2018); Mao indictment Oct 2018 | ✅ DOJ |
| First-line finding | The firm is legible as a market participant and a counterparty, and opaque as a credit — the division is lawful and common, and it is the whole onboarding problem | ⚠ assessment, not fact |

### 1.4 The Vocabulary Decoder

Each term is defined here as it applies to **this** firm, not in the abstract.

- **The proprietary trading firm.** A firm that trades its own capital as principal, with no clients' assets, no fund structure and no investor reporting. The consequence that matters commercially is that there is **no client franchise between its engineers and its P&L** — technology spend is not a support cost, it is the revenue mechanism (the point is developed for the firm type in [Hudson River Trading](hudson_river_trading_guide.md) §3 and [Jump Trading](jump_trading_guide.md) §4.1 — cross-referenced, not re-derived). Applied to Tower: the firm publishes an engineering organisation and a trading organisation, and no client-facing business at all beyond the liquidity-provision form ✅.
- **The electronic market maker.** A firm that continuously posts two-sided prices electronically and earns the spread, carrying inventory in between. Tower's own words: "on-exchange market making", and "our liquidity provision offering specializes in ensuring efficient and optimized trading experiences for our **counterparties**" ✅ (tower-research.com/liquidity-provision). Note the word the firm uses: counterparties, not clients.
- **Latency arbitrage.** A strategy family that profits from being faster than other participants to react to public information — the label the policy debate applies to firms of this class. **This guide does not attribute latency arbitrage to Tower**, because the firm does not describe its strategies and no authority alleges a specific strategy beyond the spoofing conduct in §5.2 ✅/⚠. What can be said is that the firm claims infrastructure in the latency-sensitive class: "Dozens of colocation centers around the world, where we are constantly tweaking the hardware" ✅ (tower-research.com/engineering). Infrastructure implies latency sensitivity; it does not name a strategy.
- **Co-location.** Renting space and power inside or adjacent to an exchange's matching-engine data centre so that a firm's own servers sit as close as commercially possible to the venue. Tower states "**Dozens of colocation centers**" without naming a venue or a data centre ✅. The count is the firm's own; the location set is not published.
- **The microwave / post-fibre question.** The 2010s race to replace (and then to beat) long-haul fibre with microwave and millimetre-wave links between venues, driven by the physics of signal propagation in air versus glass. **For Tower specifically: not documented.** The firm names no transport technology, no route and no link ✅/⚠ (❌ for any claim that Tower runs a specific named microwave network — see §15.3). This is a case where the genre's secondary literature is full of inference and the firm is silent; §6.4 states the boundary.
- **The spoofing offence, and why it matters to a market maker's reputation.** Spoofing is placing orders with the intent to cancel them before execution, in order to create a false impression of supply or demand. For an ordinary company, a trading offence is a legal fact. For a **market maker**, whose entire franchise is the credibility of the prices it shows, the offence strikes at the product itself: a counterparty's willingness to trade against a displayed quote depends on believing the quote is real. That is why the 2019 matter is not merely a compliance episode in this firm's file — the conduct DOJ describes is conduct in the exact activity the firm sells, and the remediation language DOJ uses (below) is about surveillance and governance of that activity ✅ (DOJ Release No. 19-1,208).
- **The registered entity.** The legal person that signs, holds the licence, bears the risk and appears in a register — as distinct from the brand. For Tower, the register-visible entities are the LLC (named by DOJ), the two European SIs (named by the firm) and whatever entity actually contracts for a given trade — which the firm does not publish for most of its footprint ⚠. Onboarding the brand will fail; the entity map in §2.1 is the work.
- **The systematic internaliser (SI).** A MiFID II designation for an investment firm that executes client orders against its own book on a systematic, frequent basis rather than routing to a venue. The firm states it operates **two** European SIs ✅. Two structural observations follow: (i) SI status is a **regulated status with obligations attached**, so its assertion is a stronger form of disclosure than a marketing sentence; and (ii) the section of the firm that acts as an SI is the one that looks most like a counterparty-facing business, which is why §12 treats European equities separately. Whether each entity's SI status is current was **not verified at a European register this pass** ⚠ (§5.8).

### 1.5 The Boundary Declaration

This guide owns the firm: its identity and entities, its founding and founders, its business lines, its technology posture, its regulatory record, its Asia presence, its peer positioning and its bank interface. It does **not** re-derive four bodies of material that other guides own. **The prop-firm archetype and the market-making franchise explanation belong to [Hudson River Trading](hudson_river_trading_guide.md) §3** — read that first if the economics of a proprietary market maker are new. **The Singapore facts about this firm belong to [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.6**, which this guide cross-references and reconciles with in §9 rather than duplicating. **The adjacent firm profiles and the discipline for recording a settled regulatory matter belong to [Jump Trading](jump_trading_guide.md) §6 and §7 and to [ExodusPoint](exoduspoint_guide.md)**, whose conventions this guide adopts. **The latency discipline itself — the engineering practice of low-latency development — belongs to [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md), [Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md) and [Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md)**, cross-referenced from §6 and never restated here. **The firm-type technology material belongs to [Citadel LLC](citadel_llc_guide.md) §7.** Where this guide names a sibling, it names it rather than copying it.

---

## 2. The Identity, the Entities and the Silence

### 2.1 The Entity Table

The table below contains **every** entity this guide can attach to a source. Nothing on it is inferred from a brand name, a domain name or a logo.

| Entity | Jurisdiction / status | What it is, on the source | Source (retrieved 27 Sep 2026 unless dated) |
| --- | --- | --- | --- |
| **Tower Research Capital LLC** | United States (New York, New York) | The firm; named by DOJ as the party entering the deferred prosecution agreement; described by DOJ as "a New York, New York-based financial services firm" | ✅ DOJ Office of Public Affairs Release No. **19-1,208**, 7 Nov 2019; ✅/⚠ Wikipedia infobox (LLC) |
| **Tower Research Capital Europe Limited** ("Tower UK") | United Kingdom | Named by the firm as one of "two Systematic Internalisers" providing European cash-equity liquidity | ✅ tower-research.com/liquidity-provision |
| **Tower Research Capital Europe B.V.** | Netherlands (B.V. form implies Netherlands, but the page states no jurisdiction) | Named by the firm as the second Systematic Internaliser for European cash equities | ✅ tower-research.com/liquidity-provision; ⚠ jurisdiction inferred from the legal form, not stated on the page |
| **TRCX** | Not stated | Appears on the firm's own liquidity page as "TRCX (US Equities and ETFs)" alongside "ETF Block Trading Platform". Whether TRCX is a venue, an ATS, a broker-dealer identifier or an internal desk name **is not stated anywhere in the sources captured** | ✅ the identifier appears; ⚠ what it is = not established (§2.5) |
| **Latour Trading** | United States (per Wikipedia, a subsidiary) | Wikipedia's infobox lists Latour Trading as a **subsidiary**, and its legal section describes it as "Tower Research's wholly owned subsidiary" | ⚠ Wikipedia only — no firm page or regulator release read this pass confirms the ownership chain (§5.6) |
| **Tower Research Ventures (TRV)** | Not stated | The venture arm: pre-seed, seed and incubation investing; **Jared Young** named as Director of Venture Capital; portfolio list and dated press releases published | ✅ tower-research.com/ventures; dated TRV releases (5 Nov 2025, 9 Oct 2025, 28 May 2025) |
| **Limestone Trading** | Not stated | "One of its internal quantitative trading teams", per Wikipedia's report of a Bloomberg story on crypto market making | ⚠ Wikipedia citing Bloomberg (Anto Antony, 5 May 2025) — secondary, not on the firm's pages captured |
| **Canada site: tower-research.ca** | Canada (Montreal) | The offices page links a **separate Canada site** and lists 2001 Blvd Robert-Bourassa, Montréal | ✅ tower-research.com/offices; ⚠ the Canadian legal entity name is not published in the sources captured |
| **A Jersey subsidiary** | Jersey (channel Islands) — **reported only** | "US trading firm Tower Research plots Jersey subsidiary" | ⚠ Financial News London, Lars Mucklejohn, 21 Apr 2026 — headline-and-citation only, **not verified this pass** |
| **A Singapore entity** | Singapore — **not identified** | Singapore appears as an office with a full address; no Singapore legal entity name appears on any firm page captured | ✅ office; ❌ entity name not found (§2.5, §9) |

**How to read the table.** Three rows are primary: the LLC named by DOJ, the two European SIs named by the firm, and the Singapore/London/New York addresses on the firm's own offices page. Three rows are second-hand and one is a headline: Latour Trading (Wikipedia), Limestone Trading (Wikipedia citing Bloomberg), the Jersey plan (Financial News London). One row is an identifier the firm itself declines to explain: TRCX. **That distribution is the identity finding**, and it is not a criticism — a private LLC has no obligation to publish its group chart, and most of its peers publish less.

### 2.2 The Founding and the Founders

**Founded February 1998 in New York, by Mark Gorton and Alistair Brown** ✅/⚠ (Wikipedia infobox, citing CNBC and Business Insider; the February 1998 date is consistently reported and appears in Wikidata-derived "founded" fields as well, but no firm page or corporate-register extract stating an incorporation date was read this pass).

Documented prior occupations:

- **Mark Gorton — documented, and unusually so.** Before Tower, Gorton was **an electrical engineer at Martin Marietta** (now part of Lockheed Martin) and then **entered fixed-income trading at Credit Suisse First Boston**, prior to launching the Lime Group of companies ✅/⚠ (Wikipedia, Mark Gorton, retrieved 27 Sep 2026, citing HuffPost, D.M. Levine, "Is Speed Trader Mark Gorton Killing Wall Street?", 18 Apr 2012). Education: Bachelor's in Electrical Engineering, Yale; Master's in Electrical Engineering, Stanford; MBA, Harvard ✅/⚠ (same source, education section). His other public activity is a substantial body of documented material and is treated separately in **§3.4**, because it is a personal and Lime Group matter and **not a Tower matter**.
- **Alistair Brown — not documented in the sources captured.** No prior occupation, no biography and no later role is established this pass ⚠. The repository does not assert one, and neither does this guide. This is a genuine gap, not an omission to be filled by inference, and it is recorded again in §15.4 and §16.1.

### 2.3 The Opacity Finding, Calibrated

The genre's default sentence about a firm like this is "it discloses nothing". **On this pass that sentence would be false**, and calibrating it correctly is more useful than repeating it. The honest division:

**Published by the firm (first-party, retrieved 27 Sep 2026):**

- A full **office list with street addresses** across fourteen city entries, including Singapore ✅.
- **Leaders**: Mark Gorton as Chairman and Albert An as Chief Executive Officer ✅.
- **Values**: Excellence, Respect, Innovation, Integrity, Teamwork ✅.
- **Business-unit pages**: Quantitative Trading, Engineering, Liquidity Provision, Ventures ✅.
- **Engineering claims with numbers attached**: "Dozens of colocation centers"; "Hundreds of engineers supporting 12+ global offices and 150+ trading venues"; "Managing 100 petabytes of data"; languages "C++ and Python while strategically investing in Rust"; "machine learning, FPGA technology, low-latency programming" ✅.
- **Product and entity disclosure on the liquidity page**: named product lines, two named European SI entities, and the distribution model ("direct, disclosed relationships, semi-disclosed, and making into all primary and secondary markets") ✅.
- **A ventures portfolio with dated press releases and blog posts** (Procurement Sciences US$30m Series B, 5 Nov 2025; Mentium US$3.2m, 9 Oct 2025; Atomic Canyon US$7m led by Energy Impact Partners, 28 May 2025; prediction-markets blog posts 10 Mar 2026 and 16 Apr 2026) ✅.

**Not published by the firm (the opacity finding):**

- **Any financial statement, revenue, profit, capital, AUM, valuation or ownership percentage** ❌ — no figure of any kind, for any entity, anywhere on the firm's pages captured.
- **The architecture and latency specifics** — no system diagram, no hardware vendor, no switch or NIC or NIC-driver detail, no latency number, no venue name, no data-centre name, no transport technology ✅/❌ (§6.4).
- **Headcount by entity, and the internal team map beyond the four business units** ⚠.
- **The Singapore and wider Asia legal entity or entities** ❌ (§9).
- **The ownership chain** — who owns the LLC, and how the group is held ⚠. Wikipedia's Mark Gorton article asserts "Gorton owns Tower Research Capital LLC" but marks the sentence **citation needed** ⚠ — which is exactly the standard this guide holds.

**And the point that must be said plainly:** this opacity is **lawful and common for a private firm**. A proprietary trading firm trading its own capital has no investors to report to, no listed securities, no client-asset regime and (on the evidence captured) no prudential supervisor publishing its capital. There is no rule requiring it to publish accounts and no register this guide found that would do so for it. The correct professional response is not suspicion — it is to record the absence as a data point (§10.5) and to price it (§13.3). The one thing a bank should **not** do is import a peer's figure, an industry average, or a valuation implied by the size of the firm's offices.

### 2.4 The Leaders

| Role | Person | Evidence |
| --- | --- | --- |
| **Chairman** | **Mark Gorton** — founder | ✅ firm's own About Us / leaders list ("OUR LEADERS"); ✅/⚠ Wikipedia citing Bloomberg (Abelson & Leising, 1 Aug 2019) for the 2019 transition |
| **Chief Executive Officer** | **Albert An** — joined the firm in 2016 and served as its technology lead before succeeding the founder in **August 2019** | ✅ firm's own leaders list; ✅/⚠ Wikipedia/Bloomberg, 1 Aug 2019 |
| Director of Venture Capital | **Jared Young** | ✅ tower-research.com/ventures |
| Chief Operating Officer | **Alan McGroarty** — quoted on the firm's growth and diversification: "Over the recent years we have enjoyed unprecedented levels of growth… As we continue to diversify into new asset classes and trading strategies, at ever increasing volumes, we need a flexible scalable solution." | ✅ A-Team Insight, "Tower Research Selects Torstone for Global Cross-Asset Post-Trade Services", **19 May 2022** |

Two observations. First, the **CEO transition is the firm's clearest dated governance fact**: a founder who built the firm from 1998 stepped back in 2019 into the chair, and a technology-side executive took over ✅/⚠. That is a legible succession, and it predates most of what the firm now publishes. Second, **the COO quote is the only firm-attributed statement about growth this guide could source that contains no number** — "unprecedented levels of growth" is the firm's own characterisation of itself, on the record, in a dated trade-press article ✅. It is worth as much as it is worth, and no more: it is a first-party qualitative claim with a date, and it is **not** a capital fact (§10).

### 2.5 What the Entity Map Does Not Establish

Stated bluntly, so that nothing on the list above is mistaken for more than it is:

- **The Singapore legal entity is not identified.** The office address (Marina One West Tower, 9 Straits View, Singapore 018937) is published ✅; the name of the Singapore company, its incorporation date and its registration number are **not** on any source captured ❌. No ACRA search was performed this pass, and the host's `web_search` is non-functional (recorded as a tool limitation, §16.1) — so the absence here is **the absence of a search**, not evidence that no entity exists.
- **The Jersey entity is unverified.** A Financial News London headline of 21 Apr 2026 reports a planned Jersey subsidiary ⚠. This guide records the report; it does **not** record a company, a date of incorporation or a purpose.
- **Ownership percentages are unknown.** Who holds what across the LLC, the two European entities, the ventures arm and any Asia entity is **not public** ❌. Wikipedia's assertion that Gorton owns the LLC carries a "citation needed" tag ⚠.
- **The internal team structure is only partly mapped.** Wikipedia describes "internal trading teams" that are "independent from one another, enjoying autonomy while accessing shared technology resources such as hardware, software, and connectivity as well as resources such as business management, legal, compliance, and risk management" ✅/⚠ — that is a structural statement with a secondary source. The **number** of teams, their strategies and their P&L attribution are not published ❌. Limestone Trading is a named team ⚠; it is the only one this guide can name.
- **What "TRCX" is, is not established.** The identifier appears on the firm's own liquidity page ✅. Whether it is an ATS, a venue, a broker-dealer or a desk label cannot be resolved from the sources captured ⚠, and this guide does not guess.
- **Latour Trading's current status is not established.** Wikipedia describes it as a wholly owned subsidiary and attaches the 2014 net-capital matter to it ⚠. No register entry, no SEC release and no firm statement was retrieved this pass; §5.6 presents the underlying claim exactly as what it is — secondary-sourced.
- **Whether the group has a bank or broker-dealer entity in the United States that faces clients is not established** ❌. The firm's own pages describe liquidity provision and market access, not a client franchise; and the entity named by the regulator is the LLC, not a registered broker-dealer.

---

## 3. The History

### 3.1 February 1998 and Its Context

Tower Research Capital was founded in **February 1998** ✅/⚠ (Wikipedia, citing CNBC and Business Insider) — which places it, on Wikipedia's own assessment, among "one of the oldest automated trading firms" ✅/⚠ (Bloomberg, 1 Aug 2019, via Wikipedia). The context matters for a bank's file in three ways.

**First, it is early.** 1998 is before electronic trading became the default venue structure in US equities, and well before the 2010 policy wave that produced the current market-structure debate. A firm founded in 1998 and still trading in 2026 has therefore lived through the entire arc from floor-adjacent electronic markets to co-located, hardware-accelerated markets — and has outlived most of its cohort.

**Second, its founder's background was not a trading-desk background.** The documented biography puts Gorton in electrical engineering and then fixed-income trading before the Lime Group of companies ✅/⚠ (§2.2) — an engineering-first path, which is consistent with the firm's self-description as a technology organisation whose clients are its own trading teams ("Our team functions more like a software company than a traditional financial organization, with Tower's quantitative trading teams as our clients. We build almost everything in-house" ✅, tower-research.com/engineering).

**Third, it started small and silent.** There is no founding-round announcement, no early client and no early regulator record in the sources captured; the first firm-specific regulatory document this guide can date is from 2014 (§5.6, secondary) and the first primary authority document is DOJ's 2018 charging release, which **does not name the firm** (§5.4). The firm's own website now frames the same period as longevity: "**Over 25 years**" ✅ (tower-research.com/about-us, retrieved 27 Sep 2026).

### 3.2 Growth as Reported

Growth at this firm is documented almost entirely as **footprint** rather than as money, which is exactly what §10 predicts. What can be dated: **headcount** — the firm's About Us page stated "more than 1,100 people worldwide" as of 2025 ⚠/✅ (via Wikipedia, which cites the page), while Wikipedia's infobox says "c. 1,400+ (2026)" ⚠, both recorded and flagged in §10.3; **engineering scale** — "hundreds of engineers supporting 12+ global offices and 150+ trading venues" and "managing 100 petabytes of data" ✅ (tower-research.com/engineering), the firm's own numbers and the closest thing to a scale statement it makes; **presence** — fourteen city entries on the live offices page ✅ against Wikipedia's eleven ⚠ and the firm's own "12+ global offices" ⚠ (§9); **real estate** — in **2023** Tower leased **121,903 square feet at 120 Broadway**, consolidating two New York offices into the Equitable Building ✅/⚠ (Commercial Observer, Mark Hallum, 6 Sep 2023; Bisnow, Ciara Long, 6 Sep 2023, via Wikipedia), while the live offices page still lists **377 Broadway** ✅ — flagged, not resolved (§9.2); and **diversification** — the COO's 2022 statement that "we continue to diversify into new asset classes and trading strategies, at ever increasing volumes" ✅ (A-Team Insight, 19 May 2022), the firm's own description of the direction its liquidity-provision product lines now evidence.

### 3.3 The Milestones Table

| Date | Event | Status |
| --- | --- | --- |
| **February 1998** | Firm founded in New York by Mark Gorton and Alistair Brown | ✅/⚠ Wikipedia (citing CNBC, Business Insider) |
| **2000** | Founder Mark Gorton creates **LimeWire** — a Lime Group matter, **not** a Tower matter (§3.4) | ✅/⚠ Wikipedia (Mark Gorton), citing 2010–2011 press |
| **1999–2010s** | Firm trades through the electronic transition; no dated firm-specific milestones published this pass | ⚠ gap |
| **2010** | *Arista Records LLC v. Lime Group LLC* finds Lime Group and Gorton personally liable; LimeWire shut down by permanent injunction; settled 2011 for US$105m — **again, not a Tower matter** (§3.4) | ✅/⚠ WSJ (Chad Bray, 26 Oct 2010); CNET (Greg Sandoval, 12 May 2011) |
| **≈March 2012 – December 2013** | The conduct window DOJ describes in the deferred prosecution agreement: a single Tower trading team places orders with intent to cancel in E-Mini S&P 500, E-Mini NASDAQ 100 and E-Mini Dow futures | ✅ DOJ Release No. 19-1,208 |
| **Early 2014** | DOJ records that Tower "swiftly moved in early 2014 to terminate the three traders" | ✅ DOJ Release No. 19-1,208 |
| **17 Sep 2014** | Latour Trading reported fined US$16m for net-capital rule violations (secondary-sourced) | ⚠ WSJ, Scott Patterson, 17 Sep 2014 |
| **12 Oct 2018** | DOJ announces three traders charged; **two agree to plead guilty**; the firm is referred to as "Trading Firm A" and **not named** | ✅ DOJ Release No. **18-1328** |
| **6 Nov 2018** | Criminal information filed in the Southern District of Texas against the firm, one count of commodities fraud (the day before the announcement) | ✅ DOJ Release No. 19-1,208 ("filed yesterday") |
| **2 & 6 Nov 2018** | Gandhi pleads guilty (two counts); Mohan pleads guilty (one count) | ✅ DOJ Release No. 19-1,208 |
| **7 Nov 2019** | DOJ announces the deferred prosecution agreement with Tower Research Capital LLC; US$67.4m combined; parallel CFTC settlement announced the same day | ✅ DOJ Release No. 19-1,208 |
| **August 2019** | Albert An succeeds Mark Gorton as CEO; Gorton becomes chairman | ✅/⚠ Bloomberg, 1 Aug 2019, via Wikipedia |
| **3 Feb 2021** | Reported US$15m settlement of a class action | ⚠ financefeeds.com, Andrew Sax-McLeod — secondary |
| **22 Jun 2021** | Second Circuit reported to uphold Tower's win in the Korean futures case (trading on the Korean exchange not subject to the US Commodity Exchange Act) | ⚠ Reuters, Jody Godoy |
| **9 Jul 2021** | Reported sentencing of an ex-Tower trader who avoided jail — **trader not named here** | ⚠ Law360, Rachel Scharf |
| **19 May 2022** | Firm selects **Torstone Technology** for global cross-asset post-trade processing; COO Alan McGroarty quoted | ✅ A-Team Insight |
| **2023** | Lease of 121,903 sq ft at 120 Broadway consolidates two NYC offices | ✅/⚠ Commercial Observer and Bisnow, 6 Sep 2023 |
| **2025** | Reported expansion of crypto trading through the internal team **Limestone Trading** | ⚠ Bloomberg, Anto Antony, 5 May 2025, via Wikipedia |
| **21 Apr 2026** | Reported plan for a **Jersey subsidiary** | ⚠ Financial News London, Lars Mucklejohn |
| **19 Jun 2026** | Reported fixed-income ETF expansion | ⚠ Financial Times, Jill Shah — headline only, paywalled |

### 3.4 The Founder's Separately Documented Public Activity — LimeWire and After

**This subsection concerns Mark Gorton personally and the Lime Group of companies. It is not a Tower Research matter, no source read this pass links it to Tower's conduct or operations, and nothing in it should be entered in a counterparty file as though it were. It is included because the dispatch asks for the founder's documented public record and because the repository contains no LimeWire material at all — this is new evidence, presented with its sources.**

- **LimeWire.** Mark Howard Gorton is the **creator of LimeWire, a peer-to-peer file-sharing client for the Java platform**, and chief executive of the **Lime Group**; Lime Group, based in New York, owns LimeWire as well as **Lime Brokerage LLC** (a stock brokerage), **Tower Research Capital LLC** and **LimeMedical LLC** ✅/⚠ (Wikipedia, Mark Gorton, retrieved 27 Sep 2026; primary-adjacent support for the Lime Group description is thin, and this guide therefore marks the ownership statement ⚠ — see the Wikipedia inconsistency noted below). **Gorton created LimeWire in 2000** ✅/⚠ (same).
- **The copyright case.** Gorton was a key figure in ***Arista Records LLC v. Lime Group LLC***, which in **2010** found Lime Group and Gorton **personally liable for copyright infringement** facilitated by LimeWire, with a permanent injunction to shut LimeWire down that year ✅/⚠ (WSJ, Chad Bray, 26 Oct 2010, via Wikipedia); the case was later **settled in 2011 with Gorton paying US$105 million to the RIAA** ✅/⚠ (CNET, Greg Sandoval, 12 May 2011, via Wikipedia). The sources for these two items are named press with dates; the case documents themselves were not retrieved this pass.
- **The Wikipedia inconsistency, flagged.** The Mark Gorton article describes Tower Research Capital LLC as "**a hedge fund**" ✅/⚠. That is wrong — Tower is a proprietary trading firm, as Wikipedia's own Tower Research Capital article states ✅ — and the error is recorded here so that a reader who follows the citation chain is not misled (§1.2, §15.2).
- **Civic and advocacy activity, stated neutrally and with dates.** Gorton backed the New York City Streets Renaissance Campaign in 2005 (an initiative associated with Streetsblog and Streetfilms), founded the non-profit **OpenPlans** in 1999 (which developed GeoServer), was at one point the single largest supporter of Transportation Alternatives, and was named by *Utne Reader* in 2009 among "50 visionaries who are changing your world" ⚠ (all via Wikipedia, sourcing Streetsblog/Streetfilms and Utne Reader). In May 2023 he was reported as a supporter of Children's Health Defense ✅/⚠ (CNBC, Brian Schwartz, 18 May 2023, via Wikipedia). **In May 2025 he launched the MAHA Institute, which he co-presides alongside Tony Lyons** ✅/⚠ (STAT, Daniel Payne, 15 May 2025, via Wikipedia), and at a 2026 MAHA Institute roundtable he was reported to have said that "the childhood vaccination schedule needs to be eliminated" ✅/⚠ (NOTUS, Margaret Manto, 9 Mar 2026, via Wikipedia). These are reported public positions of the firm's founder and chairman, recorded because they are documented and dated. **No source read this pass links them to Tower Research Capital's business, personnel or policies, and this guide draws no such link.**

---

## 4. The Business

### 4.1 What a Firm of This Kind Does

A proprietary trading firm of Tower's class does three things, and the firm's own pages describe all three without ever using the word "strategy".

1. **It makes markets.** It posts continuous two-sided prices electronically and earns the spread, carrying inventory between trades. Tower's words: "on-exchange market making" and "bilateral, disclosed liquidity in the world's most active markets" ✅ (tower-research.com/liquidity-provision).
2. **It takes risk on purpose and warehouses it.** The liquidity page states plainly: "Our ability to **warehouse many different sizes of risk** and **multiple types of trade flow** means we're prepared to meet our counterparties' precise liquidity needs" ✅. Warehousing risk is the commercial substance of market making: the firm buys because it expects to be able to lay the position off at a better price than it paid, not because it wants the position.
3. **It builds the infrastructure that does both.** "Proprietary technology for every aspect of trading: market access, data management, quantitative research, compute infrastructure, compliance, support, and beyond" ✅; "We build almost everything in-house" ✅ (tower-research.com/engineering).

What it does **not** do, on the evidence captured: it does not manage money for anyone, does not run a fund, does not custody assets for third parties, and publishes no client-facing service other than the liquidity-provision enquiry form ✅/⚠. The absence of a client franchise is not a small detail — it is the reason the credit analysis in §12 and §13 has to be built from substitutes, because there is no client flow and no client-asset regime to give the firm a supervised balance sheet.

### 4.2 The Firm's Own Asset-Class and Product Statements

This is the firm's own disclosure on its liquidity-providing business, quoted as written ✅ (tower-research.com/liquidity-provision, retrieved 27 Sep 2026):

| Asset class | The firm's own product statement | Structural reading |
| --- | --- | --- |
| **US equities and ETFs** | "US cash equities and domestic equity ETFs, with unique block trading capabilities" — listed as **TRCX (US Equities and ETFs)** and an **ETF Block Trading Platform** | The US cash-equity business is offered through an identifier the firm does not define ⚠ (§2.5); the **block** capability is a distinguishing product claim — block liquidity is a relationship product, not a latency product |
| **European equities** | "Cash equities across all major markets" — "Two Systematic Internalisers: **Tower Research Capital Europe Limited (Tower UK)** and **Tower Research Capital Europe B.V.**" | The only place the firm names regulated-facing legal entities ✅. SI status is a MiFID II designation, so this is the firm's most register-legible disclosure ⚠ (not verified at a register this pass) |
| **Global foreign exchange** | "Global liquidity provision in OTC Spot FX and Precious Metals" — "Distributed through multiple avenues: direct, disclosed relationships, semi-disclosed, and making into all primary and secondary markets" | FX and metals are **OTC**, which means bilateral credit, documentation and margin — the segment where a bank is most likely to be a counterparty (§12) |
| **Distribution posture** | "Competitive. Customized. Cross-Asset."; "AN ANALYTICAL APPROACH"; "Customizable Liquidity Streams" | The firm sells **customisation and discretion** (disclosed vs semi-disclosed) rather than pure streaming speed |
| **Counterparty posture** | "Day-to-day dialogue with counterparties to discuss their challenges, share ideas, and assess results"; "reviewing pricing decisions, performing peer analysis, and optimizing our methodology" | The relationship the firm describes is a **counterparty review cycle** — the artefacts a bank should insist on (§12.5) |

Two honest caveats. **First**, this is a marketing page: it names asset classes and a distribution model, and it publishes no volumes, no spreads, no uptime, no fill statistics and no client list ✅/❌. **Second**, the page says nothing about the firm's other trading activity — quantitative trading generally, futures, options, crypto, or the strategies behind them. The liquidity page is the **counterparty-facing slice** of a much wider business, and the rest of it is not described anywhere this guide can find ❌.

### 4.3 The Trading-Team Structure

The firm's organisational design is the one structural fact about its business that is documented, and it comes from a secondary source with a primary-ish chain:

> Tower's internal trading teams are independent from one another, enjoying autonomy while accessing shared technology resources such as hardware, software, and connectivity as well as resources such as business management, legal, compliance, and risk management. ✅/⚠ (Wikipedia, Tower Research Capital, retrieved 27 Sep 2026)

Three consequences follow for anyone reading the firm. **It is a platform, not a monolith**: the firm supplies infrastructure, capital and control functions, and the trading teams supply ideas and P&L. **Its technology claims are therefore corporate claims** — the "hundreds of engineers" and "100 petabytes" describe a shared service organisation serving multiple internal businesses ✅. And **a single team's failure is not the platform's failure** — which is precisely the shape of the 2019 matter: DOJ's own documents describe "three traders who were members of **a single trading team at Tower**" ✅ (DOJ Release No. 19-1,208). That sentence is the most important structural fact in this guide, because it is the difference between a platform's controls failing and a platform being a fraud (see §5.2 and §13.4).

| Structural element | What is documented | What is not |
| --- | --- | --- |
| Trading teams | Multiple internal teams, autonomous, sharing shared tech and control resources ✅/⚠ | How many teams; their strategies; their P&L attribution ❌ |
| Named team | Limestone Trading (crypto) ⚠ | Any other team name ❌ |
| Shared engineering function | "Hundreds of engineers"; four business-unit pages ✅ | Team-by-team allocation ⚠ |
| Control functions | "business management, legal, compliance, and risk management" as shared resources ✅/⚠ | Headcounts and reporting lines ❌ |

### 4.4 The Crypto Team — Limestone Trading

**Documented**: in **2025** Tower "increased its investment in cryptocurrency trading through **Limestone Trading**, one of its internal quantitative trading teams", expanding capital allocated to crypto strategies and upgrading infrastructure to increase its role as a **market maker on global cryptocurrency exchanges** ⚠ (Wikipedia, citing Bloomberg, Anto Antony, **5 May 2025**).

**How to hold this**: it is a **single secondary source reporting a dated business decision**, and it is the only crypto-related fact this guide can attach to the firm. The firm's own pages captured this pass do not mention Limestone Trading, crypto or digital assets at all ✅/❌, which is itself informative: the counterparty-facing disclosure (liquidity provision) is equities and FX/metals only, while the crypto activity sits behind it. For a bank, three implications follow, and they are the same three that apply to any expansion by a firm of this class: **a new asset class means new counterparties and new credit exposure** (crypto venues and their margin regimes); **it means new operational risk** (wallet and key management, 24/7 markets, venue outages); and **it means the firm's aggregate risk profile has changed in a way that is invisible in any published document** ⚠. Note also that this is one of only two dated statements about the firm's direction from the last two years — the other is the reported Jersey plan (§2.1).

### 4.5 The Ventures Arm

**Tower Research Ventures (TRV)** is the firm's venture arm, disclosed on the firm's own pages ✅. It describes itself as investing at **pre-seed, seed and incubation** stage, and names **Jared Young** as Director of Venture Capital ✅ (tower-research.com/ventures, retrieved 27 Sep 2026). Dated first-party activity:

| Date | Item | Status |
| --- | --- | --- |
| **28 May 2025** | Atomic Canyon US$7m round led by Energy Impact Partners (TRV participating) | ✅ TRV press release |
| **9 Oct 2025** | Mentium US$3.2m | ✅ TRV press release |
| **5 Nov 2025** | Procurement Sciences US$30m Series B | ✅ TRV press release |
| **10 Mar 2026** | Blog: "Prediction Markets Opportunity" Part 1 | ✅ TRV blog |
| **16 Apr 2026** | Blog: "Prediction Markets Opportunity" Part 2 | ✅ TRV blog |
| **23 Jul 2026** | Blog: The American Baby Company | ✅ TRV blog |

The published portfolio spans crypto (Alliance, a crypto accelerator; Quantstamp; Zibra Labs), market structure and trading technology (**Sk3W Technologies**, described as a firm that "levels the latency playing field and provides transparent and auditable access into the trading ecosystem"; **TXSE**), and a wide set of non-financial companies (Assembli, Atomic Canyon, Caseblink, Guard Owl, Magna — marked Acquired, Mentium, Pine Sports — marked Acquired, Procurement Sciences, Salesdraft, SharpSports, Snarkify, Spline Data, Sporttrade, Asymptote Labs, Arvor Insurance, Coplay — marked Acquired, The American Baby Company) ✅ (tower-research.com/ventures).

**Why a bank should care, and why it should not over-read it.** It cares because the portfolio is evidence of the firm's **market-structure world view** — an atomic-clock-and-audit-trail company (Sk3W) and a new US regional exchange (TXSE) sit inside it, which is a position on how markets should be built and measured ✅. It should **not** over-read it, because TRV is a venture investor: a portfolio company is not a subsidiary, not a client and not a counterparty, and **nothing in this repository may assert that any portfolio company trades with, clears through, or is a counterparty of Tower Research** ❌. Note also that the venture arm is the one part of the group that produces **dated, attributable, first-party announcements** — which is worth remembering the next time a bank's file says the firm "publishes nothing" (§2.3).

### 4.6 What a Proprietary Firm's Business Description Cannot Contain

The structural point, cross-referenced and not re-derived: a proprietary trading firm has no clients and no external investors, so its business description cannot contain **revenue composition** — no segment splits, no client-revenue concentration, no asset-gathering, no fee lines. The explanation of why that is structural rather than evasive, and what a bank should read instead, is developed for the firm type in **[Hudson River Trading](hudson_river_trading_guide.md) §3** and for the no-clients model specifically in **[Jump Trading](jump_trading_guide.md) §4.1 and §4.4** — read those sections rather than looking for the material here. What is specific to Tower is only this: **the firm does publish product lines** (US equities/ETFs, European equities, FX and precious metals — §4.2) but publishes **no volume, no share, no revenue and no margin for any of them** ✅/❌. The product map tells a bank **what the firm trades**; it tells it **nothing about how much it earns doing so**, and no amount of reading will change that, because the document that would contain the answer is not required to exist.

---

## 5. The Regulatory Record

### 5.1 How to Read This Section

This section is the guide's highest-risk material and its most disciplined. Four rules govern everything below:

1. **Every matter carries authority, instrument, date, parties and outcome** — or it does not appear.
2. **Company-level and individual-level matters are kept in separate subsections and separate tables.** They arose from the same conduct window and they are legally distinct.
3. **The authority's own language is quoted where it matters**, including the authority's own warnings about the status of allegations.
4. **Nothing is reconstructed from memory.** Where this guide could not reach the authority's own document this pass, it says so and marks the item ⚠ — most importantly for the 2014 Latour Trading matter (§5.6), where sec.gov was unreachable.

**A note on what this section is, and is not.** A regulatory record is evidence about **controls, governance and remediation behaviour at a point in time**. It is not a financial statement, it is not a statement about the firm today, and it is not a licence to treat a concluded matter as an open allegation. §13.4 works through how a bank uses this section correctly.

### 5.2 The Firm-Level Matter — the DOJ Deferred Prosecution Agreement, 7 November 2019

**Authority:** U.S. Department of Justice, Office of Public Affairs. **Instrument:** press release No. **19-1,208**, Thursday **7 November 2019**, titled "Tower Research Capital LLC Agrees to Pay $67 Million in Connection With Commodities Fraud Scheme" (justice.gov/opa/pr/tower-research-capital-llc-agrees-pay-67-million-connection-commodities-fraud-scheme), reporting a **deferred prosecution agreement (DPA)** and the criminal information filed the previous day in the **Southern District of Texas**. **Party:** **Tower Research Capital LLC**. ✅ (retrieved at justice.gov, 27 Sep 2026).

The release's own opening, verbatim:

> "Tower Research Capital LLC (Tower), a New York, New York-based financial services firm has entered into a resolution with the Department of Justice to resolve criminal charges related to a scheme involving thousands of episodes of unlawful trading activity in U.S. commodities markets by three former traders." ✅

The instrument, in the release's own words:

> "Tower entered into a **deferred prosecution agreement (DPA)** in connection with a **criminal information filed yesterday in the Southern District of Texas charging the company with one count of commodities fraud**." ✅

The terms and the conduct, as the release states them:

| Element | What DOJ's release says |
| --- | --- |
| Monetary terms | A combined **US$67.4 million** in criminal monetary penalties, criminal disgorgement and victim compensation, with the criminal monetary penalty **credited for any payments made to the CFTC** ✅ |
| Compliance terms | Tower agreed to conduct **reviews of internal controls, policies and procedures** and to modify its compliance programme to ensure it is designed to **deter and detect violations of the Commodity Exchange Act and the commodities fraud statute** ✅ |
| Conduct window | "from approximately **March 2012 until December 2013**" ✅ |
| Actors | "**three traders who were members of a single trading team at Tower**" ✅ |
| Markets | **E-Mini S&P 500** and **E-Mini NASDAQ 100** futures (CME) and **E-Mini Dow** futures (CBOT) ✅ |
| Conduct | On thousands of occasions the traders placed orders "with intent to cancel before execution", "injecting false and misleading information about genuine supply and demand into the market and deceiving other participants including by making them believe the visible order book accurately reflected market-based supply and demand" ✅ |
| Procedural posture | DOJ and Tower filed a **joint motion, subject to court approval, to defer for the term of the DPA any prosecution and trial of the criminal information** ✅ |
| Resolution factors | Tower's collaboration with the United States and its **extensive remedial efforts** ✅ |
| Remediation, verbatim | "Tower also **swiftly moved in early 2014 to terminate the three traders**, made significant investments in sophisticated trade surveillance tools, increased legal and compliance resources, revised the company's corporate governance structures and changed its senior management." ✅ |
| Victim information | DOJ victim-witness page: justice.gov/criminal-vns/case/tower-research-dpa ✅ (URL given in the release) |

**The admission question — handled precisely, because it is where secondary write-ups go wrong.** DOJ's release describes a **DPA that defers prosecution** for its term, subject to court approval. It does **not** state that Tower admitted the allegations, and it does not state that Tower entered the agreement "without admitting or denying" them either — that formulation appears in other releases and **does not appear here** ⚠. This guide therefore does not adopt either gloss: it records exactly what DOJ wrote. Anything else a reader believes about admissions is not in this source.

**What the release does not contain**: no case number for the DPA (the criminal information is described but not numbered in the text captured), no court docket reference, no judge named in the firm-level matter, no term length stated, and no statement about the firm's conduct after the DPA term ✅/❌. Nothing on that list is invented here.

### 5.3 The Parallel CFTC Settlement

Reported in the same DOJ release ✅ (DOJ Release No. 19-1,208, 7 Nov 2019), which states that the CFTC "announced today a separate settlement with Tower in connection with a related, parallel proceeding". As DOJ describes it:

| Element | What DOJ's release states about the CFTC settlement |
| --- | --- |
| Total | Approximately **US$67.4 million**, agreed by Tower ✅ |
| Composition | Includes a **civil monetary penalty of US$24.4 million**, plus **restitution and disgorgement** that will be **credited for any such payments made to DOJ** ✅ |
| Instrument | A **CFTC order**, which also imposes **remedial and cooperation obligations** in connection with any CFTC investigation pertaining to the underlying conduct ✅ |
| Referral | "The CFTC's Division of Enforcement referred the matter to the Department and provided assistance in this matter." ✅ |

**Flag ⚠ on provenance:** the CFTC's own release and order were **not retrieved this pass** — cftc.gov returned a retrieval failure and the host's web search is non-functional (§16.1). The CFTC settlement is therefore recorded here **as DOJ describes it**, with DOJ as the named authority for the description. The CFTC order's own release number, date of entry and docket are **not asserted** ❌. The credit-payment mechanism in the table above also explains why any bank comparing "$67.4m + $67.4m" is double-counting the exposure: the two payments are **cross-credited** ✅.

### 5.4 The 2018 Charging Release — Three Individuals, an Unnamed Firm

**Authority:** U.S. Department of Justice, Office of Public Affairs. **Instrument:** press release No. **18-1328**, Friday **12 October 2018**, "Three Traders Charged, and Two Agree to Plead Guilty, in Connection with over $60 Million Commodities Fraud and Spoofing Conspiracy" ✅ (justice.gov/opa/pr/three-traders-charged-and-two-agree-plead-guilty-connection-over-60-million-commodities-fraud).

**This release does not name Tower Research.** It refers to **"Trading Firm A"**, and to a second Chicago firm as **"Trading Firm B"** for conduct attributed to one individual ✅. The firm's name appears in the **2019** release only. That sequencing is a fact a bank's file must reflect: **any document that cites the firm's conduct window with a 2018 date and a named firm is citing something DOJ did not publish.**

Facts from the 2018 release that stand on their own:

- Alleged conduct window "in or around **March 2012 through March 2014**" ✅ (note the mismatch with the DPA's March 2012 – December 2013 window for the firm-level conduct; both are recorded as stated, and the difference is not explained by the sources captured ⚠).
- Markets: **E-Mini S&P 500** and **E-Mini NASDAQ 100** futures on the CME, **E-Mini Dow** futures on the CBOT ✅.
- "Thousands of orders"; market participants incurred losses of **over US$60 million** ✅.
- The release's own caution, verbatim: "The charges in the indictment and the two criminal informations are **merely allegations**, and the defendants are **presumed innocent until proven guilty beyond a reasonable doubt in a court of law**." ✅
- Investigative and prosecutorial parties as stated: Assistant Attorney General **Brian A. Benczkowski** (Criminal Division); U.S. Attorney **Ryan K. Patrick** (SDTX); Trial Attorneys **Mark Cipolletti**, **Jeffery Le Riche** and **Matthew Sullivan**; Assistant U.S. Attorney **John Lewis**; the **FBI Chicago Field Office** investigated; the **CFTC Division of Enforcement** provided substantial assistance ✅.

### 5.5 The Individual-Level Record

Everything in this subsection is about **individuals**, not the firm. It is kept separate deliberately, and the separation is substantive: the individuals **pleaded guilty**; the firm entered a **deferred prosecution agreement** whose release does not address admissions (§5.2) ✅/⚠.

| Individual (age as stated) | Instrument | Charges | Outcome as stated by DOJ | Date |
| --- | --- | --- | --- | --- |
| **Yuchun "Bruce" Mao**, 39 (2018 release) / 40 (2019 release), a citizen of the People's Republic of China; alleged co-head of a trading team working in Chicago and New York | **Indictment** (SDTX) | One count of conspiracy to commit commodities fraud, two counts of commodities fraud, two counts of spoofing ✅ | As of the 2019 release: "The Department obtained an indictment against Mao in October 2018 with charges **pending** in the SDTX. An indictment is merely an allegation and all defendants are presumed innocent until proven guilty beyond a reasonable doubt in a court of law." ✅ — **no disposition is stated** ❌ | Indicted Oct 2018 |
| **Kamaldeep Gandhi**, 36 (2018) / 37 (2019), of Chicago / New York | **Criminal information** | Two counts of conspiracy to engage in wire fraud, commodities fraud and spoofing ✅ (the 2018 release notes count two concerns conduct "from in or around **May 2014 to October 2014**" at a second Chicago firm, "**Trading Firm B**") | **Pleaded guilty on 2 November 2018** to two counts; sentencing scheduled for **7 February 2020** before SDTX U.S. District Judge **Ewing Werlein Jr.** ✅ | Plea 2 Nov 2018 |
| **Krishna Mohan**, 33 (2018) / 34 (2019), of New York | **Criminal information** | One count of conspiracy to engage in wire fraud, commodities fraud and spoofing ✅ | **Pleaded guilty on 6 November 2018** to one count; sentencing scheduled for **13 February 2020** before SDTX U.S. District Judge **Gray H. Miller** ✅ | Plea 6 Nov 2018 |

**What this table does not say, and will not be made to say.** It does not state what sentence any individual received — the releases state **scheduled sentencing dates**, not outcomes, and no later source was retrieved this pass ❌. It does not connect the later reported sentencing item in §5.7 to any named individual ⚠, because the report does not name one and this guide will not guess which trader it was. And it does not convert the individual conduct into a firm admission: DOJ's firm-level document describes conduct "by three former traders", **members of a single trading team**, and records the firm's termination of the three in early 2014 as a **mitigating factor** ✅. Both halves of that sentence matter, and a file that keeps only one of them is wrong.

### 5.6 The Latour Trading Net-Capital Matter, 2014 — Secondary-Sourced

**What is claimed.** Wikipedia states: "In 2014 Tower Research's wholly owned subsidiary **Latour Trading**, which sometimes accounted for 9% of U.S. stock trading, was fined a record **$16 million** for violating the **net capital rule**. Latour deliberately mis-estimated its exposure to risk and traded despite not holding enough capital." ⚠ — citing **WSJ, Scott Patterson, "High-Frequency Trading Firm Latour to Pay $16 Million SEC Penalty", 17 September 2014** ✅/⚠ (citation chain read; the WSJ article itself was not retrieved).

**Verification attempt, recorded honestly.** This matter must be verified at the SEC's own release or order before it is entered in a file as a securities-enforcement fact. On this pass, **sec.gov could not be retrieved** — the EDGAR full-text search endpoint, an SEC press-release URL and the SEC administrative-proceedings index were each attempted and each failed through the extraction tool (three failures, recorded in §16.1). No alternative route was available because the host's web search is non-functional. **Consequence:** this matter is presented **as secondary-sourced and flagged ⚠**, attached to a **named journalist and date**, and with **no SEC release number, no administrative-proceeding file number, no order date, no respondent entity number and no additional term of the order asserted** ❌. The words in the claim that this guide will not reproduce as established fact are the characterisations of intent ("deliberately mis-estimated") — those are the secondary source's words about the alleged conduct, and a bank's file should treat them as such until the primary order is obtained.

**What can be said structurally, without the primary document.** If the claim is correct, its significance is in three parts: (i) it would be a **securities** matter (net capital), distinct from the **commodities** matter in §5.2; (ii) it would be attributed to a **subsidiary**, not to Tower Research Capital LLC — the same entity-separation discipline as §5.5 applies; and (iii) a net-capital rule violation is a **prudential-type** failure rather than a market-conduct failure, which would make it the closest thing to a capital-adequacy data point in this firm's public record — which is precisely why it must not be repeated without its primary source. **The correct file entry today is: reported, named source, dated, ⚠ unverified at the authority.**

### 5.7 The Civil Litigation

| Matter | What is reported | Status |
| --- | --- | --- |
| Class-action settlement | Tower "subsequently settled a class-action lawsuit for **US$15 million**" | ⚠ **financefeeds.com**, Andrew Sax-McLeod, **3 Feb 2021** (via Wikipedia) — secondary; the settlement document, the court and the class definition were not retrieved ❌ |
| Korean investors' spoofing suit | The Second Circuit reportedly **upheld Tower's win** on the basis that **trading on the Korean exchange was not subject to the U.S. Commodity Exchange Act** | ⚠ **Reuters**, Jody Godoy, "2nd Circuit upholds Tower Research win in Korean futures case", **22 Jun 2021** (via Wikipedia) — named press, dated; the opinion was not retrieved ⚠ |
| An ex-Tower trader's sentencing | "Ex-Tower Research Trader Ducks Jail Time In Spoofing Case" | ⚠ **Law360**, Rachel Scharf, **9 Jul 2021** — headline-and-title only; **the trader is not named here**, because the source reviewed does not name one in the material captured |

**The discipline on civil matters.** A complaint's allegations are not findings; a settlement is not a judgment; and a reported win for the defendant is the **absence** of liability, which is a fact worth recording in a counterparty file rather than ignoring. The Korean-futures item is the clearest instance in this record of a court **declining** to extend a US commodities statute to a foreign venue's trading — a jurisdictional holding, not a finding about conduct ⚠.

### 5.8 Registrations and the Asia Regulatory Position

**United States.** No registration of any Tower entity was verified at a US register this pass ❌/⚠. FINRA BrokerCheck, the SEC's investment-adviser register, the NFA's BASIC register and the CFTC's registrant list were **not reachable or not queried** (sec.gov retrieval failed; web search non-functional). **This is a tool limitation recorded as such, not a finding of absence** — and it is an important distinction, because a proprietary trading firm that is not a broker-dealer and not a registrant is perfectly capable of trading US futures and equities through intermediaries, and this guide can neither confirm nor deny what Tower's US registration footprint is. What **is** established is that no US broker-dealer entity of the group appears anywhere on the firm's own pages ✅/❌ — the entity DOJ names is the LLC, and the firm's liquidity page names only two European entities and the unexplained "TRCX" ✅.

**Europe.** The firm states it operates **two Systematic Internalisers** ✅. SI status is a designation under the MiFID II regime, which means the underlying entities are authorised investment firms in the UK and (on the B.V. form) the Netherlands ⚠ (jurisdiction inferred from the legal form for the B.V.; not stated on the page). Whether each entity's SI status is **current**, whether each is authorised and by which regulator, and what their permissions are, were **not verified at the FCA register, the AFM register or ESMA's SI database this pass** ⚠. A bank that intends to transact with either entity should verify each at its home register — that is a fifteen-minute check and it is the single highest-value registration verification available for this firm.

**Singapore and Asia.** No Tower entity's registration or licensing in Singapore, Hong Kong, India, Dubai or China was identified this pass ❌. What is established is the **presence** of offices (§9.2) — and presence is not permission. Specifically: no MAS capital-markets-services licence, no SGX membership and no Hong Kong SFC licence was found for any Tower entity ⚠/❌, and again the absence reflects an unperformed register search rather than a confirmed negative. **Local market participation does not require local licensing where a firm trades through a local or remote member** — so the absence of a licence would not be surprising, and the presence of one would be informative. Either way, the file needs the answer, and the answer must come from a register.

### 5.9 The Dated Record

| Date | Authority | Instrument | Parties | Outcome as stated |
| --- | --- | --- | --- | --- |
| **≈Mar 2012 – Dec 2013** | (conduct window per DOJ) | — | Three traders, "a single trading team at Tower" | Described in the 2019 DPA documents ✅ |
| **≈Mar 2012 – Mar 2014** | (conduct window per DOJ 2018 release) | — | The three individuals named in §5.5 | Described as alleged conduct ✅ |
| **Early 2014** | DOJ (recorded as a mitigating factor) | — | The firm | "swiftly moved in early 2014 to terminate the three traders" ✅ |
| **17 Sep 2014** | SEC (per secondary source) ⚠ | SEC penalty order (per WSJ) ⚠ | **Latour Trading** (subsidiary, per Wikipedia) ⚠ | Reported US$16m penalty for net-capital-rule violations ⚠ — **not verified at the authority** |
| **12 Oct 2018** | DOJ, Office of Public Affairs | Release No. **18-1328** | **Three individuals**; firm referred to as "Trading Firm A" | Indictment and two criminal informations; two agree to plead guilty; charges "merely allegations" ✅ |
| **Oct 2018** | DOJ / SDTX | Indictment | **Yuchun "Bruce" Mao** | Charges pending as of Nov 2019 ✅ |
| **2 Nov 2018** | DOJ / SDTX | Guilty plea | **Kamaldeep Gandhi** | Pleaded guilty to two counts; sentencing scheduled 7 Feb 2020 (Judge Werlein) ✅ |
| **6 Nov 2018** | DOJ / SDTX | Guilty plea | **Krishna Mohan** | Pleaded guilty to one count; sentencing scheduled 13 Feb 2020 (Judge Miller) ✅ |
| **6 Nov 2018** | DOJ / SDTX | **Criminal information** (filed) | **Tower Research Capital LLC** — one count of commodities fraud | Filed the day before the announcement ✅ |
| **7 Nov 2019** | DOJ, Office of Public Affairs | Release No. **19-1,208**; **deferred prosecution agreement**; joint motion to defer prosecution and trial, subject to court approval | **Tower Research Capital LLC** | Combined **US$67.4m**; compliance reviews and programme modification; no statement about admissions in the release ⚠ |
| **7 Nov 2019** | CFTC (as described by DOJ) ⚠ | **CFTC order** | **Tower** | ≈**US$67.4m** including a **US$24.4m** civil monetary penalty, plus restitution and disgorgement cross-credited to DOJ; remedial and cooperation obligations ✅ (description) / ⚠ (not read at cftc.gov) |
| **3 Feb 2021** | (civil, reported) | Reported class-action settlement | Tower | Reported **US$15m** ⚠ |
| **22 Jun 2021** | US Court of Appeals for the Second Circuit (reported) | Reported appellate decision | Tower, Korean investors | Reported win for Tower on the Commodity Exchange Act's territorial reach ⚠ |
| **9 Jul 2021** | (criminal, reported) | Reported sentencing | An ex-Tower trader (unnamed in the material captured) | Reported no jail time ⚠ |

### 5.10 What Remains Unresolved or Unverified

- **Whether the DPA term has run, and whether the criminal information was ever dismissed.** DOJ's release states a joint motion to defer prosecution "for the term of the DPA, subject to court approval" and does not state the term length ✅/❌. Whether the matter has since been dismissed, extended or breached was **not established this pass** ⚠. A bank that describes this as a live prosecution, or as fully concluded, is asserting something this record does not support.
- **Any admission by the firm.** Addressed in §5.2: DOJ's release does not address admissions, and this guide does not supply the missing language ⚠.
- **The Latour Trading matter at the authority.** Not verified; sec.gov unreachable (§5.6) ⚠.
- **The SEC's own instrument in any matter involving any Tower entity.** Not identified ❌ — with the sole exception of the secondary-sourced 2014 claim.
- **The CFTC order's own text.** Not read; DOJ's description is the source (§5.3) ⚠.
- **Sentencing outcomes for Gandhi and Mohan**, and the **disposition of the Mao indictment**. Not established ❌ — only the scheduled dates are in the record.
- **Whether any Tower entity holds a registration, licence or membership in the US, the UK, the EU, Singapore, Hong Kong, India, Dubai or China.** Not established ❌. **This is the single largest verification gap in this guide**, and it is a gap created by unreachable registers rather than by the firm's silence — a distinction that matters because the answer is likely to be obtainable in an afternoon with working access.
- **Whether the 2021 Law360 sentencing item concerns a person also named in this record.** Not established; deliberate non-assertion ⚠.

---

## 6. The Technology and the Latency Posture

### 6.1 The Class of Infrastructure

Firms of this class run a recognisable stack: exchange-facing gateways and market-data handlers; deterministic low-latency software with minimal allocation and hot-path discipline; co-located compute inside or beside venue data centres; kernel-bypass networking and, at the fastest firms, FPGA or custom silicon in the path; precision time and synchronisation; and a research-and-simulation estate large enough to test strategies against historical tick data. **The class itself is not described here**, because it is owned elsewhere: the practice of low-latency software engineering belongs to **[Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md)**, **[Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md)** and **[Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md)**; the platform, market-access and order-path architecture belong to **[Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md)**; and the technology posture of the firm type belongs to **[Citadel LLC](citadel_llc_guide.md) §7** and to **[Hudson River Trading](hudson_river_trading_guide.md) §5**. Cross-referenced, not re-derived.

The purpose of this section is narrower: to state **what this firm itself says about its own stack**, verbatim and attributed, and then to mark exactly where the firm stops.

### 6.2 What This Firm Itself Documents

Everything in this subsection is the firm's own first-party statement ✅ (tower-research.com/engineering and /careers, retrieved 27 Sep 2026; the engineering page was re-fetched this pass and is **unchanged** from the dispatcher's read — no drift to report).

| The firm's own claim | Category | Reading |
| --- | --- | --- |
| "Proprietary technology for **every aspect of trading**: market access, data management, quantitative research, compute infrastructure, compliance, support, and beyond" | Scope | Total coverage claimed for an in-house stack — including compliance, which is unusual to list as proprietary technology ✅ |
| "**Dozens of colocation centers** around the world, where we are constantly tweaking the hardware" | Footprint | A co-location estate across dozens of sites, described qualitatively; **no venue, data centre, city or count named** ⚠ |
| "**Hundreds of engineers** supporting **12+ global offices** and **150+ trading venues**" | Scale | The firm's only headcount-shaped number (engineers only) and its only venue count — and the source of the office-count discrepancy (§9.2) ✅/⚠ |
| "Managing **100 petabytes** of data" | Data estate | A single very large number, unattributed to any system or purpose ✅ |
| "Tower employs **C++ and Python** while strategically investing in **Rust** for defensive and systems programming" | Languages | Three languages, with Rust characterised as an **investment** and scoped to "defensive and systems programming" — a precise, non-promotional statement ✅ |
| "the latest capabilities in **machine learning, FPGA technology, low-latency programming**, and beyond" | Technique | FPGA and ML named as capabilities; the careers page adds "**wireless telecommunication, hardware acceleration**" ✅ |
| "We build almost everything in-house" / "Our flat organization" | Culture-of-build | An in-house bias and a flat structure, both marketing claims with no operational detail ✅ |
| "While we can't share everything we're working on, here are some examples of how we build differently" | The boundary, in the firm's own voice | The firm states its own disclosure limit explicitly ✅ |
| "Our team functions more like a **software company** than a traditional financial organization, with Tower's quantitative trading teams as our **clients**" | Self-description | The internal-client model of §4.3 ✅ |
| "**Storage**… We embrace the latest technology in storage. Whether to house books and records or to work with alternative market data…" | Storage | "Latest technology" with **no vendor or product named** ⚠ |

Note what this list is and is not. It is **more than most peers publish** in three respects: named languages including a strategic one, a venue count, and a data-volume figure ✅. It is **not** an architecture. A reader who wants to know whether Tower runs a particular FPGA vendor's card, a particular kernel-bypass stack, a particular microwave route, or a particular matching-engine co-location, will not learn it here — and, critically, **will not learn it by reading about a peer instead** (§14.1, anti-pattern 1).

### 6.3 The One Documented Vendor Relationship — Torstone, Post-Trade

This is the **only** third-party technology relationship this guide can attribute to Tower Research Capital with a named source and a date:

> **"Tower Research Selects Torstone for Global Cross-Asset Post-Trade Services"** — Electronic market-maker Tower Research Capital is deploying **Torstone Technology's Torstone Platform globally for post-trade processing across all asset classes**, using the SaaS-delivered platform to manage trade capture, accounting, reconciliations and corporate actions for all of Tower Research Capital's **global entities**. Tower cited the platform's flexibility and scalability. ✅ (A-Team Insight, **19 May 2022**; retrieved 27 Sep 2026.)

Why it matters beyond the vendor name:

- **It is post-trade, not latency.** The one vendor the firm has publicly selected sits on the **back office** side — trade capture, accounting, reconciliations, corporate actions — and is delivered as **SaaS** ✅. That is a statement about where the firm buys versus builds: it builds the fast path (§6.2) and it buys the books-and-records path.
- **It is scoped to "all of Tower Research Capital's global entities"** ✅ — the closest thing to a group-structure statement in the entire public record, and useful corroboration that the group is multi-entity (§2.1).
- **It contains a firm-attributed growth statement**, quoted in §2.4 (COO Alan McGroarty: "unprecedented levels of growth… diversify into new asset classes and trading strategies, at ever increasing volumes") ✅.
- **It is dated 2022**, and no verification of the relationship's current status was possible this pass ⚠. The article is a firm-selected vendor announcement — **the firm chose to be named** — which is exactly why it is citable and why it is not a complete vendor inventory (§14.1, anti-pattern 1).

**What this guide will not do**: infer the rest of the vendor stack. The absence of further named vendors is the firm's disclosure choice, not evidence that it uses nothing else.

### 6.4 What Is Not Documented

Stated plainly, because each of these is a sentence that appears in secondary write-ups about firms of this class and appears **nowhere** in Tower's own material or in any authority read this pass:

| Undocumented item | Status |
| --- | --- |
| A system or platform architecture (feed handlers, gateways, risk gates, order-management path) | ❌ not published |
| Any hardware vendor (servers, NICs, switches, FPGAs, timing hardware) | ❌ not published |
| Any **microwave, millimetre-wave or shortwave** network, route or link | ❌ not published — and not asserted anywhere in this guide |
| Any **latency figure** (internal, round-trip, tick-to-trade, colocation distance) | ❌ not published |
| Any **venue or venue-group** by name | ❌ not published — the firm says "150+ trading venues" and names none |
| Any **data centre or colocation facility** by name | ❌ not published |
| Whether the firm kernel-bypasses, uses DPDK, uses a specific FPGA vendor, or builds its own NICs | ❌ not published |
| Whether the firm operates a wireless transport business for third parties | ❌ not published |
| Its cloud-versus-on-premise posture | ❌ not published — with the single exception of the **SaaS post-trade** platform (Torstone) ✅ |
| Any AI/ML framework, model class, or GPU estate detail | ❌ not published — "machine learning" is a capability claim only |

**The correct posture for a bank's file**: describe the firm as **a top-tier electronic trading firm with a documented in-house stack, a documented language set, a documented co-location and venue scale claim, and an undocumented architecture**, and stop. Any sentence of the form "Tower uses X" where X is not on the §6.2 and §6.3 lists is a fabrication with a familiar shape — the most common species of error in this genre (§14.1, anti-pattern 1).

---

## 7. Talent and Culture

### 7.1 The Class Model

The resourcing and culture model for a firm of this type — the researcher/engineer/trader split, the recruiting funnel, the compensation structure and the attrition pattern — is developed for the prop-firm archetype in **[Hudson River Trading](hudson_river_trading_guide.md) §6**, and the quant-research and engineering skill stack itself belongs to **[Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md)**. Neither is re-derived here. This section records only **what Tower itself publishes about its own people**, because that is what a bank can actually rely on and because the firm publishes more than most (a careers page, a culture page, a benefits list, a values set, a hiring-philosophy statement and a leaders list) ✅.

### 7.2 What This Firm Publishes — Roles and Structure

| Element | What the firm publishes | Source |
| --- | --- | --- |
| Role families | Roles are split across **Quantitative Trading**, **Core Engineering** and **Business Support** | ✅ tower-research.com/careers, retrieved 27 Sep 2026 |
| Engineering self-image | "Our team functions more like a software company than a traditional financial organization, with Tower's quantitative trading teams as our clients"; "Our flat organization"; "you're not just a cog in a machine" | ✅ /engineering |
| Qualities prized | Drive, Communication ("with both fellow engineers and the traders you're supporting"), Collaboration, Passion | ✅ /engineering |
| Engineering priorities | "Take Ownership", "Innovate Constantly", "Drive Success" — with technology described as "one of our biggest advantages over the competition" | ✅ /engineering |
| Trading-organisation structure | Independent internal trading teams with autonomy plus shared technology and control resources | ✅/⚠ Wikipedia (secondary) |
| Leaders | **Mark Gorton** (Chairman), **Albert An** (CEO) | ✅ /about-us |
| Geographic spread of hiring | Fourteen city entries on the offices page; "Explore All Careers" and "Build Your Career at Tower" calls to action | ✅ /offices, /careers |

The **engineering-as-internal-service-provider** framing is the firm's most substantive culture claim, and it is architecturally consistent with §4.3: a platform with autonomous trading teams needs an engineering organisation that behaves like a product company with internal customers. Whether that materially affects outcomes is not something a bank can verify — but it is a coherent, first-party statement of design intent, which is more than the record contains for most of the peer set ✅.

### 7.3 What This Firm Publishes — Benefits, Values and Hiring Philosophy

**Values** (firm's own list): **Excellence, Respect, Innovation, Integrity, Teamwork** ✅ (tower-research.com/about-us). Three of the five are process values rather than outcome values — a common shape for firms whose output is unobservable from outside, and one that should not be over-read either way ⚠.

**Benefits the firm publishes** ✅ (tower-research.com/careers, retrieved 27 Sep 2026): discretionary and **team-based** bonuses; **five weeks paid vacation**; a **401(k) match** for US employees; in-office meals. Two observations are legitimately available to a bank. First, the benefits are **entity-scoped** — the 401(k) match is described for US employees, which is a reminder that the group is not one employer and that an Asia-based hiring analysis cannot be done from the US page ⚠. Second, **team-based bonuses** are a compensation design statement: at a firm organised as autonomous internal teams (§4.3), a team-based bonus pool aligns pay with team P&L — which is the compensation analogue of the control question raised by the 2019 matter (§5.2), and is precisely the kind of incentive-design question a bank's enhanced due diligence should ask and cannot answer from a careers page ⚠.

**Hiring philosophy**: the firm states it does **not** ask "gotcha questions" in its process ✅ (tower-research.com/careers). Recorded as a first-party statement about process, with no verification possible.

**Culture page**: the firm operates a dedicated culture page and describes in-office events and outings and "support for causes that matter" ✅ (tower-research.com/engineering → /culture link; the culture page itself was not opened this pass ⚠).

### 7.4 What Is Not Published About the People Model

- **Headcount by entity, office or function** ❌ — the only headcount-shaped figures are "hundreds of engineers" (corporate) ✅ and the conflicting group totals (§10.3) ⚠.
- **Compensation levels or structure beyond the benefit list** ❌ — no salary bands, no P&L-linked formula, no deferred-compensation design.
- **Attrition, tenure or retention data** ❌.
- **The researchers-versus-engineers split, and whether a research track exists separately from trading** ⚠ — the careers taxonomy names three families; the internal composition of each is not published.
- **Any graduate programme, internship structure or academic partnership** ❌ — several peers publish these; Tower's pages captured do not ⚠.
- **Singapore or Asia-specific hiring scope** ⚠ — the Asia angle is treated in §9.

---

## 8. The Regulatory and Market-Structure Context

### 8.1 Where a Proprietary Trading Firm Sits

The position of the proprietary trading firm in the market-structure and market-conduct debate — the post-2010 policy wave, the speed-and-liquidity arguments on both sides, the designation questions and the non-bank market maker's behaviour in stress — is developed for the firm type in **[Hudson River Trading](hudson_river_trading_guide.md) §7 and §8**, and it is **not re-derived here**. What is specific to Tower, and what a bank's file should hold:

- **The firm is a liquidity provider in the equities, ETF and FX/metal markets it names, and describes itself that way** ✅ (tower-research.com/liquidity-provision). It is, therefore, a participant whose withdrawal in stress is a market-structure question, not merely a commercial one — the same structural exposure a bank carries with any non-bank market maker (worked through in §13.4 and in **[Jump Trading](jump_trading_guide.md) §14.4**).
- **It is subject to the conduct rules of the venues it trades on** — in the US, the exchange and SRO rulebooks and the Commodity Exchange Act; the 2019 matter was brought under the commodities fraud statute ✅ (DOJ Release No. 19-1,208). It is **not**, on the evidence captured, subject to any prudential supervision of its capital or liquidity ⚠, which is precisely why the Latour net-capital claim (§5.6) would be significant if verified.
- **Its two European systematic internalisers are inside the MiFID II conduct perimeter** ✅ for the entities named, which means best-execution and pre-trade transparency obligations attach to that slice of the business — a fact that matters to a bank dealing with those entities and not to a bank dealing with the LLC.

### 8.2 The Registration and Designation Questions

Three questions a bank's due-diligence checklist will ask, answered only to the extent the record supports:

| Question | The answer this guide can support |
| --- | --- |
| Is the firm a registered broker-dealer anywhere? | **Not established** ❌ — no register was reachable this pass (§5.8). The firm's own pages name no US broker-dealer entity ✅. There is therefore no registered broker-dealer to point at, and no basis for treating the group as one |
| Is the firm a designated market maker, a member of any exchange, or a participant in any incentive programme? | **Not established** ❌. The firm says "150+ trading venues" and names none ✅/⚠ |
| Is any group entity a prudential entity with a published capital position? | **Not established** ❌ — with the sole, unverified, secondary exception of the Latour net-capital claim for 2014 ⚠ |

The honest framing for a file: **this is a firm whose regulatory footprint is largely invisible from outside**, and the invisibility is a structural feature of the firm type rather than an anomaly. A proprietary trading firm trading its own capital can trade through members and clear through clearing members; it is not obliged to be a registrant itself. But the consequence is that the bank cannot substitute a register lookup for an assessment, which is the entire problem §12 and §13 exist to solve.

### 8.3 Spoofing as a Market-Conduct Offence

Spoofing — placing orders with intent to cancel before execution — became the defining market-conduct offence of the electronic era, and the Tower matter sits directly inside that story: the conduct DOJ describes (thousands of instances of orders placed with intent to cancel, injecting false supply-and-demand signals into the visible book, over roughly March 2012 to December 2013) ✅ is the archetype of the offence. Three things are worth setting down for a bank's file, and none of them is a legal opinion:

1. **The offence is about the integrity of displayed liquidity** — which is why it is more damaging to a market maker's franchise than to an ordinary firm's: the market maker's product is the credibility of its quotes (§1.4).
2. **Detection is surveillance-dependent** — which is why the remediation DOJ records for Tower is specifically about surveillance and governance: "significant investments in sophisticated trade surveillance tools, increased legal and compliance resources, revised the company's corporate governance structures and changed its senior management" ✅ (DOJ Release No. 19-1,208). A bank asking a counterparty how it detects spoofing in its own order flow is asking about the same control the authority records having been strengthened.
3. **The market-conduct perimeter has moved on since 2012–2013**, and a bank must date any conduct assessment accordingly: a matter whose conduct window closed in 2013 is evidence about controls more than a decade old, and the question a credit or compliance committee should ask is not "was there a problem" but "what has changed since, and how would the bank know" (§13.4).

### 8.4 The Record, Not the Statements

The point the repository has now made in three firm guides, restated once because it governs this whole section: **a firm's conduct record is a matter of record, not of its public statements.** Tower publishes values — "Excellence, Respect, Innovation, Integrity, Teamwork" ✅ — an engineering organisation and a culture of ownership; none of that is evidence about conduct, and none of it substitutes for an authority's document. Equally, the existence of a 2019 enforcement matter does not make the firm's current statements false. The correct method is the asymmetry this guide has used throughout: **credit first-party claims for what they claim to be (product, footprint, scale, structure), and look to authorities and dated press for what a firm's own words cannot establish (conduct, capital, outcomes)** ✅/⚠. Anything else is reading tea leaves in the marketing.

---

## 9. The Asia and Singapore Angle

### 9.1 What the Sibling Guide Already Owns

**The Singapore facts about this firm belong to [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.6**, which is cross-referenced by name here and **not re-derived**. That subsection records, in terms this guide reproduces verbatim so that the two can be reconciled rather than allowed to drift:

> "**Singapore presence:** Singapore is on Tower's official office list (tower-research.com/offices) ✅. ⚠ The **office's establishment year is not publicly disclosed** — the task brief's working assumption of '~2007–2010' could not be verified and is treated as unconfirmed (§11.4)." ⚠

That guide's §6 comparison table carries the corresponding row: **Tower Research | 1998 | New York | HFT/quant across global markets; crypto (Limestone) | Office listed (tower-research.com); year ⚠ not public | 150+ venues; US$67.4M spoofing fine (2019)** ✅/⚠. Its §4.6 also carries the "~1,400+ employees (2026); 11 offices" figures and the 2014 Latour item — both of which this guide treats as secondary and flags (§10.3, §5.6). **Where the two guides differ, this guide's evidential position is stated in §9.5, and the sibling guide's §4.6 remains the owner of the Singapore framing.**

### 9.2 What This Guide Independently Verifies

Verified this pass against the firm's own current offices page ✅ (tower-research.com/offices, retrieved 27 Sep 2026):

| Asia-Pacific item | What the firm publishes |
| --- | --- |
| **Singapore** | **Marina One West Tower, 9 Straits View, Singapore 018937** — listed as a fully addressed office entry, not a presence mention ✅ |
| **Gurugram** | Two Horizon Center, DLF Phase 5, Golf Course Road, Sector 43, Gurugram, Haryana 122002, India ✅ |
| **GIFT City** | GIFT One Tower, Gujarat International Finance Tec-City, Gandhinagar, Gujarat 382355, India ✅ |
| **Hong Kong** | Three Exchange Square, 8 Connaught Pl, Central, Hong Kong ✅ |
| **Shanghai** | 2604-2606, HKRI Taikoo Hui, 288 Shimen Road (No. 1), Shanghai, PRC, 200041, China ✅ |
| **Dubai** (adjacent region) | The Gate Building, Dubai International Financial Centre, United Arab Emirates ✅ |
| **Also in the office set** | New York (120 Broadway 38th Floor, NY 10271 **and** 377 Broadway, NY 10013), London (The Minster Building, 21 Mincing Lane, EC3R 7AG), Amsterdam (WTC, Strawinskylaan 859, Tower Ten, 1077 XX), Charleston (111 Coleman Boulevard, Mt. Pleasant, SC 29464), Chicago (330 N Wabash, IL 60611), George Town (90 N Church Street, 2nd Floor, Grand Cayman, KY1-1209), Montreal (2001 Blvd Robert-Bourassa, QC H3A 2A6, with a separate Canada site), Paris (32 Rue des Mathurins, 75008) ✅ |

**The office count, reconciled as far as the evidence allows.** The live offices page lists **fourteen city entries** ✅ (counted from the retrieved page: New York, London, Gurugram, Singapore, Amsterdam, Charleston, Chicago, Dubai, George Town, GIFT City, Hong Kong, Montreal, Paris, Shanghai). Wikipedia's infobox says **11 locations** ⚠. The firm's own engineering page says "**12+ global offices**" ✅/⚠. **This guide reports the live page as the best evidence and flags the discrepancy rather than inventing a reconciliation** — plausible explanations include the New York entries being counted as one city (which would yield 13), deliberate understatement in older corporate copy, and formulaic Wikipedia maintenance; none of them is evidence, so none is asserted. **The two-Broadway oddity** is flagged for the same reason: Wikipedia reports that the **2023 lease of 121,903 sq ft at 120 Broadway consolidated two New York City offices into one** ✅/⚠ (Commercial Observer, 6 Sep 2023; Bisnow, 6 Sep 2023), yet the live offices page still lists **377 Broadway** ✅. Both facts are recorded; the apparent inconsistency is credited to neither side, because "the page is stale" and "the consolidation did not eliminate the second address" are equally available readings ⚠.

**What is not verified.** The **Singapore entity name**, incorporation date and registration number ❌; the **establishment year of the Singapore office** ❌ (confirming, not merely repeating, the sibling guide's ⚠); the **Singapore headcount** ❌; the **Singapore activity mix** (trading, research, engineering, support) ❌. The last of these is the one a bank would most like to have, and it is not published anywhere this guide can find.

### 9.3 The Wider Asian Footprint

- **The firm's Asian footprint is broad and looks like a follow-the-market build**: **Singapore** (a financial centre with an FX and derivatives franchise), **Hong Kong** (a market-access and China-facing hub), **Shanghai** (a separate jurisdiction with its own access regime), **Gurugram and GIFT City** (India — one conventional office city and one international financial services centre), and **Dubai/GIFT City** as the two newest-regime venues of the set ✅ (tower-research.com/offices). Reading the list as evidence of what the firm *does* in each city is not supported: the page gives addresses, not functions ❌.
- **GIFT City is a specific signal worth one sentence.** GIFT City is Gujarat's international financial services centre with its own regulator (IFSCA) and its own regime for foreign firms; a Tower entry there is consistent with a market-access strategy aimed at Indian and cross-border derivatives ✅/⚠ — but the firm publishes no licence, no entity and no Indian product line, so the guide stops at the observation.
- **Montreal sits oddly in the Asia section but matters to the group map**: a separate Canada site at tower-research.ca ✅ suggests a distinct Canadian operating entity surface, none of which is named ⚠.
- **No Asian regulatory relationship of any kind is established** for any entity ❌ (§5.8, §9.4). The offices page is a real-estate document.

### 9.4 The Regulatory Position in Asia

Recorded plainly, because it is a question with a real answer that this pass did not obtain: **no Tower entity's Singapore licensing, SGX membership, Hong Kong SFC licence, IFSCA registration or Chinese regulatory status was identified** ❌. The implications:

- A bank dealing with a Tower entity **in** Singapore cannot assume the contracting entity is MAS-regulated, and cannot assume it is not. The register check (§5.8) settles it in minutes; the marketing page never will.
- Where the Singapore office trades Singapore-listed securities or SGX derivatives, it may do so as a **remote member's client, through a local member, or as a client of a clearing member** — all of which put the bank, not the firm, closer to the regulated perimeter ⚠. This is a standard structure for foreign proprietary firms and is exactly why the *entity* question in §12.1 dominates the analysis.
- Cross-reference for the regime detail: the Singapore regulatory and market-making context — SGX's structure, the CDP and SGX-DC clearing map, co-location and the MAS capital-markets overlay — is owned by **[Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §7 and §8** and by **[MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)**. Neither is re-derived here.

### 9.5 The Reconciliation

**What this guide's evidence ADDS to `market_making_singapore_guide.md` §4.6:** (1) **the full Singapore address** — Marina One West Tower, 9 Straits View, Singapore 018937 ✅ — where §4.6 cites only the office's existence, because an address is what turns a presence claim into an entity-identification task; (2) **the complete office set behind the Singapore entry** — fourteen city entries with addresses ✅ — and the explicit finding that **the three counts in circulation do not agree** (14 live / 11 Wikipedia / "12+" firm engineering page) ⚠, where §4.6 states "11 offices" as a fact without the discrepancy; (3) **the Asian footprint in full** (Gurugram, GIFT City, Hong Kong, Shanghai, Dubai) ✅, where §4.6 treats Singapore alone; (4) **the entity disclosure that bears on Asia** — the firm's own liquidity page names only **two European** systematic internalisers plus the unexplained **TRCX** and **no Asian entity at all** ✅/⚠, a sourced version of §4.6's "year not public"; (5) **the regulatory-record detail** behind §4.6's summary line — the **DOJ release numbers 18-1328 and 19-1,208**, the **DPA instrument**, the **US$67.4m / US$24.4m composition**, the **cross-credit** between the DOJ and CFTC payments, and the **individual-level table** ✅ (§5.2–§5.5); (6) **the fact that the 2018 release does not name the firm** (§5.4), correcting any reading of §4.6's "traders criminally charged" as one instrument covering firm and individuals; (7) **the LimeWire/founder material** (§3.4), absent from the sibling guide and the repository; and (8) **the calibration of the opacity finding** (§2.3) — the firm publishes materially more than "offices and a fine" ✅.

**What it CONFIRMS in §4.6:** the **February 1998** founding and the **Gorton / Brown** founders ✅/⚠; **New York** HQ ✅; quantitative/HFT trading across global markets ✅; the **Singapore office** ✅; the **undisclosed Singapore establishment year** ⚠ (independently confirmed as undisclosed, not merely unfound); the **150+ venues** figure ✅ (the firm's engineering page is the source §4.6 cites second-hand); and the **US$67.4m 2019 spoofing resolution** ✅, now sourced to DOJ rather than Wikipedia.

**What it does NOT settle:** the **Singapore entity name**, incorporation date and registration number ❌; the **office's establishment year** ❌ — §4.6's ⚠ survives this pass intact and this guide does **not** adopt the unconfirmed "~2007–2010" assumption either; the **headcount split by office** ❌; **which Asian entities contract** ❌; whether the **crypto team trades in Asia** ⚠; and the **office-count question** ⚠. Where the two guides differ, the difference is the **level of sourcing** (this guide names addresses and release numbers; §4.6 names the office and the fine), not a factual conflict — and §9.2's live-page evidence governs.

---

## 10. The Capital and the Revenue

### 10.1 The Discipline

**The firm discloses no financials.** No revenue, no profit, no capital, no AUM, no valuation, no leverage, no balance sheet, no segment economics, no ownership percentage and no dividend or distribution history is published by Tower Research Capital in any source read this pass ❌. This is not an inference from silence: it is the result of reading the firm's own pages (§2.3) and finding no financial statement of any kind, and of reading the enforcement releases (§5.2, §5.3) and finding monetary **penalties and disgorgement** — which are amounts the firm **paid**, not amounts it **earned** ✓/❌.

The rules this section holds to, identical to those adopted from **[Jump Trading](jump_trading_guide.md) §7** and **[ExodusPoint](exoduspoint_guide.md)**:

1. **No figure without an outlet and a date**, labelled as reported.
2. **No figure presented as current** — every number carries the date it belongs to.
3. **No peer's figure imported**, and no industry average substituted for a firm-specific number.
4. **Where nothing credible exists, the absence is the finding** (§10.5).

### 10.2 What Is Reported, with Outlet and Date

| Item | What is reported | Outlet and date | Quality |
| --- | --- | --- | --- |
| Penalty and disgorgement (criminal) | **US$67.4m** combined criminal monetary penalty, criminal disgorgement and victim compensation | DOJ, Release No. 19-1,208, **7 Nov 2019** | ✅ primary — but **this is a cost, not revenue** |
| Civil monetary penalty | **US$24.4m**, part of a ≈US$67.4m CFTC settlement with restitution and disgorgement cross-credited to DOJ | DOJ's description of the CFTC order, **7 Nov 2019** | ✅ as description / ⚠ not read at cftc.gov |
| Civil class-action settlement | **US$15m** | financefeeds.com (Andrew Sax-McLeod), **3 Feb 2021** | ⚠ secondary, single-sourced |
| Alleged investor losses in the commodities conduct | "over **US$60m**" | DOJ, Release No. 18-1328, **12 Oct 2018** | ✅ primary — **a loss figure for other market participants, not a Tower financial** |
| Secured/unsecured borrowings | **None identified** — no bond, no term loan, no rating action, no financing announcement found this pass | — | ❌ nothing to report |
| Rating-agency research | **None identified** for any Tower entity | — | ❌ nothing to report |
| Revenue, profit, capital, AUM, valuation | **None published** | — | ❌ **the finding** |

**Two traps this table closes.** First, **penalty ≠ revenue**: a US$67.4m payment tells a bank nothing about earnings capacity, and the temptation to size a firm by its enforcement headline (or by its fine relative to a peer's) is an anti-pattern (§14.1, anti-pattern 6). Second, **cross-credit**: because the criminal monetary penalty was credited for payments made to the CFTC, and the CFTC's restitution and disgorgement were credited for payments made to DOJ ✅, the firm did not pay two separate US$67.4m amounts, and any file that adds them is wrong.

### 10.3 The Headcount Figure and Its Discrepancy

| Figure | Source | Date | Status |
| --- | --- | --- | --- |
| "more than **1,100 people worldwide**, including quantitative researchers, software engineers, and traders" | Tower's About Us page, via Wikipedia | **2025** | ⚠ (secondary citation of a first-party page; the page itself was not re-read this pass) |
| "c. **1,400+** employees" | Wikipedia infobox | **2026** | ⚠ secondary |
| "**Hundreds of engineers**" | The firm's engineering page | retrieved **27 Sep 2026** | ✅ first-party — but engineers only, and no total |

**Flagged, not resolved.** The three figures are not necessarily inconsistent (1,100 in 2025 and 1,400 in 2026 is a plausible growth path; "hundreds of engineers" is a floor, not a total), but **the infobox figure has no visible source in the material captured**, and this guide does not build on it. For a bank, the practical consequence is the same as everywhere else in this section: **headcount is not an input to the credit decision**, but it is a legitimate input to an operational-resilience assessment (people count bears on substitutability), and where the number is uncertain the bank should ask the client rather than pick a figure.

### 10.4 What the Firm's Own Structure Implies, and What It Does Not

Two structural statements, and one refusal:

- **A firm with "dozens of colocation centers", "hundreds of engineers" and "100 petabytes" of data is a firm with a substantial fixed-cost base** ✅ (tower-research.com/engineering). That is a legitimate inference from the firm's own claims: the class of infrastructure described is expensive to build and to run, and the cost is incurred whether or not the strategy is in profit.
- **A firm that "warehouses many different sizes of risk" and trades equities, ETFs, FX and metals across "150+ trading venues" runs a broad, capital-consuming book** ✅. Broad inventory implies financing, margin and clearing relationships — which is the substance of §12.
- **The refusal:** neither inference yields a **capital adequacy** view. Infrastructure cost is not capital; breadth of venues is not a balance sheet. **No leverage, no capital ratio, no funding profile and no loss history for any Tower entity is established** ❌, and no attempt is made here to derive one.

### 10.5 The Absence Is the Finding

This is the section's conclusion, and it is a **finding about the firm's disclosure posture**, not a suspicion about the firm.

**What the absence means for a bank.** The industry-standard credit inputs — financial statements, audited accounts, ratios, rating research, a prudential supervisor's assessment — **do not exist for this counterparty** and cannot be obtained from any public source. The bank has four options, and only three of them are legitimate:

1. **Refuse to transact absent financial disclosure.** A defensible position, and the one a bank should be able to justify if its risk appetite requires a supervised balance sheet.
2. **Transact only against collateral, at short tenor, and at sizes the collateral supports.** This converts the unobservable into the bounded, and it is the approach §13.5 adopts.
3. **Convert a public-disclosure posture into a private one** — that is, make the delivery of financial information a **condition** of the facility rather than a right the bank assumes (§13.5).
4. ❌ **The illegitimate option: substitute something else for the missing financials** — a peer's revenue, an industry average, a valuation implied by the firm's real-estate footprint or headcount, or the size of a 2019 penalty. Every one of those substitutions produces a number that looks like analysis and is not (§14.1, anti-patterns 4 and 6).

**And the corollary that a credit committee should hear once, plainly:** the absence is **lawful and normal** (§2.3), it is **not evidence of distress**, and it is **not a reason to decline a market-making relationship** — because the bank's exposure to a proprietary market maker is generally short-horizon and collateralisable, and the firm's conduct and operational record is publicly legible in a way its balance sheet is not. The absence **is** a reason to cap the grade, bound the limit and price the unobservability (§13.3, §13.5). **Where the balance sheet is dark, the limit is the light** — the same conclusion the repository reaches from a different evidential base for a peer firm at **[Hudson River Trading](hudson_river_trading_guide.md) §10.6**.

---

## 11. The Peer Group and Positioning

### 11.1 The Peer Set with Dated Sources

The peer landscape for electronic market makers — including the Singapore footprint column that **[Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4 and §6** owns — is carried there in full, with the Tower row at its §6 table. Reproduced here only as it bears on **this firm's** positioning, with sources named ✅/⚠:

| Peer | Founded / HQ | Distinguishing feature | Source |
| --- | --- | --- | --- |
| **Tower Research Capital** | 1998 / New York | One of the oldest automated trading firms; 150+ venues; equities, ETFs and European SIs named; crypto via an internal team; a ventures arm | ✅ firm pages; ✅/⚠ Wikipedia |
| **Hudson River Trading** | 2002 / New York | Founded by Tower Research alumni; multi-asset algorithmic market making | ✅ [Hudson River Trading](hudson_river_trading_guide.md) §2 (the alumni founding is documented there) |
| **Jump Trading** | 1999 / Chicago | Futures-era origin; digital-asset and venture arms; a settled SEC matter against a digital-asset affiliate | ✅ [Jump Trading](jump_trading_guide.md) §1, §5, §6 |
| **Citadel LLC** | 1990 / Miami (hedge fund + Citadel Securities the market maker) | The hybrid archetype: hedge fund and electronic market maker under separate businesses | ✅ [Citadel LLC](citadel_llc_guide.md) |
| **Jane Street** | 1999 / New York | ETF and options-liquidity giant; OCaml | ✅ [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §4.3 |
| **XTX Markets** | 2015 / London | FX electronic liquidity provision; the newest of the top-tier names | ✅/⚠ market making Singapore guide §4.8 |
| **DRW** | 1992 / Chicago | Futures origin; crypto through Cumberland | ✅/⚠ market making Singapore guide §4.9 |
| **Optiver / Flow Traders / IMC / SIG** | 1986 / 2004 / 1989 / 1987 | The European exchange-traded-market-making cohort and the US options specialist | ✅/⚠ market making Singapore guide §4 and §6 |
| **AlphaGrep** | 2010 / Singapore-headquartered group | The verifiable home-grown Asian market maker | ✅/⚠ market making Singapore guide §5.1 |

**The one fact this guide adds to the peer picture at firm level** — and it explains the title of §3.1: Tower is the **1998** name in a set dominated by 1999-and-later foundings, and **Hudson River Trading was founded by Tower Research alumni** ✅ (documented in the sibling firm guide) ✅. On the repository's own record, then, Tower is not merely old; it is **upstream of a peer** that the repository already treats as an archetype.

### 11.2 The Older Latency-Led Firms Against the Newer Entrants

What separates the 1986–1999 cohort (Optiver, SIG, DRW, Tower, Jane Street, Jump) from the 2002–2015 entrants (HRT, XTX and specialist spin-offs): **origin** — the older cohort came out of floor and market-making origins or the earliest electronic era, and Tower's February 1998 start ✅/⚠ with engineering-and-trading founders sits at the electronic end of that group; **disclosure posture** — the older cohort is, on this repository's evidence, quieter about technology than the 2010s entrants, with Tower in the middle: engineering claims with numbers, no architecture (§6.2, §6.4) ✅/⚠; and **regulated-status mix** — the older cohort tends to include registered broker-dealer entities because membership was the route to market ✅/⚠ (documented for the peer set at **[Jump Trading](jump_trading_guide.md) §2.3**), whereas Tower is the notable case here where **no registered broker-dealer entity could be identified** ❌ — a verification gap (§5.8), not a confirmed finding. **What is *not* different is the economics**: all trade their own capital, none reports to clients, none publishes financials — which is why the peer set is useful for technique and structure and useless for numbers.

### 11.3 This Firm's Relative Position, Only as Far as Sources Support

- **On age and continuity Tower is at the top of the cohort**: founded February 1998 ✅/⚠, called "one of the oldest automated trading firms" in the literature ✅/⚠, still trading in 2026 with a substantial New York footprint ✅ — a 28-year survival record in a high-mortality business.
- **On scale, no comparison is available.** The firm's own numbers are not comparable to peer figures in this repository, because the peer figures that exist are revenue and Tower has none ❌. **Any ranking of Tower by size is unsupported by this record**, and this guide does not imply one (§14.1, anti-pattern 6).
- **On conduct, the record is one company-level matter (a 2019 DPA) and one unverified secondary claim (Latour, 2014).** The repository documents for a peer a **US$123,095,287** settled SEC order against a digital-asset affiliate and a net-capital order against a trading entity ✅ (**[Jump Trading](jump_trading_guide.md) §6.2, §6.4**) — and comparing penalty sizes across different instruments, respondents and eras is a category error this guide does not commit.
- **On breadth, Tower is comparatively diversified within the firm type** — equities, ETFs, European equities via two SIs, OTC FX and precious metals, plus crypto and ventures ✅/⚠ — while **on the market-structure debate its position is unstated**: no policy paper, no research publication and no association membership found ❌. Its ventures investments in market-structure companies (Sk3W, TXSE) are the closest thing to a public view ✅ (§4.5).

---

## 12. The Bank Interface

### 12.1 The Firm as Counterparty — Clearing and Margin

At mechanism level, and cross-referenced rather than re-derived (clearing, margin and collateral mechanics for a firm of this type are worked through at **[Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) §7** and **[Hudson River Trading](hudson_river_trading_guide.md) §10.2 and §10.5**):

| Mechanism | How a Tower-shaped firm appears to a bank | What the bank controls |
| --- | --- | --- |
| **Exchange-traded positions** | Novated at a CCP; the firm reaches the CCP as a **clearing member's client** or as a member, and the bank — if it is the clearing member or the clearing member's agent — carries the client-level exposure | Client margin, limit monitoring, give-up and allocation flows, intraday margin calls in stress |
| **OTC positions (spot FX and precious metals, per the firm's own product list)** ✅ | Bilateral: confirmation, netting, collateral schedules, valuation, dispute mechanics. This is the segment where the bank is most likely to face the firm **directly** rather than through a CCP | Credit lines, collateral haircuts, threshold and MTA terms, close-out netting, daily and intraday margining |
| **Financing of inventory** | A market maker "warehousing many different sizes of risk" ✅ needs funding for the long side and **borrow/locates** for the short side; repo, margin lending, securities lending and structured financing are the instruments | Collateral eligibility, haircuts, concentration by asset class, tenor |
| **Settlement** | Multi-currency, multi-venue settlement flows, likely large gross and smaller net; corporate actions on equity inventory | Settlement-risk limits (intraday principal exposure), fails management, nostro funding |
| **The entity question over all of it** | The firm names **two European SI entities** and an unexplained **TRCX**; it does **not** name the entity behind most of its footprint ⚠ (§2.1, §2.5) | **Entity-level limits and documentation** — the single most important control, and the one most often done by brand (§14.1, anti-pattern 6) |

The structural point that a bank must not lose: **the firm trading its own capital is not managed as a client of a broker's franchise model.** It has no client assets to cushion a loss, no segregation regime, and no regulator publishing its capital. The bank's protection is collateral, tenor and documentation — not disclosure.

### 12.2 The Firm as a Liquidity Provider to a Bank's Clients

The firm's own liquidity-provision page is the **primary source** here, and this is what it says ✅ (tower-research.com/liquidity-provision, retrieved 27 Sep 2026): the offering is **"on-exchange market making"** and **"bilateral, disclosed liquidity in the world's most active markets"**, covering **US equities and ETFs** ("unique block trading capabilities", offered as **TRCX** and an **ETF Block Trading Platform**), **European equities** through the two named SIs, and **global FX in OTC spot and precious metals**, distributed through **"direct, disclosed relationships, semi-disclosed, and making into all primary and secondary markets"** — a **disclosure spectrum** a bank must map before it routes client flow: a *disclosed* relationship tells the client who the liquidity provider is; a *semi-disclosed* one does not, in whole or part. The firm also offers **"Customizable Liquidity Streams"** and stresses its ability to **"warehouse many different sizes of risk and multiple types of trade flow"** — it will take size, which is exactly what creates the bank's dependence — and it describes a relationship process ("day-to-day dialogue with counterparties", "reviewing pricing decisions, performing peer analysis") ✅.

**To be explicit about what this page is not.** It is not a client list and does not name a single bank, broker or fund ✅/❌. It is the firm telling the market that it sells liquidity. **Any statement that a named bank receives Tower liquidity, or that Tower is a client or counterparty of a named institution, is unsupported by this record** and prohibited by §12.6. How a bank should use it: the product list shows which venues and asset classes the firm can stream into; the disclosure spectrum fixes what must be agreed about transparency before routing; and the "customizable"/"warehousing" language signals that the firm expects to negotiate — so the bank's best-execution evidence, mark-out analysis and provider-concentration limits must be settled **before** the panel slot is granted.

### 12.3 The Firm as a Client of Clearing and Prime Services

A firm of this class buys a recognisable product set from a bank, and **Tower discloses none of it** — so the following is **mechanism, cross-referenced, not fact about Tower** ⚠: **clearing and give-up** in venues where it is not itself a member (and membership where it is); **FX prime brokerage with give-up**, if it runs the non-bank liquidity-provider model its own FX distribution implies ✅; **stock borrow, repo and financing** for a broad inventory-carrying book ✅ (the warehousing claim); **custody, asset servicing and treasury** across multi-currency positions; and **operational services** — connectivity, colocation provisioning and market-data licensing, often intermediated rather than owned. The honest note: **Tower publishes none of this**, and a bank cannot infer a specific provider relationship from the firm's size, from the fact that it obviously needs these services, or from the existence of one named post-trade vendor ✅/❌ — inference-by-necessity is exactly the anti-pattern §14.1 catalogues.

### 12.4 The Counterparty-Credit Questions a Bank Asks

For a counterparty that publishes no financials, the credit file is built from substitutes. The questions, in the order a credit officer should ask them:

| # | Question | What the public record can answer |
| --- | --- | --- |
| 1 | **Which legal entity is the obligor?** | Only the entity names in §2.1 ✅/⚠ — the rest must come from the client |
| 2 | **What is the entity's capital, and how is it funded?** | **Nothing** ❌ — no financials of any kind (§10.1) |
| 3 | **Is the entity supervised by anyone?** | **No prudential supervision identified** ❌ (the two SIs are conduct-regulated, not prudential) |
| 4 | **How does the firm make money, and how volatile is it?** | **Product list only** ✅ — no revenue, no volatility data ❌ |
| 5 | **How does the firm behave under stress?** | **Not published** ❌ — structurally, nothing obliges it to quote (§12.2; **[Jump Trading](jump_trading_guide.md) §14.4**) |
| 6 | **What is the conduct and control history?** | **The 2019 DPA and the 2014 secondary claim** ✅/⚠, including DOJ's own remediation language ✅ (§5) |
| 7 | **What operational risks does it carry?** | Engineering scale claims ✅; **no resilience disclosures** ❌ |
| 8 | **Who owns and controls the entity — and is there any guarantee or support agreement?** | **Not public and not inferable** ❌ — a Wikipedia sentence with "citation needed" is not a source, and a shared brand is not a guarantee |
| 9 | **What collateral can the entity post, and where is it held?** | **Not published** ❌ — but obtainable contractually, and it is the answer §13.5 builds on |

**The conclusion the table forces**: eight of the nine questions have no public answer, and the ninth is answered by the **legal documentation**, not by research. That is the case for the collateral-and-condition approach and against underwriting on narrative.

### 12.5 The Artefacts a Bank Can Actually Obtain

| Artefact | Source | Availability |
| --- | --- | --- |
| The firm's **product and liquidity-provision page** (what it offers, in which asset classes, on what disclosure terms) | tower-research.com/liquidity-provision | ✅ public, first-party |
| The **signed liquidity-provider agreement**, with disclosure and pricing schedules — where the mark-out and review rights the firm advertises belong | The counterparty | ✅ contractual |
| The **entity's register entry** (name, number, jurisdiction, authorisations), once the client names the contracting entity | The relevant regulator's register | ⚠ obtainable, not obtained this pass (§5.8) |
| The **DOJ release and DPA documents**, plus the victim-witness page | justice.gov | ✅ release retrieved; DPA documents not ⚠ |
| Any **SEC/CFTC instrument** in the Latour matter, once located | sec.gov / cftc.gov | ⚠ unreachable this pass |
| The **post-trade vendor announcement** (Torstone, 19 May 2022) — evidence of the books-and-records stack | A-Team Insight | ✅ public |
| The **ventures arm's dated press releases** — evidence of the firm's market-structure view | tower-research.com/ventures | ✅ public, dated |
| A **completed due-diligence questionnaire** covering the §12.4 questions | The counterparty | ⚠ obtainable and private — the highest-value artefact a bank must actually ask for |
| **Audited financial statements of the contracting entity** | The counterparty | ⚠ not public; must be a **condition** (§13.5) |
| A **rating-agency report** | — | ❌ none exists for any Tower entity (§10.2) |

### 12.6 What This Guide Will Not Name

Stated as a rule, not a caveat:

- **No bank, broker, dealer, exchange, clearing house, administrator, fund or institutional investor is named anywhere in this guide as Tower Research Capital's counterparty, client, clearing member, prime broker, lender, shareholder or venue** ✅/❌. The repository record contains no such disclosure, and none may be manufactured from the firm's size, its product list, its real-estate footprint or the necessity of its operations.
- **The two European SI entities are named only because the firm itself names them** ✅, and they are named as the firm's own entities — not as anyone's counterparty.
- **Torstone is named only because the firm's selection of it was announced in named trade press on a stated date** ✅, and it is named as a **vendor**, which is the fullest extent the source supports.
- **No venue may be asserted**, even though the firm says "150+ trading venues" ✅ — the venues are unnamed, and no exchange may be listed as one of them.
- **The office addresses are not evidence of regulated activity in those jurisdictions** (§9.3), and no local regulator may be described as supervising a Tower entity on the strength of an address.
- Where a bank wants a counterparty list, it must obtain one **from the client** and treat it as a representation — the one class of evidence this guide cannot supply and will not fake.

---

## 13. The Cymbal Bank Worked Example

> **Illustrative and fictional.** The scenario, the counterparty name, entity names, limits, ratios and currency amounts in this section are **invented for illustration**. They are consistent with the public facts established in §1–§12, but **no number here is a disclosure by Tower Research Capital, by Cymbal Bank, or by any real institution**. The persona and worked-example conventions follow **[Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md)** and **[MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)**; the worked-example format for a firm of this type follows **[Hudson River Trading](hudson_river_trading_guide.md) §10** and **[Jump Trading](jump_trading_guide.md) §14**. **Cymbal Bank is the only bank persona in this guide**, and the counterparty in the example is a **fictional firm — "Bastion Quantitative" — not Tower Research Capital and not any real firm.**

### 13.1 The Scenario

**Cymbal Bank** (Singapore) is asked to onboard a global proprietary market maker — call it **"Bastion Quantitative"** — into two roles at once: as a **liquidity provider** to Cymbal's client-facing FX and cash-equities execution, and as a **clearing counterparty** for the portion of its Singapore-booked derivatives activity that Cymbal clears. Bastion is Tower-shaped in every dimension that matters: **founded in the late 1990s in New York; proprietary (no clients, no fund); a multi-asset electronic market maker with equities, ETF and OTC FX/metal product lines; a Singapore office and a wider Asian footprint; a published engineering organisation with scale claims and no architecture; a ventures arm; one settled company-level spoofing matter arising from a single trading team's conduct more than a decade earlier; one unverified older secondary claim about a subsidiary's net capital; and no published financial statements of any kind.**

The two requests in front of Cymbal are: (i) a **liquidity-provider panel slot** on the FX stream, with a **depth-share ceiling under negotiation**; and (ii) a **clearing-client limit** for derivatives activity booked in Singapore, for which Bastion has offered **no financial statements** and has asked Cymbal to rely on its "size and global footprint". **All names and numbers invented.**

### 13.2 Entity Identification — Which Entity Contracts

The first control, and the one §2 exists to serve. Cymbal's onboarding team has taken the easy first pass and screened the **brand**, which returned nothing adverse; the brand is not the obligor.

| Relationship component | The entity Cymbal should contract with | Why, on the public record (Bastion modelled on §2.1) |
| --- | --- | --- |
| **FX liquidity provision (OTC spot and metals)** | The entity the client identifies as the FX-facing principal — which on the Tower analogue is **not published** ⚠ | The firm's own page shows it distributes FX liquidity but names **no** entity for it ✅/❌ — so the name must come from the client, verified at a register |
| **Equity liquidity provision in Europe** | If European equities are in scope, the **named systematic internaliser** entity (the Tower analogue names **Tower Research Capital Europe Limited** and **Tower Research Capital Europe B.V.**) ✅ | These are the only regulated-facing entities the firm itself names; SI status is a MiFID II designation, so a register check is available and cheap |
| **US equities/ETF liquidity** | Confirm what the identifier on the firm's own page legally is (the Tower analogue is **TRCX**, undefined) ⚠ | An identifier the firm does not define must be resolved to an entity **before** it goes in the documentation — never after |
| **Singapore-booked derivatives clearing** | The entity that holds the Singapore book; **status not established** ❌ | The Tower analogue publishes a Singapore **address** and **no** Singapore entity ⚠ — this is the gap Cymbal must close **with the client** |
| **The ventures business** | **Out of scope** — a venture investor is a separate obligor group ✅/❌ | Portfolio companies are not subsidiaries or counterparties; the file must not aggregate them (§4.5) |
| **Any group-level guarantee** | **None may be assumed** ❌ | The ownership chain is not public (§2.5), and a shared brand is not a guarantee |

**Two cheap controls fall out.** Screen **every** name associated with the contracting entity's register entry, not just the brand — the Tower analogue is that the group is multi-entity and only two of its European names are public. And **record the obligor in the facility documentation**, because a name cluster is not a company.

### 13.3 The Counterparty-Credit Approach Given Non-Disclosure

Cymbal's credit committee has no statements, no audited accounts, no rating and no supervisor. The four substitute inputs, worked through, all numbers invented:

| The missing input | The substitute Cymbal uses | What it produces in the file |
| --- | --- | --- |
| **Financial statements** | A written request for **audited accounts of the contracting entity**, made a **condition** of the limit rather than a right (see §13.5); the firm's **public silence on financials** recorded as a fact about disclosure posture (§10.5), not as a red flag | A capital position for one entity, **or a documented refusal** — and either is an improvement on a blank |
| **A prudential regulator's view** | **The conduct record as a governance proxy**: a settled company-level matter whose remediation is documented by the authority (early termination of the responsible team, surveillance investment, governance changes, senior-management change) ✅ and whose conduct window closed in **2013**; plus one older **unverified** secondary claim about a subsidiary's net capital, carried as ⚠ with its named source | A governance score driven by **remediation behaviour and the age of the conduct**, not by headline size |
| **A rating or credit research report** | **Nothing exists** ❌ — and Cymbal does **not** substitute a peer's rating, a peer's revenue or an industry average | A limit sized to **collateral and tenor**, because an earnings multiple cannot be computed |
| **A loss history** | **None published** ❌. The closest proxy is the firm's own continuity evidence: 28 years of operation, a 2019 CEO succession into a technology-side leader, dated product and engineering disclosures into 2026, a dated post-trade vendor selection | A **medium-high** internal grade, capped by structural unobservability |

**The grade Cymbal reaches (invented): "medium-high", capped.** Not because of anything adverse — because the inputs that would justify a higher grade **do not exist**. For any proprietary trading firm, **an unobserved counterparty cannot be a high-grade counterparty**, and stating that to a credit committee *before* a senior client's reputation is raised is the whole point of the discipline. The cap is not a judgement about Bastion; it is a statement about the bank's knowledge.

### 13.4 How the Documented Conduct Record Informs the Assessment

This is the section's core, because it is where banks most often go wrong in both directions: treating a settled matter as an open allegation, or waving it away because it is old.

**What Cymbal does with it, in order:**

1. **Cites it correctly** — authority, instrument, date, parties, outcome: the DOJ release number and date, the DPA instrument, the entity that entered it, the amount, the conduct window, and the fact that the **individuals** were separately charged and pleaded (§5.2, §5.5). The file text may not say "Bastion/Tower settled a fraud case", because that sentence drops the instrument, the parties and the posture.
2. **Keeps the entities and the postures separate.** The company-level matter attaches to the LLC; the individual matters attach to three people, one of whom had a **pending** charge as of the last authority document read ⚠, while the reported class-action and sentencing items are **secondary** and dated ⚠. The file may not aggregate them into "the firm's executives pleaded guilty" — a sentence this record does not support ✅/❌ — and it may not convert the firm's deferral into an admission either.
3. **Dates the conduct, and credits the remediation in the authority's own words.** The window closed in **December 2013** with remediation recorded in **early 2014** ✅, so a control assessment anchored to it is a statement about a firm 13 years younger; the file says so and asks what has changed. And it quotes DOJ rather than paraphrasing generously: cooperation, extensive remedial efforts, the early termination of the three traders, trade-surveillance investment, governance changes and a senior-management change ✅ — a firm that self-detects, terminates and remediates within a year of a control failure is exhibiting the behaviour a counterparty-credit analyst looks for.
4. **Turns the record into a question, not a verdict.** Two questions go into the questionnaire: *how does your surveillance detect spoofing-like patterns in your own order flow today, and how is that reported?* — mapping directly onto the control the authority records as strengthened — and *what changed in your control framework after the conduct window, and who owns it?* — unanswerable from outside and answerable from inside.

**And how Cymbal avoids the opposite error:** it does not write "no relevant regulatory history", because the firm **does** have a company-level matter and a KYC file that omits it is inaccurate. The correct entry is the dated record itself, with its posture and its remediation. Accuracy runs in both directions, and a file that only errs in the counterparty's favour is as wrong as one that only errs against it.

### 13.5 The Recommendation and the Condition

**Cymbal Bank's recommendation (invented, and every figure in it invented):**

| Element | Cymbal's decision | The reason, tied to a documented fact |
| --- | --- | --- |
| **Decision** | **Approve, with limits and one condition** | The conduct record is dated and remediated ✅; the product set is published and legible ✅; the financial inputs do not exist ❌ and so the limits are built from what does |
| **Liquidity-provider panel slot** | **Granted, with a 25% maximum depth share per provider**, reviewed quarterly with mark-out analysis by provider | Liquidity is optional in stress and a proprietary firm trading its own capital has no obligation to quote (§12.2); concentration is a limit in its own right, distinct from credit |
| **Clearing-client limit (Singapore-booked derivatives)** | **US$75 million gross notional**, with **margin required at the regulated minimum plus a 25% add-on** for unobserved-model risk; **no unsecured intraday excess beyond US$5 million** | The only scalable input is collateral (§12.4 row 10); an add-on prices the fact that the bank cannot see the firm's own risk models |
| **Tenor** | **Rolling 6-month review**, with monthly collateral and exposure reporting | The unobservable window is bounded by tenor, which is the cheapest lever a bank has |
| **Entity scope** | Papered **only** with the entities identified in §13.2; the ventures business and any non-contracting affiliate **expressly out of scope**; **no group guarantee assumed** | The ownership chain is not public ⚠; the Tower analogue names two European entities and no Asian one |
| **The condition Cymbal attaches** | **Delivery, within 60 days of first drawdown, of (i) audited financial statements of the contracting entity and (ii) the register entry and authorisation status of every entity in the contracting chain.** Failure to deliver caps the facility at the fully-collateralised amount already approved | Every substitute input Cymbal has is **behavioural** — a product list, a conduct record, an engineering claim ✅ — and none is **financial**. The condition is the mechanism that converts a public-disclosure posture into a private one, and it is stated as a condition rather than assumed as a right because the firm has never claimed otherwise |
| **Conduct undertakings** | The surveillance and control questions in §13.4 written into the agreement as **annual attestation items** | The one control the authority records as strengthened becomes the one the bank can monitor directly |

**The recommendation in one paragraph, and why it is the right shape.** Cymbal does not decline, because there is nothing in the record to decline on: the conduct window closed in 2013, the remediation is documented by the authority in its own words, and the firm publishes more counterparty-facing detail than most of its peer set. Cymbal does not underwrite it as a bank-shaped counterparty either, because the prudential inputs do not exist. **What Cymbal does is convert the unobservable into the bounded**: collateral instead of earnings, tenor instead of capital, depth caps instead of relationship goodwill, attestations instead of supervision, and a stated condition instead of an assumed disclosure. And the conclusion is structural rather than firm-specific: the same shape of recommendation is reached for the peer firm type at **[Hudson River Trading](hudson_river_trading_guide.md) §10.6** and at **[Jump Trading](jump_trading_guide.md) §14.5**, from different evidence — which is the strongest available sign that the approach is a property of the firm type and not of this firm.

### 13.6 Regulatory-Reporting Consequences

A bank's dealings with a firm of this type create obligations on the **bank's** side, and the entity question recurs in every one of them:

- **Counterparty identification and classification.** The bank must record the **correct legal entity** and its beneficial-ownership chain for KYC, sanctions and prudential purposes; where the chain cannot be established publicly, the bank's file must record the **basis** on which it was satisfied — a client representation is a legitimate basis, an inference from a brand is not (§14.1, anti-pattern 6).
- **Prudential treatment.** Exposures to a **regulated financial counterparty** and to a **corporate principal** are treated differently and attract different risk weights and large-exposure outcomes. Because no Tower-shaped firm's regulated status is publicly established at the contracting entity level ❌, the classification must come from the register check in the §13.5 condition — **and it may differ between Cymbal's Singapore booking entity and any overseas subsidiary**.
- **Market-access and pre-trade-risk obligations.** Where Cymbal provides market access of any kind, the bank may itself fall inside the market-access and pre-trade-control expectations in its jurisdictions (SEC Rule 15c3-5 in the US; MAS/SGX expectations in Singapore), the architecture for which belongs to **[Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md)**. For **this** firm type, note the point worth putting in the file: the one company-level matter in Tower's record **is** a market-conduct matter arising from exactly the order-entry behaviour those controls exist to prevent ✅ (§5.2, §8.3).
- **Transaction reporting, conduct and disclosure.** Every trade must carry the correct counterparty identifier resolving to the correct entity — the same identification problem as KYC, revisited in the reporting layer — and routing client flow to the firm triggers best-execution, order-handling-disclosure and conflict-management obligations, with the **disclosure spectrum** the firm's own page advertises ("direct, disclosed… semi-disclosed") becoming a **disclosure decision the bank must document** (§12.2).
- **Operational-resilience, third-party and financial-crime risk.** A firm whose liquidity a bank's client-facing execution depends on is a **critical third-party service provider** under operational-resilience expectations — exit planning, substitutability analysis and concentration reporting all apply, and the §13.5 depth-share cap is the trading-side expression of that requirement. Multi-entity groups with an unpublicised ownership chain and named individuals in an enforcement record require entity-level screening and personnel-level enhanced review, recorded with the firm-level and individual-level matters kept distinct (§5.5).

---

## 14. The Anti-Patterns

### 14.1 The Table — Symptom, Cause, Guardrail

| # | Anti-pattern | Symptom in the file | Cause | Guardrail |
| --- | --- | --- | --- | --- |
| 1 | **Asserting a technical architecture from a blog post or a peer's disclosure** | "Uses FPGA feed handlers and kernel bypass on colocated X hardware" — a confident sentence with no source | The genre's secondary literature describes the firm *type*; a peer's disclosure gets read as this firm's; a vendor case study is read as a customer list | **Attribute only what the firm itself states.** For Tower that means the §6.2 list (in-house stack, dozens of colocation centres, hundreds of engineers, 12+ offices, 150+ venues, 100 petabytes, C++/Python/Rust, ML/FPGA/low-latency, wireless telecommunications, hardware acceleration) plus the one named vendor (**Torstone**, post-trade, 19 May 2022) — and **nothing else** (§6.4). A peer's architecture is not this firm's architecture |
| 2 | **Repeating a regulatory matter without authority, instrument and date** | "Tower was fined $67m for spoofing" or "Tower is under a DOJ investigation" — a sentence no screening system can verify | Regulatory matters are the most retold and least re-read claims in the genre; the telling compresses the instrument away | **Every matter carries authority + instrument + date + parties + outcome, or it does not go in the file** (§5.9). For this firm: **DOJ, Release No. 19-1,208, 7 Nov 2019**, a **deferred prosecution agreement** against **Tower Research Capital LLC**; **DOJ, Release No. 18-1328, 12 Oct 2018**, three **individuals**, the firm **not named**. Neither is a finding of fraud against the firm ✅ |
| 3 | **Conflating a matter against individuals with one against the firm** | "Tower's traders pleaded guilty, so the firm admitted fraud" — or the reverse, a file that records only the firm-level DPA and omits the individuals entirely | The conduct was the same and the press covers it as one story; the parties and the instruments are different | **Keep the tables separate and label the posture of each instrument** (§5.2, §5.5). The individuals **pleaded guilty**; the firm entered a **DPA** whose release **does not address admissions** ⚠. Do not borrow the individuals' admission to characterise the firm, and do not use the firm's deferral to excuse the individuals |
| 4 | **Treating a reported figure as current, or as the firm's own** | "Tower: 1,400 employees; $67m in penalties" — dated numbers in present-tense fields | Figures migrate without their dates; enforcement amounts read like financial data; infobox numbers read like disclosures | **Every figure carries outlet + date + basis, or it does not go in the field** (§10.2, §10.3). The penalties are **2019** payments, **not revenue**; the headcount figures are **2025/2026** and **contradict each other**; the **Latour $16m** is a **2014 secondary claim** ⚠. And the DOJ and CFTC amounts are **cross-credited** — adding them double-counts |
| 5 | **Inferring a counterparty relationship from the firm's size, or from necessity** | "Tower clears through Bank X" / "Tower is a client of Broker Y" / "Tower uses Vendor Z's colocation" | A firm this size obviously needs clearing, financing and connectivity, so the inference feels safe; and named peers' relationships are visible in the same guides | **No bank, broker, exchange, administrator or counterparty may be named unless that party or the firm has publicly said so** (§12.6). The firm's own liquidity page may be quoted for **what it offers** ✅; that is not a counterparty list. Torstone is citable because the firm's selection of it was **announced, dated and attributed** ✅ |
| 6 | **Onboarding a different entity than the one that bears the risk** | The facility papered with the brand, the group, or a sibling; a guarantee assumed from a shared name; the ventures arm aggregated into the trading obligor | The names are confusable, the group is multi-entity, and the ownership chain is **not public** ⚠ | **Establish the contracting entity and its register entry first, record the obligor, and name the non-contracting entities as expressly out of scope** (§13.2, §13.5). Screen every register name, not the brand — the Tower analogue names two European entities and no Asian one, and leaves one identifier (**TRCX**) unexplained ⚠ |
| 7 | **Treating the firm's silence as a finding of misconduct — or reading its marketing as reassurance** | "Publishes no accounts, therefore risk" — or the mirror image, "lives by five values, therefore safe" | Opacity is uncomfortable and values pages are comforting, and both feelings get written down as analysis | **The opacity is lawful and common for a private firm** (§2.3) and the correct response is to **record the absence and bound the limit** (§10.5, §13.5). Equally, **conduct is a matter of record, not of public statements** (§8.4): an engineering page and a values list are not evidence about conduct, and a 2019 matter is not evidence about today |
| 8 | **Comparing penalty sizes across firms as a proxy for misconduct** | "Tower's $67m is small next to Peer X's $123m" — an implicit ranking from unrelated instruments | Penalty headlines are the only firm-specific numbers this firm type produces, so they get used as a metric | **Compare instruments, not amounts.** Different authorities, respondents, eras and statutes produce different numbers (§11.3). The right question is what the record says about **controls and remediation**, which for Tower is the authority's own remediation language ✅ (§5.2, §13.4) |

### 14.2 The Unifying Failure

Every row above is one mistake wearing different clothes: **a claim migrating from a dated, attributed source into a file as an unattributed fact.** The number loses its date, the instrument loses its authority, the entity loses its name, the architecture loses its provenance, and the counterparty relationship is born from necessity rather than from a source. The guardrail is always the same instruction — **keep the outlet, keep the date, keep the instrument, keep the entity; and where a private firm discloses nothing, write the absence down and bound the exposure instead of filling the gap with something that looks like an answer.**

---

## 15. The Claims Audit

### 15.1 The Verified Claims (✅)

| Claim | Source | Date | Quality |
| --- | --- | --- | --- |
| **Deferred prosecution agreement**: Tower Research Capital LLC entered a DPA resolving criminal charges; criminal information filed in the SDTX charging the company with **one count of commodities fraud**; joint motion to defer prosecution and trial, subject to court approval; combined **US$67.4m** in criminal monetary penalties, disgorgement and victim compensation, cross-credited for CFTC payments; compliance reviews and programme modification agreed | **DOJ Office of Public Affairs Release No. 19-1,208** | **7 Nov 2019** | Primary (authority release, retrieved at justice.gov) |
| Conduct described: **March 2012 – December 2013**; three traders, "**a single trading team at Tower**"; E-Mini S&P 500 and NASDAQ 100 (CME) and E-Mini Dow (CBOT); thousands of orders with intent to cancel | Same DOJ release | **7 Nov 2019** | Primary |
| Remediation, verbatim: Tower "**swiftly moved in early 2014 to terminate the three traders**", invested in trade surveillance, increased legal and compliance resources, revised corporate governance, changed senior management | Same DOJ release | **7 Nov 2019** | Primary |
| **Parallel CFTC settlement**: ≈US$67.4m including a **US$24.4m civil monetary penalty**, plus restitution and disgorgement cross-credited to DOJ; a **CFTC order** with remedial and cooperation obligations; the CFTC Division of Enforcement referred the matter to DOJ | DOJ's description in Release No. 19-1,208 | **7 Nov 2019** | ✅ as description / ⚠ not read at cftc.gov |
| **2018 charging release**: three traders charged; two agreed to plead guilty; firm referred to as "**Trading Firm A**" (a second Chicago firm as "Trading Firm B"); alleged window ≈**Mar 2012 – Mar 2014**; losses "over **$60m**"; the release's own "**merely allegations**" caution | **DOJ Release No. 18-1328** | **12 Oct 2018** | Primary |
| **Gandhi**: pleaded guilty to two counts of conspiracy to engage in wire fraud, commodities fraud and spoofing; sentencing scheduled 7 Feb 2020 before Judge Ewing Werlein Jr. (SDTX) | DOJ Release No. 19-1,208 | **7 Nov 2019** | Primary |
| **Mohan**: pleaded guilty to one count; sentencing scheduled 13 Feb 2020 before Judge Gray H. Miller (SDTX) | DOJ Release No. 19-1,208 | **7 Nov 2019** | Primary |
| **Mao**: indicted October 2018 (SDTX); one count conspiracy, two counts commodities fraud, two counts spoofing; charges **pending** as of the release | DOJ Releases No. 18-1328 and No. 19-1,208 | 12 Oct 2018 / 7 Nov 2019 | Primary |
| Founded **February 1998**, New York; founders **Mark Gorton** and **Alistair Brown**; HQ New York City; "one of the oldest automated trading firms" | Wikipedia (citing CNBC, Business Insider; Bloomberg for the age claim) | retrieved 27 Sep 2026 | Secondary, consistently reported |
| **Albert An** CEO; **Mark Gorton** chairman — founder stepped down as CEO in **August 2019**, replaced by An, who joined in 2016 as technology lead | Firm's own leaders page ✅; Wikipedia citing Bloomberg (Abelson & Leising) | page retrieved 27 Sep 2026; transition **1 Aug 2019** | First-party + named press |
| Offices page: **14 city entries** with addresses, including **Singapore (Marina One West Tower, 9 Straits View, 018937)**, New York (120 Broadway and 377 Broadway), London, Gurugram, Amsterdam, Charleston, Chicago, Dubai, George Town, GIFT City, Hong Kong, Montreal, Paris, Shanghai | tower-research.com/offices | retrieved **27 Sep 2026** | **First-party primary** |
| Engineering claims, verbatim: proprietary technology "for every aspect of trading"; "**Dozens of colocation centers**"; "**Hundreds of engineers** supporting **12+ global offices** and **150+ trading venues**"; "Managing **100 petabytes** of data"; "**C++ and Python** while strategically investing in **Rust**"; "machine learning, **FPGA technology**, low-latency programming"; "We build almost everything in-house" | tower-research.com/engineering | retrieved **27 Sep 2026** (re-fetched; unchanged) | **First-party primary** |
| Liquidity-provision page: "on-exchange market making"; "bilateral, disclosed liquidity"; **US equities and ETFs (TRCX; ETF Block Trading Platform)**; **two Systematic Internalisers — Tower Research Capital Europe Limited (Tower UK) and Tower Research Capital Europe B.V.**; **global FX in OTC spot and precious metals**; distribution "direct, disclosed… semi-disclosed"; "warehouse many different sizes of risk" | tower-research.com/liquidity-provision | retrieved **27 Sep 2026** | **First-party primary** |
| About Us: "**Over 25 years**"; values **Excellence, Respect, Innovation, Integrity, Teamwork**; four business-unit pages | tower-research.com/about-us | retrieved **27 Sep 2026** | First-party |
| **Torstone Technology** selected for global cross-asset **post-trade** processing (SaaS); COO **Alan McGroarty** quoted ("unprecedented levels of growth"; "diversify into new asset classes and trading strategies, at ever increasing volumes") | A-Team Insight, "Tower Research Selects Torstone…" | **19 May 2022** | Named trade press, firm-selected announcement |
| **Tower Research Ventures**: pre-seed/seed/incubation; Director of Venture Capital **Jared Young**; dated releases (Procurement Sciences Series B 5 Nov 2025; Mentium 9 Oct 2025; Atomic Canyon 28 May 2025); blogs 10 Mar 2026, 16 Apr 2026, 23 Jul 2026 | tower-research.com/ventures | retrieved **27 Sep 2026** | First-party |
| Careers: roles across Quantitative Trading / Core Engineering / Business Support; benefits incl. discretionary and team-based bonuses, 5 weeks vacation, 401(k) match (US), in-office meals; "no gotcha questions" | tower-research.com/careers | retrieved **27 Sep 2026** | First-party |
| **Mark Gorton** — creator of **LimeWire** (2000), chief executive of the Lime Group; education Yale (BEng), Stanford (MEng), Harvard (MBA); engineer at Martin Marietta, then fixed-income trading at Credit Suisse First Boston | Wikipedia (Mark Gorton), citing HuffPost, D.M. Levine | bio source **18 Apr 2012**; retrieved 27 Sep 2026 | Secondary, well-cited |
| ***Arista Records LLC v. Lime Group LLC*** — 2010 finding of personal liability for Gorton and Lime Group; permanent injunction; 2011 settlement of **US$105m** to the RIAA | WSJ (Chad Bray, 26 Oct 2010); CNET (Greg Sandoval, 12 May 2011), via Wikipedia | 2010 / 12 May 2011 | Named press — **a Lime Group matter, not a Tower matter** |
| MAHA Institute launched May 2025, co-presided with Tony Lyons; 2026 roundtable statement on the childhood vaccination schedule | STAT (Daniel Payne, 15 May 2025); NOTUS (Margaret Manto, 9 Mar 2026), via Wikipedia | 2025–2026 | Secondary, named and dated; **no source links these to Tower** |

### 15.2 The Flagged Claims (⚠)

| Claim | Source | Date | Quality |
| --- | --- | --- | --- |
| **Latour Trading** fined a record **US$16m** for violating the net capital rule; "deliberately mis-estimated its exposure to risk and traded despite not holding enough capital"; subsidiary "sometimes accounted for 9% of U.S. stock trading" | WSJ, Scott Patterson, "High-Frequency Trading Firm Latour to Pay $16 Million SEC Penalty", via Wikipedia | **17 Sep 2014** | ⚠ **Secondary; not verified at sec.gov (unreachable this pass). No release number, file number or order date asserted** |
| Class-action settlement of **US$15m** | financefeeds.com (Andrew Sax-McLeod), via Wikipedia | **3 Feb 2021** | ⚠ single-sourced secondary |
| Second Circuit upheld Tower's win in the Korean investors' spoofing case — Korean exchange trading not subject to the US Commodity Exchange Act | Reuters (Jody Godoy), via Wikipedia | **22 Jun 2021** | ⚠ named press, dated; opinion not retrieved |
| An ex-Tower trader avoided jail in a spoofing case (**trader not named**) | Law360 (Rachel Scharf), via Wikipedia | **9 Jul 2021** | ⚠ headline only |
| **Limestone Trading** — internal team; 2025 crypto expansion and capital allocated to crypto strategies | Bloomberg (Anto Antony), via Wikipedia | **5 May 2025** | ⚠ secondary; not on the firm's own pages |
| **Jersey subsidiary** planned | Financial News London (Lars Mucklejohn) | **21 Apr 2026** | ⚠ headline-and-citation only, **not verified** |
| Fixed-income ETF expansion | Financial Times (Jill Shah) | **19 Jun 2026** | ⚠ paywalled, headline-only via Wikipedia's citation |
| Employees "more than 1,100 (2025)" vs "c. 1,400+ (2026)" | Wikipedia (About Us citation; infobox) | 2025 / 2026 | ⚠ **discrepancy flagged, not resolved** |
| "11 locations" (Wikipedia) vs **14 city entries** (live page) vs "12+ global offices" (firm engineering page) | Wikipedia; tower-research.com | retrieved 27 Sep 2026 | ⚠ **discrepancy flagged; live page treated as best evidence** |
| 2023 lease of **121,903 sq ft at 120 Broadway**, consolidating two NYC offices — while the live page still lists **377 Broadway** | Commercial Observer (Mark Hallum); Bisnow (Ciara Long), via Wikipedia | **6 Sep 2023** | ⚠ secondary; apparent inconsistency with the live page flagged |
| "Gorton owns Tower Research Capital LLC" | Wikipedia (Mark Gorton) | retrieved 27 Sep 2026 | ⚠ carries a **"citation needed"** tag |
| Tower described as "**a hedge fund**" | Wikipedia (Mark Gorton) | retrieved 27 Sep 2026 | ⚠ **inconsistent with Wikipedia's own Tower article; the repository treats the firm as a proprietary trading firm** |
| Whether Tower "runs microwave links" or any specific transport | no source located | — | ⚠ believed and repeated widely; **not sourced, not asserted here** |

### 15.3 The Rejected or Not-Found Claims (❌)

| Claim | Why rejected / not found |
| --- | --- |
| That **Tower Research Capital is a hedge fund** | ❌ Contradicted by the firm's own self-description and by Wikipedia's Tower Research Capital article; it has no external investors and trades its own capital (§1.2) |
| That the firm has **published financial statements, revenue, capital or a valuation** | ❌ Not found in any source read this pass, for any entity (§10.1) |
| That the firm is **any named bank's client, counterparty, or liquidity provider**, or that any named institution is its clearing member or prime broker | ❌ No source, and prohibited by §12.6 |
| That any **specific venue, exchange or data centre** is one of the firm's "150+ trading venues" or "dozens of colocation centers" | ❌ The firm names none; no source supplies one |
| That **"TRCX" is an ATS, a venue or a broker-dealer** | ❌ The identifier appears on the firm's page; what it legally is, is not stated and is not inferred (§2.5) |
| That the firm has a **Singapore legal entity named "Tower Research Capital Singapore"** or any similar name | ❌ Not found; the absence reflects an unperformed register search (§9.2, §16.1) |
| That **Tower admitted the DOJ allegations**, or that it settled "without admitting or denying" | ❌ Neither formulation appears in the authority's release, which does not address admissions (§5.2) |
| That the DOJ **criminal information has been dismissed**, or that the **DPA term has expired** | ❌ Not established; the release does not state the term and no later document was retrieved (§5.10) |
| That **sentences were imposed** on Gandhi, Mohan or Mao | ❌ Only scheduled sentencing dates and a pending indictment are in the record (§5.5) |
| That the firm's conduct window is the **2018 release's window** applied to a **named** firm | ❌ The 2018 release does not name Tower (§5.4) |
| That the firm runs a **specific microwave network, FPGA vendor, kernel-bypass stack or latency figure** | ❌ Not published (§6.4) |
| That the founder's **LimeWire or advocacy activity** has any bearing on Tower Research Capital | ❌ No source links them; this guide draws no such link (§3.4) |

### 15.4 What Could Not Be Verified

Authority documents not retrieved this pass (recorded as **tool limitations**, not as evidence of absence): **sec.gov** (three attempts: EDGAR full-text search endpoint, a press-release URL, and the administrative-proceedings index — all failed); **cftc.gov** (the CFTC release URL failed) — so **every SEC and CFTC instrument in this file is either secondary-sourced ⚠ or recorded through DOJ's description ✅/⚠**. The host's `web_search` is **non-functional** and returned empty results, so **no absence in this guide is a negative finding**.

Unverified items, itemised: the **Singapore entity** (name, number, incorporation date); the **Singapore office's establishment year**; **headcount by entity or office**; the **ownership chain and percentages**; the **Jersey subsidiary**; the **"c. 1,400+" 2026 headcount figure**; **Latour Trading's current status and the 2014 SEC matter's primary document**; the **disposition of the Mao indictment** and the **sentences of Gandhi and Mohan**; the **2021 class-action settlement and the Korean-futures opinion**; the **identity of the ex-Tower trader in the 2021 Law360 item**; any **register entry for any Tower entity in any jurisdiction**; the **DPA's term length and current status**; what **"TRCX" is**; whether the **crypto team trades in Asia**; and whether the firm's **office list is current** in the light of the 2023 New York consolidation. Each is carried with its marker in §5.10, §9.5 or §10.3, and none is filled by inference.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

**Summarised in one place so that a reader can see the shape of the uncertainty**: Tower Research Capital is a firm whose **existence, longevity, founders, leadership, offices, product lines, entity names at the European edge, engineering claims and enforcement record** are all documented, and whose **financial position, ownership structure, registration footprint, internal organisation below the business-unit level, architecture and Asia entities** are not documented at all. The second list is longer than the first, and it is longer for **two different reasons that must not be conflated**: (i) **the firm does not publish it** — the financials, the architecture, the venue list, the ownership (a lawful choice for a private firm, §2.3); and (ii) **this pass could not reach the sources that would settle it** — sec.gov, cftc.gov, the FCA/AFM/ESMA registers, FINRA BrokerCheck, ACRA, the SFC, the IFSCA, and a working web search (§15.4). **The correct file entry for every item in category (ii) is "not yet verified", never "not applicable"** — the distinction is the difference between an honest gap and a fabricated finding, and this guide has maintained it line by line.

### 16.2 Glossary

| Term | Definition as used in this guide |
| --- | --- |
| **Proprietary trading firm** | A firm trading its own capital as principal, with no clients and no external investors — the firm type this guide covers (§1.4) |
| **Electronic market maker** | A firm posting continuous two-sided prices electronically and earning the spread while carrying inventory — Tower describes "on-exchange market making" ✅ |
| **Latency** | The time between an event and the firm's reaction to it; the fundamental constraint on electronic trading, set by physics rather than by software |
| **Co-location** | Renting space, power and connectivity inside or beside an exchange's data centre so a firm's servers sit as close as possible to the matching engine — Tower claims "dozens of colocation centers" ✅ |
| **Microwave / millimetre-wave link** | Low-latency wireless transport between venues, faster than fibre over distance **in principle**; **not documented for Tower** and not asserted here (§6.4) |
| **Spoofing** | Placing orders with the intent to cancel them before execution, to create a false impression of supply or demand — the offence in the 2019 DPA's underlying conduct ✅ (§5.2, §8.3) |
| **Commodities fraud** | The US offence under which the criminal information against the firm was filed ✅ |
| **Deferred prosecution agreement (DPA)** | An agreement under which the authority files charges but defers prosecution for a term, subject to court approval and compliance conditions — the instrument in the 2019 matter ✅ |
| **Criminal information** | A charging document filed by prosecutors without a grand jury indictment — the instrument filed against the firm on 6 Nov 2018 ✅ |
| **Indictment** | A formal charge returned by a grand jury — the instrument against Mao ✅ |
| **Disgorgement / civil monetary penalty** | Surrender of unlawfully obtained profits, and the CFTC's civil fine (US$24.4m) — part of the ≈US$67.4m settlement, cross-credited ✅ |
| **Systematic internaliser (SI)** | A MiFID II designation for a firm executing client orders against its own book systematically — Tower names **two** ✅ |
| **ETF / block trade** | Exchange-traded fund, and a large trade negotiated away from the continuous book — Tower advertises "unique block trading capabilities" ✅ |
| **OTC (over-the-counter)** | Bilateral trading outside a central venue — the mode of Tower's FX and precious-metals liquidity ✅ |
| **Counterparty** | The party on the other side of a trade, with **no fiduciary duty** — the word Tower itself uses ✅, distinct from a client |
| **Clearing member** | A member of a clearing house through which another firm clears; the route by which a proprietary firm typically reaches a CCP |
| **CCP** | Central counterparty — a clearing house that novates itself into trades between members and their clients |
| **Prime brokerage** | A bundled service (clearing, financing, custody, FX, reporting) a bank provides to a trading firm |
| **Mark-out analysis** | Measuring the post-trade price movement of a counterparty's fills to quantify the cost or benefit of interacting with it (§13.5) |
| **Net capital rule** | The US broker-dealer capital requirement underlying the **unverified** Latour claim ⚠ (§5.6) |
| **Prudential supervision** | Regulation of capital, liquidity and safety-and-soundness — **no Tower entity was identified as prudentially supervised** ❌ |
| **Proprietary technology (firm's usage)** | The firm's own phrase for technology built in-house "for every aspect of trading" ✅ |
| **Enterprise resilience / third-party risk** | The regulatory lens that treats a critical liquidity provider as a service provider whose failure the bank must plan for (§13.6) |
| **Trade surveillance** | Systems that detect abusive or anomalous order-entry patterns — the control DOJ records Tower as having invested in ✅ |

### 16.3 Cross-References

**Sibling guides (`banking/`, plain filenames):** [Hudson River Trading](hudson_river_trading_guide.md) — the prop-firm archetype: §3 the business and franchise, §5 technology, §6 talent, §7/§8 regulatory and market-structure context, §10 the worked example, §11 the claims audit · [Jump Trading](jump_trading_guide.md) — the immediately preceding firm guide: §4 the no-clients model, §6 the regulatory-matter discipline, §7 capital where the finding is absence, §11 the Asia reconciliation, §14 the worked example, §15 the anti-patterns and claims audit · [Market Making and Electronic Liquidity in Singapore](market_making_singapore_guide.md) — **§4.6 owns the Singapore facts about this firm**, §5.1–§5.2 the home-grown firms, §6 the comparison table, §7 the SGX/clearing infrastructure · [Citadel LLC](citadel_llc_guide.md) — the archetype firm guide and §7 the technology of the firm type · [ExodusPoint](exoduspoint_guide.md) — the identity, capital and settled-claim recording conventions · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) — the Cymbal Bank persona and worked-example conventions · [Hedge Funds in Singapore](hedge_funds_singapore_guide.md) — the fund-management contrast.

**Technology guides (`technology/`, prefix `../technology/`):** [Low-Latency C++ Development](../technology/low_latency_cpp_development_guide.md), [Low-Latency Rust Programming](../technology/low_latency_rust_programming_guide.md) and [Low-Latency Java Programming](../technology/low_latency_java_programming_guide.md) — the latency discipline itself (§6.1) · [Trading System Software Architecture](../technology/trading_system_software_architecture_guide.md) — platform, market-access and pre-trade-control architecture (§6.1, §13.6) · [Quantitative Developer Skillset](../technology/quantitative_developer_skillset_guide.md) — the research and engineering skill stack (§7.1).

**Primary sources relied on this pass:** U.S. Department of Justice, Office of Public Affairs, Release No. **18-1328** (12 Oct 2018) and Release No. **19-1,208** (7 Nov 2019), both retrieved at justice.gov · tower-research.com (offices, about-us, engineering, liquidity-provision, ventures, careers, retrieved **27 Sep 2026**) · A-Team Insight, "Tower Research Selects Torstone for Global Cross-Asset Post-Trade Services" (**19 May 2022**) · Wikipedia, *Tower Research Capital* and *Mark Gorton* (retrieved 27 Sep 2026, as a **citation chain** for named press). Sources attempted and unreachable: **sec.gov** (three URLs), **cftc.gov** (one URL), and the host's `web_search` (non-functional).

### 16.4 Closing Summary

Tower Research Capital is the oldest firm in this repository's proprietary-trading cluster: founded in **February 1998** in New York by **Mark Gorton and Alistair Brown** ✅/⚠, still trading in 2026 under an **Albert An**–led executive with the founder as chairman ✅, and now the upstream name in its own sub-genre, since **Hudson River Trading was founded by Tower Research alumni** ✅. It trades its own capital across equities, ETFs, European cash equities through **two named systematic internalisers**, and OTC spot FX and precious metals; it runs a crypto team and a venture arm; it employs several hundred engineers on an in-house stack that it describes in more detail than most peers; and it publishes a full office list including **Singapore** ✅. It also publishes **no financial statements of any kind**, and the register questions a bank would normally ask — who is licensed, where, and by whom — **could not be answered this pass** because the relevant registers were unreachable ❌. What filled the gap is the record, and the record is a single dated and remediated company-level matter — a **deferred prosecution agreement announced on 7 November 2019** over the conduct of **three traders on a single trading team** between 2012 and 2013, resolved for **US$67.4m** with a parallel CFTC settlement including a **US$24.4m** civil penalty — alongside individual prosecutions kept strictly separate, and one older, **unverified**, secondary claim about a subsidiary's net capital ⚠. That record is what a bank can read, and reading it correctly — authority, instrument, date, parties, posture — is the whole discipline this guide has tried to demonstrate, because for a firm that never explains itself, the record is the regulator's.
