# AI for IT — The Operated Intelligence

> **A comprehensive guide to AI for IT — the application of AI/ML and LLM/agentic technology to IT operations themselves. Four pillars: AIOps (observability, anomaly detection, alert-noise reduction, root-cause analysis, predictive capacity); AI in ITSM (ticket triage, virtual agents, knowledge, auto-remediation); AI for the IT workforce (coding and admin copilots, runbook automation, self-healing); and the governance and risk of letting AI act on production infrastructure. The provenance, the vendor stacks, the verified numbers, the failure modes, the guardrails, and a worked AIOps rollout for Cymbal Bank IT operations.**

**Author:** Jack Liu Shurui, Solution Architect
**Role:** Solution Architect, Cymbal Bank
**Last Updated:** September 2026
**Version:** 1.0
**Repository:** github.com/jackliusr/research
**Series:** Technology Guides — AI/LLM & Operations track

**Companion guides.** This guide is the *inbound* twin of [`agentops_guide.md`](agentops_guide.md): where AgentOps is the operations discipline *for* AI agents, this guide is the application of AI *to* IT operations. The repo's IT-operations siblings are condensed cross-references here, not re-derived: [`../operational_support_frameworks_guide.md`](../operational_support_frameworks_guide.md) (ITIL-class service-management context), [`../technology_lifecycle_management_guide.md`](../technology_lifecycle_management_guide.md) (asset/EOL lifecycles that AIOps telemetry enriches), [`copilot_studio_agent_engineering_knowledge_guide.md`](copilot_studio_agent_engineering_knowledge_guide.md) (ServiceNow/Power Platform integration engineering), and the repo's agent cluster — [`enterprise_agentic_platform_architecture_guide.md`](enterprise_agentic_platform_architecture_guide.md), [`multi_agent_banking_guide.md`](multi_agent_banking_guide.md), [`agent_harness_engineering_guide.md`](agent_harness_engineering_guide.md), [`llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md) — plus [`ai_governance_framework_guide.md`](ai_governance_framework_guide.md) and [`implementing-responsible-ai.md`](implementing-responsible-ai.md) for governance. For banking/regulated context see [`../../banking/operational_resilience_framework_guide.md`](../../banking/operational_resilience_framework_guide.md).

**Primary sources and evidence conventions.** Facts checked this pass at a primary source — vendor product documentation, OSS project docs, standards bodies, or the original research paper — are marked **✅**. Claims that are vendor-published marketing, single-source, approximate, or not re-verified this pass are marked **⚠**. Claims contradicted at a primary source are **❌**. The consolidated ledger is the **Claims Audit** (§14.1); the honest residue is **What Could Not Be Verified** (§14.2). Web checks for this guide were bounded and partly rate-limited, so a `✅` means "confirmed at the cited primary source during this pass," not "independently re-tested."

**How to read this guide.** §1–§3 are the domain and its provenance; §4–§9 are the capability deep-dives, one per pillar layer (telemetry, detection, diagnosis, ITSM, workforce, action); §10–§11 are the evidence and the failure modes; §12–§13 are governance, risk, and economics; §14–§15 close with the claims audit, the unverified list, and the glossary. Each body section ends with a cross-reference line instead of re-deriving a sibling guide's content.

**Reading time:** ~55 minutes

---

### Table of Contents

1. [The Domain: AIOps, the Coinage, and the Analyst-Vendor Category](#1-the-domain-aiops-the-coinage-and-the-analyst-vendor-category)
2. [The Scope Map and the Demarcation Lines](#2-the-scope-map-and-the-demarcation-lines)
3. [Telemetry and Observability AI](#3-telemetry-and-observability-ai)
4. [Anomaly Detection and Alert-Noise Reduction](#4-anomaly-detection-and-alert-noise-reduction)
5. [Root-Cause Analysis: Topology, Graphs, and LLM Assistance](#5-root-cause-analysis-topology-graphs-and-llm-assistance)
6. [AI in ITSM: The Service Desk, Triage, and Deflection](#6-ai-in-itsm-the-service-desk-triage-and-deflection)
7. [AI for the IT Workforce: Copilots and Runbook Automation](#7-ai-for-the-it-workforce-copilots-and-runbook-automation)
8. [Auto-Remediation and Self-Healing: Guarded Autonomy](#8-auto-remediation-and-self-healing-guarded-autonomy)
9. [Evidence and Metrics: MTTD, MTTR, and the Published Numbers](#9-evidence-and-metrics-mttd-mttr-and-the-published-numbers)
10. [Failure Modes and Critiques](#10-failure-modes-and-critiques)
11. [Governance, Risk, and the Regulated Estate](#11-governance-risk-and-the-regulated-estate)
12. [Build vs Buy, Cost, and the Operating Model](#12-build-vs-buy-cost-and-the-operating-model)
13. [Worked Example: An AIOps Rollout at Cymbal Bank](#13-worked-example-an-aiops-rollout-at-cymbal-bank)
14. [Claims Audit and What Could Not Be Verified](#14-claims-audit-and-what-could-not-be-verified)
15. [Glossary and Closing](#15-glossary-and-closing)

---

## 1. The Domain: AIOps, the Coinage, and the Analyst-Vendor Category

### 1.1 Definition

**AI for IT** is the application of artificial intelligence and machine learning — increasingly including LLMs and agentic systems — to the operation of IT itself: the monitoring, detection, diagnosis, decision, and action loops that keep an IT estate running. Its best-known label is **AIOps** (Artificial Intelligence for IT Operations), and its four pillars are:

1. **AIOps proper** — AI-augmented IT operations: observability and telemetry, anomaly detection, alert correlation and noise reduction, root-cause analysis, predictive capacity and failure. (§2–§5)
2. **AI in ITSM** — service management: ticket triage and routing, virtual agents, knowledge management, summarisation and deflection, auto-remediation. (§6)
3. **AI for the IT workforce** — coding copilots, admin copilots, runbook automation, self-healing infrastructure. (§7–§8)
4. **Governance and risk** — the controls, guardrails, audit trail, and regulated-environment constraints that let a bank let AI *act* on production. (§11)

The load-bearing word is **operated**: this is AI pointed at the systems that run the business, not AI pointed at the customer. It is *operational intelligence* — intelligence that observes, reasons about, and acts on the estate.

### 1.2 The coinage — VERIFIED, and the framing flag

**AIOps was coined by Gartner in 2016.** That much is confirmed at multiple independent points: the encyclopaedic record states AIOps "was first defined by Gartner in 2016, combining 'artificial intelligence' and 'IT operations' to describe the application of AI and machine learning to enhance IT operations," introduced to address "increasing complexity and data volume in IT environments," aiming to automate "event correlation, anomaly detection, and causality determination" **✅** (Wikipedia, *AIOps*, citing Gartner). Independent vendor glosses agree on the year: "AIOps was originally coined by Gartner in 2016 as artificial intelligence for IT operations" **✅** (CDW). Gartner's own market definition — quoted in the *Gartner Market Guide for AIOps Platforms* — reads:

> "AIOps platforms are software systems that combine big data and AI or machine learning functionality to enhance and partially replace a broad range of IT operations processes and tasks, including availability and performance monitoring, event correlation and analysis, IT service management, and automation." **✅** *(Gartner Market Guide for AIOps Platforms, market definition; retrieved via the ServiceNow-hosted copy of the guide.)*

One detail is worth flagging rather than asserting: several secondary sources state that Gartner's **original** 2016 expansion of the acronym was "**Algorithmic** IT Operations," later softened to "Artificial Intelligence for IT Operations." **⚠** This pass could not confirm the original wording at a Gartner primary source, and the surviving encyclopaedic text uses the "artificial intelligence" reading. Treat "Algorithmic IT Operations" as *widely repeated but not verified here*.

**⚠ The analyst-vendor framing flag.** AIOps is not a natural category that enterprises discovered; it is an **analyst-created market category** that a vendor ecosystem then filled. Gartner named it, defined it, published Magic Quadrants and Market Guides around it, and the ITOps tool vendors — Dynatrace, Splunk, Datadog, ServiceNow, IBM, Broadcom, Elastic — organized their roadmaps and marketing around the label. **⚠** The practical consequences are honest ones for a buyer to hold in mind:

- **The boundary of "AIOps" is commercially elastic.** Because no standards body owns the term, almost any monitoring product with a machine-learning feature can claim membership; "does it do AIOps?" is a weaker question than "which of the §2 loop stages does it actually automate, on what data, with what accuracy?"
- **Vendor-published ROI is marketing until independently reproduced.** Nearly every headline number in this domain originates from a vendor's own customer story or a sponsored analyst note (§9). That does not make the numbers false; it makes them *unfalsified*.
- **The category is real even if the label is constructed.** The underlying problem — telemetry volume and system complexity outpacing human attention — is genuine and measurable independent of who named it. This guide keeps the term because it is the industry's shared vocabulary, not because it is a taxonomy.

### 1.3 What the domain is *not*

| Adjacent discipline | What it owns | Boundary with AI for IT |
|---|---|---|
| **Classic monitoring / APM** | Thresholds, dashboards, uptime, static alert rules | Classic monitoring *reports*; AI for IT *correlates and infers*. Thresholds are the substrate the ML layer sits on (§2.3). |
| **Observability (OTel-class)** | Traces, metrics, logs as a data platform | Observability is the *input*; AIOps is the reasoning layer over it (§3). |
| **AgentOps** | Operating *AI agents* in production | **The inverse of this guide** — AgentOps operates the AI; AI for IT is AI operating the IT. One sentence each way; see [`agentops_guide.md`](agentops_guide.md). |
| **MLOps** | Lifecycle of ML models (training, registry, drift) | MLOps produces models; AIOps consumes them for ops decisions. Different unit of production (model vs. service). |
| **ITSM / ITIL** | Service, incident, change, problem practice | ITSM is the *process frame*; AI-for-ITSM is AI applied inside it (§6). Support-framework detail lives in [`../operational_support_frameworks_guide.md`](../operational_support_frameworks_guide.md). |
| **RPA / workflow automation** | Deterministic task automation | Classic automation executes a *known* fix; AI-driven remediation *chooses* the fix — the difference §2.3 and §8 turn on. |

Cross-reference: for the AI-agent engineering stack that the *assistive* parts of AIOps (LLM RCA, copilots, agentic remediation) borrow from, see [`enterprise_agentic_platform_architecture_guide.md`](enterprise_agentic_platform_architecture_guide.md) and [`agent_harness_engineering_guide.md`](agent_harness_engineering_guide.md); this guide uses those patterns but does not re-derive them.

---

## 2. The Scope Map and the Demarcation Lines

### 2.1 The five-stage loop — VERIFIED framing

The operational scope of AI for IT is conventionally drawn as a closed loop. The framing is remarkably stable across analyst, vendor, and open-source sources, even where the labels differ:

```
   ┌───────────────────────────────────────────────────────────────────┐
   │                                                                   │
   ▼                                                                   │
 ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌─────────┴──┐
 │ OBSERVE  │──▶│  DETECT  │──▶│ DIAGNOSE │──▶│   ACT    │──▶│   LEARN    │
 │ telemetry│   │ anomaly  │   │ root     │   │ remediate│   │ feedback,  │
 │ metrics  │   │ correlate│   │ cause,   │   │ automate,│   │ retrain,   │
 │ logs,    │   │ reduce   │   │ topology │   │ notify,  │   │ tune rules │
 │ traces,  │   │ noise    │   │ graph    │   │ escalate │   │ + runbooks │
 │ events   │   │          │   │          │   │          │   │            │
 └──────────┘   └──────────┘   └──────────┘   └──────────┘   └────────────┘
```

The mapping to the literature is direct. Gartner's market definition names the same stages — "availability and performance monitoring, event correlation and analysis, IT service management, and automation" **✅**. Microsoft Research's **AIOpsLab** describes AIOps as leveraging AI "to automate and enhance IT operations, from monitoring and anomaly detection to fault diagnosis and recovery" **✅** (Microsoft Research, *AIOpsLab*, 2024) — observe, detect, diagnose, act, in one sentence. The encyclopaedic component list adds the LEARN-stage capabilities: predictive analytics, predictive capacity management, hardware-failure prediction, auto service healing, and "using LLMs for cloud ops" **✅** (Wikipedia, *AIOps*, components). The five-stage closed-loop framing is therefore **verified as the shared convention** across analyst, vendor, and research sources — the exact stage names vary, the loop does not.

| Stage | Question it answers | AI capability | Classic antecedent |
|---|---|---|---|
| **Observe** | What is the estate *emitting*? | Unified telemetry intake, log parsing, entity mapping | Monitoring agents, collectors |
| **Detect** | Is something wrong? | Anomaly detection, event correlation, noise reduction | Threshold alerts |
| **Diagnose** | *Why* is it wrong? | Topology/causal RCA, LLM summarisation and hypothesis | Manual triage, runbooks |
| **Act** | What do we *do*? | Guarded auto-remediation, ticketing, escalation | Scripted automation |
| **Learn** | How do we get better? | Feedback into models, runbooks, golden sets | Post-incident review |

### 2.2 The pillars as one operating system

The four pillars of this guide are not four products; they are four **planes over the same loop**:

- The **AIOps plane** (§3–§5) automates Observe → Detect → Diagnose.
- The **ITSM plane** (§6) connects Detect/Diagnose to the service-management process — tickets, triage, knowledge.
- The **workforce plane** (§7–§8) covers Act and the human tools that feed it — copilots, runbooks, self-healing.
- The **governance plane** (§11) constrains Act everywhere: who may approve, what the blast radius is, what is audited.

A mature capability is one telemetry foundation feeding all four. A fragmented one buys four tools that disagree about what a "service" is.

### 2.3 The demarcation lines — what is *genuinely* new

This is the question that separates a real capability from a rebrand. Three things are genuinely new; the rest is often a feature upgrade wearing an AIOps badge.

**(a) The correlation layer.** Classic monitoring produced one alert per threshold breach; the operator did the correlating in their head. AIOps performs **event correlation** automatically — grouping the dozens of alerts that share a cause into one incident. Splunk's ITSI documents exactly this: "Event iQ applies AIOps-driven event correlation, topology, and fuzzy matching to group related alerts into episodes" **✅** (Splunk ITSI product documentation). Dynatrace documents the same intent: to "prevent a flood of seemingly unrelated problem alerts for related events," its AI "correlates all events that share the exact root cause into a single, trackable problem" **✅** (Dynatrace docs, Davis AI semantic dictionary). **The correlation is the new unit of work** — from *alert* to *episode/problem*. That is a real change in kind, not degree.

**(b) Probabilistic detection instead of static thresholds.** Thresholds encode a human's guess ("CPU > 90% for 5 minutes"). ML-based anomaly detection learns a *baseline* per entity and flags deviation from it, catching slow drifts and diurnal anomalies a fixed threshold never sees (§4.1). This is genuinely new behaviour — and genuinely harder to trust, because a probabilistic detector can be *wrong*, which a threshold can only be *badly set*.

**(c) Probabilistic diagnosis.** Classic automation executes a deterministic fix; AIOps *infers* a cause. Dynatrace's causal AI "automatically evaluates all captured and ingested information and highlights entities within the causal topology identified as the root cause" and ranks "root cause contributors," using "context information, such as the code-level information topology" **✅** (Dynatrace docs, root cause analysis). LLM-assisted RCA goes further, generating a *hypothesis* in natural language (§5.2). Diagnosis is now inference under uncertainty — powerful, and the origin of the accuracy and automation-bias problems in §10.

**What is not new.** Inventory, dashboards, runbooks, scripted remediation, and ticketing are decades old and often relabelled. The honest demarcation: **if a "capability" reduces to a rule you could have written in 2010, it is not the new part.** The new part is the ML/LLM layer, the correlation, and the probabilistic diagnosis — everything downstream of those is plumbing, and the plumbing is where most rollouts actually fail.

> **One cross-reference, not a restatement:** the discipline of operating the AI agents that *perform* these tasks — their traces, evals, guardrails, and cost — is AgentOps, owned by [`agentops_guide.md`](agentops_guide.md); this guide treats those agents as actors inside the IT loop, not as the subject.

---

## 3. Telemetry and Observability AI

### 3.1 The substrate: four signals and one problem

Every AIOps capability sits on a telemetry foundation of four signal types, and the foundation is the same whether the reasoning layer is a classic ML model or an LLM agent:

| Signal | What it is | Volume problem |
|---|---|---|
| **Metrics** | Numeric time series (CPU, latency, error rate) at a scrape interval | Cardinality explosion — one metric × thousands of labels |
| **Logs** | Structured/unstructured event text | Volume explosion — terabytes/day in a mid-size estate |
| **Traces** | The causal chain of one request across services (spans) | Sampling and cost of retention |
| **Events** | Discrete state changes from infra/cloud/k8s/CI | Duplication — the same root event emitted by ten tools |

Two conventions anchor the modern substrate. **OpenTelemetry (OTel)** standardizes the wire format and semantic conventions for traces, metrics, and logs, so a signal emitted once is portable across backends; **entity/topology modelling** (what depends on what) is what turns four piles of data into an answerable question. The AI layer is only as good as these two — a recurring theme in §10 and §12.

### 3.2 The commercial platforms — verified product facts, flagged marketing

The vendor landscape is real, mature, and heavily marketed. The product facts below are **✅** (retrieved from vendor documentation this pass); the ROI claims that surround them are **⚠** and audited in §9.

- **Datadog.** Documents an AI/agent layer — **Bits AI Agents** — including **Bits Investigation** (marketed as an "AI on-call teammate" that investigates issues), **Bits Security Analyst** (autonomous triage of Cloud SIEM signals, "reduce alert fatigue"), and **Bits Agent Builder** (custom investigation/remediation agents over Datadog + third-party data, "2,000+ prebuilt actions") — alongside the long-standing **Watchdog** anomaly engine **✅** (Datadog product docs).
- **Dynatrace.** Positions **Davis AI** as "hypermodal AI" combining **predictive AI** (forecasting/anomalies via ML on Grail), **causal AI** (topology-driven root-cause determination and automated remediation), and **generative AI** (**Davis CoPilot**, now evolving into **Dynatrace Assist**) **✅** (Dynatrace docs and platform blog). Dynatrace explicitly argues that pure generative AI is *unsuitable* for precision ops because "the same prompt/question will provide different responses," and that causal/predictive AI "provide deterministic answers and reliable automation" — a vendor claim, but an architecturally important one (§5.2).
- **Splunk (Cisco).** **ITSI** documents **zero-touch event analytics** (AI-driven field discovery and alert onboarding), **Event iQ** event correlation ("AIOps-driven event correlation, topology, and fuzzy matching to group related alerts into episodes"), and **Event iQ Diagnose** ("AI-generated episode summaries, confidence-based root cause guidance, and change context from tools like ServiceNow and Jira") **✅** (Splunk ITSI product docs).
- **Elastic.** Ships an observability AI-assistant layer and ML anomaly detection in the Elastic Stack; **⚠** the specific product page could not be fetched this pass, so Elastic is cited here as part of the landscape rather than with a verified product claim.
- **IBM.** Publishes both the AIOps definitional framing and a product stack (**Instana** observability, **IBM AIOps**) that documents the same observe → separate signal from noise → identify root cause → automate response → "learn continually" loop **✅** (IBM Think, *What is AIOps?*).

**⚠ Marketing flag.** Everything in this subsection is *vendor-published*. It proves the capability exists and describes the intended behaviour; it does **not** prove accuracy, precision, or ROI in *your* estate. The recurring marketing pattern — "reduce alert fatigue," "faster root cause," "first hypermodal AI" — should be read as *claims to evaluate*, not benchmarks.

### 3.3 The open-source option — and the honest trade-off

The open-source path is genuinely viable and is the default for data-residency-constrained estates (§12). The canonical stack:

- **Prometheus** collects and stores metrics as time series with a multi-dimensional data model, ships **PromQL** for querying, and includes **Alertmanager** for alert handling **✅** (Prometheus docs). Prometheus was built at SoundCloud and joined the **CNCF in 2016 as its second hosted project, after Kubernetes** **✅**.
- **Grafana** provides dashboards and **Grafana Alerting** ("queries and expressions from multiple data sources … combine your data and alert on your metrics and logs") **✅** (Grafana docs).
- The **LGTM stack** — **L**oki (logs), **G**rafana (viz), **T**empo (traces), **M**imir (metrics) — mirrors the commercial four-signal model **✅** (referenced in Grafana's own documentation).
- The **ML/AIOps layer on top** is where OSS is thinner: options include Grafana's own ML/anomaly features, Meta's **Prophet** and general time-series libraries, seasonal-hybrid ESD, isolation forests, and — increasingly — LLM agents wired to the telemetry store via MCP. NVIDIA/Red Hat-class reference architectures assemble these, but **⚠** there is no single turnkey open-source equivalent of a Davis/ITSI-class correlation engine; teams typically build the correlation layer.

**The trade-off, plainly.** Commercial platforms give you correlation and causal modelling *out of the box* but own your telemetry and charge by volume; open source gives you data sovereignty and no per-host AI tax but asks you to build and maintain the intelligence layer yourself. For a bank, this is usually a *hybrid*: an OSS/self-hosted telemetry core, with a commercial AIOps layer only where it is proven and where data-egress rules permit (§12.4).

### 3.4 Causal AI, as Dynatrace frames it

The most architecturally specific framing of "observability AI" in the market is **causal AI**: reasoning over a *topology* (a graph of what calls what) rather than over isolated signals. Dynatrace's version processes observability, security, and business data "in the context of causal dependencies from Dynatrace Smartscape topology to precisely determine the needle in the haystack" **✅** (Dynatrace blog). The reason this matters: correlation over a flat list of alerts can only ask "do these look related?"; correlation over a graph can ask "is this the *upstream* entity the others depend on?" — which is the difference between a plausible grouping and a *root cause* (§5.1). Even if the specific vendor is not the one you buy, the *graph-not-list* principle is the transferable design idea.

### 3.5 The binding constraint: data quality

Every capability in §3.2–§3.4 degrades to noise if the underlying telemetry is poor. The failure modes are mundane and brutal:

- **Missing labels / inconsistent entity IDs** — the same service named three ways defeats correlation and topology both.
- **Gaps** — a scrape failure looks like a metric drop; detectors that do not distinguish "no data" from "bad data" raise false alarms.
- **Clock skew** — trace stitching and causal ordering break silently.
- **High-cardinality, low-signal metrics** — cost rises, detectability does not.

The practitioner rule: **spend on telemetry hygiene before you spend on the intelligence layer.** No correlation engine — vendor or bespoke — repairs a broken entity model. §10.5 makes this a named failure mode.

> Cross-reference: the observability *engineering* stack for the AI systems themselves (OTel GenAI conventions, tracing agents) is [`agentops_guide.md`](agentops_guide.md) §3; Kubernetes/OpenShift-specific telemetry plumbing lives in the estate's platform guides and is not re-derived here.

---

## 4. Anomaly Detection and Alert-Noise Reduction

### 4.1 Statistical vs ML detection

Anomaly detection is the DETECT stage: deciding that *this* signal, at *this* time, is abnormal. The technique families, roughly in order of adoption:

| Technique | How it works | Strength | Weakness |
|---|---|---|---|
| **Static thresholds** | Alert if metric crosses a fixed bound | Trivial, explainable, deterministic | Needs per-entity tuning; misses slow drift and diurnal patterns |
| **Static + seasonal baselines** | Threshold varies by hour/day pattern | Catches diurnal anomalies | Still hand-tuned; brittle to step changes |
| **Statistical anomaly** | z-score, MAD, seasonal-hybrid ESD, EWMA over learned baseline | Cheap, interpretable, no training labels needed | Assumes a distribution; one-off legitimate spikes are false positives |
| **Classical ML** | Isolation forests, one-class SVM, clustering, change-point detection | Multivariate; learns interactions | Needs feature engineering; opaque to operators |
| **Time-series forecasting** | ML/statistical forecast (e.g. Prophet-class, LSTM) then residual threshold | Predicts *and* detects; powers capacity prediction | Forecast error itself is a source of noise |
| **Deep / LLM-assisted** | Autoencoders; LLM reading logs/metrics for "does this look wrong?" | Handles unstructured logs; explains itself | Cost, latency, non-determinism; hard to calibrate |

**Verified framing.** IBM's AIOps material lists "anomaly detection, root cause analysis (RCA), event correlation and predictive analysis" as the core ML applications, and describes the platform task as separating "signal" from "noise" **✅** (IBM Think). The encyclopaedic component list includes **anomaly detection, log analysis, cohort analysis, event correlation, predictive analytics, and hardware-failure prediction** **✅** (Wikipedia, *AIOps*). There is broad agreement that *multivariate* detection (looking at many signals together) and *baseline learning* are the genuine advance over thresholds — and equally broad agreement that calibration is the hard part.

**The calibration problem, stated honestly.** A detector has two error directions: false positives (alert on normal behaviour — the noise people already drown in) and false negatives (miss the real anomaly). Tuning one down raises the other. The vendor promise of "anomaly detection" almost never states its precision/recall on *your* data — which is why the only reliable evaluation is a backtest against your own labelled incidents (§4.3, §9.2). **⚠**

### 4.2 Alert correlation: from alert to episode

The single highest-value AIOps function in practice is **correlation** — collapsing many alerts into few incidents. The two verified vendor framings converge:

- **Splunk ITSI:** "Event iQ applies AIOps-driven event correlation, topology, and fuzzy matching to group related alerts into episodes. Instead of chasing dozens of disconnected notifications, operators can focus on a smaller set of incidents" **✅**.
- **Dynatrace:** the AI "correlates all events that share the exact root cause into a single, trackable problem" to "prevent a flood of seemingly unrelated problem alerts for related events" **✅**.

Three correlation mechanisms are worth distinguishing:

1. **Temporal co-occurrence** — alerts close in time. Cheap, high recall, low precision (a batch job and a genuine failure at 02:00 group wrongly).
2. **Topological correlation** — alerts connected through the dependency graph (§3.4). Higher precision; requires an accurate topology.
3. **Similarity/fuzzy matching** — same fingerprint, message template, or entity. Good for deduplication; blind to novel relationships.

Production correlation engines combine all three. The output is an **episode** (Splunk) or **problem** (Dynatrace) — a first-class object with severity, owner, and one notification for humans.

### 4.3 Alert-noise reduction: the public case evidence — flagged

The most-cited AIOps benefit is noise reduction, and almost every number is **vendor- or customer-published** **⚠**. Representative public examples:

- Splunk's page carries a customer story headline, "Specsavers Sees 10x Faster MTTR with Splunk," and a customer's logo list (TransUnion, ENGIE, TalkTalk, MTR) **⚠** — a *customer-published* story, not an independent study (Splunk).
- Vendor pages routinely claim "reduce alert fatigue" and "fewer false positives" as feature bullet points **⚠** (Datadog, Splunk, Dynatrace).
- The encyclopaedic summary of AI's IT impact cites "improving metrics like Mean Time to Detect (MTTD) by 15–20%" and "reduc[ing] critical incidents by over 50% through AI-driven end-to-end service management," sourced to a consultancy piece **⚠** (Wikipedia, *AIOps Results*, citing Wavestone) — plausible, and *unfalsified* by any independent measurement this pass.

**How to read these.** They establish that *practitioners report* noise reduction and MTTR gains — which is real directional evidence — but none survives as a controlled result. The honest posture for a bank is to treat every such number as a **hypothesis to backtest on our own ticket and alert history** before it enters a business case (§9.3).

### 4.4 Design guidance that survives the hype

- **Deduplicate at the edge, correlate at the centre.** Suppress known-duplicate alerts in the collector; do the expensive correlation once, centrally.
- **Make suppression auditable.** Every suppressed alert must be recorded with the episode it joined — because the alert you suppressed is the incident you will be asked about (§11.3).
- **Measure the shift, not just the drop.** Noise reduction that merely moves alerts into a less-visible queue is a failure mode, not a success (§10.3).
- **Keep a raw-alert firehose.** When correlation is wrong, the operator needs the ungrouped signal back immediately.

> Cross-reference: the identical correlation/deduplication pattern for *agent* telemetry is [`agentops_guide.md`](agentops_guide.md) §5; the ITIL incident/event-management process that these episodes feed is [`../operational_support_frameworks_guide.md`](../operational_support_frameworks_guide.md).

---

## 5. Root-Cause Analysis: Topology, Graphs, and LLM Assistance

### 5.1 Graph/topology-based RCA

The DIAGNOSE stage asks *why*. The mature, non-generative answer is **topology-based RCA**: infer the causal chain from a dependency graph plus the anomalies on its nodes. Dynatrace's causal AI "automatically evaluates all captured and ingested information and highlights entities within the causal topology identified as the root cause of a complex situation," ranks "root cause contributors," and uses "code-level information topology" for precision **✅** (Dynatrace docs). Splunk's Event iQ Diagnose adds "AI-generated episode summaries, confidence-based root cause guidance, and change context from tools like ServiceNow and Jira" **✅** — note the **confidence** qualifier and the **change context**: RCA is a probabilistic ranking, and recent change is often the strongest single clue.

Why graphs beat lists: a flat alert set can only ask "related?"; a graph can ask *direction* — which node's failure explains the others' symptoms. This is also why topology accuracy is a hard prerequisite: **wrong graph, confident wrong cause.**

### 5.2 LLM-assisted RCA

The newer layer puts an LLM (or agent) over the telemetry to *summarise, hypothesise, and explain*. Its distinct value is **natural-language synthesis** — turning a trace + logs + change record into a paragraph an operator can read at 03:00 — and **query generation** (Dynatrace: Davis CoPilot "translates natural-language questions into DQL queries"; Splunk: AI-generated episode summaries) **✅**.

Its distinct risk is **plausibility**: an LLM can produce a fluent, wrong root cause with high confidence, and because it is fluent, it is *believed*. Dynatrace's own argument is instructive precisely because it is a vendor warning against LLM-only RCA: generative AI is "not rooted in precise causal data," so "a pure generative AI approach renders use cases that require precision impossible"; its architecture therefore uses deterministic predictive/causal AI to *retrieve and rank* causes and reserves the LLM for language and interaction **✅** (Dynatrace blog). The transferable architecture lesson: **let the LLM explain the diagnosis, not make it** — or if it must decide, constrain it with retrieval over the topology and never let it act on production without a gate (§8, §11).

### 5.3 Accuracy claims — flagged ⚠

**⚠ No vendor publishes RCA precision/recall in the open that this pass could verify.** "Precise root cause," "accurate determination," and "needle in the haystack" are marketing adjectives, not measured accuracy. Two honest consequences:

- **Treat RCA output as a ranked hypothesis, not a verdict.** The UI presenting a single "root cause" is a product choice; the underlying inference is probabilistic.
- **Backtest RCA on your own incident corpus** — for each historical incident with a known cause, did the engine rank it first? That number, per env/estate, is the only accuracy claim worth quoting (§9.2). For the LLM-assisted variant the measurement is the same, plus a hallucination check on the generated explanation.

### 5.4 Method comparison

| Method | Input | Strength | Honest limitation |
|---|---|---|---|
| Manual triage | Dashboards, tribal knowledge | Context-rich, no false confidence | Doesn't scale; slow; inconsistent |
| Rule-based | Hand-written dependency rules | Deterministic, auditable | Brittle; ages badly; never complete |
| Graph/topology RCA | Dependency graph + anomalies | Direction-aware; ranks causes; deterministic | Depends entirely on graph accuracy |
| Bayesian/causal inference | Graph + probabilities + history | Probabilistic rigor; explainable priors | Needs calibrated probabilities |
| LLM-assisted RCA | Telemetry + traces + changes, retrieved | Rich summaries; NL queries; finds patterns | Hallucination risk; non-deterministic; cost/latency |
| Hybrid (vendor norm) | Deterministic AI ranks, LLM explains | Precision *and* usable language | Complexity; two systems to validate |

> Cross-reference: the failure-mode catalogue for LLM/agentic reasoning — loops, plausible-but-wrong output, silent degradation — is [`llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md); this guide applies those modes to RCA in §10.2 rather than restating them.

