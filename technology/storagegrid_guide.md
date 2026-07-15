# NetApp StorageGRID — Comprehensive Guide

> **Author:** Jack Liu Shurui (Solution Architect, Crédit Agricole CIB)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Date:** July 2026 | **Version:** StorageGRID 12.x

---

## 1. What is NetApp StorageGRID?

NetApp StorageGRID is a **software-defined object storage platform** designed for managing unstructured data across hybrid and multi-cloud environments. Unlike most object stores that rely on cluster-based architectures, StorageGRID employs a **grid-based architecture** — nodes are organized into a scalable federation called a *grid*, enabling independent scaling of compute, capacity, and protocol access with no single point of failure.

### 1.1 History

- **2000**: Bycast Inc. founded in Vancouver, Canada, develops StorageGRID as a fixed-content storage platform for healthcare imaging (PACS).
- **2010**: NetApp acquires Bycast for approximately $110 million. The product is rebranded as NetApp StorageGRID.
- **2013–2016**: Evolution through StorageGRID 9.x–10.x with S3 API support, multi-site active-active, and erasure coding.
- **2018**: StorageGRID 11.0 introduced major architectural updates, native S3 Object Lock, and deeper NetApp ecosystem integration.
- **2024**: StorageGRID 11.8 — performance upgrades, flash-based SGF6000 nodes, up to 640 PB per grid.
- **2025**: StorageGRID **12.0** — AI workload acceleration, bucket branches (Git-for-data semantics), advanced caching (up to 10× throughput improvement).
- **2026**: StorageGRID **12.1** — federated namespace, up to 12 TB/s aggregate throughput, cross-grid replication enhancements.

### 1.2 Key Differentiator

StorageGRID's defining characteristic is its **grid architecture** — a flat, peer-to-peer federation of nodes that distribute both data and metadata. There is no separate metadata server, no single controller, and no head node. This eliminates the bottleneck and SPOF that constrain traditional clustered object stores (including MinIO and Ceph RGW in certain configurations).

### 1.3 Primary Use Cases

| Domain | Use Case |
|---|---|
| **Financial Services** | Compliance archives (SEC 17a-4, FINRA, MAS), long-term email retention, trade document stores |
| **Healthcare** | PACS/VNA archives, EHR image storage, medical imaging (GE/Philips/Siemens integration) |
| **Media & Entertainment** | Video archives, content repositories, SigmaStor/Avid/Telestream integration |
| **Backup & DR** | Backup target for Commvault, Veritas NetBackup, Veeam; ransomware-resilient immutable repositories |
| **AI/ML** | Training dataset versioning (bucket branches), model artifact stores, data lake foundations |
| **Cloud Tiering** | FabricPool cold data tier for NetApp ONTAP FAS/AFF arrays |
| **IoT** | Sensor data lakes, log aggregation at petabyte scale |

---

## 2. Architecture

### 2.1 Grid

The **grid** is the top-level deployment unit — a federation of nodes spanning one or more sites (up to 16 sites in active-active configuration). All nodes within a grid share a single namespace and are managed through a unified Grid Manager interface. Data can span sites transparently.

```
                    STORAGEGRID GRID
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  Site A   │   │  Site B   │   │  Site C   │   ...
  │(Primary)  │   │(Active)   │   │(Active)   │
  ├──────────┤   ├──────────┤   ├──────────┤
  │Storage   │   │Storage   │   │Storage   │
  │Admin     │   │Gateway   │   │Archive   │
  │Gateway   │   │Admin     │   │Gateway   │
  └──────────┘   └──────────┘   └──────────┘
```

### 2.2 Node Types

#### Storage Node
The workhorse of the grid. Each storage node runs StorageGRID software and manages local disks (JBOD/JBOD shelves). Storage nodes handle:
- Object data persistence (disk I/O)
- Distributed metadata storage (co-located NoSQL database)
- Erasure coding computation
- Replication and repair operations
- ILM policy evaluation

