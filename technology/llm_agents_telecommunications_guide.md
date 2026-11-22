# LLM Agents in Telecommunications: The Constraint Is Permission

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui - Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Technology / Domain Guide (technology/)
> **Audience:** Solution architects, OSS/BSS platform leads, network operations and change-management leads, and AI governance practitioners putting language-model agents anywhere near a live network
> **Last Updated:** November 2026

*A deep-research guide to what actually blocks LLM agents in telecommunications, and it is not the model. It is the authority to act. The guide defines the three-layer domain an agent must operate across - the network itself, the Operations Support Systems that observe and command it, and the Business Support Systems that own the customer and the money - and shows that each layer carries a different data model, a different change-control regime and a different cost of being wrong. It sets out a permission model in which authority is a tuple of verb, scope, blast radius, reversibility, time window and evidence, and argues that most public "AI agent" claims in this industry collapse to the first two elements of that tuple, read and recommend. It traces what TM Forum's Autonomous Networks ladder actually graded and what it explicitly declined to certify, what 3GPP, ETSI and the O-RAN Alliance specify about observation and almost nothing about actuation authority, and what the data estate does to every one of those arguments. It then tests ten representative use cases against a single question: does the public description support an agent that plans, acts, observes and adapts, or only a model that scores, classifies or predicts? It closes with an authority table by layer, six failure modes, a measurement section explaining why a grep count in this repository is not a dedup, a worked example at a fictional operator where the interesting decision was not which model to use but which verbs to grant, and the six questions a bank buying connectivity should ask a telco about its automation.*