---

## 6. AI in ITSM: The Service Desk, Triage, and Deflection

### 6.1 The ITSM AI surface

ITSM is where AI for IT meets the *process* of IT — the ticket, the incident, the change, the knowledge article. The AI capabilities, in the order they usually earn their keep:

| Capability | What the AI does | Value mechanism |
|---|---|---|
| **Triage & classification** | Read a free-text ticket, assign category/subcategory/priority | Removes manual coding; enables routing |
| **Routing & assignment** | Send to the right team/queue/individual | Lower handoff latency, fewer misroutes |
| **Summarisation** | Condense a 40-work-note incident into 5 lines | Faster onboarding to an incident; shift handover |
| **Resolution-note generation** | Draft the fix write-up from the work trail | Cheaper documentation; better knowledge |
| **Similar-incident retrieval** | "This looks like INC0042 from March" | Reuse of prior fixes |
| **Virtual agent / deflection** | Handle L1 requests in chat/portal, self-service | Volume reduction at the front door |
| **Agent assist** | Suggest replies, next steps, KB articles to the human | Speed and consistency |
| **Auto-resolution** | Execute the fix without a human (§8) | MTTR reduction on known-class incidents |

The demarcation note from §2.3 applies with force here: **summarisation and classification are genuinely new (LLMs did not exist for this before); deflection is an old ambition (IVR, decision trees, scripted bots) that generative AI finally makes tolerable** — because a generative virtual agent no longer needs a hand-authored flow tree for every utterance. That is the real shift.