Storage nodes can be physical (appliance-based) or virtual (VMware, KVM, bare metal).

#### Admin Node
Provides the Grid Manager web UI, Grid Management API, and monitoring services. A grid requires at least one primary Admin Node. Non-primary (replica) Admin Nodes provide redundancy. All Admin Nodes display an identical view of the grid.

#### Gateway Node
Protocol endpoints that terminate client connections. Gateway nodes handle:
- S3 API request processing
- Swift API (legacy)
- Load distribution across storage nodes
- Authentication and access control
- TLS termination

Gateway nodes can be scaled independently from storage nodes to match throughput requirements.

#### Archive Node
Interface to external cold storage tiers — tape libraries (via TSM middleware) or public cloud (AWS S3/Glacier, Azure Blob Archive, Google Cloud Storage). Archive nodes are optional and used when ILM policies dictate tiering to tape or cloud.

### 2.3 Metadata Architecture

StorageGRID stores metadata in a **distributed NoSQL database co-located with storage nodes**. This is a critical architectural decision:

- **No separate metadata cluster** — metadata lives alongside data, eliminating the cross-cluster read/write penalty.
- **Distributed hash table** routes metadata lookups efficiently across the grid.
- **Consistency model**: Tunable — eventual, read-after-write, or strong consistency per request.
- **Scale**: Proven to 300 billion objects in a single namespace.

### 2.4 Information Lifecycle Management (ILM)

ILM is StorageGRID's **policy engine** — the brain that governs object placement, protection, and lifecycle. ILM is not a bolt-on feature; it is integral to the architecture.

**ILM Policy Structure:**
```
ILM Policy → ILM Rules (filter by tenant/bucket/tags/size/time)
  → Placement: Replication (N copies), Erasure Coding (2+1..16+4),
    Cloud Tiering (AWS/Azure/GCP), Tape Archive
  → Duration: time-based transitions between tiers
```
Multiple concurrent policies enable tenant/bucket-specific QoS. ILM can place copies on different sites, tiers, or cloud providers simultaneously.

### 2.5 Multi-Site Topologies

- **Single Site**: Minimum 3 storage nodes for production. Suitable for dev/test or single-datacenter deployments.
- **Two Sites**: Active-active replication across two data centers. Each site holds a full copy.
- **Three Sites (Recommended for HA)**: Erasure coding across three sites (e.g., 4+2 EC per site with cross-site replication). Tolerates loss of one full site.
- **Up to 16 Sites**: Geo-distributed active-active grid for global namespace.

StorageGRID uses **link-cost groups** to model network latency and bandwidth between sites, optimizing ILM placement decisions.

---

## 3. Key Features

### 3.1 S3 API Compliance

StorageGRID implements core S3 REST APIs: GET/PUT/DELETE/HEAD Object, ListObjects, Multipart Upload, S3 Object Lock (Governance & Compliance modes, legal hold), S3 Select (push-down predicate evaluation), S3 Batch Operations, bucket lifecycle policies, bucket policies & IAM-style access control, versioning, and SSE-S3/SSE-C encryption.

### 3.2 Erasure Coding

**EC Profiles** (all at ~50% overhead): 2+1, 4+2, 6+3, 8+4 — tolerating 1–4 failures respectively.
**Lower overhead profiles**: 12+4 (33%), 16+4 (25%) — suitable for larger deployments.

EC operates at the **object level** with fragments distributed across nodes, racks, or entire sites. Multiple profiles can coexist in the same grid via different ILM rules. Objects below ~1 MB are typically replicated rather than erasure-coded to avoid EC overhead dominating storage efficiency.

### 3.3 S3 Object Lock & Compliance

- **Governance Mode**: Object can be modified by users with special permissions (controlled override).
- **Compliance Mode**: Strict WORM — no user, including root, can modify or delete until retention expires.
- **Legal Hold**: Indefinite hold for litigation or investigation scenarios.
- **Default Bucket Retention**: Enforce minimum/maximum retention periods at the bucket level.

