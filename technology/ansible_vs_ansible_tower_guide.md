# Ansible vs Ansible Tower — The Engine, the Control Plane, and the Line They Draw

> **Author:** Jack Liu Shurui, Solution Architect
> **Generated:** 2026-09-13
> **Scope:** A deep-dive on the boundary between **ansible-core** (the free, agentless engine and its CLI) and the commercial enterprise control plane known successively as **Ansible Tower → (upstream) AWX → Red Hat Ansible Automation Platform (AAP)**. The spine is the **naming lineage** — because the lineage itself encodes the capability boundary: what the engine gives you for nothing, what the control plane adds, what it costs, and at what point a team or an institution should cross that line. Facts marked **(verified)** were confirmed against primary vendor/project sources in **September 2026**; dates of verification are given inline for naming, version and licence claims, because these are exactly the claims that go stale fastest. Figures that are approximate, unpublished, or that could not be confirmed at a primary source are flagged **(flagged)** and are listed explicitly in §11.

**Related guides in this series:** [configuration_management_languages_guide.md](configuration_management_languages_guide.md) (the CM-language comparison — Ansible's YAML/Jinja2 model vs Puppet/Chef/Terraform/CUE) · [lightweight-config-mgmt-analysis.md](lightweight-config-mgmt-analysis.md) (lighter alternatives to Ansible: Salt, Fabric, Rex, single-binary tools) · [kargo_gitops_guide.md](kargo_gitops_guide.md) (the GitOps-controller contrast) · [iac_best_practices_guide.md](iac_best_practices_guide.md) (general IaC practice, CI/CD and pipeline design) · [power_platform_cicd_guide.md](power_platform_cicd_guide.md) (platform CI/CD) · [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (the operational-resilience anchor)

---

## Table of Contents

1. [The Overview and the Naming Problem](#1-the-overview-and-the-naming-problem)
2. [What the Engine Gives You: ansible-core, Verified](#2-what-the-engine-gives-you-ansible-core-verified)
3. [What the Control Plane Adds, Component by Component](#3-what-the-control-plane-adds-component-by-component)
4. [Versions, Packaging and Licensing](#4-versions-packaging-and-licensing)
5. [The Comparison Table](#5-the-comparison-table)
6. [The Decision Framework](#6-the-decision-framework)
7. [The Regulated-Enterprise and Banking Angle](#7-the-regulated-enterprise-and-banking-angle)
8. [The Alternatives Landscape](#8-the-alternatives-landscape)
9. [The Cymbal Bank Worked Example](#9-the-cymbal-bank-worked-example)
10. [The Claims Audit](#10-the-claims-audit)
11. [What Could Not Be Verified](#11-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [Cross-References and Further Reading](#13-cross-references-and-further-reading)
14. [Closing Summary](#14-closing-summary)

---

## 1. The Overview and the Naming Problem

Ask an infrastructure team in 2026 whether they "use Ansible Tower" and you will get one of three answers: *yes* (they are running something they installed years ago and never renamed), *no, we use AAP* (they have paid for the successor), or *what is Tower?* (they installed `ansible-core` from PyPI and think the question is about a building).

All three answers are consistent, because the same lineage of software has carried **at least four names** in twelve years, and each name change marked a genuine change in what the software *was* — commercial product, open-source upstream, or subscription platform. Decoding that lineage is not trivia. It is the fastest way to understand the capability boundary that this guide is about, because the boundary and the naming moved together.

### 1.1 The lineage, laid out precisely

| Era | Name | What it actually was | Status today |
| --- | --- | --- | --- |
| 2013–2014 | **Ansible Tower** (Ansible, Inc. / AnsibleWorks) | The first **commercial** product built around the open-source Ansible engine: a web UI, REST API and task engine that stored job results and gave large teams auditability. Per Red Hat's own chief architect, the team worked "for over a year to make the first commercial release of Ansible Tower" starting in **2013** **(verified** — ansible.com blog *Upcoming Changes to the AWX Project*, Matthew Jones, Red Hat Chief Architect, Ansible Automation) | Historic; still the name many teams use |
| 2015 | **Ansible Tower** under Red Hat | Red Hat announced on **16 October 2015** that it had signed a definitive agreement to acquire Ansible, Inc.; the press release states the transaction "is expected to close in **October 2015**", and lists Red Hat's management portfolio as **CloudForms, Satellite, and "Ansible and Ansible Tower"** **(verified** — redhat.com press release *Red Hat to Acquire IT Automation and DevOps Leader Ansible*, dated RALEIGH, N.C. — October 16, 2015; re-read 2026-09-13) | Historic |
| 2017 onward | **AWX** | The **open-source upstream project** of Tower: "AWX provides a web-based user interface, REST API, and task engine built on top of Ansible. It is **one of the upstream projects for Red Hat Ansible Automation Platform**" **(verified** — github.com/ansible/awx README, read 2026-09-13). Licence: **Apache 2.0** (README licence badge) | Alive but **frozen**: README carries a caution that "The last release of this repository was released on **Jul 2, 2024**" and "**Releases of this project are now paused during a large scale refactoring**" **(verified** — same README, 2026-09-13) |
| 2021 | **Red Hat Ansible Automation Platform 2** (AAP) | The renaming and re-architecture. Red Hat's launch blog for AAP 2 states the release "includes **automation controller 4.0**, the **improved and renamed** Red Hat Ansible Tower" **(verified** — redhat.com blog *What's new in Ansible Automation Platform 2: automation controller*, dated **5 October 2021**; companion launch post *Introducing Ansible Automation Platform 2* dated **29 September 2021**) | The current product family |
| 2022–2026 | **AAP 2.x** → current **2.7** | The platform is now a bundle of components (gateway, controller, hub, Event-Driven Ansible, dashboard/analytics) rather than one application; component versions move independently of the AAP version **(verified** — Red Hat AAP lifecycle page, Table 1.2 footnote 2: "AAP is versioned as 2.5, 2.4 etc. The components and services within are versioned with each release and move independently"; read 2026-09-13) | Current |

**The short version:** *Tower* was the commercial product; *AWX* is its open-source upstream; *AAP* is the paid platform that Tower became; and **Tower the component is now called the Automation Controller**. The old name survives in the plumbing: AAP 2.7 documentation still ships an **`awx-manage`** command-line utility ("Use awx-manage to access automation controller information") and the `awx.awx` collection of modules **(verified** — docs.redhat.com AAP 2.7 documentation index and the AWX 24.6.0 release notes, both read 2026-09-13). When you see `awx` in a modern AAP URL, module name or log line, you are looking at the fossil record of Tower.

### 1.2 The decoding table: reading old material correctly

| You read (old material) | Modern equivalent | Notes |
| --- | --- | --- |
| Ansible Tower / "AWX" as a product you buy | **Red Hat Ansible Automation Platform** (subscription) | Tower was never open source; AWX is its Apache-2.0 upstream. Buying "Tower" today means buying AAP |
| Tower 3.x | **Automation Controller** 4.x inside AAP (AAP 2.0 = controller 4.0; AAP 2.7 = controller **4.8**) **(verified** for 4.0 via the Oct-2021 launch blog; for **4.8** via Red Hat's Configuration-as-Code requirements table for AAP 2.7, which requires `ansible.controller >= 4.8`, read 2026-09-13) | The version number jumped 3→4 at the rename, so old upgrade notes are not linearly comparable |
| "Tower licence" | **AAP subscription** (Standard or Premium), or **AWX** if you want the unsupported upstream | See §4 |
| Tower's isolated nodes / instance groups | **Automation mesh** (control / execution / hop nodes) and **instance groups / container groups** | Mesh is the AAP 2.x topology model **(verified** for existence via the AAP 2.7 docs index, 2026-09-13) |
| "Tower analytics" / Ansible Analytics | **Automation dashboard and automation analytics**; in 2.7 a **metrics service** backs the dashboard **(verified** — AAP 2.7 install docs describe "metrics service … its role as the backend for the automation dashboard") | Renamed and re-architected |
| "Ansible Engine" (the 2018–2021 subscription SKU) | **ansible-core** (free, upstream, GPL-3.0-or-later) | The "Ansible Engine" SKU was the paid support wrapper around the engine; the engine itself is and always was open source |
| Tower "workflows" | **Workflow templates** (DAG of job templates) | Same concept, more capability |
| Red Hat Ansible Automation Platform **1.x** | Pre-2 product shell (bundled Tower + hub + services catalog) | Automation Services Catalog was retired; treat AAP 2.x docs as authoritative |

### 1.3 Why the naming problem has teeth

Three practical consequences, in order of how much money they cost:

- **Procurement confusion.** A team asking for "Tower pricing" in 2026 will be quoted **AAP Standard or Premium** — a subscription counted in **managed nodes**, not a per-install licence. Budget comparisons against an old Tower line item are meaningless without re-basing on the node count (§4.3, §5).
- **Upstream-versus-product confusion.** "AWX is free, so why pay?" is a real and defensible question — but in 2026 the honest answer is narrower than it was in 2021, because **AWX releases have been paused since July 2024** pending a large-scale refactor, while the *product* keeps shipping (AAP 2.7, release id **2.7.20260603**) **(verified** — AWX README caution plus AWX 24.6.1 release; AAP 2.7 release-notes component table, all read 2026-09-13). The upstream is no longer a drop-in free stand-in for a moving product; §8 covers the consequence.
- **Documentation drift.** Most blog posts, Stack Overflow answers and internal runbooks that mention "Tower" predate the rename, the mesh, execution environments and the platform gateway. Reading them literally — for example, looking for a `tower-cli` and a per-organization virtualenv — wastes days. §12's glossary is designed to be the decoder ring.

**The capability claim, stated up front.** The engine and the control plane run **the same playbooks**. Nothing in AAP makes a playbook more capable; everything in AAP makes a *fleet and a team* more governable. That single sentence is the whole boundary, and §2, §3 and §6 are its expansion.

---

## 2. What the Engine Gives You: ansible-core, Verified

### 2.1 The agentless push model

Ansible's defining architectural claim is that it needs **nothing installed on the managed node except an interpreter and a reachable transport**. Upstream states it plainly: "Ansible is an **agentless** automation tool that you install on a single host (referred to as the **control node**). From the control node, Ansible can manage an entire fleet of machines and other devices (referred to as **managed nodes**) remotely with **SSH, Powershell remoting, and numerous other transports**, all from a simple command-line interface **with no databases or daemons required**" **(verified** — ansible-documentation repository, `installation_guide/intro_installation.rst`, read 2026-09-13). The upstream package description repeats the design principle: "Avoid custom-agents and additional open ports, be **agentless** by leveraging the existing SSH daemon" **(verified** — PyPI `ansible-core` 2.21.4 project page, read 2026-09-13).

The commercial lineage was sold on exactly this property from day one; Red Hat's 2015 acquisition press release described "Ansible's **simple and agentless** approach, unlike competing solutions, [which] does not require any special coding skills" **(verified** — redhat.com press release, 16 October 2015).

What this buys you, concretely:

- **No fleet-wide agent rollout, no agent upgrade programme, no agent CVE stream.** In a segmented regulated network with long change-approval lead times, this is the single biggest operational advantage over agent-based configuration management. (For the comparative language model — YAML + Jinja2 vs Puppet DSL, Chef/Ruby, HCL and the newer typed-config languages — see [configuration_management_languages_guide.md](configuration_management_languages_guide.md); this guide deliberately does not re-derive it.)
- **Nothing to bootstrap.** Upstream: managed nodes are managed "instantly, without bootstrapping any software" **(verified** — PyPI design principles, 2026-09-13).
- **Push, not pull.** The control node decides when to act. That is what makes ad-hoc work, one-off remediation and "run this now, on these 40 hosts" possible at all — and it is also the property that makes *who may press the button* a governance question the moment more than one person is involved (§6, §7).

**What it does not buy you:** any notion of a *fleet*. The engine has no inventory of record beyond the files or dynamic sources you point it at, no memory of what it did last week, no concept of who is allowed to touch production, and no way to prove any of that to an auditor. Those absences are the entire subject of §3.

### 2.2 The moving parts of the engine

| Capability | What it is | Verified status |
| --- | --- | --- |
| **Inventory** | Static INI/YAML files or dynamically generated host lists via inventory plugins; groups and group-vars give hosts a shape | **(verified** — upstream installation guide treats inventory as part of the "simple command-line interface" model; the AAP docs later describe "smart inventories, constructed inventories, inventory plugins, and source control imports" as the *platform* versions, read 2026-09-13) |
| **Playbooks, roles, collections** | The unit of automation: YAML playbooks composed of tasks, packaged as roles, distributed as collections | **(verified** as the packaging model — the community `ansible` package is explicitly "a set of independent Ansible collections that are curated by the community, and it pulls in ansible-core", PyPI, 2026-09-13) |
| **Ad-hoc commands** | `ansible -m …` one-liners for inspection and quick remediation | **(verified** — PyPI describes "ad-hoc task execution" as a first-class capability; the `ansible` CLI is listed among ansible-core's "core runtime and CLI tools". Note: the *examples* are documented in the upstream ad-hoc guide, which my fetcher could not reach — see §11) |
| **`ansible-pull`** | The pull-flavoured mode: the node checks out a repository and runs the playbook locally against itself, typically from cron | **(verified** as a shipped CLI tool — it appears in upstream's list of Ansible command line utilities for shell completion, `ansible-pull`, 2026-09-13). Useful at the edge and behind one-way network paths; it is *not* a control plane, because each node decides for itself what code to run |
| **`ansible-vault`** | Symmetric encryption of variables and files so secrets can live under source control | **(verified** with an important caveat, below) |
| **Idempotent modules** | Modules declare desired state and report *changed* only when they change something | **(verified** in the sense that this is the documented module contract; treated as background here rather than re-derived — see the CM-language guide) |

### 2.3 ansible-vault: what it really protects, and what it does not

This is the most over-trusted feature in the engine, so the scope has to be exact. Upstream's warning is unambiguous:

> "Encryption with Ansible Vault **ONLY protects 'data at rest'**. Once the content is decrypted ('data in use'), play and plugin authors are responsible for avoiding any secret disclosure, see `no_log` for details on hiding output and vault_securing_editor for security considerations on editors you use with Ansible Vault." **(verified** — ansible-documentation, `vault_guide/vault.rst`, read 2026-09-13)

And the operational cost is stated just as plainly: "To use Ansible Vault you need **one or more passwords** to encrypt and decrypt content. If you store your vault passwords in a **third-party tool such as a secret manager, you need a script to access them**" **(verified** — same file).

Translate that into a regulated estate:

| Question | Answer for a bare CLI estate |
| --- | --- |
| Does it protect secrets committed to Git? | **Yes** — ciphertext at rest, safe to commit **(verified)** |
| Does it protect secrets while the playbook is running? | **No.** Decrypted plaintext exists in memory and, unless `no_log` is used correctly, in task output and logs **(verified**, same warning) |
| Does it tell you *who* decrypted what, and when? | **No.** There is no audit event; the vault password is a shared secret **(flagged** — the *absence* of an audit record is an architectural fact about a CLI tool with no server, not a published statement; flagged because I did not find a primary sentence stating it) |
| Can it enforce rotation, per-user access, or separation of duties? | **No.** Anything that can decrypt can decrypt everything that key protects. Vault IDs let you separate *which* secrets a run needs; they do not give you per-user custody **(flagged** — same reason) |
| Does it replace a secrets manager? | **No.** It is a *transport* for secrets under source control, not a custody system. The likely answer on a regulated estate is vault ciphertext in Git *for* low-value data, and a real secrets manager (plus a script, per the upstream note) for everything that matters **(verified** that the upstream model requires that script) |

### 2.4 The distribution: no ansible-core CLI before the year 2020 — and why the version numbers look wrong

Two packages share the word "Ansible", and mixing them up causes real incidents:

| Package | Contents | Versioning | Maintenance | Licence |
| --- | --- | --- | --- | --- |
| **`ansible-core`** | "a minimalist language and runtime package containing a set of built-in modules and plugins" — the engine, the CLI tools (`ansible`, `ansible-playbook`, `ansible-vault`, `ansible-galaxy`, `ansible-pull`, …) | "Continues 'classic Ansible' versioning (2.11, then 2.12)"; **does not use semantic versioning** | "Maintains latest version plus two older versions"; maintenance "continues for 3 releases" | **GPL-3.0-or-later** |
| **`ansible`** (community package) | "a much larger 'batteries included' package, which adds a community-curated selection of Ansible Collections"; still "pulls in `ansible-core`" | "Uses new versioning (2.10, then 3.0.0)"; "follows semantic versioning rules" | "Maintains only one version at a time" | **GPL-3.0-or-later** |

All rows **(verified** — ansible-documentation `reference_appendices/release_and_maintenance.rst` and `installation_guide/intro_installation.rst`, plus the PyPI project pages for `ansible` and `ansible-core`, all read 2026-09-13).

**Current versions (as of 2026-09-13):**

- **`ansible-core` 2.21.4** — released **8 September 2026**; also current in the 2.20.x and 2.19.x maintenance lines, with 2.18.19 still receiving releases **(verified** — github.com/ansible/ansible releases page and PyPI `ansible-core`, read 2026-09-13). The engine does not use semantic versioning; "2.21.4" means *feature line 2.21, patch 4*, and the parity/maturity rhythm upstream aims for is even-versus-odd core lines **(verified** — the AAP lifecycle page states Red Hat's aim of "stable supported long lifecycles for Ansible Core versions ending with 'even' suffix and short term lifecycle … for 'odd' suffixed Ansible Core versions", read 2026-09-13)**.**
- **`ansible` community package 14.4.0** — released **8 September 2026**, and the community **14** branch pins **`ansible_core_version: 2.21.0`** with Python `>=3.12` **(verified** — PyPI `ansible` project page; `ansible-community/ansible-build-data` file `14/ansible-14.build`, both read 2026-09-13).
- **Documentation lag, flagged:** the same community docs page that defines the versioning also carries a release-status table labelling **14.0.0 "In development (unreleased)"** and **13.x "Current"**, which contradicts PyPI's 14.4.0 release. Either the docs table is stale on the `devel` branch or the release cadence moved under it. **Treat PyPI + `ansible-build-data` as authoritative for what is installable, and treat the docs table as unreliable until re-checked** **(flagged** — discrepancy observed 2026-09-13; it is a documentation inconsistency, not a licensing question).

**How it is invoked in practice.** All three of these are the *same engine*, differing only in who holds the credentials:

1. **A laptop / a bastion.** `ansible-playbook -i inventory site.yml`. Fast, interactive, and the model that quietly becomes the estate's biggest risk when the laptop also holds the production SSH keys and nobody can see what was run (§6.3).
2. **Cron or a systemd timer on a control host.** `ansible-pull` on the node, or a scheduled `ansible-playbook` on a jump host. Cheap, and where most "we have Ansible in production" estates actually live. Weakness: scheduling is whoever wrote the crontab; there is no run history beyond logs you did not rotate.
3. **A CI pipeline.** The engine as one more pipeline step, with the CI system supplying audit, RBAC and secrets — see [power_platform_cicd_guide.md](power_platform_cicd_guide.md) and [iac_best_practices_guide.md](iac_best_practices_guide.md). This is the legitimate "control plane substitute" and §9 compares it head-to-head with AAP.

### 2.5 The engine's honest limits at team scale

None of the following is a criticism; they are consequences of the engine having no server. Each is a *feature* for a single operator and a *gap* for an estate:

| Limit | Consequence at scale | Where the platform answers it |
| --- | --- | --- |
| **No central RBAC** | Access control is SSH key distribution and repository permissions. "Who can patch production?" is answered by *who holds keys* | §3.3 organisations/teams |
| **No credential custody** | Secrets live in vault files, environment variables, `~/.ssh`, or a secrets manager the operator's shell can read. There is no broker that injects a credential into a run and withholds it from the operator | §3.4 |
| **No scheduler of record** | Cron, CI schedules, or someone's memory. Nothing tells you what is *supposed* to have run | §3.7 |
| **No audit trail** | Logs you chose to keep, on hosts you chose to keep them on. No tamper-evident record of *who launched what against which inventory* | §3.10 |
| **No multi-tenancy** | One estate, one set of keys, one blast radius. Two business units sharing an engine share a trust boundary | §3.3, §5 |
| **Fork/serial scaling model** | Concurrency is per-play `forks` (worker processes on the control node) and `serial` batching across hosts; the control node is a single scaling unit, and its CPU/FD/host-key limits become the ceiling | §3.12 mesh and §3.11 execution environments (note: the platform does **not** make a single play run faster — it spreads *jobs* across nodes; **(flagged** as a design characterisation rather than a quoted claim) |
| **No content governance** | Anyone with repo write access can publish a role; no namespace approval pipeline, no signing enforcement | §3.13 private automation hub |
| **No surveyed, safe self-service** | A non-specialist either gets shell access (and the keys), or waits for an engineer | §3.8 surveys, §3.14 portal |

**Bottom line for §2:** the engine gives you *automation* for zero licence cost, runs anywhere, and imposes almost no architecture. It gives you *almost no governance* — and the word that keeps appearing in the right-hand column of the table above is **custody**.

---

## 3. What the Control Plane Adds, Component by Component

Everything in this section exists in AAP; the honest reading is *"the platform does not add automation capability, it adds the ability to let many people automate the same estate without sharing keys or losing the plot."* Each subsection states what the component does **and** what it does not do. Component existence and naming are **(verified)** against the AAP 2.7 documentation index and the Red Hat AAP lifecycle page (both read 2026-09-13) unless noted otherwise.

### 3.1 The REST API

**Does:** exposes the platform's objects (inventories, credentials, job templates, jobs, organisations, users) as a versioned REST API, so everything the UI can do can be automated, integrated and reported on. The AWX lineage means the API is the same shape as the upstream project's: "AWX provides a web-based user interface, **REST API**, and task engine" **(verified** — AWX README, 2026-09-13).

**Does not:** give you an API *for your automation content itself* (it launches playbooks; it does not validate your YAML), and in **AAP 2.7 the API surface moved**: the docs state 2.7 requires **all API access to route through the platform gateway**, with "**removal of the backward compatibility layer**" and "removal of direct component-level token support", and authentication moved to the gateway **(verified** — AAP 2.7 upgrade documentation, *API changes* and *Authentication changes in 2.7*, read 2026-09-13). Any integration written against pre-2.7 per-component endpoints is an upgrade project. **This is a live, high-impact change** — plan it, do not discover it.

### 3.2 The web UI

**Does:** a single interface for operators, developers and administrators, and in 2.7 explicitly an "**Ansible Automation Platform UI**" component (version 2.7.0 in the 2.7 release-notes table) **(verified** — AAP 2.7 release notes component table, read 2026-09-13).

**Does not:** replace the CLI. Engineers still write and test with `ansible-playbook`/`ansible-lint`/dev containers locally; the UI is where jobs are *launched, delegated and observed*. Teams that try to author in the UI as their primary workflow generally regress to Git, because the UI's unit of work is a job template, not a repository **(flagged** — an experience-based characterisation, not a vendor statement).

### 3.3 RBAC: organisations, teams, roles

**Does:** layers access control on top of an identity provider: **organisations** (the tenancy boundary), **teams** (groupings within them), users, and granular permissions on objects — including, per the AWX release notes, "**permissions that come from an external Auditor role**" and role definitions that can be managed as code **(verified** — AWX 24.6.1 release notes, which fix RBAC behaviour for "managed RoleDefinitions" and read permissions for an "organization Audit role", read 2026-09-13). This is the component that answers "who is allowed to patch production" *without* answering it with SSH keys.

**Does not:** invent your duty separation. RBAC is a mechanism; the *model* (who may approve, who may execute, who may only read) is an organisational artefact you must design and then keep in Git — ideally via the Configuration-as-Code collections (`ansible.controller ≥ 4.8`, `ansible.platform ≥ 2.7` for AAP 2.7) **(verified** — Red Hat lifecycle page, Configuration as Code table, read 2026-09-13). Nor does it constrain what a *launched job* can do on the target: RBAC governs who may run *which template*, not what that template does once running.

### 3.4 Central credential management and custody

**Does:** stores credentials as platform objects, attaches them to job templates, and **injects them at run time** so operators can launch a job without ever seeing the secret. Supports machine credentials, vault password(s), cloud/API credentials, and credential types — and externalises them to supported secret managers. It also brokers auth to HashiCorp Vault: in **2.7, AAP can act as a native OIDC identity provider for HashiCorp Vault**, issuing "**short-lived, job-scoped JSON web tokens instead of stored credentials**" **(verified** — AAP 2.7 *What's new*, technology preview, read 2026-09-13; note it is explicitly a **technology preview**, so it is not a production commitment).

**Does not:** remove the secret from memory during a run, and does not make a badly scoped credential safe. The *custody* win is that the operator never holds the key; the *residual risk* is the credential's own scope — a controller credential for SSH as a privileged user is still a privileged credential, now concentrated in one highly-available system. Custody moves with the platform gateway in 2.7 as part of the authentication change **(verified** — AAP 2.7 auth changes doc, read 2026-09-13).

### 3.5 Projects (SCM-backed)

**Does:** binds a job template to a **source-control repository** (branch/tag, plus options for webhook-driven or scheduled project syncs), so the code that ran is identifiable by commit. This is the single most audit-relevant feature in the platform: it converts "we ran a playbook" into "we ran *this commit*".

**Does not:** validate or review your content. It will happily sync a branch that breaks production. Enforce review in the SCM (branch protection, CODEOWNERS) and test in CI before the controller ever sees it — see [iac_best_practices_guide.md](iac_best_practices_guide.md).

### 3.6 Job templates

**Does:** the reusable, parameterised unit of execution: inventory + project + playbook + credential(s) + execution environment + limits/tags + privilege-escalation settings. This is the *delegation primitive*: an operator with execute rights on a template gets exactly that capability, nothing more — the whole reason a non-specialist can safely run "patch this web tier".

**Does not:** restrict what the playbook contains. `become: true` with a root credential inside a template is a root-capability template; §7.2 covers why that is the design decision auditors actually probe.

### 3.7 Scheduling

**Does:** schedules job templates and (in practice) workflow templates with timezone-aware recurrence, with the schedule stored as a platform object — so the estate's *intended* run pattern is declarative and reviewable.

**Does not:** prove anything ran as intended; that is the audit trail's job (§3.10). It also does not make an unsafe scheduled job safe: **scheduled drift-remediation against production without a change record is the most common regulated-estate finding in this space** **(flagged** — practitioner characterisation, no vendor source).

### 3.8 Workflow templates (the node graph) and surveys

**Does:** **workflow templates** compose job templates (and other nodes) into a directed graph with success/failure/always branches — the "orchestration" layer that makes multi-stage change (snapshot → patch → verify → failback) explicit; **surveys** put a typed, validated question set in front of a template so a non-specialist supplies *parameters*, not code. (Both are long-standing platform features; existence is **(verified)** via the AAP docs and feature index, read 2026-09-13.)

**Does not:** make a workflow transactional. There is no rollback of a partially completed graph, no automatic compensating action, and branches on failure are only as good as the author's design. A workflow is a *script of scripts* with a picture attached. And surveys constrain the *input*, not the *outcome*.

### 3.9 Notifications

**Does:** emits events (job start/success/failure/approval) to Slack, Teams, email, webhooks and similar, per template, per workflow, per schedule. Cheap to configure and disproportionately useful for the "who knew this ran at 03:00" problem.

**Does not:** constitute an audit record — notifications are best-effort external messages, not a tamper-evident log of record.

### 3.10 Job history and the audit trail

**Does:** retains job records with **who, what template, which commit, which inventory, which credential, which host, which result, start/finish times and stdout** — queryable and exportable, and in the platform's own words designed for "**auditability and traceability**" at scale **(verified** — the AWX/Tower architect's blog: the original Tower goal was "not just manage those servers but **store the results of that automation and provide auditability and traceability**"; ansible.com, read 2026-09-13).

**Does not:** prove *authorisation*. It is a record of execution, not a record of approval. It does not tell an auditor that a *change record* existed, that the right person approved it, or that the target host was in scope of the approved change window. Mapping job history onto change management is an integration you must build — and §7.4 sets out exactly what the evidence looks like when you have.

### 3.11 Execution environments (EEs)

**Does:** containerises the *runtime* of a playbook — `ansible-core` version, Python, collections, dependencies — as a signed, versioned image, so "it worked on my laptop" becomes "it ran in `ee-supported-rhel9`, tag X". Red Hat ships versioned and version-less EE images via `registry.redhat.io`/`catalog.redhat.com`, with the version-less tags acting as rolling tags **(verified** — Red Hat AAP lifecycle page, "Versioned and Version-less Execution Environment Images", plus the `ansible-builder` install doc in the AAP 2.7 index, read 2026-09-13).

**Does not:** pin your *content* — the playbook and collections you inject still come from a project at run time; an EE pins the environment, not the code. And it does not eliminate dependency drift: you must rebuild, sign and re-tag images as upstream collections move.

**Lineage note:** EEs were introduced with **AAP 2.0** — Red Hat's own 2.0 material says the 2.0 release "primarily delivers support for automation **execution environments**… a standardized way to define, build and distribute the environments that the automation runs in" **(verified** — access.redhat.com article *What's new with Ansible Automation Platform 2.0: Updated private automation hub*, via search result, 2026-09-13). The common claim that EEs replaced the old per-organisation Python virtualenvs is **(flagged)** here: the replacement is well described in secondary material, but I did not confirm that specific wording at a primary source in this session.

### 3.12 Automation mesh (control, execution, hop nodes) and receptor

**Does:** distributes execution across nodes with explicit topology — **control**, **execution** and **hop** nodes — so jobs can run *near* the estate (in a DMZ, in a segregated zone, at the edge) instead of backhauling traffic through one datacentre, and so capacity scales by adding execution nodes **(verified** as the platform model — AAP 2.7 docs: "Plan your automation mesh topology by designing node relationships, connection patterns, and network routing to distribute automation workloads across on-premises, cloud, and edge environments"; read 2026-09-13). The underlying transport is **receptor** **(verified** by reference — the AWX 24.6.0 release notes add "a receptor work list command to sosreport", read 2026-09-13).

**Does not:** make a single play finish faster. Mesh spreads jobs; it does not parallelise inside a play beyond the `forks`/`serial` model of the engine. It also **does not remove the requirement for network reachability** — hop nodes route *through* the segmentation model, they do not tunnel around a firewall the security team has not approved. In a regulated estate this is a feature to be designed with the network team, not a bypass.

### 3.13 Private automation hub, collection curation and signing

**Does:** a local content registry inside your perimeter: sync certified/validated content from the vendor, curate and host **your own** collections as namespaces, run an **approval pipeline** before content is published, **sign** collections and container images, and serve execution environments, all without egress to the public internet in an air-gapped estate **(verified** — AAP 2.7 docs: "Sync, manage, and sign automation content collections in private automation hub, including Red Hat Certified, validated, and community content" and "Configure approval pipelines, and manage access to collections in private automation hub"; read 2026-09-13).

**Does not:** review your content *for* you. It provides the mechanism for a **reference/approved-content model** (§7.6) — namespaces, approvals, signatures — but the policy (what may be published, who approves, what gets signed) is yours. It also does not make community content certified: certified, validated and community content retain different support expectations, and Red Hat's lifecycle page is explicit that end-node support follows the **individual collection's** lifecycle, not the platform's **(verified** — AAP lifecycle page, "Ansible Collections" section, read 2026-09-13).

### 3.14 Event-Driven Ansible, the automation portal, and the 2.7 AI layer

**Does:** beyond the classic controller, the current platform ships **Event-Driven Ansible** (rulebook activations that run Ansible in response to events, with decision environments), the **automation portal** for self-service ("enables platform administrators to quickly and easily scale automation service delivery to new users and teams" — a headline feature of 2.7), and an AI layer: the **Ansible Lightspeed Intelligent Assistant**, an **MCP server** for connecting AI agents to AAP, and Code Assistant-flavoured tooling **(all verified** — AAP 2.7 release notes and docs index: automation portal RHEL appliance and OpenShift install paths, Event-Driven Ansible user guide, MCP server deployment guide, "Unlock more value with the new automation dashboard", "Operate more efficiently with the new Ansible Lightspeed Intelligent Assistant"; read 2026-09-13).

**Does not:** make the governance argument for you. The portal expands *who can trigger automation* (a governance change, and the reason a survey design and RBAC model must land before the portal does), and the AI/MCP surfaces are new enough that their enterprise support posture is explicitly narrower — MCP servers ship on a **12-month lifecycle** separate from AAP's, and "**LLM provider integrations and the behaviour of connected AI models are out of scope**" of support **(verified** — Red Hat lifecycle page, "MCP Servers Lifecycle"; read 2026-09-13). On a regulated estate, treat the AI layer as an evaluation track, not a dependency.

### 3.15 What the platform does **not** add, in one list

- **It does not make playbooks more capable.** Same `ansible-core` underneath (AAP 2.7's default EE ships **ansible-core 2.16**; core 2.18 and 2.20 streams are also published) **(verified** — AAP lifecycle page Table 1.2 and Table 1.3, read 2026-09-13).
- **It does not replace source control, code review, or CI.** Projects *sync from* SCM; they do not review it.
- **It does not test your automation.** There is no compliance scanner in the control plane that fails a bad pull request; that belongs to CI and lint/test tooling.
- **It does not change your change-management process.** It generates evidence for one (§7.4).
- **It does not remove operational overhead.** It adds a platform to patch, upgrade, back up, capacity-plan and audit — three to five years of lifecycle effort, per §4.2.
- **It does not make an unsafe credential safe** (§3.4) or an unreviewed playbook governed (§3.5).

---

## 4. Versions, Packaging and Licensing

This is the section most likely to be wrong within a year, so every claim carries its verification date, and the pieces I could **not** confirm are flagged rather than smoothed over.

### 4.1 Current versions (verified 2026-09-13)

| Thing | Current value | Source / date |
| --- | --- | --- |
| **Product** | **Red Hat Ansible Automation Platform 2.7** (docs default to 2.7; release id **2.7.20260603**) | docs.redhat.com AAP 2.7 release notes; 2026-09-13 |
| **Default engine inside AAP** | **ansible-core 2.16** (the platform default for AAP 2.5, 2.6 *and* 2.7, kept deliberately to preserve RHEL 8 control/managed support) | Red Hat AAP lifecycle page, Tables 1.2 and 1.3; 2026-09-13 |
| **Engine streams available in AAP 2.7** | Core **2.18** (stable, all three AAP lines) and Core **2.20** (EE-only stream in 2.7, **not** the platform default) | Red Hat AAP lifecycle page; 2026-09-13 |
| **Automation controller** | **4.8** — required by the Configuration-as-Code collection constraint `ansible.controller >= 4.8` for AAP 2.7 | Red Hat AAP lifecycle page, Configuration as Code table; 2026-09-13 |
| **Platform gateway** | date-stamped version **2.7.YYYYMMDD** pattern (`ansible.platform >= 2.7, < 2.7.YYYYMMDD`) | Red Hat AAP lifecycle page; 2026-09-13 |
| **Upstream engine** | **ansible-core 2.21.4** (released 2026-09-08); 2.20.9, 2.19.13, 2.18.19 also current on their branches | github.com/ansible/ansible releases, PyPI; 2026-09-13 |
| **Community package** | **ansible 14.4.0** (released 2026-09-08), built on **ansible-core 2.21.0**, Python `>=3.12` | PyPI; `ansible-build-data` `14/ansible-14.build`; 2026-09-13 |
| **Upstream controller (AWX)** | **24.6.1** — and the README's caution says the last release was **2 July 2024** with **releases paused** during a "large scale refactoring" | github.com/ansible/awx README + releases; 2026-09-13 |

**Two honest annotations.**

- **(flagged)** The AAP 2.7 release-notes page renders its "platform services in 2.7" table inside a collapsed widget. The sequence of values I could read was release id **2.7.20260603**, gateway/controller versioning consistent with **4.8**, **ansible-core 2.16**, and UI **2.7.0**; I also saw values that appear to be **automation hub** and **Event-Driven Ansible controller** versions but could not reliably bind them to their columns through the collapse. **The two that matter commercially — controller 4.8 and core 2.16 — are corroborated independently by the lifecycle page, so I treat them as verified; the hub/EDA component numbers are flagged for re-checks.** The durable point stands either way: *component versions move independently of the AAP version* (Red Hat states this explicitly in lifecycle Table 1.2, footnote 2).
- **(flagged)** The community docs' own release table contradicts PyPI about which community-package major is current (14 vs 13). See §2.4.

### 4.2 Supported versions and the deployment matrix (verified 2026-09-13)

| AAP version | Default ansible-core | Required OCP (Operator) | RHEL-provided PostgreSQL | RPM install OS | Containerized OS |
| --- | --- | --- | --- | --- | --- |
| **2.7** | 2.16 | 4.14–4.22 **(TBC)** | 15–17 **(TBC)** | **Not Available** | RHEL 9, RHEL 10 |
| 2.6 | 2.16 | 4.14–4.22 | 15–17 | RHEL 9 | RHEL 9, RHEL 10 |
| 2.5 | 2.16 | 4.12–4.20 | 15 | RHEL 8, RHEL 9 | RHEL 9, RHEL 10 |

**(verified** — Red Hat AAP lifecycle page, Table 1.2, read 2026-09-13. The **"TBC"** markers are Red Hat's own, published as-is; treat them as genuine vendor uncertainty, not as my hedging. The **RPM installer's disappearance in 2.7** is the biggest packaging change in this table.)

**What this means in practice:**

- **RPM-based installs are over.** A 2.5/2.6 estate on RPM must **migrate to containerized (or Operator) before upgrading to 2.7** — the AAP 2.7 docs include a "Migrate from existing deployment topologies" path and an upgrade-path decision page precisely for this **(verified** — AAP 2.7 docs index, read 2026-09-13).
- **Deployment models are now two:** *containerized on RHEL* (Podman) and *Operator on OpenShift*. Red Hat tests only specific **"growth"** and **"enterprise"** topologies and states plainly: "Red Hat does not fully test topologies outside of published reference architectures. Red Hat recommends using a tested topology for all new deployments and provides commercially reasonable support for deployments that meet minimum requirements" **(verified** — AAP 2.7 *Choose a deployment method and topology*, read 2026-09-13). Translation for a regulated estate: your bespoke topology may be supportable but not *tested*; adopt a reference architecture.
- **Cloud options (verified 2026-09-13, redhat.com pricing page):** a **managed service** (Red Hat manages platform *and* infrastructure, on your AWS cloud, billed by AWS, **24×7 Premium SLA support included**); a **managed application** (Red Hat manages the platform, infrastructure stays in your Azure cloud, billed by Microsoft); and **self-managed marketplace deployments** on **AWS, Microsoft Azure and Google Cloud**. Red Hat's own pricing page lists all of these as current. **(flagged)** I did **not** independently open the AWS/Azure/GCP marketplace listing pages this session, so the marketplace *listing states, SKU names and prices* are unverified — see §11.
- **Legacy-page warning (flagged):** the same pricing page still describes a self-managed option as "Red Hat Enterprise Linux **8.3+**", which cannot be true for AAP 2.7 given the lifecycle table (containerized OS: RHEL 9/10). Marketing pages lag lifecycle pages. **Lifecycle page wins.**

### 4.3 Subscriptions: what is published, and what is not

**Published (verified 2026-09-13, redhat.com *Ansible Automation Platform pricing and deployment options*):**

| Tier | Published difference |
| --- | --- |
| **Standard** | "9 AM to 5 PM support"; maintenance and upgrades; full access to the platform's built-in resources |
| **Premium** | "**24x7 support**"; maintenance and upgrades; same platform access |

And the sentence that matters more than the tiers: **"Pricing varies based on your sizing and subscription choices. Talk to a Red Hatter or one of our authorized partners for a customized quote."** Red Hat **does not publish a list price** for AAP on this page. **(verified** — read 2026-09-13)

Consequences, stated plainly:

- **Never quote an AAP price from memory or from a blog.** This guide quotes **none**, because the vendor publishes none; anything you have seen elsewhere is a negotiated figure, a regional partner figure, or stale.
- **The commercial unit is the managed node / sizing, not the install.** That is what "sizing" and the node-counting machinery imply, and it is why the *same* platform can cost a rounding error or a programme budget depending on estate size. **(flagged)** — I could **not** verify the exact counting rule (what counts as a managed node, when it is counted, whether unique-per-month or peak, and how host metrics feed the count) at a primary source in this session. The verifiable adjacent facts are: organisations carry an explicit **"Max hosts"** cap (default `0` = no limit) **(verified** — AAP 2.7 *Get started as a platform administrator*); the platform collects usage data through a **metrics service** that feeds the automation dashboard **(verified** — AAP 2.7 install docs); and AWX's changelog references **host metric** gathering in batches **(verified** — AWX 24.6.0 release notes). **Any hardware-sizing or licensing-cost exercise must therefore stop and confirm counting rules with Red Hat/your partner before it is presented to a budget holder.**
- **A "self-support level license" exists** and is materially more restricted than Standard: AAP 2.7's administrator guide states, "**If you have a self-support level license, you have only the default organization available and cannot delete it**" **(verified** — 2026-09-13). **(flagged)** The node limit, feature set and support terms of that level are **not** stated on the pages I read; do not repeat the widely-circulated "N free managed nodes" figure without checking it in the subscription itself.

### 4.4 Free means free — but know which free

| Option | Cost | Licence | Support | Runtime status |
| --- | --- | --- | --- | --- |
| **ansible-core** (the engine) | None | **GPL-3.0-or-later** **(verified**, PyPI 2026-09-13) | Community forum only | Actively released (2.21.4, 2026-09-08) |
| **ansible** community package | None | **GPL-3.0-or-later** **(verified**, PyPI) | Community | Actively released (14.4.0) |
| **AWX** (upstream controller) | None | **Apache-2.0** **(verified**, README badge) | Community | **Releases paused since 2 July 2024** pending refactor **(verified)** |
| **AAP trial** | None for the trial window | Red Hat Enterprise Agreement + Product Appendix | Trial-grade | Full platform, **evaluation only**: the trial terms state that using it "for a production installation", offering support to third parties, or supplementing third-party support "would incur additional fees and be in violation of the Agreement" **(verified** — redhat.com product-trial terms, read 2026-09-13) |
| **AAP subscription** | Negotiated | Red Hat subscription terms; EULA accepted in-product at subscription activation **(verified** — AAP 2.7 admin docs) | Standard or Premium | Full platform |

### 4.5 The licensing and packaging changes of recent years — verified, and one thing I could not verify

**(verified, all 2026-09-13):**

- **No open-source licence change to Ansible.** In the same post that announced the AWX refactor, Red Hat's Ansible chief architect wrote: "Before we conclude, we should be clear about what will not happen. **We are not changing the Ansible project. We are not adjusting our OSS license structure.**" (ansible.com blog.)
- **AWX is not dead but is frozen.** The refactor communications centre on a move to a "pluggable, service-oriented architecture", with consequences for UIs, credential types and Operator installation methods (README links to the forum series). Practical effect for 2026: **AWX 24.6.1 (July 2024) is what you get**, while the paid platform has moved on through 2.5 → 2.6 → 2.7. The upstream/product gap is the widest it has been in the relationship's history. That is an architectural fact you must price into any "we'll just run AWX" plan.
- **Packaging shifted under the platform's feet in 2.7:** RPM install gone; **all authentication funnels through the platform gateway**; direct component-level token support removed; the backward-compatibility API layer removed **(verified** — AAP 2.7 upgrade docs: authentication changes, API changes).
- **Support-scope narrowing at the edges:** MCP servers ship on a **12-month** lifecycle separate from AAP, and "**LLM provider integrations and the behaviour of connected AI models are out of scope**"; the HashiCorp Vault OIDC capability is a **technology preview**, not a supported commitment **(verified**, AAP 2.7 docs and lifecycle page).
- **Content support is per collection, not per platform:** end-node automation support follows the **individual collection's** lifecycle (e.g. OpenShift, Azure, Cisco ACI, NetApp collections each carry their own) **(verified** — lifecycle page, "Ansible Collections").

**(flagged — explicit non-answer):** I **could not verify** any specific **2025–2026 subscription-restructuring, price-book change, or automation-hub entitlement change** from a primary source in this session. Searches for such announcements returned nothing usable, and the pages I did reach publish tiers without prices, node formulas or entitlement tables. **Treat any claim of a recent "licensing overhaul" as unverified until you have a Red Hat quote or subscription document in hand.** What I *can* support as the direction of travel: the commercial model has migrated from a named product you buy (Tower) to a **subscription measured in estate size (AAP)**, with capability bundled into the platform rather than sold as separate SKUs, and with content delivery (automation hub, private automation hub, signed collections/EEs) inside the subscription perimeter.

- **(verified 2026-09-13 — entitlement *provisioning*, not pricing):** the **attachment path** for an AAP subscription did change at **AAP 2.5**. Red Hat's customer-portal article *Attaching your Ansible Automation Platform subscription to your instance* ([access.redhat.com/articles/5807761](https://access.redhat.com/articles/5807761)) states that "**Beginning with AAP 2.5, there are 4 ways to attach your subscription to your AAP instance**", listing options that include obtaining and attaching a **manifest file** and locating/attaching the subscription by entering **Red Hat Hybrid Cloud Console service-account credentials**. The move is from simple `subscription-manager`-style attachment to a **Hybrid Cloud Console service-account / manifest** model. This is an **entitlement-provisioning** change only — it is **not** evidence of a price-book change or of changed licence terms, and the caution above stands.

---

## 5. The Comparison Table

The single table both §6 and §9 lean on. "Engine-only" means `ansible-core` (optionally the community package) driven from a laptop, cron or CI.

| Dimension | Engine-only (ansible-core CLI) | Control plane (AWX / AAP) |
| --- | --- | --- |
| **Capability to automate** | Full — the engine *is* the capability | Identical — same `ansible-core` underneath (AAP 2.7 default core 2.16) **(verified)** |
| **RBAC / authorisation** | SSH keys + repo rights. No notion of "may run this against that" | Organisations, teams, users, roles; external IdP; auditor-style read roles; in 2.7 governed centrally through the platform gateway **(verified)** |
| **Credential custody** | Secrets on the operator's machine or in the pipeline; a vault password is a shared secret | Platform stores credentials and injects them per run; operators never hold the key; can externalise to vault/secrets manager; 2.7 previews short-lived job-scoped OIDC tokens to HashiCorp Vault **(verified, preview flagged)** |
| **Scheduling** | Someone's crontab, or CI cron | Schedules as first-class objects, per template, with ownership **(verified existence)** |
| **Auditability** | Logs you chose to keep; no identity attribution unless CI supplies it | Job history: who/what/commit/inventory/credential/hosts/result/stdout; the stated design goal of Tower from 2013 was "store the results … provide auditability and traceability" **(verified)** |
| **Scalability model** | One control node; `forks`/`serial`; scaling = a bigger box or more boxes each running their own copy | Automation mesh across control/execution/hop nodes, instance and container groups; vendor claim: "designed for deployments across **thousands of nodes**, with built-in clustering and capacity management" **(verified** as a vendor claim; the mesh model itself is **(verified)**) |
| **Multi-tenancy** | None. Everyone with keys shares one trust boundary | Organisations = tenancy boundary, with per-org host caps **(verified)** |
| **Content governance** | Repo permissions; nothing verifies or signs a role | Private automation hub: namespaces, approval pipelines, signing of collections and EE images, certified/validated/community separation **(verified)** |
| **Deployment complexity** | `pip install ansible-core` | Containerized or Operator deployment on tested topologies; PostgreSQL; OpenShift option; backup/restore, upgrades, migrations **(verified)** |
| **Operational overhead** | None beyond the engine and the hosts | A platform to patch, back up, capacity-plan, upgrade every few months, and later migrate (RPM→container was mandatory for 2.7) **(verified)** |
| **Cost model** | Zero licence; real cost is engineer time and risk | Subscription priced by sizing, **no published list price**; Standard (9–5) vs Premium (24×7) support **(verified** that these are the published tiers; price **(flagged)**) |
| **Skills / hiring** | Everyone who writes YAML; hiring signal is `ansible` on a CV | Adds platform-admin and RBAC-design skills; hiring signal is *Tower/AAP administrator* plus automation-architecture experience — a scarcer and more expensive profile **(flagged** — market characterisation, not a vendor fact) |
| **Break-glass difficulty** | Trivial (anyone with keys) — which is the problem | Requires deliberate design: emergency credentials, audited override, post-hoc review (§7.5) |

**The one-line reading:** the engine column wins on *time-to-first-automation* and *cost*; the control plane column wins on **custody, attribution and delegation** — every single advantage of the control plane reduces to one of those three words.

---

## 6. The Decision Framework

### 6.1 When the CLI alone is correct (the honest case for not adopting)

Adopt the platform when its advantages are *binding constraints*. Until then, they are overhead with a support contract. The CLI alone is the right answer when **most** of the following hold:

- **One or two operators, one estate, shared fate.** If the same two people already hold every key and already trust each other, a platform adds RBAC for a population of two.
- **The compliance burden is light.** You can answer "who changed production and when" from CI logs, Git history and change tickets without an auditor asking you to produce an execution record per change.
- **No delegated self-service is needed.** Nobody outside the engineering team wants to trigger automation. The moment someone does, the cheapest safe answer is a *survey-equivalent* (a parameterised CI job with approval), not a platform — but §6.2 trigger 4 has arrived.
- **Scale is genuinely modest.** One control node with sane `forks`/`serial` comfortably drives fleets in the hundreds; the platform's mesh is built for "thousands of nodes" per the vendor's own framing, and you do not need a transport topology to reach a few hundred hosts. (Fleet-sizing itself is **(flagged)** — verify with a real run rather than a vendor diagram.)
- **The environment is short-lived or disposable.** Labs, ephemeral sandboxes and migration projects rarely justify a platform whose upgrade train runs every few months.
- **The alternative control plane already exists and is paid for.** See §6.3 and §9: in 2026 the strongest CLI pattern is **CI as the control plane**, holding the credentials, the schedule, the approval gate and the log.
- **Team maturity is the real bottleneck.** A platform does not fix an estate with no inventory discipline, no role standards and no tests; it merely gives the chaos an audit trail.

### 6.2 The trigger conditions that genuinely justify the platform

Cross the line when **any** of these becomes true — not when the dashboard becomes attractive:

1. **Multiple operators with different entitlements.** People who must automate but must not hold root SSH keys. This is the primary trigger and, in a bank, it usually arrives with the first audit.
2. **Segregation of duties.** The person who writes the change must not be the person who executes it unchecked; the person who executes must not be able to edit the payload unnoticed. RBAC + approval pipelines + signed content is the mechanism (§7.1).
3. **The shared-credential problem.** Two or more workflows (or two or more teams) need the same privileged credential, and no one can cleanly answer "who used it, and when". Central custody is the answer; §7.2 quantifies what it does and does not fix.
4. **Delegation to non-specialists.** Someone non-specialist (service desk, NOC, a business-unit operator) must safely trigger automation. Credential custody plus a locked-down job template plus a survey is the only pattern that scales.
5. **The audit-evidence requirement.** An auditor/regulator/insurer explicitly wants per-execution evidence: actor, artefact, target set, result. (Note the trap: if the requirement is textual — "we need a dashboard" — you have an information problem, not a platform problem.)
6. **Fleet scale beyond a single control node's practical competence**, or execution that must happen *near* segmented/remote estate rather than through one egress point.
7. **Content governance at team scale.** Multiple teams publishing automation, with a need for approval, signing and a curated internal registry (private automation hub) rather than "whoever merged last wins".

### 6.3 Anti-patterns, in both directions

**Anti-pattern A — the platform adopted for the dashboard.** The stated driver is visibility ("we want to see what's happening"), so the platform is installed, job templates are created for a few playbooks, and the hard parts — RBAC design, credential custody, SCM discipline, signing, a CI path, break-glass design — are never done. Result: **a very expensive cron with a nice UI**, plus a new platform to patch. The tell: after go-live, most runs still happen from laptops and CI, because the platform offers no reason to move them; the UI shows a fraction of the estate's real change activity. Fix: **write the RBAC and custody model first, and make the platform the only path to production**, or do not buy it.

**Anti-pattern B — the fleet driven by an engineer's laptop.** No platform, no pipeline: the production run depends on one person's machine holding the keys, their `~/.ssh/config`, their un-committed local edits, and their memory. The estate is *entirely* governed by a person. This is not "lean"; it is **uncustodied control**, and it fails every one of: bus factor, audit, and disaster recovery — the moment that person is unavailable, so is the estate. The tell: the deployment runbook contains the phrase "ask *X*, they have the keys".

**Anti-pattern C — "we'll just run AWX" as a free AAP.** Adopting the upstream controller is a legitimate choice (§8.1) — *if* you accept that you are now the maintainer of an unpinned, frozen upstream and that **no support** covers your automation estate. What is not legitimate is adopting AWX because it looks free while assuming Red Hat-grade support, upgradeability and content curation arrive with it.

**Anti-pattern D — the platform as an org chart workaround.** Deploying AAP to *compensate* for a team that will not adopt review, tests, or inventory discipline. The platform faithfully records the resulting mess with better timestamps.

### 6.4 A one-page decision procedure

```
Do I need to automate?                      -> yes, obviously. Use ansible-core. Do not buy anything yet.
Do 2+ people need to automate the SAME estate with DIFFERENT entitlements?
      -> no  : CLI + CI control plane is likely sufficient. Revisit annually. STOP.
      -> yes : continue.
Must a non-specialist trigger automation, or must execution be attributed to an individual?
      -> no  : CI control plane probably suffices (it captures actor per pipeline run). STOP, document why.
      -> yes : continue.
Is there a binding audit-evidence requirement per execution (not just "a dashboard")?
      -> no  : continue anyway if scale/segmentation demands it; otherwise STOP.
      -> yes : cross the line.  Go to the adoption checklist below.
```

**Crossing-the-line checklist (the things people forget):** (1) identity provider decided and roles written as code; (2) credential custody model with rotation and scope minimisation; (3) inventory of record with ownership, not a text file; (4) SCM discipline: branch protection, CODEOWNERS, tags/releases as the unit of promotion; (5) CI that lints/tests before content reaches the controller; (6) survey and template design so delegated runs cannot be parameterised into danger; (7) break-glass path with logging (§7.5); (8) backup/restore and an upgrade cadence you have actually tested, including the migration off RPM if you are on 2.5/2.6; (9) an evidence-export routine (§7.4); (10) someone accountable for the platform as a product — not a side project.

---

## 7. The Regulated-Enterprise and Banking Angle

Everything above becomes sharper under supervision, because the control plane's three wins — **custody, attribution, delegation** — are the exact three things a supervisor asks about. This section is the banking translation; for the operational-resilience frame it sits inside (essential services, impact tolerances, severe-but-plausible scenarios), see the repo's anchor: [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md).

### 7.1 RBAC mapped to segregation of duties

The mapping that survives audit is the one where **the entitlement lives in the platform, not in a person's keychain**:

| SoD role | Platform expression | What "good" looks like |
| --- | --- | --- |
| Author (writes automation) | Write access to the SCM project; **no** execute rights on production templates | Author cannot push their own change to production directly; their merge is the artefact |
| Approver / change authority | Approve the change record; approve content in the private-hub pipeline; no execute | Approval is a separate identity from both author and executor |
| Executor (runs the change) | Execute on a **named job template**, bound to a credential the executor cannot read | Executor runs, cannot edit the payload, cannot see the secret |
| Operator / service desk | Execute on *survey-limited* templates only (restart-this-service class) | Delegated safely; no free-form playbook choice |
| Read-only auditor | Audience/audit roles, ideally synced from the IdP, with read-only visibility | Auditor can *see* without being able to *do* |
| Platform administrator | Full administrative control | Recognised as a **privileged** population: segregated accounts, MFA, monitored, and included in emergency-change procedures |

Two design notes that decide whether this holds up. First, **RBAC is a mechanism — the model is your artefact**: encode users, teams, organisations and role assignments as code using the Configuration-as-Code collections (`ansible.controller`, `ansible.platform`, `ansible.hub` at the versions matching your AAP release **(verified** — Red Hat lifecycle page, read 2026-09-13) and review changes to *that* repo like any other production change. Second, **mind propagation lag**: AAP 2.7 docs state that managing users/teams via the legacy controller API can take "**up to 15 minutes** to propagate to Event-Driven Ansible controller, which can result in authentication errors" and that the Unified UI or platform gateway API should be used for real-time sync **(verified** — AAP 2.7 admin docs, read 2026-09-13). A joiner/mover/leaver control that assumes instant propagation is a control with an exploitable window at exactly the wrong moment.

### 7.2 The credential-custody and secrets question

The question is not "does it encrypt secrets" (so does a vault file) but: **who can extract a usable credential, and how would you know?**

| Question | CLI estate | AAP estate |
| --- | --- | --- |
| Who holds the production credential? | Every operator with the vault password or key | The platform; injected per run |
| Can an operator read it? | Yes | No (by design), including for delegated runs |
| Rotation | A project (find everywhere it is used) | Central object; still a project, but at least enumerable |
| Blast radius of one platform compromise | N/A (no platform) | **High** — the platform becomes a credential castle; this risk must be accepted explicitly, with hardening, HA, backup protection and monitoring |
| Short-lived credentials | Manual (if at all) | 2.7 previews OIDC job-scoped tokens for HashiCorp Vault **(verified, technology preview)**; integration with an enterprise secrets manager remains the durable pattern |
| Does the target still authenticate the *job*? | Yes — SSH key, service account, etc. | Unchanged: the platform brokers custody, it does not remove the target's own auth |

**The honest counterpoint to the platform's custody win:** the platform concentrates credentials. In a regulated estate that concentration must be treated as a *new critical system* — in scope for resilience, DR testing, change control, privileged-access review and vulnerability management (see the operational-resilience anchor above). The correct comparison is not "custody vs no custody" but "**distributed undocumented custody vs concentrated documented custody**"; the second is auditable, the first merely familiar.

### 7.3 What the audit trail is — and what it is not

**It is:** an execution record — actor, template, project/commit, inventory, credential used, host set, outcome, timestamps, output **(verified** — job history is a core platform capability; the stated design intent from 2013 was auditability and traceability).

**It is not, on its own:**

- **Proof of authorisation.** Nothing in the job record says a change was approved, by whom, for that window, on that host set.
- **Immutable by default.** Job records are data in a database you administer; an estate with database write access has a record it can alter. **Unless you export/ship records to an append-only or separately-administered store, "tamper-evident audit trail" is a claim you cannot make.** **(flagged** — architectural reasoning, not a quoted vendor claim; validate against your own platform configuration and any export/WORM arrangements.)
- **Complete.** Automation bypasses the platform whenever someone uses the engine directly from CI or a laptop. **If the same estate can be changed by two control paths, the platform's audit trail covers one of them.** Closing the second path is a control decision (network policy, key custody, pipeline policy), not a product feature.
- **Self-sufficient for the change record.** The platform knows *what ran*; the ITSM tool knows *what was approved*. Only the join tells the story (§7.4).

### 7.4 ITSM/CAB integration and the evidence pack

**The join you must build:** the change ticket ID must appear in the automation run and the run ID must appear in the change record. Two mechanisms, in order of robustness:

1. **Pass it through**: the pipeline/job template receives the ticket ID as a survey/pipeline parameter, echoes it into job metadata (and ideally into a notification/webhook to the ITSM record), so job history is queryable by ticket.
2. **Query it out**: an evidence-export routine that pulls job records per window and joins them to the change calendar — ideally run by a party that is not the automation team.

**The evidence pack an auditor actually asks for** (assembly is a *process*, built once and rehearsed, not a scramble):

| Evidence item | Where it comes from | Common gap |
| --- | --- | --- |
| The approved change (what, why, window, approver, risk) | ITSM/CAB record | Ticket references a system, not the automation artefact |
| The automation artefact, versioned | SCM commit/tag, project sync record | "Latest" branch without a pinned tag at execution time |
| Who executed, when, against what | Platform job record (or pipeline run, if CI is the control plane) | CI logs expire before the retention period the regulator assumes |
| What actually changed on the target | Job output + per-host changed/OK counts; ideally host-side config or database evidence for critical objects | Trusting "PLAY RECAP ok" as proof of business state |
| Segregation of duties held | RBAC export + approver identity ≠ executor identity | The approver is also the platform admin and the executor |
| Emergency changes, retro-authorised | Break-glass log + post-hoc change record (§7.5) | Break-glass leaves no trace |
| Secrets were not exposed | `no_log`-based hygiene, vault scope, credential-type configuration | Secrets printed in job output and shipped to the same log store |
| Content provenance | Signed collections/EEs, approval pipeline records | Community roles pulled from a public galaxy at run time |

### 7.5 Break-glass and emergency change

Every regulated estate has a "the platform is down and production must be fixed at 03:00" path, and the honest design is to build it deliberately:

- **A documented emergency path exists** — offline break-glass credentials held by two named people under dual control (safe/escrow, not a shared vault file), with a defined time-to-revoke.
- **It is logged outside the platform** — because in a platform outage the platform's own log is the thing that is down. Emergency execution evidence has to land somewhere the platform's availability does not dictate.
- **It triggers retro-authorisation** — a change record raised *after* the fact, referencing the emergency, within a stated window, reviewed by someone independent.
- **It is tested** — an untested break-glass path is a document, not a control. The regulator-visible question is "when did you last exercise it, and what did you find?"
- **It is scoped to a set of hosts and time** — not "root, everywhere, forever".

### 7.6 Regulated-estate constraints: segmentation and the reference-content model

- **Network segmentation is designed *with* the platform, not around it.** The mesh (control/execution/hop nodes) exists precisely so execution can occur inside zones rather than through a single egress **(verified** as the model). But a mesh topology must still satisfy the network security model: which port, from which zone, to which zone, approved by whom. **A hop node is a network path — get it into the firewall change process.** In practice, most regulated estates end up with one execution node per zone and a control node that never crosses the boundary, plus a documented exception for the mesh ports.
- **Air-gapped and low-egress estates:** private automation hub plus **disconnected installation** guidance exists in the AAP 2.7 documentation **(verified** — the containerized install doc lists disconnected installs, and the hub docs cover syncing and signing content locally). This is the mechanism for a **reference/approved-content model**: sync certified content in, publish internal namespaces with an approval pipeline, sign what you publish, and require execution environments built from an internal image registry. The *policy* — what may be published, who approves, what is forbidden — is yours; the platform supplies the controls (namespaces, approvals, signatures).
- **Data residency and model risk:** with AI surfaces in the 2.7 release, remember the **explicit exclusions**: MCP servers carry a 12-month lifecycle and "LLM provider integrations and the behaviour of connected AI models are out of scope" of support **(verified)**. For a bank, that is a "evaluation track, no regulated dependency" classification until the vendor's support posture changes.
- **Resilience classification:** the platform is a **critical** system for the automation estate — its outage means *no change can be made*, including emergency changes, which is why §7.5's out-of-band path and the operational-resilience anchors ([../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md)) both matter more than any dashboard.

---

## 8. The Alternatives Landscape

The question "Tower or CLI?" is really "**which control plane, if any?**" — and there are at least five answers. This section states what each one is for; neighbouring subjects are cross-referenced rather than re-derived.

### 8.1 AWX: the free upstream and its honest trade-off

**What it is.** Apache-2.0 upstream of the controller, with the same conceptual object model (projects, inventories, job templates, credentials, RBAC) and effectively the same API shape — it is, after all, the project the product is built downstream from **(verified** — AWX README, 2026-09-13).

**The trade-off, in 2026 terms:**

| You gain | You accept |
| --- | --- |
| Zero licence cost for a genuine control plane with RBAC, credentials, scheduling and job history | **No vendor support**: your automation estate's availability is your problem |
| The same mental model — migration to AAP later is a skill/knowledge port, not a restart | **Frozen upstream**: last release **24.6.1, July 2024**; "releases of this project are now paused during a large scale refactoring" **(verified)** |
| Community velocity and inspectable code | A known refactor in flight covering **UI, credential types and Operator installation methods**, i.e. the areas you will be asked about in an audit **(verified)** |
| No subscription procurement | You carry the security-patch treadmill yourself, with no vendor remediation timeframe |

**The judgement:** AWX is a defensible choice for a lab, a small estate, or a team that genuinely wants to be the maintainer. For a regulated institution it is a **support and evidence decision, not a cost decision** — and §7 is written for the world where supportability has to be named in a register.

### 8.2 Semaphore

**What it is:** an actively developed open-source job-running UI (Go-based, single service plus database) in the "light control plane" class; it appears on the shortlist whenever a team wants job history, schedules and a web trigger without a subscription. **(verified** — `semaphoreui/semaphore` release page: latest **v2.19.12**, released **30 August 2026**, with **v2.20.0-alpha1** pre-released 21 August 2026; read 2026-09-13.)

**What it is not:** there is no vendor with an enterprise support contract behind it in the way Red Hat backs AAP, and its governance model (RBAC granularity, credential custody, approval pipelines, content signing) is a fraction of AAP's — assume you will be building governance in the surrounding SCM/CI/network layer instead of configuring it.
**(flagged)** I verified *release activity*, not feature parity. Before recommending it, test the specific three capabilities §6.2 asks for — per-object RBAC, whether operators can extract stored credentials, and whether job history is exportable — against your fork of the product at that time.

### 8.3 Rundeck / Runbook Automation

**What it is:** the long-standing open-source runbook-automation console, commercially backed by PagerDuty (open source "Rundeck" + commercial "Runbook Automation"). Its centre of gravity is **operational self-service and job scheduling** across many node types, with Ansible as one execution engine among several — rather than Ansible-native orchestration.

**Status:** actively maintained. **(verified** — `rundeck/rundeck` releases: **v6.2.1 (tag `v6.2.1-20260909`)**, released **9 September 2026**, with a regular cadence through 5.17–6.2 across 2025–2026; PagerDuty's 6.0 announcement and the docs' "recent updates" page both describe ongoing development of the open source project and the commercial Runbook Automation product; read 2026-09-13.)

**When it is the better fit:** heterogeneous estates where a large share of operations is *not* Ansible (Windows, network CLIs, shell, database jobs), and where the organisational need is "a safe self-service console with approvals and audit" more than "an Ansible-native content platform". **When it is the worse fit:** if your automation strategy is Ansible-first, you get an extra, non-Ansible object model in the middle of it.

### 8.4 Jenkins (or any CI system) as the de-facto control plane

**What it is:** using the CI/CD system you already own as the control plane for the engine — Jenkins, GitLab CI, GitHub Actions, Azure DevOps. Done properly, it supplies: identity and RBAC (or at least project-level permissions), a credential store, scheduling (cron/triggers), a log of record per run, approval gates (environments/manual steps), and pipeline-as-code.

**Why this is a real answer, not a workaround:** it is the *only* option that adds governance without adding a platform, because it reuses an existing governed system. It is also the pattern most regulated teams already run somewhere. Design guidance for the pipeline side (promotion, gating, artefact identity, environment control) lives in [iac_best_practices_guide.md](iac_best_practices_guide.md) and [power_platform_cicd_guide.md](power_platform_cicd_guide.md); this guide deliberately does not re-derive it.

**What it will never give you:** per-job RBAC on *automation objects* (your unit of authorisation is a pipeline, not "patch web tier in zone 3"), credential custody scoped to an Ansible credential object, survey-style safe delegation for non-specialists, or a job history shaped like an automation run (`host → task → changed`). You can emulate some of it with pipeline parameters, approval environments and log retention — and many teams do, successfully.
**(flagged)** I did not verify the current release state or maintenance status of any specific Ansible integration plugin for these CI systems in this session. Check the plugin's repository activity and security posture as part of any design; a control plane assembled from third-party plugins inherits their maintenance risk, and a CI master holding production SSH keys is a *high-value* target that must be hardened as one.

### 8.5 Cloud-native / GitOps controllers

Kubernetes-first shops often already have a controller doing continuous reconciliation with RBAC, audit and history — Argo CD/Flux/Kargo-class tooling. Whether that makes a separate Ansible control plane redundant is a genuinely different question with its own answer, developed in **kargo_gitops_guide.md**; the short version is that GitOps reconciles *declarative cluster/IaC state*, while an Ansible control plane runs *imperative change against arbitrary estate* (including things no Kubernetes controller can reach). Cross-read rather than substitute — and note that AAP itself offers an OpenShift Operator deployment if the cluster is where you want the platform to live **(verified)**.

### 8.6 Lighter-weight and non-Ansible automation tooling

Salt, Fabric, Rex, and the single-binary Go/Rust tool class are covered in **lightweight-config-mgmt-analysis.md**. That guide owns the "is Ansible even the right engine?" question; this one owns "if the engine is Ansible, what governs it?" — and the two answers can conflict: if the analysis in that guide concludes a lighter tool fits better, the control-plane question changes shape entirely (and often gets *smaller*, because those tools usually carry less governance to begin with).

### 8.7 "Do we even need a controller?" — what each option actually solves

| Need | CI control plane | AWX | Semaphore | Rundeck | AAP |
| --- | --- | --- | --- | --- | --- |
| Scheduled runs + a log of record | ✅ | ✅ | ✅ | ✅ | ✅ |
| Central credential custody for Ansible jobs | Partial (CI credentials) | ✅ | Partial | ✅ | ✅ |
| Per-object RBAC on automation (org/team/role) | ✗ (pipeline-level) | ✅ | Partial | ✅ (project-level) | ✅ |
| Safe delegation to non-specialists | Partial (approvals) | ✅ | Partial | ✅ | ✅ (surveys + portal) |
| Audit evidence shaped like an automation run | Partial (build logs) | ✅ | Partial | ✅ | ✅ |
| Support contract behind the control plane | ✅ (the CI vendor) | ✗ | ✗ | ✅ (commercial tier) | ✅ |
| Vendor-curated, signed content + private registry | ✗ | ✗ | ✗ | ✗ | ✅ |
| Zero licence cost | ✅ | ✅ | ✅ | ✅ (OSS tier) | ✗ |
| Actively moving upstream | ✅ | ✗ (frozen) | ✅ | ✅ | ✅ |

**Verdict for the honest middle:** if your requirement is *the top row only*, you already own the answer — a CI pipeline. The remaining rows are the actual case for crossing the line, and each one you can decline is money and operational effort you keep.

---

## 9. The Cymbal Bank Worked Example

> **Fiction notice.** **Cymbal Bank is a fictional institution**, used here to make a decision procedure concrete. **Every number in this section is illustrative and fabricated for modelling purposes** — no figure below is a vendor quote, a benchmark, or a real organisation's cost. AAP has **no published list price** (§4.3); any real comparison must substitute vendor-quoted numbers and your own labour rates.

### 9.1 The estate and the decision on the table

**Cymbal Bank** (fictional) runs platform engineering for a retail banking estate. Its Ansible estate has grown from a 60-host pilot into the bank's primary change mechanism, and the question is whether to keep driving `ansible-core` from CI or cross the line into the control plane.

| Estate attribute | Value (illustrative) |
| --- | --- |
| Managed nodes | **1,800** Linux nodes across four network zones (dev, pre-prod, core-banking prod, DMZ/edge) |
| Windows/network devices | 120 Windows servers, 40 network devices (same engine, different transports) |
| Ansible content | 1 Git repository, ~140 roles, 22 playbooks, 1 internally published collection, 2 execution environments built in CI |
| Operators | **11** people: 6 platform engineers (author+execute), 2 change/release approvers, 2 service-desk operators (restart/remediate only), 1 security architect (read-only) |
| Change volume | ~900 automated production change executions per month (mostly drift remediation and patching) |
| Evidence requirement | Per-change: approved change record, artefact version, executor identity, host set, outcome; retained **7 years** |
| Network constraint | No inbound access from corporate/user networks into the core-banking zone; all execution must originate from an approved jump path; a mesh/hop design would need network-change approval per zone |
| Regulatory context | Supervised institution; the automation estate is part of the resilience perimeter (see [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md)) |

### 9.2 Option A — CI as the control plane (status quo, hardened)

`ansible-core` runs in pipelines. The changes proposed are: move all production execution to runners in the core zone, put the production SSH credentials in the CI credential store only, make the pipeline reference a **pinned tag** of the automation repo, require an approval environment for prod jobs, and ship pipeline logs to the bank's log-retention platform for 7 years.

**What this genuinely delivers:** actor attribution at pipeline level, artefact pinning, a run log, schedule (pipeline cron), and reuse of a governed system. **What it does not:** per-object RBAC ("may run *this* template against *these* hosts"), safe survey-style delegation for the service desk, credential custody scoped to an automation object, or an evidence shape an auditor recognises as "an automation execution record" without a mapping document.

### 9.3 Option B — Adopt AAP (self-managed, containerized or operator)

Deploy AAP 2.7 in the core zone (growth topology to start, enterprise topology for prod), with:
- **RBAC model:** organisations = dev / pre-prod / prod / DMZ; teams = platform-eng, approvers, service-desk, security; external IdP groups mapped to teams; users/teams/roles defined **as code** in a governed repository via the Configuration-as-Code collections (***illustrative design***, based on the verified fact that these collections exist and are version-constrained per AAP release).
- **Credential custody:** production SSH credentials created as platform credential objects, attachable only to production job templates; service-desk templates bound to a **narrow** credential (service restart only); vault passwords held by the platform; long-term, secrets-manager integration for privileged accounts (and note the 2.7 **technology-preview** OIDC capability to HashiCorp Vault as an evaluation item, not a dependency) **(verified** that the preview exists).
- **Evidence:** job history export routine joining platform job records to change records by ticket ID, deposited in the retention platform; RBAC export per quarter as the SoD evidence.
- **Content governance:** private automation hub with an internal namespace, an approval pipeline, signed collections and signed EEs built in CI; disconnected/curated sync of certified content **(verified** that these hub capabilities exist).
- **Segmentation:** one execution node per zone where execution must originate locally, control node in the management zone, hop node only where the network security model explicitly approves it; each hop/gap is a change record.

### 9.4 Illustrative effort and cost comparison (FABRICATED NUMBERS)

Both options assume the same 11 operators, the same content repository and the same change volume. Labour is modelled at an illustrative fully-loaded **$150,000 per engineer-year**, rounded. **These numbers are invented for structure, not sourced.**

| Line item (illustrative, annualised, USD) | Option A: CI control plane | Option B: AAP self-managed |
| --- | --- | --- |
| Licence / subscription | **$0** | **Subscription: vendor-quoted; modelled placeholder $150,000** (must be replaced with a real quote; **the vendor publishes no price**) |
| One-off build effort | 25 person-days (pipeline hardening, runner placement in core zone, log shipping) ≈ **$15,000** | 45 person-days (install, RBAC-as-code, EE builds, hub, templates, evidence export) ≈ **$27,000** |
| Recurring platform ownership | 0.4 FTE ≈ **$60,000** | 0.9 FTE ≈ **$135,000** |
| Audit-evidence assembly per cycle | 12 person-days ≈ **$7,200** (manual joins) | 4 person-days ≈ **$2,400** (export routine) |
| Secret management engineering | 10 person-days ≈ **$6,000** | 8 person-days ≈ **$4,800** |
| Content governance (review, signing, curation) | Not implemented in this model | 15 person-days ≈ **$9,000** |
| **Illustrative year-1 total** | **≈ $88,200** | **≈ $328,200** |
| **Illustrative year-3 cumulative (build once, 3× recurring)** | **≈ $232,400** | **≈ $931,600** |

**The only defensible conclusion from fabricated numbers:** the delta is real and large, and it buys exactly the three things from §5 — **custody, attribution, delegation** — plus vendor support and curated/signed content. If Cymbal Bank's auditors are already satisfied by CI-log-derived evidence, Option A is not "immature"; it is the correct economic answer. If the audit or the SoD model requires per-object authorisation and platform-scoped custody, Option A's delta becomes an unfunded control gap, and the platform's cost is the price of the control — not a technology preference.

### 9.5 Decision criteria (the test Cymbal Bank actually applies)

| Criterion | Passes on Option A alone? | Notes |
| --- | --- | --- |
| Can a service-desk operator restart a production service **without** holding a production credential? | **No** | The platform's delegated-template model is the only mechanism that satisfies this cleanly |
| Can we prove, per execution, **which human** triggered it and **which commit** ran? | Partially | CI gives actor+commit per pipeline; Ansible-level detail needs mapping |
| Can we enforce that a change **cannot** be executed outside an approved window? | **No** (without new pipeline controls) | Platform schedules + RBAC help, but window enforcement still comes from the change process and the pipeline |
| Is content provenance (signed collection/EE) demonstrable? | **No** | Private automation hub + signing |
| Does the evidence survive an audit without a bespoke mapping document? | Weakly | This is a documentation cost, not a technology gap — and it recurs every audit |
| Is the platform itself supportable and classifiable as a critical system? | N/A | Requires HA, DR, patching cadence, and a named owner |
| Do we have the people to own a platform? | N/A | **0.9 FTE illustrative** is the number most often omitted from the business case |

### 9.6 What remains uncovered **either way** (the honest list)

Neither option below fixes these; they are organisational, and pretending otherwise is how automation programmes fail audits:

1. **Authorisation.** Neither produces an approval. Both produce execution records that must be *joined* to a change record you already have (or don't).
2. **The second control path.** If engineers can still run the engine from a laptop or an unapproved pipeline against production, the platform's audit trail covers the approved path only. **Close the bypass or the evidence is fiction.**
3. **Evidence immutability.** Job records live in a database you administer; retention, tamper-evidence and long-term storage beyond the platform's lifetime (7 years vs a 3–5 year platform lifecycle) must be engineered outside the platform.
4. **Content quality.** Neither reviews a playbook. A bad role runs faithfully, with excellent provenance.
5. **Secret hygiene inside the run.** Custody ≠ safe use: `no_log` discipline, output handling and log shipping must be designed, or the platform's job output ships secrets to your log platform at scale.
6. **The human process.** Emergency changes, dual control, approver independence, and the review that actually reads the output.
7. **Network reality.** A mesh design still has to clear segmentation change control (§9.3); no product feature waives a firewall review.
8. **Platform lifecycle shocks.** RPM retirement in 2.7, the gateway authentication/API change in 2.7 and the AWX freeze are three recent examples that a governance plan must assume recur **(verified** items).
9. **AI surfaces.** Self-service portals and assistant/MCP features expand who can trigger automation and introduce explicitly out-of-scope support boundaries **(verified** in §3.14/§4.5); treat as a second, gated programme.

---

## 10. The Claims Audit

| # | Claim | Verdict | Source (verified date) |
| --- | --- | --- | --- |
| 1 | Red Hat announced the Ansible, Inc. acquisition on **16 October 2015**, expecting to close in October 2015; the then-portfolio listed "Ansible and Ansible Tower" | **Verified** | redhat.com press release *Red Hat to Acquire IT Automation and DevOps Leader Ansible* (2026-09-13) |
| 2 | "Ansible Tower" was the first commercial product, begun 2013, and became "**automation controller 4.0**", the "improved and renamed Red Hat Ansible Tower", in **AAP 2** | **Verified** | ansible.com blog *Upcoming Changes to the AWX Project* (Matthew Jones); redhat.com blog *What's new in AAP 2: automation controller*, 5 Oct 2021 (2026-09-13) |
| 3 | AAP 2 launch context: *Introducing Ansible Automation Platform 2*, 29 September 2021 | **Verified** | redhat.com blog (2026-09-13) |
| 4 | **AWX is the Apache-2.0 upstream** providing UI + REST API + task engine, "one of the upstream projects for Red Hat Ansible Automation Platform" | **Verified** | github.com/ansible/awx README (2026-09-13) |
| 5 | **AWX releases are paused**: last release **2 July 2024** (24.6.1), refactoring announced, affecting UI/credential types/Operator install methods | **Verified** | AWX README caution + linked blog/forum posts (2026-09-13) |
| 6 | Current product is **AAP 2.7**; release id **2.7.20260603**; default engine **ansible-core 2.16** | **Verified** | docs.redhat.com AAP 2.7 release notes; Red Hat AAP lifecycle Tables 1.2/1.3 (2026-09-13) |
| 7 | Automation controller version for AAP 2.7 is **4.8** | **Verified (corroborated)** | Configuration-as-Code table (`ansible.controller >= 4.8`), lifecycle page (2026-09-13) |
| 8 | Component versions move **independently** of the AAP version | **Verified** | Lifecycle page, Table 1.2 footnote 2 (2026-09-13) |
| 9 | **RPM installation is "Not Available" for AAP 2.7**; containerized RHEL 9/10 and Operator on OCP 4.14–4.22 | **Verified** | Lifecycle page Table 1.2 (2026-09-13) |
| 10 | `ansible-core` current release **2.21.4** (8 Sep 2026); community package **14.4.0** (8 Sep 2026) on core **2.21.0** | **Verified** | github.com/ansible/ansible releases; PyPI; ansible-build-data `14/ansible-14.build` (2026-09-13) |
| 11 | `ansible-core` is a minimal language+runtime package; `ansible` adds curated collections; different versioning schemes; **both GPL-3.0-or-later** | **Verified** | ansible-documentation release/maintenance + installation guides; PyPI licence badges (2026-09-13) |
| 12 | Ansible is **agentless** (SSH / PowerShell remoting / other transports), no databases or daemons on the control node; managed nodes need Python | **Verified** | ansible-documentation installation guide; PyPI design principles (2026-09-13) |
| 13 | **Ansible Vault protects data at rest only**; decrypted content is the author's responsibility; passwords are needed and a secrets manager requires a script | **Verified** | ansible-documentation `vault_guide/vault.rst` (2026-09-13) |
| 14 | The original Tower goal was **auditability and traceability** at scale | **Verified** | ansible.com blog, Matthew Jones (2026-09-13) |
| 15 | Published subscription tiers are **Standard (9–5)** and **Premium (24×7)**; **no list price published** ("pricing varies… talk to a Red Hatter") | **Verified** | redhat.com pricing page (2026-09-13) |
| 16 | A **self-support level license** exists and is limited to the default organisation | **Verified** | AAP 2.7 *Get started as a platform administrator* (2026-09-13) |
| 17 | Managed deployments exist: AWS **managed service** (billed AWS, Red Hat managed, 24×7 SLA), Azure **managed application** (billed Microsoft), plus **self-managed marketplace** options for AWS/Azure/GCP | **Verified** | redhat.com pricing page deployment table (2026-09-13) |
| 18 | In **2.7 all authentication routes through the platform gateway**; direct component tokens and the backward-compatibility API layer are removed | **Verified** | AAP 2.7 upgrade docs (auth/API changes) (2026-09-13) |
| 19 | Execution environments are containerised runtimes; versioned and version-less EE images are published via `registry.redhat.io`/catalog; EEs arrived with **AAP 2.0** | **Verified** (EE arrival via access.redhat.com 2.0 article) | Lifecycle page; access.redhat.com *What's new with AAP 2.0: Updated private automation hub* (2026-09-13) |
| 20 | Automation mesh = control/execution/hop nodes; receptor is the underlying transport | **Verified** (mesh model via AAP 2.7 docs; receptor via AWX release notes referencing a `receptor` command) | AAP 2.7 docs; AWX 24.6.0 notes (2026-09-13) |
| 21 | Private automation hub supports namespaces, approval pipelines, **signing** of collections and EE image management | **Verified** | AAP 2.7 docs index (2026-09-13) |
| 22 | Users/teams changes via the legacy controller API can take **up to 15 minutes** to propagate to EDA | **Verified** | AAP 2.7 admin docs (2026-09-13) |
| 23 | MCP servers carry a **12-month lifecycle**; LLM provider integration and model behaviour are **out of support scope**; Vault OIDC is a **technology preview** | **Verified** | Lifecycle page; AAP 2.7 *What's new* (2026-09-13) |
| 24 | Red Hat stated it is **not changing the Ansible project** and **not adjusting the OSS licence structure** during the AWX refactor | **Verified** | ansible.com blog (2026-09-13) |
| 25 | AAP trial is **evaluation-only**; production use incurs fees and breaches the agreement | **Verified** | redhat.com product-trial terms (2026-09-13) |
| 26 | Semaphore is actively released (v2.19.12, 30 Aug 2026) | **Verified** | github.com/semaphoreui/semaphore releases (2026-09-13) |
| 27 | Rundeck is actively released and commercially backed by PagerDuty (v6.2.1, 9 Sep 2026; Runbook Automation 6.0 line) | **Verified** | github.com/rundeck/rundeck releases; docs.rundeck.com; pagerduty.com (2026-09-13) |
| 28 | Red Hat's own framing: DIY community Ansible + AWX suits smaller organisations, but "DIY automation systems become expensive to operate at scale"; AAP bundles "20+ community projects" | **Verified** | redhat.com *Compare AAP and community Ansible projects* (2026-09-13) |

### 10.1 Flagged (stated as uncertain in the text)

| # | Claim | Why flagged |
| --- | --- | --- |
| F1 | **Managed-node counting formula** (what is counted, when, peak vs unique, how metrics feed the count) | Not published on pages I could reach; a licensing/budget claim must be confirmed with Red Hat |
| F2 | **Self-support tier limits** (node cap, feature set, support terms) | Existence verified; limits not published on pages reached |
| F3 | **AWS/Azure/GCP marketplace listing states, SKUs and prices** | I used Red Hat's own deployment pages; I did not open the marketplace listings themselves |
| F4 | **"Any 2025–2026 subscription restructuring / automation-hub entitlement change"** | **No primary source found.** Explicitly **not asserted** in this guide |
| F5 | AAP 2.7 **hub / EDA controller** component version numbers | Collapsed table prevented reliable column binding |
| F6 | Community docs' release-status table (14 vs 13 current) | Contradicts PyPI release data; docs lag suspected |
| F7 | EE model "**replaced per-organisation Python virtualenvs**" | Widely described; the specific replacement wording not confirmed at a primary source this session |
| F8 | **Tamper-evidence / immutability** of job history | Stated as architectural reasoning requiring validation in your environment |
| F9 | `forks`/`serial` scaling limits as a **practical** ceiling | Design characterisation; not a quoted vendor limit |
| F10 | AAP pricing page's "**RHEL 8.3+**" self-managed line | Contradicted by the 2.7 lifecycle matrix (RHEL 9/10) — marketing page lag |
| F11 | **First use of the "Ansible Automation Platform" name** (pre-2.0 existence) | Not verified; this guide treats the *AAP 2 rename of Tower* as verified and the name's earlier history as unknown |
| F12 | Semaphore feature parity with the three §6.2 triggers | Release activity verified; feature set not independently tested |
| F13 | Third-party **Jenkins/CI Ansible plugin** maintenance status | Not verified this session |
| F14 | Hiring/market premium for AAP-platform skills | Market characterisation, no source |
| F15 | Tower's own first release **date** (2014) | Only "worked for over a year… from 2013" is verified; the exact first-release date is not |
| F16 | AWS/Azure **managed-service** components' technical constraints (e.g. allowed topologies, whether mesh is supported) | Not verified; a regulated estate must confirm, since managed service may be excluded by policy |

### 10.2 Rejected (claims that circulate and are wrong or misleading)

| Claim | Why it is rejected |
| --- | --- |
| "Ansible Tower was open source / you can still download Tower" | Tower was the **commercial** product; the open-source upstream is **AWX** (Apache-2.0). Tower as a separate downloadable product no longer exists as such; what survives is the name in old docs and the `awx` identifiers in current ones |
| "AWX is the free version of AAP, so we get the same product for free" | Same lineage, not same product: AWX is **frozen** (releases paused since July 2024) while AAP has progressed through 2.5/2.6/2.7, and AWX carries **no vendor support**. The skill model ports; the support, content curation, HA story and upgrade path do not |
| "Ansible needs nothing at all on managed nodes" | Nothing *installed for Ansible* — but managed nodes need Python (and an SSH-capable account) for most modules; network modules are the documented exception (upstream installation guide) |
| "Ansible Vault solves our secrets-management problem" | Vault protects **at rest only**; it has no custody model, no per-user access and no run-time protection (upstream vault guide) |
| "Renaming Tower to AAP happened because of a licence change" | The rename accompanied an **architecture** change (AAP 2: controller 4.0, execution environments, hub); the OSS licence structure was not changed, and Red Hat explicitly reaffirmed that during the AWX refactor |
| "The audit trail proves the change was authorised" | It proves execution. Authorisation lives in the change record; the join is yours to build |
| "A dashboard = governance" | The dashboard is the artefact most often mistaken for the control; §6.3 Anti-pattern A exists because of this claim |

---

## 11. What Could Not Be Verified

An explicit list of the things this guide **does not know**, so that no reader mistakes silence for confirmation. Each is a task for whoever acts on this material.

1. **AAP subscription price and node-counting formula.** No list price is published; the counting rule (definition of a managed node, measurement period, treatment of ephemeral/cloud/network devices) could not be confirmed at a primary source. **Do not build a budget or a node-reduction plan on an assumption here.**
2. **The limits and entitlements of the "self-support level license"** — verified to exist and to restrict you to the default organisation; nothing else about it verified.
3. **AWS / Azure / Google Cloud marketplace listing contents** (SKU names, prices, billing terms) — Red Hat's own deployment pages were verified; the marketplace pages were not opened.
4. **Any 2025–2026 subscription restructuring or price-book change.** None found; **no such claim is made in this guide.** The *entitlement-provisioning* path, however, is no longer unknown: it changed at **AAP 2.5**, from simple attachment to a **Hybrid Cloud Console service-account / manifest** model — now recorded as a verified bullet in **§4.5** (source: <https://access.redhat.com/articles/5807761>, checked 2026-09-13). That is a provisioning change, **not** a pricing or licence-terms change.
5. **AAP 2.7 automation hub and Event-Driven Ansible controller component versions** — collapsed table prevented reliable binding (see F5); the commercially significant controller/core numbers were corroborated elsewhere.
6. **The exact date of Ansible Tower's first commercial release**, and the date Tower 3.x became controller 4.x beyond the October 2021 blog. (The 2013 start of development and the 2021 rename are verified; the gaps are not.)
7. **The first use of the "Ansible Automation Platform" name** in Red Hat's product line (pre-2.0 packaging is not verified here).
8. **Whether AWX has resumed releases after this writing.** The README's paused-releases notice was accurate on 2026-09-13; the refactor's completion date is unknown, and the **next AWX release is a material event** for anyone relying on the upstream.
9. **Detailed marketplace/managed-service constraints for regulated deployment** (supported topologies, mesh support, data-residency terms) — verify with Red Hat before assuming a managed service is permissible.
10. **docs.ansible.com** — the primary documentation site was unreachable from my fetcher on 2026-09-13; I worked from the upstream documentation **source files** (`ansible/ansible-documentation`) and PyPI instead. Anything in this guide attributed to upstream docs was read there; pages not reachable at all (e.g. the ad-hoc command guide, the porting guides) were not re-verified this session, and where their content would have mattered I have flagged rather than asserted.
11. **Execution-environment "replaced virtualenvs" wording**, and the **practical** scaling ceilings of `forks`/`serial` (F7, F9).
12. **The Jenkins/GitLab/GitHub-Actions Ansible plugin landscape** — maintenance and security status not verified.
13. **Tamper-evidence properties of job history** as configured in any specific deployment — an architectural caution, not a quoted property.
14. **Semaphore's feature depth** against the three adoption triggers — release activity verified; capability not tested.

---

## 12. Glossary

A working dictionary for *this* guide: the terms that carry a specific meaning in Ansible and the control-plane lineage. Generic IT vocabulary (SSH, CI, RBAC-in-general) is deliberately omitted, and where the guide uses a common word in a narrow sense, that sense is the one given.

- **AAP (Red Hat Ansible Automation Platform):** Red Hat's commercial **subscription** platform; bundles the automation controller, automation hub, Event-Driven Ansible and automation mesh into one governed product. The successor line to Ansible Tower.
- **AWX:** the **open-source upstream** controller (Apache-2.0) from which Tower was built and from which AAP's controller descends. Now the **community** controller, frozen since July 2024.
- **Ansible Tower:** Ansible, Inc.'s **commercial** controller product (the commercial line; **renamed** into the controller in 2021 — §11 item 6 records that the *start* year is not verified); never the open-source one.
- **ansible-core:** the **engine** — the CLI, the playbook/role execution model, the bundled modules and the plugin framework. GPL-3.0-or-later. The free thing this guide is about.
- **ansible (the community package):** the PyPI `ansible` distribution bundling `ansible-core` plus the ~100 community collections. Convenient, but not the engine itself.
- **ansible-navigator:** the CLI that runs playbooks **inside an execution environment** rather than on your workstation; the platform-era replacement for bare `ansible-playbook` in governed pipelines.
- **ansible-builder:** the tool that builds an **execution environment** from a definition file; EEs are containers, and this is how one is made.
- **ansible-test:** the engine's own unit/integration/lint harness, used by collection authors and CI, not by operators running playbooks.
- **automation controller:** the modern name for the Tower component inside AAP 2.x — web UI, REST API, job engine, RBAC, scheduling, credential custody and job history. Launched as **automation controller 4.0** in 2021.
- **automation hub:** the **content** half of AAP — a curated, signed repository of certified collections and EEs that keeps automation supply inside the subscription.
- **private automation hub:** the **on-premises** automation hub; holds your own collections and mirrors certified content into an estate that cannot fetch from the internet.
- **platform gateway:** the single **authentication and API entry point** introduced in AAP 2.7; all component traffic and identity funnels through it, and direct component-level tokens were removed.
- **automation mesh:** AAP's overlay network for **distributed execution** — it lets an estate run jobs across sites and DMZs without a full mesh of pairwise SSH trust.
- **receptor:** the **node agent and transport** that implements automation mesh; mesh nodes are Receptor nodes.
- **execution node:** a mesh node that **runs jobs**. In mesh vocabulary a machine is typed by what it does, not where it sits.
- **hop node:** a mesh node that **only relays** traffic between control and execution nodes — used to cross a network boundary without opening it.
- **control node:** the node that **launches** jobs and holds the controller; in engine-only use, the laptop or CI runner that executes `ansible-playbook`.
- **hybrid node:** a mesh node that **both relays and executes**; the labelled default role of a hosted AAP node.
- **execution environment (EE):** a **container image** bundling ansible-core, collections and dependencies; the atomic unit of "what ran this job", replacing per-organisation Python virtualenvs in the AAP line.
- **collection:** the packaging format for Ansible content (modules, roles, plugins, docs); the unit of distribution, versioning and support.
- **certified content:** collections and EEs **validated by Red Hat** and delivered through automation hub; certified-ness is per collection and carries its own lifecycle.
- **job template:** the controller object fixing a **playbook + inventory + credentials + EE + limits** into something runnable and auditable. The core unit of governed execution.
- **workflow template:** a **DAG of job templates** with success/failure/always branches and approvals — the controller's answer to multi-step automation.
- **survey:** the typed, prompted **input form** attached to a job or workflow template; how a playbook is parameterised for humans without handing them the variables file.
- **project (SCM-backed):** the controller's link to a **Git (or other SCM) repository** holding playbooks; the controller syncs and versions the source it runs.
- **dynamic inventory source:** an **inventory plugin plus configuration** that queries a live system (cloud API, CMDB, database) instead of a static hosts file.
- **inventory sync:** the scheduled job that **refreshes** a dynamic inventory source and records what was found — the evidence that a run used live truth.
- **schedule:** the controller object that **triggers** a job or workflow on a timetable; the reason the control plane, rather than a human, can be the actor.
- **credential:** the **stored, access-controlled authentication material** (SSH key, password, token, cloud role) the controller injects into a run — custodied centrally, not held by each engineer.
- **credential type:** the **schema** defining what a credential holds and where the injected fields go; how new target systems are onboarded without rewriting the platform.
- **organisation:** the top-level **tenancy and RBAC boundary** in the controller; inventories, credentials, projects and templates all hang off it.
- **team:** a **subdivision of an organisation**, used to group users and grant role-based access at a finer grain.
- **RBAC (organisations / teams / roles):** the controller's model granting **users or teams specific roles** on specific objects; the governance the engine lacks.
- **managed node:** any **target** the automation acts on. Counted by **host metric** for subscription purposes.
- **host metric:** the **unit of subscription consumption** — a measure of the managed nodes you automate, not of jobs run or users. Its counting rule is the pricing question to resolve with Red Hat (§11).
- **subscription manifest:** the **entitlement file** attached to an AAP instance to activate a subscription; since AAP 2.5 it is one of four supported attachment paths (§4.5).
- **Event-Driven Ansible (EDA):** the AAP capability that **triggers automation from events** (webhooks, Kafka, monitoring alerts) via rulebooks, rather than from schedules.
- **rulebook:** the EDA **source-condition-action** document — watch a source, test a condition, run a playbook. EDA's equivalent of a playbook.
- **automation orchestrator:** the AAP 2.7-era component coordinating **large multi-step workflows across controllers and hubs**; the direction in which the platform becomes a fabric rather than a box.
- **MCP server:** the **Model Context Protocol** server shipped in recent AAP for LLM/agent integration; note its **12-month lifecycle separate from AAP** and that LLM behaviour is explicitly out of support scope (§4.5).
- **playbook:** the ordered YAML file of **plays** mapping hosts to tasks; the engine's primary artefact.
- **role:** a **reusable, structured bundle** of tasks, handlers, templates, files and defaults; the unit of engineering reuse inside a playbook.
- **vault (ansible-vault):** the engine's **file-encryption** tool. Protects secrets **at rest only** — no custody model, no per-user access, no run-time protection (§10.2).
- **ad-hoc command:** a single **one-shot module invocation** (`ansible -m <module> -a <args>`) with no playbook; fine for diagnosis, wrong as a delivery mechanism.
- **ansible-pull:** running Ansible **from the managed host**, pulling a repository and executing locally; the inversion of the push model, used where central push is impossible.
- **fork:** the **parallelism unit** of the engine (`forks`, default 5) — how many hosts are processed at once. A scaling knob, not a licence gate.
- **serial:** the **batch size** for a play — how many hosts advance together before the next batch; the roll-out control for changing one node at a time.
- **become:** the engine's **privilege-escalation** directive (`become: true`, `become_user`); how a run obtains root without the account being root.
- **break-glass:** the **deliberate, logged bypass** of normal control-plane custody — direct engine use with an emergency credential — permitted only with a recorded route back into the governed path.

### The naming lineage, for decoding old documents

Decoding rule: whenever you read "**Tower**" in a vendor or internal document, translate it to "**automation controller inside AAP**" — unless the document is **pre-2021**, in which case it means the **commercial product that AWX upstreamed**.

| Name you saw in the document | What it actually was | The modern name | Notes |
| --- | --- | --- | --- |
| **Ansible Tower** | Ansible, Inc.'s **commercial** controller (built from the AWX upstream) | **automation controller**, inside AAP 2.x | "Tower" was never the open-source product; the OSS upstream was AWX |
| **AWX** | The **open-source upstream controller** (Apache-2.0) | Still **AWX** — the community controller, now **frozen** (24.6.1, Jul 2024) | Same lineage, not the same product: no vendor support, no content curation, no upgrade path |
| **Ansible Tower** (late-1.x-era packaging) | The commercial controller as a **component of AAP 1.x** | **automation controller** in AAP 2.x | This is the "Tower you bought", before the 2.0 architecture split; the AAP 1.x date range is itself unverified (§11 item 7) |
| **Red Hat Ansible Automation Platform 2.0** (2021) | The re-launch that split the monolith into **controller + hub + EEs + mesh** | **AAP 2.x** (2.5 → 2.6 → 2.7) | Its controller launched as **automation controller 4.0 — "the improved and renamed Ansible Tower"** (Red Hat announcement, Oct 2021) |
| `tower-cli`, `awx` | The **CLI and API identifiers** of the Tower/AWX era | **`awx`** endpoints and modules; **`ansible.controller`** and legacy **`ansible.tower`** collections | These identifiers survive in current code — the fastest way to date a document you have been handed |
| **Red Hat Ansible Automation Platform** | The **subscription brand** for the whole governed stack | Same | You buy the platform, not the product, and it is measured in estate size |

---

## 13. Cross-References and Further Reading

### 13.1 In this repository

Read these for what each covers; this guide deliberately does not re-derive them.

- [`configuration_management_languages_guide.md`](configuration_management_languages_guide.md) — the **CM-language comparison**: Ansible's YAML/Jinja2 model against Puppet, Chef, Terraform, CUE, Jsonnet, Dhall, Pkl and Nix. This guide assumes that verdict and does **not** re-explain the language model.
- [`lightweight-config-mgmt-analysis.md`](lightweight-config-mgmt-analysis.md) — the **lighter-alternatives landscape**: Salt, Fabric, Rex and the single-binary Go/Rust generation. §8 names the same tools as boundary cases and does **not** re-derive their merits.
- [`kargo_gitops_guide.md`](kargo_gitops_guide.md) — the **GitOps-controller contrast** and the "do we even need a controller?" question, which is this guide's §6 discussion seen from the delivery side.
- [`iac_best_practices_guide.md`](iac_best_practices_guide.md) — **general IaC practice** and pipeline design; the working context in which the engine-versus-control-plane choice is actually made.
- [`power_platform_cicd_guide.md`](power_platform_cicd_guide.md) — **platform CI/CD**: how the runner, pipeline and promotion model around automation are built; complements §6's CI-triggered adoption path.
- [`../banking/operational_resilience_framework_guide.md`](../banking/operational_resilience_framework_guide.md) — the **operational-resilience anchor** behind §7's resilience classification and the Cymbal Bank worked example; read it for how an outage is measured and evidenced, which is the standard the control plane is asked to meet.

### 13.2 Primary sources to re-check before acting

Naming, versions and licences go stale in months, so **re-verify rather than re-quote**. The URLs this guide actually leans on:

- **<https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform>** — the current AAP version and the component version numbers (controller, hub, EDA, gateway). If this says something other than 2.7, the version claims above are already behind.
- **<https://access.redhat.com/support/policy/updates/ansible-automation-platform>** — the AAP **lifecycle** and the **per-release ansible-core default matrix**. The fastest-moving table in the subject; check it before asserting which `ansible-core` a given AAP ships.
- **<https://www.redhat.com/en/technologies/management/ansible/pricing>** — the **Standard / Premium** tiers and the deployment options. It publishes **no list price**; the page tells you which tiers exist, not what they cost.
- **<https://github.com/ansible/awx/releases>** and **<https://api.github.com/repos/ansible/awx/releases/latest>** — whether **AWX upstream has resumed releasing**. A new AWX release after Jul 2024 is a material event for any "we'll just run AWX" plan (§11 item 8).
- **<https://github.com/ansible/ansible/releases>** and **<https://api.github.com/repos/ansible/ansible/releases/latest>** — the **upstream engine** version; the free denominator against which the platform moves.
- **<https://access.redhat.com/articles/6057451>** — what an **AAP subscription includes** (components, support, content delivery); the entitlement reading.
- **<https://access.redhat.com/articles/6184841>** — the **2021 announcement** of automation controller 4.0 and the Tower rename; the primary source for the naming-lineage table in §12.
- **<https://access.redhat.com/articles/5807761>** — **subscription attachment**, which changed at **AAP 2.5** (four ways to attach, manifest file, Hybrid Cloud Console service-account credentials); the source for the §4.5 bullet.

**How to keep this guide honest:** treat the lineage, the version numbers and the licence identifiers as claims with dates, not as constants. The lineage table is stable; the version matrix is not. Re-check the two lifecycle URLs first — if they have moved, everything downstream of §4 needs a pass.

---

## 14. Closing Summary

The spine of this guide is a single boundary, and the whole document is a description of it.

- **The engine is free, capable and ungoverned.** `ansible-core` runs plays over SSH, with no agent and no licence; it is GPL-3.0-or-later and actively released. It gives you a language, a module ecosystem and idempotence — and nothing else. No custody, no evidence model, no tenancy, no scheduling authority, no support.
- **The control plane is paid, heavier and governed.** AAP wraps that engine in a controller, a content hub, a mesh and a gateway, and sells the result as a subscription measured in the size of the estate. What you are buying is not more automation but the ability to say *who* ran *what*, *with whose credentials*, *against which hosts*, *on whose authority* — and to have someone accountable when it fails.
- **The naming lineage is the map of that boundary.** Tower was the commercial product; AWX is the open-source upstream it was built from; the 2021 relaunch renamed Tower's successor **automation controller 4.0, inside AAP**. Decode every old document against that table and the capability questions answer themselves.
- **The decision is not about features.** The engine can execute almost anything the platform can. The decision is about **operators, entitlements, credential custody, evidence and scale** — who is allowed to run it, what the subscription entitles you to, where the secrets live, what proof survives the run, and whether the estate has outgrown human-scale operation.
- **Both directions fail honestly, so name the failure before choosing it.** Stay on the engine and the failure mode is **undocumented, unowned, credential-sprawling automation** that works until the person who wrote it leaves. Cross to the control plane prematurely and the failure mode is **an expensive platform operated by nobody**, with the estate's real automation still living in someone's home directory.
- **The test you can apply tomorrow:** pick the single most consequential change you currently make by hand, then ask where the record of *who authorised it and with whose credentials* lives. If the honest answer is "in someone's memory and in a Git commit", you are engine-only and you know your next step. If it is "nowhere, and that is a finding", you know why this guide exists.

The engine is free and capability was never the question. The question is who holds the credentials, and who can prove what was done — **the control plane.**
