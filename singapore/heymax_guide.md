# HeyMax: The Rewards Aggregator and Its Universal Travel Currency — A Comprehensive Guide

**HeyMax — the Singapore-founded loyalty and travel-rewards platform operated by Max Now Pte. Ltd. — what it is and what it is not (a consumer rewards aggregator built on a proprietary currency, Max Miles, not a bank, not an issuer, and not an airline programme), the company and its founding record (established 2023 by four former Meta engineers, with CEO and co-founder Joe Lu the only founder HeyMax names first-party and the other three names carried on third-party and registry-derived sources), the funding record (a US$2.6 million seed announced 2 July 2024 led by January Capital; a US$11 million Series A announced 28 January 2026 led by Peak XV Partners, each with its own source and date and no later round assumed), the product (what Max Miles is as the company defines it, the four published ways of earning, the four ways of redeeming, and the vital distinction between the marketing promise that miles "never expire" and the Terms of Use that make the balance conditional on account activity and reserve the right to introduce expiry), the card-linking mechanism (the Visa-hosted enrolment page, the transaction stream, and the company's own admission that it tracks the issuer's rewards rather than its own), the business model as published (merchant- and partner-funded, governed by the company's own rule "if you don't earn, we don't earn", with the take rate recorded as not published), the consumer proposition and the double-dip, the structural competitive frame, the technology and data position, the AI positioning (a vision statement measured against one concrete published artefact), the cooperative-and-competitive issuer and network relationship, the evidence reality and how to weigh first-party pages against company claims and third-party opinion, the Cymbal Bank worked example (clearly marked design fiction), the anti-patterns and open questions, the full claims audit, the honest ledger of what could not be verified, and the glossary and closing summary — the dedicated HeyMax deep-dive of the Singapore shelf, written to sibling-guide house style.**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Singapore / Consumer Rewards, Loyalty and Card-Linked Commerce — the deep-dive on HeyMax: the entity (Max Now Pte. Ltd., founded 2023, Singapore-headquartered, operating as HeyMax), the money (seed and Series A only, both dated and sourced; no valuation published), the currency (Max Miles, and the gap between the "never expires" promise and the terms), the mechanism (Visa card-linking via Visa On Platform, and what that partnership does and does not establish), the model (partner-funded, structurally published; take rate not published), the reader-facing proposition, the architectural competitive frame, the data and AI positions, the issuer-and-network relationship read as an arrangement rather than a judgement, the evidence taxonomy, the Cymbal Bank worked example (design fiction), and the claims audit. Sibling guides carry the Singapore SaaS, payments, card and public-sector deep-dives; this guide cross-references them and does not re-derive their content.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** October 2026
> **Primary Sources:** heymax.ai — the About page ("Our Story"), the Terms of Use (last updated 6 February 2025), the Security page; blog.heymax.ai — the seed announcement (2 July 2024), the Series A announcement (28 January 2026) and "How we get you your Max Miles" (28 May 2025); help.heymax.ai — "Card Maximiser (Singapore users)" (updated 27 February 2026), "Card Maximiser: Security and Privacy" (20 June 2024), "Adding Cards Basics" (20 June 2024), "Max AI" (6 April 2026) and the "Max Miles Turns 3!" campaign terms (updated 22 September 2026); hk.heymax.ai — the Max Miles explainer page; plus the dated third-party record (fintechnews.sg, 2 July 2024 and 28 January 2026; Lobang Sis, 3 June 2026; Crunchbase, Preqin and Tenity for the attributed founder names; MileLion for campaign coverage). Every claim's mark, source, date and kind are given inline and consolidated in §14.
> **Companion guides (sibling, same `singapore/` folder):** [sginnovate_guide.md](sginnovate_guide.md) (the state deep-tech investor — the worked-example and claims-audit formats this guide inherits) · [tradenet_platform_guide.md](tradenet_platform_guide.md) (the national single window — the provenance/output-mode discipline this guide mirrors in §8 and §11) · [starhub_software_systems_guide.md](starhub_software_systems_guide.md) (a Singapore enterprise's software systems — a useful architectural contrast to a consumer rewards platform)
> **Companion guides (sibling, `../technology/`):** [singapore_saas_companies_guide.md](../technology/singapore_saas_companies_guide.md) (§7's ecosystem map is the boundary this guide respects: that section lists the sector's firms in a line apiece; this guide is the dedicated deep-dive on HeyMax and does not re-derive the sector aggregate). Card- and payments-cluster siblings carry the issuer, scheme and payment-rail mechanics that §4 and §10 assume rather than rebuild.

**How to use this guide:** Section 1 is the overview — the short answer, the decoder of terms, the key-facts table, the Cymbal Bank lens, the thesis, and the boundary that keeps this guide from re-deriving the SaaS, payments and card siblings. Section 2 is the company — the entity, the founding record, the four founders (and which name is first-party versus attributed), the mission, and the two funding rounds. Section 3 is the product — Max Miles, the four ways of earning, the four ways of redeeming, and the precise gap between the "never expires" promise and the terms. Section 4 is the card-linking mechanism — Visa, the Visa-hosted page, the transaction stream, and the company's own "we track the issuer's rewards, not Max Miles" point. Section 5 is the business model as published. Section 6 is the consumer proposition and the honest line between what can and cannot be established about value. Section 7 is the structural competitive frame. Section 8 is the technology and data position. Section 9 is the AI positioning. Section 10 is the issuer and network relationship, read as an arrangement of converging and diverging interests. Section 11 is the evidence reality. Section 12 is the Cymbal Bank worked example (clearly marked design fiction). Section 13 is the anti-patterns and open questions. Section 14 is the claims audit. Section 15 is the honest ledger of what could not be verified. Section 16 is the glossary, cross-references and closing summary. **Reading paths:** *Bank strategist:* §1 → §10 → §12 → §5. *Payments/issuer analyst:* §1 → §4 → §8 → §10. *Product/roadmap:* §1 → §3 → §9 → §13. *Compliance/data:* §1 → §3 → §8 → §11 → §14. *Investor/VC:* §1 → §2 → §5 → §7 → §14. *Sceptic/verifier:* §1 → §11 → §14 → §15. *In a hurry:* §1, §3, §5, §10, §14, §15.

**Integrity convention.** Every factual claim in this guide carries one of three marks: **✅** verified this pass against a named source (a first-party page, a first-party announcement, or the dated independent press record — all named inline and consolidated in §14); **⚠** flagged — reported, approximate, single-sourced, dated, fast-moving, or a company's own claim about itself; **❌** refuted or not found. Unmarked statements are domain-stable explanation (what a loyalty platform is, what a card-linked transaction feed is, how a rewards currency differs from money) rather than research claims. Two conventions matter for this subject specifically. First, a claim that a company publishes about itself is a **company claim**: it is marked ⚠ and attributed to the company with its date, never laundered into a guide-level fact. Second, where this guide states an absence — a valuation that is not published, a take rate that is not published, a first-party founder list that does not exist — the absence is the finding, and it is recorded rather than filled. HeyMax is an early-stage private company, so a large share of the load-bearing questions (unit economics, profitability, later rounds, partner redemption terms) simply have no public answer, and §15 says so plainly. **The research date for this pass is 7 October 2026**; every first-party page cited was read on or before that date, and dated figures are bounded to the date at which the company or its press stated them.

**A note on marks and hazards carried throughout.** Three traps recur in writing about a company like HeyMax, and this guide names them here once so it can meet them everywhere. (1) *The promise-versus-terms trap:* "never expires" is the company's product promise; the Terms of Use make the balance conditional and reserve the right to add expiry (§3.6). (2) *The characterisation trap:* phrases such as "the miles version of ShopBack" or "leading platform" are not the company's neutral self-description or the guide's own finding — they are third-party framing or the company's own boilerplate, quoted with attribution and never adopted (§11). (3) *The cheerleader trap:* no ranking, no market-share claim, no consumer advice, no value-per-mile recommendation appears in this guide; comparisons are architectural (§7, §13).

**Table of Contents**

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Decoder: Eight Terms, Plainly Defined](#12-the-decoder-eight-terms-plainly-defined)
   - 1.3 [The Key-Facts Table](#13-the-key-facts-table)
   - 1.4 [Why a Bank Should Care: The Cymbal Bank Lens](#14-why-a-bank-should-care-the-cymbal-bank-lens)
   - 1.5 [The Thesis](#15-the-thesis)
   - 1.6 [The Boundary Against the Singapore SaaS, Payments and Card Guides](#16-the-boundary-against-the-singapore-saas-payments-and-card-guides)
   - 1.7 [The Claims-Audit Map](#17-the-claims-audit-map)
2. [The Company: Entity, Founders and Funding](#2-the-company-entity-founders-and-funding)
   - 2.1 [The Legal Entity and the Singapore Base](#21-the-legal-entity-and-the-singapore-base)
   - 2.2 [The Founding Record: 2023 and the Four Ex-Meta Engineers](#22-the-founding-record-2023-and-the-four-ex-meta-engineers)
   - 2.3 [The Founder Names: First-Party Versus Attributed](#23-the-founder-names-first-party-versus-attributed)
   - 2.4 [The Stated Mission, Quoted](#24-the-stated-mission-quoted)
   - 2.5 [The Funding Rounds: Seed (2 July 2024) and Series A (28 January 2026)](#25-the-funding-rounds-seed-2-july-2024-and-series-a-28-january-2026)
   - 2.6 [The Company-Reported Metrics, and Why They Are Company-Reported](#26-the-company-reported-metrics-and-why-they-are-company-reported)
3. [The Product: Max Miles](#3-the-product-max-miles)
   - 3.1 [What Max Miles Is, in the Company's Own Words](#31-what-max-miles-is-in-the-companys-own-words)
   - 3.2 [How Earning Works: The Four Published Routes](#32-how-earning-works-the-four-published-routes)
   - 3.3 [How Redemption Works: Transfers, FlyAnywhere, Miles + Cash, Vouchers](#33-how-redemption-works-transfers-flyanywhere-miles--cash-vouchers)
   - 3.4 [The Nominated Airline and Hotel Partners](#34-the-nominated-airline-and-hotel-partners)
   - 3.5 [The Value Question, Left Open](#35-the-value-question-left-open)
   - 3.6 [The Expiry Finding: The Promise Versus the Terms](#36-the-expiry-finding-the-promise-versus-the-terms)
   - 3.7 [The Nature of Max Miles: Not Money, Not Held on Trust](#37-the-nature-of-max-miles-not-money-not-held-on-trust)
4. [The Card-Linking Mechanism](#4-the-card-linking-mechanism)
   - 4.1 [What Card Maximiser Is](#41-what-card-maximiser-is)
   - 4.2 [How a Card Is Linked: The Visa-Hosted Page and Visa On Platform](#42-how-a-card-is-linked-the-visa-hosted-page-and-visa-on-platform)
   - 4.3 [The Data Flow, Described Mechanically](#43-the-data-flow-described-mechanically)
   - 4.4 [The Crucial Disclaimer: We Track the Issuer's Rewards, Not Our Own](#44-the-crucial-disclaimer-we-track-the-issuers-rewards-not-our-own)
   - 4.5 [What the Visa Partnership Does and Does Not Establish](#45-what-the-visa-partnership-does-and-does-not-establish)
   - 4.6 [Card-Linked Max Miles Are Campaigns, Not a Standing Rate](#46-card-linked-max-miles-are-campaigns-not-a-standing-rate)
5. [The Business Model](#5-the-business-model)
   - 5.1 [The Published Model: Merchant- and Partner-Funded](#51-the-published-model-merchant--and-partner-funded)
   - 5.2 [The Four Funding Channels, Structurally](#52-the-four-funding-channels-structurally)
   - 5.3 [The Guiding Rule, Quoted: "If You Don't Earn, We Don't Earn"](#53-the-guiding-rule-quoted-if-you-dont-earn-we-dont-earn)
   - 5.4 [What Is Not Published: Take Rate and Unit Economics](#54-what-is-not-published-take-rate-and-unit-economics)
   - 5.5 [Why This Is Not ShopBack's Model](#55-why-this-is-not-shopbacks-model)
6. [The Consumer Proposition](#6-the-consumer-proposition)
   - 6.1 [Why a User Holds Max Miles Alongside the Card's Own Rewards](#61-why-a-user-holds-max-miles-alongside-the-cards-own-rewards)
   - 6.2 [Miles Versus Cashback: The Structural Difference](#62-miles-versus-cashback-the-structural-difference)
   - 6.3 [The Double-Dip: A Third-Party Framing](#63-the-double-dip-a-third-party-framing)
   - 6.4 [What This Guide Cannot Establish About Relative Value](#64-what-this-guide-cannot-establish-about-relative-value)
7. [The Competitive Frame](#7-the-competitive-frame)
   - 7.1 [The Five Structural Neighbours](#71-the-five-structural-neighbours)
   - 7.2 [The Aggregator's Place Among Them](#72-the-aggregators-place-among-them)
   - 7.3 [Why This Is Unranked](#73-why-this-is-unranked)
8. [The Technology and Data Position](#8-the-technology-and-data-position)
   - 8.1 [What the Platform Necessarily Sees](#81-what-the-platform-necessarily-sees)
   - 8.2 [What the Company Publishes About Its Stack](#82-what-the-company-publishes-about-its-stack)
   - 8.3 [The Data Question as Architecture, Not Risk](#83-the-data-question-as-architecture-not-risk)
9. [The AI Positioning](#9-the-ai-positioning)
   - 9.1 [The Vision Line, Quoted and Attributed](#91-the-vision-line-quoted-and-attributed)
   - 9.2 [The Concrete Artefact: Max AI / Card Spend AI](#92-the-concrete-artefact-max-ai--card-spend-ai)
   - 9.3 [The Company's Own Caveats](#93-the-companys-own-caveats)
   - 9.4 [Assessing the Claim Against the Artefact](#94-assessing-the-claim-against-the-artefact)
10. [The Card-Issuer and Network Relationship](#10-the-card-issuer-and-network-relationship)
    - 10.1 [Who Provides the Cards, Who Links Them](#101-who-provides-the-cards-who-links-them)
    - 10.2 [What the Aggregator Sees, What the Issuer Sees](#102-what-the-aggregator-sees-what-the-issuer-sees)
    - 10.3 [Where the Interests Converge](#103-where-the-interests-converge)
    - 10.4 [Where the Interests Diverge](#104-where-the-interests-diverge)
    - 10.5 [The Visa Network Relationship, Same Discipline](#105-the-visa-network-relationship-same-discipline)
    - 10.6 [The Arrangement in One Table](#106-the-arrangement-in-one-table)
11. [The Evidence Reality](#11-the-evidence-reality)
    - 11.1 [The Three Kinds of Evidence](#111-the-three-kinds-of-evidence)
    - 11.2 [How to Weigh Them](#112-how-to-weigh-them)
    - 11.3 [The Characterisations That Must Stay Attributed](#113-the-characterisations-that-must-stay-attributed)
12. [The Cymbal Bank Worked Example: A Third-Party Aggregator View](#12-the-cymbal-bank-worked-example-a-third-party-aggregator-view)
    - 12.1 [The Design-Fiction Frame](#121-the-design-fiction-frame)
    - 12.2 [The Scenario: A Bank Watches Its Cardholders' Reward Value Aggregate](#122-the-scenario-a-bank-watches-its-cardholders-reward-value-aggregate)
    - 12.3 [Working Through Section 5: The Loyalty Economics](#123-working-through-section-5-the-loyalty-economics)
    - 12.4 [Working Through Section 6: The Customer Relationship](#124-working-through-section-6-the-customer-relationship)
    - 12.5 [Working Through Section 10: The Cooperative-Competitive Position](#125-working-through-section-10-the-cooperative-competitive-position)
    - 12.6 [The Flow in Sequence](#126-the-flow-in-sequence)
    - 12.7 [Ending on the Thesis](#127-ending-on-the-thesis)
13. [The Anti-Patterns and Open Questions](#13-the-anti-patterns-and-open-questions)
    - 13.1 [The Anti-Patterns: Symptom, Cause and Guardrail](#131-the-anti-patterns-symptom-cause-and-guardrail)
    - 13.2 [The Open Questions](#132-the-open-questions)
14. [The Claims Audit](#14-the-claims-audit)
    - 14.1 [The Verified-Facts Table](#141-the-verified-facts-table)
    - 14.2 [The Flagged Claims Table](#142-the-flagged-claims-table)
    - 14.3 [The Rejected and Not-Found Claims](#143-the-rejected-and-not-found-claims)
    - 14.4 [What Could Not Be Verified](#144-what-could-not-be-verified)
15. [What Could Not Be Verified: The Honest Ledger](#15-what-could-not-be-verified-the-honest-ledger)
    - 15.1 [The Ledger](#151-the-ledger)
    - 15.2 [What the Absences Mean for a Reader](#152-what-the-absences-mean-for-a-reader)
16. [Glossary, Cross-References and the Closing Summary](#16-glossary-cross-references-and-the-closing-summary)
    - 16.1 [The Glossary](#161-the-glossary)
    - 16.2 [The Cross-Reference Map](#162-the-cross-reference-map)
    - 16.3 [Primary Sources Used This Pass](#163-primary-sources-used-this-pass)
    - 16.4 [The Closing Summary](#164-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

HeyMax is a **Singapore consumer rewards platform** that issues a proprietary points currency, **Max Miles**, which a user earns on ordinary spending — buying through brand links, buying prepaid vouchers, and spending on a linked Visa card — and redeems by transferring 1:1 into third-party airline and hotel programmes, booking flights through its own FlyAnywhere feature, topping up with cash, or buying retail vouchers (✅ heymax.ai/company/about and hk.heymax.ai/maxmiles; ✅ blog.heymax.ai Series A announcement, 28 January 2026). Its legal operator is **Max Now Pte. Ltd.**, trading as HeyMax (✅ heymax.ai/company/terms, opened "Max Now Pte. Ltd. (HeyMax)", last updated 6 February 2025). It was **founded in 2023** and describes itself as founded **by four former Meta engineers** (✅ blog.heymax.ai, seed announcement 2 July 2024 and Series A announcement 28 January 2026).

It is not a bank, not a card issuer, not a payment network, and not an airline loyalty programme. It is an **aggregator**: it holds a currency of its own, but the value behind that currency is produced by other people's businesses — merchants who fund a referral fee, partners who fund a campaign, and card issuers whose reward value HeyMax merely *observes and surfaces* when a card is linked. That structure is the single most important thing to understand about the firm, and it is the subject of §4, §5 and §10.

The honest headline for a reader arriving from the banking side: **HeyMax is the clearest Singapore example of a third party building a consumer relationship on top of card issuers' reward value — cooperatively, with the issuers' transaction rails — while the loyalty economics that produce that value remain the issuers' own.** The company itself states the crux plainly: the points and miles it tracks in its Card Maximiser feature "are **not Max Miles, but rewards from the card issuer / bank**. It's purely for informational purpose as **HeyMax is not the one issuing the rewards**" (✅ help.heymax.ai "Card Maximiser (Singapore users)", updated 27 February 2026). Read that sentence twice; everything in §10 and §12 follows from it.

### 1.2 The Decoder: Eight Terms, Plainly Defined

Because this space mixes consumer, issuer and platform vocabulary, the eight load-bearing terms are defined here once, so that later sections can use them without re-explanation. These are domain-stable definitions, unmarked.

- **Loyalty platform.** A system that issues or administers a reward currency and the rules for earning and spending it. The currency may be the platform's own (as with Max Miles) or a programme it administers on another firm's behalf.
- **Rewards aggregator.** A platform whose *product* is the union of several other firms' reward programmes and merchant offers, presented through one currency or one account. The aggregator typically does not manufacture the reward value; it sources, combines and redistributes it. HeyMax is one.
- **Universal currency.** A points currency that can be converted into, or spent across, several *unrelated* redemption partners — here, multiple airline and hotel programmes plus flights and vouchers — as opposed to a currency locked inside one brand's ecosystem (§3.1).
- **Merchant network.** The set of brands and merchants through which the aggregator's users can earn. HeyMax reports earning is available across a stated and dated number of "participating brands and merchants" (a company claim, §2.6).
- **Earning.** The accumulation of Max Miles by a user, through one of the four published routes (§3.2).
- **Redemption.** The spending of accumulated Max Miles, through one of the four published routes (§3.3).
- **Card linking.** The linking of a payment card to the platform so that the platform receives a feed of that card's transactions and can track both spending and the issuer's own reward accrual (§4). HeyMax's version is Visa-only and is called Card Maximiser.
- **Cashback versus miles.** A **cashback** reward is money-like value returned against spend — spendable as, or convertible to, cash, with a roughly fixed, one-dimensional value. A **miles** reward is a points currency spent inside redemption programmes whose "value" depends on how and where it is redeemed — a factor that ranges widely by redemption and is not fixed by the issuer. The distinction matters because a user's reward from HeyMax is a miles-type currency (Max Miles), and the platform's whole proposition rests on the flexibility and eventual redemption of that currency, not on a cash return (§6.2).

### 1.3 The Key-Facts Table

| Fact | Value | Mark |
|---|---|---|
| What it is | A Singapore consumer rewards / loyalty aggregator issuing the currency Max Miles; earning via brand links, vouchers, linked Visa cards and partner actions; redemption via 1:1 transfers, FlyAnywhere, Miles + Cash and vouchers | ✅ heymax.ai/company/about; hk.heymax.ai/maxmiles; blog.heymax.ai (2 Jul 2024, 28 Jan 2026) |
| Legal operator | **Max Now Pte. Ltd.** (trading as HeyMax) | ✅ heymax.ai/company/terms (6 Feb 2025); hk.heymax.ai footer "©2025 MAX NOW PTE LTD" |
| Founded | **2023** | ✅ blog.heymax.ai Series A announcement (28 Jan 2026) |
| Founders | "Four former Meta engineers" (company); CEO & co-founder **Joe Lu** (first-party); other names **Jialu Zhong, Ke Wang (CTO), Sean Dy (COO)** — third-party sources | ✅ company (count, and Joe Lu); ⚠ third-party/registry (other names) |
| Headquarters | Singapore | ✅ blog.heymax.ai (both announcements) |
| First international market | Hong Kong, 2025 (via acquisition of HK fintech krip) | ✅ blog.heymax.ai Series A (28 Jan 2026); fintechnews.sg (28 Jan 2026) |
| Stated expansion | Japan, Taiwan, Australia targeted by end-2026 (a plan, not a result) | ✅ blog.heymax.ai Series A (28 Jan 2026) |
| Seed round | **US$2.6M (SG$3.5M), 2 July 2024**, led by January Capital; Tenity, Ascend Angels, XA Network + angels | ✅ blog.heymax.ai (2 Jul 2024); fintechnews.sg (2 Jul 2024) |
| Series A | **US$11M, 28 January 2026**, led by Peak XV Partners; Betatron; continued January Capital, Tenity; strategic investors Rob Rosenstein, David Lee | ✅ blog.heymax.ai (28 Jan 2026); fintechnews.sg (28 Jan 2026) |
| Funding beyond Series A | **None published** | ✅ absent (recorded as the finding) |
| Valuation | **Not published** | ✅ absent (recorded as the finding) |
| The currency | Max Miles — company definition "a universal travel currency you earn from your everyday spend, that never expires" | ✅ company product claim (hk.heymax.ai/maxmiles, ©2025) |
| Expiry — terms reality | 12-month inactivity forfeiture (Terms §4); reserved right to "add or change the duration taken for Max Miles to expire" (Terms §7) | ✅ heymax.ai/company/terms (6 Feb 2025) |
| Card linking | Visa cards only, via a Visa-hosted webpage; tracks the issuer's rewards, not Max Miles | ✅ help.heymax.ai "Card Maximiser (Singapore users)" (27 Feb 2026) |
| Business model | Merchant/partner-funded; company rule "if you don't earn, we don't earn" | ✅ blog.heymax.ai "How we get you your Max Miles" (28 May 2025) |
| Take rate / unit economics | **Not published** | ✅ absent (recorded as the finding) |
| AI artefact | "Max AI" / Card Spend AI — a read-only, rate-limited in-app rewards assistant (6 Apr 2026); calculations from a calculation engine, "not AI estimates" | ✅ help.heymax.ai "Max AI" (6 Apr 2026) |

### 1.4 Why a Bank Should Care: The Cymbal Bank Lens

A retail bank's card franchise rests on three things an aggregator like HeyMax can touch without ever issuing a card: **the visibility of the cardholder's spend**, **the meaning the cardholder attaches to the card's reward**, and **the layer at which the customer relationship is experienced**. HeyMax sits on all three. It sees a stream of a linked card's transactions (§4), it computes and displays the issuer's reward accrual and cap progress (§4.4), and it inserts itself between the cardholder and the card as the place where "which card do I pull out now?" is answered (§9). None of this is hostile, and much of it is mutually beneficial — better reward visibility can increase the very engagement an issuer wants (§10.3). But it is a structural fact worth a bank's attention: the aggregator monetises attention and traffic at the rewards layer, in exchange for value it *sources from* the issuer's programme rather than creates (§5).

The Cymbal Bank worked example in §12 works this through, without reaching a verdict. It is clearly marked design fiction: Cymbal Bank is the repo's fictional Singapore bank persona and the only institution in the example.

### 1.5 The Thesis

**An aggregator's proposition is convenience at the seam between many loyalty programmes, and its cost is that it does not own the value it distributes.** HeyMax earns when a consumer's ordinary spending is routed, attributed and converted — a merchant fee here, a voucher margin there, a campaign funded by a partner — and it converts that value into a currency it controls (Max Miles) but a value it does not manufacture (the airline miles and hotel points it transfers into are other firms' products). Every strength and every fragility of the model follows from that one line: flexibility is real, because a union of programmes is broader than any single one; durability is borrowed, because the union is only as good as its members' willingness to keep being aggregated. This guide holds that thesis open rather than resolving it, and §16 closes on the same note.

### 1.6 The Boundary Against the Singapore SaaS, Payments and Card Guides

This is a **company deep-dive**, not a sector or rail deep-dive. It therefore respects three boundaries. First, the Singapore SaaS sibling ([singapore_saas_companies_guide.md](../technology/singapore_saas_companies_guide.md)) already maps the local software sector, with HeyMax appearing there at most as a line item; this guide is the dedicated expansion of that line and does not re-derive the sector aggregate. Second, the payments and card clusters carry the mechanics of card schemes, interchange, issuer reward programmes and payment rails as such — this guide assumes those mechanics and treats them only as far as a rewards aggregator touches them, primarily in §4 and §10. Third, this guide does not compute, recommend or rank by value (§6.4, §7.3): those are consumer-advice territories the repo's genre avoids, and where a value figure exists at all it is attributed to the source that published it.

### 1.7 The Claims-Audit Map

Every load-bearing claim in this guide is consolidated in **§14 (The Claims Audit)**, organised into the verified-facts table (§14.1), the flagged claims table (§14.2), the rejected-or-not-found table (§14.3) and a **"What Could Not Be Verified"** note (§14.4). **§15 (What Could Not Be Verified)** is the fuller honest ledger for an early-stage private company — valuation, take rate, profitability, registration number, later rounds, reverse-redemption mechanics and the exact FlyAnywhere rate all appear there as recorded absences. A reader who wants only the audited skeleton can read §1, §3.6, §14 and §15 and be correctly calibrated.

---

## 2. The Company: Entity, Founders and Funding

### 2.1 The Legal Entity and the Singapore Base

The platform's legal operator is **Max Now Pte. Ltd.**, a Singapore private limited company trading under the brand HeyMax. The legal name is stated first-party, opened on the very first line of the Terms of Use: "**Max Now Pte. Ltd. (HeyMax)** offers, amongst other things, rewards in the form of Max Miles to HeyMax account holders…" (✅ heymax.ai/company/terms, last updated 6 February 2025). The Hong Kong market page's footer carries the same entity: "©2025 MAX NOW PTE LTD. ALL RIGHTS RESERVED." (✅ hk.heymax.ai/maxmiles). The company is **headquartered in Singapore**, described as a "Singapore-based" and "Singapore-founded" platform in both funding announcements (✅ blog.heymax.ai, 2 July 2024 and 28 January 2026).

The entity is a private limited company, which is the ordinary Singapore vehicle for a venture-funded startup: it is not listed, publishes no audited financials on its public pages, and its shareholding is not a matter of public record on the pages read this pass. The **ACRA registration number was not read this pass** and is recorded as not verified in §15 — not because it does not exist (every Singapore Pte. Ltd. has one) but because it was not captured at a primary source in this research pass, and this guide does not invent it.

### 2.2 The Founding Record: 2023 and the Four Ex-Meta Engineers

HeyMax was **founded in 2023** (✅ blog.heymax.ai Series A announcement, 28 January 2026: "Founded in 2023, HeyMax is a Singapore-based platform that accelerates and unifies loyalty and travel rewards…"). The founding **team description** is consistently first-party: both funding announcements say the company was "founded by four former Meta engineers" (✅ blog.heymax.ai, 2 July 2024; ✅ blog.heymax.ai, 28 January 2026). The seed announcement adds the operative detail — "founded in 2023 by four ex-Meta engineers" — and labels the early product a "personal finance and shopping platform that allows consumers to earn Max Miles from over 500 businesses" (✅ blog.heymax.ai, 2 July 2024).

Two cautions attach. First, "ex-Meta" describes the founders' *former* employers; it is not a claim about Meta's relationship to HeyMax, and nothing in the record suggests Meta invested in, endorsed or built the platform. Second, the **size of the founding team is first-party; the enumeration of its members is not** — see §2.3.

### 2.3 The Founder Names: First-Party Versus Attributed

Only one founder is named first-party. **Joe Lu** is named repeatedly and quoted, as "CEO and Co-founder of HeyMax.ai", in HeyMax's own announcements of both the seed and the Series A (✅ blog.heymax.ai, 2 July 2024; ✅ blog.heymax.ai, 28 January 2026; also quoted in the independent press, fintechnews.sg, both dates). Joe Lu is therefore the guide's first-party-confirmed founder.

The **other three names are not published on HeyMax's own pages** as read this pass. They appear on third-party and registry-derived sources, and must be attributed rather than adopted:

- **Crunchbase** (registry-derived company card) lists the founders as **Jialu Zhong, Joe Lu, Ke Wang, Sean Dy** (⚠ third-party/registry-derived, read this pass).
- **Preqin** states the company was "Co-founded by **Joe Lu (CEO), Jialu Zhong, Wang Ke (CTO), and Sean Dy (COO)**" (⚠ third-party database).
- **Tenity** (an accelerator and seed investor) names "**Joe Lu, Sean D, Jialu Zhong and Wang Ke**" (⚠ third-party, in an accelerator announcement/social post).

The three sources agree on the names (with "Ke Wang" and "Wang Ke" being the same person, and the CTO/COO roles given by Preqin), which is corroborating but **not first-party**. This guide therefore treats the complete founding roster as: Joe Lu (CEO) — first-party ✅; Jialu Zhong, Ke Wang (CTO), Sean Dy (COO) — attributed ⚠ to Crunchbase, Preqin and Tenity. Where this guide names the roster, it says which is which. A reader who needs the founders first-party should treat that as an open item (§15).

### 2.4 The Stated Mission, Quoted

HeyMax states a mission and a vision, and like every other company claim they are quoted here with attribution. The mission statement used in its own boilerplate: "HeyMax is on a mission to bring more joy and empathy to the world through travel" (✅ blog.heymax.ai Series A announcement, 28 January 2026). The vision, from the About page: "Our vision is to build the **default protocol for every business to engage with every consumer in the AI-Native Era**" (✅ heymax.ai/company/about; treated as a positioning claim in §9). The consumer-facing framing — "turning the things you already do into free trips, year after year" — is the company's own product promise and is quoted as such (✅ blog.heymax.ai, 28 January 2026). None of these are guide-level facts about outcomes; they are the company's stated intent.

### 2.5 The Funding Rounds: Seed (2 July 2024) and Series A (28 January 2026)

Two rounds are published, each with a date and a source. No later round is assumed, and the guide states the two-round limit explicitly.

**The seed.** On **2 July 2024**, HeyMax announced a **US$2.6 million (SG$3.5 million) seed round led by January Capital**, with participation from **Tenity, Ascend Angels, XA Network** and other strategic investors (✅ blog.heymax.ai, first-party announcement, 2 July 2024; corroborated ✅ fintechnews.sg, independent press, 2 July 2024). The seed announcement adds that Tenity had been involved since inception via its accelerator programme, and lists a long tail of named angel investors. *One discrepancy to note honestly:* the **later Series A announcement and its press coverage describe the seed as "US$2.7 million"** while the seed announcement itself says "US$2.6 million (SG$3.5 million)" (⚠ minor first-party inconsistency, recorded rather than resolved). The guide uses US$2.6M for the seed, its original stated figure, and flags the US$2.7M variant.

**The Series A.** On **28 January 2026**, HeyMax announced a **US$11 million Series A led by Peak XV Partners**, with **Betatron Venture Group** participating and continued support from existing backers **January Capital** and **Tenity**. Additional investors named were **Rob Rosenstein** (co-founder and chairman of Agoda) and **David Lee** (fintech advisor, independent bank director, and former President of Visa APAC) (✅ blog.heymax.ai, first-party announcement, 28 January 2026; corroborated ✅ fintechnews.sg, 28 January 2026, and the PR Newswire/company release of the same date). Stated use of funds: product development "focusing on AI-empowered rewards experience", and regional expansion. Stated expansion markets: beyond Singapore and Hong Kong into **Japan, Taiwan and Australia by the end of 2026** (✅ blog.heymax.ai, 28 January 2026) — a plan, dated, not a result.

Notably, one strategic investor is a former **Visa APAC president** (David Lee). That is a fact about an investor's background, not evidence of a Visa corporate investment, endorsement or exclusive arrangement with HeyMax; the guide is careful not to blur the two (§4.5, §10.5).

### 2.6 The Company-Reported Metrics, and Why They Are Company-Reported

HeyMax reports a set of growth metrics about itself. **Each is a company claim, attributable to the company with its date, and printed here only as such — never as the guide's own measurement.**

| Metric | As reported (company claim) | Date / source |
|---|---|---|
| Users | ">50,000" | ✅ blog.heymax.ai, seed announcement, 2 Jul 2024 |
| Users | ">150,000" | ✅ blog.heymax.ai, Series A announcement, 28 Jan 2026 |
| Max Miles issued | ">50 million … since Max Miles launched in September 2023" | ✅ blog.heymax.ai, 2 Jul 2024 |
| Max Miles issued | "more than 500 million … annually" | ✅ blog.heymax.ai, 28 Jan 2026 |
| Merchants | "over 500 businesses" | ✅ blog.heymax.ai, 2 Jul 2024 |
| Merchants | "over 800 participating brands and merchants" | ✅ blog.heymax.ai, 28 Jan 2026 |
| Airline/hotel partners | "25 airline and hotel partners" | ✅ blog.heymax.ai, 2 Jul 2024 |
| Airline/hotel partners | "more than 30 airline and hotel programmes" | ✅ blog.heymax.ai, 28 Jan 2026 |
| Revenue | "fivefold Y-o-Y revenue growth" and "annualized revenue run rate of US$6 million" | ⚠ company-reported, reported Jan 2026 (fintechnews.sg, 28 Jan 2026) |
| Forward target | "strong triple-digit annual GMV growth over the next two years" | ⚠ company target, not a result (blog.heymax.ai, 28 Jan 2026) |

The distinction the guide enforces throughout: these are **volume and growth claims made by the company about itself**, dated, and the guide neither endorses their accuracy nor disputes them. They are useful as the company's own stated scale, and for trend (the direction is consistently larger across the two dates), not as audited facts. The one figure in the neighbourhood that is *not* a company self-claim is Peak XV's investor thesis, quoted in the company's own release: "More than 40% of the total card revenues, totalling over $100B globally, is spent on loyalty and consumer rewards - this is HeyMax's opportunity" (✅ quoted in blog.heymax.ai, 28 January 2026 — an investor's framing, not an audited statistic, and attributed accordingly).

---

## 3. The Product: Max Miles

### 3.1 What Max Miles Is, in the Company's Own Words

Max Miles is the company's own reward currency, and the company defines it in a sentence it repeats with minor variation across pages: **"Max Miles is a universal travel currency you earn from your everyday spend, that never expires"** (✅ hk.heymax.ai/maxmiles, ©2025). The Hong Kong explainer page elaborates that Max Miles is "**Universal**" (redeemable "for flights, hotels, vouchers and other rewards across diverse brands and options"), "**Earned through everyday spend**", and "**Never expire**" (✅ hk.heymax.ai/maxmiles). The company's boilerplate adds that Max Miles "never expire, come with no fees, and offer unmatched flexibility for modern travelers" (✅ blog.heymax.ai Series A announcement, 28 January 2026).

Read carefully, these are **product claims** — how the company markets its currency — and they are marked and attributed as such. The phrase "never expires" in particular is the company's promise, contrasted with the terms in §3.6, and never stated as this guide's own fact. The words "universal" and "flexible" describe the currency's *redemption breadth* — the number of unrelated programmes it can become, which is genuinely broader than a single airline's currency — and do not by themselves establish a value.

### 3.2 How Earning Works: The Four Published Routes

HeyMax publishes exactly how its miles are funded and earned, in a deliberately transparent first-party explainer titled "How we get you your Max Miles" (✅ blog.heymax.ai, first-party article, 28 May 2025). The article's own summary: "We pass the value we receive back to you in the form of Max Miles. We earn only when your miles get to you, through four major ways: 1. Shop-through brand links 2. Vouchers for instant miles 3. Card-linked rewards 4. Special partner actions" (✅ 28 May 2025). The four routes, as published:

1. **Shop-through brand links (affiliate).** The user taps a "Shop with Max" button, which tells the brand the visit came from HeyMax; if the user buys, "the merchant sends us a small thank-you fee", which HeyMax converts into Max Miles credited on confirmed purchase. Crucially: "your checkout price never changes" (✅ blog.heymax.ai, 28 May 2025). Most merchants confirm within days; "a handful take up to 90 to 120 days", and if attribution fails (another site credited, or an ad-blocker hid the click) the fee never arrives and no miles are earned — the company states it fights for missing miles on the user's behalf.
2. **Vouchers for instant miles.** The app sells prepaid gift cards and vouchers HeyMax has already funded at bulk prices; the user "still pay[s] face value while seeing exactly how many miles you'll receive", and the margin is "baked in and displayed" — so the user pays face value and gets miles instantly (✅ 28 May 2025).
3. **Card-linked rewards.** Linking a Visa card means that when the user taps it, "your spend gets tracked within minutes and we drop the corresponding Max Miles into your account automatically" (✅ 28 May 2025). The article carries the key asterisk: "**Visa card-linked rewards run as limited-time campaigns**" — i.e., card-linked *Max Miles* are promotional and temporary, not a standing rate (§4.6).
4. **Special partner actions (CPA-style).** A partner funds a "juicy chunk of Max Miles" to nudge a specific completed action — "Apply for Card X" or "Book dinner at Restaurant Y" — credited once the partner (e.g. SingSaver, Chope) confirms the action. "If the action isn't verified, we earn nothing either" (✅ 28 May 2025).

The article is candid about mechanics too: earn rates are "dynamic and will be optimised over time"; promotions can temporarily lift a brand's rate (its example: "8 Max Miles per dollar for a fortnight and then settle back to its regular 4"); and posting speed "depends on the bank that issued your card" (some share nightly; UOB bundles and sends later) (✅ 28 May 2025). All of this is the company's own published account of its own product — first-party, dated, and authoritative on mechanics, but still the company describing itself.

### 3.3 How Redemption Works: Transfers, FlyAnywhere, Miles + Cash, Vouchers

Redemption has four published routes, presented on the Max Miles explainer page as a numbered list of ways the user can "use Max Miles" (✅ hk.heymax.ai/maxmiles):

1. **Transfer 1:1 to airline and hotel partners.** "Max Miles transfers directly into 30+ airline and hotel loyalty programs points" at a 1:1 ratio (✅ hk.heymax.ai/maxmiles; ✅ blog.heymax.ai, 28 January 2026, "transfer them to more than 30 airline and hotel programmes"). The 1:1 framing is the company's and is dated.
2. **FlyAnywhere.** The company's own redemption feature that "enables booking on nearly any airline at a fixed rate per mile" (✅ blog.heymax.ai, 28 January 2026) — "Zero blackout dates, fly as you wish" (✅ hk.heymax.ai/maxmiles). Per the company, the user books a flight and Max Miles are applied against it at a fixed rate (§3.5 on the rate itself).
3. **Miles + Cash.** "Not enough miles? … Use a combination of Max Miles and cash to unlock trips sooner" (✅ hk.heymax.ai/maxmiles) — a top-up mechanism so a partial balance is still usable.
4. **Redeem for vouchers.** "Redeem cash vouchers for your favorite retail brands using Max Miles" (✅ hk.heymax.ai/maxmiles) — i.e., miles can be spent on retail vouchers rather than travel, part of the "universal" claim.

The `1:1` transfer ratio and the partner count are company-published and dated; the *availability* of each redemption option is explicitly subject to change (the Terms: "The availability of redemption options may change over time, and HeyMax reserves the right to modify these options", ✅ heymax.ai/company/terms §5).

### 3.4 The Nominated Airline and Hotel Partners

The company and its press name a set of transfer partners, useful as illustrative breadth rather than as an exhaustive or stable list: "**Cathay, ALL Accor, and Qatar Airways**" appear in the Series A material, and the consumer blog Lobang Sis adds "**EVA Air, Qatar Airways, Japan Airlines and World of Hyatt**" among "more than 30" programmes (✅ blog.heymax.ai, 28 January 2026; ⚠ Lobang Sis, consumer blog, 3 June 2026). The seed-era material named "25 airline and hotel partners" (✅ blog.heymax.ai, 2 July 2024). Because partner lists change, this guide treats all such names as **illustrative of the redemption surface, dated, and subject to change** — it does not present them as a guaranteed catalogue. The important structural point is not which programmes are on the list today, but that the list is composed entirely of **other companies' loyalty programmes** — the aggregation thesis in one sentence.

### 3.5 The Value Question, Left Open

How much is a Max Mile worth? This is exactly the question a consumer wants answered and exactly the question this guide **declines to answer as advice** (§6.4). Two things can be said factually without crossing into advice. First, the company publishes a **"fixed rate per mile"** for FlyAnywhere (✅ blog.heymax.ai, 28 January 2026) but does **not publish the numeric rate** on its own pages as read this pass; the only specific figure encountered was **1.8 cents per mile**, published by the consumer blog **Lobang Sis** (⚠ third-party, 3 June 2026), and it is recorded here attributed to that source, not adopted as first-party. Second, the company's own FlyAnywhere examples (e.g. a Hong Kong–Taipei one-way economy fare of "~HK$900" costing "~7,000" Max Miles) are illustrative redemption illustrations, dated and company-published, not a valuation (✅ hk.heymax.ai/maxmiles). Converting those illustrations into an implied per-mile figure would itself be a value computation, so this guide does not do it; it prints the company's inputs and stops.

### 3.6 The Expiry Finding: The Promise Versus the Terms

This is the single most important factual comparison in the guide, and it must be handled precisely, because the promise and the terms say different things and both are first-party.

**The promise (marketing):** "Max Miles is a universal travel currency you earn from your everyday spend, **that never expires**" (✅ hk.heymax.ai/maxmiles, ©2025). The company's boilerplate repeats it: "Max Miles never expire" (✅ blog.heymax.ai, 28 January 2026).

**The terms (the contract):** three first-party provisions qualify the promise.

- **Terms §4 (Top-ups, Deductions and Account Inactivity):** "HeyMax reserves the right to remove a user (including the full balance in a user's account) where HeyMax determines that a user has **not used the Services for any consecutive period of 12 months or longer**" (✅ heymax.ai/company/terms, last updated 6 February 2025). A 12-month inactivity can therefore, at the company's election, forfeit the entire balance.
- **Terms §7 (Changes to Max Miles Programme):** HeyMax reserves the right to "**add or change the duration taken for Max Miles to expire**" (✅ 6 February 2025). Put plainly: the terms contemplate that expiry may be *introduced*; the "never expires" promise is not a contractual guarantee against future expiry.
- **Help centre (campaign terms):** "Max Miles do not expire, **as long as your HeyMax account stays active**. An active account is one you log in to at least once every 12 months" (✅ help.heymax.ai "Max Miles Turns 3!", updated 22 September 2026). This is the company's own reconciliation of the promise with the §4 inactivity rule: "never expires" *means* "never expires while your account is active", and "active" means logged in within the last 12 months.

**The finding, stated neutrally:** "never expires" is the company's product/marketing promise; the Terms of Use make the balance **conditional on account activity** (a 12-month inactivity forfeiture right under §4) and expressly **reserve the right to introduce a duration after which Max Miles expire** (§7). This is a factual comparison of a promise against the terms that govern it. It is **not** a statement by this guide that Max Miles expire, and it is **not** consumer advice; it is the observation that the word "never" is qualified by the contract the user accepts. A reader weighing the product should treat the inactivity rule and the reserved expiry right as the operative conditions and the marketing line as the headline.

### 3.7 The Nature of Max Miles: Not Money, Not Held on Trust

What *is* a Max Mile, legally? The Terms answer in §6 (Nature of Max Miles): "you do not gain any **proprietary right** over any monies or assets held by HeyMax when you earn Max Miles… The Max Miles accrued in your account **does not constitute monies held on trust** by HeyMax for your benefit. Your rights and entitlements are solely limited to such **personal or contractual rights of repayment** as may arise out of this Agreement" (✅ heymax.ai/company/terms, 6 February 2025). Two consequences follow, and both are standard for loyalty currencies rather than peculiar to HeyMax. First, Max Miles are a **contractual reward entitlement**, not a stored-value balance or e-money; the user is an unsecured contractual counterparty of the company for their miles, not a beneficiary of a ring-fenced pool. Second, the treatment reinforces the decoder's distinction in §1.2: Max Miles behave like a loyalty programme's points — governed by programme terms the company may vary (§7) — not like a bank balance. This is the precise legal texture behind the promise-versus-terms finding in §3.6.

---

## 4. The Card-Linking Mechanism

### 4.1 What Card Maximiser Is

**Card Maximiser** is HeyMax's card-linking feature. The company's own help centre defines it: "Card Maximiser is a feature in the HeyMax app that automatically tracks your linked credit cards' rewards, including how much you've spent, how many points/miles you've earned, and how much of your bonus caps or minimum spends you've reached — all in one place" (✅ help.heymax.ai "Card Maximiser (Singapore users)", updated 27 February 2026). Note carefully what that sentence says the feature tracks: **the linked card's rewards** — the issuer's points or miles — not Max Miles (§4.4). There are, in the app, two distinct things: *adding* a card (which just tells HeyMax what the user holds, enabling "best card" recommendations) and *linking* a card (Visa-only, which establishes the live transaction feed) (✅ help.heymax.ai "Adding Cards Basics", 20 June 2024).

### 4.2 How a Card Is Linked: The Visa-Hosted Page and Visa On Platform

The linking flow is stated plainly by the company: "We've partnered with **Visa**… You can securely link your credit card via a **Visa-hosted webpage**. With your authorisation, Visa sends us a **stream of your transactions** that happen on your linked card thereafter. This does not give HeyMax the ability to access your card number nor charge your card" (✅ help.heymax.ai "Card Maximiser (Singapore users)", 27 February 2026). The user opts in to *link* (as opposed to merely add) a Visa card; the enrolment and consent occur on a Visa-hosted page, not inside HeyMax's own UI; and the output of that enrolment is a transaction feed (✅ same page). The specific enrolment channel the company names elsewhere is **Visa On Platform (VOP)** — the Max AI help article describes transaction tracking as working "with Visa cards enrolled through **Visa On Platform (VOP)**" (✅ help.heymax.ai "Max AI", 6 April 2026). So the mechanism, mechanically: *user → Visa-hosted linking page → Visa On Platform enrolment → Visa transaction feed to HeyMax → HeyMax's engine computes issuer-reward accrual and displays it.*

### 4.3 The Data Flow, Described Mechanically

Neutrally and mechanically, the data flow is:

1. **Consent and enrolment.** The user, on a Visa-hosted page, authorises the linking of a specific Visa card and consents to the transaction feed (✅ §4.2).
2. **Feed.** Visa sends HeyMax "a stream of your transactions that happen on your linked card thereafter" (✅ help.heymax.ai, 27 February 2026). The company says most transactions track within minutes and flags the edge case where a merchant's acquiring bank equals the card issuer, so the transaction "doesn't pass through Visa's network" and can take up to two weeks (✅ 27 February 2026).
3. **What HeyMax receives and stores.** "We only collect your **transaction history and the last 4 digits of your card number** for identification purposes" (✅ help.heymax.ai, 27 February 2026). It states it does **not** gain access to the full card number and **cannot** charge the card (✅ 27 February 2026).
4. **What it computes.** Using the linked spend, its engine derives "how many points/miles you've earned, and how much of your bonus caps or minimum spends you've reached" for the supported cards (✅ 27 February 2026) — i.e. it models the *issuer's* programme rules against the *user's* spend.
5. **Deletion.** "When you unlink your card from heymax at anytime, your data is deleted forever" (✅ help.heymax.ai "Card Maximiser: Security and Privacy", 20 June 2024).

That flow is the whole of the card-linking architecture as published, and it is the substrate for both the product's convenience (§6) and the issuer-relationship questions (§10).

### 4.4 The Crucial Disclaimer: We Track the Issuer's Rewards, Not Our Own

The load-bearing sentence of the entire guide appears here, first-party and unambiguous: "**The points/miles that we track here are not Max Miles, but rewards from the card issuer / bank. It's purely for informational purpose as HeyMax is not the one issuing the rewards**" (✅ help.heymax.ai "Card Maximiser (Singapore users)", 27 February 2026). Three things follow, and each is central to §10:

- What Card Maximiser *displays* when a card is linked is a modeling of **the bank's own loyalty programme** (the issuer's points/miles, caps, minimum-spend progress) — value created by the issuer's programme rules, computed by HeyMax's engine from the transaction feed.
- Max Miles — HeyMax's own currency — are a **separate** thing, earned through card-linked *campaigns* (§4.6) and the other routes (§3.2), not through the standing reward tracking.
- The company explicitly disclaims being the issuer of the tracked rewards: "HeyMax is not the one issuing the rewards" (✅ 27 February 2026). So the aggregator **surfaces** the issuer's reward value; it does not manufacture it.

This is why the reward-tracking coverage is limited: transaction tracking works on "almost all" Singapore-issued Visa cards, but *reward* tracking (points/miles) is supported only on a named set of top miles cards — Citi PremierMiles and Citi Rewards; DBS Altitude, Vantage and yuu; HSBC Revolution; Maybank eVibes and Horizon; UOB Visa Signature and Preferred Platinum; OCBC Voyage and Premier Voyage; Standard Chartered Journey and Smart; and Chocolate Visa (✅ help.heymax.ai, 27 February 2026). Other Visa cards can be linked for spend tracking but their reward accrual is not yet modeled (✅ same page). Mastercard and American Express cards cannot be linked at all at this date (✅ same page).

### 4.5 What the Visa Partnership Does and Does Not Establish

**What it establishes.** Visa provides the **data rail**: a user-authorised linking page and a transaction feed (via Visa On Platform) that HeyMax consumes to power Card Maximiser (✅ help.heymax.ai, 27 February 2026; ✅ "Max AI", 6 April 2026). That is a real, named, first-party-disclosed integration between HeyMax and Visa, and it is the mechanism behind every "linked card" feature in the product. The seed announcement describes the same partnership in its own words: "a recent collaboration with Visa introduced the Card Maximiser, allowing consumers to track spending across all Visa-branded cards" (✅ blog.heymax.ai, 2 July 2024; ✅ fintechnews.sg, 2 July 2024).

**What it does NOT establish.** The published record does **not** show, and this guide does **not** assert: a Visa endorsement of Max Miles as a currency; any statement that Visa issues, backs or guarantees Max Miles; any exclusivity (nothing says HeyMax may not integrate other networks — indeed the company says it *plans* to support more card networks in future, ✅ help.heymax.ai, 27 February 2026); or any commercial terms (fee, revenue share, term) between HeyMax and Visa. Nor does it follow that a former **Visa APAC president** appearing as a strategic *investor* (§2.5) is a Visa corporate arrangement — the two are distinct facts. Each of these is recorded as **not established** (⚠), and restated in §15. The disciplined summary: the Visa relationship is a **card-linking data integration**, demonstrated by the product; it is not, on the public record, an endorsement, a currency-backing, an exclusivity or a disclosed commercial relationship beyond the integration itself.

### 4.6 Card-Linked Max Miles Are Campaigns, Not a Standing Rate

A subtle but important distinction, stated by the company itself: when a linked Visa card earns **Max Miles** (as opposed to tracking the issuer's rewards), that earning runs as a **limited-time campaign**. The "How we get you your Max Miles" article carries the asterisk "**Visa card-linked rewards run as limited-time campaigns**" (✅ blog.heymax.ai, 28 May 2025), and the help centre documents concrete instances: the **Chocolate Finance Visa debit card** campaign (2 Max Miles per dollar up to a monthly cap, blog 2025) and the **Trust Freedom Credit Card** "Max Miles Turns 3!" campaign (21–30 September 2026, cap 3,003 rewards of 333 Max Miles each) (✅ help.heymax.ai, 22 September 2026). A third-party example is MileLion's coverage of an August 2024 Visa campaign offering "3 mpd on bus/MRT rides" (⚠ MileLion, consumer miles blog, 29 August 2024). The structural point: **card-linked Max Miles are promotional, partner-funded and time-boxed**, which is why they are not a dependable standing earn rate and why they belong with "campaign funding" (not "standing reward") in the business model (§5.2).

---

## 5. The Business Model

### 5.1 The Published Model: Merchant- and Partner-Funded

HeyMax publishes its business model openly, in the same explainer that documents earning: "We pass the value we receive back to you in the form of Max Miles. **We earn only when your miles get to you**, through four major ways…" (✅ blog.heymax.ai "How we get you your Max Miles", 28 May 2025). The model is therefore **merchant- and partner-funded**: the value that becomes Max Miles is paid to HeyMax by merchants, partners and campaign sponsors — not by the consumer. The consumer's "checkout price never changes" (affiliate route), vouchers are sold at face value with a pre-funded margin, and partner actions are funded by the partner (✅ 28 May 2025, §3.2). This is a **published structural description** — the company states the *sources* of its revenue plainly — and it is the honest floor on which the rest of this section rests. It is *not* a full financial disclosure; the company does not publish rates, margins or margins-of-margins, which is the subject of §5.4.

### 5.2 The Four Funding Channels, Structurally

Mapping the four earning routes (§3.2) onto their funding sources, the revenue structure is:

| Channel | Who pays HeyMax | How it reaches the consumer |
|---|---|---|
| Shop-through brand links | The merchant, a referral/affiliate (CPA-style) fee | Converted to Max Miles, credited after the purchase confirms |
| Vouchers | The margin on pre-funded bulk voucher purchases | Miles credited instantly; user pays face value, margin shown |
| Card-linked rewards | The campaign sponsor (merchant/issuer/partner) funding the promotion | Max Miles awarded under a limited-time campaign cap |
| Special partner actions | The partner wanting a completed action (CPA) | Miles credited once the partner confirms the action |

(All four rows: ✅ blog.heymax.ai, 28 May 2025.) Underneath, the same structural shape recurs: HeyMax aggregates a **fee or margin funded by a third party**, retains a spread, and converts the rest into Max Miles for the consumer. The currency is the redistribution layer; the fee is the revenue.

### 5.3 The Guiding Rule, Quoted: "If You Don't Earn, We Don't Earn"

The company states its alignment rule explicitly and in bold: "**If you don't earn, we don't earn**" (✅ blog.heymax.ai, 28 May 2025). The article frames it as "the easiest guardrail against misaligned incentives" and pairs it with three claims: "There are no hidden fees, no mark-ups on flights, no expiring miles" (✅ 28 May 2025). Two of those three are structural and checkable (no hidden fees visible to the consumer; no flight mark-up, because the flight routes are affiliate/campaign-funded). The third — "no expiring miles" — is the marketing promise examined in §3.6, whose reconciliation with the inactivity rule and the reserved expiry right is the substance of the expiry finding. The rule itself is a genuine structural feature — the consumer earns only when a funding event occurs — and it is quoted, not adopted.

### 5.4 What Is Not Published: Take Rate and Unit Economics

What the company does **not** publish is the size of the spread: the **take rate** (the proportion of each merchant fee, voucher margin or campaign budget HeyMax retains), the **cost structure**, the **contribution margin** per user, or **revenue by channel**. The take rate is recorded as **not published** (✅ absent) and is restated in §15. This is the correct disposition for an early-stage private company: the *structure* of the model is public (§5.1–§5.3), the *magnitudes* are not, and the guide does not manufacture them. In particular, the guide does **not** infer HeyMax's economics from any comparable company's model (§5.5).

### 5.5 Why This Is Not ShopBack's Model

It is tempting to assume HeyMax's economics mirror those of a cashback platform such as ShopBack, because both are Singapore rewards aggregators earning merchant fees. **This guide does not make that inference.** The two differ in a way that matters to the economics even before any figure is known: a cashback platform pays the consumer **cash** (money-like value with a mostly fixed, one-dimensional worth, funded by the merchant fee net of the platform's spread), whereas HeyMax pays the consumer a **points currency it controls** whose redemption value is realised through *other firms' programmes* (miles/points with a redemption-dependent worth, some of which is campaign-funded and time-boxed). The revenue *source* is structurally similar (third-party-funded); the *liability* the platform carries is different in kind (a controlled programme currency versus a cash payout), and the *redemption cost* falls partly on partners. None of that licenses a numeric take-rate transplant from one firm to the other. The structural contrast is drawn in §7; the numeric absence is recorded here and in §15. The company's hiring history — the appointment of former ShopBack/Fave managing director **Aik-Phong Ng** as Chief Commercial Officer in 2024 (⚠ first-party blog, 2 July 2024) — is a *personnel* fact, not evidence that the two firms share a model.

---

## 6. The Consumer Proposition

### 6.1 Why a User Holds Max Miles Alongside the Card's Own Rewards

The consumer proposition is additive: a user does not give up their card's rewards to use HeyMax — they earn **on top**. The company's framing: Max Miles let a user "capture value more easily into one universal travel wallet", which also "gives our partners a stickier, higher-engagement way to reach travellers" (✅ Joe Lu, quoted in blog.heymax.ai, 28 January 2026). The mechanics of the additivity: shop-through links pay the merchant's referral fee (the card's own rewards are untouched because the checkout price is unchanged, §3.2); vouchers earn miles on spending the user would do anyway; linked-card *campaigns* add Max Miles on top of the card's own points; and Card Maximiser *shows* the user the card's own points without altering them (§4.4). So the pitch is: **the same spend, more reward surfaces**. Whether that is *worth* a user's attention is a value judgement this guide does not make (§6.4).

### 6.2 Miles Versus Cashback: The Structural Difference

The distinction defined at §1.2 matters enough to restate in the consumer context. A **cashback** reward returns money-like value against spend: it is spendable as or convertible to cash, and its worth is roughly fixed and comparable across uses. A **miles** reward is a points currency redeemed inside programmes whose worth varies by *how and where* it is redeemed — the "value" of a mile is realised at redemption, not at earning. HeyMax's currency is the second kind. This has three structural consequences for a consumer: (a) the reward is *deferred* until redeemed, so its realised value depends on future redemption choices and future programme terms; (b) the reward is *partner-dependent*, because redemption is into other firms' programmes whose rules the user (and HeyMax) do not control; and (c) the reward is *elastic in value*, because the same Max Mile can be worth different amounts depending on the redemption route (§3.3). None of these is a defect — they are what a miles currency is — but they are the honest structural shape of the proposition, and they explain why the marketing emphasises **flexibility** and **redemption breadth** rather than a fixed per-dollar return.

### 6.3 The Double-Dip: A Third-Party Framing

The claim that a user can earn **two sets of rewards on one purchase** — their card's own rewards *plus* Max Miles — is real and follows from the mechanics (the card's rewards are untouched by shop-through earning, §3.2). But the crisp name for it, "**double-dipping**", and the framing itself, come from third-party consumer writing, not from the company. The consumer blog Lobang Sis says: "When you pay with a credit card, your card already earns its usual rewards… **Double-dipping** means HeyMax lets you earn Max Miles on top of those card rewards, so the same purchase earns two sets of rewards" (⚠ Lobang Sis, consumer blog, 3 June 2026). The same post calls HeyMax "the **miles version of ShopBack**" (⚠ Lobang Sis, 3 June 2026) — a characterisation quoted **with attribution** and **never** presented as the company's self-description or as a fact (§11.3). The guide uses "double-dip" only as a labelled third-party term for the additive mechanic, which itself is first-party-documented (§3.2).

### 6.4 What This Guide Cannot Establish About Relative Value

This guide cannot and does not establish whether Max Miles are "worth it", whether they beat another card or programme, or how much a Max Mile is worth in dollars — those are consumer-advice and value-computation questions the repo's genre declines. Concretely: the guide does **not** convert the company's own FlyAnywhere illustrations into a per-mile rate (§3.5); it does **not** adopt the **1.8 cents per mile** figure except as attributed to Lobang Sis (⚠ third-party, 3 June 2026); it does **not** compute a break-even or an effective cashback percentage; and it makes **no recommendation** to use or avoid the product. What it *can* say is structural: the reward is a flexible miles-type currency, deferred to redemption, partner-dependent and redemption-route-elastic (§6.2), and the honest consumer question — "does the conversion of my spend into this currency beat my alternatives, given my own redemption habits?" — is left to the reader, with the raw company-published inputs laid out in §3 and the third-party value figure flagged as third-party.

---

## 7. The Competitive Frame

### 7.1 The Five Structural Neighbours

HeyMax sits among five structural kinds of competitor, each competing for a different axis of the same consumer (attention, spend, loyalty or redemption). This frame is **structural and unranked**: no firm is described as "leading", none is given a share, and the list is a taxonomy rather than a league table (⚠ the naming of any firm here is descriptive, not a competitive assessment).

1. **Cashback platforms** — e.g. ShopBack — which aggregate merchant offers and pay the consumer cash rather than a miles currency. Structurally adjacent (aggregation via merchant fees) but with a different reward type (§5.5) and a different consumer promise (cash back versus travel value).
2. **Airline loyalty programmes** — e.g. Singapore Airlines' KrisFlyer, Cathay's Asia Miles, Qatar Airways' Privilege Club — which issue their own miles and compete for the same travel-rewarding consumer. HeyMax *transfers into* some of these (they are redemption partners), so the relationship is at once complementary (HeyMax feeds them miles) and competitive (both want the consumer's reward attention).
3. **Other airline/hotel programmes used as transfer partners** — the full set of "30+ airline and hotel programmes" (§3.4) — which are simultaneously the aggregator's *suppliers* and the consumers' alternative places to hold reward value.
4. **Issuers' own card reward schemes** — the banks' points/cashback/miles programmes on their cards. These are where the tracked reward value in §4.4 originates; a cardholder can invest their loyalty directly with the issuer instead of routing attention through an aggregator (§10.4).
5. **Merchant and coalition loyalty programmes** — e.g. the yuu coalition, GrabRewards, Link Rewards — which aggregate *merchant-side* loyalty and, in coalition form (multi-merchant), occupy the same structural position as a rewards aggregator from a different starting point (merchants first, rather than cardholders first).

### 7.2 The Aggregator's Place Among Them

The structural observation is that HeyMax is **not in any single one of these categories and borrows from all of them**. It is an aggregator whose currency is its own (§3.1), whose earn is merchant/partner-funded like a cashback platform's (§5.1), whose redemption surface is a union of airline/hotel programmes (§3.4), whose card-linking rides the issuer's rails and displays the issuer's rewards (§4.4), and whose coalition-adjacent structure resembles a merchant coalition (§7.1 item 5). This is why the competitive frame is properly drawn as **overlap**, not as a head-to-head in one market: the aggregator's distinctness is precisely that it is the *seam* between the other categories (§16 thesis). No ranking follows from this, and none is offered.

### 7.3 Why This Is Unranked

The repo's genre forbids market-share and "leading" claims, and the subject makes that discipline necessary: HeyMax's own press boilerplate calls it a "leading loyalty and travel rewards platform" (⚠ the company's self-description, quoted in its own releases and echoed by fintechnews.sg), and that phrase must **not** be printed as fact (§11.3). Similarly, no figure is given for HeyMax's share of any market, no growth rate is compared across firms, and no firm is named "the biggest". The comparisons in this section are **architectural** — what each neighbour does and how it overlaps — because that is what the evidence supports. A reader who wants a ranking will not find one here, and its absence is deliberate, not an oversight.

---

## 8. The Technology and Data Position

### 8.1 What the Platform Necessarily Sees

By the architecture of card-linking (§4.3), a HeyMax-linked user's data footprint on the platform necessarily includes: a **transaction stream** from the linked Visa card (merchant, amount, date, and the transaction-level detail Visa emits); the **last four digits** of the card number, for identification; and, for supported cards, the **derived reward accrual** (points/miles earned, cap progress, minimum-spend progress) computed by HeyMax's engine from issuer rules and the user's spend (✅ help.heymax.ai "Card Maximiser (Singapore users)", 27 February 2026). The platform also holds the account-level data of earning and redemption: Max Miles balances, voucher holdings, redemption history, and the links to transferred programmes (domain-stable for a rewards platform of this kind). Two design consequences are worth stating neutrally: the aggregator's visibility is **transaction-level and longitudinal** (it sees a stream over time, not a snapshot), and it is **merchant- and category-identified** by the MCC/Merchant-Category-Code the transaction carries — which is why the company's own AI artefact (Max AI) reasons about MCC coding and "which card should I use" (§9.2).

### 8.2 What the Company Publishes About Its Stack

The company discloses a small, specific set of stack facts, each first-party and dated:

- **Cloud and encryption:** "We use **Google's Cloud Infrastructure**, which benefits from the same security levels as Google's own services. Your data is always **encrypted when sent to Cloud Storage**" (✅ help.heymax.ai "Card Maximiser: Security and Privacy", 20 June 2024).
- **Deletion on unlink:** "When you unlink your card from heymax at anytime, your data is **deleted forever**" (✅ 20 June 2024).
- **Access discipline:** "Only you can access your transaction data. No one else, not even heymax employees, have access to your data unless given your explicit consent" (✅ 20 June 2024).
- **No data sale:** "We will **never sell your data to third parties**" (✅ 20 June 2024).
- **Card-linking rail:** Visa-hosted linking page, Visa On Platform enrolment, transaction feed, no card-number access, no charge capability (✅ help.heymax.ai "Card Maximiser (Singapore users)", 27 February 2026; ✅ "Max AI", 6 April 2026).

What the company does **not** publish: the internal service architecture, the specific use of any particular database or ML stack beyond the Google Cloud statement, security certifications or audit attestations, and the retention schedule beyond "deleted forever on unlink". Those absences are recorded rather than filled (§15).

### 8.3 The Data Question as Architecture, Not Risk

This guide frames the data question as a matter of **architecture**, not as a risk rating. Architecturally, a card-linked aggregator occupies a **read-and-model** position: it receives a transaction feed the user consents to share (§4.3), models issuer-reward value from it (§4.4), and (on the company's published account) deletes the linked data on unlink and never sells it (§8.2). The position does not include the ability to move money: the company states the linking "does not give HeyMax the ability to access your card number nor charge your card" (✅ help.heymax.ai, 27 February 2026), and its AI is read-only (§9.2). So the architectural characterisation is: **a consented, read-and-model data position over transaction and reward data, with the scheme (Visa) as the linking intermediary** — which is why the issuer-and-network relationship (§10) is a data-and-attention relationship as much as a commercial one. Whether any particular data-protection standard is met is a compliance question outside this guide's scope; the guide states the architecture and the company's own published commitments, and stops there. It does not assign a risk score, and it does not treat the company's "never sell your data" line as anything more than a company commitment, dated.

---

## 9. The AI Positioning

### 9.1 The Vision Line, Quoted and Attributed

HeyMax's About page states a vision: "Our vision is to build the **default protocol for every business to engage with every consumer in the AI-Native Era**" (✅ heymax.ai/company/about). This is a **positioning statement** — the company's own aspirational framing of where it wants to sit — and it is quoted and attributed as such. It is **not** an existing general-purpose AI platform, not a standard, and not a demonstrated "protocol" in the technical sense; the guide says this plainly so that the sentence is not mistaken for a capability claim. The Series A materials reinforce the direction without adding substance: the round's stated use of funds includes an "AI-empowered rewards experience" and, in the press account, "AI-enabled features" (✅ blog.heymax.ai, 28 January 2026; ✅ fintechnews.sg, 28 January 2026). Aspiration is dated to the announcements; capability is assessed only against the artefact, below.

### 9.2 The Concrete Artefact: Max AI / Card Spend AI

The one concrete, published AI artefact behind the vision is **"Max AI"** (also referred to in the help centre as **Card Spend AI**; the assistant is named "Max"). The company documents it in a help-centre article updated **6 April 2026**: it is "an AI chat assistant in the HeyMax app that helps you understand your credit card spending and rewards" (✅ help.heymax.ai "Max AI", 6 April 2026). Its documented scope: answering questions such as "What's my best card for dining?", "How much did I spend last month?", "How close am I to my UOB VS cap?"; classifying merchants by MCC and recommending which card to use; explaining card terms and conditions; and helping a user reason about where they could transfer their bank points (✅ 6 April 2026). Notably, it can answer questions about card reward structures even with **no cards linked**, and works best with Visa cards linked and VOP-enrolled, enabling "real-time transaction tracking, spending analysis, and personalized reward calculations" (✅ 6 April 2026). It is described as being rolled out in stages; it is focused on **Singapore-issued cards only**; and it cannot forecast future spending or earnings (✅ 6 April 2026).

### 9.3 The Company's Own Caveats

The reveal in this article is how candid the company is about the artefact's limits — quotes worth carrying because they bound the claim:

- **Read-only.** "**No. Max is read-only.** It can analyze your spending and provide recommendations, but it cannot: change your card settings; dispute or modify transactions; enroll or unenroll cards; transfer miles or redeem rewards; access your bank account directly" (✅ 6 April 2026).
- **Calculations are not the model's guesses.** "The numbers come **directly from our calculation engine, not AI estimates**" (✅ 6 April 2026) — i.e. the reward math is done by a deterministic engine, and the AI is a natural-language front end over it.
- **Imprecision is acknowledged for interpretation.** "While Max's reward calculations are highly accurate (they use the same engine as HeyMax's card tracking), **AI responses about general advice or T&C interpretation could occasionally be imprecise**" (✅ 6 April 2026).
- **Rate limits.** "You may have hit the message limit (**20 per minute or 500 per day**)" (✅ 6 April 2026).
- **Market scope.** "Currently, Card Spend AI is focused on **Singapore-issued credit cards only**" (✅ 6 April 2026).

The article also documents pragmatics that further bound the claim: there is a short delay between a bank announcing a T&C change and Max reflecting it; MCC mismatches are a known failure mode; and "MCC Search" and "Ask Max" are separate entry points the team is "actively working to unify" (✅ 6 April 2026).

### 9.4 Assessing the Claim Against the Artefact

Holding the vision line (§9.1) next to the artefact (§9.2–§9.3), the honest assessment is:

- The **"default protocol … AI-Native Era"** line is a **vision/positioning statement**; measured against what is published, it is not yet an instantiated protocol or platform (✅ assessment based on the two sources).
- The **demonstrable artefact** is a **rewards-optimisation chat assistant** that recommends which card to use, tracks caps and minimum-spend progress, reads card T&Cs, and helps with transfer reasoning — over a deterministic reward engine, read-only, rate-limited and Singapore-scoped (✅ help.heymax.ai "Max AI", 6 April 2026).
- The company's own caveats (read-only, engine-not-estimate, "could occasionally be imprecise", rate-limited, SG-only) are the correct calibration of what the artefact does and does not do, and they are more informative than the vision line.

So: the AI *positioning* is aspirational and dated to the company's own materials; the AI *artefact* is a narrow, well-scoped, read-only rewards assistant. The guide states both, attributes both, and declines to infer a broader capability — in particular, it does **not** imply a general-purpose AI platform, an autonomous agent that acts on money, or an "AI" that computes the consumer's rewards itself (the company says the opposite: the engine computes, the AI explains).

---

## 10. The Card-Issuer and Network Relationship

This section develops the relationship that §1.4 flagged and §4.4 anchored. It is written as **architecture of an arrangement** — who provides, who links, what each party sees, where interests converge and diverge — and deliberately reaches **no judgement of either party**. The relationship is simultaneously **cooperative and competitive**, and the point of the section is to hold both truths at once rather than collapse to one.

### 10.1 Who Provides the Cards, Who Links Them

The division of roles is clean and is stated or implied by the first-party record:

- **The issuer (a bank) provides the card and owns the reward programme.** The card, its reward rules, its caps, its minimum-spend thresholds and the points/miles it confers are the issuer's product (✅ the tracked rewards are "rewards from the card issuer / bank", help.heymax.ai "Card Maximiser (Singapore users)", 27 February 2026).
- **Visa provides the linking rail.** The user enrols on a Visa-hosted page; Visa On Platform carries the enrolment and sends the transaction feed (✅ 27 February 2026; ✅ "Max AI", 6 April 2026).
- **HeyMax links, consumes and surfaces.** HeyMax provides the linking option inside its app, receives the feed, models the issuer's reward accrual, and displays the result to the user (§4.3–§4.4).

No party is displaced by another: the issuer still issues and still owns the programme; Visa still carries the rails; HeyMax adds an aggregation-and-visibility layer on top. The arrangement is **additive by construction**, which is what makes the "cooperative" half true.

### 10.2 What the Aggregator Sees, What the Issuer Sees

The information asymmetry is the heart of the arrangement, and it runs in an unusual direction.

- **What HeyMax (the aggregator) sees:** the user's transaction stream on the linked card, the last four digits of the card number, and — modelled from issuer rules — the reward accrual and cap/min-spend progress (✅ help.heymax.ai, 27 February 2026). For a user who also shops through brand links and buys vouchers, HeyMax additionally sees those earning events (§3.2). In short, HeyMax sees a **cross-issuer, cross-merchant view of the user's reward activity** — a view no single issuer has, because no issuer sees the user's other cards.
- **What the issuer sees:** the issuer sees its own cardholder's spend on its own card (as it always did) and its own programme's accrual (as it always did). What the issuer does **not** automatically see, from the aggregator, is the cross-card, cross-merchant aggregation of the user's reward behaviour — unless there is a data arrangement this guide has not found published. The public record shows the issuer's reward value flowing **into** HeyMax (via the feed), not a reverse flow of the aggregator's aggregate view back to the issuer (⚠ no reverse data arrangement is published; recorded as not established).

So the architectural asymmetry is: **the aggregator assembles a user-level reward picture across issuers and merchants; each issuer holds only its own slice.** That asymmetry is the substance of what a bank would reason about (§12.5), and it is stated here as structure, not as a wrong.

### 10.3 Where the Interests Converge

Several interests are genuinely shared, which is why the relationship is cooperative:

- **Reward visibility and engagement.** A tool that shows a cardholder their points/miles, caps and minimum-spend progress can increase the cardholder's engagement with the card — the issuer's own loyalty objective. HeyMax's Card Maximiser is, functionally, a **third-party reward-visibility layer** that the issuer did not have to build (✅ its purpose is informational, §4.4).
- **Spend encouragement.** By recommending the "best card" for a purchase (and by surfacing how close a user is to a bonus cap), the aggregator can nudge *more* usage of the issuer's card in the bonus-earning category — aligned with the issuer's interchange and engagement economics (domain-stable inference from the product's function; stated as structural alignment, not a measured outcome).
- **Campaign distribution.** When an issuer or partner funds a card-linked Max Miles campaign (§4.6), it buys distribution and an action from HeyMax's user base — a channel the issuer values, and a revenue event for HeyMax (§5.2).
- **Customer acquisition.** "Special partner actions" explicitly include card applications (✅ blog.heymax.ai, 28 May 2025), so HeyMax can act as an acquisition channel for issuers.

### 10.4 Where the Interests Diverge

Several interests genuinely pull apart, which is why the relationship is also competitive:

- **Ownership of the loyalty relationship.** The issuer's programme exists, in part, to build a direct loyalty bond with its cardholder. An aggregator that becomes the place a user checks "which card should I use" (§9.2) interposes itself in that bond: the user's reward *attention* is routed through the aggregator, even though the reward *value* is the issuer's (§4.4). The issuer's loyalty economics and a third party aggregating its cardholders' reward value are, at that seam, in structural tension.
- **Aggregation across issuers.** The aggregator's cross-issuer view (§10.2) is precisely a view no issuer has, and it is built from the issuer's own programme data flowing out. A bank might reasonably see this as its reward value being used to power a neutral cross-issuer comparison — helpful to the cardholder, and not obviously helpful to any single issuer's loyalty share.
- **Campaign dependence and disintermediation.** Because card-linked Max Miles are campaigns (§4.6), the aggregator's card-linked earning exists only while partners fund it. If an issuer stopped funding campaigns, HeyMax would still *track* the issuer's rewards (§4.4) but would no longer *pay* Max Miles on that card — a dependence that runs one way.
- **The "not the issuer" line cuts both ways.** HeyMax's own disclaimer — "HeyMax is not the one issuing the rewards" (§4.4) — is both a liability shield for the aggregator *and* a reminder that the issuer retains the reward liability. Each side has a claim on the truth of that sentence, and neither is wrong.

### 10.5 The Visa Network Relationship, Same Discipline

The network layer gets the same treatment. **What the relationship is:** a card-linking data integration. Visa provides the hosted linking page and the Visa On Platform transaction feed; HeyMax consumes it and surfaces the data (§4.5). That is a real, disclosed, functional integration, and it is the mechanism behind Card Maximiser. **What it is not:** on the public record, there is no Visa endorsement of Max Miles as a currency, no statement that Visa issues or backs Max Miles, no exclusivity, and no published commercial terms (§4.5). And the presence of a former **Visa APAC president** as a strategic *investor* (§2.5) is a fact about an individual's background, not a Visa corporate relationship — the guide does not merge the two. Structurally, a card network benefits from card usage and from anything that increases transaction volume on its rails; a card-linking aggregator can increase engagement with cards, which is aligned, while the *rewards* being aggregated are the issuers' and the programmes' rather than the network's — so the network's interest is the most cleanly aligned of the three parties, and the guide describes it without overstating it.

### 10.6 The Arrangement in One Table

| Dimension | The issuer (bank) | The network (Visa) | The aggregator (HeyMax) |
|---|---|---|---|
| Provides | The card, the reward programme, its rules and liability | The linking rail (Visa-hosted page / VOP) and the transaction feed | The app, the linking user experience, the reward-modelling engine and the currency (Max Miles) |
| Sees | Its own card's spend and programme accrual | The network's transaction flow (scheme-level) | The linked user's transaction stream, last-4, and modelled reward accrual (§10.2) |
| Gets from the arrangement | Reward visibility and spend engagement created for free (§10.3) | Increased card engagement and rails volume (§10.5) | A live data feed and a distributing user base (§4, §10.3) |
| Diverges on | Ownership of the cardholder's loyalty attention (§10.4) | — (most cleanly aligned) | Dependence on issuer/partner funding for card-linked earning (§4.6, §10.4) |

The arrangement is a **three-party cooperation around a data-and-attention exchange**, with a structural tension between the issuer's loyalty bond and the aggregator's cross-issuer visibility. The guide leaves the tension standing; §12 reasons about it from the bank's seat without resolving it.

---

## 11. The Evidence Reality

### 11.1 The Three Kinds of Evidence

Everything this guide says rests on one of three kinds of evidence, and the distinction is load-bearing for a young private company whose own pages are the main window onto it:

1. **First-party pages and announcements (verifiable, dated).** HeyMax's Terms of Use (6 February 2025), About page, help-centre articles ("Card Maximiser (Singapore users)", 27 February 2026; "Card Maximiser: Security and Privacy", 20 June 2024; "Adding Cards Basics", 20 June 2024; "Max AI", 6 April 2026; the "Max Miles Turns 3!" campaign terms, 22 September 2026), the Max Miles explainer (hk.heymax.ai, ©2025), and the blog announcements (2 July 2024; 28 May 2025; 28 January 2026). These are dated and readable; they are the strongest evidence for **what the company says it does**, and the terms are the strongest evidence for **what the user is actually bound to** (the expiry finding, §3.6).
2. **The company's own marketing and self-claims (attributed, dated).** Product promises ("never expires", §3.1), self-descriptions ("leading loyalty and travel rewards platform", §7.3), scale claims (users, merchants, miles, revenue, §2.6), and the AI vision line (§9.1). Category: the company describing itself. Dated, attributed, never adopted as fact.
3. **Third-party opinion and reporting (cited as such).** Independent press (fintechnews.sg, 2 July 2024 and 28 January 2026 — reporting that corroborates the funding facts and repeats company figures), the dated third-party record for the attributed founder names (Crunchbase, Preqin, Tenity, §2.3), consumer-blog characterisations (Lobang Sis "miles version of ShopBack" and "double-dipping", 3 June 2026) and consumer miles blogs (MileLion). Category: outside opinion and secondary reporting — quoted with attribution, never presented as the company's own words and never as fact.

### 11.2 How to Weigh Them

The weighting rule this guide applies: **first-party pages are evidence of what the company publishes; company claims are evidence of what the company asserts; third-party opinion is evidence of what observers think — and none of the three is evidence of an outcome the guide has independently measured.** Concretely:

- For **what the product is mechanically** (how linking works, how earning works, what the terms say), the first-party pages are strong and are marked ✅, because they are the authoritative description of the company's own product and contract.
- For **what the company has achieved** (scale, growth, revenue), the company's own figures are the only source and are marked ⚠ as company claims, dated and attributed — useful for trend, not for audited fact.
- For **what the product is worth to a consumer or a market**, third-party writing is opinion; it is cited as opinion (§11.3) and never converted into a guide-level finding.
- For **what is not published** (valuation, take rate, later rounds), the absence is the finding, recorded in §15 rather than filled by inference from comparables (§5.5) or by charity to the company's optimism.

### 11.3 The Characterisations That Must Stay Attributed

Three characterisations recur in this space and are handled with maximal care:

- **"The miles version of ShopBack."** A consumer blog's framing (⚠ Lobang Sis, 3 June 2026). It is quoted **with attribution**, is **not** the company's self-description, and is **not** presented as fact; the structural contrast that makes the analogy imperfect is drawn in §5.5 and §7.
- **"Leading loyalty and travel rewards platform."** The company's **own** press boilerplate (⚠ self-description, echoed by fintechnews.sg and aggregator sites). It is the company's claim about itself, never printed as fact, and no ranking or market-share claim is built on it (§7.3).
- **MileLion and consumer-blog reviews.** Consumer miles writing covering HeyMax campaigns and reviewing the product. These are **opinion**, cited as opinion (e.g., the August 2024 campaign coverage, ⚠ MileLion, 29 August 2024), never as the company's words or as neutral fact.

The rule this guide enforces throughout: outside characterisations are quoted **with attribution**, never adopted, never presented as the company's self-description, and never used as the basis for a recommendation to use or avoid the product (there is none in this guide).

---

## 12. The Cymbal Bank Worked Example: A Third-Party Aggregator View

### 12.1 The Design-Fiction Frame

**This section is design fiction.** Cymbal Bank is the repo's fictional Singapore bank persona and the **only** institution in this worked example; it is not a real bank, HeyMax is not a party to it, and nothing here is a statement about any real bank's strategy, relationship or intentions. The purpose is the repo's standard one: to let a bank reason *through* verified material (§5, §6, §10) without asserting a verdict. Every verified anchor is cited; every invented detail is labelled.

### 12.2 The Scenario: A Bank Watches Its Cardholders' Reward Value Aggregate

Cymbal Bank's cards-and-loyalty team notices a structural pattern (design fiction, built from verified §4 and §10 material): a measurable share of its cardholders has linked a Cymbal Bank Visa card to a third-party rewards aggregator of the HeyMax type (✅ card-linking via Visa On Platform is the disclosed mechanism, §4.2). Through that linking, the aggregator receives the users' transaction streams and models the Cymbal Bank programme's own reward accrual, caps and minimum-spend progress (✅ Help Centre, 27 February 2026, §4.4). The team's question is deliberately **not** "is this good or bad?" but "what does it mean for our loyalty economics and our customer relationship, and how do we hold it as an arrangement rather than a threat?" The team works this through §5, §6 and §10 in order, and reaches **no verdict**.

### 12.3 Working Through Section 5: The Loyalty Economics

The team starts from the verified model of the aggregator's own economics (§5): the aggregator is **merchant- and partner-funded**, earning referral fees, voucher margins and campaign funding, with the guiding rule "if you don't earn, we don't earn" (✅ §5.1, §5.3). The observation that follows is structural: the aggregator's revenue does **not** come from the issuer's loyal-customer spend as such; it comes from merchants and campaign sponsors who want the aggregator's users to take an action (buy, apply, book) — and Cymbal Bank can itself become such a sponsor when it funds a card-linked campaign (✅ §4.6, §10.3). So one reading is that the aggregator is a **channel Cymbal Bank can buy** — a card-application and card-usage channel — priced in Max Miles rather than cash. The open question the team writes down (not answers): since the aggregator keeps a spread it does not publish (✅ take rate not published, §5.4), what is the *effective* cost of that channel versus Cymbal Bank's own acquisition and loyalty spend? The guide does not answer this either; the point is that a bank would need the unpublished figure to know, and the figure is not public.

### 12.4 Working Through Section 6: The Customer Relationship

Working §6, the team notes the additivity of the proposition: a Cymbal Bank cardholder earns their Cymbal Bank rewards **and** Max Miles on the same spend without Cymbal Bank paying for the extra layer (✅ §6.1, §3.2). On the face of it, that is a free enhancement to the *perceived* value of holding a Cymbal Bank card — engaged cardholders who feel they are getting more from their Cymbal Bank card. But the team also notes the deferred-and-partner-dependent character of the reward (§6.2): Max Miles are realised at redemption, into other firms' programmes, with a value the cardholder judges rather than Cymbal Bank sets. The relationship question the team writes down (not answers): *when a cardholder's reward attention routes through an aggregator that is neutral across Cymbal Bank and its competitors, does Cymbal Bank's own loyalty bond strengthen — because the cardholder is more engaged with rewards — or weaken — because the reward relationship is experienced one layer removed from Cymbal Bank?* The guide holds both possibilities open, exactly as §10.4 does.

### 12.5 Working Through Section 10: The Cooperative-Competitive Position

Finally the team works §10 directly, holding the cooperative and competitive halves together:

- **Cooperative:** the aggregator hands Cymbal Bank three things it did not have to build — reward visibility for its cardholders (✅ §4.4, §10.3), a spend-encouragement nudge toward the card's bonus category, and a campaign distribution channel (§10.3). For a card program competing for "top of wallet", these are genuine, if indirect, benefits.
- **Competitive:** the aggregator assembles a **cross-issuer, cross-merchant view** of the cardholder's reward behaviour that Cymbal Bank does not itself hold (✅ §10.2), built from the Card Maximiser feed. That asymmetry — the aggregator seeing across issuers, Cymbal Bank seeing only its own slice — is a structural fact the team has to reason about, because it means the aggregator can tell the cardholder "which card is best" across all their cards, with Cymbal Bank as one option among many (✅ the product's own function, §9.2).

The team's disciplined conclusion (design fiction) is itself the guide's method: **this is an arrangement with real cooperative value and real competitive tension, and Cymbal Bank's rational response is to understand and manage the interface — how Cymbal Bank's programme rules are modelled, what a card-linked campaign is worth, whether any reverse data or co-marketing arrangement exists — rather than to read the relationship as purely one or the other.** The guide offers no verdict beyond the bank's disposition to *reason*, not to act.

### 12.6 The Flow in Sequence

The end-to-end sequence for a single Cymbal Bank cardholder (design fiction; each step cites its verified anchor):

1. **The cardholder holds a Cymbal Bank Visa card** with its own reward programme (the issuer's product — ✅ §10.1).
2. **The cardholder links the card to the aggregator**, enrolling on a Visa-hosted page; Visa On Platform carries the consent and the feed (✅ §4.2).
3. **The aggregator receives the transaction stream** and the last-4, and models the Cymbal Bank programme's accrual, caps and minimum-spend progress for the cardholder (✅ §4.3–§4.4).
4. **The cardholder also earns Max Miles separately** — via brand links, vouchers, and any card-linked campaigns Cymbal Bank or a partner funds (✅ §3.2, §4.6).
5. **The cardholder consults the aggregator** for "which card to use" — Max AI or Card Maximiser — seeing a cross-card recommendation in which the Cymbal Bank card is one option (✅ §9.2).
6. **Cymbal Bank sees its own slice** (its card's spend and programme accrual) but not the aggregator's cross-issuer aggregate, unless a data arrangement exists that this guide has not found (✅ §10.2; ⚠ no reverse arrangement published).
7. **Optionally, Cymbal Bank funds a campaign** to buy card-linked distribution or applications through the aggregator (✅ §4.6, §10.3), in which case the relationship is explicitly commercial and cooperative for the duration of the campaign.
8. **The cardholder redeems Max Miles** by transferring into an airline/hotel programme, via FlyAnywhere, with cash, or for vouchers (✅ §3.3) — a value realised outside Cymbal Bank entirely.

The sequence's design point is that Cymbal Bank is present at steps 1, 6 and (optionally) 7, while the cardholder's reward *experience* increasingly lives at steps 3–5 and 8 — which is the interface the bank would manage.

### 12.7 Ending on the Thesis

The worked example ends where §1.5 and §16 end: the aggregator's strength is convenience at the seam between programmes, and its structure is **borrowed** — value sourced from merchants, partners and issuers, handed to the consumer as a currency it controls but did not manufacture. For Cymbal Bank the takeaway is not a verdict but a posture: **understand the interface, weigh the cooperative benefit against the aggregation asymmetry, and treat the arrangement as an ongoing architecture to be managed.** The guide states the architecture and stops short of counselling either collusion or confrontation, because a private early-stage company's relationship with an unnamed bank is not a matter on which the public record supports a verdict.

---

## 13. The Anti-Patterns and Open Questions

### 13.1 The Anti-Patterns: Symptom, Cause and Guardrail

The recurring failure modes when reading a company like HeyMax, each with the guardrail this guide applies:

| # | Symptom | Cause | Guardrail |
|---|---|---|---|
| 1 | Treating "never expires" as a fact | The marketing line is repeated without the terms | Quote it as the company's promise and contrast with Terms §4 (12-month inactivity) and §7 (reserved expiry) — §3.6 |
| 2 | Adopting a company's self-description | The boilerplate "leading platform" reads like editorially-vetted fact | Attribute company self-descriptions; print no ranking — §7.3, §11.3 |
| 3 | Adopting a third-party characterisation | Consumer blogs describe the product in memorable shorthand | Quote-with-attribution only ("miles version of ShopBack"); never as self-description or fact — §11.3 |
| 4 | Assuming the funding story continues | Early-stage firms often raise more, later | Assume only the published rounds; record later rounds as not published — §2.5, §15 |
| 5 | Transplanting a comparable's economics | ShopBack and HeyMax both aggregate rewards | Describe the model structurally; record take rate as not published; do not infer figures — §5.5 |
| 6 | Mistaking tracked rewards for Max Miles | Card Maximiser's UI shows points/miles | The tracked rewards are the *issuer's*; HeyMax is "not the one issuing the rewards" — §4.4 |
| 7 | Overstating the Visa relationship | A named partner implies more than a data rail | State the card-linking integration; record no endorsement/exclusivity/backing — §4.5, §10.5 |
| 8 | Treating the issuer relationship as adversarial *or* purely friendly | The arrangement is genuinely both | Hold cooperative and competitive halves together; no judgement — §10 |
| 9 | Reading the AI vision line as capability | "Default protocol … AI-Native Era" reads as a product | Quote as positioning; assess only against the read-only Max AI artefact — §9 |
| 10 | Computing value-per-mile as advice | The FlyAnywhere rate invites a number | Print the company's inputs; attribute the third-party 1.8c figure; offer no recommendation — §3.5, §6.4 |
| 11 | Ingesting company scale figures as audited | The figures are vivid and repeated | Mark every company figure ⚠ company claim, dated, attributed — §2.6 |
| 12 | Filling absences by inference | A complete-looking guide gaping at "valuation unknown" | State the absence as the finding — §15 |

### 13.2 The Open Questions

The honest open questions — recorded, not answered, because the public record does not support an answer:

- **What is the take rate?** Not published (§5.4). Without it, no effective-cost or unit-economics view is possible from public data.
- **What does a card-linked campaign cost a sponsor?** The campaigns' existence is published (e.g. Trust Freedom, Chocolate, ✅ §4.6) but the funding terms are not.
- **Is there any reverse data or co-marketing arrangement between HeyMax and any issuer?** No such arrangement is published (§10.2, ⚠).
- **How does the "default protocol" vision relate to the Max AI artefact?** The artefact is narrow and read-only; the vision is broad (§9.4). The gap is a roadmap question the company has not published.
- **What happens to a transferred mile inside the partner programme?** Once Max Miles transfer 1:1 into, say, an airline programme, the partner's own expiry and redemption rules govern — and HeyMax does not publish reverse-redemption mechanics (§15).
- **What is the legal registration and shareholding?** The ACRA registration number was not read this pass; the cap table is not public (§15).
- **How durable is the merchant/partner funding base?** The model depends on ongoing third-party funding (§5) and campaign-dependent card-linked earning (§4.6); the persistence of that base is not published.

---

## 14. The Claims Audit

### 14.1 The Verified-Facts Table

**✅ = verified this pass at the named source (first-party page/announcement, or dated independent press), with date and kind.** Company claims about itself are verified *as claims* (the company did publish them) and are also flagged in §14.2.

| # | Claim | Mark | Source / date / kind |
|---|---|---|---|
| 1 | Legal operator is Max Now Pte. Ltd. (trading as HeyMax) | ✅ | heymax.ai/company/terms, "Max Now Pte. Ltd. (HeyMax)", last updated 6 Feb 2025 (first-party terms); hk.heymax.ai footer "©2025 MAX NOW PTE LTD" |
| 2 | Founded in 2023 | ✅ | blog.heymax.ai Series A announcement, 28 Jan 2026 (first-party announcement) |
| 3 | Founded by four former Meta engineers (count + former employer) | ✅ | blog.heymax.ai, 2 Jul 2024 and 28 Jan 2026 (first-party announcements) |
| 4 | CEO & co-founder is Joe Lu (named, quoted) | ✅ | blog.heymax.ai, 2 Jul 2024 and 28 Jan 2026; fintechnews.sg same dates |
| 5 | Headquartered in Singapore | ✅ | blog.heymax.ai, both announcements (first-party) |
| 6 | First international market: Hong Kong, 2025 (via acquisition of krip) | ✅ | blog.heymax.ai Series A, 28 Jan 2026; fintechnews.sg, 28 Jan 2026 |
| 7 | Stated expansion markets: Japan, Taiwan, Australia by end-2026 (plan) | ✅ | blog.heymax.ai Series A, 28 Jan 2026 (first-party announcement) |
| 8 | Seed: US$2.6M (SG$3.5M), 2 Jul 2024, led by January Capital; Tenity, Ascend Angels, XA Network + angels | ✅ | blog.heymax.ai, 2 Jul 2024; fintechnews.sg, 2 Jul 2024 |
| 9 | Series A: US$11M, 28 Jan 2026, led by Peak XV; Betatron; continued January Capital & Tenity; strategic investors Rob Rosenstein, David Lee | ✅ | blog.heymax.ai, 28 Jan 2026; fintechnews.sg, 28 Jan 2026 |
| 10 | Max Miles is the company's currency; company definition "a universal travel currency you earn from your everyday spend, that never expires" | ✅ (as the company's product claim) | hk.heymax.ai/maxmiles, ©2025 (first-party page) |
| 11 | Earning works through four routes: brand links, vouchers, card-linked rewards, partner actions | ✅ | blog.heymax.ai "How we get you your Max Miles", 28 May 2025 (first-party article) |
| 12 | Redemption: 1:1 to 30+ programmes; FlyAnywhere (fixed rate per mile); Miles + Cash; vouchers | ✅ | hk.heymax.ai/maxmiles; blog.heymax.ai, 28 Jan 2026 |
| 13 | Terms §4: 12-month inactivity → right to remove user (incl. full balance) | ✅ | heymax.ai/company/terms, 6 Feb 2025 (first-party terms) |
| 14 | Terms §7: reserved right to "add or change the duration taken for Max Miles to expire" | ✅ | heymax.ai/company/terms, 6 Feb 2025 |
| 15 | Help centre: miles do not expire "as long as your HeyMax account stays active" (login ≥ once / 12 months) | ✅ | help.heymax.ai "Max Miles Turns 3!", updated 22 Sep 2026 |
| 16 | Terms §6: no proprietary right; not monies held on trust; contractual rights of repayment only | ✅ | heymax.ai/company/terms, 6 Feb 2025 |
| 17 | Card linking is Visa-only, via a Visa-hosted webpage; Visa sends a transaction stream; no card-number access, no charge capability | ✅ | help.heymax.ai "Card Maximiser (Singapore users)", updated 27 Feb 2026 |
| 18 | Tracked points/miles "are not Max Miles, but rewards from the card issuer / bank"; HeyMax "is not the one issuing the rewards" | ✅ | help.heymax.ai, 27 Feb 2026 (first-party) |
| 19 | HeyMax collects transaction history + last 4 digits; deletes on unlink; uses Google Cloud; encrypts to Cloud Storage; never sells data | ✅ | help.heymax.ai "Card Maximiser: Security and Privacy", 20 Jun 2024 |
| 20 | Visa On Platform (VOP) is the card-enrolment channel | ✅ | help.heymax.ai "Max AI", 6 Apr 2026 |
| 21 | Card-linked *Max Miles* run as limited-time campaigns (e.g. Chocolate; Trust Freedom "Max Miles Turns 3!", 21–30 Sep 2026, cap 3,003) | ✅ | blog.heymax.ai, 28 May 2025; help.heymax.ai, 22 Sep 2026 |
| 22 | Business model: merchant/partner-funded; four earning routes; rule "if you don't earn, we don't earn" | ✅ | blog.heymax.ai, 28 May 2025 (first-party) |
| 23 | Max AI / Card Spend AI: read-only, calculations from the engine "not AI estimates", rate-limited (20/min, 500/day), SG cards only, "could occasionally be imprecise"; updated 6 Apr 2026 | ✅ | help.heymax.ai "Max AI", 6 Apr 2026 |
| 24 | About-page vision: "default protocol for every business to engage with every consumer in the AI-Native Era" | ✅ (as the company's positioning statement) | heymax.ai/company/about (first-party page) |
| 25 | Reward tracking supports a named set of top miles cards; transaction tracking on almost all SG Visa cards; no Mastercard/Amex linkage | ✅ | help.heymax.ai, 27 Feb 2026 |
| 26 | Company-reported scale (users, miles issued, merchants, partners) at two dates | ✅ (as published claims — see §14.2) | blog.heymax.ai, 2 Jul 2024 and 28 Jan 2026 |
| 27 | Reported revenue: "fivefold Y-o-Y growth", "annualised revenue run rate of US$6 million" | ✅ (as reported — see §14.2) | fintechnews.sg, 28 Jan 2026 (reporting the company's statement) |
| 28 | Aik-Phong Ng (ex-ShopBack/Fave MD) appointed Chief Commercial Officer in 2024 | ✅ (as first-party statement) | blog.heymax.ai, 2 Jul 2024 |

### 14.2 The Flagged Claims Table

**⚠ = flagged: reported, approximate, single-sourced, dated, fast-moving, or a claim the company makes about itself — attributed, dated, not adopted as guide fact.**

| # | Claim | Why flagged | Source / date / kind |
|---|---|---|---|
| 1 | Founders' full names: Jialu Zhong, Ke Wang (CTO), Sean Dy (COO) | Not first-party; registry/third-party derived | ⚠ Crunchbase (registry card), Preqin (database), Tenity (accelerator post) — third-party |
| 2 | Users ">50,000" → ">150,000" | Company claim | ⚠ blog.heymax.ai, 2 Jul 2024 / 28 Jan 2026 |
| 3 | Max Miles issued ">50M since Sep 2023" → ">500M annually" | Company claim | ⚠ blog.heymax.ai, 2 Jul 2024 / 28 Jan 2026 |
| 4 | Merchants ">500" → ">800"; partners "25" → "30+" | Company claim | ⚠ blog.heymax.ai, 2 Jul 2024 / 28 Jan 2026 |
| 5 | Revenue: fivefold YoY; ~US$6M annualised run rate | Company-reported; unaudited | ⚠ fintechnews.sg / Business Times, Jan 2026 |
| 6 | Target: "strong triple-digit annual GMV growth over the next two years" | Company target, not result | ⚠ blog.heymax.ai, 28 Jan 2026 |
| 7 | Seed size "US$2.6M" vs "US$2.7M" | First-party inconsistency between announcements | ⚠ blog.heymax.ai, 2 Jul 2024 (US$2.6M) vs 28 Jan 2026 (US$2.7M) |
| 8 | FlyAnywhere rate = 1.8 cents per mile | Third-party only; first-party says "fixed rate" without the figure | ⚠ Lobang Sis, consumer blog, 3 Jun 2026 |
| 9 | "Miles version of ShopBack" | Third-party characterisation | ⚠ Lobang Sis, 3 Jun 2026 |
| 10 | "Leading loyalty and travel rewards platform" | Company self-description (boilerplate) | ⚠ blog.heymax.ai (both dates); echoed by fintechnews.sg |
| 11 | "Double-dipping" as a named concept | Third-party framing of a real mechanic | ⚠ Lobang Sis, 3 Jun 2026 |
| 12 | August 2024 Visa campaign "3 mpd on bus/MRT rides" | Third-party coverage | ⚠ MileLion, consumer miles blog, 29 Aug 2024 |
| 13 | Peak XV's ">40% of card revenues … over $100B globally spent on loyalty and consumer rewards" | An investor's framing, quoted in a company release | ⚠ quoted in blog.heymax.ai, 28 Jan 2026 |
| 14 | Partner-programme list (Cathay, ALL Accor, Qatar, EVA Air, JAL, World of Hyatt) | Illustrative, dated, subject to change | ⚠ blog.heymax.ai, 28 Jan 2026 / Lobang Sis, 3 Jun 2026 |
| 15 | Order of company-reported milk/partner counts "30+" | Company claim, rounded | ⚠ blog.heymax.ai, 28 Jan 2026 |

### 14.3 The Rejected and Not-Found Claims

**❌ = refuted, or affirmatively not found in the record examined this pass.**

| # | Claim | Why ❌ / not found |
|---|---|---|
| 1 | A published valuation | ❌ Not found — no valuation is published (§15) |
| 2 | Funding rounds beyond seed + Series A | ❌ Not found — no later round is published (§15) |
| 3 | A published take rate or unit economics | ❌ Not found (§5.4, §15) |
| 4 | A published profitability figure | ❌ Not found (§15) |
| 5 | The ACRA registration number / shareholding | ❌ Not read this pass (§15) |
| 6 | First-party publication of the founders' full names | ❌ Not found — HeyMax's own pages name only Joe Lu (§2.3) |
| 7 | A Visa endorsement of Max Miles, or a statement that Visa issues/backs Max Miles | ❌ Not found (§4.5, §10.5) |
| 8 | Visa–HeyMax exclusivity or published commercial terms | ❌ Not found (§4.5) |
| 9 | Reverse-redemption / partner-programme expiry mechanics after transfer | ❌ Not published by HeyMax (§15) |
| 10 | The exact FlyAnywhere numeric rate, first-party | ❌ Not found first-party (only the third-party 1.8c figure, §14.2 row 8) |
| 11 | Any market-share, "leading", ranking or growth-versus-peer claim printed as fact | ❌ Not asserted by this guide, and no source supports it (§7.3) |
| 12 | A HeyMax guarantee that Max Miles can never expire | ❌ Refuted by the terms — §4 reserves inactivity forfeiture and §7 reserves the right to introduce expiry (§3.6) |
| 13 | Any Meta corporate relationship to HeyMax ("ex-Meta" is about founders' former employer) | ❌ Not found — no such relationship is published (§2.2) |

### 14.4 What Could Not Be Verified

The consolidated note, expanded in §15: this pass could **not** verify a **valuation**, a **take rate or unit economics**, a **profitability figure**, the **ACRA registration number / shareholding**, **funding beyond the seed and Series A**, the **founders' full names first-party**, the **reverse-redemption and partner-expiry mechanics**, or the **exact FlyAnywhere rate first-party**. It could not read a first-party **numeric FlyAnywhere rate** (the 1.8c figure is third-party), and it could not establish a **current user/merchant count** independently of the company's own claims. Tool note: one or two pages (notably the heymax.ai home page and some third-party aggregator pages) would not extract cleanly this pass; that is recorded as a **tool limitation**, not as absence of material — the facts used here come from the cached extracts of the pages that did read.

---

## 15. What Could Not Be Verified: The Honest Ledger

### 15.1 The Ledger

For an early-stage private company, the absences are as informative as the facts. Each of the following is **recorded as the finding** — not filled by inference from comparables, not softened, not corrected by assumption.

| # | Item | Status | Note |
|---|---|---|---|
| 1 | **Valuation** | ✅ not published | No post-money or any valuation appears on HeyMax's pages or the dated press; recorded, not estimated |
| 2 | **Take rate / unit economics** | ✅ not published | The model's structure is public (§5.1–§5.3); the spread, cost and contribution are not |
| 3 | **Profitability** | ✅ not published | No profit or loss figure is published; the guide states only that none is published, and does not characterise the company's financial position |
| 4 | **ACRA registration number / shareholding** | ❌ not read this pass | The entity is a Singapore Pte. Ltd. (§2.1); its registration number and cap table were not captured at a primary source this pass |
| 5 | **Funding beyond seed + Series A** | ✅ not published | Only the two rounds (§2.5) exist in the record; no later round is assumed |
| 6 | **Founders' full names (first-party)** | ❌ not first-party | Only Joe Lu is first-party (§2.3); the other three are attributed to Crunchbase/Preqin/Tenity |
| 7 | **Reverse-redemption / partner-programme expiry mechanics** | ✅ not published | What happens to a transferred Max Mile inside a partner programme (its expiry, redemption, reversal) is not published by HeyMax |
| 8 | **Exact FlyAnywhere rate** | ⚠ third-party only | First-party says only "fixed rate per mile"; the 1.8 cents/mile figure is from Lobang Sis (3 Jun 2026) |
| 9 | **Current user / merchant / miles counts, independently** | ⚠ company claim only | The figures are the company's own, dated; no independent measure was found |
| 10 | **Visa–HeyMax commercial terms, or any Visa endorsement of Max Miles** | ✅ not established | The card-linking integration is verified; nothing more is (§4.5) |
| 11 | **Any reverse data or co-marketing arrangement with an issuer** | ✅ not established | No such arrangement is published (§10.2) |
| 12 | **Internal tech stack beyond Google Cloud** | ✅ not published | The Google Cloud / encryption / deletion statements are the whole of the published stack (§8.2) |
| 13 | **Security certifications / audit attestations** | ✅ not published | None are published on the pages read |
| 14 | **Forward roadmap detail behind the "default protocol" vision** | ✅ not published | The vision line is positioning; the roadmap it implies is not published (§9) |
| 15 | **Statistical corroboration of the reported revenue and GMV target** | ✅ not published | Signed off by no auditor on the public record; treated as company-reported (§14.2) |

### 15.2 What the Absences Mean for a Reader

Three consequences. First, **scale and momentum claims should be read as the company's own, dated, and directionally useful but unaudited** — the guide prints them attributed, and a reader who needs audited figures does not have them. Second, **the economic core of the model is opaque where it matters** — the model's *shape* is admirably public (merchant/partner-funded, "if you don't earn, we don't earn"), but the *magnitudes* (take rate, unit economics, profitability) are not, which is normal for an early-stage private company and is recorded rather than guessed. Third, **the relationship questions a bank would ask (reverse data, campaign cost, endorsement) have no public answers** — so §12 stays at the level of architecture and posture, and the guide does not manufacture a verdict the evidence cannot carry. Where this guide states an absence, the absence is the finding; that is the honest disposition for a subject whose own company pages are the main window onto it.

---

## 16. Glossary, Cross-References and the Closing Summary

### 16.1 The Glossary

| Term | Meaning |
|---|---|
| **HeyMax** | The consumer brand; the platform operated by Max Now Pte. Ltd. (§2.1) |
| **Max Now Pte. Ltd.** | The Singapore private limited company that operates HeyMax (§2.1) |
| **Max Miles** | HeyMax's proprietary reward currency — the company's "universal travel currency you earn from your everyday spend, that never expires" (§3.1) |
| **Rewards aggregator** | A platform whose product is the union of other firms' reward programmes and merchant offers, presented through one currency/account (§1.2) |
| **Universal currency** | A points currency convertible into several unrelated redemption partners — a union, not a single-brand currency (§1.2, §3.1) |
| **Cashback vs miles** | Cashback is money-like, roughly fixed-value; miles are points redeemed inside programmes, worth different amounts by redemption — the distinction behind §6.2 |
| **Card Maximiser** | HeyMax's card-linking feature; tracks the linked card's spending and the *issuer's* rewards, caps and min-spend progress (§4.1) |
| **Card linking** | The consented linking of a card so the platform receives its transaction feed (§4.2–§4.3) |
| **Visa On Platform (VOP)** | The Visa card-enrolment channel through which linking occurs (§4.2) |
| **FlyAnywhere** | HeyMax's redemption feature: booking on nearly any airline at a fixed rate per mile (§3.3) |
| **Miles + Cash** | A redemption route combining Max Miles with cash (§3.3) |
| **1:1 transfer** | The stated ratio at which Max Miles convert into 30+ airline/hotel programmes (§3.3) |
| **Card-linked campaign** | A time-boxed, partner-funded promotion that awards Max Miles for linked-card spend (§4.6) |
| **MCC** | Merchant Category Code — the transaction classification by which card rewards and the Max AI recommendations are keyed (§8.1, §9.2) |
| **Double-dip** | Third-party term for earning a card's own rewards plus Max Miles on one purchase (§6.3) |
| **Max AI / Card Spend AI** | HeyMax's read-only in-app rewards assistant, updated 6 Apr 2026 (§9.2) |
| **Inactivity forfeiture** | The Terms §4 right to remove a user (incl. full balance) after 12 months of non-use (§3.6) |
| **"Default protocol"** | The About-page vision phrase — a positioning statement, not a demonstrated standard (§9.1) |
| **Cymbal Bank** | The repo's fictional Singapore bank persona — design fiction only, used in §1.4 and §12 |

### 16.2 The Cross-Reference Map

**Sibling guides cited in this guide (relative links, same repo):**

- [singapore_saas_companies_guide.md](../technology/singapore_saas_companies_guide.md) — the Singapore software-sector map; §7's ecosystem listing is the boundary this guide respects (§1.6) and the sector context for HeyMax's position.
- [sginnovate_guide.md](sginnovate_guide.md) — the state deep-tech investor; the source of the worked-example, claims-audit and closing-summary formats this guide inherits (§12, §14).
- [tradenet_platform_guide.md](tradenet_platform_guide.md) — the national single window; the provenance and neutral-mechanism conventions mirrored in §4 and §8.
- [starhub_software_systems_guide.md](starhub_software_systems_guide.md) — a Singapore enterprise's software systems; an architectural contrast to a consumer rewards platform (§8).
- **Card- and payments-cluster siblings** — carry the card-scheme, interchange, issuer-reward and payment-rail mechanics that §4 and §10 assume rather than re-derive (§1.6).

### 16.3 Primary Sources Used This Pass

**First-party (HeyMax):** heymax.ai — the About page ("Our Story", the AI-Native-Era vision line), the Terms of Use (last updated 6 February 2025; the §4 inactivity right, §6 nature of Max Miles, §7 reserved-expiry right), the Security page. blog.heymax.ai — "HeyMax announces US$2.6M seed funding round, led by January Capital" (2 July 2024), "How we get you your Max Miles" (28 May 2025), "HeyMax Secures US$11 Million Series A…" (28 January 2026). help.heymax.ai — "Card Maximiser (Singapore users)" (updated 27 February 2026), "Card Maximiser: Security and Privacy" (20 June 2024), "Adding Cards Basics" (20 June 2024), "Max AI" (6 April 2026), "Max Miles Turns 3!" and its campaign T&Cs (updated 22 September 2026). hk.heymax.ai — the Max Miles explainer page ("universal travel currency … that never expires"; the four redemption routes; the FlyAnywhere illustrations).
**Independent press (corroborating):** fintechnews.sg — the seed report (2 July 2024) and the Series A report (28 January 2026); the PR Newswire/company release of 28 January 2026.
**Third-party / registry-derived (for attributed names and opinion):** Crunchbase (legal-name founder card), Preqin (co-founder list), Tenity (accelerator post) for the founder names (§2.3); Lobang Sis, "What is HeyMax? A Complete Beginner's Guide" (3 June 2026) for the "miles version of ShopBack" analogy, "double-dipping" and the 1.8c/mile FlyAnywhere figure; MileLion for campaign coverage (e.g. 29 August 2024).
**Tool note:** a small number of pages (the heymax.ai home page and some third-party aggregator pages) would not extract cleanly this pass; that is recorded as a tool limitation, not as absence of material — the facts used here come from the readable pages above.

### 16.4 The Closing Summary

**The guide in eight lines.** HeyMax is a Singapore consumer rewards aggregator, operated by Max Now Pte. Ltd., founded in 2023 by four former Meta engineers (with Joe Lu the only founder named first-party), that issues a proprietary currency, Max Miles, earned through brand links, vouchers, card-linked campaigns and partner actions, and redeemed by 1:1 transfers into 30+ airline/hotel programmes, FlyAnywhere, Miles + Cash or vouchers. Its funding record is two rounds and no more — a US$2.6M seed led by January Capital (2 July 2024) and an US$11M Series A led by Peak XV (28 January 2026) — with no valuation and no later round published. Its most consequential product promise, "never expires", is a marketing line the Terms of Use qualify: a 12-month inactivity forfeiture right (§4) and a reserved right to introduce expiry (§7). Its card-linking mechanism is a Visa data integration — a Visa-hosted page and a Visa On Platform transaction feed — through which HeyMax tracks **the issuer's** rewards, explicitly disclaiming that it issues them. Its business model is published in shape (merchant/partner-funded; "if you don't earn, we don't earn") and absent in magnitude (no take rate, no unit economics). Its AI positioning is a vision line ("default protocol … AI-Native Era") measured against one read-only, rate-limited, engine-backed chat assistant. The relationship that matters most is the cooperative-and-competitive one with the issuers whose reward value it surfaces and the network whose rails it rides — an arrangement that converges on engagement and diverges on who owns the cardholder's loyalty attention, and one this guide describes without judging. The honest ledger is short and plain: valuation, take rate, profitability, registration, later rounds, first-party founder names and partner terms are simply not public, and the guide records each absence rather than filling it. And the one sentence that holds the whole subject together is the structural truth from which every strength and every fragility in this guide follows:

a rewards aggregator's product is the union of other people's loyalty programmes.
