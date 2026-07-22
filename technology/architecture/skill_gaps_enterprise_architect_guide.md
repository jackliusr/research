# Personalized Skill Gap Analysis: Solution Architect → Enterprise Architect

## For Jack Liu Shurui — Solution Architect (GenAI), Crédit Agricole CIB, Singapore

> **Analysis Date:** July 2026
> **Source Material:** github.com/jackliusr/research (100+ guides)
> **Target Role:** Enterprise Architect / Senior Architecture Roles in Banking

---

## 1. Executive Summary

Jack Liu Shurui is a Singapore-based Solution Architect at Crédit Agricole CIB's Corporate & Investment Bank, with a career spanning full-stack development, DevOps/SRE, solution architecture, and team leadership across multiple domains (SCADA, healthcare, ERP, e-commerce, payment systems, gaming, and core banking). His personal research repository contains **100+ comprehensive guides** — many exceeding 40,000 characters — demonstrating an extraordinary appetite for deep, structured learning.

The gap analysis reveals a **highly atypical profile**: Jack already possesses knowledge depth in EA frameworks (TOGAF, BIAN, Capability Engineering) that many practicing Enterprise Architects lack. His management/leadership research (SLII, 3D Managerial Effectiveness, Strategic Management, Managing Up/Down/Sideways, Storytelling) is unusually strong for a technical architect. The primary gaps are not in _knowledge acquisition_ — he clearly knows _how_ to learn deeply — but in: (1) **specialized banking domain depth** (trade finance, capital markets, treasury, risk models), (2) **emerging technology practice** (DevSecOps, FinOps, Zero Trust, Platform Engineering), (3) **soft skills for architect influence** (executive communication, negotiation, consulting methodology), and (4) **formal credentialing** (TOGAF certification, cloud architect certs).

**Core finding:** Jack is approximately 65–70% of the way to a senior Enterprise Architect profile. The remaining gap is bridgeable with 12–18 months of focused effort.

---

## 2. Skills Already Demonstrated

### 2.1 Technical Breadth — Cloud, Kubernetes, AI/ML

| Skill Area | Evidence from Repo | Depth Assessment |
|---|---|---|
| **Cloud Platforms** | `cloud_providers_guide.md` (1,196 lines) — AWS, Azure, GCP, Alibaba Cloud with enterprise comparison, multi-cloud strategies, decision trees, FinOps section | **Advanced.** Comprehensive, vendor-neutral enterprise analysis with selection criteria |
| **OpenShift / K8s** | `openshift_scc_comprehensive_guide.md`, `openshift_scc_service_account_guide.md`, `charmed_kubernetes_vs_openshift_guide.md`, `charmed_kubeflow_vs_openshift_ai_guide.md`, `openshift_random_uid_username_mapping.md` | **Advanced.** Deep dive into SCC admission flow, RBAC integration, platform comparisons |
| **AI/ML Platforms** | `ml_platforms_comparison_guide.md` (MLflow, Kubeflow, OpenShift AI, SageMaker, Vertex AI), `enterprise_ai_platforms_guide.md` | **Advanced.** Platform selection including deployment models for banking |
| **LLMOps** | `llm_evaluation_frameworks_guide.md`, `llm_compression_alternatives_guide.md`, `llm_guard_models_guide.md`, `multi_pass_llm_pipelines_guide.md`, `query_rewriting_rag_guide.md`, `spec_driven_development_for_llms_guide.md`, `llm_latency_optimization_guide.md` | **Very Advanced.** Production-grade depth across the full LLM lifecycle — evaluation, optimization, safety, RAG patterns |
| **Data Engineering** | `data_governance_framework.md`, `data_mesh_architectures_guide.md`, `data_lineage_tools.md`, `data_pipeline_versioning.md`, `schema_evolution_data_drift_guide.md`, `on_prem_lakehouse_analysis.md`, `data_vault_2_modeling.md`, `delta_lake_vs_iceberg.md`, `data_integration_frameworks_guide.md` | **Advanced.** Covers governance, architecture patterns, modern data stack, regulatory alignment |
| **Infrastructure & DevOps** | `iac_best_practices_guide.md`, `terraform_state_conflicts.md`, `mock_server_guide.md`, `test_orchestration_guide.md`, `ephemeral_filesystem_linux.md`, `sonatype_nexus_guide.md`, `jetty_guide.md`, `jib_container_builder_guide.md`, `elasticsearch_data_modeling_schema_design.md` | **Solid.** Practical infrastructure, build tooling, CI patterns |
| **Security** | `openscap_guide.md`, `osint_guide.md`, `ssh_key_disabled_user_guide.md`, `sbom_c_proc_cobol.md`, `vuln_scanning_c_proc_cobol.md` | **Intermediate-Advanced.** Compliance scanning, vulnerability management, SBOM |

