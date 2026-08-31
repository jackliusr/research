# Securing Red Hat OpenShift — the hardened cluster

**The Container-Platform Security Deep-Dive — the Platform Security Model (OpenShift 4 Architecture, Defense-in-Depth, SELinux, CRI-O, FIPS Mode), the Cluster Security (Security Context Constraints, RBAC, Admission Control, Pod Security Admission), the Workload Security (Image Signing and Trust, Supply Chain, Secrets, Runtime Security), the Compliance and Audit (the Compliance Operator, Audit Logging, Certificates), the Red Hat Security Product Family (Red Hat Advanced Cluster Security for Kubernetes, Red Hat Quay), the Regulated-Banking Angle (MAS Notice 658), the Cymbal Bank Worked Example — and the Final Word: the Hardened Cluster**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Platform Security — the DEDICATED Red Hat OpenShift security deep-dive of the platform cluster. Where [openshift_scc_comprehensive_guide.md](openshift_scc_comprehensive_guide.md) and [openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md) go deep on Security Context Constraints and [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) covers the availability/HA angle, this guide is the security discipline itself: the platform security model and defense-in-depth layers, the cluster security machinery (SCCs, RBAC, admission control, Pod Security Admission), the workload security chain (image signing, supply chain, secrets, runtime), the compliance and audit machinery (the Compliance Operator, audit logs, certificates), the Red Hat security product family (RHACS, Red Hat Quay), and the regulated-banking overlay (MAS supervision, Notice 658 outsourcing). It is written to stand alone and to cross-ref the cluster heavily.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass had **live web access** — the key claims were verified on 2026-08-31 via web search and direct page extraction at the primary sources: **docs.redhat.com** (the OpenShift 4.18 Security Context Constraints chapter, the 4.11 Pod Security Admission chapter, the 4.12 etcd-encryption chapter, the 4.21 FIPS-cryptography chapter, the 4.20 Compliance Operator chapter, the 4.9 audit-log-policy chapter, the 4.19 External Secrets Operator chapter, the Red Hat Quay Clair chapters), **access.redhat.com** (the image-signing article 3116561), **redhat.com** (the press release and close blog for the StackRox acquisition, the CRI-O-default blog, the Pod Security Admission in OpenShift 4.11 blog, the "OpenShift 4 is Now Available" blog — GA date via the Wayback Machine capture), **kubernetes.io** (the PodSecurityPolicy removal record), and **developers.redhat.com** (the eBPF article, the RHTAS announcement). Anything that could not be verified at a primary source is flagged ⚠ honestly — see the claims ledger at the end.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [OpenShift SCC Comprehensive](openshift_scc_comprehensive_guide.md) (**THE SCC companion — the deep SCC mechanism detail, cross-ref §4**), [OpenShift SCC Service Accounts](openshift_scc_service_account_guide.md) (the service-account/SCC interplay — cross-ref §4 and §5), [OpenShift Workload Availability](openshift_workload_availability_guide.md) (the availability/HA angle — the fail-secure tension, cross-ref §1), [IBM Cloud Guide](ibm_cloud_guide.md) (OpenShift on IBM Cloud — the platform-offering angle, cross-ref §13), [Cybersecurity](cybersecurity_guide.md) (**THE security-framework companion — NIST CSF 2.0, CIS Controls, MITRE ATT&CK — cross-ref §9 and §13**), [SCADA, ICS and OT Security](scada_guide.md) (the NIST 800-53 / 800-82 / 62443 references — cross-ref §9), [OpenSCAP](openscap_guide.md) (the OpenSCAP internals the Compliance Operator runs — cross-ref §9), [AI Platform Engineering](ai_platform_engineering_guide.md) (Kubernetes/platform practices — light), [Enterprise Middleware Integration Platform](enterprise_middleware_integration_platform_guide.md) (integration — light)
> **Companion guides (banking/, prefix `../banking/`):** [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) (**THE regulatory companion — Notice 658, the TRMG, the Outsourcing Guidelines, data residency — cross-ref §13**)

---