### 6.2 ServiceNow — Now Assist (verified product facts)

ServiceNow is the incumbent ITSM platform in most large banks, and its generative-AI layer is **Now Assist**. The verified capability set for **Now Assist for ITSM**:

- **Incident summarisation** — "when an incident has multiple work notes, Now Assist can generate a plain-language summary of the incident history" **✅** (ServiceNow documentation/community).
- **Resolution-notes generation**, **chat summarisation**, **AI search**, **flow and playbook generation** across ITSM products **✅** (ServiceNow datasheet and community documentation).
- **Agentic expansion** — in January 2026 ServiceNow announced partnerships with **Anthropic and OpenAI** to add those vendors' LLMs and agentic-AI features into its platform **✅** (company record).

**⚠ Vendor flag.** The capability descriptions are ServiceNow's own; the "reduce MTTR" benefit in its datasheets is vendor-published. ServiceNow's own community material is explicit that Now Assist is *assistive* by design — it drafts, summarises, and proposes; the human submits. That default is the right one for a bank (§11).

### 6.3 Microsoft — Copilot in Customer Service and Service Agent

Microsoft's service-desk AI is delivered inside **Copilot in Dynamics 365 Customer Service**, which "provides real-time AI assistance that helps customer service representatives automate tasks, resolve cases faster, and deliver better customer experiences," and is "powered by Microsoft 365 Copilot and exposed in Customer Service apps" **✅** (Microsoft Learn). The verified feature list includes **respond to questions, compose emails, draft chat responses, and summarise cases and conversations**; some capabilities are delivered through **Copilot agents**, "such as Service Agent" **✅**.

