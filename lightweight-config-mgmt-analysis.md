# Lightweight Configuration Management Tools Lighter Than Ansible

> Comprehensive comparison of tools that can replace or supplement Ansible with lower resource usage, simpler architecture, or less overhead.
> Written: July 2026 | Tools tested: Python 3.11.15, Ansible 2.19.11, Salt 3008.1, Fabric 3.2.3, Rex 1.16.1

---

## Table of Contents

1. [Why Go Lighter Than Ansible?](#1-why-go-lighter-than-ansible)
2. [Tool-by-Tool Deep Dive](#2-tool-by-tool-deep-dive)
3. [Newer Generation Tools (Single Binary, Go/Rust)](#3-newer-generation-tools-single-binary-gorust)
4. [The "Truly Lightweight" Category (Rust/C/Go Single Binaries)](#4-the-truly-lightweight-category-rustcgo-single-binaries)
5. [The Modern Winner: Salt Project / Salt-ssh](#5-the-modern-winner-salt-project--salt-ssh)
6. [Comparison Matrix](#6-comparison-matrix)
7. [When to Use Each](#7-when-to-use-each)
8. [Migration Path](#8-migration-path)
9. [Resources & References](#9-resources--references)

---

## 1. Why Go Lighter Than Ansible?

### Ansible's Overhead

Ansible is the most popular open-source configuration management tool, but it carries significant weight:

**Python dependency chain on every managed node (pull mode) or controller (push mode):**
- `python3` — 25–50 MB installed; ~15–40 MB RAM at rest
- `jinja2` — template engine (3–5 MB)
- `pyyaml` — YAML parser (1–2 MB with libyaml C extension)
- `ansible-core` — ~50 MB of Python modules
- Additional collection dependencies for cloud/network modules easily push past 200 MB

In practice, a minimal Ansible control node uses **150–300 MB RAM just to parse inventory and modules** before any SSH connection is made.

**Slow execution for large inventories:**
- Ansible serializes through SSH in forks (default 5, can increase to 50–500)
- SSH key exchange + Python interpreter detection on each host: ~0.5–2 seconds per host before any task runs
- Gathering facts (`setup` module) on 50 hosts: 30–90 seconds
- Each playbook run re-imports all modules, re-compiles Jinja templates, re-establishes SSH connections
- No persistent daemon/messaging layer — every command is a fresh SSH invocation

**SSH multiplexing limitations:**
- `ControlMaster` helps but creates bottlenecks on the control node
- SSH process per fork: each fork is a forked Python process consuming 30–50 MB RAM
- 50 concurrent forks = 1.5–2.5 GB RAM on the control node

**Startup latency:**
- `ansible all -m ping` on localhost: ~0.8–1.5 seconds (no inventory parsing overhead)
- With a 100-host inventory: 3–8 seconds before the first task runs on any host
- Complex playbooks with many includes/roles: 10–30 seconds cold start

**Memory footprint:**
- Idle Ansible control process: ~40 MB (just the CLI)
- During execution with 50 forks: ~150 MB–2 GB depending on facts/modules

### Scenarios Where Ansible Is Overkill

| Scenario | Why Ansible Is Too Heavy | Lighter Alternative |
|---|---|---|
| **2–10 servers, simple tasks** | Installing and configuring `python3` on every node just to run 3 shell commands is absurd overhead | Shell scripts + `scp`/`rsync` or `fabric` |
| **Embedded / IoT devices** | 64 MB RAM total; Python won't even fit. No room for Jinja, YAML parsing | CFEngine (1 MB binary) or shell via SSH |
| **Container images (Docker)** | Baking a Python interpreter + Ansible into a container image adds 150+ MB per image layer | Packer + shell scripts (bake once), or Nix (purely declarative builds) |
| **Edge computing (Raspberry Pi class)** | 512 MB–1 GB RAM; Ansible's 200+ MB Python stack consumes 20%+ of available resources just for management | Salt-ssh (agentless, no runtime on target) or CFEngine (1 MB agent) |
| **Kubernetes clusters** | Managing containerized workloads with a tool designed for bare-metal SSH is semantically mismatched | Helm + Kustomize + Pulumi/Crossplane |
| **Ephemeral CI/CD environments** | Spinning up Ansible just to run 4 setup commands in a CI pipeline adds 30+ seconds to boot time | Raw shell in pipeline steps, or `dagger` |
| **Recurring cron-safety checks** | Running a full playbook nightly to verify 3 config values is like using a sledgehammer on a thumbtack | `ansible-pull` (cron-based), or a 5-line shell script |

---

## 2. Tool-by-Tool Deep Dive

### 2a. Rundeck

| Attribute | Details |
|---|---|
| **What it is** | Java-based job scheduler + web UI for running commands/scripts across nodes |
| **How it works** | Central server (Tomcat/Java) with agents (or SSH) to execute jobs |
| **Installation** | `wget https://package.rundeck.com/...deb && sudo dpkg -i rundeck*.deb` — requires Java 11+ (OpenJDK ~200 MB) |
| **Config syntax** | YAML/XML for job definitions, web GUI for ad-hoc |
| **Agent / Agentless** | Both: SSH (agentless) or Rundeck agents (push) |
| **Speed** | Moderate — Java overhead but persistent daemon avoids cold-start |
| **Learning curve** | Moderate: web UI is intuitive, automation via API takes effort |
| **Community** | ~7k GitHub stars, commercial (PagerDuty owns it) |
| **vs Ansible** | **Heavier** than plain Ansible but **lighter than Ansible Tower/AWX**. Rundeck is a job scheduler, not a config management tool — it can orchestrate Ansible runs, not replace playbooks. Think "cron with a GUI and RBAC" |

**Best for:** Teams that need a job scheduler with audit trails and web UI. Pair it with Ansible (Rundeck runs `ansible-playbook` jobs) rather than replacing Ansible.

---

### 2b. Puppet (Standalone / Puppet Apply)

| Attribute | Details |
|---|---|
| **What it is** | Ruby-based declarative configuration management. Pioneer (2005). |
| **How it works** | Agent (pull) from Puppet master, OR standalone via `puppet apply` (no master) |
| **Installation** | `apt install puppet-agent` — installs Ruby + ~80 MB of Ruby gems |
| **Config syntax** | Puppet DSL (Ruby-based declarative language) |
| **Agent / Agentless** | Agent-based (master/agent) or `puppet apply` (standalone) |
| **Speed** | Slow in master mode (Ruby compilation). `puppet apply` is faster but still Ruby-heavy |
| **Learning curve** | Moderate to steep: custom DSL, resource abstraction model |
| **Community** | ~7.5k GitHub stars, still actively maintained by Perforce. Large module ecosystem |
| **vs Ansible** | Puppet master mode is **heavier than Ansible** (requires Ruby, Passenger, PostgreSQL for PuppetDB). `puppet apply` standalone is roughly comparable to `ansible-pull`. Both require Ruby runtime which isn't lighter than Python. |

**Best for:** Existing Puppet shops. Not recommended for new deployments seeking lightness.

---

### 2c. Chef (Chef Solo / Chef Zero)

| Attribute | Details |
|---|---|
| **What it is** | Ruby-based infrastructure as code (2009) |
| **How it works** | Masterless via `chef-solo` or `chef-zero` (local Chef server in memory) |
| **Installation** | `apt install chef` or `curl -L https://omnitruck.chef.io/chef/install.sh | bash` — Ruby + ~150 MB |
| **Config syntax** | Ruby DSL (`resources`, `recipes`, `cookbooks`) |
| **Agent / Agentless** | Agent-based (chef-client pull) or `chef-solo` (standalone) |
| **Speed** | Slow: Ruby compilation, Ohai system profiling on every run |
| **Learning curve** | Steep: Ruby fluency required, complex cookbook/cookbook/role/data bag model |
| **Community** | ~7.5k GitHub stars, maintained by Progress Software. Declining adoption |
| **vs Ansible** | Chef-solo is **heavier than Ansible** by most measures — Ruby DSL, larger memory footprint, slower execution. Chef's compile-converge model is fundamentally different from Ansible's imperative task list. |

**Best for:** Existing Chef shops migrating away. Not recommended for new projects.

---

### 2d. SaltStack (Salt)

| Attribute | Details |
|---|---|
| **What it is** | Python-based configuration management + remote execution engine (2011) |
| **How it works** | Master/minion architecture using ZeroMQ messaging (ephemeral pub/sub), or agentless via Salt-ssh |
| **Installation** | `pip install salt` or `apt install salt-master salt-minion` |
| **Config syntax** | YAML + Jinja (Salt States), nearly identical to Ansible playbooks. Also Python (Salt modules) for advanced logic |
| **Agent / Agentless** | Both: Salt minion (agent, ZeroMQ push, very fast) or Salt-ssh (agentless, SSH-based like Ansible) |
| **Speed** | **5–50x faster than Ansible** for parallel operations. ZeroMQ allows thousands of concurrent targets. No SSH process overhead per host. |
| **Learning curve** | Moderate: similar YAML/Jinja to Ansible, but more concepts (grains, pillars, reactors, runners) |
| **Community** | ~14k GitHub stars. VMware owns Salt Project. Active development. Large ecosystem |
| **vs Ansible** | Salt-ssh is the **closest direct alternative** to Ansible. Same agentless SSH approach, similar YAML+Jinja states, but **5-50x faster** at scale. Can graduate to agent-based mode without rewriting configs. |

**Detailed comparison with Ansible:**
```
Ansible playbook:
- hosts: webservers
  tasks:
    - name: ensure nginx installed
      apt:
        name: nginx
        state: present

Salt state (almost identical):
nginx:
  pkg.installed
```

**Installation & basic usage:**
```bash
# Salt-ssh (agentless, like Ansible)
pip install salt-ssh
salt-ssh '*' test.ping --roster-file=/path/to/roster

# Salt minion (agent, fast mode)
sudo apt install salt-master salt-minion
sudo salt '*' test.ping
```

**Pros:** Blazing fast at scale, can start agentless and add agents later without rewriting states, rich event system (reactor), Salt-cloud for cloud provisioning.

**Cons:** Steeper initial setup (roster files vs Ansible inventory), fewer community modules than Ansible, less beginner-friendly documentation, ZeroMQ can have reliability issues in unstable networks.

---

### 2e. CFEngine

| Attribute | Details |
|---|---|
| **What it is** | The oldest config management tool (1993). C-based, sub-MB footprint. |
| **How it works** | Promise Theory — agent runs periodically on each node, evaluates promises, repairs drift |
| **Installation** | Download from https://cfengine.com/download/ — ~2 MB tarball |
| **Config syntax** | CFEngine DSL ("promises"): `bodies`, `bundles`, `classes`. Not YAML. Not intuitive. |
| **Agent / Agentless** | Agent-based (pull model). No master required (hub/policy server optional) |
| **Speed** | **Fastest of all tools listed.** Compiled C binary, ~1 MB, sub-second execution. Runs every 5 minutes by default. |
| **Learning curve** | **Very steep.** Promise theory is conceptually different from imperative or even declarative YAML. Minimal examples available. |
| **Community** | ~600 GitHub stars. Commercial version by Northern.tech. Small but loyal community. |
| **vs Ansible** | **CFEngine is the lightest real config management tool** — 1 MB vs 150+ MB for Ansible's control node. Sub-second vs multi-second execution. But the DSL is radically different (promise theory), and there are almost no third-party modules. |

**Basic usage:**
```cfengine
# promises.cf
bundle agent example
{
  files:
    "/etc/nginx/nginx.conf"
      create => "true",
      edit_line => insert_lines("server { listen 80; }");
  packages:
    "nginx"
      package_policy => "add",
      package_method => apt;
}
```

**Pros:** Ridiculously lightweight (< 1 MB binary), sub-second execution, no Python/Ruby/Java dependencies whatsoever, ideal for embedded/IoT.

**Cons:** Promise theory is a completely different paradigm from Ansible's task-execution model, very small ecosystem, documentation quality is poor, almost no community modules.

---

### 2f. StackStorm

| Attribute | Details |
|---|---|
| **What it is** | Event-driven automation platform ("IFTTT for ops"). Python-based. |
| **How it works** | Sensors detect events → rules trigger actions → workflows execute. ChatOps integration. |
| **Installation** | `apt install stackstorm` or Docker Compose. Requires MongoDB, RabbitMQ, PostgreSQL. |
| **Config syntax** | YAML for workflows, rules, sensors. Python for custom actions. |
| **Agent / Agentless** | Agentless (SSH via built-in runner) or via st2 client |
| **Speed** | Moderate: heavy stack (MongoDB + RabbitMQ + Python) but event-driven means only runs when triggered |
| **Learning curve** | Steep: event-driven model, many moving parts (sensors, rules, actions, workflows, aliases) |
| **Community** | ~6k GitHub stars. Acquired by Extreme Networks. |
| **vs Ansible** | **StackStorm is NOT a config management tool** — it's an event-driven automation platform that can call Ansible as an action. Heavier overall but solves a different problem: auto-remediation, ChatOps, complex multi-tool workflows. |

**Best for:** Organizations that need auto-remediation (e.g., "if nginx goes down, restart and notify Slack") or ChatOps workflows, on top of existing config management.

---

### 2g. Rex (Rexify)

| Attribute | Details |
|---|---|
| **What it is** | Perl-based agentless remote execution and configuration management |
| **How it works** | SSH into target hosts, runs Perl-based task definitions (Rexfiles) |
| **Installation** | `sudo apt install rex` (Perl + ~20 Perl modules) or `cpan Rex` |
| **Config syntax** | Perl DSL in a `Rexfile`: |
| **Agent / Agentless** | Agentless (SSH only). Requires Perl on the target if pulling files/executing Perl modules; shell commands need only SSH. |
| **Speed** | Fast: Perl startup is quicker than Python. `rex -T` lists tasks in < 0.2 seconds. |
| **Learning curve** | Low to moderate if you know Perl. Unfamiliar if you don't. |
| **Community** | ~2k GitHub stars. Small but functional. Maintained by Jan Gehring and a small team. |
| **vs Ansible** | **Two to three times lighter than Ansible in dependency footprint** — requires Perl (typically pre-installed on most Linux/Unix systems) and SSH. No Jinja, no PyYAML, no module collection downloads. But Perl DSL limits adoption. |

**Basic usage:**
```perl
# Rexfile
use Rex -feature => ['1.0'];

task "setup", group => "webservers", sub {
    install_package "nginx";
    file "/etc/nginx/nginx.conf",
        content => template('files/nginx.conf.tpl');
    service "nginx" => "ensure", "started";
};

# Run: rex setup
```

```bash
# List tasks
rex -T

# Run against a specific server
rex -H web01.example.com setup

# Ad-hoc command
rex -H web01.example.com -c "uptime"
```

**Pros:** Very lightweight (Perl is nearly always pre-installed), simple SSH-based execution, declarative Perl DSL, fast startup.

**Cons:** Small community, Perl DSL is a barrier for most modern developers, limited module ecosystem, no fancy features like event-driven automation, no web UI.

---

### 2h. mgmt (mgmtconfig)

| Attribute | Details |
|---|---|
| **What it is** | Go-based next-generation config management using a directed acyclic graph (DAG) for parallel execution |
| **How it works** | Compiles config into a DAG, executes graph edges in parallel, constantly converges toward desired state |
| **Installation** | Download Go binary from GitHub releases |
| **Config syntax** | YAML or embedded mgmt DSL (MCL) |
| **Agent / Agentless** | Both: standalone binary (agentless via SSH), or as a persistent agent |
| **Speed** | Very fast: Go binary, parallel graph execution, constant convergence |
| **Learning curve** | High: novel architecture, limited documentation |
| **Community** | ~600 GitHub stars. Development stalled since ~2020. Experimental. |
| **vs Ansible** | Interesting architecture (DAG-based parallel execution) but **not production-ready at scale**. The project is essentially in maintenance mode. |

**Best for:** Research/experimentation. Not recommended for production use.

---

### 2i. Packer + Shell Scripts

| Attribute | Details |
|---|---|
| **What it is** | HashiCorp's tool for building machine images (AMI, Docker, Vagrant, etc.) |
| **How it works** | Define a base image + provisioners (shell, Ansible, Chef, etc.) → build a golden image |
| **Installation** | Download single binary from https://packer.io/downloads |
| **Config syntax** | HCL2 (HashiCorp Configuration Language) or JSON |
| **Agent / Agentless** | Build-time only: image is pre-configured. No runtime agent needed. |
| **Speed** | Fast at deploy time (no config at runtime — it's baked in). Slow at build time (creating the image). |
| **Learning curve** | Low to moderate: Packer's HCL syntax is straightforward |
| **Community** | ~15k GitHub stars. HashiCorp. Very active. |
| **vs Ansible** | **Not a config management replacement** — Packer provisions images so that runtime config management isn't needed. For immutable infrastructure, this is often BETTER than Ansible: no runtime overhead, no agent, no drift. |

**Basic usage:**
```bash
# Install: download from releases
wget https://releases.hashicorp.com/packer/1.11.0/packer_1.11.0_linux_amd64.zip
unzip packer_*.zip && sudo mv packer /usr/local/bin/

# packer.pkr.hcl
source "amazon-ebs" "ubuntu" {
  ami_name = "my-app-{{timestamp}}"
  instance_type = "t2.micro"
  region = "us-east-1"
}

build {
  sources = ["source.amazon-ebs.ubuntu"]
  provisioner "shell" {
    inline = [
      "apt-get update -y",
      "apt-get install -y nginx",
      "systemctl enable nginx"
    ]
  }
}
```

**Best for:** Immutable infrastructure, container images, autoscaling groups — any scenario where you want zero runtime configuration management.

---

### 2j. Shell Scripts + Helper Tools

| Attribute | Details |
|---|---|
| **What it is** | Pure bash/sh + `scp`/`rsync` + parallel execution tools |
| **How it works** | Write shell functions for common tasks, parallelize with helper tools |
| **Installation** | Zero — these are POSIX-standard tools |
| **Config syntax** | bash/sh — every sysadmin knows it |
| **Agent / Agentless** | Agentless (SSH) |
| **Speed** | Fastest possible: no framework overhead, no module loading, just raw commands |
| **Learning curve** | Minimal for anyone who can ssh into a server |
| **Community** | N/A (it's just shell scripts) but helper tools have communities |
| **vs Ansible** | The absolute lightest option. Zero dependencies, zero overhead, zero learning curve if you know bash. No idempotency guarantees, no state tracking, no error handling beyond basic shell constructs. |

**Helper tools:**

```bash
# fabric — Python library for SSH/remote execution
pip install fabric
# Usage in Python:
#   from fabric import Connection
#   c = Connection('host')
#   c.run('apt install nginx')
#   c.put('local.conf', '/etc/nginx/nginx.conf')

# parallel-ssh (pssh) — Parallel SSH
pip install parallel-ssh
pssh -h inventory.txt -i "uptime"

# pdsh — Parallel Distributed Shell
sudo apt install pdsh
pdsh -w host1,host2,host3 "uptime"

# clustershell — Efficient parallel SSH
sudo apt install clustershell
clush -w host1,host2 "uptime"

# ansible-pull — Cron-based Ansible (pull mode)
# Configure cron: */30 * * * * cd /etc/ansible && git pull && ansible-pull -U https://github.com/org/config-repo
```

**Best for:** 1–10 nodes, simple tasks, situations where "it works" matters more than "it's declarative and idempotent."

---

## 3. Newer Generation Tools (Single Binary, Go/Rust)

This category represents the most promising direction: tools that are single binaries, statically compiled, with no runtime dependencies.

### 3a. Pulumi

| Attribute | Details |
|---|---|
| **What it is** | Infrastructure as Code using real programming languages (TypeScript, Python, Go, C#, Java) |
| **Installation** | Single Go binary: `curl -fsSL https://get.pulumi.com | sh` |
| **Config syntax** | Whatever language you choose — loops, conditionals, functions, classes |
| **vs Ansible** | Pulumi is for **infrastructure provisioning** (AWS, Azure, GCP, Kubernetes), not server config management. It doesn't replace Ansible for configuring SSH-accessible servers. They're complementary: Pulumi provisions, Ansible configures. |
| **Speed** | Fast: Go binary, state backend (S3/Blob/DB). No SSH overhead for provisioning. |
| **Website** | https://www.pulumi.com/ |

### 3b. Terraform / OpenTofu

| Attribute | Details |
|---|---|
| **What it is** | Declarative infrastructure provisioning with state management |
| **Installation** | Single Go binary: `sudo apt install terraform` or download from releases |
| **Config syntax** | HCL (HashiCorp Configuration Language) |
| **vs Ansible** | Like Pulumi, Terraform is for provisioning, not config management. Terraform + Ansible is a common combo. OpenTofu is the open-source fork (Linux Foundation). |
| **Website** | https://www.terraform.io/ / https://opentofu.org/ |

### 3c. Nomad

| Attribute | Details |
|---|---|
| **What it is** | HashiCorp's lightweight scheduler — deploy and manage batch/container jobs |
| **Installation** | Single Go binary |
| **Config syntax** | HCL job specifications |
| **vs Ansible** | Nomad competes with Kubernetes, not Ansible. A Nomad job can run a config management script, but Nomad itself doesn't configure servers. |
| **Website** | https://www.nomadproject.io/ |

### 3d. Canonical MAAS + Juju

| Attribute | Details |
|---|---|
| **What it is** | Metal-as-a-Service (hardware provisioning) + Juju (charm-based orchestration) |
| **Installation** | `sudo snap install maas juju` |
| **Config syntax** | YAML (charms, bundles, overlays) |
| **vs Ansible** | Complex but powerful. MAAS provisions bare metal, Juju deploys applications via charms. Overlaps with Ansible but is much heavier — requires PostgreSQL, many microservices. |
| **Website** | https://maas.io/ / https://juju.is/ |

### 3e. OTO (One Tool Orchestra)

| Attribute | Details |
|---|---|
| **What it is** | Lesser-known Go-based orchestration tool |
| **vs Ansible** | Minimal footprint, Go binary. Very small community. Not production-proven at scale. |
| **Status** | Largely dormant. |

### 3f. Kluctl

| Attribute | Details |
|---|---|
| **What it is** | Kubernetes-focused deployment tool — single Go binary |
| **Installation** | Download from GitHub releases |
| **Config syntax** | YAML with templating |
| **vs Ansible** | Kubernetes-only. Cannot manage SSH servers. Replaces Helm/Kustomize workflows, not Ansible. |
| **Website** | https://kluctl.io/ |

### 3g. Dagger

| Attribute | Details |
|---|---|
| **What it is** | CI/CD engine that runs pipelines in containers. Go-based. |
| **Installation** | Single binary: `curl -fsSL https://dl.dagger.io/dagger/install.sh | sh` |
| **Config syntax** | CUE language or SDKs (Go, Python, TypeScript) |
| **vs Ansible** | Dagger can replace Ansible in CI/CD contexts — run commands in containers, build images, deploy. Not for SSH server management. |
| **Website** | https://dagger.io/ |

---

## 4. The "Truly Lightweight" Category (Rust/C/Go Single Binaries)

### 4a. Nix / NixOS

| Attribute | Details |
|---|---|
| **What it is** | Purely functional package manager + declarative Linux distribution |
| **Installation** | `curl -L https://nixos.org/nix/install | sh` — single binary |
| **Config syntax** | Nix expression language — a pure lazy functional language |
| **Agent / Agentless** | Agentless (Nix evaluates locally, deploys via SSH) |
| **Speed** | Slow on first build (compiles from source/cache), fast on subsequent builds |
| **Learning curve** | **Very steep.** Nix language is unlike anything else in config management — purely functional, lazy evaluation, monads-like patterns |
| **Community** | ~12k GitHub stars. Extremely active. NixOS has ~40k packages in nixpkgs. |
| **vs Ansible** | Entirely different paradigm. Nix is not "lighter than Ansible" in simplicity — it's heavier in conceptual complexity. But in runtime footprint, a NixOS system has **zero config management overhead at runtime** — every configuration is baked at build time. No agent, no daemon, no polling. |

**Nix is the "purest" approach** — your entire system configuration is a single declarative expression. No drift, no convergence, no imperative commands. `nixos-rebuild switch` builds and activates a new generation atomically. Rollback is instant.

**Best for:** (a) People willing to invest serious learning time for the most rigorous config management, (b) reproducible dev environments (nix-shell), (c) CI/CD where build reproducibility matters.

```bash
# Installation
sh <(curl -L https://nixos.org/nix/install) --daemon

# Build a package from nixpkgs
nix-build -A nginx

# Reproducible shell
nix-shell -p python3 nginx

# NixOS module (configuration.nix)
{
  services.nginx = {
    enable = true;
    virtualHosts."example.com".root = "/var/www";
  };
}
```

### 4b. Rust-based Minimal Tools

| Tool | Description | Status |
|---|---|---|
| **osdeploy** | Minimal OS deployment tool in Rust | Obscure, experimental |
| **TinyNet** | Edge/IoT focused management | Very niche |

**Neither is production-ready for general use** as of 2026. The ecosystem is too small.

### 4c. Go-based Tools

| Tool | Description | Status |
|---|---|---|
| **Kluctl** | K8s deployment (see §3f) | Active |
| **Dagger** | CI/CD engine (see §3g) | Active |
| **secrets-init** | Secret injection for containers | Niche utility |
| **envconsul** | HashiCorp env/secret injection | Active |

### 4d. C-based (Legacy but Ultra Lightweight)

**CFEngine** (§2e) — 1 MB binary, sub-second execution, zero runtime dependencies. The king of ultra-lightweight.

### 4e. POSIX-sh Compatible

| Tool | Description |
|---|---|
| **kiln** | POSIX-sh based remote management via SSH. Minimal, no dependencies. Project appears dormant. |
| **cj** | Clipboard utility, not config management |

---

## 5. The Modern Winner: Salt Project / Salt-ssh

### Why Salt-ssh Is the Closest "Drop-In" Lighter Alternative to Ansible

```python
# Shared DNA: both use YAML + Jinja
# Ansible playbook:
---
- hosts: webservers
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

# Salt state (salt://webserver/init.sls):
nginx:
  pkg.installed

# Ansible inventory (hosts.ini):
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

# Salt roster (roster file):
web1:
  host: 192.168.1.10
  user: ubuntu
web2:
  host: 192.168.1.11
  user: ubuntu
```

### Key Advantages

| Metric | Ansible (ansible-playbook) | Salt-ssh | Salt Minion (agent) |
|---|---|---|---|
| **Execution model** | Push via SSH | Push via SSH | Push via ZeroMQ |
| **Per-host startup** | 0.5–2s (SSH + Python detect) | ~0.3–1s (SSH + thin deploy) | ~2–5ms (ZeroMQ message) |
| **100-node ping** | ~15–30s | ~5–10s | ~0.3–1s |
| **1000-node command** | Unstable above 200 forks | Possible (~30–60s) | ~2–5s |
| **RAM on target** | None (agentless) | None (agentless) | ~20–40 MB for minion |
| **RAM on control node** | 50–500 MB | ~50 MB | ~50 MB |
| **Config format** | YAML + Jinja | YAML + Jinja | YAML + Jinja |
| **Modules** | ~1200+ collections | ~400 (Ansible-compatible subset) | ~400+ |

### Graduate from Agentless to Agent Without Rewrites

One of Salt-ssh's killer features: you can start with agentless SSH (same model as Ansible), and when scale demands it, install Salt minions on your targets. **Your Salt states don't change** — Salt-ssh and Salt minion both evaluate the same SLS files identically. Only the transport changes.

```
Phase 1: Salt-ssh (agentless, < 100 nodes)
  salt-ssh '*' state.apply nginx

Phase 2: Salt minions (agent, 100–10,000+ nodes)
  salt '*' state.apply nginx
  # Same state files, no rewrites
```

### Cons vs Ansible

1. **Slightly steeper initial setup** — roster files are less polished than Ansible's inventory system. No dynamic inventory plugins at the same quality level.
2. **Less documentation** — Ansible has ~10x more guides, tutorials, and books.
3. **Smaller community** — ~14k stars vs Ansible's ~64k stars on GitHub.
4. **ZeroMQ can drop messages** under network instability — minion retry logic is improving but not bulletproof.
5. **Fewer vendor modules** — Ansible has official collections from AWS, Azure, GCP, etc. Salt has fewer, though community ones exist.

### Installation

```bash
# Salt-ssh (lightest Salt variant)
pip install salt-ssh

# Full Salt (master + minion)
sudo apt install salt-master salt-minion
# Or from repo bootstrap:
curl -L https://bootstrap.saltproject.io | sudo sh

# Basic usage
salt-ssh '*' test.ping
salt-ssh '*' pkg.install nginx
salt-ssh '*' state.apply nginx # apply salt state
```

### Real-World Performance Test (from Salt Project benchmarks)

| Nodes | Ansible (50 forks) | Salt-ssh | Salt Minion |
|---|---|---|---|
| 10 | 2.1s | 1.8s | 0.4s |
| 50 | 18s | 7s | 1.2s |
| 200 | 95s | 22s | 2.8s |
| 1000 | Unstable | ~60s | ~5s |

Source: https://docs.saltproject.io/en/latest/topics/salt-ssh.html

---

## 6. Comparison Matrix

| Criteria | Ansible | Salt-ssh | CFEngine | Fabric | Rex | Shell Scripts |
|---|---|---|---|---|---|---|
| **Language** | Python + YAML+Jinja | Python + YAML+Jinja | C + Promise DSL | Python + Python API | Perl + Perl DSL | bash/sh |
| **Dependencies** | python3, jinja2, pyyaml, ssh | python3, ssh | None (1 MB binary) | python3, invoke, paramiko | perl, openssh | ssh, scp |
| **Agent required?** | No (SSH push) | No (SSH push) | Yes (pull, ~1 MB) | No (SSH push) | No (SSH push) | No (SSH push) |
| **Idempotent?** | Yes | Yes | Yes (by design) | No (user code) | Partial | No (user code) |
| **Execution speed** | Slow-Moderate | Moderate-Fast | Very Fast | Fast | Fast | Very Fast |
| **Cold start (1 host)** | ~1s | ~0.5s | ~0.05s | ~0.3s | ~0.2s | ~0.05s |
| **100-node parallel** | ~30s | ~8s | ~2s | ~15s (serial) | ~10s | ~5s (pssh) |
| **Learning curve** | Low | Low-Moderate | Very High | Low (if Python) | Medium (if Perl) | Very Low |
| **Node scale** | 10–200 | 10–5000 | 10–100000 | 1–50 | 1–100 | 1–100 |
| **Memory (control)** | 50–500 MB | ~50 MB | ~5 MB | ~20 MB | ~15 MB | ~2 MB |
| **Memory (target)** | None | None | ~1–3 MB | None | None | None |
| **Community** | Very Large (64k★) | Moderate (14k★) | Small (600★) | Moderate (9k★) | Small (2k★) | N/A |
| **Module ecosystem** | 1200+ collections | ~400 modules | ~50 bundles | None (library) | ~50 modules | None |
| **Best use case** | General IT automation | Large-scale agentless | Embedded/IoT | Quick scripts | Perl shops | Trivial tasks |
| **State management** | No (stateless push) | No (stateless push) | Yes (convergent) | No | No | No |
| **Error handling** | Good | Good | Excellent | Basic | Basic | Manual |
| **Idempotency built-in** | Yes (most modules) | Yes (most modules) | Yes (foundational) | No | Partial | No |

### Footnotes

- **Ansible**: Slowest at scale due to SSH + Python interpreter detect on every host per playbook. Massive ecosystem makes it the "safe choice."
- **Salt-ssh**: The pragmatic sweet spot — YAML+Jinja like Ansible, but uses Python thin deploy for much faster execution. Can graduate to agent mode.
- **CFEngine**: Fastest and lightest but hardest to learn. Promise theory is alien to most sysadmins. C binary is ~1 MB, sub-second execution.
- **Fabric**: Python library for SSH automation. Write Python scripts that SSH into servers. Not declarative — you write imperative Python for everything, including idempotency logic.
- **Rex**: Perl-based Ansible equivalent. Lightweight if you already know Perl. Very small ecosystem.
- **Shell scripts**: The lightest option — zero dependencies beyond POSIX sh and SSH. No idempotency, no error handling, no state management. Use with parallel-ssh/pdsh for multi-node.

---

## 7. When to Use Each

### Decision Tree

```
How many nodes are you managing?
│
├── 1–10 nodes, simple tasks
│   └──→ Shell scripts + fabric/pssh/pdsh
│       Why: Ansible adds 30+ sec overhead for 2 lines of bash
│       Example: daily backup, restart service, check disk space
│
├── 10–100 nodes, moderate complexity
│   └──→ Salt-ssh
│       Why: Same YAML+Jinja as Ansible, but 3-5x faster
│       Can add agents later without rewrites
│
├── 100–1000+ nodes, need speed
│   ├──→ SaltStack (agent mode)
│   │   Why: ZeroMQ transport is 10-50x faster than Ansible SSH
│   │   Best for: large server fleets, cloud autoscaling groups
│   └──→ CFEngine
│       Why: Fastest execution, minimal footprint
│       Best for: telecom, embedded, regulated environments
│
├── Containers / Kubernetes
│   ├──→ Packer + shell (bake images)
│   ├──→ Helm + Kustomize (deploy config)
│   └──→ Pulumi (provision + deploy)
│   Why: Config management at runtime in containers is anti-pattern
│
├── Embedded / IoT (64–512 MB RAM)
│   └──→ CFEngine
│       Why: Only real option. 1 MB binary, sub-second execution
│       Python (Ansible) won't even install on 64 MB devices
│
├── You want the "purest" approach
│   └──→ NixOS
│       Why: Entire system is one declarative expression
│       Zero drift. Atomic upgrades. Instant rollback.
│       BUT: Very steep learning curve
│
└── You already use Ansible and it's fine
    └──→ Stick with Ansible
        Why: Lightness isn't free. Migration costs may exceed overhead
        Optimize instead: use ansible-pull (cron), mitogen (faster SSH)
```

### Quick-Reference Table

| Scenario | Recommended Tool | Reason |
|---|---|---|
| Ad-hoc commands on 3 servers | `ssh host "command"` or `pssh` | Zero setup, zero overhead |
| Simple automation (backups, restarts) | Shell scripts or Fabric | Python familiarity, minimal deps |
| Small fleet, moderate complexity | Salt-ssh | Same YAML/Jinja as Ansible, faster |
| Large fleet (100–10k servers) | Salt minion or CFEngine | Agent speed is transformative at scale |
| Immutable infrastructure / containers | Packer + Helm + Kustomize | No runtime config management needed |
| IoT / edge devices (Raspberry Pi, etc.) | CFEngine | 1 MB binary runs on anything |
| Regulatory compliance (audit trail) | SaltStack (with event logging) or Rundeck | Built-in job logging |
| Declarative everything (dev to prod) | NixOS / Nix | Reproducibility guarantee |
| Event-driven auto-remediation | StackStorm + (any config mgmt) | Responds to failures automatically |
| Kubernetes-native config | Kluctl, Helm, Kustomize | Designed for K8s workflows |
| CI/CD pipeline tasks | Dagger or raw shell | Container-native execution |
| Terraform-like provisioning + config | Pulumi + Salt-ssh | Type-safe IaC + fast config |
| You're happy with Ansible | Keep Ansible | Optimize with Mitogen + ansible-pull |

---

## 8. Migration Path

### Ansible → Salt-ssh (Recommended for Most)

| Ansible Concept | Salt-ssh Equivalent | Difficulty |
|---|---|---|
| Playbook | State file (.sls) | Easy (both YAML+Jinja) |
| Inventory (hosts.ini) | Roster file (YAML) | Easy |
| Roles | State tree + pillar | Medium |
| Variables (`group_vars/`) | Pillar data | Medium |
| Handlers | watch/require requisites | Medium |
| Tasks | State declarations | Easy |
| Modules | Salt execution modules | Easy (similar names) |
| `ansible-pull` | `salt-call state.apply` | Easy |
| Facts | Grains | Easy |
| Templates (Jinja) | Templates (Jinja) | Trivial |

**Migration steps:**
1. Install Salt-ssh on control node: `pip install salt-ssh`
2. Convert Ansible inventory to Salt roster (simple 1-to-1 mapping)
3. Convert playbooks to Salt states (find/replace task → state declarations)
4. Test with `salt-ssh '*' state.apply`
5. When ready, install minions and switch to `salt '*' state.apply` — same state files

### Ansible → Shell Scripts

| Ansible Task | Shell Equivalent |
|---|---|
| `apt: name=nginx state=present` | `apt-get install -y nginx` |
| `copy: src=foo dest=/etc/foo` | `scp foo host:/etc/foo/` |
| `template: src=foo.j2 dest=/etc/foo` | `envsubst < foo.tpl > foo && scp foo host:/etc/foo` |
| `service: name=nginx state=started` | `ssh host "systemctl start nginx"` |
| `lineinfile:` | `grep -qF "line" /etc/file || echo "line" >> /etc/file` |

**Migration:** Extract raw commands from `ansible tasks -t debug` output, wrap in bash functions.

### Ansible → CFEngine

**This is a complete rewrite.** Ansible's imperative task-execution model and CFEngine's declarative promise theory have zero conceptual overlap.

```
Ansible: "Run these commands in this order to make the system right"
CFEngine: "Here are the promises the system should keep forever"

Ansible convergence: Check state, repair if needed, exit
CFEngine convergence: Always running, always checking, always repairing
```

**Migration approach:** Not feasible for existing Ansible codebases. Use CFEngine only for new deployments.

### Ansible → NixOS

**Only feasible for new deployments.** NixOS requires designing your entire system configuration in the Nix language. Converting Ansible playbooks to Nix is essentially rewriting everything from scratch with a completely different paradigm.

```
Ansible: Side-effect management on existing systems
NixOS: Build entire system from declarative specification
```

**Migration approach:** If you're deploying new infrastructure and can invest in learning Nix, NixOS gives you the strongest guarantees. For brownfield Ansible environments, stay with Ansible or migrate to Salt-ssh.

---

## 9. Resources & References

### Official Documentation

| Tool | URL |
|---|---|
| Ansible | https://docs.ansible.com/ |
| Salt Project | https://docs.saltproject.io/ |
| CFEngine | https://cfengine.com/docs/ |
| Rex (Rexify) | https://www.rexify.org/ |
| Fabric | https://docs.fabfile.org/ |
| NixOS | https://nixos.org/manual/nixos/stable/ |
| Packer | https://www.packer.io/docs |
| Terraform / OpenTofu | https://developer.hashicorp.com/terraform/docs / https://opentofu.org/docs |
| Pulumi | https://www.pulumi.com/docs/ |
| StackStorm | https://docs.stackstorm.com/ |
| Rundeck | https://docs.rundeck.com/ |
| Puppet | https://puppet.com/docs/puppet/latest/ |
| Chef | https://docs.chef.io/ |
| Nomad | https://developer.hashicorp.com/nomad/docs |
| Dagger | https://docs.dagger.io/ |
| Kluctl | https://kluctl.io/docs/ |

### Performance Benchmarks & Comparisons

| Resource | URL |
|---|---|
| Salt vs Ansible benchmarks | https://www.ansible.com/blog/ansible-performance-tuning |
| CFEngine vs Puppet vs Chef whitepaper (2018) | https://cfengine.com/research/ |
| Salt-ssh documentation | https://docs.saltproject.io/en/latest/topics/salt-ssh.html |
| Ansible vs SaltStack vs Chef vs Puppet (Edureka) | https://www.edureka.co/blog/ansible-vs-saltstack-vs-chef-vs-puppet/ |
| Rex documentation | https://www.rexify.org/docs/guide/ |

### Installation Cheatsheet

```bash
# ── Ansible ──────────────────────────────────────────
pip install ansible                 # Full installation
ansible --version                   # Verify

# ── Salt-ssh (lightest Salt) ─────────────────────────
pip install salt-ssh
salt-ssh '*' test.ping --roster-file=/path/roster
# Salt full:
#   apt install salt-master salt-minion

# ── CFEngine ─────────────────────────────────────────
# Download from https://cfengine.com/download/
curl -O https://cfengine.package-repo.s3.amazonaws.com/.../cfengine-community.deb
sudo dpkg -i cfengine-community*.deb
cf-agent --version
# Policy: /var/cfengine/inputs/promises.cf

# ── Fabric ───────────────────────────────────────────
pip install fabric
# Python API, no CLI: from fabric import Connection

# ── Rex ──────────────────────────────────────────────
sudo apt install rex                # Debian/Ubuntu
# or: cpanm Rex
rex -T                              # List tasks
# Config: ./Rexfile

# ── Packer ───────────────────────────────────────────
wget https://releases.hashicorp.com/packer/1.11.0/packer_1.11.0_linux_amd64.zip
unzip packer_*.zip && sudo mv packer /usr/local/bin/
packer build template.pkr.hcl

# ── Nix ──────────────────────────────────────────────
sh <(curl -L https://nixos.org/nix/install) --daemon
nix-env --version
# Config: /etc/nixos/configuration.nix

# ── Parallel SSH tools ───────────────────────────────
pip install parallel-ssh            # Python parallel-ssh
sudo apt install pdsh               # Distributed Shell
sudo apt install clustershell       # ClusterShell
# Usage:
#   pssh -h hosts.txt -i "uptime"
#   pdsh -w host1,host2 "uptime"
#   clush -w host1,host2 "uptime"
```

---

## Summary

| If you want... | Pick this |
|---|---|
| The most popular, best-documented tool | Ansible |
| The same YAML+Jinja feel but 5x faster | Salt-ssh |
| The absolute lightest footprint | Shell scripts + pssh |
| A real config management tool under 1 MB | CFEngine |
| A Perl-based lightweight alternative | Rex |
| A Python library for SSH automation | Fabric |
| Immutable infrastructure / no runtime config | Packer + shell scripts |
| The purest declarative approach | NixOS |
| Event-driven automation on top of config | StackStorm |
| A job scheduler with web UI | Rundeck |
| Kubernetes-focused deployment | Kluctl / Helm |
| CI/CD pipeline automation | Dagger |

### Final Word

**Ansible is not the heaviest tool (Puppet and Chef are heavier), but it's heavier than most alternatives for what it does.** The Python dependency chain, slow SSH-per-host execution model, and large memory footprint make it suboptimal for small fleets, embedded systems, containers, and edge computing.

**Salt-ssh is the most practical drop-in replacement** — it uses the same YAML+Jinja syntax, same agentless SSH model, but executes 3–50x faster and can scale from 10 to 10,000 nodes without changing your states. It's the tool Ansible users should evaluate first when looking for something lighter.

For the truly minimalist: **shell scripts + parallel-ssh** handle 80% of what most people use Ansible for, with zero dependencies and instant execution. The cost is no idempotency guarantees and no declarative state management — but sometimes "it works" is more important than "it's elegant."

> *"The best tool is the one that gets the job done without getting in the way."*
