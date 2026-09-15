# NetsUnion Clearing Corporation (网联清算有限公司) — The Mandated Clearing Channel for China's Third-Party Payment Business

> **Jack Liu Shurui, Solution Architect**
>
> *Companion guides in this repository:* [`banking/cnaps_guide.md`](cnaps_guide.md) covers the
> China National Clearing Center (清算总中心, CNCC) and the high-value/RTGS and bulk retail rails it
> operates; [`banking/ibps_payment_connect_guide.md`](ibps_payment_connect_guide.md) covers the
> Internet Banking Payment System (IBPS, 网上支付跨行清算系统) and the Payment Connect / 网联互联
> style bank-side connectivity work. Read those two for CNCC's and IBPS's own designs; this guide
> treats them only as boundary markers around NUCC.

## How to Read the Evidence Markers in This Guide

Every material claim below carries one of three markers. They are used consistently and they mean exactly what they say:

| Marker | Meaning | What earns it |
|---|---|---|
| ✅ | **Verified** | Confirmed by a primary/official source (PBOC, gov.cn, Xinhua, the operating company's own published material) **or** by two or more independent secondary sources that agree on the fact and the date. |
| ⚠ | **Flagged / inferred** | Single-sourced, drawn from a company-information aggregator or an encyclopaedia entry rather than a primary document, or a reasonable inference drawn from adjacent facts. Treat as *directionally useful, not citable as hard fact* in a compliance file. |
| ❌ | **Rejected** | A claim encountered in research that a stronger source contradicts, or that no source supports on inspection. Listed in the claims audit (§12) so the reader does not re-encounter it. |

Three additional conventions apply throughout:

1. **Reported figures are attributed, not adopted.** NUCC's volume, throughput and availability statistics are the operator's own numbers, most of them released at trade exhibitions or in New Year messages. They are quoted with the announcement date and the venue, and labelled as operator disclosures wherever they appear.
2. **No document number, percentage, date or volume figure in this guide is invented.** Where a regulator's instrument number could not be confirmed in this research pass, the claim is marked ⚠ and the gap is listed in §13.
3. **Institutions are named factually as subject matter.** The People's Bank of China (PBOC, 中国人民银行), UnionPay (中国银联), the PBOC's China National Clearing Center (CNCC, 清算总中心), CNAPS, IBPS, CIPS, Alipay (支付宝, Ant Group) and Tenpay (财付通, Tencent) appear as subject-matter actors. The only bank persona used for worked examples and cost estimates in this guide is **Cymbal Bank**. All Cymbal Bank figures are explicitly labelled illustrative and are fictional.
---

## Table of Contents

1. [Overview and Identity](#1-overview-and-identity)
2. [Origin and Mandate](#2-origin-and-mandate)
3. [The Customer Reserve Funds Reform](#3-the-customer-reserve-funds-reform)
4. [Platform and Technology](#4-platform-and-technology)
5. [Clearing and Settlement Model](#5-clearing-and-settlement-model)
6. [Participants](#6-participants)
7. [Volumes and Scale](#7-volumes-and-scale)
8. [The Other Chinese Retail Clearing Channels, Cross-Referenced](#8-the-other-chinese-retail-clearing-channels-cross-referenced)
9. [Cross-Border Dimension](#9-cross-border-dimension)
10. [Operational, Compliance and Risk Reality for a Bank](#10-operational-compliance-and-risk-reality-for-a-bank)
11. [Cymbal Bank Worked Example](#11-cymbal-bank-worked-example)
12. [Claims Audit](#12-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [Closing Summary](#16-closing-summary)
---

## 1. Overview and Identity
### 1.1 The one-page orientation

If you have never heard of the NetsUnion Clearing Corporation, this is the whole idea in six sentences. In China, a payment app such as Alipay or WeChat Pay is operated by a **non-bank payment institution** (非银行支付机构) — a licensed, supervised firm that is *not* a bank. When a consumer pays a merchant, the money ultimately moves between two bank accounts, and somebody has to clear and settle that move. Until 2017 the payment institutions did that themselves, over private bilateral links to each bank — the **direct-connection** or 直连 model. That model took interbank clearing out of the supervised interbank path and out of the central bank's visibility. In 2017 the PBOC ordered the traffic onto a single, purpose-built, centralised clearing house — NUCC — and in 2018 it forced the customer funds that had been parked at commercial banks into the central bank itself. NUCC is therefore best understood as *the mandated clearing leg of Chinese online retail payments*: a privately incorporated, PBOC-affiliated **licensed clearing organisation** whose shareholders include the incumbent payment institutions, and whose single most important property is that it does not compete with its participants in the payment business at all.
### 1.2 Name, form and identifiers

✅ **Verified identity:**

| Item | Value | Marker / source |
|---|---|---|
| Chinese name | 网联清算有限公司 | ✅ |
| English name | NetsUnion Clearing Corporation, abbreviated **NUCC**; Chinese short form 网联 (NetsUnion) | ✅ Baidu Baike entry 22111484; PBOC recruitment notice |
| Legal form | 其他有限责任公司 (other limited-liability company) | ✅ Baike 22111484 |
| Established | **2017-08-29** (business licence / registration date) | ✅ Baike 22111484; zh.wikipedia; media |
| Registered capital | **RMB 2.0 billion** (注册资本 200000 万人民币 = 20 亿元) | ✅ Baike 22111484 + 企查查 + 爱企查 snippets agree |
| Unified social credit code (统一社会信用代码) | 91110102MA0179UA3A | ⚠ aggregator-level detail (Baike/企查查), consistent across sources |
| Registered address | 北京市西城区北三环中路甲29号院3号楼101号内101室 | ⚠ aggregator-level detail |
| Employees | 494 (2024 figure) | ⚠ Baike citing 天眼查 (2025) |
| Chairman / legal representative (2024–26) | 罗延枫 (Luo Yanfeng) | ⚠ Baike citing 天眼查 and 财经 magazine, 2025 |
| Regulator with oversight | People's Bank of China (PBOC, 中国人民银行) | ✅ |

✅ **Status and character:** NUCC is a **特许清算组织** — a licensed (franchise) clearing organisation — and an enterprise affiliated to (所属企业) the PBOC. The PBOC's own recruitment page describes it in the PBOC's own words as *"为中国人民银行所属企业，是金融基础设施运营机构…业务范围覆盖各类 银行业机构和非银行支付机构，是全球交易笔数最大的零售支付清算机构"* ("an enterprise affiliated to the People's Bank of China; a financial-infrastructure operating institution … its business scope covers all types of banking institutions and non-bank payment institutions; it is the retail payment clearing institution with the largest transaction count in the world"). That text is the single most authoritative one-paragraph characterisation of NUCC obtained in this research pass and it comes from `pbc.gov.cn` (see §15).

NUCC is the operating institution of the **非银行支付机构网络支付清算平台** — the "Non-Bank Payment Institution Online Payment Clearing Platform", known in the market simply as the **网联平台** ("the NetsUnion platform").
### 1.3 Registered business scope

✅ The registered business scope (经营范围), per the company record reproduced by Baidu Baike, includes: **building and operating the national unified clearing system**; **providing funds clearing for the online payment business of non-bank payment institutions**; **coordinating and arbitrating business disputes** (协调和仲裁业务纠纷); providing other supporting services; software development; data processing; computer system services; technology development/transfer/promotion; market research; conference and exhibition services. The dispute-coordination item matters for §5: dispute co-ordination and arbitration is a *chartered* function, not an afterthought.
### 1.4 What NUCC is NOT — four separations

Much of the confusion about NUCC comes from conflating it with its neighbours. The four separations below are the ones that matter in a compliance or architecture document.

| Entity | What it is | How it differs from NUCC |
|---|---|---|
| **The PBOC itself (中国人民银行)** | China's central bank and the banking/payments regulator | NUCC is *supervised by and affiliated to* the PBOC; it is a company, not a regulator. The PBOC issues the mandates (§2, §3) and holds the customer reserve funds (§3); NUCC operates the clearing platform under those rules. |
| **China National Clearing Center (清算总中心, CNCC)** | The PBOC's own clearing arm, operating CNAPS (HVPS/BEPS) and ancillary national systems | CNCC is a PBOC *internal* institution operating the interbank rails; NUCC is a separate incorporated company with external shareholders. They are distinct legal and operational entities — do not file NUCC documents under CNCC. See [`banking/cnaps_guide.md`](cnaps_guide.md). |
| **UnionPay (中国银联)** | The incumbent domestic **bank-card** clearing institution, also a licensed clearing organisation | UnionPay clears card transactions (and participates in the 断直连 migration as an alternative to NUCC, §8); NUCC was built for *bank-account-based online payment business originated by non-bank payment institutions*. They are competitors in scope, peers in status, and now in a formal dialogue mechanism (§8). |
| **The payment institutions themselves (非银行支付机构)** | Alipay/Tenpay and the other licensed payment firms | These firms are NUCC's **participants and part-owners**; they originate and are responsible for the payment business. NUCC does not sell payment services to merchants or consumers, does not hold customer funds, and does not take credit risk on the payment. |

### 1.5 Why a state would mandate a single clearing house

The reform rationale, as reported consistently across the official and specialist press, rests on four arguments (⚠ rationalisation, but each element is documented — see §2.4):

1. **Visibility.** Under direct connection, cross-institution flows inside payment institutions' own ledgers were not visible to the PBOC or to the banks in real time; centralising the clearing leg makes the supervisor see the traffic as it happens.
2. **Fund safety and AML.** With flows outside the supervised path, anti-money-laundering (AML) monitoring, fraud interdiction and detection of misappropriation of customer reserve funds were difficult. NUCC's own reported posture is that traffic on the platform is connected to AML systems and reported in real time. ⚠ (secondary reporting of the design intent)
3. **Level playing field.** Bilateral direct connections gave large institutions a bargaining advantage over small ones on channel pricing and access; a single platform with one price for one interface removes that asymmetry. The 银支付〔2017〕209号 notice was accompanied by exactly this argument in the press: interfaces all lead to NUCC, at a uniform price.
4. **Reduced duplication.** Each payment institution maintaining dozens of bespoke bank links — and each bank maintaining dozens of counterparty links — is duplicated engineering. The "n" to "1" collapse was cited as a cost and complexity reduction for the industry, especially for smaller banks and smaller payment institutions.
### 1.6 Ownership in one paragraph

NUCC exists because the PBOC required it; it is owned because the industry funded it. It was organised by the **Payment & Clearing Association of China (支付清算协会, PCAC)** on a **共建、共有、 共享** ("co-build, co-own, co-share") basis, with non-bank payment institutions subscribing capital alongside PBOC-affiliated institutions. As reported on 2017-08-05 by 经济观察网 and on 2017-08-04 by 移动支付网: **seven PBOC-affiliated institutions — including CNCC (中国人民银行清算总中心), Shanghai Clearing House (上海清算所), the Shanghai Gold Exchange (上海黄金交易所) and the National Association of Financial Market Institutional Investors — together subscribed RMB 760 million, a reported shareholding of 37%**; **PCAC holds 3%, expressly to vote for small and mid-sized payment institutions that are not eligible to hold shares**; **29 third-party payment institutions hold the remaining 63%**; and **Alipay (支付宝) and Tenpay (财付通) each hold 9.61%**. The complete shareholder register is treated in §2.3, including what is *not* verifiable.

---

## 2. Origin and Mandate
### 2.1 The direct-connection model and the problem it created

Before 2017, a licensed non-bank payment institution that wanted to let its users pay from, or be paid into, a bank account had to connect to each bank. The result was a mesh: each large payment institution held bilateral 直连 (direct-connection) links to many banks, negotiated separately, on bilaterally agreed terms. Within that mesh, two things happened at once.

- **The payment institution, not a licensed clearing institution, performed the cross-institution leg.** Money moved between accounts at two different banks without passing a card network, a clearing house, or the PBOC's interbank systems in a way the supervisor could observe per transaction.
- **Customer funds accumulated at commercial banks.** Float from unsettled transactions — the industry's **备付金** (customer reserve funds) — sat in accounts at reserve-holding banks (备付金 银行), in some cases in very large amounts, earning interest for the payment institution.

The supervision argument (⚠ design intent, well documented in the 断直连 literature and in Chinese Wikipedia's summary of the platform's *Background* section) is: *"第三方支付绕开银联的账户系统内结算， 使其客户支付信息等数据，商业银行和央行等均不掌握，由此产生了一定的风险，比如洗钱等"* — third-party payment firms settled inside account systems bypassing UnionPay, so that neither the commercial banks nor the central bank possessed the payment information, creating risks such as money laundering. The same passage states the remedy plainly: NUCC's establishment corrects the irregular conduct of cross-bank clearing by third-party payment institutions, changes the problem of payment institutions connecting to banks in a many-to-many fashion, and lets the PBOC see customer account information; online payment channels no longer connect — and may not connect — directly to banks, and instead reach each bank through the NUCC platform.

A further, often-underweighted element appears in the 2017 reporting on the shareholder structure: *intermediary or "bank-affiliated clearing centre" arrangements* form a second, unofficial wire around the licensed path. 经济观察网 (2017-08-05) describes clearing centres run inside banks that in substance acted as a clearing institution for third-party payment firms, with transaction chains running 商户 → 第三方支付A → 银行系清算中心 → 第三方支付B → 银行 — and explicitly notes these were not licensed clearing institutions, so their downstream institutions were effectively treated as merchant customers of a bank's acquiring business. The same article quotes a person close to the Payment & Clearing Association saying that strictly speaking such arrangements were irregular and would be brought under licensed clearing institutions in future. This is the context in which "licensed clearing organisation" (特许清算组织) became the operative phrase: it excludes both the unofficial bank-run centres and the payment institutions themselves.
### 2.2 The instruments: what was actually issued, with numbers and dates

✅ **银支付〔2017〕209号** — *《中国人民银行支付结算司关于将非银行支付机构网络支付业务由直连模式迁移至 网联平台处理的通知》* — "Notice of the PBOC Payment & Clearing Division on Migrating the Online Payment Business of Non-Bank Payment Institutions from the Direct-Connection Model to Processing on the NetsUnion Platform". Issued **2017-08-04** by the PBOC's **支付结算司** (Payment & Clearing Division).

Note the issuing body and the numbering prefix carefully: the instrument is a **银支付** document — a Payment & Clearing Division notice — *not* a 银发 (PBOC-wide) instrument. It is **not** "银发 〔2017〕209号". Sources that renumber it as 银发209号 are in error.

The two operative requirements, as reported identically across the official and specialist press:

| Requirement | Text (as reported) | Date |
|---|---|---|
| Cut-off for business migration | 自2018年6月30日起，支付机构受理的涉及银行账户的网络支付业务全部通过网联平台处理 — from **2018-06-30**, *all* bank-account-involving online payment business accepted by payment institutions must be processed through the NUCC platform | 2018-06-30 |
| Preparation deadline | 各银行和支付机构应于2017年10月15日前完成接入网联平台和业务迁移相关准备工作 — all banks and payment institutions to complete connection to the NUCC platform and the preparatory work for migration by **2017-10-15** | 2017-10-15 |

✅ Corroborated by at least four independent sources in this pass: 移动支付网 2017-08-04 (`doc_01`, including an image of the document itself); 21世纪经济报道 2017-08-04; people.com.cn 2017-08-07; 中证网 cs.com.cn; the Baidu Baike entry 22167842 dedicated to the notice; and zh.wikipedia's *History* section.

⚠ **Discrepancy to hold open — 2017-10-15 vs 2018-10-15.** One later retrospective report (界面, 2022-vintage page) reproduces the preparation deadline as **2018-10-15**. The overwhelming weight of 2017 reporting — including a photograph of the notice and the contemporaneous national press — says **2017-10-15**. This guide uses 2017-10-15 as the operative date and flags the variant rather than silently resolving it, because if the 2018 date were correct the enforcement chronology in §2.5 would need re-reading. It does not appear to be.
### 2.3 Incorporation, the 45 signing institutions, and the shareholder question

✅ **Incorporation timeline:**

| Date | Event |
|---|---|
| 2016-10-20 | PBOC approves preparatory establishment of the platform (人民银行批准筹建该平台) |
| 2017-03-31 | Platform starts **trial run** (试运行) |
| 2017-06-30 | Platform begins to transfer and clear online payment business for ordinary users' real transaction scenarios |
| 2017-08-02 | **45 institutions sign the founding documents** of 网联清算有限公司 |
| 2017-08-29 | Company obtains its **business licence** — the establishment date |
| 2018-03-21 | Cumulative transactions pass 10 billion (破百亿笔) per company milestone list |
| 2018-03-29 | Shanghai branch established |
| 2019-06-10 | "One-click card binding" (一键绑卡) service launched |
| 2020-07-17 | Co-operation with CIPS to support Greater Bay Area construction (§9) |
| 2023-11-17 | PBOC issues a bank-card clearing licence to the Mastercard JV 万事网联 (§9) |
| 2026-02-06 | High-level exchange talks with UnionPay in Shanghai (§8) |

Sources: Baike entry 22111484 (cited to the company's own 网联大事记 page, 澎湃, 中国网络空间安全协会 and 百家号 items) — hence ⚠ for the milestone list as a whole, and ✅ for the 2016-10-20 / 2017-03-31 / 2017-08-29 triad, which media and Wikipedia independently corroborate.

✅ **The 45 institutions, 2017-08-02.** 经济观察网 (via finance.sina.cn, published 2017-08-05, headline *"清算市场大变局 网联股东明细出炉"*) reports: on 8 August — it states 8月2日 — "包括中国人民银行清算总 中心、财付通、支付宝、银联商务等在内的45家机构和公司签署的《网联清算有限公司设立协议书》", i.e. 45 institutions and companies including CNCC, Tenpay, Alipay and UnionPay Merchant Services signed the NUCC Founding Agreement. The same report states the registered capital is RMB 2 billion, that all parties contributed in cash, and that **capital was paid in three instalments of 50%, 30% and 20%**.

⚠ **Shareholding — what is documented, and what is not.** The aggregate structure is well evidenced by two 2017 sources; the complete register of 45 names with every percentage is **not** obtained (the registry pages are paywalled — §13).

| Shareholder | Reported stake | Subscribed capital | Source quality |
|---|---|---|---|
| PBOC-affiliated institutions — **7 of them**: CNCC, Wutongshu Investment Platform (外管局下属), 银行间市场清算所, 上海黄金交易所, 中国银行间市场交易商协会 and others — **aggregate** | **37%** | RMB 760 million | ✅ 经济观察网 2017-08-05; 移动支付网 2017-08-04; consistent with zh.wikipedia's *Shareholders* section |
| — of which **PCAC (中国支付清算协会)** | **3%** | — | ✅ 移动支付网 2017-08-04 — held so PCAC can **vote on behalf of small/mid-sized payment institutions that are not eligible to hold shares** |
| **29 third-party payment institutions** — **aggregate** | **63%** | — | ✅ 移动支付网 2017-08-04 |
| CNCC (中国人民银行清算总中心) — **individually** | **12%** | RMB 240 million | ⚠ 爱企查 equity snippet (aggregator); arithmetically consistent with RMB 2bn capital |
| 梧桐树投资平台有限责任公司 (Wutongshu Investment Platform) | **10%** | RMB 200 million | ⚠ 爱企查 snippet; also the subject of a reported transfer (below) |
| 支付宝 (Alipay) | **9.61%** | — | ✅ 经济观察网 2017-08-05; zh.wikipedia |
| 财付通 (Tenpay) | **9.61%** | — | ✅ 经济观察网 2017-08-05; zh.wikipedia |
| 网银在线 — largest holder **among the other payment institutions** | **4.71%** | — | ⚠ 移动支付网 2017-08-04 (single source, trade) |
| 易联支付 and 捷付睿通 — the **smallest** holders | **0.14% each** | — | ⚠ 移动支付网 2017-08-04 (single source, trade) |
| **All remaining individual shareholders and percentages** | **not obtained** | — | ❌ **not verifiable in this pass** — see §13 |

⚠ **A reported change of hands:** a xueqiu post citing a Beijing Equity Exchange (北京产权交易所) disclosure indicated that 梧桐树投资平台 (10%) planned to divest its stake, and cnstock's related-article list carries a headline "网联清算公司10%股权挂牌转让 底价为8.66亿元" (10% listed with a floor price of RMB 866 million). **The xueqiu page was WAF-blocked and the transfer article itself was not retrieved.** Do not assert a completion; do not assert the stake is unchanged either. Treat as an open item (§6.3, §13).

⚠ **Arithmetic note on the aggregates — read the table as attributed figures, not as a reconciled register.** The three aggregates that the 2017 trade and national press report (7 PBOC-affiliated institutions 37% + PCAC 3% + 29 payment institutions 63%) sum to **103%**, not 100%. The two most likely explanations are (i) the PCAC 3% is carved *out of* one of the other aggregates as reported in that source rather than additive to it, or (ii) the aggregate figures come from different stages of the capital plan (the founding agreement of August 2017 versus the later registered allocation). No source retrieved in this pass reconciles them, and the complete register of 45 names with individual percentages was not obtainable (§13). Each figure is therefore stated against its own source, and **no derived total should be quoted from this guide** — either the specific sourced line, or nothing.

One structural point is worth stating because it is repeatedly garbled in secondary summaries: the two market-leading payment institutions were **not** the controlling shareholders. 经济观察网 records that the seven PBOC-affiliated institutions together held 37% and that "央行系" (the central-bank family) was therefore the largest shareholder by a wide margin, while Alipay and Tenpay each held 9.61%. The same report notes that an earlier proposal that would have had Alipay and Tenpay effectively dominate the platform's design — the two-centre "north/south" proposal — was rejected after opposition from other third-party institutions and because it was inconsistent with the PBOC's stated aim that all third-party payment institutions compete on an equal footing on interoperability.
### 2.4 The risk rationale, in the archive's own words

✅ Short-form definition from the 断直连 encyclopaedia entry: *"在支付领域，特指第三方支付机构将涉及银行 账户的网络支付业务迁移至网联或银联平台处理"* — in payments, 断直连 specifically means third-party payment institutions migrating bank-account-involving online payment business to be processed on the NetsUnion *or UnionPay* platform. The same entry states the purpose: unlike card settlement which must run through the card organisation UnionPay, Alipay, Tenpay and peers had formed a model that bypassed UnionPay and connected directly to banks; to eliminate the risks of information insecurity, information opacity and duplicated investment under direct connection, the PBOC led the creation of the NUCC platform to provide payment institutions with unified, public funds-clearing services.

✅ The regulatory linkage between the two reforms is stated by the PBOC itself on gov.cn (2019-01-09, Xinhua): PBOC Deputy Governor 范一飞, and PBOC Payment & Clearing Division Director 温信祥, described the reserve-fund custody rule and 断直连 as *directly related* and both part of the internet-finance risk special rectification campaign, with reserve centralisation resting on the foundation laid by 断直连. That sentence is the hinge between §2 and §3.
### 2.5 Deadlines and how enforcement actually went

The published record shows a mandate that was *substantially* delivered, with slippage, dual connections and one-year-late completion in the tail. The dates below are all documented; the interpretation in the final bullet is ⚠.

| Date | Evidence of progress or slippage | Source |
|---|---|---|
| 2017-06-30 | 12 national joint-stock/state banks connected: BOC, CCB, ICBC, BOCOM, CMB, Ping An, CITIC, CEB, Hengfeng, CZ Bank, Bohai, Huaxia — covering >70% of personal bank accounts by market share. Platform begins clearing real user scenarios from that date. | ⚠ 经济观察网 2017-08-05 |
| 2017-10-15 | The 209号 preparation deadline arrives; press reports "接入网联的机构已有20余家" — a bit over 20 institutions connected | ⚠ 中国新闻网 via 移动支付网, 2017-10-15 |
| 2018-03-31 | Platform has connected ~340 banks and ~100 payment institutions; cumulative clearing >10 billion transactions, approaching RMB 3 trillion; single-day peak >100 million transactions | ⚠ 国际在线 2018-04-02; sohu/中国证券网 2018 |
| 2018 Q2 | PBOC quarterly statistics: NUCC platform processed **8.146 billion transactions, RMB 2.7 trillion**, average daily ~89.51 million | ✅ PBOC Q2-2018 payment statistics, via 金融界 2018-08-21 |
| 2018-06-30 | The statutory migration cut-off in 209号文 passes | ✅ |
| end-2018 | PBOC (温信祥): **99%** of payment business conducted between payment institutions and banks is processed via UnionPay/NUCC | ✅ gov.cn / Xinhua 2019-01-09 |
| 2019 New Year message | **All 115 network-payment-licensed payment institutions and 424 banks connected; 99% of existing cross-institution business migrated; daily funds-class transactions on the platform exceed 1 billion** | ✅ 界面新闻 2758152 reporting NUCC's New Year address |
| 2019 full year | **534 commercial banks and 115 payment institutions connected**; platform processed **397.542 billion transactions, RMB 259.84 trillion** | ✅ PBOC *2019 Payment System Operation Overall Situation*, published 2020-03-17 |
| 2019-10 | NUCC average daily transaction count up nearly 80% year on year | ⚠ 经济日报 via news.sina 2019-10-09 |
| 2022-01-01 (元旦) | NUCC processed **5.133 billion** cross-institution online payment transactions over the New Year period, **+17.86%** year on year | ⚠ 移动支付网 2022-01-04 |

⚠ **Dual connection and staged migration.** Press reporting through 2018–2019 describes institutions connecting to **both** UnionPay and NUCC and migrating their traffic gradually; 209号文 itself contemplated either platform, and the reserve-fund rule (114号) explicitly instructs payment institutions to open their centralised reserve account "根据与中国银联股份有限公司或网联清算有限公司的 业务对接情况" — *according to their business connection with UnionPay or NUCC*. Two consequences follow for a bank or payment institution reading this guide:

- "断直连" was never "everyone must use NUCC". It was "no one may bypass both licensed channels". NUCC became the dominant route for bank-account-based online payment originated by payment institutions; UnionPay remained the route for card and for institutions that chose it.
- The migration was **not complete by 2018-06-30** in a strict sense: the PBOC's own "99%" language at end-2018, and the 2019 New Year message's "99% of existing cross-institution business", both concede a residual. This guide therefore treats the cut-off dates as *enforcement milestones against which progress was measured*, not as a hard switch-off. ⚠ interpretation.
---

## 3. The Customer Reserve Funds Reform
### 3.1 Why this section exists, and what it is not

**This is a distinct reform and it would merit its own treatment.** It is *not* the same instrument as 209号文, it was *not* issued by the same PBOC department, and it did *not* concern clearing at all in its first movement. It concerns **where customer money sits**. It is covered here because the two reforms interlock: without centralised reserve custody, NUCC's clearing would still be a clearing layer sitting in front of funds parked in dozens of commercial-bank accounts, and the supervisor's visibility would be partial.

Term check: **备付金 / customer reserve funds** — the customer funds held by a non-bank payment institution in the course of its payment business (unsettled merchant proceeds, prepaid balances, in-transit funds), which the institution must hold in segregated accounts and must not use for its own purposes.
### 3.2 The instruments

✅ **银办发〔2018〕114号** — *《中国人民银行办公厅关于支付机构客户备付金全部集中交存有关事宜的通知》* ("Notice of the PBOC General Office on Matters Concerning the Centralised Deposit of Payment Institutions' Customer Reserve Funds in Full") — the operative centralisation instrument.

Its content, verified from two independent reproductions (m.sohu/大宗传媒 carrying the central-bank notice text, 2018-06-29; and finance.sina.cn 2018-06-29 reproducing the notice in full, including the addressee line):

| Element | Detail (from the notice text) |
|---|---|
| Step-up start | **2018-07-09** — from this date the centralised deposit ratio rises **month by month** |
| Completion date | **2019-01-14** — 100% centralised deposit (实现100%集中交存) |
| Deposit timing | Deposits are made on the **second Monday of each month** (deferred when it falls on a holiday) |
| Deposit base | The **average daily balance of customer reserve funds in the previous month** (上一个月客户备付金日均余额) |
| Excluded from the base | Cross-border RMB reserve accounts, fund-sales settlement dedicated accounts, and foreign-exchange reserve accounts (their balances are excluded for the time being) |
| Permitted residual accounts | One cross-border RMB reserve account; one fund-sales settlement dedicated account; for institutions licensed for prepaid-card issuance/acceptance, one reserve account, whose funds must be transferred to the PBOC "备付金集中存管账户" every working day before the HVPS business cut-off; FX reserve accounts per SAFE rules |
| Closure obligation | All **other** reserve accounts held at commercial banks must be **closed by 2019-01-14** |
| New account | Institutions must open a **"备付金集中存管账户"** at the PBOC branch where their legal person is located by **2019-01-14**, and close the former 备付金交存专户 at the entrusting custodian bank within 2 working days of opening |
| Critical routing rule | Funds transfers from the centralised custody account must be handled **through UnionPay or NUCC** — "根据与中国银联股份有限公司或网联清算有限公司的业务对接情况" (according to the institution's business connection with UnionPay or NUCC) |
| Operational duty on the clearing houses | UnionPay and NUCC must support commercial banks' and payment institutions' orderly access, in line with the centralised-deposit timetable, and ensure business continues to be processed normally |
| Addressees | PBOC Shanghai Head Office, branches, operations departments, provincial capital and sub-provincial city central sub-branches; state-owned, joint-stock and Postal Savings banks; UnionPay; NUCC; all non-bank payment institutions |

⚠ **The earlier phases, and the instrument that could not be verified.** The PBOC's own account of the sequence (gov.cn / Xinhua, 2019-01-09) is: in **2017** the central bank required non-bank payment institutions to deposit **part** of customer reserve funds into designated-purpose deposit accounts; in **early 2018** it raised the centralised deposit ratio to **about 50%**; then it issued a further notice requiring the monthly step-up to 100% by 2019-01-14. The **exact 2017 phase-one instrument number and date were NOT captured in this research pass** and are listed in §13. Do not cite a number for it.

✅ **The completion was announced at the political level.** On **2019-01-09**, PBOC Deputy Governor 范一飞 stated at a State Council Information Office policy briefing that on **14 January** payment institutions' reserve funds would be entirely deposited with the PBOC, and that the existing reserve fund management measures would be revised — meaning the safety of customer reserve funds would be guaranteed by the PBOC itself, and misappropriation or occupation of customer funds curbed. He is quoted saying that previously reserve funds were kept at commercial banks and that this "did have many problems". PBOC Payment & Clearing Division Director 温信祥 added that reserve custody and 断直连 are *directly related* and both part of the internet-finance risk special rectification, and that as of end-2018, **99%** of payment business conducted between payment institutions and banks was already processed through NUCC/UnionPay — "这为备付金集中存管打下了坚实的基础" ("this laid a solid foundation for centralised reserve custody").
### 3.3 Why 100% centralisation matters to the clearing model

✅ The mechanics, as co-constructed by 114号 and the platform design:

- Before: many reserve accounts at many commercial banks, per institution — reported in industry write-ups as numbering into the dozens or hundreds per large institution (⚠ secondary).
- After: **one** centralised custody account at the PBOC per institution, plus narrow statutory exceptions. Funds no longer sit at commercial banks at all.
- The centralised account is **not directly reachable** by the payment institution for day-to-day payment operation. Per the widely reproduced industry description of the post-reform model (⚠ secondary — a product-management practitioner write-up, 2023-07-12, carried by sohu and woshipm): the institution manages the account **through NUCC or UnionPay**, and NUCC holds a **virtual account** with two balances — a **映射额度 (mapped quota)** and a **可用额度 (available quota)**. The mapped quota is created by "圈存" — earmarking part of the centralised PBOC balance to NUCC/UnionPay, as an authorisation; the balance in the PBOC account itself is unchanged by this. Within a clearing session, **可用额度 = 映射额度 + 收款 − 出款**. Only when NUCC submits the session's net position to the PBOC does the centralised account actually move.
- Consequence 1 — **clearing is informational plus net settlement, not a conduit for parked funds.** Because the funds are already inside the central bank, the clearing platform's role is to instruct, net and confirm; it is not holding or transmitting customer float.
- Consequence 2 — **pay-out is faster than before.** Under the old model money had to settle into the reserve account before it could be paid out; under the mapped-quota model, incoming receipts increase the available quota in real time and can be used for outbound payment immediately (⚠ same source).
### 3.4 The business-economics effect on payment institutions

The direct economic effect is the loss of **interest income on reserve balances**. With 100% of customer reserves at the PBOC, no interest-bearing commercial-bank deposit balance remains from that source, and the affected institutions are the largest ones by float — Alipay and Tenpay above all. The PBOC's own framing is that centralisation "优化支付市场资源配置、提高清算效率、降低支付成本" (optimises resource allocation in the payment market, raises clearing efficiency, lowers payment cost) and, on the prudential side, permits timely monitoring and disposal of non-bank payment institution risk (✅ 温信祥, gov.cn 2019-01-09).

⚠ **The quantification is deliberately not attempted here.** Any figure for the industry's or an individual institution's lost interest income would need its own sourcing and reconciliation, and this guide does not manufacture one. The reform also, in principle, *reduced* some institution costs (account maintenance across dozens of banks, and some channel pricing asymmetries) — so the net business-economics effect is a modelled outcome, not a single number.

❌ **Rejected framing:** "the reserve-fund reform was part of the NUCC platform's design". The two have different instruments (银办发〔2018〕114号 vs 银支付〔2017〕209号), different dates and different issuers inside the PBOC (General Office vs Payment & Clearing Division). 114号 *requires* the clearing houses to support access, which is connection, not identity.

---

## 4. Platform and Technology
### 4.1 What is actually documented, and who says so

The honest summary: **the platform's functional architecture is well documented at the level of sites, capacity and outcomes; its internal design, interface specifications and data model are not public.** Everything below is either an operator statement (with its incentive labelled) or a secondary reconstruction. Do not treat any of it as an engineering specification.
### 4.2 Sites, centres and availability

| Claim | Detail | Assessment |
|---|---|---|
| **Three locations, six data centres** | 三地六中心 — the platform operates infrastructure in **Beijing, Shanghai and Shenzhen**, two centre machine rooms per location, six centres in total, with multi-site multi-active operation, redundancy/fault tolerance and intelligent traffic routing | ✅ as a *statement* — reported in 2017 by 经济观察网 ("目前，网联采用三地（北京、上海、深圳）六中心的分布式架构，即每个地方有两个中心机房，六个中心之间实现多点多活、冗余容错、智能导流") and repeated by zh.wikipedia; restated by the operator in 2025 as the 三地六中心 distributed cloud architecture |
| **Five-layer disaster-recovery system** | 五层容灾体系 | ⚠ operator statement (NUCC, 2025 China International Finance Exhibition, via 第一财经 2025-06-18) — no public detail on what the five layers are |
| **Availability "100%"** | "平台系统成功率、资金清算准确率和及时率、核心应用系统可用性连年保持在100%" — system success rate, fund-clearing accuracy and timeliness, core application availability maintained at 100% for consecutive years | ⚠ operator marketing claim; a claim of *100%* over consecutive years is a definitional claim (presumably measured against a defined denominator and excluding planned windows), not an engineering fact. Quote it as the operator's claim or not at all. |
| **Connected institutions** | >3,000 commercial banks and payment institutions (2025) | ⚠ operator statement, same venue; see §7 |

### 4.3 Capacity and throughput — two eras of figures

| When | Figure | Source and status |
|---|---|---|
| 2017 (design/trial) | Trial-phase starting capacity **1,200 tps**; target steady-state capacity **120,000 tps**; peak-phase target **180,000 tps**; distributed design permitting horizontal scaling | ⚠ press reporting at launch (新浪财经/看点 2017-06-07; 经济观察网 2017-08-05) — target figures, not achieved figures |
| 2018-03 | Achieved: **maximum single-day transaction count >100 million**; **maximum real-time peak >7,000 TPS**; "北京、上海、深圳3地6中心基础设施满配" (3-location/6-centre infrastructure fully equipped) | ⚠ 亚洲财经 via sohu 2018-03-31 |
| 2025-06 | Platform processing capability **>300,000 tps**; **average transaction latency 200 ms** | ⚠ operator statement, 第一财经 2025-06-18. Note this exceeds even the 2017 *peak* target of 180,000 tps. |
| 2025 CNY's Eve | Peak of **110,000 tps** actually processed (跨机构网络支付交易) | ⚠ operator statement; see §7 |

⚠ **Do not reconcile these into a growth curve.** The 1,200/120,000/180,000 figures are 2017 *targets* on the then-current design; the 7,000 TPS is an early 2018 realised peak; the 300,000 tps is a 2025 *capability* claim; the 110,000 tps is a 2025 realised peak. They are different measures over eight years and the operator has never published a consistent series.
### 4.4 Core systems — weak evidence, labelled as such

⚠ **Weak evidence.** Developer-community write-ups (CSDN articles, and similar syndicated posts) describe NUCC's platform as three core systems plus enabling components:

- a **transaction system (交易)** — online message acceptance and routing;
- a **clearing system (清算)** — computing cross-institution and acquiring positions;
- a **reconciliation system (对账)** — matching transaction logs against counterparty and clearing records;
- a **front-end/gateway (前置)** component for participant connectivity; and
- a **quota-mapping (额度映射)** mechanism linking the centralised reserve-custody account to real-time clearing — which is precisely the mechanism described for the reserve reform in §3.3.

Two caveats a professional reader must apply: (i) the three-system decomposition is a *generic payments-platform* decomposition and would be asserted by any practitioner; it is not evidence of NUCC's actual internal design; (ii) CSDN is a developer blog platform, not an authoritative source. The quota-mapping element is the only part of this picture corroborated by an independent description (the 2023 product write-up cited in §3.3), and even that is secondary. Label ⚠ and move on.
### 4.5 Interfaces, message standards and certification — what participants must implement

⚠ **The specifications exist but are not public.** NUCC publishes its platform standards as **enterprise standards in the Q/NUC series** (企业标准, "Q/" prefix). Titles that surfaced in this research, with year-and-version markers as printed:

| Standard (as titled in document listings) | Range |
|---|---|
| **Q/NUC 102—2022** 网络支付清算平台 报文交换技术规范（V1.5） — message-exchange technical specification, V1.5 | Message structure, format, encoding, digital signatures, identifier composition rules; "网联普通商密" classification marking with a 2022.8–2027.8 period shown on the listing |
| **Q/NUC 106—2022** 网络支付清算平台 技术接入指南（V1.4） — technical access guide, V1.4 | Access methods, admission requirements, technical requirements, operational guidance, access process |
| **Q/NUC 105.2** 网络支付清算平台 专线接入规范 — dedicated-line access specification, split into Part 1 (banks) and Part 2 (payment institutions) | Connection mode, access prerequisites, technical requirements |

Assessment: the *existence and shape* of these standards (a Q/NUC enterprise-standard series governing message exchange, technical access and dedicated-line connection, with separate parts for banks and payment institutions) is ⚠ reliable enough to state, because the listings carry version numbers and issuing years that could not be fabricated by a summary page. Their **content is not public**, and any statement about a specific mandatory field, algorithm or protocol should be treated as unverified. One widely syndicated blog post claims mandatory use of the Chinese national cryptographic algorithms SM2/SM3/SM4 across the chain, a "《网联平台接口规范》V3.2" message structure with twelve mandatory fields, and a five-year local retention obligation for transaction logs and raw messages — ❌ **none of which could be corroborated**; the last point in particular is a data-retention claim that needs its own legal basis. Do not rely on any of it.

Practical takeaway for an integrating institution: **the specification is delivered through the participant onboarding channel, not from a public website**, and planning must assume a formal specification handover, a test environment and a certification step (see §6.5 and §10).
### 4.6 Milestones that do speak to technology

- **一键绑卡 ("one-click card binding")** service launched **2019-06-10** — a user-experience feature that lets a user bind a bank card once and have that binding recognised across participating institutions (⚠ Baike, company milestone list).
- **2019 Bank Technology Development Award, Special Prize (特等奖)** — awarded at a ceremony on **2020-09-24**; the PBOC is the awarding body (⚠ Baike citing the PBOC's own award announcement; the award's existence is verifiable, the attribution to NUCC for a specific system is ⚠).
- **ISO standard participation**, 《第三方支付服务信息系统的安全目的》, **2021-10-15** (⚠ Baike).
- **China Cyberspace Security Association** membership admitted **2025-04-22** (⚠ Baike citing the association's own new-member notice of 2025-06-06).
- **Industry joint stress-testing**: NUCC states that since 2024 it has run joint peak-load stress tests with member institutions and that the industry-wide joint stress-test system success rate rose from **95.27% to above 99.99%**, that commercial banks' high-concurrency handling capacity improved by **more than 40%**, and that average end-to-end latency fell by **30%** (⚠ operator statement, 第一 财经 2025-06-18). These are the operator's own measurements of a programme it ran; they are internally coherent and are the kind of claim that a participant bank can in principle corroborate from its own test records.
### 4.7 What this section does not do

It does not describe NUCC's architecture using the operator's marketing vocabulary — 分布式 ("distributed"), 云 ("cloud"), 多地多中心 ("multi-site, multi-centre"), 高可用 ("high availability") — as if those words were a design. Where the design is not public, this guide says so. Where the operator makes a claim, the claim is attributed and its incentive noted (an exhibition announcement or a recruitment page is not an audit).

---

## 5. Clearing and Settlement Model
### 5.1 The division of labour in one sentence

**NUCC clears; the banks and the PBOC settle.** NUCC accepts, validates, routes and nets the instructions, maintains the participants' mapped/available quotas, computes positions and performs reconciliation; the actual movement of funds occurs on the participants' accounts at the PBOC and on banks' customer accounts. NUCC does not hold customer money (§3.3) and is not a bank.
### 5.2 Step-by-step flow, with the obligation holder at each step

The flow below is the *documented* bank-account-based online payment path for a payment-institution transaction, reconstructed from (i) the platform's own chartered business scope, (ii) the reserve-reform mechanics in §3.3, and (iii) the payment-flow diagrams published in the 断直连 literature (⚠ the sequence is secondary-sourced; the obligation allocation is the useful part and is consistent across sources).

| # | Step | Who acts | Who carries the obligation |
|---|---|---|---|
| 1 | Customer instructs a payment from a bank account via the payment institution's app | Customer → payment institution | Payment institution: authority, KYC, AML, authentication, and the correctness of the instruction |
| 2 | Payment institution validates and submits the transaction to the **NUCC platform** over its dedicated line | Payment institution → NUCC | Payment institution: message compliance with the Q/NUC standards, and that the transaction is within its licensed business |
| 3 | NUCC validates, deduplicates, performs risk/fraud and AML-relevant screening on the flow, and routes the instruction to the relevant bank | NUCC | NUCC: platform availability, routing correctness, and the integrity of the record it keeps for the supervisor |
| 4 | The **issuing/funding bank** checks the account and debits (or holds) the funds, returning a success/failure response | Issuing bank | Issuing bank: account state, funds availability, authentication per its own controls |
| 5 | NUCC registers the result and **adjusts the participant's available quota in real time** (可用额度 = 映射额度 + 收款 − 出款) | NUCC | NUCC: real-time clearing accuracy; the payment institution: its quota sufficiency |
| 6 | Merchant-side funds are credited to the payment institution's own or merchant's account per its settlement rules; the payment institution's pay-out to the merchant follows its contractual schedule | Payment institution → merchant | Payment institution: merchant contract, settlement schedule, and the funds it is obliged to pay |
| 7 | NUCC computes the session's **net position** per participant and submits it for **final settlement** — the actual movement on the centralised reserve account and the banks' settlement accounts | NUCC → PBOC / settlement banks | NUCC: the accuracy of the net position; the PBOC: the movement of the centralised custody account; the settlement bank: its own account movements |
| 8 | Reconciliation: transaction-level and batch files are exchanged and matched between NUCC and each participant | NUCC ↔ participants | Both: NUCC must publish authoritative files; the participant must match, confirm and raise discrepancies within the agreed window |

Note the two-sided nature of the model — a payment institution can be both a payer-side and a payee-side participant in the same session, which is why the netting in step 7 matters to its liquidity.
### 5.3 What NUCC clears vs what the banks settle

| Function | NUCC | Participants (banks, payment institutions) | PBOC |
|---|---|---|---|
| Message acceptance, validation, routing | Yes | Origination/termination | No |
| Cross-institution position calculation and netting | Yes | No | No |
| Quota management (mapped/available) against the centralised reserve account | Yes — the virtual account mechanism | Enquiries and quota adjustments (through NUCC/UnionPay) | Holds the underlying centralised custody account |
| Funds movement on customer accounts | No | Yes — the banks debit/credit their customers | No |
| Final settlement of net positions | Submits the net instruction | Settlement banks move funds | Moves the centralised reserve custody accounts |
| Dispute co-ordination and arbitration | Yes — a chartered function (协调和仲裁业务纠纷) | Escalation and evidence | Supervisory escalation |

### 5.4 Settlement cycle and timing — what is and is not public

⚠ **Partly documented, and only via secondary sources.** The picture that emerges consistently from industry descriptions is:

- **"实时清算（real-time clearing）, 定时结算（scheduled settlement）"** — the clearing of instructions and the adjustment of the participant's available quota happen in real time, while the actual funds settlement to the PBOC occurs at fixed times.
- Detailed step timings published by practitioner sources: day-cut at **T-day 24:00**, with the online transaction log synchronised to the clearing system; clearing and acquiring position computation in the **early hours of T+1**; at **T+1 10:00** the aggregated clearing result (amounts receivable by each institution's clearing account and by directly-connected merchants' settlement accounts) is sent to the institutions for confirmation; institutions confirm after reconciliation and the clearing system then issues payment instructions in sequence. (⚠ zhihu/sohu/163.com practitioner write-ups, 2020-2023 vintage, carried from the same author — treat as *the community's understanding*, single-sourced.)

✅ **What is definitively public and useful:** 银办发〔2018〕114号 fixes the *reserve-fund* deposit calendar — deposits on the **second Monday of each month** on the previous month's average daily balance. That is a monthly cadence for the *custody* flow, not for payment settlement, and it should not be confused with the daily/S settlement cycle of transactions.

❌ **Not established:** a public, authoritative statement of NUCC's settlement sessions, cut-off times, values dates or DVP-style conventions. Absent it, this guide records the practitioner consensus as ⚠ and lists the gap in §13. Any bank designing liquidity around NUCC must obtain these from the participant documentation.
### 5.5 Reconciliation and dispute arrangements

✅ **Chartered:** NUCC's registered business scope includes **协调和仲裁业务纠纷** — co-ordinating and arbitrating business disputes. This is a statutory-scope function, which means dispute handling on the platform is not merely a commercial service level: the platform is the named forum for disputes arising in the business it clears.

⚠ **In practice**, participants operate a layered reconciliation and exception model, evidenced by implementation announcements such as a provincial rural credit union going into production with a **batch-level reconciliation function (交易批次对账功能)** for its NUCC business (移动支付网, 2022-04-24). That is a bank-side artifact of the arrangement: the platform publishes files, the participant builds matching, and discrepancies are resolved within defined windows, escalating to the platform's own dispute process when they cannot be resolved bilaterally.
### 5.6 Fees and pricing basis

⚠ **No tariff schedule is published.** What can be stated with sources:

- **2017, pre-launch, the stated principle** was: provide safe, efficient and economical service on the principle of saving industry cost and not adding to market burden; **profit is not the starting point**; adhere to **保本微利** ("break even with a thin margin"); and **in the initial period the transfer-clearing service may be provided to the industry free of charge** (✅ 新浪财经/看点 2017-06-07, a launch-period profile of the platform).
- **Later reporting of actual charging** describes a structure with a **volume threshold**: below it a **monthly package price** (the figures circulated for the two largest institutions were on the order of tens of millions of RMB per month), above it **usage-based pricing** (⚠ a 2019-vintage practitioner account relayed on zhihu, single-sourced, no official document; **do not quote the figures as NUCC tariffs**).
- **Secondary legal-summary pages** assert that NUCC charges for clearing services, that the fee is set by agreement, and that the charging is supervised by the PBOC (⚠ low-grade aggregator content — useful only as corroboration that a fee exists).

The defensible statement for a bank file: *NUCC is a cost-recovery-minded clearing utility whose tariffs are agreed bilaterally with participants and are not published; a bank cannot budget the charge from public sources and must obtain the current tariff schedule through the participant channel.*

---

## 6. Participants
### 6.1 Who is in scope on the payment-institution side

✅ The platform's chartered purpose is the **online payment business of non-bank payment institutions (非银行支付机构)** where a **bank account** is involved. That scope definition is what pulls a given transaction onto NUCC rather than onto UnionPay: if the flow is bank-account-based and originated by a licensed non-bank payment institution, 209号文 sends it to NUCC (or to UnionPay, where the institution is connected there — §8).

The licensing regime that determines who may be a participant is now consolidated in two instruments:

| Instrument | Detail | Status |
|---|---|---|
| **《非银行支付机构监督管理条例》** — Regulations on the Supervision and Administration of Non-Bank Payment Institutions | State Council administrative regulation; consolidated the previous patchwork of PBOC rules for payment institutions | ✅ (referenced as the basis of the 实施细则 below) |
| **《非银行支付机构监督管理条例实施细则》** — Implementing Rules, **PBOC Order 〔2024〕第4号** | Approved at the PBOC's **5th executive meeting of 2024 on 2024-07-01**; **published 2024-07-09**; effective from the date of publication. Repeals the 2010 《非金融机构支付服务管理办法》 (PBOC Order 〔2010〕第2号) and its 2010 implementing rules | ✅ text read from the State Council Gazette (gov.cn, 2024 Issue 24) |

Under the Implementing Rules the payment business types are **储值账户运营 (stored-value account operation) Classes I and II** and **支付交易处理 (payment transaction processing) Classes I and II** (set out in Article 55 of the rules); minimum registered capital starts from **RMB 100 million** with add-ons by business type and geographic scope (Article 8); existing institutions run on a transition period ending at their licence expiry, with a floor of 12 months (Article 73). The rule that matters most for market structure: institutions with a **major shareholder (主要股东)** — defined as ≥10% of capital, or under 10% but with material influence — face continuing obligations over that shareholder and 3-year stability commitments (Articles 10, 25, 75).

⚠ **Participants in the earlier era**: press and company reporting through 2017–2019 consistently describes the in-scope population as institutions licensed for **网络支付 (online payment)** — "全部持 网络支付牌照的115家支付机构" (all 115 institutions holding network-payment licences) had connected by the time of NUCC's 2019 New Year message, and the PBOC's 2019 yearbook-statistics record **115 payment institutions** on the platform at year-end 2019. The 115 figure is therefore the historical licensed cohort; the current count has changed through merger, exit and the 2024 rulebook, and this guide does not assert a current number. ⚠
### 6.2 The banks' role on the funding and settlement side

Banks appear in **three distinct roles**, and conflating them causes design errors:

1. **Funding/issuing bank** — holds the customer's account, debits it on instruction, and returns the result to the platform. This is the bank whose customer account is the source of funds.
2. **Settlement-side participant** — receives the platform's net-position results, effects the customer and merchant-side credits, and (for reserve purposes) interacts with the PBOC's centralised custody account regime described in §3.3.
3. **Reserve-custody bank (historical)** — before the 2018–19 reform, held the payment institution's reserve accounts. The 114号 notice eliminates this role for everything except the narrow statutory exceptions (§3.2). Any current bank-side design that assumes "we hold the PI's float" is out of date by eight years.

✅ The bank population connected grew as: **424 banks** by the time of the 2019 New Year message; **534 commercial banks** at end-2019 per PBOC statistics; **>3,000 commercial banks and payment institutions** per the operator's 2025 exhibition disclosure; **3,000+ banks and 100+ payment institutions, serving over 1 billion users and over 200 million merchants** per NUCC's 2026 exhibition material.
### 6.3 Shareholder versus participant — not the same set

This distinction is frequently muddled in secondary commentary. The facts as they stand:

| Relationship | Evidence |
|---|---|
| **45 founding shareholders** signed the founding agreement on 2017-08-02, including the PBOC's CNCC, Tenpay, Alipay and UnionPay Merchant Services | ✅ 经济观察网 2017-08-05 |
| **7 PBOC-affiliated institutions hold 37%** (RMB 760 million): CNCC, Wutongshu Investment Platform (a SAFE-affiliated entity), the interbank-market clearing house, the Shanghai Gold Exchange and the National Association of Financial Market Institutional Investors among them | ✅ 经济观察网 2017-08-05; 移动支付网 2017-08-04 |
| **The Payment & Clearing Association of China (PCAC) holds 3%** and exercises voting rights **on behalf of small and mid-sized payment institutions that are not eligible to hold shares** | ✅ 移动支付网 2017-08-04 |
| **29 third-party payment institutions hold the remaining 63%**; largest non-Big-Two holder 网银在线 at 4.71%, smallest 易联支付 and 捷付睿通 at 0.14% each; Alipay and Tenpay 9.61% each | ✅ 移动支付网 2017-08-04 |
| **Nine large/mid-sized payment institutions** are the candidates for the operator's board and supervisory board seats; their combined transaction-volume market share exceeds **96%** | ⚠ same source, 2017 vintage |
| **Most participating banks are not shareholders**, and several shareholders (e.g. the Shanghai Gold Exchange) are not participants in this business at all | ⚠ inference from the two lists above — stated because it is structurally important |

⚠ **A reported transfer of a shareholder stake**: in 2025–26, cnstock.com's related-article list shows a headline **"网联清算公司10%股权挂牌转让 底价为8.66亿元"** (10% equity of NUCC listed for transfer with a floor price of RMB 866 million). The 10% stake matches 梧桐树投资平台's reported holding, and a Beijing Equity Exchange disclosure to that effect was referenced by a xueqiu post. **The transfer article itself could not be retrieved in this pass** (see §13). Note the implied whole-company valuation if the floor price were the clearing price — roughly **RMB 8.66 billion** — but do not treat that as a valuation: a floor price is a floor price, and no completion was verified.
### 6.4 What "participant" does not mean

- A participant does **not** obtain any ownership or governance right by connecting (unless it is also a shareholder).
- A participant does **not** hold customer funds at NUCC — there is nothing to hold (§3.3).
- A shareholder does **not** get preferential access or pricing; uniform treatment of interfaces and pricing was an explicit goal of the reform (§1.5).
### 6.5 Access and onboarding for a new participant

⚠ **Inferred, because NUCC's own onboarding manual is not public.** The checklist below is the pattern of a licensed Chinese clearing institution's participant onboarding, evidenced for a comparable licensed clearing institution by the published self-testing/onboarding procedure of a card organisation (qualification review → agreement → institution code → self-testing platform access → clearing agreement → interface development → offline testing → certificate application and exchange → online testing → production → operations). Treat it as a *planning template*, not as NUCC's documented process.

Planning template for a new participant:

1. **Eligibility** — hold the relevant PBOC licence (bank, or non-bank payment institution with the in-scope business type under the 2024 rulebook, §6.1).
2. **Admission review and agreement** — submit corporate and systems documentation to NUCC; sign the participation and service agreements, including the clearing service terms.
3. **Institution codes and participant records** — obtain the platform identifiers used in messages and in reconciliation files.
4. **Connectivity** — provision a dedicated line per the dedicated-line access standard; establish the production and test environments.
5. **Interface development and certification** — implement the message-exchange specification, complete offline/self-testing, then formal online testing and certification in NUCC's test environment.
6. **Cryptographic and certificate exchange** — exchange the signing/encryption credentials required for message integrity (the *requirement* for a certificate exchange step is standard; the specific algorithms are not established here — §4.5).
7. **Funds and quota arrangements** — open/confirm the PBOC centralised custody account where applicable (§3.2), and establish the mapped/available quota arrangements on the platform (§3.3).
8. **Reconciliation and operations** — build file intake, matching and exception handling; establish the operational contacts and escalation path for the platform's dispute process (§5.5).
9. **Go-live and ramp** — production cut-over with volume ramping, monitored against the platform's stress-test regime (§4.6).
---

## 7. Volumes and Scale
### 7.1 The headline figures, attributed and dated

All figures below are the **operator's own** (or the PBOC's statistics naming the platform), quoted with the date and channel of their release. None of them is an audited financial statement; an exhibition disclosure is a marketing event.

| Period | Volume | Value | Source and status |
|---|---|---|---|
| to 2018-03-31 (cumulative) | >10 billion transactions (突破100亿笔) | RMB 2.83 trillion cumulative | ⚠ sohu/亚洲财经 2018-03-31, relaying platform information |
| 2018 Q2 | 8.146 billion | RMB 2.7 trillion; daily average ~89.51 million | ✅ PBOC Q2-2018 payment statistics (金融界 2018-08-21) |
| 2019 (full year) | **397.542 billion** (3975.42亿笔) | **RMB 259.84 trillion**; daily average 1.089 billion transactions / RMB 711.897 billion | ✅ PBOC *2019 Payment System Operation Overall Situation* (2020-03-17) |
| 2021 National Day holiday | 14.069 billion (140.69亿笔) | RMB 7.60 trillion | ⚠ NUCC announcement via 移动支付网 2021-10-08 |
| 2021 Double 11 | 24.029 billion (240.29亿笔) | — | ⚠ NUCC announcement via 移动支付网 2021-11-12 |
| 2022 New Year holiday (Jan 1–3) | **5.133 billion** (51.33亿笔), +17.86% y/y | **RMB 3.23 trillion**, +12.70% | ✅-grade platform announcement via 移动支付网 2022-01-04; single-day 1 Jan: 1.833 billion transactions / RMB 1.17 trillion (+16.41%/+14.78%) |
| **2024 (full year)** | **>1 trillion** (超1万亿笔) | **RMB 520 trillion** (520万亿元) | ⚠ **operator disclosure** at the 2025 China International Finance Exhibition, Shanghai, 18–20 June 2025 — reported identically by 第一财经 2025-06-18, 上海证券报/中国证券网, 北京商报 via 搜狐 and 新浪财经 2025-06-18 |
| 2025 CNY's Eve peak | **110,000 tps** (11万笔/秒) — a record | — | ⚠ same 2025 exhibition disclosure |
| Since 2019 — single-day peak | **>4.415 billion transactions in one day** | — | ⚠ operator material at the 2026 China International Finance Exhibition (支付之家 2026-06-16) |
| Since 2019 — daily average | **3.8 billion transactions/day** | — | ⚠ same |
| 2026 CNY's Eve peak | **>125,000 tps** (超12.5万笔/秒) — a record | — | ⚠ operator material, 2026 exhibition; corroborated by NUCC President 崔嵬's interview with 金融时报 (via 中国金融新闻网/腾讯新闻, 2026-09-10) |

Scale of the participant network, for context: **>3,000 commercial banks and payment institutions** (2025 disclosure); **3,000+ banks, 100+ payment institutions, >1 billion users, >200 million merchants** (2026 exhibition material). Both are ⚠ operator statements.
### 7.2 Growth trajectory, where the record supports one

- **The platform's scale moved by roughly an order of magnitude in six years**: from **8.146 billion transactions in Q2 2018** (a ramping quarter, PBOC statistic) to the operator's **>1 trillion transactions for 2024** — i.e. a single quarter in 2018 represents on the order of 3% of a single flat-quarter equivalent in 2024. Arithmetic on the two verified data points; no interpolation is offered beyond this. ✅/⚠
- **Peak-load growth is documented at intervals**: >7,000 TPS (March 2018) → ~66,500 TPS on 2022-01-01 (元旦) → 110,000 TPS on 2025 CNY's Eve → >125,000 TPS on 2026 CNY's Eve. The 2022 point is a *concurrency* peak for a holiday window, the 2025/2026 points are CNY's Eve records; the series is indicative of direction, not a clean benchmark. ⚠
- **Post-2019 growth evidence is event-based, not annual**: the operator publishes holiday and shopping-festival figures (National Day, Double 11, New Year, Spring Festival) rather than a quarterly series, and the PBOC's annual payment-system reports name the platform only in some years. This is a real evidence gap for anyone needing a continuous series — see §13.
### 7.3 What the published figures do and do not measure

This is the most important sub-section in §7, because the headline numbers are routinely quoted without these caveats.

| Caveat | Detail |
|---|---|
| **Operator-reported** | Every NUCC-specific figure comes from NUCC or from a PBOC statistic naming the platform. They are announced at exhibitions, in New Year messages or through affiliated trade press. They are not audited statements and no independent verification exists. |
| **"Funds-class" vs everything** | NUCC distinguishes **资金类交易 (funds-class transactions)** from other messages. Its 2019 New Year message quotes "日间处理的资金类交易已突破10亿笔" (funds-class transactions in daytime exceeding 1 billion), and the 2022 New Year figure is "资金类跨机构网络支付交易". Where a figure says 资金类, non-funds messages (enquiries, quota operations, status messages) are excluded; where a figure does not say so, the basis is ambiguous. ⚠ |
| **Net vs gross** | Not disclosed. A single merchant payment spanning two participating institutions can generate both a pay-side and a receive-side record. Whether the published count is gross records or netted transactions is **not stated by any source in this pass**. ⚠ |
| **Definitional inconsistency across series** | A documented oddity: PBOC's own 2019 statistics report **非银行支付机构网络支付业务 of RMB 249.88 trillion** while also reporting the **NUCC platform processing RMB 259.84 trillion** in the same year — i.e. the platform's cleared value exceeds the total reported value of the underlying business. This indicates the two series are built on different bases (timing, double-counting of both legs, inclusion of bank-originated instructions, or reclassification). Do not compare or add them without the PBOC's definitions. ⚠ flagged as an unresolved inconsistency |
| **What it does measure** | Whatever the operator chooses to publish at that event. A 520-trillion figure is a *statement about magnitudes*, not a Basel-style exposure metric. |
| **Combined channels are not NUCC** | PBOC's holiday releases often give **UnionPay + NUCC combined** figures (e.g. 2025 Spring Festival, 28 Jan–4 Feb: **25.416 billion transactions / RMB 9.78 trillion** combined; 2026 Spring Festival, 15–23 Feb: **39.302 billion transactions / RMB 13.12 trillion**, daily average +37.45% y/y). Those are channel-combined and must never be attributed to NUCC alone. ✅ (PBOC via 中国新闻网 2025-02-05; 新华社 2026-02-24) |

### 7.4 Comparison with the other Chinese retail clearing channels

✅ The PBOC's own 2019 statistics allow a like-dated comparison across the national clearing systems — the single most useful cross-channel table available from public sources:

| System | Operator | 2019 transactions | 2019 value | Daily average |
|---|---|---|---|---|
| **NUCC platform (网联清算平台)** | NUCC | **397.542 billion** | **RMB 259.84 trillion** | 1.089 bn txns / RMB 711.9 bn |
| Bank-card cross-bank payment system (银行卡跨行支付系统) | UnionPay | 135.175 billion | RMB 173.60 trillion | 370 m txns / RMB 475.6 bn |
| Internet Banking Payment System (网上支付跨行清算系统, IBPS) | CNCC | 14.011 billion | RMB 110.77 trillion | 38.39 m txns / RMB 303.5 bn |
| High-Value Payment System (大额实时支付系统) | CNCC | 1.094 billion | RMB 4,950.72 trillion | 4.38 m txns / RMB 19.80 tn |
| Bulk Electronic Payment System (小额批量支付系统) | CNCC | 2.627 billion | RMB 60.58 trillion | 7.20 m txns / RMB 165.96 bn |
| Cross-border Interbank Payment System (CIPS) | CIPS Co. | 1.8843 million | RMB 33.93 trillion | 7,537 txns / RMB 135.7 bn |

Read this table with two cautions: (i) the numerator definitions differ (cards vs accounts vs high-value wholesale), so the columns are not a competitive ranking of like-for-like services; (ii) the value column shows the structural point — **CNCC's high-value system moves 19 times NUCC's value with 0.3% of its transaction count**. NUCC is a *count* business; CNCC's HVPS is a *value* business. This is the correct lens for §8 and for the companion CNCC guide.

⚠ Figures for 2020 onwards, per-channel, were not captured in this pass; the PBOC continues to publish the NUCC platform line in its annual reports but the specific later-year values were not retrieved.

---

## 8. The Other Chinese Retail Clearing Channels, Cross-Referenced
### 8.1 NUCC versus UnionPay

| Dimension | NUCC (网联) | UnionPay (银联) |
|---|---|---|
| Chartered focus | Bank-account-based online/mobile payment business of non-bank payment institutions | **Bank-card** clearing (issuing and acquiring across member banks); also a licensed clearing organisation |
| Legal status | 其他有限责任公司, PBOC-affiliated, 特许清算组织 | 中国银联股份有限公司, 特许清算组织 |
| Status in 断直连 | The default destination for the migrated online-payment traffic | **An explicitly permitted destination.** 209号文's scope was implemented as "migrate to 网联 **or** 银联"; the 断直连 term is defined as migrating to the NetsUnion **or** UnionPay platform |
| Evidence of coexistence | Payment institutions were reported to have connected to **both** and to have migrated their traffic gradually ⚠ | Same |
| Relationship | **High-level exchange meeting with UnionPay in Shanghai on 2026-02-06**; the two agreed to deepen industry co-ordination, support the central bank's mandate, establish a **regular catch-up mechanism** and continue normalised dialogue ⚠ (Baike citing a 百家号 report of 2026-02-06; the same meeting appears in the related-article listing of 中国证券网) |

The important, often-missed point: **the reform never made NUCC the sole retail clearing channel.** It made both licensed channels mandatory destinations and prohibited the bypass. That is why the "断直连" term itself is defined with an "or", why industry descriptions discuss payment institutions holding **two** virtual accounts (one at NUCC, one at UnionPay) with mapped quotas at each, and why a 2026 NUCC–UnionPay dialogue exists at all.
### 8.2 The banking-system retail channels

NUCC sits **beside**, not inside, the CNCC rails. For the bank-account-based retail and wholesale channels — CNAPS HVPS/BEPS, IBPS, and the Payment Connect work — read the companion guides rather than re-deriving them here:

- [`banking/cnaps_guide.md`](cnaps_guide.md) — CNCC, CNAPS (HVPS/BEPS), and the high-value/wholesale clearing architecture, including how CNCC's institution differs from NUCC.
- [`banking/ibps_payment_connect_guide.md`](ibps_payment_connect_guide.md) — IBPS (网上支付跨行清算系统) and the Payment Connect style interbank retail connectivity, including the bank-side access model.

Two boundary facts worth stating plainly:

1. **A single consumer transaction can traverse more than one of these systems.** The 断直连-era flow diagrams show merchant → acquirer → UnionPay → payment institution → UnionPay **or** NUCC → issuing bank, i.e. card-side clearing and account-side clearing in one economic transaction. Attribution of a transaction to "NUCC" or to "UnionPay" therefore depends on which leg is being counted.
2. **NUCC does not clear card transactions** and UnionPay does not clear the non-bank online-payment account business that NUCC was built for; the overlap is at the *institution* level (some payment institutions can route to either) and at the *market-share* level, not at the message level.
### 8.3 Barcode payment and the licensed-clearing requirement

⚠ A parallel rule tightened the same principle for barcode payments: the PBOC's 《条码支付业务规范 （试行）》 is reported to require that cross-bank barcode payment transactions be processed through the PBOC's interbank clearing system **or** a licensed clearing institution (i.e. UnionPay or NUCC), with implementation from **2018-04-01** (⚠ secondary practitioner summaries; the instrument itself was not retrieved in this pass). The direction is consistent with 209号文: any cross-institution retail retail payment must touch a licensed clearing institution.
### 8.4 Central bank digital currency (数字人民币 / e-CNY)

⚠ **Unverified, and stated as such.** The digital RMB is a PBOC-led programme with designated operating institutions; **no source retrieved in this research pass documents NUCC clearing retail e-CNY transactions**, and this guide therefore does not assert any NUCC role in e-CNY issuance, distribution or retail transfer. What *is* documented is adjacent and different: NUCC's 2026 exhibition material shows innovation-layer payment experiences (AI-assisted weighing and checkout, AR-glasses product recognition with voice-command payment, tap-to-pay and palm payment) and its president's interview discusses **agentic payment** — NUCC reports exploring trusted-agent identity and transaction attestation, piloting with four classes of market participants (commercial banks, payment institutions, vertical platforms and comprehensive e-commerce), with the first phase completed in a test environment. Treat digital-payment innovation at NUCC as innovation *on the existing account-based rails*, not as CBDC infrastructure, unless a primary source says otherwise.
### 8.5 Where the channel boundaries are blurry or have moved

| Ambiguity | Why it matters |
|---|---|
| **Dual connection** | An institution connected to both NUCC and UnionPay can, in principle, route account-based business either way; the "which channel" question is answered per institution on commercial and operational grounds, not purely by rule. |
| **Historical bank-run "clearing centres"** | Documented in 2017 reporting as institutions that acted as de facto clearers for third-party payment firms without a licence, treating downstream payment institutions as merchant customers of a bank's acquiring business. The 断直连 reform was intended to close this; the reform's completion statistics ("99%") do not break out such residuals. ⚠ |
| **Acquiring-side settlement** | The payment institution's settlement to the merchant is its own contractual layer; NUCC clears the inter-institution leg. Disputes that look "clearing" disputes are frequently acquiring-side disputes — determine which layer before escalating (§5.5). |
| **Combined statistics** | PBOC holiday releases aggregate UnionPay + NUCC. Any internal reporting that cites those numbers as NUCC's will overstate NUCC by the UnionPay share (§7.3). |
| **Cross-border connectivity naming** | The NUCC president's 2026 interview refers to China's cross-border **"跨境支付通" (cross-border payment connect)** model as one that opens fund-clearing channels and improves efficiency relative to long correspondent chains. The naming overlaps with the domestic "Payment Connect" vocabulary used in the companion IBPS guide — verify which system is meant before mapping a design. ⚠ |

---

## 9. Cross-Border Dimension
### 9.1 NUCC's documented cross-border links

| Item | Detail | Status |
|---|---|---|
| **NUCC + CIPS cooperation, Greater Bay Area** | **2020-07-17** — NUCC "携手CIPS助力粤港澳大湾区建设", i.e. working with the Cross-border Interbank Payment System on Greater Bay Area construction | ⚠ single encyclopaedia source (Baike milestone list, attributed to NUCC's own 大事记 page) |
| **MoU with Bank of Communications (交通银行)** on cross-border RMB business co-operation | A company-description snippet states NUCC signed a memorandum of understanding with Bank of Communications to start cross-border RMB business co-operation; **no date and no primary document obtained** | ⚠ single aggregator-sourced item — do not cite without independent confirmation |
| **CIPS itself** | A separate company and system (its own 2019 figures: 1.8843 million transactions, RMB 33.93 trillion — §7.4). NUCC's relationship to CIPS is co-operation, not hierarchy | ✅ for CIPS's separateness |

### 9.2 The Mastercard joint venture — 万事网联

This is NUCC's clearest documented move beyond online-payment clearing, and it is now well evidenced.

| Fact | Detail | Status |
|---|---|---|
| Entity | **万事网联信息技术（北京）有限公司** — Mastercard NetsUnion Information Technology (Beijing) Co., Ltd. | ✅ |
| Established | **2019-03-06**, in Beijing | ✅ official JV website; Baike |
| Registered capital | **RMB 1 billion** | ✅ official JV website |
| Founders | "**由万事达卡和网联清算公司合作发起**" — jointly initiated by Mastercard and NUCC | ✅ official JV website (mastercardnucc.com/company) — this is NUCC's formal role, stated by the JV itself |
| Bank-card clearing licence | Issued by the PBOC on **2023-11-17** | ✅ official JV website; media (澎湃 2023-11-19) |
| Business launch | **2024-05-09**, operating Mastercard-branded bank-card clearing business in China | ✅ official JV website |
| CEO | 郑笑非 | ✅ official JV website |
| Membership | Member (副会长单位, vice-president unit) of PCAC | ✅ official JV website |
| Ownership split | **Mastercard Asia/Pacific 50% + Mastercard International 1% + 网联科技有限公司 49%** — i.e. the NUCC-side shareholder is a NUCC affiliate entity, not NECESSARILY NUCC itself | ⚠ **single-sourced** (xueqiu post quoting company-registry data). The *founder* attribution is ✅; the *exact percentage split and the identity of the NUCC-side vehicle* are ⚠ |
| NUCC's engagement | NUCC's Party Secretary 罗延枫 spoke at the JV's business-launch celebration on **2024-06-13**, framing it as NUCC's contribution to financial opening | ✅ 上海证券报 via sina.com.cn / eastmoney, 2024-06-13 |
| Context | The Mastercard JV is the **second** Sino-foreign bank-card clearing JV, following 连通（杭州）技术服务有限公司 (the American Express-linked entity) | ⚠ secondary reporting |

⚠ Note the structural nuance: 万事网联 is a **bank-card clearing** institution. It is therefore closer in function to UnionPay than to NUCC, and its existence means the "licensed clearing" perimeter now contains a NUCC-affiliated participant in the *card* business. That is a genuine blurring of the NUCC / UnionPay boundary drawn in §8 — and it is worth stating that the two entities have distinct licences and lines of business even though the equity links cut across.
### 9.3 NUCC's own cross-border products

NUCC is a clear channel for two inbound-use products, whose volumes are published:

| Product | What it does | Cumulative (2019 – May 2025) |
|---|---|---|
| **外卡内绑** (foreign card, domestically bound) | Binds an overseas-issued bank card to a domestic mobile payment wallet so an inbound visitor can pay | **217 million transactions, RMB 32.524 billion; average ticket RMB 150.11** |
| **外包内用** (foreign wallet, domestically used) | Allows an overseas wallet to be used at domestic acceptance points | **162 million transactions, RMB 27.915 billion; average ticket RMB 172.31** |
| **Both** | Combined | **379 million transactions, RMB 60.439 billion** |

✅ Source: NUCC disclosure at the 2025 China International Finance Exhibition, via 上海证券报/中国证券网 2025-06-18 (which also gives the split above). Monthly run-rate: **May 2025 daily average 1.2193 million transactions / RMB 169.97 million, +185% and +135% year on year** ✅ same source.

More recent updates: **H1 2026 cross-border mobile payment transaction count +~80% and value +~50% y/y, daily average ~2 million transactions**; **外卡内绑 now serves nearly 200 countries and regions and 外包内用 covers 16 mainstream overseas wallets** (⚠ NUCC President 崔嵬 interview, 金融时报 via 腾讯新闻, 2026-09-10); and **May 2026 inbound foreign-visitor usage of the two products +70% y/y, with NUCC's cross-border business accounting for 70% of the inbound mobile-payment transaction market share** (⚠ operator material at the 2026 China International Finance Exhibition, 支付之家 2026-06-16).
### 9.4 Can a foreign institution touch NUCC?

**Largely: no — and it is better to say that plainly than to pad the answer.** What the evidence supports:

1. **Direct participation requires a Chinese licence.** NUCC's participants are PBOC-licensed banks and non-bank payment institutions (2024 rulebook, §6.1). A foreign bank or payment firm without a domestic licensed entity cannot be a NUCC participant.
2. **Practical foreign access is indirect, through domestic licensed participants**, in four documented shapes: (i) an overseas card bound into a domestic wallet (外卡内绑); (ii) an overseas wallet used domestically through NUCC-connected acceptance (外包内用); (iii) a foreign card scheme operating through a licensed domestic clearing JV (万事网联 for Mastercard; the earlier Amex-linked entity); and (iv) cross-border RMB flows through the correspondent/CIPS layer, where NUCC's documented role is co-operation, not participation.
3. **A foreign institution's realistic NUCC-touching interfaces are commercial, not technical**: being a wallet partner in 外包内用, a card issuer/network in the JV structure, or a correspondent/custody bank in the cross-border RMB chain. None of these involves a foreign entity connecting to NUCC's platform as a participant.
4. ⚠ **Unverified** and deliberately not asserted: any foreign-institution rulebook for NUCC access, any NUCC rule on remote/agent members, and NUCC's precise interface to CIPS. If a foreign institution's business case depends on direct NUCC connectivity, that case has not been evidenced in this pass and should be re-examined against primary documentation.
---

## 10. Operational, Compliance and Risk Reality for a Bank
### 10.1 What connecting actually requires

| Workstream | What the bank must do | Evidence basis |
|---|---|---|
| **Technical integration** | Dedicated-line connectivity to the platform; implement the message-exchange specification; build or adapt the transaction front-end, routing, response handling and status management; integrate with core account systems for debit/credit authorisation and with the payment-institution-facing service layer | ⚠ Q/NUC 102/105.2/106 enterprise standards exist but are not public (§4.5); the workstream is inferred from standard clearing access patterns and from §6.5 |
| **Certification and testing** | Offline/self-testing, then formal online testing and certification in the platform's test environment; certificate exchange; sign-off before production cut-over | ⚠ standard licensed-clearing practice; the operator's own published material refers to a joint industry stress-testing programme since 2024 whose reported system success rate rose from **95.27% to >99.99%** — participation in such testing is a real obligation (⚠ operator statement, 第一财经 2025-06-18) |
| **Settlement and liquidity arrangements** | Establish how the bank's own positions with the platform are settled; where the bank acts for payment institutions, support the PBOC centralised-custody arrangements (open/close reserve accounts per 银办发〔2018〕114号, funds transfers from the centralised account through UnionPay or NUCC); manage intraday liquidity around the platform's clearing sessions | ✅ 银办发〔2018〕114号 (verified text); ⚠ session timings not public (§5.4) |
| **Reconciliation** | Ingest platform files, match transaction-level and batch records, confirm positions, raise discrepancies inside the agreed window; build the operational tooling and the exception queue | ⚠ evidenced by participant implementations such as a provincial rural credit union putting a **batch-level reconciliation function** into production for its NUCC business (移动支付网 2022-04-24) |
| **Quota and account management** | Enquiry and adjustment of mapped/available quotas where the bank's client is a payment institution; supporting back-office operations against the platform's management interface | ⚠ secondary description of the post-reform model (§3.3) |
| **Fraud and dispute handling** | Fraud screening on the flow; the bank's own dispute intake; escalation into the platform's chartered dispute co-ordination and arbitration process | ✅ dispute handling is within NUCC's registered business scope (协调和仲裁业务纠纷); ⚠ the operational SLA is not public |
| **Operations and resilience** | 24×7 operational monitoring against the platform's availability; incident escalation; participation in the industry's peak-period assurance regime for holidays and shopping festivals | ⚠ platform material describes peak-assurance work as a normalised activity (§7.1) |

### 10.2 Compliance obligations that ride along

- **AML/CFT** — the platform's purpose includes feeding traffic to anti-money-laundering monitoring; NUCC reports building an industry anti-fraud system and risk-information-sharing mechanism using machine learning and knowledge graphs (⚠ operator statement, 第一财经 2025-06-18). For a bank, the obligation remains its own: customer due diligence, transaction monitoring, suspicious-activity reporting, and record-keeping on the flows it processes.
- **Data protection and data residency** — the flows carry personal and account data. Chinese obligations under the Personal Information Protection Law (PIPL), the Data Security Law, the Cybersecurity Law and financial-infrastructure/critical-information-infrastructure rules apply to the bank's handling, retention, cross-border transfer and security of that data. **This guide does not re-derive those rules**; see the repository's China data-governance material: [`technology/data/china_data_governance_frameworks.md`](../technology/data/china_data_governance_frameworks.md) and [`technology/data_governance_guide.md`](../technology/data_governance_guide.md).
- **Payment-institution supervision** — the bank's payment-institution clients are supervised under the 2024 rulebook (§6.1); the bank will be asked for cooperation in licensing, change-of-control and investigation processes, and must not design around them.
- **Reserve-fund compliance** — for a bank with payment-institution clients, the residual permitted accounts (cross-border RMB reserve, fund-sales settlement, prepaid-card, FX) are the only reserve accounts that may exist, and the rest must be closed (✅ 银办发〔2018〕114号).
- **Operational-resilience expectations** — as a participant in a designated financial infrastructure, the bank should assume it will be required to participate in joint stress testing and to evidence continuity arrangements.
### 10.3 The risk picture for a bank

| Risk | Nature | Mitigation direction |
|---|---|---|
| **Operational-continuity dependence on a single mandatory channel** | For the in-scope business there is no alternative to a licensed channel; if the platform (or the bank's sole connection to it) is unavailable, the business stops. NUCC itself calls business continuity "the platform's lifeline" | Dual channel where the institution is connected to both NUCC and UnionPay; internal cut-over/fallback design; participation in the platform's peak-assurance and stress-test activity |
| **Settlement risk** | The bank is exposed to the correctness and timing of net positions and to its clients' quota/position adequacy; a client's mapped-quota shortfall is a client-side failure that becomes a bank-side operational event | Intraday liquidity buffers; automated quota monitoring; clear client agreements on quota responsibility |
| **Concentration risk** | Every in-scope transaction depends on one operator, its six data centres and its standards; a systemic defect or standard change affects the whole industry simultaneously | Treat NUCC like other systemically important infrastructure in the operational-risk register; scenario-test industry-wide outage; keep standards-version change management funded |
| **Model and claim risk** | The operator's "100% availability / success rate / accuracy" claims are self-reported and definitionally sensitive; a bank that repeats them externally may find them unsupportable | Rely on the bank's own monitoring and reconciliation evidence for internal and regulatory reporting |
| **Regulatory-change risk** | The perimeter has moved repeatedly (2017 direct-connection ban; 2018–19 reserve centralisation; 2024 licence rulebook consolidation; 2023–24 card-clearing JV licences) | Keep the channel-boundary mapping (§8.5) under periodic review rather than treating it as fixed |
| **Data and cyber risk** | A mandatory high-volume channel carrying personal data concentrates both data-residency obligations and cyber exposure | Apply the repository's China data-governance guidance; test the channel's data flows in the bank's own DLP and monitoring estate |

---

## 11. Cymbal Bank Worked Example

> **Read this first.** Cymbal Bank is a fictional bank persona used in this repository's worked
> examples. **Every number, duration, rate and cost in this section is illustrative and fictional.** It
> is a planning sketch showing the *shape* of the work and the questions a bank must answer — not a
> benchmark, not a quotation, and not derived from any published NUCC tariff, participant schedule or
> third-party estimate. Nothing here should be used as a budget without replacing the numbers with the
> bank's own measurements and the platform's own documentation.
### 11.1 The obligation as Cymbal Bank assesses it

Cymbal Bank operates in China through a licensed domestic subsidiary that provides account and settlement services to non-bank payment institutions. The assessment question: *what must Cymbal Bank build and run to meet its obligations as a bank participating in the NUCC-cleared, bank-account-based online payment business, and which of its counterparties' payment flows must be processed on that platform?*

Scope conclusion (illustrative reasoning, grounded in §6): Cymbal Bank's obligation is **on the funding/settlement side**, not as an originator. Its payment-institution clients originate; Cymbal Bank holds the customer account that is debited, returns the result to the platform, and settles its net positions. Cymbal Bank therefore needs participant-grade connectivity, certification, reconciliation and liquidity management — and it needs the same discipline for the flows of clients that route to UnionPay rather than NUCC (§8.1).
### 11.2 Integration requirements (illustrative scope)

| Component | Illustrative build | Illustrative effort |
|---|---|---|
| Dedicated-line connectivity (primary + backup) | Two independent circuits into the platform's regional entry points; capacity sized to the bank's peak share | 2 circuits; 3 months lead time |
| Message gateway | Implement the platform's message exchange standard for the transaction types the bank will serve; signature/verification handling; message archival | 4–6 engineer-months |
| Core account interface | Real-time debit/hold/credit authorisation paths from the gateway to the account platform, with idempotency and reversal handling | 5–8 engineer-months |
| Quota and position management | Enquiry and adjustment services for client mapped/available quotas; alerting on shortfall | 2–3 engineer-months |
| Reconciliation engine | File intake, T+1 batch matching, exception queue, confirmations back to the platform | 3–4 engineer-months |
| Monitoring and operations console | End-to-end transaction tracing, volume/peak dashboards, incident workflows, on-call runbooks | 2–3 engineer-months |
| Non-functional | Peak-load performance testing against an illustrative 3× current peak; failure and fallback drills | 1–2 months elapsed |

### 11.3 Certification path (illustrative)

| Phase | Illustrative duration | Exit criterion |
|---|---|---|
| 1. Participation documentation and agreements | 6–10 weeks | Signed agreements; identifiers issued |
| 2. Test-environment connectivity and offline self-test | 4–6 weeks | Internal test pack passes |
| 3. Formal online certification testing | 4–8 weeks | Platform certification sign-off |
| 4. Certificate/credential exchange and production readiness review | 2–3 weeks | Production credentials issued |
| 5. Production cut-over and volume ramp | 6–12 weeks | Target share of traffic live; reconciliation clean for 5 consecutive cycles |

### 11.4 Message handling and settlement/liquidity design (illustrative)

- **Message handling**: strict schema conformance; deterministic idempotency keys; explicit duplication detection at the gateway; bounded retry with reconciliation-based repair rather than blind resubmission; immutable archival of request/response pairs for audit and dispute evidence.
- **Clearing/liquidity**: the bank models the platform's real-time clearing of positions against scheduled final settlement (§5.4) and sizes an intraday liquidity buffer for a peak day. Illustrative figures for Cymbal Bank's China book: **RMB 4.0 billion peak-day outbound throughput; RMB 300 million intraday buffer floor; RMB 900 million intraday buffer on the three highest-peak days (CNY's Eve, Spring Festival days and Double 11)** — *illustrative only*.
- **Client quota responsibility**: contracts make the payment institution responsible for mapped-quota adequacy; the bank monitors and alerts but does not fund client shortfalls.
### 11.5 Reconciliation and exception handling (illustrative)

- Three-way match: platform transaction records ↔ bank account entries ↔ payment-institution settlement files.
- Exception classes: unmatched transaction; amount mismatch; duplicate; timing/date-cut boundary; status mismatch (success/failure disagreement). Illustrative target: **≥99.9% auto-matched within the first matching cycle; all exceptions cleared within one business day; unresolved items escalated to the platform's dispute process after two cycles** (§5.5).
- Reconciliation is operated as a production service with its own on-call rota — not a finance afterthought.
### 11.6 Fraud and dispute operating model (illustrative)

- Pre-authorisation fraud screening on the flow (velocity, device/account anomalies, blacklists);
- AML monitoring integration, with escalations to the bank's FIU reporting process;
- Client-facing dispute intake routed with a decision tree that first identifies the *layer* of the dispute (acquiring-side vs inter-institution clearing — §8.5);
- Use of NUCC's risk-information-sharing and dispute-co-ordination mechanisms where bilateral resolution fails; the platform's arbitration function is a chartered capability, not an escalation of last resort invented by the bank.
### 11.7 Testing and cutover plan (illustrative)

| Stage | Illustrative content |
|---|---|
| Unit and integration | Message conformance, idempotency, reversal, timeout and partial-failure handling |
| Non-functional | Peak load at 3× current peak; sustained-load soak; failure injection on the dedicated line and on the account interface |
| Client parallel run | One pilot payment institution processed in parallel with its existing route for 2 weeks, comparing outputs |
| Cutover | Phased traffic ramp by client (10% → 50% → 100%) with reconciliation gated between phases; documented rollback to the client's prior route for one week after full cutover |
| Post-cutover | 30-day heightened monitoring; peak-day dry-run before the next CNY window |

### 11.8 Cost and effort comparison (illustrative, fictional)

The comparison below is **Cymbal Bank's own planning table with invented numbers**. It exists to show the relative weight of the options, not to estimate a real price.

| Option | Illustrative one-off cost | Illustrative annual run cost | Illustrative elapsed time | Cymbal Bank's assessment |
|---|---|---|---|---|
| Build direct participant connectivity (NUCC + UnionPay) | RMB 18 million | RMB 6.5 million | 9–12 months | Highest control, highest fixed cost; justified only for a bank with material flow |
| Connect via an existing participant/aggregator | RMB 5 million | RMB 9 million | 4–6 months | Faster, lower fixed cost, adds an intermediary dependency and a fee margin; weakens the bank's direct reconciliation evidence |
| Do the minimum (single channel, thin reconciliation) | RMB 7 million | RMB 3 million | 5–7 months | Cheapest on paper, but concentrates operational risk on one channel and leaves exception handling manual — **not recommended** |

### 11.9 Risk assessment (illustrative)

| Risk | Illustrative assessment | Response |
|---|---|---|
| Platform unavailability | High impact, low likelihood per the operator's claims, unverifiable independently | Dual-channel where possible; fallback plans; stress-test participation |
| Certification slippage | Medium impact, medium likelihood | Parallel-run plan with the pilot client; contractual ramp dates |
| Reconciliation failure | High impact on client confidence | Auto-matching KPIs monitored in production; manual fallback desk for one cycle |
| Liquidity shortfall at peak | Medium impact | Buffer floor plus intraday monitoring; client quota obligations in contract |
| Data-protection non-compliance on the flow | High impact | Data-flow mapping per the repository's China data-governance guidance; retention and cross-border transfer review |

### 11.10 What Cymbal Bank cannot establish before committing

This is the honest list, and it mirrors §13:

1. **The settlement sessions, cut-off times and value dates** the bank must plan liquidity around (§5.4 is not public).
2. **The tariff** — no published schedule exists; the cost line above is invented (§5.6).
3. **The full interface specification** and any mandatory cryptographic requirements (§4.5 — the Q/NUC standards are not public; one widely copied blog claim about SM2/SM3/SM4 and a "V3.2" interface specification was **not corroborated** and is listed as unverified).
4. **The complete ownership picture of the platform** — the reported 10% stake transfer ("floor price RMB 866 million") is headline-only in this pass and its effect on governance is unknown (§6.3).
5. **Whether the operator's availability and success-rate claims would withstand the bank's own audit standard** — they are self-reported and definitionally sensitive (§4.2).
6. **The current, exact participant population and licensing categories** for the bank's specific client cohort under the 2024 rulebook (§6.1).
---

## 12. Claims Audit

Every material claim in this guide, with its marker, its best source and that source's date and quality. "Quality" is judged as: **primary** (the issuing authority's own text), **trade** (specialist payments press with an identifiable journalist/date), **national** (major national media), **secondary aggregator** (encyclopaedia or company-information site), **weak** (practitioner blog, document-share listing, legal-summary content farm).

| # | Claim | Marker | Best source (date) | Quality |
|---|---|---|---|---|
| 1 | 银支付〔2017〕209号 issued 2017-08-04 by the PBOC Payment & Clearing Division, titled 《关于将非银行支付机构网络支付业务由直连模式迁移至网联平台处理的通知》 | ✅ | 移动支付网, with photograph of the notice (2017-08-04) | trade |
| 2 | From 2018-06-30 all bank-account-involving online payment business of payment institutions must be processed via NUCC | ✅ | 移动支付网 (2017-08-04); Baike entry 22167842; people.com.cn (2017-08-07); 21世纪经济报道 (2017-08-04) | trade + national |
| 3 | Banks and payment institutions to complete connection/migration preparation by 2017-10-15 | ✅ | as row 2 | trade + national |
| 4 | A variant report gives the preparation deadline as 2018-10-15 | ⚠ flagged, unresolved | 界面 retrospective page (2022 vintage) | national (contradicted by four contemporaneous sources) |
| 5 | NUCC established 2017-08-29; registered capital RMB 2 billion | ✅ | Baike 22111484 (2026-09-01 revision) + 企查查 + 爱企查 listings | secondary aggregators (three agreeing) |
| 6 | PBOC approved preparatory establishment 2016-10-20; trial run started 2017-03-31 | ✅ | Baike 22111484; zh.wikipedia; 移动支付网/亚洲财经 (2018-03-31) | secondary + trade |
| 7 | 45 institutions signed the founding agreement on 2017-08-02, including CNCC, Tenpay, Alipay and UnionPay Merchant Services | ✅ | 经济观察网 via finance.sina.cn (2017-08-05) | national |
| 8 | Seven PBOC-affiliated institutions together hold 37% (RMB 760 million); PCAC holds 3% and votes for ineligible small institutions; 29 payment institutions hold 63% | ✅ | 移动支付网 (2017-08-04); 37% and 9.61% independently in 经济观察网 (2017-08-05) | trade + national |
| 9 | Alipay and Tenpay each hold 9.61%; largest other payment-institution holder 网银在线 at 4.71%; smallest 易联支付 and 捷付睿通 at 0.14% each | ✅ (9.61% ✅; the 4.71%/0.14% figures ⚠) | 移动支付网 (2017-08-04); 经济观察网 (2017-08-05) for 9.61% | trade + national |
| 10 | CNCC's individual stake is 12% (RMB 240 million); 梧桐树投资平台 holds 10% (RMB 200 million) | ⚠ | 爱企查 equity-structure snippet | secondary aggregator |
| 11 | 10% of NUCC equity was listed for transfer with a floor price of RMB 866 million | ⚠ headline only | cnstock.com related-article listing (2025–26) | national headline; article not retrieved |
| 12 | Capital paid in three instalments of 50%/30%/20%; first RMB 1 billion in place by August 2017 | ⚠ | 经济观察网 (2017-08-05); 移动支付网 (2017-08-04) | national + trade |
| 13 | NUCC is a PBOC-affiliated enterprise and a licensed clearing organisation; the world's largest retail payment clearing institution by transaction count | ✅ | pbc.gov.cn recruitment notice (2025-12-12); Baike; operator material | primary (PBOC) |
| 14 | NUCC was organised by PCAC on a 共建、共有、共享 (co-build, co-own, co-share) basis | ⚠ | 爱企查 company description; consistent with 经济观察网 | secondary aggregator |
| 15 | Business scope includes building/operating the national unified clearing system, funds clearing for non-bank payment institutions' online payment, and **co-ordinating and arbitrating business disputes** | ✅ | Baike 22111484 reproducing the registered scope | secondary aggregator |
| 16 | 银办发〔2018〕114号: monthly step-up from 2018-07-09 to 100% centralised deposit by 2019-01-14; deposits on the second Monday; previous month's average daily balance as the base | ✅ | finance.sina.cn (2018-06-29) reproducing the notice text; m.sohu/大宗传媒 (2018); gov.cn/Xinhua (2019-01-09) | primary text + national |
| 17 | Transfers from the centralised reserve account must be handled via UnionPay or NUCC; other reserve accounts closed by 2019-01-14 | ✅ | 银办发〔2018〕114号 text via finance.sina.cn (2018-06-29) | primary text |
| 18 | PBOC raised the reserve centralisation ratio to ~50% in early 2018 and required partial deposit in 2017 | ✅ | gov.cn/Xinhua quoting PBOC officials (2019-01-09) | primary/national |
| 19 | The exact 2017 phase-one reserve instrument number/date | ❌ not verified | — | — |
| 20 | 99% of payment business between payment institutions and banks was processed via NUCC/UnionPay by end-2018; 断直连 and reserve centralisation are directly related | ✅ | gov.cn/Xinhua, PBOC officials 温信祥/范一飞 (2019-01-09) | primary/national |
| 21 | All 115 network-payment-licensed institutions and 424 banks connected; 99% of existing cross-institution business migrated; daily funds-class transactions >1 billion | ✅ | 界面新闻 2758152, reporting NUCC's New Year message (2019) | national (relaying operator) |
| 22 | 2018 Q2: 8.146 billion transactions, RMB 2.7 trillion | ✅ | PBOC Q2-2018 statistics via 金融界 (2018-08-21) | primary (PBOC statistic) |
| 23 | 2018-03-31: cumulative >10 billion transactions / RMB 2.83 trillion; 343 banks and 105 payment institutions connected; single-day peak >100 million transactions; peak >7,000 TPS | ⚠ | 亚洲财经 via sohu (2018-03-31) | trade relay |
| 24 | 2019: 397.542 billion transactions, RMB 259.84 trillion; 534 commercial banks and 115 payment institutions connected; daily average 1.089 billion | ✅ | PBOC *2019 Payment System Operation Overall Situation* via 199IT (2020-03-17) | primary (PBOC statistic) |
| 25 | 2024: >1 trillion transactions, RMB 520 trillion; 2025 CNY's Eve peak 110,000 tps; availability/success/accuracy "100% for consecutive years" | ⚠ operator disclosure | 第一财经 (2025-06-18); 上海证券报/中国证券网 (2025-06-18); 北京商报 via sohu & 新浪财经 (2025-06-18) — all reporting the 2025 China International Finance Exhibition | national, relaying operator |
| 26 | >3,000 commercial banks and payment institutions connected | ⚠ operator disclosure | same as row 25 | national, relaying operator |
| 27 | Platform capability >300,000 tps, average latency 200 ms, 三地六中心 distributed cloud architecture, five-layer DR | ⚠ operator disclosure | 第一财经 (2025-06-18) | national, relaying operator |
| 28 | 2019–2025-05 cross-border: 外卡内绑 + 外包内用 379 million transactions, RMB 60.439 billion (split: 217m/RMB 32.524bn and 162m/RMB 27.915bn) | ⚠ operator disclosure | 上海证券报/中国证券网 (2025-06-18) | national, relaying operator |
| 29 | 2025 Spring Festival (28 Jan–4 Feb): UnionPay + NUCC combined 25.416 billion transactions, RMB 9.78 trillion | ✅ | PBOC data via 中国新闻网 (2025-02-05) | primary statistic, combined channels |
| 30 | 2026 Spring Festival (15–23 Feb): combined 39.302 billion transactions, RMB 13.12 trillion, daily average +37.45% y/y | ✅ | 新华社 (2026-02-24) | primary/national, combined channels |
| 31 | Single-day peak >4.415 billion transactions and daily average 3.8 billion since 2019; 2026 CNY's Eve peak >125,000 tps | ⚠ operator disclosure | 支付之家 (2026-06-16) relaying 2026 exhibition material; 中国金融新闻网/金融时报 interview with NUCC President 崔嵬 via 腾讯新闻 (2026-09-10) | trade + national |
| 32 | H1 2026 cross-border mobile payment count +~80%, value +~50%, daily average ~2 million transactions; 外卡内绑 serves ~200 countries; 外包内用 covers 16 overseas wallets | ⚠ operator statement | 金融时报 interview with NUCC President 崔嵬 via 腾讯新闻 (2026-09-10) | national |
| 33 | NUCC exploring agentic-payment rules; pilot with banks, payment institutions, vertical platforms and e-commerce; phase 1 completed in a test environment | ⚠ operator statement | same as row 32 | national |
| 34 | 万事网联 founded by Mastercard and NUCC on 2019-03-06 with RMB 1 billion capital; bank-card clearing licence 2023-11-17; opened 2024-05-09; CEO 郑笑非; PCAC vice-president unit | ✅ | mastercardnucc.com/company (official JV site, retrieved 2026-09-15) | primary (JV's own site) |
| 35 | 万事网联 ownership: Mastercard Asia/Pacific 50%, Mastercard International 1%, 网联科技有限公司 49% | ⚠ | xueqiu post quoting registry data | weak/secondary |
| 36 | NUCC Party Secretary 罗延枫 spoke at the 万事网联 business-launch celebration on 2024-06-13 | ✅ | 上海证券报 via finance.sina.com.cn and eastmoney (2024-06-13) | national |
| 37 | NUCC + CIPS cooperation for the Greater Bay Area, 2020-07-17 | ⚠ | Baike 22111484 milestone list (attributed to NUCC's own 大事记) | secondary aggregator |
| 38 | NUCC signed an MoU with Bank of Communications on cross-border RMB business cooperation | ⚠ single-sourced, undated | 爱企查 company description snippet | secondary aggregator |
| 39 | NUCC–UnionPay high-level exchange in Shanghai on 2026-02-06; regular dialogue mechanism agreed | ⚠ | Baike citing 百家号 (2026-02-06); meeting also appears in cnstock related-article listing | secondary aggregator |
| 40 | 断直连 means migrating to 网联 **or** 银联; UnionPay remains the bank-card clearing institution | ✅ | Baike 断直连 entry (2025-02-21 revision); 银办发〔2018〕114号 routing rule | secondary + primary text |
| 41 | 2017 stated pricing principle: cost-saving, no added market burden, profit not the point, 保本微利, possibly free initially | ✅ | 新浪财经/看点 platform profile (2017-06-07) | national |
| 42 | Later charging structure with a volume threshold, monthly package pricing for the two largest institutions, usage-based above | ⚠ | zhihu practitioner account (2019 vintage) | weak |
| 43 | Clearing model is 实时清算、定时结算 with a mapped/available quota virtual account; centralised reserve earmarked (圈存) to NUCC/UnionPay; available quota = mapped quota + receipts − payments | ⚠ | product-practitioner write-up via sohu/woshipm (2023-07-12) | weak |
| 44 | Detailed settlement timings: day-cut at T-day 24:00; clearing in early T+1; results to institutions at T+1 10:00 for confirmation | ⚠ | practitioner write-ups (zhihu/sohu/163.com, 2020–23 vintage), single author lineage | weak |
| 45 | NUCC publishes Q/NUC enterprise standards (102-2022 message exchange V1.5; 106-2022 technical access guide V1.4; 105.2 dedicated-line access, bank/payment-institution parts) | ⚠ | document-listing pages (book118, CSDN library, Baidu Wenku) | weak, but version/year markers are internally coherent |
| 46 | Mandatory SM2/SM3/SM4 cryptography, a "《网联平台接口规范》V3.2" with twelve mandatory fields, and a five-year message-retention obligation | ❌ rejected as unsupported | a single syndicated blog post; no corroboration | weak |
| 47 | NUCC platform described as three core systems (transaction/clearing/reconciliation) plus a front-end and quota mapping | ⚠ | CSDN developer posts (retrieval failed in this pass; described from search snippets) | weak |
| 48 | NUCC was a "first-batch initiator" of the Digital Transformation Partner Action (2020-05-13); 一键绑卡 launched 2019-06-10; 2019 Bank Technology Development Award Special Prize (ceremony 2020-09-24); ISO standard participation 2021-10-15; China Cyberspace Security Association member 2025-04-22 | ⚠ | Baike 22111484 milestone list and its citations (人民网, PBOC, association notice) | secondary aggregator citing primary |
| 50 | NUCC's 2026 exhibition material claims 3,000+ banks, 100+ payment institutions, >1 billion users, >200 million merchants, and 70% of inbound mobile-payment market share | ⚠ operator disclosure | 支付之家 (2026-06-16) | trade, relaying operator |
| 51 | Barcode (条码) cross-bank transactions must be processed via the PBOC interbank clearing system or a licensed clearing institution, effective 2018-04-01 | ⚠ | practitioner summaries (zhihu, ailegal.baidu) | weak |
| 52 | PBOC's 2019 statistics show NUCC clearing RMB 259.84 trillion against total non-bank online payment business of RMB 249.88 trillion — an unresolved definitional inconsistency | ⚠ flagged | same source as row 24 | primary statistic, definitional caveat noted |

---

## 13. What Could Not Be Verified

This section records every material gap honestly. It is a first-class part of the guide, not a disclaimer.
### 13.1 The official site and its archive

- **`https://www.nucc.com/` was unreachable through every engine attempted** in this research pass. Firecrawl reported "All scraping engines failed" for both `nucc.com` and the alternate `nucc.cn` (the same failure was reproduced through a second extraction tool during this pass), and direct HTTP fetches to `www.nucc.com/aboutUs` returned zero bytes. A corporate website's About page, governance page, standards library and participant notices are therefore **not** available as sources in this guide. ⚠
- **The Internet Archive was offline during this pass.** The Wayback availability API returned no archived snapshots for `nucc.com`, and CDX/`web.archive.org` requests returned an "Internet Archive: Temporarily Offline" page. The previously known Wayback snapshot URL of the NUCC home page (`http://web.archive.org/web/20260511030006/https://www.nucc.com/`) **could not be retrieved**, and this guide does not claim to have used it. This is a substantive gap: NUCC's own milestone list, its published standards index and its participant notices were, until the recent past, partly reachable through the archive. ❌ not retrieved.
- **Internet search was effectively unavailable**: the built-in web-search tool returned empty result sets for every query (English and Chinese, trivial and specific); Google, DuckDuckGo, Bing, Mojeek, Brave, Ecosia, Startpage, Yandex, Sogou and searx instances were blocked, captcha'd or returned junk. Findings in this guide come from the working sources listed in §15.
### 13.2 Ownership and corporate records

- **The complete shareholder list with all 45 names and individual percentages** was not obtained. What is verified is the *aggregate structure* (37% / 3% / 63%), the two large payment-institution holdings (9.61% each) and the named largest/smallest non-Big-Two holders (4.71%, 0.14%). The full register is behind 企查查/爱企查/天眼查 paywalls, and those pages were not retrievable in this pass.
- **The reported 10% equity transfer** (headline: floor price RMB 866 million, consistent with the 梧桐树投资平台 holding) was seen only as a related-article headline; the article, the exchange disclosure and any completion status were not retrieved. Whether that stake has changed hands, and with what effect on governance, is **unknown**.
- **Whether NUCC itself or an affiliated vehicle (网联科技有限公司) holds the 49% in the Mastercard JV** is unresolved: the official JV site says the JV was co-initiated by Mastercard and NUCC, while a registry-derived secondary source names 网联科技有限公司 as the shareholder. Both statements may be true (a subsidiary acting as the holding vehicle), but the guide does not assert which.
### 13.3 Regulation and instruments

- **The exact 2017 phase-one customer-reserve-funds instrument** (number, date, title) — the PBOC's own 2019 account confirms a 2017 requirement to deposit part of customer reserves in designated accounts, but the instrument itself was not captured. **Do not cite a document number for it.**
- **The unresolved 2017-10-15 vs 2018-10-15 preparation-deadline discrepancy** in reporting of 209号文 (§2.2). The contemporaneous weight favours 2017-10-15; the variant is recorded rather than resolved.
- **The text of 《条码支付业务规范（试行）》** itself — only secondary summaries were obtained.
- **NUCC's participation rules, service terms, tariff schedule and dispute SLA** — none is public.
### 13.4 Platform, technology and interfaces

- **The content of the Q/NUC enterprise standards** (message exchange, technical access, dedicated-line access) — the *existence, titles, versions and years* surfaced, but the documents themselves are participant-facing and were not obtained. Any specific mandated field, algorithm, protocol or retention period is therefore **unverified**, and the widely copied blog claim about SM2/SM3/SM4 and a "V3.2 interface specification" is recorded as **rejected as unsupported** (§4.5, §12 row 46).
- **The three-core-system description** of the platform (transaction / clearing / reconciliation plus front-end and quota mapping) rests on developer-blog material that **could not be fetched in this pass** (the CSDN pages failed on two attempts) and is retained only with a weak-evidence label.
- **Settlement sessions, cut-off times, value dates and the reconciliation file calendar** are not public; the T/T+1 timings quoted in §5.4 are a single practitioner lineage and are labelled weak.
- **NUCC's technical relationship to CIPS** and the exact meaning of the "跨境支付通" (cross-border payment connect) model referenced by NUCC's president in 2026 — not established.
### 13.5 Volumes and statistics

- **A continuous year-by-year transaction series for 2020–2023** was not obtained. The PBOC's annual *Payment System Operation Overall Situation* reports contain a platform line in some years, but the 2020–2023 values were not retrieved in this pass; only event-based figures (National Day 2021, Double 11 2021, New Year 2022) and the 2024/2026 operator disclosures are captured.
- **The basis of the published figures** — net versus gross, and which message classes are counted — is not disclosed by any source found.
- **The 2019-era reconciliation of NUCC's cleared value against PBOC's total non-bank online payment value** cannot be resolved from public material (§7.3).
- **No independent audit or regulatory re-publication of the operator's 2024 figures** (1 trillion transactions / RMB 520 trillion) was found; the figure is an operator disclosure made at a trade exhibition, and it should be cited as such.
### 13.6 Sources that were reachable in principle but blocked in practice

- **xueqiu.com** — WAF JSON block (the 9-year retrospective and the equity-transfer posts were seen only as search snippets).
- **企查查 / 爱企查 / 天眼查** detail pages — paywalled or JS-gated; only snippets were obtained.
- **Baidu and Quark/Shenma search** — worked briefly and then served 百度安全验证 / `_____tmd_____` JS challenges for the remainder of the pass, despite spaced retries.
- **Bing** — returned unrelated junk results for finance queries even when it responded at all; its output was discarded and is not cited anywhere in this guide.
- **CSDN blog articles** (the two bank-side integration write-ups queued for this pass) — both scrape attempts failed.
- **mastercardnucc.com** — the JV's *company* page was retrieved successfully through one tool and failed through another; only retrieved content is cited.
---

## 14. Glossary

Chinese–English term pairs used in this guide. Where a term has a common abbreviation, it is given.

| Chinese | English | Notes |
|---|---|---|
| 网联 | NetsUnion | Short form of the clearing platform and its operator |
| 网联清算有限公司 | NetsUnion Clearing Corporation (NUCC) | The operator company, established 2017-08-29 |
| 非银行支付机构网络支付清算平台 | Non-Bank Payment Institution Online Payment Clearing Platform | The chartered platform; "网联平台" in short form |
| 非银行支付机构 | non-bank payment institution | Licensed payment firm that is not a bank |
| 中国人民银行 (人民银行) | People's Bank of China (PBOC) | Central bank and supervisor |
| 支付结算司 | PBOC Payment & Clearing Division | Issued 银支付〔2017〕209号 |
| 中国人民银行办公厅 | PBOC General Office | Issued 银办发〔2018〕114号 |
| 清算总中心 | China National Clearing Center (CNCC) | PBOC arm operating CNAPS; a separate entity from NUCC |
| 银联 | UnionPay (China UnionPay) | Incumbent bank-card clearing institution; the alternative licensed channel in 断直连 |
| 特许清算组织 | licensed (franchise) clearing organisation | The status shared by NUCC and UnionPay |
| 支付清算协会 | Payment & Clearing Association of China (PCAC) | Organised NUCC on a co-build/co-own/co-share basis; holds 3% and votes for small institutions |
| 断直连 | the "disconnect" (direct-connection severing) reform | Migration of bank-account online payment to 网联 or 银联 |
| 直连 | direct connection | Bilateral payment-institution-to-bank links, prohibited for the in-scope business |
| 备付金 | customer reserve funds | Customer money held by a payment institution |
| 备付金集中存管账户 | centralised customer-reserve custody account | The single PBOC account per payment institution post-reform |
| 客户备付金全部集中交存 | centralised deposit of customer reserve funds in full | 100% by 2019-01-14 per 银办发〔2018〕114号 |
| 映射额度 | mapped quota | Funds earmarked (圈存) to NUCC/UnionPay from the centralised account |
| 可用额度 | available quota | Mapped quota + receipts − payments within a clearing session |
| 圈存 | earmarking / ring-fencing | Authorising part of the PBOC-held balance to the clearing institution |
| 实时清算、定时结算 | real-time clearing, scheduled settlement | The post-reform clearing model |
| 资金类交易 | funds-class transactions | Transaction class the published volumes usually count |
| 一键绑卡 / 一键绑账户 | one-click card/account binding | User-experience product |
| 外卡内绑 | foreign card, domestically bound | Inbound visitor binds an overseas card to a domestic wallet |
| 外包内用 | foreign wallet, domestically used | Overseas wallet used at domestic acceptance points |
| 跨境支付通 | cross-border payment connect | Cross-border connectivity model referenced by NUCC in 2026 |
| 数字人民币 | digital RMB (e-CNY) | PBOC-led digital currency; no verified NUCC clearing role in this pass |
| 清算 | clearing | Computing and netting inter-institution positions |
| 结算 | settlement | Actual movement of funds |
| 对账 | reconciliation | Matching platform, bank and client records |
| 专线接入 | dedicated-line access | Physical connectivity mode for participants |
| 协调和仲裁业务纠纷 | co-ordinating and arbitrating business disputes | A chartered NUCC function |
| 非银行支付机构监督管理条例 | Regulations on the Supervision and Administration of Non-Bank Payment Institutions | Current licensing framework |
| 储值账户运营 / 支付交易处理 | stored-value account operation / payment transaction processing | The two business-type families under the 2024 rulebook |
| 支付业务许可证 | payment business licence | The PBOC licence held by participants |
| 万事网联信息技术（北京）有限公司 | Mastercard NetsUnion Information Technology (Beijing) Co., Ltd. | The Mastercard–NUCC card-clearing JV |
| 中国国际金融展 | China International Finance Exhibition | Venue of the 2025 and 2026 operator disclosures |
| 共建、共有、共享 | co-build, co-own, co-share | The principle under which NUCC was organised |

---

## 15. Cross-References and Further Reading
### 15.1 Repository paths (companion guides)

- [`banking/cnaps_guide.md`](cnaps_guide.md) — the PBOC's China National Clearing Center (清算总中心, CNCC) and CNAPS (HVPS/BEPS). Use it for the high-value and bulk retail interbank rails and for the institutional contrast with NUCC (§1.4, §7.4, §8.2).
- [`banking/ibps_payment_connect_guide.md`](ibps_payment_connect_guide.md) — IBPS (网上支付跨行清算系统) and the bank-side Payment Connect connectivity model, including the bank access and testing patterns referenced in §6.5 and §8.2.
- [`technology/data/china_data_governance_frameworks.md`](../technology/data/china_data_governance_frameworks.md) — China's data-protection and data-governance framework (PIPL, Data Security Law, Cybersecurity Law, cross-border transfer). Cross-referenced from §10.2 rather than re-derived here.
- [`technology/data_governance_guide.md`](../technology/data_governance_guide.md) — the general data-governance guide, for the operating model behind the obligations in §10.2.
### 15.2 External sources actually used in this pass

Primary and official:

- `https://www.pbc.gov.cn/renshisi/144501/144513/5836227/5836240/2025121214564112292/index.html` — NUCC 2026 campus recruitment notice on the PBOC's own site (published 2025-12-12). Retrieved 2026-09-15. The source for NUCC's official characterisation as a PBOC-affiliated financial infrastructure operator.
- `https://www.gov.cn/zhengce/2019-01/09/content_5356286.htm` — Xinhua report of the State Council Information Office briefing, PBOC Deputy Governor 范一飞 and Division Director 温信祥 (2019-01-09).
- `https://www.gov.cn/gongbao/2024/issue_11546/202408/content_6970979.html` — 《非银行支付机构监督管理 条例实施细则》, PBOC Order 〔2024〕第4号 (State Council Gazette, 2024 Issue 24). Retrieved 2026-09-15.
- `https://finance.sina.cn/bank/yhgd/2018-07-01/detail-iheqpwqz1422000.d.html` — full text of 银办发〔2018〕114号 (2018-06-29).
- `https://mastercardnucc.com/company` — the Mastercard–NUCC JV's official company page. Retrieved 2026-09-15.

Trade and national press:

- `https://m.mpaypass.com.cn/news/201708/04162524.html` — 移动支付网 report on 银支付〔2017〕209号 (2017-08-04), including a photograph of the notice.
- `http://www.mpaypass.com.cn/news/201708/04102934.html` — 移动支付网, "45家网联股东出资明细" (2017-08-04): the shareholder structure and percentages.
- `https://finance.sina.cn/2017-08-05/detail-ifyitayr9211443.d.html` — 经济观察网 via 新浪财经, "清算市场变局 网联股东明细出炉" (2017-08-05): 45 signatories, 37%/9.61%, capacity targets, direct connection analysis.
- `https://cj.sina.cn/article/detail/2109032275/275997` — 新浪财经/看点, "1分钟读懂网联" (2017-06-07): pricing principle, capacity plan, launch timeline.
- `https://www.sohu.com/a/226862271_216075` — 亚洲财经 via 搜狐 (2018-03-31): cumulative 10 billion transactions, 3 locations/6 centres, >7,000 TPS peak.
- `https://m.yicai.com/news/102676506.html` — 第一财经 (2025-06-18): the 2024 volume disclosure and related operator claims.
- `https://www.cnstock.com/commonDetail/457215` — 上海证券报·中国证券网 (2025-06-18): the 2025 exhibition disclosure including the cross-border product split and rural/treasury figures.
- `https://finance.sina.com.cn/jjxw/2025-06-18/doc-infanfyn7884602.shtml` — 北京商报 via 新浪财经 (2025-06-18): same disclosure.
- `https://www.sohu.com/a/905444131_115865` — 北京商报 via 搜狐 (2025-06-18): same disclosure.
- `https://www.199it.com/archives/1021724.html` — reproduction of the PBOC's *2019 Payment System Operation Overall Situation* (2020-03-17), source of the cross-channel comparison in §7.4.
- `https://www.chinanews.com/cj/2025/02-05/10363666.shtml` — 中国新闻网 (2025-02-05): PBOC Spring Festival 2025 combined UnionPay/NUCC data.
- `https://www1.xinhuanet.com/20260224/8299b33a60484619a482abb5287d0fcc/c.html` — 新华社 (2026-02-24): Spring Festival 2026 combined payment data.
- `https://news.qq.com/rain/a/20260910A07L4V00` — 中国金融新闻网/金融时报 interview with NUCC President 崔嵬 via 腾讯新闻 (2026-09-10): 2026 CNY's Eve peak, 2026 H1 cross-border growth, agentic payment.
- `https://www.zfzj.cn/12060.html` — 支付之家 (2026-06-16): NUCC's 2026 China International Finance Exhibition material, single-day peak and daily average, cross-border market share.
- `https://m.mpaypass.com.cn/news/202201/04142712.html` — 移动支付网 (2022-01-04): New Year 2022 volumes and peak concurrency.
- `https://m.mpaypass.com.cn/news/201710/15125900.html` — 中国新闻网 via 移动支付网 (2017-10-15): the 209号文 deadline date and connected-institution count at that point.
- `https://www.sohu.com/a/697118749_114819` — product-practitioner write-up (2023-07-12, via 搜狐 and woshipm): the post-reform clearing model, mapped/available quotas and the management back-office.

Secondary aggregators (used only for corporate-record and milestone details, always marked ⚠):

- Baidu Baike entry 22111484 (网联清算有限公司, revision dated 2026-09-01), entry 22167842 (the 209号文 notice; revision 2022-05-04) and entry 23240934 (断直连; revision 2025-02-21).
- zh.wikipedia.org, 非银行支付机构网络支付清算平台.
- 爱企查 / 企查查 company-record snippets (equity structure, company description).
### 15.3 Further reading that would close the gaps

- The PBOC's annual *Payment System Operation Overall Situation* (支付体系运行总体情况) releases for 2020–2025 — the authoritative per-channel series.
- NUCC's own published standards and participant notices, through a participant relationship or the archived official site once the Internet Archive is available again.
- The 2025 and 2026 China International Finance Exhibition releases in full, for the operator's own complete claim set.
---

## 16. Closing Summary

NUCC is the most consequential piece of payment infrastructure most banks' China desks have never had to connect to directly, and it exists because the state decided that the clearing leg of third-party online payments had to be visible, uniform and licensed. The story runs in four movements. First, a mesh of bilateral bank links was replaced, by 银支付〔2017〕209号 of 2017-08-04, with a single PBOC-mandated platform — licensed, privately incorporated, 37% held by PBOC-affiliated institutions and 63% by the payment institutions that use it, and prohibited from competing with them. Second, the money was moved: 银办发〔2018〕114号 of June 2018 stepped customer reserve funds out of commercial banks and into the PBOC, reaching 100% on 2019-01-14, after which the platform's job became the instruction, the netting and the reconciliation of flows whose funds already sit inside the central bank. Third, the scale became industrial: 8.1 billion transactions in a single 2018 quarter, 397.5 billion in 2019, and by the operator's own 2025 disclosure more than a trillion transactions and RMB 520 trillion in 2024, with a New Year's Eve peak that rose from 7,000 transactions per second in 2018 to 125,000 in 2026. Fourth, the perimeter began to widen again — a co-operation with CIPS, inbound card and wallet products, an agentic-payment pilot, and a card-clearing joint venture with Mastercard that puts a NUCC-affiliated entity into UnionPay's own line of business.

For a bank, the practical lesson is narrower than the story and firmer than the marketing. There is one mandatory channel decision to make (NUCC, UnionPay, or a genuine dual connection), one liquidity and quota problem to solve, one reconciliation service to run as production, and one honest admission to make in the business case: the settlement sessions, the tariff, the interface specification and even the platform's availability claims are not public, so the numbers in any plan are the bank's own assumptions until the participant documentation arrives. Everything this guide could not verify is written down rather than smoothed over — and what could be verified is attributed, dated and marked. The durable point is structural: in the Chinese retail payment system, licensed institutions clear, banks settle, the central bank holds the money and sees the traffic, and no payment institution may route around the licensed clearing.
