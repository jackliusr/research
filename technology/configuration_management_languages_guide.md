# Configuration Management Languages — The Languages & DSLs of CM and Infrastructure-as-Code

**Generated:** 2026-08-16
**Scope:** A comprehensive deep-dive on the languages and DSLs behind configuration management (CM) and infrastructure-as-code (IaC) tools — the declarative vs imperative split, the tool languages (Ansible/YAML, Puppet DSL, Chef/Ruby, SaltStack/YAML, Terraform/HCL, Pulumi, CloudFormation), the new generation (CUE, Jsonnet, Dhall, Pkl, Nix), language-design dimensions (typing, hermeticity, abstraction), comparison and selection guidance, a worked example in four languages, and the 2026+ trends. Facts marked **(verified)** were confirmed against primary sources in August 2026; approximate figures are flagged.

**Related guides in this series:** [terraform_state_conflicts.md](terraform_state_conflicts.md) (state deep-dive) · [audit_as_code_guide.md](audit_as_code_guide.md) (policy-as-code) · [kargo_gitops_guide.md](kargo_gitops_guide.md) (GitOps) · [cloud_providers_guide.md](cloud_providers_guide.md) (cloud IaC) · [finops_guide.md](finops_guide.md) (cost tagging) · [container_certificates_guide.md](container_certificates_guide.md) (TLS certs in configs)

---

## Table of Contents

