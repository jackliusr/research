# Charmed Kubeflow vs Red Hat OpenShift AI: Comprehensive Technical Comparison

> **Author:** Jack Liu Shurui  
> **Role:** Solution Architect, Crédit Agricole CIB  
> **Date:** July 2026  
> **Version:** 1.0  
> **Repository:** github.com/jackliusr/research  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Platform Overviews](#2-platform-overviews)
3. [Architecture Deep Dive](#3-architecture-deep-dive)
4. [Deployment and Lifecycle Management](#4-deployment-and-lifecycle-management)
5. [Supported Infrastructure](#5-supported-infrastructure)
6. [Component-by-Component Comparison](#6-component-by-component-comparison)
7. [GPU Support and Acceleration](#7-gpu-support-and-acceleration)
8. [Model Serving Options](#8-model-serving-options)
9. [IDE and User Experience](#9-ide-and-user-experience)
10. [Ecosystem Integration](#10-ecosystem-integration)
11. [Security and Governance](#11-security-and-governance)
12. [Pricing and Licensing](#12-pricing-and-licensing)
13. [Use Cases](#13-use-cases)
14. [Strengths and Weaknesses](#14-strengths-and-weaknesses)
15. [Decision Matrix](#15-decision-matrix)
16. [Conclusion](#16-conclusion)
17. [References](#17-references)

---

## 1. Executive Summary

Charmed Kubeflow (Canonical) and Red Hat OpenShift AI are the two leading **open-source-based ML/AI platform distributions on Kubernetes**. Both are built on upstream Kubeflow but diverge sharply in architecture, deployment philosophy, infrastructure coupling, and target audience.

| Dimension | Charmed Kubeflow | Red Hat OpenShift AI |
|---|---|---|
| **Vendor** | Canonical (Ubuntu) | Red Hat (IBM) |
| **Base K8s** | Any CNCF-certified K8s | OpenShift only |
| **Orchestration** | Juju charms / operators | OpenShift Operators (OLM) |
| **Upstream** | Vanilla Kubeflow + Canonical charms | Open Data Hub (ODH) + Red Hat additions |
| **License** | Fully open source (Apache 2.0) | OpenShift subscription + AI add-on |
| **Deployment verb** | `juju deploy kubeflow` | OperatorHub → DataScienceCluster CR |
| **Target user** | Cost-conscious, multi-cloud, Ubuntu shops | Enterprise regulated, security-first, OpenShift shops |

---

## 2. Platform Overviews

### 2.1 Charmed Kubeflow (Canonical)

Canonical's distribution of upstream Kubeflow, deployed entirely via **Juju charms** — operator packages that encapsulate deployment, configuration, scaling, and lifecycle management of each component.

**Key characteristics:**
- **Juju-orchestrated:** Every Kubeflow sub-project is a Juju charm. Charms handle component relations (Istio → KServe, Pipelines → MLMD), upgrades, and Day-2 ops.
- **Distribution-agnostic:** Runs on any CNCF-certified Kubernetes — MicroK8s, Charmed Kubernetes, EKS, AKS, GKE, even OpenShift itself.
- **Upstream-aligned:** Tracks upstream Kubeflow releases; contributes fixes upstream.
- **Modular:** Deploy the full bundle or select only the charms you need.
- **NVIDIA DGX certified:** One of few MLOps platforms validated for NVIDIA DGX systems.
- **Ecosystem:** Charmed MLflow, Charmed Feast, Spark Operator, Kafka, PostgreSQL — all Juju-wrapped.

### 2.2 Red Hat OpenShift AI

Red Hat's ML/AI platform (formerly OpenShift Data Science, RHOAI), deployed as an Operator on OpenShift. Evolved from the Open Data Hub (ODH) project.

**Key characteristics:**
- **ODH-based:** Built on Open Data Hub components with Red Hat enterprise hardening, testing, and support.
- **OpenShift-native:** Requires OpenShift. Deep integration with OpenShift Pipelines (Tekton), Serverless (Knative), Service Mesh (Istio), Monitoring (Prometheus/Alertmanager), GitOps (ArgoCD).
- **Operator-driven lifecycle:** Deployed via OLM. Single `DataScienceCluster` custom resource controls the entire platform.
- **Self-Managed + Cloud Service:** Available as Operator on OpenShift or managed cloud service.
- **Enterprise-focused:** SCC-based security, FIPS compliance, audit logging, OIDC/LDAP auth.
- **Components:** Workbenches (notebooks), DSP (Tekton-based), KServe, ModelMesh (deprecated in 2.19), CodeFlare + Ray (CodeFlare deprecated in 2.24), Kueue, Model Registry, TrustyAI.

---

## 3. Architecture Deep Dive

### 3.1 Charmed Kubeflow Architecture

Layered Juju model:

```
┌──────────────────────────────────────────────────┐
│  Kubeflow Components (KFP, Notebooks, Katib,     │
│  KServe, MLMD, TensorBoard, Feast)               │
├──────────────────────────────────────────────────┤
│  Istio (Ingress + mTLS)                          │
│  Dex / OIDC (Authentication)                     │
│  Kubeflow Profiles (Multi-Tenancy)               │
├──────────────────────────────────────────────────┤
│  Juju Charms (operator lifecycle per component)  │
├──────────────────────────────────────────────────┤
│  Any CNCF Kubernetes (MicroK8s, EKS, AKS, GKE)  │
└──────────────────────────────────────────────────┘
```

Each Juju charm packages a Kubernetes operator, declares **relations** to other charms (e.g., `kserve` relates to `istio-pilot` and `knative-serving`), and supports `juju refresh` for rolling upgrades. The Kubeflow bundle (`juju deploy kubeflow`) deploys ~35 charms with relations wired automatically.

| Component | Charm Name | Upstream Project |
|---|---|---|
| Central Dashboard | `kubeflow-dashboard` | Kubeflow Central Dashboard |
| Notebooks | `jupyter-ui`, `jupyter-controller`, `notebook-controller` | Kubeflow Notebooks |
| Pipelines | `kfp-api`, `kfp-ui`, `ml-pipeline`, etc. | Kubeflow Pipelines |
| Katib | `katib-controller`, `katib-ui` | Katib |
| KServe | `kserve-controller` | KServe |
| Feast | `charmed-feast` | Feast |
| MLMD | `mlmd` | ML Metadata |
| TensorBoard | `tensorboard-controller`, `tensorboard-web-app` | TensorBoard |
| Istio | `istio-pilot`, `istio-ingressgateway` | Istio |
| Knative | `knative-serving` | Knative |
| OIDC | `dex-auth` | Dex |
| Profiles | `kubeflow-profiles` | Kubeflow Profiles |

### 3.2 OpenShift AI Architecture

Operator-driven with a central `DataScienceCluster` CR:

```
┌──────────────────────────────────────────────────┐
│  Workbenches │ DSP │ CodeFlare+Røy │ KServe │   │
│  Model Registry │ TrustyAI │ Training Operator  │
├──────────────────────────────────────────────────┤
│  OpenShift Service Mesh (Istio)                  │
│  OpenShift Serverless (Knative)                  │
│  OpenShift Pipelines (Tekton)                    │
│  OpenShift Monitoring + GitOps                   │
├──────────────────────────────────────────────────┤
│  OLM + DataScienceCluster CR                     │
├──────────────────────────────────────────────────┤
│  Red Hat OpenShift                               │
└──────────────────────────────────────────────────┘
```

Single CR management — all component states toggled via `spec.components.<name>.managementState: Managed | Removed`. Requires OpenShift Service Mesh, Serverless, and Pipelines operators pre-installed as prerequisites.

| Component | ODH Sub-operator | Notes |
|---|---|---|
| Dashboard | `dashboard` | OpenShift AI console |
| Workbenches | `workbenches` | JupyterLab, RStudio, VS Code |
| DSP | `datasciencepipelines` | Tekton-based, KFP-compatible API |
| KServe | `kserve` | Single-model serving |
| ModelMesh | `modelmeshserving` | Deprecated in 2.19 |
| CodeFlare+Ray | `codeflare`, `ray` | CodeFlare deprecated in 2.24 |
| Kueue | `kueue` | Job queueing/scheduling |
| Training Operator | `trainingoperator` | PyTorch, TF, XGBoost, MPI |
| Model Registry | `modelregistry` | ODH Model Registry API v1beta1 |
| TrustyAI | `trustyai` | Explainability/bias detection |
| Feast Operator | `feastoperator` | Technology Preview |

### 3.3 Architecture Comparison

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Orchestration model** | Juju charms per component | Single Operator reconciling DataScienceCluster CR |
| **Component isolation** | Each charm owns its K8s resources | Sub-operators under RHOAI operator |
| **Declarative config** | Juju model config + charm relations | Single DataScienceCluster YAML |
| **Customizability** | Swap charms, add/remove per model | Enable/disable via `managementState` |
| **Learning curve** | Must learn Juju CLI + charm concepts | Must learn OpenShift + OLM concepts |
| **Upstream tracking** | Charms updated per Kubeflow release | ODH upstream → RHOAI downstream release lag |
| **Istio/Knative mgmt** | Juju charms deploy and manage | Requires pre-installed Service Mesh + Serverless operators |

---

## 4. Deployment and Lifecycle Management

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Deployment command** | `juju bootstrap <k8s>` → `juju add-model kubeflow` → `juju deploy kubeflow --trust` | Install Operator from OperatorHub → `oc apply -f DataScienceCluster.yaml` |
| **Deployment time** | 15–30 min | 10–20 min |
| **Prerequisites** | None (charms deploy everything) | OpenShift Service Mesh + Serverless + Pipelines operators |
| **Upgrade mechanism** | `juju refresh kubeflow --channel=<version>` | OLM subscription channel + CVO |
| **Rolling upgrade** | Supported (charms manage pod ordering) | Supported (Operator-managed) |
| **Rollback** | `juju refresh --revision <N>` | OLM operator version rollback |
| **Day-2 tools** | Juju dashboard, `juju status`, `juju debug-log` | OpenShift Console, `oc`, Must-gather |
| **Multi-cluster mgmt** | Juju controller (single pane) | OpenShift ACM |
| **Scale component** | `juju scale-application kfp-api 3` | Scale worker nodes via MachineSet |
| **Backup** | `juju create-backup` | OpenShift etcd backup + CR export |

**Upgrade flow comparison:** Charmed Kubeflow upgrades are charm-level — `juju refresh` updates each charm independently with relation-aware ordering. OpenShift AI upgrades are operator-level — CVO manages OpenShift upgrades; RHOAI operator upgrades through OLM channels, with version compatibility constraints across OpenShift, Service Mesh, and RHOAI.

---

## 5. Supported Infrastructure

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **K8s distributions** | Any CNCF-certified (MicroK8s, Charmed K8s, EKS, AKS, GKE, kubeadm, even OpenShift) | OpenShift only |
| **Public cloud** | EKS, AKS, GKE (native) | ROSA, ARO, GCP-IPI only |
| **On-premise** | MicroK8s, Charmed K8s, kubeadm | OpenShift (bare metal, vSphere, OpenStack) |
| **Edge** | MicroK8s (lightweight, single-node) | MicroShift (RHOAI not supported on MicroShift) |
| **Mainframe** | Not certified | IBM Z / Power (Tech Preview in RHOAI 2.25) |
| **NVIDIA DGX** | Certified | Supported (OpenShift on DGX) |
| **Multi-cloud portability** | High — move between clouds | Low — locked to OpenShift |

---

## 6. Component-by-Component Comparison

### 6.1 Jupyter Notebooks / Workbenches

| Feature | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Notebook server** | Kubeflow Notebooks (JupyterLab) | Workbenches (JupyterLab) |
| **Additional IDEs** | RStudio, VS Code (code-server images) | RStudio, VS Code (container images) |
| **GPU support** | Yes (NVIDIA GPU op + tolerations) | Yes (GPU tolerations, accelerator profiles) |
| **Custom images** | Kubeflow Notebook image registry | ImageStreams + custom workbench images |
| **Idle shutdown** | Notebook controller idle culler | Workbench idle culler (configurable) |
| **Persistent storage** | PVC per notebook | PVC per workbench |
| **Git integration** | Built-in JupyterLab Git | Built-in + OpenShift DevConsole |
| **Collaboration** | Per-profile K8s namespaces | Per-project (OpenShift) isolation |

### 6.2 Pipelines

| Feature | Charmed Kubeflow Pipelines (KFP) | OpenShift AI DSP |
|---|---|---|
| **Engine** | Argo Workflows (upstream KFP) | Tekton (via OpenShift Pipelines) |
| **API compatibility** | KFP v1 + v2 | KFP-compatible API (DSP) |
| **SDK** | `kfp` SDK (Python) | `kfp` SDK + `ds-pipelines` SDK |
| **Visual pipeline builder** | KFP v2 UI | DSP UI + OpenShift DevConsole |
| **Caching** | KFP v2 execution caching | DSP step caching |
| **Artifact tracking** | MLMD integration | MLMD + artifact tracking |
| **Recurring runs** | Yes (scheduled pipelines) | Yes (scheduled via CR) |
| **CI/CD integration** | External | OpenShift Pipelines (Tekton) — natural integration |

**Key difference:** KFP uses Argo; DSP uses Tekton (same engine as OpenShift CI/CD). DSP provides KFP-compatible API but has subtle behavioural differences in caching, parameter passing, and conditionals.

### 6.3 Training and Hyperparameter Tuning

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Hyperparameter tuning** | Katib (Bayesian, random, grid, NAS) | Katib (via Training Operator + Kueue) |
| **Distributed training** | Kubeflow Trainer (PyTorch, TF, JAX, MPI, DeepSpeed, HuggingFace, MLX) | Training Operator (PyTorch, TF, XGBoost, MPI) |
| **Ray** | Not built-in (add via charm) | Ray (via `ray` operator; CodeFlare deprecated in 2.24) |
| **Job queueing** | Not built-in | Kueue (cluster queues, resource flavours) |
| **GPU scheduling** | K8s node selectors + tolerations | Kueue + cluster queues + resource flavours |
| **LLM fine-tuning** | Kubeflow Trainer + DeepSpeed | Training Operator + Ray + DeepSpeed |

### 6.4 Model Serving

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Primary serving** | KServe (via Istio + Knative) | KServe (via OpenShift Serverless + Service Mesh) |
| **ModelMesh** | Not shipped | Deprecated in 2.19; removed in future |
| **Seldon Core** | Available (separate) | Not integrated |
| **Triton Inference Server** | Yes (via KServe) | Yes (via KServe) |
| **TF Serving / PyTorch** | Yes (via KServe runtimes) | Yes (via KServe runtimes) |
| **vLLM for LLMs** | Custom configuration | First-class (recommended for LLMs) |
| **RawDeployment mode** | KServe (bypass Knative) | KServe `RawDeployment` (no Serverless dependency) |
| **Scale-to-zero** | Yes (Knative) | Yes (OpenShift Serverless) |
| **Autoscaling** | KPA (CPU/concurrency) | KPA + HPA + advanced KServe autoscaling |
| **Canary / traffic split** | KServe + Istio | KServe + Service Mesh |
| **Explainability** | Not built-in | TrustyAI (built-in) |
| **Multi-model serving** | Multiple InferenceServices | Multiple InferenceServices (ModelMesh deprecated) |

### 6.5 Feature Stores

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Built-in feature store** | Charmed Feast (production, launched July 2025) | Feast Operator (Technology Preview) |
| **Backend options** | PostgreSQL, Redis, BigQuery, Snowflake | PostgreSQL, Redis |
| **Online/offline serving** | Yes (Redis online, Parquet/BigQuery offline) | Yes (via Feast Operator) |
| **Integration** | Feast charm relates to KFP + KServe charms | Feast Operator related to DSP + KServe |

### 6.6 Model Registry and Metadata

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Metadata tracking** | MLMD (via charm) | MLMD (via sub-operator) |
| **Model registry** | Custom (MLMD + K8s labels; MLflow recommended) | ODH Model Registry API (v1beta1) |
| **Versioning** | Manual via MLMD lineage | Versioned artifacts via Registry API |
| **UI** | TensorBoard + Metadata UI | Model Registry UI in Dashboards |
| **Registration workflow** | Pipeline output → MLMD → manual | Pipeline output → MLMD → Registry API (automated) |

**Key difference:** OpenShift AI's Model Registry provides a dedicated API + UI for model version management. Charmed Kubeflow relies on MLMD metadata; users typically add MLflow for structured model registry.

### 6.7 Monitoring and Observability

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Metrics** | Prometheus (via COS Lite / LMA2 charms) | OpenShift built-in monitoring (Prometheus + Alertmanager) |
| **Dashboards** | Grafana (via COS Lite) | OpenShift AI dashboards + Grafana (optional) |
| **Logging** | Loki (via COS Lite) | OpenShift Logging (Loki + Vector — optional) |
| **Alerts** | Alertmanager (LMA2) | Alertmanager (built-in) + PagerDuty (cloud) |
| **Model monitoring** | Custom (Prometheus + custom dashboards) | TrustyAI (explainability) + custom dashboards |
| **Distributed tracing** | Jaeger / Tempo (separate charms) | OpenShift Distributed Tracing (Jaeger/Tempo — optional) |

**Key difference:** OpenShift AI benefits from OpenShift's built-in monitoring stack. Charmed Kubeflow requires deploying COS Lite separately via Juju charms. COS Lite is fully charm-managed but adds operational overhead.

### 6.8 Multi-Tenancy

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Tenant isolation** | Kubeflow Profiles → K8s namespaces | OpenShift Projects → K8s namespaces |
| **Authentication** | Dex / OIDC → Istio mTLS | OpenShift OAuth (OIDC/LDAP/HTTPS) → Service Mesh |
| **Authorization** | Kubeflow RBAC + Istio AuthorizationPolicy | OpenShift RBAC + SCC + Service Mesh AuthPolicy |
| **Admin vs user roles** | Cluster admin + profile admin + contributor | OpenShift admin + RHOAI admin + Data Scientist |
| **Resource quotas** | K8s ResourceQuota per profile | OpenShift ResourceQuota per project |
| **GPU quota** | Istio + K8s limits per profile | Kueue + cluster queues per project |
| **Pod security** | K8s PSS (Pod Security Standards) | OpenShift SCC (more granular) |

**Key difference:** OpenShift AI's SCC provides finer-grained pod security than K8s PSS, allowing per-pod privilege levels within a namespace versus PSS's namespace-wide policy. This is critical for regulated environments where some pods need different security profiles.

---

## 7. GPU Support and Acceleration

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **GPU operator** | NVIDIA GPU Operator (via charm or manual) | NVIDIA GPU Operator (from OperatorHub) |
| **Node discovery** | Manual node labelling | NFD Operator (automatic) |
| **CUDA support** | CUDA-enabled notebook + serving images | CUDA-enabled workbench + serving images |
| **MIG** | Supported via GPU Operator | Supported via GPU Operator |
| **Time-slicing** | Supported | Supported |
| **GPU monitoring** | DCGM (via charm) | DCGM + OpenShift Prometheus |
| **GPU quota per tenant** | K8s ResourceQuota + pod limits | Kueue cluster queues + flavours |
| **DGX certification** | Yes (validated) | Supported (OpenShift on DGX) |

**Deployment comparison:**

- CKF: `juju deploy nvidia-gpu-operator --channel=stable` or standard K8s `kubectl apply`
- RHOAI: OperatorHub → install NFD Operator + GPU Operator → configure via `ClusterPolicy` CR

---

## 8. Model Serving Options

### Charmed Kubeflow Serving Stack

```
User Request → Istio Ingress → Knative (scale-to-zero) → KServe InferenceService
    ├── Predictor: Triton, TF Serving, PyTorch, ONNX, MLServer, HuggingFace, Custom
    ├── Transformer: Pre/post processing
    └── Explainer: Custom (not built-in)
```

**Supported runtimes:** Triton Inference Server, TensorFlow Serving, PyTorch Serve, ONNX Runtime, MLServer (sklearn, XGBoost, LightGBM), HuggingFace Inference Toolkit, **Seldon Core** (separate deployment), any custom container with KServe protocol.

### OpenShift AI Serving Stack

```
User Request → OpenShift Service Mesh (Istio) → OpenShift Serverless (Knative) → KServe InferenceService
    ├── Predictor: Triton, TF Serving, PyTorch, ONNX, vLLM, Caikit, MLServer, Custom
    ├── Transformer: Pre/post processing
    └── Explainer: TrustyAI (built-in)
```

**LLM serving patterns on RHOAI:**

| Pattern | Description | Best for |
|---|---|---|
| **KServe + vLLM** | vLLM as InferenceService with Serverless | Production LLM, high throughput |
| **KServe + Caikit** | Red Hat inference toolkit | OpenShift-native LLM serving |
| **RawDeployment** | KServe without Knative (no scale-to-zero) | Low-latency, always-on models |
| **Single-model serving** | One model per InferenceService | Large models, isolation |

**Key difference:** RHOAI has first-class vLLM support and TrustyAI explainability built in. CKF supports vLLM via custom configuration and has no built-in explainability. Both support KServe + Knative and RawDeployment modes.

---

## 9. IDE and User Experience

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Main UI** | Kubeflow Central Dashboard (profile-aware) | OpenShift AI Dashboard (project-aware) |
| **Notebook server** | JupyterLab (standard Kubeflow images) | JupyterLab (Red Hat curated images) |
| **Alternate IDEs** | RStudio, VS Code (code-server) | RStudio, VS Code |
| **Pipeline UI** | KFP UI (DAG visual, runs, experiments) | DSP UI + OpenShift DevConsole |
| **Model serving UI** | KServe list via Dashboard | KServe endpoints in AI Dashboard |
| **Hyperparameter UI** | Katib UI | Training runs via Dashboard |
| **Model Registry UI** | Manual (MLflow or custom) | Built-in Model Registry UI |
| **Explainability UI** | None | TrustyAI bias detection UI |
| **TensorBoard** | TensorBoard Web App (charm) | Custom deployment |
| **SSO integration** | Dex / OIDC | OpenShift OAuth (OIDC + LDAP + HTPasswd) |
| **Learning curve** | Moderate (Juju CLI additional) | Moderate (OpenShift concepts required) |

---

## 10. Ecosystem Integration

### Charmed Kubeflow Ecosystem

Canonical provides Juju-wrapped components that integrate declaratively via charm relations:

| Component | Charm | Purpose |
|---|---|---|
| Charmed MLflow | `mlflow` | Experiment tracking, model registry |
| Charmed Feast | `feast` | Feature store (production, July 2025) |
| Spark Operator | `spark-operator` | Large-scale data processing |
| Charmed Kafka | `kafka` | Event streaming |
| Charmed PostgreSQL | `postgresql` | Metadata, pipeline state |
| Charmed Redis | `redis` | Feast online store, caching |
| COS Lite | `cos-lite` | Monitoring (Prometheus, Grafana, Loki, Alertmanager) |

Integration pattern: `juju relate mlflow kfp-api` → declarative wiring via Juju relations.

### OpenShift AI Ecosystem

Deep integration with OpenShift's native capabilities:

| Component | Openshift equivalent | Purpose |
|---|---|---|
| OpenShift Pipelines | Tekton | CI/CD + Data Science Pipelines |
| OpenShift Serverless | Knative | Model serving autoscaling |
| OpenShift Service Mesh | Istio | Traffic mgmt, mTLS, auth |
| OpenShift GitOps | ArgoCD | Declarative ML infra deployment |
| OpenShift Monitoring | Prometheus + Alertmanager | Cluster + AI workload metrics |
| OpenShift Logging | Loki + Vector | Log aggregation |
| OpenShift SCC | Security Context Constraints | Pod security |
| RHEL AI | InstructLab | LLM fine-tuning with Granite models |
| Red Hat Quay | Container registry | Image storage |

Integration is automatic — OpenShift monitoring automatically discovers KServe service metrics; Service Mesh enforces mTLS across all ML components.

### Ecosystem Comparison

| Aspect | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Data processing** | Spark Operator (charm) | OpenShift + Spark (manual) |
| **Streaming** | Charmed Kafka | AMQ Streams (Kafka) |
| **CI/CD** | Not built-in (external) | OpenShift Pipelines (Tekton) + GitOps (ArgoCD) |
| **Monitoring** | COS Lite (deploy separately) | OpenShift Monitoring (built-in) |
| **MLflow** | Charmed MLflow (first-class) | Manual deployment |
| **LLM fine-tuning** | Not built-in | RHEL AI / InstructLab integration |
| **Vendor lock-in risk** | Low (open source, any K8s) | High (OpenShift-locked) |

---

## 11. Security and Governance

| Security Layer | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Authentication** | Dex + OIDC (LDAP, Google, GitHub, Okta) | OpenShift OAuth (OIDC, LDAP, HTPasswd, GitHub, GitLab) |
| **Ingress auth** | Istio AuthPolicy + OAuth2 proxy | OpenShift OAuth proxy + Route edge termination |
| **Authorization** | Kubeflow RBAC + Istio AuthorizationPolicy | OpenShift RBAC + RHOAI roles + Service Mesh AuthPolicy |
| **Pod security** | K8s PSS (Pod Security Standards: baseline/restricted) | OpenShift SCC (Security Context Constraints — more granular) |
| **mTLS** | Istio mTLS (strict mode supported) | OpenShift Service Mesh (Istio mTLS, strict mode) |
| **Network policy** | K8s NetworkPolicy | OpenShift NetworkPolicy (project isolation) |
| **Secrets** | K8s Secrets (Vault via external charm) | OpenShift Secrets + cert-manager (integrated) |
| **Audit** | K8s audit + Istio access logs + Juju debug logs | OpenShift audit (API, OAuth, Service Mesh) |
| **FIPS compliance** | Via Ubuntu Pro (FIPS modules) | OpenShift FIPS mode (validated cryptographic modules) |
| **Image security** | External (Harbor, Docker Registry) | ImageStreams + Quay (integrated) |

**Key differences for regulated environments:**

1. **SCC vs PSS:** OpenShift SCC allows per-pod security profiles within a namespace (mix privilege levels). K8s PSS applies namespace-wide — all pods in a namespace must meet the same level. SCC is strictly more flexible for regulated spaces where some pods require elevated capabilities.

2. **Audit:** OpenShift AI inherits OpenShift's comprehensive audit logging (API server, OAuth, operator, and mesh-level logs). Charmed Kubeflow audit is distributed across K8s audit log, Istio access logs, and Juju debug logs — more fragmented.

3. **FIPS:** OpenShift AI's FIPS mode is certified for the entire stack. Charmed Kubeflow on Ubuntu Pro can achieve FIPS compliance at the OS level, but the K8s + Kubeflow layer requires additional validation.

4. **CVE history (2025–26):** Both platforms have had notable CVEs. CKF: Istio permission CVE-2026-47237 (overly permissive `kubeflow-edit` role in Istio networking API). RHOAI: Kueue binding CVE (October 2025 — batch-user role bound to `system:authenticated`). Both actively patched by vendors.

---

## 12. Pricing and Licensing

| Component | Charmed Kubeflow | OpenShift AI |
|---|---|---|
| **Platform license** | Free (Apache 2.0) | OpenShift Platform Plus: ~$1,000–$2,500/2-core/year |
| **AI add-on** | Free | ~$500–$1,500/worker-node/year (est.) |
| **GPU node cost** | Same as any node | Same per-core rate |
| **Support** | Optional: Ubuntu Pro from $25/node/year | Included in subscription |
| **Minimum viable (3 nodes)** | $0 (self-support) | ~$12,000–$25,000/year |
| **Production (10 nodes, GPUs)** | $0–$5,000/year (with Ubuntu Pro) | ~$40,000–$80,000+/year |
| **Open source** | Fully (no proprietary components) | Core Kubeflow open; mgmt layer proprietary |

**Bottom line:** Charmed Kubeflow is free. OpenShift AI requires OpenShift subscription ($1K–$2.5K per 2-core/year) plus AI add-on ($500–$1.5K/worker-node/year). For a 10-node production cluster with GPUs, expect $40K–$80K+/year for RHOAI vs $0–$5K/year for CKF with support.

---

## 13. Use Cases

### Charmed Kubeflow Best For

| Use Case | Rationale |
|---|---|
| **Cost-sensitive AI/ML deployments** | No license fees. Self-supported or Ubuntu Pro ($25/node/year). |
| **Multi-cloud ML platforms** | Same Kubeflow across EKS, AKS, GKE, on-prem. Juju cross-controller mgmt. |
| **Small-to-medium ML teams (5–50 data scientists)** | Simpler architecture, less operational overhead than OpenShift. |
| **Research / academic environments** | Upstream-aligned, latest Kubeflow features, low cost. |
| **Ubuntu-focused organisations** | Natural fit for Ubuntu Server/Pro shops. |
| **Max flexibility / custom MLOps stacks** | Modular charms, swap components, avoid vendor lock-in. |
| **NVIDIA DGX environments** | Certified for DGX + DGX SuperPOD. |

### OpenShift AI Best For

| Use Case | Rationale |
|---|---|
| **Enterprise regulated AI/ML (banking, insurance, healthcare)** | SCC + FIPS + audit meet MAS TRM, PCI DSS, SOC 2, HIPAA. |
| **Security-first ML platforms** | Zero-trust, strict pod security, OAuth, compliance reporting. |
| **Large enterprises with OpenShift standardisation** | Natural extension for existing OpenShift infrastructure. |
| **Integrated DevOps + MLOps on single platform** | Pipelines, Serverless, Mesh, GitOps, Monitoring all built-in. |
| **Production ML at scale (>100 data scientists)** | Kueue scheduling, enterprise scalability, 24×7 support. |
| **Mainframe / IBM Z customers** | IBM Z and Power support (Technology Preview). |
| **Red Hat ecosystem shops** | RHEL, Ansible, ACM, Quay, AMQ — single vendor. |

---

## 14. Strengths and Weaknesses

### Charmed Kubeflow

**Strengths:**
- Fully open source (Apache 2.0, no proprietary components).
- Any Kubernetes — deploy on EKS, AKS, GKE, MicroK8s, or even OpenShift.
- Modular — deploy only what you need, swap components freely.
- No license fees. Support optional and predictable (Ubuntu Pro).
- Strong upstream alignment; Canonical contributes upstream.
- Charm ecosystem: MLflow, Feast, Spark, Kafka, PostgreSQL — all Juju-wired.
- NVIDIA DGX certified.
- Small footprint (single-node MicroK8s for dev/test).
- Multi-cloud portable via Juju abstraction.

**Weaknesses:**
- Smaller ecosystem — fewer pre-built integrations than OpenShift AI.
- Less mature enterprise support than Red Hat's.
- Juju learning curve — less widely known than K8s-native operators.
- Monitoring, logging, security require separate deployment (COS Lite).
- No built-in CI/CD — external integration required.
- No built-in explainability (TrustyAI equivalent).
- No FIPS compliance out of the box (requires Ubuntu Pro).
- Smaller community than upstream Kubeflow or OpenShift.

### OpenShift AI

**Strengths:**
- Complete integrated MLOps + DevOps platform — pipelines, serverless, mesh, GitOps, monitoring all built-in.
- OpenShift SCC — most granular pod security in Kubernetes ecosystem.
- Red Hat enterprise support (24×7, defined SLAs).
- Strong enterprise adoption in regulated industries.
- FIPS mode with validated cryptographic modules.
- Built-in Model Registry with API and UI.
- Built-in TrustyAI explainability and bias detection.
- Kueue-based advanced GPU scheduling.
- Large OperatorHub ecosystem of certified integrations.
- RHEL AI / InstructLab LLM fine-tuning integration.

**Weaknesses:**
- OpenShift-locked — cannot run on plain K8s (EKS, AKS, GKE).
- Expensive — $40K–$80K+/year for production clusters.
- Heavier footprint — 3+ control plane nodes, multiple prerequisite operators.
- Requires RHEL/OpenShift skills — steeper learning curve than standard K8s.
- Upstream lag — RHOAI releases trail ODH/Kubeflow releases.
- Deprecation churn — ModelMesh (deprecated 2.19), CodeFlare (deprecated 2.24), v1alpha1 Model Registry API (deprecated 2.24).
- Less modular — difficult to swap individual components.
- Cannot manage non-OpenShift K8s clusters.

---

## 15. Decision Matrix

### Quick Decision Table

| Scenario | Recommended | Rationale |
|---|---|---|
| Budget < $10K/year for ML platform | **Charmed Kubeflow** | OpenShift AI alone costs $12K+ before AI add-on |
| Already running OpenShift in production | **OpenShift AI** | Natural extension, integrated security, single vendor |
| Multi-cloud strategy (AWS + Azure + GCP native K8s) | **Charmed Kubeflow** | OpenShift AI requires OpenShift on each cloud |
| Bank / insurance / regulated industry | **OpenShift AI** | SCC + FIPS + audit are best-in-class for compliance |
| Small team (3–10 data scientists) | **Charmed Kubeflow** | Lower cost, simpler architecture, fewer operators |
| Large enterprise ML (>100 data scientists) | **OpenShift AI** | Scalability, enterprise support, Kueue scheduling |
| Ubuntu-focused / Canonical shop | **Charmed Kubeflow** | Natural alignment with Ubuntu Pro strategy |
| IBM Z / mainframe deployment | **OpenShift AI** | Only option with IBM Z support (Tech Preview) |
| Research / academic / non-profit | **Charmed Kubeflow** | Free, upstream-aligned, small-footprint capable |
| Need built-in explainability (AI governance) | **OpenShift AI** | TrustyAI built-in; CKF requires custom solution |
| Maximum customizability / component swap | **Charmed Kubeflow** | Modular charms allow component replacement |
| FIPS / FedRAMP / government compliance | **OpenShift AI** | OpenShift FIPS more mature than Ubuntu Pro FIPS for K8s |
| Mixed-platform team (K8s + OpenShift) | **Charmed Kubeflow** | Works on both; RHOAI is OpenShift-only |
| Already using RHEL / Ansible ecosystem | **OpenShift AI** | Unified vendor, common tooling, single support contract |

### Weighted Scoring Template

| # | Criterion | Weight (1–5) | CKF (1–5) | OAI (1–5) | Notes |
|---|---|---|---|---|---|
| 1 | Cost sensitivity | | | | Weight higher if budget-constrained |
| 2 | Existing K8s platform | | | | 5 if OpenShift is standard; 5 if vendor-agnostic |
| 3 | Integrated DevOps+MLOps | | | | Weight higher if single-platform desired |
| 4 | Security requirements | | | | Weight higher for regulated industries |
| 5 | Multi-cloud requirement | | | | Weight higher if multi-cloud is strategic |
| 6 | Team skills (Juju vs OpenShift) | | | | Weight based on existing expertise |
| 7 | Ecosystem alignment | | | | Canonical vs Red Hat ecosystem |
| 8 | Regulatory environment | | | | Weight 5 for banking/insurance |
| 9 | Scalability requirements | | | | Weight higher for large-scale ML |
| 10 | GPU workload complexity | | | | Weight higher for complex GPU needs |
| 11 | Vendor lock-in tolerance | | | | Weight higher if lock-in is a concern |
| 12 | Support requirements | | | | Weight based on SLA needs |
| | **Total** | **60 (max)** | **60 (max)** | **60 (max)** | **Higher = recommended** |

---

## 16. Conclusion

Charmed Kubeflow and Red Hat OpenShift AI represent two fundamentally different philosophies for deploying Kubeflow on Kubernetes:

**Charmed Kubeflow** is the **open-source-first, Kubernetes-agnostic option**. Its Juju charm architecture provides modularity, portability across any K8s distribution, and zero licensing cost. Ideal for organisations that value flexibility, avoid vendor lock-in, operate on a budget, or run multi-cloud K8s.

**Red Hat OpenShift AI** is the **enterprise integrated, security-first option**. Its deep integration with OpenShift's pipelines, serverless, service mesh, security, and monitoring provides a complete DevOps + MLOps platform out of the box. Ideal for regulated industries, large enterprises on OpenShift, and teams needing enterprise support and comprehensive security.

**Determining factors:**
1. **Existing infrastructure:** OpenShift or standard K8s?
2. **Budget:** Can you afford $12K–$80K+/year?
3. **Regulatory requirements:** Need SCC + FIPS out of the box?
4. **Multi-cloud strategy:** Need same platform across multiple K8s distributions?
5. **Team skills:** Juju or OpenShift proficiency?

There is no universal "better" platform — only the platform that better fits your organisation's specific constraints and requirements.

---

## 17. References

1. **Charmed Kubeflow Docs** — https://charmed-kubeflow.io/docs
2. **CKF System Architecture** — https://charmed-kubeflow.io/docs/system-architecture
3. **CKF General Installation** — https://charmed-kubeflow.io/docs/general-installation
4. **CKF Authorisation** — https://documentation.ubuntu.com/charmed-kubeflow/latest/explanation/security/authorisation/
5. **CKF GPU Guide** — https://documentation.ubuntu.com/charmed-kubeflow/latest/how-to/use/use-nvidia-gpus/
6. **Charmed Feast Launch (Canonical)** — https://ubuntu.com/blog/charmed-feast-feature-store-launch
7. **FAQ MLOps with Charmed Kubeflow** — https://canonical.com/blog/faq-mlops-charmed-kubeflow
8. **CKF vs Kubeflow** — https://canonical.com/blog/charmed-kubeflow-vs-kubeflow
9. **RHOAI Self-Managed Docs** — https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/
10. **OpenShift AI Architecture** — https://docs.redhat.com/en/documentation/red_hat_openshift_ai_cloud_service/1/html/installing_the_openshift_ai_cloud_service/architecture-of-openshift-ai_install
11. **RHOAI 2.25 Release Notes** — https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.25/
12. **RHOAI Model Serving** — https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.25/html/deploying_models/
13. **OpenShift Pricing** — https://www.redhat.com/en/technologies/cloud-computing/openshift/pricing
14. **Red Hat AI Subscription Guide** — https://www.redhat.com/en/resources/ai-subscription-guide-detail
15. **OpenShift SCC** — https://docs.openshift.com/container-platform/latest/authentication/managing-security-context-constraints.html
16. **Kubeflow Upstream Architecture** — https://www.kubeflow.org/docs/started/architecture/
17. **Open Data Hub** — https://opendatahub.io/docs
18. **Kubeflow vs OpenShift AI (Platform Engineer Perspective)** — https://www.linkedin.com/posts/tarun92057_kubeflow-openshiftai-kubernetes-activity-7413837827286683650-b10O
19. **Juju Documentation** — https://juju.is/docs
20. **NVIDIA GPU Operator MIG** — https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-operator-mig.html

---

> **Disclaimer:** Pricing figures are estimates based on publicly available information and may vary based on licensing agreements, geography, support tiers, and negotiating position. Verify current pricing with Canonical and Red Hat sales teams. OpenShift AI feature references are based on version 2.25 (self-managed). Charmed Kubeflow references are based on version 1.10/stable.