### 2.2 Management & Leadership

| Skill Area | Evidence | Depth |
|---|---|---|
| **Situational Leadership II** | `situational_leadership_slii_guide.md` (495 lines) — Full D1–D4 model, style matching, goal setting, tech leadership application | **Advanced.** Theoretical depth plus practical application for engineering leaders |
| **3D Managerial Effectiveness** | `3d_managerial_effectiveness_guide.md` (479 lines) — Reddin's model, eight styles, situational determinants | **Advanced.** Obscure but valuable framework, well-researched |
| **Strategic Management** | `strategic_management_guide.md` (751 lines) — Mintzberg's 10 schools, strategy formulation, implementation, evaluation, tools | **Very Advanced.** Comprehensive coverage for a tech professional |
| **Managing Up/Down/Sideways** | `managing_up_down_sideways_guide.md` (525 lines) — Influence without authority, matrix orgs, stakeholder management, organizational awareness | **Advanced.** Directly applicable to architect role |
| **Storytelling** | `storytelling_skills_guide.md` (442 lines) — Neuroscience, narrative structures, ADR writing, executive communication | **Advanced.** Architect-specific application (ADRs, trade-off narratives) |
| **The Manager's Path** | `the_managers_path_research.md` — Camille Fournier's model for engineering management career | **Intermediate.** Good foundation for management track |
| **Team Growth** | `grow_team_guide.md` | **Intermediate.** Practical team development |

### 2.3 Enterprise Architecture Frameworks

| Skill Area | Evidence | Depth |
|---|---|---|
| **TOGAF** | `togaf_guide.md` (566 lines) — Full ADM phases, deliverables, content metamodel, TOGAF 9.2 → 10th Edition evolution, banking application | **Very Advanced.** Comprehensive, includes banking-specific guidance and certification path |
| **BIAN** | `bian_standard_guide.md` (896 lines) — Service Landscape, metamodel, APIs, regulatory alignment, microservices, cloud/SaaS, real-world adoption | **Very Advanced.** Exceptional depth — includes BIAN v10.0+ service domains, real adoption case studies |
| **Capability Engineering** | `capability_engineering_guide.md` (1,374 lines) — Modeling, decomposition, heat mapping, capability-based planning, banking decomposition example | **Very Advanced.** One of the most comprehensive guides in the repo — capstone-level |
| **Enterprise Architecture (Umbrella)** | `enterprise_architecture_guide.md` (812 lines) — BDAT domains, architecture maturity models, EA vs Solution Architecture, banking EA practice | **Advanced.** Solid overview integrating all frameworks |

### 2.4 Banking Domain Knowledge

| Skill Area | Evidence | Depth |
|---|---|---|
| **End-to-End Banking Processes** | `end_to_end_banking_processes.md` (545 lines) — FO/MO/BO workflows, account opening/KYC, trade lifecycle, payments, regulatory reporting, MAS Notice 626 | **Advanced.** Comprehensive coverage of wholesale banking operations |
| **Financial Trading Infrastructure** | `financial_trading_order_infrastructure.md` (502 lines) — Implied order processing, exchange architectures (CME, ICE, Eurex), latency optimization | **Very Advanced.** Specialized derivatives trading depth |
| **FinTech Overview** | `financial_technology_overview.md` (494 lines) — Payments, lending, digital banking, RegTech, Singapore FinTech landscape | **Advanced.** Broad coverage with Singapore-specific regulatory context |
| **Data Governance & Compliance** | `data_governance_framework.md` (569 lines), `data_compliance_frameworks.md`, `china_data_governance_frameworks.md` | **Advanced.** Framework comparison, regulatory alignment |
| **Payment Systems** | Via `axway_transfer_cft_guide.md`, `axway_cft_controlm_integration.md`, `control_m_guide.md`, `control_m_external_conditions_guide.md` | **Solid-Skilled.** Covers file transfer and batch scheduling in banking context |
| **Mainframe Integration** | `mainframe_file_sharing_notification.md`, `linux_file_sharing_notification.md` | **Intermediate.** Practical interop patterns |
| **Singapore-Specific** | `singapore_gcc_guide.md`, `singapore_security_clearance_guide.md`, `govtech_architecture_principles.md` | **Unique.** Valuable local context for Singapore banking |

