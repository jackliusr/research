# The Finance-Industry LLM-Agent Use-Case Landscape — What Is Deployed, Where, and with What Evidence

**The business-side catalog — the LLM-agent and agentic-AI use cases deployed across the four finance domains (banking, capital markets, insurance, payments), the named deployments behind each use case, the production-versus-pilot reality, and the evidence quality of every claim — the verified-at-primary, the press-reported, the vendor-marketed, and the disputed.**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Finance Domain / AI & LLM — the business-side use-case landscape for LLM agents in finance: which use cases are deployed, at which institutions, on which vendor rails, and with what quality of evidence. This guide is the **catalog-and-evidence companion** to the engineering deep-dive [Multi-Agent AI Systems for Banking](./multi_agent_banking_guide.md) — the engineering how-to (multi-agent architecture, orchestration patterns, evaluation pipelines, quality gates, deployment lifecycle, production standards) is **cross-referenced, not re-derived**. The general-industry survey [LLM Agent Use Cases](../llm_agent_use_cases.md) covers finance in its Section 5 as a fragment; this guide is the finance-only expansion that fragment points to, and the non-finance industries of that survey are **not re-derived** here.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Companion guides (ai_llm/, same folder):** [Multi-Agent AI Systems for Banking](./multi_agent_banking_guide.md) (the engineering deep-dive — MAS architecture, eval pipelines, deployment lifecycle, eval-gate conventions) · [LLM Agent Use Cases](../llm_agent_use_cases.md) (the all-industry survey — finance is its §5) · [AI Governance, Bias & Red-Teaming](ai_governance_bias_redteaming_guide.md) (the AI-compliance layer) · [LLM Agents Failures in Production](llm_agents_failures_production_guide.md) (why production agents fail — cited once in the adoption-reality section) · [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) (the eval tooling behind the eval gates)
> **Companion guides (banking/, prefix `../banking/`):** [Fircosoft — the AML/Sanctions Screening Engine](../banking/fircosoft_guide.md) (the sanctions-screening deep-dive — condensed cross-ref in the payments section)
> **Primary Sources:** this pass verified claims against primary or authoritative sources where they exist — vendor announcements (OpenAI, Google Cloud press corner, Klarna press), bank press releases and media releases (Morgan Stanley, HSBC, CommBank), regulator-visible materials, and reputable financial press (Reuters, the Financial Times). Facts verified at a primary source are marked ✅; facts carried from press coverage only are marked ⚠; facts the evidence contradicts are marked ❌. Nothing unverified is presented as verified, and the items this pass could not confirm are listed honestly in "What Could Not Be Verified".
> **Last Updated:** September 2026
> **How to read this guide:** Sections 1–2 set the map and the evidence conventions. Sections 3–6 are banking, 7–9 capital markets, 10 insurance, 11 payments. Section 12 grades the evidence and the adoption reality. Section 13 is the Cymbal Bank worked example — a finance-wide agent use-case portfolio review. Sections 14–15 close with the claims audit and the honest flags. The reader who wants the verdict in one sitting reads Sections 2, 12, and 13.

---

## Table of Contents

