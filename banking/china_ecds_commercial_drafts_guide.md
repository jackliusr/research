# China's Electronic Commercial Draft System: The Accepted Draft

> **This guide is a deep-research reference on China's commercial draft (商业汇票) and the electronic infrastructure that carries it — the Electronic Commercial Draft System (ECDS, 电子商业汇票系统), the Shanghai Commercial Paper Exchange (上海票据交易所) that absorbed it, and the new-generation platform that replaced both.** It owns the *instrument and its market*: what a commercial draft legally is under the Negotiable Instruments Law of the PRC (票据法), how the two economic species — bank-accepted (银行承兑汇票) and commercial-accepted (商业承兑汇票) — differ in credit substance, the current maximum tenor and its change history, the lifecycle mechanics (出票/承兑/背书/贴现/质押/保证/提示付款/追索), the trading and money-market layer, the regulatory stack, credit and fraud risk, the supply-chain bill (供应链票据) and the standardised note (标准化票据), the technology a participating bank must build, and the banking business model around a bill desk. It is deliberately scoped *to China's own bill regime*: the Western bill of exchange used under a documentary credit — the draft fields 42C/42A/D of a UCP 600 letter of credit, forfaiting, acceptance finance under Anglo-American negotiable-instruments law — lives in the trade-finance guides, not here, and the two must not be conflated. The funds leg of every bill transaction settles through CNAPS (中国现代化支付系统), which is owned by the CNAPS guide; this guide cross-references it and does not re-derive it.
>
> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Chinese bill and draft market infrastructure — ECDS and its successor regime under the Shanghai Commercial Paper Exchange, the commercial draft instrument, system mechanics, the money-market layer, regulation, credit and fraud risk, supply-chain bills, banking architecture and practice  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Primary Sources:** People's Bank of China (PBOC, 中国人民银行) regulations, announcements and statistics (pbc.gov.cn), including 《商业汇票承兑、贴现与再贴现管理办法》 (中国人民银行 中国银行保险监督管理委员会令〔2022〕第4号), 《中国人民银行关于修改部分规章的决定》 (令〔2024〕第5号) and 《标准化票据管理办法》 (公告〔2020〕第6号); the Chinese government portal and State Council gazette (中国政府网, gov.cn); National Financial Regulatory Administration (国家金融监督管理总局, nfra.gov.cn); Shanghai Commercial Paper Exchange (上海票据交易所, shcpe.com.cn) notices and disclosure lists; the Negotiable Instruments Law of the PRC (中华人民共和国票据法) on npc.gov.cn; NPC Standing Committee annual legislative work plans; and Chinese financial press — 第一财经, 新浪财经, 上海证券报, 财新, 汇票圈 and bill-industry portals  
> **Last Updated:** 2026-09  
> **Companion guides:** [cnaps_guide.md](cnaps_guide.md) (owns CNAPS/HVPS and the funds leg), [supply_chain_finance_guide.md](supply_chain_finance_guide.md) (owns the SCF discipline), [supply_chain_finance_technologies_guide.md](supply_chain_finance_technologies_guide.md), [trade_finance_guide.md](trade_finance_guide.md) (owns UCP 600 and documentary credits), [trade_finance_systems_guide.md](trade_finance_systems_guide.md), [financial_infrastructure_guide.md](financial_infrastructure_guide.md), [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md), [core_banking_processes_guide.md](core_banking_processes_guide.md), [banking_limits_domain_guide.md](banking_limits_domain_guide.md), [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md), [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md)

---

## Table of Contents