1. [The CM Language Landscape](#1-the-cm-language-landscape)
2. [Declarative, Idempotency, Convergence — The Core Principles](#2-declarative-idempotency-convergence--the-core-principles)
3. [Ansible and YAML](#3-ansible-and-yaml)
4. [Puppet and Chef](#4-puppet-and-chef)
5. [Terraform and HCL](#5-terraform-and-hcl)
6. [Pulumi and CloudFormation](#6-pulumi-and-cloudformation)
7. [The New Generation: CUE, Jsonnet, Dhall, Pkl, Nix](#7-the-new-generation-cue-jsonnet-dhall-pkl-nix)
8. [Language Design: Typing, Hermeticity, Abstraction](#8-language-design-typing-hermeticity-abstraction)
9. [Comparison and Selection Guidance](#9-comparison-and-selection-guidance)
10. [Worked Example: The Same Config in Four Languages](#10-worked-example-the-same-config-in-four-languages)
11. [The Future (2026+): Typed Config, Policy-as-Code, GitOps](#11-the-future-2026-typed-config-policy-as-code-gitops)
12. [Glossary](#12-glossary)

---

## 1. The CM Language Landscape

Every configuration management and infrastructure-as-code tool is built around a language: the syntax in which you express "what the machine should look like." Understanding the tool landscape is, at its core, understanding the *languages* the tools speak. This guide maps that landscape, from the 2003-era pioneers (Nix, Puppet) to the 2024 entrants (Pkl), and the design ideas that separate them.

Two axes organize the entire field:

- **What you express** — *desired state* (declarative) vs *steps to execute* (imperative); **what you manage** — server/OS state (CM) vs cloud infrastructure (IaC) vs raw config data (templating/language layer).

### 1.1 The Declarative vs The Imperative

**The declarative (the desired state).** You describe the end state — "this package is installed, this file contains this content, this service is running" — and the tool figures out the steps. The configuration is a *specification*, not a recipe. **(verified)** — this is the standard definition of declarative configuration used by Puppet, Terraform, and NixOS documentation alike. Declarative configs are generally:

- **Re-runnable** — expressing state means running it twice is safe (idempotency, §2.2); **comparable** — current state diffs against desired state (drift, §2.3); **portable** — one spec targets many environments.

**The imperative (the steps).** You write the sequence of actions: "install package X, then edit file Y, then restart service Z." Shell scripts are the archetype. Imperative configs are:

- **Order-dependent** (step 3 assumes step 2); **not re-runnable by default** (running twice may double-install or fail); **hard to reason about mid-failure** (what state is the machine in after 40% of the steps?).

The dominant trend across four decades of tooling has been *from imperative toward declarative* — but as §4 (Chef) shows, many production tools are pragmatically hybrid: a declarative model executed by imperative machinery, or a general-purpose language driving a declarative engine (Pulumi, §6).

### 1.2 The CM Tools and Their Languages

Configuration management (CM) tools manage the state of servers and their software. Each ships with its own language:

**Ansible (the YAML).** **(verified)** Ansible was created by Michael DeHaan and first released in 2012; Red Hat acquired it in October 2015. Configs are **playbooks written in YAML** — a data-serialization format, not a programming language — extended with Jinja2 templating and a library of **modules** (small Python programs that perform a single idempotent action). Agentless over SSH (see §3).

**Puppet (the DSL).** **(verified)** Puppet (2005, Luke Kanies) introduced a purpose-built **declarative DSL** — the Puppet language — where resources are declared (`package { 'nginx': ensure => installed }`) and the puppet agent converges the node toward that state. The DSL is deliberately small: it is not a general programming language, which is precisely its point — you *cannot* express a non-idempotent side-effect sequence in it.

**Chef (the Ruby).** **(verified)** Chef (2009, Opscode/Adam Jacob) chose the opposite strategy: configuration is **Ruby code** (recipes and cookbooks), using Ruby's expressiveness with a library of declarative resource helpers (`package 'nginx' do action :install end`). Full programming power — loops, conditionals, libraries — at the cost of requiring Ruby literacy and discipline.

**SaltStack (the YAML).** **(verified)** SaltStack (Salt, 2011, Thomas Hatch) expresses states in **YAML SLS files** (Salt State files) with Jinja2 templating, layered over a fast, parallel, agent-based master/minion transport (with agentless SSH modes). Its language resembles Ansible's YAML approach but its execution model is pull-based and event-driven.

### 1.3 The IaC Languages

Infrastructure-as-code (IaC) tools provision cloud infrastructure — networks, VMs, containers, serverless — rather than configuring the inside of a single OS. (Cross-ref [cloud_providers_guide.md](cloud_providers_guide.md) for the provider-side landscape.)

**Terraform (the HCL).** **(verified)** Terraform was released by HashiCorp in July 2014 together with its dedicated language, **HCL — the HashiCorp Configuration Language** (§5). Declarative resource blocks (`resource "aws_instance" "web" { ... }`) are evaluated into a plan, then applied against the cloud provider APIs, with a state file tracking reality.

**Pulumi (the TypeScript/Python/Go).** **(verified)** Pulumi (founded 2017, product released 2018) inverts the HCL bet: you write infrastructure in **general-purpose languages** — TypeScript, JavaScript, Python, Go, C#, Java, and others — as ordinary code that constructs resources via SDK objects, while a declarative engine underneath diffs and applies the desired state.

**CloudFormation (the YAML/JSON).** **(verified)** AWS CloudFormation (launched 2011, YAML support added 2018) is AWS's native IaC: **templates written in YAML or JSON** declare resources (`AWS::EC2::Instance`), which CloudFormation creates, updates, and deletes as a managed stack, including rollback on failure. Proprietary and AWS-only, but zero extra tooling for AWS shops.

### 1.4 The New Generation

A wave of **configuration languages proper** — languages whose *entire job* is configuration data, independent of any single tool — emerged from 2014 onward:

- **Jsonnet** — Google's data-templating language, open-sourced 2014 **(verified)**; JSON with functions, inheritance, and imports; pure and side-effect-free.
- **CUE** — "Configure Unify Execute" **(verified)**, open-sourced 2018; constraint-based language where schemas and data are the same value (see §7.1).
- **Dhall** — **(verified)** released 2016 by Gabriel (now Gabriella) Gonzalez; a programmable, *total* (guaranteed-terminating), strongly typed configuration language.
- **Pkl** — **(verified)** Apple's configuration-as-code language, open-sourced February 2024 under Apache 2.0, pronounced "Pickle"; typed classes compiling to JSON/YAML/TOML.
- **Nix** — **(verified)** the Nix language dates to 2003 (Eelco Dolstra, Utrecht University; 2006 PhD thesis *The Purely Functional Software Deployment Model*); a pure, lazy, functional language that defines *derivations* — reproducible builds — and powers NixOS, the purely functional Linux distribution.

These languages are the "meta-layer": they generate the YAML/JSON/HCL that the tools consume, and several (CUE, Nix, Pkl) can also validate it. Their rise is the central plot of the 2020s (§7, §8, §11).

### 1.5 The Landscape Table

| Category | Languages | Tools | Paradigm |
|---|---|---|---|
| Configuration management (server/OS state) | YAML + Jinja2 | Ansible | Declarative-ish, task-ordered, agentless |
| Configuration management (server/OS state) | Puppet DSL | Puppet | Declarative resource model, agent pull |
| Configuration management (server/OS state) | Ruby (DSL) | Chef | Declarative resources in imperative language, agent pull |
| Configuration management (server/OS state) | YAML SLS + Jinja2 | SaltStack | Declarative states, master/minion (or agentless) |
| Infrastructure provisioning (IaC) | HCL (HCL2) | Terraform / OpenTofu | Declarative, plan-apply, state-tracked |
| Infrastructure provisioning (IaC) | TypeScript/Python/Go/C#/Java | Pulumi | Declarative engine, imperative-looking code |
| Infrastructure provisioning (IaC) | YAML/JSON templates | AWS CloudFormation | Declarative stacks, AWS-native |
| Configuration data (meta-layer) | Jsonnet | Tanka, ksonnet, Grafana | Pure data templating, functional |
| Configuration data (meta-layer) | CUE | Timoni, Dagger, cue CLI | Constraint-based, schema+data unified |
| Configuration data (meta-layer) | Dhall | dhall CLI, dhall-to-yaml | Total functional, strongly typed |
| Configuration data (meta-layer) | Pkl | pkl CLI, build plugins | Typed classes → JSON/YAML/TOML |
| Package/system configuration | Nix | Nix, NixOS, nixpkgs, Home Manager | Pure functional, derivations, hermetic |

---

## 2. Declarative, Idempotency, Convergence — The Core Principles

Three principles do the real work behind every CM language. They are worth stating precisely, because each language in this guide is essentially a different *mechanism* for delivering them.

### 2.1 The Declarative: The Desired State

**(verified)** A declarative configuration is a statement of *desired state* — the condition the system should be in — not a script of actions. In Puppet:

```puppet
package { 'nginx': ensure => installed }
service { 'nginx': ensure => running, enable => true }
```

The manifest says "nginx is installed and running." The Puppet agent compares the node's actual state against this and performs only the actions needed to close the gap. The same config on a fresh VM, an already-correct VM, or a half-configured VM produces three different action sets and one identical outcome. That property is the entire value proposition of declarative CM.

### 2.2 The Idempotency: The Re-Runnable

**(verified)** An operation is **idempotent** if applying it multiple times has the same effect as applying it once. A declarative config is inherently idempotent at the *specification* level: the tool determines "already satisfied?" before acting, so re-runs are no-ops for anything already correct. Concretely:

- **Ansible modules** check state before changing it — `state: present` on an installed package does nothing (the documented idempotent module contract) **(verified)**.
- **Terraform plan** diffs state vs config; unchanged resources generate no API calls.
- **NixOS** `nixos-rebuild switch` recomputes the system derivation and activates only what changed; **Puppet** resources check state and only *sync* when out of compliance.

Idempotency is why CM is safe to run from cron, CI, or a Friday-afternoon panic — repeatedly.

### 2.3 The Convergence: The Drift Correction

**Convergence** is the runtime behavior that follows from declarative + idempotent: the system is driven *toward* the desired state whenever it is found *away* from it. **Drift** is the divergence between actual state and desired state — caused by manual `ssh` edits, failed runs, hotfixes, or time. The CM loop is:

1. Read desired state → 2. Read actual state (inventory, facts, state file, or live API) → 3. Diff → 4. Apply only the missing changes → 5. Report.

- **Ansible** converges only when run (push model); **Puppet/SaltStack/Chef** converge on a schedule (agent pull, e.g., every 30 min).
- **Terraform** converges on `apply` (`-auto-approve` in CI) — the plan/apply cycle *is* drift correction; see [terraform_state_conflicts.md](terraform_state_conflicts.md). **GitOps controllers** (see [kargo_gitops_guide.md](kargo_gitops_guide.md)) converge *continuously* — reconciliation is drift correction made permanent.

### 2.4 The Principles Table

| Principle | Meaning | Example |
|---|---|---|
| **Declarative** | Config states the desired end state, not the steps | `resource "aws_instance" "web" { instance_type = "t3.micro" }` — Terraform figures out create/update/delete |
| **Desired state** | The specification itself: "this is how the system must be" | Puppet `service { 'nginx': ensure => running }` |
| **Idempotency** | Re-running the config is safe and converges to the same result | `ansible.builtin.apt: name=nginx state=present` twice → one install, one no-op |
| **Convergence** | The system is actively corrected toward desired state | Puppet agent runs every 30 min and fixes drift |
| **Drift** | Actual state ≠ desired state (manual changes, time) | Someone runs `systemctl stop nginx` on a managed host; next agent run restarts it |

---

## 3. Ansible and YAML

### 3.1 The Ansible: The Agentless

**(verified)** Ansible's defining architectural choice: **no agent**. The control node (where Ansible runs) reaches managed nodes over SSH (Windows: WinRM) using Python on the target. Nothing is installed on the managed host beforehand — it needs only SSH access and Python. The five design pillars, all traceable to Ansible's 2012 origin story:

1. **Agentless** — no daemon to deploy, version, or trust; 2. **SSH transport** — leverages existing security; 3. **Python** — ubiquitous on Linux targets; 4. **YAML** — minimal learning curve; 5. **Idempotent modules** — the module contract (§2.2).

This made Ansible the most accessible CM tool and drove its dominant adoption after the 2015 Red Hat acquisition (now Red Hat Ansible Automation Platform; the community project is ansible-core). *(Market-share claims are approximate — the consistent finding across surveys through 2024-2026 is that Ansible remains the most widely used CM tool, with Puppet and Chef in decline.)*

### 3.2 The YAML: The Playbooks, The Modules

**(verified)** Ansible configs are **playbooks**: YAML documents listing plays (who to run against) and tasks (what to do). The actual work happens in **modules** — self-contained Python units with a documented contract: *check state, change only if needed, report changed/ok/failed*. There are thousands of modules (`ansible.builtin.*`, `ansible.posix.*`, vendor modules for cloud, network, Windows, containers). The YAML + Jinja2 combination (variables, loops, conditionals via `when:`) gives Ansible just enough programming power without becoming a language.

### 3.3 The Ansible Deep-Dive: Playbook Structure

A playbook's anatomy:

```yaml
---
- name: Configure nginx reverse proxy
  hosts: webservers
  become: true
  vars:
    server_name: "example.internal"
    backend_port: 8080
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
    - name: Deploy server block
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/example
        mode: "0644"
      notify: reload nginx
    - name: Enable site
      ansible.builtin.file:
        src: /etc/nginx/sites-available/example
        dest: /etc/nginx/sites-enabled/example
        state: link
    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
  handlers:
    - name: reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

Key mechanics: **hosts** select targets from the inventory (static INI/YAML or dynamic cloud inventory); **tasks** run in order and are each idempotent; **handlers** run once at play end, only if notified; **facts** (gathered per host) feed conditionals; **roles** package tasks/vars/templates into reusable units — Ansible's answer to abstraction (§8.3).

### 3.4 The Ansible Table

| Aspect | Description | Example |
|---|---|---|
| **Language** | YAML (data format) + Jinja2 templating; no custom language | `- ansible.builtin.copy: src=nginx.conf dest=/etc/nginx/` |
| **Model** | Agentless push over SSH; ordered task list; per-task idempotency | Control node → SSH → Python module runs on target |
| **State tracking** | No central state file; each module checks target state at run time | `state: present` vs `state: absent` |
| **Idempotency** | Module contract: report `changed` or `ok`, never "already-done failure" | Re-running a playbook is safe by design |
| **Convergence** | Only when invoked (manual, cron, AWX/Ansible Tower, CI) | `ansible-playbook -i inventory nginx.yml` |
| **Abstraction** | Roles, includes, vars, Jinja2 templates, collections | `ansible-galaxy collection install community.docker` |
| **Ecosystem** | ansible-core, Ansible Automation Platform, Ansible Galaxy, AWX, ~thousands of modules | `ansible.builtin.*`, `amazon.aws.*`, `kubernetes.core.*` |
| **Typing/schema** | None intrinsic to YAML; no validation beyond syntax (lint via ansible-lint) | Typos surface at runtime, not parse time |

---

## 4. Puppet and Chef

### 4.1 The Puppet: The Declarative DSL

**(verified)** Puppet (2005) introduced the first mainstream **declarative configuration DSL**. The Puppet language is deliberately minimal: you declare *resources* with *attributes* and let the agent converge. No loops-as-statements, no arbitrary side effects, no imperative escape hatch in the core language (iteration exists via resource collectors and functions, but the model stays resource-graph-based):

```puppet
package { 'nginx':
  ensure => installed,
}

file { '/etc/nginx/sites-available/example':
  ensure  => file,
  content => template('web/nginx.conf.erb'),
  require => Package['nginx'],
}

service { 'nginx':
  ensure    => running,
  enable    => true,
  subscribe => File['/etc/nginx/sites-available/example'],
}
```

Puppet compiles manifests into a **catalog** — a graph of resources and their dependencies (`require`, `subscribe`) — which the agent (pull model, default ~30 min) applies. The DSL's constraint is the feature: it structurally prevents non-idempotent configurations. The trade-off: expressing complex conditional logic is awkward, pushing users toward Hiera (data lookup) and roles/profiles patterns.

### 4.2 The Chef: The Ruby, The Recipes

**(verified)** Chef (2009) made the opposite bet: **configuration is Ruby**. Recipes (files in cookbooks) run real Ruby code on the node, using a resource DSL:

```ruby
package 'nginx' do
  action :install
end

template '/etc/nginx/sites-available/example' do
  source 'nginx.conf.erb'
  variables(server_name: 'example.internal', backend_port: 8080)
  notifies :reload, 'service[nginx]'
end

service 'nginx' do
  action [:enable, :start]
end
```

Because recipes are Ruby, you get the full language — libraries, loops, conditionals, gems, monkey-patching — and Chef's convergence engine (chef-client, pull model, run-list of roles/recipes) still applies resources idempotently. The cost: teams must be fluent in Ruby, and the imperative freedom must be self-disciplined. Chef was acquired by Progress Software in 2020; enterprise usage has declined significantly since (approximate — industry consensus).

### 4.3 The Comparison: Puppet vs Chef

Both are agent-based, pull-model, declarative-resource CM tools from the same era — the difference is the language philosophy:

| Dimension | Puppet | Chef |
|---|---|---|
| **Language** | Purpose-built declarative DSL (the Puppet language) | Ruby with a resource DSL |
| **Expressiveness** | Deliberately constrained — safe by construction | Full programming language — powerful, needs discipline |
| **Model** | Manifest → catalog (resource graph) → agent applies | Recipes → resources → chef-client converges |
| **Ecosystem** | Puppet Forge modules, Hiera, roles/profiles, Bolt | Supermarket cookbooks, Test Kitchen, InSpec (now part of the compliance story — see [audit_as_code_guide.md](audit_as_code_guide.md)) |
| **Learning curve** | Learn the DSL (small, but new) | Learn Ruby (big, but reuse existing skills) |
| **Status 2026** | Acquired by Perforce (2022); enterprise niche, legacy-heavy | Acquired by Progress (2020); niche, legacy-heavy |
| **Notable legacy** | Proved declarative DSLs work for config; influenced NixOS options & Terraform | Proved general-purpose languages can drive config; direct ancestor of Pulumi's bet |

### 4.4 The Table

| Tool | Language | Model | Ecosystem |
|---|---|---|---|
| **Puppet** | Puppet DSL (declarative, purpose-built) | Agent pull; manifest → catalog → converge | Puppet Forge, Hiera, PE (Puppet Enterprise), Bolt |
| **Chef** | Ruby (resource DSL) | Agent pull; run-list of recipes; converge | Supermarket cookbooks, Chef Infra, InSpec, Test Kitchen |
| **Ansible** | YAML + Jinja2 | Agentless push over SSH; ordered tasks | Galaxy, AWX/AAP, collections, ansible-core |
| **SaltStack** | YAML SLS + Jinja2 | Master/minion (or SSH); event-driven, parallel | Salt modules, SaltCloud, SaltSSH, (VMware 2020→Broadcom) |

---

## 5. Terraform and HCL

### 5.1 The HCL: The HashiCorp Configuration Language

**(verified)** Terraform shipped in July 2014 with its companion language, **HCL — the HashiCorp Configuration Language**, designed by Mitchell Hashimoto and the HashiCorp team. HCL's goals were explicit: a syntax friendlier than JSON for humans (blocks, comments, `=` assignment), yet machine-readable enough to parse, and (later) a JSON-compatible variant (`.tf.json`) so machines can emit it. HCL is not a general-purpose language — it has variables, expressions, functions, and (in HCL 2) iteration, but no arbitrary side effects; it evaluates to a *config tree* that Terraform interprets.

```hcl
variable "instance_type" { type = string default = "t3.micro" }

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
}
```

### 5.2 The HCL 2

**(verified)** HCL 2 shipped with **Terraform 0.12 (May 2019)** — a major language rewrite, not just a version bump:

- **First-class expressions** — `for` expressions, splat (`[*]`), conditionals, rich operators; **`dynamic` blocks** generate repeated nested blocks from collections.
- **Typed variables** — `string`, `number`, `bool`, `list`, `map`, `set`, `object(...)`, `tuple(...)` with per-variable `validation` blocks; **functions** — built-in library plus custom functions.
- **Filesystem layout** — `*.tf` files evaluated per directory; `locals`, `outputs`, `terraform` blocks (required providers, backend).

HCL 2 also became the configuration language for all HashiCorp products (Vault, Consul, Nomad, Packer, Waypoint) — one syntax across the portfolio, with Terraform-specific extensions.

### 5.3 The Terraform Deep-Dive: Resources, Providers, State

**Resources** are the unit of declaration: `resource "<type>" "<name>" { ... }` maps to a cloud object (instance, bucket, SG rule). **Providers** are plugins that translate Terraform's abstract resource model into a specific API (AWS, Azure, GCP, Kubernetes, GitHub, ...); `terraform init` downloads them. The core cycle:

1. **`terraform init`** — fetch providers/modules.
2. **`terraform plan`** — read config + state + live API → compute the diff (add/change/destroy).
3. **`terraform apply`** — execute the plan; record results in **state**.
4. **`terraform state`** — the source of truth for what exists; the plan is only as good as the state.

**State** is where Terraform's power and its dangers live: local `terraform.tfstate` or remote backends (S3+DynamoDB, GCS, Azure blob, Terraform Cloud/OpenTofu remotes), with locking, versioning, and `terraform import`/`state mv` operations. State conflicts, race conditions, and corruption are the classic production incidents — this series' companion guide [terraform_state_conflicts.md](terraform_state_conflicts.md) is the dedicated deep-dive on locking, serialization, and disaster recovery. Two ecosystem notes: OpenTofu (Linux Foundation fork, after HashiCorp moved Terraform to the BUSL license in August 2023) is HCL-compatible; and HCL's declarative model is why drift shows up as *plan diffs* — see [kargo_gitops_guide.md](kargo_gitops_guide.md) for the GitOps loop built around that diff.

### 5.4 The HCL Table

| Aspect | Description | Example |
|---|---|---|
| **Language** | HCL/HCL2 — purpose-built declarative config language (2014; HCL2 2019) | `resource "aws_s3_bucket" "logs" { bucket = "logs-${var.env}" }` |
| **Paradigm** | Declarative desired state + plan/apply execution | `terraform plan` shows the diff before `apply` |
| **Providers** | Plugins mapping resources to APIs | `aws`, `azurerm`, `google`, `kubernetes`, `github`, 2,000+ in registry |
| **State** | `terraform.tfstate` — local or remote backend, locked, versioned | S3 + DynamoDB locking (see terraform_state_conflicts.md) |
| **Typing** | Typed variables + validation blocks + custom conditions (HCL2) | `variable "env" { type = string validation { condition = contains(["dev","prod"], var.env) } }` |
| **Abstraction** | Modules (reusable `.tf` packages), locals, data sources, functions | `module "vpc" { source = "terraform-aws-modules/vpc/aws" }` |
| **Idempotency** | Plan/apply only touches drifted resources | Re-applying an unchanged config → "No changes" |
| **Ecosystem** | Terraform Registry (modules & providers), OpenTofu fork, Terragrunt, tfsec/Checkov (see audit_as_code_guide.md) | `terraform-aws-modules`, `hashicorp/aws` |

---

## 6. Pulumi and CloudFormation

### 6.1 The Pulumi: The General-Purpose Languages

**(verified)** Pulumi (2017/2018) is the "Chef bet" applied to IaC: keep a declarative engine, but let users write infrastructure in **general-purpose languages** — TypeScript/JavaScript, Python, Go, C#, Java, and YAML — using SDK objects instead of a custom DSL:

```typescript
import * as aws from "@pulumi/aws";

const bucket = new aws.s3.Bucket("logs", {
  bucket: `logs-${pulumi.getStack()}`,
  acl: "private",
});
export const bucketName = bucket.id;
```

Because it is real code, you get loops, conditionals, classes, unit tests, static analysis, IDE tooling, and dependency management — and Pulumi's engine still performs the declarative work: it captures the resource graph, diffs against its state, and applies changes. This is the modern expression of the Chef philosophy (general-purpose language + declarative core), and its main selling point against Terraform: no second language to learn, and code-level testability.

### 6.2 The CloudFormation: The YAML/JSON

**(verified)** AWS CloudFormation (2011) is the original mainstream IaC service: **templates in YAML or JSON** (YAML preferred for readability) declare resources, and CloudFormation manages the full lifecycle as a **stack** — ordered create, update with change sets, rollback on failure, deletion. No state file to manage (AWS holds it), no extra tooling, deep AWS integration (IAM, drift detection, stack sets for multi-account).

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t3.micro
```

The language is YAML/JSON with intrinsic functions (`!Ref`, `!Sub`, `!GetAtt`, `!Join`, `!If`) and (since 2018) macros and nested stacks. Its constraints are the mirror of its virtues: AWS-only, no modules ecosystem comparable to Terraform's registry, and template-level abstraction is limited (hence tooling like AWS CDK, which *compiles TypeScript/Python into CloudFormation* — a fascinating hybrid that is effectively "Pulumi-style authoring, CloudFormation execution").

### 6.3 The Comparison: General-Purpose vs DSL

| Dimension | Terraform (HCL DSL) | Pulumi (general-purpose) | CloudFormation (YAML/JSON) |
|---|---|---|---|
| **Language** | Purpose-built DSL | TypeScript, Python, Go, C#, Java, YAML | YAML/JSON templates + intrinsic functions |
| **Learning curve** | Learn HCL (small, but new syntax) | Reuse existing language skills | YAML is familiar; AWS concepts are the real curve |
| **Expressiveness** | HCL2 loops/dynamics; bounded | Full language power (loops, classes, tests) | Limited; macros/nested stacks; CDK on top |
| **State** | tfstate (self-managed, see terraform_state_conflicts.md) | Pulumi service/self-hosted state | AWS-managed stack state |
| **Clouds** | Multi-cloud, 2,000+ providers | Multi-cloud, ~same provider set | AWS-only |
| **Typing** | Typed variables, validation | The host language's types (TS/Python/etc.) | JSON Schema-ish validation, YAML linting |
| **Testing** | `terraform plan` as assertion; external scanners | Native unit tests in-language | `aws cloudformation validate-template`; cfn-lint |
| **Best for** | Standardized multi-cloud IaC, huge module registry | Engineering teams who want code-level power & testing | AWS-native shops wanting managed, tool-less IaC |

### 6.4 The Table

| Tool | Language | Paradigm | Notes |
|---|---|---|---|
| **Pulumi** | TypeScript/JS, Python, Go, C#, Java (+ YAML) | Declarative engine; code-authored resources | General-purpose-language IaC; state via Pulumi Cloud or self-hosted |
| **CloudFormation** | YAML/JSON templates | Declarative stacks, AWS-managed | 2011; change sets, rollback, drift detection; CDK compiles to it |
| **AWS CDK** | TypeScript, Python, Go, Java, C# | Code → CloudFormation templates | Not a new language — a compiler to CloudFormation |
| **AWS SAM / CDK** (serverless) | YAML (+ code) | Declarative serverless IaC | SAM template = CloudFormation subset for serverless |

---

## 7. The New Generation: CUE, Jsonnet, Dhall, Pkl, Nix

These five are *configuration languages* in the strongest sense — their product is validated, generated, reproducible configuration data, independent of any single tool. They are the answer to "YAML hell" (untyped, duplicated, hand-assembled manifests) that Kubernetes surfaced at industrial scale.

### 7.1 The CUE: The CUE Language

**(verified)** **CUE = Configure Unify Execute.** It is a constraint-based configuration language designed by **Marcel van Lohuizen** — Go team veteran and co-designer of Go's `text/template` — informed by his earlier work on Borg's configuration system at Google. First open-sourced in 2018. The defining idea: **schemas and data are the same language**. A CUE value can be concrete data, a schema (constraints), or both simultaneously; unification merges them. This yields:

- **Single source of truth** — the schema *is* the validation, not a separate artifact.
- **Structural typing** — `#Deployment` definitions constrain any value that matches their shape.
- **Order-independent merging** — CUE values unify, so environments can *add* constraints without *copying* config.

```cue
package kube

#Deployment: {
	apiVersion: "apps/v1"
	kind:       "Deployment"
	metadata: name: string
	spec: {
		replicas: int & >=1 & <=10
		image:    =~"^[a-z0-9/:.]+$"
	}
}

web: #Deployment & {
	metadata: name: "web"
	spec:     replicas: 3
	image:    "nginx:1.27"
}
```

`cue vet` against Kubernetes' OpenAPI schemas validates generated manifests before they ever reach a cluster. Adopters/ecosystem include Dagger (CI/CD, originally CUE-based), **Timoni** (CNCF-sandboxed Kubernetes package manager powered by CUE), and internal use at major cloud providers. *(Adoption scale is approximate; CUE is the strongest of the new-gen languages in 2024-2026 Kubernetes-adjacent tooling, per the CNCF landscape and Timoni/Dagger docs.)*

### 7.2 The Jsonnet: The Jsonnet

**(verified)** **Jsonnet** is Google's **data-templating language** — open-sourced August 2014 (Apache 2.0) — described by its authors as "JSON with functions, variables, conditionals, and inheritance" and, crucially, **side-effect free**: evaluation is a pure function producing JSON (or YAML via converters). It answers duplication in large config trees (Kubernetes manifests, Prometheus rules, dashboards) with object-oriented inheritance:

```jsonnet
local base = {
  image: 'nginx:1.27',
  replicas: 2,
  resources: { cpu: '100m', memory: '128Mi' },
};

{
  dev:  base { replicas: 1 },
  prod: base { replicas: 5, resources: { cpu: '500m', memory: '512Mi' } },
}
```

Ecosystem: ksonnet (Kubernetes, largely superseded), **Grafana Tanka** (Jsonnet-based k8s config tooling), Prometheus/Grafana dashboards as code, and Google-internal config at scale. Jsonnet's honest limits: **no schema/type checking** (it is data generation, not validation) — a gap the next three languages target directly.

### 7.3 The Dhall: The Dhall

**(verified)** **Dhall** (released 2016 by Gabriel/Gabriella Gonzalez) is a *programmable, total* configuration language: **guaranteed termination** (non-Turing-complete), **strong static typing**, and **safety for untrusted imports** — evaluating untrusted Dhall code cannot loop forever, access files, or do I/O; all imports are content-addressed and can be pinned by hash. Philosophy: configuration should be *generated by a language you can reason about*, not by a general-purpose program that might hang or lie.

```dhall
let Server = { Type = { serverName : Text, backendPort : Natural }
             , default = { serverName = "example.internal", backendPort = 8080 }
             }
in  Server.default // { backendPort = 9090 }
```

Dhall compiles to JSON/YAML; the type system catches errors at config-build time. Its adoption is niche-but-loved (the cost: Haskell-adjacent syntax and a small community — approximate but honest; Dhall's GitHub remains active with a devoted following, and its ideas influenced typed-config discourse).

### 7.4 The Pkl: The Pkl

**(verified)** **Pkl** (pronounced "Pickle") is Apple's configuration-as-code language, **open-sourced February 2024 under Apache 2.0** — after years of internal use at Apple for build and service configuration. Pkl is a *typed, class-based* language that evaluates to plain data (JSON, YAML, TOML, XML, or property lists), with first-class validation:

```pkl
module nginx

class ServerBlock {
  serverName: String
  backendPort: Int(1, 65535)
  listenPort:  Int = 443
  tls:         Boolean = true
}

server: ServerBlock = new {
  serverName = "example.internal"
  backendPort = 8080
}
```

`pkl eval -f yaml nginx.pkl` emits YAML. Pkl is notable for: real tooling from day one (CLI, IntelliJ/JetBrains plugins, Gradle/Maven/Spring Boot integrations), the credibility of Apple's production use, and explicit design goals against YAML's weaknesses. As of 2024-2026 it is the youngest of the new-gen languages, with adoption concentrated in JVM/Apple toolchains and evaluation elsewhere (approximate).

### 7.5 The Nix: The Nix Language

**(verified)** The **Nix language** (2003, Eelco Dolstra, Utrecht University; 2006 thesis *The Purely Functional Software Deployment Model*) is the oldest and most radical entry: a **pure, lazy, functional** language whose values are *derivations* — build recipes whose outputs are stored under content-derived hashes in `/nix/store`. Every build input is pinned; builds run in sandboxes; nothing is overwritten in place — upgrades create new store paths and switch symlinks, making **rollback trivial** and **reproducibility the default**. NixOS is the Linux distribution built from Nix modules: the *entire system* — packages, services, users, network — is one declarative config tree:

```nix
{ config, pkgs, lib, ... }:
{
  services.nginx = {
    enable = true;
    virtualHosts."example.internal" = {
      listen = [{ addr = "0.0.0.0"; port = 443; ssl = true; }];
      locations."/" = {
        proxyPass = "http://127.0.0.1:8080";
      };
    };
  };
}
```

`nixos-rebuild switch` builds the new system generation and atomically activates it. The Nix language is **dynamically typed** — its safety comes from the module system (typed *options* with validation, e.g., `services.nginx.virtualHosts` checking hostname shape) rather than the language itself. Ecosystem: nixpkgs (~100k packages, approximate), flakes (pinned, hash-locked inputs), Home Manager, devenv, and the Nix/NixOS communities. Nix is the purest realization of hermetic, reproducible configuration — and the template for what GitOps wants to be.

### 7.6 The New-Gen Table

| Language | Org | Year | Typing | Philosophy |
|---|---|---|---|---|
| **Jsonnet** | Google | 2014 (open-sourced) | Untyped (data generation) | Pure, side-effect-free JSON templating with inheritance |
| **Dhall** | Community (G. Gonzalez) | 2016 | Strongly, statically typed | Total (guaranteed termination), safe imports, programmable config |
| **CUE** | Community (M. van Lohuizen) | 2018 (open-sourced) | Structural / constraint-based | Schema and data are one; "Configure Unify Execute" |
| **Nix** | Community (E. Dolstra, Utrecht) | 2003 | Dynamically typed (module-system typed options) | Pure functional derivations; hermetic, reproducible builds |
| **Pkl** | Apple | 2024 (open-sourced) | Statically typed classes + constraints | Typed config-as-code compiling to JSON/YAML/TOML |

---

## 8. Language Design: Typing, Hermeticity, Abstraction

Underneath the tools, the languages differ along three design axes that predict their behavior — and their future (see §11).

### 8.1 The Typed Configuration: Schema and Validation

**(verified)** YAML, JSON, and HCL (mostly) are *untyped data*: the parser accepts anything, and errors appear at runtime — a typo in a Kubernetes manifest is a failed deploy, not a failed compile. The new generation makes **typing the headline feature**:

- **CUE** — schemas are first-class values; `#Deployment` is both a type and a value; unification enforces constraints structurally. `cue vet` / `cue eval` validate before anything ships **(verified)**.
- **Dhall** — full static type system (records, unions, functions, `Natural`/`Text`/`Bool`); type errors at evaluation time **(verified)**. **Pkl** — typed classes with constraint types (`Int(1, 65535)`, regex via `matches`) plus custom validation functions **(verified)**.
- **Nix** — dynamically typed language; the NixOS module system gives every *option* a declared type and validation, pushing errors to evaluation time **(verified)**. **Jsonnet** — deliberately untyped; validation is external (JSON Schema, tests): fastest authoring, weakest safety net.

The schema trend is also arriving in the old guard: Terraform HCL2 has typed variables + `validation` blocks; Pulumi inherits the host language's types. Typed configuration is the single clearest direction of the 2024-2026 era (§11.1).

### 8.2 The Hermeticity: The Pure

**Hermetic** configuration is sealed against its environment: the same input yields the same output, anywhere, anytime. **Pure** means no side effects during evaluation — no I/O, no network, no time, no hidden state.

- **Nix** is the gold standard: sandboxed builds with no network, pinned inputs (flake locks, hashed store paths), outputs addressed by input hash — reproducibility *enforced*, not hoped for **(verified)**.
- **Dhall** is pure and total: no I/O, always terminates; imports are content-addressed and hash-pinnable **(verified)**. **Jsonnet** is side-effect free by design **(verified)**.
- **CUE** is pure in evaluation (no side effects; static imports) but *not* total — it can in principle loop, so it ships cycle detection and limits. **Pkl** allows bounded input access (`read` for files, env vars) but supports sandboxed/deterministic execution; it is *less* pure than Nix/Dhall by design — an authoring language, not a build system.

Hermeticity is what makes configuration *testable* and *auditable*: you can diff, cache, and reproduce it.

### 8.3 The Import and the Abstraction

Every useful configuration language needs to compose:

- **Imports** — split configs across files, and (in Dhall/Nix) across *content-addressed* sources: `import "https://example.com/config.dhall" sha256:...`, `fetchurl`/flake inputs in Nix, `import "k8s.cue"`, `import "nginx.pkl"`, `local ... in ...` for Jsonnet.
- **Abstraction** — functions and parameterized units: Ansible *roles* and *collections*; Terraform *modules*; Puppet *defined types*; CUE *definitions* (`#Deployment`); Pkl *classes* and *templates*; Nix *functions* over derivations and *modules*; Jsonnet *functions* and *inheritance*.

The design trade: **DSL abstractions are safe but limited** (Terraform modules are parameterized templates, not code); **general-purpose abstractions are powerful but must be self-policed** (Chef libraries, Pulumi classes); **language-level abstractions** (CUE/Dhall/Pkl/Nix) aim for the middle — rich enough to factor config, constrained enough to stay pure.

### 8.4 The Design Table

| Design aspect | Languages | Notes |
|---|---|---|
| **Static/structural typing** | CUE, Dhall, Pkl, (NixOS options) | Errors at authoring time; schemas reusable as validation |
| **Untyped data generation** | Jsonnet, YAML, JSON, HCL (mostly) | Fast authoring; runtime/deploy-time surprises |
| **Hermetic/pure evaluation** | Nix, Dhall, Jsonnet, CUE (pure, bounded) | Deterministic output; reproducible; auditable |
| **Sandboxed builds** | Nix | Store paths by hash; no network in builds; rollback via generations |
| **Content-addressed imports** | Nix (flakes), Dhall | Pin-by-hash — supply-chain verifiable config |
| **First-class schemas** | CUE | Schema = data = validation; unification instead of templating |
| **Classes/templates** | Pkl, (CUE definitions) | Typed reusable units with defaults and constraints |
| **Functions/abstraction** | All (modules, roles, imports, inheritance) | The dimension every language must solve |

---

## 9. Comparison and Selection Guidance

### 9.1 The Master Table

| Language | Tool | Paradigm | Typing | Ecosystem | Best for |
|---|---|---|---|---|---|
| **YAML + Jinja2** | Ansible | Declarative-ish, ordered tasks, agentless push | Untyped | Galaxy, AAP/AWX, thousands of modules | Server/OS config across many hosts, app deploys, network devices; lowest barrier to entry |
| **Puppet DSL** | Puppet | Declarative resource model, agent pull | Resource-model typed (no arbitrary types) | Forge, PE, Hiera | Large fleets needing scheduled convergence; compliance-heavy shops |
| **Ruby (DSL)** | Chef | Declarative resources in full language, agent pull | Ruby types | Supermarket, InSpec, Test Kitchen | Teams that want code power; legacy enterprise estates |
| **YAML SLS** | SaltStack | Declarative states, master/minion, event-driven | Untyped | Salt modules, SaltCloud | High-scale parallel event-driven config; cloud orchestration |
| **HCL/HCL2** | Terraform, OpenTofu | Declarative desired state, plan/apply, state-tracked | Typed vars + validation | Terraform Registry (2,000+ providers), Terragrunt, tfsec | Provisioning cloud infra, multi-cloud, standard IaC |
| **TypeScript/Python/Go/C#/Java** | Pulumi | Declarative engine, code-authored | Host-language types | Pulumi registry, SDKs | Engineering teams wanting code-level testing and reuse in IaC |
| **YAML/JSON templates** | CloudFormation | Declarative stacks, AWS-managed | Template validation | cfn-lint, CDK on top | AWS-native shops; managed, tool-less IaC |
| **Jsonnet** | Tanka, ksonnet, jsonnet CLI | Pure data templating, inheritance | Untyped | Grafana Tanka, Prometheus/Grafana dashboards | Large generated k8s/dashboard configs; DRY without a schema |
| **CUE** | Timoni, Dagger, cue CLI | Constraint-based; schema=data=validation | Structural typing | timoni.sh, Dagger, Kubernetes OpenAPI import | Type-safe k8s manifests, schema-first config, policy-like validation |
| **Dhall** | dhall CLI, dhall-to-yaml | Total functional, programmable | Strong static typing | dhall-haskell/dhall-lang | Config you can prove terminates; untrusted-import safety |
| **Pkl** | pkl CLI, Gradle/Maven/Spring plugins | Typed classes to data | Static + constraints | apple/pkl, JetBrains plugins, Spring Boot ext | Replacing hand-written YAML with typed, validated, templated config |
| **Nix** | Nix, NixOS, nixpkgs | Pure functional derivations, hermetic | Dynamic (typed module options) | nixpkgs, flakes, Home Manager, devenv | Reproducible dev environments, full-system config, CI caches |

### 9.2 The Selection Guidance: Which Language for Which Need

The honest selection rule: **the language follows the job, and the ecosystem follows the language.** Concretely:

1. **"I manage servers and deploy apps across a fleet, and I want it simple."** → Ansible (YAML). The default choice for heterogeneous estates; no agents; the largest module library; the easiest to hire for. Accept: no types, no schedule (run it from cron/CI/AWX).
2. **"I need scheduled, self-healing convergence on thousands of nodes."** → Puppet (or SaltStack for event-driven parallelism). Puppet's catalog model is convergence-by-design; the DSL keeps configs safe. Accept: DSL learning curve, agent lifecycle.
3. **"I provision cloud infrastructure and need multi-cloud, plan/apply, and a huge module ecosystem."** → Terraform/OpenTofu (HCL). Still the default IaC answer; the Registry is the moat. Accept: HCL is another language to learn; state must be managed well (see terraform_state_conflicts.md).
4. **"My team already writes TypeScript/Python and wants tests and real code structure in IaC."** → Pulumi. Same declarative guarantees, no DSL tax; strength is code-level power, weakness is that power (same discipline problem as Chef).
5. **"I'm AWS-only and want zero extra tooling and managed state."** → CloudFormation (YAML), or CDK if you want code authoring.
6. **"Kubernetes manifests are drowning me in copy-pasted YAML."** → Jsonnet (fast, no schema — Tanka) or CUE (schema-first, validation — Timoni). Pick CUE if you want the type safety; Jsonnet if you want minimal ceremony.
7. **"I want my configs validated by their own language, with schemas that ARE the config."** → CUE (constraint-based) or Pkl (class-based). CUE for k8s/OpenAPI-adjacent worlds; Pkl for JVM/Apple toolchains and simple typed data.
8. **"I need reproducible builds, dev environments, and full-system configuration that rolls back."** → Nix/NixOS. The price: the hardest language in this guide; the reward: hermeticity nothing else matches.
9. **"I must safely import config from untrusted sources and prove termination."** → Dhall. Rarely the team's first choice; the principled choice.

### 9.3 The Selection Table

| If your need is... | Start with | Because |
|---|---|---|
| Server config / app deploy, simple, no agents | Ansible (YAML) | Lowest barrier; largest module ecosystem; agentless |
| Scheduled self-healing at fleet scale | Puppet (DSL) / SaltStack (YAML) | Convergence-by-schedule is the core model |
| Multi-cloud provisioning, standard IaC | Terraform / OpenTofu (HCL) | De-facto standard; Registry; plan/apply |
| Code-level IaC with tests | Pulumi (TS/Python/Go) | General-purpose languages + declarative engine |
| AWS-native, managed, tool-less | CloudFormation (YAML) | Built into AWS; managed state; CDK optional |
| K8s config DRY without schema | Jsonnet (+ Tanka) | Pure templating; minimal ceremony |
| K8s config with validation | CUE (+ Timoni) | Schema = config; OpenAPI import; vet before apply |
| Typed config replacing YAML | Pkl | Classes, constraints, multi-format export, Apple-grade tooling |
| Reproducibility / hermetic builds | Nix / NixOS | Purity and pinning are structural |
| Safe untrusted imports, provable config | Dhall | Total, typed, content-addressed |
| Policy and compliance gates | CUE/Dhall validation + OPA/Rego | See audit_as_code_guide.md — validate at config time and admission time |
| GitOps delivery | Any declarative source + Argo CD/Flux/Kargo | See kargo_gitops_guide.md — Git is the state, diff is the loop |

---

## 10. Worked Example: The Same Config in Four Languages

### 10.1 The Scenario

Deploy an **nginx reverse proxy** on a Linux host: TLS-enabled server block on port 443, `server_name example.internal`, proxying to a backend on `127.0.0.1:8080`. The same intent, expressed in Ansible (YAML), Terraform (HCL), CUE, and Pkl — each in the shape its ecosystem expects. This is deliberately *not* the same code; it is the same *desired state*, written where each language is strongest.

### 10.2 The Ansible (YAML playbook)

Ansible owns the *inside of the machine*: install the package, render the config from a Jinja2 template, manage the service.

```yaml
---
- name: Configure nginx reverse proxy
  hosts: webservers
  become: true
  vars:
    server_name: "example.internal"
    backend_port: 8080
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
    - name: Render server block
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/example
        mode: "0644"
      notify: reload nginx
    - name: Enable site
      ansible.builtin.file:
        src: /etc/nginx/sites-available/example
        dest: /etc/nginx/sites-enabled/example
        state: link
    - name: Ensure nginx running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
  handlers:
    - name: reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

Plus `nginx.conf.j2` (Jinja2):
```jinja2
server {
  listen 443 ssl;
  server_name {{ server_name }};
  location / { proxy_pass http://127.0.0.1:{{ backend_port }}; }
}
```

### 10.3 The Terraform (HCL)

Terraform owns the *infrastructure around the machine* — the instance, network, and bootstrap. It is honest here that Terraform does *not* manage in-OS file state (that is Ansible's or cloud-init's job — and that boundary is exactly the lesson of §10.7):

```hcl
variable "server_name" {
  type    = string
  default = "example.internal"
}

variable "backend_port" {
  type    = number
  default = 8080
  validation {
    condition     = var.backend_port > 0 && var.backend_port < 65536
    error_message = "backend_port must be a valid TCP port."
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
  user_data     = templatefile("user_data.sh", {
    server_name  = var.server_name
    backend_port = var.backend_port
  })
  tags = { Name = "nginx-web", Service = "nginx" }
}

output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

`user_data.sh` (cloud-init bootstrap — note the embedded heredoc):
```bash
#!/bin/bash
set -euo pipefail
apt-get update && apt-get install -y nginx
cat > /etc/nginx/sites-available/example <<'NGINX'
server {
  listen 443 ssl;
  server_name example.internal;
  location / { proxy_pass http://127.0.0.1:8080; }
}
NGINX
ln -sf /etc/nginx/sites-available/example /etc/nginx/sites-enabled/example
systemctl enable --now nginx
```

### 10.4 The CUE (CUE)

CUE expresses the *schema and the data together*, and can validate the generated config against the nginx shape before anything runs:

```cue
package nginx

// The schema: any value that unifies with #ServerBlock is a valid block.
#ServerBlock: {
	serverName: string
	backendPort: int & >=1 & <=65535
	listen: *443 | int
	tls: *true | bool
}

server: #ServerBlock & {
	serverName: "example.internal"
	backendPort: 8080
}

staging: server & { backendPort: 9090 }
```

`cue vet` against the definitions fails at authoring time on a typo like `backendPort: "8080"`. `cue export` emits JSON/YAML for the templating layer that renders the nginx file — CUE is the *validation and composition* layer, not the executor.

### 10.5 The Pkl (Pkl)

Pkl is a typed, class-based authoring language that renders to plain data:

```pkl
module nginx

class ServerBlock {
  serverName: String
  backendPort: Int(1, 65535)
  listenPort:  Int = 443
  tls:         Boolean = true

  function render(): String = """
    server {
      listen \(listenPort) ssl;
      server_name \(serverName);
      location / {
        proxy_pass http://127.0.0.1:\(backendPort);
      }
    }
  """
}

server: ServerBlock = new {
  serverName = "example.internal"
  backendPort = 8080
}
```

`pkl eval -f yaml nginx.pkl` emits YAML for consumption by Ansible, Helm, or any templating stage; `Int(1, 65535)` rejects invalid ports at evaluation time. Pkl can also output the rendered nginx config directly via the `render()` function — a neat demonstration of "config-as-code that produces its own artifact."

### 10.6 The Comparison: Trade-Offs

| Language | What it owns | Strengths | Trade-offs |
|---|---|---|---|
| **Ansible (YAML)** | In-machine state (packages, files, services) | Agentless; idempotent modules; huge ecosystem; readable by non-programmers | Untyped; ordering implicit in task list; no schema/validation; drift only when run |
| **Terraform (HCL)** | Cloud infrastructure (instances, networks, DNS) | Plan/apply diff; state-tracked; multi-provider; validation blocks | Wrong tool for in-OS file state (needs user_data/Ansible); state management burden |
| **CUE** | Schema + composition of config data | Schema = data; unification over templating; vet before apply; OpenAPI interop | No executor — needs a consumer (Timoni, templating, scripts); community smaller than YAML's |
| **Pkl** | Typed authoring of config data | Classes, constraints, multi-format export, render-to-artifact; JVM tooling | Young ecosystem (2024+); less suitable for large graph-of-resources IaC |

The deep lesson of the table: **each language is the right answer for a different slice of the same problem.** YAML says "what," HCL says "what + where in the cloud," CUE/Pkl say "what, validated, composed, and generated." Mature platforms compose them — Terraform provisions, Ansible configures, CUE/Pkl generate and validate the data both consume, and OPA/Rego polices the result (see [audit_as_code_guide.md](audit_as_code_guide.md)).

### 10.7 The Lessons: Choose the Language for the Need

1. **Match the language to the layer.** Cloud object → HCL/CloudFormation/Pulumi. In-OS file+service state → Ansible/Puppet/Chef/Salt. Config *data* with validation → CUE/Pkl/Dhall. Whole-system reproducibility → Nix. Fighting the layer match is the #1 source of "this tool is terrible" opinions.
2. **The untyped tax is real.** YAML/JSON accept anything; a typo is a failed deploy at 2 a.m. If your config is large or safety-critical, a typed generation layer (CUE/Pkl) pays for itself in authoring-time errors.
3. **Validation belongs as early as possible.** The further left you move checks — schema in the language (CUE/Pkl), `terraform plan` gates in CI, OPA admission policies at deploy — the cheaper every mistake becomes. See [audit_as_code_guide.md](audit_as_code_guide.md).
4. **Hermeticity is a feature you notice after the first incident.** Reproducible, hash-pinned config (Nix flakes, Dhall imports) turns "why is prod different?" into a non-question.
5. **Heterogeneity is normal.** Production estates in 2026 routinely run Ansible + Terraform + CUE (or Jsonnet) + a GitOps controller simultaneously. The languages interoperate through *data*: YAML/JSON out of CUE/Pkl, state out of Terraform, facts out of Ansible.

---

## 11. The Future (2026+): Typed Config, Policy-as-Code, GitOps

### 11.1 The Trend: Typed Configuration (CUE/Pkl Adoption)

**(verified)** The 2024-2026 period has moved from "typed config is interesting" to "typed config is the default answer to YAML hell." Evidence, flagged where approximate:

- **CUE** — engine of Timoni (CNCF sandbox; "package manager for Kubernetes powered by CUE"), historically core to Dagger's early design; used internally by major cloud vendors for schema-first config; the `cue` CLI became a standard k8s validation tool. Strongest momentum of the new-gen set.
- **Pkl** — Apple's production pedigree, Apache 2.0 licensing, and day-one tooling (CLI, IntelliJ plugin, Gradle/Maven/Spring Boot integrations) drove fast enterprise entry, concentrated in JVM toolchains and config-heavy services (2024-2026, approximate).
- **Nix** — continued growth via flakes, devenv, and CI caching; the reference implementation of the reproducibility half of the trend. **The incumbents absorbed typing** — Terraform validation blocks, Pulumi's host-language types, cfn-lint all push validation earlier. Direction of travel: *configuration is becoming a typed discipline*.

### 11.2 The Trend: Policy-as-Code

Configuration languages and policy engines are converging. CUE/Dhall/Pkl validate *at config time* ("this manifest is malformed"); OPA/Rego, Kyverno, and InSpec validate *at admission and runtime* ("this manifest violates policy — no `latest` image tags in prod"). The two layers compose: typed configs produce well-formed inputs, policy engines constrain them. This series' dedicated treatment is [audit_as_code_guide.md](audit_as_code_guide.md) — the short version: expect the schema of your config language (CUE especially) and the rules of your policy engine to grow together, because both are just *constraints on data*.

### 11.3 The Trend: GitOps

GitOps (see [kargo_gitops_guide.md](kargo_gitops_guide.md)) makes Git the source of truth and a controller the convergence engine: the declarative config in the repo *is* the desired state, and reconciliation is drift correction running forever. Its demands shape the languages:

- **Diffability** — controllers diff live state vs Git; typed, normalized configs (CUE/Pkl/Jsonnet output) diff cleanly, hand-edited YAML does not.
- **Reproducibility** — pinned, hermetic config generation (Nix-style thinking, even outside Nix) makes the Git history an honest audit trail.
- **Validation before merge** — the config languages' schemas become CI gates in the GitOps PR loop.

The synthesis of all three trends: **schema-validated, policy-gated, Git-reconciled, typed configuration** — generated by a language (CUE/Pkl/Nix/Jsonnet), policed by policy-as-code, delivered by GitOps.

### 11.4 The Trends Summary Table

| Trend | Driver | Languages/tools | Status (2026, approximate) |
|---|---|---|---|
| **Typed configuration** | YAML hell, k8s scale, earlier validation | CUE, Pkl, Dhall; Terraform validation; Pulumi types | CUE strongest in k8s tooling; Pkl fast enterprise entry; direction unambiguous |
| **Schema-first authoring** | Config as data with contracts | CUE, Pkl, OpenAPI interop | Standard practice in new k8s tooling (Timoni); growing in JVM |
| **Policy-as-code** | Compliance gates, security | OPA/Rego, Kyverno, InSpec + CUE/Dhall validation | Converging with config languages; see audit_as_code_guide.md |
| **GitOps delivery** | Git-as-truth, continuous convergence | Argo CD, Flux, Kargo, OpenTofu | Default delivery model for k8s; expanding to infra; see kargo_gitops_guide.md |
| **Hermetic/reproducible builds** | Supply-chain security, reproducibility | Nix, flakes, devenv, Dhall hash-pinning | Nix growth continues; reproducibility becomes a procurement requirement |
| **AI-assisted config authoring** | Copilots generating manifests | CUE/Pkl/Jsonnet as *validation targets* for generated config | Early; typed languages make AI output checkable — a strong 2025+ theme |

### 11.5 The Summary: Configuration as a Language

One page, the whole arc:

1. **Configuration management began as scripts (imperative) and matured into desired-state tools (declarative)** — Puppet's DSL (2005) proved a constrained language could make config safe; Ansible's YAML (2012) made it simple; Terraform's HCL (2014) made infrastructure declarative; Chef and Pulumi proved general-purpose languages could drive declarative engines.
2. **The 2020s added a new layer: configuration *languages* proper** — Jsonnet (2014) killed duplication, CUE (2018) made schema and data one thing, Pkl (2024) brought industrial-grade typed authoring, Dhall (2016) brought provable config, and Nix (2003) remains the north star of hermetic reproducibility.
3. **The design axes that separate them are few and knowable**: declarative vs imperative; typed vs untyped; pure/hermetic vs ambient; DSL vs general-purpose; executor vs generator. Every tool is a point in that space.
4. **The operating principle for practitioners**: choose the language for the *layer* you're configuring, push validation as far left as possible, keep configs hermetic and diffable, and let the tools interop through data (YAML/JSON/state files) rather than trying to make one language do everything.
5. **The final word — configuration as a language**: configuration stopped being a file format and became a language discipline: typed, composable, pure where it matters, validated before it ships, reconciled forever. The tools will keep churning; the languages — and the design ideas behind them — are the durable asset. Learn the ideas; the syntax follows.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **Configuration management (CM)** | The practice of managing the state of servers and software (packages, files, services, users) automatically and reproducibly. |
| **CM** | Abbreviation for configuration management. |
| **Infrastructure as code (IaC)** | Managing cloud infrastructure (networks, VMs, storage, serverless) through machine-readable definition files instead of manual console operations. |
| **IaC** | Abbreviation for infrastructure as code. |
| **Declarative** | A config style that states the desired end state and lets the tool compute the steps (e.g., "nginx is running"). |
| **Imperative** | A config style that lists the exact steps to execute in order (e.g., "install nginx; edit config; restart service"). |
| **Desired state** | The specification of how a system should be, which declarative tools converge reality toward. |
| **Idempotency** | The property that applying an operation multiple times has the same effect as applying it once; re-running configs is safe. |
| **Convergence** | The process of driving actual state toward desired state, correcting drift. |
| **Drift** | The gap between actual system state and the declared desired state (manual changes, hotfixes, time). |
| **Ansible** | Agentless CM tool (2012, Michael DeHaan; Red Hat 2015) using SSH and YAML playbooks with idempotent Python modules. |
| **YAML** | "YAML Ain't Markup Language" — a human-readable data-serialization format; the substrate of Ansible playbooks, SaltStack SLS, CloudFormation, and Kubernetes manifests. |
| **Playbook** | An Ansible YAML document defining plays (hosts) and tasks (module invocations). |
| **Module** | A self-contained Ansible unit (usually Python) that performs one idempotent action and reports changed/ok/failed. |
| **Puppet** | Declarative CM tool (2005, Luke Kanies) with a purpose-built DSL; agent pull, resource-graph catalog. |
| **DSL** | Domain-specific language — a language designed for one domain (e.g., the Puppet language, HCL) rather than general programming. |
| **Chef** | CM tool (2009, Opscode) where configuration is Ruby code (recipes/cookbooks) over a declarative resource layer. |
| **Recipe** | A Chef configuration file: Ruby code declaring resources to converge on a node. |
| **SaltStack (Salt)** | CM and orchestration tool (2011, Thomas Hatch) using YAML SLS state files over a master/minion (or SSH) transport. |
| **Terraform** | IaC tool (2014, HashiCorp; OpenTofu fork 2023) with plan/apply, providers, and state files. |
| **HCL** | HashiCorp Configuration Language — Terraform's purpose-built declarative config language (2014; HCL2 with Terraform 0.12, 2019). |
| **HashiCorp Configuration Language** | Full name of HCL; also used by Vault, Consul, Nomad, Packer. |
| **Provider** | A Terraform plugin translating the resource model into a specific API (AWS, Azure, GCP, Kubernetes, ...). |
| **Resource** | A declared infrastructure object in Terraform/CloudFormation/Pulumi (e.g., `aws_instance.web`). |
| **State** | The record (file or backend) of what Terraform/Pulumi currently manages; input to the plan diff (see terraform_state_conflicts.md). |
| **Pulumi** | IaC tool (2017/2018) letting users write infrastructure in general-purpose languages (TypeScript, Python, Go, C#, Java). |
| **CloudFormation** | AWS's native IaC service (2011) using YAML/JSON templates managed as stacks. |
| **CUE** | "Configure Unify Execute" — constraint-based configuration language (2018, Marcel van Lohuizen) where schema and data are unified. |
| **Jsonnet** | Google's pure data-templating language (open-sourced 2014) — JSON with functions, inheritance, and imports. |
| **Dhall** | Programmable, total (guaranteed termination), strongly typed configuration language (2016, G. Gonzalez). |
| **Pkl** | Apple's typed configuration-as-code language (open-sourced Feb 2024, Apache 2.0, pronounced "Pickle"). |
| **Nix** | Pure, lazy, functional package/build language (2003, Eelco Dolstra) powering the Nix package manager. |
| **NixOS** | Linux distribution built entirely from Nix modules; the whole system is one declarative, reproducible config. |
| **Typed configuration** | Configuration with schemas/types so errors are caught at authoring time (CUE, Dhall, Pkl; HCL validation blocks). |
| **Schema** | A formal description of allowed config shape (fields, types, constraints) — reusable for validation and codegen. |
| **Validation** | Checking config against a schema/policy before (or during) use. |
| **Hermetic** | Sealed from the environment: same inputs produce same outputs anywhere (Nix builds, pinned imports). |
| **Pure** | Free of side effects during evaluation: no I/O, no network, no hidden state (Nix, Dhall, Jsonnet). |
| **Import** | Bringing config from another file or content-addressed source into the current config. |
| **Abstraction** | Parameterized, reusable config units (roles, modules, classes, definitions, functions). |
| **Policy-as-code** | Expressing compliance/security rules as code and enforcing them at config or admission time (OPA/Rego, Kyverno, InSpec — see audit_as_code_guide.md). |

---

*End of guide. Related series: [terraform_state_conflicts.md](terraform_state_conflicts.md) · [audit_as_code_guide.md](audit_as_code_guide.md) · [kargo_gitops_guide.md](kargo_gitops_guide.md) · [cloud_providers_guide.md](cloud_providers_guide.md) · [finops_guide.md](finops_guide.md) · [container_certificates_guide.md](container_certificates_guide.md)*
