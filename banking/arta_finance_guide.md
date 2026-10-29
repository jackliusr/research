# Arta Finance: The Licence Behind the Product - A Comprehensive Guide to a Digital Family Office That Became an AI Supplier to Wealth Managers

*A companion deep-dive in the Singapore financial-institutions series of the [jackliusr/research](https://github.com/jackliusr/research) repository, sitting alongside the other per-firm Singapore guides ([Mirae Asset Securities Singapore](mirae_asset_securities_singapore_guide.md), [ExodusPoint](exoduspoint_guide.md), [TD Securities](td_securities_software_systems_guide.md)) and the thematic ones this firm keeps colliding with ([Private Banking](private_banking_guide.md), [Wealth Management](wealth_management_guide.md), [Private Equity Singapore](private_equity_singapore_guide.md), [Singapore Private Markets](singapore_private_markets_guide.md), [MAS Regulations & Guidelines](mas_regulations_guidelines_guide.md)). This guide is about **one licensed legal person** - Arta Wealth Management Pte. Ltd., recorded by the Monetary Authority of Singapore under directory entry id 413895 - and about the gap between that entry and the marketing surface called "Arta Finance". It is deliberately not about the US entity, not about the founders' Google careers, and not about the AI. The AI is the part everyone asks about and the part that explains the least.*

**Verification convention used throughout: ✅ = verified in this research pass at a primary or named source; ⚠ = flagged (inferred, single-source, aggregator-reported, or structural inference); ❌ = disputed or not established. Unmarked text is structural or industry knowledge presented as such and should be read that way. The consolidated [Claims Audit is in section 14](#14-the-claims-audit) and the explicit gaps are in [section 15](#15-what-could-not-be-verified-the-glossary-and-the-cross-references).**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Singapore Financial Institutions - the licensing position, business model, AI claims and disclosed-vs-undisclosed boundary of Arta Wealth Management Pte. Ltd., a Singapore Capital Markets Services licensee trading as Arta Finance
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** October 2026
> **Companion guides:** [MAS Regulations & Guidelines Guide](mas_regulations_guidelines_guide.md), [Private Banking Guide](private_banking_guide.md), [Wealth Management Guide](wealth_management_guide.md), [Private Equity Singapore Guide](private_equity_singapore_guide.md), [Singapore Private Markets Guide](singapore_private_markets_guide.md), [Banks in Singapore Guide](banks_in_singapore_guide.md), [Mirae Asset Securities Singapore Guide](mirae_asset_securities_singapore_guide.md), [Agent Harness Engineering](../technology/ai_llm/agent_harness_engineering_guide.md), [Multi-Agent Banking](../technology/ai_llm/multi_agent_banking_guide.md), [AI Governance Framework](../technology/ai_llm/ai_governance_framework_guide.md), [LLM Agent Use Cases in Finance](../technology/ai_llm/llm_agent_use_cases_finance_guide.md)

---

**Table of Contents**

1. [The Overview, the Decoder, the Identity, the Thesis and the Boundary](#1-the-overview-the-decoder-the-identity-the-thesis-and-the-boundary)
2. [What a Family Office Is, and What "Digital" Changes](#2-what-a-family-office-is-and-what-digital-changes)
3. [The Business Model, As Far As It Is Disclosed](#3-the-business-model-as-far-as-it-is-disclosed)
4. [The Licensing, Which Is the Business](#4-the-licensing-which-is-the-business)
5. [The AI Agents, Attributed](#5-the-ai-agents-attributed)
6. [The Technology, As Far As It Is Disclosed](#6-the-technology-as-far-as-it-is-disclosed)
7. [The Alternatives Question](#7-the-alternatives-question)
8. [The Singapore and US Dual Structure](#8-the-singapore-and-us-dual-structure)
9. [The Honest Assessment](#9-the-honest-assessment)
10. [Worked Example: Cymbal Bank Looks at a Digital Family Office](#10-worked-example-cymbal-bank-looks-at-a-digital-family-office)
11. [The Failure Modes for This Class of Firm](#11-the-failure-modes-for-this-class-of-firm)
12. [The Competitive and Category Context](#12-the-competitive-and-category-context)
13. [The State of Practice](#13-the-state-of-practice)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified, the Glossary and the Cross-References](#15-what-could-not-be-verified-the-glossary-and-the-cross-references)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. The Overview, the Decoder, the Identity, the Thesis and the Boundary

### 1.1 The thesis in one line

**A digital family office is not a bank with an app; it is a licensing question wearing a product.**

Everything interesting about Arta Finance is downstream of what one Singapore legal person is permitted to do, for whom, with whose money, under which exemption. The Capital Markets Services licence defines the client population, shapes the product menu, dictates where custody has to sit, and decides which sentences the software is allowed to say. The artificial intelligence is the delivery mechanism. It is not the business, and a guide that starts with the AI will end up describing a demo rather than a firm.

### 1.2 The decoder: six words that get confused

The vocabulary below is used throughout this guide. Where a term has a Singapore regulatory meaning, that meaning is the one intended. The failure to keep these six apart is the single most common error in commentary about firms of this type.

| Term | What it actually means | What it is not |
|---|---|---|
| **Digital family office** | A marketing and product category: software that attempts to deliver some family-office functions (consolidated reporting, portfolio construction, access to alternatives, planning content) to clients who do not have a family office. There is no licence, statute or register entry for it. | Not a legal structure, not a fiduciary form, not a regulator-recognised category |
| **Family office proper** | A private organisation, usually a company or trust structure, that manages the affairs of one family (single-family office) or several (multi-family office), typically with dedicated staff, a CIO function, consolidated books, and coordination of tax, estate and legal advisers | Not a product, and not something a platform becomes by adding features |
| **Wealth-as-a-service** | A business-to-business model in which a licensed firm sells its platform, tooling or investment infrastructure to other institutions which then serve their own clients | Not a distribution channel for the seller's own advice; typically a technology or infrastructure supply relationship |
| **CMS licence** | Capital Markets Services licence under the Securities and Futures Act 2001, required to conduct the regulated activities listed in that Act - dealing in capital markets products, fund management, advising on corporate finance, REIT management, product financing, credit rating services, custodial services for securities | Not a banking licence, not a deposit-taking licence, not a financial adviser's licence, not a passport |
| **Accredited investor** | A statutory client classification under the SFA carrying prescribed wealth or income thresholds, which opens access to products and exemptions unavailable to retail investors, at the cost of certain protections | Not a sophistication test administered by the firm, and not a synonym for "wealthy" |
| **The agents** | Three conversational assistants the company announced in April 2025: an Investment Planner, a Product Specialist and a Research Analyst | Not licensed persons, not representatives under the SFA, and not the entity that owes the client a duty |

### 1.3 The identity, established at primary sources

The entity question is not a formality. The licensed person and the brand are different strings, and one of them can be checked against a regulator while the other cannot.

| Field | Value | Source and status |
|---|---|---|
| Licensed legal name | **ARTA WEALTH MANAGEMENT PTE. LTD.** | MAS Financial Institutions Directory, entry id 413895 ✅ |
| Directory key | **413895**; page footer "Last Updated : 22 Sep 2026" | MAS FID, retrieved 29 Oct 2026 ✅ |
| Domicile | "Incorporated in Singapore" | MAS FID ✅ |
| Licence type and status | **Capital Markets Services Licensee** | MAS FID ✅ |
| Regulated activity 1 | **Fund Management** | MAS FID ✅ |
| Regulated activity 2 | **Dealing In Capital Markets Products** -> Securities; Collective Investment Schemes | MAS FID ✅ |
| Exemption status | **Exempt Financial Adviser** - Advising on Investment Products -> CIS; Securities | MAS FID ✅ |
| Exemption status (cont.) | **Exempt Financial Adviser** - Issuing or Promulgating Analyses/Reports on Investment Products -> CIS; Securities | MAS FID ✅ |
| Key personnel | Chief Executive Officer -> **AMANDA ONG QIU HUI** | MAS FID ✅ |
| Address | 21 Collyer Quay #07-103, Singapore 049320 | MAS FID ✅ |
| Contact on register | artafinance.com; +65 69905831 | MAS FID ✅ |
| Brand in use | "Arta Finance" | Company website ✅ |
| Licence number as the company states it | 101441 | Company disclosure on its own contact and partner pages ⚠ (not a directory field) |
| Pre-rebrand name | **Arbo Works** | TechCrunch, 17 Nov 2021 ✅ |
| US regulated affiliate | Arta Finance Wealth Management LLC, SEC-registered investment adviser | Company disclosure block, 18 Jul 2024 ✅ (first party) |

Two observations follow immediately, and both are load-bearing for the rest of the guide. First, the register has no licence number field, no grant date field and no conditions field populated for this entry - so any document that cites "CMS licence 101441 granted on 18 July 2024" is stitching a company statement to a company announcement and calling the result a register extract. Second, the register records a CEO who is not the CEO quoted in every launch article about the firm, which is a reminder that the licensed entity and the brand's public face are governed by different considerations.

### 1.4 The boundary of this guide, declared rather than implied

This is **not** the private-banking guide. [Private Banking](private_banking_guide.md) covers the relationship-managed, balance-sheet-backed segment, its thresholds, its booking-centre logic and its Lombard lending mechanics. Arta has no balance sheet to lend from and no relationship managers in the private-bank sense, which is precisely why it cannot be read as a private bank with better software.

This is **not** the robo-advisory guide. [Wealth Management](wealth_management_guide.md) sets out the low-touch, public-markets, low-minimum model and its unit economics; Arta's public-market proposition overlaps with it, but the alternatives menu, the credit line and the institutional business do not, and the licence is a different one.

This is **not** one of the agent guides. The repository's agent material - [Agent Harness Engineering](../technology/ai_llm/agent_harness_engineering_guide.md) and its siblings - covers how such systems are built and governed generically. Section 5 points at that material rather than re-deriving it, because the firm publishes nothing about its architecture and a guide that invents one would be writing fiction with a straight face.

What remains is a per-firm profile of a licensed wealth manager with an unusual distribution strategy, an unusual client-acquisition story, and an unusually thin public disclosure surface.

### 1.5 A note on method and on the tools that failed

The MAS Financial Institutions Directory is not a documented search API. Its landing page at `mas.gov.sg/financial-institutions-directory` returned 404 during this pass; the working route is `eservices.mas.gov.sg/fid` with a `term` query parameter, and the entry was reached by URL. The MAS page describing the CMS licence regime itself was located and extracted successfully, but it carries no publication date, so it is cited as retrieved 29 October 2026 rather than dated.

Three sources could not be read in full and are used only for what a search-result description or a first screen disclosed: The Edge Singapore's report of the licence grant (paywall or bot block), the Business Times coverage of the EDBI investment (first paragraphs only before the paywall), and the apex domain `arta.finance`, which failed to resolve usefully. Where a fact rests on a paywalled source it is marked accordingly. Nothing in this guide is reconstructed from memory to fill one of those holes.

---

## 2. What a Family Office Is, and What "Digital" Changes

### 2.1 The institution being imitated

A family office proper is an organisational answer to a coordination problem. A family with substantial, multi-jurisdictional, multi-asset wealth needs somebody to hold the whole picture: what is owned, where it is booked, what it costs, what it is worth, what the tax exposure is, what happens on death or divorce or relocation, and which adviser is responsible for which part of it. The office exists so that the family does not have to be the integration layer itself.

The service list is fairly stable across firms:

- **Consolidated reporting** across accounts, custodians, entities and currencies, on a schedule the family chooses, with a single number that reconciles.
- **Investment policy and portfolio construction** - an investment policy statement, an asset allocation, manager selection and monitoring, and a rebalancing discipline.
- **The CFO and controller function** - books and records for family entities, cash-flow forecasting, liquidity planning, capital commitment tracking for private funds, and oversight of the entities' auditors.
- **Tax and estate coordination** - not usually the tax return itself, but the orchestration of external counsel and accountants so the structures, the investment activity and the reporting are consistent.
- **Administration and risk** - insurance review, custody and counterparty oversight, cyber and physical security for the family, and increasingly data hygiene.
- **Access** - to managers, to deals, to co-investment, to banking facilities that are priced for the size of the relationship rather than the size of one account.

### 2.2 The cost structure, described as components rather than as prices

A guide that prints a fee range it cannot source is a guide that has stopped being research. What can be said without inventing numbers is how the cost is built.

A family office's cost base is **predominantly fixed**. Salaries for a CIO, an investment analyst or two, a controller, a reporting specialist and an operations person; systems for portfolio accounting and reporting; audit, tax and legal fees; custody and administration; rent and insurance. Almost none of that scales with the size of the assets, and all of it is incurred before the first dollar of return. On top of the fixed base, an office typically pays **external manager fees** - which are the family's cost but not the office's revenue - and may charge its own clients, in the multi-family case, through a mix of retainer, assets-under-management fee, and project or entity-level charges for accounting and reporting work.

The firm in this guide does not publish a fee schedule. What it does publish is narrower and is treated in section 3.4: the existence of a performance-based pricing option, and a subscription price for its AI product. Those two data points are not a fee schedule and should never be presented as one.

### 2.3 The constraint the digital model actually attacks

Two constraints, and only two, are genuinely attacked by software in this model.

**The first is the minimum scale at which a family office's fixed cost is bearable.** Because the cost base is fixed and professional, there is a level of wealth below which a dedicated office is an absurdity - the office would consume a large share of the return it is meant to protect. The digital proposition is that the same functional list can be delivered against a marginal cost per client that is close to zero, which pushes the viable threshold down by orders of magnitude. That claim is coherent. Whether it holds depends on which functions are genuinely automatable and which are a person in a chair, and the firm's own behaviour - building an institutional business that sells to banks with relationship managers - is evidence that the founders concluded some of it is the latter.

**The second is the segmentation gap between the accredited investor and the ultra-high-net-worth client.** Private banks have a threshold below which a client is not economic for a relationship manager, and above which the full service exists. The gap between "too small for a private bank" and "not small at all in absolute terms" is the addressable market for every digital wealth firm, and it is why the accredited-investor classification is the commercial hinge of the whole model rather than a compliance detail. A client with a seven-figure portfolio is simultaneously too small for a private bank's economics and too large for a retail robo-adviser's product menu.

### 2.4 What "digital" does not change

Four things survive the technology untouched, and each of them is a place where digital wealth firms get into trouble.

- **The fiduciary question.** Whether an entity owes a client a duty of care, loyalty or best execution is a matter of the licence it holds, the contract it signs and the jurisdiction it operates in - not of how good its software is. A conversational interface does not create or discharge a fiduciary duty.
- **The suitability question.** Somebody has to be accountable for the proposition that this product is appropriate for this client with this objective and this horizon. That accountability attaches to a licensed entity and, in most regimes, to a named representative. An agent can draft the reasoning; it cannot hold the accountability.
- **The liquidity of the underlying assets.** Software changes how quickly a client can be told what they own. It does not change how quickly the asset can be sold. A private credit fund with a multi-year lock is a private credit fund with a multi-year lock whether the app is beautiful or not.
- **The custody chain.** Client assets sit somewhere, with somebody who is authorised to hold them. That is a separate regulated function in Singapore, and it is a separate balance-sheet and operational-risk question everywhere. A platform that is not a custodian is a platform whose client assets are safe to exactly the extent that its custodian is.

---

## 3. The Business Model, As Far As It Is Disclosed

### 3.1 The consumer platform, in the company's own terms

The 18 July 2024 launch release, issued by the company through PR Newswire APAC, is the fullest first-party description of the consumer business. Its own framing is "a digital family office for Accredited Investors", and its factual claims are specific enough to be checked against the licence:

- **Alternatives access.** "private equity, venture capital, private debt, and real estate... starting with funds from top-10 fund managers, like KKR and TPG", with members able to "start investments as low as S$10,000" (company, 18 Jul 2024).
- **Liquidity.** The platform "unlocks liquidity through lines of credit for eligible members" (company, 18 Jul 2024). The word "eligible" is doing a great deal of work and is examined in section 4.6.
- **Public markets.** It "harnesses AI to enable intelligent investing in public markets", including "direct indexing, with a personalization level that can be essential for investors looking to manage risk and heavy concentrations in employer stock" (company, 18 Jul 2024).
- **Custody.** "Arta partners with BNY Pershing, the world's largest custodian bank, to provide custody services" (company, 18 Jul 2024). The "world's largest" is a company characterisation.
- **Infrastructure.** "Arta Finance runs on the Google Cloud Platform, utilizes the highest grades of public key encryption, and employs biometric and hardware-based authentication" (company, 18 Jul 2024).
- **Pricing.** "Eligible Arta members can choose performance-based pricing" (company, 18 Jul 2024).
- **Corporate.** "dual-headquartered in Singapore and the U.S.", with a US business described as "a U.S. SEC-registered investment advisor" that had "successfully launched and expanded its presence in the U.S. market" (company, 18 Jul 2024).

Read as a set, these are the claims of a firm selling access and consolidation to a client who is too big for a retail app and too small for a private bank. Read against the register, they are also the claims of a firm whose Singapore licence supports fund management and dealing, sits its custody with a third party, and extends credit through an arrangement the register does not describe.

### 3.2 The institutional business, which is where the company now points

By the time of this research pass, the company's own `/arta-ai` route served an institutions page rather than a consumer product page. Its language is not the language of a family office.

- Headline: "arta for institutions - The AI transformation partner built for wealth management" (company, retrieved 29 Oct 2026).
- Body: "Institutions know they need to move to AI. The question is how. Arta brings AI infrastructure built on years of R&D investment in wealth management. Already in production, deployed alongside institutions like yours." (company, retrieved 29 Oct 2026).
- Social proof: "Trusted by leading institutions and industry partners", followed by a logo wall naming Bank of Singapore, Income (NTUC Income), Wio, Ethivo, HLB Private Banking (Hong Leong Bank), Capco and Google Cloud (company, retrieved 29 Oct 2026).
- Testimonial strip: "We've seen tech redefine finance before. AI is taking wealth management to a whole new level, and Arta is leading the way." - Ralph Hamers, described as ex-CEO of UBS and ING (company, retrieved 29 Oct 2026).
- A named endorsement framing: "In their own words - LG Lim on why Bank of Singapore chose to work with Arta" (company, retrieved 29 Oct 2026).
- Value propositions: "Profitably serve segments others can't", "Transform unit economics", "RM capacity, multiplied", "Every advisor, every conversation, starts informed" (company, retrieved 29 Oct 2026).

The value propositions are the interesting part, because they are addressed to a bank's economics rather than to an end client's needs. "Profitably serve segments others can't" is a direct statement that the buyer's problem is the segmentation gap described in section 2.3. "RM capacity, multiplied" is a statement that the product is a tool for relationship managers, not a replacement for them.

The logo wall is not a contract list. No deal value, no client count and no revenue share is disclosed for any partnership named on that page. A logo on a supplier's website establishes that a relationship is being publicised. It does not establish its legal form, its scale, whether it is in production or in pilot, or whether any money changes hands.

### 3.3 The awards, kept in their place

The institutions page carries an awards block, with the company's own disclaimer that awards are not indicative of future results: Best Technology Partner to Private Banks (Global Private Banking Innovation Awards 2026, presented Singapore, June 2026); Best AI Supplier to Private Banks (PWM Wealth Tech Awards 2026, April 2026); World's Best AI-Powered Wealth Manager (Global AI Innovation Awards, October 2025); and Forbes Fintech 50 2024 (published February 2024). The 2024 launch release separately cites a 2023 "Fintech Trailblazer" award from Nasdaq and a GGV Fintech Innovation Award (company, 18 Jul 2024).

An award establishes that a panel, at a point in time, thought well of a submission. It does not establish assets, clients, revenue, licence conditions, or that any named institution has bought anything. Section 13 returns to this.

### 3.4 The fee structure, only as far as stated

Two pricing facts are in the public domain, both from named sources:

1. **A performance-based pricing option** for eligible members (company, 18 Jul 2024). No rate, no hurdle, no high-water-mark mechanic, no scope of what it applies to is published.
2. **An AI subscription starting at US$20 per month**, with free access for "clients with over US$100,000 managed by Arta" (Fintech News Singapore, 3 Apr 2025). The article also reports that the company planned to extend the AI offering to partner banks as part of its B2B platform - which, read in 2026, is the strategy that the institutions page shows was pursued.

That is the whole of it. There is no published management fee, no platform fee, no alternatives carry, no custody fee, no minimum account size, and no statement of whether the US$100,000 threshold is measured per household, per account or per product. The US$100,000 figure is a subscription-waiver threshold, not an account minimum and not an asset figure, and conflating it with either would be a category error that this guide declines to make.

### 3.5 The positioning shift, dated at both ends

This is the single most informative fact about the company's trajectory, and it requires no interpretation beyond putting two of its own sentences side by side.

| Date | The company's own description of itself | Source |
|---|---|---|
| 18 Jul 2024 | "a digital family office for Accredited Investors" | Company release via PR Newswire APAC |
| 29 Oct 2026 (retrieved) | "The AI transformation partner built for wealth management", selling "AI infrastructure" to institutions | Company institutions page |

Both are the company's own words. Neither is a misquotation and neither is a critic's characterisation. The movement is from selling a service to end clients to selling infrastructure to the institutions that have end clients - from competing with the private bank to supplying it. A firm that makes that move is telling you something about which side of the business had the better unit economics, and section 3.2's value propositions say it explicitly.

The guide's reading, offered as reading: a consumer platform serving accredited investors needs to acquire those investors one at a time, at a cost, against competitors with branches and brands; a supplier to banks acquires institutions one at a time, at a much higher price per acquisition and with a much longer sales cycle. The second is a harder sale and a better business if you can make it. Whether this one has made it is not established by a logo wall.

### 3.6 The "not disclosed" list, stated once and carried forward

The following are not published anywhere consulted in this pass, and this guide asserts nothing about any of them: assets under management; number of clients or members; revenue, profitability, burn or runway; valuation, post-money or share price; headcount; the full fee schedule; licence conditions, regulatory capital or enforcement history; ownership percentages, cap table or share classes; technology vendors beyond Google Cloud and BNY Pershing; and the entity structure beyond the three US names in the company's own disclosure block.

Third-party company profiles do carry point-in-time asset figures and member counts. They cannot be dated or reconciled against anything the company has said, and this guide deliberately does not reproduce them as measurements. The correct statement is that the company publishes neither figure, and that a profile page is not a disclosure document.

---

## 4. The Licensing, Which Is the Business

This is the longest section in the guide because it is the only one where the evidence is primary, dated and checkable by the reader.

### 4.1 What the register says, field by field

The MAS Financial Institutions Directory entry for id 413895 was retrieved on 29 October 2026. The page's own footer reads "Last Updated : 22 Sep 2026". Both dates matter: the retrieval date is when this guide looked, and the page date is the last time the regulator touched the record.

The entry, reproduced as fields rather than paraphrased:

- **Name:** ARTA WEALTH MANAGEMENT PTE. LTD.
- **Domicile line:** "Incorporated in Singapore"
- **Licence Type/Status:** Capital Markets Services Licensee
  - Fund Management
  - Dealing In Capital Markets Products -> Securities; Collective Investment Schemes
- **Licence Type/Status:** Exempt Financial Adviser
  - Advising on Investment Products -> Collective Investment Schemes; Securities
  - Issuing or Promulgating Analyses/Reports on Investment Products -> Collective Investment Schemes; Securities
- **Key Personnel:** Chief Executive Officer -> AMANDA ONG QIU HUI
- **Contact:** http://www.artafinance.com; +65 69905831; 21 COLLYER QUAY #07-103 049320

Four things are present, and they are the four things a counterparty actually needs: who the legal person is, what it may do, who is accountable for it, and where it is. What is absent is treated in section 4.7 and it is not a trivial absence.

### 4.2 What a CMS licence is, cited to the regime

The regime can now be cited properly rather than gestured at. MAS's own page on the Capital Markets Services licence, retrieved 29 October 2026, states:

> "A company must hold a capital markets services (CMS) licence to conduct activities regulated under the Securities and Futures Act 2001 (SFA)."

The same page enumerates the regulated activities that require the licence:

- Dealing in capital markets products
- Advising on corporate finance
- Fund management
- Real estate investment trust management
- Product financing
- Providing credit rating services
- Providing custodial services for securities

It defines the product perimeter as well: "Capital markets products include securities, units in a collective investment scheme (CIS), over-the-counter (OTC) derivatives, exchange-traded derivatives and spot foreign exchange for the purposes of leveraged foreign exchange trading."

Two structural features of the regime are worth stating because they shape everything downstream. First, the licence is **activity-based**: a firm holds a licence for a list of activities, and the list is public. Second, individuals conducting regulated activities on behalf of a licensee must be appointed as **representatives** - so the licensed entity is not the only regulated person in the chain; the humans in front of clients are separately accountable.

The MAS page also confirms the enforcement-adjacent mechanics that make the directory reliable as a source: all entities holding a CMS licence are listed on the Financial Institutions Directory, and "all entities that cease to hold a licence will be removed from the Financial Institutions Directory". The licence remains valid until the licensee stops conducting all regulated activities and MAS cancels it, until MAS revokes it, or until it lapses under section 95 of the SFA. On admission, MAS states that it assesses fitness and propriety of the applicant, its shareholders and directors; track record and management expertise; ability to meet minimum financial requirements; internal risk management and compliance; and the business model and its risks - and expects up to six months to review a complete application.

### 4.3 What the four listed permissions actually buy

Read together, the register entry describes a firm that can do the following, and the list is wider than the phrase "digital family office" suggests:

**Fund management.** The firm can manage a portfolio of capital markets products on behalf of clients under a management mandate - the discretionary or advisory portfolio service that is the core of a wealth manager's licence. This is the permission that makes "we run your money" a lawful sentence.

**Dealing in capital markets products, for securities and CIS.** The firm can deal in securities and in units of collective investment schemes. This is the permission that lets the platform transact - execute client orders, place them with a broker or market, and handle the subscription and redemption mechanics of funds. The product scope is explicit: securities and CIS, not the full list of capital markets products.

**Advising on investment products, as an exempt financial adviser, for CIS and securities.** This is the permission that lets the firm tell a client what it thinks. It is scoped to the same two product classes.

**Issuing or promulgating analyses or reports on investment products, as an exempt financial adviser, for CIS and securities.** This is the permission that lets the firm publish research-like material - the "Research Analyst" function in product form. Note how specific this is: publishing analyses about investment products is itself a regulated activity under the Financial Advisers Act framework, and the firm holds the exemption for it rather than relying on a general freedom to write.

Note what is *not* on the list even though the firm's marketing implies it: OTC derivatives, exchange-traded derivatives, spot FX for leveraged FX trading, advising on corporate finance, REIT management, product financing, and providing custodial services for securities. Each of those is a separate regulated activity on MAS's own enumeration, and none appears on this entry.

### 4.4 What the licence does not permit - and the exemption trap

The negative space is where the analysis earns its keep. Six statements, each of which a careful reader should be able to defend.

**It is not a banking licence.** Nothing on the entry authorises the firm to carry on banking business in Singapore. It cannot take deposits from the public, and the prudential architecture that surrounds a bank - capital adequacy, liquidity ratios, deposit insurance, supervisory intervention powers - does not apply to it. A client's recourse against this firm is a contractual and conduct-based recourse, not the recourse a depositor has against a bank.

**It is not a deposit-taking licence in any form.** The MAS enumeration of CMS activities contains no deposit-taking permission, and the SFA route is not the Banking Act route. When a firm of this type describes cash management or a yield product, the question to ask is which entity holds the cash and under what permission - and the answer is usually a bank or a money market fund, not the platform.

**It is not a licensed financial adviser.** This is the sentence most often got wrong, including by the company's own shorthand. The register says **Exempt Financial Adviser**, which is a different legal status from holding a financial adviser's licence. The exemption is available to a CMS licensee that notifies MAS of commencement of business as an exempt financial adviser - MAS's page describes the Form 26 lodgement under the Financial Advisers Regulations - and the firm must still appoint representatives and comply with the conduct requirements of the Financial Advisers Act. So the firm is exempt from *holding* a licence, not exempt from *conduct regulation*.

**The exemption is activity-scoped, and the scope is narrow.** The register lists two advisory activities, each limited to CIS and securities. Advising on other investment products, or advising on anything outside the listed classes, is not covered. An exempt financial adviser's permissions are a list, not a general permission to give financial advice.

**It is not a custody licence.** "Providing custodial services for securities" is its own regulated activity under the SFA and it is not on this entry. The company's own disclosure resolves where custody sits: "Clearing and custody of all securities are provided by BNY Pershing" (company, 18 Jul 2024). The licence and the disclosure agree, which is a good sign for the coherence of the firm's regulatory design.

**It does not, by itself, authorise a lending business.** There is no moneylending permission on the entry, no banking permission, and - notably - the CMS activity called **product financing** is absent. Product financing is the SFA-regulated activity that covers financing a client's acquisition or holding of capital markets products, which is the closest regulated analogue to a securities-backed line of credit. Section 4.6 takes this apart.

### 4.5 The accredited-investor boundary, and why the product is shaped around it

The client population is not a marketing choice. It is the perimeter inside which the exemptions work.

The Singapore definition of an accredited investor is a statutory classification under the SFA with prescribed thresholds. Secondary summaries of the definition converge on three alternative limbs for an individual: income of at least S$300,000 in the preceding twelve months; net personal assets exceeding S$2 million, with the primary residence counted only up to a capped amount net of secured loans; or net financial assets exceeding S$1 million. These thresholds are **cross-referenced rather than independently verified in this pass** - the MAS explainer page consulted returned a 404 and the working definition was not read at the statute - and readers who need the operative text should go to the SFA itself and to [MAS Regulations & Guidelines](mas_regulations_guidelines_guide.md). The structure of the classification, which is what matters here, is not in doubt: it is a prescribed-threshold test, and it opens a product perimeter that retail investors do not get.

Why the whole product is shaped around it:

- **The alternatives menu is only distributable to this population.** Private funds and private-credit vehicles are not retail products in Singapore, and the exemptions that permit their distribution run through the accredited-investor classification.
- **The advisory exemptions are drawn on the same line.** The firm's exempt-financial-adviser permissions sit alongside a client base that is presumed able to evaluate risk, which is the bargain the classification embodies: wider product access in exchange for narrower protections.
- **The economics require it.** A client who clears the threshold has enough to make a platform relationship worthwhile and enough complexity - employer stock concentrations, multiple accounts, illiquid holdings - to need the consolidation the product sells.
- **The marketing follows from it.** "Digital family office for Accredited Investors" is not a slogan with a compliance footnote attached; it is a product description that names its own regulatory perimeter.

The cost of that perimeter is that the firm has, by construction, excluded the mass market. Any claim that a firm of this type "democratises" wealth management has to explain how a product restricted to millionaires is democratic, and the honest answer is that it democratises access *within* a wealthy population, not access to that population.

### 4.6 The narrower-than-it-looks analysis: fund management, dealing, and the credit-line edge

The permissions sound expansive. Here is the deflation, in four moves.

**First, the combination is narrower than the sum of its words.** Fund management plus dealing plus exempt advice on securities and CIS is a coherent wealth-management licence. It covers discretionary portfolios, execution, and telling clients what you think about securities and funds. It does not cover derivatives, structured products outside the CIS perimeter, corporate finance advice, or custody. A firm holding exactly this entry can run a managed portfolio business and cannot run an investment bank, a broker-dealer with a derivatives book, or a custodian.

**Second, custody is structurally elsewhere, which is a constraint on the product, not just a vendor choice.** Because the firm does not hold a custodial-services permission, client assets must sit with a custodian that does. BNY Pershing is that custodian per the company's disclosure. This is a healthy arrangement - segregation, an independent record, a second set of books - and it also means the firm's client-asset story is only as strong as a third party's controls, and that the firm cannot unilaterally change its custody model without changing its licence position or its contracts.

**Third, the lending edge is genuinely unresolved.** The platform "unlocks liquidity through lines of credit for eligible members" (company, 18 Jul 2024). A line of credit against a securities portfolio is, functionally, Lombard lending. The register shows no banking licence, no moneylending permission and no product-financing activity. There are at least four structures that could make the statement true without contradicting the register, and the public sources consulted do not distinguish between them:

1. A third-party bank or licensed lender extends the credit, with the platform acting as introducer or servicing front end, and the client's contract is with the lender.
2. An affiliate outside Singapore extends the credit from another jurisdiction, and the Singapore entity markets it.
3. The "line of credit" is in substance a margin facility provided by the broker-dealer or custodian within the custody arrangement, governed by a securities margin agreement rather than a loan.
4. The facility is structured so that the credit exposure is not the Singapore entity's at all - for example a subscription-financing or capital-call facility for the alternatives programme rather than a personal loan.

**This guide cannot determine, from public sources, which entity extends the credit, under which licence, on whose balance sheet, and with which disclosures to the client.** That is not a rhetorical hedge. It is the specific question a bank's counsel would ask first, because the answer determines who bears credit risk, who bears conduct risk under the moneylending and banking regimes, what happens on a margin call, and whether the client's recourse is to a regulated lender or to a platform. Any reader evaluating this firm as a counterparty, a partner or a competitor should treat the credit line as an open item until the facility agreement is seen.

**Fourth, the licence says nothing about size.** A CMS licence is not scaled to assets. The entry is identical in kind for a firm managing a modest book and a firm managing a large one, so the register tells you what the firm may do and nothing about whether it is doing it at scale, profitably, or at all. Sections 9 and 14 carry that distinction forward.

### 4.7 The three negative findings, stated as findings

These are not omissions in this guide's research. They are absences in the primary source, and they should be quoted as such.

**No licence number.** The directory does not print one. The number **101441** appears in the company's own disclosure on its contact and partner pages: "holds a Capital Markets Services license for fund management, dealing in securities and collective investment schemes (CMS License 101441), as well as being exempt from the requirement to hold a financial adviser's licence for specified activities". That sentence is a first-party statement. It corroborates the directory's activity list exactly, which raises confidence that the company is describing its own entry accurately - but the number itself is not a register field, and a due-diligence file that records "verified at MAS: 101441" is recording something MAS did not publish.

**No grant date.** The directory does not print one. "18 July 2024" is the date of the company's launch announcement (PR Newswire APAC, 18 Jul 2024) and the date reported by The Edge Singapore, whose article could not be extracted in this pass and whose content is known only from a search-result description stating the licence was received "on July 18". The Business Times, in its 11 September 2024 coverage, says the firm "in July this year... received its capital markets services licence from the Monetary Authority of Singapore" - a month, not a day. So the month is corroborated by a named outlet, the day rests on the company's own announcement and a paywalled report, and the register is silent.

**No conditions, no capital requirement, no enforcement history, no client list.** The directory publishes none of these for this entry. Nothing in this guide should be read as implying anything about any of them, in either direction. A clean entry with no visible enforcement is what a compliant firm looks like and also what a firm nobody has yet examined looks like; the register does not distinguish.

### 4.8 The entity-name finding, which is the thesis in miniature

The licensed person is **Arta Wealth Management Pte. Ltd.** "Arta Finance" is a brand. Those are different strings and the difference is not cosmetic:

- A statement that "Arta Finance holds a CMS licence" is imprecise shorthand. Arta Finance is not a legal person on any register consulted; Arta Wealth Management Pte. Ltd. is.
- A contract with "Arta Finance" is a contract with whichever entity is named in the signature block, and that entity may be the Singapore licensee, the US investment adviser, or a holding company.
- A complaint about "Arta Finance" has to be addressed to a legal person, and the correct addressee is determined by the contract, not by the website.
- The CEO on the register is Amanda Ong Qiu Hui. The CEO quoted in the launch material is Caesar Sengupta, co-founder. Both can be true - a group founder and a Singapore entity CEO are different roles - and the register is the authority on who is accountable to MAS for the Singapore entity.

This is the thesis in miniature. The product is called one thing, the licence is held by another, and the obligations attach to the second. Any analysis that stays at the level of the brand will be wrong about who owes what to whom.

---

## 5. The AI Agents, Attributed

### 5.1 What was announced, and by whom

The agents entered the public record on 3 April 2025, in Fintech News Singapore's report "Arta Finance Unveils Conversational AI Tool for Investors":

> "Arta AI includes three primary agents: an Investment Planner that helps build tailored portfolios, a Product Specialist to guide users through Arta's platform, and a Research Analyst to assist with investment ideas and security selection."

The same report gives the commercial wrapper: "Currently available to early adopters, Arta AI will be offered via monthly subscriptions starting at US$20", and "Clients with over US$100,000 managed by Arta will receive free access." It records that the interface accepts voice or text, with example questions "How is my portfolio performing?" and "What should my portfolio look like?", and that the company "plans to extend the AI offering to partner banks and financial institutions as part of its business-to-business platform".

The Asset covered the same launch under the headline "Arta Finance rolls out AI wealth management offerings", naming the same three agents and characterising the product as combining "financial expertise with advanced AI capabilities".

Three agents, then, each with a job description at the level of a press release:

| Agent | As described in coverage | Date and source |
|---|---|---|
| **Investment Planner** | "helps build tailored portfolios" | Fintech News Singapore, 3 Apr 2025 |
| **Product Specialist** | "guide users through Arta's platform" | Fintech News Singapore, 3 Apr 2025 |
| **Research Analyst** | "assist with investment ideas and security selection" | Fintech News Singapore, 3 Apr 2025 |

Note the register's echo: "Issuing or Promulgating Analyses/Reports on Investment Products" is an exempt-adviser activity the firm actually holds, which is the permission under which a Research Analyst function can lawfully produce research-like output. That alignment is not accidental; it is what a licensed product looks like when it is designed by people who read their own licence.

### 5.2 The separation table: said, verifiable, unknowable

The discipline this guide applies to every AI claim is to sort it into three columns and refuse to let anything migrate.

| What the company says | What is verifiable | What is not knowable from public sources |
|---|---|---|
| Three agents exist: Investment Planner, Product Specialist, Research Analyst | The announcement, its date, its named outlet, and the company's subsequent institutional positioning ✅ | Whether the three "agents" are three models, three prompts, three workflows, or three labels on one system |
| The Planner builds "tailored portfolios" | That the product markets portfolio construction, consistent with a fund-management licence ✅ | Whether a portfolio is generated, recommended, or approved by a human; what the optimisation objective is; what constraints bind |
| The Specialist guides users through the platform | That a navigation assistant is a plausible product for a platform with an alternatives menu and a credit facility ⚠ | Whether it can transact, quote, or commit the firm to anything |
| The Analyst assists with "investment ideas and security selection" | That the firm holds an exempt-adviser permission to issue analyses on securities and CIS ✅ | Whether output is reviewed by a licensed representative before a client sees it; what the evaluation harness is; what the hallucination controls are |
| Voice or text interaction | Stated in coverage ✅ | The speech stack, the model vendors, the data residency, the retention policy |
| Priced from US$20/month, free above US$100,000 managed | Stated in a named outlet, 3 Apr 2025 ✅ | Whether the threshold is per household or per account; what is included at each tier |
| To be extended to partner banks | Stated intent, 3 Apr 2025 ✅; the 2026 institutions page is consistent with it ✅ | Which institutions are live, on what terms, with what liability allocation |

What the firm does **not** disclose, and what this guide therefore does not assert: which models it uses and from which vendors; whether the agents can place or execute orders; the human-in-the-loop design; the evaluation methodology; hallucination controls; the audit trail; data residency; the allocation of liability between firm and client for an agent's output; and whether any agent output is reviewed by a licensed representative before it reaches a client.

**The guide cannot say how the agents work. This is a deliberate absence, not an oversight.** A wealth-management agent is a system that produces text which a client may act on, inside a regulated relationship. Describing its architecture would require inventing it, and an invented architecture would be worse than no architecture at all, because it would read as due diligence.

### 5.3 What an agent can and cannot be liable for

This is the part of the AI discussion that actually matters to a bank, and it has nothing to do with model capability.

**An agent can produce a recommendation.** That is a technical fact about software. Given a portfolio, an objective and a corpus, a well-built system will produce a coherent, well-formatted, plausible recommendation - and it will do so at a marginal cost near zero, which is the entire commercial case for the product.

**An agent cannot hold the suitability duty.** The duty to ensure that a recommendation is appropriate for a particular client attaches to a licensed entity and, in Singapore, to appointed representatives. It is discharged through a documented process: know-your-client, risk profiling, product due diligence, a recorded basis for the recommendation, and a complaints and remediation path. None of those is a property of a model. A firm can automate steps in that process. It cannot automate away the accountability, and it cannot delegate the duty to a vendor's software.

**An output that reads like personal advice engages the Financial Advisers Act regardless of who or what produced it.** This is the sentence to carry away from this section. The regulatory question is not "was a human involved" but "is this a recommendation about an investment product, made to a client, in the course of a business". If it is, the fair-dealing and conduct requirements apply - the same requirements that would apply to a relationship manager saying the same words in a branch. A conversational interface changes the evidence trail and the scale; it does not change the character of the act.

**Where the agents sit relative to the licence.** The Planner's "tailored portfolios" is portfolio construction, which sits under fund management ✅. The Analyst's research output sits under the exempt-adviser permission to issue analyses ✅. The Specialist's platform navigation is the one that looks least like a regulated activity and is the one most likely to drift into advice, because a navigation assistant answering "what should my portfolio look like?" stops being a navigation assistant. Coverage records that exact question as an example prompt, which is either a marketing illustration or a design statement, and the public sources do not say which.

### 5.4 Where to read about the generic problems instead

The repository has substantial material on how agents of this class are built and where they break. Rather than re-derive it, the relevant entry points, each with the reason it applies here:

- [Agent Harness Engineering](../technology/ai_llm/agent_harness_engineering_guide.md) - the scaffolding around the model: tool access, state, retries, guardrails. Applies because a financial agent's risk lives in its harness, not its weights, and none of Arta's harness is disclosed.
- [Agent Tool Routing](../technology/ai_llm/agent_tool_routing_guide.md) - how an agent chooses among tools and when it must not. Applies directly to whether a Planner can reach an order path or a credit facility.
- [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) - the rules that decide when the agent acts, escalates or refuses. This is where a suitability gate would live if one exists.
- [Multi-Agent Banking](../technology/ai_llm/multi_agent_banking_guide.md) - coordination among specialised agents in a banking context. The three-agent split is exactly the topology that guide examines.
- [AI Governance Framework](../technology/ai_llm/ai_governance_framework_guide.md) - accountability, model risk, documentation. Applies because a licensed firm needs a governance artefact for an advice-producing system, and none is published.
- [Agents That Fall Apart](../technology/ai_llm/agents_work_fall_apart_guide.md) - the observed failure patterns in production agents. The most useful prior for reading a vendor's claims sceptically.
- [LLM Agent Use Cases in Finance](../technology/ai_llm/llm_agent_use_cases_finance_guide.md) - the map of what finance agents are actually deployed for, which is a good check on whether three agents is a product or a demo.
- [AgentOps](../technology/ai_llm/agentops_guide.md) - running agents in production: monitoring, evaluation, incident response. The absence of any published incident or evaluation practice is the gap this guide flags.

---

## 6. The Technology, As Far As It Is Disclosed

### 6.1 The disclosed list, and it is short

Everything the firm has said publicly about its technology fits into six items, each attributable and each dated:

| Claim | Exact characterisation | Source and date |
|---|---|---|
| Cloud platform | "Arta Finance runs on the Google Cloud Platform" | Company, 18 Jul 2024 |
| Encryption | "utilizes the highest grades of public key encryption" | Company, 18 Jul 2024 |
| Authentication | "employs biometric and hardware-based authentication" | Company, 18 Jul 2024 |
| Custody | "Arta partners with BNY Pershing... to provide custody services" | Company, 18 Jul 2024 |
| Direct indexing | "direct indexing, with a personalization level that can be essential for investors looking to manage risk and heavy concentrations in employer stock" | Company, 18 Jul 2024 |
| Interface | Voice or text conversational interaction, with example prompts | Fintech News Singapore, 3 Apr 2025 |

That is the whole of the technical disclosure. It is enough to establish that the firm runs a cloud-native, API-mediated platform with an independent custodian and a conversational front end. It is not enough to establish anything about how the system makes decisions.

Two of the six deserve a note. "The highest grades of public key encryption" is not a specification - it is a qualitative claim that cannot be tested, and a serious security review would ask for key management architecture, rotation policy, HSM use, and the boundary between the firm's systems and the custodian's. "Biometric and hardware-based authentication" describes a login, not an authorisation model; the interesting question for a wealth platform is not how a client authenticates but what a compromised session can do.

### 6.2 The questions that cannot be answered, named as questions

For a private firm, the technology section of a guide is necessarily a list of questions. Each of the following is a question a bank's technology due-diligence team would ask, and none of them has a public answer.

- **The model layer.** Which models, from which vendors, self-hosted or API-mediated? Is there a router across vendors? Is any model fine-tuned on the firm's own data, and if so, whose data and under what consent? MAS's own technology-risk expectations for financial institutions turn on exactly this kind of inventory, and none of it is published.
- **The retrieval and data plane.** Where does client data live, in which region, under which retention schedule? What is retrieved into a prompt when a client asks "how is my portfolio performing" - a live position feed, a cached snapshot, or a document store? The answer determines both the accuracy of the answer and the blast radius of a breach.
- **The portfolio optimisation engine.** Is the "tailored portfolio" produced by a mean-variance optimiser, a rules engine, a model, or a human with a template? What are the constraints - concentration limits, tax-lot awareness for direct indexing, liquidity buckets? Whether the output is reproducible is the difference between an investment process and a chatbot.
- **The order path.** Can an agent place, amend or cancel an order? Is there a separate order-management system, and does a human approve? The register permits dealing; the question is whether the software is wired to that permission and with what interlocks.
- **Reconciliation and books-and-records.** With custody at a third party, the firm's records must reconcile to the custodian's. What is the reconciliation frequency, what is the break tolerance, and who signs off? This is unglamorous and it is where a platform either has an operating discipline or does not.
- **The evaluation harness.** How does the firm test whether the Research Analyst is right, or at least not harmful? Is there a golden set, a red-team suite, a hallucination rate, a human review sample? Nothing is published.
- **Incident history.** Has the platform had an outage, a data exposure, a misdirected order or a bad recommendation? A private firm has no obligation to publish, and the absence of a published incident is not evidence of absence.

### 6.3 Why this section is short and why that is correct

A guide that filled this section with architecture diagrams would be describing a firm it invented. The honest form of a technology section for a private company is a short list of what it said, followed by a longer list of what it has not. Anything else is fan fiction with a systems diagram.

What can be said with confidence is structural: a firm with a third-party custodian, a cloud platform and a conversational interface has chosen an architecture in which the firm owns the client relationship, the advice surface and the data model, and rents the balance sheet, the custody and the compute. That is a coherent architecture for a firm that wants to be capital-light. It is also an architecture in which the firm's most important dependencies are all outside its control, which is a governance question before it is a technology one.

---

## 7. The Alternatives Question

### 7.1 What access actually requires

The firm's most distinctive consumer claim is access to private markets from S$10,000. It is worth being precise about the machinery that sits behind any such claim, because the machinery is what determines whether the claim is a product or a slogan.

Gaining exposure to a private equity or private credit fund as a non-institutional investor normally requires all of the following:

- **A feeder vehicle.** The investor subscribes to a vehicle - typically a Singapore or Cayman feeder, sometimes a segregated portfolio or a sub-fund - which subscribes to the underlying fund. The feeder aggregates tickets, holds the investor register, and handles the administration the underlying manager will not do at small size.
- **A minimum subscription.** Set by the feeder's terms, not the underlying fund's, and therefore capable of being much lower than the manager's own institutional minimum.
- **Subscription documents and eligibility.** A subscription agreement, an investor questionnaire, AML and KYC, source-of-funds evidence, tax residency and CRS/FATCA declarations, and a determination that the investor is in the eligible class.
- **Transfer restrictions.** Interests in private funds are typically non-transferable without consent, and secondary transfers are subject to the fund's terms and to securities-law restrictions.
- **Capital call mechanics.** The investor commits a total and funds it in tranches on notice. The commitment is a liability, not an option; failing to fund has consequences.
- **Ongoing reporting.** Periodic capital account statements, valuations that are estimates rather than prices, and K-1 or equivalent tax reporting depending on structure.

None of that is exotic. All of it is operational work, and the cost of that work per investor is the reason the gatekeeping exists.

### 7.2 The liquidity mismatch, stated without softening

The underlying assets are illiquid. A private equity fund holds unlisted companies; a private credit fund holds loans that are not traded. The investor's money is committed for a multi-year term, often with an investment period followed by a harvest period, and there is no daily net asset value to redeem against. Valuations are periodic and model-based.

The consequence is that a platform offering this access is offering a client an asset that cannot be sold when the client wants to sell it, wrapped in an app that makes everything else instant. That mismatch is the central product risk of the category, and no interface design resolves it. Secondary sales exist but clear at a discount, subject to consent and to the fund's transfer provisions, and often with a lock-up before any secondary is permitted at all.

There is a specific and underappreciated form of this risk: a client who is told they can "unlock liquidity" through a line of credit against a portfolio that includes illiquid alternatives may believe the illiquidity has been solved. It has not been solved; it has been financed. The loan is callable on the lender's terms and the collateral cannot be sold quickly, which is a combination that behaves badly in exactly the scenario where the client needs it to behave well.

### 7.3 Why the gatekeeping exists

Four reasons, and only the first is about the client.

**Suitability and investor protection.** Private markets are risky, opaque, and priced infrequently. Regulators restrict retail distribution because the loss experience in an illiquid vehicle is asymmetric - the investor cannot exit, cannot reprice, and often cannot tell whether the manager is any good until the fund is nearly over.

**Statutory limits on retail distribution.** In Singapore, distribution of private funds runs through exemptions tied to the accredited-investor classification and to the sophistication of the offeree. The gate is a legal gate, not a marketing preference, which is why every firm in this category is, structurally, selling to the same population.

**The issuer's own investor-base limits.** US securities law, which governs most of the underlying funds, limits the number and type of investors a private fund may admit. A feeder with thousands of small holders can strain those limits or force structuring work. This is the constraint that most directly caps how "democratic" access can become.

**The administrative cost of many small tickets.** Every investor needs KYC, a subscription file, a capital account, statements, tax reporting and a redemption queue. At institutional ticket sizes that cost is trivial relative to the fee. At S$10,000 it is not, and the only way it works is aggregation plus heavy automation.

### 7.4 What has to be true for a S$10,000 ticket to work

The company states that members "can start investments as low as S$10,000" (company, 18 Jul 2024), with the initial menu drawn from "top-10 fund managers, like KKR and TPG". Presented as analysis of how such structures normally work - **the firm's own vehicle documentation was not reviewed in this pass** - the following must hold for that number to be real:

1. **A feeder exists and the firm controls or partners on it.** The underlying manager's minimum is met in aggregate across all feeder investors, not per investor. The S$10,000 is a feeder-level minimum, which is a statement about the firm's own vehicle design and says nothing about the manager's terms.
2. **The economics survive the small ticket.** A feeder charging a platform or administration fee on a S$10,000 position generates very little revenue per investor per year. The model only works at volume, or with a fee stack that includes the underlying manager's fees, or as a customer-acquisition cost carried against the rest of the relationship.
3. **Onboarding is genuinely automated.** Manual KYC at S$10,000 tickets destroys the unit economics. The firm's identity verification, accreditation checks and subscription flow must be straight-through, which in turn means the eligibility determination is software-mediated - a compliance design question, not just an efficiency one.
4. **The manager accepts a feeder with many small holders.** Not every manager does; some cap the number of underlying investors, some require minimum feeder sizes, some refuse retail-facing aggregation entirely.
5. **Liquidity expectations are managed at the point of sale.** The client must understand that the position cannot be sold, and the platform's disclosure must say so in a way that survives a complaint.

Each of those is an inference about structure, not a report of this firm's structure. The point of listing them is that "from S$10,000" is a claim about operational engineering, and a reader should treat it as such rather than as a price tag.

### 7.5 Where the depth is in this repository

The Singapore private-markets regime and the fund structures are treated in [Private Equity Singapore](private_equity_singapore_guide.md), whose section 7 sets out the MAS regime for private equity in condensed form, and in [Singapore Private Markets](singapore_private_markets_guide.md), which covers the vehicle landscape. A reader who wants the feeder mechanics, the exemption structure and the tax treatment should read those rather than expect them here; this section exists only to explain what the S$10,000 claim implies.

---

## 8. The Singapore and US Dual Structure

### 8.1 What "dual-headquartered" establishes

The company describes itself as "dual-headquartered in Singapore and the U.S." (company, 18 Jul 2024). The same release carries the disclosure block that makes the structure legible:

> "Arta Finance Wealth Management LLC ("Arta") is an investment adviser registered with the Securities and Exchange Commission ("SEC"). Arta's affiliates include Arta Finance Insurance LLC which offers insurance brokering services, and Arta Finance Club which offers tax and estate planning educational services. Clearing and custody of all securities are provided by BNY Pershing."

Three functions, three entities. That is the thesis in miniature again, and it is worth spelling out because it is the most instructive thing in the entire disclosure:

| Function | Entity | Regulatory character |
|---|---|---|
| Investment advice and portfolio management (US) | Arta Finance Wealth Management LLC | SEC-registered investment adviser |
| Insurance broking | Arta Finance Insurance LLC | Insurance broking affiliate |
| Tax and estate planning **educational services** | Arta Finance Club | Educational services - note the word "educational" |
| Fund management, dealing, exempt advice (Singapore) | Arta Wealth Management Pte. Ltd. | CMS licensee; exempt financial adviser |

A firm that wants to offer a family-office-shaped service discovers that the service decomposes into separately regulated activities, each needing its own permission, frequently in its own entity. Insurance broking is not investment advice. Tax and estate planning, if given as advice rather than as education, is a different regulated activity again - which is presumably why the disclosure says "educational services" rather than "advice". The structure is not bureaucratic clutter; it is the product's true shape.

### 8.2 The two regulator relationships are separate

The Singapore authorisation covers Singapore-regulated activity, for clients within the Singapore perimeter, under the SFA and the FAA. The US registration covers US activity under the Advisers Act and applicable state law. Neither regulator supervises the other's entity, neither licence is recognised by the other, and a compliance programme built for one does not satisfy the other.

This matters practically in three ways. Client money does not move between the entities freely. Marketing in one jurisdiction does not authorise marketing in the other. And a group-level statement like "Arta is regulated by MAS and the SEC" is true of the group and imprecise about any particular contract - the same shorthand problem as section 4.8, one level up.

### 8.3 Passporting, stated plainly

**A CMS licence is not a passport.** It authorises the holder to conduct the listed regulated activities in Singapore. It does not authorise the firm to serve retail clients in another jurisdiction, it does not create a branch right, and it does not travel. A Singapore-licensed firm serving a client in another country is relying on that country's rules, not on its Singapore licence.

What follows: the guide does not establish which entity contracts with a Singapore client versus a US client, which entity's terms of business govern, which entity holds the client's assets in each case, or how a client who moves jurisdictions is transitioned. Those are contract-level questions and the public sources do not answer them. A prospective client, or a bank contemplating a partnership, should ask for the contracting entity in writing before anything else.

### 8.4 What dual HQ does not mean

- **It does not mean two licences for one business.** It means two entities, each with its own permission, doing overlapping but distinct things.
- **It does not mean one balance sheet that can be read.** The group is private. There is no consolidated statement, no capital ratio, no supervisory return in the public domain. A counterparty assessing the group's financial strength has nothing to read.
- **It does not mean supervisory equivalence.** MAS and the SEC do not run a joint programme for this firm. Each supervises its own perimeter.
- **It does not mean the brand is the counterparty.** It means the brand sits above at least four legal persons, and the obligations live in the persons, not the brand.

---

## 9. The Honest Assessment

### 9.1 Three columns, and the discipline of keeping them apart

| Demonstrably is (verified) | Claims (attributed, dated) | Cannot be known (no public source) |
|---|---|---|
| A Singapore-incorporated CMS licensee, directory id 413895, holding fund management and dealing in capital markets products (securities, CIS) ✅ | That it operates a "digital family office for Accredited Investors" (18 Jul 2024) | Assets under management |
| An exempt financial adviser for advising on, and issuing analyses about, investment products limited to CIS and securities ✅ | That members can invest in alternatives from S$10,000 (18 Jul 2024) | Number of clients or members |
| CEO on the register: Amanda Ong Qiu Hui ✅ | That it provides lines of credit for eligible members (18 Jul 2024) | Revenue, profitability, burn, runway |
| Registered at 21 Collyer Quay #07-103, Singapore 049320 ✅ | That it runs on Google Cloud with public-key encryption and biometric/hardware authentication (18 Jul 2024) | Valuation or share price |
| A brand, "Arta Finance", distinct from the licensed name ✅ | That custody is provided by BNY Pershing (18 Jul 2024) | Headcount |
| A documented predecessor name, Arbo Works, reported by TechCrunch in Nov 2021 ✅ | Three AI agents with the described functions (3 Apr 2025) | The full fee schedule |
| A US affiliate, Arta Finance Wealth Management LLC, an SEC-registered investment adviser (company disclosure) ✅ | That it is "the AI transformation partner built for wealth management" (2026 institutions page) | Licence conditions, capital, enforcement |
| A 2021 founding year, attested by TechCrunch and the Business Times ✅ | That institutions named on its logo wall are partners (2026 institutions page) | Cap table, ownership, share classes |
| Institutional positioning with named awards, 2025-2026 ✅ | Award claims, as stated with the company's own disclaimer (2026) | The AI architecture, models, vendors, controls |

### 9.2 The funding figures, printed side by side and left unresolved

Four numbers circulate and they do not agree. The guide prints all of them, attributes each, and picks none.

| Figure | Source | Date | Character |
|---|---|---|---|
| US$90 million | TechCrunch (Manish Singh) | 2 Nov 2022 | Secondary, contemporaneous with the announcement |
| S$120 million (US$90 million) | Arta Finance release via PR Newswire APAC | 18 Jul 2024 | First party; Singapore-dollar figure expressly equated to US$90m |
| "more than S$120 million" | Fintech News Singapore | 18 Jul 2024 | Secondary, restating the release |
| "over USD120m" | AsiaFirst | Undated recut of the launch story | Secondary, currency restated |
| US$92 million | Forbes company profile | Undated profile page | Secondary aggregator-style figure |
| US$92 million total funding | Tracxn | Undated profile | Aggregator |

The figures conflict. This guide does not pick one, and the company has not published an audited total.

One observation, offered as an observation and not as a resolution: the first-party figure is **in Singapore dollars** and is **expressly equated in the same sentence** to US$90 million. On that reading, the "US$120 million" in circulation is most plausibly the Singapore-dollar number restated in US dollars by secondary coverage - a currency substitution rather than a new round. The US$90 million and the S$120 million are then the same fact in two currencies, consistent with the 2022 TechCrunch report. The US$92 million figures sit close enough to US$90 million to be a rounding or a partial inclusion, and far enough to be unexplained.

None of that is a finding. It is the shape a resolution would take if the company ever published a reconciled number, which it has not.

### 9.3 The EDBI investment

On 11 September 2024, the Business Times reported that EDBI - the strategic investment arm of the Singapore Economic Development Board - had made "an undisclosed strategic investment into digital wealth platform Arta Finance", with the announcement made by the company "on Wednesday (Sep 11)". The amount is undisclosed. A Preqin profile describes the same transaction as Arta Finance Inc. raising "venture funding from EDBI and Peak XV Capital, with participation from Coatue Management and Ribbit Capital" (Preqin, as consulted).

Two things follow. A sovereign-linked strategic investor taking a position is a meaningful signal about the firm's standing in Singapore's financial ecosystem, and it is not a licence, an endorsement of the product, or a disclosure of scale. And the entity named in the Preqin description is Arta Finance **Inc.**, which is a third name in the structure - neither the Singapore licensee nor the US adviser LLC - and a reminder that the group's entity map is not public.

### 9.4 The verdict on the thesis

The thesis holds, with one complication that should be stated rather than smoothed over.

It holds because every substantive constraint on this firm is a licensing constraint. The client population is the accredited-investor class because the exemptions are drawn there. The product menu stops at securities and CIS because that is what the permissions cover. Custody is at BNY Pershing because the firm does not hold a custodial-services permission. The Research Analyst agent can exist because the firm holds an exempt-adviser permission to issue analyses. The credit line is the one place where the product appears to outrun the register, and it is exactly there that the public record goes quiet. The AI explains none of this. The licence explains all of it.

The complication is that the firm's own strategy has moved toward selling the licence-adjacent capability - the advice surface, the tooling, the operating discipline - to institutions that already hold licences. In that business, the licensing question does not disappear; it migrates to the customer. The bank that buys the AI holds the obligation, and the supplier's licence becomes irrelevant to the end client. That is a coherent business and it is a different business from the one the 2024 release described, and it is the reason section 10's worked example ends where it does.

---

## 10. Worked Example: Cymbal Bank Looks at a Digital Family Office

*Cymbal Bank is entirely fictional. Nothing in this section describes a real institution, a real transaction, or a real client, and no institution named anywhere else in this guide appears here as a party.*

### 10.1 The segmentation question

Cymbal Bank's wealth division has a threshold below which a client does not get a relationship manager. Below that threshold the bank offers a self-directed platform and a call centre, which is to say it offers almost nothing, and the clients in that band - accumulated savings, an employer-stock concentration, maybe a small holding in a private fund inherited from a previous adviser - drift to whichever platform markets to them most aggressively.

The division's head of strategy puts the problem in the terms section 2.3 supplies. The band is not poor. It is simply **too expensive to serve with the current operating model and too valuable to lose**. A relationship manager carrying a book needs a certain revenue per client to be economic, and the band's clients cannot generate it at the bank's fee level and cost base. Every year the bank declines to serve them properly, it hands a cohort to a competitor at the exact moment the client's wealth crosses the threshold at which the bank would like to have them.

The question the division asks is therefore not "should we build an app" but "**can the service be delivered at a marginal cost that makes this band profitable, without degrading the advice to the point where it becomes a liability**". That framing is the correct one, and it is why the division's first call is to a firm that has already built the thing.

### 10.2 The build-versus-buy question

Cymbal Bank's technology function can build a conversational interface. It has done harder things. What it cannot build quickly is the rest of the wrapper, and the division's paper to the technology committee says so:

- A **licensed advice surface**. The bank already holds the licences. What it does not have is a documented, tested process for producing personalised recommendations at scale through software, with the suitability reasoning recorded per client.
- An **operating discipline** around an advice-producing system: who reviews output, what the escalation path is, what the evidence trail looks like when a client complains, how the model is changed and re-approved.
- A **product shelf** that includes alternatives at a ticket size the band can afford, which means feeder vehicles the bank does not have and would take years to construct.
- A **direct-indexing engine** with tax-lot awareness and concentration management, which is a specialist build.
- **Regulatory precedent** - evidence that a regulator has seen this shape of product before and not objected.

The paper's key sentence, and the one the committee argues about: **"We would not be buying a model. We would be buying a regulated wrapper and the operating discipline around it. The model is the cheap part."**

That sentence is correct and it is also the reason the deal becomes difficult, because the wrapper is not separable from the obligations. If the bank buys the wrapper, it buys the obligation with it - unless the contract is drawn so that it does not.

### 10.3 The decision the licensing view changed

The division's original proposal is a white-label: Cymbal Bank's brand, the supplier's platform, the supplier's advice engine producing recommendations that the bank's clients receive. Commercially clean. The bank pays a platform fee, the supplier runs the technology, the bank's clients get the service at the band's economics.

Counsel's review changes the deal, and the change is specific.

The proposal as drafted has the supplier's system producing the recommendation and the bank's brand presenting it. That means the entity that designs the advice is not the entity that holds the licence under which the advice is given, and the entity that holds the licence has no operational control over the system that produces the advice. On a client complaint, the bank would be defending a recommendation it did not design, produced by a system it cannot audit, under a suitability process it did not operate. The regulator's question - "who decided this was suitable for this client" - would have no clean answer.

**The decision: the deal is re-scoped from a white-labelled advice arrangement to a technology supply agreement in which Cymbal Bank retains the advisory obligation and the suitability duty.**

The specific clause that carries the decision is the one that defines the supplier's output. Instead of "Supplier shall provide personalised investment recommendations to Bank's clients", the agreement says, in substance: *"Supplier shall provide portfolio construction tooling, research content and conversational interface components. All recommendations delivered to Clients shall be generated, approved and issued by Bank through its licensed representatives, who shall retain sole responsibility for suitability determinations. Supplier makes no recommendation to any Client and provides no advice."*

Three supporting terms matter as much as the clause itself:

1. **A model-change notification right.** The bank gets notice of, and approval rights over, changes to the components that affect a recommendation. Without it, the bank is accountable for a system that can change under it.
2. **An audit right and an evidence obligation.** The bank must be able to reconstruct, for any client and any date, what the system produced, what the representative saw, and what was sent. If the supplier cannot supply that, the bank cannot discharge its duty.
3. **A liability allocation that matches the obligation.** The bank carries the conduct risk it now owns; the supplier carries the technology risk it now owns - and the indemnity is drafted against the actual division of labour rather than against the marketing description of the product.

What Cymbal Bank gives up is speed to market and a clean single-vendor story. What it keeps is the ability to answer the only question that matters: *who was responsible for this advice*. The licensing view did not make the deal impossible. It made the deal a different deal, and a worse-looking one on the term sheet, and the correct one.

### 10.4 What the example establishes

The example is fictional and the structure is not. A bank buying an AI advice product from a licensed supplier is buying, whether it says so or not, a set of regulated activities. The licence held by the supplier does not travel with the software. The obligation lands wherever the client relationship and the recommendation-issuing function sit, and the contract has to say so before the first client is onboarded rather than after the first complaint.

Which is the thesis, applied. The product looks like software. The thing being transacted is a permission. And the party that ends up holding the permission holds the risk, no matter whose logo is on the screen.

---

## 11. The Failure Modes for This Class of Firm

Five patterns. Each has a symptom that shows up first, a cause that explains it, and a guardrail that is a specific artefact rather than a sentiment.

### 11.1 A licence narrower than the product

**Symptom.** The marketing describes a capability - derivatives exposure, structured notes, custody, lending, tax advice - that does not appear on the firm's register entry. Clients discover the gap at the point of a transaction, or a regulator discovers it at the point of an examination.

**Cause.** Product teams design against the customer journey, and the licence is checked late. The register entry lists activities, not features, and the mapping between the two is not obvious to anyone who has not done it.

**Guardrail.** A **product-to-permission matrix** maintained as a controlled document: every client-facing capability mapped to a named regulated activity, a licence or exemption reference, and a named owner. Reviewed at every product change and at least annually. If a row has no permission reference, the capability does not ship. This artefact is the single most useful thing a firm of this type can build, and it is cheap.

### 11.2 An agent that gives advice the firm is not licensed to give

**Symptom.** A conversational assistant answers a question in a way that constitutes a personal recommendation about a product the firm cannot advise on - a derivative, an insurance product, a tax position - and the transcript is the evidence.

**Cause.** Three compounding causes. The prompt surface is open-ended, so the model is asked questions outside its permitted scope. The guardrail is a disclaimer rather than an intercept. And the evaluation set was built from intended use cases, not from adversarial ones, so nobody tested the questions that cross the line.

**Guardrail.** A **scope intercept at the tool and intent layer**, not in the prompt: the agent's permitted intents are enumerated, out-of-scope intents route to a human or a decline, and the routing is logged. Plus a **pre-release red-team suite** containing the boundary questions, run on every model or prompt change, with the pass criteria written down. Plus **transcript retention** long enough to reconstruct any recommendation. Cross-reference [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) and [AI Governance Framework](../technology/ai_llm/ai_governance_framework_guide.md) for the generic patterns.

### 11.3 An access story that outruns the liquidity

**Symptom.** Clients believe an illiquid position can be exited, or believe a credit line has made it liquid. The failure appears at the worst moment - a market drawdown, when the collateral has fallen and the facility is callable and the underlying fund is closed to redemptions.

**Cause.** The interface makes everything else instant, so the one thing that is not instant gets misread. And the credit line, marketed as liquidity, is a loan with its own covenants.

**Guardrail.** **Liquidity disclosure at the point of subscription**, in a form that states the lock-up, the redemption mechanics and the secondary-sale discount risk in the client's own transaction document rather than in a general risk page - plus a **collateral-eligibility rule** that excludes or haircuts illiquid positions in the lending calculation, plus a **stress test** of the combined position (illiquid asset plus callable loan) under a drawdown scenario, documented and reviewed by risk. The contractual term that matters is the one that says what happens on a margin call when the collateral cannot be sold.

### 11.4 A wealth-as-a-service deal that transfers the obligation somewhere nobody checked

**Symptom.** A partnership is signed as a technology supply agreement, but the supplier's system produces the recommendation and the institution's brand presents it. When a client complains, both parties discover that neither has a documented suitability process for the output.

**Cause.** The commercial negotiation runs ahead of the legal characterisation. The term sheet says "platform", the operating reality says "advice", and nobody writes down which one the contract means.

**Guardrail.** A **regulatory characterisation clause** in the agreement, agreed before signature, stating which party issues recommendations and which party holds the obligation - plus a **named accountable function** on each side, plus an **evidence obligation** (audit right, reconstruction capability, model-change notification). The register entry test is simple: if the deal requires the supplier to do something its register entry does not cover, the deal is mischaracterised. Section 10.3 is the worked version.

### 11.5 A digital family office that is a robo-adviser with a new vocabulary

**Symptom.** The product is a risk questionnaire, a model portfolio, a mobile app and a marketing site that says "family office". There is no consolidated reporting across entities, no CFO function, no tax or estate coordination, no access to anything a robo-adviser could not offer. Clients who needed a family office discover they bought a portfolio.

**Cause.** The words are cheaper than the functions. "Family office" signals a service level that the product does not deliver, and the gap is only visible to a client who has seen the real thing.

**Guardrail.** **A published scope statement** that enumerates what the service does and does not do, in the same language the marketing uses, reviewed against the register entry - plus **an internal definition of the service tier** with an explicit list of functions not provided. The commercial guardrail is candour: a firm that says "we do consolidated reporting, portfolio construction and alternatives access, and we do not do your tax return" is more defensible than one that implies the whole office.

### 11.6 The common thread

Every one of the five is a gap between a description and a permission. The description is written by product and marketing; the permission is written by a regulator. The failure modes are the places where the two documents were never compared, and the guardrails are all, in one form or another, the act of comparing them on a schedule.

---

## 12. The Competitive and Category Context

### 12.1 The rule for this section

No rankings. No market-share figures. No "leading", no "largest", no "fastest-growing". Those words require a measurement this guide does not have, and in a category this young the measurements that exist are vendor-sponsored. What follows is structural: what each adjacent category *is*, what it is built on, and where a firm of Arta's shape sits relative to it.

### 12.2 Private banks

**Structurally:** relationship-led, balance-sheet-backed, and priced on the relationship rather than the transaction. The bank's proposition rests on three things a platform cannot rent: a named relationship manager who knows the family, a balance sheet that can lend against the portfolio (Lombard credit), and a booking centre that offers jurisdiction, currency and confidentiality choices. The licence is a banking licence, which brings deposit-taking, prudential capital and a supervisory relationship - and with them, the ability to hold cash and lend.

The economics are high-touch and high-threshold. The segment below the threshold is served by digital channels that are, in most banks, a cost-avoidance measure rather than a product. This is the gap section 2.3 identified and the gap the subject firm addresses commercially.

### 12.3 Independent asset managers and multi-family offices

**Structurally:** fiduciary, relationship-led, and *without* a balance sheet. They manage or advise on client assets held at a third-party custodian, charge a fee on assets or a retainer, and compete on independence - no product manufacturing, no proprietary book, no conflict. The multi-family office adds the CFO and consolidated-reporting functions described in section 2.1.

This is the closest structural analogue to what a digital family office claims to be, and the comparison is clarifying: an independent asset manager's value is a person's judgement plus an absence of conflicts. Software can replicate the reporting and the portfolio construction; it cannot replicate the relationship, and it does not automatically replicate the independence, because a platform that also manufactures or distributes product has a conflict an independent adviser does not.

### 12.4 Robo-advisers

**Structurally:** low-touch, public-markets-only, low-minimum, and priced at a small basis-point fee on a model portfolio. The proposition is a risk questionnaire, a diversified allocation, automatic rebalancing and a low price. The regulatory shape is typically a licence to advise or manage, scoped to securities and funds, with a client base that may include retail.

The overlap with the subject firm is the public-markets half of the product and the digital delivery. The difference is threefold: the alternatives menu, which a robo-adviser generally does not have; the client classification, which here is accredited-only; and the institutional business, which robo-advisers do not run. A firm that has both a consumer platform and an AI-supply business is not a robo-adviser with better branding, but the failure mode in section 11.5 exists precisely because it can drift into being one.

### 12.5 Wealth platforms and B2B technology suppliers

**Structurally:** they sell the rail, not the advice. The customer is an institution - a bank, an insurer, an asset manager, a private bank's product team - and the product is software, data, or infrastructure. The supplier typically does not hold the client relationship, does not issue recommendations, and is paid a licence or subscription fee.

This is the category the subject firm has moved into, per its own 2026 institutions page. The structural attractions are obvious: institutional customers are fewer, larger, and cheaper to acquire per dollar of revenue than consumers; the revenue is contracted rather than transactional; and the supplier is not carrying the conduct risk of advice to end clients - the customer is. The structural hazard is equally obvious: the supplier's product is only valuable if it improves the customer's economics, which means the supplier has to understand a business it does not run.

### 12.6 The alternatives platforms

**Structurally:** distribution and aggregation. They build feeder vehicles, do the onboarding and administration, and give a manager access to a client base the manager could not economically reach. The revenue is a spread, a platform fee, or a share of the manager's fee. The regulatory shape is distribution plus, often, dealing - and the client base is the accredited or professional investor population, because that is the only population the exemptions permit.

This category is the one where the operational claim is hardest: the S$10,000 ticket in section 7.4 is a statement about feeder design and onboarding automation, and firms in this category live or die on whether that machinery works at volume.

### 12.7 Where Arta sits, and the conflict the double position creates

Placed by structure rather than by claim, the firm is **a licensed wealth manager that also sells the rail**. It holds fund management and dealing permissions, it serves accredited clients, it distributes alternatives through vehicles it does not describe, and it sells AI infrastructure to institutions that compete with it for the same client population.

That double position is the analytically interesting fact, and it creates a conflict-of-interest question that a pure platform does not have:

- **A pure technology supplier** sells to banks and does not compete with them for end clients. Its incentive is to make the customer successful.
- **A pure wealth manager** competes with banks and does not supply them. Its incentive is to win the client.
- **A firm doing both** supplies the tools of competition to its own competitors, and competes for the same accredited investors with the same tools. Every product decision then has two audiences: does this feature help my client, and does this feature help my customer's client - who may be the same person.

There are legitimate answers. A supplier can firewall its institutional engagements, license capability rather than client data, and treat the consumer business as a proving ground whose output is productised for institutions - which is roughly what "built on years of R&D investment in wealth management" claims. Whether any of that is in place here is not disclosed: no partnership economics, no exclusivity terms, no data-use terms, and no conflict-of-interest policy are public.

The question a bank's counsel would ask before signing is therefore not "is the product good" but "**if I deploy this, does my competitor get better at competing with me, and does my client data train my supplier's consumer product**". That question is answerable only in the contract, and the contract is not public.

### 12.8 What the category context does not tell you

It does not tell you who is winning. Every category above contains firms with more capital, longer track records and larger distribution than the subject of this guide, and it also contains firms that have failed. The structural placement tells you what kind of business this is and which constraints bind it. It tells you nothing about scale, and the guide does not pretend otherwise.

---

## 13. The State of Practice

### 13.1 What is disclosed, what is claimed, what is unknowable

**Disclosed**, at primary sources: the licensed entity's name, domicile, licence type, four regulated activities with their product scope, exempt-adviser status, CEO, address and contact details. A US affiliate's SEC registration and two further affiliate entities, in the company's own disclosure block. A predecessor name and a founding year, in named outlets. A launch date and a funding announcement.

**Claimed**, attributed and dated: the digital family office framing, the S$10,000 alternatives minimum, the lines of credit, the custody and cloud arrangements, the three agents and their subscription price, the institutional positioning and the logo wall, the awards.

**Unknowable** from public sources: scale of any kind, the fee schedule, the licence conditions, the credit-line structure, the AI architecture, the entity that contracts with a given client, and the legal form of any partnership.

### 13.2 The discipline a reader should carry away

Three habits, and they are the whole of the method:

1. **Separate the brand from the legal person before reading anything else.** The register is the only document in this guide that a regulator wrote, and it names a company the marketing never mentions.
2. **Date every claim and attribute it to whoever made it.** A company statement in 2024 and a company statement in 2026 can contradict each other without either being a lie, and the contradiction is the most informative thing in the file.
3. **Treat an absence as data.** No licence number, no grant date, no conditions, no assets, no client count. A firm that publishes nothing about scale is a firm whose scale you cannot assess, and the correct response is to say so rather than to reach for the nearest profile page.

### 13.3 The awards, and what an award does not establish

The awards are real in the sense that named bodies gave them on named dates: Best Technology Partner to Private Banks (Global Private Banking Innovation Awards 2026, Singapore, June 2026); Best AI Supplier to Private Banks (PWM Wealth Tech Awards 2026, April 2026); World's Best AI-Powered Wealth Manager (Global AI Innovation Awards, October 2025); Forbes Fintech 50 2024 (February 2024); and, per the 2024 release, a 2023 Nasdaq "Fintech Trailblazer" award and a GGV Fintech Innovation Award.

They are **claimed by the company and third-party-conferred, with caveats**. The caveats: the company publishes them on its own site, with its own disclaimer that awards are not indicative of future results; the judging criteria and the size and composition of the applicant pool are not published; several are category awards in which the category was defined by the organiser; and an award to a product is not an audit of a licence, a set of accounts, or a client outcome.

**What an award does not establish:** assets, clients, revenue, profitability, licence conditions, regulatory standing, the security of the platform, the accuracy of any agent output, or that any institution named on the same page has purchased anything. An award is a signal about a submission. It is not a measurement of a business.

---

## 14. The Claims Audit

**Mark convention: ✅ = verified in this research pass at a primary or named source; ⚠ = flagged (inferred, single-source, aggregator-reported, company claim awaiting corroboration); ❌ = disputed or not established.**

| # | Claim | Source | Date of source | Kind | Mark |
|---|---|---|---|---|---|
| 1 | Licensed name is ARTA WEALTH MANAGEMENT PTE. LTD. | MAS FID, entry 413895 | Directory page last updated 22 Sep 2026; retrieved 29 Oct 2026 | Register field | ✅ |
| 2 | Directory entry id is 413895 | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 3 | The entity is "Incorporated in Singapore" | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 4 | Licence type/status is Capital Markets Services Licensee | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 5 | Regulated activity: Fund Management | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 6 | Regulated activity: Dealing In Capital Markets Products | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 7 | Dealing scope limited to Securities and Collective Investment Schemes | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 8 | Exempt Financial Adviser: Advising on Investment Products (CIS; Securities) | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 9 | Exempt Financial Adviser: Issuing or Promulgating Analyses/Reports (CIS; Securities) | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 10 | CEO is Amanda Ong Qiu Hui | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 11 | Registered address 21 Collyer Quay #07-103, Singapore 049320 | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 12 | Register contact: artafinance.com, +65 69905831 | MAS FID | Retrieved 29 Oct 2026 | Register field | ✅ |
| 13 | "Arta Finance" is a brand, distinct from the licensed legal name | MAS FID plus company site | Retrieved 29 Oct 2026 | Comparison | ✅ |
| 14 | Licence number is 101441 | Company contact and partner pages | Retrieved 29 Oct 2026 | First-party statement, not a register field | ⚠ |
| 15 | Licence granted 18 July 2024 | Company release; The Edge Singapore (unextractable) | 18 Jul 2024 | Company announcement; day not on register | ⚠ |
| 16 | Licence received in July 2024 (month only) | The Business Times | 11 Sep 2024 | Named outlet, corroborates month | ✅ |
| 17 | Licence conditions, capital requirement, enforcement history | MAS FID (absent) | Retrieved 29 Oct 2026 | Not published | ❌ |
| 18 | Pre-rebrand name was Arbo Works | TechCrunch (Manish Singh) | 17 Nov 2021 | Named outlet | ✅ |
| 19 | Company founded in 2021 | TechCrunch; The Business Times | 17 Nov 2021; 11 Sep 2024 | Two named outlets | ✅ |
| 20 | Founding month was October 2021 | Secondary sources | Undated aggregation | Not verified at ACRA | ⚠ |
| 21 | Full founding team (five named individuals) | Tracxn profile | Undated | Aggregator, single source | ⚠ |
| 22 | Funding of US$90 million | TechCrunch | 2 Nov 2022 | Named outlet, contemporaneous | ✅ |
| 23 | Funding of S$120 million expressly equated to US$90 million | Company release via PR Newswire APAC | 18 Jul 2024 | First party | ✅ |
| 24 | "More than S$120 million" | Fintech News Singapore | 18 Jul 2024 | Secondary restatement | ⚠ |
| 25 | "Over USD120m" | AsiaFirst | Undated recut | Currency-restated secondary | ❌ |
| 26 | Total funding of US$92 million | Forbes company profile | Undated | Secondary | ⚠ |
| 27 | Total funding of US$92 million | Tracxn | Undated | Aggregator | ⚠ |
| 28 | EDBI made a strategic investment | The Business Times | 11 Sep 2024 | Named outlet | ✅ |
| 29 | Amount of the EDBI investment | The Business Times (undisclosed) | 11 Sep 2024 | Not disclosed | ❌ |
| 30 | Three agents: Investment Planner, Product Specialist, Research Analyst | Fintech News Singapore; The Asset | 3 Apr 2025 | Named outlets, company-sourced | ✅ |
| 31 | Agent functions as described (planner, navigation, research) | Fintech News Singapore | 3 Apr 2025 | Single named outlet | ⚠ |
| 32 | AI subscription from US$20 per month | Fintech News Singapore | 3 Apr 2025 | Single named outlet | ⚠ |
| 33 | Free AI access above US$100,000 managed | Fintech News Singapore | 3 Apr 2025 | Single named outlet | ⚠ |
| 34 | Alternatives available from S$10,000 | Company release | 18 Jul 2024 | Company claim, verbatim | ✅ |
| 35 | Custody and clearing by BNY Pershing | Company release | 18 Jul 2024 | Company claim; consistent with no custody permission on register | ✅ |
| 36 | Platform runs on Google Cloud Platform | Company release | 18 Jul 2024 | Company claim | ✅ |
| 37 | Public-key encryption; biometric and hardware authentication | Company release | 18 Jul 2024 | Company claim, unverifiable in substance | ⚠ |
| 38 | Direct indexing with personalisation for concentrated stock | Company release | 18 Jul 2024 | Company claim | ✅ |
| 39 | Performance-based pricing option for eligible members | Company release | 18 Jul 2024 | Company claim; no rate published | ⚠ |
| 40 | Bank of Singapore is a partner | Company institutions page, logo wall and testimonial framing | Retrieved 29 Oct 2026 | Company claim; no contract disclosed | ⚠ |
| 41 | Income, Wio, Ethivo, HLB Private Banking, Capco, Google Cloud are partners | Company institutions page, logo wall | Retrieved 29 Oct 2026 | Company claim; no contract disclosed | ⚠ |
| 42 | Quote attributed to Ralph Hamers, ex-CEO UBS and ING | Company institutions page | Retrieved 29 Oct 2026 | Company-published testimonial | ⚠ |
| 43 | Four awards, 2024-2026 | Company institutions page, with own disclaimer | Retrieved 29 Oct 2026 | Company claim; third-party bodies named | ⚠ |
| 44 | Dual-headquartered in Singapore and the US | Company release | 18 Jul 2024 | Company claim | ✅ |
| 45 | Arta Finance Wealth Management LLC is an SEC-registered investment adviser | Company disclosure block | 18 Jul 2024 | Company claim | ✅ |
| 46 | Affiliates: Arta Finance Insurance LLC; Arta Finance Club | Company disclosure block | 18 Jul 2024 | Company claim | ✅ |
| 47 | "Arta Finance Inc." raised from EDBI and Peak XV | Preqin profile | Undated | Aggregator; third entity name | ⚠ |
| 48 | Assets under management | No source | - | Not disclosed | ❌ |
| 49 | Number of clients or members | No company source; undated profile figures declined | - | Not disclosed | ❌ |
| 50 | Revenue, profitability, burn, runway | No source | - | Not disclosed | ❌ |
| 51 | Valuation, post-money, share price | No source | - | Not disclosed | ❌ |
| 52 | Headcount | No source | - | Not disclosed | ❌ |
| 53 | Full fee schedule | No source | - | Not disclosed | ❌ |
| 54 | Ownership, cap table, share classes | No source | - | Not disclosed | ❌ |
| 55 | Which entity extends the lines of credit, under which licence | No source | - | Not established | ❌ |
| 56 | Agent architecture, models, vendors, human review, audit trail | No source | - | Not established | ❌ |

Fifty-six rows. Where a row is marked ❌ for non-disclosure, the mark records that the claim cannot be established from public sources - not that the underlying fact is adverse. A firm with nothing published about its scale is not thereby small or large; it is unmeasured.

---

## 15. What Could Not Be Verified, the Glossary and the Cross-References

### 15.1 The gap list

1. **The licence grant date.** The directory does not print one. The company's announcement date (18 July 2024) and a named outlet's month-level corroboration (July 2024) are the best available; the day rests on the company and a paywalled report that could not be extracted in this pass.
2. **The licence number.** Not a register field. The number 101441 is the company's own disclosure, corroborated in substance by the exact match between the company's description of its activities and the directory's fields.
3. **Licence conditions, regulatory capital and enforcement history.** None published for this entry. Nothing is asserted in either direction.
4. **Assets under management.** Not published by the company. Third-party profiles carry undated figures that cannot be reconciled, and this guide declines to reproduce them.
5. **Client or member count.** Same position. The company publishes neither figure.
6. **Revenue, profitability, burn and runway.** Not published. A private company with no filed segment disclosure in the public domain.
7. **Valuation.** No round price, post-money or share price is published anywhere consulted.
8. **The full founding team.** The company says "a team of former Google executives" and "founded by former leaders at Google"; one aggregator lists five names. The aggregator is a single source and the names are flagged, not asserted.
9. **The month of incorporation.** The year 2021 is attested by two named outlets; "October 2021" appears in secondary sources and was not verified at ACRA.
10. **The entity and licence behind the lines of credit.** The product claim is documented; the facility structure, lender, balance sheet and disclosure are not. This is the largest single unanswered question in the file.
11. **The agent architecture.** Models, vendors, orchestration, whether orders can be placed, human review, evaluation method, hallucination controls, audit trail, data residency, liability allocation. None disclosed.
12. **Which entity contracts with which client.** The Singapore licensee, the US adviser LLC and Arta Finance Inc. all appear in different sources; no source states which one a given client contracts with, or on whose terms.
13. **The fee schedule.** Only two pricing data points exist: a performance-based option, and an AI subscription from US$20 per month with a waiver above US$100,000 managed.
14. **Partner-deal economics.** No deal value, revenue share, exclusivity or term is disclosed for any partnership, including those named on the logo wall.
15. **Whether the Bank of Singapore relationship is a licence, a pilot, or a technology supply agreement.** The company's page frames it as a choice to work with Arta and names an executive in a testimonial framing; the legal form is not published.

That is fifteen entries, and they are not evenly important. Numbers 10 and 12 are the ones a counterparty should resolve before signing anything, because they determine who bears credit risk and who owes the client a duty.

### 15.2 Glossary

**Digital family office.** A product category, not a legal form: software attempting to deliver family-office functions - consolidated reporting, portfolio construction, alternatives access, planning content - to clients who do not employ a family office. No licence or register entry corresponds to the term.

**Family office proper.** A private organisation managing the affairs of one family (single-family) or several (multi-family), typically with dedicated staff, an investment policy, consolidated books, and coordination of external tax, legal and audit advisers.

**CMS licence.** Capital Markets Services licence under the Securities and Futures Act 2001, required to conduct the regulated activities enumerated in that Act: dealing in capital markets products, advising on corporate finance, fund management, REIT management, product financing, credit rating services, and custodial services for securities. Activity-based, publicly listed on the MAS Financial Institutions Directory, and not a banking or deposit-taking licence.

**Accredited investor.** A statutory client classification under the SFA, meeting prescribed thresholds (income, net personal assets or net financial assets), which permits access to products and exemptions not available to retail investors in exchange for reduced protections. The threshold figures used in this guide are cross-referenced from secondary summaries, not read at the statute in this pass.

**Wealth-as-a-service.** A B2B model in which a licensed firm supplies platform, tooling or investment infrastructure to other institutions which then serve their own clients under their own licences and their own obligations.

**Alternatives.** Investments outside listed securities and cash - private equity, venture capital, private credit, real estate, real assets, hedge funds. Characterised by illiquidity, periodic valuation, and restricted distribution.

**Private credit.** Lending to companies outside the public bond market, typically by a fund that originates or participates in loans and holds them to maturity. Illiquid, valued periodically, and exposed to credit and recovery risk that is hard to observe from outside.

**Direct indexing.** Holding the individual constituents of an index rather than a fund that tracks it, so that the portfolio can be personalised - tax-loss harvesting, exclusion screens, and the management of a concentrated employer-stock position.

**Exempt financial adviser.** An entity exempt from holding a financial adviser's licence for specified activities, typically because it holds another licence (a CMS licensee notifying MAS of commencement of business as an exempt financial adviser). The exemption is activity-scoped, and the entity must still appoint representatives and comply with the Financial Advisers Act conduct requirements.

**Feeder vehicle.** A fund or company that pools investor subscriptions and subscribes to an underlying fund, allowing a manager's minimum to be met in aggregate and enabling smaller individual tickets at the cost of an extra layer of fees and administration.

### 15.3 Cross-references

Sibling guides in this directory:

- [Private Banking](private_banking_guide.md) - segments and thresholds, the relationship-manager model, booking centres, Lombard lending, and the bank-side view of the client this guide's subject competes for.
- [Wealth Management](wealth_management_guide.md) - business models, the client journey, platform technology, and the robo-advisory worked example that section 11.5 warns against resembling.
- [Private Equity Singapore](private_equity_singapore_guide.md) - section 7 is the condensed view of the MAS regime for private equity, which is the regime behind the alternatives menu in section 7.
- [Singapore Private Markets](singapore_private_markets_guide.md) - the vehicle landscape, feeder structures and the operational machinery section 7.4 describes.
- [MAS Regulations & Guidelines](mas_regulations_guidelines_guide.md) - the regulatory corpus this guide cites by regime rather than by section.
- [Banks in Singapore](banks_in_singapore_guide.md) - the licensed-entity landscape against which a CMS licensee should be placed.
- [Mirae Asset Securities Singapore](mirae_asset_securities_singapore_guide.md) - the genre model for this series, and a second CMS-licensee profile whose section 8 is the direct comparator for section 4 here.
- [ExodusPoint](exoduspoint_guide.md) - the per-firm genre applied to a fund manager, useful for how much a private firm does and does not disclose.

Agent and AI material:

- [Agent Harness Engineering](../technology/ai_llm/agent_harness_engineering_guide.md) - the scaffolding around a model, which is where a financial agent's risk actually lives.
- [Agent Tool Routing](../technology/ai_llm/agent_tool_routing_guide.md) - how an agent selects tools, and the control point for whether it can reach an order path.
- [Agent Decision Policy](../technology/ai_llm/agent_decision_policy_guide.md) - when an agent acts, escalates or refuses; where a suitability gate would sit.
- [Multi-Agent Banking](../technology/ai_llm/multi_agent_banking_guide.md) - coordination among specialised agents in banking, the topology a three-agent product implies.
- [AI Governance Framework](../technology/ai_llm/ai_governance_framework_guide.md) - accountability, model risk and documentation for an advice-producing system.
- [Agents That Fall Apart](../technology/ai_llm/agents_work_fall_apart_guide.md) - observed production failure patterns, the right prior for reading vendor claims.
- [LLM Agent Use Cases in Finance](../technology/ai_llm/llm_agent_use_cases_finance_guide.md) - what finance agents are actually deployed for, and whether three agents is a product or a demo.
- [AgentOps](../technology/ai_llm/agentops_guide.md) - monitoring, evaluation and incident response for agents in production.

Repository root: [jackliusr/research](../).

---

## 16. The Closing Summary

The licensed person is **Arta Wealth Management Pte. Ltd.**, directory entry 413895, holding a Capital Markets Services licence for fund management and dealing in capital markets products limited to securities and collective investment schemes, and exempt financial adviser status for advising on and publishing analyses about the same two product classes, with Amanda Ong Qiu Hui as chief executive and an address at 21 Collyer Quay. That is the whole of what a regulator publishes about this firm, and it is more informative than everything else combined.

The register prints no licence number, no grant date and no conditions. The number 101441 is the company's own; the date 18 July 2024 is the company's announcement, corroborated to the month by a named outlet. Records that present either as a register extract are stitching a press release to a directory page.

The client population is the accredited-investor class because the exemptions are drawn there, and the product is shaped around that boundary: alternatives that are not retail-distributable, a research function that maps onto an exempt-adviser permission to publish analyses, and a marketing line - "a digital family office for Accredited Investors" - that names its own regulatory perimeter. The licence also stops where the product seems to continue: no banking permission, no deposit-taking, no custodial-services permission, no product-financing activity, and no explanation in public sources of which entity extends the lines of credit that the launch material advertises.

The AI is the delivery mechanism. Three agents were announced in April 2025 - an Investment Planner, a Product Specialist and a Research Analyst - with a subscription from US$20 per month and free access above US$100,000 managed. Nothing about their architecture, their evaluation, their human review or their liability allocation is published, and this guide deliberately does not invent any of it. What can be said with confidence is structural: an agent can produce a recommendation, and the licensed entity remains the party that owes the suitability duty. An output that reads like personal advice engages the Financial Advisers Act whether a human or a model produced it.

The company's own description of itself moved between July 2024 and 2026, from a digital family office for accredited investors to "the AI transformation partner built for wealth management", selling infrastructure to institutions and naming banks, insurers and a cloud provider on a logo wall. Both descriptions are the company's own words. The move is from competing with the private bank to supplying it, and it is the most informative single fact about the firm's trajectory.

The funding record does not reconcile. US$90 million in 2022, S$120 million expressly equated to US$90 million in the 2024 release, "more than S$120 million" and "over USD120m" in secondary coverage, and US$92 million in two profile databases. The first-party figure is in Singapore dollars; the US$120 million in circulation is most plausibly that figure restated. The guide prints all of them and picks none, because the company has never published a reconciled total.

Scale is unmeasured. No assets, no client count, no revenue, no valuation, no headcount, no fee schedule, no cap table. Third-party profiles carry undated figures and this guide declines to reproduce them as measurements. An award establishes that a panel liked a submission; it establishes nothing about a business.

The structural placement is a licensed wealth manager that also sells the rail, and the double position creates a conflict a pure platform does not have: the firm supplies the tools of competition to institutions that compete with it for the same accredited investors. There are legitimate answers to that conflict. None of them is public.

For a bank, the lesson is in the worked example rather than the profile. Buying an AI advice product from a licensed supplier means buying a set of regulated activities, and the licence does not travel with the software. The obligation lands where the client relationship and the recommendation-issuing function sit, and the contract has to say which is which before the first client is onboarded. Cymbal Bank re-scoped its deal to a technology supply agreement with the advisory obligation retained, and that is the only version of the deal in which the question "who decided this was suitable" has an answer.

Read the register before the website, and the licence before the model. The product is the visible part and the permission is the load-bearing part, and in this firm - as in every firm of its kind - the permission is the thing that decides what the product is allowed to be.

a digital family office is not a bank with an app; it is a licensing question wearing a product.
