# Trade Finance: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Trade Finance / Banking Architecture — Trade Instruments, LC Lifecycle, ICC Rules, SWIFT Messages, Risk, Digitization, Banking Systems, Worked Example  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [Overview](#1-overview)
   - 1.1 [What Is Trade Finance?](#11-what-is-trade-finance)
   - 1.2 [Why Trade Finance Exists: The Trust and Timing Gap](#12-why-trade-finance-exists-the-trust-and-timing-gap)
   - 1.3 [The Trade Finance Gap](#13-the-trade-finance-gap)
   - 1.4 [The Trade Finance Landscape at a Glance](#14-the-trade-finance-landscape-at-a-glance)
2. [The Instruments](#2-the-instruments)
   - 2.1 [Letter of Credit (LC)](#21-letter-of-credit-lc)
   - 2.2 [Documentary Collections](#22-documentary-collections)
   - 2.3 [Open Account](#23-open-account)
   - 2.4 [Guarantees and Bonds](#24-guarantees-and-bonds)
   - 2.5 [Instruments Comparison](#25-instruments-comparison)
3. [The LC Lifecycle](#3-the-lc-lifecycle)
   - 3.1 [The Parties](#31-the-parties)
   - 3.2 [The Flow: Application to Payment](#32-the-flow-application-to-payment)
   - 3.3 [LC Lifecycle Step-by-Step](#33-lc-lifecycle-step-by-step)
4. [The Rules](#4-the-rules)
   - 4.1 [UCP 600](#41-ucp-600)
   - 4.2 [URC 522](#42-urc-522)
   - 4.3 [URDG 758](#43-urdg-758)
   - 4.4 [Incoterms 2020](#44-incoterms-2020)
   - 4.5 [Rules Comparison](#45-rules-comparison)
5. [The SWIFT Messages](#5-the-swift-messages)
   - 5.1 [The SWIFT Network and Trade](#51-the-swift-network-and-trade)
   - 5.2 [The MT700 Family](#52-the-mt700-family)
   - 5.3 [Message Comparison](#53-message-comparison)
6. [The Risks](#6-the-risks)
   - 6.1 [Credit and Country Risk](#61-credit-and-country-risk)
   - 6.2 [Documentary Fraud](#62-documentary-fraud)
   - 6.3 [AML and Sanctions: Trade-Based Money Laundering](#63-aml-and-sanctions-trade-based-money-laundering)
   - 6.4 [Operational Risk: Discrepancies](#64-operational-risk-discrepancies)
   - 6.5 [Risk Comparison](#65-risk-comparison)
7. [The Digitization](#7-the-digitization)
   - 7.1 [Why Trade Finance Is Still Paper-Heavy](#71-why-trade-finance-is-still-paper-heavy)
   - 7.2 [The Platform Generation: Bolero, essDOCS, Contour](#72-the-platform-generation-bolero-essdocs-contour)
   - 7.3 [The Blockchain Era: TradeLens, we.trade, Marco Polo](#73-the-blockchain-era-tradelens-wetrade-marco-polo)
   - 7.4 [What the Shutdowns Taught Us](#74-what-the-shutdowns-taught-us)
   - 7.5 [Digitization Comparison](#75-digitization-comparison)
8. [The Systems](#8-the-systems)
   - 8.1 [The Bank Trade Systems Landscape](#81-the-bank-trade-systems-landscape)
   - 8.2 [Vendors](#82-vendors)
   - 8.3 [Systems Comparison](#83-systems-comparison)
9. [Worked Example: A Commodity LC Through a CACIB-Style Bank](#9-worked-example-a-commodity-lc-through-a-cacib-style-bank)
   - 9.1 [The Scenario](#91-the-scenario)
   - 9.2 [The Full LC Flow](#92-the-full-lc-flow)
   - 9.3 [The Lessons](#93-the-lessons)
10. [Summary: One Page](#10-summary-one-page)
11. [Glossary](#11-glossary)
12. [Related Guides and Sources](#12-related-guides-and-sources)

---

# 1. Overview

## 1.1 What Is Trade Finance?

**Trade finance** is the financing and risk-mitigation apparatus that sits underneath international trade: the instruments, rules, and payment mechanisms that let a seller in one country ship goods to a buyer in another and get paid — and let the buyer receive the goods and pay — without either side having to trust the other, or fund the whole transaction itself, upfront.

The definition has three layers, and it is worth being precise about each because they map to different parts of a bank's trade franchise:

- **The payment/settlement layer** — *how* the buyer pays the seller. The choice of instrument (open account, documentary collection, letter of credit) allocates risk between the parties.
- **The financing layer** — *who funds the working capital* between shipment and payment. Pre-shipment finance (packing loans, PO finance), post-shipment finance (negotiation, discounting, forfaiting), receivables finance, and inventory finance.
- **The risk-mitigation layer** — *who guarantees the performance*. Guarantees and bonds (bid, performance, advance payment), standby letters of credit, and export credit insurance.

A widely repeated industry statistic is that **80–90% of world trade relies on trade finance** in some form (financing, guarantees, or insurance). That figure is routinely quoted in WTO and ICC material, but treat it as an industry convention rather than a precisely audited number — **flagged**: it is a commonly cited estimate, not a measured census. ICC's *Rethinking Trade & Finance 2023* report estimated the global trade finance market at roughly **USD 5.2 trillion** in 2022 — again an industry estimate, **flagged** as such.

What makes trade finance distinctive as a banking business:

- It is **documentary**: banks deal in documents, not goods (UCP 600 Article 5 — see Section 4).
- It is **rule-governed**: one set of private rules (the ICC's) is used by essentially every bank in the world, which is what makes the instruments portable across borders.
- It is **balance-sheet light relative to notional**: an LC is a contingent liability until documents are presented; a guarantee is contingent until called.
- It is **deeply embedded in physical supply chains**: the finance follows the cargo, the bill of lading, the customs declaration — which is why trade finance and trade logistics digitization keep colliding (Section 7).

Trade finance sits at the junction of several sibling guides in this repository: it is a **process** (see `end_to_end_banking_processes.md`), a **product family** inside the universal banking model (see `universal_banking_model_guide.md`), a **limits/exposure problem** (see `banking_limits_domain_guide.md` — an LC creates a contingent exposure that consumes a customer limit until it expires or is drawn), and a **system integration problem** (see `core_banking_systems_guide.md`, `temenos_guide.md`, `oracle_flexcube_data_model_guide.md` — all have trade modules). It is also the **payments-adjacent** world of SWIFT messages (see `iso_20022_core_processes_guide.md` and, for the cross-border settlement angle, `../technology/late_arriving_data_guide.md`).

## 1.2 Why Trade Finance Exists: The Trust and Timing Gap

Two parties want to trade. The exporter wants to ship only against payment (or a guarantee of payment); the importer wants to pay only against goods (or proof of shipment). And even when they trust each other, the money sits idle for the 20–60 days the goods are in transit. Trade finance instruments exist to bridge exactly these two gaps:

| Gap | Problem | Trade finance answer |
|---|---|---|
| **Trust gap** | Seller doesn't trust buyer to pay; buyer doesn't trust seller to ship | A bank intermediates: the LC makes the bank's credit stand behind payment; collections use banks as document conduits |
| **Timing gap** | Money is tied up in transit / before resale | Financing: the bank funds the exporter pre-shipment or post-shipment, or funds the importer's payables |
| **Performance gap** | A party must commit to a contract before the other performs | Guarantees and bonds: a bank promises to pay if the principal fails to perform |

The instruments in Section 2 are essentially a spectrum of how much of the trust gap the bank absorbs — from open account (bank absorbs none) to confirmed LC (bank absorbs nearly all).

## 1.3 The Trade Finance Gap

The **trade finance gap** is the difference between the trade financing corporates request and the financing banks actually approve — i.e., *unmet demand for trade finance*. The Asian Development Bank (ADB) measures it through its biennial *Trade Finance Gaps, Growth, and Jobs Survey*.

**Verified numbers (ADB surveys, flagged as survey estimates rather than audited statistics):**

| Survey year | Gap measured for | Reported gap | Notes |
|---|---|---|---|
| 2019 survey | 2018 | ~USD 1.5 trillion | Baseline before COVID |
| 2021 survey | 2020 | ~USD 1.7 trillion | COVID shock year |
| 2023 survey | 2022 | **USD 2.5 trillion** | Record; largest single-period increase since the survey began; +47% vs 2020 |

The 2023 survey (released 5 September 2023) attributes the jump to rising interest rates, inflation, flagging growth, and geopolitical volatility shrinking banks' capacity and appetite for trade risk. The 2025 survey (published 2025, gathering input from 110+ trade finance providers) reported the gap **remaining at roughly USD 2.5 trillion** and flagged that trade diversification and supply-chain reorganization would keep lifting demand. **Flag**: all of these are ADB survey *estimates* — the underlying methodology is self-reported demand-and-rejection data from banks and corporates, not a measured flow.

Two structural points matter for an architect:

1. **The gap is concentrated where trade finance is needed most** — SMEs and frontier markets. ADB surveys consistently report that rejection rates for SME trade finance applications run far higher than for corporates (ADB has reported rejection rates in the 40–45% range for SME applications in recent surveys — **flagged** as survey-reported ranges).
2. **De-risking amplifies the gap** — banks withdrawing correspondent and trade relationships from high-risk or low-return jurisdictions push more demand into the unmet bucket. This is a risk/limits problem as much as an economics problem (see `banking_limits_domain_guide.md`).

For a bank, the gap is opportunity: it is precisely the demand that more efficient processing, better credit structures, and digitization (Section 7) can serve profitably. The gap is also why multilateral institutions (ADB's Trade Finance Program, IFC's Global Trade Finance Program) exist — they share risk with banks to keep trade lines open.

## 1.4 The Trade Finance Landscape at a Glance

| Aspect | Description |
|---|---|
| **Definition** | Financing and risk mitigation for international trade: payment instruments, working capital, and guarantees across borders |
| **Core problem solved** | The trust gap (payment vs delivery) and the timing gap (funding the goods-in-transit period) |
| **Market scale** | Global trade finance market est. ~USD 5.2 trillion (2022, ICC estimate — **flagged**); unmet demand (the gap) ~USD 2.5 trillion (2022, ADB survey — **flagged**) |
| **Primary instruments** | Letter of credit, documentary collection, open account, guarantees/bonds (Section 2) |
| **Governing rules** | UCP 600 (LCs), URC 522 (collections), URDG 758 (guarantees), Incoterms 2020 (delivery terms) — Section 4 |
| **Messaging network** | SWIFT: MT700 family for LCs, MT400/MT422 for collections, MT760 for guarantees — Section 5 |
| **Risk profile** | Credit, country, documentary fraud, trade-based money laundering, sanctions, operational — Section 6 |
| **Digitization state** | Paper-based core with a digital overlay: eBL platforms (Bolero, essDOCS/ICE), DLT consortia (Contour), and high-profile shutdowns (TradeLens, we.trade, Marco Polo) — Section 7 |
| **Bank systems** | Trade processing platforms (Finastra, Surecomp, CGI), core-banking trade modules (Temenos, Oracle Flexcube), SWIFT connectivity — Section 8 |

---

# 2. The Instruments

Trade finance instruments form a spectrum. At one end the bank absorbs almost no risk (open account); at the other it absorbs almost all of it (confirmed LC). The choice is a negotiation between buyer and seller, governed by the sale contract and — where the parties choose to import them — by the ICC rules.

## 2.1 Letter of Credit (LC)

A **letter of credit** is a written undertaking by a bank (the *issuing bank*), issued at the request of the buyer (the *applicant*), to pay the seller (the *beneficiary*) a stated amount against presentation of documents that comply with the credit's terms. It is the classic trade instrument: **verified definition** — under UCP 600 Article 2, a credit is "any arrangement, however named or described, that is irrevocable and thereby constitutes a definite undertaking of the issuing bank to honour a complying presentation."

Core properties (all verified against UCP 600):

- **Irrevocable**: since UCP 600, all credits are irrevocable unless stated otherwise (Article 3). The issuing bank cannot unilaterally cancel or amend.
- **Documentary**: the bank's obligation is triggered by *documents*, not goods or services (Article 5 — "Credits are separate transactions from the sale or other contract(s)"). A bank that receives a complying presentation must pay even if the goods are defective, missing, or destroyed — the remedy lies in the sale contract, not the credit.
- **Independent**: the credit is independent of the underlying contract (Article 4). The bank never reads the sale contract.
- **Definite undertaking**: the issuing bank's obligation is primary, not secondary (Article 7); the confirming bank's is identical (Article 8).

Common LC types:

| Type | Mechanism | Notes |
|---|---|---|
| **Sight LC** | Payment immediately against complying presentation | Simplest form; documents presented, checked, paid |
| **Usance / deferred-payment LC** | Bank pays at a future date (e.g., 30/60/90 days after sight or after B/L date) | Buyer gets credit; seller gets a bank-accepted obligation that can be discounted |
| **Confirmed LC** | A second bank (in the seller's country) adds its own undertaking | Removes issuing-bank and country risk for the seller — the workhorse for trade into emerging markets |
| **Transferable LC** | Beneficiary can transfer rights to an upstream supplier | Used with intermediaries/traders; only if the credit expressly says "transferable" (Article 38) |
| **Back-to-back LC** | Two separate LCs: one issued against the other | Trader finances purchase against a confirmed sale |
| **Standby LC (SBLC)** | A "standby" promise to pay if the applicant fails to perform an obligation | A guarantee in LC clothing; often governed by ISP98 or URDG rather than UCP |

The LC is the most risk-complete instrument for the seller and the most balance-sheet-consuming for the bank (a contingent exposure for the applicant — see `banking_limits_domain_guide.md`). It is also the instrument with the most operational machinery: a full lifecycle (Section 3), a dedicated rule set (UCP 600, Section 4), a dedicated SWIFT message family (MT700, Section 5), and dedicated examination discipline (ISBP 745).

## 2.2 Documentary Collections

A **documentary collection** is the arrangement where the seller ships the goods, hands the shipping documents to *its* bank (the *remitting bank*), and instructs that the documents be released to the buyer only against payment or acceptance. **Verified**: governed by the ICC Uniform Rules for Collections, URC 522 (1995 revision, ICC Publication No. 522).

Two forms, distinguished by what the buyer must do to get the documents:

- **D/P — Documents against Payment (sight collection)**: the buyer pays the full amount, then receives the documents. The documents are the leverage: no payment, no bill of lading, no goods.
- **D/A — Documents against Acceptance (usance collection)**: the buyer *accepts* a bill of exchange (promises to pay at a future date), then receives the documents. Payment happens later, at maturity.

Critical property — **the banks are agents, not principals**: under URC 522 the remitting and collecting banks act only as conduits and have **no obligation to pay** if the buyer refuses. They must present documents in accordance with the collection instruction (URC 522 Articles 4, 5, 12) and must act in good faith (Article 9), but the payment risk stays with the seller. If the buyer defaults, the seller's remedy is against the buyer — the bank has no liability (subject to its duties of care and to following instructions).

Collections are cheaper than LCs (no issuance of a bank obligation, lower fees) and are the traditional instrument for established trade relationships, moderate-value transactions, and markets where LCs are considered expensive or unnecessary. The risk, however, is asymmetric: the seller ships first and has no bank payment undertaking.

## 2.3 Open Account

**Open account** is the simplest trade settlement: the seller ships the goods and invoices the buyer, who pays at an agreed later date (typically 30–90 days). No bank instrument, no documents routed through banks, no payment undertaking. **Verified as a concept**: it is simply trade settled on the buyer's promise, the mirror image of cash-in-advance.

Open account dominates *intra-group* and *established-partner* trade, and is the fastest-growing share of world trade — ironically, because trade is moving *away* from bank-intermediated instruments toward corporate-to-corporate credit. The seller bears the full payment risk and finances the whole cycle; the buyer gets the goods and free credit.

Banks still participate, but indirectly, through:

- **Receivables/payables finance** (factoring, forfaiting, approved payables finance) — see `supply_chain_finance_guide.md`, which covers this product family in depth.
- **Credit insurance** — the seller insures the receivable with an ECA or private insurer, then often sells the insured receivable.
- **Dynamic discounting platforms** — buyer-funded early payment.

The instrument-level risk table (2.5) treats open account as the risk benchmark: seller risk ~maximum, bank risk ~zero (until the receivable is financed).

## 2.4 Guarantees and Bonds

A **bank guarantee** (or **bond** — the terms are used interchangeably in practice, with "bond" more common in construction/commodities and "guarantee" in banking) is a written undertaking by a bank to pay the beneficiary a sum of money if the bank's *principal* (the bank's customer) fails to perform an underlying obligation. **Verified**: the ICC's governing rules are the Uniform Rules for Demand Guarantees, URDG 758 (2010 revision, ICC Publication No. 758).

Common types:

| Bond type | Protects | Trigger |
|---|---|---|
| **Bid bond / tender bond** | The buyer/tenderer | Bidder wins and fails to sign the contract or provide the performance bond |
| **Performance bond** | The buyer | Contractor fails to perform per contract |
| **Advance payment guarantee** | The buyer | Contractor fails to repay the advance paid to it |
| **Retention / warranty bond** | The buyer | Defects during the warranty period; release of retention money |
| **Payment guarantee** | The seller | Buyer fails to pay |

The defining distinction is **demand vs conditional**:

- **Demand guarantee (on-demand)**: payable on the beneficiary's *demand* plus (under URDG 758) a supporting statement — the bank does not investigate the underlying dispute. URDG 758 requires the demand to be accompanied by a statement of the principal's breach (Article 15) but the bank does not adjudicate it.
- **Conditional guarantee**: payable only on proof of the underlying breach (court judgment, arbitration award, third-party certificate) — increasingly rare in cross-border trade because beneficiaries dislike litigating before getting paid.

Standby LCs (SBLCs) occupy the same functional space as demand guarantees but are issued under LC mechanics (typically ISP98 or, where UCP-governed, UCP 600). Banks treat guarantees as **contingent liabilities** — they consume limit but not cash until called — which is exactly the exposure-angle cross-reference into `banking_limits_domain_guide.md`.

## 2.5 Instruments Comparison

| Instrument | Mechanism | Risk | Notes |
|---|---|---|---|
| **Letter of credit (LC)** | Issuing bank undertakes to pay beneficiary against complying documents (UCP 600) | Seller: low (bank risk instead of buyer risk). Bank: contingent credit risk on applicant + documentary/operational risk | The trust instrument par excellence; full lifecycle, MT700 family, examination discipline (ISBP 745); confirmation removes country risk |
| **Documentary collection (D/P, D/A)** | Banks transmit documents; release against payment or acceptance (URC 522) | Seller: high — no bank payment undertaking; buyer pays at document release | Cheaper than LC; banks are agents, not principals; D/A adds acceptance (usance) risk |
| **Open account** | Seller ships, invoices, buyer pays later | Seller: maximum. Bank: none unless the receivable is financed | Dominant for established partners; banks re-enter via factoring/APF (see `supply_chain_finance_guide.md`) |
| **Guarantee / bond** | Bank pays beneficiary if principal fails to perform (URDG 758) | Bank: contingent liability until expiry/cancellation; fraud risk on abusive calls | On-demand vs conditional; bid/performance/advance-payment/retention variants; SBLCs are the LC-mechanics twin |

---

# 3. The LC Lifecycle

The letter of credit has the richest lifecycle of any trade instrument, and it is the one that exercises the most bank machinery: limits, issuance, SWIFT messaging, document examination, payments, and reimbursement. This section walks the parties and the flow end to end; Section 9 turns the same flow into a worked example with timings and messages.

## 3.1 The Parties

**Verified party roles** (per UCP 600 Articles 2, 7, 8, 9, 12 — and standard trade practice):

| Party | Role | Bank or not | UCP anchor |
|---|---|---|---|
| **Applicant** | The buyer/importer who requests the credit and whose obligation the credit secures | Corporate | Art 2 (definition) |
| **Beneficiary** | The seller/exporter in whose favour the credit is issued; the party entitled to payment against complying documents | Corporate | Art 2 (definition) |
| **Issuing bank** | The applicant's bank; issues the credit and gives the *definite undertaking* to honour complying presentations | Bank | Art 2, Art 7 |
| **Advising bank** | The bank (usually in the beneficiary's country) that advises the credit to the beneficiary and authenticates it; has no payment obligation | Bank | Art 2, Art 9 |
| **Confirming bank** | A bank that *adds its confirmation* — its own definite undertaking, identical to the issuing bank's | Bank | Art 2, Art 8 |
| **Nominated bank** | The bank with which the credit is available (e.g., the confirming bank, or the negotiating bank); pays/accepts/negotiates per the credit | Bank | Art 2, Art 12 |

Two structural points:

1. **Issuing vs confirming is the critical risk split.** The issuing bank's undertaking is the applicant's country and the applicant's credit. The confirming bank's undertaking is the confirming bank's own — which is why confirmation is priced on the *issuing bank's* risk and the *issuing country's* risk, not the applicant's. For a CACIB-style bank, confirmation of Asian/EM-issued credits is a standard product with dedicated country limits and pricing grids.
2. **The advising bank's job is authentication, not payment.** Article 9 requires the advising bank to advise the credit (or an amendment) "without responsibility" for the content, but it must take reasonable care to check the apparent authenticity of the message. A bank that merely forwards without authenticating has breached its duty.

## 3.2 The Flow: Application to Payment

The canonical LC flow, in eleven steps. (Timings and message numbers are elaborated in the worked example, Section 9.)

1. **Sale contract** — buyer and seller agree terms: goods, price, Incoterms (e.g., FOB Santos), delivery window, and that payment is by LC.
2. **Application** — the buyer (applicant) applies to its bank (the issuing bank) for the credit, specifying every term the documents must satisfy. The application is, in effect, the credit's constitution: anything not in the application cannot be in the credit.
3. **Credit assessment / limits** — the issuing bank checks the applicant's facility and draws a **contingent limit** for the LC (see `banking_limits_domain_guide.md`); KYC/sanctions screening of the parties, goods, and jurisdictions (see Section 6.3).
4. **Issuance** — the issuing bank issues the credit, almost always as an **MT700** over SWIFT to the advising bank (Section 5).
5. **Advice** — the advising bank authenticates and advises the credit to the beneficiary, and (if instructed) adds confirmation.
6. **Shipment** — the beneficiary ships the goods within the shipment window (44C/44D) and obtains the transport documents — the bill of lading being the key title document.
7. **Presentation** — the beneficiary assembles the required documents (invoice, B/L, insurance, certificates) and presents them to the nominated bank (typically the advising/confirming bank) within the presentation period (typically 21 days after shipment, per the credit; UCP default, Article 14(c)).
8. **Examination** — the nominated bank checks the documents against the credit terms: on their face, within a maximum of **five banking days following the day of presentation** (UCP 600 Article 14(b)). Any discrepancy must be communicated in a single notice (Article 16).
9. **Honour / payment** — for a sight credit: pay on complying presentation. For a usance credit: accept the draft or incur the deferred-payment undertaking, payable at maturity.
10. **Settlement and reimbursement** — the paying/negotiating bank claims reimbursement from the issuing bank (via SWIFT; reimbursement instructions per URR 725 where applicable), and the issuing bank settles (typically MT202/MT103 — see `../technology/late_arriving_data_guide.md` for the cross-border payment mechanics).
11. **Document release and delivery** — the issuing bank releases the documents to the applicant against payment or acceptance per the credit terms; the applicant uses the bill of lading to take delivery of the goods.

## 3.3 LC Lifecycle Step-by-Step

| Step | Party acting | Action |
|---|---|---|
| 1. Sale contract | Applicant + beneficiary | Agree goods, price, Incoterms, delivery window, LC terms |
| 2. LC application | Applicant → issuing bank | Submit application stating all credit terms and documents required |
| 3. Credit and limits | Issuing bank | Check facility, book contingent limit, KYC/sanctions screening |
| 4. Issuance | Issuing bank | Issue credit via SWIFT **MT700** to advising bank |
| 5. Advice (+ confirmation) | Advising bank (possibly confirming) | Authenticate, advise beneficiary; add confirmation if instructed |
| 6. Shipment | Beneficiary | Ship goods; obtain clean on-board bill of lading etc. |
| 7. Presentation | Beneficiary → nominated bank | Present documents within validity and presentation period |
| 8. Examination | Nominated / issuing bank | Check documents on their face (max 5 banking days, Art 14(b)); reject with single notice if discrepant (Art 16) |
| 9. Honour | Nominated bank (sight: pay; usance: accept/defer) | Pay at sight, or accept draft / incur deferred undertaking |
| 10. Reimbursement | Nominated bank → issuing bank | Claim reimbursement; issuing bank settles (MT202/103); applicant's account debited |
| 11. Delivery | Issuing bank → applicant | Release documents against payment/acceptance; applicant takes delivery of goods |

---

# 4. The Rules

Trade finance runs on a surprisingly small set of private, non-governmental rules published by the **International Chamber of Commerce (ICC)** in Paris. Because every bank and every trade jurisdiction adopts them, a letter of credit issued in Singapore is examined the same way in Santos, Hamburg, and Mumbai. The four pillars are UCP 600 (credits), URC 522 (collections), URDG 758 (guarantees), and Incoterms 2020 (delivery terms).

## 4.1 UCP 600

**Verified**: the *Uniform Customs and Practice for Documentary Credits* (UCP), 2007 Revision, ICC Publication No. 600. Approved by the ICC Banking Commission on 25 October 2006 (91-0 vote) and effective **1 July 2007**, replacing UCP 500 (1993). It applies to any documentary credit that expressly incorporates it (Article 1).

Structure: 39 articles covering definitions (Article 2), interpretation (Article 3), the credit/contract separation (Articles 4–5), the banks' undertakings (Articles 7–10), the examination standard (Article 14), discrepant presentations (Article 16), document-specific rules (Articles 18–28: commercial invoice, transport documents, insurance), disclaimers (Articles 34–37), and transfer/assignment (Articles 38–39).

The articles that matter most to a practitioner:

| Article | Subject | Why it matters |
|---|---|---|
| Art 2 | Definitions | The agreed vocabulary: honour, complying presentation, applicant, beneficiary, nominated bank |
| Art 4 / 5 | Independence / documents-not-goods | The bank's obligation is documentary only — the doctrinal heart of the LC |
| Art 7 / 8 | Issuing / confirming bank undertaking | The definite, irrevocable undertakings that make the LC a bank obligation |
| Art 14 | Standard for examination | Examination on the face of documents; max **5 banking days** after presentation |
| Art 15 / 16 | Complying presentation / discrepancies | Single-notice rejection rule; banks that fail to give one notice lose the right to claim discrepancies |
| Art 18–28 | Document rules | Invoice, transport (incl. bill of lading, Art 20), insurance specifics |
| Art 34–37 | Disclaimers | Banks are not liable for acts of third parties, transmission, force majeure — the fraud-risk boundary |

Two companions to UCP 600 are worth knowing (both **verified as ICC publications**):

- **eUCP** — the *Supplement to UCP for Electronic Presentation* (version 2.1, 2019; original 1.1, 2007): how credits may provide for presentation of electronic records.
- **ISBP 745** — the *International Standard Banking Practice* (2013 revision): the detailed examination practice banks must follow under UCP 600 — the "how to examine" companion to the "what the rules are" UCP.

## 4.2 URC 522

**Verified**: the *Uniform Rules for Collections*, 1995 Revision, ICC Publication No. 522. Adopted by the ICC Council in June 1995, replacing URC 322 (1978). The URC lineage dates to 1956 (first publication), revised 1967 and 1978. Applies to any collection incorporating it (Article 1).

26 articles governing documentary collections (Section 2.2): the collection instruction and its mandatory content (Articles 4–5), the presentation (Articles 5–6), release of documents against payment/acceptance (Article 7), the banks' duty of good faith and reasonable care (Article 9), disclaimer for acts of third parties (Article 14), the collecting bank's obligation to notify outcome (Article 26), and the critical rule that **banks have no liability for payment** — they are agents, not principals.

The key asymmetry with UCP 600: a collection's banks execute instructions; an LC's banks give undertakings. URC 522 creates duties, not payment obligations.

## 4.3 URDG 758

**Verified**: the *Uniform Rules for Demand Guarantees*, 2010 Revision, ICC Publication No. 758. Replaced URDG 458 (1992), effective 1 July 2010. Applies to any demand guarantee or counter-guarantee that incorporates it (Article 1). 35 articles, with model forms in the annexes.

Key mechanics (verified against the published text):

- **Demand guarantees are payable on demand plus a supporting statement** — Article 15 requires the demand to be accompanied by a statement that the principal is in breach; the guarantor does not adjudicate the dispute.
- **Expiry** — by date, by event, or the earlier of both if specified (Article 2 definitions); guarantees must state an expiry (Article 25).
- **Counter-guarantees** — the inter-bank layer: the instructing party's bank issues a counter-guarantee to the local issuing bank, which issues the guarantee to the beneficiary — the standard structure for cross-border bonds (a foreign bank cannot always issue locally; the local bank issues against the counter-guarantee).
- **Fraud/abuse** — URDG 758 Article 26 addresses the "unfair calling" problem, but the rules themselves are payment-first: the fraud exception is a matter of applicable law (e.g., injunction jurisprudence in England and Singapore), not the rules.

## 4.4 Incoterms 2020

**Verified**: *Incoterms 2020* — the ICC Rules for the Use of Domestic and International Trade Terms, ICC Publication No. 723, published September 2019 and **effective 1 January 2020**, replacing Incoterms 2010. Eleven three-letter terms in two groups.

| Group | Terms | Notes |
|---|---|---|
| **Any mode of transport** | EXW, FCA, CPT, CIP, DAP, DPU, DDP | DPU (Delivered at Place Unloaded) is the renamed DAT (Delivered at Terminal) |
| **Sea and inland waterway only** | FAS, FOB, CFR, CIF | FOB remains the most-used commodity term |

Incoterms allocate three things between buyer and seller: **delivery point (risk transfer), costs (freight, insurance, duties), and obligations (export/import clearance, security)**. They do *not* govern payment — they sit alongside the payment instrument. What they do for trade finance:

- They determine **who arranges and pays for carriage and insurance** — which decides which transport and insurance documents an LC must require (e.g., FOB → buyer's insurance; CIF/CIP → seller's, at 110% of value for CIF).
- The 2020 changes of note: **FCA + on-board bill of lading option** (buyer instructs carrier to issue an on-board B/L after FCA delivery — solves a long-standing LC mismatch), **CIP now requires Institute Cargo Clauses (A)** while CIF stays at (C), and explicit **security-related obligations** for both parties.

## 4.5 Rules Comparison

| Rule | Scope | Notes |
|---|---|---|
| **UCP 600** (2007, ICC Pub 600) | Documentary credits (LCs) | 39 articles; irrevocability, documentary independence, 5-banking-day examination; companions: eUCP 2.1, ISBP 745 |
| **URC 522** (1995, ICC Pub 522) | Documentary collections (D/P, D/A) | 26 articles; banks are agents, no payment undertaking; release against payment/acceptance |
| **URDG 758** (2010, ICC Pub 758) | Demand guarantees and counter-guarantees | 35 articles; on-demand + supporting statement (Art 15); expiry rules; model forms in annexes; replaced URDG 458 |
| **Incoterms 2020** (2019, ICC Pub 723) | Delivery terms in sale contracts | 11 terms; FOB/FAS/CFR/CIF (sea) + EXW/FCA/CPT/CIP/DAP/DPU/DDP (any mode); effective 1 Jan 2020 |

Related rule sets not covered in depth here but worth knowing: **URR 725** (Uniform Rules for Bank-to-Bank Reimbursements, 2008) — governs reimbursement claims between banks under LCs; **ISP98** (ICC Pub 590, 1998) — standby LC practice; **eUCP** (electronic presentation). All **verified as ICC publications**.

---

# 5. The SWIFT Messages

## 5.1 The SWIFT Network and Trade

**SWIFT** (Society for Worldwide Interbank Financial Telecommunication) is the cooperative owned by its member banks that runs the secure FIN messaging network used for almost all cross-border bank-to-bank trade messages, plus the BIC directory (ISO 9362 bank identifiers — the 8/11-character codes like `CRLYLALA`). Trade finance messages are a large share of SWIFT's traffic alongside payments and securities.

Two message standards coexist:

- **MT (Message Type) format** — the classic FIN format (e.g., MT700, MT103), fixed-field, character-limited, still dominant for trade.
- **MX format** — the ISO 20022 XML standard, now mandatory for many payment categories and making inroads into trade (see `iso_20022_core_processes_guide.md` for the payment side; trade migration to ISO 20022 is ongoing — **flagged**: trade MTs are migrating to MX equivalents under SWIFT's roadmap, but MT700-family remains the de facto standard in production).

## 5.2 The MT700 Family

**Verified**: the **MT700** is the SWIFT message "Issue of a Documentary Credit" — the message by which an issuing bank transmits an operative credit to the advising bank (or directly to the beneficiary, or via a third bank). It is the LC's digital embodiment: everything the beneficiary needs to know, in fields.

Key MT700 fields (standard SWIFT field numbers — **verified for the core fields**, exact code-letter details are per the SWIFT User Handbook):

| Field | Content |
|---|---|
| 20 | Documentary credit number (mandatory) |
| 27 | Sequence of total (for MT700 + MT701 splits) |
| 31C | Date of issue |
| 31D | Date and place of expiry |
| 32B | Currency code, amount |
| 39A/B/C | Amount tolerance (percentage), max amount, additional amounts |
| 40A | Form of documentary credit (irrevocable / irrevocable transferable / standby) |
| 41A/D | Available with ... by (bank + by payment/acceptance/negotiation/deferred payment) |
| 42C | Drafts at (e.g., "at sight", "60 days after B/L date") |
| 42A/D | Drawee (the bank on which drafts are drawn) |
| 43P / 43T | Partial shipments / transshipment allowed or not |
| 44A/B/C/D/E | Place of taking in charge / port of loading / port of discharge / place of final destination / latest shipment date |
| 45A | Description of goods and/or services |
| 46A | Documents required (the checklist that drives examination) |
| 47A | Additional conditions |
| 48 | Period for presentation (e.g., "21 days after shipment date") |
| 49 | Confirmation instructions (confirm / without / may add) |
| 50 | Applicant |
| 59 | Beneficiary |
| 71A | Charges (who bears which charges) |
| 72 | Sender to receiver information |
| 78 | Instructions to paying/accepting/negotiating bank |

The **MT700 family** — the messages surrounding issuance and its lifecycle:

- **MT701 — Issue of a Documentary Credit (additional amount/conditions)**: used with MT700 when the credit exceeds one message's field capacity or is issued in tranches (the "sequence of total" fields 27 tie them together).
- **MT707 — Amendment to a Documentary Credit**: every change to an issued credit (extension, amount change, document change) — subject to beneficiary acceptance (UCP 600 Article 10).
- **MT710 — Advice of a Third Bank's (or Non-Bank's) Documentary Credit**: when the advising bank advises a credit issued by another bank.
- **MT720 / MT721 — Transfer of a Documentary Credit**: the transferable-credit mechanism (Article 38); MT721 when transferring a third bank's credit.
- **MT730 — Acknowledgement**: confirming receipt of an MT700/MT707/MT710/MT720.
- **MT750 — Advice of Discrepancy**: the nominated bank notifies the issuing bank that the presentation is discrepant and asks for a waiver.
- **MT752 — Authorization to Pay, Accept or Negotiate**: the issuing bank's response authorizing settlement (or MT799 for refusals/negotiation).
- **MT754 — Advice of Payment/Acceptance/Negotiation**: settlement confirmation.
- **MT740/742 — Reimbursement authorization / claim** (under URR 725).
- **MT799 — Free Format**: the catch-all for anything not covered (discrepancy waivers, clarifications, negotiation requests).

Two related families complete the trade picture: **MT400/MT422** (collections: advice of payment / advice of honour and/or acceptance) and **MT760** (guarantee — the demand-guarantee/standby message). All **verified as standard SWIFT message types** per the SWIFT standards documentation.

## 5.3 Message Comparison

| Message | Purpose | Notes |
|---|---|---|
| **MT700** | Issue of a documentary credit | The operative LC; field 20 = LC number; mandatory core fields drive the whole lifecycle |
| **MT701** | Additional amount(s) / conditions | Continuation of MT700 (long credits, tranches); sequence-of-total pairing |
| **MT707** | Amendment to a credit | Needs beneficiary acceptance (UCP 600 Art 10) |
| **MT710** | Advice of third bank's credit | Advising path when issuer is not the sender |
| **MT720 / MT721** | Transfer of credit | Transferable-credit mechanics (Art 38) |
| **MT730** | Acknowledgement | Confirms receipt of an MT700/707/710/720 |
| **MT750** | Advice of discrepancy | Nominated bank flags discrepancies, requests waiver |
| **MT752** | Authorization to pay/accept/negotiate | Issuing bank's settlement authorization |
| **MT754** | Advice of payment/acceptance/negotiation | Settlement confirmation back to nominated bank |
| **MT740/742** | Reimbursement authorization / claim | Bank-to-bank reimbursement under URR 725 |
| **MT760** | Guarantee / standby | The bond message (URDG 758 / ISP98 world) |
| **MT400 / MT422** | Collections advice | Documentary collection settlement (URC 522 world) |

---

# 6. The Risks

Trade finance looks like lending, but its risk profile is a blend of credit risk, documentary risk, financial-crime risk, and operational risk — and the documentary layer is where trade finance is unique.

## 6.1 Credit and Country Risk

The bank's credit exposure in trade is *transaction-anchored*: the LC exposure is a contingent liability of the applicant until documents are presented; on presentation it converts to a funded exposure until the applicant pays; a guarantee converts only if called. Key features:

- **Contingent → funded conversion**: the limits treatment (see `banking_limits_domain_guide.md`) must book the LC/guarantee as a contingent limit draw from issuance, then convert on honouring.
- **Country and bank risk**: for confirming banks, the exposure is to the *issuing bank* and its jurisdiction — which is why confirmation lines carry country limits and why the ADB/IFC risk-sharing programs exist.
- **Self-liquidating character**: the trade itself is the repayment source — the goods being sold. That self-liquidating logic is exactly what fraud attacks (Section 6.2) and what TBML exploits (Section 6.3).
- **Concentration**: commodity trade books can concentrate risk on a handful of traders and corridors; portfolio limits matter (see `banking_limits_domain_guide.md` for the limits mechanics).

## 6.2 Documentary Fraud

**Verified as a recognized risk class** in trade finance practice and literature: documentary fraud is the deliberate presentation of documents that are false, forged, or misleading, to extract payment under an LC or collection. Because banks deal in documents (UCP 600 Article 5), a compliant-looking document set defeats the process.

Typical vectors:

- **Phantom shipments** — documents for goods that were never shipped (invoices, B/Ls, insurance certs all fabricated).
- **Forged/false bills of lading** — the title document is the crown jewel; false on-board B/Ls and warehouse receipts are the classic instruments.
- **Duplicate financing / double pledging** — the same goods (often stored commodities) pledged to multiple banks; the Qingdao port metals scandal of 2014 (duplicate warehouse receipts for the same copper/aluminum piles — **flagged**: press-reported) is the canonical modern example.
- **Misdescription** — goods described in the documents that don't match what was shipped (grade, quantity, origin).
- **Sham intermediaries** — shell companies inserted to launder the paper trail.

Why the bank is vulnerable despite the rules: UCP 600 Articles 34–37 *disclaim* bank liability for the genuineness of documents, the acts of third parties, and transmission — the rules assume honest documents. The fraud exception in law (the beneficiary's fraud defeats the credit — established in English law since *Sztejn v Henry Schroder* (1941), applied in Singapore and common-law jurisdictions; the *United City Merchants* (1983) case is the leading authority) lets banks/beneficiaries stop payment only on proven fraud, and courts grant injunctions reluctantly. The bank's practical defenses are **pre-issuance due diligence** (who is the counterparty, does the trade make sense, is the price sane) and **document examination discipline** (ISBP 745) — not post-facto lawyering.

## 6.3 AML and Sanctions: Trade-Based Money Laundering

**Verified as a recognized risk class** (FATF defines trade-based money laundering (TBML) as one of the principal money-laundering methods; the Wolfsberg Group publishes Trade Finance Principles; banks universally screen trade transactions): TBML is laundering through the trade system itself, exploiting the same documentary opacity that legitimate trade needs.

Primary typologies (all **verified** against FATF/Wolfsberg/Treasury descriptions):

- **Over-invoicing / under-invoicing** — mispricing to move value: over-invoice to pay excess to the exporter; under-invoice to transfer value to the importer.
- **Phantom shipments / phantom trade** — invoices for goods never shipped.
- **Multiple invoicing** — the same goods invoiced repeatedly to different buyers.
- **Misdescribed goods** — wrong commodity codes, wrong grade, to evade duties or sanctions.
- **Dual-use and sanctioned goods** — goods with military/civilian uses, or goods to/from sanctioned jurisdictions, routed via transshipment hubs.

The bank's control stack (standard practice — **flagged** as practice, not a single mandated standard):

1. **KYC/EDD on parties** — applicant, beneficiary, and the transaction's economic substance.
2. **Sanctions screening** — parties, vessels, ports, goods, jurisdictions against OFAC/EU/UN and local lists, on issuance *and* at presentation (vessels change).
3. **Transaction monitoring** — price/volume sanity checks against market benchmarks for the commodity (the classic red flag: invoice price far from market).
4. **Document consistency checks** — invoice vs B/L vs customs data (this is where digitization and data-sharing platforms, Section 7, add real AML value).
5. **Regulatory reporting** — suspicious transaction reports; FATF red-flag indicators inform case investigation.

The tension to design for: TBML controls fight the *opacity* of paper documents, so AML is one of the strongest business cases for trade digitization — but also one reason banks hesitate to share data on shared platforms (competition + data-protection law).

## 6.4 Operational Risk: Discrepancies

The operational heart of LC processing is **document examination**, and the operational reality is that a large share of first presentations are discrepant — industry sources commonly cite **60–70% of first presentations containing discrepancies** (**flagged**: widely cited industry figure, not an audited statistic). Each discrepancy is a fork in the road: waiver request to the issuing bank (MT750), amendment, or rejection (UCP 600 Article 16 — and a bank that fails to give a single notice loses the right to claim discrepancies).

The cost of discrepancy-driven rework is the single biggest operational pain point in trade — and the clearest target for automation (Section 7) and for good systems (Section 8).

## 6.5 Risk Comparison

| Risk | Vector | Mitigation |
|---|---|---|
| **Credit risk** | Applicant default on funded/unfunded exposure; issuing-bank default for confirmations | Limits and contingent-limit booking (`banking_limits_domain_guide.md`), self-liquidating structures, risk participation |
| **Country risk** | Issuing country transfer/convertibility risk; sanctions on jurisdiction | Country limits, confirmation pricing, risk-sharing programs (ADB/IFC), only-against-issuance policies |
| **Documentary fraud** | Forged documents, phantom cargo, duplicate financing | Pre-issuance due diligence, examination discipline (ISBP 745), fraud-exception litigation as backstop |
| **TBML / AML** | Over/under-invoicing, phantom trade, dual-use goods | KYC/EDD, sanctions screening (parties, vessels, goods), price benchmarking, STR filing |
| **Sanctions** | Screened party/vessel appears at presentation; goods to/from sanctioned routes | Screening at issuance and presentation, vessel screening, list management |
| **Operational / discrepancies** | Non-compliant presentations, missed 5-day window, single-notice failure | Examination standards (ISBP 745), automation/STP, discrepancy dashboards |
| **FX risk** | Currency mismatch between credit currency and settlement currency | FX cover at booking; USD/CNY/RMB corridors matter for commodity trade |
| **Legal/enforceability** | UCP incorporation, governing law, local court attitudes to fraud injunctions | Jurisdiction clauses, legal opinions, ISBP/UCP incorporation text |

---

# 7. The Digitization

## 7.1 Why Trade Finance Is Still Paper-Heavy

Trade finance is the last paper-based corner of banking. The reason is structural: the instruments are built on **paper title documents** — the bill of lading as a document of title, signed originals, stamped certificates — and the legal systems that give those documents their magic (the bill of lading *is* the goods, for negotiability purposes) were built before computers. A negotiable paper document has a bearer; a PDF does not. Digitizing trade therefore requires *law* (legal recognition of electronic transferable records), not just software.

The legal foundation exists and is spreading:

- **MLETR** — the UNCITRAL Model Law on Electronic Transferable Records (2017): the template statute that gives e-records the same legal status as paper transferable documents.
- **UK Electronic Trade Documents Act 2023**, **Singapore Electronic Transactions Act (ETA) 2021 amendment**, **Bahrain, Abu Dhabi Global Market, France, Spain, Germany** (among others) — the jurisdictions that have enacted MLETR-style law (**flagged**: the exact list of enacting jurisdictions is growing; verify per jurisdiction).
- **eUCP** (electronic presentation under UCP 600) and **ICC Uniform Rules for Digital Trade Transactions (URDTT, 2021)** — the rule-layer adaptations (**URDTT verified as an ICC publication**).

Until recently, a bank doing a cross-border LC still had to handle physical originals for the title documents — which is why the 2020s' legal wave is the real digitization catalyst, not any single platform.

## 7.2 The Platform Generation: Bolero, essDOCS, Contour

The pre-blockchain digitization wave produced three anchor platforms, all **verified as operating** (status as of this guide):

**Bolero** — the oldest eBL platform (originated in the late 1990s with SWIFT and TT Club involvement). Operates a rulebook (Bolero Rulebook) and a title registry that makes its electronic bills of lading functionally equivalent to paper for title-transfer purposes. Buyers, sellers, banks, and carriers exchange title documents on the platform. Status: **operating**; a niche but persistent player in eBL and e-document exchange.

**essDOCS** — founded 2005, built around **CargoDocs**, the eBL/e-document platform for commodity and liner trades, plus bank-side document workflows. In 2022 essDOCS was acquired by Intercontinental Exchange (**ICE**), becoming **ICE Digital Trade** — **verified via the FIT Alliance/industry references** (the platform is now branded ICE CargoDocs). Status: **operating**, and the most widely cited eBL platform in commodity trade.

**Contour** — the bank-backed blockchain LC network, launched from the **Voltron** pilot (2018–2020) and spun up as Contour in 2020 with founding banks including HSBC, Standard Chartered, BNP Paribas, ING, and others. Built on R3 Corda (**flagged**: platform stack per press reporting), it digitizes the LC lifecycle — issuance, amendment, presentation — as a shared, permissioned workflow among banks and corporates. **Verified operating**: press-reported deals continued into 2023–2024 (e.g., Rio Tinto–Baosteel RMB iron-ore LC on Contour; MUFG joining the network). Contour's status as of 2024–2025 is the notable uncertainty: its parent company went through restructuring, and the network's long-term trajectory is **flagged as uncertain** — verify current status before citing it as active in a pitch.

The pattern to observe: Bolero and essDOCS are *document* platforms (they digitize the title document); Contour is a *workflow* platform (it digitizes the bank-to-bank process). The failed blockchain era (7.3) tried to be both and neither.

## 7.3 The Blockchain Era: TradeLens, we.trade, Marco Polo

The 2016–2022 period produced a wave of DLT trade consortia. Three of the highest-profile have ended or collapsed — and the pattern of *how* they failed is the most instructive material in trade digitization.

**TradeLens** — **Verified**: the Maersk–IBM joint venture (Maersk's GTD Solution division + IBM), announced 2018 as a blockchain-enabled global shipping/trade platform (shipping data, customs, documents, built on Hyperledger Fabric — **flagged**: stack per press reporting). On **29 November 2022** Maersk and IBM announced discontinuation; the platform went offline by the **end of Q1 2023**. The stated reason: failure to reach commercial viability — "TradeLens was founded on the bold vision to make a leap in global supply chain digitization... however, we have not succeeded in achieving the level of commercial viability necessary to continue work and meet the financial expectations as an independent business" (Maersk announcement, **verified**). Despite impressive reach (hundreds of organizations, millions of events), it could not convert participation into revenue — the classic consortium problem: plenty of parties, no one willing to pay.

**we.trade** — **Verified**: the European bank consortium (Deutsche Bank, HSBC, Santander, Société Générale, UniCredit, and others; launched 2017, IBM Hyperledger Fabric stack — **flagged**) for SME trade finance. The consortium announced **discontinuation of activities in May 2022** (per S&P Global Market Intelligence reporting; an attempted restart under a new operator did not materialize — **flagged**).

**Marco Polo** — **flagged**: the R3 Corda-based trade finance network (banks incl. BNP Paribas, Commerzbank, ING, Standard Chartered, NatWest, and others), which pivoted from an LC-replacement to a "trade finance operating system" model. Trade press reported that the network's holding company entered **insolvency proceedings in Ireland in 2023** after a Bank of America investment fell through, with cumulative losses of USD 85 million reported for 2021 (GTReview/Ledger Insights reporting — **verified as press-reported, flagged as not independently audited**).

## 7.4 What the Shutdowns Taught Us

The post-mortem of the DLT era is consistent across all three failures:

1. **Network effects never reached critical mass** — trade digitization is a chicken-and-egg game: a platform is worthless with 3 of the 20 banks in a corridor. TradeLens had users but not *paying* users; we.trade and Marco Polo had founders but not liquidity.
2. **Consortium governance is hard** — competing banks on one shared ledger means agreeing on who owns the data, who pays, and who the operator is. Neutral operators (like SWIFT) work; operator-banks (Maersk/IBM) alarm the other participants.
3. **The document, not the ledger, is the bottleneck** — the hard problem was never consensus technology; it was legal recognition of e-titles and banks' willingness to accept them. The ledger was the easy 10%.
4. **Regulation and data protection** — shared trade data collides with bank-secrecy, GDPR, and AML data-sharing rules.
5. **The survivors are the ones with a narrow, revenue-generating job** — Bolero/essDOCS charge per document; Contour charges per transaction. Platforms that monetize a *specific* pain (eBL issuance, LC digitization) survive; platforms that monetize "industry transformation" do not.

The current generation has learned this: **digital-trade-hub** style initiatives (e.g., SWIFT's and ICC's trade digitization workstreams, MLETR adoption drives, eBL standards from DCSA and the FIT Alliance) are narrower, standards-first, and legal-regime-first. The contrast with the DeFi world is instructive — see `../technology/defi_guide.md`, which explores programmable finance; trade finance needs *legally enforceable* documents, not just *programmatically enforceable* ones.

## 7.5 Digitization Comparison

| Platform | Status | Notes |
|---|---|---|
| **Bolero** | Operating | eBL/e-document platform since ~1999; rulebook + title registry; SWIFT/TT Club origins |
| **essDOCS → ICE Digital Trade** | Operating | CargoDocs eBL for commodity/liner trade; acquired by ICE (2022), now ICE CargoDocs |
| **Contour** (ex-Voltron) | Operating, trajectory **flagged/uncertain** | Bank-backed DLT LC network (R3 Corda); live deals through 2023–24 (Rio Tinto–Baosteel RMB; MUFG joining); parent restructuring reported — verify before citing |
| **TradeLens** (Maersk–IBM) | **Shut down Q1 2023** | Announced discontinuation 29 Nov 2022; "not commercially viable"; the canonical consortium-failure case study |
| **we.trade** | **Discontinued May 2022** | European bank consortium for SME trade; IBM Fabric stack; restart attempt did not materialize |
| **Marco Polo** | **Insolvency proceedings (2023, press-reported)** | R3 Corda trade network; BoA investment fell through; ~USD 85M losses reported 2021 |
| **MLETR / ETDA-style legal regimes** | Active and spreading | The legal substrate that makes eBLs work; UK 2023, Singapore ETA 2021, Bahrain, ADGM, etc. — **flag**: jurisdiction list evolves |

---

# 8. The Systems

## 8.1 The Bank Trade Systems Landscape

Trade finance in a bank runs on a dedicated processing stack, because the workflow (document examination, contingent limits, SWIFT messaging, guarantee lifecycle) does not fit a vanilla core-banking loan module. The stack has three layers:

1. **Client channel / front office** — trade portals and APIs where the corporate applies for LCs, submits documents digitally, tracks status (often integrated with the ERP).
2. **Processing / middle office** — the trade operations platform: LC and guarantee origination, document checking (increasingly automated), discrepancy management, AML/sanctions screening hooks, SWIFT message generation and parsing.
3. **Back office / core integration** — bookings on the core (Temenos, Oracle Flexcube — see `core_banking_systems_guide.md`, `temenos_guide.md`, `oracle_flexcube_data_model_guide.md`), contingent-limit draws against the limits domain (`banking_limits_domain_guide.md`), accounting, and reporting. Settlement runs through the payments stack (`payments_hub_guide.md`, and the cross-border mechanics in `../technology/late_arriving_data_guide.md`).

Architecturally, the trade platform is the *orchestrator*: it owns the trade document of record (the credit, the guarantee, the collection), and calls out to the core for limits and accounting, to SWIFT (via a messaging gateway like SWIFT Alliance or a vendor's own) for MT700/707/760 traffic, and to screening engines for compliance. This matches the process view in `end_to_end_banking_processes.md` and the product-factory view in `universal_banking_model_guide.md`.

## 8.2 Vendors

**Flagged**: the vendor landscape below is **verified as the commonly cited set of trade-systems vendors** (per industry sources), but market-share figures are **not verified** — treat rankings and percentages as directional only.

- **Finastra** — *Trade Innovation* (the former Misys Trade Innovation; the platform branded **TI Plus**) — the most widely cited bank trade processing suite: LC origination, document checking, SWIFT messaging, guarantee management, with API/digital-channel layers.
- **Surecomp** — *DOKO*, *iTrade*, *TradeVision* — trade finance software for banks and corporates, SWIFT-certified; strong in document-centric workflows and cloud delivery.
- **CGI** — *Trade360* — the CGI trade and supply-chain finance suite, common in mid/large banks.
- **Intellect Design Arena** — trade products (e.g., eTrade/transaction banking suite) — **flagged**: exact current product names vary.
- **IBS (International Banking Systems)** — long-standing trade and treasury systems provider — **flagged**: presence/activity should be verified for current deals.
- **Edge Tech Systems** — trade finance and treasury ops systems.
- **Core-banking trade modules** — Temenos (Trade/TT modules), Oracle Flexcube (trade module), FIS, Silverlake — the trade capability embedded in the core for smaller/retail-commercial banks.
- **Specialists at the edges** — eBL platforms (Bolero, ICE Digital Trade — Section 7), KYC/AML screening engines, and SWIFT connectivity (Alliance, or vendor-native).

The buying decision pattern for a CACIB-style bank: the core is not the trade system; the trade platform (Finastra/Surecomp/CGI-class) is the system of record for trade products, and it must integrate to core, SWIFT, limits, screening, and the client channel — plus, increasingly, to external digital-trade networks (Contour, ICE Digital Trade, bank APIs).

## 8.3 Systems Comparison

| Layer | Representative vendors | Notes |
|---|---|---|
| **Trade processing platform** | Finastra (Trade Innovation / TI Plus), Surecomp (DOKO, iTrade, TradeVision), CGI (Trade360), Intellect, IBS, Edge Tech | System of record for LCs/guarantees/collections; document checking, SWIFT messaging, lifecycle **— share figures flagged** |
| **Core banking (trade modules)** | Temenos, Oracle Flexcube, FIS, Silverlake | Booking, limits, accounting; see `core_banking_systems_guide.md`, `temenos_guide.md`, `oracle_flexcube_data_model_guide.md` |
| **SWIFT connectivity** | SWIFT Alliance, vendor-native gateways | MT700 family in/out; MX migration (see `iso_20022_core_processes_guide.md`) |
| **Screening / AML** | Sanctions list management, transaction monitoring engines | TBML analytics; screening at issuance and presentation |
| **Digital-trade integration** | Bolero, ICE Digital Trade, Contour, bank APIs | eBL and network connectivity — Section 7 |
| **Client channel** | Trade portals, API/ERP integration | Corporate self-service LC applications, document upload |

---

# 9. Worked Example: A Commodity LC Through a CACIB-Style Bank

## 9.1 The Scenario

A Singapore-based commodity trading company ("**SG Trader Pte Ltd**") buys **25,000 MT of Brazilian soybeans** from "**AgroExport SA**" in São Paulo. The sale contract: **FOB Santos**, price **USD 500/MT**, total **USD 12.5 million**, shipment window **15–30 September**, payment by **irrevocable confirmed sight LC**.

The bank: **Crédit Agricole CIB, Singapore** (the CACIB-style issuing bank — familiar territory: commodity trade finance is a CACIB strength, and Singapore is the regional commodity hub). AgroExport insists on confirmation — standard for Brazil-bound soybean deals where the exporter wants the issuing-country risk removed.

**Participants**: Applicant (SG Trader), Issuing bank (CACIB Singapore), Advising **and** confirming bank (a São Paulo bank, "Banco do Brasil-style local player" — fictional stand-in), Beneficiary (AgroExport). Governing rules: UCP 600 + ISBP 745; Incoterms 2020 (FOB Santos); SWIFT for all messaging.

## 9.2 The Full LC Flow

| # | Day | Step | Party | Message/System action | Details |
|---|---|---|---|---|---|
| 1 | T-30 | Sale contract | SG Trader, AgroExport | — | FOB Santos, 25,000 MT, USD 12.5M, shipment 15–30 Sep, LC terms agreed |
| 2 | T-20 | LC application | SG Trader → CACIB SG | Trade portal / TI-Plus-class platform | Application lists: amount USD 12.5M ±10% (field 39A), FOB Santos (44E), latest shipment 30 Sep (44C), expiry 21 Oct at confirming bank's counters (31D), documents required: commercial invoice, full set clean on-board B/L "freight collect", certificate of origin, phytosanitary certificate (46A), 21-day presentation period (48), confirmation requested (49) |
| 3 | T-18 | Credit & limits | CACIB SG | Limits domain (`banking_limits_domain_guide.md`); screening | Facility check: SG Trader's trade line; **contingent limit booked** for USD 12.5M; KYC + sanctions screening of both parties, Brazil, soybeans (no dual-use issue), vessels TBD at presentation |
| 4 | T-17 | Issuance | CACIB SG | **MT700** via SWIFT to São Paulo bank | Fields 20/27/31C/31D/32B/39A/40A/41A/42C/43P/43T/44C/44E/45A/46A/47A/48/49/50/59/71A/78; confirmation instruction "confirm" (49); fees: all charges outside SG for beneficiary account (71A) |
| 5 | T-15 | Advice + confirmation | São Paulo bank | MT730 acknowledgement; MT710-style advice to beneficiary | Bank authenticates (Art 9), confirms (Art 8) at its confirmation margin (indicative: 0.25–0.50% p.a. on the LC value — **flagged as indicative pricing**), advises AgroExport |
| 6 | T-5 | Shipment preparation | AgroExport | — | Schedules vessel for the 15–30 Sep window |
| 7 | 18 Sep | Shipment | AgroExport, carrier | B/L issued | 25,000 MT loaded on board at Santos; clean on-board B/L dated 18 Sep; freight collect (FOB = buyer pays freight) |
| 8 | T+2 | Presentation | AgroExport → São Paulo bank | Paper or digital (eUCP/CargoDocs-style) | Invoice (USD 12.5M, FOB Santos), full set B/L, cert of origin, phytosanitary cert; presented 2 days after shipment — inside the 21-day window |
| 9 | T+4 | Examination | São Paulo bank | Document checking system | Checked against credit on face (Art 14): B/L on-board, dated 18 Sep within 44C, freight collect per FOB, quantity 25,000 MT within 39A tolerance, invoice matches 45A; **5-banking-day clock running** (Art 14(b)) |
| 10 | T+7 | Honour | São Paulo bank | MT754 (advice of payment) | Complying presentation → pays AgroExport USD 12.5M at sight (Art 8); claims reimbursement from CACIB SG |
| 11 | T+8 | Reimbursement | CACIB SG | MT752 authorization + MT202/MT103 settlement | CACIB reimburses the São Paulo bank; debits SG Trader's account per the credit terms (T+8 credit terms if usance; here sight) |
| 12 | T+9 | Document release | CACIB SG → SG Trader | Trade platform release | Documents released against payment; SG Trader endorses B/L to its freight forwarder, takes delivery at Singapore port; **contingent limit released** |

**What the bank systems did, in architecture terms**: the trade platform originated the LC (step 2), generated the MT700 (step 4), ingested the MT730/MT754 (steps 5, 10), tracked the contingent limit draw and its conversion to funded exposure (steps 3, 11), screened the presentation (steps 3, 8), and handed settlement to the payments stack (step 11). The whole lifecycle is the process view of `end_to_end_banking_processes.md`, with the limits mechanics of `banking_limits_domain_guide.md` and the cross-border settlement of `../technology/late_arriving_data_guide.md`.

**The discrepancy branch** (what happens when it does not go clean): if the B/L had said "freight prepaid" (a classic FOB error), the São Paulo bank would issue **MT750** (advice of discrepancy) to CACIB SG, which would ask SG Trader for a waiver. If waived, payment proceeds; if not, the presentation is rejected with the single-notice requirement (Art 16). Industry practice: ~60–70% of first presentations carry at least one discrepancy (**flagged**: commonly cited figure, not audited) — the discrepancy workflow is the operational cost center of the LC product.

## 9.3 The Lessons

1. **The LC is a risk-allocation machine, not a payment machine.** CACIB SG's credit stands behind payment; the confirming bank's credit stands behind the issuing bank's; FOB put freight on the buyer and insurance on the buyer — and every one of those allocations showed up as a document requirement. Get the Incoterms wrong and the documents don't match (the "freight prepaid" trap).
2. **Limits are booked at issuance, not at payment.** The USD 12.5M contingent draw existed from step 3; it converted to funded exposure only at step 11. Trade exposure management is lifecycle exposure management — see `banking_limits_domain_guide.md`.
3. **The rules are the contract.** UCP 600's 5-banking-day clock, single-notice rule, and face-of-document standard decide who eats a discrepancy. The bank's systems must encode the rule timers, not approximate them.
4. **Every step is a message.** The MT700 family is the workflow's nervous system; message mismatches (field 44C vs B/L date, 71A charges) are where operational risk lives.
5. **Digitization attacks the paper steps.** The B/L endorsement (step 12) is the last paper step; eBL platforms (Bolero/ICE CargoDocs) and MLETR-style law are what move it to digital — the worked example runs identically under eUCP.
6. **Screening is a lifecycle activity.** The vessel was only known at shipment; sanctions screening must re-run at presentation, not just at issuance.

---

# 10. Summary: One Page

**Trade finance** is the financing and risk-mitigation machinery of international trade: payment instruments (LC, documentary collections, open account), guarantees (bonds), working-capital finance, and the rules and messaging that make them work across borders. It solves two problems — the trust gap between seller and buyer, and the timing gap while goods are in transit.

**The landscape in one page:**

| Dimension | State of play |
|---|---|
| **Instruments** | LC (bank undertaking), collections (bank agency), open account (no bank), guarantees/bonds (bank contingent liability) |
| **Rules** | UCP 600 (2007), URC 522 (1995), URDG 758 (2010), Incoterms 2020 — the ICC rule stack is global and stable |
| **Messaging** | SWIFT MT700 family for LCs, MT760 for guarantees, MT400/422 for collections — the operational nervous system |
| **Risk** | Credit/country risk plus the trade-specific risks: documentary fraud, TBML/sanctions, discrepancy-driven operational loss |
| **The gap** | ~USD 2.5 trillion unmet demand (2022, ADB survey — flagged estimate); concentrated in SMEs and frontier markets |
| **Digitization** | Platforms survive by monetizing narrow jobs (eBL: Bolero, ICE CargoDocs; workflow: Contour); consortia failed on network effects (TradeLens, we.trade, Marco Polo); the legal substrate (MLETR/ETDA) is now the growth driver |
| **Systems** | Dedicated trade platforms (Finastra, Surecomp, CGI — flagged share), core trade modules, SWIFT gateways, screening engines, digital-network integration |
| **For the architect** | Trade is a lifecycle product: contingent limits at issuance, funded at honouring; rules as code (timers, notices); messages as the workflow; digitization as the roadmap |

**The final word: the paper-based world's digital future.** Trade finance is the last great paper-based world in banking — its instruments were perfected in the 19th century, its rules in the 20th, and its documents still travel by courier. But the direction is unambiguous: the legal regime (MLETR, the UK's Electronic Trade Documents Act, Singapore's ETA) has finally given electronic documents the same standing as paper; the platforms that failed taught the industry that digitization is a *monetizable narrow job*, not a consortium vision; and the systems now on the market can process a complying presentation in hours, not weeks. The digital future of the paper-based world is not the abolition of the letter of credit — it is the letter of credit without the paper: the same rules, the same undertakings, the same documents — presented, examined, and settled as electronic records. The bank that builds its trade stack for that future — rules as code, messages as workflow, documents as data — is the bank that will serve the trade finance gap that the paper world cannot.

---

# 11. Glossary

| Term | Definition |
|---|---|
| **Trade finance** | Financing and risk-mitigation instruments supporting international trade: payment methods, working-capital finance, and guarantees |
| **Letter of credit (LC)** | A bank's irrevocable undertaking to pay the beneficiary against a complying presentation of documents (UCP 600) |
| **LC** | Abbreviation for letter of credit; also "documentary credit" |
| **Documentary collection** | Settlement where banks transmit shipping documents for release against payment (D/P) or acceptance (D/A); banks act as agents (URC 522) |
| **Open account** | Trade settled on the buyer's promise: seller ships and invoices, buyer pays later; no bank instrument |
| **Guarantee** | A bank's undertaking to pay the beneficiary if the principal fails to perform (URDG 758) |
| **Bond** | Common trade term for a guarantee (bid, performance, advance-payment, retention) |
| **Applicant** | The buyer who requests an LC from its bank |
| **Beneficiary** | The seller in whose favour an LC/guarantee is issued |
| **Issuing bank** | The applicant's bank; gives the definite undertaking to honour complying presentations |
| **Advising bank** | The bank that authenticates and advises the credit to the beneficiary; no payment obligation |
| **Confirming bank** | A bank that adds its own undertaking to the credit, identical to the issuing bank's |
| **UCP 600** | ICC Uniform Customs and Practice for Documentary Credits, 2007 revision (Pub 600) |
| **URC 522** | ICC Uniform Rules for Collections, 1995 revision (Pub 522) |
| **URDG 758** | ICC Uniform Rules for Demand Guarantees, 2010 revision (Pub 758) |
| **Incoterms** | ICC delivery terms (Incoterms 2020: 11 terms) allocating risk/cost between buyer and seller |
| **SWIFT** | The interbank messaging cooperative (FIN network, BIC directory) carrying trade and payment messages |
| **MT700** | SWIFT message "Issue of a Documentary Credit" — the operative LC message |
| **Fraud (documentary)** | Presentation of false/forged documents to extract payment; mitigated by diligence and examination discipline |
| **AML** | Anti-money laundering — the control regime against laundering through the financial system |
| **Sanctions** | Trade/financial restrictions against parties, goods, or jurisdictions (OFAC, EU, UN) |
| **TBML** | Trade-based money laundering — laundering via invoice/price/document manipulation (over/under-invoicing, phantom shipments) |
| **Bolero** | eBL/e-document platform (operating) for electronic bills of lading and trade documents |
| **essDOCS (ICE Digital Trade)** | CargoDocs eBL platform; acquired by ICE (2022), now ICE CargoDocs |
| **Contour** | Bank-backed DLT LC network (ex-Voltron, R3 Corda); status flagged as uncertain |
| **TradeLens** | Maersk–IBM blockchain trade platform; shut down Q1 2023 |
| **we.trade** | European bank blockchain trade consortium; discontinued May 2022 |
| **Marco Polo** | R3 Corda trade finance network; insolvency proceedings reported 2023 |
| **Digitization** | Replacing paper trade documents/processes with electronic records and automated workflows |
| **Blockchain / DLT** | Distributed ledger technology used for shared trade workflows; the failed-consortium era 2016–2022 |
| **Trade finance gap** | Unmet demand for trade finance (ADB survey: ~USD 1.7T in 2020, ~USD 2.5T in 2022 — flagged estimates) |

---

# 12. Related Guides and Sources

## Related Guides

**Banking (this series)** — plain filenames:
- `end_to_end_banking_processes.md` — the end-to-end process view that trade products instantiate
- `universal_banking_model_guide.md` — where trade finance sits in the product factory
- `banking_limits_domain_guide.md` — the contingent-vs-funded exposure mechanics (LCs and guarantees are the canonical contingent products)
- `core_banking_systems_guide.md`, `temenos_guide.md`, `oracle_flexcube_data_model_guide.md` — core systems with trade modules
- `murex_mx3_platform_guide.md` — the treasury platform; trade-adjacent for commodity-hedging flows
- `supply_chain_finance_guide.md`, `supply_chain_finance_technologies_guide.md` — the adjacent receivables/payables product family
- `iso_20022_core_processes_guide.md`, `payments_hub_guide.md` — the settlement rails trade uses
- `financial_risk_compliance_systems_guide.md` — screening/AML systems
- `dbs_software_systems_guide.md`, `ocbc_software_systems_guide.md` — bank trade operations in practice (Singapore peers)
- `insurance_open_source_commercial_guide.md` — trade credit insurance adjacency
- `core_banking_processes_guide.md` — process patterns

**Technology** — `../technology/` prefix:
- `../technology/late_arriving_data_guide.md` — cross-border payment settlement (the P-7731/MT103 example) — the reimbursement step in Section 9
- `../technology/distributed_auth_guide.md` — digital identity for trade (KYC/e-signature substrate)
- `../technology/message_queue_data_loss_guide.md` — the payment pipeline messaging infrastructure
- `../technology/defi_guide.md` — the programmable-finance contrast: trade needs legally enforceable documents
- `../technology/ica_systems_guide.md` — customs systems, the logistics-side counterpart

## Sources and Verification Notes

**Verified facts** (via targeted web research, August 2026):
- UCP 600: ICC 2007 Revision, Pub 600, approved by ICC Banking Commission 25 Oct 2006, effective 1 Jul 2007 (ICC/Wikipedia/industry sources).
- URC 522: ICC 1995 Revision, Pub 522, adopted June 1995 (ICC Digital Library).
- URDG 758: ICC 2010 Revision, Pub 758 (ICC Digital Library/ICC text).
- Incoterms 2020: ICC Pub 723, 11 terms, published Sept 2019, effective 1 Jan 2020 (ICC/industry sources).
- MT700 = SWIFT "Issue of a Documentary Credit"; field 20 (credit number) mandatory; MT700 family (701/707/710/720/721/730/750/752/754/740/742/760/400/422) per SWIFT standards documentation.
- TradeLens: Maersk–IBM (GTD Solution), announced 2018, discontinuation announced 29 Nov 2022, offline by end Q1 2023, "not commercially viable" (Maersk announcement).
- we.trade: discontinuation of activities May 2022 (S&P Global Market Intelligence).
- Marco Polo: insolvency proceedings in Ireland reported 2023; BoA investment fell through; USD 85M cumulative losses reported 2021 (GTReview, Ledger Insights) — **press-reported, flagged**.
- Trade finance gap: ADB surveys — USD 1.5T (2018), USD 1.7T (2020), USD 2.5T (2022, +47%, record single-period increase); 2025 survey: gap remained ~USD 2.5T (ADB publications).
- TBML typologies (over/under-invoicing, phantom shipments, multiple invoicing, misdescribed goods): FATF red flags, Wolfsberg Trade Finance Principles, US Treasury descriptions.

**Flagged / not independently verified** (treat with care):
- "80–90% of world trade relies on trade finance" — widely quoted industry convention, not audited.
- USD 5.2 trillion global trade finance market (2022) — ICC estimate.
- ADB gap figures — survey estimates, not measured flows; SME rejection rates ~40–45% are survey-reported ranges.
- 60–70% first-presentation discrepancy rate — commonly cited industry figure, not audited.
- Contour's current operating status / parent restructuring — press-reported; trajectory uncertain; verify before citing as active.
- Marco Polo insolvency details — press-reported only.
- Bank trade-systems vendor market share (Finastra/Surecomp/CGI/Intellect/IBS/Edge Tech) — directional; no verified rankings.
- Confirmation-margin pricing (0.25–0.50% p.a. indicative) — indicative pricing only.
- MLETR enacting-jurisdiction list — evolving; verify per jurisdiction.
- eUCP version 2.1 (2019) and URDTT (2021) — ICC publications per ICC references.