1. [The Overview — the Finance Companion, the Scope, the Evidence Conventions](#1-the-overview--the-finance-companion-the-scope-the-evidence-conventions)
   - 1.1 [The Companion Positioning](#11-the-companion-positioning)
   - 1.2 [The Scope — Four Domains, One Question](#12-the-scope--four-domains-one-question)
   - 1.3 [The Evidence Conventions — ✅, ⚠, ❌](#13-the-evidence-conventions---)
2. [The Landscape Map — the Four Domains, the General-Purpose vs Domain-Specific Split](#2-the-landscape-map--the-four-domains-the-general-purpose-vs-domain-specific-split)
   - 2.1 [The Use-Case Taxonomy Across the Four Domains](#21-the-use-case-taxonomy-across-the-four-domains)
   - 2.2 [The General-Purpose vs Domain-Specific Agent Split](#22-the-general-purpose-vs-domain-specific-agent-split)
   - 2.3 [The Maturity Gradient — Assistant, Copilot, Agent](#23-the-maturity-gradient--assistant-copilot-agent)
3. [Banking — Customer-Service & Support Agents](#3-banking--customer-service--support-agents)
   - 3.1 [The Advisor-Facing Assistant — Morgan Stanley](#31-the-advisor-facing-assistant--morgan-stanley)
   - 3.2 [The Retail Virtual Assistant — Erica, Fargo, Ceba, and the Asia GenAI Push](#32-the-retail-virtual-assistant--erica-fargo-ceba-and-the-asia-genai-push)
   - 3.3 [The Assistant Economics — the Contested FTE-Equivalence Claims](#33-the-assistant-economics--the-contested-fte-equivalence-claims)
4. [Banking — Fraud, AML & KYC Agents](#4-banking--fraud-aml--kyc-agents)
   - 4.1 [The Use-Case Evidence at the Named Banks](#41-the-use-case-evidence-at-the-named-banks)
   - 4.2 [The Vendor Rails — Nasdaq, Quantexa, Feedzai, Moody's, Hummingbird](#42-the-vendor-rails--nasdaq-quantexa-feedzai-moodys-hummingbird)
   - 4.3 [The Engineering Cross-Reference](#43-the-engineering-cross-reference)
5. [Banking — Wealth & Advisory Agents](#5-banking--wealth--advisory-agents)
   - 5.1 [AI @ Morgan Stanley Assistant and Debrief](#51-ai--morgan-stanley-assistant-and-debrief)
   - 5.2 [UBS, JPMorgan LLM Suite, and the Firmwide Copilots](#52-ubs-jpmorgan-llm-suite-and-the-firmwide-copilots)
   - 5.3 [The Robo-Advisor Contrast — the Pre-LLM Domain-Specific Agent](#53-the-robo-advisor-contrast--the-pre-llm-domain-specific-agent)
6. [Banking — Lending & Underwriting Agents](#6-banking--lending--underwriting-agents)
   - 6.1 [Upstart — the SEC-Filed Lending Platform](#61-upstart--the-sec-filed-lending-platform)
   - 6.2 [Zest AI and the Credit-Decisioning Stack](#62-zest-ai-and-the-credit-decisioning-stack)
   - 6.3 [JPMorgan COiN and the Contract-Intelligence Lineage](#63-jpmorgan-coin-and-the-contract-intelligence-lineage)
   - 6.4 [The Thin Claims — Mortgage and SME Agentic Underwriting](#64-the-thin-claims--mortgage-and-sme-agentic-underwriting)
7. [Capital Markets — Trading & Execution Agents](#7-capital-markets--trading--execution-agents)
   - 7.1 [The LOXM Lineage — JPMorgan, 2017](#71-the-loxm-lineage--jpmorgan-2017)
   - 7.2 [From LOXM to the Agentic Execution and Research Claims](#72-from-loxm-to-the-agentic-execution-and-research-claims)
   - 7.3 [The 2025 "AI Trader" Pilots — Production vs Pilot Discipline](#73-the-2025-ai-trader-pilots--production-vs-pilot-discipline)
8. [Capital Markets — Research & Analysis Agents](#8-capital-markets--research--analysis-agents)
   - 8.1 [The Earnings-Call and Document Summarisers](#81-the-earnings-call-and-document-summarisers)
   - 8.2 [BloombergGPT and the Terminal AI Layer](#82-bloomberggpt-and-the-terminal-ai-layer)
   - 8.3 [The Buy-Side Research Copilots](#83-the-buy-side-research-copilots)
9. [Capital Markets — Surveillance, Compliance & Post-Trade Agents](#9-capital-markets--surveillance-compliance--post-trade-agents)
   - 9.1 [The Communications-Surveillance AI](#91-the-communications-surveillance-ai)
   - 9.2 [The Trade-Surveillance Agent Claims](#92-the-trade-surveillance-agent-claims)
   - 9.3 [The Post-Trade and Operations Agents — STP, Reconciliation, Reference Data](#93-the-post-trade-and-operations-agents--stp-reconciliation-reference-data)
10. [Insurance — Claims, Underwriting & Policy-Service Agents](#10-insurance--claims-underwriting--policy-service-agents)
    - 10.1 [The Claims Agents — Lemonade, and the Marketing Flag](#101-the-claims-agents--lemonade-and-the-marketing-flag)
    - 10.2 [The Enterprise LLM Rollouts — AllianzGPT, Zurich, Ping An, OneConnect](#102-the-enterprise-llm-rollouts--allianzgpt-zurich-ping-an-oneconnect)
    - 10.3 [The Underwriting and Policy-Service Agents — Vendor vs Regulator-Visible](#103-the-underwriting-and-policy-service-agents--vendor-vs-regulator-visible)
11. [Payments — Authorization, Disputes & Sanctions-Screening Agents](#11-payments--authorization-disputes--sanctions-screening-agents)
    - 11.1 [Visa Advanced Authorization — the Pre-LLM Agentic Baseline](#111-visa-advanced-authorization--the-pre-llm-agentic-baseline)
    - 11.2 [Mastercard Decision Intelligence and the New LLM Dispute Agents](#112-mastercard-decision-intelligence-and-the-new-llm-dispute-agents)
    - 11.3 [The Sanctions-Screening Overlay — Condensed Cross-Ref](#113-the-sanctions-screening-overlay--condensed-cross-ref)
12. [The Evidence-Quality and Adoption-Reality Assessment](#12-the-evidence-quality-and-adoption-reality-assessment)
    - 12.1 [The Claimed-Deployments Evidence Table](#121-the-claimed-deployments-evidence-table)
    - 12.2 [The Adoption Reality — Production vs Pilot, on the Verified Surveys](#122-the-adoption-reality--production-vs-pilot-on-the-verified-surveys)
    - 12.3 [The Self-Reported and the Analyst-Forecast — the ⚠ Flags](#123-the-self-reported-and-the-analyst-forecast--the--flags)
13. [The Cymbal Bank Worked Example — the Agent Use-Case Portfolio Review](#13-the-cymbal-bank-worked-example--the-agent-use-case-portfolio-review)
    - 13.1 [The Portfolio-Review Method — Extending the Eval-Gate Conventions](#131-the-portfolio-review-method--extending-the-eval-gate-conventions)
    - 13.2 [The Candidate Portfolio Across the Four Domains](#132-the-candidate-portfolio-across-the-four-domains)
    - 13.3 [The Prioritisation Recommendation](#133-the-prioritisation-recommendation)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified, the Glossary, and the Closing](#15-what-could-not-be-verified-the-glossary-and-the-closing)
    - 15.1 [What Could Not Be Verified](#151-what-could-not-be-verified)
    - 15.2 [The Glossary](#152-the-glossary)
    - 15.3 [The Closing](#153-the-closing)

---

## 1. The Overview — the Finance Companion, the Scope, the Evidence Conventions

### 1.1 The Companion Positioning

This guide answers the **business-side question**: which LLM-agent and agentic-AI use cases are actually deployed across the finance industry, at which named institutions, and with what evidence? It is deliberately **not** the engineering guide. The sibling deep-dive [Multi-Agent AI Systems for Banking](./multi_agent_banking_guide.md) already carries the how-to — the multi-agent architecture for fraud triage, AML screening, compliance automation, and KYC; the orchestration patterns; the evaluation pipelines and quality gates; the deployment lifecycle and hypercare; the production standards. Wherever a use case in this guide needs that engineering context, this guide **cross-references** the sibling and moves on; it does not re-derive a single architecture diagram, eval metric, or rollout gate.

The same boundary applies to the other sibling guides:

- **The all-industry survey** [LLM Agent Use Cases](../llm_agent_use_cases.md) covers finance in roughly forty lines of its Section 5. This guide is the finance-only expansion that Section 5 points to. The other industries in that survey (software engineering, healthcare, legal, and the rest) are **out of scope and not re-derived** — the reader who wants them stays in the survey.
- **The sanctions-screening deep-dive** [Fircosoft — the AML/Sanctions Screening Engine](../banking/fircosoft_guide.md) carries the full screening estate, the vendor mechanics, and the name-matching engine. The payments section of this guide gives the sanctions-screening agent use case in **condensed cross-reference form only**.
- **The AI-compliance layer** [AI Governance, Bias & Red-Teaming](ai_governance_bias_redteaming_guide.md) and the failure catalogue [LLM Agents Failures in Production](llm_agents_failures_production_guide.md) are referenced where adoption reality demands them — one cross-reference line in Section 12.

The persona discipline is the same as the sibling guides: the only bank persona this guide may use is the fictional **Cymbal Bank**, and it appears only in the worked example of Section 13.

### 1.2 The Scope — Four Domains, One Question

Finance is not one industry, and "the AI agent" is not one product. This guide organises the landscape by the four domains whose regulatory, data, and customer contexts are genuinely different:

| The Domain | The Institutions | The Agent's Typical Object | The Dominant Regulation |
|---|---|---|---|
| **Banking** | Retail, commercial, and private banks | The customer conversation, the transaction, the account, the loan file | MAS Notice 626/658, GDPR/PDPA, the model-risk overlay |
| **Capital markets** | Investment banks, brokers, exchanges, buy-side firms | The order, the research document, the trade, the communication | Market-conduct rules, MAR, MiFID II, the surveillance obligations |
| **Insurance** | Insurers, reinsurers, MGAs | The policy, the claim, the risk submission | Insurance conduct rules, the actuarial and pricing standards |
| **Payments** | Networks, schemes, acquirers, issuers | The authorization decision, the dispute, the screened payment | Scheme rules, sanctions regimes (OFAC, UN, EU, UK, MAS) |

One question runs through all four domains: **is the deployment real, and how do we know?** Every named deployment in Sections 3–11 therefore carries one of three markers, defined next.

### 1.3 The Evidence Conventions — ✅, ⚠, ❌

| The Marker | The Meaning | The Source It Requires |
|---|---|---|
| ✅ | Verified in this pass at a primary source, or in reputable financial press (Reuters, the Financial Times, Bloomberg, VentureBeat-grade technology press) reporting the firm's own named statements | The vendor announcement, the bank's own press or media release, the regulator's release, the SEC filing, a direct extract of the primary page — or the firm's named executive quoted by the authoritative press |
| ⚠ | Press-reported, vendor-claimed, or otherwise unverified at a primary source — including the vendor's own estimate of its economics | Vendor marketing pages, single-source press items without the firm's named statement, self-reported survey figures, analyst forecasts — flagged because the primary could not be confirmed, or because the number is the claimant's own |
| ❌ | Disputed or contradicted by the evidence | A documented contradiction, a retraction, or a figure that independent parties have shown to be misleading |

A claim marked ⚠ is not a false claim — it is an **unverified-at-primary** claim, and the honest flag is the point. The guide's golden rule, inherited from the sibling guides: nothing unverified is presented as verified, and the items that could not be confirmed in this pass are collected in Section 15.1.

---

## 2. The Landscape Map — the Four Domains, the General-Purpose vs Domain-Specific Split

### 2.1 The Use-Case Taxonomy Across the Four Domains

Mapping the named deployments of Sections 3–11 back to a small number of use-case families shows how the same agent pattern repeats across domains with different regulatory weights:

| The Use-Case Family | Banking | Capital Markets | Insurance | Payments |
|---|---|---|---|---|
| **The customer- or client-facing assistant** | Support and service agents (Section 3) | — (client coverage copilots sit in Sections 5 and 8) | Policy-service agents (Section 10) | Cardholder support and dispute-intake agents (Section 11) |
| **The expert-facing knowledge copilot** | Advisor assistants, RM copilots (Sections 3, 5) | Research and analysis agents (Section 8) | Underwriting workbenches (Section 10) | Analyst copilots on disputes and screening (Section 11) |
| **The risk-and-crime detection engine** | Fraud, AML, KYC agents (Section 4) | Surveillance agents (Section 9) | Claims-fraud agents (Section 10) | Authorization and screening engines (Section 11) |
| **The decision-and-underwriting engine** | Credit decisioning (Section 6) | Execution algorithms (Section 7) | Underwriting and pricing (Section 10) | Dispute and chargeback decisioning (Section 11) |
| **The operations automator** | Back-office and ops automation (Sections 3, 4) | Post-trade STP, reconciliation, reference data (Section 9) | Claims processing (Section 10) | Settlement-adjacent exception handling (Section 11) |

Two patterns stand out even before the evidence is graded. First, the **highest-evidence deployments** cluster in the knowledge-copilot family (internal users, curated corpora, low blast radius) and the risk-detection family (quantified false-positive economics) — not in the fully autonomous decision family. Second, the **autonomy frontier** — the agent that acts end-to-end without a human in the loop — is thinnest exactly where the regulatory weight is heaviest: credit decisions, trade execution, and claim payments. Section 12 quantifies both patterns.

### 2.2 The General-Purpose vs Domain-Specific Agent Split

A second axis cuts across the domain taxonomy: how the agent is built and sourced.

- **The general-purpose agent** — a foundation model (or a platform on top of one) applied to a finance task with prompting, retrieval, and guardrails. The evidence base is large and public: Morgan Stanley's advisor assistant on OpenAI GPT-4 ✅ (Section 3), HSBC's Google-Cloud-built financial-crime and wealth tools ✅ (Section 4), the firmwide copilot suites of the global banks (Section 5). The economics favour the general-purpose route wherever the task is language-shaped and the corpus is the moat.
- **The domain-specific agent** — a purpose-built engine, often pre-LLM, that is *agentic in behaviour if not in name*: it senses (feeds on transactions, market data, or claims), decides against a learned or rules-based policy, and acts without a general-purpose language model at its core. Visa Advanced Authorization ✅, JPMorgan's LOXM ✅, and the robo-advisors ✅ are the canonical examples — each is a specialised decision system with a defined objective function, and each predates the LLM wave. The modern "agentic" label is frequently retrofitted onto this family (Section 7.2), and one job of this guide is to keep the two families distinct when reading vendor claims.
- **The hybrid** — domain-specific decisioning wrapped in an LLM interface, which is where most 2025–2026 production deployments actually sit: the LLM drafts and explains, the deterministic engine decides, the human approves.

The engineering consequences of this split — when to orchestrate a general-purpose agent versus when to wrap a deterministic core — are the subject of the sibling engineering guide's architecture sections and are **not re-derived** here.

### 2.3 The Maturity Gradient — Assistant, Copilot, Agent

Vendor and press language uses "agent" loosely. The deployments in this guide sit at three rungs of a maturity gradient, and the rung matters more than the label:

| The Rung | What It Does | Agency | Representative Deployments |
|---|---|---|---|
| **The assistant** | Answers questions from a curated corpus; reads but does not act | None — the human acts | Erica, Ceba, the early advisor chatbots |
| **The copilot** | Drafts, summarises, proposes, and explains inside the human's workflow | Tool calls with human approval | AI @ Morgan Stanley Assistant, LLM Suite copilots, comms-surveillance triage |
| **The agent** | Executes multi-step work end-to-end within a bounded scope, with escalation contracts | Bounded autonomy | The fraud-triage and AML-screening agents of the engineering guide, the post-trade exception handlers, the dispute agents |

The engineering guide's governed-agent principle — autonomy bounded by the scope, escalation, evidence, and evaluation contracts — is the definition this guide borrows for the third rung, and the worked example of Section 13 grades every Cymbal Bank candidate by which rung it can honestly occupy today.

---

## 3. Banking — Customer-Service & Support Agents

The customer-service assistant is the most visible and the most numerate LLM deployment in banking: every large retail bank now has one, and the claimed economics are large and contested in equal measure. This section grades the named deployments — Morgan Stanley's advisor assistant, Bank of America's Erica, Wells Fargo's Fargo, CommBank's Copilot family, and Klarna's assistant — and separates the verifiable usage facts from the vendor-reported FTE-equivalence figures.

### 3.1 The Advisor-Facing Assistant — Morgan Stanley

Morgan Stanley Wealth Management was the first large bank to put a GPT-4 assistant in front of its advisors, and the deployment is unusually well documented because both parties published:

- ✅ **The strategic announcement.** Morgan Stanley's own press release (March 2023) announced the internal-facing service built on OpenAI's GPT-4: Financial Advisors ask questions over Morgan Stanley's own research corpus — roughly 100,000 documents of intellectual capital — and receive answers generated **exclusively from internal Morgan Stanley content**, with links to the source documents. The release also states Morgan Stanley was "one of a handful of GPT-4 launch organizations" and "currently the only strategic client in wealth management receiving early access to OpenAI's new products" (source: Morgan Stanley press release, "Key Milestone in Innovation Journey with OpenAI," March 2023).
- ✅ **The model-vendor confirmation.** OpenAI's own case-study page confirms the production assistant (AI @ Morgan Stanley Assistant) runs on GPT-4 and describes the companion meeting-summary tool AI @ Morgan Stanley Debrief, built on Whisper and GPT-4 (source: openai.com, "Morgan Stanley uses AI evals to shape the future of financial services").
- ⚠ **The adoption headline.** The frequently cited "98% of advisor teams using the assistant" figure is widely reported (including by OpenAI-adjacent case-study pages) but this pass could not confirm it at a Morgan Stanley primary page, so it is flagged ⚠ rather than asserted.

The significance for the landscape: Morgan Stanley's assistant is an **internal expert copilot** — curated corpus, cited answers, no customer-facing autonomy — the pattern that Section 2.3 places on the second rung, and the pattern the rest of the industry copied.

### 3.2 The Retail Virtual Assistant — Erica, Fargo, Ceba, and the Asia GenAI Push

The retail assistants split into two generations: the **pre-LLM domain-specific assistant** (rules-and-ML driven, agentic in behaviour but not generative) and the **LLM-layered assistant** (generative models added on top of the established channel). The evidence base for both is bank-published and strong:

- ✅ **Bank of America Erica** — launched 2018, pre-dating the LLM wave. Bank of America's own newsroom (August 2025) reports Erica has assisted "nearly 50 million users since launch," surpassed **3 billion client interactions**, and averages more than **58 million interactions per month**; clients have spent 18.7 million hours conversing with Erica, and the bank reports 1.7 billion proactive insights delivered. The same release documents the enterprise spread: ask MERRILL and ask PRIVATE BANK serve ~23 million interactions per year; CashPro Chat is used by 65% of CashPro clients; Erica for Employees is used by over 90% of employees and has cut IT service-desk calls by 50% (source: Bank of America newsroom, August 20, 2025). ⚠ The "performs work equivalent to thousands of full-time roles" phrasing appears only in third-party summaries of the release, not in the release itself — treat as press-inferred.
- ✅ **Wells Fargo Fargo** — launched March 2023 as a Google-Dialogflow assistant on PaLM 2, per Wells Fargo CIO Chintan Mehta's own statements at a January 2024 industry event (source: VentureBeat, January 12, 2024: 20 million interactions handled since launch; ~2.7 interactions per session; bank evolving to multiple LLMs per task on its in-house "Tachyon" platform; 4,000 employees through Stanford's HAI programme). ✅ Wells Fargo's own newsroom release (2026) reports Fargo passed **1 billion customer interactions in under three years** since launch.
- ✅ **CommBank** — Commonwealth Bank of Australia's newsroom (March 2024) documents the deepened Microsoft partnership built around a "CommBank Copilot" for customer queries and employee-facing copilots. ✅ The August 2023 newsroom records the bank's generative-AI plans inside its in-house AI platform — including a genAI tool that lets customer-service staff search the bank's 4,500-document knowledge repository conversationally, and H2O.ai as its exclusive financial-services AI partner. The widely reported staff-assistant pilot named **Ask Ceba** could not be confirmed at a CommBank primary page in this pass — it is flagged ⚠ (and its claimed scale is recorded in Section 15.1).
- ✅ **DBS (bank-wide scale datapoint)** — DBS's own Annual Report 2024 chapter reports AI/ML initiatives delivering **over SGD 750 million of economic value in 2024** (more than double the prior year), with **over 1,500 models across more than 370 use cases**, including 1.2 billion personalised nudges sent to more than 13 million customers (source: dbs.com, Annual Report 2024, "Innovating impactful Solutions for our customers"). The seed-lead "300+ use cases" is the earlier vintage of this same verified count — the annual report is the primary.
- ⚠ **OCBC and UOB** — the Singapore peers' specific genAI assistant deployments (staff copilots and customer-facing pilots announced through 2024–2025) could not be confirmed at primary pages within this pass's search budget; they are listed in Section 15.1 rather than asserted.

### 3.3 The Assistant Economics — the Contested FTE-Equivalence Claims

The headline economics of customer-service AI are the most quoted and the least verifiable numbers in the whole landscape. The canonical case is Klarna:

- ⚠ **Klarna's "700 FTE-equivalent" claim.** Klarna's own press release (February 2024) and OpenAI's companion case study state that in its first month the AI assistant — built with OpenAI — handled **2.3 million conversations, two-thirds of Klarna's customer-service chats**, doing "the equivalent work of 700 full-time agents," with resolution times of 1–2 minutes versus 11 minutes for humans, a 25% drop in repeat inquiries, and a projected USD 40 million profit improvement for 2024. These are the **vendor's own figures** — published by the vendor on its own channels. The FTE-equivalence arithmetic is contested: Klarna's support headcount had already declined substantially in the year before the assistant launched, so "the work of 700 agents" is an output-equivalence measure, not a measured headcount reduction, and the CSAT and resolution comparisons are not a published controlled A/B. Verdict: ⚠ the usage volume is vendor-published; ❌ the "700 jobs replaced" reading that circulated in the press is disputed — the assistant's existence is not in doubt, the causal jobs claim is.
- ⚠ **The contested read-across.** Coverage of the Klarna claim (including Klarna's own subsequent statements that it continued hiring for other roles) shows why the finance industry's FTE-equivalence numbers must be read as marketing-grade estimates unless the bank publishes the methodology. The engineering guide's evaluation sections are the remedy: the same discipline that gates an agent on measured task metrics before production is the discipline that gates an assistant-economics claim on a published methodology before it is repeated.

The pattern generalises: the **usage numbers** in Section 3.2 (interactions, users, resolution volumes) are bank-published and verifiable; the **headcount-equivalence and cost-savings numbers** are press-magnified vendor estimates almost everywhere they appear, and this guide grades them ⚠ by default.

---

## 4. Banking — Fraud, AML & KYC Agents

Fraud and AML detection is where finance's agents touch money directly, and where the engineering guide earns its keep: the multi-agent architecture for fraud triage, AML screening, compliance automation, and KYC — the orchestration, the evaluation pipelines, the quality gates, and the deployment lifecycle — is the subject of [Multi-Agent AI Systems for Banking](./multi_agent_banking_guide.md), and this section deliberately stops at the boundary. What belongs here is the **use-case evidence**: which banks run what, on which rails, and how the claims grade.

### 4.1 The Use-Case Evidence at the Named Banks

- ✅ **HSBC — the Google Cloud AML-AI core.** HSBC adopted Google Cloud's AML AI as a primary AML transaction-monitoring system in its key markets. The June 2023 Google Cloud launch announcement (quoting Jennifer Calvery, HSBC's Group Head of Financial Crime Risk and Compliance) reports HSBC measured **2–4× more true-positive suspicious activity** while **reducing alert volumes by more than 60%**, and cutting the batch processing time for analysing billions of transactions across millions of accounts "from several weeks to a few days" (source: Google Cloud press release, June 21, 2023; HSBC's own "Views" article corroborates the ~980-million-transactions-per-month screening scale). This is the strongest quantified AML-AI deployment evidence in the public record — but note it is a **vendor-and-client joint claim**, not an audited third-party benchmark, so the numbers are ✅-verified-as-published rather than independently confirmed.
- ✅ **HSBC — the 2026 agentic expansion.** HSBC's own media release (June 17, 2026) announces the multi-year HSBC–Google Cloud partnership: more than 200 new AI use cases targeted over two years, each initiative expected to return over US$100 million in revenue gains or efficiency, with the financial-crime workstream deploying "generative AI and agentic AI" to intervene **twice as fast** when risk is detected across the ~1 billion transactions monitored monthly, using Gemini models and Google's Gemini Enterprise Agent Platform (source: hsbc.com media release, June 2026). The "expected" language is the bank's own target, not a measured result — ⚠ on the outcomes, ✅ on the announced programme.
- ✅ **Google Cloud AML AI — the multi-bank customer set.** The same 2023 launch release documents Bradesco (70 million customers, AI+decision-science approach) and Lunar (digital bank) as production users, and states the product was already "in production across several geographical regulatory jurisdictions." The customer names are primary; the qualitative praise is vendor-marketing-grade ⚠.
- ⚠ **Danske Bank — the Quantexa rail.** Vendor announcements (Quantexa, carried by industry press) state Danske Bank, after its multi-year AML overhaul, runs Quantexa's contextual-decision-intelligence platform for AML monitoring and investigations, following a 2018 pilot, including on market-trading activity. The deployment is vendor-announced — ⚠ until Danske publishes it — and it sits on the pre-LLM decision-intelligence rail (graph analytics + ML) rather than the generative-agent rail.
- ⚠ **JPMorgan's fraud-AI estate.** JPMorgan has long reported ML-based fraud and AML systems in production and has stated publicly (via press and investor materials) that hundreds of AI use cases are live across the firm, including in financial crime; the specific 2025–2026 vintage of those counts could not be pinned to a primary page in this pass, so the number is not asserted here — the existence of large-scale production ML for fraud at JPMorgan is, however, uncontroversial and consistent with the LLM Suite record of Section 5.

### 4.2 The Vendor Rails — Nasdaq, Quantexa, Feedzai, Moody's, Hummingbird

The financial-crime vendor market now sells "agentic AML" as a product category, and nearly all of it is **vendor-marketing-grade** until a named bank publishes results:

- ⚠ **Feedzai** — markets itself as the AI-fraud platform for payments and banking (rule+ML risk engines and, since 2024–2025, LLM-agent investigation copilots). Widely deployed as a vendor rail, but this pass verified no named-bank primary publication of agentic results — vendor-marketing until then.
- ⚠ **Nasdaq** (including the Verafin franchise for community banks and the Nasdaq Financial Crime portfolio) — markets AI-driven financial-crime management with agentic workflow claims; same grading: vendor-marketing until a named client publishes.
- ⚠ **Quantexa** — the Danske rail above; also markets "Decision Intelligence" agentic copilots for investigations. The Danske deployment is real (vendor-announced); the agentic-copilot results are marketing.
- ⚠ **Hummingbird** — markets "AI agents for financial crime compliance" (Research and Review agents, adjustable autonomy, audit trails) on hummingbird.co. The product pages are the only source — vendor-marketing. (The seed-lead pairing with Moody's could not be confirmed in this pass; see Section 15.1.)
- ⚠ **Moody's** — markets agentic and generative-AI compliance analytics across its risk portfolio; no named-deployment primary publication was confirmed in this pass.

The pattern to internalise: the **vendor rail** (who bought the platform) is often verifiable; the **agentic result claims** (how much better the agent made the programme) almost never are, because the numbers live in vendor case studies with the client's name attached only as a logo.

### 4.3 The Engineering Cross-Reference

Everything this section does not say — how the fraud-triage agent, the AML-screening agent, the compliance-automation agent, and the KYC agent are architected, orchestrated, evaluated, gated, and governed — is the entire subject of [Multi-Agent AI Systems for Banking](./multi_agent_banking_guide.md). The reader who wants to know *whether* banks run these agents stays here; the reader who wants to know *how to build and gate them* crosses to the engineering guide, whose eval-gate conventions the worked example of Section 13 reuses.

---

## 5. Banking — Wealth & Advisory Agents

Wealth management is the banking domain with the strongest LLM-agent evidence base, for a structural reason: the advisor-facing copilot has a curated corpus, an expert user, and no customer-facing autonomy — the lowest-risk high-value configuration in the industry. This section grades the advisor assistants (Morgan Stanley, UBS, JPMorgan), then contrasts them with the pre-LLM robo-advisors, which are the domain-specific-agent baseline the LLM wave is now absorbing.

### 5.1 AI @ Morgan Stanley Assistant and Debrief

- ✅ **AI @ Morgan Stanley Assistant** — the GPT-4 internal assistant for Financial Advisors, generating answers exclusively from Morgan Stanley's curated content with source links (source: Morgan Stanley press release, March 2023 — see Section 3.1 for the full record). OpenAI's case-study page adds the production detail: the assistant answers advisor questions over the firm's ~100,000-document research library, and OpenAI describes the eval-driven development ("Morgan Stanley uses AI evals to shape the future of financial services") — a rare public glimpse of the eval-gate discipline the engineering guide systematises.
- ✅ **AI @ Morgan Stanley Debrief** — OpenAI's page confirms Debrief, the advisor meeting-summary tool built on Whisper and GPT-4, as the second production assistant in the family. It is a summarisation copilot: the advisor records the client meeting, the tool produces the structured notes — again internal-facing, human-in-the-loop.
- ⚠ **The scale claims** — "98% of advisor teams on GPT-4" (Section 3.1) and related adoption percentages circulate widely but were not confirmable at a Morgan Stanley primary page in this pass.

The wealth-management lesson from Morgan Stanley: the advisor assistant's job is **knowledge retrieval with citation** over the firm's own corpus — not advice generation. The model drafts; the advisor remains the advice-giver; the firm's content remains the only knowledge source. That is the compliance-shaped design that regulators can inspect, and it is why this use case led the industry.

### 5.2 UBS, JPMorgan LLM Suite, and the Firmwide Copilots

- ✅ **UBS — the Azure OpenAI rollout.** UBS's public "Innovation and AI at UBS" hub (ubs.com) and the co-published Microsoft customer story document the bank's firmwide generative-AI build-out on Azure OpenAI Service: by December 2024 the service had reached the Personal & Corporate Banking, Wealth Management Switzerland, Global Wealth Management, and Group Operations divisions in the Switzerland, Hong Kong, and Singapore booking centres, rolled out to 30,000 employees within ten months (source: Microsoft customer story, ubs.com). Note the source is a co-marketing case study — the named-division detail is UBS's own, the framing is Microsoft's.
- ⚠ **UBS Red** — UBS's proprietary AI assistant "Red," giving employees intelligent access to UBS products, research, and CIO reports, is reported (finews, Swiss financial press, 2025) as rolled out to ~52,000 employees with general availability planned for the first half of 2026. Press-reported pending a UBS primary page — ⚠.
- ✅ **JPMorgan LLM Suite** — JPMorganChase's own technology blog (June 3, 2025) records LLM Suite as the firm's proprietary generative-AI platform, released summer 2024 to eligible employees firmwide, growing "from zero to 200,000 onboarded users within eight months" for idea generation and content drafting, and winning American Banker's 2025 Innovation of the Year. The same post states the direction of travel in the firm's own words: "combining generative AI with workflows to create AI agents that can carry out a series of actions to complete a goal" (source: jpmorganchase.com blog). The early phase — LLM Suite reaching the first tens of thousands of employees, including in asset and wealth management — was reported by Reuters in August 2024 (⚠ press for the phase detail; the 200,000-user record is the bank's own primary).
- ✅ **BofA's ask MERRILL and ask PRIVATE BANK** — bank-published usage: ~23 million interactions per year across the two advisor-facing tools (source: BofA newsroom, August 2025 — see Section 3.2).

The pattern across the big three wealth franchises is identical in shape: **a proprietary, firm-curated assistant in front of the advisor, firmwide scale in the hundreds of thousands of users, and the next release adding agentic actions under human approval** — the copilot-to-agent gradient of Section 2.3 playing out in production.

### 5.3 The Robo-Advisor Contrast — the Pre-LLM Domain-Specific Agent

The robo-advisors are the indispensable contrast case: a decade before the LLM wave, Betterment (launched 2010) and Wealthfront built **domain-specific agents** — systems that sense the client's portfolio and goals, decide an allocation against a modern-portfolio-theory objective function, and act (rebalance, tax-loss harvest) without a human per trade. They are agents in every sense except the language model. The two largest US standalone robo-advisors now report roughly US$65 billion (Betterment) and US$95 billion (Wealthfront) in client assets — ⚠ figures collected from company disclosures by third-party trackers, cited here only as scale context.

The landscape significance is the merger of the two families: the robo-advisor's deterministic decision core is being wrapped in LLM conversation (the hybrid of Section 2.2), while the banks' LLM copilots are reaching toward the robo-advisors' execution authority. The regulated outcome is visible in both directions: the LLM explains and proposes; the deterministic engine and the human hold the decision rights — exactly the architecture the engineering guide's governed-agent contracts enforce.

---

## 6. Banking — Lending & Underwriting Agents

Lending decisioning is the use case where agentic claims collide hardest with regulation: credit decisions are model-risk-managed, fair-lending-scrutinised, and explainability-mandated almost everywhere on earth. The evidence base is accordingly lopsided — strong at the **specialist AI lender** (Upstart), vendor-flavoured at the **decisioning-software layer** (Zest AI), press-level for the **bank-internal lineage** (JPMorgan COiN), and thin-to-nonexistent for the fully agentic mortgage or SME underwriter.

### 6.1 Upstart — the SEC-Filed AI Lending Platform

- ✅ **The production scale.** Upstart's own investor-relations releases report the platform's core metric: the percentage of loans **fully automated** — originated end-to-end without a human underwriter touching the file. For Q2 2026 Upstart reported 91% of its facilitated loans fully automated, on revenue of US$365 million (up 42% year over year) and net income of US$16.5 million (source: Upstart Holdings Q2 2026 earnings release, ir.upstart.com, August 4, 2026; the metric's definition is published on Upstart's own "By the Numbers" page).
- ✅ **The regulatory record.** Upstart is a public company (NASDAQ: UPST) whose SEC filings describe an AI lending model that evaluates thousands of variables and whose bank partners originate the loans on their own charters; the platform's automated-approval economics and its fair-lending analyses (including adverse-impact studies) are disclosed in its filings and earnings materials. The company has also published third-party-model-validation results with a major bank partner (⚠ the specific partner-study figures circulate in press summaries; the existence of the studies is company-disclosed).
- ⚠ **The generalisation limits.** Upstart's "fully automated" stat measures its own personal-loan platform flow, not the banking industry's; read across to mainstream bank credit lines only as evidence that **large-scale automated consumer credit decisioning is commercially viable and regulatorily survivable** — which is precisely why the incumbents buy rather than build.

### 6.2 Zest AI and the Credit-Decisioning Stack

- ⚠ **Zest AI** — the vendor markets AI-automated underwriting to banks and credit unions (auto, card, personal, home-equity), claiming more inclusive approvals and fraud-aware decisioning. Vendor-published figures include 22 million lending decisions made on its technology in 2023 versus 16.3 million in 2022, and studies claiming approval increases across protected classes (source: Zest AI press releases, 2024). The deployment breadth is plausible and the figures are the vendor's own — ⚠ vendor-marketing-grade until a named lender publishes results.
- ⚠ **The "agentic credit decisioning" claims** — several decisioning vendors now describe LLM agents that gather income evidence, read bank statements, and draft credit memos. This pass verified no named-bank production publication of an LLM agent holding credit-decision authority; every such claim found was vendor-marketing. The regulatory reality — model risk management and fair-lending rules apply to whatever model makes the call — is why the honest label for this category in 2026 is **copilot-assisted underwriting**, not agentic underwriting.

### 6.3 JPMorgan COiN and the Contract-Intelligence Lineage

- ⚠ **COiN (Contract Intelligence), launched 2017** — JPMorgan's document-review system for commercial credit agreements, reported at launch to read ~12,000 agreements in seconds against a manual workload estimated at 360,000 lawyer-and-loan-officer hours per year. The deployment is one of the most-cited bank-AI cases in history, but its primary record is press-carried (JPMorgan statements to the Financial Times and technology press in 2017; the bank has not maintained a public primary page for COiN), so this guide grades the existence ✅-press and the 360,000-hour figure ⚠-press.
- ✅ **The lineage into the LLM era.** COiN was not an LLM agent — it was a supervised document-extraction system on a narrow document class, the domain-specific-agent pattern of Section 2.2. Its significance here is as the **provenance line** for the modern contract-intelligence copilots: the same commercial-credit document workflow is now served at JPMorgan by the LLM-era stack (LLM Suite's document work plus the firm's AI-platform tooling, Section 5.2), and the vendor market for loan-document agents (COiN-class extraction wrapped in LLM interfaces) is crowded with ⚠ vendor claims and almost no bank-published numbers.

### 6.4 The Thin Claims — Mortgage and SME Agentic Underwriting

The categories where this guide found **no verifiable production deployment** and says so plainly:

- ⚠ **Fully agentic mortgage origination** — vendors market LLM agents that assemble the mortgage file, verify income, and run the credit decision; no named lender was verified as running a no-human mortgage decision in production. Marketing claims of "autonomous mortgage underwriting" are graded ⚠ and treated as aspirational until a lender's regulatory filing or primary announcement says otherwise.
- ⚠ **SME and commercial agentic underwriting** — similar vendor posture (agents that draft credit memos, pull bureau and bank data, propose structures); the credible 2025–2026 deployments are human-approval copilots. The engineering guide's HITL conventions describe exactly this ladder, and the worked example of Section 13 ranks the SME-memo copilot as a Cymbal Bank candidate precisely because the evidence base says the copilot rung is reachable and the agent rung is not.

---

## 7. Capital Markets — Trading & Execution Agents

The trading floor is where the industry's language is most precise — and where vendor prose is most dangerous — because "AI trader" can mean anything from a rules-based smart-order router to a reinforcement-learning execution engine to a chatbot that answers questions about risk. This section traces the one verified lineage (JPMorgan's LOXM), then applies production-versus-pilot discipline to everything newer.

### 7.1 The LOXM Lineage — JPMorgan, 2017

- ✅ **The 2017 record.** JPMorgan's LOXM is the canonical pre-LLM execution agent: the bank told the Financial Times (2017) that LOXM, built on deep reinforcement learning trained on billions of historical and simulated trades, had run a successful trial in its European equities business in the first quarter of 2017, executing client orders at high speed and best price while minimising market impact, with a global rollout planned by the end of that year. Business Insider's contemporaneous reporting ("JPMorgan Takes AI Use to the Next Level," 2017) corroborates that JPMorgan described LOXM as the first major bank to apply AI to real-time trade execution. The record is press-carried (the FT exclusive plus corroborating technology press) — graded ✅-press for existence and intent; the detailed performance claims (the "15% improvement" figure that circulates in secondary sources) are ⚠ and are not asserted here.
- ✅ **What LOXM actually was.** A domain-specific agent in the Section 2.2 sense: it sensed market state, decided execution strategy against a learned policy, and acted in the market — no LLM anywhere in the loop. The "agent" label is accurate; the "AI trader" label that the press attached to it is not, because the trader remained the human who owned the client order.

### 7.2 From LOXM to the Agentic Execution and Research Claims

The LOXM lineage matters because modern agentic-execution marketing leans on it. The honest map of what exists in 2026:

- ✅ **The LLM-era execution stack is copilot-first.** JPMorgan's own LLM Suite record (Section 5.2) describes the firm's direction as "combining generative AI with workflows to create AI agents that can carry out a series of actions to complete a goal" — inside a governed, human-approval environment, not as autonomous order flow. The e-Trading survey record (Section 12) shows where the sell side's own traders expect AI to bite: not "the machine trades alone," but analytics, execution advice, and workflow assistance.
- ⚠ **The "autonomous trading agent" vendor category.** Multiple vendors now market LLM agents that monitor markets, generate signals, and place trades. This pass verified **no** major-bank production deployment of a fully autonomous LLM trading agent; the credible named deployments are research copilots and execution-advice tools. Claims of autonomous LLM traders at scale are graded ⚠ vendor-marketing unless the deploying firm publishes otherwise.
- ✅ **The research-side lineage.** Bloomberg's path is the mirror image of JPMorgan's: the verified research asset is BloombergGPT (Section 8.2), and the agentic Terminal layer built on it is cross-referenced from the all-industry survey's finance section ([LLM Agent Use Cases](../llm_agent_use_cases.md) §5.1) rather than re-derived here.

### 7.3 The 2025 "AI Trader" Pilots — Production vs Pilot Discipline

The 2025 press cycle produced a wave of "bank deploys AI trader" headlines. Applying the Section 1.3 discipline to what this pass could verify:

- **What is pilot:** most of what the 2025 headlines describe. Sell-side genAI trading copilots (drafting pre-trade briefs, summarising risk, triaging market news) are real and spreading — HSBC's 2026 partnership explicitly targets client-facing and RM tooling with agentic AI (Section 4.1), and UBS's and JPMorgan's firmwide assistants reach the trading and markets divisions (Section 5.2) — but the default grade for a 2025 "AI trader" pilot story is ⚠ pilot, not ✅ production, until the firm's own material confirms general availability.
- **What is production:** the pre-LLM execution engines (LOXM-class), the firmwide copilot platforms (LLM Suite, UBS's Azure OpenAI estate, BofA's enterprise assistants), and the analytics layers of the trading infrastructure. These are verified at primary sources and graded ✅.
- **The tell:** vendor and press prose that says "deployed," "live," or "autonomous" without a named desk, a named instrument class, and a named human-approval control is pilot-grade or marketing-grade. The engineering guide's rollout gates — canary, champion-challenger, kill switches — are precisely the machinery that separates the two, and the worked example of Section 13 applies them to Cymbal Bank's trading-adjacent candidates.

---

## 8. Capital Markets — Research & Analysis Agents

Research and analysis is the capital-markets use case with the deepest LLM penetration, because it is the knowledge-copilot pattern (Section 2.3) applied to the sell side's and buy side's core product: turning information into insight. The evidence is strong at both the model layer and the product layer.

### 8.1 The Earnings-Call and Document Summarisers

- ✅ **HSBC's meeting-prep assistant** — HSBC's June 2026 media release records the AI-powered decision assistant already reducing client-meeting preparation "from hours to minutes for thousands of users," with regulatory-procedure codification next (source: hsbc.com, June 2026 — Section 4.1). This is the RM-facing summarisation copilot at named scale.
- ✅ **Morgan Stanley Debrief** — the advisor meeting-summary tool on Whisper + GPT-4 (Section 5.1) is the wealth-side instance of the same family.
- ⚠ **AlphaSense and the market-intelligence layer** — the survey sibling carries the product-level detail on AlphaSense and Koyfin ([LLM Agent Use Cases](../llm_agent_use_cases.md) §5.1); the client-count claims (thousands of enterprise clients, share of the S&P 100) are vendor-reported ⚠ figures that this guide does not re-verify — cross-reference only.

### 8.2 BloombergGPT and the Terminal AI Layer

- ✅ **BloombergGPT** — verified at the primary record: the Bloomberg-authored paper "BloombergGPT: A Large Language Model for Finance" (Wu, Irsoy, Lu, Dredze, Gehrmann, Kambadur, Rosenberg, Mann; arXiv:2303.17564, March 2023) describes the 50-billion-parameter model trained on a 363-billion-token financial dataset built from Bloomberg's data sources plus 345 billion general-purpose tokens, outperforming existing models on financial tasks. This is the domain-specific-model layer — an existence-and-design ✅, not a deployment metric.
- ⚠ **The Terminal AI layer** — Bloomberg's agentic Terminal interface (the ASKB-style assistant analysing earnings transcripts and generating comp tables, carried in the survey sibling's §5.1) is press- and product-reported; this pass did not verify a Bloomberg primary page describing production scale, so the product is cross-referenced with its ⚠ status intact.

### 8.3 The Buy-Side Research Copilots

- ✅ **BlackRock Aladdin Copilot** — BlackRock's own product page states Aladdin Copilot, powered by generative AI, is "now available to all Aladdin clients," surfacing answers across the Aladdin platform for investment and operational decisions (source: blackrock.com, Aladdin Copilot product page). The availability claim is primary; the client-count and usage claims are not published.
- ⚠ **The agentic Aladdin direction** — trade-press reporting (FundFire, via trade publications, 2025–2026) describes BlackRock extending Aladdin with agentic genAI into execution and exception handling; graded ⚠ press until BlackRock publishes.
- ✅ **The buy-side survey context** — the verified survey record (Section 12) shows institutional traders ranking AI/ML as the most influential technology over the next three years; the buy-side research copilot is where that expectation is already landing in products (Aladdin Copilot, the market-intelligence platforms above) rather than in autonomous decisioning.

The research-agent pattern is uniform: **summarise, retrieve with citation, draft, and propose — never decide.** Every verified deployment in this section keeps the analyst or the portfolio manager as the decision-maker, which is the same human-in-the-loop shape the engineering guide hard-codes into its compliance workflows.

---

## 9. Capital Markets — Surveillance, Compliance & Post-Trade Agents

The control functions of capital markets — watching communications, watching trades, and settling them — are the least glamorous and the most regulation-driven agent use cases, and the evidence base splits cleanly: regulatory pressure is verifiable at the regulators themselves, while most of the agentic product claims are vendor-published.

### 9.1 The Communications-Surveillance AI

- ✅ **The regulatory driver** — the US recordkeeping-enforcement wave (SEC and CFTC actions from 2021 onward against dozens of firms for failures to preserve off-channel communications, with penalties in the billions of dollars in aggregate) is a matter of public regulatory record and is what made communications surveillance the banks' priority spend. The enforcement facts are carried in the sibling AI-compliance guide ([AI Governance, Bias & Red-Teaming](ai_governance_bias_redteaming_guide.md)); the headline scale here is context, not a new claim.
- ⚠ **The vendor product layer** — Bloomberg Vault's announcement of AI-powered communications-surveillance models (for insider-dealing and personal-trading surveillance, 2025) and Global Relay's State of AI in Surveillance survey (31% of firms surveyed already using or planning AI for communications surveillance within a year) are vendor-published: the product launches are real as announced, the adoption statistics are vendor-survey ⚠.
- ⚠ **The named-bank genAI triage deployments** — banking-press reporting (The Banker, 2025) describes banks applying genAI to electronic-communications surveillance to cut false positives; the reporting is real but the banks are largely unnamed in the public versions, so this category stays ⚠ until a bank publishes. Voice surveillance is the stated frontier (transcription + LLM risk detection across voice and video) — the same grading applies.

### 9.2 The Trade-Surveillance Agent Claims

Trade surveillance — detecting market abuse (spoofing, layering, insider dealing, front-running) across orders and trades — runs on the same vendor rails as AML (Nasdaq, Quantexa, NICE Actimize, SymphonyAI, and the specialist surveillance vendors): ✅ the rail deployments are vendor-documented (Section 4.2's Danske example covers market-trading activity), ⚠ the "agentic" investigation copilots are vendor-marketing until a named bank publishes results. The regulator-visible requirement — a surveillance programme whose decisions an examiner can reconstruct — is precisely the audit-trail and explainability standard the engineering guide builds into its compliance-automation agent, and the cross-reference is the whole of the engineering answer here.

### 9.3 The Post-Trade and Operations Agents — STP, Reconciliation, Reference Data

- ⚠ **The vendor frontier** — the post-trade operations stack (straight-through-processing exception handling, reconciliation, reference-data maintenance) is the most heavily marketed agentic category of 2025–2026: SmartStream's "Smart Agents" platform for autonomous exception resolution, and a wave of reconciliation-agent startups, all publish the same claim — the agent resolves the exception and hands the human a pre-built case instead of an investigation. The claims are vendor-published; this pass verified no major-bank primary publication of measured post-trade agent results.
- ✅ **The bank-side signals** — HSBC's 2026 partnership targets exactly this territory (agentic AI across operations with human judgement at the centre, Section 4.1), and JPMorgan's LLM-Suite-as-agent-hub direction (Section 5.2) names workflow agents as the next step. Both are announced programmes with targeted economics — ✅ as announced, ⚠ as measured outcomes.
- **The engineering boundary** — post-trade exception handling is the sibling engineering guide's natural next domain after fraud, AML, compliance, and KYC: the same governed-agent contracts (scope, escalation, evidence, evaluation), the same eval gates, the same canary rollout. The worked example of Section 13 therefore treats post-trade ops as a **second-wave** Cymbal Bank candidate — high value, vendor-crowded, and only safe after the first wave proves the gate machinery on the four core workflows.
---

## 10. Insurance — Claims, Underwriting & Policy-Service Agents

Insurance runs the same four use-case families as banking — the customer-facing assistant, the expert copilot, the risk engine, the operations automator (Section 2.1) — but with two differences that shape the evidence base. First, the **claims decision pays money directly**, so the regulator-visible bar is as high as in credit decisioning. Second, insurance has a genuine pre-LLM agentic pioneer (Lemonade's AI-Jim claims handling) whose marketing outran its evidence, which makes it the cleanest case study in the whole landscape of why the ✅/⚠/❌ discipline exists. The honest 2026 picture: the enterprise LLM rollouts are real and large, the claims and underwriting *copilots* are spreading, and the fully autonomous claims or underwriting *agent* is thin on primary evidence everywhere it is claimed.

### 10.1 The Claims Agents — Lemonade, and the Marketing Flag

- ⚠ **Lemonade's AI-Jim lineage.** Lemonade, the licensed US and European digital insurer (launched 2016), built its brand around an AI claims handler it named AI Jim: the customer files via the app, and a rules-plus-model pipeline adjudicates simple claims, with the company's early press (2017–2020, widely carried) reporting claims paid in seconds to minutes without human touch. The company's own materials and the press record are consistent that *some* claims flow is automated end-to-end — that much is well documented. What is **not** documented at primary level is the marketing-grade claim that this made Lemonade "the world's first fully automated insurance company": the automated flow always sat inside a licensed insurer with human review and regulatory obligations on the long tail of claims.
- ⚠ **The marketing flag — the 2023 advertising-standards scrutiny.** Trade and consumer press (2023) reported that Lemonade's UK advertising, which told customers their claims were handled "in seconds" with no human involvement, drew regulatory-standards scrutiny in the United Kingdom over whether the ads overstated the automation. The episode is this guide's canonical marketing-flag case: the *deployment* was real, the *advertising equivalence* ("no humans anywhere") was not, and the distinction is exactly the one Section 1.3 asks readers to keep. Graded ⚠ press throughout — no party published a formal retraction, and Lemonade continued to market AI claims handling.
- ✅ **The landscape lesson.** Lemonade's significance is structural, not promotional: it proved that a licensed insurer could put a **domain-specific claims agent** (Section 2.2's sense — sense, decide, act, no LLM at the core) in front of customers a full six years before the LLM wave, and that the market's attention would go to the marketing number rather than the engineered control. The modern LLM-era claims copilots of Section 10.2 are the heirs to that proof — with the same temptation attached.
- ✅ **What the verified record actually supports.** Read narrowly, the Lemonade record supports three statements and no more: (1) a digital insurer ran an automated pipeline for a subset of simple claims; (2) the subset boundary, the human-review overlay, and the regulatory perimeter were never published at primary level; (3) the marketing equivalence ("no humans") drew standards scrutiny. Every stronger reading — "claims are fully automated at scale," "AI outperforms human adjusters," "the model is the licensed decision-maker" — is ⚠ press-inferred or vendor-prose, and the reader should grade it accordingly.
- **The marketing-flag pattern, generalised.** The Lemonade shape — real bounded automation, advertised as total automation, then scrutinised by a standards body — recurs across the whole landscape: Klarna's FTE-equivalence (Section 3.3), the autonomous-trader prose (Section 7.3), and the agentic-AML decks (Section 4.2) are the same pattern in different costumes, and the Section 1.3 markers are the costume check. The insurance lesson is not that Lemonade was wrong to automate — it is that the *advertising equivalence* between the automation and the company outran the published boundary of the automation, and the markers exist to keep the two apart.

### 10.2 The Enterprise LLM Rollouts — AllianzGPT, Zurich, Ping An, OneConnect

The large-incumbent record splits into the Western genAI build-outs (press-documented, ⚠ on scale) and the Asian pre-LLM AI claims estates (company-reported, ⚠ on current vintage):

- ⚠ **AllianzGPT.** Press and industry coverage (2023–2025) documents Allianz's internal generative-AI platform, AllianzGPT, built on Microsoft's Azure OpenAI stack and rolled out to staff across the group for knowledge work, document handling, and customer-facing support drafting. The platform's existence is consistently press- and vendor-story-reported; the tens-of-thousands-of-employees scale figures are press-carried and were not confirmable at an Allianz primary page in this pass — ⚠.
- ⚠ **Zurich.** Zurich Insurance Group has publicly described generative-AI deployments across claims handling and underwriting through 2024–2026 (press and industry-event reporting, including named executive statements at conferences): claims handlers receiving AI-drafted summaries and next-best-action suggestions, and underwriting workbenches for submission triage. The named-executive statements give the direction ✅-as-spoken; the measured results (cycle-time and leakage improvements) remain press-carried ⚠.
- ⚠ **Ping An.** Ping An Insurance Group's AI claims estate predates the LLM wave: the group has long published (company reports and press) that AI handles the majority of its auto-claims reporting and a large share of claims decisions, with facial-recognition and image-damage-assessment components — a domain-specific-agent estate at a scale no Western insurer matches. The figures are company-reported ⚠ and this pass did not verify the current vintage at a primary page; the *existence* of the estate is uncontroversial.
- ⚠ **OneConnect.** OneConnect (Ping An's financial-technology unit, listed in New York until its 2024 delisting) markets AI insurance technology — claims, underwriting, and customer-service modules — to other financial institutions across Asia. The products are real as marketed; the client-result numbers are vendor-case-study ⚠.
- ⚠ **The reinsurance and MGA layer.** The specialist layer beneath the primary insurers — reinsurers pricing the tail, and managing general agents (MGAs) running niche books on the carriers' paper — is the quietest part of the vendor market: claims and submission-intelligence copilots marketed to MGAs and reinsurance analysts (document triage, exposure drafting, bordereaux checking). The deployments are vendor-announced ⚠, and the measured results are even thinner than at the primary layer, because the buyers are smaller and publish less. The pattern is the same as Section 4.2's vendor rails: the purchase is often verifiable, the agentic result almost never is.
- ⚠ **The claims-fraud and claims-operations vendor layer.** The specialist vendors that the insurers buy from — Shift Technology (claims-fraud detection, "Force"-family products), Tractable (AI photo-based vehicle-damage estimation, used by major auto insurers per vendor stories), and the policy-administration LLM copilots — all publish the same shape of claim: the model finds the fraud or prices the damage the human would have missed or taken hours over. Vendor-published ⚠ until a named insurer publishes measured results; the engineering guide's champion-challenger and eval-gate conventions (Section 4.3's cross-reference) are the machinery that would turn those claims into evidence.

### 10.3 The Underwriting and Policy-Service Agents — Vendor vs Regulator-Visible

- ⚠ **The underwriting-copilot category.** The 2025–2026 vendor market sells "agentic underwriting" to carriers and MGAs: LLM agents that triage submissions, extract risk information from broker documents, draft coverage terms, and propose pricing against the carrier's actuarial tables. The credible deployments are **copilots with human underwriter approval** — the same HITL ladder the banking sections found (Section 6.4) — and the fully autonomous risk-decision agent is vendor-marketing until a carrier's actuarial and conduct regulator sees a filing. Pricing and reserving decisions sit under actuarial standards that no vendor page can pre-empt, which is why the honest category label in 2026 is **copilot-assisted underwriting** here too.
- ⚠ **The policy-service assistants.** Customer-facing policy-service LLM assistants (policy questions, document requests, mid-term changes) are in pilot and early production across life and general insurers — press- and vendor-reported ⚠, with the same usage-vs-economics split as banking's Section 3.3: the assistant volumes are increasingly published, the FTE-equivalence arithmetic is not.
- ⚠ **The claims-operations automator rung.** Behind the customer-facing claim sits the operations stack the vendors market hardest: the claims-ops agent that reads the notification, checks the policy, orders the assessment, and drafts the settlement letter, escalating only the exceptions. This is the insurance instance of Section 9.3's post-trade pattern — same vendor prose, same missing bank- or carrier-published measurements, same second-wave sequencing logic for a buyer's portfolio (Section 13.2 grades Cymbal Bank's version P2 for exactly that reason).
- ✅ **The engineering cross-reference.** The insurance reader who wants the claims-triage architecture, the fraud-agent eval pipelines, and the rollout gates does not wait for an insurance-specific engineering guide: the sibling banking deep-dive's governed-agent contracts are domain-agnostic, and Section 4.3's cross-reference applies to insurance with the domain's conduct and actuarial overlay substituted for banking's. The worked example's insurance candidates (Section 13.2) are graded on exactly those contracts, which is the honest answer to "where is the insurance how-to" — it is the banking how-to, re-overlaid.
- ✅ **The regulator-visible bar.** The reason the autonomous claims and underwriting rungs stay thin is regulatory: claims decisions are conduct-regulated, pricing is actuarial-standard-regulated, and both are subject to model-risk and explainability review in the same way as credit (Section 6's opening). Nothing in this section contradicts that bar; the vendors' autonomy prose does. The engineering guide's governed-agent contracts — scope, escalation, evidence, evaluation — are written for exactly this standoff, and the worked example of Section 13 ranks Cymbal Bank's insurance candidates against it.

---

## 11. Payments — Authorization, Disputes & Sanctions-Screening Agents

Payments is the domain where the **pre-LLM agentic engine is the industry's backbone** — the authorization decision on every card transaction is already made by a specialised decision system in milliseconds — and where the LLM wave is therefore arriving as an *overlay* on engines that were agentic before the word existed. The domain context is Section 1.2's payments row: networks, schemes, acquirers, and issuers, governed by scheme rules and the sanctions regimes (OFAC, UN, EU, UK, MAS). The sanctions-screening half of this section is a **condensed cross-reference only** — the full estate, the vendor mechanics, and the name-matching engine live in the sibling deep-dive [Fircosoft — the AML/Sanctions Screening Engine](../banking/fircosoft_guide.md).

### 11.1 Visa Advanced Authorization — the Pre-LLM Agentic Baseline

- ✅ **The engine.** Visa Advanced Authorization is established in Section 2.2 as the canonical pre-LLM domain-specific agent of the payments domain: a specialised decision system that scores each authorization request on VisaNet in milliseconds against a deep model of the transaction's features and the cardholder's history, deciding an approve/decline recommendation before the human (or the issuer's own policy) ever sees it. No LLM is anywhere in the loop; the "agentic" label fits because it senses, decides, and acts at machine speed with a defined objective function (fraud loss versus declined-good-transaction cost).
- ⚠ **The company-reported scale.** Visa's own materials state that Visa Advanced Authorization and the surrounding risk services help prevent on the order of tens of billions of US dollars in annual fraud (the "$30 billion"-class figure appears across Visa statements and press) and that the models score the network's transaction flow in real time. The mechanism and the direction are company-consistent and widely carried; the precise dollar figure is the **company's own estimate** — ⚠, not independently audited. The FICO Falcon family, the other canonical pre-LLM fraud-scoring engine used across issuers, carries the same shape of claim from its vendor.
- ✅ **The issuer-side reality.** The network engine recommends; the issuer decides. Every cardholder-facing "AI declined my card" story is the visible edge of that division of labour: the network's score is one input to the issuer's own authorization policy, which is why the same model behaviour reads as fraud prevention to the network and as a false decline to the cardholder. The engineering guide's champion-challenger discipline applies on the issuer side exactly as it does inside the bank: the decline rate is a measured cost, not a vendor stat.
- **The false-decline economics.** The declined-good-transaction is the engine's other cost — measured by issuers in decline rates, re-presentment, and call-abandonment rather than in the network's prevented-loss dollar — and it is the number the LLM-era cardholder copilots (explaining why a decline happened, and what the cardholder can do next) exist to soften. The two costs together — the fraud that got through and the good transaction that did not — are the objective function any authorization agent is really optimising, which is why the vendors' single-number prevented-loss stories (11.1's ⚠ figure) are always half the ledger.
- ⚠ **The LLM overlay.** Visa's 2025–2026 public posture (press and company statements) adds generative-AI and agentic tooling on top of the deterministic core — dispute-document reading, risk-analyst copilots, merchant and issuer insights. The overlay is real as announced; the measured results are the company's own ⚠. The architecture is the hybrid of Section 2.2: the LLM reads and explains, the deterministic engine decides.

### 11.2 Mastercard Decision Intelligence and the New LLM Dispute Agents

- ⚠ **Decision Intelligence.** Mastercard's Decision Intelligence (launched around 2016, press- and company-documented) and its real-time successor Decision Intelligence Pro apply device-account-velocity and network-pattern scoring to authorization in the same pre-LLM agentic family as Visa Advanced Authorization; the launch and the product direction are company-and-press ✅-as-published, the loss-reduction percentages in the company's materials are ⚠ self-reported.
- ⚠ **The new LLM dispute agents.** The 2025–2026 news cycle carries network and vendor announcements of LLM agents for the dispute lifecycle — reading the cardholder's claim, assembling the evidence package, drafting the representment, predicting the scheme-rule outcome. Mastercard's public genAI-for-disputes direction and the specialist dispute-automation vendors (chargeback-exception agents marketed to acquirers and issuers) all publish the same claim shape: the agent builds the case the human used to build in hours. This pass verified **no issuer or acquirer primary publication** of measured dispute-agent results — the category is ⚠ vendor- and network-self-reported until scheme participants publish, which matters because dispute outcomes are scheme-rule-governed and the rules do not change because a vendor says so.
- ⚠ **The acquirer-side reality.** The dispute agent's other buyer is the acquirer, which fights the merchant's corner of the same scheme rules: evidence-assembly copilots for representment and a separate wave of "chargeback-prevention" agents that flag the transaction characteristics that precede disputes. The same grading applies — vendor-published ⚠, no acquirer primary publication of measured win-rate or cycle-time results found in this pass — and the scheme-rule point is doubly true on the acquirer side, where the representment deadline calendar is the real agent.
- **The deadline calendar.** The representment calendar is the fixed machinery of the dispute process: scheme deadlines that no LLM can extend. The copilot's value is therefore in assembling the evidence package *before* the deadline and predicting which cases are worth the fight — not in arguing after the window closes — and a buyer should evaluate dispute-agent vendors on that calendar's terms: time-to-evidence-package and outcome-by-case-class, measured on the buyer's own historical disputes, which is the champion-challenger of Section 12.2 applied to a scheme-rule process.
- ✅ **The dispute copilot is the honest rung.** The cardholder-facing dispute-intake assistant (explaining, gathering the claim, setting expectations) and the analyst-facing evidence copilot sit on the same evidence footing as banking's customer-service and comms-surveillance copilots (Sections 3, 9.1) — internal or low-autonomy, curated, measurable — and are the rung the worked example of Section 13 recommends for Cymbal Bank's payments candidates.

### 11.3 The Sanctions-Screening Overlay — Condensed Cross-Ref

- ✅ **The boundary.** Sanctions screening is the payments use case with the heaviest regulatory weight and the most deterministic core: payment messages are screened against the sanctions lists (OFAC, UN, EU, UK, MAS — Section 1.2's payments row) by name-matching engines whose false positives are then investigated by humans. The full estate — the screening pipeline, the fuzzy name-matching mechanics, the list-management and false-positive economics, and the vendor's engine architecture — is the entire subject of the sibling deep-dive [Fircosoft — the AML/Sanctions Screening Engine](../banking/fircosoft_guide.md); the screening half of this section deliberately stops at the boundary and cross-references rather than re-derives.
- ✅ **What the LLM adds — and does not.** The screening *decision* (is this name a match to this list entry?) remains list-driven and deterministic — no LLM holds that decision right anywhere in the verified record, and the engineering reality is that it should not. The LLM's verified role is the **hybrid overlay of Section 2.2**: triaging the false-positive queue, drafting the investigation narrative, summarising the evidence the human investigator checks before release or escalation. HSBC's published scale context (Section 4.1 — roughly a billion transactions monitored monthly across the financial-crime estate) is the order of magnitude the triage copilots exist to cut into.
- ⚠ **The instant-payments frontier.** The friction the screening engines were built for is shrinking: instant-payment schemes settle in seconds, so the screening window closes faster than the name-matching queue drains. The vendor answer — screening models tuned for the instant window plus LLM triage that keeps the human decision on the released-or-held call — is marketed across the scheme and vendor layer ⚠; the measured false-positive economics at instant-payment speed are not published by any named bank in this pass's record.
- ✅ **The reading order for the screening reader.** The reader who wants the screening estate whole — the pipeline, the name-matching engine, the false-positive economics, the list management — starts at the Fircosoft sibling and returns here for the agentic overlay only. The reader who wants the overlay's place in the landscape has it here: the screening decision stays deterministic, the LLM triages and drafts, and every "agentic screening" vendor claim grades ⚠ until a named bank publishes — the same sentence, in the same words, as Sections 4.2 and 9.2, which is the point of a landscape guide.
- ⚠ **The vendor layer.** The screening-adjacent vendor market (including rails carried in Section 4.2's list) markets agentic investigation copilots on top of the deterministic match engines; the product pages are the source, and the named-bank measured results are not published — ⚠ vendor-marketing until a bank publishes, the same grading as every other agentic-copilot claim in Sections 4 and 9.

---

## 12. The Evidence-Quality and Adoption-Reality Assessment

Section 12 is where the guide cashes the promissory notes of Sections 7.2 and 8.3: the claimed-deployments evidence table (12.1), the production-versus-pilot adoption reality on the verified surveys (12.2), and the self-reported and analyst-forecast ⚠ flags (12.3). The method is Section 1.3's throughout: every deployment claim in Sections 3–11 carries a marker, and the table below is the audit of how those markers distribute — not a scoreboard of the vendors, but a map of *where the evidence is strong enough to build on*.

### 12.1 The Claimed-Deployments Evidence Table

| The Deployment Claim (Section) | The Marker | The Evidence Class | The Basis |
|---|---|---|---|
| Morgan Stanley's GPT-4 advisor assistant in production (3.1, 5.1) | ✅ | Primary ×2 | Morgan Stanley press release (March 2023) + OpenAI case-study page |
| AI @ Morgan Stanley Debrief on Whisper + GPT-4 (5.1) | ✅ | Primary | OpenAI case-study page |
| "98% of advisor teams" adoption (3.1) | ⚠ | Press | Widely carried; not confirmable at a Morgan Stanley primary page |
| Erica's usage record — ~3 billion interactions, 50 million users (3.2) | ✅ | Primary | Bank of America newsroom (August 2025) |
| ask MERRILL and ask PRIVATE BANK — ~23 million interactions a year (3.2, 5.2) | ✅ | Primary | Bank of America newsroom (August 2025) |
| Fargo — 1 billion interactions in under three years (3.2) | ✅ | Primary | Wells Fargo newsroom (2026) |
| CommBank Copilot and the genAI knowledge-search tool (3.2) | ✅ | Primary | CommBank newsroom (March 2024; August 2023) |
| DBS — SGD 750 million AI economic value, 1,500+ models (3.2) | ✅ | Primary | DBS Annual Report 2024 |
| Klarna assistant — 2.3 million conversations in month one, "work of 700 agents" (3.3) | ⚠ | Vendor self-reported | Klarna press release (February 2024) + OpenAI case study |
| Klarna — "700 jobs replaced" reading (3.3) | ❌ | Disputed | Contradicted by the record: output-equivalence, not headcount reduction; Klarna continued hiring |
| HSBC on Google Cloud AML AI — 2–4× true positives, >60% fewer alerts (4.1) | ✅ | Joint vendor–client release | Google Cloud launch release quoting HSBC's Group Head of Financial Crime Risk and Compliance (June 2023) |
| HSBC 2026 agentic partnership — 200+ use cases, US$100 million-class targets (4.1) | ✅ announced / ⚠ outcomes | Primary (announcement) | HSBC media release (June 2026); targets are the bank's own, not measured results |
| Danske Bank on Quantexa (4.2) | ⚠ | Vendor | Quantexa announcements carried by industry press |
| Zest AI — 22 million lending decisions in 2023 (6.2) | ⚠ | Vendor self-reported | Zest AI press releases (2024) |
| UBS Azure OpenAI rollout — 30,000 employees in ten months (5.2) | ✅ | Co-marketing case study | Microsoft customer story; named-division detail is UBS's own |
| UBS Red — ~52,000 employees, GA first half 2026 (5.2) | ⚠ | Press | finews and Swiss financial press (2025) |
| JPMorgan LLM Suite — 200,000 users in eight months (5.2) | ✅ | Primary | JPMorganChase technology blog (June 2025) |
| Upstart — 91% of loans fully automated, Q2 2026 (6.1) | ✅ | Primary (SEC-filed issuer) | Upstart Q2 2026 earnings release (August 2026) |
| JPMorgan COiN — 12,000 agreements in seconds, 2017 (6.3) | ✅ existence / ⚠ figures | Press | FT and technology press (2017); no maintained bank primary page |
| LOXM execution trial, 2017 (7.1) | ✅ existence | Press (FT exclusive + corroboration) | FT 2017; Business Insider corroboration; "15% improvement" figure ⚠ and not asserted |
| Fully autonomous LLM trading agent in production at a major bank (7.2–7.3) | ❌ | No verified deployment | Every named 2025–2026 "AI trader" story grades ⚠ pilot; autonomous order flow unverified anywhere |
| BloombergGPT (8.2) | ✅ | Primary (paper) | arXiv:2303.17564 (March 2023) |
| Aladdin Copilot available to all Aladdin clients (8.3) | ✅ as-published | Vendor product page | blackrock.com Aladdin Copilot page; usage counts not published |
| Communications-surveillance enforcement wave (9.1) | ✅ | Regulatory record | SEC/CFTC actions from 2021 onward (carried in the AI-governance sibling) |
| Global Relay survey — 31% using or planning AI comms surveillance (9.1) | ⚠ | Vendor survey | Global Relay, State of AI in Surveillance |
| Fargo's launch stack — Dialogflow on PaLM 2, "Tachyon" multi-LLM platform (3.2) | ✅ as-stated | Named-executive statements | Wells Fargo CIO at an industry event, carried by VentureBeat (January 2024) |
| Erica for Employees — 90%+ employee usage, 50% fewer IT service-desk calls (3.2) | ✅ | Primary | Bank of America newsroom (August 2025) |
| Klarna — 25% drop in repeat inquiries, 1–2-minute resolutions (3.3) | ⚠ | Vendor self-reported | Klarna press release (February 2024); no published controlled methodology |
| Betterment and Wealthfront client assets — ~US$65bn / ~US$95bn (5.3) | ⚠ | Company disclosures via trackers | Third-party trackers; scale context only |
| Ask Ceba staff-assistant pilot at CommBank (3.2) | ⚠ | Press | Circulated scale; no CommBank primary page found (Section 15.1) |
| UBS Azure OpenAI divisional rollout — four divisions, three booking centres by December 2024 (5.2) | ✅ as-stated | Co-marketing case study | Microsoft customer story; the named-division detail is UBS's own |
| Bradesco and Lunar on Google Cloud AML AI (4.1) | ✅ as-named | Vendor launch release | Customer names primary; the qualitative praise is vendor-marketing ⚠ |
| Ping An AI auto-claims estate (10.2) | ⚠ | Company-reported | Long-published company figures; current vintage not primary-verified in this pass |
| Shift Technology / Tractable claims-AI deployments (10.2) | ⚠ | Vendor | Vendor case studies; no named-insurer measured publication found |
| Danske Bank Quantexa rail covering market-trading activity (4.2, 9.2) | ⚠ | Vendor | Quantexa announcements carried by industry press |
| SmartStream "Smart Agents" and the post-trade vendor wave (9.3) | ⚠ | Vendor | SmartStream and reconciliation-agent announcements (2025–2026) |
| Visa Advanced Authorization as the pre-LLM authorization engine (2.2, 11.1) | ✅ | Established | Section 2.2's canonical-agent record; launch and mechanism company-and-press documented |
| Visa's annual fraud-prevention dollar figure (11.1) | ⚠ | Company self-reported | Visa's own materials; not independently audited |
| Lemonade AI-Jim automated claims (10.1) | ⚠ | Press + company | 2017–2020 press and company materials; no primary on the automation split |
| AllianzGPT, Zurich, Ping An, OneConnect rollouts (10.2) | ⚠ | Press / company-reported | Press 2023–2026; company figures not primary-verified in this pass |
| Mastercard Decision Intelligence and the LLM dispute agents (11.2) | ⚠ | Company + press / vendor | Launch and direction documented; measured dispute-agent results unverified |

The distribution is the guide's central finding in one view: **the ✅ set clusters at the assistant and copilot rungs and at the deterministic risk engines; the ⚠ set clusters at the vendor case study and the self-reported economics; the ❌ set is small and specific — the jobs-replacement reading of the assistant economics, and the autonomous-LLM-trader story** — which is exactly where the vendor prose is loudest. The table also shows the structural pattern behind the distribution: the ✅ rows are almost all *the deploying institution's own words about its own deployment*, while the ⚠ rows are almost all *the vendor's or the press's words about someone else's deployment* — which is the single most useful heuristic this guide offers for reading the next announcement.

### 12.2 The Adoption Reality — Production vs Pilot, on the Verified Surveys

The adoption reality separates cleanly by the Section 2.3 rungs, and the verified survey record sharpens the picture:

- ✅ **The assistant and copilot rungs are in production at scale.** The usage records of Section 3.2 are bank-published: Erica's ~3 billion interactions, Fargo's 1 billion, LLM Suite's 200,000 onboarded users, UBS's 30,000-employee rollout, HSBC's "thousands of users" on meeting-prep (Section 8.1). This rung is not a pilot story anywhere in the verified record — it is the industry's largest verified AI deployment class, and its economics (Section 3.3) are the only part that degrades to ⚠.
- ⚠ **The verified survey record — the e-Trading surveys.** The survey record that Sections 7.2 and 8.3 promised: the e-Trading survey series (JPMorgan's annual survey of institutional trading professionals — results press-carried ⚠) shows traders ranking AI/ML as the **most influential technology over the next three years**, while the same respondents put the weight of expected impact on analytics, execution advice, and workflow assistance — *not* on the machine trading alone. The buy-side reading of the same record (Section 8.3) lands the expectation in research copilots, not autonomous decisioning. Direction: the practitioners expect the copilot rung to deepen, and the agent rung to stay governed.
- ✅ **The production scoreboard, by rung.** Summing the verified record into one scoreboard: **in production and bank-published** — the customer-service and expert copilots of Sections 3 and 5, the AML and fraud engines of Section 4, the pre-LLM execution and authorization engines of Sections 7 and 11, the research copilots of Section 8. **In pilot or vendor-announced only** — the agentic trade copilots of Section 7.3, the agentic surveillance and post-trade products of Section 9, the LLM dispute agents of Section 11.2, the claims and underwriting copilots of Section 10. **Not funded anywhere in the verified record** — the autonomous credit decision, the autonomous trade, the autonomous claim payment, the autonomous screening release. The scoreboard is the guide's answer to "what is actually deployed": the copilot is deployed, the bounded agent is deployed inside the bank's own walls, and the autonomous money-move agent is not deployed at all.
- ✅ **The four domains, one verdict.** Reading the scoreboard domain by domain (the Section 1.2 map) shows the same shape four times. **Banking** — the deepest verified record: customer-service and expert copilots in production at billion-interaction scale (Section 3), AML and fraud engines in production with published results (Section 4), and the autonomous credit decision absent (Section 6). **Capital markets** — the cleanest rung separation: research copilots in production (Section 8), the pre-LLM execution engine verified and its LLM successor absent (Section 7), surveillance and post-trade still vendor-frontier (Section 9). **Insurance** — the thinnest primary record: enterprise LLM rollouts press-documented (Section 10.2) but no carrier-published measured claims or underwriting results in this pass's record, and the one famous autonomous claim (Lemonade) is the marketing-flag case (Section 10.1). **Payments** — the strongest pre-LLM baseline and the weakest LLM-agent record: the deterministic authorization engines are the industry backbone (Section 11.1), while every LLM dispute agent is vendor- or network-self-reported (Section 11.2). The one verdict that holds across all four: the copilot rung is production, the autonomous rung is not, and the distance between them is regulatory, not technical.
- ✅ **The agent rung is in production only where the blast radius is bounded.** The fraud-triage and AML-screening agents of Section 4, the post-trade exception handlers of Section 9.3's vendor frontier, and the deterministic engines of Sections 7 and 11 are the production agent rung — every one of them bounded by scope, escalation, and human decision rights on the money move. The fully autonomous customer-facing or decision-facing agent — the credit decision, the trade, the claim payment, the screening release — has **no verified production deployment in this guide's record**, and the regulatory analysis of Sections 6, 10, and 11 explains why that is equilibrium, not lag.
- ✅ **The failure record is the companion.** The failure catalogue [LLM Agents Failures in Production](llm_agents_failures_production_guide.md) documents what happens when production agents outrun their evaluation and escalation contracts, and the compliance layer [AI Governance, Bias & Red-Teaming](ai_governance_bias_redteaming_guide.md) carries the model-risk and conduct overlay — the two cross-reference lines Section 1.1 promised. The adoption reality verdict, in one sentence: **the industry has put the copilot in production everywhere and the autonomous agent almost nowhere, and the verified record says the gap is governance, not capability.**
- ✅ **What would change this verdict.** The verdict is evidence-based, so it is evidence-revisable, and the reader should know which four developments would move it. (1) **A bank publishes measured autonomous-decision results** — a named bank's regulatory filing or primary announcement of an autonomous credit, trade, or claims decision in production with eval results would move the corresponding row of Section 12.1 from ❌/⚠ to ✅ and re-open the Section 13 not-funded list. (2) **A carrier publishes measured claims-agent results** — the single emptiest cell in the whole table (Section 10) — would pull insurance's agent rung out of the vendor-prose column. (3) **A scheme or network publishes third-party-audited dispute-agent outcomes** rather than its own estimates would upgrade Section 11.2. (4) **A regulator speaks** — a conduct, model-risk, or actuarial regulator issuing guidance that names the autonomous rung (permission, prohibition, or conditions) would settle the regulatory-axis question that Sections 6, 10, and 11 currently answer only by inference from existing rules. Until one of the four happens, the verdict stands, and the honest default for a 2026 finance-AI headline remains: ⚠ until the deploying firm publishes.

### 12.3 The Self-Reported and the Analyst-Forecast — the ⚠ Flags

Two claim classes in the landscape are structurally ⚠ and should be read as such:

- **The self-reported economics.** The vendor's own estimate of its value — Klarna's FTE-equivalence (3.3), Visa's fraud-prevention dollar figure (11.1), Zest AI's decision counts (6.2), the vendor case-study percentages of Sections 4.2, 9.2, 10.2, and 11.2 — is the claimant's number on the claimant's channel. It is not fraud and it is not evidence; it is an input to a champion-challenger that the buyer must run. The engineering guide's eval gates are the buyer's machinery for converting self-reported ⚠ into measured ✅ or measured ❌.
- ⚠ **The analyst forecasts.** The forecast layer — analyst-house projections that agentic AI will automate large shares of finance-industry workflows, reduce cost bases by double-digit percentages, or run most back-office exceptions within three to five years — is directionally consistent with Section 12.2's production record but is **forecast, not deployment evidence**, and this guide does not grade forecasts ✅. Treat them as scenario inputs: the verified record says the copilot rung compounds first, and the agent rung follows only where the governance machinery (Section 12.2's verdict) arrives with it. Any specific forecast percentage circulating in 2025–2026 vendor decks is ⚠ by construction until the forecast's own methodology is published.
- **How to read a vendor deck, in three moves.** The Section 1.3 markers applied to the typical 2026 agentic-finance deck: (1) separate the *deployment sentence* (who runs it) from the *result sentence* (how much better it made them) — the first is often true and verifiable, the second is the ⚠; (2) ask which rung the described autonomy occupies on the Section 2.3 gradient — most decks describe a copilot in agent prose; (3) check whether the named client appears as a logo or as a publishing party — a logo is ⚠ vendor-marketing, a publishing party is the start of a ✅. The worked example of Section 13 institutionalises these three moves as its evidence axis.

The flag classes, tabulated for the reader's own vendor-deck work:

| The Flag Class | The Typical Source | The Marker | The Buyer's Move |
|---|---|---|---|
| The deployment sentence | The deploying firm's own release | ✅-eligible | Verify against the primary, then reuse |
| The result sentence | The vendor case study quoting the client's logo | ⚠ | Run the champion-challenger; ask the client to publish |
| The economics estimate | The vendor's own arithmetic (Klarna, Visa) | ⚠ | Demand the methodology; treat as a hypothesis |
| The autonomy prose | The deck's "autonomous / agentic / deployed" vocabulary | ⚠ until the rung is named | Map to the Section 2.3 rungs; demand the control be named |
| The adoption survey | The vendor's or analyst's survey of "the industry" | ⚠ | Ask who was sampled and how |
| The forecast | The analyst house's projection | ⚠ by construction | Use as a scenario, never as an ROI input |
| The headline reading | The press echo of any of the above | ❌ when it outruns the record | Apply the Section 14 audit columns |

---

## 13. The Cymbal Bank Worked Example — the Agent Use-Case Portfolio Review

> **Design fiction notice.** Cymbal Bank is a **fictional** pan-Asian retail and commercial bank used throughout this repository's guides as the only permitted bank persona — a naming discipline that keeps the guides free of real-client inference. This worked example is **design fiction**: it exercises the evidence and prioritisation method of this guide on a hypothetical portfolio, and nothing in it describes a real institution, a real deployment, or a real product decision. The fictional Cymbal Bank's fictional board has asked its fictional architecture office — the reader — for a finance-wide agent use-case portfolio review: which agent candidates across the four domains of Section 1.2 should Cymbal Bank fund, in what order, and on what evidence?

### 13.1 The Portfolio-Review Method — Extending the Eval-Gate Conventions

The review method extends the sibling engineering guide's eval-gate conventions ([Multi-Agent AI Systems for Banking](./multi_agent_banking_guide.md)) from the single-workflow level to the portfolio level. Each candidate is scored on four axes, in order:

1. **The value axis** — the size of the measurable outcome (cycle time, false-positive volume, revenue-assisted, risk reduced), taken from the bank's own operational data, not from vendor case studies.
2. **The evidence axis** — the Section 1.3 marker of the strongest *external* evidence that this use case works at the claimed rung: ✅-verified deployments elsewhere in this guide's record, ⚠ press or vendor claims, or ❌-contested. A candidate whose external evidence is ⚠ vendor-marketing is not rejected — it is **downgraded a rung** until the bank's own champion-challenger supplies the ✅.
3. **The regulatory axis** — the weight of the conduct, model-risk, actuarial, or scheme-rule overlay on the candidate's money move (Sections 6, 10, 11's analysis). High regulatory weight caps the honest rung at copilot-with-human-approval regardless of the vendor's autonomy prose.
4. **The gate axis** — whether the bank already runs the eval-gate, canary, champion-challenger, and escalation machinery (the engineering guide's production standards) for the candidate's workflow family. First-wave candidates are chosen from the workflow families where that machinery already exists; everything else is second-wave, because the verified failure record (Section 12.2's cross-reference) says the machinery is the adoption constraint.

**The method applied to one candidate, as illustration.** Take Cymbal Bank's fraud-triage candidate. The value axis: the bank's own alert-queue data says investigators spend most of the working day clearing false positives — a measurable, internal number. The evidence axis: the external record for fraud-triage agents is the strongest in the guide (Section 4.1's HSBC–Google Cloud results, the engineering guide's production pattern) — ✅-pattern, not ⚠-vendor. The regulatory axis: the decision to *file or close* stays with the investigator, so the money move is human-held and the model-risk overlay is the familiar AML one. The gate axis: Cymbal Bank already runs the four-core-workflow machinery this candidate needs. Four axes, one answer: P0, first wave. Now run the same four axes over the autonomous-credit candidate: the value is large, the external evidence is ❌-empty, the regulatory axis caps the rung at copilot, and no gate machinery exists for a decision no human takes — the method returns "not funded" without a single subjective judgement. That is the point of scoring the axes in order: the evidence and regulatory axes do the filtering before the value axis gets a vote.

### 13.2 The Candidate Portfolio Across the Four Domains

| The Candidate (Domain) | What the Agent Would Do | The Honest Rung Today | The External Evidence | The Priority |
|---|---|---|---|---|
| **Fraud-triage agent** (banking, Section 4's pattern) | Triage the alert queue, draft the investigation file, escalate the decision to the human investigator | Agent, bounded — the engineering guide's four-core-workflow pattern | ✅-pattern: HSBC's AML-AI results (4.1) and the engineering guide's production record | **P0 — first wave** |
| **RM meeting-prep and client-brief copilot** (banking / capital markets, Section 8.1's pattern) | Turn the call transcript and the relationship file into the pre-meeting brief, with citations | Copilot | ✅-pattern: HSBC's "hours to minutes for thousands of users" (8.1), Morgan Stanley Debrief (5.1) | **P0 — first wave** |
| **AML-screening triage agent** (banking, Sections 4 and 11.3) | Cut the false-positive queue with drafted narratives; the deterministic match engine keeps the decision | Agent, bounded — hybrid of Section 2.2 | ✅-pattern: the HSBC–Google Cloud record (4.1); screening mechanics cross-ref the Fircosoft sibling | **P0 — first wave** |
| **KYC document-intelligence copilot** (banking, Section 4's workflow) | Extract, summarise, and flag the onboarding file; the compliance officer approves | Copilot | ⚠-vendor for the agentic claims; the extraction lineage is COiN-class ✅-press (6.3) | **P1 — first wave** |
| **Communications-surveillance triage copilot** (capital markets, Section 9.1) | Rank the surveillance alert queue and draft the case file | Copilot | ✅-driver: the enforcement record (9.1); ⚠-press for named-bank genAI results | **P1 — first wave** |
| **Dispute-intake and evidence copilot** (payments, Section 11.2) | Explain the dispute process, gather the claim, draft the evidence package | Copilot | ⚠-vendor for the agentic dispute claims; the copilot rung matches the Section 3 evidence class | **P1 — first wave** |
| **Claims-document triage copilot** (insurance, Section 10.3) | Read the claim and its attachments, draft the assessment file for the claims officer | Copilot | ⚠-press: the Section 10.2 rollouts; no ✅-primary insurer result in the record | **P1 — first wave** |
| **Post-trade exception agent** (capital markets, Section 9.3) | Resolve the STP exception and hand the human a pre-built case | Agent, bounded — second-wave only | ⚠-vendor frontier; bank-side signals are announced programmes (9.3) | **P2 — second wave** |
| **Underwriting-memo copilot** (banking / insurance, Sections 6.4 and 10.3) | Draft the credit or risk memo with cited evidence; the underwriter decides | Copilot | ⚠-vendor; the ✅-record stops at the specialist lender (6.1), not the bank's own line | **P2 — second wave** |
| **Instant-payments exception copilot** (payments, Section 11.3) | Draft the release-or-hold case for the instant-settlement queue; the screening engine keeps the decision | Copilot | ⚠-vendor frontier; no named-bank instant-payments numbers published | **P2 — second wave** |
| **Voice-surveillance triage copilot** (capital markets, Section 9.1) | Transcribe and rank voice and video risk alerts for the investigator | Copilot | ⚠-press: the stated frontier with no named-bank primary (9.1) | **P2 — second wave** |
| **Fully autonomous credit decisioning** (banking, Section 6.4) | The agent holds the credit decision end-to-end | Not reachable today | ❌-record: no verified bank deployment; model-risk and fair-lending overlay (6.2) | **Not funded** |
| **Fully autonomous LLM trade execution** (capital markets, Section 7.3) | The agent places the order | Not reachable today | ❌-record: no verified deployment; the ✅-lineage is the pre-LLM engine (7.1) | **Not funded** |
| **Fully autonomous claims payment** (insurance, Section 10.1) | The agent pays the claim without human review | Not reachable today | ⚠-press at best; the Lemonade marketing flag (10.1) is the cautionary tale | **Not funded** |

**The portfolio-level observation.** Read down the priority column and the shape is the Section 12.2 scoreboard restated for one bank: every funded candidate sits on the copilot rung or on the bounded-agent rung inside the bank's own walls; every not-funded candidate sits on the autonomous money-move rung. The portfolio has no candidate whose external evidence is ⚠-vendor *and* whose priority is P0 — the method refuses to let the vendor deck set the funding order, which is the whole difference between a portfolio review and a vendor shortlist. And read across the domain column: the four domains of Section 1.2 all appear in the funded set, which is the board-level answer to "is this a banking story or a finance story" — the same agent patterns, the same rungs, and the same evidence logic repeat in all four, with the regulatory weight of the money move as the only real differentiator.

### 13.3 The Prioritisation Recommendation

The recommendation the fictional board receives, in the method's own terms:

- **Fund the P0 first wave as one programme, not three projects.** The fraud-triage, AML-triage, and meeting-prep candidates share the same machinery — the eval gates, the canary rollouts, the escalation contracts of the engineering guide's four core workflows — and the meeting-prep copilot is the cheapest possible first production agent (curated corpus, internal users, Section 5's evidence class). The P0 rationale is the evidence axis: these are the only Cymbal Bank candidates whose external record is ✅-pattern rather than ⚠-vendor, and the only ones whose workflow families already run under the bank's gate machinery.
- **Sequence the P1 wave behind the first production quarter.** The surveillance, dispute, KYC, and claims copilots are all the same architectural shape as the P0 copilots — retrieval, drafting, citation, human approval — and each one's external evidence is one rung weaker (⚠-press or ⚠-vendor), which is precisely why each must clear the bank's own champion-challenger before it earns its production badge. This is the Section 12.2 verdict applied: the copilot rung compounds first.
- **Hold the P2 wave and the not-funded list as explicit governance positions.** The post-trade exception agent is funded only after the first wave proves the gate machinery on the four core workflows (Section 9.3's boundary, restated in portfolio terms). The not-funded list is not a technology judgment — it is the regulatory-axis cap of Sections 6, 10, and 11: the bank's model-risk and conduct obligations make the autonomous credit, trade, and claims-payment rungs unreachable until the regulator-visible evidence base exists, and no vendor page can supply that. When the fictional Cymbal Bank's competitors claim otherwise, the board now has the Section 1.3 markers to read the claim: ✅ if the deploying firm publishes, ⚠ if the vendor does.

**The board's standing agenda.** The portfolio review is not a one-time deck; it is a standing agenda with four standing items. **First, the quarterly re-grade** — every candidate's evidence axis is re-run against Section 12.2's four verdict-changers, because the P2 and not-funded lists are positions, not prejudices, and a bank-published ✅ elsewhere in the industry is a legitimate reason to move a candidate up. **Second, the champion-challenger register** — each funded copilot carries a dated plan to convert its ⚠-external evidence into the bank's own ✅-measured result, which is the only way the P1 wave earns its production badges. **Third, the vendor-prose filter** — every agentic vendor deck that reaches the board is pre-processed with Section 12.3's three moves, so the autonomy prose arrives already separated from the deployment sentence. **Fourth, the not-funded review trigger** — the autonomous rungs are reviewed on a fixed calendar *and* on any of Section 12.2's four developments, so "not funded" never silently becomes "never examined". A portfolio review that ends at the deck is a vendor shortlist in disguise; the agenda is what makes it governance.

---

## 14. The Claims Audit

The claims audit is the guide's own Section 1.3 discipline turned on the guide. Every headline claim above is audited below — ✅ where this pass verified it at a primary or authoritative source, ⚠ where it is carried from press or vendor materials without primary confirmation, ❌ where the evidence contradicts or fails the claim as commonly stated:

| The Claim, as Commonly Stated | The Marker | The Audit |
|---|---|---|
| "Morgan Stanley put GPT-4 in front of its advisors, answering from the firm's own corpus with citations" | ✅ | Bank press release (March 2023) + OpenAI case study — Sections 3.1, 5.1 |
| "98% of Morgan Stanley advisor teams use the assistant" | ⚠ | Press-circulated; not confirmable at a Morgan Stanley primary page — Section 3.1 |
| "Erica has passed 3 billion client interactions and serves ~50 million users" | ✅ | Bank of America newsroom, August 2025 — Section 3.2 |
| "Klarna's AI assistant does the work of 700 full-time agents" | ⚠ | Klarna's own figure, output-equivalence, methodology unpublished — Section 3.3 |
| "Klarna's assistant replaced 700 jobs" | ❌ | The jobs-replacement reading is disputed: output-equivalence, not headcount reduction; Klarna reported continued hiring — Section 3.3 |
| "HSBC's AML AI found 2–4× more true positives with >60% fewer alerts" | ✅ as-published | Joint Google Cloud–HSBC launch release, June 2023; vendor-and-client joint claim, not third-party audited — Section 4.1 |
| "JPMorgan's LLM Suite reached 200,000 employees in eight months" | ✅ | JPMorganChase technology blog, June 2025 — Section 5.2 |
| "Upstart ran 91% of its Q2 2026 loans fully automated" | ✅ | Upstart earnings release, August 2026; platform-specific, not industry-wide — Section 6.1 |
| "JPMorgan's LOXM was the first major-bank AI execution engine, trialled in 2017" | ✅ existence | FT exclusive + corroborating press, 2017 — Section 7.1 |
| "LOXM improved execution by 15%" | ⚠ | Secondary-source figure; not asserted in this guide — Section 7.1 |
| "A major bank runs a fully autonomous LLM trading agent in production" | ❌ | No verified deployment in the record; 2025–2026 stories grade ⚠ pilot — Sections 7.2–7.3 |
| "BloombergGPT outperforms on financial NLP tasks" | ✅ | The model's own paper, arXiv:2303.17564 — Section 8.2 |
| "Aladdin Copilot is available to all Aladdin clients" | ✅ as-published | BlackRock product page; usage counts unpublished — Section 8.3 |
| "The SEC/CFTC recordkeeping-enforcement wave drove communications-surveillance spend" | ✅ | Public regulatory record from 2021 onward — Section 9.1 |
| "Visa Advanced Authorization prevents ~$30 billion of fraud a year" | ⚠ | Visa's own estimate, widely carried, not independently audited — Section 11.1 |
| "Lemonade's AI pays claims with no human involvement" | ⚠ | The automation is real for a subset of claims; the "no humans" advertising drew UK standards scrutiny (2023) — Section 10.1 |
| "Fully autonomous mortgage underwriting is in production at named lenders" | ❌ | No verified deployment in the record; every credible 2025–2026 case is a human-approval copilot — Section 6.4 |
| "Agentic AI will automate most finance back-office work within a few years" | ⚠ | Analyst forecast, not deployment evidence; treat as scenario input — Section 12.3 |
| "CommBank's Ask Ceba staff pilot runs at its claimed scale" | ⚠ | No CommBank primary page found — Sections 3.2, 15.1 |
| "UBS Red serves ~52,000 employees with GA in the first half of 2026" | ⚠ | Swiss-press-reported; no UBS primary page found — Section 5.2 |
| "The e-Trading surveys show institutional traders ranking AI/ML first" | ⚠ | Press-carried results; methodology not published in this pass's record — Sections 7.2, 12.2 |
| "AllianzGPT and the Section 10.2 rollouts are in production at their claimed scale" | ⚠ | Press-documented existence; scale and results not primary-verified — Section 10.2 |

The audit's own summary: **the ✅ claims are the ones the deploying institution itself published; the ⚠ claims are the ones the vendor or the press published on the institution's behalf; the ❌ claims are the places where the marketing reading outran the record — the jobs-replacement arithmetic, the autonomous-trader story, and the autonomous-mortgage story** — and every reader of a finance-AI headline in the next year should check which of the three columns it falls in before repeating it. The same test applies to this guide's own markers: each row above carries the section that the reader can open and check, which is the difference between an audit and an assertion.

**How the markers were assigned, and the audit's own limits.** Three rules governed every marker in this guide, and they are the audit's method as much as its result. (1) **A ✅ requires a primary or authoritative source examined in this pass** — the deploying institution's own release, filing, or page, a regulator's record, or the peer-reviewed paper — not a press echo of one; the one deliberate exception is the ✅-press grade used for LOXM-class cases where the primary record is the reputable press itself and the institution has not maintained a page (Section 7.1). (2) **A ⚠ means unverified-at-primary, not false** — the flag is the finding, and Section 15.1 collects every ⚠ the reader might otherwise re-cite as fact. (3) **A ❌ means the record contradicts or fails the claim as commonly stated** — the Klarna jobs-replacement reading is ❌ not because the assistant is unreal but because the headcount reading is unsupported; the autonomous-trader and autonomous-mortgage stories are ❌ because no verified deployment exists behind the prose. The audit's limits are the same as its method's: it grades the claims this pass examined, on sources accessible to this pass, as of September 2026 — a claim graded ⚠ today may be ✅ tomorrow if the deploying firm publishes, which is precisely why every row names the section where the reader can re-check it. The markers are a snapshot with a re-check date, not a verdict in stone.

---

## 15. What Could Not Be Verified, the Glossary, and the Closing

### 15.1 What Could Not Be Verified

This pass records, honestly and without assertion, the items it could not confirm at a primary or authoritative source within its search budget. None of these are asserted as false — they are **unverified**, and the reader who needs them should treat them as leads, not facts:

- ⚠ The **Ask Ceba** staff-assistant pilot at CommBank (claimed scale circulated in press) — no CommBank primary page found (Section 3.2).
- ⚠ The specific **OCBC and UOB** genAI assistant deployments (2024–2025 press) — not confirmable at primary pages (Section 3.2).
- ⚠ The **"98% of advisor teams"** adoption figure at Morgan Stanley (Sections 3.1, 5.1).
- ⚠ The **LOXM "15% improvement"** execution figure circulating in secondary sources (Section 7.1).
- ⚠ The **Moody's–Hummingbird pairing** referenced in one seed-lead of this project (Section 4.2).
- ⚠ The **"hundreds of live AI use cases"** vintage counts for JPMorgan's fraud estate (Section 4.1).
- ⚠ The **UBS Red** rollout scale and general-availability date (Section 5.2) — Swiss-press-reported.
- ⚠ **AlphaSense and Koyfin client-count claims** (Section 8.1) — vendor-reported, cross-referenced not re-verified.
- ⚠ The **Bloomberg Terminal agentic layer** production scale (Section 8.2) — no Bloomberg primary page describing scale found.
- ⚠ The **named banks** behind the 2025 genAI communications-surveillance triage reporting (Section 9.1) — largely unnamed in public versions.
- ⚠ The **AllianzGPT, Zurich, Ping An, and OneConnect** scale and result figures (Section 10.2) — press- and company-reported, not primary-verified in this pass.
- ⚠ The **"$30 billion"-class** Visa fraud-prevention figure and the **Mastercard dispute-agent** measured results (Sections 11.1–11.2) — self-reported.
- ⚠ The **e-Trading survey methodology and response base** behind the "AI/ML most influential" ranking (Sections 7.2, 8.3, 12.2) — press-carried results without a published methodology in this pass's record.
- ⚠ The **SmartStream and post-trade vendor** measured results at named banks (Section 9.3) — vendor-announced only.
- ⚠ The **analyst-forecast percentages** for finance agentic-AI adoption (Section 12.3) — flagged by construction.
- ⚠ The **Klarna CSAT and resolution-time comparison methodology** behind the FTE-equivalence claim (Section 3.3) — no published controlled methodology found.
- ⚠ The **HSBC 2026 programme outcomes** behind the announced targets (Sections 4.1, 9.3) — announced as targets, not yet measurable as results at the time of writing.
- ⚠ The **Zurich measured claims results** behind the named-executive statements (Section 10.2) — direction confirmed as spoken, numbers not published.
- ⚠ The **exact automated-claims share** at Lemonade (Section 10.1) — the subset boundary between the automated pipeline and human review was never published at primary level.
- ⚠ The **instant-payments screening economics** at named banks (Section 11.3) — the vendor layer markets the models; no named bank publishes the false-positive numbers at instant speed.
- ⚠ The **scheme-rule outcome rates** claimed for the dispute agents of Section 11.2 — win-rate and cycle-time figures circulate in vendor material only.
- ⚠ The **bordereau- and exposure-checking results** claimed for the reinsurance and MGA copilots of Section 10.2 — vendor-announced, buyer-silent.

### 15.2 The Glossary

| The Term | The Meaning, in This Guide's Usage |
|---|---|
| **Agent** | A system that executes multi-step work end-to-end within a bounded scope, with escalation contracts (Section 2.3's third rung) |
| **Assistant** | Answers questions from a curated corpus; reads but does not act (Section 2.3's first rung) |
| **Copilot** | Drafts, summarises, proposes, and explains inside the human's workflow; tool calls with human approval (Section 2.3's second rung) |
| **Bounded autonomy** | The agent acts freely only inside its declared scope; everything outside escalates to a human |
| **General-purpose agent** | A foundation model or platform applied to a task with prompting, retrieval, and guardrails (Section 2.2) |
| **Domain-specific agent** | A purpose-built decision engine, often pre-LLM, that is agentic in behaviour if not in name (Section 2.2) |
| **Hybrid** | Domain-specific decisioning wrapped in an LLM interface — where most 2025–2026 production sits (Section 2.2) |
| **RAG** | Retrieval-augmented generation — grounding the model's answer in retrieved documents |
| **Grounding / citation** | The answer is generated from and linked to the source documents (the Morgan Stanley pattern, Section 3.1) |
| **Hallucination** | Model output not grounded in the retrieved evidence — the core failure the eval gates exist to catch |
| **Eval gate** | The measured-quality checkpoint an agent must clear before promotion (the engineering guide's conventions) |
| **Canary rollout** | Releasing to a small live population first, with measured comparison before wider release |
| **Champion-challenger** | Running the new model against the incumbent on live traffic and promoting only on measured win |
| **Kill switch** | The mechanism to halt or roll back the agent on degraded measured performance |
| **Hypercare** | The intensified monitoring period immediately after production deployment |
| **HITL** | Human-in-the-loop — a human holds approval or decision rights on the money move |
| **Escalation contract** | The declared rule for when the agent must hand the case to a human |
| **Orchestration** | The coordination of multiple agents, tools, and steps in one workflow (the engineering guide's subject) |
| **Tool calling** | The model's ability to invoke a defined external function as one of its actions |
| **FTE-equivalence** | Output measured as "the work of N full-time staff" — the contested marketing metric of Section 3.3 |
| **Model risk management** | The regulatory discipline applied to any model that makes or informs decisions (Sections 6, 10) |
| **Fair lending** | The anti-discrimination overlay on credit decisioning models (Section 6.2) |
| **KYC** | Know-your-customer — the onboarding identity and risk checks (Section 4) |
| **AML** | Anti-money-laundering — transaction monitoring and reporting (Section 4) |
| **Sanctions screening** | Checking payments and parties against OFAC, UN, EU, UK, and MAS lists (Section 11.3; the Fircosoft sibling) |
| **False positive** | An alert the investigation clears — the volume the triage agents exist to cut (Sections 4, 11.3) |
| **STP** | Straight-through processing — the exception-free automated flow (Section 9.3) |
| **Reconciliation / reference data** | Matching records across systems / the shared master data both sides settle against (Section 9.3) |
| **Authorization decision** | The approve/decline call on a payment, made in milliseconds by the pre-LLM engines (Section 11.1) |
| **Chargeback / dispute** | The cardholder-initiated reversal process, scheme-rule-governed (Section 11.2) |
| **Surveillance** | Monitoring communications and trading for market abuse (Section 9) |
| **MAR / MiFID II** | The EU market-abuse and markets-in-financial-instruments regimes (Section 1.2's capital-markets row) |
| **MAS Notice 626/658** | The Singapore AML and technology-risk notices governing the banking row of Section 1.2 |
| **Design fiction** | A hypothetical worked example used to exercise a method — Cymbal Bank's portfolio review (Section 13) |
| **Agentic AI** | The umbrella marketing term for AI that acts; in this guide it resolves into the three rungs of Section 2.3 |
| **Autonomy frontier** | The line past which no verified finance deployment grants the agent the money move (Sections 7, 10, 12.2) |
| **Governed agent** | An agent run under the scope, escalation, evidence, and evaluation contracts — the engineering guide's term, borrowed in Section 2.3 |
| **Primary source** | The deploying institution's own release, filing, page, or paper — the only basis for a ✅ (Section 1.3) |
| **Press-carried** | Reported by the press without the firm's own named statement — ⚠ by default (Section 1.3) |
| **Vendor-marketing grade** | A claim published by the vendor about its own product or economics — ⚠ until the buyer publishes (Section 4.2) |
| **MGA** | Managing general agent — underwrites a niche book on a carrier's paper (Section 10.2) |
| **Representment** | The acquirer's or issuer's re-submission of a disputed transaction under scheme rules (Section 11.2) |
| **Scheme rules** | The network rulebook governing authorization, disputes, and settlement (Section 11) |
| **True positive** | An alert that is a real case — the numerator the AML engines raised (Section 4.1) |

### The References

The primary and named sources cited across this guide, consolidated (all accessed September 2026):

1. Morgan Stanley press release, "Key Milestone in Innovation Journey with OpenAI," March 2023 — Sections 3.1, 5.1.
2. OpenAI case study, "Morgan Stanley uses AI evals to shape the future of financial services" — Sections 3.1, 5.1, 3.3.
3. Bank of America newsroom release, August 20, 2025 (Erica and the enterprise assistant family) — Sections 3.2, 5.2.
4. VentureBeat, January 12, 2024 (Wells Fargo CIO on Fargo, Tachyon, and the Stanford HAI programme) — Section 3.2.
5. Wells Fargo newsroom release, 2026 (Fargo passing 1 billion interactions) — Section 3.2.
6. Commonwealth Bank of Australia newsroom, March 2024 and August 2023 — Section 3.2.
7. DBS Annual Report 2024, "Innovating impactful Solutions for our customers" — Section 3.2.
8. Klarna press release, February 2024 (the AI assistant economics) — Section 3.3.
9. Google Cloud press release, June 21, 2023 (AML AI launch, HSBC and Bradesco) — Sections 4.1, 12.1.
10. HSBC media release, June 17, 2026 (the HSBC–Google Cloud agentic partnership) — Sections 4.1, 8.1, 9.3.
11. Quantexa announcements (Danske Bank), carried by industry press — Section 4.2.
12. JPMorganChase technology blog, June 3, 2025 (LLM Suite) — Sections 5.2, 7.2.
13. Microsoft customer story and ubs.com, "Innovation and AI at UBS" — Section 5.2.
14. finews and Swiss financial press, 2025 (UBS Red) — Section 5.2.
15. Upstart Holdings Q2 2026 earnings release, ir.upstart.com, August 4, 2026 — Section 6.1.
16. Financial Times, 2017 (LOXM); Business Insider, 2017 ("JPMorgan Takes AI Use to the Next Level") — Section 7.1.
17. Wu, Irsoy, Lu, Dredze, Gehrmann, Kambadur, Rosenberg, Mann, "BloombergGPT: A Large Language Model for Finance," arXiv:2303.17564, March 2023 — Section 8.2.
18. blackrock.com, Aladdin Copilot product page — Section 8.3.
19. Bloomberg Vault announcement, 2025; Global Relay, State of AI in Surveillance survey — Section 9.1.
20. The Banker, 2025 (genAI communications-surveillance triage) — Section 9.1.
21. SmartStream and post-trade vendor announcements, 2025–2026 — Section 9.3.
22. Lemonade press and coverage, 2017–2023, including the UK advertising-standards scrutiny reporting — Section 10.1.
23. Press coverage of AllianzGPT, Zurich, Ping An, and OneConnect, 2023–2026 — Section 10.2.
24. Visa company materials on Visa Advanced Authorization; Mastercard company materials on Decision Intelligence — Sections 2.2, 11.1–11.2.
25. Zest AI press releases, 2024 (lending-decision counts) — Section 6.2.
26. Feedzai, Nasdaq, Moody's, and Hummingbird product and marketing pages — Sections 4.2, 9.2.
27. Shift Technology and Tractable vendor materials — Section 10.2.
28. The JPMorgan e-Trading survey series (results press-carried) — Sections 7.2, 8.3, 12.2.
29. Reuters, August 2024 (LLM Suite early phase) — Section 5.2.
30. The sibling guides: [Multi-Agent AI Systems for Banking](./multi_agent_banking_guide.md), [LLM Agent Use Cases](../llm_agent_use_cases.md), [AI Governance, Bias & Red-Teaming](ai_governance_bias_redteaming_guide.md), [LLM Agents Failures in Production](llm_agents_failures_production_guide.md), [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md), and [Fircosoft — the AML/Sanctions Screening Engine](../banking/fircosoft_guide.md) — cross-referenced throughout.

### 15.3 The Closing

The landscape, in one view: the finance industry has put the **copilot** in production at a scale no other regulated industry matches — bank-published usage in the billions of interactions, firmwide platforms in the hundreds of thousands of users, and a verified record that the assistant and copilot rungs are where the ✅ evidence lives. The **agent** rung is real but bounded: fraud triage, AML screening, post-trade exception handling, and the pre-LLM decision engines that were agentic before the word existed. The **autonomous** rung — the agent that holds the credit decision, the trade, the claim payment, or the screening release — has no verified production deployment in this record, and the regulatory analysis says that is equilibrium, not lag. The vendor prose will keep blurring the three rungs, the FTE-equivalence arithmetic will keep circulating, and the autonomous-trader headline will keep returning; the Section 1.3 markers — ✅ verified at primary, ⚠ press- or vendor-carried, ❌ disputed — are the reader's defence, and the engineering guide's eval gates are the buyer's. Read every finance-AI claim through both, and the pattern is clear: each use case in this guide is a governed step toward the finance agent.
