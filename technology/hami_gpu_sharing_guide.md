# HAMi and Kubernetes GPU Sharing: Virtualization, Fractional Allocation, and the NVIDIA-Native Alternatives Compared

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Systems / Cloud-Native & AI Infrastructure — Kubernetes GPU Sharing, GPU Virtualization, Device Scheduling, MIG/MPS, Banking Technology  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** September 2026

---
> A head-to-head deep-dive into Kubernetes GPU sharing and virtualization, centered on **HAMi** — the CNCF-incubating "Heterogeneous AI Computing Virtualization Middleware" — and comparing it against the NVIDIA-native sharing mechanisms (MIG, time-slicing, MPS), the NVIDIA device-plugin/GPU Operator stack, the vCUDA-style interception projects, and the scheduler-level sharing frameworks. The guide establishes HAMi's identity from primary sources first (the Mitigant discipline), then compares mechanisms on isolation, memory partitioning, scheduling model, overhead, ecosystem fit, and governance — closing with a decision framework and a worked example for the Cymbal Bank inference platform.

**Audience**: Solution architects, ML platform engineers, and infrastructure leads running AI workloads on Kubernetes in banking or enterprise settings. Assumes working knowledge of Kubernetes (pods, resources, device plugins) and a basic mental model of NVIDIA GPUs; no CUDA programming required. GPU *performance engineering* (kernels, profiling) lives in the sibling guide [gpu_optimization_guide.md](gpu_optimization_guide.md) — this guide covers the *sharing and virtualization* layer above it.

**What this guide deliberately will not do**: it will not teach CUDA optimization, kernel tuning, or model quantization (see [gpu_optimization_guide.md](gpu_optimization_guide.md) and the LLM sibling guides); it will not re-derive Kubernetes cluster administration (see the cluster siblings); and it will not endorse any mechanism as universally "best" — every recommendation here is conditional on the workload mix and the tenancy/regulatory context, which is why §11 is structured as questions rather than a ranking. What it *will* do is give you a verified map of the sharing landscape (who does what, at which layer, with which isolation guarantees), the governance reality behind each option, and a repeatable decision procedure. ⚠️ Where vendor documentation, project self-reports, or fast-moving cloud offerings could not be fully verified, they are flagged inline and collected in §13.

---

## Table of Contents

