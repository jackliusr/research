# 1xBet: A Brand, Its Licences and Its Merchant Risk — A Comprehensive Guide to an Online Gambling Operator as a Bank Counterparty

*Companion deep-dive in the banking/ company and sector series of the [jackliusr/research](https://github.com/jackliusr/research) repository — the **operator-layer** counterpart to the supplier-layer [Playtech & Its Competitors](../technology/playtech_competitors_guide.md) guide, sitting alongside the entity-profile guides of the same folder ([Mirae Asset Securities (Singapore)](mirae_asset_securities_singapore_guide.md), [ExodusPoint](exoduspoint_guide.md), [Hudson River Trading](hudson_river_trading_guide.md)), the merchant- and fraud-risk machinery ([Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md), [AML Certifications & Exam Content](aml_certifications_exam_content_guide.md)), the payments-rail map ([Payment Rails](payment_rails_guide.md), [Payments Hub](payments_hub_guide.md)) and the adjacent trading-platform boundary ([Online Investment Trading Platforms](online_investment_trading_platforms_guide.md)). This guide is about **one brand and whoever holds its licences** — 1xBet, the online sportsbook-and-casino brand whose dot-com estate is certified by the Curaçao Gaming Authority as operated by **Caecus N.V.** (Curaçao company number 163779, licence **OGL/2024/1262/0493**, granted 7 November 2024, status Active, verified at the authority's own certificate in this pass) — and deliberately **not** about the gambling-software suppliers, not about the affiliate networks, and not about the individual businesses that trade under neighbouring brand names in the same corporate family. It covers what the regulators' own registers and releases actually say, what the operator says about itself, the group and brand-family picture as far as sources carry it, the dated regulatory record with its conclusions, its open matters and its unverified claims kept strictly apart, the sponsorship machine, and an honest account of what remains unknown.*

**Verification convention used throughout: ✅ = verified in this research pass at the authority's, the sponsor's or the company's own material (with the date it was read); ⚠ = flagged (reported, single-source, dynamic, unverifiable in this pass, or structural inference); ❌ = rejected (a claim this pass found to be contradicted or unsupported); unmarked = structural/industry knowledge presented as such. The consolidated status table is in [§15](#15-the-claims-audit), and the negative findings are collected in [§16](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Gambling-Operator Counterparty Risk — the identity, licensing, regulatory record, marketing machine and merchant-risk assessment of the online gambling brand 1xBet
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides:** [Playtech and Its Competitors](../technology/playtech_competitors_guide.md) (the B2B supplier layer — this guide's nearest neighbour and its explicit boundary), [Gaming Data Warehouse](../technology/data/gaming_dw_bet_recommendation.md) and [Gambling Datasets](../technology/data/gambling_datasets.md) (the data and analytics layer), [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) (a betting operator is not a broker), [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md), [AML Certifications & Exam Content](aml_certifications_exam_content_guide.md) (the fraud and merchant-risk machinery), [Payment Rails](payment_rails_guide.md), [Payments Hub](payments_hub_guide.md), [Mirae Asset Securities (Singapore)](mirae_asset_securities_singapore_guide.md), [ExodusPoint](exoduspoint_guide.md), [Hudson River Trading](hudson_river_trading_guide.md)

---

## Table of Contents

1. [The Overview, the Identity Gate, the Decoder and the Boundary](#1-the-overview-the-identity-gate-the-decoder-and-the-boundary)
2. [The Identity and the Group Structure](#2-the-identity-and-the-group-structure)
3. [The History](#3-the-history)
4. [The Business](#4-the-business)
5. [The Licensing Position](#5-the-licensing-position)
6. [The Regulatory Record](#6-the-regulatory-record)
7. [The Sponsorships and the Legitimacy Question](#7-the-sponsorships-and-the-legitimacy-question)
8. [The Affiliate and Marketing Model](#8-the-affiliate-and-marketing-model)
9. [The Bank's View — the Merchant and Payment-Counterparty Assessment](#9-the-banks-view--the-merchant-and-payment-counterparty-assessment)
10. [The AML and Payment-Rails Angle](#10-the-aml-and-payment-rails-angle)
11. [The Comparison Set](#11-the-comparison-set)
12. [The Data and Analytics Angle](#12-the-data-and-analytics-angle)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Identity Gate, the Decoder and the Boundary

### 1.1 The thesis in one line

**The licence is the identity.** For an online gambling operator, the question "who is this?" and the question "who holds the licence?" are the same question — and for 1xBet they have different answers in different markets, because 1xBet is a **brand**, not a legal person. Everything else in this guide is elaboration of that sentence, plus an honest account of what could not be established at source.

That sentence is not a rhetorical posture. It is the operational consequence of a specific verified fact: the Curaçao Gaming Authority's own certificate states that **1xbet.com is operated by Caecus N.V.**, a Curaçao company, under a licence in Caecus N.V.'s name ✅. The brand is 1xBet. The licensee is Caecus N.V. A bank that onboards "1xBet" has onboarded neither.

### 1.2 The single most important distinction in this guide

Five things share the words "1xBet" or something close to them, and they are not interchangeable:

| # | What it is | What it is not | Identifier | Status |
|---|---|---|---|---|
| (i) | **1xBet** — the brand, marketed across 1xbet.com and a dot-com estate | not a legal person; it holds nothing | brand; no registry entry | ✅ (as a brand) |
| (ii) | **Caecus N.V.** — the Curaçao company named by the Curaçao Gaming Authority as the operator of 1xbet.com | not the whole group; not the Russian brand; not the Irish licensee | Curaçao company no. **163779**; licence **OGL/2024/1262/0493** (granted 7 Nov 2024, Active) | ✅ Curaçao Gaming Authority certificate |
| (iii) | **The Russian-facing brand** (1xStavka / 1хСтавка) — a separately licensed Russian bookmaker that Russian sources and the international press associate with 1xBet in different and contradictory terms | **not** the dot-com operation, and one Russian-facing source states it has no legal relationship to it | FNS bookmaker licence number is reported by Russian aggregators; **not verified at the FNS register in this pass** | ⚠ reported; relationship disputed between sources |
| (iv) | **The related brand family** — the cluster of betting brands widely described as sharing origins, staff or infrastructure with this operation | not established by any source read at register level in this pass | — | ⚠ see §2.3; association only where a source states it |
| (v) | **Terminus Platform Ireland Limited** — named in the encyclopaedia record as the operator of the 1xBet brand in Ireland under an Irish remote bookmaker's licence | not the Curaçao licensee; not verified at the Irish register in this pass | Remote Bookmaker's Licence No. 1019276 (as recorded by the encyclopaedia) | ⚠ encyclopaedia-sourced, not register-verified |

This guide exists in part to prevent the conflation of (i) with (ii), of (ii) with (iii), and of any of them with the marketing entities, affiliate entities and local franchisees that a gambling brand's payments and advertising actually flow through. [§2](#2-the-identity-and-the-group-structure) does the work; [§13](#13-the-anti-patterns) turns it into guardrails.

### 1.3 What this guide covers

- **[§2 The Identity and the Group Structure](#2-the-identity-and-the-group-structure)** — the licensee as the authority's own certificate names it, the brand-family question handled as sourced-or-not, the ownership question recorded as established-or-not, and a blunt list of what the identity gate does not establish.
- **[§3–§4 The History, the Business](#3-the-history)** — the founding narrative with its sourcing limits, the offshore move and the market exits, then the product set at the level the operator describes it and the structural economics of an online operator.
- **[§5 The Licensing Position](#5-the-licensing-position)** — the analytical core: what is held, from whom, of what class, for what scope, verified at registers or recorded as absent; and the jurisdictional-scope point developed as the transferable finding.
- **[§6 The Regulatory Record](#6-the-regulatory-record)** — a dated, register-style record with concluded, ongoing and reported-but-unverified matters in **separate** groups.
- **[§7–§8 The Sponsorships, the Affiliate and Marketing Model](#7-the-sponsorships-and-the-legitimacy-question)** — the sponsorship machine with terminations dated and reasons attributed, and how an online operator acquires customers through the affiliate layer.
- **[§9–§10 The Bank's View, the AML and Payment-Rails Angle](#9-the-banks-view--the-merchant-and-payment-counterparty-assessment)** — the payoff: high-risk-merchant classification, the deposit/withdrawal rail mix, the customer-jurisdiction problem, and the financial-crime profile cross-referenced rather than re-derived.
- **[§11–§12 The Comparison Set, the Data Angle](#11-the-comparison-set)** — the operator landscape described by **tier and licence class**, never by this guide's own ranking of named competitors; and the pointer to the repo's gambling-data guides.
- **[§13–§14 The Anti-Patterns, the Worked Example](#13-the-anti-patterns)** — symptom, cause and guardrail; then the fictional Cymbal Bank case, in which the reasoning turns on the licence and the customer-jurisdiction mismatch and not on the character of the business or the people in it.
- **[§15–§16 The Claims Audit, the Negative Findings](#15-the-claims-audit)** — every claim with source, date and quality, regulatory and licensing items in a separate block; the explicit list of what could not be established; the glossary, cross-references and closing summary.

### 1.4 The decoder

The vocabulary below is used throughout. Where a term has a regulatory meaning, that meaning is the one intended.

- **The operator** — the business that holds the gambling licence, takes the customer's stake into its own books, owes the customer the payout, and carries the regulatory risk for how that customer was acquired, verified and treated. This guide is about an operator. The **supplier** — the platform, the odds engine, the game studio — sells technology to the operator and is a different layer (see [§1.6](#16-the-boundary-declared-by-name)).
- **The licence and the licensing jurisdiction** — a gambling licence is granted by one authority in one jurisdiction, for one class of activity, to one legal person. It authorises activity **there**. It does not travel. A Curaçao licence does not authorise the serving of a Dutch, British, Spanish, French or Nigerian customer any more than an Irish licence does. It is the *destination market's* regulator — never this guide — that determines whether an operator may lawfully serve that market's customers.
- **The licence class** — licences are not a ladder of quality from bad to good; they are different instruments for different activities, with different capital, reporting, player-protection and AML conditions attached. Distinguish at least: a **national remote-gambling licence** (the destination market's own instrument); an **offshore/transitional licensing jurisdiction's** remote licence (granted by a jurisdiction that is not the customer's home market); a **B2B supply licence**; and a **white-label arrangement**, in which a brand rides on another entity's licence and is therefore *not itself licensed at all* for that market.
- **The brand family** — a set of brand names that share a common origin, ownership, staff, platform or marketing infrastructure without being the same legal person. This guide asserts a brand-family relationship only where a named source states it, and says which source ([§2.3](#23-the-brand-family)) — otherwise it records the relationship as not established.
- **The merchant of record** — the legal entity that appears on the acquiring/banking side of the transaction, contracts with the acquirer, and receives settlement. It is not the brand, and it is frequently not the entity whose licence is displayed on the website. Identifying it is a document-level exercise, not a website-reading exercise.
- **The affiliate** — a third-party marketer paid per referred depositing customer. Affiliates are how the online sector acquires customers; they are also the layer through which advertising reaches markets, and audiences, that the operator's own licence does not cover ([§8](#8-the-affiliate-and-marketing-model)).
- **The deposit and withdrawal rails** — the payment methods by which a customer funds an account and withdraws winnings: cards, e-wallets, account-to-account/open-banking, instant-bank-transfer, prepaid vouchers, local schemes and, in some markets, crypto. The mix is a risk variable in its own right ([§10](#10-the-aml-and-payment-rails-angle)).
- **The grey market and the black market** — usage in the sector is loose and this guide uses it precisely. **Grey**: a market where the operator serves customers without a licence but where the activity is not formally branded unlawful by that market's authority; **black**: a market where the authority has publicly determined the activity to be unlicensed or unlawful, or where local law prohibits it. **Only the authority of the market concerned can make that determination**; where this guide records it, it attributes it to that authority ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)).
- **The sponsored-legitimacy signal** — a top-tier club or tournament sponsorship, read by the operator as evidence of respectability and read by a bank as evidence of marketing spend, not of regulatory standing ([§7](#7-the-sponsorships-and-the-legitimacy-question)).

### 1.5 The legal-risk discipline this guide is written under

This is a real company in a heavily regulated industry that litigates, and parts of its public record consist of allegations, investigations and administrative actions issued by bodies in several jurisdictions. The following rules are not caveats bolted onto the text; they are the method, and they are applied line by line throughout.

**(i) Every regulatory, legal or licensing claim is verified at the authority's own release, order, register or docket — or it is flagged and not stated as fact.** Each such claim in this guide carries the authority, the instrument, the date, the parties and the outcome or status, or it carries an explicit "unverified" flag. Where the authority's own material could not be reached in this pass, the guide says so and does not substitute a press rendering.

**(ii) An allegation is reported as an allegation, attributed to whoever made it, never restated as a finding, and never in this guide's own voice.** The construction "X is reported by Y to…" is used, not "X is…".

**(iii) No characterisation the source does not make.** This guide never describes the company, any entity in its family, or any person as illegal, criminal, fraudulent, sanctionable or "banned" on its own authority. Where an authority has used such words, the words are quoted to that authority and no further.

**(iv) Ongoing, concluded and unreported matters are distinguished explicitly.** A pending matter is not a finding; an unsourced matter is not a matter. Where a matter's status cannot be established, the guide says "status not established".

**(v) No private individual is named in connection with an allegation unless a regulator, a court or a legislature has publicly done so — and then only in the exact terms that body used.** A practical consequence, applied throughout: individuals named by press reporting in connection with ownership or alleged conduct are **not** named in this guide where the sourcing chain is journalistic rather than an authority's own public act. This is a deliberate choice, and it is stated here so the reader knows an omission is a rule being applied and not an oversight.

**(vi) No invented case number, penalty amount, raid, investigation, licence number, licence date or outcome.** Where a figure or a reference appears, it is a figure or reference that a named source published, reproduced verbatim.

**(vii) Press-only claims carry the outlet and the date and are labelled as reported.** Encyclopaedia entries are treated as *press round-ups*, not as sources: where this guide relies on one, it says so and names the underlying publication that the entry cites, and marks the item ⚠.

**Where the correct output is an absence, the absence is the finding.** A register that does not list an entity, a regulator that declines to confirm an action, a licence that cannot be found for a market — these are recorded as the **negative results of dated searches**, with the register, the date and the search performed. An absence proves nothing by itself; it is recorded because a documented negative is the only honest way to hold a gap, and because in this guide's subject matter the gaps are the material.

### 1.6 The boundary, declared by name

The repository covers the gambling industry's **supplier layer**, not its **operator layer**. This guide is the operator-layer document, and it owns exactly one thing: **this operator** — its identity, history, business, licensing position, regulatory record, sponsorship and marketing model, and the bank's assessment of it. The neighbours below own their material and are cross-referenced, not re-derived.

- **[Playtech and Its Competitors](../technology/playtech_competitors_guide.md)** owns the **B2B gambling software and platform-supplier landscape** — the platform, content and live-casino vendors, the MGA/UKGC supplier-licensing frame, and the supplier-side banking angle. **A supplier sells the technology; it is not the operator.** This guide is about the layer that holds the licence, takes the customer's money and carries the regulatory risk. Where the supplier layer matters here (the white-label question in §5, the platform dependency in §9), it is cross-referenced to that guide; its competitor tables are not reproduced.
- **[Gambling Datasets](../technology/data/gambling_datasets.md)** and **[Gaming Data Warehouse & Bet Recommendation](../technology/data/gaming_dw_bet_recommendation.md)** own the **gambling data and analytics** material — the player/bet/wallet data model, RTP and recommendation analytics, and the KYC flags inside those models. §12 points at them; it does not re-derive them.
- **[Online Investment Trading Platforms](online_investment_trading_platforms_guide.md)** owns **investment and broking platforms**. **A betting operator is not a broker.** The customer's relationship to the money is different (a stake is not a client asset; winnings are not a custody balance), the conduct regime is different, and the licensing authorities are different. The comparison appears once, in §11, and is not developed.
- **[Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)**, **[AML Certifications & Exam Content](aml_certifications_exam_content_guide.md)** and the material behind the **merchant-category** treatment of gambling (MCC 7995 in the card networks' own scheme rules) own the **fraud and merchant-risk machinery**. §9 and §10 use that machinery and cross-reference it; they do not re-teach it.
- **A declaration of what this guide does *not* have**: no proprietary or leaked data, no confidential filings, no non-public register extracts, no financial statements of any entity in the family. Everything here is public-record, retrieved in this pass, with the retrieval dated.

### 1.7 What was verifiable, in one paragraph

The brand is real, widely marketed, and currently operating: the **Curaçao Gaming Authority's own certificate** names **Caecus N.V.** (Curaçao company number 163779) as the operator of 1xbet.com under licence **OGL/2024/1262/0493**, granted **7 November 2024**, status **Active** ✅ — a positive, register-grade licensing fact and the single most important item in this guide. The Curaçao statute the certificate cites is named in the certificate itself: the **National Ordinance on Games of Chance** (*Landsverordening op de kansspelen*, P.B. 2024, no. 157) ✅. The **Great Britain** position is a **sourced negative**: the Gambling Commission's published response to a January 2024 information request about "the licence and subsequent suspension and possible ban on the betting and gambling brand 1xBet" **refused to confirm or deny whether it holds any information at all**, citing section 31(3) of the Freedom of Information Act 2000 ✅ — so the widely repeated claim that a British licence was "revoked" is **not** established by the regulator's own record, and is treated in this guide as unattributed characterisation unless an instrument can be produced ([§5.3](#53-great-britain-a-licence-that-was-not-there)). **Sponsorship** is the best-documented part of the public record because the sponsors themselves published the deals: FC Barcelona announced 1xBet as a Global Partner on **3 July 2019** ✅ and renewed it on **1 July 2024** through **June 2029** ✅, both at fcbarcelona.com; the **2019** suspensions and terminations by Liverpool, Chelsea and Tottenham Hotspur are reported by the gambling trade press with the Gambling Commission's own quoted reasoning ✅⚠. What is **not** established: the **ownership and control** of Caecus N.V. and of the wider family beyond what a licensing authority's certificate states; the **full corporate map**; **financial statements** of any entity; the **merchant of record** for any given market; and the **status of most alleged matters** in most jurisdictions. Sections 2, 6 and 16 carry those gaps explicitly.

---

## 2. The Identity and the Group Structure

*This is the guide's first real section, and it follows the identity-gate pattern of [ExodusPoint §2](exoduspoint_guide.md) and [Mirae Asset Securities (Singapore) §2](mirae_asset_securities_singapore_guide.md): establish the legal entities at the regulators' own records, dated, before any other claim — and then state plainly what the gate does not establish.*

### 2.1 The brand is not an entity — the register entry that names the licensee

The cleanest single fact available about this brand is also the least-known one, because it sits on the **licensing authority's own certificate** rather than in any news article. The Curaçao Gaming Authority's license register portal serves a per-brand **Certificate of Operation** (retrieved in this research pass, September 2026). Its operative text reads ✅:

> "This is to certify that **1xbet.com** is operated by **Caecus N.V.**, a company incorporated under the laws of Curaçao with Company Number **163779** and licensed by the Curaçao Gaming Authority to offer games of chance under license number **OGL/2024/1262/0493** in accordance with the National Ordinance on Games of Chance (Landsverordening op de kansspelen, P.B. 2024, no. 157)."

> "The license was granted on **07/11/2024** and its current status is **Active**."

Three things about that paragraph matter more than the rest.

1. **The operator is a named legal person, and the brand is not.** A bank's file on "1xBet" that does not begin with "Caecus N.V., Curaçao company number 163779" has not begun. Where a market requires a locally licensed entity, the licensee will be a *different* legal person again — which is the entire point of §5.
2. **The licence is of a class and a date.** It is a **B2C online gaming licence issued under the post-2024 Curaçao statute**, granted **7 November 2024**, and it is **Active** as at the retrieval date. It is not a master licence, not a sub-licence under an older regime, and not a claim on a website — it is an authority-issued certificate naming an entity, a company number, a licence number and a grant date ✅.
3. **The same certificate series covers more than one domain.** A second certificate in the same series names the same licensee, the same company number and the same licence for the domain **1x-bet.com** ✅, which is a concrete illustration of a banking-relevant point: a brand's domain estate is not the same thing as its licensed perimeter, and a bank onboarding on the basis of one domain has not mapped the others.

### 2.2 The entity-and-brand map, with the source for each relationship

The table below is the guide's identity map. **Every row carries the source that states the relationship; where no source establishes a relationship, the row says so rather than inferring it.**

| # | Name | What it is | The source that establishes it | Status |
|---|---|---|---|---|
| 1 | **1xBet** | The brand; the customer-facing trading name across 1xbet.com and a multi-domain, multi-language estate | The brand's own site and the Curaçao Gaming Authority certificate, which names "1xbet.com" as the operated domain | ✅ brand confirmed; no registry entry (a brand cannot have one) |
| 2 | **Caecus N.V.** | Curaçao-incorporated company; the **operator of 1xbet.com** and the holder of the Curaçao licence | **Curaçao Gaming Authority Certificate of Operation** (cert.gcb.cw), retrieved September 2026: operator, company number 163779, licence OGL/2024/1262/0493, granted 07/11/2024, Active | ✅ authority certificate |
| 3 | **Caecus N.V.'s ownership and control** | Who owns the licensee | No source read in this pass establishes the ownership chain. Journalistic reporting (Follow the Money, 2026, via trade-press summaries) describes Curaçao Gaming Authority assessment files naming an individual as owner and CEO and recording assessors' suspicion of undisclosed additional owners; under this guide's rule (v), **the individual is not named here** because the naming is journalistic, not an act of a regulator or court that this pass could read | ❌ **not established**; the reporting is ⚠ and treated as an allegation |
| 4 | **Terminus Platform Ireland Limited** | Named in the encyclopaedia record as the operator of the 1xBet brand in Ireland, trading as "1XBET", at a Cork address, under an Irish remote bookmaker's licence (No. 1019276 recorded) | Encyclopaedia entry (Wikipedia, "1xBet", retrieved September 2026), citing the Irish licence entry; **the Irish register was not reached in this pass** | ⚠ reported, **not register-verified** |
| 5 | **The Russian-facing brand 1xStavka / 1хСтавка** | A Russian bookmaker bearing a Russian licence, associated in secondary and press sources with 1xBet | Russian-language aggregators report an FNS bookmaker licence with a number and a 2010 grant date (dates given inconsistently); one Russian-language aggregator states explicitly that it has no legal relationship to 1xBet; the international encyclopaedia record treats it as the "Russian version" | ⚠ **contradictory across sources**; FNS register not reachable in this pass (§5.5) |
| 6 | **The related brand family** (other betting brand names widely described as sharing an origin, staff or infrastructure with this operation) | Related brands, each a separate legal person with its own licensee and its own licensing footprint | **No source read in this pass establishes the relationships at register level.** Where the operator's own material or sponsors' material names partner brands, that is a marketing statement and is treated as such | ❌ **not established** at register level (§2.3) |
| 7 | **The marketing/affiliate entities** | The affiliates, sub-affiliates and local marketing companies that carry acquisition | Not researched to register level in this pass; a bank's own diligence, not a public register, is where these are established | ❌ not established here |
| 8 | **The merchant of record per market** | The entity that contracts with acquirers and receives settlement | Not public in any source read; it is a document-level fact obtainable only from the merchant and its acquirer | ❌ not established here |

**How to read row 2 against row 4**: the same brand carries a **Curaçao** licence held by Caecus N.V. and, per a non-register source, an **Irish** licence held by a differently-named Irish company. That is not an inconsistency; it is the normal architecture of a multi-market online operator, and it is exactly why the guide's first question is "which entity, and licensed where?" rather than "which brand?".

### 2.3 The brand family

A family of related betting brands is publicly associated with this operation. The guide's discipline on that association is as follows, and it is worth stating as a rule because it governs several later sections:

- The guide states a brand-family relationship **only where a named source states it**, and names that source in the same sentence.
- The guide does **not** treat "same platform", "same odds feed", "same payment pages", "same affiliate programme", "same design", "shared customer support" or "similar name" as evidence of common ownership. Those are exactly the inferences this guide is written to refuse.
- Where a relationship is reported by a **co-regulator's public act** — an authority's decision, a published enforcement instrument, a court's judgment — that is register-grade and is recorded in [§6](#6-the-regulatory-record). Where it is journalistic or commercial, it is recorded here as **reported**, with the outlet and date, or not at all.
- The practical consequence for this pass: **the family map is a gap, not a finding.** Aggregator and affiliate-industry descriptions of which brands sit together exist in quantity and are, by their nature, unverifiable; this guide does not reproduce them.

What *can* be said, and is said, is narrower and more useful: the operator's own announcements and the sponsors' own announcements name **brands and rightsholders** with which the brand claims partnerships (FC Barcelona, Paris Saint-Germain, LOSC Lille, Italian Serie A and the Confederation of African Football appear in the company-supplied "About 1XBET" text carried inside FC Barcelona's July 2024 announcement ✅ — a **company claim reproduced by a sponsor**, which is a marketing statement and not a licensing or ownership fact). That is a partner list. It is not a corporate-structure finding, and the two must not be conflated.

### 2.4 The corporate-structure question, stated honestly

The differences between these five questions are the substance of a bank's onboarding file, and only the first has a register-grade answer in this pass:

| Question | Answer available | Status |
|---|---|---|
| Who holds the licence under which 1xbet.com operates? | Caecus N.V., Curaçao company number 163779, licence OGL/2024/1262/0493 | ✅ authority certificate |
| Who owns and controls Caecus N.V.? | Not established at source in this pass | ❌ |
| What is the parent company of Caecus N.V., and what is the group's ultimate beneficial ownership? | Not established at source in this pass | ❌ |
| Which entities sit in the family across markets, and how are they related? | Not established at register level; sources conflict | ❌ |
| Which entity is the merchant of record for a given market's card and bank settlement? | Not public; obtainable from the merchant and its acquirer | ❌ |

A diligence file at this point has **one** solid corporate fact and **four** open questions. That is a normal starting position for a gambling-operator counterparty and it is a materially different position from "no information", which is how this pass's negative findings on the *other four* questions should be read.

### 2.5 What the identity gate establishes, in one line each

- The brand is 1xBet; the dot-com operator is **Caecus N.V.** ✅ Caecus N.V. is a **Curaçao** company with company number **163779** ✅.
- Caecus N.V. holds Curaçao online gaming licence **OGL/2024/1262/0493**, granted **7 November 2024**, **Active** as at this pass ✅.
- The licence is issued under the **National Ordinance on Games of Chance** (*Landsverordening op de kansspelen*, P.B. 2024, no. 157) ✅.
- The certificate series covers more than one domain, and the same licensee/licence covers at least **1xbet.com** and **1x-bet.com** ✅.
- The licensing authority operates a **public certificate/register portal** through which a counterparty can verify these facts themselves ✅ — this is the guide's recommended verification route for any bank screening the brand ([§9.5](#95-the-verification-and-monitoring-checklist)).

### 2.6 What could not be established about this identity — bluntly

Stated once here and again in [§16](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary):

1. **Ownership and control of Caecus N.V.** — not established at source. Journalistic reporting of a licensing file describes ownership questions; this guide does not restate them as findings, and does not name the individual the reporting names.
2. **The parent and intermediate holding structure** — not established.
3. **The full list of entities in the family and their relationships** — not established at register level; where sources conflict, the conflict is recorded rather than resolved.
4. **Whether the brand holds a licence in the markets it serves**, other than where a market's own register was reached — for most markets, not established in this pass (§5).
5. **The merchant of record** — not public.
6. **Financial statements, revenue, customer numbers, employee numbers** for any entity in the family — not public. The only figures in this guide are figures a named source published, each labelled ([§4.4](#44-performance-claims-the-company-makes-about-itself)).
7. **The Irish licence** — reported by an encyclopaedia record, **not verified at the Irish register** in this pass.
8. **The Russian licence and the Russian entity** — reported inconsistently; the FNS register was **not reachable** in this pass.

---

## 3. The History

*The history below is written under the guide's sourcing rules: dated facts where a source carries a date, press claims labelled with the outlet and labelled as reported, and no rendering of an allegation as a finding. Where a date is contested between sources, both are shown.*

### 3.1 Founding and origin market — what sources say, and how well they say it

The founding account available to this pass is **entirely secondary**, and the guide says so before repeating any of it.

| Claim | Source and date | Status |
| --- | --- | --- |
| "**1xBet** is an online gambling company founded in **2007**" | Encyclopaedia entry, "1xBet", retrieved September 2026 | ⚠ encyclopaedia; the founding year **2007** is repeated by the operator's own narrative ("more than 12 years of experience" in July 2019 ✅, and "17 years of experience" in July 2024 ✅ — both consistent with a 2007 origin) |
| Founded **in Russia** | Encyclopaedia entry; Bellingcat, 21 October 2024 ("a Cyprus-based bookmaker") | ⚠ |
| Headquarters **Limassol, Cyprus** | Encyclopaedia infobox | ⚠ encyclopaedia; Bellingcat's 2024 piece describes the operator as "Cyprus-based" ⚠ press |
| The company operates a **franchise business model** | Encyclopaedia entry | ⚠ unverified in this pass; if true it is structurally important (§8), and the guide records it as reported only |
| The dot-com estate is described in the operator's own material as having offices in **Europe, Asia and Latin America** and "over 5,000 professionals" | The company's own "About 1XBET" text, as carried in FC Barcelona's announcement of 3 July 2019 ✅ | ✅ as the company's own claim (2019 vintage); not a verified headcount |

**The date the guide will use.** The only dates in this guide's own voice are the ones a sponsor or an authority published. For the founding, the guide uses **"reported as 2007"** and does not assert it.

### 3.2 The two-entity origin — and the naming trap that follows

The earliest licensing-stage entities that appear in the *authority-issued* record are **1X Corp N.V.** and **Exinvest Limited** — the two companies the Dutch gambling authority found, in a decision of 4 January 2019, had operated the websites **1xbet.com** and **xbet-1.com** in contravention of the Netherlands' *Wet op de kansspelen* ✅ ([§6.1](#61-concluded-matters)). The later Curaçao authority certificate names **Caecus N.V.** as the operator of 1xbet.com ✅ (§2.1).

Two lessons follow, and they are the reason this subsection sits in the history rather than in a footnote:

1. **The name behind the licence changes over time.** A bank's file that records "1X Corp N.V." as the 1xBet entity is recording a *point-in-time* fact that the 2019 Dutch instruments support and that the 2024 Curaçao certificate does not carry forward. Both can be true of different periods; only the current register tells a bank which entity is live today. This is precisely why the guide insists on a **dated** register reading rather than a name.
2. **A second company appears alongside the first.** In the Dutch matter, the two named parties were a Curaçao holding-type company (1X Corp N.V.) and a second company (Exinvest Limited), and the authority's own advisory committee recorded that the second was described in the investigation as a **"factureringsagent"** (*billing agent*) ⚠ — a description quoted from the Ksa file, and exactly the kind of operational/merchant-side entity a bank has to identify separately from the licensee. The guide does **not** infer what that company does today; the point is that the separation of licence-holder from billing/processing entity is visible in the public record as early as 2018–2019.

### 3.3 The offshore move, and what "offshore" means here

"Offshore" in this sector is not a euphemism and not necessarily an impropriety: it is the ordinary consequence of a licensing model in which a jurisdiction grants a remote-gambling licence that is not the customer's home jurisdiction. For this operation the offshore anchor is documented: the **Curaçao** licence held by Caecus N.V. ✅, and the earlier Curaçao-domiciled entities in the Dutch instruments ✅, and the Ukraine sanctions record's description of 1XCorp N.V. as registered in **Curaçao** with a Willemstad address ✅.

The guide records the *mechanics* and refuses the *motive*: what a bank needs to know is that the licence under which the dot-com estate operates is a Curaçao one, that Curaçao is not the home market of most of the customers the brand serves, and that the consequence of that structure is jurisdictional (§5).

### 3.4 Documented exits from, or exclusions in, markets — the dated list

Exits and exclusions are the part of a gambling operator's history a bank actually reads. The list below is dated and attributed. **Note the three columns' different epistemic status: only the middle column is register-grade.**

| Market | What happened | Source, with date | Status |
| --- | --- | --- | --- |
| **Netherlands** | Websites operated by 1X Corp N.V. and Exinvest Limited investigated Feb–Jul 2018; administrative fine imposed 4 Jan 2019; objections decided 7 May 2019; the authority later began recovery proceedings and the outstanding amount was reported unpaid | Ksa decision documents on the authority's own site (`kansspelautoriteit.nl`), 26 April 2019 and 7 May 2019 ✅; recovery and the authority's "strong indication that the provider is unreliable" remark reported by Dutch/Curaçao press ⚠ | ✅ concluded authority matter + ⚠ press |
| **Great Britain** | A 1xBet-branded UK-facing site was taken offline in August 2019 following a *Sunday Times* investigation; the clubs sponsoring the brand suspended or ended their deals; in January 2024 the Gambling Commission **declined to confirm or deny** whether it held any information about "the licence and subsequent suspension and possible ban on … 1xBet" | Gambling Commission published FOI response, request dated 16 January 2024, outcome "Information withheld" (s31(3) FOIA) ✅; the club severances reported by Bellingcat 21 October 2024 and by the gambling trade press, 2019 ⚠ | ⚠ **no instrument found**; see §5.3 and §15.3 |
| **Russia** | Reported as prohibited from operating in Russia; the Russian-facing brand is a separately licensed Russian bookmaker; Russian-language sources disagree about the relationship between the two | Bellingcat, 21 October 2024 ("prohibited from operating in Russia") ⚠; Russian-language aggregators (September 2026) ⚠; **FNS register not reachable in this pass** | ⚠ reported; register gap (§5.5) |
| **Ukraine** | A sanctions instrument of 10 March 2023 designates entities connected to the betting and lottery business, including **1XCorp N.V.** | Presidential Decree No. 145/2023 of 10 March 2023 enacting the NSDC decision of the same date, as reported by the Ukrainian national news agency (Ukrinform) and as cited in a sanctions database entry for 1XCorp N.V. ✅⚠ — the decree's annex was **not read at source** in this pass | ⚠ instrument identified, listing not read at source (§6.1) |
| **Liberia** | An administrative tribunal of the National Lottery Authority found the local licensee liable for, among other things, permitting an unlicensed foreign entity to operate under its licence, revoked that licence, and **dismissed the complaint against "1 X BET" because a brand is not a legal entity** | NLA Administrative Hearing, *NLA v. LIPAY, Inc. and the Management of 1 X Bet*, Ruling and Judgment dated 22 June 2026, published by the NLA ✅ | ✅ concluded authority matter (§6.1.2) |
| **Morocco** | Reported mass investigation by the National Judicial Police Brigade following a complaint by a Moroccan operator (2023) | Encyclopaedia entry citing Bellingcat, 21 October 2024 ⚠ | ⚠ reported; status not established |
| **Nepal, Sri Lanka, Morocco (blocking)** | Reported national site-blocking measures naming the brand among others | Trade press and news aggregators, 2026 ⚠ | ⚠ reported only; not authority-verified here |

**An honest note on this table.** It is deliberately short on elegant chronology and long on provenance, because the alternative — a smooth narrative of a company "banned in many countries" — is exactly the construction this guide's rules forbid. Several of the items above are single-source and none of the four "reported" rows should be read as a finding.

### 3.5 The ownership narrative — recorded as reported, not as fact

Journalistic investigations have published accounts of the group's ownership and of regulatory files relating to it: Follow the Money (20 June 2023, "Crypto billions and illegal gambling sites: 'Russian' 1xBet conquers the world from Curaçao") ⚠; Follow the Money again in 2026 with reporting on Curaçao Gaming Authority assessment files ⚠; Bellingcat with Josimar (21 October 2024) ⚠; Josimar (20 January 2023) ⚠.

The guide's treatment of that body of reporting is as follows, and it is a rule applied consistently:

- **It is reported, with the outlet and the date, and it is not restated as fact.**
- **No individual is named here** in connection with any allegation or ownership claim, because the naming in the sources this pass could reach is journalistic. The single exception the rules allow — a regulator, a court or a legislature having publicly named the person — was not established at source in this pass, so the exception is not used. The guide states this explicitly so that the omission is understood as a **rule**, not as a gap in the research. ⚠
- **The corporate structure is not established** by any of this reporting in a form a bank could file: no parent company, no shareholding percentages, no ultimate beneficial ownership. That is the finding (§2.6).

---

## 4. The Business

### 4.1 The product set, at the level the operator itself describes it

An online gambling operator's product set is not exotic, and the correct description is the operator's own. The brand's own site (1xbet.com, retrieved September 2026) lists, across its navigation ✅:

- **Sports betting** — pre-match ("line") and in-play ("live") markets, including a "Multi-LIVE" view, long-term bets, and sport-specific hubs for football, cricket and others ✅.
- **Esports** ✅.
- **Casino and games** — "Live Casino", "Slots", "1xGames" (the operator's own instant/quick games), bingo, scratch cards, lotto, TV games and virtual sports ✅.
- **Pool betting / toto** ✅.
- **Statistics and results** services alongside the betting product ✅.

The company's own description of its scale and reach, as carried in a sponsor's announcement (FC Barcelona, 3 July 2019 and 1 July 2024) ✅: an international gaming and technology company with offices in Europe, Asia and Latin America; markets in pre-match and live odds; online slots, live casino and table games; **"accepts more than 250 payment solutions"** (2019 text) and a website and app **"available in 70 languages"** (2024 text); around-the-clock support in 30 languages (2019 text). **These are the company's own claims, reproduced by its own sponsor** ✅ — labelled as claims, not verified, and dated by the announcement that carries them.

### 4.2 The customer-acquisition model

The mechanism is described structurally here and in detail in [§8](#8-the-affiliate-and-marketing-model); the business-model point belongs here.

An online betting operator acquires a customer in essentially four ways: paid media (search, social, display, streaming and broadcast); **affiliates** (third-party publishers paid per referred depositing customer); **sponsorship and brand ambassadors** (the legitimacy route, [§7](#7-the-sponsorships-and-the-legitimacy-question)); and **local franchisees or licence partners** who hold the local licence and operate the brand locally. The operator's own material describes the fourth explicitly — the encyclopaedia record describes a **franchise business model** ⚠ and the Liberian ruling documents, in an authority's own words, a **trademark licence agreement** by which a local company acquired "the right to use the 1XBET trademarks in Liberia" for a monthly royalty ✅ ([§6.2](#61-concluded-matters)).

For a bank, that fourth channel is the one that matters most, because it is the channel in which the **brand** and the **licensed entity** diverge most sharply — and it is the channel that produced the guide's single best-sourced identity fact.

### 4.3 The structural economics of an online operator

The economics of this business are the economics of any high-churn digital merchant, with three sector-specific overlays. The description below is **structural** (unmarked), not sourced to this company, because no company in this family publishes accounts.

| Line | What drives it | Banking relevance |
| --- | --- | --- |
| **Gross gaming revenue (GGR)** | stakes less winnings paid; the operator's margin across sports (a bookmaker's overround) and casino (house edge less bonuses) | The revenue base a bank would size exposure against — **not published by this family** |
| **Bonusing and promotions** | the dominant customer-acquisition cost; drives multi-accounting and bonus abuse | A dispute and chargeback source ([§9.3](#93-the-merchant-risk-classification-and-why-it-applies)) |
| **Marketing** | sponsorship, affiliates, paid media; the largest discretionary line | Where the "legitimacy purchase" is booked ([§7.4](#74-what-sponsorship-buys--the-guides-own-analysis-labelled)) |
| **Payment costs** | many rails, many markets, high transaction count, high decline and reversal rates | The reason a bank is being asked to help at all ([§9.2](#92-the-deposit-and-withdrawal-rail-mix)) |
| **Licensing and compliance** | per-market licences, local entities, local reporting, player-protection obligations | A fixed cost that rises with the number of markets; a barrier that favours scale |
| **Payouts** | winnings paid out; the operator's own liability to the customer | The reputational and conduct risk that shows up as complaints and, sometimes, as unpaid winnings allegations ([§6](#6-the-regulatory-record)) |

The structural consequence a bank should carry away: an online operator's cost base is **front-loaded into customer acquisition and payments**, its revenue is **volatile per market**, and its **licence footprint is its product-market boundary**. Those three facts, not any narrative about gambling, are what a credit or merchant-risk view actually turns on.

### 4.4 Performance claims the company makes about itself

Two kinds of performance claim exist about this business, and the guide keeps them apart.

**Claims the company makes about itself** (labelled, dated, unverified):

- "**offices in Europe, Asia, and Latin America, employing over 5,000 professionals**" and "**accepts more than 250 payment solutions**" and "**around the clock customer support in 30 languages**" — the company's own "About 1XBET" text, July 2019 ✅ as a company claim.
- "**17 years of experience in the betting and gambling industry**"; website and app "**in 70 languages**"; partners listed as FC Barcelona, Paris Saint-Germain, LOSC Lille, Italian Serie A and the Confederation of African Football — the company's own "About 1XBET" text, July 2024 ✅ as a company claim.
- A statement, reported for January 2026, that the brand "held local licences in more than 35 markets across Latin America, Africa and Western Europe" and identified Serbia and Guatemala as recent regulated-market entries — a **company statement**, reported, with no register-by-register support produced and **none verified in this pass** ⚠.

**Third-party figures about the business** (each with its source and each flagged):

- **Turnover "exceeded $2 billion in 2020"** — attributed to Forbes Russia, 9 December 2020, via the encyclopaedia entry; **not verified in this pass and not reconciled with any other figure** ⚠.
- **"more than US$655 million" in illegal gambling** (2021) — an **Investigative Committee allegation figure as reported by Bellingcat (21 October 2024)**; it is an allegation with an authority behind it and it is recorded in [§6.3](#63-reported-but-unverified-matters-kept-out-of-the-record), not here.
- **Monthly average visits "exceeded 5 million"** (2024) — Bellingcat, 21 October 2024, citing SimilarWeb, a web-traffic measurement firm ⚠ (attributed methodology: third-party traffic estimation, not audited data).
- **"Tens of billions in terms of revenues"** — a **third-party commentator's estimate** quoted by Bellingcat (21 October 2024); explicitly an estimate by a person, not a figure ⚠.

**Where the guide records an absence.** No audited financial statement, no revenue figure for any named entity, no customer count, no employee count and no market-share figure for this operation is established anywhere in this pass. **That absence is the finding** and is carried in [§16](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary).

---

## 5. The Licensing Position

**This is the analytical core of the guide.** The rule of the section is that a licence is a fact only where a register, a certificate or an instrument says so, and that everything else is either a company claim or an absence.

### 5.1 What is held, verified at the authority's own record

| Item | Detail | Source |
| --- | --- | --- |
| **Operator entity** | **Caecus N.V.** | Curaçao Gaming Authority Certificate of Operation, retrieved September 2026 ✅ |
| **Company number** | **163779** | same ✅ |
| **Licence number** | **OGL/2024/1262/0493** | same ✅ |
| **Class** | B2C online gaming licence to "offer games of chance", issued under the post-2024 Curaçao statute | same ✅ |
| **Grant date** | **7 November 2024** | same ✅ |
| **Status at retrieval** | **Active** | same ✅ |
| **Statutory basis named in the instrument** | National Ordinance on Games of Chance (*Landsverordening op de kansspelen*), **P.B. 2024, no. 157** | same ✅ |
| **Authorised domains named** | **1xbet.com** and, in a parallel certificate, **1x-bet.com** | same ✅ |
| **Issuing authority** | Curaçao Gaming Authority (CGA), via the certificate portal at `cert.gcb.cw` | same ✅ |

**Why this table is the most important thing in the guide.** It converts the question "is 1xBet licensed?" — which is unanswerable, because a brand cannot hold anything — into two answerable questions: **which entity**, and **licensed by whom, in what class, as at when**. The answers are Caecus N.V., the Curaçao Gaming Authority, a B2C online gaming licence, as at 7 November 2024 with Active status at the September 2026 retrieval.

### 5.2 The Curaçao regime and what changed at the end of 2024

The instrument itself carries the regime change: Caecus N.V.'s licence is issued under the **National Ordinance on Games of Chance**, cited in the certificate as **P.B. 2024, no. 157** ✅ — the Curaçao statute that the authority's own certificate treats as the basis of licences issued from late 2024. Structurally (and this is context, unmarked): under the **pre-2024** regime, Curaçao online licences were issued by the Minister of Justice and administered through the licensing and trust-office structure known in the industry as Curaçao eGaming, and licences carry the familiar **".../JAZ..."** numbering; under the **post-2024** regime the **Curaçao Gaming Authority** issues licences in the **"OGL/…"** series, as Caecus N.V.'s certificate shows ✅, and operates a certificate/register portal through which third parties can verify a brand's operator and licence ✅.

Three practical consequences for a diligencer:

1. **The licence number's format is itself a dating device.** An "OGL/…" certificate is post-2024; a "…/JAZ…" number belongs to the earlier regime. A screening system that treats the two as interchangeable will mis-date a licence.
2. **A certificate naming the operator is the artefact to demand.** The CGA portal resolves a brand to an operator, a company number, a licence number, a grant date and a status ✅. That is a materially better artefact than a footer on a website.
3. **A register is a point in time.** The guide's retrieval is September 2026 and the licence's own grant date is November 2024 ✅; nothing in this guide establishes the licence's position at any other date, and a bank must re-pull at onboarding and at each periodic review.

**What could not be established about the Curaçao position:** the licence's expiry or renewal cycle (the certificate states the status, not the term) ⚠; whether the pre-2024 licence under which the same brand operated was held by the same or a different company ⚠; and whether any licence exists in the Curaçao register for any *other* entity in the brand family — **the register was consulted through per-brand certificates rather than enumerated, so no negative finding is asserted about other entities** ⚠.

### 5.3 Great Britain: a licence that was not there

The most-repeated licensing claim about this brand is that a British licence was revoked or suspended. **In this pass, that claim is not established, and the reason is structural rather than evidential.**

What the regulator's own record shows ✅:

- The Gambling Commission maintains public registers of **gambling businesses**, **personal licences**, **regulatory actions** and **public statements**, plus an FOI disclosure log (retrieved September 2026) ✅. The **regulatory actions** register reported **136 records** at retrieval ✅.
- An FOI request published by the Commission, **request date 16 January 2024**, asks for "details on the licence and subsequent suspension and possible ban on the betting and gambling brand 1xBet". The **outcome recorded is "Information withheld"**: the Commission stated it was "unable to confirm or deny whether we hold any information within the scope of your request", relying on **section 31(3) of the Freedom of Information Act 2000 (Law Enforcement)** and adding that "only once or if a formal regulatory decision has been made or there is agreement of a regulatory settlement the Commission will ordinarily publish all such decisions in full" ✅.

What that means, stated carefully:

- The Commission has **published no regulatory action, public statement or licence entry naming this brand** that this pass located, and it has expressly declined to confirm or deny that any exists ✅. That is a **documented negative** — not proof that nothing happened, and not proof that something did.
- The explanation that the trade press supplied in 2019 is that the UK-facing site was **not operating under a licence held by the brand at all**, but as a **white-label arrangement under the Gambling Commission licence of a technology partner (reported as FSB Technology)** ⚠ (iGamingBusiness, 2019). If that is right, the phrase "its licence was revoked" would be a category error: there was no 1xBet licence in Great Britain to revoke. **The guide reports that explanation as reported and does not adopt it.**
- The Commission's own reasoning, as quoted by the trade press in 2019, is worth recording exactly for what it establishes about the **sponsorship** question: a Commission spokesperson said the Commission had "recently wrote to Liverpool FC, Chelsea FC and Tottenham Hotspur FC to remind them that organisations engaging in sponsorship, and associated advertising arrangements, with an unlicensed operator may be liable to prosecution under **section 330** of the Act for the offence of advertising unlawful gambling" ✅⚠ (quoted by iGamingBusiness; the statement is the Commission's, carried by trade press).
- **The guide does not enumerate the 136 regulatory-action records**, and makes no claim about whether any of them names a company trading as 1xBet ⚠. The register exists and is the place to look; this pass did not search it to exhaustion.

### 5.4 The jurisdictional-scope point (the transferable finding)

This is the part of the guide that generalises beyond the subject, and it is the part a bank should carry to every gambling-operator file.

**A gambling licence is jurisdiction-scoped.** Caecus N.V.'s licence was issued by the Curaçao Gaming Authority ✅. It authorises the operator to offer games of chance **under Curaçao's statute**. It does not, and cannot, authorise the operator to serve customers in the Netherlands, Great Britain, France, Spain, Greece, Poland, Czechia, Lithuania, Estonia, Cyprus, Italy, Nigeria, Kenya, Uganda, Liberia, Russia, Ukraine or anywhere else. Each of those markets is governed by its own law and its own authority, and those authorities enforce against brands and operators that take their residents' money without the local instrument:

- **The authority that can determine illegality in a market is that market's authority.** When this guide says a brand appears on a regulator's unauthorised-operator list, it is **reporting the regulator's published listing**, and it attributes it. The guide does not make the determination itself, and it does not generalise from one market's list to a global characterisation.
- **The lawful-at-home / unlicensed-elsewhere combination is the norm, not the exception, in this sector.** An operator that is genuinely licensed and supervised in its licensing jurisdiction may simultaneously be serving customers in markets where it holds no licence. Both facts can be true. The reason the combination is dangerous for a bank is not moral but operational: **revenue booked from customers in markets where the operator is not licensed is revenue exposed to enforced market exit**, and, in several regimes, to the exposure of the payment intermediaries that carried it.
- **The only correct statement of scope is per market, dated.** A licensing map is not "licensed / unlicensed". It is: market — authority — instrument or listing — date read. The guide's own map is [§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached), and its gaps are marked.

### 5.5 The market-by-market position, as far as this pass reached

Every row below is either **(a)** an authority's own record, **(b)** an aggregator that states it reads a named regulator's own register or unauthorised-operator list, with the date the aggregator last read it, or **(c)** an explicit "not reached". **Rows marked (b) are leads that a bank must re-verify at the authority; this pass did not verify them individually.**

| Market | Authority | Position recorded | Sourcing | Status |
| --- | --- | --- | --- | --- |
| **Curaçao** | Curaçao Gaming Authority | **LICENSED** — Caecus N.V., OGL/2024/1262/0493, granted 7 Nov 2024, Active, for 1xbet.com / 1x-bet.com | Authority certificate retrieved September 2026 | ✅ **verified at source** |
| **Netherlands** | Kansspelautoriteit | **NO LICENCE — enforcement concluded.** Fines imposed for offering games of chance to Dutch players without a licence | Ksa decisions of 4 Jan 2019 and 7 May 2019 on the authority's own site | ✅ **verified at source** |
| **Great Britain** | Gambling Commission | **NO ACTION PUBLISHED AND NONE DENIED OR CONFIRMED** — the Commission declined to confirm or deny; no regulatory action or public statement naming the brand was located | Commission FOI response (request 16 Jan 2024) and public-register pages | ✅ **documented negative** |
| **Ukraine** | President of Ukraine / NSDC | **SANCTIONED ENTITY** — 1XCorp N.V. designated by the instrument of 10 March 2023 | Decree reported by the national news agency; sanctions-database entry citing the decree | ⚠✅ instrument identified; annex not read at source |
| **Liberia** | National Lottery Authority | **LOCAL LICENSEE'S LICENCE REVOKED** (22 June 2026); the complaint against the brand was dismissed because a brand is not a legal entity | NLA Ruling and Judgment, 22 June 2026, on the authority's own site | ✅ **verified at source** |
| **Spain** | DGOJ | **REPORTED LICENSED** — a named Spanish licensee (Wagerfair, S.A.) recorded as holding DGOJ general licences, matched by the authorised domain 1xbet.es | Aggregator stating it reads the DGOJ register | ⚠ lead; **DGOJ register returned an error page (HTTP 503) in this pass and was NOT read** |
| **Mexico** | SEGOB (DGJS) | **REPORTED LICENSED** — master permit recorded against a Mexican permit-holder, matched by domain 1xbet.com.mx, permit number quoted by the aggregator | Aggregator | ⚠ lead; not verified at SEGOB |
| **Serbia** | Uprava za igre na sreću | **REPORTED LICENSED** — online games-of-chance licence recorded against a Serbian company, matched by domain 1xbet.rs | Aggregator | ⚠ lead; not verified at the Serbian authority |
| **Peru** | Mincetur (DGJMT) | **REPORTED LICENSED** — recorded against the brand, domain 1xbet.pe | Aggregator | ⚠ lead |
| **Ghana** | Gaming Commission of Ghana | **REPORTED LICENSED** — sports betting and online casino recorded | Aggregator | ⚠ lead |
| **Uganda** | NLGRB | **REPORTED LICENSED** — general betting operating licence quoted by the aggregator against a named Ugandan company, domain 1xbet.ug | Aggregator | ⚠ lead; not verified at NLGRB |
| **Nigeria** | Lagos State Lotteries and Gaming Authority | **REPORTED LICENSED — EXPIRED** — the aggregator records the licence as expired | Aggregator | ⚠ lead; **an expir
ed licence is not a licence** and must be re-checked |
| **Kenya** | Betting Control and Licensing Board | **REPORTED LICENSED** — the brand appears on a reported BCLB list for the 2025/26 financial year, associated in trade reporting with a named licensee | Trade press reporting of the BCLB list | ⚠ lead; not verified at BCLB |
| **France, Italy, Greece, Cyprus, Czechia, Poland, Lithuania, Estonia** | ANJ; ADM; Hellenic Gaming Commission; National Betting Authority; Ministerstvo financí; Ministerstwo Finansów; Lošimų priežiūros tarnyba; Estonian Tax and Customs Board | **APPEARS ON PUBLISHED UNAUTHORISED-OPERATOR OR BLOCKING LISTS** — the aggregator lists the brand across eight regulators' lists, stating it last read each on dates in late September 2026 | Aggregator that states it reads the regulators' own lists | ⚠ **reported, not verified individually at each authority in this pass** |
| **Ireland** | Office of the Revenue Commissioners | **REPORTED LICENSED** — a named Irish company recorded as holding a remote bookmaker's licence, trading name "1XBET" | Encyclopaedia entry citing the Irish licence record | ⚠ **not register-verified in this pass** |
| **Russia** | Federal Tax Service (and the Russian regulator of bookmaking) | **NOT ESTABLISHED** — the Russian-facing brand is a separately licensed Russian bookmaker; the FNS register was **not reachable in this pass** (404 on the register path attempted) | Russian aggregators (conflicting licence dates); register unreachable | ❌ **not established**; tool limitation recorded |
| **Brazil** | Secretaria de Prêmios e Apostas (Ministry of Finance) | **REPORTED LICENSED** — a claim that the brand is authorised federally under a named Brazilian company | A commercial review site | ⚠ **not verified; low-quality source; treated as a lead only** |
| **Non-EU/EEA and other markets** | various | **NOT ESTABLISHED.** The company's own January-2026 statement claiming local licences in "more than 35 markets" is a **company claim** with no register-by-register support in this pass | Company statement via encyclopaedia | ⚠ company claim; ❌ not verified |

**The honest summary of §5.5.** One licence is verified at source (Curaçao) ✅. One market's enforcement is verified at source (Netherlands) ✅. One market's negative is documented (Great Britain) ✅. One market has a concluded local-licensee proceeding verified at source (Liberia) ✅. One sanctions designation is verified as to instrument and reported as to listing (Ukraine). **Everything else is a lead or an absence.** The licensing map of a brand like this is not a page of facts; it is a page of leads and negatives, and the guide reports it as such.

### 5.6 The licence-quality spectrum, by class and jurisdiction

The guide's rules forbid it from ranking named operators on its own authority. What it *can* do — and this is the transferable analytical content — is set out the **dimensions** along which gambling licences differ, because those dimensions are what a bank's risk rating actually depends on. This is **structural analysis, labelled as the guide's own** and not attributed to any regulator.

| Dimension | What varies | Why a bank should care |
| --- | --- | --- |
| **Issuing jurisdiction's supervisory intensity** | Some licensing jurisdictions impose fit-and-proper tests, capital requirements, player-protection rules, AML supervision, reporting, and active enforcement against their own licensees; others operate a lighter registration-style regime. Both are "licences". | A licence from a light-touch regime tells a bank far less about conduct than a licence from an intensive supervisor. The instrument's *issuer* is a risk input, not the instrument's existence. |
| **Scope versus the customer base** | A licence's scope is the markets its holder may lawfully serve. An operator's customer base is a different set. | The gap between them is the **customer-jurisdiction mismatch** ([§9.4](#94-the-customer-jurisdiction-risk)) and it is the single most important licensing-derived risk in the file. |
| **Local subsidiary versus white label** | A directly licensed local subsidiary is a legal person answerable to the local regulator; a white-label arrangement means the brand is riding on someone else's licence and the brand itself holds nothing locally. | Under a white label, the entity the bank is onboarding may not be the entity the local regulator supervises at all. |
| **Licence class** | B2C operator licences, B2B supply licences, and activity-specific licences (sports betting, casino, lottery) are different instruments with different conditions. | A B2B licence is not evidence of anything about the operator; a supply licence does not permit taking bets. |
| **Single-jurisdiction versus multi-jurisdiction** | A single-licence operator is a concentrated regulatory risk; a multi-licensed operator is a portfolio of jurisdictional risks, each of which can move independently. | Portfolio thinking replaces binary thinking: how many of the operator's revenue-bearing markets are properly licensed, and how much revenue sits in the unlicensed remainder? |
| **Term, renewal and provisionality** | Some licences run multi-year terms; some licensing regimes operate short, rolling provisional periods with renewal; some statuses are easily suspended or allowed to lapse. | A status field is a point-in-time reading. A licence due to expire in weeks is a different credit input from one running for years. |
| **Enforcement record in the issuing jurisdiction** | Whether the issuer has acted against the holder — and how the holder responded. | The Netherlands matter in this guide is the archetype: the enforcement is public, the fine was imposed, and the authority's subsequent published position was that non-payment is itself a reliability signal ⚠ (attributed by the guide to the Ksa via press reporting). |

**The guide's own conclusion, labelled as reasoning:** a bank should treat **"is it licensed?"** as an incomplete question and replace it with the five-part form — *which entity, licensed by which authority, in which class, for which markets, as at which date* — and then measure the mismatch between that answer and the markets in which the operator's revenue actually arises. That formulation is the licence-quality spectrum in operational dress, and it is the one piece of this section a reader can lift into another company's file unchanged.

---

## 6. The Regulatory Record

*This section is the guide's highest-risk writing and is therefore structured for audit rather than for narrative. Every entry is a **dated instrument**: authority, jurisdiction, instrument, date, parties, outcome or status, source. **Concluded matters, ongoing matters and reported-but-unverified matters are in three separate groups**, and the unverified groups live in [§15](#15-the-claims-audit), not here. Nothing in this section is the guide's own characterisation of the company or of any person.*

### 6.1 Concluded matters

**6.1.1 Netherlands — Kansspelautoriteit — administrative fines, 2019 ✅**

| Field | Detail |
| --- | --- |
| **Authority** | Kansspelautoriteit (the Netherlands gambling authority) |
| **Jurisdiction** | Netherlands |
| **Instrument(s)** | (a) Sanctiebesluit (administrative-fine decision) of **4 January 2019**, kenmerk **12924/01.046.245**; (b) openbaarmakingsbesluit of the same date, kenmerk **12924/01.046.246**; (c) Adviescommissie bezwaarschriften advice dated **26 April 2019**, kenmerk 12924/01.056.115; (d) **Besluit op bezwaar** (decision on objection) of **7 May 2019**, kenmerken **12924/01.056.071** and **13175/01.056.072** |
| **Dates** | Investigation of websites 16 February 2018 – 26 July 2018; investigation report **18 September 2018**; fine decision **4 January 2019**; publication of the sanction decision on the authority's website **25 February 2019**, after the preliminary-relief judge of the District Court of The Hague refused on **22 February 2019** to restrain immediate publication (case references recorded as SGR 19/284 and SGR 19/286); objections filed **11 January 2019**; hearing **4 March 2019**; advice **26 April 2019**; decision on objection **7 May 2019** |
| **Parties** | **1X Corp N.V.** and **Exinvest Limited** (as the two named undertakings) |
| **Finding** | The authority's investigation report of 18 September 2018 concluded that 1X Corp N.V. and Exinvest Limited had acted contrary to **article 1(1)(a) of the Wet op de kansspelen** by offering, on the websites **1xbet.com** and **xbet-1.com**, the opportunity to compete for prizes determined by chance without a licence |
| **Outcome** | Original sanction decision: a **€400,000** administrative fine on both, jointly and severally liable. On objection, the decision was **varied**: the board declared the objection well-founded as to joint liability and imposed **separate fines of €200,000 on 1X Corp N.V. and €200,000 on Exinvest Limited**, awarded **€1,024** in legal costs jointly, dismissed the remaining objections, and left the publication decision standing. The decision records the right of appeal to the District Court of The Hague |
| **Status** | **CONCLUDED** at the administrative stage; whether an appeal was lodged and its outcome is **not established** in this pass |
| **Source** | The authority's own published documents on `kansspelautoriteit.nl` (`12924_advies_bac_ov.pdf`, `12924_beslissing_op_bezwaar_ov.pdf`), retrieved September 2026 ✅ |
| **Subsequent reporting (not part of the instrument)** | Dutch and Curaçao-based reporting states that the fines went unpaid and that the authority began recovery (invordering) proceedings, and attributes to the authority the remark that non-payment gives "a strong indication that the provider is unreliable" ⚠ — reported by press, attributed, and not verified at an authority document in this pass |

**6.1.2 Liberia — National Lottery Authority — licence revoked, complaint against the brand dismissed, 2026 ✅**

This is the single most instructive administrative document in the guide, because the tribunal **dismissed the case against the brand** precisely on the ground that this guide's §1 thesis asserts.

| Field | Detail |
| --- | --- |
| **Authority** | National Lottery Authority of the Republic of Liberia (NLA), sitting as an administrative hearing before a Hearing Officer |
| **Jurisdiction** | Liberia |
| **Instrument** | *NLA v. The Management of LIPAY, Inc. (1st Defendant) and The Management of 1 X BET (2nd Defendant)* — **Ruling and Judgment**, given 22 June 2026 |
| **Date** | Judgment **22 June 2026**; the underlying suspension of the first defendant's operating licence was announced **16 December 2025**; the investigating report cited is dated **9 December 2025** |
| **Parties** | Complainant: the NLA. 1st Defendant: **LIPAY, Inc.** (a Liberian licensed operator). 2nd Defendant: **"The Management of 1 X BET"** |
| **What the authority alleged** | That the 1st Defendant violated the NLA Act and **NLA Gaming Regulation 001** by (i) engaging in the **unauthorised assignment of its sports betting licence** to the 2nd Defendant, and (ii) **permitting illegal online gaming operations in Liberia** (the ruling records these as the Complainant's allegations) |
| **What the ruling records on the facts** | Public advertising in Monrovia bearing the brand: billboards at seven named locations, radio advertisements and social-media posts featuring the brand, with, in the investigative report's words, no readily visible reference to any entity other than a "reference link" to the licensee. A **Trademarks License Agreement** between the licensee and **DIDIANE LTD**, a company registered in the Republic of Cyprus, executed **13 May 2025**, granting the licensee the right to use the 1XBET trademarks in Liberia in exchange for a monthly royalty of **US$5,000**. Testimony by the authority's ICT director that the platform resolved to infrastructure operated by "1xBet Global", that under the Orange and MTN mobile-money systems no system was installed in Liberia for the licensee, and that payments went to accounts in the name of 1XBET rather than to the licensee, and that the authority had no licence or agreement with 1xBet |
| **Judgment** | (A) The complaint against the **2nd Defendant, the Management of 1 X Bet, was DISMISSED WITHOUT PREJUDICE for misjoinder and failure to sue a legal entity**; (B) the 1st Defendant, LIPAY, Inc., was found liable under §§6.4, 6.9, 14.1(b) and 14.1(d) of the Regulation and also in breach of Liberian intellectual-property and general business law; (C) the sports betting licence issued to LIPAY, Inc. was **REVOKED**, effective immediately; (D) administrative fines totalling **US$10,000** (US$2,500 per violation) were imposed; (G) LIPAY was barred from reapplying for a Liberian gaming licence for **two years** |
| **Status** | **CONCLUDED** at the administrative hearing level **as at the judgment date**; the guide found no appellate outcome. Separate reporting in 2026 states that the licence was subsequently **restored** to the licensee under a settlement with a one-year 2026/2027 licence — **reported by the Liberian press, not verified at an NLA instrument in this pass** ⚠ |
| **Source** | The NLA's own published ruling, `nla.gov.lr`, retrieved September 2026 ✅; the restoration reported by the *Liberian Observer* ⚠ |

**Why this matters more than the outcome.** The tribunal's first holding is the identity-gate lesson performed by an authority: **an administrative action against "1 X BET" was dismissed because a brand is not a legal entity and cannot be sued.** A bank that treats the brand as its counterparty is making the same category error, one step earlier and at its own expense. Note also the *structure* the ruling reveals — a local licensee with a **trademark licence** to a **third company** (Cyprus), branded premises, and payment flows to the brand's own wallet ✅ — which is a concrete, court-grade illustration of the "merchant of record is not the brand" problem in [§9.1](#91-identifying-the-counterparty-which-entity-is-the-bank-onboarding).

**6.1.3 Ukraine — sanctions designating entities in the betting and lottery business, 2023 ⚠✅**

| Field | Detail |
| --- | --- |
| **Authority** | President of Ukraine, enacting a decision of the National Security and Defence Council of Ukraine (NSDC) |
| **Jurisdiction** | Ukraine |
| **Instrument** | **Decree No. 145/2023 of 10 March 2023**, enacting the NSDC decision of 10 March 2023 on the application of and amendments to personal special economic and other restrictive measures (sanctions) |
| **Date** | **10 March 2023** |
| **Scope as reported** | Sanctions against individuals and legal entities connected in particular with the betting and lottery business; the President's address is reported to describe "more than 280 companies and 120 people", with sanctions imposed for **50 years**; among the named legal entities in the reported annex are Russian betting and lottery operators |
| **Relevance to this guide** | A sanctions-database entry records **1XCorp N.V.** (Ukrainian rendering "Уан-ЕксКорп Н.В."), described as registered in **Curaçao** at a Willemstad address with registration data **130189**, as designated with effect from **10 March 2023**, citing Decree No. 145/2023, and with a listed expiry of **10 March 2073** |
| **Status** | The instrument's **existence, number, date and subject** are established from the Ukrainian national news agency's report and the sanctions database; **the decree's annex was not read at source in this pass**, so the specific listing of 1XCorp N.V. is **reported, not independently verified** ⚠ |
| **Source** | Ukrinform (Ukrainian National News Agency) report of the sanctions and the decree number ✅; a sanctions database entry for 1XCorp N.V. citing the decree ⚠; the presidential website URL attempted for the decree text did not resolve to a retrievable document in this pass ⚠ |
| **A caution the guide will not skip** | The relationship between **1XCorp N.V.** and **Caecus N.V.** (the current Curaçao licensee, §2.1) is **not established** by any source read. Journalistic reporting has described 1XCorp N.V. as a parent or related company; that is a **description in press reporting**, not a register finding, and this guide does not adopt it |

**6.1.4 The refusal-to-confirm entry (Great Britain)** — recorded here because it is itself an authority's public act, although its substance is a negative. The Gambling Commission's published response to an FOI request dated 16 January 2024 declined to confirm or deny whether it held any information about a licence, suspension or ban concerning the brand, citing s31(3) FOIA ✅. **This is a documented non-finding, not a finding**, and it is treated as such: see [§5.3](#53-great-britain-a-licence-that-was-not-there).

### 6.2 Ongoing matters, and matters whose status is not established

The guide distinguishes these carefully, because "ongoing" is a claim in itself and can only be made where a source says a matter is live.

| Matter | What is established | What is not | Status |
| --- | --- | --- | --- |
| **Morocco** — reported investigation by the National Judicial Police Brigade following a complaint by a Moroccan operator (reported 2023) | That the investigation is **reported** (encyclopaedia entry citing Bellingcat, 21 October 2024) | Whether it is live, stayed, closed or resulted in any action; the instrument; the parties; the outcome. **No Moroccan instrument was located in this pass** | ⚠ reported; **STATUS NOT ESTABLISHED** |
| **Russia** — reported criminal case and reported international arrest warrants (2020–2021), with assets reported seized | That the encyclopaedia record and Bellingcat report the existence of a Russian criminal case and of arrest warrants, and attribute the naming of individuals to the **Investigative Committee for the Bryansk Region**; Bellingcat attributes to the investigators a figure of "more than 63 billion rubles" (reported as ~US$655 million) in 2021 | The instrument; the docket; the current status; whether any defendant has been convicted. **The Russian authority's own release was not read in this pass.** **No individual is named in this guide** ([§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under) rule (v)) | ⚠ reported; **STATUS NOT ESTABLISHED** |
| **The Curaçao entity described as "1xCorp MV"** — reported insolvency proceedings (petition reported November 2021; declared bankrupt reported June 2022; a Netherlands ruling reported January 2023) | That the sequence is reported by investigative journalism and by an encyclopaedia entry citing it | The court, the docket, the date, the parties, the outcome; and whether the entity is related to Caecus N.V. **No court record was read in this pass** | ⚠ reported; **STATUS NOT ESTABLISHED** |
| **Any matter in any market in which the brand is served but which this pass did not research** | Nothing | Everything | ❌ **not established by design**: the guide does not assert the existence of an unenumerated market's silence or of an unenumerated market's action |

**A discipline note on this subsection.** It would be easy, and wrong, to write a paragraph asserting that "investigations continue in several jurisdictions". The guide has found **reports** of matters, not instruments. Where it cannot produce an instrument or an authority's own public act, it says so and stops — and it does not name the individuals the reporting names.

### 6.3 Reported but unverified matters, kept out of the record

Each item below is a claim this pass met in **press, encyclopaedia or commercial-aggregator** form, without an authority's own instrument. They are recorded **here**, and in the [§15 claims audit](#15-the-claims-audit), so that no reader mistakes them for part of §6.1. None is stated as fact anywhere in this guide.

1. **The Great Britain "licence revoked" claim** — repeatedly stated in secondary sources; **contradicted in structure** by the fact that the UK-facing operation is reported as a white-label on another company's licence and by the regulator's own refusal to confirm or deny. **Treated as unestablished.** ⚠❌
2. **The 2020 Russian "blacklist of payment processors and the Federal Tax Service" claim** — encyclopaedia-sourced, citing Russian business press; no Russian instrument was read. ⚠
3. **Reported annual-visit estimates ("more than 5 million monthly")** — Bellingcat citing a traffic-estimation firm. ⚠
4. **Reported partner-network descriptions** — the brand lists FC Barcelona, Paris Saint-Germain, LOSC Lille, Serie A and the Confederation of African Football among its partners (company text, July 2024) ⚠. Only the **FC Barcelona** relationship is verified at the sponsor's own announcement ✅; the others are **company claims** unless a rightsholder's own announcement is produced.
5. **Reported and subsequently ended tournament sponsorships and ambassadorship arrangements** (Billie Jean King Cup, ATP Challenger Tour, esports organisations, individual ambassadors, a Philippine basketball league title sponsorship reported as later dropped mid-season) — encyclopaedia-sourced from press releases and trade press; the sponsor announcements themselves were not rendered in this pass. ⚠ (**Billie Jean King Cup and ATP Challenger Tour are the most dated and most checkable of these**; a bank should verify them at the rightsholder.)
6. **An aggregator-sourced claim of "active Indian money laundering probe"** — encountered in a commercial dossier-style site, with no authority named and no instrument produced. **The guide does not rely on it, does not repeat it as a matter, and records it only as a rejected-for-lack-of-support item.** ❌
7. **A reported oddity about the operator's own spokesperson identity** — Bellingcat reported (21 October 2024) that the profile photograph used for a named press spokesperson was that of a journalist at another broadcaster, and that the broadcaster said neither it nor its employee knew the photograph was being used that way. Recorded as **reported**; it is a media-identity observation, not a regulatory matter. ⚠
8. **Reported national site-blocking measures (e.g. Nepal, Sri Lanka)** naming the brand among many — trade-press and news-aggregator sourced, no instrument read. ⚠

---

## 7. The Sponsorships and the Legitimacy Question

### 7.1 Why this section is adjacent to the licensing section

In this industry, top-tier sponsorship is the principal instrument by which an operator converts marketing spend into **perceived legitimacy** — with the public, with payment providers, and with the local regulators whose licensing decisions it may later need. That is precisely why the **end** of a sponsorship is as informative as its beginning: a regulator's warning to a club about advertising unlawful gambling, or a club's own decision to suspend, is a market signal about the operator's regulatory standing at that date. This section therefore presents the dated, sourced deals first, and the analysis after, labelled.

### 7.2 Verified sponsorships — the sponsor's own announcement

| Partner | What was announced | Date | Source | Status |
| --- | --- | --- | --- | --- |
| **FC Barcelona** | 1XBET becomes a **Global Partner** of the club for five seasons, the agreement taking effect 1 July 2019 and running **through 30 June 2024** | Announcement published **3 July 2019** | **fcbarcelona.com** — the club's own announcement, which quotes the club's commercial-area board member and the operator's spokesperson | ✅ **verified at the sponsor's own announcement** |
| **FC Barcelona** (renewal) | The partnership renewed: the brand continues as **Global Partner and Official Betting Partner** for five more seasons, **through June 2029** | Announcement published **1 July 2024** | **fcbarcelona.com** — the club's own announcement, quoting the club's marketing vice-president and the operator's spokesperson | ✅ **verified at the sponsor's own announcement** |

The July 2024 announcement also reproduces the operator's own "About 1XBET" text, which names the brand's other partners — **Paris Saint-Germain, LOSC Lille, Italian Serie A and the Confederation of African Football** ✅ — as a **company claim carried by the sponsor**. That distinction (company claim, not sponsor confirmation) is preserved here and is the reason those relationships appear below as flagged rather than verified.

### 7.3 Sponsorships reported, flagged or ended

| Partner | Position recorded | Source, with date | Status |
| --- | --- | --- | --- |
| **Tottenham Hotspur** | The club **ended** its agreement with the operator, which had served as its **African betting partner**; the termination is reported as preceding the Liverpool and Chelsea suspensions by about a week | Reported by the gambling trade press (iGamingBusiness, 2019) ⚠; the severance of the Chelsea, Liverpool and Tottenham deals is reported by **Bellingcat, 21 October 2024** ✅ as a report of the severance | ⚠ **the termination is the fact;** the reason is attributed below |
| **Liverpool FC and Chelsea FC** | Both clubs **suspended** their partnerships, announced in July 2019, following the 2019 newspaper investigation and questions about the operator's conduct | Same sources ⚠; the Gambling Commission's own quoted statement (below) ✅⚠ | ⚠ **reported; not verified at either club's own announcement in this pass** |
| **The regulator's warning, which is the best-attributed reason available** | A Gambling Commission spokesperson is quoted saying the Commission "recently wrote to Liverpool FC, Chelsea FC and Tottenham Hotspur FC to remind them that organisations engaging in sponsorship, and associated advertising arrangements, with an unlicensed operator **may be liable to prosecution under section 330 of the Act for the offence of advertising unlawful gambling**", and that "the best way for sports bodies to protect themselves against this risk is to ensure that they only promote gambling operators licensed by us" | Quoted by iGamingBusiness, 2019; the Commission is named as the source of the statement | ✅⚠ **the statement is the Commission's, carried by trade press** |
| **Paris Saint-Germain** | Reported as a partner of the brand; one dated journalistic source states in October 2024 that the brand "remains a sponsor" of the club, citing the club's own announcement page; the encyclopaedia record gives the relationship as 2022–2025 | Bellingcat, 21 October 2024 ⚠; encyclopaedia entry ⚠; the club's own announcement page did **not render** in this pass | ⚠ **reported; sponsor announcement not retrieved** |
| **Italian Serie A; Confederation of African Football; LOSC Lille** | Named as partners in the operator's own text (July 2024); the operator's 2019 text (July 2019) also named Serie A and Tottenham | Company text carried in FC Barcelona's announcements ✅ as a company claim | ⚠ **company claim; no rightsholder announcement retrieved** |
| **Billie Jean King Cup** | Reported as the brand's first standalone betting partnership for the tournament (announced April 2025); encyclopaedia entry cites the Billie Jean King Cup's own press release | Encyclopaedia entry citing the Billie Jean King Cup press release; the press-release page did **not render** in this pass | ⚠ **reported; sponsor release not retrieved** |
| **ATP Challenger Tour** | Reported as the tournament series' Official Betting Partner from October 2025, covering 30+ tournaments | Encyclopaedia entry citing trade press | ⚠ reported |
| **Esports organisations and individual ambassadors** | Reported partnerships with esports organisations (OG, Tundra, and others) and ambassadorship arrangements with a Lethwei champion (November 2022), a musician (October 2023), and UFC fighters (2026) | Encyclopaedia entries citing the organisations' own posts, iGaming Brazil (August 2022), and press | ⚠ reported; **one dated primary-style citation exists for OG esports (16 May 2022) but was not retrieved** |
| **Maharlika Pilipinas Basketball League (Philippines)** | Reported title sponsorship from February 2025 with a contract reported to run to 2026; the encyclopaedia record states the sponsorship was **later dropped mid-season** | Encyclopaedia entry citing press | ⚠ reported; **the end-of-sponsorship claim is itself reported and unverified** |

### 7.4 What sponsorship buys — the guide's own analysis, labelled

*The following is the guide's reasoning, not a sourced finding, and is labelled as such throughout.*

1. **Perceived legitimacy is a payment for a licensing journey.** A tier-one club partnership is the cheapest available substitute for a regulatory imprimatur: it signals to consumers, to affiliates and to the local partners an operator needs in a new market that a reputable institution was willing to be publicly associated with the brand. For an operator whose licence footprint is thin in its largest customer markets, that signal is doing work that a licence would otherwise do.
2. **The signal is asymmetric in time.** The announcement of a sponsorship is an asset; the end of one is a liability disclosed at the worst moment. Both are public, and both are dated. For a bank, the *termination* therefore has greater information content than the signing: it dates the moment at which a sophisticated counterparty re-priced the relationship.
3. **A rightsholder's decision is not a regulatory finding, and a regulatory warning is not a sanction.** In the 2019 episode the clubs' suspensions followed a **warning letter** from the Commission, and the Commission's quoted reasoning was about the clubs' own exposure under **section 330** of the Act for advertising unlawful gambling ✅⚠. No instrument against the brand was published, and the Commission later declined to confirm or deny the existence of any ✅. The correct reading is that **sponsorship ended because of a legal-risk warning to the sponsor, not because of an adjudicated finding about the operator** — and any guide that states it the other way has overstated the record.
4. **Sponsorship is a banking-relevant fact in two directions.** It is an **expense** (marketing cost, cash outflow, often hard-currency and paid to regulated sporting entities, which makes it *visible* in account activity), and it is a **counterparty-reputation variable** (a sponsor's exit is a dated negative signal that a monitoring system can be built to catch).
5. **What sponsorship does not do.** It does not create a licence, move a licensing boundary, cure a customer-jurisdiction mismatch, or substitute for player-protection and AML compliance. Reader-facing framing inside the sector frequently implies otherwise; a bank's file should never contain the sentence "they sponsor Barcelona, so they must be regulated".

---

## 8. The Affiliate and Marketing Model

### 8.1 How online gambling acquires customers

*Structural description; the mechanics are industry-standard and unmarked. Cross-reference the supplier-side treatment in [Playtech & Its Competitors §3, §7](../technology/playtech_competitors_guide.md), which owns the platform-and-content layer this acquisition runs on.*

An online operator's funnel is: **marketing → registration → identity verification → first deposit → repeat play → retention**. Each stage has a channel:

- **Paid media** — search, social, display, video and broadcast, subject in most markets to advertising restrictions specific to gambling.
- **Affiliate marketing** — third parties who place links and content and are paid on referred registrations or depositing customers. The affiliate layer is the sector's characteristic acquisition channel because it scales into markets and audiences that paid media cannot reach — including, notoriously, markets where the operator holds no licence, and content whose tone no brand would put in its own name.
- **Sponsorship and ambassadors** — the legitimacy channel ([§7](#7-the-sponsorships-and-the-legitimacy-question)).
- **Local licence partners and franchisees** — a local company holds the licence, brands itself with the operator's trademark, and drives local acquisition; the operator supplies brand, platform and payment plumbing.
- **Cross-sell and retention** — bonus mechanics, VIP schemes and CRM, which are platform features rather than marketing channels, and which are the origin of most player-protection complaints in the sector.

### 8.2 The affiliate layer, and what it means for a bank

- **The affiliate is a separate business, usually unlicensed.** Affiliates are ordinarily not themselves gambling licensees, so the market's advertising rules bind the **operator** even when the advertising was placed by an affiliate. Regulators in several markets have made exactly this point by enforcement against operators for affiliate-placed advertising.
- **Affiliate networks are cross-border by construction.** A single campaign can place advertising in dozens of jurisdictions from one affiliate account. This is the mechanism by which "we do not target market X" and "our advertising is in market X" coexist.
- **The affiliate layer is where the reputation of a gambling brand is actually built and damaged** — its content is fast, cheap and largely outside the operator's editorial control.
- **For a bank, affiliates are invisible on the rails but visible in the volumes.** Marketing spend passes through the operator's accounts to a long tail of small counterparties and platforms in many jurisdictions; that pattern is itself a monitoring input in [§9.6](#96-ongoing-monitoring-what-the-bank-watches-after-onboarding).

### 8.3 The grey-market advertising problem, at the level sources establish

The guide states this at the level its sources support, and does not generalise beyond it.

- **The Netherlands (2018–2019)**: the authority's investigation covered websites it recorded as operated by the two named companies and found they offered games of chance to Dutch players **without the licence the statute requires**, and the enforcement followed ✅. The *mechanism* of the offering was the website itself.
- **Great Britain (2019)**: the Commission's own quoted position is that bodies promoting an **unlicensed operator** may be liable for the offence of **advertising unlawful gambling** under section 330, and it wrote in those terms to three Premier League clubs ✅⚠. This is an authority's statement that **advertising exposure** runs to the advertiser — which is why sponsorship terminations followed.
- **Liberia (2025–2026)**: the NLA's own investigative record found **billboards at seven named locations in Monrovia, radio advertisements and social-media posts** featuring the brand with no readily visible reference to the licensed entity, and the tribunal treated that presentation as a licence violation ✅. An authority's own document, therefore, establishes that the marketing presentation *was itself* the regulatory breach in that market — not the betting activity alone.
- **Reported blocking measures in a number of other markets** (flagged, §6.3) rest on the same logic in reverse: where the advisory or licensing status is absent, the practical remedy applied has been to block the domain.

**What the guide will not say.** It will not describe the acquisition model as unlawful, deceptive or fraudulent; it will not assert that the operator targets specific markets; and it will not repeat affiliate-industry or competitor-sourced claims about the model's conduct. Where an authority has said something, the authority is quoted; where a sponsor or a journalist has said something, the source is named and the claim is labelled.

### 8.4 The reputational and regulatory consequence of the acquisition model

Four consequences are identifiable at the level of sources this guide reached, and each is a **banking** consequence as much as a reputational one:

1. **Advertising exposure reaches the advertiser** (section-330 warning, Great Britain, 2019) ✅⚠ — which means a bank financing a sponsor, or banking an affiliate network, is inside the same risk perimeter as the operator.
2. **Presentation can itself be the breach** (the Liberian finding on billboards and brand visibility) ✅ — which means the marketing artefacts are evidence, and a bank's marketing-materials diligence on a gambling client is not a formality.
3. **Market exit is the enforcement remedy of choice** ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)) — which converts an unlicensed-market advertising programme into a **revenue-durability question** for the bank, not merely a compliance one.
4. **Acquisition cost is the largest variable cost** ([§4.3](#43-the-structural-economics-of-an-online-operator)) — which is why operators with thin licensing positions spend so heavily on the legitimacy channel of [§7](#7-the-sponsorships-and-the-legitimacy-question), and why the sponsorship line and the licensing line in the same file should always be read together.

---

---

## 9. The Bank's View — the Merchant and Payment-Counterparty Assessment

*This is the guide's payoff section and, after [§5](#5-the-licensing-position), its most substantial. The rule it applies throughout: **the classifications, the obligations and the decisions described here are the bank's own and its acquirer's**, taken under the frameworks of whatever authority licenses the bank and whatever authority licenses the acquirer. Nothing in this section is a rating of the operator's character, and nothing in it is a finding about this operator that [§5](#5-the-licensing-position) and [§6](#6-the-regulatory-record) did not already establish. Where a subsection states the guide's own reasoning, it says so.*

### 9.1 Identifying the counterparty: which entity is the bank onboarding

A bank cannot onboard a brand, because a brand is not a legal person. That is not a stylistic point; it is the holding of a tribunal. In the Liberian matter, the National Lottery Authority's complaint against "**The Management of 1 X BET**" was **dismissed without prejudice for misjoinder and failure to sue a legal entity** ✅ ([§6.1.2](#61-concluded-matters)). A bank that contracts with "1xBet" makes the same category error one step earlier, and at its own expense.

The counterparty a bank can actually onboard is the legal person, and the guide's register-grade answer for the dot-com operation is **Caecus N.V.** — Curaçao company number **163779**, licence **OGL/2024/1262/0493**, granted **7 November 2024**, status **Active**, per the Curaçao Gaming Authority's own Certificate of Operation ✅ ([§2.1](#21-the-brand-is-not-an-entity--the-register-entry-that-names-the-licensee), [§5.1](#51-what-is-held-verified-at-the-authoritys-own-record)). Where a market requires a locally licensed entity, the counterparty is a **different** legal person again, and the Curaçao licence says nothing about it.

Three structural features of this sector make the identification harder, and all three are visible in this guide's public record rather than asserted by the guide:

| Feature | The evidence in this guide | Why it defeats a name-based onboarding |
| --- | --- | --- |
| **The licence-holder and the billing/settlement entity are different companies** | In the Dutch instrument, the two named undertakings were **1X Corp N.V.** and **Exinvest Limited**, the second described in the authority's own file as a **"factureringsagent"** (*billing agent*) ✅⚠ ([§3.2](#32-the-two-entity-origin--and-the-naming-trap-that-follows), [§6.1.1](#61-concluded-matters)) | The company that holds the licence is not necessarily the company on the acquiring side of the transaction |
| **The trademark and the operation are licensed apart from each other** | The Liberian ruling records a **Trademarks License Agreement** executed **13 May 2025** between the Liberian licensee and **DIDIANE LTD** (a Cyprus company), granting the right to use the 1XBET trademarks in Liberia for a monthly royalty of **US$5,000**; the authority's ICT director testified that payments went to accounts **in the name of 1XBET** rather than to the licensee ✅ ([§6.1.2](#61-concluded-matters)) | The brand-owner, the licensee and the collection account can be three different parties, and the bank may see only the third |
| **The merchant of record for a given market is not public** | No source read in this pass establishes it for any market ❌ ([§2.2](#22-the-entity-and-brand-map-with-the-source-for-each-relationship) row 8, [§2.6](#26-what-could-not-be-established-about-this-identity--bluntly)) | It is a document-level fact obtainable only from the merchant and its acquirer, not from a website or a register |

**The onboarding questions, restated as a file.** For a gambling brand, the bank's file must answer these as separate items, and must not collapse them into one "entity name" field:

| Question | Where the answer lives | Status in this guide |
| --- | --- | --- |
| Which legal person is the counterparty? | Commercial register + the licensing authority's certificate | ✅ **Caecus N.V.**, Curaçao company no. 163779 (for the dot-com operation) |
| Which entity holds the licence covering the markets of concern? | Each market's own register | ✅ Curaçao; ❌ not established for most other markets ([§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached)) |
| Which entity is the merchant of record / receives settlement? | Merchant and acquirer documents | ❌ not public |
| Who ultimately owns and controls the licensee? | Commercial register, licensing file, or the bank's own UBO diligence | ❌ not established ([§2.6](#26-what-could-not-be-established-about-this-identity--bluntly) items 1–2) |
| Whose name is on the operating account receiving stake deposits? | The onboarding documents and the payment instruction | ❌ not public; and the Liberian record shows it can differ from the licensee ✅ |

That table is the whole of [§1](#1-the-overview-the-identity-gate-the-decoder-and-the-boundary)'s thesis in operational dress: **the licence is the identity, and the settlement account is a separate question from both.**

### 9.2 The deposit and withdrawal rail mix

A gambling merchant's payment profile is not one profile; it is a **portfolio of rails**, and the mix is a risk variable in its own right because each rail differs in **reversibility**, in **who bears the loss when a customer disputes**, and in **how easily it moves value out of the banking system's view**. The description below is **structural** (unmarked): the guide has no rail-level figure for this operator, and records that absence rather than inventing a percentage breakdown.

The only rail-mix claim in the public record for this brand is the company's own, carried inside FC Barcelona's July 2019 announcement: the operator stated it **"accepts more than 250 payment solutions"** ✅ — a **company claim**, dated by the announcement that carries it, and not a verified rail inventory. The guide does not turn it into a rail-mix estimate.

| Rail class | Typical role for a gambling merchant | Reversibility / dispute exposure | Banking significance |
| --- | --- | --- | --- |
| **Card (credit and debit)** | First-deposit rail in most developed markets; the rail most associated with chargeback exposure | **High reversibility.** Card-network dispute rules in the networks' own scheme rules — the same rules that place gambling in the high-risk merchant category, commonly referenced by **MCC 7995** — shape how disputes are raised and who funds them | Drives reserve/holdback structures, chargeback-rate thresholds and, in some cases, termination of acquiring |
| **E-wallets / PSP accounts** | A common first and repeat rail; the wallet sits between the card rail and the operator | Medium: the dispute often settles inside the wallet's own terms | The bank may be banking the **wallet**, not the operator — the interposition question of [§9.7](#97-the-three-hats-kept-distinct) |
| **Account-to-account / open banking** | Direct pull from the customer's bank account under a destination-market scheme | Lower on cards-style chargeback, higher on **authorised-push-fraud** and mandate abuse | The customer's own bank is in the flow; the dispute lands with it |
| **Instant bank transfer / local schemes** | Rail of choice where card acceptance for gambling is restricted or refused | Low reversibility; high finality | Finality cuts both ways: good for the operator, and a reason the customer's bank matters as a complainant venue |
| **Prepaid voucher / cash-in retail** | Anonymous-ish top-up where cards are unavailable or the customer is unbanked | High finality; weak identity linkage | The identity question of [§10](#10-the-aml-and-payment-rails-angle) in its sharpest form |
| **Crypto / alt rails** | Deposit and, especially, **withdrawal** rail | Irreversible outbound; no chargeback discipline at all | The "deposit by card, withdraw to crypto" pattern below |

**Three structural consequences the guide will state, and one it will not.**

1. **Reversibility determines who carries the loss.** On a reversible rail, the merchant (and its reserve) absorbs the chargeback; on an irreversible rail, the customer does. That asymmetry is the commercial reason operators steer payouts and is the reason an operator's **payout** rail mix is a different risk from its **deposit** rail mix.
2. **The "deposit with a card, withdraw to crypto" pattern is a monitoring signal, not an accusation.** Where an account funds by one rail and pays out by another, the pattern is consistent with a variety of explanations, including ordinary rail-availability constraints. It is described here as a pattern a monitoring system is built to notice — never as evidence about this operator, for which the guide has no such data.
3. **Where a destination market prohibits its payment institutions from processing gambling transactions for unlicensed operators, the rail question becomes a legality question.** Several markets operate regimes of that shape (the "PSD2-style" prohibition on gambling transactions is the usual shorthand). The guide attributes the prohibition to **the destination market's own payment-services regime** and does not attribute any specific prohibition to any named authority it has not sourced ✅; the point for the bank is that in such markets the rail, not the bet, is the regulated act.
4. **What the guide will not state:** that this operator uses any particular rail, in any particular proportion, in any particular market, absent a source. **The rail inventory is a gap**, and the gap is recorded here as the finding ([§9.5](#95-the-verification-and-monitoring-checklist) makes it a checklist item).

### 9.3 The merchant-risk classification, and why it applies

**This subsection describes the bank's and the acquirer's own classification framework — not the operator's conduct.** Gambling is treated as a **high-risk merchant category** on the acquiring side for reasons that are structural and long-established, and the reasons are what a bank should be able to state in its own file:

| Reason the classification exists | Mechanism | Who bears the consequence |
| --- | --- | --- |
| **Category-level scheme rules** | The card networks treat gambling as a restricted/high-risk category in their own scheme rules and assign it a merchant category code, commonly referenced as **MCC 7995** ⚠ (cited here **by number only**, as the networks themselves publish it; this pass did not read a scheme rulebook) | The acquirer, through registration requirements, reserves and monitoring |
| **Chargeback and dispute exposure** | High transaction counts, high decline and reversal rates, and dispute reasons specific to a service delivered over time | The merchant first, the acquirer's reserve second, the acquirer's portfolio economics third |
| **Regulatory expectations on the bank** | The **bank's own** prudential and financial-crime supervisors, in whatever jurisdiction licenses it, expect risk-based treatment of higher-risk customers and sectors; that expectation is the bank's to satisfy and is not sourced in this guide to any named regulator as a specific rule | The bank's own supervisory relationship — not the merchant |
| **Jurisdictional mismatch between acquirer and customer** | The acquirer may be licensed in one market while the merchant's customers and flows are in many others | The acquirer, which has limited tools to assess legality in each destination market |
| **Reversibility and payout-rail abuse** | The asymmetry of [§9.2](#92-the-deposit-and-withdrawal-rail-mix) | The operator's or acquirer's fraud-loss line, depending on the rail |
| **Refund and payout complaint volume** | Player complaints about unpaid winnings or account closure are a sector feature that shows up as disputes, chargebacks and regulatory complaints | The merchant's reputation and the acquirer's dispute metrics |

**The classification is a statement about the transaction profile, not about the business.** A high-risk merchant classification is what an institution applies to a category of activity; it is not a finding that a given operator has done anything wrong, and this guide does not make such a finding. The classification exists whether the operator is the most compliant licensee in the sector or the least.

**Where the machinery is owned elsewhere.** The fraud-detection and merchant-risk machinery — velocity rules, chargeback analytics, merchant monitoring, the cross-referencing of transaction data against risk typologies — is the subject of **[Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)** and is cross-referenced, not re-taught, here. This section states the *category-level* facts a gambling-merchant file needs; that guide owns the *instrumentation*.

### 9.4 The customer-jurisdiction risk

**This is the analytical heart of the section, and the single most important licensing-derived risk in a gambling-operator file.**

The rule is [§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)'s: **a gambling licence is jurisdiction-scoped.** Caecus N.V.'s licence was issued by the **Curaçao Gaming Authority** ✅ and authorises activity under Curaçao's statute. It does not authorise serving customers in the Netherlands, Great Britain, France, Spain, Italy, Greece, Poland, Nigeria, Kenya, Uganda, Liberia, Russia or anywhere else. Those markets are governed by their own authorities, and **only the authority of a market can determine whether serving that market is lawful** — the bank attributes, it never decides.

The bank's task is therefore a **mapping exercise**, not a yes/no question:

1. **Where does the licence permit business?** For this operator: Curaçao ✅, plus whatever local licences each market's own register establishes (§5.5, mostly leads and absences).
2. **Where are the customers and the payment flows actually?** Not established for this operator in this pass ❌ — no transaction geography, no customer distribution, no per-market revenue is public ([§4.4](#44-performance-claims-the-company-makes-about-itself), [§2.6](#26-what-could-not-be-established-about-this-identity--bluntly) item 6).
3. **What is in the gap?** Market by market, and dated, on the evidence [§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached) reached:

| Where, on the evidence of this pass | Position recorded | Consequence for a bank's file |
| --- | --- | --- |
| **Curaçao** | **Licensed** — Caecus N.V., OGL/2024/1262/0493, granted 7 Nov 2024, Active ✅ | The one register-grade licensing anchor |
| **Netherlands** | **No licence — enforcement concluded** ✅ (Ksa, 2019) | Precedent that an unlicensed market's exposure is real and can end in a published instrument and a fine ([§6.1.1](#61-concluded-matters)) |
| **Great Britain** | **No action published; none confirmed or denied** ✅ (documented negative) | The honest reading is "not established in either direction", not a revocation ([§5.3](#53-great-britain-a-licence-that-was-not-there)) |
| **Liberia** | **Local licensee's licence revoked; complaint against the brand dismissed** ✅ (NLA, 22 June 2026) | The brand-licensed-locally model's failure mode, and the identity lesson of [§9.1](#91-identifying-the-counterparty-which-entity-is-the-bank-onboarding) |
| **Ukraine** | **Sanctions instrument designates 1XCorp N.V.** ⚠✅ (instrument identified; annex not read at source) | A sanctions dimension that a bank's own screening, not this guide, must resolve |
| **Spain, Mexico, Serbia, Peru, Ghana, Uganda, Kenya, Ireland, Brazil** | **Reported licensed (leads)** ⚠ — aggregator or encyclopaedia sourced, not verified at the authority | Each is a **re-verification task at that market's register**, not a fact |
| **France, Italy, Greece, Cyprus, Czechia, Poland, Lithuania, Estonia** | **Appears on published unauthorised-operator or blocking lists** ⚠ — aggregator stating it reads those regulators' lists | If it holds, revenue from those markets is exposed to enforced exit; the bank must read the lists itself, dated |
| **Russia; "more than 35 markets"** | **Not established** ❌ / **company claim** ⚠ | Absence, and a claim with no register-by-register support |
| **Every market not researched here** | **Nothing established by design** ❌ | The guide does not assert a market's silence or a market's action |

4. **Why the gap is a banking risk and not merely a legal one.** Revenue booked from customers in markets where the operator holds no licence is revenue exposed to **enforced market exit** — and, in regimes that prohibit their own payment institutions from processing for unlicensed operators ([§9.2](#92-the-deposit-and-withdrawal-rail-mix)), to the exposure of the intermediaries that carried it. That is a **revenue-durability** question before it is a compliance question, which is what makes it a banking question.

**The correct question, in one sentence.** Not "is it licensed?" but "**which of the operator's revenue-bearing markets are covered by a licence, as at what date, and what proportion of revenue sits in the uncovered remainder?**" ([§5.6](#56-the-licence-quality-spectrum-by-class-and-jurisdiction).)

### 9.5 The verification and monitoring checklist

The Curaçao Gaming Authority operates a **public certificate/register portal** through which a counterparty can verify a brand's operator, company number, licence number, grant date and status ✅ ([§2.5](#25-what-the-identity-gate-establishes-in-one-line-each)). That is the guide's recommended verification route for any bank screening this brand, and it is a *better artefact than a website footer*. The checklist below converts this guide's method into a file a bank can run.

| # | Check | Where to look | What counts as evidence | Status in this guide |
| --- | --- | --- | --- | --- |
| 1 | Legal entity of the operator | Authority certificate portal (`cert.gcb.cw`) + commercial register | Certificate naming brand, entity, company number, licence number, grant date, status ✅ | ✅ obtained — Caecus N.V., 163779 |
| 2 | Licence class, scope, term, status | Same certificate; note the **"OGL/…" post-2024 numbering** as a dating device ✅ | The certificate's own status field, **read at a stated date** | ✅ Active at the September 2026 retrieval |
| 3 | Whether any licence exists in the **destination market** | That market's own register, unauthorised-operator list and blocking list | The regulator's own page, dated | ⚠ leads and absences per market ([§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached)) |
| 4 | Published regulatory actions | Each authority's register (in Great Britain: a regulatory-actions register reporting **136 records** at retrieval ✅, plus the FOI disclosure log) | A named instrument, not a press summary | ✅ the register exists; ⚠ not searched to exhaustion in this pass |
| 5 | Sanctions exposure | The bank's own sanctions screening, against designation instruments | The instrument itself, not a database entry alone | ⚠ instrument identified, annex not read ([§6.1.3](#61-concluded-matters)) |
| 6 | Merchant of record and settlement-account holder | The merchant's own documents + the acquirer | Contract, settlement instructions, account-name evidence | ❌ not public; must be obtained at onboarding |
| 7 | Ultimate beneficial ownership of the licensee | Commercial register, licensing file, or the bank's own UBO diligence | A register or a filing, not journalism | ❌ not established ([§2.6](#26-what-could-not-be-established-about-this-identity--bluntly)) |
| 8 | Sponsorship and legitimacy claims | The **rightsholder's own announcement** | The sponsor's own publication, dated | ✅ for FC Barcelona (3 July 2019; renewal 1 July 2024); ⚠ for all others ([§7.2](#72-verified-sponsorships--the-sponsors-own-announcement)) |
| 9 | Deposit/withdrawal rail inventory and geography | The merchant's own MI, mapped to the licence map of item 3 | Merchant-provided data, not a website list | ❌ not available ([§9.2](#92-the-deposit-and-withdrawal-rail-mix)) |

**A note on what a checklist cannot fix.** Items 1, 2 and 8 are obtainable from public sources and are now established. Items 6, 7 and 9 are **not public at all** and can only come from the merchant. A diligence file that treats a public certificate as a substitute for items 6–9 has done the easy half and skipped the half the merchant must supply.

### 9.6 Ongoing monitoring: what the bank watches after onboarding

A licence status is a **point-in-time reading** ✅ ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)), so onboarding is not the end of the exercise. The monitoring signals this guide's own material identifies, each with the guide's own reasoning labelled:

1. **Licence status re-pull at each periodic review** — the same certificate portal, re-read and re-dated. Reasoning: a status field can move, and the guide's reading is September 2026 only.
2. **Additions to unauthorised-operator and blocking lists** — the market lists of [§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached). Reasoning: a listing is the earliest published signal of enforced exit.
3. **Sanctions and designation changes** — the bank's own screening, since the Ukraine instrument shows a designation touching an entity associated with this business name ✅⚠.
4. **Sponsorship termination** — a sponsor's exit is a **dated** negative signal and is more informative than the signing ([§7.4](#74-what-sponsorship-buys--the-guides-own-analysis-labelled) reasoning 2). Reasoning: it dates the moment a sophisticated counterparty re-priced the relationship.
5. **Affiliate spend patterns** — marketing spend passing to a long tail of small counterparties and platforms across many jurisdictions ([§8.2](#82-the-affiliate-layer-and-what-it-means-for-a-bank)). Reasoning: it is the visible footprint of an acquisition layer that reaches markets the licence may not cover.
6. **Rail-mix drift** — a material shift toward irreversible outbound rails ([§9.2](#92-the-deposit-and-withdrawal-rail-mix)). Reasoning: it changes both the dispute profile and the traceability profile.
7. **Volume, decline and dispute metrics against the merchant's own baseline** — the instrumentation owned by **[Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)**, not re-derived here.

### 9.7 The three hats kept distinct

A gambling operator can be three different things to one bank at the same time, and the failure mode in this sector is to diligence one hat and price another. The guide keeps them apart:

| Hat | What the bank is actually exposed to | What the diligence is | What it is **not** |
| --- | --- | --- | --- |
| **(i) Acquirer/merchant client** | Settlement risk, chargeback and reserve exposure, commercial concentration, category-level scheme rules (MCC 7995 ⚠) | Merchant-category underwriting, reserves/holdbacks, volume and dispute monitoring | Not a supervisory judgment, and not evidence about the operator's conduct |
| **(ii) Payments counterparty** | The interposition problem: the bank may be banking the operator's **PSP, wallet or payment intermediary** rather than the operator; the **correspondent-banking** question — whether the flows are ultimately carried over the bank's own rails by another institution — arises whenever the settlement account is not the licensee's | Mapping the payment chain from the customer's bank to the settlement account; identifying every intermediary; confirming the account holder | Not resolved by the operator's own website payment icons, and not resolved by naming the brand |
| **(iii) AML and licensing-diligence subject** | The counterparty's status as a **customer of the bank's financial-crime controls** and as an entity whose market lawfulness the bank must assess | Source of funds and source of wealth; UBO; licence verification per market; market-lawfulness mapping ([§9.4](#94-the-customer-jurisdiction-risk)) | Not a criminal allegation, and not a finding about the operator — it is the bank's own obligation, discharged under the bank's own framework |

**Two consequences of the distinction.** First, the three hats have **different escalation paths and different owners** inside a bank, and a file that answers hat (iii) with the artefacts of hat (i) — a merchant-category approval — has answered a different question. Second, the Liberian record shows that hat (ii) is where the surprises live: a trademark licence to a third company and payments into accounts in the brand's name rather than the licensee's ✅ ([§6.1.2](#61-concluded-matters)).

### 9.8 The de-risking question

Stated neutrally, because it is a real decision with real costs on both sides:

- **Banks do exit high-risk merchant categories**, including gambling, and that exit is a legitimate exercise of risk appetite under the bank's own framework.
- **De-risking is also a supervised expectation, not only a commercial choice**: risk-based frameworks on the bank's own supervisors mean that a bank that cannot evidence its assessment of a higher-risk customer is itself exposed — so "exit because the file cannot be completed" and "exit because the category is unappetising" are different decisions that produce the same outcome.
- **The guide takes no position on which exit is right for a given bank**, because that turns on the bank's own appetite, its own supervisors, and information (§9.5 items 6, 7, 9) that is not public.
- **The guide's synthesis, labelled as the guide's own reasoning:** **for a bank, the operator's licence is the instrument of diligence, and the licensing jurisdiction is the risk variable.** The licence tells the bank *which entity* it is dealing with and *which markets* the counterparty is authorised to serve; the licensing jurisdiction tells it *how much supervision* stands behind that authorisation and *what the supervisor has done when its licensee breached*. Everything else in the file — sponsorship, traffic, brand recognition, scale claims — is context, and context is not diligence.

### 9.9 What is the bank's own, and what it is not

**The following obligations and classification decisions are the BANK'S OWN and are not a judgment about the operator's character or conduct:**

- The **high-risk merchant classification** and its consequences (reserves, holdbacks, enhanced monitoring) are the acquirer's and the bank's decisions under the card networks' published rules ⚠ and under the bank's own risk framework.
- The **risk-based AML treatment** of the counterparty — identification, UBO, source of funds, ongoing monitoring — is the bank's obligation under its own supervisors' frameworks, not a finding about the operator.
- The **assessment of whether the operator may lawfully serve a given market** is made by **that market's authority**, never by the bank and never by this guide ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)).
- The **decision to onboard, price, reserve against, or exit** the relationship is the bank's, under the bank's own appetite.

**No part of this section states, or should be read as stating, that the operator is unlawful, criminal, fraudulent or sanctionable.** Where allegations exist in the public record, they are attributed, dated and kept in [§6.3](#63-reported-but-unverified-matters-kept-out-of-the-record) and [§15](#15-the-claims-audit), and they are not restated here.

---

## 10. The AML and Payment-Rails Angle

*This section is a **cross-reference, not a second guide**. The financial-crime machinery — typologies, detection instrumentation, AML programme design, certification and exam content — is owned by the guides named in [§10.4](#104-where-this-material-is-owned-elsewhere--the-cross-reference) and is not re-derived here. What this section does is state the **sector's** financial-crime profile at the level the standards bodies establish it, and then the **specific diligence questions a gambling operator raises that an ordinary merchant does not**.*

### 10.1 The sector profile, at the level the standards establish it

Online and land-based gambling is characterised by standards bodies and supervisors as a **high-velocity, cash-intensive-adjacent sector** with a characteristic set of vulnerabilities. The description below is the **sector's** profile, presented as structural context:

| Sector vulnerability | Why it attaches to the sector | Status |
| --- | --- | --- |
| **Customer identity** | Accounts can be opened and funded remotely and at volume; the identity behind the account is the control that everything else depends on | Sector characteristic (unmarked) |
| **Source of funds / source of wealth** | Deposits are high-frequency and small at the tail and large at the head; the funding origin of a single large deposit is not visible from the stake itself | Sector characteristic (unmarked) |
| **Third-party funding** | Stakes can be funded from accounts, cards, wallets or vouchers that are not the account-holder's, and payouts can be withdrawn to destinations that are not the depositor's | Sector characteristic (unmarked); see the rail asymmetry in [§9.2](#92-the-deposit-and-withdrawal-rail-mix) |
| **Value transfer / layering via the account** | An account can function as a two-way value-transfer channel — in as stake, out as winnings — without any underlying gambling intent | Sector characteristic (unmarked) |
| **Bonus and multi-accounting abuse** | Promotional mechanics create an incentive for duplicate identities ([§4.3](#43-the-structural-economics-of-an-online-operator)) | Sector characteristic (unmarked) |
| **Payout destination risk** | Payouts to a third party's account or to an irreversible rail are the sector's characteristic outbound risk | Sector characteristic (unmarked) |

**Three honest limits on the paragraph above.** (i) It is a **sector** profile, and **no part of it is a finding about this operator** — the guide asserts nothing about the operator's customers, funding, or AML controls beyond what is cited elsewhere. (ii) The characterisation is attributed to the **standards bodies and supervisors collectively** ("FATF-style"); **this pass did not read an FATF typology or mutual-evaluation report at source**, so no specific body, report or date is cited for it ⚠. (iii) The absence of an AML enforcement instrument against this operator in this guide's §6 record is **an absence of a sourced instrument in this pass**, not a clean bill of health and not an implication of one either — either reading would be a claim the sources do not carry.

### 10.2 The diligence questions an ordinary merchant does not raise

Some questions in a gambling file have no counterpart in a normal commercial onboarding. Each row below states the question, why it is gambling-specific, and where this guide takes it up.

| # | The question | Why it is gambling-specific | Where it goes |
| --- | --- | --- | --- |
| 1 | **Who is the licensed entity, and is the account holder that entity?** | The brand is not a legal person; the settlement account can bear the brand's name rather than the licensee's ✅ | [§9.1](#91-identifying-the-counterparty-which-entity-is-the-bank-onboarding) |
| 2 | **Which markets is the operator authorised to serve, and where are its customers?** | The licence is jurisdiction-scoped, and the customer base is not constrained by it | [§9.4](#94-the-customer-jurisdiction-risk), [§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding) |
| 3 | **What is the funding provenance of a high-value deposit, and is the withdrawer the depositor?** | Two-way value transfer is native to the account, not an exception to it | [§10.1](#101-the-sector-profile-at-the-level-the-standards-establish-it) row 3 |
| 4 | **Which rails fund deposits, and which rails pay out?** | The asymmetry of reversibility is where the loss lands ([§9.2](#92-the-deposit-and-withdrawal-rail-mix)) | [§9.2](#92-the-deposit-and-withdrawal-rail-mix) |
| 5 | **Is the operator's own AML/player-protection obligation being discharged, and who supervises it?** | The operator is itself an obliged entity in most regulated markets; the bank's risk includes the quality of that supervision | [§5.6](#56-the-licence-quality-spectrum-by-class-and-jurisdiction) (supervisory intensity) |
| 6 | **Is the market itself one where processing gambling transactions is restricted for the bank's own payment clients?** | In such markets the **payment**, not the bet, is the regulated act | [§9.2](#92-the-deposit-and-withdrawal-rail-mix) point 3 |
| 7 | **Are there sanctions designations touching entities associated with the business name?** | A designation touching a related entity is a screening event, whether or not the onboarded entity is named | [§6.1.3](#61-concluded-matters) |
| 8 | **How much of the revenue is durably licensed revenue?** | The mismatch between the licence map and the customer map is a **revenue-durability** question | [§9.4](#94-the-customer-jurisdiction-risk) step 4 |

**The structural point these eight share.** For an ordinary merchant, the bank's AML question is mostly about the **customer's** customers and the merchant's own controls. For a gambling operator, the bank's question set splits: the operator is simultaneously (a) a customer of the bank's controls, (b) the operator of a system that itself performs customer due diligence on millions of people, and (c) an entity whose **market footprint** may be outside the scope of its licence. Those are three different supervisory relationships, and the sector's risk is concentrated in the seam between them.

### 10.3 The jurisdictional seam, and why it is the AML-relevant fact

The financial-crime relevance of the licensing analysis is not the licence's existence; it is what the licence **omits**. An operator lawfully licensed in Curaçao ✅ that takes customers in markets where no local licence is established for it (the leads and absences of [§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached)) is, in each such market, carrying on a regulated activity that may be unlicensed **as that market's authority determines**, not as this guide or the bank determines. Where that is the case, the local regime's AML obligations — customer due diligence, suspicious-transaction reporting, record-keeping — do not attach to the operator in that market, and the bank's counterparty is, from that market's supervisory perspective, an unsupervised one.

**What the guide does and does not say here.** It says that the seam exists, that it is a structural feature of a multi-market offshore-licensed operation, and that a bank's file should record the **per-market** position. It does **not** say that any particular market's activity is unlawful, because only that market's authority can say so, and this pass did not read a determination for most markets ✅⚠.

### 10.4 Where this material is owned elsewhere — the cross-reference

The following guides in this repository own the material and are cross-referenced, not reproduced (**filenames verified present on disk in this pass** ✅):

| Guide | What it owns | How this guide uses it |
| --- | --- | --- |
| **[Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)** | Fraud typologies and detection instrumentation at scale — velocity, network and behavioural analytics, the merchant-risk monitoring build | §9.3 and §9.6 point to it for the **instrumentation** applied to a gambling merchant; it is not re-derived |
| **[AML Certifications & Exam Content](aml_certifications_exam_content_guide.md)** | AML programme design, the obliged-entity framing, the certification and exam content that encodes the sector expectations | §9.3 (regulatory expectations) and §10.2 (the diligence question set) reference it rather than restating it |
| **[Payment Rails](payment_rails_guide.md)** | The rail-by-rail map — card, A2A, instant transfer, wallet, vouchers, local schemes, crypto — their mechanics, settlement and dispute characteristics | §9.2's rail table is stated at category level only; the **mechanics** belong to that guide |

**A deliberate restraint.** This section could be expanded indefinitely with typology detail, and that is precisely why it is kept short: the machinery is another guide's, and the gambling-specific content is the eight questions of [§10.2](#102-the-diligence-questions-an-ordinary-merchant-does-not-raise) and the seam of [§10.3](#103-the-jurisdictional-seam-and-why-it-is-the-aml-relevant-fact).

### 10.5 The sector pattern, mapped to what the bank actually watches

Two sector features of [§10.1](#101-the-sector-profile-at-the-level-the-standards-establish-it) have a direct monitoring expression, and the mapping is this section's practical content:

| Sector feature | What it looks like from the bank's side | Where the watch belongs |
| --- | --- | --- |
| **Third-party funding and payout-destination risk** | Funding inflows that do not match the ordinary profile of the relationship; a payout rail that differs from the funding rail | [§9.2](#92-the-deposit-and-withdrawal-rail-mix) consequence 2; [§9.6](#96-ongoing-monitoring-what-the-bank-watches-after-onboarding) item 6 |
| **The acquisition layer's cross-border tail** | Marketing spend spreading to many small counterparties and platforms across many jurisdictions | [§8.2](#82-the-affiliate-layer-and-what-it-means-for-a-bank); [§9.6](#96-ongoing-monitoring-what-the-bank-watches-after-onboarding) item 5 |
| **Two-way value transfer through the account** | Settlement or payout flows whose destination account is not the licensee's | [§9.1](#91-identifying-the-counterparty-which-entity-is-the-bank-onboarding); [§9.7](#97-the-three-hats-kept-distinct) |

**The caveat that travels with this table.** These are patterns a monitoring build is designed to surface for a merchant of this category; **they are not observations about this operator**, and this guide holds no transaction data for it. The line between "the sector's characteristic pattern" and "this operator's pattern" is exactly the line [§10.1](#101-the-sector-profile-at-the-level-the-standards-establish-it) limit (i) draws, and a file that blurs it has converted a typology into an accusation.

---

## 11. The Comparison Set

*The guide's rules forbid it from ranking named operators on its own authority. What it can do is describe the landscape **by tier and licence class** — the dimensions that actually drive a bank's risk rating — and then state this operator's position **only as far as sources carry it**. Where they do not, the absence of a basis is itself the finding.*

### 11.1 The four tiers, by licence class

| Tier | What it is | Where the licence sits | Bank-relevant characteristic |
| --- | --- | --- | --- |
| **T1 — The nationally licensed operator** | An operator holding the **destination market's own remote-gambling licence**, usually through a locally incorporated entity | In the market it serves, from that market's regulator | Highest supervisory visibility in the markets that matter to the bank: capital, reporting, player-protection and AML conditions enforceable locally; the licence is an instrument the bank can read at the issuing authority |
| **T2 — The offshore-licensed multi-market operator** | An operator holding a licence in a jurisdiction that is **not** the customer's home market, and serving many markets from it | Offshore (for example, Curaçao ✅), often alongside local licences in some markets | The **customer-jurisdiction mismatch** of [§9.4](#94-the-customer-jurisdiction-risk) is native to this tier; the licence is real, and its scope is narrow relative to the customer base |
| **T3 — The white-label brand** | A brand riding on **another entity's licence**; the brand itself is not licensed in that market | In a third party's name, under a contractual arrangement | The entity a bank is onboarding may not be the entity the local regulator supervises at all; **a brand in this tier holds nothing locally** ([§1.4](#14-the-decoder), [§5.6](#56-the-licence-quality-spectrum-by-class-and-jurisdiction)) |
| **T4 — The grey/black-market operator** | An operator serving a market with **no licence there** — *grey* where the market's authority has not formally branded the activity unlawful, *black* where it has | Nowhere, for that market | The licence map has a hole where the revenue is; enforcement remedy is typically market exit or blocking ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)) |

**Two clarifications the tiers invite and the guide refuses.** First, the tiers are **not a quality ladder**: a T1 operator is not morally better than a T2 operator, it is *differently supervised* in the markets it serves, and a bank's risk rating follows the supervision, not the tier's ordinal position. Second, the tiers **overlap in one operator**: the same corporate family can be T1 in one market, T2 in another, T3 through a trademark-licensed local partner in a third, and absent from a fourth — the architecture the Liberian ruling documents ✅ ([§6.1.2](#61-concluded-matters)).

### 11.2 The licensed-versus-grey-market line, and why it is the line that matters

For a bank, the distinction that carries weight is **not** "on-shore versus off-shore" and **not** "reputable brand versus unknown brand". It is **licensed in the market where the customer and the payment flow are** versus **not**. Three reasons:

1. **It is the only line an authority can draw with an instrument.** A licence, an unauthorised-operator list, a blocking order and a fine are all market-specific acts ([§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached)). Marketing spend, brand recognition and sponsor lists are not, and cannot substitute.
2. **It is the line that determines whether the revenue is durable.** Revenue from an unlicensed market is exposed to enforced exit, and to the exposure of the payment intermediaries that carried it ([§9.4](#94-the-customer-jurisdiction-risk) step 4).
3. **It is the line the sector's own practice tests.** The Gambling Commission's quoted 2019 position — that bodies engaging in sponsorship and associated advertising arrangements with an **unlicensed operator** may be liable for **advertising unlawful gambling** under section 330 ✅⚠ ([§7.3](#73-sponsorships-reported-flagged-or-ended)) — is an authority stating that the licensed/unlicensed line runs through the *advertiser* as well as the operator.

### 11.3 This operator's position, on the sources this pass reached

| Question | Answer available | Status |
| --- | --- | --- |
| Which tier is the dot-com operation in? | **T2** on the register-grade fact: an offshore (Curaçao) licence held by Caecus N.V., serving markets that are not its licensing jurisdiction ✅ | ✅ supported for the licensing structure; the customer footprint is not established |
| Is there a **comparative** claim — better/worse than named competitors? | **No.** No source read in this pass compares this operator to any named competitor on licence quality, supervisory record, complaint volume, market share or conduct | ❌ **not established — and the absence is the finding** |
| Is it T3 anywhere? | The white-label **explanation** for the 2019 UK-facing site (reported as running under a technology partner's Gambling Commission licence) is **reported, not adopted**; the Liberian record shows a **trademark-licence** local-partner structure, which is a neighbouring but not identical arrangement ✅⚠ | ⚠ reported/reasoned, not register-verified |
| Is it T4 anywhere? | The **eight regulators' unauthorised-operator or blocking lists** on which an aggregator reports the brand are **leads** ⚠; only the market that publishes the list can determine the position, and this pass did not read each list | ⚠ reported; **not verified individually** |
| Is it T1 anywhere? | Reported local licences in Spain, Mexico, Serbia, Peru, Ghana, Uganda, Kenya, Ireland and Brazil are **leads** ⚠; none verified at the market's authority in this pass | ⚠ leads; ❌ not verified |

**The finding, stated plainly: there is no comparative basis.** This pass established **one** register-grade licence (Curaçao) ✅, **one** concluded enforcement (Netherlands) ✅, **one** documented negative (Great Britain) ✅, **one** concluded local-licensee proceeding (Liberia) ✅, and **one** identified sanctions instrument (Ukraine) ⚠✅. Everything else is a lead, an absence or a company claim ([§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached), [§5.6](#56-the-licence-quality-spectrum-by-class-and-jurisdiction)). A guide that produced a ranking from that base would be inventing the comparison, which is the operation this guide's rules exist to prevent.

### 11.4 The one declared boundary comparison — a betting operator is not a broker

The repository's nearest adjacent persona is the trading platform, and the comparison is declared in [§1.6](#16-the-boundary-declared-by-name) as the guide's only boundary case. It is a **structural** distinction (unmarked), and it turns on the customer's relationship to the money:

| Dimension | A betting operator | A broker / investment platform |
| --- | --- | --- |
| **The customer's payment in** | A **stake**: the operator's own money the moment it is placed; not held for the customer | A **client asset**: held for, or on behalf of, the customer |
| **The customer's payment out** | **Winnings**: the operator's contractual liability to pay, settled from its own funds | A **custody balance** returned to its owner |
| **Insolvency treatment** | The customer is an unsecured creditor for unpaid winnings | Client assets are generally segregated, with distinct insolvency treatment |
| **The conduct regime** | Gambling regulation: licensing, advertising, player protection | Securities/investment conduct regulation: suitability, best execution, market conduct |
| **The authority** | A gambling regulator (Curaçao Gaming Authority ✅, Kansspelautoriteit, Gambling Commission, DGOJ, and their peers) | A securities/markets regulator |
| **The banking classification** | High-risk merchant category; acquiring and settlement risk | Brokerage/custody and client-money risk |

**Why the distinction is worth one table and no more.** A file that maps a betting operator onto a broker's control framework will look for client-asset segregation and custody balances that do not exist in this business, and will miss the two things that do: the **customer-jurisdiction mismatch** of [§9.4](#94-the-customer-jurisdiction-risk) and the **rail reversibility asymmetry** of [§9.2](#92-the-deposit-and-withdrawal-rail-mix). The comparison is cross-referenced to **[Online Investment Trading Platforms](online_investment_trading_platforms_guide.md)**, which owns it, and is not developed further.

---

## 12. The Data and Analytics Angle

*Short section by design. The gambling **data and analytics** material is owned by two guides in this repository, cross-referenced here and **not re-derived** ([§1.6](#16-the-boundary-declared-by-name)). What this section does is say what that material means for a bank.*

### 12.1 Where the material is owned (filenames verified present on disk in this pass ✅)

| Guide | What it owns | This guide's pointer |
| --- | --- | --- |
| **[Gambling Datasets](../technology/data/gambling_datasets.md)** | The gambling data landscape: the datasets themselves, their structure and provenance, betting and odds data, and how gambling data is generated and published | The source for **what a gambling operator's data estate looks like**; not reproduced here |
| **[Gaming Data Warehouse & Bet Recommendation](../technology/data/gaming_dw_bet_recommendation.md)** | The data-warehouse model behind an operator: the **player/bet/wallet** schemas, RTP and hold computation, bet-recommendation analytics, and the **KYC flags** carried inside those models | The source for **how an operator's own transaction data is structured**; not reproduced here |

Both are cross-referenced in this guide's title block and in [§1.6](#16-the-boundary-declared-by-name) as the owners of this layer. Nothing in the two paragraphs below adds facts to them.

### 12.2 What that material means for a bank — four lines, no more

1. **The operator's own data is a diligence artefact, on request.** The player/bet/wallet model owned by the Gaming Data Warehouse guide is the shape in which an operator's own MI — deposit and payout rail mix ([§9.2](#92-the-deposit-and-withdrawal-rail-mix)), customer geography ([§9.4](#94-the-customer-jurisdiction-risk)), and the identity fields behind each account — actually exists. A bank's information request should be written against that model, not against a website.
2. **The KYC flags inside the operator's data model are the operator's controls, not the bank's.** The presence of KYC fields in an operator's warehouse is evidence that controls exist in the system; it is not evidence that they are discharged for any given customer, and this guide draws no conclusion about this operator's controls from it.
3. **Data-derived monitoring of the relationship is the instrument the bank already has.** The merchant-monitoring build — volume, rail mix, decline and dispute patterns, counterparty tails — is owned by **[Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)** and applied to a gambling merchant through [§9.6](#96-ongoing-monitoring-what-the-bank-watches-after-onboarding).
4. **The data angle is not an alternative to the licensing angle.** Analytics can describe the shape of the flows; only a register can establish the entity and the licence ([§9.8](#98-the-de-risking-question) synthesis). The two are complements, and the guide's whole method depends on keeping them apart.

### 12.3 What the data angle does not do

The two data guides can describe an operator's estate in detail; they cannot answer the questions this banking guide is built on. Stated so the boundary is unambiguous:

- **It does not establish the entity.** No dataset names the licence holder; only a register does ([§9.1](#91-identifying-the-counterparty-which-entity-is-the-bank-onboarding)).
- **It does not establish the licence's scope.** Traffic and volume data describe reach, not authorisation; reach is not a licence ([§9.4](#94-the-customer-jurisdiction-risk)).
- **It does not establish market lawfulness.** No dataset substitutes for the market authority's own list or instrument ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)).
- **It does not establish revenue durability.** It can describe the flows; it cannot say whether the revenue behind them is licensed revenue ([§9.4](#94-the-customer-jurisdiction-risk) step 4).
- **It is not a compliance artefact on its own.** Analytics support a merchant file; they do not complete it.

**The order in which a bank's file should therefore be read:** register and licence first ([§9.1](#91-identifying-the-counterparty-which-entity-is-the-bank-onboarding), [§9.5](#95-the-verification-and-monitoring-checklist)), market-by-market licence map second ([§9.4](#94-the-customer-jurisdiction-risk)), data and monitoring third ([§12.2](#122-what-that-material-means-for-a-bank--four-lines-no-more)). That ordering is this guide's method in one sentence, and it is the reason [§12](#12-the-data-and-analytics-angle) is the shortest section in it.

---

## 13. The Anti-Patterns

*This section converts the guide's findings into a checklist. Each entry is a **symptom** a diligence file exhibits, the **cause** that produces it, and the **guardrail** that prevents it. Entries 1–6 are the general set that any gambling-operator file needs; entries 7–10 are raised specifically by this subject and by the public record this pass reached. Everything here is the guide's own reasoning, labelled as such, and grounded in the sourced material of [§2](#2-the-identity-and-the-group-structure), [§5](#5-the-licensing-position) and [§6](#6-the-regulatory-record).*

### 13.1 How to read the entries

The anti-patterns are written as **failures of a file**, not as criticism of a company. A bank that onboards a brand instead of a licensee, or that records a licence claim without a dated register read, has made a documentary error with a credit and a financial-crime consequence — and the error is fixable *before* signing, which is the entire point of listing it.

Each entry therefore carries four things: the **symptom** as it appears in a real onboarding or periodic-review file; the **cause**, which is nearly always a shortcut taken at the identity or licensing step; the **guardrail**, which is the artefact or control that closes it; and the **underlying sourced fact** in this guide. The guardrails are the same ones the bank's own diligence would produce from [§9](#9-the-banks-view--the-merchant-and-payment-counterparty-assessment) and [§10](#10-the-aml-and-payment-rails-angle); they are restated here in incident form.

A useful habit: read each symptom as a question a reviewer would ask of the file — *"which company number is this?"*, *"when was the register read?"*, *"is that a finding or an allegation?"* — because every anti-pattern below survives only until someone asks the right question out loud.

### 13.2 The six core anti-patterns

| # | Symptom in the file | Cause | Guardrail |
| --- | --- | --- | --- |
| **1** | **The brand is treated as a legal entity.** The counterparty field reads "1xBet"; the contracting entity is blank or is a trading name; no company number appears anywhere in the file. | The brand is what the website says, what the merchant presented itself as, and what the press writes — it is the *only* name a casual search returns, so it becomes the file's name by default ([§1.2](#12-the-single-most-important-distinction-in-this-guide)). | **Contract with a named legal person and record its company number.** The licensing authority's own certificate names **Caecus N.V.**, Curaçao company number **163779**, as the operator of 1xbet.com under licence **OGL/2024/1262/0493** ✅ ([§2.1](#21-the-brand-is-not-an-entity--the-register-entry-that-names-the-licensee)). The mirror image of this error, performed by a complainant, is the Liberian tribunal's dismissal of a complaint against "1 X BET" because a brand cannot be sued ✅ ([§6.1.2](#61-concluded-matters)). |
| **2** | **A licence claim is accepted without checking the register.** A licence number, a flag icon or a regulator's logo in a website footer is recorded as "licensed"; no certificate, entry or instrument is attached and no retrieval date is recorded. | A website footer is free text, and a brand can carry several licence numbers belonging to several entities and several periods, none of which is the entity in front of the bank. Marketing presents licensing as a property of the brand. | **Verify at the authority's own artefact, dated, and attach it.** The Curaçao Gaming Authority serves a per-brand **Certificate of Operation** through its certificate portal (`cert.gcb.cw`) that resolves a domain to an operator, a company number, a licence number, a grant date and a status ✅ ([§5.1](#51-what-is-held-verified-at-the-authoritys-own-record)). A licence number's *format* is itself a dating device: the post-2024 Curaçao **"OGL/…"** series is not the older **"…/JAZ…"** series ✅ ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)). |
| **3** | **A licence is assumed to travel across borders.** "Licensed in Curaçao" is recorded, and the market list in the file is then left as the operator's own "we operate in 70+ markets" statement. | The sector's own vocabulary encourages it — an "international licence" sounds global — and the operator's marketing treats reach as if it were authorisation. | **Treat scope as per-market and dated.** The verified licence authorises offering games of chance **under the Curaçao statute** ✅; it does not and cannot authorise serving a Dutch, British, Spanish, French, Irish or Nigerian customer ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)). A licensing map is *market — authority — instrument or listing — date read*, never "licensed/unlicensed" ([§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached)). |
| **4** | **The marketing entity is onboarded rather than the merchant of record.** The bank contracts with a company that appears on the website, or with the affiliate or franchise entity that introduced the deal, while settlement flows to a different company. | The entities that *market* the brand are the ones that make contact; the entity that must be identified sits on the settlement side and is not public ([§2.2](#22-the-entity-and-brand-map-with-the-source-for-each-relationship) rows 7–8). | **Identify the entity that contracts with acquirers and receives settlement, in writing, from the merchant — not from a website.** The Liberian record shows the pattern concretely: a local licensee, a **trademark licence** to a third company registered in Cyprus, and payment flows attributed to accounts in the **brand's own name** rather than the licensee's ✅ ([§6.1.2](#61-concluded-matters)). |
| **5** | **Sponsorship is used as a proxy for regulatory standing.** The file notes a top-tier club partnership and the risk narrative softens; "they sponsor Barcelona, so they must be regulated" appears in a memo. | Sponsorship is the sector's principal purchase of *perceived* legitimacy, and it is genuinely public, dated and reassuring in tone ([§7.4](#74-what-sponsorship-buys--the-guides-own-analysis-labelled)). | **Read sponsorship as marketing spend and as a dated signal, never as a licence.** The FC Barcelona relationship is verified at the sponsor's own announcements of **3 July 2019** and **1 July 2024** ✅, and it establishes that money changed hands and that a reputable rightsholder was willing to be associated with the brand — nothing about licences. Sponsorship does not create a licence or move a licensing boundary ([§7.4](#74-what-sponsorship-buys--the-guides-own-analysis-labelled) point 5). |
| **6** | **An allegation is treated as a finding — or a finding as an allegation.** A press-reported investigation is written into the risk paragraph as a fact; alternatively, an authority's concluded decision is softened to "reported" and lost. | Both directions are common: some sources seek impact and state allegations flatly; others are cautious about everything and therefore blur the difference between a Ksa decision and a trade-press sentence. | **Encode the epistemic status in the sentence.** A concluded instrument is written with authority, instrument, date, parties and outcome; an allegation is written as "*X is reported by Y to…*" and never as "*X is…*" ([§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under) rules (i)–(iv)). The Dutch matter is the archetype of a **finding** (administrative fines of €200,000 on each of two named companies, decision on objection of 7 May 2019 ✅); the Great Britain "licence revoked" claim is the archetype of an **unestablished** one, contradicted in structure by the regulator's own refusal to confirm or deny ✅ ([§6.3](#63-reported-but-unverified-matters-kept-out-of-the-record) item 1). |

**Two further failure modes sit underneath all six.** First, **undated reading**: a register read that carries no date cannot be re-performed and cannot be reconciled with a later status, yet the Curaçao certificate states a status *at retrieval*, not a term ✅ ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)). Second, **single-source closure**: closing a question because one source answered it, when the guide's own method requires the authority's artefact for anything regulatory ([§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under) rule (i)).

### 13.3 Four more anti-patterns this subject raises

These are the additions the material itself demands — each is visible in the partial guide's own sourced record rather than imported from general practice.

| # | Symptom in the file | Cause | Guardrail |
| --- | --- | --- | --- |
| **7** | **The domain estate is treated as the licensed perimeter.** The onboarding note says "the licensed site is 1xbet.com" and stops; the other domains, language versions and app surfaces the same brand operates are unmapped. | A licence certificate resolves *one* domain, and diligenced files are usually built around the URL the merchant put in the deck. | **Map the whole estate, and check each domain against the certificate series.** The same licensee, company number and licence cover at least **1xbet.com** and **1x-bet.com** ✅ ([§2.1](#21-the-brand-is-not-an-entity--the-register-entry-that-names-the-licensee)); any domain *not* named on a certificate is outside the verified perimeter until evidenced, and the Curaçao register was consulted per brand rather than enumerated, so no negative finding is asserted about other domains ⚠ ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)). |
| **8** | **Regulatory silence is read as clearance.** "No action found against them in market X" is written as approval, and the market is added to an allow-list. | A published negative and a published clearance look similar in a summary line, and diligence fatigue rewards the shorter wording. | **Distinguish three records: an instrument (finding), a refusal to confirm or deny (non-finding), and no search performed (gap).** The Gambling Commission's response to a request dated **16 January 2024** declined to confirm or deny whether it held any information, citing **section 31(3) FOIA 2000** ✅ — which is a documented non-finding and neither clearance nor condemnation ([§5.3](#53-great-britain-a-licence-that-was-not-there)). An absence is recorded as the negative result of a dated search and proves nothing by itself ([§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under)). |
| **9** | **A co-branded family member is conflated with the licensee.** Two brands share design, odds feed, payment pages or an affiliate programme, and the file silently extends one brand's licence, or one brand's enforcement history, to the other. | Shared infrastructure is visually and commercially persuasive, and the sector's aggregators describe brand clusters as if they were corporate groups. | **Assert a relationship only where a named source states it, and at register level where the claim is corporate.** The guide treats "same platform", "same odds feed", "same payment pages", "same affiliate programme", "same design" or "similar name" as **not** evidence of common ownership, and records the family map as **not established** at register level ([§2.3](#23-the-brand-family)). Where two named companies appear together in an authority's instrument, that is the register-grade link to use ([§3.2](#32-the-two-entity-origin--and-the-naming-trap-that-follows)). |
| **10** | **A licensing jurisdiction's register is read as permission in a destination market.** "Verified licensed at the Curaçao register" is treated as a green light for flows from markets the licence does not cover. | The verification step was done properly, so the conclusion is assumed to travel with the correct artefact — a licensing-record fact converted into a market-authorisation fact. | **Separate the *licensing-record* question from the *market-permission* question and answer each at its own authority.** Only the destination market's regulator can determine whether the operator may lawfully serve that market's customers ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)); the mismatch between the licensed perimeter and the customer base is the **customer-jurisdiction mismatch**, and it is the single most important licensing-derived risk in the file ([§9.4](#94-the-customer-jurisdiction-risk)). |

### 13.4 The anti-pattern index as a pre-signing checklist

Ten questions, each answerable from documents rather than from impressions. If any answer is "the brand", "the website says so", or "press reported it", the file has an anti-pattern in it.

1. **Which legal person is the counterparty, and what is its company number?** (Anti-pattern 1.) ✅-grade answer for the dot-com operation: Caecus N.V., 163779.
2. **What is the licence number, its grant date, and the date the register was read?** (2.) ✅-grade: OGL/2024/1262/0493, granted 7 November 2024, read September 2026.
3. **Which markets does that licence cover, and which markets do the revenues come from?** (3, 10.) ❌-grade at present: the customer-jurisdiction map is not public.
4. **Which entity contracts with acquirers and receives settlement, per market?** (4.) ❌-grade at present: not public, obtainable only from the merchant.
5. **Is any sponsorship being used as a licensing argument?** (5.) Remove it if so; date the announcements instead.
6. **Is every regulatory sentence carrying authority, instrument, date, parties and outcome — or an explicit unverified flag?** (6.) Enforce [§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under) rules (i)–(iv).
7. **Is the full domain estate mapped against the certificate series?** (7.) ✅ partial: 1xbet.com and 1x-bet.com.
8. **Are refusals-to-confirm recorded as non-findings rather than as clearance or as findings?** (8.) ✅-grade for Great Britain.
9. **Is every brand-family link sourced, or deleted?** (9.) ❌-grade: not established at register level.
10. **Has the licensing-record fact been kept apart from the market-permission question?** (10.) The distinction is [§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding).

**The one-line failure this section exists to prevent: a file whose counterparty is a brand, whose licence is a claim, whose market list is marketing, and whose comfort is a sponsorship.** The next section puts those four errors in a bank and shows what a review that avoids them actually looks like — and why avoiding them still leaves open the question of whether to do the business at all.

---

## 14. The Cymbal Bank Worked Example

**This scenario is FICTIONAL and ILLUSTRATIVE.** Cymbal Bank is not a real institution; no real bank, payment provider, sponsor, regulator or operator is described as a party, client, counterparty or acquirer here, and nothing below states or implies how any real bank, any real operator or any real market has decided or would decide any question. The facts about the operator used in the walkthrough are the public-record facts sourced elsewhere in this guide, each marked; the bank, its policies, its risk appetite and its decision are invented in order to show the reasoning. **Cymbal Bank is the repository's only bank persona.**

### 14.1 The request

**The ask.** An online gambling operator's payments side approaches Cymbal Bank for **acquiring services** — card acquiring and/or local payout accounts — for a multi-market dot-com sportsbook-and-casino brand. The introduction comes from a payments broker; the deck is branded with a global sportsbook-and-casino name; the deck states the brand is licensed, lists more than seventy market presences, and names two tier-one football club partnerships.

**What Cymbal is being asked to underwrite.** Cymbal is not being asked to lend. It is being asked to (i) take on a **high-risk merchant** whose deposits and withdrawals will run through Cymbal's rails; (ii) accept the **chargeback, dispute and refund** exposure attached to gambling transactions; and (iii) become, in effect, the bank that is visible in the payment chain of an operator serving customers in many jurisdictions, a number of which Cymbal cannot itself assess from the deck.

**The four questions the review must answer, in order.** Who is the counterparty? Is it licensed, and where, and when was that read? Where does the licence permit business, and where are the customers and the flows? What risk does the gap create, and what conditions close it? Everything below is those questions in sequence, plus a decision table and the reason Cymbal restricts the relationship.

### 14.2 Step 1 — identify the legal entity, and the merchant of record

| Step | Action | Finding in this illustrative case | Discipline applied |
| --- | --- | --- | --- |
| 1.1 | Ask the merchant to name the **contracting legal entity**, with its registration number and jurisdiction — in writing, from the merchant. | The merchant's own correspondence names a **Curaçao company** as the operator of the brand's principal domain, with a company number and a licence number. | Anti-pattern 1 ([§13.2](#132-the-six-core-anti-patterns)); the brand on the deck is **not** recorded as the counterparty. |
| 1.2 | Verify the named entity against the **licensing authority's own certificate**, dated. | The Curaçao Gaming Authority's Certificate of Operation, retrieved **September 2026**, states that **1xbet.com** is operated by **Caecus N.V.**, Curaçao company number **163779**, licence **OGL/2024/1262/0493**, granted **7 November 2024**, status **Active** under the National Ordinance on Games of Chance (*Landsverordening op de kansspelen*, P.B. 2024, no. 157) ✅. | Anti-pattern 2; the artefact is the authority's certificate, not the deck ([§5.1](#51-what-is-held-verified-at-the-authoritys-own-record)). |
| 1.3 | Map the **domain estate** against the certificate series. | A parallel certificate in the same series names the same licensee, company number and licence for **1x-bet.com** ✅; other domains in the estate were **not** evidenced to Cymbal. | Anti-pattern 7; the licensed perimeter is the certified domain set, not the brand. |
| 1.4 | Establish the **merchant of record** per market — the entity that contracts with acquirers and receives settlement. | **Not established.** The merchant's answers describe the brand's wallet and platform at a group level and do not reconcile to a single settling entity per market. | Anti-pattern 4; Cymbal records the gap rather than accepting the brand's name on the settlement side. |
| 1.5 | Establish **ownership and control** of the licensee. | **Not established.** No source read in this pass carries a parent company, a shareholding or an ultimate beneficial owner for Caecus N.V. | Recorded as an open item, not as an adverse one ([§2.4](#24-the-corporate-structure-question-stated-honestly)). |

**What Step 1 does and does not deliver.** It converts "1xBet" — a brand that holds nothing and can hold nothing — into one named company with a company number and a verified active licence, plus three explicit gaps (ownership, the merchant of record per market, the rest of the estate). That is a materially better file than the deck produced, and it is still an incomplete one: **a verified licence for the operator of a domain is not a verified permission to serve any market's customers.**

### 14.3 Step 2 — verify the licence at the register, and record what it does and does not establish

The Curaçao route is the one artefact in this file that is register-grade, so Cymbal builds the licensing section around it and states its boundaries in the same breath.

| The certificate establishes ✅ | The certificate does not establish ❌ / ⚠ |
| --- | --- |
| The **operator of the named domain** is a specific legal person — Caecus N.V. — with company number 163779. | **Who owns or controls** that company (❌ not established; [§2.4](#24-the-corporate-structure-question-stated-honestly)). |
| The **licence number, class and grant date**: OGL/2024/1262/0493, a B2C online gaming licence to offer games of chance, granted **7 November 2024** ✅. | The **term, expiry or renewal cycle** — the certificate states a status, not a term ⚠ ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)). |
| The **status as at the retrieval date**: Active, read September 2026 ✅. | **Any other entity** in the brand family, and any other domain beyond those certified ⚠. |
| The **statutory basis** named in the instrument: the National Ordinance on Games of Chance, P.B. 2024, no. 157 ✅. | **Permission to serve customers in any specific market.** The licence is issued by Curaçao, under Curaçao's statute, and authorises activity there ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)). |
| The **existence of a public verification route** — the authority's certificate portal — that Cymbal can re-pull itself ✅. | **Anything about the operator's other markets**: for Ireland the licence is encyclopaedia-sourced and not register-verified ⚠; for Russia the register was not reachable ❌; for most markets, nothing was reached ([§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached)). |

**How Cymbal records it.** One register-grade licensing fact, one dated retrieval, an explicit status (Active at September 2026), and an explicit list of what the licence does not reach. The Great Britain position is recorded in the same section as what it is — a **documented negative**: the Gambling Commission declined to confirm or deny whether it held information about a licence, suspension or ban concerning the brand, citing **section 31(3) FOIA 2000**, in a response to a request dated **16 January 2024** ✅ ([§5.3](#53-great-britain-a-licence-that-was-not-there)). Cymbal does **not** write "no regulatory issues"; it writes "no action published, none confirmed, none denied, as at the date read".

### 14.4 Step 3 — map where the licence permits business against where the customers and the flows are

The licence is Curacao-issued. The customers are not, mostly. This step is the heart of the review, and it is done as a market-by-market table with an explicit sourcing column, because the mismatch is only visible when the two sets are written side by side.

| Illustrative market | Licence/permission position recorded in this pass | Customers and flows assumed present (illustrative) | Sourcing of the position | Status |
| --- | --- | --- | --- | --- |
| **Curaçao** | Licensed — Caecus N.V., OGL/2024/1262/0493, Active | Yes (home licensing jurisdiction) | Curaçao authority certificate, September 2026 | ✅ verified at source |
| **Netherlands** | No licence; enforcement concluded — fines imposed in 2019 for offering games of chance to Dutch players without a licence | Yes (Dutch-language estate) | Ksa decisions of 4 January 2019 and 7 May 2019 | ✅ verified at source |
| **Great Britain** | No action published; regulator refused to confirm or deny | Yes (English-language estate) | Gambling Commission FOI response, request 16 January 2024 | ✅ documented negative |
| **Liberia** | Local licensee's licence revoked **22 June 2026**; the complaint against the brand dismissed because a brand is not a legal entity | Yes (branded billboards and radio advertising documented by the authority) | NLA Ruling and Judgment, 22 June 2026 | ✅ verified at source |
| **Ukraine** | Sanctions instrument of **10 March 2023** designates entities connected to the betting and lottery business, including **1XCorp N.V.** | Assumed absent-to-minimal | Decree reported by the national news agency; annex not read at source | ⚠✅ instrument identified, listing reported |
| **Spain, Mexico, Serbia, Peru, Ghana, Uganda, Kenya, Ireland** | **Reported licensed** against variously named local companies/domains — none verified by this pass at the local register | Significant, multi-language | Aggregators, encyclopaedia, trade press | ⚠ leads only |
| **Nigeria** | **Reported licensed but recorded expired** | Significant | Aggregator | ⚠ "an expired licence is not a licence" — re-check |
| **France, Italy, Greece, Cyprus, Czechia, Poland, Lithuania, Estonia** | Reportedly appear on **published unauthorised-operator or blocking lists** | Significant, EU | Aggregator that states it reads each regulator's own list, late September 2026 | ⚠ reported, not verified individually |
| **Russia** | **Not established** — the Russian-facing brand is a separately licensed Russian bookmaker; the FNS register was not reachable in this pass | Russian-speaking customers directed to a separate licensed brand | Russian aggregators, conflicting; register unreachable | ❌ not established (tool limitation) |
| **The rest of the world** | **Not established** beyond the operator's own claim of local licences in "more than 35 markets" | Material, cross-border | Company statement, January 2026 | ⚠ company claim; ❌ not verified |

**The mismatch read off the table.** The only permission Cymbal can verify as at a dated reading is a **Curaçao** one. The markets in which the brand's language estates, reported licensing footprint and enforcement history show material customer presence are **mostly not Curaçao** — and in three of them the public record is an authority's own concluded matter (Netherlands), a concluded local proceeding (Liberia) and a reported blocking/unauthorised listing (eight EU markets). Cymbal's own words, in its memo: *the customers whose deposits Cymbal would be settling are, in the aggregate, customers the verified licence does not reach.*

### 14.5 Step 4 — rate the AML and jurisdictional risk

Cymbal rates the relationship on the two axes that the licence map actually generates, and it does so against its own risk framework, cross-referencing rather than re-deriving the financial-crime machinery ([Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)), [AML Certifications & Exam Content](aml_certifications_exam_content_guide.md))).

| Risk axis | What drives it in this case | Illustrative rating (Cymbal's own, fictional) |
| --- | --- | --- |
| **Jurisdictional risk** | Verified permission covers the licensing jurisdiction only; the customer base and the language estates sit largely outside it; reported unauthorised-operator listings in several destination markets create **enforced-exit risk** on revenue the flows carry ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)). | **High.** Not because of the activity, but because the revenue base is exposed to a regulator's decision in markets where the verified licence does not apply. |
| **Licence-verification risk** | One register-grade fact (Curaçao), dated; for every other market, a lead, a company claim or an absence; possible local white-label arrangements in which the brand holds nothing locally ([§5.6](#56-the-licence-quality-spectrum-by-class-and-jurisdiction)). | **High.** The file's licensing evidence is thin relative to the market footprint it would settle. |
| **Merchant/chargeback risk** | Gambling is a high-risk merchant category; disputes, bonus-abuse claims and refund requests generate chargebacks and reversals that Cymbal bears as acquirer (§4.3 of this guide; machinery cross-referenced, not restated). | **High.** Standard for the category and managed by conditions, not by declining the category as such. |
| **Customer-jurisdiction mismatch** | The verified perimeter and the customer/flow map are materially different sets ([§9.4](#94-the-customer-jurisdiction-risk)). | **High — the controlling risk.** This is the risk Cymbal's decision turns on. |
| **Sponsorship/legitimacy signal** | Two verified tier-one club partnerships, dated 2019 and 2024 ✅ ([§7.2](#72-verified-sponsorships--the-sponsors-own-announcement)). | **Not a risk mitigant.** Recorded as marketing spend and as a dated reputational variable, never as evidence of regulatory standing ([§7.4](#74-what-sponsorship-buys--the-guides-own-analysis-labelled)). |
| **Ownership opacity** | Ownership and control of the licensee are not established at source ([§2.4](#24-the-corporate-structure-question-stated-honestly)). | **Elevated.** An open item to be resolved by documentary evidence, not an adverse finding. |

### 14.6 Step 5 — the conditions Cymbal sets

Cymbal's committee does not treat "no" and "yes" as the only options. It sets a conditional, restricted structure and prices the residual risk; the conditions below are the shape such a structure takes, each tied to a specific gap identified in Steps 1–4.

| # | Condition | The gap it closes |
| --- | --- | --- |
| C1 | **Contract only with the identified licensee** (or a named group entity Cymbal has itself verified), never with a brand name, an affiliate, a franchisee or a broker entity. | Anti-patterns 1 and 4; the merchant-of-record gap (Step 1.4). |
| C2 | **Licence evidence on file before activation, with a re-verification cadence**: the authority certificate re-pulled at onboarding and at each periodic review, with the retrieval date recorded each time. | Anti-pattern 2; the register is a point in time ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)). |
| C3 | **A jurisdiction allow-list, driven by the customer's own market**: flows accepted only from markets on Cymbal's list; a market is added only against that market's own authority's instrument, never against the licensing jurisdiction's certificate. | Anti-patterns 3 and 10; the customer-jurisdiction mismatch (Step 3, Step 4). |
| C4 | **Rail restrictions**: settlement only to accounts in the contracting entity's name and in named jurisdictions; no settlement to accounts held in the brand's trading name; restrictions on high-risk or opaque rails and on any rail whose counterparty Cymbal cannot identify. | The Liberian record's payment-flow pattern — funds attributed to accounts in the brand's own name rather than the licensee's ✅ ([§6.1.2](#61-concluded-matters)); [§10](#10-the-aml-and-payment-rails-angle). |
| C5 | **Transaction monitoring, with gambling-specific typologies** and defined thresholds for velocity, device/account multiplicity, bonus-abuse patterns and dispute ratios; alerting routed to the fraud machinery the repository already documents. | Merchant and AML risk (Step 4); [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md)), [AML Certifications & Exam Content](aml_certifications_exam_content_guide.md)). |
| C6 | **Exposure limits**: a rolling settlement exposure cap, a chargeback-ratio trigger, and a reserve or rolling hold where the ratio breaches a set level; limits sized to the verified revenue base, not to the claimed footprint. | Chargeback and settlement risk; the claim-versus-verification gap ([§4.4](#44-performance-claims-the-company-makes-about-itself)). |
| C7 | **Termination triggers, listed and dated in the agreement**: an adverse licensing event in the licensing jurisdiction; the addition of a market to a destination regulator's unauthorised-operator or blocking list; a refusal by the licensee to provide the merchant-of-record documentation; any change of control of the contracting entity without notice. | The whole map of Steps 2–4; converts a compliance observation into a contractual exit. |
| C8 | **Marketing-materials diligence before activation**, because in at least one jurisdiction the marketing presentation itself was the regulatory breach. | [§8.4](#84-the-reputational-and-regulatory-consequence-of-the-acquisition-model) point 2; the NLA's finding on branded billboards and radio advertising with no visible reference to the licensed entity ✅. |

### 14.7 Step 6 — ongoing monitoring

| What Cymbal watches | Frequency | The change that triggers action |
| --- | --- | --- |
| The **licensing authority's certificate** for the contracted operator and its domain set | At onboarding, each periodic review, and on any adverse signal | Status change, a new grant date, a change of named operator, or the domain dropping out of the certificate series |
| The **destination markets' unauthorised-operator and blocking lists** | Monthly sweep against Cymbal's allow-list markets | A listed market's flows are suspended pending re-verification |
| **Regulatory publications and FOI disclosure logs** of the relevant authorities | Quarterly | A published instrument naming the counterparty or a related entity |
| **Sponsorship changes** — signings, renewals and terminations — at the rightsholders' own announcements | Quarterly | A termination or non-renewal, read as a dated third-party re-pricing signal ([§7.4](#74-what-sponsorship-buys--the-guides-own-analysis-labelled)) |
| **Settlement-account names and jurisdictions** against the contracted entity | Continuous, by payment monitoring | Any settlement to an account in a brand, affiliate or third-party name |
| **Transaction-monitoring typologies and dispute ratios** | Continuous | Threshold breaches under C5–C6 |
| **Ownership and control of the contracting entity** | Annually, by documentary request | A change of control, or a refusal to provide the documentation |

### 14.8 The decision table, and why Cymbal restricts rather than declines outright

**Cymbal Bank's illustrative decision.** Cymbal **restricts** the relationship rather than declining it outright — and, on the illustrative fact pattern, one plausible route is that it restricts so sharply that the operator's payments side walks away, which is a decline in substance.

| Draft decision | What it would mean | Why Cymbal does or does not take it |
| --- | --- | --- |
| **Decline** | No account, no acquiring, relationship closed at diligence. | Defensible, but the risk is not the *category* or the *people*: a genuinely licensed operator with a verified active licence is a legitimate counterparty for a bank willing to run a high-risk merchant book. Declining outright because a brand is controversial would be a decision about reputation, not about the licence and the flows. |
| **Approve on standard terms** | Standard acquiring, standard monitoring. | **Not available.** The verified licence reaches one jurisdiction while the customer base and language estates sit across many, three markets carry an authority's own concluded adverse record, and the merchant of record is unidentified. Standard terms would mean Cymbal settling flows in markets it has not shown the counterparty may serve. |
| **Approve restricted** | Contract with the identified licensee; flows **only** from markets on Cymbal's verified allow-list (initially the licensing jurisdiction and any market where the destination authority's own instrument is produced); settlement to named accounts only; exposure caps; C2–C8 conditions. | **This is Cymbal's position.** It converts the mismatch into a perimeter: Cymbal accepts the risk it can see (the licensed jurisdiction) and refuses the risk it cannot see (the unlicensed remainder). The decision turns on **the licence and the customer-jurisdiction mismatch** — how much of the flow base sits outside the verified perimeter — and not on the character of the business or of any person in it. |
| **Approve and rely on the sponsor relationships** | Accept the deck's legitimacy argument. | **Rejected on the record.** Sponsorship is dated marketing spend, verified at the sponsor's own announcements ✅, and is explicitly not evidence of regulatory standing ([§7.4](#74-what-sponsorship-buys--the-guides-own-analysis-labelled) point 5). It is not a decision input at all. |
| **Approve and rely on the register fact alone** | Treat the Curaçao Active certificate as a global green light. | **Rejected.** That is anti-pattern 10, and it is the specific error this worked example is built to expose: a correct verification step converted into an incorrect authorisation assumption. |

**The reasoning in one paragraph.** Cymbal's restriction is a statement about a **perimeter**, not about a business: the one thing the bank could verify at an authority's own artefact was a licence issued by one jurisdiction to one company for one domain set, while the deposits it was asked to settle would predominantly come from customers in markets that licence does not reach, from an operator whose merchant of record per market it could not identify, and with part of that flow base subject to a destination regulator's published adverse listing. Cymbal therefore restricted the relationship to the licensed perimeter it could verify and set conditions to re-open markets only against each market's own authority's instrument. **No part of this decision rests on a view about gambling, or about anyone's character.**

### 14.9 What this worked example is not

- It is **not** a description of any real bank's policy, any real acquirer's position, or any real market's decision, and no real institution is named as a party to it.
- It is **not** a risk rating of the operator by this guide. The problem the guide diagnoses is a **documentary and jurisdictional** one — the difference between a licence and a claim, and between a licensing jurisdiction and a customer market — and that problem is generic to multi-market online operators, not unique to this brand.
- It is **not** a finding that any flow, any market or any arrangement is unlawful. Where the guide records an authority's conclusion, that authority is named and the instrument is dated ([§6](#6-the-regulatory-record)); where the guide records an allegation or a report, it is labelled as such ([§6.3](#63-reported-but-unverified-matters-kept-out-of-the-record)).
- It is **not** a substitute for the bank's own file. The diligence artefacts named in Steps 1–2 are the authority certificate, the merchant's written identification of the contracting entity, and the destination-market instruments; those four documents are the review, and everything else in this section is the shape they fit into.

---

## 15. The Claims Audit

*This is the consolidated status table promised in [§1](#1-the-overview-the-identity-gate-the-decoder-and-the-boundary)'s verification-convention paragraph. It lists the claims this guide met, each with its **source**, the source's **date**, and the **quality** of that source — an authority-issued instrument, the company's own claim, named press, or an encyclopaedia round-up. The audit is deliberately split: **regulatory and licensing items sit in Block A, separate from everything else in Block B**, and the dated searches that produced **nothing** sit in Block C. Block A is the block a regulator's artefact supports; Block B is the block a commercial interest supports; Block C is the block that proves nothing and is recorded anyway.*

### 15.1 How the audit is built

| Element | The guide's rule |
| --- | --- |
| **A claim's place in the blocks is determined by its subject, not by its strength.** A licensing claim stays in Block A even when unverified; a company claim stays in Block B even when the company is confident. | Mixing the blocks is how a marketing statement acquires the authority of a register entry. |
| **Quality is a property of the source, and the strongest available source is the authority's own artefact** — a certificate, a decision, a judgment, a published FOI response. | Where no such artefact was reached, the item is flagged, not softened into a finding ([§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under) rule (i)). |
| **Every row carries a date** — the instrument's date or the retrieval date. An undated row is not admissible in this guide's method. | Registers are points in time ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)). |
| **An absence is a row.** A market searched at a register with no result is recorded with the register, the search and the date. | [§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under); [§16.1](#161-16a-what-could-not-be-verified). |

### 15.2 Block A — regulatory and licensing claims

**Block A holds only claims about licences, regulators, enforcement, sanctions and court or tribunal proceedings.** Each row names the authority and the instrument where one exists, or carries an explicit unverified flag where none was reached.

| # | Claim | Authority and instrument (or the absence of one) | Date | Quality of source | Status |
| --- | --- | --- | --- | --- | --- |
| A1 | 1xbet.com is operated by **Caecus N.V.**, Curaçao company number **163779** | Curaçao Gaming Authority, **Certificate of Operation** (cert.gcb.cw) | Certificate retrieved **September 2026** | **Authority-issued certificate** | ✅ verified at source |
| A2 | The licence is **OGL/2024/1262/0493**, granted **7 November 2024**, status **Active** | Same authority certificate | Retrieved September 2026; grant 07/11/2024 | **Authority-issued certificate** | ✅ verified at source |
| A3 | The licence is issued under the **National Ordinance on Games of Chance** (*Landsverordening op de kansspelen*), **P.B. 2024, no. 157** | Same certificate (statute named in the instrument itself) | Retrieved September 2026 | **Authority-issued certificate** | ✅ verified at source |
| A4 | The same licensee, company number and licence also cover **1x-bet.com** | Same authority, parallel certificate in the same series | Retrieved September 2026 | **Authority-issued certificate** | ✅ verified at source |
| A5 | The **ownership and control** of Caecus N.V. | **No authority artefact reached.** Journalistic reporting of licensing-assessment files describes ownership questions; the individual named in that reporting is **not named in this guide** | Reporting dated 2023 and 2026 | Press/investigative journalism | ❌ **not established**; reporting treated as ⚠ allegation only |
| A6 | A British licence was **revoked, suspended or banned** | Gambling Commission, published **FOI response**: the Commission **declined to confirm or deny** whether it held any information, citing **section 31(3) FOIA 2000**; no regulatory action or public statement naming the brand was located | Request dated **16 January 2024**; registers retrieved September 2026 | **Authority-issued published response** | ✅ documented **negative** — claim **not established** |
| A7 | The 2019 UK-facing operation ran as a **white label under another company's Gambling Commission licence** | **No authority artefact.** Trade press reported the explanation; this guide reports it as reported and does not adopt it | Trade press, 2019 | Named trade press | ⚠ reported; not verified |
| A8 | The Gambling Commission warned Liverpool, Chelsea and Tottenham that sponsoring an **unlicensed operator** may expose the sponsor to prosecution under **section 330** of the Act | The Commission's **own quoted statement**, carried by trade press | 2019 | Authority statement carried by named trade press | ✅⚠ statement is the authority's; carried by press, not read on the authority's own site |
| A9 | Websites operated by **1X Corp N.V.** and **Exinvest Limited** offered games of chance to Dutch players contrary to the **Wet op de kansspelen** | Kansspelautoriteit: investigation report **18 September 2018**; sanction decision **4 January 2019**, kenmerk **12924/01.046.245** | Decision 4 January 2019 | **Authority-issued decision documents** on the authority's own site | ✅ verified at source (concluded matter, [§6.1.1](#61-concluded-matters)) |
| A10 | Fine of **€400,000** joint and several, later varied on objection to **separate fines of €200,000** on each of the two companies, plus **€1,024** in costs | Kansspelautoriteit **Besluit op bezwaar** of **7 May 2019**, kenmerken 12924/01.056.071 and 13175/01.056.072 | 7 May 2019 | **Authority-issued decision** | ✅ verified at source |
| A11 | The Dutch fines went **unpaid** and recovery proceedings began; the authority is reported to have called non-payment "a strong indication that the provider is unreliable" | **No authority document read** for the recovery or the remark in this pass | Press reporting, later than 2019 | Press, attributed to the authority | ⚠ reported; not verified at an authority document |
| A12 | Entities connected to the betting and lottery business, including **1XCorp N.V.** (Curaçao, Willemstad, registration 130189), were **sanctioned** by Ukraine | Presidential **Decree No. 145/2023** of 10 March 2023 enacting the NSDC decision of the same date; the decree's **annex was not read at source** | 10 March 2023 | Instrument identified via the national news agency; listing via a sanctions database | ⚠✅ instrument verified; specific listing reported only |
| A13 | The relationship between **1XCorp N.V.** and **Caecus N.V.** | **No authority artefact.** Press has described a parent/related relationship; no register finding supports it | Reporting 2023–2026 | Press | ❌ **not established** |
| A14 | A Liberian administrative tribunal **revoked a local licensee's licence** and **dismissed the complaint against "1 X BET" because a brand is not a legal entity** | National Lottery Authority of Liberia, **Ruling and Judgment** in *NLA v. LIPAY, Inc. and the Management of 1 X Bet*, published on the authority's own site | Judgment **22 June 2026** | **Authority-issued administrative judgment** | ✅ verified at source (concluded matter, [§6.1.2](#61-concluded-matters)) |
| A15 | The revoked Liberian licence was later **restored** under a settlement with a one-year 2026/2027 licence | **No NLA instrument read** | Press reporting, 2026 | Named Liberian press | ⚠ reported; not verified |
| A16 | The brand holds an **Irish remote bookmaker's licence** (recorded as No. 1019276) through **Terminus Platform Ireland Limited** | **Not verified at the Irish register in this pass** | Encyclopaedia entry retrieved September 2026 | Encyclopaedia round-up | ⚠ **not register-verified** |
| A17 | The **Russian-facing brand** holds a Russian FNS bookmaker licence; its relationship to 1xBet is disputed and licence dates are inconsistent | **FNS register not reachable in this pass** (404 on the register path attempted) | Aggregators, September 2026 | Russian-language aggregators | ❌ **not established**; tool limitation recorded |
| A18 | The brand **appears on published unauthorised-operator or blocking lists** in France, Italy, Greece, Cyprus, Czechia, Poland, Lithuania and Estonia | **Not verified individually at each authority in this pass**; the aggregator states it read each regulator's own list in late September 2026 | Late September 2026 (per the aggregator) | Commercial aggregator stating a register read | ⚠ reported; **not verified at each authority** |
| A19 | The brand is **licensed in Spain, Mexico, Serbia, Peru, Ghana, Uganda and Kenya**, and in Nigeria on a licence the aggregator records as **expired** | **Not verified at any of those registers** (the Spanish authority's register returned HTTP 503 in this pass) | Aggregator reads, 2026 | Commercial aggregator; one commercial review site (Brazil) | ⚠ leads only; the expired Nigerian entry is **not** a licence |
| A20 | A reported **Moroccan investigation**, reported **Russian criminal case and arrest warrants**, and reported **Curaçao insolvency proceedings** touching a related entity | **No instrument read for any of them** | Reports 2020–2024 | Press and encyclopaedia | ⚠ reported; **status not established** ([§6.2](#62-ongoing-matters-and-matters-whose-status-is-not-established)) |

**Block A's summary line.** Of twenty regulatory and licensing items, **seven are supported by an authority's own artefact** (A1–A4 as one certificate series, A6, A8, A9–A10, A12 as to instrument, A14) ✅; **eight are reported and unverified** ⚠; and **three are rejected or not established** ❌ (A5, A13, A17). **No licensing item in this guide rests on a commercial aggregator, and no allegation appears in Block A as a finding.**

### 15.3 Block B — all other claims: company, sponsorship, business and marketing

**Block B holds company claims, sponsorship announcements, business and performance claims, and marketing statements.** None of these items is a licensing fact, and a row in this block must never be cited as one.

| # | Claim | Source | Date | Quality of source | Status |
| --- | --- | --- | --- | --- | --- |
| B1 | 1XBET becomes an **FC Barcelona Global Partner** for five seasons, effective 1 July 2019, running **through 30 June 2024** | **fcbarcelona.com** — the club's own announcement | Published **3 July 2019** | Sponsor's own announcement (rightsholder's own primary statement) | ✅ verified at sponsor's own announcement |
| B2 | The partnership **renewed**: Global Partner and Official Betting Partner for five more seasons, **through June 2029** | **fcbarcelona.com** — the club's own announcement | Published **1 July 2024** | Sponsor's own announcement | ✅ verified at sponsor's own announcement |
| B3 | The company describes itself as employing "**over 5,000 professionals**", accepting "**more than 250 payment solutions**", with "around the clock customer support in **30 languages**" | The company's own "About 1XBET" text, carried inside FC Barcelona's announcement | July 2019 (company text of that vintage) | **Company's own claim**, reproduced by a sponsor | ✅ as a company claim; ❌ not verified |
| B4 | The company claims "**17 years of experience**", a website and app "**in 70 languages**", and partners including Paris Saint-Germain, LOSC Lille, Italian Serie A and the Confederation of African Football | Company's own "About 1XBET" text, carried in FC Barcelona's 2024 announcement | July 2024 | **Company's own claim**, reproduced by a sponsor | ✅ as a company claim; the named partners ⚠ unconfirmed by any rightsholder announcement reached |
| B5 | The company states it holds **local licences in more than 35 markets** across Latin America, Africa and Western Europe, naming Serbia and Guatemala as recent regulated-market entries | Company statement, reported | January 2026 | **Company's own claim**, reported | ⚠ company claim; ❌ **no register-by-register support; not verified** |
| B6 | **Liverpool FC and Chelsea FC suspended** their partnerships with the brand, and **Tottenham Hotspur ended** its agreement | Bellingcat (21 October 2024) reporting the severances; gambling trade press, 2019 | 2019 (severances); report 21 October 2024 | Named press and investigative journalism | ⚠ **the severance is reported**; the reason is attributed to the regulator's warning (A8) |
| B7 | Reported sponsorships and ambassadorships: **Paris Saint-Germain** (reported still a sponsor as at October 2024), **Billie Jean King Cup** (announced April 2025), **ATP Challenger Tour** (from October 2025), esports organisations, individual ambassadors, and a **Philippine basketball league title sponsorship later reported dropped mid-season** | Encyclopaedia entries citing press releases and trade press; Bellingcat 21 October 2024 | 2019–2026 | Encyclopaedia round-ups and press | ⚠ reported; **rightsholders' own announcements not retrieved in this pass** |
| B8 | Turnover "**exceeded $2 billion in 2020**" | Forbes Russia, 9 December 2020, via an encyclopaedia entry | 2020 (figure), cited 2026 | Named business press | ⚠ **not verified and not reconciled with any other figure** |
| B9 | Monthly average visits "**exceeded 5 million**" (2024) | Bellingcat (21 October 2024), citing SimilarWeb | 2024 | Third-party traffic-estimation methodology, not audited data | ⚠ attribution: estimation, not measurement |
| B10 | "**Tens of billions in terms of revenues**" | A third-party commentator, quoted by Bellingcat | 21 October 2024 | A person's quoted estimate | ⚠ an estimate by a person; **not a figure** |
| B11 | An "**active Indian money laundering probe**" | A commercial dossier-style site, no authority named, no instrument produced | Undated in the source | Commercial dossier site | ❌ **rejected for lack of support**; not relied on, not repeated as a matter |
| B12 | A reported observation that the profile photograph used for a named press spokesperson was that of a journalist at another broadcaster | Bellingcat (21 October 2024) | 21 October 2024 | Investigative journalism | ⚠ reported; a media-identity observation, **not a regulatory matter** |
| B13 | The group operates a **franchise business model** | Encyclopaedia entry | Retrieved September 2026 | Encyclopaedia round-up | ⚠ reported; structurally important if true, so recorded as reported only |
| B14 | The **brand-family relationships** — which other brands share origin, ownership, staff or infrastructure with this operation | **No source read in this pass establishes the relationships at register level** | — | — | ❌ **rejected as unestablished**; only sources' own statements are used ([§2.3](#23-the-brand-family)) |

**Block B's summary line.** Two sponsorship items are verified at the sponsor's own announcements ✅ (B1, B2); **every other item in this block is a company claim, a press report or an estimate** ⚠, and one is rejected ❌ (B11). **No performance, revenue, customer, employee or market-share figure for any entity in this family is established at source anywhere in this pass** — the figures in B3–B5, B8–B10 are all company claims, third-party estimates or commercial press. That absence is the finding ([§4.4](#44-performance-claims-the-company-makes-about-itself)).

### 15.4 Block C — licensing absences recorded as negative findings of dated searches

Each row records **what was searched, where, and when, and what was not found**. Per [§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under), an absence proves nothing by itself; it is recorded because a documented negative is the only honest way to hold a gap, and because in this subject matter the gaps are the material.

| # | What was searched | Where | Date of search | What was not found |
| --- | --- | --- | --- | --- |
| C1 | A **British regulatory action, licence entry or public statement** naming the brand | Gambling Commission public registers and its FOI disclosure log; the regulatory-actions register reported **136 records** at retrieval | September 2026 | No action, entry or statement naming the brand; and the Commission expressly declined to confirm or deny whether any information exists (A6) ✅ |
| C2 | An **authority instrument supporting the claim of a revoked British licence** | Searches of the Commission's published material and trade press | September 2026 | No instrument, decision or public statement; the structural explanation reached is the white-label account (A7), which is reported, not adopted |
| C3 | The **Irish remote bookmaker's licence** at register level | The Irish register was **not reached** in this pass; the licence is known only at encyclopaedia level | September 2026 | No register reading — the item is a gap, not a verified licence (A16) |
| C4 | The **Russian FNS bookmaker register** and the Russian-facing brand's licence | The register path attempted returned **404** | September 2026 | No register reading; licence dates reported by aggregators are mutually inconsistent (A17) |
| C5 | The **Spanish** authority's licence register, to test a reported licence matched to the 1xbet.es domain | DGOJ register page, attempted | September 2026 | **No reading — the register returned HTTP 503**; the item remains an aggregator lead (A19) |
| C6 | The **Curaçao register for entities other than the certified licensee**, and for domains beyond those certified | Curaçao Gaming Authority certificate portal, consulted **per brand** | September 2026 | Nothing enumerated: the portal resolves brands rather than listing entities, so **no negative finding is asserted** about other entities or other domains ([§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024)) |
| C7 | **Financial statements, audited accounts or any statutory filing** for Caecus N.V. or any entity in the family | No public filing source reached | September 2026 | No statements, accounts, revenue, customer or employee figures — **the whole of Block B's performance section is unproven** |
| C8 | **Ownership, parentage or ultimate beneficial ownership** of the licensee | No register or filing reached | September 2026 | Nothing beyond a press description; the corporate structure remains unestablished ([§2.4](#24-the-corporate-structure-question-stated-honestly)) |

**The point of Block C.** Three of these eight rows are **register absences** (C1, C2, and the negative half of C6), three are **registers that could not be read at all** (C3, C4, C5) and two are **sources that do not exist in public** (C7, C8). A file that reads only Blocks A and B will treat the operator's licensing position as richer than it is; Block C is what tells the reviewer how much of that apparent richness is a gap.

### 15.5 What this audit does not do

- It does **not** rate the operator. It records the source quality of claims, not the size of the counterparty's risk; the bank's assessment is [§9](#9-the-banks-view--the-merchant-and-payment-counterparty-assessment) and [§10](#10-the-aml-and-payment-rails-angle), and the illustrative treatment is [§14](#14-the-cymbal-bank-worked-example).
- It does **not** convert any flagged item into a finding, and it does not read a documented negative as a clearance. A6 in particular is a refusal, not an exoneration ([§5.3](#53-great-britain-a-licence-that-was-not-there)).
- It does **not** name any private individual in connection with any allegation; the single exception the guide's rules allow — a regulator, a court or a legislature having publicly named the person — was not established at source in this pass ([§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under) rule (v)).
- It does **not** include any item this pass could not source or attribute; items met only in a commercial dossier without an authority were rejected outright (B11).

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

*This closing section has four parts: (16.a) the explicit checklist of what could not be established — the item §2.6 promised here; (16.b) the glossary of the vocabulary the guide uses, defined precisely; (16.c) the cross-references and the boundary against the repository's neighbouring guides; and (16.d) the closing summary.*

### 16.1 (16.a) What Could Not Be Verified

**This subsection is the deliverable [§2.6](#26-what-could-not-be-established-about-this-identity--bluntly) promised.** It lists, in one place, everything this pass could not establish at source. **It is a feature of the guide and not a failure of it.** [§1.5](#15-the-legal-risk-discipline-this-guide-is-written-under) sets the rule: *"Where the correct output is an absence, the absence is the finding. A register that does not list an entity, a regulator that declines to confirm an action, a licence that cannot be found for a market — these are recorded as the negative results of dated searches, with the register, the date and the search performed. An absence proves nothing by itself; it is recorded because a documented negative is the only honest way to hold a gap, and because in this guide's subject matter the gaps are the material."* Everything below is written to that rule. Nothing below is an adverse finding about the operator; a gap is equally consistent with "not true" and with "true but not public".

**The eight items §2.6 listed, delivered:**

| # | What could not be established | Why it matters to a bank | The dated search recorded |
| --- | --- | --- | --- |
| **1** | **Ownership and control of Caecus N.V.** — who owns the licensee, and who controls it. Journalistic reporting of licensing-assessment files describes ownership questions; under rule (v) **no individual named only in that reporting is named here**, and the reporting is treated as an allegation, not a finding. | Ownership determines who the real counterparty is, who benefits from the flows, and how a change of control would move the relationship. | No register or filing source reached, September 2026 (Block C, C8). |
| **2** | **The parent and intermediate holding structure** — no parent company, no shareholding, no ultimate beneficial owner for any entity in the family. | A bank cannot assess group support, group exposure or group-wide sanction exposure without the chain. | No filing reached, September 2026 ([§2.4](#24-the-corporate-structure-question-stated-honestly)). |
| **3** | **The full list of entities in the brand family and their relationships** — not established at register level; where sources conflict, the conflict is recorded rather than resolved. | Family membership is what a bank is implicitly asked to accept when it accepts a brand; unsourced membership is not evidence of common ownership ([§2.3](#23-the-brand-family)). | No register-level source reached; aggregator descriptions rejected, September 2026 (Block B, B14). |
| **4** | **Whether the brand holds a licence in the markets it serves**, other than where that market's own register was reached. For most markets, not established. | The customer-jurisdiction mismatch is the controlling risk, and it cannot be sized without this ([§5.4](#54-the-jurisdictional-scope-point-the-transferable-finding)). | Per-market positions in [§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached); Spanish register unreadable (503), September 2026 (Block C, C5). |
| **5** | **The merchant of record** for any given market — the entity contracting with acquirers and receiving settlement. | It is the entity a bank actually onboards and settles with; the brand is neither ([§9.1](#91-identifying-the-counterparty-which-entity-is-the-bank-onboarding)). | Not public in any source read; obtainable only from the merchant and its acquirer (Block C, C7 note). |
| **6** | **Financial statements, revenue, customer numbers, employee numbers** for any entity in the family. No audited accounts, no revenue figure attributable to a named entity, no customer or employee count. | Credit and exposure sizing require them; the guide's only figures are the company's own claims and third-party estimates ([§4.4](#44-performance-claims-the-company-makes-about-itself)). | No public filing source reached, September 2026 (Block C, C7). |
| **7** | **The Irish licence at register level** — the licence recorded as No. 1019276 against a named Irish company, trading as "1XBET", is encyclopaedia-sourced and **not register-verified**. | A licence asserted in a European market is materially different from a licence read at that market's register. | Irish register not reached in this pass, September 2026 (Block A, A16; Block C, C3). |
| **8** | **The Russian licence and the Russian entity** — the FNS register was not reachable in this pass, and Russian-language sources give inconsistent licence dates for a separately licensed Russian-facing brand whose relationship to the dot-com operation is itself disputed. | Russia is a large language market with its own licensing regime; the absence of a register read leaves the position unknown rather than negative. | FNS register path returned 404, September 2026 (Block A, A17; Block C, C4). |

**Beyond the eight, these also could not be established, and are recorded with the same method:**

| What could not be established | The record of the search |
| --- | --- |
| **The licensing position in most markets** — the guide's map is one verified licence (Curaçao), one verified enforcement (Netherlands), one documented negative (Great Britain), one concluded local proceeding (Liberia) and one identified sanctions instrument (Ukraine); everything else is a lead or an absence. | [§5.5](#55-the-market-by-market-position-as-far-as-this-pass-reached); aggregator leads not verified individually, September 2026 (Block A, A18–A19). |
| **The status of most alleged matters** — the Moroccan investigation, the reported Russian criminal case and arrest warrants, and the reported insolvency proceedings touching a related entity were met as reports, not instruments; each is recorded as "status not established". | [§6.2](#62-ongoing-matters-and-matters-whose-status-is-not-established); no instrument read, September 2026 (Block A, A20). |
| **Whether any sponsor relationship was terminated and why, where no sponsor announcement states it** — the verified terminations are the 2019 club severances, and the reason attributed to them is the regulator's section-330 warning; where a relationship is reported ended (a Philippine league title sponsorship reported dropped mid-season) with no rightsholder statement read, the ending is itself reported only. | [§7.3](#73-sponsorships-reported-flagged-or-ended); rightsholders' announcements not retrieved, September 2026 (Block B, B7). |
| **The licence's term, expiry and renewal cycle** — the Curaçao certificate states a status, not a term, so the guide knows the licence's position at one retrieval date and its grant date, and nothing about its duration. | [§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024); certificate read September 2026. |
| **Whether the pre-2024 Curaçao licence under which the same brand operated was held by the same company** — the Curaçao register was consulted per brand, so no negative is asserted about other entities or other periods. | [§5.2](#52-the-curaçao-regime-and-what-changed-at-the-end-of-2024); Block C, C6. |
| **The identity and role of the marketing, affiliate and franchise entities in each market** — not researched to register level in this pass; establishing them is a bank's own diligence, not a public register exercise. | [§2.2](#22-the-entity-and-brand-map-with-the-source-for-each-relationship) row 7; [§8](#8-the-affiliate-and-marketing-model). |

**How to read this subsection.** Not one item above is an accusation, and not one is a clean bill of health. Each is a **documented gap**: a question asked of a named source on a named date that the source did not answer. The guide's asymmetry is deliberate — it records the gap rather than filling it with the most available narrative, because the most available narrative about a brand like this is precisely the thing this guide was written to refuse. **Where the correct output is an absence, the absence is the finding.**

### 16.2 (16.b) The Glossary

The guide's decoder terms (restated with the precision it uses) plus the licence-class, merchant-of-record, affiliate, rail and market-colour vocabulary that the later sections depend on.

| Term | Definition as used in this guide |
| --- | --- |
| **Brand** | A marketed trading name, and **not a legal person**. It holds no licence, can hold no registry entry, and (as a Liberian tribunal held on 22 June 2026) cannot be sued as a party. "1xBet" is a brand. |
| **Licence** | An instrument granted by **one authority in one jurisdiction**, for **one class of activity**, to **one legal person**, authorising activity **there**. It does not travel. |
| **Licensing jurisdiction** | The jurisdiction whose authority issued the licence — here the **Curaçao Gaming Authority** for the dot-com estate's operator. |
| **Destination market** | The jurisdiction in which a customer resides, and whose law governs whether the operator may serve that customer. **The destination market's authority — never this guide — determines whether the activity is permitted or unlawful there.** |
| **Operator** | The business that holds the gambling licence, takes the customer's stake into its own books, owes the payout and carries the regulatory risk. This guide is about an operator. |
| **Supplier** | The platform, odds engine or game studio that sells technology to an operator. **A supplier is not the operator.** The supplier layer is a neighbouring guide's material. |
| **Licensee** | The legal person named on the licence. Here the named licensee for 1xbet.com is **Caecus N.V.**, Curaçao company number **163779**. |
| **Certificate of Operation** | The licensing authority's per-brand artefact resolving a domain to an operator, a company number, a licence number, a grant date and a status — the register-grade document this guide relies on ([§5.1](#51-what-is-held-verified-at-the-authoritys-own-record)). |
| **Register** | The authority's own listing of licences, actions or public statements. A register is a **point in time**: a retrieval date is part of the fact. |
| **Licence class** | The type of instrument, not a rank of quality. Distinguish at least: a **national remote-gambling licence** (the destination market's own instrument); an **offshore or transitional licensing jurisdiction's remote licence** (issued by a jurisdiction that is not the customer's home market — the Curaçao instrument is this kind); a **B2B supply licence** (authorises supplying operators, not taking bets); and a **white-label arrangement** (a brand riding on another entity's licence, and therefore itself holding nothing locally for that market). |
| **B2C online gaming licence** | The class named on the Curaçao certificate — a licence to "offer games of chance" to players, as opposed to a supply licence. |
| **"OGL/…" versus "…/JAZ…" series** | The post-2024 Curaçao licence numbering (issued by the Curaçao Gaming Authority, as on Caecus N.V.'s certificate) versus the pre-2024 series (issued under the earlier regime). The number's **format dates the licence**. |
| **Merchant of record** | The legal entity that contracts with the acquirer and receives settlement. **It is not the brand**, and it is frequently not the entity whose licence is on the website. Identifying it is a document-level exercise. |
| **Acquirer / acquiring** | The bank or payment institution that takes the card transaction on the merchant's behalf and carries the chargeback and settlement exposure. Cymbal Bank, in the illustrative worked example, is in this role. |
| **High-risk merchant / MCC 7995** | The card networks' classification for gambling transactions; the merchant category through which the acquiring exposure in this sector arises. The machinery is a neighbouring guide's material, cross-referenced not re-derived. |
| **Affiliate** | A third-party marketer paid per referred depositing customer. Affiliates are ordinarily **not** gambling licensees, and the market's advertising rules bind the operator even when the advertising was placed by an affiliate. |
| **Deposit and withdrawal rails** | The payment methods by which a customer funds an account and withdraws winnings: cards, e-wallets, account-to-account/open-banking, instant bank transfer, prepaid vouchers, local schemes and, in some markets, crypto. The mix is a risk variable in itself. |
| **Grey market** | A market where the operator serves customers **without** a licence but where the market's authority has **not** formally branded the activity unlawful. |
| **Black market** | A market where that market's authority has publicly determined the activity to be unlicensed or unlawful, or where local law prohibits it. **Only the authority of the market concerned can make that determination**, and where this guide records it, it attributes it to that authority. |
| **Unauthorised-operator list / blocking list** | An authority's published listing of operators not authorised to serve its market (sometimes paired with blocking measures). Where this guide records such a listing, it is **the regulator's listing**, reported and attributed. |
| **Sponsored-legitimacy signal** | A top-tier club or tournament sponsorship, read by the operator as evidence of respectability and by a bank as evidence of marketing spend — **not** of regulatory standing. |
| **Instrument** | An authority's own published artefact: a decision, a judgment, an order, a certificate, a published FOI response. In this guide, an instrument is the only thing that supports a regulatory finding. |
| **Allegation** | A claim attributed to whoever made it, reported as a claim, never restated as a finding and never in this guide's own voice. |
| **Negative finding** | The **documented result of a dated search that found nothing** — e.g. a regulator's refusal to confirm or deny, or a register that does not list the entity. It proves nothing by itself; it is recorded because the gaps are material. |
| **Status not established** | The explicit label used where a matter is reported but its live/closed/concluded status cannot be established from any source read. |

### 16.3 (16.c) The Cross-References, and the Boundary

**The boundary, stated once and plainly:** *the repository covers the gambling industry's **supplier layer**, not its **operator layer**; a **supplier** sells the technology and is **not the operator**; this guide owns the **operator layer** — **this operator** — and nothing else.* The neighbours below own their material; they are cross-referenced, not re-derived, and their tables are not reproduced here.

| Boundary | The neighbouring guide (path) | What it owns, and what this guide takes from it |
| --- | --- | --- |
| **Supplier layer — B2B gambling software and platforms** | [Playtech & Its Competitors](../technology/playtech_competitors_guide.md) (`/home/ubuntu/research/technology/playtech_competitors_guide.md`) | Owns the platform, content and live-casino vendor landscape, the MGA/UKGC supplier-licensing frame and the supplier-side banking angle. **A supplier sells the technology; it is not the operator.** Used here for the white-label and platform-dependency points ([§5](#5-the-licensing-position), [§9](#9-the-banks-view--the-merchant-and-payment-counterparty-assessment)) and for nothing else. |
| **Gambling data and analytics** | [Gambling Datasets](../technology/data/gambling_datasets.md) (`/home/ubuntu/research/technology/data/gambling_datasets.md`) and [Gaming Data Warehouse & Bet Recommendation](../technology/data/gaming_dw_bet_recommendation.md) (`/home/ubuntu/research/technology/data/gaming_dw_bet_recommendation.md`) | Own the player/bet/wallet data model, RTP and recommendation analytics, and the KYC flags inside those models. This guide points at them ([§12](#12-the-data-and-analytics-angle)); it does not re-derive them. |
| **Investment and broking platforms** | [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) (`/home/ubuntu/research/banking/online_investment_trading_platforms_guide.md`) | Owns investment and broking platforms. **A betting operator is not a broker**: a stake is not a client asset, winnings are not a custody balance, the conduct regime is different and the authorities are different. The comparison appears once, in [§11](#11-the-comparison-set), and is not developed. |
| **Fraud and merchant-risk machinery** | [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) (`/home/ubuntu/research/banking/financial_fraud_detection_at_scale_guide.md`) and [AML Certifications & Exam Content](aml_certifications_exam_content_guide.md) (`/home/ubuntu/research/banking/aml_certifications_exam_content_guide.md`) | Own the fraud-monitoring and AML/merchant-risk machinery, including the **merchant-category (MCC 7995)** treatment of gambling in the card networks' own scheme rules. [§9](#9-the-banks-view--the-merchant-and-payment-counterparty-assessment), [§10](#10-the-aml-and-payment-rails-angle) and the illustrative C2–C6 conditions in [§14.6](#146-step-5--the-conditions-cymbal-sets) use that machinery and cross-reference it; they do not re-teach it. |
| **The subject's own identity and licensing core** | This guide, [§2](#2-the-identity-and-the-group-structure) and [§5](#5-the-licensing-position) | The identity gate, the licence map and the jurisdictional-scope finding are this guide's own, and are the parts a reader can lift into another operator's file. |

**What this guide will not do with its neighbours.** It does not rank this operator against named competitors (the comparison set is by tier and licence class, [§11](#11-the-comparison-set)); it does not reproduce the supplier layer's vendor tables; and it does not use a neighbouring guide's material as a source for any regulatory or licensing claim about this operator.

### 16.4 (16.d) The Closing Summary

**What this guide established.** One brand, and one register-grade licensing fact: the Curaçao Gaming Authority's own **Certificate of Operation** names **Caecus N.V.** (Curaçao company number **163779**) as the operator of 1xbet.com under licence **OGL/2024/1262/0493**, granted **7 November 2024**, status **Active** at the September 2026 retrieval, under the **National Ordinance on Games of Chance** (*Landsverordening op de kansspelen*, P.B. 2024, no. 157) ✅. Alongside it: a concluded Dutch enforcement matter with dated instruments and named parties ✅; a Liberian tribunal judgment that revoked a local licensee's licence and dismissed the complaint against the brand because a brand is not a legal entity ✅; a documented British negative in which the Gambling Commission declined to confirm or deny whether it held any information at all, citing s.31(3) FOIA 2000 ✅; an identified Ukrainian sanctions instrument whose annex was not read at source ⚠✅; and two FC Barcelona announcements, dated **3 July 2019** and **1 July 2024**, verified at the sponsor's own site ✅.

**What this guide refused.** It refused to restate an allegation as a finding, to name a private individual on the strength of journalistic sourcing, to treat a company claim as a fact, to read a regulator's silence as clearance, to treat a sponsorship as evidence of regulatory standing, and to accept a brand as a counterparty. It respected the repository's rule that **Cymbal Bank is the only bank persona**, and it named no real bank, payment provider, sponsor or regulator as this operator's client, partner, acquirer or counterparty absent that party's or the operator's own public statement.

**What this guide could not establish, and why that is the point.** The ownership and control of the licensee, the parent and holding structure, the brand family's register-level relationships, the merchant of record, every financial and performance figure, the Irish licence at register level, the Russian register, the licensing position in most markets, and the status of most alleged matters. Each of those is recorded as the **negative result of a dated search** — the only honest way to hold a gap — and not one of them is an accusation. In this subject matter the gaps are the material.

**The single sentence a bank should carry from this file, in place of every narrative it replaces:** the identity of a gambling counterparty is not the brand on the deck, it is the legal person named at the register, licensed by a named authority, in a named class, for a named set of markets, as at a named date — and every other question in the file, from the merchant of record to the customer-jurisdiction mismatch, is downstream of that one.

**a brand is not a licence.**