In March 2026 Microsoft introduced **Service Agent in Microsoft 365 Copilot** — a "declarative agent that runs inside Microsoft 365 Copilot, designed specifically for customer service scenarios," grounding answers in both Microsoft 365 data (Outlook, Teams, SharePoint) and Dynamics 365 service data, and able to "prioritize cases, update records, draft responses to customers, and trigger workflows" via natural language, released in **public preview** **✅** (Microsoft Dynamics 365 blog, 31 March 2026).

**⚠ Vendor flag & naming caution.** These product boundaries move fast — "Copilot for Service" (an earlier standalone SKU), "Copilot in Dynamics 365 Customer Service," and "Service Agent" are related but not identical, and Microsoft is consolidating them around Microsoft 365 Copilot as the "primary system of engagement." A bank's procurement question should be *which* surface is licensed for *which* desk **⚠**. The platform-integration engineering — Copilot Studio, Dataverse, connectors, ALM — is owned by [`copilot_studio_agent_engineering_knowledge_guide.md`](copilot_studio_agent_engineering_knowledge_guide.md) and not re-derived here.

### 6.4 Virtual agents, deflection, and knowledge

**Deflection** is the cleanest ROI story in ITSM AI and the hardest to measure honestly. A virtual agent that "resolves" a request may have (a) genuinely fixed it, (b) told the user how to fix it, (c) routed them to the right place, or (d) convinced them to give up — and only (a) and (b) are real deflection. The measurement trap is real and named in §10.4.

**Knowledge management is the unglamorous multiplier.** Every AI capability above — summarisation, retrieval, agent-assist, auto-resolution — is bounded by the quality of the KB and the CMDB behind it. LLMs make *stale, contradictory, or undocumented* knowledge more dangerous, not less, because they synthesise confidently across it. The practitioner rule: **if you cannot answer a question with a curated knowledge search today, do not expect an LLM to answer it correctly tomorrow.**

### 6.5 The ITSM loop closes into AIOps

The two pillars are one pipeline: AIOps emits correlated **episodes** (§4.2) and ranked **root causes** (§5); ITSM turns them into **incidents** with owners and, where possible, **auto-remediation actions** (§8). A bank that buys an AIOps engine but keeps a manual ticket wall in front of it has automated detection and manualised response — the classic half-rollout (§13 shows the integrated target).

> Cross-reference: the process frames (incident, problem, change, knowledge) are [`../operational_support_frameworks_guide.md`](../operational_support_frameworks_guide.md); the Cymbal Bank service-desk posture in the repo's agent guides is reused in §13, not re-derived.

---

## 7. AI for the IT Workforce: Copilots and Runbook Automation

### 7.1 The coding copilots — verified evidence

Coding copilots are the most mature, best-evidenced AI-for-IT capability, and the repo's agent cluster covers the engineering in depth; here the focus is the *operational* evidence.

**GitHub Copilot — the canonical controlled result.** In a 2022 controlled experiment, GitHub recruited **95 professional developers**, split them randomly, and timed a standard JavaScript HTTP-server task. Developers using Copilot completed it **55% faster** (average 1h11m vs 2h41m), with a **higher completion rate (78% vs 70%)**; the result was statistically significant (**P=.0017**), with a 95% confidence interval on the speed gain of **[21%, 89%]** **✅** (GitHub Blog / Kalliamvakou, 2022; updated 2024). The survey side found **60–75%** of users reported greater job fulfilment, with **73%** reporting better flow and **87%** less effort on repetitive tasks **✅**.

**⚠ How to read it.** This is a *vendor-run* experiment on a *synthetic* task with *professional* developers — strong evidence that the tool accelerates code *typing*, weaker evidence about system-level delivery outcomes (defect rates, review load, maintenance). Treat "55% faster" as a verified lab result, not a verified productivity guarantee — the same discipline §9.3 applies to every ROI number.

**Claude Code and agentic coding.** Anthropic's **Claude Code** is documented as "an agentic coding tool … understands your codebase, edits files, runs commands, and helps you ship faster" — a terminal/IDE-native agent rather than an inline-completion assistant **✅** (Anthropic, claude.com). Its launch is reported as a February 2025 research preview alongside Claude 3.7 Sonnet, reaching GA with Claude 4 **⚠** (secondary source). The distinction matters operationally: an inline copilot suggests; an agentic coder *acts* — it edits files and runs commands — which reintroduces every guardrail question of §8 and §11 into the developer's own shell.

**Cross-reference, condensed.** The repo's coding-agent engineering — harnesses, sandboxing, spec-driven development, failure modes — lives in [`coding_agents_research.md`](coding_agents_research.md), [`agent_harness_engineering_guide.md`](agent_harness_engineering_guide.md), [`agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md), and [`spec_driven_development_for_llms_guide.md`](spec_driven_development_for_llms_guide.md); this guide does not re-derive them.

### 7.2 Admin/ops copilots

The same copilot pattern applied to operators themselves:

- **Dynatrace Assist** (evolution of Davis CoPilot) — natural-language queries over telemetry, dashboard/notebook generation, workflow code suggestion **✅**.
- **Datadog Bits Chat / Bits Code** — conversational interface over observability data and code **✅** (Datadog product docs).
- **Splunk** — AI-assisted setup, episode summaries, and natural-language investigation **✅**.
- **ServiceNow Now Assist** — incident and change management assistance (§6.2) **✅**.

The value is genuine and bounded: these tools **lower the skill floor for complex query languages and lower the time-to-context at 03:00**. The risk is the same plausibility trap as LLM RCA (§5.2) — a confident natural-language answer over a mis-modelled dataset.

### 7.3 Runbook automation vs runbook *generation*

Two different things are often conflated:

- **Runbook automation** — executing a *pre-authored*, deterministic runbook (Ansible, Rundeck, ServiceNow workflows, Terraform, CI jobs). This is classic automation and has been good for a decade.
- **Runbook generation** — an LLM drafting the runbook or remediation script from a ticket, a trace, or a natural-language description (Dynatrace: "auto-coded workflows"; ServiceNow: "flow generation") **✅**.

The second is new and useful for *toil reduction* — but a generated runbook is **unreviewed code that will run against production**. The governance rule from §11 applies: a generated remediation artefact is a *change proposal*, and it enters the change-control path like any other (§8.3, §11.2).

### 7.4 The workforce reality

The honest reading of the evidence: copilots **shift** work, they do not delete it. They reduce time on routine authoring and query-writing and increase the premium on *review, judgement, and system understanding* — which is exactly the skill set an ITSM/ops function needs more of as automation grows. The automation-bias risk (§10.6) is the workforce-side failure mode: reviewers rubber-stamping plausible AI output because it is fast and usually right.

> Cross-reference: adoption strategy and the human side of AI in an enterprise is [`ai_adoption_strategies_guide.md`](ai_adoption_strategies_guide.md); this section keeps to the IT-workforce application.

---

## 8. Auto-Remediation and Self-Healing: Guarded Autonomy

### 8.1 The action ladder

The ACT stage is where AI for IT stops advising and starts *doing*. The useful model is an **autonomy ladder** — the same dial used across the repo's agent guides, applied to infrastructure actions:

| Level | Name | Who acts | Example | Blast radius control |
|---|---|---|---|---|
| **L0** | Observe only | Human | Dashboards, alerts | None (read-only) |
| **L1** | Advise | Human | "Root cause is X; runbook Y" | None (suggestion) |
| **L2** | Approve-then-act | Human approves, AI executes | Operator clicks "remediate" | Approval gate + audit |
| **L3** | Act-with-guardrails | AI acts if in policy | Restart unhealthy pod; scale out | Pre-approved action class, limits |
| **L4** | Fully autonomous | AI acts, human audits | Self-healing service mesh | Post-hoc audit, rollback, kill switch |