---

## 3. Skill Gaps to Address

### 3.1 Banking Domain Depth (High Priority)

| Gap | Current State | Target State | Impact |
|---|---|---|---|
| **Trade Finance** | Mentioned peripherally in E2E banking processes; no dedicated guide | Deep understanding of LC (Letters of Credit), SBLC, documentary collections, supply chain finance, factoring, forfaiting. Trade finance systems (e.g., Trade Innovation, Marco Polo, we.trade, Contour) | **High.** Trade finance is a core CIB product — essential for EA at a European investment bank |
| **Capital Markets Architecture** | Derivatives trading infrastructure is well-covered; but missing front-to-back capital markets architecture | Full understanding of securities issuance, clearing & settlement (CCPs, CSDs, T2S), prime brokerage, collateral management, securities financing (repo/securities lending) | **High.** CIB's DNA is capital markets — this gap limits EA influence in the bank's most profitable division |
| **Risk Management Models** | No dedicated research on risk models | Credit risk (PD, LGD, EAD, IFRS 9), market risk (VaR, stressed VaR, FRTB), operational risk (AMA, SMA), counterparty credit risk (CVA, DVA, CCR), model risk management | **High.** Risk is the single biggest concern for bank architecture — every major system decision has risk implications |
| **Treasury & ALM** | Not covered | Asset-liability management (ALM), funds transfer pricing (FTP), liquidity coverage ratio (LCR), net stable funding ratio (NSFR), intraday liquidity, treasury management systems | **Medium-High.** Treasury is a key stakeholder for any bank-wide architectural change |
| **Basel III/IV Regulatory Capital** | Not covered | Capital adequacy (CET1, Tier 1, Tier 2), RWAs (credit, market, operational), leverage ratio, output floor, capital planning, ICAAP | **High.** Regulatory capital drives bank strategy — EA must operate within these constraints |
| **BCBS 239 (Risk Data Aggregation)** | Referenced implicitly in data governance but no dedicated deep-dive | Risk data aggregation principles, data lineage for regulatory reporting, BCBS 239 compliance architecture, remediation programs | **High.** A major regulatory initiative at European banks — directly relevant to his role at CACIB |
| **IFRS 9 / CECL** | Not covered | Expected credit loss modeling, staging (SICR), macroeconomic scenario generation, data architecture for ECL computation | **Medium.** Relevant for credit risk data architecture |

### 3.2 Emerging Technology Depth (Medium-High Priority)

| Gap | Current State | Target State | Impact |
|---|---|---|---|
| **Zero Trust Architecture (ZTA)** | OpenSCAP and vulnerability scanning covered; no ZTA | NIST SP 800-207 ZTA pillars, micro-segmentation, identity-aware proxies, beyondCorp/beyondProd, ZTA in banking | **High.** Regulatory push for zero trust in Singapore (MAS TRM) and global banking |
| **DevSecOps at Scale** | IaC best practices covered; no dedicated DevSecOps guide | Pipeline security scanning (SAST, DAST, SCA), secrets management, policy-as-code (OPA/Kyverno), supply chain security (SLSA, in-toto), shift-left in regulated environments | **Medium-High.** Critical for deploying GenAI solutions safely in banking |
| **FinOps at Scale** | Section in cloud providers guide; no dedicated deep-dive | Cloud cost governance, showback/chargeback, commitment discounts, spot/preemptible instances, cost optimization for K8s, FinOps lifecycle, FinOps in multi-cloud banking | **Medium-High.** Cloud costs are a growing board-level concern for banks |
| **Platform Engineering** | Not covered in dedicated form | Internal developer platforms (IDPs), Backstage (Spotify), platform as product, golden paths, developer portals, service catalogs, platform maturity models | **Medium-High.** Directly relevant for improving developer experience in a large bank |
| **Event-Driven Architecture at Scale** | Control-M/batch covered; event-driven patterns not deep-dived | Kafka event sourcing, CQRS, schema registry, async API design, event-driven microservices vs event-driven serverless, saga patterns, exactly-once semantics in banking | **Medium.** Increasingly important for real-time banking |
| **Service Mesh / Istio** | Mentioned in profile context; no dedicated guide | Istio architecture, mTLS, traffic management, observability, canary deployments, service mesh in regulated environments | **Medium.** Growing adoption in enterprise K8s deployments |

