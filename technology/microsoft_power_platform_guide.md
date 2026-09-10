# Microsoft Power Platform: The Low-Code Lever — Apps, Automate, BI, Pages, Dataverse, Copilot Studio, Governance, and the Banking Angle

**Abstract.** This guide maps Microsoft Power Platform — the low-code/no-code suite that Microsoft presents as one platform for building *apps* (Power Apps), *automations* (Power Automate), *analytics* (Power BI), *external websites* (Power Pages), and *AI agents* (Copilot Studio), over a shared data layer (Microsoft Dataverse) and a shared connector fabric. It frames what the platform *is* and why it is sold as a suite rather than as products, walks the 2015–2023 component timeline (Power BI, PowerApps and Flow, the Flow→Power Automate and Power Virtual Agents→Copilot Studio renames, the Common Data Service→Dataverse rename, and the 2022 birth of Power Pages), and situates the suite beside its two neighbours — **Dynamics 365**, whose applications run on the same Dataverse, and **Azure**, the substrate beneath it. It then takes each component in turn: canvas versus model-driven apps; cloud flows versus desktop flows and the entered world of RPA; semantic models and reports and the relationship to **Microsoft Fabric**; Dataverse's tables, Common Data Model lineage, security roles, and business rules; Copilot Studio's agents, topics, knowledge sources, and channels; and AI Builder alongside the connector ecosystem whose "1,000+ connectors" figure this guide flags as marketing framing. Two chapters carry the governance spine: the **environment strategy**, data loss prevention policies, the **Center of Excellence (CoE) Starter Kit** and its migration into the Power Platform admin center, and application lifecycle management through solutions and deployment pipelines — plus the licensing facts, flagged as they change. A **banking angle** follows: citizen-developer governance and the shadow-IT risk, regulated-industry adoption evidenced by public HSBC and Axis Bank stories, and the security controls a bank needs. The guide closes with a **Cymbal Bank worked example** — a governance rollout for the bank's citizen-developer program, covering the environment strategy, the DLP policy design, CoE inventory adoption, and ALM discipline — and the outcomes and governance metrics. Facts are flagged ✅ (verified this pass against primary sources), ⚠ (approximate, vendor-claimed, or single-source), ⚠-knowledge (well-established industry knowledge not re-verified this pass), or ❌ (could not be verified); a claims-audit table and a "What Could Not Be Verified" section keep the honest ledger.

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Technology / Low-Code & Citizen-Development Platform Landscape — Power Apps, Power Automate, Power BI, Power Pages, Dataverse, Copilot Studio, AI Builder, Connectors, Governance and ALM, with a Banking Angle  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Primary Sources:** learn.microsoft.com (the Power Platform, Power Apps, Power Automate, Power BI, Power Pages, Copilot Studio, AI Builder, Connectors, admin, and ALM documentation hubs), the Microsoft product/legal docs (the Power Platform Licensing Guide and the licensing overview page), the Microsoft Release Planner/roadmap, Microsoft Learn training, the Microsoft Copilot Studio and Power Platform blogs, microsoft.com customer stories (HSBC, Axis Bank), the Microsoft Ignite 2016 and 2023 announcements, and Wikipedia as cross-check. NOTE: this pass had live web access; facts were checked against primary sources on 2026-09-10 where possible; anything not verified is flagged ⚠ or ❌ and logged in §12.  
> **Last Updated:** September 2026  
> **Companion guides (repo-relative links):** Enterprise-software genre and the Microsoft Dynamics 365 Finance lineage → [financial_management_systems_guide.md](financial_management_systems_guide.md) §5.3; middleware, connectors, and the integration taxonomy → [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) §3; BI, analytics, and the Fabric relationship → [advanced_analytics_solutions_guide.md](advanced_analytics_solutions_guide.md), [data/data_fabric_guide.md](data/data_fabric_guide.md); enterprise data platforms → [data/enterprise_data_platforms_guide.md](data/enterprise_data_platforms_guide.md); AI agent platforms and Copilot Studio → [ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md), [ai_llm/ai_agent_platform_selection_guide.md](ai_llm/ai_agent_platform_selection_guide.md) §1.2.3, [ai_llm/autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md); enterprise architecture and the platform operating model → [architecture/enterprise_architecture_guide.md](architecture/enterprise_architecture_guide.md); TCO modeling → [tco_modeling_guide.md](tco_modeling_guide.md); enterprise risk and the three lines of defence → [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md); MAS technology-risk expectations → [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md); treasury/ALM depth → [../banking/treasury_alm_guide.md](../banking/treasury_alm_guide.md); banks in Singapore → [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md); AI/GenAI banking compliance → [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md); vendor and IT-strategy discipline → [../management/vendor_management_guide.md](../management/vendor_management_guide.md), [../management/it_strategy_guide.md](../management/it_strategy_guide.md); analytics management → [../management/analytics_management_guide.md](../management/analytics_management_guide.md); Singapore context → [../singapore/sg_gdp_industry_distribution.md](../singapore/sg_gdp_industry_distribution.md)

---

**How to read this document.** §1–§3 are the framing: what Power Platform is and its component map (§1), the origins and component timeline (§2), and the relationship to Dynamics 365 and Azure (§3). §4–§8 are the components: Power Apps (§4), Power Automate (§5), Power BI (§6), Power Pages and Dataverse (§7), and Copilot Studio with AI Builder and the connectors (§8). §9 is governance and ALM — the chapter that decides whether a Power Platform estate is an asset or a liability. §10 is the banking angle, §11 the Cymbal Bank worked example. §12 is the combined claims-audit and verification ledger, §13 the glossary, and §14 the closing. **Completeness conventions:** ✅ = verified this pass against a primary or named source; ⚠ = approximate / vendor claim / single secondary source; ⚠-knowledge = well-established industry knowledge not re-verified this pass; ❌ = could not be verified. Cross-references follow repo convention: same-directory guides by plain filename, `../banking/...` for banking guides, `../technology/...` for technology guides, `data/...` and `ai_llm/...` for those sub-folders, `../management/...` and `../singapore/...` for those folders. No fact here is fabricated; where this pass could not confirm a claim, the claim is flagged rather than asserted.

---

## Table of Contents

