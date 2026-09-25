# InfoSec Release Readiness

**The Release-Gate Deep-Dive — the Overview, the Identity and the Decoder; Where Release Readiness Sits (versus the Security Review, the Penetration Test, the Vulnerability Scan and the Change Approval); the Evidence Set and What Each Item Does *Not* Prove; the Standards Frame Verified at the Issuing Bodies (NIST SP 800-218 / SSDF, OWASP ASVS 5.0.0 and SAMM 2.2.0, SLSA, SPDX and CycloneDX, the EU Cyber Resilience Act, PCI DSS v4.0.1); the Record and the Audit Trail; the Gate Design (Blocking versus Warning, the Break-Glass Path); the Decision and the Sign-Off (Go / Conditional Go / No-Go, Advising versus Owning); Risk Acceptance and Exceptions (Owner, Rationale, Compensating Control, Expiry); the Gate in the Delivery Pipeline; the Regulated-Institution Angle (Change Governance, Operational Resilience, MAS, PCI, Independence); the Theatre Problem; the Metrics That Tell You Whether the Gate Is Real; the Cymbal Bank Worked Example; the Anti-Patterns (Symptom / Cause / Guardrail); the Claims Audit; and What Could Not Be Verified — the Final Word: a Gate That Has Never Stopped Anything Is Not a Gate**