Most banks in 2026 sit around **L1–L2**; a narrow band of **L3** is defensible for well-understood, low-blast-radius actions; **L4 for broad infrastructure is aspirational and, for regulated workloads, usually out of bounds** **⚠** (this maturity claim is my synthesis from the practice, not a published framework).

### 8.2 The guarded-autonomy pattern

The engineering pattern that makes L3 tolerable — and it is an engineering pattern, not a policy wish — has five parts:

1. **Constrain the action space.** The remediator can only call a *pre-approved catalogue* of actions (restart, scale, drain, failover, rollback) with parameter bounds; never arbitrary shell.
2. **Blast-radius limits.** Per-action limits (max instances, max %, one change at a time), plus change-freeze respect and business-hours awareness.
3. **Idempotence and dry-run.** Every action is idempotent and can be simulated first; the simulation is logged.
4. **Precondition checks.** Act only if the *current* state still matches the precondition the rule was written for (the world changed since the model trained).
5. **Kill switch + automatic rollback.** A human can halt automation instantly; every action carries a defined reversal.

This is the IT-operations instance of the guardrail patterns in [`production_ready_llm_agents_guide.md`](production_ready_llm_agents_guide.md) and [`agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md) — the agent is powerful only inside an envelope it cannot escape.

### 8.3 Blast radius, change control, and rollback

Auto-remediation is a **change**, and in a regulated estate it must be governed as one:

- **Change authority** — which action classes may run autonomously, at which autonomy level, approved by which change advisory function. This is a *policy artefact*, versioned and audited.
- **Segregation of duties** — the person who authors a remediation rule should not be the only approver of the autonomy level that lets it run.
- **Audit trail** — every autonomous action records: trigger (episode/RCA), actor (automation identity), action, parameters, before/after state, outcome, and the human accountable for the rule.
- **Rollback and containment** — a defined, tested reversal; automation that cannot be undone should not run at L3.
- **Emergency stop** — the kill switch is itself a tested control with a named owner (§11.4).

### 8.4 Self-healing infrastructure — maturity, flagged

"Self-healing" spans three very different maturities, and vendor marketing blurs them **⚠**:

- **Mature:** Kubernetes-native self-healing (restarting failed containers, rescheduling pods, liveness/readiness probes, replica reconciliation). This is *declarative* control, decades of lineage, and genuinely works. It is not AI, though it is often sold alongside AIOps.
- **Maturing:** AIOps-triggered remediation (an episode triggers a pre-approved runbook) — the guarded-autonomy L3 pattern above; increasingly real in 2024–2026 **⚠**.
- **Emerging/aspirational:** LLM agents that *diagnose and fix novel problems* autonomously. Mostly lab-grade (e.g. Microsoft Research's **AIOpsLab**, a research framework for evaluating autonomous AIOps agents) **✅** as research, **⚠** as production practice.

The honest 2026 line: **self-healing for known failure classes with approved actions is real and valuable; self-healing for unknown problems is research.** Claiming the latter sells the former short.

> Cross-reference: Kubernetes/OpenShift operational self-healing mechanics live in the estate's platform guides (and are not re-derived); the agentic-control-loop theory behind autonomous remediation is [`autonomous_agents_guide.md`](autonomous_agents_guide.md) and [`agent_harness_engineering_guide.md`](agent_harness_engineering_guide.md).

---

## 9. Evidence and Metrics: MTTD, MTTR, and the Published Numbers

### 9.1 The metric vocabulary

Before auditing any AIOps claim, fix the vocabulary — much of the disagreement in this domain is people quoting different metrics under one label.

| Metric | Definition | How AIOps is claimed to move it |
|---|---|---|
| **MTTD** | Mean time to **detect** — incident start → detection | Faster/earlier detection via anomaly ML and prediction |
| **MTTA** | Mean time to **acknowledge** — detection → human owns it | Better routing, one correlated episode instead of 40 alerts |
| **MTTR** | Mean time to **resolve** — start → fix confirmed | Faster diagnosis (RCA) + faster action (auto-remediation) |
| **MTBF** | Mean time between failures | Fewer incidents via proactive/predictive action |
| **Alert volume** | Alerts per period reaching a human | Correlation/dedup reduces the count |
| **Noise ratio / actionability** | % of alerts needing action | Precision improvement |
| **Precision / recall** | Detection correctness (false positives vs misses) | The metric vendors rarely publish |
| **Auto-resolution rate** | % of incidents fixed without a human | The guarded-autonomy L3 metric |
| **Deflection rate** | % of contacts resolved without a ticket/agent | The ITSM front-door metric — and the most abused |

The two that matter most and are quoted least are **precision/recall** (does the detector actually work?) and **sustained** MTTR (does the gain survive the novelty period?).

### 9.2 The only trustworthy evaluation: backtest

Because no vendor's numbers bind to your estate, the discipline is **backtest on your own history**:

1. **Replay your alert stream** through the correlation engine. Measure: alerts-in → episodes-out, and how many episodes were actually one incident.
2. **Replay a labelled incident corpus** through the RCA engine. Measure: for each incident with a known cause, did the engine rank it first / in the top 3?
3. **Backtest detectors** on labelled normal and abnormal periods. Measure precision/recall and the false-positive rate *per operator per night* (the number that actually drives fatigue).
4. **Pilot auto-remediation on a sandbox + one low-blast-radius action class**, measure success rate and (crucially) the rate of actions taken that a human would *not* have taken.
5. **Measure the tool's own cost and latency** — inference, storage, per-host licences — against the toil saved (§12.3).

This is more work than accepting a datasheet number, and it is the only evaluation that predicts *your* outcome.

### 9.3 The vendor-number audit

| Claim | Source type | Status |
|---|---|---|
| AIOps defined by Gartner, 2016 | Analyst (via encyclopaedic record) | **✅** |
| Gartner market definition ("combine big data and AI/ML …") | Analyst (Market Guide, via ServiceNow copy) | **✅** |
| GitHub Copilot: 55% faster task completion, P=.0017, N=95 | Vendor research (controlled experiment) | **✅** (result), **⚠** (generalisation) |
| MTTD improved 15–20% by AI monitoring | Consultancy piece (via Wikipedia) | **⚠** |
| Critical incidents reduced "over 50%" via AI-driven ITSM | Consultancy piece (via Wikipedia) | **⚠** |
| "Specsavers 10x faster MTTR" | Vendor customer story | **⚠** |
| "Reduce alert fatigue" / "fewer false positives" | Vendor feature bullets | **⚠** |
| "First hypermodal AI," "precise root cause" | Vendor marketing | **⚠** |
| Gartner "by 2026, X% of enterprises will…" AIOps adoption | Secondary citation | **❌** not verified this pass |

**The honest summary.** The *directional* evidence — that AI-augmented operations detect earlier, correlate more, resolve some incidents faster — is credible and widely reported. The *magnitude* evidence is almost entirely vendor-published and unfalsified. A bank's business case must therefore cite the direction as a hypothesis (§9.2) and its own backtest as the number (**⚠**).

### 9.4 Adoption studies — flagged

**⚠ This pass could not verify a single methodologically rigorous, peer-reviewed adoption study of AIOps with reproducible magnitude figures.** The available material is analyst market-sizing, vendor case studies, and practitioner surveys — all with selection or sponsorship biases. The citation discipline for a bank deliverable: quote adoption *patterns* only with the sponsorship disclosed, and never let a sponsored survey stand in for a measured outcome. A fuller honest statement is in §15.

> Cross-reference: how the repo treats vendor ROI and evaluation discipline generally is [`ai_adoption_strategies_guide.md`](ai_adoption_strategies_guide.md); the FinOps side of AIOps tool cost is [`../finops_guide.md`](../finops_guide.md).

---

## 10. Failure Modes and Critiques

The single most important section for a sceptical reader. AI for IT fails in specific, nameable ways — and every one has a mitigation.

### 10.1 Hallucinated remediation

An LLM-driven remediator invents an action that does not exist or applies it wrongly — "restart the primary database" as a fix for a replication lag. This is the highest-severity failure: AIOps that *acts* can cause the outage it was meant to prevent.

**Mitigation:** never let a generative model call arbitrary actions; constrain to a pre-approved catalogue with parameter bounds (§8.2); dry-run and precondition-check; require approval above a blast-radius threshold; log everything for rollback.

### 10.2 False correlation (and confident wrong RCA)

Correlation that groups unrelated failures hides the real cause inside an episode; RCA that ranks the wrong node as root cause sends responders in the wrong direction. Splunk itself qualifies its output as "**confidence-based** root cause guidance" **✅** — a tell that the underlying inference is probabilistic. Dynatrace's own architecture warns that generative AI is "not rooted in precise causal data," making pure-LLM precision impossible **✅**.

**Mitigation:** keep the raw alert stream retrievable (§4.4); surface confidence and ranked alternatives rather than a single verdict; measure RCA top-1/top-3 accuracy on your incident corpus (§9.2); never suppress without an audit record.

### 10.3 Alert fatigue shifting, not disappearing

The subtlest failure: AIOps reduces the *alert count* but the remaining alerts are **harder** — a correlated episode can be a 40-alert mega-incident that is harder to reason about than any single alert, and suppression can hide the one signal that mattered. Noise *volume* falls; noise *cost per event* can rise.

**Mitigation:** measure actionability, not just volume (§9.1); keep suppression auditable; sample suppressed alerts and verify they were genuinely redundant; watch the "episodes with no actionable content" rate.

### 10.4 The deflection measurement trap

A virtual agent that closes a ticket without resolving the need is not a deflection, it is a *bounce* — the user returns, angrier, with a reopened ticket. Vendors count the closed ticket; the customer counts the experience.

**Mitigation:** define deflection as *sustained* resolution (no re-contact within a window); measure reopen rate and CSAT alongside deflection; audit a sample of "resolved" conversations.

### 10.5 Data-quality dependency

Every capability above is bounded by telemetry and knowledge quality (§3.5, §6.4). A broken entity model defeats correlation; stale KB articles make LLM answers confidently wrong. This is the failure that most often *looks like* a tool problem and is actually a data problem.

**Mitigation:** fund telemetry hygiene and CMDB/KB curation *first*; make data-quality metrics (label coverage, entity resolution rate, KB freshness) first-class dashboards; gate AIOps expansion on data-quality thresholds.

### 10.6 Automation bias

Humans over-trust the machine. An operator who has seen the AI be right 95 times stops checking the 96th — which is exactly the one that caused the incident. Automation bias compounds with fluency: a confidently worded LLM diagnosis is *more* trusted than a terse one.

**Mitigation:** keep humans meaningfully in the loop on high-risk actions (not rubber-stamp approvals); periodically inject control cases that test whether reviewers are still reviewing; rotate "adversarial reviewer" duty; measure the rate at which humans override the AI, and investigate both directions (over- and under-trust).

### 10.7 The AIOps stack has its own operations problems

The AI layer is a system too, and it drifts: models trained on last year's normal flag this year's normal as anomalous; thresholds decay as the estate changes; LLM prompts and model versions change behaviour under you. **⚠** This is the *inverse* of AgentOps — the AIOps models are themselves production ML assets needing monitoring, retraining, and versioning, and most AIOps rollouts under-invest here.

**Mitigation:** monitor detector precision over time (concept drift); re-baseline after major change; version and evaluate prompts/models for the generative layers; treat the AIOps stack as a service with its own SLOs. (The engineering discipline for operating AI systems is [`agentops_guide.md`](agentops_guide.md).)

> Cross-reference: the general LLM failure taxonomy these modes instantiate is [`llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md); the drift mechanics are [`ai_agent_drift_guide.md`](ai_agent_drift_guide.md). This section maps them to IT operations; it does not restate the taxonomy.