### 3.3 Soft Skills for Architects (Medium Priority)

| Gap | Current State | Target State | Impact |
|---|---|---|---|
| **Executive Presentation & Communication** | Storytelling guide covers narrative; but executive-level pitch architecture not specifically addressed | Executive-level communication frameworks (Pyramid Principle, Minto, BLUF), engaging C-level attention within 30 seconds, board-level deck design, handling Q&A with senior stakeholders | **High.** EA operates at C-level — this is the most important soft skill gap |
| **Negotiation for Architects** | Managing sideways touches conflict; no dedicated negotiation coverage | Principled negotiation (Harvard method), BATNA/ZOPA, multi-stakeholder negotiation, negotiating architecture trade-offs with business, vendor negotiation | **Medium-High.** Architects negotiate every day — scope, standards, timelines, resources |
| **Consulting Methodology** | No coverage | McKinsey/BCG problem-solving, hypothesis-driven consulting, issue trees, MECE frameworks, deliverable packaging, client relationship management | **Medium-High.** EA roles increasingly require consulting-style engagement with business units |
| **Influencing Without Authority** | Section in managing sideways guide | Deeper frameworks: Cohen-Bradford influence model, reciprocity/social capital, coalition building, technical authority vs organizational authority | **Medium.** Covered partially; needs depth specific to architect context |
| **Facilitation & Workshop Design** | Not covered | Design thinking facilitation, architecture workshop facilitation, whiteboarding with executives, collaborative decision-making (consent-based decision-making, RAPID, DACI) | **Medium.** Architects run architecture reviews, retrospectives, and design workshops |
| **Change Management** | Not covered | ADKAR, Kotter's 8 steps, Prosci methodology, organizational change for technology transformations, managing resistance | **Medium.** EA drives transformation — change management literacy is essential |

### 3.4 Strategic Thinking & Business Acumen (Medium Priority)

| Gap | Current State | Target State | Impact |
|---|---|---|---|
| **Business Case Development** | Not covered | Cost-benefit analysis (CBA), NPV/IRR/payback period, options-based business cases, quantifying architecture value, TCO vs total value of ownership (TVO) | **High.** Every major architecture initiative requires a business case to get funded |
| **TCO Modeling** | Not explicitly covered | Total Cost of Ownership frameworks (Gartner, Forrester), cloud vs on-prem TCO, migration TCO, 3-5 year cost projections, sensitivity analysis | **Medium-High.** Essential for build-vs-buy and cloud migration decisions in banking |
| **Vendor Evaluation Frameworks** | Selection criteria in individual tool guides; no unified framework | Structured vendor evaluation (RFP process, weighted scoring, proof of concept design), vendor lock-in risk assessment, total value assessment (not just cost) | **Medium-High.** Banks spend heavily on vendors — EA needs rigorous evaluation methodology |
| **Innovation Management** | Digital innovation guide exists but not dedicated to enterprise innovation | Innovation portfolio management (horizon model), stage-gate vs lean startup, innovation accounting, corporate venture/incubation models, innovation metrics | **Medium.** Relevant for GenAI innovation leadership at CACIB |
| **IT Investment Portfolio Management** | Not covered | Application portfolio rationalization (triage: retain, retire, replace, rehost), run/grow/transform spend analysis, IT budgeting cycles for architecture programs | **Medium.** EA oversees application portfolio decisions |
| **Architecture Governance at Scale** | TOGAF ADM governance covered theoretically; practical enterprise architecture board operation not addressed | Architecture Review Board (ARB) operations, architecture compliance reviews, exception process, architecture KPIs, maturity assessments (e.g., NASCIO, US EA Scorecard) | **Medium-High.** EA governance is where theory meets practice in a bank |

### 3.5 Certification Gaps

