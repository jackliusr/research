# Nutanix vs OpenStack — Commercial HCI vs Open-Source IaaS

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Technology Research — Infrastructure / Private-Cloud series; the dedicated vendor head-to-head between the Nutanix Cloud Platform (commercial hyperconverged infrastructure) and OpenStack (the open-source infrastructure-as-a-service platform)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** OpenStack/OpenInfra Foundation materials and user surveys; Rackspace and NASA launch coverage; Red Hat, Canonical and Mirantis product pages; Nutanix product documentation (cross-ref [nutanix_products_guide.md](nutanix_products_guide.md)); industry press (The Register, The New Stack, Computer Weekly)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Overview — The Two Platforms](#1-overview--the-two-platforms)
2. [The Nutanix Side — The Commercial HCI](#2-the-nutanix-side--the-commercial-hci)
3. [The OpenStack Side — The Open-Source IaaS](#3-the-openstack-side--the-open-source-iaas)
4. [Head-to-Head — The Comparison](#4-head-to-head--the-comparison)
5. [The Distributions — OpenStack's Vendor Layer](#5-the-distributions--openstacks-vendor-layer)
6. [The Ecosystem Comparison](#6-the-ecosystem-comparison)
7. [Selection Guidance — Which for Which Need](#7-selection-guidance--which-for-which-need)
8. [Worked Example — A Private-Cloud Platform Selection](#8-worked-example--a-private-cloud-platform-selection)
9. [Summary — The Buy vs the Build of the Private Cloud](#9-summary--the-buy-vs-the-build-of-the-private-cloud)
10. [Glossary](#10-glossary)
11. [Claims Status, References and Further Reading](#11-claims-status-references-and-further-reading)

### How to Read This Guide

This is the dedicated deep-dive on **Nutanix vs OpenStack** — the head-to-head comparison between the two most common answers to the private-cloud question — in the `technology/` infrastructure / private-cloud series. Sibling guides carry adjacent depth and are cross-referenced inline:

- **The Nutanix side** — [nutanix_products_guide.md](nutanix_products_guide.md) is the full Nutanix portfolio survey (AOS/AHV/Prism, Files/Objects/Volumes, NDB, Flow, NC2, GPT-in-a-Box); this guide summarizes the platform and cross-refers rather than repeating it.
- **The comparison patterns** — [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) and [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) follow the same head-to-head structure (overview → platform A → platform B → head-to-head → selection → worked example); [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) is the adjacent "alternatives" survey. The K8s guides matter here because OpenStack's modern distributions increasingly run *on* Kubernetes (Red Hat OpenStack Services on OpenShift, Mirantis OpenStack for Kubernetes — see §5).
- **The cloud angle** — [cloud_providers_guide.md](cloud_providers_guide.md) is the vendor-neutral cloud comparison that frames why a private cloud exists at all; [finops_guide.md](finops_guide.md) is the cost lens that §4.4's economics mirror.
- **The storage angle** — [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) (CephFS), [dell_objectscale_guide.md](dell_objectscale_guide.md) and [s3_architecture_guide.md](s3_architecture_guide.md) are the storage references that both Ceph (OpenStack's usual backing store) and Nutanix Unified Storage compete against.
- **The sovereign-infra angle** — [htx_ngine_guide.md](htx_ngine_guide.md) shows Nutanix as the platform layer in Singapore's sovereign-AI build — a data-point for the "buy" side of §9.
- **The bank reality** — the `banking/` guides give the regulated-sector context that the worked example in §8 reuses; [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) is the on-prem AI playbook relevant to both platforms' AI trajectories.

**Note on verification.** This guide was researched in August 2026. Claims are marked **Verified** (confirmed against primary sources or reliable secondary coverage), **Reported** (widely reported but not independently confirmed), or **flagged** inline where specifics are approximate or could not be pinned down. The full claims-status table is in §11.

---

## 1. Overview — The Two Platforms

### 1.1 The Two Platforms

> **Nutanix is the commercial HCI: a proprietary, subscription-licensed, hyperconverged platform that pools compute and storage into one software-defined cluster. OpenStack is the open-source IaaS: a free, modular cloud-operating-system project that turns pools of servers, storage and network into a self-service cloud. One is a product you buy; the other is software you build with.**

The private-cloud question — "we want cloud-style self-service, but the data stays in our data center" — has two canonical answers, and they are philosophically opposite:

**Nutanix** (Nasdaq: NTNX, founded 2009, the company that invented the modern HCI category — Verified, cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §1) sells a **commercial, integrated platform**. Compute virtualization and enterprise storage are fused into a distributed operating system (AOS), a built-in hypervisor (AHV) is included free, and one management console (Prism) runs the whole estate. You buy a subscription, you get a supported, opinionated product, and the vendor does the integration work for you. Nutanix's FY2025 annualized recurring revenue reached **$2.223B, up 17% YoY, at a 108% net dollar-based retention rate** (Reported — Q4 FY2025 earnings, The Motley Fool transcript; exact figures move quarterly).

**OpenStack** is a **free, open-source cloud platform** launched in **October 2010** by **Rackspace and NASA** (Verified — see §3.1) and now governed by the **OpenInfra Foundation** (the renamed OpenStack Foundation, 2021 — Verified). It is not a product: it is a collection of loosely coupled services (Nova for compute, Neutron for networking, Cinder for block storage, and so on) that you assemble, configure, operate, and support yourself — or buy as a packaged **distribution** from a vendor such as Red Hat, Canonical, or Mirantis (§5). OpenStack reports **40+ million cores in production as of 2022 and 55+ million by the 2025 user survey** (Reported — self-reported survey data, The Register / OpenInfra; treat as directional). It is the invisible foundation of much of the world's telco and research clouds, yet it is rarely the story an enterprise tells about itself.

The cleanest way to hold the comparison: **Nutanix sells you a private cloud as a product ("buy"); OpenStack gives you the raw materials to build one ("build").** Everything else — architecture, operations, economics, skills, risk — flows from that difference.

### 1.2 The Comparison Scope

This guide compares the two platforms across five lenses, in the same order the sibling head-to-head guides use (see [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md)):

| # | Lens | What it covers |
|---|---|---|
| 1 | **Positioning** | What each platform claims to be, who it targets, and where it sits in the market (product vs project; HCI vendor vs open-source community) |
| 2 | **Architecture** | How compute, storage and network are organized — Nutanix's converged (hyperconverged) fabric vs OpenStack's disaggregated, composable service layer |
| 3 | **Components** | The building blocks on each side: AOS/AHV/Prism vs Nova/Neutron/Cinder/Glance/Keystone/Swift/Heat |
| 4 | **Operations** | Day-1 deployment and day-2 operations: Nutanix's one-console simplicity vs OpenStack's many-services complexity |
| 5 | **Economics** | Licensing and support: Nutanix's subscription fees vs OpenStack's open-source license with optional vendor support |

A sixth lens — the **ecosystem** (distributions, community, market trajectory) — is covered in §5–§6 because it is where the "OpenStack is declining / OpenStack is thriving" debate actually lives, and the honest treatment matters (§6.2).

### 1.3 The Overview Table

| Aspect | Nutanix | OpenStack |
|---|---|---|
| **What it is** | Commercial hyperconverged infrastructure (HCI) platform + cloud services (block/file/object storage, DBaaS, networking, AI) | Open-source infrastructure-as-a-service (IaaS) platform — a modular collection of cloud services |
| **Model** | Product — proprietary software, subscription licensing, vendor-supported | Project — free software (Apache 2.0), assembled by the user or a distribution vendor |
| **Origins** | Founded 2009, San Jose, CA; HCI pioneer (Verified — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md)) | Launched October 2010 by Rackspace + NASA (Verified — §3.1) |
| **Governance** | Nutanix, Inc. (Nasdaq: NTNX) | OpenInfra Foundation (ex-OpenStack Foundation, renamed 2021; joined Linux Foundation 2024 — Reported) |
| **Core components** | AOS (distributed OS), AHV (KVM hypervisor, free), Prism (management) | Nova, Neutron, Cinder, Glance, Keystone, Swift, Heat (+ Horizon, Ceilometer, …) |
| **Architecture** | Hyperconverged — compute + storage fused per node, scale-out by adding nodes | Disaggregated — separate pools of compute/storage/network, composed via APIs |
| **Deployment** | Appliance-style: buy nodes, run installer, one console; hours to days | Project-style: design, integrate, deploy 10+ services; weeks to months |
| **Operations** | One vendor, one console (Prism), one upgrade path, one support contract | Many services, many moving parts; ops burden on you or your distro vendor |
| **Licensing cost** | Subscription: per-core + per-TiB (AHV included); no hardware requirement | Free (Apache 2.0); you pay for hardware, engineering, and optional vendor support |
| **Typical buyer** | Mainstream enterprise, regulated industries (banking, healthcare, government), VMware-replacement deals | Telcos (NFV), research/HPC, hyperscale-style cloud builders, large enterprises with cloud-engineering teams |
| **Ecosystem** | One vendor's integrated portfolio + OEM hardware partners | Large open-source community + commercial distributions (Red Hat, Canonical, Mirantis) + 55M+ production cores (Reported) |
| **The one-line story** | "The private cloud you buy" | "The private cloud you build" |

---

## 2. The Nutanix Side — The Commercial HCI

### 2.1 Nutanix and HCI — The Converged Model (Verified)

Nutanix is the company that invented the modern **hyperconverged infrastructure (HCI)** category (Verified — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §1–§2, which this section summarizes). The founding insight (2009, founders Dheeraj Pandey, Mohit Aron, Ajeet Singh) was that the classic three-tier data center — separate servers, a SAN, and a network fabric — was expensive, fragile, and hard to scale. Nutanix's bet: run the *entire* stack, compute virtualization and enterprise storage, as distributed software on standard x86 servers.

The result is **convergence**: in a Nutanix cluster, every node contributes CPU, RAM, and local disk, and the AOS distributed operating system pools them into one resilient, self-healing fabric. There is **no separate SAN** — storage is virtualized and distributed across every node's local disks, replicated (RF2/RF3) or erasure-coded across nodes, and managed as one logical pool. Scale-out means adding a node, which grows compute *and* capacity together — the operational difference Nutanix customers cite first (Verified — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §2.6).

Key facts (all **Verified** — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §1–§2):

- **Founded** 23 September 2009; **IPO** 30 September 2016 on Nasdaq (NTNX), priced $16, closed day one ~$37 (+131%).
- **Business model today:** 100% software, subscription/term licensing, hardware-agnostic — servers come from OEM partners (Dell, HPE, Lenovo, Cisco, Fujitsu), and the same software runs on-prem or on public-cloud bare metal via **Nutanix Clusters (NC2)**.
- **Scale:** ~35,000+ customers claimed across ~150 countries (Flagged — company claim, not independently audited); ARR $2.223B at end of FY2025, +17% YoY (Reported).
- **Market position:** HCI pioneer; long-time Leader in Gartner's HCI Magic Quadrant (2019–2022 era, Reported); competes with VMware vSAN, HPE SimpliVity, Scale Computing — and, at the platform level, with OpenStack.

### 2.2 The Components — AOS, AHV, Prism (Verified)

The Nutanix platform core is three tightly integrated layers, all **Verified** against Nutanix product documentation (and summarized from [nutanix_products_guide.md](nutanix_products_guide.md) §2):

**AOS — Acropolis Operating System.** The distributed OS and engine room of the platform. It runs a distributed, replica-based storage engine (historically "Stargate") that stripes data across all nodes, keeps cluster metadata in a distributed (Cassandra-style) store, and builds in the data services: snapshots, clones, thin provisioning, deduplication, compression, self-healing re-replication, and async DR replication. AOS natively serves block (iSCSI — Volumes), file (NFS/SMB — Files), and object (S3 — Objects) from the same fabric — which is why Nutanix's storage story is "one engine, every protocol" rather than three siloed products.

**AHV — Acropolis Hypervisor.** Nutanix's built-in enterprise hypervisor, based on open-source **KVM** (libvirt for VM lifecycle, OVS for virtual switching). It is **included free with AOS** — no per-socket license — which is the single biggest cost differentiator versus VMware vSphere/vSAN stacks (and, for this comparison, versus an OpenStack build where the hypervisor (usually KVM) is free but everything around it is your engineering). AHV covers live migration, HA, snapshots, and clones, and upgrades ship with AOS as one-click rolling updates. Honest caveat (Flagged as experience, not vendor claim): some advanced vSphere features and third-party integrations are thinner on AHV.

**Prism — The Management Plane.** Prism Element manages a single cluster (the "single pane" for VMs, storage, alerts, capacity); Prism Central manages fleets — multi-cluster governance, capacity planning, policy-based automation (tags/categories), RBAC self-service projects, and a marketplace-style app catalog. Prism is where the other services hang off: Calm (automation), Flow (networking/security), the licensing dashboard. The design philosophy is deliberately "cloud console meets enterprise ops": health is scored (red/yellow/green), maintenance is wizard-driven rather than runbook-driven.

### 2.3 The Nutanix Table

| Component | Function | Notes |
|---|---|---|
| **AOS** (Acropolis Operating System) | Distributed OS: pools compute + storage across nodes; serves block/file/object protocols; snapshots, clones, dedupe, compression, DR built in | The "engine room"; licensed per core + flash capacity; upgrades in place, orchestrated by Prism (Verified — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §2.5) |
| **AHV** (Acropolis Hypervisor) | KVM-based enterprise hypervisor for VMs: live migration, HA, snapshots, clones | Included free with AOS — the cost killer vs vSphere; upgrades ship with AOS |
| **Prism** | Management plane: Prism Element (per cluster) + Prism Central (multi-cluster governance, self-service, marketplace) | The "single pane"; also hosts Flow, Calm, and the licensing console |
| **Supporting cast** | Stargate (storage engine), NDFS (distributed filesystem), Controller VM (per-node I/O path), metadata store | Internal AOS components — explain why Nutanix behaves like one big distributed system |
| **Services (context)** | Files, Objects, Volumes, NDB, Flow, Calm, Beam, Mine, NC2, GPT-in-a-Box | The portfolio beyond the HCI core — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §3–§8 for the full survey |

### 2.4 What the Nutanix Side Means for This Comparison

**The stack at a glance** (how the layers relate — the "one platform" picture):

```
┌──────────────────────────────────────────────────────────┐
│  Services layer:  Files │ Objects │ Volumes │ NDB │ Flow │
│                    Calm │ Beam │ Mine │ GPT-in-a-Box      │
├──────────────────────────────────────────────────────────┤
│  Management plane:  Prism Element + Prism Central        │
├──────────────────────────────────────────────────────────┤
│  Compute:  AHV (KVM-based hypervisor, included free)     │
├──────────────────────────────────────────────────────────┤
│  OS:  AOS — distributed storage fabric (RF2/RF3, EC-X),  │
│       NDFS, Controller VMs, snapshots/dedupe/compression │
├──────────────────────────────────────────────────────────┤
│  Hardware:  standard x86 nodes (OEM partners)            │
└──────────────────────────────────────────────────────────┘
```

Three things carry forward into the head-to-head:

1. **Integration is the product.** Nutanix sells you the *absence of integration work*: compute, storage, management, upgrades, and support are one system from one vendor. The price of that is lock-in — you cannot take AOS apart and swap in another storage engine.
2. **The hypervisor is free, the platform is not.** The license economics are subscription (cores + TiB), which is a predictable opex line — and, unlike vSphere-era VMware, there is no separate hypervisor tax.
3. **The cloud story is "same platform anywhere."** NC2 runs the identical stack on AWS/Azure/GCP bare metal — a private-cloud-first hybrid story that is orthogonal to OpenStack's "build your own cloud" model.

### 2.5 Deploying Nutanix — The Walkthrough

| Step | What happens | Skills required |
|---|---|---|
| 1 — Order & rack | Servers from an OEM partner (Dell/HPE/Lenovo/Fujitsu), racked and cabled on 10/25 GbE | Rack-and-stack hands; standard DC ops |
| 2 — Foundation install | Run the Foundation tool: boots nodes, discovers hardware, configures IPMI and networking | Basic Linux/network literacy; ~1 week of vendor training |
| 3 — Cluster creation | Prism Element wizard: name the cluster, add nodes, choose RF2/RF3/EC-X and storage containers | Certified platform admin (~2 weeks of training) |
| 4 — AHV enablement | AHV is enabled during cluster creation — there is no separate hypervisor install or upgrade project | Same as above (no vSphere-style hypervisor lifecycle) |
| 5 — Prism Central | Deploy Prism Central for multi-cluster management, RBAC, projects, marketplace | Platform admin |
| 6 — First workloads | Create VMs, provision NDB databases, stand up Files shares — all from one console | App teams consume self-service; admin approves |
| 7 — Day 2 | Capacity monitoring, health checks, rolling AOS+AHV upgrades, add nodes to scale | Platform admin; upgrades are wizard-driven |
| **Typical timeline** | **First VM in hours-to-days; production cluster in 1–2 weeks** | **Small team, vendor-trained** |

---

## 3. The OpenStack Side — The Open-Source IaaS

### 3.1 OpenStack — Origins: 2010, Rackspace + NASA (Verified)

**OpenStack is a free, open-source cloud-computing platform, launched in October 2010 as a joint project of Rackspace and NASA** (Verified — Rackspace's own history pages and The Register's coverage of the first release; the project was announced July 2010 and the first release, "Austin," shipped 21 October 2010).

The founding details matter because they explain the project's DNA:

- **NASA** contributed the compute controller from its **Nebula** cloud project, which became **OpenStack Nova** — the compute service.
- **Rackspace** contributed its **Cloud Files** platform, which became **OpenStack Swift** — the object-storage service.
- The two services were released together as the Austin release in October 2010, and the project quickly attracted a who's-who of backers (Canonical, Dell, HP, IBM, Intel, Red Hat among the early supporters — Reported, widely documented).
- Governance moved to the nonprofit **OpenStack Foundation** in 2012, which was renamed the **OpenInfra Foundation** in 2021 (Verified — OpenInfra/OpenStack materials); in 2024 the foundation announced it would join the **Linux Foundation** (Reported — Network World coverage).
- The project releases on a six-month cadence, named by letter (Austin → … → Zed → 2023.1 Antelope → 2025.2 Flamingo, the current release as of this guide — Reported; release names and dates move).

The one-sentence positioning OpenStack itself uses: an open-source platform that delivers **infrastructure-as-a-service (IaaS)** — virtual servers, storage, and networking provisioned through a self-service API and dashboard, deployable in public or private clouds. In plain terms: **OpenStack is the software blueprint for running a cloud the way AWS runs one, inside your own data center — if you have the engineering to assemble and operate it.**

### 3.2 The Components — Nova, Neutron, Cinder, Glance, Keystone, Swift, Heat (Verified)

OpenStack is not one program; it is a collection of **loosely coupled services** that communicate over message queues and REST APIs, each with its own database and (usually) its own horizontally scalable set of worker processes. The core seven (all **Verified** against OpenStack project documentation and the project's own "What is OpenStack" materials):

- **Nova — Compute.** Manages the lifecycle of virtual machines (instances): scheduling, provisioning, live migration, resize, and termination. Nova itself is hypervisor-agnostic — it drives KVM (the default), VMware, Hyper-V, Xen, and (for containers/edge) Kata Containers and Zun through driver layers. The compute workhorse: if you run OpenStack, Nova is what runs your VMs.
- **Neutron — Networking.** Provides networking-as-a-service: virtual networks, subnets, routers, load balancers, firewalls (security groups), and floating IPs, built on overlays (VXLAN/Geneve) with an SDN-style plugin architecture (Open vSwitch is the common default; vendor plugins hook in hardware or commercial controllers).
- **Cinder — Block Storage.** Provides persistent block volumes that attach to instances (like EBS to EC2). It manages volume lifecycle — create, snapshot, clone, attach, detach, resize — over a huge range of backends through drivers (LVM, Ceph RBD, NFS, and commercial arrays).
- **Glance — Images.** The image registry: stores and serves VM disk images and snapshots (QCOW2, RAW, VHD, …) in a catalog that Nova boots instances from. Typically backed by Swift or Ceph or a filesystem.
- **Keystone — Identity.** The authentication and authorization service for the whole platform: users, projects (tenants), roles, tokens, and service catalogs. Every API call in OpenStack passes through Keystone's token validation — it is the front door of the cloud.
- **Swift — Object Storage.** The original Rackspace contribution: distributed, highly available object storage with an S3-compatible-ish API surface (S3 middleware exists, though true S3 compatibility is a perpetual project — Flagged: OpenStack's Swift API is its own REST API; S3 compatibility is provided via middleware and varies in completeness). Built for scale-out on commodity hardware with replication and erasure coding.
- **Heat — Orchestration.** Template-driven orchestration: declares stacks of cloud resources (instances, volumes, networks, and even autoscaling groups) in HOT (Heat Orchestration Template) format — OpenStack's answer to AWS CloudFormation. Heat is the "infrastructure-as-code on OpenStack" layer.

**The supporting cast** (worth knowing, commonly deployed in real clouds): **Horizon** (the web dashboard), **Ceilometer/Gnocchi** (metering and metrics), **Ironic** (bare-metal provisioning), **Octavia** (load-balancer-as-a-service), **Manila** (shared file storage), **Magnum** (Kubernetes clusters on OpenStack), **Barbican** (secret/key management), **Designate** (DNS), and — critically for storage in production — **Ceph** is not an OpenStack project but is the de-facto standard backing store for Cinder and Glance in real deployments (cross-ref [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) for the Ceph angle).

### 3.3 The OpenStack Table

| Component | Function | Notes |
|---|---|---|
| **Nova** | Compute: VM lifecycle (schedule, provision, migrate, resize) | Hypervisor-agnostic drivers; KVM is the default; the workload workhorse (Verified) |
| **Neutron** | Networking-as-a-service: VPCs, subnets, routers, LB, security groups, floating IPs | Overlay-based (VXLAN/Geneve) with plugin architecture; Open vSwitch common default |
| **Cinder** | Block storage: persistent volumes attached to instances | Backend-agnostic drivers (LVM, Ceph RBD, arrays); the EBS analogue |
| **Glance** | Image registry: VM disk images and snapshots for booting | Backed by Swift/Ceph/filesystem; the AMI analogue |
| **Keystone** | Identity: users, projects, roles, tokens, service catalog | The front door — every API call authenticates through it |
| **Swift** | Object storage: distributed, replicated/erasure-coded | The Rackspace heritage (2010); its own REST API with S3 middleware (Flagged: S3-compat completeness varies) |
| **Heat** | Orchestration: template-driven stacks (HOT format) | The CloudFormation analogue; infrastructure-as-code for the cloud |
| **Supporting cast** | Horizon (UI), Ceilometer (metering), Ironic (bare metal), Octavia (LBaaS), Manila (files), Magnum (K8s), Barbican (secrets) | Production clouds typically run most of these; Ceph is the usual external storage backbone |

### 3.4 What the OpenStack Side Means for This Comparison

**The stack at a glance** (how the services relate — the "assemble it yourself" picture):

```
┌──────────────────────────────────────────────────────────┐
│  User surface:  Horizon (UI) │ CLI │ REST APIs │ SDKs     │
├──────────────────────────────────────────────────────────┤
│  Identity:  Keystone (users, projects, roles, tokens)    │
├──────────────────────────────────────────────────────────┤
│  Compute:  Nova (+ KVM)          Orchestration:  Heat    │
│  Network:  Neutron (+ OVS)       Images:  Glance         │
│  Block:    Cinder (→ Ceph)       Object:  Swift / RGW    │
├──────────────────────────────────────────────────────────┤
│  Shared plumbing:  RabbitMQ (queue) │ MariaDB (DBs)      │
├──────────────────────────────────────────────────────────┤
│  Hardware:  compute nodes + storage nodes + network      │
└──────────────────────────────────────────────────────────┘
```

Three things carry forward:

1. **Freedom is the product, and so is the work.** No license fees, no vendor lock-in, every component swappable — but someone must assemble, integrate, secure, upgrade, and support a dozen-plus services. That "someone" is your team or a distribution vendor (§5).
2. **It is an IaaS platform, not an appliance.** OpenStack gives you cloud APIs, multi-tenancy, self-service, and quota management — the *public-cloud operating model* — which is precisely what classic virtualization (vSphere) and appliance-style HCI (Nutanix) deliver only partially. This is OpenStack's durable differentiator: **it is the only one of the two that is a real cloud platform by default.**
3. **Scale and gravity are real, if quiet.** 40M+ cores in production (2022 survey), 55M+ (2025 survey), dominant in telco NFV and research — OpenStack is the world's largest open-source cloud, even as the enterprise conversation moved on (the honest trends treatment is in §6.2).

### 3.5 Deploying OpenStack — The Walkthrough

| Step | What happens | Skills required |
|---|---|---|
| 1 — Architecture design | Decide which services to deploy (Nova, Neutron, Cinder, Glance, Keystone, Heat, Horizon, …), network topology, storage backend (Ceph vs drivers), HA model, sizing | Cloud architect — the scarce, expensive skill |
| 2 — Hardware | Compute nodes *plus* separate storage nodes (Ceph OSDs) *plus* network fabric — different hardware profiles per role | DC ops + storage engineering |
| 3 — Base services | Install OS, MariaDB, RabbitMQ, then Keystone first (identity is the front door everything depends on) | Linux engineering (RHEL/Ubuntu admin level) |
| 4 — Control plane | Deploy Glance, Nova control services, Neutron server/agents, Cinder — each configured against the DB and message queue | OpenStack administrator; distro tooling (Juju/director/k0rdent) automates much of this |
| 5 — Storage backend | Deploy and configure the Ceph cluster; create Cinder pools and the Glance backend; test volume attach/detach | Ceph engineer — a recognized specialty |
| 6 — Networking | Configure Neutron overlays (VXLAN/Geneve), Open vSwitch, router/gateway nodes, security groups, floating IPs | Network engineer with OpenStack networking experience |
| 7 — Verification & hardening | API smoke tests, create tenants and quotas, RBAC, TLS everywhere, back up the control-plane databases | Cloud ops team |
| 8 — Day 2 | Patch per service, run twice-yearly release upgrades, watch Ceph OSD health, maintain RabbitMQ/MariaDB | Full-time cloud ops team (3–5+ people) |
| **Typical timeline** | **2–6+ months to a production cloud with a distribution; longer DIY** | **A cloud-engineering team, or a distro vendor's support** |

### 3.6 The OpenStack Release History — A Timeline

OpenStack ships twice a year on a lettered/numbered cadence. The milestones worth remembering (release names and years are **Verified** against the project's release pages; the interpretations are Reported/directional):

| Year | Release | Why it matters |
|---|---|---|
| 2010 | **Austin** (first) | The launch: Nova + Swift from NASA and Rackspace |
| 2011 | Cactus → Diablo | Early governance forms; the project goes from 2 to ~20 services |
| 2012 | Essex | First release with Horizon, Keystone, and a real dashboard story; OpenStack Foundation established |
| 2013 | Grizzly | Neutron (then "Quantum") becomes the networking service; the platform matures |
| 2014 | Icehouse | Broad enterprise adoption wave begins; distros (Red Hat, Canonical, Mirantis) become the consumption model |
| 2015–2017 | Kilo → Ocata | The NFV pivot: SR-IOV, DPDK, and telco-grade networking features land; Kubernetes begins eating the mindshare |
| 2018–2021 | Queens → Wallaby | The "OpenStack is dead?" era — actually steady telco/research growth; OpenStack Foundation becomes OpenInfra (2021) |
| 2022–2024 | Zed → Caracal | 40M+ production cores (2022 survey); distros pivot to OpenStack-on-Kubernetes (Red Hat GA Aug 2024) |
| 2025 | **2025.2 Flamingo** | Current release; AI/HPC features (GPU, edge) and 55M+ cores claimed in the 2025 survey (Reported) |

The arc this table tells: **from cloud-native novelty (2010–2014) to telco workhorse (2015–2021) to quietly massive substrate (2022–2026)** — with the enterprise conversation having moved to Kubernetes somewhere in the middle. That arc is why §6.2's honest treatment matters.

---

## 4. Head-to-Head — The Comparison

### 4.1 The Comparison Table

| Dimension | Nutanix | OpenStack |
|---|---|---|
| **Category** | Commercial HCI platform (+ cloud services) | Open-source IaaS platform (project) |
| **License** | Proprietary, subscription (per-core + per-TiB; AHV included) | Apache 2.0 — free; costs are hardware + engineering + optional support |
| **Compute** | AHV (KVM-based) included; also runs ESXi and (via NC2) cloud bare metal | Nova driving KVM by default (also VMware, Hyper-V, Xen, Kata); hypervisor choice is yours |
| **Storage** | AOS distributed fabric: block (iSCSI), file (NFS/SMB), object (S3) from one engine | Disaggregated: Cinder (block) + Swift (object) + Manila (file), typically on Ceph; you integrate |
| **Networking** | Flow (built-in SDN, microsegmentation) | Neutron (overlays, plugins); depth depends on your build |
| **Management** | Prism Element + Prism Central — one console, health scores, wizard-driven ops | Horizon dashboard + CLI + APIs; each service managed separately; ops burden is yours |
| **Multi-tenancy / self-service** | Partial (Prism Central projects/RBAC, marketplace) | Native, full IaaS: projects, quotas, self-service API — the core of the platform |
| **Orchestration** | Calm (blueprint automation, application-level) | Heat (template stacks) + Magnum (K8s) + Mistral (workflows) |
| **Identity** | Prism Central RBAC (platform-level) | Keystone (full multi-tenant identity, tokens, service catalog) |
| **Upgrades** | One-click rolling upgrades via Prism; AOS + AHV together | Per-service upgrade projects; release cadence twice a year; rollback is painful |
| **Support** | One vendor, one contract, SLAs | Community (free) or distribution vendor (Red Hat/Canonical/Mirantis); you are the integrator either way |
| **Time to first workload** | Days (installer → cluster → VMs) | Weeks to months (design, integrate, deploy, tune 10+ services) |
| **Skills required** | Small team, vendor-trained; wizard-driven | Cloud-engineering team (Linux, networking, distributed storage); rare and expensive |
| **Cloud APIs** | Prism APIs + vSphere-style management; no native IaaS API | Full IaaS API surface — the closest thing to AWS-in-your-DC |
| **Hybrid cloud** | NC2: same stack on AWS/Azure/GCP bare metal | No vendor story; DIY federation (rarely done) or distro-specific integrations |
| **Ecosystem** | One vendor portfolio + OEM hardware partners | OpenInfra community + distros + 55M+ production cores (Reported) |

### 4.2 The Architecture Comparison — HCI vs Disaggregated (Verified)

The architectural difference is the deepest one, and it is a genuine **philosophy split**:

**Nutanix = convergence.** In a hyperconverged cluster, compute and storage are fused on every node. Each node runs a small Controller VM that hosts the AOS data path; guest VMs route I/O through the CVM, which can reach any node's disk (Verified — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §2.6). The cluster behaves as one big distributed disk: replicas (RF2/RF3) or erasure coding (EC-X) spread across nodes, self-healing re-replication after failures, one-click expansion. **Consequences:** (a) scaling is node-shaped — add a node and compute *and* capacity grow together (which is a constraint when you want only storage or only compute); (b) performance is predictable because the data path is engineered by one vendor; (c) there is no SAN/NAS/object silo to manage — protocol services are front-ends on the same fabric.

**OpenStack = disaggregation.** OpenStack is deliberately *not* converged: compute (Nova) and storage (Cinder/Swift) are **separate pools** that you size, scale, and operate independently, connected by the network (Neutron). Storage typically lives on dedicated servers running **Ceph** (the de-facto standard backing store — cross-ref [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md)); compute pools are hypervisor farms; network functions run as separate services. **Consequences:** (a) you can scale storage without touching compute (a real advantage for capacity-heavy workloads); (b) every layer is independently swappable — Ceph can be replaced by commercial arrays, KVM by VMware, Neutron's OVS by hardware controllers; (c) the integration burden is yours — the platform's composability is also its complexity.

**Where the workloads actually run** — the practical table:

| Workload type | Nutanix reality | OpenStack reality |
|---|---|---|
| General-purpose VMs (Windows/Linux) | Native on AHV, one console | Native on Nova/KVM, Horizon/CLI |
| Databases | NDB (DBaaS) on AOS; or VMs on AHV | VMs on Nova with Cinder volumes; DBaaS is DIY |
| File services | Nutanix Files (NFS/SMB) built in | Manila (if deployed) or a separate NAS/CephFS |
| Object storage | Nutanix Objects (S3) built in | Swift (its own API) or Ceph RGW (true S3) |
| Containers/Kubernetes | NKE (Nutanix Kubernetes Engine) on AHV | Magnum, or K8s directly on Nova VMs/bare metal (Ironic) |
| Bare metal | Not a native model (Volumes iSCSI for external hosts) | Ironic — first-class bare-metal provisioning |
| Telco/NFV | Possible but not the pitch | The home turf: SR-IOV/DPDK networking, NFVI reference architectures |
| AI/GPU | GPT-in-a-Box (2023), NVIDIA integration | Nova with PCI passthrough + Cyborg (GPU management); DIY |

**The honest architectural verdict (Flagged as analyst/experience view):** Nutanix's convergence wins on operational simplicity and predictable performance for the *mainstream enterprise VM estate*; OpenStack's disaggregation wins on composability, per-layer scale, and raw flexibility for *cloud builders*. Neither is wrong — they answer different questions. Nutanix asks "how do I run my VMs and storage with the least fuss?" OpenStack asks "how do I operate a cloud at scale with full control?"

### 4.3 The Operations Comparison — Simplicity vs Complexity

**Nutanix operations: the "one of everything" model.** One installer, one console (Prism), one upgrade path (AOS + AHV together, rolling, wizard-driven), one support contract, one health dashboard. Day-2 tasks — capacity planning, patching, adding nodes, DR replication, backup (Mine/HYCU) — are productized. A small platform team (2–4 people for a mid-size estate) trained by the vendor can run it. The operational contract is essentially: **Nutanix owns the integration; you own the configuration.** The cost: limited freedom to change components, and renewal leverage is a real negotiation topic (Flagged — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §11.5).

**OpenStack operations: the "many of everything" model.** You operate a fleet of services, each with its own lifecycle: Nova computes, Cinder volumes, Neutron agents, Keystone, Glance, Heat, message queues (RabbitMQ), databases (MariaDB), and the Ceph cluster underneath. Day-2 reality includes: per-service upgrades (with a twice-yearly release cadence), RabbitMQ/MariaDB maintenance, Ceph OSD health, network agent debugging, and the occasional "it worked in dev" integration surprise. This is why production OpenStack is almost always operated either by a large cloud-engineering team or by a **distribution vendor's tooling and support** (§5). The operational contract: **you (or your distro vendor) own the integration; nothing is productized until you productize it.**

**The operations table:**

| Operations aspect | Nutanix | OpenStack |
|---|---|---|
| Install | Hours–days; installer + node bootstrapping | Weeks–months; design + deploy 10+ services (or use a distro's installer) |
| Day-2 management | One console, health scores, wizards | Horizon + CLI + per-service config; runbooks required |
| Patching/upgrades | Rolling, one-click, vendor-tested | Per-service, twice-yearly releases; upgrade projects with rollback planning |
| Monitoring | Built into Prism (health, alerts, capacity) | Ceilometer/Gnocchi + external stack (Prometheus/Grafana) — you build it |
| Backup/DR | Built-in (snapshots, async replication, Mine) | cinder-backup + Swift/Ceph + Heat stacks; you assemble |
| Self-service for app teams | Prism Central projects (with approvals) | Native: Horizon + API + quotas — the strongest self-service story |
| Staffing | Small vendor-trained team | Cloud-engineering team (or distro vendor support) |
| Failure modes | Vendor defect = vendor fixes it | Your integration defect = you fix it |

### 4.4 The Economics — Licensing vs Open-Source + Support

**Nutanix — the subscription.** Licensing is per **CPU core + flash capacity (TiB)** under the Nutanix Cloud Infrastructure (NCI) base SKU, with add-ons for storage (NUS), databases (NDB per engine tier), security (Flow), and backup (Mine) — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §9 for the full model and a worked sizing. Directional arithmetic from that guide: an 8-node cluster (2×16-core CPUs, ~7 TiB flash/node) ≈ 256 cores + ~60 TiB per cluster. AHV being included is the structural cost win versus vSphere-era stacks. Renewals are 3-year terms, negotiated at ELA level. **Flagged:** actual pricing/SKUs come from Nutanix or your partner; discounts vary widely.

**OpenStack — the free license, the expensive build.** The software is Apache-2.0 free. The costs are: (1) hardware (compute + a Ceph storage tier + network — often *more* hardware than an HCI cluster for equivalent workloads, because storage and compute are separate pools); (2) engineering — the dominant line: design, deployment, integration, automation, and ongoing operations by a skilled cloud team; (3) optional vendor support from a distribution (Red Hat, Canonical, Mirantis — §5), which converts the unpredictable engineering cost into a subscription but still leaves integration work on your side of the line. The break-even intuition (Flagged as planning-grade): **OpenStack gets cheaper than a commercial platform only at meaningful scale or with strong in-house engineering** — the open-source license saves license fees, but the build labor is real money (cross-ref [finops_guide.md](finops_guide.md) for the cost-governance lens).

**The economics table:**

| Cost line | Nutanix | OpenStack |
|---|---|---|
| License | Subscription: cores + TiB (+ service add-ons) | $0 (Apache 2.0) |
| Hardware | Standard x86 servers (any OEM); converged sizing | Servers + separate storage tier (usually Ceph) + network; often more total hardware |
| Integration | Included in the product | Your engineering (or distro vendor's tooling) |
| Operations staffing | Small team, low skill ceiling | Cloud-engineering team, high skill ceiling |
| Support | One vendor contract with SLAs | Community (free) or distro subscription (paid) |
| Predictability | High — subscription is a known line item | Variable — engineering costs dominate and fluctuate |
| Lock-in risk | Vendor lock-in (but reversible: VMware-era precedent, NC2) | No software lock-in; but your *build* is your lock-in (skills, custom automation) |
| TCO intuition | Predictable per-node cost; scales linearly | High fixed engineering cost; amortizes at scale |

#### The TCO Scenario Sketch (36-month, planning-grade — Flagged)

Directional arithmetic for the worked example's estate (1,500 VMs, two DCs, ~20%/yr growth toward 3,500 VMs) so the buy-vs-build economics are concrete. All numbers are illustrative planning figures, not quotes — model your own (cross-ref [finops_guide.md](finops_guide.md) for the cost-governance lens):

| Cost line (3-year) | Nutanix (buy) | OpenStack build (with a distro support subscription) |
|---|---|---|
| Software license / subscription | ~$1.2–1.8M (NCI cores+TiB, two DCs, ~3-year term — Flagged) | $0 license + ~$0.3–0.5M distro support subscription |
| Hardware (servers + storage) | ~$0.8–1.2M (converged nodes) | ~$1.0–1.5M (compute + separate Ceph tier + network) |
| Integration & deployment | Included in the product | ~$0.2–0.4M (partner SI engagement) |
| Operations staffing (3 years) | ~$0.5–0.8M (2–3 platform admins) | ~$1.2–2.0M (4–6 cloud engineers, including a Ceph specialist) |
| Training | ~$50–100k | ~$100–200k |
| **Total (3-year)** | **~$2.6–3.9M** | **~$2.6–4.6M** |
| **TCO per VM-month (3,500-VM plateau)** | **~$20–30** | **~$20–37** |

**The break-even intuition:** at this estate size the two land surprisingly close — the open-source license savings are roughly consumed by extra hardware and labor. OpenStack's advantage grows only when (a) the estate is much larger (10k+ VMs) so fixed engineering amortizes, (b) the organization already employs the cloud engineers, or (c) the commercial platform's licensing delta is unusually large. The real differentiators are therefore operational simplicity (R1 in §8) and time-to-value, not the license fee itself.

### 4.5 The Feature-Parity Matrix

| Feature | Nutanix | OpenStack | Notes |
|---|---|---|---|
| VM lifecycle (create/migrate/resize) | ✅ AHV, one console | ✅ Nova, full API | Parity; OpenStack's API is richer programmatically |
| Live migration | ✅ | ✅ (KVM) | Parity |
| High availability | ✅ Node-failure restart, self-healing storage | ✅ Nova evacuate + Ceph replication | Parity; HA is a design exercise on OpenStack |
| Snapshots & clones | ✅ AOS, native | ✅ Cinder/Glance snapshots | Parity; Nutanix is more turnkey |
| Block storage | ✅ Volumes (iSCSI) + native AHV | ✅ Cinder (+ Ceph) | Parity |
| File storage | ✅ Files (NFS/SMB) | ⚠️ Manila (often not deployed; DIY) | Nutanix stronger out of the box |
| Object storage | ✅ Objects (S3 API) | ⚠️ Swift (own API) or Ceph RGW for true S3 | Nutanix simpler; Ceph RGW is the open answer |
| Multi-tenancy & quotas | ⚠️ Prism Central projects (governance, not full IaaS) | ✅ Native projects/quotas | OpenStack's strongest feature vs Nutanix |
| Self-service API | ⚠️ Prism APIs, partial IaaS surface | ✅ Full IaaS API | The core OpenStack differentiator |
| Identity / RBAC | ✅ Prism Central RBAC | ✅ Keystone (tokens, federation) | Keystone is deeper |
| Network overlays | ✅ Flow (built-in) | ✅ Neutron (OVS/VXLAN) | Parity; Neutron is more extensible |
| Microsegmentation | ✅ Flow Security | ⚠️ Neutron security groups (+ FWaaS; less turnkey) | Nutanix simpler for east-west policy |
| Load balancing | ⚠️ Via Flow / third party | ✅ Octavia (LBaaS) | OpenStack has a native service |
| Orchestration | ✅ Calm (blueprints) | ✅ Heat (templates) + Mistral | Different philosophies, both capable |
| Kubernetes | ✅ NKE on AHV | ✅ Magnum or K8s on VMs/Ironic | Both work; K8s-first shops may skip both |
| Bare metal | ❌ Not native | ✅ Ironic | OpenStack unique |
| Telco networking (SR-IOV/DPDK) | ⚠️ Possible | ✅ Native NFV features | OpenStack's home turf |
| Backup / DR | ✅ Built-in + Mine | ⚠️ cinder-backup + Swift/Ceph + Heat; DIY | Nutanix more turnkey |
| Monitoring | ✅ Prism health, built-in | ⚠️ Ceilometer + external stack (Prometheus/Grafana) | Nutanix out of the box |
| Upgrades | ✅ One-click rolling | ⚠️ Per-service, twice-yearly | The biggest ops difference |
| Hybrid / multi-cloud | ✅ NC2 (AWS/Azure/GCP) | ❌ No productized story | Nutanix unique |

### 4.6 The Upgrade and Lifecycle Comparison

| Lifecycle aspect | Nutanix | OpenStack |
|---|---|---|
| Release cadence | AOS major/minor on the vendor's schedule; upgrades bundle AOS + AHV together | Twice-yearly named releases (e.g., 2025.2 Flamingo); distributions choose which lines to support |
| Upgrade mechanics | Rolling node-by-node, orchestrated by Prism, one click | Per-service upgrade projects; orchestrated by distro tooling (Juju/director/k0rdent) or manual runbooks |
| Downtime | Minimal by design (rolling) | Depends on HA design; control-plane upgrades need rehearsal |
| Rollback | Supported (pre-upgrade snapshots) | Painful — the classic OpenStack pain point |
| Security patching | Vendor patches, pushed through Prism | You or your distro; CVE triage across many services |
| Hypervisor upgrades | Bundled with AOS | Part of compute-node maintenance (KVM/qemu) |
| Storage upgrades | Bundled (AOS) | Separate project: Ceph upgrades are their own exercise (cross-ref [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md)) |
| Long-term support | Vendor-defined (typically N-1/N-2) | Distro-defined (e.g., Red Hat's multi-year supported lines) |

### 4.7 The API and Cloud-Native Story

- **OpenStack's API surface *is* the product.** Keystone-authenticated REST APIs for every service, a published API contract, SDKs in every major language, and first-class Terraform/Ansible providers. This is what makes OpenStack "AWS-like" for developers — and it is the reason cloud-builder teams love it despite the ops burden.
- **Nutanix's API surface is an enabler, not the product.** Prism exposes REST APIs (v3) with Terraform/Ansible providers, but the primary interface is the console; the APIs serve automation *of the platform* rather than a developer-facing IaaS contract. For a dev team that wants to `openstack server create` or `terraform apply` a tenanted project, Prism Central projects + Calm are a governed middle ground — not a true IaaS API.
- **Cloud-native implication:** OpenStack integrates natively with Kubernetes (Magnum, standard CCM/CSI providers, Cluster API); Nutanix integrates via NKE and CSI drivers (cross-ref [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) for the K8s-platform layer). Both are workable; OpenStack's is more native and automated, Nutanix's is more integrated with its storage and network services.

---

## 5. The Distributions — OpenStack's Vendor Layer

### 5.1 Why Distributions Exist

Because upstream OpenStack is a project, not a product, a **distribution** is a vendor's packaged, supported, enterprise-ready version: tested combinations of services, an installer and lifecycle tooling, security patches, and a support contract. This is the closest OpenStack gets to "buying" — and it is how most enterprises consume OpenStack at all. The three established commercial distributions are **Red Hat, Canonical, and Mirantis** (all **Verified** as the commonly cited distro trio — Canonical's own distro-comparison pages, vendor sites; the shape of each is verified below, and details flagged where they move).

### 5.2 Red Hat OpenStack Platform (and Its Successor) — Verified

**Red Hat OpenStack Platform (RHOSP)** was the enterprise heavyweight: RHEL-based, sold per-socket subscriptions, with Red Hat's support and the Ansible-based director installer. Its customer base skews telco and large enterprise.

**The important 2024 development (Verified — Red Hat blog, TechCrunch):** Red Hat is transitioning the platform to **Red Hat OpenStack Services on OpenShift**, generally available from 26 August 2024. The new architecture runs the OpenStack **control plane as containers on OpenShift** (Red Hat CoreOS underneath), with the data plane on RHEL — i.e., Red Hat is converging its OpenStack story onto its Kubernetes platform (the same family of platforms covered in [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) and [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md)). RHOSP 17.x is the last classic line; new deployments are expected on the OpenShift-based successor (Reported — migration economics by destination, per Red Hat's own guidance; check current support windows before planning).

**What this signals for the whole comparison:** the leading commercial OpenStack distribution is now *Kubernetes-shaped* — a nod to where the industry went, and a fact that narrows the "OpenStack vs the world" story considerably.

### 5.3 Canonical Charmed OpenStack — Verified

**Charmed OpenStack** (Canonical) is the Ubuntu-based distribution, built on **Juju** (model-driven operations) and **charms** (operator packages) — the same tooling family as Charmed Kubernetes ([charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md)). Every OpenStack service is deployed and operated by a charm, so Day-2 operations (scale, upgrade, replace) are declarative. Canonical positions it for telcos, financial institutions, and government (Verified — Canonical materials), with **Ceph integrated** as the storage backbone, and an all-open-source, no-per-core-fee model — Canonical's subscription covers support and the tooling, not the software itself. Its appeal: **open-source purity with operator-based automation**; its cost: Juju/charms is its own skill set, and Ubuntu-centric.

### 5.4 Mirantis — Verified

**Mirantis** is the OpenStack services pioneer (founded 2011, the "OpenStack company" that ran large managed OpenStack deployments) that pivoted into containers (it acquired the Docker Enterprise business in 2019 — Reported, widely covered) and now anchors on Kubernetes. Its current offering, **Mirantis OpenStack for Kubernetes** (GA April 2025, Verified — Mirantis announcement), deploys containerized OpenStack control planes, Ceph storage, and compute on **Mirantis Kubernetes Engine** (formerly Docker Enterprise/UCP), managed through Mirantis' k0rdent/Container Cloud tooling. Mirantis also sells managed services. **What this signals:** the third distribution is also Kubernetes-based — the industry's OpenStack answer has converged on "OpenStack on K8s."

### 5.5 The Distribution Table

| Distribution | Vendor / base | Architecture | Licensing model | Best fit | Status (Aug 2026) |
|---|---|---|---|---|---|
| **Red Hat OpenStack Platform** → **Red Hat OpenStack Services on OpenShift** | Red Hat (IBM) / RHEL + CoreOS | Control plane containerized on OpenShift (GA 26 Aug 2024) | Per-socket subscription; RHOSP 17.x classic line | Telco/enterprise needing Red Hat support and RHEL estate | Transition in progress — classic RHOSP EOL-ing (Reported; verify current windows) |
| **Canonical Charmed OpenStack** | Canonical / Ubuntu LTS | Juju + charms, Ceph integrated | Subscription for support/tooling; no per-core license | Ubuntu-standardized shops, open-source-first, telco/finance/gov | Active; the "K8s-adjacent" Juju story also powers Charmed Kubernetes |
| **Mirantis OpenStack for Kubernetes** | Mirantis / Ubuntu + Mirantis Kubernetes Engine | OpenStack control plane + Ceph on MKE (K8s) | Commercial subscription; managed services available | Legacy Mirantis/Docker Enterprise estates; K8s-centric cloud builders | GA April 2025; Mirantis acquired by IREN (Reported — 2025) |
| **Upstream / DIY** | OpenInfra community | Whatever you assemble | Free | Cloud-engineering teams that want zero vendor | Always an option; the default for hyperscale-style operators |

**The takeaway:** all three major distributions now sit on or beside Kubernetes. The "pure OpenStack" era is over even among its vendors — which matters for any 2026 architecture decision (see §6.2 and §7).

---

## 6. The Ecosystem Comparison

### 6.1 The Ecosystem Table

| Ecosystem aspect | Nutanix | OpenStack |
|---|---|---|
| **Community** | Single vendor (Nutanix, Inc.); user groups and .NEXT conference; partner/OEM channel | OpenInfra Foundation: 110,000+ community members claimed; semi-annual user surveys; OpenStack Summits (Verified — openinfra.org; counts are self-reported) |
| **Distributions** | N/A — the product *is* the distribution | Red Hat, Canonical, Mirantis (§5) + upstream DIY |
| **Hardware ecosystem** | Any standard x86 server; OEM partners (Dell, HPE, Lenovo, Cisco, Fujitsu); NC2 on AWS/Azure/GCP | Any hardware that runs Linux; telco reference architectures; certified hardware catalogs per distro |
| **Storage ecosystem** | One engine (AOS) with protocol front-ends; Ceph is a competitor, not a component | Ceph is the de-facto backbone (Cinder/Glance/Swift); commercial arrays via drivers |
| **Container ecosystem** | NKE (Kubernetes on AHV); integration with OpenShift via CSI (cross-ref the K8s guides) | Native K8s paths: Magnum, or K8s on Nova/Ironic; distros now run OpenStack *on* K8s |
| **Skills market** | HCI admin skills; vendor certifications; moderate scarcity | Cloud-engineering skills (Linux, Ceph, Neutron, RabbitMQ); scarcer and more expensive |
| **Analyst coverage** | Gartner MQ Leader (HCI), Forrester, IDC — standard vendor coverage | OpenInfra surveys + analyst reports (Forrester "State of OpenStack"); less mainstream attention |

### 6.2 The Market and Trends — Being Honest About "OpenStack Is Dying"

No honest Nutanix-vs-OpenStack guide can skip the adoption question, because the narratives contradict each other and both contain truth:

**The bullish data (Reported — self-reported survey numbers, treat as directional):**
- OpenStack reported **40M+ cores in production in 2022** (a 60% increase over 2021 and 166% over 2020 — The Register / OpenInfra press), and the 2025 user survey's early findings claim **55M+ cores** and growth in AI/HPC adoption (Flamingo release coverage).
- It runs in **300+ public cloud datacenters** and remains the de-facto standard for **telco NFV** (telecoms lead industry share at ~28% per Mordor Intelligence's market sizing — Flagged: third-party market-research estimates vary widely) and much of **research/HPC** (CERN, etc.).
- Canonical's own defense ("OpenStack is dead? The numbers speak for themselves," 2022): it is no longer at the forefront of technology, but it is *everywhere as a foundation* — hidden in data centers, powering K8s and AI stacks underneath.

**The bearish data (Reported):**
- The New Stack (Feb 2025): "OpenStack use plummets" — multi-cloud surveys show OpenStack deployment stagnating while managed/hyperscaler options grow; discounts dry up as vendors focus elsewhere.
- The enterprise conversation moved to **Kubernetes** a decade ago; OpenStack is rarely the headline platform in new greenfield designs.
- The three major distributions all pivoted their OpenStack stories **onto Kubernetes** (§5) — vendors themselves treat classic OpenStack as a legacy-and-telco business, not a growth platform.

**The honest synthesis (Flagged as analyst view):** OpenStack is **not dying — it is a mature, quietly massive infrastructure substrate** with a durable telco/research/private-cloud footprint, but **it is not growing in the mainstream enterprise mindshare** and its vendors have repositioned around Kubernetes. For a bank or a mainstream enterprise in 2026, "should we build on OpenStack?" is a very different question than it was in 2015 — it is now a *specialist's* choice (telco NFV, research, cloud-provider-scale private clouds, regulatory environments with strong in-house engineering), not the default enterprise answer. The default enterprise answer, when it is not the public cloud, is an integrated commercial platform — Nutanix or its competitors. That asymmetry is the single most important market fact in this comparison.

### 6.3 Where Each Platform Actually Runs — The Adoption Reality

| Sector | Nutanix | OpenStack |
|---|---|---|
| Banking / finance | Strong — regulated modernization, VMware replacement, predictable TCO (§8) | Present but niche; usually via a distro with strong in-house engineering |
| Telco | Present (edge, private-5G-adjacent workloads) | **Dominant** — NFV/NFVI reference architectures; ~28% of OpenStack deployments (Flagged — Mordor Intelligence estimate) |
| Government / sovereign | Strong — sovereign-AI builds (HTX NGINE pattern, [htx_ngine_guide.md](htx_ngine_guide.md)) | Strong in some regions — national clouds and public-sector IaaS |
| Research / HPC | Moderate — GPU clusters, GPT-in-a-Box | **Strong** — CERN and similar; the original Nova heritage |
| Healthcare / retail / mid-market | Strong — mainstream enterprise HCI | Rare — complexity doesn't pay at this scale |
| Cloud providers / hosting | Via NC2 (DR/burst) | **Strong** — 300+ public cloud datacenters (Reported) and hyperscale-style builders |
| Edge / ROBO branches | Strong (small clusters; Scale Computing competes) | Rare — too heavy for the edge |

The pattern is unmistakable: **OpenStack's gravity sits where scale and engineering capability already exist (telco, research, cloud builders); Nutanix's gravity sits where a mainstream enterprise wants a private cloud without building one.** Both adoption curves are real — they just live in different sectors.

---

## 7. Selection Guidance — Which for Which Need

### 7.1 The Which-for-Which-Need Table

| Your need | Choose Nutanix | Choose OpenStack |
|---|---|---|
| **"Modernize my VMware estate without a cloud-engineering team"** | ✅ — AHV included, wizard-driven migration, vendor support | ❌ — you would be building a cloud to replace a hypervisor |
| **"Run a real multi-tenant cloud with self-service APIs, quotas, and projects"** | ⚠️ — Prism projects are governance, not full IaaS APIs | ✅ — that is literally what OpenStack is |
| **"Telco/NFV workloads — SR-IOV, DPDK, VNF reference architectures"** | ⚠️ — possible, not the pitch | ✅ — the home turf; NFVI distro references exist |
| **"Research/HPC or hyperscale-style private cloud at 10k+ cores"** | ⚠️ — works, but you pay subscription on everything | ✅ — free license, composable at scale; the proven model |
| **"Regulated bank: predictable TCO, audit-friendly, small platform team"** | ✅ — one vendor, one contract, one console (see §8) | ⚠️ — viable via a distro, but engineering cost and skills are the risk |
| **"Zero-vendor-lock-in, full control of every layer"** | ❌ — proprietary by definition | ✅ — Apache 2.0, everything swappable |
| **"Kubernetes-first platform; VMs are a side quest"** | ⚠️ — NKE/CSI integrations are solid | ⚠️ — OpenStack-on-K8s is the new distro model; consider K8s directly instead |
| **"Cheapest possible path to private cloud"** | ⚠️ — predictable but never free | ⚠️ — free license, but the build labor usually isn't cheaper at small scale |
| **"Hybrid cloud: same platform on-prem and in AWS/Azure/GCP"** | ✅ — NC2 is a productized story | ❌ — no equivalent; DIY federation is rare and hard |
| **"I have 3 months to show a private-cloud PoC"** | ✅ — days to first VM | ❌ — a real OpenStack PoC is a project itself (or buy a distro appliance) |

### 7.2 The Selection Criteria (Verified Checklist)

A scored checklist for the architect — the criteria that actually decide these deals (directional weights; adjust to your context). Cross-ref the blank scoring template pattern in [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md).

| # | Criterion | Why it matters | Favours |
|---|---|---|---|
| 1 | **In-house cloud-engineering capability** | OpenStack's total cost is dominated by skills you either have or must hire/buy | Have a cloud team → OpenStack; else Nutanix |
| 2 | **Time-to-value / deployment window** | A bank's board cares about months, not quarters | Nutanix (days) vs OpenStack (weeks–months) |
| 3 | **True multi-tenancy & self-service requirement** | Do app teams need real cloud APIs, quotas, and isolated projects? | Real IaaS requirement → OpenStack; else Nutanix suffices |
| 4 | **Scale trajectory** | OpenStack's economics amortize at scale; Nutanix scales linearly with subscription | Large (10k+ cores) & growing → OpenStack; steady mid-size → Nutanix |
| 5 | **Regulatory/compliance posture** | Auditors want evidence, predictability, and support accountability | One vendor with SLAs → Nutanix; in-house-controlled → OpenStack |
| 6 | **Existing skills & estate** | VMware/RHEL/Ubuntu heritage, container strategy, storage standards (Ceph?) | Alignment decides real costs |
| 7 | **Hybrid-cloud strategy** | Is "same platform in the public cloud" a requirement? | NC2 → Nutanix; DIY federation → almost never OpenStack |
| 8 | **Risk tolerance for vendor lock-in vs DIY risk** | Proprietary lock-in vs the risk that *your* build becomes your lock-in | Depends on organizational philosophy |
| 9 | **Support & SLA expectations** | Who answers at 2 a.m., and what is the contractual backstop? | Vendor (Nutanix or a distro) — never upstream-alone for regulated workloads |
| 10 | **Total cost of ownership (3-year view)** | License + hardware + labor + ops; model it with your own numbers | Compute it — see §4.4 and the worked example |

**The one-line rule:** Nutanix wins where **predictability, integration, and a small team** matter more than freedom; OpenStack wins where **multi-tenant IaaS at scale, composability, and zero license cost** matter more than simplicity — and in 2026, OpenStack's win conditions are increasingly specialist (telco, research, cloud-builder) rather than mainstream-enterprise.

### 7.3 The Blank Weighted-Scoring Template

Copy this table, set your own weights (sum to 100), score both platforms 1–5 per criterion, and multiply by the weights — §8 is a fully worked instance of this template:

| # | Criterion | Weight | Nutanix (1–5) | OpenStack (1–5) | Notes / evidence to collect |
|---|---|---|---|---|---|
| 1 | In-house cloud-engineering capability | | | | Headcount you actually have vs need |
| 2 | Time-to-value (deployment window) | | | | Months to first production workload |
| 3 | Multi-tenancy & self-service requirement | | | | Real IaaS APIs vs governed portals |
| 4 | Scale trajectory (3–5 yr) | | | | VM/core count at the planning horizon |
| 5 | Regulatory / compliance posture | | | | Audit evidence, SLAs, accountability |
| 6 | Existing skills & estate alignment | | | | VMware/RHEL/Ubuntu/Ceph heritage |
| 7 | Hybrid-cloud strategy | | | | Same-platform cloud requirement (NC2 vs none) |
| 8 | Vendor lock-in vs DIY risk tolerance | | | | Organizational philosophy |
| 9 | Support & SLA expectations | | | | Who answers at 2 a.m. |
| 10 | 3-year TCO (license + hardware + labor) | | | | Model with your own market rates |
| | **Total** | **100** | | | Weighted score decides |

**Scoring discipline:** the worked example in §8 shows the pattern — score each criterion against *your* evidence, not the vendors' slideware, and let the weights reflect your constraints (a bank's binding constraint is rarely the license fee). Re-run the template at least annually; the private-cloud pendulum swings with licensing changes, skills markets, and mandates.

---

## 8. Worked Example — A Private-Cloud Platform Selection

### 8.1 The Scenario (the familiar context)

**"Merlion Bank"** — the same mid-size Singapore bank from [nutanix_products_guide.md](nutanix_products_guide.md) §11 (think ~1,500–3,000 staff, retail + private banking + a treasury desk; the profile mirrors the regulated-Asian-bank context in the `banking/` guides). Two years ago it chose Nutanix HCI for its data-center modernization. Now the group CIO has asked the architecture team to evaluate the alternative the board keeps hearing about at industry events: **"why not build on open-source OpenStack instead of paying Nutanix?"** This worked example runs that evaluation honestly — and, unusually, it is a **build-vs-buy re-derivation**, not a greenfield decision.

**The bank's data-center reality in 2026:**

- **Current estate:** two data centers (DC-A, DC-B), each with an 8-node Nutanix cluster (AHV), Nutanix Files for file services, NDB for SQL Server/Oracle/PostgreSQL, Flow for microsegmentation; a measured NC2-on-AWS footprint for DR testing and month-end burst. MAS-grade resilience expectations, data-residency rules (customer data stays in Singapore), and VMware/Broadcom licensing pain are in the rear-view mirror.
- **The new pressure:** a group-level "open-source-first" directive (cost and sovereignty optics), and a growing **developer-services demand**: application teams want true self-service, per-project isolation, quotas, and cloud-style APIs — the things Prism Central's projects deliver only partially (§4.2).

### 8.2 The Requirements (Weighted)

The architecture team scores the decision criteria (weights sum to 100):

| # | Requirement | Weight | Notes |
|---|---|---|---|
| R1 | **Operational simplicity with a small team** (platform team of 4, no cloud-engineering specialists) | 25 | The binding constraint — hiring a Ceph/Neutron team in Singapore is slow and expensive |
| R2 | **Time-to-value: new platform live within 12 months, workloads migrated within 24** | 15 | Board-level commitment; delay is failure |
| R3 | **True multi-tenancy & self-service for 12 application teams** (APIs, quotas, projects, RBAC) | 15 | The gap the CIO wants closed; Prism Central projects are partial |
| R4 | **Predictable 3-year TCO, MAS-audit-friendly** (support SLAs, evidence, one accountable vendor) | 15 | Finance + audit both have vetoes |
| R5 | **Data residency & sovereignty** (customer data never leaves Singapore) | 10 | Non-negotiable; rules out foreign public-cloud residency for core data |
| R6 | **Scale trajectory** (estate grows ~20%/yr; from ~1,500 VMs today to ~3,000–4,000 in 3 years) | 10 | Enough to make OpenStack's amortization argument worth modeling |
| R7 | **Open-source-first optics** (board narrative, not a hard mandate) | 10 | Satisfiable by *any* credible open-source component story |
| R8 | **Hybrid-cloud reversibility** (DR/burst in the cloud without re-platforming) | — | Carry-over preference from the Nutanix decision; weighted within R1/R4 |

### 8.3 The Comparison Application

**Option A — Stay on Nutanix (buy), extend it.** Keep the AOS/AHV/Prism core; close the self-service gap with Prism Central's projects + marketplace, Calm blueprints for app-team self-service, and the Prism APIs; add GPT-in-a-Box later (cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §11 for the full design).

| Requirement | Score (1–5) | Rationale |
|---|---|---|
| R1 Simplicity (w=25) | 5 | Already running; team trained; one console, one upgrade path |
| R2 Time-to-value (w=15) | 5 | No new platform; enhancements ship in months |
| R3 Self-service (w=15) | 3 | Projects/Calm cover most app-team needs but are not full IaaS APIs; power users will still want more |
| R4 TCO & audit (w=15) | 4 | Predictable subscription; one accountable vendor; audit trail in Prism |
| R5 Residency (w=10) | 5 | Fully on-prem; NC2 DR is burst/test only |
| R6 Scale (w=10) | 4 | Linear subscription growth; node additions are easy |
| R7 Open-source optics (w=10) | 3 | AHV is KVM (open source); the platform is proprietary — partial credit |
| **Weighted total** | **4.25 / 5** | |

**Option B — Build on OpenStack (build).** Engage a local partner + a distribution (Canonical Charmed OpenStack was the team's leading candidate: Ubuntu-aligned, Ceph integrated, Juju-operated, no per-core fee) and build a true multi-tenant cloud: Nova/KVM, Cinder+Ceph, Neutron, Keystone, Horizon, Heat, with Horizon/API self-service for the 12 teams.

| Requirement | Score (1–5) | Rationale |
|---|---|---|
| R1 Simplicity (w=25) | 2 | A 12–18 month build; the 4-person team cannot operate it without hiring 3–5 cloud engineers (R1 is the binding constraint) |
| R2 Time-to-value (w=15) | 2 | Realistic timeline is 18–24 months to production + migration — misses the 12-month gate |
| R3 Self-service (w=15) | 5 | This is the one requirement OpenStack nails natively — full IaaS APIs, quotas, projects |
| R4 TCO & audit (w=15) | 3 | No license fees, but build + operations labor is real and harder to forecast; audit evidence is DIY |
| R5 Residency (w=10) | 5 | Fully on-prem |
| R6 Scale (w=10) | 4 | Ceph + Nova scale well; at 3,000+ VMs the model starts to pay off — but the bank isn't there yet |
| R7 Open-source optics (w=10) | 5 | The whole stack is Apache 2.0 |
| **Weighted total** | **3.25 / 5** | |

### 8.4 The Choice — Verified

**The bank stays on Nutanix (Option A), with two extensions:** (1) a **12-month program to productize Prism Central self-service** — projects, RBAC, Calm blueprints, and API-driven provisioning for the 12 app teams, closing most of the R3 gap without a platform change; (2) a **re-evaluation trigger**: if the estate crosses ~5,000 VMs or a genuine multi-tenant IaaS requirement emerges (e.g., an internal developer-platform mandate), re-run the OpenStack (or OpenShift-based, or public-cloud) analysis with a dedicated cloud-engineering budget — the numbers change when R1 is no longer the binding constraint.

**Why the choice is defensible (the verification):**

1. **The binding constraint decides.** R1 (a small team, no cloud-engineering specialists) is worth 25 points and is exactly where OpenStack scores worst (2/5). Hiring is the hidden cost the open-source license doesn't cover — the classic build-vs-buy trap (§4.4).
2. **The real gap is narrower than the narrative.** The bank's R3 pain (self-service) is addressable *within* the Nutanix platform via Prism Central projects + Calm; it does not require rebuilding the platform. The board's "OpenStack" ask was really a "self-service + open-source optics" ask — and both are partially satisfiable on the buy path.
3. **The economics check out.** A 3-year TCO model (Flagged: planning-grade, directional): Nutanix subscription renewal ≈ predictable per-node cost; the OpenStack build's incremental 3–4 cloud engineers at Singapore rates would exceed the subscription delta for the modeled 1,500→3,500 VM trajectory — the amortization point (where build beats buy) lands beyond the bank's 3-year horizon.
4. **The market trend supports patience.** All three major OpenStack distributions have pivoted onto Kubernetes (§5); the "classic OpenStack build" the board imagines is not where the vendor ecosystem is investing. If the bank ever needs true IaaS, it should evaluate OpenStack *on K8s* or an OpenShift-based path then — not a 2015-style greenfield build now.

### 8.5 The Lessons

1. **"Open-source vs commercial" is rarely the real decision.** The real decision is *who owns the integration risk* — your team (build) or a vendor (buy). Banks with small platform teams buy; cloud builders with engineering benches build.
2. **Self-service demand is the strongest OpenStack argument — and the most often misdiagnosed.** Before choosing a platform, quantify whether "self-service" means full IaaS APIs (OpenStack) or governed project portals (Prism Central, vRealize-style tools). Most enterprises need the latter.
3. **TCO models must price labor, not licenses.** OpenStack's $0 license is the cheapest line item in its TCO; the engineering line is the expensive one. Model your own market rates — the break-even moves a lot.
4. **Market trajectory is a legitimate criterion.** A 2026 OpenStack decision is implicitly an OpenStack-on-Kubernetes decision (§5); evaluate the platform the ecosystem is actually building, not the one in the slideware.
5. **The choice can be revisited cheaply if you design for it.** Keeping workloads portable (standard VMs, standard storage interfaces), documenting the self-service gap, and setting a scale trigger means the build option stays warm — the bank's decision is reversible at the right moment, which is the best hedge either philosophy offers.

### 8.6 The Risk Register (what could go wrong)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| **Self-service program underdelivers** — Prism projects/Calm don't close the R3 gap for power users | Medium | Medium | Pilot with the two most demanding teams first; publish the API roadmap; keep the OpenStack evaluation warm |
| **Skills attrition** — the 4-person platform team *is* the platform | Medium | High | Vendor training pipeline, runbook documentation in Prism, partner shadowing, cross-train two more staff |
| **Subscription renewal leverage** — Nutanix renewal pricing drifts up | Medium | Medium | Negotiate the ELA in year 2; keep competitive quotes (vSAN/SimpliVity/OpenStack-distro) warm; track licensed vs used capacity monthly |
| **Scale trigger misfires** — the estate passes 5,000 VMs and the build option has gone stale | Low–Med | Medium | Pre-committed re-evaluation gate with a refreshed TCO and a proposed cloud-engineering budget |
| **Audit/regulatory surprise** — MAS review finds the new self-service controls insufficient | Low | High | Involve internal audit in the Prism projects design from day one; use Flow policies + Prism RBAC logs as the evidence artifacts |
| **Group mandate hardens** — "open-source-first" becomes a hard directive | Low–Med | Medium | Lean on the KVM-based AHV story and open-source components in the stack; document an open-source roadmap with dates |

### 8.7 The Phased Roadmap

| Phase | Window | Scope | Success signal |
|---|---|---|---|
| **0 — Re-verify** | Months 1–2 | Re-run the §7.3 scoring template with Finance + audit; confirm the TCO model; publish the decision memo | CFO + CRO sign-off on staying with the platform; the memo answers the board's open-source question |
| **1 — Self-service pilot** | Months 3–5 | Prism Central projects + RBAC + Calm blueprints for two pilot app teams; API access for CI/CD | Pilot teams provision their own VMs with approvals; provisioning time drops from days to hours |
| **2 — Self-service rollout** | Months 6–12 | Roll projects/blueprints to all 12 teams; marketplace catalog; quota and cost reporting (Beam-style) | 80% of routine provisioning is self-service; audit trail covers all changes |
| **3 — Open-source optics** | Months 6–12 | Document the open-source components (KVM, Linux guests, any OSS in the stack); publish the open-source roadmap | Board narrative satisfied with facts, not a platform swap |
| **4 — Re-evaluation gate** | Month 24 (or scale trigger) | If estate > 5,000 VMs or a true IaaS mandate emerges: re-run §7.3 with a cloud-engineering budget proposal | Decision is evidence-based either way — the build option is warm, not feared |

---

## 9. Summary — The Buy vs the Build of the Private Cloud

**The one-page scorecard:**

| Question | Nutanix | OpenStack |
|---|---|---|
| What is it? | A product: commercial HCI platform, subscription | A project: open-source IaaS, Apache 2.0 |
| Who made it? | Nutanix, Inc. (2009, San Jose; Nasdaq: NTNX) | Rackspace + NASA (2010); OpenInfra Foundation today |
| Core components | AOS + AHV + Prism | Nova, Neutron, Cinder, Glance, Keystone, Swift, Heat |
| Architecture | Converged (hyperconverged) | Disaggregated (composable) |
| Deployment | Days, installer-driven, one console | Weeks–months, integration-heavy, distro tooling helps |
| Operations | One vendor, one console, one upgrade path | Many services; you or your distro own day 2 |
| Licensing | Per-core + per-TiB subscription (AHV free) | Free license; pay for engineering + optional support |
| Best at | Simplicity, predictability, small teams, VMware replacement | Multi-tenant IaaS APIs, composability, scale, telco/research |
| Market gravity | Mainstream enterprise and regulated sectors | Telco NFV, research/HPC, cloud builders (55M+ cores, Reported) |
| Trends (2026) | Platform breadth (storage/DB/net/AI), NC2 hybrid | Mature substrate; distros pivoted to OpenStack-on-Kubernetes |
| The verdict | Buy when integration and time-to-value rule | Build when scale, freedom, and engineering capability rule |

**Nutanix and OpenStack are the two canonical answers to the same question — "we want cloud-style infrastructure, but the data stays with us" — and they are philosophical opposites.** Nutanix (founded 2009, HCI pioneer, NASDAQ: NTNX, ARR $2.2B+, cross-ref [nutanix_products_guide.md](nutanix_products_guide.md)) sells the private cloud as a **product**: AOS distributed OS + AHV (free KVM hypervisor) + Prism management, one vendor, one console, one upgrade path, one support contract, subscription-priced per core and TiB. OpenStack (launched October 2010 by Rackspace + NASA, now the OpenInfra Foundation) is the private cloud as a **project**: Nova/Neutron/Cinder/Glance/Keystone/Swift/Heat assembled into a multi-tenant IaaS platform — Apache-2.0 free, infinitely composable, and operationally yours (or your distribution vendor's).

**The head-to-head in one breath:** Nutanix wins on **integration, simplicity, time-to-value, and predictable economics**; OpenStack wins on **multi-tenant IaaS APIs, composability, zero license cost, and scale** — and its market is real but specialist (telco NFV, research/HPC, cloud-builder private clouds, 55M+ production cores reported), while its three major distributions (Red Hat, Canonical, Mirantis) have all pivoted onto Kubernetes. The honest 2026 read: OpenStack is a mature, quietly massive substrate, not a growth platform for mainstream enterprises; the enterprise default remains integrated commercial platforms like Nutanix — or the public cloud ([cloud_providers_guide.md](cloud_providers_guide.md)).

**The worked example is the whole argument compressed:** a mid-size bank with a small platform team and a 12-month gate evaluates both, scores Nutanix 4.25/5 vs OpenStack 3.25/5, and stays on Nutanix — extending Prism self-service to close the real gap, and setting a scale/skills trigger for revisiting the build option. The decision turned on *who owns the integration risk*, not on license fees.

**The final word:** the private cloud is not really a choice between **buy vs build** — it is a choice between **buying a product (Nutanix) and building a capability (OpenStack)**. Buy when your organization's scarce resource is engineering time and your need is predictable; build when your scarce resource is license budget and your need is a cloud at scale. The bank buys; the telco builds; and the wise architect keeps the other option warm — because the private-cloud pendulum swings with every licensing change, every skills shortage, and every sovereign-infrastructure mandate ([htx_ngine_guide.md](htx_ngine_guide.md) is the reminder that even sovereign builds often end up buying the platform underneath).

---

## 10. Glossary

- **Nutanix** — The company (founded 2009, Nasdaq: NTNX) that pioneered hyperconverged infrastructure and sells the Nutanix Cloud Platform as a subscription software stack (cross-ref [nutanix_products_guide.md](nutanix_products_guide.md)).
- **OpenStack** — The open-source IaaS cloud platform launched October 2010 by Rackspace and NASA; a modular collection of services (Nova, Neutron, Cinder, …) governed by the OpenInfra Foundation.
- **HCI** — Hyperconverged Infrastructure: pooling compute and storage across standard x86 servers into one software-defined cluster with no separate SAN.
- **Hyperconverged** — The architecture where compute and storage are fused per node and scaled together (Nutanix's model), as opposed to disaggregated pools.
- **AOS** — Acropolis Operating System: Nutanix's distributed operating system; the storage fabric and platform engine of the Nutanix Cloud Platform.
- **AHV** — Acropolis Hypervisor: Nutanix's built-in KVM-based enterprise hypervisor, included free with AOS.
- **Prism** — Nutanix's management plane: Prism Element (per-cluster) and Prism Central (multi-cluster governance, self-service, marketplace).
- **Nova** — OpenStack's compute service: VM lifecycle management (schedule, provision, migrate, resize).
- **Neutron** — OpenStack's networking service: virtual networks, routers, LB, security groups, floating IPs.
- **Cinder** — OpenStack's block-storage service: persistent volumes attached to instances.
- **Glance** — OpenStack's image service: the registry of VM disk images and snapshots.
- **Keystone** — OpenStack's identity service: users, projects, roles, tokens, service catalog.
- **Swift** — OpenStack's object-storage service (the Rackspace heritage from 2010).
- **Heat** — OpenStack's orchestration service: template-driven stacks (HOT format).
- **Rackspace** — The cloud-hosting company that co-launched OpenStack in 2010, contributing Cloud Files → Swift.
- **NASA** — The US space agency whose Nebula cloud project contributed the compute controller → Nova.
- **Red Hat OpenStack Platform** — Red Hat's commercial OpenStack distribution (RHEL-based); transitioning to Red Hat OpenStack Services on OpenShift (GA Aug 2024).
- **Charmed OpenStack** — Canonical's Ubuntu-based OpenStack distribution, operated with Juju charms.
- **Mirantis** — The OpenStack-services pioneer (acquired Docker Enterprise 2019) now offering Mirantis OpenStack for Kubernetes (GA April 2025).
- **Private cloud** — Cloud-style infrastructure (self-service, multi-tenancy) operated inside an organization's own data center.
- **IaaS** — Infrastructure-as-a-Service: compute, storage, and networking provisioned through APIs — the service model OpenStack delivers natively.
- **Disaggregated** — The architecture where compute, storage, and network are separate pools, scaled independently (OpenStack's model).
- **Convergence** — The architecture where components are fused into one integrated system (HCI's model).
- **Distribution** — A vendor's packaged, supported version of an open-source project (e.g., Red Hat's, Canonical's, Mirantis' OpenStack).
- **Licensing** — The legal/business terms for using software: Nutanix's proprietary subscription (cores + TiB) vs OpenStack's Apache 2.0 open-source license.
- **Open source** — Software whose source is freely available and modifiable; OpenStack is Apache-2.0 licensed.
- **Support** — The accountable help contract: one vendor for Nutanix; community or a distribution vendor for OpenStack.

---

## 11. Claims Status, References and Further Reading

### 11.1 Claims Status

| Claim | Status |
|---|---|
| Nutanix: founded 2009, IPO 2016 (Nasdaq, +131% first day), AOS/AHV/Prism roles, subscription licensing (cores + TiB), NC2, Files/Objects/NDB/Flow portfolio | **Verified** — cross-ref [nutanix_products_guide.md](nutanix_products_guide.md) §1–§9 (its own claims-status table is §14 there) |
| Nutanix FY2025 ARR $2.223B, +17% YoY, 108% NDR | **Reported** — Q4 FY2025 earnings coverage (Motley Fool transcript, StorageNewsletter); figures move quarterly |
| Nutanix customer counts (~35k), Gartner MQ Leader history | **Flagged** — company claims / directional, not independently audited |
| OpenStack launched by Rackspace + NASA; first release "Austin" 21 October 2010; Nova from NASA's Nebula, Swift from Rackspace Cloud Files | **Verified** — Rackspace history, The Register launch coverage, NASA tech-transfer article |
| OpenStack core components (Nova/Neutron/Cinder/Glance/Keystone/Swift/Heat + Horizon/Ceilometer etc.) and their functions | **Verified** — OpenStack project documentation and multiple independent references |
| OpenStack Foundation renamed OpenInfra Foundation 2021; joined Linux Foundation 2024 | **Verified** (2021 rename); **Reported** (Linux Foundation move — Network World) |
| OpenStack production scale: 40M+ cores (2022), 55M+ cores (2025 survey), 300+ public-cloud DCs | **Reported** — self-reported OpenInfra survey data; directional |
| Red Hat OpenStack Services on OpenShift GA 26 August 2024; RHOSP 17.x classic line ends; Red Hat positions OpenShift Virtualization as parallel destination | **Verified** (GA date — Red Hat blog, TechCrunch); migration economics **Reported/Flagged** (verify current support windows) |
| Canonical Charmed OpenStack: Ubuntu-based, Juju/charms, Ceph integrated, no per-core license | **Verified** — Canonical materials |
| Mirantis OpenStack for Kubernetes GA April 2025; Docker Enterprise acquisition 2019; IREN acquisition of Mirantis | **Verified** (GA — Mirantis announcement); IREN deal **Reported** |
| Telco leads OpenStack industry share (~28%); market size estimates | **Flagged** — third-party market research (Mordor Intelligence); estimates vary widely |
| "OpenStack use plummets" (The New Stack, Feb 2025) vs OpenInfra's growth surveys | **Reported** — competing narratives; the honest synthesis is in §6.2 |
| Comparative assessments (architecture verdicts, TCO break-even intuition, which-for-which-need, worked-example scores) | **Flagged** — analyst/experience views, marked inline; model with your own numbers |

### 11.2 References and Further Reading

- OpenStack / OpenInfra: openstack.org (project docs, releases — incl. 2025.2 Flamingo), openinfra.org (foundation, user surveys), Rackspace "What is OpenStack" history page; The Register — "OpenStack passes 40 million cores" (Nov 2022) and Austin-release coverage (Oct 2010); NASA tech-transfer article on the Nebula/OpenStack origins.
- Distributions: Red Hat blog "Navigating the future: Red Hat OpenStack Platform 17.1 to Red Hat OpenStack Services on OpenShift"; TechCrunch (Aug 2024) GA coverage; Canonical — Charmed OpenStack pages and "OpenStack distros" comparison; Mirantis — "Announcing Mirantis OpenStack for Kubernetes" (Apr 2025) and press history; Wikipedia (Mirantis) for the Docker Enterprise acquisition.
- Market/trends: The New Stack — "Bad News for Cloud Computing: OpenStack Use Plummets…" (Feb 2025); Canonical — "OpenStack is dead? The numbers speak for themselves" (Mar 2022); Forrester — "The State of OpenStack, 2025"; Mordor Intelligence — OpenStack Services Market (Flagged: directional).
- Sibling guides in this repo: [nutanix_products_guide.md](nutanix_products_guide.md) (the Nutanix side, verified in full), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) + [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) + [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) (the head-to-head pattern and the K8s platforms OpenStack now runs on), [cloud_providers_guide.md](cloud_providers_guide.md) (the cloud framing), [finops_guide.md](finops_guide.md) (the economics lens), [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) + [dell_objectscale_guide.md](dell_objectscale_guide.md) + [s3_architecture_guide.md](s3_architecture_guide.md) (the storage layer both platforms touch), [htx_ngine_guide.md](htx_ngine_guide.md) (sovereign-infra angle), [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) (on-prem AI), and the `banking/` guides (regulated data-center modernization).

---

*End of guide — Nutanix vs OpenStack: Commercial HCI vs Open-Source IaaS. Author: Jack Liu Shurui, Solution Architect, Crédit Agricole CIB. August 2026.*