1. [Scope and Reading Path](#1-scope-and-reading-path)
2. [HAMi Identity: What HAMi Actually Is](#2-hami-identity-what-hami-actually-is)
3. [HAMi Architecture: From Request to Isolation](#3-hami-architecture-from-request-to-isolation)
4. [HAMi Capabilities: Resource Model, Scheduling and MIG](#4-hami-capabilities-resource-model-scheduling-and-mig)
5. [HAMi Maturity, Adoption and Governance](#5-hami-maturity-adoption-and-governance)
6. [NVIDIA-Native Sharing: MIG, Time-Slicing and MPS](#6-nvidia-native-sharing-mig-time-slicing-and-mps)
7. [The NVIDIA Device-Plugin Landscape: k8s-device-plugin and GPU Operator](#7-the-nvidia-device-plugin-landscape-k8s-device-plugin-and-gpu-operator)
8. [vCUDA-Style and Other Sharing Frameworks](#8-vcuda-style-and-other-sharing-frameworks)
9. [Cloud-Vendor GPU Sharing (Condensed)](#9-cloud-vendor-gpu-sharing-condensed)
10. [Head-to-Head: HAMi vs the Alternatives](#10-head-to-head-hami-vs-the-alternatives)
11. [Decision Guidance: When to Use What](#11-decision-guidance-when-to-use-what)
12. [Worked Example: The Cymbal Bank Inference Platform Choice](#12-worked-example-the-cymbal-bank-inference-platform-choice)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary and References](#14-glossary-and-references)

---

### How to Read This Guide

**Relationship to sibling guides.** This is the dedicated deep-dive on the *GPU sharing and virtualization* layer of the Kubernetes AI stack. It deliberately does not re-derive what siblings already cover: GPU *performance engineering* (SMs, warps, CUDA, profiling, tensor cores) lives in [gpu_optimization_guide.md](gpu_optimization_guide.md), which this guide references for the hardware fundamentals that explain *why* sharing mechanisms differ (e.g. why MIG partitions SMs but time-slicing does not). The *inference workloads* that the sharing layer serves are covered in [ai_llm/ollama_xinference_localai_guide.md](ai_llm/ollama_xinference_localai_guide.md) (Ollama, Xinference, LocalAI, llama.cpp serving) and the platform layer above in [ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) and [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md). Cluster and deployment mechanics (OpenShift AI, KubeFlow, on-prem GPU clusters) live in [nutanix_enterprise_ai_vs_openshift_ai_guide.md](nutanix_enterprise_ai_vs_openshift_ai_guide.md), [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md), [devstack_openstack_guide.md](devstack_openstack_guide.md), [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md), [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md), and [cloud_providers_guide.md](cloud_providers_guide.md). The comparison-table and decision-guidance genre follows the vendor head-to-head precedents [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) and [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md).

**Suggested reading paths.** Platform architect choosing a sharing mechanism: §2 → §6 → §10 → §11 → §12. HAMi evaluator: §2 → §3 → §4 → §5. NVIDIA-pure stack user deciding MIG vs time-slicing: §6 → §7 → §11. Manager needing the bottom line: §1 → §2 → §10 → §11 → §12.

**Note on verification.** Facts were researched in September 2026 against primary sources — the HAMi GitHub repository and CNCF pages, NVIDIA documentation, vendor repos, and cloud-provider docs. Claims are marked inline: ✅ **Verified** (confirmed against a primary source or multiple consistent sources during research), ⚠️ **Approximate** (widely cited but version/architecture-dependent), ❓ **Reported** (vendor marketing or single-source claims, treat with care). A consolidated claims-status table follows this section; anything that could not be verified is listed honestly in §13 rather than asserted.

### Claims Status at a Glance

| Claim | Status | Note |
|---|---|---|
| HAMi = **Heterogeneous AI Computing Virtualization Middleware** | ✅ Verified | CNCF project page header + GitHub README (September 2026) |
| HAMi is a CNCF project; formerly named `k8s-vGPU-scheduler` | ✅ Verified | GitHub README ("Formerly known as `k8s-vGPU-scheduler`"); repo commit "Remake this repo to HAMi", Jan 31, 2024 |
| HAMi accepted to CNCF **Sandbox** on August 21, 2024 | ✅ Verified | cncf.io/projects/hami |
| HAMi moved to CNCF **Incubating** maturity on July 2, 2026 (TOC vote; blog July 15, 2026) — *not* graduated | ✅ Verified | cncf.io/projects/hami + CNCF blog "HAMi becomes a CNCF incubating project", July 15, 2026 |
| License: Apache-2.0 | ✅ Verified | GitHub repository license file + README badge |
| Current stable release v2.10.0 (Aug 21, 2026); 17 releases total | ✅ Verified | GitHub releases page (September 2026) |
| HAMi core = a **CUDA-interception library** (HAMi-core, `libvgpu`) enforcing memory/compute limits in-container | ✅ Verified | CNCF incubating blog component list; HAMi architecture docs; repo submodule `libvgpu → Project-HAMi/HAMi-core` |
| HAMi is middleware: mutating webhook + scheduler extender + device plugins + in-container virtualization + monitoring | ✅ Verified | GitHub README "How It Works"; CNCF blog components |
| HAMi supports heterogeneous accelerators: NVIDIA GPUs, Huawei Ascend NPUs, Cambricon MLUs, Hygon DCUs, Iluvatar, Kunlunxin XPU, MetaX, Moore Threads, Enflame, Biren, and more | ✅ Verified | GitHub README supported-devices list + repo commit history (e.g. Kunlunxin XPU added Aug 12, 2026; Ascend 910C vNPU templates July 8, 2026) |
| **AMD support is on the roadmap, not yet shipped** as of the incubating announcement | ✅ Verified | CNCF incubating blog roadmap: "expand device support to include AMD Mi Series and PPU" (July 2026). AMD logo appears on project website marketing as future ecosystem |
| HAMi born inside ByteDance; ByteDance is the original creator | ⚠️ Approximate | Widely reported lineage via `k8s-vGPU-scheduler`; ByteDance appears in the project's contributor/adopter lists, but no CNCF/GitHub primary source names ByteDance as founder — see §2.4 and §13 |
| Maintainers span dynamia.ai, NVIDIA, and independent developers | ✅ Verified | CNCF incubating blog (July 15, 2026) |
| MIG supported on NVIDIA data-center GPUs only: A100/A30 (Ampere) and later (H100, H200, B200 etc.) | ✅ Verified | NVIDIA MIG User Guide "Supported GPUs" |
| MIG provides hardware-level partitioning with dedicated SMs/memory slices and fault isolation | ✅ Verified | NVIDIA MIG User Guide |
| NVIDIA time-slicing provides **no** isolation — deliberate | ✅ Verified | NVIDIA k8s-device-plugin docs on time-slicing |
| MPS (Multi-Process Service) shares a GPU among processes with SM partitioning, no memory isolation | ✅ Verified | NVIDIA MPS docs (see §6.4) |
| vCUDA (original) = academic research (IPDPS 2009) on GPU sharing for VMs via CUDA API interception; research-grade, not a maintained production option | ✅ Verified / ⚠️ | Paper is primary; maintenance status of the original code is not publicly active — several unrelated repos reuse the name (see §8.1) |
| "vGPU" is an overloaded term: NVIDIA vGPU (virtualized GPU for VMs, licensed), HAMi's software vGPU slices, and assorted research projects all use it | ✅ Verified | See §8 |
| HAMi metrics are Prometheus-compatible via the scheduler monitor endpoint (default port 31993), plus HAMi-WebUI and Grafana dashboards | ✅ Verified | GitHub README "Observability and WebUI" |

---

## 1. Scope and Reading Path

A Kubernetes cluster with NVIDIA GPUs typically starts life in the simplest possible configuration: **one whole GPU per pod**. The NVIDIA device plugin advertises each physical GPU as an indivisible resource (`nvidia.com/gpu: 1`), the scheduler hands a pod an entire card, and the pod uses a fraction of it. For AI inference fleets — dozens of small LLM and embedding workloads, each needing 2–8 GB of a 40–96 GB card — this leaves most of the fleet idle and makes GPU cost per useful token absurdly high.

This guide is about the layer of the stack that fixes that: **GPU sharing and virtualization on Kubernetes**. It is organized around one project — **HAMi**, a CNCF-incubating middleware that slices physical accelerators into fractional, isolatable, schedulable units — and compares it systematically against everything else a platform team could reach for instead:

- **NVIDIA-native hardware sharing**: MIG (hardware partitioning), time-slicing (temporal sharing), MPS (concurrent process sharing);
- **The NVIDIA device-plugin stack**: `k8s-device-plugin` and the GPU Operator that deploys it;
- **The vCUDA/vGPU-style interception projects**, including the traps in that category's terminology;
- **Scheduler-level sharing frameworks** (Volcano, Koordinator, KAI-scheduler, and the Kubernetes DRA mechanism that threatens to change the rules);
- **Cloud-vendor GPU sharing** (GKE time-sharing and friends) in condensed form.

The through-line is a question every GPU platform team must answer: *when dozens of tenants need fractions of expensive GPUs, who partitions the card — the hardware, a CUDA-level shim, a scheduler, or the cloud?* The answer determines your isolation story, your utilization ceiling, and your regulatory posture.

### 1.1 Why this is a banking-relevant problem

GPU cost is a measurable P&L line at Cymbal Bank, and the regulatory environment adds constraints that pure-engineering comparisons ignore. MAS (Monetary Authority of Singapore) technology-risk guidance and the bank's own model-risk governance require clear ownership of data, auditable access control, and evidence that one business unit's workload cannot observe or corrupt another's — including at the accelerator boundary. A mechanism that is great at utilization but weak at isolation may be fine for a dev sandbox and unacceptable for a production model serving customer data alongside another business unit's workloads. §11 and §12 turn that tension into a decision procedure.

### 1.2 Layer map: where GPU sharing sits

| Stack layer | What lives there | Covered in |
|---|---|---|
| Model/framework | PyTorch, TensorFlow, vLLM, Ollama, Xinference | [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md), [ai_llm/ollama_xinference_localai_guide.md](ai_llm/ollama_xinference_localai_guide.md) |
| Platform | Kubeflow, OpenShift AI, ML platforms | [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md), [ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) |
| **Sharing/virtualization** | **HAMi, MIG, time-slicing, MPS, device plugins, vGPU shims** | **This guide** |
| Scheduler/cluster | Kubernetes, Volcano, Kueue, Koordinator | [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md), [devstack_openstack_guide.md](devstack_openstack_guide.md) |
| Hardware | NVIDIA SMs, HBM, NVLink, tensor cores | [gpu_optimization_guide.md](gpu_optimization_guide.md) §1 |

---

## 2. HAMi Identity: What HAMi Actually Is

> **Mitigant discipline first.** Before comparing HAMi to anything, this section pins down what HAMi verifiably *is*, from primary sources: the GitHub repository ([github.com/Project-HAMi/HAMi](https://github.com/Project-HAMi/HAMi)), the CNCF project page, the CNCF incubating announcement, and the project's own documentation at [project-hami.io](https://project-hami.io/). Every claim carries its source; where a popular belief (such as the ByteDance founding story) could not be confirmed from a primary source, it is flagged rather than repeated as fact.

### 2.1 The verified one-paragraph identity

**HAMi is an open-source, Kubernetes-native GPU virtualization and heterogeneous-accelerator scheduling middleware.** ✅ Verified — the GitHub README opens: *"Kubernetes GPU virtualization and heterogeneous accelerator scheduling for AI infrastructure"* and the project site describes it as *"an open-source, cloud-native GPU virtualization middleware that brings sharing, isolation and scheduling of heterogeneous accelerators to AI workloads on Kubernetes."* Its stated purpose is to let platform teams **share expensive GPUs and other AI accelerators across Kubernetes workloads, isolate device memory and compute, and schedule pods with device-aware policies without changing application code.** ✅ Verified (README).

The name expands to **Heterogeneous AI Computing Virtualization Middleware**. ✅ Verified — CNCF project page header and GitHub README agree verbatim.

It is a **CNCF Incubating project** (as of July 2026), Apache-2.0 licensed, written ~96% in Go, with the core CUDA-interception library maintained as a separate repository. ✅ Verified (see §2.2 and §2.3).

**Quick facts (all ✅ verified September 2026).**

| Fact | Value |
|---|---|
| Full name | Heterogeneous AI Computing Virtualization Middleware |
| One-line job | Kubernetes GPU virtualization + heterogeneous accelerator scheduling for AI infrastructure |
| Repo / org | github.com/Project-HAMi/HAMi, under the Project-HAMi org |
| Former name | `k8s-vGPU-scheduler` (rebranded January 2024) |
| License | Apache-2.0 |
| CNCF status | Incubating (Sandbox Aug 21, 2024 → Incubating July 2, 2026) |
| Latest release (research date) | v2.10.0, Aug 21, 2026 (17th release) |
| Language | Go (~96%), Shell (~3%) |
| Core mechanism | CUDA driver-API interception (HAMi-core / `libvgpu`) + scheduler extender + device plugins |
| Primary GPU path | NVIDIA; heterogeneous backends incl. Ascend, Cambricon, Hygon, Iluvatar, Kunlunxin, MetaX, Moore Threads, Enflame, Biren |
| AMD | Roadmap (Mi-series, PPU) — not shipped at research date |
| Community | ~4.5k GitHub stars, ~800 forks, 177 repo contributors; 5 CNCF case studies |

### 2.2 What HAMi is not (boundary-setting)

Equally important is what HAMi is **not**, because the surrounding ecosystem invites confusion:

- **Not a device driver** — it sits above the vendor driver and CUDA runtime, intercepting the CUDA driver API for NVIDIA devices from inside the container. ✅ Verified (CNCF blog component description, §3).
- **Not a fork or replacement of NVIDIA's k8s-device-plugin** — HAMi ships its *own* device plugins (including an NVIDIA one) and a scheduler extension, but is explicitly designed to coexist with the NVIDIA GPU Operator when the Operator manages drivers and HAMi manages scheduling. ✅ Verified (README ecosystem table).
- **Not a hardware-partitioning technology like MIG** — HAMi's default NVIDIA path virtualizes at the CUDA API layer (software slices it calls vGPUs); it *also* orchestrates MIG where hardware support exists (dynamic MIG mode). ✅ Verified (README scheduling policies; §4.4).
- **Not NVIDIA-only** — the "H" in HAMi is the point: NVIDIA is one backend among many (Ascend, Cambricon, Hygon, Iluvatar, Kunlunxin, MetaX, Moore Threads, Enflame, Biren, AWS Neuron and more per the current support matrix). ✅ Verified (README, project site, commit history).
- **Not a graduated CNCF project** — status is **Incubating** (July 2, 2026). Common online descriptions still call it "a CNCF sandbox project"; that was true from August 21, 2024 until July 2026. ✅ Verified — see the correction in §2.3 and the claims table.

### 2.3 Provenance and CNCF timeline (verified)

| Date | Event | Source |
|---|---|---|
| 2018–2023 (approx.) | Pre-history as `k8s-vGPU-scheduler`, a Kubernetes vGPU sharing project | ✅ README: "Formerly known as `k8s-vGPU-scheduler`"; CNCF project insights record a first commit date of March 12, 2018 |
| Jul 12, 2021 | Earliest LICENSE commit in the current repository lineage | ✅ GitHub commit history |
| Jan 31, 2024 | Repo commit "Remake this repo to HAMi" — the rename/rebrand point | ✅ GitHub commit `5ba2177` |
| ~Mar–Apr 2024 | Open-sourcing widely referenced; maintainer quotes recall HAMi "had just been open-sourced" at KubeCon Paris 2024 (March 2024) | ❓ Reported (maintainer quote in CNCF blog) |
| Aug 21, 2024 | **Accepted into CNCF Sandbox** | ✅ cncf.io/projects/hami |
| Jul 2, 2026 | **Moved to CNCF Incubating** (TOC vote) | ✅ cncf.io/projects/hami |
| Jul 15, 2026 | CNCF blog "HAMi becomes a CNCF incubating project" published | ✅ cncf.io blog |
| Aug 21, 2026 | v2.10.0 released (17th release); active development continuing into September 2026 (commits Sep 5, 2026) | ✅ GitHub releases + commit history |

### 2.4 The ByteDance question

The claim that HAMi was born out of ByteDance's internal GPU-sharing work is widespread in community write-ups. What the primary sources actually support:

- ✅ The project's pre-rename identity, `k8s-vGPU-scheduler`, is a Kubernetes GPU-sharing scheduler of the kind large Chinese internet companies built internally in the 2019–2022 era.
- ⚠️ ByteDance appears in HAMi's ecosystem materials as a *contributor/adopter* (project-hami.io contributor and contributor-count pages include the ByteDance logo alongside Alibaba, Huawei, JD.com and others).
- ❌ No primary source examined for this guide — the GitHub README, CNCF project page, or CNCF incubating blog — states that ByteDance created HAMi. The incubating blog attributes the project's growth to a multi-company maintainer set (dynamia.ai, NVIDIA, independents) and does not mention ByteDance.

**Honest verdict:** the ByteDance founding story is *plausible and widely repeated* but **not verified from a primary source** in this research pass; treat it as reported lineage (⚠️). It does not affect any engineering or procurement conclusion in this guide. See §13.

### 2.5 Governance and community snapshot (September 2026)

- **License:** Apache-2.0; copyright "Contributors to HAMi, established as HAMi a Series of LF Projects, LLC." ✅
- **Organization:** hosted under the `Project-HAMi` GitHub org, which also carries `HAMi-core` (the interception library), `HAMi-WebUI`, the community repo, and the Helm chart repository. ✅
- **Maintainers:** span multiple companies including dynamia.ai and NVIDIA, plus independent developers — multi-vendor governance consistent with CNCF incubation requirements. ✅ (CNCF blog). Maintainer names visible in the repo include Mengxuan Li and Xiao Zhang (both listed as maintainers in the CNCF announcement). ✅
- **Community metrics (September 2026):** ~4.5k GitHub stars, ~800 forks, 177 repository contributors (GitHub); project site snapshots cite 471+ DevStats contributors across 25 countries and ~306k Docker pulls; CNCF LFX insights cite 3,630 total contributors across 699 organizations. ✅ (multiple consistent sources; exact numbers drift daily)
- **Governance and security hygiene:** OWNERS file, AUTHORS.md, community governance doc, OpenSSF Best Practices badge, OpenSSF Scorecard workflow, Trivy scanning, security policy. ✅ (repo). One honest counter-signal: CNCF LFX "health score" for the project was rated **Concerning (38)** in the September 2026 snapshot — a composite contributor/security metric worth watching, not a disqualifier (see §5.3).
- **Case studies:** five CNCF-published case studies as of July 2026, including DaoCloud (10,000+ GPUs across 10+ data centers) and China Merchants Bank (100% hardware-pool utilization claim via topology-aware scheduling, July 2026), plus SNOW Corp (1,000+ GPUs for a GenAI consumer service). ✅ (CNCF project page and incubating blog). Note: China Merchants Bank is a real bank — it is cited here only as a public CNCF case study of HAMi adoption, never as a persona in this guide's worked examples.

### 2.6 Where HAMi lives today (release and repo state)

As of this research pass (September 2026): **v2.10.0** is the latest release (Aug 21, 2026), on the `master` branch with 1,726+ commits and near-daily activity (latest commit observed Sep 5, 2026). ✅ The repo carries `charts/` (Helm), `dashboards/` (Grafana), `benchmarks/`, `docs/`, `examples/`, and vendor-specific code under `pkg/` — an actively maintained, release-train project, not a research artifact. ✅

---

## 3. HAMi Architecture: From Request to Isolation

HAMi is composed of a **mutating webhook**, a **scheduler extender**, **device plugins**, and **device-specific in-container virtualization components**. ✅ Verified (README "How It Works"; CNCF blog components). The pieces, verified against the README pipeline diagram and the CNCF blog:

### 3.1 Component map

| Component | Role (verified) |
|---|---|
| **Mutating webhook** | Intercepts pod submissions and rewrites scheduler fields and resource requests for workloads requesting virtualized devices |
| **Scheduler extender** (`hami-scheduler`) | Filters, scores, and binds pods to nodes/devices using binpack, spread, topology-aware, and device-specific policies; writes the chosen device allocation into pod annotations |
| **Device plugins** (`hami-device-plugin`, per vendor) | Register accelerators with Kubernetes and allocate fractional device resources into containers at `Allocate()` time |
| **HAMi-core** (a.k.a. `libvgpu`, separate repo) | In-container virtualization layer; for NVIDIA it intercepts the native CUDA driver API to enforce hard runtime limits on GPU memory and compute |
| **HAMi-WebUI** | Visual cluster/device management |
| **Observability layer** | Prometheus-compatible metrics endpoint on the scheduler (default port 31993) + Grafana dashboards |

### 3.2 The request-to-isolation pipeline

The README documents the flow as:

```
Pod submission
  -> HAMi mutating webhook
  -> HAMi scheduler filter / score / bind
  -> device allocation written to pod annotations
  -> device plugin Allocate()
  -> container runtime environment
  -> HAMi monitor and metrics
```

In plain terms: a user submits an ordinary Pod requesting, say, `nvidia.com/gpu: 1` plus `nvidia.com/gpumem: 3000` (3,000 MiB of GPU memory). The webhook rewrites the request, the HAMi scheduler extension picks a physical device with enough free memory, the chosen allocation is recorded in pod annotations, the device plugin injects the right environment (device index, HAMi-core library preload), and inside the container HAMi-core makes the workload *believe* it owns a GPU with the allocated memory and compute envelope — while enforcing those limits against the real device. ✅ Verified (README example + docs).

### 3.3 How HAMi-core enforces limits: CUDA API interception

The mechanism that makes HAMi's software vGPU work — and that fundamentally distinguishes it from MIG — is **CUDA API interception**. For NVIDIA devices, HAMi-core intercepts the CUDA driver API (a technique in the family of the LD_PRELOAD shims used by vCUDA-style projects, see §8) so that calls like memory allocation (`cuMemAlloc` family) are routed through HAMi's accounting rather than the real driver. ✅ Verified (CNCF blog: "enforcing hard runtime limits on GPU memory and compute, intercepting the native CUDA driver for NVIDIA devices"; HAMi docs "GPU Virtualization Principles").

What this buys, and what it costs, sets up the entire comparison in §10:

- **MB-granularity memory slicing**: a 96 GB H100 can be carved into any mix of, e.g., 4 GB / 8 GB / 16 GB slices, because the limit is enforced in software, not by fixed hardware partitions. ✅
- **Compute control**: HAMi-core can cap compute (core) usage per container; repository commit history documents a contention policy where a container is *blocked* when a strictly higher-priority task shares the device and *throttled* when another container of its own priority class contends (observed via the shared-memory region the device plugin mounts into the container). ✅ (repo commit messages, e.g. #2931)
- **The cost**: isolation is only as strong as the interception layer. It is a software enforcement boundary, not a hardware one — a malicious or buggy workload running *native* driver calls through an un-hooked path, or exploiting a gap in the intercepted API surface, operates on the assumption set of software virtualization. This is the single most important consideration for regulated multi-tenant use (see §11.3).

**The interception walk-through (what actually happens).** For a CUDA workload running under HAMi: (1) the device plugin injects HAMi-core into the container (library preload plus environment variables identifying the allocated vGPU); (2) the application's CUDA driver calls — context creation, `cuMemAlloc`/`cuMemFree`, launches — route through HAMi-core's interposed implementations; (3) HAMi-core tracks the workload's live device memory against its `gpumem` allowance and fails allocations that would exceed it (the workload sees CUDA out-of-memory rather than destabilizing co-tenants); (4) compute-envelope and priority logic applies to scheduling/launch behavior so a capped container cannot monopolize SMs; (5) the monitor path (vGPUmonitor) reads per-container usage from the same shared-memory region and exports it to Prometheus. The application itself is unmodified and generally unaware — which is precisely the "zero application changes" property, and precisely why the enforcement is only as trustworthy as the interposed layer. ✅ (architecture and commit evidence cited above; step-level detail ⚠️ synthesized from repo/docs — see §13)

### 3.4 Modes: device-plugin mode and DRA mode

The project site and release notes document two operating modes for HAMi's runtime: the classic **device-plugin mode** (extended resources `nvidia.com/gpu` + `gpumem`/`gpucores`, described above) and a **DRA mode** (Dynamic Resource Allocation, the Kubernetes 1.26+ alpha/beta resource model that is intended to eventually supersede device plugins). ✅ (project site architecture tab; v2.10 changelog commits "remove dra from charts" indicate the DRA path was being reworked in the 2.10 cycle). Kubernetes DRA's relationship to HAMi is discussed in §8.4, and the CNCF published "Does Kubernetes DRA Replace HAMi?" (Aug 7, 2026) — see §5.4.

---

## 4. HAMi Capabilities: Resource Model, Scheduling and MIG

### 4.1 The fractional resource model

HAMi extends the standard Kubernetes extended-resource model with two additional requestable dimensions for NVIDIA devices: memory and compute cores. ✅ Verified (README, docs):

```yaml
resources:
  limits:
    nvidia.com/gpu: 1        # one physical GPU (the pod requests a fraction of it)
    nvidia.com/gpumem: 3000  # 3,000 MiB of GPU memory on that GPU
    # nvidia.com/gpucores: 50   # optional compute-envelope percentage
```

Two README notes matter for anyone reading HAMi manifests: (1) after HAMi is installed, the node's advertised `nvidia.com/gpu` count defaults to the number of *vGPUs* the node can host, and (2) in a pod request, `nvidia.com/gpu` refers to the number of *physical GPUs* the pod needs. ✅ Verified (README). Memory granularity is in MiB, which is the property that lets dozens of small inference workloads share one card cleanly — the core use case of this guide's worked example.

### 4.2 Scheduling policies

HAMi's scheduler extension supports, per the README: **binpack** (consolidate onto fewer devices/nodes), **spread** (distribute to reduce contention), **topology-aware scheduling** (device selection respects GPU/NVLink topology when supported), and **dynamic MIG** (§4.4). ✅ Verified. It runs along the default kube-scheduler path (as an extender) and is documented to work alongside Volcano for gang-scheduling and queue-based batch workloads, with Kueue integration via resource transformation, and with Koordinator and NVIDIA's KAI-scheduler. ✅ Verified (README + project site ecosystem list).

### 4.3 Observability: metrics, WebUI, dashboards

- Prometheus-format metrics from the scheduler monitor endpoint (`http://<scheduler-ip>:<monitor-port>/metrics`, default port 31993), including per-container GPU metrics — memory usage and utilization — exported by the in-node monitoring path. ✅ Verified (README; repo commit #2898 documents per-container vGPU allocation metrics and #2931 documents the vGPUmonitor contention metrics).
- HAMi-WebUI for cluster/device management, and prebuilt Grafana dashboards under `dashboards/`. ✅

### 4.4 MIG mode: dynamic MIG

HAMi's scheduler supports **dynamic MIG**: creating and allocating NVIDIA MIG instances on the fly for supported cards, rather than requiring the cluster admin to pre-partition GPUs with `nvidia-mig-parted` (the tool the GPU Operator uses). ✅ Verified (README scheduling-policy list; repo build history pins `nvidia-mig-parted` v0.14.4, and July 2026 commits added dedicated MIG end-to-end CI test targets — `e2e-mig-test`, `e2e-mig-smoke-test` — gating releases). HAMi's MIG support means the same control plane can hand out *hardware-isolated* MIG slices to tenants that need them and *software vGPU* slices to tenants that do not, on the same fleet. This hybrid capability is rare among the alternatives and is discussed in §10 and §11.

### 4.5 Device support matrix (heterogeneous)

HAMi's differentiating scope is multi-vendor. The README's supported list (September 2026) spans NVIDIA GPUs plus Ascend (NPUs, including 910C vNPU fractional templates added July 2026), Cambricon (MLUs), Hygon (DCUs), Iluvatar, Kunlunxin (XPUs, added August 2026), MetaX, Moore Threads, Enflame, Biren, and others (the docs maintain a full support matrix at project-hami.io/docs/userguide/device-supported). ✅ Verified (README, commit history). Crucially for accuracy: **AMD is not yet in the shipped matrix** — the CNCF blog lists "AMD Mi Series and PPU" as roadmap items (July 2026), and the AMD logo on the project site is future-looking marketing. Do not deploy HAMi today expecting AMD ROCm backends. ⚠️ (see claims table).

### 4.6 Ecosystem integrations (verified)

vLLM (running inference servers under GPU memory caps so multiple models share one GPU), Volcano (gang/queue scheduling), Kueue (batch queueing via ResourceTransformation), Koordinator, KAI-scheduler (NVIDIA's Kubernetes AI scheduler; the HAMi community reports KAI-scheduler adopting HAMi-core as its sharing engine — ❓ Reported, see §13), Prometheus, Grafana, and coexistence with the NVIDIA GPU Operator (HAMi schedules; Operator manages drivers). ✅ Verified (README ecosystem table) except where noted.

### 4.7 Known limits and honest caveats

A balanced capability section also records what HAMi does *not* do, so downstream decisions are not built on a marketing read:

- **Software boundary**: in its default (non-MIG) mode the enforcement boundary is user-space CUDA interception — robust for *cooperative* tenancy and runaway-memory protection, not a hardware security or fault-isolation boundary. MIG mode (dynamic or static) is the hardware-grade path where supported. ✅ (§3.3, §10)
- **Driver/API coupling**: interception libraries track the CUDA driver API surface; major driver/CUDA upgrades should be regression-tested against HAMi-core versions (the project pins driver/CUDA toolchains in CI and releases in lockstep — ✅ release-train evidence — but the burden of upgrade testing sits with the platform team).
- **MIG requires MIG-capable hardware and mode**: dynamic MIG only helps on cards where MIG exists and is enabled (data-center Ampere+); on other cards HAMi's software mode is the only option. ✅ (§4.4, §6.1)
- **Vendor support is uneven by design**: the README's own caveat — "device capabilities vary by vendor, model, driver, and hardware generation" — means the *heterogeneous* story is real but the *NVIDIA* path is the most mature; non-NVIDIA backends should be validated against the current support matrix before committing. ✅ (README)
- **AMD is not here yet**: as of September 2026, AMD (Mi-series) is roadmap, not shipped. ⚠️ (§2 claims table)

---

## 5. HAMi Maturity, Adoption and Governance

### 5.1 Maturity trajectory

HAMi's arc from rebrand (January 2024) to CNCF incubation (July 2026) is unusually fast for a CNCF project, and the release train is steady: 17 releases by August 2026, with v2.9.0 stable at the incubation announcement and v2.10.0 (Aug 21, 2026) current. ✅ Verified. Community growth numbers are consistent across CNCF LFX and GitHub (see §2.5). The project carries real production references, including two financial-sector adjacent deployments (China Merchants Bank case study, July 2026; DaoCloud's multi-tenant GPU cloud at 10,000+ GPUs). ✅

### 5.2 Adoption patterns

The adopter list (project-hami.io) skews heavily to Chinese cloud/AI companies (DaoCloud, Baidu AI Cloud, Huawei, China Mobile, iFlytek, 4Paradigm, etc.) with growing international names (LinkedIn, SAP, NIO, Visa, Oracle, Snow, Viettel). ⚠️ Adopter logos are self-reported by the project; treat individual entries as marketing-grade evidence unless backed by a CNCF case study. The five CNCF case studies are the stronger evidence. ✅

**What the adoption evidence does — and does not — prove.** The DaoCloud (10,000+ GPUs, 10+ data centers), SNOW Corp (1,000+ GPUs, consumer GenAI traffic spikes), and China Merchants Bank (hardware-pool utilization) case studies prove that HAMi *scales and is operated in production at serious fleet sizes by sophisticated platform teams*. They do not prove: (a) that those deployments used HAMi's software vGPU mode for *regulated multi-tenant* isolation (the CMB study centers on scheduling utilization, not on security-boundary claims); (b) that the interception layer has been adversarially audited — no independent security audit of HAMi-core was found in this research pass (⚠️, see §13); or (c) that HAMi is the right choice for a *small* NVIDIA-only cluster (the §11 decision procedure may well land on plain GPU Operator there). Reading case studies as capability proof for *your* exact constraint set is a classic procurement error; the worked example in §12 shows the constraint-first way to use them.

### 5.3 Governance and risk signals

CNCF incubation brings neutral governance: the project is "a Series of LF Projects, LLC," maintainers are multi-company (dynamia.ai, NVIDIA, independents), and there is a community governance document, contributor ladder, and code of conduct. ✅. Honest risk flags for a bank doing due diligence: (a) the CNCF LFX health score was "Concerning (38)" in the September 2026 snapshot — driven largely by contributor-concentration and security-hygiene sub-scores — worth re-checking at procurement time; (b) maintainer gravity is in Asia (China-time-zone community meetings); (c) single-vendor *dominance* has been replaced by dynamia.ai+NVIDIA, but it is not yet a broad foundation-style community like Kubernetes itself. None of these are blockers; they are diligence items. ✅/⚠️

### 5.4 The DRA question

Kubernetes **Dynamic Resource Allocation** (alpha in 1.26, still evolving through 2026) is the upstream mechanism that may one day subsume device plugins. HAMi already has a DRA mode, and the CNCF published "Does Kubernetes DRA Replace HAMi?" (August 7, 2026) — the short answer being that DRA standardizes the *allocation plumbing* while HAMi supplies the *device virtualization and heterogeneous scheduling* above it. ✅ (CNCF blog title and project materials). This is the strategic question to watch through 2026–2027, revisited in §8.4.

---

## 6. NVIDIA-Native Sharing: MIG, Time-Slicing and MPS

NVIDIA offers three native mechanisms for sharing one physical GPU, plus a fourth (vGPU) that is a licensed virtualization product. They are frequently confused, so this section pins down each one against NVIDIA documentation before the comparison in §10. The most important discipline note up front: **the three mechanisms occupy completely different layers** — MIG partitions in *hardware* (space), time-slicing shares in *time* (software scheduler), and MPS shares *execution contexts* so processes can overlap (a hybrid of space and time at the driver level).

### 6.1 MIG — Multi-Instance GPU (hardware partitioning) ✅

**What it is.** MIG partitions a supported GPU into multiple isolated hardware instances, each with its own dedicated compute (SMs), L2 cache slices, and memory slices. ✅ Verified — NVIDIA's MIG User Guide: MIG partitions GPUs "into multiple isolated instances, each with dedicated compute and memory resources… with guaranteed performance," and each instance behaves like an independent, smaller GPU with its own fault domain.

**Supported hardware.** Data-center GPUs from Ampere onward: A100 and A30, Hopper H100/H200, and Blackwell-era data-center parts (B200 and successors); also available on DGX/HGX systems. ✅ Verified (NVIDIA MIG User Guide "Supported GPUs"). Consumer/GeForce and most laptop GPUs do **not** support MIG — a common misconception. ⚠️ Exact per-GPU profiles and instance maxima (e.g., up to seven compute instances on A100/H100-class parts) vary by SKU and memory size; treat specific slice profiles (e.g. `1g.5gb`, `2g.10gb`, `3g.20gb` on A100) as ⚠️ architecture-dependent.

**Properties that matter for comparison.**

- **Hard isolation**: dedicated SMs and memory slices mean one instance cannot consume another's memory or compute, and hardware faults are contained. This is the *only* NVIDIA-native mechanism with genuine multi-tenant isolation. ✅
- **Fixed granularity**: partitioning follows a discrete profile table (compute-instance × memory-slice combinations); you cannot ask for an arbitrary 6.3 GB. Memory slices come in coarse fractions of the card. ✅/⚠️
- **Operational cost**: enabling MIG requires the GPU to be put into MIG mode (driver-level reconfiguration, often needing a reboot or GPU reset), and instances must be created/configured — either statically up front or dynamically by tooling. ✅ (NVIDIA docs; and see the GPU Operator's MIG manager, §7).
- **Performance guarantees**: dedicated resources give near-linear per-instance performance and predictable QoS — the property that HAMi's software slices cannot fully replicate. ✅
- **Not supported everywhere**: no MIG on many cloud instance types where the hypervisor does not expose it, no MIG with certain virtualization stacks, and some multi-GPU communication (NVLink/peer-to-peer across MIG) is restricted. ⚠️ (cloud section, §9).

**Kubernetes integration.** The k8s-device-plugin exposes MIG with three strategies (`none`/`single`/`mixed`) via `MIG_STRATEGY`; in `mixed` mode it advertises per-profile resources such as `nvidia.com/mig-1g.5gb`. ✅ Verified (k8s-device-plugin README). In practice, nodes are pre-partitioned with `nvidia-mig-parted` (static MIG profiles applied at node bring-up), which is what the GPU Operator automates. ✅

### 6.2 Time-slicing (temporal sharing) ✅

**What it is.** The k8s-device-plugin can advertise each physical GPU as N replicas; CUDA time-slicing then lets workloads sharing a GPU interleave on it. ✅ Verified (k8s-device-plugin README, "Shared Access to GPUs"). There is no hardware or software partitioning at all — every replica sees the *entire* GPU memory and the whole device.

**Properties.** Configurable per node via the device-plugin config file (`sharing.timeSlicing.resources[].replicas`); simplest way to raise utilization; near-zero setup. But the NVIDIA documentation is unusually blunt about the trade-off: *"nothing special is done to isolate workloads that are granted replicas from the same underlying GPU, and each workload has access to the GPU memory and runs in the same fault-domain as all the others (meaning if one workload crashes, they all do)."* ✅ Verified — quote from the k8s-device-plugin README. Time-slicing deliberately provides **no memory isolation, no compute isolation, and no fault isolation**. It is a utilization tool for trusted workloads, not a multi-tenancy mechanism.

### 6.3 MPS — Multi-Process Service (concurrent process sharing) ✅

**What it is.** MPS is a lightweight runtime service (control daemon + server + client runtime built into the CUDA driver) that lets multiple CUDA processes submit work concurrently and have their kernels and memcpys overlap on one GPU. ✅ Verified (NVIDIA MPS docs). Its documented benefits are utilization (overlapping work from many processes), reduced context-switching overhead, and reduced on-GPU context storage. ✅

**Properties.** MPS is a *cooperative* sharing mechanism for many small processes — the classic case being one node running dozens of MPI ranks or many model replicas. It is **not**, in its classic form, an isolation boundary: MPS clients historically share the GPU's memory space (a single client can use the whole device memory) and fault domain. ✅ (NVIDIA MPS docs' architecture and limitations). Newer driver-level controls (pinned-device-memory limits, per-process SM limits, and the k8s-device-plugin's MPS sharing mode) add *configurable* memory/compute partitioning and per-workload enforcement. ✅ (k8s-device-plugin README notes its MPS mode "does space partitioning and allows memory and compute resources to be explicitly partitioned and enforces these limits per workload"). The device-plugin README also states **time-slicing and MPS modes are mutually exclusive** in that configuration. ✅

### 6.4 NVIDIA vGPU (the licensed one) — a naming hazard ⚠️

Separate from all of the above is **NVIDIA vGPU** (formerly GRID): the licensed virtualization product that presents *virtual* GPUs to VMs, managed by the NVIDIA vGPU manager on the host and typically consumed through vSphere/Hyper-V/OpenStack. ✅ (NVIDIA vGPU product docs; the GPU Operator even ships a "NVIDIA vGPU Device Manager" image for such environments). The name collides constantly with HAMi's marketing of its software slices as "vGPUs" and with assorted research projects — when someone says "vGPU sharing," ask *whose* vGPU. See §8.

### 6.5 The native-mechanism summary

| Mechanism | Layer | Isolation | Granularity | Main use |
|---|---|---|---|---|
| MIG | Hardware (SMs + memory slices) | Strong (dedicated, fault-contained) | Fixed profiles | Multi-tenant, QoS-critical, regulated |
| Time-slicing | Software scheduler | None (deliberate) | Whole GPU per replica | Utilization of trusted workloads |
| MPS | Driver-level process multiplexing | Classic: none; configurable limits in plugin mode | Process-level | Many small processes on one GPU |
| NVIDIA vGPU | Licensed host hypervisor | VM-level (licensed) | Virtual GPU profiles | VM/VDI virtualization stacks |

### 6.6 Choosing among the three native mechanisms

A fast rule of thumb for the NVIDIA-native trio, formalized later in §11:

- Reach for **MIG** when the question is *"can tenant A affect tenant B?"* and the answer must be *no, in hardware* — even though you pay in setup complexity and coarse granularity.
- Reach for **time-slicing** when the question is *"can I squeeze more jobs onto a card I fully trust?"* — you accept that a neighbor can OOM you or crash you, because the neighbors are your own team's jobs.
- Reach for **MPS** when the question is *"can many small processes of one workload overlap efficiently?"* — MPS is about *process* concurrency (HPC ranks, many model replicas of one owner), not about *tenant* separation; it is the wrong tool for multi-tenant isolation and only a partial tool even with the device-plugin's partitioned MPS mode.
- And remember the device-plugin constraint: time-slicing and MPS sharing modes are mutually exclusive per node configuration, and the same sharing method applies to all GPUs on a node — you cannot mix per-GPU sharing methods with the stock plugin the way HAMi's per-request model allows. ✅ (k8s-device-plugin README)

---

## 7. The NVIDIA Device-Plugin Landscape: k8s-device-plugin and GPU Operator

### 7.1 The baseline: NVIDIA k8s-device-plugin ✅

The k8s-device-plugin ([github.com/NVIDIA/k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin), Apache-2.0, ~3.9k stars, very active) is the *foundation* of every NVIDIA GPU on Kubernetes: it registers GPUs with the kubelet via the device-plugin API and exposes the `nvidia.com/gpu` resource. ✅ Verified. By itself it treats each GPU as an indivisible unit; its config file adds the sharing and MIG capabilities from §6.2/§6.3/§6.1 (time-slicing replicas, MPS mode, MIG strategies `none|single|mixed`). ✅ Verified (README). It is vendor-maintained, ubiquitous, and the *baseline* this guide compares everything against: any HAMi or MIG deployment still assumes a working device-plugin layer underneath (HAMi ships its own; the GPU Operator deploys NVIDIA's).

### 7.2 The NVIDIA GPU Operator ✅

The GPU Operator ([github.com/NVIDIA/gpu-operator](https://github.com/NVIDIA/gpu-operator), docs.nvidia.com/datacenter/cloud-native/gpu-operator) is a Kubernetes operator that automates the *entire* NVIDIA node software stack: drivers (via the Driver Manager), the NVIDIA Container Toolkit, the k8s-device-plugin, GPU Feature Discovery (node labeling), DCGM-based monitoring, and more — installable as a Helm chart on bare metal, EKS, AKS, GKE, OpenShift, and others. ✅ Verified (GPU Operator docs, release train current to 26.7 in 2026). It also manages MIG configuration via `nvidia-mig-parted` (applying static MIG profiles to nodes at bring-up) and can deploy the vGPU device manager for licensed vGPU environments. ✅

### 7.3 How the pieces relate (and where HAMi fits)

- GPU Operator = **lifecycle automation** for NVIDIA software components (drivers → runtime → plugin → monitoring). It does not itself virtualize or share GPUs beyond what the device-plugin config does (time-slicing/MPS/MIG exposure). ✅
- k8s-device-plugin = the **resource-exposure layer** (whole GPUs, MIG devices, time-sliced replicas). ✅
- HAMi = an **additional control plane above/beside the plugin layer** that (a) does fine-grained MB-level memory/compute allocation and CUDA-level enforcement (its own plugin + HAMi-core), (b) adds device-aware scheduling (binpack/spread/topology/dynamic MIG), and (c) generalizes to non-NVIDIA accelerators. ✅ (README). The two stacks are explicitly designed to coexist — the HAMi README lists GPU Operator coexistence ("HAMi manages scheduling and the Operator manages drivers") as a supported integration. ✅

For a team that only runs NVIDIA and only needs whole-GPU or coarse sharing, GPU Operator + k8s-device-plugin is the boring, well-supported default. For fine-grained slicing, hard per-workload memory caps, heterogeneous vendors, or dynamic MIG, HAMi adds capability the NVIDIA stack does not have natively — at the cost of another moving part (see §10).

### 7.4 Practical split: what runs where

The coexistence story (§7.3) has a concrete operational shape that platform teams should plan from day one:

- **Drivers, toolkit, DCGM**: GPU Operator on every GPU node (or the vendor's equivalent) — regardless of which sharing layer sits above. Nobody should hand-roll driver lifecycle in 2026. ✅
- **Whole-GPU + MIG + time-slicing pools**: NVIDIA k8s-device-plugin, configured per pool (MIG `mixed` strategy on the MIG pool; time-slicing replicas on the dev pool). Zero extra components.
- **Fractional memory-sharing pools**: HAMi's device plugin + scheduler extender + webhook on labeled nodes only (`gpu=on`), with HAMi-core injected into those pods. The NVIDIA plugin and the HAMi plugin can coexist because they manage *different nodes'* devices — the label model is the isolation boundary between the two stacks. ✅ (README label semantics)
- **One monitoring plane**: DCGM for node/GPU health (GPU Operator) plus HAMi's per-container allocation metrics for the shared pools — two sources, two purposes, both feeding the same Prometheus/Grafana.

The mistake to avoid is running HAMi and NVIDIA's plugin *racing* to advertise the same physical GPU: unlabeled nodes stay with the NVIDIA stack, labeled nodes move to HAMi, and node pools are tainted accordingly so no workload lands on the wrong stack by accident. This split is exactly the shape of the Cymbal Bank deployment in §12.

---

## 8. vCUDA-Style and Other Sharing Frameworks

### 8.1 vCUDA: the ancestor and the name-collision trap ⚠️

**Original vCUDA** is an academic research project — "vCUDA: GPU Accelerated High Performance Computing in Virtual Machines," published at IPDPS 2009 (Shi, Chen, Sun). ✅ Verified (the paper itself is archived at aimlab.org). It demonstrated GPU sharing for VMs by intercepting CUDA API calls and forwarding them to the host — the intellectual ancestor of the interception technique HAMi-core uses (with the crucial difference that vCUDA targeted VMs in 2009, while HAMi targets containers on Kubernetes in the 2020s and enforces limits in-container). ⚠️ The research code had no sustained upstream maintenance; no maintained official repository was found in this research pass. **Do not treat original vCUDA as a production option** — it is a research artifact, and the task brief's suggested `github.com/chenzhao/vcuda` returned a 404 during verification (September 2026), which is consistent with the project being unmaintained/orphaned.

**Name collision warning.** "vCUDA" is now a generic term reused by unrelated projects of wildly different quality: `tkestack/vcuda-controller` (a Tencent-originated Kubernetes GPU virtualization controller that intercepts CUDA/NVML calls to enforce memory/compute limits — a real, HAMi-like architecture in the same interception family), assorted `vcuda-hook` shims, and Chinese cloud write-ups that use "vCUDA" loosely. ✅ Verified (GitHub). When evaluating anything named vCUDA, verify the exact repository, license, activity, and maintainer before assuming it is the 2009 research project or anything production-grade.

### 8.2 The Chinese cloud-vendor in-house family: cGPU, qGPU and siblings ⚠️

Chinese hyperscalers built internal GPU-sharing/kernel-level slicing systems in the same era HAMi's predecessor came from: Alibaba's **cGPU** and Tencent's **qGPU** both ship as proprietary kernel-module-based GPU slicing on their public clouds (aliyun ACK, tencent TKE), offering in-kernel memory/compute enforcement that is stronger than pure CUDA-user-space interception but is **closed-source and bound to the vendor's cloud**. ❓ Reported (vendor documentation and third-party comparisons; not independently verified here). These matter as the commercial context for HAMi's popularity in China — HAMi is effectively the *open, multi-cloud* answer to the same problem, which is why DaoCloud and others built GPU clouds on it (see §5).

### 8.3 Scheduler-level sharing frameworks (active, verified)

Rather than virtualizing at the CUDA layer, scheduler-level frameworks make the *scheduler* GPU-memory-aware and let multiple pods co-locate on a GPU with **no in-container enforcement** (isolation by convention, not by mechanism):

- **Alibaba `gpushare-scheduler-extender`** (github.com/AliyunContainerService/gpushare-scheduler-extender): an early (2019-era) scheduler extender that tracks per-GPU memory so multiple pods share one card via `aliyun.com/gpu-mem`. ✅ Verified (repo). Now largely **superseded/legacy** — Alibaba's current ACK offering is cGPU-based and the community energy moved to Volcano/Koordinator-style stacks. ⚠️
- **Volcano** (CNCF incubating): the batch scheduler for AI/HPC workloads; it contributes *gang scheduling, queues, preemption* and can co-schedule GPU-sharing pods, but Volcano itself is a scheduler, not a virtualization layer — GPU sharing in Volcano-based clusters is typically provided by a device-sharing plugin underneath (HAMi documents Volcano integration; Volcano's own docs describe a gpushare device plugin for fractional GPUs). ✅ (CNCF; HAMi README; Volcano docs).
- **Koordinator** (CNCF sandbox/incubating trajectory): Alibaba-originated QoS/colocation scheduler that includes GPU-sharing support; HAMi documents Koordinator integration for GPU-sharing workflows. ✅ (CNCF blog "HAMi… with Koordinator for GPU-sharing workflows"; Koordinator docs).
- **NVIDIA KAI-scheduler**: NVIDIA's Kubernetes AI scheduler (the GPU-Operator-adjacent batch scheduler); the HAMi community reports KAI-scheduler adopting HAMi-core as its GPU-sharing engine. ❓ Reported (project-hami.io blog headline; not independently confirmed on NVIDIA's repo in this pass — see §13).

**The pattern:** scheduler-level sharing maximizes flexibility and fleet utilization with **zero isolation** and is best paired with either trusted single-team fleets or a virtualization layer underneath. HAMi deliberately integrates *with* the schedulers (Volcano/Kueue/Koordinator/KAI) rather than replacing them — its scheduler extender handles device placement, and the batch schedulers handle gang/queue semantics. ✅ (README, project site).

### 8.4 The Kubernetes DRA (Dynamic Resource Allocation) wildcard ✅/⚠️

DRA (Kubernetes 1.26+, alpha; evolving through 2026) is the upstream replacement for the device-plugin extended-resource model: pods request *structured* resource claims, and a driver (e.g., the NVIDIA DRA driver for GPUs) fills them, enabling per-pod device configuration, sharing, and even network-attached resources. ✅ (Kubernetes docs; AWS EKS docs already recommend the NVIDIA DRA driver for K8s 1.34+). DRA does not by itself partition a GPU — it standardizes allocation plumbing — so sharing still needs a mechanism (MIG, time-slicing, or an interception layer like HAMi-core). The CNCF's own August 2026 blog asks "Does Kubernetes DRA Replace HAMi?" and the current answer in HAMi's materials is no: HAMi ships a DRA mode so its virtualization/scheduling rides on top of the new plumbing. ✅/⚠️ (see §5.4, §13 for what could not be fully verified about the DRA-mode roadmap).

---

## 9. Cloud-Vendor GPU Sharing (Condensed)

For teams buying managed Kubernetes, the cloud vendor often decides the sharing menu. Condensed and ⚠️-flagged where offerings shift quickly:

- **Google Kubernetes Engine (GKE)** — the most explicit menu. ✅ Verified (Google Cloud docs, updated August 2026): GKE offers **time-sharing GPUs** (multiple pods per GPU, Autopilot and Standard), **multi-instance GPUs (MIG)** (Autopilot and Standard, where the underlying machine supports it), and **NVIDIA MPS** (Standard clusters). GKE's GPU selection spans GB300/GB200/B200/H200/H100/A100 down to L4/T4 in Autopilot. Google's docs treat MIG as the isolation option and time-sharing as the utilization option — the same framing as this guide's §11.
- **Amazon EKS** — ✅ Verified (AWS docs, 2026): MIG is supported via the GPU Operator (label nodes to define MIG partition strategies) and via the k8s-device-plugin/DRA driver; AWS's current guidance recommends the **NVIDIA DRA driver** for GPU allocation and sharing on Kubernetes 1.34+ (Karpenter, managed, or self-managed nodes; not yet on EKS Auto Mode). AWS does not expose MIG on all GPU instances — availability depends on the instance type and hypervisor (Nitro instances generally expose MIG where the GPU supports it, e.g. p4d/p4de A100 and p5 H100 families; ⚠️ exact matrix changes; check instance docs). ⚠️
- **Microsoft Azure AKS** — ⚠️ MIG has historically been a *preview/limited* offering on AKS (MIG support on AKS is documented as preview on specific VM sizes, e.g. NC A100 v4-series); Azure's mainstream GPU story has been whole-GPU node pools with the GPU Operator. ⚠️ Offerings in this area have changed repeatedly; verify the current AKS preview list before design.
- **The takeaway** ⚠️: managed clouds give you MIG and/or time-slicing as *node-configuration* features, and — except for GKE's clean documentation — the exact surface (DRA vs device plugin vs MIG profiles per instance type) differs by vendor and changes year to year. None of the three majors ships a HAMi-equivalent *fractional memory* sharing layer as a managed feature; that remains the domain of the open-source middleware this guide is about, or of the vendor clouds' closed in-house stacks (cGPU/qGPU, §8.2).

---

## 10. Head-to-Head: HAMi vs the Alternatives

This is the section the whole guide has been building toward: the same physical NVIDIA GPU, six different ways to divide it. The comparison table is followed by the analysis that matters — because a table of features is not a decision, and the decision turns on *which axis you cannot afford to lose* (usually isolation, utilization, or ecosystem trust).

### 10.1 The big table

| Dimension | **HAMi** (software vGPU + dynamic MIG) | **NVIDIA MIG** | **NVIDIA time-slicing** | **NVIDIA MPS** | **vCUDA-style interception** (e.g. tkestack/vcuda-controller) | **Plain k8s-device-plugin** (no sharing config) |
|---|---|---|---|---|---|---|
| Isolation level | Software-enforced memory + compute caps in-container; fault domain shared with co-tenants ⚠️; hardware isolation only in MIG mode | Hardware: dedicated SMs, L2, memory slices, fault isolation | None (deliberate) — same memory, same fault domain | Classic: none between clients; plugin mode adds configurable per-workload partition | Software-enforced (CUDA/NVML interception), similar to HAMi | n/a (whole GPU per pod) |
| Memory partitioning | MB-granularity (e.g. `gpumem: 3000` = 3,000 MiB), software cap | Fixed profile table (e.g. `1g.5gb`…); hardware-dedicated | None — every replica sees full GPU memory | Classic: shared; pinned limits in plugin mode | MB-granularity software cap | None |
| Compute partitioning | Core/percentage caps + priority-based contention control (block/throttle) | Dedicated SM slices | None (interleaved time) | SM-percentage caps (plugin mode) / cooperative overlap | Compute-utilization caps | None |
| Scheduling model | Scheduler extender: binpack/spread/topology/dynamic MIG; integrates Volcano/Kueue/Koordinator/KAI | Node-level pre-partition (mig-parted) or dynamic creation; device-plugin resource per profile | Device-plugin replicas; kube-scheduler treats replicas as whole GPUs | Device-plugin MPS mode (mutually exclusive with time-slicing) | Device-plugin/controller + scheduler co-location | kube-scheduler; one pod = one whole GPU |
| Multi-tenant fit | Good for cooperative tenants with software trust boundary; audit via WebUI/metrics; not a hardware security boundary | Best — genuine hardware isolation, guaranteed QoS | Poor for untrusted tenants | Poor as isolation; good for one tenant's many processes | Good (with the same software-boundary caveat as HAMi) | Poor (fragmentation, one tenant per card) |
| Overhead | Low user-space interception overhead per CUDA call ⚠️ | Near-zero runtime overhead; setup/reboot cost at enable time | Minimal scheduling overhead; context-switch cost under contention | Reduced context-switch overhead vs raw sharing | Interception overhead similar to HAMi ⚠️ | None |
| Compatibility/ecosystem | Any CUDA app (interception is transparent); vLLM, Volcano, Kueue, Koordinator, KAI; coexists with GPU Operator | Any app that fits the profile; some peer-to-peer/NVLink limits; no consumer GPUs | Any CUDA app; zero code change | Multi-process apps, MPI-style; needs MPS-aware config | CUDA apps; project-dependent maturity | Any CUDA app |
| GPU support | NVIDIA (primary) + Ascend/Cambricon/Hygon/Iluvatar/Kunlunxin/MetaX/Moore Threads/Enflame/Biren etc.; AMD on roadmap | NVIDIA data-center only (A100/A30+, H100/H200, B200-class) | NVIDIA (any GPU the plugin sees) | NVIDIA | NVIDIA | NVIDIA |
| Governance/maintenance | CNCF Incubating (July 2026), Apache-2.0, multi-company maintainers incl. NVIDIA; very active (v2.10, Aug 2026); LFX health "Concerning" flag ⚠️ | NVIDIA product (docs + driver releases) | NVIDIA open source (k8s-device-plugin, Apache-2.0, very active) | NVIDIA product (driver component) | Varies wildly by repo: original research orphaned; tkestack/vcuda-controller maintained but smaller community ⚠️ | NVIDIA open source, ubiquitous default |
| Use-case fit | Fractional inference fleets, heterogeneous accelerator pools, dynamic MIG where needed, cost reduction at scale | Regulated / strict multi-tenant, QoS-guaranteed inference and training slices | Dev/test, batch utilization, trusted single-owner fleets | Many small processes of one workload (HPC ranks, model replicas) | Small/medium clusters wanting vCUDA-style control without CNCF governance | The status quo baseline |

### 10.2 The four axes that actually decide

**Axis 1 — Isolation strength vs. utilization ceiling.** MIG and HAMi sit at opposite ends of a spectrum, and the spectrum is *not* "good vs bad" — it is "how much do you trust your co-tenants?" MIG gives hardware-enforced isolation with coarse, fixed slices and real reconfiguration costs. HAMi gives software-enforced caps with surgical MB granularity — but its enforcement is an interception layer, which is a *trust boundary in user space*, not a hardware boundary. A pod that shares a card under HAMi shares the card's fault domain with its co-tenants (as does time-slicing; MIG does not). For a bank running one business unit's production model next to another unit's workload, that distinction is the whole ballgame — see §11.3 and §12.

**Axis 2 — Granularity vs. predictability.** If your workloads need 6 GB, MIG on an 80 GB A100 offers you coarse profile choices (roughly 5/10/20/40 GB-class slices, ⚠️ approximate per SKU); HAMi gives you exactly 6,000 MiB and packs the card to its last gigabyte. Predictability of *performance* is the mirror image: MIG's dedicated SMs guarantee that your 2 GB slice's throughput is not at the mercy of a neighbor's busy kernel; HAMi's software caps control memory and (with compute caps/priority throttling) contention *statistically*, not with hard SM separation.

**Axis 3 — Ecosystem and governance trust.** The NVIDIA stack is a vendor product: first-party docs, driver release trains, and a support contract path. HAMi is a CNCF-incubating community project: Apache-2.0, multi-company maintainers (including NVIDIA engineers), 17 releases in ~2 years, five CNCF case studies — but also a "Concerning" LFX health score, Asia-heavy maintainer gravity, and a young governance history relative to Kubernetes itself. For regulated procurement this is a *diligence difference, not a disqualifier*; the CNCF Sandbox→Incubating trajectory and the NVIDIA participation are precisely the signals an enterprise risk committee will want to see.

**Axis 4 — The heterogeneous hedge.** If your roadmap includes non-NVIDIA accelerators (Ascend, Cambricon, Hygon, Kunlunxin… — the de facto reality for many Asia-based financial institutions facing supply constraints), HAMi is currently the only project in this comparison with a *shipped* multi-vendor support matrix and one scheduling/allocation workflow across all of them. NVIDIA-native mechanisms obviously do not apply to non-NVIDIA silicon at all.

### 10.3 Two mechanisms, one deployment: HAMi's hybrid position

The strongest single insight from this research: **HAMi is not a competitor to MIG — it is an orchestrator that can use MIG when the workload demands hardware isolation and software vGPU when it demands density.** Its dynamic-MIG mode creates and allocates MIG instances on supported cards (with dedicated MIG end-to-end CI added in 2026), while its default mode hands out MB-grained software slices. That means a platform team can run one control plane and offer tenants a *menu*: "hardware-isolated MIG slice" for the regulated workload, "dense software slice" for the internal batch job. No other project in this comparison spans both. ✅ (README; §4.4.) This hybrid is the backbone of the Cymbal Bank recommendation in §12.

### 10.4 Five takeaways from the table

1. **Isolation is a ladder, not a switch.** From none (time-slicing) → software caps (HAMi/vCUDA-style) → configurable partition (plugin MPS mode) → hardware isolation (MIG), each rung costs density and buys guarantee. Choose the *lowest* rung that satisfies the requirement — not the highest one marketing makes you feel safe.
2. **HAMi and MIG answer different questions.** "How do I pack 6 GB workloads onto 80 GB cards?" → HAMi. "How do I prove tenant B cannot touch tenant A's memory?" → MIG. Asking one to do the other's job is where most failed GPU-sharing projects start.
3. **The vendor-native stack is the floor, not the ceiling.** k8s-device-plugin + GPU Operator is what everyone should run first; HAMi is an *addition* that the NVIDIA stack explicitly tolerates (coexistence is a documented integration), so the adoption risk is incremental rather than all-or-nothing.
4. **Every software mechanism shares a fault domain.** HAMi, vCUDA-style shims, MPS, and time-slicing all mean "if one co-tenant crashes the GPU context, everyone on the card notices." Only MIG contains faults. Operations plans must assume co-tenant blast radius wherever hardware isolation is absent.
5. **Governance is a selection criterion, not a footnote.** CNCF Incubating + Apache-2.0 + NVIDIA maintainer participation is a materially different procurement answer than "research project from 2009" or "closed vendor-cloud kernel module" — and the LFX health-score flag is a reminder that community health must be re-checked at decision time, not just cited once.

---

## 11. Decision Guidance: When to Use What

The decision procedure below is deliberately reductive — it encodes the analysis of §10 into four questions. Answer them in order.

### 11.1 Question 1: Do you need a *hardware* security boundary between tenants?

If yes — the tenants are untrusted relative to each other, a regulator or audit will ask "what happens if tenant A's workload is malicious or buggy?" and the answer must be "it cannot touch tenant B's memory or SMs" — then **the only defensible choice in this entire guide is MIG** (hardware partitioning), possibly delivered *through* HAMi's dynamic-MIG mode or the GPU Operator's mig-parted so the control plane stays sane. Software interception (HAMi vGPU mode, vCUDA-style, time-slicing, MPS) is a cooperative-tenancy mechanism: it enforces caps, it does not create a hardware fault domain. This is the regulated-banking answer, period. (§12 shows how Cymbal Bank threads this needle.)

### 11.2 Question 2: Is this a trusted, single-owner fleet where the goal is pure utilization?

If yes — one team's dev/batch/CI GPUs, no cross-tenant data sensitivity — then **time-slicing** (or plain MPS for many-small-process workloads) is the honest answer: near-zero operational cost, maximal flexibility, and no pretense of isolation. HAMi's software vGPU adds *memory-capping* value even here (time-slicing lets a runaway pod OOM the card for everyone; HAMi caps it), which is why GPU clouds serving many untrusted-ish customers prefer cap-enforcing middleware over raw time-slicing.

### 11.3 Question 3: Is this a cooperative-but-multi-tenant inference platform that needs *density*?

This is the classic Cymbal Bank shape (see §12): dozens of small LLM/embedding inference workloads, each 2–16 GB, owned by different business units that are *cooperative* (same bank, same legal entity, policy-controlled access) but not *co-located-by-default*. The engineering answer is **HAMi-style MB-granularity software sharing**: it turns an 80 GB card into a packable pool, caps every tenant's memory so one bad query storm cannot OOM the card, exposes per-container metrics for chargeback, and (via priority throttling) protects latency-critical serving from batch neighbors. The governance answer is a *policy overlay*: same-entity tenants, quota per business unit, no cross-unit data paths beyond the model, audit logs — because the software boundary is not a hardware boundary, the compensating controls live in policy and monitoring rather than in silicon. If any single tenant later demands hardware-grade separation (e.g., a third-party model or an outsourced workload), carve that one out to a MIG slice (possibly via HAMi dynamic MIG) or a dedicated node.

### 11.4 Question 4: What is your accelerator roadmap?

Heterogeneous (NVIDIA + any non-NVIDIA accelerator): HAMi is the only candidate with shipped multi-vendor support — the decision makes itself. NVIDIA-only and happy with the vendor stack: GPU Operator + device-plugin (whole GPU or MIG/time-slicing per §6) is the low-risk default; adopt HAMi only when you hit the density or dynamic-MIG requirements it uniquely serves.

### 11.5 A decision cheat-sheet

| Scenario | Recommended mechanism | Why (one line) |
|---|---|---|
| Regulated multi-tenant, untrusted tenants, audit demands hardware isolation | **MIG** (via GPU Operator mig-parted or HAMi dynamic MIG) | Only hardware partitioning gives a real fault/security boundary |
| Dozens of small inference workloads, cooperative tenants, density is the goal | **HAMi software vGPU** (+ MIG carve-outs for special tenants) | MB-granular packing + per-container caps + metrics, with a MIG escape hatch |
| Dev/test/batch on a trusted fleet, want maximum simplicity | **Time-slicing** | Zero setup, deliberate no-isolation is acceptable |
| One workload, many small processes (HPC ranks, model replicas) | **MPS** | Overlap without context-switch thrash |
| Only whole GPUs, everything vendor-supported and boring | **Plain k8s-device-plugin / GPU Operator** | The baseline; nothing to gain from complexity |
| Heterogeneous accelerator fleet (NVIDIA + Ascend/Cambricon/etc.) | **HAMi** | Only shipped multi-vendor sharing/scheduling workflow |

### 11.6 Operating and cost considerations

Beyond mechanism choice, shared-GPU operations change how the platform team runs day to day — these are the operational deltas that show up after the pilot, not before:

- **Chargeback changes granularity.** Whole-GPU billing was simple ("one card per team-hour"); fractional sharing needs per-container metrics to bill honestly. HAMi's Prometheus endpoint (default port 31993) and Grafana dashboards exist precisely for this; plan the finance pipeline that consumes them before rollout, or the first cross-unit billing dispute will be your pilot review. ✅ (§4.3)
- **Capacity planning becomes pack-shape-dependent.** A 30%-memory-utilized fleet may still *fragment*: if the 40 workloads are 6 GB each, 32× 80 GB cards pack beautifully; if a few jobs need 40 GB+, the pack leaves holes. Binpack policies consolidate, spread policies trade density for contention-resilience — choose per pool, and model the actual workload mix (the §12 worked example does this arithmetic explicitly).
- **Autoscaling and preemption interact with sharing.** Cluster autoscalers and batch schedulers (Volcano/Kueue) reason about *resource totals*; a sharing layer changes what "headroom" means (free MB on an existing card vs. a new card). HAMi's roadmap explicitly includes gang-scheduling, preemption, and autoscaling enhancements (✅ CNCF blog), signaling these gaps are known; test autoscaler behavior against fractional allocations before promising it.
- **Node lifecycle gets a new axis.** Unlabeled nodes bypass HAMi entirely (the README's `gpu=on` label model), which is the supported way to run a mixed fleet — shared serving nodes + whole-GPU training nodes + MIG-partitioned sensitive nodes in one cluster. Document the label schema as infrastructure-as-code, not tribal knowledge.
- **The cost case must be modeled, not asserted.** Sharing raises utilization but adds an operator layer and a small per-call interception cost; the honest ROI model compares *fleet cost per useful token/job* before and after, not "utilization went up." The Cymbal Bank example (§12.6) shows the density math that makes the case defensible to finance.

### 11.7 Anti-patterns (things this research corrected)

- **"MIG on our laptops/desktop GPUs"** — MIG is data-center-GPU-only (A100/A30+). ❌ corrected
- **"Time-slicing gives us isolation"** — the NVIDIA docs say the opposite in so many words; it shares memory and fault domain. ❌ corrected
- **"vCUDA is a production product we can adopt"** — the original is a 2009 research artifact with no maintained upstream; name reusers vary wildly in quality. ❌ corrected (⚠️ for the name-reusers)
- **"HAMi is a CNCF sandbox project"** — true until July 2026; it is now **Incubating**. ✅ corrected status
- **"HAMi supports AMD GPUs"** — AMD is on the roadmap; it is not in the shipped matrix as of this research pass. ❌ corrected
- **"MPS and MIG are the same thing"** — MPS multiplexes processes at the driver level; MIG partitions hardware. ❌ corrected

---

## 12. Worked Example: The Cymbal Bank Inference Platform Choice

> The fictional-persona convention of this repository applies: Cymbal Bank is the only bank in this narrative; real organizations appear only as public case-study citations (§5), never as personas.

### 12.1 The situation

The ML platform team at **Cymbal Bank** (Singapore) runs the bank's shared inference platform on a Kubernetes estate spanning two on-premises data centers (an OpenShift-based cluster, see [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) and [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) for the platform layer) with a burst option to a public cloud ([cloud_providers_guide.md](cloud_providers_guide.md)). The accelerator fleet is 32 physical NVIDIA GPUs — A100-80GB and H100-80GB cards — managed today as *whole GPUs*: each business unit's model gets an entire card, whether it needs it or not.

The workload reality, measured over a quarter:

- **~40 inference workloads**, most needing between 2 GB and 16 GB of GPU memory: embedding models for RAG pipelines, small fine-tuned LLMs for document Q&A and summarization, transaction-category classifiers, and the bank's internal copilot. Served through vLLM / Xinference / Ollama-class runtimes (see [ai_llm/ollama_xinference_localai_guide.md](ai_llm/ollama_xinference_localai_guide.md) for that layer; [gpu_optimization_guide.md](gpu_optimization_guide.md) for the performance-engineering layer that determines how much memory and compute each model actually needs).
- **A handful of fine-tuning jobs** (LoRA-style, 1–4 GPUs each) that run nightly and on weekends ([ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md) for the frameworks above this layer).
- **Measured fleet GPU-memory utilization: ~30%.** Most cards sit at 2–8 GB used out of 80 GB. The platform's GPUs are a visible P&L line, and the platform team has been told to serve 2× the workloads on the existing fleet before any new hardware request.

**Constraints that shape the decision:**

1. **Regulatory**: MAS technology-risk management and the bank's model-risk governance require demonstrable control over data access and isolation between business units. The audit function will ask, per co-tenant pair, "what is the isolation mechanism, and what happens on fault or misbehavior?"
2. **Cooperative tenancy**: all tenants are Cymbal Bank business units — the same legal entity, with policy-controlled access and no *intentional* cross-unit data flows. Tenants are cooperative but not interchangeable: a retail-loans model and a markets model must not share memory *by accident*.
3. **Heterogeneous hedge**: supply-chain experience has the architecture team evaluating non-NVIDIA accelerators for non-latency-critical workloads.
4. **Observability for chargeback**: business units are billed per GPU-hour; per-workload metrics are a finance requirement.

### 12.2 The options the team actually debated

| Option | Utilization upside | Isolation | Friction the team found |
|---|---|---|---|
| A. Status quo (whole GPU, plain device plugin) | None — the problem | Perfect by construction (one tenant per card) | Cost untenable; 70% idle |
| B. Time-slicing (device-plugin replicas) | High, trivial setup | **None** — docs say so; same memory + fault domain | Unacceptable for production serving customer data |
| C. Static MIG (mig-parted profiles on all cards) | Medium (coarse slices waste memory: an 8 GB need burns a ~10–20 GB-class slice) | Hardware-grade | Fixed profiles fight a workload mix of 2/4/6/8/16 GB needs; reconfiguration is disruptive |
| D. HAMi software vGPU everywhere | Highest (MB-granular packing to the last gigabyte) | Software caps; shared fault domain | Needs the policy overlay (§12.4) to satisfy audit |
| E. **HAMi + MIG carve-outs (recommended)** | High, with an isolation escape hatch | Software caps for cooperative tenants; hardware MIG slices where audit demands | Slightly more moving parts than D |

### 12.3 What the workloads actually need (the crux)

The team's mistake would have been to pick a mechanism before profiling the workloads ([gpu_optimization_guide.md](gpu_optimization_guide.md) §7's measure-first discipline). Profiling showed the fleet is dominated by *memory-bound, low-occupancy serving*: embeddings and small-LLM replicas use 2–16 GB of memory and 10–40% of one card's compute. These are the canonical HAMi workload: small memory footprints, latency-tolerant-ish, many replicas, cooperative owners. The fine-tuning jobs, by contrast, are whole-card or multi-card and want no neighbors at all.

**Latency-profile nuance.** The mix is not homogeneous: embedding workloads are high-QPS, low-latency-per-token, and mostly memory-bandwidth-bound; the small-LLM workloads are bursty and latency-sensitive at the p99 (interactive copilot users); the nightly fine-tuning is throughput-oriented and tolerant of hours-long runtimes. That ordering drives the *priority* design: interactive LLM serving gets the highest priority class (so HAMi's contention control throttles batch and embedding before it touches interactive traffic — §3.3), embedding runs in the middle, and fine-tuning is preemptible at the queue level via Volcano/Kueue. This is why the worked example recommends HAMi's per-container compute caps plus priority classes rather than a flat time-slicing replica count, which would treat all three shapes identically and let the nightly fine-tune degrade the interactive p99.

### 12.4 The recommendation: two-tier sharing with a policy overlay

**Tier 1 — HAMi software vGPU slices for the serving fleet (the default).** Deploy HAMi (Helm, per the README quick start: label nodes `gpu=on`, `helm install hami hami-charts/hami`) on the inference node pool. Each model deployment requests exactly what it needs:

```yaml
resources:
  limits:
    nvidia.com/gpu: 1        # one physical GPU, shared
    nvidia.com/gpumem: 8192  # 8 GiB cap for this embedding model
    # nvidia.com/gpucores: 40   # optional: cap compute envelope at ~40%
```

HAMi packs ~6–10 of the bank's typical 4–8 GB serving workloads per 80 GB card (utilization from ~30% toward 75–85% on the serving pool, in line with the density math DaoCloud and the China Merchants Bank case study report at fleet scale — ✅ cited from CNCF case studies, §5). Per-container Prometheus metrics feed the existing Grafana stack ([ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) for the observability platform context), which satisfies finance chargeback. The vLLM integration (memory caps per model server) is the documented, supported way to run multiple model replicas per card under HAMi (✅ README ecosystem table).

**Tier 2 — MIG carve-outs for the workloads audit will scrutinize.** Any model serving data whose co-tenancy an auditor would challenge — models consuming third-party data, models being retired/outsourced, or any cross-legal-entity pilot — goes onto **MIG slices** instead of software vGPU: either pre-partitioned nodes via the GPU Operator's mig-parted, or HAMi's dynamic-MIG mode so the slice is created on demand. Same HAMi control plane, hardware isolation underneath, and the pod manifests differ by one resource name (`nvidia.com/mig-*` profile vs `nvidia.com/gpumem`). This tier is small by design (a handful of cards) — it exists to make the *default* tier defensible.

**Tier 0 — whole-GPU for training.** Fine-tuning jobs keep requesting whole cards (`nvidia.com/gpu: 2`) on a training pool with no sharing configured. HAMi's node-level configuration makes this trivial: unlabeled nodes are simply not managed by HAMi (README: HAMi only manages nodes labeled `gpu=on`).

**Explicitly rejected:**

- *Raw time-slicing for production serving* — no memory caps; one runaway embedding job OOMs the card and takes every co-tenant down (same fault domain, per NVIDIA's own docs, §6.2). Acceptable only on the dev pool.
- *Static MIG everywhere* — the 2–16 GB workload mix does not fit coarse fixed profiles without wasting the exact resource the bank is short on; and re-partitioning cards for a changing model portfolio is operationally expensive.
- *vCUDA-original or lookalike shims* — unmaintained or unproven (⚠️ §8.1); HAMi is the maintained, CNCF-governed member of that interception family.
- *Expecting AMD/ROCm support from HAMi this year* — roadmap item only (✅ §2 claims table); the heterogeneous hedge is planned, not present.

### 12.5 The governance overlay that makes Tier 1 audit-ready

Because software vGPU is a *trust boundary in user space* (not a hardware one), Cymbal Bank's model-risk and security teams add compensating controls rather than pretending the mechanism is something it is not:

1. **Tenancy policy**: software-vGPU co-tenancy is permitted only between Cymbal Bank business units under the same legal entity; any external/third-party workload automatically routes to Tier 2 (MIG) or a dedicated node. Enforced in the platform's admission policy, not in the runtime.
2. **Quota and priority**: per-business-unit GPU quotas (HAMi integrates with Kueue/Volcano for queue semantics, §4.6) and latency-critical serving marked higher priority than batch, so HAMi's contention control blocks/throttles batch before serving (✅ repo-documented priority behavior, §3.3).
3. **Monitoring and audit evidence**: per-container memory/utilization metrics retained for chargeback and for the "who shared what with whom, when" questions auditors ask; HAMi-WebUI gives operators the allocation view.
4. **Incident carve-out**: the first time a co-tenant fault or contention event crosses a service-level threshold, the affected workload is moved to Tier 2 — the escape hatch exists and is rehearsed.
5. **Vendor-diligence file**: CNCF Incubating status (July 2026), Apache-2.0, multi-company maintainers including NVIDIA, the LFX "Concerning" health-score flag, and the ByteDance-origin question (⚠️ unverified) all documented in the procurement record — the same diligence discipline the vendor head-to-head siblings apply ([nutanix_enterprise_ai_vs_openshift_ai_guide.md](nutanix_enterprise_ai_vs_openshift_ai_guide.md)).

### 12.6 Expected outcome and rollback

Expected: 2× serving capacity on the existing fleet (the density math: 40 workloads × ~6 GB average ≈ 240 GB ≈ 3–4 cards of *capacity* if perfectly packed vs. 40 whole cards today), with production latency protected by priority throttling and the small MIG tier absorbing the genuinely sensitive workloads. Rollback is clean: HAMi is a control-plane addition — uninstall the Helm chart, remove labels, and pods fall back to whole-GPU scheduling (standard resources still resolve); no application code changed at any point (✅ the "zero application changes" property is a HAMi design goal, §2.1). The team re-runs the decision when Kubernetes DRA matures (§8.4): DRA may replace the *plumbing*, but the bank will still need a virtualization mechanism above it — and the same two-tier logic will apply.

### 12.7 Phased rollout plan

A regulated bank does not flip a production serving fleet to a new sharing layer in one weekend. The plan the Cymbal Bank platform team adopted:

- **Phase 0 — Lab proof (2 weeks).** Two labeled A100s in the non-production cluster; deploy HAMi via Helm; replay the three representative workload shapes (embedding, small-LLM serving, LoRA fine-tune) against software-vGPU and dynamic-MIG slices; capture before/after latency (p50/p99), throughput, and per-container metrics. Exit criterion: no regression beyond the noise band on any shape, and metrics visible in Grafana.
- **Phase 1 — Dev and test pools (2 weeks).** Move all dev/test GPU workloads onto HAMi-managed nodes with quotas via the existing Kueue queueing ([ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md) for the platform context). This is where the team learns the operational deltas from §11.6 — chargeback data, contention behavior, node-label hygiene — without production risk.
- **Phase 2 — Non-sensitive production serving (4 weeks).** Migrate serving workloads whose data is *not* customer-regulated (internal tooling, public-data models) to Tier 1 software-vGPU slices. Monitor contention-throttling events (HAMi's priority mechanism, §3.3) and set the incident carve-out trigger. Exit criterion: two weeks without a co-tenant incident and chargeback reports accepted by finance.
- **Phase 3 — Sensitive serving with MIG carve-outs (4 weeks).** Stand up the Tier 2 MIG nodes (GPU Operator mig-parted profiles; HAMi dynamic-MIG evaluated side by side), migrate the customer-data workloads onto hardware-isolated slices, and document the per-workload isolation mechanism in the audit register.
- **Phase 4 — Fine-tuning pool and heterogeneous evaluation.** Keep training whole-GPU; begin the non-NVIDIA accelerator evaluation on HAMi's supported matrix (§4.5) for the planned heterogeneous hedge.

The phases deliberately sequence *trust* before *density*: the bank proves the software boundary on its own non-sensitive workloads before any auditor-relevant co-tenancy exists — and by Phase 3, the isolation-sensitive workloads have already been routed to hardware slices, so the software tier never holds the bank's hardest requirement.

---

## 13. What Could Not Be Verified

Honesty list — every fact this guide could not confirm from a primary source during the September 2026 research pass. Do not repeat any of these as established fact.

1. **ByteDance as HAMi's originator** ⚠️ — widely repeated in community write-ups and consistent with the `k8s-vGPU-scheduler` lineage, but no primary source examined (GitHub README, CNCF project page, CNCF incubating blog) names ByteDance as the creator. ByteDance appears only as a contributor/adopter logo on project-hami.io. The task brief's assumption that this is established fact was *not* confirmed.
2. **NVIDIA KAI-scheduler adopting HAMi-core** ❓ — reported in a project-hami.io blog headline ("HAMi-core Adopted by NVIDIA KAI Scheduler"); not independently confirmed against NVIDIA's KAI-scheduler repository within this research pass.
3. **Original vCUDA repository status** ⚠️ — `github.com/chenzhao/vcuda` (the URL in the research brief) returns 404 (September 2026); the original project is verified as IPDPS-2009 research via the paper, but no maintained official code repository was located, so "unmaintained since ~2019–2020" is an inference from absence, not a verified date.
4. **cGPU (Alibaba) and qGPU (Tencent) internals and current availability** ❓ — described in vendor docs and third-party comparisons only; not independently verified; treated as closed-source vendor-cloud features.
5. **Exact MIG profile tables and per-SKU instance maxima** ⚠️ — "supported GPUs are A100/A30 and later data-center parts" is ✅ verified, but exact slice profiles and instance counts per SKU were not all re-derived from the current MIG user guide; treat as architecture-dependent.
6. **Azure AKS MIG offering currency** ⚠️ — historically preview/limited on specific VM sizes; not re-verified against current AKS documentation in this pass.
7. **AWS MIG availability per EC2 instance family** ⚠️ — EKS-level MIG support (GPU Operator, DRA driver guidance) is ✅ verified from AWS docs; the per-instance-type exposure matrix changes frequently and was not exhaustively checked.
8. **HAMi DRA-mode maturity in v2.10** ⚠️ — existence of a DRA mode is verified (project site tabs, release-train commit history), but its production maturity/feature parity with the device-plugin mode could not be confirmed; commit history in the 2.10 cycle shows the DRA charts being reworked.
9. **HAMi's historical MPS-mode support in current releases** ⚠️ — earlier HAMi documentation described MPS as an orchestrated mode; the current README's scheduling list emphasizes binpack/spread/topology/dynamic-MIG and does not surface MPS mode, so its status in v2.10 was left unstated rather than guessed.
10. **CUDA-interception overhead numbers** ⚠️ — HAMi ships a `benchmarks/` directory but no independently reproducible overhead figures were verified here; per-CUDA-call interception overhead is real but unquantified in this guide.
11. **Project website metrics (471+ contributors, 25 countries, 306k Docker pulls, adopter logos)** ⚠️ — self-reported by the project; GitHub/CNCF numbers used in §2.5 are the stronger evidence; the CNCF LFX health score (38, "Concerning") is a point-in-time snapshot observed during research.
12. **Koordinator's CNCF maturity stage at time of writing** ⚠️ — trajectory described without pinning a precise stage; verify at koordinator.sh before citing.
13. **"First commit March 12, 2018"** ⚠️ — as reported by CNCF project insights; the current repository's LICENSE dates to July 2021 and the HAMi rename to January 2024; the 2018 date plausibly reflects predecessor-project history but was not independently confirmed.
14. **China Merchants Bank case-study details beyond the published claims** ⚠️ — "100% hardware-pool utilization through topology-aware scheduling" is the CNCF case-study headline claim (July 2, 2026), cited as *reported*, not independently audited.

---

## 14. Glossary and References

### Glossary

| Term | Meaning |
|---|---|
| **HAMi** | Heterogeneous AI Computing Virtualization Middleware — CNCF-incubating Kubernetes middleware for GPU/accelerator sharing, isolation, and scheduling (formerly `k8s-vGPU-scheduler`). |
| **HAMi-core** (a.k.a. `libvgpu`) | HAMi's in-container virtualization library; for NVIDIA it intercepts the CUDA driver API to enforce per-workload memory and compute limits. |
| **vGPU** | Overloaded term: (1) HAMi's software-sliced virtual GPU; (2) NVIDIA's licensed VM virtualization product (formerly GRID); (3) assorted research/commercial slices. Always ask which one. |
| **MIG** | Multi-Instance GPU — NVIDIA hardware partitioning of a data-center GPU into isolated instances with dedicated SMs, L2, and memory slices. |
| **Time-slicing** | Device-plugin mechanism that advertises one GPU as N replicas sharing the whole device in time; deliberately no isolation. |
| **MPS** | Multi-Process Service — NVIDIA runtime (control daemon + server + client) letting many CUDA processes overlap execution on one GPU. |
| **Device plugin** | Kubernetes mechanism (kubelet API) by which a vendor registers hardware (e.g. `nvidia.com/gpu`) and allocates it to containers. |
| **Scheduler extender** | A secondary scheduler consulted by kube-scheduler for filter/score/bind on device-specific placement (how HAMi schedules). |
| **Extended resources** | Kubernetes resources beyond CPU/memory advertised by device plugins (e.g. `nvidia.com/gpu`, `nvidia.com/gpumem`). |
| **DRA** | Dynamic Resource Allocation — Kubernetes' evolving structured-resource model (claims/drivers) intended to eventually supersede device-plugin extended resources. |
| **CDI** | Container Device Interface — standard for injecting devices (GPUs) into containers via OCI hooks. |
| **binpack / spread** | Scheduling policies: consolidate onto fewest devices/nodes vs. distribute to reduce contention. |
| **Topology-aware scheduling** | Placement that respects hardware topology (e.g. NVLink domains) when choosing devices. |
| **Gang scheduling** | Scheduling a group of pods (a "gang") only when all can be placed — needed for multi-GPU training jobs. |
| **Dynamic MIG** | Creating and allocating MIG instances at runtime (HAMi mode) rather than statically pre-partitioning nodes. |
| **mig-parted** | NVIDIA tool that applies static MIG profile configurations to GPUs at node start; used by the GPU Operator. |
| **GPU Operator** | NVIDIA Kubernetes operator automating drivers, container toolkit, device plugin, GFD labeling, and DCGM monitoring. |
| **DCGM** | NVIDIA Data Center GPU Manager — the metrics/monitoring framework for data-center GPUs. |
| **NVML** | NVIDIA Management Library — the API for querying/controlling GPU state (what `nvidia-smi` uses). |
| **CUDA API interception** | Technique of interposing on CUDA driver calls (e.g. via preloaded shim libraries) to virtualize/enforce limits — the basis of HAMi-core and vCUDA-style projects. |
| **Fault domain** | The set of components that fail together; sharing a fault domain means a co-tenant crash can take you down. |
| **QoS** | Quality of Service — guaranteed performance/isolation level; MIG provides it, software sharing approximates it. |
| **CNCF Sandbox / Incubating / Graduated** | CNCF project maturity stages: early experimentation → production-adoption evidence → broad production use. HAMi: Sandbox (Aug 2024) → Incubating (July 2026). |
| **LFX Insights** | Linux Foundation analytics platform (contributor counts, health scores) used for CNCF project metrics. |
| **ROCm** | AMD's GPU compute stack; HAMi's AMD support (Mi-series) is a roadmap item, not shipped as of September 2026. |
| **OOM** | Out-of-memory — on a shared GPU, a tenant exceeding its cap can fail the allocation or (without caps) destabilize co-tenants. |

### References and Companion Guides

### Primary sources — HAMi

- HAMi GitHub repository: [github.com/Project-HAMi/HAMi](https://github.com/Project-HAMi/HAMi) (README, releases, commit history — verified September 2026; v2.10.0, Aug 21, 2026)
- HAMi-core (interception library): [github.com/Project-HAMi/HAMi-core](https://github.com/Project-HAMi/HAMi-core)
- HAMi documentation and website: [project-hami.io](https://project-hami.io/) — including the device support matrix and "GPU Virtualization Principles"
- HAMi-WebUI: [github.com/Project-HAMi/HAMi-WebUI](https://github.com/Project-HAMi/HAMi-WebUI)

### Primary sources — CNCF

- CNCF project page: [cncf.io/projects/hami](https://cncf.io/projects/hami/) (Sandbox Aug 21, 2024; Incubating July 2, 2026)
- "HAMi becomes a CNCF incubating project," CNCF blog, July 15, 2026: [cncf.io/blog/2026/07/15/hami-becomes-a-cncf-incubating-project](https://www.cncf.io/blog/2026/07/15/hami-becomes-a-cncf-incubating-project/)
- "Does Kubernetes DRA Replace HAMi?," CNCF blog, Aug 7, 2026
- CNCF case studies: China Merchants Bank (July 2, 2026), DaoCloud, SNOW Corp. — via the HAMi CNCF page

### Primary sources — NVIDIA

- MIG User Guide (supported GPUs, concepts): [docs.nvidia.com/datacenter/tesla/mig-user-guide](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/)
- MPS documentation: [docs.nvidia.com/deploy/mps](https://docs.nvidia.com/deploy/mps/index.html)
- k8s-device-plugin (time-slicing, MPS mode, MIG strategies): [github.com/NVIDIA/k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)
- GPU Operator docs: [docs.nvidia.com/datacenter/cloud-native/gpu-operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html); repo: [github.com/NVIDIA/gpu-operator](https://github.com/NVIDIA/gpu-operator)

### Other projects and cloud docs

- vCUDA paper (IPDPS 2009): Shi, Chen, Sun — "vCUDA: GPU Accelerated High Performance Computing in Virtual Machines" (aimlab.org archive)
- tkestack/vcuda-controller (name-reuse caution): [github.com/tkestack/vcuda-controller](https://github.com/tkestack/vcuda-controller)
- AliyunContainerService/gpushare-scheduler-extender: [github.com/AliyunContainerService/gpushare-scheduler-extender](https://github.com/AliyunContainerService/gpushare-scheduler-extender)
- Volcano: [volcano.sh](https://volcano.sh/); Koordinator: [koordinator.sh](https://koordinator.sh/); KAI-scheduler: [github.com/NVIDIA/kai-scheduler](https://github.com/NVIDIA/kai-scheduler)
- GKE GPUs (time-sharing, MIG, MPS): [cloud.google.com/kubernetes-engine/docs/concepts/gpus](https://cloud.google.com/kubernetes-engine/docs/concepts/gpus)
- Amazon EKS NVIDIA GPU management (MIG, DRA driver): [docs.aws.amazon.com/eks/latest/userguide/device-management-nvidia.html](https://docs.aws.amazon.com/eks/latest/userguide/device-management-nvidia.html)

### Companion guides in this repository

- GPU *performance engineering* (hardware fundamentals, profiling, kernels — the layer below sharing): [gpu_optimization_guide.md](gpu_optimization_guide.md)
- Inference serving runtimes the shared GPUs host: [ai_llm/ollama_xinference_localai_guide.md](ai_llm/ollama_xinference_localai_guide.md)
- ML platforms and frameworks above the sharing layer: [ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md), [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md)
- Vendor head-to-head genre precedents: [nutanix_enterprise_ai_vs_openshift_ai_guide.md](nutanix_enterprise_ai_vs_openshift_ai_guide.md), [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md), [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md)
- Cluster and deployment mechanics: [devstack_openstack_guide.md](devstack_openstack_guide.md), [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md), [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md), [cloud_providers_guide.md](cloud_providers_guide.md)

### Keeping this guide current

This guide's facts were verified in September 2026, and the GPU-sharing landscape moves quickly on four clocks: **NVIDIA driver/CUDA releases** (MIG profiles and MPS controls change per generation), **Kubernetes DRA** (its graduation path may rewrite the device-plugin layer this guide describes), **HAMi releases** (v2.10.0 at research time; the roadmap adds AMD Mi-series, gang scheduling, and preemption), and **cloud-vendor menus** (GKE/AKS/EKS sharing features shift yearly — §9 is deliberately ⚠️-flagged). Before making a procurement or architecture decision, re-verify against the primary sources in this section; the HAMi status markers to re-check are its CNCF maturity level, current release, the device-support matrix, and the LFX health score. Everything marked ⚠️ or ❓ in this guide is a re-verification candidate by design, not an oversight.

**The bottom line.** MIG is the isolation answer, time-slicing is the simplicity answer, MPS is the many-processes answer, and HAMi is the density-and-heterogeneity answer — and because HAMi can also orchestrate MIG, a platform team is not forced to choose once. Profile your workloads, decide which tenants need a hardware boundary and which only need a cap, then pick the mechanism per tier rather than per fleet. That is how a bank like Cymbal Bank turns a 30%-utilized GPU estate into a defensible, auditable, shared platform — doing more useful work, safely, on the shared GPU.