| Certification | Current State | Priority |
|---|---|---|
| **TOGAF 10 Certification** | Has comprehensive research but likely not certified | **IMMEDIATE.** Level 1 (Foundation) + Level 2 (Certified). Required or strongly preferred for EA roles. The research shows readiness — a 2-4 week prep + exam cycle is realistic |
| **AWS Solution Architect – Professional** | Deep AWS knowledge demonstrated in cloud guide | **SHORT-TERM.** Validates cloud architecture expertise. SAP (AWS) is the gold standard |
| **Azure Solutions Architect – Expert (AZ-305)** | Azure knowledge demonstrated | **SHORT-TERM.** CACIB likely runs a multi-cloud strategy; Azure is common in European banking |
| **GCP Professional Cloud Architect** | GCP knowledge demonstrated | **MEDIUM-TERM.** Lower priority unless CACIB has specific GCP strategy |
| **SAFe Architect (SAFe Practice Consultant / Architect)** | Extensive SAFe guide already written | **MEDIUM-TERM.** Valuable for scaled agile architecture governance in banking |
| **CISSP or CISM** | Security knowledge demonstrated | **MEDIUM-TERM.** Adds security governance credibility for EA roles in regulated environments |
| **SABSA (Security Architecture)** | Not researched | **LONG-TERM.** Niche but valuable for banking security architecture |

---

## 4. Prioritized Learning Roadmap

### 4.1 Immediate (Next 3 months) — Foundation & Credibility

| Priority | Focus Area | Why Now | Suggested Actions |
|---|---|---|---|
| **P1** | **TOGAF 10 Certification** | Opens doors to EA roles; research already done | Take Open Group TOGAF 10 training or self-study with the research guide. Target: both Level 1 and Level 2 within 8 weeks |
| **P1** | **Trade Finance Deep Dive** | Core CIB product; his bank's DNA | Research trade finance instruments (LC, SBLC, collections), trade finance technology (Contour, we.trade, TradeIX), and how trade finance systems integrate with core banking and payments |
| **P1** | **BCBS 239 & Risk Data Architecture** | Major regulatory program at European banks | Study BCBS 239 principles, data lineage for risk reporting, risk data aggregation architecture patterns, parallel with MAS TRM requirements. Essential context for his day job |
| **P2** | **Risk Management Fundamentals** | Every architecture decision interfaces with risk | Learn credit risk (PD/LGD/EAD, IFRS 9), market risk (VaR, FRTB), operational risk (loss data, RCSA), and how risk data flows through bank architecture |

### 4.2 Short-Term (3–6 months) — Deepening Technical Architecture

| Priority | Focus Area | Why Now | Suggested Actions |
|---|---|---|---|
| **P2** | **Zero Trust Architecture** | MAS TRM alignment; security is existential for banking | Study NIST SP 800-207, ZTA for financial services, micro-segmentation patterns, identity-aware proxy architectures. Write a dedicated research guide |
| **P2** | **Platform Engineering** | IDPs are the next wave of enterprise K8s; his OpenShift depth is a foundation | Research Backstage, platform as product, golden paths, developer portal architecture. Write a comparison guide |
| **P2** | **Capital Markets Architecture (Front-to-Back)** | CIB's core business | Research the full trade lifecycle (execution → confirmation → clearing → settlement → reporting), understand CCPs, CSDs, T2S, SWIFT, collateral management systems |
| **P2** | **AWS/Azure Architect Certification** | Cloud expertise is table stakes for EA | Choose one cloud cert based on CACIB's primary cloud strategy. AWS SAP (Professional) is the most vendor-respected |
| **P3** | **FinOps at Scale** | Cloud governance is a board-level concern | Research the FinOps Foundation framework, cloud cost optimization for K8s, showback models for banking. Extend the FinOps section in cloud_providers_guide.md into a full guide |

### 4.3 Medium-Term (6–12 months) — Influence & Leadership

