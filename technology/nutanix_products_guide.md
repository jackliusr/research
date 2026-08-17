# Nutanix Products — A Comprehensive Guide to the Nutanix Cloud Platform

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Technology Research — Infrastructure / Hybrid-Cloud series; dedicated product-portfolio survey of the Nutanix Cloud Platform, from the hyperconverged (HCI) core to storage, databases, networking, cloud, desktop, and AI
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** Nutanix corporate site and press releases; Nutanix portal/licensing documentation; The Nutanix Bible (nutanixbible.com); earnings/IPO filings coverage; vendor comparisons (TechTarget, Enterprise Storage Forum)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Company Overview — Nutanix](#1-company-overview--nutanix)
2. [The HCI Core — AOS, AHV and Prism](#2-the-hci-core--aos-ahv-and-prism)
3. [The Storage Products — Files, Objects, Volumes, Unified Storage](#3-the-storage-products--files-objects-volumes-unified-storage)
4. [The Database Products — Nutanix Database Service (NDB)](#4-the-database-products--nutanix-database-service-ndb)
5. [Networking and Security — Flow](#5-networking-and-security--flow)
6. [The Cloud Products — Clusters, NC2 and the Hybrid Cloud](#6-the-cloud-products--clusters-nc2-and-the-hybrid-cloud)
7. [Desktop and Other Products — Frame, Beam, Calm, Mine](#7-desktop-and-other-products--frame-beam-calm-mine)
8. [The AI Products — GPT-in-a-Box](#8-the-ai-products--gpt-in-a-box)
9. [Licensing — The Subscription Model](#9-licensing--the-subscription-model)
10. [Comparison — Nutanix vs the Alternatives](#10-comparison--nutanix-vs-the-alternatives)
11. [Worked Example — A Mid-Size Bank's HCI Deployment](#11-worked-example--a-mid-size-banks-hci-deployment)
12. [Summary — Nutanix in One Page](#12-summary--nutanix-in-one-page)
13. [Glossary](#13-glossary)
14. [Claims Status, References and Further Reading](#14-claims-status-references-and-further-reading)

### How to Read This Guide

This is the dedicated deep-dive on **Nutanix products** — the product-portfolio survey of the Nutanix Cloud Platform — in the `technology/` infrastructure / hybrid-cloud series. Several sibling guides carry adjacent depth and are cross-referenced inline:

- **The sovereign-AI tie** — [htx_ngine_guide.md](htx_ngine_guide.md) covers HTX NGINE, Singapore's first enterprise-grade AI infrastructure, built by ST Engineering with NVIDIA + **Nutanix** + Google (Nutanix is the platform layer there; §8 of this guide connects GPT-in-a-Box to that pattern).
- **The cloud angle** — [cloud_providers_guide.md](cloud_providers_guide.md) is the vendor-neutral cloud comparison that §6's hybrid-cloud discussion builds on; [finops_guide.md](finops_guide.md) is the cost lens that §9's licensing economics mirror.
- **The storage angle** — [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) covers CephFS and other distributed-file alternatives to §3's Files; [dell_objectscale_guide.md](dell_objectscale_guide.md) and [s3_architecture_guide.md](s3_architecture_guide.md) are the object-storage references that Nutanix Objects competes against.
- **The Kubernetes-platform angle** — [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md), [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) and [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) are the container-platform alternatives to Nutanix's own Kubernetes story (Karbon / NKE), relevant when the platform choice is OpenShift rather than AHV.
- **The on-prem AI angle** — [on_prem_llm_deployment_guide.md](ai_llm/on_prem_llm_deployment_guide.md) is the playbook for the kind of on-prem GPU estate GPT-in-a-Box (§8) delivers; the `banking/` guides give the regulated-sector context that the worked example in §11 reuses.

**Note on verification.** This guide was researched in August 2026. Claims are marked **Verified** (confirmed against Nutanix primary sources or reliable secondary coverage during research), **Reported** (widely reported but not independently confirmed), or **flagged** inline where specifics are approximate or could not be pinned down. The full claims-status table is in §14.

---

## 1. Company Overview — Nutanix

### 1.1 What Nutanix Is

> **Nutanix is the company that invented the modern hyperconverged infrastructure (HCI) category — a software-defined platform that pools compute and storage across standard x86 servers — and has since pivoted from an appliance vendor into a subscription-based hybrid-multicloud platform company.**

Nutanix, Inc. (NASDAQ: NTNX) is headquartered in San Jose, California. Its flagship offering, the **Nutanix Cloud Platform (NCP)**, is a software stack that delivers virtualized compute, enterprise storage (block, file and object), networking and security, database services, and increasingly AI — runnable on its own commodity-hardware clusters, on partner hardware, or inside the big three public clouds. The one-sentence positioning Nutanix itself uses: *"one platform to run apps, data, and AI anywhere."*

### 1.2 The Founding — September 2009 (Verified, with a Correction)

Nutanix was founded on **September 23, 2009** (verified against Nutanix history coverage and multiple independent sources: Wikiwand, MarketWatch, The Network DNA). The founders were:

- **Dheeraj Pandey** (first CEO, 2009–2020) — previously an engineer at Oracle and Microsoft, and a co-founder of Aster Data.
- **Mohit Aron** (founding CTO/chief architect) — previously at Google and Aster Data; he left Nutanix in early 2013 to found **Cohesity** (backup/data-management company).
- **Ajeet Singh** — previously at Aster Data; left Nutanix in 2014.

> **Verification correction — flag:** the commonly circulated founder list "Dheepak Bhardwaj, Binny Gill, Mohit Aron, Ajeet Singh" is **not correct**. Public records consistently name **Dheeraj Pandey, Mohit Aron and Ajeet Singh** as the three co-founders. **Binny Gill** was a very early employee and long-time chief architect / CTO-figure at Nutanix (and later co-founder of Kognitos), but he is not listed among the company's founders in filings or Nutanix's own history. The task brief's founder list appears to have mixed up "Dheeraj Pandey" with a phonetically similar name and promoted an early employee to founder — corrected here.

The founding insight was that data-center infrastructure was stuck in the 1990s: separate server, SAN, and networking tiers that were expensive, hard to scale, and operationally fragile. Nutanix's bet was to run the entire stack — compute virtualization and storage — as distributed software on commodity servers, with intelligence embedded in the data path. The first product shipped in 2011–2012 as a hardware appliance bundle ("Nutanix Complete Cluster"), and the category became known as **hyperconverged infrastructure (HCI)**.

### 1.3 The IPO — 30 September 2016, Nasdaq (Verified)

Nutanix went public on **September 30, 2016** on the **Nasdaq** exchange under the ticker **NTNX** (verified via Nasdaq's own welcome release and TechCrunch's first-day coverage). The IPO details are worth noting because they show how hot the HCI wave was:

- Shares priced at **$16** (above the initial $13–$15 range).
- Opened at **$26.50** and closed the first day around **$37** — a **+131% first-day pop**.
- The market cap crossed **$3 billion** on day one (Reported, Mohit Aron's later account).

Post-IPO, Nutanix executed a long, painful, and ultimately successful financial transition: from selling hardware-bundled appliances with perpetual licenses to selling **pure software on a subscription basis** — a change management began around fiscal 2019 and completed by fiscal 2021 (the company stopped bundling hardware entirely, partnering with OEMs such as Dell, HPE, Lenovo, Cisco and Fujitsu for the servers). The stock gave back much of its early gains during this transition; the subscription metrics (ARR, NRR) only turned convincingly profitable in the mid-2020s (Reported).

### 1.4 The Pivot — From HCI Appliance Vendor to Hybrid-Multicloud Platform (Verified, 2020s)

The 2020s pivot is real and is the single most important thing to understand about Nutanix's current portfolio. It happened in three visible moves:

1. **Software-first, subscription-first (2018–2021).** Nutanix decoupled its software from any specific hardware, introduced term-based licensing, and positioned AOS as "the cloud OS for the data center." This is the licensing model described in §9.
2. **The platform umbrella — Nutanix Cloud Platform (2020).** Nutanix rebranded its portfolio around the NCP: HCI (AOS/AHV/Prism) as the core, with **Files, Objects, Volumes, NDB, Flow, Calm and Beam** as services on top — explicitly competing with the public-cloud operating model inside the private data center.
3. **Hybrid multicloud — NC2 (2020–2022).** Nutanix launched **Nutanix Clusters on AWS (2020), on Azure (2021) and on Google Cloud (2022)**, letting customers run the exact same AOS/AHV/Prism stack on public-cloud bare metal. Combined with the 2020 CEO transition (Dheeraj Pandey stepped down; **Rajiv Ramaswami**, ex-VMWare/Google/NVIDIA, became CEO in December 2020 — Verified), the company's story became: *"one cloud platform, running identically on-prem and in every public cloud."* The 2023 GPT-in-a-Box launch (§8) extended the same story to AI.

The strategic bet behind the pivot: the 2020s enterprise isn't choosing "on-prem OR cloud" — it's running a hybrid mix, and it wants the *same* operational model, tooling, and skills everywhere. Nutanix's wager is that the HCI core plus this "anywhere" story is a credible third way between pure on-prem legacy stacks and full public-cloud consumption.

### 1.5 The Company at a Glance

| Aspect | Description |
|---|---|
| **Founded** | 23 September 2009, San Jose, California (Verified) |
| **Founders** | Dheeraj Pandey, Mohit Aron, Ajeet Singh (Verified — see correction in §1.2) |
| **IPO** | 30 September 2016, Nasdaq (NTNX); priced $16, closed first day ~$37 (+131%) (Verified) |
| **CEO (2020–)** | Rajiv Ramaswami (Verified); founder Dheeraj Pandey served 2009–2020 |
| **Core product** | Nutanix Cloud Platform — AOS, AHV, Prism (§2), storage services (§3), NDB (§4), Flow (§5), NC2 (§6), GPT-in-a-Box (§8) |
| **Business model** | 100% software, subscription/term licensing, hardware-agnostic; OEM partners provide servers (Verified — see §9) |
| **Scale** | ~35,000+ customers claimed across ~150 countries (Flagged: company claim, exact figure moves quarterly and is not independently audited) |
| **Market position** | HCI pioneer and long-time Leader in Gartner's HCI Magic Quadrant (2019–2022 era; Reported); competes with VMware vSAN, HPE SimpliVity, Scale Computing (§10) |
| **Singapore/APJ** | Strong APJ presence; Nutanix platform underpins HTX NGINE's enterprise layer (see [htx_ngine_guide.md](htx_ngine_guide.md)) |
| **Adoption** | Nutanix marketing claims a majority of Fortune 500 enterprises as customers (Flagged: "majority of the Fortune 500" is a company marketing claim, not independently verifiable) |

---

### 1.6 The Product Timeline — How the Portfolio Was Built

The portfolio map in this guide is easier to hold onto with the launch chronology. Verified dates are marked; the rest are Reported (widely documented in Nutanix .NEXT keynotes and press coverage).

| Year | Milestone | Product family |
|---|---|---|
| 2009 | Founded, 23 September (Verified) | — |
| 2011–2012 | First HCI appliance ships ("Nutanix Complete Cluster") (Verified) | HCI core |
| 2013 | Mohit Aron departs to found Cohesity (Verified); Acropolis platform branding emerges (Reported) | Platform |
| 2015 | AHV (Acropolis Hypervisor) and the Prism management story introduced (Reported) | HCI core |
| 2016 | **IPO, 30 September, Nasdaq** (Verified) | — |
| 2017 | **Calm** (application automation) launched at .NEXT (Reported) | NCM services |
| 2018 | **Files, Objects, Beam, Mine** (with HYCU) launched; **Frame acquired** (Aug, Verified) | Storage + services |
| 2019 | **Era** (DBaaS) launched; Xi LEAP DR service (Reported) | Database |
| 2020 | **Clusters on AWS** (Verified); CEO transition to Rajiv Ramaswami (Verified); subscription-first transition completes (Reported) | Cloud |
| 2021 | **Clusters on Azure** (Verified); **Unified Storage (NUS)** packaging; Era → NDB rebrand begins (Reported) | Cloud + storage |
| 2022 | **Clusters on Google Cloud** (Reported — see §6); Era → NDB rebrand completes (Verified via court filing) | Cloud + database |
| 2023 | **GPT-in-a-Box** announced 15 Aug (Verified); **Frame divested to Dizzion** (Jun, Verified) | AI |
| 2024–2025 | NC2 on GCP broadens; NVIDIA NIM/NeMo integration deepens; Nutanix is the platform layer in Singapore's HTX NGINE sovereign-AI build (Verified — see [htx_ngine_guide.md](htx_ngine_guide.md)) | AI + cloud |

Read this table top-to-bottom and you see the pivot: **2016–2019 = the HCI era (compute + storage + services), 2020–2022 = the cloud era (NC2, NUS, NDB), 2023+ = the AI era (GPT-in-a-Box)** — each era layering on top of the previous one rather than replacing it. That layering is why the portfolio is broad but coherent: everything still runs on the same AOS/AHV/Prism core.

---

## 2. The HCI Core — AOS, AHV and Prism

### 2.1 What HCI Is, In One Paragraph

Hyperconverged infrastructure replaces the classic three-tier data center (servers + SAN + network) with **clusters of standard x86 servers where storage is virtualized and distributed across every node's local disks**. There is no separate SAN: every node contributes CPU, RAM, and disk, and software pools them into a resilient, self-healing fabric. Scale-out means adding a node (or a few) grows compute *and* capacity together. Nutanix's HCI core is three tightly integrated layers — **AOS** (the operating system), **AHV** (the hypervisor), and **Prism** (the management plane) — and that trio is the engine under every other product in this guide.

### 2.2 AOS — Acropolis Operating System (Verified)

**AOS (Acropolis Operating System)** is Nutanix's distributed operating system, the foundation of the Nutanix Cloud Platform (Verified against Nutanix product docs; the acronym expands to Acropolis Operating System). Key facts:

- **Distributed storage fabric.** AOS runs a distributed, replica-based storage engine (historically known internally as "Stargate") that stripes data across all nodes in the cluster, maintaining 2 or 3 copies (RF2/RF3) or erasure coding across nodes. Every node serves I/O for the whole cluster — there is no controller bottleneck.
- **Metadata plane.** AOS keeps cluster metadata (where each block lives) in a distributed, Cassandra-style metadata store, so any node can route any I/O request — the "web-scale" design borrowed from search-engine architectures.
- **Data services built in.** Snapshots, clones, thin provisioning, deduplication, compression, self-healing re-replication, and DR (async replication between clusters) are native AOS capabilities — not bolt-on products.
- **Protocol services on top.** AOS natively serves **block (iSCSI — Volumes), file (NFS/SMB — Files), and object (S3 — Objects)** from the same distributed fabric (§3). This is what makes "Unified Storage" a packaging question rather than a new engine.
- **Versioning.** AOS versions (e.g. 5.x, 6.x) are long-lived and upgrade in-place via rolling node reboots; upgrades are orchestrated by Prism.

The "Acropolis" name extends across the family: **AHV = Acropolis Hypervisor**, and the platform was historically branded "Acropolis" before the Nutanix Cloud Platform umbrella took over.

### 2.3 AHV — Acropolis Hypervisor (Verified)

**AHV (Acropolis Hypervisor)** is Nutanix's built-in enterprise hypervisor, based on open-source **KVM** (Verified — Nutanix documents AHV as "a KVM-based enterprise hypervisor"; AHV also uses libvirt for VM lifecycle and OVS for virtual switching). Key facts:

- **Free with the platform.** AHV carries no per-socket license fee — it is included with AOS. This is the single biggest cost differentiator versus VMware vSAN/vSphere stacks (§10) and the lever that makes "hypervisor choice" a real option in Nutanix deals.
- **Full enterprise feature set.** Live migration, high availability (HA) with node-failure restart, snapshots, clones, and QoS — comparable to what vSphere offers for mainstream VM workloads.
- **One-click lifecycle.** AHV upgrades ship with AOS and are applied through Prism as rolling updates, removing the separate hypervisor-upgrade project that plagues vSphere shops.
- **Guest OS coverage.** Windows Server, Linux distributions (RHEL, Ubuntu, SUSE), and modern Windows/Linux guests are supported; AHV also runs Nutanix's own Kubernetes distribution (Nutanix Kubernetes Engine, NKE) on VMs.

**Honest caveat (Flagged as opinion/experience, not vendor claim):** AHV is not a drop-in for every VMware shop. Some advanced vSphere features (SR-IOV maturity, DRS-style automated balancing depth, third-party ecosystem like Veeam/NSX integrations) are thinner on AHV, and team skills matter. For greenfield or migration-driven deals, though, AHV removes a six-figure licensing line item.

### 2.4 Prism — The Management Plane (Verified)

**Prism** is Nutanix's management plane (Verified — Nutanix docs: "Prism is the management layer of the Nutanix Cloud Platform"). It comes in two forms:

- **Prism Element** — per-cluster management, embedded in the cluster itself. One-click visibility into VMs, storage, alerts, and capacity; the "single pane" for day-2 operations of one cluster.
- **Prism Central** — multi-cluster management and governance: fleet-wide monitoring, capacity planning, policy-based automation (tags, categories), self-service via RBAC and projects, and marketplace-style app deployment. Prism Central is where Calm (automation, §7), Flow (networking/security, §5), and the licensing dashboard hang off.

Prism's design philosophy is deliberately "cloud console meets enterprise ops": everything is clickable, health is scored (the "health dashboard" with red/yellow/green checks), and most maintenance (upgrades, capacity additions, expansion) is wizard-driven rather than runbook-driven.

### 2.5 The HCI Core — Product Table

| Product | Function | Notes |
|---|---|---|
| **AOS** (Acropolis Operating System) | Distributed OS: pools compute + storage across nodes; serves block/file/object protocols; snapshots, clones, dedupe, compression, DR built in | The "engine room"; licensed per core + flash capacity (§9); upgrades in place, orchestrated by Prism |
| **AHV** (Acropolis Hypervisor) | KVM-based enterprise hypervisor for VMs; live migration, HA, snapshots | Included free with AOS — the cost killer vs vSphere; upgrades ship with AOS |
| **Prism** | Management plane: Prism Element (per cluster) + Prism Central (multi-cluster governance, self-service, marketplace) | The "single pane"; also hosts Flow, Calm, and the licensing console |
| **Supporting cast** | Stargate (distributed storage engine), Cassandra-style metadata store, distributed file system (NDFS) | Internal AOS components; useful to know because they explain why Nutanix behaves like one big distributed system |

---

### 2.6 How the Platform Works Internally

A few internals are worth knowing because they explain *why* Nutanix behaves the way it does — and they come up in every architectural interview about HCI (the Nutanix Bible, nutanixbible.com, is the canonical community reference for this layer):

- **NDFS (Nutanix Distributed File System)** — the cluster-wide filesystem that spans all nodes' local disks; there is no shared storage of any kind. Every disk in every node is a member of one logical pool.
- **The Controller VM (CVM)** — each node runs a small controller VM that hosts the AOS data path (the I/O engine, historically called **Stargate**). Guest VMs do *not* read their local disk directly; they go through the CVM, which can route any I/O to any node. This indirection is what makes the cluster behave as one big distributed disk.
- **Metadata store** — extent-location metadata lives in a distributed, Cassandra-derived store (historically "Medusa"), so any node can locate any block without a central index. This is the "web-scale" trick borrowed from search-engine architecture.
- **Replication and erasure coding** — data is protected by full copies (RF2 = 2 copies, RF3 = 3 copies) or by erasure coding (EC-X, e.g., EC-4:2 tolerating two failures with ~1.5× overhead instead of 2×). Policies are per-storage-container, so hot VMs can run RF2/RF3 while archive data runs EC.
- **Self-healing** — after a node or disk failure, AOS re-replicates the affected data automatically, prioritizing availability; rebuilds are throttled to protect performance but need no human runbook.
- **Failure domains** — administrators can define rack/block awareness so replicas land in different fault domains (a rack loss doesn't take both copies).
- **One-click expansion** — adding a node rebalances data across the cluster automatically. There is no "add a shelf to the SAN" project, which is the operational difference that most Nutanix customers cite first.

### 2.7 Cluster Design Notes (directional)

- **Minimum footprints:** a 3-node cluster is the practical minimum (RF2 with a witness consideration); RF3 and meaningful resilience want 4+ nodes; production designs typically start at 4–8 nodes per cluster (§11 sizes 8).
- **Node shapes:** all-flash nodes (hot VM workloads), hybrid nodes (capacity/archive), and GPU nodes (the G-series) for VDI and AI (§8). Mixing shapes in one cluster is supported but flag: keep the *storage* characteristics homogeneous per cluster for predictable behavior.
- **Cluster-per-workload vs one-big-cluster:** Nutanix supports both; banks typically run separate clusters per environment (prod/DR/dev) and sometimes per compliance domain (see §11.2).
- **DR is cluster-to-cluster:** the standard model is async replication between clusters (VM, Files, and volume replication), with Prism Central orchestrating failover playbooks. Metro/stretch-cluster-style synchronous setups are possible in limited configurations — Flagged: verify current "Nutanix Metro"-style offerings before designing for synchronous RPO=0; most regulated deployments accept async RPO of minutes.

---

## 3. The Storage Products — Files, Objects, Volumes, Unified Storage

Nutanix's storage story is: **one distributed engine (AOS) that speaks every protocol**. The products below are the protocol front-ends and their packaging — they all run on the same cluster and are managed from the same Prism console. This is the direct architectural answer to the siloed world of a separate SAN (block), NAS (file), and object store (§10 comparison, and see [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) and [dell_objectscale_guide.md](dell_objectscale_guide.md) for the alternatives).

### 3.1 Files — File Storage (Verified)

**Nutanix Files** (formerly Acropolis File Services / AFS, rebranded ~2018) is the scale-out NAS / file-services product (Verified — Nutanix docs: "Nutanix Files is a software-defined scale-out file storage solution"). Key facts:

- **Protocols:** NFS (v3/v4) and SMB (Windows file shares), plus SMB for home directories.
- **Architecture:** file-server VMs (FSVMs) run on the cluster and serve the distributed NDFS filesystem; scale by adding nodes or file-server VMs without disruption.
- **Enterprise features:** snapshots and self-service restore (Windows Previous Versions / .snapshot), async replication for DR, anti-virus integration, and quota/entitlement management.
- **Typical uses:** home directories, departmental shares, application file storage, VDI user profiles, and — importantly for §8 — the dataset/checkpoint store for on-prem AI.
- **Licensing:** capacity-based (per TiB of licensed capacity), a different metric from the core-based AOS license (§9).

### 3.2 Objects — Object Storage (Verified)

**Nutanix Objects** (formerly Acropolis Object Storage, rebranded ~2018) is the S3-compatible object store running on the same cluster (Verified — Nutanix docs: "Nutanix Objects is a software-defined object storage solution with an S3-compatible API"). Key facts:

- **S3 compatibility:** the S3 API surface (buckets, objects, lifecycle policies, multipart upload) so S3-native applications work without rewriting — the same compatibility play Dell ObjectScale also makes ([dell_objectscale_guide.md](dell_objectscale_guide.md)).
- **Erasure coding:** objects are erasure-coded across nodes (more capacity-efficient than replication) with bucket-level policies.
- **Use cases:** backup targets (S3-compatible backup apps), archives, media repositories, data lakes, and AI training data — the same workloads S3 serves ([s3_architecture_guide.md](s3_architecture_guide.md)).
- **Deployment:** runs as VMs/containers on the cluster; scales out independently of the VM tier.

### 3.3 Volumes — Block Storage (Verified)

**Nutanix Volumes** (formerly Acropolis Block Services) is the block-storage front-end that lets **non-AHV** consumers use cluster storage via **iSCSI** (Verified — Nutanix docs describe Volumes as "iSCSI-based block services"). Key facts:

- **Purpose:** presents the distributed cluster storage as LUNs to external hosts — e.g., VMware ESXi clusters, bare-metal servers, or legacy applications that require raw block.
- **Note:** VMs running on AHV do *not* need Volumes — they consume storage natively through AOS at wire speed. Volumes exists for the *other* hypervisors and bare-metal consumers. This distinction confuses many architects; it matters for sizing (§11).
- **Features:** thin provisioning, snapshots, clones, QoS per volume.

### 3.4 Unified Storage — Block + File + Object on One Platform (Verified)

**Nutanix Unified Storage (NUS)** is the packaging (introduced ~2021) that bundles **Volumes (block) + Files (file) + Objects (object)** into one licensed platform, managed through one Prism console, with one upgrade path and one support contract (Verified — Nutanix datasheet: "Nutanix Unified Storage is a software-defined data services platform that consolidates the management and protection of siloed block, file, and object storage into a single, unified platform"). Key facts:

- **The pitch:** replace three separate storage silos (a SAN, a NAS, an object store) with one scale-out platform that a "storage generalist" can run, starting as small as ~1 TiB and growing to multi-PB.
- **Data services are unified:** snapshots, replication, and DR policies apply uniformly across protocols — a file share, a volume, and a bucket all get the same protection story.
- **The honest comparison (Flagged as analyst/experience view):** NUS is competitive with dedicated arrays (NetApp ONTAP for file, Pure/Dell for block, AWS S3/on-prem object stores) in the majority of mainstream cases, but dedicated products still win specific niches — extreme random-IO block performance (pure-play AFA), very deep object ecosystems, or exotic NAS features (e.g., some multiprotocol or protocol-locking edge cases). The durable Nutanix win is *integration and consolidation*, not peak per-protocol performance.

### 3.5 The Storage Products — Table

| Product | Type | Use Case |
|---|---|---|
| **Files** | Scale-out NAS (NFS/SMB) | Home directories, departmental shares, app files, VDI profiles, AI datasets |
| **Objects** | S3-compatible object store | Backup targets, archives, data lakes, media, AI training data |
| **Volumes** | Block via iSCSI | External hosts needing raw block: ESXi clusters, bare metal, legacy apps |
| **Unified Storage (NUS)** | Bundle: block + file + object | Consolidating storage silos into one platform with one ops model |

---

### 3.6 Storage Design Notes (directional)

- **Do the capacity math with protection overhead, not raw disk.** Example: 10 TiB of logical data needs ~20 TiB of raw capacity at RF2, but only ~13–15 TiB with EC-4:2 (plus filesystem and metadata overhead, typically 10–15% more). The classic design error is sizing from raw disk and discovering usable capacity is half of it.
- **Files sizing:** file-server VMs (FSVMs) serve the namespace; run at least 2 per cluster for HA, scale the FSVM count as client connections grow. SMB and NFS traffic patterns differ (SMB is chatty at scale — size for it).
- **Objects sizing:** object data is erasure-coded per bucket; enable **immutability (WORM-style object lock)** for backup buckets — this is the ransomware story that Mine (§7.4) and HYCU lean on, and it maps to MAS/audit expectations.
- **Snapshots are free-ish but not free:** native snapshots are cheap and frequent (hourly is normal), but snapshot chains still consume capacity; set retention policies in Prism rather than letting them grow forever.
- **Protocol coexistence:** block (Volumes), file (Files) and object (Objects) can run on the *same* cluster, but mixed-protocol clusters need care with capacity planning — one protection domain's re-replication affects neighbors. Separate the backup/archive tier from the production VM tier when workloads are heavy (the bank in §11 runs a separate 4-node cluster for exactly this reason).
- **Licensing coupling:** NUS capacity licenses are measured in licensed TiB; over-allocating capacity beyond the license is blocked or flagged at the console — size the license with headroom at renewal time (§9.4).

---

## 4. The Database Products — Nutanix Database Service (NDB)

### 4.1 NDB — Nutanix Database Service, formerly Era (Verified)

**Nutanix Database Service (NDB)** — formerly **Nutanix Era** — is Nutanix's Database-as-a-Service (DBaaS) layer for the private/hybrid cloud (Verified: Nutanix's product page — "Nutanix Database Service is a Database-as-a-Service (DBaaS) platform that enables teams to provision, operate, and scale databases across on-premises and hybrid cloud environments"; the Era→NDB rebrand is confirmed by Nutanix's own materials and by third-party documentation, with the rename landing in the 2021–2022 timeframe — one exam-prep source dates it to the NDB 2.4 release, and a 2024 court filing notes "Nutanix Era was later renamed Nutanix Database Service ('NDB') in 2022"). Key facts:

- **Engines supported:** Microsoft SQL Server, Oracle Database, PostgreSQL, MySQL, and MariaDB — the mainstream enterprise transactional set. (MongoDB and other NoSQL support has appeared in roadmap/talk; Flagged: check current support matrix before architecting.)
- **What it automates:** one-click provisioning from templates, patching and minor upgrades, **point-in-time recovery (PITR)** with continuous log shipping, time-travel cloning (clone any database to any point in time, seconds-fast), and HA/failover orchestration.
- **Why DBAs care:** NDB removes the *day-2* toil — the provisioning, patching, cloning, and restore work that consumes DBA time — while keeping the database itself standard (it provisions *real* SQL Server/Oracle/PostgreSQL instances, not a proprietary database). It slots into the "database modernization without rewriting" narrative that regulated banks like Crédit Agricole CIB need (§11).
- **Integration:** NDB sits on AOS storage, so clones and snapshots ride the cluster's native snapshot engine; it is managed from Prism Central; and it can run in NC2 public-cloud clusters too (§6).
- **Strategic note:** NDB is a pillar of Nutanix's "run the database tier the way the cloud does, but in your data center" story, and it pairs with GPT-in-a-Box (§8) for the data layer of on-prem AI.

### 4.2 The Database Products — Table

| Product | Function | Notes |
|---|---|---|
| **NDB** (Nutanix Database Service, formerly Era) | DBaaS for SQL Server, Oracle, PostgreSQL, MySQL, MariaDB: provisioning, PITR, cloning, patching, HA | Formerly "Nutanix Era" (rebrand ~2021–2022, Verified); managed from Prism Central; rides AOS snapshots |

---

## 5. Networking and Security — Flow

### 5.1 Flow — Software-Defined Networking (Verified)

**Nutanix Flow** is the software-defined networking (SDN) and security layer of the Nutanix Cloud Platform, delivered from Prism Central (Verified — Nutanix product page: "Nutanix Flow supports Zero Trust initiatives…"; Nutanix docs describe Flow Virtual Networking as the SDN service for AHV). Key capabilities:

- **Virtual networking (Flow Virtual Networking / FVN):** overlay networks for AHV VMs — VPC-style isolation, private subnets, DHCP, and security policy per tier — without touching physical switches. This is Nutanix's answer to VMware NSX for AHV-centric shops.
- **Policy-based segmentation:** security policies are expressed against *application tiers and categories* (e.g., "web tier can talk to app tier on 443 only"), not against IP lists — policies follow VMs as they move.
- **Microsegmentation (Flow Security):** per-VM, east-west firewall enforcement — the zero-trust mechanism that stops lateral movement inside the data center (see §5.2).
- **Service insertion / flow analytics:** visibility into flows between VMs and integration with third-party security appliances.

### 5.2 Flow Security — Microsegmentation (Verified)

**Flow Security** (now branded within Flow as **Flow Network Security**) is Nutanix's microsegmentation engine: distributed, per-VM enforcement of security policies across the AHV cluster (Verified — Nutanix product page "Flow Network Security — Microsegmentation & Data Center Security"; TechTarget's comparison of "Nutanix Flow vs VMware NSX" confirms the positioning as the microsegmentation/networking answer to NSX). Key facts:

- **East-west enforcement at the vNIC:** every VM's traffic is filtered by policy as it enters/leaves the virtual NIC — no hair-pinning through a central firewall, so east-west traffic stays fast.
- **Application-aware identity:** Flow identifies applications by L7 traffic characteristics, so policies can say "database traffic only from the app tier" without static IPs — the classic three-tier zero-trust pattern (web → app → db) is a one-policy exercise.
- **Integration:** policies are managed in Prism Central; flows are visible in the Prism UI; third-party security tools can consume flow logs.
- **Honest scope note (Flagged):** Flow's security is primarily *virtual-network/VM-level* segmentation. Physical switch ACLs, and deep threat inspection (IDS/IPS, malware sandboxing), remain the domain of dedicated security stacks — Flow complements, not replaces, a NGFW/EDR estate.

### 5.3 Networking and Security — Table

| Product | Function | Notes |
|---|---|---|
| **Flow** | SDN for the Nutanix stack: overlays, VPCs, subnets, policy-based networking | VMware-NSX-style networking for AHV; managed from Prism Central |
| **Flow Security** (Flow Network Security) | Microsegmentation: per-VM east-west enforcement, L7 app identification, zero-trust policies | The NSX-T segmentation answer on AHV; complements (not replaces) NGFW/EDR |

---

## 6. The Cloud Products — Clusters, NC2 and the Hybrid Cloud

### 6.1 The Big Idea — The Same Platform in Every Cloud

The 2020s pivot (§1.4) made Nutanix's cloud story concrete: **run AOS + AHV + Prism — the exact same stack — on bare-metal servers inside the public clouds.** No re-platforming, no new skills, no new tooling: the private data center and the public cloud become one fleet. This is the "hybrid cloud" definition Nutanix sells: *hybridity through operational identity, not through a different API in every cloud.*

### 6.2 Nutanix Clusters / NC2 — Nutanix Cloud Clusters (Verified)

**Nutanix Cloud Clusters (NC2)** is the product family that runs the full Nutanix stack on public-cloud bare metal (Verified — Nutanix marketing and partner documentation: "Nutanix Cloud Clusters (NC2) runs the full Nutanix stack — AOS, AHV, and Prism — directly on dedicated bare-metal infrastructure"; Nutanix's site lists NC2 across AWS, Azure, and Google Cloud, with Google Cloud support added most recently). Availability milestones:

- **AWS** — Nutanix Clusters on AWS launched **2020** (runs on AWS bare-metal instances; supports AOS, AHV, and ESXi guest/host options) (Verified).
- **Azure** — Nutanix Clusters on Azure launched **2021** (on Azure bare-metal / M-series dedicated hosts) (Verified).
- **Google Cloud** — support for **Google Cloud** announced at .NEXT 2024/2025-era events (Reported/Verified via Nutanix announcements and partner posts; exact GA date — flag: check current NC2 documentation for region availability).

How it works: NC2 provisions dedicated bare-metal instances in the cloud, installs AOS/AHV/Prism on them, and the resulting cluster looks to Prism Central exactly like an on-prem cluster — **one Prism Central can manage on-prem and cloud clusters in the same fleet**, with the same licensing dashboards (§9).

### 6.3 What NC2 Is Actually Used For — The Use Cases

- **Disaster recovery:** replicate an on-prem cluster to an NC2 cluster in the cloud; fail over with the same DR tooling — no cloud-native re-architecture of the DR site.
- **Burst capacity:** overflow compute/storage to the cloud during peak seasons (e.g., month-end banking batch) using the same OS and tools.
- **Migration:** lift-and-shift VMs between on-prem and NC2 with vMotion-style moves (for ESXi workloads) or AHV live migration — the "escape hatch" that makes cloud adoption reversible.
- **Homogeneous multicloud:** run the identical stack in two clouds so a single operating model covers all three planes.
- **Sovereign/regulated deployments:** for banks and public-sector bodies that must keep *some* workloads on-prem but want cloud elasticity, NC2 keeps one platform story while honoring data-residency (see [cloud_providers_guide.md](cloud_providers_guide.md) §13 for the government-cloud framing, and [htx_ngine_guide.md](htx_ngine_guide.md) for the sovereign-AI mirror).

**Honest caveats (Flagged):** NC2 is not cheaper than native cloud for cloud-native apps — it buys *consistency*, not *cloud-native economics*. And NC2 runs on bare-metal instances, so instance availability and regional coverage depend on the hyperscaler's bare-metal catalog; it is a niche play relative to native IaaS in most cloud estates.

### 6.4 The Cloud Products — Table

| Product | Function | Notes |
|---|---|---|
| **Nutanix Clusters (on AWS/Azure/GCP)** | Run AOS+AHV+Prism on public-cloud bare metal | AWS 2020, Azure 2021, GCP 2022–2024-era additions (Verified/Reported) |
| **NC2 (Nutanix Cloud Clusters)** | The product family: identical Nutanix stack in public clouds, managed with on-prem Prism | DR, burst, migration, homogeneous multicloud; one operating model everywhere |
| **Hybrid cloud (the strategy)** | On-prem + public cloud as one fleet with one OS, one console, one license | The 2020s pivot; the "anywhere" story that frames the whole portfolio |

---

### 6.5 How NC2 Works in Practice

For architects, the operational mechanics of NC2 matter more than the marketing:

- **Provisioning:** you subscribe through the cloud marketplace (AWS Marketplace / Azure Marketplace / GCP), which provisions **dedicated bare-metal instances** (no noisy neighbors — the instance is yours), installs AOS/AHV/Prism, and joins the cluster to your Prism Central. From Prism Central's perspective the cloud cluster is just another cluster in the fleet.
- **Networking:** the Nutanix stack runs its overlay networking (Flow) on top of the cloud VPC — VMs get Nutanix-style networking inside the cloud, and you connect the cluster to the rest of the cloud account via standard VPC peering/routing.
- **Licensing:** consumed as **metered, per-node-hour** subscriptions through the cloud marketplace — the cloud billing model, but for the Nutanix software (cross-ref §9).
- **Mobility:** VMs move between on-prem and cloud clusters through the platform's replication/failover tooling; for ESXi-on-Nutanix deployments, host-based migration tools work the same way they do between on-prem clusters (Flagged: verify the current migration feature matrix — some paths require network connectivity between sites).
- **Termination:** when the DR test ends or the burst window closes, you can tear the cluster down and stop paying — the operational difference from buying DR hardware that sits idle 11 months a year. This is the economic argument that wins budget approval in most bank DR discussions.
- **Caveats (Flagged):** bare-metal instance types are a limited catalog (generation, region, availability); latency between on-prem and the NC2 region matters for replication; and NC2 does not give you cloud-native services (no native S3, no serverless) — it is the *same Nutanix platform*, deliberately.

---

## 7. Desktop and Other Products — Frame, Beam, Calm, Mine

### 7.1 Frame — DaaS, Acquired 2018 (Verified — Including the 2023 Divestment)

**Nutanix Frame** is a Desktop-as-a-Service (DaaS) and application-streaming platform: end users get a full Windows desktop or app delivered from the cloud (or on-prem) through a browser, on any device (Verified — Nutanix's August 2018 announcement: "With Frame, Nutanix customers will be able to deliver desktops-as-a-service (DaaS) from multiple clouds"). Timeline:

- **Announced 2 August 2018** and **closed ~October 2018** — Nutanix acquired Frame, Inc. (formerly Mainframe2) (Verified).
- **Positioning:** Frame brought a "web-scale, consumer-grade" DaaS that could run on AWS, Azure, Google Cloud or on-prem Nutanix infrastructure — at the time a differentiator versus Citrix/VMware Horizon's traditional VDI delivery.
- **2023 divestment (Verified, important correction to the "still-a-Nutanix-product" assumption):** in **June 2023, Nutanix sold Frame to Dizzion** (Verified via Dizzion's June 2023 acquisition announcement). Frame's Nutanix era ended; DaaS is no longer a Nutanix product line. This guide includes it for completeness and because many older Nutanix overviews still list it.

### 7.2 Beam — FinOps / Multi-Cloud Cost Governance (Verified)

**Nutanix Beam** (historically **Xi Beam**) is Nutanix's FinOps / cost-governance service: it ingests billing data from AWS, Azure, GCP, and the Nutanix private cloud, and provides spend visibility, right-sizing recommendations, anomaly detection, and policy-driven cost controls (Verified — Nutanix's own materials: "Beam is a multi-cloud cost optimization service… to help organizations gain visibility into cloud spend across multiple cloud environments"; third-party listings note it is now positioned inside **Nutanix Cloud Manager (NCM) Cost Governance**). Key facts:

- **One cost pane for hybrid estates:** on-prem Nutanix clusters get cost-metering too (what does a VM really cost me on-prem?) — the same question FinOps asks of public clouds ([finops_guide.md](finops_guide.md)).
- **Automation:** scheduled rightsizing, idle-resource detection, budget alerts, and one-click optimizations.
- **Licensing note:** Beam is a SaaS-delivered service with its own subscription, separate from AOS core licensing.

### 7.3 Calm — Multi-Cloud Application Automation (Verified)

**Nutanix Calm** (launched at .NEXT 2017, now inside Nutanix Cloud Manager) is the application-automation / orchestration layer: **blueprint-driven provisioning of multi-VM application stacks** (web + app + db tiers as one deployable blueprint), with lifecycle operations (start/stop/scale/upgrade) and integrations with Ansible, Terraform, and CI/CD pipelines (Verified — Nutanix docs and launch coverage describe Calm as multi-cloud application automation with blueprint-based orchestration). Key facts:

- **Blueprints, not scripts:** an app topology with dependencies and day-2 operations is captured once and deployed on AHV, ESXi, or cloud clusters.
- **Marketplace:** a marketplace of prebuilt blueprints (DBs, middleware, app servers) ships with Prism Central.
- **Relationship to NCM:** Calm, Flow, and Beam are the three services Nutanix packages under **Nutanix Cloud Manager (NCM)** — the "cloud console" for the private/hybrid estate.

### 7.4 Mine — Integrated Backup with HYCU (Verified)

**Nutanix Mine** (launched ~2018) is Nutanix's integrated backup-and-recovery solution: a **turnkey backup appliance built from a Nutanix cluster pre-loaded with backup software from HYCU** (Verified — Nutanix launched Mine with HYCU as the primary backup engine; HYCU's own materials document "Nutanix Mine with HYCU"). Key facts:

- **The pitch:** no separate backup-infrastructure project — backup storage, compute for backup jobs, and the backup software arrive as one integrated product, managed from Prism.
- **Capabilities:** application-consistent backups for VMs, databases (SQL Server, Oracle, PostgreSQL…), file shares, and objects; replication of backups to secondary sites or cloud; **ransomware protection** (immutable/WORM backups, air-gap options) — a major selling point for banks.
- **Ecosystem:** Mine also supports other backup ISVs (Veeam, Commvault, Veritas) in its broader definition as "backup target of choice"; the HYCU edition is the signature one.
- **Positioning note (Flagged as analyst view):** Mine competes with dedicated backup appliances and with cloud backup services; its win is operational simplicity for Nutanix-centric estates, not raw backup-feature breadth versus Veeam on general storage.

### 7.5 Other Notable Products (Brief)

- **Nutanix Kubernetes Engine (NKE)** — Kubernetes distribution for AHV (formerly Karbon): managed K8s clusters on the Nutanix platform, positioning Nutanix against OpenShift in the private-cloud container race (cross-ref [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md)).
- **Nutanix Cloud Manager (NCM)** — the umbrella SaaS service bundling Calm + Flow + Beam + governance (see above).
- **Nutanix Central / cloud services** — SaaS-hosted management and licensing telemetry (Pulse, Central), and the SaaS control plane behind Prism Central's cloud mode.
- **Xi services (historical)** — the earlier Xi-branded SaaS products (Xi LEAP DR, Xi Frame, Xi Beam); most were folded into NCM/NC2 branding as the platform consolidated.

### 7.6 Desktop and Other Products — Table

| Product | Function | Notes |
|---|---|---|
| **Frame** | DaaS / application streaming | Acquired 2018 (Verified); **divested to Dizzion June 2023** (Verified) — historical product |
| **Beam** | FinOps: multi-cloud + private-cloud cost visibility, rightsizing, budgets | Now part of NCM Cost Governance; SaaS subscription |
| **Calm** | Blueprint-based application automation and orchestration | Part of NCM; blueprints + marketplace + Terraform/Ansible integration |
| **Mine** | Integrated backup appliance (HYCU-powered) with ransomware protection | Turnkey backup for Nutanix estates; also supports Veeam/Commvault/Veritas as backup targets |
| **NKE / Karbon** | Managed Kubernetes on AHV | Container-platform play vs OpenShift (cross-ref k8s guides) |
| **NCM** | Umbrella cloud-manager SaaS (Calm + Flow + Beam + governance) | The "cloud console" for the private/hybrid estate |

---

## 8. The AI Products — GPT-in-a-Box

### 8.1 GPT-in-a-Box — The On-Prem Generative-AI Stack (Verified, 2023)

**Nutanix GPT-in-a-Box** was announced on **15 August 2023** (Verified — Nutanix press release: "Nutanix Simplifies Adoption of Generative AI with New Nutanix GPT-in-a-Box Solution", Aug 15, 2023; storage-review coverage the same day). It is Nutanix's turnkey, on-premises stack for running generative-AI / LLM workloads with data staying inside the enterprise. Key facts:

- **What's in the box:** an opinionated, validated reference architecture combining **NVIDIA GPUs (A100/L40S-class)**, the **Nutanix Cloud Platform (AOS/AHV/Prism)** as the foundation, **Nutanix Files + Objects** as the data and model store, and the **model-serving/foundation-model software layer** (NVIDIA NIM/NeMo-style tooling, open-source models like Llama 2/3, vector databases for RAG).
- **The pitch:** enterprises get "AI-ready infrastructure" without assembling GPU servers, storage, and MLOps tooling themselves — and with *data sovereignty* (models fine-tuned and served on-prem, not in a public-cloud AI API).
- **Positioning:** squarely aimed at regulated industries (financial services, healthcare, government) that cannot send data to public AI services — the same sovereignty logic as Singapore's HTX NGINE, where Nutanix is the platform layer ([htx_ngine_guide.md](htx_ngine_guide.md) §6.3 covers the Google/Nutanix role there).
- **Ecosystem:** certified against NVIDIA AI Enterprise; supports RAG with vector databases; pairs with NDB (§4) for the operational data feeding AI.
- **Honest scope note (Flagged):** GPT-in-a-Box is an *infrastructure-and-starter-software* solution — it provides the platform and validated stack, not a finished enterprise AI platform with your models, guardrails, and governance. Teams still build the application layer; for that layer, see [on_prem_llm_deployment_guide.md](ai_llm/on_prem_llm_deployment_guide.md) and the enterprise-AI-platform references in the `ai_llm/` series.
- **Evolution:** since 2023 the offering has expanded (new GPU generations, NIM integration, cloud options) — flag: check the current GPT-in-a-Box datasheet for today's exact stack.

### 8.2 The AI Products — Table

| Product | Function | Notes |
|---|---|---|
| **GPT-in-a-Box** | Turnkey on-prem GenAI stack: NVIDIA GPUs + Nutanix platform + model serving + RAG tooling | Announced 15 Aug 2023 (Verified); the sovereignty play for regulated AI; tied to the HTX NGINE pattern ([htx_ngine_guide.md](htx_ngine_guide.md)) |
| **NDB for AI / data layer** | Operational database services feeding AI workloads | Pairs with GPT-in-a-Box; DBaaS on the same platform (§4) |

---

### 8.3 The AI Portfolio Beyond the Box

GPT-in-a-Box is the flagship, but the AI story draws on the whole platform:

- **NVIDIA partnership:** GPT-in-a-Box is certified against NVIDIA AI Enterprise; NVIDIA NIM (inference microservices) and NeMo (model tooling) run on the stack, and Nutanix is a named platform partner in sovereign-AI builds such as Singapore's HTX NGINE (ST Engineering + NVIDIA + Nutanix + Google — see [htx_ngine_guide.md](htx_ngine_guide.md) §6.3 for the division of roles).
- **The data layer is the differentiator:** the same cluster that serves VMs provides the *data* for AI — Files for datasets/checkpoints, Objects for training corpora and model artifacts, NDB for operational data feeding RAG pipelines. "AI-ready infrastructure" in Nutanix's telling is mostly *"your data is already here, under the same protection and governance."*
- **RAG support:** GPT-in-a-Box reference architectures include vector databases (e.g., Milvus/Qdrant-class) running on the platform for retrieval-augmented generation — the pattern covered in depth by the `ai_llm/rag/` guides in this repo.
- **Evolving stack (Flagged):** since the 2023 launch, the offering has tracked the GPU generation curve (A100 → L40S → newer) and cloud options. Check the current GPT-in-a-Box datasheet for today's exact component list; the *positioning* (turnkey, sovereign, on-prem GenAI) has been stable.
- **Honest boundary (Flagged):** as noted in §8.1, GPT-in-a-Box is infrastructure-plus-starter-tooling, not a finished MLOps platform. Enterprises still build model governance, evaluation, and serving pipelines on top — [on_prem_llm_deployment_guide.md](ai_llm/on_prem_llm_deployment_guide.md) is the playbook for that layer.

---

## 9. Licensing — The Subscription Model

### 9.1 From Perpetual Appliances to 100% Subscription (Verified)

Nutanix's licensing is the **subscription/term model** — a deliberate, completed transition from the original perpetual-license-plus-appliance model (Verified: Nutanix's licensing documentation describes term-based, capacity-licensed subscriptions managed through the Nutanix portal and Prism Central; the company's earnings narrative of the 2019–2021 era was explicitly the move to "100% software, subscription-first"). Key mechanics:

- **What you subscribe to:** software capacity, expressed as **CPU cores** (compute) and **flash/TiB capacity** (storage), per the licensing guide — "Each license stores the currently licensed capacity (CPU cores/Flash TiBs)" (Verified, Nutanix portal licensing docs).
- **Term lengths:** 1-year, 3-year, or 5-year subscriptions, with ELA (enterprise license agreement) options for large estates; support and upgrades (SSP — Software Support and Subscription... historically "L4/L5") ride the subscription.
- **How it's managed:** Prism Central hosts the licensing console; the Nutanix portal manages entitlements; cloud-marketplace consumption (NC2) is metered per node/hour on AWS/Azure/GCP.
- **The base SKU:** **Nutanix Cloud Infrastructure (NCI)** is the base software stack (compute + storage + hypervisor + networking + resilience) — "a complete software stack to unify your hybrid cloud infrastructure including compute, storage and network, hypervisors and containers" (Verified, Nutanix Cloud Platform software-options page). **Nutanix Unified Storage (NUS)** adds the file/object/block capacity licenses; **NDB, Flow (security), NCM (Calm/Beam)** are separate subscription services; **GPT-in-a-Box** adds the GPU/software bundle.
- **Capacity model nuance (Flagged as complexity warning):** mixing core-based (AOS) and capacity-based (Files/Objects/NUS per-TiB) licenses across a hybrid fleet is a common source of confusion in real deals — align licensing capacity with hardware sizing at contract time, and re-check at renewal, because adding nodes changes core counts.

### 9.2 What the Model Means Commercially

- **For the customer:** predictable opex, no big-bang capex refresh, hardware-vendor independence (buy servers from any OEM), and the ability to flex capacity at renewal.
- **For Nutanix:** recurring revenue (ARR), which is why the market values NTNX on SaaS-like metrics; the subscription transition is why the stock's story changed from "HCI vendor" to "software platform company" (Reported, earnings commentary).
- **The honest comparison (Flagged):** versus VMware (vSphere+vSAN+NSX licensing stacks), Nutanix's single-vendor subscription is usually simpler; versus pure cloud, it is opex either way but you keep the hardware choice. Versus Scale Computing's perpetual-plus-support model, Nutanix is subscription-only — a real difference for cost-sensitive mid-market buyers (§10).

### 9.3 Licensing — Table

| License / SKU | What It Covers | Typical Metric |
|---|---|---|
| **NCI** (Nutanix Cloud Infrastructure) | Base platform: AOS + AHV + Prism + core data services, resilience, DR basics | Per CPU core + per flash TiB |
| **NUS** (Nutanix Unified Storage) | Files + Objects + Volumes capacity | Per TiB of file/object/block capacity |
| **NDB** | Database-as-a-Service automation | Per database engine/instance tier (and/or core-based; flag: check current SKU) |
| **Flow / Flow Security** | SDN + microsegmentation | Per core (flag: check current SKU) |
| **NCM (Calm, Beam)** | Automation, FinOps, governance SaaS | Per subscription seat/node (flag: check current SKU) |
| **GPT-in-a-Box** | AI stack: GPU platform + model software | Bundle / per GPU node |
| **NC2 (cloud)** | AOS/AHV/Prism in AWS/Azure/GCP | Metered per node-hour on cloud marketplaces |

---

### 9.4 Sizing the License — A Worked Example

Directional arithmetic so the licensing model is concrete (all numbers are illustrative — Flagged: actual pricing/SKUs come from Nutanix or your partner):

- **The estate:** 2 DCs × 8 nodes, each node 2 × 16-core CPUs, ~7 TiB usable flash per node (the §11 bank).
- **Compute license:** 8 nodes × 32 cores = **256 cores per cluster** → 512 cores across both DCs. Core licenses are typically sold in shelf increments (e.g., 10-core packs), so budget for ~520 licensed cores.
- **Storage license (NUS/NCI):** 8 nodes × 7 TiB = 56 TiB per cluster; with snapshots and headroom, license ~60–70 TiB per cluster — again in TiB increments (Flagged: increment sizes vary by SKU generation).
- **Services:** NDB per database-engine tier (SQL Server + Oracle + PostgreSQL = 3 tiers); Flow/security per core (often bundled in NCI or an add-on pack); Mine/backup per capacity; NCM (Calm/Beam) per SaaS subscription.
- **The rule of thumb:** platform (NCI) dominates the license cost; storage add-ons (NUS) and services each add meaningful but smaller lines; the *hypervisor* adds **zero** — AHV is included, which is the line that makes the total land below a vSphere+vSAN stack of the same size (Reported pattern, §10.2).
- **Renewal planning:** 3-year terms are the sweet spot for banks (capital planning cycles); negotiate ELA-level discounts at year-2–3 when the estate has proven out; and reconcile licensed vs actual cores/TiB at every renewal — under-licensed estates get console warnings, over-licensed estates waste budget.

---

## 10. Comparison — Nutanix vs the Alternatives

### 10.1 The Competitive Landscape

Nutanix's HCI core competes in a field it largely created. The three most relevant alternatives for an architect evaluating the Nutanix Cloud Platform (plus the public-cloud option covered in [cloud_providers_guide.md](cloud_providers_guide.md)):

- **VMware vSAN** — now Broadcom/VMware: hypervisor-coupled HCI (vSAN runs *inside* ESXi), sold as part of vSphere/vCF stacks. The incumbent's answer, strong where the estate is already VMware.
- **HPE SimpliVity** — HPE's HCI (acquired from SimpliVity in 2017): appliance-style, KVM/ESXi-based OmniStack, famous for built-in dedupe/compression on every write.
- **Scale Computing HC3** — the mid-market/edge HCI: simple, low-cost, self-healing, no separate management VM, targeted at distributed sites and SMBs rather than large enterprise fleets.

### 10.2 Nutanix vs VMware vSAN

- **Architecture:** vSAN is a storage layer *inside* the hypervisor; Nutanix is a full platform with its own hypervisor (AHV) plus storage as a first-class distributed OS. Nutanix's storage is hypervisor-agnostic in the sense that AOS runs beneath ESXi, AHV, or (via NC2) cloud bare metal; vSAN only exists where ESXi exists.
- **Licensing economics (the big one):** a VMware HCI stack needs vSphere + vSAN (+ NSX for the networking that Nutanix bundles via Flow), licensed per CPU. Nutanix's AHV is included free, so the all-in Nutanix price typically undercuts a vSAN stack of equivalent size (Reported — consistently the pattern in public comparisons; exact numbers vary by deal and discount).
- **Feature parity:** vSAN's maturity in large VM fleets and its deep vSphere ecosystem (DRS, vMotion, every third-party integration) remain advantages in VMware-centric shops; Nutanix counters with one-platform simplicity, Files/Objects/NDB integration, and AHV's zero license cost.
- **2020s twist:** Broadcom's post-2023 VMware licensing changes (per-core moves, bundle restructures) pushed many VMware customers to *evaluate* alternatives — Nutanix's "move off vSphere onto AHV" migration program is explicitly aimed at that wave (Reported; treat vendor claims as directional).
- **Microsegmentation comparison:** Flow vs NSX is the well-documented head-to-head (TechTarget); Flow wins on simplicity-for-AHV estates, NSX wins on breadth in VMware worlds.

### 10.3 Nutanix vs HPE SimpliVity

- **Model:** both are appliance-centric HCI, but Nutanix is software-first (any server, subscription) while SimpliVity is HPE-hardware-tied (OmniStack on HPE servers, though SimpliVity software also runs on some HPE platforms).
- **Differentiators:** SimpliVity's always-on dedupe/compression is its signature (efficient for VDI and backup-heavy loads); Nutanix counters with protocol breadth (block+file+object on one platform), the wider cloud story (NC2), and the services portfolio (NDB, Flow, GPT-in-a-Box).
- **Ecosystem:** SimpliVity rides the HPE support/hardware channel — attractive to HPE-centric accounts; Nutanix's OEM partnerships (Dell, Lenovo, Fujitsu, Cisco…) give more server choice.
- **Analyst view (Flagged as directional):** Enterprise Storage Forum-style comparisons generally rate Nutanix higher on platform breadth and innovation cadence, SimpliVity higher on simplicity-of-purchase for HPE shops and on dedupe efficiency for capacity-heavy workloads.

### 10.4 Nutanix vs Scale Computing HC3

- **Target market:** Scale HC3 is aimed at mid-market, edge, and distributed sites (retail branches, schools, SMBs) — simplicity and price first; Nutanix is aimed at enterprise and regulated workloads with a full services portfolio.
- **Pricing model:** HC3 historically sells with perpetual-plus-support pricing (Flagged: Scale's model has evolved; check current terms) versus Nutanix's subscription-only — a genuine difference for cost-sensitive buyers.
- **Capabilities:** HC3 has no equivalent of NDB, Flow microsegmentation depth, NC2, or GPT-in-a-Box; its management is deliberately minimal. Nutanix is overkill for a 3-node branch cluster; HC3 is the pragmatic choice there.
- **The honest read:** they rarely compete head-to-head in the same deal. When they do (mid-market general workloads), Scale wins on price/simplicity and Nutanix wins on features, roadmap, and enterprise credibility (Peerspot-style user reviews echo this pattern — Reported).

### 10.5 Comparison — Table

| Vendor / Product | Model | Licensing | Strengths | Weaknesses |
|---|---|---|---|---|
| **Nutanix (AOS/AHV/Prism)** | Software-defined HCI + platform services; any hardware | Subscription (cores + TiB), AHV free | Platform breadth (block/file/object/DB/net/AI), hybrid-multicloud (NC2), single console, no vSphere tax | Subscription-only; AHV ecosystem thinner than vSphere; premium price vs mid-market |
| **VMware vSAN** | Hypervisor-coupled HCI (inside ESXi) | Per-CPU vSphere + vSAN (+NSX); Broadcom-era bundles | Deep vSphere maturity, ecosystem, enterprise ubiquity, skills | License cost stack, hypervisor lock-in, licensing changes post-Broadcom |
| **HPE SimpliVity** | Appliance HCI (OmniStack on HPE) | Appliance + support (perpetual/term options) | Always-on dedupe/compression, HPE channel/support, simple purchase | Hardware-tied, narrower platform/services story, HPE-centric |
| **Scale Computing HC3** | Simple self-healing HCI | Historically perpetual + support (Flagged: verify current) | Lowest complexity/cost, edge & mid-market fit, minimal ops | Limited services (no NDB/NC2/AI depth), smaller ecosystem, enterprise-feature gaps |
| **Public cloud (context)** | Native IaaS (AWS/Azure/GCP) | Consumption-based | Elasticity, innovation velocity | Opex at scale, no same-OS private option; see [cloud_providers_guide.md](cloud_providers_guide.md) |

---

### 10.6 Decision Framework — When Nutanix, When Not

A practical checklist for the architect (opinionated, directional):

**Choose Nutanix when…**

- Your VMware renewal has become a cost problem (Broadcom-era licensing) and you can absorb a hypervisor migration.
- You want to consolidate storage silos (SAN + NAS + object + backup) onto one platform with one ops team.
- You need a *reversible* cloud story — DR/burst in AWS/Azure/GCP without rewriting applications.
- You run databases at scale and want DBaaS-style operations (NDB) without a proprietary database.
- You are regulated and want on-prem AI (GPT-in-a-Box) or sovereign-AI platforms (the HTX NGINE pattern).
- Your team values one-vendor operational simplicity over best-of-breed per layer.

**Choose an alternative when…**

- The estate is deeply VMware (custom DRS rules, NSX in production, third-party tooling tied to vSphere) — the migration cost exceeds the license savings.
- You are an HPE-centric shop wanting one hardware vendor and one support chain (SimpliVity).
- You need ultra-low-cost, low-ops clusters at the edge or in mid-market branches (Scale HC3).
- Your strategy is cloud-native-first: containers and serverless in the public cloud — Nutanix is a platform, not a substitute for native cloud ([cloud_providers_guide.md](cloud_providers_guide.md)).
- You need extreme niche storage performance or features that dedicated arrays still own (see §3.4).

**The one-line rule:** Nutanix wins where *consistency across environments* and *portfolio breadth* beat per-component best-of-breed — which is most regulated-enterprise data centers, and exactly why the mid-size-bank worked example in §11 resolves the way it does.

---

## 11. Worked Example — A Mid-Size Bank's HCI Deployment

### 11.1 The Scenario (the familiar context)

**"Merlion Bank"** — a mid-size Singapore bank (think ~1,500–3,000 staff, a retail arm, private-banking arm, and an FX/treasury desk; the profile mirrors the regulated-Asian-bank context covered in the `banking/` guides). Its data-center reality in 2026:

- **Legacy estate:** two data centers with aging three-tier stacks — VMware vSphere clusters, a pair of SANs (primary + DR), an old NAS for file shares, and a backup regime running on tape-adjacent appliances. SQL Server and Oracle run the core banking and payments applications; PostgreSQL powers newer services.
- **Pressures:** (1) MAS-grade resilience expectations and audit findings around DR; (2) application teams demanding self-service and faster provisioning; (3) a cloud mandate from the group, tempered by data-residency rules that forbid sending customer data to foreign clouds; (4) VMware/Broadcom licensing renewals that made the incumbent stack notably more expensive; (5) a data-growth curve (KYC documents, statements, surveillance recordings) that the SAN/NAS model can't absorb cheaply.
- **The decision:** replace the three-tier core with **Nutanix HCI**, keep a measured cloud footprint via **NC2** for DR/burst, and standardize file + backup + database services on the same platform. The board-level driver is the *familiar* one: modernize the data center without losing control of data, skills, or budget to a single hyperscaler.

### 11.2 The Design — AOS + AHV + Prism + Files + NDB

**The stack (what gets deployed):**

| Layer | Choice | Rationale in this bank |
|---|---|---|
| **Hypervisor** | **AHV** (KVM) — *not* ESXi | Kills the vSphere license line; one-click upgrades with AOS; the migration team retrains on AHV (2–4 weeks) rather than paying VMware tax forever |
| **OS** | **AOS** (distributed storage + resilience) | RF2/RF3 replication across nodes, snapshots, async DR replication to the second DC; self-healing re-replication removes the SAN outage class |
| **Management** | **Prism Element** per DC + **Prism Central** for the fleet | One console for both DCs; capacity planning, health scores, RBAC for MAS-aligned change control; self-service projects for app teams (with approvals) |
| **File** | **Nutanix Files** | Replaces the old NAS: KYC/document shares, branch file stores, and the surveillance-recording repository; snapshots + replication to DR |
| **Block** | **Nutanix Volumes (iSCSI)** | Only for the handful of bare-metal/legacy consumers that can't be virtualized (e.g., a hardware security module-adjacent app); everything else consumes storage natively |
| **Database** | **NDB** | SQL Server (core banking) + Oracle (GL) + PostgreSQL (new services) provisioned, patched, cloned, and PITR-restored from Prism Central — DBAs reclaim weeks per quarter |
| **Backup** | **Nutanix Mine (HYCU)** | Immutable/WORM backups, ransomware-ready; replaces the aging backup appliances; backups land on the same platform (with Objects as a secondary target) |
| **Cloud** | **NC2 on AWS (DR/burst)** | Async replication from DC-A to an NC2 cluster; quarterly DR failover tests run the *same* platform in the cloud; month-end batch can burst — without re-architecting for cloud-native APIs |
| **Security** | **Flow + Flow Security** | Microsegmentation for the three-tier app patterns (web → app → db) and the payment network; zero-trust posture for MAS surveillance reviews |
| **AI (later phase)** | **GPT-in-a-Box** | Phase 2: on-prem GenAI for KYC document Q&A and customer-service copilots, data never leaving the bank (cross-ref [htx_ngine_guide.md](htx_ngine_guide.md) for the sovereign-AI pattern) |

**Sizing sketch (directional, not a quote):**

- Two DCs, each with **one production cluster of ~8 nodes** (2× 16-core CPUs, ~512 GB RAM, mixed NVMe+SSD per node) — enough for ~700–1,000 VMs per DC including the DB tier; a **smaller 4-node cluster** per DC for Files/backup-adjacent workloads, or file-server VMs hosted on the main clusters depending on license arithmetic.
- **DR model:** async replication of VMs and Files to the peer DC (RPO minutes), plus **NC2 in AWS as a third site for DR testing and peak burst** — the cloud play that satisfies the group's "cloud-first where possible" mandate without residency violations.
- **Licensing:** NCI (core + flash) for the platform, NUS for file/object capacity, NDB per engine tier, Mine/Flow as services — a 3-year subscription with hardware purchased separately from a local OEM partner (Dell/Lenovo). Budget note: the AHV-without-vSphere line item alone typically funds a meaningful share of the Nutanix subscription (Reported pattern).

### 11.3 The Lessons

1. **The platform decision is a licensing decision first.** The AHV-is-free lever is what made the whole deal pencil out against the VMware renewal; everything else (Files, NDB, Flow) is a bonus on top of the cost win. Verify your own numbers — discounts vary widely.
2. **Unified storage kills the storage-silo projects.** The bank replaced SAN + NAS + backup-appliance refresh projects with one platform. The savings were in *operations* (one console, one upgrade path, one support contract) more than in raw hardware price.
3. **DBaaS wins the DBAs, not just the CFO.** NDB's PITR and cloning turned a skeptical Oracle/SQL Server team into the project's loudest advocates — restores that took a day now take minutes.
4. **Hybrid cloud is a DR/burst story, not a re-platform story.** NC2 gave the bank cloud credibility and a real third site without rewriting the core banking apps for cloud-native APIs. That reversibility is the strategic win.
5. **Microsegmentation is a compliance artifact.** Flow Security policies mapped directly onto MAS surveillance expectations ("database only reachable from the app tier") — security that auditors can read from one console.
6. **AI comes later, on the same platform.** GPT-in-a-Box in phase 2 reused the existing cluster, Files (data), NDB (operational data), and Flow (isolation) — the "AI-ready platform" story is real *if* the foundation was built with it in mind.
7. **The honest caveats.** Team retraining (AHV, Prism) is a real cost; the subscription model means renewals are leverage points — plan the ELA conversation early; and for peak *cloud-native* workloads, native cloud remains better than NC2. The bank's success came from using Nutanix where its model fits, not everywhere.

---

### 11.4 The Phased Roadmap

| Phase | Window | Scope | Success signal |
|---|---|---|---|
| **0 — Assess & PoC** | Months 1–2 | Workload inventory, sizing, licensing model sign-off, 4-node PoC in the lab | MAS-style DR test passes on the PoC; TCO model signed by Finance |
| **1 — Pilot** | Months 3–5 | First production cluster (non-core workloads: file services, dev/test, VDI pilots) on AHV | Prism Central health green for 60 days; helpdesk tickets flat |
| **2 — Core migration** | Months 6–12 | Migrate tier-2 apps, then core banking SQL Server + GL Oracle onto AOS (AHV), wave by wave | Zero-downtime migrations; rollback plan never needed (but rehearsed) |
| **3 — Data services** | Months 9–14 | Files (KYC/records), Mine backup consolidation, NDB for all new databases | Restore drills: PITR of core DB in minutes; backup appliance retired |
| **4 — Cloud & DR** | Months 12–18 | NC2 on AWS: async replication, quarterly failover tests, month-end burst | DR failover < RTO; burst capacity delivers month-end batch on time |
| **5 — Security & AI** | Months 18–24 | Flow microsegmentation on all tier-1 flows; GPT-in-a-Box pilot for KYC document Q&A | Auditor sign-off on segmentation; AI pilot shows measurable ops savings |

### 11.5 The Risk Register (what could go wrong)

- **Migration complexity** — the #1 risk. Mitigate: wave planning, application owners on the hook, rehearse rollback before each wave, and keep ESXi on Nutanix as a transitional option (Nutanix runs ESXi fine) so the AHV migration is decoupled from the platform migration. (Flagged as standard practice, not vendor claim.)
- **Skills gap** — AHV/Prism are different from vSphere. Mitigate: certified training for the platform team, 2–4 week shadow period with the partner, and document runbooks in Prism Central's playbook feature before go-live.
- **Subscription renewal leverage** — a 3-year lock-in with annual true-ups. Mitigate: negotiate ELA early, track licensed vs used capacity monthly, and keep the competitive quote (vSAN, SimpliVity) warm — renewal pricing follows competition.
- **AHV feature gaps** — some advanced vSphere capabilities don't exist on AHV. Mitigate: validate the *specific* features your workloads use during the PoC, not the marketing feature list.
- **NC2 dependency** — DR in the cloud depends on bare-metal instance availability and region latency. Mitigate: keep the on-prem second DC as primary DR, treat NC2 as the test/burst tier, and document the failback path.
- **Regulatory surprises** — MAS-style reviews of the new architecture. Mitigate: involve internal audit in Phase 0, and use Flow's policy console as the *evidence* artifact for segmentation and change control.

---

## 12. Summary — Nutanix in One Page

**Nutanix is the HCI pioneer that became a hybrid-multicloud platform company.** Founded in 2009 by Dheeraj Pandey, Mohit Aron, and Ajeet Singh, it invented hyperconverged infrastructure — pooling compute and storage as distributed software on commodity servers — went public in 2016 (Nasdaq, +131% first day), and spent the late 2010s and the 2020s pivoting from hardware-bundled appliances to a 100%-software, subscription-first platform that runs identically in the data center and in AWS, Azure, and Google Cloud.

**The portfolio, in one breath:** the **HCI core** (AOS distributed OS + AHV KVM hypervisor, free, + Prism management plane) supports **Unified Storage** (Files for NAS, Objects for S3, Volumes for iSCSI block), **NDB** (DBaaS for SQL Server/Oracle/PostgreSQL/MySQL, formerly Era), **Flow** (SDN + Flow Security microsegmentation), **NC2** (the same stack on public-cloud bare metal for DR, burst, and migration), the management/automation services (**Calm**, **Beam**, **Mine**), and the AI layer (**GPT-in-a-Box**, 2023, on-prem GenAI with NVIDIA GPUs). Licensing is subscription-based, measured in cores and TiB, with AHV included — the cost lever that defines most deals. Frame (DaaS, 2018) was divested to Dizzion in 2023.

**Where it wins:** cost and simplicity versus vSphere/vSAN stacks; platform breadth (one platform for VMs, file, block, object, databases, networking, and AI) versus appliance competitors like SimpliVity; enterprise credibility and services depth versus mid-market HCI like Scale Computing. For a mid-size bank, the worked example shows the pattern: AHV removes the hypervisor tax, Unified Storage kills the storage-silo projects, NDB wins the DBAs, Flow satisfies the auditors, and NC2 delivers a reversible cloud story — with GPT-in-a-Box waiting for the AI phase.

**The final word:** Nutanix is **the HCI pioneer's cloud platform** — the company that defined hyperconvergence in the 2010s and spent the 2020s proving that the private data center can run with the operating model of the public cloud, one platform, one console, one license, anywhere.

---

## 13. Glossary

- **Nutanix** — The company (founded 2009, Nasdaq: NTNX) that pioneered hyperconverged infrastructure and now sells the Nutanix Cloud Platform as a subscription software stack.
- **HCI** — Hyperconverged Infrastructure: pooling compute and storage across standard x86 servers into one software-defined cluster; no separate SAN.
- **Hyperconverged infrastructure** — See HCI; the category Nutanix created, now also offered by VMware vSAN, HPE SimpliVity, Scale Computing, and others.
- **AOS** — Acropolis Operating System: Nutanix's distributed operating system; the storage fabric and platform engine of the Nutanix Cloud Platform.
- **Acropolis Operating System** — The full name of AOS; "Acropolis" was also the earlier platform brand.
- **AHV** — Acropolis Hypervisor: Nutanix's built-in KVM-based enterprise hypervisor, included free with AOS.
- **Acropolis Hypervisor** — The full name of AHV.
- **Prism** — Nutanix's management plane: Prism Element (per-cluster) and Prism Central (multi-cluster governance, self-service, marketplace).
- **Files** — Nutanix's scale-out NAS/file product (NFS/SMB), part of Unified Storage.
- **Objects** — Nutanix's S3-compatible object storage, part of Unified Storage.
- **Volumes** — Nutanix's iSCSI block-services product for external (non-AHV) consumers, part of Unified Storage.
- **Unified Storage** — Nutanix Unified Storage (NUS): block + file + object on one platform with one ops model.
- **NDB** — Nutanix Database Service: DBaaS for SQL Server, Oracle, PostgreSQL, MySQL, MariaDB.
- **Nutanix Database Service** — The full name of NDB.
- **Era** — NDB's former name (rebranded ~2021–2022).
- **Flow** — Nutanix's software-defined networking layer (overlays, VPCs, policies).
- **Flow Security** — Flow's microsegmentation engine (per-VM east-west enforcement), now also branded Flow Network Security.
- **Microsegmentation** — Per-workload network isolation and policy enforcement (zero-trust east-west security).
- **Clusters** — Nutanix Clusters: the product line running AOS/AHV/Prism on AWS/Azure/GCP bare metal.
- **NC2** — Nutanix Cloud Clusters: the product family for running the Nutanix stack in public clouds.
- **Nutanix Cloud Clusters** — The full name of NC2.
- **Frame** — Nutanix's DaaS product (acquired 2018, divested to Dizzion 2023).
- **DaaS** — Desktop-as-a-Service: desktops delivered from the cloud/browser.
- **Beam** — Nutanix's FinOps/cost-governance service (now part of NCM Cost Governance).
- **Calm** — Nutanix's blueprint-based application automation product (part of NCM).
- **Mine** — Nutanix's integrated backup solution (HYCU-powered) with ransomware protection.
- **GPT-in-a-Box** — Nutanix's turnkey on-prem generative-AI stack (announced 15 Aug 2023; NVIDIA GPUs + Nutanix platform + model serving).
- **vSAN** — VMware's hypervisor-coupled HCI storage (vSphere + vSAN stack).
- **SimpliVity** — HPE's HCI appliance line (acquired 2017; OmniStack).
- **Scale Computing** — Vendor of HC3, the mid-market/edge HCI.
- **Licensing** — Nutanix's subscription/term licensing, measured by CPU cores and flash TiB capacity.
- **Subscription** — The recurring-license model Nutanix adopted (100% software, term-based).
- **Hybrid cloud** — Running private and public cloud as one fleet with one operating model (Nutanix's NC2 story).
- **Multicloud** — Using multiple public clouds (AWS/Azure/GCP) — for Nutanix, with the same stack on each via NC2.

---

## 14. Claims Status, References and Further Reading

### 14.1 Claims Status

| Claim | Status |
|---|---|
| Founded 23 Sep 2009; founders Dheeraj Pandey, Mohit Aron, Ajeet Singh | **Verified** (multiple independent sources); the "Dheepak Bhardwaj/Binny Gill" founder list is **incorrect** — corrected in §1.2 |
| IPO 30 Sep 2016, Nasdaq (NTNX), $16 → ~$37 first day (+131%) | **Verified** (Nasdaq release, TechCrunch) |
| Era → NDB rebrand (2021–2022 era) | **Verified** (Nutanix product page; court filing "renamed…in 2022"; 2.4-release dating per exam-prep source) |
| Frame acquired 2018; divested to Dizzion June 2023 | **Verified** (Nutanix 2018 press release; Dizzion 2023 press release) |
| Clusters: AWS 2020, Azure 2021, GCP later | **Verified** (AWS/Azure launch coverage); GCP timing **Reported/Flagged** (check NC2 docs for GA details) |
| GPT-in-a-Box announced 15 Aug 2023 | **Verified** (Nutanix press release, StorageReview) |
| Beam (FinOps), Calm (automation), Mine (HYCU backup) | **Verified** (Nutanix materials; HYCU materials) |
| AOS/AHV/Prism naming and roles | **Verified** (Nutanix product documentation) |
| Subscription licensing (cores + TiB), NCI as base SKU | **Verified** (Nutanix licensing portal docs, software-options page); per-SKU details flagged |
| Customer counts (~35k+), Fortune-500 claims, market share, "AHV is free saves X%" | **Flagged** — company claims / directional, not independently audited |
| Comparative feature assessments (vs vSAN/SimpliVity/Scale; AHV vs vSphere gaps; NC2 economics) | **Flagged** — analyst/experience views, marked inline |

### 14.2 References and Further Reading

- Nutanix — product pages: AOS/AHV/Prism, Files, Objects, Volumes, Unified Storage (NUS) datasheet, NDB, Flow / Flow Network Security, GPT-in-a-Box press release (Aug 2023), Cloud Platform software-options, licensing portal docs (portal.nutanix.com).
- Nasdaq "Welcome Nutanix" release (30 Sep 2016); TechCrunch first-day coverage (30 Sep 2016).
- Nutanix press release: intent to acquire Frame (2 Aug 2018); Dizzion press release: acquisition of Frame (6 Jun 2023).
- The Nutanix Bible (nutanixbible.com) — community-maintained deep reference on the platform internals (Stargate, NDFS, AHV).
- TechTarget — "Microsegmentation networking: Nutanix Flow vs VMware NSX"; Enterprise Storage Forum — SimpliVity vs Nutanix; Peerspot/HyperConvergence.org comparison roundups (directional).
- HYCU — Nutanix Mine with HYCU ransomware/backup materials.
- Sibling guides in this repo: [htx_ngine_guide.md](htx_ngine_guide.md) (Nutanix in sovereign AI), [cloud_providers_guide.md](cloud_providers_guide.md) (hybrid/multi-cloud), [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) + [dell_objectscale_guide.md](dell_objectscale_guide.md) + [s3_architecture_guide.md](s3_architecture_guide.md) (storage alternatives), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) + [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) + [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) (K8s platform alternatives), [on_prem_llm_deployment_guide.md](ai_llm/on_prem_llm_deployment_guide.md) (on-prem AI), [finops_guide.md](finops_guide.md) (cost/licensing), and the `banking/` guides (regulated data-center modernization).

---

*End of guide. Researched and written August 2026 — Jack Liu Shurui, Solution Architect, Crédit Agricole CIB (Singapore). Verification notes are inline; anything not explicitly marked Verified should be treated as Reported or directional.*
