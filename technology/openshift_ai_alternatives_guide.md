# Red Hat OpenShift AI and Its Alternatives — A Comprehensive Guide

> **A practical comparison of Red Hat OpenShift AI against the major MLOps/AI platforms for enterprise deployment, with a focus on on-premise and air-gapped scenarios relevant to banking.**

**Author:** Jack Liu Shurui  
**Context:** Cymbal Bank — Singapore  
**Repository:** https://github.com/jackliusr/research  
**Version:** 1.0  
**Last Updated:** August 2026

---

## Table of Contents

1. [What Is Red Hat OpenShift AI?](#1-what-is-red-hat-openshift-ai)
2. [Architecture and Components](#2-architecture-and-components)
3. [Licensing and Packaging](#3-licensing-and-packaging)
4. [Strengths](#4-strengths)
5. [Weaknesses](#5-weaknesses)
6. [Category 1 — Open-Source Self-Managed Stacks](#6-category-1--open-source-self-managed-stacks)
7. [Category 2 — Cloud-Managed ML Platforms](#7-category-2--cloud-managed-ml-platforms)
8. [Category 3 — Commercial Data Science / MLOps Platforms](#8-category-3--commercial-data-science--mlops-platforms)
9. [Category 4 — GenAI-Specific Platforms](#9-category-4--genai-specific-platforms)
10. [Selection Framework](#10-selection-framework)
11. [Banking Considerations](#11-banking-considerations)
12. [Implementation Approach](#12-implementation-approach)
13. [Common Pitfalls](#13-common-pitfalls)
14. [Comparison Summary Table](#14-comparison-summary-table)
15. [Conclusion](#15-conclusion)

---

## 1. What Is Red Hat OpenShift AI?

### 1.1 Definition

**Red Hat OpenShift AI** is Red Hat's enterprise MLOps and AI platform, built natively on **Red Hat OpenShift** (the Kubernetes distribution). It provides a single, consistent, hybrid-cloud platform covering the full AI/ML lifecycle — experimentation, model development, distributed training, model serving, monitoring, and governance — packaged as a supported commercial subscription with an open-source core.

Until May 2024 it was named **Red Hat OpenShift Data Science (RHODS)**; before that, the technology originated in the **Open Data Hub (ODH)** project. The product has evolved from "notebooks on OpenShift" into a full MLOps platform with model serving (KServe), a model registry (MLflow-based), distributed training (Ray/KubeRay), data science pipelines (Kubeflow Pipelines compatible), explainability (TrustyAI), and generative AI capabilities (fine-tuning with LoRA/PEFT, serving with vLLM, integration with Red Hat Enterprise Linux AI).

### 1.2 History: From Open Data Hub to OpenShift AI

| Era | Name | What changed |
|---|---|---|
| 2019–2020 | **Open Data Hub (ODH)** | Open-source project started by Red Hat with partners (IBM, Intel, etc.) as a "blueprint" for AI-as-a-service on OpenShift — Jupyter notebooks, Spark, Seldon/KServe, Prometheus, etc. |
| 2021 | **Red Hat OpenShift Data Science (RHODS)** | Red Hat's first supported product built on ODH; initial focus on managed Jupyter notebooks and model serving for data scientists. |
| 2023–2024 | RHODS 2.x | Added distributed training (CodeFlare/Ray), pipelines, KServe/ModelMesh serving, model registry. |
| May 2024 | **Red Hat OpenShift AI** | Rebranded at Red Hat Summit; positioned as the AI/ML platform layer for the hybrid cloud, aligned with the RHEL AI announcement. |
| 2024–2025 | OpenShift AI 2.x | Fine-tuning support (InstructLab, LoRA/PEFT), vLLM runtime for KServe, TrustyAI GA, RHEL AI integration. |
| 2025–2026 | **OpenShift AI 3.x** | CodeFlare Operator functionality folded into the KubeRay Operator; Kueue for cluster-wide job queueing; deeper GenAI and agent support; telco-scale deployments (e.g. Telenor AI Factory). |

### 1.3 What OpenShift AI Is Not

OpenShift AI is **not** a data platform (no built-in data warehouse, feature store, or data catalog — though it integrates with OpenShift Data Foundation, Kafka/Red Hat AMQ Streams, and external data platforms). It is **not** an AutoML product. It is **not** a SaaS-only offering — it runs on your OpenShift cluster, wherever that cluster runs (bare metal, vSphere, private cloud, or public cloud via ROSA/ARO). Understanding what it is *not* matters for the platform-selection discussion in this guide.

### 1.4 How OpenShift AI Maps to the ML Lifecycle

| Lifecycle stage | OpenShift AI component | Typical alternative |
|---|---|---|
| Experimentation | Workbenches (Jupyter/VS Code/RStudio) + MLflow tracking | SageMaker Studio, Dataiku, Domino |
| Feature engineering | Notebooks + pipelines + external data platforms | SageMaker Feature Store, Databricks |
| Distributed training | Ray/KubeRay, Training Operator (TFJob/PyTorchJob), Kueue | SageMaker training, Vertex Training |
| Model registry | MLflow-based registry (versions, stages) | SageMaker Model Registry, Azure ML registry |
| Serving | KServe (Knative), ModelMesh, vLLM/Triton runtimes | SageMaker endpoints, Vertex endpoints |
| Monitoring | TrustyAI (drift, bias) + Prometheus/Grafana | SageMaker Model Monitor, Azure ML monitoring |
| Governance | Registry lineage + OpenShift RBAC/audit + GitOps | Domino, SAS Model Manager, watsonx.governance |

The table above is the recurring frame for the rest of this guide: OpenShift AI covers every row *on your cluster*; each alternative covers some rows *in its own environment*.

### 1.5 Who It Is For

- **Data scientists** who need managed Jupyter/VS Code/RStudio workbenches, GPU access, and experiment tracking.
- **ML engineers** who need reproducible pipelines, a model registry, and production-grade serving with autoscaling.
- **Platform teams** in banks, telcos, and government who already run OpenShift and want AI/ML workloads on the same platform with the same security, networking, GitOps, and compliance posture as their applications.
- **Regulated enterprises** with on-premise or air-gapped requirements, where data sovereignty rules out public-cloud-managed ML platforms.

---

## 2. Architecture and Components

### 2.1 Operator-Based Architecture

OpenShift AI is installed and managed through the **Red Hat OpenShift AI Operator** (available from OperatorHub / Red Hat catalog). On the upstream side, the **ODH Operator** drives the same components. The operator reconciles a single custom resource, `DataScienceCluster`, which lets the platform team enable or disable each component independently (`managementState: Managed/Removed`). Everything is declarative and GitOps-friendly.

```yaml
apiVersion: datasciencecluster.opendatahub.io/v1
kind: DataScienceCluster
metadata:
  name: default-dsc
spec:
  components:
    dashboard:        { managementState: Managed }
    workbenches:      { managementState: Managed }
    kserve:           { managementState: Managed }
    modelmeshserving: { managementState: Managed }
    modelregistry:    { managementState: Managed }
    datasciencepipelines: { managementState: Managed }
    kueue:            { managementState: Managed }
    ray:              { managementState: Managed }
    trainingoperator: { managementState: Managed }
    trustyai:         { managementState: Managed }
```

This operator pattern is the core architectural difference from monolithic SaaS platforms: **the platform is assembled from independently deployable components on your own cluster**, which is what makes on-premise and air-gapped deployments feasible.

### 2.1.1 Component Map (OpenShift AI ↔ Upstream)

Every component ships as open source in ODH and is packaged/supported by Red Hat in OpenShift AI:

| Component | Upstream project | Purpose |
|---|---|---|
| Dashboard | ODH dashboard | Single entry point, spawner UI, component status |
| Workbenches | JupyterHub / code-server / RStudio images | Notebook & IDE environments per user |
| Model serving | **KServe** (CNCF) + Knative | Serverless inference, canary, inference graphs |
| Multi-model serving | **ModelMesh** (KServe cluster serving) | Many models on Triton-backed shared pods |
| Model registry | **MLflow** | Model versioning, stages, metadata, lineage |
| Distributed workloads | **Ray + KubeRay** (CodeFlare merged into KubeRay in 3.x) | Distributed training, Ray Serve, Ray Tune |
| Job scheduling | **Kueue** (Kubernetes sig-scheduling) | Cluster-wide quota, fair-share job queueing |
| Training operators | Kubeflow Training Operator | TFJob, PyTorchJob, MPIJob, XGBoostJob |
| Pipelines | **Kubeflow Pipelines / DSPO** | Declarative, reproducible ML pipelines |
| Explainability | **TrustyAI** | SHAP/LIME explanations, bias detection, drift |

### 2.1.2 Architecture at a Glance

```
┌──────────────────────────────────────────────────────────────┐
│                   Red Hat OpenShift AI                        │
├──────────────────────────────────────────────────────────────┤
│ Dashboard │ Workbenches │ Pipelines (DSPO) │ Model Registry  │
│ TrustyAI  │ Kueue      │ Training Operators │ Accelerator     │
│           │            │ (TFJob/PyTorchJob)│  Profiles        │
├──────────────────────────────────────────────────────────────┤
│ Serving: KServe (Knative) │ ModelMesh │ Ray / KubeRay        │
├──────────────────────────────────────────────────────────────┤
│ OpenShift platform: RH-SSO auth, Service Mesh, Prometheus,   │
│ GitOps (Argo CD), ACS, Quay, NVIDIA GPU Operator, ODF        │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 Workbenches (Notebooks and IDEs)

- **Jupyter** notebooks (default), **RStudio**, **VS Code / code-server**, plus **CUDA-enabled notebook images** for GPU work.
- Data scientists get a **self-service spawner UI**; each user gets an isolated project namespace (e.g. `rhods-notebooks`); images can be custom-built and added to the catalog.
- GPU resources are exposed through **Accelerator Profiles** — the platform team defines which accelerators (e.g. NVIDIA A100, L40S, MIG slices) are available to which namespaces, and the UI surfaces them as selectable options.

#### 2.2.1 Notebook Image Catalog (Examples)

| Image | Contents | Typical use |
|---|---|---|
| `minimal-ubi9-python-3.11` | Python, pip, conda | Lightweight general work |
| `cuda-ubi9-python-3.11` | Python + CUDA 12.x, cuDNN, NCCL | GPU training and inference dev |
| `pytorch-ubi9` / `tensorflow-ubi9` | Framework + CUDA + common libs | Deep learning (the most used images) |
| `rstudio-ubi9` | R, tidyverse, caret | Statistical / regulatory modeling |
| `code-server-ubi9` | VS Code in the browser | Engineering-style development |

Custom images are added by committing to the internal registry and registering them with the dashboard — the same pattern banks use for their approved-tooling lists.

### 2.3 Model Serving: KServe, Knative, ModelMesh

- **KServe** (standard for model serving on Kubernetes) provides **serverless inference** on **Knative** — scale-to-zero, request-based autoscaling, canary/blue-green rollouts, inference graphs.
- **ModelMesh** (now "cluster serving" within KServe) provides **multi-model serving** — many small models packed onto a few pods, with Triton as the inference runtime; ideal for many models with modest traffic.
- **Custom runtime support**: vLLM (for LLMs), NVIDIA Triton, and any KServe runtime; inference **graphs** chain pre/post-processing with the model.
- GPU sharing and autoscaling make KServe on OpenShift AI one of the more cost-effective inference platforms for high-volume serving.

#### 2.3.1 Serving an LLM with the vLLM Runtime

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: granite-8b
  namespace: model-serving
spec:
  predictor:
    model:
      modelFormat: { name: vllm }        # KServe custom runtime
      runtime: vllm                       # from the OpenShift AI runtime catalog
      storageUri: s3://bank-models/granite-8b
      resources:
        requests: { cpu: "4", memory: 32Gi, nvidia.com/gpu: 1 }
        limits:   { cpu: "8", memory: 64Gi, nvidia.com/gpu: 1 }
      args:
        - --tensor-parallel-size=1
        - --max-model-len=8192
```

The same pattern serves Triton, Hugging Face, and custom runtimes — and it is portable to plain Kubernetes (which matters if the bank ever evaluates Kubeflow).

### 2.4 Model Registry (MLflow-Based)

- The **model registry** component is built on **MLflow**, exposed through a Red Hat-managed CRD/API rather than the raw MLflow server.
- Provides model **versioning, stages** (e.g. staging → production), lineage from experiments, and metadata needed for governance and SR 11-7-style model inventory.
- Integrates with the dashboard so registered models can be deployed to KServe from the UI.

### 2.5 Distributed Workloads: Ray, KubeRay, Kueue

- Historically **CodeFlare** provided the Ray-on-OpenShift stack; from OpenShift AI 3.x the functionality (mTLS, network isolation, authentication) is **included in the KubeRay Operator**, which manages **RayCluster** resources.
- **Ray** gives Python-native distributed training (Ray Train), hyperparameter search (Ray Tune), and data processing (Ray Data) on the same cluster.
- **Kueue** provides cluster-wide **job queueing and fair-share scheduling** (admission of training jobs against quota), plus the **Training Operator** for native **TFJob / PyTorchJob / MPIJob** workloads.

#### 2.5.1 Distributed Training on Ray (KubeRay)

```yaml
apiVersion: ray.io/v1
kind: RayCluster
metadata:
  name: credit-risk-train
  namespace: ds-prod
spec:
  headGroupSpec:
    rayStartParams: { dashboard-host: 0.0.0.0 }
    template:
      spec:
        containers:
        - name: ray-head
          image: quay.io/modh/ray:2.35-py311-cu121
          resources: { requests: { cpu: "4", memory: 16Gi },
                       limits: { nvidia.com/gpu: 1 } }
  workerGroupSpecs:
  - groupName: gpu-workers
    replicas: 3
    template:
      spec:
        containers:
        - name: ray-worker
          image: quay.io/modh/ray:2.35-py311-cu121
          resources: { requests: { cpu: "8", memory: 32Gi },
                       limits: { nvidia.com/gpu: 1 } }
```

Kueue admits the RayCluster only when quota is available, and the Training Operator covers framework-native jobs (e.g. a `PyTorchJob` with N workers) for teams that prefer the native APIs.

### 2.6 Data Science Pipelines (DSPO)

- The **Data Science Pipelines Operator (DSPO)** runs **Kubeflow Pipelines**-compatible pipelines on OpenShift — pipeline definitions, run scheduling, artifact tracking.
- Pipelines are defined in Python (KFP SDK), versioned, and executed as Kubernetes-native steps; they are the mechanism for moving from notebook experiments to repeatable, auditable training workflows.

### 2.7 TrustyAI

- **TrustyAI** is the explainability and AI-fairness component: **SHAP/LIME-style explanations**, **bias detection** (group fairness metrics such as disparate impact), and **drift monitoring** for models in production.
- Available as a service that model-serving endpoints call, with results visualized in the dashboard — directly relevant to model-risk and fair-lending obligations in banking.

### 2.8 Fine-Tuning and Generative AI

- **Fine-tuning** of open-weight models using **LoRA / PEFT**, integrated with **InstructLab** tooling (llama.cpp-based `ilab` pipeline) for aligning models like Granite and Llama.
- **Serving** of fine-tuned and foundation models via **KServe with the vLLM runtime**, with OpenAI-compatible APIs.
- **RHEL AI integration**: models aligned and packaged in RHEL AI (a bootable RHEL image with optimized runtimes) can be imported into OpenShift AI for serving at scale.

### 2.9 Platform Integration

Because OpenShift AI runs on OpenShift, it inherits the platform services out of the box: **RH-SSO/Keycloak** authentication, **OpenShift Service Mesh** (Istio) for traffic control, **Prometheus/Grafana** monitoring, **Loki** logging, **Advanced Cluster Security (ACS)** for vulnerability and compliance scanning, **Quay** for image registry, and **Argo CD / OpenShift GitOps** for declarative delivery. This inheritance is the single biggest differentiator versus platforms that bring their own (incompatible) infrastructure.

---
## 3. Licensing and Packaging

### 3.1 Subscription Model

OpenShift AI is a **paid subscription** from Red Hat. Two delivery forms:

| Form | Runs on | Typical use |
|---|---|---|
| **OpenShift AI Self-Managed** | Your own OpenShift cluster (on-prem, vSphere, bare metal, or cloud) | On-premise / air-gapped banking, telco, government |
| **OpenShift AI Cloud Service** | Red Hat-managed OpenShift (ROSA on AWS, ARO on Azure) | Banks that want managed OpenShift in the cloud |

OpenShift AI is **included with Red Hat OpenShift Platform Plus** (the full platform bundle: OpenShift Container Platform + management/governance layers) and can also be purchased as a standalone entitlement. Pricing is typically **per core/vCPU** of the eligible cluster with an annual subscription; enterprise agreements are negotiated with Red Hat.

#### 3.1.1 What the Subscription Actually Buys

| Item | Included | You provide |
|---|---|---|
| OpenShift AI operators + components | ✅ (all components, supported) | OpenShift cluster + entitlements |
| Supported notebook/runtime images | ✅ (Red Hat-maintained) | Custom bank images (built on the catalog) |
| Security updates, CVE patches, FIPS | ✅ (per support window) | Upgrade testing/planning |
| Red Hat SLA & support | ✅ (severity-based) | Internal runbooks, on-call |
| GPU operator integration | ✅ (NVIDIA GPU Operator guidance) | GPU hardware + driver entitlements |
| Data platform connectors | ❌ (integration is yours) | Spark/Kafka/lakehouse connectivity |

### 3.2 The Open-Source Core: Open Data Hub

The open-source heart of OpenShift AI is the **Open Data Hub (ODH)** project — the operator framework plus the same component set (KServe, ModelMesh, Ray, KubeRay, pipelines, workbenches, TrustyAI) published under Apache-2.0. This means:

- You can build the equivalent platform yourself from ODH without paying Red Hat — the cost of OpenShift AI is the **support, testing, security hardening, and packaging** Red Hat provides on top.
- The components are also the same ones you would assemble for a fully self-managed Kubeflow-style stack — which is why the "build vs buy" question in Section 6 is really "how much assembly and support do you want to pay for."

### 3.3 License Gotchas (Read Before Budgeting)

- **OpenShift AI ≠ OpenShift.** You need an OpenShift platform subscription (OCP or Platform Plus) *and* the OpenShift AI entitlement. The platform cost is the baseline.
- **Red Hat does not license the GPUs separately**, but your GPU hardware, NVIDIA driver/operator entitlements, and any third-party software (e.g. Ray integrations, data platform connectors) are separate costs.
- **Support scope**: Red Hat supports the platform components; your custom code, custom runtimes, and third-party libraries (e.g. a hand-built vLLM image) are your responsibility — although Red Hat ships and maintains supported runtime images.
- **Version cadence**: OpenShift AI releases roughly twice a year and tracks OpenShift versions; upgrades are operator-driven but must be planned and tested (see Section 12).

---

## 4. Strengths

### 4.1 Native OpenShift Integration

OpenShift AI runs **on the same cluster as your applications** — the same operators, the same network policies, the same Service Mesh, the same GitOps pipelines, the same monitoring stack. In a bank that has standardized on OpenShift, AI/ML workloads inherit the existing operating model instead of introducing a parallel platform. Consistency of operations, security, and networking across apps and AI is the strongest architectural argument for OpenShift AI.

### 4.2 On-Premise and Air-Gapped by Design

Because it is Kubernetes-native and operator-driven, OpenShift AI supports **disconnected (air-gapped) installation**: images are mirrored into a private registry (Quay mirror / Red Hat Satellite), operators are staged from the Red Hat catalog, and the cluster never needs internet access. This is a decisive requirement for banks with data-residency mandates (MAS TRM, PDPA, GDPR) and for sovereign/government deployments.

### 4.3 Enterprise Support and Security

- Red Hat **SLA, security updates, and CVE patching** for the whole platform stack.
- **FIPS mode** support for US government/military; OpenShift itself is deployed in **FedRAMP-authorized** environments and OpenShift AI follows the same compliance trajectory.
- Component images are Red Hat-tested and maintained, with container signatures and SBOMs available — critical for software supply-chain audits.

### 4.4 GPU Support

Integration with the **NVIDIA GPU Operator** (drivers, device plugin, MIG partitioning, time-slicing), **Accelerator Profiles** to control which GPU resources users can request, and GPU-aware autoscaling for KServe inference. For banks running modest GPU pools, OpenShift AI manages them like any other cluster resource.

### 4.5 Trusted AI and Model Governance

**TrustyAI** delivers explainability, bias detection, and drift monitoring as a platform service; the **MLflow-based model registry** provides versioning, staging, and lineage. Together they cover much of the model-risk-management tooling that SR 11-7-style validation demands — something most open-source stacks leave entirely to the user.

### 4.6 Multi-Model Serving (ModelMesh)

ModelMesh packs many models onto shared Triton-backed pods with per-model autoscaling — the most cost-effective way to serve hundreds of small-to-medium models (e.g. per-counterparty or per-product models), a very common banking pattern.

### 4.7 The Red Hat Ecosystem

RHEL, **RHEL AI** (InstructLab + Granite models), Ansible Automation Platform, Quay, Advanced Cluster Security, RH-SSO/Keycloak, Satellite — plus the telco reference architectures (e.g. Telenor's AI Factory on OpenShift Platform Plus + OpenShift AI) and US government deployments. For enterprises already inside the Red Hat ecosystem, OpenShift AI is the path of least resistance.

---

## 5. Weaknesses

| Weakness | Detail | Mitigation |
|---|---|---|
| **Complexity** | OpenShift expertise required — operators, CRs, networking, service mesh, GPU node tuning are not turnkey | Budget platform-engineering time; use GitOps and Red Hat consulting |
| **Smaller ecosystem than cloud ML platforms** | Fewer prebuilt algorithms, connectors, and marketplace offerings than SageMaker/Vertex | Bring your own; integrate with data platforms you already run |
| **Limited built-in AutoML** | No Autopilot/Driverless-AI-style automated ML in the box | Use H2O, AutoGluon, or Ray Tune on top |
| **Less mature training features** | No built-in hyperparameter tuning at scale (Ray Tune available but DIY), distributed training tooling is younger than SageMaker's | Standardize on Ray/KubeRay + Kueue |
| **UI less polished** | Dashboard is functional, not Dataiku/SageMaker Studio-grade | Acceptable for platform teams; DS experience is notebook-based anyway |
| **Data platform integration requires work** | No native feature store (Feast operator exists upstream but is typically removed), limited data versioning, no data catalog | Integrate with your lakehouse/warehouse, Kafka, and object storage |
| **KServe/ModelMesh configuration complexity** | Knative gateways, inference graphs, runtimes, and traffic rules are powerful but fiddly | Use the dashboard and Red Hat examples; standardize runtime templates |

The honest summary: OpenShift AI trades **polish and breadth for control and consistency**. It is the right trade when you already live on OpenShift, must stay on-premise, and need a supported platform — it is the wrong trade if you want the fastest path to a fully managed, feature-rich cloud ML experience.

---
## 6. Category 1 — Open-Source Self-Managed Stacks

The default alternative to OpenShift AI is assembling your own MLOps stack from open-source components on your own Kubernetes cluster. Every component of OpenShift AI is itself open source (via ODH), so a self-managed stack is literally "OpenShift AI without Red Hat packaging." The question is whether your platform team can absorb the assembly, integration, and support burden.

### 6.1 Kubeflow

**Kubeflow** is the canonical open-source MLOps platform for Kubernetes. It is a CNCF project and the closest single-project equivalent to OpenShift AI's component set:

| Component | Purpose |
|---|---|
| **Kubeflow Pipelines** | Declarative ML pipelines (Argo Workflows under the hood), runs, artifact tracking |
| **Katib** | Hyperparameter tuning / neural architecture search (HPO) |
| **Training Operators** | **TFJob, PyTorchJob, XGBoostJob, MPIJob** — native distributed training CRDs |
| **Notebooks** | Managed Jupyter notebooks with multi-user isolation |
| **KServe** | Model serving (Kubeflow and KServe now evolve in lockstep) |
| **Metadata / Fairing** | Experiment metadata, model building/packaging (legacy) |
| **Central Dashboard** | Multi-user UI with RBAC (dex/istio-based auth) |

**Strengths:** standard and portable — runs on any CNCF Kubernetes (EKS, AKS, GKE, OpenShift, bare metal); large community; covers the full lifecycle with open components; no license cost.

**Weaknesses:** notoriously complex install/upgrade (each component versions independently — **version skew** is the #1 operational complaint); heavy resource footprint; dated UI; **no enterprise support** out of the box. Commercial distributions exist — **Canonical (Charmed Kubeflow), Arrikto (Enterprise Kubeflow), Nutanix, SUSE** — which restore the support layer at a price.

**Verdict:** the "DIY OpenShift AI." Best when you have strong Kubernetes platform engineering and want zero license cost; painful when you don't.

#### 6.1.1 A Kubeflow Pipeline in Python (KFP SDK)

```python
from kfp import dsl, compiler

@dsl.pipeline(name="credit-risk-train")
def pipeline(features_path: str, params: dict):
    prep = dsl.ContainerOp(
        name="featurize", image="registry.internal/bank/prep:1.0",
        arguments=[features_path])
    train = dsl.ContainerOp(
        name="train", image="registry.internal/bank/train:1.0",
        arguments=[prep.output, params])
    evaluate = dsl.ContainerOp(
        name="evaluate", image="registry.internal/bank/eval:1.0",
        arguments=[train.output])

compiler.Compiler().compile(pipeline, "credit_risk.yaml")
```

The same KFP SDK works on OpenShift AI's pipelines (DSPO) — which is why Kubeflow users migrate to OpenShift AI with minimal pipeline rewrites.

### 6.2 Ray (with KubeRay)

**Ray** is a distributed-compute engine for Python, increasingly the de facto substrate for training and serving workloads — and the same engine OpenShift AI itself uses for distributed workloads.

- **RayCluster / KubeRay** — Ray clusters on Kubernetes via operator.
- **Ray Train** — distributed training (PyTorch, TensorFlow, XGBoost, Hugging Face).
- **Ray Serve** — model serving with autoscaling and batching.
- **Ray Data** — distributed data loading/processing for training.
- **Ray Tune** — distributed hyperparameter search.
- **Ray AI Runtime (AIR)** — batteries-included libraries for end-to-end ML programs.

**Strengths:** unified compute for training + serving + data in one Python-native runtime; excellent autoscaling and fine-grained resource management (fractional GPUs, placement groups); a rich ecosystem (RLlib for reinforcement learning, Serve for LLM serving with vLLM integration).

**Weaknesses:** **not a full MLOps platform** — no pipeline UI, no model registry, no governance out of the box; you must combine it with MLflow (tracking), KServe (production serving), and Argo/Airflow (orchestration); a real learning curve; state management and observability are your problem.

**Verdict:** the compute layer of a modern self-managed stack — pair it with MLflow + KServe + a workflow engine.

#### 6.2.1 Ray Serve for Model Deployment

```python
from ray import serve
from transformers import pipeline

@serve.deployment(ray_actor_options={"num_gpus": 1})
class LLM:
    def __init__(self):
        self.pipe = pipeline("text-generation",
                             model="/models/granite-8b", device=0)
    async def __call__(self, request):
        return self.pipe(request.json["prompt"], max_new_tokens=512)

serve.run(LLM.bind())
```

Ray Serve handles autoscaling and request batching out of the box — a pragmatic middle ground between notebook experimentation and KServe production serving.

### 6.3 MLflow

**MLflow** is the de facto standard for **experiment tracking and model registry** (created at Databricks, now Linux Foundation). Components: Tracking (experiment/metrics/params), Model Registry (versioning, stages, approvals), Model Serving (single-model REST server), Projects (packaging).

**Strengths:** simple, widely adopted (works with any training framework and any cloud), multi-cloud, and its registry semantics (staging → production with annotations) map cleanly onto model-governance requirements.

**Weaknesses:** **not a platform** — no orchestration, no training infrastructure, no multi-user governance at enterprise scale, basic UI. The recommended pattern is a stack: **MLflow + Ray + KServe + Airflow/Argo** — which is, not coincidentally, what OpenShift AI bundles for you.

#### 6.3.1 MLflow Tracking + Registry in Practice

```python
import mlflow

mlflow.set_tracking_uri("http://mlflow.internal:5000")
with mlflow.start_run(run_name="lgbm-v3"):
    mlflow.log_params({"learning_rate": 0.05, "n_estimators": 1000})
    mlflow.log_metric("auc", 0.841)
    mlflow.log_metric("gini", 0.682)
    mlflow.log_artifact("confusion_matrix.png")
    mlflow.sklearn.log_model(model, "model",
        registered_model_name="credit_risk_lgbm")
```

The registry stage transition (Staging → Production) plus the run metadata is what auditors actually inspect — keep it populated from CI, not from notebooks.

### 6.4 KServe

**KServe** is the standard for model serving on Kubernetes (a CNCF project; its architecture was originally the serving piece of Kubeflow). Serving modes: **serverless** (Knative — scale-to-zero, autoscaling, canary), **cluster serving** (ModelMesh — multi-model packing on Triton), and **raw deployment**. Supports **custom runtimes** (vLLM, Triton, Hugging Face), **inference graphs**, and GPU serving with autoscaling.

**Strengths:** the portable serving standard — same runtime definitions work on OpenShift AI, Kubeflow, and vanilla K8s; model mesh for many-model workloads; GPU-aware autoscaling.

**Weaknesses:** serving only (no pipelines, training, or tracking); configuration complexity (Knative, gateways, runtimes, traffic policies) is real.

#### 6.4.1 A KServe InferenceService (Raw Deployment + Transformer)

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata: { name: xgb-credit, namespace: models }
spec:
  predictor:
    model:
      modelFormat: { name: sklearn }
      storageUri: s3://bank-models/credit-risk/xgb/42
  transformer:
    containers:
    - name: preprocess
      image: registry.internal/bank/feat-eng:2.1
```

The `transformer` runs feature engineering as part of the request path — the same pattern OpenShift AI uses to keep feature logic versioned with the model.

### 6.5 Argo Workflows + Argo CD

**Argo Workflows** is the workflow engine that powers Kubeflow Pipelines (and many ML orchestrations); **Argo CD** is the GitOps continuous-delivery tool.

**Strengths:** extremely flexible DAG orchestration; GitOps-native deployment (cluster state from Git — the audit-friendly pattern banks love); battle-tested in production Kubernetes.

**Weaknesses:** not ML-specific — you assemble the ML components (image building, data versioning, tracking, registry hooks) yourself; significant manual glue code.

#### 6.5.1 An Argo Workflow for a Training Run

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata: { generateName: ml-train- }
spec:
  entrypoint: train
  templates:
  - name: train
    steps:
    - - name: featurize
        template: featurize
      - name: train-model
        template: train-model
    - - name: register
        template: register
```

Argo gives you retries, timeouts, and DAG dependencies; what it does *not* give you is any ML semantics — every ML step (image, data version, tracking URI) is glue you write.

### 6.6 Self-Managed Stack Components — Comparison Table

| Capability | Kubeflow | Ray | MLflow | KServe | Argo |
|---|---|---|---|---|---|
| Notebooks / IDE | ✅ Jupyter (multi-user) | ❌ (via KubeRay + Jupyter) | ❌ | ❌ | ❌ |
| Pipelines / orchestration | ✅ KFP | ❌ (Ray Data/DAGs, DIY) | ❌ (Projects, minimal) | ❌ | ✅ Workflows |
| Distributed training | ✅ TFJob/PyTorchJob/MPIJob | ✅ Ray Train (best-in-class) | ❌ | ❌ | ❌ |
| Hyperparameter tuning | ✅ Katib | ✅ Ray Tune | ❌ (integrate Tune) | ❌ | ❌ |
| Model serving | ✅ KServe (integrated) | ✅ Ray Serve | ⚠️ single-model server | ✅ (its whole job) | ❌ |
| Experiment tracking | ⚠️ Metadata (legacy) | ❌ (needs MLflow) | ✅ (de facto standard) | ❌ | ❌ |
| Model registry | ❌ (needs MLflow) | ❌ | ✅ versions/stages | ❌ | ❌ |
| Governance / RBAC | ⚠️ multi-user via dex/Istio | ❌ | ⚠️ basic | ❌ | ✅ GitOps audit trail |
| GPU management | ✅ via device plugins | ✅ fine-grained (fractional) | ❌ | ✅ GPU autoscaling | ❌ |
| Enterprise support | ⚠️ via vendors (Canonical, Arrikto, Nutanix) | ⚠️ via Anyscale / vendors | ⚠️ via Databricks/vendors | ⚠️ community + vendors | ⚠️ community + vendors |

**The takeaway:** a production self-managed stack is a **combination** — typically Kubeflow (or just KFP) + Ray + MLflow + KServe + Argo CD, with you as the integration vendor. OpenShift AI is that same combination, pre-integrated, tested, and supported.

#### 6.6.1 Recommended Open-Source Combinations

| Combination | Profile | Typical users |
|---|---|---|
| **Full Kubeflow** (KFP + Katib + training ops + KServe + dashboard) | All-in-one on Kubernetes | Teams that want an integrated platform and accept install/upgrade complexity |
| **Ray + MLflow + KServe + Argo** | Compute-first, component best-of-breed | ML-engineer-heavy teams; the "modern" open-source stack |
| **MLflow + Airflow + serving on managed K8s** | Minimal viable MLOps | Small teams already on EKS/AKS/GKE |
| **ODH on OpenShift (no subscription)** | OpenShift-native DIY | OpenShift shops avoiding the license but keeping the architecture |

All four are legitimate; the first two are the ones banks actually run in production. The deciding variable is almost always **platform-engineering capacity**, not license cost.

---
## 7. Category 2 — Cloud-Managed ML Platforms

The three hyperscaler ML platforms are fully managed: you use the platform, the vendor runs the infrastructure. They are the strongest competitors to OpenShift AI when the bank has no on-premise mandate — and the ones with the hardest on-premise story.

### 7.1 Amazon SageMaker

**SageMaker** is AWS's fully managed ML platform, and the most comprehensive ML platform on the market:

- **Studio** — IDE for the whole lifecycle (notebooks, data prep, training, deployment).
- **Training** — built-in algorithms (XGBoost, factorization machines, etc.), **distributed training** (SageMaker Distributed, EFA), managed spot instances.
- **Hyperparameter tuning** — managed Bayesian/random search at scale.
- **Autopilot** — AutoML (auto feature engineering, model selection, tuning).
- **Inference** — real-time endpoints, **serverless inference**, batch transform; multi-model endpoints.
- **Pipelines, Feature Store, Model Registry** — all first-party.
- **SageMaker Canvas** — no-code ML for business analysts; **Ground Truth** — managed data labeling.

**Strengths:** most comprehensive feature set; deep AWS integration (S3, Glue, Redshift, Lambda, IAM); fully managed infrastructure; the broadest algorithm and framework library; mature MLOps.

**Weaknesses:** **AWS lock-in** (SageMaker has few portable abstractions); cost at scale (endpoint pricing, especially with always-on real-time endpoints); less control over the platform internals; **data egress costs** for moving data in/out — a real issue for a bank that might need to repatriate.

**Banking angle:** works for banks fully on AWS, but the **no-on-prem** limitation and egress costs often disqualify it for data-residency-sensitive workloads. AWS does offer **SageMaker on Outposts** for limited on-prem, but it is a fraction of the platform.

### 7.2 Azure Machine Learning (Azure ML)

**Azure ML** is Microsoft's enterprise ML platform in the Azure ecosystem:

- **Studio** — web UI; **notebooks**; **Designer** — drag-and-drop pipeline designer; **AutoML**.
- **Pipelines** — orchestrated ML workflows; **managed compute** (compute clusters/instances).
- **Model registry** — versioning; **endpoints** — online (real-time) and batch.
- **Responsible AI tools** — error analysis, fairness assessment, interpretability (error analysis dashboard, counterfactuals) — first-party and unusually strong.
- Integration with **Azure ML Studio, Data Factory, Synapse Analytics, Purview** (data governance).

**Strengths:** deep Microsoft ecosystem fit (Azure AD, Power BI, Synapse); enterprise-ready (RBAC, workspaces, private endpoints); the strongest **responsible-AI toolkit** of the big three; **hybrid story**: Azure Machine Learning can run **on-prem via Azure Arc** — the closest the hyperscalers get to OpenShift AI's model.

**Weaknesses:** Azure lock-in; UI complexity (the platform has many overlapping surfaces); cost management is a known pain point.

**Banking fit:** Azure ML is the most bank-friendly of the three hyperscaler platforms — Microsoft has banking-specific compliance programs, private endpoints/workspaces map cleanly onto network segmentation, and **Azure Arc** extends the platform into on-premise vSphere environments. Banks running Azure (or "Azure-first, on-prem eventually") should shortlist Azure ML alongside OpenShift AI on ARO; the deciding factor is usually whether the existing operating model is Azure-native or Kubernetes-native.

### 7.3 Google Vertex AI

**Vertex AI** is Google Cloud's unified ML platform, rebuilt as the GenAI-first platform:

- **AutoML** (tabular, image, text), **custom training** (Vertex Training), **pipelines** (KFP-based — note the same engine OpenShift AI pipelines use), **Feature Store**, **Model Registry**, **endpoint serving** (Vertex Prediction).
- **Vertex AI Search / Conversation** — managed RAG and conversational AI.
- **Model Garden** — curated foundation models (Gemini, Llama, Mistral, open weights).
- **Vertex AI Agent Builder** — agentic applications; **Agent Engine** for deploying agents.

**Strengths:** Google-scale infrastructure; **GenAI-first** (Gemini access, Model Garden, agent tooling); strong data integration with **BigQuery**; KFP-native pipelines make it the easiest cloud platform to migrate from a Kubeflow stack.

**Weaknesses:** GCP lock-in; **less common in banking** than AWS/Azure (a procurement and skills issue); pricing complexity (many SKUs, per-second billing confusion).

**Banking fit:** GCP is rarely the bank's primary cloud today — Vertex AI is usually evaluated in subsidiaries or greenfield GenAI projects (its Model Garden and Gemini access are genuinely best-in-class for foundation-model work). If the bank runs GCP at all, Vertex AI is the obvious choice; otherwise it stays a niche recommendation.

### 7.4 Cloud Platforms — Comparison Table

| Capability | SageMaker | Azure ML | Vertex AI |
|---|---|---|---|
| **AutoML** | ✅ Autopilot + Canvas | ✅ AutoML | ✅ AutoML |
| **Training** | ✅ built-in algos + distributed | ✅ managed compute + distributed | ✅ Vertex Training + distributed |
| **Serving** | ✅ real-time/serverless/batch | ✅ online + batch endpoints | ✅ endpoints + serverless |
| **Pipelines** | ✅ Pipelines | ✅ Pipelines + Designer | ✅ KFP-based |
| **Feature store** | ✅ | ⚠️ (via Azure Feature Store / Databricks) | ✅ Feature Store |
| **Model registry** | ✅ | ✅ | ✅ |
| **GenAI** | ✅ Bedrock adjacency + Canvas | ✅ Azure OpenAI + Foundry | ✅ Model Garden + Gemini (strongest) |
| **Integration** | AWS (S3, Redshift, IAM) | Azure (AD, Synapse, Purview) | GCP (BigQuery, Looker) |
| **On-prem** | ⚠️ Outposts only | ✅ Azure Arc (best hybrid) | ❌ |
| **Pricing** | complex, endpoint-heavy | complex, workspace + compute | complex, many SKUs |
| **Lock-in** | high | high | high |
| **Best for** | AWS-centric enterprises | Microsoft-centric enterprises | GenAI-first, GCP-centric enterprises |

**The pattern:** all three are excellent *if* the bank is already committed to that cloud and *if* there is no data-residency mandate. None of them is a first choice for an on-premise or air-gapped banking platform — for those, you are looking at OpenShift AI, Kubeflow, or the commercial platforms in the next section.

**Migration note:** moving off a cloud platform later is expensive and mostly manual (pipelines, feature stores, and registries have no portable standard; only KServe/KFP-based pieces transfer cleanly). Treat the cloud choice as a 5–10-year commitment, and if there is any chance of repatriation, prefer platforms whose building blocks (KFP, KServe, MLflow) have open equivalents — which is an argument for OpenShift AI or Kubeflow even in cloud-committed banks.

---

## 8. Category 3 — Commercial Data Science / MLOps Platforms

Independent software vendors selling full-stack data-science platforms. These are the realistic alternatives when the bank wants on-premise capability and a vendor SLA but does not want to standardize on OpenShift.

### 8.1 Databricks

**Databricks** is the lakehouse platform (data engineering + warehousing + ML in one), with the **Mosaic AI** stack (MLflow, feature store, AutoML, model serving, vector search, agent framework) on top.

**Strengths:** the **best data + AI integration** on the market — your ML platform is where your data already lives (Delta Lake); strong SQL and data engineering; managed serverless compute; Unity Catalog for governance across data and models.

**Weaknesses:** **cloud-only** — Databricks runs on AWS/Azure/GCP and has **no true on-premise offering** (a fundamental blocker for many banks); cost (DBU-based pricing adds up fast); **data gravity to the cloud** — the lakehouse model pushes data out of the data center.

**Verdict:** excellent for a bank that has already committed to cloud data (or runs a regulated cloud tenancy), irrelevant for on-premise mandates.

### 8.2 Dataiku

**Dataiku DSS** is a visual data-science platform aiming at the whole team — from analysts (visual, low-code) to expert data scientists (code-first):

- Visual pipelines (no-code feature engineering), code recipes (Python/R/SQL) side by side.
- **AutoML** (AutoML with model comparison), **MLOps** (deployment, monitoring, drift).
- Collaboration, project governance, **SSO/RBAC/audit** — enterprise-grade.
- **On-premise available** on Kubernetes (and air-gapped installs are supported).

**Strengths:** the best **citizen-data-science** experience of the commercial platforms; strong collaboration and governance; genuine on-premise deployment.

**Weaknesses:** **cost** — per-user licensing makes platform-wide rollout expensive (this is a per-seat product, not a cluster product); less deep for heavy code-first ML than a notebook+Ray stack; UI-centric workflows can frustrate pure engineers.

**Banking fit:** strong when the bank wants a **single governed environment for both analysts and data scientists** on-premise (Kubernetes-based deployment, air-gapped installs documented). The per-user pricing model favors rolling it out to a defined population (e.g. the risk analytics COE) rather than the whole bank.

### 8.3 Domino Data Lab

**Domino** is the enterprise MLOps platform with the strongest governance story:

- Model development environments (IDE, notebooks, **workspaces**), experiment tracking, model registry.
- Deployment as REST endpoints (Kubernetes-based).
- **Governance**: audit trails, RBAC, model approval workflows, reproducible runs (every run pinned to code + data + environment).
- **Nexus** — Domino's **on-premise/air-gapped** offering, explicitly designed for regulated industries (banking, pharma, government).
- GPU management and scheduling.

**Strengths:** **enterprise governance and reproducibility** (built for SR 11-7-style model risk); on-premise and air-gapped (Nexus) is a first-class product, not an afterthought; IT-approved environments (no rogue images).

**Weaknesses:** **expensive** (enterprise licensing); smaller community and fewer integrations than Databricks/SageMaker; less data-engineering capability (it is ML-centric, not a data platform).

**Banking fit:** the strongest pure-MLOps governance story for **on-premise/air-gapped** banks — Nexus is built for exactly the Crédit Agricole-type environment (data cannot leave the data center, every model run must be reproducible and auditable). If the bank's constraint is model risk governance rather than data engineering, Domino is the commercial platform to beat.

### 8.4 H2O AI Cloud

**H2O** offers **Driverless AI** (the strongest AutoML product in the enterprise space), H2O-3 (distributed ML algorithms), model ops, feature store, and an app store.

**Strengths:** best-in-class **AutoML** (automatic feature engineering, model tuning, explainability baked in); works **on-premise and air-gapped** (H2O is common in bank and government environments); speed to model.

**Weaknesses:** **AutoML-centric** — less platform breadth (pipelines, governance, serving are thinner than Domino/Dataiku); smaller ecosystem and community.

**Banking fit:** a good **complement, not a platform** — banks commonly run Driverless AI alongside OpenShift AI or Kubeflow to close the AutoML gap (Section 5 listed "limited built-in AutoML" as an OpenShift AI weakness; H2O is the standard answer). It installs cleanly on-premise and in air-gapped environments.

### 8.5 SAS Viya

**SAS Viya** is SAS's cloud-native analytics platform on Kubernetes: SAS Studio (code), visual analytics, machine learning, **Model Manager** (governance and deployment), and the SAS governance framework.

**Strengths:** the SAS ecosystem (a deep bench of regulated-industry analytics, especially credit risk and fraud); **very strong in banking** historically; on-premise capable; mature model governance (Model Manager with validation workflows).

**Weaknesses:** **legacy perception** and a closed architecture (SAS language, proprietary components); **cost** (enterprise licensing); a smaller modern-AI/GenAI story than competitors.

**Banking fit:** still deeply embedded in **credit risk, fraud, and regulatory reporting** functions — many banks run SAS in production for IFRS 9 / Basel models and cannot migrate quickly. Viya's Model Manager gives those teams a governed on-prem ML platform without leaving the SAS ecosystem. Choose it where SAS skills and models already dominate; choose OpenShift AI or Domino where the bank is modernizing.

### 8.6 Commercial Platforms — Comparison Table

| Capability | Databricks | Dataiku | Domino | H2O | SAS Viya |
|---|---|---|---|---|---|
| **Deployment** | AWS/Azure/GCP only | Cloud + on-prem (K8s) | Cloud + on-prem + **air-gapped (Nexus)** | Cloud + on-prem + air-gapped | On-prem + cloud |
| **AutoML** | ✅ (AutoML + AutoML for LLMs) | ✅ | ⚠️ (via integrations) | ✅✅ Driverless AI (best) | ✅ |
| **Code-first** | ✅ notebooks | ✅ code + visual | ✅ IDE/notebooks | ⚠️ Python/R + UI | ⚠️ SAS + Python/R |
| **Visual / low-code** | ⚠️ SQL + dashboards | ✅✅ (strength) | ⚠️ | ✅ | ✅ |
| **Governance / audit** | ✅ Unity Catalog | ✅ SSO/RBAC/audit | ✅✅ (strength) | ⚠️ | ✅✅ Model Manager |
| **Feature store** | ✅ (Mosaic) | ⚠️ | ❌ | ✅ | ⚠️ |
| **Model serving** | ✅ Mosaic serving | ✅ | ✅ REST endpoints | ✅ | ✅ Model Manager |
| **Pricing model** | DBU consumption | per-user license | enterprise license | per-user / capacity | enterprise license |
| **Best for** | Lakehouse + ML in cloud | Citizen DS + governance on-prem | Regulated on-prem ML ops | AutoML-first teams | SAS-shop regulated banks |

---
## 9. Category 4 — GenAI-Specific Platforms

Since 2024 the platform conversation has shifted toward generative AI: open-weight models, fine-tuning, RAG, and agents. This section covers the GenAI-specific options and how OpenShift AI fits in.

### 9.1 Red Hat Enterprise Linux AI (RHEL AI)

**RHEL AI** is Red Hat's foundation-model platform for the enterprise: a **bootable RHEL image** with an optimized runtime (vLLM, llama.cpp), the **InstructLab** toolchain for aligning open models, and the open-weight **Granite** model family (from IBM and Red Hat).

- **InstructLab (LAB method)**: a community-driven model-alignment approach — skill and knowledge "taxonomy" contributions generate synthetic training data, tuned with **LoRA** and merged into the model. Runs on a single server with GPUs; no Kubernetes required.
- **Granite models**: open-source (Apache-2.0), trained with enterprise use (code, language, time-series) in mind; sizes from ~3B to 34B — smaller than frontier models but deployable on modest hardware.
- **OpenShift AI integration**: models aligned in RHEL AI can be exported and served at scale on OpenShift AI via KServe + vLLM, and OpenShift AI's fine-tuning tooling (LoRA/PEFT) complements InstructLab.

**Strengths:** truly **open models** (no API dependency, no vendor lock-in on the model itself); on-premise and air-gapped friendly; enterprise-grade security posture (RHEL base, FIPS-capable); the **sovereign-AI** story Red Hat pushes for governments and regulated industries.

**Weaknesses:** **new and immature** (InstructLab is young; the ecosystem of tools, adapters, and skills is small); Granite models are **smaller** than frontier models (fine for many enterprise tasks, not for frontier-level reasoning); you still need OpenShift AI (or your own stack) for production serving and governance at scale.

#### 9.1.1 The InstructLab Alignment Workflow

1. **Curate taxonomy** — domain experts write skills/knowledge contributions as YAML (e.g. "how to compute Basel leverage ratio").
2. **Generate** — the LAB method creates synthetic training data from the taxonomy (teacher model + critique).
3. **Train** — LoRA fine-tuning with `ilab` on a GPU server (or Ray for scale on OpenShift AI).
4. **Evaluate** — run the bank's evaluation suite (accuracy, refusal behavior, hallucination checks).
5. **Merge & serve** — merge the LoRA adapter into the base model, export, and serve via KServe + vLLM on OpenShift AI.

The key property for banks: the **model stays on-premise and open** — no proprietary API, no data leaving the environment.

### 9.2 OpenShift AI for GenAI

OpenShift AI covers the GenAI lifecycle natively:

- **Fine-tuning**: LoRA/PEFT fine-tuning of open-weight models (Granite, Llama, Mistral) with InstructLab integration; distributed fine-tuning via Ray.
- **Serving**: KServe with the **vLLM runtime** — OpenAI-compatible endpoints, continuous batching, tensor parallelism across GPUs; Triton for non-LLM models.
- **RAG and guardrails**: vector databases (Milvus, pgvector, OpenSearch) run as ordinary workloads on the same cluster; **TrustyAI** provides explainability and bias detection as the guardrail layer.
- **Agents**: OpenShift 4.2x and OpenShift AI 3.x add agent-serving patterns (Telenor's AI Factory is the reference telco deployment).

#### 9.2.1 Fine-Tuning in Practice (LoRA + InstructLab Path)

```bash
# Align a Granite model with bank-specific skills (runs on OpenShift AI GPU node)
ilab model train \
  --data-dir /workspace/taxonomy \
  --model granite-8b --device cuda \
  --num-epochs 4 \
  --lora-r 16 --lora-alpha 32

# Export the merged model, then register and serve it via KServe vLLM runtime
ilab model export --model granite-8b-lora --output /models/granite-bank
```

PEFT/LoRA keeps the adapter small (MBs vs GBs), which makes model versioning and rollback practical under governance — a real advantage over full fine-tuning for regulated environments.

### 9.3 Local GenAI Stacks

For single-server or lightweight deployments, **vLLM**, **Ollama**, and **llama.cpp** are the pragmatic choices — easy to install, GPU-optimized, and ideal for prototyping or small-scale serving. See the companion guide **`ollama_xinference_localai_guide.md`** in this repository for a detailed comparison. These are runtime tools, not platforms: no governance, no multi-tenancy, no registry — which is why banks graduate from them to OpenShift AI or a commercial platform for production.

### 9.4 Enterprise GenAI Platforms

| Platform | What it is | Banking relevance |
|---|---|---|
| **IBM watsonx** | watsonx.ai (foundation models + ML), watsonx.data (lakehouse), watsonx.governance (model risk mgmt) | On-prem capable; governance-first; Granite family shared with RHEL AI |
| **NVIDIA AI Enterprise** | **NIM** microservices (containerized inference), DGX hardware, full-stack AI (NeMo, Triton, RAPIDS) | Runs on Kubernetes/OpenShift; the GPU-platform choice when NVIDIA stack is standard |
| **Hugging Face Enterprise Hub** | Private model/weights hosting, inference endpoints, dataset versioning | Complements any platform; the model-supply-chain layer (esp. for air-gap model mirroring) |
| **Dataiku LLM Mesh** | Governed access to many LLMs (public + private) behind one API with guardrails | Fits Dataiku shops; adds LLM governance |
| **Domino + GenAI** | GenAI model hosting on Domino's governed platform | Fits Domino shops needing governed GenAI |

### 9.5 GenAI Platforms — Comparison Table

| Capability | OpenShift AI + RHEL AI | IBM watsonx | NVIDIA AI Enterprise | Local (vLLM/Ollama/llama.cpp) |
|---|---|---|---|---|
| **Open-weight models** | ✅ (Granite + any) | ✅ (Granite) | ✅ (any via NIM) | ✅ |
| **Fine-tuning** | ✅ LoRA/PEFT/InstructLab | ✅ (tuning studio) | ✅ NeMo | ⚠️ (manual, llama.cpp LoRA) |
| **Production serving** | ✅ KServe + vLLM | ✅ | ✅ NIM/Triton | ⚠️ single-server scale |
| **Governance / guardrails** | ✅ TrustyAI + registry | ✅✅ watsonx.governance | ⚠️ (via platform) | ❌ |
| **On-prem / air-gapped** | ✅✅ | ✅✅ | ✅ | ✅ |
| **Sovereign AI** | ✅✅ (open + on-prem) | ✅✅ | ✅ | ✅ |
| **Maturity** | medium (3.x) | high (enterprise track record) | high (NVIDIA ecosystem) | high (as runtimes) |
| **Cost model** | subscription | enterprise license | per-GPU / NIM | free (your hardware) |

---

## 10. Selection Framework

### 10.1 Decision Factors

Work through these ten factors in order — the answer usually emerges before you reach the end:

1. **Deployment** — cloud, on-premise, or air-gapped? (Air-gapped immediately eliminates SageMaker/Vertex/Databricks and points to OpenShift AI, Kubeflow, Domino Nexus, Dataiku, H2O, SAS, RHEL AI.)
2. **Existing stack** — are you standardized on OpenShift, AWS, Azure, GCP, or bare metal/vSphere? (The strongest single predictor: banks on OpenShift should take OpenShift AI seriously; banks on Azure should look at Azure ML or OpenShift AI on ARO.)
3. **Team skills** — data scientists only, or do you have ML engineers and platform engineers? (Citizen-DS-heavy teams → Dataiku; platform-engineer-heavy teams → OpenShift AI/Kubeflow; DS-only teams → managed cloud.)
4. **MLOps maturity** — experimentation only, or production with monitoring/governance? (Experimentation → notebooks/MLflow is enough; production → you need serving, registry, drift — a platform.)
5. **Model types** — traditional ML (XGBoost/GLM), deep learning (PyTorch/TF), or GenAI? (Traditional ML → any platform; DL → GPU strategy matters; GenAI → vLLM/RHEL AI/watsonx capabilities.)
6. **Scale** — GPU needs; training-heavy vs inference-heavy? (Inference-heavy with many models → ModelMesh; training-heavy → Ray/KubeRay or SageMaker distributed.)
7. **Governance needs** — audit, RBAC, model registry, explainability? (This is where OpenShift AI, Domino, SAS, and Azure ML score; open-source stacks score last.)
8. **Budget** — license vs infrastructure vs people? (Open-source = infrastructure + people; commercial = licenses; cloud = consumption.)
9. **Compliance** — MAS TRM, BCBS 239, SR 11-7, GDPR/PDPA, FIPS? (See Section 11.)
10. **Support needs** — vendor SLA vs community? (Only Red Hat, hyperscalers, and the commercial vendors give you an SLA.)

#### 10.1.1 Factor → Platform Quick Matrix

| Factor | OpenShift AI | Kubeflow | Cloud (SM/Azure/Vertex) | Databricks | Dataiku/Domino/H2O/SAS |
|---|---|---|---|---|---|
| Air-gapped mandate | ✅✅ | ✅ (DIY) | ❌ | ❌ | ✅ (except Databricks) |
| Already on OpenShift | ✅✅ | ✅ | ⚠️ | ❌ | ⚠️ |
| Cloud-committed | ⚠️ (ROSA/ARO) | ✅ | ✅✅ | ✅✅ | ⚠️ |
| Governance-first | ✅ | ⚠️ | ✅ | ✅ | ✅✅ |
| Citizen data scientists | ❌ | ❌ | ⚠️ | ⚠️ | ✅✅ |
| GenAI on-prem | ✅✅ | ⚠️ | ❌ | ❌ | ⚠️ |
| Zero license budget | ❌ | ✅✅ | ⚠️ | ❌ | ❌ |
| Vendor SLA | ✅ | ❌ | ✅ | ✅ | ✅ |

Read it as a shortlist filter: eliminate rows where the bank's constraints are firm (air-gap, on-prem, budget), then evaluate the survivors on the factors that remain flexible.

### 10.2 Scenario-Based Recommendations

| # | Scenario | Recommendation | Rationale |
|---|---|---|---|
| 1 | Bank on OpenShift, on-prem/air-gapped, needs full MLOps with Red Hat support | **OpenShift AI** | Obvious choice — native integration, same ops model, supported air-gap |
| 2 | Bank on OpenShift, cost-sensitive / open-source mandate | **Kubeflow + KServe + MLflow + Ray** (ODH components, self-managed) | Zero license cost; you are the integrator |
| 3 | Bank on AWS | **SageMaker** (or Kubeflow on EKS) | Deep AWS integration; EKS option keeps portability |
| 4 | Bank on Azure | **Azure ML** (or OpenShift AI on **ARO**) | Azure-native, or Red Hat platform on Azure Red Hat OpenShift |
| 5 | Bank on GCP | **Vertex AI** | GCP-native; KFP pipelines ease Kubeflow migration |
| 6 | Bank needs on-prem data science with strong governance | **Domino (Nexus)**, **Dataiku (on-prem)**, or **SAS Viya** | Air-gapped + governance + vendor SLA without OpenShift dependency |
| 7 | Bank needs lakehouse + ML | **Databricks** | Best data+AI integration — only if cloud is acceptable |
| 8 | Bank needs AutoML-first | **H2O Driverless AI** or **Dataiku** | Best-in-class automated ML |
| 9 | Bank needs GenAI on-prem | **OpenShift AI + RHEL AI** (Granite/vLLM), **IBM watsonx**, or **NVIDIA AI Enterprise** | Open models, on-prem serving, sovereign AI |
| 10 | Multi-cloud strategy | **Kubeflow** (portable) or **OpenShift AI** (consistent across clouds) | Avoid per-cloud platform fragmentation |

**Cross-cutting rule:** in a bank, the platform decision is rarely made by data scientists — it is made by platform architecture, risk, and procurement. The winning platform is the one that fits the existing infrastructure and compliance posture, not the one with the best notebook experience.

---
## 11. Banking Considerations

### 11.1 Regulatory Landscape

| Regulation | What it demands | Platform implication |
|---|---|---|
| **MAS TRM** (Technology Risk Management, Singapore) | Platform security, access control, audit trails, change management, BCM | RBAC/SSO, audit logging, controlled change (GitOps), HA/DR design — all first-class on OpenShift AI; weakest in ad-hoc open-source stacks |
| **SR 11-7** (Model Risk Management, US) | Model validation, documentation, governance, inventory of models | Model registry with versioning/stages, reproducible runs, documentation hooks — OpenShift AI (MLflow registry), Domino, SAS Model Manager, Azure ML score well |
| **BCBS 239** (Risk Data Aggregation) | Data lineage for risk data, including model inputs | Feature/pipeline lineage: KFP artifacts + registry lineage on OpenShift AI; harder on notebook-only stacks |
| **GDPR / PDPA** | Data residency, purpose limitation, right to erasure | **On-premise/air-gapped** is the cleanest compliance answer — the core reason banks evaluate OpenShift AI, Domino Nexus, or Dataiku on-prem |
| **FIPS** (US) | Approved cryptographic modules | OpenShift/FIPS mode + RHEL base; required for US federal/gov-adjacent banking workloads |

### 11.2 Air-Gapped Deployment (No Internet Access)

The defining constraint for many banking AI platforms. What has to be staged before the cluster goes dark:

- **Container images** — pre-stage all images into an internal mirror registry: **Quay mirror**, **Red Hat Satellite**, or the OpenShift **disconnected catalog** (oc-mirror). This includes operator images, notebook images, runtime images (vLLM, Triton), and any custom images built by the bank.
- **Model registry mirroring** — foundation models and datasets must be **pre-downloaded** before air-gap: `huggingface_hub` supports **offline mode** (HF_HUB_OFFLINE=1) with a local cache; stage weights onto internal storage/object store and point serving runtimes at the local path. Plan this as a repeatable, audited process — model provenance becomes part of the risk record.
- **Vulnerability scanning offline** — Trivy/Grype need **local vulnerability databases** (air-gapped DB updates); OpenShift **Advanced Cluster Security** can run fully disconnected with periodic offline database syncs.
- **SBOM management** — every image's SBOM (CycloneDX/SPDX) must be captured at build time and archived for supply-chain audits.
- **Software supply chain** — **signed images (cosign/signature verification)** enforced at admission; Red Hat images ship signed, and your internal CI must sign everything else. In an air-gapped bank, the mirror registry is the trust anchor — control it like a production system.

#### 11.2.1 Air-Gap Staging Checklist

| Item | Mechanism | Notes |
|---|---|---|
| OpenShift + operator images | `oc-mirror` / disconnected catalog | Stage the *entire* dependency graph before install |
| Platform & notebook images | Quay mirror / Red Hat Satellite | Continuous sync while still connected |
| Foundation models | `huggingface_hub` offline mode + pre-downloaded weights | Verify checksums; record provenance |
| PyPI / conda packages | Internal devpi/Nexus mirror | Data-science dependencies must be pre-approved |
| Vulnerability DBs | Trivy/Grype offline DBs; ACS offline syncs | Plan a periodic (manual) update process |
| SBOMs | Capture at build time (CycloneDX/SPDX) | Archive with the model's risk record |
| Signing keys | cosign verification keys distributed to clusters | Rotate and audit |

```bash
# Stage operators and images for a disconnected OpenShift AI install
oc mirror --config=imageset-config.yaml   # produces a tarball
# Import into the internal registry on the dark side
oc image mirror --from-dir=./mirror \
  --to-dir=./mirror --registry-config=/etc/pull-secret.json
```

Rule of thumb: **anything a data scientist can `pip install` or `huggingface-cli download` is something you must pre-stage** — the air-gap fails the day after go-live if this is improvised.

### 11.3 Security

- **RBAC / SSO** — OpenShift AI authenticates via OpenShift OAuth; integrate **RH-SSO/Keycloak or external IdP (LDAP/AD, Entra ID, Okta)**; per-namespace RBAC for data scientists vs ML engineers vs admins. Role-based access to GPU resources via Accelerator Profiles.
- **Network policies** — restrict model-serving ingress to approved consumers (API gateways), isolate training from production namespaces, deny egress except to the mirror registry and approved endpoints.
- **Encryption** — TLS everywhere (OpenShift default ingress + Service Mesh mTLS), **disk encryption** (LUKS/encrypted PVCs, ODF encryption), and secrets in **OpenShift Secrets / HashiCorp Vault** (external secrets operator) rather than plain env vars.
- **GPU isolation** — **MIG partitioning** (A100/H100) for hard isolation, **time-slicing** for shared best-effort workloads; pin Accelerator Profiles to approved MIG slices.
- **Audit logging** — OpenShift audit logs + platform audit (who spawned which workbench, who registered/deployed which model) forwarded to the bank SIEM.
- **Secrets management** — Vault integration for model credentials, registry tokens, and data-platform keys.

#### 11.3.1 Accelerator Profile Example (MIG Slice)

```yaml
apiVersion: dashboard.opendatahub.io/v1
kind: AcceleratorProfile
metadata:
  name: a100-mig-2g
spec:
  displayName: NVIDIA A100 MIG 2g.10gb
  identifier: nvidia.com/gpu
  tolerations:
  - key: nvidia.com/gpu
    operator: Exists
    effect: NoSchedule
  resources:
    limits:
      nvidia.com/gpu: "1"
```

Publishing a small set of named profiles (e.g. `a100-full`, `a100-mig-2g`, `l40s-full`) instead of raw GPU quotas is what makes GPU access auditable and chargeable per team.

### 11.4 Model Governance

| Concern | Tooling on OpenShift AI | Notes |
|---|---|---|
| Registry / versioning | **MLflow-based model registry** (versions, stages, metadata) | Every promoted model has an immutable record |
| Approval workflows | Registry stage transitions + external review (ServiceNow/Jira hooks) | SR 11-7 wants human sign-off before production |
| Monitoring | KServe metrics + **drift monitoring** (TrustyAI, Evidently/Arize can bolt on) | Data drift + performance degradation alerts |
| Explainability | **TrustyAI** (SHAP/LIME) | Required for fair-lending / model-risk reviews |
| Model risk documentation | Reproducible pipeline runs (KFP artifacts) + registry lineage | Feed the SR 11-7 documentation pack |
| Bias monitoring | **TrustyAI bias detection** (group fairness metrics) | Increasingly expected by regulators |

### 11.5 GPU Strategy

- **NVIDIA GPU Operator** — automates driver/DCGM/device-plugin deployment; the standard on OpenShift AI.
- **Partitioning**: MIG (hard isolation, A100/H100) vs **time-slicing** (shared) — banks typically give training jobs MIG slices and serving workloads time-sliced GPUs with autoscaling.
- **Serving autoscaling**: KServe + Knative scales replicas on requests/GPU utilization — the biggest cost lever for inference-heavy banks (scale-to-zero for non-critical endpoints).
- **Training jobs**: Ray (KubeRay) for Python-native distributed training; TFJob/PyTorchJob via the Training Operator for framework-native jobs; **Kueue** for quota and fair-share.
- **Monitoring**: **DCGM exporter → Prometheus → Grafana** for GPU utilization, temperature, memory; alert on idle GPU pools (the classic waste).
- **Capacity planning**: the arithmetic is `GPU memory ≥ model size × overhead` for inference (e.g. a 70B model at FP16 needs ~140GB → 2×H100 or quantization) and `concurrency × batch size` for throughput; for training, `model size × optimizer overhead (≈16–20× params bytes for AdamW mixed-precision) × batch` per rank. Size the pool for the **peak training job + steady-state inference**, not the sum of every team's request.

#### 11.5.1 Inference GPU Sizing Reference

| Model (FP16) | VRAM needed | Typical allocation |
|---|---|---|
| 3–4B (Granite 3/4, Llama 3.2 3B) | ~8–12 GB | 1×L40S / A10G, or MIG slice |
| 7–8B (Granite 8B, Llama 3.1 8B) | ~16–24 GB | 1×L40S / A100 40GB |
| 13–14B | ~28–40 GB | 1×A100 40GB / 2×L40S |
| 70B (Llama 3.1 70B) | ~140–160 GB | 2×H100 / 4×A100 80GB |
| 405B | ~800+ GB | Cluster + quantization (AWQ/GPTQ) mandatory |

Quantization (FP8/INT8/AWQ) typically cuts VRAM 2–4× at a small quality cost — for inference-heavy banks this is often the difference between "we can't afford it" and "fits on the existing pool."

### 11.6 Cost Considerations

| Model | Cost profile | When it wins |
|---|---|---|
| **OpenShift AI subscription** | Platform (OCP/Platform Plus) + AI entitlement, per-core; GPUs + storage + people on top | Already on OpenShift; on-prem mandate; predictable multi-year cost |
| **Self-managed (Kubeflow)** | Zero license; infrastructure + **platform-engineering people** (often 2–4 FTE) | Cost-sensitive, strong platform team |
| **Cloud-managed (SageMaker/Azure ML/Vertex)** | Pay-as-you-go consumption; endpoint hours, storage, egress | Cloud-committed banks; variable workloads |
| **Commercial (Dataiku/Domino/SAS/H2O)** | Per-user or enterprise licenses (often 5–7 figures/year) | Governance-heavy on-prem needs without OpenShift |
| **GPU hardware** | **The dominant line item** — NVIDIA A100 (~$10–15k), H100 (~$25–40k), L40S (~$3–5k) per unit; plus power/cooling | Plan capacity before buying; shared pools beat per-team GPUs |

TCO = infrastructure + licenses + **people** (platform engineering, integration, governance) + support + migration. In practice, the people and integration terms dominate the license line — which is why a supported platform (OpenShift AI) is often cheaper than a "free" open-source stack in a bank that lacks Kubernetes expertise.

---

## 12. Implementation Approach

| Step | Scope | Typical duration | Deliverables |
|---|---|---|---|
| **1. Assess** | Current state (clusters, data platforms, teams), requirements (compliance, GPU, integrations), constraints (air-gap, budget, skills) | 2–4 weeks | Decision memo, target architecture, ROI/TCO model |
| **2. PoC** | Small cluster (3–5 nodes, 2–4 GPUs): notebook workbench + a training run + KServe endpoint end-to-end | 2–4 weeks | Working PoC, platform runbook, feasibility sign-off |
| **3. Pilot** | **One use case end-to-end**: data → training → model registry → serving → monitoring, with the real data platform and the real security controls | 4–8 weeks | Production-shaped pipeline, registry entries, drift alerts, SR 11-7 documentation pack |
| **4. Productionize** | Multi-tenant (RBAC, quotas, Accelerator Profiles), GPU pool management (Kueue), CI/CD pipelines (GitOps), governance (approvals, audit), monitoring/SLOs, DR | 2–4 months | Production platform, SLAs, runbooks, compliance evidence |
| **5. Scale** | Multi-team onboarding, multi-cluster (DR/second region), GenAI workloads (fine-tuning + vLLM serving), cost governance | Ongoing | Platform as an internal product with a roadmap |

**Ordering principle:** never start with infrastructure scale — start with **one model in production**, then widen. The PoC validates the platform; the pilot validates the operating model; everything after is repetition.

#### 12.1 "Done When" Checklist (Pilot)

- [ ] A data-science team trained a real model on the platform, using the real data path.
- [ ] The model is registered with metadata (owner, version, training run, data snapshot).
- [ ] An endpoint serves it with autoscaling; latency/throughput SLOs are defined.
- [ ] Drift monitoring and alerting are active and routed to the model owner.
- [ ] RBAC is enforced (DS cannot deploy to production without the ML-engineer role).
- [ ] The SR 11-7 documentation pack (development, validation, implementation, monitoring) is populated.
- [ ] A rollback drill succeeded (previous registry version redeployed in < 1 hour).

If the pilot cannot tick all seven boxes, the platform is not production-ready — extend the pilot rather than declaring victory.

---

## 13. Common Pitfalls

1. **Underestimating GPU requirements** — buying per-team instead of pooled; ignoring the 16–20× optimizer memory multiplier for training; no idle-GPU monitoring.
2. **Ignoring data platform integration** — the ML platform needs data access (object storage, warehouse, Kafka); planning the platform without the data path is the most common failure.
3. **Skipping governance until late** — RBAC, registry, audit, and approval workflows retrofitted after models are in production is painful and regulator-visible. Build them in the pilot.
4. **Treating OpenShift AI as turnkey** — it is a platform *framework*: operators install quickly, but Accelerator Profiles, networking, GPU nodes, image catalogs, and integrations need real configuration.
5. **Not planning for air-gap** — model downloads (Hugging Face), image updates, and vulnerability-DB syncs all break silently in disconnected environments. Stage everything before going dark.
6. **Forgetting monitoring and drift** — models degrade; without drift detection and performance monitoring, the registry is just a catalog.
7. **License misunderstanding** — OpenShift AI is *not* included with plain OpenShift; budget for the AI entitlement on top of the platform subscription (it ships with Platform Plus).
8. **Underestimating MLOps maturity** — the hard part is people and process (model validation, handover, documentation), not software; a platform cannot fix an immature organization.

**The pattern behind all eight:** every pitfall is a *planning* failure, not a platform failure — GPU math, data access, governance, air-gap staging, licensing, and operating-model maturity are all knowable *before* procurement. Run the Section 10 decision framework and the Section 12 assessment step with the platform team and risk involved, and most of these disappear. The classic sequence to avoid is: choose the platform first, then discover the constraints that should have chosen it.

---

## 14. Comparison Summary Table

| Platform | Deployment | Open-source | On-prem | Air-gapped | GenAI | AutoML | Governance | Support | Pricing | Best for |
|---|---|---|---|---|---|---|---|---|---|---|
| **OpenShift AI** | Hybrid (any OCP) | Core (ODH) | ✅✅ | ✅✅ | ✅✅ (RHEL AI/vLLM) | ⚠️ (DIY) | ✅ | ✅ Red Hat SLA | Per-core subscription | OpenShift banks, on-prem/air-gap, GenAI |
| **Kubeflow** | Any K8s | ✅ | ✅ | ✅ | ⚠️ | ✅ Katib | ⚠️ | ⚠️ vendors | Free (+people) | Open-source mandate, K8s-savvy teams |
| **SageMaker** | AWS only | ❌ | ⚠️ Outposts | ❌ | ✅ (Bedrock-adjacent) | ✅✅ Autopilot | ✅ | ✅ AWS SLA | Consumption | AWS-committed banks |
| **Azure ML** | Azure + Arc | ❌ | ✅ (Arc) | ⚠️ (limited) | ✅ (Azure OpenAI) | ✅ | ✅✅ + Responsible AI | ✅ Microsoft SLA | Consumption | Microsoft shops, hybrid |
| **Vertex AI** | GCP only | ❌ | ❌ | ❌ | ✅✅ (Gemini/Model Garden) | ✅ | ✅ | ✅ Google SLA | Consumption | GCP + GenAI-first |
| **Databricks** | Cloud (3 clouds) | Core (MLflow/Delta) | ❌ | ❌ | ✅ (Mosaic AI) | ✅ | ✅ Unity Catalog | ✅ | DBU consumption | Lakehouse + ML in cloud |
| **Dataiku** | Cloud + on-prem K8s | ❌ | ✅ | ✅ | ✅ (LLM Mesh) | ✅✅ | ✅ | ✅ | Per-user license | Citizen DS + governance |
| **Domino** | Cloud + on-prem | ❌ | ✅✅ (Nexus) | ✅✅ | ✅ (GenAI hosting) | ⚠️ | ✅✅ | ✅ | Enterprise license | Regulated on-prem ML ops |
| **H2O AI Cloud** | Cloud + on-prem | Core (H2O-3) | ✅ | ✅ | ⚠️ | ✅✅ Driverless AI | ⚠️ | ✅ | Per-user/capacity | AutoML-first teams |
| **SAS Viya** | On-prem + cloud | ❌ | ✅ | ✅ | ⚠️ | ✅ | ✅✅ | ✅ | Enterprise license | SAS-shop regulated banks |

---

## 15. Conclusion

For a bank on OpenShift with on-premise or air-gapped requirements — the profile this guide was written for — **Red Hat OpenShift AI is the default answer**: it is the only platform that combines a supported enterprise SLA, native integration with the platform the bank already runs, first-class disconnected deployment, GPU management, model governance, and a credible GenAI path (RHEL AI, Granite, vLLM, TrustyAI). Its weaknesses — polish, built-in AutoML, training breadth — are addressable with tooling (Ray Tune, H2O) and process.

The alternatives remain valid in their lanes: **Kubeflow** when cost or open-source mandate dominates and the platform team is strong; **SageMaker/Azure ML/Vertex AI** when the bank is cloud-committed and residency permits; **Domino/Dataiku/SAS** when governance-heavy on-prem capability is needed without OpenShift; **Databricks** when the data platform itself is the driver; **H2O** when AutoML speed matters most; and **RHEL AI / watsonx / NVIDIA AI Enterprise** for GenAI-specific sovereignty plays.

The decision framework in Section 10 exists because there is no universal winner — the right platform is the one that fits the bank's infrastructure, compliance posture, and operating model. In most cases that analysis lands on the same conclusion: **the platform that runs where the bank's data is allowed to be.**
