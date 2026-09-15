# IBPS and Payment Connect — the cross-boundary rail

**Jack Liu Shurui, Solution Architect**

*Deep-research guide. Capture date: 15 September 2026 (UTC).*

---

## Scope, boundary and how to read this guide

This guide covers two things, and it is deliberate about the join between them:

1. **IBPS** — the Mainland Chinese Internet Banking Payment System (网上支付跨行清算系统), the domestic real-time retail payment system operated by the China National Clearing Center (CNCC, 中国人民银行清算总中心) under the guidance of the People's Bank of China (PBoC, 中国人民银行).
2. **Payment Connect** — the cross-boundary linkage of IBPS with Hong Kong's Faster Payment System (FPS, 转数快), launched on 22 June 2025.

Every rule, limit, institution name and date is attributed. Where a fact is first-party HKMA/HKICL/CNCC material, that is said. Where it is secondary, that is said. Where it could not be verified, it appears in section 13 and is **not** asserted elsewhere.

**Boundary — what this guide does not own.** `banking/cnaps_guide.md` owns CNAPS (中国现代化支付系统) as a whole and treats IBPS only at component level (its launch and its 24x7 positioning, and what it enabled relative to the batch system). **This file owns IBPS depth and the cross-boundary linkage.** Do not expect a re-derivation here of HVPS, BEPS, the city processing centre estate or the CIPS distinction (CIPS — the Cross-border Interbank Payment System, 人民币跨境支付系统 — is a cross-border RMB system, structurally separate from the domestic retail rails discussed here). `banking/nucc_netsunion_guide.md` owns the NetsUnion Clearing Corporation (网联); this guide mentions it only to position IBPS against it. Both companions are being written **concurrently** with this file — they are cross-referenced **by path only**, this guide did not read them, and it did not edit them or any other untracked file.

**A warning about the numbers.** Payment Connect's rules — limits, hours, quotas, institutions — are **living consumer data** published by the HKMA and HKICL with a page revision date. This guide reproduces the snapshot **as of 08 September 2026** (HKMA page revision date) and the institution lists **as of 2 September 2026** (stated as-of date). These will change. Before relying on any figure — in a design, a test case, a customer communication or a contract — re-check <https://www.hkma.gov.hk/eng/smart-consumers/payment-connect/>.

---

## Table of contents