> **Author:** Jack Liu Shurui, Solution Architect
> **Purpose:** Research reference for the information-security **release-readiness gate** — the decision that stands between a built release and a shipped one. The guide covers what release readiness actually is and where it sits in the lifecycle; what the gate asks (is the residual risk acceptable to ship, and who says so); the evidence set and, item by item, what each artefact proves and what it does not; the standards frame verified at the issuing bodies with versions and dates; the release-readiness record and the reconstruction question; the gate's design (automated versus judgement checks, blocking versus warning, false-positive survivability, the break-glass path); the decision and the sign-off and the difference between *advising* and *owning* residual risk; risk acceptance, exceptions and compensating controls with the owner / rationale / control / expiry quadriga; the gate inside the delivery pipeline; the regulated-institution frame (change governance, operational resilience, supervisor expectations, the card-industry standard, and the independence question); the theatre failure mode analysed structurally; the small set of metrics that say whether the gate is real; a worked example at a Cymbal Bank; the anti-patterns; the claims audit; and the honest ledger of what this pass could not verify.
> **Repo:** https://github.com/jackliusr/research
> **Scope note:** this guide **owns the release-readiness gate itself** — the evidence set, the standards frame, the record, the gate design, the decision and sign-off, risk acceptance and exceptions, the pipeline integration, the regulated-institution frame, the theatre failure mode, the metrics and the worked example. It deliberately does **not** re-derive the disciplines its near-neighbours own, and cross-references them by name throughout.
> **Primary Sources:** the standards bodies' own publications, read this pass — **NIST SP 800-218, *Secure Software Development Framework (SSDF) Version 1.1*, February 2022 (36 pp., doi 10.6028/NIST.SP.800-218)** ✅ and the **CSRC SSDF project page** ✅ (the four practice groups PO/PS/PW/RV, the four-part practice definition, SP 800-218A the GenAI community profile); **owasp.org** — the **ASVS project page** ✅ (latest stable **5.0.0**, the `v<version>-<chapter>.<section>.<requirement>` identifier convention) and the **SAMM project page** ✅ (flagship, Standards classification, **latest version 2.2.0**); **slsa.dev** ✅ (the **v1.0 specification's Build track, levels L0–L3**, and the standing notice that **v1.2 is the current version**; the v0.1 → v1.0 change that removed the Source aspects and retired the single unnamed SLSA 1–4 track); **spdx.dev** ✅ (SPDX as the **ISO/IEC 5962:2021** international open standard, with the project's own release timeline: 2.2 in May 2020 satisfying NTIA minimum elements, 2.2.1 to ISO in August 2020, ISO/IEC 5962 available August 2021, 2.3 in August 2022, 3.0.0 released April 2024); **cyclonedx.org** ✅ (current version **1.7**, release date **2025-10-21**, standardised as **ECMA-424** published **2025-12-10**, developed by the OWASP Foundation with Ecma International under TC54); **digital-strategy.ec.europa.eu** (the European Commission's own pages) ✅ for the **Cyber Resilience Act, Regulation (EU) 2024/2847** — in force **10 December 2024**, Chapter IV conformity-assessment notifications from **11 June 2026**, Article 14 reporting obligations from **11 September 2026**, full application from **11 December 2027**; **pcisecuritystandards.org** ✅ (the **PCI DSS v4.0.1** standard listed as the current PCI DSS document in the Council's own document library, and the **PCI Software Security Framework** assessor/vendor categories in the Council's own listing); the repo's verified regulatory cross-references for **MAS** (the TRM Guidelines published **18 January 2021**; the sectoral TRM notices cancelled with effect from **10 May 2024** and replaced by the harmonised FSMA notices — **FSM-N05** for banks, **FSM-N22** for cyber hygiene) and **DORA (Regulation (EU) 2022/2554, applied 17 January 2025)**; and one research paper for the checklist framing — **Patel, Boucher, Fallahzadeh, Hassan & Adams, "A State-of-the-practice Release-readiness Checklist for Generative AI-based Software Products", arXiv:2403.18958v1, 27 March 2024** ✅ (retrieved from the arXiv API over HTTPS this pass). **Tooling note:** `web_search` returns empty from this host — a tool limitation, **not** evidence of absence; every source above was reached by direct extraction of the issuing body's own page. Anything that could not be read at the issuing body is flagged ⚠ rather than paraphrased from a vendor summary.
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — plain filenames):** [Security-by-Design](security_by_design_guide.md) (**THE design-time companion and this guide's closest neighbour — it owns building security *in*: §2 threat modelling, §3 design principles, §4 secure coding, §5 security testing, §6 DevSecOps/CI-CD gates/SBOM/SLSA, §7 cloud security, §8 regulatory, §9 its own worked example. This guide cross-refs §2, §5 and §6 heavily and does **not** re-derive threat modelling, testing technique or DevSecOps practice**), [Cybersecurity](cybersecurity_guide.md) (**THE programme companion — it owns the security programme: the threat landscape, security operations, offensive security, the security organisation, the frameworks map, the domains map and the banking context; this guide cross-refs it for the organisational and operations frame rather than restating it**), [Technology Lifecycle Management](technology_lifecycle_management_guide.md) (**THE lifecycle-governance companion — §2 phases, §3 vendor EOL/EOSL, §4 TLM as technology risk, §5 MAS technology-risk expectations, §7 inventory/tracking/governance gates, §8 software lifecycle including patches and supply chain. **Note carefully: its §7 "Governance Gates" is about *lifecycle inventory* gates — knowing what you have and what state it is in — not about release readiness; the two gate families touch but they are not the same gate**), [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) (**THE supply-chain-inventory deep-dive for the COBOL/procedural-language estate** — SBOM generation, tooling and the EO 14028 / OMB M-22-18 / SP 800-218 lineage), [Vulnerability Scanning for C, Pro\*C and COBOL](vuln_scanning_c_proc_cobol.md) (the scanning reality for the procedural estate — the repo's single existing SSDF citation lives here), [Power Platform CI/CD](power_platform_cicd_guide.md) (**THE low-code delivery-pipeline companion** — pipeline promotion and environment gates in the Microsoft low-code estate), [LLM Development Risks & Security](llm_development_risks_security_guide.md) (the GenAI-release risk angle — model and prompt-release readiness), [Beyond Zero: Enterprise Security for the AI Era](beyond_zero_enterprise_security_guide.md) (the agent-era paradigm — where "release" stops being a single event), [Dockerfile Best Practices](dockerfile_best_practices_guide.md) and [Docker Image Comparison Tools](docker_image_comparison_tools_guide.md) (the image/artefact-integrity and SBOM-diff angle), [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md) (the platform-side admission and supply-chain controls), [Deterministic Engineering](deterministic_engineering_guide.md) (reproducibility as the precondition for a trustworthy build claim)
> **Companion guides (banking/, prefix `../banking/`):** [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) (**THE operational-resilience companion — it owns the DORA (Regulation (EU) 2022/2554) and MAS operational-resilience frame, the important-business-service and impact-tolerance machinery, third-party resilience and the testing regime; this guide cross-refs it by name and does not re-derive it**), [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) (the risk-taxonomy container the release gate's accepted risks land in), [Risk Management Models](../banking/risk_management_models_guide.md) (the risk-measurement vocabulary), [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) (the verified MAS instrument map — TRMG, FSM-N05, FSM-N22)
> **Companion guides (management/, prefix `../management/`):** [Resilience Engineering](../management/resilience_engineering_guide.md) (the sociotechnical angle on why gates are routed around — cross-ref §11), [Vendor Management](../management/vendor_management_guide.md) (the third-party assessment and exit frame the supply-chain evidence leans on), [Team Lead Methodologies](../management/team_lead_methodologies_guide.md) (the delivery-management context — the repo's single passing use of the phrase "release readiness" sits in its §702)

---

**How to use this guide:** Section 1 is the overview — what release readiness is, the one-line thesis, the vocabulary decoder, and the boundary declared against the four near-neighbour guides by name. Section 2 positions release readiness — what it is *not* (a security review, a penetration test, a vulnerability scan, a change approval) and the question the gate actually asks. Section 3 is the evidence set — each artefact with what it proves and what it does not. Section 4 is the standards frame, verified per standard at the issuing body with version and date. Section 5 is the record and the audit trail. Section 6 is the gate design. Section 7 is the decision and the sign-off. Section 8 is risk acceptance and exceptions — the guide's most valuable section. Section 9 is the gate in the delivery pipeline. Section 10 is the regulated-institution angle. Section 11 is the theatre problem, analysed structurally. Section 12 is the metrics. Section 13 is the Cymbal Bank worked example. Section 14 is the anti-patterns. Section 15 is the claims audit. Section 16 is what could not be verified, the glossary, the cross-references and the closing summary. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` and `management/` are prefixed `../banking/` and `../management/`; guides in the `ai_llm/` subdirectory are prefixed `ai_llm/`. **Integrity convention:** ✅ = verified this pass against the issuing body's own publication (or verified in the cross-referenced guide's own ledger); ⚠ = flagged / unverified, and never to be relied on without a re-check; ⚠-structural = standard industry practice widely documented but not attributable to a single primary source verified this pass. **The most important single idea of the whole guide:** *a release gate is a decision that transfers risk to a named person, not a checklist that transfers nothing — and a gate that has never stopped a release is not a gate.*

---

## Table of Contents

1. [The Overview, the Identity and the Decoder](#1-the-overview-the-identity-and-the-decoder)
   - 1.1 [The Short Answer, and the Thesis in One Line](#11-the-short-answer-and-the-thesis-in-one-line)
   - 1.2 [What Release Readiness Is, and Where It Sits](#12-what-release-readiness-is-and-where-it-sits)
   - 1.3 [The Decoder — the Vocabulary of the Gate](#13-the-decoder--the-vocabulary-of-the-gate)
   - 1.4 [The Overview Table — Aspect / Description](#14-the-overview-table--aspect--description)
   - 1.5 [The Boundary, Declared](#15-the-boundary-declared)
2. [Where Release Readiness Sits](#2-where-release-readiness-sits)
   - 2.1 [The Lifecycle Position](#21-the-lifecycle-position)
   - 2.2 [What It Is Not — Four Distinctions That Matter](#22-what-it-is-not--four-distinctions-that-matter)
   - 2.3 [The Distinction Table](#23-the-distinction-table)
   - 2.4 [The Question the Gate Actually Asks](#24-the-question-the-gate-actually-asks)
   - 2.5 [The Consequence That Follows](#25-the-consequence-that-follows)
3. [The Evidence Set](#3-the-evidence-set)
   - 3.1 [Evidence Is the Gate's Currency](#31-evidence-is-the-gates-currency)
   - 3.2 [The Evidence Set — Item by Item](#32-the-evidence-set--item-by-item)
   - 3.3 [The Evidence Table — What It Proves / What It Does Not](#33-the-evidence-table--what-it-proves--what-it-does-not)
   - 3.4 [A Green Scanner Is Not a Low-Risk Release](#34-a-green-scanner-is-not-a-low-risk-release)
4. [The Standards Frame, Verified](#4-the-standards-frame-verified)
   - 4.1 [How to Read a Standards Frame Without Fabricating One](#41-how-to-read-a-standards-frame-without-fabricating-one)
   - 4.2 [The US Federal Frame — SSDF / NIST SP 800-218 and Its Lineage](#42-the-us-federal-frame--ssdf--nist-sp-800-218-and-its-lineage)
   - 4.3 [The OWASP Frame — ASVS and SAMM](#43-the-owasp-frame--asvs-and-samm)
   - 4.4 [The Supply-Chain Frame — SLSA, SPDX and CycloneDX](#44-the-supply-chain-frame--slsa-spdx-and-cyclonedx)
   - 4.5 [The EU Instrument — the Cyber Resilience Act](#45-the-eu-instrument--the-cyber-resilience-act)
   - 4.6 [The Card-Industry Standard — PCI DSS](#46-the-card-industry-standard--pci-dss)
   - 4.7 [The Verified Standards Table — Requirement / What It Does Not Require / Verified At / Date](#47-the-verified-standards-table--requirement--what-it-does-not-require--verified-at--date)
5. [The Record and the Audit Trail](#5-the-record-and-the-audit-trail)
6. [The Gate Design](#6-the-gate-design)
7. [The Decision and the Sign-Off](#7-the-decision-and-the-sign-off)
8. [Risk Acceptance and Exceptions](#8-risk-acceptance-and-exceptions)
9. [The Gate in the Delivery Pipeline](#9-the-gate-in-the-delivery-pipeline)
10. [The Regulated-Institution Angle](#10-the-regulated-institution-angle)
11. [The Theatre Problem](#11-the-theatre-problem)
12. [The Metrics That Tell You Whether the Gate Is Real](#12-the-metrics-that-tell-you-whether-the-gate-is-real)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns — Symptom / Cause / Guardrail](#14-the-anti-patterns--symptom--cause--guardrail)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Identity and the Decoder

### 1.1 The Short Answer, and the Thesis in One Line

**Release readiness is the state of having enough evidence, reviewed by the right people, to make a defensible decision that a specific release's residual risk is acceptable to ship.** It is not a document, a meeting or a ticket type. It is the point at which an organisation stops discovering what a release contains and starts *owning* what it contains.

The thesis of this guide is one line:

> **A release gate is a decision that transfers risk to a named person, not a checklist that transfers nothing — and a gate that has never stopped a release is not a gate.**

Each half of that sentence carries weight. *A decision that transfers risk to a named person* — the output of a working gate is not a green tick, it is a human being who has accepted a specific, bounded risk and whose name is on it. *Not a checklist that transfers nothing* — a checklist distributes attention across boxes and responsibility across nobody; when the release goes wrong, the checklist cannot be asked why. And *a gate that has never stopped a release is not a gate* — the only evidence that a control has authority is that it has exercised authority, which means at least one release was delayed, conditioned, or refused. A gate with a spotless record of never obstructing anything is not a well-tuned gate; it is either a very lucky organisation or, far more often, a decorative one.

### 1.2 What Release Readiness Is, and Where It Sits

A release is a *promotion event*: a set of artefacts moves from an environment where mistakes are cheap (a build pipeline, a lower environment) to an environment where mistakes are expensive (production, customers, regulated data). Release readiness is the discipline of the moment immediately before that promotion.

Three properties define the discipline:

1. **It is a point in time.** Unlike a security programme (continuous) or a threat model (living), the release gate is *episodic* — it exists at a decision boundary. This is its weakness (it can be treated as a ceremony) and its strength (it is the last moment at which a problem can still be stopped cheaply, before customers see it).
2. **It is evidence-consuming.** The gate does not *create* security; it adjudicates. The material it adjudicates — threat models, test results, scan output, dependency inventories, pentest findings — is produced by activities that happened earlier and elsewhere. §3 is about that material; §9 is about where it is produced.
3. **It is a risk decision, not a compliance decision.** Compliance asks "is the control in place?"; readiness asks "is the residual risk acceptable, and who says so?" Those two questions have different failure modes, and conflating them is the origin of most theatre (§11).

Where it sits: after the security activities (design review, secure coding, testing, scanning, penetration testing) have produced their outputs, and before the change-management system authorises deployment. In a modern pipeline the gate is partly automated and partly human (§6); in a regulated institution it also feeds the change-approval frame (§10).

### 1.3 The Decoder — the Vocabulary of the Gate

The vocabulary of release readiness is unusually overloaded, because it borrows from security, from delivery engineering, from risk management and from change management at once. The decoder below fixes the sense in which this guide uses each term.

| Term | What it means here | The trap |
|---|---|---|
| **The release** | A named, identifiable set of artefacts (a build, an image set, a configuration state, a model version) proposed for promotion to a target environment | Treating "the release" as the *project* or the *sprint*, which has no artefact identity and therefore cannot be gated |
| **The gate** | The control point that must be satisfied before promotion proceeds — a mix of automated checks and human judgement with the authority to stop the promotion (§6) | Confusing the gate with the *meeting* or the *ticket*, which are the gate's administrative shadow |
| **The readiness review** | The structured examination of the evidence set against the release's risk profile, producing a recommendation (§5, §7) | Running the review after the deployment window has been announced, which converts a review into a ratification |
| **The sign-off** | A named individual's recorded acceptance of the decision and its residual risk (§7) | A sign-off by someone who cannot say no — a signature that carries no authority transfers no risk |
| **Residual risk** | The risk that remains after the controls in place, which the release carries into production | Believing residual risk can be *zero*; the question is never "is there risk?" but "is this the risk we mean to carry?" |
| **The exception / waiver** | A time-boxed, owned, justified decision to ship without meeting a stated requirement or criterion (§8) | An exception with no expiry, which is not an exception but a silent amendment to the standard |
| **The compensating control** | A different control that addresses the same risk by another route when the primary control is absent or unmet (§8) | Naming a control that does not actually reduce the risk, which converts a compensating control into a comfort blanket |
| **The go / no-go** | The decision vocabulary: go, conditional go (ship with named conditions and an owner for each), no-go (§7) | Collapsing everything into "go with actions", which is a no-go in disguise — the release ships and the actions evaporate |
| **Security debt** | The accumulated, uncarried backlog of unmet security expectations that later releases inherit | Measuring debt as a count of findings rather than as an ageing exposure with a consequence attached |
| **The evidence artefact** | An object that records what was done and what was found: a report, a signed attestation, a stored provenance record, a log (§3, §5) | Confusing an artefact's *existence* with its *availability* — evidence you cannot produce on demand is not evidence (§5) |

Two more terms are used in the guide with deliberate care:

- **Readiness** versus **quality.** Readiness is about *this* release's fitness to ship now; quality is about the product's general standard. A high-quality product can have an unready release (a dependency changed, a threat model went stale) and a mediocre product can have a perfectly ready one. Conflating them is what produces the "we always ship, we're a quality shop" argument.
- **Gate** versus **guardrail.** A gate can *stop* a promotion. A guardrail only *warns*, *reports* or *routes around* (§6.3). Calling a guardrail a gate is one of the most common failures in the field, and it is exactly the failure the thesis sentence is written to catch.

### 1.4 The Overview Table — Aspect / Description

| Aspect | Description | Section |
|---|---|---|
| **Definition** | The state of having enough reviewed evidence to make a defensible decision that a release's residual risk is acceptable | §1.1–1.2 |
| **The thesis** | A gate is a decision that transfers risk to a named person; a gate that has never stopped a release is not a gate | §1.1 |
| **Position** | Episodic, at the promotion boundary; evidence-consuming rather than evidence-creating | §1.2, §2.1 |
| **What it is not** | Not a security review, not a pentest, not a vulnerability scan, not a change approval | §2.2–2.3 |
| **The question** | Is the residual risk acceptable to ship, and who says so — a risk question, not a compliance question | §2.4 |
| **The evidence set** | Threat-model currency; SAST/DAST/SCA results; pentest findings *and their disposition*; open-vulnerability position with severity **and age**; dependency and supply-chain state including SBOM and provenance; secrets and configuration posture; pipeline integrity | §3 |
| **The standards frame** | SSDF/NIST SP 800-218; OWASP ASVS and SAMM; SLSA; SPDX and CycloneDX; the EU Cyber Resilience Act; PCI DSS — each verified at the issuing body with version and date | §4 |
| **The record and the decision** | The stored, retrievable artefact that lets the decision be reconstructed; go / conditional go / no-go by a named person who can say no | §5, §7 |
| **Risk acceptance** | Owner, rationale, compensating control, expiry — and the loan against the future repaid with interest in remediation | §8 |
| **The gate in the pipeline** | Where checks run, what blocks a build versus a release, the artefact-provenance question | §9 |
| **The regulated frame** | Change governance, operational resilience, supervisor technology-risk expectations, the card-industry standard, independence | §10 |
| **The failure mode and the metrics** | Theatre — a property of badly-designed gates; and the five diagnostic numbers, plus the one metric that cannot be gamed | §11, §12 |
| **The final word** | A gate that has never stopped anything is not a gate | §16 |

### 1.5 The Boundary, Declared

This guide owns **the release-readiness gate**: its evidence set, its standards frame, its record, its design, its decision and sign-off, its exceptions and risk acceptance, its pipeline integration, its regulated-institution frame, its theatre failure mode, its metrics, and the worked example. Four near-neighbours own the material this guide consumes, and this guide does not re-derive any of it. **[Security-by-Design](security_by_design_guide.md)** owns *building security in* — its §2 threat modelling, §3 design principles, §4 secure coding, §5 security testing, §6 DevSecOps (CI/CD gates, secret management, SBOM and SLSA), §7 cloud security, §8 regulatory and §9 worked example are the producing disciplines whose *output* this gate adjudicates; this guide cross-references §2, §5 and §6 heavily and assumes their technique rather than restating it. **[Cybersecurity](cybersecurity_guide.md)** owns *the security programme* — the threat landscape, security operations, offensive security, the security organisation, the frameworks map and the domains map — and this guide takes the organisational frame (who the security function is, what the SOC and the offensive function produce) from it rather than rebuilding it. **[Technology Lifecycle Management](technology_lifecycle_management_guide.md)** owns *lifecycle governance* — its §2 phases, §3 vendor EOL/EOSL, §4 technology risk, §5 MAS technology-risk expectations, §7 inventory/tracking/governance gates and §8 software lifecycle including patches and supply chain; and it is important to be blunt here: **TLM's §7 "Governance Gates" are lifecycle *inventory* gates — they answer "do we know what this asset is, what state it is in, and when its support ends?" — and they are not release-readiness gates.** The two gate families touch at the point where a release's dependency state depends on an inventory fact (whether a component is still supported), and this guide cross-references TLM for that fact rather than owning it. **[Operational Resilience Framework](../banking/operational_resilience_framework_guide.md)** owns the **DORA / MAS operational-resilience frame** — important business services, impact tolerances, third-party resilience and the testing regime — and this guide cross-references it by name for the regulatory container in which a release decision sits, without re-deriving its machinery. Where this guide needs a neighbouring discipline's content, it names the guide and the section; where it states something itself, that statement is this guide's own.

---

## 2. Where Release Readiness Sits

### 2.1 The Lifecycle Position

A useful way to place the release gate is to say what it is the *last cheap moment* of. Every security activity has a cost curve, and the release gate sits near the expensive end of it: a design flaw caught in a threat model costs a conversation; the same flaw caught in the release gate costs a delay; the same flaw caught in production costs an incident. The release gate is therefore not the *best* place to find a problem — it is the *last* place a problem can still be found before a customer can find it.

That produces the gate's characteristic double duty, and the tension running through this whole guide:

- **It is a safety net.** Its existence is an admission that earlier stages miss things. If nothing ever reached the gate, it would be redundant.
- **It is not a substitute for the earlier stages.** A gate that is *discovering* problems rather than *adjudicating* them is failing at its design: it has become the quality-assurance stage that the earlier stages were supposed to make unnecessary. A gate doing all the finding is a symptom that §9's pipeline work is not being done.

The clean division of labour: the earlier stages **find and fix**; the gate **decides whether what remains is acceptable**. The gate can find things — and sometimes it must — but a gate whose consistent role is discovery is a pipeline telling you something.

### 2.2 What It Is Not — Four Distinctions That Matter

Release readiness is routinely confused with four neighbours. Each confusion has a cost.

**Not a security review.** A security review is **per-change, design-time**. It examines a proposed change — a new integration, a new data flow, a new architecture — and asks whether the design is safe. It is triggered by *design activity*, it is timed to *before the code is written*, and its output is design guidance. The release gate is triggered by *promotion activity*, timed to *after everything is built*, and its output is a ship/don't-ship decision. The repository's design-time discipline is [Security-by-Design](security_by_design_guide.md) §2 (threat modelling) and §3 (design principles); a threat model produced there becomes *evidence* at the gate (§3.2, item 1) — which is the point: the gate consumes the review, it does not perform it.

**Not a penetration test.** A penetration test is **periodic and sample-based**. It takes a trained human, a scope, and a window, and produces an adversarial, end-to-end attempt against *a sample of the attack surface at a point in time*. Its findings are enormously valuable and its coverage is inherently incomplete — no pentest proves the absence of a vulnerability. A pentest completed three months before the release and not re-scoped since is *evidence of the state of the system three months ago*; treating it as evidence of the state of the release is a category error, and one of the most frequent at the gate. Note carefully: **it is the *disposition* of pentest findings that matters at the gate, not the count.** "Twelve findings, all closed" and "one finding, closed" and "twelve findings, nine accepted" are different risk positions, and only the last two contain a decision. §3.3 makes this explicit.

**Not a vulnerability scan.** A vulnerability scan is **continuous and noisy**. It runs on a schedule against running systems, it generates findings at a rate no organisation can triage in full, and its signal-to-noise ratio is a function of tuning rather than of truth. A scan is an *input* to the gate's open-vulnerability position (§3.2, item 4) — with severity and, crucially, **age** attached — and it is not a readiness verdict. The deep treatment of the scanning reality in the repository's procedural-language estate is [Vulnerability Scanning for C, Pro\*C and COBOL](vuln_scanning_c_proc_cobol.md); the general testing arsenal is [Security-by-Design](security_by_design_guide.md) §5.

**Not a change-approval process.** A change-approval process is **broader and usually not security-specific**. It handles scheduling, impact analysis, rollback plans, communications, maintenance windows, dependency ordering across the estate — a great deal that has nothing to do with security, and a great deal of security that it does not see. The release gate *feeds* the change-approval process with a security verdict; it does not replace it, and the change-approval process does not replace the gate. §10 develops the regulated-institution version of that interface, where change governance is a supervised control in its own right.

### 2.3 The Distinction Table

| Neighbour | Trigger | Timing | Coverage model | Output | What the gate takes from it |
|---|---|---|---|---|---|
| **Security review** | A design change | Design-time, pre-code | The design under discussion | Design guidance, mitigations | The threat model and its currency (§3.2 item 1) |
| **Penetration test** | A schedule or a major change | Periodic, bounded window | Sample-based and adversarial | Findings with severities | The findings **and their disposition** (§3.2 item 3) |
| **Vulnerability scan** | Automation, continuous | Continuous | Broad, noisy, tuneable | A finding stream | The open-vulnerability position with severity **and age** (§3.2 item 4) |
| **Change approval** | A proposed change | Pre-deployment | Whole-change, mostly non-security | Authorisation to deploy | The deployment envelope the verdict plugs into (§10) |
| **Release readiness (this guide)** | A promotion event | At the promotion boundary | The named release, all evidence | Go / conditional go / no-go, with an owner | — |

The table is the fastest way to answer the question that derails the most readiness conversations: *"but we already did a security review."* A security review is a design-time artefact; the release is a shipment; the two are related by **consumption**, not by **substitution**.

### 2.4 The Question the Gate Actually Asks

The gate asks a risk question, and the precise wording matters:

> **Is the residual risk of shipping this release acceptable, and who says so?**

Reading the question closely:

- **"residual"** — not the risk before controls, and not the risk in theory. The question is entirely about what remains *after* everything the organisation has built and tested. That is a much smaller and much more specific quantity than "is this system secure?"
- **"acceptable"** — a judgement against a standard. And the standard is not "acceptable in general"; it is acceptable *for this release, in this environment, at this time*. A release can be unacceptable this week (an unpatched critical in a component reachable from the internet) and acceptable next week (the patch shipped), with no change to the product at all.
- **"to ship"** — the decision's scope is the *shipment*, not the product's forever-state. A conditional go ships now with named conditions; that is a legitimate answer, and it is only legitimate if the conditions are owned and dated (§8).
- **"and who says so"** — the part that converts a quality question into a risk decision. A gate that answers "the evidence is good" has not answered the question. The question requires a person, a name, and a scope of authority (§7).

### 2.5 The Consequence That Follows

Because the gate asks a risk question, three consequences follow immediately, and they shape the rest of the guide: **someone must be able to say no**, because a risk decision made by a party who cannot refuse is not a decision but a notification (§7.3, and §11's "the sign-off goes to whoever is available"); **someone must own the residual risk after the decision**, because the gate transfers risk to a named person, not away from the organisation, and when that person is not identifiable a week later the organisation is carrying risk it never consciously accepted (§8); and **the evidence must be adjudicable, which is stricter than accurate**, because evidence that is accurate but arrives after the decision, or that cannot be produced on demand six months later, fails the gate's purpose even when its contents are correct (§5).

---

## 3. The Evidence Set

### 3.1 Evidence Is the Gate's Currency

The NIST SSDF defines an **artifact** as "a piece of evidence", and **evidence** as "grounds for belief or disbelief; data on which to base proof or to establish truth or falsehood" — and adds that artifacts "provide records of secure software development practices" ✅ (NIST SP 800-218, February 2022). That phrasing is worth taking literally at the gate: evidence is what lets a decision-maker *believe* something about the release rather than *assume* it. A conversation is not evidence. A memory is not evidence. A dashboard screenshot with no provenance is a claim dressed as evidence.

The gate's evidence set is smaller than most teams assume. Nine items cover it. For each, the useful question is not "do we have one?" but the double question of §3.3: **what does this prove, and what does it not?**

### 3.2 The Evidence Set — Item by Item

**1. Currency of the threat model.** A release changes a system; the question is whether the change invalidated the model. Not "does a threat model exist" (it almost always does) but "is it *current for this release*" — does it cover the new integration, the new data flow, the new trust boundary, the new third-party component? This guide takes the modelling *method* entirely from [Security-by-Design](security_by_design_guide.md) §2 (STRIDE, DREAD, PASTA, attack trees) and the SSDF's own framing of the task: **PW.1.1** — "Use forms of risk modeling – such as threat modeling, attack modeling, or attack surface mapping – to help assess the security risk for the software" ✅ (SP 800-218). What the gate asks is narrower than modelling: *has the model been re-examined against this release's delta, and who says so?* **What it proves:** that someone thought about the release's new attack surface with a method. **What it does not:** that the model is complete, or that the mitigations it names were implemented. A stale-but-excellent threat model is a liability at the gate precisely because it *looks* like evidence.

**2. Static, dynamic and composition analysis results.** SAST, DAST and SCA output, from the pipeline, for this artefact. The technique is [Security-by-Design](security_by_design_guide.md) §5; what the gate needs from the output is not the finding count but the *disposition*: how many are open, at what severity, and what was decided about each. A scan result with no triage decision attached is raw material, not evidence. **What it proves:** that automated analysis ran against this artefact and produced these findings. **What it does not:** that the findings were adjudicated, or that the absence of findings means absence of the class of defect (see §3.4).

**3. Penetration-test findings *and their disposition*.** The findings matter; the disposition matters more. For each finding: was it fixed, accepted, mitigated by another control, or judged out of scope — by whom, and when? This is where the gate exercises judgement rather than arithmetic, and where the count is actively misleading (§2.2). **What it proves:** what a skilled adversary found in the tested scope at the tested time. **What it does not:** anything about the untested scope, or about the code that changed after the test.

**4. The open-vulnerability position, with severity and age.** The two mandatory dimensions are **severity** (how bad) and **age** (how long it has been known). A moderate finding open for nine months is a different risk from a moderate finding open for nine days, because age is the best available proxy for *whether the organisation is actually able to fix things*. Age also converts a static snapshot into a trend, which is the input §12's metrics need. **What it proves:** the current known-vulnerability state of the release's components. **What it does not:** whether the vulnerabilities are exploitable in *this* deployment's configuration — which is a judgement, and one the gate must make or delegate explicitly.

**5. Dependency and supply-chain state, including the SBOM and any provenance attestation.** What third-party and open-source components are in the artefact, at what versions, from where, built how, and by whom. The inventory artefact is the SBOM — the SSDF task is explicit: **PS.3.2** — "Collect, safeguard, maintain, and share provenance data for all components of each software release (e.g., in a software bill of materials [SBOM])" ✅ (SP 800-218). The formats and their standing are in §4.4; the repository's deep treatment for the procedural-language estate is [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md). **What it proves:** what the artefact is composed of and, where provenance exists, something about how it was built. **What it does not:** that the components are safe, that the SBOM is complete, or that the provenance was *verified* rather than merely *present* — a distinction §4.4 returns to.

**6. Secrets and configuration posture.** Evidence that the release does not ship credentials, keys or tokens in its artefacts or configuration, and that its configuration baseline is the secure one. This is a class where the failure is not incremental but binary: one committed key invalidates the release's security story. **What it proves:** that the artefact and its configuration were checked for this class of defect. **What it does not:** that the *runtime* environment is configured securely — a release can be clean and its target environment unhardened.

**7. The pipeline's own integrity.** The gate exists inside a delivery system, and the release's trustworthiness is bounded by that system's. Who can merge, who can trigger a production deployment, whether the build is reproducible or at least attributable, whether the artefact that was tested is the artefact that will ship (the *identity* question, not the *equivalence* question). The SSDF's **PS.3.1** task is the anchor — "Securely archive the necessary files and supporting data (e.g., integrity verification information, provenance data) to be retained for each software release" ✅ — and SLSA's levels (§4.4) are the ladder for how much of this can be *proven* rather than asserted. **What it proves:** something about how much of the release's construction is attributable and tamper-evident. **What it does not:** anything about the code's security quality. Pipeline integrity answers "is this the thing we built?" not "is this thing safe?"

**8. The change's blast radius and reversibility.** What the release touches, who is affected, and whether it can be undone — the rollback or forward-fix path, and its tested-ness. Not a security artefact in origin, but the single biggest determinant of how much risk a *given* release carries, and therefore central to the decision. **What it proves:** what the organisation understands about the release's footprint. **What it does not:** that the understanding is complete — releases that surprise their authors are precisely the ones whose readiness review under-scoped the footprint.

**9. The disposition of prior exceptions.** Every previous gate decision that shipped something with conditions produces an obligation. The gate's evidence set must include the status of those obligations, because an organisation that accumulates open conditions across releases is carrying a distributed liability it cannot see. The SSDF handles the general principle in **PW.1.2** — "Maintain records of design decisions, risk responses, and approved exceptions that can be used for auditing and maintenance purposes throughout the rest of the software life cycle", with the further task to "Periodically re-evaluate all approved exceptions to the security requirements, and implement changes as needed" ✅ (SP 800-218). §8 and §12 operationalise that.

### 3.3 The Evidence Table — What It Proves / What It Does Not

| Evidence item | What it proves | What it does **not** prove | Owner of the artefact |
|---|---|---|---|
| **Threat-model currency** | The release's delta was examined with a method, by someone | Completeness; that mitigations were implemented | Design / security architecture ([Security-by-Design](security_by_design_guide.md) §2) |
| **SAST / DAST / SCA results** | Automated analysis ran against this artefact | That findings were adjudicated; that absence of findings is absence of defect | Engineering, via the pipeline ([Security-by-Design](security_by_design_guide.md) §5) |
| **Pentest findings + disposition** | What an adversary found in the tested scope at the tested time | Anything about untested scope or later changes | Offensive security / the testing function |
| **Open vulnerabilities with severity + age** | The known-vulnerability state of the components | Exploitability in *this* configuration | Vulnerability management / engineering |
| **SBOM + provenance attestation** | Component inventory; something about how it was built | That components are safe; that the SBOM is complete; that provenance was *verified* | Software supply-chain owner ([SBOM](sbom_c_proc_cobol.md)) |
| **Secrets / configuration posture** | The artefact and its config were checked for the injected-secret class | That the target environment is hardened | Engineering / platform |
| **Pipeline integrity** | Attribution and tamper-evidence of the build route | Anything about code quality | Platform engineering ([Security-by-Design](security_by_design_guide.md) §6) |
| **Blast radius / reversibility** | The organisation's understanding of the footprint and the undo path | That the understanding is complete | Delivery lead / change management |
| **Prior-exception disposition** | The status of obligations from earlier decisions | That the obligations were met on time | The exception owner (§8) |

### 3.4 A Green Scanner Is Not a Low-Risk Release

This is stated here in the evidence section because it is the single most consequential misreading at the gate, and it deserves its own heading.

A scanner is **capable of only one kind of claim**: that the specific classes of defect it detects are absent from the specific artefacts it examined under the specific configuration in which it ran. Everything else about the release is outside its reach. A fully green scanner board does **not** establish that **the architecture is sound** (scanners find defects in code; they do not evaluate whether trust boundaries are in the right places, whether the design conflates authentication with authorization, or whether the system's fault model is coherent — that work is design-time, [Security-by-Design](security_by_design_guide.md) §2–§3); that **the business logic is safe** (the highest-impact vulnerabilities in financial systems are usually *logical* — a missing authorisation check on a specific transaction path, a workflow that can be driven in an unintended order — not the pattern classes a scanner recognises); that **the dependencies are in good standing** (a component can be vulnerability-free today and unsupported tomorrow; the lifecycle question belongs to [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §3 and §8, and the scan board will not raise it); or that **the runtime configuration matches the tested configuration** (a clean scan of an artefact and a misconfigured deployment produce an insecure production system).

And there is a fifth, deeper point: **a green board records the absence of findings in a tool's model of the world, and contains no judgement about acceptability — so it transfers no risk.** The gate's whole function is the judgement the scanner cannot make, which is why §7's question is "and who says so", and why a gate whose pass condition is "the board is green" has outsourced the decision to a tool's false-negative rate.

The corollary is uncomfortable and should be faced: **a release can be genuinely ready with a non-green scan board** (findings understood, accepted, compensated and owned) **and genuinely unready with a green one** (a stale threat model, an architecture the change invalidated, a dependency that went unsupported last month). The gate exists to make that distinction. A gate that cannot make it is doing arithmetic, not risk.

---

## 4. The Standards Frame, Verified

### 4.1 How to Read a Standards Frame Without Fabricating One

Release readiness is a *practice* before it is a *standard*: no single standard is titled "release readiness", and no standards body publishes a release-gate checklist that an organisation can simply adopt. The frame is assembled from instruments that each address part of the problem, and the discipline in assembling it is refusing to supply the parts that do not exist.

Three rules govern this section:

1. **No fabricated clause numbers.** A plausible-looking clause reference that does not exist is worse than no reference at all, because it survives into an audit document and then has to be retracted. Where this pass read the practice or the task at the issuing body, the identifier is given; where it did not, the identifier is withheld and the absence is stated.
2. **Every entry carries version, date and reading point.** The §4.7 table has `Verified at` and `Date` columns for exactly this reason. Standards move: SLSA restructured its levels at v1.0 and is now at v1.2; SPDX's specification line has moved from 2.2 to 3.0.0; CycloneDX is at 1.7 with an Ecma standardisation. A frame quoted from memory is a frame quoted from the wrong version.
3. **A standard's silence is a finding.** For each instrument, what it *does not* require is as load-bearing as what it does. Most "the standard requires X" errors at the gate are claims about something the standard never addressed.

A note on what a standards frame can and cannot do for a gate. Standards describe **practices and outcomes**; the gate needs **evidence and a decision**. The SSDF says this about itself in terms that every gate designer should internalise: "The intention of the SSDF is not to create a checklist to follow, but to provide a basis for planning and implementing a risk-based approach" ✅ (SP 800-218, §1). A gate built as a literal adoption of a framework's structure has misread the framework — and §11 shows exactly what that misreading produces.

### 4.2 The US Federal Frame — SSDF / NIST SP 800-218 and Its Lineage

**NIST SP 800-218, *Secure Software Development Framework (SSDF) Version 1.1: Recommendations for Mitigating the Risk of Software Vulnerabilities*, February 2022** ✅ — read this pass at `csrc.nist.gov` (publication page and the SSDF project page) and at the NIST publication server (the full 36-page document; doi 10.6028/NIST.SP.800-218). Authors: Murugiah Souppaya (NIST), Karen Scarfone (Scarfone Cybersecurity), Donna Dodson. It supersedes CSWP 13 (April 2020), which defined SSDF version 1.0; the final was posted 3 February 2022, with the draft on 30 September 2021.

**What it actually requires.** The SSDF is a set of **fundamental, sound, and secure software development practices**, organised into **four practice groups** — the CSRC project page and SP 800-218 §2 both name exactly these four: *Prepare the Organization (PO)* ("Ensure that the organization's people, processes, and technology are prepared to perform secure software development"); *Protect the Software (PS)* ("Protect all components of the software from tampering and unauthorized access"); *Produce Well-Secured Software (PW)* ("Produce well-secured software with minimal security vulnerabilities in its releases"); and *Respond to Vulnerabilities (RV)* ("Identify residual vulnerabilities in software releases and respond appropriately to address those vulnerabilities and prevent similar vulnerabilities from occurring in the future") ✅. Each practice is defined with four elements: **Practice** (name and unique identifier), **Task** (an action), **Notional Implementation Example** (an illustrative, non-mandatory route), and **Reference** (a pointer to an established practice document). The task identifiers are of the form `PO.1.1`, `PS.3.2`, `PW.1.1`, `RV.2.2` — this pass enumerated the identifiers present in the published table: PO.1.1–PO.5.2, PS.1.1–PS.3.2, PW.1.1–PW.9.2, RV.1.1–RV.3.4, with the v1.1 change log's retired identifiers for the deleted/moved practice and tasks **PW.3, PW.3.1, PW.3.2, PW.4.3 and PW.5.2** (PW.3 merged into PW.4; PW.3.1 moved to PO.1.3; PW.3.2 moved to PW.4.4 per the appendix's "Formerly" annotation — the change-log summary prints PW.4.5, an internal inconsistency in the published document; PW.4.3 moved to PW.1.3; PW.4.5 merged into PW.4.1 and PW.4.4; PW.5.2 demoted to a PW.5.1 example) ✅.

**The tasks that speak directly to a release gate** — all quoted from the published document, all with their identifiers read at NIST:

| SSDF task | The task text (as published) | Why the gate cares |
|---|---|---|
| **PO.1.2 / PO.1.3** | The requirement-definition practice includes the example "Establish and follow processes for handling requirement exception requests, including periodic reviews of all approved exceptions"; and PO.1.3 requires communicating requirements to third parties providing commercial components, including "periodically reviews of all approved exceptions to requirements" | The gate's **exception machinery** has a standards-level anchor — including the periodic review that §8 makes a condition of legitimacy |
| **PO.2.1 / PO.2.3** | PO.2.3's first example is to "Appoint a single leader or leadership team to be responsible for the entire secure software development process, **including being accountable for releasing software to production** and delegating responsibilities as appropriate" | The **accountable release owner** is a named SSDF practice example — the clearest standards-side support for §7's "someone must own the decision" |
| **PS.3.1** | "Securely archive the necessary files and supporting data (e.g., integrity verification information, provenance data) to be retained for each software release" — with the example of storing integrity and provenance data separately from the release files, or signing it | **The record** (§5) and the artefact-identity question (§9) |
| **PS.3.2** | "Collect, safeguard, maintain, and share provenance data for all components of each software release (e.g., in a software bill of materials [SBOM])" — sharing in standards-based formats, making it available to operations and response teams, protecting integrity and letting recipients verify it, and updating it on every component change | **The supply-chain evidence item** (§3.2 item 5) with the verification requirement spelled out |
| **PW.1.1** | "Use forms of risk modeling – such as threat modeling, attack modeling, or attack surface mapping – to help assess the security risk for the software" | **Threat-model currency** as a task, not a custom (§3.2 item 1) |
| **PW.1.2** | "Track and maintain the software's security requirements, risks, and design decisions" — recording the response to each risk including rationales for approved exceptions, keeping the records for audit and maintenance, and periodically re-evaluating approved exceptions | **The exception register and its review** (§8) |
| **PW.6.1** | "Use compiler, interpreter, and build tools that offer features to improve executable security" — including following change-management processes when updating build tools and auditing unexpected changes to them | **Pipeline integrity** (§9) at the toolchain level |
| **RV.1.1 / RV.1.2** | Gather vulnerability information from acquirers, users and public sources and investigate all credible reports; review, analyse and/or test the code to identify or confirm previously undetected vulnerabilities, with the toolchain configured to run automated analysis "on a regular or continuous basis for all supported releases" | **The open-vulnerability position** (§3.2 item 4) as a continuous obligation, not a gate-day scramble |
| **RV.2.1 / RV.2.2** | Analyse each vulnerability to gather enough information about risk to plan its remediation or other risk response; then "Plan and implement risk responses", explicitly including the example "Make a risk-based decision as to whether each vulnerability will be remediated or if the risk will be addressed through other means (e.g., **risk acceptance**, risk transference)", and the example of temporary mitigation where a permanent fix is unavailable | **Risk acceptance is a named SSDF option** — the standards-side warrant for §8's whole subject |
| **RV.3.1–RV.3.4** | Analyse identified vulnerabilities for root causes; analyse root causes over time to identify patterns; review the software for similar vulnerabilities; review and update the SDLC process | **Recurrence and trend metrics** (§12) have a standards anchor |

**What it does not require.** The SSDF does **not** mandate a specific SDLC, tool, language or gate; it explicitly does not prescribe *how* to implement its practices, focusing on outcomes rather than tools. It does **not** define a release-approval step, a sign-off role, an exception expiry period, or a readiness template — every one of those is this guide's design, not NIST's requirement. It is **voluntary for non-federal organisations** (NIST states the publication may be used by nongovernmental organisations on a voluntary basis) and its development was driven by its statutory role under FISMA 2014 with the OMB Circular A-130 consistency note. And it is **not a checklist** — NIST says so in terms.

**The lineage, and the AI extension.** The US federal frame reaches the gate through **Executive Order 14028, *Improving the Nation's Cybersecurity*** (12 May 2021) — from which the SBOM minimum-element work, the secure-software-development guidance and SP 800-218 derive; SP 800-218 contains a mapping table from **EO 14028 Section 4e clauses to the SSDF practices and tasks** ✅ (this pass read the appendix listing and the chapter list). The repository records the same lineage in its supply-chain guides (EO 14028 / OMB M-22-18 / NIST SP 800-218) ✅ via [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) and [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §8. There is also a **community profile for generative AI**: NIST has finalised **SP 800-218A, *Secure Software Development Practices for Generative AI and Dual-Use Foundation Models: An SSDF Community Profile***, which "augments SP 800-218 by adding practices, tasks, recommendations, considerations, notes, and informative references that are specific to AI model development throughout the software development life cycle" ✅ (CSRC SSDF project page). For teams releasing model-bearing artefacts, this is the relevant extension — and it cross-references the repo's [LLM Development Risks & Security](llm_development_risks_security_guide.md) rather than being re-derived here. The **OMB memorandum M-22-18** is cited in the repo's supply-chain guide as the operational reinforcement of the EO's secure-software attestation expectations ✅ cross-reference (its own date and title were not re-read at the issuing body this pass — see §16).

### 4.3 The OWASP Frame — ASVS and SAMM

Two OWASP projects matter to a release gate, and they do different jobs. Getting them the wrong way round is a common error: **SAMM is the maturity model; ASVS is the verification standard.**

**OWASP ASVS (Application Security Verification Standard)** ✅ — read this pass at `owasp.org/www-project-asvs/`. The project page states the latest stable version as **5.0.0**, offers the requirements in CSV and JSON formats for programmatic use, and defines the referencing convention: each requirement has an identifier of the form `chapter.section.requirement`, and because identifiers may change between versions, documents, reports and tools should prefix the version tag, e.g. **`v5.0.0-1.2.5`** — meaning the fifth requirement in the *Injection Prevention* section of the *Encoding and Sanitization* chapter **of v5.0.0**. The page adds that the `v` preceding the version must be lowercase, and that an identifier used without the version element should be assumed to refer to the latest content — which "as the standard grows and changes this becomes problematic", which is why writers should include the version element.

**What it requires, and what it gives the gate.** The ASVS is a **requirements catalogue** organised into chapters — verifiable, testable security requirements at defined levels of assurance — intended to establish "a level of confidence in the security of Web applications" and to normalise the range of rigour in the market for application security verification. For the gate it is the natural source of the **criteria** against which readiness is judged for application-layer controls, and it is what SSDF's own reference column points at for its PW.1.1 risk-modelling task (SP 800-218 lists OWASPASVS sections 1–13 against PW.1.1, and 1.1.3/1.1.4 against PW.1.2) ✅ — a concrete demonstration that the standards-frame pieces are designed to interlock. Note the version discipline: the ASVS v5.0.0 release is now verified at OWASP this pass; the sibling [Security-by-Design](security_by_design_guide.md) guide's ledger carries it as ⚠, and this pass upgrades it to ✅ while leaving the sibling's v4-lineage discussion untouched.

**OWASP SAMM (Software Assurance Maturity Model)** ✅ — read this pass at `owasp.org/www-project-samm/`. The project page classifies SAMM as a **Flagship Project**, type **Standards**, and states the **latest version as 2.2.0**; the model is published at `owaspsamm.org/model/`, with the project's mission to "provide an effective and measurable way for you to analyze and improve your secure development lifecycle", supporting the complete software lifecycle and being "technology and process agnostic", described as "evolutive and risk-driven in nature, as there is no single recipe that works for all organizations".

**What it gives the gate.** SAMM is not a release checklist and does not contain a release gate. It is an **organisational maturity instrument**: it lets an organisation evaluate its existing software-security practices, build a balanced assurance programme in iterations, demonstrate concrete improvements, and define and measure security activities. Its relevance to release readiness is indirect but real — a maturity model is how an organisation can honestly answer "is our gate as strong as our gate claims to be", and it is the natural counterweight to §11's theatre problem, because maturity levels are hard to fake in a specific way: the *evidence* for a level tends to be the practice actually running. Its further relevance here is linguistic: SAMM is what people often mean when they say "we follow OWASP" at a gate discussion, and it is not a verification standard — for verification you need ASVS, and for the weakness taxonomy the sibling guide covers the CWE Top 25 ([Security-by-Design](security_by_design_guide.md) §4.4), which this guide does not re-derive. The release-gate reading of the pair is one line: **ASVS supplies the criteria; SAMM supplies the maturity claim; neither supplies the decision** — the decision is §7's.

### 4.4 The Supply-Chain Frame — SLSA, SPDX and CycloneDX

Three instruments cover the supply-chain evidence item (§3.2 item 5), and they answer different questions: **SLSA** answers *how was it built and can I tell?*; **SPDX and CycloneDX** answer *what is in it?*

**SLSA (Supply-chain Levels for Software Artifacts)** ✅ — read this pass at `slsa.dev`. The site carries a standing notice that **version 1.2 is the current version** of the specification, and the v1.0 levels page describes the structure that matters most for a gate: **SLSA levels are split into *tracks*, each track having its own set of levels measuring a particular aspect of supply-chain security** — the purpose of tracks being "to recognize progress made in one aspect of security without blocking on an unrelated aspect". The **Build track** in v1.0 has four levels:

| Level | Requirement (as published at slsa.dev) | Focus |
|---|---|---|
| **Build L0** | None — L0 represents the lack of SLSA | (n/a) |
| **Build L1** | Provenance showing how the package was built | Mistakes, documentation |
| **Build L2** | Signed provenance, generated by a hosted build platform | Tampering *after* the build |
| **Build L3** | Hardened build platform | Tampering *during* the build |

The v1.0 page is explicit about the change from the previous version: "The previous version of the specification used a single unnamed track, **SLSA 1–4**. For version 1.0 the **Source aspects were removed to focus on the Build track**. A Source track may be added in future versions" ✅. This matters at the gate for two reasons. First, **any readiness criterion written as "SLSA level 3" must say which track** — a v0.1-era "SLSA 3" and a v1.0 "Build L3" are not the same claim, and a criterion that does not name its track is unenforceable. Second, the levels describe **provenance trustworthiness and completeness**, not code quality — the Build L2/L3 requirements are about who signed the provenance, whether the build platform hosts it, and whether the platform prevents runs from influencing one another and keeps the signing material away from user-defined build steps. **What SLSA gives the gate:** an answer to "is this artefact's construction attributable and tamper-resistant?" **What it does not give:** any statement about the artefact's security, or a requirement that any particular level be reached — SLSA levels are descriptive targets, not obligations. Note also that SLSA is not a single-vendor standard (the site is the specification's home; the framework's stewardship sits with the Open Source Security Foundation, per the sibling [Security-by-Design](security_by_design_guide.md) §6.5, which this guide does not re-derive).

**SPDX** ✅ — read this pass at `spdx.dev`. SPDX is described on its own front page as "an open standard capable of representing systems with software components in as SBOMs (Software Bill of Materials) and other AI, data and security references", and the specification is "a freely available international open standard (**ISO/IEC 5962:2021**)". The project's own overview page supplies the version lineage that a gate's SBOM requirements must be pinned to: SPDX 1.0 in August 2011; 2.0 in May 2015; 2.1 in August 2016; **2.2 in May 2020, including SPDX-lite and satisfying the NTIA minimum SBOM element requirements**; **2.2.1 prepared for ISO submission in August 2020**; **ISO/IEC 5962 available in August 2021**; **2.3 published August 2022** to improve interoperability with other formats; **3.0.0 released April 2024**, adding Security, Build, Dataset and AI profiles. SPDX is hosted by the Linux Foundation, with a technical, legal and outreach workstream structure and a formal governance repository.

**CycloneDX** ✅ — read this pass at `cyclonedx.org/specification/overview/`. The specification details page gives **current version 1.7**, **release date 2025-10-21**, developed by **OWASP Foundation** and **Ecma International**, standardised as **ECMA-424**, **published 2025-12-10**, with the technical committee **TC54**; media types `application/vnd.cyclonedx+json` and `application/vnd.cyclonedx+xml` (plus a protobuf media type), conventional file names (`bom.json`, `bom.xml`, `*.cdx.json`, `*.cdx.xml`), and the object model covering BOM metadata, components, services, dependencies, compositions, vulnerabilities and more. Notably, the page records that OWASP recognises `https://cyclonedx.org/bom` as the official predicate type for all CycloneDX bill-of-material varieties, including SBOM, SaaSBOM and HBOM — the in-toto attestation linkage that a provenance-aware gate can consume.

**The gate's reading of all three.** The SBOM is **necessary and insufficient**. Its necessity: without a component inventory, the gate cannot answer a downstream question — when a critical vulnerability is announced in a widely used library next month, *which of our releases are affected?* — and no retrospective answer is possible without the inventory having been captured at the time. Its insufficiency: an SBOM is a *claim about composition* generated by a tool from a build; it can be incomplete (the compositions element exists because completeness is a variable, not a constant), and it says nothing about components being vulnerable or supported. **The gate should therefore require three things and no more**: that an SBOM exists for the release in a named format at a named version; that its completeness is characterised rather than assumed; and that it is *stored with the release* so the retrospective question is answerable. Note the version discipline the repo already carries: [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) references SPDX 3.0 and CycloneDX 1.5; this pass reads CycloneDX's own page as **1.7** — so a gate criterion should pin the format *version*, not the format *name*. The SBOM/SLSA *practice* — where it runs, how it is generated in a COBOL/procedural estate — is [Security-by-Design](security_by_design_guide.md) §6.4–6.5 and the two supply-chain siblings; this guide consumes their output as evidence and does not re-derive the machinery.

### 4.5 The EU Instrument — the Cyber Resilience Act

**Regulation (EU) 2024/2847 — the Cyber Resilience Act (CRA)** ✅ — read this pass at the European Commission's own pages (`digital-strategy.ec.europa.eu`), both the policy page and the Commission services' summary of the legislative text. The Commission's summary states plainly that "the text below summarises the main provisions of **Regulation (EU) 2024/2847**, known as the Cyber Resilience Act (CRA)" and links the Official Journal text. It is a **horizontal regulatory framework** applying to hardware and software products ("products with digital elements") made available on the Union market, including both final products and components placed separately on the market.

**The dates, as the Commission states them** ✅ — and these are the ones a release-readiness frame must get right, because they are the frame's own clocks:

| Milestone | Date (as stated by the Commission) |
|---|---|
| Entry into force | **10 December 2024** |
| Chapter IV (notification of conformity assessment bodies) applies from | **11 June 2026** |
| Article 14 reporting obligations apply from | **11 September 2026** |
| Full application | **11 December 2027** |

**What it requires, and what it does not.** The CRA imposes **mandatory cybersecurity requirements on manufacturers covering the planning, design, development and maintenance of products with digital elements** — obligations the Commission describes as having to be met "at every stage of the value chain" — and requires manufacturers to "handle vulnerabilities during the lifecycle of their products". Products bear **CE marking** to indicate compliance, with national market surveillance authorities enforcing. For products of particular relevance to cybersecurity, a **third-party assessment by a notified body** may be required before sale on the EU market; the Commission's summary sets out the conformity-assessment routes available (the internal control procedure based on module A; the EU-type examination procedure based on module B followed by conformity to EU-type based on internal production control, module C; and conformity assessment based on full quality assurance, module H, per Annex VIII), with the manufacturer choosing or having carried out the applicable procedure **before placing a product on the market** (Article 32; Annex VIII). The Commission published practical guidance for manufacturers, developers and businesses on **27 July 2026**.

**What it does not do.** The CRA does **not** define a release gate, does not require a security sign-off artefact by that name, and does not prescribe an SBOM format. It regulates **products placed on the market** — a manufacturer-side obligation about products with digital elements — not the internal promotion of an application release inside a bank's own estate. Two honest consequences for this guide. First, the CRA's release-relevant force is on **manufacturers of products with digital elements**, so for an institution whose releases *are* products placed on the EU market (a bank's own software sold as a product, or its role as a component manufacturer for another's product) the CRA is a direct driver of the design/development/maintenance evidence; for an institution shipping internal releases only, the CRA is a **supplier-facing** instrument whose practical effect at the gate is on the vendor evidence (§10.4) rather than on the institution's own readiness criteria. Second, the **reading caveat**: this pass read the CRA at the European Commission's own pages, not at EUR-Lex directly — the EUR-Lex ELI and legal-content URLs returned no renderable content to this host (see §16). The Commission's pages are an official EU source and state the Regulation number and dates above; a reader pinning a control to a specific Article should take the citation from the Official Journal text via the link the Commission provides. Treat the **Article-level** claims here as ⚠ unless re-read at EUR-Lex.

**Repository relationship.** The repo's existing CRA content is a passing supply-chain cite in the two SBOM guides and [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §8 ✅ cross-reference; the EU regulatory container for financial entities is [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) (DORA — Regulation (EU) 2022/2554, applied 17 January 2025 ✅ via the repo's verified cross-reference). The CRA and DORA are distinct instruments with distinct scopes; a release-readiness frame should not merge them.

### 4.6 The Card-Industry Standard — PCI DSS

**PCI DSS v4.0.1** ✅ — verified this pass as the current PCI DSS standard **at the PCI Security Standards Council's own document library** (`pcisecuritystandards.org/document_library/`), where it is listed among the featured documents alongside the *PCI DSS Summary of Changes v4.0 to v4.0.1*. The Council's own standard page describes PCI DSS as developed "to encourage and enhance payment card account data security and facilitate the broad adoption of consistent data security measures globally", providing "a baseline of technical and operational requirements designed to protect payment account data", with the intended audience being entities that "store, process, or transmit cardholder data (CHD) and/or sensitive authentication data (SAD) or could impact the security of the cardholder data environment (CDE)". The Council's own resource hub additionally lists the v4.x supporting instruments a readiness process will encounter — *Scoping and Segmentation Guidance for Modern Network Architectures*, the *Targeted Risk Analysis Guidance*, the *Extra Compensating Controls Worksheet*, the *Prioritized Approach*, the ROC template and the SAQs. The same resource list confirms the Council's separate **Software Security Framework** stream through its assessor/vendor categories (**Secure SLC Qualified Software Vendor**, **Software Security Framework Assessor**) ✅ — the card industry's own secure-software-lifecycle qualification track, distinct from the DSS itself.

**What it relates to the gate — and an honest limit.** PCI DSS is the card industry's baseline of technical and operational requirements, and its secure-development, change-control and testing requirements are the ones a payment-scope release-readiness process will most often be asked about. **The limit is explicit:** this pass could **not** retrieve the PCI DSS v4.0.1 standard text — the Council's document-hosting endpoints returned no renderable content to this tooling — and therefore **no PCI DSS requirement number, requirement title or sub-requirement is asserted anywhere in this guide.** The widely circulated notion of a secure-development "Requirement 6" family is *reported to the reader as unverified* rather than cited: a requirement number the guide has not read at the issuing body is exactly the fabrication §4.1 rule 1 exists to prevent. What the gate designer should do instead is take the requirement numbers, titles and applicability straight from the current standard PDF via the Council's own document library, and pin them to **v4.0.1** — because the Council documents its own version transitions (the v4.0 → v4.0.1 summary of changes is itself a Council publication), and a readiness criterion written against an unversioned "PCI DSS" drifts the moment the Council revises the standard. The Council's **Targeted Risk Analysis** guidance is the closest thing in its own estate to the "risk-based, documented decision" shape this guide's §8 recommends — but that is a structural observation from the Council's own document titles, not a citation of requirement text.

### 4.7 The Verified Standards Table — Requirement / What It Does Not Require / Verified At / Date

| Standard / instrument | Version read this pass | What it actually requires (at the issuing body) | What it does **not** require | `Verified at` | `Date` | Badge |
|---|---|---|---|---|---|---|
| **NIST SP 800-218 / SSDF** | **1.1** (36 pp., doi 10.6028/NIST.SP.800-218) | Practices/tasks in four groups (PO, PS, PW, RV); the tasks and examples listed in §4.2 — including PO.2.3's accountable release owner, PS.3.1 archive, PS.3.2 SBOM/provenance, PW.1.1 risk modelling, PW.1.2 tracked exceptions, RV.2.2 risk acceptance | A specific SDLC, tool or gate; a release-approval step; an exception expiry; a readiness template; it is not a checklist and is voluntary for non-federal organisations | `csrc.nist.gov/pubs/sp/800/218/final` and `csrc.nist.gov/projects/ssdf`; full text at `nvlpubs.nist.gov` | Published **February 2022** (final posted 3 Feb 2022; supersedes CSWP 13, April 2020) | ✅ |
| **NIST SP 800-218A** (SSDF community profile for GenAI) | finalised; version not read in detail | Adds practices, tasks, recommendations, considerations, notes and references specific to AI model development across the SDLC, augmenting SP 800-218 | A stand-alone release gate; a replacement for SP 800-218 | `csrc.nist.gov/projects/ssdf` (project page announcement) | Not dated on the page read this pass — ⚠ date unverified | ⚠ (date) |
| **OWASP ASVS** | **5.0.0** (latest stable) | A catalogue of verifiable application-security requirements by chapter, with levels of assurance; the `v<version>-<chapter>.<section>.<requirement>` referencing convention | A release checklist; a maturity model; an obligation to reach a named level; it is a verification standard | `owasp.org/www-project-asvs/` | Version 5.0.0 stated as latest stable on the project page read this pass; page undated | ✅ (version) |
| **OWASP SAMM** | **2.2.0** (latest) | A framework to evaluate existing software-security practices, build a balanced assurance programme, demonstrate improvements and measure activities; technology- and process-agnostic | A verification standard; a release gate; specific technical controls | `owasp.org/www-project-samm/` (flagship, type: Standards) | Version 2.2.0 stated on the project page read this pass; page undated | ✅ (version) |
| **SLSA** | **v1.0 levels structure** read; site states **v1.2 is current** | Build track levels L0–L3: L1 provenance exists; L2 signed provenance from a hosted build platform; L3 hardened build platform | Any artefact-quality requirement; any obligation to reach a level; a single unnamed "SLSA 1–4" scale after v1.0 (Source aspects removed) | `slsa.dev/spec/v1.0/levels`; `slsa.dev` current-version notice | v1.0 spec structure read this pass; v1.2 stated as current on the site; no date read for v1.2 — ⚠ | ✅ (structure) / ⚠ (v1.2 date) |
| **SPDX** | **3.0.0** released April 2024 (ISO/IEC 5962:2021 for the standardised line) | An open standard for communicating SBOM information — provenance, licence and security data — machine- and human-readable; 2.2 (May 2020) satisfies the NTIA minimum SBOM elements | A completeness guarantee; a security assessment of components; a mandated SBOM format for any regulation cited here | `spdx.dev` and `spdx.dev/about/overview/` | ISO/IEC 5962 available **August 2021**; 3.0.0 released **April 2024** | ✅ |
| **CycloneDX** | **1.7** (release date 2025-10-21) | A modular SBOM/supply-chain object model (components, services, dependencies, compositions, vulnerabilities, and more); standardised as **ECMA-424** (published 2025-12-10); **TC54** | A completeness guarantee; a security verdict; that any tool can produce a full inventory | `cyclonedx.org/specification/overview/` | Release **2025-10-21**; ECMA-424 published **2025-12-10** | ✅ |
| **EU Cyber Resilience Act** | **Regulation (EU) 2024/2847** | Mandatory cybersecurity requirements for manufacturers covering planning, design, development and maintenance of products with digital elements, at every stage of the value chain; vulnerability handling across the lifecycle; CE marking; possibly third-party assessment by a notified body; conformity assessment before placing on the market | A release gate inside a bank; an SBOM format; a named sign-off artefact; obligations on entities that are not manufacturers of products with digital elements placed on the EU market | European Commission pages at `digital-strategy.ec.europa.eu` (policy page + legislative summary) — **not** EUR-Lex, which did not render this pass | In force **10 December 2024**; Chapter IV from **11 June 2026**; Art. 14 reporting from **11 September 2026**; full application **11 December 2027** | ✅ (number, dates) / ⚠ (Article-level text) |
| **PCI DSS** | **v4.0.1** (current per the Council's document library) | A baseline of technical and operational requirements to protect payment account data, for entities that store, process or transmit CHD/SAD or could impact the CDE | **Requirement numbers are not asserted in this guide** — the standard text could not be retrieved at the issuing body this pass; no PCI DSS clause is cited anywhere here | `pcisecuritystandards.org/document_library/` (and the Council's PCI DSS standard page and resource hub) | v4.0.1 verified as the listed current standard this pass; publication date of v4.0.1 **not read** — ⚠ | ✅ (version) / ⚠ (requirement numbers, v4.0.1 date) |
| **DORA** (context) | **Regulation (EU) 2022/2554** | ICT risk management, incident reporting, resilience testing and ICT third-party risk for EU financial entities | Owned by [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) — not re-derived here | Repo's verified cross-reference (EIOPA's DORA page, per the repo's ledger) | Applied **17 January 2025** | ✅ (cross-reference) |
| **MAS instruments** (context) | TRMG; **FSM-N05** (banks); FSM-N22 (cyber hygiene) | Technology-risk expectations on reliability, availability and recoverability of critical IT systems and IT controls protecting customer information | Owned by [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §5 and the repo's MAS guide — not re-derived here | Repo's verified mas.gov.sg cross-references | TRMG published **18 January 2021**; sectoral TRM notices cancelled with effect from **10 May 2024** | ✅ (cross-reference) |

---

## 5. The Record and the Audit Trail

### 5.1 Evidence You Cannot Produce on Demand Is Not Evidence

The SSDF's definition of an *artifact* — "a piece of evidence" — and its instruction to "securely archive the necessary files and supporting data … to be retained for each software release" (PS.3.1) ✅ point at the same operational truth: **evidence is a retrieval property, not a content property.**

A finding that was investigated and correctly resolved, but whose resolution cannot be produced when asked, is indistinguishable from a finding that was ignored — to an auditor, to a supervisor, and to the incident responder at 03:00 six months later. The differentiation between "we checked and it was fine" and "we never looked" exists *only* in the record. This has three consequences for the gate:

1. **The record must be created at the decision, not after it.** A record assembled retroactively is a reconstruction, and reconstructions carry the biases of the people reconstructing them. This is the mechanism behind §11's "the evidence is assembled after the decision".
2. **The record must live where it can be retrieved by someone who was not in the room, without asking the people who were.** If the only route to the evidence is an email thread involving a person who has left, the evidence does not exist in any operationally meaningful sense.
3. **The record must survive the release.** A readiness record tied to a deployment ticket that is closed and archived on completion is a record that has been stored by being lost; retention is part of the artefact's definition, and §4.2's PS.3.1 example — keeping integrity and provenance data in a separate location from the release files, or signing it — is the standards-side version of the same instinct.

### 5.2 What a Release-Readiness Record Should Contain

The record is the gate's *output artefact*, and its content is the evidence set (§3) plus the decision (§7) plus the obligations (§8). A defensible record has these fields:

| Field | Content | Why it is in the record |
|---|---|---|
| **Release identity** | The release name, the target environment, the artefact identity (build identifier, image digest, configuration version, model version) and the intended deployment window | Without artefact identity the record cannot be tied to a thing; "the payments release" is not an identity |
| **Evidence inventory** | Each evidence item from §3.2, where it is stored, and its date | The *where* is what makes the record auditable; the *date* is what makes it a record of a state at a time |
| **Evidence gaps** | Items that were not available, and the disposition of that gap | The absence of an item is itself a fact the decision rests on; an unrecorded gap looks like an oversight and is treated as one |
| **The decision** | Go / conditional go / no-go, with the conditions and the criteria they were judged against | The decision is the point of the record; everything else is support |
| **The decider** | A named individual, their role, and the scope of what they were authorised to accept | The record's central purpose: to identify who took the risk, months later |
| **The residual risk accepted** | A description of what was knowingly accepted, in plain terms, with the compensating controls (if any) and their owners | Risk accepted but not described cannot be monitored; §8's expiry machinery depends on this field |
| **Exceptions and their expiry** | Every waiver granted, its owner, its rationale, its compensating control and its expiry date | The exception register is the record's most operationally useful field (§8, §12) |
| **Conditions and obligations** | Every "conditional go" condition, with a named owner and a due date | The mechanism by which a conditional go stays honest rather than decaying into an unconditional one |
| **Prior-obligation status** | The status of obligations carried forward from earlier releases | The defence against invisible, accumulating liability (§3.2 item 9) |
| **The rollback/reversibility state** | What the undo path is, and whether it was tested for this release | The only field that matters if the release goes wrong an hour after promotion |
| **Attestations collected** | Third-party or supplier attestations relied on, with their scope and date | The boundary between what the institution verified and what it was told (§10.4) |

### 5.3 Who Signs It, How Long It Is Kept

**Who signs.** The signature belongs to the person who owns the decision (§7), not to the security function as a body and not to the release manager by default. Two practical rules apply. **The signer must be identifiable as an individual, not a role in the abstract** — "Approved by Information Security" is a record of a *function's* involvement, not of a person's acceptance, and where an approval is granted by a committee the record should name the chair or the accountable member, because a committee cannot be asked why. And **the signature must record what was signed** — a signature attached to "the release" rather than to a described residual risk is an unscopeable acceptance, and §7.5 shows why the description is what makes the decision defensible.

**How long it is kept.** This guide does not state retention periods, and that is deliberate and load-bearing: retention is set by the institution's own regulatory and legal obligations, by jurisdiction and by instrument, and by the technology-risk expectations of its supervisor. The repository's verified cross-references for those obligations are [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §5 (MAS technology-risk expectations), [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) (the instrument map) and [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) (the resilience container). The *principle* this guide will state plainly: **the retention period should be derived from the longest period over which any obligation arising from the record could still be enforced, disputed or investigated — and it should be a written decision, not an artefact of the tool's default.** A twelve-month default retention on a release register, chosen because the ticketing system's archive policy says twelve months, is not a retention decision; it is an inherited accident.

### 5.4 The Reconstruction Question

The record's purpose is singular and worth stating as a test:

> **Six months later, can you show why this release was allowed to ship?**

Run it as an exercise against a real release from last quarter and the answers sort into four buckets, in ascending order of quality:

| Answer | What it means | Verdict |
|---|---|---|
| "I don't know" | The record is absent, or unfindable | The gate produced no record — it was a meeting |
| "It shipped, so it must have been approved" | The record is being inferred from the outcome | Circular — the outcome is what needed justifying |
| "Here is the ticket with a green tick from the security team" | The record shows an *approval* but not a *reasoned decision* | Insufficient for risk acceptance; adequate only for trivial releases |
| "Here is the record: these were the criteria, this was the evidence with these gaps, this was the residual risk described in these terms, this person accepted it with these conditions, and here is what happened to the conditions" | The record shows the decision | The target state |

The fourth answer is not bureaucratic weight — it is the difference between an institution that *knows* what risk it is carrying and one that is discovering it in an incident review. And note what the fourth answer does *not* require: it does not require that the decision was correct. A record showing that a release was knowingly shipped with a moderate unpatched finding, accepted with a compensating control and a dated review, is a **better** record than one showing an unexamined green tick — because a decision that was made consciously can be improved, and a decision that was never made cannot.

---

## 6. The Gate Design

### 6.1 Where the Gate Sits and What It Is Made Of

The gate sits at the **promotion boundary** — the point at which the artefacts leave an environment where mistakes are recoverable and enter one where they are not. Concretely, in a modern delivery pipeline, that boundary is usually the transition to a protected environment: the merge to a release branch, the publication of a signed artefact to a release registry, the approval of a production deployment, or the promotion of a release candidate to the production ring.

What the gate is *made of* is four components, and a gate missing any of them is a gate with a hole:

| Component | What it is | What breaks without it |
|---|---|---|
| **Criteria** | The stated, versioned conditions the release must satisfy — drawn from the standards frame (§4), the organisation's own policy, and the release's risk profile | The gate becomes a conversation about opinions on the day |
| **Evidence intake** | Where the artefacts in §3 come from, in what format, and how they are validated as *this release's* evidence rather than a neighbour's | The gate adjudicates stale or mislabelled material, and does not know it |
| **Decision mechanism** | Who or what evaluates the criteria against the evidence — a policy engine, a review, or both (§6.2) | The gate has no output, or the output is an opinion with no owner |
| **Enforcement** | The technical or procedural means by which a "no" actually stops the promotion (§6.3) | The gate is advisory, which is to say it is not a gate |

The fourth component is the one teams under-invest in, and the failure is silent: a gate whose "no" is *rhetorical* — a red mark on a dashboard, a comment on a pull request, a strongly worded email — functions as a warning, and the organisation learns within one release cycle that warnings can be outrun. Here is the design rule in one line:

> **A gate's authority comes from having stopped something. A control that has never had an enforcing effect has never been tested, and a control that has never been tested is a hypothesis.**

This is not a demand for a gate to be obstructive. It is a demand that the gate's *stop* capability be real, exercised, and demonstrably able to halt a promotion — because the moment the organisation discovers that the stop is fictional, the entire gate's signalling value collapses retroactively, including the parts that were working.

### 6.2 Automated Checks vs Judgement

The gate's criteria split into two classes, and the split is one of the most practically useful design decisions available.

| Class | Character | Examples | Design treatment |
|---|---|---|---|
| **Mechanical** | Determinable from the artefact or a system of record, without interpretation | Presence and age of an SBOM; whether a scan ran against this build identity; whether a secret of a known class appears in the artefact; whether all required approvers recorded their approval; whether a named dependency is on the prohibited list | **Automate, and make it block** (§6.3). A mechanical check adjudicated by a human at 17:40 on a Friday is a check performed unreliably and unverifiably |
| **Judgemental** | Requires weighing evidence against context — severity against exposure, findings against disposition, precedent against this change | Whether a moderate unpatched finding is acceptable given the component's reachability; whether a pentest from eleven weeks ago still speaks to this release's delta; whether a compensating control actually compensates; whether the residual risk is within the release's tolerance | **Delegate to a named human with the authority in §7.1.** Automating a judgement call produces a gate that is confidently wrong, or one that is gamed within a quarter (§6.4) |

The classic error runs in both directions. Gates that **automate judgement** produce criteria like "no critical or high findings permitted", which are trivially satisfiable by reclassifying a finding's severity, expanding the suppression list, or buying a scanner with different thresholds — the gate is passed by changing the measurement, not the system. Gates that **leave mechanical checks to humans** produce the signed-without-reading pattern of §11, because a human manually confirming nine mechanical facts in a review meeting will confirm all nine, and will confirm them faster as familiarity grows.

The practical test for which class an item belongs to: **can two competent people, given the same artefact, disagree about whether the criterion is met?** If no, it is mechanical. If yes, it is judgemental, and it needs a person with a name.

### 6.3 Blocking versus Warning — and the Real Definition of a Gate

Terminology in this area is loose enough to be dangerous, so this guide fixes it:

| Term | Definition | Legitimate use |
|---|---|---|
| **Blocking gate** | The promotion *cannot proceed* while the criterion is unsatisfied. Failure requires an override with its own authority, and the override is logged | Mechanical criteria; judgemental criteria whose failure is severe enough that shipping without a recorded decision is unacceptable |
| **Warning** | The promotion proceeds; the failure is recorded, notified and visible | Criteria that are informational, or newly introduced while the false-positive rate is being measured (§6.4), or where the cost of stopping is genuinely disproportionate |
| **Guardrail** | A constraint that shapes how work is done without stopping this promotion — a template, a default, a routing rule | Preventing problems rather than detecting them |

**A guardrail called a gate is the most expensive naming error in the discipline**, because it produces the wrong response to a failure. A warning that fails has done its job if the failure was seen and recorded; a *gate* that fails has not done its job unless the promotion stopped. Organisations that label warnings as gates get the worst of both: the reputational cost of a gate (people believe they will be stopped) with none of the protective effect.

**When each is right.** Blocking is right where the failure indicates the evidence itself is absent or untrustworthy (no SBOM, no scan for this build, unreviewed secrets, missing approver) — a gate cannot adjudicate material it does not have, and proceeding means adjudicating nothing. Warning is right where the criterion is a *judgement about a finding*, because findings vary enormously in real exposure and a hard block on a class of finding converts the gate into a severity-threshold exercise (§6.4).

### 6.4 False Positives — a Gate That Cries Wolf Is Trained Around

**A gate that cries wolf is not ignored by accident; it is trained around deliberately.** Developers and release managers optimise for throughput, and a gate that blocks a meaningful fraction of releases on criteria the organisation does not consider real will be worked around — by suppression lists, by severity reclassification, by pre-emptive waivers, by releasing through a different environment, or by simply not registering the release with the gate. None of these are acts of malice; each is a rational response to a gate that is wrong more often than it is right. When the workaround becomes the normal path, the gate's blocking capability still exists in the negative case — it now stops *nothing that matters* and produces *nothing that is visible*, because the traffic has been routed around it.

Three design consequences follow, and they are the practical core of keeping a gate alive:

1. **Measure the false-positive rate explicitly, and treat a high rate as a gate defect, not a user problem.** §12's metrics include this. A gate blocking a quarter of all releases for reasons the organisation overrides three-quarters of the time is generating noise, and the noise is what destroys it.
2. **Introduce new criteria in warning mode first.** A new criterion's real-world precision is unknown until it runs against real releases. Shipping it straight into blocking mode is how a gate acquires its first reputation.
3. **Distinguish *wrong* from *inconvenient*.** A gate that blocks for a reason that is true but that the organisation has decided not to care about is not producing a false positive — it is producing an *unagreed* criterion. Those must be fixed by changing the criterion deliberately and on the record, never by teaching people to route around it. §11's "the gate blocks the small thing and waves through the large one" is what happens when criteria get calibrated by attrition rather than by decision.

### 6.5 The Break-Glass Path

Every real gate needs a documented path for the case where the promotion must happen despite an unmet criterion — an active incident needing a fix, a regulatory deadline, a critical supplier change. **The design rule is not that the path must be hard; it is that the path must be logged, owned and reviewed.**

A break-glass that is not logged is not a break-glass; it is a gate with an undocumented off-switch, and its off-switch is invisible to everyone who is accountable for the risk. The minimum viable break-glass:

| Requirement | Why |
|---|---|
| **A named human authorises it, at a defined level of seniority** | The risk is transferred to that person (§7.1); an anonymous emergency override transfers it to nobody |
| **A recorded reason, captured at the time** | "Critical incident" is a category; "payment authorisation failure in the card switch affecting live traffic" is a reason. Only the second can be reviewed |
| **The unmet criteria enumerated** | The reviewer six weeks later must know what was waived, not merely that something was |
| **The compensating controls in place for the interval** | Shipping without the gate's normal assurance means something must substitute (§8.4) |
| **A mandatory, dated follow-up** | The break-glass's most common decay is that the release stays shipped and the criteria stay unmet, indefinitely and silently |
| **Its own metric** | The break-glass *rate* is one of the most diagnostic numbers an organisation can watch (§12); a rising rate is a statement about the gate, not about the emergencies |

One discipline sits alongside the break-glass: **the break-glass authoriser should be a different person from the standing gate decider where the organisation's size permits**, so that the emergency path cannot be used to accumulate unreviewed risk within a single role.

### 6.6 The Design Rule, Restated

Write these three properties as acceptance criteria for the gate itself: **it can stop a promotion** — not in policy but in mechanism, and it has done so; **its "no" is answerable** — there is a named person behind it who can explain the criterion and the evidence; and **its "yes" transfers risk** — the decision record identifies who accepted what (§5, §7). A gate that satisfies all three, even a simple one, is doing the work. A gate that satisfies none of them, however elaborate its dashboard, is §11's subject.

---

## 7. The Decision and the Sign-Off

### 7.1 Who Decides and What They Own

The decision belongs to a **named individual with the authority to accept the residual risk being transferred to them**. That sentence contains three requirements, each of which is frequently violated:

- **Named individual.** Not a team, not a committee, not "the platform". Committees can be consulted; a committee cannot be asked why. Where a committee decision is unavoidable, the record must name the chair or the accountable member, and the accountability must be theirs (§5.3).
- **With the authority.** The authority must match the *size* of the risk. A release-readiness process in which a junior engineer signs off the same class of decision as the CISO produces an inverted risk gradient: the largest releases get the least scrutiny, because the senior decision-makers are busy.
- **To accept the residual risk.** Not to *approve the release* — to *accept the risk the release carries*. The distinction is the whole point of §2.4 and it survives into the record: an approval is a permission, an acceptance is a liability.

A practical, tiered assignment that organisations converge on — stated here as **this guide's recommendation**, not as any standard's requirement:

| Release risk tier | Typical decider (illustrative) | Rationale |
|---|---|---|
| Routine, low-blast-radius, all mechanical criteria met | The delivery lead, with the security function's criteria embedded in the automated gate | The judgement content is minimal; the mechanical criteria carry the assurance |
| Material change, judgement criteria engaged, no severe findings | The security function's designated reviewer **and** the application owner | The reviewer adjudicates the evidence; the owner owns the system's risk |
| Severe open findings, novel attack surface, external exposure, or a break-glass | An accountable business or technology owner at a defined seniority, informed by the security function | The risk is larger than the security function's mandate to absorb |
| Regulatory-scope or customer-impacting with known unmet criteria | The accountable executive owner, with the exception recorded under §8 | The transfer is institutional, so the acceptance must be institutional |

**On the independence question** (developed in §10.6): where the same individual or team is expected to build the control, verify the control and sign off on the control, the decision is structurally weak regardless of how competent the person is. That is not a comment on anyone's integrity; it is a comment on the impossibility of being surprised by one's own work.

### 7.2 Go, Conditional Go, No-Go

Three outcomes, and the middle one is where the discipline lives.

| Outcome | Meaning | The discipline it requires |
|---|---|---|
| **Go** | The residual risk is acceptable as it stands; the release ships on the accepted evidence | The record must still describe the residual risk (§5.2) — "go" is a decision, not an absence of one |
| **Conditional go** | The release ships, with named conditions and a named owner for each, and a date by which each must be satisfied | Every condition needs **an owner and a date**. A condition without both is a go with a footnote, and footnotes are not monitored |
| **No-go** | The release does not ship until a stated criterion is met | The *stated criterion* is the gift: a no-go that does not say what would change its mind is an obstructed release, not a decision, and it converts the gate into an adversary |

The most important structural warning here: **"go with actions" is usually a no-go in disguise or a go with no accountability, and the organisation should decide which.** If the actions are conditions that will genuinely be tracked with owners and dates, it is §7.2's conditional go and should be recorded as such. If the actions are a list of nice-to-haves appended to get the release out, the release is a go and the actions are decoration. Labeling it "conditional" gives the decision-maker the comfort of a no-go and the speed of a go, which is the combination that produces untracked risk.

### 7.3 The Signer Must Be Able to Say No

**A signature from someone who cannot refuse does not transfer risk.** This is a design property, not an ethical one, and it is testable: ask of any sign-off role in the process, *what happens in this organisation when this person says no?* The answers sort the roles:

| Answer | What the role actually is |
|---|---|
| The release does not ship, and the reason is recorded | A real gate decider — the signature transfers risk |
| The release ships anyway, and the signature is a formality | An administrative step — the signature transfers nothing, and the record is misleading |
| The release ships anyway, and the person's objection is recorded but not actioned | A documented dissent — valuable, but it is not a gate; calling it one hides an unwatched risk |
| Saying no requires escalation to someone who overrules as a matter of course | A role with negative authority: it creates friction without creating a decision |

The practical implication is uncomfortable for organisations that locate the sign-off in the security function while locating the schedule pressure in the delivery function: **if the security function can be overruled by the delivery function on every occasion, then the delivery function holds the gate, and the record says otherwise.** That mismatch — the recorded decider is not the effective decider — is one of the highest-impact structural problems a readiness process can have, because every subsequent audit of the gate will be an audit of a fiction.

### 7.4 Advising versus Owning Residual Risk

This distinction is drawn precisely and with the honest statement that **regulators care about it and practice blurs it**:

| Stance | What the security function does | Who carries the consequence |
|---|---|---|
| **Advising** | Assesses the release, states the findings and the risk in plain terms, recommends | The business or technology owner who accepts the recommendation — including when the recommendation is overruled |
| **Owning** | Accepts the residual risk on behalf of the institution | The security function, and by extension whoever holds its mandate — which is a governance question the function usually cannot answer alone |

The two are legitimate models when they are **named**. What is illegitimate is the hybrid that is common in practice: the security function is described as an *adviser* when it wants to avoid accountability for a business decision, and described as an *owner* when the organisation needs to demonstrate that security has control. That hybrid is **not a compliance defect on this guide's part to diagnose** — this guide does not give compliance advice, and each institution's instrument set determines where the control must sit. It is stated here as a design problem: a decision process in which the accountable party changes according to the outcome cannot be reviewed, cannot be improved, and cannot be defended. The governance question — which model the institution uses, and whether the security function's mandate matches — belongs with [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) and the institution's own supervisory frame ([Operational Resilience Framework](../banking/operational_resilience_framework_guide.md), and the MAS technology-risk cross-reference in [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §5).

There is one clean formulation worth writing into a gate charter: **the security function's job at the gate is to make the risk legible; the accountable owner's job is to decide.** Legibility means the risk is described in terms a non-security executive can act on — what could happen, to what, how likely, how bad, and what would change the assessment. A security function that makes the risk legible and then is overruled has done its job. A security function that compresses the risk into "12 findings, 2 critical" and calls it a recommendation has not.

### 7.5 Recording a Decision So It Can Be Defended

The record from §5.2 is produced here, and the quality of the record determines whether the decision can be defended. Three properties make a decision defensible:

1. **It states the criteria it was judged against.** "Approved" against unstated criteria is unappealable and unreviewable. "Approved against criteria A–F, versions X–Y, with C1 waived under exception E-4471" is both.
2. **It states what was known and what was not.** A decision that records its evidence *gaps* is stronger, not weaker — a gap that is disclosed and accepted is a managed unknown; a gap that is silently filled by assumption becomes the thing the post-incident review finds.
3. **It states what would have changed the decision.** This is the strongest single addition to a readiness record, and the least commonly made. A decision-maker who can say "we would have gone no-go if the finding had been reachable from the internet; it is not, per this assessment" has recorded *why the risk was acceptable*, not merely that it was accepted — which is precisely what makes the decision reviewable, improvable and defensible six months later.

---

## 8. Risk Acceptance and Exceptions

This is the guide's most valuable section, and it is written with unusual care because it is the section where an organisation's stated practice and its actual practice diverge most often — and where the divergence is invisible until it is expensive.

### 8.1 Who Is Empowered to Accept a Risk

Risk acceptance is a **governance grant**, and like any grant it must be explicit, bounded and revocable. Four questions define it:

1. **Which risks?** The grant must name the *class* of risk the person may accept — a moderate unpatched dependency in an internal system; a deferred penetration test on a low-exposure service — not "security risk" generically.
2. **Up to what extent?** A monetary or severity ceiling, or a scope ceiling (systems, data classifications, customer impact). An unbounded grant is not a grant; it is a delegation of the whole mandate.
3. **For how long?** Every acceptance is time-bounded by construction (§8.3).
4. **With what review?** Who sees the accepted risk, and at what cadence. An acceptance register that nobody reviews is a ledger of decisions nobody is monitoring.

The SSDF lends this a standards-side anchor in two places. **PW.1.2** requires tracking the software's security requirements, risks and design decisions, with the example of recording "the response to each risk, including how mitigations are to be achieved and what the rationales are for any approved exceptions to the security requirements", maintaining records of approved exceptions "that can be used for auditing and maintenance purposes", and periodically re-evaluating them ✅. And **RV.2.2** makes risk acceptance an explicit, legitimate risk-response option — "Make a risk-based decision as to whether each vulnerability will be remediated or if the risk will be addressed through other means (e.g., risk acceptance, risk transference)" ✅. The SSDF also anticipates the second-order problem: where a permanent mitigation is not yet available, the task is to determine how the vulnerability can be temporarily mitigated until the permanent solution is available, and to add that temporary remediation to the plan ✅ — the temporary-mitigation discipline that §8.4 generalises.

What the SSDF does not supply, and what this guide therefore states as its own recommendation: the **empowerment structure** (§8.1's four questions), the **expiry norm** (§8.3), and the **compensating-control patterns** (§8.4) are all practice design, not standards text. An organisation adopting the SSDF has adopted the *concept* of approved exceptions and their periodic review; it has not adopted a template.

### 8.2 What a Waiver Must Carry

**A waiver is an exception to a stated requirement or gate criterion, granted for a defined period, with a defined owner.** The minimum content — and this guide's recommendation, stated as a quadriga because the four elements are mutually load-bearing:

| Element | What it is | What its absence produces |
|---|---|---|
| **An owner** | A named individual accountable for the exception and for its resolution | The exception becomes everyone's and therefore no one's; the follow-up does not happen |
| **A rationale** | Why the requirement cannot be met now, in specific terms — not "business pressure" but "the vendor's patched release is scheduled for the next maintenance window on <date>" | Reviewers cannot distinguish a justified exception from an unexamined one, so they stop reviewing; the register becomes noise |
| **A compensating control** | The different control that addresses the same risk in the interim (§8.4), or an explicit statement that none is available and the risk is accepted bare | The exception is a silent removal of a control with nothing substituted — the risk is accepted by default rather than by decision |
| **An expiry** | A date on which the exception lapses and the requirement either is met or the exception is re-decided | The exception becomes permanent, which (§8.3) makes it not an exception |

Two further fields make the quadriga auditable: **the requirement being waived**, referenced precisely enough that a reader can find it (an SSDF task identifier such as `RV.2.2`, an ASVS requirement cited as `v5.0.0-<chapter>.<section>.<requirement>` per §4.3, or an internal criterion's version), and **the evidence that the compensating control works** — because a compensating control with no evidence is a compensating *assertion*.

**The design of the register.** Exceptions should live in one register with one schema, visible to the gate (so §3.2 item 9's prior-exception disposition is available at the next release), visible to the risk function, and reviewed on a cadence with the expiry dates as the sort key. The single most diagnostic field in the entire register is the **expiry date**, because sorting by it produces the list of decisions that are about to become undecided.

### 8.3 A Permanent Waiver Is a Contradiction in Terms

The sentence deserves its own heading because it is the most common self-deception in the discipline:

> **An exception with no expiry is not an exception. It is a silent, permanent amendment to the standard — made by whoever had the authority to sign the waiver, not by whoever had the authority to change the standard.**

The mechanism is worth tracing, because it is rational at every step and wrong at the end. A requirement cannot be met this release; the team is busy; the waiver is granted to unblock; the follow-up is handed to a busy owner; the release ships; nothing breaks; the next release arrives and the waiver is *still technically open*, so the requirement is *still technically waived*, so no one re-decides. Two years later the requirement is not met and no one has decided not to meet it. The organisation has a control it believes in, an exposure it does not see, and a register entry that gives both a clean appearance.

Three consequences drive the design:

1. **Expiry is what makes a waiver a decision rather than a deletion.** The date forces the re-decision; the re-decision is what keeps the acceptance conscious.
2. **Expiry must be short enough to force a decision and long enough to be achievable.** An expiry set beyond the plausible remediation horizon is a permanent waiver wearing a date. As a rule of thumb — this guide's recommendation, not a standard — an expiry should be within the span in which the compensating control can be maintained and the remediation realistically scheduled, and where a remediation has no credible date, the correct output is not a long-dated waiver but a **decision to change the requirement**, taken by whoever owns the standard and recorded as a change.
3. **Lapsing must have a consequence.** An expiry that passes silently is not an expiry. The register must drive a notification, the gate must show the lapsed exception, and the release that depends on it must be blocked or re-decided. When there is no consequence, the expiry column becomes a decorative date field — which is §11's "the waiver has no expiry" in its more sophisticated form.

### 8.4 Compensating-Control Patterns

A compensating control is **a different control that addresses the same risk by another route**. The test is not whether it is well-intentioned but whether it actually reduces the risk the primary control was addressing. For a release shipped without a normal control, three patterns cover most real cases — stated as **practice, not prescription**:

**Pattern 1 — the compensating detective control.** The primary control (preventive) is absent; substitute a *detection* capability that would catch the failure quickly. The patch cannot be applied before the window; the compensating control is enhanced monitoring and alerting on the specific exploit signature, with a named responder. Full disclosure of the reasoning: a detective control changes the risk's *shape*, not its existence — it converts an unbounded, undetected exposure into a bounded, detected one. That is a genuine reduction, and it is also clearly not equivalent to the patch. Both facts belong in the exception's rationale.

**Pattern 2 — reduced exposure.** Narrow the risk's reach while the gap is open: restrict the affected function behind an access control, move the component out of the internet-reachable path, disable the feature, reduce the population of accounts or customers on the affected route, shorten the window in which the affected system runs. This is the pattern with the strongest risk-reduction claim, because it reduces the *likelihood × impact* product rather than only the detection latency. It has an honest cost: reduced functionality is a business decision, and it must be taken by someone who owns the affected function, not by the security function alone.

**Pattern 3 — time-boxed acceptance with a hardware commitment.** Where neither detection nor exposure reduction is available, the honest residue is: **we are carrying this risk, for this long, because we choose to — and here is the paid-for commitment that ends it.** This pattern's integrity rests entirely on the commitment being real: a scheduled, funded, resourced remediation with a date, not an intention. Where the "commitment" is a backlog item with no owner, Pattern 3 degenerates into §8.3's permanent waiver and should be recorded as such, because at least then the organisation can see it.

**Anti-patterns in this space**, named so they are recognisable in the wild: the compensating control that is a *policy statement* ("users are instructed not to…"); the control that is a *restatement of the risk* ("the exposure is monitored by the security team"); the control that is *the thing that failed* (the primary control, renamed); and the control with *no instrument* (nobody can say whether it is working). Each of these is a comfort blanket rather than a control, and each is recognisable by the same test: **does it reduce the likelihood or the impact, and can you tell whether it is operating?**

### 8.5 The Exception as a Loan Against the Future

The honest framing, and the one to write into the register's own documentation:

> **An exception is a loan against the future, repaid with interest in remediation.** The principal is the control you did not apply; the interest is the compounding cost of applying it later — under time pressure, after a near-miss, or during an incident, when the cost is orders of magnitude higher than it would have been at the gate.

Three consequences of taking the loan metaphor seriously:

- **The interest is real and predictable.** Remediation costs rise when the context in which the fix must be applied has moved on: the engineer who wrote it has changed team, the version has drifted, the test coverage around it has decayed, the release has become a dependency of three other releases. This is exactly the dynamic the SSDF's shifting-left principle names — addressing security earlier in the SDLC "less effort and cost is ultimately required to achieve the same level of security", and shifting left "minimizes any technical debt that would require remediating early security flaws late in development or **after the software is in production**" ✅ (SP 800-218, §1). The exception is the moment the interest starts accruing.
- **Borrowing is legitimate; borrowing without a repayment schedule is not.** An organisation that grants exceptions *and* retires them on schedule is managing risk. An organisation that grants exceptions and never retires them is amortising a liability it has stopped counting.
- **The aggregate is the number that matters, not the individual loan.** Any single exception is defensible. The property to watch is the *stock* — how many open, how old is the oldest, what fraction recur (§12). The aggregate is what determines whether the organisation has a gate with exceptions or an exception with a gate.

### 8.6 Requirement, Expectation, Recommendation — Kept Apart, Item by Item

The three registers must not be blended, because they have different owners, different enforcement and different consequences. **A standard's requirement** is text in a published standard and is met or not met; **a regulator's expectation** is a supervisory expectation derived from the institution's instrument set — it is not advice from this guide, and the governing instrument text is what applies; **this guide's recommendation** is practice design, and an institution may reasonably do otherwise.

| Item | A standard's requirement | A regulator's expectation | This guide's recommendation |
|---|---|---|---|
| **Exception register** | ✅ SSDF PW.1.2: maintain records of design decisions, risk responses and **approved exceptions** usable for auditing and maintenance; and re-evaluate approved exceptions periodically (SP 800-218) | ⚠ — the institution's supervisor expects technology risk to be visible and governed; the *register* is a control by which that visibility is demonstrated (*this guide's reading*; the instrument text governs) | One register, one schema, visible to the gate and the risk function (§8.2) |
| **Exception expiry** | Not required — the SSDF requires *periodic re-evaluation*, not a dated expiry ✅ (absence is the finding) | ⚠-structural | **A dated expiry, with a consequence for lapse** (§8.3) |
| **Compensating control** | ⚠ Not named as such in the SSDF tasks read this pass; RV.2.2's *temporary mitigation* example is the nearest published language ✅ | ⚠-structural | The three patterns, with an evidence requirement for each (§8.4) |
| **Risk acceptance as a legitimate response** | ✅ SSDF RV.2.2 names risk acceptance and risk transference as risk-response options, to be chosen by a **risk-based decision** (SP 800-218) | ⚠ — supervisors expect accountability for accepted risk; this guide gives no compliance advice | Acceptance must be an explicit decision with a named accepter (§7.1, §8.1) |
| **Named individual signer** | ✅ SSDF PO.2.3 example: appoint a single leader or leadership team "accountable for releasing software to production" (SP 800-218) | ⚠-structural | The signer is a named individual; committee approvals name the accountable member (§5.3, §7.1) |
| **Threat-model currency at release** | ✅ SSDF PW.1.1: use risk modelling (threat modelling, attack modelling, attack surface mapping) to assess software security risk | ⚠-structural | Currency *for this release's delta*, not mere existence (§3.2 item 1) |
| **SBOM at release** | ✅ SSDF PS.3.2: collect, safeguard, maintain and share provenance data for all components of each release, e.g. in an SBOM, in standards-based formats, with integrity protection and recipient verification | ⚠-structural (supply-chain expectations vary by instrument and jurisdiction) | Require existence, a named format **version**, characterised completeness, and storage with the release (§4.4) |
| **Release-record retention period** | Not specified by the standards read this pass — ⚠ | Determined by the institution's obligations and its supervisor's expectations; **not stated by this guide** (§5.3) | Derive from the longest period over which an obligation could be enforced, disputed or investigated; make it a written decision, not a tool default |
| **Break-glass logging** | Not specified — ⚠ | ⚠-structural | Named authoriser, recorded reason, enumerated waivers, compensating controls, dated follow-up, its own metric (§6.5) |

Read the table's shape: the **left column** is thin (the standards are about practices and outcomes, not about gate mechanics), the **middle column is deliberately mostly ⚠** (this guide is not a compliance source and will not invent supervisory expectations), and the **right column is where the guide earns its keep** (practice design, clearly owned as the guide's own). A reader who needs the middle column filled from authority should go to the instruments themselves — [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md), [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md), [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §5 — and not to this guide.

---

## 9. The Gate in the Delivery Pipeline

### 9.1 Where the Automated Security Checks Run

The engineering side of the gate is the pipeline, and the pipeline's security machinery is **owned elsewhere in the repository** — the practice, the tooling and the gate mechanics at each stage are [Security-by-Design](security_by_design_guide.md) §6 (CI/CD security gates, secret management, SBOM and SLSA), with the low-code estate's pipeline version in [Power Platform CI/CD](power_platform_cicd_guide.md) and the image-layer specifics in [Dockerfile Best Practices](dockerfile_best_practices_guide.md) and [Docker Image Comparison Tools](docker_image_comparison_tools_guide.md). This guide does not re-derive where to run SAST or how to store secrets. What belongs here is the **readiness question about the pipeline**: *which of the pipeline's checks are evidence for a release decision, and which are hygiene?*

| Pipeline stage | What it produces | Is it release evidence? |
|---|---|---|
| **Commit / pull request** | SAST on the diff, secret scanning, dependency policy checks | **Partially** — the aggregate state matters, not the pass/fail of any one PR. A clean PR history does not mean a clean release artefact |
| **Build** | The artefact; build provenance; SBOM generation; toolchain integrity | **Yes** — this is where the artefact is *created*, so this is where attribution and composition evidence originates (§3.2 items 5 and 7) |
| **Artefact publication** | The signed, stored release artefact; its digest; its stored evidence bundle | **Yes** — the artefact identity the whole record hangs on (§5.2) |
| **Pre-production deployment** | DAST against a running instance, integration tests, configuration validation | **Yes, if and only if** it ran against this artefact in a configuration representative of production |
| **Production admission** | Policy-as-code enforcement, image signature verification, admission control | **Yes** — and this is the last mechanical point at which a promotion can be stopped without human intervention |

The pattern to internalise: **pipeline checks are evidence when they are tied to the release's artefact identity, and hygiene when they are not.** A SAST run on the feature branch three weeks and forty commits before the release candidate is hygiene. The same tool's run against the release candidate's exact commit, with the results stored against the artefact digest, is evidence.

### 9.2 What Blocks a Build versus a Release

The distinction is frequently collapsed, and collapsing it is what makes pipelines both obstructive and ineffective at the same time.

| | Blocking a **build** | Blocking a **release** |
|---|---|---|
| **Purpose** | Stop defective code entering the mainline / the artefact store | Stop an unacceptable residual risk reaching production |
| **Scope** | The change under construction | The named release, judged in aggregate |
| **Cost of the stop** | Low — the developer fixes and reruns, minutes to hours | High — a release schedule, a change window, a business commitment |
| **Who should own it** | Engineering, with criteria expressed as code | The gate decider (§7.1), with criteria expressed as policy and evidence requirements |
| **Characteristic failure** | Over-blocking on the developer's inner loop, which gets the check disabled or bypassed | Under-blocking because the criteria were designed for the inner loop |

The most consequential design error in this area is **inheriting the build gate's criteria as the release gate's criteria**. The build gate's criteria are chosen for *fast feedback on a small change* — a linter's opinion, a coverage threshold, a scanner's finding on a diff. Applied to a release decision, they produce a gate that blocks on the small and misses the large (§11.6), because the things that make a release unready — a stale threat model, a dependency that lost support last month, a pentest that no longer speaks to the current attack surface, an architecture the change invalidated — are not visible to build-time tools at all. [Security-by-Design](security_by_design_guide.md) §6 covers the gate mechanics inside the pipeline; the readiness point is that **a release gate must have at least some criteria that only a release-context evaluation can satisfy**, or it is a build gate with a bigger name.

### 9.3 The Artefact-Provenance Question

The pipeline's most important contribution to the gate is not a finding — it is an **identity**. Before any question about a release's safety can be answered, the organisation must be able to answer a prior question: *is the thing we are about to ship the thing we tested?*

That question has three levels of answer, in ascending strength:

1. **Asserted** — the artefact is labelled with a version, and the version is trusted to refer to what was tested. This is the default state of most organisations and it is weaker than it feels: rebuilds from the same source can differ, artefacts can be replaced in a registry, and tags are mutable.
2. **Attributed** — the artefact carries provenance describing what was built, by what process, from what inputs, and the provenance is distributed with the artefact. This is roughly SLSA Build L1's territory ✅ (slsa.dev, §4.4): enough "to prevent mistakes", but, as the specification itself says of L1, "trivial to bypass or forge".
3. **Verified** — the provenance is signed, generated by a platform with tamper controls, and *checked by the consumer* at admission, so that a mismatch stops the promotion. This is SLSA Build L2–L3 territory, and note the levels' own framing: L2 addresses tampering *after* the build; L3 addresses tampering *during* it ✅.

Two readiness consequences. **First, the gate's other evidence is only as good as this answer.** A pentest result, a scan report and an SBOM are all statements about a *specific artefact*; if the identity chain is asserted rather than verified, the gate is adjudicating evidence about something it cannot confirm is the thing shipping. **Second, verification is a consumer-side act.** Provenance that is generated and never verified provides documentation, not assurance — the SSDF's PS.3.2 asks for provenance to be protected and for recipients to be given "a way for recipients to verify provenance data integrity" ✅, and the second half of that requirement is the half organisations skip. The deep treatment of the tools and the estate specifics is [Security-by-Design](security_by_design_guide.md) §6.4–6.5 and [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md); the reproducibility precondition for believing a build claim at all is [Deterministic Engineering](deterministic_engineering_guide.md).

### 9.4 Shift-Left, and the Last Cheap Moment

There is a genuine tension between two true statements, and this guide resolves it explicitly rather than choosing a slogan. **True statement one: shift left** — the SSDF states the principle plainly, calling it "shifting left" and noting it "is critically important regardless of the SDLC model" and that it "minimizes any technical debt that would require remediating early security flaws late in development or after the software is in production" ✅ (SP 800-218, §1); the entire discipline of [Security-by-Design](security_by_design_guide.md) sits on this. **True statement two: the release gate is the last place a problem can still be stopped cheaply** — an hour before promotion, the cost of stopping is a delay; an hour after promotion, the cost of the same problem is an incident, with detection latency, customer impact, potentially a notification obligation, and a far more expensive fix under pressure.

These are not in competition, because they answer different questions. Shift-left is about **where defects are prevented**; the gate is about **what happens to the defects that were not prevented**. An organisation that reads shift-left as an argument *against* a strong release gate has made a category error — it has assumed the pipeline is perfect and therefore the gate is redundant, at precisely the point in history when the evidence (the whole supply-chain instrument set of §4.4, the existence of SSDF PS.3.1 and PS.3.2, the entire SLSA framework) says the pipeline is not perfect. Conversely, an organisation that reads the gate as a substitute for shift-left has built an expensive late-detection machine and will pay the cost in release delays.

The correct relationship, stated as a division of labour:

> **Shift left to make the gate's job small. Do not weaken the gate because the shift-left work was done.**

And a corollary that matters operationally: **the gate is where the cost of a weak pipeline becomes visible.** A gate that is repeatedly catching problems that should have been caught at design or build time is giving the organisation a diagnostic — not a reason to loosen the gate, but a measurement of how much the earlier stages are actually doing.

---

## 10. The Regulated-Institution Angle

### 10.1 The Change-Management and Change-Approval Frame

A bank's change governance is broader than security, and it must be, because it is answering a different question: *can this change be made safely, in this window, with this dependency ordering, and can it be undone?* That question covers scheduling, impact analysis across the estate, communication, maintenance windows, regulator-facing implications and rollback planning — a great deal that has nothing to do with security, plus a great deal of security that the change process does not itself evaluate.

The correct interface is therefore directional, not substitutive:

| Direction | What flows | What must not happen |
|---|---|---|
| Release gate → change approval | The security verdict: the decision, the residual risk accepted, the conditions, the exceptions, the rollback state | The change record must not *restate* the security verdict as its own; it should reference the readiness record (§5) |
| Change approval → release gate | The deployment envelope: window, blast radius, dependency ordering, freeze state, communications | The change process must not *become* the security gate by default, which is what happens when the security gate is considered optional |

**Two failure modes are worth naming.** The first is **the security gate as a change-management checkbox** — where "security approval" is one field among fifteen, completed by the change raiser on behalf of a security function that never saw the release. The second is **the change advisory board as gate theatre** — a weekly meeting that reviews forty changes in fifty minutes, in which the security content of any individual item cannot possibly be examined. Neither is a claim about any institution's practice; both are structural consequences of a security gate that has not been designed to be a gate (§6).

The repository's change-management frame is spread across [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §7 (governance gates, in the lifecycle-inventory sense) and §12 (its change/asset worked example), with the risk-accountability angle in [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md).

### 10.2 Operational-Resilience Expectations

The release gate sits inside an operational-resilience frame it does not own. **This is entirely the territory of [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md)** — its important-business-service and impact-tolerance machinery, its severe-but-plausible scenario work, its third-party and ICT-risk treatment, its testing regime, and its regulatory landscape including **DORA (Regulation (EU) 2022/2554, applied 17 January 2025)** ✅. This guide cross-references that frame by name and does not re-derive it.

Three of that frame's properties change what a release gate must evidence, and they are worth stating as *readiness consequences* rather than as resilience content:

- **Impact tolerance gives the release decision a yardstick.** Where the institution has stated how much disruption an important business service can tolerate, a release that changes that service's failure characteristics is not a generic change — it is a change to a measured tolerance, and the readiness review is the last place the tolerance can be reasoned about before it is tested in production.
- **The third-party dimension makes vendor evidence part of the release's evidence.** Where a release depends on an ICT third party — a platform change, a shared service, a component sourced from a provider — the readiness gate's supply-chain item (§3.2 item 5) is the point at which the institution's *knowledge* of that dependency is tested. The boundary between what the institution verified and what it was told is exactly the boundary §5.2's attestations field records.
- **The testing regime and the release gate are different instruments.** Resilience testing (including the regulator-mandated forms) exercises the institution's ability to withstand disruption; the release gate decides whether a specific change is acceptable to promote. A release that has passed a resilience test is not thereby release-ready, and a release that is release-ready does not establish that the service meets its impact tolerance.

### 10.3 Supervisor Technology-Risk Expectations

Supervisory technology-risk expectations are held in **verified cross-reference and are not re-derived here**: [Technology Lifecycle Management](technology_lifecycle_management_guide.md) §5 owns the MAS technology-risk angle, and [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) owns the instrument map. What this guide will state, because it is a design consequence rather than a regulatory claim:

- The MAS instruments in the repo's verified ledger include the **Technology Risk Management Guidelines, published 18 January 2021**, and — since the sectoral TRM notices were **cancelled with effect from 10 May 2024** — the harmonised FSMA notices, **FSM-N05** for banks and **FSM-N22** for cyber hygiene ✅ (repo cross-reference; the instrument text governs, and this guide gives no compliance advice). Their subject matter — requirements for a high level of reliability, availability and recoverability of critical IT systems, and IT controls protecting customer information ✅ — is the container in which a release decision sits.
- **The gate is the record-producing mechanism for a class of change.** A supervisory examination that asks "how does the institution assure itself that changes to critical systems do not degrade reliability or control effectiveness?" is asking a question whose answer is either a release-record set (§5) or an assurance narrative. Organisations that have the records answer with evidence; those that do not answer with assurance prose.
- **Nothing in this guide should be read as a statement of what any supervisor requires.** The instruments govern; this guide is a design reference.

### 10.4 The Supply-Chain and Third-Party Evidence Boundary

The gate's supply-chain evidence is where the institution's own assurance ends and a supplier's assertion begins, and **marking that boundary is a readiness activity in its own right.** Three distinct things must not be blended in the record:

1. **Verified by the institution** — the artefact's digest, its SBOM as generated from the artefact the institution holds, the results of scans the institution ran.
2. **Attested by the supplier** — a signed statement that a component was built under certain practices, or that a vulnerability was remediated. The SSDF anticipates exactly this shape in **PO.1.3**, whose examples include requiring third parties to attest that their software complies with the organisation's security requirements, defining security criteria for selecting software including the third party's vulnerability-disclosure programme and incident-response capabilities, requiring third-party provenance data and integrity verification mechanisms, and establishing processes to handle risk where acquired components do not meet requirements, "including periodic reviews of all approved exceptions to requirements" ✅ (SP 800-218). Note the last clause: **supplier-side exceptions get the same periodic-review discipline as internal ones** — the SSDF's exception logic is not scoped to your own code.
3. **Assumed by nobody** — the dependencies for which the institution has neither verified nor been told anything. This class must appear in the record as a disclosed gap (§5.2), because the most dangerous supply-chain state is not a bad attestation; it is an unexamined one.

For an institution with a card-data environment, the card industry's own secure-software-lifecycle qualification track applies to the *suppliers*: the PCI Security Standards Council's own listings include the **Secure SLC Qualified Software Vendor** category and **Software Security Framework Assessor** ✅ (pcisecuritystandards.org) — meaning a purchaser can, in principle, distinguish a supplier that has been qualified under the Council's software-lifecycle programme from one that has not. The repository's vendor-side frame is [Vendor Management](../management/vendor_management_guide.md); the SBOM-specific mechanics are [Security-by-Design](security_by_design_guide.md) §6.4 and [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md).

### 10.5 The Card-Industry Standard's Secure-Development Expectations — and the Reading Limit

PCI DSS v4.0.1 is the current standard ✅ (§4.6). Its secure-development, change-control and testing requirements are the ones a payment-scope release process will be asked about — **and, as §4.6 states plainly, this guide does not cite a single PCI DSS requirement number, because the standard text could not be retrieved at the issuing body this pass.** No requirement number, title or sub-requirement appears anywhere in this guide, and any that a reader has seen elsewhere should be re-read at the Council's own document library before being written into a control.

Two structural observations that are defensible from what *was* read at the Council's own site. **The Council publishes the transition artefacts alongside the standard** — a *Summary of Changes v4.0 to v4.0.1*, a *Targeted Risk Analysis Guidance*, and an *Extra Compensating Controls Worksheet* ✅ — so whatever a bank's process does with compensating controls, the card industry has its own vocabulary and its own worksheet for the concept, and readiness processes in payment scope should use the Council's artefacts rather than inventing parallel ones. And **the Council separates the DSS from its software-lifecycle framework stream** ✅ (§4.6/§10.4): a readiness process that conflates "the DSS applies to my cardholder-data environment" with "my software suppliers are qualified under the Council's software security framework" will misstate its own assurance in both directions.

### 10.6 Independence — Who May Sign

The blunt statement this section exists to make:

> **In a regulated institution, the release gate is a control — so it needs an owner, evidence, and independence.**

Each of the three is a design requirement, and each is separately testable:

- **An owner.** A control with no named owner is a control an examination will find unattributed. §7.1 covers who the decision-maker is; the same logic applies one level up, to whoever owns the *gate* — its criteria, its false-positive rate, its break-glass rate. Note the SSDF's own framing here: the accountable release owner is not an inference from the framework but an example in it — **PO.2.3**'s first example is to appoint a single leader or leadership team "responsible for the entire secure software development process, **including being accountable for releasing software to production**" ✅ (SP 800-218).
- **Evidence.** The gate's operation must itself be evidenced, not merely its decisions: how many releases were evaluated, how many blocked, how many conditioned, how many waivers granted and retired (§12). A control whose operation is unevidenced is a control whose effectiveness is asserted.
- **Independence.** This is the hardest and the most important. The question is *who may sign*, and specifically **whether the same function that built the control may attest to it.** The structural problem is not integrity, it is visibility: a team that designed a control, implemented it and operates it has a model of how it works that includes all its known limitations, and that model is exactly the model that will blind it to the limitations it does not know about. Three constructions mitigate this, and institutions vary in which they use:

| Construction | How it works | Its limit |
|---|---|---|
| **Separate the roles** | The function that builds attests to nothing; an independent function (a second-line risk or security-assurance function) reviews the evidence and signs | Costs a second team's capacity; the independent function can still lack the technical depth to challenge |
| **Separate the artefacts** | The builder produces the evidence; the signer's authority derives from being able to *test* the evidence rather than accept it — sampling the SBOM, re-running the scan, verifying the provenance independently | Depends on the signer actually testing rather than reading |
| **Separate the incentives** | The signer's performance is not measured on release throughput; if release velocity is the signer's metric, the signer's judgement has a known direction | Hardest to sustain, because the delivery organisation's incentives are rarely symmetrical with the gate's |

The repository's organisational frame for the three-lines and independence machinery is [Cybersecurity](cybersecurity_guide.md) §5 (the security organisation, governance, three lines of defence) and [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md); the resilience-side accountability treatment is [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) §3. The point to carry out of this guide is narrow and structural: **if the answer to "who could tell the gate it is wrong?" is "the gate", independence is absent, whatever the organogram says.**

---

## 11. The Theatre Problem

### 11.1 How a Gate Degenerates — and the Honest Framing

Everything in this section describes a **property of badly-designed gates**, not an accusation against any institution, team or person. That framing is not politeness; it is accuracy. Each degeneration below is the *rational* behaviour of competent people responding to the incentives a particular gate design creates. None of them requires anyone to be lazy, dishonest or cynical, and most of them are invisible from inside the process — which is exactly why they persist.

The organising insight: **a gate degenerates by being easier to satisfy symbolically than substantively, and then by having its symbolic satisfaction become the normal path.** §11.2–11.7 take the six observable degenerations in turn, and §11.8 collects them.

### 11.2 It Has Never Stopped a Release

**The observable:** the gate's block count is, over a long period, zero.

**The mechanism.** A gate with no recorded blocks is a gate that has never converted its authority into an effect. The mechanism is usually cumulative and unremarkable: the criteria are set at a level everything passes; judgement criteria are resolved by the decider's prior belief that the release will ship; and the release only reaches the gate once it is already certain to. Each individual interaction is defensible — nobody blocked a release for a trivial reason, which is the correct behaviour — and the aggregate is a control with no evidence of ever having done anything.

**Why it is rational.** Blocking a release in a real organisation is an expensive, relationship-costly act: it delays a committed date, creates a visible escalation, and requires the blocker to defend a judgement against a delivery organisation that has every incentive to argue. A decider who never blocks never pays that cost. And human judgement is genuinely uncertain — a criterion that *could* be read either way will be read the way that does not require a confrontation.

**The design fix.** Treat a zero-block history as a **finding about the gate**, not as evidence of excellence, and investigate it the way one would investigate a fraud-detection system with no alerts: are the criteria reachable? Are the releases reaching the gate? Is the decider the effective decider (§7.3)? And record the near-misses: a gate that conditions a release rather than blocking it is exercising authority (§7.2), and conditions are the more common shape of a working gate — but a gate with neither blocks nor conditions has produced no evidence of operation at all.

### 11.3 The Checklist Is Signed Without Reading

**The observable:** the readiness checklist is completed, complete, and indistinguishable across months and across releases of wildly different risk.

**The mechanism.** A checklist is a compression of judgement into boxes, and each box's satisfaction depends on facts the signer must obtain and weigh. As the checklist stabilises, the signer's task shifts from *evaluating the release* to *confirming the boxes are ticked* — a much easier task, and one that can be completed with progressively less engagement and faster with repetition. The checklist does not degrade; the *reading* behind it does, and the artefact looks identical.

**Why it is rational.** The checklist is long, the signer has fourteen other releases that day, the previous two hundred releases were fine, and the cost of a missed box is abstract and deferred while the cost of the extra twenty minutes is concrete and immediate. There is also a genuine asymmetry: for low-risk releases, the checklist genuinely *is* a formality, and treating every release as if its boxes needed deep scrutiny is a waste of scarce senior attention. The failure is that the same engagement level is applied to the high-risk release.

**The design fix.** Three things, in combination: **(a) tier the checklist by release risk** (§7.1), so that the deep review is reserved for releases that warrant it and the routine path is honestly routine — a checklist everyone knows is a formality for 90% of cases will be treated as a formality in the 10% too, unless the 10% is visibly different; **(b) move mechanical items out of the human checklist and into the automated gate** (§6.2), so the human reviews only what needs judgement; and **(c) require the record to state what would have changed the decision** (§7.5) — a field that cannot be completed by box-ticking, because it requires the signer to have understood the risk.

### 11.4 The Sign-Off Goes to Whoever Is Available

**The observable:** the recorded decider varies with availability rather than with the size or nature of the release; the largest releases are signed by whoever happens to be on duty.

**The mechanism.** A gate with a required sign-off and a real deadline develops a supply problem: the authorised decider is in a meeting, on leave, or in a different time zone, and the release will not wait. The organisation solves the immediate problem by widening the set of people who can sign, and widening it again, until the sign-off is available on demand. Each widening is a reasonable operational fix. The cumulative effect is an authority gradient inverted against risk: the person who signs is the person who was free.

**Why it is rational.** A blocked release with an available-but-unauthorised signer is a *visible* failure of the process; a release signed by a marginally-underskilled person is an invisible one. Organisations optimise for visible failures.

**The design fix.** Match authority to release tier explicitly, and make the *tier assignment* mechanical (§6.2) so that it does not depend on the availability of the person who would rather not escalate. Where the tier requires a senior decider, the correct answer to "they are unavailable" is a **deferral to the next window or a break-glass with the recorded reason** (§6.5) — not a substitution. And measure the substitution rate: if the recorded decider and the authorised decider diverge routinely, the record is describing a process that is not the process.

### 11.5 The Waiver Has No Expiry

**The observable:** the exception register contains entries whose age is measured in years, whose "follow-up" is a backlog item, and whose compensating control is a sentence rather than a control.

**The mechanism** and **the design fix** are §8.3's subject; here it belongs to the theatre catalogue, because its *symptom* is different from its cause. The observable symptom is not the missing date; it is that **the exemption list has become longer than the criteria list**. In the mature form of this degeneration, the gate's effective criteria are the criteria minus a large and stable set of standing exemptions — a state in which the gate's published criteria describe a control the organisation does not operate, while the register describes the control it does. Nobody has lied; the exemptions were each granted for a reason, and the reasons were never revisited.

**Why it is rational.** Granting an exception unblocks a release and creates one administrative task (the register entry). Retiring an exception requires someone to redo the work the exception avoided, which is a real cost attached to a person who has since moved on. Systems grant exceptions at a low cost and retire them at a high one; the register's composition is the predictable equilibrium.

**The design fix.** §8.3's expiry with a consequence, plus §12's metric — **the age of the oldest open exception** — watched as a first-class indicator. When the oldest open exception stops ageing (because the register was cleaned by deletion rather than by remediation), that too is a finding.

### 11.6 The Evidence Is Assembled After the Decision

**The observable:** the readiness record is complete, consistent and *written in one voice, at one time*, well after the deployment it describes.

**The mechanism.** The decision is made on time, for good operational reasons, by the people who were going to make it anyway — in a stand-up, on a call, in a chat thread. The gate's documentation requirement is then satisfied afterwards, by collating the evidence that supports the decision already taken. The result is a record that is *accurate about the evidence and false about the process*: it presents a decision derived from an examination, when the examination was performed in reverse.

**Why it is rational.** The decision-makers had the context in their heads, the trade-off was real, and the deployment could not wait for the paperwork. Assembling the record afterwards is diligent behaviour by the person doing it — they are capturing real evidence about a real decision — and it is the only route available to them once the sequencing has slipped.

**The design fix.** **The sequencing is the control.** Evidence must be *ingested* into the record before the decision is recorded; in a pipeline, this is a mechanical property (§9.1) — the record is assembled by the gate as the criteria are evaluated, and the decision field is written last, with a timestamp. Where the process is manual, the equivalent is that the readiness review happens before the deployment window is announced, so that the window cannot create the pressure that reverses the sequence. The retrospective reconstruction of §5.4's earlier answers is the diagnostic: if the record can only be produced by the people who made the decision, from memory, the record was assembled after the fact.

### 11.7 It Blocks the Small Thing and Waves Through the Large One

**The observable:** the gate blocks on formatting, on a missing ticket reference, on a documentation field — and approves an architectural change with a stale threat model, a lapsed dependency and a pentest that predates the rewrite.

**The mechanism.** Criteria that are cheap to verify get verified; criteria that are expensive to verify get assumed. The gate's *effort* allocation follows the availability of evidence rather than the size of the risk, and because mechanical criteria produce evidence trivially while structural ones require genuine analysis, the gate's attention migrates to the trivial. This is §9.2's build-gate/release-gate confusion in its most damaging form, and it is also §6.2's automate-judgement error seen from the other side: the gate has automated what was easy and delegated to humans precisely the things they had no time to do.

**Why it is rational.** The trivial criterion is *certain* — the decider knows the missing field is missing, and knows that refusing on that basis is defensible. The structural question is *uncertain* — the decider suspects the threat model is stale but cannot prove the gap matters, and refusing on an unprovable suspicion against a delivery organisation is a career-costly act. Certainty of a small wrong beats uncertainty about a large one, every time, under pressure.

**The design fix.** Two, and they must be used together. **(a) Give the gate criteria that only a release-context evaluation can satisfy** (§9.2) — a currency check on the threat model, a dependency-support check, a pentest-coverage check against the change's delta, an architecture-change flag that escalates the review tier. Without these, the gate *has* nothing large to block on. **(b) Make the judgement criteria decidable** by requiring the evidence that settles them: not "is the threat model still valid?" but "which of this release's changes touch a trust boundary, and does the model cover them?" A question with a determinate answer can be blocked on without the decider having to bet their credibility.

### 11.8 The Pattern Behind All Six

Six degenerations, six mechanisms, six fixes — and one pattern that holds across every row: **each fix is a change to what the gate can *decide*, what it can *see*, or *when* it is sequenced, and not one of them is a change to what the people are asked to try harder about.** No amount of emphasis in a policy document changes a gate whose criteria are unreachable, whose decider is the effective decider by accident, or whose evidence arrives after the decision. Theatre is not a discipline problem. It is a design problem wearing a discipline problem's clothes. §14 restates the six as symptom, cause and guardrail for use as a review checklist.

---

## 12. The Metrics That Tell You Whether the Gate Is Real

### 12.1 A Small Honest Set

Most gate metrics are vanity: release counts, average review duration, percentage of releases "security approved" — a figure that will read 100% in an organisation with a strong gate and 100% in one with none. A short, diagnostic set is worth more than a dashboard. Five numbers, each with its reading:

| Metric | What it is | What it tells you |
|---|---|---|
| **Blocks and conditions** | How many releases the gate stopped, and how many shipped with named conditions — **both, over time** | The gate's exercising of authority. Neither number should be zero, and the *conditions* count is usually where a working gate lives (§7.2, §11.2) |
| **Open exceptions, and the age of the oldest** | The stock of live waivers and the age of the longest-lived | The organisation's accumulated, uncarried liability (§8.3, §11.5). The age figure is the single most diagnostic number in the set, because it is the one that cannot be improved by better reporting |
| **Time from detection to remediation** | For findings the gate or the pipeline raised, how long until closure | The organisation's actual ability to fix things — which is the capability every exception and every condition implicitly assumes exists |
| **Recurrence rate** | The proportion of findings that are repeats of a class previously found and closed | Whether the organisation is fixing instances or causes. The SSDF's RV.3 tasks make this an explicit expectation — analysing root causes, analysing them over time to identify patterns, reviewing for similar vulnerabilities, and updating the SDLC ✅ (SP 800-218) |
| **Break-glass rate** | How often the emergency path is used, and for what | Whether the gate's normal path is fit for purpose. A rising rate is a statement about the gate, not about the emergencies (§6.5) |

Two secondary but useful additions: **the false-positive/false-override rate** (§6.4 — how often the gate blocks and the block is subsequently overridden, which measures the gate's precision directly), and **the divergence between the recorded decider and the authorised decider** (§11.4 — a governance indicator that no delivery metric will surface).

### 12.2 A Gate's Metrics Can Be Gamed in Either Direction

The caution is structural and applies to the whole set:

- **Too few blocks means the gate is not working.** A gate with no stops has produced no evidence of authority (§6.6, §11.2). The response to a low block count is to inspect the criteria and the routing — not to celebrate.
- **Too many blocks means it will be routed around.** A gate that stops a large fraction of releases on criteria the organisation overrides will be circumvented, and the circumvention will be invisible, because routed-around traffic does not appear in the gate's own numbers (§6.4). A high block count is therefore *also* not a success signal, and in the worst case it is a symptom of an organisation that has not yet built the workaround but is about to.
- **Every metric in the set can be improved without improving the gate.** Exceptions can be retired by deletion; recurrence can be reduced by reclassifying findings into a different category; detection-to-remediation can be shortened by raising the closure threshold; the break-glass rate can be reduced by never calling it a break-glass. **The defence is not a better metric; it is reading the metrics together and against the records.** An exception stock that fell while remediation throughput did not is a reclassification event. A block count that fell while the release count rose is a criteria relaxation. The numbers are diagnostic only in combination, and the record (§5) is what makes the combination interpretable.

### 12.3 The One Metric That Cannot Be Gamed

Sort the exception register by expiry date and look at how many entries expired in the last quarter — then check whether each expiry produced a re-decision. **Entries that expired without a re-decision are, by themselves, proof that the gate's exception machinery is nominal.** No reporting change, reclassification or threshold adjustment produces that state as a false positive. It is the closest thing this guide has to a single honest indicator, and it costs one query against the register.

---

## 13. The Cymbal Bank Worked Example

### 13.1 The Honesty Note, First

**This example is explicitly illustrative and fictional.** Cymbal Bank is the house persona; the release, the systems, the findings, the dates, the counts and the people in this section are pedagogical constructions, none of them drawn from any real institution's disclosures, and none of them verifiable by design. **Any figure in §13 is illustrative.** The example exists to demonstrate the mechanics of §6–§9 and §11–§12 in one continuous narrative, including the parts that do not go well.

### 13.2 The Release

Cymbal Bank's technology division has a platform release — call it **Release 2026.09 of the Cymbal Bank Corporate Onboarding Portal** (illustrative) — scheduled for the second Tuesday of the month. The portal is internet-facing, used by corporate clients to submit onboarding documentation and by internal operations staff to process it. The release is a **material change**: it re-platforms the document-handling service, adds an outbound integration to a third-party identity-document verification provider, and changes the authorisation model on the internal processing screens.

The risk tier lands, per §7.1's illustrative table, at **material change with a novel external integration** — so the readiness review runs with the security function's designated reviewer and the application owner, and escalates to an accountable technology owner if any severe finding is open at decision time.

### 13.3 The Evidence Assembled

| Evidence item (§3.2) | State at review | Note |
|---|---|---|
| **Threat-model currency** | A model exists, produced eleven months ago. It covers the portal's original trust boundaries and **does not cover** the new identity-verification integration or the changed internal authorisation model | The gap is recorded as a gap, not resolved by assumption (§5.2) |
| **SAST / DAST / SCA** | SAST and SCA ran against the release candidate's commit; DAST ran against the pre-production instance. Two high-severity SAST findings in the new authorisation code, both triaged as real and both open | Findings recorded with disposition, not as a count (§2.2) |
| **Pentest** | The last portal pentest was **seven months ago**, before the re-platforming; the stated scope did not include the onboarding document service or the new integration | The currency question is stated explicitly: the test does not speak to this release's delta |
| **Open vulnerabilities, with severity and age** | Nineteen open findings across the portal's components: two high (the SAST findings above), seven moderate, ten low. **The oldest moderate is 141 days old** — a component the platform team inherited and has deprioritised twice | The age figure is what makes this item decision-relevant (§3.2 item 4) |
| **Dependency and supply-chain state** | SBOM generated for the release candidate; completeness characterised as *incomplete-third-party* for two vendored components with no upstream SBOM. No provenance attestation from the identity-verification provider — the integration is new and the provider's security documentation was supplied as a PDF questionnaire response, not as an attestation | The verified / attested / assumed split is marked (§10.4) |
| **Secrets and configuration** | Clean — no injected secrets; configuration baseline validated in pre-production | The one item with no gap |
| **Pipeline integrity** | Build from a hosted platform with signed provenance; artefact digest recorded; a pre-production admission policy verifies the signature | Attributable and verified at admission — roughly the L2–L3 shape of §9.3 |
| **Blast radius and reversibility** | The release touches a client-facing journey and an internal processing path. Rollback plan exists for the application deployment; **the re-platformed document service's data migration is not reversible** — a point the review flags | The rollback asymmetry is the single largest determinant of the release's risk (§3.2 item 8) |
| **Prior-exception disposition** | Two open exceptions from the previous release: one 60-day waiver on a moderate dependency (expiry in 19 days), one condition to complete an internal authorisation test suite (date passed four weeks ago, not completed) | The passed condition is a live finding about the *previous* decision (§3.2 item 9) |

### 13.4 Two Releases, Two Outcomes

**The first release: the gate conditions and then blocks.** The review produces a conditional go with three conditions: (1) the two high-severity authorisation findings closed or documented with a compensating control and a dated waiver; (2) the identity-verification integration's threat-model coverage produced before the window; (3) the previous release's overdue condition closed. Conditions 1 and 2 are met within the week. Condition 3 is not — and here is the part worth studying: **the gate blocks on condition 3, which is not a finding about this release at all.**

The organiser's first reaction is that this is precisely §11.7's "the gate blocks the small thing" — the release is being held up by a four-week-old test-suite obligation from a different release. The gate decider's answer is the design argument, not the bureaucratic one: the overdue condition is on the *same authorisation model* this release changes, so it is not a separate matter but an unknown that the current change sits on top of. The release is deferred by one cycle.

**What this shows.** The block is defensible and the record shows why (§7.5): the criterion, the evidence, the residual risk, what would have changed the decision. It also shows the cost of a gate with real authority: a committed client-facing date moved, an escalation to the accountable technology owner, and a conversation at the next delivery review. **A gate's credibility is purchased with exactly these costs, and paid for either in them or in its own irrelevance.**

**The second release — and the honest possibility that the gate was wrong.** Three cycles later, the gate blocks again, this time on a moderate finding in a component that is not internet-reachable: the criterion reads "no moderate findings in a release's changed components", and the finding is in a *changed* file whose execution path is only reachable from an internal operations screen behind privileged access. The delivery team pushes back with the reachability analysis, and — after the review — **the gate decider agrees that the block was wrong.** The finding was real and the criterion was mis-calibrated: it treated "changed component" as a proxy for "exposed component", which was true of the internet-facing services where the criterion originated and false here.

**The correct resolution is the one the guide should be judged on: the criterion is renegotiated, not the release. The release ships, the criterion is amended on the record with the reachability qualifier, and the amendment is reviewed at the next gate calibration.** That is §6.4's third consequence in action. The wrong resolution — the one that appears in §11 and §14 — is the silent one: the finding is reclassified, or the component declared out of scope, or a waiver granted with no expiry, and the criterion stays nominally in force while ceasing to operate. The difference between the two resolutions is *not* whether the release shipped. It is whether the organisation's stated criteria and its operating criteria are still the same document.

### 13.5 The Exception Negotiation, With a Date

The moderate finding that stays open is handled as §8 specifies.

| Field | Value (illustrative) |
|---|---|
| **Requirement being waived** | Cymbal Bank internal criterion **RR-07 v3** (no moderate-or-higher findings in changed components) — amended mid-period by the §13.4 reachability qualifier |
| **Owner** | Named application owner for the onboarding portal (accountable for the finding's resolution, not the security function) |
| **Rationale** | The finding is in a dependency's parsing routine; the vendor's patched release is scheduled for the next quarterly maintenance window; the exposed path requires privileged internal access and is behind the existing access-control layer |
| **Compensating control** | *Pattern 2 (reduced exposure) + Pattern 1 (compensating detection)*: the affected processing path disabled for non-privileged internal roles; a detection rule added on the parsing routine's error signature with a named responder, and the rule's firing tested before the release |
| **Evidence the control works** | The detection rule was tested against a synthetic payload and fired; the disabled path's removal was verified in pre-production; the privileged-role population was enumerated and reviewed |
| **Expiry** | The maintenance window date — **43 days from the decision** — after which the exception lapses and the release that depends on it is re-decided or the component is remediated |
| **Register** | Entered once, visible to the gate, reviewed at the monthly exception review with the expiry as the sort key |

### 13.6 The Escalation to the Accountable Owner

The first release's deferral escalates, and the escalation is the point of §7.1's tiering: the decision that a client-facing committed date moves is not the security reviewer's to take unilaterally, and the decision that the release should ship anyway is not the security reviewer's to take either. The accountable technology owner receives the record — criteria, evidence, gaps, the two open high findings' disposition, the threat-model gap, the overdue condition, the irreversibility of the data migration, and what would have changed the decision — and decides. In the illustrative outcome, the owner **upholds the deferral**, on reasoning the record made available: the irreversibility of the migration plus an authorisation model with unclosed test obligations is a combination where a one-cycle delay is cheaper than the alternative, and the alternative's cost is not bounded by the release's own scope.

The transfer is therefore explicit: the deferral is the owner's decision, recorded as such, with the rationale. **The security function advised; the owner decided; the record names both.** That is §7.4's formulation, and note that it required the security function to make the risk legible in commercial terms rather than to assert a finding count.

### 13.7 The Review a Month Later

At the review, four questions are asked of the *record*, not of the release:

| Question | What the review found (illustrative) |
|---|---|
| **Did the conditions get met?** | Condition 3 was closed eleven days after the deferral; the two high findings were remediated; the threat-model gap was closed by the design team, who found one further trust boundary the original model had missed |
| **Did the exceptions behave?** | The 60-day dependency waiver from the previous release expired and was **re-decided** — eight days late, which is a finding about the notification path (§8.3 consequence 3); the new 43-day exception remained open with five weeks to run |
| **Did the gate block anything it should not have?** | The §13.4 mis-calibrated block. The review produced the criterion amendment and scheduled a calibration pass on the remaining criteria |
| **Did the decisions hold?** | The deferral held — the delayed release shipped a cycle later with the migration's irreversibility explicitly accepted, and the client committed date moved without incident. The release that followed the amended criterion shipped on schedule with no security event in the following month |

**The lesson the example is built to deliver** is in the last two rows. A release that shipped on an amended criterion is not a gate failure; a criterion that was amended *on the record, for a stated reason, and reviewed* is a gate doing its job — including the part of its job that consists of being wrong and finding out. What the example does not show, because it is the other story, is what a month-later review finds in an organisation where none of these fields existed: it finds that the questions cannot be asked.

---

## 14. The Anti-Patterns — Symptom / Cause / Guardrail

Seven symptoms, each with the structural cause behind it and the guardrail that addresses the cause rather than the symptom. The guardrail column is deliberately short and points at the section that develops it — an anti-pattern list that ends in new paperwork has made the problem worse.

| # | Symptom | Cause (structural) | Guardrail |
|---|---|---|---|
| 1 | **The checklist is signed unread** — the artefact is complete, identical across wildly different releases | The signer's task drifted from evaluating the release to confirming boxes; mechanical items were left to humans while judgement items got no time | Tier the checklist by release risk; automate the mechanical items into the gate; require the "what would have changed the decision" field (§6.2, §11.3) |
| 2 | **The waiver with no expiry** — the exemption list is longer than the criteria list | Exceptions are cheap to grant and expensive to retire, so the stock accumulates and the register becomes the real criteria set | Owner + rationale + compensating control + **dated expiry with a consequence for lapse**; monitor the age of the oldest open exception (§8.2–8.3, §11.5) |
| 3 | **The sign-off by someone who cannot say no** — the recorded decider is not the effective decider | Where the sign-off sits is decided by the organogram; where the schedule pressure sits is decided by the delivery plan. When they differ, the pressure holds the gate | Test the role by asking what happens when it says no; match authority to release tier mechanically; measure the recorded-versus-authorised divergence (§7.3, §11.4) |
| 4 | **The scanner count treated as the risk** — "12 findings, 2 critical" as the entire risk statement | The gate's pass condition is a number a tool produces, so the gate has outsourced the judgement to a tool's false-negative rate | Report findings **with disposition**; require severity **and age**; require the risk to be described in a form a non-security owner can act on (§2.2, §3.4, §7.4) |
| 5 | **The gate blocks trivia and misses the architecture** — a missing ticket field stops the release; a stale threat model does not | Effort follows evidence availability, not risk size; cheap criteria are certain and defensible, structural ones are uncertain and career-costly | Give the gate criteria only a release-context evaluation can satisfy; make judgement criteria decidable by naming the evidence that settles them (§9.2, §11.7) |
| 6 | **The break-glass path with no log** — the emergency route exists and leaves no trace | The emergency path is designed for speed, and logging is experienced as friction at the moment of the emergency | Named authoriser, recorded reason, enumerated unmet criteria, compensating controls, dated follow-up, its own rate metric (§6.5) |
| 7 | **The release-ready claim made before the evidence exists** — the record is complete, consistent and written in one voice after the deployment | The decision was made on time in a chat; the documentation requirement was satisfied retrospectively by collating supporting evidence | Sequencing as the control: evidence ingested before the decision is recorded, decision field written last with a timestamp; the review precedes the window announcement (§5.4, §11.6) |

Read the table down the *guardrail* column and the pattern is the same as §11.8's: no row asks anyone to try harder. Each row changes what the gate can decide, what it can see, or when it is sequenced.

---

## 15. The Claims Audit

This pass had **live web access by direct page extraction only** — `web_search` returns empty from this host, which is a tool limitation and **not** evidence of absence. Every source below was reached by extracting the issuing body's own page (or the paper's own API). The ledger records what this pass stood behind, what it flagged, and what it rejected. **Verification pass: 2026-09-25.**

### 15.1 Verified — standards, versions, identifiers and dates

| Claim | Where cited | Verdict | Verification basis and its quality |
|---|---|---|---|
| NIST SP 800-218, **SSDF Version 1.1**, published **February 2022**, 36 pp., doi 10.6028/NIST.SP.800-218; final posted 3 Feb 2022; supersedes CSWP 13 (April 2020) | §4.2 | ✅ | Full document extracted from the NIST publication server; publication metadata from `csrc.nist.gov/pubs/sp/800/218/final`. Primary, dated, high quality |
| SSDF practice groups **PO / PS / PW / RV** with their published definitions; the four-element practice definition (Practice / Task / Notional Implementation Example / Reference) | §4.2 | ✅ | `csrc.nist.gov/projects/ssdf` project page. Primary, high quality |
| SSDF task identifiers PO.1.1–PO.5.2, PS.1.1–PS.3.2, PW.1.1–PW.9.2, RV.1.1–RV.3.4, with the v1.1 retired/moved identifiers | §4.2 | ✅ | Identifiers enumerated from the published table in SP 800-218 and cross-checked against the change log. Primary, high quality |
| Task texts and examples quoted for **PO.1.3, PO.2.1, PO.2.3, PO.3.1, PS.3.1, PS.3.2, PW.1.1, PW.1.2, PW.6.1, PW.6.2, RV.1.1, RV.1.2, RV.2.1, RV.2.2** | §4.2, §8.1, §10.4, §12.1 | ✅ | Quoted verbatim from the extracted SP 800-218 table. Primary, high quality |
| Artifact = "a piece of evidence"; Evidence = "grounds for belief or disbelief; data on which to base proof or to establish truth or falsehood" — definitions added in v1.1 | §3.1, §5.1 | ✅ | SP 800-218. Primary, high quality |
| SP 800-218 states the **shifting-left** principle and that the SSDF "is not intended to create a checklist"; its practices are voluntary for non-governmental organisations, consistent with OMB Circular A-130 and FISMA 2014 | §4.1, §4.2, §8.5, §9.4 | ✅ | SP 800-218 §1 and front matter. Primary, high quality |
| **SP 800-218A** exists and has been finalised — an SSDF community profile for generative AI and dual-use foundation models | §4.2 | ✅ (existence) / ⚠ (date) | `csrc.nist.gov/projects/ssdf` announcement. Primary for existence; the page read carried no publication date |
| **OWASP ASVS latest stable 5.0.0**; the `v<version>-<chapter>.<section>.<requirement>` convention (lowercase `v`), CSV/JSON availability | §4.3 | ✅ | `owasp.org/www-project-asvs/` project page, this pass. Primary, high quality — and an **upgrade** of the sibling guide's ⚠ marker |
| **OWASP SAMM latest version 2.2.0**; flagship project, Standards classification, published at owaspsamm.org | §4.3 | ✅ | `owasp.org/www-project-samm/` project page, this pass. Primary, high quality |
| **SLSA**: v1.0 restructured the levels into **tracks**; the **Build track L0–L3** with the published requirements; v0.1's single unnamed SLSA 1–4 track was replaced and the **Source aspects removed**; **v1.2 is the current version** | §4.4, §9.3 | ✅ (structure) / ⚠ (v1.2 date) | `slsa.dev/spec/v1.0/levels` and the site's current-version notice. Primary, high quality for the structure; the v1.2 date was not read |
| **SPDX** is **ISO/IEC 5962:2021**; the project's own release timeline (2.2 May 2020 satisfying NTIA minimum elements; 2.2.1 to ISO Aug 2020; ISO/IEC 5962 available Aug 2021; 2.3 Aug 2022; **3.0.0 April 2024**) | §4.4 | ✅ | `spdx.dev` front page and `spdx.dev/about/overview/`, this pass. Primary, high quality |
| **CycloneDX 1.7**, release date **2025-10-21**; standardised as **ECMA-424** published **2025-12-10**; developed by the OWASP Foundation with Ecma International; **TC54**; the `https://cyclonedx.org/bom` predicate type | §4.4 | ✅ | `cyclonedx.org/specification/overview/`, this pass. Primary, high quality |
| **Cyber Resilience Act = Regulation (EU) 2024/2847**; in force **10 December 2024**; Chapter IV from **11 June 2026**; Article 14 reporting from **11 September 2026**; full application **11 December 2027**; conformity assessment per Article 32 / Annex VIII; practical guidance published **27 July 2026** | §4.5 | ✅ (number and dates) / ⚠ (Article-level text) | European Commission pages (`digital-strategy.ec.europa.eu`) — the policy page (last update 7 September 2026) and the legislative summary (last update 3 December 2025). Official EU source, but **not** EUR-Lex, which would not render (see §16) |
| **PCI DSS v4.0.1** is the current PCI DSS standard listed in the Council's own document library | §4.6, §10.5 | ✅ (version) | `pcisecuritystandards.org/document_library/`, this pass. Primary for the version |
| The Council's own listings include **Secure SLC Qualified Software Vendor** and **Software Security Framework Assessor** categories; and the v4.x supporting instruments (Summary of Changes v4.0→v4.0.1, Targeted Risk Analysis Guidance, Extra Compensating Controls Worksheet, Prioritized Approach, ROC/SAQ set) | §4.6, §10.4–10.5 | ✅ | `pcisecuritystandards.org` document library, standard page, resource hub and FAQ company-type listing, this pass |
| **DORA = Regulation (EU) 2022/2554, applied 17 January 2025** | §10.2 | ✅ (cross-reference) | The repo's verified ledger (EIOPA's DORA page) — recorded, not re-read this pass |
| **MAS**: TRM Guidelines published **18 January 2021**; sectoral TRM notices cancelled with effect from **10 May 2024**; **FSM-N05** (banks) and **FSM-N22** (cyber hygiene); the shared notice language on reliability, availability, recoverability of critical IT systems and IT controls | §10.3 | ✅ (cross-reference) | The repo's verified `mas.gov.sg` extracts, recorded in the MAS guide and TLM guide ledgers |
| Patel, Boucher, Fallahzadeh, Hassan & Adams, **"A State-of-the-practice Release-readiness Checklist for Generative AI-based Software Products"**, **arXiv:2403.18958v1, 27 March 2024** | §1, header | ✅ | Retrieved from the arXiv API over HTTPS with a browser User-Agent. Primary for the paper's existence, authors, date and abstract |

**Reading the verified table.** The centre of gravity is exactly where a guide like this should have it: the **standards frame's versions and dates** are verified at the issuing bodies, and the **task-level SSDF content** is quoted from the primary document rather than paraphrased from a vendor summary. Two items are verified for existence but flagged for their associated dates (§15.2).

### 15.2 Flagged — recorded as ⚠ and not to be relied on without a re-check

| Claim | Where | Why it is flagged |
|---|---|---|
| **SP 800-218A's publication date** | §4.2 | The CSRC project page announces it as finalised but the page read carried no date |
| **SLSA v1.2's release date** | §4.4 | The v1.0 levels page and the site banner confirm v1.2 is current; no v1.2 date was read |
| **CRA Article-level text** (Article 14, Article 32, Annex VIII enumerated by the Commission) | §4.5 | Read at the Commission's summary, not at EUR-Lex; the Official Journal text governs |
| **OMB M-22-18's own title and date** | §4.2 | Cited in the repo's supply-chain guide lineage; not re-read at the issuing body this pass |
| **PCI DSS requirement numbers, titles and sub-requirements** | §4.6, §10.5 | **Deliberately absent from this guide.** The standard text could not be retrieved at the Council's hosting endpoints; a requirement number not read at the issuing body is exactly the fabrication §4.1 rule 1 forbids |
| **PCI DSS v4.0.1's own publication date** | §4.7 | The document library confirms it as current; its publication date was not read |
| **MAS instrument detail** (TRMG text, FSM-N05/FSM-N22 wording) | §10.3 | Held in verified cross-reference, not re-extracted from `mas.gov.sg` this pass |
| **Any regulator's expectation** stated in §8.6's middle column | §8.6 | Mostly ⚠-structural: the guide is not a compliance source and will not invent supervisory expectations |
| **Expiry-norm, compensating-control patterns, tiering tables, retention principle** | §7.1, §8.3–8.4, §5.3 | This guide's own recommendations, clearly labelled; practice design, not standards text |
| **All Cymbal Bank worked-example content** | §13 | Illustrative and fictional by design; every figure is a pedagogical construction and unverifiable (§13.1) |
| **OWASP Top 10 version currency** | §4.3 | Not asserted this pass — the repo's Top 10 content is the sibling guide's, and this guide does not claim a current edition |

### 15.3 Rejected — false friends and dedup findings

| Term | Finding | Disposition |
|---|---|---|
| **`release gate`** — **REJECTED TERM in this repo** | The phrase appears in **22 files**, but as a **false friend**. In this repository it is overwhelmingly the **AI-evaluation** sense ("Evals are the release gate" in `ai_native_companies_guide.md`) plus coverage-threshold, phase-gate and tranche-release senses — **not** the information-security sense this guide owns | Recorded here in the same way the repo's earlier guides report the *rust/trust*, *CAMS/scams*, *ARES/Ares Management* and *RGB/colour-space* false friends. Do **not** treat the 22-file count as prior art for this guide's subject |
| **`release readiness`** | Appears in exactly **1 file** — a passing line in `management/team_lead_methodologies_guide.md` line 702 ("The lead owns release readiness: what's in the release, who signs off, what the rollback is, who's on call") | A **team-lead methodology guide**, not an owner of the subject. No prior owner exists; this guide is the first |
| **`security sign-off`** | **2 files**, both AI/agent or vendor-onboarding contexts | Not prior art for the enterprise release gate; noted, not adopted |
| **`\bSSDF\b`** | **1 file** — a passing citation in `technology/vuln_scanning_c_proc_cobol.md` line 31 ("US Executive Order 14028, NIST SP 800-218 (SSDF)") | A passing cite, not an owner. This guide is where the SSDF's release-relevant tasks are read at NIST |
| **`SBOM` / `SLSA` ownership** | **24 files** mention SBOM (deepest: `technology/sbom_c_proc_cobol.md`, 73 mentions); **6 files** mention SLSA (none dedicated) | No single owner; this guide consumes the inventory as evidence (§3.2 item 5) and cross-references the two supply-chain siblings rather than re-deriving generation |
| **`TLM §7 "Governance Gates"`** | Confirmed by reading `technology/technology_lifecycle_management_guide.md` §7 — its subject is **inventory, tracking and lifecycle gates** (do we know what the asset is, its state, and when support ends?) | **Not** release readiness. Explicitly stated in §1.5 so no reader infers an overlap that does not exist |

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

Nothing below is guessed. Each item is either flagged in the body or is an explicit absence, and this section names the gaps so a reader knows exactly where to re-check before relying on the guide.

- **`web_search` returned empty from this host.** This is a **tool limitation, not evidence of absence**: every source was therefore reached by direct extraction of the issuing body's own page, and where a fact was unavailable by that route it is flagged rather than inferred from search-absence.
- **EUR-Lex did not render.** Two EUR-Lex URLs for the Cyber Resilience Act (`.../eli/reg/2024/2847/oj` and `.../legal-content/EN/TXT/?uri=OJ:L_202402847`) returned no renderable content to this tooling. The CRA's regulation number and dates are therefore cited from the **European Commission's own pages** — an official EU source — and **Article-level** citations are flagged ⚠.
- **The PCI DSS v4.0.1 standard text was not retrievable.** The Council's document-hosting endpoints returned no content. **Consequence: this guide cites no PCI DSS requirement number anywhere.** The version is verified at the Council; the requirement-level content is an explicit gap, and the reader must take it from the current standard PDF via the Council's own library. Its publication date and the prior-version timeline were likewise not read.
- **SP 800-218A's publication date** and the detail of its AI-specific tasks were not read — only its existence and its stated purpose on the CSRC project page. **SLSA v1.2's requirements and date** were not read either: the v1.0 Build-track structure is verified, and any criterion written against "the current SLSA" should be pinned to a version and a track before use.
- **SPDX 3.0.x's patch level beyond the 3.0.0 release** was not read; the release timeline is verified, the current patch level is not. **ISO/IEC 5962's clause content** is paywalled and was not read — the standard's identity and its correspondence to the SPDX line come from `spdx.dev`'s own timeline.
- **The OWASP Top 10's current edition** is deliberately not asserted here; that material belongs to [Security-by-Design](security_by_design_guide.md) §4.2 and was not re-read this pass. **The MAS instruments' text** (TRMG, FSM-N05, FSM-N22) and **DORA's text** are held in verified cross-reference rather than re-extracted.
- **No specific retention period, exception expiry norm or approval-tier structure** is presented as an external requirement. Those are this guide's recommendations (§5.3, §7.1, §8.3) and are labelled as such. **No regulator's expectation is stated as verified** beyond the cross-referenced instrument facts — §8.6's middle column is deliberately ⚠.
- **All Cymbal Bank worked-example content is illustrative and fictional** by design (§13.1). **No institution is asserted anywhere in this guide as a user of any practice described in it.**

### 16.2 The Glossary

| Term | Definition | Section |
|---|---|---|
| **Release readiness** | The state of having enough evidence, reviewed by the right people, to make a defensible decision that a specific release's residual risk is acceptable to ship | §1.1 |
| **The release gate** | The control point that must be satisfied before promotion — automated checks plus human judgement with the authority to stop the promotion | §6.1 |
| **The sign-off** | A named individual's recorded acceptance of the decision and its residual risk | §7.3 |
| **Residual risk** | The risk remaining after the controls in place, which the release carries into production | §2.4 |
| **Go / conditional go / no-go** | The decision vocabulary; a conditional go requires an owner and a date for each condition | §7.2 |
| **Exception / waiver** | A time-boxed, owned, justified decision to ship without meeting a stated requirement or criterion | §8.2 |
| **Compensating control** | A different control that addresses the same risk by another route when the primary control is unmet | §8.4 |
| **Evidence artefact** | An object recording what was done and found — a report, attestation, provenance record, log; an artefact is "a piece of evidence" in the SSDF's own definition ✅ | §3.1 |
| **Blocking gate / warning / guardrail** | The three enforcement strengths; only the first can stop a promotion | §6.3 |
| **Break-glass** | The documented, logged, owned path for promoting despite an unmet criterion | §6.5 |
| **SSDF** | NIST's Secure Software Development Framework, SP 800-218 v1.1 (February 2022), with the four practice groups **PO / PS / PW / RV** — Prepare the Organization; Protect the Software; Produce Well-Secured Software; Respond to Vulnerabilities ✅ | §4.2 |
| **ASVS** | OWASP's Application Security Verification Standard — the **verification standard**; latest stable **5.0.0** ✅ | §4.3 |
| **SAMM** | OWASP's Software Assurance Maturity Model — the **maturity model**; latest **2.2.0** ✅ | §4.3 |
| **SLSA** | Supply-chain Levels for Software Artifacts — levels organised into **tracks**; v1.0's Build track L0–L3 ✅ | §4.4 |
| **SBOM** | Software Bill of Materials — the component inventory; SPDX (ISO/IEC 5962:2021) and CycloneDX (1.7 / ECMA-424) are the two formats read here ✅ | §3.2, §4.4 |
| **CRA** | The EU Cyber Resilience Act, **Regulation (EU) 2024/2847** — in force 10 December 2024, full application 11 December 2027 ✅ | §4.5 |
| **PCI DSS / PCI SSF** | The card industry's Data Security Standard, current at **v4.0.1** ✅ with requirement numbers deliberately not cited here; and the Council's separate software-security-framework stream (Secure SLC Qualified Software Vendor) ✅ | §4.6, §10.4 |
| **DORA** | Regulation (EU) 2022/2554, applied 17 January 2025 ✅ (cross-reference) | §10.2 |
| **MAS / FSM-N05 / FSM-N22 / TRMG** | The Singapore instruments in verified cross-reference; the instrument text governs | §10.3 |
| **Theatre** | A gate that is easier to satisfy symbolically than substantively, whose symbolic satisfaction becomes the normal path | §11 |

### 16.3 The Cross-References

**In `technology/` (plain filenames):** [Security-by-Design](security_by_design_guide.md) — **THE design-time companion**, and the producer of most of this gate's evidence: §2 threat modelling (the currency question in §3.2 item 1), §3 design principles, §4 secure coding, §5 the testing arsenal (SAST/DAST/IAST/SCA/pentest), §6 DevSecOps — CI/CD gates, secret management, SBOM and SLSA (§9.1–9.3), §7 cloud security, §8 regulatory, §9 its own worked example. · [Cybersecurity](cybersecurity_guide.md) — **THE programme companion**: the threat landscape, security operations, offensive security, the security organisation and three lines (§10.6), the frameworks map, the domains map and the banking context. · [Technology Lifecycle Management](technology_lifecycle_management_guide.md) — **THE lifecycle-governance companion**: §3 vendor EOL/EOSL, §4 technology risk, §5 MAS technology-risk expectations (§10.3), §7 inventory/tracking/governance gates (**inventory gates, not release gates** — §1.5), §8 software lifecycle including patches and supply chain. · [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) — **THE supply-chain-inventory deep-dive**; §3.2 item 5, §9.3. · [Vulnerability Scanning for C, Pro\*C and COBOL](vuln_scanning_c_proc_cobol.md) — the scanning reality; the repo's single existing SSDF citation. · [Power Platform CI/CD](power_platform_cicd_guide.md) — the low-code pipeline (§9.1). · [Dockerfile Best Practices](dockerfile_best_practices_guide.md) and [Docker Image Comparison Tools](docker_image_comparison_tools_guide.md) — image integrity and SBOM diffing (§9.1). · [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md) — platform-side admission and supply-chain controls. · [Deterministic Engineering](deterministic_engineering_guide.md) — reproducibility as the precondition for a build claim (§9.3). · [LLM Development Risks & Security](llm_development_risks_security_guide.md) and [Beyond Zero: Enterprise Security for the AI Era](beyond_zero_enterprise_security_guide.md) — the GenAI/agent release angle, and where "release" stops being a single event (§4.2, SP 800-218A).

**In `banking/` (prefix `../banking/`):** [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) — **THE operational-resilience companion**: the DORA/MAS frame, impact tolerances, third-party resilience, testing (§2.5, §10.2, §10.6). · [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) — the risk container for accepted risk (§7.4, §10.1, §10.6). · [Risk Management Models](../banking/risk_management_models_guide.md) — the risk-measurement vocabulary. · [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) — the verified MAS instrument map (§10.3).

**In `management/` (prefix `../management/`):** [Resilience Engineering](../management/resilience_engineering_guide.md) — the sociotechnical reading of why gates are routed around (§11). · [Vendor Management](../management/vendor_management_guide.md) — third-party assessment and exit (§10.4). · [Team Lead Methodologies](../management/team_lead_methodologies_guide.md) — the delivery-management context, and the repo's single passing use of "release readiness" (line 702).

**Primary sources and further reading:** NIST SP 800-218, *SSDF Version 1.1* (February 2022, doi 10.6028/NIST.SP.800-218) and SP 800-218A; `csrc.nist.gov/projects/ssdf`; Executive Order 14028 (12 May 2021) and OMB M-22-18 (⚠ date unverified); OWASP ASVS 5.0.0 and OWASP SAMM 2.2.0 at `owasp.org`; SLSA at `slsa.dev` (v1.0 Build track; v1.2 current); SPDX at `spdx.dev` (ISO/IEC 5962:2021; 3.0.0 released April 2024); CycloneDX 1.7 / ECMA-424 at `cyclonedx.org`; the Cyber Resilience Act, Regulation (EU) 2024/2847, at the European Commission's `digital-strategy.ec.europa.eu`; PCI DSS v4.0.1 at `pcisecuritystandards.org` (the Council's document library, standard page and resource hub); DORA, Regulation (EU) 2022/2554, via the repo's verified EIOPA cross-reference; MAS instrument facts via the repo's verified `mas.gov.sg` cross-references; and Patel, Boucher, Fallahzadeh, Hassan & Adams, *A State-of-the-practice Release-readiness Checklist for Generative AI-based Software Products*, arXiv:2403.18958v1 (27 March 2024).

### 16.4 The Closing Summary

**A release gate is a decision, not a document** — its output is a named person accepting a described residual risk, and its evidence set is the nine items of §3 in which the *disposition* of findings matters more than their count.

**Its standards frame is thinner than people expect, and that is the point** — the SSDF supplies practices and a shared vocabulary, ASVS supplies criteria, SAMM supplies a maturity claim, SLSA supplies provenance strength, SPDX and CycloneDX supply composition, the CRA supplies a manufacturer obligation and PCI DSS supplies a card-data baseline; **none of them supplies the decision**, which is why §7's question ends with "and who says so".

**The machinery that makes a gate real is unglamorous** — the record that can be produced on demand (§5), the blocking/enforcement distinction that keeps a guardrail from masquerading as a gate (§6.3), the signer who can actually say no (§7.3), and the exception that carries an owner, a rationale, a compensating control and a *dated* expiry (§8.2–8.3).

**Theatre is a design outcome, not a character flaw** — every degeneration in §11 is the rational behaviour of competent people responding to a gate that is easier to satisfy symbolically than substantively, and every fix in §14's guardrail column changes what the gate can decide rather than what people are asked to try harder about.

**And the test is behavioural, not documentary** — you cannot read a gate's quality off its policy, its dashboard or its checklist; you read it off its history. A gate with no recorded blocks and no conditions has produced no evidence that it exists, and the exception register sorted by expiry date will tell you more about whether an organisation's gate is real than any assurance narrative ever will.

The gate earns its place the first time it stops something that mattered, and it keeps its place every time after that — a gate that has never stopped anything is not a gate.