---

## 11. Governance, Risk, and the Regulated Estate

### 11.1 The control set

Letting AI observe, diagnose, and act on production in a regulated estate requires controls, not enthusiasm. The minimum control set:

| Control | What it enforces | Artefact |
|---|---|---|
| **AI action register** | No unreviewed automation on production | Inventory of automated action classes + owners |
| **Human-in-the-loop policy** | Which action classes need approval, by whom | Versioned autonomy policy per action class |
| **Autonomy limits** | Max blast radius per action (§8.2) | Per-action parameter bounds |
| **Change control** | AI actions are governed changes | Change record per autonomous action class |
| **Audit trail** | Every AI action is reconstructable | Append-only, trace-linked event log |
| **Kill switch** | Automation can be halted instantly | Tested mechanism + owner |
| **Model/prompt versioning** | Behaviour changes are traceable | Registry with version + validation record |
| **Data protection** | Telemetry/tickets with PII are handled lawfully | Redaction, retention, tenancy policy |

The load-bearing point: **none of these are AI-specific inventions.** They are the estate's existing change-management, audit, and operational-resilience controls, extended to a new class of actor (the AI). That framing is what makes them defensible to a risk committee.

### 11.2 Change control for AI actions

A bank's change-management function already asks "what is this change, who approved it, what is the rollback, what is the window?" An autonomous remediation answers all four — but the *rule* is the change, not each invocation. So the governance model is:

- **Approve the rule once** (action class, preconditions, blast radius, rollback, autonomy level), through normal change governance.
- **Each invocation is logged and attributed** to the approved rule and its accountable owner.
- **Rule changes are changes** — re-reviewed, re-approved, versioned.
- **Exception handling** — the escape hatch from automation (a human override) is itself a control with an audit record.

### 11.3 Audit trail and the "explain this incident" test

For a bank, the AIOps audit trail must answer, for any incident: *what did the AI see, what did it conclude, what did it do, under whose authority, with what data and version?* The design that passes this test stores: the correlated episode, the ranked root causes with confidences, the actions taken (human or AI), the autonomy policy version at the time, and the accountable owner. Splunk's "change context from tools like ServiceNow and Jira" **✅** is the same idea from the vendor side — join the AI's conclusion to the change that may have caused it.

**⚠ Practice flag.** Audit-trail completeness for *AI-driven* operations is not yet a standard product feature; most platforms log plugin actions but not the full decision chain (why this RCA, why this confidence). Expect to build the decision-log join yourself, and budget for it.

### 11.4 Operational resilience and the kill switch

Regulators (and any competent CISO) will ask: *if the AI misbehaves at scale, how fast can you turn it off, and what happens to the estate when you do?* Two obligations follow:

- **The kill switch is a tested control**, exercised in disaster-recovery/operational-resilience drills, with a named owner — not a config flag nobody has flipped in anger.
- **Fallback to manual operation is rehearsed.** If AI-driven triage/remediation is disabled, humans must be able to run the estate — the automation must not have *eroded* the ability to operate without it (a real risk once tribal runbook knowledge is replaced by an opaque engine).

Operational-resilience expectations (impact tolerances, severe-but-plausible scenarios) apply here and are owned by [`../../banking/operational_resilience_framework_guide.md`](../../banking/operational_resilience_framework_guide.md).

### 11.5 The regulated-environment constraints

For a bank specifically:

- **Model risk management** — AIOps models (detectors, RCA, LLM assistants) are models in the SR 11-7-class sense: they need inventories, validation, and ongoing monitoring — the same machinery the repo applies to AI agents in [`ai_governance_framework_guide.md`](ai_governance_framework_guide.md) and [`implementing-responsible-ai.md`](implementing-responsible-ai.md).
- **Data residency and cross-border movement** — telemetry, logs, and tickets contain customer data; the AIOps layer's data egress is a regulated decision, and it is a common reason to choose a self-hosted/OSS core (§12.4). Microsoft's own Copilot docs make cross-region data movement an explicit admin toggle **✅** (Microsoft Learn) — a bank must decide that toggle deliberately, not by default.
- **Third-party/outsourcing risk** — a SaaS AIOps vendor is a material service provider; exit, concentration, and incident-notification obligations apply.
- **Explainability and contestability** — an adverse outcome (a botched auto-remediation) must be explainable to an auditor, which is another argument for conservative autonomy levels.

**⚠ Practice flag.** Regulation is not uniform: the specific obligations (MAS, EBA, SR 11-7, DORA-style resilience) differ by jurisdiction, and none of them names "AIOps" — the controls map onto existing expectations rather than new AI-specific rules. That mapping must be done locally by the compliance function, not assumed from this guide.

### 11.6 Data protection inside the AIOps pipeline

Telemetry and tickets are a data estate. Redact PII at ingestion; minimise what crosses a trust boundary; set retention (traces are expensive *and* sensitive); isolate tenants/LOBs. The AIOps pipeline is, in data-protection terms, indistinguishable from any other system holding customer data — and it is often built *first* and secured *later*, which is the wrong order.

> Cross-reference: agent/LLM governance is [`ai_governance_framework_guide.md`](ai_governance_framework_guide.md) and [`implementing-responsible-ai.md`](implementing-responsible-ai.md); banking regulatory context is [`../../banking/ai_genai_banking_compliance_guide.md`](../../banking/ai_genai_banking_compliance_guide.md) and [`../../banking/mas_regulations_guidelines_guide.md`](../../banking/mas_regulations_guidelines_guide.md) — condensed here, not re-derived.

---

## 12. Build vs Buy, Cost, and the Operating Model

### 12.1 The build-vs-buy decision

The realistic options are not "build a Dynatrace" or "buy everything"; they are three positions on a spectrum.

| Position | What it means | Fits when | Risks |
|---|---|---|---|
| **Buy (full platform)** | Commercial AIOps/observability suite, SaaS or on-prem licence | Speed matters; no appetite to build the ML layer; data egress is permissible | Vendor lock-in; per-host/volume cost; telemetry leaves the boundary |
| **Build (OSS core + bespoke AI)** | Prometheus/Grafana/Loki/Tempo + in-house ML/LLM layer | Data sovereignty is mandatory; strong platform-engineering team; cost-sensitive at scale | You own the correlation quality; thin off-the-shelf RCA; slow to value |
| **Hybrid (the bank norm)** | OSS/self-hosted telemetry core + commercial AIOps layer only where proven; LLM assistants inside the perimeter | Regulated estate with mixed requirements | Integration complexity; two governance surfaces |

The decision criteria, in order of weight for a bank: **data residency → integration with the incumbent ITSM (ServiceNow-class) → RCA/correlation quality → cost at your telemetry volume → exit/portability**. Note that "which has the best AI" is deliberately *fourth*; the first three decide whether you can use any of it.

### 12.2 Cost drivers — the ones that surprise budgets

| Cost | Driver | Why it surprises |
|---|---|---|
| **Telemetry ingest & retention** | Metrics cardinality, log volume, trace retention | Often the *largest* line item, and grows with the estate, not with value |
| **Per-host / per-GB AIOps licence** | Number of hosts/entities or data volume | Scales with the infrastructure you already have |
| **LLM inference** | Tokens per RCA/summary/agent task × volume | Non-deterministic loops can multiply cost; needs per-task caps |
| **Storage & compute for ML** | Feature stores, model training, anomaly backends | Under-estimated; usually an ongoing opex, not one-off |
| **Integration & data engineering** | CMDB/entity model, ITSM connectors, data hygiene | The project that actually consumes the budget |
| **People** | Platform engineers, ML/LLM engineers, SRE time | The recurring cost that outlives the licence |

The FinOps discipline for AIOps is the same as for any AI opex: **meter everything, attribute per service/LOB, cap the non-deterministic paths** ([`../finops_guide.md`](../finops_guide.md)).

### 12.3 The honest ROI model

A defensible AIOps business case does not start from a vendor's "10x." It starts from *your* baseline:

```
  Annual value ≈ (incidents_avoided × cost_per_incident)
               + (MTTR_reduction_hours × loaded_hourly_cost × incident_volume)
               + (alerts_eliminated × minutes_per_alert × loaded_hourly_cost)
               + (deflected_contacts × cost_per_contact)
  Annual cost  ≈ telemetry_ingest + licences + inference + storage + people + integration_amortised
```

Populate every term with **measured** (not claimed) values from the §9.2 backtest and a pilot. The rule that keeps the case honest: **the toil-saved side must be measured on the same estate, over the same period, as the cost side — and the "sustained" qualifier applies to both** (does the alert reduction *stay* reduced after month six?).

### 12.4 The hybrid bank pattern (illustrative architecture)

A pragmatic regulated-estate architecture:

- **Collect once** — OTel-based agents emit metrics/logs/traces to a **self-hosted** store inside the boundary (Prometheus/Thanos or equivalent, Loki, Tempo/Jaeger). No raw customer-data telemetry leaves the perimeter.
- **Reason locally** — correlation, anomaly detection, and (where used) LLM assistants run on the in-boundary store; a commercial AIOps layer is admitted only for functions that cannot be built and only over de-identified/aggregated data, if at all.
- **Act through the approved channel** — remediation calls go through the existing orchestration/ITSM (ServiceNow, Ansible, Terraform, CI) as governed changes (§11.2), never direct from the AI to the shell.
- **Govern centrally** — one AI action register, one autonomy policy, one audit trail, regardless of how many tools sit underneath.

### 12.5 The operating model: who runs AI for IT

AIOps is not a tool installation; it is an operating model. The roles that must exist:

| Role | Owns |
|---|---|
| **Platform/observability engineer** | Telemetry pipeline, entity model, data quality |
| **AI/ML engineer (or SRE-with-ML)** | Detectors, RCA models, LLM prompts, drift monitoring |
| **SRE/operations** | Episodes, runbooks, auto-remediation rules, kill switch |
| **ITSM/service owner** | Ticket integration, triage/deflection quality, KB/CMDB curation |
| **Risk/compliance partner** | AI action register, autonomy policy, audit, model risk |
| **FinOps/owner** | Cost attribution, caps, build-vs-buy review |

The failure pattern to avoid: AIOps bought as a *project* with no standing owner, so the models drift, the entity model rots, and the tool becomes another dashboard nobody trusts. **It is a capability with an owner, or it is shelfware.**

> Cross-reference: the lifecycle/governance of the tooling itself follows [`../technology_lifecycle_management_guide.md`](../technology_lifecycle_management_guide.md); the agent-engineering roles for the assistive layer are described in the repo's platform guides.

---

## 13. Worked Example: An AIOps Rollout at Cymbal Bank

> **⚠ All numbers in this section are ILLUSTRATIVE.** This is a designed scenario that shows the *shape* of a rollout — foundations, the detection/diagnosis layer, guarded auto-remediation, ITSM integration, governance gates, and the target metrics. The structure generalises; the figures do not. Cymbal Bank is the repo's fictional persona (see the repo's Cymbal Bank conventions), and no figure here is a verified benchmark.

### 13.1 The scenario

Cymbal Bank's IT operations run a hybrid estate: core banking on a mainframe-adjacent stack, a Kubernetes/OpenShift digital-banking platform, a large ServiceNow ITSM instance, and a mixture of monitoring tools accumulated over a decade. The pain, as the CIO's office states it: **alert volume is unmanageable** (roughly 4,000 alerts/day reaching operators, most non-actionable), **MTTR on the digital channel averages ~6 hours** (dominated by manual triage and correlation), and **ticket triage is manual** with a rising L1 backlog. The mandate: stand up a governed AIOps capability in four phases over 12 months, with a hard rule that **no autonomous action runs without an approved rule, a blast-radius cap, and an audit trail**.

### 13.2 Phase 1 — Telemetry foundation (months 1–3)

The first phase does *no AI*. It fixes the substrate (§3.5), because everything downstream depends on it.

| Deliverable | What it does | Illustrative target |
|---|---|---|
| Unified telemetry intake | OTel-based collection of metrics/logs/traces from digital + core channels | 90% of in-scope services instrumented |
| Entity model / topology | Service→host→dependency graph, one canonical ID per entity | ≥95% entity-resolution rate |
| Data-quality dashboard | Label coverage, gap detection, clock-skew alerts, KB freshness | Coverage ≥90% before Phase 2 |
| Raw alert pipeline | Normalise all legacy monitoring feeds into one stream | 100% of feeds ingested, none dropped |

**Gate to Phase 2:** entity-resolution ≥95% and label coverage ≥90% on the pilot domain. *No correlation engine repairs a broken entity model; the gate is non-negotiable (§10.5).*

### 13.3 Phase 2 — Detect and diagnose (months 4–7)

Now the ML/AI layer, on the fixed substrate:

- **Detectors** — baseline-learning anomaly detection on the pilot domain's metrics/logs, alongside retained static thresholds (never rip out the working threshold first).
- **Correlation engine** — temporal + topological + fuzzy matching (§4.2), emitting **episodes** rather than alerts.
- **RCA** — topology-based ranked root causes, plus an LLM assistant that *summarises* the episode and *explains* the ranked cause (the "LLM explains, deterministic AI ranks" architecture of §5.2).
- **Backtest discipline** — every engine is evaluated on Cymbal's own 12-month incident corpus before it is trusted (§9.2).

| Illustrative metric | Baseline | Phase-2 target |
|---|---|---|
| Alerts/day reaching operators | ~4,000 | ≤800 (correlated episodes + alerts) |
| Episode→incident accuracy | n/a | ≥85% on backtest |
| RCA top-3 accuracy | n/a | ≥80% on labelled corpus |
| Detector false-positive rate | high (thresholds) | ≤15% |

**Gate to Phase 3:** RCA top-3 ≥80% and detector FP ≤15% on the backtest, plus a documented audit trail for every episode.

### 13.4 Phase 3 — Guarded auto-remediation (months 7–10)

The autonomy ramp, deliberately narrow (§8):

| Action class | Autonomy level | Blast-radius cap | Rollback |
|---|---|---|---|
| Restart unhealthy stateless pod | **L3** (auto) | ≤5 instances, one at a time | k8s reconciliation; drain first |
| Scale out stateless service | **L3** (auto) | ≤2× current replicas, business-hours aware | Scale-in rule |
| Restart a core-banking batch job | **L2** (approve) | single job, window-only | Restart/checkpoint |
| Restart/failover a database node | **L1** (advise only) | n/a — human executes | DBA runbook |

Every L3 action runs through the guarded-autonomy pattern (§8.2): pre-approved catalogue, idempotent, dry-run logged, precondition-checked, kill-switch-armed. Illustrative target: **L3 auto-resolves ≥60% of the pilot domain's known-class incidents**, with a human override available at all times.

**Gate to Phase 4:** 90 days of L3 operation with zero un-rolled-back failures, kill switch tested in a resilience drill, and audit-trail completeness at 100% for autonomous actions.

### 13.5 Phase 4 — ITSM and workforce integration (months 10–12)

- **Now Assist-class summarisation** on incidents (§6.2) and **AI triage/routing** on the L1 queue.
- **Agent-assist** for the service desk; a **virtual agent** for the top-20 request types, with simulated deflection measured honestly (§10.4 — re-contact within 72h counts as a bounce, not a deflection).
- **Ops copilots** (§7.2) for SREs and shift handover.
- **Knowledge/CMDB curation** as a standing workstream (§6.4).

### 13.6 MTTR targets — ILLUSTRATIVE

| Metric | Baseline (illustrative) | 12-month target (illustrative) | Mechanism |
|---|---|---|---|
| **MTTD** (digital channel) | ~25 min | ≤5 min | Anomaly detection + correlation |
| **MTTA** | ~40 min | ≤10 min | One episode, routed to owner |
| **MTTR** (digital, known-class) | ~6 h | ≤90 min | Auto-remediation + RCA |
| **MTTR** (novel incidents) | ~6 h | ~4 h | LLM episode summaries cut triage time |
| **Alerts/day to humans** | ~4,000 | ≤800 | Correlation/dedup |
| **L3 auto-resolution rate** | 0% | ≥60% of known-class | Guarded autonomy |
| **Ticket auto-triage accuracy** | manual | ≥85% agreement with human triage | Classification |

### 13.7 The governance gates (non-negotiable)

| Gate | Evidence required to pass |
|---|---|
| **G1 — Foundation** | Entity-resolution ≥95%, label coverage ≥90% |
| **G2 — Trust** | RCA top-3 ≥80%, detector FP ≤15%, backtest documented |
| **G3 — Autonomy** | Approved action register, blast-radius caps, rollback tested, kill switch drilled |
| **G4 — Audit** | 100% decision-chain logging for AI actions; "explain this incident" test passed |
| **G5 — Expansion** | Cost-per-incident ≤ baseline, sustained ≥3 months; no alert-fatigue *shift* (§10.3) |

Any gate failed = the phase does not advance. This is what makes the rollout a *governed capability* rather than an unmanaged experiment.

### 13.8 The honest caveat

The shapes generalise — telemetry first, then detection/diagnosis, then narrow guarded autonomy, with governance gates between; MTTR collapses on *known-class* incidents and only modestly on novel ones; alert volume falls but the residual must be measured for actionability. The *numbers* do not: Cymbal Bank's figures are a designed illustration with plausible magnitudes, and **your backtest (§9.2) is the only number that counts**.

> Cross-reference: the persona conventions and worked examples used across the repo are set by the Cymbal Bank guides in `banking/`; the agent-side Cymbal scenarios (customer-support agents, service-desk agents) are reused, not re-derived, from [`multi_agent_banking_guide.md`](multi_agent_banking_guide.md) and [`production_ready_llm_agents_guide.md`](production_ready_llm_agents_guide.md).

---

## 14. Claims Audit and What Could Not Be Verified

### 14.1 Claims Audit

The consolidated ledger of the material claims in this guide, with their verification status and section. `✅` = confirmed at the cited primary source this pass; `⚠` = vendor-published, approximate, single-source, or not re-verified; `❌` = could not be verified / contradicted.

| # | Claim | Status | § |
|---|---|---|---|
| 1 | AIOps was first defined/coined by Gartner in 2016 | ✅ | 1.2 |
| 2 | Gartner's market definition: "combine big data and AI or machine learning functionality to enhance and partially replace a broad range of IT operations processes and tasks…" | ✅ | 1.2 |
| 3 | Gartner's *original* 2016 expansion was "Algorithmic IT Operations" | ⚠ | 1.2 |
| 4 | AIOps is an analyst-created market category (framing flag) | ⚠ | 1.2 |
| 5 | The observe → detect → diagnose → act → learn loop is the shared framing (Gartner definition, Microsoft AIOpsLab, IBM, encyclopaedic sources) | ✅ | 2.1 |
| 6 | Microsoft Research AIOpsLab frames AIOps as "monitoring and anomaly detection to fault diagnosis and recovery" | ✅ | 2.1 |
| 7 | Event correlation groups alerts into episodes/problems (Splunk Event iQ, Dynatrace Davis) | ✅ | 2.3, 4.2 |
| 8 | Dynatrace "Davis AI" = predictive + causal + generative ("hypermodal AI"); causal AI ranks root-cause contributors | ✅ | 3.2, 5.1 |
| 9 | Dynatrace states pure generative AI is unsuitable for precision ops ("same prompt … different responses") | ✅ | 3.2, 5.2 |
| 10 | Datadog ships Bits AI Agents (Investigation, Security Analyst, Agent Builder) and Watchdog | ✅ | 3.2 |
| 11 | Splunk ITSI ships zero-touch event analytics, Event iQ correlation, Event iQ Diagnose (confidence-based RCA) | ✅ | 3.2, 4.2, 5.1 |
| 12 | IBM documents the AIOps loop and lists anomaly detection, RCA, event correlation, predictive analysis as core ML applications | ✅ | 3.2, 4.1 |
| 13 | Prometheus is an OSS metrics/alerting toolkit built at SoundCloud, CNCF's second hosted project (2016, after Kubernetes) | ✅ | 3.3 |
| 14 | Grafana Alerting queries multiple data sources and alerts on metrics/logs | ✅ | 3.3 |
| 15 | There is no single turnkey open-source equivalent of a Davis/ITSI-class correlation engine | ⚠ | 3.3 |
| 16 | No vendor publishes verifiable RCA precision/recall | ⚠ | 5.3 |
| 17 | MTTD improved 15–20% by AI monitoring | ⚠ | 4.3, 9.3 |
| 18 | "Critical incidents reduced over 50%" via AI-driven ITSM | ⚠ | 4.3, 9.3 |
| 19 | "Specsavers 10x faster MTTR" (vendor customer story) | ⚠ | 4.3, 9.3 |
| 20 | GitHub Copilot: 55% faster task completion, N=95, P=.0017, CI [21%,89%] | ✅ (result) / ⚠ (generalisation) | 7.1, 9.3 |
| 21 | ServiceNow Now Assist for ITSM provides incident summarisation, resolution notes, chat summarisation, AI search, flow generation | ✅ | 6.2 |
| 22 | ServiceNow announced Anthropic and OpenAI LLM partnerships (Jan 2026) | ✅ | 6.2 |
| 23 | Microsoft Copilot in Dynamics 365 Customer Service is powered by Microsoft 365 Copilot; Service Agent is a Copilot agent | ✅ | 6.3 |
| 24 | Microsoft Service Agent in Microsoft 365 Copilot released in public preview (March 2026) | ✅ | 6.3 |
| 25 | Microsoft exposes cross-region Copilot data movement as an admin toggle | ✅ | 11.5 |
| 26 | Claude Code is an agentic coding tool (edits files, runs commands) | ✅ | 7.1 |
| 27 | Claude Code launched as a Feb-2025 research preview; GA with Claude 4 | ⚠ | 7.1 |
| 28 | Kubernetes-native self-healing is mature, non-AI, declarative control | ✅ | 8.4 |
| 29 | AIOps-triggered remediation (L3) is maturing; LLM autonomous remediation is largely research-grade | ⚠ | 8.1, 8.4 |
| 30 | No methodologically rigorous peer-reviewed AIOps adoption study with reproducible magnitudes found | ⚠ | 9.4, 15 |
| 31 | Gartner "by 2026, X% of enterprises…" AIOps adoption stat | ❌ | 9.3, 15 |
| 32 | Audit-trail completeness for AI-driven operations is not yet a standard product feature | ⚠ | 11.3 |
| 33 | Regulators do not name "AIOps"; controls map onto existing expectations | ⚠ | 11.5 |

