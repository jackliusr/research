# Power Platform CI/CD: The Deployed Solution — Solutions, Source Control, Pipelines, the pac CLI, Azure DevOps and GitHub Actions, Quality Gates, and Deployment Governance

**Abstract.** This guide is the deployment-engineering deep dive that sits underneath the platform overview. Where [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) maps what the Power Platform *is* and how it is governed at platform level, this guide takes one question and follows it all the way down: **how does a Power Platform solution actually get built, versioned, tested, and promoted from a maker's development environment into a bank's production estate without breaking an audit?** It treats application lifecycle management (ALM) as an engineering discipline rather than a licensing footnote. It works through the **solution** as the unit of deployment — managed versus unmanaged, the layer model, components and dependencies, the publisher and prefix, and the create/update/upgrade/patch lifecycle — then the **source-control strategy** Microsoft now documents (solution source control versus developer source control, the evolving Dataverse Git integration, and the classic solution pack/unpack flow). It documents the **build and release tooling** at primary-source fidelity, quoting the actual task and action names: the Microsoft Power Platform Build Tools task family for Azure DevOps, the GitHub Actions for Microsoft Power Platform, and the **`pac` CLI** command groups. It covers **Power Platform Pipelines** — the native, in-product hosted deployment path — as the engineering *around* them rather than a re-derivation of the feature. It deep-dives the **deployment settings file**, the mechanism that makes environment variables and connection references work across a build-once-deploy-many pipeline, and the **service principal / application user** authentication model behind it. The middle chapters cover **quality gates**: the Power Apps **solution checker**, the Power Apps **Test Engine**/**Test Studio** automated-testing stack, and the validation and pre-deployment checks the pipeline performs. The governance chapter covers deployment history and audit, the who-deployed-what-when evidence chain, rollback and recovery semantics, and the CoE Starter Kit transition. A **banking chapter** situates the whole thing in change-management and separation-of-duties terms, and a **Cymbal Bank worked example** rolls out a complete CI/CD pipeline programme — branch strategy, stages, gates, environment-variable/connection-reference design, and a rollback drill. It closes with a claims audit, a "What Could Not Be Verified" ledger, and a glossary. **Dedup note:** this guide exists because the technology library had no dedicated CI/CD or DevOps/release-engineering deep dive — the ground was previously owned in fragments by §9.6 of the platform guide and §7 of the Copilot Studio agent-engineering guide; both are cross-referenced, not repeated.

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Technology / Power Platform ALM & Deployment Engineering — Solutions and the Layer Model, Source Control and Git Integration, Azure DevOps Build Tools, GitHub Actions, the `pac` CLI, Power Platform Pipelines, Deployment Settings, Service-Principal Authentication, the Solution Checker and Test Engine, Deployment Audit and Rollback, with a Banking / Regulated-Context Angle  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Primary Sources:** learn.microsoft.com (the ALM overview, ALM basics, solution concepts, solution layers, the environment strategy guidance, Dataverse Git integration, the Power Platform Build Tools and Build Tools task reference, GitHub Actions for Power Platform, the `pac` CLI command-group and command reference, the pipelines overview, delegated deployments setup, the connection-reference/environment-variable deployment-settings article, the solution-checker and Test Engine articles, the application-user/service-principal documentation, and the CoE Starter Kit docs), Microsoft Learn training, the Power Platform release planner, the public CoE Starter Kit and ALM Accelerator GitHub repositories, and Wikipedia as cross-check. NOTE: this pass had live web access; facts were checked against primary sources on 2026-09-10 where possible; anything not verified is flagged ⚠ or ❌ and logged in §13.  
> **Last Updated:** September 2026  
> **Companion guides (repo-relative links):** platform anatomy, environments, managed environments, DLP and licensing → [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) §9; CI/CD as a single Copilot Studio responsibility → [ai_llm/copilot_studio_agent_engineering_knowledge_guide.md](ai_llm/copilot_studio_agent_engineering_knowledge_guide.md) §7; integration context → [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md); audit evidence chains → [audit_as_code_guide.md](audit_as_code_guide.md); test orchestration → [test_orchestration_guide.md](test_orchestration_guide.md); data deployment and environment tiers → [data/dataops_guide.md](data/dataops_guide.md); cloud and region context → [cloud_providers_guide.md](cloud_providers_guide.md); enterprise architecture and the operating model → [architecture/enterprise_architecture_guide.md](architecture/enterprise_architecture_guide.md); operational resilience and change management → [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md); enterprise risk and the three lines of defence → [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md); MAS technology-risk expectations → [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md); IT strategy and platform policy → [../management/it_strategy_guide.md](../management/it_strategy_guide.md); vendor and support discipline → [../management/vendor_management_guide.md](../management/vendor_management_guide.md).

---

**How to read this document.** §1–§2 are the model: the solution as the unit of deployment and the source-control strategy that wraps it. §3–§6 are the tooling: build and release pipelines (§3), the `pac` CLI (§4), native Power Platform Pipelines (§5), the deployment settings mechanism (§6), and authentication (§7). §8–§9 are quality: the solution checker and the automated-testing stack. §10 is validation and pre-deployment checks. §11 is governance and operations — deployment history, audit, rollback, and the CoE tooling transition. §12 is the banking/regulated context, §13 the Cymbal Bank worked example, §14 the combined claims-audit and verification ledger plus glossary and closing. **Completeness conventions:** ✅ = verified this pass against a primary or named source; ⚠ = approximate / vendor claim / single secondary source / documentation still in flux; ⚠-knowledge = well-established industry practice not re-verified as a product claim this pass; ❌ = could not be verified. Cross-references follow repo convention: same-directory guides by plain filename, `../banking/...` for banking guides, `../management/...` for management guides, and `data/...`, `ai_llm/...`, `architecture/...` for those sub-folders. No task name, action name, or CLI subcommand in this guide is invented; every one is quoted from the Microsoft reference and anything unconfirmed is flagged rather than asserted.

---

## Table of Contents

1. [The ALM Model on the Power Platform](#1-the-alm-model-on-the-power-platform)
   - 1.1 [Solutions as the Unit of Deployment](#11-solutions-as-the-unit-of-deployment)
   - 1.2 [Managed versus Unmanaged, and the Layer Model](#12-managed-versus-unmanaged-and-the-layer-model)
   - 1.3 [Components, Dependencies, and the Publisher](#13-components-dependencies-and-the-publisher)
   - 1.4 [The Solution Lifecycle: Create, Update, Upgrade, Patch](#14-the-solution-lifecycle-create-update-upgrade-patch)
   - 1.5 [Environments and the Environment Strategy](#15-environments-and-the-environment-strategy)
2. [Source Control and the Repository Strategy](#2-source-control-and-the-repository-strategy)
   - 2.1 [Solution Source Control versus Developer Source Control](#21-solution-source-control-versus-developer-source-control)
   - 2.2 [Solution-Aware versus Non-Solution-Aware Repositories](#22-solution-aware-versus-non-solution-aware-repositories)
   - 2.3 [Pack and Unpack: The Classic File Representation](#23-pack-and-unpack-the-classic-file-representation)
   - 2.4 [Dataverse Git Integration in the Maker Portal](#24-dataverse-git-integration-in-the-maker-portal)
   - 2.5 [Branching and Team Development](#25-branching-and-team-development)
3. [Build and Release Pipelines: Build Tools for Azure DevOps and GitHub Actions](#3-build-and-release-pipelines-build-tools-for-azure-devops-and-github-actions)
   - 3.1 [The Four Task Categories and the Tool Installer](#31-the-four-task-categories-and-the-tool-installer)
   - 3.2 [The Azure DevOps Task Reference](#32-the-azure-devops-task-reference)
   - 3.3 [GitHub Actions for Microsoft Power Platform](#33-github-actions-for-microsoft-power-platform)
   - 3.4 [Which Tool, When](#34-which-tool-when)
4. [The pac CLI: The Command Surface](#4-the-pac-cli-the-command-surface)
   - 4.1 [The Command Groups](#41-the-command-groups)
   - 4.2 [pac solution in Detail](#42-pac-solution-in-detail)
   - 4.3 [pac pipeline, pac admin, and the Rest](#43-pac-pipeline-pac-admin-and-the-rest)
5. [Power Platform Pipelines: The Native Hosted Path](#5-power-platform-pipelines-the-native-hosted-path)
   - 5.1 [The Capability and the Host](#51-the-capability-and-the-host)
   - 5.2 [Stages, Delegated Deployment, and Approvals](#52-stages-delegated-deployment-and-approvals)
   - 5.3 [Sequential-and-Tamper-Proof Artifacts and Prevalidation](#53-sequential-and-tamper-proof-artifacts-and-prevalidation)
   - 5.4 [Which Environments Must Be Managed](#54-which-environments-must-be-managed)
6. [Deployment Settings: Environment Variables and Connection References](#6-deployment-settings-environment-variables-and-connection-references)
   - 6.1 [Why They Exist: The Build-once-Deploy-many Problem](#61-why-they-exist-the-build-once-deploy-many-problem)
   - 6.2 [The Deployment Settings JSON](#62-the-deployment-settings-json)
   - 6.3 [Generating and Consuming the File](#63-generating-and-consuming-the-file)
7. [Authentication: Service Principals, Application Users, and Secrets](#7-authentication-service-principals-application-users-and-secrets)
   - 7.1 [The Three Connection Types](#71-the-three-connection-types)
   - 7.2 [The Setup Path](#72-the-setup-path)
   - 7.3 [`pac admin create-service-principal` and the Admin Steps](#73-pac-admin-create-service-principal-and-the-admin-steps)
8. [Quality Gates: The Solution Checker](#8-quality-gates-the-solution-checker)
   - 8.1 [What It Is and the Rule Model](#81-what-it-is-and-the-rule-model)
   - 8.2 [Running It: Portal, Pipeline, and CLI](#82-running-it-portal-pipeline-and-cli)
9. [Automated Testing: Test Engine, Test Studio, Regression and UAT](#9-automated-testing-test-engine-test-studio-regression-and-uat)
   - 9.1 [The Power Apps Test Engine](#91-the-power-apps-test-engine)
   - 9.2 [Maturity, Licensing, and Limits](#92-maturity-licensing-and-limits)
   - 9.3 [Regression and UAT Practice](#93-regression-and-uat-practice)
10. [Validation and Pre-Deployment Checks](#10-validation-and-pre-deployment-checks)
11. [Governance and Operations: History, Audit, Rollback, and CoE Tooling](#11-governance-and-operations-history-audit-rollback-and-coe-tooling)
12. [The Banking and Regulated Context](#12-the-banking-and-regulated-context)
13. [Cymbal Bank Worked Example: A CI/CD Rollout](#13-cymbal-bank-worked-example-a-cicd-rollout)
14. [Claims Audit, Glossary, and Closing](#14-claims-audit-glossary-and-closing)

---

## 1. The ALM Model on the Power Platform

### 1.1 Solutions as the Unit of Deployment

**Verified (✅).** Microsoft's ALM overview states the object plainly: *"Solutions are the mechanism for implementing ALM; you use them to distribute components across environments through export and import. A component represents an artifact used in your application and something that you can potentially customize. Anything that can be included in a solution is a component, such as tables, columns, canvas and model-driven apps, Power Automate flows, agents, charts, and plug-ins."* The same page names the four concepts that make up the model: solutions, Dataverse (which *"stores all the artifacts, including solutions and in-product deployment pipelines"*), source control (*"should be your source of truth for storing and collaborating on your components"*), and a CI/CD platform such as Azure DevOps that automates the build-test-deploy pipeline (learn.microsoft.com/power-platform/alm/overview-alm).

**Verified (✅).** Microsoft also defines ALM itself as spanning *"requirements management, software architecture, development, testing, maintenance, change management, support, continuous integration, project management, deployment, release management and governance"*, divided into three key areas — governance, application development, and maintenance — cycling through *"plan and track, develop, build and test, deploy, operate, monitor, and learn from discovery"* (learn.microsoft.com/power-platform/alm/overview-alm). The deployment-engineering reader should note that the **deployment** discipline this guide covers is one slice of that cycle; the governance discipline is the slice that turns a working pipeline into an auditable one.

**Verified (✅).** Microsoft's ALM-basics article adds two boundaries that matter for design. First, a solution *"include[s] metadata and certain tables with configuration data. Solutions don't contain any business data."* Second, *"not all tables can be included in a solution. For example, the Application User, Custom API, and Organization Setting system tables can't be added to a solution"* (learn.microsoft.com/power-platform/alm/basics-alm). The first boundary is why a "deploy my data too" request is a data-migration project, not a pipeline; the second is why some configuration must be re-created per environment by pipeline steps rather than carried in the artifact.

**Verified (✅).** A solution carries a hard size ceiling: *"A solution can be up to 95 MB in size"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). When a single monolithic solution approaches that ceiling, the documented answer is the multi-solution pattern — see §2.

### 1.2 Managed versus Unmanaged, and the Layer Model

**Verified (✅).** Microsoft states the distinction as a *role* distinction, not a checkbox: *"Unmanaged solutions are developed… used in development environments while you make changes to your application"*; *"Managed solutions are deployed… deployed to any environment that isn't a development environment for that solution. These environments include test, user acceptance testing (UAT), system integration testing (SIT), and production environments"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). Three operational consequences follow directly from the docs:

- *"You can't edit components directly within a managed solution."* To edit a managed component you must first add it to an unmanaged solution, which *"create[s] a dependency between your unmanaged customizations and the managed solution"* — and *"the managed solution can't be uninstalled until you remove the dependency."*
- *"You can't export a managed solution. But you can export an unmanaged solution as managed."*
- *"When a managed solution is deleted (uninstalled), all the customizations and extensions included with it are removed"*, including data stored in custom tables and columns that are part of the managed solution.

**Verified (✅)** — and this is the rule that most often bites a naive dev→test promotion: *"You can't import a managed solution into the same environment that contains the originating unmanaged solution. To test a managed solution, you need a separate environment to import it into"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). The lifecycle is therefore inherently multi-environment; a "test in place" shortcut is not supported.

**Verified (✅).** Exported unmanaged versions are the source of truth for source control: *"Exported unmanaged versions of your solutions should be checked into your source control system"*, and *"As an ALM best practice, managed solutions should be generated by exporting an unmanaged solution as managed and considered a build artifact"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). This is the single most important sentence in Power Platform CI/CD: **the managed artifact is a build output, not a hand-made file.**

**Verified (✅).** The **layer model** explains why unmanaged drift in a downstream environment is dangerous. Dataverse has two distinct layer levels: an *"Unmanaged layer. All imported unmanaged solutions and unmanaged customizations exist at this layer. The unmanaged layer is a single layer"*, and *"Managed layers. All imported managed solutions and the system solution exist at this level"* (learn.microsoft.com/power-apps/maker/data-platform/solution-layers). Layers stack — *"the last one installed is above the managed solution installed previously"* — and conflicting definitions resolve by *"'Last one wins' or a merge logic"*. Uninstalling a managed solution reveals the layer beneath; uninstalling all of them falls back to the system layer, which *"contains the tables and components that are required for the platform to function."*

**Verified (✅).** Critically for a production target: *"Unmanaged customizations reside at the top layer for a component and subsequently define the runtime behavior of the component. In most situations you don't want unmanaged customizations determining the behavior of your components in your test and production environments"* (learn.microsoft.com/power-apps/maker/data-platform/solution-layers). A production environment that has collected unmanaged customizations is running a version of the solution that **does not exist in source control** — the audit's worst case. The docs provide the **See solution layers** feature to enumerate layers for a component (*Advanced > See solution layers*) and a **Remove active customizations** command to drop an unmanaged layer, with the warning that *"Removing active unmanaged customizations can't be reversed or undone."*

### 1.3 Components, Dependencies, and the Publisher

**Verified (✅).** A component is *"something that you can potentially customize"*; components nest — *"a table contains forms, views, charts, columns, tables relationships, messages, and business rules… Except for choice columns, all other columns can't exist outside of a table"* — so a column is *dependent* on its table (learn.microsoft.com/power-platform/alm/solution-concepts-alm). The purpose of modelling components and their dependencies is stated as being able to *"keep track of any limitations on what can be customized using managed properties and all the dependencies so that it can be exported, imported, and (in managed solutions) deleted without leaving anything behind."*

**Verified (✅).** **Cross-solution dependencies** exist: some managed solutions depend on components in other managed solutions, *"Some solution publishers take advantage of this to build solutions that are modular."* The platform enforces them — *"If you try to install a solution that requires a base solution that isn't installed, you won't be able to install the solution"*, and you must uninstall the dependent solution before the base (learn.microsoft.com/power-platform/alm/solution-concepts-alm). Dependency tracking is a first-class platform service (learn.microsoft.com/power-platform/alm/dependency-tracking-solution-components).

**Verified (✅).** **The publisher is the ownership boundary.** *"Every app and other solution components… is part of a solution. Because every solution has a publisher, you should create your own publisher rather than use the default."* The publisher *"is considered the owner of that component"*, and *"Once you introduce a publisher for a component in a managed solution, you can't change the publisher for the component. Because of this restriction, it's best to define a single publisher so you can change the layering model across solutions later"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm).

**Verified (✅).** The **prefix** is the collision-avoidance device: *"The publisher prefix is a mechanism to help avoid naming collisions. This allows for solutions from different publishers to be installed in the same environment with few conflicts."* The docs warn: *"When you change a solution publisher prefix, you should do it before you create any new apps or metadata items because you can't change the names of metadata items after they're created"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). In a bank, the prefix is a naming standard, not a preference — set it once, at the start, and refuse to let it drift.

### 1.4 The Solution Lifecycle: Create, Update, Upgrade, Patch

**Verified (✅).** Microsoft documents four lifecycle actions with precise semantics (learn.microsoft.com/power-platform/alm/solution-concepts-alm):

| Action | Documented semantics | Can it delete components? |
| --- | --- | --- |
| **Create** | *"Author and export unmanaged solutions."* | n/a |
| **Update** | *"Create updates to a managed solution that are deployed to the parent managed solution."* | **No** — *"You can't delete components with an update."* |
| **Upgrade** | *"Import the solution as an upgrade to an existing managed solution, which removes unused components and implements upgrade logic."* Roll up (merge) patches into a new version. | **Yes** — *"Solution upgrades delete components that existed but are no longer included in the upgraded version."* |
| **Patch** | *"A patch contains only the changes for a parent managed solution, such as adding or editing components and assets."* Layered on top of the parent. | **No** — *"You can't delete components with a patch."* |

**Verified (✅).** The ALM-basics article restates the same semantics operationally: updates *"don't create an additional solution layer"*; a patch is for *"small updates (similar to a hotfix)"* and *"when patches are imported, they're layered on top of the parent solution"*; and *"Upgrading a solution installs a new solution layer immediately above the base layer and any existing patches"* — *"Applying solution upgrades involves deleting all existing patches and the base layer"* (learn.microsoft.com/power-platform/alm/basics-alm). The deployment-engineering takeaway: **update for the common daily promotion (same component set), upgrade when the component set changed or you want to compact the layer stack, patch only as a deliberately reversible hotfix.** Every one of these is a first-class option on the import task and CLI (§3, §4).

### 1.5 Environments and the Environment Strategy

**Verified (✅).** Microsoft's ALM-basics article enumerates the environment types used in ALM (learn.microsoft.com/power-platform/alm/basics-alm):

- **Sandbox** — *"any nonproduction environment of Dataverse… the place to safely develop and test application changes with low risk"*, and it *"include[s] capabilities that would be harmful in a production environment, such as reset, delete, and copy operations."*
- **Production** — *"The environment where apps and other software are put into operation for their intended use."*
- **Developer** (formerly *Community*) — *"a single-user environment, and can't be used to run or share production apps."*
- **Default** — *"A single default environment is automatically created for each tenant and shared by all users."*

**Verified (✅).** The same article gives the access table that a bank should treat as a control: development is for *"App makers and developers"* with *"App users shouldn't have access"*; test is for *"Admins and people who are testing"* with *"App makers, developers, and production app users shouldn't have access"*; production is for *"Admins and app users"* with *"App makers and developers shouldn't have access, or should only have user-level privileges"* (learn.microsoft.com/power-platform/alm/basics-alm). Those separations are exactly what a separation-of-duties auditor looks for (§12).

**Verified (✅).** The **promotion pattern** is documented as distribution *"from dev to test environments"* — makers *"work in development environments using unmanaged solutions, then import them to other downstream environments—such as test—as managed solutions"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). Microsoft's ALM-environment-strategy article (learn.microsoft.com/power-platform/alm/environment-strategy-alm) is the companion for sizing that pattern.

**Verified (✅).** The **tenant environment strategy** guidance — the newer, product-aligned view — introduces several capabilities that a CI/CD programme must plan around (learn.microsoft.com/power-platform/guidance/adoption/environment-strategy):

- **Managed environments** *"activate a set of premium capabilities that enable greater visibility and control"*, and the article frames environment-group rules as the way to apply *"consistent governance and security policies."*
- **Default environment routing** *"automatically moves makers away from creating resources in the default environment to their own personal environment"* — the *"developer-environment model"* promoted at enterprise scale.
- **Environment groups** let *"newly created maker environments… be automatically added to a group that applies rules."*
- **Preferred solutions** *"help makers ensure that all the assets they build are in a Dataverse solution, making it easier to promote them to other environments."*
- **Pipelines in Power Platform** *"provide… a simplified process for promoting assets from development to test and production environments, making continuous integration and deployment (CI/CD) available to all makers."*
- The article is explicit that a strategy *"should evolve over time"* and that organisations should *"prioritize the built-in features of the platform when possible, instead of building your own tools."*

**Verified (✅).** The environment-strategy article also states Microsoft's own intent for the developer-environment model: makers should build in *"their own, personal environment where they can safely build their apps protected from makers working on unrelated assets"*, and *"When the resource is ready to be shared, the maker can use the continuous integration attached to the developer environment to promote it to a production environment"* (learn.microsoft.com/power-platform/guidance/adoption/environment-strategy). **⚠ Flag — evolving guidance.** Microsoft explicitly describes this article as *"vision"* text that *"aligns with Microsoft's latest product direction and uses current features and near-term planned enhancements."* Treat the environment-model guidance as directional and re-read it each quarter; the platform guide's §9.1 and §9.3 are the condensed cross-reference for the environment and managed-environment anatomy and are not re-derived here.

**Cross-reference.** Environment types, managed environments, environment groups, the default environment, and the licensing of premium governance are owned by [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) §9.1–§9.3 and §9.7. This guide assumes them and focuses on what the deployment pipeline does with them.

## 2. Source Control and the Repository Strategy

### 2.1 Solution Source Control versus Developer Source Control

**Verified (✅).** Microsoft's ALM model names source control as *"your source of truth for storing and collaborating on your components"* (learn.microsoft.com/power-platform/alm/overview-alm), and its ALM-basics article defines the discipline: *"A source control system helps organizations achieve healthy ALM because the assets maintained in the source control system are the 'single source of truth'—or, in other words, the single point of access and modification for your solutions"* (learn.microsoft.com/power-platform/alm/basics-alm). The docs distinguish two ways to get solution source into that repository — the classic **solution source control** flow (export, unpack, commit) and the newer **developer source control** flow (native Git integration) — and this guide keeps the same names.

**Verified (✅).** The classic flow is documented as a repeatable loop. For an initial solution: *"Export the unmanaged solution. Using the Solution Packager tool, extract the solution into component files. From those extracted component files, add the necessary files to source control. Submit these changes to source control."* For modification: *"Synchronize or get the latest solution component file sources… pack component files into an unmanaged solution .zip file… Import the unmanaged solution file into an environment… Customize and edit… Export… extract… Synchronize… Reconcile if any conflicts exist… Submit"* (learn.microsoft.com/power-platform/alm/use-source-control-solution-files). Two ordering rules are called out explicitly: *"Steps 2 and 3 must be done before further customizations occur in the development organization"*, and within the submit step, *"step b must be completed before step c."*

### 2.2 Solution-Aware versus Non-Solution-Aware Repositories

**Verified (✅).** The distinction that a repository reorganisation hinges on is whether the applications in it are *solution-aware*. Microsoft's Build Tools FAQ is explicit: *"Yes, flows and canvas apps are solution aware, so if you add these components to your solution, they can participate in the lifecycle of your app"* (learn.microsoft.com/power-platform/alm/devops-build-tools). Solution-aware means the object can be added to a solution and therefore versioned, exported, packed, and imported as a unit; **non-solution-aware** objects (for example, objects left in the default solution, or unsupported system tables) fall outside the pipeline and must be handled out-of-band or avoided. The ALM-basics article reinforces the boundary from the other direction: *"not all tables can be included in a solution. For example, the Application User, Custom API, and Organization Setting system tables can't be added to a solution"* (learn.microsoft.com/power-platform/alm/basics-alm).

**Verified (✅) — solution organisation strategy.** Microsoft documents three strategies, ordered simple to complex (learn.microsoft.com/power-platform/alm/organize-solutions):

| Strategy | Shape | Recommended for |
| --- | --- | --- |
| **Single solution** | *"All customizations are grouped into one unmanaged solution during development, which is later exported as a single managed solution for deployment."* | *"Small-medium scale implementations"*, *"Scenarios where future modularization is unlikely"* |
| **Multiple solutions, one dev environment** | *"Multiple unmanaged solutions are maintained within a single development environment, each typically dedicated to unrelated features or modules."* | *"Small-medium scale implementations with distinct and independent functional areas that don't share components."* |
| **Multiple solutions, dedicated dev environments** | *"developing each unmanaged solution in its own isolated Dataverse development environment… A base solution containing common components… is created and deployed as a managed solution into each app-specific development environment."* | *"Large-scale enterprise projects"*, *"Teams with multiple developers or partners"*, *"Scenarios requiring strict governance and CI/CD pipelines."* |

**Verified (✅).** The dedicated-environment strategy carries the governance rules a bank should treat as hard constraints: *"use the same publisher for all your solutions across your environments"*; *"Don't include the same unmanaged component in more than one solution"*; *"Have only one solution that includes all your tables"*; *"Use only one solution publisher"*; and *"Avoid creating dependencies between solutions."* The docs explain the failure mode of violating the last rule — *"if you have one solution for tables and another for cloud flows, and a flow relies on a custom column, it works in development because the column exists. However, if only the cloud flow solution is imported into the target environment, the import process might not recognize the dependency on the custom column. As a result, the flow solution installs successfully, but the flow itself doesn't work"* (learn.microsoft.com/power-platform/alm/organize-solutions). **This partial-deploy failure mode is a distinct risk from a hard import failure and is invisible to a green pipeline — it must be caught by post-deploy smoke tests (§10).**

**Verified (✅).** The layer-building sequence is documented as: build a base solution of common tables, export as managed; stand up a second dev environment; import the managed base; create an unmanaged app-layer solution on top; extend; export as managed; *"In your production environment, you import the managed base layer and then import the managed app layer. This creates two managed layers in the environment with clear dependencies between the managed solutions"* (learn.microsoft.com/power-platform/alm/organize-solutions).

### 2.3 Pack and Unpack: The Classic File Representation

**Verified (✅).** The **SolutionPackager** tool *"can reversibly decompose a Microsoft Dataverse compressed solution file into multiple XML files and other files. You can then easily manage these files by using a source control system"* (learn.microsoft.com/power-platform/alm/solution-packager-tool). ⚠ **Status flag:** Microsoft now states *"The SolutionPackager tool is no longer the recommended way to unpack and pack solutions. The capabilities of the SolutionPackager tool are incorporated into the Power Platform CLI"* — the `pac solution` command's `unpack`, `pack`, `clone`, and `sync` verbs *"incorporate the same underlying capabilities."* New work should prefer the CLI.

**Verified (✅).** SolutionPackager supports **two folder layouts** (learn.microsoft.com/power-platform/alm/solution-packager-tool):

| | XML format (legacy) | YAML source control format |
| --- | --- | --- |
| Solution manifest | `Other\Solution.xml` + `Other\Customizations.xml` | `solutions/<name>/solution.yml` and supporting YAML files |
| Multi-solution repo | Not supported | Supported — *"multiple solutions… share one folder"* |
| Canvas apps (`.msapp`) | Not supported | Supported |
| Modern flows | Not supported | Supported |
| Native Git integration | Not used | *"Always used — Git integration always writes YAML"* |

**Verified (✅).** The YAML layout is *"autodetected by the presence of a `solutions/` subfolder containing `*solution.yml` files"*; if the manifests sit at the root instead, *"the tool falls back to the XML path and reports a misleading error about a missing `Customizations.xml`"* (learn.microsoft.com/power-platform/alm/solution-packager-tool). The documented guidance is unambiguous: *"For all new projects, and whenever you use native Dataverse Git integration"* use YAML; use XML *"Only when working with existing repositories that already use the XML format, or when using legacy tooling that doesn't support YAML"* (learn.microsoft.com/power-platform/alm/use-source-control-solution-files). **⚠ Flag — the guidance is actively migrating to YAML**; a repository started in XML format in 2023 should not assume it can stay there indefinitely.

**Verified (✅).** One further control matters for source control: *"Except for the sections described in When to edit the customizations file, manual editing of extracted component files and .zip files isn't supported"* (learn.microsoft.com/power-platform/alm/use-source-control-solution-files). A reviewer who "fixes" an unpacked file by hand is outside the supported surface and can produce an artifact that packs but does not round-trip faithfully.

### 2.4 Dataverse Git Integration in the Maker Portal

**Verified (✅).** The newer, product-native source-control path is **Dataverse Git integration**. Microsoft describes it as enabling *"development teams to sync solutions and solution objects across one or more Microsoft Dataverse environments by using a supported Git provider, such as Azure DevOps or GitHub"*, with *"the solutions experiences natively provide the source control integration functionality"* so that *"citizen developers, code-first developers, and administrators can all benefit from version control, change tracking, and seamless team collaboration"* (learn.microsoft.com/power-platform/alm/git-integration/overview). The placement rule is explicit: *"Use Git integration with developer environments, not in your test or production environments. Use builds to create solution artifacts and pipelines in Power Platform to deploy."*

**Verified (✅).** The documented benefits map directly to CI/CD outcomes (learn.microsoft.com/power-platform/alm/git-integration/overview): **source control as source of truth**; **safety, auditing, and compliance using SDLC best practices** (*"version control, code reviews, and static source code analysis"* plus *"auditing, compliance, and traceability"*); **short-lived dev environments** (*"you can rehydrate development environments from source control quickly and easily"*); **fusion development teams**; and **protection** (*"you can restore to a previous state or version"*).

**Verified (✅).** The managed/unmanaged split under Git integration is stated the same way as the rest of the model: *"solutions stored in source control come from unmanaged solutions in a maker's environment… Managed solutions are built from source control and deployed into downstream environments… and aren't editable in those environments"* (learn.microsoft.com/power-platform/alm/git-integration/overview). On file representation, *"the solution objects in source control are no longer duplicated for each solution. Instead, the solution objects are stored in a single location and can be shared across multiple solutions in the same repository and folder."*

**Verified (✅) — the code-first trap.** For components like PCF controls and plug-ins, Microsoft warns against deploying built binaries directly into an unmanaged solution: doing so means *"only their compiled (built) version is stored in source control… you end up with two copies of the object in source control"*, and *"Storing binaries in your repository can lead to confusion and potential conflicts if the source code and the built version aren't kept in sync. This practice isn't recommended."* The recommended pattern is to *"build code-first objects as part of a solution build process and import the generated unmanaged solution into the maker environment"*, creating *"Azure Pipelines or GitHub workflows… that create artifacts for pipelines in Power Platform"* (learn.microsoft.com/power-platform/alm/git-integration/overview).

### 2.5 Branching and Team Development

**Verified (✅).** Microsoft states that *"Nearly every source control system has some form of branching and merging support"* and names *"trunk-based branching, release branching, and feature branching"* as common strategies, pointing to Azure DevOps' *"Adopt a Git branching strategy"* guidance (learn.microsoft.com/power-platform/alm/basics-alm). **⚠-knowledge:** which strategy to pick is practice, not product — this guide's Cymbal Bank example (§13) uses a trunk-plus-release-branch model as an illustrative choice, not a Microsoft mandate.

**Verified (✅).** The **team-development constraint** is stated frankly: *"working with configurations in a solution system can create challenges for team development. Organizations must orchestrate changes from multiple developers to avoid merge conflicts as much as possible, because source control systems have limitations on how merges occur. We recommend that you avoid situations where multiple people make changes to complex components—such as forms, flows, and canvas apps—at the same time"* (learn.microsoft.com/power-platform/alm/basics-alm). The source-control article shows the failure and the reconcile path in detail: when two developers touch the *same* component file, *"The correct process… Before developer A submits to source control, they must get latest sources… Before developer B submits… get the latest sources… There's a conflict… Developer B must reconcile the conflict"* (learn.microsoft.com/power-platform/alm/use-source-control-solution-files). **Version-control merging of Power Platform component files is a "small files, small conflicts" game — the smaller the decomposed unit, the fewer clashing edits.**

**Cross-reference.** The platform-level view of solutions, managed environments, and the governance wrapper is owned by [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) §9.6; Git integration's maker-portal mechanics are a platform feature whose deployment engineering this guide covers.

---

## 3. Build and Release Pipelines: Build Tools for Azure DevOps and GitHub Actions

### 3.1 The Four Task Categories and the Tool Installer

**Verified (✅).** Microsoft describes the **Power Platform Build Tools for Azure DevOps** as *"a collection of Power Platform-specific Azure DevOps build tasks that eliminate the need to manually download custom tooling and scripts to manage the application lifecycle of apps built on Microsoft Power Platform"*, categorised into four types: *"Helper, Quality check, Solution, Environment management"* (learn.microsoft.com/power-platform/alm/devops-build-tools). The documented common pipeline shapes are *"Initiate, Export from Dev, Build, and Release."* Two hard prerequisites are called out: the tools *"are supported only for a Microsoft Dataverse environment with a database"*, and they are *"now available for use in GCC and GCC High regions."*

**Verified (✅).** The version story is a governance point: *"version 2.0 of Power Platform Build Tools is Power Platform CLI based. Power Platform Build Tools version 1.0 is based on PowerShell. Power Platform Build Tools 2.0 is the version that Microsoft services and adds newer features to."* Microsoft states *"Move to Power Platform Build Tools version 2.0 as soon as possible"*, and — critically — *"You can't mix and match task versions for Power Platform Build Tools in your build pipeline"* (learn.microsoft.com/power-platform/alm/devops-build-tools). **A greenfield 2026 pipeline should be 2.0-only.**

**Verified (✅).** The **Power Platform Tool Installer** is the first task in any pipeline: *"Add this task once before any other Power Platform Build Tools tasks in build and release pipelines. This task installs a set of Power Platform–specific tools required by the agent to run the Microsoft Power Platform build tasks"* (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks). The task ID quoted from the docs is `microsoft-IsvExpTools.PowerPlatform-BuildTools.tool-installer.PowerPlatformToolInstaller@2`, and its `AddToolsToPath` input *"Adds the pac CLI to the `PATH` environment variable."*

### 3.2 The Azure DevOps Task Reference

**Verified (✅).** The following task names, descriptions, and categories are quoted from Microsoft's *Microsoft Power Platform Build Tools tasks* reference (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks). Nothing here is invented; where the task ID is quoted, it is quoted verbatim from the YAML snippets in that article.

| Category | Task (display name) | Documented description |
| --- | --- | --- |
| Helper | **Power Platform Tool Installer** | *"Installs a set of Power Platform–specific tools required by the agent."* |
| Helper | **Power Platform WhoAmI** | *"Verifies a Power Platform environment service connection by connecting and making a WhoAmI request… useful to include early in the pipeline, to verify connectivity before processing begins."* |
| Quality check | **Power Platform Checker** | *"runs a static analysis check on your solutions against a set of best-practice rules to identify any problematic patterns."* |
| Solution | **Power Platform Import Solution** | *"Imports a solution into a target environment."* |
| Solution | **Power Platform Add Solution Component** | *"Adds a solution component to an unmanaged solution."* |
| Solution | **Power Platform Apply Solution Upgrade** | *"Upgrades a solution that has been imported as a holding solution."* |
| Solution | **Power Platform Export Solution** | *"Exports a solution from a source environment."* |
| Solution | **Power Platform Unpack Solution** | *"Takes a compressed solution file and decomposes it into multiple XML files so that these files can be more easily read and managed by a source control system."* |
| Solution | **Power Platform Pack Solution** | *"Packs a solution represented in source control into a solution.zip file that can be imported into another environment."* |
| Solution | **Power Platform Delete Solution** | *"Deletes a solution in the target environment."* |
| Solution | **Power Platform Publish Customizations** | *"Publishes all customizations in an environment."* |
| Solution | **Power Platform Set Solution Version** | *"Updates the version of a solution."* |
| Solution | **Power Platform Set Connection Variables** | *"Sets `PowerPlatformSetConnectionVariables.BuildTools.*` variables to provide custom script tasks access to use the service connection as a single source of truth."* |
| Solution | **Power Platform Deploy Package** | *"Deploys a package to an environment. Deploying a package as opposed to a single solution file provides an option to deploy multiple solutions, data, and code into an environment."* |
| Environment management | **Power Platform Create Environment** | *"Creates a new environment. Creating a new environment also automatically creates `BuildTools.EnvironmentUrl`."* |

**Verified (✅) — import semantics.** The Import Solution task's parameters encode the whole lifecycle vocabulary: `SolutionInputFile` is *"(Required) The path and file name of the solution .zip file to import"*; `HoldingSolution` is *"used when a solution needs to be upgraded. This parameter hosts the solution in Dataverse but does not upgrade the solution until the Apply Solution Upgrade task is run"*; `OverwriteUnmanagedCustomizations` *"Specify whether to overwrite un-managed customizations (true|false)"*; `PublishWorkflows` *"Specify whether any processes (workflows) in the solution should be activated after import"*; `AsyncOperation` runs the import *"asynchronously. This is recommended for larger solutions as this task will automatically timeout after 4 minutes otherwise"*; and `UseDeploymentSettingsFile` / `DeploymentSettingsFile` wire in the deployment settings JSON (§6) (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks).

**Verified (✅) — export semantics.** The Export Solution task exposes `Managed: true` to *"export the solution as a managed solution"* and a family of `Export*` toggles (`ExportAutoNumberingSettings`, `ExportCalendarSettings`, `ExportCustomizationSettings`, `ExportEmailTrackingSettings`, `ExportGeneralSettings`, `ExportIsvConfig`, `ExportMarketingSettings`, `ExportOutlookSynchronizationSettings`, `ExportRelationshipRoles`, `ExportSales`). For a managed-solution build, the documented shape is: export unmanaged → unpack (recommended as *"Unmanaged"*) → commit → pack as `Managed` → import (§3.4) (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks).

**Verified (✅) — the Pack/Unpack contract.** Unpack *"Takes a compressed solution file and decomposes it into multiple XML files"*, with `SolutionType` options **Unmanaged** (recommended), **Managed**, and **Both**; Pack runs the inverse, with `SolutionType` **Managed** (recommended), **Unmanaged**, and **Both** (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks). Note the asymmetry in the recommendations: **unpack unmanaged, pack managed** — the artifact that lands downstream is the managed one.

**Verified (✅) — versioning.** Set Solution Version *"Updates the version of a solution"* and takes `SolutionVersionNumber`. The docs recommend driving it from a pipeline variable such as `BuildId` — *"This provides options to define the exact shape of version number under the 'Options' tab"* (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks). **The solution version is therefore a build-server property, not a maker property** — an auditor's first question ("which build is in production?") has a machine answer.

**Verified (✅) — connection variables.** The Set Connection Variables task sets `PowerPlatformSetConnectionVariables.BuildTools.TenantId`, `...ApplicationId`, `...ClientSecret`, and `...DataverseConnectionString` (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks). This is what lets a custom script step reuse the same service connection the build tasks use, rather than re-authenticating.

### 3.3 GitHub Actions for Microsoft Power Platform

**Verified (✅).** Microsoft documents **GitHub Actions for Microsoft Power Platform** as a *"collection of Microsoft Power Platform–specific GitHub Actions that eliminate the need to manually download custom tooling and scripts to manage the application lifecycle"*, *"categorized into four types: Helper, Quality check, Solution, Environment management"* — the same taxonomy as the Azure DevOps tools — and notes they *"can run on both Windows agents and Linux agents"* (learn.microsoft.com/power-platform/alm/devops-github-actions). They reach the marketplace as **`powerplatform-actions`**, and Microsoft states the CLIs are shared: the Build Tools FAQ says the CLI-based tasks *"are the same tasks that are available with Power Platform Actions for GitHub"* (learn.microsoft.com/power-platform/alm/devops-build-tools).

**Verified (✅) — the actual action names.** The following are the action directory names published in the `microsoft/powerplatform-actions` repository (github.com/microsoft/powerplatform-actions, retrieved 2026-09-10), cross-checked against the Microsoft Learn *Available GitHub Actions* article (learn.microsoft.com/power-platform/alm/devops-github-available-actions):

| Category | Action | Microsoft Learn description / notes |
| --- | --- | --- |
| Helper | **`actions-install`** | Install Power Platform tools on the runner; *"must add the Install Power Platform Tools task as a first task in your workflow prior to any other GitHub Actions for Power Platform."* |
| Helper | **`who-am-i`** | *"Verifies the service connection by connecting to the service and sending a WhoAmI… request."* |
| Solution | **`import-solution`** | *"Imports a solution into a target environment."* |
| Solution | **`export-solution`** | *"Exports a solution from a source environment."* |
| Solution | **`unpack-solution`** | *"Takes a compressed solution file and decomposes it into multiple XML files so these files can be more easily read and managed by a source control system."* |
| Solution | **`pack-solution`** | *"Packs a solution represented in source control into a solution.zip file that can be imported into another environment."* |
| Solution | **`publish-solution`** | *"Publishes the solution customizations."* |
| Solution | **`clone-solution`** | *"Clones the solution for a given environment."* |
| Solution | **`upgrade-solution`** | Apply a solution upgrade (action present in the repository listing). |
| Solution | **`update-solution-version`** | Update the solution version (action present in the repository listing). |
| Solution | **`set-online-solution-version`** | Get or set the version of a solution loaded in Dataverse (action present in the repository listing). |
| Quality check | **`check-solution`** | Runs the Power Apps checker against the solution. |
| Solution | **`deploy-package`** | Deploy a package (multiple solutions, data, and code) into an environment. |
| Solution | **`delete-solution`** | Delete a solution (action present in the repository listing). |
| Solution | **`add-solution-component`** | Add a solution component (action present in the repository listing). |
| Solution | **`branch-solution`** | Branch a solution into a source-controlled folder (action present in the repository listing). |
| Solution | **`build`** | Build a solution project (action present in the repository listing). |
| Solution | **`install-application`** | Install a Dataverse application (action present in the repository listing). |
| Environment | **`create-environment`**, **`delete-environment`**, **`reset-environment`**, **`copy-environment`**, **`backup-environment`**, **`restore-environment`**, **`assign-user`**, **`assign-group`**, **`install-ms-cli`**, **`ms-app-pack`**, **`ms-app-deploy`**, **`upload-paportal`**, **`download-paportal`**, **`export-data`**, **`import-data`** | Environment-lifecycle and Power Pages actions present in the repository listing. |

**⚠ Flag — the action names in some briefs do not exist.** This pass could **not** find GitHub Actions named `upload-pipeline-artifact`, `download-pipeline-artifact`, or `set-connection-variables` in the `microsoft/powerplatform-actions` listing. The verified action names in the repository are those in the table above. Anyone who is told to use `upload-download-pipeline-artifact` should check the marketplace listing before writing it into a workflow: the repository exposes `upload-paportal`/`download-paportal` (Power Pages) and `export-solution`/`import-solution` for solutions, not a combined pipeline-artifact action. **Do not invent the missing names into a workflow.** Also note the GitHub action is `publish-solution`, whereas the Azure DevOps task is **Power Platform Publish Customizations** — the two platforms do not use the same display names for the same capability.

**⚠ Flag — maintenance posture.** The `microsoft/powerplatform-actions` repository is a MIT-licensed Microsoft repository (github.com/microsoft/powerplatform-actions, retrieved 2026-09-10) that *"wraps the existing Power Platform CLI"*. It is actively published (392 commits; releases linked from the repo), and the release notes are surfaced at `github.com/microsoft/powerplatform-actions/releases` (linked from learn.microsoft.com/power-platform/alm/devops-github-actions). However, the Learn article's own guidance is that the *product* ALM direction is **pipelines in Power Platform extended into CI/CD tools**, and the platform guide's §9.6 records the ALM Accelerator deprecation. **Treat the GitHub Actions as a maintained but not strategically-invested surface**: functional, documented, but not where Microsoft is putting new ALM features. Verify the release cadence before standardising on it.

**Verified (✅).** The cost statement is explicit and useful for a business case: *"GitHub Actions are available at no cost. However, you need a valid GitHub subscription to use the actions on GitHub"*, with *"2,000 action minutes per month for free"* (learn.microsoft.com/power-platform/alm/devops-github-actions). The Build Tools cost statement is the Azure DevOps mirror: *"The build tools are available at no cost. However, you need a valid subscription to Azure DevOps to use the Build Tools"* (learn.microsoft.com/power-platform/alm/devops-build-tools).

### 3.4 Which Tool, When

**⚠-knowledge (this synthesis, not a Microsoft claim).** The deployment-engineering decision that actually matters is not "Azure DevOps or GitHub" but "hosted pipelines or external pipeline". Microsoft's own position is documented in the pipelines FAQ: *"We encourage customers to use pipelines for core deployment functionality, and when needed, extend pipelines to integrate with other CI/CD tools. When used together, the workloads required within CI/CD tools often become less complicated and costly to maintain"* (learn.microsoft.com/power-platform/alm/pipelines). The practical rule this guide draws:

- **Native pipelines** for the maker-initiated dev→test→prod promotion with approvals (§5) — the cheapest, most governance-friendly path.
- **Azure DevOps Build Tools or GitHub Actions** for what the native pipeline will not do: source-controlled pre-deployment builds, code-first compiled components, solution checker as a hard gate, environment provisioning, and the "build once, deploy many" artifact flow (§3.2, §6).
- **Both**, joined by `pac pipeline` and the pipelines extensibility model (learn.microsoft.com/power-platform/alm/extend-pipelines), when a bank wants a governed native front door and an auditable external build — the configuration §13 recommends.

## 4. The pac CLI: The Command Surface

### 4.1 The Command Groups

**Verified (✅).** The **Microsoft Power Platform CLI** ("`pac`") is documented as a developer/administrator command-line surface whose command groups are enumerated at learn.microsoft.com/power-platform/developer/cli/reference. The full command-group table, quoted from that page, is:

| Command group | Documented description |
| --- | --- |
| **`pac admin`** | *"Work with your Power Platform Admin Account."* |
| **`pac application`** | *"Commands for listing and installing available Dataverse applications from Microsoft Marketplace."* |
| **`pac auth`** | *"Manage how you authenticate to various services."* |
| **`pac canvas`** | *"Operating with Power Apps .msapp files."* |
| **`pac catalog`** | *"Commands for working with Catalog in Power Platform."* |
| **`pac code`** | *"(Preview) Commands to manage your Code apps."* |
| **`pac connection`** | *"Commands for working with Dataverse connection."* |
| **`pac connector`** | *"Commands for working with Power Platform Connectors."* |
| **`pac copilot`** | *"Tools and utilities for copilot management."* |
| **`pac data`** | *"Import and export data from Dataverse."* |
| **`pac env`** | *"Work with your Dataverse organization."* |
| **`pac help`** | *"Show help for the Microsoft Power Platform CLI."* |
| **`pac managed-identity`** | *"Commands for managing Managed Identity records for Dataverse components."* |
| **`pac model`** | *"Commands for working with model-driven apps."* |
| **`pac modelbuilder`** | *"Code Generator for Dataverse APIs and Tables."* |
| **`pac package`** | *"Commands for working with Dataverse package projects."* |
| **`pac pages`** | *"Commands for working with Power Pages website."* |
| **`pac pcf`** | *"Commands for working with Power Apps component framework projects."* |
| **`pac pipeline`** | *"Work with Pipelines."* |
| **`pac plugin`** | *"Commands for working with Dataverse plug-in class library."* |
| **`pac power-fx`** | *"(Preview) Commands for working with Power Fx."* |
| **`pac solution`** | *"Commands for working with Dataverse solution projects."* |
| **`pac telemetry`** | *"Manage telemetry settings."* |
| **`pac test`** | *"(Deprecated) Execution of automated tests for a Power App."* |
| **`pac tool`** | *"Power Platform tools that can be installed and launched."* |

**⚠ Flag — `pac paportal` is no longer a command group.** Some briefs and older tutorials refer to `pac paportal` for Power Pages deployments. The current command-group index (learn.microsoft.com/power-platform/developer/cli/reference) lists **`pac pages`** — *"Commands for working with Power Pages website"* — and does not list `pac paportal`. Treat `pac paportal` as superseded and verify against the live index before scripting it.

**✅ Flag — `pac test` is deprecated.** The command-group table itself marks `pac test` as *"(Deprecated) Execution of automated tests for a Power App."* This aligns with the Test Engine deprecation (§9): the CLI's automated-test entry point and the Test Engine it drove are being retired.

### 4.2 pac solution in Detail

**Verified (✅).** `pac solution` is *"Commands for working with Dataverse solution projects."* Its subcommands and descriptions, quoted from learn.microsoft.com/power-platform/developer/cli/reference/solution, are:

| Subcommand | Documented description |
| --- | --- |
| `pac solution init` | *"Initializes a directory with a new Dataverse solution project."* |
| `pac solution clone` | *"Create a solution project based on an existing solution in your organization."* |
| `pac solution export` | *"Export a solution from Dataverse."* |
| `pac solution import` | *"Import the solution into Dataverse."* |
| `pac solution pack` | *"Package solution components on local filesystem into solution.zip (SolutionPackager)."* |
| `pac solution unpack` | *"Extract solution components from solution.zip onto local filesystem (SolutionPackager)."* |
| `pac solution upgrade` | *"Apply solution upgrade."* |
| `pac solution list` | *"List all Solutions from the current Dataverse organization."* |
| `pac solution create-settings` | *"Create a settings file from solution zip or solution folder."* |
| `pac solution check` | *"Upload a Dataverse solution project to run against the Power Apps Checker service."* |
| `pac solution add-reference` | *"Adds a reference from the project in the current directory to the project at 'path'"* |
| `pac solution add-solution-component` | *"Add one or more solution components to the target unmanaged solution in Dataverse."* |
| `pac solution delete` | *"Delete a solution from Dataverse in the current environment."* |
| `pac solution online-version` | *"Gets or sets version for solution loaded in Dataverse."* |
| `pac solution version` | *"Update build or revision version for the solution."* |
| `pac solution publish` | *"Publishes all customizations."* |
| `pac solution sync` | *"Sync the current Dataverse solution project to the current state of the solution in your organization."* |
| `pac solution add-license` | *"Add license and plan info to the solution."* |

**Verified (✅).** The `clone`/`export` distinction is worth internalising because it determines whether a repository is buildable. `pac solution clone` *"renders the original solution into a buildable format… you'll see a `cdsproj` file"* — the developer *"doesn't have to unpack the cloned solution because it's rendered in an unpacked format within the src (source) folder."* Conversely, `pac solution export` *"feel[s] like exporting the solution using the maker portal, and the resulting output is a solution zip file… The only difference is that you can't add references to this unpacked solution, as it doesn't have the `.cdsproj` project file"* (learn.microsoft.com/power-platform/developer/cli/reference/solution). **If the repository must build code-first components (`dotnet build` / `msbuild` on the `.cdsproj`), use `clone`; if it is a pure low-code artifact pipeline, `export`/`unpack`/`pack` is enough.**

**Verified (✅).** `pac solution create-settings` is the generator for the deployment settings file: *"Create a settings file from solution zip or solution folder"*, invoked as `pac solution create-settings --solution-zip <solution_zip_file_path> --settings-file <settings_file_name>` (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools). It is the mechanical link between §4 and §6.

### 4.3 pac pipeline, pac admin, and the Rest

**Verified (✅) — `pac pipeline`.** The command group is *"Work with Pipelines"*, with exactly two verbs (learn.microsoft.com/power-platform/developer/cli/reference/pipeline):

| Subcommand | Documented description | Key parameters |
| --- | --- | --- |
| `pac pipeline deploy` | *"Start pipeline deployment."* | Required: `--currentVersion`, `--newVersion`, `--solutionName`, `--stageId`; optional `--environment`, `--wait` |
| `pac pipeline list` | *"List Pipelines"* | optional `--environment`, `--pipeline` |

**Verified (✅).** The docs frame the developer experience as *"List pipelines to view pertinent details such as which stages and environments are ready to deploy their solutions to"* and *"Deploy a solution with a single command… the system orchestrates all the end-to-end deployment operations in compliance with the organizational policies. No need to connect to multiple environments, export solutions, download solution files, manually create connections and populate deployment settings files, import solutions, or handle various other tasks that were required previously"* (learn.microsoft.com/power-platform/alm/pipelines). **`pac pipeline deploy` is the bridge that lets an external CI/CD agent trigger the native governed pipeline.**

**Verified (✅) — `pac admin`.** The admin command group covers tenant and environment administration. Selected entries verbatim (learn.microsoft.com/power-platform/developer/cli/reference/admin): `pac admin list` ("List all environments from your tenant"), `pac admin create` ("Creates a Dataverse instance in your tenant"), `pac admin delete`, `pac admin reset`, `pac admin copy`, `pac admin backup` ("Takes a manual backup of your environment"), `pac admin restore` ("Restores an environment to a given backup"), `pac admin list-backups`, `pac admin set-backup-retention-period` ("Sets the backup retention period in days as provided. Valid values are: 7, 14, 21, 28"), `pac admin assign-user` ("Assign a user to a target Dataverse environment with specified security role"), `pac admin assign-group`, `pac admin add-group`, `pac admin list-roles`, `pac admin create-service-principal` ("Add Microsoft Entra ID application and associated application user to the Dataverse environment"), `pac admin list-service-principal`, `pac admin self-elevate`, `pac admin set-governance-config` ("Enable, disable, and edit managed environments"), `pac admin set-runtime-state`, `pac admin status` ("lists the status of all the operations in progress"), and `pac admin dlp-policy list` / `pac admin dlp-policy show`.

**Verified (✅) — `pac auth`.** *"Manage how you authenticate to various services"*, with `pac auth create`, `list`, `select`, `name`, `update`, `delete`, `clear`, and `who` (learn.microsoft.com/power-platform/developer/cli/reference/auth). The docs explain the multi-profile model: *"Running `pac auth create` creates an authentication profile on your machine. You can have multiple authentication profiles available… when your company uses multiple tenants (for development, production, and test) or when you're a consultant."* For headless environments, *"adding the `--deviceCode` parameter makes sure you can still connect"*, and *"in GitHub Codespaces, the `--deviceCode` parameter automatically gets added."* For service-principal auth: `pac auth create --name MyOrg-SPN --applicationId <id> --clientSecret $clientSecret --tenant <tenant>`; for federation there is a `--azureDevOpsFederated` switch ("Use Azure DevOps Federation for Service Principal Auth"); and there is an `--managedIdentity` path using the Default Azure Credential.

**Verified (✅) — `pac application`.** *"Commands for listing and installing available Dataverse applications from Microsoft Marketplace"*, with `pac application install` (*"Installs or updates Dataverse application to target environment"*) and `pac application list` (*"List available Dataverse applications from Microsoft Marketplace"*). `install` accepts `--application-name` or `--application-list <json file>` and `--environment` (learn.microsoft.com/power-platform/developer/cli/reference/application). **This is how a pipeline pins a baseline set of first-party apps to every downstream environment.**

**Verified (✅) — `pac canvas`.** *"Operating with Power Apps .msapp files"*, with `pac canvas create` (*"Generate a canvas app from a custom connector"*), `pac canvas download`, `pac canvas list`, `pac canvas pack` (*"(Preview) Pack sources into an msapp file"*), and `pac canvas unpack` (*"(Preview) Extract an msapp file into sources"*). **⚠ Flag:** *"The `pack` and `unpack` commands are deprecated. To source control your canvas app, use the Power Platform Git Integration"* (learn.microsoft.com/power-platform/developer/cli/reference/canvas). `pac canvas create` is generally available; `pack`/`unpack` are preview and deprecated.

**Verified (✅) — `pac plugin`.** *"Commands for working with Dataverse plug-in class library"*, with `pac plugin init` (*"Initializes a directory with a new Dataverse plug-in class library"*) and `pac plugin push` (*"Import plug-in into Dataverse"*) (learn.microsoft.com/power-platform/developer/cli/reference/plugin). This is the code-first developer's on-ramp; in a bank, these run in the build pipeline, never interactively against production.

**Verified (✅) — `pac data`.** *"Import and export data from Dataverse."* ⚠ **Two limitations matter for a pipeline:** *"These commands are intend[ed] to handle configuration data. They are not suitable for large volumes of data"* and *"These commands are only available for the .NET Full Framework version of the PAC CLI"* (learn.microsoft.com/power-platform/developer/cli/reference/data). Subcommands include `pac data export`, `pac data import`, the `pac data bulk-delete` family (`cancel`, `list`, `pause`, `resume`, `schedule`, `show`), and the `pac data retention` family (`enable-entity`, `list`, `set`, `show`, `status`). **The "configuration data only, .NET Full Framework only" caveat means `pac data` cannot be the answer to a bulk data migration — that is a data-migration project, cross-referenced to [data/dataops_guide.md](data/dataops_guide.md).**

---

## 5. Power Platform Pipelines: The Native Hosted Path

**Cross-reference and scope note.** Power Platform Pipelines — the native, in-product hosted deployment feature — is a platform capability owned by [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) §9.6 and §9.7 (solutions, managed environments, and solutions/pipelines licensing). This section does **not** re-derive the feature. It captures the deployment-engineering properties a CI/CD programme needs to design around, condensed.

### 5.1 The Capability and the Host

**Verified (✅).** Microsoft states the intent plainly: *"Pipelines in Power Platform aim to democratize application lifecycle management (ALM) for Power Platform and Dynamics 365 customers by bringing ALM automation and continuous integration and continuous delivery (CI/CD) capabilities into the service in a manner that's more approachable for all makers, admins, and developers"* (learn.microsoft.com/power-platform/alm/pipelines). The value claims are that *"Admins easily configure automated deployment pipelines in minutes rather than days or weeks"*, *"Makers have an intuitive user experience for easily deploying their solutions"*, and *"Professional developers can (optionally) extend pipelines and run them using the Power Platform command line interface (CLI)."*

**Verified (✅).** The **pipelines host** is a distinct environment that runs the feature. From the licensing FAQ: *"The pipelines host should be a production environment, but the pipelines host doesn't have to be a managed environment"* (learn.microsoft.com/power-platform/alm/pipelines). The host is where deployment requests are staged, where the exported artifacts are stored, and where the pipelines app and its flows live.

**Verified (✅).** Hosts are configured centrally: *"Tenant admins can automatically convert pipeline target environments to managed environments… go to the Power Platform admin center **Deployments** > **Settings**. Turn on the automatic managed environment setting for each pipeline host"* (learn.microsoft.com/power-platform/alm/pipelines).

### 5.2 Stages, Delegated Deployment, and Approvals

**Verified (✅).** A pipeline is composed of **stages**, each linked to an environment typed as *"Development Environment"* or *"Target Environment"*. Microsoft notes that only source environments show the pipeline: *"You'll only be able to view your pipeline in the assigned source environments, such as your development environments"* (learn.microsoft.com/power-platform/alm/pipelines). The common setup table from the docs:

| Environment purpose | Environment type | Standalone license required |
| --- | --- | --- |
| Host | Production | No |
| Development | Developer | No |
| QA | Developer | No |
| Production | Production | Yes |

**Verified (✅) — delegated deployments with approvals.** Microsoft documents *"Secure production environments with approval based delegated deployments"* (learn.microsoft.com/power-platform/alm/pipelines). The setup article states: *"You can run delegated deployments as a service principal or pipeline stage owner. When you enable this feature, the pipeline stage deploys as the delegate (service principal or pipeline stage owner) instead of the requesting maker"* (learn.microsoft.com/power-platform/alm/delegated-deployments-setup). **⚠ Flag — the admin steps are dense here; this is where most bank rollouts stall.** The service-principal path requires: (1) create an enterprise application (service principal) in Microsoft Entra ID, and *"Anyone enabling or modifying service principal configurations in pipelines must be an owner of the enterprise application"*; (2) *"Add the enterprise application as a server-to-server (S2S) user in your pipelines host environment and each target environment it deploys to"*; (3) *"Assign the Deployment Pipeline Administrator security role to the S2S user within the pipelines host, and System Administrator security role within target environments. Lower permission security roles can't deploy plug-ins and other code components"*; (4) check **Is delegated deployment** on a stage, select **Service Principal**, enter the Client ID. The approval flow itself is a cloud flow with the **`OnApprovalStarted`** trigger that calls the Dataverse unbound action **`UpdateApprovalStatus`** *"using the service principal's connection"*, with *"ApprovalStatus: 20 = approved, 30 = rejected"*. A common error is called out by name: *"The deployment stage isn't an owner of the service principal (<AppId>). Only owners of the service principal may use it for delegated deployments."* — and the docs note *"You might be the owner of the App Registration, but not the Enterprise Application."*

**Verified (✅).** Sharing-on-deploy is a native capability of delegated deployments with service principals: *"admins only need to approve the deployment request, and the system automatically handles sharing."* Currently *"the system supports sharing security roles, canvas apps, and cloud flows"*, and *"Sharing is available the first time an object is deployed to the target environment. You can't update sharing when new versions are deployed"* (learn.microsoft.com/power-platform/alm/delegated-deployments-setup). **Design implication: get the security group right on first deploy; it is not editable on later versions.**

### 5.3 Sequential-and-Tamper-Proof Artifacts and Prevalidation

**Verified (✅) — the artifact is frozen at submit.** This is the property that makes native pipelines auditable: *"Solutions are exported as soon as a deployment request is submitted (when the maker selects **Deploy** from within their development environment), and the same solution artifact will be deployed. Similarly, the system doesn't re-export a solution for deployments to subsequent stages in a pipeline. The same solution artifact must pass through pipeline stages in sequential order. The system also prevents any tampering or modification to the exported solution artifact. This ensures customization can't bypass QA environments or your approval processes"* (learn.microsoft.com/power-platform/alm/pipelines). **One artifact, one hash, one path — the property a change-management auditor wants.**

**Verified (✅) — prevalidation.** *"Solution deployments are prevalidated against the target environment to prevent mistakes and improve success rates. For example, missing dependencies and other issues are detected before deployment and makers are immediately guided to take the appropriate action"*, and *"Connections and environment variables are provided upfront and validated before the deployment begins"* (learn.microsoft.com/power-platform/alm/pipelines).

**Verified (✅) — the automatic backup.** *"Yes. Both managed and unmanaged solutions are automatically exported and stored in the pipelines host for every deployment"* (learn.microsoft.com/power-platform/alm/pipelines). **Every deployment therefore leaves both a managed artifact (what was deployed) and an unmanaged artifact (the recoverable source form) on the host — see §11 on rollback.**

**Verified (✅) — documented limitations.** Native pipelines will *not*: deploy *"Power BI Dashboards (preview) and Power BI Datasets (preview)"*; deploy unmanaged solutions (*"No. We recommend that you always deploy managed solutions to nondevelopment environments"*); deploy multiple solutions at once (*"Not currently… You'll need to submit a different deployment for each solution"*); publish unmanaged customizations before export (*"Not currently"*); or support multi-developer teams working in isolated development environments (*"The current implementation uses a single development environment for a given solution"*) (learn.microsoft.com/power-platform/alm/pipelines). **These are the boundaries that force the external CI/CD tooling of §3 to exist.**

### 5.4 Which Environments Must Be Managed

**Verified (✅).** The rules are stated as a short list: *"Developer environments aren't required to be managed environments… The pipelines host should be a production environment, but the pipelines host doesn't have to be a managed environment. All other environments used in pipelines must be enabled as managed environments. Licenses granting premium use rights are required for all managed environments"* (learn.microsoft.com/power-platform/alm/pipelines).

**Verified (✅) — the February 2026 auto-enable.** *"Starting February 2026, Microsoft will start enabling managed environments for any pipeline target environments that aren't already enabled. Customers will be notified via Microsoft 365 Message center. We recommend you review and enable managed environments for all pipeline targets now"* (learn.microsoft.com/power-platform/alm/pipelines). Admins can opt to have it happen automatically by configuring the automatic managed-environment setting per pipeline host. **⚠ Flag — this is a live 2026 change with a licensing consequence** (managed environments require premium-use licenses), so it must be in the bank's licence and cost model, not just its pipeline design.

**Cross-reference.** The platform guide §9.3 (managed environments and environment groups) and §9.7 (licensing) own the anatomy and the cost model; §9.6 owns the pipelines-vs-ALM-Accelerator comparison. This guide assumes them.

---

## 6. Deployment Settings: Environment Variables and Connection References

### 6.1 Why They Exist: The Build-once-Deploy-many Problem

**Verified (✅).** Microsoft states the problem and the mechanism: environment variables *"enable the basic application lifecycle management (ALM) scenario of moving an application between Power Platform environments. In this scenario, the application stays exactly the same except for a few key external application references (such as tables, connections, and keys) that are different between the source environment and the destination environment… Environment variables allow you to specify which of these different external references should be updated as the application is moved across environments"* (learn.microsoft.com/power-apps/maker/data-platform/environmentvariables). The alternative — *"leaving hard-coded parameter values within the components that use them"* — *"is often problematic; especially when the values need to be changed during ALM operations."*

**Verified (✅).** The documented benefits that make this the build-once-deploy-many mechanism: *"Provide new parameter values while importing solutions to other environments"*; *"Store configuration for the data sources used in canvas apps and flows"*; *"Package and transport your customization and configuration together"*; *"Package and transport secrets… separately from the components that use them"*; *"One environment variable can be used across many different solution components"*; and *"The environment variables can be unpacked and stored in source control. You might also store different environment variables values files for the separate configuration needed in different environments"* (learn.microsoft.com/power-apps/maker/data-platform/environmentvariables).

**Verified (✅).** Environment variables are a **solution component** with a **definition** and, optionally, a **value**: the data types are *"Decimal number, Text, JSON, Two options, Data source, or Secret"*; the *"Current Value… is a part of the environment variable value table. A defined value is used even if a default value is also present"*; and the *"Default Value… is part of the environment variable definition table… The default value is used if there's no current value"* (learn.microsoft.com/power-apps/maker/data-platform/environmentvariables). **The separation is what lets you remove a production value from a solution before export — "Remove from this solution" under Current Value — so the pipeline supplies the per-environment value instead.**

**Verified (✅).** **Connection references** are the companion mechanism for connections: the deployment settings file's `ConnectionReferences` section pairs a `LogicalName` with a `ConnectionId` and a `ConnectorId`, and *"During solution import, the connection references are validated to ensure that the connections placed inside them will be usable by the owner of the connection reference. Ensure that the connections are owned by the owner of the connection references or that the connections are shared"* (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools).

### 6.2 The Deployment Settings JSON

**Verified (✅).** Microsoft's description of the file: *"To pre-populate the connection reference and environment variable information for your deployment, use the deployment settings file (JSON) to store the information, and pass it as a parameter when importing the solution using Power Platform Build Tools. You can store the JSON file in your source control system to update and manage as required for your organization"* (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools). The documented shape has two top-level arrays — `EnvironmentVariables` (each with `SchemaName` and `Value`) and `ConnectionReferences` (each with `LogicalName`, `ConnectionId`, and `ConnectorId`):

```json
{
  "EnvironmentVariables": [
    { "SchemaName": "tst_Deployment_env", "Value": "Test" },
    { "SchemaName": "tst_EnvironmentType", "Value": "UAT" }
  ],
  "ConnectionReferences": [
    {
      "LogicalName": "tst_SharepointSiteURL",
      "ConnectionId": "ef3d1cbb2c3b4e7987e02486584689d3",
      "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline"
    }
  ]
}
```

**Verified (✅).** The rationale is stated as the automated-deployment enabler: *"After importing a solution containing connection reference and environment variable information, you are prompted to provide information specific to your environment in the UI. However, entering this information does not work well for fully automated Continuous Integration/Continuous Delivery (CI/CD) scenarios. To enable a fully automated deployment, you can now pre-populate the connection reference and environment variable information specific to the target environment so that you don't have to interactively provide it after importing a solution"* (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools).

### 6.3 Generating and Consuming the File

**Verified (✅).** There are two documented ways to generate the file (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools):

1. **`pac solution create-settings`** — *"Use the `create-settings` property with Power Platform CLI"*: `pac solution create-settings --solution-zip <solution_zip_file_path> --settings-file <settings_file_name>`. The command emits a JSON file whose `ConnectionReferences` values are initially empty and must be gathered after the connections exist in the target environment.
2. **By cloning the solution** — after `pac solution clone`, *"Proceed to create the settings file in the context of the current folder and populate the value of the settings file."*

**Verified (✅).** The per-environment population steps are documented: get the connection ID *"Sign in to Power Apps and select your target environment. In the left navigation pane, select Data > Connections, select the connection… and then look at the URL to get the connection ID"*; get the environment-variable values from the maker portal; then *"Manually edit the deployment settings file (JSON) to add the connection and environment variable information appropriately."*

**Verified (✅).** The consuming step is the import task: *"When using the Import Solution task in Power Platform Build Tools, select **Use deployment settings file** and provide the path to the deployment settings file. Next, initiate the pipeline"* (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools). The import task parameters `UseDeploymentSettingsFile` / `DeploymentSettingsFile` (§3.2) are the concrete switch.

**Verified (✅) — the pipeline-of-record pattern.** Because the JSON is per-environment, the mature pattern is one settings file per target environment, checked into source control alongside the solution, with the environment-specific `Value`/`ConnectionId` fields populated from the target environment. **⚠-knowledge:** splitting the settings file per environment and parameterising it in the pipeline (rather than committing secrets) is established practice, not a Microsoft-stated mandate; the documented mechanism is simply the JSON file plus the import parameter.

---

## 7. Authentication: Service Principals, Application Users, and Secrets

### 7.1 The Three Connection Types

**Verified (✅).** Microsoft documents three connection types for Build Tools (learn.microsoft.com/power-platform/alm/devops-build-tools):

| Connection type | Documented description |
| --- | --- |
| **Service Principal via Workload Identity Federation (recommended)** | *"Uses service principal based authentication via federated credentials. Recommended for tenants which require multi-factor authentication."* |
| **Service principal and client secret** | *"Uses service principal based authentication."* |
| **Username/password** | *"A generic service connection with username and password. Does not support users requring multi-factor authentication."* |

**Verified (✅) — for GitHub.** The GitHub Actions article states two types — username/password (*"doesn't support multifactor authentication"*) and **service principal and client secret** (*"supports multifactor authentication"*) — and documents the secret plumbing: the client secret *"must be added and stored as a GitHub Secret"*, referenced as `${{secrets.CLIENT_SECRET_GITHUB_ACTIONS}}`, while the application ID and tenant ID can be set as workflow environment variables (`WF_APPLICATION_ID`, `WF_TENANT_ID`) (learn.microsoft.com/power-platform/alm/devops-github-available-actions). **⚠ Flag for a bank: username/password with an MFA-exempt service account is exactly the control an auditor will reject. The only defensible choices are service principal with a rotated secret, or workload identity federation.**

### 7.2 The Setup Path

**Verified (✅).** The documented setup path for a service principal is a five-step chain (learn.microsoft.com/power-platform/alm/devops-build-tools):

1. **Create the service principal / app registration.** Microsoft recommends the CLI: *"Use the `pac admin create-service-principal` command to create and configure the service principal to use with the Microsoft Power Platform Build Tools tasks. This command first registers an Application object and corresponding Service Principal Name (SPN) in Microsoft Entra ID. Then it adds the application as an administrator user to the Microsoft Power Platform tenant."*
2. **Capture the credentials.** *"When successful, the command displays four columns: Power Platform TenantId, Application ID, Client Secret (in clear text), Expiration."* With the warning: *"Keep the client secret safe and secure. Once the command prompt is cleared, you can't retrieve the same client secret again."*
3. **Add the Application ID as an Application User** in the target environment: *"You must add the Application ID as an Application User in the Microsoft Power Platform environment you're connecting to."*
4. **Assign the security role.** *"Ensure that the added Application User has the system administrator role assigned (available from 'Manage Roles' in the security settings for the application user)."*
5. **Configure the Azure DevOps service connection** using the Tenant ID, Application ID, and Client Secret from step 2.

**Verified (✅).** The `pac admin create-service-principal` parameters are *"`environment` — The ID or URL of the environment to add an application user to"* (required) and *"`role` — Name or ID of security role to apply to the application user. The default value is: 'System Administrator'"* (optional), with the note that *"Solution Checker requires a role with prvAppendmsdyn_analysisjob privilege to append to the msdyn_analysisjob table. The System Administrator role has this privilege by default"* (learn.microsoft.com/power-platform/alm/devops-build-tools).

**Verified (✅) — the application-user alternative and the admin-center path.** Microsoft offers two ways to create the application user: *"You can review this article: Application user creation. You can use Power Platform CLI"* — the CLI example being `pac admin assign-user --user "<app id>" --role "System administrator" --environment <env> --application-user` (learn.microsoft.com/power-platform/alm/devops-build-tools). The ALM Accelerator documentation independently confirms the **admin-center path** (*"Sign in to the Power Platform admin center… Select Users + permissions > Application users. Select + New app user"*), and its recommendation *"you give the app user system administrator security role privileges"* (learn.microsoft.com/power-platform/guidance/coe/almaccelerator-components).

**⚠ Flag — least privilege versus documented defaults.** Every documented path uses **System Administrator** for the pipeline identity or its app user, and the docs concede *"Lower permission security roles can't deploy plug-ins and other code components"* (learn.microsoft.com/power-platform/alm/delegated-deployments-setup). For a bank this is a genuine tension: the platform's documented happy path is a high-privilege machine identity. The defensible mitigations are the ones the platform itself offers — a dedicated application user per environment, no interactive login, secret rotation and expiry, and the **Block unmanaged customizations** managed-environment control that Microsoft recommends *"when assigning higher permissions"* (learn.microsoft.com/power-platform/alm/delegated-deployments-setup) — plus the masking of these credentials behind the pipeline's secret store.

### 7.3 pac admin create-service-principal and the Admin Steps

**Verified (✅).** Where the admin steps are dense, treat the `pac` CLI as the authoritative, reversible path: `pac admin create-service-principal` creates the app-plus-app-user in one command; `pac admin list-service-principal` lists *"Microsoft Entra ID applications which have access to Dataverse"*; `pac admin application register` / `unregister` register and unregister Entra ID applications with the tenant; and `pac admin assign-user` assigns the security role (learn.microsoft.com/power-platform/developer/cli/reference/admin). **⚠ Flag:** the portal walkthroughs (admin center → Users + permissions → Application users) and the CLI path are two routes to the same end state, and the docs do not keep the two perfectly in step; pick one, script it, and verify with `pac admin list-role`/`list-service-principal` after each environment build.

---

## 8. Quality Gates: The Solution Checker

### 8.1 What It Is and the Rule Model

**Verified (✅).** The **solution checker** is a static-analysis service: *"a powerful tool that performs a comprehensive static analysis of your solution objects against a set of best practice rules. By using solution checker, you can quickly identify problematic patterns in your components and receive detailed reports that highlight issues, affected components, and provide links to documentation on how to resolve each issue"* (learn.microsoft.com/power-apps/maker/data-platform/use-powerapps-checker). A critical scope limit is stated: *"Solution checker works with unmanaged solutions that can be exported from an environment."*

**Verified (✅).** The components analysed are named: *"Dataverse custom workflow activities; Dataverse web resources (HTML and JavaScript); Dataverse configurations, such as SDK message steps; Power Automate flows (via flow checker); Power Fx expressions (via app checker)"* (learn.microsoft.com/power-apps/maker/data-platform/use-powerapps-checker). A language-level limit is also stated: the checker *"supports global variables for ECMAScript 2015 (ES6) and up to ECMAScript 2018 (ES9) syntax. When JavaScript is detected using global variables later than ES6 or syntax later than ES9, a web-unsupported-syntax issue for the web resource is reported."*

**Verified (✅) — the rule and severity model.** The report columns are the model, quoted verbatim (learn.microsoft.com/power-apps/maker/data-platform/use-powerapps-checker):

| Report column | Documented meaning |
| --- | --- |
| **Issue** | *"The title of the issue identified in the solution."* |
| **Category** | *"The categorization of the issue identified, such as **Performance**, **Maintainability**, **Usage**, **Supportability**, **Design**, **Security**, **Accessibility**, or **Upgrade readiness**."* |
| **Severity** | *"Represents the potential impact of the issue identified. Available impact types are **Critical**, **High**, **Medium**, **Low**, and **Informational**."* |
| **Guidance** | *"Link to article detailing the issue, impact, and recommended action."* |
| **Component** / **Location** | The component and its file/assembly location. |
| **Line #**, **Module**, **Type**, **Member**, **Statement**, **Comments** | Component-type-specific detail columns. |

**Verified (✅).** The ruleset model is documented in the checker web API page: a solution check requires *"a list of rules… in the form of individual rules or a grouping of rules… referred to as a ruleset."* Two rulesets are documented — the **Solution Checker** ruleset, which *"contains a set of impactful rules that have limited chances for false positives"*, and the **Marketplace certification** ruleset used for Marketplace certification (learn.microsoft.com/power-platform/alm/checker-api/overview). The report output format is **SARIF**: *"the result of the solution analysis is a zip file containing one or more reports in a standardized JSON format… based on static analysis results referred to as Static Analysis Results Interchange Format (SARIF)… The service uses version two of the OASIS standard"* (learn.microsoft.com/power-platform/alm/checker-api/overview).

**Verified (✅) — the honest caveat that matters for gating.** Twice the docs warn: *"Use of solution checker doesn't guarantee that a solution import will be successful. The static analysis checks performed against the solution don't know the configured state of the destination environment and import success might be dependent on other solutions or configurations in the environment"* (learn.microsoft.com/power-apps/maker/data-platform/use-powerapps-checker; learn.microsoft.com/power-platform/alm/checker-api/overview). **The checker is a quality gate, not a deployability gate — it cannot replace the pre-deployment validation of §10.**

### 8.2 Running It: Portal, Pipeline, and CLI

**Verified (✅) — portal.** *"Sign in to Power Apps… next to the unmanaged solution that you want to analyze, select **...**, point to **Solution checker**, and then select **Run**"*; the results appear *"in the portal, or you can download the report from your web browser"*, and the report is *"in Excel format"* with drill-down by **Issue**, **Location**, or **Severity** (learn.microsoft.com/power-apps/maker/data-platform/use-powerapps-checker). The solution-check states documented are: *"Hasn't been run"*, *"Running"*, *"Couldn't be completed"*, *"Results as of date and time"*, *"Couldn't be completed. Result as of date and time"*, *"Checked by Microsoft"*, and *"Checked by Publisher"*.

**Verified (✅) — Azure DevOps pipeline.** The **Power Platform Checker** task *"runs a static analysis check on your solutions against a set of best-practice rules."* Its parameters are the gate controls (learn.microsoft.com/power-platform/alm/devops-build-tool-tasks):

| Parameter | Documented meaning |
| --- | --- |
| `RuleSet` | *(Required) "Specify which rule set to apply"* — **Solution checker** (same as the maker portal) or **Marketplace** (the extended certification ruleset). The docs give the Solution Checker ruleset GUID as `0ad12346-e108-40b8-a956-9a8f95ea18c9` in the YAML example. |
| `ErrorLevel` | *"Combined with the error threshold parameter, this value defines the severity of errors and warnings that are allowed. Supported threshold values are `<level>IssueCount` where level=Critical, High, Medium, Low, and Informational."* |
| `ErrorThreshold` | *"Defines the number of errors (>=0) of a specified level that are allowed for the checker to pass the solutions being checked."* |
| `FailOnPowerAppsCheckerAnalysisError` | *"When true, fail if the Power Apps Checker analysis returns as Failed or FinishedWithErrors."* |
| `RulesToOverride` | *"A JSON array containing rules and levels to override. Accepted values for `OverrideLevel` are: Critical, High, Medium, Low, Informational."* |
| `ArtifactDestinationName` | *"Specify the Azure Artifacts name for the checker .sarif file."* |
| `FilesToAnalyze` / `FilesToExclude` / `FilesToAnalyzeSasUri` | Scope the analysis; the docs note *"Reference an exported solution file and not the unpacked source files in your repository. You can analyze both managed and unmanaged solution files."* |

**Verified (✅).** `ErrorLevel` + `ErrorThreshold` are the concrete "solution checker threshold" a bank sets as a gate: for example, fail the build if more than a set number of Critical issues are found, while allowing Informational issues through. The SARIF artifact (`ArtifactDestinationName`) is the machine-readable evidence record. The Build Tools FAQ confirms the developer consumption path: *"The output of the Checker task is a SARIF file… Both VS Code and Visual Studio extensions are available for viewing and taking action on SARIF files"* (learn.microsoft.com/power-platform/alm/devops-build-tools).

**Verified (✅) — GitHub Actions and CLI.** The GitHub Actions collection includes a **`check-solution`** action (github.com/microsoft/powerplatform-actions), and the CLI exposes **`pac solution check`** — *"Upload a Dataverse solution project to run against the Power Apps Checker service"* — with options including `--path`, `--outputDirectory`, `--geo`, and `--clearCache` (*"Clears the solution checker enforcement cache, for your tenant, of all records that pertain to past results for your solutions"*) (learn.microsoft.com/power-platform/developer/cli/reference/solution). **All three routes hit the same service and the same ruleset, so the gate is consistent whether a maker runs it in the portal or an agent runs it in a pipeline.**

---

## 9. Automated Testing: Test Engine, Test Studio, Regression and UAT

### 9.1 The Power Apps Test Engine

**❌ / ⚠ — Test Engine is deprecated; do not build new estates on it.** Effective **April 2026**, *"Test Engine is deprecated. The documentation and GitHub repository are no longer maintained by Microsoft and will be removed in a future release"* (learn.microsoft.com/power-platform/important-changes-coming, §"Deprecation of Test Engine"). The stated reason: *"Test Engine has near-zero usage and failed to meet customer needs in an evolving AI landscape. The Power Fx implementation created unnecessary limitations that are avoided if using Playwright directly (Test Engine is built on Playwright)."*

**✅ Verified — the replacement direction.** Microsoft names the successor: *"The Power Platform Playwright samples guide the use of Playwright for Power Platform test automation. These practical examples resemble how Microsoft tests first-party applications internally, with best-practice guidance baked in"* (learn.microsoft.com/power-platform/important-changes-coming). ⚠ **Flag:** the Playwright-samples URL given in the deprecation notice (`.../test-engine/developer/playwright-samples/overview`) returned **404** on this pass (retrieved 2026-09-10). The *direction* is verified; the *landing page* could not be confirmed at the time of writing. **Treat "use Playwright directly for Power Platform UI test automation" as the verified guidance, and treat the specific sample URLs as volatile.**

**✅ Verified — what Test Engine was.** For historical and migration context, Test Engine was the previously documented mechanism described in the same Learn corpus as a YAML-driven, Playwright-based test harness for canvas apps, and the `pac test` command group is explicitly marked *"(Deprecated) Execution of automated tests for a Power App"* (learn.microsoft.com/power-platform/developer/cli/reference/). ⚠ **Flag — the YAML test-definition detail.** The brief for this guide asked for verification of *"the YAML-based test definitions, the supported app types, the Playwright/browser-driving model"* for Test Engine. Because Microsoft has deprecated the feature and removed/redirected its documentation, **the YAML test-plan schema and the supported-app-type matrix could not be verified at primary source this pass** and are logged in §14 rather than asserted. The honest position for a 2026 bank is: **do not design a test strategy around Test Engine.**

### 9.2 Maturity, Licensing, and Limits

**⚠ Flag — honest maturity assessment.** For a bank, the automated UI-testing story on Power Platform in 2026 is **immature and in transition**: the first-party productised harness (Test Engine) is retired, the CLI entry point (`pac test`) is deprecated, and the replacement is a **samples-based, Playwright-direct** approach rather than a supported product. This is a materially different posture from a mature DevOps platform. **Do not promise an auditor "automated end-to-end regression on every deployment" on the strength of a product feature; promise it only where the team has genuinely built and maintains a Playwright suite.** Licensing for such a suite is a matter of the browser-driving runners and the environment access, not a separate Power Platform SKU.

**Verified (✅) — Test Studio, the surviving low-code option.** **Power Apps Test Studio** remains documented and is the low-code test authoring surface: *"Power Apps Test Studio is a low-code solution to write, organize, and automate tests for canvas apps. In Test Studio, you can write tests using Power Apps expressions or use a recorder to save app interaction to automatically generate the expressions. You can play written tests back within the Test Studio to validate app functionality, and also run the tests in a web browser and build the automated tests into your app deployment process"* (learn.microsoft.com/power-apps/maker/canvas-apps/test-studio). Its terminology: **test cases** are *"made up of a series of instructions or actions, called test steps"* written *"using the Power Apps expression language"*; **test suites** *"are used to organize or group test cases together"* and *"Test cases contained in test suites are run sequentially. The app state is persisted across all test cases in a suite"*; and **test assertions** — *"An assertion is an expression that evaluates to true or false in the test. If the expression returns false, the test case will fail."*

**Verified (✅) — Test Studio prerequisites and limits.** *"You must be creator or co-owner of an app to test the app with Test Studio."* The documented **known limitations** are unusually candid and should be read before committing to Test Studio coverage (learn.microsoft.com/power-apps/maker/canvas-apps/test-studio): components; code components written in PCF; nested galleries; media controls; the formula-level error management experimental feature must be on; controls not listed in the `Select`/`SetProperty` functions; person-type columns; and — decisively for a CI/CD programme — *"Test Studio is not compatible with the experimental Git version control feature, and will not work properly if that feature is enabled."* **⚠-knowledge:** the ecosystem carries community reports that Test Studio itself is a fragile, maker-portal-bound authoring surface, and Microsoft's own documentation frames its automated-test path as moving to Playwright; a bank should treat Test Studio as a supplementary aid, not the regression backbone.

**Verified (✅) — what to automate (Microsoft's own best-practice framing).** *"It's difficult to automate all tests and we do not recommend that you completely rely on test automation. Manual testing should be performed in addition to test automation."* The documented best-fit automation targets: *"Repetitive tests; High business impact functionality tests; Features that are stable and not undergoing significant change; Features that require multiple data sets; Manual testing that takes significant time and effort"* (learn.microsoft.com/power-apps/maker/canvas-apps/test-studio).

### 9.3 Regression and UAT Practice

**⚠-knowledge — practice, not product.** The following is established industry practice and the deployment literature's view; Microsoft's docs state some of it (in the Test Studio intro: *"Testing is an important part of the software development life cycle (SDLC)… Regression testing of the app might be longer than the time spent to develop new features"* — learn.microsoft.com/power-apps/maker/canvas-apps/test-studio), but the *shape* of a regression suite and a UAT gate is a practice decision this guide marks ⚠-knowledge:

- **Layered testing.** Unit-level checks in code-first components (plug-ins, PCF) run in the build agent; solution-level static analysis (the checker) runs as a gate; app-level UI tests run against a deployed test environment; and human UAT runs last against a representative environment with representative data.
- **The regression suite is a curated subset, not everything.** Microsoft's own guidance (above) is that automating everything is discouraged; curate the high-business-impact, stable, data-heavy flows.
- **UAT is a business sign-off, not a test execution.** The point of the UAT environment is to let business owners validate behaviour and to produce a formal acceptance record — which is an audit artefact, not a pipeline stage (§12).
- **Data matters as much as code.** A UAT environment whose data does not resemble production will pass flows that production fails. Microsoft's own docs note solutions *"don't contain any business data"* (learn.microsoft.com/power-platform/alm/basics-alm), so a UAT data strategy is a separate workstream, cross-referenced to [data/dataops_guide.md](data/dataops_guide.md).

---

## 10. Validation and Pre-Deployment Checks

**Verified (✅).** The native pipeline's prevalidation is the first line: *"Solution deployments are prevalidated against the target environment to prevent mistakes and improve success rates. For example, missing dependencies and other issues are detected before deployment and makers are immediately guided to take the appropriate action"* (learn.microsoft.com/power-platform/alm/pipelines). The same article adds that *"Connections and environment variables are provided upfront and validated before the deployment begins"* — which is the deployment-settings mechanism of §6 acting as a validation input.

**Verified (✅).** **Missing-dependency detection** is also a platform-level service independent of pipelines. Microsoft documents dependency tracking for solution components (learn.microsoft.com/power-platform/alm/dependency-tracking-solution-components), and the solution-concepts article states the enforcement: *"The system tracks these dependencies between solutions. If you try to install a solution that requires a base solution that isn't installed, you won't be able to install the solution. You get a message saying that the solution requires another solution to be installed first"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). **This is a hard stop at import, which is good for correctness and bad for a surprise in a production window — hence the value of a pre-production dry run.**

**Verified (✅).** **Connection and environment-variable validation** happens during import: *"During solution import, the connection references are validated to ensure that the connections placed inside them will be usable by the owner of the connection reference. Ensure that the connections are owned by the owner of the connection references or that the connections are shared"* (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools). The practical pre-deployment check is that every `ConnectionReference` in the target's deployment settings JSON points at a connection owned by (or shared with) the deploying identity (§7.2).

**Verified (✅) — managed-environment rules as pre-deployment constraints.** *"All other environments used in pipelines must be enabled as managed environments"*, and from February 2026 Microsoft auto-enables pipeline targets (learn.microsoft.com/power-platform/alm/pipelines). A pre-deployment checklist should confirm each target's managed status (via the admin center or `pac admin set-governance-config`), because the managed-environment setting changes what the deployment can do — for example, the **Block unmanaged customizations** control Microsoft recommends when higher permissions are assigned (learn.microsoft.com/power-platform/alm/delegated-deployments-setup).

**Verified (✅).** The broader validation philosophy is stated by Microsoft's ALM overview: ALM's governance area includes *"data security, user access, change tracking, review, audit, deployment control, and rollback"* (learn.microsoft.com/power-platform/alm/overview-alm) — **validation is the "review" and "deployment control" half of that list, and it is a governance concern, not only an engineering one.**

**⚠-knowledge — the pre-deployment checklist.** No single Microsoft page publishes a "pre-deployment validation checklist" for Power Platform. The following is this guide's synthesis (⚠-knowledge), built only from verified mechanisms: (1) checker run clean against the threshold (§8.2); (2) all connection references resolved for the target (§6.2); (3) all environment variables have target values (§6.1); (4) solution dependencies installed in the right order (§1.3); (5) target is a managed environment (§10); (6) deploying identity has the required role (§7); (7) backup/restore point exists (§11); (8) rollback plan agreed (§11). A bank should encode these as pipeline stage gates, not run them by hand.

---

## 11. Governance and Operations: History, Audit, Rollback, and CoE Tooling

### 11.1 Connection References and Environment Variables in ALM

**Verified (✅).** Reiterating the ALM role because it is the governance payload of the deployment settings: connection references and environment variables *"enable you to interactively specify the connection details and configuration settings specific to the target environment where your app or solution is deployed"* (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools). Because the same solution artifact carries the *keys* while the per-environment JSON carries the *values*, **the artifact is provably identical across environments and only the configuration differs** — the mechanical meaning of build-once-deploy-many, and the reason a bank can assert that what was tested is what was deployed. Secrets can be carried separately: environment variables of type **Secret** point at Azure Key Vault (learn.microsoft.com/power-apps/maker/data-platform/environmentvariables).

### 11.2 DLP Constraints Affecting Pipelines (Condensed)

**Cross-reference.** Data loss prevention policies — connector classification (Business/Non-business/Blocked), the effect on apps and flows, and the policy model — are owned by [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) §9.4 and are not re-derived here.

**⚠-knowledge.** The CI/CD-relevant consequence, stated as practice rather than product text: a DLP policy that blocks a connector will prevent an app or flow that depends on that connector from functioning, **even if the solution imports successfully**. Because a green import is not proof of a working deployment (§2.2's partial-deploy failure mode), a pipeline's post-deploy checks should confirm the connectors the solution relies on are permitted in the target environment. The platform's DLP inventory tooling (`pac admin dlp-policy list` / `dlp-policy show`) is the machine-readable way to assert this in a gate.

### 11.3 Deployment History and Audit: The who-deployed-what-when Chain

**Verified (✅) — native pipeline run history.** Native pipelines provide run history, and the approval flow is linked into it: Microsoft's delegated-deployment guidance notes, for the approval flow, *"To improve the debug experience, select ApprovalProperties and insert workflow() from the dynamic content menu. This link[s] the flow run to the pipeline stage run (run history)"* (learn.microsoft.com/power-platform/alm/delegated-deployments-setup). The pipelines overview also states that with pipelines *"Customizations and audit log saved automatically and are easily accessible"* and *"Out-of-the-box analytics provides better visibility within a central location"* (learn.microsoft.com/power-platform/alm/pipelines).

**Verified (✅) — Dataverse auditing.** *"Dataverse auditing logs changes that you make to customer records in an environment with a Dataverse database. Dataverse auditing also logs user access through an app or through the SDK in an environment"* (learn.microsoft.com/power-platform/admin/manage-dataverse-auditing). The audit views are named: *"You can view audit logs in the **Audit History** tab for a single record and in the **Audit Summary** view for all audited operations in a single environment. You can also retrieve audit logs by using the web API or the SDK for Microsoft .NET."* The questions it answers are exactly the auditor's: *"Who created or updated a record and when? Which fields in a record were updated? What was the previous field value before the update? Who was accessing the system and when? Who deleted a record?"*

**Verified (✅) — solution changes are auditable.** Among the *"Common entities across Dynamics 365 apps"* that can be enabled for auditing in one click, the Security category includes the **`solution`** table (alongside `entity`, `team`, `organization`, `businessunit`, and others) (learn.microsoft.com/power-platform/admin/manage-dataverse-auditing). **This is the fact that makes the Power Platform solution-change history auditable from within Dataverse, not just from pipeline logs.** Auditing is configured at environment, table, and column level; *"You must have a system administrator or system customizer role or equivalent permissions to turn auditing on or off."*

**Verified (✅) — the limits that shape the chain.** *"Auditing isn't supported on table or column definition changes or during authentication. Furthermore, auditing doesn't support retrieve operations or export operations."* To cover retrieve and export, Microsoft points to **activity logging**: *"In addition to Dataverse auditing, you can turn on Dataverse and model-driven apps activity logging to log data retrieve operations and export operations"* — and *"User access or activity logging is sent to Purview for production environments only"* (learn.microsoft.com/power-platform/admin/manage-dataverse-auditing). Reporting at scale is supported via **Azure Synapse Link for Dataverse** (link the audit table, report in Power BI) and **Microsoft Purview** for activity-log data. Audit log retention is configurable, with *"Default: Forever"* and a custom maximum of 24,855 days; audit logs *"consume log storage capacity."*

**Verified (✅) — admin-center Inventory.** *"Power Platform inventory gives tenant administrators a unified view of all agents, apps, and flows built on Power Platform across their organization."* Resources *"appear within 15 minutes"*, the inventory can be *"Export[ed]… to a CSV file"*, and it is accessible via the admin center UI, the **Power Platform for Admins V2 connector**, the **Power Platform API** / **Inventory API**, and **Azure Resource Graph** (learn.microsoft.com/power-platform/admin/power-platform-inventory). **Inventory answers "what exists and who owns it", not "who deployed it when" — the deployment chain itself comes from the pipeline run history plus solution/Dataverse auditing.**

**⚠-knowledge — assembling the chain.** No single Microsoft page describes a bank-grade "who-deployed-what-when" chain end to end; this guide's synthesis (⚠-knowledge, from the verified pieces above) is a four-link chain: (1) **pipeline run history** (which stage, which approver, which delegation identity — §5.2), (2) the **solution version** stamped by the build server (§3.2), (3) the **platform's automatic export of the deployed artifact** to the pipelines host (§5.3), and (4) **Dataverse auditing of the `solution` table** plus record-level audit for post-deployment changes (§11.3). An auditor who asks "why is this app different in production?" should be answerable by walking those four links.

### 11.4 Rollback and Recovery Practice

**Verified (✅) — what the lifecycle gives you.** The four lifecycle actions (§1.4) are the primary in-place recovery tooling: an **update** deploys to the parent managed solution and *"doesn't create an additional solution layer"* but *"can't delete components"*; an **upgrade** *"delete[s] components that existed but are no longer included in the upgraded version"* and rolls patches into a new version; a **patch** is *"small updates (similar to a hotfix)"* layered on the parent, and is the lever *"when… you require it to possibly be uninstalled"* (learn.microsoft.com/power-platform/alm/solution-concepts-alm). **Implication: a patch is the platform's designed "reversible small change"; an upgrade is the designed "compacting change"; but neither is a one-click downgrade to the previous version.**

**Verified (✅) — the automatic artifact backup.** *"Both managed and unmanaged solutions are automatically exported and stored in the pipelines host for every deployment"* (learn.microsoft.com/power-platform/alm/pipelines). This is the deployment history's raw material: the previous version's artifacts exist, on the host, without a separate backup step.

**Verified (✅) — environment backup and restore.** *"The system automatically creates backups for environments that have a database. By default, the system retains backups of all production and nonproduction environments for seven days. However, for production managed environments, you can extend the retention period up to 28 days"*; manual backups can be created *"before major customizations, applying a version update, or making significant changes to the environment"* (learn.microsoft.com/power-platform/admin/backup-restore-environments). Key operational limits, all documented: *"You can't directly restore backups to production environments. To restore a backup to a production environment, first change the environment type to sandbox, perform the restore, and then switch the environment type back to production"*; *"You must restore an environment in the same region where it was backed up"*; *"You can restore a managed environment only to another managed environment"*; *"Restoration of an environment requires 1 GB of available capacity"*; and *"Backup and restore operations include only apps… and flows… in a Dataverse solution"* — i.e. not all configuration. A recently deleted environment can be recovered *"within seven days of deletion"*. The CLI mirrors this: `pac admin backup`, `pac admin restore`, `pac admin list-backups`, `pac admin set-backup-retention-period` (learn.microsoft.com/power-platform/developer/cli/reference/admin).

**⚠ Flag — where the documented position is thin.** Microsoft documents the *mechanics* of solutions, patches, and environment restore, but **does not document a supported "roll a managed solution back to version N-1" operation**. The realistic, documented recovery options are: (a) **redeploy the previous artifact** from the pipelines host / build repository as a new version (a forward move, not a true rollback); (b) **uninstall** the managed solution (which *"removes all the customizations and extensions"* and can lose data in custom tables/columns — learn.microsoft.com/power-platform/alm/solution-concepts-alm); or (c) **restore the environment** from backup (which requires the sandbox-type flip for production, is region-bound, and rolls back data as well as solution). **A bank should treat "rollback" as "redeploy the prior known-good artifact", rehearse it, and document that a restore is the nuclear option** — not assume a version-downgrade button exists.

### 11.5 CoE and Admin-Center ALM Tooling (Condensed)

**Verified (✅) — the CoE Starter Kit has stopped.** *"The Power Platform CoE Starter Kit is no longer actively maintained. Its core capabilities are part of the Power Platform admin center. Issues are no longer reviewed or addressed."* The transition is spelled out: core capabilities *"map to Power Platform admin center capabilities"* — **Inventory** (*"view and govern all apps, flows, and agents created across your tenant"*), **Usage** (*"track adoption and identify top resources and their owners"*), **Monitor** (*"track the operational health of heavily used resources"*), and **Actions** (*"identify risks, enforce best practices, and take action on governance insights"*). The kit *"remains available for existing and new deployments, but it will not be enhanced with new capabilities"* (learn.microsoft.com/power-platform/guidance/coe/starter-kit). Microsoft directs organisations to the admin center *"as the central experience for governance, monitoring, and insights"*, with programmatic access via the CLI, the Power Platform API, the inventory API, and the **Power Platform for Admins V2** connector.

**Verified (✅) — the ALM Accelerator has stopped too.** *"The ALM Accelerator is deprecated and no new features are being added. Issues are no longer reviewed or addressed."* Microsoft's replacement direction is explicit: *"Use Pipelines in Power Platform to bring ALM automation capabilities to Power Platform and Dynamics 365 services. Pipelines can be used with source code integration or extended to integrate with Azure DevOps, GitHub, and other providers"* (learn.microsoft.com/power-platform/guidance/coe/almaccelerator-components). **Two ALM toolkits a bank might have adopted in 2023 — the CoE Starter Kit and the ALM Accelerator — are both now deprecated/maintenance-only in favour of native admin-center capabilities and native pipelines.** That is a portfolio decision, not a footnote.

**Cross-reference.** [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) §9.5 owns the CoE Starter Kit transition at platform level and §9.6 the pipelines-vs-ALM-Accelerator comparison; this section records only the deployment-engineering consequence.

---

## 12. The Banking and Regulated Context

**Scope note.** This chapter is deliberately condensed. The regulated-context depth lives in the banking companions, which this guide cross-references rather than re-derives: change-management and resilience obligations → [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md); the enterprise risk and three-lines-of-defence architecture → [../banking/enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md); MAS technology-risk expectations → [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md); platform policy and sourcing → [../management/it_strategy_guide.md](../management/it_strategy_guide.md) and [../management/vendor_management_guide.md](../management/vendor_management_guide.md).

### 12.1 Change Management and Separation of Duties

**⚠-knowledge (cross-referenced practice).** A bank running Power Platform at scale must fit the low-code estate into its existing change-management and separation-of-duties regime. The deployment-engineering implications of the controls the banking companions describe are:

- **Change authority is not the maker.** The platform's own access guidance — *"Production: Admins and app users. App makers and developers shouldn't have access, or should only have user-level privileges"* (learn.microsoft.com/power-platform/alm/basics-alm) — is the product-level expression of separation of duties: the person who builds is not the person who releases, and the person who releases is not the sole approver.
- **Approval is a recorded action, not a chat message.** The native delegated-deployment approval flow (§5.2) produces a pipeline stage run and an approval-status record — evidence, not an email thread.
- **The deploying identity is a controlled machine identity, not a person's login.** The application user / service principal model (§7) is what lets the bank answer "who deployed?" with a governed, non-human identity that cannot be phished into deploying.
- **Emergency changes still traverse a pipeline.** A patch (the documented hotfix mechanism, §1.4) is the platform's concession to urgency without abandoning the managed-solution discipline.

### 12.2 Auditability: The Chain an Auditor Walks

**Verified where possible.** An auditor asking "prove that the production app equals what was tested, and tell me who deployed it" can be answered, on the Power Platform, from verified artefacts: the **frozen solution artifact** that *"must pass through pipeline stages in sequential order"* and cannot be tampered with (learn.microsoft.com/power-platform/alm/pipelines); the **solution version** stamped by the build server (§3.2); the **pipeline run history** and its linked approval flow (learn.microsoft.com/power-platform/alm/delegated-deployments-setup); the **automatic managed-and-unmanaged export** stored in the pipelines host per deployment (learn.microsoft.com/power-platform/alm/pipelines); and **Dataverse auditing** of the `solution` table and record changes, with access and export activity logging routed to Purview in production (learn.microsoft.com/power-platform/admin/manage-dataverse-auditing). **The gap an auditor will probe is the one this guide flags in §11.4: the platform documents recovery mechanics but not a version-downgrade operation.** The controlled answer is a rehearsed "redeploy prior known-good artifact" drill (see §13.6), not an assumption of rollback.

### 12.3 Why Power Platform Is Not Exempt from Bank Discipline

**⚠-knowledge.** The platform is a bank's data plane as much as its compute plane: it holds Dataverse tables, runs flows that move financial data, and exposes external surfaces (Power Pages). The deployment-engineering disciplines in this guide — managed-only promotion, source-control-as-truth, checker gates, service-principal deployment, audited approvals, frozen artifacts — are the mechanisms that let a low-code platform inherit the bank's existing change-management and audit controls rather than sit outside them. This is where the platform guide's §10 (banking angle) and this guide meet: the platform guide says *what controls are needed*; this guide says *how the deployment pipeline enforces them*.

---

## 13. Cymbal Bank Worked Example: A CI/CD Rollout

> **Cymbal Bank is a fictional, illustrative persona used across this research library.** It is not a real institution and none of the figures, environments, or events below describe a real deployment. The example exists to make the verified mechanics in §1–§12 concrete. Every *platform mechanism* it uses is source-backed in the earlier sections; the *bank's choices* (naming, cadence, thresholds, org structure) are illustrative and marked ⚠-knowledge where they are practice, not product.

### 13.1 The Scenario

Cymbal Bank, Singapore, runs a Power Platform estate for internal operations: a customer-onboarding canvas app, a dispute-intake model-driven app over Dataverse tables, a set of approval flows, and a Copilot Studio agent for internal policy Q&A. The estate began as citizen-developer shadow IT and has been brought under governance (see [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) §11 for the governance rollout). The problem that triggers this programme: **releases are still done by hand** — a maker exports a zip, emails it, an admin imports it, and nobody can prove that what was tested is what is in production. The bank's internal audit has raised a finding on change evidence. This worked example is the CI/CD programme that closes it.

### 13.2 The Environment Topology

Cymbal Bank adopts the documented environment types (§1.5) in a promotion chain:

| Environment | Type | Purpose | Managed? | Who can deploy |
| --- | --- | --- | --- | --- |
| `CB-Dev-<team>` | Developer | Maker development, unmanaged solutions | No (per docs) | Makers + developers |
| `CB-Host` | Production | Pipelines host; deployment history and artifacts | Yes (recommended) | Admins only |
| `CB-Test` | Sandbox / Developer | Automated tests, checker, integration | **Yes** | Pipeline identity |
| `CB-UAT` | Sandbox | Business acceptance; representative data | **Yes** | Pipeline identity + UAT users |
| `CB-Prod` | Production | Live apps and flows | **Yes** | Delegated service principal |

This maps to the documented rule that *"All other environments used in pipelines must be enabled as managed environments"* while the host *"should be a production environment, but the pipelines host doesn't have to be a managed environment"* (learn.microsoft.com/power-platform/alm/pipelines). **⚠-knowledge:** Cymbal Bank chooses to make the host a managed environment anyway, for the premium controls; that is a choice, not a requirement.

### 13.3 The Branch Strategy

Cymbal Bank uses **trunk-based development with release branches** (one of the documented common strategies, §2.5):

- `main` — the integration trunk; every merge triggers a **build** (unpack → check → pack managed → publish artifact).
- `release/<version>` — cut from `main` at a business-approval gate; the source of the artifact promoted through `Test` → `UAT` → `Prod`.
- `feature/<ticket>` — short-lived maker/developer branches, merged to `main` via pull request with the solution checker as a required check.

The per-team `CB-Dev-<team>` environments implement the documented **dedicated-development-environment** strategy for the estate's flagship apps, with a shared **base solution** (common tables) exported as managed and layered beneath each app solution (§2.2). **⚠-knowledge:** the exact cadence (a two-week release train) is Cymbal Bank's choice; Microsoft documents the strategies, not the cadence.

### 13.4 The Pipeline Stages and Gates

Cymbal Bank wires a **build pipeline** and a **release pipeline**, using Azure DevOps Build Tools 2.0 only (§3.1) and native pipelines as the governed front door (§5). The stage/gate table:

| Stage | Runs on | Steps (verified task/tool names in **bold**) | Gate to proceed |
| --- | --- | --- | --- |
| **Build** | PR merge on `main` / `release/*` | **Power Platform Tool Installer**; **Export Solution** (unmanaged) from dev; **Unpack Solution**; **Set Solution Version** from `$(Build.BuildId)`; **Pack Solution** (managed); `pac solution create-settings` to refresh the settings template | Checker clean; build succeeds |
| **Check** | Same agent | **Power Platform Checker** with `RuleSet` = Solution checker, `ErrorLevel`/`ErrorThreshold` set; publish the **SARIF** artifact (`ArtifactDestinationName`) | No Critical issues above threshold |
| **Deploy-Test** | `CB-Test` | **Power Platform Import Solution** with `UseDeploymentSettingsFile`; **Publish Customizations**; post-deploy smoke tests | Import success + smoke tests pass |
| **Deploy-UAT** | `CB-UAT` | Import as above; automated UI tests (Playwright-based, ⚠ see §9); business UAT sign-off | UAT acceptance recorded |
| **Deploy-Prod** | `CB-Prod` | Native **pipeline stage** with **`Is delegated deployment`** = Service Principal; or `pac pipeline deploy`; import with the production settings file | Recorded approval via the delegated-deployment approval flow |

**Verified anchors for each gate.** The checker threshold is `ErrorLevel` + `ErrorThreshold` and `FailOnPowerAppsCheckerAnalysisError` (§8.2). The deployment-settings wiring is `UseDeploymentSettingsFile` / `DeploymentSettingsFile` (§6.3). The `AsyncOperation`/`MaxAsyncWaitTime` parameters matter for larger solutions (§3.2). The production approval is the delegated-deployment flow with the `OnApprovalStarted` trigger and the `UpdateApprovalStatus` action (§5.2). **⚠-knowledge:** Cymbal Bank's specific threshold ("zero Critical, ≤5 High per solution") is illustrative.

### 13.5 Environment-Variable and Connection-Reference Design

Cymbal Bank makes every environment-specific reference a **solution component**, not a hard-coded value (§6.1):

| Reference | Type | Dev value | Test | UAT | Prod |
| --- | --- | --- | --- | --- | --- |
| `cb_ApiBaseUrl` | Environment variable (Text) | sandbox URL | test URL | uat URL | prod URL |
| `cb_EnvironmentType` | Environment variable (Text) | `Dev` | `Test` | `UAT` | `Prod` |
| `cb_KeyVaultSecretName` | Environment variable (Secret) | dev secret | test secret | uat secret | prod secret |
| SharePoint connection | Connection reference (`LogicalName` + `ConnectionId`) | dev connection | test connection | uat connection | prod connection |
| Dataverse connector | Connection reference | per env | per env | per env | per env |

Each target environment has its **own deployment settings JSON** checked into the repository (`deployment-settings/test.json`, `.../uat.json`, `.../prod.json`), generated with `pac solution create-settings` and then populated with the target's connection IDs and variable values (§6.3). **The values are per-environment data; the schema is source-controlled.** ⚠-knowledge: Cymbal Bank keeps *non-secret* values in the settings files and routes **Secret**-typed environment variables through Azure Key Vault, so no credential is ever committed.

### 13.6 The Rollback Drill

Because §11.4 established that the platform has no documented version-downgrade button, Cymbal Bank treats rollback as a **rehearsed forward operation**, not an assumed feature. The drill, run quarterly and before any release train:

1. **Capture the known-good.** Before the release window, record the current production solution version and confirm both the managed and unmanaged artifacts exist in `CB-Host` (the platform's automatic per-deployment export, §5.3).
2. **Take a manual backup.** Create a manual environment backup of `CB-Prod` (documented as appropriate *"before major customizations, applying a version update, or making significant changes"* — learn.microsoft.com/power-platform/admin/backup-restore-environments), and confirm its retention.
3. **Simulate the bad release.** Deploy the new version to `CB-UAT`, then execute the rollback: **redeploy the previous known-good managed artifact** as a new (forward) version via the release pipeline, using the `HoldingSolution` → **Apply Solution Upgrade** path where a component set changed (§1.4), or a plain **Import Solution** update where it did not.
4. **Verify recovery.** Confirm the app/flow behaviour matches the prior version and that Dataverse auditing shows the solution-table change (§11.3).
5. **Prove the nuclear option.** Confirm that a production restore requires the documented sandbox-type flip and the region and capacity conditions (§11.4) — and document that this is *not* the first-choice recovery.
6. **Record the drill.** The drill itself is evidence: timestamp, participants, the artifact version redeployed, and the outcome go into the change-management record.

**⚠-knowledge.** The specific drill steps are Cymbal Bank's design, built only from documented mechanisms. The one Microsoft-sourced hard truth the drill encodes is that **if you cannot redeploy a prior artifact, your only documented recovery is an environment restore — which requires the sandbox flip and rolls back data.** That is why the drill is run *before* the release, not discovered *during* an incident.

### 13.7 Outcomes and What It Took

After a year, Cymbal Bank reports: releases are pipeline-driven, the internal-audit finding is closed by a **walkable who-deployed-what-when chain** (§11.3, §12.2), the solution checker is a merge gate rather than a maker suggestion, and the production approval is a recorded delegated-deployment action. What it took, in the bank's own words, was **not** the tooling — the Build Tools and the CLI are free and documented — but the discipline: one publisher, one solution-per-concern, managed-only promotion, a service principal per environment, a rehearsed rollback, and a refusal to let a production environment accrue unmanaged layers. **That last point is the whole ballgame:** on this platform, a healthy pipeline produces managed artifacts and nothing else, because an unmanaged production system is one whose source of truth is whatever a maker last clicked.

---

## 14. Claims Audit, Glossary, and Closing

### 14.1 Claims Audit

Each row states a load-bearing claim, its status, and the primary source named inline. ✅ = verified this pass; ⚠ = approximate/vendor/evolving; ⚠-knowledge = practice not re-verified; ❌ = could not be verified.

| # | Claim | Status | Source |
| --- | --- | --- | --- |
| 1 | Solutions are the mechanism for ALM; components include tables, apps, flows, agents, charts, plug-ins | ✅ | learn.microsoft.com/power-platform/alm/overview-alm |
| 2 | Solutions contain metadata and config tables, not business data; some system tables can't be added | ✅ | learn.microsoft.com/power-platform/alm/basics-alm |
| 3 | A solution can be up to 95 MB | ✅ | learn.microsoft.com/power-platform/alm/solution-concepts-alm |
| 4 | Managed = deployed; unmanaged = developed; can't edit managed components directly | ✅ | learn.microsoft.com/power-platform/alm/solution-concepts-alm |
| 5 | A managed solution can't be imported into the environment holding its originating unmanaged solution | ✅ | learn.microsoft.com/power-platform/alm/solution-concepts-alm |
| 6 | Unmanaged layer is a single layer; managed layers stack last-one-wins/merge; system layer at base | ✅ | learn.microsoft.com/power-apps/maker/data-platform/solution-layers |
| 7 | Publisher owns components and can't change later; prefix avoids collisions | ✅ | learn.microsoft.com/power-platform/alm/solution-concepts-alm |
| 8 | Lifecycle: create/update/upgrade/patch; update and patch can't delete components, upgrade can | ✅ | learn.microsoft.com/power-platform/alm/solution-concepts-alm; basics-alm |
| 9 | Environment types: sandbox, production, developer, default; documented access split | ✅ | learn.microsoft.com/power-platform/alm/basics-alm |
| 10 | Environment routing moves makers from the default environment to personal environments | ✅ | learn.microsoft.com/power-platform/guidance/adoption/environment-strategy |
| 11 | Git integration is for developer environments, not test/production; YAML is always written | ✅ | learn.microsoft.com/power-platform/alm/git-integration/overview |
| 12 | SolutionPackager is no longer recommended; use the pac CLI | ✅ | learn.microsoft.com/power-platform/alm/solution-packager-tool |
| 13 | Two folder formats (XML legacy, YAML source control); YAML supports multi-solution and .msapp | ✅ | learn.microsoft.com/power-platform/alm/use-source-control-solution-files |
| 14 | Three solution-organisation strategies; dedicated-dev-environment is for strict governance/CI-CD | ✅ | learn.microsoft.com/power-platform/alm/organize-solutions |
| 15 | Build Tools 2.0 is CLI-based; version 1.0 is PowerShell; can't mix versions | ✅ | learn.microsoft.com/power-platform/alm/devops-build-tools |
| 16 | Build Tools four categories: Helper, Quality check, Solution, Environment management | ✅ | learn.microsoft.com/power-platform/alm/devops-build-tools |
| 17 | The Azure DevOps task names in §3.2 | ✅ | learn.microsoft.com/power-platform/alm/devops-build-tool-tasks |
| 18 | Checker task controls: RuleSet, ErrorLevel, ErrorThreshold, FailOn…, RulesToOverride, SARIF artifact | ✅ | learn.microsoft.com/power-platform/alm/devops-build-tool-tasks |
| 19 | GitHub Actions include export/import/unpack/pack/publish-solution, check-solution, deploy-package, who-am-i | ✅ | github.com/microsoft/powerplatform-actions; .../devops-github-available-actions |
| 20 | GitHub Actions and Build Tools are free (subscription required) | ✅ | .../devops-github-actions; .../devops-build-tools |
| 21 | pac command groups and the pac solution/pipeline/admin subcommands in §4 | ✅ | learn.microsoft.com/power-platform/developer/cli/reference/* |
| 22 | `pac test` is deprecated; `pac canvas pack/unpack` are deprecated/preview | ✅ | .../cli/reference/ ; .../cli/reference/canvas |
| 23 | Pipelines host should be production but not necessarily managed; all other pipeline envs must be managed | ✅ | learn.microsoft.com/power-platform/alm/pipelines |
| 24 | Feb 2026: Microsoft auto-enables managed environments for pipeline targets | ✅ | learn.microsoft.com/power-platform/alm/pipelines |
| 25 | One artifact, frozen at submit, sequential stages, tamper-proof | ✅ | learn.microsoft.com/power-platform/alm/pipelines |
| 26 | Pipelines prevalidate missing dependencies and connection/env-variable configuration | ✅ | learn.microsoft.com/power-platform/alm/pipelines |
| 27 | Both managed and unmanaged solutions auto-exported to the host every deployment | ✅ | learn.microsoft.com/power-platform/alm/pipelines |
| 28 | Delegated deployments: SPN/stage-owner, roles, `OnApprovalStarted`, `UpdateApprovalStatus` (20/30), sharing rules | ✅ | learn.microsoft.com/power-platform/alm/delegated-deployments-setup |
| 29 | Environment variables: definition + value; types incl. Secret; unpacked to source control | ✅ | learn.microsoft.com/power-apps/maker/data-platform/environmentvariables |
| 30 | Deployment settings JSON shape (EnvironmentVariables / ConnectionReferences) and import consumption | ✅ | learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools |
| 31 | Three connection types; workload identity federation is recommended | ✅ | learn.microsoft.com/power-platform/alm/devops-build-tools |
| 32 | `pac admin create-service-principal` registers an app and adds an app user | ✅ | learn.microsoft.com/power-platform/alm/devops-build-tools |
| 33 | Solution checker categories, severities, rulesets, SARIF; doesn't guarantee import success | ✅ | .../use-powerapps-checker; .../checker-api/overview |
| 34 | Test Engine is deprecated (April 2026); replacement is Playwright-direct | ✅ | learn.microsoft.com/power-platform/important-changes-coming |
| 35 | Test Studio is low-code, expression-driven, with the documented known limitations | ✅ | learn.microsoft.com/power-apps/maker/canvas-apps/test-studio |
| 36 | Dataverse auditing: Audit History/Audit Summary, questions answered, `solution` table auditable, Purview for prod access logs | ✅ | learn.microsoft.com/power-platform/admin/manage-dataverse-auditing |
| 37 | Backup: 7 days default, up to 28 for production managed; no direct prod restore; region-bound; 1 GB needed | ✅ | learn.microsoft.com/power-platform/admin/backup-restore-environments |
| 38 | CoE Starter Kit no longer actively maintained; capabilities in admin center Inventory/Usage/Monitor/Actions | ✅ | learn.microsoft.com/power-platform/guidance/coe/starter-kit |
| 39 | ALM Accelerator is deprecated; use pipelines in Power Platform | ✅ | learn.microsoft.com/power-platform/guidance/coe/almaccelerator-components |
| 40 | Admin-center Inventory: unified view, 15-minute freshness, CSV export, API/connector/ARG access | ✅ | learn.microsoft.com/power-platform/admin/power-platform-inventory |
| 41 | No dedicated CI/CD or DevOps/release-engineering guide exists under technology/ — this guide fills that gap | ✅ | repository listing (technology/) |
| 42 | Practice guidance (branching cadence, thresholds, rollback drill, pre-deployment checklist) is this guide's synthesis | ⚠-knowledge | §10, §12.1, §13 |
| 43 | DLP-blocked connector breaks a deployed app/flow even on a green import | ⚠-knowledge | §11.2 (DLP anatomy owned by platform guide §9.4) |
| 44 | GitHub Actions `upload-pipeline-artifact` / `set-connection-variables` / `publish-customizations` exist | ❌ (not found) | §14.2 |
| 45 | Test Engine YAML test-plan schema and supported-app-type matrix | ❌ (unverifiable — feature deprecated, docs removed) | §14.2 |
| 46 | `pac paportal` as a current command group | ❌ (superseded by `pac pages`) | §14.2 |

### 14.2 What Could Not Be Verified

**The following could not be verified at primary source on this pass (2026-09-10). They are logged rather than asserted, and no claim in the body of this guide depends on them.**

1. **GitHub Actions named `upload-pipeline-artifact` / `download-pipeline-artifact`.** The `microsoft/powerplatform-actions` repository listing shows `upload-paportal` / `download-paportal` (Power Pages) but **no combined pipeline-artifact action**. ❌ — the names in the brief do not match the published action set. The verified solution-artifact actions are `export-solution`, `import-solution`, `pack-solution`, `unpack-solution`, `clone-solution`, `branch-solution`, and `build` (§3.3).
2. **GitHub Action `set-connection-variables`.** No action of this name appears in the repository listing. ❌ The equivalent capability on the Azure DevOps side is the **Power Platform Set Connection Variables** *task*; there was no verified GitHub mirror.
3. **GitHub Action `publish-customizations`.** No such action. ❌ The GitHub action is `publish-solution`; the Azure DevOps task is **Power Platform Publish Customizations** (§3.3). The names are close enough to be a plausible source of a copy-paste error — verify before use.
4. **Test Engine's YAML test-definition schema and supported app types.** The Test Engine documentation URL redirects to the deprecations page, and Microsoft states the documentation *"and GitHub repository are no longer maintained… and will be removed."* ❌ The schema and app-type matrix could not be verified. **Do not build a 2026 test strategy on Test Engine** (§9).
5. **The Power Platform Playwright samples landing page.** The URL cited in the deprecation notice (`.../test-engine/developer/playwright-samples/overview`) returned **HTTP 404** on this pass. ⚠ The *direction* (use Playwright directly) is verified; the specific sample page could not be confirmed.
6. **`pac paportal` as a current command group.** The current CLI reference lists `pac pages`, not `pac paportal`. ❌ Older tutorials referencing `pac paportal` are superseded; verify against the live index.
7. **A supported managed-solution "downgrade to version N-1" operation.** No such operation is documented; the documented options are redeploy-previous-artifact, uninstall, or environment restore (§11.4). ⚠ This is a *documentation thinness* finding rather than a missing feature — but it must not be assumed away.
8. **A single Microsoft-published pre-deployment validation checklist.** No such page was found; the checklist in §10 is this guide's synthesis from verified mechanisms. ⚠-knowledge.
9. **Live `web_search`.** The search tool returned empty results on this pass; research was done by direct `web_extract` against named primary URLs. ⚠ The consequence is that *discovery* of pages beyond those named in the brief was limited, though all cited pages were retrieved directly.

### 14.3 Glossary

- **ALM (Application Lifecycle Management)** — *"the lifecycle management of applications, which includes governance development and maintenance"*, spanning requirements, architecture, development, testing, maintenance, change management, support, CI, project management, deployment, release management, and governance (learn.microsoft.com/power-platform/alm/overview-alm).
- **Build Tools** — Microsoft Power Platform Build Tools for Azure DevOps, a set of Azure DevOps tasks (v2.0 CLI-based) for solutions (learn.microsoft.com/power-platform/alm/devops-build-tools).
- **Component** — *"something that you can potentially customize"*; anything includable in a solution (learn.microsoft.com/power-platform/alm/solution-concepts-alm).
- **Connection reference** — a solution component that carries a logical connection whose physical connection ID is supplied per environment (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools).
- **Delegated deployment** — a pipeline stage that deploys *"as the delegate (service principal or pipeline stage owner) instead of the requesting maker"* (learn.microsoft.com/power-platform/alm/delegated-deployments-setup).
- **Deployment settings file** — the JSON file pre-populating environment variables and connection references at import (learn.microsoft.com/power-platform/alm/conn-ref-env-variables-build-tools).
- **Environment variable** — a solution component storing a parameter key and value so config differs per environment without changing the artifact (learn.microsoft.com/power-apps/maker/data-platform/environmentvariables).
- **Git integration (Dataverse)** — native source-control sync of solutions and solution objects to a Git provider, used in developer environments (learn.microsoft.com/power-platform/alm/git-integration/overview).
- **Managed environment** — an environment with premium governance capabilities activated; required for pipeline targets other than the host and developer environments (learn.microsoft.com/power-platform/alm/pipelines).
- **Managed solution** — the deployable solution form; can't be edited directly and is removed in full on uninstall (learn.microsoft.com/power-platform/alm/solution-concepts-alm).
- **Patch** — a solution containing only changes to a parent managed solution, layered on top, for hotfix-style updates (learn.microsoft.com/power-platform/alm/solution-concepts-alm).
- **Pipelines in Power Platform** — the native, in-product hosted deployment capability with stages, approvals, and frozen artifacts (learn.microsoft.com/power-platform/alm/pipelines).
- **Publisher** — the owner of solution components, defined once; includes the prefix that avoids naming collisions (learn.microsoft.com/power-platform/alm/solution-concepts-alm).
- **Solution** — *"the mechanism for implementing ALM"*; the unit of deployment (learn.microsoft.com/power-platform/alm/overview-alm).
- **Solution checker** — the static-analysis service assessing solution objects against best-practice rules, with severities Critical→Informational (learn.microsoft.com/power-apps/maker/data-platform/use-powerapps-checker).
- **SolutionPackager** — the legacy tool (superseded by `pac solution`) that reversibly unpacks a solution zip into files (learn.microsoft.com/power-platform/alm/solution-packager-tool).
- **Unmanaged solution** — the development-form solution; the source of truth checked into source control (learn.microsoft.com/power-platform/alm/solution-concepts-alm).

### 14.4 Closing

The Power Platform gives a bank an unusual gift: a deployment model whose *default* behaviour — managed solutions moving through stages as frozen, tamper-proof artifacts, with approvals recorded and both solution forms archived — is exactly the shape an auditor wants, provided the estate is disciplined about it. The risk is the opposite of the platform's marketing: that the maker-friendly front door becomes an excuse to skip the source-controlled build, the checker gate, the service-principal deployment, and the rehearsed rollback. This guide has tried to show that the engineering is not exotic — a solution project in Git, a build pipeline running the documented Build Tools tasks, `pac` for the commands, a deployment settings file per environment, a service principal per stage, the checker as a merge gate, and a rollback drill that assumes nothing the platform does not document. Get those right, and the low-code estate stops being a shadow-IT liability and becomes simply another governed system whose every change can be traced from a maker's pull request to the production environment. The discipline is the product; the pipeline is only the machinery that delivers the deployed solution.