| Priority | Focus Area | Why Now | Suggested Actions |
|---|---|---|---|
| **P2** | **Executive Communication & Pyramid Principle** | EA communicates strategy to C-level; this is the #1 soft skill gap | Study Barbara Minto's Pyramid Principle, practice writing 1-page executive briefs, learn board-level narrative construction. Write an "Architect's Guide to Executive Communication" |
| **P3** | **Business Case & TCO Modeling** | Every EA initiative must be justified in financial terms | Study business case methodology, NPV/IRR calculation, TCO frameworks (Gartner, Forrester), sensitivity analysis. Create a template for architecture business cases |
| **P3** | **Consulting Methodology** | EA roles increasingly resemble internal consulting | Study hypothesis-driven problem solving, issue trees, MECE frameworks, executive briefing design. Consider resources like "The McKinsey Way" or "Cracked it!" |
| **P3** | **Negotiation for Architects** | Influence without authority requires negotiation skills | Study Harvard Negotiation Project, BATNA/ZOPA, principled negotiation. Practice through architecture review governance scenarios |
| **P3** | **Basel III/IV Regulatory Capital** | Regulatory capital drives bank strategy | Study capital adequacy framework, RWA calculation (credit, market, operational), capital planning, ICAAP/ SREP, output floor. Write a dedicated guide |

### 4.4 Long-Term (12–18 months) — Strategic Differentiation

| Priority | Focus Area | Why Now | Suggested Actions |
|---|---|---|---|
| **P3** | **Innovation Management** | His GenAI focus is an innovation domain | Study innovation portfolio management (Three Horizons), stage-gate vs lean startup, corporate venture building, innovation metrics (adoption, ROI, time-to-market) |
| **P3** | **Architecture Governance & EA Maturity** | Transition from solution to enterprise architect | Research EA maturity models (US EA Scorecard, Gartner EAPM, NASCIO), architecture review board operations, compliance frameworks, EA metrics. Write a guide on "Building an EA Practice in Banking" |
| **P4** | **Event-Driven Architecture & Stream Processing** | Future-proofing for real-time banking | Deep-dive on Kafka event sourcing, CQRS, saga patterns, stream processing for fraud detection, real-time risk, transaction monitoring |
| **P4** | **DevSecOps in Regulated Banking** | Bridging speed and safety | Research policy-as-code (OPA Kyverno), compliance-as-code, SLSA framework, secure supply chain patterns for highly regulated environments. This ties together his security and DevOps knowledge |
| **P4** | **SAFe Architect Certification** | Valuable for scaled agile governance | Enroll in SAFe certification path. His existing SAFe guide demonstrates readiness |
| **P4** | **Industry Thought Leadership** | Differentiation for senior EA roles | Start publishing architecture content under CACIB brand. Convert research guides into conference talks, LinkedIn articles, or banking technology blog posts. Targets: Singapore FinTech Festival, local meetups, LinkedIn thought leadership |

---

## 5. Personalized Book Recommendations

### 5.1 Banking & Regulatory Domain

| Book | Author | Why for Jack |
|---|---|---|
| **The Principles of Banking** | Moorad Choudhry | Comprehensive foundation in banking operations, ALM, risk management, regulatory capital. Ideal for bridging the banking domain gap (gaps 3.1) |
| **Trade Finance: Principles and Practice** | Stephen Jones | The go-to text for trade finance instruments and processes. Directly applicable to CACIB's CIB context |
| **BCBS 239: A Guide to Risk Data Aggregation** | KPMG / Deloitte (practitioner guides) | Practical reference for risk data architecture compliance. No single canonical book — use practitioner/consultancy frameworks |
| **The Lombard Street Chronicles / Misunderstanding Financial Crises** | Various | Context on why risk management architecture matters — helps an architect understand the business drivers behind compliance requirements |
| **IFRS 9: Implementation Guidance** | PwC / EY guides | Practical reference for expected credit loss modeling and data architecture requirements |

### 5.2 Enterprise Architecture & Frameworks

| Book | Author | Why for Jack |
|---|---|---|
| **Enterprise Architecture as Strategy** | Ross, Weill & Robertson | The classic EA text. Jack's EA guide already cites it — reading the full book will deepen the framework-to-practice connection |
| **TOGAF 10 Study Guide** | The Open Group / Rachel Harrison | Exam prep for TOGAF 10 certification. Jack's research shows readiness — this book closes the certification gap |
| **A Practical Guide to Enterprise Architecture** | James McGovern et al. | Practical, real-world EA guidance (not just theory). Includes EA governance, architecture reviews, and political navigation |
| **Architecting for Scale** | Lee Atchison | Practical book on making architectural decisions that scale. Bridges the gap between solution architecture and EA thinking |
| **The BIAN Service Landscape** | BIAN e.V. | The official BIAN reference. Jack's BIAN guide is strong — the full service landscape document is the next level |

### 5.3 Soft Skills & Influence