Reading the ledger: the *definitional and product* claims are mostly ✅ (they are checkable vendor/analyst facts); the *benefit magnitude* claims are overwhelmingly ⚠ (vendor-published, unfalsified); and the one ❌ is a widely repeated analyst statistic this pass could not source. That distribution is itself the finding: the domain is real, the ROI is asserted.

---

### 14.2 What Could Not Be Verified

Honest residue — the things this guide asserts cautiously or declines to assert at all.

- **A Gartner primary source for the 2016 definition.** The year (2016) and the market definition are **✅** via the encyclopaedic record and the ServiceNow-hosted *Market Guide* copy, but this pass could not retrieve the original 2016 Gartner note directly. The precise original wording (including the "Algorithmic IT Operations" reading, ⚠) is therefore unresolved.
- **Any independent, reproducible AIOps ROI study.** No peer-reviewed or vendor-neutral study with reproducible magnitude figures for alert reduction, MTTR, or incident avoidance was verifiable this pass. Every magnitude in §4.3 and §9.3 traces to a vendor, a customer story, or a consultancy — always disclose the sponsorship.
- **The widely cited "Gartner by 2026 X% will use AIOps" adoption statistic** — could not be sourced to a verifiable Gartner page; marked **❌**, not repeated in the body.
- **RCA accuracy for any commercial engine.** "Precise root cause" is marketed; no precision/recall figures were found at a primary source. Measure your own (§9.2).
- **Elastic's specific observability-AI feature set.** The product page could not be fetched this pass; Elastic is cited only as part of the landscape, without a verified product claim.
- **A single open-source correlation engine of vendor quality.** No turnkey OSS "Davis/ITSI-equivalent" was verifiable; the correlation layer is typically bespoke.
- **The "Copilot for Service" vs "Copilot in Dynamics 365 Customer Service" vs "Service Agent" product-boundary mapping.** Microsoft's surfaces overlap and are consolidating; the exact current SKU boundaries were not fully verifiable this pass (⚠).
- **Claude Code's launch/GA dates** — taken from secondary sources (⚠); the product's description is verified at Anthropic, the timeline is not.
- **Exact current pricing / per-host cost for commercial AIOps platforms** — not published in a form verifiable this pass; the cost model in §12.2 is directional only.
- **The four-pillar taxonomy and the autonomy-ladder maturity claim** — my synthesis as a solution architect, not a published framework (⚠ throughout).

Where a reader needs a hard number, the guide's position is consistent: **backtest it on your own estate; every other number is a hypothesis.**

---

## 15. Glossary and Closing

### 15.1 Glossary

| Term | Definition | § |
|---|---|---|
| **AI for IT / AIOps** | The application of AI/ML and LLM/agentic technology to IT operations themselves; AIOps (coined by Gartner, 2016) is its best-known label | 1 |
| **ITOA** | IT Operations Analytics — the pre-AIOps discipline AIOps generalized | 1.2 |
| **Observability** | Understanding a system's internal state from its outputs (traces, metrics, logs) | 3.1 |
| **Telemetry** | The raw signals an estate emits: metrics, logs, traces, events | 3.1 |
| **OpenTelemetry (OTel)** | Standardized wire format/semantic conventions for telemetry, for backend portability | 3.1 |
| **Causal AI** | AI reasoning over a dependency topology (graph) rather than isolated signals | 3.4 |
| **Hypermodal AI** | Dynatrace's term for combining predictive, causal, and generative AI | 3.2 |
| **Anomaly detection** | Flagging signals that deviate from a learned baseline | 4.1 |
| **Correlation** | Grouping related alerts into one episode/problem | 4.2 |
| **Episode / problem** | The correlated unit of work (Splunk episode, Dynatrace problem) replacing the alert | 4.2 |
| **Alert fatigue** | Operator desensitization to alert volume — reduced in *volume* but possibly *shifted* in cost | 10.3 |
| **RCA** | Root-cause analysis — inferring why an incident happened | 5 |
| **Topology / dependency graph** | The model of what calls/depends on what; the substrate for causal RCA | 5.1 |
| **Noise reduction** | Lowering non-actionable alerts reaching humans | 4.3 |
| **MTTD / MTTA / MTTR / MTBF** | Mean time to detect / acknowledge / resolve / between failures | 9.1 |
| **Deflection** | Resolving a contact without an agent; honest only if *sustained* | 6.4, 10.4 |
| **Virtual agent** | A conversational front door for service requests | 6.4 |
| **Now Assist** | ServiceNow's generative-AI layer for ITSM/CSM | 6.2 |
| **Service Agent** | Microsoft's declarative customer-service agent inside Microsoft 365 Copilot | 6.3 |
| **Copilot** | AI assistance embedded in a workflow (code, ops, service desk) | 6.3, 7 |
| **Runbook automation** | Deterministic execution of a pre-authored runbook | 7.3 |
| **Runbook generation** | An LLM drafting a runbook/script — a change proposal, not a deployment | 7.3 |
| **Auto-remediation** | AI-triggered automated action on production | 8 |
| **Guarded autonomy** | Acting within a pre-approved action catalogue, blast-radius caps, and a kill switch | 8.2 |
| **Blast radius** | The maximum impact an automated action may have | 8.2–8.3 |
| **Kill switch** | A tested control that halts automation instantly | 11.4 |
| **Self-healing** | Automatic recovery from failure — mature (k8s) to aspirational (novel LLM fixes) | 8.4 |
| **Automation bias** | Human over-trust in machine output, compounding with fluency | 10.6 |
| **Backtest** | Replaying your own history through the AIOps engines to measure real accuracy | 9.2 |
| **Human-in-the-loop (HITL)** | A required human approval before a defined class of actions | 11.1 |
| **Model risk management (MRM)** | Inventory/validation/monitoring of models — applied to AIOps models too | 11.5 |
| **AI action register** | The inventory of automated action classes and their accountable owners | 11.1 |
| **FinOps** | Financial-operations discipline for cloud/AI spend: metering, attribution, caps | 12.2 |
| **AgentOps** | The operations discipline *for* AI agents — the inverse of this guide | 1.3 |

### 15.2 Summary: AI for IT in one page

**The domain.** AI for IT is AI/ML and LLM/agentic technology pointed at IT operations themselves — observe, detect, diagnose, act, learn — across four pillars: AIOps, ITSM AI, workforce copilots and runbook automation, and governance. Its best-known label, **AIOps, was coined by Gartner in 2016** ✅ — an **analyst-created market category** ⚠ that a vendor ecosystem then filled.

**What is genuinely new.** The **ML/LLM layer** (baseline-learning detection, probabilistic diagnosis), the **correlation** (alert → episode), and the **probabilistic diagnosis** (ranked root causes under uncertainty). Everything downstream — dashboards, runbooks, ticketed workflows — is plumbing, and the plumbing is where rollouts fail.

**The capability stack.** Telemetry (Datadog, Dynatrace, Splunk, Elastic, IBM; Prometheus/Grafana OSS), detection (statistical → ML → LLM), diagnosis (topology/causal RCA + LLM explanation, "LLM explains, deterministic AI ranks"), ITSM (Now Assist, Microsoft Copilot/Service Agent, triage, deflection, knowledge), workforce (GitHub Copilot — **55% faster in a controlled experiment** ✅, Claude Code-class agents, ops copilots), and action (guarded autonomy L0–L4).

**The evidence.** Directional gains are credible and widely reported; **magnitudes are almost entirely vendor-published and unfalsified** ⚠. The only trustworthy evaluation is the **backtest on your own incident corpus** — precision/recall, RCA top-1/top-3, sustained (not novelty-period) MTTR.

**The failure modes.** Hallucinated remediation, false correlation and confident wrong RCA, alert fatigue *shifting* rather than disappearing, the deflection measurement trap, data-quality dependency, automation bias, and the AIOps stack's own drift.

**The governance.** Change control over the *rule*, blast-radius caps, an AI action register, a decision-chain audit trail that passes the "explain this incident" test, a drilled kill switch, and MRM applied to the AIOps models — none of them AI-specific inventions, all of them the estate's existing controls extended to a new actor.

**The economics.** Cost lives in telemetry ingest/retention, per-host licences, LLM inference, storage, integration, and people; the business case is only as honest as the backtest that populates it.

**The bottom line.** AI for IT is real, maturing, and worth doing — *as a governed capability with an owner*, funded first on telemetry hygiene, evaluated on your own data, and allowed to act only inside an envelope it cannot escape. The vendors sell a category; the value is in the operated intelligence.

---

*— End of guide.* Authored by Jack Liu Shurui, Solution Architect, September 2026. Verify everything on your own estate; treat every vendor number as a hypothesis, and every autonomous action as a governed change. The intelligence that matters is not the model in the demo — it is the one operating production, inside the guardrails, every day: the operated intelligence.