1. [What the Power Platform Is: The Suite and Its Component Map](#1-what-the-power-platform-is-the-suite-and-its-component-map)
   - 1.1 [The Platform, Defined](#11-the-platform-defined)
   - 1.2 [The Component Map at a Glance](#12-the-component-map-at-a-glance)
   - 1.3 [Why Microsoft Sells It as a Suite](#13-why-microsoft-sells-it-as-a-suite)
2. [Origins and History: The 2015–2023 Component Timeline](#2-origins-and-history-the-20152023-component-timeline)
   - 2.1 [Power BI: From Project Crescent to a Platform Cornerstone](#21-power-bi-from-project-crescent-to-a-platform-cornerstone)
   - 2.2 [PowerApps and Flow: Generally Available 1 November 2016](#22-powerapps-and-flow-generally-available-1-november-2016)
   - 2.3 [Flow Becomes Power Automate (2019)](#23-flow-becomes-power-automate-2019)
   - 2.4 [Common Data Service Becomes Dataverse (2020)](#24-common-data-service-becomes-dataverse-2020)
   - 2.5 [Power Pages (2022)](#25-power-pages-2022)
   - 2.6 [Power Virtual Agents Becomes Copilot Studio (2023)](#26-power-virtual-agents-becomes-copilot-studio-2023)
   - 2.7 [The Timeline at a Glance](#27-the-timeline-at-a-glance)
3. [Power Platform Beside Dynamics 365 and Azure](#3-power-platform-beside-dynamics-365-and-azure)
   - 3.1 [The Dynamics 365 Relationship](#31-the-dynamics-365-relationship)
   - 3.2 [Dataverse as the Common Data Layer](#32-dataverse-as-the-common-data-layer)
   - 3.3 [The Azure Foundation](#33-the-azure-foundation)
4. [Power Apps: Canvas, Model-Driven, and the App Types](#4-power-apps-canvas-model-driven-and-the-app-types)
   - 4.1 [Canvas Apps](#41-canvas-apps)
   - 4.2 [Model-Driven Apps](#42-model-driven-apps)
   - 4.3 [The App Type Table](#43-the-app-type-table)
5. [Power Automate: Cloud Flows, Desktop Flows, and RPA](#5-power-automate-cloud-flows-desktop-flows-and-rpa)
   - 5.1 [Cloud Flows: The Trigger and Action Model](#51-cloud-flows-the-trigger-and-action-model)
   - 5.2 [Desktop Flows and Robotic Process Automation](#52-desktop-flows-and-robotic-process-automation)
   - 5.3 [Attended versus Unattended RPA](#53-attended-versus-unattended-rpa)
   - 5.4 [Process Mining and Task Mining](#54-process-mining-and-task-mining)
6. [Power BI: Semantic Models, Reports, and the Fabric Relationship](#6-power-bi-semantic-models-reports-and-the-fabric-relationship)
   - 6.1 [Semantic Models and Reports](#61-semantic-models-and-reports)
   - 6.2 [The Microsoft Fabric Relationship](#62-the-microsoft-fabric-relationship)
   - 6.3 [Self-Service Analytics and Its Governance Burden](#63-self-service-analytics-and-its-governance-burden)
7. [Power Pages and Dataverse: The External Surface and the Data Platform](#7-power-pages-and-dataverse-the-external-surface-and-the-data-platform)
   - 7.1 [Power Pages: External-Facing Low-Code Websites](#71-power-pages-external-facing-low-code-websites)
   - 7.2 [Dataverse: Tables and the Common Data Model Lineage](#72-dataverse-tables-and-the-common-data-model-lineage)
   - 7.3 [The Dataverse Security Model](#73-the-dataverse-security-model)
   - 7.4 [Business Rules and Logic](#74-business-rules-and-logic)
8. [Copilot Studio, AI Builder, and the Connectors](#8-copilot-studio-ai-builder-and-the-connectors)
   - 8.1 [Copilot Studio: The Agent-Building Platform](#81-copilot-studio-the-agent-building-platform)
   - 8.2 [Agents, Topics, Knowledge, and Channels](#82-agents-topics-knowledge-and-channels)
   - 8.3 [Autonomous Agents and the Harness Model](#83-autonomous-agents-and-the-harness-model)
   - 8.4 [AI Builder](#84-ai-builder)
   - 8.5 [The Connectors and the "1,000+ Connectors" Claim](#85-the-connectors-and-the-1000-connectors-claim)
9. [Governance and ALM: The Chapter That Decides the Outcome](#9-governance-and-alm-the-chapter-that-decides-the-outcome)
   - 9.1 [The Environment Strategy](#91-the-environment-strategy)
   - 9.2 [The Default Environment](#92-the-default-environment)
   - 9.3 [Managed Environments and Environment Groups](#93-managed-environments-and-environment-groups)
   - 9.4 [Data Loss Prevention Policies](#94-data-loss-prevention-policies)
   - 9.5 [The CoE Starter Kit and Its Transition](#95-the-coe-starter-kit-and-its-transition)
   - 9.6 [ALM: Solutions, Managed versus Unmanaged, and Pipelines](#96-alm-solutions-managed-versus-unmanaged-and-pipelines)
   - 9.7 [Licensing: The Public Facts](#97-licensing-the-public-facts)
10. [The Banking Angle: Citizen Developers Under Supervision](#10-the-banking-angle-citizen-developers-under-supervision)
    - 10.1 [Citizen-Developer Governance and the Shadow-IT Risk](#101-citizen-developer-governance-and-the-shadow-it-risk)
    - 10.2 [Regulated-Industry Adoption: The Public Evidence](#102-regulated-industry-adoption-the-public-evidence)
    - 10.3 [The Security Controls a Bank Needs](#103-the-security-controls-a-bank-needs)
11. [Cymbal Bank Worked Example: A Power Platform Governance Rollout](#11-cymbal-bank-worked-example-a-power-platform-governance-rollout)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [The Environment Strategy](#112-the-environment-strategy)
    - 11.3 [The DLP Policy Design](#113-the-dlp-policy-design)
    - 11.4 [CoE Starter Kit Adoption](#114-coe-starter-kit-adoption)
    - 11.5 [The ALM Discipline](#115-the-alm-discipline)
    - 11.6 [Outcomes and Governance Metrics](#116-outcomes-and-governance-metrics)
12. [Claims Audit and What Could Not Be Verified](#12-claims-audit-and-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Closing](#14-closing)

---

## 1. What the Power Platform Is: The Suite and Its Component Map

### 1.1 The Platform, Defined

**Verified (✅):** Microsoft's own documentation hub describes the platform as the place to "build and manage **agents, apps, automations, analytics, and websites**, and how to govern and extend your solutions" (learn.microsoft.com/power-platform, retrieved 2026-09-10). Its products are stated plainly: "Build agents and workflows with Copilot Studio, apps with Power Apps, automations with Power Automate, analytics with Power BI, and websites with Power Pages." Its *capabilities* — as distinct from its *products* — are listed as "AI Builder, connectors, Copilot and generative AI, Microsoft Dataverse, and the low-code Power Fx language." The Wikipedia cross-check frames the same object as "a collection of low-code development tools that allows users to build custom business applications, automate workflows, and analyze data," with integration to GitHub, Azure, Dynamics 365, and Teams (Wikipedia, retrieved 2026-09-10).

The important structural fact is the split between **products** (the five named experiences you buy and use) and **capabilities** (the shared services beneath them: the data layer, the connectors, the AI models, and the formula language). Power Fx — the low-code formula language Microsoft built for the platform (Wikipedia, cross-checked) — is the connective syntax that makes canvas apps, Power Pages custom logic, and Dataverse low-code plug-ins feel like one dialect rather than five products with five dialects.

### 1.2 The Component Map at a Glance

Each component is verified at its own product documentation this pass; the "what it is for" column is the verifier's synthesis of Microsoft's own framing, not a marketing claim.

| Component | What it is | What it is for | Verified framing |
| --- | --- | --- | --- |
| **Power Apps** | A low-code app-building service | Rapidly build web and mobile business apps without writing code | "quickly build low-code apps that modernize processes" (learn.microsoft.com/power-apps ✅) |
| **Power Automate** | A workflow-automation service | Create automated workflows between apps and services; adds RPA | "streamline your business processes and automate repetitive tasks" (learn.microsoft.com/power-automate ✅) |
| **Power BI** | A business-analytics platform | Connect, model, visualise, and share data | "Microsoft's business analytics platform" (learn.microsoft.com/power-bi ✅) |
| **Power Pages** | A low-code SaaS website platform | "creating, hosting, and administering modern external-facing business websites" | Stated verbatim in the docs (learn.microsoft.com/power-pages ✅) |
| **Microsoft Dataverse** | A cloud data service | "securely store and manage data that's used by business applications" | Stated verbatim (learn.microsoft.com/power-apps/maker/data-platform ✅) |
| **Copilot Studio** | A low-code agent studio | "build and manage AI-powered agents and workflows" | Stated verbatim (learn.microsoft.com/microsoft-copilot-studio ✅) |
| **AI Builder** | A Power Platform capability | "bring the power of Microsoft AI to your organization, without the need for coding or data science skills" | Stated verbatim (learn.microsoft.com/ai-builder ✅) |
| **Connectors** | The integration fabric | Let services "talk to" Copilot Studio, Power Automate, Power Apps, and Logic Apps | Stated verbatim (learn.microsoft.com/connectors ✅) |

### 1.3 Why Microsoft Sells It as a Suite

Three properties make the suite framing more than packaging:

- **One data layer.** Dataverse stores the data for Power Apps, Power Automate, Power BI, Power Pages, and Copilot Studio *and* for Dynamics 365 applications — so an app, a flow, a report, and an agent can all read the same tables without an integration project (✅, learn.microsoft.com Dataverse intro).
- **One connector fabric.** The same connectors serve Power Apps, Power Automate, Copilot Studio, and Azure Logic Apps; Microsoft states "**all connectors are agent ready**" (✅, connectors overview). A connector built once is consumable across the suite.
- **One governance surface.** Environments, data policies, managed environments, and the Power Platform admin center govern every product in one place (✅, learn.microsoft.com/power-platform/admin). This is the difference between a *tool* and a *platform* — the ability to apply a single policy to everything a maker builds, whatever product they built it in.

The counterweight is equally real: one platform means one blast radius. The governance chapter (§9) is where the suite's promise is either kept or broken.

---

## 2. Origins and History: The 2015–2023 Component Timeline

### 2.1 Power BI: From Project Crescent to a Platform Cornerstone

**Verified (✅):** Power BI's lineage is a Microsoft-internal product: it began as **Power Pivot and Power Query** in Excel, conceived by Thierry D'Hers and Amir Netz of the SQL Server Reporting Services team, and was designed as **"Project Crescent"** by Ron George in the summer of **2010**; Project Crescent was first publicly downloadable on **11 July 2011**, bundled with SQL Server "Denali" (Wikipedia, citing the sources on the Power BI article). It was renamed **Power BI** and unveiled by Microsoft in **September 2013** as "Power BI for Office 365," and **first released to the general public on 24 July 2015** (Wikipedia). Microsoft also acquired the Canadian mobile-BI company **Datazen on 14 April 2015** to round out mobile capabilities (Wikipedia). Useful nuance: the Wikipedia infobox dates the *release* to 11 July 2011 (Project Crescent) while the *general public release* of Power BI is 24 July 2015 — the two dates answer different questions, and the guide records both.

### 2.2 PowerApps and Flow: Generally Available 1 November 2016

**Verified (✅):** Microsoft announced the **general availability of Microsoft PowerApps and Microsoft Flow** in an Official Microsoft Blog post dated **31 October 2016**, effective "beginning Tuesday" — i.e., **1 November 2016** — "available to users around the world and in 42 languages." The same post introduced the **Common Data Service** as "a unifying backbone for your business data," storing data "in a secure Microsoft Azure-hosted database," and framed PowerApps, Flow, and Power BI as the **"power trio."** The public preview had launched in **April 2016**; the GA post reports "over 160,000 users from 71,000 organizations in 145 countries" had created apps and workflows during the preview (✅, blogs.microsoft.com 2016-10-31). The Power Automate Wikipedia article independently dates the GA to **1 November 2016** (✅ cross-check). This is the founding moment of the modern platform.

### 2.3 Flow Becomes Power Automate (2019)

**Verified (✅):** **On 4 November 2019, Microsoft announced the rebranding of Microsoft Flow to Microsoft Power Automate** and its inclusion in Microsoft Power Platform, "with a shift from solely workflows to also include business processes," alongside new **robotic process automation (RPA)** capabilities (Wikipedia, citing the Microsoft Ignite 2019 announcement). The product page confirms the current name and positions Power Automate as the automation workload of the suite (✅, learn.microsoft.com/power-automate). The lineage matters for anyone reading older contracts, licences, or runbooks: "Flow" and "Power Automate" are the same product across the 2019 line.

### 2.4 Common Data Service Becomes Dataverse (2020)

**Verified (✅):** **Microsoft Dataverse was "formerly known as Microsoft Common Data Service until November 2020"** (Wikipedia, citing Microsoft Docs). Microsoft's own documentation records the companion **terminology change**: "effective **November 2020** we're updating some terminology in Dataverse" — *entity/entities → table/tables*, *field/attribute → column/columns*, *record/records → row/rows* (✅, learn.microsoft.com Dataverse intro). The rename is not cosmetic: it signals that the data layer is now the platform's foundation rather than a PowerApps-only service, and it explains why older Dataverse artefacts (the `CreateEntityRequest` API message, for example) still carry the legacy vocabulary — Microsoft states the API and message names "won't change" (✅).

### 2.5 Power Pages (2022)

**Verified (✅):** Power Pages is "the newest member of the Microsoft Power Platform family" and is a "secure, enterprise-grade, low-code software as a service (SaaS) platform for creating, hosting, and administering modern external-facing business websites" (learn.microsoft.com/power-pages). Wikipedia records that it was **"formerly part of Power Apps as 'Power Apps Portals' until 2022"** and cites the Microsoft announcement "Announcing Microsoft Power Pages: Build secure, low-code websites" (✅). The 2022 date is thus verified to the year; the guide flags the exact launch *day* as not pinned to a primary source this pass (⚠ on the day, ✅ on the year).

### 2.6 Power Virtual Agents Becomes Copilot Studio (2023)

**Verified (✅):** Microsoft's Copilot blog states that "as of **November 15th**, Power Virtual Agents capabilities are now part of Microsoft's new product, **Microsoft Copilot Studio** … the Power Virtual Agents name will no longer be used" (microsoft.com Copilot blog). This was the **Microsoft Ignite 2023** announcement (November 15–16, 2023). The Wikipedia cross-check confirms Copilot Studio "(formerly Power Virtual Agents)" (✅). The rebrand was more than a name change — Copilot Studio was built on the Power Virtual Agents foundation *and* the broader Microsoft conversational-AI stack — but for governance and licensing purposes the important fact is that **Power Virtual Agents ceased to be a distinct product name on 15 November 2023**.

### 2.7 The Timeline at a Glance

| Year / date | Event | Status |
| --- | --- | --- |
| Summer 2010 | "Project Crescent" designed; precursor of Power BI | ✅ (Wikipedia) |
| 11 July 2011 | Project Crescent public download (with SQL Server Denali) | ✅ (Wikipedia) |
| September 2013 | Renamed Power BI, unveiled as "Power BI for Office 365" | ✅ (Wikipedia) |
| 14 April 2015 | Microsoft acquires Datazen for mobile BI | ✅ (Wikipedia) |
| 24 July 2015 | Power BI released to the general public | ✅ (Wikipedia) |
| April 2016 | PowerApps and Flow public preview | ✅ (Microsoft blog) |
| 1 November 2016 | PowerApps and Flow general availability; Common Data Service announced | ✅ (Microsoft blog; Wikipedia) |
| 4 November 2019 | Flow rebranded Power Automate; RPA added; Ignite 2019 | ✅ (Wikipedia; Ignite) |
| 2020 | Microsoft acquires Softomotive (RPA: ProcessRobot, WinAutomation) | ✅ (Wikipedia) |
| November 2020 | Common Data Service renamed Dataverse; entity→table terminology | ✅ (Wikipedia; Microsoft docs) |
| 2022 | Power Pages launched; Microsoft acquires Minit (process mining) | ✅ for Power Pages (regex year); ✅ for Minit |
| 15 November 2023 | Power Virtual Agents folded into Microsoft Copilot Studio (Ignite 2023) | ✅ (Microsoft Copilot blog) |

---

## 3. Power Platform Beside Dynamics 365 and Azure

### 3.1 The Dynamics 365 Relationship

Microsoft states plainly that "**Dynamics 365 applications … also use Dataverse to store and secure the data they use**," which is what lets a maker "build apps by using Power Apps and Dataverse directly against your core business data … **without the need for integration**" (✅, learn.microsoft.com Dataverse intro). The relationship is therefore a shared-data relationship, not a subordination: Power Platform sits **beside** Dynamics 365, over the same Dataverse, so that a Power Apps canvas app can read a Dynamics 365 Sales table and a Power Automate flow can act on it. The licensing corollary is stated on the licensing page: "**Qualifying Dynamics 365 licenses include Power Apps and Power Automate use rights for extending and customizing the licensed Dynamics 365 application**," and "these rights must be used in the context of the licensed Dynamics 365 application" (✅, licensing overview). That restriction — *in the context of* — is the commercial seam a buyer must respect. The Dynamics 365 Finance and Operations lineage, and the enterprise-software genre Power Platform sits inside, are developed in the FMS guide and **not re-derived here** ([financial_management_systems_guide.md](financial_management_systems_guide.md) §5.3).

**Verified nuance (✅):** for the **Finance and Operations** apps specifically, Microsoft notes that "Finance and Operations apps currently require the configuration of the **Data Integrator** to make your business data … available in Dataverse" — i.e., the seamless "without integration" claim holds natively for the customer-engagement apps (Sales, Customer Service, Human Resources) but requires a configured integrator for the ERP-side apps (learn.microsoft.com Dataverse intro).

### 3.2 Dataverse as the Common Data Layer

The load-bearing sentence for architects: Dataverse "is a data storage and management engine serving as a foundation for Microsoft's Power Platform, Office 365 and Dynamics 365 apps," and "**it decouples the data from the application**" (Wikipedia, cross-checked against the Microsoft docs). Decoupling data from application is the property that makes the suite coherent: the same table can be the source of a canvas app, the trigger of a flow, the semantic layer of a report, and the knowledge source of an agent. The detailed anatomy — tables, columns, security roles, business rules, capacity — is in §7.

### 3.3 The Azure Foundation

The platform is hosted on and composable with Azure. Microsoft's GA post framed the platform as "backed by the full power of Microsoft Azure" and "hosted on Azure," with an explicit collaboration story: "a developer might build an **Azure Function** using Java or Python that can serve as a building block used by a PowerApps creator" (✅, blogs.microsoft.com). Today the seams run deeper — **Azure Logic Apps** is described as "the workflow engine for Power Automate," sharing the connector ecosystem (✅, connectors overview), and Power Apps offers an "**Azure Synapse Link**" path to export data to the data lake (listed in the Power Apps docs hub). This is the integration-architecture context that the middleware guide develops as the iPaaS category ([enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) §3): Power Automate is, functionally, Microsoft's low-code iPaaS lane inside a larger Azure integration estate.

---

## 4. Power Apps: Canvas, Model-Driven, and the App Types

Power Apps is the app-building arm of the suite, and its central design decision is a fork: build a **canvas app** (pixel-level control, any data source) or a **model-driven app** (data-model-first, Dataverse-only, mostly generated UI). Microsoft documents both, and the distinction is the single most consequential choice a maker makes.

### 4.1 Canvas Apps

**Verified (✅):** Microsoft's definition is direct: "**Canvas apps are custom business applications you can create in Microsoft Power Apps without writing code.** Think of a canvas as a blank design surface where you can drag and drop components to build exactly the user interface you need" (learn.microsoft.com/power-apps/maker/canvas-apps). The verified characteristics:

- **Data:** canvas apps "connect to data from hundreds of sources, including Microsoft 365, Dataverse, SharePoint, and other data sources" (✅ docs).
- **Design:** the maker manipulates control properties using **Power Fx expressions** — the low-code formula language (✅, and confirmed by the model-driven comparison table).
- **Deployment surface:** run in a browser, on Windows, on iOS/Android, and embedded in SharePoint, Power BI, and Teams (✅ docs).
- **Responsiveness:** "only responsive if designed in this way" (✅, comparison table) — canvas apps do not become responsive by accident.
- **Migration:** "potentially complex given that the datasources might need to be updated" (✅, comparison table).

The honest framing: canvas apps are the fastest path from "this process lives in a spreadsheet" to "this process lives in an app," and the flexibility that makes them fast is also why they need solution discipline (§9.6) and coding guidelines (Microsoft publishes Power Apps coding guidelines — ✅ listed in the docs hub).

### 4.2 Model-Driven Apps

**Verified (✅):** "**Model-driven app design is an approach that focuses on adding components such as forms, views, charts, and dashboards to tables using an app designer tool**" (learn.microsoft.com/power-apps/maker/model-driven-apps). The essential constraints and benefits, verbatim from the docs:

- **Dataverse-only.** "Without a data model housed within Microsoft Dataverse, you can't create a model-driven app" (✅). This is the hard boundary between the two app types.
- **Process-driven fit.** Model-driven apps "are especially well suited to process driven apps that are data dense and make it easy for users to move between related records," and Microsoft names **bank member relationships** among its own examples of a good fit (✅ docs).
- **Generated experience.** "the experience is consistent across all model-driven apps" because "much of the user interface is determined for you," driven by the components added (✅).
- **Inherited quality.** Apps are "**accessible** and **responsive** automatically," and "migrating apps between development, test, and production environments is relatively straightforward by using solutions" (✅).

Model-driven apps are the disciplined sibling: less design freedom, more governance for free, and a same-shape experience that reduces training load across a large user base.

### 4.3 The App Type Table

Microsoft publishes the comparison directly; this table reproduces its verified rows (✅, learn.microsoft.com model-driven app overview).

| Category | Model-driven apps | Canvas apps |
| --- | --- | --- |
| **Data platform** | Dataverse only | Dataverse + many others using connectors |
| **Design experience** | No-code component-focused design | Manipulation of control properties using Power Fx expressions |
| **UI control** | Limited, predominantly customization | Full control |
| **App consistency** | High — differs predominantly by chosen tables and views | Often low, given the significant control designers have |
| **Migration between environments** | Simple | Potentially complex given that the data sources might need to be updated |
| **Speed of creation** | Rapid | Relative to the complexity of the design |
| **Responsive** | Automatically responsive | Only responsive if designed in this way |
| **Navigation through relationships** | Automatic, provided relationships exist | Only where designed and applied using Power Fx formulas |
| **Accessibility features** | Built in | Designed into the app |

There is a third, newer type — **code apps**, for professional developers who want to build with code against the platform (✅ listed in the Power Apps docs hub). A fourth, the **Plan designer**, generates a plan (business problem, roles, stories, data model, technologies) before an app is built (✅ docs hub), and an AI-conversation path lets makers "build apps through conversation" (✅ docs hub). These matter for the governance story: the lower the barrier to creation, the more important the environments and DLP policies that contain it (§9).

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [financial_management_systems_guide.md](financial_management_systems_guide.md) §5.3 | Technology/ | The Dynamics 365 lineage Power Apps extends |
| [architecture/enterprise_architecture_guide.md](architecture/enterprise_architecture_guide.md) | Technology/ | The application-landscape context citizen apps join |
| [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) | Banking/ | The cores a low-code app must sit beside, not inside |

---

## 5. Power Automate: Cloud Flows, Desktop Flows, and RPA

### 5.1 Cloud Flows: The Trigger and Action Model

**Verified (✅):** Microsoft names **three types of flows**: "**Cloud flows** — Create a cloud flow when you want your automation to be triggered either **automatically, instantly, or via a schedule**"; "**Desktop flows** — Use desktop flows to automate tasks on the web or the desktop"; and "**Generative actions (preview)** — specify only the *intent* of the action and then have the AI choose the right set of actions in the right order" (learn.microsoft.com/power-automate/flow-types). The Wikipedia cross-check lists the classic taxonomy as automated flows (event-triggered), instant flows (user-triggered, often via button), scheduled flows, business process flows, and desktop flows (✅).

The flow model is **trigger → actions**: a trigger fires (a new email, a new Dataverse row, a schedule, an HTTP request), and a sequence of actions executes, each action usually being a **connector operation** (§8.5). This is the same mental model as any workflow engine, and it is why Power Automate belongs in the integration taxonomy of the middleware guide ([enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) §3) — it is a low-code orchestration layer, and the durability/ordering questions that guide raises apply to it.

**Verified (✅):** Power Automate documents a **process mining** capability — "Understand your processes and identify opportunities with process mining" — and a **task mining** capability to "Streamline workflows and identify inefficiencies," both surfaced in the product docs hub. This is the platform's answer to the question "which process should we automate?" — measure first.

### 5.2 Desktop Flows and Robotic Process Automation

**Verified (✅):** "**Desktop flows broaden the existing robotic process automation (RPA) capabilities in Power Automate and enable you to automate all repetitive desktop processes**" (learn.microsoft.com/power-automate/desktop-flows/introduction). The verified mechanics:

- **Targets:** Power Automate "allows you to automate both **legacy applications, such as terminal emulators**, modern web and desktop applications, Excel files, and folders" (✅) — the explicit mention of terminal emulators is the tell that this is industrial-grade RPA, not just browser scripting.
- **Interaction method:** "You can interact with the machine by using application **UI elements, images, or coordinates**" (✅) — three escalating fragilities, in that order.
- **Authoring:** "prebuilt drag-and-drop actions or **recording your own desktop flows**" (✅).
- **History:** Microsoft acquired **Softomotive** (makers of ProcessRobot and WinAutomation) in **2020** and **Minit** (process mining) in **2022** to build out Power Automate's RPA and mining capabilities (✅, Wikipedia).

**Why this matters in a bank:** desktop flows are how automation reaches systems that have no API — often the oldest, most critical systems in a regulated estate (mainframe green-screens, legacy ERPs, vendor client software). The trade-off is that UI-level automation is *brittle*: a screen change breaks the flow, and a bank running unattended RPA against a core system must treat those flows as production-change-managed artefacts, not as maker side-projects.

### 5.3 Attended versus Unattended RPA

The attended/unattended distinction is the operating-model decision for RPA. **⚠-knowledge** (standard RPA industry framing; Microsoft's licensing tiers encode it, but this pass did not pin an explicit definition page):
- **Attended** RPA runs on a human's desktop, triggered by the human, alongside the human — the robot is a helper (e.g., pulling data into a screen the agent is already working on). It has no separate runtime cost per robot and no credential-vault custody problem to the same degree.
- **Unattended** RPA runs on a server or virtual machine with no human present, on a schedule or queue, typically using a dedicated machine identity. It requires a runtime licence, a **machine group**, and — critically for a bank — secure custody of the credentials the robot uses to log into target systems. Unattended RPA is where the operational-risk and access-management questions concentrate.

The guide flags the split ⚠-knowledge because Microsoft's licensing page confirms the *licence types* exist (see §9.7) but this pass did not retrieve a single canonical "attended vs unattended" definition page; the substance above is standard RPA practice applied to Power Automate.

### 5.4 Process Mining and Task Mining

Covered in §5.1 (✅ for existence): process mining analyses system event logs to reconstruct how a process actually runs; task mining analyses desktop activity to find the steps humans repeat. Together they are the evidence base for an automation programme — and the reason a serious Power Automate rollout starts with a measured process, not with a maker's hunch.

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) §3 | Technology/ | The iPaaS category and connector-flow integration taxonomy |
| [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) | Technology/ | The integration patterns desktop flows work around |
| [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §4 | Banking/ | Operational-risk framing for unattended automation |

---

## 6. Power BI: Semantic Models, Reports, and the Fabric Relationship

### 6.1 Semantic Models and Reports

**Verified (✅):** Power BI is "**Microsoft's business analytics platform that helps you turn data into actionable insights**" (learn.microsoft.com/power-bi/fundamentals/power-bi-overview). The architecture splits into:

- **Power BI Desktop** — the Windows authoring app: "Connect to more than 100 data sources," "Use Power Query Editor for data transformation," "Create data models with **DAX**, calculated columns, and relationships," "Access more than 30 built-in and custom visuals" (✅).
- **The Power BI service** — the cloud platform for "publishing, sharing, and collaboration": workspaces, apps, shared datasets, scheduled refresh, alerts, RLS (row-level security), sensitivity labels, usage metrics, and audit logs (✅).
- **Semantic models** — Microsoft's current term for what used to be called *datasets*: the documented guidance directs readers to "**Semantic models**" under "Analyze data in Microsoft Fabric with Power BI" (✅, Power BI docs hub). A semantic model is the reusable, governed data-and-measure layer that many reports can share; the persona guidance lists a distinct **semantic model designer** role (✅ docs hub), which is the platform's recognition that the highest-value BI skill is modelling, not charting.
- **Reports, dashboards, and paginated reports** — interactive reports, curated dashboards, and "pixel-perfect, printable reports" built with Power BI Report Builder for structured outputs such as invoices (✅).

For the analytics discipline behind this — descriptive vs predictive vs prescriptive analytics, and where BI tools sit — see [advanced_analytics_solutions_guide.md](advanced_analytics_solutions_guide.md); this guide does not re-derive it.

### 6.2 The Microsoft Fabric Relationship

**Verified (✅):** "**Power BI is a core component of Microsoft Fabric**, providing analytics and visualization capabilities. In Fabric, Power BI shares features like data integration, dataflows (legacy), and security with other Fabric experiences. However, some features — such as Power BI reports, dashboards, and the Power BI service — are unique to Power BI" (learn.microsoft.com/power-bi/fundamentals/power-bi-overview). The docs stress continuity: "**No migration needed** — your Power BI content and workspaces stay the same," and "your existing Power BI or Microsoft 365 account" keeps working (✅). The shared-feature table confirms that **OneLake integration, Direct Lake mode, dataflows Gen2/notebooks, Data Activator, Purview-based security/governance, and Copilot** are all Fabric-and-Power-BI shared capabilities, while Desktop, the service, paginated reports, the visuals marketplace, Q&A, and Goals are Power BI-specific (✅).

The architectural takeaway: **Power BI is now the visualization and semantic-model lane of a larger SaaS analytics platform (Fabric) that also owns the lakehouse, pipeline, and warehouse lanes.** The Fabric/OneLake/Synapse story — and the Direct Lake mode that reads lakehouse data without import — is developed in [data/data_fabric_guide.md](data/data_fabric_guide.md) and [data/enterprise_data_platforms_guide.md](data/enterprise_data_platforms_guide.md) and is **not re-derived here**. For the bank, the important seam is that a Power BI semantic model can source from Dataverse (the operational low-code data) *or* from the Fabric lakehouse (the analytical estate) — and the two should not be conflated in governance.

### 6.3 Self-Service Analytics and Its Governance Burden

Power BI is the component most likely to leak: a maker can connect a personal gateway to a production database and publish a report to a workspace shared with the wrong audience in an afternoon. Microsoft's mitigations are real but require configuration — RLS, sensitivity labels, Purview integration, and the tenant/admin settings in the Fabric admin portal (✅, shared-security row and the admin links in the docs hub). The management-side discipline (metrics catalogues, the analytics operating model) is developed in [../management/analytics_management_guide.md](../management/analytics_management_guide.md); the data-lineage discipline is in [data_governance_guide.md](data_governance_guide.md) and [data/data_lineage_tools.md](data/data_lineage_tools.md).

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [advanced_analytics_solutions_guide.md](advanced_analytics_solutions_guide.md) | Technology/ | BI vs advanced analytics, tool landscape (do not re-derive) |
| [data/data_fabric_guide.md](data/data_fabric_guide.md) | Technology/ | Fabric, OneLake, Direct Lake (do not re-derive) |
| [data/enterprise_data_platforms_guide.md](data/enterprise_data_platforms_guide.md) | Technology/ | The data-platform comparison Fabric sits in |
| [../management/analytics_management_guide.md](../management/analytics_management_guide.md) | Management/ | The analytics operating model and governance |


## 7. Power Pages and Dataverse: The External Surface and the Data Platform

### 7.1 Power Pages: External-Facing Low-Code Websites

**Verified (✅):** "Microsoft Power Pages is a **secure, enterprise-grade, low-code software as a service (SaaS) platform for creating, hosting, and administering modern external-facing business websites**" (learn.microsoft.com/power-pages/introduction). The verified properties that matter architecturally:

- **Audience is external.** Power Pages is for "users outside your organization" who can "sign in with a wide variety of identities, create and view data in Dataverse, or browse content anonymously" (✅, Power Pages docs hub). This is the platform's edge — the component that faces customers, partners, and citizens.
- **Data is Dataverse.** "With Power Pages, you can build sites by using the **same shared business data stored in Microsoft Dataverse** that you use for building apps, workflows, intelligent virtual agents, reports, and analytics with other Microsoft Power Platform components" (✅, introduction page). Same data layer, external surface.
- **Security is a first-class concern.** The product documents site authentication, **web roles**, **table permissions**, a **Web Application Firewall**, and site-visibility controls (✅ docs hub). Because the surface is external, Power Pages carries the platform's most exposed attack surface, and its `table permissions` / `web roles` model is the equivalent of Dataverse security roles for anonymous and authenticated outsiders.
- **Governance is standard.** Power Pages supports solutions, ALM, and Power Platform pipelines like the rest of the suite (✅ docs hub sections "Use solutions with Power Pages," "Use Power Platform pipelines with Power Pages").
- **Copilot and AI.** Microsoft offers AI-generated sites and forms via Copilot and the ability to "Add an agent to your site" (✅ docs hub) — the seam where Power Pages and Copilot Studio meet.

**Lineage (✅):** Power Pages was "formerly part of Power Apps as 'Power Apps Portals' until 2022" (Wikipedia; the product's own page calls it "the newest member of the Microsoft Power Platform family"). A generator note for anyone reading older documentation: "portals" and "Power Pages" are the same product pre- and post-2022.

### 7.2 Dataverse: Tables and the Common Data Model Lineage

**Verified (✅):** "Dataverse lets you **securely store and manage data that's used by business applications**. Data within Dataverse is stored within a set of **tables**. A *table* is a set of rows (formerly referred to as records) and columns (formerly referred to as fields/attributes)" (learn.microsoft.com Dataverse intro). Verified facts:

- **Standard and custom tables.** "Dataverse includes a base set of standard tables that cover typical scenarios, but you can also create custom tables specific to your organization" (✅).
- **Common Data Model lineage.** Dataverse "is based on **Common Data Model principles**" and "is based on Microsoft's Common Data Model as its common data model and is built on Microsoft Azure SQL, where its physical data also is stored" (Wikipedia, cross-checked; the Azure SQL claim carries a "citation needed" tag on Wikipedia, so this guide flags the *physical storage detail* ⚠ while the Common Data Model lineage is ✅ from the product page).
- **Beyond relational.** "In addition to relational data, Dataverse also has support for file and blob storage, data lakes and semi-structured data" (Wikipedia, cross-checked).
- **Why it earns its place.** Microsoft's own list of benefits: "Easy to manage," "Easy to secure," "**Access your Dynamics 365 Data**," "Rich metadata," "Logic and validation," and "Productivity tools" (✅).

The Common Data Model lineage is the reason Dataverse's standard tables (Account, Contact, Opportunity, and so on) look familiar across the Microsoft estate: the platform inherits a pre-built business schema rather than starting from a blank database, which is both its convenience and its lock-in.

### 7.3 The Dataverse Security Model

**Verified (✅):** Dataverse "has a rich security model … You can combine **business units, role-based security, row-based security, and column-based security** to define the overall access to information that users have in a Power Platform environment" (learn.microsoft.com Dataverse intro). The role mechanics, verified from the security-roles page (learn.microsoft.com/power-platform/admin/security-roles-privileges):

- **Security roles are cumulative.** "A user can have multiple security roles. Security role privileges are cumulative" (✅).
- **Privileges come in three types:** **tables** (Read, Create, Delete, Write, Assign, Share, Append, Append To — ✅ each defined), **miscellaneous** task-based privileges, and **privacy-related** privileges (export/print) (✅).
- **Access levels define depth:** Organization, Parent: Child Business Unit, Business Unit, User, and None (✅ each defined).
- **Predefined permission settings** for a table include No Access, Full Access, Collaborate, Private, Reference, and Custom (✅).

For the bank, this is the model that satisfies the segregation-of-duties and least-privilege expectations of the ERM framework ([../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §4): the low-code era does not suspend the need for role design — it moves it to *Dataverse's* role editor, and the audit question becomes "who can read this table, at which access level, and how is that reviewed?"

### 7.4 Business Rules and Logic

**Verified (✅):** Dataverse applies "rich server-side logic and validation to ensure data quality and reduce repetitive code in each app that creates and uses data within a table" (learn.microsoft.com Dataverse intro). The documented mechanisms:

- **Business rules** "validate data across multiple columns and tables, and provide warning and error messages, **regardless of the app used to create the data**" (✅) — the enforcement is at the data layer, not the app, which is exactly the property a controlled environment needs.
- **Business process flows** "guide users to ensure they enter data consistently and follow the same steps every time" and are "currently only supported for model-driven apps" (✅).
- **Workflows** "allow you to automate business processes without user interaction" (✅).
- **Business logic with code** — plug-ins and custom APIs for advanced developer scenarios (✅).

The design principle worth stating explicitly: because business rules live on the **table**, they protect the data no matter which of the platform's five products touched it. A canvas app, a flow, a Power Pages form, and an agent all hit the same rule. That is the governance dividend of a shared data layer — and the reason "just put the validation in the app" is an anti-pattern on this platform.

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [data_governance_guide.md](data_governance_guide.md) | Technology/ | The data-governance discipline Dataverse roles implement |
| [data/crm_data_warehouse_modelling.md](data/crm_data_warehouse_modelling.md) | Technology/ | Modelling the CRM-style data Dataverse standard tables carry |
| [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §4 | Banking/ | SoD, least privilege, and the three lines of defence |

---

## 8. Copilot Studio, AI Builder, and the Connectors

### 8.1 Copilot Studio: The Agent-Building Platform

**Verified (✅):** "Microsoft Copilot Studio is a **graphical, low-code studio for building and managing AI-powered agents and workflows**. Build agents and workflows, connect them to your organization's data and systems, and publish them to the channels where your users already work" (learn.microsoft.com/microsoft-copilot-studio). The verified positioning:

- **Where it sits.** Copilot Studio is named first among the platform's products in Microsoft's own hub ("Build agents and workflows with Copilot Studio, apps with Power Apps…"), and it appears in the Power Platform admin center's scope alongside the other products (✅, admin docs hub: "manage environments and settings for Power Apps, Power Automate, Power Pages, Microsoft Copilot Studio, and customer engagement apps").
- **Two things get built.** **Agents** — "an AI assistant that handles conversations and completes tasks" — and **workflows/agent flows** — drag-and-drop automations that can run standalone or be attached to an agent as a tool (✅, Copilot Studio overview).
- **Lineage.** Copilot Studio is the successor to **Power Virtual Agents**, effective **15 November 2023** (Ignite 2023), with the PVA name retired (✅, Microsoft Copilot blog; Wikipedia cross-check).

**Cross-reference (condensed, do not re-derive):** Copilot Studio is also profiled as an agent platform in the AI-agent guides — see [ai_llm/ai_agent_platform_selection_guide.md](ai_llm/ai_agent_platform_selection_guide.md) §1.2.3 (which notes the progressive rebranding toward "Agent Builder" in Microsoft's 2026 messaging and the autonomous-agent capability) and [ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md). This guide covers only the Power Platform-governance view of Copilot Studio.

### 8.2 Agents, Topics, Knowledge, and Channels

**Verified (✅)** from the Copilot Studio overview page:

- **Knowledge sources.** An agent "follows the instructions you give it, draws on the **knowledge sources** you connect, and uses **tools** to take action — reasoning through a request and deciding the best next step" (✅). On the standard harness, an agent matches a request to a **topic** — "a portion of a conversation you design with connected steps, questions, and conditions" — and when a request falls outside the topics, "the agent can still generate a conversational answer from its connected knowledge sources" (✅).
- **Channels.** Agents "can work with employees and customers in multiple languages across **Microsoft Teams, Microsoft 365 Copilot, websites, mobile apps, and other channels**" (✅).
- **Proactive agents.** "Some agents can be given their own account so they can work proactively on tasks and take part in shared business processes, such as onboarding a new employee or coordinating a recurring meeting" (✅) — the agent-as-digital-colleague pattern.
- **Operations.** Analytics (performance, custom metrics, session outcomes), **evaluations** (test sets and a shared grader library before and after publishing), and **administration** (agent inventory, role-based access, cost management) are all documented (✅).
- **Extensibility.** Agents extend through **connectors** and through **MCP servers** ("Enhance your agents with MCP servers," ✅ docs hub) — the same Model Context Protocol interoperability layer the agent guides flag as the default expectation ([ai_llm/ai_agent_platform_selection_guide.md](ai_llm/ai_agent_platform_selection_guide.md) §4).

### 8.3 Autonomous Agents and the Harness Model

**Verified (✅):** Copilot Studio introduces a **harness** abstraction — "the engine that carries out the work behind the scenes. Your choice of harness affects how your agent or workflow reasons, how complex a task it can take on, what it can do out of the box, and how it's billed" (learn.microsoft.com/microsoft-copilot-studio/fundamentals-what-is-copilot-studio). The three documented harnesses:

- **The GitHub Copilot harness** "for reasoning-heavy, multi-step work and complex business processes."
- **The standard harness** "for rule-based agents and structured, repeatable conversations."
- **The Copilot chat harness** "for extending Microsoft 365 Copilot Chat with your organization's knowledge."

This is the platform's answer to the copilot-to-autonomous-agent spectrum that [ai_llm/autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §5 frames (copilot → copilot with tools → agent with human approval → semi-autonomous → fully autonomous): the harness selects how much reasoning the agent does and how much it is scripted. The governance consequence is direct — a reasoning-heavy autonomous agent that can take actions through connectors is a materially different risk object from a topic-based FAQ bot, and it must be inventoried and DLP-scoped accordingly (§9.4, §10.3).

### 8.4 AI Builder

**Verified (✅):** "AI Builder is a Microsoft Power Platform capability you can use to **bring the power of Microsoft AI to your organization, without the need for coding or data science skills**" (learn.microsoft.com/ai-builder). The verified scope:

- **Prompts and models.** The docs cover creating prompts, "Model selection and temperature settings," using your own data in a prompt, and building/training/managing AI models (✅ docs hub).
- **Prebuilt models.** Microsoft documents "**prebuilt models**" and "**prebuilt prompts**" that can be used without training (✅ docs hub).
- **Where it is consumed.** AI Builder models and prompts are usable in **Power Automate**, **Power Apps**, and **Copilot Studio** (✅ — the docs hub lists "Use in Power Automate," "Use in Power Apps," and "Use prompt actions in Copilot Studio").
- **Licensing.** AI Builder is licensed by **capacity/credits**, allocated through the Power Platform admin center (✅, the licensing page lists "AI Builder credits" as a capacity add-on and links "AI Builder licensing").

AI Builder is the component that lets a non-data-scientist add a document-extraction or classification step to a flow: "Use AI Builder GPT to extract information from documents" is a documented scenario (✅ docs hub). For a bank, the classic fit is document-heavy back-office work — forms, invoices, IDs, correspondence — where the model reads and the flow routes. The *model risk* questions that attach to any AI usage in a regulated firm are outside this guide and sit in [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md).

### 8.5 The Connectors and the "1,000+ Connectors" Claim

**Verified (✅) — the mechanism:** "Connectors, at their most basic level, are **strongly typed representations of RESTful application programming interfaces, also known as APIs**" (learn.microsoft.com/power-platform/admin/wp-data-loss-prevention). When wrapped into a connector, an API "becomes easier for makers and citizen developers to use… in their low-code apps, workflows, and chatbots" (✅). Connectors expose **actions** (e.g., "create a file") and **triggers** (e.g., "when a new item is added") that agents and flows use (✅, connectors overview). Microsoft states "**All connectors are agent ready**" (✅).

**Verified (✅) — the taxonomy.** Microsoft documents these connector classes: **prebuilt** (ready-made, no modification) versus **custom** (built for tailored scenarios) (✅, connectors overview); **standard** versus **premium** tiers (✅, connector reference by tier); **certified** connectors, which "Microsoft tests and certifies to ensure they meet Microsoft's standards for security, reliability, and compliance" (✅, data policies page); **custom** connectors; **virtual** connectors (governance "on/off" controls that are "not based on a RESTful API" — e.g., Copilot Studio feature toggles) (✅); and **Model Context Protocol (MCP) connectors**, "a class of connectors that provide more metadata to expose MCP-enabled API endpoints, known as *tools*" (✅).

**⚠ The "1,000+ connectors" claim.** Microsoft's connectors documentation describes "our **large ecosystem** of software as a service (SaaS) connectors" and links a full list, but this pass did **not** find a fixed number stated on the Learn connectors overview page (✅ for the page content; ⚠ for the number). The "1,000+ connectors" figure is a recurring **marketing** framing across Microsoft's Power Platform pages, and — honestly stated — the count has moved over time (early framing spoke of "hundreds," the GA post of a "large and growing family," and later marketing of four-figure totals). This guide therefore treats the number as ⚠: **the connector *ecosystem* is large and the *direction* of travel is upward (verified); the specific count is a moving vendor figure and should be checked against the current connector reference at evaluation time, not quoted from a slide.** The connector reference can be filtered by product, tier, release status, and publisher (✅ docs hub) — that filtered list, not a marketing total, is the defensible number for a procurement document.

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [ai_llm/ai_agent_platform_selection_guide.md](ai_llm/ai_agent_platform_selection_guide.md) §1.2.3, §4 | Technology/AI | Copilot Studio as an agent platform; MCP; selection criteria |
| [ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) | Technology/AI | The enterprise-AI platform landscape (do not re-derive) |
| [ai_llm/autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) §5 | Technology/AI | The copilot→autonomous-agent spectrum |
| [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) §3 | Technology/ | The connector-based iPaaS taxonomy |
| [api_governance_guide.md](api_governance_guide.md) | Technology/ | API lifecycle discipline for the APIs connectors wrap |


## 9. Governance and ALM: The Chapter That Decides the Outcome

A low-code platform without governance is a shadow-IT factory. Microsoft's own guidance is explicit that adoption should be managed and secure at scale (learn.microsoft.com/power-platform/guidance/adoption/environment-strategy). This chapter sets out the governance machinery — environments, managed environments, environment groups, data policies, the CoE Starter Kit and its transition, ALM through solutions and pipelines, and licensing.

### 9.1 The Environment Strategy

**Verified (✅):** "A **Power Platform environment** is a space to store, manage, and share your organization's business data, apps, chatbots, and flows. It also serves as a **container to separate apps that might have different roles, security requirements, or target audiences**" (learn.microsoft.com/power-platform/admin/environments-overview). The verified boundaries:

- **Tenant and geography.** "Each environment is created under a Microsoft Entra tenant, and its resources can only be accessed by users within that tenant. An environment is also bound to a **geographic location**… When you create an app in an environment, that app is routed only to datacenters in that macro region geography" (✅) — this is the data-residency lever.
- **Data isolation.** "When you create an app in an environment, that app is **only permitted to connect to the data sources that are also deployed in that same environment**" (✅). An app in the Test environment cannot reach the Dev database — environments are not just labels, they are security boundaries.
- **Environment roles.** The built-in **Environment Admin** role "can perform all administrative actions on an environment," and the **Environment Maker** role "can create resources within an environment including apps, connections, custom connectors, and flows" (✅). Crucially, "users or groups assigned to these environment roles **aren't automatically given access to the environment's database** (if it exists) and must be given access separately" (✅) — the admin plane and the data plane are distinct.
- **Environment types** (all ✅, from the type table): **Production** ("intended to be used for permanent work"), **Default** ("intended for experimentation, exploration, and lightweight, app trial development"), **Sandbox** ("nonproduction environments, which offer features like copy and reset"), **Trial** ("expire after 30 days and are limited to one per user"), **Developer** ("intended only for use by the owner"), and **Microsoft Dataverse for Teams** (auto-created for a team) (✅ each described verbatim).

**Verified (✅) — Microsoft's strategic direction:** the adoption guidance describes a shift **away from a single shared central environment and toward environment routing** — automatically giving each maker a personal environment with consistent governance applied by group rules — because "in a shared, central environment strategy, makers build, use, and share apps in the default environment. This strategy can result in lack of isolation and makers encroaching on each other. **Imagine if everyone in the company shared a single OneDrive folder for all their documents**" (✅, environment-strategy guidance). That analogy is the cleanest statement of why the flat-environment anti-pattern fails.

### 9.2 The Default Environment

**Verified (✅):** "A single **default environment** is automatically created by Power Apps for each tenant and shared by all users in that tenant. Whenever a new user signs up for Power Apps, they're automatically added to the **Maker role** of the default environment" (learn.microsoft.com/power-platform/admin/environments-overview). Its verified constraints and risks:

- "**The default environment doesn't provide any backup guarantees and shouldn't be used for production workloads**" (✅, type table).
- "You can't delete the default environment. You can't manually back up the default environment" (✅).
- Included capacity: **3 GB Dataverse database capacity, 3 GB file capacity, 1 GB log capacity**, and "the default environment is limited to 1 TB of storage capacity" (✅).

For a regulated organisation, the default environment is the **first security problem to solve**: every licensed user (including guest-ish M365 users) can be a maker in it, it cannot be deleted, and it has no backup guarantee. Microsoft's own guidance treats "securing your default environment" as "a critical first step in implementing an environment strategy" (✅) and recommends **default environment routing** to move makers to their own environments (✅).

### 9.3 Managed Environments and Environment Groups

**Verified (✅):** "**Managed environments are a suite of premium capabilities that allow admins to manage Power Platform at scale with more control, less effort, and more insights**" (learn.microsoft.com/power-platform/admin/managed-environment-overview). The documented feature set includes environment groups, **limit sharing**, weekly usage insights, data policies, pipelines, **solution checker**, **IP firewall**, IP cookie binding, **Customer Managed Key (CMK)**, **Lockbox**, extended backup, and default environment routing (✅ — each listed on the page). Managed environments are "included as an entitlement with standalone Power Apps, Power Automate, Microsoft Copilot Studio, Power Pages, and Dynamics 365 licenses" (✅), but not with the Developer Plan when users run their assets (✅).

**Verified (✅):** "**Environment groups offer a premium governance solution** designed to streamline management tasks by organizing environments into logical collections and enforcing uniform policies and configurations" (learn.microsoft.com/power-platform/admin/environment-groups). The verified rules of the construct:

- "**Environment groups can only contain managed environments.** Each environment can belong to only one group, and groups can't overlap or be nested" (✅).
- "A rule published at the environment group level is enforced across every environment within that group," and the corresponding setting "becomes **locked (read-only)** within individual environments" so local admins can't override it (✅).
- Per-environment exceptions "aren't currently supported" (✅).

Microsoft's own named use cases (✅ each): **Personal productivity environments** ("each maker can automatically get their own personal developer environment… similar to each person having their own OneDrive"), **AI feature management** ("a *Copilot Pilot* group with sandbox environments where AI features are turned on for early testing… production or sensitive environments can remain in a separate group with a more gradual rollout"), **Global environment strategy** (grouping by region to "enforce region-specific compliance and data residency rules"), and **Development vs production** ("a *Dev/Test Group* with relaxed policies… and a *Production Group* with stricter rules such as one that forces solution checker, blocks previews or unmanaged changes") (✅).

This is the machinery a bank's platform team uses to run a **tiered governance model**: one group per lifecycle stage, one group per region, one personal-productivity group with tight sharing limits and agent-sharing restricted, all governed by published rules rather than by individual environment configuration.

### 9.4 Data Loss Prevention Policies

**Verified (✅):** "**Data policies** are a critical aspect of maintaining data security and compliance within the Microsoft Power Platform ecosystem. Create data policies that act as **guardrails** to help reduce the risk of users unintentionally exposing organizational data" (learn.microsoft.com/power-platform/admin/wp-data-loss-prevention). The verified mechanics:

- **What they control.** "Power Platform admin center data policies allow administrators to control access to these **connectors** in various ways to help reduce risk" (✅). The classic classification into **Business / Non-business / Blocked** is the practical pattern Microsoft describes through the *effect* of policies: a connector classified such that it "blocks the use of [a] connector" prevents a maker from saving an app or flow that uses it, and previously created resources "in a *suspended* or *quarantine* state so that it can't operate" (✅ describes blocked/suspended/quarantine semantics; the ⚠ is that this pass did not retrieve a page that uses the literal "Business/Non-business/Blocked" labels — the behaviour is verified, the label set is standard practice and flagged ⚠-knowledge).
- **Enforcement process (✅, verbatim sequence):** (1) save policy configuration; (2) cascade to each environment; (3) resources periodically check for updates; (4) evaluate each app/flow/agent for violations; (5) if a violation occurs, put the resource "in to a *suspended* or *quarantine* state so that it can't operate"; (6) "scan connections… set the connection to a *disabled* state"; (7) "any resources that are running and attempting to use an inactive connection, action, trigger, or MCP server that is blocked, **fail at runtime**."
- **Design-time vs runtime.** A block impacts both: makers can't save new resources that use the connector, and running resources break when their connection is disabled (✅).
- **Latency.** "For the most extreme cases, the latency for full enforcement is **24 hours**. In most cases, it's within an hour" (✅) — a critical planning fact: DLP is near-real-time, not instantaneous, and a bank's incident response must account for that window.
- **Governance evolution.** Microsoft documents **advanced connector policies (ACP)** as the more granular successor for real certified connectors, noting virtual connectors (Copilot Studio toggles, desktop-flow controls) are "evolving into their own dedicated governance rules, separate from both data policies and ACP" (✅).

### 9.5 The CoE Starter Kit and Its Transition

**Verified (✅):** "The Power Platform Center of Excellence (CoE) Starter Kit has historically helped organizations establish governance, visibility, and best practices for Power Platform adoption" — **and, importantly, "the Power Platform CoE Starter Kit is no longer actively maintained."** Microsoft's page states: "**Issues are no longer reviewed or addressed.**" Its core capabilities "are part of the Power Platform admin center" through the in-product experiences **Inventory, Usage, Monitor, and Actions**, which "provide real-time visibility into resources, usage, operational health, and governance insights in a centralized, enterprise-scale experience" (✅, all verbatim from learn.microsoft.com/power-platform/guidance/coe/starter-kit, last updated 05/07/2026).

The verified mapping of CoE toolkit scenarios to admin-center capabilities (✅ each):
- **Inventory** — "view and govern all apps, flows, and agents created across your tenant."
- **Usage** — "track adoption and identify top resources and their owners."
- **Monitor** — "track the operational health of heavily used resources."
- **Actions** — "identify risks, enforce best practices, and take action on governance insights across your tenant."

The CoE Starter Kit "remains available for existing and new deployments, but it will not be enhanced with new capabilities" (✅). The **status/migration story is the single most important governance fact in this guide for anyone holding an older deployment plan**: a program designed around the CoE Starter Kit in earlier years should now be re-baselined onto the admin-center Inventory/Usage/Monitor/Actions experiences, with automation via the **Power Platform CLI, the Power Platform API, the inventory API, and the Power Platform for Admins V2 connector** (✅). Public evidence that banks adopted the CoE pattern exists: the Microsoft Axis Bank story credits a "Power Platform CoE Toolkit [that] enabled centralized visibility into apps and flows" (✅, customer story, §10.2).

### 9.6 ALM: Solutions, Managed versus Unmanaged, and Pipelines

**Verified (✅):** "**Solutions are the mechanism for implementing ALM**; you use them to distribute components across environments through export and import. A component represents an artifact used in your application… such as tables, columns, canvas and model-driven apps, Power Automate flows, agents, charts, and plug-ins" (learn.microsoft.com/power-platform/alm/overview-alm). Verified supporting facts:

- **All environments that participate in ALM must include a Dataverse database** (✅).
- **Source control** "should be your source of truth," and a **CI/CD platform** such as Azure DevOps "allows you to automate your build, test, and deployment pipeline" (✅).
- **Solutions overview (✅, Power Apps docs):** two types of solutions (**managed and unmanaged**), solution components, lifecycle (create, update, upgrade, patch), solution publisher, and dependencies. "We recommend that you always deploy **managed** solutions to nondevelopment environments" (✅, pipelines FAQ) — managed solutions are the controlled artefacts; unmanaged customizations are the development-layer ones.
- **Managed properties** "control which of your managed solution components are customizable," and Microsoft recommends setting them "so that your managed components can't be modified," protecting a solution "from modifications that might cause it to break after it's imported" (✅).
- **Source control integration (✅):** "Connect to Git" allows teams to "sync solutions and solution objects across one or more Dataverse environments using an **Azure DevOps Git repository**."

**Verified (✅) — Pipelines:** "**Pipelines in Power Platform aim to democratize application lifecycle management (ALM)** … bringing ALM automation and continuous integration and continuous delivery (CI/CD) capabilities into the service in a manner that's more approachable for all makers, admins, and developers" (learn.microsoft.com/power-platform/alm/pipelines). Verified properties that matter for a controlled rollout:

- **Sequential, tamper-proof promotion.** "The same solution artifact must pass through pipeline stages in sequential order. The system also prevents any tampering or modification to the exported solution artifact. This ensures **customization can't bypass QA environments or your approval processes**" (✅). This is the audit property a regulated firm needs — the artefact that reaches production is provably the artefact that passed QA.
- **Backups.** "Both managed and unmanaged solutions are automatically exported and stored in the pipelines host for every deployment" (✅).
- **Environments and licensing.** "Developer environments aren't required to be managed environments. The pipelines host should be a production environment… **All other environments used in pipelines must be enabled as managed environments.** Licenses granting premium use rights are required for all managed environments" (✅).
- **Approvals.** Approval-based **delegated deployments** are supported (✅).
- **Managed-environment enforcement.** "Starting February 2026, Microsoft will start enabling managed environments for any pipeline target environments that aren't already enabled" (✅).
- **Pipelines vs the ALM Accelerator.** Pipelines are "more simplistic in nature and can be set up and managed with less effort" and are "an official Microsoft Power Platform product feature"; the ALM Accelerator "is sometimes a better fit for more advanced ALM scenarios" (✅).
- **Do not deploy unmanaged.** "Can I deploy unmanaged solutions? **No.** We recommend that you always deploy managed solutions to nondevelopment environments" (✅).

### 9.7 Licensing: The Public Facts

**⚠ Pricing is the moving part.** Microsoft states explicitly: "This article provides a general overview. **Licensing programs and product terms can change.** For complete and current terms, see the Microsoft Power Platform Licensing Guide and your licensing agreement" (✅, learn.microsoft.com/power-platform/admin/pricing-billing-skus). The structural facts (✅) are durable; the *prices* are ⚠ and should never be quoted from a guide.

**Verified (✅) — the licensing *shapes*:**
- **Three licence families:** "user licenses, capacity-based licenses, and consumption-based meters" (✅).
- **Standalone licences** "provide use rights for a specific Microsoft Power Platform product" — e.g., **Power Apps Premium** and **Power Automate Premium** "provide premium capabilities for licensed users," and product licences "can also include Dataverse and other capacity entitlements" (✅).
- **Power Apps per app** "is a capacity-based license for running a specific app in an environment. **Effective January 2, 2026, the Power Apps per app SKU is no longer available to new customers through some purchasing channels**" (✅) — a concrete example of why ⚠ pricing flags exist: the per-app model itself changed during the guide's currency window.
- **Microsoft 365 use rights** are "limited" and "intended for scenarios that use Microsoft 365 data and standard connectors. They don't provide the same rights as standalone premium licenses" (✅).
- **Dynamics 365 use rights** are restricted to extension/customization "in the context of the licensed Dynamics 365 application" (✅).
- **Capacity add-ons** "provide tenant-level or environment-level capacity for specific services and features. Examples include **Dataverse storage, AI Builder credits, Power Pages capacity, and Copilot Credits**" (✅).
- **Pay-as-you-go** "links an environment to an Azure subscription through a billing plan… eligible usage is billed through Azure meters," with meters for Power Apps, Power Automate, Dataverse, Power Pages, and Copilot Studio (✅).
- **Power Apps Developer Plan** provides "an individual developer environment… **intended for development and test use, not production workloads**" (✅).
- **Dataverse capacity** is metered in **database, file, and log** categories, and "there's no technical limit on the size of a Dataverse environment. The limits… are **entitlement limits based on product licenses**" (✅, capacity-storage page). The default environment's included capacity is 3 GB database / 3 GB file / 1 GB log (✅). Enforcement is currently "**soft enforcement** through email notification," with alerts when usage "exceeds **85 percent** of the allocated capacity" (✅).

The purchasing-shape summary for a CFO-level conversation (⚠ synthesis): you pay **per user** (premium licences), **per app** (being retired), **per capacity** (Dataverse storage, AI Builder credits, Copilot Credits), or **per metered use** (pay-as-you-go against Azure). The TCO discipline that prices these over a multi-year horizon — not the sticker prices — is [tco_modeling_guide.md](tco_modeling_guide.md).

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [tco_modeling_guide.md](tco_modeling_guide.md) §3–§6 | Technology/ | The TCO taxonomy and sensitivities the licensing feeds |
| [data_governance_guide.md](data_governance_guide.md) | Technology/ | Data-governance framing for DLP and Dataverse roles |
| [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §3–§4 | Banking/ | The control framework governance implements |
| [../management/vendor_management_guide.md](../management/vendor_management_guide.md) | Management/ | Vendor/contract discipline for licence terms |

---

## 10. The Banking Angle: Citizen Developers Under Supervision

### 10.1 Citizen-Developer Governance and the Shadow-IT Risk

The promise of the platform is that the business solves its own problems; the risk is that it does so outside IT's visibility. Microsoft's DLP page names the exact hazard: guardrails exist to "reduce the risk of users unintentionally exposing organizational data" (✅), and the default environment — open to every licensed user as a maker, undeletable, no backup guarantee (✅, §9.2) — is where ungoverned shadow IT accumulates.

The bank-specific amplification of that risk:
- **Data sensitivity.** A low-code app that reads a customer table and posts to a personal connector is a potential data-exfiltration path — hence the DLP classification of connectors and the block-list of consumer/unknown endpoints (✅ mechanism, §9.4).
- **Regulatory perimeter.** Any system that touches customer data, or that an auditor relies on, is inside the regulated perimeter regardless of how it was built. MAS technology-risk expectations ([../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §5) and the operational-risk families of the ERM framework ([../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §3–§4) do not exempt a citizen-built flow.
- **Change and access discipline.** Unattended desktop flows using shared credentials (⚠-knowledge, §5.3) and maker-owned connections that break when a maker leaves are classic operational-risk findings.
- **Model and AI risk.** Copilot Studio agents and AI Builder models that reason over bank data add a new control surface (inventory, evaluation, DLP-scoping) — the AI-specific compliance layer is in [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md).

The governance answer is not prohibition — it is **controlled enablement**: personal environments with tight rules, a curated connector allow-list, promotion through managed solutions and pipelines, and an inventory that makes the estate visible. That is precisely what the platform's premium governance stack (§9.3–§9.6) was built for.

### 10.2 Regulated-Industry Adoption: The Public Evidence

Microsoft publishes customer stories that name banks explicitly. Two are verified this pass and are the honest public evidence base (both ⚠-vendor in the sense that they are Microsoft-authored marketing artefacts, but ✅ as documents that exist and make the stated claims):

- **HSBC (published 27 May 2026)** — "HSBC drives innovation in Power Platform security." Verified content: HSBC "used Microsoft Power Platform, Dynamics 365, and Microsoft Dataverse security features such as **virtual network support and customer-managed keys**" to "strengthen data protection, improve control and transparency, and enable teams to innovate securely while meeting regulatory requirements." Named controls: **Virtual Network support** for data egress control, **customer-managed keys (CMK)** for at-rest encryption with customer-controlled key rotation, **Customer Lockbox** requiring "explicit customer approval before Microsoft engineers can access their data," and CMK extending to **Copilot Studio**. The story is tagged industry: **Banking**, country: **United Kingdom**, size: **10,000+ employees**, citing HSBC's scale in "over 50 markets." (✅ as the story's claims; the bank's actual internal metrics are not independently verifiable — ⚠.)
- **Axis Bank (published 24 February 2026)** — "Accelerating Digital Transformation: Axis Bank's Success with Microsoft Power Platform." Verified content: Axis Bank "established a Power Apps Centre of Excellence (CoE) and scale[d] Microsoft Power Platform across the organization," and "**in two years, Axis Bank built over 120 solutions used by 50,000+ employees**." The story reports "over 120 solutions processing over 100,000 transactions each month and reduced operational latency by **40%**," training of "over **1,000 employees** on Power Platform," a "**Power Platform CoE Toolkit [that] enabled centralized visibility into apps and flows,"** and "a Power Platform App Lifecycle Management (ALM) setup [that] supported controlled, regulated, scalable app deployments across environments." Named regulatory use cases include **Centralized EDD (enhanced due diligence)**, **Audit Workflow Management**, **Statutory Notices Management**, and **Independent Cash Verification**. (✅ as the story's claims; the metrics are vendor-published and ⚠ as independent fact.)

**What the evidence does and does not prove.** It **proves** (✅) that Microsoft names banks publicly as Power Platform adopters and that a major bank's CoE-and-ALM governance pattern is documentable. It **does not prove** (❌ this pass) a systematic census of which banks run which Power Platform estates, nor independently audited efficiency figures. Treat the stories as *existence proofs of the pattern*, not as benchmarks.

### 10.3 The Security Controls a Bank Needs

Synthesising the verified Microsoft capabilities into a control checklist a bank should be able to tick:

| Control need | Power Platform mechanism (verified) | Where |
| --- | --- | --- |
| Tenant/data-residency boundary | Environment bound to a geography; apps route only to that macro region | §9.1 ✅ |
| Least privilege on data | Dataverse business units + role/row/column security; cumulative roles; access levels | §7.3 ✅ |
| Prevent data exfiltration | DLP/data policies classifying and blocking connectors; design-time + runtime enforcement | §9.4 ✅ |
| Isolate maker activity | Personal productivity environment groups with sharing/agent-sharing limits; default environment routing | §9.3 ✅ |
| Govern at scale | Managed environments + environment-group published rules (locked settings) | §9.3 ✅ |
| Encrypt under customer control | Customer Managed Key (CMK); key rotation | §9.3, §10.2 ✅ |
| Network isolation | Virtual Network support; IP firewall; IP cookie binding | §9.3, §10.2 ✅ |
| Microsoft-access approval | Customer Lockbox | §10.2 ✅ |
| Controlled deployment | Managed solutions, solution checker, pipelines with enforced sequential promotion; Git source control | §9.6 ✅ |
| Inventory and audit | Admin-center Inventory/Usage/Monitor/Actions; audit logs; CLI/API automation | §9.5 ✅ |
| Segregation of duties | Dataverse security-role design (initiator ≠ approver, per the ERM SoD matrix) | §7.3, [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §4 ✅ mechanism |

The honest gap: these controls are **available but not automatic**. A tenant with no managed environments, no DLP policy, and the default environment left open has none of them switched on. Governance on this platform is a configuration project, not a default state — which is why the worked example in §11 exists.

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §5, §7 | Banking/ | MAS technology-risk and outsourcing expectations |
| [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §3–§4 | Banking/ | Operational risk, SoD, three lines of defence |
| [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md) | Banking/ | Model/AI risk for Copilot Studio and AI Builder |
| [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) | Banking/ | The Singapore banking landscape Cymbal Bank operates in |


## 11. Cymbal Bank Worked Example: A Power Platform Governance Rollout

### 11.1 The Scenario

**Cymbal Bank** — the repo's global corporate and investment bank persona: headquarters in Paris, a major APAC hub in Singapore, a balance sheet of roughly €400bn, and four businesses — global markets, structured finance, trade finance, and corporate banking (persona per [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §10; the group parent applies full Basel III under EU CRR3 and the Singapore branch is supervised by MAS). The Dynamics 365 Finance estate that Power Platform sits beside is the subject of the FMS guide ([financial_management_systems_guide.md](financial_management_systems_guide.md) §10); this example is the *low-code* rollout, not the ERP selection.

**The trigger.** A platform review finds the classic shadow-IT picture, discovered rather than designed:
- **~3,400 apps and ~9,000 flows** across the tenant (⚠ — the bank's own inventory estimate at the start of the programme), the large majority in the **default environment** with no backup guarantee.
- **No DLP policy** beyond a default — makers free to connect bank data to whatever connector they choose.
- **Two incidents**: a maker shared a customer-data canvas app with "Everyone" in the tenant; and an unattended desktop flow broke when a maker left, taking a month-end reconciliation with it.
- **No ALM**: solutions unmanaged in production, no pipelines, no source control.

The **programme mandate**: keep the business building, put guardrails around it, and make the estate visible and auditable — the controlled-enablement answer of §10.1, not a freeze. The numbers below are Cymbal Bank's own design and estimates (⚠ — analysis, not verifiable fact), consistent with the persona and with the verified Microsoft capabilities cited in §9.

### 11.2 The Environment Strategy

The bank adopts Microsoft's **environment-routing + environment-groups** direction (§9.1, §9.3) rather than a flat shared environment. The tiering:

| Environment group | Environment types | Purpose | Key published rules (⚠ — the bank's design) |
| --- | --- | --- | --- |
| **Personal Productivity** | Developer (routed per maker) | Every licensed maker gets a personal sandbox | Sharing restricted; agent sharing off; maker welcome content on; no premium connectors until approved |
| **Dev/Test** | Sandbox / Developer | Team development and integration testing | Relaxed: preview features allowed; unmanaged customizations allowed; solution checker advisory |
| **Pre-Production (UAT)** | Sandbox | User acceptance and controlled validation | Solution checker enforced; unmanaged changes blocked; longer retention for test evidence |
| **Production** | Production (regional: EU, APAC) | Live business apps, flows, agents | Solution checker enforced; unmanaged changes blocked; CMK on; VNet on; conditional access on granular apps; extended backup |
| **Platform / CoE** | Production (host) | Pipelines host, inventory, catalog | Admin-only; pipelines host |

Design points, each traceable to a verified capability:
- **Region-aligned production.** EU and APAC production environments are separate, honouring the geo-binding of environments for the group's data-residency posture (✅ mechanism, §9.1) and the MAS-supervised Singapore branch ([../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §7).
- **Default environment locked down.** Consistent with Microsoft's "securing your default environment is a critical first step" guidance (✅), routing is enabled so new makers land in personal environments, and the default environment is stripped of unmanaged production assets as the inventory migration completes.
- **Managed environments everywhere that matters.** All production, UAT, and pipeline-participating environments are managed (✅ requirement for pipeline environments, §9.6); personal environments are managed too, so sharing limits and agent controls apply.

### 11.3 The DLP Policy Design

The bank defines a **connector classification** — the Business / Non-business / Blocked pattern of §9.4 (⚠ the literal label set is standard practice; the *behaviour* is verified ✅) — and publishes it as tenant-wide data policies, with environment-group rules layering on top. The starter classification (⚠ — the bank's design; connector names are illustrative and must be checked against the live connector reference, §8.5):

| Class | Representative connectors (illustrative) | Rationale |
| --- | --- | --- |
| **Business** (allow) | Dataverse, SharePoint, Office 365 Outlook, Teams, SQL Server (via approved gateway), Azure services on the allow-list, the bank's own certified custom connectors | Sanctioned corporate data flows |
| **Non-business** (blocked from mixing with Business) | Consumer file-share and social connectors; generic HTTP/HTTP-with-Azure-AD unless reviewed; unvetted third-party SaaS | Prevents Business↔Non-business data mixing in one flow |
| **Blocked** | Consumer email/file-transfer with no enterprise agreement; unknown-publisher custom connectors; any connector to a personal cloud drive | Hard block — the exfiltration path |
| **Virtual connectors / Copilot controls** | Copilot Studio feature toggles; desktop-flow controls | Governed via environment-group rules and Copilot Studio's own governance (✅ mechanism, §9.4) |

Operational rules the bank adds:
- **Custom connectors require review.** Because custom connectors "require careful consideration to ensure that they comply with data policies" (✅), the bank routes every custom connector through a platform-team review gate before certification.
- **Desktop flows get their own policy.** Manage and restrict desktop-flow connectors via data policies for desktop flows in managed environments (✅, listed as a managed-environment feature, §9.3) — a direct response to the broken unattended flow incident.
- **Enforcement is not instant.** The bank documents the **up to 24-hour** full-enforcement latency (✅, §9.4) in its runbook so that incident response does not assume a block takes effect immediately.
- **Advanced connector policies** are the planned successor for granular certified-connector control, with virtual connectors migrating to dedicated rules (✅, §9.4).

### 11.4 CoE Starter Kit Adoption

Because the **CoE Starter Kit is no longer actively maintained** (✅, §9.5), the bank proceeds in two phases and does **not** build a new strategy on the kit:

1. **Interim (existing deployment).** Where the bank already runs CoE Starter Kit components, it keeps them for continuity but freezes new investment — consistent with Microsoft's "remains available… but will not be enhanced" status (✅).
2. **Target state (admin center).** The programme re-baselines onto the in-product **Inventory, Usage, Monitor, and Actions** experiences (✅ each, §9.5), with automation through the **Power Platform CLI, the Power Platform API, the inventory API, and the Power Platform for Admins V2 connector** (✅). Deliverables:
   - **A complete app/flow/agent inventory** — "view and govern all apps, flows, and agents created across your tenant" (✅) — feeding the bank's CMDB and the AI/agent register.
   - **Adoption analytics** — "track adoption and identify top resources and their owners" (✅) — used to find the top 100 apps by usage and put them into the managed-solution pipeline first.
   - **Operational monitoring** of heavily used resources (✅), so the failed-run/alert story is proactive rather than an end-of-month surprise.
   - **Actions for risk** — enforce best practices and act on governance insights (✅) — driving remediation tickets into the platform team's queue.

The public precedent the bank cites internally is the Axis Bank CoE pattern — a CoE, an app/flow inventory via the CoE toolkit, and an ALM setup (✅, §10.2) — with the caveat that the kit's *tooling* is being retired even as the *pattern* endures.

### 11.5 The ALM Discipline

The bank adopts the verified ALM stack of §9.6:

- **Everything in solutions.** Every app, flow, and agent is built in a Dataverse solution ("all environments that participate in ALM must include a Dataverse database" — ✅), with a preferred solution per project.
- **Managed in production.** "We recommend that you always deploy managed solutions to nondevelopment environments" (✅) — the bank enforces managed-only in UAT and Production, and sets **managed properties** so production components can't be modified in place (✅).
- **Pipelines with enforced sequence.** The bank stands up **pipelines in Power Platform** with Dev→Test→UAT→Prod stages; the platform guarantees "the same solution artifact must pass through pipeline stages in sequential order" and "prevents any tampering" so "customization can't bypass QA environments or your approval processes" (✅) — the audit property the bank's internal audit team required.
- **Approvals and segregation of duties.** UAT→Prod requires **delegated-deployment approval** (✅), with the approver distinct from the maker — the Power Platform mirror of the finance SoD matrix ([../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §4): *initiator ≠ approver ≠ releaser*.
- **Source control.** Solutions sync to an **Azure DevOps Git repository** (✅) as the source of truth, with the pipelines host and dev/test environments *not* requiring standalone licences while the production target does (✅, the pipeline licence table).
- **Managed environments as a guardrail.** All pipeline target environments are managed (✅ requirement); the bank also enables the automatic managed-environment setting for pipeline targets ahead of Microsoft's February 2026 enablement (✅).
- **Solution checker** runs on every promotion (✅ available and enforced via the Production group's rules).

### 11.6 Outcomes and Governance Metrics

The programme's scorecard (⚠ — the bank's targets and design, not measured results; consistent with the verified capabilities):

| Metric | Baseline (⚠) | Target (⚠) | Lever |
| --- | --- | --- | --- |
| Governance visibility | ~3,400 apps / ~9,000 flows, largely invisible | **100% of apps/flows/agents inventoried** | Admin-center Inventory + API export to CMDB |
| Default-environment production assets | Majority of estate | **0 production workloads in the default environment** | Environment routing + personal-productivity group |
| DLP coverage | No effective policy | **Tenant-wide Business/Non-business/Blocked policy; all custom connectors reviewed** | Data policies + environment-group rules |
| Production deployment control | Manual, unmanaged | **Managed-only, pipeline-enforced Dev→Test→UAT→Prod with approval** | Managed solutions + pipelines |
| Segregation of duties | None | **Approver ≠ maker on every production deployment** | Delegated deployments + platform roles |
| Encryption / network isolation | Not enabled | **CMK on for production; VNet support for data-egress-sensitive workloads** | Managed-environment security features |
| Incident recurrence | 2 in the baseline year | **No repeat of the "shared to Everyone" or "orphaned unattended flow" classes** | Sharing limits + desktop-flow data policies |
| Business enablement | Ad-hoc | **Retain or grow the maker base; promote the top 100 apps to managed pipelines** | Inventory-driven prioritisation |

The design philosophy, stated once: **the guardrails should be invisible to the good maker and impassable to the bad one.** A maker who builds a legitimate app in a personal environment, uses allow-listed connectors, and promotes it through the pipeline should feel almost no friction; the same controls should make it structurally impossible to connect bank data to a blocked consumer endpoint, promote an untested artefact, or leave a production workload in an environment with no backup guarantee. That is the whole point of doing governance *on the platform* rather than *against* it.

| Reference | Where it lives | What it gives this section |
| --- | --- | --- |
| [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §10 | Banking/ | The Cymbal Bank persona and ERM framework |
| [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §5, §7 | Banking/ | MAS technology-risk and branch supervision |
| [financial_management_systems_guide.md](financial_management_systems_guide.md) §10 | Technology/ | The sibling worked example (FMS selection) for the same bank |
| [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) §7 | Technology/ | The integration-side worked example |
| [../management/it_strategy_guide.md](../management/it_strategy_guide.md) | Management/ | The platform operating model the CoE sits in |

---

## 12. Claims Audit and What Could Not Be Verified

Every key factual claim in this guide, with its verification status. ✅ = verified this pass against a primary or named source; ⚠ = approximate / vendor claim / single secondary source; ⚠-knowledge = well-established industry knowledge not re-verified this pass; ❌ = could not be verified.

| # | Claim | Status | Source this pass |
| --- | --- | --- | --- |
| 1 | Power Platform is presented as one platform for agents, apps, automations, analytics, and websites | ✅ | learn.microsoft.com/power-platform (2026-09-10) |
| 2 | Platform capabilities are AI Builder, connectors, Copilot/GenAI, Dataverse, Power Fx | ✅ | learn.microsoft.com/power-platform |
| 3 | Power Platform is "a collection of low-code development tools" | ✅ | Wikipedia (Microsoft Power Platform) |
| 4 | Power BI began as Power Pivot/Power Query; "Project Crescent" designed summer 2010 | ✅ | Wikipedia (Microsoft Power BI) |
| 5 | Project Crescent first public download 11 July 2011 | ✅ | Wikipedia |
| 6 | Renamed Power BI and unveiled September 2013 | ✅ | Wikipedia |
| 7 | Power BI released to the general public 24 July 2015 | ✅ | Wikipedia |
| 8 | Microsoft acquired Datazen 14 April 2015 | ✅ | Wikipedia |
| 9 | PowerApps and Flow GA announced 31 Oct 2016, effective 1 Nov 2016, 42 languages | ✅ | blogs.microsoft.com (2016-10-31); Wikipedia |
| 10 | Common Data Service announced in the GA post as the unifying data backbone | ✅ | blogs.microsoft.com (2016-10-31) |
| 11 | PowerApps/Flow public preview April 2016; "power trio" framing | ✅ | blogs.microsoft.com (2016-10-31) |
| 12 | Flow rebranded Power Automate 4 Nov 2019 (Ignite 2019); RPA added | ✅ | Wikipedia (Microsoft Power Automate) |
| 13 | Microsoft acquired Softomotive (2020) and Minit (2022) | ✅ | Wikipedia |
| 14 | Common Data Service renamed Dataverse effective November 2020 | ✅ | Wikipedia; learn.microsoft.com Dataverse intro |
| 15 | Dataverse terminology changed entity→table, field→column, record→row (Nov 2020) | ✅ | learn.microsoft.com Dataverse intro |
| 16 | Power Pages was "Power Apps Portals" until 2022; "newest member" of the platform | ✅ | Wikipedia; learn.microsoft.com/power-pages |
| 17 | Power Virtual Agents folded into Copilot Studio effective 15 November 2023 | ✅ | Microsoft Copilot blog; Wikipedia |
| 18 | Power Automate has three flow types: cloud, desktop, generative actions (preview) | ✅ | learn.microsoft.com/power-automate/flow-types |
| 19 | Desktop flows "broaden the existing RPA capabilities"; automate terminal emulators; UI elements/images/coordinates | ✅ | learn.microsoft.com/power-automate/desktop-flows/introduction |
| 20 | Process mining and task mining capabilities exist | ✅ | learn.microsoft.com/power-automate docs hub |
| 21 | Canvas apps: blank design surface, drag-and-drop, Power Fx, hundreds of sources | ✅ | learn.microsoft.com/power-apps/maker/canvas-apps |
| 22 | Model-driven apps: component-based, Dataverse-only, auto-responsive/accessible | ✅ | learn.microsoft.com/power-apps/maker/model-driven-apps |
| 23 | Microsoft names "bank member relationships" as a model-driven app fit | ✅ | learn.microsoft.com model-driven app overview |
| 24 | Code apps and Plan designer exist | ✅ | learn.microsoft.com/power-apps docs hub |
| 25 | Power BI is "Microsoft's business analytics platform"; Desktop vs service split | ✅ | learn.microsoft.com/power-bi/fundamentals/power-bi-overview |
| 26 | "Semantic models" is the current term; a semantic model designer persona exists | ✅ | learn.microsoft.com/power-bi docs hub |
| 27 | Power BI is "a core component of Microsoft Fabric"; no migration needed | ✅ | learn.microsoft.com/power-bi/fundamentals/power-bi-overview |
| 28 | Power Pages: "secure, enterprise-grade, low-code SaaS… external-facing business websites"; uses Dataverse | ✅ | learn.microsoft.com/power-pages/introduction |
| 29 | Dataverse: tables/rows/columns; standard and custom tables; Common Data Model principles | ✅ | learn.microsoft.com Dataverse intro; Wikipedia |
| 30 | Dataverse is built on Azure SQL (physical storage) | ⚠ | Wikipedia ("citation needed" tag) |
| 31 | Dataverse security: business units + role + row + column security | ✅ | learn.microsoft.com Dataverse intro |
| 32 | Security roles cumulative; table/misc/privacy privileges; five access levels | ✅ | learn.microsoft.com admin security-roles-privileges |
| 33 | Business rules validate regardless of the app used | ✅ | learn.microsoft.com Dataverse intro |
| 34 | Copilot Studio: "graphical, low-code studio for building and managing AI-powered agents and workflows" | ✅ | learn.microsoft.com/microsoft-copilot-studio |
| 35 | Copilot Studio harnesses: GitHub Copilot, standard, Copilot chat | ✅ | learn.microsoft.com Copilot Studio overview |
| 36 | Agents publish to Teams, M365 Copilot, websites, mobile apps; MCP servers supported | ✅ | learn.microsoft.com Copilot Studio |
| 37 | AI Builder: "bring the power of Microsoft AI… without coding or data science skills"; credits licensing | ✅ | learn.microsoft.com/ai-builder; licensing page |
| 38 | Connectors are "strongly typed representations of RESTful APIs"; prebuilt vs custom | ✅ | learn.microsoft.com admin data-policies; connectors overview |
| 39 | Standard vs premium connector tiers; certified connectors; virtual connectors; MCP connectors | ✅ | learn.microsoft.com connectors/admin docs |
| 40 | "All connectors are agent ready" | ✅ | learn.microsoft.com/connectors/overview |
| 41 | The "1,000+ connectors" number | ⚠ | Microsoft Learn states a "large ecosystem" but no fixed number on the overview page |
| 42 | Environment: container bound to an Entra tenant and a geography; data isolation between environments | ✅ | learn.microsoft.com admin environments-overview |
| 43 | Six environment types (Production, Default, Sandbox, Trial, Developer, Dataverse for Teams) | ✅ | learn.microsoft.com admin environments-overview |
| 44 | Default environment: undeletable, no manual backup, no backup guarantee, 3/3/1 GB included | ✅ | learn.microsoft.com admin environments-overview |
| 45 | Managed environments: premium capability set incl. CMK, Lockbox, IP firewall, VNet | ✅ | learn.microsoft.com admin managed-environment-overview |
| 46 | Environment groups: managed-environments-only; rules locked; no per-environment exceptions | ✅ | learn.microsoft.com admin environment-groups |
| 47 | DLP enforcement process (7 steps); design-time + runtime; up to 24-hour latency | ✅ | learn.microsoft.com admin wp-data-loss-prevention |
| 48 | Business/Non-business/Blocked connector classification labels | ⚠-knowledge | The *behaviour* is ✅; the exact label set is standard practice |
| 49 | CoE Starter Kit is no longer actively maintained; core capabilities in admin center (Inventory/Usage/Monitor/Actions) | ✅ | learn.microsoft.com guidance/coe/starter-kit (updated 2026-05-07) |
| 50 | Solutions are the ALM mechanism; managed vs unmanaged; managed properties; Git integration | ✅ | learn.microsoft.com ALM overview; solutions overview |
| 51 | Pipelines: sequential tamper-proof promotion; QA cannot be bypassed; managed-only deploy | ✅ | learn.microsoft.com/power-platform/alm/pipelines |
| 52 | All pipeline environments except host/dev must be managed; Feb 2026 auto-enablement | ✅ | learn.microsoft.com/power-platform/alm/pipelines |
| 53 | Power BI core is "a core component of Microsoft Fabric" sharing OneLake/Direct Lake/Purview | ✅ | learn.microsoft.com/power-bi/fundamentals/power-bi-overview |
| 54 | Licensing: user + capacity + consumption families; Power Apps per app retiring Jan 2, 2026 | ✅ | learn.microsoft.com admin pricing-billing-skus (updated 2026-09-08) |
| 55 | Pricing figures | ⚠ | Microsoft states terms can change; no prices asserted here |
| 56 | Dataverse capacity: database/file/log; no technical size limit; 85% soft-enforcement alerts | ✅ | learn.microsoft.com admin capacity-storage |
| 57 | HSBC uses Power Platform with CMK, VNet, Lockbox (story dated 27 May 2026) | ✅ story exists / ⚠ metrics | microsoft.com customer story (HSBC) |
| 58 | Axis Bank: CoE, 120+ solutions, 50,000+ employees, 40% latency reduction, 1,000+ trained | ✅ story claims / ⚠ independent metrics | microsoft.com customer story (Axis Bank, 24 Feb 2026) |
| 59 | Dynamics 365 apps use Dataverse; extension rights scoped to the licensed app; F&O needs Data Integrator | ✅ | learn.microsoft.com Dataverse intro; licensing page |
| 60 | Cymbal Bank persona: Paris HQ, ~€400bn, Singapore APAC hub, four businesses, MAS branch | ✅-repo convention | [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §10 |
| 61 | Cymbal Bank worked-example baselines, targets, and design (env strategy, DLP, CoE, ALM) | ⚠-analysis | Cymbal Bank's own design judgment (§11) |

---

### 12.1 What Could Not Be Verified

This pass had live web access but could not confirm the following; each is flagged honestly rather than asserted:

- **⚠ The "1,000+ connectors" figure.** Microsoft's connector documentation describes a "large ecosystem" and links a full reference list, but this pass found **no fixed number stated on the primary Learn overview page**. The figure is a recurring marketing claim whose value has drifted upward over time ("hundreds" at GA → four-figure totals later). The *ecosystem* and its *growth* are verified; the specific count is not, and should be taken from the live connector reference at evaluation time.
- **❌ Power Pages' exact launch day in 2022.** The product's existence, its SaaS/external-facing nature, and its "Power Apps Portals until 2022" lineage are verified; the precise 2022 launch date was not pinned to a primary source this pass (the year is ✅, the day is ❌).
- **⚠ The attended-versus-unattended RPA definition.** Microsoft's licensing page confirms the licence types exist, but this pass did not retrieve a single canonical Microsoft page defining attended vs unattended RPA; the substance in §5.3 is standard RPA practice applied to Power Automate (⚠-knowledge).
- **⚠ The literal "Business / Non-business / Blocked" DLP label set.** The *enforcement behaviour* (block, suspend, quarantine, disable connections, fail at runtime) is verified verbatim; the exact three-way label wording is standard practice and flagged ⚠-knowledge rather than quoted from Microsoft.
- **⚠ Dataverse's physical storage claim.** The "built on Microsoft Azure SQL" statement comes from Wikipedia **with a "citation needed" tag**; the Common Data Model lineage is verified from the product page, but the physical-storage detail is ⚠.
- **⚠ Microsoft customer-story metrics.** The HSBC and Axis Bank stories are verified as *documents* making the stated claims (✅ existence and quoted content); the banks' internal efficiency numbers (e.g., Axis Bank's 40% latency reduction) are vendor-published and cannot be independently audited from public sources. They are existence proofs of the pattern, not benchmarks.
- **❌ A systematic census of which banks run which Power Platform estates.** The banking chapter (§10) is architectural and evidence-based on public stories; it is deliberately **not** a vendor-installation census, and no such census was attempted.
- **⚠ The Cymbal Bank worked-example numbers.** The ~3,400 apps / ~9,000 flows baseline, the environment-group design, the connector classification table, and the outcome targets in §11 are Cymbal Bank's own design judgment (⚠-analysis), consistent with the verified Microsoft capabilities but not themselves verifiable facts. The connector names in the DLP table are illustrative.
- **⚠ Pricing.** No prices are asserted anywhere in this guide. Microsoft states plainly that licensing programs and product terms can change; the structural facts (§9.7) are verified as of the source pages' "last updated" dates (some in September 2026), and prices must be taken from the current Power Platform Licensing Guide.
- **⚠ The "Agent Builder" rebranding direction.** The AI-agent guides note Microsoft's progressive rebranding of Copilot Studio toward "Agent Builder" in 2026 messaging ([ai_llm/ai_agent_platform_selection_guide.md](ai_llm/ai_agent_platform_selection_guide.md) §1.2.3); this pass verified the Copilot Studio naming and harness model on Learn but did not independently pin the full rebranding timeline.
- **❌ Concrete pricing of Copilot Credits and AI Builder credit consumption rates.** Capacity-based meters exist (✅) but their per-unit rates are set in the licensing guide and change; not asserted here.

### 12.2 Sources Verified This Pass (URLs)

The following external sources were actually retrieved and verified during the 2026-09-10 research pass (all facts cited as ✅ trace to one of these; Wikipedia pages were used as cross-checks with their cited primary material):

- Power Platform hub: https://learn.microsoft.com/en-us/power-platform/
- Power Apps docs hub: https://learn.microsoft.com/en-us/power-apps/ ; canvas apps: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/getting-started ; model-driven apps: https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/model-driven-app-overview
- Power Automate docs hub: https://learn.microsoft.com/en-us/power-automate/ ; flow types: https://learn.microsoft.com/en-us/power-automate/flow-types ; desktop flows: https://learn.microsoft.com/en-us/power-automate/desktop-flows/introduction
- Power BI: https://learn.microsoft.com/en-us/power-bi/ ; what is Power BI: https://learn.microsoft.com/en-us/power-bi/fundamentals/power-bi-overview ; Fabric tutorial: https://learn.microsoft.com/en-us/power-bi/fundamentals/fabric-get-started
- Power Pages: https://learn.microsoft.com/en-us/power-pages/ ; what is Power Pages: https://learn.microsoft.com/en-us/power-pages/introduction
- Dataverse: https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-intro
- Copilot Studio: https://learn.microsoft.com/en-us/microsoft-copilot-studio/ ; what is Copilot Studio: https://learn.microsoft.com/en-us/microsoft-copilot-studio/fundamentals-what-is-copilot-studio
- AI Builder: https://learn.microsoft.com/en-us/ai-builder/
- Connectors: https://learn.microsoft.com/en-us/connectors/ ; connectors overview: https://learn.microsoft.com/en-us/connectors/overview
- Admin: https://learn.microsoft.com/en-us/power-platform/admin/ ; environments: https://learn.microsoft.com/en-us/power-platform/admin/environments-overview ; environment groups: https://learn.microsoft.com/en-us/power-platform/admin/environment-groups ; managed environments: https://learn.microsoft.com/en-us/power-platform/admin/managed-environment-overview ; data policies: https://learn.microsoft.com/en-us/power-platform/admin/wp-data-loss-prevention ; security roles: https://learn.microsoft.com/en-us/power-platform/admin/security-roles-privileges ; capacity: https://learn.microsoft.com/en-us/power-platform/admin/capacity-storage ; licensing: https://learn.microsoft.com/en-us/power-platform/admin/pricing-billing-skus
- ALM: https://learn.microsoft.com/en-us/power-platform/alm/overview-alm ; pipelines: https://learn.microsoft.com/en-us/power-platform/alm/pipelines ; solutions: https://learn.microsoft.com/en-us/power-apps/maker/data-platform/solutions-overview
- Adoption guidance (environment strategy): https://learn.microsoft.com/en-us/power-platform/guidance/adoption/environment-strategy
- CoE Starter Kit transition: https://learn.microsoft.com/en-us/power-platform/guidance/coe/starter-kit
- General availability announcement: https://blogs.microsoft.com/blog/2016/10/31/microsoft-powerapps-flow-generally-available-starting-tomorrow/
- Power Virtual Agents → Copilot Studio: https://www.microsoft.com/en-us/microsoft-copilot/blog/copilot-studio/microsoft-power-virtual-agents-now-part-of-microsoft-copilot-studio/
- Customer stories: https://www.microsoft.com/en/customers/story/26540-hsbc-microsoft-power-platform ; https://www.microsoft.com/en/customers/story/26090-axis-bank-limited-power-apps
- Wikipedia cross-checks: https://en.wikipedia.org/wiki/Microsoft_Power_Platform ; https://en.wikipedia.org/wiki/Microsoft_Power_BI ; https://en.wikipedia.org/wiki/Microsoft_Power_Automate
- Repo cross-checks: [financial_management_systems_guide.md](financial_management_systems_guide.md) §10 (Cymbal Bank persona, Dynamics 365); [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) §10 (persona)

---

## 13. Glossary

| Term | Meaning |
| --- | --- |
| **Agent** | In Copilot Studio, "an AI assistant that handles conversations and completes tasks," following instructions, drawing on knowledge, and using tools (§8.1) |
| **Agent flow (workflow)** | A drag-and-drop automation native to Copilot Studio, runnable standalone or attached to an agent as a tool (§8.1) |
| **AI Builder** | The Power Platform capability for adding AI models/prompts without coding or data-science skills (§8.4) |
| **ALM** | Application lifecycle management — requirements, development, testing, deployment, release management, and governance (§9.6) |
| **Attended RPA** | Robotic automation that runs on a human's desktop alongside the human (§5.3) |
| **Canvas app** | A Power Apps app built on a blank surface with drag-and-drop controls and Power Fx (§4.1) |
| **CDS** | Common Data Service — the pre-November-2020 name for Dataverse (§2.4) |
| **Cloud flow** | A Power Automate flow triggered automatically, instantly, or on a schedule (§5.1) |
| **CMK** | Customer Managed Key — customer-controlled encryption keys for Dataverse data (§9.3, §10.2) |
| **CoE Starter Kit** | The Power Platform Center of Excellence Starter Kit — governance templates now superseded by admin-center capabilities; no longer actively maintained (§9.5) |
| **Common Data Model** | The standardised business schema lineage Dataverse inherits, giving familiar standard tables (§7.2) |
| **Connector** | A strongly-typed representation of a RESTful API, exposing actions and triggers to apps/flows/agents (§8.5) |
| **Copilot Studio** | The low-code studio for building AI agents and workflows; successor to Power Virtual Agents (Nov 2023) (§8) |
| **Dataverse** | Microsoft's cloud data service storing data in tables for Power Platform and Dynamics 365 (§7.2) |
| **Default environment** | The single auto-created tenant environment every licensed user can make in; undeletable, no backup guarantee (§9.2) |
| **DLP / data policy** | A Power Platform policy that classifies and blocks connectors to prevent data exposure (§9.4) |
| **Desktop flow** | A Power Automate flow that automates tasks on the web or desktop — the RPA lane (§5.2) |
| **Environment** | A container for data, apps, flows, and agents, bound to an Entra tenant and a geography (§9.1) |
| **Environment group** | A logical collection of managed environments with published, locked rules applied at scale (§9.3) |
| **Fabric** | Microsoft Fabric — the analytics platform of which Power BI is a core component (§6.2) |
| **Flow** | The pre-November-2019 name for Power Automate (§2.3) |
| **Harness** | The engine that carries out an agent's or workflow's work in Copilot Studio (GitHub Copilot / standard / Copilot chat) (§8.3) |
| **Managed environment** | A premium tier of environment governance (sharing limits, solution checker, CMK, Lockbox, VNet, pipelines) (§9.3) |
| **Managed solution** | A packaged, deployable solution artefact used for promotion to test/production; recommended for nondevelopment environments (§9.6) |
| **MCP** | Model Context Protocol — a standard exposing tools/endpoints to agents; supported via MCP connectors and Copilot Studio (§8.5) |
| **Model-driven app** | A Power Apps app built by composing components over a Dataverse data model; Dataverse-only (§4.2) |
| **Power Apps** | The low-code app-building product of the suite (§4) |
| **Power Automate** | The workflow-automation product of the suite (§5) |
| **Power BI** | The business-analytics product of the suite; a Fabric component (§6) |
| **Power Fx** | The low-code formula language spanning the platform's canvas/Power Pages/plug-in logic (§1.1) |
| **Power Pages** | The low-code SaaS platform for external-facing websites, over Dataverse (§7.1) |
| **Pipeline (Power Platform)** | The in-product CI/CD feature promoting solutions through stages with enforced sequencing and approvals (§9.6) |
| **Power Virtual Agents** | The pre-November-2023 name for Copilot Studio (§2.6) |
| **RPA** | Robotic process automation — automating repetitive tasks via desktop flows (§5.2) |
| **Semantic model** | The reusable, governed data-and-measure layer (formerly "dataset") that Power BI reports consume (§6.1) |
| **SoD** | Segregation of duties — no single person initiates and approves (ERM guide §4) |
| **Solution** | The transport unit for apps, flows, tables, and other components across environments; the ALM mechanism (§9.6) |
| **Unattended RPA** | Robotic automation that runs without a human present, on a schedule or queue (§5.3) |
| **Unmanaged solution** | The editable development-layer solution; not recommended for deployment to nondevelopment environments (§9.6) |
| **VNet support** | Power Platform virtual-network integration for private-network data egress control (§9.3, §10.2) |

---

## 14. Closing

Microsoft Power Platform is best understood not as five products sharing a logo but as one low-code platform with a single data layer, a single connector fabric, and a single governance surface — which is why it can be, for the right organisation, a genuine release of trapped capability, and why, for the wrong one, it can be a governance liability at the speed of self-service. The component map is coherent: Power Apps turns a process into an app, Power Automate turns a repetition into a flow, Power BI turns data into a decision, Power Pages turns it outward to customers and partners, Dataverse holds it all in one governed place, and Copilot Studio adds an agent that can reason over it. The 2015–2023 timeline traced in §2 — Crescent to Power BI, PowerApps and Flow to general availability in 2016, Flow to Power Automate, Common Data Service to Dataverse, Power Pages, and Power Virtual Agents into Copilot Studio — is a decade of a platform consolidating toward one coherent story. Sitting beside Dynamics 365 over a shared Dataverse, and on an Azure foundation, it is the extension layer of the Microsoft business-application estate rather than a competitor to it. Chapter by chapter, the deep-dive facts are the platform's real texture: canvas versus model-driven apps, cloud versus desktop flows, semantic models over Fabric, Dataverse's role-and-rule security, Copilot Studio's harnesses, and — counterweight to the whole promise — the governance machinery of environments, managed environments, environment groups, DLP policies, the CoE Starter Kit and its migration into the product, and the ALM discipline of managed solutions and pipelines. For a bank, none of this is optional: the shadow-IT risk is real, the regulated perimeter does not care how an artefact was built, and the public evidence — HSBC hardening the platform with CMK, VNet support, and Lockbox; Axis Bank running a CoE, an ALM setup, and citizen development at scale — shows both the pattern and the possibility. The Cymbal Bank example is the same lesson told as a programme: route makers to personal environments, lock the default environment, classify and block connectors before an incident forces the question, re-baseline off the retiring CoE toolkit onto the admin center, and promote nothing to production except a managed solution that has provably passed its gates. Do that, and the platform stops being a risk to be contained and becomes a capability to be steered. The organisations that win with it are not the ones that block the business from building — they are the ones that give the business a safe place to build, and then let it run — because that, in the end, is the low-code lever.