**Cross-references:** [../singapore/starhub_software_systems_guide.md](../singapore/starhub_software_systems_guide.md) (the OSS/BSS estate of one real operator, layer by layer - this guide cross-references it and never re-derives it), [nec_asia_pacific_guide.md](nec_asia_pacific_guide.md) (the vendor side of the same estate: what a network-equipment and systems-integration vendor actually sells), [ai_llm/agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md) (how an agent decides; this guide owns what it is permitted to decide about), [ai_llm/agent_tool_routing_guide.md](ai_llm/agent_tool_routing_guide.md) (tool selection - the mechanism by which a granted verb becomes an executed one), [ai_llm/agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md) (containment; the technical half of the permission boundary described here), [ai_llm/agentops_guide.md](ai_llm/agentops_guide.md) (running agents in production), [ai_llm/agent_versioning_guide.md](ai_llm/agent_versioning_guide.md) (why a change to a prompt is a change to a control surface), [ai_llm/ai_agent_drift_guide.md](ai_llm/ai_agent_drift_guide.md) (behavioural drift against a fixed permission set), [ai_llm/ai_governance_framework_guide.md](ai_llm/ai_governance_framework_guide.md) (the governance scaffolding this domain instantiates), [ai_llm/agents_work_fall_apart_guide.md](ai_llm/agents_work_fall_apart_guide.md) and [ai_llm/autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) (what breaks, and what "autonomous" is usually taken to mean), [ai_llm/building_vertical_ai_agents_guide.md](ai_llm/building_vertical_ai_agents_guide.md) (the vertical-agent pattern applied here), [measurement_lab_guide.md](measurement_lab_guide.md) (the counting discipline section 1.5 applies), [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (impact tolerance and the evidence model - the same argument in a different regulated industry, condensed here and owned there), [../banking/ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) (the risk-register form of the same controls).

**How to read this guide.** Section 1 is the decoder, the thesis, the boundary and the measurement that establishes the gap. Section 2 is the three-layer domain. Section 3 is the industry's own autonomy framework, verified at source with dates, and its relationship to 3GPP, ETSI and O-RAN. Section 4 tests ten use cases on the analytics-versus-agent question. Sections 5 to 8 take the layers in turn: fault and performance, service assurance, the RAN and field operations, and BSS. Section 9 is the data and integration reality. Sections 10 and 11 are the permission model, its authority table, and the regulated analog. Sections 12 to 14 are the failure modes, the worked example and the anti-patterns. Sections 15 and 16 are the audit trail.

**Integrity convention.** Verified tick = read this pass at the publisher's own page or document, with the identifier and date recorded in section 15. Warning sign = partially verified, class-of-behaviour only, version-gated and not re-confirmed, or sourced from a research pass other than this one. Claims carrying no mark are architectural reasoning, not sourced facts, and are written so that a reader can tell the difference. **No market size, market share, deployment count, vendor ranking, or percentage improvement appears anywhere in this guide** - section 15 records that such figures exist upstream and were deliberately excluded.

---

**Contents**

1. [The Overview: The Decoder, the Thesis, and the Boundary](#1-the-overview-the-decoder-the-thesis-and-the-boundary)
2. [The Three-Layer Domain: Network, OSS, BSS](#2-the-three-layer-domain-network-oss-bss)
3. [The Ladder: What TM Forum Graded and What It Did Not](#3-the-ladder-what-tm-forum-graded-and-what-it-did-not)
4. [Analytics or Agent: Settling the Question Use Case by Use Case](#4-analytics-or-agent-settling-the-question-use-case-by-use-case)
5. [Fault and Performance Management: The Anatomy of One Use Case](#5-fault-and-performance-management-the-anatomy-of-one-use-case)
6. [Service Assurance and the Customer Tier](#6-service-assurance-and-the-customer-tier)
7. [The RAN and Field Operations](#7-the-ran-and-field-operations)
8. [BSS and the Customer-Facing Estate](#8-bss-and-the-customer-facing-estate)
9. [The Data and Integration Reality](#9-the-data-and-integration-reality)
10. [The Permission Model: Authority, Blast Radius, Reversibility, and the Table It Produces](#10-the-permission-model-authority-blast-radius-reversibility-and-the-table-it-produces)
11. [The Regulated and Enterprise Analog](#11-the-regulated-and-enterprise-analog)
12. [The Failure Modes](#12-the-failure-modes)
13. [The Worked Example: A Fictional Operator](#13-the-worked-example-a-fictional-operator)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [The Glossary, the Cross-References, and the Closing Summary](#16-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview: The Decoder, the Thesis, and the Boundary

### 1.1 The Thesis

**A telecom agent is not limited by what it knows about the network but by what it is allowed to change on it.**

The sentence is not a complaint about model quality. It is a statement about where the difficulty actually lives, and it survives whatever happens to model capability over the next few years. Three observations support it, and the rest of this guide is the elaboration.

- **The knowledge problem is largely solved at the observation layer, and was solved before language models arrived.** A modern network emits performance counters, alarms, configuration state, topology and streaming telemetry continuously. 3GPP has specified the management data model for this since long before anyone called it AI: performance management, fault management, and the network resource model are mature, standardised and widely implemented. An agent that can read all of that has, in principle, more observation than any human operator ever had. Reading is not the constraint.
- **The actuation problem is not solved, and is not primarily a modelling problem.** Every change to a live network is a change to a shared, customer-visible, partially irreversible state. The question "may this be done at 02:40 without a change record" is answered by a change-management process, a regulator, a vendor support contract and a rollback plan, none of which is a language model concern. The agent's competence is necessary and nowhere near sufficient.
- **The permission boundary is where institutions already have machinery, and the machinery predates agents.** Telecommunications operators run change advisory boards, maintenance windows, method-of-procedure documents, segregation of duties, and - in many jurisdictions - regulator-facing notification duties for network changes. That machinery is the thing an agent must be admitted into. The industry did not have to invent a governance model for agents; it had to decide whether its existing one could admit a non-human actor.

Two corollaries follow and are tested throughout:

1. **An agent that only reads is a productivity tool; an agent that writes is a control.** The distinction is not about the model. It is about whether a state change occurred that a customer could observe and an auditor could ask about.
2. **The most dangerous agent is the one scoped to a layer it does not understand.** A BSS agent that can submit an order has, by proxy, network authority, because the order becomes a provisioning request that becomes a configuration change. Section 8 develops this as the proxy-permission problem.

### 1.2 The Decoder

| Term | What it means here | Why the precision matters |
|---|---|---|
| **Network element (NE)** | A physical or virtual function that forwards, terminates or manages traffic: a base station, a router, a switch, a core function, an optical terminal. | The NE is where state is physically real. Everything above it is a representation, and representations drift. |
| **RAN** | Radio Access Network - the radio edge: base stations and the functions that control them. Written uppercase and anchored, because the lowercase English verb "ran" is a different word entirely (section 1.5). | RAN faults are the canonical autonomous-network scenario in the public literature, and the most cited example of a use case that is genuinely hard to act on safely. |
| **OSS** | Operations Support Systems: the telecom layer that observes and commands the network - fault, performance, configuration, inventory, provisioning, service assurance. **In the rest of this repository OSS means open-source software.** | The collision is the single most common false friend in this domain and it corrupts naive repository searches (section 1.5). |
| **BSS** | Business Support Systems: the commercial layer - customer, product catalogue, order, billing, charging, credit, collections, care. | BSS owns money and identity. Its change-control regime is financial, not operational, and that changes what an agent may do. |
| **Fault management** | The discipline of detecting, correlating, diagnosing, ticketing and clearing network faults, with alarm lifecycle states. | It is the highest-volume candidate for agency and the one where "correlation" is most often mistaken for "action". |
| **Alarm correlation** | Reducing many alarms to fewer, causally-related problem records. A modelling task. | Correlation produces a hypothesis. Acting on it is a different permission. The distinction is the whole of section 4. |
| **Closed loop** | A control cycle that observes, decides and acts without external intervention, then re-observes. ETSI's ZSM architecture names the stages Observe, Orient, Decide, Act. | "Closed loop" is the industry's older, more honest word for what is now marketed as agency. It was always a statement about authority, not about intelligence. |
| **Intent** | A declarative statement of a desired outcome, with the mechanism left to the system. | Intent is the interface that makes permission grantable: you authorise an outcome and a boundary, not a command sequence. |
| **AN Level** | TM Forum's 0-5 maturity score for autonomy in a *specific network scenario*. | The score is per scenario, not per company (section 3). Treating it as a corporate grade is the most common misreading. |
| **MOP** | Method of procedure: the written, reviewed sequence a change follows, including verification and rollback. | A MOP is the artefact an agent would have to execute or replace. Its existence is why "the agent did it differently" is a finding. |
| **Northbound / southbound** | Northbound = toward OSS/BSS and consumers of the network's services; southbound = toward the network elements themselves. | Permission is directional. An agent granted northbound read is not thereby granted southbound write, and conflating them is a design error, not a wording issue. |

### 1.3 The Boundary, Stated Once

This guide owns the *domain*: the three layers, their data models, their change-control regimes, the permission model, and the use-case classification. It does not own the agent mechanics. Decision policy, tool routing, sandboxing, versioning, drift detection and observability are owned by the guides cross-referenced above, and are cited by filename rather than summarised.

It also does not own any single operator's estate. [../singapore/starhub_software_systems_guide.md](../singapore/starhub_software_systems_guide.md) is the reference case for what a real OSS/BSS estate looks like layer by layer - the billing and CRM stack, the provisioning chain, the RAN and core, the fibre estate. Where this guide needs a concrete estate to point at, it points there. It does not re-derive it, and it does not treat one operator's disclosed architecture as the industry's.

What remains is the part that is genuinely domain-specific and genuinely unsolved: **deciding, verb by verb and layer by layer, what a non-human actor may be permitted to do to a live public network, and what evidence makes that permission defensible afterwards.**

### 1.4 The Measurement Trap, and Why It Applies Here

Every guide in this repository that makes a claim about coverage has to survive a counting test, and this domain is the hardest case in the repository for one reason: **its vocabulary collides with the repository's own vocabulary in three separate ways**. A search that produces a number here is almost certainly measuring the wrong thing, and the number will look plausible.

**First false friend: OSS.** In telecommunications, OSS means Operations Support Systems. Everywhere else in this repository - and in most of the software industry - OSS means open-source software. The two meanings are not adjacent; they are unrelated, and they share an acronym with no distinguishing context.

**Second false friend: RAN.** In telecommunications, RAN means Radio Access Network, and it is written uppercase. The string "ran" is also the past tense of "run", which is one of the most common verbs in English prose about software. A case-insensitive word-boundary search for `\bran\b` does not find radio access networks; it finds the verb, thousands of times.

**Third false friend: LTE.** In telecommunications, LTE is a radio technology. As a substring, "LTE" appears inside ALTER, FILTER, RALTER, and a long list of SQL and code identifiers. A substring search for LTE measures database schema more than it measures mobile networks.

### 1.5 The Numbers, and What Each One Actually Measures

The figures below were measured this session with `grep -rliE <pattern> --include='*.md' .` at the repository root. They are reported as measured, because the point of the section is that a measured number can still be the wrong number.

| Pattern | Files | Occurrences | What it actually measures |
|---|---|---|---|
| telecom or telco | 78 | - | Almost entirely incidental: telcos named as bank customers and counterparties, the StarHub estate guide, the NEC vendor guide, an eSIM guide, a defence-electronics vendor. **Not a measure of telecom depth.** |
| 5G | 35 | 180 | Of which only 19 files and 148 occurrences are the standalone token. The remainder are 5GbE (18 occurrences), 5GB (9) and 5GHz (4) - Ethernet, storage and Wi-Fi. |
| LTE | 26 | 92 | Of which only 3 files and 6 occurrences are the standalone token (4 in the eSIM guide, 1 in an RTLS guide, 1 in a VPN guide). **61 occurrences are inside ALTER, 17 inside FILTER or FILTERS**, the rest in RALTER, EOFILTER, ALLTEL and ELTE. |
| `\bRAN\b` (uppercase, anchored) | 5 | 47 | The genuine radio-access sense: the StarHub guide, a Kairos guide, a low-latency C++ guide, the measurement lab guide, the NEC vendor guide. |
| `\bran\b` (case-insensitive) | 188 | 476 | **The English verb.** Roughly ten times the true RAN population, and the count would be reported as "RAN coverage" by anyone who did not check. |
| `OSS` or `BSS` (uppercase, naive) | 83 | - | Dominated by open-source software: 65 files contain `\bOSS\b`, and 63 of those also discuss open source. |
| `\bOSS\b` (uppercase only) | 65 | - | Of which **63 also mention open source**. OSS overwhelmingly means open-source software in this repository. |
| `\bBSS\b` (case-insensitive, anchored) | 5 | - | Four are the telecom sense (the StarHub guide, a Camel-K research note, the measurement lab guide, a database comparison). **One is a false positive: "Bachelor of Social Science".** |
| TM Forum, `\bTMF\b`, or eTOM | 9 | - | The genuine standards vocabulary. |
| autonomous network, self-healing network, or zero-touch | 7 | - | The genuine autonomy vocabulary. |
| intent-based network | **0** | 0 | The concept the permission model depends on, entirely absent from the repository's prose. |
| `\bNOC\b` | 7 | - | Network operations centres, mostly incidental. |
| network fault, fault management, or alarm correlation | 4 | - | The fault-management vocabulary. |
| "alarm correlation" (exact phrase) | **0** | 0 | The phrase, in a repository with four files of fault vocabulary. |
| A line containing both "agent" and telecom, telco or "network operator" | 2 | - | The only two files where the two vocabularies meet: an OpenShift AI alternatives guide and a general LLM agent use-cases guide. |
| A line containing both "agent" and `\bNOC\b` | **0** | 0 | Zero. |

### 1.6 Why a Count Is Not a Dedup

**Lesson 1: the pattern determines the answer more than the corpus does.** The same corpus yields 5 files or 188 files for "RAN" depending on one flag. Neither number is wrong; they answer different questions, and only one of them answers the question a reader thinks is being asked. A coverage claim that does not print its pattern is unfalsifiable.

**Lesson 2: an anchored uppercase search is a domain-aware search.** `\bRAN\b` works not because uppercase is magic but because the domain convention is uppercase. That convention is doing the disambiguation, and it works only for acronyms with a strong case convention. It does not work for OSS, which is uppercase in both senses - which is why OSS needs semantic checking (does the file also discuss licences, forks or distributions?) rather than a better regex.

**Lesson 3: substring matches require a second pass, always.** The LTE figure is the clearest case in the repository: 92 occurrences, of which 6 are the technology. A 26-file claim would be wrong by a factor of roughly nine, and it would be wrong in the direction of overstating coverage, which is the direction that flatters.

**Lesson 4: the interesting number is often zero.** "Alarm correlation" appears zero times. "Intent-based network" appears zero times. A line containing both "agent" and "NOC" appears zero times. Those zeros are the finding: the vocabulary of agent engineering and the vocabulary of telecom operations do not co-occur in this repository, which is precisely the gap this guide addresses. **A search that only reports positives cannot report a gap.**

---

## 2. The Three-Layer Domain: Network, OSS, BSS

### 2.1 Why the Layer Split Is the First Architectural Decision

Every failed agent programme in this domain that I have seen described publicly fails in the same place: it treats "the telecom estate" as one system with one API surface. It is not. It is three systems with three data models, three change-control regimes and three different costs of being wrong, joined by interfaces that were designed for humans moving tickets between departments.

The split matters because **permission is not a property of the agent; it is a property of the object being changed**. The same model, the same tools, the same prompt, can be entirely safe against an object in one layer and unacceptable against an object in another. An architect who has not drawn the layer boundary cannot answer the question "what may this thing do" with anything more precise than "read mostly".

| | **Network** | **OSS** | **BSS** |
|---|---|---|---|
| **What it is** | The forwarding and radio plane, plus the element managers and controllers that command it | The operational systems that observe, inventory, provision and assure the network | The commercial systems that own customer, product, order, bill and balance |
| **Canonical objects** | Cell, sector, carrier, interface, tunnel, route, network function, optical channel | Alarm, problem ticket, performance counter, configuration item, service order, work order, topology link | Customer, subscription, product order, bill, charge, credit, balance, case |
| **Data model** | Hierarchical, strongly typed, standardised object model (3GPP NRM being the reference for mobile); plus high-volume time series and event streams | Relational plus graph for topology; event-driven; frequently federated across many products with no single schema | Transactional, normalised, financially controlled; the system of record for money and identity |
| **Write latency that matters** | Milliseconds to seconds for control-plane actions; minutes for configuration commit | Seconds to minutes for ticket state; hours for provisioning orchestration | Seconds for a payment authorisation; days for a bill cycle |
| **Reversibility of a wrong write** | **Low.** Dropping live sessions, rehoming traffic, changing a radio parameter - the customer experiences it immediately and cannot be un-experienced | **Medium.** A wrong ticket state is correctable; a wrong inventory record propagates into every future change and is expensive to unpick | **High for money** (a credit can be reversed), **low for trust** (a wrongly disconnected service is a complaint and possibly a regulatory event) |
| **Change-control regime** | Operational: MOP, change window, rollback plan, often vendor-supported configuration only | ITIL-style: change advisory board, release train, test evidence | Financial and privacy: segregation of duties, audit trail, retention, data-protection obligations |
| **Who is accountable** | Network operations and the change authority | Service operations and the platform owner | Finance, revenue assurance, and the customer-facing business owner |
| **Cost of one wrong autonomous action** | Customer-visible outage across a geographic area, potentially including emergency services | A truck roll dispatched to nowhere, or a fault silently mis-recorded | A wrong bill run, a wrong credit, or a disclosure - each of which is a reportable event |

Three things follow from the table that are worth stating outright, because they are where most designs go wrong.

**First, reversibility is not uniform, and it is the variable that should drive the permission grant.** A BSS credit is reversible; a network configuration commit is not, in the sense that matters. An architecture that grants both the same autonomy because both are "writes to a system of record" has confused the layers.

**Second, the layers were joined by human workflows, and those workflows are the permission surface.** The order that leaves BSS and arrives in OSS as a provisioning request is a handoff that was designed to be inspected by a person. Section 8 shows that this handoff, not the billing system, is where a BSS agent acquires network authority by proxy.

**Third, the observability of an action degrades as you go down.** A BSS write leaves an audit trail by construction. An OSS ticket state change leaves a record. A network configuration change leaves a record only if the element's own logging, the configuration management system, and the change system agree - and they frequently do not. **An agent acting at the lowest layer produces the least evidence**, which inverts the usual assumption that the most consequential action is the best instrumented.

### 2.2 The Network Layer: State That Is Physical

The network layer's defining property is that its state is not a record of the world; it *is* the world. When an inventory system says a cell is configured with one parameter and the cell is configured with another, the cell is right and the inventory is wrong. This is the opposite of the BSS, where the system of record is authoritative by definition and reality is expected to conform.

Three consequences for agency:

- **Configuration is not data entry.** A commit to a network element changes behaviour for every attached customer within seconds. The element's vendor support contract typically covers only configurations the vendor has validated; a novel configuration produced by a generative system may be technically valid and commercially unsupported. That is a procurement and liability fact, not a technical one, and it belongs in the permission decision.
- **Telemetry is high-volume and lossy in ways the model cannot see.** Performance counters are aggregated on intervals. Alarms have lifecycle states and suppression rules. Streaming telemetry may be sampled. An agent reasoning over this is reasoning over a *view*, and the view was shaped by collection policy that nobody documented for its benefit.
- **The blast radius of a change is a topology question, not a parameter question.** The same parameter change on a cell with three neighbours is a different act from the same change on a cell that is the only coverage for a transport corridor. Blast radius is computable from topology, which means it can be computed by the system rather than estimated by the model - an important design opportunity developed in section 10.4.

### 2.3 The OSS Layer: Representation, and the Drift Between It and Reality

OSS is where the industry's oldest unresolved problem lives: the representation is always somewhat wrong, and every subsequent action inherits the error. Inventory says a link exists; the link was decommissioned last quarter. Topology says a service runs over a path; it was rehomed during an incident. The ticket says a fault is cleared; the customer says otherwise.

This has a specific and underappreciated implication for agents. **An agent that acts on OSS state is acting on a belief about the network, not on the network.** Its competence at reading OSS fluently makes the belief *plausible*, not *true*. An agent that is good at its job will therefore produce confident actions derived from stale inventory, and the confidence will be correlated with how well it reads - which is exactly the failure mode an operator cannot detect by inspection.

### 2.4 The BSS Layer: Money, Identity, and the Proxy Problem

BSS is the layer where the institution's controls are strongest, its auditors most attentive, and its data most regulated. It is also, counter-intuitively, the layer where agents are most likely to be deployed first, because the use cases look low-risk: answer a billing question, draft a care response, summarise a case, propose a retention offer.

The risk profile is genuinely different from the network, and it is worth being precise:

- **Money is reversible; disclosure is not.** A wrong credit can be reversed in the next cycle. A wrongly disclosed account detail cannot.
- **Segregation of duties is the control, and an agent collapses it.** If one actor can both propose a credit and approve it, the control is gone regardless of whether the actor is human. An agent given both verbs has removed a control that the audit function believes exists.
- **The dangerous permission is not the credit. It is the order.** An order submitted into the provisioning chain becomes a network change. The agent never touched the network; the agent's order did. This is the proxy problem, and it is the single most important thing to understand about BSS agency.

### 2.5 What the Layer Split Predicts

If the analysis in this section is right, then agent deployments should cluster where reversibility is high and blast radius is small, and should stall where the opposite holds - regardless of model capability. That is broadly what the public record shows: read-and-recommend assistants in care and in network operations are common and uncontroversial; autonomous execution against the network is rare and heavily gated; and the most ambitious claims are made about scenarios that are scored rather than executed. Section 4 tests that prediction case by case, and section 1.5 explains why you should not try to prove it with a file count.

---

## 3. The Ladder: What TM Forum Graded and What It Did Not

### 3.1 Why This Framework and Not Another

There is no shortage of autonomy maturity models in this industry, and most of them are unverifiable by design. TM Forum's is the one that matters for this argument because it is per-scenario, independently validated on request, and published with an explicit disclaimer about what validation means - and the disclaimer is the most important sentence in it.

### 3.2 The Taxonomy, in TM Forum's Own Words

The mission page describes the framework as "a six-step taxonomy that CSPs can use to measure their AN progress. Each level has a set of characteristics describing the evolutionary stage of the CSP's journey from fully manual to fully autonomous operations."

The resources page states the range plainly: "AN Levels are classified from Level 0 to Level 5, from manual management to fully Autonomous Networks."

Two of the six levels are described in terms worth quoting, because they are the two that the industry actually argues about.

**Level 3, verified wording.** Machines assist humans in decision-making but still rely on human oversight. Read that twice. At L3, the machine's contribution is to the *decision*, and the accountability remains with the human because the human still has to agree. That is precisely rung 3 on the verb ladder in section 10.2 - recommend - dressed in maturity-model language.

**Level 4, verified wording.** "Decision-making based on intent-driven, predictive analysis, and the capability to perform closed-loop management of service-driven and customer experience-driven networks via AI modelling and continuous learning."

This is a materially different claim, and it is worth decomposing because each clause is doing work:

- **"intent-driven"** - the input is an outcome and a boundary, not a command. That is what makes a permission envelope expressible in the first place.
- **"predictive analysis"** - the system acts on a forecast, not only on a threshold. This moves the failure mode from "reacted wrongly" to "predicted wrongly and acted confidently", which is harder to detect.
- **"continuous learning"** - the system's behaviour changes over time. Which is the clause that makes [ai_llm/ai_agent_drift_guide.md](ai_llm/ai_agent_drift_guide.md) a telecom problem and not only an AI-ops problem.

**The current state, verified.** "Today, most operators are between Levels 2 and 3, but some are moving rapidly towards Level 4 in certain domains." This sentence is the single most useful calibration in the public literature, and it is routinely ignored by vendor material that implies L4 is the baseline.

Nothing here asserts the wording of L0, L1, L2 or L5. The governing methodology document is gated behind a login in practice (section 15.3), and inventing plausible definitions for the missing levels is exactly the kind of fabrication this guide's integrity convention exists to prevent. What can be said structurally, and is flagged as inference rather than quotation: L0 is the absence of automation, L1 and L2 are progressively richer assisted or partially automated operation, and L5 is the limit case in which the network manages itself across scenarios. **Those are structural descriptions, not the framework's text.**

### 3.3 The Score Is Per Scenario, Not Per Company

TM Forum's resources page frames AN Levels as measuring "the level of autonomic capability in a given network scenario (for example: fault management on the radio access network, quality optimization on the IP network...)". The evaluation tools are published as "a series of spreadsheet-based questionnaires" per scenario, and the organisation publishes an evaluation tool for each named scenario:

| Tool | Scenario |
|---|---|
| GB1523A | RAN fault management |
| GB1523D | Transport network fault management |
| GB1523E | IP network fault management |
| GB1526A | RAN energy efficiency optimization |
| GB1525A | RAN quality optimization |
| GB1505A | Individual service fault management |
| GB1503A | Service assurance - individual services |
| GB1059A / GB1059C | The evaluation tool series itself, cited as the instrument used for some validations |

**The first consequence: an "AN Level" without a scenario is meaningless.** "We are a Level 4 operator" is not a statement the framework supports. "Our RAN energy efficiency scenario assessed at Level 4" is. The difference is not pedantry: the scenarios have wildly different permission profiles. Energy efficiency optimisation can be bounded and is partially reversible (you can turn the feature off). Individual service fault management involves acting on live customer services. Scoring the same number on both says nothing about whether either is safe to close the loop on.

**The second: the level can be fractional and partial.** The public validation table shows scores such as 3.6, 3.9, 3.95 and entries marked "4.0 (Partial)". TM Forum explains the second of those directly: "Validations labeled 'partial' were undertaken with a subset of the sub scenarios specified in the AN Level Evaluation Tool." So the instrument is a questionnaire scored against sub-scenarios, and a validation can cover some of them.

### 3.4 The Disclaimer That Does the Work

The resources page carries this sentence immediately below the validation table:

> "Note that the above validation does not certify that the network actually performs at the level claimed, which would require more detailed investigation and testing."

This is the best sentence in the entire framework, and it is the thesis of this guide in the industry's own voice. An independently validated Level 4 means that an assessor agreed the *assessment* was carried out correctly against the methodology and that "the evidence provided is appropriate for the AN Level score stated". It does not mean the network behaves that way. The gap between "the evidence supports the score" and "the system performs" is the gap between a governance artefact and an operational fact, and it is exactly the gap that this guide's permission model is designed to make visible.

### 3.5 What the Ladder Measures, and What It Cannot

The framework measures maturity of autonomy in a scenario. It does not measure, and does not claim to measure:

- **Whether the autonomy is safe.** A scenario can score highly on the questionnaire's automation and closed-loop criteria while the rollback is untested. The instrument is about capability, not about the quality of the envelope.
- **Whether the autonomy is permitted.** Permission is an institutional fact, not a maturity fact. A scenario can be technically capable of L4 and institutionally forbidden from operating at L4 - and that combination is common.
- **Whether the loop is run by an agent.** Nothing in the ladder distinguishes a rules-based closed loop from a learned policy from a language model in the decision seat. This is the single biggest gap for anyone reading the ladder as an agent-readiness index. It is not one.

The last point deserves emphasis because it is where the industry's language shifted without the instrument changing. TM Forum's mission page describes the target as "Level 4 and beyond" and "Zero-X" operations - "zero wait, zero touch, zero trouble" - and lists Catalysts including "trusted multi-agent collaboration for autonomous networks" and work on "near real-time, autonomous fault resolution". That is genuine evidence that the industry's vocabulary moved from closed-loop automation to agent language. It is not evidence that the ladder began measuring agency. **The vocabulary moved; the instrument did not.** Section 4 is the test that the instrument cannot perform.

### 3.6 The Documents Behind It

For a reader who will meet the identifiers: the levels methodology is IG1252 (v1.2.0, team-approved 16 June 2023, listed as available to all); the framework document is IG1218F (v2.1.0, 29 September 2026, members only); the evaluation instrument is GB1059, the AN Level Evaluation Tool (v1.1.0, 21 January 2026, members only). **The methodology is public; the framework and the instrument are not** - which is a practical obstacle to independent scrutiny of any AN Level claim.

### 3.7 The Asymmetry

There is a structural asymmetry in the standards that explains most of the difficulty in this domain, and it can be stated in one sentence: **the standards specify the observation surface exhaustively and the actuation authority hardly at all.**

3GPP tells you, in detail and with a normative model, how a cell's attributes are represented, how performance is measured and reported, how faults are raised and cleared, and how analytics may be produced and consumed. It does not tell you who is allowed to change the cell, under what approval, with what rollback, or with what evidence. Those questions are answered by the operator's change-management regime, the vendor's support contract, and in some jurisdictions the regulator.

For an agent builder, this means the integration surface is well-specified and the governance surface is not. The temptation is to build against the specified surface and assume the unspecified one will be handled by process. Section 10's argument is that this is exactly backwards: the unspecified surface is where the design work is.

### 3.8 The 3GPP Management Specifications

The relevant titles, verified this pass at 3GPP's own specification reports:

| Specification | Title | What it contributes to the domain |
|---|---|---|
| TS 28.104 | Management and orchestration; Management Data Analytics (MDA) | Standardises analytics *as a management function* - a producer of analysis consumed by management, with defined inputs and outputs. This is the standards ancestor of every "AI in OSS" architecture. |
| TS 28.541 | Management and orchestration; 5G Network Resource Model (NRM); Stage 2 and stage 3 | The information model of what exists and what is configurable: the schema of the objects an agent would act on. |
| TS 28.533 | Management and orchestration; Architecture framework | Where management functions sit relative to each other and to the network. |
| TS 32.401 | Telecommunication management; Performance Management (PM); Concept and requirements | How performance is measured, collected and reported. The substrate of every threshold and every trend. |
| TS 32.111-1 | Telecommunication management; Fault Management; Part 1: 3G fault management requirements | The alarm lifecycle and fault-management requirements. The substrate of section 5. |
| TS 23.288 | Architecture enhancements for 5G System (5GS) to support network data analytics services | The core-network analytics architecture (NWDAF): analytics as a network function, consumed by other network functions. |

The analytical point is in the second row. **TS 28.541 is the closest thing the standards have to a permission schema**, because it defines the attributes that can be set and the relationships between managed objects. But it defines them as *data*, not as *authority*. It says a parameter exists and has a type; it does not say who may write it. An agent's tool surface in a well-built OSS will be derived from an NRM-shaped model, and that derivation is where permission must be layered on, because the model will happily describe an attribute that no automated actor should ever set.

The other analytical point is in the first and sixth rows, and it is easy to miss. Both MDA and NWDAF treat analytics as a **function that produces analysis for a consumer**. Neither defines analytics as an actor. The standards architecture is, in this respect, an architecture of recommendation - which is rung 3 - and it is 3GPP's, not a vendor's. Anyone claiming that the standards endorse autonomous actuation should be asked for the clause.

**A version warning.** TS 28.104's specification report lists an initial planned Release of 17, and version numbers extending to 20.1.0 for Release 20. This is exactly the kind of detail that decays: quote the title confidently, quote the version only with the date you saw it. Section 15 records this as warning-flagged.

### 3.9 ETSI NFV-MANO: The Three Management Roles

ETSI GS NFV-MAN 001 V1.1.1 (2014-12), "Network Functions Virtualisation (NFV); Management and Orchestration", defines the management and orchestration architectural framework for virtualised network functions, with three roles that recur in every subsequent orchestration design:

- **NFV Orchestrator (NFVO)** - orchestration of network services and of the resources they need, across the infrastructure.
- **VNF Manager (VNFM)** - lifecycle management of virtualised network functions: instantiate, scale, heal, terminate.
- **Virtualised Infrastructure Manager (VIM)** - management of the compute, storage and network resources that host the functions.

The contribution to this argument is conceptual rather than normative: the framework separates "what service should exist" from "which resources realise it" from "which function is running". That separation is what allows a permission envelope to be expressed at the service level while enforcement happens at the resource level - the same split section 10.4 proposes for blast radius.

### 3.10 ETSI ZSM: The Closed-Loop Vocabulary, and Why It Is the Right One

ETSI GS ZSM 002 V1.1.1 (2019-08), "Zero-touch network and Service Management (ZSM); Reference Architecture", is the most directly useful document in the set for this argument, because it names the mechanism rather than the marketing.

**Principle 05, "Closed-loop management automation", verbatim:**

> "Closed-loop management automation is a feedback-driven process. It seeks to reach and preserve a set of objectives without any intervention external to the specific loop."

> "Closed loops (e.g. using the stages Observe, Orient, Decide, Act) allow e.g. self-optimization, improvement of network and resource utilization, and automated service assurance and fulfilment."

That note is the whole architecture in one line, and it is worth stating what it implies. A closed loop has four stages, and **the agent's contribution is to one of them**. An LLM does not observe (sensors do), does not act (actuators do), and does not re-observe (the loop does). It contributes to Orient and Decide. Everything that makes the loop safe lives in the connections between the stages: what the observation contains, what the decision is permitted to request, and what happens when the re-observation contradicts the prediction.

**Principle 10, "Intent-based interfaces", verbatim:**

> "Intent-based interfaces aim to hide complexity, technology- and vendor-specific details from the user by exposing..."
> "Intent-based interfaces express the consumer request(s) in a declarative form."

Declarative intent is not a convenience feature. It is the precondition for a permission envelope, because an envelope is a statement about acceptable outcomes and boundaries, and a command sequence cannot express one. This is why "intent-based network" is the phrase that appears **zero times** in this repository (section 1.5) and yet is the concept the whole permission model depends on: the vocabulary arrived in the standards before it arrived in the industry's writing.

**The management domain structure.** ZSM 002 defines management domains that federate management services, and a special case:

> "The E2E service management domain is a special management domain that provides end-to-end management of customer-facing services, composed from the customer-facing or resource-facing services provided by one or more management domains. The E2E service management domain does not directly manage infrastructure resources."

"The E2E service management domain does not directly manage infrastructure resources" is a permission statement embedded in an architecture document, and it is the pattern worth copying. The layer that owns the customer-facing outcome is explicitly denied direct write access to the resources, and must instead compose services offered by domains that do. **That is a designed separation of the authority to promise from the authority to change**, and it is the structural answer to the proxy problem in section 8.

### 3.11 O-RAN: Described Structurally, Not Quoted

The O-RAN Alliance's specification repository sits behind a membership portal and could not be opened from this host. The public specifications page loaded; the documents themselves did not. What follows is therefore described **structurally, in terms that can be defended without quoting a specification**, and is flagged as not re-verified at O-RAN's own repository this pass.

The architectural idea is a split of RAN control into two loops with different time scales:

- a **near-real-time control loop**, hosted in a controller that sits close to the radio and operates on a fast cadence;
- a **non-real-time control loop**, hosted further up, operating on a slower cadence and typically responsible for policy, analytics and model management;
- a **controller that hosts third-party applications**, which is the mechanism by which functions from outside the equipment vendor can participate in control;
- **standardised interfaces** between the controller and the RAN, and between the two controllers.

For this guide the consequence is time-scale discipline, and it is the most practically useful thing in the whole O-RAN idea. A language model cannot sit inside a loop measured in milliseconds; the inference latency alone disqualifies it, and the determinism requirement disqualifies it twice. It can sit in the non-real-time loop, where the cadence is minutes to hours and where the decision is "what policy should the fast loop pursue" rather than "which handover should happen now". **Most unrealistic agent designs in this domain fail this one test**, and the failure is visible without opening a single specification: they put a generative model in a loop whose period is shorter than the model's own response time.

---

## 4. Analytics or Agent: Settling the Question Use Case by Use Case

### 4.1 The Test

The industry uses "AI agent" to describe two very different things, and the ambiguity is not innocent: it lets a product that scores a number be discussed as though it changes a network. So the test has to be explicit.

An **agent**, for the purposes of this guide, is a system that does four things: it **plans** a course of action toward a goal, **acts** by invoking a tool that changes state outside itself, **observes** the result of the action, and **adapts** its plan based on that observation. Remove any one and you have something else - a model, a pipeline, or a workflow.

A **model**, by contrast, **scores, classifies or predicts**. It produces an output that something else must act on.

The test for any public description, then, is not "does it say agent" but "does the description contain a mechanism by which a state change occurs and is observed". If it does not, the claim is a model claim regardless of the label.

### 4.2 The Ten Use Cases

Ten use cases recur across the public material. Each is classified here on the evidence of what public descriptions actually describe - not on what the label says. The verdict is about the **public description**, never about a product's capability, which this guide cannot assess.

| Use case | What the public description typically shows | Verdict |
|---|---|---|
| **RAN fault management** | Alarm ingestion, correlation to a probable root cause, ticket creation, and in the more ambitious descriptions a proposed remediation. TM Forum publishes a dedicated evaluation tool for this scenario (GB1523A), which is itself evidence that the scenario is being *assessed* rather than merely *executed*. | **Unsettled.** The correlation and diagnosis layers are clearly model work. Whether remediation executes without a human is not settled by public material, and the existence of a per-scenario assessment tool suggests the industry is still grading it rather than shipping it closed. |
| **IP network fault management** | Similar shape: detection, correlation, root-cause hypothesis, and remediation proposals. A dedicated tool exists (GB1523E), and public validations for the scenario are marked partial. | **Unsettled, leaning model.** A "partial" validation means a subset of sub-scenarios was assessed, which is consistent with a capability that exists in pieces. |
| **Transport network fault management** | Detection and correlation over optical and transport domains; heavily dependent on alarm semantics that differ per vendor. Tool GB1523D. | **Unsettled.** Same class as the two above. |
| **RAN energy efficiency optimisation** | The most plausible closed-loop candidate in the set: a bounded objective, measurable outcomes, and a partially reversible action (disable the feature). Tool GB1526A; public validations for this scenario include full and partial entries. | **Agent-shaped where closed loop is claimed, but the permissions are the interesting part.** Energy saving that shuts a carrier down is reversible in configuration and irreversible in customer experience for the duration. |
| **RAN quality optimisation** | Parameter and feature tuning toward a quality objective. Tool GB1525A. | **Model-supported, actuation unsettled.** Optimisation implies a loop; whether the loop writes to the network or recommends to a human is not established publicly. |
| **Service assurance for individual services** | Monitoring, SLA evaluation, and fault attribution per service. Tools GB1503A and GB1505A. | **Model.** Assurance is an observation and judgement function by construction. Where it acts, it acts on tickets, not on the network. |
| **Customer care assistance** | Retrieval over knowledge and account data, drafted responses, summarisation of interaction history. | **Model** in almost every public description: it drafts, a human sends. Where "agent" is claimed, the state change is a note on a case, not a change to service. |
| **Field force dispatch and scheduling** | Work-order creation, technician assignment, travel optimisation. | **Agent-shaped** in mature descriptions - it plans, creates work orders (a real state change in OSS), observes completion, and re-plans. The state change is in the operational system, not the network, which makes it the most defensible genuine agent use case in the list. |
| **Order fallout and provisioning exception handling** | Detecting stuck orders, diagnosing the failing step, and either correcting it or routing it. | **Unsettled, and the most consequential of the unsettled cases.** A correction here reaches the network by proxy (section 8.4). Public descriptions rarely say which verbs are granted. |
| **Billing dispute and credit** | Explaining a charge, proposing an adjustment, drafting a response. | **Model** in public descriptions. The credit itself is a financial control and is almost always human-approved - correctly. |

### 4.3 What the Table Shows

Three patterns, each worth stating because each is a design instruction.

**The scenarios the industry scores most are the ones it acts on least.** Fault management and energy efficiency are the two most-assessed scenarios in TM Forum's public validation table. They are also the two where the public descriptions most consistently stop at diagnosis or at human-gated remediation. That is not hypocrisy; it is the permission model working. The scenarios were chosen *because* they are the ones where the question "may the machine act" is live, and the answer so far is mostly "not without a human on the artefact".

**The genuinely agent-shaped use cases are in OSS, not in the network.** Field dispatch is the clearest case: it plans, it writes work orders, it observes outcomes, it re-plans. Nobody calls it an AI agent breakthrough, because the state it changes is an operational record rather than a radio parameter. **The industry's least glamorous agent is its most real one**, which is the same observation section 2.5 predicted from the reversibility analysis.

**The labels are uncorrelated with the verdicts.** The strongest "agent" claims attach to the network scenarios, where the public descriptions are weakest on actuation. The weakest claims attach to care and billing, where the descriptions are strongest on mechanism. A reader who ranked these products by the confidence of their agent language would get the ordering almost exactly backwards.

### 4.4 How to Settle It in a Room

For an architect facing a claim that has to be settled rather than classified, four questions do it. They are ordered so that a "no" early saves time.

1. **Name the tool call that changes state, and the system that accepts it.** Not the capability, the call. If the answer is a dashboard, it is a model.
2. **Name what the system observes after the call.** If nothing is observed and fed back, it is a pipeline, not a loop.
3. **Name what the system does when the observation contradicts the prediction.** If the answer is "raises an alert", a human is in the loop and it is rung 3 or 4.
4. **Name who approved the envelope and where that approval is recorded.** If nobody can, the answer is that the permission does not exist and the agent is operating on a default.

An organisation that cannot answer question 1 has a model and is calling it an agent. An organisation that cannot answer question 4 has an agent and is calling it a pilot.

---

## 5. Fault and Performance Management: The Anatomy of One Use Case

### 5.1 Why Fault Management Is the Right Case to Open

Fault management is the most-assessed scenario in the public autonomous-network material, the most common subject of "AI in OSS" products, and the place where the analytics-versus-agent confusion is at its densest. It is also the case where the domain's structure is easiest to see, because a fault has a lifecycle with stages, and each stage has a different permission profile.

The discipline is old and standardised. 3GPP TS 32.111-1, "Telecommunication management; Fault Management; Part 1: 3G fault management requirements", is the requirements document for the fault-management function, and it predates the current wave of AI by a long way. Alarms have lifecycle states - raised, updated, acknowledged, cleared - and the management system has a defined relationship to them. Any agent in this space is operating inside a lifecycle that was specified without it in mind, and the specification is the constraint surface.

### 5.2 The Seven Stages

A fault moves through seven stages. The industry's "AI" claims concentrate on stage two and stage three, which are the two stages that change no state at all.

| Stage | What happens | Data involved | What a model can do | What an agent would have to be permitted to do |
|---|---|---|---|---|
| **1. Detect** | An alarm is raised, a threshold is crossed, a KPI departs from baseline, a customer complains | Alarm stream, PM counters, streaming telemetry, care contacts | Score an anomaly; classify severity; suppress noise | Nothing. Detection is instrumentation, not agency. An agent that "detects" is reading a detector. |
| **2. Correlate** | Many alarms are reduced to fewer problem records | Alarm stream plus topology plus timing | Cluster by topology proximity and temporal proximity; learn suppression rules | Nothing yet. Correlation produces a hypothesis. |
| **3. Diagnose** | A probable root cause is proposed | Correlated alarms, configuration state, change history, known-error database | Retrieve similar past incidents; rank candidate causes; generate a narrative | Read the change log, read inventory, read topology. Still no write. |
| **4. Decide** | A remediation is selected, or an escalation | Diagnosis, runbook library, service impact, current change state | Propose a remediation from an enumerated set; estimate impact | Consult the envelope; check window; check blast radius. Still no write - but the first stage where a refusal can occur. |
| **5. Act** | The remediation is executed | MOP, configuration diff, element access | Generate the diff; sequence the steps | **This is the permission boundary.** Rung 4 (stage), 5 (approve), 6 (notify), or 7 (autonomous). |
| **6. Verify** | The effect is observed: did the alarm clear, did the KPI recover, did the customer's service come back | Alarm state, PM, synthetic probes, care contacts | Compare predicted to observed; decide whether to escalate | Read. Note that verification is the stage most often omitted from designs. |
| **7. Close** | The ticket is resolved, the record is written | Ticket, problem record, known-error entry | Draft the resolution note; update the knowledge base | Write to the operational record, which is reversible and therefore a lower-risk grant than stage 5. |

Two observations from the table, both of which matter more than they look.

**The interesting stages are 5, 6 and 7, and the industry's effort is concentrated on 2 and 3.** Correlation and diagnosis are the stages where models are genuinely strong and where the data is abundant, so that is where products are built. But a system that stops at stage 3 has not closed a loop; it has produced a better ticket. The value of better correlation is real - less alarm noise, faster human diagnosis - and it is a productivity claim, not an autonomy claim.

**Stage 6 is the tell.** A design that includes stage 5 but not stage 6 is not a closed loop; it is an open-loop actuator, which is the single most dangerous pattern in this domain (section 14.1). If a proposed architecture cannot say what it observes after acting, it cannot adapt, and by section 4.1's test it is not an agent at all - it is a script with a language model in front of it.

### 5.3 What Correlation Actually Produces, and Why It Is Not an Action

An alarm storm is the canonical fault-management problem: one physical fault produces hundreds of alarms across a topology as the failure propagates, and the operator's job is to find the one that matters.

Correlation reduces the hundreds to a handful of problem records. It does this with a combination of topology proximity (which alarms are on adjacent objects), temporal proximity (which alarms arrived within a window of each other), and learned suppression rules (which alarm types are known to follow which others). The output is a set of hypotheses, each with a confidence and an evidence trail.

None of that is an action. The output is a **belief about the network**, and the belief has a specific and unavoidable property: it is derived from a representation. If the topology graph is wrong, correlation is wrong. If a suppression rule was added during an incident two years ago and never removed, correlation is wrong. If the alarm stream lost messages during a collector restart, correlation is wrong - and it will be wrong in a way that looks *more* confident, because fewer alarms is what a healthy network looks like.

This is the mechanism that connects section 2.3 to this section. **Correlation is a function of representation quality, and representation quality is the OSS's oldest unsolved problem.** An agent that correlates well and acts on its correlation has amplified a representation error into a network change.

### 5.4 The Remediation Catalogue: Why Small and Enumerated Beats General

The single most useful design decision in a fault-management agent is to give it a **catalogue of remediations** rather than a general capability to change the network.

A remediation catalogue is a finite list of named actions, each with: a fault class it applies to, a precondition set, a MOP, a computed blast radius, a verification check, and a tested rollback. Examples of the class (deliberately generic, because specific actions are vendor- and estate-specific): restart a managed function, fail traffic to a protected path, disable a feature on one object, re-home a service off a degraded element, clear a stuck state.

Four properties make this better than a general capability, and each is a permission property rather than a model property:

- **It bounds the blast radius by construction.** The action's worst case was computed when the action was authored, not estimated at 03:00 by a model.
- **It makes verification definable.** Each catalogue entry has a specific check that says whether it worked. A general capability has no such check, because the space of possible actions has no common verification.
- **It makes the rollback real.** A tested rollback exists per entry. "Roll back the change" for an arbitrary change is a hope.
- **It makes the audit tractable.** "The agent executed remediation R-14 on object X at 03:12, verification passed at 03:19" is a record. "The agent changed the network" is not.

The cost is honest and should be stated: a catalogue is a maintenance obligation. Every new fault class, every vendor upgrade, every topology change can invalidate entries. That obligation is the real price of agency in this domain, and it is why the catalogue is usually small.

### 5.5 The Escalation Path Is Part of the Design

An agent in fault management will be wrong, and the design has to say what happens then. There are four exits, and a design that has only some of them is incomplete:

- **Refusal** - the envelope check fails, the action is not taken, the case escalates to a human with the reason recorded. This is the safest exit and should be the most common.
- **Abort** - the action was started and verification failed mid-way; the rollback runs and the case escalates with both the failed action and the rollback in the record.
- **Timeout** - the action was taken and verification did not complete within its window; the case escalates as *unverified*, which is a distinct and more alarming state than *failed*.
- **Silent success that was wrong** - the action was taken, verification passed, and the customer is still affected because the verification check was inadequate. This is the exit that destroys trust, and the only defence is verification checks designed adversarially: what would this check pass on that is not actually fixed?

That last point is worth making sharply. **A verification check is a test, and tests are written by the same people who wrote the action, with the same blind spots.** The discipline that fixes this is the same one that fixes it in software: make the check independent of the mechanism, and prefer an external observation (a synthetic probe, a customer-facing measurement) over an internal one (the alarm cleared).

### 5.6 Performance Management: The Same Anatomy, One Stage Shorter

Performance management runs the same stages with one difference: **there is no fault object to anchor on.** 3GPP TS 32.401 specifies performance measurement, collection and reporting, and it produces counters and files rather than alarms.

- **The symptom is statistical, so the threshold is a policy choice.** "Degraded" is not a state the network reports; it is a judgement encoded in a baseline. A model trained on that baseline learns the policy, not the physics.
- **The objective has no natural stopping point.** A KPI can be improved in ways the KPI does not measure - a drop-rate metric improves if the network becomes less willing to admit marginal users, and the improvement is real in the metric and false in the service.

The permission consequence is the sharpest in the guide. A performance write is a network write with a **delayed and diffuse** effect: it lands on many customers at once, shows up over hours, and cannot be attributed to the change without a control - which makes verification hard and rollback ambiguous. Performance optimisation is therefore the use case most often proposed with a closed-loop story and least suited to one, and its failure mode is section 12.1.

---

## 6. Service Assurance and the Customer Tier

### 6.1 The Difference Between an Element and a Service

Fault management (section 5) is organised around the element. Service assurance is organised around the **promise**: a customer bought a service with an availability, a latency and a support commitment attached, and the operator's obligation is expressed against that promise rather than against any particular piece of equipment.

The distinction is not academic, because the two views disagree routinely and correctly. Every element can be up while the service is degraded, because the service traverses several elements and the degradation lives in a combination. Every element can be alarming while the service is fine, because of redundancy that the element view does not model. An operator that manages elements well can still breach a service commitment, and the customer only ever sees the second view.

For an agent this changes the question in a specific way. In fault management the question is "what is broken"; in assurance it is "is the customer getting what was contracted, and if not, who is accountable for the difference". The second question has a financial answer attached, which is why assurance sits at the boundary between OSS and BSS and belongs to neither.

### 6.2 Where the Assurance Data Comes From

Assurance draws on four kinds of evidence, and they disagree with each other in characteristic ways.

| Source | What it shows | Its characteristic failure |
|---|---|---|
| Element and network counters | Load, errors, latency at the equipment | Accurate about the element, silent about the experience |
| Service-level models | Roll-up of element state into a service view | Only as correct as the service-to-resource mapping behind it (section 9) |
| Synthetic probes | What a scripted client experiences, on a schedule | Measures the path the probe takes, not the path the customer's traffic takes |
| The customer | A complaint, a ticket, a churn signal | Late, sparse, and often the only signal that was right |

The fourth row is the one an agent changes the economics of. Care contacts and complaints are unstructured, arrive in natural language, and were historically too expensive to read systematically - which is exactly the work a language model does well and cheaply. **Assurance is therefore the place where an agent's first real contribution is likely to be reading, not acting**: turning a complaint into a service-level hypothesis, and turning a probe result into a statement about which customers are affected.

### 6.3 The Two Assurance Loops, and Why They Must Not Share an Envelope

An assurance loop starts from a symptom and ends in a confirmation. There are two of them in a real estate, and they have different risk shapes.

- **The service-assurance loop** starts from an SLA breach, a probe failure or a complaint, orients to a probable cause, and decides on a remediation. Its characteristic risk is **acting on a wrong diagnosis**, and its mitigation is not a better diagnosis but a remediation with a computed blast radius and a tested reversal (section 5.4's catalogue discipline).
- **The fulfilment loop** starts from an order and ends in an activated service. Its characteristic risk is **executing a correct plan against a stale inventory**, and its mitigation is a validation gate that re-reads the target state immediately before execution.

The two loops share machinery - topology, service models, tickets, the change system - and they must not share a grant. An agent that has demonstrated it can remediate a known fault class has demonstrated nothing about its competence to execute a customer order, and the habit of granting both because "it is the same agent" is the architecture error section 10's table exists to prevent.

### 6.4 What Changes When the Customer Is an Enterprise

Consumer assurance is statistical: one degraded service is a complaint, and the operator's exposure is a support cost and a churn risk. Enterprise assurance is contractual: the same degradation is a service credit, a breach notice, and possibly a right of termination. Three consequences follow for an agent's permission.

- **The artefact has a counterparty.** An enterprise customer often measures the service itself and holds the operator to its own numbers. An agent that drafts a breach explanation is drafting a document that will be compared against an independent measurement, so the evidence obligation is higher than in the consumer case, not lower.
- **The credit is a financial commitment.** A service credit is a BSS write with a contract behind it. Drafting one is a recommendation; issuing one is a financial control, and the segregation-of-duties rule in section 8.5 applies without modification.
- **The clock is contractual.** An SLA clock starts on notification, and the decision of when a fault became a breach is a judgement that determines money. An agent permitted to reclassify a fault (section 10.9's must-not column) is an agent permitted to move a financial liability, which is why that particular verb is denied at every rung.

### 6.5 Analytics or Agent, in This Section

Assurance is where the analytics-versus-agent boundary is easiest to draw and most often blurred.

- **SLA scoring, threshold breach detection, anomaly detection on probes, and trend forecasting** are models. They produce a number or a flag. This is where most "AI assurance" products live, and it is a legitimate and useful place to live.
- **Complaint triage, service-impact attribution and breach-narrative drafting** are model work with an agentic wrapper: the agent reads, retrieves, and drafts, and a human releases. The state change, if any, is a note on a case.
- **Ticket closure and remediation** are where agency actually appears, and they are the two verbs that reach systems other than the case record. Everything else in assurance is reading.

TM Forum's own per-scenario evaluation tools include assurance scenarios - the public material lists tools for individual-service fault management and for service assurance of individual services - which is evidence that the industry treats assurance as a *graded scenario* rather than a solved one. Whether the graded capability executes or recommends is not settled by the tool identifiers, and this guide does not assert that it does.

---

## 7. The RAN and Field Operations

### 7.1 Time Scale Is the First Filter

Before any question of authority comes a question of physics: **a loop's period bounds what can sit inside it**. A control action that must complete in milliseconds cannot contain a component whose response time is measured in seconds. That single constraint removes most of the radio network from the reach of language models, and it is worth being concrete about the bands rather than hand-waving.

| Band | Period | What lives there | Can a language model be in the decision seat? |
|---|---|---|---|
| **Real-time control** | Sub-second | Radio scheduling, power control, fast handover, forwarding behaviour | No. Determinism and latency both disqualify it. |
| **Near-real-time control** | Seconds | Radio resource decisions, some policy enforcement | No, in practice. The loop tolerates a fast deterministic policy; it does not tolerate a variable-latency generative step. |
| **Operational loop** | Minutes to hours | Fault triage, ticket handling, remediation of a known fault class, provisioning steps | **Yes**, with a bounded verb set. This is where the real designs live. |
| **Planning loop** | Days to weeks | Capacity, configuration baselines, policy authoring, change scheduling | **Yes**, and the permission is cheapest here because the output is an artefact a human reviews. |

### 7.2 Why the RAN Is the Most Sensitive Layer and the Least Agentic in Practice

The radio access network is the layer where an agent's action has the shortest path to a customer's handset, and it is also the layer where agentic execution is least evident in public material. The reasons are structural rather than cultural.

- **The medium is shared and physical.** A parameter change on one cell changes the interference environment of its neighbours, because radio does not respect the boundary drawn on the planning diagram. A change that is correct cell-by-cell can be wrong cluster-by-cluster, and the interaction is not visible in the model that proposed it.
- **The unit of harm is a handset, not a session.** A dropped session is a data loss; a coverage gap is a person unable to call. The reversibility test in section 10.3 puts this in the worst quadrant, and no improvement in diagnosis moves it out.
- **The configuration surface is vendor-mediated.** The element's data model is standardised at the level of the managed object (section 3.8), but the parameters that actually move performance are frequently vendor-specific extensions, and the vendor's support contract covers only validated configurations. An agent writing outside that set produces a state the operator owns and the vendor will not support - a procurement consequence that no technical validation catches.
- **The control plane is being reorganised, and the reorganisation is about who may act.** The O-RAN architecture described structurally in section 3.11 splits radio control into a fast loop close to the radio and a slower loop above it, with a controller that hosts third-party applications. That design is, in effect, a permission architecture: it decides which functions may influence the radio and at what cadence. An agent belongs in the slow loop authoring policy for the fast loop, not in the fast loop itself.

### 7.3 What RAN Automation Actually Is

The functions that already run closed loops on the radio - mobility tuning, load balancing, energy saving, coverage and capacity optimisation - are deterministic or statistical controllers that predate the current wave of AI, and they are engineered the way control systems are engineered: bounded inputs, bounded outputs, and a stability argument. Calling one of them an agent because a model now supplies a parameter is a category error of the kind section 4 tests for.

The honest classification is therefore:

- **Parameter tuning and energy saving** are model-and-policy work inside a controller whose envelope was set by an engineer. The interesting question is not whether the model is good but who set the bounds and how they are re-validated.
- **Anomaly detection on radio KPIs** is a model. It produces a flag.
- **Cell-level diagnosis and remediation proposals** are model work with a human in the path, which is rung 3.
- **Autonomous remediation on the radio** is rare in public material, and where claimed, the scope is a named parameter class inside a vendor's controller rather than a general capability.

### 7.4 Field Operations: The Most Genuine Agent in the Estate

Field operations is the counter-example that makes the argument sharp. Work that looks least like AI - dispatching technicians - is the most defensibly agentic use case in the domain, and the reason is that its state changes are records rather than radio.

An agent that plans a dispatch does all four things the test in section 4.1 requires: it plans an allocation of work to people and times, it acts by creating work orders in OSS (a real state change in a real system of record), it observes completion and the technician's findings, and it re-plans the remainder of the day when the first job overruns. That is a closed loop, and it has been running in workforce-management products for years without anyone calling it agentic.

Its permission profile is correspondingly modest, and the modesty is instructive:

- **The state change is reversible.** A work order can be reassigned; a schedule can be re-planned. The cost of a wrong dispatch is a wasted trip, not a dropped call.
- **The actuator is a person with judgement.** The technician is the sensor the agent does not have: site access, safety, the physical condition of an installation, whether the fault is what the ticket says. An agent that closes a job on telemetry alone has removed its own best instrument.
- **Safety is not in the model's world.** Working at height, electrical safety, lone-working rules and site access permissions are constraints the agent must be given as hard limits, not inferred from a job description. A dispatch optimisation that improves a utilisation metric by ignoring a safety rule is the KPI-versus-service failure in its most literal form.

The lesson generalises beyond field work: **the closer the state change is to a record, the more agency the estate can absorb; the closer it is to a radio or a route, the less.** The industry's attention runs the other way, which is why so much of what is called an autonomous network is a controller with a better model and so little of what is called AI is the dispatch system that actually closes a loop.

---

## 8. BSS and the Customer-Facing Estate

### 8.1 Why BSS Deserves Its Own Section

The naive view is that BSS is the easy layer: it is enterprise software, the data is relational, the use cases are textual, and the controls are the ones every enterprise already has. That view is wrong in one specific and consequential way, and this section is about that way.

BSS is where **permission is cheapest to grant and most expensive to get wrong**, because the same system that holds a customer's bill also holds the order that provisions their service. The agent that can adjust a charge is a financial actor. The agent that can submit an order is, by proxy, a network actor. Those are different grants with different envelopes, and they live in the same product.

### 8.2 The Chain, and Where the Seams Are

A BSS estate is a chain of handoffs, and each handoff is a place where an agent's authority can leak.

| Step | System | State change | Reversible? | What an agent here is really touching |
|---|---|---|---|---|
| **Quote and configure** | Product catalogue, CPQ | A configured offering, not yet an order | Yes | Commercial logic. Low consequence, high volume - the classic rung-3 case. |
| **Order capture** | Order management | An order record | Yes, before submission | Customer intent. Mis-capture is cheap to fix and expensive to discover late. |
| **Order decomposition** | Order management to service order | Service orders per component | Partly | The point at which one commercial intent becomes several technical commitments. |
| **Provisioning handoff** | OSS orchestration | A provisioning request | **No, once executed** | **The proxy boundary.** The order is now a network change. |
| **Billing** | Billing and charging | A charge, a balance | Yes, next cycle | Money, and the reconciliation that goes with it. |
| **Adjustment and credit** | Billing, credit management | A credit, a write-off | Yes for money; no for trust | A financial control with segregation-of-duties requirements. |
| **Care** | CRM, case management | A case, a note, a commitment | Yes | The customer relationship, and every promise made in writing. |
| **Collections** | Collections, dunning | A suspension, a disconnection | **No, in the sense that matters** | Service withdrawal. A wrongly disconnected service is a complaint and possibly a regulatory event. |

Two rows carry most of the risk and get least design attention.

**The provisioning handoff.** An agent that "only" submits orders has network authority. It never sees a network element, never holds a network credential, and never appears in the network change process - and yet the order it submits becomes a configuration change on a live element. If the order is wrong in a way that provisions the wrong service on the wrong access line, the resulting fault will be diagnosed by network operations as a network fault. **The proxy boundary is where accountability becomes ambiguous**, and ambiguity is the failure mode that survives longest.

**Collections and suspension.** A suspension is a service withdrawal initiated by a commercial system. It is reversible in principle (reinstate the service) and effectively irreversible in experience. An agent granted the ability to suspend for non-payment is an agent granted a customer-facing outage with a commercial justification. The permission here is a billing permission and the consequence is a network permission, which is the proxy problem again in a different direction.

### 8.3 The Four BSS Use Cases, Ranked by What They Actually Touch

| Use case | Verb ceiling that is defensible | Why |
|---|---|---|
| **Knowledge and explanation** ("why is my bill this amount") | Read and summarise, optionally recommend | The output is a statement to a customer. Wrongness is embarrassing and correctable. The real risk is disclosure, which is a data-scoping problem. |
| **Case summarisation and drafting** | Read, summarise, draft | The human sends. The moment the agent sends, it is making commitments on the institution's behalf, which is a different grant. |
| **Adjustment proposal** | Read, recommend, stage | Stage means drafting the adjustment for approval. Execution must remain with a human who is not the proposer, or segregation of duties is gone. |
| **Order correction in fallout** | Read, recommend, stage; execute only within an enumerated and tested correction catalogue | This is the one that reaches the network. It belongs to the same discipline as section 5.4: a named catalogue, not a general capability. |

The ordering is not intuitive to product teams, who tend to rank by customer visibility. It is the correct ordering by consequence, and it follows from the reversibility test alone.

### 8.4 The Proxy Permission Problem, Stated Formally

The problem: **authority composes across system boundaries even when credentials do not.**

An agent in the order-management system holds an order-management credential. It submits an order. The order-management system holds an orchestration credential. The orchestration system holds an element-management credential. The element-management system holds a network credential. No single actor in that chain is authorised to change the network autonomously; the composition is.

Three consequences for design:

1. **The envelope must be evaluated at the composite, not at the call.** A policy that checks "may this agent submit an order" is checking the wrong thing. The right question is "what is the maximum network effect of any order this agent could submit", and that requires the policy engine to reason across the chain.
2. **The blast radius computation in section 10.4 must span layers.** A BSS-sourced action's affected-customer count is not derivable from BSS alone; it needs the service mapping that lives in OSS.
3. **The audit trail must be unbroken across the handoff.** If the order record does not carry the identity of the agent that originated it into the provisioning request, the network change cannot be attributed. Most estates lose this link, because the handoff was designed for a human who would be identifiable in a ticket comment.

ETSI ZSM's structural answer, quoted in section 3.10, is the pattern to copy: the domain that owns the customer-facing service "does not directly manage infrastructure resources". It composes services offered by domains that do, and each of those domains retains its own authority. **The separation of the authority to promise from the authority to change is an architectural property, not a procedural one**, and it has to be built rather than documented.

### 8.5 Segregation of Duties Is the Control an Agent Silently Removes

In a financial-control environment, segregation of duties means that no single actor can both initiate and approve a transaction of a given class. It is a control that auditors test by looking at role assignments.

An agent granted both "propose adjustment" and "execute adjustment" has removed the control, regardless of whether the actor is human. The role assignment still looks correct, because the agent holds one role with two permissions and the reviewer sees a service account. **This is a control failure that passes a role review**, which is the worst kind.

The design rule is mechanical: **if a control depends on two actors, an agent may hold at most one of the two roles.** The agent can be the proposer or the executor, never both, and the envelope should enforce that with distinct credentials rather than distinct prompts. Where an estate genuinely wants one agent to do both for efficiency, the honest move is to retire the control explicitly and record that decision with an owner - not to leave it in the audit pack as though it still operates.

---

## 9. The Data and Integration Reality

### 9.1 Four Model Families, None of Which Is the Estate

The section that decides whether an agent is feasible is not the one about models. It is this one. A telecom estate does not have a data model; it has at least four, layered on top of each other, and the agent's competence is bounded by how well they join.

| Family | What it models | Where it comes from | What it does not tell you |
|---|---|---|---|
| **Network resource models** | The managed objects of the network: cells, functions, links, their attributes and containment | 3GPP, principally the 5G Network Resource Model (TS 28.541), plus earlier generations and vendor extensions | Who may change an attribute, or what a change costs commercially |
| **Vendor element models** | The parameter set that actually moves performance on a given vendor's equipment | The equipment vendor's own schema, and its support contract | Anything about another vendor's equipment, or about the service |
| **Operations models** | Faults, performance, inventory, provisioning, service instances | The OSS products, each with its own schema and its own notion of "service" | A stable identity for the customer, or for the service across products |
| **Commercial models** | Customer, subscription, product, order, bill, charge | The BSS suite, frequently several suites bought at different times | The resources that realise the service |

The first row is the one with the strongest standards story, and it is genuinely strong: a standardised, hierarchical, strongly typed object model for the mobile network, with stage 2 and stage 3 definitions, is the reason an OSS can be built against more than one vendor at all. TM Forum's own material offers a process framework (eTOM) and a shared information model (SID) for the commercial and operational side, and the Open Digital Architecture as the current packaging of that work ⚠ - the frameworks are real and widely referenced; their current document identifiers were not re-read at TM Forum for this pass, so they are named here as frameworks rather than cited as documents.

The point of the table is the last column. Each model is authoritative inside its own boundary and silent outside it. An agent that reasons over the network model knows what a cell is and nothing about who depends on it. An agent over the commercial model knows what was promised and nothing about what delivers it. **The join between them is not standardised by anybody**, and it is the join that an action's blast radius depends on.

### 9.2 The Estate Is an Accretion, and the Seams Are Human

Most large operators did not buy an OSS/BSS stack. They bought a billing engine in one decade, a CRM in the next, a fault-management product when the previous one went end-of-life, an inventory system that was meant to replace two others and replaced one, an orchestration layer added during a virtualisation programme, and a service catalogue that was supposed to become the single source of truth and instead became a fifth source. Each purchase was rational. The composite has no common schema, no common identifier space, and no common change process - and the interfaces between the products are, in a remarkable number of estates, batch files and message queues that a human used to watch.

This matters for agents in a way that is easy to underestimate. **An agent's tool surface is the integration surface, not the products' APIs in the abstract.** Where the integration exists, the agent has a verb. Where it does not, the agent has a document and a person. So the practical question "can an agent do this" reduces to "is there a maintained, authenticated, idempotent interface to the system that would have to change", and that question has a depressing answer in most estates for anything older than the last merger.

The vendor classes each bring their own model to this problem, and none of them solves it alone: the network equipment vendors (Nokia, Ericsson, Huawei) own the element and domain-controller models; the OSS and BSS suite vendors (Amdocs, Netcracker) own the operational and commercial models; the enterprise platform vendors (Salesforce) own the customer-interaction model; and the hyperscalers own the runtime and the model-serving layer and know nothing about any of it. **The join is the operator's problem, and no vendor is architecturally accountable for it.** Naming the classes is as far as this guide goes; they are not compared, ranked or sized.

### 9.3 Identity Is the Hard Part, and It Is Usually Manual

The single most consequential data defect in a telecom estate is not a missing field. It is that **the same thing has a different name in every system**: the customer, the subscription, the service, the circuit, the element and the site all have local identifiers, and the mapping between them is maintained by hand, by a small team, in a spreadsheet or a workflow that nobody has documented.

An agent that computes blast radius needs exactly this mapping: given a cell, which services, which customers, which contracts, which protected classes. Given an order, which resources, which elements, which maintenance windows. The computation in section 10.4 is only as good as the mapping, and the mapping is the part of the estate with the least engineering discipline and the most institutional knowledge held in people's heads.

This has a design consequence that is easy to state and hard to accept: **the data-mapping work is the project, and the agent is the demo at the end of it.** In the worked example (section 13), the three months that mattered were spent producing a service mapping that OSS had never maintained, because no previous system had needed it. That work would have been needed for any serious automation programme, with or without a language model, which is the strongest evidence that the model is not the constraint.

### 9.4 Freshness Is a Permission Property

Every observation an agent makes has an age, and the age is not visible in the value. A topology graph that is four hours old looks exactly like a current one. An inventory record for a decommissioned link is indistinguishable from a live one. A performance counter for a rehomed service is a number with no indication that it describes a different path.

Two practices follow, and both are refusals rather than warnings.

- **Every observation carries a freshness assertion**, and the envelope refuses actions whose blast-radius computation depends on data older than a declared threshold. A warning inside a rationale is read by a model; a refusal is enforced by a broker and cannot be argued with.
- **Synchronisation failures are treated as faults.** The quiet inventory sync job that has been failing since a platform upgrade is the single most common precursor to an agent acting correctly on a network that no longer exists (section 12.1). An estate that does not monitor its own representations cannot delegate decisions to anything.

### 9.5 What This Section Decides

The honest summary is uncomfortable for anyone selling an agent into a telecom estate.

- **Feasibility is per domain, not per estate.** An agent is feasible where the mapping exists and the interfaces are maintained - usually one domain, one product family, one service class. An agent scoped to "the estate" is scoped to a data programme that has not been funded.
- **The join is worth more than the model.** A modest model over a correct service mapping will out-perform a strong model over a stale one, because the failure that matters is not a wrong recommendation but a confident action against the wrong object.
- **The data work is reusable and the agent is not.** Service mapping, identifier reconciliation, freshness monitoring and interface hardening survive a change of model, a change of vendor and a change of programme sponsor. They are the only part of an agent programme that appreciates.

---

## 10. The Permission Model: Authority, Blast Radius, Reversibility, and the Table It Produces

### 10.1 Why "Agent" Is the Wrong Unit of Analysis

The industry argues about whether an agent is ready. The question is malformed. Readiness is not a property of an agent; it is a property of a **grant**. The same agent that is unacceptable when granted a configuration commit is unremarkable when granted read access to a performance counter, and both statements are true simultaneously.

So the unit of analysis is not the agent but the tuple:

**Authority = (verb, object scope, blast radius, reversibility, time window, evidence obligation)**

Each element is independently decidable, and each is enforceable by something other than a prompt. That last point is the important one. A permission that exists only as an instruction in a system prompt is not a permission; it is a hope. A permission that exists as a credential scope, a network policy, a change-system gate, or a broker that refuses the call is a permission. Section 10.5 draws that line hard.

### 10.2 The Verb Ladder

Verbs are not binary. There are seven that matter in this domain, and they are ordered by the authority they confer.

| # | Verb | What the agent does | What must exist for this to be safe | Typical enforcement point |
|---|---|---|---|---|
| 1 | **Read** | Retrieves state: counters, alarms, inventory, tickets, bills | Access control and data-protection scoping | Credential scope, row-level policy |
| 2 | **Summarise** | Compresses what it read into a human-facing artefact | Provenance: every claim traceable to a source record | Output citation requirement |
| 3 | **Recommend** | Proposes an action with a rationale, changing nothing | A human who can evaluate it, and a record of the proposal | Ticket comment, advisory channel |
| 4 | **Stage** | Prepares the change - drafts the MOP, pre-fills the change record, builds the config diff - without committing | A diff that a human reviews against the same artefact the change system will execute | Change system draft state |
| 5 | **Execute with approval** | Performs the change after an explicit human authorisation on the specific artefact | Approval bound to the artefact, not to the intent; approval cannot be blanket | Change gate, approval token, four-eyes |
| 6 | **Execute with notification** | Performs the change and reports it afterwards within a bounded scope | A scope small enough that the report is a courtesy rather than a discovery | Policy engine plus audit stream |
| 7 | **Execute autonomously** | Performs the change with no human in the path, within a declared envelope | All of: computed blast radius, a tested rollback, a kill switch, and a stopping rule | Closed-loop controller with hard limits outside the model |

Most public claims of "AI agent" in this industry sit at rungs 1 to 3. A meaningful number sit at 4. Very few defensible deployments sit at 6 or 7, and when they do, the scope is narrow enough that the interesting engineering is in the envelope, not the model.

The ladder is also a diagnostic. Ask a vendor or an internal sponsor which rung they mean. If the answer is "it depends" or "it recommends and the operator decides", you have rung 3 and you should stop worrying about the model and start worrying about whether anyone reads the recommendation. If the answer is rung 5 or above, ask for the artefact the approval binds to. There is no rung 5 without one.

### 10.3 The Two Tests That Decide the Rung

Two questions, asked of the object rather than of the agent, determine the maximum defensible rung.

**The reversibility test.** If this action is wrong, can it be undone without a customer noticing? A BSS credit: yes. A ticket state change: mostly. A configuration commit on a live cell: no - the sessions are already dropped, the emergency call already failed. Non-reversible actions belong at rung 5 or below unless the envelope is tight enough that wrongness is bounded to something the operator would accept without escalation.

**The blast radius test.** If this action is wrong, how many customers, for how long, and is the boundary computable before the action rather than discovered after it? A change whose worst case is one cell in a rural cluster is a different decision from a change whose worst case is a routing domain. Crucially, blast radius is a **computable** property: topology, service mapping and customer counts are all in OSS. An agent need not estimate its own blast radius; the platform can compute it and refuse the call. Section 10.4 makes this the centre of the design.

Cross the two and the grant falls out:

| | **Reversible** | **Irreversible** |
|---|---|---|
| **Small blast radius** | Rung 6 defensible with notification and audit | Rung 5; rung 4 if the rollback is untested |
| **Large blast radius** | Rung 5 with a named approver; rung 4 preferred | Rung 3 only. This quadrant is where "the agent decided" becomes an incident finding |

That last cell is the one the industry keeps trying to occupy with better models. Better models do not change the quadrant. Only a smaller scope, a tested rollback, or a genuine reversal capability does.

### 10.4 Blast Radius Must Be Computed Outside the Model

The most useful architectural move in this domain is to take blast-radius estimation away from the language model and give it to deterministic code.

The reasoning: a language model estimating its own blast radius is a model grading its own homework, using a representation of the network that may be stale, on a question whose wrong answer is the one that hurts. A graph query over topologically-correct inventory, joined to service and customer mapping, answers it with a number and a list, and that answer can be a **precondition on the tool call** rather than a paragraph in the rationale.

The pattern that follows:

1. The agent proposes an action against an object.
2. A deterministic resolver expands the object into affected services and customers using inventory and topology.
3. A policy engine evaluates the expansion against the grant envelope: affected customers, affected services, whether any is a protected class (emergency services, wholesale commitments, regulated circuits), whether the change falls inside an approved window.
4. If the expansion exceeds the envelope, the call is refused by the broker - not by the model - and the refusal is recorded.
5. If it passes, the action executes and the expansion is archived with it.

Step 4 is the point. The refusal must be structurally impossible to talk past. A model that can argue its way around a limit does not have a limit. [ai_llm/agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md) owns the containment mechanics; the domain contribution here is that **the envelope's unit is customers and services, not API calls**, because that is the unit in which the operator's risk appetite is actually expressed.

### 10.5 Permission Is an Enforced Boundary, Not a Prompt Instruction

There is a specific failure that recurs in agent designs across industries, and it is worth naming plainly: the belief that a well-written system prompt constitutes a control.

It does not, for three structural reasons.

- **The instruction and the capability live in different places.** The instruction is text in a context window. The capability is a credential, a network path, or an API that accepts the call. If the capability exists, the instruction is advisory. Prompt injection does not need to defeat the instruction; it only needs to reach a context where the instruction is not repeated.
- **The instruction does not survive composition.** An agent that delegates to a sub-agent, or calls a tool that calls another tool, has no mechanism to propagate its instruction. It does have a mechanism to propagate its credential, because that is what credentials do. [ai_llm/agent_tool_routing_guide.md](ai_llm/agent_tool_routing_guide.md) covers routing; the permission consequence is that authority must be scoped per call, not per conversation.
- **The instruction is unauditable.** "The agent was told not to" is not evidence. "The broker refused 14 calls that exceeded the envelope, and here are the 14" is evidence. Only one of those survives an audit.

The design rule that follows: **every verb above rung 3 must terminate in a system that does not know what a language model is.** A change gate that expects a human approval token does not care who is asking. A configuration broker that validates against a schema and a policy does not care who is asking. That indifference is the property you want, and it is why the permission model is an integration problem rather than a prompting problem.

### 10.6 The Accountable Human Does Not Disappear at Any Rung

Every rung has a named accountable role, and the rung changes what that role is accountable *for*, never whether it exists.

- At rungs 1 to 3, the accountable role is accountable for **acting on or ignoring the output**. That is a real accountability and it is routinely unstaffed, which is why rung-3 deployments quietly become rung-0 deployments that nobody reads.
- At rungs 4 and 5, the role is accountable for **the artefact and the approval**. The approval must bind to the specific diff, not to the intention, or the human is approving a description rather than a change.
- At rungs 6 and 7, the role is accountable for **the envelope and its continuing validity**. This is the hardest accountability in the set, because it is a standing obligation rather than a per-event one: the envelope was correct when granted, and it must be re-validated whenever topology, customer mix, vendor support status or the agent's own behaviour changes.

That last point connects this section to [ai_llm/ai_agent_drift_guide.md](ai_llm/ai_agent_drift_guide.md) and [ai_llm/agent_versioning_guide.md](ai_llm/agent_versioning_guide.md). A permission envelope is granted against an agent whose behaviour was characterised at a point in time. If the model is upgraded, the prompt edited, or the tool surface extended, the envelope is now granted against something that was never assessed. **In this domain a prompt change is a change to a control surface and should travel through change control**, which is a sentence that sounds bureaucratic until the first time an agent's behaviour shifts inside an unchanged envelope.

### 10.7 What the Permission Model Does Not Solve

Two honest limits.

- **It does not make an unsafe action safe; it makes an unsafe action *unauthorised*.** If the envelope itself is wrong - too wide, computed from stale inventory, or granted by someone without the standing to grant it - the model will faithfully operate inside a bad envelope. Section 12.3 treats the stale-representation case as a failure mode with its own guardrail.
- **It does not address the recommendation layer's failure mode, which is inattention.** A rung-3 deployment where nobody reads the output is not governed by this model at all. It is governed by whatever the humans do, and the humans are the bottleneck the deployment was meant to relieve.

### 10.8 Where the Human Sits, and Why "Human in the Loop" Is Not One Thing

The phrase "human in the loop" covers four very different arrangements, and conflating them is why approvals so often fail to be controls.

- **Human before the decision** - the human decides, the model informs. Rung 1 to 3. Safe, slow, and the human is the bottleneck the deployment was meant to relieve.
- **Human on the artefact** - the model decides and prepares, the human approves a specific diff. Rung 5. This is the only arrangement in which approval is a control, and it requires that the human can actually read the artefact.
- **Human on the envelope** - the model decides and acts inside a boundary a human set earlier. Rung 6. The human's control is the boundary, not the event, which means the boundary's continuing validity is the control.
- **Human after the fact** - the model acts, the human reviews a report. Rung 6 at best, rung 7 in practice. This is not a loop with a human in it; it is a loop with an auditor attached, and the auditor's power is retrospective.

The second arrangement is where most telecom programmes should aim and where most fail, for a prosaic reason: **the artefact is often unreadable**. A configuration diff of two hundred attributes is not reviewable by a human at 02:00, which means the approval is a signature on a description rather than on a change. The design response is to constrain the envelope until the diff is small enough to read - which is a permission decision made for a human-factors reason, and a good one.

### 10.9 How to Read It

The table is the guide's operational deliverable, and it is deliberately short. For each layer it states what an agent may do without further approval, what it must not do under any grant this guide would defend, and who is accountable when the boundary is tested. The middle column is the important one: a design that has never written its must-not list has not made a permission decision, it has made a capability decision and hoped.

| Layer | **May** (default grants, subject to envelope) | **Must not** (no grant defended here) | **Accountable** |
|---|---|---|---|
| **Network** | Read counters, alarms, configuration, topology. Summarise a fault. Draft a configuration diff against a named object. Stage a MOP. Execute actions from a named remediation catalogue, within a computed blast radius, inside an approved window, with verification and rollback present. | Author an arbitrary configuration and commit it. Modify more than one object per action without per-object approval. Change any object whose blast radius includes protected services without a named human approver. Act outside a change window. Act while the verification check for the action is absent or failing. Disable or extend its own envelope. | The change authority for the window in which the action occurs; for rung 6 and 7 grants, the owner of the envelope, who carries a standing obligation to re-validate it. |
| **OSS** | Read inventory, topology, tickets, performance history. Correlate alarms. Propose a diagnosis with provenance. Draft ticket updates. Create and update tickets and work orders within an enumerated taxonomy. Update the knowledge base. | Change an inventory record's topology relationships without approval - inventory errors propagate into every future change. Close a ticket whose verification did not pass. Reclassify a fault to a category that changes its SLA clock. Delete or merge problem records. | The platform owner for the OSS product concerned, plus the service owner for any ticket whose closure is a commitment. |
| **BSS** | Read account, product, order and case data within a data-scoping policy. Summarise. Draft a response for human release. Propose an adjustment. Stage an order correction. Explain a charge with citations to source records. | Issue a credit or a write-off without a human approver who is not the proposer. Send a customer communication without human release. Suspend or disconnect a service. Submit an order that reaches provisioning outside a named correction catalogue. Combine proposal and approval in one actor. | The financial controller for money, the customer-operations owner for commitments, and the data-protection owner for any processing of personal data. |
| **Cross-layer** | Read across layers to compute blast radius and to attribute an action to its origin. Refuse an action. Escalate with a recorded reason. | Act on a representation it has not checked for freshness. Treat a permission in one layer as a permission in another. Inherit authority through a chain of systems without the composite being evaluated. | The architect who defined the grant boundary, jointly with each layer's accountable owner for their segment of it. |

### 10.10 Three Rules the Table Encodes

**Rule 1: the must-not column is enforced outside the agent.** Every entry in that column has a mechanism: a credential that does not exist, a policy engine that refuses, a change gate that requires a token, a data scope that excludes the field. If an entry in that column is enforced only by instruction, the table is aspirational and should be labelled as such.

**Rule 2: accountability does not move down the ladder.** A rung-7 grant does not transfer accountability to the agent; it transfers the *event* out of the human path while leaving the *obligation* with the envelope owner. This is why the cross-layer row names the architect: someone has to own the composition, and in most estates nobody does.

**Rule 3: the may column is narrower than it looks.** "Draft a configuration diff" is a read-plus-generate permission on an object. It becomes a write permission the moment the diff is applied, and the grant that applies it is a separate grant. Systems that conflate drafting and applying have collapsed rungs 4 and 5, which is the most common architectural error in this space.

---

## 11. The Regulated and Enterprise Analog

### 11.1 Why the Banking Comparison Is Worth Making, and Where It Breaks

This repository argues agent governance mostly from a banking position: an agent's authority should be bounded by reversibility and by the institution's ability to reconstruct the decision afterwards. That argument transfers to telecommunications intact. The **calibration** does not, and the difference is the reason this section exists.

In a bank, a misdirected payment is usually recoverable. The receiving institution can be asked to return it, the ledger can be corrected, the customer can be made whole, and the whole exchange leaves a trail that makes reconstruction possible. Recovery is expensive, slow, sometimes unsuccessful, and it exists as a process. **Banking is designed around reconciliation**, which is why an agent that acts and is later corrected is a defensible pattern there.

In a network, a misconfigured cell during a busy hour is not recoverable in that sense. The dropped sessions are gone. The emergency call that did not complete is not re-routable. There is no counterparty to ask for the state back, and the record of what happened is a by-product of equipment that was designed to forward traffic rather than to testify. **The correct response to a wrong network action is prevention, not recovery**, which pushes the maximum defensible rung down by one or two steps for equivalent consequence - and which is why the telecom permission model leans on preconditions computed before the call rather than on post-hoc reconciliation.

The regulatory shape differs too. A bank's automation sits inside prudential and conduct supervision, with a well-developed operational-resilience vocabulary of impact tolerances and important business services; those concepts transfer directly and the repository already owns them in [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md). A telecom operator's automation sits inside licence conditions, security obligations, emergency-service duties and, in many markets, coverage and quality commitments attached to spectrum. The regulator is asking a different question - not "can you survive this" but "is the service you are licensed to provide being provided" - and the evidence an operator must produce reflects that.

What transfers without modification is the governance scaffolding: change control with an approved artefact, segregation of duties, an audit trail that names the actor, a refusal log, and a named accountable owner per decision class. [ai_llm/ai_governance_framework_guide.md](ai_llm/ai_governance_framework_guide.md) owns that scaffolding and [../banking/ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) owns the risk-register form of it; this guide's contribution is the layer-specific calibration, not another framework.

### 11.2 The Asymmetry That Matters Most to a Buyer

There is one further difference, and it is the one an enterprise customer feels directly: **the customer of a telecom service is often better instrumented than the customer of a bank's ledger.**

An enterprise connectivity customer measures the service it bought - latency, loss, availability, on its own probes, against its own contract. It does not measure the bank's ledger, and cannot. So an automated change on the telco side is observable by the counterparty in a way that an automated posting inside a bank is not, and the observation is adversarial: it becomes the evidence in a credit claim.

That inverts the usual assumption about which regulated industry has the harder evidence problem. The bank must justify its decisions to a supervisor after the fact; the telco must justify them to a customer, in real time, against measurements the customer took independently.

### 11.3 Six Questions a Bank Should Ask a Telco About Its Automation

Written for the buyer rather than the seller, because this is where the repository's banking audience actually meets this subject. Each question is answerable by any operator that has made the permission decision, and the inability to answer is itself the finding.

1. **Which changes to my service can be made with no human approval?** Not "do you use AI" - which verbs, on which objects, in the path that carries my traffic. A named remediation catalogue is a good answer; "closed-loop automation" is not an answer.
2. **For an automated change that affected my circuit, what record will you show me?** The actor, the artefact, the approval (or the grant that stood in for it), the blast radius as computed at the time, and the verification result. If the answer is a ticket comment, the trail does not exist.
3. **How do you compute the blast radius of a change, and how fresh is the inventory it uses?** This question separates operators who have a service mapping from operators who have a promise. Ask for the freshness threshold and the refusal behaviour when it is exceeded.
4. **Who signs for an emergency-path change during an incident, and how is the invocation recorded?** Every estate has an emergency path. The question is whether it is a documented authority with a record, or a person deciding at 03:00 that the rules do not apply.
5. **Can you show a rollback exercise for an automated action, with the verification that triggered it?** Not a rollback plan. An exercise, dated, with the observed result.
6. **What does your autonomy claim actually cover?** Which scenario, which evaluation tool, which sub-scenarios, and does it include the service class I am buying. A level without a scenario is a marketing number (section 3.3), and a level scored on a scenario that is not mine says nothing about my service.

---

## 12. The Failure Modes

Six modes, in the order they tend to appear in a programme's life. Each is stated as a symptom an operator would actually observe, the cause, and a guardrail that is a design decision rather than a process aspiration.

### 12.1 (a) The KPI That Improved While the Service Degraded

**Symptom.** The optimisation loop reports success for a quarter: the metric it was told to improve has improved, the dashboard is green, and the complaints have not fallen. In one bad case the complaints have risen in exactly the area the metric covers.

**Cause.** The objective was a proxy, and the agent optimised the proxy. Every KPI can be improved in ways the KPI does not measure - a drop-rate metric improves if the network becomes less willing to admit marginal users, a utilisation metric improves if traffic is pushed to a worse path, an energy metric improves if capacity is removed from where it was about to be needed. The agent is not gaming the metric; it is doing exactly what it was asked, which is why this failure is invisible to the team that set the objective.

**Guardrail.** Every optimisation grant carries a **guardrail metric the agent is not optimising**, evaluated by something other than the optimiser, and a stopping rule that halts the loop when the guardrail degrades. The pair (objective, guardrail) is the permission, not the objective alone. This is also the argument for keeping optimisation in the planning loop: a human reviewing a proposed parameter set against two metrics sees the trade-off that a closed loop is structurally blind to.

### 12.2 (b) The Correlation That Hid the True Fault in the Storm

**Symptom.** Alarm volume falls, the operations dashboard is calmer, and three months later a major fault is found to have been suppressed by a correlation rule that a model learned. The rule's rationale was never reviewable because it was a weight.

**Cause.** Correlation produces a hypothesis, and suppression is a **write to the observation surface** - authority over what the operator can see, which is more consequential than authority over what the operator can change and is almost never recognised as a grant. In a storm the failure compounds: the true root cause is one of the alarms that was collapsed into a parent, and the parent's narrative is plausible enough that nobody reopens it.

**Guardrail.** Treat every suppression or correlation rule the agent proposes as configuration with review, not as model behaviour: it must be expressible as a rule a human can read, with a scope, an expiry and an owner. Sample the suppressed population periodically - a random audit of what the correlation discarded is the only test that finds the fault that was hidden. **An unexplainable suppression is an outage that has not happened yet.**

### 12.3 (c) The Change to an Element That No Longer Exists

**Symptom.** The agent's diagnosis is defensible, its remediation is correct for the topology it read, and the change makes things worse - or fails against an object that was decommissioned two quarters earlier. The post-incident review finds the inventory sync job had been failing quietly since a platform upgrade.

**Cause.** The representation was treated as the network (section 2.3's drift), amplified by an agent that reads fluently and therefore confidently. The wrong-element action is the specific case: the agent selected the object the inventory named, not the object that exists.

**Guardrail.** Every observation carries a freshness assertion, and the envelope refuses actions whose blast-radius computation depends on data older than a declared threshold. Make staleness a **refusal condition**, not a warning: a warning inside a rationale is read by a model, a refusal is enforced by a broker and cannot be argued with. Then monitor the synchronisation jobs as production services, because a stale representation is a fault in its own right.

### 12.4 (d) The Rollback That Was Slower Than the Change

**Symptom.** The change was applied in seconds and the rollback took forty minutes, during which customers were affected. The change record shows both, correctly, and the incident review concludes that the change was "reversible".

**Cause.** Reversibility was assessed as a property of the configuration rather than of the customer experience. A parameter can be restored exactly and the sessions it dropped cannot. The gap between the change's speed and the reversal's speed is the window in which harm accrues, and it is the number the reversibility test in section 10.3 should actually be using.

**Guardrail.** For every catalogue entry, record the **measured** time to reverse, not the intended one, and put it in the grant. If reversal is slower than the harm it is meant to bound, the action belongs at a lower rung regardless of how cleanly it reverts on paper. Prefer actions whose worst case is bounded by a hard timer held outside the agent (the worked example's energy case, section 13.3, is the pattern).

### 12.5 (e) The Use Case That Was a Dashboard With a Prompt Attached

**Symptom.** Eighteen months into an "autonomous operations" programme, the audit finds that no state change ever occurred without a human, the model never observed an outcome, and the "agent" is a retrieval layer over an analytics product. Nothing is broken - except that the permission machinery was never built, and the next release adds a button that writes.

**Cause.** The label arrived before the architecture. This is the failure that section 4's test is designed to catch early, and it is not a technical failure at all: it is a programme that mistook a demonstration of language for a demonstration of agency.

**Guardrail.** Settle the classification before funding: name the tool call that changes state, the system that accepts it, and the observation that follows. A dashboard with a prompt is a legitimate product; it is simply not evidence that the estate can absorb an actuator, and treating it as such defers the permission work to the moment it is most expensive.

### 12.6 (f) The Autonomy Claim With No Permission Trail

**Symptom.** A successful pilot, no incident, and an audit committee that asks what the agent is permitted to do and receives three different answers from three teams. The programme is paused pending a governance review that has no artefact to review. In the worse variant, an automated change is traced afterwards and nobody can produce the grant under which it was made.

**Cause.** No envelope document, no must-not list, no refusal log, no rollback evidence, no attribution through the proxy handoff. The pilot produced confidence rather than evidence, and confidence does not survive a change of sponsor - or an incident.

**Guardrail.** The artefacts are cheap and should exist before the pilot: a one-page grant (verb, scope, blast radius, reversibility, window, evidence), a refusal log the agent wrote, one recorded exercise in which the agent was wrong and the rollback ran, and an unbroken identity trail from the originating actor to the element that changed. **Four artefacts, produced in the first month, are worth more than eighteen months of a clean shadow mode** - and they are the difference between a control and a demonstration.

---

## 13. The Worked Example: A Fictional Operator

**Meridian Telecom is fictional, and so is every entity, number and event in this section.** It contains no data from any real operator, and no real operator, vendor or institution is named in it. Numbers appear only to make the reasoning concrete; none is a benchmark, a market figure, or an improvement claim.

### 13.1 The Estate

Meridian operates a mobile and fixed estate in one country: a radio access network of several thousand cells across two technology generations, a transport and core network, an OSS estate of five products that were integrated over a decade, and a BSS suite with a separate billing engine and a CRM. It runs a network operations centre with three shifts and a change process with a weekly change advisory board and a nightly low-risk window.

Two agent proposals reached the architecture review in the same quarter. They are worth comparing because they arrived with the same label and deserved opposite answers.

### 13.2 Proposal One: The Fault Triage Agent

**The proposal.** An agent reads alarms, performance counters, topology, the change log and the ticket history. It correlates, diagnoses, and drafts a remediation for the twenty most common fault classes. It executes the remediation when the fault class matches a catalogue entry and the blast radius is under a declared threshold.

**What the review found.** The proposal was, in substance, an agent: it planned, it acted, it observed, it adapted. The team had thought carefully about the catalogue and had written twenty entries with preconditions, verification checks and rollbacks. They had not thought about three things.

- **The envelope was expressed in the wrong unit.** Their threshold was "fewer than N alarms affected", which is a measure of the fault's noise, not of its customer impact. A single alarm on a transport link can affect more customers than fifty on a rural cluster.
- **The verification checks were internal.** Every check was "the alarm cleared". None was a customer-facing or service-facing observation, which meant a remediation that silenced the alarm without fixing the service would pass.
- **The change window was assumed.** The design executed whenever the fault occurred, including during the daytime peak, because the team had reasoned that a fault is an emergency and emergencies are exempt. The change authority had not agreed.

**What was granted.** Rung 5 for the twelve catalogue entries whose blast radius could be computed from service mapping and whose rollback was tested; rung 3 for the remaining eight, which were staged as drafts for a human. Blast radius re-expressed as affected services and affected customers, computed by a graph query over inventory rather than estimated by the agent. Verification changed to an external observation for every entry. Execution restricted to the nightly window unless a duty manager invoked an emergency path that recorded the invocation.

**What it cost.** Three months, most of it spent not on the model but on the service mapping that made the blast radius computable - a mapping that OSS had never maintained because no previous system had needed it. That work is reusable and was the most valuable output of the programme.

### 13.3 Proposal Two: The Energy Agent

**The proposal.** An agent watches traffic load per cell and shuts down carriers during low-traffic hours to save energy, restoring them ahead of predicted demand. It was presented as the same kind of thing as proposal one, with a stronger business case.

**Why it was refused autonomous execution.** Three reasons, and the review recorded them because the refusal was the decision that mattered.

- **The action is reversible in configuration and irreversible in experience.** A carrier shut down and restored leaves no trace in the configuration; it leaves a trace in the sessions that were dropped and in the coverage that was absent for however long. The reversibility test put it in the wrong quadrant.
- **The blast radius includes protected services in a way that is hard to compute before the fact.** A cell covering a transport corridor may carry emergency traffic at 04:00. The topology knows the cell; it does not know what is happening on the road.
- **The prediction is the action.** The agent's competence is its forecast of demand. A forecast that is wrong by twenty minutes produces a coverage gap, and the failure is invisible until someone reports it - which is exactly the shape of a failure mode that a clean pilot will not detect.

**What was granted instead.** Rung 3 with a strong staging story: the agent produces a nightly plan of proposed carrier shutdowns with the predicted restore time, the affected service list and the emergency-service flags. A duty engineer approves the plan as a whole. Shutdowns execute with a hard restore timer that does not depend on the agent, and any cell carrying a flagged service is excluded by the platform rather than by the agent's judgement.

**What it cost.** The business case weakened, because the engineer's approval is a per-night cost. The team's honest estimate was that the saving survived the approval cost but not by much. That is a normal outcome and it is worth recording: **the permission model sometimes makes a use case uneconomic, and discovering that in review is cheaper than discovering it in an incident.**

### 13.4 The Comparison, and What It Demonstrates

| | Fault triage | Energy |
|---|---|---|
| Verb reached | Rung 5 (twelve entries), rung 3 (eight) | Rung 3, with platform-enforced exclusions |
| Why the difference | Reversibility, and a blast radius computable from existing mapping | Irreversibility in experience, and a blast radius that depends on facts outside the topology |
| The work that mattered | Service mapping, external verification | Exclusions enforced outside the agent, independent restore timer |
| What the model contributed | Orientation and diagnosis | Forecasting |
| What decided the outcome | The envelope | The envelope |

Both proposals were technically sound. Both teams had competent models, careful prompts, and a real understanding of their domain. **The difference between them was not the quality of the agent; it was the answer to two questions asked of the object rather than of the agent** - can this be undone, and can the boundary be computed before the act.

That is the thesis in operational form. Neither team was limited by what their model knew about the network. Both were limited, in opposite directions, by what they were allowed to change on it - and the programme that succeeded was the one that treated the permission as the design problem and the model as the easy part.

---

## 14. The Anti-Patterns

Six patterns recur. Each is stated as the thing an architect will actually see proposed, followed by what is wrong with it.

### 14.1 The Open-Loop Actuator

**The pattern.** A model decides, an action executes, and nothing observes the result. The design has stages 1 to 5 of section 5.2 and none of 6.

**Why it is the most dangerous pattern here.** It cannot detect its own error, so every error persists until a human notices - and the human notices via a customer. It also cannot be safely widened or narrowed, because there is no feedback signal to calibrate against.

**The guardrail.** No remediation ships without its verification check, and the check must be external to the mechanism that performed the action. This is a release criterion, not a maturity goal.

### 14.2 The Envelope in the Prompt

**The pattern.** The agent is told, at length and in bold, what it must not do. The credential it holds can do all of it.

**Why it fails.** Section 10.5. The instruction and the capability are in different places, the instruction does not survive composition, and the instruction is unauditable.

**The guardrail.** For every line in a system prompt that says "never", there must be a corresponding line in a policy file or a credential scope. A prompt line without one is documentation of an intention, not a control.

### 14.3 The Layer-Blind Grant

**The pattern.** One agent, one credential, read access "across the estate" for convenience.

**Why it fails.** It collapses section 2's three layers into one authorisation surface, which means a BSS-shaped question can produce a network-shaped answer. It also makes blast radius incomputable, because the credential does not know which layer it is being used against.

**The guardrail.** Credentials are per layer and per verb. Convenience is bought with a broker that holds several narrow credentials and decides which to use, not with one wide credential that the model selects from.

### 14.4 The Approval That Binds to an Intention

**The pattern.** The agent says "I will restart the degraded function", a human clicks approve, and the agent then executes a diff the human never saw.

**Why it fails.** The approval was of a description. The change was something else. In an incident review this is indistinguishable from no approval at all.

**The guardrail.** Approval binds to the artefact: the diff, the MOP, the order payload. If the artefact changes after approval, the approval is void and must be re-obtained. This is a change-system property, not an agent property, and most change systems can already do it.

### 14.5 The Score as a Permission

**The pattern.** An AN Level, a model confidence, or an accuracy figure is used to justify a grant. "It is Level 4, so it can run closed loop."

**Why it fails.** Section 3.4 in the industry's own words: the validation "does not certify that the network actually performs at the level claimed". A score describes assessed capability. A grant describes permitted action. They are different objects and neither implies the other.

**The guardrail.** Permission decisions cite the envelope, the rollback, the verification and the accountable owner. They may cite a score as context. They may not cite it as the reason.

### 14.6 The Pilot That Never Ends

**The pattern.** The agent runs in shadow mode for eighteen months. Nobody reads the shadow output after month two. The programme reports success because nothing broke.

**Why it fails.** Shadow mode measures what the agent *would* have done, not what it *can* do, and the measurement decays as attention moves. It also never produces the evidence a grant requires, because nothing was ever executed, verified and rolled back.

**The guardrail.** Shadow mode has an exit criterion defined before it starts: a named set of cases, a review process with a named reviewer, and a decision date. An unbounded shadow mode is a decision to not decide, and it consumes the credibility the next attempt will need.

---

## 15. The Claims Audit

**Legend.** Verified tick = read this pass at the publisher's own page or document, with the identifier and date recorded. Warning sign = partially verified, class-of-behaviour only, version-gated and not re-confirmed, or sourced from a research pass other than this one. Cross mark = rejected, with the reason. Unmarked claims are architectural reasoning.

**Source-quality note.** `www.tmforum.org` was read directly (the mission page and the AN resources page, with redirect-following; `/resources/` paths return 403 and were not used). `www.3gpp.org` DynaReport specification pages were read directly. `www.etsi.org` deliver PDFs were downloaded and converted to text. **`www.o-ran.org` specification downloads are gated behind a membership portal** - the public specifications page loaded, the documents did not. No vendor product documentation was relied on for any architectural claim in this guide, by design: section 9 names vendor classes and asserts nothing about their internals.

### 15.1 Verified at the Publisher's Own Page or Document

| # | Claim | Verdict | Source | Date |
|---|---|---|---|---|
| **(a)** | **The taxonomy:** "TM Forum's Autonomous Networks Project has developed a six-step taxonomy that CSPs can use to measure their AN progress. Each level has a set of characteristics describing the evolutionary stage of the CSP's journey from fully manual to fully autonomous operations." | ✅ | tmforum.org, /missions/autonomous-networks | Read 22 Nov 2026 |
| **(a)** | **The range:** "AN Levels are classified from Level 0 to Level 5, from manual management to fully Autonomous Networks." | ✅ | tmforum.org, /topics/an-resources/ | Read 22 Nov 2026 |
| **(a)** | **Level 3, verified wording:** at L3, "machines assist humans in decision-making but still rely on human oversight". | ✅ | Same mission page | Read 22 Nov 2026 |
| **(a)** | **Level 4, verified wording:** "decision-making based on intent-driven, predictive analysis, and the capability to perform closed-loop management of service-driven and customer experience-driven networks via AI modelling and continuous learning". | ✅ | Same mission page | Read 22 Nov 2026 |
| **(a)** | **Current state of the industry:** "Today, most operators are between Levels 2 and 3, but some are moving rapidly towards Level 4 in certain domains." | ✅ | Same mission page | Read 22 Nov 2026 |
| **(a)** | **The disclaimer:** "Note that the above validation does not certify that the network actually performs at the level claimed, which would require more detailed investigation and testing." | ✅ | tmforum.org, /topics/an-resources/, immediately below the validation table | Read 22 Nov 2026 |
| **(a)** | **Per-scenario framing:** AN Levels "measure the level of autonomic capability in a given network scenario (for example: fault management on the radio access network, quality optimization on the IP network...)". | ✅ | Same page | Read 22 Nov 2026 |
| **(a)** | **ANLAV:** TM Forum "provides a service to independently validate that an operator's Autonomous Network Level assessment has been carried out correctly according to the methodology detailed in IG1252 ... and that the evidence provided is appropriate for the AN Level score stated." | ✅ | Same page | Read 22 Nov 2026 |
| **(a)** | **Fractional and partial scores exist in the public table:** entries at 3.6, 3.9, 3.95, and 4.0 marked "(Partial)". | ✅ | Same page, validation table | Read 22 Nov 2026 |
| **(a)** | **Document metadata:** IG1252 v1.2.0, team-approved 16 June 2023, available to all; IG1218F v2.1.0, 29 September 2026, members only; GB1059 (ANLET) v1.1.0, 21 January 2026, members only. | ✅ | Same page, AN Framework Resources and AN KEI Resources tables | Read 22 Nov 2026 |
| **(a)** | **Per-scenario tool identifiers present in the public validation table:** GB1523A, GB1523D, GB1523E, GB1525A, GB1526A, GB1505A, GB1503A, GB1059A, GB1059C. | ✅ | Same page, validation table | Read 22 Nov 2026 |
| **(a)** | **Direction of travel:** the mission sets the target as AN maturity "to Level 4 and above"; frames operations as autonomous "Zero-X" (zero wait, zero touch, zero trouble); and lists Catalysts including "using trusted multi-agent collaboration for autonomous networks" and "near real-time, autonomous fault resolution". | ✅ | Same mission page | Read 22 Nov 2026 |
| **(b)** | **3GPP TS 28.104** "Management and orchestration; Management Data Analytics (MDA)", status under change control, initial planned Release 17. | ✅ | 3gpp.org DynaReport/28104.htm | Read 22 Nov 2026 |
| **(b)** | **3GPP TS 28.541** "Management and orchestration; 5G Network Resource Model (NRM); Stage 2 and stage 3". | ✅ | DynaReport/28541.htm | Read 22 Nov 2026 |
| **(b)** | **3GPP TS 28.533** "Management and orchestration; Architecture framework". | ✅ | DynaReport/28533.htm | Read 22 Nov 2026 |
| **(b)** | **3GPP TS 32.401** "Telecommunication management; Performance Management (PM); Concept and requirements". | ✅ | DynaReport/32401.htm | Read 22 Nov 2026 |
| **(b)** | **3GPP TS 32.111-1** "Telecommunication management; Fault Management; Part 1: 3G fault management requirements". | ✅ | DynaReport/32111-1.htm | Read 22 Nov 2026 |
| **(b)** | **3GPP TS 23.288** "Architecture enhancements for 5G System (5GS) to support network data analytics services". | ✅ | DynaReport/23288.htm | Read 22 Nov 2026 |
| **(c)** | **ETSI GS NFV-MAN 001 V1.1.1 (2014-12)** exists as titled and defines the NFV Orchestrator, VNF Manager and Virtualised Infrastructure Manager as distinct management roles. | ✅ | etsi.org deliver, gs_NFV-MAN001v010101p.pdf | Read 22 Nov 2026 |
| **(c)** | **ETSI GS ZSM 002 V1.1.1 (2019-08)**, Principle 05 "Closed-loop management automation": "a feedback-driven process. It seeks to reach and preserve a set of objectives without any intervention external to the specific loop", with the note that closed loops use "the stages Observe, Orient, Decide, Act". | ✅ | etsi.org deliver, gs_ZSM002v010101p.pdf, clause 4.2.5 | Read 22 Nov 2026 |
| **(c)** | **ZSM Principle 10, "Intent-based interfaces"**: such interfaces "express the consumer request(s) in a declarative form". | ✅ | Same, clause 4.2.10 | Read 22 Nov 2026 |
| **(c)** | **ZSM E2E service management domain**: "a special management domain that provides end-to-end management of customer-facing services ... The E2E service management domain does not directly manage infrastructure resources." | ✅ | Same, clause 6.3.1 | Read 22 Nov 2026 |

### 15.2 Class-of-Behaviour Only, or Not Re-Confirmed This Pass

| # | Claim | Verdict | Why |
|---|---|---|---|
| **(d)** | **Tool-to-scenario pairings beyond GB1526A.** The guide's table maps GB1523A to RAN fault management, GB1523D to transport, GB1523E to IP, GB1525A to RAN quality, GB1505A to individual service fault management, GB1503A to service assurance. The identifiers and the scenarios are both in the published table; **not every pairing was legible in the text extraction this pass**, and the GB1526A to RAN energy efficiency pairing is the one directly confirmed. | ⚠ | Table extraction is column-flattened; pairings should be re-read against the rendered table before being placed in a procurement document. |
| **(e)** | **IG1401A / IG1401B journey guides, v6.0.0, 31 October 2024, available to all.** | ⚠ | Reported in the source research pass; not re-located in the AN resources page as fetched this pass. The guide states the identifiers only, and asserts nothing about their content. |
| **(f)** | **TS 28.104 version currency** - the spec report lists versions to 20.1.0 for Release 20. | ⚠ | Verified as of the read date only. A specification's version list is the most perishable fact in this guide; quote it with a date or not at all. |
| **(g)** | **O-RAN architecture described structurally** - near-real-time and non-real-time control loops, a controller hosting third-party applications, standardised interfaces. | ⚠ | O-RAN specification downloads are membership-gated. The description is at the level defensible without quoting a specification. **No interface name and no document number is asserted anywhere in this guide.** |
| **(h)** | **The industry's shift from closed-loop language to agent language.** | ⚠ as a general claim, ✅ as a specific one | The specific evidence is verified (the Catalyst titles and the Zero-X framing on the mission page). The general claim that the industry as a whole made this shift is an interpretation and is marked as such in section 3.5. |

### 15.3 What Could Not Be Verified

Listed explicitly, because an unlisted gap becomes an assumed fact.

1. **The per-level criteria for L0, L1, L2 and L5.** IG1252 is listed as "Available to all", but the download path is gated behind a login in practice, and the framework document IG1218F is members-only. **Section 3.2 describes L0, L1, L2 and L5 only in structural terms and flags them as inference, not quotation.** Do not cite this guide for the framework's definitions of those levels. Only L3 and L4 are quoted, and only from the public mission page.
2. **The exact wording of the six level names.** The framework refers to levels by number in the public material; a canonical set of descriptive names was not read and is not asserted.
3. **The framework's original publication year.** Deliberately not asserted. The dates verified here are the document revision dates (2023, 2024, 2026), which are not the framework's origin.
4. **Any operator's AN Level.** The public validation table names operators alongside scores. **This guide prints none of them**, because any such pairing in a document like this becomes a ranking, and a ranking is precisely what the disclaimer in section 3.4 says the score cannot support. The table's existence and its shape (scenario, level, tool version, date) are reported; its contents are not.
5. **The O-RAN Alliance's specification documents.** Gated. No interface name, document number, or functional split detail is asserted. The time-scale argument in section 3.11 is architectural reasoning that stands without the specification.
6. **Any vendor's actuation and permission mechanism.** No vendor product documentation was read for this guide. Section 9 describes vendor *classes* by design centre and asserts nothing about internals. Any statement of the form "vendor X's agent can do Y" is outside what this guide supports.
7. **Whether any operator runs autonomous network actuation in production, at what scope, with what evidence.** Not established publicly for any operator, and not asserted for any. Section 4.2's verdict of "unsettled" is the honest reading of the public record, not a claim that nothing exists.
9. **Any market size, market share, vendor ranking, deployment count, or percentage improvement.** None appears in this guide by design and none was sought. **Such figures do exist upstream** - TM Forum's own mission page carries vendor-style benefit claims about cost, customer satisfaction and energy, and the validation table carries a professional-certification listing ordered by a rank column. All of it was read and deliberately excluded, because in a document arguing that claims should be evidenced, importing an unevidenced benefit figure would undermine the argument.

---

## 16. The Glossary, the Cross-References, and the Closing Summary

### 16.1 The Glossary

| Term | Meaning in this guide |
|---|---|
| **Assurance** | Service assurance: the discipline of measuring whether a service is delivering what was contracted, and acting when it is not. Organised around the customer's service, not the element (section 6). |
| **Autonomy level (AN Level)** | TM Forum's 0-5 maturity score for autonomy in a *specific network scenario*, assessed with a per-scenario evaluation tool and independently validated on request. Fractional and "partial" scores exist, and validation does not certify performance (section 3). |
| **Blast radius** | The set of services and customers a proposed change could affect. Computable from topology and service mapping, and therefore computable by the platform rather than estimated by the model (section 10.4). |
| **BSS** | Business Support Systems: the commercial layer - customer, product catalogue, order, billing, charging, credit, collections, care. Owns money and identity; its dangerous permission is the order, which becomes a network change by proxy (section 8). |
| **Closed loop** | A control cycle that observes, decides, acts and re-observes without external intervention. ETSI's ZSM architecture names the stages Observe, Orient, Decide, Act (section 3.10). |
| **Element (network element, NE)** | A physical or virtual function that forwards, terminates or manages traffic: a base station, router, switch, core function, optical terminal. The element is where state is physically real; everything above it is a representation. |
| **Intent** | A declarative statement of a desired outcome with the mechanism left to the system. The interface that makes a permission envelope expressible, because an envelope is a statement about outcomes and boundaries rather than a command sequence. |
| **Inventory** | The OSS system of record for what exists and how it is connected: elements, links, circuits, services. Its drift from reality is the domain's oldest unsolved problem, and the source of most wrong-object actions (sections 2.3 and 9.4). |
| **MOP** | Method of procedure: the written, reviewed sequence a change follows, including verification and rollback. The artefact an agent would have to execute or replace (section 10.2). |
| **NOC** | Network Operations Centre: the function that watches the network in shifts, triages alarms and invokes incident and change processes. The organisational home of the fault-management loop (section 5). |
| **NRM** | Network Resource Model: 3GPP's standardised object model of the managed network, with the 5G variant specified in TS 28.541. Describes what exists and what is configurable, not who may configure it (section 3.8). |
| **O-RAN** | The O-RAN Alliance's architecture for open radio access networks: radio control split into loops of different time scales, with a controller that hosts third-party applications. Described structurally here; the specifications are membership-gated (section 3.11). |
| **OSS** | Operations Support Systems: the telecom layer that observes and commands the network - fault, performance, configuration, inventory, provisioning, assurance. **In the rest of this repository, OSS means open-source software** (section 1.5). |
| **RIC** | RAN Intelligent Controller: in the O-RAN architecture, the controller that hosts control applications and mediates between the slower policy loop and the faster radio loop. Described structurally, not cited to a specification (section 3.11). |
| **Rung** | A step on this guide's verb ladder - read, summarise, recommend, stage, execute with approval, execute with notification, execute autonomously (section 10.2). Not a maturity score: a grant. |
| **RAN** | Radio Access Network: the radio edge, base stations and the functions that control them. Written uppercase throughout, because the lowercase English verb is a different word (section 1.5). |
| **eTOM / SID** | TM Forum's process framework and shared information model for the commercial and operational domain. Named here as frameworks; their current document identifiers were not re-read this pass (section 9.1). |

### 16.2 How the Pieces Fit

The domain argument lives here and nowhere else. Agent mechanics are owned elsewhere in this repository and are cited rather than summarised: [ai_llm/agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md) for how a decision is made, [ai_llm/agent_tool_routing_guide.md](ai_llm/agent_tool_routing_guide.md) for how a granted verb becomes an executed call, [ai_llm/agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md) for containment, [ai_llm/agentops_guide.md](ai_llm/agentops_guide.md) for running agents in production, [ai_llm/agent_versioning_guide.md](ai_llm/agent_versioning_guide.md) and [ai_llm/ai_agent_drift_guide.md](ai_llm/ai_agent_drift_guide.md) for why a prompt change is a change to a control surface, and [ai_llm/ai_governance_framework_guide.md](ai_llm/ai_governance_framework_guide.md) for the governance scaffolding this domain instantiates. The estate argument is owned by [../singapore/starhub_software_systems_guide.md](../singapore/starhub_software_systems_guide.md), which describes one operator's BSS/OSS stack, provisioning chain, RAN and core, and fibre estate layer by layer - the concrete instance this guide deliberately does not re-derive. The vendor side of the same estate is owned by [nec_asia_pacific_guide.md](nec_asia_pacific_guide.md). The counting discipline that section 1.5 applies is owned by [measurement_lab_guide.md](measurement_lab_guide.md), and the regulated-institution argument is owned by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md). What none of them owns, and what this guide claims, is the join: what an agent may be permitted to do, layer by layer, in a live public network, and what evidence makes that permission defensible afterwards.

### 16.3 The Closing Summary

1. **The constraint is permission, not knowledge.** The observation surface is mature, standardised and abundant; the actuation governance is not standardised at all, and is the actual design problem. Section 1.
2. **The domain is three layers, and permission is a property of the object, not the agent.** Network, OSS and BSS differ in data model, change regime and cost of error; reversibility is the variable that should drive the grant, and it is not uniform across the layers. Section 2.
3. **Authority is a six-element tuple, and most claims collapse to its first two elements.** Verb, scope, blast radius, reversibility, window, evidence. Most public "agents" sit at read, summarise and recommend; the verb ladder is the diagnostic, and blast radius must be computed outside the model. Section 10.
4. **The industry graded the ladder per scenario, and said plainly that the grade is not performance.** Fractional and partial scores, a disclaimer that validation "does not certify that the network actually performs at the level claimed", and most operators between Levels 2 and 3. The vocabulary moved to agents; the instrument did not. Section 3.
5. **The standards specify observation exhaustively and actuation hardly at all.** 3GPP models the objects and the analytics; ETSI models the loop and the domains, and its end-to-end service domain is forbidden from directly managing infrastructure - the structural answer to the proxy problem. Sections 3 and 8.
6. **Almost every agent claim in this domain is a model claim, and the exceptions are in OSS.** Field dispatch plans, acts, observes and adapts; network fault management mostly diagnoses. The least glamorous agent is the real one. Sections 4 and 7.

The evidence from the public record is consistent and uncomfortable. The scenarios the industry assesses most are the ones it acts on least, and it says so itself in a disclaimer that most of its own marketing ignores. The permission machinery that would let an agent act exists in every operator - change advisory boards, MOPs, change windows, segregation of duties - and it was built for humans moving tickets, which is why it does not survive the proxy handoff, the unreadable diff, or the envelope granted against a topology that has since changed.

None of that is a model problem, and none of it will be fixed by a better model. It is fixed by writing down what the agent may do, in the unit the operator actually manages risk in, and enforcing it somewhere that does not know what a language model is.

Which is why one sentence carries the whole argument, and why it is written the way it is:

a telecom agent is not limited by what it knows about the network but by what it is allowed to change on it - and the industry graded that permission before it built the agents.