1. [Overview and the Hand-off](#1-overview-and-the-hand-off)
   - 1.2 [The Hand-off from the CNAPS Guide](#12-the-hand-off-from-the-cnaps-guide) · 1.3 [What This Instrument Is Not](#13-what-this-instrument-is-not-the-western-documentary-draft) · 1.4 [Chinese-English Term Decoder](#14-chinese-english-term-decoder)
2. [The Instrument](#2-the-instrument)
   - 2.2 [Tenor: the Current Rule and Its Change History](#22-tenor-the-current-rule-and-its-change-history) · 2.4 [Draft Against Loan, Letter of Credit, Paper Note and Receivable](#24-draft-against-loan-letter-of-credit-paper-note-and-receivable)
3. [Institutional History](#3-institutional-history)
   - 3.2 [Build, Launch, Nationwide Rollout](#32-build-launch-nationwide-rollout) · 3.4 [The New-Generation System and the End of the ECDS](#34-the-new-generation-system-and-the-end-of-the-ecds)
4. [System Mechanics](#4-system-mechanics)
   - 4.1 [Participants and Access Modes](#41-participants-and-access-modes) · 4.4 [The Funds Leg and Delivery-versus-Payment](#44-the-funds-leg-and-delivery-versus-payment)
5. [The Trading and Money-Market Layer](#5-the-trading-and-money-market-layer)
   - 5.1 [贴现, 转贴现 and 再贴现](#51-贴现-转贴现-and-再贴现) · 5.3 [标准化票据: the Standardised Note](#53-标准化票据-the-standardised-note)
6. [Regulatory Framework](#6-regulatory-framework)
   - 6.1 [The Negotiable Instruments Law and Its Amendment Status](#61-the-negotiable-instruments-law-and-its-amendment-status) · 6.4 [Disclosure and the Overdue Lists](#64-disclosure-and-the-overdue-lists) · 6.6 [The 2025 Supply-Chain Notice](#66-the-2025-supply-chain-notice)
7. [Credit, Fraud and Risk](#7-credit-fraud-and-risk)
   - 7.2 [Documented Default Evidence](#72-documented-default-evidence) · 7.3 [Fraud Typologies and What Electronics Changed](#73-fraud-typologies-and-what-electronics-changed)
8. [The Supply-Chain Finance Angle](#8-the-supply-chain-finance-angle)
   - 8.2 [等分化 and What It Economically Enables](#82-等分化-and-what-it-economically-enables) · 8.5 [The 2025–2027 Rectification and the Shift Toward Bills](#85-the-20252027-rectification-and-the-shift-toward-bills)
9. [Technology and Integration](#9-technology-and-integration)
   - 9.1 [What a Bank Must Build](#91-what-a-bank-must-build) · 9.4 [Blockchain and Platform Initiatives](#94-blockchain-and-platform-initiatives)
10. [Banking Practice](#10-banking-practice)
    - 10.2 [Credit Limits, Pricing and Credit Quota](#102-credit-limits-pricing-and-credit-quota) · 10.3 [Accounting, Reporting and Margin-Thin Versus Fee-Rich](#103-accounting-reporting-and-margin-thin-versus-fee-rich)
11. [Market Context and Wider Significance](#11-market-context-and-wider-significance)
    - 11.2 [Bills in the Credit and Social-Financing Aggregates](#112-bills-in-the-credit-and-social-financing-aggregates) · 11.3 [SME Tool Versus Shadow-Financing Vehicle](#113-sme-tool-versus-shadow-financing-vehicle)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
    - 12.2 [Instrument Selection and the Credit Consequence](#122-instrument-selection-and-the-credit-consequence) · 12.4 [Illustrative Economics, Failure Modes and Recommendation](#124-illustrative-economics-failure-modes-and-recommendation)
13. [Claims Audit](#13-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [Cross-References and Further Reading](#16-cross-references-and-further-reading)
17. [Closing Summary](#17-closing-summary)

---

## 1. Overview and the Hand-off

### 1.1 What a Chinese Commercial Draft Is

A commercial draft (商业汇票) is a negotiable instrument drawn by a drawer (出票人) instructing a payer to pay a fixed sum unconditionally to a payee or holder (持票人) on sight or at a specified date. It is an instrument in the Chinese civil-law tradition, not a trade document attached to a shipment: there is no bill of lading behind it, no documentary collection, no compliance presentation. The instrument is the credit.

The defining legal feature is **acceptance (承兑)** — the payer's unconditional promise to pay at maturity. Acceptance is what converts a data message into bank-grade or corporate-grade money-market paper, and everything about how this instrument is priced, held and regulated follows from who signs that acceptance.

**Why it matters more than its reputation suggests.** Outside China the commercial draft is often treated as an obscure relic. That reading is wrong on scale. According to PBOC's 2025 financial-market statistics, commercial draft acceptance volume in 2025 was RMB 42.7 trillion and discounting volume was RMB 33.9 trillion, with acceptance balances at end-2025 of RMB 21.2 trillion, up 7.2% on end-2024. PBOC's own social-financing release put total social-financing stock at RMB 442.12 trillion at end-2025. A market whose annual creation approaches a tenth of the entire social-financing stock is not a niche.

The instrument does three jobs at once: a **deferred payment mechanism** for real trade, a **short-term financing channel** for firms that discount it, and a **money-market asset class** that banks trade against each other and against the central bank.

**The ECDS, in one paragraph.** Approved and built by the People's Bank of China, the ECDS (电子商业汇票系统) was the central registry and processing platform for electronic commercial drafts — the data message replacing the paper bill, the electronic signature replacing the physical seal. It ran from 2009 to 2024 and carried three functional modules: electronic draft business processing (the core), registration and enquiry for paper drafts, and a public quotation function for interbank discounting.

**The Exchange, in one paragraph.** Established with State Council agreement and PBOC approval, the Shanghai Commercial Paper Exchange (上海票据交易所) opened on 8 December 2016 as the national unified bill trading platform, providing bill trading, registration and custody, clearing and settlement, and information services. It is now the operator of the market's infrastructure, the publisher of acceptor-disclosure data, and the body to which data on the adjacent receivables-certificate market is reported. The relationship is the point: the ECDS was a *processing* system in the central bank's orbit; the Exchange is a *market* institution.

### 1.2 The Hand-off from the CNAPS Guide

The companion CNAPS guide draws the boundary. Quoted verbatim from its system map:

> | **Commercial-paper system — ECDS** | 电子商业汇票系统 | Separate system | Electronic commercial drafts/bills (电子商业汇票) issuance, acceptance, discounting and clearing; settlement of funds legs flows through CNAPS. Operated in the PBOC/上海票据交易所 (Shanghai Commercial Paper Exchange) orbit, not as a CNAPS application |

And from its glossary:

> | 电子商业汇票系统 | Electronic Commercial Draft System (ECDS) | Separate; operated in the Shanghai Commercial Paper Exchange orbit |

That is the whole hand-off in two rows: the ECDS was never a CNAPS application, but it *produced* obligations whose funds leg then settled inside CNAPS. This guide picks up where those sentences stop. See [cnaps_guide.md](cnaps_guide.md) for HVPS windows, queuing and settlement-account arrangements; this guide does not re-derive them.

### 1.3 What This Instrument Is Not: the Western Documentary Draft

This is the most common conflation in trade-finance documentation, and it is worth being blunt about.

| Dimension | Chinese commercial draft (商业汇票) | Western draft under a documentary credit |
| --- | --- | --- |
| Governing law | Negotiable Instruments Law of the PRC (票据法), civil-law tradition | Anglo-American negotiable-instruments law / local bills-of-exchange acts |
| Where it lives | Free-standing negotiable instrument; the credit *is* the acceptance | One document among many under a UCP 600 letter of credit |
| Typical UCP 600 fields | n/a | Field 42C (drafts at …), 42A (drawee), 42D (drawer) |
| Acceptance concept | 承兑 — payer's unconditional promise; a regulated act | Acceptance of a usance draft by the nominated/issuing bank |
| Discounting | 贴现 / 转贴现 / 再贴现 — a regulated market with a central venue | Forfaiting, bill discount, negotiation — bilateral, unregulated as a market |
| Funds leg | CNAPS, on a delivery-versus-payment basis on the trading side | Correspondent banking / RTGS, bank by bank |
| Central infrastructure | Shanghai Commercial Paper Exchange (上海票据交易所) | None; SWIFT messaging plus correspondent relationships |
| Acceptor credit disclosure | Mandatory regime (信息披露) with published overdue lists | Not applicable |

A trade-finance professional reading fields 42C/42A/D of an MT700 or MT798 is **not** looking at a Chinese commercial draft. The trade-finance guides own that instrument; nothing here describes it.

### 1.4 Chinese-English Term Decoder

| Chinese | English | One-line note |
| --- | --- | --- |
| 汇票 | draft / bill of exchange | The order-to-pay instrument; parent category of 商业汇票 |
| 本票 | promissory note | A promise-to-pay instrument; not this guide's subject |
| 支票 | cheque | Demand instrument drawn on a bank; separate regime |
| 商业汇票 | commercial draft | The bill used in real-economy settlement and short-term financing |
| 承兑 | acceptance | The payer's unconditional promise to pay at maturity; the credit event |
| 银行承兑汇票 | bank-accepted draft (银票) | Accepted by a bank or rural credit cooperative; bank credit |
| 商业承兑汇票 | commercial-accepted draft (商票) | Accepted by a non-bank legal person; corporate credit |
| 财务公司承兑汇票 | finance-company-accepted draft | Accepted by an enterprise-group finance company |
| 贴现 | discounting | Holder sells an unmatured draft to a licensed lender at a discount |
| 转贴现 | interbank rediscount | Bank-to-bank trading of already-discounted drafts |
| 再贴现 | central-bank rediscount | PBOC discounts eligible drafts; a monetary-policy tool |
| 背书 | endorsement | Transfer of rights by endorsement on the instrument |
| 质押 | pledge | Using a draft as collateral without transferring ownership |
| 保证 | guarantee | Third-party guarantee of the draft obligation |
| 提示付款 | presentation for payment | Holder demands payment at or after maturity |
| 追索权 | right of recourse | Claim against prior endorsers and the drawer on dishonour |
| 出票人 | drawer | Issues the draft |
| 承兑人 | acceptor | Promises to pay; the credit obligor |
| 持票人 | holder | Current owner of the instrument |
| 票据池 | bill pool | Bank product aggregating a client's bills for pledge/financing |
| 标准化票据 | standardised note | Beneficiary certificate over a pool of drafts; a money-market instrument |
| 供应链票据 | supply-chain bill | Electronic commercial draft issued via the Exchange's supply-chain platform |
| 电票 | e-bill | Colloquial for electronic commercial draft |
| 票据法 | Negotiable Instruments Law | The PRC statute governing the instrument |
| 票交所 | the Exchange | Shanghai Commercial Paper Exchange |
| 承兑余额 | acceptance balance | Outstanding accepted amount; a regulated metric |
| 贴现余额 | discounting balance | Outstanding discounted amount; a balance-sheet metric |
| 等分化 | equalisation / divisibility | Splitting a draft into units down to a standard minimum |
| 票款对付 | delivery-versus-payment (DVP) | Simultaneous exchange of the bill and the funds |
| 真实交易背景 | real trade background | The underlying-transaction requirement |
| 融资性票据 | financing-only draft | A draft with no real underlying trade; prohibited |

**One-page orientation.** Read the guide in four layers. **Layer one, the instrument:** a negotiable instrument with an acceptance at its heart, in two species — bank-accepted (bank credit) and commercial-accepted (corporate credit) — with the gap between those credits the most important fact in the market, and tenor capped at six months from issue to maturity under the 2022 joint measures. **Layer two, the infrastructure:** the ECDS built the electronic instrument, the Exchange built the market, the new-generation system merged both, and the ECDS stopped external service on 27 July 2024. **Layer three, the market:** discounting moves bills from enterprises to banks, interbank rediscounting between banks, central-bank rediscounting into PBOC, with the Exchange publishing rates and acceptor-disclosure lists. **Layer four, the risk:** the commercial-accepted species carries genuine corporate credit risk, the Exchange quantifies it in published overdue lists, and fraud has migrated from forged paper to forged trade background and intermediary conduct.

---

## 2. The Instrument

### 2.1 The Two Economic Species

The Negotiable Instruments Law of the PRC (中华人民共和国票据法) was adopted on 10 May 1995 by the Standing Committee of the Eighth National People's Congress, and amended by decision of the Standing Committee of the Tenth National People's Congress on 28 August 2004, taking effect on promulgation. Its draft structure is conventional: general provisions, then 汇票 in six sections — issue (出票), endorsement (背书), acceptance (承兑), guarantee (保证), payment (付款) and recourse (追索权) — then promissory notes, cheques, foreign-related instruments, liability and supplementary provisions. Two consequences follow: the instrument is *statutory*, not contractual, so the holder's claim against the acceptor runs on the instrument independently of the underlying trade contract; and the enforcement machinery is Chinese civil-law machinery, which is why the interplay between suing on the instrument and suing on the underlying relationship recurs in Chinese bill litigation.

| Species | Chinese | Acceptor | Credit | Typical spread |
| --- | --- | --- | --- | --- |
| Bank-accepted draft | 银行承兑汇票 (银票) | A PRC bank or rural credit cooperative licensed with bill acceptance in scope | Bank credit | Tight, near money-market |
| Finance-company-accepted draft | 财务公司承兑汇票 | An enterprise-group finance company, licensed with bill acceptance in scope | Quasi-corporate, tied to the group | Between the two |
| Commercial-accepted draft | 商业承兑汇票 (商票) | A non-bank legal person or unincorporated organisation in the PRC | Corporate credit | Wide, credit-sensitive |

The classification is legal, not commercial: a draft does not become bank-accepted because a bank guarantees it; the acceptor's identity classifies it. This matters because the whole regulatory apparatus — disclosure obligations, acceptance ceilings, the prohibition on institutions serving persistently-overdue acceptors — keys off that identity.

**The credit gap is the instrument.** Bank-accepted paper trades close to money-market rates because the acceptor is a licensed deposit-taking institution subject to capital, liquidity and acceptance-ceiling rules. Commercial-accepted paper trades at a spread reflecting the acceptor's own credit — or is unsaleable. The 2022 rulebook recognises the asymmetry directly (Articles 22 and 36): acceptors and discounters must be financially sound, and must not have experienced persistent bill overdue or non-disclosure in the preceding two years. So the market is, in one sentence: **the instrument is uniform, the credit is not, and the market is a spread ladder between the two ends.**

### 2.2 Tenor: the Current Rule and Its Change History

This is the fact class most often reported wrongly.

| Item | Value |
| --- | --- |
| Instrument | 《商业汇票承兑、贴现与再贴现管理办法》 |
| Issuing bodies | 中国人民银行 (PBOC), 中国银行保险监督管理委员会 (CBIRC) |
| Number | 中国人民银行 中国银行保险监督管理委员会令〔2022〕第4号 |
| Adopted | 24 August 2022 (PBOC 2022 第7次行务会议) |
| Signed | 11 November 2022 (行长 易纲; 银保监会主席 郭树清) |
| Published | 18 November 2022 |
| Effective | 1 January 2023 |
| Tenor article | **第二十五条** — the payment term must match the real transaction's performance period and, from issue date to maturity date, **must not exceed 6 months** |

The change history is the load-bearing part:

- **Before 2023**, the two forms had different ceilings. Paper drafts were capped at six months, while electronic drafts could run to **one year** under the ECDS-era 《电子商业汇票业务管理办法》 — a deliberate policy lever to promote electronic adoption.
- **From 1 January 2023**, 第二十五条 removed the distinction: six months from issue to maturity for all commercial drafts, paper and electronic alike. The effect was to compress the maximum financing period obtainable from a single draft, pushing more of the SME financing burden toward the divisible supply-chain bill and toward receivables instruments.
- **第四十二条** of the same order repealed the 1997 interim measures (《商业汇票承兑、贴现与再贴现管理暂行办法》, 银发〔1997〕216号文印发) and the 2001 PBOC notice (银发〔2001〕236号). Chinese financial press reported this as the first full revision of the 1997 regime in roughly a quarter of a century.

**What the instrument is.** 第二条 defines a commercial draft as a drawer-issued instrument instructing a payer to pay a fixed sum unconditionally to a payee or holder on sight or at a specified date, expressly including paper and electronic forms of 银行承兑汇票, 财务公司承兑汇票 and 商业承兑汇票. 第三条 requires that the issuance, acceptance, discounting, pre-discount endorsement, pledge, guarantee, presentation and recourse of an electronic commercial draft be handled **through a PBOC-recognised bill-market infrastructure**, and states explicitly that a supply-chain bill (供应链票据) **is** an electronic commercial draft — the quietest but most consequential line in the regulation, since it subjects the supply-chain instrument to the entire bill regime. 第四条 defines acceptance; 第五条 defines discounting and imposes the pivotal condition that the holder's acquisition be lawful and backed by a real transaction and debtor-creditor relationship, with a carve-out for value-free acquisition by taxation, inheritance or gift.

### 2.3 Endorsement, Joint Liability and Recourse

Endorsement (背书) transfers the instrument and carries joint liability: every endorser stands behind the instrument alongside the acceptor, so a dishonoured holder may pursue the acceptor, the drawer and any prior endorser. Article 29 adds a verification duty on the transferee side — on endorsement the endorsee **may** check instrument information against disclosure records and, where information is absent or inconsistent, may take effective measures to verify authenticity and credit risk. The permissive wording is deliberate; the corresponding discounting duty is mandatory (see 6.4).

Recourse (追索) is the remedy. Dishonour is recorded centrally when the presenter's demand is refused, and the holder then litigates either on the instrument or on the underlying legal relationship. Electrification of this step — particularly notification — is where the newest rule changes sit (see 7.4).

### 2.4 Draft Against Loan, Letter of Credit, Paper Note and Receivable

| Comparison | Commercial draft | Bank loan | Letter of credit | Trade receivable | Paper draft |
| --- | --- | --- | --- | --- | --- |
| Legal nature | Negotiable instrument | Contract | Undertaking under UCP 600 / ISP98 | Contract claim | Same, but physical |
| Transferability | By endorsement; rights travel with the instrument | Assignment, restricted | Not transferable as an instrument | Assignment | By delivery + endorsement |
| Credit enhancement | Acceptance by bank or corporation | Borrower's own credit | Issuing bank's undertaking | Usually none | Acceptance |
| Secondary market | Deep and regulated (Exchange) | Syndication / assignment | Forfaiting, bilateral | Factoring, SCF platforms | Historically none |
| Encoding | Central register record | Core-banking record | SWIFT message | Platform record | Paper |
| Settlement | DVP via CNAPS on the trading side | Account credit | Documentary presentation | Payment on due date | Bank clearing |

The clean distinction: a loan is a **balance-sheet relationship**; a draft is a **negotiable claim**. A bank that discounts a draft acquires an instrument with joint-liability recourse, which is legally different from advancing a loan even though both sit in the same accounting family.

**The credit-versus-instrument distinction.** Three things are separate and should be analysed separately. **Instrument risk** — whether the draft is valid, properly endorsed and enforceable: a legal and operational question that electronic processing largely solved. **Credit risk** — whether the acceptor, and on recourse the prior parties, will pay: untouched by electrification, and the real risk in this market. **Conduct and compliance risk** — whether the underlying trade was real, disclosure accurate, intermediaries proper: the regulatory focus, which has moved from paper fraud to document fraud. Electrification collapsed the first, left the second to the business cycle, and sharpened the third by making the paper trail auditable.

---

## 3. Institutional History

### 3.1 The Paper Era and Its Failure Modes

Before centralised electronic processing, commercial drafts moved physically. The consequences were predictable and severe: **forgery (伪造票据) and alteration (变造票据)**, controllable only by physical inspection; **custody risk**, with bills lost, stolen or substituted in transit; **duplicate financing**, since no central register recorded the instrument's current holder and the same paper could be presented to two banks; **branch-level risk**, because a head office could not see in real time what its branches had accepted or discounted; and **illiquidity**, since a paper bill could only be sold to whoever was physically reachable, so the interbank market was fragmented by geography and trust.

The bill scandals of the early 2010s were not one event but a class of failures with one root cause: an instrument with cash-like characteristics handled with paper-like controls. This guide does not name particular institutions or cases from that period, because it could not establish primary records for them (see 14).

The stated motivation for the ECDS was legible from the outset. Chinese authorities judged that commercial draft electronification had lagged the rest of the payments and instruments landscape: cheques had the national cheque-image exchange system from 2007, and bank promissory notes and certain regional bank drafts were cleared electronically from 2008, while commercial drafts remained hand-processed. The diagnosis was specific — individual banks had experimented with their own electronic bill systems from around 2005, but **the absence of a unified central registration mechanism and a cross-bank trading platform** confined those efforts to intra-bank business. The ECDS was therefore designed as a central registry, not merely a messaging layer. That design decision made everything downstream possible.

### 3.2 Build, Launch, Nationwide Rollout

| Date | Event |
| --- | --- |
| January 2008 | PBOC decides to organise construction of an electronic commercial draft system |
| June 2008 | Project formally established (正式立项) |
| 16 October 2009 | PBOC publishes 《电子商业汇票业务管理办法》 |
| 28 October 2009 | ECDS built and put into operation (建成投入运行) |
| 28 June 2010 | PBOC organises nationwide rollout (在全国推广应用) |
| 10 September 2010 | PBOC publishes 《电子商业汇票系统管理办法》, governing operator, participants, business processing and risk management |
| 8 December 2016 | Shanghai Commercial Paper Exchange opens (PBOC's 《票据交易管理办法》 issued two days earlier, designating it as the trading, custody, clearing and information-services institution) |
| 9 October 2017 | ECDS operations transferred to the Exchange |
| 3 June 2022 | New-Generation Bill Business System self-run go-live |
| 20 August 2022 | First batch of 51 member institutions (41 banks + 10 finance companies) go live with full-lifecycle functionality |
| 29 December 2023 | Exchange notice 票交所发〔2023〕82号 sets the ECDS migration plan |
| 27 July 2024 | ECDS stops external service |

The 2008–2010 dates rest on PBOC's own 2009 press background and on the ECDS encyclopedia entry citing it; the 2016–2017 rows on PBOC and press coverage of the handover; the 2022–2024 rows are sourced in 3.4 and 13.

**The mandatory-electronic thresholds.** The ECDS became universal by mandate rather than attraction, via an amount threshold above which commercial drafts had to be handled electronically: from **1 January 2017**, single-issue amounts above RMB 3 million had to go through the ECDS; from **1 January 2018** the threshold fell to above RMB 1 million. The effect was swift retirement of large-denomination paper, leaving paper only below the threshold and in residual form. These thresholds are reported from the encyclopedia entry's citation trail rather than primary texts read for this guide, and are flagged accordingly in 13.

### 3.3 Why a Central Venue Was Created

The ECDS solved **processing**: it gave the instrument a central register and a cross-bank platform. It did not solve **market structure** — bills were still traded bilaterally with opaque pricing, and the central bank could not see the market in aggregate.

The Exchange was the answer. Established with State Council agreement and PBOC approval, with registered capital of RMB 5 billion and shareholders including the PBOC Clearing Centre and the National Association of Financial Market Institutional Investors, it opened on 8 December 2016. Two days earlier, on 6 December 2016, PBOC had issued the 《票据交易管理办法》, which designated the Exchange as the institution providing bill trading, registration and custody, clearing and settlement, and information services.

The shift from a *processing utility* to a *trading venue with registration* is the structural move. Once trades are centrally registered and centrally settled, three things become possible that were not: transparent pricing, central-bank visibility, and the standardised instruments that need a central custodian to exist at all.

**A source nuance.** Some accounts describe 8 December 2016 as the Exchange's 成立 (establishment), others as its 开业 (opening); Wikipedia records the opening ceremony on that date, while a commercial Exchange-information site says it was 成立 on that date. The readings do not seriously conflict — a company can be incorporated and open on the same date, and PBOC's own account frames it as opening — and this guide treats 8 December 2016 as the operating start without asserting a separate incorporation date.

### 3.4 The New-Generation System and the End of the ECDS

The New-Generation Bill Business System (新一代票据业务系统) is the Exchange's successor platform, built to fuse the electronic commercial draft system and the China Bill Trading System (中国票据交易系统) into **one system with one set of interfaces**, covering the full lifecycle of paper and electronic drafts. Note a naming nuance: the Exchange's own migration notice calls the successor the **中国票据业务系统**, while policy and press usually call it the 新一代票据业务系统.

| Date | Event | Source |
| --- | --- | --- |
| 3 June 2022 | New-generation system itself goes live (投产上线) | Bill-industry press reporting the Exchange's revised engineering plan |
| 20–21 August 2022 | First batch of market members go live | Reuters-line press; Shanghai municipal financial office |
| 22 August – 30 September 2022 | Trial running (试运行) for that first batch | Bill-industry press |
| 20 August 2022 | Announced as the system's "successful launch" date; 51 first-batch institutions live with full-lifecycle functionality | Shanghai municipal financial office, reporting the Exchange's announcement |

**The conflicts, and how they resolve.** The parent reconnaissance flagged a discrepancy in the composition of the 51 first-batch institutions: one source said 41 banks plus 10 finance companies, another 23 commercial banks, 18 village banks and 10 finance companies. These are **the same thing** — 23 + 18 = 41 banks — so the discrepancy is a level-of-detail artefact, not a factual conflict. The apparent go-live conflict resolves the same way: the *system* was commissioned on 3 June 2022 and the *first member batch* went into production on 20 August 2022, with a supervised trial to end-September. Both dates are correct at their levels, and this guide reports them as two distinct events rather than picking one.

What the new system changed substantively: **one platform and one interface set** replacing two systems, removing a reconciliation boundary that had existed since 2016; **divisibility (等分化 / 票据拆分)**, letting issuers create "bill packages" of standard-amount units that holders can split across endorsement, discounting, guarantee and pledge (see 8.2); and a **unified presentation-for-payment workflow** across paper and electronic instruments, with a dishonour-notice machinery extended further in 2025 (see 6.4).

**The decommissioning.** Exchange notice 票交所发〔2023〕82号, dated 29 December 2023, set the migration plan: the ECDS would **stop external service on 27 July 2024**, and all of its unsettled, unvoid and unexpired bills would migrate to the successor system **in one batch**, with presentation-for-payment and recourse handled thereafter in the new system. Members had to elect direct-connect mode by 8 January 2024 or be defaulted into client mode. The ECDS therefore ran for roughly fifteen years, and the market's rulebook and platform both moved to the new stage simultaneously.

### 3.5 Regulatory Tightening Eras

The market's regulatory history alternates loosening (to promote adoption) and tightening (to contain what adoption enabled). Four eras are recognisable, each with a distinct instrument.

**Era 1 — construction and promotion (2008–2013).** Build the ECDS, publish the 2009 《电子商业汇票业务管理办法》, roll out nationally in 2010, then mandate electronic handling above thresholds. Positive incentives included the extended one-year tenor on electronic drafts.

**Era 2 — post-scandal tightening (2013–2016).** Fraud losses in the paper and semi-paper market triggered tightened branch-level controls and the recognition that bilateral trading left no audit trail. The response was institutional: create a central venue.

**Era 3 — centralisation and standardisation (2016–2022).** The Exchange opens (2016) and assumes ECDS operations (2017); the standardised-note regime is created (2020); the mandatory-disclosure regime for commercial-accepted drafts is introduced (2020, effective 2021); the supply-chain bill platform launches (2020); and the new-generation system lands (2022).

**Era 4 — prudential-boundary setting (2022–present).** The 2022 joint measures set acceptance ceilings and cut the tenor; the 2025 six-department supply-chain notice pulls the electronic-receivables-certificate market inside the perimeter and pushes participants toward the bill instrument; the ECDS is retired (2024); and the disclosure machinery is extended (2025–2026).

The direction of travel is consistent: **narrower tenor, harder ceilings, more disclosure, one platform, and the deliberate absorption of shadow instruments into the regulated bill perimeter.**

---

## 4. System Mechanics

### 4.1 Participants and Access Modes

Participation is a two-layer structure: **bill-market infrastructure participants** with access to the Exchange's platform, and **enterprises**, which reach the system through their bank.

| Actor | Chinese | Role |
| --- | --- | --- |
| Bill-market infrastructure operator | 票据市场基础设施运营者 | The Exchange; operates the platform, maintains the register, takes disclosure data |
| Access institution | 接入机构 | A financial institution connecting to the platform and serving enterprise clients |
| Direct-access participant | 直连接入 | Develops to the Exchange's published interface specification and connects its own internal system |
| Indirect participant (client mode) | 客户端 / 间接参与者 | Keys business into the Exchange's terminal on the client's instruction, with a message channel to notify clients |
| Enterprise user | 用票企业 | Draws, accepts, endorses, discounts and presents through corporate e-banking |
| Broker | 票据经纪机构 / 票据中介 | A licensed financial institution acting as a matched-bargain broker, regulated separately from proprietary trading |

The **direct-versus-indirect distinction is architecturally load-bearing** and survived the ECDS retirement intact. The Exchange's migration notice restated both modes for the successor system: mode one ("direct connection") requires the member to develop its internal system *and* e-banking system to the Exchange's migrated-bill interface specification so corporate clients continue acting through e-banking; mode two ("client mode") has the member key business elements in manually on the client's mandate while developing a notification message so clients can see the bill and its outcome. A direct connector owns an interface implementation and a permanent engineering obligation; a client-mode participant accepts a manual dependency and must still build its own notification layer. The 8 January 2024 default to mode two quietly reveals which the Exchange considered less effortful.

### 4.2 The Lifecycle Operations

第三条 of 令〔2022〕第4号 lists the operations that must run through a PBOC-recognised bill-market infrastructure:

| Step | Chinese | Who acts | What happens |
| --- | --- | --- | --- |
| Drawing | 出票 | Drawer | Instrument created in the register with its elements and maturity date |
| Acceptance | 承兑 | Payer (bank, finance company or corporate) | The unconditional payment promise is recorded; the credit-creating event |
| Endorsement | 背书 | Holder | Rights transfer; all endorsers become jointly liable |
| Discounting | 贴现 | Bank / licensed lender | Holder sells before maturity at a discount |
| Interbank rediscounting | 转贴现 | Bank to bank | Trading of the already-discounted asset |
| Central-bank rediscounting | 再贴现 | Bank to PBOC | Eligible paper discounted by the central bank |
| Pledge | 质押 | Holder / pledgee | Draft encumbered as collateral, ownership retained |
| Guarantee | 保证 | Third party | Guarantee recorded against the draft obligation |
| Presentation | 提示付款 | Holder | Demand for payment at or after maturity |
| Payment or dishonour | 付款 / 拒付 | Acceptor | Funds move, or the refusal is recorded |
| Recourse | 追索 | Holder | Claim against acceptor, drawer and prior endorsers |

Note the ordering constraint: 第三条 says "贴现前的背书" — *endorsement before discounting* — must also run through the infrastructure. The system therefore covers not only the financing events but the transfer chain preceding them, which is what makes the register a complete ownership ledger rather than a partial one.

### 4.3 The Register as the Instrument, and Signals

**The central register is not a copy of the instrument; it is the instrument's authoritative record.** Legal effect attaches to the data message recorded in the system, and the electronic signature recorded there is the signature. Three regulatory layers support this: the 2022 measures require the listed operations to run through a recognised infrastructure, so a transaction completed outside it has no clean legal path for an electronic draft; the ECDS-era rules made paper-bill registration and enquiry mandatory for institutions handling paper, so the register covered the residue of paper too; and the penalty regime attaches consequences to an access institution whose *internal system records diverge materially from the central system's records* — making reconciliation between the bank's book and the register a legally enforceable duty rather than merely a control. For a bank architect, the register is the system of record for the instrument and the bank's own store is a *derived* view that must be proven to agree.

**Signatures.** Electronification replaces the physical seal (印章) with an electronic signature (电子签名) and a digital certificate (数字证书), turning signature validity into a cryptographic and identity-management problem rather than a physical-inspection problem. Article 80 of the 《电子商业汇票业务管理办法》, as amended by 令〔2024〕第5号, sanctions an access institution that fails to properly review the **authenticity of a client's electronic signature** where this causes loss, and separately sanctions the platform operator for failing to properly review an access institution's identity and signature. The same article reaches clearing failure from insufficient clearing funds, failure to forward bill messages in time, material divergence between the bank's stored data and the central system's data, and internal-system or platform faults not promptly remedied. Operationally, that is a list of the failure modes the regulator expects to be the real ones: key management is a regulated control rather than a utility, and the notification path is a regulated obligation, because a client that does not learn its bill was presented or dishonoured cannot defend its rights.

### 4.4 The Funds Leg and Delivery-versus-Payment

The instrument leg travels through the bill platform; the **funds leg travels through CNAPS** — the ECDS and its successor are not CNAPS applications, but settlement of funds legs flows through CNAPS (see [cnaps_guide.md](cnaps_guide.md)).

What matters here is the *coupling*. The ECDS was designed so that a discount, interbank rediscount or central-bank rediscount transaction, or a presentation for payment, would trigger immediate transfer settlement — **completing delivery of the instrument and payment of the funds simultaneously, achieving delivery-versus-payment (票款对付, DVP)**. That design is why a bill transaction can be treated as an atomic exchange rather than a sequential delivery-then-payment exposure. The practical consequence: any bank bill platform is a two-leg system requiring a compensating action, and the failure the regulator explicitly sanctions — clearing funds insufficient so that bill clearing fails and a party loses money — is precisely the failure of the funds leg after the instrument leg has committed. Reconciliation and pre-funding discipline on the funds leg are therefore not back-office hygiene; they are the control that keeps the DVP promise true.

---

## 5. The Trading and Money-Market Layer

### 5.1 贴现, 转贴现 and 再贴现

**贴现 — discounting.** The primary-market transaction: a holder of an unmatured draft sells it to an institution with lending qualifications, receiving face value less interest to maturity. The 2022 measures constrain both sides: Article 14 requires the discounter to be a PRC-established legal person or branch with lending qualifications and the applicant to be a natural person, PRC-established legal person or branch, or unincorporated organisation; Article 15 requires the applicant to have acquired the draft lawfully with a real transaction and debtor-creditor relationship with the drawer or a prior party (value-free acquisition excepted); Article 16 requires a discount application, an endorsed unmatured draft, and **materials capable of reflecting the real transaction and debtor-creditor relationship** — which is where the compliance cost lives, since discounting is legally a purchase of an instrument, economically a loan, and regulatorily carries a documentary condition borrowed from lending.

**转贴现 — the interbank leg.** Article 19 defers to the PBOC and CBIRC bill-trading rules rather than restating them, which is informative: the interbank leg is governed by the trading regime centred on the Exchange rather than the acceptance-and-discount rulebook. This is the leg that makes the instrument a money-market asset — a bank with a large discounted book can sell down, and a bank with surplus liquidity can buy short-dated bank-credit or corporate-credit exposure. Central registration and settlement let this happen within a single counterparty framework rather than bilateral credit lines against every other bank.

**再贴现 — the central-bank window.** Article 6 defines it as **the PBOC discounting unmatured commercial drafts already discounted by financial institutions — a monetary-policy tool of the central bank**; Article 20 provides that institutions conducting discounting business may apply, with conditions, rates, tenors and methods set by PBOC rules. Its current role is as a **structural, targeted liquidity tool rather than a general one**:

| Date | Programme action | Amount |
| --- | --- | --- |
| 30 June 2023 | PBOC increases the 支农支小 (agriculture and small-business support) relending and rediscount quotas | Rediscount quota +RMB 40 billion, to RMB 740 billion; relending quotas +RMB 160 billion in total |
| 15 January 2026 | PBOC announces, at a State Council Information Office briefing by Deputy Governor 邹澜, an increase in the agriculture/small-business relending quota plus a dedicated private-enterprise facility | Relending quota +RMB 500 billion; a dedicated private-enterprise relending quota of RMB 1 trillion, with the agriculture/small-business relending and rediscount quotas to be used in combination |

The 2026 announcement is the most recent structural development located by this guide, and notably folds the rediscount quota into a combined facility with relending. Market commentary also reports the PBOC rediscount rate held at 1.75% through 2025; that is not asserted here as primary-verified (see 13). The economic reading: rediscounting gives banks a funding backstop on eligible bill assets, compressing the spread they need to hold bills and hence lowering the rate at which they can afford to discount — a transmission channel from policy rate to enterprise financing cost, which is exactly what the 2022 measures mean by calling it a monetary-policy tool.

### 5.2 Brokerage, Pricing and the Investor Base

**Brokerage.** Article 17 permits a holder to conduct discount pricing enquiry and execution **through a bill brokerage institution**, and requires brokered discount transactions to run **through a PBOC-recognised bill-market infrastructure**. Article 18 defines the brokerage institution as a financial institution with good market reputation and active bill business, with a **dedicated brokerage department**, sound internal controls, a **dedicated brokerage channel**, professional staff, and **strict segregation between brokerage and proprietary business**. That segregation is the notable part: broking revenue is fee revenue, proprietary revenue is spread, and mixing them creates exactly the conflict the "bill intermediary" (票据中介) risk category exists to police. The 2020 standardised-note rules use the same vocabulary (Article 8).

**Pricing.** Bill pricing is quoted as a discount rate to maturity, with reference points at the rates of the highest-quality bank-accepted paper; market commentary routinely cites a rate for state-owned and joint-stock bank bill interbank rediscounting (国股银票转贴现利率) as the observable benchmark. **This guide could not verify the Exchange's exact published rate series — which rates it publishes, under what names, at what frequency** — recorded as an explicit gap in section 14 rather than asserted. What is structurally confident: the market prices on a ladder by acceptor credit, the tight end is bank-accepted paper of the largest banks, and the rates at which PBOC rediscounts eligible paper provide a floor-like reference for eligible collateral.

**The investor base.** Discounted bills are held predominantly by commercial banks through their discounting and trading books, with rural credit institutions and finance companies to a degree, and bond-market investors only through the standardised note, since an ordinary draft is not a bond-market instrument. Holdings follow a specific balance-sheet arithmetic: bills are short-dated (at most six months), carry an acceptance, and discounting is treated as an extension of credit — attractive when a bank wants short-dated low-risk earning assets, unattractive when yield compresses below funding cost. Reported market behaviour through 2023–2025 shows exactly this, with some mid-sized banks sharply reducing discounting balances while large banks increased theirs and discount yields compressing toward the low single figures. Those figures come from aggregation commentary and are flagged accordingly.

### 5.3 标准化票据: the Standardised Note

The standardised note is the market's attempt to build a bond-market-ready instrument out of bills.

| Item | Value |
| --- | --- |
| Instrument | 《标准化票据管理办法》 |
| Issuing body | 中国人民银行 (PBOC) |
| Number | 中国人民银行公告〔2020〕第6号 |
| Signed / issued | 24 June 2020 |
| Effective | 28 July 2020 |
| Governing law referenced | 《中国人民银行法》, 《信托法》, 《票据法》 |
| Definition (第二条) | A depository institution aggregates commercial drafts with similar core credit elements and similar tenors into an underlying asset pool and creates **divisible beneficiary certificates** (等分化受益凭证) backed by the pool's cash flows |
| Legal character (第四条) | A **money-market instrument**, under PBOC macro-control and supervision |
| Trading venue (第十七条) | The **interbank bond market and the bill market**, under the interbank bond trading rules |
| Transaction types (第十八条) | Outright sale and purchase, repurchase, and forward |
| Depository institution qualifications (第六条) | A commercial bank or securities company familiar with bill and bond market business, adequately staffed and controlled, financially sound, with no material violation in the preceding two years |

The design is recognisable securitisation: an originator (原始持票人) endorses drafts into a pool to a depository institution (存托机构) for consideration; the drafts must be held independently of the depository institution's own property (第十条); rights must be transferred completely by endorsement with no fraudulent or fictitious deposit and no self-subscription (第七条); a bill brokerage institution may aggregate the assets (第十二条); and the bill-market infrastructure provides registration, custody, clearing and settlement for the underlying assets while a bond-market depository handles the notes (第十三条, 第十六条).

**Scale and status — stated honestly, not promotionally.** On the regime's first day, 28 July 2020, the Exchange published **14 standardised note products with aggregate creation volume of RMB 1.144 billion**; 13 of the 14 were backed by undiscounted commercial-accepted drafts and one by an undiscounted bank-accepted draft. That is a pilot-scale number, not a market. Chinese bill-industry commentary subsequently described the instrument as **suspended** — an industry retrospective published in 2024 was titled "three years since the standardised note was suspended" — and this guide found no evidence of an active primary market. The plausible revival path is now via supply-chain bills: the April 2025 six-department notice encourages financial institutions, lawfully and with controlled risk, to **run pilot asset-securitisation of supply-chain bills** to broaden bill financing channels. The honest position: **the legal regime is live, the market has been dormant, and policy has re-opened a route back through supply-chain bills.** Any statement that the standardised note market is thriving would be false.

---

## 6. Regulatory Framework

### 6.1 The Negotiable Instruments Law and Its Amendment Status

The statute is the 中华人民共和国票据法, adopted 10 May 1995 and amended 28 August 2004. Its two-decade-plus record, and the fact that the electronic regime is built on a statute drafted for paper, is why an amendment has long been discussed.

**Where the amendment actually stands.** The NPC Standing Committee's 2025 annual legislative work plan — the full text of which was examined for this guide — does **not** list the 票据法 among laws to be enacted, revised or held for preliminary review. The financial-law items it does list are telling by contrast: enactment of a Financial Law (金融法) and a Financial Stability Law (金融稳定法); revision of the Law of the People's Bank of China (中国人民银行法) and the Law on Supervision and Administration of the Banking Industry (银行业监督管理法); and, among preparatory review items, revision of the Commercial Bank Law (商业银行法). A widely circulated market retrospective reports that PBOC's second-half 2025 work conference mentioned **accelerating the amendment of the 票据法**, describing the basic legislation as entering a substantive phase — a secondary report of an official meeting statement, flagged as such in 13. The defensible statement is a negative finding plus a flagged secondary signal:

- ❌ **Rejected:** "the 票据法 has been amended to accommodate electronic instruments" — no such amendment was located.
- ⚠ **Flagged:** an amendment has long been discussed; PBOC's 2025 second-half work conference reportedly called for accelerating it.
- ✅ **Verified:** the statute in force is the 1995 text as amended in 2004, and it is not on the NPC Standing Committee's 2025 legislative work plan.

### 6.2 The ECDS Business Measures and the 2024 Revision

The electronic instrument's rulebook is the 《电子商业汇票业务管理办法》, originally published by PBOC on 16 October 2009 as 中国人民银行令〔2009〕第2号, alongside the 《电子商业汇票系统管理办法》 published 10 September 2010, which governs the platform operator, participants, business processing and risk management.

**The 2024 revision — verified.** The 《电子商业汇票业务管理办法》 was amended by 《中国人民银行关于修改部分规章的决定》, adopted at PBOC's 8th administrative meeting of 2024 on 3 September 2024, promulgated as **中国人民银行令〔2024〕第5号 on 22 October 2024**, effective **1 January 2025**. The decision amended six regulations to implement the Administrative Penalty Law of the PRC and clean up penalty provisions; the bill-specific change is to **Article 80**, which now provides that where specified circumstances exist and are serious, penalties apply under Article 46 of the Law of the People's Bank of China. The enumerated circumstances are set out in 4.3. Note what the revision *is*: a penalty-provision amendment, not a restructuring. It changed no tenor, no acceptance ceiling and no disclosure requirement, and treating it as a substantive overhaul would be wrong.

### 6.3 The 2022 Joint Bill Measures

《商业汇票承兑、贴现与再贴现管理办法》, 中国人民银行 中国银行保险监督管理委员会令〔2022〕第4号 — the tenor detail is in 2.2; here is what it constrains and whom it binds.

| Article | Constraint | Who it binds |
| --- | --- | --- |
| 3 | Specified operations must run through a PBOC-recognised bill-market infrastructure; 供应链票据 is an electronic commercial draft | All parties to electronic drafts |
| 5, 15 | Acquisition must be lawful and backed by a real transaction and debtor-creditor relationship (value-free acquisition excepted) | Holders and discounters |
| 8–10 | Defines the three acceptor categories; requires PRC establishment plus a licence with bill acceptance in scope | Acceptors |
| 11 | Must strictly review the drawer's real trade relationship and acceptance risk; accepted amount must match the real transaction and the applicant's solvency | Banks, rural credit cooperatives, finance companies |
| 12 | Margin accounts separately established; margins not misappropriable or withdrawable at will | Acceptors |
| 13 | Bill acceptance business must sit inside the unified credit-limit and risk-management framework | Deposit-taking institutions |
| 14–16 | Discounter must have lending qualifications; applicant and documentation requirements | Discounters and applicants |
| 17–18 | Brokered discounting through a recognised infrastructure; brokerage segregated from proprietary business | Brokerage institutions |
| 22 | Acceptor and discounter must be financially sound, free of persistent bill overdue or non-disclosure for two years | Acceptor and discounter |
| 24 | Bank-accepted and finance-company-accepted **maximum acceptance balance ≤ 15% of the acceptor's total assets**; **margin balance ≤ 10% of absorbed deposits**; PBOC and CBIRC may set further ratios | Acceptors — **effective 1 January 2024, deferred by Article 41** |
| 25 | Payment tenor must match the real transaction's performance period; **not more than 6 months** from issue to maturity | All commercial drafts |
| 26–31 | Disclosure obligations; mandatory check of disclosure information before discounting | Acceptors, discounters, endorsees, infrastructure |
| 36–39 | Institutions may not serve persistently overdue or non-disclosing acceptors; unlicensed discounting handled under relevant law; fraudulent procurement of acceptance or discounting attracts liability and, if criminal, referral | Institutions and parties |

Article 24 deserves separate emphasis as the deferred and most quantitatively constraining provision. **Article 41 provides expressly that Article 24 takes effect on 1 January 2024**, one year after the rest of the order. It caps a bank's or finance company's acceptance balance at 15% of total assets and its margin balance at 10% of absorbed deposits — for a bank with a large bill book, a hard ceiling on balance-sheet capacity, and the provision most likely to force a business-model change rather than a paperwork change.

### 6.4 Disclosure and the Overdue Lists

**The disclosure regime.** 中国人民银行公告〔2020〕第19号, on disclosure for commercial-accepted drafts, was issued on 23 December 2020 and took effect on **1 August 2021**. Operatively: the acceptor must disclose when acceptance is completed and thereafter; disclosure covers the bill's key elements and credit information for commercial-accepted and finance-company-accepted drafts, while bank-accepted draft acceptors disclose acceptor credit information; the bill-market infrastructure is directed to issue detailed **operating rules for commercial-accepted draft disclosure**, filed with PBOC and reported periodically; and the infrastructure monitors disclosure and, where an acceptor is persistently overdue or has filed false, omitted or delayed information, takes measures and reports to PBOC. The Exchange's 《商业汇票信息披露操作细则》 is the resulting operating rulebook.

Article 28 of the 2022 measures hardens this into a transaction condition: a discounter **must check** the disclosed information, and where it does not exist or the recorded particulars are inconsistent with the disclosure, it **must not** discount for the holder. The endorsement-side duty (Article 29) is permissive — the endorsee "may" check — a deliberate allocation of the verification burden toward the discounter, who is the lending bank.

**The published overdue lists** are the regime's most consequential output:

| As at | Finding | Published |
| --- | --- | --- |
| 31 December 2025 | 4,226 acceptors on the commercial-acceptance overdue list; **774** met the Exchange's "persistently overdue" (持续逾期) standard — 3 or more overdue events between 1 July and 31 December 2025 with a month-end overdue balance or an overdue event in the month | 9 January 2026 |
| 31 October 2025 | Lists published; reporting noted 81 bond-issuing local-government financing vehicles with bill overdue, 6 appearing for the first time | November 2025 |
| 31 May 2025 | 887 acceptors had 3 or more overdue bills between 1 December 2024 and 31 May 2025 with a month-end overdue balance or an overdue event in the month | 2025 |

Under the 《商业汇票信息披露操作细则》, the Exchange **suspends** bill acceptance services for listed acceptors and restores service the following month if the acceptor no longer meets the overdue criteria — a direct operational sanction, not a reputational one: an acceptor that stops paying its bills cannot issue new ones.

**Notification rules, 2025–2026.** On 2 January 2025 a new dishonour-notice function for presentation went live: when an enterprise's presentation is refused, the system automatically generates a dishonour notice, pushes it to the enterprise's bank's e-banking system, and the bank notifies all prior parties, synchronising dishonour information across the chain. From 1 November 2025 the 《上海票据交易所间接参与者服务指引》 took effect, establishing that a dishonour notice **takes effect when it reaches the bill-using enterprise's account bank** — closing the argument that an enterprise can escape responsibility by not logging in. On 30 January 2026 the Exchange issued 《关于优化票据披露有关事项的通知》, optimising disclosure enquiry from February 2026 so participants can query overdue status directly on the acceptor credit-information page. These three developments are the clearest recent evidence that the regulator's operational focus has moved to **information symmetry along the chain**.

### 6.5 Real Trade Background and Financing-Only Issuance

The real-trade-background requirement (真实交易背景) is the spine of the regime. It appears in the definitions (Article 5), the acceptance duty (Article 11), the discounting application (Articles 15–16), and the penalty chapter: Article 37 provides that where a financial institution processes acceptance or discounting for a drawer or holder without a real transaction and debtor-creditor relationship (value-free acquisition excepted), the regulator may suspend its bill business or impose administrative penalties and pursue responsible directors, senior managers and directly responsible staff.

**Financing-only issuance (融资性票据)** — a draft issued purely to raise money with no underlying trade — is therefore prohibited by construction, with no separate prohibition article because the real-trade requirement already excludes it. The practical difficulty is that "real trade background" is a documentary test applied to an instrument designed to circulate independently of documents; the 2022 measures acknowledge the tension by confining the requirement to *acquisition* and acceptance rather than every subsequent transfer, which is why downstream endorsees have a permission rather than an obligation to re-check.

**The AML and risk-control overlay** therefore layers standard financial-crime controls over the bill business rather than creating bill-specific ones: client identification and basic-information review by access institutions (Article 80 of the electronic-draft measures, as amended), credit-limit and risk-management integration (Article 13), prohibited unlicensed discounting (Article 39), and a general requirement that institutions maintain sound bill-business management and internal controls covering market, credit and operational risk (Article 21). The unlicensed-discounting provision is the sharpest anti-disintermediation tool, reaching the informal "bill brokerage" operators who are the classic vector for financing-only paper.

### 6.6 The 2025 Supply-Chain Notice

**The most significant recent addition to the perimeter.** 《关于规范供应链金融业务 引导供应链信息服务机构更好服务中小企业融资有关事宜的通知》, issued by six bodies — 中国人民银行, 金融监管总局, 最高人民法院, 国家发展改革委, 商务部, 市场监管总局 — as **银发〔2025〕77号**, dated **26 April 2025**, effective **15 June 2025**. Its provisions bearing on the bill market:

- It brings **electronic receivables certificates (应收账款电子凭证)** inside the perimeter for the first time, defining them as electronic records issued by a supply-chain core enterprise to a creditor through a supply-chain information service system, promising payment on schedule (Article 9).
- It repositions supply-chain finance platforms as **supply-chain information service institutions** that must return to an information-service role, may not conduct payment settlement, financing guarantee, factoring or lending without a licence, may not directly or indirectly pool funds, and must not let an information intermediary degenerate into a credit intermediary (Article 4).
- It imposes a **term limit on receivables certificates**: in principle within 6 months and in no case more than 1 year; above 6 months, banks must scrutinise the reasonableness of the term and industry settlement custom before financing (Article 11).
- It requires receivables-certificate financing to be registered at PBOC's Credit Reference Centre **movable-asset financing unified registration and publicity system** (Article 13), and settlement to run through banks rather than the platform's own account (Article 14).
- It requires the platform to **stop providing new certificates** to a core enterprise that fails to pay at maturity, has defaulted on bond issuance, or is persistently overdue on accepted bills without having cleared the position (Article 15).
- It directs **上海票据交易所 (the Exchange)** to organise information aggregation for receivables certificates, run statistical monitoring and analysis, and provide enquiry services (Article 19).
- On bills: it encourages banks to improve supply-chain bill management and systems and to **expand supply-chain bill application**, directs research toward a pilot of **limited-recourse supply-chain bill services**, and encourages **orderly pilots of supply-chain bill asset securitisation** to broaden bill financing channels (Article 2).
- It gives receivables-certificate provisions a **two-year transition period from implementation**, i.e. running to about 15 June 2027.

This is a perimeter move with teeth: the unregulated substitute that grew alongside the bill market — the platform-issued receivables certificate — is now defined, term-limited, registered, settlement-constrained and monitored by the same infrastructure as bills. Press reporting has characterised the affected stock in the trillions of renminbi and described hundreds of platforms adjusting; those are press estimates, not PBOC statistics, and are flagged as such.

### 6.7 Who Is Bound by What

| Regime | Instrument | Binds | Constrains |
| --- | --- | --- | --- |
| Statute | 中华人民共和国票据法 (1995, amended 2004) | Everyone dealing in the instrument | Legal character, form, transfer, recourse |
| PBOC/CBIRC order | 令〔2022〕第4号, effective 1 Jan 2023; Articles 24 and 41 deferred to 1 Jan 2024 | Acceptors, discounters, brokers, in-scope institutions | Tenor, acceptance ceilings, margin rules, disclosure duties, penalties |
| PBOC rule | 《电子商业汇票业务管理办法》 (2009, amended by 令〔2024〕第5号, effective 1 Jan 2025) | Platform operator, access institutions, enterprise clients | Electronic operations and penalty regime |
| PBOC announcement | 公告〔2020〕第19号, effective 1 Aug 2021 | Commercial-accepted and finance-company-accepted draft acceptors | Disclosure obligations |
| PBOC announcement | 公告〔2020〕第6号, effective 28 Jul 2020 | Depository institutions, originators, brokers, investors | Standardised note creation and trading |
| Six-department notice | 银发〔2025〕77号, effective 15 Jun 2025 | Banks, core enterprises, supply-chain platforms | Receivables certificates, platform conduct, bill expansion policy |
| Exchange rules | Business rules, disclosure operating rules, indirect-participant service guide | Market members and users | Operational procedure, disclosure lists, notification effect |
| PBOC trading rules | 《票据交易管理办法》 (issued 6 Dec 2016) | The Exchange and market participants | Designates the Exchange; trading, custody, clearing, information services |

---

## 7. Credit, Fraud and Risk

### 7.1 Credit Risk of the Commercial-Accepted Species

The commercial-accepted draft is the market's credit instrument. Its acceptor is a non-bank legal person — typically a large manufacturer, property developer, construction group or trading company — and its value is exactly the acceptor's ability and willingness to pay on a date no more than six months away. Three features make this risk sharper than an equivalent loan. **It is unsecured in substance:** there is usually no mortgage, and the holder's security is the instrument plus the joint liability of the endorsement chain. **The holder may not have chosen the risk:** a small supplier that receives a commercial-accepted draft from a large customer did not underwrite that customer, and if it discounts the draft the bank then holds an exposure it never originated. **It is correlated within a supply chain:** a single anchor enterprise's distress propagates across every supplier holding its acceptances, and the endemic practice of extending supplier payment terms with acceptances amplifies that correlation.

The 2022 measures address this prudentially (Articles 22, 36: no service to acceptors persistently overdue or non-disclosing for two years) but cannot address it on the credit side. No rule makes a weak acceptor strong.

### 7.2 Documented Default Evidence

This guide does not name individual defaulting corporates, because it could not establish primary documentation for any specific company's bill default, and naming one on the strength of secondary aggregation would be invention. What can be documented is the aggregate, published by the market infrastructure itself: 4,226 acceptors appearing on the commercial-acceptance overdue list as at 31 December 2025, of which **774** met the Exchange's persistently-overdue (持续逾期) standard — three or more overdue events between 1 July and 31 December 2025 with a month-end overdue balance or an overdue event in that month (Exchange 《持续逾期名单》, published 9 January 2026); 887 acceptors with three or more overdue bills between 1 December 2024 and 31 May 2025; and 81 bond-issuing local-government financing vehicles with bill overdue as at 31 October 2025, six appearing for the first time (Exchange lists, reported November 2025).

Two conclusions are defensible. First, **an overdue list of 4,226 acceptors, with 774 of them meeting a three-strikes-in-six-months persistent-default test, is a systemic, not idiosyncratic, credit condition.** Second, sectoral concentration is reported by industry press as skewed toward property and construction, and the Exchange publishes sectoral slices of the persistent-overdue list; this guide treats the sector observation as press-reported rather than independently verified.

**How this became knowable is the important part.** Before the disclosure regime there was no public, systematic count of bill dishonour. The lists are a regulatory artefact of 公告〔2020〕第19号 and the Exchange's disclosure operating rules. Credit transparency in this market is roughly five years old.

### 7.3 Fraud Typologies and What Electronics Changed

| Typology | Chinese | Pre-electronic mechanism | Status under the new regime |
| --- | --- | --- | --- |
| Forged bills | 伪造票据 | Fabricate a physical instrument with a copied seal | Largely eliminated: the register is authoritative; a "bill" not in the system has no legal existence |
| Altered bills | 变造票据 | Change a denomination or date on paper | Largely eliminated: elements are immutable register fields |
| Forged seals | 私刻印章 | Counterfeit a corporate or bank seal | Now a cryptographic problem: the control is certificate and signature authenticity review, sanctioned by Article 80 of the electronic measures |
| Fake trade background | 虚假贸易背景 | Fabricate a contract to justify financing-only paper | **Intensified in relative importance** — its dominance is why Articles 15–16, 28 and 37 exist |
| Broker/intermediary misconduct | 票据中介风险 | Unlicensed intermediaries aggregating and mismatching paper | Constrained by restricting brokered dealing to licensed brokers with segregated books (Articles 17–18) and prohibiting unlicensed discounting (Article 39) |
| Custody and original-document risk | 票据保管风险 | Lost, stolen or substituted physical bills in custody | Eliminated — no physical original to misplace |
| Duplicate financing | 一票多卖 | The same paper bill financed twice at different banks | Eliminated by the single authoritative register |

The pattern is consistent: **electronification eliminated the physical fraud typologies and left the informational ones.** The surviving risk is that the instrument is genuine but the transaction behind it is not. Disclosure as a credit-information mechanism has corresponding limits worth stating explicitly — it covers acceptors rather than drawers or endorsers; it is behavioural (bill payment history) rather than financial (solvency); it cannot reach paper residual below the electronic mandate threshold; it gives no forward-looking warning; and completeness depends on the acceptor, as Article 31's contemplation of *false, omitted or delayed* information acknowledges.

### 7.4 Recourse in Practice and the 票据池 Product

On dishonour the holder's remedy is 追索权 — a claim against the acceptor, the drawer and every prior endorser jointly. Three things changed with electronification and the recent notification rules. The **dishonour record is now automatic**: from 2 January 2025, on refusal, the system generates a dishonour notice, pushes it to the presenting enterprise's bank, and the bank notifies all prior parties; from 1 November 2025, under the Exchange's indirect-participant service guide, the notice **takes effect on reaching the bill-using enterprise's account bank**, removing the "I never logged in" defence. The **endorsement chain is fully reconstructible**, making joint liability practical rather than theoretical. And the **dual-track litigation choice remains**: a holder may sue on the instrument, with the joint-liability advantage, or on the underlying legal relationship. Market commentary reports that Chinese courts have engaged with the effectiveness of off-system recourse where parties acted outside the electronic channel; this guide did not verify that case law and asserts no legal position on it.

**The 票据池 (bill pool) product.** A bank product rather than a regulated instrument: the bank aggregates a corporate client's incoming bills into a managed pool and, against that pool, provides financing, pledge facilities or a settlement service. It solves a real problem — a supplier holding many small bills maturing on different dates wants liquidity against the aggregate, not bill by bill. On the new-generation platform the product is materially more capable, because **divisibility (等分化)** lets a client hold a bill package and split off exactly the amount needed for a payment, pledge or discount, turning the pool from an off-system ledger construct into something the central register itself can represent. The risk shape follows: the pool is a secured facility whose collateral is a set of accepted instruments, so residual risks are acceptor credit concentration within the pool, valuation (a bill is worth the acceptor's credit, not the face amount), pledge-registration correctness, and release and margin mechanics when bills mature inside the pool.

### 7.5 Risk-Control Implications for a Bill Desk

1. **Acceptor limits, not instrument limits.** The exposure is the acceptor's credit; a bill limit framework not keyed to the acceptor is not a limit framework. Joint liability justifies separate limits on drawers and major endorsers, but the primary limit is the acceptor.
2. **Mandatory disclosure verification as a hard gate.** Article 28 makes it a legal condition, not a preference: no discount where disclosure is absent or inconsistent. This belongs in the system as a blocking control, not in an operations manual.
3. **Overdue-list screening at intake and on a schedule.** The Exchange's lists are the market's only systematic early-warning dataset; a desk that does not screen them daily is unmanaged.
4. **Trade-background evidentiary discipline.** Since the surviving fraud type is fake trade background, the control is documentary verification quality — and Article 37 makes the *institution* liable for accepting a fabricated relationship.
5. **Reconciliation as a legal duty.** Article 80 penalises material divergence between the bank's stored bill data and the central system's data, so reconciliation is a compliance control with a sanction attached.

---

## 8. The Supply-Chain Finance Angle

### 8.1 供应链票据: Instrument and Platform

A supply-chain bill (供应链票据) is, by express regulation, **an electronic commercial draft** — 第三条 of 令〔2022〕第4号 says so, and that sentence determines everything about it: the disclosure regime applies, the acceptance rules apply, the tenor cap applies, and the instrument carries the same legal character as any other electronic draft. Its distinguishing feature is the channel of issue, through the Exchange's **supply-chain bill platform** (供应链票据平台).

| Item | Verified detail |
| --- | --- |
| Operator | 上海票据交易所 (Shanghai Commercial Paper Exchange) |
| Platform | 供应链票据平台 (supply-chain bill platform) |
| Trial launch | 24 April 2020, under Exchange notice 票交所发〔2020〕58号, with PBOC consent |
| Policy framing | 应收账款票据化 — "the bill-isation of accounts receivable" |
| First batch | 4 supply-chain finance platforms participated |
| First-day activity | 17 enterprises issued 17 bills totalling RMB 1.0442 million; 2 enterprises endorsed 3 bills totalling RMB 51,000 |
| Access | Supply-chain finance platforms may connect to the bill platform on agreed conditions |

That first-day volume — just over one million renminbi — is the honest scale of the launch. The instrument became a policy priority only gradually, and its recent prominence is largely a consequence of the 2025 receivables-certificate rectification described in 8.5.

### 8.2 等分化 and What It Economically Enables

Divisibility (等分化, also described as 票据拆分 — bill splitting) is what makes the supply-chain bill economically different from a conventional draft.

| Property | Detail |
| --- | --- |
| Mechanism | The issuer issues a **bill package** (票据包) composed of units of a standard amount rather than one indivisible instrument |
| Standard minimum unit | **RMB 0.01** |
| What the holder can do | Split the package by actual amount across endorsement, discounting, guarantee, pledge and deposit operations |
| Stated benefit | The holder's bill amount can match its payment obligation exactly, solving the mismatch between the bill held and the amount payable |
| Reported split limit | Up to 500 splits per new-generation bill (industry press) |

Three economic consequences follow. A large supplier can pay many small suppliers from one package without issuing many instruments or arranging many transfers — precisely how supply-chain settlement actually works. A small supplier can discount only the amount it needs, so financing cost scales with the funding need. And divisibility makes the draft functionally similar to a platform-issued receivables certificate — but inside the regulated bill perimeter, with disclosure, a central register and the acceptance regime attached. That third point is the strategic one: **divisibility lets the regulated instrument compete with the unregulated substitute on the substitute's own terms.**

### 8.3 Bills Alongside Receivables Financing

China's supply-chain finance market offers a supplier roughly four ways to be paid late and financed early. They differ on the dimension that matters — whose credit the holder relies on, and which regulator owns the instrument.

| Instrument | Credit relied on | Legal character | Regulatory owner | Transferability |
| --- | --- | --- | --- | --- |
| Trade receivable + factoring | Anchor enterprise's payment obligation | Contractual claim | Civil law; factoring rules | Assignment, with notice issues |
| Electronic receivables certificate | Anchor enterprise, via platform record | Recorded contractual promise | Defined and constrained by 银发〔2025〕77号 | Within the platform; layer/transaction limits |
| Conventional commercial draft | Acceptor (bank or corporate) | Negotiable instrument under 票据法 | PBOC / Exchange / regulators | Free by endorsement |
| Supply-chain bill | Acceptor, with anchor-enterprise context | Electronic commercial draft under 票据法 | PBOC / Exchange | Free by endorsement, divisible |

The crucial asymmetry is transferability and legal robustness: a negotiable instrument travels with joint liability and a statutory holder position, whereas a platform receivable is a contractual claim recorded in a private system. That is why the policy direction since 2025 has been to move volume toward the bill.

**The anchor-enterprise model.** Both the certificate and the bill model are anchor-centric: a near-investment-grade core enterprise's credit is extended to its supplier tiers so a small supplier can be financed at a rate closer to the anchor's than to its own. The bill variant adds two things the certificate variant does not: the acceptance is a **statutory unconditional payment promise on a negotiable instrument**, enforceable independently of the underlying contract; and the instrument sits inside a **central register with a disclosure regime**, so the anchor's bill payment behaviour is publicly scored — and since the 2025 notice a core enterprise persistently overdue on accepted bills is cut off from new receivables certificates as well. The structural weakness is unchanged: **concentration.** If the anchor fails, every tier holding its acceptances fails at once, and divisibility increases the number of holders. Divisibility is therefore a distribution mechanism for anchor credit risk, which is why the Exchange monitors split layers and transaction counts and why the 2025 notice requires information-service institutions to manage split-transfer layers and report abnormal splitting.

### 8.4 Where the Chinese Bill Is Structurally Different

Read [supply_chain_finance_guide.md](supply_chain_finance_guide.md) for the SCF discipline itself — the working-capital gap, payables finance, receivables finance, platform models, the technology stack. That guide owns the discipline; this section only draws the contrast.

A conventional Western payable-finance programme is a **bilateral financing arrangement** over a **non-transferable contractual payable**, documented by a **master agreement**, financed by **one or a small group of funders**, with the anchor's credit imported by contract and no secondary market. A Chinese supply-chain bill is a **negotiable instrument**, **transferable by endorsement with joint liability**, registered in a **national central system**, subject to a **mandatory public disclosure regime for the acceptor**, **divisible to RMB 0.01**, potentially **discounted, interbank-traded and central-bank-rediscounted**, and now with a policy gateway toward **securitisation**. It is not a financing programme; it is money-market paper issued at the point of trade settlement. Three practical consequences follow for an architect used to Western SCF: there is no programme agreement to negotiate, because the terms are regulation and infrastructure rules; there is no funder-concentration question in the Western sense, because the instrument is designed to be sold; and credit assessment is not bilateral underwriting but a lookup against a public acceptor record plus a market spread.

### 8.5 The 2025–2027 Rectification and the Shift Toward Bills

The single most consequential recent development in Chinese supply-chain finance is the perimeter move described in 6.6, whose effect on the bill market is directional and specific:

- Receivables certificates are now **defined, term-limited (in principle ≤6 months, in no case >1 year), registration-required and settlement-constrained**, with a **two-year transition to about 15 June 2027**.
- Platforms issuing them are repositioned as **information-service institutions**, barred from unlicensed financial business and from pooling funds.
- The notice explicitly encourages **expansion of supply-chain bill application**, **limited-recourse supply-chain bill pilots**, and **orderly supply-chain bill asset-securitisation pilots**.
- The Exchange is designated to **aggregate information and run statistical monitoring** on the receivables-certificate business.

The market read — reported in Chinese financial press and consistent with the notice's text — is that anchor enterprises and their platforms will migrate volume toward supply-chain bills to stay inside the perimeter. Press estimates of the affected certificate stock run into trillions of renminbi; those are press figures, not official statistics, and this guide labels them accordingly. For a foreign bank the strategic implication is straightforward: **the regulated Chinese bill instrument is the growth channel, and the unregulated certificate is the sunset channel.**

---

## 9. Technology and Integration

### 9.1 What a Bank Must Build

| Block | Purpose | Key requirement |
| --- | --- | --- |
| Connectivity | Talk to the Exchange's new-generation platform | A direct-connect interface implementation, or a client-mode operating process with notification messaging |
| Signing and identity | Make electronic signatures legally effective | Certificate lifecycle, key management, separation of signing authority from business authority |
| Internal bill system | Model the instrument and its lifecycle | Register-agreeing data model covering 出票/承兑/背书/贴现/质押/保证/提示付款/追索 |
| Settlement integration | Move the funds leg atomically with the instrument leg | Pre-funding and DVP discipline against CNAPS settlement |
| Accounting and reporting | Book the asset and report it | Discounted-bill classification, PBOC and regulator aggregates, social-financing inputs |
| Reconciliation and control | Prove the bank's book agrees with the register | Scheduled and event-driven reconciliation with an exception workflow |

### 9.2 Interface, Signing and the Data Model

The access-mode choice in 4.1 is the first architectural decision. Because the Exchange's migration notice required direct-connect members to have their internal system *and* e-banking system developed to the published interface specification, and because the election had a hard date (8 January 2024), the realistic advice is: **decide the mode before the interface specification changes, not after.** A direct connector owns an interface implementation and a testing relationship it must sustain across specification changes; a client-mode participant accepts a manual dependency and must still build a notification channel so its corporate clients see presentation, dishonour and maturity events.

Signing infrastructure is a regulated control surface, since Article 80 sanctions an access institution for failing to properly review the authenticity of a **client's electronic signature** where loss results. Three practical risks dominate: certificate expiry without operational alerting (a bill that cannot be signed at the moment of maturity), signing keys held by operations staff without dual control, and signature verification delegated entirely to the client-facing channel with no control point on the bank side.

The data model must treat the central register as the system of record for the instrument and the bank's store as a derived replica whose divergence is sanctionable. That inverts the usual design instinct: the bank's bill module is not master data, it is a **cached, reconciled view plus the bank's own transactional and accounting state**. A workable decomposition: an **instrument store** (bill identity, elements, acceptance, maturity, current holder, endorsement chain, dispute and overdue flags, keyed on register identifiers); a **transaction store** (the bank's own business events with the settlement reference for each); a **position and limit store** (acceptor exposure, drawer and endorser contingency exposure, maturity ladder, availability under the 15%-of-assets and 10%-of-deposits ceilings); and a **collateral and pledge store** mapping pooled instruments to facilities for the bill-pool product. Reconciliation must run at two frequencies — **event-driven** on every lifecycle message, proving the instrument leg and funds leg both completed, and **scheduled** against the register, instrument by instrument, with exception classification, because the divergence itself is the regulated event.

### 9.3 Integration, Reporting and Resilience

Integration points and the trap in each: **core banking and customer** — the bill platform's enterprise identity must resolve to the same customer record used for lending limits and AML; **lending and limit systems** — discounting is credit and must consume the credit-limit framework, and bill limits keyed on the *customer* rather than the *acceptor* understate concentration (see [banking_limits_domain_guide.md](banking_limits_domain_guide.md)); **general ledger** — discounted bills are assets while acceptance is a contingency, and the 15%-of-assets ceiling is computed on the acceptance book; **regulatory reporting** — bill volumes feed PBOC aggregates including the social-financing series, and the classification question is which discounted bills count as credit to the real economy and whether an acceptance is on or off balance sheet; and **trade/supply-chain platforms** — where the bank acts as information-service counterparty on a supply-chain bill flow, the boundary of responsibility is set by 银发〔2025〕77号, including the prohibition on a bank providing its own supply-chain information service system to third parties.

Reporting obligations in one place: acceptance and discounting volumes and balances; the acceptance-ceiling and margin-ratio metrics of Article 24; disclosure-status checks evidenced per discount; overdue and dishonour events; and, where the bank operates adjacent receivables-certificate business, information reported to the Exchange and to industry self-regulatory bodies under the 2025 notice. **Resilience is unusually well specified here, because the regulator has already enumerated the failure classes it expects.** Article 80 makes an access institution liable for an internal-system fault not promptly remedied where it has significant impact, and for failure to forward bill messages in time causing loss. The obligations that follow: **message forwarding as a monitored service level** (it is a sanctioned failure mode), **the funds-leg pre-funding position as a first-class operational control**, **a maturity-date alerting regime** so instruments cannot lapse for want of attention, and **a tested reconciliation exception workflow**, because unreconciled divergence is itself a sanctioned state.

### 9.4 Blockchain and Platform Initiatives

This is the area with the most announced-and-abandoned activity, and the honest position is that this guide **could not verify any live, in-production blockchain-based bill platform of market significance.** The space has seen bank consortium prototypes, distributed-ledger pledge concepts and pilot announcements; the material at primary sources reviewed here does not support asserting any of them as operational infrastructure. What *is* verifiable and material is the opposite of blockchain decentralisation: the market has moved decisively **toward a single centralised authoritative register** — one platform, one interface set, one register, one disclosure dataset, operated by the Exchange. Where this market has chosen a technology direction, it is centralised integration plus cryptographic signatures plus divisibility, not distributed ledger. Any proposal pitching a distributed bill ledger to a Chinese participant should be assessed against the fact that the regulator's entire architecture rests on a single system of record with a statutory role.

---

## 10. Banking Practice

### 10.1 Acceptance, Discounting and the Trading Book

A bank's bill business has three distinct desks, and confusing them is the most common operating-model error.

**Acceptance (承兑) origination.** The bank accepts drafts drawn on it by its corporate clients, in effect lending its own credit to the client's supplier. Economically this is a contingent liability and a fee business; regulatorily it is subject to Article 13 (inside the unified credit-limit framework), Article 11 (strict review of the drawer's real trade relationship and acceptance risk) and Article 24 (the 15%-of-total-assets acceptance ceiling and 10%-of-deposits margin ceiling, effective 1 January 2024). Acceptance generates fee income and deposit balances from the required margin — and consumes balance-sheet capacity against a hard regulatory ceiling.

**Discounting (贴现) desk.** The bank buys drafts from holders, taking acceptor credit risk onto its balance sheet. This is a lending business in substance: it consumes credit limits, appears in the loan book family, and is subject to the Article 28 mandatory disclosure check and the Articles 15–16 documentary requirements. Its return is the discount spread over funding cost.

**Interbank trading book (转贴现).** The bank buys and sells already-discounted bills. This is a trading and liquidity business: it manages the maturity ladder, can sell down concentration, and can buy short-dated bank-credit exposure when yield exceeds funding cost. It is where a bank manages the same asset it originated, and where the Exchange's central registration and settlement earn their keep.

### 10.2 Credit Limits, Pricing and Credit Quota

**Credit and limit framework.** The primary limit is the **acceptor**, not the client. A bank that limits bill exposure by drawer identity misses that a discounted commercial-accepted draft in its book is an exposure to a third party it did not assess — and that acceptance-side exposure to its own clients is a separate contingency book under the same ceiling. The practical framework is three books: acceptance contingency (client-driven, ceiling-constrained), discounting asset (acceptor-driven, credit-constrained), and trading position (liquidity and market-risk constrained).

**Pricing and profitability.** Pricing is a discount rate to maturity. Its floor is the funding cost of the liability funding the asset, plus capital and operating cost; its ceiling is what the market will bear given the acceptor's credit and comparable instruments. Because discounting assets are short-dated and often bank-guaranteed, they are frequently funded from short-tenor wholesale or deposit funding, which makes the business acutely sensitive to the funding curve's slope — a compression at the short end can erase the spread entirely. This is the mechanism behind reported market behaviour in which discount yields compressed to low single-figure percentages while several mid-sized banks reduced discounting balances and large banks increased theirs.

**Why discounting absorbs credit quota.** In Chinese regulatory practice, bills that a bank discounts are generally treated as credit extended, so they consume the bank's lending capacity and its regulatory loan-related metrics. That has two consequences. First, a bank with surplus capacity and weak loan demand finds short-dated bills an efficient way to use quota, which is why bill discounting has historically behaved as a **quota-filler** — bid up precisely when corporate loan demand is weak, and abandoned when better lending opportunities appear. Second, a bank near its capacity must choose between bills and loans on relative return, which is why the instrument functions as a marginal asset at the margin of the credit cycle. The market-size statistics bear this out: bill activity routinely rises when real-economy loan demand softens.

### 10.3 Accounting, Reporting and Margin-Thin Versus Fee-Rich

**Accounting and regulatory-report classification.** Three classification questions matter, and this guide states them as questions because it did not verify the governing accounting standards line by line. First, whether a discounted bill is measured as a loan-type financial asset at amortised cost or can be designated at fair value through profit or loss — which determines whether the bank's P&L shows a spread or a mark. Second, whether an acceptance sits on balance sheet or as a contingency, since the Article 24 ceiling is computed on the acceptance book regardless. Third, whether the bank's discounted-bill balance is reported inside the credit-to-the-real-economy aggregate, which is a statistical-classification question treated in 11.2. The correct posture for an implementer is to treat these as configuration decisions owned by finance and regulatory reporting, not assumptions a technologist may set.

**Margin-thin versus fee-rich.** The honest division of the business is this. **Discounting is margin-thin.** The asset is short-dated, low-risk at the bank-accepted end, and priced against money-market rates; the spread is small, the balance must be large, and profitability depends on funding cost and turnover rather than on spread. Reported market behaviour — yields compressing toward the low single figures, mid-sized banks retreating, large banks accumulating — is consistent with a business that is viable at scale and marginal at small size. **Acceptance is fee-rich by comparison.** The bank lends its credit, charges an acceptance fee, and in many cases holds margin deposits; the revenue is not a spread but a fee plus a deposit relationship, and the constraint is the regulatory ceiling rather than the market spread. **Trading is fee-rich.** Interbank bill trading generates turnover-based revenue with limited balance-sheet footprint. **Brokerage is fee-rich and separately regulated.** The operating conclusion follows: a small bill operation cannot win on discounting economics; it can win on acceptance and trading, which are relationship and flow businesses.

---

## 11. Market Context and Wider Significance

### 11.1 What Bill Activity Signals

Bill-market activity is one of the more informative real-time signals of Chinese real-economy credit conditions, precisely because the instrument sits at the point of trade settlement. Three signal channels operate.

**The level of acceptance volume** measures how much real trade is being settled on deferred terms rather than cash or open account. Rising acceptance volume alongside soft loan demand typically indicates trade is happening but buyers are stretching payment terms — a stress signal dressed as an activity signal, and the direct mechanism behind the SME payment-extension problem that the 2025 supply-chain notice was written to address.

**The ratio of bank-accepted to commercial-accepted volume** measures how much of the deferred-payment load is being carried by bank credit rather than corporate credit. A shift toward bank-accepted paper is a flight to quality and reduces the market's credit content; a shift toward commercial-accepted paper indicates corporate credit is being accepted on its own terms.

**The spread between outright discounting rates and PBOC's rediscount rate** measures how much the market believes it needs over the central bank's backstop. Widening spreads with stable volumes indicate rising perceived acceptor credit risk; compressed spreads with rising volumes indicate quota-filling rather than genuine credit demand.

### 11.2 Bills in the Credit and Social-Financing Aggregates

Bill financing enters China's monetary statistics in a specific and frequently misread way. The social-financing-scale (社会融资规模) series includes a component for **undiscounted bank-accepted drafts (未贴现银行承兑汇票)**. The reporting nuance is the direction of the definition: the series counts the portion of bank-accepted paper that has **not** been discounted by a bank, because once a bill is discounted it becomes part of the bank-loan aggregate and counting it again as undiscounted paper would double count.

The consequences are counter-intuitive and worth stating plainly:

- **Discounting a bill can reduce reported social-financing stock.** When a bank discounts previously undiscounted paper, the undiscounted-bank-acceptance component falls while the loan component rises; if the loan is classified within the real-economy loan aggregate the net effect is neutral, but if any discrepancy exists in classification the net reported figure can move.
- **A rise in the undiscounted component is not necessarily good news.** It can mean banks are declining to discount — i.e. that the paper is not wanted.
- **PBOC's published data illustrate the scale.** The 2025 financial statistics report put social-financing stock at RMB 442.12 trillion at end-2025, up 8.3% year on year, and the undiscounted bank-accepted draft balance at RMB 2.15 trillion, down 0.3%; the full-year social-financing increment was RMB 35.6 trillion, with the undiscounted bank-accepted draft component contributing an increase of RMB 112 billion, RMB 340.5 billion more than in the prior year. Separately, PBOC's 2025 financial-market statistics put commercial draft acceptance volume at RMB 42.7 trillion, discounting volume at RMB 33.9 trillion, and acceptance balances at RMB 21.2 trillion (+7.2%), with the discounting figure reported as RMB 33.9 trillion.

The key comparison for a reader: **acceptance volume of RMB 42.7 trillion against a social-financing stock of RMB 442.12 trillion**, and **acceptance balances of RMB 21.2 trillion against an undiscounted-bank-acceptance component of RMB 2.15 trillion**. The gap between the second pair is the point: the overwhelming majority of outstanding bank-accepted paper has been discounted and therefore sits inside the loan aggregate, not the undiscounted component.

### 11.3 SME Tool Versus Shadow-Financing Vehicle

The policy tension in this market is genuine and unresolved, and it explains most of the regulatory movement since 2020.

**The SME-financing motive.** Bills are the SME financing instrument of choice for structural reasons: a small supplier can be paid in a negotiable instrument that is transferable by endorsement, discountable at a bank, and, with divisibility, splittable to match its payment needs. The standardised note was created explicitly to serve SME financing and supply-chain development; the supply-chain bill platform was launched explicitly to serve SMEs and normalise supply-chain finance; and the 2025 notice's bill provisions — expanded supply-chain bill application, limited-recourse pilots, securitisation pilots — continue that motive. Independent figures put the SME share of bill holders at a large majority, which is consistent with the instrument's design purpose.

**The shadow-financing motive.** The same instrument can serve the opposite purpose: an anchor enterprise can extend its payment terms by paying suppliers in acceptances rather than cash, effectively obtaining interest-free supplier financing and, if its acceptances are discounted at a rate below its own borrowing cost, arbitraging its credit standing to raise money outside the loan book. Financing-only paper, unlicensed intermediaries, opaque pricing and, latterly, the parallel receivables-certificate market are all expressions of this second motive. The 2025 notice's language — reducing the occupation of SME funds and late payment of accounts — is aimed directly at it.

**The policy resolution so far** has been to keep the instrument and shrink the abuse: cap the tenor; cap the acceptance balance; mandate disclosure and publish the delinquents; require a real trade background; license the intermediaries; retire the paper; consolidate onto one platform; and bring the adjacent unregulated substitute inside the perimeter. The direction is clear even though the tension is not resolved — and the honest summary is that China is deliberately using a regulated negotiable instrument as an SME financing tool while systematically narrowing the space in which the same instrument can be used as a shadow-financing vehicle.

---

## 12. The Cymbal Bank Worked Example

### 12.1 Scope, Business Case and Participation Design

> ⚠ **This section is illustrative design fiction.** Cymbal Bank is a fictional institution and every number below is a **clearly-labelled illustrative figure**, constructed to show the shape of an analysis, not a market data point. No real bank is used as the persona, and no real bank's system is described. All regulatory references are to the real, verified instruments cited elsewhere in this guide.

**Setting.** Cymbal Bank's China-facing corporate banking arm serves mid-sized exporters and their domestic suppliers out of Shanghai. Its clients increasingly ask to be paid in, and to discount, Chinese commercial drafts; the bank currently refers them to a third-party bank and earns nothing on the flow.

**Business case.** Three revenue lines justify building direct capability. *Acceptance* is fee-rich and relationship-deepening, but consumes capacity against the 15%-of-total-assets ceiling and requires the bank to be a licensed acceptor with bill acceptance in scope — which Cymbal's China entity would need to hold. *Discounting* is margin-thin but anchors the client relationship and uses short-dated assets to fill capacity when loan demand is soft. *Interbank trading* is fee-rich with limited balance-sheet footprint but requires a credit and settlement framework with counterparties. Referral generates none of the three and leaves the client's experience with another bank.

**Participation design.** Cymbal elects **direct connection** rather than client mode. Rationale: the bank's volume justifies straight-through processing; client mode would place a manual keying dependency between Cymbal and its clients' maturity events, and dishonour notifications are now a regulated obligation with a defined effectiveness rule (see 6.4). The cost is a permanent interface-maintenance obligation against the Exchange's specification, plus a testing relationship. **Rejected alternative:** client mode, on the grounds that a bank whose clients hold bills to maturity cannot afford to be the party that misses a presentation deadline (see 12.4).

### 12.2 Instrument Selection and the Credit Consequence

Cymbal will support all three acceptance species but with different limits and treatment.

| Species | Cymbal posture | Credit consequence |
| --- | --- | --- |
| 银行承兑汇票 (bank-accepted) | Buy actively as a trading and discounting asset | Exposure to the accepting bank; short-dated bank credit; tight spread; the natural liquidity asset |
| 财务公司承兑汇票 (finance-company-accepted) | Selectively, per-group limits | Quasi-corporate exposure tied to the parent group's financial health; requires separate limit architecture rather than a bank-equivalence assumption |
| 商业承兑汇票 (commercial-accepted) | Originate only against an approved-acceptor list; no general buying | True corporate credit; wide spread; requires per-acceptor limits, mandatory disclosure verification and overdue-list screening |

The critical design decision is the **acceptor limit architecture**. Cymbal models the acceptor as the primary obligor and the drawer and major endorsers as secondary obligors on recourse, rather than modelling the client as the counterparty. The illustrative reason: a commercial-accepted draft discounted for Client A is an exposure to Acceptor B, a company Cymbal has never met, which Client A did not underwrite and may not have assessed. Any limit model keyed on the client systematically understates this.

### 12.3 Controls, Integration and Reporting

**Mandatory controls, in the system rather than in a manual.**

1. **Disclosure verification gate.** No discount transaction proceeds unless the acceptor's disclosure record exists and its recorded particulars are consistent with the presented instrument — a blocking control implementing Article 28, with the check result stored as audit evidence.
2. **Overdue-list screening.** Daily automated screening of all held and proposed acceptors against the Exchange's overdue and persistently-overdue lists, with automatic escalation and a hard block for listed acceptors.
3. **Acceptance-ceiling monitoring.** Real-time tracking of Cymbal's own acceptance balance against 15% of total assets and its margin balance against 10% of absorbed deposits, with a soft warning at an illustrative 85% of the ceiling and a hard block thereafter.
4. **Trade-background evidence capture.** Structured capture of the documents evidencing the real transaction at acceptance and discount, stored against the instrument, because Article 37 makes the institution — not only the client — liable for a fabricated relationship.
5. **Signature and certificate monitoring.** Certificate-expiry alerting, dual control over signing keys, and bank-side verification rather than reliance on the client channel alone.

**Integration and reconciliation.** The register is treated as the system of record for the instrument; Cymbal's bill store is a derived view. Event-driven reconciliation proves, on every lifecycle message, that the instrument leg and the CNAPS funds leg both completed; scheduled reconciliation compares Cymbal's store against the register instrument by instrument with exception classification, because Article 80 makes material divergence sanctionable. Integration points: customer and account master (single enterprise identity across bill platform, lending and AML); the limit system (acceptor-keyed); the general ledger (discounted bills as assets, acceptance as contingency); and regulatory reporting for acceptance and discounting volumes, ceiling metrics and dishonour events.

**Accounting and reporting treatment.** Cymbal books discounted bills within its lending-asset family and reports acceptance as a contingency; it treats the measurement classification of discounted bills and the treatment of the discounted balance in the credit-to-the-real-economy aggregate as **finance-owned configuration decisions, not engineering assumptions** (see 10.3 and 11.2).

### 12.4 Illustrative Economics, Failure Modes and Recommendation

> ⚠ **Every figure in this table is illustrative design fiction, not market data.** They are chosen to show relationships and sensitivity, not to represent any real bank or market outcome.

| Line | Illustrative annual value | Note |
| --- | --- | --- |
| Acceptance fee income | RMB 12.0m | On an illustrative RMB 4.0bn acceptance book at a 30bp fee |
| Acceptance-related margin deposits | RMB 400m | At an illustrative 10% margin — at the Article 24 ceiling |
| Discounting net interest income | RMB 6.0m | On an illustrative RMB 3.0bn average discounted balance at a 20bp net spread |
| Interbank trading revenue | RMB 4.5m | Turnover-based, limited balance-sheet footprint |
| Build cost (one-off, two years) | RMB 18.0m | Interface, signing infrastructure, bill module, reporting, testing |
| Run cost (annual) | RMB 5.0m | Includes interface maintenance against specification change |
| Illustrative payback | ~2.8 years | Driven almost entirely by fee lines, not by the discount spread |

**What the numbers are saying.** The discounting line, despite the largest asset footprint, contributes the least. The acceptance and trading lines carry the economics. A build justified on discounting spread alone would fail the illustrative payback test; a build justified on acceptance fees, margin deposits and trading flow passes.

**Failure modes, ranked.**

1. **Maturity and presentation failure.** A bill presented late, or not presented, transfers a loss to Cymbal's client. Mitigation: a maturity alerting regime with an owner per instrument, and direct-connect straight-through processing.
2. **Funds-leg failure.** Insufficient clearing funds at the moment of settlement, after the instrument leg has committed — the exact failure the amended Article 80 sanctions. Mitigation: the funds-leg pre-funding position as a first-class operational control with a hard pre-settlement check.
3. **Register divergence.** Cymbal's book disagrees with the central register, which is itself a sanctionable state. Mitigation: scheduled instrument-level reconciliation with a tested exception workflow.
4. **Acceptor concentration.** Cymbal accumulates correlated commercial-accepted exposure through one anchor enterprise's supplier chain. Mitigation: acceptor limits plus anchor-group aggregation, and screening of split-transfer layers.
5. **Interface drift.** The Exchange changes its specification and Cymbal's direct connection fails. Mitigation: interface maintenance funded in the run cost, and a tested client-mode contingency procedure.
6. **Security-fraud exposure.** A forged digital signature or compromised signing key on the client side. Mitigation: bank-side signature verification and certificate lifecycle control.

**Recommendation.** Cymbal should build direct-connect capability, but should **sequence it around fee income rather than discount spread**: launch acceptance and interbank trading first (both fee-rich, both limited balance-sheet footprint), and treat discounting as a relationship-preserving service rather than a profit centre. It should not build client-mode capability as a primary route, and it should treat the disclosure-verification gate and overdue-list screening as go-live blocking requirements rather than phase-two enhancements.

**What Cymbal could not establish.** Which rate series the Exchange publishes as its official benchmarks, and therefore what a defensible internal curve should reference; whether foreign-invested bank entities face any additional qualification requirements for the acceptance licence it would need; the accounting measurement classification the group auditor will accept for discounted bills held for both collection and sale; and whether the supply-chain bill route is open to a bank of Cymbal's position in the anchor-enterprise chains it serves.

---

## 13. Claims Audit

| Claim | Verdict | Source | Date | Notes |
| --- | --- | --- | --- | --- |
| Maximum tenor is 6 months from issue to maturity — changed from 1 year | ✅ verified | 《商业汇票承兑、贴现与再贴现管理办法》 第二十五条, text at gov.cn | 令 issued 18 Nov 2022; effective 1 Jan 2023 | Primary text read directly; the pre-2023 1-year electronic tenor is itself from secondary sources (next row) |
| The pre-2023 electronic draft tenor was 1 year while paper was capped at 6 months | ⚠ flagged | Baidu Baike entry on the ECDS citing PBOC material; PBOC 2009 press background cited therein | Rule 2009; superseded 1 Jan 2023 | Consistent across secondary sources but the pre-2023 办法 article text was not read at primary source |
| The regulation is 令〔2022〕第4号, adopted 24 Aug 2022, signed 11 Nov 2022 by 易纲 and 郭树清, published 18 Nov 2022, effective 1 Jan 2023 | ✅ verified | gov.cn gazette text of the order | As dated | Full text read |
| Article 24 caps acceptance balances at 15% of total assets and margin at 10% of deposits, effective 1 Jan 2024 | ✅ verified | 令〔2022〕第4号 第二十四条 and 第四十一条 | Article 24 deferred to 1 Jan 2024 | Full text read; Article 41 is the deferral provision |
| The 2022 order repealed the 1997 interim measures and the 2001 PBOC notice | ✅ verified | 令〔2022〕第4号 第四十二条 | Effective 1 Jan 2023 | Full text read; repeals 银发〔1997〕216号 and 银发〔2001〕236号 |
| 供应链票据 is legally an electronic commercial draft | ✅ verified | 令〔2022〕第4号 第三条 | Effective 1 Jan 2023 | Primary text read |
| The ECDS ran from 28 Oct 2009 and was rolled out nationwide from 28 Jun 2010 | ⚠ flagged | Baidu Baike ECDS entry citing PBOC 2009 press background; the parent reconnaissance corroborates | 2009–2010 | Not confirmed at a PBOC primary page in this session |
| The ECDS stopped external service on 27 July 2024 with a single-batch migration | ✅ verified | Exchange notice 票交所发〔2023〕82号, dated 29 Dec 2023, quoted in full by Chinese bill-industry press; corroborated by several bank client notices | Notice 29 Dec 2023; event 27 Jul 2024 | Notice text quoted extensively; bank announcements corroborate the date |
| 《电子商业汇票业务管理办法》 was originally 中国人民银行令〔2009〕第2号, published 16 Oct 2009 | ⚠ flagged | Baidu Baike and secondary legal-database summaries; the amending order references the original number | 2009 | The amending order (令〔2024〕第5号) does confirm the original is 令〔2009〕第2号 |
| The 办法 was amended in 2024, effective 1 Jan 2025, changing Article 80 | ✅ verified | 《中国人民银行关于修改部分规章的决定》, 中国人民银行令〔2024〕第5号, full text at gov.cn gazette | Adopted 3 Sep 2024, promulgated 22 Oct 2024, effective 1 Jan 2025 | Full text read, including the Article 80 wording |
| The Shanghai Commercial Paper Exchange opened on 8 Dec 2016 | ✅ verified | PBOC/press coverage (人民网, 央广网, 中新网), Wikipedia, Shanghai municipal government page | 8 Dec 2016 | Multiple independent sources; 成立 vs 开业 wording varies (see 3.3) |
| PBOC issued 《票据交易管理办法》 on 6 Dec 2016, designating the Exchange | ⚠ flagged | 人民网 report of the Exchange's opening; Wikipedia | 6 Dec 2016 | Corroborated by two sources but the rule text was not read |
| The new-generation system went live 3 Jun 2022; the first member batch on 20 Aug 2022 | ✅ verified | Chinese bill-industry press reporting the Exchange's engineering plan; 上海证券报 / Reuters-line report; Shanghai municipal financial office page | Jun–Aug 2022 | Two distinct events; the apparent conflict is a level-of-detail artefact (see 3.4) |
| The first batch was 51 institutions (41 banks + 10 finance companies) | ✅ verified | 上海证券报 via cnstock; Shanghai municipal financial office | Aug 2022 | The 23 commercial + 18 village bank breakdown sums to the same 41 banks |
| 标准化票据 regime: 公告〔2020〕第6号, issued 24 Jun 2020, effective 28 Jul 2020; first-day creation 14 products / RMB 1.144bn | ✅ verified (regime) / ⚠ flagged (volume) | gov.cn gazette text for the regime; bill-industry portal for the volume | 2020 | Regime text read in full; the first-day volume rests on a single secondary source |
| The standardised note market is dormant/suspended | ⚠ flagged | Bill-industry retrospection published 2024, titled "three years since the standardised note was suspended" | 2024 | No formal suspension notice located; characterised as dormancy, not a verified regulatory suspension |
| 供应链票据 platform trial launch 24 Apr 2020 under 票交所发〔2020〕58号 | ✅ verified | Exchange notice quoted in full by financial press; 新浪财经 / 上海证券报 / 新华社经济参考报 coverage | 24 Apr 2020 | Multiple independent sources |
| Divisibility minimum unit is RMB 0.01, with up to 500 splits | ⚠ flagged | Bill-industry press and platform vendor material | 2022–2024 | The RMB 0.01 unit is consistently reported; the 500-split limit is single-sourced |
| Disclosure regime: 公告〔2020〕第19号, effective 1 Aug 2021 | ✅ verified | Text of 中国人民银行公告〔2020〕第19号 reproduced by 中国银行业协会 and Shanghai municipal financial office | Issued Dec 2020; effective 1 Aug 2021 | Article 10 fixed the effective date; issue date reported as 23 Dec 2020 by one source |
| 4,226 acceptors on the overdue list (774 persistently overdue) as at 31 Dec 2025 | ✅ verified | Exchange 《持续逾期名单》, published 9 Jan 2026, reported by multiple outlets | Published 9 Jan 2026 | The 4,226 is the list total; the 774 is the subset meeting the persistent-overdue test (3+ overdue events, 1 Jul–31 Dec 2025, with a month-end overdue balance or an overdue event that month) |
| Acceptance volume 2025 = RMB 42.7tn; discounting = RMB 33.9tn; acceptance balances = RMB 21.2tn (+7.2%) | ✅ verified | PBOC 2025 金融市场运行情况, reported by 第一财经 and 新浪财经/格隆汇 | PBOC release reported Feb 2026 | PBOC statistical release, not an independent estimate |
| Social-financing stock = RMB 442.12tn (+8.3%) at end-2025; undiscounted bank-accepted drafts = RMB 2.15tn (−0.3%) | ✅ verified | PBOC 2025年金融统计数据报告, 15 Jan 2026 | 15 Jan 2026 | PBOC statistical release |
| 票据法: adopted 10 May 1995, amended 28 Aug 2004 | ✅ verified | npc.gov.cn text of 中华人民共和国票据法 (修正); gov.cn gazette of the amending decision | 1995; 2004 | Full text read at npc.gov.cn |
| 票据法 is NOT in the NPC Standing Committee 2025 legislative work plan | ✅ verified | 全国人大常委会2025年度立法工作计划, full text examined | 2025 | A negative finding established from the full plan text; 金融法, 金融稳定法, 中国人民银行法, 银行业监督管理法, 商业银行法 are listed |
| PBOC's H2 2025 work conference called for accelerating 票据法 amendment | ⚠ flagged | Market retrospective in Chinese financial press | Dec 2025 | Secondary report of an official meeting statement; not confirmed at a PBOC primary page |
| 银发〔2025〕77号: dated 26 Apr 2025, effective 15 Jun 2025, two-year transition for receivables certificates | ✅ verified | gov.cn State Council gazette text of the six-department notice | Dated 26 Apr 2025; effective 15 Jun 2025 | Full text read; Article 21 and the closing transition paragraph |
| Receivables certificate term cap: in principle ≤6 months, max 1 year | ✅ verified | 银发〔2025〕77号 第十一条 | Effective 15 Jun 2025 | Full text read |
| PBOC rediscount rate held at 1.75% through 2025 | ⚠ flagged | Chinese financial-press market retrospective | Dec 2025 | Single secondary source; not asserted in the body as verified |
| Mid-sized banks reduced discounting balances while large banks increased theirs; yields fell to ~1.2% | ⚠ flagged | Aggregation commentary on Chinese finance portals | 2025 | Commentary aggregation, not an official dataset; used only as directional colour |
| Property/construction sector dominates the overdue lists | ⚠ flagged | Industry press reporting Exchange sectoral list slices | 2025 | Consistent with the Exchange publishing sectoral slices, but not independently verified here |
| Interbank rediscounting benchmark is quoted as 国股银票转贴现利率 | ⚠ flagged | Market commentary | Ongoing | The Exchange's exact published rate series could not be verified (see 14) |
| No live, significant blockchain-based Chinese bill platform exists | ⚠ flagged | Negative finding from this guide's primary-source review | 2026 | An absence-of-evidence finding, not proof of absence |

---

## 14. What Could Not Be Verified

Each item below is a gap this guide refused to fill by inference.

1. **The Exchange's published rate series.** Which benchmark bill rates the Exchange publishes, under what names, at what frequency and on what methodology, could not be established. Market commentary quotes a state-owned/joint-stock bank bill rediscounting rate, but the official series identity and specification remain unverified. This directly limits section 5.2 and the Cymbal design in 12.4.
2. **A formal suspension notice for 标准化票据.** The instrument is described in industry commentary as suspended and no active primary market was found, but no regulatory or Exchange notice of suspension was located. The status is therefore reported as dormancy, not as a verified regulatory action.
3. **A primary text of the pre-2023 electronic-draft tenor rule.** The one-year electronic tenor is consistent across secondary sources but was not confirmed from the 2009 rule's own article text.
4. **The 2009–2010 ECDS launch and rollout dates at a PBOC primary page.** The 28 October 2009 operation date, the 28 June 2010 nationwide rollout and the reported initial participant count are corroborated by encyclopedic and press material citing PBOC background, not confirmed at a PBOC page in this session.
5. **The mandatory-electronic amount thresholds.** The 2017 (above RMB 3 million) and 2018 (above RMB 1 million) thresholds are taken from an encyclopedia entry's citation trail and were not confirmed from the originating regulatory notice.
6. **Named corporate commercial-acceptance defaults.** No specific company's bill default could be documented at primary source, so no company is named. The aggregate overdue data published by the Exchange is used instead.
7. **The bill-volume share of the market attributable to supply-chain bills and to standardised notes.** No reliable published split was located.
8. **The exact scale of the receivable-certificate stock under rectification.** Press reporting gives trillions of renminbi and hundreds of platforms; no official statistic was found.
9. **Whether any foreign-invested bank entity faces additional qualification requirements** for the bill-acceptance licence and for Exchange participation.
10. **The 票据法 amendment's actual content and timetable.** Only a secondary report of a PBOC meeting statement was found; the legislative plan does not list the law.
11. **Any live, market-significant blockchain-based bill infrastructure.** No verifiable operational platform was located.
12. **Whether the specific fraud cases of the early 2010s can be sourced at primary record.** They are widely discussed but were not documented here, so no case is named.
13. **The split-transfer layer limits and their enforcement practice** under the 2025 notice's Article 12.
14. **A complete data series on acceptance ceilings actually binding in practice.** Article 24 is verified as a rule; how frequently it binds individual institutions was not established.
15. **The precise 2026 policy calendar for the bill market.** The most recent verified developments are the 15 January 2026 relending/rediscount announcement and the 30 January 2026 Exchange disclosure-optimisation notice; no further material change was located for the remainder of 2026 to the guide's last-updated date, and that absence is reported as an absence rather than as a quiet period.

---

## 15. Glossary

| Chinese | English | Notes |
| --- | --- | --- |
| 票据法 | Negotiable Instruments Law of the PRC | Adopted 10 May 1995; amended 28 Aug 2004 |
| 商业汇票 | Commercial draft | The bill used in real-economy settlement and short-term financing |
| 银行承兑汇票 (银票) | Bank-accepted draft | Accepted by a PRC bank or rural credit cooperative |
| 商业承兑汇票 (商票) | Commercial-accepted draft | Accepted by a non-bank legal person; corporate credit |
| 财务公司承兑汇票 | Finance-company-accepted draft | Accepted by an enterprise-group finance company |
| 电子商业汇票 / 电票 | Electronic commercial draft / e-bill | The electronic form of the instrument |
| 电子商业汇票系统 (ECDS) | Electronic Commercial Draft System | Ran 2009–2024; operations transferred to the Exchange on 2017-10-09; retired 2024-07-27 |
| 新一代票据业务系统 | New-Generation Bill Business System | The Exchange's successor platform; system go-live 3 Jun 2022 |
| 中国票据业务系统 | China Bill Business System | The name used in the Exchange's own migration notice for the successor |
| 中国票据交易系统 | China Bill Trading System | The Exchange's trading system, merged into the new-generation platform |
| 上海票据交易所 (票交所) | Shanghai Commercial Paper Exchange | Established and opened 8 Dec 2016; the market's infrastructure operator |
| 票据市场基础设施 | Bill-market infrastructure | The regulated operator role held by the Exchange |
| 票据交易管理办法 | Measures for the Administration of Bill Trading | Issued 6 Dec 2016, designating the Exchange |
| 承兑 | Acceptance | The payer's unconditional promise to pay at maturity |
| 出票 / 出票人 | Drawing (issue) / drawer | Creation of the instrument; the drawer issues it |
| 承兑人 | Acceptor | Promises to pay; the credit obligor |
| 持票人 | Holder | Current owner of the instrument |
| 背书 / 被背书人 | Endorsement / endorsee | Transfer of rights, creating joint liability; the endorsee receives by endorsement |
| 贴现 | Discounting | Sale of an unmatured draft to a licensed lender |
| 转贴现 | Interbank rediscounting | Bank-to-bank trading of discounted drafts |
| 再贴现 | Central-bank rediscounting | PBOC discounting eligible drafts; a monetary-policy tool |
| 质押 / 保证 | Pledge / guarantee | Draft encumbered as collateral; third-party guarantee of the obligation |
| 提示付款 / 拒付 | Presentation for payment / dishonour | Demand at or after maturity; recorded refusal to pay |
| 追索 / 追索权 | Recourse / right of recourse | Claim against acceptor, drawer and prior endorsers |
| 票据经纪机构 | Bill brokerage institution | Licensed broker; segregated from proprietary business |
| 票据中介 | Bill intermediary | Colloquial term covering licensed and unlicensed intermediation |
| 票据池 / 票据包 | Bill pool / bill package | Aggregated client bills for financing; a divisible package of standard-amount units |
| 等分化 / 票据拆分 | Equalisation (divisibility) / bill splitting | Standard-unit issuance enabling splitting down to RMB 0.01 |
| 标准化票据 | Standardised note | Divisible beneficiary certificate over a pool of drafts |
| 存托机构 | Depository institution | Creates and manages the standardised note |
| 原始持票人 | Original holder / originator | Endorses drafts into the pool |
| 供应链票据 / 供应链票据平台 | Supply-chain bill / supply-chain bill platform | Electronic commercial draft issued via the Exchange's platform; launched in trial 24 Apr 2020 |
| 应收账款票据化 | Bill-isation of accounts receivable | The policy framing for supply-chain bills |
| 应收账款电子凭证 | Electronic receivables certificate | Now regulated by 银发〔2025〕77号 |
| 供应链信息服务机构 | Supply-chain information service institution | The regulated role for supply-chain platforms |
| 真实交易背景 / 融资性票据 | Real trade background / financing-only draft | The underlying-transaction requirement; a draft with no real trade is prohibited |
| 信息披露 | Disclosure / information disclosure | The mandatory acceptor-disclosure regime |
| 商业汇票信息披露操作细则 | Operating rules for commercial draft disclosure | The Exchange's disclosure rulebook |
| 持续逾期 | Persistent overdue | The Exchange's watch-list criterion for acceptors |
| 承兑余额 / 贴现余额 | Acceptance balance / discounting balance | Outstanding accepted amount (capped at 15% of total assets); outstanding discounted amount |
| 承兑发生额 / 贴现发生额 | Acceptance volume / discounting volume | Flow measures for a period |
| 未贴现银行承兑汇票 | Undiscounted bank-accepted draft | A component of the social-financing-scale series |
| 社会融资规模 | Social financing scale | China's broad credit aggregate |
| 票款对付 | Delivery-versus-payment (DVP) | Simultaneous delivery of instrument and funds |
| 直连接入 / 客户端 | Direct connection / client mode | Member connects its own system; or keys business in manually on the client's mandate |
| 间接参与者 | Indirect participant | A platform participant without direct connection |
| 接入机构 | Access institution | Financial institution connecting to the platform |
| 电子签名 / 数字证书 | Electronic signature / digital certificate | Replaces the physical seal; underpins signature validity |
| 伪造票据 / 变造票据 / 私刻印章 | Forged bill / altered bill / forged seal | Fabrication, alteration and counterfeit seal — physical-era typologies |
| 虚假贸易背景 | Fake trade background | Fabricated underlying transaction |
| 一票多卖 | Duplicate financing | Financing the same bill twice |
| 支农支小再贴现额度 | Agriculture and small-business rediscount quota | Structural PBOC facility; quota RMB 740bn after Jun 2023 |
| 国股银票转贴现利率 | State-owned/joint-stock bank bill rediscounting rate | Market-quoted benchmark; the Exchange series is unverified |

---

## 16. Cross-References and Further Reading

**Sibling guides in this repository.**

| Guide | Owns | Relationship to this guide |
| --- | --- | --- |
| [cnaps_guide.md](cnaps_guide.md) | CNAPS/HVPS/BEPS/IBPS, settlement mechanics, funds-leg windows | Upstream of this guide's funds leg; the ECDS hand-off row is quoted in 1.2 |
| [supply_chain_finance_guide.md](supply_chain_finance_guide.md) | The SCF discipline — payables finance, receivables finance, platform models | Contrasted in 8.4; does not cover the Chinese bill |
| [supply_chain_finance_technologies_guide.md](supply_chain_finance_technologies_guide.md) | SCF platform technology | Relevant to the platform boundary in 9.3 |
| [trade_finance_guide.md](trade_finance_guide.md) | UCP 600, documentary credits, forfaiting | Owns the *Western* draft sense of fields 42C/42A/D; see 1.3 |
| [trade_finance_systems_guide.md](trade_finance_systems_guide.md) | Trade-finance systems architecture | Comparison frame for 9.1 |
| [financial_infrastructure_guide.md](financial_infrastructure_guide.md) | International RTGS/CSD/CCP comparative frame | Places the Exchange in a wider infrastructure taxonomy |
| [chinese_bank_core_systems_guide.md](chinese_bank_core_systems_guide.md) | Chinese core-banking landscape | Integration context for 9.3 |
| [core_banking_processes_guide.md](core_banking_processes_guide.md) | End-to-end banking process detail | Process context for 10.1 |
| [banking_limits_domain_guide.md](banking_limits_domain_guide.md) | Limit and credit-quota mechanics | Underpins the acceptor-limit design in 10.2 |
| [basel_regulatory_capital_guide.md](basel_regulatory_capital_guide.md) | Capital treatment frame | Relevant to the acceptance/asset capital questions in 10.3 |
| [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) | Fraud detection at scale | Analytics context for the typologies in 7.3 |

**Primary sources consulted.** PBOC regulatory texts and statistical releases (pbc.gov.cn); the Chinese government portal and State Council gazette (gov.cn), including the full texts of 令〔2022〕第4号, 令〔2024〕第5号, 公告〔2020〕第6号 and 银发〔2025〕77号; the NPC's text of 中华人民共和国票据法 (npc.gov.cn); the 全国人大常委会2025年度立法工作计划; the Shanghai Commercial Paper Exchange (shcpe.com.cn) and its notices, disclosure lists and service guides as reported; 国家金融监督管理总局 (nfra.gov.cn); and Chinese financial press — 第一财经, 新浪财经, 上海证券报, 财新, 新华社经济参考报, 中国银行业协会 and bill-industry portals including 汇票圈 and 票风信息网.

**A note on source quality.** The regulatory facts in this guide rest on primary texts read directly from gov.cn gazettes wherever possible. Operational facts — dates, participant counts, system behaviours — frequently rest on Exchange notices quoted in full by financial press, because the Exchange's own site resisted direct extraction. Market data rests on PBOC statistical releases. Everything remaining is flagged in sections 13 and 14, and a reader who needs a regulatory fact to be certain should go to the gazette text named in the claims-audit row.

---

## 17. Closing Summary

China's commercial draft is best understood as **one instrument with two credits, carried by one centralised register, and regulated by a shrinking perimeter of permissible behaviour.** The instrument is uniform — a negotiable instrument under the 票据法, transferable by endorsement, carrying joint liability on recourse — but its economics split at the acceptance. Bank-accepted paper is money-market paper; commercial-accepted paper is corporate credit wearing an instrument's clothes. Everything about how this market is priced, held and regulated follows from that split, and no amount of electrification changes it.

The infrastructure story is a story of consolidation. The ECDS solved processing by making one register authoritative, and in doing so eliminated the physical fraud typologies — forged bills, altered bills, forged seals, duplicate financing, custody loss — that defined the paper era. The Exchange solved market structure by making one venue the centre of trading, registration, custody, settlement and disclosure. The new-generation system solved the resulting fragmentation by merging both into one platform with one interface set, and added divisibility down to RMB 0.01, which turned the regulated instrument into a genuine competitor to the unregulated receivables certificate. The ECDS stopped external service on 27 July 2024, and its retirement is the clearest marker that this market now runs on a single system of record.

The regulatory story is a story of narrowing. Tenor cut from a year to six months, effective 1 January 2023, for all drafts without distinction. Acceptance balances capped at 15% of total assets and margins at 10% of deposits, effective 1 January 2024. Disclosure made mandatory, published, and enforced by suspending the offender's ability to issue. A real trade background required at acceptance and at discounting, with liability resting on the institution, not only the client. Brokerage confined to licensed institutions with segregated books. And then, in April 2025, the perimeter itself extended outward to swallow the adjacent unregulated substitute, with the Exchange designated to aggregate and monitor it and a two-year transition running to about June 2027.

The risk story has not changed shape, only visibility. Electronification eliminated the fraud that lived in the paper and left the fraud that lives in the transaction: fake trade backgrounds, intermediary conduct, and the credit risk of acceptors who simply did not pay. What the market gained was not less risk but the ability to see it — 4,226 acceptors on the overdue list as at the end of 2025, 774 of them persistently overdue, published and named, with acceptance services suspended for those on the persistent list. That is a young transparency, roughly five years old, and it is the single most valuable structural improvement in the market.

For a bank, the conclusion is unromantic and clear. Discounting is margin-thin, short-dated and capacity-consuming; it is a relationship service and a credit-quota tool, not a profit centre. Acceptance and interbank trading carry the economics. The build is not an interface project but a control project — a register that must be reconciled because divergence is sanctioned, a disclosure check that must block rather than warn, a funds leg that must settle atomically with the instrument leg, and an acceptor limit architecture that must be keyed to the party who actually owes the money. A bank that gets those five things right is a participant in this market. A bank that gets them wrong is holding an instrument it has verified but cannot enforce, against an acceptor it never assessed, on a bill it cannot pay for at maturity — which is the failure mode the entire regime was rebuilt to prevent. The instrument's whole legal design exists to make one promise enforceable, and the market's whole infrastructure exists to tell you whether that promise is worth anything: the accepted draft.
