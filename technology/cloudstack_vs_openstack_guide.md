# Apache CloudStack vs OpenStack — an integration project and a product

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — Infrastructure / Private-Cloud series; the direct head-to-head between Apache CloudStack (an Apache Software Foundation IaaS project) and OpenStack (an OpenInfra Foundation IaaS project), compared on a single, deliberately narrow axis: CloudStack's **single management server with agents** versus OpenStack's **set of cooperating services behind a shared API**
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** Apache CloudStack official site and project documentation (`cloudstack.apache.org`, `docs.cloudstack.apache.org` — the 4.23.0.0 documentation set and the site's history, users, downloads, who, and Kubernetes pages; all retrieved 2 October 2026); the OpenStack project-team guide's "A Bit of OpenStack History"; OpenInfra Foundation and Linux Foundation published materials (retrieved 2 October 2026); sibling guides in this repository, cross-referenced by name
> **Last Updated:** October 2026

---

## Table of Contents

1. [Overview — The Trap, the Decoder, the Thesis](#1-overview--the-trap-the-decoder-the-thesis)
2. [Each Project in One Page](#2-each-project-in-one-page)
3. [The Architectural Difference — Management Server vs Integrated Services](#3-the-architectural-difference--management-server-vs-integrated-services)
4. [The CloudStack Side in Depth](#4-the-cloudstack-side-in-depth)
5. [The OpenStack Side as a Cross-Reference](#5-the-openstack-side-as-a-cross-reference)
6. [The History and Governance Comparison](#6-the-history-and-governance-comparison)
7. [The Operational Reality](#7-the-operational-reality)
8. [The Capability Comparison](#8-the-capability-comparison)
9. [The Ecosystem](#9-the-ecosystem)
10. [Adoption and Workload Fit](#10-adoption-and-workload-fit)
11. [The Bank and Enterprise Angle](#11-the-bank-and-enterprise-angle)
12. [How to Decide — A Framework, Not a Verdict](#12-how-to-decide--a-framework-not-a-verdict)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, Glossary and References](#16-what-could-not-be-verified-glossary-and-references)

### How to Read This Guide

This guide sits in the `technology/` Infrastructure / Private-Cloud series, and it is deliberately **not** a re-run of the comparisons this repository has already published. Two sibling guides own the OpenStack material that a naive "CloudStack vs OpenStack" article would duplicate, and this guide cross-references them by name instead of restating them:

- **[technology/nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md)** owns the **OpenStack side of a comparison** — its §3.1 gives OpenStack's origins (Rackspace + NASA, 2010), its §3.2 gives the component roster (Nova, Neutron, Cinder, Glance, Keystone, Swift, Heat), its §3.3 gives the OpenStack component table, its §3.6 gives the release timeline, and its §4 runs the head-to-head on the **HCI-versus-disaggregated** axis (Nutanix's converged product versus OpenStack's disaggregated service layer). Its §5 covers the OpenStack commercial distributions.
- **[technology/devstack_openstack_guide.md](devstack_openstack_guide.md)** owns **DevStack**, the all-in-one OpenStack lab topology, and the OpenStack service roster as deployed on a single host — its §4.2 gives the service roster and its §6 gives the release naming / calendar versioning.

**What this guide owns is different.** Nothing in this repository previously covered Apache CloudStack at all (`cloudstack` appeared in zero of the repo's 646 files before this guide), and no guide runs the direct **CloudStack-versus-OpenStack** comparison. So this guide carries CloudStack in full, and runs the comparison on a **different axis**: not "converged appliance versus disaggregated services" (that is the Nutanix guide's axis) and not "how do you stand up a lab" (that is the DevStack guide's axis), but **CloudStack's management-server-and-agent model versus OpenStack's integrated-services model** — where the control plane lives, what an operator has to run, and how the two scale.

**A note on tone.** This guide names no winner. §12 gives a decision framework with named inputs; §13 works a fictional Cymbal Bank example through that framework; neither produces a "recommended platform". That restraint is not hedging — §12 explains why a verdict would be dishonest.

**A note on verification.** Claims are marked inline as ✅ **Verified** (confirmed against the primary source named), ⚠️ **Flagged** (reported or approximate, or true but dated/disputed), or ❌ **Rejected / not established** (could not be confirmed and therefore not asserted). §15 collects every CloudStack claim into an audit table with its source and date; §16 lists what could not be verified.

**A note on the words.** Two collisions of vocabulary make this topic harder than it should be, and §1 names them explicitly before anything else: **OpenStack is not OpenShift**, and **Apache CloudStack is not "a cloud stack"** in the generic sense. If you take one thing from this guide, take the decoder in §1.3.

---

## 1. Overview — The Trap, the Decoder, the Thesis

### 1.1 The trap, named

Before a single architectural fact is stated, one hazard must be nailed down, because in *this repository* the numbers make the trap statistically likely:

> **HAZARD 1 — OpenShift is not OpenStack.** In this repository the string `openshift` appears in **64 files** and `openstack` in **18**. A reader who is skimming will conflate two products that share four letters and nothing else.

**Red Hat OpenShift** is a Kubernetes-based **container platform** — an application runtime and container orchestration system, with its own security model (SCCs), its own packaging, and its own virtualisation story (OpenShift Virtualization). **OpenStack** is an **infrastructure-as-a-service (IaaS) control plane** — software that turns pools of servers, storage and network into self-service virtual-machine infrastructure. One schedules containers; the other provisions virtual machines and the storage and networks they attach to. They are not versions of each other, not forks of each other, and not substitutes for each other. A bank can, and sometimes does, run both — often with OpenStack underneath and OpenShift on top — but it never "picks OpenStack instead of OpenShift" as if they were alternatives in the same category.

(Sibling guides in this repo already cover the OpenShift side of things — [openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md), [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md). This guide never treats OpenShift as the thing OpenStack is being compared against. Where OpenShift appears below, it appears only because a specific OpenStack *distribution* happens to run on top of it — and that is labelled as such.)

### 1.2 The other trap, named

> **HAZARD 2 — "CloudStack" is not a generic phrase.** "Cloud stack" is a common, generic English phrase meaning "the whole set of software that makes a cloud". **Apache CloudStack** is a *specific named project* of the Apache Software Foundation. When someone says "our cloud stack", they do not mean this project, and when someone says "we evaluated CloudStack", they should. Every use of the bare word **CloudStack** in this guide means the Apache project.

### 1.3 The decoder — terms used in this guide

The two projects use some of the same words to mean different things, and some of their words are unique to them. Fixed definitions for the length of this guide:

- **The IaaS layer** — the software tier that provisions infrastructure *as a service*: virtual machines, the block storage behind them, the virtual networks between them, and the images they boot from, all driven by an API. Both CloudStack and OpenStack deliver this layer.
- **The control plane** — the part of the platform that decides *what* to run and *where*: it receives an API call, picks a host, hands out an IP, allocates storage, records usage. It is distinct from the **data plane**, which is the actual traffic between the workload and its storage and network.
- **The compute service, the storage service, the network service** — in **OpenStack**, these are *separate projects with separate APIs* (Nova for compute, Cinder and Swift for block and object storage, Neutron for networking — see [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2). In **CloudStack**, these are *capabilities of one orchestrator*: the management server allocates compute, storage and network as one function.
- **The management server** — CloudStack's term for its central orchestrator. It runs in an Apache Tomcat container, needs a MySQL database, exposes the web UI and the API, and directs the cloud's resources ✅ **Verified** (CloudStack concepts documentation, §4).
- **The agent** — CloudStack's `cloudstack-agent`, a process that runs **on each hypervisor host** and executes the management server's instructions locally (start this VM, attach this volume). It connects to the management server on port 8250 ✅ **Verified** (CloudStack hosts documentation, §4).
- **The system VM** — a small, CloudStack-managed virtual machine that provides a cloud *service to the cloud itself*: console access (the console proxy VM), template and snapshot processing (the secondary storage VM), and routing/DHCP/NAT/firewall for guest networks (the virtual router). It is not a user workload ✅ **Verified** (CloudStack system-VM documentation, §4).
- **The zone / pod / cluster / host** — CloudStack's physical-organisation hierarchy, from largest to smallest. A **region** contains **zones**; a zone contains **pods**; a pod contains **clusters**; a cluster contains **hosts**. Stated as nesting: **region ⊃ zone ⊃ pod ⊃ cluster ⊃ host** ✅ **Verified** (CloudStack concepts documentation, §4).
- **The service offering** — CloudStack's packaged description of *what a user is allowed to get*: a compute offering (CPU, RAM, tags), a disk offering (size, IOPS), a network offering (which network features are available) ✅ **Verified** (CloudStack service-offerings documentation, §4).
- **The hypervisor** — the layer that actually runs virtual machines on a host: KVM, VMware ESXi/vSphere, XenServer, Hyper-V, and others. Both projects sit *above* the hypervisor and drive it; neither replaces it.
- **The image / the template** — the bootable disk a virtual machine is created from. **OpenStack** calls it an **image** (managed by Glance); **CloudStack** calls it a **template** (stored on secondary storage and registered per zone).
- **The catalogue** — the discoverable list of what a user may instantiate: in OpenStack, the image list and the flavor list; in CloudStack, the template list plus the service offerings. The catalogue is a *governance surface* — it is where a platform team decides what the business is allowed to build.
- **The distribution** — a vendor's packaged, supported, tested build of an open-source project. OpenStack is almost always consumed through a distribution; CloudStack is more often consumed as the Apache release plus optional support.

### 1.4 The thesis

> **OpenStack is an integration project and CloudStack is a product; the difference you feel is the difference you staff.**

That is the one-sentence thesis, and it is also the guide's closing line. Everything between here and there exists to justify it with evidence rather than assertion.

The shape of the argument:

- **OpenStack** is, by its own governance documents, a *set of cooperating projects* — born in 2010 from Rackspace wanting to rewrite its cloud and NASA contributing beta Nova code, and later refactored (December 2014, the "big tent") into "a community-centric definition of OpenStack" ✅ **Verified** (OpenStack project-team guide, §6). Its architecture expresses that history: many services, each with an API, loosely coupled behind a shared identity and catalogue.
- **CloudStack** is, by its own documentation, a *turnkey* system — "a turnkey solution that includes the entire 'stack' of features most organizations want with an IaaS cloud" ✅ **Verified** (cloudstack.apache.org, §2). Its architecture expresses *that*: a single management server that orchestrates everything, thin agents on the hosts, and a handful of system VMs for the plumbing.

Both are real architectures with real trade-offs. The claim is *not* that one is better. The claim is that they demand **different operator shapes**, and that a bank choosing between them is really choosing **which kind of team and which kind of failure it is prepared to own**. That is what §7, §12 and §13 are for.

### 1.5 The boundary — declared by name

To keep this guide honest about what it is adding, here is the explicit division of labour with its siblings:

| Material | Where it lives | What this guide does |
|---|---|---|
| OpenStack origins (Rackspace + NASA, 2010) | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.1 | Cross-reference only — one paragraph in §6 |
| OpenStack component roster (Nova, Neutron, Cinder, Glance, Keystone, Swift, Heat, Horizon, Ceilometer) | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2 | Cross-reference only — §5 |
| OpenStack release timeline | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.6 | Cross-reference only — §5, §6 |
| The Nutanix-vs-OpenStack head-to-head (HCI vs disaggregated) | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §4 | Not re-derived |
| DevStack, the all-in-one lab topology, the service roster on one host | [devstack_openstack_guide.md](devstack_openstack_guide.md) §4.2, §6 | Cross-reference only — §5 |
| OpenStack commercial distributions (Red Hat, Canonical, Mirantis) | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5 | Cross-reference, named factually — §9 |
| **CloudStack, in full** | **This guide** §2, §4 | The repo's first and only CloudStack coverage |
| **CloudStack-vs-OpenStack on the management-server-vs-integrated-services axis** | **This guide** §3, §6–§8 | The repo's only coverage of this comparison |

Everything this guide states about CloudStack is drawn from Apache's own documentation or the project's own published material, dated. Everything it states about OpenStack beyond the fact pack is cross-referenced to the sibling guides rather than re-argued.

---

## 2. Each Project in One Page

This section states what each platform *is*, symmetrically enough to be fair, but with the OpenStack half kept deliberately short and pointed at the sibling guides. The CloudStack half is developed fully because nothing else in this repository covers it.

### 2.1 Apache CloudStack, in one page

**Identity.** Apache CloudStack is an open-source Infrastructure-as-a-Service platform, developed as a project of the **Apache Software Foundation** ✅ **Verified** (cloudstack.apache.org and the project's own site footer/trademark notices, retrieved 2 October 2026). The project's own one-paragraph description is worth quoting in full because every later section is a consequence of it:

> "Apache CloudStack is open-source software designed to deploy and manage large networks of virtual machines, as a highly available, highly scalable Infrastructure as a Service (IaaS) cloud computing platform. CloudStack is used by a number of service providers to offer public cloud services, and by many companies to provide an on-premises (private) cloud offering, or as part of a hybrid cloud solution." ✅ **Verified** (cloudstack.apache.org home page, retrieved 2 October 2026)

And its own claim about *what kind of thing it is*:

> "CloudStack is a turnkey solution that includes the entire 'stack' of features most organizations want with an IaaS cloud: compute orchestration, Network-as-a-Service, user and account management, a full and open native API, resource accounting, and a first-class User Interface (UI)." ✅ **Verified** (cloudstack.apache.org home page, retrieved 2 October 2026)

That word **turnkey** is the load-bearing one. It is why the thesis calls CloudStack "a product": not because it is proprietary (it is Apache-2.0 licensed), but because it presents itself as *one assembled thing you deploy*, rather than a set of projects you assemble.

**What it does.** CloudStack manages and orchestrates "pools of storage, network, and computer resources to build a public or private IaaS compute cloud" ✅ **Verified** (CloudStack concepts documentation, 4.23.0.0, retrieved 2 October 2026). Concretely, that means it allocates virtual machines onto hosts, hands out public and private IP addresses, allocates storage during instance creation, and manages snapshots, templates and ISO images ✅ **Verified** (CloudStack concepts documentation, management-server section).

**Interfaces.** Three ways in: a browser-based web UI, command-line tools (CloudMonkey, `cmk`), and "a full-featured RESTful API" — the concepts documentation calls it "a REST-like API for the operation, management and use of the cloud" ✅ **Verified** (cloudstack.apache.org and CloudStack concepts documentation, retrieved 2 October 2026). CloudStack additionally documents "an EC2 API translation layer to permit the common EC2 tools to be used in the use of a CloudStack cloud", and a home-page statement that its API "is compatible with AWS EC2 and S3 for organizations that wish to deploy hybrid clouds" ✅ **Verified** (as documented) — but see §16: whether that EC2 translation layer is *actively maintained* today is ❌ **not established** from the documentation, so this guide reports it as *documented*, not as *currently maintained*.

**Scale claim.** From the project's own material: "CloudStack can manage tens of thousands of physical servers installed in geographically distributed data centers. It is a powerful IaaS management solution, but it is still easy to use and implement with a small team." ✅ **Verified** (cloudstack.apache.org home page, retrieved 2 October 2026). The concepts documentation adds: "The management server scales near-linearly eliminating the need for cluster-level management servers. Maintenance or other outages of the management server can occur without affecting the virtual machines running in the cloud." ✅ **Verified** (CloudStack concepts documentation).

**Current release.** The site banner at retrieval read "Apache CloudStack 4.23.0.0 is out! This is the latest Regular release." The current LTS release was **4.22.1.1** ✅ **Verified** (cloudstack.apache.org and the downloads page, retrieved 2 October 2026, §4.7).

**Licence and home.** Apache-2.0 licensed; source at `github.com/apache/cloudstack`; governed by an Apache Project Management Committee (§6).

### 2.2 OpenStack, in one page (short — cross-referenced)

**Identity.** OpenStack is an open-source **IaaS** platform: a collection of cooperating cloud services (Nova compute, Neutron networking, Cinder block storage, Glance images, Keystone identity, Swift object storage, Heat orchestration, and others) governed by the **OpenInfra Foundation** ✅ **Verified** (OpenStack project-team guide; OpenInfra Foundation materials, retrieved 2 October 2026).

**Origins and components — see the siblings, not here.** OpenStack's origins (created in the first months of 2010, Rackspace rewriting its cloud offering, Anso Labs' beta Nova code for NASA, the first Design Summit 13–14 July 2010, announced at OSCON 21 July 2010) are covered in [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.1. Its component roster is covered in that guide's §3.2 and §3.3; its release timeline in §3.6. The service roster as deployed by DevStack on a single host is covered in [devstack_openstack_guide.md](devstack_openstack_guide.md) §4.2, and release naming/calendar versioning in its §6. **This guide does not restate any of it.**

**What this guide needs from OpenStack.** Three properties only, each developed where it matters:

1. It is **many services behind a shared API** — the architectural premise of §3.
2. It is **governed by a foundation/consortium with a commercial ecosystem** — the premise of §6 and §9.
3. It is **consumed almost always through a distribution** — the premise of §9 and §11.

### 2.3 The symmetry, stated plainly

Laid side by side, in the terms each project uses about itself:

| | Apache CloudStack | OpenStack |
|---|---|---|
| **Self-description** | "a turnkey solution that includes the entire 'stack' of features" ✅ | "a set of cooperating projects… open source cloud computing software" ✅ (project-team guide) |
| **Foundation** | The Apache Software Foundation (a meritocratic foundation) ✅ | The OpenInfra Foundation (a foundation/consortium with corporate membership tiers) ✅ |
| **Governance body** | Apache PMC + PMC Chair ✅ | Foundation Board + Technical Committee ✅ |
| **Component model** | one orchestrator (management server) + agents + system VMs ✅ | many services, each with an API ✅ |
| **Origin** | VMOps, 2008 → Cloud.com → Citrix → Apache (2012) ✅ | Rackspace + NASA, 2010 ✅ |
| **License** | Apache-2.0 ✅ | Apache-2.0 ✅ |

The two rows that matter for the rest of this guide are the **component model** and the **governance body**. The first is §3; the second is §6.

**Where the comparison runs.** Because both are Apache-2.0 IaaS platforms, the naive comparison degenerates into a feature matrix, and a feature matrix is exactly the wrong instrument (§14, first anti-pattern). This guide instead compares the two *control-plane shapes* and the *staffing and governance consequences* that flow from them.

---

## 3. The Architectural Difference — Management Server vs Integrated Services

This is the guide's core. Everything else in the document is downstream of the difference stated here.

### 3.1 CloudStack's control plane in one paragraph

CloudStack's control plane is **one orchestrator**. The management server "orchestrates and allocates the resources in your cloud deployment"; it typically runs on a dedicated machine or VM; it "runs in an Apache Tomcat container and requires a MySQL database for persistence" ✅ **Verified** (CloudStack concepts documentation, 4.23.0.0). It provides the web UI and the API, manages assignment of guest VMs to compute resources, manages public and private IP addresses, allocates storage during VM instantiation, manages snapshots, templates and ISOs, and provides "a single point of configuration for the cloud" ✅ **Verified** (CloudStack concepts documentation).

On each hypervisor host, a **thin agent** (`cloudstack-agent`) carries out the management server's instructions locally. In **CloudStack's own words**, the cloud also runs "a pool of virtual appliances [that] support the operation of configuration of the cloud itself. These appliances offer services such as firewalling, routing, DHCP, VPN, console proxy, storage access, and storage replication." ✅ **Verified** (CloudStack concepts documentation). Those appliances are the **system VMs** (§4.4).

So the CloudStack control plane is three kinds of thing: **one management server (plus its MySQL database)**, **agents on hosts**, and **a small, bounded set of system VMs**.

### 3.2 OpenStack's control plane in one paragraph

OpenStack's control plane is **a set of cooperating services**. Nova schedules and runs compute; Neutron builds networks; Cinder brokers block storage; Glance stores images; Keystone issues identity and the service catalogue; Swift provides object storage; Heat orchestrates templates ✅ **Verified** (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2, where the roster and each service's function are set out in full). Each is its own project with its own API and its own database schema; a request that creates a VM with a network and a volume *traverses several services*. The DevStack guide shows the roster on a single host ✅ (cross-ref [devstack_openstack_guide.md](devstack_openstack_guide.md) §4.2).

The consequence for the control plane is: OpenStack is **one logical control plane built from many independently-versioned services**, whereas CloudStack is **one process group that already contains them**.

### 3.3 The head-to-head table

| Dimension | Apache CloudStack | OpenStack |
|---|---|---|
| **Control-plane unit** | One management server (Tomcat + MySQL) ✅ | A set of services, each with its own API and DB ✅ |
| **How hosts are driven** | An agent per host, connecting to the management server on port 8250 ✅ | Each service talks to the compute host / hypervisor via its own driver model ✅ |
| **Where the "plumbing" runs** | System VMs (console proxy, secondary storage, virtual router) ✅ | Services managed by the operator, plus network agents (e.g. Neutron agents) ✅ |
| **State** | Management-server MySQL database; per-host agent state is minimal ✅ | A database per service (Nova's, Neutron's, Cinder's, Keystone's, …) ✅ |
| **Failure of the control plane** | No new instances; running instances keep running; UI/API/dynamic load distribution/HA stop ✅ | Depends which service failed: identity down can block everything that authenticates; compute down stops scheduling but running VMs persist ✅ |
| **Scaling the control plane** | Add management servers behind a load balancer; MySQL replication ✅ | Scale each service independently ✅ |
| **Extending the platform** | Via its API, plugins, and the Apache project's release cycle ✅ | By adding services/projects, and by the many upstream projects ✅ |

The row that a bank should read twice is **"where state lives"**. CloudStack concentrates control-plane state in the management-server MySQL database (with replication for HA) ✅ **Verified** (CloudStack reliability documentation). OpenStack distributes it across a database per service ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2, §4.2). Both are the same *kind* of risk — stateful control planes need backups and HA — but the *number* of stateful things to protect is one versus many.

### 3.4 The honest consequences — components an operator must run

Stated without editorialising, from each project's own documentation:

- **CloudStack**: the operator runs the management server (ideally a multi-node, load-balanced set), a MySQL database (ideally replicated), the hypervisor hosts with the cloudstack-agent, and gets the system VMs automatically — "CloudStack manages these system VMs and creates, starts, and stops them as needed based on scale and immediate needs" ✅ **Verified** (CloudStack system-VM documentation). The component count is **small and bounded**.
- **OpenStack**: the operator runs Nova, Neutron, Cinder, Glance, Keystone, a message bus (e.g. RabbitMQ in a typical DevStack-class deployment), a database (per-service or shared), plus the network agents and the placement/scheduler components ✅ (cross-ref [devstack_openstack_guide.md](devstack_openstack_guide.md) §4.2 and [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2). The component count is **larger and grows with the services you adopt**.

This is not a claim that one is "simpler" in a pejorative sense. It is the direct expression of §3.1 and §3.2: a turnkey system ships with its components **already integrated**; an integration project ships them **as components you integrate**.

### 3.5 What a failure looks like — the honest picture for each

CloudStack states its partial-failure behaviour very precisely, and it is worth reporting in its own words because it is one of the clearest failure-mode statements either project publishes:

> "The Management Server itself (as distinct from the MySQL database) is stateless and may be placed behind a load balancer." … "Normal operation of Hosts is not impacted by an outage of all Management Servers. All Guest Instances will continue to work." … "When the Management Server is down, no new Instances can be created, and the end User and admin UI, API, dynamic load distribution, and HA will cease to work." ✅ **Verified** (CloudStack reliability documentation, retrieved 2 October 2026)

So CloudStack's failure decomposition is **clean and predictable**: the running workload is unaffected; the *management* functions stop. The database is the stateful part that must be protected; the management server is stateless and can be load-balanced ✅ **Verified**.

OpenStack's failure decomposition is **per-service**. Because each service has its own process and database, a Nova outage stops scheduling but not running VMs; a Keystone outage can stop everything that authenticates; a Neutron outage affects network changes and can affect new networking. The sibling guide covers the operational comparison on the HCI axis ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §4.3). The point here is structural: **a single-orchestrator model localises the blast radius of a control-plane failure to "you cannot change the cloud"**, while a **multi-service model localises it per service but multiplies the number of failure modes to reason about**.

Neither is inherently safer. They fail *differently*, and a bank's incident-response runbook is written for one shape or the other.

### 3.6 How the two models scale — differently, not one better

This is the part that a lazy comparison gets wrong. The two models do not sit on a single "scalability" axis; they scale along *different* axes.

- **CloudStack scales the control plane by linear addition.** The project states the management server "scales near-linearly eliminating the need for cluster-level management servers" ✅ **Verified** (CloudStack concepts documentation), and documents multi-node management servers behind a load balancer with MySQL replication for database failover ✅ **Verified** (CloudStack reliability documentation). The scaling question is essentially a *database* question: as the cloud grows, the management-server MySQL database becomes the thing to size, tune, and replicate.
- **OpenStack scales the control plane by decomposition.** Each service scales on its own terms; you add Nova conductors, Neutron agents, Cinder volume services independently ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §4.2). The scaling question is a *coordination* question: keeping many independently-scaled services mutually consistent.

**The honest conclusion:** if your growth is "more of the same VM estate", CloudStack's near-linear management-server model matches it directly. If your growth is "more *kinds* of service" — object storage, containers, bare metal, specialised networking — OpenStack's decomposition matches it directly, because you bolt on the service rather than wait for the vendor/project to add the capability. The axis on which each is "easier to scale" is **not the same axis**.

### 3.7 The one-line summary of §3

> CloudStack puts the whole control plane in one place and makes the *hosts* thin; OpenStack puts the control plane in many places and makes the *integration* the product. The first is easier to reason about on day one; the second is easier to grow sideways. §7 turns this into staffing; §12 turns it into a decision.

---

## 4. The CloudStack Side in Depth

All facts in this section are from Apache CloudStack's own documentation and site, retrieved **2 October 2026**, and describe the **4.23.0.0** documentation set. Every claim carries its source and status inline; the consolidated audit is §15.

### 4.1 The deployment hierarchy — exactly as documented

CloudStack organises physical infrastructure in five nested levels. This is its most distinctive concept and it must be reproduced exactly. From the concepts documentation (each quote ✅ **Verified**, CloudStack concepts documentation, retrieved 2 October 2026):

**Region — the largest organisational unit.**
> "A region is the largest available organizational unit within a CloudStack deployment. A region is made up of several availability zones, where each zone is roughly equivalent to a datacenter. Each region is controlled by its own cluster of Management Servers, running in one of the zones."

Zones inside a region are "typically located in close geographical proximity"; regions give fault tolerance and disaster recovery; "User accounts can span regions"; usage records "can also be consolidated and tracked at the region level"; and "Regions are visible to the end user."

**Zone — the second largest.**
> "A zone typically corresponds to a single datacenter, although it is permissible to have multiple zones in a datacenter." … "A zone consists of: One or more pods…; A zone may contain one or more primary storage servers, which are shared by all the pods in the zone; Secondary storage, which is shared by all the pods in the zone."

Zones "can be public or private" and "are visible to the end user". Hosts in the same zone are directly reachable; hosts in different zones reach each other "through statically configured VPN tunnels".

**Pod — the third largest.**
> "A pod often represents a single rack. Hosts in the same pod are in the same subnet… A pod consists of one or more clusters of hosts and one or more primary storage servers." … "Pods are not visible to the end user."

**Cluster — the fourth largest.**
> "a cluster is a XenServer server pool, a set of KVM servers, or a VMware cluster preconfigured in vCenter. The hosts in a cluster all have identical hardware, run the same hypervisor, are on the same subnet, and access the same shared primary storage."

VMs "can be live-migrated from one host to another within the same cluster, without interrupting service to the user". "Even when local storage is used exclusively, clusters are still required organizationally, even if there is just one host per cluster". "When VMware is used, every VMware cluster is managed by a vCenter server. An Administrator must register the vCenter server with CloudStack."

**Host — the smallest.**
> "A host is a single computer. Hosts provide the computing resources that run guest virtual machines. Each host has hypervisor software installed on it to manage the guest VMs." … "The host is the smallest organizational unit within a CloudStack deployment."

"hosts within a cluster must all be homogeneous". "Hosts are not visible to the end user."

**The nesting order, stated as the guide's fixed reference:** **region ⊃ zone ⊃ pod ⊃ cluster ⊃ host**. Visibility to the end user is deliberately partial: **regions and zones are visible**, while **pods, clusters and hosts are not**.

### 4.2 The management server

> "The management server orchestrates and allocates the resources in your cloud deployment." It "typically" runs "on a dedicated machine or VM", "runs in an Apache Tomcat container and requires a MySQL database for persistence" ✅ **Verified**.

Its documented responsibilities (each ✅ **Verified**): provides the web UI for administrator and end user; provides the API interfaces for both the CloudStack API and the EC2 interface; manages assignment of guest VMs to compute resources; manages assignment of public and private IP addresses; allocates storage during VM instantiation; manages snapshots, disk images (templates) and ISO images; provides "a single point of configuration for the cloud"; and "scales near-linearly eliminating the need for cluster-level management servers".

The smallest possible deployment is documented too: "one machine running the Management Server + another as cloud infrastructure; in the smallest deployment a single machine can act as both Management Server and hypervisor host (KVM)" ✅ **Verified**. That single-machine option is why CloudStack can be *learned* cheaply even though it is *operated* at scale.

### 4.3 Management-server high availability

From the reliability documentation ✅ **Verified** (CloudStack reliability documentation, retrieved 2 October 2026):

> "The CloudStack Management Server should be deployed in a multi-node configuration such that it is not susceptible to individual server failures. The Management Server itself (as distinct from the MySQL database) is stateless and may be placed behind a load balancer."

Failure semantics, in the project's own words:

> "Normal operation of Hosts is not impacted by an outage of all Management Servers. All Guest Instances will continue to work."
> "When the Management Server is down, no new Instances can be created, and the end User and admin UI, API, dynamic load distribution, and HA will cease to work."

The documented **load-balancer port rules** (✅ Verified): 80 or 443 → 8080 (or 20400 with AJP), HTTP/AJP, persistence **required**; 8250 → 8250, TCP, persistence **required**; 8096 → 8096, HTTP, persistence **not required**.

One documented gotcha worth recording, because it is a common self-inflicted outage: the administrator must set the `host` global configuration value from the management-server IP to the **load-balancer VIP**; "If the 'host' value is not set to the VIP for Port 8250 and one of your management servers crashes, the UI is [still] available" but "the system VMs cannot contact the management server" ✅ **Verified**.

**Database HA.** CloudStack provides database replication "using the MySQL connector parameters and two-way replication", with chain replication for more than two nodes; the documentation states it was "Tested with MySQL 5.1 and 5.5" ✅ **Verified** — ⚠ **freshness flag:** that test matrix is long superseded, so a 2026 operator should confirm the current supported MySQL/MariaDB matrix before relying on the exact recipe. Configuration lives in `/etc/cloudstack/management/db.properties` with settings including `db.ha.enabled=true`, `db.cloud.replicas=node2,node3,node4`, `db.usage.replicas=…`, and `db.cloud.secondsBeforeRetrySource` (default 1 hour) ✅ **Verified**.

### 4.4 The agent on the hosts

✅ **Verified** (CloudStack hosts documentation, retrieved 2 October 2026): the `cloudstack-agent` runs **on the hypervisor host** (the documentation's example is KVM) and connects to the management server on **port 8250**, using certificate-based authentication with a `cloud.jks` keystore and the `ca.plugin.root.auth.strictness` setting. "Starting CloudStack 4.11+ the host setting can accept [a] comma separated list of management server IPs to which new CloudStack hosts/agents will get a shuffled list of the same to which they can cycle reconnections in a round-robin way" (before that, a TCP load balancer on port 8250 was the mechanism). "The cloudstack-agent package will install the qemu script in the /etc/libvirt/hooks directory of Libvirt" (KVM). For KVM the agent uses a `gpudiscovery.sh` script to discover GPU devices, requiring `lspci` and `xmlstarlet`. To remove a KVM host: place it in maintenance mode, stop the cloud-agent service, then remove it via the UI. The 4.11+ certificate-authority framework (`ca.plugin.root.auth.strictness`, `ca.plugin.root.allow.expired.cert`, the root CA key/cert settings) governs agent authentication ✅ **Verified**.

### 4.5 The system VMs — CloudStack's built-in plumbing

CloudStack's documentation is unusually explicit that these are *not* user workloads:

> "CloudStack uses several types of system Instances to perform tasks in the cloud. In general CloudStack manages these system VMs and creates, starts, and stops them as needed based on scale and immediate needs. Unlike user VMs, system VMs are expunged on destroying them." ✅ **Verified** (CloudStack system-VM documentation, retrieved 2 October 2026)

**The system VM template.** A **single** template ships for all system-VM types: "Debian 12 (bookworm), 6.1.0 kernel with the latest security patches", a "minimal set of packages installed thereby reducing the attack surface", 64-bit, with "pvops kernel with Xen PV drivers, KVM virtio drivers, and VMware tools", including HAProxy, iptables, IPsec, Apache and a JRE ✅ **Verified**. Since **4.20.0**, KVM supports both Intel/AMD 64-bit (x86_64) and ARM 64-bit (aarch64); other hypervisors are x86_64 only, and ARM templates are not bundled ✅ **Verified**. The template is bundled in the `cloudstack-management` DEB/RPM for KVM, VMware and XenServer, and registered onto secondary storage automatically; alternatives are the `cloud-install-sys-tmplt` script and the settings `system.vm.templates.download.repository` / `system.vm.preferred.architecture` ✅ **Verified**.

**The documented system-VM types** (each ✅ **Verified**):

| System VM | What it does | Operator-relevant detail |
|---|---|---|
| **Console Proxy VM (CPVM)** | Provides instance console access — VNC through the console proxy; the `createConsoleEndpoint` API exists | `consoleproxy.*` settings; LB port 443→443 and 8080→8080 |
| **Secondary Storage VM (SSVM)** | Template/snapshot processing; "Every CloudStack zone has a single System VM for Template processing tasks"; handles traffic between SSVM and secondary storage | LB port 443→443; SSL-offload settings `secstorage.ssl.cert.domain`, `secstorage.encrypt.copy` |
| **Virtual Router** | "one of the most frequently used service providers in CloudStack" — routing, DHCP, NAT/port forwarding, LB, VPN and firewall for a guest network | The end user "has no direct access"; "There is no mechanism for the administrator to log in to the virtual router"; restarting "interrupts public network access" |

The virtual router's characteristics "are set by its system service offering" (type `Domain Router`); **all virtual routers in a single guest network use the same system service offering**; an administrator can upgrade a router by creating and applying a custom system service offering ✅ **Verified**. **Accessing system VMs** ✅ **Verified**: SSH from the host to the link-local IP on XenServer/KVM (key `/root/.ssh/id_rsa.cloud` on each agent), or from the management server to the private IP on ESXi (key `~cloud/.ssh/id_rsa`), on **port 3922**.

### 4.6 Primary vs secondary storage

✅ **Verified** (CloudStack concepts and "Working with Storage" documentation, retrieved 2 October 2026). **Primary storage** "is associated with a cluster, and it stores virtual disks for all the VMs running on hosts in that cluster. On KVM and VMware, you can provision primary storage on a per-zone basis." At least one is required; multiple per cluster or zone are allowed; it is "typically located close to the hosts for increased performance". **Zone-wide primary storage** avoids extra data copies (cross-cluster data otherwise has to be staged through secondary storage). Notes: "For Hyper-V, SMB/CIFS storage is supported. Note that Zone-wide Primary Storage is not supported in Hyper-V."; "Ceph/RBD storage is only supported by the KVM hypervisor. It can be used as Zone-wide Primary Storage."; and it works with standards-compliant iSCSI and NFS servers — the documentation names SolidFire and Dell EqualLogic for iSCSI, NetApp filers for NFS and iSCSI, and Scale Computing for NFS. If using local disk only, you can skip adding separate primary storage.

**Secondary storage** "is a zone-wide resource which stores disk templates, ISO images, and snapshots", available to all hosts in its scope (per zone or per region). Object storage can be added (Swift or S3 plugins) alongside the zone-based NFS staging store, which forwards to the cloud-wide object store. Caveats: "Heterogeneous Secondary Storage is not supported in Regions."; for Hyper-V, SMB/CIFS is supported. **Secondary-storage data loss is explicitly consequential:** "Secondary storage data loss will impact recently added user data including Templates, Snapshots, and ISO Images. Secondary storage should be backed up periodically. Multiple secondary storage servers can be provisioned within each zone to increase the scalability of the system." ✅ **Verified** (reliability documentation).

### 4.7 The offerings model

✅ **Verified** (CloudStack service-offerings documentation, retrieved 2 October 2026), which groups "Service Offerings, Disk Offerings, Network Offerings, and Templates":

- **Compute (service) offerings** "provide a choice of CPU speed, number of CPUs, RAM size, tags on the root disk, and other choices." They may be **"fixed"**, **"custom constrained"** or **"custom unconstrained"**; in a fixed offering the CPU count, memory and CPU frequency are predefined by the administrator. Disk offerings can be linked into a compute offering to define the root volume.
- **Disk offerings** "provide a choice of disk size and IOPS (Quality of Service) for primary data storage."
- **Network offerings** "describe the feature set that is available to end users from the virtual router or external networking devices on a given guest network."
- **Templates** are "the base OS images that the user can choose from when creating a new Instance", defined by the administrator or any user.
- **System service offerings** are available only to the root administrator and configure virtual-infrastructure resources (e.g. upgrading a virtual router).
- CloudStack "emits usage records that can be integrated with billing systems".
- **Scoping:** "Since version 4.13; compute offerings, disk offerings, network offerings and VPC offerings can be scoped to (made available in) combinations of specific domain(s) and zone(s) or to all domains and zones", via `updateServiceOffering`, `updateDiskOffering`, `updateNetworkOffering` and `updateVpcOffering`.

The offerings model is where a bank's *governance* lives: the catalogue (offerings + templates) is the list of infrastructure shapes the business is permitted to request.

### 4.8 The API, CLI and UI

- "CloudStack provides a REST-like API for the operation, management and use of the cloud." ✅ **Verified** (concepts documentation).
- A browser-based UI and the CloudMonkey CLI (`cmk`) are the documented operator surfaces ✅ **Verified** (cloudstack.apache.org).
- The EC2 API translation layer and AWS EC2/S3 compatibility statement are documented ✅ **Verified** (as documented), but their current maintenance status is ❌ **not established** (§16).

### 4.9 Supported hypervisors — with the date of the source

CloudStack documents multiple-hypervisor support, and — critically — **the list is dated to the documentation set retrieved**. From the 4.23.0.0 concepts documentation (retrieved **2 October 2026**): "A single cloud can contain multiple hypervisor implementations. As of the current release CloudStack supports: BareMetal (via IPMI), Hyper-V, KVM, LXC, vSphere (via vCenter), Xenserver, Xen Project." ✅ **Verified**. The site's own navigation additionally lists targets "VMware, XenServer, KVM, XCP-ng, BareMetal" ✅ **Verified** (cloudstack.apache.org, retrieved 2 October 2026) — note **XCP-ng** appears on the front-page list as a target, a slightly different presentation from the concepts page's "Xen Project" entry; both are reported, and this guide does not reconcile them beyond noting the difference. ⚠ **Flagged — dated list:** any procurement document must pin the hypervisor list to the version it will actually run, not to this guide.

### 4.10 Releases and lifecycle

✅ **Verified** (CloudStack downloads page, retrieved 2 October 2026): the project maintains **two release types**: "the main releases and the LTS (Long Term Support) releases". "The LTS releases receive bug and security fixes for a period of 18 months after the main release… The main releases receive only critical bug fixes for a short period. The general expectation is that the users of the main version will upgrade to a new version in order to receive fixes." **Latest release at retrieval: 4.23.0.0**, "a CloudStack Regular release". **Current LTS: 4.22.1.1**; the latest 4.20.x.x LTS maintenance release: **4.20.3.1**. Community package repositories are listed for Ubuntu (DEB), EL10/EL9/EL8 (RPM), SUSE/openSUSE 15 (RPM), and experimental ARM64. The practical read for a bank: **LTS = 18 months of fixes; Regular = upgrade-to-stay-fixed** — a documented, published lifecycle policy, and a real input to §12 (upgrade appetite).

### 4.11 Kubernetes and containers on CloudStack

✅ **Verified** (cloudstack.apache.org/kubernetes, retrieved 2 October 2026) — two *distinct* paths: **CloudStack Kubernetes Service (CKS)** is "developed as a plug-in to Apache CloudStack" and "gives Cloud Service Providers a Container as a Service (CaaS) offering within their existing IaaS environments", letting "users … create Kubernetes clusters within an existing multi-tenant environment provided by CloudStack", managed "in the same user-interface they use to manage their existing compute, network and storage". Separately, the **Kubernetes Cluster API Provider for Apache CloudStack (CAPC)** is "available under the Apache 2 open-source license and is managed by the Cloud Native Computing Foundation (CNCF)". So CloudStack's container story is a *plug-in service for the operator* (CKS) plus a *CNCF-managed Cluster API provider for Kubernetes users* (CAPC) — both the project's own published material.

### 4.12 The CloudStack summary box

| Attribute | Value | Source (retrieved 2 Oct 2026) |
|---|---|---|
| What it is | Open-source IaaS; "turnkey solution" | cloudstack.apache.org |
| Control plane | Management server (Tomcat) + MySQL | CloudStack concepts doc |
| Host daemon | `cloudstack-agent`, port 8250 | CloudStack hosts doc |
| Plumbing | system VMs (CPVM, SSVM, virtual router) | CloudStack system-VM doc |
| Hierarchy | region ⊃ zone ⊃ pod ⊃ cluster ⊃ host | CloudStack concepts doc |
| Latest release | 4.23.0.0 (Regular); LTS 4.22.1.1 | CloudStack downloads page |
| LTS window | 18 months of fixes | CloudStack downloads page |
| Hypervisors (4.23.0.0 docs) | BareMetal (IPMI), Hyper-V, KVM, LXC, vSphere, Xenserver, Xen Project | CloudStack concepts doc |
| Kubernetes | CKS plug-in; CAPC (CNCF) | cloudstack.apache.org/kubernetes |
| Licence | Apache-2.0 | cloudstack.apache.org |

---

## 5. The OpenStack Side as a Cross-Reference

This section is intentionally short. OpenStack's detail lives in sibling guides, and this guide points at them rather than re-deriving their material.

### 5.1 Where to read the OpenStack material

The §1.5 boundary table already maps every piece of OpenStack material to its owning sibling, so §5 points rather than repeats: origins → [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.1; the component roster and per-service functions → its §3.2; the OpenStack component table → its §3.3; the release timeline → its §3.6; the HCI-vs-disaggregated head-to-head → its §4; the commercial distributions → its §5. DevStack's all-in-one topology and single-host service roster → [devstack_openstack_guide.md](devstack_openstack_guide.md) §4.2; release naming and calendar versioning → its §6. **None of it is restated here.**

### 5.2 The three facts this comparison actually needs

Everything §3, §6 and §9 assert about OpenStack reduces to three verified properties:

1. **Many services, one logical platform.** "OpenStack was created during the first months of 2010. Rackspace wanted to rewrite the infrastructure code running its Cloud servers offering… At the same time, Anso Labs (contracting for NASA) had published beta code for Nova, a Python-based 'cloud computing fabric controller'." ✅ **Verified** (OpenStack project-team guide, "A Bit of OpenStack History", retrieved 2 October 2026). The "big tent" reform — "In December 2014, the Technical Committee introduced a Project structure reform (dubbed the 'big tent') that moved to a community-centric definition of 'OpenStack'" — is the governance expression of the same plural structure ✅ **Verified**.

2. **A foundation/consortium with a commercial ecosystem.** "In September 2012, the OpenStack Foundation was launched as an independent body providing shared resources to protect, empower, and promote OpenStack software and the community around it." The Foundation had a **Board of Directors** (objectives, budget, trademark) and a **Technical Committee** (authority over the upstream project), with a User Committee that "has been merged into the Technical Committee" since June 2020 ✅ **Verified** (OpenStack project-team guide). The Foundation was subsequently renamed the **Open Infrastructure Foundation** and, as of 2026, "OpenInfra Foundation is part of the nonprofit Linux Foundation" ✅ **Verified** (openinfra.org/about, retrieved 2 October 2026; see §6 for the date discrepancies).

3. **Almost always consumed through a distribution.** Red Hat, Canonical and Mirantis distribute OpenStack commercially ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5). That is the ecosystem fact §9 builds on.

### 5.3 The one OpenStack nuance this guide flags

The sibling guide [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) states OpenStack was "launched October 2010" and dates the first release "Austin" to 21 October 2010. The project's own governance document dates the *announcement* differently: the first Design Summit was "held in Austin, TX on July 13-14, 2010, and the project was officially announced at OSCON in Portland, OR, on July 21st, 2010", while the wiki page appeared "on May 24th, 2010" ✅ **Verified** (OpenStack project-team guide).

> ⚠️ **Flagged — reconcile honestly.** The project's own docs date the *announcement* to **July 2010**; the sibling guide dates the *launch/first release* to **October 2010**. These are not necessarily contradictory — a project can be announced in July and its first release ships in October — but this guide does not silently pick one. It states both and points the reader at the sibling guide for the release event.

### 5.4 What §5 deliberately does not do

It does **not** re-list Nova/Neutron/Cinder/etc. (the sibling guide's §3.2 is authoritative, and duplicating it would create two sources of truth in the repo), does **not** re-argue the HCI-vs-disaggregated comparison (sibling §4), and does **not** present OpenStack's release timeline (sibling §3.6).

---

## 6. The History and Governance Comparison

This is the intellectual core of the guide. The argument is not that one project has better engineering; it is that **the two governance models produce different kinds of product, and a bank's technology committee is really choosing between those models.**

### 6.1 Apache CloudStack — the route to Apache, with verified years

Every date below is quoted from Apache CloudStack's own history page ✅ **Verified** (cloudstack.apache.org/history, retrieved 2 October 2026):

- **2008** — "The CloudStack project began as a project of a start-up known as VMOps in 2008."
- **May 2010** — "The company eventually changed its name to Cloud.com, and it released much of the source to CloudStack in May 2010 under the GNU General Public License version 3 (GPLv3)."
- **July 2011** — "Cloud.com was purchased in July 2011 by Citrix, and the remainder of CloudStack's code was released (again, under the GPLv3) in August 2011."
- **Early 2012** — "Citrix released CloudStack 3.0 in early 2012."
- **April 2012** — "In April 2012, Citrix re-licensed CloudStack under the Apache Software License 2.0 (ASLv2) and submitted CloudStack to the Apache Incubator. It was accepted into the Incubator on April 16th, 2012."
- **6 November 2012** — "CloudStack made its first major release (4.0.0-incubating) from the Apache Incubator on November 6th, 2012."
- **12 February 2013** — "The first minor release (4.0.1-incubating) came out on February 12, 2013."
- **20 March 2013** — "Apache CloudStack graduated from the Incubator on March 20, 2013, and the announcement was released on March 25, 2013." Graduating from the incubator makes it an **Apache top-level project**.

The trajectory is a **commercial-origin project that was donated into a neutral foundation and re-licensed to Apache-2.0**. That is a common Apache pipeline, and it is the reason CloudStack's identity is now inseparable from the identity of the Apache Software Foundation.

### 6.2 CloudStack's governance today

✅ **Verified** (cloudstack.apache.org/who, retrieved 2 October 2026):

- The project is governed by an **Apache Project Management Committee (PMC)**, with a **PMC Chair** — the page names **Wido den Hollander** as PMC Chair — and a published list of PMC members and committers.
- Project membership and PMC information are published via the Apache projects infrastructure (`projects.apache.org`, and board minutes at Whimsy), and participation follows **"The Apache Way"**.
- The source lives at `github.com/apache/cloudstack`, under the Apache Software Foundation.
- The current PMC chair's published focus, quoted from the page: "I'm excited to continue as VP this year, with a clear focus: supporting a[an] active, open community and ensuring the project is healthy, especially as reliable, private clouds are ever more important." ✅ **Verified**.

The key governance property — the one that matters more than any feature — is this: **Apache CloudStack is governed by a meritocratic foundation whose authority is the PMC and whose releases are made by the project community, not by a product manager at a company.** The release cadence is the community's (§4.10: Regular releases plus 18-month LTS releases), it is published, and it is not gated by a single vendor's commercial calendar.

### 6.3 OpenStack — the route to the OpenInfra Foundation, with verified years

✅ **Verified** (OpenStack project-team guide, "A Bit of OpenStack History", retrieved 2 October 2026):

- **Early 2010** — "OpenStack was created during the first months of 2010. Rackspace wanted to rewrite the infrastructure code running its Cloud servers offering… At the same time, Anso Labs (contracting for NASA) had published beta code for Nova."
- **13–14 July 2010 / 21 July 2010** — "The first Design Summit was held in Austin, TX on July 13-14, 2010, and the project was officially announced at OSCON in Portland, OR, on July 21st, 2010." (A wiki page appeared "on May 24th, 2010".)
- **September 2012** — "In September 2012, the OpenStack Foundation was launched as an independent body providing shared resources to protect, empower, and promote OpenStack software and the community around it."
- **Governance bodies** — the Foundation had a **Board of Directors** ("objectives, budget, trademark") and a **Technical Committee** ("technical matters, authority over upstream projects"), and originally a **User Committee**.
- **June 2013** — the Technical Committee "decided to switch to 13 directly-elected members instead" of being "formed by all the PTLs + five members" directly elected; "Half of those are [renewed every 6 months]".
- **December 2014** — "the Technical Committee introduced a Project structure reform (dubbed the 'big tent') that moved to a community-centric definition of 'OpenStack'."
- **June 2020** — "Since June, 2020 User Committee has been merged into the Technical Committee and is not a separate body anymore."

### 6.4 OpenStack — the rename and the Linux Foundation move

Both of these are published facts with **honestly flagged date discrepancies** against the sibling guide:

- **Rename to OpenInfra Foundation.** The Foundation's own community article, "Virtual Open Infrastructure Summit Recap", by Allison Price, is dated **28 October 2020** and describes the event as "Hosted by the Open Infrastructure Foundation (previously the OpenStack Foundation (OSF))", reporting that the announcement was made at the first virtual Open Infrastructure Summit in 2020 ✅ **Verified** (superuser.openinfra.org article, dated 28 October 2020). The openinfra.org blog similarly states: "At the first virtual Open Infrastructure Summit in 2020, Open Infrastructure Foundation officially announced its ongoing evolution" ✅ **Verified** (openinfra.org blog, post dated 04/06/2021).
  > ⚠️ **Flagged — discrepancy.** The sibling guide [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) states the rename was "2021". The primary sources above date the *announcement* to **October 2020** (with subsequent published material in 2021). This guide reports **October 2020** as the announcement date, notes the sibling's 2021, and does not silently override it.
- **Join to the Linux Foundation.** The Linux Foundation press release "Open Infrastructure Foundation Board Announces Intent to Join the Linux Foundation", dated **12 March 2025**, states: "the Open Infrastructure Foundation (OpenInfra) has signaled its intent to join as a member foundation, following unanimous approval from both the OpenInfra and Linux Foundation boards." ✅ **Verified** (Linux Foundation press release, 12 March 2025). As of 2026, openinfra.org states "OpenInfra Foundation is part of the nonprofit Linux Foundation" ✅ **Verified** (openinfra.org/about, retrieved 2 October 2026).
  > ⚠️ **Flagged — discrepancy.** The sibling guide [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) says OpenStack "joined Linux Foundation 2024". The primary source is an **intent announcement dated 12 March 2025**, with membership described as pending at that time. This guide reports the **12 March 2025 intent** and the **2026 "part of the Linux Foundation" status**, and notes the sibling's 2024 rather than choosing silently.

### 6.5 The governance contrast — stated plainly

| Property | Apache CloudStack | OpenStack |
|---|---|---|
| Origin | VMOps (2008) → Cloud.com → Citrix → Apache (2012) ✅ | Rackspace + NASA (2010) → OpenStack Foundation (2012) ✅ |
| Governing body | Apache PMC + PMC Chair ✅ | Foundation Board + Technical Committee ✅ |
| Parent organisation | The Apache Software Foundation ✅ | OpenInfra Foundation, part of the Linux Foundation (2026) ✅ |
| Membership model | Apache meritocracy ("The Apache Way") — committers and PMC members by contribution ✅ | Foundation with corporate **membership tiers** (Gold/Platinum members etc.) ✅ (openinfra.org) |
| Release cadence | Community-driven: Regular + 18-month LTS ✅ | Community/project-driven, coordinated across many projects ✅ (cross-ref [devstack_openstack_guide.md](devstack_openstack_guide.md) §6) |
| Roadmap control | The PMC, on community consensus ✅ | The Technical Committee across projects, within a corporate-member ecosystem ✅ |
| Commercial layer | Optional support, integrators around an Apache project ✅ | Commercial **distributions** are the normal consumption path ✅ |

### 6.6 The consequence — what a technology committee is actually choosing

Four consequences flow directly from the contrast above, and they are the reasons this section is the guide's core:

1. **Release cadence.** CloudStack publishes a cadence the community controls — Regular releases plus LTS releases with an 18-month fix window ✅ **Verified** (CloudStack downloads page). OpenStack's cadence is coordinated across many projects and releases ✅ (cross-ref [devstack_openstack_guide.md](devstack_openstack_guide.md) §6). *Consequence:* CloudStack lets a bank pin an LTS and stop; OpenStack's model expects more continuous alignment across services.

2. **Roadmap control.** In CloudStack, the roadmap is the PMC's and the community's. In OpenStack, the Technical Committee has "authority over upstream projects" but operates inside a foundation with corporate members who have "a vested interest" and, in the foundation's own words, "an inside track to proof of concept (POC) and production use" ✅ **Verified** (openinfra.org/about, retrieved 2 October 2026). *Consequence:* CloudStack's direction is community-pulled; OpenStack's direction is community-pushed *with* a commercial ecosystem pulling alongside.

3. **Commercial ecosystem.** OpenStack's entire consumption model assumes distributions ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5). CloudStack is more often the Apache release plus optional support and integrators ✅ (§9). *Consequence:* with OpenStack you are more likely to buy an integrated product from a vendor than to run the upstream; with CloudStack the reverse is more common.

4. **Purchaser's long-term risk.** Here is the honest asymmetry, stated as a *shape* and not a verdict:
   - A bank choosing **CloudStack** is betting on an **Apache project's continuity** — the Apache Software Foundation's track record of sustaining donated projects, and the PMC's ability to keep releasing. The risk is *community and contributor continuity*, not vendor continuity.
   - A bank choosing **OpenStack** is betting on an **ecosystem's continuity** — that at least one distribution vendor (Red Hat, Canonical, Mirantis, or another) keeps shipping a supported build. The risk is *vendor and ecosystem continuity*, which is a different risk with a different mitigation (a support contract).

**The intellectual core, stated as the guide's central claim:** *the two projects are not distinguished first by a feature list; they are distinguished by who decides what gets built and who is accountable when it breaks.* A bank's technology committee that works through a feature matrix and skips this question has optimised the wrong variable, and §14 records that as the first anti-pattern.

---

## 7. The Operational Reality

§3 described the shapes; this section describes what they cost a team. It is deliberately about *work*, not features.

### 7.1 What each model demands of a team

**CloudStack demands a small team that can own one orchestrator and a database.** The project positions it that way: it is "still easy to use and implement with a small team" ✅ **Verified** (cloudstack.apache.org). Concretely, the operator must run and protect:

- the management server (multi-node, load-balanced) ✅,
- its MySQL database (replicated for HA) ✅,
- the hypervisor hosts with the cloudstack-agent ✅,
- and *supervise* the system VMs, which CloudStack creates and manages automatically ✅.

That is a bounded, learnable surface. One competent engineer can run a modest CloudStack cloud; the same engineer is also the person who owns the database, which is the real skill. The system VMs remove the need to build the cloud's own plumbing.

**OpenStack demands a team that can own many services and the integration between them.** The operator must run Nova, Neutron, Cinder, Glance, Keystone, a message bus, databases, and the network agents ✅ (cross-ref [devstack_openstack_guide.md](devstack_openstack_guide.md) §4.2, [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2). The surface is larger and, crucially, *grows with what you turn on*: adopt object storage and Swift joins; adopt orchestration and Heat joins. The sibling guide's §4.3 covers the operations comparison on the HCI axis.

> The staffing claim is not "CloudStack needs fewer people". It is: **CloudStack needs a team that owns one system; OpenStack needs a team that owns a system *of systems*.** Those are different hiring briefs, and §12 makes "team shape" a named decision input for exactly that reason.

### 7.2 The skills each requires

| Skill | CloudStack's emphasis | OpenStack's emphasis |
|---|---|---|
| Virtualisation | Hypervisor management (KVM/XenServer/ESXi/Hyper-V) + agent troubleshooting ✅ | Same, plus per-service hypervisor drivers ✅ |
| Databases | One MySQL database, replication, backup ✅ | Several service databases ✅ |
| Linux systems | Management-server host, system VM access (port 3922) ✅ | Many service hosts ✅ |
| Networking | Physical/virtual network setup, zones/pods/subnets, VLAN isolation, virtual-router offerings ✅ | Neutron, agents, ML2-style driver stacks, overlay networking ✅ |
| Python / config | Plugin and template work; CloudMonkey scripting ✅ | Deep Python service debugging; each service is a Python project ✅ |
| Integration | API-driven automation (Terraform, etc.) ✅ | Integration across services + distributions ✅ |

The honest read: **CloudStack's hardest skill is the database, plus the network model; OpenStack's hardest skill is distributed-service debugging.** A bank that already has strong DBA and Linux skills is closer to CloudStack-ready; a bank with strong Python platform-engineering and Kubernetes-adjacent skills is closer to OpenStack-ready.

### 7.3 What an upgrade looks like in each

- **CloudStack** publishes an explicit two-track lifecycle: **LTS releases receive bug and security fixes for 18 months** after the main release, and **main/Regular releases "receive only critical bug fixes for a short period"**, with "the general expectation … that the users of the main version will upgrade to a new version in order to receive fixes" ✅ **Verified** (CloudStack downloads page, retrieved 2 October 2026). Upgrades are documented in the "upgrade section" of the documentation for each release (the downloads page links to the upgrade instructions for 4.23.0.0, 4.22.1.1 and 4.20.3.1) ✅ **Verified**. Because the management server is stateless and the workload is decoupled from it, the *shape* of a CloudStack upgrade is: upgrade the management server(s), keep the running estate, accept a management-plane pause ✅ (CloudStack reliability documentation).
- **OpenStack** releases on a coordinated, calendar-driven cadence across projects ✅ (cross-ref [devstack_openstack_guide.md](devstack_openstack_guide.md) §6). In practice, operators upgrade *through a distribution* (Red Hat, Canonical, Mirantis — §9), and the distribution sets the tests and the path. The shape is: upgrade the distribution's services in a supported sequence, keeping services mutually version-compatible.

> **The honest difference:** CloudStack's upgrade is **one system's upgrade** with a published LTS window; OpenStack's upgrade is **a distribution's upgrade** with a vendor's compatibility matrix. A bank that wants "upgrade one thing, on my cadence" is pulled toward the first. A bank that wants "a vendor tells me the tested path" is pulled toward the second.

### 7.4 How a partial failure is diagnosed in each

- **In CloudStack**, the decomposition is documented and clean. If the whole management server set is down: "no new Instances can be created", but "All Guest Instances will continue to work" and host operation is unaffected ✅ **Verified** (CloudStack reliability documentation). The diagnosis path is therefore narrow: *is the management server up, is the database reachable, is the agent connected on port 8250?* The system VMs add a second, bounded surface: a misconfigured `host` value means "the UI is still available" but "the system VMs cannot contact the management server", which produces a specific, recognisable symptom ✅ **Verified**.
- **In OpenStack**, the decomposition is per service. A symptom in the API may have its cause in the message bus, a database, a network agent, or a downstream service's own database. Diagnosis is *tracing a request across services*. That is a distinct skill (§7.2) and a distinct runbook.

Neither is better. **The failure modes are as different as the architectures**, and §14 records "assuming a smaller footprint means less resilience" as an anti-pattern because it is the mistake people make when they *don't* think this through.

### 7.5 The honest asymmetry — easier to start, easier to extend

This is the sentence the section exists to deliver:

> **The smaller-component-count model is easier to *start*; the larger one is easier to *extend* — and the architecture is what determines which is which.**

- CloudStack's single management server, thin agents, and auto-managed system VMs mean fewer things to stand up, fewer integration points to get right, and a shorter path from "bare metal" to "self-service VM cloud" ✅ (§4). That is "easier to start".
- OpenStack's set of cooperating services means a new capability is usually *a new service or a new driver*, added without waiting for one orchestrator to grow it ✅ (§3.6). That is "easier to extend".

The two are not in tension because they are the *same* property seen from two ends: **integration you don't have to do is flexibility you don't get to choose later.** CloudStack does the integration for you (turnkey) and so moves faster to day one but along a more defined path. OpenStack leaves you the integration and so moves slower to day one but along a path you compose.

### 7.6 The operational one-liner

> CloudStack's operations job is **run one system really well**; OpenStack's operations job is **integrate many systems really well**. Both are full-time jobs — they are just different jobs, for different people, hired against different briefs.

---

## 8. The Capability Comparison

A capability table on this pair is a **trap** for the reason §1 gave: both are Apache-2.0 IaaS platforms with deliberately overlapping feature sets, so the table can be made to say anything by choosing rows. This guide therefore uses the table only where each row can be **anchored to a dated source on the CloudStack side**, and it prints **"not established"** wherever the two cannot be compared on public documentation *from this guide's sources*. Every CloudStack column fact is from Apache documentation retrieved **2 October 2026** (4.23.0.0 docs). No adoption or cost figure appears here.

| Capability | Apache CloudStack | OpenStack | Source & date |
|---|---|---|---|
| **Compute (VM lifecycle)** | Orchestrates pools of compute resources; assigns guest VMs to hosts; live-migration within a cluster ✅ | Nova is the compute service — schedule, provision, migrate, resize ✅ | CloudStack concepts doc, 2 Oct 2026; sibling §3.2 |
| **Block storage** | Primary storage associated with cluster or zone; standards-compliant iSCSI and NFS; Ceph/RBD on KVM ✅ | Cinder is the block-storage service ✅ | CloudStack "Working with Storage" doc, 2 Oct 2026; sibling §3.2 |
| **Object storage** | Object storage addable via **Swift or S3 plugins** alongside the NFS secondary staging store ✅ | Swift is the object-storage service ✅ | CloudStack storage doc, 2 Oct 2026; sibling §3.2 |
| **Software-defined networking** | Basic and Advanced zone networking; VLAN isolation; network offerings; VPC; network service providers; system reserved IP range ✅ | Neutron is the networking service — virtual networks, routers, LB, security groups, floating IPs ✅ | CloudStack networking doc, 2 Oct 2026; sibling §3.2 |
| **Image / template model** | **Templates** — "the base OS images that the user can choose from when creating a new Instance"; secondary storage holds templates, ISOs and snapshots ✅ | Glance is the image service — registry of VM disk images and snapshots ✅ | CloudStack service-offerings + concepts docs, 2 Oct 2026; sibling §3.2 |
| **Orchestration / templating** | API-driven automation; Terraform integration is the project's own published pattern ("CloudStack and Terraform bring scalability and flexibility") ✅; a native Heat-style template engine is **not established from this guide's sources** | Heat is the orchestration service, template-driven stacks (HOT format) ✅ | CloudStack integrations page + solutions brief, 2 Oct 2026; sibling §3.2 |
| **Kubernetes provisioning** | Two published paths: **CKS** (a CaaS plug-in to CloudStack) and **CAPC** (CNCF-managed Cluster API provider) ✅ | Not established from this guide's sources (see §16) | cloudstack.apache.org/kubernetes, 2 Oct 2026 |
| **Bare metal** | **BareMetal (via IPMI)** appears in the 4.23.0.0 documented hypervisor/target list ✅; site nav lists "BareMetal" ✅ | Not established from this guide's sources (see §16) | CloudStack concepts doc + site nav, 2 Oct 2026 |
| **GPU support** | KVM agent discovers GPU devices via `gpudiscovery.sh` (requires `lspci`, `xmlstarlet`) ✅ | Not established from this guide's sources (see §16) | CloudStack hosts doc, 2 Oct 2026 |
| **Hypervisor breadth** | BareMetal (IPMI), Hyper-V, KVM, LXC, vSphere (via vCenter), Xenserver, Xen Project ✅; "A single cloud can contain multiple hypervisor implementations" ✅ | Not restated here — see sibling §3.2 | CloudStack concepts doc, 2 Oct 2026 |
| **API** | "REST-like API"; EC2 API translation layer (documented); AWS EC2/S3 compatibility claim ✅ (as documented; current maintenance ❌ not established) | Each service exposes its own API ✅ | CloudStack concepts doc, 2 Oct 2026; sibling §3.2 |

### 8.1 How to read this table honestly

Three cautions, each of which is a rule of this guide:

1. **"Not established" is not "not capable".** Where the OpenStack column says "not established from this guide's sources", it means *this guide did not verify it at a primary source*, **not** that OpenStack lacks the capability. Asserting absence would violate this guide's first hard rule.
2. **Overlap is the norm, not the signal.** Compute, block storage, object storage, SDN, images — both sides do all of these. The discriminating rows are **orchestration/templating, Kubernetes provisioning, bare metal, and GPU support**, and even there the honest answer for OpenStack is "read the sibling guides and the project's own docs", not a verdict from a table cell.
3. **The table cannot carry the decision.** This is exactly why §12 is a framework and not a score: a capability grid where nine of eleven rows are near-identical and two are "not established" has almost no discriminating power, and choosing on it is the first anti-pattern (§14.1).

### 8.2 What the table actually shows

The table shows a **turnkey platform that documents its own plumbing** (CloudStack lists its system VMs, storage plugins, hypervisor targets, GPU-discovery script and Kubernetes plug-ins *as parts of the product*) versus a **project whose capabilities are separate projects** (OpenStack's compute, storage, network and image capabilities are *named services*, each with its own documentation home). That is §3 again through a different door: the feature rows are downstream of the architecture, not the reverse.

---

## 9. The Ecosystem

The honest observation this section opens with:

> **The ecosystem, not the code, is often what a bank is buying.** Both projects are Apache-2.0 software you can download for free. What a bank actually procures is *skill, support, integration and continuity* — and those live in the ecosystem around the code, not in the code.

This section names ecosystem participants **factually and without ranking**. No vendor is recommended, scored, or credited with a capability beyond its own documentation. Nothing here is an endorsement.

### 9.1 OpenStack's ecosystem — the distribution layer

OpenStack is almost always consumed through a **commercial distribution**. The distributions and their documented positioning are covered in full in [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5, and are named here factually so this guide's contrast is complete:

- **Red Hat** — the classic **Red Hat OpenStack Platform** line and its successor **Red Hat OpenStack Services on OpenShift** (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5.2). Note explicitly, because of §1's HAZARD 1: the successor *runs on OpenShift*, which is Red Hat's **Kubernetes** platform — which is precisely why the OpenStack-versus-OpenShift conflation is a live hazard in this repo and not a hypothetical one.
- **Canonical** — **Charmed OpenStack**, Ubuntu-based, operated with Juju charms (cross-ref §5.3).
- **Mirantis** — **Mirantis OpenStack for Kubernetes** (cross-ref §5.4).

These are **named as ecosystem participants only.** This guide does not say which is best, cheapest, or most suitable; the sibling guide's §5 carries the detail and its own status markers.

**What the distribution layer means for a bank.** With OpenStack, the *normal* procurement is: pick a distribution, buy its support, and let the vendor own the tested compatibility path across services. That is a genuine, verifiable ecosystem property: the distributions exist precisely because integrating the services is hard enough that a market formed around doing it. A bank that chooses OpenStack but refuses to buy a distribution (or fund an equivalent internal integration team) has chosen the harder half of OpenStack on purpose.

### 9.2 CloudStack's ecosystem — integrations, integrators and support

Apache's own pages are the only acceptable source for CloudStack ecosystem names in this guide, and they support three verified statements:

1. **There is an integrators and integrations ecosystem.** Apache's integrations page states, in the project's own words, that CloudStack "offers extensive integration options to seamlessly align with your technology stack", and that the project's own community publishes "solution briefs" and "new strategic partnerships" ✅ **Verified** (cloudstack.apache.org integrations page, retrieved 2 October 2026). The user page separately names "systems integrators that offer CloudStack related services" as a category of CloudStack user ✅ **Verified** (cloudstack.apache.org/users, retrieved 2 October 2026).
2. **Named technology-integration partners published by the project** — the integrations page names, factually: **Apiculus** (a cloud management / billing portal — the project quotes "our aim is to position the combination of CloudStack and Apiculus as a robust cloud solution in 100+ countries"), **Tungsten Fabric** ("an open-source software defined network and security orchestrator"), **StorPool** (a "high-performance primary storage platform"), and **Terraform** (automation: "CloudStack and Terraform bring scalability and flexibility") ✅ **Verified** (cloudstack.apache.org integrations page, retrieved 2 October 2026). These are named **as integrations the project publishes**, not as rankings or recommendations.
3. **There are commercial support vendors.** The integrations page and the user page both reference commercial activity around CloudStack. However:

> ❌ **Not verified here — the specific commercial support-vendor list.** This guide could not verify a specific vendor of *commercial CloudStack support* at a primary or vendor source within its research window. It therefore names **no** CloudStack support vendor. In particular, any specific vendor name you may have seen associated with CloudStack support is **not asserted here**, because this guide could not confirm it at an acceptable source. See §16.

### 9.3 The contrast — ecosystem as a purchase

| Ecosystem property | CloudStack | OpenStack |
|---|---|---|
| Normal consumption path | Apache release + optional support/integrators ✅ | A commercial **distribution** ✅ |
| Named technology integrations | Published by the project (Apiculus, Tungsten Fabric, StorPool, Terraform) ✅ | Published by each distribution ✅ |
| Commercial support vendors | Exist per Apache's own pages ✅; **specific list not verified here** ❌ | Distributions (Red Hat, Canonical, Mirantis) ✅ |
| What a bank is buying | Integration + continuity around an Apache project ✅ | A vendor's tested integration of many services ✅ |
| Where the integration risk sits | Mostly with the bank and its integrators ✅ | Mostly with the distribution vendor ✅ |

### 9.4 The honest reading

The two ecosystems are shaped by their governance (§6) and their architectures (§3):

- **OpenStack's ecosystem is a distribution market** because OpenStack is an integration project — the integration is the product, so a market formed to sell it. The ecosystem is therefore *deep but commercial*: the bank's continuity depends on a distribution vendor staying in the market.
- **CloudStack's ecosystem is an integrations-and-integrators market** because CloudStack is turnkey — the product is already assembled, so the market around it supplies *adjacent capabilities* (billing, SDN, storage, automation) and *services*. The ecosystem is therefore *broader but shallower on the platform itself*: the bank's continuity depends on the Apache project staying healthy (§6), plus its own or a partner's skill.

> **Which is why "the ecosystem, not the code" is the honest headline.** The code is downloadable and free in both cases. What differs is *who you call when the control plane misbehaves at 03:00* — and the answer points back at §6's governance contrast and §7's staffing contrast, not at the feature table.

---

## 10. Adoption and Workload Fit

### 10.1 The honest position on public adoption numbers

**No adoption, market-share or cost figure in this guide is printed as fact.** Public adoption numbers in this space are unreliable, for four structural reasons:

1. Both platforms are free, self-installable, and frequently deployed *inside* organisations that never announce it.
2. The only comprehensive numbers are **self-reported by the projects' own foundations or user surveys**, which cannot be independently audited.
3. Vendors and integrators have commercial incentives to report the number that supports their pitch.
4. "Adoption" is undefined — downloads, running instances, production clouds and lab experiments are not the same metric.

Therefore this section reports only **each project's own published user/case material, dated**, and says plainly what it is: self-published, not audited.

### 10.2 Apache CloudStack's own published user material

✅ **Verified** (cloudstack.apache.org/users and cloudstack.apache.org/cloud-builders, retrieved 2 October 2026). The project's own framing:

> "The following organisations are known users of Apache CloudStack (or a commercial distribution of CloudStack). Our users include many major service providers running CloudStack to offer public cloud services, product vendors who incorporate or integrate with CloudStack in their own products, organisations who have used CloudStack to build their own private clouds, and systems integrators that offer CloudStack related services."

The page is explicit that it is **self-declared** — entries are added by "pull request to the GitHub" repository — and it states the list is **"subject to change"** ✅ **Verified**. That self-declaration is exactly why this guide treats it as *the project's own published material*, not as a market-share figure.

The published list (project-published; **not ranked, not endorsed, and reproduced here only as the project's own claim**) includes service providers, universities, and integrators, among them — the project's own naming: **Bell Canada, British Telecom (BT), China Telecom, Colt, Disney, Autodesk, Apple, Amdocs, Alcatel-Lucent, Cloudera, Cloudian, Dell, Fujitsu FIP Corporation, Exoscale, Datapipe, EVRY, Imperial College, INRIA, KTH, Melbourne University, University of Cologne, Telia Latvia, NxtGen Cloud Technologies, Severalnines, Datacenter Services, Axians, Codero, Exaserve, Miriadis, SafeSwiss Cloud, Serverion, Tucha** ✅ **Verified** (project's own user page, retrieved 2 October 2026). The presence of a name on the project's page is a claim by the project; this guide asserts only that *the project publishes it*.

Apache's own **case-study material** (cloud-builders page, retrieved 2 October 2026) names, as worked examples:

- **IKOULA** (France) — "IKOULA Simplifies the Management of Large-Scale Cloud Infrastructure with CloudStack and XCP-ng" ✅ **Verified**.
- **Your.Online** — "Future-Proof Open-Source Platform Hosting Millions of Websites for Your.Online Powered by CloudStack, KVM and Ceph" ✅ **Verified**.
- **AT&T** — "During the annual CloudStack Collaboration Conference 2023, Alex Dometrius, Associate Director - Technology at AT&T, presented their journey with Apache CloudStack." ✅ **Verified** (this is the project's description of a conference presentation; it is reported as such, not as an audited deployment claim).
- A **Tungsten Fabric** solution brief ✅ **Verified**.

The workload patterns this material associates with CloudStack, all from the above: **service-provider public clouds**, **private clouds**, **hosting at scale (millions of websites)**, and **cloud infrastructure managed with KVM/XCP-ng/Ceph** ✅ **Verified** (as the project publishes).

### 10.3 OpenStack's own published adoption claims

✅ **Verified** (openinfra.org home, retrieved 2 October 2026). The foundation's own published claims, quoted, are:

- **"OpenStack Powers 300+ Public Cloud Data Centers"**
- **"OpenInfra Contributors Have Merged 580k+ Changes Over the Past Decade"**
- **"OpenStack is Deployed On 55M+ Cores Around The [world]"**

These are reported here **as the foundation's own published claims, dated by retrieval (2 October 2026), and they are not audited facts.** They are exactly the kind of self-reported figure §10.1 warns about, and this guide presents them as such — no more and no less. The sibling guide's own treatment of the "OpenStack is dying" debate and its survey numbers is at [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §6.2, and its §3.1 carries the origins narrative.

The workload patterns the foundation associates with its own material: **public cloud data centers**, **telecommunications infrastructure**, and **research/academic clouds** ✅ **Verified** (openinfra.org, retrieved 2 October 2026) — with the caveat that this is the foundation's framing, not an independent classification.

### 10.4 Workload fit — what the published material supports, stated carefully

Combining §10.2 and §10.3, and drawing **only** on what the two projects publish about their own users, the following *patterns* are supported. They are **associations in the projects' own material**, not recommendations:

| Workload pattern | Evidence in the projects' own material | Confidence |
|---|---|---|
| Service-provider / public cloud | Both projects' own user material (CloudStack's user page; OpenInfra's "300+ public cloud DCs") ✅ | Pattern supported; numbers self-reported ⚠️ |
| Private / on-prem cloud | CloudStack user page explicitly ("used CloudStack to build their own private clouds") ✅; OpenStack private-cloud use is standard practice but **not quantified here** ⚠️ | Supported on CloudStack's own page |
| Telco / network-heavy | OpenInfra's own material and the sibling guide ✅ | Cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3, §6.2 |
| Hosting at scale | CloudStack's Your.Online case (millions of websites) ✅ | Project-published case study ⚠️ |
| Academic / research | CloudStack user page names universities ✅ | Project-published list ⚠️ |

### 10.5 What this section refuses to conclude

- It does **not** say one project has more users. The data does not permit it, and §10.1 explains why.
- It does **not** convert any number into a market share, growth rate, or cost. None is verified.
- It does **not** treat a case study as a survey. A named deployment is evidence the platform *can* do a thing; it is not evidence about how common the thing is.

> **The honest summary:** the projects' own published material supports the workload *patterns* above with reasonable confidence, and supports exactly nothing about *sizes, shares or trends* with confidence. A bank that needs to know "is this platform growing?" must answer it from its own vendor-relationship diligence (§6, §9), not from a published number.

---

## 11. The Bank and Enterprise Angle

This section does not re-derive the private-cloud case or the virtualisation landscape — the repo already carries that material, and this guide cross-references it by name. It adds only the four questions that are **specific to choosing between CloudStack and OpenStack inside a regulated institution**.

### 11.1 The private-cloud case, cross-referenced

Why a bank runs its own cloud at all — data residency, latency, control, cost shape — is covered in [cloud_providers_guide.md](cloud_providers_guide.md). The private-cloud *build-versus-buy* framing, the HCI-versus-OpenStack economics, and a worked private-cloud selection are covered in [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §7–§9. The storage layer both platforms depend on is in [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) and [s3_architecture_guide.md](s3_architecture_guide.md); the on-prem AI deployment case (a heavy consumer of an IaaS layer) is in [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md). **This guide does not restate any of that.**

### 11.2 Data sovereignty and jurisdictional control

For a regulated institution, the sovereignty question is not "where is the vendor" but **"where are the control plane, the data, and the keys, and who can reach them"**. On that question the two platforms behave differently *because of §3*, not because of features:

- **CloudStack** places control-plane state (the management-server MySQL database) at a **location the bank chooses** — it is the bank's own server, in the bank's own zone ✅ (CloudStack concepts documentation). The hierarchy itself is a sovereignty instrument: `region ⊃ zone ⊃ pod ⊃ cluster ⊃ host`, with zones that "can be public or private" and private zones "reserved for a specific domain" ✅ **Verified** (CloudStack concepts documentation). A bank can map "data may not leave jurisdiction X" onto a zone boundary in the platform's own vocabulary.
- **OpenStack** distributes control-plane state across per-service databases on the bank's own hosts, similarly under the bank's control ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2). Its sovereignty instrument is the placement/region configuration of each service — more places to get the boundary right, and more places to audit.

> **No jurisdiction's rule is asserted in this guide.** Which residency, outsourcing or data-localisation rules apply to a given bank in a given market is a legal question with a legal source. For the Singapore regulatory context this repo carries [banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md); the reader must take any specific obligation from that guide or the regulator's own text, not from here. This guide states only the architectural property: **both platforms put the control plane under the bank's control; CloudStack concentrates it in one place, OpenStack distributes it across services, and that difference changes how many boundaries a bank must configure and audit.**

### 11.3 The licence-driven reassessment — as reported context, dated

Across the infrastructure industry, organisations have been reassessing incumbent virtualisation stacks — a reassessment that broad-based reporting attributes in part to commercial licensing changes. This guide treats that as **reported context, not as a fact about any vendor's intent**, and it names **no** vendor's licensing position, because it has no dated primary source for one.

The one dated, primary quotation available within this guide's research window is from the Linux Foundation press release of **12 March 2025**, in which the OpenInfra Foundation's executive director, Jonathan Bryce, is quoted:

> "The data center infrastructure market is undergoing a fundamental reinvention, driven by the colossal demands of AI as well as virtualization migration and digital sovereignty." ✅ **Verified** (Linux Foundation press release, 12 March 2025)

That is evidence that a *participant in the market publicly frames the current reassessment as driven partly by "virtualization migration" and "digital sovereignty"*. It is **not** evidence about any individual vendor's licensing decisions, and this guide draws no such inference.

> ❌ **Not established here.** No licence, support-price or vendor-litigation claim is made anywhere in this guide, because none was verified at a dated primary source within the research window. Any such claim a bank is weighing must come from the vendor's own published terms, dated.

### 11.4 The skills-availability question

From §7, restated as a procurement input: the two platforms demand different hires, and a bank must be honest about which it can **retain**.

- **CloudStack** needs engineers who can own a Tomcat-based orchestrator, a MySQL database, hypervisor management and a CloudStack-specific network model ✅ (§7.2). That is a *narrower, more traditional* infrastructure skill set: Linux, database, virtualisation, networking.
- **OpenStack** needs engineers who can debug many Python services, their message bus, their per-service databases and their integration ✅ (§7.2). That is a *broader, more software-engineering* skill set.

The honest procurement question is not "which skill set is better" but **"which skill set can this bank hire, pay and retain in its market — and which can it outsource if it cannot"**. §9 answers the outsourcing half: OpenStack has a distribution market to buy the integration from; CloudStack's support market was **not verified** in this guide (§9.2, §16), which makes the *internal* skills question sharper for CloudStack.

### 11.5 The support and longevity question

Two distinct risks, from §6:

- **CloudStack's longevity risk is project continuity** — the Apache Software Foundation sustaining the project and its PMC continuing to release (Regular + 18-month LTS) ✅ (CloudStack downloads page, history page, who page). The mitigations are internal capability and the Apache foundation's stewardship; commercial support exists per Apache's pages but its specific vendor list was **not verified here** (§9.2).
- **OpenStack's longevity risk is ecosystem continuity** — that at least one distribution vendor keeps shipping a supported build ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5). The mitigation is a support contract with a distribution vendor.

> A bank's technology committee should write these two risks down **separately**, because they have different mitigations (internal capability vs. a support contract) and different exit costs (§12). Treating "both are open source, so both are equally supported" is a category error and is §14's third anti-pattern.

### 11.6 The bank one-liner

> For a regulated bank the two platforms differ on four specific questions — where the control plane sits (concentrated vs distributed), how sovereignty boundaries are configured, which skills must be hired and retained, and which continuity risk is being underwritten (project/PMC vs distribution/vendor) — and §12 turns exactly those into decision inputs.

---

## 12. How to Decide — A Framework, Not a Verdict

### 12.1 Why this guide names no winner

Naming a winner would be dishonest for four stated reasons:

1. **The two platforms are architecturally different, not better-or-worse.** §3 established that they scale along *different axes*: CloudStack along "more of the same", OpenStack along "more kinds of service". A verdict would have to first collapse those onto one axis, which is the mistake the guide exists to prevent.
2. **The answer depends on inputs this guide cannot see** — the bank's team, market, workload mix, upgrade appetite and support posture (§11). The same two projects produce opposite correct answers under different inputs.
3. **The evidence base does not support a ranking.** The adoption numbers are self-reported and unreliable (§10.1); the feature overlap is large (§8); and no independent, audited, dated comparison exists within this guide's sources.
4. **A verdict invites the first anti-pattern** (§14.1): choosing on a feature matrix and discovering the operational cost later.

So this section is a **method with named inputs**. Run it; it will produce a *defensible* answer for *your* bank, and it will make the assumptions explicit enough to be challenged.

### 12.2 The method — six inputs, each with a discriminating question

The method is: **name the input, answer the question honestly, and record which platform the answer pulls toward.** The inputs are ordered roughly by how strongly they discriminate, based on §3, §6 and §7.

**Input 1 — Team capability and size.**
- Question: *Can we hire and retain engineers who debug a system of Python services, or engineers who own one orchestrator and its database?*
- What it discriminates: a small, traditional infrastructure team pulls toward **CloudStack** (one system, bounded surface, §7.1); a larger, software-engineering-heavy platform team pulls toward **OpenStack** (many services, integration, §7.2).
- Evidence: §7.1, §7.2, §11.4.

**Input 2 — Scale and tenancy model.**
- Question: *Are we mostly adding "more of the same" capacity, or "more kinds" of service, and how many independent tenants must we isolate?*
- What it discriminates: near-linear growth of one VM estate pulls toward **CloudStack** (management-server scales near-linearly, §3.6); growth by adding service types pulls toward **OpenStack** (decomposition, §3.6). Multi-tenant isolation maps to CloudStack's zones/domains and to OpenStack's projects/domains respectively.
- Evidence: §3.6, §4.1 (zones public/private; regions span accounts).

**Input 3 — Workload classes.**
- Question: *What actually runs on it — VM estates, containers, bare metal, GPU/accelerated, network-heavy telco-style?*
- What it discriminates: plain VM estates are well-served by either; container provisioning has a CloudStack published path (CKS/CAPC, §4.11) while OpenStack's container path is **not established here** (§8); bare metal and GPU have documented **CloudStack** entries (BareMetal via IPMI; KVM GPU discovery) while the OpenStack side is **not established here** (§8).
- Evidence: §8, §4.11.

**Input 4 — Upgrade appetite.**
- Question: *Do we need to pin a version and stop for up to 18 months, or can we move on the community's/our vendor's cadence?*
- What it discriminates: pin-and-hold pulls toward **CloudStack** (LTS with 18-month fixes, §4.10); vendor-guided continuous alignment pulls toward **OpenStack** (distributions, §7.3).
- Evidence: §4.10, §7.3, §9.1.

**Input 5 — Support requirement.**
- Question: *Do we need a single accountable vendor with a contract, or can we run on internal capability plus optional support?*
- What it discriminates: "one throat to choke, contractually" pulls toward **OpenStack** (the distribution market, §9.1); "we'll run the project ourselves" pulls toward **CloudStack** (turnkey, small-team posture, §7.1), with the caveat that CloudStack's specific support-vendor market was **not verified here** (§9.2, §16).
- Evidence: §9, §6.6, §11.5.

**Input 6 — Exit cost.**
- Question: *If we leave in year five, what do we lose — a support contract, an integration layer, or the platform itself?*
- What it discriminates: leaving **CloudStack** risks the integrated orchestrator and its project continuity (§6.6); leaving **OpenStack** risks the integration work and the distribution vendor's tested path (§6.6, §9.1). Whichever integration you did *not* do yourself is the one that costs most to leave.
- Evidence: §6.6, §9.4.

### 12.3 The method, as a template

Fill this in for your bank. **The framework does not compute a score** — it forces the inputs to the surface so a committee can argue about the *right* inputs rather than about brochure facts.

| # | Input | Our honest answer | Pulls toward | Weight we assign | Confidence (H/M/L) |
|---|---|---|---|---|---|
| 1 | Team capability and size | | | | |
| 2 | Scale and tenancy model | | | | |
| 3 | Workload classes | | | | |
| 4 | Upgrade appetite | | | | |
| 5 | Support requirement | | | | |
| 6 | Exit cost | | | | |

Rules for using it:

- **Answer in specifics, not adjectives.** "We have four engineers who currently run vSphere and one who writes Python" beats "we have a strong team".
- **Name the dominant input before scoring anything.** In practice one input usually dominates; §13's example shows this.
- **Record confidence.** A high-weight, low-confidence input is a research task, not a decision.
- **Re-run it when an input changes.** Team composition and workload mix change faster than platforms do.

### 12.4 What the method deliberately leaves out

**No feature-matrix weightings** (§8 explained why they carry almost no signal here), **no cost model** (no cost, licence or support-price claim is made in this guide, §11.3, so none can be an input), and **no recommendation** — the outputs are *inputs to a human decision*, which is the only honest place to stop.

---

## 13. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is this repo's only bank persona, and the scenario below is invented to demonstrate the §12 method — not to record a real deployment, a real procurement, or a real vendor decision. No real institution, vendor or jurisdiction is described. Nothing here is a recommendation for any other bank.

### 13.1 The scenario (invented)

Cymbal Bank, an Asia-Pacific bank, decides to build a **private IaaS** for a defined subset of workloads (non-customer-facing test/dev, a virtualisation estate being refreshed, and an emerging GPU/ML sandbox). It will **not** move core banking. It has:

- a **platform engineering team of twelve**, strong in Python and Kubernetes, moderate in traditional infrastructure, with a database specialist;
- a **procurement frame that requires a single accountable support vendor** with a contractual SLA and a named escalation path — the bank's model has no room for "the community will help";
- an estate that is expected to grow by **adding more of the same** over three years, with a smaller expected growth in *kinds* of workload;
- a policy preference to **pin a version where possible** and upgrade on a predictable cadence.

### 13.2 Working the §12 inputs

| # | Input | Cymbal's honest answer | Pulls toward | Weight | Confidence |
|---|---|---|---|---|---|
| 1 | **Team capability and size** | Twelve engineers, strong Python/K8s, one DBA; can debug distributed services | **OpenStack** (slight — the team is the right shape for it) | Medium | High |
| 2 | **Scale and tenancy model** | Mostly "more of the same"; several business-unit tenants to isolate | **CloudStack** | Medium | High |
| 3 | **Workload classes** | VM estate + emerging GPU/ML & K8s sandbox | **CloudStack** (documented BareMetal/GPU/K8s paths; OpenStack side not established here) | Medium | Medium |
| 4 | **Upgrade appetite** | Pin where possible; predictable cadence | **CloudStack** (LTS, 18-month window) | Medium | High |
| 5 | **Support requirement** | **Must contract a single accountable vendor** | **OpenStack** (verified distribution market) | **Dominant — see below** | High |
| 6 | **Exit cost** | A support contract is easier to leave than in-house integration | **OpenStack** (mild) | Low | Medium |

### 13.3 The criterion that dominates

**Input 5 — the support requirement — dominates the decision**, and it is a *procurement* criterion, not a technical one.

Why it dominates: *every* other input is a preference that Cymbal could trade away or mitigate — a team can be re-skilled, a workload mix can be re-homed, an upgrade cadence can be renegotiated. The support requirement is a **constraint**, and Cymbal's procurement frame does not let a constraint be traded away. A platform that cannot satisfy a constraint is out, no matter how well it does on preferences.

Applying the constraint honestly, from the verified material:

- **OpenStack satisfies the constraint**: it has a **verified distribution market** — Red Hat, Canonical and Mirantis ship commercially supported OpenStack distributions ✅ (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5). Cymbal can sign a support contract with a distribution vendor.
- **CloudStack's satisfaction of the constraint is ❌ not established** within this guide's research window: the Apache project's own pages confirm that commercial support *exists as a category* ✅ (systems integrators "that offer CloudStack related services"), but this guide **could not verify a specific commercial CloudStack support vendor at a primary or vendor source** (§9.2, §16). Cymbal's own diligence, working from the same sources, reaches the same impasse.

### 13.4 The contender ruled out — for a reason that is not technical

**CloudStack is ruled out, and it is important to be explicit that the reason is NOT technical.**

On the technical preferences, CloudStack actually did *well* — arguably better than OpenStack:

- It **won Inputs 2, 3 and 4** outright: near-linear management-server scaling for "more of the same" (§3.6); documented BareMetal (via IPMI), GPU discovery, and CKS/CAPC paths for the workload mix (§8, §4.11); and an 18-month LTS window that fits Cymbal's pin-and-hold preference (§4.10).
- It **matched** OpenStack on operational shape for a modest estate: one orchestrator, a bounded component count, a clean failure decomposition (§7.4), and a small-team posture (§7.1).

CloudStack was ruled out **because Cymbal's procurement frame requires a single accountable support vendor and Cymbal could not verify one** — a contractual and market-diligence reason, not a technological one. If Cymbal's procurement frame allowed "internal capability plus an integrator", or if its diligence had surfaced a verifiable support vendor, the answer could plausibly have gone the other way, because the *technical* preferences tilted the other way.

> **This is the honest lesson of the example:** a bank's decision between these two platforms can be decided by a factor that never appears in a feature matrix at all — here, the ability to sign a support contract.

### 13.5 Cymbal's provisional choice (illustrative)

**Provisional decision: an OpenStack distribution, contracted with a distribution vendor**, with the decision to be **re-run if any input changes** — in particular, if Cymbal acquires CloudStack operational expertise or verifies a CloudStack support vendor, Input 5 flips from "OpenStack" to "either", and Inputs 2–4 would then pull the decision back toward CloudStack.

Note what the framework did *not* do: it did not declare OpenStack "better". It declared OpenStack a *better fit for Cymbal's stated constraints*, which is a different and narrower claim, and one a committee can defend.

### 13.6 Lessons from the example

1. **Constraints beat preferences.** A hard procurement constraint can outweigh three technical preferences. Find the constraint first.
2. **The technical tilt was against the chosen product.** Cymbal picked the platform it did *despite* a technical tilt the other way, because the constraint dominated. Any comparison that only looked at technology would have picked wrong *for Cymbal*.
3. **"Not verified" is a real input.** The absence of a verifiable support vendor was decisive. This guide honours ❌ **not established** rather than filling the gap with a plausible name — precisely because, in a real decision, that gap is decision-relevant (§9.2, §16).
4. **Write the re-run trigger down.** The decision is provisional; naming the trigger that would flip it is what makes it an engineering decision rather than a preference.

### 13.7 The thesis, restated

Cymbal did not choose between two feature sets. It chose between **two kinds of thing to own**: an integration project it would buy the integration of, and a product it would have run itself. The constraint that decided it — who signs the support contract — is a statement about *what kind of team and vendor Cymbal is prepared to staff*, which is exactly the thesis:

> **OpenStack is an integration project and CloudStack is a product; the difference you feel is the difference you staff.**

---

## 14. The Anti-Patterns

Five failure modes recur when institutions evaluate these two platforms. Each is given as **symptom → cause → guardrail**, so a reader can recognise it in their own programme.

### 14.1 Anti-pattern 1 — Choosing on a feature matrix, then discovering the operational cost

- **Symptom.** The programme produces a beautiful weighted feature matrix, the platform with the higher score wins, and six to twelve months later the same programme is failing on *operations*: nobody can run the thing, upgrades break, incidents take too long to diagnose.
- **Cause.** The feature matrix measures *what the platform can do*, while the actual cost lives in *what the team must do* to keep it doing it. §8 showed why the matrix has almost no discriminating power here (nine of eleven rows overlap; the rest are "not established"), so the score mostly reflects the evaluator's assumptions, not the platforms. §7 is where the real signal was all along.
- **Guardrail.** Score the **§12 inputs**, not features — team capability, scale/tenancy, workload classes, upgrade appetite, support requirement, exit cost — and require the team that will *operate* the platform to answer Inputs 1 and 5, not the architecture group alone. If a feature row cannot be sourced to a dated primary document, it is not an input.

### 14.2 Anti-pattern 2 — Conflating OpenShift with OpenStack

- **Symptom.** A design document says "we'll standardise on OpenStack" in one paragraph and "OpenShift manages our containers" in another, as if one were the other or one replaced the other; or a committee minutes the two as alternatives. In *this repo* the trap is statistically live: `openshift` appears in 64 files versus `openstack` in 18 (§1.1).
- **Cause.** Four shared letters and a shared "open" prefix, plus genuine adjacency — OpenStack *hosts* Kubernetes platforms and OpenShift *is* one, so the two do co-occur in real architectures, which makes the conflation feel plausible.
- **Guardrail.** State the definitions once, up front (§1.1): **OpenShift is a Kubernetes container platform; OpenStack is an IaaS control plane.** Require every use of either word to carry the category. Where an OpenStack *distribution* runs on OpenShift — as Red Hat's successor distribution does (cross-ref [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5.2) — say so explicitly as a *layering*, never as an identity.

### 14.3 Anti-pattern 3 — Treating one project's commercial ecosystem as the project's own capability

- **Symptom.** A comparison awards "OpenStack" points for a feature that actually lives in a *distribution* (Red Hat's, Canonical's, Mirantis'), or awards "CloudStack" points for a capability that actually lives in a *third-party integration* (an SDN orchestrator, a storage platform, an automation tool). The scoreboard credits the project for someone else's work.
- **Cause.** §9's central observation — the ecosystem, not the code, is often what a bank is buying — is true, but it is true *about the ecosystem*, and collapsing the two hides who is actually accountable for the capability.
- **Guardrail.** Attribute every capability to its **owner**: "OpenStack exposes the API; *this distribution* implements the supported path"; "CloudStack provides the plugin framework; *this storage vendor* provides the driver" (§9.2). In a comparison table, keep a **project** column and a **named-ecosystem** column distinct. Name ecosystem participants factually and never rank them (§9).

### 14.4 Anti-pattern 4 — Planning an upgrade without a documented path

- **Symptom.** The upgrade window is scheduled, the change is approved, and the upgrade is attempted from a version whose documented upgrade path does not exist or skips a required hop.
- **Cause.** Both platforms document upgrades, but not from *any* version to *any* version. CloudStack's downloads page links the upgrade instructions **per release** (4.23.0.0, 4.22.1.1, 4.20.3.1) ✅ **Verified** (CloudStack downloads page, retrieved 2 October 2026), and OpenStack upgrades run through a distribution's supported sequence (§7.3) ✅. A plan that ignores the intermediate hops is a plan with an undocumented step.
- **Guardrail.** For every upgrade, cite the **release-specific** upgrade document (CloudStack) or the **distribution's** supported-path document (OpenStack), and pin the source **version**, not just the target. Build the pin-fix window into the plan: CloudStack LTS gives 18 months of fixes, which is the budget an upgrade plan is written against ✅ **Verified** (CloudStack downloads page).

### 14.5 Anti-pattern 5 — Assuming a smaller footprint means less resilience

- **Symptom.** "CloudStack is one management server, so it's a single point of failure" — or the inverse, "OpenStack has more components, so it's more resilient." Either is asserted without checking.
- **Cause.** Confusing **component count** with **failure probability**. The two are not the same: a *stateless* management server behind a load balancer (✅ CloudStack reliability documentation) can be more available than a larger system with more stateful components, and a system with per-service failure domains can be more available than a smaller one with a shared database.
- **Guardrail.** Reason about the **failure decomposition**, not the component count (§7.4): *what keeps running when this part fails?* CloudStack documents its decomposition precisely — running instances survive a full management-server outage; only management functions stop ✅ **Verified** (CloudStack reliability documentation). OpenStack's decomposition is per service ✅. Compare **documented failure semantics**, and require each platform's claim to be sourced.

The five guardrails in one line each: **(1)** score staffing inputs, not features; **(2)** define OpenShift and OpenStack before comparing them; **(3)** attribute every capability to its actual owner; **(4)** pin every upgrade to a release-specific or distribution-specific document; **(5)** compare documented failure decompositions, never component counts.

---

## 15. The Claims Audit

Every substantive claim in this guide is listed here with a **status**, its **source**, the **date** of the source (or of retrieval), and a **quality** assessment. Status legend: ✅ **Verified** (confirmed at the named primary source), ⚠️ **Flagged** (true but approximate, dated, self-reported, or disputed), ❌ **Rejected / not established** (could not be confirmed and therefore not asserted). Retrieval dates are **2 October 2026** unless a publication date is given. "Project doc" means Apache CloudStack's own documentation; "project site" means `cloudstack.apache.org`.

### 15.1 Apache CloudStack — identity, architecture and operations

| # | Claim | Status | Source | Date | Quality |
|---|---|---|---|---|---|
| C1 | Apache CloudStack is open-source IaaS software, an Apache Software Foundation project, Apache-2.0 licensed | ✅ | cloudstack.apache.org; site footer/trademarks | retrieved 2 Oct 2026 | Primary (project site) |
| C2 | "turnkey solution that includes the entire 'stack' of features most organizations want" | ✅ | cloudstack.apache.org home | retrieved 2 Oct 2026 | Primary (project site) |
| C3 | CloudStack's API "is compatible with AWS EC2 and S3"; a "EC2 API translation layer" is documented | ✅ | cloudstack.apache.org; concepts doc | retrieved 2 Oct 2026 | Primary (project site/doc) |
| C4 | The EC2 translation layer is *actively maintained today* | ❌ | — | — | Not established; guide reports "documented", not "maintained" |
| C5 | CloudStack is "an open source Infrastructure-as-a-Service platform that manages and orchestrates pools of storage, network, and computer resources" | ✅ | concepts doc (4.23.0.0) | retrieved 2 Oct 2026 | Primary (project doc) |
| C6 | Supported hypervisors: BareMetal (via IPMI), Hyper-V, KVM, LXC, vSphere (via vCenter), Xenserver, Xen Project | ✅ | concepts doc (4.23.0.0) | retrieved 2 Oct 2026 | Primary; ⚠ dated to 4.23.0.0 docs |
| C7 | Front page nav lists targets VMware, XenServer, KVM, XCP-ng, BareMetal | ✅ | cloudstack.apache.org | retrieved 2 Oct 2026 | Primary; ⚠ presentation differs from C6, both reported |
| C8 | Manages "tens of thousands of physical servers"; management server "scales near-linearly"; management-server outages don't affect running VMs | ✅ | cloudstack.apache.org; concepts doc | retrieved 2 Oct 2026 | Primary (project site/doc) |
| C9 | Pool of virtual appliances provides firewalling, routing, DHCP, VPN, console proxy, storage access/replication | ✅ | concepts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C10 | "REST-like API" for operation/management/use | ✅ | concepts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C11 | Deployment hierarchy: region ⊃ zone ⊃ pod ⊃ cluster ⊃ host, with the definitions quoted in §4.1 | ✅ | concepts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C12 | Region = largest unit; zone ≈ datacenter; pod ≈ rack; cluster = homogeneous hosts + shared primary storage; host = smallest unit | ✅ | concepts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C13 | Regions and zones visible to end user; pods, clusters, hosts not | ✅ | concepts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C14 | Smallest deployment: one machine as Management Server + hypervisor (KVM) | ✅ | concepts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C15 | Management server runs in Apache Tomcat, requires MySQL; its responsibility list (UI, API/EC2, VM assignment, IPs, storage, snapshots/templates/ISOs, single point of config) | ✅ | concepts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C16 | Management server is stateless (vs MySQL) and may sit behind a load balancer; multi-node recommended | ✅ | reliability doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C17 | Hosts unaffected by management-server outage; "All Guest Instances will continue to work" | ✅ | reliability doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C18 | When management server is down: no new instances; UI, API, dynamic load distribution, HA stop | ✅ | reliability doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C19 | LB port rules: 80/443→8080 (or 20400 AJP) persistence required; 8250→8250 persistence required; 8096→8096 no persistence | ✅ | reliability doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C20 | `host` global config must be the LB VIP for port 8250, else system VMs cannot reach the management server | ✅ | reliability doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C21 | DB replication via MySQL connector parameters (two-way/chain); db.properties settings (`db.ha.enabled`, `db.cloud.replicas`, `db.usage.replicas`, `db.cloud.secondsBeforeRetrySource`) | ✅ | reliability doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C22 | DB HA "Tested with MySQL 5.1 and 5.5" | ⚠ | reliability doc | retrieved 2 Oct 2026 | Verified as written, but **dated test matrix** |
| C23 | Secondary-storage data loss impacts templates/snapshots/ISOs; back up; provision multiple SSVMs | ✅ | reliability doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C24 | `cloudstack-agent` runs on the host, connects on port 8250, certificate auth with `cloud.jks` / `ca.plugin.root.auth.strictness` | ✅ | hosts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C25 | Starting 4.11+, host setting accepts a comma-separated management-server list with round-robin reconnect | ✅ | hosts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C26 | cloudstack-agent installs the qemu script in `/etc/libvirt/hooks` (KVM) | ✅ | hosts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C27 | KVM uses `gpudiscovery.sh` for GPU discovery; requires `lspci`, `xmlstarlet` | ✅ | hosts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C28 | Removing a KVM host: maintenance mode → stop cloud-agent → remove in UI | ✅ | hosts doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C29 | System VMs: managed/created/stopped automatically; expunged on destroy (unlike user VMs) | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C30 | System VM template: single template, Debian 12 (bookworm), 6.1.0 kernel, pvops/Xen PV + KVM virtio + VMware tools, HAProxy/iptables/IPsec/Apache/JRE | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C31 | Since 4.20.0 KVM supports x86_64 and aarch64; other hypervisors x86_64 only; ARM templates not bundled | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C32 | Template bundled in the management DEB/RPM (KVM/VMware/XenServer); `cloud-install-sys-tmplt`; `system.vm.templates.download.repository`, `system.vm.preferred.architecture` | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C33 | System VM types: Console Proxy VM, Secondary Storage VM, Virtual Router (roles as described) | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C34 | Virtual router: no admin login; restart interrupts public access; characteristics set by system service offering (`Domain Router`) | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C35 | System VM SSH access (link-local from host; private IP from management server), port 3922 | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C36 | Console/SSVM LB ports (443/8080 CPVM; 443 SSVM); `consoleproxy.url.domain`, `secstorage.ssl.cert.domain`, `secstorage.encrypt.copy` | ✅ | system-VM doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C37 | Primary storage associated with a cluster (per-zone on KVM/VMware); at least one required; located close to hosts | ✅ | concepts/storage doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C38 | Hyper-V SMB/CIFS, no zone-wide primary storage; Ceph/RBD KVM-only, usable zone-wide | ✅ | storage doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C39 | iSCSI/NFS works with standards-compliant servers; docs name SolidFire, Dell EqualLogic, NetApp, Scale Computing | ✅ | storage doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C40 | Secondary storage is zone-wide (templates/ISOs/snapshots); object storage via Swift or S3 plugins; heterogeneous secondary storage not supported in regions | ✅ | concepts/storage doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C41 | Offerings model: compute (CPU/RAM/tags), disk (size/IOPS), network (features), templates, system service offerings | ✅ | service-offerings doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C42 | Compute offerings may be fixed / custom constrained / custom unconstrained | ✅ | service-offerings doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C43 | Since 4.13, offerings can be scoped to domain(s)/zone(s) via update*Offering APIs | ✅ | service-offerings doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C44 | CloudStack emits usage records integrable with billing systems | ✅ | service-offerings doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C45 | Networking: Basic vs Advanced zones; traffic types; VLAN isolation; VPC; network service providers | ✅ | concepts/networking doc | retrieved 2 Oct 2026 | Primary (project doc) |
| C46 | CloudStack Kubernetes Service (CKS) is a CaaS plug-in | ✅ | cloudstack.apache.org/kubernetes | retrieved 2 Oct 2026 | Primary (project site) |
| C47 | Kubernetes Cluster API Provider for Apache CloudStack (CAPC) is Apache-2.0 and CNCF-managed | ✅ | cloudstack.apache.org/kubernetes | retrieved 2 Oct 2026 | Primary (project site) |
| C48 | Two release types (Regular, LTS); LTS gets fixes for 18 months; Regular gets critical fixes briefly | ✅ | downloads page | retrieved 2 Oct 2026 | Primary (project site) |
| C49 | Latest Regular release 4.23.0.0; current LTS 4.22.1.1; latest 4.20.x maintenance 4.20.3.1 | ✅ | downloads page | retrieved 2 Oct 2026 | Primary (project site) |
| C50 | Community package repos for Ubuntu, EL10/9/8, SUSE/openSUSE 15, experimental ARM64 | ✅ | downloads page | retrieved 2 Oct 2026 | Primary (project site) |

### 15.2 Apache CloudStack — history, governance and published users

| # | Claim | Status | Source | Date | Quality |
|---|---|---|---|---|---|
| C51 | Originated as VMOps (2008); renamed Cloud.com; source released May 2010 under GPLv3 | ✅ | history page | retrieved 2 Oct 2026 | Primary (project site) |
| C52 | Acquired by Citrix July 2011; remainder of code released August 2011; CloudStack 3.0 early 2012 | ✅ | history page | retrieved 2 Oct 2026 | Primary (project site) |
| C53 | Re-licensed to Apache-2.0 April 2012; accepted to Apache Incubator 16 April 2012 | ✅ | history page | retrieved 2 Oct 2026 | Primary (project site) |
| C54 | First major incubator release 4.0.0-incubating 6 November 2012; first minor 4.0.1-incubating 12 February 2013 | ✅ | history page | retrieved 2 Oct 2026 | Primary (project site) |
| C55 | Graduated from incubator 20 March 2013 (announced 25 March 2013) → Apache top-level project | ✅ | history page | retrieved 2 Oct 2026 | Primary (project site) |
| C56 | Governed by an Apache PMC with a PMC Chair; the page names Wido den Hollander as PMC Chair | ✅ | who page | retrieved 2 Oct 2026 | Primary (project site); ⚠ roles change over time |
| C57 | Contributions follow "The Apache Way"; membership published via Apache infrastructure | ✅ | who page | retrieved 2 Oct 2026 | Primary (project site) |
| C58 | Source at github.com/apache/cloudstack | ✅ | project site | retrieved 2 Oct 2026 | Primary (project site) |
| C59 | The published user list (Bell Canada, BT, China Telecom, …) | ⚠ | users page | retrieved 2 Oct 2026 | Project-published, **self-declared via pull request**, "subject to change"; not a market figure |
| C60 | Case material: IKOULA (CloudStack + XCP-ng); Your.Online (CloudStack/KVM/Ceph); AT&T presentation at CloudStack Collaboration Conference 2023; Tungsten Fabric brief | ⚠ | cloud-builders page | retrieved 2 Oct 2026 | Project-published case material, dated; not audited |
| C61 | Integrations page names Apiculus, Tungsten Fabric, StorPool, Terraform as integrations/partners | ⚠ | integrations page | retrieved 2 Oct 2026 | Project-published; **named factually, not ranked** |
| C62 | A specific commercial CloudStack **support vendor** | ❌ | — | — | Not verified at a primary/vendor source in this guide's window (§9.2, §16) |

### 15.3 OpenStack — claims used by this guide

| # | Claim | Status | Source | Date | Quality |
|---|---|---|---|---|---|
| O1 | OpenStack created in the first months of 2010; Rackspace rewriting its cloud; Anso Labs' beta Nova for NASA | ✅ | OpenStack project-team guide | retrieved 2 Oct 2026 | Primary (project doc) |
| O2 | First Design Summit Austin 13–14 July 2010; announced at OSCON 21 July 2010; wiki 24 May 2010 | ✅ | OpenStack project-team guide | retrieved 2 Oct 2026 | Primary (project doc) |
| O3 | First release "Austin" October 2010 | ⚠ | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.1 | — | Sibling guide; **discrepancy with O2 flagged in §5.3** |
| O4 | OpenStack Foundation launched September 2012; Board + Technical Committee + (orig.) User Committee | ✅ | OpenStack project-team guide | retrieved 2 Oct 2026 | Primary (project doc) |
| O5 | TC moved to 13 directly-elected members (June 2013, half renewed every 6 months) | ✅ | OpenStack project-team guide | retrieved 2 Oct 2026 | Primary (project doc) |
| O6 | "Big tent" project-structure reform December 2014 | ✅ | OpenStack project-team guide | retrieved 2 Oct 2026 | Primary (project doc) |
| O7 | User Committee merged into Technical Committee since June 2020 | ✅ | OpenStack project-team guide | retrieved 2 Oct 2026 | Primary (project doc) |
| O8 | Rename to OpenInfra Foundation announced October 2020 | ✅ | superuser.openinfra.org article, 28 Oct 2020; openinfra.org blog 2021 | 28 Oct 2020 | Primary (foundation); ⚠ sibling says 2021 — discrepancy flagged §6.4 |
| O9 | OpenInfra signaled intent to join the Linux Foundation, 12 March 2025 | ✅ | Linux Foundation press release | 12 Mar 2025 | Primary (press release); ⚠ intent, not completion at that date |
| O10 | As of 2026, "OpenInfra Foundation is part of the nonprofit Linux Foundation" | ✅ | openinfra.org/about | retrieved 2 Oct 2026 | Primary (foundation); ⚠ sibling says 2024 — discrepancy flagged §6.4 |
| O11 | Foundation's published claims: "300+ Public Cloud Data Centers"; "580k+ Changes"; "55M+ Cores" | ⚠ | openinfra.org home | retrieved 2 Oct 2026 | **Self-reported foundation claims**; not audited |
| O12 | Component roster (Nova/Neutron/Cinder/Glance/Keystone/Swift/Heat + Horizon/Ceilometer) | ⚠ | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §3.2 | — | Cross-referenced, not independently re-verified here |
| O13 | OpenStack distributions: Red Hat, Canonical, Mirantis | ⚠ | [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) §5 | — | Cross-referenced; their status markers live there |
| O14 | OpenStack Kubernetes provisioning, bare metal, GPU support | ❌ | — | — | **Not established from this guide's sources** (§8) |

### 15.4 Comparison and judgement claims

| # | Claim | Status | Note |
|---|---|---|---|
| X1 | CloudStack = management server + agents + system VMs; OpenStack = many services behind a shared API | ✅ | Directly sourced from each project's own documentation/roster (§3) |
| X2–X5 | Scaling differs by axis; smaller component count is easier to start and the larger easier to extend; the governance difference is what a bank committee actually chooses; the ecosystem is often what a bank is buying | ⚠ | Judgements, each argued from ✅ sources and marked inline (§3.6, §6.6, §7.5, §9.4) |
| X6 | The §12 framework and §13 worked example | ⚠ | **Method and a fictional illustration**, not an empirical finding; §13 explicitly invented |
| X7 | Any adoption, market-share, cost, licence or support-price figure | ❌ | **None is printed as fact** anywhere in this guide (§10.1, §11.3) |

---

## 16. What Could Not Be Verified, Glossary and References

### What Could Not Be Verified

An honest guide records its gaps. Each entry below is something this guide wanted to state and could not source to an acceptable primary document within its research window (Apache CloudStack's own documentation, the projects' own published material, and the sibling guides). **None of these was filled with a plausible-sounding guess.**

1. **A specific commercial CloudStack support vendor.** This is the most consequential gap, because §13's worked example turned on it. Apache's own pages confirm that commercial support *exists as a category* ("systems integrators that offer CloudStack related services") ✅ **Verified** (cloudstack.apache.org/users, retrieved 2 October 2026), but this guide **could not verify a named vendor of commercial CloudStack support at a primary or vendor source**, so it names **none**. Any specific CloudStack support vendor you may have encountered is deliberately **not asserted here**; a reader who needs one must verify it at the vendor's own documentation.
2. **Whether CloudStack's EC2 API translation layer is currently maintained.** The EC2 translation layer and the AWS EC2/S3 compatibility statement are documented in the current documentation ✅ (as documented), but the documentation does not state the layer's current maintenance status. ❌ **Not established** → reported as "documented", not as "maintained" (C4, §2.1, §4.8).
3. **OpenStack's Kubernetes-provisioning, bare-metal and GPU capabilities in a directly comparable form.** The pack and the sibling guides did not carry these in a form this guide could verify at a primary source, so the §8 table prints **"not established from this guide's sources"** for the OpenStack column on those rows — which is a statement about *this guide's sources*, **not** about OpenStack's capabilities (O14, §8.1).
4. **The exact date of the OpenStack Foundation → OpenInfra Foundation rename.** The Foundation's own article dates the announcement to the virtual Summit of 2020 (article dated **28 October 2020**) and the openinfra.org blog references it in 2021; the sibling guide says "2021". ⚠ **Discrepancy flagged, not resolved** (§6.4, O8).
5. **The exact completion date of the OpenInfra Foundation → Linux Foundation move.** The primary source is an **intent announcement dated 12 March 2025**; openinfra.org in 2026 states membership is a fact ("part of the nonprofit Linux Foundation"); the sibling guide says "2024". ⚠ **Discrepancy flagged, not resolved** (§6.4, O9, O10).
6. **The current supported database versions for CloudStack management-server HA.** The reliability documentation's tested matrix reads "MySQL 5.1 and 5.5" ✅ (as written), which is long superseded. ⚠ **Dated**; a 2026 operator must confirm the current matrix in the current admin guide (C22, §4.3).
7. **All adoption, market-share, cost and support-price figures.** ❌ **None verified**; the only adoption numbers available are self-reported by the projects' own foundations (C59–C61, O11), and §10.1 explains why public adoption numbers in this space are unreliable.

### Glossary

**CloudStack terms**

- **Apache CloudStack** — the Apache Software Foundation's open-source IaaS platform (§2.1). **Management server** — its central orchestrator; runs in Tomcat, needs MySQL, exposes UI + API (§4.2). **Agent (`cloudstack-agent`)** — the process on each hypervisor host, connecting on port 8250 (§4.4).
- **System VM** — a small CloudStack-managed VM that provides a service to the cloud itself: the **Console Proxy VM (CPVM)** gives console access, the **Secondary Storage VM (SSVM)** does template/snapshot processing, and the **Virtual Router** provides routing, DHCP, NAT, LB, VPN and firewall for a guest network with no admin login (§4.5).
- **Region / Zone / Pod / Cluster / Host** — the hierarchy, largest to smallest: region ⊃ zone ⊃ pod ⊃ cluster ⊃ host (§4.1). **Primary storage** stores VM virtual disks (per-zone on KVM/VMware); **secondary storage** is the zone-wide store of templates, ISO images and snapshots (§4.6).
- **Service / Disk / Network offering** — the packaged choices granted to users (CPU/RAM; disk size/IOPS; network features) (§4.7). **Template** — CloudStack's bootable image for instance creation. **CKS** — the CloudStack Kubernetes Service CaaS plug-in; **CAPC** — the CNCF-managed Kubernetes Cluster API Provider for Apache CloudStack (§4.11).

**OpenStack terms**

- **OpenStack** — the OpenInfra Foundation's open-source IaaS platform; many cooperating services (§2.2). **Nova / Neutron / Cinder / Glance / Keystone / Swift / Heat** — compute / networking / block storage / images / identity / object storage / orchestration (§5.1, cross-ref sibling §3.2).
- **OpenInfra Foundation** — the former OpenStack Foundation, renamed 2020, now part of the Linux Foundation; its **Technical Committee** holds "authority over upstream projects", and the December 2014 **"big tent"** reform moved to a community-centric definition of OpenStack (§6.3–6.4). **Distribution** — a vendor's packaged, supported OpenStack build (§9.1).

**Shared terms**

- **IaaS** (infrastructure-as-a-service), the **control plane / data plane** split, the **hypervisor**, the **image / template** pair (image in OpenStack, template in CloudStack) and the **catalogue** as a governance surface are all defined in §1.3. **Cymbal Bank** is the fictional persona of §13 and the repository's only bank persona.

### Cross-References

Sibling guides in this repository, all under `technology/` unless noted:

- **[nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md)** — owns OpenStack's origins (§3.1), component roster (§3.2), OpenStack table (§3.3), release timeline (§3.6), the HCI-vs-disaggregated head-to-head (§4), and the OpenStack distributions (§5). **Primary cross-reference for the OpenStack side of this guide.**
- **[devstack_openstack_guide.md](devstack_openstack_guide.md)** — owns DevStack, the all-in-one topology and the service roster (§4.2), and release naming / calendar versioning (§6).
- **[openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md)**, **[charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md)**, **[openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md)** — the OpenShift/Kubernetes material that §1.1's HAZARD 1 depends on being *distinct* from OpenStack.
- **[cloud_providers_guide.md](cloud_providers_guide.md)** — the vendor-neutral cloud framing behind §11.1.
- **[cephfs_alternatives_guide.md](cephfs_alternatives_guide.md)** and **[s3_architecture_guide.md](s3_architecture_guide.md)** — the storage layer both platforms depend on (§4.6, §11.1).
- **[on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md)** — the on-prem AI workload case that consumes an IaaS layer (§11.1).
- **[../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)** — the Singapore regulatory context referenced (by name only, with no rule asserted) in §11.2.

### Primary Sources

- Apache CloudStack: `cloudstack.apache.org` (home, history, who, users, downloads, Kubernetes, integrations, cloud-builders pages) and `docs.cloudstack.apache.org` (concepts, reliability, hosts, system VM, storage, service-offerings, networking documentation) — the **4.23.0.0** documentation set; all retrieved **2 October 2026**.
- OpenStack: the OpenStack project-team guide, "A Bit of OpenStack History" (`docs.openstack.org/project-team-guide/introduction.html`) — retrieved 2 October 2026.
- OpenInfra Foundation / Linux Foundation: `openinfra.org` (home, about), `superuser.openinfra.org` ("Virtual Open Infrastructure Summit Recap", 28 October 2020), and the Linux Foundation press release "Open Infrastructure Foundation Board Announces Intent to Join the Linux Foundation" (12 March 2025) — retrieved 2 October 2026.

### Closing Summary

Two platforms, one shared licence, and two architectures that are not on the same axis. CloudStack is a **product**: one management server, thin agents, system VMs that do the plumbing, a documented hierarchy of region ⊃ zone ⊃ pod ⊃ cluster ⊃ host, a published Regular-plus-LTS cadence, and an Apache PMC's stewardship — easy to start, defined to extend, concentrated where it fails. OpenStack is an **integration project**: many cooperating services behind a shared API, a foundation and a distribution market, consumed almost always through a vendor's tested build — slower to start, composed to extend, distributed where it fails. Neither is better; they are different, and §12 exists to turn that difference into a decision rather than a slogan. The thing a bank's technology committee is really choosing is not a feature set but a **shape of team**, because the architecture decides what its operators will spend their days doing. Which is why this guide ends where it began:

OpenStack is an integration project and CloudStack is a product; the difference you feel is the difference you staff.