**How to use this guide:** Section 1 is the platform security model — what OpenShift 4 actually is (GA 4 June 2019), the four defense-in-depth layers, and the overview table. Section 2 is the host layer — SELinux, CRI-O, FIPS mode. Section 3 is the data layer — etcd encryption at rest and key management. Section 4 is Security Context Constraints — the mechanism, the priority ordering, the default SCCs. Section 5 is the rest of cluster security — RBAC, admission control, and the Pod Security Admission relationship. Section 6 is the network layer — NetworkPolicy and namespace isolation. Section 7 is workload security — image signing and trust. Section 8 is the rest of workload security — supply chain, secrets, and runtime security (seccomp, SELinux, eBPF). Section 9 is the compliance layer — the Compliance Operator, its OpenSCAP engine and its supported profiles. Section 10 is the audit layer — audit logging profiles and certificate management. Section 11 is the first half of the Red Hat security product family — Red Hat Advanced Cluster Security for Kubernetes (RHACS). Section 12 is the second half — Red Hat Quay and Clair. Section 13 is the regulated-banking angle — OpenShift under MAS supervision. Section 14 is the worked example — a regulated Cymbal Bank OpenShift deployment. The glossary, the claims-status ledger, the "What Could Not Be Verified" section and the cross-references close the file. **Integrity convention:** ✅ = verified this pass at a primary source (or in the cross-referenced guide's ledger); ⚠ = flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source verified this pass. The most important single idea of the whole guide: **an OpenShift cluster is a security platform, not a security product** — the hardening is a discipline of configuration, policy and evidence, and for a MAS-supervised bank that discipline is the audit trail.

---

## Table of Contents

1. [The Platform Security Model — OpenShift 4 Architecture and Defense-in-Depth](#1-the-platform-security-model--openshift-4-architecture-and-defense-in-depth)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The OpenShift 4 Architecture — a Kubernetes Platform, Rebuilt](#12-the-openshift-4-architecture--a-kubernetes-platform-rebuilt)
   - 1.3 [The Defense-in-Depth Layers — Host / Container / Cluster / Application](#13-the-defense-in-depth-layers--host--container--cluster--application)
   - 1.4 [The Platform Security Table — Layer / Mechanism / What It Stops](#14-the-platform-security-table--layer--mechanism--what-it-stops)
   - 1.5 [Why the Layers Matter in a Bank](#15-why-the-layers-matter-in-a-bank)
2. [The Host Layer — SELinux, CRI-O and FIPS Mode](#2-the-host-layer--selinux-cri-o-and-fips-mode)
   - 2.1 [SELinux — the Mandatory Access Control Layer](#21-selinux--the-mandatory-access-control-layer)
   - 2.2 [CRI-O — the Kubernetes-Native Container Runtime](#22-cri-o--the-kubernetes-native-container-runtime)
   - 2.3 [FIPS Mode — the Cryptographic Floor](#23-fips-mode--the-cryptographic-floor)
   - 2.4 [The Host-Layer Table — Control / Mechanism / Verified](#24-the-host-layer-table--control--mechanism--verified)
3. [The Data Layer — etcd Encryption at Rest and Key Management](#3-the-data-layer--etcd-encryption-at-rest-and-key-management)
   - 3.1 [What etcd Holds — and Why It Is the Crown Jewels](#31-what-etcd-holds--and-why-it-is-the-crown-jewels)
   - 3.2 [Enabling etcd Encryption — the Mechanism](#32-enabling-etcd-encryption--the-mechanism)
   - 3.3 [Key Rotation and Backup Keys](#33-key-rotation-and-backup-keys)
   - 3.4 [External Key Management — KMS v2](#34-external-key-management--kms-v2)
   - 3.5 [The Data-Layer Table](#35-the-data-layer-table)
4. [The Cluster Security Layer — Security Context Constraints](#4-the-cluster-security-layer--security-context-constraints)
   - 4.1 [What an SCC Is — the Pod Permission Contract](#41-what-an-scc-is--the-pod-permission-contract)
   - 4.2 [The Admission Mechanism — RBAC-Granted, Priority-Ordered](#42-the-admission-mechanism--rbac-granted-priority-ordered)
   - 4.3 [The Default SCCs — the 4.18 Catalog](#43-the-default-sccs--the-418-catalog)
   - 4.4 [SCC Strategy Fields — Boolean, Allowable-Set, Strategy](#44-scc-strategy-fields--boolean-allowable-set-strategy)
   - 4.5 [The SCC Table — Default SCC / What It Allows / Warning](#45-the-scc-table--default-scc--what-it-allows--warning)
5. [The Cluster Security Layer — RBAC, Admission Control and Pod Security Admission](#5-the-cluster-security-layer--rbac-admission-control-and-pod-security-admission)
   - 5.1 [RBAC — Roles, Cluster Roles and Bindings](#51-rbac--roles-cluster-roles-and-bindings)
   - 5.2 [Admission Control — Built-in Plugins and Webhooks](#52-admission-control--built-in-plugins-and-webhooks)
   - 5.3 [Pod Security Admission and Pod Security Standards](#53-pod-security-admission-and-pod-security-standards)
   - 5.4 [The SCC ↔ PSA Relationship — the 4.11 Synchronization](#54-the-scc--psa-relationship--the-411-synchronization)
   - 5.5 [Namespace Isolation — the Tenant Boundary](#55-namespace-isolation--the-tenant-boundary)
   - 5.6 [The Cluster-Security Table](#56-the-cluster-security-table)
6. [The Network Layer — NetworkPolicy and Namespace Isolation](#6-the-network-layer--networkpolicy-and-namespace-isolation)
   - 6.1 [OVN-Kubernetes — the Default Network Provider](#61-ovn-kubernetes--the-default-network-provider)
   - 6.2 [NetworkPolicy — the Micro-Segmentation Primitive](#62-networkpolicy--the-micro-segmentation-primitive)
   - 6.3 [The Network Table](#63-the-network-table)
7. [The Workload Security Layer — Image Signing and Trust](#7-the-workload-security-layer--image-signing-and-trust)
   - 7.1 [Why Image Trust Is a Bank Control](#71-why-image-trust-is-a-bank-control)
   - 7.2 [Red Hat's Signed-Image Infrastructure](#72-red-hats-signed-image-infrastructure)
   - 7.3 [Sigstore and cosign — the Signature Formats](#73-sigstore-and-cosign--the-signature-formats)
   - 7.4 [Enforcing Signature Verification — policy.json and Admission](#74-enforcing-signature-verification--policyjson-and-admission)
   - 7.5 [The Image-Trust Table](#75-the-image-trust-table)
8. [The Workload Security Layer — Supply Chain, Secrets and Runtime Security](#8-the-workload-security-layer--supply-chain-secrets-and-runtime-security)
   - 8.1 [The Supply-Chain Discipline — SBOM, Provenance, CI/CD Gates](#81-the-supply-chain-discipline--sbom-provenance-cicd-gates)
   - 8.2 [Secrets Management — Encrypted at Rest, External Stores](#82-secrets-management--encrypted-at-rest-external-stores)
   - 8.3 [Runtime Security — seccomp, Capabilities, SELinux, eBPF](#83-runtime-security--seccomp-capabilities-selinux-ebpf)
   - 8.4 [The Workload-Security Table](#84-the-workload-security-table)
9. [The Compliance Layer — the Compliance Operator](#9-the-compliance-layer--the-compliance-operator)
   - 9.1 [What the Compliance Operator Is — and Is Not](#91-what-the-compliance-operator-is--and-is-not)
   - 9.2 [The Mechanism — ComplianceScan, ScanSetting, OpenSCAP Under the Hood](#92-the-mechanism--compliancescan-scansetting-openscap-under-the-hood)
   - 9.3 [The Supported Profiles — CIS, NIST 800-53, PCI-DSS, STIG and the Rest](#93-the-supported-profiles--cis-nist-800-53-pci-dss-stig-and-the-rest)
   - 9.4 [Remediation and Tailoring — the Evidence Chain](#94-remediation-and-tailoring--the-evidence-chain)
   - 9.5 [The Compliance-Operator Table](#95-the-compliance-operator-table)
10. [The Audit Layer — Audit Logging and Certificate Management](#10-the-audit-layer--audit-logging-and-certificate-management)
    - 10.1 [Kubernetes API Audit Logs — the Evidence Spine](#101-kubernetes-api-audit-logs--the-evidence-spine)
    - 10.2 [The Audit Policy Profiles — Default, WriteRequestBodies, AllRequestBodies](#102-the-audit-policy-profiles--default-writerequestbodies-allrequestbodies)
    - 10.3 [Certificate Management — the In-Cluster CA, Rotation, Service Certificates](#103-certificate-management--the-in-cluster-ca-rotation-service-certificates)
    - 10.4 [The Audit-and-Certificate Table](#104-the-audit-and-certificate-table)
11. [The Red Hat Security Product Family — Red Hat Advanced Cluster Security for Kubernetes](#11-the-red-hat-security-product-family--red-hat-advanced-cluster-security-for-kubernetes)
    - 11.1 [The StackRox Lineage — 2021, the Acquisition](#111-the-stackrox-lineage--2021-the-acquisition)
    - 11.2 [RHACS Capabilities — Vulnerability, Compliance, Segmentation, Runtime, Enforcement](#112-rhacs-capabilities--vulnerability-compliance-segmentation-runtime-enforcement)
    - 11.3 [The Architecture — Central, Sensor, Collector](#113-the-architecture--central-sensor-collector)
    - 11.4 [Built-In OpenShift Security vs RHACS — the Comparison Table](#114-built-in-openshift-security-vs-rhacs--the-comparison-table)
12. [The Red Hat Security Product Family — Red Hat Quay](#12-the-red-hat-security-product-family--red-hat-quay)
    - 12.1 [Quay as the Trusted Registry](#121-quay-as-the-trusted-registry)
    - 12.2 [Clair — the Vulnerability Scanner](#122-clair--the-vulnerability-scanner)
    - 12.3 [Mirroring and Robot Accounts — the Bank Pattern](#123-mirroring-and-robot-accounts--the-bank-pattern)
    - 12.4 [The Quay Table](#124-the-quay-table)
13. [OpenShift Under MAS Supervision — the Regulated-Banking Angle](#13-openshift-under-mas-supervision--the-regulated-banking-angle)
    - 13.1 [The MAS Floor — Notice 658 and the Outsourcing Overlay](#131-the-mas-floor--notice-658-and-the-outsourcing-overlay)
    - 13.2 [Data Residency and Segregation](#132-data-residency-and-segregation)
    - 13.3 [Audit-Evidence Obligations — the Platform as Evidence Machine](#133-audit-evidence-obligations--the-platform-as-evidence-machine)
    - 13.4 [The Shared-Responsibility Boundary in a Bank Context](#134-the-shared-responsibility-boundary-in-a-bank-context)
    - 13.5 [The MAS-Mapping Table](#135-the-mas-mapping-table)
14. [The Cymbal Bank Worked Example — a Regulated OpenShift Deployment](#14-the-cymbal-bank-worked-example--a-regulated-openshift-deployment)
    - 14.1 [The Scenario — the Payments-Workload Landing Zone](#141-the-scenario--the-payments-workload-landing-zone)
    - 14.2 [The SCC Strategy](#142-the-scc-strategy)
    - 14.3 [The RBAC Design](#143-the-rbac-design)
    - 14.4 [The Compliance Operator Profile Selection](#144-the-compliance-operator-profile-selection)
    - 14.5 [The ACS/RHACS Integration](#145-the-acsrhacs-integration)
    - 14.6 [The Audit-Trail Design](#146-the-audit-trail-design)
    - 14.7 [The Lessons](#147-the-lessons)
- [The Glossary](#the-glossary)
- [Claims Status and Verification Notes](#claims-status-and-verification-notes)
- [What Could Not Be Verified](#what-could-not-be-verified)
- [Cross-References and Further Reading](#cross-references-and-further-reading)

---

## 1. The Platform Security Model — OpenShift 4 Architecture and Defense-in-Depth

### 1.1 The Short Answer

**Red Hat OpenShift Container Platform is Red Hat's enterprise Kubernetes distribution, and its security model is defense-in-depth expressed as platform machinery.** OpenShift 4 (generally available **4 June 2019** ✅ — "As of today, Red Hat OpenShift 4 is generally available to Red Hat customers," the Red Hat blog of that date) was a re-architecture of the platform around Kubernetes Operators, Red Hat Enterprise Linux CoreOS (RHCOS) nodes and the CRI-O container engine. Security on OpenShift is not one control but a stack of them: **SELinux** on the host, **CRI-O** as the Kubernetes-native runtime, **Security Context Constraints (SCCs)** as the pod-permission contract, **RBAC** for human and machine identity, **NetworkPolicy** for micro-segmentation, **admission control** (built-in plugins plus webhooks) as the decision point, **image signing and trust** for the supply chain, and — for the regulated bank — the **Compliance Operator** and **audit logs** turning configuration into evidence. The sections that follow take each layer apart; this section frames the model.

### 1.2 The OpenShift 4 Architecture — a Kubernetes Platform, Rebuilt

OpenShift 4 differs from vanilla Kubernetes in three ways that matter for security:

1. **The platform is an operator-managed machine.** Control-plane and worker nodes run RHCOS, and the platform itself is installed, upgraded and repaired by Operators — the Cluster Version Operator (CVO) drives upgrades, the Machine Config Operator (MCO) renders host configuration (including the CRI-O configuration and FIPS-relevant settings) as declarative MachineConfig objects ✅-structural (verified in the CRI-O-default blog: "The task of configuring and versioning of the operating system and container engine is automated by what's called the Machine Config Operator"). Security configuration becomes *code* — a MachineConfig is a Kubernetes resource, which means it can be reviewed, versioned and audited like any other manifest. This is the property a bank's change-management function wants.
2. **The container engine is CRI-O, not Docker.** OpenShift 4 moved the default container engine from Docker to CRI-O, a container engine built specifically to implement the Kubernetes Container Runtime Interface (CRI) ✅ (Red Hat blog, "Red Hat OpenShift Container Platform 4 now defaults to CRI-O as underlying container engine"). Docker is neither installed nor required; users interact with containers through Podman (for debugging) and Buildah (for building), and the cluster runs containers through CRI-O. CRI-O's minimal scope — a CRI implementation that launches OCI-compliant runtimes such as runc — is itself a security property: fewer components, less attack surface, and a runtime that moves in lock-step with the Kubernetes version (cross-ref §2.2).
3. **etcd is the source of truth, and everything else is derived.** The cluster's entire desired state — secrets, config maps, routes, OAuth tokens — lives in etcd, which makes etcd encryption at rest (§3) one of the most important controls in the whole platform.

The OpenShift API surface adds OpenShift-specific resources on top of Kubernetes — Routes (the platform's HTTP ingress object, which influenced the Kubernetes Ingress concept ✅-structural, documented in the Wikipedia lineage and OpenShift docs), DeploymentConfigs, BuildConfigs, ImageStreams, and the security-relevant SCCs and PodSecurityAdmission synchronization (§4–§5). The `oc` CLI is a superset of `kubectl`.

### 1.3 The Defense-in-Depth Layers — Host / Container / Cluster / Application

Red Hat's own framing for OpenShift security is a **layered approach** — the same "security must be an integrated part of every deployment, not an afterthought" message appears in the StackRox acquisition press release ✅. The layers, from the bottom up:

- **Host layer** — the RHCOS node: SELinux enforcing, FIPS mode where required, CRI-O as the only container engine, encrypted storage (§2).
- **Container layer** — what a single container may do: seccomp profiles, Linux capabilities, read-only root filesystems, non-root UIDs — much of it enforced through the SCC machinery (§4) and the CRI-O defaults.
- **Cluster layer** — what a workload or a user may do in the cluster: SCCs, RBAC, NetworkPolicy, admission control, namespace isolation (§4–§6).
- **Application layer** — what an application may do end-to-end: signed images, external secrets, runtime monitoring, compliance evidence (§7–§11).

The point of the layering is that no single control is trusted: a container breakout (container layer) still hits SELinux and FIPS (host layer); a compromised service account (cluster layer) still cannot pull unsigned images or reach another tenant's namespace (application/network layers). For a bank, the layering maps directly onto the regulator's expectation of proportionate, defence-in-depth technology risk management (cross-ref §13 and the [Cybersecurity](cybersecurity_guide.md) framework map).

### 1.4 The Platform Security Table — Layer / Mechanism / What It Stops

| Layer | Primary mechanisms | What it stops | Verified |
|---|---|---|---|
| Host | SELinux (MCS), FIPS mode, CRI-O, encrypted node storage | Container escape into the host; untrusted runtimes; weak crypto | ✅ §2 |
| Container | seccomp, capability drops, non-root UID, read-only rootfs (via SCC defaults) | Privilege escalation inside the container; host-namespace access | ✅ §4 |
| Cluster | SCCs, RBAC, admission control, Pod Security Admission, NetworkPolicy | Unauthorized pod privileges; unauthorized API access; lateral movement | ✅ §4–§6 |
| Data | etcd encryption at rest, KMS, Secrets hygiene | Data exposure via etcd backup or disk theft | ✅ §3 |
| Supply chain | Image signing/trust, registry scanning (Clair), SBOM/provenance | Tampered or vulnerable images entering the cluster | ✅ §7–§8, §12 |
| Compliance/audit | Compliance Operator (OpenSCAP), audit-log profiles, certificates | Drift from the security baseline; unprovable security | ✅ §9–§10 |
| Runtime detection | RHACS (eBPF collector), policy enforcement | Active threats inside the cluster; policy violations at deploy time | ✅ §11 |

### 1.5 Why the Layers Matter in a Bank

A MAS-supervised bank does not get to choose between "secure" and "fast"; it must demonstrate both, continuously. The layered model gives the platform team the vocabulary to answer the regulator's question — "show me the controls" — with a table, not a narrative: host controls (§2), data controls (§3), cluster controls (§4–§6), workload controls (§7–§8), compliance and audit evidence (§9–§10), and the product family that operationalizes detection and enforcement (§11–§12). The Cymbal Bank worked example (§14) shows the layers assembled into one deployment. The alternative — a cluster that "works" but whose controls are undocumented — is precisely the estate a supervisory examination flags (cross-ref [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) for the examination frame).

---

## 2. The Host Layer — SELinux, CRI-O and FIPS Mode

### 2.1 SELinux — the Mandatory Access Control Layer

SELinux (Security-Enhanced Linux) is enabled and enforcing on OpenShift 4 nodes by default, and it is the host layer's core isolation mechanism. The OpenShift-specific part is **MCS (Multi-Category Security) labeling**: each namespace is allocated an MCS label, and containers are launched with a pre-allocated SELinux context tied to that namespace. The consequence is a strong containment property — a process that escapes its container can only access files and sockets carrying its own MCS category, not the node's or another namespace's. The OpenShift documentation states the contract directly: the `restricted` SCC "requires pods to be run with a UID, and SELinux context that are allocated to the namespace" ✅ (4.18 SCC chapter), and the admission controller "look[s] up pre-allocated values from a namespace and populate[s] the SCC before processing the pod" ✅ — that pre-allocation is the MCS label. The SCC's `seLinuxContext` strategy field is one of the three strategy categories (§4.4). **⚠-structural note:** the claim "SELinux is enabled and enforcing by default on RHCOS" is uniformly documented Red Hat practice but was not separately extracted at a primary source this pass; the namespace MCS pre-allocation itself is ✅ via the SCC chapter.

### 2.2 CRI-O — the Kubernetes-Native Container Runtime

**CRI-O is the default — and on OpenShift 4, effectively the only — container engine**, replacing the Docker engine that OpenShift 3 used ✅ (Red Hat blog). Verified lineage from the same blog: CRI-O was available in OpenShift 3.7 as Tech Preview and 3.9 as Generally Available, and Red Hat had already run it for production workloads in OpenShift Online since 3.10 ✅. On OpenShift 4, CRI-O is not an option among runtimes; it is the engine the nodes are manufactured with, versioned in lock-step with Kubernetes (the blog's "the version of CRI-O maps directly to the version of Kubernetes" ✅).

Why this matters for security:

- **CRI compliance by construction.** CRI-O exists to implement the Kubernetes CRI ("CRI-O's purpose is to be the container engine that implements the Kubernetes Container Runtime Interface" ✅, Red Hat docs) — no Docker daemon, no Docker socket, no privileged daemon with broad host access sitting next to the kubelet. The Docker socket attack surface (a classic container-escape vector) does not exist on OpenShift 4.
- **OCI runtimes underneath.** CRI-O launches OCI-compliant runtimes — runc by default, with Kata Containers available for stronger VM-based isolation ✅ (Red Hat docs 3.11: "engaging OCI-compliant runtimes like runc, the default OCI runtime, or Kata Containers"). For a bank's highest-risk workloads, Kata adds a hardware-virtualization boundary beneath the container.
- **Capability defaults.** CRI-O ships a defined default capability list per container (CHOWN, DAC_OVERRIDE, FSETID, FOWNER, SETGID, SETUID, SETPCAP, NET_BIND_SERVICE, KILL) ✅ (4.18 SCC chapter) — and the SCC layer can require dropping ALL of them (`requiredDropCapabilities: ALL`, which is what the `restricted-v2` SCC does ✅).

### 2.3 FIPS Mode — the Cryptographic Floor

OpenShift Container Platform can be installed in **FIPS mode** — the cryptographic floor for any regulated workload, and effectively mandatory for government and heavily regulated sectors ✅ (docs.redhat.com, 4.21 "Support for FIPS cryptography"):

- **Install-time only.** "Because FIPS must be enabled before the operating system that your cluster uses boots for the first time, you cannot enable FIPS after you deploy a cluster" ✅. Set `fips: true` in `install-config.yaml` before deployment, and use a FIPS-capable installation program (extracted via `oc adm release extract --command=openshift-install-fips`) ✅.
- **The crypto stack.** When RHEL/RHCOS boot in FIPS mode, OpenShift core components use the RHEL cryptographic libraries submitted to NIST for FIPS 140-2/140-3 validation, on x86_64, ppc64le and s390x only ✅. "All OpenShift Container Platform components that require cryptographic signing call RHEL and RHCOS cryptographic libraries" ✅.
- **etcd interplay.** In FIPS mode, etcd encryption at rest uses the FIPS-approved AES-CBC algorithm (§3.2) ✅.
- **Honest limits.** The docs' own limitation table: TLS FIPS support "is not complete but is planned for future releases," and the FIPS implementation does not use single-step hash-and-sign functions ✅ — a bank's crypto team should read the limitation table per release rather than assume blanket FIPS validation.

### 2.4 The Host-Layer Table — Control / Mechanism / Verified

| Control | Mechanism | Verified |
|---|---|---|
| SELinux | Enforcing on nodes; MCS labels pre-allocated per namespace; SELinux context strategy in SCCs | ✅ §2.1 |
| Container engine | CRI-O only; implements the Kubernetes CRI; no Docker daemon/socket; OCI runtimes (runc default, Kata optional) | ✅ §2.2 |
| Capability baseline | CRI-O default capability list; SCCs can require dropping ALL | ✅ §2.2, §4 |
| FIPS mode | `fips: true` at install; FIPS 140-2/140-3-submitted RHEL/RHCOS crypto libs; x86_64/ppc64le/s390x; not enabled post-deploy | ✅ §2.3 |
| Host config as code | MachineConfig objects via the Machine Config Operator | ✅-structural §1.2 |

---

## 3. The Data Layer — etcd Encryption at Rest and Key Management

### 3.1 What etcd Holds — and Why It Is the Crown Jewels

etcd is the cluster's source of truth: every object the API servers persist lives there. The OpenShift documentation is explicit about what is at stake: **"By default, etcd data is not encrypted in OpenShift Container Platform. You can enable etcd encryption for your cluster to provide an additional layer of data security. For example, it can help protect the loss of sensitive data if an etcd backup is exposed to the incorrect parties"** ✅ (4.12 "Encrypting etcd data"). For a bank, the "exposed backup" scenario is not hypothetical — backup media, snapshot stores and clone environments are a recurring data-leak vector, and the MAS data-protection expectations (cross-ref §13) treat backup exposure as a first-class risk. Note the scope: etcd encryption encrypts **values, not keys** — "resource types, namespaces, and object names are unencrypted" ✅.

### 3.2 Enabling etcd Encryption — the Mechanism

Encryption is a cluster-level configuration, not per-secret:

1. Edit the `APIServer` object: `oc edit apiserver`.
2. Set `spec.encryption.type: aescbc` — "AES-CBC with PKCS#7 padding and a 32 byte key" ✅.
3. The encryption process runs in the background — "It can take 20 minutes or longer for this process to complete, depending on the size of your cluster" ✅ — and the operators report progress via `Encrypted` status conditions on the OpenShift API server (`EncryptionCompleted`), the Kubernetes API server and the OAuth API server ✅.

When enabled, the following resources are encrypted in etcd ✅: **Secrets, Config maps, Routes, OAuth access tokens, OAuth authorize tokens**. That list is exactly the bank-sensitive set — credentials, routing configuration and the OAuth tokens that gate administrative sessions. Disabling is the inverse (`type: identity`, the default, meaning no encryption) ✅.

### 3.3 Key Rotation and Backup Keys

The key-management story is deliberately simple and operationally safe:

- **Keys rotate automatically every 7 days** ✅.
- **Up to 10 historical encryption keys are preserved after rotation** — "to facilitate the decryption of older backups and provide an extra layer of data recovery safety" ✅.
- **Backup coupling:** if etcd encryption was enabled at backup time, the `_static_kuberesources_<datetimestamp>.tar.gz` file in the backup contains the encryption keys for the snapshot — "For security reasons, store this file separately from the etcd snapshot. However, this file is required to restore a previous state of etcd" ✅. This is a two-person rule in practice: the keys file and the snapshot must not travel together, or the encryption is theater.

### 3.4 External Key Management — KMS v2

For the regulated bank, in-cluster key storage is a control, but the key-management boundary the auditors look for is external: "You can configure Kubernetes Key Management Service (KMS) v2 on OpenShift Container Platform to centralize encryption key management and meet regulatory compliance requirements" ✅ (docs.redhat.com, 4.22 "External encryption key management"). KMS v2 moves the actual key material to an external KMS (the pattern the [Cybersecurity](cybersecurity_guide.md) and the enterprise key-management estates expect), so that a compromised cluster cannot exfiltrate its own encryption keys. Note the compliance-operator interplay: automatic remediation for API-server encryption applies AES-GCM on OpenShift 4.13+, AES-CBC on earlier versions ✅ (Compliance Operator 1.9 release notes) — the compliance layer actively enforces the encryption baseline (§9).

### 3.5 The Data-Layer Table

| Property | Value | Verified |
|---|---|---|
| Default state | Not encrypted; opt-in via `APIServer` object | ✅ §3.2 |
| Cipher | AES-CBC (aescbc), PKCS#7 padding, 32-byte key; AES-GCM on 4.13+ via compliance remediation | ✅ §3.2, §9 |
| Encrypted resources | Secrets, ConfigMaps, Routes, OAuth access/authorize tokens | ✅ §3.2 |
| Rotation | Every 7 days; up to 10 historical keys retained | ✅ §3.3 |
| Backup keys | `_static_kuberesources_*.tar.gz`; store separately from the snapshot | ✅ §3.3 |
| External KMS | KMS v2 supported (4.22 docs) for centralized key management | ✅ §3.4 |
| FIPS interplay | FIPS-approved AES-CBC when nodes boot in FIPS mode | ✅ §2.3 |

---

## 4. The Cluster Security Layer — Security Context Constraints

### 4.1 What an SCC Is — the Pod Permission Contract

**A Security Context Constraint (SCC) is OpenShift's pod-level permission contract: a cluster-scoped object that defines the set of conditions a pod must run under to be accepted into the system.** The documentation's framing: "Similar to the way that RBAC resources control user access, administrators can use security context constraints (SCCs) to control permissions for pods. These permissions determine the actions that a pod can perform and what resources it can access" ✅ (4.18 "Managing security context constraints"). An SCC controls, among other things ✅: whether a pod can run privileged containers (`allowPrivilegedContainer`), whether privilege escalation is allowed (`allowPrivilegeEscalation`), the Linux capabilities a container can request, host-directory volumes, the SELinux context, the container user ID, host namespaces and networking, FSGroup allocation, supplemental groups, read-only root filesystems, allowable volume types and allowable seccomp profiles. In other words: the SCC is the object that turns "least privilege" from a slogan into an admission decision. The deep mechanism detail lives in the dedicated sibling guides — [openshift_scc_comprehensive_guide.md](openshift_scc_comprehensive_guide.md) and [openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md) — this section covers the verified mechanism and the operational rules.

**Operational rule one: do not modify the default SCCs.** "Customizing the default SCCs can lead to issues when some of the platform pods deploy or OpenShift Container Platform is upgraded. Additionally, the default SCC values are reset to the defaults during some cluster upgrades" ✅. The supported pattern is to create custom SCCs and leave the defaults alone.

### 4.2 The Admission Mechanism — RBAC-Granted, Priority-Ordered

The SCC admission mechanism has three verified properties that together define how a pod gets its constraints:

1. **RBAC-granted.** An SCC does nothing by itself — a user or service account must be *granted* the right to use it. The admission controller "can inspect the user information made available in the context to retrieve an appropriate set of SCCs," and "the set of SCCs that admission uses to authorize a pod are determined by the user identity and groups that the user belongs to. Additionally, if the pod specifies a service account, the set of allowable SCCs includes any constraints accessible to the service account" ✅ (4.18). The grant is the RBAC `use` verb on the SCC resource — the `restricted` SCC, for example, is "no longer available to users of new OpenShift Container Platform 4.11 or later installations, unless the access is explicitly granted" ✅. This is the property that makes SCCs auditable: every pod's privileges trace back to a RoleBinding/ClusterRoleBinding.
2. **Admission-time.** SCCs are enforced by the admission controller when the pod is created or updated — the pod is validated against the retrieved SCC set before it is accepted ✅ (§4.2 mechanism above; the "admission controller" language is the docs' own).
3. **Priority-ordered selection.** "Security context constraints (SCCs) have a priority field that affects the ordering when attempting to validate a request by the admission controller" ✅. "A priority value of `0` is the lowest possible priority. A nil priority is considered a `0`, or lowest, priority. Higher priority SCCs are moved to the front of the set when sorting" ✅ — the controller tries SCCs in priority order and uses the first one that validates the pod. The docs' warning: "Setting an SCC priority greater than 0 for the default OpenShift Container Platform SCCs can cause critical cluster instability" ✅ — priority tuning belongs on custom SCCs. The `privileged` SCC note is the canonical illustration: "Setting `privileged: true` in the pod specification does not necessarily select the `privileged` SCC. The SCC that has `allowPrivilegedContainer: true` and has the highest prioritization will be chosen if the user has the permissions to use it" ✅ — pod spec does not choose; admission does, within the granted set.

The admission controller also performs **value pre-allocation**: "The admission controller is aware of certain conditions in the security context constraints (SCCs) that trigger it to look up pre-allocated values from a namespace and populate the SCC before processing the pod" ✅ — this is how the namespace's UID range and MCS label get injected into the pod's security context (§2.1).

### 4.3 The Default SCCs — the 4.18 Catalog

The default SCC set installed with the cluster (verified from the 4.18 table) ✅:

| SCC | Nature |
|---|---|
| `anyuid` | All features of `restricted`, but allows any UID/GID |
| `hostaccess` | All host namespaces; UID/SELinux still namespace-allocated. "Should only be used by trusted pods" |
| `hostmount-anyuid` | Host mounts + any UID, including UID 0. "Grant with caution" |
| `hostnetwork` | Host networking and host ports; UID/SELinux still namespace-allocated |
| `hostnetwork-v2` | Like `hostnetwork` but drops ALL capabilities, `seccompProfile: runtime/default`, no privilege escalation |
| `node-exporter` | For the Prometheus node exporter; host filesystem access |
| `nonroot` | All features of `restricted`, but any non-root UID |
| `nonroot-v2` | Like `nonroot`, hardened (ALL capabilities dropped, runtime/default seccomp, no escalation) |
| `privileged` | Everything: privileged pods, host namespaces, any user, any MCS label, any seccomp, any capability. "The most relaxed SCC and should be used only for cluster administration" |
| `restricted` | Denies all host features; requires namespace-allocated UID, MCS label and FSGroup |
| `restricted-v2` | Like `restricted`, hardened: ALL capabilities dropped, `seccompProfile: runtime/default` by default, `allowPrivilegeEscalation` must be false. "The most restrictive SCC provided by a new installation and will be used by default for authenticated users" |

The `-v2` family (introduced with the 4.11 Pod Security Admission work, §5.4) is the modern default posture: **a new 4.11+ install defaults authenticated users to `restricted-v2`**, not to the older `restricted` ✅. The Cymbal Bank strategy (§14) builds on this.

### 4.4 SCC Strategy Fields — Boolean, Allowable-Set, Strategy

SCC settings fall into three categories ✅ (4.18):

- **Controlled by a boolean** — fields that default to the most restrictive value ("`AllowPrivilegedContainer` is always set to `false` if unspecified" ✅).
- **Controlled by an allowable set** — fields checked against a set of permitted values.
- **Controlled by a strategy** — fields with a strategy that both generates a value (when unset) and validates a specified value against the allowable set — e.g. the `RunAsUser`, `SELinuxContext`, `FSGroup` and `SupplementalGroups` strategies ✅.

This three-way design is why SCCs can both *enforce* and *complete*: the strategy fields fill in the values a pod did not request (namespace UID range, MCS label) while the booleans and allowable sets reject what the pod should not have.

### 4.5 The SCC Table — Default SCC / What It Allows / Warning

| Default SCC | Allows | Warning (docs' own) |
|---|---|---|
| `restricted-v2` | Non-privileged, namespace UID/MCS/FSGroup, seccomp runtime/default, ALL capabilities dropped | None — the 4.11+ default for authenticated users |
| `restricted` | Non-privileged, namespace UID/MCS/FSGroup | No longer default-granted on 4.11+ installs |
| `nonroot` / `nonroot-v2` | `restricted` features + any non-root UID (v2 hardened) | v2 preferred |
| `anyuid` | `restricted` features + any UID/GID | Grant deliberately |
| `hostnetwork` / `hostnetwork-v2` | Host networking/ports (v2 hardened) | A `hostnetwork` workload on a control-plane host "is effectively root on the cluster" |
| `hostmount-anyuid` | Host mounts + any UID incl. root | "Grant with caution" |
| `hostaccess` | All host namespaces | "Should only be used by trusted pods" |
| `privileged` | Everything | "Should be used only for cluster administration" |

---

## 5. The Cluster Security Layer — RBAC, Admission Control and Pod Security Admission

### 5.1 RBAC — Roles, Cluster Roles and Bindings

OpenShift inherits Kubernetes RBAC wholesale: **Roles and ClusterRoles** define permission sets (verbs on resources), **RoleBindings and ClusterRoleBindings** attach them to users, groups or service accounts ✅-structural (standard Kubernetes RBAC, documented throughout docs.redhat.com's authentication-and-authorization book, which this pass verified for SCCs and PSA but not re-extracted for the RBAC basics). Three OpenShift-specific security points worth stating:

1. **Service accounts are identities.** Every pod runs as a service account, and that service account's RBAC grants — including its SCC `use` grants (§4.2) — determine what the pod can do. The dedicated sibling [openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md) goes deep here.
2. **Cluster-admin is the crown.** The `cluster-admin` ClusterRole is the platform's root; the SCC chapter's warnings about `privileged` and `hostnetwork` are all warnings about *who* holds cluster-admin-equivalent power. In a bank, cluster-admin sessions are the equivalent of a production-DB admin session — break-glass, monitored, and logged (§10).
3. **RBAC is the audit unit.** The admission mechanism makes every privilege traceable to a binding (§4.2); the audit layer (§10) makes every use of a privilege traceable to a request. RBAC is the spine that connects them.

### 5.2 Admission Control — Built-in Plugins and Webhooks

Admission control is the cluster's decision point: built-in admission plugins (compiled into the API servers) plus dynamically-registered **validating and mutating admission webhooks** evaluate every request before it is persisted ✅-structural (Kubernetes admission-control architecture; OpenShift's SCC admission and the pod-security admission it runs are themselves admission plugins — see §4.2 and §5.3, which are ✅ at the docs). The SCC admission controller is the OpenShift-native admission plugin; Pod Security Admission is the Kubernetes-native one that OpenShift 4.11 turned on (below). For the bank, webhooks are where policy-as-code lands: image-signature enforcement (§7.4), label enforcement, and the RHACS policy enforcement (§11) all hook admission. The operational rule for a regulated estate: **mutating webhooks are a higher-risk class than validating ones** — a mutating webhook rewrites requests, so it must itself be treated as a trusted platform component with change control (⚠-structural, standard admission-control practice).

### 5.3 Pod Security Admission and Pod Security Standards

**Pod Security Admission (PSA) is the Kubernetes built-in admission controller that implements the three Pod Security Standards (PSS) levels — `privileged`, `baseline`, `restricted`** ✅ (docs 4.11 "Understanding and managing pod security admission": "Pod security admission is an implementation of the Kubernetes pod security standards. Use pod security admission to restrict the behavior of pods"). PSA is namespace-scoped via labels (`pod-security.kubernetes.io/enforce`, `-warn`, `-audit`) and supports a graduated rollout: warn, then audit, then enforce ✅ (the Red Hat 4.11 blog's "iterative approach": "a cluster administrator can turn on just client logging, audit event logging and finally even cluster-wide enforcement of a specific pod security level" ✅).

The historical context the Red Hat blog documents explicitly ✅: PSA is the replacement for the old **PodSecurityPolicy (PSP)** admission, which Kubernetes deprecated in v1.21 and **removed in v1.25** ✅ (kubernetes.io: "PodSecurityPolicy was deprecated in Kubernetes v1.21, and removed from Kubernetes in v1.25"). OpenShift never adopted PSP as its primary pod-permission mechanism — SCCs predate and outlived it — which is why the SCC↔PSA relationship (§5.4) matters: OpenShift runs *both* admission systems, coordinated.

### 5.4 The SCC ↔ PSA Relationship — the 4.11 Synchronization

The verified design (docs 4.11 + Red Hat blog, "Pod Security Admission in OpenShift 4.11") ✅:

- **Both admission systems run.** "In OpenShift, there is an OpenShift-specific dedicated pod admission system called Security Context Constraints. This system resembles the now deprecated PodSecurityPolicy admission... Our aim is to keep the Security Context Constraints pod admission system while also allowing users to have access to the Kubernetes Pod Security Admission" ✅.
- **Global defaults are permissive-with-warnings.** "With OpenShift 4.11, we are turning on the Pod Security Admission with global `privileged` enforcement. Additionally we set the `restricted` profile for warnings and audit" ✅ — i.e., out of the box nothing is blocked by PSA, but violations are surfaced as warnings and audit events (`PodSecurityViolation` alert ✅, §10.1), and namespaces can opt in to enforcement via the `pod-security.kubernetes.io/enforce` label ✅.
- **A controller synchronizes SCC permissions into PSA labels.** "The controller examines `ServiceAccount` object permissions to use security context constraints in each namespace. Security context constraints (SCCs) are mapped to pod security profiles based on their field values... Pod security admission `warn` and `audit` labels are set to the most privileged pod security profile found in the namespace" ✅. The mapping is per-namespace-label: `security.openshift.io/scc.podSecurityLabelSync` controls it (default on for user namespaces, off for `openshift-`-prefixed system namespaces) ✅.
- **New hardened SCCs.** The 4.11 work added the `-v2` SCC family (§4.3) to meet the pod security standards — `restricted-v2` is the default for authenticated users on new installs, and the older `restricted` SCC lost its default grant ✅.

**The practical reading for a bank:** PSA gives you the Kubernetes-standard vocabulary (privileged/baseline/restricted) for reporting and interop, while SCCs remain the OpenShift-native enforcement with the deeper field-level control. A namespace that is `restricted` by both systems is the target state; the synchronization controller makes the two views consistent so that a team does not get whipsawed by conflicting admissions.

### 5.5 Namespace Isolation — the Tenant Boundary

Namespaces are the cluster's tenant boundary, and OpenShift hardens the boundary in three ways that this pass verified at the docs: (1) **MCS label isolation** — each namespace gets its own SELinux category (§2.1), so even a broken-out container cannot read another namespace's files; (2) **pre-allocated UID ranges and FSGroups per namespace** (§4.2's pre-allocation) — workloads in different namespaces run as different UIDs, preventing UID-collision attacks on shared storage; (3) **PSA label synchronization per namespace** (§5.4) — the pod-security posture is a namespace property. Add NetworkPolicy (§6) and RBAC (only the namespace's service accounts and users can touch its objects), and the namespace becomes a credible tenant boundary — the unit the bank's environment-segregation policy (dev/test/prod, cross-ref §13) maps onto.

### 5.6 The Cluster-Security Table

| Mechanism | What it enforces | Verified |
|---|---|---|
| SCCs | Pod permission contract: privilege, capabilities, SELinux, UID, volumes, seccomp — RBAC-granted, admission-time, priority-ordered | ✅ §4 |
| RBAC | Who may do what on which resource, for users, groups and service accounts | ✅-structural §5.1 |
| Admission plugins & webhooks | Request-time policy: SCC admission, PSA, image/trust, RHACS enforcement | ✅ §5.2–§5.4 |
| Pod Security Admission | Kubernetes pod security standards (privileged/baseline/restricted) per namespace, warn→audit→enforce | ✅ §5.3 |
| PSA↔SCC sync | Label controller mapping SCC permissions to PSA warn/audit labels | ✅ §5.4 |
| Namespace isolation | MCS labels, pre-allocated UID/FSGroup, PSA labels, RBAC scoping | ✅ §5.5 |

---

## 6. The Network Layer — NetworkPolicy and Namespace Isolation

### 6.1 OVN-Kubernetes — the Default Network Provider

**OVN-Kubernetes is the default CNI (Container Network Interface) network provider for the OpenShift cluster network** ✅ (docs.redhat.com, "OVN-Kubernetes default CNI network provider": "The OpenShift Container Platform cluster uses a virtualized network for pod and service networks. The OVN-Kubernetes Container Network Interface (CNI) plugin is a network provider for the default cluster network. OVN-Kubernetes is based on Open Virtual Network (OVN)... provides an overlay-based networking implementation"). Two security-relevant properties: the overlay isolates pod traffic from the host network, and OVN-Kubernetes is the enforcement point for NetworkPolicy (below) and for the network-segmentation policies that RHACS can generate (§11.2). For the bank, the overlay + NetworkPolicy combination is the micro-segmentation substrate that the zero-trust estate expects (cross-ref the [Cybersecurity](cybersecurity_guide.md) Zero Trust discussion).

### 6.2 NetworkPolicy — the Micro-Segmentation Primitive

NetworkPolicy is the Kubernetes-native object for pod-to-pod and pod-to-service traffic rules: `ingress` and `egress` rules with podSelector/namespaceSelector/IPBlock selectors ✅-structural (standard Kubernetes NetworkPolicy, enforced by the OVN-Kubernetes provider on OpenShift). The security doctrine for a bank estate is the **default-deny namespace**: create the namespace, apply a deny-all NetworkPolicy, then open exactly the paths the workloads need — and treat every opened path as a reviewed, documented dependency. The RHACS network-segmentation capability (§11.2) generates exactly these policies from observed traffic, which is the evidence-backed way to build the default-deny mesh without guessing.

### 6.3 The Network Table

| Control | Mechanism | Verified |
|---|---|---|
| Default network provider | OVN-Kubernetes CNI, OVN-based overlay | ✅ §6.1 |
| Micro-segmentation | NetworkPolicy ingress/egress, default-deny namespaces | ✅-structural §6.2 |
| Generated segmentation | RHACS network policies from observed flows | ✅ §11.2 |
| Ingress | OpenShift Routes (HTTP) + Ingress; TLS termination at the edge | ✅-structural §1.2 |

---

## 7. The Workload Security Layer — Image Signing and Trust

### 7.1 Why Image Trust Is a Bank Control

Every workload on the cluster starts as a container image, and an image is an immutable bundle of code, libraries and configuration. For a bank, the supply-chain question is not "do we scan images" but "**can a tampered image even enter the cluster?**" — which is a signing-and-verification question, not a scanning one. Scanning (Clair, §12.2; RHACS, §11) finds *known* vulnerabilities; signing establishes *who built it and that it is unchanged*. The two are complementary, and the regulated estate needs both: scanning for risk, signing for provenance. This section covers the Red Hat signed-image infrastructure and the enforcement options; the broader supply-chain discipline (SBOM, provenance, SLSA-style integrity) is covered in the [Cybersecurity](cybersecurity_guide.md) DevSecOps sections and cross-ref'd in §8.1 rather than re-derived here.

### 7.2 Red Hat's Signed-Image Infrastructure

Verified facts about how Red Hat signs and serves the images in its own registry ✅:

- **Red Hat delivers signatures for images in the Red Hat Container Registry** — "Red Hat delivers signatures for the images in the Red Hat Container Registry," and trust is configured with (1) the registry/repository name, (2) the Red Hat release public GPG key (the same key used to verify RPMs), and (3) a signature server — with signature stores served from `https://access.redhat.com/webassets/docker/content/sigstore` and `https://registry.redhat.io/containers/sigstore` ✅ (access.redhat.com article 3116561, "Verifying image signing for Red Hat Container Registry"). The article's own hardening note is the bank-relevant one: "For robust security the default trust policy should be set to reject by default" (`podman image trust set -t reject default`) ✅.
- **The lineage is long-standing.** Red Hat has been serving signatures for Red Hat Container Catalog images since the "simple signing" era — the "Signed Images from the Red Hat Container Catalog" blog ✅ (the blog's existence and framing verified via search; the mechanism details via article 3116561).
- **The RHEL toolchain signs in sigstore format.** RHEL 8's container-tools documentation describes `--sign-by-sigstore-private-key`, which "adds a sigstore signature... The image and the sigstore signature are uploaded to the remote registry" ✅ (RHEL 8 "Signing container images" docs) — i.e., the platform's native tooling produces sigstore-format signatures.
- **Red Hat Trusted Artifact Signer (RHTAS)** is Red Hat's "production-ready deployment of the Sigstore project for enterprise use," whose technical preview was announced on **15 November 2023** ✅ (the RHTAS/Enterprise Contract blog linking the developers.redhat.com announcement) — the enterprise cosign/fulcio/rekor stack, deployable on OpenShift, giving the bank an in-house sigstore for its *own* images.

**⚠ on one claim:** the widely-repeated statement that "Red Hat began signing UBI/RHEL images with cosign in ~2022" could not be pinned to a primary source this pass — the cosign-format signing of Red Hat's own registry images is documented via the sigstore signature-store paths (above) but the specific 2022 announcement was not found; it is flagged ⚠ in the ledger and detailed in "What Could Not Be Verified".

### 7.3 Sigstore and cosign — the Signature Formats

The sigstore project's tooling — **cosign** for signing and verifying OCI artifacts, with the Fulcio certificate authority and Rekor transparency log — is the de-facto standard for container signing ✅-structural (sigstore.dev; the cosign/RHTAS integration is documented in Red Hat's own "How to integrate Cosign image signature with RHACS" blog ✅). For the bank's own images, the pattern is: CI signs the image with cosign (using either a key pair or RHTAS-managed certificates), the signature is stored alongside the image in the registry (Quay supports this — §12), and admission-time verification (§7.4) refuses unsigned images. OpenShift 4.19's `oc-mirror` v2 supports mirroring images *and their cryptographic signatures* into disconnected registries ✅ (developers.redhat.com, "How to verify container signatures in disconnected OpenShift") — the air-gapped bank pattern is explicitly supported.

### 7.4 Enforcing Signature Verification — policy.json and Admission

Verification enforcement happens at two points:

1. **Node/host pull-time** — the containers/image policy (`/etc/containers/policy.json` plus per-registry config in `/etc/containers/registries.d/`) tells the runtime which registries must be `signedBy` which keys ✅ (article 3116561: the `signedBy` / `keyType: GPGKeys` policy entries, and the "reject by default" recommendation). On OpenShift, CRI-O consults this policy when pulling images — a node configured with a strict policy will refuse unsigned pulls ✅-structural (the policy mechanism is the containers/image standard; the article documents the RHEL-side configuration).
2. **Admission-time** — image-signature and trust policies enforced through admission webhooks (image policy admission plugins or policy engines such as RHACS' enforce-at-deploy controls, §11.2) so that even a *referenced* unsigned image cannot become a running pod ✅-structural (admission-based image policy is standard practice; RHACS' deployment-time enforcement is documented in the RHACS product materials).

The bank pattern is belt-and-braces: strict `policy.json` on the nodes (reject by default), admission enforcement for deploy-time, and the registry as the single trusted source (§12.3's mirroring pattern).

### 7.5 The Image-Trust Table

| Capability | Mechanism | Verified |
|---|---|---|
| Red Hat registry signatures | Signatures served for registry images; sigstore signature stores; trust via public key + signature server | ✅ §7.2 |
| Reject-by-default posture | `podman image trust set -t reject default` | ✅ §7.2 |
| Sigstore-format tooling | RHEL `--sign-by-sigstore-private-key`; cosign; RHTAS (Sigstore deployment, TP 15 Nov 2023) | ✅ §7.2–§7.3 |
| Pull-time enforcement | `/etc/containers/policy.json` `signedBy` policy on nodes | ✅ §7.4 |
| Deploy-time enforcement | Admission webhooks / RHACS enforce-at-deploy | ✅-structural §7.4 |
| Disconnected pattern | `oc-mirror` v2 mirrors images + signatures (4.19) | ✅ §7.3 |

---

## 8. The Workload Security Layer — Supply Chain, Secrets and Runtime Security

### 8.1 The Supply-Chain Discipline — SBOM, Provenance, CI/CD Gates

The end-to-end supply-chain discipline — SBOMs, provenance, SLSA-style integrity levels, CI/CD security gates, secret hygiene in pipelines — is the subject of the [Cybersecurity](cybersecurity_guide.md) DevSecOps chapters and is **not re-derived here**; this section only maps it onto the OpenShift platform. The platform-side anchors: image signing and trust (§7) as the artifact-integrity gate; the integrated registry (§12) as the single ingestion point; RHACS' build-phase scanning and CI integrations (§11.2) as the vulnerability gate; and the admission layer (§5.2) as the enforcement point that turns pipeline policy into cluster fact. For a MAS-supervised bank, the SBOM-plus-signature pair is the *evidence* that a deployed image is exactly what the change record says it is — the artifact-level link between change management and runtime state (cross-ref §13.3).

### 8.2 Secrets Management — Encrypted at Rest, External Stores

The secrets story has three verified layers:

1. **Secrets at rest in etcd.** Kubernetes `Secret` objects are encrypted in etcd when etcd encryption is enabled — Secrets are the first item on the platform's own encrypted-resources list ✅ (§3.2). This is the baseline control: no plaintext secrets on disk in the cluster's backing store.
2. **The platform's own secret distribution.** OpenShift injects service-account tokens and the internal registry credentials into pods via the API; the internal image-registry credentials and the OAuth tokens are part of the encrypted set (§3.2) ✅.
3. **External secret stores.** The **External Secrets Operator for Red Hat OpenShift** "operates as a cluster-wide service... integrates with external secrets management systems and performs secret fetching, refreshing, and provisioning within the cluster," supporting **AWS Secrets Manager, HashiCorp Vault, Google Secrets Manager, Azure Key Vault, IBM Cloud Secrets Manager and AWS Systems Manager Parameter Store** ✅ (docs.redhat.com 4.19 + the openshift/external-secrets-operator repository). This is the bank pattern: the cluster holds a reference (`ExternalSecret` → `SecretStore`), the store holds the material, and the operator syncs it — so a cluster compromise does not equal a credential compromise, and rotation happens at the store.

The operational rules for the bank estate: never put secrets in image layers or ConfigMaps; treat every `Secret` as encrypted-at-rest but *short-lived where possible*; prefer store-side rotation with operator propagation; and audit both sides (store access logs + cluster audit events, §10).

### 8.3 Runtime Security — seccomp, Capabilities, SELinux, eBPF

Runtime security is the last line inside the workload — what a *running* container may do, and what watches it:

- **seccomp.** The `restricted-v2` SCC sets `seccompProfile: runtime/default` by default, and SCCs can constrain allowable seccomp profiles ✅ (§4.3 — "The configuration of allowable seccomp profiles" is in the SCC control list ✅). `runtime/default` applies the container runtime's default seccomp profile — the standard blocklist of dangerous syscalls.
- **Capabilities.** CRI-O's default capability list (§2.2) is the starting set; the SCC layer can require `ALL` capabilities dropped (`restricted-v2` does exactly that) ✅. The container runs with the minimal capability set it was granted, not root's.
- **SELinux.** The MCS-labeled SELinux context (§2.1) constrains even a compromised process at the kernel level — the deepest runtime boundary the platform has.
- **eBPF monitoring.** The platform and its security products use eBPF for runtime observability: "The advanced cluster security operator relies on eBPF for collecting runtime cluster information" ✅ (developers.redhat.com, "Using eBPF in Red Hat products"), and RHACS' Collector component uses kernel-level monitoring (§11.3) ✅-structural (the Collector's eBPF/kernel-driver modes are documented in the RHACS troubleshooting docs). eBPF-based detection observes syscall-level behavior without modifying the workload — the detection layer above the prevention layers.

### 8.4 The Workload-Security Table

| Control | Mechanism | Verified |
|---|---|---|
| Secrets at rest | etcd encryption covers Secrets (plus ConfigMaps, Routes, OAuth tokens) | ✅ §3.2 |
| External secrets | External Secrets Operator: AWS SM, Vault, GCP SM, Azure KV, IBM SM, SSM Parameter Store | ✅ §8.2 |
| seccomp | `runtime/default` default in `restricted-v2`; SCC-controlled allowable profiles | ✅ §4.3, §8.3 |
| Capabilities | CRI-O default list; `ALL` dropped by `restricted-v2` | ✅ §2.2, §4.3 |
| SELinux | MCS labels per namespace; enforced at kernel level | ✅ §2.1 |
| Runtime detection | RHACS collector (eBPF/kernel mode) | ✅ §8.3, §11.3 |

---

## 9. The Compliance Layer — the Compliance Operator

### 9.1 What the Compliance Operator Is — and Is Not

**The Compliance Operator automates the inspection of a cluster's technical implementations against industry standards, benchmarks and baselines — it is not an auditor.** The docs' own words: "The Compliance Operator assists users by automating the inspection of numerous technical implementations and compares those against certain aspects of industry standards, benchmarks, and baselines. **The Compliance Operator is not an auditor.** To be compliant or certified under these various standards, you need to engage an authorized auditor such as a Qualified Security Assessor (QSA), Joint Authorization Board (JAB), or other industry recognized regulatory authority to assess your environment" ✅ (4.20 Compliance Operator chapter). That framing is exactly right for a bank: the operator produces *evidence of control state*, and the regulator or its appointed assessor renders the *judgment*. It "lets OpenShift Container Platform administrators describe the required compliance state of a cluster and provides them with an overview of gaps and ways to remediate them" ✅.

### 9.2 The Mechanism — ComplianceScan, ScanSetting, OpenSCAP Under the Hood

The operator is declarative, like the rest of OpenShift — the administrator describes the desired compliance state, and the operator executes scans and reports results ✅ (the CRD model below is from the 4.20 chapter's custom-resource-definitions section ✅):

- **`Profile`** — a named set of rules (the XCCDF profile), e.g. the CIS OpenShift profile or a NIST 800-53 baseline.
- **`ScanSetting`** — *how* to run scans: schedule, storage of raw results (`rawResultStorage`), tolerations, and the roles/accounts used (the operator runs scans under a service account with the privileges the scan needs — a security-sensitive design point in itself).
- **`ScanSettingBinding`** — binds one or more profiles to a `ScanSetting` — the "run these profiles this way" object.
- **`ComplianceScan`** — a single scan instance; **`ComplianceSuite`** groups scans; **`ComplianceCheckResult`** and **`ComplianceRemediation`** are the outputs — per-rule pass/fail results and generated remediations.
- **`TailoredProfile`** — a profile customized for the environment (rules enabled/disabled/with adjusted values), the sanctioned way to handle legitimate deviations ✅.

**Under the hood, the engine is OpenSCAP.** The operator runs OpenSCAP-based content (XCCDF profiles and OVAL checks) against the platform (`ocp4` content) and the nodes (`rhcos4` content for RHCOS) ✅-structural (the OpenSCAP engine relationship is documented across the operator's release notes — the `rhcos4-*` rule names and XCCDF references are pervasive in the 4.20 chapter ✅ — and the engine internals are the subject of the sibling [openscap_guide.md](openscap_guide.md)). The operator also ships a CEL-based `CustomRule` scanner in Tech Preview for custom checks without XCCDF ✅ (1.9 release notes).

### 9.3 The Supported Profiles — CIS, NIST 800-53, PCI-DSS, STIG and the Rest

The supported-profile set verified from the 4.20 chapter and its release notes ✅:

| Profile family | Verified details |
|---|---|
| **CIS** | CIS OpenShift benchmarks — e.g. CIS 1.7.0/1.9.0 support (the `ocp4-cis-*` rules) ✅ |
| **NIST 800-53** | NIST 800-53 Moderate and High baselines; the "NIST 800-53 Moderate-Impact Baseline for OpenShift Container Platform platform and node profiles" ✅ (1.3.0 release note); FedRAMP Moderate Revision 4 ✅ |
| **PCI-DSS** | PCI-DSS 3.2.1 and 4.0.0 profiles, `ocp4-pci-dss` and `ocp4-pci-dss-node` ✅ |
| **DISA STIG** | DISA STIG V1R1/V2R1 (and the modern V2R1) ✅ |
| **Other** | BSI (German baseline, `ocp4-bsi`/`rhcos4-bsi`) ✅; the Australian Essential Eight (`rhcos4-e8`) ✅; the `ocp4-e8` lineage ✅ |

**⚠ on HIPAA:** the task brief's checklist included HIPAA among the supported profiles; this pass found **no HIPAA profile in the current supported-profiles record** (the 4.20 chapter's release notes list CIS, NIST 800-53, PCI-DSS, STIG, FedRAMP, BSI and E8, with no HIPAA entry). HIPAA as a *Compliance Operator supported profile* is therefore flagged ⚠ — see "What Could Not Be Verified". The NIST 800-53 reference for the bank's control mapping lives in the [SCADA, ICS and OT Security](scada_guide.md) sibling (which carries the NIST 800-53/800-82 context) and the [Cybersecurity](cybersecurity_guide.md) framework map — cross-ref, do not re-derive.

### 9.4 Remediation and Tailoring — the Evidence Chain

The operator's remediation model completes the evidence chain ✅ (4.20 chapter): a failed rule produces a `ComplianceRemediation` — an auto-applied MachineConfig or Kubernetes-object change (with `outdated`/`applied` states) — and the operator tracks it so the next scan confirms the fix. "Automatic remediation for API server encryption now applies the appropriate encryption mode based on OpenShift version: AES-GCM for OpenShift 4.13.0 and higher, AES-CBC for earlier versions" ✅ (1.9 release notes) — the compliance layer actively enforces the §3 data-at-rest baseline. Raw results are retrievable (`rawResultStorage`; can be disabled for edge deployments ✅) for export to the evidence store. The bank discipline: **remediation is a change, and a change needs a ticket** — auto-remediation must be paired with the bank's change-management capture (§13.3), or the evidence chain breaks at the exact point the auditor checks: who changed what, when, and why.

### 9.5 The Compliance-Operator Table

| Aspect | Detail | Verified |
|---|---|---|
| Role | Automated inspection against standards; **not an auditor** | ✅ §9.1 |
| CRDs | Profile, ScanSetting, ScanSettingBinding, ComplianceScan, ComplianceSuite, ComplianceCheckResult, ComplianceRemediation, TailoredProfile | ✅ §9.2 |
| Engine | OpenSCAP-based XCCDF/OVAL content (ocp4 + rhcos4); CEL CustomRule scanner (TP) | ✅ §9.2 |
| Profiles | CIS; NIST 800-53 Moderate/High + Moderate-Impact Baseline; FedRAMP Moderate Rev 4; PCI-DSS 3.2.1/4.0; DISA STIG; BSI; E8 | ✅ §9.3 |
| Remediation | Auto-applied MachineConfig/API changes; encryption-mode remediation (AES-GCM 4.13+) | ✅ §9.4 |

---

## 10. The Audit Layer — Audit Logging and Certificate Management

### 10.1 Kubernetes API Audit Logs — the Evidence Spine

Every request that reaches the OpenShift API server, the Kubernetes API server or the OAuth API server can be recorded in the **Kubernetes API audit log** — the cluster's evidence spine. For a MAS-supervised bank this is the technical backbone of the audit-evidence obligations (§13.3): the audit log is the *who did what, when, and with what result* record for every privileged action in the estate. The OpenShift documentation's own frame: "To monitor activity and maintain compliance, you can apply audit log profiles that define the level of detail recorded for API server requests" ✅ (4.9 "Configuring the audit log policy"). Two platform specifics verified ✅:

- **The `PodSecurityViolation` alert.** When Pod Security Admission denies or flags a pod at the audit level, the API server reports it and the alert fires, persisting for one day ✅ (docs 4.11, §5.3) — a built-in tripwire for pod-security drift.
- **The investigation path is documented.** Finding the violating workload means searching the API-server audit logs for the `pod-security.kubernetes.io/audit-violations` annotation, and the docs give the exact `oc adm must-gather -- /usr/bin/gather_audit_logs` + `zgrep` recipe ✅ — the audit log is a *usable* artifact, not a compliance tombstone.

### 10.2 The Audit Policy Profiles — Default, WriteRequestBodies, AllRequestBodies

OpenShift ships four predefined audit policy profiles, selectable on the `APIServer` object (`spec.audit.profile`) ✅ (4.9 chapter):

| Profile | What it logs | Notes |
|---|---|---|
| `Default` | Metadata for read and write requests; request bodies only for OAuth access-token requests | The cluster default ✅ |
| `WriteRequestBodies` | Metadata for all requests + request bodies for every write (`create`, `update`, `patch`) | "More resource overhead than the `Default` profile" ✅ |
| `AllRequestBodies` | Metadata + request bodies for reads and writes (`get`, `list`, `create`, `update`, `patch`) | "The most resource overhead" ✅ |
| `None` | Nothing — not even OAuth token requests | "Not recommended... unless you are fully aware of the risks" ✅ |

Two details matter for the bank ✅: **sensitive resources are never logged past the metadata level** — "Sensitive resources, such as `Secret`, `Route`, and `OAuthClient` objects, are never logged past the metadata level" — so choosing `AllRequestBodies` does not dump secrets into the log (the bodies are redacted by resource class); and **custom rules** (`spec.audit.customRules`) let the platform team apply different profiles per user group, evaluated top-to-bottom, first match wins ✅ — the mechanism for "audit everything the platform team does, metadata-only for application traffic." The bank's audit policy is a documented decision: `WriteRequestBodies` (or `AllRequestBodies`) for privileged groups, `Default` for the rest, `None` never, and the audit stream forwarded to the SIEM (cross-ref the [Cybersecurity](cybersecurity_guide.md) SOC/SIEM chapters).

### 10.3 Certificate Management — the In-Cluster CA, Rotation, Service Certificates

OpenShift runs a self-contained certificate infrastructure, and its properties are what a bank's certificate-management function needs to know ✅-structural (the in-cluster CA model is standard, documented OpenShift practice; the service-certificate annotation below is the canonical mechanism, ⚠-structural where not re-extracted this pass):

- **The in-cluster CA.** OpenShift generates and manages its own cluster CA (the `ingress-operator`-managed default ingress certificate, the API-server serving certificates, the kubelet and etcd certificates) — the cluster is its own PKI, with the CA material held in cluster secrets. The platform rotates component certificates as part of normal operation; the documented operational pattern is that the *default ingress certificate* is routinely replaced with the enterprise CA-signed certificate, while internal component certificates remain cluster-signed ⚠-structural (widely documented; not re-extracted at a primary source this pass).
- **Service certificates.** The **service-ca operator** signs per-service certificates automatically via the `service.beta.openshift.io/serving-cert-secret-name` annotation — a workload annotates its Service, and the operator generates and mounts a signed serving certificate, with rotation handled by the platform ⚠-structural (the service-ca mechanism is canonical OpenShift documentation; flagged because this pass did not re-extract that specific page).
- **The bank overlay.** Internal cluster certificates are fine for east-west traffic; the bank's external endpoints use the enterprise PKI (HSM-backed, cross-ref the [Cybersecurity](cybersecurity_guide.md) identity/CA discussion); and the *certificate inventory* — which certs exist, who signed them, when they rotate — is itself audit evidence (§13.3).

### 10.4 The Audit-and-Certificate Table

| Control | Mechanism | Verified |
|---|---|---|
| Audit log spine | Kubernetes API audit logs for the OpenShift, Kubernetes and OAuth API servers | ✅ §10.1 |
| Policy profiles | `Default` / `WriteRequestBodies` / `AllRequestBodies` / `None`; custom per-group rules; sensitive resources metadata-only | ✅ §10.2 |
| PSA tripwire | `PodSecurityViolation` alert + `pod-security.kubernetes.io/audit-violations` annotation search path | ✅ §10.1 |
| In-cluster PKI | Cluster CA, default ingress certificate, component certificates; service-ca operator for service certificates | ✅-structural §10.3 |
| Bank overlay | Enterprise PKI for external endpoints; certificate inventory as audit evidence | ⚠-structural §10.3 |

---

## 11. The Red Hat Security Product Family — Red Hat Advanced Cluster Security for Kubernetes

### 11.1 The StackRox Lineage — 2021, the Acquisition

**Red Hat acquired StackRox in 2021, and StackRox became Red Hat Advanced Cluster Security for Kubernetes (RHACS).** The verified timeline ✅:

- **7 January 2021** — Red Hat announced its intent to acquire StackRox: "Red Hat to Acquire Kubernetes-Native Security Leader StackRox" ✅ (press release; the business-wire record dates it 7 January 2021). The press release frames the rationale exactly as this guide's §1 does: "Securing Kubernetes workloads and infrastructure cannot be done in a piecemeal manner; security must be an integrated part of every deployment, not an afterthought. Red Hat adds StackRox's Kubernetes-native capabilities to OpenShift's layered security approach" ✅ (Paul Cormier quote). It also records the product's prior identity and history: **StackRox was "founded in 2014"** and evolved to focus on Kubernetes security, shipping a "Kubernetes-native security platform" ✅, and it launched **KubeLinter** in October 2020 ✅.
- **22 February 2021** — "Red Hat closes acquisition of StackRox" ✅ (Red Hat blog, dated 22 February 2021).
- **The product rename.** The acquired technology is today sold and documented as **Red Hat Advanced Cluster Security for Kubernetes (RHACS)**, "powered by StackRox technology" ✅ (Red Hat Ecosystem Catalog entry) — the StackRox Kubernetes Security Platform name belongs to the pre-acquisition product ✅ (the press release's framing of StackRox as "the industry's first Kubernetes-native security platform" ✅).

### 11.2 RHACS Capabilities — Vulnerability, Compliance, Segmentation, Runtime, Enforcement

RHACS' capability set, verified from the Red Hat product materials ✅: it "supports a holistic security strategy across your applications and application platform... Build security into your build, deploy, and runtime workflows with consistent vulnerability management and policy guardrails" (product page), and protects "across build, deploy, and runtime" with "vulnerability management, policy and configuration violations, and application... security" (datasheet) ✅. Concretely, the capability families a bank uses:

1. **Vulnerability management** — continuous scanning of images (in registries and in clusters) against vulnerability feeds, with severity and fixability context; integrated into CI/CD so a vulnerable image fails the pipeline ✅ (product materials).
2. **Compliance** — built-in compliance checks (CIS Benchmarks, NIST frameworks) across clusters, with the ability to receive Compliance Operator results into the ACS compliance dashboard (the 4.20 Compliance Operator release notes document the results-appearing-in-ACS integration ✅).
3. **Network segmentation** — observing real traffic and generating NetworkPolicy (and OVN-Kubernetes admin-network-policy) recommendations to build/verify the default-deny mesh (§6.2) ✅ (product materials; the network-segmentation capability is a headline RHACS feature).
4. **Runtime detection** — process allow-listing, behavioral baselining and anomaly detection on running workloads (the catalog entry: "combining custom policies, process allow lists, application and network baselines, and behavioral modeling to identify anomalous behavior") ✅.
5. **Policy enforcement** — the policy engine with "hundreds of built-in controls to enforce security best practices, industry standards such as CIS Benchmarks and NIST, and configuration management of both containers and Kubernetes, and runtime security" ✅ (press release) — enforced at build, deploy and runtime, including admission-time blocking of violating deployments ✅-structural.

### 11.3 The Architecture — Central, Sensor, Collector

The RHACS deployment model is the standard StackRox architecture ✅-structural (documented in the RHACS docs, which this pass touched via the troubleshooting and eBPF pages ✅): a **Central** service (the management plane: policies, vulnerability data, the UI/API, and the compliance dashboards), per-cluster **Sensor** components (policy evaluation and enforcement in the cluster), and the **Collector** runtime component that gathers process, network and syscall-level data — with kernel-module and eBPF collection modes ✅ (the eBPF reliance is documented in "Using eBPF in Red Hat products": "The advanced cluster security operator relies on eBPF for collecting runtime cluster information" ✅). Central aggregates across clusters, which is the estate-level view a bank's security operations need — one pane for N clusters (cross-ref §13 and the [Cybersecurity](cybersecurity_guide.md) SOC chapter).

### 11.4 Built-In OpenShift Security vs RHACS — the Comparison Table

| Capability | Built-in OpenShift | RHACS | Notes |
|---|---|---|---|
| Pod permission control | SCCs (RBAC-granted, priority-ordered) ✅ | Policy engine can add deploy-time checks ✅ | SCCs are the enforcement; ACS policies complement |
| Pod security standards | PSA with SCC↔PSA label sync ✅ | Compliance views (CIS/NIST) ✅ | Both speak privileged/baseline/restricted |
| Image vulnerability scanning | — (registry-side via Clair, §12.2) | Build/deploy/runtime scanning ✅ | ACS scans what runs, not just what is stored |
| Compliance evidence | Compliance Operator (OpenSCAP) ✅ | Compliance dashboards + Operator results ✅ | Operator = evidence; ACS = estate-wide view |
| Network segmentation | NetworkPolicy (OVN-Kubernetes) ✅ | Generates/recommends policies from traffic ✅ | ACS operationalizes default-deny |
| Runtime detection | eBPF-based platform telemetry ⚠-structural | Collector (eBPF/kernel) with behavioral baselines ✅ | ACS is the dedicated detector |
| Policy enforcement | Admission plugins/webhooks ✅ | Enforce at build/deploy/runtime ✅ | ACS is the policy-as-code engine |
| Multi-cluster view | Per-cluster tooling | Central aggregates all clusters ✅ | Estate-level ops for the bank |

The honest reading of the table: **OpenShift built-in security is the enforcement substrate; RHACS is the detection-and-assurance layer on top of it.** A bank deploys the substrate first (SCCs, RBAC, PSA, NetworkPolicy, etcd encryption, audit logs) and adds RHACS for the continuous, estate-wide view that supervision expects — the §14 worked example shows the pairing.

---

## 12. The Red Hat Security Product Family — Red Hat Quay

### 12.1 Quay as the Trusted Registry

Red Hat Quay is Red Hat's enterprise container registry, and in the bank pattern it is the **single trusted ingestion point** for the cluster: everything that runs in production came through Quay, so Quay is where image trust (§7), scanning (§12.2) and access control converge. The verified facts: Quay is deployable as a registry operator on OpenShift ✅-structural (the Red Hat Quay Operator is the standard deployment path, documented in the Quay docs), and its feature set for the bank estate is mirroring (§12.3), robot accounts (§12.3), Clair scanning (§12.2) and the Quay API for automation ⚠-structural (the API/automation surface is standard Quay documentation; the specific features named below are ✅ where cited).

### 12.2 Clair — the Vulnerability Scanner

**Clair is the open-source security scanner packaged with Red Hat Quay**: "Clair is a set of micro services that can be used with Red Hat Quay to perform vulnerability scanning of container images associated with a set of Linux operating systems" ✅ (Red Hat Quay 3.4 docs) and "Clair is an open source security scanner that analyzes container images and reports vulnerabilities... Clair v4 (Clair) leverages static code analyses for parsing image content and reporting vulnerabilities affecting the content. Clair is packaged with Red Hat Quay" ✅ (Quay 3.17 docs). Two properties matter for the bank: **Clair is a static analyzer** — it parses image content and matches against vulnerability databases (including OSV — "Open Source Vulnerability... a vulnerability database and monitoring service that focuses on tracking and managing security vulnerabilities in open source software" ✅, Quay 3.11 docs) — and it **runs at the registry**, so every image that passes through the trusted ingestion point is checked before it can be referenced by a deployment. The scan results feed the image-trust story (§7) and the RHACS vulnerability view (§11.2), and the bank's pipeline fails the build on critical findings — the shift-left gate (cross-ref [Cybersecurity](cybersecurity_guide.md) §6).

### 12.3 Mirroring and Robot Accounts — the Bank Pattern

Two Quay features are the bank's operational core:

- **Mirroring.** Quay can mirror repositories from upstream registries (including registry.redhat.io) into the bank's own Quay, so the cluster never pulls directly from the internet — the controlled, air-gapped-friendly ingestion pattern that pairs with `oc-mirror`'s signature mirroring (§7.3) ✅-structural (Quay repository mirroring is standard documented functionality; the disconnected-cluster signature mirroring is ✅ via the developers.redhat.com article).
- **Robot accounts.** Quay's robot accounts provide scoped, rotating credentials for automation — CI systems push, clusters pull, each with its own robot and quota ⚠-structural (robot accounts are standard Quay documentation; flagged because not re-extracted this pass). The bank discipline: one robot per pipeline and per cluster, rotated like any credential, and every robot's usage logged.

### 12.4 The Quay Table

| Capability | Mechanism | Verified |
|---|---|---|
| Registry | Red Hat Quay, deployable via the Quay Operator on OpenShift | ✅-structural §12.1 |
| Scanning | Clair v4 static analysis of image content; OSV and other vulnerability feeds; packaged with Quay | ✅ §12.2 |
| Ingestion control | Mirroring from upstream registries into the bank Quay; cluster pulls only from Quay | ✅-structural §12.3 |
| Automation identity | Robot accounts (scoped credentials for CI and clusters) | ⚠-structural §12.3 |

---

## 13. OpenShift Under MAS Supervision — the Regulated-Banking Angle

### 13.1 The MAS Floor — Notice 658 and the Outsourcing Overlay

For a bank in Singapore, the binding regulatory floor for a platform like OpenShift is set by the Monetary Authority of Singapore (MAS), and the most platform-relevant instrument is **Notice 658 — "Management of Outsourced Relevant Services for Banks"**, issued 11 December 2023 ✅ (verified in the sibling guide's ledger, [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §3.2: "the outsourcing notice: materiality assessment, risk management of outsourced relevant services, notification of material outsourcing, exit plans — read with the 2023 Outsourcing Guidelines"). This is a **condensed cross-ref**, not a re-derivation: the full instrument map (the Banking Act, the 600-series notices, the TRMG, the Outsourcing Guidelines, the BCM Guidelines) lives in the MAS sibling — read it there.

Why Notice 658 hits a container platform harder than legacy infrastructure: **an OpenShift cluster is an outsourcing vehicle and an outsourced service at the same time.** When the bank runs OpenShift on a cloud provider's managed offering (cross-ref [ibm_cloud_guide.md](ibm_cloud_guide.md) for the IBM Cloud / ROKS form), the platform itself is an outsourced relevant service — materiality assessment, risk management, notification to MAS for material outsourcing, and exit plans all apply ✅ (the sibling's summary of 658's requirements). When the bank's application teams deploy workloads onto the platform, the platform team is an intra-group service provider with the same discipline: documented controls, service levels, audit rights, and exit evidence. The platform-security controls of §1–§12 are precisely the evidence 658's risk-management expectations demand — materiality assessments need the control inventory, and exit plans need the data-portability and key-escrow facts of §3 (backup keys) and §12 (registry mirroring).

### 13.2 Data Residency and Segregation

MAS supervision carries data-residency expectations: regulated data must remain in approved jurisdictions, and the estate must demonstrate segregation of environments and tenants ✅-structural (the data-residency overlay is documented in the MAS sibling's outsourcing/TRM discussion; this pass did not re-extract mas.gov.sg). On OpenShift, the platform-native answers are the ones this guide verified: **namespace isolation** as the tenant boundary (§5.5 — MCS labels, pre-allocated UIDs, PSA labels, RBAC scoping), **environment segregation as cluster/namespace topology** (dev/test/prod landing zones with separate RBAC, PSA and NetworkPolicy postures, §14), **data-at-rest controls** (§3 — etcd encryption, external KMS, backup-key separation) and **the registry as the single ingestion point** (§12) so that what enters the cluster — and where it came from — is known and controlled. The residency question "where is this data stored?" has a platform answer: the storage classes, the node topology and the backup locations are all declared objects, auditable like everything else.

### 13.3 Audit-Evidence Obligations — the Platform as Evidence Machine

The MAS regime — like every modern prudential regime — is evidence-driven: supervision asks the bank to *demonstrate* controls, not assert them. OpenShift's security machinery is unusually well-suited to this because **the platform is its own evidence machine** (verified mechanisms from the earlier sections): the Kubernetes API audit log with per-group profiles (§10.2) records every privileged action; the Compliance Operator produces OpenSCAP-based scan results against CIS/NIST/PCI-DSS profiles with remediation state (§9); SCC grants trace every pod privilege to an RBAC binding (§4.2); the audit policy protects sensitive resources from body logging while keeping metadata (§10.2); RHACS maintains estate-wide policy-violation history (§11); and the cluster's declarative nature (MachineConfigs, ScanSettings, NetworkPolicies as code) means configuration history is versionable (§1.2). The bank's obligation is to *operationalize* the evidence: forward audit streams to the SIEM (cross-ref [Cybersecurity](cybersecurity_guide.md) SOC/SIEM), archive Compliance Operator raw results to the evidence store, ticket every remediation (§9.4), and retain audit logs per the bank's retention policy — because a control that cannot produce its log is, for MAS purposes, a control that does not exist.

### 13.4 The Shared-Responsibility Boundary in a Bank Context

Every deployment of OpenShift rests on a shared-responsibility boundary, and the bank must document where it falls. The verified platform facts map cleanly onto the boundary ✅: **Red Hat's layer** — the OpenShift platform itself: the control plane, the operators, CRI-O, the SCC machinery, the Compliance Operator, the audit pipeline (the platform's security features are the *subject* of this guide); **the bank's layer** — everything configured on top: the SCC grants and custom SCCs (§4), RBAC design (§5), NetworkPolicy mesh (§6), image-trust policy (§7), secrets strategy (§8), profile selection and remediation governance (§9), audit policy and SIEM forwarding (§10), and the ACS/Quay operations (§11–§12). When the platform runs on a hyperscaler or IBM Cloud (cross-ref [ibm_cloud_guide.md](ibm_cloud_guide.md)), a third layer — the provider's — sits underneath, and Notice 658's materiality and notification logic (§13.1) applies to *that* boundary too. The deliverable is a written boundary document: who secures what, which controls are whose evidence, and where the platform team's authority ends (⚠-structural — the shared-responsibility model is universal doctrine; the boundary *document* is the bank's artifact).

### 13.5 The MAS-Mapping Table

| MAS expectation | Platform answer (verified section) |
|---|---|
| Outsourcing risk management (Notice 658, 11 Dec 2023 ✅) | Control inventory (§1–§12), exit evidence: backup keys (§3.3), registry mirroring (§12.3) |
| Material outsourcing notification | Managed-platform boundary documented (§13.4); provider controls mapped |
| Data residency | Namespace/tenant isolation (§5.5), environment topology (§14), storage/backup locations declared (§3) |
| Audit evidence | API audit logs + profiles (§10), Compliance Operator results (§9), RBAC-traceable SCC grants (§4.2) |
| Segregation of duties | RBAC design (§5.1, §14.3), cluster-admin as break-glass (§5.1), maker-checker on remediation (§9.4) |
| Incident readiness | PodSecurityViolation tripwire (§10.1), RHACS runtime detection (§11.2), SIEM forwarding (§10.2, §13.3) |

---

## 14. The Cymbal Bank Worked Example — a Regulated OpenShift Deployment

### 14.1 The Scenario — the Payments-Workload Landing Zone

Cymbal Bank is establishing a new OpenShift 4 landing zone for a payments-processing workload family (card-scheme connectivity, transaction orchestration, settlement reporting) in Singapore. The constraints: MAS-supervised (Notice 658 applies because parts of the estate run on a managed cloud offering), production data in Singapore, and the platform must produce audit evidence for the next supervisory examination. The deployment decisions below follow the house conventions used across this repository's Cymbal Bank examples (the obligations×controls mapping of [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) §7–§8, and the security-by-design review discipline of the [security-by-design guide](security_by_design_guide.md)) and are **illustrative of the house context — not a description of any real system** ⚠ (the standing repo convention).

### 14.2 The SCC Strategy

The SCC posture follows the verified mechanics of §4:

- **Defaults stay default.** The bank does not modify any default SCC ✅ (the docs' own rule, §4.1). New clusters install with `restricted-v2` as the default for authenticated users ✅ (§4.3), which is already the bank's target posture.
- **One custom SCC, narrowly drawn.** A single custom SCC — `cymbal-payments-hostport` — grants exactly two things the payments workloads need and `restricted-v2` denies: `NET_BIND_SERVICE` capability and a specific host port range for the scheme-connectivity pods. It is created as a new SCC (never a mutation of defaults), assigned a priority only within the custom range (never touching default-SCC priorities — the docs' instability warning, §4.2), and its RBAC `use` grant is scoped to the payments service accounts only.
- **`privileged` and `hostnetwork` are break-glass.** No application workload ever receives them; the grants exist only on the platform-administration break-glass account, mirrored in the SOC runbook (cross-ref [Cybersecurity](cybersecurity_guide.md) incident response).
- **The PSA side is synchronized.** The landing-zone namespaces keep `security.openshift.io/scc.podSecurityLabelSync` enabled (the default for user namespaces, §5.4), so the SCC↔PSA label sync reflects the real grants, and the platform team sets `pod-security.kubernetes.io/enforce: restricted` on the payments namespaces once the workloads pass at `warn`/`audit` — the graduated 4.11 rollout (§5.3).

### 14.3 The RBAC Design

The RBAC design implements least privilege and separation of duties (§5.1) with three role families, each bound to an AD group and each auditable:

| Role family | ClusterRole/Role | Grants | Bound to |
|---|---|---|---|
| Platform admin | `cluster-admin` (break-glass) + `system:cluster-admins` group membership | Everything, monitored, short-lived sessions | Platform ops leads (2-person rule) |
| Platform operator | Custom `cymbal-platform-operator` ClusterRole | Cluster-scoped read + operator management (Compliance Operator, RHACS, Quay), no `use` on `privileged`/`hostnetwork` SCCs | Platform ops team |
| Workload owner | Namespace-scoped `Role`s per landing zone | Deploy/edit within own namespaces, read audit annotations, `use` on `cymbal-payments-hostport` (payments only) | Application teams |
| Auditor | Custom `cymbal-auditor` ClusterRole | Read-only access to audit logs, Compliance Operator results, SCC bindings | Internal audit (no write) |

Every SCC grant traces to one of these bindings (§4.2), cluster-admin sessions are break-glass with SIEM-flagged audit events (§10), and the auditor role is what the examiners get — read-only evidence access, no ability to alter it.

### 14.4 The Compliance Operator Profile Selection

The profile set follows §9.3 and the bank's obligations×controls map (cross-ref the MAS sibling §7–§8) ✅:

- **CIS OpenShift** (`ocp4-cis` + node profile) — the hygiene baseline, scanned on a schedule via `ScanSetting`.
- **NIST 800-53 Moderate** (platform and node profiles) — the control-framework overlay the bank's risk team maps its MAS obligations onto (the NIST 800-53 context lives in the [SCADA guide](scada_guide.md); the bank reuses that mapping rather than re-deriving it).
- **PCI-DSS 4.0** (`ocp4-pci-dss`, `ocp4-pci-dss-node`) — the card-scheme requirement for the payments workloads.
- **Governance rules:** every `ComplianceRemediation` is tied to a change ticket before apply (§9.4); raw results (`rawResultStorage`) are exported to the evidence store; a `TailoredProfile` documents the legitimate deviations (e.g. the custom host-port SCC) with written risk acceptance; and the scan schedule is a declared `ScanSetting`, so "we scan continuously" is a fact, not a claim.

### 14.5 The ACS/RHACS Integration

The detection-and-assurance layer (§11) is deployed as the pairing with the built-in substrate:

- **Deployment:** Central in a dedicated `acs` namespace on a management cluster; Sensor + Collector (eBPF mode, §11.3) on every workload cluster.
- **Policies:** the built-in policy library (CIS/NIST-aligned, §11.2) enabled in *enforce* mode at deploy-time for the payments namespaces — a violating deployment is blocked at admission, with the failure surfaced to the platform team.
- **Network segmentation:** ACS network-flow baselining generates the NetworkPolicy recommendations for the payments mesh (§6.2); the platform team reviews, tickets and applies them — the default-deny mesh is evidence-backed, not guessed.
- **Compliance view:** the Compliance Operator results appear in the ACS compliance dashboard (the verified integration, §11.2) — one pane for the examiner: scan state, policy violations, runtime anomalies.
- **Vulnerability flow:** Quay/Clair scans (§12.2) + ACS runtime scanning; critical findings fail the CI pipeline (shift-left, §8.1) and, for already-running images, trigger the incident process.

### 14.6 The Audit-Trail Design

The evidence chain (§13.3) is explicit:

- **Audit policy:** `WriteRequestBodies` for the platform-admin and platform-operator groups, `Default` for everyone else, `None` never — via `spec.audit.customRules` on the `APIServer` object (§10.2). Sensitive resources stay metadata-only by platform design (§10.2).
- **Forwarding:** API audit logs stream to the bank SIEM (cross-ref [Cybersecurity](cybersecurity_guide.md) §5), with the `PodSecurityViolation` alert wired to the SOC.
- **Evidence store:** Compliance Operator raw results, scan schedules, SCC bindings and RBAC bindings are exported on a schedule to the bank's evidence repository — the examinable artifact set.
- **Retention and integrity:** audit logs are retained per the bank's retention schedule, written to append-only storage, and the backup-key separation rule (§3.3) applies to every etcd snapshot.

### 14.7 The Lessons

1. **The substrate is the story.** SCCs, RBAC, PSA, NetworkPolicy, etcd encryption and audit profiles — all verified platform mechanisms — are the controls the examiners see; the product family (RHACS, Quay) operationalizes them but does not replace them (§11.4).
2. **Defaults are a posture.** A new-install default of `restricted-v2` (§4.3) means the secure state is the *default* state — the bank's SCC work is narrow exceptions, not broad remediation.
3. **Evidence is a design input.** The audit policy, the Compliance Operator schedules and the RBAC auditor role were designed *before* the workloads, because MAS examines evidence, not intentions (§13.3).
4. **Change is the risk.** Remediation, policy application and SCC grants are all changes — ticketed, reviewed and logged — because the audit trail is only as good as the discipline around it (§9.4, §14.6).

---

## The Glossary

| Term | Definition |
|---|---|
| **OpenShift Container Platform** | Red Hat's enterprise Kubernetes distribution; OpenShift 4 GA 4 June 2019; operator-managed, RHCOS-based, CRI-O runtime ✅ (§1) |
| **RHCOS** | Red Hat Enterprise Linux CoreOS — the immutable node OS of OpenShift 4 ✅ (§1.2) |
| **CRI-O** | The OCI container engine that implements the Kubernetes CRI; the default engine on OpenShift 4, replacing Docker ✅ (§2.2) |
| **SELinux** | Mandatory access control on the host; OpenShift applies per-namespace MCS labels ✅ (§2.1) |
| **MCS label** | Multi-Category Security SELinux label pre-allocated per namespace — container isolation at the kernel level ✅ (§2.1) |
| **FIPS mode** | Install-time cryptographic floor (`fips: true`); FIPS 140-2/140-3-submitted RHEL/RHCOS crypto; x86_64/ppc64le/s390x ✅ (§2.3) |
| **etcd encryption** | Opt-in AES-CBC encryption of Secrets/ConfigMaps/Routes/OAuth tokens at rest; keys rotate every 7 days, 10 historical keys ✅ (§3) |
| **KMS v2** | Kubernetes Key Management Service v2 — external key management for etcd encryption ✅ (§3.4) |
| **SCC** | Security Context Constraint — OpenShift's pod-permission contract; RBAC-granted, admission-time, priority-ordered ✅ (§4) |
| **SCC priority** | Ordering field in SCC selection; 0 = lowest; higher priority tried first; never raise default-SCC priorities ✅ (§4.2) |
| **restricted-v2** | The most restrictive default SCC on new 4.11+ installs: ALL capabilities dropped, runtime/default seccomp, no escalation ✅ (§4.3) |
| **PSA** | Pod Security Admission — the Kubernetes built-in admission controller implementing the three Pod Security Standards ✅ (§5.3) |
| **PSS** | Pod Security Standards — privileged / baseline / restricted levels ✅ (§5.3) |
| **PSP** | PodSecurityPolicy — deprecated in Kubernetes 1.21, removed in 1.25; predated and outlived by SCCs on OpenShift ✅ (§5.3) |
| **podSecurityLabelSync** | The `security.openshift.io/scc.podSecurityLabelSync` label controlling SCC→PSA warn/audit label synchronization ✅ (§5.4) |
| **NetworkPolicy** | Kubernetes-native ingress/egress traffic rules; enforced by OVN-Kubernetes ✅ (§6) |
| **OVN-Kubernetes** | The default CNI network provider on OpenShift; OVN-based overlay ✅ (§6.1) |
| **Image signing** | Cryptographic signatures for container images; Red Hat serves signatures for registry images; sigstore/cosign formats ✅ (§7) |
| **RHTAS** | Red Hat Trusted Artifact Signer — production-ready Sigstore deployment; technical preview announced 15 Nov 2023 ✅ (§7.2) |
| **policy.json** | The containers/image trust policy (`signedBy` rules; reject-by-default recommended) ✅ (§7.4) |
| **External Secrets Operator** | OpenShift operator syncing AWS SM, Vault, GCP SM, Azure KV, IBM SM, SSM Parameter Store into the cluster ✅ (§8.2) |
| **seccomp** | Syscall filtering; `runtime/default` default in restricted-v2 ✅ (§8.3) |
| **Compliance Operator** | OpenSCAP-based compliance scanner with Profile/ScanSetting/ScanSettingBinding/ComplianceScan CRDs; not an auditor ✅ (§9) |
| **OpenSCAP** | The scanner engine (XCCDF/OVAL content) under the Compliance Operator ✅ (§9.2) |
| **ComplianceScan / ScanSetting** | The CRs that declare what to scan and how; bound via ScanSettingBinding ✅ (§9.2) |
| **TailoredProfile** | Sanctioned profile customization for legitimate deviations ✅ (§9.2) |
| **Audit log profiles** | Default / WriteRequestBodies / AllRequestBodies / None; sensitive resources metadata-only ✅ (§10.2) |
| **RHACS** | Red Hat Advanced Cluster Security for Kubernetes — the StackRox-derived security platform (acquisition announced 7 Jan 2021, closed 22 Feb 2021) ✅ (§11) |
| **StackRox** | The Kubernetes-native security platform founded 2014, acquired by Red Hat 2021, renamed RHACS ✅ (§11.1) |
| **Central / Sensor / Collector** | RHACS architecture: management plane, per-cluster enforcement, runtime data collection (eBPF) ✅ (§11.3) |
| **Clair** | The open-source vulnerability scanner packaged with Red Hat Quay; static analysis of image content ✅ (§12.2) |
| **Robot accounts** | Scoped Quay credentials for CI/cluster automation ⚠-structural (§12.3) |
| **Notice 658** | MAS "Management of Outsourced Relevant Services for Banks" (issued 11 Dec 2023) — the outsourcing overlay ✅ (§13.1) |
| **Shared responsibility** | The documented boundary: Red Hat secures the platform, the bank secures the configuration on top ✅-structural (§13.4) |

---

## Claims Status and Verification Notes

**Verification pass: 2026-08-31, live web access (web search + direct page extraction; several searches returned empty/rate-limited and were retried via direct extraction of primary URLs, which succeeded).** Primary sources verified live this pass: **docs.redhat.com** (OpenShift 4.18 "Managing security context constraints" — the default SCC catalog, the admission mechanism, priority rules, the CRI-O capability list; OpenShift 4.11 "Understanding and managing pod security admission" — the global privileged enforcement, the SCC↔PSA label-sync controller, the `podSecurityLabelSync` label; OpenShift 4.12 "Encrypting etcd data" — default-off, aescbc, 7-day rotation, 10 historical keys, encrypted-resource list, backup key file; OpenShift 4.21 "Support for FIPS cryptography" — install-time FIPS, `fips: true`, FIPS 140-2/140-3-submitted libraries, architecture limits; OpenShift 4.20 "Compliance Operator" — the CRD model, the profiles record, remediation, "not an auditor"; OpenShift 4.9 "Configuring the audit log policy" — the four profiles, sensitive-resource metadata-only rule, customRules; OpenShift 4.19 "External Secrets Operator for Red Hat OpenShift" — the provider list; Red Hat Quay 3.4/3.17 "Clair" chapters — Clair as the packaged scanner), **access.redhat.com** (article 3116561 — Red Hat registry signatures, sigstore signature servers, reject-by-default), **redhat.com** (the StackRox acquisition press release — announced, expected Q1 2021 close, StackRox founded 2014, KubeLinter Oct 2020; the "Red Hat closes acquisition of StackRox" blog — 22 February 2021; the "OpenShift 4 now defaults to CRI-O" blog — CRI-O default, 3.7 TP/3.9 GA lineage, Machine Config Operator; the "Pod Security Admission in OpenShift 4.11" blog — the dual admission design, new SCCs; the RHTAS/Enterprise Contract blog — Sigstore deployment, TP announcement link), **web.archive.org** (the "Red Hat OpenShift 4 is Now Available" blog — GA, 4 June 2019), **kubernetes.io** (PodSecurityPolicy deprecated 1.21 / removed 1.25), **developers.redhat.com** (the eBPF article — ACS relies on eBPF; the disconnected-signatures article — oc-mirror v2 mirrors signatures in 4.19), and the **catalog.redhat.com** RHACS entry ("powered by StackRox technology").

**✅ Verified this pass (live sources):** OpenShift 4 GA 4 June 2019; CRI-O as the OpenShift 4 default container engine implementing the Kubernetes CRI (Docker removed); the SCC admission mechanism (RBAC `use` grants, user/group/service-account-derived SCC sets, admission-time validation, priority-ordered selection with 0 lowest); the default SCC catalog incl. `restricted-v2` as the most restrictive new-install default and the "do not modify default SCCs" rule; PSA in OpenShift 4.11 (global privileged enforce + restricted warn/audit, warn→audit→enforce rollout, `PodSecurityViolation` alert, SCC↔PSA label-sync controller); PSP deprecated in 1.21 and removed in 1.25; FIPS mode (install-time, `fips: true`, FIPS 140-2/140-3-submitted libs, x86_64/ppc64le/s390x, not enableable post-deploy); etcd encryption (default off, aescbc, Secrets/ConfigMaps/Routes/OAuth tokens, 7-day rotation, 10 historical keys, `_static_kuberesources_` backup key file, KMS v2 external key management in 4.22); Compliance Operator (CRDs, OpenSCAP-based, profiles CIS / NIST 800-53 Moderate+High / FedRAMP Moderate Rev 4 / PCI-DSS 3.2.1+4.0 / DISA STIG / BSI / E8, encryption-mode remediation AES-GCM on 4.13+, "not an auditor"); audit log profiles (Default/WriteRequestBodies/AllRequestBodies/None; sensitive resources never logged past metadata); OVN-Kubernetes as default CNI; the External Secrets Operator provider list; Red Hat registry image signatures with sigstore signature servers and reject-by-default guidance; RHEL 8 sigstore-format signing (`--sign-by-sigstore-private-key`); RHTAS technical preview announced 15 November 2023; oc-mirror v2 signature mirroring in OpenShift 4.19; the StackRox acquisition (announced 7 January 2021, closed 22 February 2021, founded 2014, KubeLinter October 2020, "powered by StackRox technology" for RHACS); Clair as the scanner packaged with Red Hat Quay; ACS eBPF reliance; and Notice 658 (Management of Outsourced Relevant Services for Banks, 11 December 2023 — verified in the MAS sibling's ledger).

**⚠ Flagged / unverified this pass (honest ledger):** the claim that "Red Hat began signing UBI/RHEL images with cosign in ~2022" — not pinned to a primary source (the sigstore signature-store paths and RHTAS are ✅; the specific 2022 cosign announcement is not) (§7.2, "What Could Not Be Verified"); **HIPAA as a Compliance Operator supported profile** — not found in the current supported-profiles record (CIS/NIST/PCI-DSS/STIG/FedRAMP/BSI/E8 verified; HIPAA absent) (§9.3); **SELinux enforcing-by-default on RHCOS** — ⚠-structural (the namespace MCS pre-allocation itself is ✅ via the SCC chapter) (§2.1); the **in-cluster CA / service-ca operator** details — ⚠-structural, not re-extracted this pass (§10.3); **Quay repository mirroring and robot accounts** — ⚠-structural (§12.3); **mutating-webhook risk classification** — ⚠-structural (§5.2); **RHACS deploy-time enforcement and network-segmentation generation** — ✅ at product-materials level, enforcement mechanics ⚠-structural (§11.2); **data-residency overlay under MAS** — ⚠-structural (§13.2); **Kata Containers availability** — ✅ via the 3.11 CRI-O docs quote, current-version availability not re-verified (§2.2); and **any Cymbal Bank-internal system details** — ⚠, the §14 worked example is illustrative (§14.1).

**Repository convention:** ✅ = verified this pass at a primary source, or verified in the cross-referenced guide's ledger; ⚠ = flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source verified this pass.

---

## What Could Not Be Verified

This section is the honest ledger of claims that this pass could not confirm at primary sources, exactly as the house convention requires:

1. **"Red Hat began signing RHEL/UBI container images with sigstore/cosign in ~2022."** The task brief's checklist included this claim. What *is* verified: Red Hat serves signatures for its registry images with sigstore-format signature stores (`https://registry.redhat.io/containers/sigstore`), the RHEL toolchain produces sigstore-format signatures, and RHTAS (a Sigstore deployment) reached technical preview on 15 November 2023. What could not be found: a primary-source announcement dated ~2022 stating that Red Hat's own UBI/RHEL images are cosign-signed. The claim is therefore reported as ⚠ and should not be repeated as fact without a primary citation.
2. **HIPAA as a supported Compliance Operator profile.** The brief's checklist listed HIPAA among the operator's supported profiles. The 4.20 Compliance Operator chapter and its release notes (verified this pass) record CIS, NIST 800-53 (Moderate/High, Moderate-Impact Baseline), FedRAMP Moderate Rev 4, PCI-DSS 3.2.1/4.0, DISA STIG, BSI and E8 — with no HIPAA entry. If the operator's profile catalog changed in a newer release, that was not verifiable this pass; the verified profile set is stated in §9.3 and HIPAA is flagged ⚠.
3. **Exact date of the StackRox acquisition announcement.** Verified as announced 7 January 2021 (business-wire record) and closed 22 February 2021 (Red Hat blog); the press release page itself does not display a date in the extracted text, so the 7 January date rests on the wire-service record — solid but noted.
4. **SELinux "enforcing by default on RHCOS" and the service-ca/in-cluster CA mechanics.** Both are canonical, widely-documented OpenShift behavior, but neither page was re-extracted at a primary source this pass; both are marked ⚠-structural (§2.1, §10.3).
5. **Quay mirroring and robot-account details.** Standard documented Quay functionality; the specific feature pages were not re-extracted this pass (§12.3) — ⚠-structural.
6. **Any Cymbal Bank-internal system facts.** The §14 worked example is an illustrative deployment in the house's context, not a description of any real Cymbal Bank system — the standing repository convention.

---

## Cross-References and Further Reading

**The platform cluster (sibling, technology/):**
- **[openshift_scc_comprehensive_guide.md](openshift_scc_comprehensive_guide.md)** — **THE SCC companion.** The deep mechanism detail behind §4: the SCC object model, strategies, and operational playbooks. This guide carries the verified admission/priority mechanics; the sibling carries the full depth.
- **[openshift_scc_service_account_guide.md](openshift_scc_service_account_guide.md)** — the service-account↔SCC interplay behind §4.2 and §5.1.
- **[openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)** — the availability/HA angle; the fail-secure vs availability tension (§1.5's framing, §14's design).
- **[ibm_cloud_guide.md](ibm_cloud_guide.md)** — OpenShift on IBM Cloud (ROKS): the managed-platform form of §13.1's outsourcing overlay and §13.4's shared-responsibility boundary.
- **[cybersecurity_guide.md](cybersecurity_guide.md)** — **THE security-framework companion.** The NIST CSF 2.0 / CIS Controls / MITRE ATT&CK maps behind §9 and §13; the SOC/SIEM and DevSecOps chapters behind §8.1, §10.2 and §13.3; the Zero Trust frame behind §6.
- **[scada_guide.md](scada_guide.md)** — the NIST 800-53/800-82 and 62443 references; the bank reuses its NIST 800-53 mapping for the Compliance Operator profile selection (§9.3, §14.4).
- **[openscap_guide.md](openscap_guide.md)** — the OpenSCAP engine internals that the Compliance Operator runs (§9.2).
- **[security_by_design_guide.md](security_by_design_guide.md)** — the security-by-design review discipline applied in §14's worked example (STRIDE, least privilege, defense-in-depth).
- **[ai_platform_engineering_guide.md](ai_platform_engineering_guide.md)** — Kubernetes/platform engineering practices (light cross-ref).
- **[enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md)** — the integration-platform angle (light).

**The banking cluster (prefix `../banking/`):**
- **[../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)** — **THE regulatory companion.** Notice 658 (issued 11 December 2023), the TRMG, the Outsourcing Guidelines, data-residency expectations and the obligations×controls map — the §13 overlay in full.

**Primary sources and further reading:** docs.redhat.com — OpenShift 4.18 "Managing security context constraints"; 4.11 "Understanding and managing pod security admission"; 4.12 "Encrypting etcd data"; 4.21 "Support for FIPS cryptography"; 4.20 "Compliance Operator"; 4.9 "Configuring the audit log policy"; 4.19 "External Secrets Operator for Red Hat OpenShift"; 4.22 "External encryption key management"; Red Hat Quay 3.4/3.17 "Clair security scanner" chapters; access.redhat.com article 3116561 ("Verifying image signing for Red Hat Container Registry"); redhat.com — the StackRox press release (7 Jan 2021) and close blog (22 Feb 2021), the CRI-O-default blog, "Pod Security Admission in OpenShift 4.11", "Red Hat Trusted Artifact Signer with Enterprise Contract", "Red Hat OpenShift 4 is Now Available" (4 June 2019, via the Wayback Machine); kubernetes.io — "PodSecurityPolicy Deprecation: Past, Present, and Future" and the Pod Security Policies reference (deprecated 1.21, removed 1.25); developers.redhat.com — "Using eBPF in Red Hat products" and "How to verify container signatures in disconnected OpenShift"; catalog.redhat.com — the RHACS entry; sigstore.dev; and the repo's own verified cluster listed above.

---

*End of guide — the hardened cluster: layered controls verified at the platform's own documentation, evidence the supervision can read, and the discipline to keep it that way — that is the hardened cluster.*