| Book | Author | Why for Jack |
|---|---|---|
| **The Pyramid Principle** | Barbara Minto | The definitive text on executive communication. Every architect going for EA should master this. Directly addresses the biggest soft skill gap |
| **Getting to Yes** | Fisher, Ury & Patton | The foundational text on principled negotiation. Essential for stakeholder management without positional authority |
| **Influence Without Authority** | Cohen & Bradford | The classic on lateral influence. Complements Jack's managing sideways research with deeper frameworks |
| **Cracked It!** | Garard, Spaan & Garard | Consulting-style problem solving for non-consultants. Issue trees, MECE, hypothesis-driven analysis. Excellent for architects who need consulting methodology |
| **The McKinsey Way** | Ethan Rasiel | Insider's view of McKinsey problem-solving. Delivers consulting methodology in an accessible format |
| **Made to Stick** | Heath & Heath | Why some ideas survive and others die. Complements Jack's storytelling research with SUCCESs framework — ideal for making architecture recommendations memorable to executives |

### 5.4 Technology & Emerging Practice

| Book | Author | Why for Jack |
|---|---|---|
| **FinOps: Cloud Financial Operations** | FinOps Foundation | The definitive FinOps resource. Directly fills the FinOps gap with a structured framework |
| **Zero Trust Architecture: A Practitioner's Guide** | Kindervag & Lyons | Foundation for ZTA implementation. NIST SP 800-207 is the standard, but this provides practical implementation guidance |
| **Platform Engineering on Kubernetes** | Salatino et al. | Practical guide to building internal developer platforms on K8s. Jack's existing K8s/OpenShift depth makes this a natural next step |
| **Designing Event-Driven Systems** | Ben Stopford (Confluent) | Concise guide to event-driven architecture with Kafka. Practical patterns for banking use cases |
| **Team Topologies** | Skelton & Pais | Essential for organizing teams around architecture — aligns with Conway's Law thinking for EA. Bridges management research with architecture practice |

### 5.5 Strategy & Innovation

| Book | Author | Why for Jack |
|---|---|---|
| **Good Strategy Bad Strategy** | Richard Rumelt | The most practical strategy book for practitioners. Helps architects distinguish real strategy from vague aspirations |
| **The Innovator's Dilemma** | Clayton Christensen | Classic on innovation — essential for understanding why established banks struggle with GenAI adoption. Jack's AI adoption work would benefit from this framing |
| **Playing to Win** | Lafley & Martin | Strategy framework (where to play, how to win) from ex-P&G CEO. Excellent for developing the strategic thinking needed for EA roles |
| **The Hard Thing About Hard Things** | Ben Horowitz | Honest look at the challenges of leadership. Complements Jack's management guides with real-world CEO perspective |

---

## 6. Suggested Next Research Topics

These are prioritized to fill the most important gaps first, ordered by impact on the EA transition:

### Immediate Research Topics (Next Research Sprint)

| # | Topic | Gap Addressed | Why |
|---|---|---|---|
| 1 | **Trade Finance Systems & Architecture in CIB** | Banking depth (3.1) | Directly applicable to CACIB. Research trade finance platforms (Contour, we.trade, TradeIX, bank proprietary systems), instruments (LC, SBLC, collections, supply chain finance), and architecture patterns for trade finance integration with core banking/payments |
| 2 | **BCBS 239: Risk Data Aggregation & Reporting Architecture** | Banking depth (3.1) | Major regulatory program. Research the 14 BCBS 239 principles, data lineage for risk data, risk data warehouse patterns, BCBS 239 vs MAS TRM alignment, remediation architecture |
| 3 | **Credit Risk Architecture in Banking** | Banking depth (3.1) | Research credit risk systems architecture, IFRS 9 ECL data flows, credit decision engines, risk rating models integration, stress testing infrastructure |
| 4 | **Business Case & TCO Modeling for Architects** | Strategic thinking (3.4) | Build a practical framework for architecture business cases. Include templates for TCO comparison (cloud vs on-prem), NPV calculation, vendor evaluation scorecard — reusable in his day job |

### Short-Term Research Topics (Next 2-3 Sprints)

