# DevStack — the stacked lab

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — Infrastructure / Private-Cloud series; the dedicated deep-dive on **DevStack**, the scripted single-node OpenStack development environment: what it is and is not, the stack.sh/local.conf mechanics, the all-in-one architecture and plugin model, the release alignment, the use cases from edit-and-restart loops to the CI gate, the operations and troubleshooting reality, the alternatives (PackStack, Kolla-Ansible, TripleO, OpenStack-Helm, OpenStack-Ansible, MicroStack, Sunbeam), and a Cymbal Bank evaluation-lab worked example
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** docs.openstack.org/devstack (official DevStack docs — quick start, configuration, networking, plugins, systemd, Tempest, FAQ, overview, development, hacking); the openstack/devstack repository at opendev.org and its GitHub mirror; releases.openstack.org (release chronology incl. 2026.1 Gazpacho); docs.openstack.org/tempest (Tempest testing project); RDO documentation (PackStack); the kolla-ansible, tripleo-docs, openstack-helm and openstack-ansible projects; cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) (OpenStack IaaS context and worked-example conventions)
> **Last Updated:** September 2026

---

## Table of Contents

1. [The Project — What DevStack Is (and Is Not)](#1-the-project--what-devstack-is-and-is-not)
2. [The History — From the 2010 Launch to the Devon/Stacker Story](#2-the-history--from-the-2010-launch-to-the-devonstacker-story)
3. [The Mechanics — stack.sh, local.conf and the Stack Cycle](#3-the-mechanics--stacksh-localconf-and-the-stack-cycle)
4. [The Architecture — The All-in-One Topology and the Service Roster](#4-the-architecture--the-all-in-one-topology-and-the-service-roster)
5. [The Plugin Model — extras.d Hooks and DevStack Plugins](#5-the-plugin-model--extrasd-hooks-and-devstack-plugins)
6. [The Release Alignment — Calendar Versions and Stable Branches](#6-the-release-alignment--calendar-versions-and-stable-branches)
7. [The Use Cases — Development, CI, Testing and Demos](#7-the-use-cases--development-ci-testing-and-demos)
8. [The Operations — Workflows, Logs and Troubleshooting](#8-the-operations--workflows-logs-and-troubleshooting)
9. [The Limitations — Resources and the Not-for-Production Posture](#9-the-limitations--resources-and-the-not-for-production-posture)
10. [The Alternatives — The Deployment-Tooling Ecosystem](#10-the-alternatives--the-deployment-tooling-ecosystem)
11. [The Worked Example — A Cymbal Bank DevStack Evaluation Lab](#11-the-worked-example--a-cymbal-bank-devstack-evaluation-lab)
12. [The Summary — One Script, Many Labs](#12-the-summary--one-script-many-labs)
13. [The Glossary](#13-the-glossary)
14. [Claims Status and Verification Notes](#14-claims-status-and-verification-notes)

### How to Read This Guide

This is the dedicated deep-dive on **DevStack — the scripted, single-node OpenStack development environment** — in the `technology/` infrastructure / private-cloud series. Sibling guides carry adjacent depth and are cross-referenced inline rather than re-derived:

- **The OpenStack IaaS context** — [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) is the sibling head-to-head (Nutanix vs OpenStack) that carries the platform-level context: what OpenStack is, the service model, the distribution landscape (Red Hat, Canonical, Mirantis), and the repo's Cymbal Bank worked-example conventions. This guide goes *down a level*: not OpenStack-the-platform, but DevStack-the-tool that stands up OpenStack on one host for development and testing. Where this guide says "Verified — cross-ref the Nutanix guide," it is leaning on that sibling's already-audited claims ledger (the repository convention for shared facts such as the October 2010 launch).
- **The container-platform sibling** — [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) is the Kubernetes/OpenShift deep-dive; it matters here because the modern OpenStack distribution layer increasingly runs *on* Kubernetes (§10), while DevStack deliberately does not — it is the anti-Kubernetes answer: raw processes under systemd on one host.
- **The dev-environment and CI/CD themes** — [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) carries the platform-engineering canon (internal developer platforms, golden paths, CI/CD). §7.2 condenses the CI cross-reference to a short paragraph instead of re-deriving it.
- **The middleware sibling** — [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) is the integration-platform angle (light cross-ref only).

**Note on verification.** This guide was researched in September 2026 against the primary sources listed in the header (live web access to docs.openstack.org/devstack, the devstack repository, releases.openstack.org, docs.openstack.org/tempest, and the deployment-tooling projects). Claims are marked **Verified** (confirmed against a primary source this pass, or carried from the cross-referenced sibling's verified ledger), **Reported-approximate** (widely reported but not pinned to a primary source), or **Failed-to-verify** (checked and not confirmable this pass) in the claims audit in the final section. Where the task brief's own framing differed from what the primary sources say — the calendar-versioning start date and the "history page" that does not exist — this guide follows the primary sources and says so honestly. The full claims-status table is in the final section, with a dedicated "What Could Not Be Verified" ledger.

---

## 1. The Project — What DevStack Is (and Is Not)

> **DevStack is a series of extensible shell scripts that bring up a complete OpenStack environment on a single host from git master — a development and functional-testing tool, not an installer for production clouds.** (Verified — DevStack docs, project index: *"DevStack is a series of extensible scripts used to quickly bring up a complete OpenStack environment based on the latest versions of everything from git master. It is used interactively as a development environment and as the basis for much of the OpenStack project's functional testing."*)

### 1.1 The One-Sentence Positioning

OpenStack the platform is a collection of loosely coupled cloud services (Nova compute, Neutron networking, Cinder block storage, Glance images, Keystone identity, and friends) that an operator assembles into a private cloud (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3). DevStack is the fastest legitimate way to *see that assembly happen*: one script, `./stack.sh`, on a clean Linux host, and 15–30 minutes later a working all-in-one OpenStack is up, with every service running from a git checkout under `/opt/stack` rather than from packages (Verified — DevStack docs quick start: *"This will take 15 - 30 minutes, largely depending on the speed of your internet connection"*; the FAQ explains the package question: *"Unlike packages, DevStack leaves your cloud ready to develop — checkouts of the code and services running locally under systemd, making it easy to hack on and test new patches"*).

The GitHub README states the goals plainly (Verified — openstack/devstack README): to quickly build dev OpenStack environments in a clean Ubuntu or Rocky Linux environment; to *describe working configurations* (which code branches work together, what their config files look like); to make it easier for developers to dive into OpenStack; to make cross-project feature prototyping easy; and — the CI pillar — *"to provide an environment for the OpenStack CI testing on every commit to the projects."*

### 1.2 The Identity Table

| Aspect | What the primary sources say |
|---|---|
| **What it is** | A set of Bash scripts (plus utilities, libraries and docs) that deploy an OpenStack cloud from git source trees on one host (Verified — repo README) |
| **Primary purpose** | Interactive development environment + the basis for much of OpenStack's functional testing (Verified — docs index) |
| **What it is not** | A general OpenStack installer, a production deployment tool, or a supported distribution (Verified — docs overview and FAQ: *"DevStack is not a general OpenStack installer and was never meant to be everything to everyone"*; *"It makes many choices that are not appropriate for production systems"*) |
| **Language** | UNIX shell script, Bash 4+ required (Verified — contributing docs) |
| **License** | Apache-2.0 (Verified — GitHub repo) |
| **Official repository** | opendev.org/openstack/devstack, mirrored at github.com/openstack/devstack (Verified — docs and repo) |
| **Documentation** | docs.openstack.org/devstack/latest; devstack.org is now a redirect to it (Verified — contributing docs: *"devstack.org (now a redirect to https://docs.openstack.org/devstack/latest/)"*) |
| **Typical host** | A dedicated, disposable VM — the docs warn DevStack *"will make substantial changes to your system"* and the README jokes that it *"runs rampant over the system it runs on"* (Verified) |
| **Supported base OS** | The two latest Ubuntu LTS releases plus current dev release, and the current RHEL-family major (docs name Rocky Linux 9 and openEuler explicitly; Ubuntu 24.04 Noble is called *"the most tested"*) (Verified — quick start and overview) |

### 1.3 The Script Family

The repo keeps the primary scripts at the root (Verified — contributing docs, "Repo Layout"):

| Script / file | Role (all Verified — repo layout, README, configuration docs) |
|---|---|
| `stack.sh` | The primary script; performs the bulk of the deployment work, flowing through project-specific sub-scripts in `lib/` |
| `unstack.sh` | Tears the stack down — stops the services (plugin contract: called *before* other services are shut down) |
| `clean.sh` | Removes state and transient data for a from-scratch re-run; calls `unstack.sh` first (Verified — plugin docs) |
| `local.conf` | The configuration file; a modified INI format with `[[phase|file]]` meta-sections (Verified — configuration docs) |
| `stackrc` | The global configuration file, sourced early; it calls `local.conf` (or legacy `localrc`) so local settings are recognized (Verified — contributing docs) |
| `openrc` | The environment file that sets `OS_*` credentials for the OpenStack CLI (Verified — configuration docs) |
| `lib/` | Project-specific sub-scripts (Keystone, Nova, each service) exposing the `install_/configure_/init_/start_/stop_/cleanup_XXXX` entry points `stack.sh` calls (Verified — contributing docs) |
| `extras.d/` | In-tree dispatch scripts called by hooks in `stack.sh`, `unstack.sh` and `clean.sh` (Verified — contributing docs) |
| `tools/` | Stand-alone helpers, notably `tools/create-stack-user.sh` (creates the stack user) and `tools/info.sh` (records installed package/git versions) (Verified — README and FAQ) |
| `samples/` | Sample configuration files, including `samples/local.conf` (Verified — docs) |

### 1.4 Why a Bank Would Care

DevStack is not a bank product. But every serious OpenStack evaluation — including Cymbal Bank's in §11 — starts by needing a *safe, cheap, throwaway* place to learn the platform's real behavior: its APIs, its failure modes, its operational texture. DevStack is the canonical answer to that need, which is why it appears in every OpenStack family of guides this repository carries. The honest framing, repeated throughout this guide: **DevStack is the learning-and-development layer of the OpenStack world; production is someone else's job** (§9, §10).

---

## 2. The History — From the 2010 Launch to the Devon/Stacker Story

### 2.1 The Platform's Birth: Rackspace and NASA, 2010 (Verified)

OpenStack itself was launched in 2010 by **Rackspace** (the cloud-hosting company) and **NASA** (whose internal Nebula cloud project contributed the compute controller that became Nova; Rackspace's Cloud Files object storage became Swift). The first release, **Austin**, shipped on **21 October 2010** (Verified — cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.1 and its claims ledger, which records the Austin release date against Rackspace history, The Register's launch coverage and NASA's tech-transfer article). Governance eventually moved to the OpenStack Foundation and then, renamed in 2021, the **OpenInfra Foundation** (Verified — cross-ref the same sibling guide).

### 2.2 DevStack's Own Timeline — What the Primary Sources Actually Document

DevStack's history is documented in fragments across the docs and the repository rather than in a single narrative page (see §2.4 for what is *not* documented). The verifiable fragments:

| When | What is verifiable | Source |
|---|---|---|
| **~2011 (Diablo cycle)** | DevStack maintains a stable branch for every OpenStack release *starting with Diablo*: *"a separate branch is maintained for all OpenStack releases starting with Diablo (stable/diablo)"* — the earliest hard evidence of the tool's existence in the project's release machinery | Verified — contributing docs |
| **Early hosting** | The repo was long mirrored under the `cloudbuilders` GitHub org with the banner *"THIS IS A MIRROR OF https://github.com/openstack-dev/devstack"* — a trace of its pre-opendev hosting on GitHub, in the era when Rackspace's cloud-builders tooling lived outside the main OpenStack org | Verified (mirror banner on github.com/cloudbuilders/devstack); the Rackspace team association is **⚠ Reported-approximate** |
| **Oct 2013** | `local.conf` was introduced to simplify configuration: *"Historically DevStack obtained all local configuration and customizations from a `localrc` file. In Oct 2013 the `local.conf` configuration method was introduced (in review 46768)"* | Verified — configuration docs ("Historical Notes") |
| **2015** | The contributing docs (dated 2015 in the built pages) describe the repo layout that still holds today — stack.sh, extras.d, lib, tools | Verified — hacking docs |
| **2020** | The systemd documentation explains the migration from screen: *"By default DevStack is run with all the services as systemd unit files"*; screen was the model *"when the number of services ... was typically < 10"*, but the landscape changed — services under Apache were not stoppable in screen, and there are *"typically at least 20 items"* today | Verified — systemd docs |
| **Ongoing** | *"DevStack started off as an explanation as much as an install script"* — the project's own description of its didactic DNA | Verified — FUTURE.rst in the repo |

A telling code-level trace: the tree still carries inline author-attribution comments (the `dtroyer` handle appears in `lib/tls`) — `dtroyer` is the OpenStack handle of **Dean Troyer**, a long-time DevStack core developer whose shocco fork the docs themselves link for rendering scripts as documentation (**⚠ inference** from code comments plus the docs' own link to github.com/dtroyer/shocco; not a biographical source).

### 2.3 The Devon/Stacker Name Story — Anecdote, Honestly Flagged

The task brief for this guide asked about a "Devon/Stacker" origin story — community lore that the tool's name and character trace to a Rackspace-era developer (the anecdote runs roughly: stack.sh's author was a developer whose handle or story involved "Devon," and "Stack" + a person's name produced "DevStack"; a companion anecdote mentions "Stacker"). **⚠ This is anecdote, not documented history.** This pass found:

- No project-history page in the current or recent DevStack documentation. The URL the brief expected (`docs.openstack.org/devstack/latest/history.html`) returns **404**, as do the same path on the 2024.1 and Zed documentation generations — the page does not exist (❌ Failed-to-verify).
- The OpenInfra wiki page that once carried DevStack history (wiki.openstack.org/wiki/DevStack) is behind an anti-bot proof-of-work wall and could not be read this pass (❌ Failed-to-verify).
- The verified kernel is only what §2.2 records: an early-Rackstack-era tool (cloud-builders mirror), present in the release machinery by the Diablo cycle (2011), designed from the start as *"an explanation as much as an install script."*

Until a primary source (an interview, an early commit message, a Rackspace archive) documents the naming, the "Devon" story should be retold as color, not as fact. The claims audit in the final section records this as **⚠ Reported-approximate / ❌ Failed-to-verify**.

### 2.4 Why the History Matters for the 2026 Reader

The trajectory explains every design choice in this guide: DevStack was born when OpenStack had a handful of services and one developer could hold the whole cloud in their head; it kept its shell-script soul because the script *is* the documentation (*"The script is meant to be read by humans (as well as ran by computers); it is the primary documentation after all"* — Verified, FAQ); and it survived because the project's CI gate — the "check" and "gate" jobs that test every patch to every OpenStack service — still runs on DevStack-shaped environments (§7). The release machinery it now tracks is calendar-versioned and codenamed (§6), but the tool underneath is the same genre of thing Dean Troyer's generation built: readable Bash that stacks a cloud.

---

## 3. The Mechanics — stack.sh, local.conf and the Stack Cycle

### 3.1 The First Run, End to End (Verified — docs quick start and single-machine guide)

The canonical path, straight from the official quick start:

```bash
# 1. Start from a clean, minimal Linux install (Ubuntu 24.04 is "the most tested")
# 2. Create a non-root user with passwordless sudo (the docs' example: stack)
sudo useradd -s /bin/bash -d /opt/stack -m stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo -u stack -i
# 3. Clone DevStack and write the minimal local.conf
git clone https://opendev.org/openstack/devstack
cd devstack
# 4. local.conf with the four passwords preset
#    [[local|localrc]]
#    ADMIN_PASSWORD=secret
#    DATABASE_PASSWORD=$ADMIN_PASSWORD
#    RABBIT_PASSWORD=$ADMIN_PASSWORD
#    SERVICE_PASSWORD=$ADMIN_PASSWORD
# 5. Run it
./stack.sh
```

`stack.sh` must **not** be run as root: it uses `sudo` for privileged tasks but refuses to start as root, and the OpenStack services themselves run unprivileged (Verified — README: *"stack.sh specifically does not run if started as root"*; the `tools/create-stack-user.sh` helper encodes the account expectations). The run takes **15–30 minutes** and ends with a summary screen of URLs, accounts and passwords (Verified — quick start and single-machine guide). What you get: *"keystone, glance, nova, placement, cinder, neutron, and horizon"* installed, floating IPs available, guests with external access, Horizon at `http://<host>/`, and an `openrc` file to source for the CLI (Verified — quick start: *"You can `source openrc` in your shell, and then use the `openstack` command line tool"*).

### 3.2 local.conf — The Configuration Language (Verified — configuration docs)

DevStack configuration lives in `local.conf`, *"a modified INI format file that introduces a meta-section header to carry additional information regarding the configuration files to be changed."* The meta-section header is `[[ <phase> | <config-file-name> ]]` — double brackets, a pipe, no spaces:

| Phase | When it runs during a stack.sh |
|---|---|
| `local` | Extracts the `localrc` content from `local.conf` *before* `stackrc` is sourced — the home of almost all DevStack variables |
| `post-config` | After layer-2 services are configured, before they start |
| `extra` | After services are started, before `extras.d/` files execute |
| `post-extra` | After `extras.d/` files execute |
| `test-config` | After Tempest (and plugins) are configured |

Two syntaxes coexist, and the docs are explicit about the relationship:

- `[[local|localrc]]` is the modern home for DevStack's own variables — it *"provides a default `localrc` file (actually `.localrc.auto`)"*, so all custom settings fit in one file. If a legacy top-level `localrc` file exists, it is used instead for backward compatibility.
- `[[post-config|$NOVA_CONF]]`-style sections write settings *into a service's own configuration file* (the filename is eval'ed in the stack.sh context, so variables like `$NOVA_CONF` work). Example from the docs: `[[post-config|$NOVA_CONF]]` with `[DEFAULT] use_syslog = True`.

Because `localrc` content is *sourced as a shell fragment*, it must obey shell rules — notably **no whitespace around `=`**. The file is processed strictly in sequence, and duplicated settings resolve to the last occurrence. Passwords must be **alphanumeric only** — the docs warn *"some services fail to work when using special characters"* (Verified — configuration docs, repeated in the guides).

### 3.3 The Variables That Matter Most (all Verified — configuration docs)

| Variable | Meaning |
|---|---|
| `ADMIN_PASSWORD`, `DATABASE_PASSWORD`, `RABBIT_PASSWORD`, `SERVICE_PASSWORD` | The four passwords the quick start presets; if unset, `stack.sh` prompts interactively. `DATABASE_*` is the MySQL admin password (default is an inconvenient random hex string); `SERVICE_*` is used by services to authenticate to Keystone |
| `HOST_IP` | The host IP used for API endpoints; normally auto-detected on the first run, but detection can be indeterminate on later runs because the IP moves from an Ethernet interface onto a bridge — set it explicitly to be safe, and `openrc` uses it for `OS_AUTH_URL` |
| `FLOATING_RANGE` | The floating-IP (external) range; default behavior gives a `172.24.4.0/24` range with gateway `172.24.4.1` (Verified — networking docs) |
| `FIXED_RANGE` / `IPV4_ADDRS_SAFE_TO_USE` | The private address space for instances; the default demo project is configured on a subnet from `10.0.0.0/22` (Verified — networking docs). The docs warn the ranges *"must not overlap with any networks in use on the host"* — RFC-1918 overlap is a classic first-run failure |
| `LOGFILE=$DEST/logs/stack.sh.log` | Sends stack.sh output to a file (a timestamp is appended per run); `LOGDAYS` prunes old logs; services themselves log to the systemd journal (Verified — configuration docs, "Logging") |
| `DEST` | The install directory, default `/opt/stack` — where every project repo is cloned (Verified) |
| `RECLONE=yes` | Freshen every project repo on each run instead of only cloning missing ones (Verified — configuration docs; the FAQ frames the default behavior: repos are *"subject to hard resets"* to stay in sync with upstream) |
| `enable_service` / `disable_service` | Turn services on and off, e.g. `disable_service c-vol`, `enable_service q-svc`, `enable_service swift` (Verified — FAQ) |
| `LIBS_FROM_GIT` | Pull client libraries from git instead of released PyPI versions when testing library changes (Verified — configuration and development docs) |
| `*_REPO` / `*_BRANCH` | Point any project at a custom git tree or branch, e.g. `NOVA_REPO=/home/sdague/nova`, `NOVA_BRANCH=fold_disk_config`, or a Gerrit ref like `NOVA_BRANCH=refs/changes/10/353710/1` (Verified — development docs) |

### 3.4 The Stack Cycle: stack.sh / unstack.sh / clean.sh (Verified)

The development rhythm is a three-command cycle:

- **`./stack.sh`** — deploy (or re-deploy, picking up local.conf changes and freshening git trees). Re-running after editing `local.conf` is the standard "re-stack" workflow; the docs' development page recommends `./unstack.sh && ./stack.sh` when iterating on a patch series, noting it *"does take longer per iteration than direct patching, as the whole devstack needs to rebuild."*
- **`./unstack.sh`** — stop the stack's services (the plugin contract describes it as running *before other services are shut down* — Verified, plugins docs).
- **`./clean.sh`** — remove state and transient data for a genuinely clean next run; it calls `unstack.sh` first (Verified — plugin example comment: *"Remember clean.sh first calls unstack.sh"*). The FAQ recommends it for eradicating a broken state: *"in an extreme case, use `clean.sh` to eradicate it and try again."*

**The volatility warning is central to the model** (Verified — FAQ and development docs): every repo under `/opt/stack/<project>` is a git checkout DevStack considers disposable. *"Uncommitted work, or work committed to the master branch, may be overwritten during subsequent DevStack runs."* The corollary the FAQ draws: do not treat `/opt/stack/<project>` as your single master repository — push work to Gerrit (or back it up) before re-running `stack.sh`, or set `RECLONE=no` and restart services manually.

### 3.5 Process Management: systemd Today, screen Yesterday (Verified — systemd docs)

*"By default DevStack is run with all the services as systemd unit files."* Every service runs as a unit named `devstack@<service>.service` (for example `devstack@n-cpu.service` for the Nova compute service, `n-api` for the Nova API, `g-api` for Glance), inside a dedicated `devstack` slice. Systemd wildcards make group operations trivial:

```bash
sudo systemctl status "devstack@*"          # see everything DevStack runs
sudo systemctl restart devstack@n-cpu.service    # restart one service
sudo systemctl restart "devstack@n-*"            # restart all Nova services
sudo systemctl restart devstack@*                # restart the whole stack
sudo journalctl -f --unit devstack@n-cpu.service # follow one service's logs
sudo journalctl -a --unit devstack@n-* | grep <uuid>  # search service logs
```

The **screen** era is history: the screen model fit when a DevStack ran fewer than ten services; with two dozen processes and services hosted under Apache that screen could not stop, systemd won (Verified — systemd docs, "Why this instead of screen?"). Old tutorials still say "attach to screen `stack`" — on a modern DevStack the answer is `systemctl`, and this guide says so plainly. The systemd docs also document the **debugger workflow** used in §7: to break into `pdb` on a systemd-controlled service, stop the unit, read its `ExecStart` (`systemctl show devstack@n-sch.service -p ExecStart`), inject `import pdb; pdb.set_trace()`, and run the command manually (nova-scheduler runs as-is; nova-compute needs `sg libvirt -c '...'`); or install `remote-pdb` and telnet into the session the service opens (Verified — systemd docs, "Debugging").

---

## 4. The Architecture — The All-in-One Topology and the Service Roster

### 4.1 What "All-in-One" Means (Verified — docs overview, guides)

The supported node configurations are *"single node"* plus *"multi-node configurations as are tested by the gate"* (Verified — overview docs). The all-in-one topology — everything on one host — is DevStack's signature: controller services, compute service, network service, database, queue and dashboard all on the same machine, with the compute service using the host's own hypervisor (KVM, or QEMU emulation inside a VM). The docs' guides section offers several flavors, all "All-In-One": **single VM** (run DevStack inside a VM on an existing cloud — *"Use the cloud to build the cloud!"*), **single machine** (bare metal), **LXC container**, plus multi-node lab and KVM-nested-virtualization variants (Verified — guides index). The single-VM guide notes the emulation tax honestly: VMs launched inside such a stack run in QEMU and are slow, *"but their primary use is testing OpenStack development and operation."*

### 4.2 The Service Roster — What Actually Gets Deployed

The quick start's "Profit!" list names what a default install brings up: **keystone, glance, nova, placement, cinder, neutron, and horizon** (Verified — docs index). The overview page's supported-services list is slightly wider and older in flavor — *"Identity (keystone), Object Storage (swift), Image Service (glance), Block Storage (cinder), Compute (nova), Placement (placement), Networking (neutron), Dashboard (horizon)"* — and notably still names **Swift**, which the quick-start roster omits (⚠ Reported-approximate: the two official pages disagree; in current practice Swift, like Heat and Octavia, is enabled by choice, e.g. `enable_service swift`). The roster table, with the roles cross-referenced from the platform sibling:

| Service | Role | Default in DevStack |
|---|---|---|
| **Keystone** | Identity: users, projects, roles, tokens, the service catalog | ✅ core (Verified) |
| **Nova** (+ Placement) | Compute: VM lifecycle; Placement tracks resource inventories | ✅ core (Verified) |
| **Glance** | Image service: the registry of disk images (CirrOS is uploaded by default) | ✅ core (Verified) |
| **Neutron** | Networking: networks, routers, floating IPs, security groups (with OVS + L3 agent) | ✅ core (Verified) |
| **Cinder** | Block storage: volumes attached to instances (LVM-backed by default) | ✅ core (Verified) |
| **Horizon** | The web dashboard | ✅ core (Verified) |
| **Swift** | Object storage (the Rackspace heritage) | ⚠ listed in overview docs; not in the quick-start roster; enable with `enable_service swift` |
| **Heat** | Orchestration (HOT templates) | Optional — external plugin / opt-in |
| **Octavia** | Load-balancer-as-a-service | Optional — the guides carry a dedicated "Configure Octavia" page (Verified — guides index) |
| **Tempest** | The integration test suite — *"installed by default"* | ✅ installed with the stack (Verified — Tempest docs) |

Underneath the OpenStack services sits the support infrastructure DevStack also deploys and manages as "services" in its own accounting: **MySQL** (default database; PostgreSQL is switchable), **RabbitMQ** (the RPC queue), and **Apache** (the WSGI front end for the API services) (Verified — overview and configuration docs). FUTURE.rst is candid that `ENABLED_SERVICES` is *"entirely too overloaded"* because it mixes OpenStack services (n-cpu, g-api) with system backends (mysql, rabbitmq) — a known wart, not a mystery.

### 4.3 The Default Networking — Isolated by Design (Verified — networking docs)

The networking docs' "Defaults" section is the authoritative description of what a no-configuration stack gives you:

- Neutron, including the L3 agent, with **Open vSwitch**;
- **private project networks** for each OpenStack project (network name `private` by default);
- a **floating IP range of `172.24.4.0/24`** with gateway `172.24.4.1` (the external network is named `public` by default);
- the demo project with fixed IPs on a subnet allocated from `10.0.0.0/22`;
- a **`br-ex` bridge** controlled by Neutron for all its networking — *"this is not connected to any physical interfaces"*;
- guest DNS resolution based on the host's `resolv.conf`;
- an **IP masquerade rule** letting guests route out.

The consequence is the sentence that defines DevStack networking: *"This creates an environment which is isolated to the single host. Guests can get to the external network for package updates. Tempest tests will work in this environment."* Guests are reachable from the host; they are not reachable from the LAN unless you connect `br-ex` to a physical interface via `PUBLIC_INTERFACE` (dedicated interface, recommended) — or share the single interface, a configuration the docs flag with a warning: *"because of interactions between OVS and bridging, if you reboot your box with active networking you may lose network connectivity to your system"* (Verified — networking docs).

**The security-group default surprises everyone once**: *"By default all OpenStack environments have security group rules which block all inbound packets to guests."* To SSH or ping a test VM you must open the default group (Verified — networking docs):

```bash
openstack security group rule create --proto icmp --dst-port 0 default
openstack security group rule create --proto tcp --dst-port 22 default
```

The canonical smoke test the docs prescribe: create a keypair, boot a CirrOS server on `private`, allocate a floating IP from `public`, attach it, then `openstack server ssh test-server -- -l cirros` (Verified — networking docs, "SSH access to instances").

---

## 5. The Plugin Model — extras.d Hooks and DevStack Plugins

### 5.1 Two Layers of Extensibility (Verified — plugins and hacking docs)

DevStack's ecosystem breadth comes from two mechanisms that are easy to conflate:

1. **`extras.d/` — the in-tree hook directory.** The repo contains an `extras.d/` directory holding *"the dispatch scripts called by the hooks in `stack.sh`, `unstack.sh` and `clean.sh`"* (Verified — contributing docs, repo layout). Anything shipped inside the DevStack tree can hook the stack lifecycle this way.
2. **DevStack plugins — out-of-tree extension repos.** *"DevStack plugins are bits of bash code that live outside the DevStack tree. They are called through a strong contract, so these plugins can be sure that they will continue to work in the future as DevStack evolves"* (Verified — plugins docs). This is how the ecosystem attaches services DevStack itself does not ship — the docs' overview puts it directly: *"Additional services not included directly in DevStack can be tied in to `stack.sh` using the plugin mechanism to call scripts that perform the configuration and startup of the service."*

FUTURE.rst states the trajectory plainly: the project intends to *"move the bulk of the support code out of devstack itself and into external plugins,"* keeping upstream DevStack focused on the core compute layer (Glance + Nova + Cinder + Neutron core + Keystone).

### 5.2 The Plugin Contract (Verified — plugins docs)

An external plugin is a git repository with a top-level `devstack/` directory containing up to three files:

| File | Role in the contract |
|---|---|
| `override-defaults` | Global variables sourced before the `lib/*` files, letting the plugin override defaults (example from the docs: exporting `CINDER_ENABLED_BACKENDS` to add a plugin-specific storage backend alongside the default LVM one) |
| `settings` | Global variables sourced very early — including the `enable_service` lines the plugin needs (services must be enabled for `run_process` to work) and a `define_plugin <NAME>` line declaring the plugin's name (conventionally the last component of the repo path) |
| `plugin.sh` | The actual plugin, executed at well-defined points of `stack.sh`, `unstack.sh` and `clean.sh` |

A plugin is registered in the `localrc` section of `local.conf`:

```bash
[[local|localrc]]
enable_plugin <NAME> <GITURL> [GITREF]
```

`name` is arbitrary (the docs cite glusterfs, docker, zaqar, congress as examples), `giturl` is any cloneable git URL, and `gitref` — optional, defaulting to master — pins a branch, ref or tag. The docs' example: `enable_plugin ec2-api https://opendev.org/openstack/ec2-api`.

`plugin.sh` is sourced as `source $PATH/TO/plugin.sh <mode> [phase]`, where the **modes** are `stack`, `unstack` and `clean`, and the **stack phases** are `pre-install` (after OS setup, before project source install), `install` (after layer-1/2 sources and dependencies are installed), `post-config` (after layer-1/2 services are configured — *"All configuration files for enabled services should exist at this point"*), `extra` (near the end, after layer-1/2 services have started), and `test-config` (at the very end, to configure Tempest or other test environments). All existing mode/phase points are *"considered strong contracts"* that will not be removed without a reasonable deprecation period. Plugins execute **after** in-tree services at every stage, and multiple plugins run **in order** at each phase — the docs note that a formal dependency mechanism is beyond the current scope, which is why `settings` files may declare `plugin_requires <PLUGIN> <OTHER>` for external tooling even though DevStack itself does not yet act on it. System packages can be declared via bindep files or DevStack's own package-list mechanism (Verified — plugins docs, "System Packages").

### 5.3 What Plugins Are For in Practice

The plugin registry and the guides document the shape of the ecosystem: Octavia configuration, LDAP integration, and the long tail of service plugins (many Tempest plugin packages *"also include DevStack plugin to do things like pre-create required static resources"* — Verified, Tempest docs). For an evaluator the practical takeaway is: **if a service is not in the core roster of §4.2, assume it arrives via `enable_plugin`** — and assume the combination is tested far less than the default (the overview's warning: *"the majority of configuration combinations are rarely, if ever, tested"*).

---

## 6. The Release Alignment — Calendar Versions and Stable Branches

### 6.1 The Naming Regime: Codenames Ended at Zed, Calendar Versions Since 2023 (Verified)

OpenStack shipped codename releases from **Austin (2010)** through **Zed (2022.3, October 2022)** — the full A–Z run. The OpenStack Technical Committee's release-naming reference records that *"The Zed release is the last release for which the OpenStack Technical Committee was involved in the selection of the release name"* (Verified — governance.openstack.org). Starting with **2023.1 Antelope**, releases use **calendar versioning — `YYYY.N`** (two per year, roughly April and October) while *also* carrying alphabetical codenames that restarted at A:

| Release | Date | Notes |
|---|---|---|
| Austin … Zed | 2010 → Oct 2022 | The A–Z codename era; Zed was the last (Verified — governance docs) |
| 2023.1 Antelope | April 2023 | First `YYYY.N` release (Verified — OpenStack charm-guide and runtime references for 2023.1) |
| 2023.2 Bobcat | 4 October 2023 | The "28th version" (Verified — releases.openstack.org and openstack.org) |
| 2024.1 Caracal / 2024.2 Dalmatian | 2024 | Calendar cadence continues (Verified — releases.openstack.org) |
| 2025.1 Epoxy / 2025.2 Flamingo | 2025 | (Verified — releases.openstack.org; the Nutanix sibling's references already record Flamingo 2025.2) |
| **2026.1 Gazpacho** | **1 April 2026** | The **33rd version**; the current supported release at the time of writing (Verified — releases.openstack.org and openstack.org) |
| 2026.2 Hibiscus | due October 2026 | In development; named in OpenStack-Ansible's support-status table as "Under Development" (Verified — docs.openstack.org/openstack-ansible) |

**A correction to a commonly repeated framing:** the task brief for this guide asserted that calendar versioning *"started with the 2023.2 'Bobcat' release."* The primary sources do not support that — **2023.1 Antelope was the first `YYYY.N` release**, and Bobcat continued the scheme. The DevStack "latest" documentation footer itself confirms the 2026 state of play: *"This release is under development. The current supported release is 2026.1."* (Verified — every docs.openstack.org/devstack/latest page carries this footer).

### 6.2 How DevStack Tracks Releases (Verified — repo README and FAQ)

DevStack's relationship to the release train is branch-based and explicitly documented:

- **`master` tracks trunk.** *"The DevStack master branch generally points to trunk versions of OpenStack components"* (Verified — README). The docs index puts it the same way: DevStack builds *"based on the latest versions of everything from git master."*
- **Stable branches mirror stable OpenStack.** *"For older, stable versions, look for branches named `stable/[release]` in the DevStack repo"* (Verified — README), with the example *"you can do the following to create a Zed OpenStack cloud: `git checkout stable/zed` `./stack.sh`."* The FAQ says it directly: *"DevStack master tracks the upstream master of all the projects. If you would like to run a stable branch of OpenStack, you should use the corresponding stable branch of DevStack as well."* Stable branches exist from Diablo onward (Verified — hacking docs, §2.2).
- **Per-project pinning is available but untested territory.** Setting `*_BRANCH` variables (e.g. `GLANCE_BRANCH=11.0.0.0rc1`, `NOVA_BRANCH=12.0.0.0.rc1`) or pointing `*_REPO` at custom trees lets you test milestones and Gerrit patch refs — but the FAQ warns *"Upstream DevStack is only tested with master and stable branches. Setting custom BRANCH definitions is not guaranteed to produce working results"* (Verified).

For an organization evaluating OpenStack in 2026, the practical rule: **evaluate on the stable branch matching the release you would actually deploy** (as of September 2026 that is `stable/2026.1`, tracking Gazpacho), and treat `master` as the developer/CI frontier that will become the October release.

---

## 7. The Use Cases — Development, CI, Testing and Demos

### 7.1 Development and Debugging — The Edit-and-Restart Loop (Verified — development docs)

DevStack's reason for being is the fast inner loop against *real* OpenStack code. Because every service runs from a git checkout under `/opt/stack/<project>`, the development workflow documented in "Developing with Devstack" is:

1. **Change the code directly** in `/opt/stack/$service`.
2. **Restart the affected daemons**: `sudo systemctl restart devstack@n-cpu.service`, or wildcard across everything your change touches: `sudo systemctl restart "devstack@n-*"`.
3. **Iterate** — then remember the warning: *"All changes you are making are in checked out git trees that DevStack thinks it has full control over. Uncommitted work ... may be overwritten during subsequent DevStack runs."*

For larger patch series the documented pattern is dedicated git trees wired in via `local.conf` (`NOVA_REPO=/home/sdague/nova`, `NOVA_BRANCH=fold_disk_config`), iterating with `./unstack.sh && ./stack.sh`; for review-in-progress code, point `*_BRANCH` at the Gerrit ref (`refs/changes/10/353710/1`). Library changes (oslo and friends) need `LIBS_FROM_GIT` plus a `sudo pip install -U .` in the library tree and a wildcard restart (Verified — development docs).

**Running services under the debugger** is documented in the systemd pages: the `pdb` route (stop the unit, extract its `ExecStart`, run the process by hand with `import pdb; pdb.set_trace()` injected, using `sg libvirt` for nova-compute) or the `remote-pdb` route, which works while the service stays under systemd — inject `import remote_pdb; remote_pdb.set_trace()`, restart the unit, and telnet to the port the log announces (Verified — systemd docs, "Debugging"; see §3.5).

### 7.2 CI — The Gate That Tests Every Patch (condensed cross-ref)

DevStack is the substrate of OpenStack's own CI: the docs index lists Zuul v3 roles and jobs for using DevStack in CI, and the README's goals include providing *"an environment for the OpenStack CI testing on every commit to the projects."* In the platform-engineering vocabulary of the sibling [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) (its CI/CD and golden-path chapters), DevStack is the *ephemeral test environment generator*: the gate spins up a disposable DevStack per patch, runs Tempest and project tests against it, and discards it — continuous integration where the environment itself is created and destroyed by automation rather than maintained. Teams outside OpenStack reuse the same trick (the repo's own guides do: e.g. Chameleon's CHI-in-a-box, a DevStack-derived single-node cloud, is maintained on stable/2023.1-style branches) — the CI theme is not re-derived here; read the AI-platform sibling for the general pipeline discipline, and note that DevStack's contribution is making the *OpenStack-shaped* target cheap enough to throw away on every commit.

### 7.3 Tempest and Functional Testing (Verified — DevStack Tempest docs, Tempest project docs)

**Tempest is the OpenStack integration test suite** — the Tempest project's own docs title it "Tempest - The OpenStack Integration Test Suite," and the DevStack docs say it plainly: *"Tempest is the OpenStack Integration test suite. It is installed by default and is used to provide integration testing for many of the OpenStack services"* (Verified — docs.openstack.org/devstack tempest page and docs.openstack.org/tempest). DevStack installs a Tempest checkout at `/opt/stack/tempest`, pre-configured for the stack it just built — *"Tempest tests will work in this environment"* (Verified — networking docs) — and the quick start points you at it: *"You can `cd /opt/stack/tempest` and run tempest tests that have been configured to work with your devstack."* The documented invocation is:

```bash
cd /opt/stack/tempest
/opt/stack/data/venv/bin/tempest run ...
```

Tempest itself is plugin-extensible, and the docs note the ecosystem pattern: *"many Tempest plugin packages also include DevStack plugin to do things like pre-create required static resources"* (Verified). For an evaluator, Tempest on DevStack is the sanctioned way to prove a service behaves to the OpenStack API contract before trusting any deployment — and the same tests are what the upstream gate runs against DevStack-shaped CI nodes (§7.2).

### 7.4 Learning and Demos — The "Try OpenStack in 20 Minutes" Use Case (Verified)

The classic pitch is real and documented: a clean VM, one `local.conf`, `./stack.sh`, and **15–30 minutes** later a complete OpenStack with Horizon, CLI credentials, a demo project, and a CirrOS image ready to boot (Verified — quick start; the CirrOS default upload is confirmed by the networking docs' SSH walkthrough). This is the single most common entry point into OpenStack on earth — the docs' "All-In-One Single VM" guide even turns it into a cloud-init one-shot: boot a VM with userdata that clones DevStack, writes the four-password `local.conf`, and runs `./stack.sh` (Verified — single-VM guide). The demo day then writes itself: Horizon at `http://<host>/`, log in as admin or demo, launch an instance from the CirrOS image, allocate a floating IP from `public`, ping it from the host. **The honest caveat for demo purposes**: the default stack is isolated to the host (§4.3) — reachable guests require the `PUBLIC_INTERFACE` configuration, and the docs warn that shared-interface mode can break host connectivity across reboots.

---

## 8. The Operations — Workflows, Logs and Troubleshooting

### 8.1 The Day-to-Day Command Set (all commands Verified — docs quick start, configuration, systemd, FAQ)

The operational vocabulary of a DevStack host, distilled from the official docs:

| Operation | Command | Verified source |
|---|---|---|
| First deployment | `./stack.sh` (as a non-root sudo user, from the devstack clone) | Quick start |
| Re-stack after `local.conf` edits | `./stack.sh` again — it re-reads configuration, freshens repos and restarts services | Configuration docs (RECLONE semantics), development docs |
| Full iteration on a patch series | `./unstack.sh && ./stack.sh` | Development docs |
| Stop the stack | `./unstack.sh` | Plugin docs (unstack mode) |
| Wipe state for a clean run | `./clean.sh` (calls `unstack.sh` first) | Plugin docs, FAQ |
| Restart one service / a group / everything | `sudo systemctl restart devstack@n-cpu.service` / `"devstack@n-*"` / `devstack@*` | Systemd docs |
| Status of everything DevStack runs | `sudo systemctl status "devstack@*"` | Development docs |
| Live service logs | `sudo journalctl -f --unit devstack@n-cpu.service` (wildcards supported) | Systemd docs |
| stack.sh's own run log | `LOGFILE=$DEST/logs/stack.sh.log` in `local.conf` — a timestamped file per run, pruned by `LOGDAYS` | Configuration docs |
| Environment for the CLI | `. openrc` then `openstack server list` | Quick start, README |
| Inventory of what is installed | `tools/info.sh` (apt/pip/git versions) | FAQ |

Two operational reflexes the docs drill in: **(1)** treat every `/opt/stack/<project>` tree as disposable — push work out before re-stacking (§3.4); **(2)** when a re-run misbehaves because the host has drifted, the sanctioned reset is `clean.sh` and a fresh `stack.sh`, not hours of hand-fixing — the FAQ's own advice for a poisoned state is *"use `clean.sh` to eradicate it and try again."*

### 8.2 The Troubleshooting Ledger — Verified Fixes and Honest Caveats

| Symptom | What the docs actually say | Status |
|---|---|---|
| **RabbitMQ will not start on a fresh VM** | *"This is often caused by `erlang` not being happy with the hostname resolving to a reachable IP address"* — fix the hostname in `/etc/hosts` (127.0.0.1 is usually enough for single-node); extreme case: `clean.sh` and retry | ✅ Verified — FAQ |
| **Configuration changes seem ignored** | The package-prerequisite check inside `tools/install_prereqs.sh` has a timer (`PREREQ_RERUN_HOURS`, default 2 hours) that skips re-checks; `FORCE_PREREQ=1` forces them | ✅ Verified — FAQ |
| **Repos stale / stuck on old code** | `RECLONE=yes` freshens every project repo on each run (default behavior only clones missing repos) | ✅ Verified — configuration docs |
| **My code edits vanished** | The volatile-tree model: uncommitted work may be overwritten on the next `stack.sh` — back up to Gerrit first (§3.4) | ✅ Verified — FAQ and development docs |
| **Network ranges clash with the LAN** | RFC-1918 overlap between the host network and DevStack's fixed/floating ranges is called out as common; set `IPV4_ADDRS_SAFE_TO_USE`, `FLOATING_RANGE` away from local use | ✅ Verified — configuration docs |
| **Guests unreachable from outside** | By design — the default stack is host-isolated (§4.3); connect `br-ex` via `PUBLIC_INTERFACE` (dedicated NIC recommended) | ✅ Verified — networking docs |
| **Host loses network after reboot in shared-interface mode** | The docs warn the OVS/bridging interaction can break connectivity when sharing one interface — a known reason to prefer a dedicated interface or a VM | ✅ Verified — networking docs |
| **Inbound SSH/ping to guests fails** | Security groups block all inbound by default; add ICMP + TCP 22 rules to the default group | ✅ Verified — networking docs |
| **Apt/pip downloads stall behind a corporate proxy** | The docs assume direct internet; proxy handling is standard OS-level apt/pip configuration, not DevStack-specific | ⚠ Reported-approximate — general practice, not covered in the DevStack docs this pass |
| **"Insecure" / TLS-trust errors on clients** | DevStack can run services behind its own TLS proxy with a self-signed DevStack CA (`lib/tls` builds a root + intermediate CA); clients must trust that CA or bypass verification — the mechanics are in `lib/tls`, but the "insecure flag" folklore is community practice | ⚠ Reported-approximate — `lib/tls` verified (the CA machinery exists); client-flag guidance not in the docs this pass |
| **Network-namespace / OVS oddities after host reboots or on nested virtualization** | Real and community-documented, but not in the core docs pages; the official guides route around it (dedicated VM, KVM nested-virtualization guide, multinode-lab guide) | ⚠ Reported-approximate — community/experience; the guide list itself is ✅ |
| **A "FORCE=yes" style re-clone** | The brief's folklore item could not be confirmed at a primary source this pass; the documented equivalents are `RECLONE=yes` and `FORCE_PREREQ=1` | ❌ Failed-to-verify as stated — see the claims audit |

The pattern behind the ledger: **the official docs cover the failures that come from DevStack's own design** (volatile trees, host isolation, security-group defaults, hostname/erlang, prereq timers). The failures that come from *the environment around it* — proxies, nested virtualization, rebooted OVS, TLS trust — are real but live in community channels, and this guide flags them as such rather than laundering them into doc citations.

---

## 9. The Limitations — Resources and the Not-for-Production Posture

### 9.1 The Resource Question — What the Docs Actually Say

The official documentation is **deliberately light on hard numbers**, and this guide will not invent any:

- The single-VM guide states: *"DevStack should run in any virtual machine running a supported Linux release. It will perform best with 4GB or more of RAM."* (✅ Verified — this is the only explicit memory guidance in the core docs this pass.)
- The quick start warns the first run takes 15–30 minutes *"largely depending on the speed of your internet connection"* and that *"many git trees and packages will be installed"* — disk headroom of tens of GB is implied by the git-tree model (`DEST=/opt/stack` holds every project checkout plus data), but no official minimum is stated (⚠ soft guidance).
- The commonly quoted community recipe — **8 GB RAM and 4 vCPUs** for a comfortable all-in-one with Horizon, Neutron and Tempest, more for Swift/Octavia — is **⚠ Reported-approximate**: sound advice, widely repeated in tutorials, but not present in the official DevStack docs this pass. The docs' own floor is the 4 GB line above; plan 8 GB if you want the demo day in §7.4 to feel good, and note that the single-VM guide's honest caveat applies — inside a VM, instances run under QEMU emulation and are slow.

### 9.2 The Not-for-Production Posture (Verified — FAQ and overview docs)

DevStack's own documentation is the strongest source for this section, and it is unambiguous:

> *"DevStack is targeted at developers and CI systems to use the raw upstream code. It makes many choices that are not appropriate for production systems. Your best choice is probably to choose a distribution of OpenStack."* — FAQ, "Can I use DevStack for production?"

The supporting evidence the docs stack up: the overview warns DevStack is *"not a general OpenStack installer and was never meant to be everything to everyone"* and that most configuration combinations are *"rarely, if ever, tested"*; the quick start warns it *"will make substantial changes to your system"* and should run only on dedicated hosts; the README's execution-environment section says running it on a system you care about is *"a recipe for disappointment, or worse."* Every one of these is a primary-source quote (Verified — FAQ, overview, quick start, README). The production-shaped alternatives are §10's subject; DevStack's job ends where production begins.

### 9.3 The Security Posture — What a Lab Host Is (and Is Not)

For the Cymbal Bank context in §11, the security reading of the docs matters more than the feature reading:

- DevStack installs **default, shared, well-known passwords** (the four `*_PASSWORD` values in `local.conf`), creates **admin and demo accounts** with them, and its docs tell you to use alphanumeric-only passwords for compatibility (Verified — configuration docs, single-machine guide). A DevStack is therefore trivially compromisable by design — it must be **network-isolated** (the default networking already isolates guests; the host itself must be isolated by the operator).
- The docs recommend running DevStack on a **dedicated, disposable host or VM** (Verified — quick start warning; README's "run it in a VM ... snapshot capabilities" advice).
- DevStack's own TLS story is a lab story: a self-signed CA chain generated by `lib/tls`, fine for exercising Keystone-over-TLS code paths, not a substitute for real PKI (Verified — lib/tls; the operational conclusion is ⚠ inference).
- There is **no supported upgrade path, no HA, no backup story** — the model is destroy-and-recreate (`clean.sh`). Anything that needs continuity, audit-grade security, or multi-tenant isolation belongs on a real distribution (Verified — FAQ's production answer; cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5 for the distribution landscape).

### 9.4 The Honest Limitations List

1. **Single-node by default** — multi-node exists but only *"as are tested by the gate"* (Verified — overview docs).
2. **Trunk-first** — master tracks git master; stable behavior requires checking out a `stable/*` branch (Verified — README, FAQ).
3. **Volatile working trees** — uncommitted work is at risk on every re-run (Verified — FAQ).
4. **Not a general installer** — exotic configurations are untested (Verified — overview docs).
5. **Not production** — see §9.2; the quotes are the docs' own.

---

## 10. The Alternatives — The Deployment-Tooling Ecosystem

DevStack occupies one narrow band of the OpenStack deployment spectrum: **developer/CI environments on one host, from git**. Every other tool in this section exists because someone needed OpenStack to be something DevStack is explicitly not — reproducible, supported, production-shaped, or containerized. (For the distribution *products* built on these tools — Red Hat, Canonical, Mirantis — cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5.)

### 10.1 The Quickstart / Dev Tools

- **DevStack** — the subject of this guide: single-node, from git, systemd-run, dev/CI/learning (Verified throughout).
- **MicroStack (Canonical)** — *"an Ubuntu solution for the installation of OpenStack on a single machine. MicroStack was designed for small-scale cloud environments, edge deployments, testing, and development"* (✅ Verified at OpenInfra's Superuser publication). It is snap-packaged — "OpenStack in a snap," with services and libraries in confined snaps rather than git trees. **⚠ Reported-approximate on status**: MicroStack remains installable, but Canonical's product energy has moved to Sunbeam; a primary-source statement of MicroStack's current maintenance status could not be pinned down this pass.
- **PackStack + RDO quickstart (the CentOS Stream path)** — RDO is *"a free, community supported distribution of OpenStack for RHEL and CentOS"* that *"should work on RHEL but is currently only tested on CentOS Stream"* (✅ Verified — OpenStack install guide and RDO docs). PackStack is its quickstart tool: *"an OpenStack deployment tool intended to install Proof of Concept small environments in a quick and easy way ... Production features such as High Availability, OpenStack upgrades or other day-2 operations are out of the scope of Packstack"* (✅ Verified — rdoproject.org). It is the DevStack analogue for the RHEL family — with the difference that it deploys packaged (not git) services (⚠ the Puppet-under-the-hood detail is widely documented but was not re-verified at a primary source this pass).

### 10.2 The Production Deployment Tools

- **Kolla-Ansible (containerized, production-oriented)** — *"Kolla provides Docker containers and Ansible playbooks to meet Kolla's mission. Kolla's mission is to provide production-ready containers and deployment tools for operating OpenStack clouds. Kolla is highly opinionated out of the box, but allows for complete customization"* (✅ Verified — project-deploy-guide). Two linked deliverables: the **Kolla** image-build project and **Kolla-Ansible**, which *"deploys OpenStack services and infrastructure components in Docker containers"* (✅ Verified — repo README). This is the most common open answer to "we evaluated on DevStack, now deploy it for real."
- **TripleO (retired)** — the undercloud/overcloud project: *"installing, upgrading and operating OpenStack clouds using OpenStack's own cloud facilities as the foundation — building on Nova, Ironic, Neutron and Heat to automate cloud management at datacenter scale"* (✅ Verified — TripleO docs, describing the model). **Status: retired.** The TripleO repositories on opendev carry the standard retirement banner — *"RETIRED ... The contents of this repository are still available ... before it reached its end of life"* — and the ecosystem records the retirement around the **Epoxy (2025.1)** cycle, with Red Hat's replacement direction being Red Hat OpenStack Services on OpenShift (✅ Verified — opendev RETIRED banners; the Epoxy-cycle timing and RHOSO successor are ⚠ Reported-approximate — secondary sources; the banners themselves do not carry dates).
- **OpenStack-Helm (Kubernetes-native)** — *"a collection of Helm charts that simply, resiliently, and flexibly deploy OpenStack and related services on Kubernetes"* (✅ Verified — repo README). It is an active official OpenStack project whose documentation is versioned with the calendar releases (docs builds for 2025.2.1 and 2026.1.1 were visible this pass — ✅ Verified). This is the "OpenStack on Kubernetes" lineage that the Nutanix sibling notes the major distributions have pivoted toward (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5).
- **OpenStack-Ansible** — an official OpenStack project *"which aims to deploy production environments from source in a way that makes it scalable while also being simple to operate, upgrade, and grow"* (✅ Verified — repo README), deploying services into isolated LXC containers or onto bare metal via Ansible playbooks and roles. Its docs' support-status table (version 33.x, tracking the 2026 releases) is also where the next release name, **2026.2 Hibiscus**, is recorded as "Under Development" (✅ Verified — docs.openstack.org/openstack-ansible).
- **Sunbeam / Canonical OpenStack** — the snap-based Canonical product line: *"Canonical OpenStack (based on Sunbeam) is an enterprise cloud solution that distills the maturity and comprehensiveness of the upstream OpenStack project"* (✅ Verified — snapcraft.io/openstack). Sunbeam is the current Canonical vehicle for OpenStack from single-node evaluation to multi-node production, operated through the `openstack` snap and the `sunbeam` command line (✅ Verified at product-materials level; deep mechanics ⚠ not re-verified this pass).

### 10.3 The Spectrum at a Glance

| Tool | What it deploys | Target posture | Status (Sept 2026) |
|---|---|---|---|
| **DevStack** | Single-node OpenStack from git, systemd-run | Development, CI, learning — explicitly not production | ✅ Active, core to OpenStack's own gate (Verified) |
| **MicroStack** | Single-node OpenStack as snaps (Ubuntu) | Dev/testing/small edge | ✅ Installable; ⚠ development energy moved to Sunbeam (Verified at Superuser; status ⚠) |
| **PackStack quickstart** | Packaged OpenStack on CentOS Stream (RDO) | Proof-of-concept on RHEL family | ✅ Active for RDO (Verified) |
| **Kolla-Ansible** | Containerized (Docker) OpenStack via Ansible | Production-oriented | ✅ Active, GA-aligned releases (Verified) |
| **TripleO** | Undercloud/overcloud via Nova/Ironic/Heat | Production (Red Hat director era) | ❌ **Retired** — repos marked RETIRED/EOL (Verified); timing ⚠ |
| **OpenStack-Helm** | OpenStack on Kubernetes via Helm charts | Production, K8s-native | ✅ Active, calendar-versioned docs (Verified) |
| **OpenStack-Ansible** | OpenStack from source into LXC/bare metal via Ansible | Production | ✅ Active (Verified) |
| **Sunbeam (Canonical OpenStack)** | Snap-based OpenStack, single→multi node | Evaluation → production (Canonical-supported) | ✅ Active product line (Verified at product level) |

The decision rule the table encodes: **match the tool to the job**. DevStack when the job is *learning, developing or gating OpenStack code*; PackStack or MicroStack when the job is *a quick proof-of-concept on the team's preferred distro*; Kolla-Ansible, OpenStack-Ansible, OpenStack-Helm or a distribution product when the job is *operating a cloud* — and nothing from the left column when the word "production" appears in the requirement.

---

## 11. The Worked Example — A Cymbal Bank DevStack Evaluation Lab

### 11.1 The Scenario

The infrastructure team at **Cymbal Bank** (Singapore) is conducting the standing private-cloud evaluation described in [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §8. The platform team has never operated OpenStack and needs first-hand evidence — API behavior, operational texture, failure modes — before any distribution conversation. The decision: stand up a **DevStack evaluation lab** on an isolated host, run it for a fixed evaluation window, and produce evidence the architecture board can read. The conventions follow the sibling worked examples: an isolated lab, zero production data, a documented teardown, and the bank as the only persona.

### 11.2 The Lab Host and the Branch Choice

| Decision | Choice | Rationale |
|---|---|---|
| Host | A dedicated lab VM on the bank's isolated lab VLAN (sizing ⚠: 8 vCPU / 16 GB RAM / 100 GB disk — above the docs' 4 GB floor, comfortable for Horizon + Neutron + Tempest) | The docs demand a dedicated, disposable host (§9.3) |
| OS | Ubuntu 24.04 LTS | "The most tested" (Verified — quick start) |
| Branch | `stable/2026.1` (Gazpacho) | Match the release the bank would actually deploy; master is the developer frontier (§6.2) |
| Network | Lab VLAN only; no route to bank production; host firewall allows only the team's jump host | The not-for-production posture, enforced at the network layer (§9) |
| Lifecycle | Snapshot before first `stack.sh`; destroy-and-recreate is the operational model (`clean.sh`) | DevStack's own model; also the bank's audit-friendly story |

### 11.3 The local.conf

```bash
[[local|localrc]]
# Cymbal Bank DevStack evaluation lab — September 2026 (stable/2026.1 Gazpacho)
# Lab-only values. No production credentials, no production data, isolated VLAN.
HOST_IP=10.99.0.10
ADMIN_PASSWORD=Lab0nlyStack1
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

# Log the run for the evaluation record
LOGFILE=$DEST/logs/stack.sh.log
LOGDAYS=7

# Keep the default isolated networking (floating range 172.24.4.0/24 on br-ex,
# private tenant networks) — the lab VLAN has no overlap with it.

# Optional extras for the evaluation (each requires a re-stack to take effect):
# enable_service swift            # object-storage API hands-on
# enable_service h-eng h-api      # Heat orchestration hands-on
```

Every line is a documented knob (§3.3): the four passwords (alphanumeric only — Verified warning), `HOST_IP` pinned because bridge creation can confuse later auto-detection, `LOGFILE`/`LOGDAYS` for the evidence trail. The commented extras reflect §4.2's roster reality: Swift/Heat are opt-in, and each change means a re-stack (`./stack.sh`) or a full cycle (`./clean.sh && ./stack.sh`).

### 11.4 The Validation Checklist After stack.sh

The docs prescribe the post-install vocabulary; the bank's evaluation runs it as a scripted checklist (all commands Verified in §4.3, §7.3 and the quick start):

1. **Services are registered** — `. openrc` (admin), then `openstack service list` shows keystone, glance, nova, placement, cinder, neutron (and horizon as a web service); `openstack endpoint list` confirms the catalog.
2. **The dashboard answers** — browse `http://10.99.0.10/` (Horizon), log in as `admin` (or `demo`) with the lab password. Horizon is one of the seven default installs (Verified — quick start).
3. **The image is there** — `openstack image list` shows the CirrOS image DevStack uploads by default.
4. **A test VM lives and breathes** — the docs' canonical smoke test: create a keypair; open the default security group (`icmp`, `tcp/22`); boot on `private`; allocate a floating IP from `public`; attach it; then `openstack server ssh test-server -- -l cirros`. The VM should ping the host and reach the outside for package updates (the default masquerade) — while remaining invisible to the rest of the VLAN (the default isolation, §4.3).
5. **The API contract is exercised** — run the pre-configured Tempest suite from `/opt/stack/tempest` (`/opt/stack/data/venv/bin/tempest run --smoke` ⚠ smoke-selection flag is the Tempest CLI convention, not a DevStack-doc quote) and record the pass rate as evaluation evidence.
6. **The edit-restart loop is demonstrated once** — change a log-level string in `/opt/stack/nova/...`, `sudo systemctl restart "devstack@n-*"`, observe the change in `journalctl` — the one demo that proves why DevStack exists (§7.1).
7. **The teardown is rehearsed** — `./unstack.sh`, then `./clean.sh`, and a restore from the pre-stack snapshot, proving the environment is disposable (Verified — §3.4).

### 11.5 The Security Posture — Consistent with the House Conventions

The worked examples across this repository (the Nutanix guide's §8, the OpenShift guide's §14) share a security frame that this lab applies unchanged:

- **Isolation**: dedicated host on an isolated lab VLAN, reachable only via the team's jump host; no route to production; the lab's default networking keeps even *guest* traffic host-local.
- **No production data, no production credentials**: every password is lab-generated, alphanumeric, and documented as such; the evaluation images (CirrOS) and workloads are synthetic. Nothing from the bank's systems touches the lab.
- **Known-weak by design**: DevStack ships shared default accounts and a self-signed TLS story (§9.3) — acceptable only inside the isolation boundary, which is exactly why the isolation boundary exists.
- **Ephemerality as a feature**: snapshot before stacking, destroy after the evaluation window, and the audit trail is the `LOGFILE`, the validation-checklist output and the written findings — not a system that must be maintained.
- **The evaluation's output**: an evidence pack (service list, Tempest results, VM smoke test, operational notes from §8's troubleshooting ledger) that feeds the distribution-selection conversation in §10 — the board reads evidence about the *platform*, produced on a tool that was never meant to be the platform. 
---

## 12. The Summary — One Script, Many Labs

This guide opened with a deliberately narrow question — what exactly is DevStack, the script that sits at the bottom of every OpenStack tutorial — and answering it turned out to require the whole picture of how OpenStack is developed, tested and evaluated.
Sections 1–11 each established one part of that picture; this section synthesizes them, and the glossary and claims audit that follow close the guide in the repository's standard shape.

### 12.1 The Guide at a Glance

| Section | What it established |
|---|---|
| §1 The Project | DevStack is a series of extensible scripts that bring up a complete OpenStack environment on one host from git master — a development and functional-testing tool, not a production installer |
| §2 The History | OpenStack launched October 2010 (Rackspace + NASA); DevStack's verifiable fragments run from the Diablo-era stable branches (2011) through the systemd migration; the Devon/Stacker name story is anecdote, honestly flagged |
| §3 The Mechanics | The stack.sh / unstack.sh / clean.sh cycle; local.conf's `[[phase|file]]` language; the four passwords; volatile `/opt/stack` git trees; systemd `devstack@` units |
| §4 The Architecture | All-in-one: keystone, glance, nova, placement, cinder, neutron, horizon over MySQL/RabbitMQ/Apache; networking isolated by design (`172.24.4.0/24` floating range, `br-ex` unconnected) |
| §5 The Plugin Model | `extras.d/` in-tree hooks plus the out-of-tree plugin contract (`enable_plugin`, `override-defaults`, `settings`, `plugin.sh`) with strong mode/phase guarantees |
| §6 The Release Alignment | Codenames ended at Zed; calendar versioning since 2023.1 Antelope; master tracks trunk, `stable/<release>` tracks releases — evaluate on `stable/2026.1` (Gazpacho) |
| §7 The Use Cases | Edit-and-restart development, the CI gate on DevStack-shaped nodes, Tempest integration testing, and the 20-minute demo |
| §8 The Operations | A short command vocabulary (stack.sh, `systemctl devstack@*`, `journalctl`, clean.sh) and a troubleshooting ledger split between doc-verified fixes and community-channel reality |
| §9 The Limitations | Soft resource guidance (the docs' only floor: 4 GB RAM); an unambiguous not-for-production posture quoted from DevStack's own FAQ |
| §10 The Alternatives | A spectrum from quickstart tools (PackStack, MicroStack) to production tooling (Kolla-Ansible, OpenStack-Ansible, OpenStack-Helm, retired TripleO, Sunbeam); match the tool to the job |
| §11 The Worked Example | A Cymbal Bank evaluation lab: isolated-VLAN host, Ubuntu 24.04, `stable/2026.1`, lab-only passwords, a scripted validation checklist and a rehearsed teardown |

### 12.2 The Project

DevStack is the fastest legitimate way to see an OpenStack cloud assemble itself: one script on a clean host, 15–30 minutes later a working all-in-one cloud is up, every service running from a git checkout rather than from packages.
The official definition carries two corollaries the docs repeat at every turn: DevStack is *"used interactively as a development environment and as the basis for much of the OpenStack project's functional testing,"* and it is not a general installer — *"DevStack is not a general OpenStack installer and was never meant to be everything to everyone."*
The not-for-production posture is the project's own words: *"It makes many choices that are not appropriate for production systems. Your best choice is probably to choose a distribution of OpenStack."* DevStack is the learning-and-development layer of the OpenStack world; production is someone else's job (§9, §10).

### 12.3 The Mechanics and the Architecture

The mechanics are deliberately small: `local.conf` — a modified INI format whose `[[local|localrc]]` section holds the four passwords and whose `[[post-config|...]]` sections write into service configuration files — the three-command stack cycle, and the volatility warning that every `/opt/stack/<project>` tree is a git checkout DevStack owns.
The architecture is the all-in-one topology: seven default services on one host, with isolated-by-design networking — private project networks, a `172.24.4.0/24` floating range on an OVS bridge connected to nothing physical, and a masquerade rule for egress. Everything outside the core roster arrives as a plugin (§4, §5).

### 12.4 The Use Cases and the Operations Reality

The same mechanism serves four audiences: the developer living in the edit-and-restart loop against real code; the CI system spinning up a disposable DevStack per patch and running Tempest against it; the tester exercising the API contract with the pre-configured suite at `/opt/stack/tempest`; and the evaluator who wants first-hand OpenStack behavior without a production commitment.
Operationally the tool is honest about its own shape: a short command vocabulary, services as `devstack@*.service` units, logs in the journal, and a troubleshooting ledger whose verified fixes cover the failures DevStack's own design produces — while failures from the surrounding environment (proxies, nested virtualization, rebooted OVS) are flagged as community-documented rather than laundered into doc citations (§7, §8).

### 12.5 The Alternatives Decision Rule

DevStack occupies one narrow band of the deployment spectrum: single-node, from git, for development and CI.
The rule the spectrum encodes is to match the tool to the job — DevStack when the job is learning, developing or gating OpenStack code; PackStack or MicroStack when the job is a quick proof-of-concept on a preferred distro; Kolla-Ansible, OpenStack-Ansible, OpenStack-Helm or a distribution product when the job is operating a cloud; and nothing from the dev-tool column when the word "production" appears in the requirement.
The release alignment closes the loop: master tracks trunk, `stable/<release>` tracks releases, and a September-2026 evaluation belongs on `stable/2026.1` (Gazpacho) (§6.2, §10.3).

### 12.6 The Cymbal Bank Evaluation

The worked example in §11 applied the guide the way a bank actually would: a dedicated lab VM on an isolated VLAN, Ubuntu 24.04 ("the most tested"), the `stable/2026.1` branch, a minimal `local.conf` with lab-only alphanumeric passwords and a logged run, then a scripted validation checklist — service list, Horizon login, CirrOS image, floating-IP smoke test, a Tempest smoke run, one demonstration of the edit-and-restart loop, and a rehearsed teardown.
The security posture followed the repository's house conventions: isolation, no production data or credentials, known-weak-by-design accepted only inside the isolation boundary, and ephemerality as a feature — snapshot before stacking, destroy after the evaluation window, with the `LOGFILE`, checklist output and written findings as the audit trail (§11.3–11.5).

One script, many labs: the same `./stack.sh` that raises a developer's disposable cloud is the substrate of the gate that guards OpenStack's master branch, the environment where Tempest proves the API contract, and the evaluation host on which Cymbal Bank formed its first honest opinion of the platform — sixteen years after the first services were stitched together with shell, the script is still OpenStack's front door.

---

## 13. The Glossary

Terms as this guide uses them; each is defined at first use in the section cited, and the definitions below are mined from the body text so the glossary stays consistent with the doc.

| Term | Meaning |
|---|---|
| **DevStack** | The series of extensible Bash scripts that bring up a complete OpenStack environment on one host from git master — a development and functional-testing tool, not a production installer (§1) |
| **stack.sh** | DevStack's primary deployment script; performs the bulk of the work, refuses to run as root, and takes 15–30 minutes on a first stack (§1.3, §3.1) |
| **stack user** | The non-root account with passwordless sudo that stack.sh expects; the docs' example is `stack`, and `tools/create-stack-user.sh` encodes the account expectations (§3.1) |
| **local.conf** | The configuration file: a modified INI format with `[[phase|file]]` meta-sections; the home of DevStack variables and of writes into services' own config files (§3.2) |
| **localrc** | The legacy top-level configuration file that predates `local.conf` (introduced October 2013); `[[local|localrc]]` still provides a default `.localrc.auto` for backward compatibility (§2.2, §3.2) |
| **stackrc** | The global configuration file sourced early by the scripts; it calls `local.conf` (or legacy `localrc`) so local settings are recognized (§1.3) |
| **openrc** | The environment file that sets `OS_*` credentials so the `openstack` CLI talks to the right cloud (§1.3, §3.1) |
| **unstack.sh** | Tears the stack down, stopping the services; in the plugin contract it runs before other services are shut down (§3.4) |
| **clean.sh** | Removes state and transient data for a from-scratch re-run; calls `unstack.sh` first (§3.4) |
| **re-stack** | Re-running `./stack.sh` after `local.conf` edits; the standard redeploy workflow, and the answer to most drift problems (§3.4, §8.1) |
| **/opt/stack** | The default `DEST` install directory holding every project's git checkout — trees DevStack considers disposable, so uncommitted work must be pushed out before re-runs (§3.3–3.4) |
| **Keystone** | OpenStack identity: users, projects, roles, tokens, the service catalog; one of the seven default services (§4.2) |
| **Nova** | OpenStack compute: VM lifecycle management; runs on the host's own hypervisor, KVM (or QEMU emulation inside a VM) (§4.1–4.2) |
| **libvirt** | The virtualization library through which nova-compute drives the hypervisor; the systemd debugger workflow runs nova-compute by hand under `sg libvirt -c '...'` (§3.5) |
| **Placement** | The resource-inventory service that tracks what Nova can schedule onto; one of the seven default services (§4.2) |
| **Glance** | The image service; the CirrOS test image is uploaded into it by default (§4.2) |
| **Neutron** | OpenStack networking: networks, routers, floating IPs, security groups, with OVS and the L3 agent; isolated-by-design defaults (§4.2–4.3) |
| **Cinder** | Block storage: volumes attached to instances, LVM-backed by default (§4.2) |
| **Horizon** | The web dashboard, at `http://<host>/`; one of the seven default services (§4.2) |
| **Swift** | Object storage (the Rackspace heritage from 2010); named in the overview roster but opt-in in current practice (`enable_service swift`) (§4.2) |
| **Heat** | Orchestration via HOT templates; opt-in rather than part of the default roster (§4.2) |
| **Octavia** | Load-balancer-as-a-service; opt-in, with a dedicated configuration guide (§4.2) |
| **Tempest** | The OpenStack integration test suite; installed by default with the stack and pre-configured at `/opt/stack/tempest` (§7.3) |
| **CirrOS** | The tiny test Linux image DevStack uploads by default; the standard boot target for smoke tests (§3.1, §7.4, §11.4) |
| **all-in-one** | The single-host topology: controller, compute, network, database, queue and dashboard all on one machine (§4.1) |
| **multi-node** | The topology beyond single node — supported only "as are tested by the gate"; DevStack's core supported configurations are single-node plus the gate-tested multi-node shapes (§4.1) |
| **screen vs systemd** | The old process model (screen sessions, from the era of fewer than ten services) versus today's default: `devstack@<service>.service` units under a `devstack` slice (§3.5) |
| **devstack slice** | The dedicated systemd slice inside which every `devstack@*.service` unit runs, making group operations (`systemctl restart devstack@*`) trivial (§3.5) |
| **extras.d** | The in-tree directory of dispatch scripts called by the hooks in `stack.sh`, `unstack.sh` and `clean.sh` (§5.1) |
| **plugin** | Out-of-tree Bash code that attaches a service DevStack does not ship, registered via `enable_plugin <NAME> <GITURL> [GITREF]`; contract files: `override-defaults`, `settings`, `plugin.sh` (§5.2) |
| **enable_plugin** | The `localrc`-section directive that registers an external plugin with DevStack: `enable_plugin <NAME> <GITURL> [GITREF]`, where the optional gitref (default master) pins a branch, ref or tag (§5.2) |
| **enable_service / disable_service** | Turn roster services on or off in `local.conf`, e.g. `disable_service c-vol`, `enable_service swift` (§3.3) |
| **gate/check jobs** | OpenStack CI jobs that test every patch to every service on DevStack-shaped environments — the reason the tool survives (§2.4, §7.2) |
| **calendar versioning** | The `YYYY.N` release naming used since 2023.1 Antelope, two releases per year (roughly April and October); the A–Z codenames ended at Zed (2022.3) (§6.1) |
| **stable/\<release\> branch** | A DevStack branch tracking a specific OpenStack release, maintained from Diablo onward — e.g. `stable/zed`, `stable/2026.1` (§6.2) |
| **RDO** | The free, community-supported OpenStack distribution for RHEL and CentOS; currently tested on CentOS Stream rather than RHEL itself (§10.1) |
| **PackStack** | RDO's quickstart deployment tool for proof-of-concept environments; HA, upgrades and other day-2 operations are out of scope (§10.1) |
| **Kolla-Ansible** | The containerized production tooling: Kolla's Docker images plus Ansible playbooks for deploying and operating OpenStack clouds (§10.2) |
| **TripleO** | The retired undercloud/overcloud project that used OpenStack's own services (Nova, Ironic, Neutron, Heat) to deploy OpenStack at datacenter scale (§10.2) |
| **OpenStack-Helm** | The Helm charts that deploy OpenStack and related services on Kubernetes; an active official project with calendar-versioned docs (§10.2) |
| **OpenStack-Ansible** | The official project that deploys production environments from source into isolated LXC containers or bare metal via Ansible playbooks (§10.2) |
| **MicroStack** | Canonical's snap-packaged single-machine OpenStack, aimed at small-scale, edge, testing and development; product energy has moved to Sunbeam (§10.1) |
| **Sunbeam** | Canonical's current snap-based OpenStack line (Canonical OpenStack), spanning single-node evaluation to multi-node production (§10.2) |
| **ENABLED_SERVICES** | The internal service list — admittedly *"entirely too overloaded"* — that mixes OpenStack services (n-cpu, g-api) with system backends (mysql, rabbitmq) (§4.2) |
| **FLOATING_RANGE** | The external floating-IP range; default `172.24.4.0/24` with gateway `172.24.4.1`, the external network named `public` (§3.3, §4.3) |
| **IPV4_ADDRS_SAFE_TO_USE** | The configuration variable steering the private address space away from ranges in use on the host; RFC-1918 overlap between the LAN and DevStack's fixed/floating ranges is a classic first-run failure (§3.3) |
| **br-ex** | The OVS bridge Neutron controls for all its networking; connected to no physical interface in the default isolated setup (§4.3) |
| **PUBLIC_INTERFACE** | The configuration that attaches `br-ex` to a physical NIC when guests must be reachable beyond the host; a dedicated interface is recommended over sharing the host's single interface (§4.3) |
| **security group** | The per-project firewall; by default all inbound traffic to guests is blocked, so ICMP and TCP/22 rules must be opened before SSH/ping works (§4.3) |
| **floating IP** | The public-range address attached to a guest for external reachability; allocated from `public` and attached in the canonical smoke test (§4.3) |
| **demo project** | The default OpenStack project created by a stock stack, with fixed IPs on a subnet from `10.0.0.0/22` (§4.3) |
| **RECLONE** | `RECLONE=yes` freshens every project repo on each `stack.sh` run instead of only cloning missing ones; the default behavior is plain cloning plus hard resets (§3.3) |
| **HOST_IP** | The host IP used for API endpoints; normally auto-detected on the first run, but bridge creation can confuse later detection — pin it in `local.conf` (§3.3) |
| **LOGFILE / LOGDAYS** | Send stack.sh's own output to `$DEST/logs/stack.sh.log` (timestamped per run) and prune old logs; services themselves log to the journal (§3.3, §8.1) |
| **cloud-init one-shot** | The single-VM guide's trick: boot a VM whose userdata clones DevStack, writes the four-password `local.conf`, and runs `./stack.sh` automatically (§7.4) |
| **Gerrit ref** | A review-in-progress patch reference (`refs/changes/10/353710/1`) that `*_BRANCH` variables can point at so a stack tests unmerged code (§3.3, §7.1) |
| **lib/tls** | The in-tree code that builds DevStack's self-signed root-plus-intermediate CA chain for exercising TLS code paths — a lab story, not real PKI (§9.3) |
| **openstack CLI** | The command-line client used after sourcing `openrc`; the quick start's own path is `source openrc`, then `openstack server list` (§3.1, §8.1) |
| **remote-pdb** | The debugger route that works while a service stays under systemd: inject `remote_pdb.set_trace()`, restart the unit, telnet to the announced port (§3.5) |
| **Zuul** | The CI system whose v3 roles and jobs run DevStack in OpenStack's gate; the docs index lists them for using DevStack in CI (§7.2) |

---

## 14. Claims Status and Verification Notes

This guide was researched in September 2026 with live web access to the primary sources listed in the header: docs.openstack.org/devstack (quick start, configuration, networking, plugins, systemd, Tempest, FAQ, overview, development and hacking pages), the openstack/devstack repository on opendev.org and its GitHub mirror, releases.openstack.org and governance.openstack.org for the release chronology, docs.openstack.org/tempest, and the deployment-tooling projects (RDO/PackStack, kolla-ansible, tripleo-docs, openstack-helm, openstack-ansible, and Canonical's MicroStack/Sunbeam materials).
Statuses follow the house convention announced in the introduction: **✅ Verified** (confirmed against a primary source this pass, or carried from the cross-referenced sibling's verified ledger), **⚠ Reported-approximate** (widely reported or soft, not pinned to a primary source), **❌ Failed-to-verify** (checked and not confirmable). The table audits the claims that carry inline markers in the body; the ledger of what could not be verified is §14.1.

The audit also records two places where this guide deliberately corrected the framing it was handed: the calendar-versioning start date (2023.1 Antelope, not 2023.2 Bobcat — §6.1) and the expected DevStack "history page" (which does not exist — §2.3).

| Claim | Status | Basis |
|---|---|---|
| DevStack is a series of extensible scripts that bring up a complete OpenStack environment on a single host from git master — a development and functional-testing tool | ✅ Verified | DevStack docs index, quoted in §1.1 |
| DevStack is "not a general OpenStack installer," "was never meant to be everything to everyone," and "makes many choices that are not appropriate for production systems" | ✅ Verified | DevStack FAQ and overview docs; §9.2 |
| A first `stack.sh` run takes 15–30 minutes, "largely depending on the speed of your internet connection" | ✅ Verified | DevStack quick start; §1.1, §3.1 |
| Supported base OS: the two latest Ubuntu LTS releases plus the current dev release and the current RHEL-family major (Rocky Linux 9 and openEuler named); Ubuntu 24.04 Noble is "the most tested" | ✅ Verified | Quick start and overview docs; §1.2 |
| `stack.sh` refuses to run as root and expects a dedicated sudo user (the docs' example: `stack`) | ✅ Verified | Repo README; §3.1 |
| `local.conf` is "a modified INI format" with `[[phase|file]]` meta-sections; its content is sourced as a shell fragment (no spaces around `=`); passwords must be alphanumeric | ✅ Verified | Configuration docs; §3.2–3.3 |
| The four passwords (`ADMIN_PASSWORD`, `DATABASE_PASSWORD`, `RABBIT_PASSWORD`, `SERVICE_PASSWORD`) preset by the quick start; `stack.sh` prompts interactively when they are unset | ✅ Verified | Quick start and configuration docs; §3.3 |
| The stack.sh / unstack.sh / clean.sh cycle: stop with `unstack.sh`, wipe with `clean.sh` (which calls `unstack.sh` first), redeploy with `stack.sh` after `local.conf` edits | ✅ Verified | Plugin docs, FAQ, development docs; §3.4, §8.1 |
| Services run as systemd units (`devstack@<service>.service`) by default; screen was the model "when the number of services ... was typically < 10" (docs dated 2020 document the migration) — the brief's "systemd since Xena" phrasing is not how the primary sources put it | ✅ Verified | Systemd docs, "Why this instead of screen?"; §3.5 |
| A default install brings up keystone, glance, nova, placement, cinder, neutron and horizon — seven services, Horizon among them — with compute on the host hypervisor (KVM, or QEMU in a VM) and Cinder LVM-backed | ✅ Verified | Docs index quick-start roster; guides and roster docs; §4.1–4.2 |
| The overview roster still names Swift, which the quick-start roster omits; Swift/Heat/Octavia are opt-in in current practice | ⚠ Reported-approximate | The two official pages disagree; §4.2 |
| Default networking is isolated to the host: OVS plus the L3 agent, private project networks, floating range `172.24.4.0/24` (gateway `172.24.4.1`), `br-ex` "not connected to any physical interfaces," and an egress masquerade rule | ✅ Verified | Networking docs, "Defaults"; §4.3 |
| Security groups block all inbound by default; ICMP and TCP/22 rules must be added before guests answer SSH or ping | ✅ Verified | Networking docs; §4.3 |
| Extensibility has two layers: `extras.d/` in-tree hook scripts and an out-of-tree plugin contract (`override-defaults`, `settings`, `plugin.sh`) whose mode/phase call points are "strong contracts" | ✅ Verified | Contributing docs and plugins docs; §5 |
| Release naming: A–Z codenames ran Austin (2010) through Zed (2022.3); calendar versioning `YYYY.N` began with 2023.1 Antelope — correcting the brief's "started with 2023.2 Bobcat" framing — and 2026.1 Gazpacho (1 April 2026, the 33rd version) is the current release | ✅ Verified | governance.openstack.org and releases.openstack.org; §6.1 |
| DevStack master tracks trunk; stable branches exist from Diablo onward and match the corresponding OpenStack stable release (`stable/2026.1` for Gazpacho); custom `*_BRANCH` pinning is untested territory | ✅ Verified | Repo README and FAQ; §6.2 |
| Tempest is "the OpenStack Integration test suite," installed by default and pre-configured at `/opt/stack/tempest` — "Tempest tests will work in this environment" | ✅ Verified | DevStack Tempest page and Tempest project docs; §7.3 |
| OpenStack launched October 2010 by Rackspace and NASA; the first release, Austin, shipped 21 October 2010; Nova came from NASA's Nebula project | ✅ Verified | Cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) claims ledger (Rackspace history, The Register, NASA tech-transfer); §2.1 |
| The Devon/Stacker naming anecdote (stack.sh's author and the tool's name) | ⚠ Anecdotal | Community lore; no primary source found this pass; §2.3 |
| A DevStack history page exists at `docs.openstack.org/devstack/latest/history.html` | ❌ Failed-to-verify | Returns 404 on the latest, 2024.1 and Zed documentation generations; the page does not exist; §2.3 |
| The OpenInfra wiki DevStack history page is readable | ❌ Failed-to-verify | `wiki.openstack.org/wiki/DevStack` sits behind an anti-bot proof-of-work wall; §2.3 |
| Resource guidance: "DevStack ... will perform best with 4GB or more of RAM" is the docs' only explicit floor; no official disk minimum; the common 8 GB RAM / 4 vCPU recipe is community advice | ⚠ Soft guidance | Single-VM guide (4 GB line) versus community tutorials; §9.1 |
| TripleO is retired, with repositories carrying opendev RETIRED banners; ecosystem timing points to the Epoxy (2025.1) cycle | ✅ Verified (retired); ⚠ Reported-approximate (timing) | opendev banners (undated) plus secondary sources; §10.2 |
| MicroStack is an installable snap-based single-machine OpenStack, but Canonical's product energy has moved to Sunbeam; MicroStack's current maintenance status | ✅ Verified (existence); ⚠ Reported-approximate (status) | OpenInfra Superuser publication; status not pinned at a primary source; §10.1 |
| Troubleshooting folklore from §8.2's ledger: RabbitMQ failures on fresh VMs trace to erlang/hostname issues (fix `/etc/hosts`; extreme case: `clean.sh`) and the prereq check is timer-gated (`PREREQ_RERUN_HOURS`, `FORCE_PREREQ=1`) — while the brief's "FORCE=yes"-style forced re-clone does not exist as stated (documented equivalents: `RECLONE=yes`, `FORCE_PREREQ=1`) | ✅ Verified (FAQ items); ❌ Failed-to-verify (FORCE=yes as stated) | DevStack FAQ; §8.2 |

### 14.1 What Could Not Be Verified

Checked and not confirmable — or only partially confirmable — during the September 2026 research pass:

- **A DevStack "history page" (❌).** The URL the brief expected, `docs.openstack.org/devstack/latest/history.html`, returns 404, as do the same paths on the 2024.1 and Zed documentation generations. No project-history narrative page exists in the current or recent DevStack docs (§2.3).
- **The Devon/Stacker name anecdote (⚠).** No interview, early commit message or archive documenting the naming surfaced this pass; the anecdote should be retold as color, not fact. The verified kernel is only what §2.2 records: an early Rackspace-era tool, present in the release machinery by the Diablo cycle (2011), designed as "an explanation as much as an install script" (§2.3).
- **The OpenInfra wiki page (❌).** `wiki.openstack.org/wiki/DevStack`, which once carried DevStack history, sits behind an anti-bot proof-of-work wall and could not be read this pass (§2.3).
- **The early-hosting Rackspace association (⚠).** The cloudbuilders GitHub mirror and its banner are verified, but the inference that DevStack began as Rackspace cloud-builders tooling is reported-approximate — no primary source this pass tied the tool's earliest hosting to the Rackspace team (§2.2).
- **Resource minimums (⚠).** The only explicit memory guidance in the core docs is the single-VM guide's "4GB or more of RAM" line. No official disk minimum is stated (tens of GB are implied by the git-tree model), and the widely repeated 8 GB RAM / 4 vCPU recipe lives in community tutorials, not the official docs (§9.1).
- **TripleO retirement timing (⚠).** The opendev RETIRED banners are undated; the Epoxy (2025.1)-cycle timing and the Red Hat OpenStack Services on OpenShift successor direction come from secondary sources (§10.2).
- **MicroStack's current maintenance status (⚠).** MicroStack's purpose is documented at OpenInfra's Superuser publication and it remains installable, but a primary-source statement of its present maintenance status could not be pinned down; Canonical's energy has visibly moved to Sunbeam (§10.1).
- **The "FORCE=yes" forced re-clone folklore (❌).** Not confirmable at a primary source; the documented equivalents are `RECLONE=yes` (freshen all repos) and `FORCE_PREREQ=1` (force the prereq re-check) (§8.2).
- **Environment-side failure lore (⚠).** Corporate-proxy behavior, the TLS "insecure flag" client practice, and network-namespace/OVS oddities after host reboots or under nested virtualization are real in the field but live in community channels, not the official docs pages (§8.2).
- **The `dtroyer` attribution (⚠).** Identifying the code-comment handle in `lib/tls` with Dean Troyer is inference from those comments plus the docs' own link to his shocco fork — not a biographical source (§2.2).
- **The Tempest smoke-selection flag (⚠).** `tempest run --smoke` follows the Tempest CLI convention; it is not a DevStack-doc quote (§11.4).
- **PackStack's Puppet internals (⚠).** The Puppet-under-the-hood detail is widely documented but was not re-verified at a primary source this pass (§10.1).

---

*End of guide — Author: Jack Liu Shurui, Solution Architect, Cymbal Bank. September 2026.* DevStack: the stacked lab.