**Certifications**: SEC Rule 17a-4(f), FINRA Rule 4511(c), CFTC, DoD DODIN APL, SOC 2, ISO 27001, FIPS 140-2 Level 2, Common Criteria EAL2+.

### 3.4 Multi-Tenancy

- **Tenant Accounts**: S3 tenant accounts with isolated namespaces, access keys, and usage metering.
- **Bucket Policies**: Per-bucket access control with S3-style policy language.
- **Group-Based Access**: LDAP/AD group integration for RBAC at grid and tenant level.
- **Usage Monitoring**: Per-tenant storage consumption, API request counts, and bandwidth tracking.
- **Tenant Manager**: Self-service UI for tenant administrators to manage users, groups, bucket policies, and access keys.

### 3.5 Load Balancing

StorageGRID includes **built-in load balancing** across gateway nodes and storage nodes. Requests are automatically distributed based on node health, capacity, and network proximity. External load balancers (KEMP, F5, NGINX) can also front-end gateway nodes.

### 3.6 Grid Federation

Introduced in StorageGRID 12.1, **Grid Federation** allows multiple independent grids to be connected:

- **Cross-Grid Replication (CGR)** — object-level replication between grids.
- **Account Cloning** — tenant accounts can be cloned across federated grids.
- **Unified Management** — monitor and manage federated grids from a single Grid Manager.

### 3.7 Versioning & Bucket Branches (12.0+)

**Bucket Branches** (introduced in StorageGRID 12.0) provide Git-for-data semantics:

- Instantly create space-efficient dataset copies for AI/ML training, testing, and version control.
- Branches share underlying object data via copy-on-write — no physical duplication until modifications occur.
- Enable reproducible ML experiments by snapshotting training datasets at branch points.

### 3.8 Security Features

| Feature | Details |
|---|---|
| **Encryption at Rest** | AES-256 (SSE-S3), SSE-C, KMIP key management (external KMS) |
| **Encryption in Transit** | TLS 1.2/1.3 for all grid communication and client access |
| **Authentication** | Local accounts, LDAP/AD, SAML 2.0 SSO (AD FS, Azure AD, PingFederate) |
| **FIPS Compliance** | FIPS 140-2 Level 2 validated cryptographic modules |
| **Audit Logging** | Comprehensive object-level audit trail (who accessed what, when, from where) |
| **RBAC** | Role-based access control with predefined roles (Grid Admin, Tenant Admin, Read-only) |

---

## 4. Storage Efficiency

### 4.1 Erasure Coding vs Replication

| Method | Effective Overhead | Usable Capacity (per 100 TB raw) |
|---|---|---|
| 3× Replication | 200% overhead | 33 TB |
| 8+4 EC (50% overhead) | 50% overhead | 67 TB |
| 16+4 EC (25% overhead) | 25% overhead | 80 TB |
| 12+4 EC (33% overhead) | 33% overhead | 75 TB |

EC provides the same or better durability than replication at significantly lower overhead. StorageGRID's ILM allows mixing both in the same policy (replication for small/hot objects, EC for larger/cold objects).

### 4.2 Compression

Per-object compression (gzip) is supported but must be explicitly enabled. It is best suited for compressible data types (logs, text, documents). Already-compressed formats (JPEG, MP4, encrypted data) gain little benefit.

### 4.3 Deduplication

StorageGRID does **not** perform inline deduplication — it is an object store designed for fixed-content and archive workloads, not primary VM storage. For block-level deduplication, NetApp recommends ONTAP (FAS/AFF). However, StorageGRID's ILM can avoid storing duplicate copies of objects that share identical content via reference-counting at the object level.

### 4.4 Cloud & Tape Tiering

ILM rules can automatically transition objects to lower-cost tiers:

- **Cloud Storage Pools**: AWS S3 Standard → S3 Glacier → S3 Glacier Deep Archive; Azure Blob Archive; Google Cloud Archive.
- **Tape**: Via Archive Node and TSM middleware.
- **Estimated Savings**: Up to 95% on storage costs for cold data tiered to cloud archive.

### 4.5 FabricPool Integration

When used as a FabricPool cloud tier for NetApp ONTAP:

- **Hot data** stays on ONTAP flash (AFF) or hybrid (FAS) storage.
- **Cold data** is transparently tiered to StorageGRID, freeing up to 90% of primary storage capacity.
- Tiering is policy-driven (age, frequency of access) and transparent to applications.
- Requires StorageGRID 11.4+ and ONTAP 9.4+.

---

## 5. Deployment Options

### 5.1 Software-Only Deployment

StorageGRID can be deployed as software on commodity hardware or hypervisors:

- **Supported Hypervisors**: VMware vSphere (ESXi), KVM, Hyper-V
- **Bare Metal**: RHEL, CentOS, Ubuntu (x86_64)
- **Containerized**: Docker containers on Kubernetes / OpenShift (newer deployment model via NetApp-StorageGRID/Kubernetes GitHub repository)
- **Minimum Specs per Storage Node**: 16 CPU cores, 64 GB RAM, dedicated disks (SSD/HDD)

### 5.2 Appliance-Based Deployment (SG Series)

NetApp offers pre-configured, validated hardware appliances:

| Appliance Model | Type | Description |
|---|---|---|
| **SG5712** | Storage Node | 4U, 12 drives, high-density HDD |
| **SG5760** | Storage Node | 4U, 60 drives, ultra-dense |
| **SG6060** | Storage Node | 4U, 60 drives, FIPS-certified drives option |
| **SGF6000** | Storage Node (Flash) | All-flash, 24× NVMe SSDs, high-performance |
| **SG1000** | Admin/Gateway | Service appliance for admin + load balancing |
| **SG6000-CN** | Compute Node | Gateway node / admin node appliance |

Appliance-based deployment simplifies procurement, cabling, and initial setup (factory pre-configured).

### 5.3 Cloud Deployment

- **AWS**: StorageGRID can run on EC2 instances, with EBS or instance store for data persistence.
- **Azure**: Supported on Azure VMs with managed disks.
- **GCP**: Supported on Google Compute Engine.
- **Hybrid**: Mixed on-premises + cloud nodes in the same grid.

### 5.4 Containerized Deployment

StorageGRID nodes can run as Docker containers orchestrated by:

- **Kubernetes**: Via sample YAML configurations on [GitHub](https://github.com/NetApp-StorageGRID/Kubernetes).
- **Red Hat OpenShift**: Certified operator integration with Trident CSI for persistent storage.
- **Kubernetes Operator**: Community operator ([bedag/storagegrid-operator](https://github.com/bedag/storagegrid-operator)) for managing S3 tenants and buckets as CRDs.

### 5.5 Deployment Requirements

| Component | Minimum (Dev/Test) | Recommended (Production) |
|---|---|---|
| Storage Nodes | 3 | 4+ per site |
| Admin Nodes | 1 | 2 (primary + non-primary) |
| Gateway Nodes | 0 (co-located) | 2+ per site |
| Sites | 1 | 3 for HA |
| Networking | 1 GbE | 10/25 GbE inter-node, 25/100 GbE client-facing |
| Latency between sites | — | <20 ms RTT recommended for synchronous operations |

---

## 6. StorageGRID vs Other Object Stores

### 6.1 Comparison Table

| Feature | StorageGRID | MinIO (AIStor) | Ceph RGW | AWS S3 | Dell EMC ECS |
|---|---|---|---|---|---|
| **Architecture** | Grid (peer-to-peer) | Standalone/Cluster | Cluster (RADOS) | Regional AZ | Cluster |
| **Erasure Coding** | Node/rack/site | Erasure sets | Via libscatter | AZ-based | Site-level |
| **Multi-Site Active-Active** | Up to 16 sites | ❌ Federation | ⚠️ RGW multi-site | Regions | Up to 9 sites |
| **SEC/FINRA Compliance** | ✅ Certified | ⚠️ Partial | ⚠️ Partial | ✅ | ✅ |
| **NFS/SMB Gateway** | ✅ File gateway | ❌ | ⚠️ NFS-Ganesha | ❌ | ✅ |
| **ILM Policy Engine** | Deep multi-rule | Basic lifecycle | Lifecycle | Lifecycle | ✅ |
| **Tape/Cloud Tiering** | ✅ Built-in | ❌ | Via S3 | ❌ | ✅ |
| **Enterprise Support** | NetApp | MinIO Inc. | Community/Red Hat | AWS | Dell |
| **Latency** | Moderate | Very low | Moderate | Very low | Moderate |
| **Max Namespace** | 640 PB / 300B obj | ~PB-scale | ~EB-scale | Unlimited | ~PB-scale |
| **K8s-Native** | Container support | ✅ Fully native | ✅ Rook Operator | ❌ | ⚠️ |
| **Min Deployment** | 3 storage nodes | 2 nodes | 3 mon + OSDs | N/A | 3 nodes |

### 6.2 Decision Guide

**Choose StorageGRID when**: You need certified compliance (SEC 17a-4/FINRA), multi-site active-active (up to 16 sites), sophisticated ILM policies, NetApp ecosystem (ONTAP/FabricPool), NFS/SMB gateway alongside S3, or tape archiving integrated into storage policy.

**Consider alternatives when**: MinIO (K8s-native, ultra-low latency, open-source preference); Ceph (unified block/file/object, massive open-source scale); AWS S3 (cloud-native, pure cloud, unlimited scale); Dell ECS (Dell incumbent, appliance preference).

---

## 7. StorageGRID in Banking & Finance

### 7.1 Regulatory Compliance Archives

Financial institutions use StorageGRID as the foundation for compliance-archived data:

- **SEC Rule 17a-4(f)**: Requires electronic records to be stored in WORM format, non-rewritable, non-erasable for retention periods (typically 3–7 years). StorageGRID's S3 Object Lock in Compliance mode satisfies this requirement, verified by KPMG certification.
- **FINRA Rule 4511(c)**: Records must be preserved in a compliant WORM format for at least 6 years.
- **MAS (Monetary Authority of Singapore)**: Notice 644 on record keeping — retention of all records for at least 5 years, with immutability requirements.
- **MiFID II (Europe)**: 5-year retention for communications and transaction records.
- **BCBS 239**: Risk data aggregation and reporting — StorageGRID provides audit-proof immutable storage for risk data repositories.

### 7.2 Common Banking Use Cases

| Use Case | Description | Typical Retention |
|---|---|---|
| **Email Archiving** | Global Relay, Smarsh, Enterprise Vault integration for regulatory email | 7+ years |
| **Trade Finance Documents** | Letters of credit, invoices, bills of lading as immutable objects | 10+ years |
| **Loan Processing** | Origination, servicing, securitization document stores | 7+ years |
| **Backup Target** | Commvault/Veritas backup of critical systems | 30 days–7 years |
| **FabricPool Tier** | Cold data tier for ONTAP-based NAS | Indefinite |
| **Splunk Log Archive** | SIEM/audit log long-term retention | 2–7 years |
| **SWIFT Message Archive** | FIN/MX messages for reconciliation and audit | 10+ years |

### 7.3 Why Banks Choose StorageGRID

1. **Certified Compliance**: KPMG-validated S3 Object Lock for SEC/FINRA/MAS — eliminating expensive third-party WORM appliances.
2. **Active-Active Geo-Redundancy**: Up to 16 sites — data survives regional disasters without manual failover.
3. **Ransomware Resilience**: Immutable object storage prevents encryption or deletion by attackers.
4. **Economic Efficiency**: Erasure coding (50% overhead) vs 3× replication (200% overhead) dramatically reduces TCO at petabyte scale.
5. **Data Fabric**: Seamless tiering from ONTAP → StorageGRID → public cloud archive.
6. **Multi-Tenant Isolation**: Isolated tenant accounts for separate business units or subsidiaries.

---

## 8. Integration & Ecosystem

### 8.1 NetApp Ecosystem

| Integration | Description |
|---|---|
| **FabricPool** | ONTAP auto-tiering of cold data blocks to StorageGRID. Transparent to applications. |
| **Cloud Backup Service** | NetApp's SaaS backup service using StorageGRID as a target. |
| **SnapCenter** | Application-consistent backup of databases (Oracle, SQL Server, SAP) to StorageGRID. |
| **Cloud Volumes ONTAP** | ONTAP in AWS/Azure with StorageGRID as tiering target. |
| **Trident CSI** | Dynamic persistent volume provisioning in Kubernetes/OpenShift from StorageGRID. |

### 8.2 Backup Software

- **Commvault**: S3 backup target with StorageGRID optimization; S3 Object Lock for immutable backup chains.
- **Veritas NetBackup**: v14.2.2+ with S3 Object Lock; Enterprise Vault for email archiving.
- **Veeam**: Backup & Replication v12 with S3-compatible object repository; immutable backup support.
- **IBM Spectrum Protect**: TSM integration via S3.
- **Rubrik**: Backups to StorageGRID S3 for long-term retention.

### 8.3 Compliance & Archiving Software

- **Global Relay / Smarsh**: Email archiving and supervision for financial services, stores to StorageGRID S3.
- **Veritas Enterprise Vault**: Enterprise archiving with StorageGRID as WORM-compliant storage.
- **Mimecast / Proofpoint**: Cloud email security and archiving with StorageGRID archive tier.

### 8.4 Analytics & ML Integration

- **Apache Spark / Presto / Trino**: Access data via S3A connector with S3 Select push-down filtering.
- **Bucket Branches (12.0)**: Version-controlled ML training datasets for reproducible experiments.

### 8.5 Healthcare & Media Integrations

- **Healthcare**: PACS (GE, Philips, Siemens), VNA archives, EHR (Epic, Cerner) document/image repositories.
- **Media & Entertainment**: SigmaStor, Telestream Vantage, Avid Media Composer for video archives.

### 8.6 Security Integration

- **Auth**: LDAP, Active Directory, SAML 2.0 (AD FS, Azure AD, PingFederate)
- **Key Mgmt**: KMIP-compliant KMS (Thales, Gemalto, HashiCorp Vault)
- **SIEM**: Syslog forwarding to Splunk, QRadar, ArcSight
- **TLS**: Public CA-trusted certificates for Grid Manager and S3 endpoints

---

## 9. Operations & Management

### 9.1 Grid Manager

The Grid Manager is the browser-based administrative interface for:

- **Dashboard**: Health, capacity, performance at-a-glance
- **ILM**: Create, edit, activate, and simulate ILM policies
- **Tenants**: Create and manage tenant accounts, monitor usage
- **Storage**: View storage node health, disk status, capacity trends
- **Network**: Configure networks, VLANs, routes, link-cost groups
- **Alerts**: SNMP traps, email notifications, syslog forwarding
- **Upgrades**: Rolling software upgrades with zero downtime
- **Logs**: Centralized log collection, filtering, and export

### 9.2 REST APIs

StorageGRID exposes comprehensive REST APIs for automation:

- **Grid Management API**: Full administrative operations (ILM, tenants, nodes, monitoring).
- **Tenant Management API**: Tenant-level operations (users, groups, buckets, access keys).
- **Prometheus Metrics API**: Expose grid and tenant metrics for external monitoring.
- **Ansible Collection**: `netapp.storagegrid` Ansible modules for infrastructure-as-code (ILM policies, tenants, buckets, identity federation).

### 9.3 Monitoring & Observability

- **SNMP**: MIB support for grid health, capacity, and performance alerts.
- **Syslog**: Forward audit and system logs to Splunk/ELK.
- **Prometheus**: Embedded metrics endpoint; federate to external Prometheus + Grafana with pre-built dashboards from [NetApp-StorageGRID/storagegrid-monitoring](https://github.com/NetApp-StorageGRID/storagegrid-monitoring).
- **Alerts**: Configurable rules for capacity thresholds, node health, ILM violations.
- **Health API**: REST endpoint for automated health checks and orchestration integration.

### 9.4 Audit Log

Comprehensive object-level audit trail recording:

- Object creation, read, update, deletion
- Authentication events (login, logout, failed attempts)
- ILM policy changes
- Configuration changes
- Tenant administration actions

Audit logs are tamper-evident and can be forwarded to SIEM for compliance reporting.

### 9.5 Maintenance Operations

| Operation | Impact | Notes |
|---|---|---|
| **Software Upgrade** | Zero downtime | Rolling node-by-node; client traffic continues |
| **Add/Remove Storage Node** | Zero downtime | Grid rebalances automatically; objects migrate off before decommission |
| **Replace Disk** | Zero downtime | Degraded EC mode until rebuild completes |
| **Add Site** | Zero downtime | Extended planning for network and replication topology |
| **Modify ILM Policy** | Zero downtime | New policy applies to new objects; existing objects transition async |

---

## 10. Getting Started & Sizing

### 10.1 Sizing Considerations

**Key factors for StorageGRID sizing:**

1. **Object Count vs Capacity**: StorageGRID is proven to 640 PB and 300 billion objects. Estimate both — object count drives metadata storage requirements.
2. **Object Size Distribution**: Small objects (<1 MB) consume disproportionately more metadata overhead — factor 1.5–2× for metadata on very small objects.
3. **Protection Level**: Replication (2×, 3×) vs erasure coding (2+1, 4+2, 8+4) — EC with 50% overhead is typically the sweet spot.
4. **Growth Rate**: Annual capacity growth projections (3–5 years typical planning horizon).
5. **Number of Sites**: Multi-site deployments require additional capacity for cross-site protection copies.
6. **Access Patterns**: Read-heavy vs write-heavy — gateway node count and network architecture differ significantly.
7. **Retention Periods**: Longer retention directly translates to more capacity (compounded over time).
8. **Performance Requirements**: IOPS, throughput (MB/s), and latency SLAs per tenant.

### 10.2 Quick Sizing Example (Banking Compliance Archive)

```
Assumptions: 100 TB/year email + document archiving, 7-year retention = 700 TB
3-site active-active, 8+4 EC (50% overhead), avg object size 256 KB

Calculation: 700 TB × 1.5 (EC) × 2 (multi-site) = 2,100 TB raw
Nodes: 3 sites × 4 storage nodes = 12 nodes → ~175 TB raw/node
Recommend: SG5760 (60×18 TB HDD) per node — room for growth
Gateway: 2 per site | Admin: 1 primary + 1 replica per site
```

### 10.3 Network Considerations

| Segment | Recommended | Purpose |
|---|---|---|
| **Client-to-Gateway** | 25/100 GbE | Tenant data access |
| **Inter-Node (same site)** | 10/25 GbE | Replication, EC, ILM operations |
| **Inter-Site** | Dedicated WAN (10+ GbE) | Cross-site replication, active-active access |
| **Management** | 1 GbE (separate VLAN) | Grid Manager, API, monitoring |

**Latency Budget**: For synchronous multi-site operations, keep inter-site RTT under 20 ms. ILM can be configured for asynchronous replication over higher-latency links.

### 10.4 Licensing

- **Per-Terabyte Licensing**: Based on consumed storage (raw capacity).
- **Perpetual vs Subscription**: Both models available.
- **FabricPool Licensing**: Separate tiering license when used with ONTAP FabricPool.
- **Support**: NetApp SupportEdge (Standard, Premium, Premium+).

### 10.5 Quick Start Resources

- **Free Trial**: StorageGRID software trial available at [netapp.com/storagegrid](https://www.netapp.com/storagegrid/)
- **Lab Deployment**: Deploy on VMware Workstation or KVM with 3 VM nodes for evaluation.
- **Documentation**: [docs.netapp.com/us-en/storagegrid](https://docs.netapp.com/us-en/storagegrid/)
- **Kubernetes Samples**: [github.com/NetApp-StorageGRID/Kubernetes](https://github.com/NetApp-StorageGRID/Kubernetes)
- **Monitoring**: [github.com/NetApp-StorageGRID/storagegrid-monitoring](https://github.com/NetApp-StorageGRID/storagegrid-monitoring)

---

## 11. Version History

| Version | Year | Key Features |
|---|---|---|
| **11.0** | 2018 | Major architectural overhaul, native S3 Object Lock, FabricPool support |
| **11.4** | 2020 | Enhanced ILM, Prometheus metrics, multi-site operations |
| **11.6** | 2021 | S3 Select, default bucket retention, Veritas integration |
| **11.8** | 2024 | SG6000/SGF6000 appliances, up to 640 PB |
| **12.0** | Sep 2025 | Bucket branches (Git-for-data), 10× caching, AI acceleration |
| **12.1** | 2026 | Federated namespace, cross-grid replication, 12 TB/s throughput |

---

## 12. References

1. **NetApp StorageGRID Docs**: <https://docs.netapp.com/us-en/storagegrid/>
2. **Product Page**: <https://www.netapp.com/storagegrid/>
3. **Architecture**: <https://docs.netapp.com/us-en/storagegrid-116/primer/storagegrid-architecture-and-network-topology.html>
4. **ILM Guide**: <https://docs.netapp.com/us-en/storagegrid/ilm/>
5. **Security Features**: <https://docs.netapp.com/us-en/storagegrid-enable/technical-reports/data-security-administration-security.html>
6. **KPMG Compliance Cert**: <https://community.netapp.com/t5/Tech-ONTAP-Blogs/StorageGRID-achieves-KPMG-compliance-certification/ba-p/440343>
7. **Monitoring (Prometheus/Grafana)**: <https://github.com/NetApp-StorageGRID/storagegrid-monitoring>
8. **K8s Deployment**: <https://github.com/NetApp-StorageGRID/Kubernetes>
9. **Veeam Best Practices**: <https://docs.netapp.com/us-en/storagegrid-enable/tools-apps-guides/veeam-br.html>
10. **Commvault Integration**: <https://docs.netapp.com/us-en/storagegrid-enable/technical-reports/commvault-index.html>
11. **Blocks & Files — 12.0**: <https://blocksandfiles.com/2025/09/10/netapp-sends-in-the-clones-to-boost-ai-pitch-as-storagegrid-hits-12/>
12. **StorageReview — 12.1**: <https://www.storagereview.com/news/netapp-storagegrid-12-1-adds-a-federated-namespace-and-up-to-12-tb-s-for-ai-workloads>
13. **PeerSpot vs MinIO**: <https://www.peerspot.com/products/comparisons/minio_vs_netapp-storagegrid>
14. **WWT Analysis**: <https://www.wwt.com/article/what-sets-netapps-storagegrid-platform-apart>
15. **FabricPool Best Practices (TR-4598)**: NetApp Technical Report TR-4598

---

*This guide is maintained as part of the [research](https://github.com/jackliusr/research) repository by Jack Liu Shurui. Last updated: July 2026. Corrections and contributions welcome via pull request.*