| # | Topic | Gap Addressed |
|---|---|---|
| 5 | **Zero Trust Architecture for Financial Services** | Emerging tech (3.2) |
| 6 | **Platform Engineering & Internal Developer Platforms** | Emerging tech (3.2) |
| 7 | **Capital Markets Front-to-Back Architecture** | Banking depth (3.1) |
| 8 | **DevSecOps Compliance-as-Code for Regulated Banking** | Emerging tech (3.2) |
| 9 | **FinOps Cloud Cost Governance in Multi-Cloud Banking** | Emerging tech (3.2) |

### Medium-Term Research Topics

| # | Topic | Gap Addressed |
|---|---|---|
| 10 | **Architect's Guide to Executive Communication** | Soft skills (3.3) |
| 11 | **Basel IV / FRTB Architecture Implications** | Banking depth (3.1) |
| 12 | **Enterprise Architecture Governance in Practice** | Strategic thinking (3.4) |
| 13 | **Building an EA Practice from Solution Architecture** | Career transition |
| 14 | **Event-Driven Architecture for Real-Time Banking** | Emerging tech (3.2) |

---

## 7. Summary Assessment

### Where Jack Excels (Differentiators)

- **Exceptional breadth and depth in research.** Few practicing architects invest this deeply in structured learning across technology, management, and EA.
- **AI/ML & LLMOps depth** is genuinely advanced — this is a rare skill that positions him well for the intersection of AI architecture and enterprise architecture.
- **Management/leadership knowledge** is significantly ahead of typical architects, who often neglect this dimension until reaching senior levels.
- **EA framework knowledge** (TOGAF, BIAN, Capability Engineering) is already at EA-level depth — unusual for someone currently in a Solution Architect role.

### Where Jack Needs Focus

1. **Banking domain specificity.** The gap between "knows banking processes" and "understands trade finance architecture" or "can discuss FRTB implications" is where credibility with banking domain experts lives. This is the highest-impact gap for his CIB context.

2. **Executive presence.** Writing about storytelling is different from standing in front of a Managing Director and communicating architecture trade-offs in 5 minutes. The Pyramid Principle and consulting methodology will bridge this.

3. **Credentials.** TOGAF certification is table stakes. Cloud architect certifications add weight. Both are knowledge areas he already has — it's the exam investment that's missing.

4. **Financial decision-making.** Business cases, TCO, vendor evaluation — these are the language of business stakeholders. Architecture decisions without financial framing don't get funded.

### Jack's Unfair Advantage

Jack's combination of **deep GenAI/LLM expertise + EA framework knowledge + management/leadership literacy** is genuinely rare. As banks shift from "what is AI?" to "how do we architect AI governance, platform, and integration at enterprise scale?", someone who can speak both EA framework language and LLM architecture language is uniquely positioned. The question is whether he can complement this with enough banking domain depth and executive communication skill to be taken seriously at C-level.

### Recommended 18-Month Roadmap Summary

```
Month 1-3:   TOGAF certification + Trade Finance + BCBS 239 + Risk fundamentals
Month 4-6:   Zero Trust guide + Cloud certification + Platform Engineering guide
Month 6-9:   Executive communication mastery + Business case/TCO + Capital markets
Month 9-12:  Negotiation + Consulting methodology + Basel IV/FRTB
Month 12-18: Innovation management + Governance maturity + Industry thought leadership
```

---

## 8. Appendix: Repository Content Audit Summary

| Category | File Count | Average Depth | Overall Assessment |
|---|---|---|---|
| Cloud & Infrastructure | ~12 | Very High | Ready for EA-level |
| K8s/OpenShift | ~6 | Very High | Ready for EA-level |
| AI/ML & LLMs | ~12 | Exceptional | Competitive advantage |
| MLOps & Data | ~20 | Very High | Ready for EA-level |
| Security & Compliance | ~5 | Intermediate | Needs ZTA depth |
| Management & Leadership | ~7 | Very High | Ahead of peers |
| EA Frameworks (TOGAF, BIAN, Capability) | ~4 | Exceptional | Already EA-level |
| Banking & Finance | ~6 | High | Needs domain specificity |
| Education, Personal, Singapore | ~20 | High | Unique local context |
| **Total** | **~100+** | **Very High avg.** | **Strong foundation** |

---

*This gap analysis is based on the contents of github.com/jackliusr/research as of July 2026. It evaluates demonstrated knowledge (what has been researched and written about) rather than work experience or certification status, which may differ. The analysis is intended to guide focused learning investment for the transition from Solution Architect to Enterprise Architect in a banking context.*