1. [Overview — what IBPS is, and what Payment Connect is](#1-overview--what-ibps-is-and-what-payment-connect-is)
2. [IBPS itself — launch, mandate and design goal](#2-ibps-itself--launch-mandate-and-design-goal)
3. [Architecture, participants and technical model](#3-architecture-participants-and-technical-model)
4. [The cross-boundary linkage: Payment Connect](#4-the-cross-boundary-linkage-payment-connect)
5. [The Payment Connect rules and limits in full](#5-the-payment-connect-rules-and-limits-in-full)
6. [The participating institutions](#6-the-participating-institutions)
7. [The user experience and the operational flow](#7-the-user-experience-and-the-operational-flow)
8. [The bank-side implementation requirements](#8-the-bank-side-implementation-requirements)
9. [Compliance, AML and risk](#9-compliance-aml-and-risk)
10. [Strategic significance](#10-strategic-significance)
11. [The Cymbal Bank worked example](#11-the-cymbal-bank-worked-example)
12. [The claims audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-references and further reading](#15-cross-references-and-further-reading)
16. [Closing summary](#16-closing-summary)

---

## 1. Overview — what IBPS is, and what Payment Connect is

IBPS is the Mainland's **real-time retail** payment system. The HKMA's own definition is the cleanest one available in English:

> "The Mainland's Internet Banking Payment System (IBPS) is the faster payment system in the Mainland, built by the China National Clearing Center under the guidance of the People's Bank of China. IBPS enables user to make real-time cross-bank retail payment transactions through the mobile banking or online banking of his/her bank. By using the system, user can handle cross-bank account management, fund transfer, fund consolidation and other services without leaving home, and obtain instant transaction processing results."
> — HKMA, *Payment Connect* consumer page, last revision 08 September 2026.

Three facts are embedded in that paragraph and all three matter architecturally: **real-time** (the system responds synchronously and the user learns the final result in the same interaction — this is not a batch queue); **retail** (built for the small-value, high-count end of the market, not large-value wholesale); and **cross-bank** (an in-bank transfer needs no interbank rail at all, so IBPS exists precisely for the case where payer and payee bank at different institutions).

CNCC describes it from the participant's side: users "can handle cross-bank account management, fund transfer, fund consolidation and other services without leaving home, and can obtain the business processing results online in real time" (足不出户办理跨行账户管理、资金汇划、资金归集等多项业务，并可实时在线获取业务的处理结果). **足不出户 / without leaving home** is not marketing colour; it describes the product promise the system was built to deliver as e-commerce grew in the 2000s.

### Where IBPS sits in the Chinese retail payment architecture

Drawn here only to position IBPS — the CNAPS companion file owns the detail:

| System | Role | Typical profile | Settlement |
| --- | --- | --- | --- |
| **HVPS** (大额实时支付系统) | RTGS equivalent | Large-value, time-critical | Real-time gross in central bank money |
| **BEPS** (小额批量支付系统) | Batch / ACH equivalent | Payroll, recurring, mass low-value | Deferred net, settled through HVPS |
| **IBPS** (网上支付跨行清算系统) | Real-time retail rail | Internet/mobile banking; e-commerce via bank channels | Real-time clearing; net positions settled in batches through HVPS |
| **NetsUnion** (网联) | Third-party payment platform clearing | Alipay / WeChat Pay flows | Real-time clearing, deferred net through HVPS |
| **CFXPS** | Domestic FX clearing | Domestic interbank FX | Hybrid |
| **UnionPay bankcard interbank system** | Card clearing | Debit/credit card flows | Net through HVPS |

The structural point: **IBPS is not the final settlement layer.** Individual retail payments are processed and confirmed in real time, but final interbank settlement is by netted positions settling through HVPS in central bank money. A "real-time" confirmation to a customer does not mean a real-time movement of central-bank funds between the two banks.

### The naming caution — the acronym "IBPS" collides

- **"IBPS" ≠ internet banking generally.** Internet banking (网上银行) is a *channel* a bank offers; IBPS is a *clearing system* such channels may use.
- **"IBPS" ≠ Hong Kong's FPS.** FPS (转数快) is the HKMA's faster payment system, launched September 2018, connecting banks and stored-value facility (SVF) operators so the public can transfer funds in real time by mobile number or email address. IBPS and FPS are two different domestic systems in two different jurisdictions. **Payment Connect is the linkage between them** — both remain domestic systems; the linkage is a layer of arrangements on top.
- **"IBPS" ≠ NetsUnion (网联).** NetsUnion clears third-party payment platform flows; IBPS clears bank-channel retail flows. The overwhelming majority of Alipay and WeChat Pay volume clears through NetsUnion, not IBPS. Confusing the two produces a badly wrong assumption about whose rules apply.
- **"IBPS" ≠ CNAPS, and ≠ CNAPS2.** CNAPS is the umbrella programme; CNAPS II its second generation; IBPS is one business subsystem *within* CNAPS2.
- **"IBPS" ≠ the batch system.** BEPS is the batch rail. Assuming batch cut-offs and return windows means you are describing BEPS.

### One-page orientation to Payment Connect

**Payment Connect refers to the linkage of the Mainland's IBPS and Hong Kong's FPS.** The HKMA and HKICL use that identical framing. With the joint support of the PBoC and the HKMA, the linkage was implemented "to support the provision of real-time cross-boundary payment services for residents and institutions in both places". The participating institutions from the two places launched the service to the public **on 22 June 2025**.

What that buys, in HKICL's first-party language: the receiving institution need not be in the same group; recipients need not be same-name or immediate family; a wide range of current-account payment including P2P and daily-living payments; anytime, anywhere, settled instantly; mobile number and other account proxies usable; low transaction costs; RMB or HKD remittance with defined receipt-currency behaviour per direction. Structurally, a Hong Kong resident can send a small-value cross-boundary remittance from their existing mobile banking app to a personal account at a Mainland bank **without documentary proof**, 24x7 northbound, addressed by mobile number or account number; and a Mainland resident can send to Hong Kong using mobile number, email address, FPS ID or account number, inside a shorter daily window. By simply inputting the recipient's mobile number or account number, residents in both places can make instant small-value cross-boundary remittances.

---

## 2. IBPS itself — launch, mandate and design goal

### The mandate

IBPS is a **renminbi interbank payment and clearing infrastructure** built by the PBoC and operated by CNCC. Its mandate, in CNCC's own framing, is to support the interbank (and intra-bank) fund transfer processing of emerging electronic payment businesses such as online payment (主要支持网上支付等新兴电子支付业务的跨行（同行）资金汇划处理) and to satisfy users' round-the-clock payment needs (能满足用户全天候的支付需求). Note the parenthetical: **跨行（同行）** — cross-bank *(and same-bank)*. IBPS is specified to handle both, which is why it is sometimes used as the intra-bank retail rail for very high-volume institutions — the source of the hotspot-account problem below.

### The design goal, in the operator's words

CNCC states the goal in terms of what a *user* can do: **足不出户** (without leaving home); handle **cross-bank account management** (跨行账户管理); handle **fund transfer** (资金汇划); handle **fund consolidation** (资金归集); and obtain the business processing result **in real time, online** (可实时在线获取业务的处理结果). Separately, CNCC describes the transaction mechanics as a **real-time transmission and response mechanism** (实时传输及回应机制) covering cross-bank payment (跨行支付), cross-bank account information inquiry (跨行账户信息查询) and online signing (在线签约).

That last trio distinguishes IBPS from a pure "send money" rail. The system was designed to let a customer, from one bank's online banking, **query and manage accounts held at other banks** — the "super online banking" (超级网银) identity IBPS acquired in popular usage, and the reason the fund-consolidation use case exists: pull balances together from multiple banks into one.

### Launch and rollout — a dated fact table

The widely repeated launch date was checked rather than copied. The result is favourable: **the operator itself publishes the date**, so it can be asserted on a first-party basis rather than flagged as rumour.

| Fact | Value | Source | Status |
| --- | --- | --- | --- |
| First production go-live of IBPS | **30 August 2010** (2010年8月30日), 先期投产运行 — put into production ahead of the rest | CNCC, 网上支付跨行清算系统 IBPS, 15 Jun 2016, <https://www.cncc.cn/zfxt/201606/t20160615_416.html> | **Verified at the operator's own site** |
| Nationwide rollout completed | **24 January 2011** (2011年1月24日), 推广至全国 | Same CNCC page | **Verified at the operator's own site** |
| Position within CNAPS2 | The **first business system of the second-generation payment system to be put into production** (第二代支付系统率先投产的业务系统) | CNCC 银清科技 page, <https://tech.cncc.cn/ywjs/202212/t20221212_1182.html> | **Verified at the operator's own site** |
| Operating model | 7×24 continuous operation; real-time transmission and response | CNCC pages; HKMA consumer page (24x7 northbound) | Verified |
| Per-transaction system-level limit | Reported as **under RMB 50,000 per transaction** (单笔5万元以下) for online-banking flows | Chinese-language encyclopaedia entry (Baidu Baike), secondary | **Reported, not confirmed at primary source** — indicative only |
| Business-processing model change | **2016**: optimised from "real-time forwarding, real-time netting" (实时转发、实时轧差) to **"real-time forwarding, scheduled netting"** (实时转发、定时轧差), resolving the hotspot-account problem for very-high-volume participants | CNCC 银清科技 page | **Verified at the operator's own site** |
| Volumes (2023) | ~17bn transactions, ~RMB 301 trillion | ClearingPost industry guide, secondary | **Attributed secondary** |

### Why the 2016 change is the most instructive technical fact in this section

"Real-time forwarding, real-time netting" meant every individual payment triggered a netting update against the counterparty's position as it was forwarded. For a participant with very high retail volume that created **hotspot accounts** — concentration of netting traffic on particular accounts, throttling throughput. The 2016 shift to **"real-time forwarding, scheduled netting"** decoupled the two: the customer-facing leg stays real-time while netting runs on a schedule. Two consequences: (1) **customer-visible real-time and settlement-time real-time are different clocks**, and the 2016 change is documentary proof that the operator deliberately runs them on different clocks; (2) **a "real-time" rail can still carry netting-driven exposure windows** — as a direct participant, your intraday net position against the system is a real risk object even though your customers see instant confirmations.

### How IBPS differs from the large-value and bulk systems

| Dimension | HVPS (大额) | BEPS (小额批量) | **IBPS** |
| --- | --- | --- | --- |
| Processing | Real-time gross | Batch cycles | **Real-time, per-transaction response** |
| Availability | Business days, business hours | Business days, batch windows | **24x7x365** |
| Value profile | Large value | Low value, high volume, non-urgent | **Small-value retail, instant need** |
| Settlement | Gross, central bank money | Deferred net through HVPS | **Real-time clearing; net positions through HVPS** |
| Typical use | Interbank, securities, corporates | Payroll, benefits, recurring | **Online/mobile banking, e-commerce via bank channels** |

The one-line summary: **IBPS is the rail you choose when the customer is waiting.** And what IBPS is *not*: not a card rail (that is the UnionPay bankcard interbank system); not a third-party wallet rail (that is NetsUnion — see `banking/nucc_netsunion_guide.md`); and **not a cross-border rail in its own right** — IBPS is a domestic system, whose cross-border reach exists only through the Payment Connect arrangement (section 4), for the corridors and directions that arrangement enables.

---

## 3. Architecture, participants and technical model

Deliberately conservative. IBPS's full interface documentation is distributed **to participants by CNCC**, not published as a public standard. Where a technical claim cannot be cited, this guide says so rather than filling the gap with plausible-sounding detail — and what a bank's architecture team actually needs is *which questions to put to CNCC*.

### The tiered topology

The CNAPS architecture IBPS sits inside is documented in the BIS CPSS *Red Book* country volume for China and described in practitioner literature as hierarchical: a **National Processing Center (NPC)** operated by the PBoC head office at national level, and **City Clearing Processing Centers (CCPCs, 城市处理中心)** as regional nodes — commonly cited at **32 cities** — connecting local banking participants to the national payment network. For IBPS specifically, CNCC publishes *business function* rather than topology: 7×24 continuous operation, real-time transmission/response, cross-bank payment, cross-bank account information inquiry and online signing.

> **Status note.** The NPC/CCPC hierarchy and the "32 cities" figure are **widely reported** from practitioner synthesis of the BIS *Red Book* and the PBoC's payment-system overview, not from a first-party architectural specification retrieved for this guide. Confirm against the PBoC/BIS source before use. **Node counts, redundancy topology and throughput ceilings are not established here at all.**

### Participant categories

The standard CNAPS-family vocabulary — and the vocabulary you will hear from CNCC and your counterparty bank:

| Category | What it means | Consequence for you |
| --- | --- | --- |
| **Direct participant** (直接参与者) | Holds a settlement account and connects directly; settles in its own name | You carry the connection, certification, netting exposure and operational obligations |
| **Indirect participant** (间接参与者) | Reaches the system through a direct participant — typically a smaller bank, a foreign bank branch, or a non-bank institution | You inherit your sponsor's availability, cut-offs and error handling; incident response is partly somebody else's |
| **Clearing centre / operator** | CNCC | The interface you must conform to, and the source of your interface specifications |
| **Processing centre** | The national processing node and the CCPC estate | The availability domain your service inherits |

Practical point for a cross-boundary design: whether a Mainland participating institution reaches IBPS directly matters for **who you talk to when something fails** — an indirect participant adds a party to the incident path.

### The access model — what a bank must implement

The definitive list is the one CNCC issues to participants. What can be said with confidence from the public record: (1) **a settlement/clearing relationship** — direct participants hold the relevant accounts, indirect participants contract with a direct participant; (2) **an interface implementation** to the system's message and file exchange conventions, in whatever form CNCC specifies for that generation; (3) **certification** against CNCC's test environment before production traffic; and (4) **operational readiness** consistent with a 7×24 service — monitoring, incident process, and a service desk that does not close.

### Message and interface standards — what is documented, and what is not

- **Documented at the level of direction, not specification: CNAPS II is widely described as having moved to ISO 20022-based message formats.** The practitioner literature and vendor access-product documentation describe CNAPS2 processing for HVPS, BEPS Regular and BEPS Real-Time using contemporary message-format conventions, and ISO 20022 migration trackers list the Chinese domestic systems among migrated or migrating systems. **This guide therefore asserts only that ISO 20022 alignment is widely reported for the CNAPS2 generation — not that IBPS message identifiers take any particular form.**
- **Not publicly documented, and not asserted here:** specific message type identifiers, a proprietary standard name, an interface specification number, file/field layouts, or the business-to-system message mapping. A bank must obtain these **from CNCC under the participation agreement**. Do not design against a spec you found on the internet.
- **Practical remedy.** Write "interface specification to be obtained from CNCC under the participation agreement" in your architecture document rather than a guessed standard name. That sentence is defensible; a guessed identifier is not.

### Account proxies and the identifier model

The mechanism that makes a *retail* real-time system usable is the **account proxy**: an identifier a payer types that the system resolves to a receiving account.

| Direction | Proxies supported | Source |
| --- | --- | --- |
| **Northbound** (HK → Mainland) | (1) Mobile number; (2) Bank account number | HKMA consumer page, as of 08 Sep 2026 |
| **Southbound** (Mainland → HK) | (1) **Mobile number, email address and FPS ID**; (2) Bank account number | HKMA consumer page, as of 08 Sep 2026 |

On the Hong Kong side FPS natively addresses payments by mobile number or email address, so the southbound proxy set reflects what FPS already supports, plus **FPS ID** (a system-assigned identifier, useful when a customer does not want to expose a phone number or email address). The asymmetry is a direct function of **what each domestic system can resolve**, which depends on **what the receiving account holder has registered**. HKICL's guidance makes the dependency explicit, and it is the number-one cause of failed first attempts: the Mainland payee must have **enabled cross-boundary fund receipt via mobile number** (their bank account linked to a mobile number, information updated as required); if the Northbound payer uses the **bank account number** path instead, the payee should verify whether their personal information includes an **English (or Pinyin) name** in the bank's records, and consult the Mainland participating institution; and if the account is **already linked to an FPS proxy**, no further FPS registration is required. In other words: **the proxy is not universal** — it exists only where the receiving bank has the customer's record in the required shape. That single fact drives the name-matching and onboarding requirements in sections 7 and 8.

### Capacity and volume characteristics

Only attributed figures, and they are secondary: **IBPS 2023 ≈ 17 billion transactions, ≈ RMB 301 trillion**; for scale, **HVPS 2023 ≈ 382 million transactions, ≈ RMB 8,481 trillion** — both from ClearingPost's industry guide, which synthesises PBoC/BIS/World Bank material. The useful read is one of order of magnitude: IBPS is among the largest real-time retail systems in the world by transaction count, while HVPS carries enormous value on comparatively few transactions. **These are not PBoC-published statistics as reproduced in this guide**; if your use case depends on the precise figure, take it from the PBoC's own payment system reports.

### Questions an architecture team must put to CNCC / HKICL

The public record does not answer these — own them explicitly: (1) which **interface specification and version** applies to our participation, and what is the certification path and timeline; (2) what are the **participant-level throughput, message-size and rate limits**, and how are they enforced; (3) which **participant categories** may we hold, and what are the obligations of each; (4) for the corridor specifically, what are our obligations on **name matching, proxy registration state, and error/return codes**; (5) what are the **operational escalation paths and incident-notification obligations** for a 7×24 service; (6) what **reporting and record-retention obligations** apply on our side of the corridor.

---

## 4. The cross-boundary linkage: Payment Connect

### What the linkage is

Payment Connect is the **linkage of two domestic faster payment systems**: the Mainland's IBPS and Hong Kong's FPS. The HKMA and HKICL state this identically, and the framing matters: neither system is replaced or merged. Each continues to operate domestically under its own operator and rules; the linkage is an arrangement — legal, commercial and technical — letting each system reach across for a defined set of retail payment flows. The purpose, in the HKMA's words, is to support "the provision of real-time cross-boundary payment services for residents and institutions in both places", under the joint support of the PBoC and the HKMA.

### Joint governance

The corridor is governed jointly by the **PBoC** and the **HKMA**. Every authority statement about the service's scope and use cases is presented as coming from both: HKICL's page states plainly that "the following service scope and use cases are provided by the PBOC and the HKMA", and points to the HKMA page for detail. Treat *the two authorities together* as the source of the rules and the two consumer pages as one authority set. Operationally the corridor runs through the two clearing infrastructures: **HKICL** on the Hong Kong side (operator of FPS, publisher of the operational guidance) and **CNCC** on the Mainland side (operator of IBPS).

### The MOU — 2 August 2024

| Fact | Detail | Status |
| --- | --- | --- |
| Release | "PBOC and HKMA Sign MOU on Cross-Boundary Linkage of Payment Systems between the Mainland and Hong Kong", **02 Aug 2024**, <https://www.hkma.gov.hk/eng/news-and-media/press-releases/2024/08/20240802-5/> | Verified |
| Signatories | **Mr Lu Lei**, Deputy Governor of the PBoC, and **Mr Howard Lee**, Deputy Chief Executive of the HKMA | Verified |
| Occasion | The two met on 2 August 2024, exchanged views on financial cooperation between the Mainland and Hong Kong and on **pushing forward the linkage of fast payment systems**, and signed the MOU establishing a **cooperation framework for the linkage** | Verified |
| The MOU text | **Not public.** The documented content is that it establishes a cooperation framework for the linkage; the operative terms were not published | **Could not verify — see section 13** |

**Be precise about this.** The MOU is a *cooperation framework* — the political and administrative foundation on which the service was built, not a rulebook. Its terms (who settles what, how the FX leg is priced, what the liability allocation is) are not in the public domain. Any claim of the form "the MOU provides that…" beyond "it established a cooperation framework for the linkage of the fast payment systems" is **unsupported**.

### The launch — 22 June 2025

| Fact | Detail | Status |
| --- | --- | --- |
| Announcement | HKMA press release "The HKMA and the PBoC Launch Payment Connect", **20 June 2025**, <https://www.hkma.gov.hk/eng/news-and-media/press-releases/2025/06/20250620-4/> | Verified |
| Service launch to the public | **22 June 2025** | Verified — HKMA press release; HKICL: "The participating institutions from the two places launched the service to the public on 22 June, 2025" |
| Launch ceremony | Held in **Beijing** that day | Verified |
| Initial participation | "**Six institutions each from the Mainland and Hong Kong** will participate in the Payment Connect upon its launch. They will **roll out the service gradually** and **more institutions will join over time**." | Verified — HKMA press release |
| Attendees | Mr **Pan Gongsheng**, Governor of the PBoC; Mr **Wang Linggui**, Deputy Director of the Hong Kong and Macao Work Office of the CPC Central Committee and the Hong Kong and Macao Affairs Office of the State Council; Mr **Eddie Yue**, Chief Executive of the HKMA — all delivered remarks. Ceremony officiated by Mr **Lu Lei**, Deputy Governor of the PBoC; Mr **Zhang Hui**, President of the Bank of China, attended | Verified |
| Launch institution annex | An annex PDF listed the initial institutions: <https://www.hkma.gov.hk/media/eng/doc/key-information/press-release/2025/20250620e4a1.pdf> | Cited by the release; **contents not retrieved by this guide** (§13.9) |
| Companion document | A Joint Announcement of the PBoC and the HKMA was also issued on **20 Jun 2025** | Verified — listed on the HKMA Payment Connect page |

**The 6+6 launch and the 26+18 position are the same story at two points in time.** The release that announced six institutions each also announced that "more institutions will join over time". By the snapshot used throughout this guide that growth had taken the Hong Kong list to **26** and the Mainland list to **18** (section 6). Read them together — 6+6 is the launch cohort, 26+18 the current position — and do not cite one as the other.

### What the linkage means technically and operationally — as far as documented

- **Two domestic systems, connected.** Payments originate in one domestic system, are addressed using that system's proxy conventions where applicable, and are received into an account at a participating institution in the other market.
- **Real-time cross-boundary payment.** HKICL's benefits list says "anytime, anywhere, and **settled instantly**"; its receive-side flow says funds are "credited immediately" with a notification to the payer once the payment is completed.
- **Instant results at the payer**, whose flow ends with a payment notification, and whose confirmation step displays the RMB amount and HKD equivalent before the customer commits.
- **Instant confirmation, mitigated by a name check.** The northbound mobile-number flow displays a **partially masked payee name** for the payer to verify — a control that exists precisely because the rail is instant and effectively irreversible.
- **A defined currency model per direction.** Northbound: RMB or HKD remittance (pay directly from RMB accounts, or real-time exchange of HKD for RMB), with receipt in **RMB**. Southbound: RMB remittance, with receipt in **either HKD or RMB**.
- **Proxy resolution across the boundary.** A Hong Kong payer can input a **Mainland mobile number** to reach a Mainland resident's account that has enabled cross-boundary receipt by mobile number.

### What is *not* documented — flag it, then ask

Each is phrased as a **question for CNCC and HKICL**; none is speculated about here. (1) **The clearing and settlement arrangement between the two systems** — how does value move between the two clearing estates? Bilateral settlement position, settlement bank, correspondent chain, or an existing arrangement? (2) **The FX conversion leg** — who converts, at what rate, at what time, and with what rate-disclosure obligation to the payer? The consumer material tells the payer only that they must *confirm the applicable exchange rate* before committing. (3) **Message translation between the two systems** — mapping of identifiers, names, purpose codes and status codes is not public; ask for the cross-boundary message mapping specification. (4) **Liquidity and settlement-bank arrangements** — who funds the intraday position, in which currency, at what time, and what the exposure is if one side's window closes with a payment in flight. (5) **Return, recall and refund semantics** natively supported across the boundary — the consumer material describes successful credit only. (6) **Liability allocation** for a payment credited to the wrong person because a name did not match — a contractual question, not a technical one.

> **Constructed note.** The six questions above are **constructed by this guide** as the questions a bank must ask; they are not quoted from a published specification. The *subject matter* of each (settlement, FX, messaging, liquidity, returns, liability) is a real gap in the public record. The phrasing is this guide's.

### Why the linkage is a payment-system event, not just a product feature

Eddie Yue, on launch: *"Payment Connect is one of the key initiatives in strengthening cooperation between the Mainland and Hong Kong. The connection between the faster payment systems in both places enhances the efficiency of cross-boundary payments, supporting trade activities and personnel exchange. This development will further promote Hong Kong's position as an international financial centre and offshore Renminbi business hub. Payment Connect will cater to the daily needs of residents in both places for cross-boundary remittances and payments. It also signifies another milestone for the FPS in expanding cross-boundary payment."*

The operative phrases — **efficiency of cross-boundary payments**; **trade activities and personnel exchange**; **daily needs**; **another milestone for FPS in expanding cross-boundary payment** — position the service explicitly at the *daily-living* end of the market, not the wholesale end. That is consistent with every rule in section 5: limits are small, evidence requirements are nil, and the use cases are tuition, medical, utilities and salary.

---

## 5. The Payment Connect rules and limits in full

> **Read this first.** Every figure below is **first-party HKMA/HKICL material, current as of the HKMA page revision date of 08 September 2026** (institution lists as of 2 September 2026). These are consumer-facing service rules and **they will change**. Re-check <https://www.hkma.gov.hk/eng/smart-consumers/payment-connect/> before relying on any of them.

### The P2P remittance rules table

| | **Northbound (Hong Kong → The Mainland)** | **Southbound (The Mainland → Hong Kong)** |
| --- | --- | --- |
| **Eligible Payer** | Hong Kong Identity Card holder | The Mainland Identity Card holder |
| **Limit** | Daily limit of **HKD 10,000 per person**, and annual limit of **HKD 200,000 per person (for each participating institution)**. The limit is **separate from the existing daily remittance limit of RMB 80,000 that applies to the same-name transfers** | Follow the **existing facilitative foreign exchange arrangement available for Mainland residents**, subject to an annual quota equivalent to **USD 50,000 per person** |
| **Documentary Proof Required** | **Nil** — (the payee is **restricted to personal account holders only**) | **Nil** |
| **Service Hours** | **24x7** | **16x7** — "The current service hours are from **7:00 AM to 11:00 PM daily**, which may vary subject to individual Mainland participating institutions" |
| **Types of Account Proxy Supported** | (1) Mobile number; (2) Bank account number | (1) **Mobile number, email address and FPS ID**; (2) Bank account number |

*Source: HKMA Payment Connect consumer page; identical rules table on HKICL's FPS Payment Connect page. Snapshot as of the 08 September 2026 revision date.*

### The use-cases table

| **Use Cases** | • **Personal payment of fees**, such as tuition fees, medical expenses and utility bills<br>• **Business-to-personal remittances**, such as salary disbursements<br>• **Remittance on a pilot basis**, currently limited to pilot institutions, to be arranged based on actual needs — **please consult the receiving banks before making the payment** |
| --- | --- |

*Source: HKMA / HKICL. The use-case list is explicitly stated to be provided by the PBoC and the HKMA.*

### The progressive-rollout note — reproduce it in substance

Both first-party pages carry a note that is easy to skim past and is, operationally, one of the most important paragraphs in the arrangement. In substance: services are provided by participating institutions and **the functions and scope of coverage offered by each institution may vary**; additionally **some features may be introduced gradually over time** — for example, Hong Kong payers can input a Mainland mobile number to transfer funds to Mainland residents' accounts that have enabled cross-boundary fund receipt via mobile number; and as for **Mainland accounts held by Hong Kong residents, Mainland participating institutions are optimizing the relevant functions**, rolling the feature out gradually depending on development progress; consult participating institutions for details.

In plain terms: (1) **there is no single uniform feature set** — the rules table tells you what the *scheme* permits, while your institution's and your counterparty's implementation tells you what actually works today, in which direction, from which channel; (2) the northbound "send to a Mainland mobile number" capability is conditional on the *Mainland payee's* account having enabled cross-boundary receipt by mobile number; (3) **Hong Kong residents holding Mainland accounts** is a feature area still being optimised and rolled out on the Mainland side — do not assume symmetric capability.

### The asymmetry between the directions — and why it matters

The most consequential analytical point in this section: the two directions are governed by **different regimes**, not by two sides of one symmetric scheme.

| Dimension | Northbound (HK → Mainland) | Southbound (Mainland → HK) | Why the difference matters |
| --- | --- | --- | --- |
| **Governing regime** | HKMA/PBoC scheme rules with an explicit HKD cap per person | The **existing facilitative FX arrangement** for Mainland residents | Northbound is a *new* rule set with its own caps; southbound rides on a *pre-existing* national FX framework |
| **Headline cap** | HKD 10,000/day and HKD 200,000/year, **per person, for each participating institution** | Annual quota equivalent to **USD 50,000 per person** | Not comparable units; no single conversion makes them equivalent — they answer different questions |
| **Limit granularity** | **Per institution** | **Per person**, under a national FX arrangement | Northbound usage counting is **per institution** — the design trap below |
| **Interaction with other limits** | Explicitly **separate from** the existing daily RMB 80,000 same-name remittance limit | Governed by the FX arrangement itself | Same-name remittance usage does **not** consume northbound headroom, and vice versa |
| **Service hours** | **24x7** | **16x7** — 07:00–23:00 daily, institution-variable | One side always open, one side with a daily close |
| **Proxy set** | Mobile number; account number | Mobile number, email address, **FPS ID**; account number | Different addressing, therefore different UX and different failure modes |
| **Recipient restriction** | **Same-name / immediate-family restriction lifted** (HKICL: recipients "not confined to same-name or immediate family members") | Same relief applies in principle; receiving institution likewise "not confined to those within the same group" | Removes the classic agent-banking constraint |
| **Receipt currency** | Receipt in **RMB** | Receipt in **either HKD or RMB** | Only one side offers a choice of receipt currency |
| **Documentary proof** | Nil, but payee restricted to personal account holders | Nil | No evidence is produced, so the control must be identity, limits and monitoring |

### Product-design consequences of the asymmetry

**Constructed analysis**, labelled as such, derived from the documented rules above:

1. **You cannot build one symmetric product.** The directions differ in limit basis, currency behaviour, proxy set, hours and recipient conditions. A single product page with one set of terms will be wrong for at least one direction. Design two flows, or one flow with direction-specific rule engines.
2. **The southbound window forces cut-off and batching design.** 07:00–23:00 daily, institution-variable, means there is a **daily close** southbound. Any "instant" promise must be honest about an instruction received at 22:58 or 23:01 — rejected, queued for the next window, or accepted and settled later? Decide and disclose. Northbound has no such problem, which is precisely why a shared codebase must not impose northbound assumptions on southbound.
3. **The per-institution nature of the northbound annual limit changes how you count.** Your obligation is to enforce *your institution's* HKD 200,000 for *each customer*. A customer using three institutions can, under the stated rule, have three separate HKD 200,000 allowances — **you cannot see, and are not responsible for, their usage elsewhere**, and must not attempt a cross-institution global cap the rule does not state. Your **usage counter must be per customer per institution** and must be a real, reconcilable ledger (section 8).
4. **Limit unit ≠ FX unit.** The northbound cap is in HKD while the receipt is RMB; the southbound quota is USD-equivalent while the remittance may be RMB. **A limit check requires a rate** — and therefore a rate source, a rate timestamp and a rate-validity window. That is an FX-risk and disclosure decision, not merely technical.
5. **The lifted same-name/family restriction changes the risk model.** When recipients had to be same-name or immediate family, the recipient's identity was partly self-proving. That control is gone. HKICL lists the lifting as a **benefit** — it genuinely widens usability — and the compensating controls are the **partially masked payee name display** and the payer's own confirmation duty. A bank's fraud and disputes strategy must absorb the shift.
6. **"Separate from RMB 80,000 daily" is a customer-communication requirement.** A customer who believes their northbound transfer consumes, or is blocked by, their existing same-name remittance allowance is a call to your contact centre. Say it explicitly in the product terms and in the app.
7. **"Nil documentary proof" is not "no controls".** It moves the control surface from documents to *identity verification, limit enforcement, proxy-resolution accuracy, name matching, velocity monitoring and post-event analytics*. Budget accordingly.

---

## 6. The participating institutions

> **Living document warning.** The lists are published by the HKMA and HKICL as consumer information with an explicit as-of date. **The copy below is a dated snapshot: 26 Hong Kong institutions and 18 Mainland institutions, as of 2 September 2026.** Institutions will join, some may leave, channels will be added, and some institutions will extend from one direction to both. Verify before use.

### Correction to a widely circulated figure

The figure of **"16 Mainland participating institutions"** circulates in secondary commentary. **It is wrong.** The HKMA consumer page numbers the Mainland list **1 through 18**, and the HKICL page lists the same **18** institutions by name. The correct figures at the stated as-of date are **Hong Kong: 26** (numbered 1–26) and **Mainland: 18** (numbered 1–18). The likely origin of the error is the launch cohort — the launch release announced **six institutions each** — so any "16" derived from launch-cohort reporting is stale. **Use 26 HK / 18 Mainland as of 2 September 2026**, and re-check before quoting.

### Hong Kong participating institutions (26) — as of 2 September 2026

Channels are as stated by the HKMA; the HKMA notes they are **self-reported by the institution**.

| # | Institution | Channels supported |
| --- | --- | --- |
| 1 | Ant Bank (Hong Kong) Limited | Mobile banking app |
| 2 | Bank of China (Hong Kong) Limited | Mobile banking app; Internet banking |
| 3 | Bank of Communications (Hong Kong) Limited | Mobile banking app; Internet banking |
| 4 | The Bank of East Asia, Limited | Mobile banking app; Internet banking |
| 5 | China CITIC Bank International Limited | Mobile banking app |
| 6 | China Construction Bank (Asia) Corporation Limited | Mobile banking app; Internet banking |
| 7 | China Guangfa Bank Co., Ltd Hong Kong Branch | Mobile banking app |
| 8 | China Merchants Bank Co., Ltd. | Mobile banking app |
| 9 | China Minsheng Banking Corp., Ltd. Hong Kong Branch | Mobile banking app; Internet banking |
| 10 | Chiyu Banking Corporation Limited | Mobile banking app; Internet banking |
| 11 | Chong Hing Bank Limited | Mobile banking app; Internet banking |
| 12 | CMB Wing Lung Bank Limited | Mobile banking app |
| 13 | Dah Sing Bank, Limited | Mobile banking app; Internet banking |
| 14 | Ele Bank Limited | Mobile banking app |
| 15 | Hang Seng Bank, Limited | Mobile banking app |
| 16 | The Hongkong and Shanghai Banking Corporation Limited | Mobile banking app; Internet banking |
| 17 | Industrial and Commercial Bank of China (Asia) Limited | Mobile banking app |
| 18 | Industrial Bank Co., Ltd., Hong Kong Branch | Mobile banking app |
| 19 | Mox Bank Limited | Mobile banking app |
| 20 | Nanyang Commercial Bank, Limited | Mobile banking app; Internet banking |
| 21 | Public Bank (Hong Kong) Limited | Mobile banking app; Internet banking |
| 22 | Shanghai Commercial Bank Limited | Mobile banking app |
| 23 | **Shanghai Pudong Development Bank Co., Ltd.** | Mobile banking app |
| 24 | Standard Chartered Bank (Hong Kong) Limited | Mobile banking app |
| 25 | WeChat Pay Hong Kong Limited | Mobile app |
| 26 | ZA Bank Limited | Mobile banking app |

*Source: HKMA, Payment Connect consumer page, "Participating Institutions — As of 2 September 2026", arranged alphabetically by English name per the HKMA's own note. The HKMA also publishes a website link and hotline per institution; this guide reproduces a representative sample in section 15 rather than a full link farm.*

### Mainland participating institutions (18) — as of 2 September 2026

| # | Institution | Note |
| --- | --- | --- |
| 1 | Agricultural Bank of China Limited | |
| 2 | Bank of China Limited | |
| 3 | Bank of Communications Co., Ltd. | |
| 4 | Bank of NingBo | |
| 5 | China Bohai Bank Co., Ltd | |
| 6 | China CITIC Bank Corporation Limited | |
| 7 | China Construction Bank Corporation | |
| 8 | China Everbright Bank Co., Ltd. | |
| 9 | China Guangfa Bank | |
| 10 | China Merchants Bank Co., Ltd | |
| 11 | China Minsheng Banking Corp., Ltd | |
| 12 | Fubon Bank (China) Co., Ltd. | |
| 13 | Industrial and Commercial Bank of China Limited | |
| 14 | Industrial Bank Co., Ltd. | |
| 15 | Ping An Bank Co., Ltd | |
| 16 | **Shanghai Pudong Development Bank Co., Ltd.** | **Footnote 3: "Only supports Southbound (The Mainland to Hong Kong) P2P Remittance."** |
| 17 | The Bank of East Asia (China) Limited | |
| 18 | ZHEJIANG CHOUZHOU COMMERCIAL BANK | |

*Source: HKMA consumer page, "Participating Institutions — As of 2 September 2026". Numbering is this guide's indexing of the HKMA's numbered rows; the HKICL version is unnumbered but contains the same 18 names — the two lists agree.*

### The one institution on both lists — and its asymmetry

**Shanghai Pudong Development Bank Co., Ltd. appears on BOTH lists**: Hong Kong #23 (mobile banking app) and Mainland #16, where it carries **footnote 3: "Only supports Southbound (The Mainland to Hong Kong) P2P Remittance."** That is a real, dated, first-party fact, and it matters: the Mainland-side entity is restricted to southbound, and does **not** support the northbound direction. For an architect it is the concrete demonstration that **participation is directional, not binary** — a "participating institution" may participate in one direction, one channel or one product type. Any registry, product catalogue or routing table must key on **(institution, direction, channel, product)**, not on institution alone. Build that fourth dimension in from day one; retrofitting it is expensive.

### The pattern the lists reveal

**Hong Kong side — retail and wallet-oriented, full spectrum of institution type.** The 26 span **the large incumbents** (Bank of China (Hong Kong), HSBC, Standard Chartered, Hang Seng, Bank of East Asia — including the note-issuing banks); **mid-tier and Chinese-heritage banks** with strong cross-boundary customer bases (Chiyu, Chong Hing, Nanyang Commercial, Shanghai Commercial Bank, Dah Sing, CMB Wing Lung, Public Bank (Hong Kong)); **Mainland banks' Hong Kong arms and branches** (Bank of Communications (Hong Kong), China CITIC Bank International, China Construction Bank (Asia), China Merchants Bank, China Minsheng Hong Kong Branch, China Guangfa Bank Hong Kong Branch, ICBC (Asia), Industrial Bank Hong Kong Branch, and Shanghai Pudong Development Bank); **all four virtual banks** (Mox, ZA Bank, Ele Bank, Ant Bank (Hong Kong)); and **a stored-value facility / e-wallet** (WeChat Pay Hong Kong Limited, channel "Mobile app"). The presence of four virtual banks and an SVF is the strongest evidence in the list that the linkage is deliberately **retail- and wallet-oriented** — virtual banks have no branch estate, so the corridor is their natural cross-boundary capability, and listing an SVF alongside banks confirms the Hong Kong side is addressed on **FPS terms**, since FPS connects banks *and* SVF operators.

**Mainland side — the banking mainstream, no big-tech wallet, no rural tier.** The 18 comprise **state-owned large banks** (Agricultural Bank of China, Bank of China, Bank of Communications, China Construction Bank, ICBC — five of the "big six"; Postal Savings Bank of China is not on the list as of the snapshot); **national joint-stock commercial banks** (China CITIC, China Everbright, China Guangfa, China Merchants, China Minsheng, Industrial Bank, Ping An Bank, Shanghai Pudong Development Bank); **city commercial banks and smaller institutions** (Bank of NingBo, China Bohai Bank, Zhejiang Chouzhou Commercial Bank — Chouzhou notably small by national standards, evidence the scheme is not restricted to the largest players); **a Taiwan-affiliated bank** (Fubon Bank (China)); and **a Hong Kong group's Mainland subsidiary** (The Bank of East Asia (China) Limited — the mirror image of a Mainland group's Hong Kong arm).

Two absences are as informative as the presences. First, **no Mainland big-tech wallet**: Alipay and WeChat Pay (Mainland) do not appear, while WeChat Pay **Hong Kong** appears on the *Hong Kong* list as an SVF — a different legal entity in a different market. This guide does not read the absence as a claim about intent or policy; it states only what the list shows, namely that Mainland participation is currently **bank-channel**, consistent with IBPS being the bank-channel rail and with third-party platform flows clearing through NetsUnion (see `banking/nucc_netsunion_guide.md`). Second, **no rural commercial banks and no county-level banks** — the Mainland tier that exists in the tens of thousands is absent. Practical consequence: **coverage of a Mainland payee is a coverage question, not a nationality question**, and a Hong Kong customer sending to a relative who banks with a rural commercial bank may find no reachable receiving institution at all.

### What a bank should do with these lists

> **Constructed — this guide's analysis, not published instruction.** (1) **Store the lists as data, not prose** — institution name, jurisdiction, direction(s) supported, channels supported, as-of date, source URL; re-verify on a schedule, because the lists change without a version number. (2) **Key routing on (institution, direction, channel, product)** — the SPDB footnote is the proof case. (3) **Never hard-code a count** — "26 and 18" is a snapshot, and a count embedded in a test assertion or a UI string becomes wrong silently. (4) **Model the "no reachable receiving institution" case explicitly** — it is a real customer outcome today and needs a defined message, not a generic failure. (5) **Do not conflate the HK-side entity with the Mainland-side entity of the same group** — several groups appear on both lists as separate legal entities with separate capabilities.

---

## 7. The user experience and the operational flow

HKICL publishes the consumer flows in step form. This section reproduces them **in substance** — faithfully to each step, not as a copy-paste — and adds the operational reading a bank's ops team needs.

### Northbound (Hong Kong → Mainland): the mobile-number path — 5 steps

| Step | In substance |
| --- | --- |
| **Step 1** | **Select payment account (HKD / RMB account)** |
| **Step 2** | **Enter the payee's mobile number and bank name**, and **verify whether the partially masked payee's name displayed on the screen is correct** |
| **Step 3** | **Enter the payment amount, purpose of transfer and remarks (if applicable)**, and **confirm the applicable exchange rate when selecting the HKD account as payment account** |
| **Step 4** | **Verify the transaction details**, including the payment account, payee's name, mobile number, payee's bank name, **RMB amount and equivalent to HKD (if applicable)**. Confirm the transaction only after ensuring accuracy of all information |
| **Step 5** | **The payer receives a payment notification** |

### Northbound (Hong Kong → Mainland): the bank-account-number path — 5 steps

| Step | In substance |
| --- | --- |
| **Step 1** | **Select payment account (HKD / RMB account)** |
| **Step 2** | **Enter the payee's bank name, bank account number and name in English** |
| **Step 3** | **Enter the payment amount, purpose of transfer and remarks (if applicable)**, and **confirm the applicable exchange rate when selecting the HKD account for the transfer** |
| **Step 4** | **Verify the transaction details**, including the payment account, payee's name, mobile number, payee's bank, **RMB amount and equivalent to HKD (if applicable)**. Confirm the transaction only after ensuring accuracy of all information |
| **Step 5** | **The payer receives a payment notification** |

*Source: HKICL, FPS Payment Connect consumer page — "How to use / How to Transfer Funds to a Payee in the Mainland".* HKICL attaches four notes: **receiving bank options include Mainland participating institutions that support Payment Connect** (i.e. the bank-name picker is a filtered list whose contents change as the participant list changes); **purpose-of-transfer options include personal payments, tuition fees, medical expenses and utility bills** (a **coded enumeration**, not free text, mirroring the use-case table in section 5); **process may vary depending on your service provider**; and for more detail refer to the **Smart Tips**.

### Southbound (Mainland → Hong Kong): receiving funds from a Mainland payer — 6 points

For a Hong Kong resident *receiving* money from a Mainland payer — receiving-side UX that product design often neglects: (1) **if the account for receiving funds is already linked to an FPS proxy, no further registration for FPS is required**; (2) **if bank account number is selected as the payment method, the payer should select the bank name**; (3) **if mobile number is selected, the payer should select the bank name and enter the payee's English name**; (4) **it is recommended that the payee communicates with the Mainland payer in advance to indicate the chosen currency for receipt, either HKD or RMB, and provides a bank account number or a linked FPS proxy that can receive the chosen currency**; (5) **certain Mainland banks may require the payer to input the payee's full name and address to meet with their internal risk management requirements**; (6) **the funds will be credited immediately and a notification will be sent to the payer once the payment is completed**.

*Source: HKICL, "How to Receive Funds from a Payer in the Mainland".*

### The Smart Tips list

In substance: **ensure the mobile banking app is updated to the latest version**; **ensure the Mainland payee has enabled cross-boundary fund receipt via mobile number** (their bank account linked to a mobile number, information updated as required); **if bank account number is selected as the Northbound payment method**, the Mainland payee can verify **whether their personal information includes an English (or Pinyin) name in record** through mobile banking or customer service, consulting the Mainland participating institutions; **communicate and confirm the payment details with the Mainland payee in advance**, and verify the transaction information before paying; and **ensure the receiving account can receive funds in the chosen currency** before providing the account information to the Mainland payer.

**Read the Smart Tips as a release-gate checklist.** Every item is a precondition that, if unmet, produces a failed or misdirected payment on an **instant, effectively irrevocable** rail. They are not "nice to have" hints.

### Timing and confirmation

Northbound availability is **24x7**; southbound is **16x7 (07:00–23:00 daily, may vary by Mainland institution)**; settlement is "**settled instantly**" per HKICL's benefits list, with funds "credited immediately" on the receive side; the payer receives a payment notification on completion; the **applicable exchange rate** is confirmed at the point of entry/confirmation when a HKD account is the payment account; and the confirmation screen displays the **RMB amount and the HKD equivalent** (if applicable). *Sources: HKMA/HKICL rules tables; HKICL "How to use".*

### Failure and return behaviour — the boundary of the evidence

**Documented:** the flows end in a payment notification on completion; the receive-side notes state funds are credited immediately; the name-verification step exists to prevent misdirection; and the Smart Tips exist to prevent failed attempts arising from unmet preconditions. **Not documented in the consumer material retrieved for this guide:** a published **cross-boundary return/recall protocol** (timeouts, reversal windows, who may initiate, what happens if the receiving account is closed or the credit rejected); published **error/decline reason codes** for the corridor and their mapping to customer messages; what happens to a **payment accepted just inside the southbound window** when the receiving side's window closes; and published **timing expectations for credit** when the rail or a participant is unavailable. A bank must therefore obtain the operational rulebook from CNCC/HKICL and the bilateral terms from each counterparty, and **design its own exception taxonomy** on top. A useful working assumption — flagged as *assumption*, not fact — is that a real-time retail credit is treated as **final on credit**, with recourse through dispute/claim processes rather than automated reversal. **Confirm this before relying on it** (see section 9.5).

### Practical friction points a bank ops team should anticipate

> **Constructed — analysis and operational inference from the documented flows and Smart Tips; not a published list.**

1. **Name matching, and the English/Pinyin name record.** The account-number path asks for the payee's **name in English**, and HKICL's Smart Tip says the Mainland payee should check whether their record includes an **English (or Pinyin) name**. In practice this is name comparison across scripts: anticipate **transliteration variants, surname-first vs surname-last ordering, spacing and hyphenation, and missing English names on the record**. A near-match is a decision point, not a bug — decide your policy (block, warn, allow-with-confirmation) and log it.
2. **The partially masked name display.** Your UI must render the masked string faithfully (not locally reformatted), and your wording must explain what the customer is being asked to verify. A masked name is a weak-but-real control, and it fails if the customer habitually taps through it.
3. **Mobile-number registration prerequisites on the receiving side.** The Mainland payee must have **linked a mobile number** and **enabled cross-boundary receipt via mobile number**; the HK sender cannot satisfy this on the recipient's behalf. Anticipate a **high rate of first-attempt friction** and provide forwardable guidance.
4. **Receiving-account currency capability.** Northbound receipt is in **RMB**; southbound receipt may be **HKD or RMB**. HKICL explicitly tells payees to confirm the currency **in advance** and provide an account/proxy that can receive it. Anticipate **credits missing their target**; model currency capability per account and surface it before sending.
5. **Certain Mainland banks may require the payee's full name and address.** This varies by institution and is not discoverable in advance from the published material. Provide a "what your payer may ask you for" guide and make it easy for the customer to retrieve their own details verbatim.
6. **FX rate disclosure timing and validity.** The customer confirms a rate at step 3/4 (HKD account path). Anticipate **rate-refresh races** — rate at input vs at confirmation vs applied. Choose one, stamp it, show it, store it, and be able to reproduce it in a dispute. This is also a fair-value/disclosure question.
7. **Purpose-of-transfer coding.** The purpose is an enumeration (personal payments, tuition fees, medical expenses, utility bills). Anticipate **mapping pressure** — customers will want purposes that are not on the list, and the "pilot basis / consult the receiving bank" use case maps to no code at all. Decide how the code interacts with limits, monitoring and reporting, and keep the code set versioned.
8. **Cut-off interaction with the 16-hour southbound window.** Anticipate **an instruction received outside the window** and **a payment in flight at the close**; both need defined, disclosed behaviour. Northbound has no such problem — which is precisely why a shared flow gets it wrong.
9. **Limit checks need a rate, and the customer needs a remaining-allowance view.** Northbound daily HKD 10,000 / annual HKD 200,000 is per person **per institution**, so "how much you have left with us" is not a global figure you can compute. Anticipate customer confusion about scope and design the wording accordingly.
10. **Per-institution participant variation.** The progressive-rollout note means institution A may not support what institution B does. The receiving-bank list must be filtered by capability, and support scripts must answer "why can't I send to bank X".

---

## 8. The bank-side implementation requirements

> **This section is CONSTRUCTED.** A checklist built by this guide from the documented rules in sections 4–7 plus ordinary payments-engineering practice. It is **not** a published implementation specification and is **not** taken from any CNCC, HKICL, HKMA or PBoC document. Use it as a programme-plan template and completeness check; obtain the authoritative requirements from the operators and your counterparties.

**A. Connectivity and integration.** Domestic rail connector **FPS** (Hong Kong) or **IBPS** (Mainland) — a prerequisite, not a programme deliverable. Cross-boundary capability layer: obtain the operator's specification under the participation agreement and **do not design against a guessed standard** (section 3). **Participant routing table** keyed on (institution, jurisdiction, direction, channel, product), informed by the SPDB southbound-only footnote, with effective-from dates. **Proxy resolution service** per direction: northbound (mobile number, account number); southbound (mobile number, email address, FPS ID, account number). **Institution capability registry** for "which banks can receive from us today", refreshed and versioned.

**B. Product and rules mapping.** Map each domestic product to the cross-boundary rule set **direction by direction** — do not assume domestic FPS/IBPS terms carry over. **Direction-specific rule engine** (limits, hours, proxies, currency, recipient restrictions). **Purpose-of-transfer code set**, versioned, mapped to the sanctioned enumeration and to your product codes. **Payer eligibility validation** at entry and re-validated at execution (northbound: **Hong Kong Identity Card holder**; southbound: **Mainland Identity Card holder**). **Payee eligibility validation** — northbound payee **restricted to personal account holders only**, determined by data not assumption. **Service-hours gating per direction** — northbound 24x7; southbound gated to the receiving institution's window (07:00–23:00 daily, institution-variable), stored as institution-level data.

**C. Limits and quotas.** Northbound **daily HKD 10,000** per person per institution, enforced against your own ledger. Northbound **annual HKD 200,000 per person, per participating institution** — **the per-institution scope is the trap**: build a **per-customer, per-institution usage ledger** with a defined accumulation basis and reconciliation against executed transactions. **Cross-channel aggregation** — mobile banking, internet banking, contact centre and assisted channels must post to **one** counter; per-channel counters are a control failure and a customer-experience failure. **Multi-institution customers** — the rule states no cross-institution cap, so **do not enforce one**; do proactively **disclose the per-institution scope** and record the acknowledgement. **Independence from the RMB 80,000 same-name daily limit** — **separate counters**, no netting. **Southbound quota** — annual USD 50,000 equivalent per person under the existing facilitative FX arrangement; if you are the Mainland institution you own this check, and if you are the Hong Kong institution you must know who does and obtain the confirmation evidence. **FX-aware limit checking** — define the rate source, rate time, validity window and rounding rule, and store all four with the transaction. **Limit ledger reconciliation** as a recurring control.

**D. KYC, identity and financial-crime controls.** Identity document regime per direction (HKID holder northbound; Mainland ID holder southbound) and the evidence standard for asserting it. **Name-record readiness** — the customer's English/Pinyin name on record for account-number-path payees, with a retrieval path for the customer. **Sanctions and AML screening overlay on the cross-boundary leg** (section 9) — screen the beneficiary and, where feasible, the beneficiary's institution and jurisdiction, and handle false-positive management at scale. **Velocity and behavioural monitoring** sized for a small-value, high-count, instant rail. **Payer confirmation evidence** — the fact and timestamp of the customer's confirmation of payee and rate, persisted for dispute and audit. **Suspicious-activity escalation** that works at real-time speed, so a hold/decline is possible inside the customer's session.

**E. FX conversion and disclosure.** Rate source and rate of record per direction. Rate display and confirmation at the documented points. Rate-validity window and refresh behaviour with defined customer-visible consequences. **Fee disclosure** — total cost = explicit fee + FX spread; disclose both, and note that HKICL lists "low transaction costs" as a scheme benefit. **Reproducibility** — given a transaction reference, reproduce the exact rate, amount and disclosure text shown to the customer.

**F. Reconciliation and accounting.** **Two clearing systems, two currencies** — reconcile against FPS-side and IBPS-side records in HKD and RMB respectively, and reconcile the internal limit ledger against both. **Three-way reconciliation** — customer instruction → your ledger → clearing record → settlement, with tolerances and an ageing/exception process. **Suspense and unallocated-credit handling** for credits arriving without a resolvable proxy or with a name mismatch. **FX position and treasury integration** if you carry conversion risk. **Per-institution, per-direction MIS**.

**G. Exception handling, returns and disputes.** **Decline reason taxonomy** mapped to customer messages — one message per real cause, no generic "payment failed". **Cut-off behaviour** for instructions inside/outside the southbound window and for payments in flight at the close. **Unreachable receiving institution** path — a distinct outcome with its own message. **Recall/refund request process** across the boundary; where no automated protocol exists, a documented manual claim path with SLAs. **Dispute and complaint handling** integrated with your existing framework, with cross-boundary specifics documented. **Error-code mapping**, versioned against operator releases.

**H. Customer disclosure and servicing.** Product terms stating direction-specific limits, hours, proxies, currency behaviour and the **per-institution** nature of the northbound annual cap. In-app guidance mirroring the Smart Tips, including receiving-side prerequisites (mobile-number linkage, English/Pinyin name, receipt currency, possible full-name-and-address requests). Contact-centre scripts for the top failure modes (insufficient headroom, out-of-hours, unresolvable proxy, name mismatch, unreachable bank). Multi-institution scope wording. Fraud awareness content on the corridor itself (section 9).

**I. Testing and certification.** Operator (**CNCC / HKICL**) certification as required for the corridor, scheduled as a critical-path item. Counterparty connectivity testing with a representative subset of receiving institutions. **Rule-engine test matrix** covering both directions; both proxy types; HKD-account and RMB-account payment; each limit boundary (daily and annual, at limit and one cent over); window boundaries (before open, at open, at close, after close); unreachable institution; name exact/partial/mismatch; missing English name; insufficient-currency receiving account; rate-refresh race. **Limit-ledger tests** including cross-channel aggregation, counter reset dates and reconciliation break injection. **Failure and recovery tests** — operator unavailability, partial credit, duplicate-submission protection. **Non-functional** — in-session latency, availability consistent with a 24x7 northbound service, corridor-specific monitoring. **Cutover and rollback plan**, with a staged participant rollout aligned to your counterparties.

### The two things teams most often get wrong

1. **Treating the annual cap as a global customer cap.** It is **per participating institution**. A global cap creates a self-inflicted service outage for customers who legitimately use more than one participant, and an incorrect disclosure; ignoring the cap fails a control obligation. The correct design is a precise **per-customer, per-institution** ledger with clean disclosure of scope.
2. **Reusing the northbound flow for southbound.** Different proxies, hours, receipt-currency behaviour and payer-eligibility rule. Reuse the framework; do not reuse the assumptions.

---

## 9. Compliance, AML and risk

> **What is documented vs what is inference.** This section distinguishes three things throughout: (a) **documented** first-party facts, (b) **standard supervisory practice** stated as such, and (c) **inference/analysis by this guide**, labelled. Do not read labelled inference as regulatory guidance.

### 9.1 The control problem the scheme creates by design

The corridor is, as designed, a **small-value, no-documentary-proof, instant, cross-jurisdictional, retail rail** in which the same-name and immediate-family restriction has been **lifted** (HKICL: recipients "not confined to same-name or immediate family members"; receiving institution "not confined to those within the same group"). Each feature is a deliberate benefit; together they mean **documentary control is absent by design** (documentary proof required = **Nil**, both directions); **relationship-based control is absent by design** (payer and payee need not be related or even known to each other); and **reversal is not the control** (the rail is instant and the consumer material offers no reversal mechanism). The controls that remain are **payer identity/eligibility verification, per-institution limit enforcement, proxy-resolution integrity, name verification at confirmation** (the partially masked name display), purpose coding, and **post-event monitoring and analytics**. That is a coherent design — but its control surface sits heavily in **monitoring and detection**, which is why the repo's fraud content is directly relevant: `banking/financial_fraud_detection_at_scale_guide.md`.

### 9.2 AML and sanctions considerations across the boundary

**Documented:** the service rules are silent on the AML/sanctions overlay, and there is no published corridor-specific AML rulebook in the material retrieved for this guide. **Standard supervisory practice is not optional** — a cross-boundary payment is a cross-boundary payment:

| Control area | Consideration for this corridor | Basis |
| --- | --- | --- |
| **Sanctions screening** | Screen the beneficiary name, beneficiary institution and jurisdiction exposure. The **best-effort name-only data** available (a partially masked name; an English/Pinyin string typed by the payer) is materially weaker than a structured beneficiary record — expect elevated false positives and design a proportionate review workflow | Standard practice + data quality implied by the documented flows |
| **Name/identity data quality** | The corridor transmits **name-in-English**, **masked names** and **account numbers**. Design screening to consume what actually arrives, and record the input form | Inferred from documented flows |
| **Transaction monitoring** | Small-value instant retail into a *different currency zone* with no documentary proof requires **velocity, aggregation and counterparty-behaviour** monitoring rather than document review | Standard practice |
| **Payer identity** | Eligibility is defined by identity document holder type (HKID holder / Mainland ID holder). Verifying the *identity document*, not merely the account relationship, is the substantive control | Documented eligibility rule + standard KYC |
| **Structuring and mule patterns** | Per-institution annual caps create a structural incentive to spread usage across institutions; the lifted same-name restriction widens the beneficiary population. Both are classic monitoring signals | Inferred analysis — labelled |
| **Purpose codes** | A structured purpose enumeration supports thematic monitoring and reporting; a purpose code that does not match behaviour is a signal | Documented enumeration + standard practice |
| **Record-keeping** | Persist the full instruction, the proxy used, confirmation evidence, the rate and the outcome. Cross-boundary records must be producible to two regulators | Standard practice |

This guide does **not** assert which specific regulatory notices, licences or reporting obligations attach to the corridor in each jurisdiction beyond the participant eligibility rules published by the authorities. Those are questions for your compliance function and your regulator.

### 9.3 Data protection in two jurisdictions

**The rules tables do not address data protection** — a fact, not an omission in this guide. What is materially true: the corridor causes **personal data to move between two legal regimes**, the Mainland's **Personal Information Protection Law (PIPL, 个人信息保护法 / 个保法)** and Hong Kong's **Personal Data (Privacy) Ordinance (PDPO)**; both impose obligations on collection, use, cross-border transfer, notice and security, and both create obligations engaged by a **cross-border transfer**; and the **data elements in flight are exactly the kind that are regulated** — mobile number, email address, FPS ID, bank account number, personal name (including English/Pinyin), identity-document-linked eligibility, and transaction amount.

**Inference (labelled).** A bank participating in the corridor should expect to need: a **lawful basis and a notice** covering the cross-boundary transfer and the purposes involved; a **data-transfer mechanism** satisfying whichever regime's cross-border rules attach to each direction of flow, documented per flow; **data-minimisation decisions** (e.g. whether full name and address are genuinely required per HKICL point 5, versus the minimum necessary); **retention and access rules** that satisfy two regimes simultaneously — in practice jurisdiction-specific retention with clearly-scoped access; and a **privacy impact assessment** before launch and on material change. **Honest boundary statement:** the specific transfer mechanisms, contractual clauses and consent/notice wording are **not established by this guide** and must be determined by your legal and privacy functions against the current text of PIPL and the PDPO. Nothing here is legal advice.

### 9.4 Fraud vectors on a real-time cross-boundary retail rail

> **Constructed — threat-modelling analysis by this guide, informed by the corridor's documented features. Not a published fraud taxonomy.**

1. **Social-engineering / authorised-push-payment (APP) fraud scaled across a border.** The lifted same-name/relationship restriction, instant irrevocability and a familiar "send to a mobile number" UX is precisely the shape APP fraud exploits. The cross-boundary dimension adds a **jurisdictional recovery gap**: the victim bank, the receiving bank and the two jurisdictions may have different recall norms, and recovery across that gap is slower and less certain than domestically.
2. **Name-masking exploitation.** The **partially masked** payee name is a weak signal; a fraudster's account may display a masked name consistent with the victim's expectation, and a habituated "tap to confirm" defeats the control. Mitigation is UX and education, not a technical block.
3. **Proxy-hijack and proxy-reuse.** Because so much addressing is by **mobile number**, proxy-ownership changes (number recycled, account relinked, SIM-swap) create directed misrouting risk — as do **email address** and **FPS ID** southbound.
4. **Mule accounts on the receiving side.** A per-institution annual cap plus a lifted relationship restriction gives a mule network a reason to spread funds across many paying customers and many institutions.
5. **Fake "payment received" and fake-support scams.** Customers who have just used a cross-boundary rail are a target population for follow-up impersonation — of the bank, of HKICL/CNCC, or of the receiving institution.
6. **Fraudulent look-alike websites and apps.** HKICL publishes recurring warnings to the public about fraudulent websites, and the HKMA maintains a "Beware of Fraudsters!" consumer page — **both real and citable**. The HKICL alerts-public-of-fraudulent-websites releases include items dated **14 September 2026** and **11 September 2026**, i.e. the warning stream is current and active, not historical. See <https://www.hkma.gov.hk/eng/smart-consumers/beware-of-fraudsters>.
7. **First-party, coercion and scam-centre flows**, where the payer is directed by someone else. Monitoring signals: unusual purpose codes, rapid repeat payments up to the daily cap, new beneficiaries, device/geography anomalies.

**On the fraud-warning sources:** the **existence and recency** of the HKMA page and the HKICL alerts are verified. This guide cites the September 2026 alert dates as reported; it does **not** reproduce the alert contents, because it did not retrieve them. Treat the dates as a pointer to re-check the live alerts.

### 9.5 The dispute and recourse path — the honest position

**Documented:** the scheme is real-time, with instant credit and a payer notification; there is no published chargeback mechanism for Payment Connect in the material retrieved. **Inference (labelled), stated as the working position:** on a real-time credit-push rail, once funds are credited the payment is ordinarily treated as **final**, with recourse through **claims and dispute processes** rather than automated reversal. A bank should therefore (1) treat pre-execution controls — name verification, rate confirmation, purpose, limit headroom, customer warning — as the **primary** loss-prevention mechanism, because post-execution recovery is structurally weaker; (2) build a **documented claim path** with defined evidence requirements and SLAs, and be honest internally that cross-jurisdiction recovery is uncertain; (3) **not promise customers a chargeback right the scheme does not provide**; and (4) ensure a **fraud-flagged** case has a defined escalation route on both sides and a contact at the counterparty institution. **What cannot be verified here:** whether any cross-boundary recall mechanism exists, its conditions, its time limits, or the liability allocation for a misdirected credit (section 13).

### 9.6 Operational continuity and concentration risk

> **Constructed — resilience analysis by this guide.**

| Dependency | Failure effect | Mitigation direction |
| --- | --- | --- |
| **The linkage itself** | The northbound corridor stops; there is no native alternative rail at this price point and service model | Fall back to a conventional cross-border remittance product with different cost, turnaround and evidence — **not** an equivalent substitute; disclose the difference |
| **Your domestic rail** (FPS or IBPS) | Your corridor capability stops even if the other side is healthy | Standard domestic-rail resilience — a prerequisite you already carry |
| **The operator on the other side** (CNCC / HKICL) | One direction fails | No local mitigation; requires counterparty communication and customer messaging |
| **The FX leg** | Payments cannot be priced or confirmed | Rate-source redundancy and a defined "rate unavailable" customer outcome |
| **Counterparty institution availability** | A specific receiving institution is unreachable | Routing/capability registry with a clear customer outcome — not a bank-wide outage |
| **Proxy-resolution dependency** | Failures at the receiving institution for mobile-number-linked accounts | Fall back to the account-number path where available; guidance to pre-check the record |

**The single-point-dependence statement for your resilience documentation:** *the corridor has no substitute with the same service characteristics.* A bank that has told customers the rail is "instant and 24x7" must have a scripted, honest answer for the period when it is not. Cross-references for the frameworks this section relies on: `banking/enterprise_risk_management_guide.md` (risk framework, operational-risk treatment, control design) and `banking/financial_fraud_detection_at_scale_guide.md` (detection and monitoring at retail volumes).

### 9.7 Questions for compliance and privacy functions

(1) What is our **notice and lawful basis** for the cross-boundary transfer, per direction, and is it already in our onboarding documents? (2) Which **cross-border data transfer mechanism** applies to our flows, documented per flow? (3) What **AML/sanctions screening standard** do we apply to a beneficiary known only by a masked name and an English/Pinyin string? (4) What is our **escalation route** to the counterparty institution for a fraud-flagged Payment Connect transaction? (5) What **record-retention** period and access regime satisfies both jurisdictions for this corridor? (6) What do we **promise customers** by way of recourse, and does that promise match what the scheme provides?

---

## 10. Strategic significance

### 10.1 The Greater Bay Area frame

Payment Connect is best read as an item in the **Guangdong–Hong Kong–Macao Greater Bay Area (GBA, 粤港澳大湾区)** integration agenda: a policy environment in which the movement of *people* between Hong Kong and the Mainland has increased, and in which the *payments* to support those people were historically served by slower, more expensive, evidence-heavy channels. It addresses a specific and unglamorous problem — **the daily-living payment**. HKMA's framing is explicit: the service will "cater to the daily needs of residents in both places for cross-boundary remittances and payments", and the published use cases are tuition fees, medical expenses and utility bills, plus business-to-personal disbursements such as salaries. This is not a wholesale rail and not a capital-account liberalisation measure; it is a **retail convenience rail** for a population that already crosses the boundary routinely. Four consequences: (1) a Hong Kong resident with family, property, education or medical needs in the Mainland can move small sums to an individual quickly, without assembling documentary evidence — the intended effect; (2) employers and platforms paying Mainland-based individuals gain a business-to-personal channel within the published use cases (salary disbursements are explicitly listed); (3) **Hong Kong's role as an offshore Renminbi business hub is reinforced in a retail-visible way** — Eddie Yue's statement ties the initiative directly to "an international financial centre and offshore Renminbi business hub", and receipt northbound is denominated in RMB; and (4) the corridor is denominated in the two currencies that matter to the GBA, with a defined FX behaviour per direction rather than a homogenised "convert at the door" model.

### 10.2 Payment interoperability between the two markets

What Payment Connect demonstrates at the level of principle is **interoperability between two domestic fast-payment systems with different rules, different currencies and different clearing operators** — achieved without merging or replacing either system. The pattern: keep each domestic rail domestic; agree a cross-boundary arrangement; expose a curated, direction-specific, limit-bounded service scope. That is what makes it interesting beyond this corridor — interoperability of *faster payment systems* is a recurring theme in international payments policy, and a live bilateral implementation between systems as large as FPS and IBPS is a reference point for extending a national fast-payment system cross-border **without** turning it into a general-purpose cross-border payment system. Be precise about what has *not* happened: the corridor does not give IBPS direct reach into Hong Kong for all purposes, does not give FPS direct reach into the Mainland for all purposes, and does not create a new currency or settlement asset. It is a bounded arrangement.

### 10.3 What it means for participating vs non-participating institutions

> **Constructed — competitive and architectural analysis built on the documented fact that the service is delivered by *participating* institutions only.**

The hard fact underneath everything here: **the service is provided by participating institutions.** You are either on the list, or your customers cannot use you for this corridor.

| | **Participating institution** | **Non-participating institution** |
| --- | --- | --- |
| **Customer capability** | Native: send northbound / receive southbound per your implemented scope | None natively; you must route the customer elsewhere, or lose the flow |
| **Customer experience** | In-app, mobile number or account number, name verification, instant confirmation | A third-party remittance journey: different cost, different evidence, different turnaround |
| **Cost position** | HKICL lists "low transaction costs" as a scheme benefit — a low-cost rail is available to you | You cannot match the corridor's price on this use case |
| **Competitive read** | Your cross-boundary-adjacent base is retained in-app for a daily-living need | A customer with a recurring Mainland payment need has a concrete reason to hold an account at a participant |
| **Operational burden** | Certification, limit ledgers, reconciliation across two systems, KYC per direction, monitoring, disclosure | None of that — but also none of the capability |
| **Strategic option** | Extend scope over time (directions, channels, product types) as counterparties roll out | Re-assess periodically; participation grows over time per the launch statement |

The asymmetry is the ordinary one for network participation: the cost of *not* being on the list is borne by the customer relationship, and is most acute for institutions whose customers have the strongest Hong Kong–Mainland ties. The mitigating structural fact is that **participation is directional and incremental** — institutions "will roll out the service gradually and more institutions will join over time", and the SPDB footnote proves a participant can be live in one direction only. A bank can enter in the direction where its customer need is strongest and extend later.

### 10.4 The wider cross-boundary payment agenda — documented vs direction

**Documented and citable:** Payment Connect itself (MOU **02 Aug 2024**; launch **22 Jun 2025**; HKMA and PBoC as joint authorities); the **HKMA publishes a "Payment Connect" consumer page** as one of its Smart Consumers topics, alongside the Faster Payment System, e-Wallets and Prepaid Cards, e-Payment and Transfer, Internet Banking and Beware of Fraudsters — i.e. the corridor is presented as part of Hong Kong's mainstream consumer payment landscape, not a niche scheme; and the HKMA's own framing of Payment Connect as "another milestone for the FPS in **expanding cross-boundary payment**", which posits cross-boundary expansion as a continuing direction of travel.

> ### Direction, not fact
>
> The following are areas where you will encounter commentary, roadmaps and speculation. **This guide does not assert any of them as established.** They are listed so you know what to re-verify, not to be inferred from: **further expansion of FPS cross-boundary linkages to other jurisdictions** (the HKMA's framing supports a *direction* of expansion; no specific additional corridor is asserted here); **multi-central-bank digital currency (multi-mCBD) arrangements** and their relationship to retail cross-boundary payments, including **e-CNY**'s cross-boundary use in Hong Kong (active policy topics — this guide makes no claim about their status, scope or relationship to Payment Connect, having not verified them at primary source); **extension of Payment Connect's service scope** beyond P2P remittance and the published daily-living use cases (the progressive-rollout note establishes that features are added over time, and that one specific pending item is the **optimisation and gradual rollout of support for Mainland accounts held by Hong Kong residents** — that item is documented; anything beyond it is not); **inclusion of Mainland third-party wallets or additional institution classes** (the participant lists are the evidence base — do not extrapolate); and **changes to the limits, quotas or hours** (treat every figure in section 5 as a snapshot that will move). **How to handle this block in your own documents:** copy the "Documented and citable" list, and either delete this block or reproduce it verbatim as an explicitly-unverified watch list. Do not promote a direction to a fact.

---

## 11. The Cymbal Bank worked example

> **Cymbal Bank is a fictional institution used as the only bank persona in this guide. All figures in this section are fictional and illustrative only. They are not benchmarks, not market data, and not derived from any real institution's economics. Do not use them for planning.**

### 11.1 The decision to be taken

**Situation.** Cymbal Bank is a mid-sized Hong Kong retail bank with customers including a substantial cohort with Mainland family, education and property ties, and a smaller cohort of corporate clients paying Mainland-based individuals. Cymbal Bank is **not** currently a Payment Connect participant; its cross-boundary retail offering is a conventional outward remittance product (forms or in-app instruction, evidence requirements, a daily cut-off, T+0/T+1 value, a per-transaction fee and an FX spread). **Observed customer behaviour (illustrative):** a meaningful share of outward remittance volume is small-value and personal — tuition support, family support, medical costs, utility payments for a Mainland property — which is precisely the set of use cases the scheme is published for. **The decision:** whether to participate in the HK–Mainland Payment Connect linkage, in which direction(s), and how to prepare operations for the **Northbound** arrangement (Hong Kong → Mainland P2P remittance).

### 11.2 Eligibility and product design under the limits and quotas

**Step 1 — Confirm eligibility.** The corridor is delivered by *participating institutions*. Cymbal Bank must (a) already be, or become, an FPS-connected institution on the Hong Kong side, and (b) obtain participation through the applicable process with HKICL and the scheme's authorities, including certification. **This is a gate, not a task.**

**Step 2 — Design the product against the published rules, per direction.** Cymbal Bank elects **Northbound only** in phase 1, so its product definition has exactly one rule set:

| Product parameter | Cymbal Bank's Northbound design (illustrative) | Source of the constraint |
| --- | --- | --- |
| Eligible payer | Hong Kong Identity Card holder, identity-verified at onboarding and asserted at instruction | Published eligibility rule |
| Daily limit | HKD 10,000 per person, per institution | Published limit |
| Annual limit | HKD 200,000 per person, **per institution** | Published limit — the per-institution scope is decisive for design |
| Documentary proof | None collected | Published rule |
| Payee restriction | **Personal account holders only** | Published rule |
| Payees reachable | Mainland participating institutions supporting the corridor and the relevant proxy path | Participant list + progressive-rollout note |
| Proxies supported | Mobile number; bank account number | Published proxy set |
| Service hours | 24x7 | Published |
| Receipt currency | RMB | Published |
| Rate treatment | Rate confirmed at confirmation step when the HKD account is the payment account; RMB amount and HKD equivalent displayed | Published flow |

**Step 3 — Decide what *not* to build in phase 1.** Explicitly out of scope: Southbound as a *sending* capability (Cymbal Bank is a Hong Kong institution; southbound sending is a Mainland-resident capability); the southbound *receiving* capability (retained for a later phase); corporate/business-to-personal disbursement products (published as a use case, but a different product and control set); and the "pilot basis" remittance use case, explicitly "currently limited to pilot institutions… Please consult the receiving banks before making the payment".

**Step 4 — Positioning: replace or complement?** The rules point the answer — the published limit is explicitly **separate from the existing daily remittance limit of RMB 80,000 that applies to same-name transfers**, so the two products have **independent headroom**. Cymbal Bank positions the corridor as an **additional, lower-friction small-value channel**, and must state clearly in its terms that using one does not consume the other.

### 11.3 Limit enforcement and reconciliation design

This is where the per-institution scope of the annual limit creates real engineering. Design decisions (illustrative): (1) **one usage ledger per customer, per institution**, keyed (customer ID, institution = Cymbal Bank, direction = northbound, currency = HKD, period = annual), with a separate daily counter; (2) **an accumulation basis defined once** — what counts (executed payments) and at what lifecycle point (on acceptance, on credit confirmation, or on settlement); recommended is **on accepted execution**, with a reconciliation adjustment path, documented because the interpretations differ at the margin; (3) **cross-channel aggregation** — mobile banking, internet banking, contact centre and assisted channels all post to the same counter, with per-channel counters explicitly rejected as a control failure; (4) **no cross-institution global cap** — Cymbal Bank enforces *its own* HKD 200,000 per customer and **does not** model the customer's usage elsewhere, while **disclosing the per-institution scope** in the app and terms and recording the acknowledgement; (5) **independence from the RMB 80,000 same-name limit** — physically separate counters, no netting, separate disclosure; (6) **FX-aware limit checking** — because the cap is HKD and the receipt is RMB, every limit check carries a rate reference and timestamp, with a defined, stored rounding rule; (7) **remaining-allowance display** — the app shows "your remaining allowance with Cymbal Bank", not "your remaining allowance", because the wording is a design requirement, not a nicety; and (8) **reconciliation** — daily against executed northbound payments, weekly against the clearing records available to Cymbal Bank, monthly aged exception review with each unreconciled item owned by a named function. The ledger is a **control artefact**, not a UI convenience.

**Why this matters, concretely.** The most likely failure is a customer who uses Cymbal Bank and one other participant and is told by Cymbal Bank that they have exhausted "their" allowance. Under the published rule that statement is wrong — it is *Cymbal Bank's* allowance that is exhausted. Fixing this in production is a customer-relations problem; fixing it in design is a wording and data-model decision.

### 11.4 KYC / AML overlay (illustrative design)

**Payer eligibility as a KYC fact** — HKID-holder status captured at onboarding, versioned, asserted at each instruction; because documentary proof is **Nil** by scheme design, the *identity verification already on file* is the substantive control. **Payee screening with what actually arrives** — the beneficiary is screened on the **name-in-English** string or the **partially masked name**, plus the beneficiary institution; Cymbal Bank accepts an elevated false-positive rate as a design parameter and staffs a review queue accordingly, rather than pretending the name data equals a structured beneficiary record. **Real-time-decision capability** — a small set of in-session decline triggers, with everything else routed to post-event review with defined follow-up. **Monitoring tuned for the corridor** — velocity per customer, per beneficiary, per device and per geography; repeat payments at or near the daily cap; rapid beneficiary churn; purpose-code/behaviour mismatch; first-time payment to a new beneficiary at high value relative to the customer's history. **Purpose codes as a monitoring and reporting input**, versioned and mapped to Cymbal Bank's product taxonomy. **Privacy** — cross-boundary transfer notice and lawful basis per direction; a data-minimisation review of the full-name-and-address case (HKICL receive-side point 5) before transmitting anything beyond the minimum; and a privacy impact assessment as a phase gate.

### 11.5 Customer disclosure and dispute handling (illustrative)

**Disclosure pack — mandatory items:** per-direction rules and limits; the **per-institution** scope of the annual cap; service hours and what happens outside them; proxies supported; receipt currency; when and how the rate is fixed; total cost (fee + spread); that documentary proof is not required but the payee must be a personal account holder; **the receiving-side prerequisites** (Mainland payee must have enabled cross-boundary receipt via mobile number; English/Pinyin name on record for the account-number path); that the payment is **instant and not ordinarily reversible**; and the fraud warning. **Dispute handling:** Cymbal Bank takes an explicit position — pre-execution controls are the primary loss prevention; disputes run through a documented claim path with defined evidence and SLAs; **no chargeback right is promised**; a fraud-flagged case has a named escalation contact at the counterparty institution and a defined cross-boundary escalation route; and complaint themes feed back into the Smart Tips-style in-app guidance.

### 11.6 Integration and testing approach (illustrative)

FPS connectivity is assumed existing (prerequisite — confirm version and certification status). The corridor capability is obtained from the operator under the participation agreement and implemented in a **dedicated cross-boundary service**, not by extending the domestic flow inline. The rule engine is direction-parameterised, with phase 1 instantiating northbound only. The participant registry is data-driven and keyed on (institution, jurisdiction, direction, channel, product), with a defined refresh process. Proxy resolution covers the mobile-number and account-number paths, with the account-number path requiring name-in-English capture and a name-match policy. The limit ledger is dedicated and reconciled per customer per institution. FX is a rate service with timestamps, validity window and full disclosure reproducibility. Reconciliation is an automated daily ledger reconciliation with an aged-exception queue. Testing covers operator certification, counterparty connectivity, a rule matrix spanning limit boundaries, window boundaries, proxy failures, name mismatches, missing English names, unreachable institutions and rate-refresh races, plus failure/recovery and duplicate-submission protection, plus non-functional latency and availability. Cutover is staged — staff pilot, then a limited customer cohort, then general availability — with defined rollback.

### 11.7 Cost / benefit — illustrative numbers only

> **All figures below are fictional and illustrative. They are invented for this worked example and do not represent any real bank's costs, revenues or market data.**

| Item | Illustrative figure | Note |
| --- | --- | --- |
| Programme cost, phase 1 (build + certification + testing) | ~HKD 18m | Dominated by integration, certification and testing |
| Run cost increase (annual) | ~HKD 3.5m | Monitoring, reconciliation, contact centre, compliance review |
| Customer service calls avoided (annual) | ~9,000 | Customers self-serving a need that currently generates manual remittance queries |
| Small-value outward remittance volume addressed | ~HKD 1.4bn/yr | The subset within the corridor's use cases and limits |
| Revenue at materially lower per-transaction economics | ~HKD 6m/yr | Lower unit price on higher-retained volume |
| Retention effect (illustrative) | ~+1.5% of the cross-boundary-cohort balance | The strategic, hardest-to-verify line item |
| **Payback** | ~3–4 years on the above | Driven by retention, not by fee revenue |

**The honest read:** the corridor's direct fee economics are unattractive on their own — "low transaction costs" is a documented scheme *benefit*, which is good for customers and bad for unit revenue. The business case rests on **retention and relationship depth** in the cross-boundary cohort, plus avoiding customers opening an account at a participant specifically to get this capability. If strategic planning does not accept a retention-based case, the correct decision is **not to participate**, and to route customers to a participant instead.

### 11.8 Gate plan

> **Constructed plan. Phases, criteria and figures are illustrative.**

| Gate | Phase | Go/no-go criteria |
| --- | --- | --- |
| **G0 — Eligibility** | Pre-commitment | FPS participation confirmed; participation route and certification requirements obtained from the operator; named executive sponsor |
| **G1 — Business case** | Pre-build | Retention-based case accepted; phase 1 scope (northbound only) fixed; disclosure requirements costed; complement-vs-replace decision recorded |
| **G2 — Design readiness** | Pre-build | Per-institution limit-ledger design signed off; per-direction rule engine design signed off; name-match policy decided; rate-disclosure and reproducibility design signed off; privacy impact assessment complete |
| **G3 — Build complete** | Pre-certification | Rule matrix executed green; limit ledger reconciling; monitoring rules in production config; disclosure pack legal-reviewed; contact-centre scripts and training complete |
| **G4 — Certification** | Pre-launch | Operator certification passed; counterparty connectivity tests passed; failure/recovery tests passed; cutover and rollback plans signed off |
| **G5 — Limited launch** | Launch | Staff pilot clean; first-cohort limit utilisation within expectation; zero control failures; complaint themes reviewed weekly |
| **G6 — General availability** | Post-launch | Stability target met; reconciliation breaks within tolerance; phase 2 decision (southbound receiving, business-to-personal) taken on evidence |

### 11.9 What Cymbal Bank must still establish before committing

None of these can be answered from the public record, and each is a reason not to commit early: (1) the corridor's **clearing, settlement and FX mechanics as they apply to Cymbal Bank** — who provides the rate, at what time, at whose risk, with what disclosure obligation (section 4, questions 1–2); (2) the **cross-boundary message/interface specification and certification requirements** from the operator (section 3); (3) the **return, recall and refund protocol**, if any, and the **liability allocation** for a misdirected credit (section 4, questions 5–6); (4) the **participation process, costs and timetable** for a new participant, including whether participation is granted per direction and what obligations attach; (5) the **counterparty picture** — which Mainland institutions Cymbal Bank's customers actually need to reach, and whether the coverage gap (section 6) is acceptable to them; (6) the **privacy and data-transfer position** per direction, signed off by legal (section 9.3); and (7) the **supervisory position** — whether any notification, approval or reporting obligation attaches to participation in each jurisdiction. Until these are answered in writing, the correct programme status is **"assessing"**, not **"committed"**.

---

## 12. The claims audit

**Verified** = established at a first-party source (HKMA/HKICL/CNCC/PBoC) or from a press release reproduced for this guide. **Flagged** = attributed to a secondary source, or true-with-a-caveat. **Rejected** = asserted in circulating material and contradicted by a first-party source.

| # | Claim | Source | Date of claim | Status |
| --- | --- | --- | --- | --- |
| 1 | Payment Connect is the linkage of the Mainland's IBPS and Hong Kong's FPS | HKMA Payment Connect consumer page; HKICL FPS Payment Connect page | HKMA page revision 08 Sep 2026; HKICL current | **Verified** |
| 2 | PBoC and HKMA jointly support/govern the linkage; service scope and use cases provided by both | HKMA + HKICL pages | As above | **Verified** |
| 3 | MOU signed 02 Aug 2024 by Lu Lei (PBoC Deputy Governor) and Howard Lee (HKMA Deputy Chief Executive), establishing a cooperation framework for the linkage of fast payment systems | HKMA press release 20240802-5 | 02 Aug 2024 | **Verified** |
| 4 | MOU establishes a *cooperation framework*; its operative terms are not public | HKMA press release 20240802-5; absence of published text | 02 Aug 2024 | **Verified (as a limit on knowledge)** — see §13 |
| 5 | Announcement on 20 Jun 2025; service launched to the public **22 June 2025** | HKMA press release 20250620-4; HKICL Payment Connect page | 20 Jun 2025 / 22 Jun 2025 | **Verified** |
| 6 | Six institutions each from the Mainland and Hong Kong at launch; gradual rollout; more institutions to join over time | HKMA press release 20250620-4 | 20 Jun 2025 | **Verified** |
| 7 | Launch ceremony held in Beijing; attendees as listed (Pan Gongsheng, Wang Linggui, Eddie Yue, Lu Lei, Zhang Hui) | HKMA press release 20250620-4 | 20 Jun 2025 | **Verified** |
| 8 | Eddie Yue quotation on Payment Connect (full text reproduced in §4) | HKMA press release 20250620-4 | 20 Jun 2025 | **Verified** |
| 9 | **Northbound daily limit HKD 10,000 per person; annual HKD 200,000 per person, per participating institution** | HKMA + HKICL rules tables | 08 Sep 2026 page revision | **Verified** |
| 10 | The northbound limit is **separate from** the existing daily RMB 80,000 same-name remittance limit | HKMA + HKICL rules tables | 08 Sep 2026 | **Verified** |
| 11 | **Southbound** limit: existing facilitative FX arrangement for Mainland residents, annual quota equivalent to **USD 50,000 per person** | HKMA + HKICL rules tables | 08 Sep 2026 | **Verified** |
| 12 | Documentary proof required: **Nil** both directions; northbound payee restricted to **personal account holders only** | HKMA + HKICL rules tables | 08 Sep 2026 | **Verified** |
| 13 | Northbound hours **24x7**; southbound **16x7**, currently 07:00–23:00 daily, may vary by Mainland institution | HKMA + HKICL rules tables + footnote 1 | 08 Sep 2026 | **Verified** |
| 14 | Northbound proxies: mobile number, bank account number. Southbound proxies: mobile number, email address and FPS ID, bank account number | HKMA + HKICL rules tables | 08 Sep 2026 | **Verified** |
| 15 | Use cases: personal fees (tuition, medical, utilities); business-to-personal (e.g. salary); pilot-basis remittance limited to pilot institutions — consult receiving banks | HKMA + HKICL use-case tables | 08 Sep 2026 | **Verified** |
| 16 | Progressive rollout: functions/scope vary by institution; some features introduced gradually; HK payers can input a Mainland mobile number for accounts that enabled cross-boundary receipt by mobile number; Mainland accounts held by HK residents still being optimised and rolled out gradually | HKMA + HKICL notes | 08 Sep 2026 | **Verified** |
| 17 | **Hong Kong participating institutions: 26** (numbered 1–26) | HKMA Payment Connect consumer page | as of **2 Sep 2026** | **Verified** |
| 18 | **Mainland participating institutions: 18** (numbered 1–18) | HKMA consumer page (1–18) and HKICL page (same 18 names) | as of **2 Sep 2026** | **Verified** |
| 19 | **"16 Mainland participating institutions"** | Circulating secondary commentary / launch-cohort-derived summaries | stale | **REJECTED** — contradicted by the first-party lists, which show 18 |
| 20 | **Shanghai Pudong Development Bank Co., Ltd. appears on BOTH lists** (HK #23; Mainland #16), Mainland entry carrying footnote 3: **"Only supports Southbound (The Mainland to Hong Kong) P2P Remittance"** | HKMA consumer page, footnote 3 | 08 Sep 2026 revision | **Verified** — participation is directional |
| 21 | HKICL benefits: receiving institution not confined to the same group; recipients not confined to same-name or immediate family; wide range of current-account payment including P2P and daily-living; anytime, anywhere, settled instantly; mobile number and other proxies; low transaction costs; RMB or HKD remittance with receipt in RMB (northbound) and receipt in HKD or RMB (southbound) | HKICL Payment Connect page, "Benefits" | current | **Verified** |
| 22 | Northbound mobile-number flow: the 5 documented steps, including the **partially masked payee name** verification and the exchange-rate confirmation | HKICL page, "How to use" | current | **Verified** |
| 23 | Northbound account-number flow: the 5 documented steps, including **payee name in English** | HKICL page, "How to use" | current | **Verified** |
| 24 | Southbound receive-side: the 6 documented points, including no further FPS registration if the account is already linked to an FPS proxy; advance agreement of receipt currency; certain Mainland banks may require the payee's full name and address; funds credited immediately with a notification to the payer | HKICL page, "How to Receive Funds from a Payer in the Mainland" | current | **Verified** |
| 25 | Smart Tips list (app updated; Mainland payee enabled mobile-number receipt; English/Pinyin name in record; confirm details in advance; verify receiving-account currency capability) | HKICL page, "Smart Tips" | current | **Verified** |
| 26 | IBPS is built by CNCC under the guidance of the PBoC; enables real-time cross-bank retail payments via mobile/online banking; cross-bank account management, fund transfer, fund consolidation "without leaving home"; instant results | HKMA consumer page "About IBPS"; CNCC's own site | 08 Sep 2026 / CNCC pages | **Verified** |
| 27 | **IBPS launched 30 August 2010**, rolled out nationwide **24 January 2011**; the first business system of the second-generation payment system to enter production | CNCC, 网上支付跨行清算系统 IBPS, 15 Jun 2016 (<https://www.cncc.cn/zfxt/201606/t20160615_416.html>); CNCC 银清科技 page | CNCC pages | **VERIFIED at the operator's own site** — the previously "reported, not confirmed" launch date is now confirmed |
| 28 | 2016 IBPS model change from "real-time forwarding, real-time netting" to **"real-time forwarding, scheduled netting"**, addressing the hotspot-account problem for high-volume participants | CNCC 银清科技 page (<https://tech.cncc.cn/ywjs/202212/t20221212_1182.html>) | CNCC page | **Verified at operator's site** |
| 29 | IBPS supports cross-bank payment, cross-bank account information inquiry and online signing; 7×24 continuous operation | CNCC pages | CNCC pages | **Verified at operator's site** |
| 30 | IBPS transaction limit "under RMB 50,000 per transaction" | Chinese-language encyclopaedia entry (secondary) | secondary | **Flagged** — reported, not confirmed at primary source |
| 31 | IBPS 2023 volumes ≈ 17bn transactions / ≈ RMB 301 trillion; HVPS 2023 ≈ 382m / ≈ RMB 8,481 trillion | ClearingPost industry guide (secondary synthesis of PBoC/BIS/World Bank sources) | guide published 12 Mar 2026 | **Flagged** — attributed secondary, not PBoC-published as reproduced here |
| 32 | CNAPS architecture: National Processing Center plus City Clearing Processing Centers in ~32 cities as regional nodes | Practitioner synthesis of BIS *Red Book* country volume for China; PBoC payment-system overview | secondary | **Flagged** — widely reported, not confirmed against the primary specification here |
| 33 | CNAPS II widely described as having moved to ISO 20022-based message formats | Practitioner and vendor documentation; ISO 20022 migration trackers | secondary | **Flagged** — direction only; **no specific message identifiers are asserted in this guide** |
| 34 | FPS launched September 2018, connecting banks and SVF operators, enabling real-time transfers by mobile number or email address | HKMA press release 20250620-4 footnote 2; HKMA consumer page | 20 Jun 2025 | **Verified** |
| 35 | HKMA "Beware of Fraudsters!" consumer page exists and is current | HKMA Smart Consumers topic list | 08 Sep 2026 | **Verified** |
| 36 | HKICL publishes recurring "alerts public of fraudulent websites" releases, including items dated 14 Sep 2026 and 11 Sep 2026 | HKICL press releases as reported | Sep 2026 | **Verified as to existence and recency** — contents not retrieved or reproduced here |
| 37 | Mainland list includes no Mainland big-tech wallet and no rural/county banks; Hong Kong list includes four virtual banks and one SVF | Derived by inspection of the HKMA lists | as of 2 Sep 2026 | **Verified by inspection** — an observation about the snapshot only |
| 38 | Participant counts, limits, hours and use cases are the **fastest-moving** data in this guide | Judgement based on the pages' revision and as-of dates | 08 Sep 2026 / 2 Sep 2026 | **Flagged as a standing caution** |

### The staleness ranking — what to re-check, in order

1. **Limits and quotas** (§5) — consumer service rules that move with policy.
2. **Participating institution lists and channels** (§6) — the launch statement promised continued additions, and the as-of dates exist because the lists change.
3. **Service hours, especially the southbound window** (§5) — the footnote itself says hours "may vary subject to individual Mainland participating institutions", so even the published figure is a variable.
4. **Feature availability per institution** (§5 note, §7) — the progressive-rollout note means the scheme's *permitted* scope and any given institution's *delivered* scope differ, and the latter changes without a scheme-level announcement.
5. **Volumes and architecture descriptions** (§3) — the slowest-moving, but also the least first-party in this guide.

---

## 13. What Could Not Be Verified

Everything below is a gap in the public record as retrieved for this guide. **None of it is asserted elsewhere in this guide as fact.** Obtain it from the operator or your counterparty — do not infer it.

**13.1 The MOU's internal terms.** **Not public.** The documented content of the 02 Aug 2024 MOU is that it **establishes a cooperation framework for the linkage of the fast payment systems** between the Mainland and Hong Kong, signed by Mr Lu Lei (PBoC Deputy Governor) and Mr Howard Lee (HKMA Deputy Chief Executive). No operative term is reproduced here, because the text was not published; any statement about what the MOU "provides" beyond the cooperation framework is unsupported.

**13.2 Message standard and interface specification details.** **Not publicly documented.** The specific interface specification, version, message identifiers, field layouts, or a proprietary standard name, for IBPS or for the cross-boundary corridor. This guide asserts only that CNAPS II is **widely reported** to be ISO 20022-aligned and explicitly declines to name message types. A bank must obtain these from CNCC (and/or HKICL) under the participation agreement. No specification number is cited because none was verifiable.

**13.3 Settlement, clearing and FX mechanism.** **Not public:** how value moves between the two clearing estates; whether there is a bilateral settlement position, a settlement bank or a correspondent chain; who performs FX conversion, at what rate and at whose risk; the rate's validity window; the intraday liquidity and funding arrangement; and the exposure if one side's window closes with a payment in flight. The consumer material documents only that the customer confirms "the applicable exchange rate" and sees the RMB amount and HKD equivalent.

**13.4 Returns, recalls, refunds and dispute mechanics.** **Not public in the material retrieved:** a cross-boundary return/recall protocol; reversal windows; who may initiate; behaviour when the receiving account is closed or the credit rejected; error/decline reason codes and their mapping; liability allocation for a misdirected or mis-credited payment; and the formal customer recourse path across the boundary. Section 9.5 states a **working position** (final on credit; recourse by claim) and labels it explicitly as inference.

**13.5 IBPS's own launch date — now resolved.** The launch date was **verified at the operator's own site** rather than left "reported": CNCC states IBPS went into production on **30 August 2010** and was extended nationwide on **24 January 2011**, as the first business system of the second-generation payment system. **This item is closed**; it is recorded here because it began as an unverified claim and the closure is part of the audit trail (§12, row 27).

**13.6 IBPS volume, capacity and limit figures.** **Not verified at a primary source in this guide.** The ~17bn / ~RMB 301 trillion (2023) figures are **attributed secondary**, as are the HVPS comparators; the "under RMB 50,000 per transaction" figure is **Chinese-language secondary**. **Participant throughput ceilings, message limits, redundancy topology and node counts are not established here at all.** For a design document, take these from PBoC's own payment system reports and from CNCC as a participant.

**13.7 The CNAPS architecture details.** The NPC/CCPC topology and the commonly cited "32 cities" figure are **widely reported from practitioner synthesis of the BIS *Red Book* and PBoC overview material** — not confirmed here against the primary source. The `banking/cnaps_guide.md` companion is the right place for that question; it is being written concurrently and this guide has not read it.

**13.8 Participant capability details beyond the channel field.** The published lists give **name, channels supported and contact details**, with channels self-reported. **Not established here:** which proxy types each institution supports; which directions each institution actually offers in practice (beyond the SPDB southbound-only footnote); per-institution service hours; per-institution limits or sub-limits beyond the scheme rules; and each institution's cut-offs. The progressive-rollout note guarantees these vary; it does not tell you how.

**13.9 The launch annex institution list.** The **20 Jun 2025 launch annex** (listing the six-plus-six launch institutions) is cited on the press release as a PDF at <https://www.hkma.gov.hk/media/eng/doc/key-information/press-release/2025/20250620e4a1.pdf>. **This guide did not retrieve and confirm the annex contents**, and therefore does not name the launch cohort. The cohort size (six each) is verified from the release text; the names are not.

**13.10 Onward/underlying FX arrangement details for the southbound direction.** The southbound direction follows "the existing facilitative foreign exchange arrangement available for Mainland residents, subject to an annual quota equivalent to USD 50,000 per person". **The underlying arrangement's terms are not reproduced here** — the guide quotes the rule as published and does not describe the arrangement itself, its documentation or its administration.

**13.11 Cross-boundary recall and fraud escalation contacts.** **Not public:** a published cross-boundary fraud escalation route, or a named contact framework between institutions for flagging a Payment Connect fraud. Section 11.5 therefore treats establishing one as a bank's own duty.

**13.12 Items deliberately not asserted.** For the avoidance of doubt, the following are **not claimed** anywhere in this guide: any additional Payment Connect corridors to other jurisdictions; the status, scope or relationship of multi-mCBD or e-CNY cross-boundary work to this corridor; any extension of the scheme beyond the published use cases; any specific regulatory reporting obligation beyond the published eligibility rules; and any specific rate, fee or volume figure for the scheme itself.

---

## 14. Glossary

Terms are given English ↔ Chinese. Where a Chinese term is **standard and sourced in this guide** it is marked *(sourced)*; where a romanisation or translation is **uncertain or provided for orientation only**, that is marked explicitly. Do not treat an "uncertain" entry as an authoritative rendering.

| English | Chinese | Notes | Basis |
| --- | --- | --- | --- |
| **Internet Banking Payment System (IBPS)** | **网上支付跨行清算系统** | The Mainland's real-time retail payment system; the subject of sections 2–3 | *(sourced)* — CNCC's own site and the HKMA consumer page use this pairing |
| **Faster Payment System (FPS)** | **转数快** | Hong Kong's faster payment system, launched September 2018, connecting banks and SVF operators; real-time transfers by mobile number or email address | *(sourced)* — HKMA's Chinese-language pages use 转数快 |
| **People's Bank of China (PBoC)** | **中国人民银行** | The central bank; joint authority for Payment Connect | *(sourced)* — the official pairing |
| **Hong Kong Monetary Authority (HKMA)** | **香港金融管理局** | Hong Kong's central banking institution; joint authority for Payment Connect | *(sourced)* — official pairing |
| **China National Clearing Center (CNCC)** | **中国人民银行清算总中心** | Operator of IBPS and the CNAPS family, under PBoC guidance | *(sourced)* — the entity's own published Chinese name |
| **City Clearing Processing Center (CCPC)** | **城市处理中心** | Regional node in the CNAPS architecture connecting local participants | *(sourced)* — standard CNAPS terminology; see §3 status note on "32 cities" |
| **High Value Payment System (HVPS)** | **大额实时支付系统** | The RTGS component of CNAPS; large-value, time-critical payments; the final settlement layer | *(sourced)* — standard CNAPS terminology |
| **Bulk Electronic Payment System (BEPS)** | **小额批量支付系统** | The batch component of CNAPS; mass low-value payments | *(sourced)* — standard CNAPS terminology |
| **NetsUnion Clearing Corporation (NUCC)** | **网联** | The clearing platform for third-party payment platform flows (Alipay, WeChat Pay); separate from IBPS | *(sourced)* — 网联 is the common short form; full name 网联清算有限公司. **Covered in depth in `banking/nucc_netsunion_guide.md`** |
| **Cross-border Interbank Payment System (CIPS)** | **人民币跨境支付系统** | The cross-border RMB payment system; structurally separate from the domestic retail rails discussed here | *(sourced)* — standard terminology |
| **CNAPS (China National Advanced Payment System)** | **中国现代化支付系统** | The umbrella programme for the PBoC's core payment infrastructure; IBPS is one subsystem within its second generation | *(sourced)* — standard terminology; **covered as a whole in `banking/cnaps_guide.md`** |
| **Greater Bay Area (GBA)** | **粤港澳大湾区** | The Guangdong–Hong Kong–Macao Greater Bay Area policy framework | *(sourced)* — standard official rendering |
| **Personal Information Protection Law (PIPL)** | **个人信息保护法**; commonly **个保法** | The Mainland's personal information protection statute | *(sourced)* — official title; 个保法 is the common short form |
| **Personal Data (Privacy) Ordinance (PDPO)** | **個人資料（私隱）條例** | Hong Kong's data protection statute | *(sourced)* — official title in traditional Chinese |
| **Mobile number** (account proxy) | **手机号码** | Supported as a proxy in **both** directions | *(sourced)* — standard term; proxy support from the HKMA/HKICL rules tables |
| **Bank account number** (account proxy) | **银行账号** | Supported as a proxy in **both** directions | *(sourced)* — standard term |
| **Email address** (account proxy) | **电子邮件地址** | Supported as a proxy in the **southbound** direction only, per the published rules table | *(sourced)* — standard term; directional assertion from the rules table |
| **FPS ID** (account proxy) | *(no separate Chinese term asserted here)* | A system-assigned FPS identifier; supported **southbound** per the published rules table. Frequently written "FPS ID" in Chinese-language HK material | **Marked uncertain** — no distinct official Chinese rendering could be verified, so none is asserted |
| **Northbound** | *(no single official Chinese term asserted here)* | Hong Kong → The Mainland | **Marked uncertain** — the directional labels appear in English in the sources retrieved |
| **Southbound** | *(no single official Chinese term asserted here)* | The Mainland → Hong Kong | **Marked uncertain** — as above |
| **Cross-bank** | **跨行** | As in 跨行支付, cross-bank payment; the property that makes a clearing system necessary | *(sourced)* — CNCC's own wording (跨行（同行）资金汇划处理) |
| **Fund consolidation** | **资金归集** | Pulling balances together across accounts at different banks; a documented IBPS use case | *(sourced)* — CNCC's own wording |
| **Fund transfer / remittance** | **资金汇划** | Transfer of funds across institutions; a documented IBPS use case | *(sourced)* — CNCC's own wording |
| **Online signing / online contracting** | **在线签约** | A documented IBPS service, alongside cross-bank payment and cross-bank account information inquiry | *(sourced)* — CNCC's own wording |
| **Cross-bank account information inquiry** | **跨行账户信息查询** | A documented IBPS service | *(sourced)* — CNCC's own wording |
| **Without leaving home** | **足不出户** | CNCC's and the HKMA's phrase for the customer proposition | *(sourced)* — appears in both CNCC's Chinese text and the HKMA's English text |
| **Real-time forwarding, scheduled netting** | **实时转发、定时轧差** | The IBPS business-processing model from 2016 (previously **实时转发、实时轧差**) | *(sourced)* — CNCC 银清科技 page |
| **Direct participant** | **直接参与者** | A participant connecting directly, with its own settlement relationship | *(sourced)* — standard CNAPS-family terminology |
| **Indirect participant** | **间接参与者** | A participant reaching the system through a direct participant | *(sourced)* — standard CNAPS-family terminology |

**Note on the glossary's honesty.** Three entries are marked **uncertain** and deliberately left without a Chinese rendering: **FPS ID**, **Northbound** and **Southbound**. This is intentional — inventing a plausible Chinese term for a payment-scheme concept is exactly the kind of error that survives into a production specification. For the official Chinese rendering of any of these, use the HKMA's or HKICL's **Chinese-language** Payment Connect page (both publish 繁 and 简 versions; the HKICL page links to `/chi/` and `/schi/` variants of the same URL).

---

## 15. Cross-references and further reading

### 15.1 First-party sources (authoritative — read these)

| Source | URL | What it is |
| --- | --- | --- |
| **HKMA — Payment Connect** | <https://www.hkma.gov.hk/eng/smart-consumers/payment-connect/> | **The authoritative English source for the rules, limits, hours, proxies, use cases and institution lists.** Page revision 08 Sep 2026; lists as of 2 Sep 2026. **Re-check this first, always.** |
| **HKICL / FPS — Payment Connect** | <https://fps.hkicl.com.hk/eng/fps/consumers/payment_connect.php> | The operational companion: **Benefits**, the **How to use** flows (northbound 5+5 steps; southbound receive-side 6 points) and **Smart Tips**. Same rules table as the HKMA page |
| **HKMA press release — launch** | <https://www.hkma.gov.hk/eng/news-and-media/press-releases/2025/06/20250620-4/> | "The HKMA and the PBoC Launch Payment Connect", 20 Jun 2025: the **22 June 2025** launch, the **six institutions each** launch cohort, the attendees, the Eddie Yue quotation, and the **IBPS and FPS footnotes** quoted in §1–2 |
| **HKMA press release — Joint Announcement** | <https://www.hkma.gov.hk/eng/news-and-media/press-releases/2025/06/20250620-3/> | Joint Announcement of the PBoC and the HKMA, 20 Jun 2025 |
| **HKMA press release — MOU** | <https://www.hkma.gov.hk/eng/news-and-media/press-releases/2024/08/20240802-5/> | "PBOC and HKMA Sign MOU on Cross-Boundary Linkage of Payment Systems between the Mainland and Hong Kong", 02 Aug 2024 |
| **HKMA press release — launch annex** | <https://www.hkma.gov.hk/media/eng/doc/key-information/press-release/2025/20250620e4a1.pdf> | The launch institution annex referenced by the 20 Jun 2025 release. **Not retrieved by this guide** (§13.9) |
| **HKMA — Faster Payment System** | <https://www.hkma.gov.hk/eng/smart-consumers/faster-payment-system/> | The FPS consumer page, for the domestic system behind the corridor's Hong Kong leg |
| **HKMA — Beware of Fraudsters!** | <https://www.hkma.gov.hk/eng/smart-consumers/beware-of-fraudsters> | The HKMA's standing consumer fraud-warning page. Relevant to §9.4 |
| **CNCC — IBPS** | <https://www.cncc.cn/zfxt/201606/t20160615_416.html> | The operator's own IBPS page: **2010-08-30 go-live, nationwide 24 Jan 2011**, and the 实时传输及回应机制 description. **The primary basis for §2's dated fact table** |
| **CNCC (银清科技) — IBPS** | <https://tech.cncc.cn/ywjs/202212/t20221212_1182.html> | The operator's technical-affiliate page: the **2016 model change** to 实时转发、定时轧差, and the 足不出户 / cross-bank account management / fund transfer / fund consolidation description |
| **HKICL — FPS consumer FAQs** | <https://fps.hkicl.com.hk/eng/fps/consumers/frequently_asked_questions.php> | FPS consumer FAQs, linked from the Payment Connect page |

### 15.2 Institution product pages (representative sample, not a link farm)

The HKMA and HKICL pages carry a website link and hotline for **every** participating institution. That full directory is deliberately **not** reproduced here, because it duplicates a source that changes. A representative sample spanning institution type:

| Institution | Type represented | Product page |
| --- | --- | --- |
| Bank of China (Hong Kong) | Large incumbent, mobile + internet banking | <https://www.bochk.com/dam/more/paymentconnect/en.html> |
| Standard Chartered Bank (Hong Kong) | International bank | <https://www.sc.com/hk/bank-with-us/sc-pay/fps-ibps-payment-connect/> |
| ZA Bank | Virtual bank | <https://bank.za.group/en/payment-connect> |
| Mox Bank | Virtual bank | <https://mox.com/faqs/faq-transfer/?category=payment-connect> |
| WeChat Pay Hong Kong | SVF / e-wallet | <https://pay.wechat.com/en/payment-connect.shtml> |

**For the complete directory, with channels supported and service hotlines per institution, use the HKMA Payment Connect page** (§15.1) — the source of this guide's §6 snapshot.

### 15.3 Secondary and industry sources used, with their limits

| Source | Used for | Limit |
| --- | --- | --- |
| ClearingPost, *China's Domestic Clearing Stack: A Practitioner's Guide to CNAPS, NetsUnion, and the World's Largest Payment Market* (12 Mar 2026), <https://clearingpost.com/insights/china-domestic-clearing-stack-cnaps-netsunion-guide/> | IBPS/HVPS 2023 volume figures; the CNAPS system map; NPC/CCPC topology; the ISO 20022 direction | **Secondary synthesis**, not a first-party specification. Figures attributed, not asserted as PBoC-published |
| BIS CPSS *Red Book* country volume for China (`d105_cn.pdf`), as cited by the above | CNAPS architecture and system descriptions | Authoritative but dated; the primary document was not retrieved for this guide |
| PBoC payment system general information and quarterly payment system reports (`pbc.gov.cn`), as cited by the above | The ultimate source for volume statistics | **The right place to go for precise volumes.** Not retrieved directly for this guide |
| World Bank *Fast Payments* IBPS case study (URL cited by the above; **retrieval failed**) | Intended as a capacity/volume cross-check | **Not used** — the document could not be retrieved (§13.6) |
| Chinese-language encyclopaedia entries (e.g. Baidu Baike) | The "under RMB 50,000 per transaction" figure | **Secondary and Chinese-language only** — flagged, not confirmed |

### 15.4 Repo cross-references

| Path | Relationship to this guide |
| --- | --- |
| `banking/cnaps_guide.md` | **Companion, written concurrently.** Owns CNAPS as a whole and treats IBPS only at component level. **This guide owns IBPS depth and the cross-boundary linkage.** Not read by this guide |
| `banking/nucc_netsunion_guide.md` | **Companion, written concurrently.** Owns NetsUnion (网联); this guide positions IBPS against it only in §1 and §6. Not read by this guide |
| `banking/financial_fraud_detection_at_scale_guide.md` | **Referenced in §9.1, §9.4 and §9.6.** The detection, monitoring and analytics frameworks a real-time retail rail with no documentary proof depends on |
| `banking/enterprise_risk_management_guide.md` | **Referenced in §9.6.** The risk framework and operational-risk/control design context for the corridor's concentration and resilience exposure |
| `banking/financial_infrastructure_guide.md` | Broader payments and market-infrastructure context; useful for positioning the corridor among other clearing arrangements |
| `banking/chinese_bank_core_systems_guide.md` | Relevant when working the Mainland-side implementation of the corridor from a core-banking perspective |

> **Concurrency note.** `banking/cnaps_guide.md` and `banking/nucc_netsunion_guide.md` are being written **at the same time as this file**. They are cross-referenced **by path only**; this guide has not read them, does not depend on their contents, and their internal structure may differ from the references given here.

### 15.5 A minimal reading order, if you have limited time

1. **HKMA Payment Connect consumer page** — the rules, limits, hours, proxies and institution lists. Everything else is elaboration.
2. **HKICL Payment Connect page** — the flows, benefits and Smart Tips; this is where the operational reality lives.
3. **HKMA press release 20250620-4** — the launch, the cohort, and the two footnotes defining IBPS and FPS.
4. **CNCC's IBPS pages** — the operator's own account of what IBPS is and when it launched.
5. This guide's **§5 (rules), §6 (institutions), §7 (flows), §8 (checklist), §12 (claims audit), §13 (unverified)** — for the structured reading of the above.

---

## 16. Closing summary

IBPS is the Mainland's real-time retail payment rail: built by the China National Clearing Center under the guidance of the People's Bank of China, live in production from **30 August 2010**, extended nationwide on **24 January 2011**, running 7×24, resolving a payment and returning the result inside the customer's session. It is a domestic bank-channel system — not a card rail, not a third-party wallet rail, and not the final settlement layer, which is HVPS. Its 2016 shift from real-time netting to scheduled netting is the single most instructive technical fact about it, because it proves that customer-visible real-time and settlement-time real-time are deliberately different clocks.

Payment Connect is the linkage of that rail to Hong Kong's FPS: signed into existence by an MOU between Lu Lei of the PBoC and Howard Lee of the HKMA on **2 August 2024**, announced on 20 June 2025, and launched to the public on **22 June 2025** with six institutions from each side and a commitment to gradual expansion. As of the snapshot in this guide — HKMA page revision **08 September 2026**, lists as of **2 September 2026** — it connects **26 Hong Kong institutions and 18 Mainland institutions**, spans the full Hong Kong institutional spectrum from the note-issuing incumbents to four virtual banks and an e-wallet, and reaches the Mainland banking mainstream while pointedly excluding the big-tech wallets and the rural tier.

The rules are small and specific: northbound, an HKID-holder payer sends up to **HKD 10,000 a day** and **HKD 200,000 a year per participating institution**, 24x7, on a mobile number or an account number, with **no documentary proof**, to a personal account holder, receiving RMB. Southbound, a Mainland ID-card holder sends under the existing facilitative FX arrangement, subject to an annual quota equivalent to **USD 50,000**, inside a **16-hour daily window** from 07:00 to 23:00, using a mobile number, email address, FPS ID or account number, receiving HKD or RMB. In both directions the same-name and immediate-family restriction that used to define this corridor has been lifted, and documentary proof has been reduced to nil.

Those facts, taken together, are the whole design problem. A rail that is instant, unproven, low-value, cross-currency, cross-jurisdictional, unconstrained by relationship and effectively irreversible cannot be controlled at the point of documentary review, because there is no documentary review. It has to be controlled at the point of **identity, limit, proxy resolution, name verification, rate disclosure, purpose coding, and detection**. That is why the annual-limit counter is per customer **per institution** and must be a reconciled ledger rather than a display; why the partially masked payee name is a control rather than a nicety; why the receiving-side prerequisites on mobile-number linkage and English/Pinyin name records are release gates rather than hints; why the southbound window forces genuine cut-off design rather than an assumption; and why a bank's fraud, AML, privacy and resilience functions sit in the critical path of the product, not downstream of it.

What remains genuinely unknown is the engine room: the settlement, clearing and FX mechanics between the two estates; the message translation; the liquidity and settlement-bank arrangements; the return and recall semantics; and the liability allocation when a name does not match. Those are not gaps in research effort — they are not public. They have to be obtained from the operators and the counterparties, in writing, before a bank commits. A guide that filled those gaps with plausible invention would be more comfortable to read and far more dangerous to use.

For an institution, the decision reduces to a small number of questions with unglamorous answers. Is the retained customer relationship in the Hong Kong–Mainland cohort worth more than the fee revenue the corridor will not generate at this price point? Can you build a per-institution limit ledger that reconciles, disclose the scope of that limit honestly, and absorb the fact that the payment is final on credit? Can you staff a review queue sized for the false positives that weak beneficiary data will produce? If the answer to any of these is no, the correct move is to route the customer to a participant rather than to participate — the participant lists will keep growing, the limits will keep moving, and the capability will still be there in a year. What will not wait is the engineering, because the rules that make this corridor usable are also the rules that make it unforgiving: it is a rail designed to move small sums between people who need them, instantly, and it is designed to be the final word on the payment — the cross-boundary rail.
