# CephFS and Its Alternatives: A Comprehensive Guide to Distributed and Network Filesystems

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Data Engineering / Storage — Distributed Filesystems, Kubernetes Storage, HPC, Object Storage  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

## Table of Contents

1. [Ceph Overview: The Unified Storage Platform](#1-ceph-overview-the-unified-storage-platform)
2. [CephFS in Depth](#2-cephfs-in-depth)
3. [Deployment and Operations](#3-deployment-and-operations)
4. [Alternatives — Network Filesystems](#4-alternatives--network-filesystems)
5. [Alternatives — Modern and POSIX-on-Object](#5-alternatives--modern-and-posix-on-object)
6. [The Comparison Framework](#6-the-comparison-framework)
7. [Worked Example: Shared Filesystem for a Kubernetes Platform](#7-worked-example-shared-filesystem-for-a-kubernetes-platform)
8. [The Future (2026+)](#8-the-future-2026)
9. [Glossary](#9-glossary)

---

## 1. Ceph Overview: The Unified Storage Platform

Ceph is the most important open-source distributed storage platform you are likely to encounter in an enterprise infrastructure decision. It is the rare system that genuinely delivers on the "one storage platform for everything" promise: the same cluster serves **object** (S3/Swift), **block** (RBD), and **file** (CephFS) workloads from a single pool of commodity servers and disks, managed by a single set of daemons and a single command line. This guide is the dedicated deep-dive on **CephFS** — the file interface of Ceph — and the wider landscape of distributed/network filesystems it competes with (NFS, GlusterFS, Lustre, BeeGFS, JuiceFS, object storage, cloud filesystems), ending with a selection framework and a worked Kubernetes example.

This guide deliberately **cross-references rather than duplicates** the sibling guides in this repository:

- **Object storage (S3) side:** [cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) (S3 semantics, lakehouse patterns), [storagegrid_guide.md](storagegrid_guide.md) (NetApp StorageGRID), [dell_objectscale_guide.md](dell_objectscale_guide.md) (Dell ObjectScale). Ceph's RADOS Gateway is an S3-compatible object store and is compared against those products there; here we only summarize the object side as it relates to CephFS.
- **Local/Linux filesystems:** [ephemeral_filesystem_linux.md](ephemeral_filesystem_linux.md) covers tmpfs/overlayfs and the local, host-scoped filesystem world. CephFS is the opposite end of the spectrum: a *shared, network-accessible* filesystem.
- **Kubernetes context:** [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) and [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) cover the platforms on which CephFS is most commonly consumed today (via Rook and CSI).

### 1.1 What Ceph Is

Ceph is a **free, open-source, software-defined storage platform** that runs on commodity hardware (standard servers with HDD/SSD/NVMe and Ethernet) and presents three storage interfaces from one cluster:

| Interface | Component | Protocol / Access | Typical Use |
|---|---|---|---|
| **Object** | RADOS Gateway (RGW) | S3 and Swift APIs (HTTP) | Application buckets, backup targets, data lakes (see the object-storage guides) |
| **Block** | RBD (RADOS Block Device) | Kernel/`librbd` block device, QEMU, iSCSI | Virtual machine disks (OpenStack, Proxmox), Kubernetes RWO volumes |
| **File** | CephFS | POSIX (kernel client, FUSE, libcephfs), NFSv4 export | Shared/ReadWriteMany filesystems, home directories, HPC-lite |

The three interfaces share the same underlying storage engine — there is no separate "file cluster" and "object cluster." This unification is Ceph's core value proposition and its main differentiator against every alternative in this guide: **one set of servers, one set of daemons, one operational model, three protocols.** For example, a bank running OpenStack (RBD for VMs), a data lake (RGW/S3), and shared application filesystems (CephFS) can run all three on a single Ceph cluster.

**Key open-source facts (verified August 2026):** Ceph is developed in the open by a large community (core contributors from Red Hat/IBM, SUSE, Canonical, and many independents), stewarded by the **Ceph Foundation** (a directed fund under the Linux Foundation, launched November 2018 with 30+ founding members). It is licensed under **LGPL-2.1** (libraries) and LGPL-3.0 for core daemons. Commercial support and distributions: Red Hat Ceph Storage (now part of OpenShift Data Foundation), SUSE Enterprise Storage, Proxmox VE (bundled Ceph), Canonical Charmed Ceph.

### 1.2 Ceph Architecture: RADOS and Friends

Ceph's architecture is unusual and worth understanding before any comparison, because almost every strength and weakness of CephFS traces back to it.

**RADOS — the Reliable Autonomic Distributed Object Store — is the foundation.** Everything in Ceph (RGW objects, RBD images, CephFS file data *and* metadata) is ultimately stored as objects in RADOS. RADOS provides:

- **Self-managing placement** — no central index or lookup table; placement is computed, not looked up (see CRUSH below).
- **Self-healing replication** — replicas or erasure-coded shards are maintained automatically; failed OSDs trigger background recovery/backfill.
- **Dynamic rebalancing** — when OSDs are added or removed, data migrates automatically.
- **Failure domains** — placement rules can spread replicas across hosts, racks, or datacenters.

**The daemons:**

- **OSD (Object Storage Daemon)** — one per disk (or in some setups per device). The OSD serves object I/O, replicates data to peer OSDs, and participates in recovery, scrubbing, and rebalancing. Since Ceph 12 (Luminous), OSDs use **BlueStore**, Ceph's native storage backend that writes directly to raw devices (no dependency on an underlying local filesystem such as XFS). BlueStore delivers substantially better performance than the legacy FileStore backend. OSDs are where the capacity and the I/O performance of the cluster live: **more OSDs = more bandwidth and IOPS.**
- **MON (Monitor)** — a small cluster (odd number, typically 3 or 5) that maintains the authoritative **cluster map** (which OSDs exist, their state, pool configuration, CRUSH map) and the **PG (placement group) maps**. Monitors reach consensus using **Paxos** and form a quorum; they are *not* in the data path — client data flows directly to OSDs, which is why Ceph can scale out without a metadata bottleneck for data. If monitors lose quorum, the cluster stops serving (data safety over availability), which is why monitor placement and maintenance are sensitive operations.
- **MGR (Manager)** — introduced in Luminous (12.x), the manager daemon hosts the **Ceph Dashboard** (web UI), the Prometheus metrics exporter, the balancer module (automatic PG/OSD rebalancing), and many other mgr modules (e.g., the NFS module that manages CephFS NFS exports, the iSCSI module, the Rook integration). Run two MGRs for high availability; they are lightweight and not in the data path.

**CRUSH — Controlled Replication Under Scalable Hashing — is the placement algorithm.** Given an object ID, a pool's replica/EC configuration, and the CRUSH map (the hierarchical description of the cluster topology: hosts → racks → rows → datacenters), CRUSH computes *deterministically* which OSDs hold that object's replicas. There is **no central index, no lookup table, no metadata server for data placement** — any client can compute placement locally from the cluster map. This is the property that lets Ceph scale to exabytes without a single point of failure in the data path. Placement is governed by **pools** (logical namespaces with replication factor, EC profile, PG count, and CRUSH rulesets) and **placement groups (PGs)** — intermediate shards that map objects to OSDs (objects → PGs via hash, PGs → OSDs via CRUSH).

**Data protection: replicas and erasure coding.**

- **Replicated pools** (default): 2x or 3x copies (3x typical for production). Simple, low CPU overhead, high read performance; capacity cost = replication factor.
- **Erasure-coded (EC) pools**: k+m coding (e.g., 4+2, 8+3) — data is sharded into k chunks plus m parity chunks spread across OSDs. Much better capacity efficiency (e.g., 4+2 uses 1.5x overhead vs 3x for 3-replica) at the cost of CPU for coding and more complex partial-write handling. CephFS historically required replicated pools for *metadata* and recommended replicated pools for *data* (EC data pools for CephFS exist since later releases but with caveats around small writes and the need for a replicated "EC overwrite" capable setup); RGW and RBD are the main EC consumers.

**How clients talk to the cluster:** clients (kernel, FUSE, librados/librbd/libcephfs) contact the monitors to fetch the current cluster map, then talk **directly to OSDs** over the network (TCP, with msgr v2 encryption/auth since Nautilus; RDMA/NVMeoF support arriving in Tentacle, see §8). This direct client-to-OSD path is the reason Ceph throughput scales roughly linearly with OSD count.

### 1.3 Ceph History

| Year | Milestone |
|---|---|
| ~2001–2004 | **Sage Weil** begins Ceph as his doctoral research at UCSC (University of California, Santa Cruz); the name comes from the Greek for "squid" — hence the sea-creature release names |
| 2006 | First public open-source release of Ceph (the CRUSH paper and the early object store are the academic origin) |
| 2007–2011 | Ceph incubates at **DreamHost** (Weil co-founded DreamHost), where it is used for real production storage |
| 2012 | **Inktank** founded by Sage Weil to provide commercial support/services for Ceph |
| 2014 | **Red Hat acquires Inktank** (April 2014), making Ceph the storage engine of Red Hat's software-defined storage strategy; Ceph becomes the default for Red Hat OpenStack/OpenShift storage |
| 2015–2017 | Ceph adopted as default storage for **OpenStack** (RBD), **Proxmox VE**, and many telco/cloud deployments; BlueStore introduced in Luminous (12, 2017) |
| 2018 | **Ceph Foundation** launched under the **Linux Foundation** (Nov 2018), 30+ founding members; Intel's Lustre business sold to DDN in the same year (see §4.4) |
| 2019–2021 | cephadm (containerized deployment) introduced (Octopus, 2020) and made the default; Pacific (16, 2021); Ceph becomes the storage engine of **OpenShift Data Foundation** |
| 2023–2024 | **Reef (18.x, Aug 2023)** and **Squid (19.x, Aug 2024)** releases; Red Hat Gluster Storage reaches end of life (Dec 31, 2024), cementing Ceph as Red Hat's only scale-out filesystem story (see §4.3) |
| 2025 | **Tentacle (20.x)** released November 18, 2025 — the current stable series (latest point release 20.2.x at time of writing); introduces NVMe-over-Fabrics (NVMeoF) support, a major step for Ceph's high-performance/AI story |
| 2026+ | Next named release expected in 2026 (name not yet announced at time of writing — see §8.1) |

**Releases and cadence — verified and worth flagging:** the widely repeated "two releases per year" claim is *historical*. Ceph's early years did produce ~2 major releases per year (e.g., 2015: Hammer + Infernalis; 2016: Jewel; 2017: Kraken + Luminous). Since 2019 the project has settled into **roughly one major named release per year**, each with frequent monthly point releases (bugfix/security backports). The current series (verified August 2026):

- **Reef — 18.x** (Aug 2023): still receiving backports; widely deployed in conservative enterprises.
- **Squid — 19.x** (Aug 2024): the previous stable; default in Ubuntu 24.04-era deployments; 19.2.x point releases current.
- **Tentacle — 20.x** (Nov 2025): **current stable** (20.2.x). Notable: NVMeoF, improved scrub, continued CephFS work.
- **Next release:** expected during 2026; name unannounced at time of writing (Ceph names releases alphabetically after sea creatures — the 2026 release will follow Tentacle in that sequence; the name was not yet public when this guide was written, so treat any specific 2026 name as unverified).

Each series receives ~2 years of bugfix backports; the project supports upgrades from the previous two major releases (N → N+1 rolling upgrade supported, N-2 supported for some paths).

### 1.4 Where Ceph Sits in the Storage Landscape

Ceph competes in three adjacent markets simultaneously:

1. **Software-defined storage (SDS)** for private/edge clouds — against NetApp, Dell PowerStore/PowerFlex, VMware vSAN, StorPool, etc.
2. **Network/parallel filesystems** — CephFS against NFS, GlusterFS, Lustre, BeeGFS (the subject of this guide).
3. **Object storage** — RGW against MinIO, Cloudian, NetApp StorageGRID, Dell ObjectScale, and the hyperscaler object services (see the sibling object-storage guides).

The rest of this guide focuses on #2: the file interface and the distributed/network filesystem landscape.

---

## 2. CephFS in Depth

### 2.1 What CephFS Is

**CephFS is the POSIX-compliant distributed filesystem of Ceph** — the "file interface to the Ceph cluster." It presents a single, coherent, hierarchical namespace that can be mounted by thousands of clients concurrently, with file data striped across the RADOS object store and metadata managed by dedicated metadata servers (MDS). CephFS is a *general-purpose* shared filesystem: not a specialized HPC parallel filesystem (that is Lustre/BeeGFS territory, §4.4–4.5), but far more scalable and highly available than a classic NFS server.

CephFS history in brief: designed by Sage Weil in the original Ceph papers (2006–2007) as the "POSIX filesystem on RADOS" proof case; reached production quality in the **Luminous (12.x, 2017)** era after a re-architected metadata layer (the "new" MDS with dynamic subtree partitioning replacing the original path-based hashing); has been the default filesystem of OpenShift Data Foundation, Proxmox, and many OpenStack Manila deployments since. CephFS is fully open source (LGPL) and is the file component of every Ceph distribution.

### 2.2 CephFS Architecture

CephFS has three logical parts:

**1. The Metadata Server (MDS).** The MDS is the brain of the filesystem. It maintains the namespace (directories, files, inodes, permissions, timestamps) in a distributed, cached fashion and coordinates metadata operations (lookup, create, rename, chmod, open, readdir) between clients:

- **Metadata is stored in RADOS** (a dedicated replicated metadata pool) — the MDS is *not* the store of record; it is a **cache with journaling** over the durable metadata in RADOS. This is why an MDS can crash and fail over without losing the namespace.
- **Dynamic subtree partitioning:** the MDS splits the directory tree into subtrees and distributes them among active MDS daemons. "Dynamic" means the partition boundaries move at runtime based on measured load — a hot directory subtree can be split, migrated, or replicated to balance metadata work across MDS daemons. This is the mechanism behind CephFS metadata scale-out.
- **Multi-MDS (active-active):** each CephFS is created with one active MDS by default, but you can run **multiple active MDS daemons** (e.g., 2–8+ for large namespaces); they partition the tree and share the metadata workload. Additional MDS daemons run as **standby** or **standby-replay** (standby-replay tails the active MDS journal so failover takes seconds, not minutes).
- MDS memory sizing matters: metadata cache limits (e.g., `mds_cache_memory_limit`) govern how much namespace fits in RAM; large namespaces with many millions of files want multiple active MDS and generous RAM.

**2. The data path — files as RADOS objects.** File *data* is striped across the data pool as RADOS objects (default object size 4 MiB, stripe unit 4 MiB — configurable per file/directory via file layouts). Clients read/write file data **directly to/from OSDs**; the MDS is involved only for metadata operations (open, lookup, size updates, locks). This separation is why CephFS throughput scales with OSD count while metadata operations scale with MDS count — the two scale dimensions are independent.

**3. The clients.** Three production client types:

- **Kernel client** — the in-tree Linux kernel CephFS client (`mount -t ceph ...`). The most widely used, best-performing client; supports the v2 (msgr2, encrypted) protocol, multiple active MDS, snapshots, quotas, and (with recent kernels) most CephFS features. Note: the kernel client is feature-coupled to the kernel version — older kernels may lack newer CephFS features, so matching kernel and cluster versions matters.
- **FUSE client (`ceph-fuse`)** — userspace client; useful when you cannot or do not want a kernel client (containers without privileged mounts, exotic kernels, older kernels needing newer features). Slightly higher overhead than the kernel client.
- **libcephfs** — the userspace CephFS library (C/C++, with Python bindings); used by NFS-Ganesha (FSAL_CEPH) for the CephFS NFS gateway, by the ceph-fuse daemon, and by applications embedding the filesystem directly.

### 2.3 CephFS Features

**POSIX semantics.** CephFS aims for POSIX compliance: hierarchical namespace, hard links, symlinks, file locking (POSIX/BSD/OFD locks, fcntl), permissions (mode bits, ACLs — POSIX ACLs via `setfacl`), rename semantics, mmap, sparse files, extended attributes. It is the most POSIX-complete of the scale-out filesystems in this guide (Lustre and BeeGFS are "POSIX-flavored" rather than fully POSIX). Caveats exist (see §2.6): some POSIX edge cases (e.g., strict readdir semantics across the distributed cache, `O_DIRECT` semantics) behave slightly differently than a local filesystem, and full POSIX is only guaranteed among CephFS-native clients — NFS-exported access inherits NFS's weaker semantics.

**Dynamic namespace / subtree partitioning.** As described above, the namespace is partitioned dynamically across active MDS daemons, giving *metadata* scale-out — the feature that puts CephFS in a different class from a single-NFS-server design and from GlusterFS's metadata-less design. For very large single-directory workloads (one directory with millions of entries), CephFS still serializes on that directory's inode — a known limitation shared with most POSIX filesystems.

**Snapshots.** CephFS supports filesystem snapshots (since Luminous): any directory can be snapshotted (`mkdir .snap/name`), producing an instant, efficient, read-only point-in-time view of the subtree. Snapshots are implemented by copy-on-write metadata and do not require a full data copy; they are the basis of **CephFS mirroring** (snap-scheduled, asynchronous replication to a peer cluster, since Pacific) used for disaster recovery.

**Quotas.** Directory-level quotas (since Luminous): max bytes and max file count can be set on any directory (`ceph.fs.quota.max_bytes`, `ceph.fs.quota.max_files` extended attributes) and are enforced by clients/MDS. Quotas are *per-directory* (they apply to the subtree), not per-user globally — a common misconception and a real gap vs. NFS server-side quotas in some deployments.

**Multi-MDS.** See §2.2 — active-active MDS for metadata scale-out, standby/standby-replay for fast failover. The practical envelope: 1 active MDS handles roughly 10–50k metadata ops/sec depending on hardware and workload; adding active MDS scales aggregate metadata throughput until the metadata pool (RADOS) becomes the limit. Subtree partitioning is automatic but *sensitive to namespace structure*: workloads with a few giant directories partition poorly.

**NFS export (CephFS NFS gateway).** CephFS can be exported to NFSv4 clients via an **NFS-Ganesha** gateway (the `ceph nfs` mgr module): Ganesha uses FSAL_CEPH (libcephfs) to mount CephFS and re-exports it as NFSv4.1. Multiple Ganesha nodes behind a virtual IP give HA NFS. This is the standard answer to "we have legacy NFS clients" — they get NFSv4, the cluster gets Ceph's scale and HA (details and comparison in §4.2). Note the NFS export is an *access protocol translation*, not a separate storage system.

**High availability.** Everything in the CephFS path is HA by design: MDS failover is automatic (standby promotes in seconds; standby-replay reduces this to ~seconds with journal replay); OSD failures trigger recovery without filesystem downtime (degraded mode while replicas rebuild); monitors are a Paxos quorum; clients tolerate MDS/OSD churn via capability-based caching and journal replay. Planned maintenance (rolling upgrades, MDS restarts) is supported without unmounting clients.

**Performance characteristics.** Verified public benchmarking and Red Hat/IBM guidance support these qualitative (not vendor-promised) expectations:

- **Sequential throughput:** scales with OSD count and network. A well-built all-NVMe/25GbE cluster delivers multi-GB/s per client stream and tens of GB/s aggregate; HDD-based clusters deliver hundreds of MB/s per client with aggregate limited by OSD count. CephFS does not hit Lustre-class single-stream bandwidth (Lustre can saturate 100GbE per client with one file), but it scales out across many clients.
- **Metadata operations:** roughly 10–50k ops/s per active MDS (mdtest-style benchmarks on NVMe/RAM-sized caches); multi-MDS multiplies this. Latency for metadata ops is a few ms on a healthy cluster — noticeably higher than local disk, fine for shared workloads, wrong for latency-critical single-node apps.
- **Small files:** CephFS small-file performance is *adequate but not stellar* — each small file costs an object allocation plus metadata ops; mixed workloads with millions of small files want NVMe OSDs, tuned pools (e.g., `bulk` vs `ssd` classes), and enough MDS memory. This is the most common CephFS performance complaint and the main reason some teams choose JuiceFS's object-backed cache or a parallel filesystem for small-file-heavy HPC.

### 2.4 CephFS Use Cases

- **Shared application filesystems (ReadWriteMany).** The dominant modern use case: multiple application instances (especially Kubernetes pods) sharing one filesystem with POSIX semantics. CephFS CSI (`cephfs.csi.ceph.com`) provides RWX volumes natively — the "shared volume" class that RBD (RWO block) cannot provide. This is the sweet spot vs. NFS (scale, HA) and vs. object (POSIX semantics).
- **Kubernetes / OpenShift platform storage.** Rook-Ceph or OpenShift Data Foundation (ODF) exposes CephFS StorageClasses for RWX PVCs (logs, shared caches, model weights, multi-replica apps). See §3 and §7 for the worked example.
- **HPC/analytics (moderate scale).** CephFS is used for analytics clusters, grid/edge HPC, and research shared storage where the scale is "hundreds of clients, tens of TB to a few PB" rather than "exascale." It lacks Lustre's raw single-stream bandwidth but offers snapshots, quotas, and much simpler operations. OpenStack Manila's CephFS driver made it the default shared FS for private clouds.
- **Home directories.** CephFS is a strong home-directory backend (the original use case in the Ceph papers): POSIX semantics, quotas per home dir, snapshots for backup, multi-site mirroring.
- **Media/content.** Shared media stores, render farms, and content repositories where file semantics, ACLs, and scale-out capacity matter; RGW serves the same content as object where S3 semantics suffice.
- **Machine learning data platforms (growing).** Ceph is increasingly positioned as the data platform for GPU clusters: CephFS for shared training datasets/checkpoints, RGW for object-first pipelines, NVMeoF/RDMA in Tentacle for low latency (see §8).

### 2.5 CephFS vs. Its Own Siblings (RBD, RGW)

A frequent design question: *CephFS or RBD or RGW for my workload?*

- **RBD** if the consumer is a VM/container that wants a *block device* and does not need multiple writers: RBD is Ceph's highest-performance interface (kernel-level block, thin provisioning, snapshots, cloning, mirroring). One writer only (or clustered filesystems like XFS-on-RBD-on-Multiple-Nodes which are not recommended).
- **RGW (S3)** if the semantics are object: HTTP access, buckets, no POSIX needed, cross-cluster multi-site replication, EC-friendly. See the object-storage guides for the full object discussion.
- **CephFS** if you need *shared, POSIX, multiple writers* — the only interface that provides it.

### 2.6 CephFS Limitations (Honest Assessment)

- **Not a true parallel HPC filesystem.** Single-stream throughput per client is lower than Lustre/BeeGFS on the same hardware; there is no client-side data caching (all I/O goes to the network), so latency-sensitive or single-node-banded workloads suffer. Choose Lustre/BeeGFS for extreme bandwidth; choose CephFS for manageability and feature completeness.
- **Small-file / metadata performance.** Millions of tiny files stress MDS memory and RADOS object overhead. Mitigations exist (NVMe OSDs, multiple MDS, `fast` pool classes, tuning `mds_cache_memory_limit`) but small-file-heavy is not CephFS's home turf.
- **Operational complexity.** Ceph is a distributed system with real operational requirements: monitor quorum discipline, network design (no lossy/oversubscribed fabric), PG/pool planning, capacity headroom for recovery, and upgrade discipline. It is *not* "NFS on two servers." Teams without storage ops experience should use a managed distribution (Rook/ODF, Proxmox, SUSE) or a vendor (Red Hat/IBM, Canonical).
- **POSIX caveats.** Full POSIX only among native clients; NFS-exported access carries NFS semantics; some edge semantics differ from local filesystems; Windows clients need SMB gateways (Samba/CTDB on CephFS is possible but adds complexity — RBD+SMB is the more common Windows path).
- **Quotas are directory-scoped, not user-scoped.** No global per-user quota enforcement out of the box (can be scripted via dir quotas + auth caps, but it is not a feature).
- **Namespace-shape sensitivity.** Multi-MDS partitioning works best with balanced trees; a few gigantic directories bottleneck metadata ops on single inodes.
- **No multi-writer same-file byte-range locking beyond POSIX locks** — e.g., no HDFS-style append; concurrent writers to the same file rely on normal POSIX locking discipline.

---

## 3. Deployment and Operations

### 3.1 Deployment Options

**cephadm — the modern, containerized deployment (recommended for bare-metal/VM clusters).** Since Octopus (2020), cephadm is Ceph's standard deployment and management tool (replacing the older ceph-deploy and ceph-ansible). cephadm deploys every Ceph daemon as a **container** managed by systemd, over SSH, from a bootstrap node:

```bash
# 1. Bootstrap a single-node cluster (creates the first MON + MGR + a host entry)
cephadm bootstrap --mon-ip 10.10.10.10

# 2. Add more hosts (from the bootstrap node, using the SSH key cephadm created)
cephadm shell -- ceph orch host add node2 10.10.10.11
cephadm shell -- ceph orch host add node3 10.10.10.12

# 3. Deploy OSDs on all available devices across the cluster
cephadm shell -- ceph orch apply osd --all-available-devices

# 4. Deploy the CephFS metadata servers
cephadm shell -- ceph orch apply mds cephfs

# 5. Create the filesystem (metadata pool + data pool)
cephadm shell -- ceph fs new cephfs cephfs_metadata cephfs_data
```

Highlights: declarative **service specs** (YAML describing MON/OSD/MDS/RGW/NFS placements), automatic daemon restart/upgrade orchestration (`ceph orch upgrade`), the mgr-based orchestration layer (`ceph orch ...` commands), and a built-in HA for MONs/MGRs. cephadm is the path used by upstream docs, SUSE, Canonical, and most bare-metal enterprises.

**Rook — Ceph on Kubernetes (recommended when the platform is Kubernetes).** Rook is a CNCF project (graduated) that runs Ceph *inside* a Kubernetes cluster as the **Rook operator**: you declare a `CephCluster` custom resource, and the operator deploys and manages MONs, MGRs, OSDs (from node devices or PVCs), MDS, RGW, and the CSI drivers. The Ceph provider in Rook is **stable** (verified 2025–2026: current Rook releases, 1.x series, support Ceph Reef/Squid/Tentacle). Rook is the engine under OpenShift Data Foundation (ODF) and the standard for self-managed Kubernetes storage. Details and YAML in §7. Note: Rook's CephFS and RBD CSI drivers are enabled by default; the NFS (Ganesha) driver is disabled by default.

**Vendor distributions:** Red Hat Ceph Storage / OpenShift Data Foundation (Rook-based), SUSE Enterprise Storage (cephadm-based), Canonical Charmed Ceph (Juju operator), Proxmox VE (bundled Ceph with web UI), and managed services (e.g., IBM/SoftLayer, OVH, Scaleway offer Ceph-backed services). For a bank, ODF or a supported vendor distribution is the usual entry point.

### 3.2 Cluster Setup Essentials

- **Monitors (MON):** 3 (or 5) small-footprint nodes/VMs in different failure domains; 1 GiB RAM + small disk each is enough. Never deploy an odd→even monitor count by accident (quorum = majority).
- **Managers (MGR):** 2 for HA (active/standby). Host the dashboard and metrics.
- **OSDs:** one OSD per physical device (BlueStore); mix HDD (capacity), SSD/NVMe (performance), and optional `device classes` (e.g., `hdd`, `ssd`, `nvme`) with CRUSH rules to build tiered pools (fast data pool + bulk pool). NVMe OSDs are the norm for CephFS performance workloads. Leave ~15–20% capacity headroom for recovery.
- **Networks:** separate public (client) and cluster (OSD-to-OSD replication) networks at 10/25/100 GbE; the cluster network absorbs replication/backfill traffic. Ceph is unforgiving of packet loss on the cluster network.
- **Pools and PGs:** create pools with a sensible PG count (PGs per OSD ≈ 50–100 for replicated pools at scale; the `pg_autoscaler` mgr module manages this automatically since Pacific); use `bulk` flag for large data pools. For CephFS: a replicated **metadata pool** (3x, small, fast, `bulk` not set) and a **data pool** (replicated 3x, or EC 4+2 for capacity efficiency if workloads are large-object; note EC data pools for CephFS require the `ec_overwrites` capability and careful small-write consideration).
- **Filesystem creation:** `ceph fs new <name> <metadata_pool> <data_pool>` (optionally `--force` when reusing pools); then `ceph orch apply mds <name>` (cephadm) or the Rook `CephFilesystem` CR. `ceph fs status` shows active MDS and partitions.

### 3.3 Clients

**Kernel mount (Linux):**

```bash
# Using a client keyring; mon IPs/hostnames from the cluster
mount -t ceph mon1:6789,mon2:6789,mon3:6789:/ /mnt/cephfs -o name=admin,secretfile=/etc/ceph/admin.secret
```

(msgr2: use port 3300 or let the client auto-negotiate; modern kernels negotiate v2 automatically.)

**FUSE mount:**

```bash
ceph-fuse -m mon1:6789,mon2:6789 /mnt/cephfs   # as root, or with -k for a keyring
```

**Kubernetes CSI:** the `cephfs.csi.ceph.com` provisioner creates volumes dynamically from a `StorageClass` (see §7). For OpenShift/ODF, StorageClasses are pre-provisioned (`ocs-storagecluster-cephfs`).

### 3.4 Operations

- **Monitoring:** the Ceph Dashboard (mgr module) provides cluster health, capacity, performance graphs, and a Prometheus endpoint (`<mgr>:9283/metrics`); standard practice is Prometheus + Grafana with the `ceph-exporter`/node exporters and alerting on `ceph health` states (`HEALTH_WARN`, `HEALTH_ERR`), OSD down counts, PG states, and MDS status. `ceph -s` / `ceph health detail` are the first things to teach operators.
- **Capacity management:** `ceph df detail` (pool usage, quotas), pool autoscaling, OSD fullness warnings (`nearfull`/`backfillfull`/`full` thresholds), and `ceph osd rebalance` via the balancer module. CephFS quotas (max_bytes per directory) are the per-tree control; pool quotas are the safety net.
- **Failure handling:** OSD down → clients keep reading/writing degraded (replicas elsewhere); recovery/backfill runs in background (`ceph -w` to watch); MON quorum loss → cluster read-only or down (rare, but the reason monitor nodes are conservative); MDS failover is automatic. The dashboard's "Performance" and "OSD" pages plus `ceph health detail` are the troubleshooting entry points. Scrubbing (deep scrub for bit-rot detection) runs periodically.
- **Performance tuning:** network (jumbo frames, 25/100GbE, separate cluster network), NVMe OSDs, `mds_cache_memory_limit` sizing, multiple active MDS, pool/CRUSH layout (`crush_rule` mapping data to NVMe class), `bluestore` cache sizing, client-side options (`mount -o` readahead, `rsize/wsize`), and (in Tentacle) NVMeoF for direct high-performance access. Benchmark with `rados bench` (RADOS), `rbd bench`, and `fio`/`mdtest` on CephFS before blaming the filesystem.
- **Upgrades:** rolling, per-daemon-type upgrades via `ceph orch upgrade` (cephadm) or Rook's upgrade orchestration; always read the release notes (e.g., the known iSCSI upgrade bug in early Squid 19.2.0, tracker 68215 — a good reminder that point releases matter); the community advises letting a new stable series bake for a few point releases before production adoption.
- **Multi-site / DR:** RGW has mature **multi-site** (active-active bucket replication, the object DR story — see the object guides); **RBD mirroring** (journal/async) for block DR; **CephFS mirroring** (snapshot-based, async, per-directory, since Pacific) for file DR. Multi-site is typically "separate clusters + replication," not stretched single clusters (stretched clusters exist for RBD via `stretch mode` but are advanced).

---

## 4. Alternatives — Network Filesystems

This section covers the classic network/parallel filesystem alternatives. The modern POSIX-on-object and cloud-managed alternatives follow in §5. A note on framing: these are *alternatives in the shared-filesystem decision*, not necessarily competitors in every situation — each occupies a distinct zone of the scale/ops/semantics trade-off space, and §6 formalizes the selection framework.

### 4.1 NFS — The De Facto Standard

**What it is.** The Network File System is the oldest and most universal shared-filesystem protocol: a client-server protocol where one (or a small cluster of) NFS server(s) export directories from their local filesystems to many clients over the network. NFS is a *protocol and an ecosystem*, not a distributed storage system — the server's local filesystem does the real storage work.

**Protocol generations:**

- **NFSv3** (1995): stateless (server crashes are survivable by design), UDP/TCP, 32-bit file handles, no locking in the protocol itself (lock manager is separate), weak security (AUTH_SYS). Still everywhere because it is simple and fast for basic workloads.
- **NFSv4** (2000): stateful (open/close/lock state on the server, with lease-based recovery), compound operations (fewer round trips), integrated locking and mount, ACLs, stronger security (Kerberos/RPCSEC_GSS), Unicode names. Single protocol for mount+lock+stat.
- **NFSv4.1** (2010): **sessions** (exactly-once semantics, better crash recovery), and **pNFS** (parallel NFS) — a *layout* mechanism where the metadata server hands out data layouts so clients can talk directly to multiple data servers in parallel. pNFS is the protocol-level answer to scale-out NFS, but real pNFS implementations (NetApp ONTAP, some others) are the exception, not the rule.
- **NFSv4.2** (2016): server-side copy (copy_file_range), sparse-file support, space reservation, clone.

**Server implementations.** The Linux kernel NFS server (knfsd, free, ubiquitous, good for modest scale), NetApp ONTAP (the enterprise NFS gold standard — FlexVol, snapshots, dedupe, multi-protocol SMB/NFS), Dell EMC (PowerScale/OneFS, PowerStore, PowerMax NFS), IBM Storage Scale (GPFS), and cloud-managed NFS (EFS, Filestore, Azure Files — §5.4).

**NFS vs CephFS — the honest comparison:**

| Dimension | NFS (classic) | CephFS |
|---|---|---|
| Architecture | One server (or active/passive pair) exporting local FS | Fully distributed: data across OSDs, metadata across MDS |
| Scale | ~hundreds of clients / tens of TB on a big box | Thousands of clients / PB+ |
| HA | VIP failover, CTDB/Samba clusters, hardware HA — complexity grows with scale | Built-in: no single point of failure |
| Consistency | Server-centric, cache-coherency via delegation (v4) | Distributed cache with capability-based coherence |
| Ops | Trivial (one box) to moderate (clustered) | Real distributed-system ops (§3) |
| Performance | Excellent single-stream (local disk + network), limited aggregate | Scales with OSDs; lower per-client latency ceiling |
| Features | Snapshots depend on the server FS (NetApp: excellent; Linux: LVM/btrfs tricks) | Native snapshots, quotas, multi-site mirroring |

**The verdict on NFS:** NFS wins on **simplicity and ubiquity** — every OS, every cloud, every tool speaks NFS. CephFS wins on **scale, HA, and feature depth**. The modern architectural pattern is not "NFS vs CephFS" but **"NFS as a front-end to Ceph"** — see below. In the bank context: legacy app servers and mainframe-adjacent systems that only speak NFS keep NFS; greenfield scale-out shared storage goes to CephFS or a managed equivalent.

**NFS on Ceph (the CephFS NFS gateway).** Ceph ships an **NFS-Ganesha-based NFS gateway** managed by the `ceph nfs` mgr module: Ganesha's FSAL_CEPH mounts CephFS via libcephfs and exports it as NFSv4(.1) to legacy clients. HA is achieved by running multiple Ganesha instances behind a virtual IP (and optionally CTDB). This gives you "NFS in, Ceph underneath" — legacy clients, modern storage:

```bash
ceph nfs cluster create mynfs <node1> <node2>        # HA Ganesha cluster on hosts
ceph nfs export create cephfs mynfs /export/path cephfs \
    --client_addr 10.0.0.0/8 --squash none           # export a CephFS path as NFSv4
```

Rook also supports the NFS gateway (NFS CSI driver, disabled by default). This is the recommended answer to "our app only does NFS."

### 4.2 GlusterFS — The Declining General-Purpose Alternative

**History (verified).** GlusterFS began as **Gluster** (originally "Z Research"), founded ~2005 in Bangalore/Chennai, funded by Nexus Venture Partners and Index Ventures; **Red Hat acquired Gluster in October 2011** and built the Red Hat Gluster Storage (RHGS) product on it. Through the mid-2010s GlusterFS was Red Hat's scale-out file story and a popular alternative to CephFS for simple shared storage. Then Red Hat's strategic storage bet shifted to Ceph: **Red Hat deprecated GlusterFS in favor of OpenShift Data Foundation (ODF), whose engine is Ceph; Red Hat Gluster Storage reached end of life on December 31, 2024.** The upstream Gluster community continues (gluster.org releases: 11.x in 2023, 12.x in 2024–2025) with much-reduced corporate investment — effectively **community maintenance mode**. If you are evaluating GlusterFS *today* (2026), the honest position is: it is a mature, simple, working filesystem with a shrinking ecosystem and no major vendor roadmap; new deployments should strongly prefer CephFS (same vendor lineage, actively developed) or the alternatives below. Migration from GlusterFS to CephFS is a well-trodden path (rsync/robocopy-style data migration; ODF even ships guidance).

**Architecture.** GlusterFS is deliberately **metadata-less**:

- **Bricks** — plain directories exported from servers (each server contributes its local filesystem; no special disk format).
- **Volumes** — logical filesystems composed of bricks with **translators (Xlators)** in a stack: **distributed** (file-level sharding across bricks), **replicated** (mirror copies), **striped** (data striping — legacy/discouraged), **distributed-replicated** (the common production form: groups of replica pairs), plus **arbiter** volumes (a lightweight third copy holding only metadata+checksums for split-brain prevention) and **tiering** (hot/cold).
- **No central metadata server** — file placement is computed by **elastic hashing (DHT)** of filenames; each brick can serve any lookup directly. This is GlusterFS's simplicity win (nothing to fail, no metadata bottleneck) and its weakness: DHT has well-known pathologies (rename = remote file migration, directory hot spots, balancing churn, no efficient directory-wide enumeration across bricks, and consistency challenges around heals).

**GlusterFS vs CephFS:**

| Dimension | GlusterFS | CephFS |
|---|---|---|
| Metadata | None (DHT) — simpler, but weaker semantics | MDS with subtree partitioning — stronger, more scalable |
| Consistency | Eventual-ish; self-heal daemon reconciles replicas | Stronger POSIX-coherent via capability/cache model |
| Scale | Hundreds of nodes fine; large-directory and rename workloads hurt | Scales metadata AND data independently |
| Features | Snapshots (volume-level), quotas, geo-replication (rsync-based), NFS/SMB via gateways | Directory snapshots, dir quotas, CephFS mirroring, NFS gateway |
| Ops | Simpler day-1, but heals/balancing need attention | More components, richer tooling/dashboard |
| Trajectory | Maintenance mode; Red Hat EOL'd the product (Dec 2024) | Actively developed, default in Red Hat's own ODF |

**GlusterFS today.** Community releases continue; **forks/derivatives**: the codebase is stable and forks exist mostly as packaging (e.g., in Debian/Fedora); the notable ecosystem shift is *migration*: Red Hat documents GlusterFS → CephFS/ODF migration, and most Gluster deployments from the 2015–2020 era have moved or are moving. Treat GlusterFS as "legacy with a migration path," not a greenfield choice.

### 4.3 Lustre — The HPC Standard

**History (verified).** Lustre was created by **Cluster File Systems Inc.** (Peter Braam, ~2001–2003; Lustre 1.0 released December 2003, named after the Linux + cluster portmanteau). It went through **Sun Microsystems** (2007), **Oracle** (2010), and **Intel** (2013, which acquired Lustre from Oracle and drove the 2.x series, OpenSFS/Lustre 2.4+). In **June 2018, DDN (DataDirect Networks) acquired Intel's Lustre business and assets, re-forming the Whamcloud team as a DDN division** — Whamcloud (a DDN company) is the current steward of upstream Lustre (verified 2026: Whamcloud/DDN maintain releases, support matrix, and the Lustre wiki; LTS releases like 2.15.x and 2.16.x track current kernels).

**Architecture.** Lustre separates metadata from data into distinct server roles:

- **MGS/MGT** — Management Server / Management Target: cluster-wide configuration (like a tiny control plane).
- **MDS/MDT** — Metadata Server / Metadata Target: one or more (DNE — Distributed Namespace — supports multiple MDTs since 2.4/2.5) servers holding directories/inodes; metadata is journaled (ldiskfs/zfs-backed).
- **OSS/OST** — Object Storage Server / Object Storage Target: the data servers; files are **striped across OSTs** (stripe count/size configurable per file) for parallel bandwidth.
- **Clients:** a kernel module client speaking **LNet** (Lustre's own network layer, with RDMA over InfiniBand/RoCE and TCP); a large client-side page cache and **LDLM** (Lustre Distributed Lock Manager) provide aggressive caching and coherence.

**Scale.** Lustre is *the* supercomputer filesystem: the majority of **TOP500** systems run Lustre (Frontier, El Capitan, Fugaku-class machines), with deployments reaching **hundreds of PB and multi-TB/s aggregate bandwidth**; single clients can saturate 100–400 GbE/InfiniBand per stream. It is tuned for huge sequential files and massive concurrency — the opposite of CephFS's general-purpose design.

**Lustre vs CephFS:**

| Dimension | Lustre | CephFS |
|---|---|---|
| Design goal | Maximum parallel bandwidth for HPC | General-purpose distributed POSIX + unified storage |
| Data path | Stripe across OSTs, client-side caching, RDMA (LNet) | Striped RADOS objects, no client data cache, TCP/RDMA |
| Metadata | MDS/MDT, DNE multi-MDT, single-directory limits (one MDT owns a dir) | MDS + dynamic subtree partitioning |
| Single-stream BW | Highest in class (multi-GB/s per client) | Lower per client; aggregate scales with OSDs |
| POSIX | "POSIX-flavored" — good but with HPC compromises (e.g., locking semantics, no full ACLs historically) | Closer to full POSIX among native clients |
| Features | Snapshots (recent, via project quotas/`lfs snapshot`), no dir quotas historically, HSM/archive tiering, no built-in multi-site | Snapshots, dir quotas, mirroring, NFS gateway, S3/RBD in the same cluster |
| Ops | Specialist skill set (LNet, OST/MDT tuning, kernel-client/kernel-version coupling) | General storage ops; dashboard; containerized |
| Ecosystem | HPC schedulers (Slurm), parallel I/O (MPI-IO, HDF5, NetCDF), vendor appliances (DDN, Cray/HPE) | Kubernetes, OpenStack, cloud-native |

**Use cases.** HPC and scientific computing (climate, genomics, simulation), national labs, and — increasingly — **AI training clusters** (large checkpoints, dataset staging). Lustre's positioning vs CephFS: if your workload is "few huge files, massive bandwidth, Slurm cluster, specialist HPC staff," Lustre (or BeeGFS) is the right tool; if your workload is "general enterprise/Kubernetes/cloud-native shared storage," CephFS. The cloud form is **Amazon FSx for Lustre** (§5.4), which makes Lustre accessible without on-prem HPC ops.

### 4.4 BeeGFS — The Pragmatic Parallel Filesystem

**History (verified).** BeeGFS (originally **FhGFS**) was developed at the **Fraunhofer Institute for Industrial Mathematics (ITWM), Kaiserslautern, Germany, starting ~2005–2007**, with the first public release ~2007. **ThinkParQ** was spun out in **2014** to provide commercial support, development, and tools. **Acquisition status: ThinkParQ remains independent** — the persistent "NetApp acquired BeeGFS" rumor is incorrect (verified 2026): **NetApp is a global partner** (Diamond Partner since 2019) delivering BeeGFS on NetApp E-Series hardware (including the EF600), and ThinkParQ took over the NetApp-created BeeGFS CSI driver in 2023 — a partnership, not an acquisition. BeeGFS is free for unlimited use (open-source components since 2022, source-available model); ThinkParQ sells support and the commercial HA/management tooling.

**Architecture.** BeeGFS is a *parallel* filesystem with the same metadata/data separation as Lustre but a much simpler implementation:

- **Management service** (`beegfs-mgmtd`) — lightweight configuration/cluster membership; no data path.
- **Metadata services** (`beegfs-meta`) — one or more; **metadata is striped across metadata servers** (since 7.2), so a single hot directory can be served by multiple metadata nodes in parallel — a real differentiator vs. Lustre's single-MDT-per-directory and CephFS's single-inode serialization.
- **Storage services** (`beegfs-storage`) — the data servers; files striped across storage targets (stripe count/granularity configurable per directory/file).
- **Clients** — kernel module client with RDMA (InfiniBand/RoCE) and TCP; also a userspace client for containers.

**BeeGFS vs CephFS:**

| Dimension | BeeGFS | CephFS |
|---|---|---|
| Design | Parallel FS: metadata servers + storage servers, client kernel module | Unified RADOS: OSDs + MDS + RGW/RBD in one platform |
| Metadata scale | Striped metadata servers — parallel even for one directory | Subtree partitioning — serializes on huge single directories |
| Throughput | Excellent per-client and aggregate bandwidth with RDMA | Good aggregate; lower per-client ceiling |
| Resilience | Data replication (RAID1-style per target); no erasure coding; **no self-healing rebalance** in the base product — HA is "failover via replicated targets + Pacemaker" | Replication + EC, automatic recovery/rebalance, self-healing by design |
| Features | Snapshots (limited, 7.x), quotas (directory), no native multi-site; NFS export via gateway | Snapshots, quotas, mirroring, NFS gateway, S3/RBD unified |
| Consistency | POSIX-flavored; strong for HPC workloads | Closer to full POSIX |
| Ops | Simpler than Ceph (fewer moving parts); still parallel-FS ops | Richer but heavier operations (§3) |
| Maturity/community | Mature (2007), focused HPC/AI ecosystem, small but active community, ThinkParQ support | Very large community, Red Hat/IBM/SUSE/Canonical backing |

**Use cases.** HPC clusters, university/research compute, and — heavily since 2023 — **AI/ML training and inference infrastructure** (BeeGFS markets itself explicitly as "parallel file system for HPC and AI"). If you need Lustre-class bandwidth with simpler administration, or want parallel metadata for AI dataloaders hammering one dataset directory, BeeGFS is the pragmatic pick; if you want the unified platform (file+object+block, snapshots, self-healing, Kubernetes-native), CephFS.

---

## 5. Alternatives — Modern and POSIX-on-Object

### 5.1 JuiceFS — POSIX on Object Storage

**What it is.** JuiceFS is an open-source (Apache 2.0) distributed POSIX filesystem that **decouples data from metadata**: file *data* lives in object storage (S3, Alibaba OSS, Tencent COS, GCS, Azure Blob, MinIO, etc.), while *metadata* lives in a separate metadata engine. A JuiceFS client on each application node provides the filesystem interface (POSIX via FUSE, plus Hadoop/JuiceFS Java SDK, Kubernetes CSI, and an S3 gateway). It was created by **Juicedata** (Beijing; founded 2017 by former Megvii/face++ engineers), and as of 2025–2026 it is one of the fastest-growing filesystems in the cloud-native/AI space (verified: active GitHub project, tens of thousands of stars, large production user base in China and globally; an enterprise edition adds a proprietary distributed metadata service and HA tools on top of the open core). The 2025 Amazon **S3 Mountpoint** GA ("S3 as a filesystem") is a related but narrower trend (see §5.3) — JuiceFS has been doing "POSIX on object" since 2017.

**Architecture.** Three pieces:

1. **JuiceFS client** — runs on each host (FUSE, or the kernel-free approach): formats data into 4 MiB "chunks" → 64 KiB "slices" → uploaded to object storage as objects; maintains a local cache (disk + memory) for reads/writes; speaks to the metadata engine for namespace operations. Multiple clients see a **consistent, coherent namespace** because all metadata operations go through the metadata engine (strong consistency — no object-store eventual-consistency surprises).
2. **Object storage** — any S3-compatible or cloud object store holds the data objects (compressed and optionally encrypted client-side). Capacity and durability come from the object store.
3. **Metadata engine** — a key-value/SQL store: **Redis** (single or cluster — the most common community choice), **TiKV** (distributed, for larger scale), **PostgreSQL/MySQL/SQLite** (SQL engines; SQLite for single-node/tests). Enterprise uses a proprietary distributed metadata service. The metadata engine is the coherence point and the scalability knob: Redis cluster or TiKV for big namespaces; a single Redis for small/medium.

**Features (verified against JuiceFS docs):** POSIX semantics (via FUSE; file locking, permissions, rename); **snapshots** (metadata-level, cheap); **quotas** (directory/volume); **local caching** (the performance lever — hot data served from local NVMe, so read performance often *beats* direct object access and approaches local disk for cached working sets); client-side **encryption and compression**; **HDFS-compatible API** (Hadoop SDK — drop-in for Spark/Hive on object storage); **S3 gateway** (expose the filesystem as S3); multi-mount consistency; garbage collection; and a file system **trash**.

**JuiceFS vs CephFS:**

| Dimension | JuiceFS | CephFS |
|---|---|---|
| Data storage | Object storage you already have (S3/MinIO/cloud) — capacity is someone else's problem | Self-contained: OSDs you build and operate |
| Metadata | External engine (Redis/TiKV/PG/SQLite or enterprise service) | MDS + RADOS metadata pool (built-in, self-healing) |
| Deployment | Client install on each host + metadata engine + object store — small footprint, no storage cluster to run | Full Ceph cluster (MON/MGR/OSD/MDS) |
| POSIX | Strong via FUSE/SDK (single-writer cache semantics per client) | Strong via kernel client |
| Performance | Depends on cache hit rate + object store; cached reads very fast, cold reads pay object latency | Predictable scale-out; no cache dependency for correctness |
| Consistency | Strong (metadata engine is the arbiter) | Strong (capability/cache model) |
| Ops | Simple: patch clients, run Redis/TiKV, point at S3 | Heavy: cluster lifecycle, OSD care, network (§3) |
| Cost model | Pay-as-you-go capacity (object), small compute footprint | CAPEX/OPEX servers + disks + ops |
| Sweet spot | Cloud-native, Kubernetes, big data, AI on top of object storage you already pay for | On-prem unified platform, private cloud, regulated environments |

**Use cases.** Cloud-native shared filesystems (K8s RWX on EKS/GKE/ACK via CSI), big-data workloads (Spark/Hive/Hadoop on S3 with POSIX convenience), AI/ML (dataset staging with local cache, model training reading cached data), and any scenario where "I already have object storage; I want POSIX on top of it." The trade-off to weigh: you are outsourcing durability to the object store and consistency to the metadata engine — two systems to reason about instead of one — and cold-read latency is object-latency-bound unless caching is sized well.

### 5.2 MinIO and Object Storage as a "Filesystem Alternative"

Object storage (S3 semantics) is the most important *modern* alternative to a network filesystem — not because it is a filesystem, but because **most new applications are being built object-first** and no longer need POSIX at all. The sibling guides cover this in depth ([cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md), [storagegrid_guide.md](storagegrid_guide.md), [dell_objectscale_guide.md](dell_objectscale_guide.md)); here we summarize only what the filesystem decision needs.

**S3 semantics vs POSIX — the fundamental differences:**

| Operation | POSIX filesystem | Object storage (S3) |
|---|---|---|
| Model | Hierarchical tree of files/dirs, inode semantics | Flat namespace of buckets + keys (prefixes simulate dirs) |
| Write | In-place, arbitrary offset, append | Whole-object PUT; multipart upload for large objects |
| Rename | Atomic, O(1) metadata op | Copy + delete (two operations, not atomic) — the classic pain |
| Locking | POSIX locks, byte-range | Bucket/object locking (WORM), conditional writes (If-Match/None-Match) |
| Append | Native | Not native — rewrite or multipart tricks |
| Consistency | Strong | Strong (S3 since Dec 2020) for reads-after-writes; multi-region replication is eventually consistent |
| Directory listing | Efficient, ordered, recursive | Prefix scans (S3 ListObjectsV2) — paginated, no server-side recursion |
| Metadata | Inode, xattrs, permissions | User metadata headers, tags, versioning |

**The "object as a file" problem set:** for filesystem-shaped workloads, the gaps that bite are **rename (non-atomic copy+delete)**, **locking** (no POSIX flock; concurrent writers need application coordination or conditional writes), and **append** (no in-place append). That is why object storage is right for: application buckets, backups, archives, data lakes, media assets, ML datasets (read-mostly), and anything accessed via HTTP/SDKs — and wrong for: shared databases, live-edited documents, anything needing POSIX locking/rename/append.

**The middle ground — gateways and hybrids:**

- **POSIX gateways over object storage:** s3fs (fuse, weak consistency), **JuiceFS** (§5.1, the serious option), S3 Mountpoint (AWS's high-performance read/sequential-write client, GA 2025, *not* a general POSIX FS — no random writes/rename guarantees), and vendor NAS gateways (e.g., NetApp StorageGRID + NAS fabric, Dell ObjectScale with S3-only semantics).
- **Object + filesystem in one platform:** the classic case is **Ceph itself** — RGW (object) and CephFS (file) share one cluster, so you can serve the same data platform to both access models, and tier data between a CephFS "hot" tier and RGW "cold"/archive. NetApp ONTAP (NAS + S3 in one array) and Dell PowerScale (OneFS + S3) do the same commercially.
- **Filesystem → object tiering:** HSM/archival (Lustre HSM to S3, CephFS + RGW archive jobs, AWS FSx for Lustre's native S3 data repository integration) — keep POSIX hot data, spill cold data to cheap object storage.

**MinIO specifically** (the most popular self-hosted S3): MinIO is a high-performance S3-compatible object store for Kubernetes/bare metal, frequently used *under* JuiceFS, s3fs-style tooling, and data-lake stacks. It is the usual self-hosted object alternative to Ceph RGW (compare: RGW rides a full Ceph cluster; MinIO is a lightweight standalone with its own erasure coding and lifecycle features — see the object guides for the detailed comparison). In the filesystem decision, MinIO enters as "the object side" of a JuiceFS or lakehouse architecture, not as a filesystem.

### 5.3 Cloud-Managed Filesystems

For teams that do not want to operate *any* storage cluster, the hyperscalers offer fully managed shared filesystems. These are the strongest competitors to CephFS in cloud-greenfield scenarios — and they are also the reference points for what a "managed CephFS" would feel like (several vendors, e.g., IBM, offer managed Ceph-backed file services; OpenShift Data Foundation on ROSA/AKS/EKS is effectively managed CephFS).

| Service | Protocol | Type | Scale (approx., per service limits) | Typical use case |
|---|---|---|---|---|
| **AWS EFS** (Elastic File System) | NFSv4.1 | Elastic shared file (regional/AZ), auto-scaling | Petabyte-scale, thousands of NFS clients; throughput modes: bursting (~100 MiB/s base) and elastic/provisioned (multi-GiB/s) | Cloud NFS for EKS/EC2, shared app data, home dirs |
| **Azure Files** | SMB 3.x (primary), **NFSv4.1** (premium, Linux) | Managed SMB/NFS share | 100 TiB per share max (large shares), up to 100k IOPS/10 GiB/s (premium) | Windows/Linux shared storage, lift-and-shift NAS, AKS |
| **Google Filestore** | NFSv3 (basic/enterprise), NFSv3+ (high-scale SSD) | Managed NFS tiers | Enterprise: up to 10k IOPS / ~1.2 GiB/s; High-scale SSD: 100k+ IOPS, 10+ GiB/s, 100 TiB volumes | GKE shared storage, HPC-lite on GCP |
| **Amazon FSx for Lustre** | Lustre (POSIX-flavored) | Managed parallel FS (Lustre) | Up to hundreds of GB/s aggregate, PB-scale; native S3 repository integration | HPC, EDA, ML training on AWS — "Lustre without the ops" |
| **Amazon FSx for NetApp ONTAP** | NFSv3/v4, SMB, iSCSI | Managed NetApp ONTAP (FlexVol, snapshots, clones, dedupe) | Multi-TiB/s aggregate (scale-out FSx ONTAP), PB-scale | Enterprise NAS migration, multi-protocol, snapshots |
| **Azure NetApp Files** | NFSv3/v4.1, SMB | Managed NetApp ONTAP on Azure | Up to 4.5 GiB/s per volume, 1 PiB volumes, multi-PB accounts | Enterprise NAS, HPC, SAP, VDI on Azure |
| (IBM, OVH, Scaleway, …) | NFS / CephFS | Managed Ceph-backed file services | Vendor-dependent | Managed CephFS without self-ops |

**Cloud vs self-hosted — the honest framing:**

- **Managed wins** on: time-to-value (minutes vs weeks), no storage ops team required, elastic capacity billing, integrated IAM/backup/DR, and per-GB pricing that looks cheap at small scale.
- **Self-hosted CephFS wins** on: total cost at sustained scale (managed file services price per GB/TiB-month that exceeds hardware CAPEX for large, always-on workloads), data residency/regulatory control (a bank's reason #1), multi-protocol unification (file+object+block in one cluster), no egress costs, performance predictability, and avoiding cloud lock-in.
- **The hybrid pattern** that increasingly wins in enterprises: CephFS/ODF on-prem for regulated/primary data + managed file services in the cloud for elastic overflow and dev/test, with object storage as the sync/DR plane.

The rule of thumb: if your workload is already in a hyperscaler and the compliance story allows it, buy the managed service; if you are building a private cloud, a regulated platform, or a scale-out platform across clouds, build on Ceph (CephFS + RGW covers the file and object planes with one ops model).

---

## 6. The Comparison Framework

### 6.1 The Master Comparison Table

The following table condenses everything above into one decision view. "POSIX" here means how completely the system honors POSIX semantics for typical applications; "HA" means how much of the high-availability story is built-in versus assembled by the operator; "ops complexity" is day-2 operational burden.

| Dimension | CephFS | NFS (classic) | GlusterFS | Lustre | BeeGFS | JuiceFS | Object (S3/MinIO) |
|---|---|---|---|---|---|---|---|
| **Architecture** | Unified RADOS cluster (OSD+MON+MGR+MDS); also RBD/RGW | Server exports local FS (client-server) | Bricks + DHT, no metadata server | MGS/MDS/MDT + OSS/OST, parallel | Mgmtd + meta + storage services, parallel | Client + metadata engine + object store | Buckets/keys over object servers (EC) |
| **Metadata** | MDS, dynamic subtree partitioning, multi-MDS | On the server (single node) | None (elastic hashing) — simple, weaker | MDS/MDT, DNE multi-MDT | Striped metadata servers (parallel) | External engine: Redis/TiKV/PG/SQLite | None (prefix scan) |
| **Scale** | Thousands of clients, PB+ | ~Hundreds of clients, tens of TB | Hundreds of nodes, PB (with caveats) | TOP500-class, 100s of PB, TB/s | HPC-class, 10s–100s of PB | Limited by metadata engine + object store | Exabyte-class (cloud) |
| **POSIX** | High (native clients) | High (per-server) | Moderate (DHT semantics) | High-flavored (HPC compromises) | High-flavored | High via FUSE/SDK | None (object semantics) |
| **HA** | Built-in (self-healing, auto-failover) | Operator-built (VIP, clusters) | Partial (replicas, self-heal daemon) | Built-in failover, specialist ops | Replication + Pacemaker, no auto-rebalance | Depends on metadata engine HA (Redis cluster/TiKV) | Built-in durability (cloud SLA) |
| **Ecosystem** | Kubernetes (Rook/CSI), OpenStack, huge community | Universal (every OS/cloud) | Declining; OpenShift legacy | HPC (Slurm, MPI, vendors) | HPC + AI vendors (DDN, NetApp E-Series) | K8s CSI, Hadoop, AI, cloud | Everything (S3 standard) |
| **Ops complexity** | Medium–high (distributed system) | Very low | Low–medium | High (specialist) | Medium | Low–medium (two systems) | Low (managed) / medium (self-hosted) |
| **Best use cases** | K8s RWX, unified on-prem, home dirs, analytics | Legacy app shares, simple NAS | (Legacy) simple scale-out NAS | HPC, scientific, huge bandwidth | HPC, AI training | POSIX-on-object, cloud-native, big data | Lakes, backup, media, object-first apps |
| **License** | LGPL-2.1/3.0 | GPLv2 (Linux server) / protocol | GPLv2+ (community) | GPLv2 (client GPLv2) | Free source-available / proprietary tools | Apache 2.0 (community) | AGPLv3 (MinIO) / proprietary (cloud) |
| **Maturity** | Very high (2004–present, 20.x) | Very high (1985–present) | High but in maintenance (2005–present) | Very high (2003–present, TOP500) | High (2007–present) | Growing fast (2017–present) | Very high (S3, 2006–present) |

### 6.2 The Selection Framework (Decision Tree)

Use this as a fast filter; the detailed factors in §6.3 refine it.

```
Need shared/network storage?
│
├─ Workload lives in a hyperscaler & compliance allows it?
│   └─ YES → Cloud-managed: EFS (generic NFS) · Azure Files (SMB/NFS) ·
│            Filestore (GCP NFS) · FSx for Lustre (HPC) · FSx ONTAP / ANF (enterprise NAS)
│
├─ HPC / scientific / extreme bandwidth (TB/s, Slurm, MPI)?
│   └─ YES → Lustre (max bandwidth, TOP500 standard) or BeeGFS (simpler, AI-friendly)
│            · in cloud: Amazon FSx for Lustre
│
├─ Kubernetes platform needing shared ReadWriteMany volumes?
│   └─ YES → CephFS via Rook / ODF (CSI, RWX) — the default answer
│
├─ Application is object-first (SDK/HTTP, lakes, backup, media)?
│   └─ YES → Object storage: S3/MinIO/RGW (see the object-storage guides) — no POSIX needed
│
├─ Simple shared filesystem, small team, few hundred clients?
│   └─ YES → NFS (Linux knfsd, NetApp, or a cloud NFS) — simplest possible
│
├─ Already pay for object storage, need POSIX on top (cloud-native/big data/AI)?
│   └─ YES → JuiceFS (or S3 Mountpoint for read-mostly sequential access)
│
└─ On-prem / regulated / private cloud, unified file+object+block?
    └─ YES → Ceph (CephFS + RGW + RBD) — one cluster, one ops model
```

**Reading the tree honestly:** the two "default" answers in modern architectures are **CephFS for Kubernetes/on-prem shared storage** and **object storage for application data** — most new storage demand lands on one of those two. NFS survives as the compatibility layer (including "NFS in front of CephFS" via the NFS gateway), and the parallel filesystems (Lustre/BeeGFS) own the specialized HPC/AI-bandwidth niche where CephFS deliberately does not compete at the top end.

### 6.3 Decision Factors

1. **Scale and access pattern.** How many clients, how many TB/PB, and *what shape* is the I/O? Sequential huge files → parallel FS territory; many small files + metadata churn → metadata design matters (CephFS multi-MDS, BeeGFS striped metadata, or cache-centric JuiceFS); mostly read/append → object storage.
2. **Protocol requirement.** POSIX only, NFS only, SMB/Windows, S3? CephFS gives POSIX + NFS gateway + (via the cluster) S3 — the widest single-platform coverage. Windows clients point to SMB (Azure Files, ONTAP, or a Samba gateway) or object.
3. **Ops team.** Do you have storage engineers? Ceph's power is paid for in operational discipline; if the team is app-focused, prefer managed services, NFS, or JuiceFS-on-object. This factor alone decides more projects than any performance number.
4. **Performance envelope.** Bandwidth per client vs aggregate; metadata ops/s; latency sensitivity. Set concrete targets (e.g., "2 GB/s per training node, 50k metadata ops/s, <5 ms p99 metadata latency") and benchmark the shortlist — vendor tables (including this guide's) are directional, not contractual.
5. **Cost.** TCO across hardware (CAPEX), power/floor space, ops effort, and licensing, vs managed per-GB pricing at *your* sustained utilization. Ceph shines at large sustained scale; managed shines at small/elastic scale; object storage (self-hosted or cloud) is the cheapest per-GB tier.

### 6.4 Verdict: CephFS vs the Alternatives

- **CephFS is the default choice for software-defined, on-prem, Kubernetes-native shared filesystems** — the only fully open-source option that combines strong POSIX, snapshots, quotas, multi-site mirroring, an NFS gateway, and a file+object+block unified platform, with a very large community and multiple vendor distributions. If your starting point is "we need a shared filesystem and we already run Kubernetes / OpenStack / a private cloud," the burden of proof is on *not* choosing CephFS.
- **Choose against CephFS only for concrete reasons:** you need true parallel bandwidth (Lustre/BeeGFS/FSx for Lustre), you want minimal ops (NFS/managed), your data is object-shaped (S3/MinIO — see the object guides), you're already cloud-native and want POSIX over existing object storage (JuiceFS), or you're in a hyperscaler with no residency constraints (managed file services).
- **GlusterFS is no longer a greenfield option** (Red Hat EOL Dec 2024; Ceph is the successor in Red Hat's own portfolio) — treat existing Gluster deployments as migration candidates to CephFS/ODF.
- **The most common real-world architecture** is not one filesystem but a **layered platform**: object storage for the data plane (S3/RGW/MinIO), CephFS for shared POSIX/Kubernetes workloads, and NFS gateways for legacy clients — increasingly on a single Ceph cluster.

---

## 7. Worked Example: Shared Filesystem for a Kubernetes Platform

This section walks through the canonical CephFS deployment: **Rook-Ceph on a Kubernetes cluster, CephFS StorageClass, ReadWriteMany PVC, and a multi-replica application** — with cephadm notes for the non-Kubernetes case. This is the setup a platform team would run for a shared application filesystem (logs aggregation, shared model weights, multi-writer caches, document stores). For the Kubernetes platform context, see [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) and [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md).

### 7.1 Step 1 — Deploy the Ceph cluster with Rook

Install the Rook operator (namespace `rook-ceph`), then declare the cluster. A minimal production-shaped `CephCluster` (3 worker nodes, OSDs on raw devices):

```yaml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: quay.io/ceph/ceph:v20.2.2        # Tentacle series (current stable)
  dataDirHostPath: /var/lib/rook
  mon:
    count: 3                                 # odd number, spread across nodes
    allowMultiplePerNode: false
  mgr:
    count: 2                                 # HA managers (dashboard + metrics)
  dashboard:
    enabled: true
  storage:
    useAllNodes: false
    useAllDevices: false
    nodes:
      - name: "node1"
        devices: [{ name: "nvme0n1" }, { name: "nvme1n1" }]
      - name: "node2"
        devices: [{ name: "nvme0n1" }, { name: "nvme1n1" }]
      - name: "node3"
        devices: [{ name: "nvme0n1" }, { name: "nvme1n1" }]
```

Rook creates MONs, MGRs, and OSDs; verify with `kubectl -n rook-ceph get pods` and `ceph status` (`kubectl -n rook-ceph exec deploy/rook-ceph-mgr-... -- ceph -s` → expect `HEALTH_OK`).

### 7.2 Step 2 — Create the CephFS filesystem

```yaml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: cephfs-shared
  namespace: rook-ceph
spec:
  metadataPool:
    replicated:
      size: 3                                  # metadata: 3 replicas, small pool
  dataPools:
    - name: data
      replicated:
        size: 3                                # data: 3 replicas (or erasureCoded 4+2)
  preservePoolsOnDelete: true
  metadataServer:
    activeCount: 2                             # 2 active MDS (metadata scale-out)
    activeStandby: true                        # keep standby MDS for fast failover
```

This yields a CephFS named `cephfs-shared` (metadata pool `cephfs-shared-metadata`, data pool `cephfs-shared-data`) with two active MDS daemons. With cephadm (non-Kubernetes) the equivalent is: `ceph osd pool create cephfs_metadata 32`, `ceph osd pool create cephfs_data 128`, `ceph fs new cephfs cephfs_metadata cephfs_data`, `ceph orch apply mds cephfs 2`.

### 7.3 Step 3 — StorageClass for dynamic provisioning (CSI)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cephfs-shared-sc
provisioner: rook-ceph.cephfs.csi.ceph.com    # CephFS CSI provisioner (Rook default)
parameters:
  clusterID: rook-ceph
  fsName: cephfs-shared                       # the CephFilesystem created above
  pool: cephfs-shared-data                    # data pool for the volumes
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
  csi.storage.k8s.io/fuseMountOptions: []     # kernel client by default; FUSE optional
mountOptions:
  - discard
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

(On OpenShift Data Foundation the StorageClass `ocs-storagecluster-cephfs` exists out of the box — same concepts.)

### 7.4 Step 4 — ReadWriteMany PVC and a shared-volume application

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-data-pvc
spec:
  accessModes:
    - ReadWriteMany          # ← the CephFS superpower: many pods, one volume
  storageClassName: cephfs-shared-sc
  resources:
    requests:
      storage: 100Gi
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shared-app
spec:
  replicas: 3
  selector:
    matchLabels: { app: shared-app }
  template:
    metadata:
      labels: { app: shared-app }
    spec:
      containers:
        - name: app
          image: nginx:alpine            # stand-in for your application
          volumeMounts:
            - name: shared
              mountPath: /data
      volumes:
        - name: shared
          persistentVolumeClaim:
            claimName: shared-data-pvc   # all 3 replicas read/write the same 100Gi
```

All three replicas mount the same CephFS volume — write in pod A, read in pod B, immediately visible (POSIX coherence via CephFS capabilities). This is the workload pattern RBD (ReadWriteOnce block) cannot serve and the reason CephFS is the RWX answer on Kubernetes. For a StatefulSet/analysis workload with per-pod directories, simply mount subpaths (`subPath: "pod-$(POD_NAME)"` via env) — a common CephFS pattern for logs/checkpoints.

### 7.5 Performance Expectations and Sizing

- **Throughput:** with 6 NVMe OSDs (2 per node) on 25 GbE, expect roughly 2–5 GB/s aggregate sequential read/write for the filesystem (directionally; benchmark with `fio` before signing off). Aggregate scales ~linearly with OSD count; per-client streams are network-bound (~2–3 GB/s on 25 GbE).
- **Metadata:** 2 active MDS handle ~20–100k metadata ops/s combined for typical mixed workloads; bump `activeCount` for namespaces with tens of millions of files; size MDS memory generously (`mds_cache_memory_limit`, default 1 GiB → raise to 4–8 GiB per MDS for large caches).
- **Latency:** healthy cluster: ~1–5 ms metadata ops, data I/O at network speed; keep the cluster network lossless and the data pool on NVMe for interactive workloads.
- **Sizing rules of thumb:** 3 MONs on separate nodes (small VMs fine); 2 MGRs; 1 OSD per NVMe device; ~15–20% free capacity headroom for recovery; data pool 3x replicated (or EC 4+2 if capacity efficiency matters and writes are large/sequential); metadata pool 3x on fast storage; separate public/cluster networks at ≥25 GbE for performance workloads. Start small (3 nodes × 2 OSDs) and grow by adding nodes — CRUSH rebalances automatically.
- **Operational hygiene:** enable the dashboard + Prometheus/Grafana monitoring from day 1, set alerting on `HEALTH_WARN`, OSD down, and PG states; use Rook's upgrade orchestration for Ceph point releases; test failover (kill an OSD pod, kill an MDS) in staging before production.

---

## 8. The Future (2026+)

### 8.1 Ceph Roadmap

- **Releases (verified August 2026):** **Tentacle (20.x)** is current stable, released Nov 18, 2025 (20.2.x at time of writing), with **NVMe-over-Fabrics (NVMeoF)** support as its flagship — direct high-performance access paths that matter for AI/GPU clusters. The **next major release is expected in 2026; its name had not been announced when this guide was written** — treat any circulating 2026 name as unverified. Squid (19.x) and Reef (18.x) continue to receive backports; the cadence is ~one major per year with frequent point releases.
- **CephFS improvements to watch:** continued multi-MDS scaling and faster failover (standby-replay maturation), snapshot/mirroring polish, performance work for NVMe/RDMA paths, and better small-file behavior (object/OSD-level optimizations). CephFS is a "mature core, steady improvement" component — no rewrite is coming, but the edges (AI workloads, very large namespaces) are the active frontier.
- **AI/ML storage:** the project and its vendors (Red Hat/IBM's ODF, SUSE, Canonical) are explicitly positioning Ceph as **the open data platform for AI**: CephFS for shared training data and checkpoints, RGW for object-first pipelines and model registries, NVMeoF/RDMA for GPU-direct/low-latency access, and Rook/ODF for Kubernetes-native AI platforms (GPU clusters on OpenShift). Expect "Ceph for AI" to be the dominant marketing and investment theme of the 2026–2027 cycle.
- **Container-native:** Rook remains the Kubernetes standard (Ceph provider stable; CSI for RBD/CephFS enabled by default, NFS optional; ceph-csi operator maturing); expect tighter integration with K8s storage features (CSI snapshots/clone/expand, volume groups, and the new Kubernetes "storage capacity" model) and more managed-Ceph offerings from vendors.

### 8.2 Alternatives Evolution

- **GlusterFS decline is structural.** Red Hat Gluster Storage EOL (Dec 31, 2024) plus Red Hat's Ceph-centric ODF strategy means the community project is in maintenance mode; expect continued migration to CephFS/ODF and a slow fade in new deployments. Any new "Gluster vs X" decision in 2026+ is really "Ceph vs X."
- **JuiceFS growth.** POSIX-on-object is a secular trend (cloud-native + AI + "we already pay for S3"): JuiceFS, S3 Mountpoint (read-mostly), and vendor equivalents are the growth segment. Expect JuiceFS to keep expanding (enterprise metadata service, Kubernetes integration, AI data platforms).
- **HPC filesystems + AI.** Lustre and BeeGFS are riding the AI wave (training clusters, exascale-class datasets); DDN/Whamcloud (Lustre) and ThinkParQ (BeeGFS, still independent — NetApp remains a partner) are investing in AI-specific features (checkpoint offload, metadata parallelism, NVMe/GPU-direct). DAOS (Intel's object/transactional store) is a watch item at the very top of the HPC spectrum.
- **Object storage keeps absorbing filesystem workloads.** S3 is now the "system of record" for data lakes, backup, and AI datasets; filesystem vendors (including Ceph) increasingly *front* object storage rather than compete with it. The long-term shape is layered: object at the bottom, POSIX/parallel/cloud filesystems on top where semantics demand it.

### 8.3 Trends Summary

1. **Unification wins:** one platform for file+object+block (Ceph's core bet) is winning in on-prem/regulated environments where ops cost is the binding constraint.
2. **Kubernetes is the primary storage consumer:** CSI-native provisioning (CephFS RWX, RBD RWO) is now table stakes; storage decisions are made in StorageClass terms, not mount commands.
3. **AI reshapes the performance envelope:** GPU clusters want parallel bandwidth, low-latency metadata, and checkpoint resilience — pushing Ceph (NVMeoF/RDMA), Lustre/BeeGFS (AI features), and JuiceFS (caching) all in the same direction.
4. **Object is the data plane, filesystems are the access planes:** the durable data increasingly lives in object storage; POSIX/parallel/NFS layers provide semantics on demand.
5. **Managed beats self-managed unless scale or compliance says otherwise** — and for banks, compliance says otherwise often enough that CephFS/ODF remains a core platform skill.

---

## 9. Glossary

| Term | Definition |
|---|---|
| **Ceph** | Open-source distributed storage platform providing object (RGW/S3), block (RBD), and file (CephFS) interfaces over one cluster of commodity servers; LGPL-licensed, stewarded by the Ceph Foundation (Linux Foundation, 2018). |
| **RADOS** | Reliable Autonomic Distributed Object Store — Ceph's self-managing, self-healing replicated object store and the foundation under RGW, RBD, and CephFS. |
| **OSD** | Object Storage Daemon — one per device; serves object I/O, replicates data, performs recovery/scrubbing; stores data with the BlueStore backend. |
| **MON** | Monitor — small Paxos-consensus cluster maintaining the cluster map (membership, pools, CRUSH); not in the data path; quorum loss stops the cluster. |
| **MGR** | Manager — daemon hosting the dashboard, Prometheus exporter, balancer, and modules such as the NFS module; run two for HA. |
| **CRUSH** | Controlled Replication Under Scalable Hashing — deterministic placement algorithm computing OSD targets from the cluster topology map; no central index. |
| **Erasure coding** | k+m coding scheme spreading data+parity shards across OSDs (e.g., 4+2) for capacity-efficient durability; alternative to replica copies. |
| **CephFS** | The POSIX distributed filesystem of Ceph; file data in RADOS objects, metadata via MDS; mountable via kernel client, FUSE, or libcephfs. |
| **MDS** | Metadata Server — CephFS daemon managing namespace/metadata with dynamic subtree partitioning; multiple active MDS scale metadata out. |
| **FUSE** | Filesystem in Userspace — mechanism used by the `ceph-fuse` userspace CephFS client. |
| **Kernel client** | The in-tree Linux kernel CephFS client (`mount -t ceph`); the primary production client. |
| **libcephfs** | Userspace CephFS library (C/C++/Python); used by ceph-fuse and NFS-Ganesha (FSAL_CEPH). |
| **Snapshot** | Point-in-time, read-only view of a directory subtree (`.snap/`); basis of CephFS mirroring. |
| **Quota** | Directory-scoped limit (max bytes / max files) enforced by CephFS. |
| **cephadm** | Containerized Ceph deployment/orchestration tool (since Octopus): bootstrap, service specs, upgrades via `ceph orch`. |
| **Rook** | CNCF operator running Ceph in Kubernetes (CephCluster CR, CSI drivers); engine of OpenShift Data Foundation. |
| **CSI** | Container Storage Interface — Kubernetes storage plugin standard; `cephfs.csi.ceph.com` provisions CephFS volumes. |
| **NFS** | Network File System — the universal client-server file-sharing protocol (v3, v4, v4.1 with pNFS, v4.2). |
| **NFSv4 / pNFS** | NFSv4: stateful protocol with integrated locking/security; pNFS (v4.1): layout-based parallelism to multiple data servers. |
| **GlusterFS** | Metadata-less scale-out filesystem (bricks + elastic hashing); Red Hat product EOL Dec 31, 2024 — community maintenance mode. |
| **Brick / Volume / Xlator** | GlusterFS building blocks: exported directory (brick), logical filesystem of bricks (volume), translator-stack plugins (Xlator). |
| **Lustre** | The HPC parallel filesystem standard; MGS/MGT + MDS/MDT + OSS/OST; stewarded by Whamcloud (DDN). |
| **MGS / MGT** | Lustre Management Server / Target — cluster configuration. |
| **OSS / OST** | Lustre Object Storage Server / Target — the data servers; files striped across OSTs. |
| **BeeGFS** | Parallel filesystem from Fraunhofer ITWM / ThinkParQ (independent; NetApp is a partner); striped metadata, RDMA, HPC/AI focus. |
| **ThinkParQ** | Company behind BeeGFS (spun out of Fraunhofer, 2014); provides support and commercial tooling. |
| **JuiceFS** | Open-source POSIX filesystem on object storage + external metadata engine (Redis/TiKV/PostgreSQL); by Juicedata (2017). |
| **MinIO** | Popular self-hosted S3-compatible object store (AGPLv3); frequently the object layer under JuiceFS/lakehouse stacks. |
| **S3** | Amazon Simple Storage Service API — the de facto object-storage protocol (buckets/keys, HTTP). |
| **EFS / Azure Files / Filestore** | Managed cloud NFS/SMB services: AWS Elastic File System (NFSv4.1), Azure Files (SMB + NFSv4.1), Google Filestore (NFS). |
| **FSx / ANF / ONTAP** | Managed parallel/enterprise NAS: Amazon FSx for Lustre, FSx for NetApp ONTAP, Azure NetApp Files (NetApp ONTAP on Azure). |
| **POSIX** | IEEE standard defining Unix-like OS interfaces, including the filesystem API (open/read/write/rename/lock semantics). |
| **ReadWriteMany** | Kubernetes access mode: many pods/nodes may mount the volume read-write — CephFS's core Kubernetes value (vs RBD ReadWriteOnce). |
| **StorageClass / PVC** | Kubernetes storage abstractions: StorageClass defines provisioning (provisioner, pool, options); PVC requests a volume from it. |
| **HPC** | High-Performance Computing — parallel compute clusters; drives the Lustre/BeeGFS performance envelope. |
| **Metadata server** | Dedicated daemon(s) managing namespace/inode state (MDS in CephFS/Lustre, meta services in BeeGFS) — the scaling axis for metadata-heavy workloads. |
| **Subtree partitioning** | MDS mechanism splitting the directory tree across active MDS daemons dynamically by load — CephFS metadata scale-out. |



