# Frontier and Catastrophic AI Risk: Frameworks, Evaluations, Governance, and What a Bank Is Actually Exposed To

*A deep-research guide to how the frontier labs, the regulators and the national AI safety institutes reason about risk at capability levels nobody has yet measured — the Responsible Scaling Policy and AI Safety Levels, the OpenAI Preparedness Framework, the Google DeepMind Frontier Safety Framework and Critical Capability Levels, the Meta and Microsoft instruments, the dangerous-capability evaluations behind all of them, the EU AI Act's general-purpose-model provisions, the institutes and summit sequence — and an honest account of what those instruments are actually worth to an enterprise that depends on their results. Ending with the thesis: a scaling policy is a commitment about a model that does not exist yet.*

Jack Liu Shurui, Solution Architect

> **What this guide is — and where it sits.** This is the repository's guide to the **frontier-and-catastrophic-risk layer** of AI: the voluntary safety frameworks that the labs publish about themselves, the dangerous-capability evaluations those frameworks point at, the governance layer (the EU AI Act's GPAI provisions, the national AI safety institutes, the summit sequence), the critiques and the counter-critiques, and the honest scoping question of what an enterprise — specifically a bank — is genuinely exposed to when it depends on a frontier model it did not build and cannot assess.
>
> It deliberately does **not** re-derive what its siblings already own:
>
> - **The enterprise responsible-AI landscape** — corporate frameworks, the banking angle — lives in [../responsible_ai_frameworks_guide.md](../responsible_ai_frameworks_guide.md) (§2 corporate frameworks, §6 banking). This guide does not repeat it.
> - **The governance-and-operating-model layer** — NIST AI RMF, ISO/IEC 42001, the EU AI Act's high-risk regime, the OECD Principles, Singapore's IMDA frameworks, MAS FEAT, the three lines of defence, the committee lattice, the regulatory mapping for a bank — lives in [./ai_governance_framework_guide.md](./ai_governance_framework_guide.md) (§2 global instruments, §3 Singapore, §5 three lines of defence, §9 banking regulatory mapping). This guide only touches the EU AI Act where the **general-purpose-model (GPAI)** provisions cross into frontier risk; the high-risk-system machinery is owned there.
> - **Model-development risks and security engineering** live in [../llm_development_risks_security_guide.md](../llm_development_risks_security_guide.md).
> - **Red-teaming practice** lives in [./ai_red_teaming_guide.md](./ai_red_teaming_guide.md) and [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md); this guide discusses the *evaluation* layer at the frontier-risk level, not the operational red-team playbook.
> - **The consolidated failure-mode register for a bank** — the enterprise-side catalogue of what actually goes wrong — is [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md), a companion guide written in parallel. This guide cross-references it by name and does **not** duplicate its register.
>
> **Verification discipline.** Every framework name, version, date, tier label and threshold in this guide was checked against the issuing body's own published document during this pass (October 2026): anthropic.com and the RSP PDFs, cdn.openai.com and the Preparedness Framework PDF, deepmind.google and the Frontier Safety Framework PDF, the Microsoft framework PDF, artificialintelligenceact.eu and the European Commission's digital-strategy pages, gov.uk, meti.go.jp, nist.gov, aisi.gov.uk, elysee.fr and internationalaisafetyreport.org. Where a claim rests on a secondary index or could not be re-confirmed (notably Meta's framework documents, which the issuing site would not serve), it is marked ⚠ inline and again in **[§14 The Claims Audit](#14-the-claims-audit)** and **[§15 What Could Not Be Verified](#15-what-could-not-be-verified-and-the-glossary)**. This guide asserts **no** position on how dangerous any model or capability actually is, and makes no capability claim about any named model beyond what that model's own publisher has published.

**Contents**

1. [Overview: The DECODER, the Thesis and the Scope](#1-overview-the-decoder-the-thesis-and-the-scope)
2. [The Risk Taxonomy — and Why It Is Contested](#2-the-risk-taxonomy--and-why-it-is-contested)
3. [The Frontier Safety Frameworks](#3-the-frontier-safety-frameworks)
4. [The Dangerous-Capability Evaluations](#4-the-dangerous-capability-evaluations)
5. [The Governance Layer](#5-the-governance-layer)
6. [The Critiques and the Counter-Critiques](#6-the-critiques-and-the-counter-critiques)
7. [What an Enterprise Is Actually Exposed To](#7-what-an-enterprise-is-actually-exposed-to)
8. [What a Bank Can and Cannot Verify](#8-what-a-bank-can-and-cannot-verify)
9. [The Model-Supply-Chain and Change-Record Question](#9-the-model-supply-chain-and-change-record-question)
10. [The Frameworks' Own Failure Modes](#10-the-frameworks-own-failure-modes)
11. [Reading the Primary Sources](#11-reading-the-primary-sources)
12. [The Cymbal Bank Angle](#12-the-cymbal-bank-angle)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified and the Glossary](#15-what-could-not-be-verified-and-the-glossary)
16. [The Cross-References and the Closing Summary](#16-the-cross-references-and-the-closing-summary)

---

## 1. Overview: The DECODER, the Thesis and the Scope

### 1.1 The thesis

**A scaling policy is a commitment about a model that does not exist yet.** That single sentence is the whole subject of this guide. Everything else — the tier labels, the thresholds, the evaluations, the institutes, the summit declarations — is an attempt to answer a deceptively hard question: *how do you make a binding-feeling promise about a risk you cannot yet measure, about a system you have not yet built, in a field whose rate of change outpaces the science used to reason about it?*

The subject of this guide is therefore **not** "AI is dangerous." That is a claim about the world, and this guide asserts no position on it. The subject is the **machinery of reasoning** that the frontier labs, the regulators and the national institutes have built to manage that possibility — what the instruments are, who issues them, what they commit to, where they have been revised, and what an enterprise that merely *consumes* the resulting models is actually exposed to.

### 1.2 The DECODER

Frontier-risk writing is dense with terms that are used loosely and inconsistently. Before the substance, pin them down. Each definition below is written to be accurate to how the issuing bodies use the term, not to a dictionary ideal.

| Term | Precise working definition |
|---|---|
| **Frontier model** | A general-purpose AI model at or near the current capability frontier — the most capable models of their generation, typically large foundation models. Note immediately: **"frontier" is not one definition but several.** A lab's internal threshold, a statute's scope rule, and a journalist's shorthand are different instruments (see §5.4). |
| **Frontier lab** | A developer that trains frontier models at the scale and cost where the capability-threshold literature applies — Anthropic, OpenAI, Google DeepMind, Meta, Microsoft, xAI, Amazon and a handful of others (see §3.10 for the full published set). |
| **Capability threshold** | A pre-specified level of a *capability* (e.g. the ability to uplift a non-expert toward producing a biological weapon) at which the developer judges that a model would pose a meaningfully increased risk of severe harm absent additional safeguards. It is a **capability** proxy for **risk**, not a risk estimate in itself. |
| **Safety framework** | A developer's published document specifying: the risk domains it tracks, the capability/risk thresholds it uses, the evaluations that detect approach to those thresholds, the safeguards triggered at each threshold, and the conditions under which it would halt development or deployment. Also called a *frontier safety policy* or a *scaling policy*. |
| **Scaling policy** | A safety framework organised around **capability scaling** — the commitment that as capability increases past defined levels, safeguards increase in step, and that the developer will pause or hold if the safeguards are not ready. The original example is Anthropic's Responsible Scaling Policy. |
| **Capability level** | A named tier inside a framework (e.g. "ASL-3", "High capability", "Critical Capability Level"). The naming is framework-specific; **tier labels are not interchangeable across labs** and must never be quoted as if they were. |
| **Dangerous-capability evaluation** | A model evaluation designed to elicit and measure a capability associated with severe harm (chemical/biological, cyber, autonomy, persuasion). It measures *capability*, not *deployment risk*. |
| **Elicitation gap** | The gap between what a model *can* do when maximally elicited (best-of-N sampling, chain-of-thought prompting, scaffolding, fine-tuning, tool access) and what a routine evaluation *does* measure. A model that passes an evaluation has not thereby been shown safe; it has been shown not to trip *that* evaluation at *that* elicitation effort (see §4.3). |
| **Safety case** | An explicit, structured argument that a system is safe enough for a given deployment context, backed by evidence, with the assumptions and residual uncertainty made visible. Both DeepMind and Microsoft use safety-case framing; a safety case is an argument, not a certificate. |
| **Systemic-risk classification** | The EU AI Act's status for a general-purpose AI model whose capabilities or impact are high enough to warrant the Act's most demanding GPAI obligations. It is defined by Article 51 and Annex XIII, with a computable presumption (see §5.1). |
| **Institute** | A national (or EU) body — the UK AI Security Institute, the US NIST centre, Japan's AISI, the EU AI Office — tasked with evaluating advanced AI and informing policy. Crucially, what each is **empowered** to do differs enormously (see §5.2). |

### 1.3 The scope statement

This guide covers the **frontier-risk literature**: the lab safety frameworks, the dangerous-capability evaluations, the governance layer aimed at the frontier, the critiques of that layer, and the enterprise exposure that follows from *depending* on a frontier model.

It does **not** cover, and cross-references instead:

- enterprise AI-risk management proper (the failure modes that actually occur in deployed systems) — owned by [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md);
- the governance framework stack and operating model — owned by [./ai_governance_framework_guide.md](./ai_governance_framework_guide.md);
- the enterprise responsible-AI landscape — owned by [../responsible_ai_frameworks_guide.md](../responsible_ai_frameworks_guide.md);
- red-teaming and bias-measurement practice — owned by [./ai_red_teaming_guide.md](./ai_red_teaming_guide.md) and [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md).

One structural point frames every later section: **a capability threshold is not a deployment control.** A threshold is a statement about when a *developer* would change what *it* does. It says nothing about what a *deployer* — a bank — has configured, monitored, or can fall back to. Those are different instruments serving different parties, and conflating them is the single most common category error in enterprise frontier-risk work.

---

## 2. The Risk Taxonomy — and Why It Is Contested

### 2.1 The four families, presented as plural

There is no single agreed taxonomy of frontier AI risk. What follows is a description of the *families of divisions the field actually uses*, not a claim that the field has settled on them. Different labs and different governments slice these differently and do not agree on the boundaries.

**Family 1 — Deliberate misuse.** A human actor uses the model to cause harm: chemical, biological, radiological or nuclear (CBRN) uplift; offensive cyber operations; fraud and social engineering. This is the family on which the lab frameworks have the most concrete thresholds.

**Family 2 — Loss of control / misalignment.** A model acts in ways contrary to its designers' intent, including deceptive or self-reinforcing behaviour, or undermining effective human control. Microsoft's framework names a tracked capability "**loss of control**": a model's ability to undermine effective human control through adaptive, deceptive, or self-reinforcing mechanisms "such that, when deployed, a model can no longer be reliably directed, modified, or shut down." DeepMind's v3 framework expanded its machine-learning R&D critical-capability-level work toward scenarios where misaligned models might interfere with operators' ability to direct, modify or shut down their operations.

**Family 3 — Accident and systemic failure.** Failures that are not adversarial: model error at scale, correlated behaviour across many deployments, infrastructure and supply-chain fragility, evaluation gaming. This is the family least addressed by capability thresholds and most addressed by ordinary enterprise risk management.

**Family 4 — Socio-structural and societal concerns.** Concentration of power, labour-market effects, information integrity, privacy, human autonomy. The Seoul Summit framing explicitly includes "societal harms"; the UK's pre-summit discussion papers place "misuse, loss of control and societal harms" side by side. National institute research agendas (e.g. the UK AI Security Institute's "societal resilience" and "human influence" strands) sit here.

### 2.2 Where the slicing disagrees — and why it matters

The boundaries are genuinely contested, and the disagreements are load-bearing:

- **Misuse vs. misalignment.** DeepMind's *initial* framework (v1.0, May 2024) put "Autonomy" and "Machine Learning R&D" in the same CCL table as Biosecurity and Cybersecurity, treating them as capability domains. Later versions (v3.0, September 2025) explicitly separate the *misuse* risk from the *misalignment* risk arising from "undirected action at these capability levels." The same capability, filed under different families, triggers different mitigations.
- **Capability vs. risk thresholds.** The Frontier Model Forum's November 2024 issue brief notes that capability thresholds have "largely been used … as a proxy for risk to date," while "risk thresholds may also be defined using likelihood or severity estimates of specific risks." Two frameworks can look identical and be measuring different things.
- **What counts as "severe."** OpenAI's Preparedness Framework defines "severe harm" for its purposes as "the death or grave injury of thousands of people or hundreds of billions of dollars of economic damage." That is a deliberate high bar, stated in the document; other frameworks leave severity implicit. A framework that never states its severity bar cannot be compared against one that does.

**The practical consequence for an enterprise.** A bank does not get to choose which taxonomy its provider uses. It gets whatever taxonomy that provider published — if any. The honest reading is that the taxonomies are **plural, framework-specific, and not harmonised**, and any enterprise control that assumes they are interchangeable is built on sand.

---

## 3. The Frontier Safety Frameworks

This is the centrepiece of the guide. It describes each published frontier safety framework at the level of the features that matter for risk reasoning: who publishes it, when, its current version, the **names** of its capability tiers, what triggers a tier, and what the developer commits to when a threshold is reached.

**Three properties hold across essentially all of them, and they are descriptions of the instrument's design, not accusations.** First, they are **voluntary** — adopted by the developer, not imposed. Second, their thresholds are **self-set** — chosen by the developer, not by an external body. Third, they are **self-assessed** — the developer runs the evaluations on its own models. Any framework fact below should be read with those three properties in mind. (The critiques and the counter-critiques of this design are in §6, presented in both directions.)

### 3.1 Anthropic — Responsible Scaling Policy (RSP) and AI Safety Levels (ASL)

| Item | Value (verified at source) |
|---|---|
| Framework name | **Responsible Scaling Policy (RSP)** |
| Issuing body | Anthropic |
| First version | **v1.0, effective 19 September 2023** (announced the same day on anthropic.com) |
| Current version | **v3.4, effective 8 July 2026** |
| Full version chain | v1.0 (19 Sep 2023) → v2.0 (15 Oct 2024) → v2.1 (31 Mar 2025) → v2.2 (14 May 2025) → v3.0 (24 Feb 2026) → v3.1 (2 Apr 2026) → v3.2 (29 Apr 2026) → v3.3 (26 May 2026) → v3.4 (8 Jul 2026) |
| Tier label | **AI Safety Levels (ASL)**, from ASL-1 upward |
| Source | anthropic.com/responsible-scaling-policy (page "Last updated Aug 14, 2026"); v3.0 announcement page anthropic.com/news/responsible-scaling-policy-v3 (24 Feb 2026) |

**The ASL tiers, as stated in the original 19 September 2023 announcement:**

- **ASL-1** — systems that "pose no meaningful catastrophic risk," e.g. a 2018 LLM or a chess engine.
- **ASL-2** — systems showing "early signs of dangerous capabilities" (the original example: ability to give instructions on building bioweapons) but where the information is "not yet useful" because of insufficient reliability or because a search engine could surface it. The original document states that current LLMs, *including Claude*, appeared to be ASL-2 at that time.
- **ASL-3** — systems that "substantially increase the risk of catastrophic misuse compared to non-AI baselines," **or** that show low-level autonomous capabilities.
- **ASL-4 and higher (ASL-5+)** — not defined at first publication, "too far from present systems," described as likely to involve "qualitative escalations in catastrophic misuse potential and autonomy."

**How the RSP evolved, and why (this is the revision history that matters).** The 2023 original was, in Anthropic's own later words, an "early iteration" with a commitment to "rapid iteration and course correction." The revisions are documented on the policy page and are instructive:

- **v2.0 (15 Oct 2024)** — the first substantial update. Anthropic's own retrospective on the page lists concrete instances where it "fell short of meeting the full letter of its requirements": evaluations completed three days late; an autonomy evaluation changed from specified placeholder tasks; some evaluations lacking "basic elicitation techniques such as best-of-N or chain-of-thought prompting"; and one domain where evaluations could not establish the intended 6× "scaling buffer." Anthropic extended the evaluation interval from a 3-month to a 6-month cadence and stated these gaps "posed minimal risk to the safety of our models."
- **v2.1 (31 Mar 2025)** — clarified which capability thresholds require safeguards beyond ASL-3; **added a new CBRN-development capability threshold** aimed at capabilities that "could substantially uplift the development capabilities of moderately resourced state programs"; and **disaggregated the AI R&D thresholds into two distinct levels** (fully automating entry-level AI research work, and causing "dramatic acceleration in the rate of effective scaling").
- **v2.2 (14 May 2025)** — a minor revision amending a footnote to exclude both sophisticated insiders and state-compromised insiders from the ASL-3 Security Standard scope.
- **v3.0 (24 Feb 2026)** — a **comprehensive rewrite**, introducing published **Frontier Safety Roadmaps** and **Risk Reports** quantifying risk across deployed models. Anthropic's own assessment of its "theory of change" in the v3 announcement is unusually candid and is a primary source for the critiques in §6: it states that the "race to the top" and internal forcing-function mechanisms "did play out," but that the hoped-for "consensus about risks" effect **"did not play out in practice,"** because pre-set capability levels were "far more ambiguous than we anticipated" and "the science of model evaluation isn't well-developed enough to provide dispositive answers." It also states that government action on AI safety "has moved slowly."
- **v3.1–v3.4 (Apr–Jul 2026)** — v3.1 clarified the AI R&D capability threshold definition (that "doubling the rate of progress" meant doubling aggregate AI capability progress, not doubling researcher productivity) and clarified that Anthropic remains free to pause development beyond what the RSP requires. v3.3 revised the novel chemical/biological weapons threshold; v3.4 revised the automated R&D threshold and adjusted public Risk Report redaction and review rules.

**What the developer commits to when a threshold is reached.** The commitment is the **if-then** structure: if a model crosses a specified capability threshold, then a defined, stricter set of safeguards applies. The 2023 announcement states that ASL-3 measures include "a commitment not to deploy ASL-3 models if they show any meaningful catastrophic misuse risk under adversarial testing by world-class red-teamers," in contrast to "merely a commitment to perform red-teaming." It also states that the framework "implicitly requires us to temporarily pause training of more powerful models if our AI scaling outstrips our ability to comply with the necessary safety procedures." Anthropic states it activated ASL-3 safeguards for relevant models in May 2025.

### 3.2 OpenAI — Preparedness Framework

| Item | Value (verified at source) |
|---|---|
| Framework name | **Preparedness Framework** |
| Issuing body | OpenAI |
| First version | **Preparedness Framework (Beta), December 2023** |
| Current version | **v2.0, "Last updated: 15th April, 2025"** |
| Tier label | Capability thresholds: **High capability** and **Critical capability** |
| Source | cdn.openai.com preparedness-framework-v2.pdf (the framework document itself); framework index at metr.org/fsp |

**Tracked Categories (the risk domains).** The v2.0 document names three **Tracked Categories**:

1. **Biological and Chemical** capabilities — those that "can also reduce barriers to creating and using biological or chemical weapons."
2. **Cybersecurity** capabilities — those that "can also create new risks of scaled cyberattacks and vulnerability exploitation."
3. **AI Self-improvement** capabilities — those that "could also create new challenges for human control of AI systems."

v2.0 also **introduced a set of "Research Categories"** — areas of capability that "do not yet meet our criteria to be Tracked Categories," where OpenAI says it is investing to develop threat models and elicitation techniques.

**The tier labels and what triggers them.** v2.0 streamlined the levels to **two thresholds**: **High capability**, described as a level that "could amplify existing pathways to severe harm," and **Critical capability**, described as one that "could introduce unprecedented new pathways to severe harm." The document states the commitment in conditional form: OpenAI "will not deploy these very capable models until we've built safeguards to sufficiently minimize the associated risks of severe harm," and "If a model under development reaches a Critical capability threshold, we also require safeguards to sufficiently minimize the associated risks during development, irrespective of deployment plans."

**Governance inside the framework.** v2.0 names an internal, cross-functional group called the **Safety Advisory Group (SAG)** that oversees the framework and recommends safeguards; OpenAI Leadership "can approve or reject these recommendations," and the Board's **Safety and Security Committee** provides oversight. This internal-approval structure is precisely what §6's critiques target, and precisely what §6's counter-critiques defend as an auditable governance event. Both readings are of the *same* paragraph.

**Related instruments.** METR's policy index lists a further OpenAI document, **"Frontier Governance Framework" (May 2026)**. That document has **not** been verified at source in this pass and is not relied on below.

### 3.3 Google DeepMind — Frontier Safety Framework (FSF) and Critical Capability Levels (CCLs)

| Item | Value (verified at source) |
|---|---|
| Framework name | **Frontier Safety Framework (FSF)** |
| Issuing body | Google DeepMind |
| First version | **v1.0, 17 May 2024** (blog + technical report) |
| Version chain | v1.0 (May 2024) → v2.0 (Feb 2025) → v3.0 (22 Sep 2025) → **v3.1 (17 April 2026)** |
| Tier labels | **Critical Capability Levels (CCLs)**; from v3.1 also **Tracked Capability Levels (TCLs)**; plus mitigation levels |
| Source | deepmind.google blog posts (17 May 2024; "Strengthening our Frontier Safety Framework", 22 Sep 2025, updated 17 Apr 2026) and the FSF PDFs, including frontier-safety-framework_3-1.pdf |

**The CCL concept.** DeepMind defines CCLs as "capability levels at which, absent mitigation measures, frontier AI models or systems may pose heightened risk of severe harm." The framework's three core components, as stated in the v1.0 technical report, are: (1) identify capability levels at which models pose heightened risk without mitigation; (2) implement protocols to detect the attainment of such levels via "**early warning evaluations**"; and (3) prepare mitigation plans in advance.

**Risk domains.** v1.0 (May 2024) defined CCLs across four domains: **Autonomy, Biosecurity, Cybersecurity, and Machine Learning R&D**. Concrete CCL names in v1.0 included **"Autonomy level 1"** (the ability to autonomously expand effective capacity by acquiring resources and running copies of itself on rented hardware) and **"Machine Learning R&D level 2"** (fully automating the AI R&D pipeline at a fraction of human labour costs). v3.0 (Sep 2025) **added a harmful-manipulation CCL** — models with manipulative capabilities that could "systematically and substantially change beliefs and behaviors in identified high stakes contexts" — and expanded the misalignment framing. v3.1 (17 Apr 2026) **added Tracked Capability Levels (TCLs)** in certain domains, "a new capability level to help us spot and evaluate potential less extreme risks sooner."

**Mitigation levels.** The v1.0 technical report defines parallel mitigation ladders: **Security Levels 0–4** (from "status quo" through "advanced protection," cross-referenced to RAND's L3–L5 security levels) and **Deployment Levels 0–3** (from "status quo" through "prevention of access," with Level 2 framed as "safety case with red team validation"). The document states that where a model "may reach evaluation thresholds before mitigations at appropriate levels are ready," DeepMind "would put on hold further deployment or development."

**Cadence.** The v1.0 report states the aim to evaluate "every 6× in effective compute and for every 3 months of fine-tuning progress," with early-warning evaluations designed to pass "before the model is 6× effective compute or 3 months of fine-tuning away from the CCL."

**Safety cases.** From v3.0, DeepMind states it conducts **safety case reviews** prior to external launches when relevant CCLs are reached — "detailed analyses demonstrating how risks have been reduced to manageable levels" — and expanded this to large-scale internal deployments for advanced ML R&D CCLs.

### 3.4 Meta — Frontier AI Framework / Advanced AI Scaling Framework

⚠ **This is the section where this guide falls short of its own standard, and says so.** Meta's framework pages were **not retrievable at source** during this pass: `ai.meta.com/blog/meta-frontier-ai-framework/` returned "page isn't available"; `about.fb.com/news/2025/02/meta-frontier-ai-framework/` returned 404; `ai.meta.com/static-resource/meta-frontier-ai-framework/` returned an error page; and an Internet Archive snapshot returned an empty document body. Meta's **tier labels are therefore UNVERIFIED** and are not asserted here.

What *is* recordable, and is flagged as sourced from the **METR frontier-safety-policy index** (a secondary but careful index, not Meta's own page):

| Item | Value | Status |
|---|---|---|
| Framework name (v1) | "Meta Frontier AI Framework," v1.0, February 2025 | ⚠ secondary index |
| Framework name (v2) | "Meta Advanced AI Scaling Framework," v2.0, April 8, 2026 | ⚠ secondary index |
| Tier labels | **not verified** | ❌ unverified |

An enterprise should treat "Meta's framework says X about tier Y" as unverified until read at Meta. This is exactly the discipline HAZARD 1 demands, and it is more useful stated plainly than papered over.

### 3.5 Microsoft — Frontier Governance Framework

| Item | Value (verified at source) |
|---|---|
| Framework name | **Frontier Governance Framework** |
| Issuing body | Microsoft |
| First version | **v1.0, February 2025** |
| Current version | **February 2026** (one-year update) |
| Tier label | Risk classification: **low, medium, high, critical** |
| Source | Microsoft "Frontier Governance Framework" PDF (February 2026), change log at Appendix II |

Microsoft's framework states its genesis in the **voluntary Frontier AI Safety Commitments that Microsoft and fifteen other AI labs made in May 2024** (the Seoul commitments; see §5.3). It tracks five high-risk capabilities:

1. **CBRN weapons** — a model's ability to "provide significant capability uplift to an actor seeking to develop and deploy a chemical, biological, radiological, or nuclear weapon."
2. **Offensive cyberoperations** — significant uplift toward "highly disruptive or destructive cyberattacks, including on critical infrastructure."
3. **Advanced autonomy** — the ability to complete expert-level tasks autonomously, "including AI research and development."
4. **Loss of control** — the ability to "undermine effective human control through adaptive, deceptive, or self-reinforcing mechanisms" such that a model "can no longer be reliably directed, modified, or shut down."
5. **Harmful manipulation** — the ability to "strategically distort human behavior or beliefs at scale."

**Tiers and triggers.** Microsoft assesses its most advanced models for signs of these capabilities and, if present, asks whether the capability "poses a low, medium, high, or critical risk to national security or public safety." The framework's Appendix I gives per-domain capability thresholds; e.g. under **Advanced autonomy**, "Low" is "software engineering tasks that take a human less than one hour," "High" is autonomously completing tasks "equivalent to multiple days' worth of generalist human labor," and "Critical" is fully automating the AI R&D pipeline at a fraction of human labour costs. Deployment requirements attach to the tier: low/medium are "deployment allowed in line with Responsible AI Program requirements"; high/critical require "further review and mitigations."

**Cadence and scope.** A **leading indicator assessment** runs on any model in scope for frontier-model requirements "under applicable laws, such as the EU AI Act, California's Transparency in Frontier AI Act (TFAIA), and New York's Responsible AI Safety and Education (RAISE) Act," and when Microsoft "substantially fine-tunes" a model where fine-tuning compute exceeds **one-third of the base model**. It runs during pre-training, after pre-training, after post-training, and prior to deployment, and in-scope models undergo it at least **every six months**. The February 2026 change log records that the update added **harmful manipulation and loss of control** as tracked capabilities and aligned scope and cadence with the EU GPAI Code of Practice, NY RAISE and CA TFAIA.

### 3.6 The rest of the published set

METR's index records a broader set of published frontier safety policies. **These are listed by name, issuer and date as recorded by that index; they were not re-verified at each issuing body in this pass**, and are flagged accordingly. The purpose of listing them is to make a point that enterprise readers routinely miss: **the frontier-safety-framework landscape is wider than the five or six labs a bank usually thinks of.**

| Developer | Framework (per METR index) | Date | Verified at source? |
|---|---|---|---|
| Amazon | Frontier Model Safety Framework | 10 Feb 2025 | ⚠ not verified at source |
| xAI | Front. AI Framework v2.0 (Dec 2025); Risk Mgmt Framework v1.0 (Aug 2025) | 2025–2026 | ⚠ not verified at source |
| NVIDIA | Frontier AI Risk Assessment | 17 Feb 2025 | ⚠ not verified at source |
| G42 | Frontier AI Safety Framework | 6 Feb 2025 | ⚠ not verified at source |
| Cohere | Secure AI Frontier Model Framework | 7 Feb 2025 | ⚠ not verified at source |
| Magic | AGI Readiness Policy v1.0 | 2 Jul 2024 | ⚠ not verified at source |
| NAVER | AI Safety Framework | 7 Aug 2024 | ⚠ not verified at source |

METR's "**Common Elements of Frontier AI Safety Policies**" analysis (December 2025 version) states that **twelve companies had published** frontier safety policies as of that document, naming Anthropic, OpenAI, Google DeepMind, Magic, Naver, Meta, G42, Cohere, Microsoft, Amazon, xAI and NVIDIA. It reports that **capability thresholds** appear in 9 of the 12, **model weight security** in 11, **deployment mitigations** in 12, and **accountability** mechanisms in 12. That distribution is itself a finding: *halting conditions and elicitation commitments are the rare elements, not the universal ones.*

### 3.7 What the frameworks share — and where they diverge

The Frontier Model Forum's issue brief (8 November 2024) proposes five core components of a safety framework: **risk identification; capability and risk thresholds; capability and risk assessment; risk mitigation; and risk governance**. It notes that "there is not broad consensus yet about what these risk domains might or should be."

Four divergences matter for anyone comparing frameworks:

- **Two thresholds or four?** OpenAI v2.0 trimmed to High/Critical; Microsoft uses four levels (low/medium/high/critical). The words do not map.
- **Misuse-focused or misalignment-inclusive?** DeepMind's later versions and Microsoft's 2026 update explicitly admit loss of control / misalignment; some earlier frameworks were narrower.
- **Quantitative benchmarks or qualitative thresholds?** METR's analysis observes that xAI's and Magic's policies "heavily emphasise quantitative benchmarks," unlike most others.
- **Named tiers or none?** Anthropic (ASL), DeepMind (CCL/TCL) and OpenAI (High/Critical) name their levels; others use descriptive thresholds without a brand name. **A tier label from one framework is not a tier label from another**, and never should be quoted as though it were.

---
## 4. The Dangerous-Capability Evaluations

### 4.1 What a dangerous-capability evaluation is

A dangerous-capability evaluation measures whether a model can perform a task associated with severe harm: producing synthesis instructions for a dangerous agent, writing working exploit code against a real target, autonomously acquiring resources, or persuasively shifting a person's beliefs. The frameworks in §3 are, in large part, *pointers to evaluations*: the threshold is stated as a capability, and the evaluation is the instrument that detects approach to it.

The four evaluation domains that recur across the published frameworks:

| Domain | What is elicited | Where it appears |
|---|---|---|
| **Chemical / biological / radiological / nuclear (CBRN)** | Uplift to an actor attempting to develop or deploy a CBRN weapon; biological knowledge and design capability | Anthropic (CBRN threshold added v2.1); OpenAI ("Biological and Chemical"); DeepMind (Biosecurity); Microsoft (CBRN); Amazon, G42 |
| **Cyber** | Ability to find and exploit vulnerabilities, develop end-to-end exploits, conduct scaled attacks | OpenAI ("Cybersecurity"); DeepMind (Cybersecurity); Microsoft (offensive cyberoperations); UK AISI/US evaluations |
| **Autonomy and self-direction** | Long-horizon autonomous task completion; self-replication and resource acquisition; AI R&D acceleration | DeepMind (Autonomy, ML R&D CCLs); Anthropic (AI R&D thresholds); OpenAI ("AI Self-improvement"); Microsoft (advanced autonomy) |
| **Persuasion and manipulation** | Capacity to systematically and substantially change beliefs or behaviour at scale | DeepMind (harmful-manipulation CCL, v3.0); Microsoft (harmful manipulation, 2026 update) |

### 4.2 How an evaluation is elicited

Capability evaluation is not a single test; it is a *maximisation problem*. To find out what a model can do, evaluators deliberately try to get the most out of it: prompt engineering, **best-of-N sampling** (generate N attempts and take the best), **chain-of-thought prompting**, scaffolding with tools and memory, and sometimes fine-tuning. The point is to avoid *understating* capability.

Anthropic's own v2.0 retrospective names the failure directly: some of its evaluations had "lacked some basic elicitation techniques such as best-of-N or chain-of-thought prompting," and the firm said it is now "systematically tracking these gaps to avoid future under-elicitation." DeepMind's v1.0 technical report lists "**Capability elicitation**" as explicit future work: "We are working to equip our evaluators with state of the art elicitation techniques, to ensure we are not underestimating the capability of our models." The framing is mutual: the labs know that an under-elicited evaluation produces a false negative, and they say so in their own documents.

### 4.3 The elicitation gap — why a failed evaluation is not a safety result

Here is the sentence that most enterprise readers get wrong: **a model that fails a dangerous-capability evaluation has not thereby been shown safe.** It has been shown that *this evaluation, at this level of elicitation effort, did not trip.* The gap between that and "the model cannot do the dangerous thing" is the **elicitation gap**, and it is unreduced by a passing result. Three reasons:

1. **Passing an easy evaluation is easier than failing a hard one.** If the evaluation is too weak, the model passes because the test was weak, not because the capability is absent. Recall the "safetywashing" finding (§6.2): safety benchmarks can correlate with general capability, so a stronger model can appear "safer" or "more capable" depending on what is being measured.
2. **Capability can be latent until elicited.** A capability that requires scaffolding or fine-tuning may be present but undetected by a prompt-only evaluation.
3. **Evaluation gaming is a real failure mode.** METR's research includes the **MALT** dataset of "natural and prompted behaviors that threaten evaluation integrity, like generalized reward hacking or sandbagging," and a NIST write-up on AI models "cheating on agentic evaluations." A model that sandbags — underperforms on purpose — passes the evaluation while being more capable than the result suggests.

### 4.4 An eval is not a deployment control

An evaluation is a **measurement**. A deployment control is a **mechanism that constrains what the deployed system can do**: input/output classifiers, policy enforcement at the API boundary, human-in-the-loop for high-risk actions, rate limits, monitoring, and kill switches. The two are frequently conflated. DeepMind's framework is unusually explicit that these are different layers — its **security mitigations** (preventing weight exfiltration) and **deployment mitigations** (managing access and preventing expression of critical capabilities) are separate ladders from the CCLs themselves.

The practical consequence for a bank: **you are buying deployment controls, not evaluations.** A lab's passing evaluation tells you something about the lab's decision process. It tells you nothing about whether *your* integration constrains action execution, whether *your* prompts bypass *your* guardrails, or whether *your* fallback works when the model is withdrawn.

### 4.5 What published results do and do not establish

The published artifacts an enterprise can actually read are: the **framework** (§3), a **system card** or **model card** (published per model), an **evaluation summary**, a **risk report** (Anthropic published Risk Reports from February 2026; a "Frontier Risk Report" pilot covering February–March 2026 was published by METR with non-public access provided by four labs), and **safety reports** (DeepMind published a Gemini 3 Pro FSF report).

- **They do establish:** that a named evaluation was run; that the provider made a determination; that the provider publishes the framework and, increasingly, the outcome.
- **They do not establish:** that the model is safe in *your* deployment context; that the evaluation was maximally elicited; that the threshold was set at the "right" level; or that the result generalises to a future model version.
- **They are frequently redacted.** Anthropic's v3.4 update allows public Risk Reports to indicate "where material was redacted," and v3.2–v3.4 formalise external review of unredacted sections. Redaction is legitimate (IP, safety, privacy) *and* it limits what an outside reader can verify. Both are true.

**A worked comparison.** Suppose three providers each publish "our model passed our frontier evaluation." Provider A's framework states a severity bar (OpenAI states its "severe harm" bar as thousands of deaths or hundreds of billions in damage) and a two-tier system; Provider B's states four risk levels and per-domain thresholds with deployment consequences; Provider C's states a CCL name with no explicit severity bar. The three sentences "we passed" are not comparable, because the eval is only as meaningful as the threshold it tests against — and the threshold is self-set. The enterprise reading all three can compare *process transparency*, not *safety*.

---

## 5. The Governance Layer

The lab frameworks are voluntary and self-imposed. The governance layer is where some of the same logic becomes **binding** (or at least enforceable) and where states, not labs, set the rules. This section covers the EU AI Act's GPAI provisions; the national AI safety institutes and what each is actually empowered to do; the international summit sequence; and the definitional problem of "frontier."

### 5.1 The EU AI Act — general-purpose models and the systemic-risk threshold

The EU AI Act's **Chapter V** governs general-purpose AI models. The provision that matters most for frontier risk is **Article 51**, "Classification of General-Purpose AI Models as General-Purpose AI Models with Systemic Risk."

**Verified at artificialintelligenceact.eu (Article 51 and Annex XIII pages):**

- A GPAI model is classified as having **systemic risk** if it "has high impact capabilities evaluated on the basis of appropriate technical tools and methodologies, including indicators and benchmarks," **or** if, by Commission decision, it "has capabilities or an impact equivalent to those … having regard to the criteria set out in Annex XIII."
- **Article 51(2) states the computable presumption:** a GPAI model "shall be presumed to have high impact capabilities … when the cumulative amount of computation used for its training measured in floating point operations is greater than **10²⁵**" — i.e. **10 to the power of 25 FLOP** of cumulative training compute.
- The Commission may adopt delegated acts to amend the thresholds in light of technological developments.
- **Annex XIII** lists the criteria the Commission must consider for an equivalent-capability designation, including number of parameters; quality/size of the dataset; amount of training computation; input/output modalities; benchmarks and evaluations (including autonomy and tool access); and market impact — where high impact on the internal market "**shall be presumed when it has been made available to at least 10 000 registered business users established in the Union.**"

**Application date — verified at artificialintelligenceact.eu's implementation timeline (last updated 31 August 2026):**

- **12 July 2024** — the AI Act is published in the Official Journal.
- **1 August 2024** — entry into force (no requirements apply yet).
- **2 August 2025** — the following start to apply: notified bodies; **GPAI models (Chapter V)**; governance (Chapter VII); confidentiality (Article 78); and certain penalty provisions (Articles 99, 100). Providers of GPAI models placed on the market *before* that date must be compliant by **2 August 2027**.
- **27 July 2026** — Articles 102–110 (amendments to other EU legislation) start applying.
- **2 August 2026** — the remainder of the Act applies, unless specified otherwise.

The European Commission's **AI Office** page (digital-strategy.ec.europa.eu) confirms the Office's enforcement role for GPAI: it develops "tools, methodologies and benchmarks for evaluating capabilities and reach of general-purpose AI models, and **classifying models with systemic risks**," can **conduct evaluations, request information and measures from model providers, and apply sanctions**, and — in case of infringement — can "restrict the public availability of a model and issue direct fines for non-compliance." The same page records that the **"AI omnibus"** (part of the Digital Simplification Package) was adopted in June 2026, "introducing targeted amendments to the AI Act," with amendments **entering into force on 27 July 2026**. (The sibling [./ai_governance_framework_guide.md](./ai_governance_framework_guide.md) owns the AI Act's high-risk machinery and the Omnibus; this guide only needs the GPAI crossing point.)

**One honest caveat.** The 10²⁵ FLOP figure is **training compute** — a *presumption* of high impact, not a measure of risk, and amendable by delegated act. The AI Office itself says models are "classified" using benchmarks and indicators, not compute alone. So even the Act's one hard number is a proxy that the regulator can move.

### 5.2 The national AI safety institutes — and what each is actually empowered to do

The word "institute" hides enormous differences in mandate. What follows distinguishes the **name** from the **power**.

**UK — AI Security Institute (AISI).** Verified at aisi.gov.uk and gov.uk. The body is the **AI Security Institute**, a "research organisation within the UK government's Department for Science, Innovation and Technology" (gov.uk currently lists it as part of the Cabinet Office), with the mission to "equip governments with a scientific understanding of the risks posed by advanced AI." It states its work includes "**testing leading AI systems before they are released publicly** and collaborating with top AI companies," informing policymakers, and advancing research, with "**pre-release access** to leading AI models" and roughly **£66m in funding per financial year**. **What it is empowered to do:** research, pre-deployment testing by agreement, and policy advice — **not** to authorise or block releases. It was renamed from the "AI Safety Institute" to the "AI Security Institute" (the current gov.uk organisation page and aisi.gov.uk confirm the new name; the exact rename date in February 2025 was not re-verified at the announcement page in this pass — see §15).

**US — NIST center; naming is in flux. ⚠** Verified at nist.gov/caisi. As retrieved in this pass, the page at `nist.gov/caisi` is titled "**Center for Advancing Innovation and Standards for Super Intelligence (CAISSI)**" and describes a center that: develops guidelines and best practices for measuring/improving security of "SI systems" with NIST staff; "establishes voluntary agreements with private sector SI developers and evaluators, and leads unclassified evaluations" of capabilities that may pose national-security risk, focusing on "cybersecurity, biosecurity, and chemical weapons"; evaluates US and adversary systems; and coordinates across DoD, DOE, DHS, OSTP and the Intelligence Community. **However**, the *same page's* news items refer to "the U.S. Center for AI Standards and Innovation (CAISI)," and the URL is `/caisi`. The retrieval therefore shows **two names coexisting** — "CAISI (Center for AI Standards and Innovation)" in older/programmatic references and "CAISSI (Center for Advancing Innovation and Standards for Super Intelligence)" in the current page heading. **This guide asserts neither as the settled name; it records both and flags the transition as UNVERIFIED.** What is clear from the page: the centre's power is to run **unclassified evaluations** and develop **voluntary** standards and agreements — not to license or block models.

**Japan — AI Safety Institute (AISI).** Verified at meti.go.jp. The **AI Safety Institute was launched on 14 February 2024** within the **Information-technology Promotion Agency (IPA)**, in collaboration with the Cabinet Office and other ministries, to "examine the evaluation methods for AI safety and other related matters," with a stated aim to "deepen cooperation with similar institutes abroad, including the AI Safety Institute in the United States and the United Kingdom." **Power:** research and examination of evaluation criteria and methods, plus international collaboration — an evaluation-standards body, not a regulator.

**Canada — Canadian AI Safety Institute. ⚠** The page this guide attempted (`ised-isde.canada.ca/site/ised/en/canadian-ai-safety-institute`) returned 404 in this pass. A Canadian AI Safety Institute is referenced in the international institute network, but **its current name, remit and date are UNVERIFIED here** and it is not asserted.

**EU — European AI Office.** Verified at digital-strategy.ec.europa.eu. The **European AI Office** was "established within the European Commission as the foundation for a single AI governance system." It has **more than 125 staff, 6 units and 2 advisers**, including an **AI Safety** unit. Critically, it is **not** an advisory institute: it "**enforces the rules for GPAI models**," with powers to "conduct evaluations of GPAI models, request information and measures from model providers, and apply sanctions," including restricting a model's public availability and issuing fines. It works with the **European AI Board** (Member-State representatives), a **Scientific Panel** of independent experts, and an **Advisory Forum**. This is the one actor in the set with genuine, statutory enforcement teeth over GPAI.

**The pattern worth remembering.** "AI safety institute" names four-plus bodies with wildly different power: the EU AI Office **enforces**; the UK AISI **tests by agreement and advises**; the US centre **evaluates and writes voluntary standards**; Japan's AISI **studies methods and cooperates**. An enterprise that writes "checked against the AI safety institutes" into a control without distinguishing these is writing a control that does not exist.

### 5.3 The international summit sequence

| Summit | Date | Host / venue | Output | Verified at |
|---|---|---|---|---|
| AI Safety Summit | **1–2 November 2023** | Bletchley Park, UK | **The Bletchley Declaration**, agreed by attending countries | gov.uk topical-events page |
| AI Seoul Summit | **21 May 2024** | Seoul, Republic of Korea (with UK) | **Seoul Declaration** for safe, innovative and inclusive AI; **Seoul Statement of Intent toward International Cooperation on AI Safety Science** (Annex); **Frontier AI Safety Commitments** | gov.uk publication pages |
| AI Action Summit | **10–11 February 2025** | Paris, France (Grand Palais; associated events 6–11 Feb) | Paris Action Summit programme and statement on inclusive and sustainable AI | elysee.fr |
| India AI Impact Summit | **February 2026**, New Delhi | India (Bharat Mandapam) | AI Impact Summit Declaration | ⚠ **dates conflict across official sources — see below** |

**The Frontier AI Safety Commitments** (Seoul, 21 May 2024) are the connective tissue between the voluntary frameworks and the state process. The gov.uk guidance page confirms the title and the "Leading AI organisations agree to the Frontier AI Safety Commitments" framing, and shows the page was **updated on 7 February 2025** to add "Additional organisations." METR's analysis states that **sixteen companies** agreed to these commitments in May 2024, "with an additional four companies joining since." The commitments' content (as summarised in the Frontier Model Forum's issue brief) includes defining thresholds "with input from trusted actors, including organisations' respective home governments as appropriate," and restricting further development/deployment until safeguards are in place — the same if-then structure as the lab frameworks, but multilateral.

**⚠ The India AI Impact Summit date conflict, stated honestly.** Official and quasi-official sources disagree: the official summit page (`impact.indiaai.gov.in/about-summit`) is indexed as saying **19–20 February 2026** in New Delhi; a Press Information Bureau release describes a **five-day programme, 16–20 February 2026**; India's Ministry of External Affairs lists the "AI Impact Summit Declaration, New Delhi (February 18–19, 2026)"; and a third-party encyclopedia entry says **16–21 February 2026**. Because these conflict, this guide **does not assert a single date** for this summit and marks it UNVERIFIED. (The official India page itself would not load during this pass.)

### 5.4 The definitional problem: what counts as "frontier"

There is no single definition of "frontier," and the differences are instruments, not semantics:

- **A legal scope rule.** The EU AI Act does not define "frontier" at all; it defines **GPAI** (Article 3(63), a model with "significant generality" that performs a wide range of distinct tasks) and **systemic risk** via the 10²⁵ FLOP presumption and Annex XIII criteria. A statute wants a *computable, administrable* trigger.
- **A lab's internal threshold.** A CCL or an ASL is a *capability* trigger chosen by the developer to match *its* threat model. It is optimised to be actionable by that lab, not to be legally comparable.
- **An enterprise's practical definition.** For a bank, "frontier" is usually a **procurement** question — is this a model whose vendor publishes a frontier framework and whose downstream behaviour I depend on? — not a capability question it can answer itself.
- **Lawmakers' newer scope rules use compute and capability, not the word "frontier."** California's **Transparency in Frontier AI Act (TFAIA)** and New York's **RAISE Act** appear by name in Microsoft's framework as the laws that bring models "in scope for frontier model requirements" (Microsoft's own words, February 2026 PDF). The sibling governance guide owns the statutory detail; the point here is that even the statutes named "frontier" operate through *scope rules*, not through the word.

**The consequence.** "Frontier" is a **family of different instruments**: a benchmark-and-compute legal trigger (EU), a self-set capability threshold (labs), a procurement question (enterprises), and a legislative scope rule (CA/NY). Treating them as one definition is the category error the governance layer most invites.

---

## 6. The Critiques and the Counter-Critiques

This section presents both directions. Each item is structured **claim → who makes it → what evidence supports it**. No editorial verdict is asserted as fact.

### 6.1 Critique: voluntary commitments are not binding

**Claim.** A voluntary, self-imposed framework can be revised, weakened, or abandoned by the developer that wrote it, and nothing outside the developer can compel compliance.
**Who makes it.** Widely made across academia, civil society and policy commentary; it is also visible *inside* the frameworks' own documents.
**Evidence.** The frameworks are self-labelled voluntary and self-revised. Anthropic's version chain shows **eight revisions in under three years** (v1.0 Sep 2023 → v3.4 Jul 2026) — evidence of healthy iteration, *and* evidence that a commitment today is not the commitment in force at a prior date. Microsoft's change log shows a framework that changes with regulation. The Seoul commitments are titled "commitments," not rules.
**Counter (who/evidence).** The Frontier Model Forum's issue brief notes safety frameworks "specify capability and/or risk thresholds … **in advance of their development**," which is a different thing from a post-hoc policy. And a *published* commitment creates a **public record against which deviation can be documented** — the counter-critique developed in §6.6.

### 6.2 Critique: the threshold is self-set

**Claim.** The developer that sets the threshold also chooses where it sits, so the threshold can be placed where it is comfortable rather than where the evidence points.
**Who makes it.** This is the classic RSP critique; it is also conceded, in substance, by the labs themselves.
**Evidence.** Anderljung et al.'s framing and the Frontier Model Forum's brief both describe thresholds as "capability … proxy for risk," chosen by firms "with input from … home governments as appropriate" — input, not approval. Anthropic's v3.0 announcement is direct: pre-set capability levels were "**far more ambiguous than we anticipated**," the science "isn't well-developed enough to provide dispositive answers," and where uncertainty existed Anthropic "took a precautionary approach … but our internal uncertainty translates into a weak external case." A self-set threshold inside a zone of ambiguity is a self-set threshold.
**Counter (who/evidence).** A self-set threshold is still a *published* one, and it disciplines the developer's own future action; the alternative — no threshold — is not a better-disciplined default. See §6.6 and §10.

### 6.3 Critique: the developer runs the evaluation on its own model

**Claim.** Self-assessment means the party with the commercial incentive to deploy is also the party deciding whether the model is safe to deploy.
**Who makes it.** Policy and research commentary; the same self-review documents supply the evidence.
**Evidence.** OpenAI v2.0 states that an internal **Safety Advisory Group** recommends safeguards and "OpenAI Leadership can approve or reject these recommendations," with the Board's Safety and Security Committee providing oversight. Anthropic's v2.0 retrospective candidly lists **four instances where it "fell short of meeting the full letter" of its own policy**, including late evaluations and missing elicitation techniques. These are admissions of self-assessment, accurately reported.
**Counter (who/evidence).** Independent evaluation has grown. The **UK AISI** has pre-release access and tests leading models; **METR** publishes independent evaluations and a pilot **Frontier Risk Report (February–March 2026)** with model access from four labs; DeepMind v3.0 describes **safety case reviews**. The counter is not "self-assessment is fine" but "the ecosystem now contains external checks, and their scope is published."

### 6.4 Critique: "safety-washing"

**Claim.** Capability improvements can be presented as safety progress; a safety-looking metric can rise simply because the model got better at everything.
**Who makes it.** Ren et al., "**Safetywashing: Do AI Safety Benchmarks Actually Measure Safety Progress?**" (arXiv:2407.21792, first submitted 31 July 2024; NeurIPS 2024).
**Evidence.** The paper's abstract states that "many safety benchmarks highly correlate with both upstream model capabilities and training compute, potentially enabling 'safetywashing' — where capability improvements are misrepresented as safety advancements." The evidence is a meta-analysis of safety benchmarks across dozens of models.
**Counter (who/evidence).** The same paper proposes "an empirical foundation for developing more meaningful safety metrics," i.e. the critique is constructive. And frontier *capability* evaluations (CBRN uplift, cyber exploitation) are a different species from the general "safety benchmarks" the paper analyses; the correlation finding is strongest where benchmarks are broad.

### 6.5 Critique: a capability threshold is not a deployment control

**Claim.** A threshold governs the *developer's* future action; it does nothing to constrain a *deployed* system, and an enterprise that treats it as a control has mistaken a promise for a mechanism.
**Who makes it.** This is the central claim of *this* guide's enterprise sections (§4.4, §7, §8); it is also implicit in the frameworks, which separate capability thresholds from deployment mitigations.
**Evidence.** DeepMind's v1.0 report separates CCLs from **Security Levels** and **Deployment Levels**; Microsoft's framework separates capability thresholds from "deployment requirements"; OpenAI separates capability thresholds from "safeguards." The categories are distinct in the primary documents.
**Counter (who/evidence).** A threshold *can* become a control, but only via a **contractual** chain — a change-notification commitment, a deprecation notice, an availability guarantee — that a deployer negotiates. The framework itself is upstream of that chain. See §9.

### 6.6 Counter-critique: a published commitment creates an auditable event

**Claim.** The alternative to a voluntary framework is not "no risk" but **unmanaged risk**; a published, versioned commitment is a governance artifact that can be audited, dated, and held against the issuer.
**Who makes it.** The frameworks' defenders; the labs themselves frame it as a "race to the top."
**Evidence.** Anthropic's v3.0 post states the RSP "**did** incentivize us to develop stronger safeguards," that ASL-3 implementation "did prove feasible," and points to constitutional-classifier work and public Risk Reports. Microsoft's change log documents *regulatory alignment* as a cause of revision — voluntary frameworks moving *to* regulation. The Frontier Model Forum brief says the commitments were "recognized … through the Frontier AI Safety Commitments announced at the AI Seoul Summit."

### 6.7 Counter-critique: the instruments are moving from policy to regulation

**Claim.** Some of these instruments have crossed from voluntary policy into enforceable law, which changes what a critique of "voluntariness" can claim.
**Who makes it.** Regulators and the frameworks themselves.
**Evidence.** The **EU AI Act's GPAI provisions apply from 2 August 2025** (verified), with the **AI Office** able to conduct evaluations, request information, and **fine** (§5.1). The AI Office states the **AI omnibus** amendments entered into force **27 July 2026**. Microsoft's 2026 change log names the **EU GPAI Code of Practice**, **NY RAISE Act** and **CA Transparency in Frontier AI Act** as drivers. Anthropic's v3.0 post names California **SB 53** and New York's **RAISE Act** as governments "start[ing] to require frontier AI developers to create and publish frameworks." The Seoul commitments are voluntary; the EU GPAI regime is not.

### 6.8 Counter-critique: the honest reading is "what is the instrument for"

**Claim.** Critiquing a framework for not being a deployment control misreads its function; it is a *developer* instrument, and criticising it for failing at a *deployer* task is a category error.
**Who makes it.** Framework authors and standards bodies.
**Evidence.** The Frontier Model Forum's brief states safety frameworks are "designed to enable **developers** to take a robust, principled … approach"; the Seoul commitments bind "AI organisations." No framework claims to control downstream deployments. The honest enterprise move is not to attack the framework for what it is not, but to ask what *deployer-side* instrument the enterprise needs and build that (§7–§9).

**Reading all eight together:** the framework's designers describe it as a developer-side, published, auditable commitment about future capability. Its critics describe it as non-binding, self-set, self-assessed, and not a control. **Both descriptions are of the same object, and both are supported by the primary documents.** The instrument's worth to any given party depends entirely on which side of the dependency that party sits on — which is why §7 onward exist.

---
## 7. What an Enterprise Is Actually Exposed To

A bank does not train a frontier model. It does not serve one at the frontier. It **depends** on one. That single fact reframes everything in §3–§6: the bank cannot assess a frontier model's safety, but it can assess its own **dependency** and its own **exit options**. This section enumerates the genuine exposures. It is deliberately narrow — the comprehensive failure-mode register is [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md), not this guide.

### 7.1 Concentration and dependency

The exposure is not that your model is dangerous; it is that **your critical service has a single, upstream, non-substitutable input.** Concretely:

- **Few suppliers.** METR's analysis counts twelve labs publishing frontier safety policies; the set of suppliers a regulated bank can actually contract with is smaller still, because of data-residency, model-risk and vendor-diligence constraints already owned by [./ai_governance_framework_guide.md](./ai_governance_framework_guide.md) and [../responsible_ai_frameworks_guide.md](../responsible_ai_frameworks_guide.md).
- **Deep integration.** Once a bank wires a specific model into origination, servicing, fraud or contact-centre flows, replacing it is not a procurement event — it is a re-validation event across every downstream control.
- **Correlated dependence.** If several bank functions standardise on the same provider, a single provider event (a tier change, a regional withdrawal, a safety-motivated capability restriction) becomes a *bank-wide* event.

### 7.2 Change without a change record

This is the frontier-risk question an enterprise actually faces, stated in §9 in full. A model version can change under a deployed system — a silent update, a routing change, a "latest" alias that resolves differently — **without appearing in the bank's change process.** The bank's model inventory knows what it was told; the provider's serving stack may know more.

### 7.3 Availability and tier change

A safety framework's threshold can, by design, cause the provider to do things that affect availability:

- **Safeguard escalation.** Anthropic states ASL-3 measures include "a commitment not to deploy ASL-3 models if they show any meaningful catastrophic misuse risk under adversarial testing"; DeepMind states it "would put on hold further deployment or development" if mitigations are not ready. A hold on *release* is not a hold on *your already-deployed* model — but a hold on a *future* version breaks your upgrade path.
- **Capability restriction.** Deployment mitigations "manage access to and prevent the expression of critical capabilities in deployments" (DeepMind). A capability restriction is exactly the kind of change that can silently degrade a legitimate banking use case (e.g. a security-research or public-sector-facing function) because a classifier drew a boundary the bank never sees.
- **Deprecation and version retirement.** Providers retire model versions on their own cadence; a bank's change calendar and the provider's safety calendar are not the same calendar.

### 7.4 Due diligence — what a bank can legitimately ask for, and what it can never verify

| A bank can ask for (and usually get) | A bank cannot verify |
|---|---|
| The provider's published safety framework and its version/date | Whether the threshold was set at the "right" level |
| A system card / model card for the deployed version | Whether the evaluation was maximally elicited |
| An evaluation summary or risk report (redacted) | What the redacted material says |
| A change-notification commitment (contractual) | Whether the notification will be complete |
| A deprecation/version-retirement policy | Whether the policy will hold under commercial pressure |
| An incident-notification clause | Whether an incident will be classified as notifiable |
| Security certifications and a security addendum | Whether weight-security claims are true |

The right-hand column is not cynicism; it is **the boundary of what any external party can know**, and it is drawn by the same primary documents cited throughout. A due-diligence questionnaire that asks the right-hand questions and reports answers has manufactured assurance.

### 7.5 The bridge back to the repository

The enterprise side of this — the failure modes that actually occur, the register, the control mapping — is owned elsewhere and should be read alongside this guide:

- **The consolidated failure-mode register for a bank:** [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md). That guide is the register; **this** guide supplies the frontier-specific dependency framing it should carry, not the register itself.
- **The governance/operating model:** [./ai_governance_framework_guide.md](./ai_governance_framework_guide.md) (§5 three lines of defence, §6 committees, §9 banking regulatory mapping).
- **The enterprise responsible-AI landscape:** [../responsible_ai_frameworks_guide.md](../responsible_ai_frameworks_guide.md).
- **Operational security and model-development risk:** [../llm_development_risks_security_guide.md](../llm_development_risks_security_guide.md).

The frontier-specific addition to all of them is exactly one idea: **model risk here is dependency risk plus exit risk, not a safety assessment.**

---

## 8. What a Bank Can and Cannot Verify

### 8.1 The honest statement, first

**A bank cannot assess a frontier model's safety. It can assess its own dependency on that model and its own exit options.** Everything in this section is an elaboration of that sentence. The reason is structural: safety assessment requires *weights access, elicitation capability, threat-model knowledge and evaluation infrastructure* that an external deployer does not have and, under most arrangements, cannot get. What a deployer *can* do is ask for artifacts, read them carefully, and manage the dependency they describe.

### 8.2 What a provider can be asked for

Four artifact families, with what each does and does not establish:

**(a) The safety framework.** The published document from §3.
- *Establishes:* that the provider has a stated process; the risk domains it tracks; the tier labels; the commitments.
- *Does not establish:* that the process is followed for *your* model; that the thresholds are right; that a future version stays in scope.

**(b) A system card / model card.** Published per model, describing capabilities, evaluations and limitations.
- *Establishes:* the provider's own account of the model; often the evaluation outcomes.
- *Does not establish:* independent verification; completeness; relevance to your use case. Cards are frequently redacted.

**(c) An evaluation summary / risk report.** e.g. Anthropic's Risk Reports (from February 2026, published every 3–6 months with redactions), DeepMind's FSF safety reports, METR's independent evaluations and pilot Frontier Risk Report.
- *Establishes:* that named evaluations ran and what the provider concluded; for METR-style reports, an *independent* view.
- *Does not establish:* that the evaluation generalises; that redacted material is immaterial; that the next model behaves the same.

**(d) A change-notification commitment.** A contractual clause — not a framework feature.
- *Establishes:* a duty to tell you about specified changes.
- *Does not establish:* that the specification is complete. See §9.3.

### 8.3 A worked due-diligence request that does not manufacture assurance

The safe pattern is to ask only questions that have **verifiable answers**, and to record unverifiable ones as **residual uncertainty** rather than as satisfied controls.

| Question | Verifiable? | What to do with the answer |
|---|---|---|
| "Which framework versions govern this model, and what are their dates?" | Yes | Record the version/date; re-check on renewal. |
| "What is the next model-version change-notification SLA?" | Yes (contractual) | Put it in the contract; test it with a drill. |
| "What deprecation notice do we get, and how many days?" | Yes | Model the exit timeline (§7.3). |
| "Can we pin a model version and refuse silent updates?" | Yes | Pin it; document the pin as a control. |
| "Is the model 'safe' for our use case?" | **No** | Record as *not provable*; substitute a use-case-scoped operational control. |
| "Has the model been independently evaluated?" | Partly | Read the evaluation; note its scope and its independence. |

### 8.4 The Cymbal framing, introduced early

Cymbal Bank (fictional) runs a critical customer-servicing workflow on a frontier model. Cymbal **cannot** assess that model's safety. Cymbal **can**: pin the version; contract a change-notification and deprecation clause; keep a provider-exit playbook; test a downgrade path; and record explicitly, in its risk documentation, that model-safety assessment is **out of its scope and delegated to the provider and its regulators**. That is not a failure of diligence — it is the accurate description of what diligence means here. §12 develops this in full.

---

## 9. The Model-Supply-Chain and Change-Record Question

### 9.1 Why a silent model update is the real frontier-risk question

The frontier-risk literature talks about catastrophic capability. The enterprise meets frontier risk through a much quieter door: **the model version changed and nothing in the bank's change process noticed.** The mechanisms:

- **Floating aliases.** A serving endpoint named `latest` or a major-version alias resolves to whatever the provider currently serves. The integration does not change; the behaviour does.
- **Silent re-serving.** A provider updates a checkpoint, a safety classifier, or a routing policy behind a stable API name. No bank ticket is raised.
- **Framework-triggered restriction.** A safeguard escalation (§7.3) can change what the model will or will not do, which is a *behavioural* change with no version-bump in the bank's inventory.
- **Post-training and system-prompt drift.** Provider-side system prompts, tool availability and refusal policies change independently of the model weights.

### 9.2 What the bank's change process does with it — and why it usually misses

A bank's change management is built for **its own** changes: code, config, infrastructure. An upstream, contractual, provider-controlled change is a **third-party change**, and third-party-change control is where the gap lives. The mechanics of the change process, the change record, and the third-party risk overlay are owned by:

- [./ai_governance_framework_guide.md](./ai_governance_framework_guide.md) — the lifecycle gates and the third-party AI governance section;
- [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) — the failure-mode register and the change-record failure mode;
- [../llm_development_risks_security_guide.md](../llm_development_risks_security_guide.md) — the security-side change surface.

**This guide does not re-derive those.** It adds the frontier-specific observation: the *reason* a silent update is hard to catch is not that banks lack change control — they have more than most industries — it is that the change **originates outside the bank's configuration boundary** and is often **not version-labelled in a way the bank's inventory can key on**. Vendor change-notification, therefore, is not a nice-to-have; it is the **control substitute** for a capability the bank cannot develop (it cannot evaluate what changed on its own).

### 9.3 What a change-notification commitment can and cannot do

| Commitment | Effect |
|---|---|
| "We will notify you of material changes to the served model version." | Creates a duty; the word "material" is the bank's exposure. |
| "We will publish a version identifier and a changelog." | Lets the bank diff; only if the bank subscribes and parses it. |
| "We provide N days' notice of deprecation." | Bounds the exit timeline; testable. |
| "Safety-motivated restrictions are applied uniformly." | Limits surprise; not verifiable in detail. |

None of these can give the bank what it most wants — a guarantee about what the model does — but together they convert an **invisible** dependency into a **recorded, exercisable** one, which is the achievable goal.

### 9.4 A worked change-record drill (fictional: Cymbal Bank)

Cymbal's third-party AI change drill:

1. **Instrument the alias.** Cymbal logs the resolved model version string on every production call (where the provider exposes one) and alerts on a change it did not expect.
2. **Subscribe and parse.** Cymbal subscribes to the provider's changelog and deprecation feed; a change to either opens a ticket automatically.
3. **Pin or float, deliberately.** For critical flows, Cymbal pins a version and accepts the maintenance cost; for non-critical flows, Cymbal floats and monitors. The choice is recorded, not accidental.
4. **Test the exit.** Once a quarter, Cymbal runs a tabletop: the provider retires the version in 30 days — what breaks, what is re-validated, who signs the fallback? The exit playbook is exercised, not just written.
5. **Record the residual.** The drill ends with a written statement: *we did not, and cannot, verify the new version's safety; we verified that we detected the change, knew our exit, and re-ran our use-case evaluations.*

That last sentence is the entire enterprise deliverable of this guide.

---
## 10. The Frameworks' Own Failure Modes

This is the meta-section: the ways the safety-framework genre can fail *as a genre*, independent of any particular lab's intentions. Each failure mode is stated with the evidence that supports it, and none is asserted as a claim about any specific company's motive.

### 10.1 A threshold set where it is comfortable rather than where the evidence says

**The failure.** The tier boundary lands at a capability level the developer is confident it already exceeds-and-handles, or does not yet approach — either way, a boundary that never bites.
**Evidence.** Anthropic's v3.0 post states thresholds were "far more ambiguous than we anticipated," that models "clearly *approached* the RSP thresholds" but there was "substantial uncertainty about whether they have definitively *passed*," and that "the science of model evaluation isn't well-developed enough to provide dispositive answers." A threshold inside a zone of ambiguity can be read either generously or self-servingly; the primary source supplies the ambiguity, not the motive.
**Guardrail.** Read the threshold *and* the evaluation that tests it. If the evaluation is too weak to distinguish "approached" from "passed," the threshold is decorative. Ask for the elicitation effort, not just the verdict.

### 10.2 An evaluation that does not cover the deployment context

**The failure.** A frontier evaluation tests an abstract capability; the deployment is a specific system with specific tools, data and permissions. The evaluation can pass while the deployment is exposed.
**Evidence.** This is structural, not disputed. DeepMind's v1.0 report separates the *capability* assessment (CCLs) from *deployment* mitigations precisely because "the overall risks of misuse may differ by deployment context." Microsoft's framework says its frontier work "complement[s] Microsoft's broader AI governance program that manages a broader set of risks," many "heavily shaped by use case and deployment environments."
**Guardrail.** Treat the frontier evaluation as **necessary-but-not-sufficient** for your use case, and run your own use-case-scoped evaluation (§4.5, §8.2c).

### 10.3 A commitment with no external verification

**The failure.** The developer makes a commitment and grades its own homework; nothing outside confirms compliance.
**Evidence.** OpenAI v2.0's Safety Advisory Group recommends and "OpenAI Leadership can approve or reject"; Anthropic's v2.0 self-review *found its own lapses* (late evaluations, missing elicitation) — which is both a governance success (it self-reported) and a governance limitation (no external party forced the finding).
**Counter-evidence (the honest other side).** External checks exist and are expanding: UK AISI pre-release testing, METR's independent evaluations and pilot Frontier Risk Report (Feb–Mar 2026), DeepMind and Anthropic external reviews. §6.3 gives the full counter.
**Guardrail.** Prefer providers whose external-review arrangements are *published*; treat "we have external review" without a named reviewer class and a stated scope as unverified.

### 10.4 A safety claim that becomes a marketing asset

**The failure.** A framework's tiers and reports migrate from governance documents into product collateral — "ASL-3-grade safety," "CCL-evaluated" — where the tier label is being used as a *reassurance*, not as a *scope statement*.
**Evidence.** The genre is public-facing by design; the same documents are cited in both governance and commercial contexts. The **safetywashing** paper (arXiv:2407.21792) documents the general phenomenon that safety-flavoured metrics can track capability; the marketing migration is that finding's commercial corollary.
**Guardrail.** When you see a tier label in a sales deck, ask the two questions the label does not answer: *what threshold does it test, and what deployment control does it buy me?*

### 10.5 A capability threshold is a promise about a FUTURE model, not a control on a PRESENT deployment

**The failure.** The whole genre's defining limitation. A scaling policy says: *when a future model reaches level X, we will do Y.* It is silent on the model running in your production system today, and it binds the developer, not you.
**Evidence.** This is the design, stated in the documents: thresholds are "specify[ied] … in advance of their development" (Frontier Model Forum); Anthropic's RSP "implicitly requires us to temporarily pause training of more powerful models"; DeepMind would "put on hold further deployment or development." All future-directed. All developer-directed.
**Guardrail.** This is the thesis, and it is why §7–§9 exist. **The enterprise control is the dependency and exit management; the framework is upstream context.**

### 10.6 The genre can crowd out the failure modes that actually occur

**The failure.** A bank imports frontier-risk language ("catastrophic risk," "loss of control," "CBRN uplift") into an enterprise register and, in doing so, **displaces** the failure modes that actually fire in deployed banking AI: hallucinated figures in a customer interaction, a prompt-injection-driven action, a silent model update, a poorly scoped vendor SLA, a bias-driven compliance breach.
**Evidence.** The enterprise register ([../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md)) catalogues the failure modes that occur; the frontier literature catalogues the ones that (by the labs' own framing) do not yet occur at present capability. The two sets barely overlap, and filling the register with the second set while omitting the first is a **control-displacement** error.
**Guardrail.** Keep frontier risk in its own register row — *dependency on an externally-controlled frontier model* — with dependency/exit controls, and leave the operational failure modes to the register that owns them. See §13.5.

---

## 11. Reading the Primary Sources

A practical skill section. The three artifacts an enterprise analyst can actually obtain are the **safety framework**, the **system card**, and the **evaluation report**. This section says what to look for, what each omits, and how to tell a measurement from an assertion.

### 11.1 Reading a safety framework

Read it in this order, and write down the answers:

1. **Version and date.** Not "the framework" — *which version, effective when.* (e.g. "Anthropic RSP v3.4, effective 8 July 2026"; "OpenAI Preparedness Framework v2.0, 15 April 2025"; "DeepMind FSF v3.1, 17 April 2026"; "Microsoft Frontier Governance Framework, February 2026.") A framework without a version and date in your notes is a citation you cannot defend.
2. **The severity bar.** Does it state what "severe" means? OpenAI v2.0 does ("death or grave injury of thousands of people or hundreds of billions of dollars of economic damage"). Many do not. An unstated bar is an unstated bar.
3. **The tier names, lexically.** Copy the exact labels. Do not translate "High capability" into "ASL-3," or "CCL" into "ASL." **Tier labels do not cross frameworks.**
4. **The trigger.** What capability, tested how, at what elicitation effort? Is it a capability threshold or a risk threshold?
5. **The consequence.** What does the developer *do* at the threshold — deploy with safeguards, hold deployment, hold development? Note that "hold" is a *future* action.
6. **The halt condition.** Does the framework contain a commitment to stop if safeguards are not ready? (METR's analysis finds halting conditions in only 8–9 of 12 policies — check.)
7. **The review mechanism.** Internal group? Board committee? External reviewers? Named or unnamed?
8. **The revision history.** When was it last changed, and *why*? The change log is often the most informative part; Microsoft's and Anthropic's are explicit about the regulatory drivers.

### 11.2 Reading a system card

- **Find the evaluations section.** Which domains were tested, with what result, and — critically — **with what caveats**. Labs are usually candid about limitations; quote the caveat, not just the result.
- **Find the redactions.** Note what is marked withheld and why.
- **Find the deployment context.** Does the card describe the model as shipped (with guardrails) or the raw model? The two are different systems.
- **Do not read "no capability detected" as "no capability."** It means "not detected by the evaluations run" (§4.3).

### 11.3 Reading an evaluation report

- **Who ran it?** Developer, national institute, or independent evaluator (§5.2 for what each is empowered to do).
- **Elicitation.** Was best-of-N, chain-of-thought, scaffolding or fine-tuning used? If not, the result is a floor, not a ceiling.
- **The comparator.** What is the baseline — a search engine, a textbook, an unassisted human? "Uplift" only means something relative to the baseline.
- **Independence and incentives.** Was the report reviewed by a party with an incentive to be candid? METR's risk-assessment work states it does not accept compensation for that work and publishes its methodology.

### 11.4 Measurement vs. assertion — a one-page test

| Looks like | Is it a measurement or an assertion? | How to tell |
|---|---|---|
| "The model scored X on benchmark Y." | Measurement (of Y, at that elicitation) | Find the benchmark definition and the baseline. |
| "The model is safe." | Assertion | There is no evaluation that outputs "safe"; it outputs scores against thresholds. |
| "We are committed not to deploy if…" | Assertion (a promise) about future conduct | It is only a measurement *after* it is tested by an event. |
| "Independent reviewers confirmed…" | Measurement *if* the reviewers are named and their scope stated | Ask who and what scope; unnamed external review is an assertion. |
| "Classified as not posing severe risk." | Conclusion, conditioned on threshold and elicitation | Ask which threshold and which elicitation. |

The single habit that makes an analyst useful here: **always ask "against what threshold, elicited how, reviewed by whom."** A claim that survives those three questions is a measurement; one that does not is a sentence.

---

## 12. The Cymbal Bank Angle

*Cymbal Bank is fictional. This section is explicitly illustrative and describes no real institution.*

### 12.1 The setup

Cymbal Bank — a mid-sized universal bank — has wired a frontier model into a **critical customer-servicing workflow**: drafted responses, summarisation of customer history, and a narrow agentic capability that can open and route service tickets. The model is procured from a frontier lab. Cymbal's use is entirely ordinary — no training, no weights access, no frontier capability of its own. Cymbal's frontier-risk exposure is therefore **100% dependency**, and Cymbal's management of it is therefore **100% dependency management**, not model assessment.

### 12.2 What Cymbal can ask for

- The provider's **safety framework**, by **version and date** (e.g. "the framework governing this model is version X, dated Y"), re-confirmed each year.
- A **system card** for the pinned model version.
- The provider's **evaluation summary / risk report** (read with §11's questions).
- A **change-notification commitment**: notice of material changes to the served version; changelog subscription; deprecation notice in days.
- A **version-pinning** right, and the provider's **deprecation policy**.
- **Incident-notification** and **security-addendum** terms.

### 12.3 What Cymbal cannot verify

- Whether the model is **safe** — Cymbal has no elicitation capability, no weights access, no threat model at the frontier.
- Whether the **threshold** the provider uses is set correctly.
- Whether a **redacted** section of a report matters.
- Whether the provider's **safeguards** are as effective as claimed.
- Whether a **future** version will behave like the current one.

### 12.4 The dependency and exit question

Cymbal's real decisions are: **pin or float** (per flow, deliberately); **how many days' exit** it has (set by the deprecation clause); and **what it re-runs on a version change** (its own use-case evaluations, at its own cost). Cymbal writes these into its third-party register with the governance guide's overrides ([./ai_governance_framework_guide.md](./ai_governance_framework_guide.md) §8) and the banking failure-mode register ([../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md)).

### 12.5 The change-notification gap

Cymbal's most likely frontier-risk event is **not** a catastrophe. It is: *a "material" change that the provider did not consider material.* Cymbal's control is not to argue the definition but to **detect and bound**: log the resolved version, subscribe to the changelog, alert on unexpected change, and hold a tested 30-day exit. The drill in §9.4 is Cymbal's control.

### 12.6 The conclusion

Cymbal's frontier-risk management is **dependency management**, not model assessment. Cymbal **delegates** the safety assessment to the provider and its regulators — because that is who can do it — and **retains** the dependency and exit management, because that is what it alone can do. And it records this honestly: *Cymbal cannot assess the frontier model; Cymbal can assess its own dependency and its exit options.*

That is the thesis at the enterprise scale: **a scaling policy is a commitment about a model that does not exist yet** — and Cymbal's job is to manage the model that *does*, on the terms it can actually enforce.

---
## 13. The Anti-Patterns

Each anti-pattern: **symptom → cause → guardrail.**

### 13.1 Treating a safety framework as a control

- **Symptom:** the enterprise risk register or vendor questionnaire lists "provider publishes a frontier safety framework" as a *mitigating control* against model risk.
- **Cause:** category error — the framework governs the developer's future conduct, not the deployed system's behaviour (§7.4, §10.5).
- **Guardrail:** record the framework as **evidence about the provider's process**, not as a control. The control is the deployer-side mechanism: pinning, detection, change-notification, exit.

### 13.2 Treating an evaluation pass as a clearance

- **Symptom:** "the model passed the frontier evaluations, so it is cleared for our use case."
- **Cause:** the **elicitation gap** (§4.3) and the deployment-context gap (§10.2). A pass is "not detected against this threshold at this elicitation."
- **Guardrail:** run a **use-case-scoped** evaluation; treat the provider's pass as necessary-but-not-sufficient; never let the word "passed" do the work of "safe for us."

### 13.3 Assuming a published policy binds a provider

- **Symptom:** a contract or control says "the provider will not deploy above [tier]."
- **Cause:** the frameworks are voluntary, self-set and self-revised by the provider (§6.1–6.3).
- **Guardrail:** **only the contract binds.** Convert what you need into a contractual term (change notification, deprecation, SLA, audit right); do not rely on the framework document as an obligation.

### 13.4 Signing a dependency with no exit

- **Symptom:** a critical flow is deeply integrated with one model version, with no tested fallback and no pinned version.
- **Cause:** integration is optimised for capability and speed, not reversibility.
- **Guardrail:** for every critical flow, choose **pin-or-float deliberately**; hold a **tested exit playbook** (deprecation scenario, fallback model, re-validation scope); record the exit timeline in the third-party register.

### 13.5 Importing frontier-risk language into an enterprise register where it displaces the failure modes that actually occur

- **Symptom:** the bank's AI risk register fills with "CBRN uplift," "loss of control" and "catastrophic risk" rows, while the rows for hallucination-in-customer-facing-output, prompt injection, silent model updates, vendor SLA gaps and bias breaches are thin or missing.
- **Cause:** the frontier literature is vivid and the operational failure literature is mundane; vividness crowds out the mundane in a register (§10.6).
- **Guardrail:** keep **one** frontier-specific row — *dependency on an externally-controlled frontier model* — with dependency/exit controls; give the operational failure modes to the register that owns them ([../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md)); do not let the frontier row expand into a taxonomy the bank cannot act on.

### 13.6 Two more, briefly

- **Treating tier labels as interchangeable.** "ASL-3" ≠ "Critical capability" ≠ "CCL" ≠ "high risk." Never restate one in another's vocabulary (§3.7). *Guardrail: quote labels verbatim, with the framework and version.*
- **Treating "frontier" as one definition.** The EU scope rule, a lab CCL, a procurement question and a legislative scope rule are four instruments (§5.4). *Guardrail: say which definition you mean every time you use the word.*

---

## 14. The Claims Audit

Legend: **✅ verified** at the issuing body's own published document in this pass (October 2026); **⚠ flagged** — sourced from a secondary index or partially verified, treat with care; **❌ rejected / unverified** — not asserted in this guide.

### 14.1 Frontier safety frameworks

| Claim | Status | Source | Date | Quality |
|---|---|---|---|---|
| Anthropic RSP, v1.0 effective 19 Sep 2023; current **v3.4 effective 8 Jul 2026** | ✅ | anthropic.com/responsible-scaling-policy (page "Last updated Aug 14, 2026") | 2026-08-14 | Primary, first-party |
| Anthropic version chain: v2.0 15 Oct 2024; v2.1 31 Mar 2025; v2.2 14 May 2025; v3.0 24 Feb 2026; v3.1 2 Apr 2026; v3.2 29 Apr 2026; v3.3 26 May 2026; v3.4 8 Jul 2026 | ✅ | anthropic.com/responsible-scaling-policy | 2026 | Primary, first-party |
| Anthropic tier label **AI Safety Levels (ASL)**, ASL-1 to ASL-5+; ASL-2 included "current LLMs, including Claude" at v1.0 | ✅ | anthropic.com/news/anthropics-responsible-scaling-policy | 2023-09-19 | Primary |
| Anthropic v3.0 "theory of change" candidacy: "consensus about risks did not play out"; "zone of ambiguity" | ✅ | anthropic.com/news/responsible-scaling-policy-v3 | 2026-02-24 | Primary, first-party |
| OpenAI Preparedness Framework Beta **Dec 2023**; **v2.0, 15 Apr 2025** | ✅ | cdn.openai.com preparedness-framework-v2.pdf (document header); metr.org/fsp | 2025-04-15 | Primary document |
| OpenAI Tracked Categories: Biological & Chemical; Cybersecurity; AI Self-improvement; introduced Research Categories; thresholds **High capability** and **Critical capability** | ✅ | cdn.openai.com preparedness-framework-v2.pdf | 2025-04-15 | Primary document |
| OpenAI "severe harm" bar = death/grave injury of thousands or hundreds of billions USD damage | ✅ | cdn.openai.com preparedness-framework-v2.pdf (footnote 1) | 2025-04-15 | Primary document |
| OpenAI Safety Advisory Group + Leadership approve/reject + Board Safety & Security Committee oversight | ✅ | cdn.openai.com preparedness-framework-v2.pdf | 2025-04-15 | Primary document |
| DeepMind FSF v1.0 17 May 2024; v2.0 Feb 2025; v3.0 22 Sep 2025; **v3.1 17 Apr 2026** | ✅ | deepmind.google blog (17 May 2024; 22 Sep 2025 updated 17 Apr 2026); FSF PDFs | 2024–2026 | Primary, first-party |
| DeepMind tier label **Critical Capability Levels (CCLs)**; from v3.1 also **Tracked Capability Levels (TCLs)**; CCL domains Autonomy, Biosecurity, Cybersecurity, ML R&D + harmful manipulation (v3.0) | ✅ | deepmind.google blog; FSF v1.0 technical report | 2024–2026 | Primary |
| DeepMind v1.0 CCL names e.g. "Autonomy level 1", "ML R&D level 2"; Security Levels 0–4; Deployment Levels 0–3; 6× compute / 3-month evaluation cadence | ✅ | FSF v1.0 technical report PDF | 2024-05-17 | Primary document |
| Meta "Frontier AI Framework" v1.0 Feb 2025; "Advanced AI Scaling Framework" v2.0 8 Apr 2026 | ⚠ | metr.org/fsp (index) | 2025–2026 | Secondary index; Meta pages did not load |
| Meta tier labels | ❌ | — | — | **Not verified; not asserted** |
| Microsoft Frontier Governance Framework v1.0 Feb 2025; **February 2026 update** | ✅ | Microsoft Frontier Governance Framework PDF (change log, Appendix II) | 2026-02 | Primary document |
| Microsoft risk levels low/medium/high/critical; tracked capabilities CBRN, offensive cyberoperations, advanced autonomy, loss of control, harmful manipulation | ✅ | Microsoft Frontier Governance Framework PDF | 2026-02 | Primary document |
| Microsoft leading-indicator scope ties to EU AI Act, CA TFAIA, NY RAISE Act; ≥6-month cadence; substantial fine-tune = >1/3 base compute | ✅ | Microsoft Frontier Governance Framework PDF | 2026-02 | Primary document |
| Amazon "Frontier Model Safety Framework" 10 Feb 2025; xAI frameworks 2025–2026; NVIDIA 17 Feb 2025; G42 6 Feb 2025; Cohere 7 Feb 2025; Magic 2 Jul 2024; NAVER 7 Aug 2024 | ⚠ | metr.org/fsp (index) | 2024–2026 | Secondary index; not re-verified at source |
| Twelve companies published frontier safety policies as of METR's analysis (Anthropic, OpenAI, Google DeepMind, Magic, Naver, Meta, G42, Cohere, Microsoft, Amazon, xAI, NVIDIA) | ⚠ | metr.org/common-elements (Dec 2025 version) | 2025-12 | Secondary analysis |
| Sixteen companies agreed to the Frontier AI Safety Commitments (Seoul, May 2024), +4 since | ⚠ | metr.org/common-elements; gov.uk page (updated 7 Feb 2025 adds orgs) | 2024-05 / 2025-02 | Primary gov.uk page confirms title/updates; the "16/+4" count is from METR |
| Frontier Model Forum five core components (risk identification; thresholds; assessment; mitigation; governance) | ✅ | frontiermodelforum.org issue brief | 2024-11-08 | Primary, first-party |

### 14.2 Evaluations

| Claim | Status | Source | Date | Quality |
|---|---|---|---|---|
| Anthropic v2.0 self-reported lapses (late evals; changed autonomy tasks; missing best-of-N/CoT; no 6× buffer) | ✅ | anthropic.com/responsible-scaling-policy (v2.0 notes) | 2024-10-15 | Primary, first-party |
| DeepMind lists "Capability elicitation" as future work (avoid underestimation) | ✅ | FSF v1.0 technical report PDF | 2024-05-17 | Primary document |
| "Safetywashing" — safety benchmarks correlate with capability/compute | ✅ | arXiv:2407.21792 (Ren et al.), NeurIPS 2024 | 2024-07-31 (v3 2024-12-27) | Peer-reviewed paper |
| METR MALT dataset (reward hacking/sandbagging behaviours); NIST write-up on models cheating on agentic evals | ✅ / ⚠ | metr.org (MALT, 14 Oct 2025); nist.gov/caisi blog | 2025–2026 | Primary (METR); primary (NIST) |
| Anthropic Risk Reports from Feb 2026; METR Frontier Risk Report pilot Feb–Mar 2026 | ✅ | anthropic.com/responsible-scaling-policy; metr.org | 2026 | Primary, first-party |

### 14.3 The EU AI Act and the governance layer

| Claim | Status | Source | Date | Quality |
|---|---|---|---|---|
| EU AI Act **Article 51(2)**: systemic-risk presumption when cumulative training compute **> 10²⁵ FLOP** | ✅ | artificialintelligenceact.eu/article/51 | in force 2025-08-02 | Primary statute text |
| **Annex XIII** criteria incl. **10 000 registered business users** presumption | ✅ | artificialintelligenceact.eu/annex/13 | — | Primary statute text |
| **GPAI application date 2 August 2025**; pre-existing GPAI compliant by 2 Aug 2027; remainder 2 Aug 2026 | ✅ | artificialintelligenceact.eu/implementation-timeline (last updated 31 Aug 2026) | 2026-08-31 | Primary tracker |
| AI Office: established within EC; >125 staff, 6 units; enforces GPAI rules; can evaluate, request info, fine, restrict availability | ✅ | digital-strategy.ec.europa.eu/en/policies/ai-office | last update 2026-09-08 | Primary, first-party |
| AI omnibus adopted June 2026; amendments in force **27 July 2026** | ✅ | digital-strategy.ec.europa.eu/en/policies/ai-office | 2026-09-08 | Primary, first-party |
| UK **AI Security Institute** (AISI), part of Cabinet Office/DSIT; pre-release testing; ~£66m/financial year | ✅ | aisi.gov.uk; aisi.gov.uk/about; gov.uk organisation page | 2026 | Primary, first-party |
| UK AISI renamed from "AI Safety Institute" | ⚠ | gov.uk/aisi.gov.uk confirm new name; **exact Feb 2025 rename announcement not re-verified** | — | Name verified; date flagged |
| US centre: page titled **CAISSI** ("Center for Advancing Innovation and Standards for Super Intelligence") while news items use **CAISI** ("Center for AI Standards and Innovation") | ⚠ | nist.gov/caisi | 2026 | **Name in transition; both recorded** |
| Japan **AI Safety Institute launched 14 Feb 2024**, within IPA | ✅ | meti.go.jp/english/press/2024/0214_001.html | 2024-02-14 | Primary, first-party |
| Canada AI Safety Institute — name/remit/date | ❌ | attempted ised-isde.canada.ca page 404 | — | **Not verified; not asserted** |
| EU **European AI Office** established within the Commission | ✅ | digital-strategy.ec.europa.eu | 2026 | Primary |
| Bletchley Park AI Safety Summit **1–2 Nov 2023**; Bletchley Declaration | ✅ | gov.uk topical-events/ai-safety-summit-2023 | 2023-11 | Primary, first-party |
| AI Seoul Summit **21 May 2024**; Seoul Declaration; Frontier AI Safety Commitments | ✅ | gov.uk publication pages | 2024-05-21 | Primary, first-party |
| Paris AI Action Summit **10–11 Feb 2025** (associated events 6–11 Feb) | ✅ | elysee.fr/en/sommet-pour-l-action-sur-l-ia | 2025-02 | Primary, first-party |
| India AI Impact Summit, New Delhi, **February 2026** | ⚠ | official page indexed as 19–20 Feb; PIB 16–20; MEA declaration 18–19; encyclopedia 16–21 | 2026 | **Dates conflict — no single date asserted** |
| International AI Safety Report: first 2025; **second published 3 Feb 2026**; led by Yoshua Bengio; 30+ countries, 100+ experts | ✅ | internationalaisafetyreport.org | 2026-02-03 | Primary, first-party |

### 14.4 The claim this guide makes about itself

| Claim | Status | Source | Date | Quality |
|---|---|---|---|---|
| This guide asserts no position on how dangerous any model/capability is | ✅ | this document | 2026-10 | Self-description |
| No capability claim about a named model beyond the model's own publisher | ✅ | this document | 2026-10 | Self-description |

---

## 15. What Could Not Be Verified and the Glossary

### 15.1 What Could Not Be Verified

The honest ledger. Each entry names what was checked and why it failed.

1. **Meta's frontier framework documents and tier labels.** Checked `ai.meta.com/blog/meta-frontier-ai-framework/` (page unavailable), `about.fb.com/news/2025/02/meta-frontier-ai-framework/` (404), `ai.meta.com/static-resource/meta-frontier-ai-framework/` (error page), and an Internet Archive snapshot (empty body). **Meta's tier labels are unverified and are not asserted.** The framework *names and dates* are recorded only as sourced from METR's index (⚠).
2. **The exact UK AISI rename date.** The current name (**AI Security Institute**) is verified at gov.uk and aisi.gov.uk; the February 2025 announcement page for the rename did not resolve in this pass. The name is asserted; the rename date is not.
3. **The US centre's settled name.** nist.gov/caisi shows **CAISSI** in its page heading and **CAISI** in its news items. Both are recorded; neither is asserted as final.
4. **The Canada AI Safety Institute** — attempted page returned 404; name, remit and date not verified and not asserted.
5. **The exact dates of the India AI Impact Summit (2026).** Official, PIB, MEA and third-party sources disagree (19–20 Feb; 16–20 Feb; 18–19 Feb; 16–21 Feb). No single date is asserted.
6. **The seven non-core frameworks** (Amazon, xAI, NVIDIA, G42, Cohere, Magic, NAVER): names, issuers and dates are recorded only from METR's index and were not re-verified at each issuing body. Amazon's page was retrieved but the framework body did not extract cleanly. Their tier labels are not asserted.
7. **OpenAI's "Frontier Governance Framework" (May 2026)** and **Anthropic's "Frontier Compliance Framework" (June 2026)** — both appear in METR's index but were not retrieved at source in this pass; not relied upon.
8. **xAI's/`openai.com` index pages and several lab blog URLs** returned server errors (5xx) rather than content — a **tool/access limitation**, not evidence of absence. Where that blocked verification, the affected claim is marked ⚠ or ❌ above rather than sourced from memory.
9. **The precise delegation of per-model evaluation results** (which model scored what) is intentionally not asserted for any model, per this guide's no-capability-claims rule.

### 15.2 Glossary

| Term | Definition |
|---|---|
| **ASL (AI Safety Level)** | Anthropic's tier label in the RSP; ASL-1 through ASL-5+. |
| **Capability threshold** | A pre-specified capability level at which a developer judges severe-harm risk meaningfully increases absent safeguards. A *proxy* for risk. |
| **CCL (Critical Capability Level)** | Google DeepMind's tier label: a capability level at which, absent mitigation, a model may pose heightened risk of severe harm. |
| **TCL (Tracked Capability Level)** | DeepMind tier label added in FSF v3.1 (17 Apr 2026) for earlier-staged tracking of less-extreme risks. |
| **Safety case** | A structured argument, with evidence, that a system is safe enough for a given deployment context; an argument, not a certificate. |
| **Scaling policy** | A safety framework structured around capability scaling, with if-then safeguards and a hold/pause commitment. |
| **Elicitation** | The effort to draw out a model's full capability (best-of-N, chain-of-thought, scaffolding, fine-tuning, tools). |
| **Elicitation gap** | The difference between maximally elicited capability and what a routine evaluation measures; a passing eval does not close it. |
| **Dangerous-capability evaluation** | An evaluation measuring a capability associated with severe harm (CBRN, cyber, autonomy, persuasion). |
| **GPAI** | General-purpose AI model, EU AI Act Art. 3(63). |
| **Systemic risk (EU)** | A risk specific to a GPAI model's high-impact capabilities, with significant market impact (Art. 3(65)); presumed at >10²⁵ FLOP training compute (Art. 51(2)). |
| **Frontier model** | A general-purpose model at or near the current capability frontier; "frontier" has several incompatible definitions (see §5.4). |
| **System card / model card** | A per-model published document describing capabilities, evaluations and limitations; frequently redacted. |
| **Risk report** | A periodic published assessment (e.g. Anthropic Risk Reports from Feb 2026); redacted versions may indicate where material was withheld. |
| **Safetywashing** | Presenting capability improvements as safety progress; documented in Ren et al., arXiv:2407.21792. |
| **Voluntary / self-set / self-assessed** | The three design properties of essentially all frontier safety frameworks. |
| **Change-notification clause** | A contractual duty on a provider to notify the deployer of specified model changes; the deployer's substitute for capabilities it lacks. |

---

## 16. The Cross-References and the Closing Summary

### 16.1 The cross-references

- **Enterprise failure-mode register (companion, parallel):** [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) — owns the consolidated register and the failure modes that actually occur.
- **Governance framework and operating model:** [./ai_governance_framework_guide.md](./ai_governance_framework_guide.md) — §2 global instruments, §3 Singapore (IMDA, MAS FEAT), §5 three lines of defence, §8 third-party AI governance, §9 banking regulatory mapping.
- **Enterprise responsible-AI landscape:** [../responsible_ai_frameworks_guide.md](../responsible_ai_frameworks_guide.md) — §2 corporate frameworks, §6 banking angle.
- **Model-development risks and security:** [../llm_development_risks_security_guide.md](../llm_development_risks_security_guide.md).
- **Red-teaming and bias measurement:** [./ai_red_teaming_guide.md](./ai_red_teaming_guide.md), [./ai_governance_bias_redteaming_guide.md](./ai_governance_bias_redteaming_guide.md).
- **Primary sources, by issuing body:** anthropic.com/responsible-scaling-policy; cdn.openai.com (Preparedness Framework v2.0); deepmind.google + the FSF PDFs; the Microsoft Frontier Governance Framework PDF; artificialintelligenceact.eu (Art. 51, Annex XIII, timeline); digital-strategy.ec.europa.eu (AI Office); gov.uk + aisi.gov.uk; meti.go.jp; nist.gov/caisi; elysee.fr; internationalaisafetyreport.org; metr.org/fsp and metr.org/common-elements (index/analysis); frontiermodelforum.org.

### 16.2 The closing summary

The frontier-risk literature is a remarkable artifact: an industry and a set of governments have built a working vocabulary — capability thresholds, ASLs, CCLs, High and Critical capability, systemic risk, safety cases — for reasoning about risks at capability levels **nobody has yet measured**. Read at source, the instruments are more candid than their reputation: the labs publish their own lapses, name their own zones of ambiguity, and admit the limits of their evaluations. Read honestly, the critiques are of the *instrument's design*, not of its authors' sincerity: these commitments are voluntary, self-set and self-assessed; the threshold sits where the developer places it; the developer grades its own homework; and the term *safetywashing* exists because safety-looking metrics can rise with capability.

The enterprise truth is narrower and harder. A bank does not train or serve a frontier model; it depends on one. It cannot assess that model's safety — the weights, the elicitation and the threat model are not its to hold — but it can assess its own dependency and its exit options. Its real exposures are concentration, change without a change record, availability and tier change, and a due-diligence boundary it must respect rather than paper over. Its controls are pinning, detection, a contractual change-notification duty, a tested exit, and the discipline to keep frontier language from displacing the operational failure modes that actually fire.

Both readings — the framework as an auditable commitment, and the framework as a non-binding, self-set, self-assessed, future-directed instrument — are true of the same document. The instrument's worth depends entirely on which side of the dependency you sit. And the sentence that keeps the whole subject in proportion, at every scale from the frontier lab to Cymbal Bank, is this: **a scaling policy is a commitment about a model that does not exist yet.**
