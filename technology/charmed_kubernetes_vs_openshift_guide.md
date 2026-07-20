# Charmed Kubernetes vs Red Hat OpenShift — Enterprise Kubernetes Platform Comparison

> **Author:** Jack Liu Shurui  
> **Role:** Solution Architect, Crédit Agricole CIB  
> **Date:** July 2026  
> **Version:** 1.0  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Platform Overview](#2-platform-overview)
3. [Architecture Comparison](#3-architecture-comparison)
4. [Installation and Lifecycle Management](#4-installation-and-lifecycle-management)
5. [Upgrade Experience](#5-upgrade-experience)
6. [Supported Infrastructure](#6-supported-infrastructure)
7. [Add-Ons and Ecosystem](#7-add-ons-and-ecosystem)
8. [Networking](#8-networking)
9. [Security Features](#9-security-features)
10. [Storage](#10-storage)
11. [Pricing and Licensing](#11-pricing-and-licensing)
12. [Enterprise Support and SLAs](#12-enterprise-support-and-slas)
13. [Community and Adoption](#13-community-and-adoption)
14. [Strengths and Weaknesses](#14-strengths-and-weaknesses)
15. [Use Case Comparison](#15-use-case-comparison)
16. [Decision Matrix](#16-decision-matrix)
17. [Summary and Recommendations](#17-summary-and-recommendations)

---

## 1. Executive Summary

This guide provides a comprehensive technical comparison between **Charmed Kubernetes** (Canonical/Ubuntu) and **Red Hat OpenShift**, two leading enterprise Kubernetes platforms. Both platforms deliver production-grade container orchestration but differ fundamentally in architecture philosophy, operational model, ecosystem maturity, licensing cost, and target deployment profiles.

**Charmed Kubernetes** is Canonical's open-source Kubernetes distribution, built around Juju — a model-driven operations framework that uses "charms" (operator packages) for lifecycle management of every cluster component. It is modular, infrastructure-agnostic, and carries no license fees.

**Red Hat OpenShift** is Red Hat's enterprise Kubernetes platform, built on the upstream OKD (OpenShift Origin) project. It is a complete application platform that includes built-in monitoring, logging, registry, networking, developer console, CI/CD pipelines, serverless, and service mesh capabilities out of the box. It is licensed per-core with subscription tiers.

**Key decision drivers:**

| Criterion | Favours Charmed K8s | Favours OpenShift |
|-----------|---------------------|-------------------|
| Cost sensitivity | ✅ | ❌ |
| Ubuntu standardised | ✅ | ❌ |
| RHEL standardised | ❌ | ✅ |
| Full enterprise platform OOTB | ❌ | ✅ |
| Maximum security/compliance | Partial | ✅ |
| Infrastructure flexibility | ✅ | Moderate |
| Team already knows K8s ecosystem | Moderate | ✅ |
| Vendor-backed SLAs | Canonical | Red Hat |
| Largest operator ecosystem | ❌ | ✅ |

---

## 2. Platform Overview

### 2.1 Charmed Kubernetes

**Charmed Kubernetes** (formerly known as Canonical Kubernetes Distribution / CDK) is Canonical's offering for deploying, managing, and operating Kubernetes clusters across any infrastructure. It is fully open-source and built on three core principles:

- **Model-driven operations** — Juju provides a declarative model for describing the entire cluster state, including relationships between components.
- **Operator-based lifecycle** — Every cluster component (control plane, workers, CNI, CSI, ingress, monitoring) is managed by a Juju charm — an operator package that encodes Day 1 and Day 2 operations.
- **Multi-cloud abstraction** — A single Juju model can deploy across bare metal, public clouds, private clouds, and edge infrastructure with the same workflow.

**Project origins:** Charmed Kubernetes evolved from the Ubuntu Kubernetes distribution initiative. Canonical is a founding member of the CNCF and a significant upstream Kubernetes contributor, particularly in test infrastructure (kubetest2, Sonobuoy conformance).

**Key characteristics:**

| Attribute | Detail |
|-----------|--------|
| Maintainer | Canonical Ltd. |
| License | Apache 2.0 (fully open source) |
| Upstream base | Pure upstream Kubernetes |
| Operational model | Juju model-driven + charms |
| Certified CNCF conformant | Yes |
| Kubernetes release cadence | Follows upstream (4 releases/year) |
| Primary package format | Snap + Juju charm bundles |
| Default OS | Ubuntu LTS |

### 2.2 Red Hat OpenShift

**Red Hat OpenShift** is Red Hat's enterprise Kubernetes platform, originating from the OpenShift Origin project (now OKD). It wraps Kubernetes with a comprehensive set of integrated services, security controls, and developer tooling, presenting a cohesive application platform rather than a raw Kubernetes distribution.

**Four tiers:**
1. **OKD** — Community-supported upstream (free, no support)
2. **OpenShift Container Platform (OCP)** — Self-managed enterprise tier
3. **OpenShift Platform Plus** — OCP + advanced security (ACS), advanced management (ACM), trustable registry (Quay)
4. **OpenShift Dedicated / ROSA / ARO** — Managed cloud offerings on AWS, Azure, GCP

**Key characteristics:**

| Attribute | Detail |
|-----------|--------|
| Maintainer | Red Hat (IBM subsidiary) |
| License | OpenShift Container Platform — commercial subscription; OKD — Apache 2.0 |
| Upstream base | OKD (OpenShift Origin) → upstream Kubernetes |
| Operational model | Cluster Version Operator + OLM + Operators |
| Certified CNCF conformant | Yes |
| Kubernetes release cadence | 3-month cycle, with backport releases |
| Primary package format | RPM + OCI images + Operator bundles |
| Default OS | Red Hat CoreOS (RHCOS) |

---

## 3. Architecture Comparison

### 3.1 Charmed Kubernetes Architecture

Charmed Kubernetes uses Juju as the central orchestration engine. Juju operates on a **controller-worker model**:

```
┌─────────────────────────────────────────────────────┐
│                  Juju Controller                     │
│  (manages models, lifecycles, relation bindings)     │
└──────┬──────────────────────────────────────┬────────┘
       │                                      │
       ▼                                      ▼
┌──────────────┐                    ┌──────────────┐
│  Model: k8s  │                    │  Model: k8s   │
│  (prod-us)   │                    │  (prod-eu)    │
└──────┬───────┘                    └──────┬────────┘
       │                                   │
       ▼                                   ▼
┌──────────────────┐             ┌──────────────────┐
│  Charm Bundle    │             │  Charm Bundle    │
│  - kubernetes-   │             │  - kubernetes-   │
│    control-plane │             │    control-plane │
│  - kubernetes-   │             │  - kubernetes-   │
│    worker        │             │    worker        │
│  - calico        │             │  - cilium        │
│  - ceph-radosgw  │             │  - ceph-osd      │
│  - prometheus    │             │  - cos-lite      │
│  - ingress       │             │  - ingress       │
└──────────────────┘             └──────────────────┘
```

**Key architectural concepts:**

- **Juju Controller:** A central management server that stores model state, runs reconciliation loops, and distributes operations to machine or container agents.
- **Models:** Logical deployment environments — each cluster is typically one model.
- **Charms:** Operators that encapsulate all lifecycle actions for a component (install, configure, upgrade, scale, integrate, remove).
- **Relations:** Declarative connections between charms (e.g., `kubernetes-control-plane:loadbalancer` ↔ `nginx-ingress:loadbalancer`) that auto-configure integration.
- **Bundles:** Collections of charms with pre-defined relations — a bundle.yaml deploys an entire cluster.

**Modular design:** Charmed Kubernetes ships as individual charms:
- `kubernetes-control-plane` — API server, scheduler, controller-manager, etcd
- `kubernetes-worker` — Kubelet, kube-proxy
- `kubernetes-master` — (legacy, superseded by control-plane)
- `calico`, `cilium`, `flannel` — CNI providers
- `ceph-radosgw`, `ceph-osd`, `ceph-mon` — storage
- `cos-lite` — observability stack
- `nginx-ingress`, `traefik` — ingress controllers

This means you deploy exactly what you need — no more, no less.

### 3.2 OpenShift Architecture

OpenShift has a layered architecture built around operators:

```
┌───────────────────────────────────────────────────────────┐
│                    OpenShift Cluster                       │
├───────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐  │
│  │           Cluster Version Operator (CVO)             │  │
│  │    Manages cluster-state of all OCP components      │  │
│  └──────────────────────┬──────────────────────────────┘  │
│                         │                                  │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │         Operator Lifecycle Manager (OLM)             │  │
│  │   Manages install, upgrade, RBAC for Operators      │  │
│  └──────────────────────┬──────────────────────────────┘  │
│                         │                                  │
│  ┌─────────┬─────────┬──▼──┬─────────┬─────────┬──────┐  │
│  │Monitor  │Logging  │K8s  │Registry │ Router  │CVO   │  │
│  │Oper.    │Oper.    │Core │Oper.    │ Oper.   │Oper. │  │
│  ├─────────┼─────────┼─────┼─────────┼─────────┼──────┤  │
│  │Dev      │Pipeline │SM   │Server   │Knative  │GitOps│  │
│  │Console  │Op.(Tek) │Mesh │less     │         │Op.   │  │
│  └─────────┴─────────┴─────┴─────────┴─────────┴──────┘  │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │         Red Hat CoreOS (RHCOS)                       │  │
│  │     Immutable OS, Ignition-provisioned, ostree       │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │     Infrastructure (Bare Metal / Cloud / vSphere)    │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────┘
```

**Key architectural concepts:**

- **Cluster Version Operator (CVO):** The central reconciliation loop that ensures all cluster components are at the desired version. CVO is the single source of truth for cluster state.
- **Operator Lifecycle Manager (OLM):** Manages the lifecycle of add-on operators — installs, upgrades, RBAC, and dependency resolution for operators from OperatorHub.
- **Cluster Operators:** Each functional area (monitoring, logging, registry, authentication, ingress, storage, etc.) is managed by its own operator reporting to the CVO.
- **RHCOS:** An immutable, container-optimised host OS based on Fedora CoreOS. All management happens declaratively via Ignition configs; no SSH-based package management.
- **HyperShift** (optional): Turns the control plane into a hosted service for multi-tenant clusters.

### 3.3 Architecture Side-by-Side

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Orchestration engine** | Juju (model-driven, relation-based) | CVO + OLM (operator-driven) |
| **Component management** | Charms (operator packages) | Cluster Operators |
| **State management** | Juju controller database | CVO-reconciled desired manifests |
| **OS** | Ubuntu LTS (any LTS) | Red Hat CoreOS (immutable) |
| **Default modularity** | Fully modular — add what you need | Integrated monolith — everything included |
| **Custom resource model** | Juju relations + config | OLM CatalogSources + CRDs |
| **Bootstrap mechanism** | `juju bootstrap` + `juju deploy` | `openshift-install create cluster` |

---

## 4. Installation and Lifecycle Management

### 4.1 Charmed Kubernetes Installation

Charmed Kubernetes installation is entirely Juju-driven:

```bash
# 1. Bootstrap a Juju controller on target infrastructure
juju bootstrap aws/us-east-1 k8s-controller

# 2. Create a model for the cluster
juju add-model production-k8s

# 3. Deploy the Charmed Kubernetes bundle
juju deploy charmed-kubernetes --trust

# 4. Wait for reconciliation
juju status --watch-query
```

**Key workflow characteristics:**

- **`juju bootstrap`** creates a Juju controller on any supported cloud or MAAS-managed bare metal. The controller runs its own MongoDB-backed state database and reconciles models continuously.
- **`juju deploy`** deploys a charm bundle — a pre-baked collection of charms with relations. The bundle YAML defines every component, its configuration, and how components relate.
- **Multi-cloud abstraction:** The same Juju workflow works across AWS, Azure, GCP, OpenStack, VMware vSphere, MAAS bare metal, and LXD containers. Only the initial `juju bootstrap` cloud credential differs.
- **Day 2 operations** are built into charms:
  ```bash
  juju upgrade-charm kubernetes-control-plane
  juju scale-application kubernetes-worker 10
  juju config kubernetes-worker channel=1.31/stable
  juju integrate kubernetes-control-plane cos-lite
  ```

**Juju controller architecture:**

```
┌────────────────────────┐
│  Juju Controller (1 or 3 units for HA)   │
│  ┌──────────────────┐  │
│  │  MongoDB (state)  │  │
│  │  API server       │  │
│  │  Workers          │  │
│  └──────────────────┘  │
└────────────────────────┘
         │
         ▼
┌────────────────────────────────────┐
│  Model "production-k8s"             │
│  ┌──────────┐ ┌──────────┐ ┌─────┐ │
│  │k8s-cont. │ │k8s-worker│ │ceph │ │
│  │  (×3)    │ │  (×5)    │ │(×3) │ │
│  └──────────┘ └──────────┘ └─────┘ │
└────────────────────────────────────┘
```

### 4.2 OpenShift Installation

OpenShift installation has two primary methods:

**Installer-Provisioned Infrastructure (IPI):**

```bash
# 1. Create install-config.yaml
# 2. Generate cluster manifests
./openshift-install create install-config
./openshift-install create manifests

# 3. Deploy cluster (cloud-provisioned infra)
./openshift-install create cluster

# 4. Wait for bootstrap completion
#    Cluster URL + kubeadmin credentials printed on completion
```

**User-Provisioned Infrastructure (UPI):**

```bash
# 1. Generate ignition configs
./openshift-install create ignition-configs

# 2. Provision infrastructure manually
#    (VMs, networks, load balancers, DNS — your setup)

# 3. Boot RHCOS nodes with ignition configs
#    (ISO, PXE, or image-based boot)

# 4. Approve CSRs and monitor bootstrap
./openshift-install wait-for bootstrap-complete
./openshift-install wait-for install-complete
```

**Key workflow characteristics:**

- **IPI:** Terraform-based infrastructure provisioning. Cloud provider resources (VMs, load balancers, security groups, DNS) are fully automated. The installer destroys cleanly via `openshift-install destroy cluster`.
- **UPI:** Full manual control over infrastructure. Requires separate provisioning of all resources. Used for air-gapped, custom, or restricted environments.
- **Boostrap VM:** A temporary machine that runs etcd bootstrap and initial control plane. Auto-destroyed after control plane formation.
- **Agents-based installation** (for bare metal): A single ISO installs both the OS and cluster without a separate installer host.
- **must-gather:** The primary Day 2 troubleshooting tool:
  ```bash
  oc adm must-gather
  ```
  Collects cluster state, operator logs, events, and node-level data into a single archive.

**Post-install Day 2:**

```bash
# Scale cluster
oc edit machineset <name> -n openshift-machine-api

# Configure cluster operators
oc edit configmap cluster-config-v1 -n kube-system

# Set update channel
oc adm upgrade channel fast-4.16

# Monitor operator health
oc get clusteroperators
oc describe clusteroperator kube-apiserver
```

### 4.3 Installation Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Install time (3-node)** | 15–30 min (varies by infra) | 30–60 min (IPI) |
| **Prerequisites** | Juju controller, cloud creds or MAAS | `openshift-install` binary, cloud creds or infra |
| **Infrastructure abstraction** | Juju providers (multi-cloud) | IPI (per-cloud) or UPI (full manual) |
| **Destruction** | `juju destroy-model` | `openshift-install destroy cluster` |
| **Concurrent clusters** | Multiple Juju models | Multiple install dirs or Hosted Control Planes |
| **Air-gapped support** | Via Juju offline mode + snap proxies | Via Operator Lifecycle Manager disconnected installs |
| **Day 2 tooling** | `juju status`, `juju config`, charm hooks | `oc`, cluster operators, must-gather |
| **Host OS management** | Standard Ubuntu (apt, snap) | RHCOS (immutable, Ignition, rpm-ostree) |

---

## 5. Upgrade Experience

### 5.1 Charmed Kubernetes Upgrades

Charmed Kubernetes upgrades are performed component-by-component using Juju charm upgrade commands, following a prescribed order:

```bash
# 1. Check upgrade path
juju upgrade-charm --dry-run kubernetes-control-plane

# 2. Upgrade in order: etcd → control plane → workers → addons
juju upgrade-charm etcd                    # snapshot etcd first
juju upgrade-charm kubernetes-control-plane
juju upgrade-charm kubernetes-worker
juju upgrade-charm calico                  # or cilium, etc.
juju upgrade-charm nginx-ingress

# 3. Verify
juju status
```

**Key characteristics:**

- **Rolling upgrades:** Each charm performs rolling updates within its application. Control-plane units are upgraded one at a time (API server lease ensures availability). Worker nodes are cordoned, drained, upgraded, and uncordoned sequentially.
- **Component-by-component:** You control the order and pacing. You can pause between components, roll back a single charm, or run mixed versions temporarily.
- **Snap channel management:** Kubernetes version is controlled via snap channels (e.g., `1.31/stable`). Upgrading the channel triggers the charm's upgrade hooks:
  ```bash
  juju config kubernetes-control-plane channel=1.32/stable
  ```
- **No built-in canary:** Canary upgrades are manual — you upgrade a subset of workers, validate, then upgrade the rest.
- **Rollback:** Snap-based rollback is possible by switching back to the previous channel, but etcd schema upgrades are one-way. Juju improves rollback safety in newer versions.

### 5.2 OpenShift Upgrades

OpenShift upgrades are managed centrally by the Cluster Version Operator:

```bash
# 1. Check available updates
oc adm upgrade

# 2. Upgrade to latest in channel
oc adm upgrade --to-latest

# 3. Upgrade to specific version
oc adm upgrade --to=4.16.5

# 4. Monitor upgrade
oc adm upgrade --watch
oc get clusterversion
oc get clusteroperators
```

**Key characteristics:**

- **Single CVO-driven upgrade:** The CVO coordinates all cluster operators (monitoring, networking, auth, registry, etc.) through the upgrade. Operators upgrade in dependency order — typically etcd first, then control plane components, then workers.
- **z-stream vs y-stream upgrades:**
  - **z-stream** (e.g., 4.16.1 → 4.16.5): Patch releases. CVO manages automatically. Rolling worker node updates.
  - **y-stream** (e.g., 4.15 → 4.16): Minor version. May require specific upgrade path through an intermediate version. EUS-to-EUS upgrades require careful planning.
- **Upgrade channels:**
  - `candidate-4.16` — Pre-release testing
  - `fast-4.16` — Recommended stable releases (GA + ~2 weeks)
  - `stable-4.16` — Fully vetted releases (GA + ~4 weeks)
  - `eus-4.14` — Extended Update Support (long-term stable)
- **Pause and resume:**
  ```bash
  # Pause workers (mcp)
  oc patch mcp worker --type=merge -p '{"spec":{"paused":true}}'
  # Resume
  oc patch mcp worker --type=merge -p '{"spec":{"paused":false}}'
  ```
- **Canary upgrades:** Use MachineConfigPools to create a canary pool of workers:
  ```bash
  oc label node node-canary-1 node-role.kubernetes.io/canary=
  # This worker upgrades first; validate before rolling rest
  ```
- **Managed rollback:** If an upgrade degrades, CVO supports rollback to the previous version (limited to the previous z-stream; y-stream rollback requires restore).
- **Upgrade graph embedded:** The CVO has a built-in graph of valid upgrade paths — you cannot skip a required intermediate version.

### 5.3 Upgrade Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Upgrade mechanism** | Juju charm upgrade (component-by-component) | CVO-driven (coordinated single command) |
| **Automation level** | Semi-automated (manual per-component) | Fully automated (single `oc adm upgrade`) |
| **Rollback support** | Snap channel rollback (limited, etcd one-way) | Supported for z-stream; y-stream via backup |
| **Canary upgrades** | Manual (upgrade subset of workers) | Built-in via MachineConfigPools |
| **Upgrade validation** | Manual verification after each component | CVO monitors operator health throughout |
| **Mixed-version clusters** | Supported during upgrade | Not recommended (CVO enforces consistency) |
| **Upgrade path validation** | Manual (check changelog) | Built-in upgrade graph prevents invalid paths |
| **Pause capability** | Per-component via `juju refresh --dry-run` | Per-operator via MachineConfigPool pause |
| **Typical minor upgrade time** | 60–120 min (manual steps) | 45–90 min (mostly automated) |

---

## 6. Supported Infrastructure

### 6.1 Charmed Kubernetes

| Infrastructure | Support Level | Provisioning Method |
|---------------|---------------|-------------------|
| **Bare metal (MAAS)** | Fully supported, production | Juju + MAAS provider |
| **OpenStack** | Fully supported, production | Juju OpenStack provider |
| **VMware vSphere** | Fully supported, production | Juju vSphere provider |
| **AWS** | Fully supported, production | Juju AWS provider |
| **Azure** | Fully supported, production | Juju Azure provider |
| **GCP** | Fully supported, production | Juju GCP provider |
| **LXD (system containers)** | Supported, dev/test | Juju LXD provider |
| **Oracle Cloud** | Community | Juju OCI provider |
| **Equinix Metal** | Community | Juju MAAS provider |
| **Edge/IoT devices** | Supported | Juju LXD / microk8s |

**Key differentiator:** Juju's cloud-agnostic model means a single `juju bootstrap` + `juju deploy` workflow works identically across all supported clouds. The Juju controller abstracts away cloud-specific API differences.

### 6.2 OpenShift

| Infrastructure | Support Level | Provisioning Method |
|---------------|---------------|-------------------|
| **Bare metal** | Fully supported, production | IPI (Agents-based), UPI (manual infra) |
| **AWS** | Fully supported, production | IPI (Terraform), UPI, ROSA (managed) |
| **Azure** | Fully supported, production | IPI, UPI, ARO (managed) |
| **GCP** | Fully supported, production | IPI, UPI |
| **VMware vSphere** | Fully supported, production | IPI, UPI |
| **OpenStack (RHOSP)** | Fully supported, production | IPI (since 4.12), UPI |
| **IBM Cloud** | Fully supported, production | IPI |
| **Nutanix AHV** | Fully supported, production | IPI |
| **IBM Z / LinuxONE** | Supported, production | UPI (z/VM, KVM) |
| **IBM Power** | Supported, production | UPI |
| **Alibaba Cloud** | Supported, production | UPI |
| **Any infrastructure** | Supported (UPI) | Manual all resources |

**Key differentiator:** OpenShift's UPI mode supports essentially any platform where you can run RHCOS VMs. IPI is cloud-provider specific and requires active development by Red Hat or a partner.

### 6.3 Infrastructure Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Bare metal** | Via MAAS (automated) | Agents-based IPI (automated) or UPI (manual) |
| **Public cloud CI** | Single Juju command per cloud | Per-cloud `openshift-install` |
| **Air-gapped** | Juju offline mode + air-gapped snaps | Disconnected installs via OLM + mirror registry |
| **Edge deployment** | Well-suited (LXD, microk8s, charms) | OpenShift Lightweight / Single Node (SNO) |
| **Multi-cloud management** | Single Juju controller | ACM (Advanced Cluster Management) separately |
| **VMware integration** | Juju vSphere provider | IPI with vSphere CSI + CPI |
| **IBM mainframe** | Not supported | Supported (Z, LinuxONE, Power) |

---

## 7. Add-Ons and Ecosystem

### 7.1 Charmed Kubernetes (Charm Ecosystem)

Charmed Kubernetes relies on the Juju charm ecosystem. Key charms and bundles:

**Observability (Cosmic LMA2 / COS Lite):**

| Component | Charm | Description |
|-----------|-------|-------------|
| Metrics | `prometheus2` + `grafana` | Prometheus/Grafana stack |
| Logging | `loki` + `grafana` | Log aggregation with Loki |
| Tracing | `tempo` + `grafana` | Distributed tracing |
| Alerting | `alertmanager` | Alert routing and notification |
| Dashboards | `grafana-dashboards` | Pre-built K8s dashboards |

**Storage:**

| Component | Charm | Description |
|-----------|-------|-------------|
| Block storage | `ceph-mon` + `ceph-osd` | Ceph integration |
| Object storage | `ceph-radosgw` | S3-compatible object store |
| Filesystem | `ceph-fs` | CephFS |
| CSI driver | `ceph-csi` | CSI for Ceph RBD and CephFS |
| OpenEBS | `openebs` | Container-attached storage |

**Ingress and Networking:**

| Component | Charm | Description |
|-----------|-------|-------------|
| Ingress | `nginx-ingress` + `nginx-ingress-operator` | NGINX ingress controller |
| Ingress | `traefik-k8s` | Traefik ingress |
| DNS | `bind` | DNS integration |
| Certificates | `easyrsa` | Internal CA |
| LB | `haproxy` | Layer 4 load balancing |

**Security:**

| Component | Charm | Description |
|-----------|-------|-------------|
| Secrets | `vault` | HashiCorp Vault integration |
| Auth | `dex-auth` | OpenID Connect for Kubernetes |
| Encryption | `easyrsa` | TLS certificate management |

**AI/ML:**

| Component | Charm | Description |
|-----------|-------|-------------|
| Kubeflow | `kubeflow` | Full Kubeflow ML platform |
| MLflow | `mlflow` | ML experiment tracking |

**Compute and Batch:**

| Component | Charm | Description |
|-----------|-------|-------------|
| GPU | `nvidia-gpu` | NVIDIA GPU operator |
| Batch | `slurm` | HPC workload manager |

### 7.2 OpenShift (OperatorHub Ecosystem)

OpenShift's ecosystem centres on the **Operator Lifecycle Manager (OLM)** and **OperatorHub** — Red Hat's curated marketplace of Kubernetes Operators.

**Built-in (no additional install):**

| Service | Implementation | Openshift Capability |
|---------|---------------|---------------------|
| Monitoring | Cluster Monitoring (Prometheus + Grafana + AlertManager) | Included, pre-configured |
| Logging | Cluster Logging (Loki + OpenShift Logging) | Installable via Operator |
| Registry | Integrated Image Registry | Included, auto-configured |
| Router | HAProxy-based or OpenShift Router | Included |
| Developer Console | Web console with topology, pipelines, serverless | Included |
| Authentication | Built-in OAuth/OIDC, htpasswd, LDAP, GitHub, etc. | Included |
| Secrets | Built-in secrets + cert-manager Integration | Included |

**OperatorHub (partial list, 200+ certified operators):**

| Category | Operators |
|----------|-----------|
| **CI/CD** | OpenShift Pipelines (Tekton), OpenShift GitOps (ArgoCD), Jenkins, GitLab Runner |
| **Serverless** | OpenShift Serverless (Knative Serving + Eventing) |
| **Service Mesh** | OpenShift Service Mesh (Istio + Kiali + Jaeger + Elasticsearch) |
| **Distributed Tracing** | Red Hat OpenShift Distributed Tracing (Jaeger, Tempo) |
| **Storage** | OpenShift Data Foundation (ODF — Ceph + NooBaa), TopoLVM, OpenShift Virtualization |
| **Security** | Red Hat Advanced Cluster Security (ACS — StackRox), cert-manager, Vault, External Secrets |
| **Databases** | CrunchyData PostgreSQL, MongoDB, Redis, MySQL, MariaDB, CockroachDB |
| **Messaging** | Apache Kafka (Strimzi), AMQ, RabbitMQ |
| **AI/ML** | OpenShift AI (Red Hat OpenShift AI — Kserve, ModelMesh), NVIDIA GPU Operator |
| **Networking** | Cilium, Calico, Multus, OVN-Kubernetes (built-in) |
| **Edge** | OpenShift Lightweight, Single Node OpenShift (SNO) |
| **Management** | Red Hat Advanced Cluster Management (ACM — Kubernetes multi-cluster lifecycle) |

### 7.3 Ecosystem Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Add-on discovery** | Charmhub.io | OperatorHub |
| **Number of certified add-ons** | ~200 charms | 300+ certified Operators |
| **Built-in monitoring** | Not included (charm available) | Included (Cluster Monitoring) |
| **Built-in logging** | Not included (charm available) | Included (Cluster Logging via Loki) |
| **Built-in image registry** | Not included | Included (auto-configured) |
| **Built-in router** | Not included | Included (HAProxy) |
| **Built-in developer console** | None | Full web console + DevConsole |
| **Built-in CI/CD** | None (charmable via Jenkins, Tekton) | OpenShift Pipelines (Tekton) + GitOps |
| **Built-in serverless** | None | Knative-based |
| **Built-in service mesh** | None | Istio-based |
| **Built-in distributed tracing** | None | Jaeger / Tempo |
| **AI/ML platform** | Charmed Kubeflow | OpenShift AI |
| **Multi-cluster management** | Juju controller (native) | ACM (add-on) |

---

## 8. Networking

### 8.1 Charmed Kubernetes Networking

Charmed Kubernetes supports pluggable CNI providers selected at deploy time via charms:

| CNI | Default | Characteristics |
|-----|---------|-----------------|
| **Calico** | Yes | BGP-based networking, NetworkPolicies, eBPF mode (newer), DSR |
| **Cilium** | Optional | eBPF-based, Hubble observability, NetworkPolicies, cluster mesh, L7 policies |
| **Flannel** | Optional | Simple overlay vxlan, no NetworkPolicies (requires calico for policy) |
| **Multus** | Optional (add-on) | Multi-network support — attach secondary interfaces (SR-IOV, DPDK, MACVLAN) |
| **Kube-OVN** | Optional | OVN-based, enterprise features |

**Key networking features:**
- **Network Policies** supported (Calico or Cilium provide full enforcement)
- **Load balancing:** Charm-selectable (MetalLB, HAProxy, or cloud LB integration)
- **Ingress:** NGINX or Traefik charm (configurable)
- **Service mesh:** Not built-in, but Istio or Linkerd can be charmed
- **DNS:** `easyrsa` charm for internal DNS; `bind` charm for external
- **Dual-stack:** Supported (IPv4/IPv6) via CNI configuration
- **Egress controls:** CiliumClusterwideNetworkPolicy or Calico GlobalNetworkPolicy for egress filtering

### 8.2 OpenShift Networking

OpenShift has a built-in, integrated networking stack:

**Default SDN — OVN-Kubernetes:**

```
┌─────────────────────────────────────────────┐
│               OVN-Kubernetes                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │
│  │geneve   │  │Network  │  │Egress       │  │
│  │overlay  │  │Policies │  │Services     │  │
│  └─────────┘  └─────────┘  └─────────────┘  │
│  ┌─────────┐  ┌─────────┐  ┌─────────────┐  │
│  │Subnet   │  │ARP      │  │DNS          │  │
│  │Mgmt     │  │Proxy    │  │Integration  │  │
│  └─────────┘  └─────────┘  └─────────────┘  │
└─────────────────────────────────────────────┘
```

**Networking components:**

| Component | Default Technology | Description |
|-----------|-------------------|-------------|
| **CNI/SDN** | OVN-Kubernetes | Geneve overlay, full NetworkPolicy support, distributed ACLs |
| **Ingress/router** | HAProxy-based OpenShift Router (hostNetwork) | HTTP/HTTPS, SNI, re-encrypt, edge termination, passthrough |
| **DNS** | CoreDNS (built-in) | Cluster DNS, plus BIND integration for external |
| **Service mesh** | OpenShift Service Mesh (Istio + Kiali + Jaeger) | Add-on via Operator |
| **Egress** | EgressRouter, EgressIP, EgressFirewall | Multiple options for controlled outbound traffic |
| **Multus** | CNI plugin | Secondary network interfaces (SR-IOV, DPDK, VLAN) |
| **Network Policies** | OVN-Kubernetes (native) | Full Kubernetes NetworkPolicy + AdminNetworkPolicy |
| **Load balancer** | MetalLB (bare metal) or cloud LB | In-cluster LB for bare metal via MetalLB Operator |

**Key networking features:**
- **EgressIP:** Assign a stable IP to egress traffic from a namespace, enabling firewall whitelisting.
- **EgressRouter:** Dedicated pod acting as egress proxy for cluster traffic.
- **EgressFirewall:** Simple allow/deny rules for outbound traffic per namespace.
- **AdminNetworkPolicy:** Cluster-scope network policies (beta) for multi-tenant controls.
- **DNS wildcards:** `*.apps.<cluster>` resolves to the router by default.
- **Integrated ingress:** Every Route created via `oc expose` automatically gets a DNS name via the router.

### 8.3 Networking Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Default CNI** | Calico | OVN-Kubernetes |
| **CNI flexibility** | Multiple choices (Calico, Cilium, Flannel, Kube-OVN) | OVN-K default; Cilium and Calico via Operators |
| **Built-in ingress** | Not included (charm-based) | Fully integrated HAProxy router |
| **Multi-networking** | Multus (charm) | Multus (Operator) |
| **Egress controls** | Cilium/Calico network policies | EgressIP, EgressRouter, EgressFirewall, AdminNetworkPolicy |
| **Service mesh** | None (installable) | Integrated Service Mesh (Istio) |
| **DNS integration** | CoreDNS + bind charm | CoreDNS + BIND + Router-created wildcards |
| **Network observability** | Via Cilium Hubble (if Cilium chosen) | OVN-K trace, Kiali (mesh), Network Observability (eBPF) |

---

## 9. Security Features

### 9.1 Charmed Kubernetes Security

| Feature | Implementation | Notes |
|---------|---------------|-------|
| **Host security** | AppArmor (default on Ubuntu) | LSM for mandatory access control |
| **Secrets management** | Vault charm | HashiCorp Vault for secret storage, rotation, and auditing |
| **Certificate management** | easyrsa charm or cert-manager charm | Internal CA or Let's Encrypt integration |
| **RBAC** | Kubernetes RBAC + Juju RBAC | Juju governs who can deploy/configure; K8s RBAC governs in-cluster |
| **Pod security** | Pod Security Admission (PSA) | Enforce pod security standards (baseline, restricted) |
| **Image security** | Not built-in | Can integrate with registry scanning tools (Trivy, Anchore) |
| **Network security** | Calico/Cilium NetworkPolicies | Per-namespace ingress/egress rules |
| **Encryption at rest** | etcd encryption | configurable via charm (static encryption key or Vault-backed) |
| **Encryption in transit** | TLS via easyrsa or cert-manager | All component communication via TLS |
| **AuthN/AuthZ** | K8s certificates or OIDC | OIDC via dex-auth charm; LDAP via charm integration |
| **Audit logging** | Kubernetes audit log | Configurable via kube-apiserver charm config |
| **FIPS** | Ubuntu Pro with FIPS | Available via Ubuntu FIPS charm or configuration |
| **Vulnerability scanning** | Not built-in | Ubuntu Pro includes USN + Livepatch; Trivy/charm integration |
| **Runtime security** | Not built-in | Falco charm available |

### 9.2 OpenShift Security

| Feature | Implementation | Notes |
|---------|---------------|-------|
| **Host security** | SELinux (enforced) | Mandatory Access Control at process and file level |
| **Security Context Constraints** | SCC (built-in) | Granular pod-level security policies — 8 built-in SCCs (anyuid, restricted, privileged, etc.) |
| **Pod Security Admission** | Built-in (mapped to SCC) | Dual enforcement via SCC + PSA |
| **Secrets management** | Vault or OpenShift built-in | Internal secrets + External Secrets Operator + Vault integration |
| **Certificate management** | cert-manager Operator | Let's Encrypt, internal CA, Vault issuer |
| **Built-in OAuth** | OAuth server (built-in) | GitHub, Google, LDAP, htpasswd, OpenID Connect, RH-SSO |
| **RBAC** | Kubernetes RBAC + OpenShift RBAC | ClusterRole, Role, ClusterRoleBinding, RoleBinding + OpenShift-specific roles |
| **Image security** | Integrated Image Registry | Image signing, image stream triggers, container image scanning via ACS |
| **Network security** | OVN-Kubernetes NetworkPolicies + AdminNetworkPolicy | Multi-tenant isolation with built-in enforcement |
| **Encryption at rest** | etcd encryption (built-in) | AES-CBC or AES-GCM; integrated with cluster install |
| **Encryption in transit** | TLS everywhere | Ingress router TLS, API server TLS, internal TLS |
| **AuthN/AuthZ** | Multiple identity providers | OAuth, LDAP, OIDC, GitHub, htpasswd, Request Header |
| **Audit logging** | Full Kubernetes audit + OCP audit | Audit log forwarding to Logging stack or external SIEM |
| **FIPS** | FIPS 140-2/140-3 validated | FIPS mode enabled at install; validated cryptographic modules |
| **Vulnerability scanning** | Red Hat ACS (StackRox) | CVE scanning, compliance checks (NIST, CIS), runtime threat detection |
| **Runtime security** | Red Hat ACS | Container drift prevention, process whitelisting, network anomaly detection |
| **Multi-tenant isolation** | Namespace isolation + SCCs | Tenant isolation via SCC + Network Policies + quota |

### 9.3 Security Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Host MAC** | AppArmor | SELinux (enforced) |
| **Pod security policy** | Pod Security Admission | SCC + PSA (dual) |
| **Built-in OAuth** | No (charm available) | Yes (built-in, multiple providers) |
| **Secrets management** | Vault charm | Built-in + Vault Integration + External Secrets Operator |
| **Image registry security** | Not built-in | Integrated registry with image streams, signing, and scanning |
| **Runtime security** | Falco charm | Red Hat Advanced Cluster Security |
| **FIPS compliance** | Via Ubuntu Pro | Built-in (FIPS validated modules) |
| **CIS benchmark** | Available (charm/kube-bench) | Built-in compliance operator |
| **Audit log forwarding** | Manual config | Built-in forwarding to Logging stack |
| **Vulnerability management** | Ubuntu Pro USN + Trivy | Red Hat ACS + bundled scanner |

---

## 10. Storage

### 10.1 Charmed Kubernetes Storage

Charmed Kubernetes uses a flexible CSI-based storage model:

| Storage Type | Implementation | Provisioner |
|-------------|---------------|-------------|
| **Block (RWO)** | Ceph RBD via ceph-csi charm | `rbd.csi.ceph.com` |
| **Block (RWO)** | OpenEBS LocalPV | `openebs.io/local` |
| **Block (RWO)** | AWS EBS | `ebs.csi.aws.com` |
| **Block (RWO)** | Azure Disk | `disk.csi.azure.com` |
| **Block (RWO)** | GCP Persistent Disk | `pd.csi.storage.gke.io` |
| **Filesystem (RWX)** | CephFS via ceph-csi charm | `cephfs.csi.ceph.com` |
| **Filesystem (RWX)** | NFS via nfs-provisioner charm | `nfs.csi.k8s.io` |
| **Filesystem (RWX)** | AWS EFS | `efs.csi.aws.com` |
| **Object Storage** | Ceph RADOS Gateway | S3 API compatible |
| **Object Storage** | MinIO charm | S3-compatible object store |

**Key characteristics:**
- **Any CSI driver** can be integrated — the platform does not mandate a specific storage backend.
- **Ceph/Rook charms** are preferred for self-managed storage (especially on bare metal/private clouds).
- **Cloud provisioners** work via Juju cloud integration (AWS EBS, Azure Disk, GCP PD are all supported).
- **Storage classes** are defined per-environment via Juju configuration.
- **Volume snapshots** supported via CSI when the provisioner supports it.

### 10.2 OpenShift Storage

OpenShift has a multi-layered storage strategy:

| Storage Type | Implementation | Provisioner |
|-------------|---------------|-------------|
| **Block (RWO)** | TopoLVM / LVMS (local) | `topolvm.io` / `openshift-storage.ceph.rook.io` |
| **Block (RWO)** | ODF Ceph RBD | `openshift-storage.rbd.csi.ceph.com` |
| **Block (RWO)** | AWS EBS CSI | `ebs.csi.aws.com` |
| **Block (RWO)** | Azure Disk CSI | `disk.csi.azure.com` |
| **Block (RWO)** | GCP PD CSI | `pd.csi.storage.gke.io` |
| **Filesystem (RWX)** | ODF CephFS | `openshift-storage.cephfs.csi.ceph.com` |
| **Filesystem (RWX)** | NFS CSI | `nfs.csi.k8s.io` |
| **Filesystem (RWX)** | AWS EFS CSI | `efs.csi.aws.com` |
| **Object Storage** | ODF NooBaa / Ceph RGW | MCG (Multi-Cloud Gateway) |
| **Shared Volume Encryption** | ODF | LUKS-based volume encryption |

**Key storage options:**

**a) OpenShift Data Foundation (ODF):**
- Red Hat's recommended storage platform for stateful workloads
- Based on Red Hat Ceph + NooBaa Multi-Cloud Gateway
- Provides block (RWO), filesystem (RWX), and object storage
- Supports local and cloud-backed modes
- Includes volume encryption, snapshots, cloning, and replication
- Licensed separately or as part of OpenShift Platform Plus

**b) TopoLVM / LVMS (Logical Volume Manager Storage):**
- Lightweight, local-storage CSI for hyperconverged workloads
- Creates thin-provisioned LVM volumes from node-local storage
- Ideal for databases and performance-sensitive workloads
- Bundled with OCP, no additional license

**c) Shared Volume Encryption (ODF):**
- ODF provides LUKS-based encryption for persistent volumes
- Key management integrated with the cluster

**d) `kubernetes-csi-addons`:**
- Volume replication, cloning, and snapshots via CSI
- Integration with ODF for replication and failover

### 10.3 Storage Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Default storage backend** | None (choice of: Ceph, OpenEBS, cloud CSI) | TopoLVM (local) or ODF (distributed) |
| **CSI driver support** | Any CSI driver (charm integration) | Any CSI driver (OperatorHub) |
| **Self-managed distributed storage** | Ceph via charms | ODF (Ceph + NooBaa) |
| **Hyperconverged storage** | OpenEBS | TopoLVM / LVMS |
| **Cloud storage integration** | Via Juju cloud integration (manual CSI) | Via CSI Operators (automated) |
| **Volume encryption** | LUKS via CSI | ODF shared volume encryption (built-in) |
| **Disaster recovery** | Via Velero charm | ODF volume replication + ACM + Velero |
| **Multi-cloud gateway** | MinIO charm | NooBaa (built-in ODF) |
| **Thin provisioning** | Via storage backend | TopoLVM thin-LVM |
| **Filesystem RWX** | CephFS or NFS | CephFS (ODF) or NFS |

---

## 11. Pricing and Licensing

### 11.1 Charmed Kubernetes

| Component | Cost | Notes |
|-----------|------|-------|
| **Charmed Kubernetes software** | Free (Apache 2.0) | Fully open source, no license fee |
| **Juju / charms** | Free (Apache 2.0) | Open source orchestration |
| **Ubuntu OS** | Free (GPL) | No cost for the OS itself |
| **Canonical Support (Ubuntu Pro)** | From $25/node/year (Infra) to $500/node/year (Full) | Includes security patching, 24/7 support, Livepatch |
| **Canonical K8s Support** | Included in Ubuntu Pro + K8s Support | Adds K8s-specific SLA and support |
| **Managed K8s (Canonical)** | Per-cluster/month | Fully managed service by Canonical |

**Summary:** The platform software itself costs nothing. The cost comes from optional Canonical support subscriptions (Ubuntu Pro + K8s Support), priced per node per year. For organisations comfortable with self-support, the total cost of ownership can be very low.

### 11.2 Red Hat OpenShift

| Edition | Pricing Model | Typical Cost (est.) | Notes |
|---------|--------------|---------------------|-------|
| **OKD (community)** | Free | $0 | Community-supported, no SLA, no backports |
| **OpenShift Container Platform** | Per core (vCPU) subscription | ~$5,000–$10,000/core/year (varies) | Full OCP with Red Hat support |
| **OpenShift Platform Plus** | Per core subscription | ~$10,000–$15,000/core/year | OCP + ACS + ACM + Quay |
| **OpenShift Dedicated** | Per cluster/hour + node | Variable | Managed by Red Hat on AWS/GCP |
| **ROSA (Red Hat OpenShift on AWS)** | Per cluster + node | Pay-as-you-go on AWS | Managed by Red Hat and AWS |
| **ARO (Azure Red Hat OpenShift)** | Per node | Pay-as-you-go on Azure | Managed jointly by Red Hat and Azure |

**Pricing details:**
- **OCP Standard:** For any infrastructure, priced per core (minimum 2 cores per worker node).
- **OpenShift Platform Plus:** OCP + ACS (Advanced Cluster Security) + ACM (Advanced Cluster Management) + Quay (container registry).
- **OpenShift Sandbox:** Free, limited environment for development and testing.
- **Additional costs:** ODF storage is licensed separately per TB. Red Hat support is included in the subscription.
- **VMware-specific:** OCP on vSphere requires subscription; vSphere licenses separate.

### 11.3 Pricing Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Software license** | Free | Paid subscription |
| **Pricing unit** | Per node (support) | Per vCPU core |
| **Minimum commitment** | None | 2 cores/worker node minimum |
| **Typical 3-node cluster/year** | $0–$1,500 (support-only) | $30,000–$90,000+ (est.) |
| **OS license** | Free (Ubuntu) | Included (RHCOS — no separate cost) |
| **Support add-ons** | Optional (Ubuntu Pro tiers) | Included in subscription |
| **Disconnected/air-gapped** | Same pricing | Same pricing |
| **Development environments** | Free (no support needed) | Free OKD or Sandbox |
| **Managed option** | Canonical Managed K8s | OpenShift Dedicated, ROSA, ARO |

---

## 12. Enterprise Support and SLAs

### 12.1 Canonical Support (Charmed Kubernetes)

| Tier | SLA | Features | Typical Use Case |
|------|-----|----------|-----------------|
| **Ubuntu Pro (Infra)** | 24/7 — 1 hour response (critical) | Security patching, Landscape management, Livepatch, 10-year ESM | Basic K8s support, Linux-only needs |
| **Ubuntu Pro + K8s Support** | 24/7 — 30 min response (critical) | Full control plane + worker support, charm support, upgrade guidance, architecture reviews | Production K8s |
| **Canonical Managed K8s** | Full operations by Canonical | Cluster deployed, operated, upgraded, patched by Canonical SRE team | Hands-off operations |

**Support details:**
- **24/7 phone and ticket support** at higher tiers
- **Knowledge base** access (Ubuntu Pro portal)
- **Landscape management** for fleet-level Ubuntu management
- **CVE patching** with guaranteed SLA (Ubuntu Pro)
- **Livepatch** for kernel patching without reboot
- **FIPS and Common Criteria** certifications available

### 12.2 Red Hat Support (OpenShift)

| Tier | SLA | Features | Typical Use Case |
|------|-----|----------|-----------------|
| **Standard (Standard)** | 24/7 — 1 hour response (urgent) | Ticket + phone support, knowledge base, errata, backports | Production workloads |
| **Premium** | 24/7 — 30 min response (urgent) | Same as Standard + named support engineer, proactive monitoring, quarterly reviews | Critical production |
| **Premium (Cluster Health)** | 24/7 — 30 min response | Adds proactive health monitoring by Red Hat SRE | Mission-critical |

**Support details:**
- **Included with subscription** — no separate support purchase needed
- **Red Hat Customer Portal** — knowledge base, case management, errata notifications
- **Backport security fixes** to supported OCP versions (typically 3 versions)
- **OpenShift upgrade assistance** via Red Hat Support tools
- **must-gather integration** — support requests include pre-collected cluster diagnostics
- **Red Hat Consulting** available (separate engagement)

### 12.3 Support Comparison

| Aspect | Charmed Kubernetes (Canonical) | Red Hat OpenShift |
|--------|-------------------------------|-------------------|
| **Support model** | Per-node/year subscription | Included with per-core subscription |
| **Critical response time** | 30 min (Pro + K8s) | 30 min (Premium) |
| **Support included in base price** | No (optional add-on) | Yes |
| **Managed service option** | Canonical Managed K8s | OpenShift Dedicated / ROSA / ARO |
| **Third-party support** | Supported via Canonical partner network | Supported via Red Hat partner network |
| **Backport policy** | Ubuntu LTS (10 years ESM) + snap backports | 3 concurrent OCP versions, ~18 months each |
| **Proactive monitoring** | Via Landscape (add-on) | Premium tier includes health checks |

---

## 13. Community and Adoption

### 13.1 Charmed Kubernetes Community

**Community size and health:**
- **GitHub stars (Juju):** ~2,000+
- **Charmhub.io:** ~400 published charms
- **Discourse forums:** Active (Canonical-operated)
- **Matrix/IRC:** Active community channels
- **Conferences:** KubeCon, Ubuntu Summit, Canonical events

**Notable adopters and use cases:**

| Sector | Examples | Use Case |
|--------|----------|----------|
| **Telecommunications** | Nokia, Bell Canada, AT&T (exploring) | NFVI, 5G core, network functions (CNFs) |
| **Financial services** | Select banks in Europe, S.E. Asia | Cost-sensitive deployments, region-specific compliance |
| **HPC/Research** | CERN, various universities | HPC workload orchestration |
| **AI/ML** | Multiple enterprises via Charmed Kubeflow | End-to-end ML pipelines |
| **Edge/IoT** | Canonical edge deployments | Lightweight K8s at the edge |
| **Public sector** | Various governments (EU) | Sovereign cloud requirements |

**Community characteristics:**
- Primarily Ubuntu-centric — strongest within the Ubuntu ecosystem
- Smaller than OpenShift, but engagement is high in specific verticals (telco, HPC)
- Strong contributor to upstream K8s (Canonical runs 40% of k8s CI test jobs)
- Juju community is separate from K8s community — adds to learning curve
- Documentation quality is good but less curated than Red Hat's

### 13.2 OpenShift Community

**Community size and health:**
- **GitHub stars (OKD/Origin):** ~2,400+
- **OperatorHub:** 300+ certified Operators, 200+ community Operators
- **Red Hat Customer Portal:** Large knowledge base and community forums
- **Red Hat Developer Program:** Free sandboxes, tutorials, certifications
- **Conferences:** KubeCon, OpenShift Commons, Red Hat Summit

**Notable adopters and use cases:**

| Sector | Examples | Use Case |
|--------|----------|----------|
| **Financial services** | JPMorgan Chase, Bank of America, HSBC, Barclays, Deutsche Bank, Société Générale, Santander, ING | Largest enterprise K8s deployments, containerised trading, core banking |
| **Government** | US DoD, UK Gov, EU agencies | FedRAMP, classified workloads, IL5+, sovereign cloud |
| **Telecommunications** | Verizon, T-Mobile, Vodafone, Deutsche Telekom | 5G core, NFVI, telco cloud |
| **Retail** | Walmart, Target, Best Buy | E-commerce, supply chain, digital transformation |
| **Manufacturing** | Siemens, Bosch, Airbus | Industry 4.0, IoT, edge |
| **Healthcare** | Novartis, Roche | Clinical systems, AI/ML research |
| **Technology** | Adobe, S&P Global, Fidelity | SaaS, cloud-native platforms |

**Community characteristics:**
- Largest enterprise K8s install base in the world
- Strong upstream contributor — Red Hat is the #2 K8s contributor (after Google)
- Wide ecosystem of certified partners and ISVs
- Extensive documentation, training, and certification (EX280, EX380)
- Active community via OpenShift Commons, a vendor-neutral user group

### 13.3 Community and Adoption Comparison

| Aspect | Charmed Kubernetes | Red Hat OpenShift |
|--------|-------------------|-------------------|
| **Enterprise install base** | Smaller but growing (telco, HPC) | Largest enterprise K8s install base |
| **K8s upstream contribution** | Significant (CI, testing) | #2 contributor overall |
| **Certified operator ecosystem** | ~200 charms | 300+ certified Operators |
| **Documentation quality** | Good | Excellent (extensive guides, tutorials, courses) |
| **Training & certification** | Ubuntu + Canonical K8s training | EX280/EX380 certified + DO series courses |
| **Cloud provider integrations** | Good (via Juju) | Extensive (via CSI, CPI, OLM, cloud-specific Operators) |
| **SI/partner ecosystem** | Moderate (Canonical + partners) | Extensive (Red Hat, IBM, Accenture, Deloitte, etc.) |
| **Market perception** | Niche, cost-effective, emerging enterprise | Industry standard enterprise K8s |

---

## 14. Strengths and Weaknesses

### 14.1 Charmed Kubernetes — Strengths

| Strength | Detail |
|----------|--------|
| **Fully open source** | No license fees; Apache 2.0 license for all components |
| **Juju model-driven operations** | Declarative state management simplifies Day 1 and Day 2 operations — Juju reconciles desired state continuously |
| **Multi-cloud abstraction** | One workflow (juju deploy) works across bare metal, private cloud, and all major public clouds |
| **Modular and composable** | Deploy only what you need — no heavyweight defaults or unnecessary components |
| **No license costs** | Free to use at any scale; pay only for optional Canonical support |
| **Strong bare metal story** | Deep MAAS integration makes bare metal provisioning as easy as cloud |
| **Telco/NFVI strength** | CNF-friendly, low-touch, modular design well-suited to 5G/NFV workloads |
| **Ubuntu ecosystem** | Seamless integration with Ubuntu Pro, Landscape, Livepatch, 10-year ESM |
| **Charmed Kubeflow** | A full, integrated Kubeflow ML platform for MLOps |

### 14.2 Charmed Kubernetes — Weaknesses

| Weakness | Detail |
|----------|--------|
| **Smaller ecosystem** | Fewer charms than OpenShift Operators; fewer certified ISV integrations |
| **Juju learning curve** | Juju is a separate model-driven orchestration system that operators must learn — not standard K8s tooling |
| **Fewer built-in features** | No built-in monitoring, logging, registry, router, or developer console — must be added manually via charms |
| **Smaller community** | Smaller pool of talent, fewer SI partners, less third-party tooling support |
| **Less enterprise OOTB** | Not a complete platform — requires assembly and configuration of desired components |
| **Weakest in financial services** | Minimal adoption in banking or regulated industries compared to OpenShift |
| **Charm maturity varies** | Some charms are well-maintained; others lag behind upstream releases |
| **Limited certification path** | Fewer training/certification offerings than Red Hat |

### 14.3 OpenShift — Strengths

| Strength | Detail |
|----------|--------|
| **Complete enterprise platform OOTB** | Everything included — monitoring, logging, registry, router, dev console, auth, security — no assembly required |
| **Largest enterprise K8s ecosystem** | 300+ certified Operators, extensive ISV partnerships, broad SI network |
| **Built-in everything** | Cluster Monitoring, Cluster Logging, integrated registry, HAProxy router, OAuth, DevConsole, pipelines, serverless, service mesh, tracing |
| **Security Context Constraints (SCC)** | Granular, robust pod security that predates and exceeds upstream PSA |
| **SELinux (enforced)** | Mandatory access control at the OS level for defence in depth |
| **Strong upstream contributor** | #2 K8s contributor; Red Hat engineers commit to every layer of the upstream stack |
| **Largest enterprise install base** | Most widely deployed enterprise K8s platform — proven at massive scale |
| **Financial services dominance** | Widely adopted by global banks, insurance, and regulated financial institutions |
| **FIPS compliance** | FIPS-validated cryptographic modules; certified for US government workloads |
| **Subscription includes support** | Red Hat support is bundled with the subscription — no separate purchase needed |
| **Extensive training & certification** | Industry-recognised OpenShift certification paths (EX280, EX380) |

### 14.4 OpenShift — Weaknesses

| Weakness | Detail |
|----------|--------|
| **Expensive per-core licensing** | Significant cost at scale — per-vCPU subscriptions add up quickly, especially for large clusters |
| **Heavy default footprint** | Full OCP includes many components by default; resource overhead is substantially higher than vanilla K8s |
| **Slower to adopt upstream innovations** | Red Hat stabilises and backports; newer upstream features may be delayed by one or two releases |
| **Opinionated defaults** | Some architectural choices (RHCOS immutability, default networking, operator-only management) limit flexibility |
| **Complex troubleshooting** | Without must-gather, OCP-specific operator faults can be hard to diagnose; deep knowledge of OCP internals required |
| **RHEL dependency** | RHCOS is the only supported OS — no choice of operating system version or distribution |
| **Vendor lock-in risk** | Once deeply invested in OCP-specific APIs (Routes, SCCs, BuildConfigs), migration difficulty increases |
| **Upgrade complexity** | y-stream upgrades can be disruptive if an intermediate version path is required; EUS upgrades need careful planning |

---

## 15. Use Case Comparison

### 15.1 Charmed Kubernetes — Best For

**1. Cost-Sensitive Deployments**
Organisations that need production Kubernetes at scale but cannot justify per-core licensing costs. Charmed Kubernetes is free; only support is optional.

**2. Ubuntu-Standardised Organisations**
Teams already running Ubuntu as their standard server OS benefit from seamless integration: the same security patching (Ubuntu Pro), tooling (Landscape), and operational practices extend naturally to K8s.

**3. Bare Metal with MAAS**
The MAAS + Juju + Charmed Kubernetes combination is arguably the most mature automated K8s-on-bare-metal story in open source. Ideal for organisations running their own hardware.

**4. Telecommunications / NFVI**
Telco workloads (5G core, CNFs, vCPE) benefit from Charmed K8s' modularity, low latency networking (DPDK/SR-IOV via Multus), and Juju's model-driven operations for lifecycle management across distributed edge sites.

**5. Custom Infrastructure / Edge**
Organisations with non-standard infrastructure requirements — disconnected environments, small footprint clusters, IoT/edge nodes — benefit from the ability to build exactly the cluster they need, no more, no less.

**6. AI/ML with Kubeflow**
The Charmed Kubeflow bundle provides an integrated AI/ML platform that can be deployed alongside Charmed Kubernetes on any infrastructure.

**7. Dev/Test/Staging Environments**
With no license cost, organisations can spin up as many clusters as needed for development, testing, and staging without budget concerns.

**8. Smaller Deployments**
For clusters under 10 nodes, the cost advantage of Charmed K8s is most pronounced, and the light modular footprint is an operational benefit.

### 15.2 OpenShift — Best For

**1. Enterprise Regulated Environments**
Banks, insurance, healthcare, and government organisations operating under strict compliance regimes (PCI-DSS, HIPAA, SOC2, FedRAMP, GDPR) benefit from OpenShift's built-in security controls, audit logging, and certification.

**2. Security-First Organisations**
SCC + SELinux + built-in OAuth + FIPS + runtime security (ACS) provides a defence-in-depth posture that is unmatched by any other K8s distribution out of the box.

**3. Full-Featured Container Platform**
Teams that want a complete application platform — not just a K8s distribution — value having monitoring, logging, registry, pipelines, serverless, service mesh, and tracing built in.

**4. Large Enterprises with RHEL Standard**
Organisations that standardise on Red Hat Enterprise Linux find OpenShift a natural extension of their existing OS, support, and vendor relationship.

**5. Mixed Workloads**
Teams running a variety of workload types — stateless microservices, stateful databases, batch jobs, AI/ML training, serverless functions — benefit from the breadth of built-in capabilities.

**6. Financial Services (Global Banks)**
OpenShift is the de facto standard container platform in global banking. JPMorgan, HSBC, Deutsche Bank, Société Générale, and many others run OCP in production. This means easier talent acquisition, more third-party integrations, and community knowledge.

**7. Multi-Cluster / Federation**
For organisations running 10+ clusters across multiple data centres and clouds, OpenShift + ACM provides mature multi-cluster lifecycle management, policy governance, and application placement.

**8. Organisations Needing Vendor Assurance**
Red Hat is a $5B+ company with IBM backing — for organisations that require long-term vendor stability, enterprise SLAs, and a single throat to choke, OpenShift delivers.

---

## 16. Decision Matrix

Use the following decision matrix to evaluate which platform fits a given scenario. Score each criterion from **1–5** (5 = strongly favours this platform).

| # | Criterion | Weight | Charmed K8s | OpenShift | Notes |
|---|-----------|--------|-------------|-----------|-------|
| **C1** | **Cost sensitivity** | ___/10 | ⭐⭐⭐⭐⭐ (5) | ⭐⭐ (2) | If budget is primary constraint, Charmed K8s wins decisively |
| **C2** | **Existing OS standard** | ___/10 | | | |
| | • Ubuntu standard | | ⭐⭐⭐⭐⭐ (5) | ⭐⭐ (2) | Ubuntu shops — Charmed is natural fit |
| | • RHEL standard | | ⭐⭐ (2) | ⭐⭐⭐⭐⭐ (5) | RHEL shops — OpenShift is natural fit |
| **C3** | **Enterprise features needed OOTB** | ___/10 | ⭐⭐ (2) | ⭐⭐⭐⭐⭐ (5) | OpenShift is a complete platform; Charmed needs assembly |
| **C4** | **Security requirements** | ___/10 | | | |
| | • Standard security (RBAC, PSA, TLS) | | ⭐⭐⭐ (3) | ⭐⭐⭐⭐ (4) | Both adequate; OpenShift has more built-in |
| | • High security (SCC, SELinux, FIPS, ACS) | | ⭐⭐ (2) | ⭐⭐⭐⭐⭐ (5) | OpenShift dominates for regulated environments |
| **C5** | **Infrastructure flexibility** | ___/10 | ⭐⭐⭐⭐⭐ (5) | ⭐⭐⭐ (3) | Charmed's Juju abstraction covers more platforms uniformly |
| **C6** | **Team Kubernetes expertise** | ___/10 | | | |
| | • K8s-native team (yaml, helm, operators) | | ⭐⭐⭐ (3) | ⭐⭐⭐⭐ (4) | OpenShift is more accessible to K8s-native teams |
| | • Juju/Ubuntu team | | ⭐⭐⭐⭐⭐ (5) | ⭐⭐ (2) | If team knows Juju, Charmed is lower friction |
| **C7** | **Support requirements** | ___/10 | | | |
| | • Community support acceptable | | ⭐⭐⭐⭐⭐ (5) | ⭐⭐⭐ (3) | Charmed full OSS; OKD is community OpenShift |
| | • Vendor-backed SLA required | | ⭐⭐⭐ (3) | ⭐⭐⭐⭐⭐ (5) | Red Hat support bundled; Canonical support extra |
| **C8** | **Ecosystem requirements** | ___/10 | ⭐⭐ (2) | ⭐⭐⭐⭐⭐ (5) | OpenShift's OperatorHub + ISV network is much larger |
| **C9** | **Talent availability** | ___/10 | ⭐⭐ (2) | ⭐⭐⭐⭐ (4) | More OpenShift engineers available in market |
| **C10** | **Bare metal / on-prem** | ___/10 | ⭐⭐⭐⭐⭐ (5) | ⭐⭐⭐⭐ (4) | Both strong; Charmed + MAAS is exceptionally mature |
| **C11** | **Regulated industry (finance, gov, health)** | ___/10 | ⭐⭐ (2) | ⭐⭐⭐⭐⭐ (5) | OpenShift is the market standard in regulated sectors |

### Scoring Guide

**To use the matrix:**

1. Assign a weight (1–10) to each criterion based on your specific requirements.
2. Multiply weight × platform score for each row.
3. Sum each platform column for a total weighted score.

**Example: Financial Services Bank (RHEL shop)**

| # | Criterion | Weight | Charmed K8s (× weight) | OpenShift (× weight) |
|---|-----------|--------|----------------------|---------------------|
| C1 | Cost sensitivity | 3 | 15 | 6 |
| C2 | RHEL standard | 9 | 18 | 45 |
| C3 | Enterprise features OOTB | 8 | 16 | 40 |
| C4 | High security | 10 | 20 | 50 |
| C5 | Infrastructure flexibility | 6 | 30 | 18 |
| C6 | Team expertise | 5 | 15 | 20 |
| C7 | Vendor SLA required | 9 | 27 | 45 |
| C8 | Ecosystem | 7 | 14 | 35 |
| C9 | Talent availability | 4 | 8 | 20 |
| C10 | Bare metal | 5 | 25 | 20 |
| C11 | Regulated industry | 10 | 20 | 50 |
| **Total** | | | **208** | **349** |

**Result:** OpenShift is the clear choice for a regulated financial institution on RHEL.

**Example: Telco NFVI (Ubuntu shop, cost-sensitive)**

| # | Criterion | Weight | Charmed K8s (× weight) | OpenShift (× weight) |
|---|-----------|--------|----------------------|---------------------|
| C1 | Cost sensitivity | 9 | 45 | 18 |
| C2 | Ubuntu standard | 9 | 45 | 18 |
| C3 | Enterprise features OOTB | 4 | 8 | 20 |
| C4 | Security | 6 | 18 | 30 |
| C5 | Infrastructure flexibility | 8 | 40 | 24 |
| C6 | Team expertise (Juju) | 7 | 35 | 14 |
| C7 | Support | 5 | 15 | 25 |
| C8 | Ecosystem | 3 | 6 | 15 |
| C9 | Talent availability | 3 | 6 | 12 |
| C10 | Bare metal | 10 | 50 | 40 |
| C11 | Regulated industry | 4 | 8 | 20 |
| **Total** | | | **276** | **236** |

**Result:** Charmed Kubernetes is the better choice for a cost-conscious telco on Ubuntu with strong bare metal requirements.

---

## 17. Summary and Recommendations

### When to Choose Charmed Kubernetes

Choose Charmed Kubernetes when:
- **Cost is the primary constraint** — no license fees at any scale
- **Ubuntu is the standard OS** — you're already invested in the Canonical ecosystem
- **You need maximum infrastructure flexibility** — especially bare metal via MAAS
- **You want a modular, lightweight platform** — deploy only what you need
- **You're in telecommunications / NFVI / edge** — the telco-optimised sweet spot
- **Your team is comfortable with Juju** — the learning curve is not a barrier
- **You need Kubeflow on any infrastructure** — Charmed Kubeflow integrates seamlessly

### When to Choose OpenShift

Choose OpenShift when:
- **You work in regulated financial services, government, or healthcare** — OpenShift is the market standard
- **Security is your top priority** — SCC + SELinux + FIPS + ACS provide unmatched depth
- **You need a complete platform out of the box** — no assembly required
- **RHEL is your standard OS** — OpenShift extends your existing Red Hat investment
- **You need certified operator and ISV integrations** — the largest enterprise ecosystem
- **You need enterprise support bundled with the product** — subscription includes support
- **Talent availability matters** — more OpenShift engineers and documentation exist
- **You're running multi-cluster federated deployments** — ACM provides mature lifecycle management

### Final Thoughts

Both Charmed Kubernetes and Red Hat OpenShift are production-grade, CNCF-conformant Kubernetes platforms. The choice is not a technical capability question — both can run any Kubernetes workload — but a **strategy, cost, and operating model decision**.

Charmed Kubernetes offers **freedom, modularity, and low cost** at the expense of ecosystem size and out-of-box convenience. It excels in environments where teams want minimal platform overhead and maximum infrastructure choice.

OpenShift offers **completeness, security depth, and enterprise ecosystem** at a premium price point. It excels in environments where compliance, security, and vendor assurance carry more weight than cost savings.

For a Solution Architect evaluating these platforms, the recommended approach is:
1. **Score the Decision Matrix** with your organisation's specific weights.
2. **Run a proof-of-concept** on both platforms with a representative workload.
3. **Evaluate total cost of ownership** including license fees, operational overhead, and training costs.
4. **Consider migration paths** — Charmed K8s is closer to vanilla K8s, making migration easier. OpenShift's custom APIs increase stickiness.

---

> **Disclaimer:** Pricing and feature details are based on publicly available information as of July 2026 and may vary by region, agreement, and Red Hat/Canonical partner terms. Always verify current pricing with the vendor.

---

*End of document — 670 lines*
