# NVIDIA DCGM: Telemetry, Health and Diagnostics for a GPU Fleet

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Systems / AI Infrastructure — GPU Fleet Observability, Health Monitoring, Diagnostics, Failure Taxonomy, Prometheus/Kubernetes Integration, Banking Technology  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** September 2026

---
> A deep-dive into **NVIDIA Data Center GPU Manager (DCGM)** — the telemetry, health and diagnostics layer that GPU fleets are actually run on. The guide settles the comparison most often muddled (NVML vs `nvidia-smi` vs DCGM), then works through DCGM's architecture, its metric surface with exact field identifiers, the passive health model, the active diagnostic suites with their runtimes and blast radius, the XID failure taxonomy, the policy engine, and the Kubernetes/Prometheus integration via DCGM Exporter and the GPU Operator. It closes with the operational practice of watching a fleet — what to alert on, what a "utilisation" number really means, the regulated-enterprise evidence question — and a worked Cymbal Bank example. Every metric identifier, diagnostic level, XID code and licence term here was read from NVIDIA's own documentation or repository files on **19 September 2026**; identifiers are quoted, never reconstructed.

**Audience**: Platform and infrastructure engineers running NVIDIA GPU fleets, SREs who own GPU nodes in Kubernetes, and solution architects who must sign off on what a GPU estate will and will not prove. Assumes comfort with Linux, containers, Prometheus and Kubernetes; no CUDA programming required. GPU *performance engineering* lives in [gpu_optimization_guide.md](gpu_optimization_guide.md); the *sharing and virtualization* layer lives in [hami_gpu_sharing_guide.md](hami_gpu_sharing_guide.md).

**What this guide deliberately will not do**: it will not re-derive GPU sharing, MIG partitioning or time-slicing semantics (that is [hami_gpu_sharing_guide.md](hami_gpu_sharing_guide.md)'s subject — DCGM is only a consumer of MIG entity IDs here); it will not re-derive performance optimisation or profiling methodology ([gpu_optimization_guide.md](gpu_optimization_guide.md)); it will not re-derive the platform stacks that consume DCGM as a component ([openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md), [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md), [ai_llm/scalable_ai_deployment.md](ai_llm/scalable_ai_deployment.md)); and it will not re-derive general monitoring or incident discipline, which the repository's observability and operational-resilience content owns. It also will not claim DCGM is the only way to observe a GPU fleet — where vendor-agnostic approaches exist, they are named. What it *will* do is give you the exact strings, the exact levels, the exact codes, and the operational reasoning to run a fleet on them. ⚠️ Where a fact could not be verified against NVIDIA's own material it is flagged inline and collected in §16.

---

### Table of Contents

1. [DCGM Identity: The One-Paragraph Version and the Decoder](#1-dcgm-identity-the-one-paragraph-version-and-the-decoder)
2. [The Layering: NVML, nvidia-smi and DCGM](#2-the-layering-nvml-nvidia-smi-and-dcgm)
3. [Architecture and Components](#3-architecture-and-components)
4. [The Metric Surface: Fields, Groups and Sampling Cost](#4-the-metric-surface-fields-groups-and-sampling-cost)
5. [Health Monitoring: The Passive Model](#5-health-monitoring-the-passive-model)
6. [Diagnostics: Levels, Runtimes and Blast Radius](#6-diagnostics-levels-runtimes-and-blast-radius)
7. [The XID Failure Taxonomy](#7-the-xid-failure-taxonomy)
8. [The Policy Engine and Automated Response](#8-the-policy-engine-and-automated-response)
9. [Kubernetes, Prometheus and the Exporter](#9-kubernetes-prometheus-and-the-exporter)
10. [GPU Fleet Observability in Practice](#10-gpu-fleet-observability-in-practice)
11. [The Economics and the Capacity Angle](#11-the-economics-and-the-capacity-angle)
12. [The Regulated-Enterprise Angle](#12-the-regulated-enterprise-angle)
13. [Worked Example: The Cymbal Bank Shared GPU Platform](#13-worked-example-the-cymbal-bank-shared-gpu-platform)
14. [Gotchas and Anti-Patterns](#14-gotchas-and-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified](#16-what-could-not-be-verified)
17. [Glossary, Cross-References and Closing Summary](#17-glossary-cross-references-and-closing-summary)

---

### How to Read This Guide

**Verification convention.** Facts were researched on **19 September 2026** against NVIDIA's primary documentation: the DCGM documentation set at `docs.nvidia.com/datacenter/dcgm/latest/` (whose release-notes page opens at **DCGM 4.7.0**), the XID Errors documentation at `docs.nvidia.com/deploy/xid-errors/`, the GPU Operator documentation at version **26.7**, and the `NVIDIA/DCGM` and `NVIDIA/dcgm-exporter` repositories. Markers: ✅ **Verified** (read from a primary source during this research), ⚠️ **Caution** (documented but version- or hardware-dependent, or a clearly-labelled operational inference), ❓ **Unverified** (could not be confirmed; treat as a research task, not a fact).

**A note on the documentation set.** NVIDIA reorganised the DCGM documentation into an "About / Installation / Learn / Reference / Release Notes" structure whose *Learn* area is organised by **module** — Core Services plus ten loadable modules (NVSwitch and ConnectX, vGPU, Introspection, Health, Policy, Configuration, Diagnostics, Profiling, CPU Monitoring, Multi-Node Diagnostics). Older material and third-party tutorials still describe the pre-reorganisation layout ("DCGM User Guide", "API Reference", "Field Group reference"). Section names differ; field IDs and command syntax are the same, and where an older name is aliased (e.g. NVVS for the diagnostic engine) this guide says so.

**Relationship to sibling guides.** This guide owns *DCGM itself*. [hami_gpu_sharing_guide.md](hami_gpu_sharing_guide.md) owns sharing and its "one monitoring plane" recommendation — this is the missing explanation of what that plane is. [gpu_optimization_guide.md](gpu_optimization_guide.md) owns application-side profiling; DCGM's profiling fields are the fleet-level, interval-averaged complement (§4.5). The platform guides consume the exporter as a stack line; this is the layer beneath it. [chaos_engineering_guide.md](chaos_engineering_guide.md) and [capacity_sizing_guide.md](capacity_sizing_guide.md) own fault-injection campaigns and fleet sizing respectively.

**Suggested reading paths.** Standing up GPU nodes: §1 → §2 → §3 → §5 → §6. Just paged on a GPU estate: §5 → §6 → §7 → §10 → §14. Designing the monitoring plane: §4 → §9 → §11 → §12 → §13. Wanting the bottom line: §1 → §2 → §10 → §11 → §13.

---
## 1. DCGM Identity: The One-Paragraph Version and the Decoder

**In one paragraph.** NVIDIA Data Center GPU Manager is a **host-level daemon and client framework** that turns the GPUs on a machine into a queryable, watchable, checkable set of *entities* and *fields*. It runs a persistent process (the *host engine*, `nv-hostengine`) that discovers the NVIDIA hardware in the node — GPUs, MIG GPU instances, MIG compute instances, NVSwitches, NVLink links, vGPUs, ConnectX devices, and supported NVIDIA CPUs and cores — holds a cache of sampled telemetry with timestamps and statuses, and exposes that model through a CLI (`dcgmi`), C/C++/Python/Go APIs, an optional Prometheus exporter, and *modules* implementing health, diagnostics, policy, configuration, profiling and workload accounting. NVIDIA describes it as "a host-level framework for inventory, telemetry, health, policy, configuration, diagnostics, profiling, and workload accounting on NVIDIA data-center hardware", and in its own repository as "a suite of tools for managing and monitoring NVIDIA datacenter GPUs in cluster environments" including "active health monitoring, comprehensive diagnostics, system alerts and governance policies including power and clock management". ✅ Both quoted statements verified 19 Sep 2026.

**The problem it solves that ad-hoc inspection does not.** Every operator starts with `nvidia-smi` over SSH, and for one node that is genuinely sufficient. It stops being sufficient at the second node — not because `nvidia-smi` gives wrong answers, but because per-node inspection has **no memory and no aggregate**:

- **No history.** A GPU that throttled for forty seconds during last night's training run, or that logged a row-remap event at 03:14, looks perfectly healthy when you look in the morning. DCGM's watch model retains timestamped samples in the host engine for a bounded window; the exporter turns that into a durable time series.
- **No policy, no callback.** `nvidia-smi` has no notion of "tell me when this numeric field crosses this threshold". DCGM's policy module evaluates a field with an operator and a threshold, retains violations, and can notify a listener — which is what makes monitoring an integration rather than a human habit.
- **No shared, typed model.** A shell script scraping `nvidia-smi` text produces untyped, unversioned strings per script author. DCGM exposes *fields* with published IDs, units, entity levels, scopes, value types and counter semantics, shared verbatim across the CLI, the language bindings and the C/C++ API — so a dashboard built by one team and an exporter configured by another refer to the same measurement by the same identifier.
- **No active testing.** Retained telemetry tells you a device *has been* behaving; it cannot tell you whether it *can* execute a workload. The diagnostic module applies real load and compares against thresholds — a different question (§6).
- **No fleet surface.** One host engine answers for one host, but the model is uniform: N nodes produce N identical `/metrics` endpoints for one Prometheus and one Grafana. That uniformity is the whole trick.

**The framing that matters.** DCGM is the layer that turns a GPU fleet from *a set of machines you can log into* into *a measurable system*. Once adopted, every fleet question becomes a query rather than a tour: how many devices are degraded, which card model thermal-throttles under the training workload, whether last Tuesday's tail-latency regression coincided with a link error on one node. None of those are answerable by looking at one machine; all are answerable by watching all of them the same way.

### 1.1 The decoder: vocabulary a platform engineer must hold

| Term | What it actually is | Where it lives |
|---|---|---|
| **Host engine** | The long-lived DCGM process that discovers entities, collects fields, caches samples, and owns all module state. Clients connect to it; it is the node's shared management plane. | `nv-hostengine`; on packaged installs the `nvidia-dcgm.service` unit |
| **`dcgmi`** | The command-line client. Sends requests to a host engine over localhost, TCP, Unix socket or VSOCK. Not a daemon; holds no state beyond a connection. | Packaged install |
| **DCGM Exporter** | A separate Go program that reads DCGM telemetry and publishes Prometheus text at `/metrics`. Its own repository, image and release train — not part of the DCGM package. | `github.com/NVIDIA/dcgm-exporter`; port **9400** |
| **Entity** | A managed object: GPU, GPU instance (MIG), compute instance (MIG), vGPU, NVSwitch, link, CPU, CPU core, ConnectX. Entity *families* are distinct types, not just IDs. | `dcgmi discovery --list` |
| **Field** | One item of telemetry or state with a numeric ID and metadata (entity level, scope, value type, semantics). The `DCGM_FI_*` name is the C header symbol; interfaces such as `dcgmi dmon` accept the **numeric value**. | `dcgmi dmon --list`; the vendor's *Field Identifiers* reference |
| **Field group** | A named, reusable list of field IDs in the host engine. Selects *columns*. Does not start collection, set cadence, choose retention, or validate device support. | `dcgmi fieldgroup --create … --fieldids …` |
| **Entity group** | A named, reusable set of entities (numeric persistent groups plus built-in aliases). Selects *rows*. Creating one does not imply any field is watched. | `dcgmi group --create … --add gpu:0,gpu:2` |
| **Watch** | The operation combining entities, fields, an update interval and a retention limit — and therefore the operation that costs resources. Samples are *shared host-engine state*. | Field APIs; `dcgmi dmon` creates connection-scoped watches |
| **Telemetry metric** | A field *as exposed to a monitoring system* — normally the exporter's Prometheus metric family, named after the configured DCGM field. | `DCGM_FI_DEV_GPU_TEMP` as a Prometheus gauge |
| **Health check** | Passive assessment: a watch over selected *systems* whose retained samples are evaluated into incidents with severity `Healthy`/`Warning`/`Failure`. Applies **no workload**. | `dcgmi health --set …`, `dcgmi health --check` |
| **Diagnostic level** | The numbered active-test suite: `1` quick, `2` medium, `3` long, `4` extended, with documented aliases and runtimes. Applies a real workload. | `dcgmi diag --run <n>` |
| **Policy** | A condition on a numeric field (operator plus threshold, or the changed-value operator) that DCGM evaluates and retains as a violation, with console/file/callback notification. It does **not** remediate. | `dcgmi policy --create …` |
| **XID** | An error report *from the NVIDIA driver* written to the OS kernel or event log. Not a DCGM concept; DCGM reports the last one it saw as a field and uses XIDs as health inputs. | OS logs (`grep "NVRM: Xid"`); `docs.nvidia.com/deploy/xid-errors/` |
| **NVML** | NVIDIA Management Library — the low-level per-device management API. Its relationship to `nvidia-smi` and DCGM is §2. | `docs.nvidia.com/deploy/nvml-api/` |
| **GPU Operator** | A Kubernetes operator automating the whole NVIDIA node software stack — drivers, container toolkit, device plugin, GFD labelling, DCGM-based monitoring. In practice the recommended way to deploy DCGM and the exporter on Kubernetes. | `docs.nvidia.com/datacenter/cloud-native/gpu-operator/`; doc set 26.7 ✅ |
| **Module** | DCGM's internal feature boundary. Core Services is module ID 0, always loaded; ten further modules are loadable and can be denylisted. | `dcgmi modules --list`; `nv-hostengine --denylist-modules` |

### 1.2 The boundary with sibling guides

| Question | Owner | Why |
|---|---|---|
| How do I split one GPU between tenants (MIG, time-slicing, MPS, HAMi)? | [hami_gpu_sharing_guide.md](hami_gpu_sharing_guide.md) | Sharing semantics and isolation guarantees; DCGM only *observes* the resulting instance IDs and per-tenant allocation |
| Why is my kernel slow, and how do I profile it? | [gpu_optimization_guide.md](gpu_optimization_guide.md) | Application profiling with Nsight and CUDA tooling; DCGM's profiling fields are the fleet-level complement (§4.5) |
| What should the monitoring platform be, and how are alerts routed? | The repository's observability and operational-resilience content | General monitoring discipline, alert routing, ITSM integration |
| What does DCGM expose, how is it deployed, what is healthy, what is failing, and what do I do? | **This guide** | §2–§14 |
| How big should the GPU estate be, and what does it cost? | [capacity_sizing_guide.md](capacity_sizing_guide.md) (sizing) and §11 here (the *measurement* input) | Sizing maths belongs there; what DCGM can and cannot tell you about utilisation belongs here |

**The one-sentence version.** DCGM is NVIDIA's host-level telemetry, health, diagnostics and policy framework for datacenter GPUs — the layer between the driver stack and your monitoring system.

---
## 2. The Layering: NVML, nvidia-smi and DCGM

Three things get conflated here, and they sit in a strict stack: a **library**, a **command-line tool built on that library**, and a **framework that adds state, health and fleet semantics on top of low-level device management**.

**NVML (NVIDIA Management Library).** NVIDIA's low-level management and monitoring API for NVIDIA devices: per-device queries and settings — power, clocks, temperature, memory, ECC counters, topology, XID state — exposed as a C library that ships with the driver. It is stateless and per-call: you ask for a value, you get a value. It is documented separately in NVIDIA's *NVML API* reference, which the DCGM documentation links as a related reference for field semantics. ✅ DCGM's own description of its position: "DCGM uses the low-level NVIDIA management software appropriate to each device and presents the results through one host-level model."

**`nvidia-smi`.** The command-line program that **installs with the NVIDIA driver** and "reports basic monitoring and configuration data about each GPU in the system" — NVIDIA's own description, from the XID Errors documentation. ✅ Its purpose is *point-in-time, local, human-readable inspection and administration*: `nvidia-smi`, `-q`, `topo -m`, `nvlink`, `mig -lgip`. It is the right tool for "what is this machine doing right now" and for one-off administrative actions. It is not a collector, has no watch model, and retains no history. NVIDIA's XID documentation cites it as one of the tools that helps interpret XID conditions — it "can list ECC error counts (Xid 48), indicate if a power cable is unplugged (Xid 54), or provide any applicable GPU Recovery Action (Xid 154)". ✅

**DCGM.** Sits *above* the low-level management layer and adds what a fleet needs: a persistent host engine, an entity/field model shared across interfaces, watches with cadence and retention, passive health evaluation, active diagnostics, policy conditions, low-overhead hardware-counter profiling, workload accounting, and group objects. Its documentation is explicit about what it does **not** displace: "Drivers and platform services still control the hardware. On NVSwitch systems, Fabric Manager remains responsible for fabric configuration and lifecycle." ✅ DCGM does not become the driver; it becomes the *observer and validator* above it.

### 2.1 The decision table

| You need to… | Reach for | Why not the others |
|---|---|---|
| Check one node's GPU state interactively, right now | `nvidia-smi` | Zero setup, ships with the driver, human-legible |
| Change a device setting on one node (persistence mode, power limit, clock) | `nvidia-smi`, or `dcgmi config` for the configuration module's supported settings | Direct and immediate; no daemon required |
| List MIG profiles and instance layout to *configure* MIG | `nvidia-smi mig -lgip` / `-cgi` | MIG *configuration* is a driver/device-plugin concern; DCGM *observes* the instances |
| Get a single value into a script once | `nvidia-smi --query-gpu=… --format=csv`, or an NVML call | No daemon, no state, no auth surface |
| Continuously collect GPU telemetry from many nodes into a time series | **DCGM Exporter → Prometheus** | `nvidia-smi` has no collector or history; NVML has no exporter or fleet model |
| Evaluate a fleet against health rules and get `Warning`/`Failure` per device | **DCGM health** | `nvidia-smi` has no health model; NVML exposes raw values only |
| Prove a device *can execute a workload*, not merely that it *has been reporting* | **`dcgmi diag`** | Nothing else in the stack applies a validated test workload with pass/fail semantics |
| Attribute GPU activity to a process or job | **`dcgmi stats`** (process watches and job records) | `nvidia-smi` shows current processes but keeps no job records |
| Understand which kernel in a running application is slow | Nsight Systems / Nsight Compute | DCGM profiling fields are interval averages, not kernel traces (§4.5) |

### 2.2 The four operational distinctions that change design

**(a) What can be queried cheaply.** One raw device value through NVML or `nvidia-smi` is cheap — one call to an already-loaded driver component. But "cheap" is not "free at fleet scale": the cost that matters is N nodes × M devices × K fields × S samples per second, plus storage. DCGM's own guidance is explicit: "Sharing does not make telemetry free. Resource use can grow when an administrator watches more entity-field combinations, requests shorter intervals, retains samples for longer, subscribes more clients to updates, or transfers larger result sets." ✅

**(b) What requires a daemon.** `nvidia-smi` requires nothing. DCGM's health, policy, diagnostics, profiling and accounting features require the **host engine to be running**, because they are stateful. Two consequences: `nvidia-dcgm.service` becomes part of the node's observability critical path; and "host-engine groups, watches, samples, and job records are runtime state and normally do not survive an engine restart" ✅ — a restart resets watches and health configuration. On Kubernetes with the GPU Operator the exporter pod owns its own DCGM process, which moves where that state lives (§9.1).

**(c) What is available inside a container.** `nvidia-smi` is normally usable inside an NVIDIA container because the NVIDIA Container Toolkit injects the driver's user-mode libraries and device nodes, and DCGM's *device telemetry* is likewise reachable — which is why the exporter can run as a pod. But the parts of DCGM touching privileged, host-scoped state (diagnostics, some configuration, NVSwitch/NVLink fabric paths, `nv-hostengine` as a host service) depend on device nodes, libraries, permissions and — for fabric — matching host packages. NVIDIA names them: on NVSwitch systems install **Fabric Manager**, plus **NSCQ** (Hopper and earlier) or **NVSDM** (Blackwell and later), and "Fabric Manager, NSCQ, and NVSDM package major versions must match the NVIDIA driver major version" — driver 565 implies `libnvidia-nscq-565` and `libnvsdm-565`. ✅ This is the most common cause of "DCGM works on bare metal but half the metrics are blank in Kubernetes".

**(d) Which of them can tell you a device is FAILING rather than merely BUSY.** The distinction that separates a dashboard from an operations capability, and the clearest argument for DCGM:

| Question | `nvidia-smi` | NVML | DCGM |
|---|---|---|---|
| Is the GPU busy? | ✅ (utilisation, memory) | ✅ | ✅ `DCGM_FI_DEV_GPU_UTIL`, `DCGM_FI_DEV_FB_USED` |
| Has it been busy consistently for the last hour? | ❌ no history | ❌ (you would poll and store it) | ✅ retained samples, watches with cadence and retention |
| Is it running hot, or throttling? | ✅ instantaneous | ✅ | ✅ plus throttle-time counters (`DCGM_FI_DEV_THERMAL_VIOLATION`, `…_THERMAL_THROTTLE_SECONDS_TOTAL`) |
| Is it accumulating correctable ECC errors? | ✅ counters | ✅ | ✅ `DCGM_FI_DEV_ECC_SBE_VOL_TOTAL`, `…_SBE_AGG_TOTAL`, row-remap fields |
| **Is it currently degraded or failed, by a defined rule, now?** | ❌ no health model | ❌ raw values only | ✅ `dcgmi health --check` → `Healthy`/`Warning`/`Failure` with `DCGM_FR_*` incidents |
| **Can it execute a known workload?** | ❌ | ❌ | ✅ `dcgmi diag --run 1..4`, `Pass`/`Fail`/`Skip` per test and entity |

Read that as a layering statement, not a competition. `nvidia-smi` and NVML expose *facts*; DCGM turns facts into *state over time* and *judgements* — and the judgement is what alerting and drain decisions need. A card reporting 41 °C and 0 % utilisation is "not busy". A card that accumulated 4 seconds of thermal-throttle time in the last hour under a workload that previously throttled for none, with a `Warning` from the memory watch, is **suspect**. Only DCGM of the three can make the second statement.

### 2.3 What DCGM adds — and what it does not cover

| Capability | In `nvidia-smi`/NVML? | Added by DCGM |
|---|---|---|
| Typed field catalogue with stable numeric IDs | partial (query names, no shared field model) | ✅ |
| Persistent entity/field groups; watches with cadence and bounded retention | ❌ | ✅ |
| Passive health rules with severity and `DCGM_FR_*` incident codes | ❌ | ✅ |
| Active diagnostics with numeric run levels; field-based policy | ❌ | ✅ |
| Interval-averaged profiling metrics; process and job accounting | ❌ | ✅ |
| Remote access to one host's model (TCP / Unix socket / VSOCK) | ❌ | ✅ |
| A uniform Prometheus surface across a fleet | ❌ | ✅ via DCGM Exporter |
| Multi-node coordinated diagnostics | ❌ | ✅ optional package, `dcgmi mndiag` |

⚠️ **Caution on scope.** All of the above is *NVIDIA-specific hardware observability*. If the estate contains AMD Instinct, Intel Gaudi or other accelerators, DCGM will not see them, and golden dashboards must either be built on vendor-neutral signals (node exporter plus scheduler and application metrics) per accelerator type, or be explicitly per-vendor. This is a decision made once; §12 returns to it as a vendor-concentration question.

❓ **Refused.** This guide does not assert *which* driver interfaces DCGM compiles against for each device family beyond NVIDIA's own statement that it "uses the low-level NVIDIA management software appropriate to each device". The claim that "DCGM is just a wrapper around NVML" is directionally true for GPU fields and wrong for the fabric, CPU, vGPU and profiling paths; the vendor publishes no one-to-one mapping, so none is stated here.

---
## 3. Architecture and Components

### 3.1 The pieces

| Component | Installed item | What it does | Notes that bite |
|---|---|---|---|
| DCGM shared library | `libdcgm.so.4` | Client-side API library; every interface talks through it | Version-coupled to the host engine and modules |
| DCGM modules | Loadable feature-module libraries | Implement non-core feature areas (health, policy, diagnostics, profiling, configuration, NVSwitch/ConnectX, vGPU, introspection, CPU monitoring, multi-node diagnostics) | Lazy-loaded on first use; can be denylisted at runtime or at host-engine start |
| Diagnostic plugins | Plugin libraries plus supporting executables | The individual active tests (`software`, `memory`, `pcie`, …) | Need an executable path and a plugin directory readable by the diagnostic account (`NVVS_BIN_PATH`, `NVVS_PLUGIN_DIR`) |
| NVIDIA Host Engine | `nv-hostengine` | Persistent server owning entities, fields, watches, samples, groups, module state | The daemon; everything stateful depends on it |
| systemd service | `nvidia-dcgm.service` | Brings the host engine up at boot on packaged installs | Environment set for the *service* governs client behaviour such as `CUDA_VISIBLE_DEVICES` |
| Command-line interface | `dcgmi` | The client | Connects to `localhost` by default |
| Profiling test utility | `dcgmproftester11` / `12` / `13` | Active load generator for the profiling measurement path | Suffix tracks the installed CUDA-major package |
| Python bindings | `/usr/share/datacenter-gpu-manager-4/bindings/python3/` | Programmatic access from Python | Installed with the runtime package |
| Multi-node diagnostics | Optional `datacenter-gpu-manager-4-multinode-cuda<major>` | Coordinated cross-host diagnostics | CUDA 12 and later only |

✅ Read from the DCGM documentation's *Install DCGM* page (Installed Components) and the *Command Line* reference, 19 Sep 2026.

### 3.2 Package structure, and why it matters

| Package | Contents |
|---|---|
| `datacenter-gpu-manager-4-core` | CUDA-independent runtime components. Both the CUDA runtime package and the development package require the **same version** of this |
| `datacenter-gpu-manager-4-cuda<major>` | CUDA-major-specific runtime components (e.g. `-cuda12`). Requires `-core` |
| `datacenter-gpu-manager-4-proprietary*` | "The subset of DCGM functionality that is not included in the DCGM open-source project". Installed as a *recommendation* — which is why the documented install commands enable package recommendations |
| `datacenter-gpu-manager-4-multinode-cuda<major>` | Optional multi-node diagnostic components (CUDA 12+) |
| `datacenter-gpu-manager-4-dev` / `-devel` | Headers under `/usr/include/datacenter-gpu-manager-4/`, the unversioned `libdcgm.so` linker name, CMake files including the `DCGM::dcgm` imported target, stub library `libdcgm_stub.a`, SDK samples |

Two consequences. First, **the open-source repository is not the whole product**: `github.com/NVIDIA/DCGM` is Apache-2.0 and builds the public library, CLI and modules, while the packaged `-proprietary*` components carry NVIDIA product terms and may be absent from a purely source-built install. Second, **packaging is CUDA-major-aware** — you select `-cuda12` or `-cuda13` from the driver's reported CUDA version, and NVIDIA documents a specific trap: on Maxwell, Pascal and Volta systems with driver 580, set `CUDA_VERSION=12`, because CUDA Toolkit 13.0 does not support those GPU generations even though the driver exposes the CUDA 13.0 user-mode API. ✅

### 3.3 Deployment models: embedded vs standalone, single-node vs fleet

**Embedded host engine.** A client can create and own an in-process host engine instead of connecting to the daemon — "choose standalone or embedded host-engine operation". Its state (watches, caches, module state) lives and dies with *your* process and is shared with nobody. Right for a self-contained collector; wrong for a node-wide management plane.

**Standalone host engine (the daemon).** `nv-hostengine` as a service is the fleet model and "a long-lived host engine as a shared management plane". Its value is *shared collection*: "The host engine tracks watchers independently but collects one stream for a given entity-field pair. If several clients request that same pair, DCGM uses the fastest requested interval and the longest requested retention needed by the active watchers." ✅ That is the architectural heart of the design and the reason two monitoring integrations on one node do not double collection cost.

| Node shape | What runs where | Why choose it |
|---|---|---|
| Single node, packaged | `nvidia-dcgm.service` + `dcgmi` + optional `dcgm-exporter` as a service or container | Bare metal or VMs with GPU passthrough; the reference deployment in NVIDIA's admin guide |
| Kubernetes node, Operator-managed | GPU Operator deploys driver, container toolkit, device plugin, GFD, **DCGM and DCGM Exporter** as one managed stack | The recommended path on Kubernetes; the exporter arrives as a DaemonSet with a Service and (by default) a ServiceMonitor |
| Kubernetes node, exporter only | Hand-installed DCGM plus the exporter DaemonSet | When something else owns the driver; the exporter README nudges you back to the Operator |
| Air-gapped / hardened | Packages staged from the NVIDIA repository, images mirrored internally | Financial-services norm; the Operator's chart and images have different licences (§15) |
| Multi-node fabric validation | `dcgmi mndiag` with the optional multinode package | Rack- or cluster-level NVLink/fabric validation; DCGM 4.7 documents host-file input and batched fan-out for as many as 576 hosts and 9,216 entities |

### 3.4 Privileges, drivers and containers — the practical concerns

| Concern | The position, from the documentation |
|---|---|
| **Root / privileges** | Package installation and system-service changes require root. Diagnostics have their own requirements: the diagnostic service account must reach device nodes, libraries, plugin files and external test executables |
| **Driver** | DCGM is tested with **NVIDIA Datacenter Drivers**; other driver types can omit functionality. Documented baseline: CUDA 7.5+ and driver **R450** or later. NVSwitch systems additionally need Fabric Manager and NSCQ/NVSDM at matching major version |
| **Host resources** | Minimum host RAM **≥ 16 GB**; minimum CPU cores **≥ number of GPUs** |
| **Platforms** | All Kepler (K80) and newer datacenter GPUs; NVSwitch on DGX A100 / HGX A100 and newer; all Maxwell and newer non-datacenter GPUs including GeForce and Quadro; bare metal and VMs with full GPU passthrough. On non-datacenter GPUs some functionality is unavailable — notably **policy notifications are "Not supported"** and single-node diagnostics are **Level 1 only** unless a higher level is explicitly documented for the product |
| **Remote access** | `dcgmi` targets `localhost`, a hostname, `host:5555`, `unix:///run/nvidia-dcgm/nv-hostengine.sock`, or `vsock://<cid>:<port>`. NVIDIA's own note: "A TCP address can cross a host boundary and is subject to routing and firewall policy; a Unix-socket path is host-local and relies on directory and socket permissions." For a same-host collector, a Unix socket or localhost is both faster and a smaller attack surface |
| **Systemd environment** | Environment changes are process-local. Setting `CUDA_VISIBLE_DEVICES` in the `dcgmi` client does **not** select the GPUs DCGM uses — "Set it for `nv-hostengine` before the host engine starts, or use the entity and group selectors of the requested command" |
| **Modules** | Deny at runtime (`dcgmi modules --denylist Policy`) or at startup (`nv-hostengine --denylist-modules 3,4`). Module ID 0 (Core) cannot be denylisted. Denylisting prevents loading but does not unload a resident module. The NVSwitch module is the special case: it can load *automatically* when supported hardware is present, so startup denylisting is the only reliable exclusion |

**⚠️ The container gotcha, stated once.** A DCGM Exporter pod that starts and scrapes proves the *telemetry* path works. It does not prove the *diagnostic*, *fabric* or *configuration* paths work, because those need host packages (Fabric Manager, NSCQ/NVSDM), privileges, and — for diagnostics — a plugin directory and executable path inside whatever environment runs `nvvs`. Teams discover this the first time they try `dcgmi diag` from inside the cluster rather than on the node.

---
## 4. The Metric Surface: Fields, Groups and Sampling Cost

### 4.1 What a field is, formally

| Metadata | Meaning | Why it changes your code |
|---|---|---|
| **Entity level** | The family for which the value is meaningful — GPU, MIG GPU instance, MIG compute instance, vGPU, NVSwitch, link, CPU, CPU core, ConnectX | Asking a GPU-scoped field of a MIG instance returns a blank, not an error you will notice |
| **Scope** | Whether the value belongs to an entity or is global to the host engine | Host-scoped values do not fan out per device |
| **Value type** | Integer, floating-point, string, timestamp, or binary | Determines how to test the vendor's *blank sentinel*: `DCGM_FT_INT64` applies to fields 1600–1611 and 1625–1649, `DCGM_FT_DOUBLE` to 1612–1624 and 1650 |
| **Semantics** | Attribute, instantaneous gauge, cumulative counter, event, or other | Determines whether you average it, difference it, or treat it as a state change |

A *field sample* is the tuple `(entity family, entity ID, field ID, timestamp, value, status)`, where the timestamp is the collection time — not the time your client read it — and the status distinguishes a usable value from "not supported", "not found", "no permission" or "not yet available". ✅

**Finding the right field, in order of authority.** `dcgmi dmon --list` enumerates the fields known to the *installed client release* with numeric ID, short column name and description. It is a metadata catalogue, not a hardware probe: a listed field may still be unavailable for a particular entity family, GPU generation, driver or platform; a *failed watch request is a command failure*, while an *accepted watch can still produce `N/A`* for a particular row. For meaning, units, entity support and hardware requirements, use the vendor's **Field Identifiers** reference. NVIDIA's own rule is the right one: "Do not infer units or counter behavior from a short `dmon` column name." ✅

### 4.2 Exact identifiers by category

Every identifier below was read from NVIDIA's *Field Identifiers* reference, its *DCGM Exporter Metrics* reference, or the released `default-counters.csv`, on **19 September 2026**, against the DCGM documentation set whose release notes open at **DCGM 4.7.0**. Where the exporter's configured name differs from the canonical field name, both are given — that divergence is itself an operational hazard (§14).

| Category | Identifier(s) | ID(s) | Notes from the source |
|---|---|---|---|
| Clocks | `DCGM_FI_DEV_SM_CLOCK`, `DCGM_FI_DEV_MEM_CLOCK`, `DCGM_FI_DEV_VIDEO_CLOCK` | 100, 101, 102 | MHz. Hertz variants: `…_SM_CLOCK_HERTZ` (1603), `…_MEMORY_CLOCK_HERTZ` (1604), `…_VIDEO_CLOCK_HERTZ` (1605) |
| Clocks | `DCGM_FI_DEV_APP_SM_CLOCK`, `DCGM_FI_DEV_MAX_SM_CLOCK` | 110, 113 | Application clock target and maximum supported, MHz. Hertz variants 1606 and 1608 |
| Thermal | `DCGM_FI_DEV_GPU_TEMP_CELSIUS` | 150 | The vendor's own worked example: "the definition `DCGM_FI_DEV_GPU_TEMP_CELSIUS 150` identifies field `150`" |
| Thermal | `DCGM_FI_DEV_MEMORY_TEMP_CELSIUS`, `DCGM_FI_DEV_FAN_SPEED` | —, 191 | Memory temperature, °C; fan percentage (ratio variant `DCGM_FI_DEV_FAN_SPEED_RATIO`, 1612, DOUBLE, can exceed 1.0). Exporter's configured name for the former is `DCGM_FI_DEV_MEMORY_TEMP` |
| Power and energy | `DCGM_FI_DEV_BOARD_POWER_WATTS`, `DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION` | —, 156 | Canonical field behind the exporter's configured `DCGM_FI_DEV_POWER_USAGE` (field 155 in DCGM's own field-group example). Energy is cumulative mJ since driver reload; joule variant `DCGM_FI_DEV_GPU_ENERGY_JOULES_TOTAL` (1611, INT64) |
| Utilisation | `DCGM_FI_DEV_GPU_UTIL` | 203 | Integer percentage 0–100 — this ID "remains" `DCGM_FI_DEV_GPU_UTIL` in 4.7.0 |
| Utilisation | `DCGM_FI_DEV_GPU_UTIL_RATIO` | 1613 | The *same measurement* as a `DCGM_FT_DOUBLE` ratio 0.0–1.0. NVIDIA's explicit warning: "Code that changes IDs must also change its value-type and scaling assumptions." |
| Utilisation | `DCGM_FI_DEV_MEM_COPY_UTIL`, `DCGM_FI_DEV_ENC_UTIL`, `DCGM_FI_DEV_DEC_UTIL` | 204, 206, 207 | Integer percent; ratio variants `DCGM_FI_DEV_MEMORY_UTIL_RATIO` (1614), `…_ENC_UTIL_RATIO` (1615), `…_DEC_UTIL_RATIO` (1616) |
| Framebuffer | `DCGM_FI_DEV_FB_TOTAL`, `DCGM_FI_DEV_FB_FREE`, `DCGM_FI_DEV_FB_USED`, `DCGM_FI_DEV_FB_RESERVED` | 250, 251, 252, 253 | Whole MiB. Byte-exact variants `…_FB_CAPACITY_BYTES` (1625), `…_FB_FREE_BYTES` (1626), `…_FB_USED_BYTES` (1627), `…_FB_RESERVED_BYTES` (1628), all INT64. `FB_RESERVED` was added to distinguish real usage from reserved memory |
| BAR1 | `DCGM_FI_DEV_BAR1_TOTAL`, `…_USED`, `…_FREE` | 90, 92, 93 | Whole MiB; byte variants 1600, 1601, 1602 |
| ECC | `DCGM_FI_DEV_ECC_SBE_VOL_TOTAL`, `…_DBE_VOL_TOTAL`, `…_SBE_AGG_TOTAL`, `…_DBE_AGG_TOTAL` | 310, 311, 312, 313 | Volatile and aggregate single/double-bit totals. Also `DCGM_FI_DEV_ECC_MODE` (300) and `DCGM_FI_DEV_ECC_PENDING` (301) |
| Row remap | `DCGM_FI_DEV_ROW_REMAP_UNCORRECTABLE_TOTAL`, `…_CORRECTABLE_TOTAL`, `…_FAILED`, `…_PENDING` | 393, 394, 395, 396 | `…_FAILED` is the vendor's own warning flag that remapping has failed |
| Page retirement | `DCGM_FI_DEV_PAGE_RETIRED_SBE_TOTAL`, `DCGM_FI_DEV_PAGE_RETIRED_DBE_TOTAL`, `DCGM_FI_DEV_PAGE_RETIRED_PENDING` | 390, 391, 392 | Canonical names; the exporter CSV ships the same three rows renamed `DCGM_FI_DEV_RETIRED_SBE`, `…_DBE`, `…_PENDING`, all **commented out** |
| PCIe | `DCGM_FI_DEV_PCIE_REPLAY_TOTAL` | 202 | Canonical field behind the exporter's configured `DCGM_FI_DEV_PCIE_REPLAY_COUNTER` ("Total number of PCIe retries") |
| NVLink | `DCGM_FI_DEV_NVLINK_THROUGHPUT_TOTAL` | 449 | Canonical field behind the exporter's configured `DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL` ("Aggregate NVLink throughput across all lanes, MB/s"). Reports MB/s derived by ÷1,000 from the KiB/s sample — not the ×1,024 byte fields below |
| NVLink | `DCGM_FI_DEV_NVLINK_THROUGHPUT_L0_BYTES_PER_SECOND` … `_L17_…` | 1630–1647 | Base-unit per-lane; for link *n* the ID is `1630 + n`. Converted ×1,024 from KiB/s — deliberately *different* from the legacy per-lane fields' ÷1,000 conversion, which is why the docs warn not to derive one from the other. Aggregate: `DCGM_FI_DEV_NVLINK_THROUGHPUT_BYTES_PER_SECOND` (1648) |
| NVLink | `DCGM_FI_DEV_NVLINK_CRC_ERROR_TOTAL` | 497 | Hopper-generation data-link CRC counter (commented in the exporter CSV). Also commented there: `DCGM_FI_DEV_NVLINK_RECOVERY_TOTAL`, `DCGM_FI_DEV_NVLINK_REPLAY_TOTAL` and the Blackwell+ `…_RECOVERY_SUCCESSFUL_TOTAL` / `…_RECOVERY_FAILED_TOTAL` / `…_RECOVERY_EVENT_TOTAL` — **IDs not read**; do not infer them from 497 |
| NVLink | `DCGM_FI_DEV_NVLINK_LINK_STATUS` | — | Canonical alias for the deprecated `DCGM_FI_DEV_NVLINK_GET_STATE` |
| C2C | `DCGM_FI_DEV_C2C_MAX_BANDWIDTH`, `…_BYTES_PER_SECOND` | 287, 1629 | MB/s; byte-exact INT64 variant |
| Throttle time | `DCGM_FI_DEV_POWER_VIOLATION`, `…_THERMAL_VIOLATION` | 240, 241 | Legacy: integer **nanoseconds** of constrained time |
| Throttle time | `DCGM_FI_DEV_GPU_POWER_THROTTLE_SECONDS_TOTAL`, `…_THERMAL_…_SECONDS_TOTAL` | 1617, 1618 | DOUBLE seconds; the base-unit pairs for 240 and 241 |
| Throttle time | `…_SYNC_BOOST_…` (1619), `…_LIMIT_…` (1620), `…_LOW_UTIL_…` (1621), `…_RELIABILITY_…` (1622), `…_APP_…` (1623), `…_BASE_…` (1624) | 1619–1624 | All DOUBLE seconds; legacy nanosecond counterparts are 242–247 |
| Errors | `DCGM_FI_DEV_XID_ERROR` | 230 | "Value of the **last** XID error encountered" — a gauge of the last one, not a counter of all |
| vGPU | `DCGM_FI_DEV_VGPU_LICENSE_STATUS`, `DCGM_FI_DEV_VGPU_MEMORY_USAGE` | —, 525 | Licence status; usage in MiB (byte variant `DCGM_FI_DEV_VGPU_FB_USED_BYTES`, 1649) |
| Identity | `DCGM_FI_SYSTEM_DRIVER_VERSION` | — | Canonical field behind the exporter's `DCGM_FI_DRIVER_VERSION` **label** |
| Sentinel | `DCGM_FI_MAX_FIELDS` | 1651 | One greater than the maximum allocatable field id (`1650 + 1`) |

### 4.3 The exporter's shipped default set, and why "20 metrics" is a trap

The exporter publishes only the fields selected by its collector configuration — a CSV, a ConfigMap, or YAML. The shipped default collector CSV contained, on 19 Sep 2026, **26 active rows**: 25 metric families plus one `label` row (`DCGM_FI_DRIVER_VERSION`), with everything else present but commented out as examples. The vendor's *DCGM Exporter Metrics* reference and the raw `etc/default-counters.csv` were both read and agree on this count. ✅

| Group | Active default rows |
|---|---|
| Clocks | `DCGM_FI_DEV_SM_CLOCK`, `DCGM_FI_DEV_MEM_CLOCK` |
| Temperature | `DCGM_FI_DEV_GPU_TEMP`, `DCGM_FI_DEV_MEMORY_TEMP` |
| Power and energy | `DCGM_FI_DEV_POWER_USAGE`, `DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION` |
| Utilisation | `DCGM_FI_DEV_GPU_UTIL`, `DCGM_FI_DEV_MEM_COPY_UTIL`, `DCGM_FI_DEV_ENC_UTIL`, `DCGM_FI_DEV_DEC_UTIL` |
| Memory | `DCGM_FI_DEV_FB_FREE`, `DCGM_FI_DEV_FB_USED`, `DCGM_FI_DEV_FB_RESERVED` |
| PCIe | `DCGM_FI_DEV_PCIE_REPLAY_COUNTER`, `DCGM_FI_PROF_PCIE_TX_BYTES`, `DCGM_FI_PROF_PCIE_RX_BYTES` |
| Errors | `DCGM_FI_DEV_XID_ERRORS` (field 230) |
| Remapping | `DCGM_FI_DEV_UNCORRECTABLE_REMAPPED_ROWS`, `DCGM_FI_DEV_CORRECTABLE_REMAPPED_ROWS`, `DCGM_FI_DEV_ROW_REMAP_FAILURE` |
| NVLink, vGPU | `DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL`; `DCGM_FI_DEV_VGPU_LICENSE_STATUS` |
| Profiling | `DCGM_FI_PROF_GR_ENGINE_ACTIVE`, `DCGM_FI_PROF_PIPE_TENSOR_ACTIVE`, `DCGM_FI_PROF_DRAM_ACTIVE` |
| Label | `DCGM_FI_DRIVER_VERSION` |

Two consequences, both operational:

1. **The default set is a *telemetry* set, not a *health* set.** It contains no ECC counters, no throttle-time counters, no NVLink error counters, and no retired-page detail — those are all shipped **commented out**. A team deploying the exporter with defaults and calling it "GPU monitoring" has GPU *utilisation* monitoring. Adding health-relevant rows is a deliberate act: a custom CSV, a YAML `metrics.fields` list, or the Helm `customMetrics` value. ✅
2. **The exporter owns metric families DCGM does not.** The collector CSV documents `DCGM_EXP_CLOCK_EVENTS_COUNT`, `DCGM_EXP_CLOCK_EVENTS_TOTAL`, `DCGM_EXP_XID_ERRORS_COUNT`, `DCGM_EXP_XID_ERRORS_TOTAL`, `DCGM_EXP_GPU_HEALTH_STATUS` and `DCGM_EXP_P2P_STATUS`. `DCGM_EXP_XID_ERRORS_TOTAL` exists precisely because `DCGM_FI_DEV_XID_ERROR` is a last-value gauge you can miss between scrapes. ✅

### 4.4 Field groups, and what they do not do

```bash
dcgmi fieldgroup --create gpu-environment --fieldids 150,155
dcgmi fieldgroup --info --fieldgroup <field-group-id>
dcgmi fieldgroup --delete --fieldgroup <field-group-id>
```

The documentation is unusually firm, and getting this right saves a class of confused dashboards: "A field group does not start collection, and an entity group does not imply that fields are watched. The watch is the operation that combines them." ✅ Field-group names and IDs are unique within one host engine; user-created groups survive the `dcgmi` process but not the host engine; internal groups can be listed but not deleted. Deleting a group "prevents future use of that selector" but does not necessarily stop watches another client or module already owns. The mental model the docs draw:

```
                     field 150    field 155    field ...
                   ┌────────────┬────────────┬───────────
GPU 0              │  watched   │  watched   │
GPU 1              │  watched   │  watched   │
entity ...         │            │            │
entity group ──────> rows
field group  ──────> columns
watch request ─────> selected cells + cadence + retention
```

### 4.5 Profiling fields: the interval-averaged layer

The profiling module (module ID 8) exposes hardware-counter metrics answering *how much work is actually happening*, as opposed to *whether the device is allocated*. They are interval averages, not kernel traces, and NVIDIA is direct about the implication: they "do not identify which source line, CUDA kernel, or instruction caused a value. Start with DCGM to locate an underused or imbalanced device, then use a developer profiler such as Nsight Systems or Nsight Compute when the investigation needs application-level attribution." ✅

| Metric | Field | ID | Definition (abridged from the source) |
|---|---|---|---|
| Graphics Engine Activity | `DCGM_FI_PROF_GR_ENGINE_UTIL_RATIO` | 1001 | Fraction of time any portion of the graphics or compute engines were active; an interval average, not instantaneous |
| SM Activity | `DCGM_FI_PROF_SM_UTIL_RATIO` | 1002 | Fraction of time at least one warp was active on an SM, averaged over all SMs. "A value of 0.8 or greater is necessary, but not sufficient, for effective use of the GPU. A value less than 0.5 likely indicates ineffective GPU usage." |
| SM Occupancy | `DCGM_FI_PROF_SM_OCCUPANCY_RATIO` | 1003 | Fraction of resident warps relative to the maximum concurrent warps per SM. "Higher occupancy does not necessarily indicate better GPU usage." |
| Tensor Activity | `DCGM_FI_PROF_TENSOR_UTIL_RATIO` | 1004 | Fraction of cycles the tensor (HMMA/IMMA) pipe was active; 1.0 means a tensor instruction every other cycle for the whole interval |
| FP64 / FP32 Activity | `DCGM_FI_PROF_FP64_UTIL_RATIO`, `DCGM_FI_PROF_FP32_UTIL_RATIO` | 1006, 1007 | Fraction of cycles the FP64 pipe, and the FMA (FP32 and integer) pipe, were active |
| DRAM Activity | `DCGM_FI_PROF_DRAM_UTIL_RATIO` | — | Fraction of cycles the device memory interface was active sending or receiving data |
| PCIe throughput | `DCGM_FI_PROF_PCIE_TX_BYTES`, `DCGM_FI_PROF_PCIE_RX_BYTES` | — | Rate over the PCIe bus including protocol headers and payloads, bytes/s |

Three rules stop these being misused: **discover, do not assume** (`dcgmi profile --list --entity-id gpu:0` asks the target GPU for its runtime catalogue; platform support is datacenter products from Volta onward on Linux x86_64, Arm64 and POWER ✅); **one number never proves a bottleneck** (the docs' own example — high SM activity alone does not prove a compute or memory bound; compare several related metrics over a representative interval and correlate with application throughput); and **profiling counters are a shared resource** (`dcgmi profile --pause` before a developer profiler takes ownership, `dcgmi profile --resume` after — the pause is **host-engine-wide**, not scoped to one GPU or client, and while paused profiling watches return blank values; a team that forgets `--resume` loses profiling telemetry fleet-wide).

The exporter also aliases profiling names, another place a dashboard silently breaks: configured `DCGM_FI_PROF_GR_ENGINE_ACTIVE` corresponds to canonical `DCGM_FI_PROF_GR_ENGINE_UTIL_RATIO`, `DCGM_FI_PROF_PIPE_TENSOR_ACTIVE` to `DCGM_FI_PROF_TENSOR_UTIL_RATIO`, and `DCGM_FI_PROF_DRAM_ACTIVE` to `DCGM_FI_PROF_DRAM_UTIL_RATIO`. ✅ **Boundary with [gpu_optimization_guide.md](gpu_optimization_guide.md):** "why is this kernel slow" belongs to the application profilers; "which of my 200 nodes is doing real work, and which is allocated but idle" is a question only DCGM scales to (§11).

### 4.6 Sampling cost, cardinality, and where the monitoring bill comes from

**Cost model, as documented.** Five levers increase resource consumption: watching **more entity-field combinations**, **shorter intervals**, **longer retention**, **more subscribed clients**, and **larger result sets**. ✅ On top of that, "provider and hardware limits can also make a requested interval impractical or enforce a minimum" — a cadence you can ask for is not automatically one the hardware will honour.

**The three clocks, which are separate and must be reconciled:**

| Control | What it governs | Documented example/default |
|---|---|---|
| Exporter collection interval / YAML watch-group interval | How often DCGM refreshes cached samples | Collector configuration; watch groups vary cadence per field set. Exporter `--interval` default `30000` **ms** |
| Prometheus `scrape_interval` | How often Prometheus reads `/metrics` | Example configuration uses **30 s** |
| Prometheus `scrape_timeout` | One read's budget | Must stay below both `scrape_interval` and the exporter's `--web-write-timeout`; example 25 s |
| Exporter `--web-write-timeout` / chart `service.webWriteTimeout` | How long the exporter may spend writing an HTTP response | Chart values show `webReadTimeout: 10s`, `webWriteTimeout: 30s` |

The decisive sentence: "**A scrape does not force DCGM to query the hardware, so scraping more frequently than the effective DCGM watch cadence can return the same cached sample more than once.**" ✅ Scraping at 1 s against a 30 s watch buys 29 duplicate samples per measurement — storage and cardinality cost for zero information.

| Class | Examples | Cost behaviour |
|---|---|---|
| **Cheap per sample, high value** | Device identity/brand/serial and driver version (as *labels*), SM/memory clock, GPU and memory temperature, board power, framebuffer used/free, GPU and memory utilisation | Read from cached samples; safe to include by default across a large fleet |
| **Moderate** | Throttle-time counters, PCIe replay counter, NVLink throughput aggregates | Counters need baseline subtraction; per-lane fan-out multiplies series count |
| **Expensive / high cardinality** | Per-lane NVLink fields (L0…L17 × GPUs × nodes), per-MIG-instance samples (`GPU_I_PROFILE`, `GPU_I_ID` labels), fine-grained profiling counters, per-process/job statistics | Cardinality, not CPU, is the cost. 18 lanes × 8 GPUs is 144 series per node before anything else |
| **Whole-host, not per-device** | `nv-hostengine` CPU and memory use (introspection module) | Watch this whenever you shorten intervals fleet-wide |

**Cardinality arithmetic to do before deploying:**

```
series ≈ nodes × (devices_per_node × active_metric_families
                  + links_per_device × link_metrics
                  + mig_instances × mig_metrics)
```

A 64-node estate of 8-GPU nodes on the 25 default metric families is `64 × 200 = 12,800` series — comfortable. Add 18 per-lane NVLink metrics and it becomes `64 × (200 + 144) = 22,016`. Add per-MIG-instance slices for a shared platform and it jumps again by roughly the device count multiplied by the instance count. Prometheus does not degrade gracefully at the top end: the symptoms are scrape timeouts (which is why `scrape_timeout < scrape_interval` matters), long query latencies, and memory-driven restarts.

**Guidance worth writing into a runbook.** (1) Choose the slowest cadence that still meets the response requirement — NVIDIA's own design procedure lists this as step 4 of 8 for a new monitoring integration. ✅ (2) Alert thresholds need a cadence that can see the event: `DCGM_FI_DEV_XID_ERROR` is a last-value gauge, so if XID visibility matters use the exporter's windowed/counted families instead. (3) Retention is an operational buffer, not an archive — "the cache is held in host memory… Export samples to a time-series or accounting system when they need durable retention." ✅ (4) A scratch `dcgmi dmon` is a live watch: default delay 1,000 ms, minimum 1 ms, runs until interrupted unless `--count` is given, and retains at most two samples for its client. (5) Cost grows superlinearly with ambition: entity-field × cadence × retention × clients is a product, and each factor is under someone's control.

---
## 5. Health Monitoring: The Passive Model

The health module is **module ID 4**, and NVIDIA's definition draws the whole boundary: it "uses passive field watches to assess subsystem health **without running a dedicated workload**. The module is loaded when health watches or checks are first requested." ✅ That word — *passive* — is the design. Health does not prove capability; it answers the narrower, cheaper question a running fleet needs: **does recent telemetry contain evidence of a known problem?** It can stay enabled while applications run, which is what makes it usable on a shared platform where you cannot take nodes out of service on a hunch.

**Subsystems watched (the `--set` selector alphabet), and what each evaluates:**

| Selector | System | Incidents evaluated |
|---|---|---|
| `p` | PCIe | Excessive replay rate; bus- or fabric-related XIDs |
| `m` | GPU memory | Volatile double-bit ECC errors, faulty memory, pending or excessive page retirement, row-remap failures, contained/uncontained errors |
| `i` | InfoROM | Reported InfoROM corruption |
| `d` | Driver | Recovery states requiring traffic drain, reset or reboot |
| `t` | Thermal **and** power | Thermal or power clock events, threshold violations, unreadable GPU power |
| `n` | NVLink | Link errors and error rates, BER thresholds, unhealthy fabric state |
| `e` | IMEX (Internode Memory Exchange) | Unhealthy IMEX daemon or domain state |
| `x` | ConnectX | Unhealthy device state; fatal or non-fatal PCIe AER errors |
| `a` | All of the preceding plus NVSwitch fatal and non-fatal | The preceding incidents plus NVSwitch errors and down links. `a` must be used alone and notably does **not** include the SM watch shown by `--fetch` |

**Severity semantics — the vendor's own words, which are the alerting contract.** *Warning*: "an issue has been detected that won't prevent current work from completing, but the issue should be examined and potentially addressed in the future." *Failure*: "a critical issue has been detected and the current work is likely compromised or interrupted. These situations typically correspond to fatal RAS events and usually indicate the need for job termination and GPU health analysis." `dcgmi health --check` reports `Healthy`, `Warning` or `Failure` overall, "followed by affected entities and their incidents" — a group-level score plus per-GPU rows (the documented example shows group-level `Warning` with one GPU on PCIe replay rate and one on a corrupt InfoROM). ✅

**Cadence and retention.** `--update-interval` defaults to **30 seconds**; `--max-keep-age` defaults to **600 seconds**. And a warning that catches people out: "Some watches require approximately 60 seconds of samples, so an immediate first check may not be meaningful." ✅ A check run seconds after enabling watches can return `Healthy` simply because nothing is retained to evaluate — absence of evidence misread as evidence of health.

### 5.1 Failure conditions: the `DCGM_FR_*` incident codes

These are the codes alerts branch on. All read from the Health Monitoring learn page's Failure Conditions table, 19 Sep 2026.

| System | Code | Cause |
|---|---|---|
| Any | `DCGM_FR_FALLEN_OFF_BUS` | A GPU has fallen off the bus |
| Any | `DCGM_FR_XID_ERROR` | One or more "devastating" XIDs associated with a critical hardware error has occurred |
| Driver | `DCGM_FR_GPU_RECOVERY_DRAIN_P2P` | A GPU requires peer-to-peer traffic to be quiesced |
| Driver | `DCGM_FR_GPU_RECOVERY_DRAIN_RESET` | A GPU is operating at reduced capacity due to a fault |
| Driver | `DCGM_FR_GPU_RECOVERY_REBOOT` | A GPU fault may have left the operating system in an inconsistent state |
| Driver | `DCGM_FR_GPU_RECOVERY_RESET` | A GPU requires a reset to recover from a fault |
| Driver | `DCGM_FR_GPU_RECOVERY_IMEX_DOMAIN` | GPU recovery requires restoring the IMEX domain (distinct from an IMEX health-watch incident) |
| PCIe | `DCGM_FR_PCI_REPLAY_RATE` | More than **8** PCIe replays in the last minute |
| PCIe, NVLink | `DCGM_FR_XID_ERROR` | XIDs associated with bus or fabric errors; or with NVLink flow control, C2C link correction, or fabric linear addressing (FLA) privilege |
| Memory | `DCGM_FR_FAULTY_MEMORY` | Faulty memory elements detected on a GPU |
| NVLink | `DCGM_FR_NVLINK_ERROR_CRITICAL` | An NVLink reported 1 or more errors (data-link CRC, recovery, replay on pre-Blackwell; link recovery events on Blackwell+) |
| NVLink | `DCGM_FR_NVLINK_ERROR_THRESHOLD`, `…_CRC_ERROR_THRESHOLD` | An NVLink reported a rate of errors exceeding the minimum threshold; or more than **100 CRC errors per second** |
| NVLink | `DCGM_FR_NVLINK_EFFECTIVE_BER_THRESHOLD`, `…_SYMBOL_BER_THRESHOLD` | Effective bit error rate, or symbol bit error rate, exceeds the minimum threshold |
| NVSwitch (fatal) | `DCGM_FR_NVSWITCH_FATAL_ERROR`, `DCGM_FR_NVLINK_DOWN` | An NVSwitch reported one or more fatal errors; one or more NVLinks is being reported as down |
| NVSwitch (non-fatal) | `DCGM_FR_NVSWITCH_NON_FATAL_ERROR` | An NVSwitch reported one or more non-fatal errors |
| ConnectX | `DCGM_FR_FIELD_VIOLATION` | A ConnectX device reported an unhealthy status, or PCIe AER uncorrectable errors (fatal or non-fatal) |
| IMEX | `DCGM_FR_IMEX_UNHEALTHY` | Unhealthy IMEX daemon or domain state (added in DCGM 4.7.0) |
| (diagnostics) | `DCGM_FR_*` | The diagnostic module also publishes `DCGM_FR_*` identifiers with categories, severities and response guidance — a **different catalogue**, documented under Diagnostic Errors |

That last row is a naming hazard worth stating plainly: `DCGM_FR_` is a shared prefix across health incidents and diagnostic findings. The identifiers are not interchangeable, the references are separate pages, and a runbook that greps for `DCGM_FR_` without knowing which producer it is reading will confuse a health warning with a diagnostic failure.

### 5.2 The workflow, and where health sits between telemetry and diagnostics

```bash
dcgmi group --create job-4815 --add gpu:0,gpu:1
dcgmi health --set pmitnd --group <group-id> --update-interval 30 --max-keep-age 600
dcgmi health --fetch --group <group-id>          # confirms what is ENABLED; does not assess health
dcgmi health --check --group <group-id> --json   # evaluates retained samples; reports severity
dcgmi health --clear --group <group-id>          # cleanup, then: dcgmi group --delete <group-id>
```

Four documented facts decide whether that works in production: **watches outlive the CLI** ("Health watches remain active in the host engine after `dcgmi` exits") but are **not durable across a host-engine restart**, so a long-lived monitor should use a stable group, leave watches enabled, and keep retention long enough for the monitoring system to read it; **`--set` replaces the whole mask** (there is no "add one more watch"); **retention is a window on truth, not a log** — a short `--max-keep-age` "can discard the evidence for an earlier incident", which is exactly why fleets push health state into Prometheus continuously rather than checking on demand; and **built-in group aliases** are available (`g` all GPUs, `s` all NVSwitches, `i` all GPU instances, `c` all compute instances, `a` all entities; default `g`).

| Stage | Mechanism | Applies load? | Answers |
|---|---|---|---|
| Collection | Core field watches | No | What are the measurements? |
| Judgement (continuous) | Health — converts selected measurements and events into warning/failure incidents that can be polled | No | Does recent telemetry contain evidence of a known problem? |
| Judgement (targeted) | Policy — watches a smaller set of conditions and can notify a listener or request an action | No | Did *this specific field* cross *this specific threshold*? |
| Proof | Diagnostics — active tests when hardware can be taken out of service | **Yes** | Can this device execute a known test successfully? |

### 5.3 What a health check does NOT cover

- **A `Healthy` result is not a clean bill of health.** NVIDIA: "A `Healthy` result means that no enabled health rule found an incident in the retained data. It does not mean that every subsystem was watched or that an active stress test passed." ✅ Three caveats in one sentence: the *enabled* rule set is the ceiling, the *retained* window is the floor, and nothing here proves capability.
- **It cannot detect the absence of performance.** A GPU silently running at half its usual throughput because of an application-side change produces no health incident. Health detects *conditions*, not *regressions*; regression detection needs a baseline (§10).
- **It cannot attribute a fault to a workload.** Health says an entity is unwell; it does not say which process caused it or which job is affected — `dcgmi stats` and your scheduler own that correlation.
- **It does not manage the fabric.** Fabric Manager remains responsible for fabric configuration and lifecycle; health observes link and NVSwitch *state* and can raise IMEX or NVSwitch incidents, but it is not a fabric manager.
- **It runs in the installed environment.** Because these are *online* checks, "there is potential for factors beyond the GPU to influence behavior in negative ways". NVIDIA's conclusion: "full offline diagnostics delivered via a different NVIDIA tool are required for complete hardware validation, and are required for RMA." ✅ A health check can never substitute for an RMA determination.
- **`Warning` is not self-evidently actionable, and `Failure` is not self-evidently fatal.** Mapping `Warning` to "raise a ticket" and `Failure` to "drain now" is a policy decision your platform team makes, not something DCGM decides. §10 and §13 do that work.

---
## 6. Diagnostics: Levels, Runtimes and Blast Radius

The diagnostic module is **module ID 7**. It "coordinates active deployment, hardware, integration, and stress tests and is loaded when a diagnostic is first requested". Older material and configuration call the same engine by its previous name, **NVIDIA Validation Suite (NVVS)**; the supported customer workflow is `dcgmi diag`. ✅ The question is the one health cannot answer — **can this accelerator and its surrounding software or interconnect execute a known test?** — and it comes with a price: diagnostics "can consume GPU compute, memory, PCIe, NVLink, CPU, power, and cooling capacity. Longer suites can reset state or conflict with running processes." ✅ That one warning is why diagnostics are a *maintenance-window* tool, not a monitoring tool.

### 6.1 The levels, with documented runtimes

| Level | Aliases | Vendor's documented runtime | Scope, in the vendor's words |
|---|---|---|---|
| **`1`** | `quick`, `short` | "Quick deployment checks, **normally lasting seconds**" | Deployment readiness, including the built-in `software` checks; the prologue/readiness gate |
| **`2`** | `medium` | "Medium validation, **normally lasting approximately two minutes**" | Investigation after a failed workload; a mid-weight hardware pass |
| **`3`** | `long` | "Long hardware diagnostics, **normally lasting approximately 15 minutes**" | Administrator-led isolation; adds deeper hardware, interconnect, performance and stress tests |
| **`4`** | `xlong` | "Extended, **longer-running** hardware diagnostics" | The only level including `memtest` and `pulse_test`; no fixed runtime is documented |
| **`production_testing`** | none | Not a level | A *meta-suite*: "the level-3 set with production-test behavior for supported plugins. This selector is **not a numbered validation level** and **does not add the level-4 `memtest` or `pulse_test` tests**" |

Three rules follow. **Later suites are supersets**: "later suites include the checks from earlier suites and add more invasive hardware, interconnect, performance, and stress tests". **Level 4 is where the blast radius is** — only it brings `memtest` (memory-cell, address-line, stuck-bit, coupling, retention and data-pattern tests) and `pulse_test` (board or power-delivery instability under rapid changes in GPU current and power demand). And **non-datacenter GPUs are level-1 only** unless a higher level is explicitly documented for the product.

⚠️ **The runtimes are the vendor's own characterisations ("normally lasting"), not SLAs.** Wall-clock depends on hardware, plugin availability, `--iterations`, parameters such as `targeted_power.test_duration`, and whether tests are skipped. Budget maintenance windows from measured runs on your own hardware, and set `--timeout` deliberately: a zero `--timeout` means *unlimited*.

### 6.2 What each test exercises

| Test | Defects or conditions it is designed to identify |
|---|---|
| `software` | Missing or unusable NVML/CUDA libraries; inaccessible device nodes or cgroup restrictions; conflicting drivers or processes; problematic environment variables; page-retirement, row-remap, InfoROM, memory-health or Fabric Manager state. Run level 1 to select these built-in checks alone |
| `context_create` | Failure to resolve a GPU and create a basic CUDA context — driver initialisation, device access, incompatible compute mode |
| `memory`, `memory_bandwidth` | Framebuffer allocation, write/read, data-integrity, ECC and L1 cache/tag faults; local framebuffer bandwidth below the device threshold, or inability to complete the bandwidth workload |
| `pcie`, `nvbandwidth` | Low host/device or peer bandwidth, excessive latency, broken peer access, PCIe replays or AER events, NVLink state or error problems; failed GPU copy paths over NVLink or PCIe |
| `diagnostic` | Compute or framebuffer data-path errors under sustained matrix work — miscompares, XIDs, ECC events, thermal excursions, unexpectedly inconsistent performance |
| `targeted_stress`, `targeted_power` | Inability to sustain a requested compute rate, unstable performance, or telemetry errors while a controlled GEMM workload runs; inability to reach the requested power threshold, or errors under sustained power load |
| `memtest` | Memory-cell, address-line, stuck-bit, coupling, retention and data-pattern errors across a configurable set of memory tests |
| `pulse_test` | Board or power-delivery instability exposed by rapid changes in GPU current and power demand |

Named tests combine — `dcgmi diag --run pcie,diagnostic` — and "an explicitly named test also performs the built-in software checks". ✅

### 6.3 What a diagnostic can invalidate or disrupt

- **It consumes the resources the workload needs** — compute, memory, PCIe, NVLink, CPU, power and cooling. Running level 3 on a node also serving inference means measuring a confounded system and degrading a customer.
- **"Longer suites can reset state or conflict with running processes."** A GPU reset clears other tenants' contexts. On a shared node that is someone else's training job dying.
- **It can trip throttling and clock events.** DCGM provides `--clocksevent-mask` to ignore selected clock-event reasons (`HW_SLOWDOWN`, `SW_THERMAL`, `HW_THERMAL`, `HW_POWER_BRAKE`, or a decimal mask built from 8, 32, 64, 128). Masking is legitimate; masking a real thermal limit to make a suite pass is how you ship a bad card.
- **It can hang.** Diagnostic *task* hang detection is available in DCGM 4.4 and later, is enabled by default, and is tunable via `NVVS_HANGDETECT_DISABLE` and `NVVS_HANGDETECT_EXPIRY_SEC`; host-engine hang detection is available in 4.4.2 and later. A client can enable a heartbeat with `--enable-heartbeat` (4.4.2+), and "if the initiating client disconnects or stops sending heartbeats, the host engine cancels its active diagnostic". ✅
- **It fails for non-hardware reasons.** The `software` test exists because diagnostics can fail on missing libraries, blocked device nodes, cgroup restrictions, conflicting processes or bad environment variables. A diagnostics "failure" that is really a container misconfiguration is a classic false alarm.

### 6.4 Reading a diagnostic result, and reconciling it with health

Two independent outputs must be read together: **`Pass`/`Fail`/`Skip`** describes "the outcome of that test invocation" for that entity, while **the associated error code and message** "distinguish a diagnostic finding from a setup, execution, or availability problem". So `Skip` is not a pass — it means the suite never exercised the device. A runbook line worth keeping: **treat `Skip` as `Unknown`, never as `Pass`.** Diagnostic exit statuses are documented and scriptable:

| Shell status | DCGM status | Meaning |
|---|---|---|
| `0` | `DCGM_ST_OK` | Diagnostics completed without a reported error |
| `226` | `DCGM_ST_NVVS_ERROR` | The diagnostic ran but reported an error |
| `217` | `DCGM_ST_DIAG_ALREADY_RUNNING` | Another diagnostic is already running |
| `215` | `DCGM_ST_DIAG_BAD_LAUNCH` | DCGM could not launch the diagnostic |
| `205` | `DCGM_ST_NVVS_ISOLATE_ERROR` | The diagnostic reported a condition that requires **isolation** |
| `204` | `DCGM_ST_NVVS_BINARY_NOT_FOUND` | The diagnostic executable was not found |
| `203` | `DCGM_ST_NVVS_KILLED` | The diagnostic process was terminated by a signal |
| `198` | `DCGM_ST_NVVS_NO_AVAILABLE_TEST` | No available diagnostic matched the request |

`205` deserves separate treatment in any automation: it is DCGM's own signal that the device should be isolated — a stronger statement than "a test failed". There is also `--ignoreErrorCodes`, accepting a comma-separated code list (`28,140`), semicolon-separated per-GPU clauses (`gpu0:28;gpu1:140`; DCGM 4.6 supports per-entity clauses **for GPUs only**), or `*:*` for every suppressible code on all entities. Suppression is legitimate for a known-benign finding; it is abuse when used to quiet a suite during a bake-off.

**When a diagnostic disagrees with the health checks**, NVIDIA answers directly — "that is normal":

> "A node can have no passive health incidents and still fail an active diagnostic, **or** can retain a historical health warning after the underlying hardware passes a later test. Use both results as evidence in the wider node and workload workflow." ✅

| Observation | What it means | What to do |
|---|---|---|
| Health `Healthy`, diagnostic `Fail` | A capability problem telemetry alone never showed — the classic case for diagnostics | Treat the diagnostic as stronger evidence **for this device's ability to work now**; preserve both records; investigate with the specific failed test |
| Health `Warning`/`Failure`, diagnostic `Pass` | Either the incident was transient (the diagnostic ran later, on an idle and possibly cooled device), or the health rule is noisy | Do not close the incident on the pass alone; check the retention window and the incident timestamp, re-enable watches, observe under real load |
| Both `Failure` | Strongest evidence | Isolate — `DCGM_ST_NVVS_ISOLATE_ERROR` (205) exists exactly for this handoff |
| Health `Failure`, diagnostic cannot run (`204`/`198`/`215`) | Deployment or packaging problem, not a GPU problem | Fix the diagnostic plumbing before drawing hardware conclusions |
| Both clear, workload still fails | The problem is not the device | Move to application evidence: sanitizers, debuggers, the application profiler; see [gpu_optimization_guide.md](gpu_optimization_guide.md) |

### 6.5 When each level is appropriate — the operational rule

| Context | Level | Reasoning |
|---|---|---|
| Node acceptance / bring-up before it enters the pool | **`1`** then **`3`** (or `4` once, if the window allows) | Level 1 is a seconds-long readiness gate; level 3 is the burn-in worth doing once per node |
| Prologue check before a large job starts | **`1`** | "Prologue diagnostics use the quick suite to test readiness before a job starts" — seconds, non-disruptive |
| After a failed job, once the GPUs are idle | **`2`** | "Epilogue diagnostics use a medium suite or selected tests after a failed job, when the GPU is no longer serving the workload" — ~2 minutes |
| Incident triage with a specific suspicion | **Named tests** | Suspect PCIe/NVLink → `pcie` or `nvbandwidth`; suspect a memory cell → `memtest`; suspect power delivery → `pulse_test`. Precision beats escalation |
| Administrator-led isolation before returning a node to service | **`3`** or **`4`**, in a maintenance window, node drained | Level 4's `memtest` and `pulse_test` are the invasive ones |
| Rack/cluster fabric validation | `dcgmi mndiag`, e.g. `nvloom` / `mnnvbandwidth` in DCGM 4.7 | Coordinated across hosts; a separate module, command, configuration model and catalogue |

**The vendor's pre-flight checklist before an invasive run** — reproduce it as your own checklist: discover and select explicit entities; drain applications and peer users from those entities; confirm the diagnostic service account can access device nodes, required libraries, plugin files and external test executables; and choose a timeout longer than the requested test durations. ✅ Notice that *two of the four are about permissions and coordination*, not the GPU — which is exactly where real deployments fail.

⚠️ **Two managed expectations.** Diagnostics are *online*, so "a failure can originate in the host, permissions, libraries, topology, cooling, power, or workload interference as well as in the target processor". And DCGM is explicit about what diagnostics are not: they "do not repair faults, replace NVIDIA offline field diagnostics, or determine RMA eligibility". ✅ A platform team presenting a DCGM pass as a vendor-accepted hardware clearance is overstating the tool; a vendor requiring a DCGM *fail* before accepting an RMA is asking for evidence DCGM explicitly declines to provide. **Configuration is first-class, not a detail:** diagnostics resolve each plugin's parameters from plugin fallbacks → the packaged SKU configuration (`diag-skus.yaml`) → an optional `dcgm-diag-v1` YAML overlay via `--configfile` → command-line `--parameters` (final priority). A configuration "can allow or disable an available plugin and set SKU-specific expectations; **it cannot make an absent plugin or unsupported test available**." ✅ The vendor's example overlay matches a SKU by `id` (e.g. `H100 80GB PCIe`, id `2331`) and warns to "use the device ID for the target GPU rather than applying the H100 identifier to another product" — the mistake that produces meaningless passes on a heterogeneous fleet.

---
## 7. The XID Failure Taxonomy

### 7.1 What an XID is, and where it comes from

XIDs are not a DCGM concept, and it is worth saying so first: they come from the **NVIDIA driver**, not from DCGM, not from Prometheus, not from your application.

> "The Xid message is an error report from the NVIDIA driver that is printed to the operating system's kernel log or event log. Xid messages indicate that a general GPU error occurred, most often due to the driver programming the GPU incorrectly or to corruption of the commands sent to the GPU. The messages can be indicative of a **hardware problem**, an **NVIDIA software problem**, or a **user application problem**." ✅ — XID Errors, *Introduction*, `docs.nvidia.com/deploy/xid-errors/`, checked 19 September 2026.

Three further statements from the same page shape how they may be used. **"The meaning of each message is consistent across driver versions"** — so an XID code is a durable identifier you can encode in runbooks and dashboards. **XIDs are debugging guides, not diagnoses**: "Because many problems can have multiple possible root causes, it's not always feasible to understand each issue from the Xid value alone" — the page's own illustration being that an XID might indicate a user program accessed invalid memory, "but in theory, memory corruption due to PCIE or frame buffer ('FB') problems could corrupt any command and thus cause almost any error". And the classifications "should be used as a starting point for further investigation of each problem", with NVIDIA pointing onward to the *GPU Debug Guidelines* manual.

**SXid**, for completeness: NVIDIA drivers for NVSwitch report NVSwitch hardware error conditions through a similar mechanism, called "Switch Xids" or **SXids**, documented separately in the *Fabric Manager User Guide*. The vendor notes a scope limit: "SXid messages apply only to **Hopper and earlier** generation GPUs." ✅

### 7.2 How XIDs surface, and how to collect them

Under Linux they go to the kernel log buffer; "depending on Linux distribution these are typically logged to journal and flushed to files, such as `/var/log/messages` or `/var/log/syslog`". ⚠️ The vendor names those *paths* rather than a reader command; the documented discovery command is a grep for **`NVRM: Xid`**. The documented message shape:

```
[...] NVRM: GPU at 0000:03:00: GPU-b850f46d-d5ea-c752-ddf3-c4453e44d3f7
[...] NVRM: Xid (0000:03:00): 14, Channel 00000001
```

The **first** Xid in a log is preceded by a line containing the GPU GUID — "a globally unique, immutable identifier for each GPU". Join on that, not on the PCI address: PCI enumeration can change across reboots and re-seating, the GUID does not. Each subsequent line carries the PCI bus/device/function ID, the XID identifier, and data specific to that XID. NVIDIA names three tools for XID work: **`nvidia-smi`** ("basic monitoring and configuration data about each GPU"; can list ECC error counts for Xid 48, indicate an unplugged power cable for Xid 54, or show the applicable GPU Recovery Action for Xid 154); **DCGM**, whose diagnostics "can check for basic GPU health, including the presence of ECC errors, PCIe problems, bandwidth issues, and general problems with running CUDA programs"; and **`nvidia-bug-report.sh`**, which collects debug logs and driver logs into a single compressed `nvidia-bug-report.log.gz`. That script is "typically quick, but in rare cases may run slowly. Allow up to one hour for it to complete", with a `--safe-mode --extra-system-data` variant for hosts where collection hangs. ✅

And the recommended actions per failure class — the honest answer to "what do I do with an XID":

| Issue class | Recommended action |
|---|---|
| Suspected **user programming** issues | Run debugger tools: Compute Sanitizer `memcheck`, CUDA-GDB |
| Suspected **hardware** problems | Contact the hardware vendor, who run their hardware diagnostic process |
| Suspected **driver** problems | File a bug with NVIDIA, including `nvidia-bug-report.sh` output |

Notably, **the vendor's own guidance for suspected hardware faults is to contact the hardware vendor — not to run DCGM and decide for yourself.** DCGM diagnostics are a triage and isolation aid; the RMA path runs elsewhere (§6.5).

### 7.3 The catalogue, and the vendor's own classification scheme

The catalogue is versioned and hardware-scoped, and this is the most important sourcing fact in the chapter: NVIDIA's catalog page states that for **Volta and older** GPUs you must consult a separate archived catalogue, while for **Ampere and newer (including PCIe form-factor GPUs)** the current graph/table applies, with a downloadable `Xid-Catalog.xlsx`. The modern table's columns are `Type (XID)`, `Code`, `Mnemonic`, `Description`, **`Applies to A100` / `H100` / `B100` / `GB200`**, `Resolution Bucket (Immediate Action)`, `Resolution Bucket (Investigatory Action)`, `Xid 154 linkage`, and `Trigger Conditions`. ✅

The right way to present the taxonomy to a fleet team is therefore to use **the vendor's own action vocabulary**, not an invented one. The resolution buckets observed on 19 September 2026 include the immediate actions `CONTACT_SUPPORT`, `RESTART_APP`, `IGNORE`, `RESET_GPU`, `RESTART_BM`, `RESTART_FM`, `UPDATE_SWFW`, and workflows `WORKFLOW_XID_13`, `WORKFLOW_XID_31`, `WORKFLOW_XID_45`, `WORKFLOW_XID_48`, `WORKFLOW_NVLINK_ERR`, `WORKFLOW_NVLINK5_ERR`, `XID_154`; plus investigatory values `CHECK_APP/CUDA`, `INVESTIGATE_SW` and `CONTACT_SUPPORT`. Those are literal strings from the table. Where the vendor's own documentation supports a **hardware / software / application** grouping — as the Introduction does at the level of the mechanism, and as specific `Trigger Conditions` cells do case by case — this guide groups below and cites that text. Where the vendor does not classify a code that way, neither does this guide.

**Representative codes, with the vendor's own descriptions.** ⚠️ Every row is quoted or closely paraphrased from the Xid catalog page as read on 19 September 2026, including its "Applies to" columns. This is a **selection**, not the catalogue; the catalogue is the source of truth and should be pinned, versioned and re-downloaded when the driver major version changes.

| XID | Mnemonic (as published) | Description | Applies A100/H100/B100/GB200 | Immediate bucket | Class indicated, per the vendor's trigger text |
|---|---|---|---|---|---|
| **8** | `ROBUST_CHANNEL_FIFO_ERROR_IDLE_TIMEOUT` | GPU stopped processing | Y/Y/Y/Y | `RESTART_APP` | Driver/hardware condition; restart the application first |
| **11**, **25** | `…GR_ERROR_MISSING_HW`, `…GR_ILLEGAL_NOTIFY` | Invalid or corrupted / illegal push buffer stream | Y/Y/Y/Y | `RESTART_APP` | Investigatory bucket `CHECK_APP/CUDA` |
| **13** | `ROBUST_CHANNEL_GR_EXCEPTION` | Graphics Engine Exception | Y/Y/Y/Y | `RESTART_APP` (`WORKFLOW_XID_13`) | **Application**: "logged for general user application faults. Typically this is an out-of-bounds error… but could also be an illegal instruction, illegal register, or other case." The vendor also notes "In rare cases, it's possible for a hardware failure or system software bugs to materialize as XID 13", and recommends cuda-gdb / Compute Sanitizer memcheck, `CUDA_DEVICE_WAITS_ON_EXCEPTION=1`, or filing a bug if those are inconclusive |
| **14** | `ROBUST_CHANNEL_FAKE_ERROR` | (Unused) | Y/Y/Y/Y | `IGNORE` | "Fake or injected error from userspace" — the code used by error-injection tooling |
| **31** | `ROBUST_CHANNEL_FIFO_ERROR_MMU_ERR_FLT` | GPU memory page fault | Y/Y/Y/Y | `RESTART_APP` (`WORKFLOW_XID_31`) | **Application**: "a fault is reported by the MMU, such as when an illegal address access is made… Typically these are application-level bugs, but can also be driver bugs or hardware bugs." Same three investigation paths as XID 13 |
| **32** | `ROBUST_CHANNEL_PBDMA_ERROR` | Invalid or corrupted push buffer stream | Y/Y/Y/Y | `RESTART_APP` | **Hardware/quality, explicitly not the application**: the DMA controller managing the driver↔GPU stream over PCIe; "these failures primarily involve quality issues on PCI, and are generally **not caused by user application actions**" |
| **37**, **38** | `…FECS_ERR_UNIMP_FIRMWARE_METHOD`, `…FECS_ERR_WATCHDOG_TIMEOUT` | Driver firmware error | Y/Y/Y/Y | `IGNORE` | **NVIDIA software/firmware** |
| **39**–**41**, **70**–**72**, **75**–**77**, **85** | `…CE0` … `…CE9_ERROR` | Copy Engine Exception / "CE<n>: Unknown Error" | Y/Y/Y/Y | `RESTART_APP` | Copy-engine condition; investigatory bucket `CONTACT_SUPPORT` |
| **43** | `ROBUST_CHANNEL_RESETCHANNEL_VERIF_ERROR` | GPU stopped processing | Y/Y/Y/Y | `IGNORE` | **Application**: "logged when a user application hits a software induced fault and must terminate. **The GPU remains in a healthy state.** In most cases, this is not indicative of a driver bug but rather a user application error." |
| **45** | `ROBUST_CHANNEL_PREEMPTIVE_REMOVAL` | Preemptive cleanup, due to previous errors — "Most likely to see when running multiple cuda applications and hitting a DBE" | Y/Y/Y/Y | `WORKFLOW_XID_45` — "Solo: `RESTART_FM` / Not Solo: `IGNORE` (follow other Xid)" | **Usually not a fault at all**: "logged when the user application aborts and the kernel driver tears down the GPU application… Control-C, GPU resets, sigkill are all examples… In many cases, this is not indicative of a bug but rather a user or system action." ⚠️ The "Solo: RESTART_FM" split refers to Fabric Manager — the one row whose action text travels outside the GPU |
| **46** | `ROBUST_CHANNEL_GPU_TIMEOUT_ERROR` | GPU stopped processing | Y/Y/Y/Y | **`RESET_GPU`** | Hardware/driver condition requiring a device reset |
| **48** | `ROBUST_CHANNEL_GPU_ECC_DBE` | Double Bit ECC Error | Y/Y/Y/Y | `WORKFLOW_XID_48` (both buckets) | **Hardware memory fault**: "logged when the GPU detects that an uncorrectable error occurs… A GPU reset or node reboot is needed to clear this error." If reported for SRAM (excluding framebuffer), check SRAM DBE thresholds and follow the RMA flow if exceeded. The `Xid 154 linkage` column notes CUDA 12.7 / driver R565 for this row |
| **62** | `PMU_HALT_ERROR` | Internal micro-controller halt (newer drivers) | Y/Y/Y/Y | **`RESET_GPU`** | Hardware/firmware |
| **63**, **64** | `INFOROM_DRAM_RETIREMENT_EVENT`, `…_RETIREMENT_FAILURE` | GPU memory remapping event / failure | Y/Y/Y/Y | `IGNORE` / **`RESET_GPU`** | **63 is informational** (dynamic page retirement / row-remapper activity, linking onward to *GPU Memory Error Management* and *Dynamic Page Retirement*); **64 is a hardware memory fault** — remapping could not complete. Pair 64 with the DCGM `DCGM_FI_DEV_ROW_REMAP_FAILED` field (395) |
| **74** | `NVLINK_ERROR` | NVLINK Error | Y/Y/**N**/**N** | `WORKFLOW_NVLINK_ERR` | **Interconnect**: "a problem with a connection from the GPU to another GPU or NVSwitch over NVLink. A GPU reset or node reboot is needed to clear this error." Crucially, "this event may indicate a hardware failure with the link itself, or may indicate a problem with the device **at the remote end** of the link. For example, if a GPU fails, another GPU connected to it over NVLink may report an Xid 74 simply because the link went down as a result." If repeated and a reset/reboot does not clear it, contact the hardware vendor |
| **78** | `VGPU_START_ERROR` | vGPU Start Error | Y/Y/Y/Y | `UPDATE_SWFW` (both buckets) | **Software/firmware stack** issue |
| **79** | `…GPU_HAS_FALLEN_OFF_THE_BUS` | GPU has fallen off the bus | Y/Y/Y/Y | **`RESTART_BM`** | **Hardware**: "the GPU driver attempts to access the GPU over its PCI Express connection and finds that the GPU is not accessible… often caused by hardware failures on the PCI Express link… may also be caused by failing GPU hardware or other driver issues." Also surfaces as DCGM's `DCGM_FR_FALLEN_OFF_BUS` |
| **80** | `PBDMA_PUSHBUFFER_CRC_MISMATCH` | Corrupted data sent to GPU | Y/Y/**N**/**N** | `RESTART_APP` | Investigatory bucket `CHECK_APP/CUDA` |
| **92** | `EXCESSIVE_SBE_INTERRUPTS` | High single-bit ECC error rate | Y/Y/Y/Y | `IGNORE` | **Degradation signal, not a failure**: single-bit errors are corrected, but the *rate* is the warning. These pair with DCGM's ECC counters |
| **94** | `ROBUST_CHANNEL_CONTAINED_ERROR` | Contained memory error | Y/Y/Y/Y | `RESTART_APP` | **Hardware, contained**: "these errors are contained to one application, and the application that encountered this error must be restarted. All other applications running at the time of the Xid are unaffected. It is recommended to reset the GPU when convenient. Applications can continue to be run until the reset can be performed." One possible cause is ECC memory error handling; Xid 45 accompanies it |
| **95** | `ROBUST_CHANNEL_UNCONTAINED_ERROR` | Uncontained memory error | Y/Y/Y/Y | **`RESET_GPU`** | **Hardware, uncontained**: "these errors affect multiple applications, and the affected GPU must be reset before applications can restart." Again, Xid 45 accompanies it. **This is the code that forces a drain on a shared node** |
| **119**, **120** | `GSP_RPC_TIMEOUT`, `GSP_ERROR` | GSP RPC Timeout / GSP Error | Y/Y/Y/Y | **`RESET_GPU`** | **Firmware/software**: "may be logged when an error occurs in code running on the GSP core of the GPU and/or a timeout occurs while waiting for the GSP core… to respond to an RPC message. A GPU reset or node power cycle may be needed if the error persists." Investigatory bucket `INVESTIGATE_SW` |
| **140** | `UNRECOVERABLE_ECC_ERROR_ESCAPE` | ECC Unrecovered Error | Y/Y/Y/Y | **`RESET_GPU`** | **Hardware memory fault**: "may occur when the GPU driver has observed uncorrectable errors in GPU memory, in such a way as to interrupt the GPU driver's ability to mark the pages for dynamic page offloading or row remapping. Reset the GPU, and if the problem persists, contact your hardware vendor" |
| **144** | `NVLINK_SAW_ERROR` | NVLINK: SAW Error | **N**/**N**/Y/Y | `WORKFLOW_NVLINK5_ERR` (both buckets) | Interconnect; note the "Applies to" columns — **not** on A100 or H100 in this catalogue revision |
| **154** | `GPU_RECOVERY_ACTION_CHANGED` | GPU Recovery Action Changed | Y/Y/Y/Y | `XID_154` | **Informational, and the most operationally useful row.** "Xid 154 will be seen in conjunction with other Xids and summarises the **recovery action required** for other Xids. The string will be similar to 'Xid 154 GPU recovery action changed from 0x0 (None) to 0x2 (Node Reboot Required)'" — where the expected values are **None, Drain P2P, Drain and Reset, GPU Reset Required, Node Reboot Required** |

Codes whose rows are marked `Unused` for all of A100/H100/B100/GB200 (for example 1–7, 9, 10, 12, 15–24, 26–30, 33–36, 42, 47, 73, 81, 87) are still listed with `CONTACT_SUPPORT` as the immediate bucket and "N/A; Unused" as the trigger condition. Do not read `Unused` as "harmless" — read it as "not expected to be raised on these products", and investigate if one appears.

### 7.4 Using XIDs operationally

**Rule 1 — the code alone is a starting point, never a conclusion.** A runbook mapping `79 → replace the card` is defensible (falling off the bus is not ambiguous, and the immediate action is `RESTART_BM`). A runbook mapping `13 → replace the card` is wrong: the vendor's own trigger text calls it a general user application fault and prescribes sanitizers and debuggers before hardware conclusions.

**Rule 2 — triage off the resolution bucket first, the description second.** The bucket answers "what do I do in the next five minutes", which is what on-call needs: `IGNORE` (do not page), `RESTART_APP`, `RESTART_BM`, `RESTART_FM`, `RESET_GPU`, `UPDATE_SWFW`, `CONTACT_SUPPORT`, or a `WORKFLOW_*` pointing at the vendor's own handling procedure. The mnemonic answers "what subsystem", which is what post-incident work needs. Inverting the order produces either useless pages or missed isolation.

**Rule 3 — XID 154 is the drain signal.** "The recovery action required" is exactly the input a scheduler adapter needs: `Drain P2P`, `Drain and Reset`, `GPU Reset Required`, `Node Reboot Required`. A platform team can reasonably automate a first response on 154 while keeping raw XID interpretation human (§8).

**Rule 4 — collect XIDs deliberately, because the default metrics path will not.** `DCGM_FI_DEV_XID_ERROR` (field 230) is documented as the **"value of the last XID error encountered"** — a gauge of the last one, not a counter. A device raising two XIDs between two scrapes can show only the second; one that raised and cleared may show nothing. And XIDs are per-device kernel-log events: correlating them to a device needs the **GUID**, which the exported metric carries as `UUID` while the kernel line carries a PCI address. Practical mitigation — and why the exporter's own families exist — is to use `DCGM_EXP_XID_ERRORS_COUNT` (windowed gauge) and `DCGM_EXP_XID_ERRORS_TOTAL` (counter since exporter start) alongside the kernel-log pipeline, and to join on GPU GUID rather than PCI address or index.

**Explicit refusal.** This guide will not interpret an XID code not quoted above from the vendor's catalogue, and will not assign a hardware/software/application class to a code whose vendor `Trigger Conditions` text does not support it. XIDs are vendor-defined identifiers whose meaning is stable across driver versions — which is precisely why a plausible-looking interpretation is worse than no interpretation. If a code is not in the catalogue: check the archived catalogue if the GPU is Volta or older, then contact support or the hardware vendor. **Versioning caution:** the catalogue's `Applies to` and `Xid 154 linkage` columns encode driver and CUDA dependencies (several rows cite "CUDA 12.7; GPU driver R565"), so a catalogue copied two years ago may classify a code differently for a current GPU generation. Pin the version you triage against and re-download when the driver major version changes.

---
## 8. The Policy Engine and Automated Response

The policy module is **module ID 5**. It "can evaluate policies against supported numeric GPU fields and retain the resulting violations. The older group policy interface remains available for compatibility. The module is loaded when policy functionality is first requested." ✅ A **field-based policy** identifies a numeric field, an operator, and (except for `changed`) a threshold, targeting **all GPUs or a list of GPU entity IDs**. Operators are `>`, `>=`, `<`, `<=`, `==`, `!=`, and `changed`. Notification channels are **console, file, or callback**, and "a per-entity rate limit controls repeated notifications". Policies "are persisted by the host engine and restored when the policy module loads". ✅ A field ID and a policy ID together select a policy for get, modify, delete, enable and disable operations; `--list` discovers the generated policy ID.

```bash
dcgmi policy --create --field 150 --name "GPU temperature warning" \
    --operator '>' --threshold 85 --entities all
dcgmi policy --watch --field 150
```

Note the shape: the field is identified by **numeric ID** (`150` being the GPU temperature field whose symbolic name is `DCGM_FI_DEV_GPU_TEMP_CELSIUS`). The legacy group policy interface "watches **seven fixed condition categories**", and NVIDIA's migration guidance is unambiguous: "for new automation, prefer a field-based policy because it identifies the exact numeric field and comparison being evaluated and can retain violations without an attached `dcgmi` listener." ✅

> "Keep recovery in a scheduler-aware consumer: legacy action and validation values are stored for compatibility but **do not reset a GPU or run a diagnostic**." ✅

A team configuring a legacy policy action expecting DCGM to act on it has misread the tool: the values persist, nothing happens.

**The vendor's own position on remediation, which this guide adopts rather than argues around:**

> "Policy is useful when an operator knows which field and condition matter and wants DCGM to **retain violations** for polling or callback delivery. For response logic — such as draining a scheduler node, resetting a GPU, running diagnostics, paging an on-call engineer, or correlating several metrics — **an external controller should consume DCGM health, fields, or policy notifications and perform that automation**." ✅

| Concern | Owner, per the vendor |
|---|---|
| Field, operator, threshold, entities | DCGM policy |
| Retaining the violation, notifying a listener | DCGM policy |
| Deciding to drain, reset, diagnose, page, or correlate | **An external controller you build, configure and test** |

### 8.1 What can and cannot be expressed

| Expressible in a field-based policy | Not expressible (needs an external controller) |
|---|---|
| A numeric field crossing a scalar threshold in either direction | Boolean combinations across fields with different cadences |
| Equality and inequality; the `changed` operator on a supported field | Thresholds varying by time, tenant or workload class |
| Per-GPU entity lists, or all GPUs | Sustained-condition logic ("for more than 10 minutes") unless the consumer implements it over the time series |
| Console/file/callback notification with per-entity rate limiting | Entity sets that change dynamically (a scheduler's current allocation); correlating a violation with a job, tenant or topology peer |

⚠️ **Inferred, not vendor-stated:** the "sustained for N minutes" gap follows from the documented condition model (field, operator, threshold) rather than from a documented limitation. In practice it is why almost every mature deployment implements thresholds as Prometheus alert rules over the exporter's time series, and uses DCGM policy, if at all, for host-local actions where no Prometheus exists. Say that in your design document; do not present the tool as having features it does not document.

### 8.2 Does automated response belong in a regulated estate's runbook or in a machine?

The honest position is that the answer depends on the **action**, not on the technology.

| Action | Automate? | Reasoning |
|---|---|---|
| Alert on `Warning`-severity health incidents and policy violations | **Yes** | Notification, not response — the entire point of the policy module |
| Cordon a node automatically on `GPU Reset Required` / `Node Reboot Required` (XID 154) or an uncontained memory error (XID 95) | **Yes, with guardrails** | The device state already means "do not place new work here". Cordon-and-drain is reversible and tenant-safe. Guardrails: idempotent, logged, observable — a silent auto-cordon is an outage nobody can explain |
| Page a human | **Yes** | Notification with a routing rule |
| Run a diagnostic suite automatically on the affected node | **Conditionally** | Only when the node is already drained, and only with an explicit diagnostic policy naming the level and window. Diagnostics consume resources and longer suites "can reset state or conflict with running processes" |
| Reset a GPU or reboot a node | **No, absent an explicit tested change process** | A GPU reset clears other tenants' contexts; on a shared node that is a multi-tenant incident caused by automation. The vendor does not expect DCGM to do it — legacy action values "do not reset a GPU" |
| RMA a card | **No** | Diagnostics "do not repair faults, replace NVIDIA offline field diagnostics, or determine RMA eligibility" |
| Reimage or move a node between pools | **No** | Out of scope for GPU observability; squarely in change management |

**The regulated-estate framing.** In an operational-resilience or ITSM-governed environment, the reviewer's question is not "did the automation work" but **"can you show what it did, to which asset, on whose authority, and why"**. That is a *record* requirement, and it changes the design even when automation is desirable: **every automated action must emit an event** (a cordon existing only as Kubernetes node state is not an audit record — emit the triggering condition, the asset identity as a GPU GUID rather than an index, and the timestamp); **automation scope must be bounded by policy, not by code**, as a reviewed, versioned artefact closer to a change record than a script; **every automated action needs a documented manual equivalent**, so the runbook still works when the controller is down; **reversible before irreversible** (cordon and drain first, reset and reboot only with process); and **the human decision points must remain visible** — a design that automates everything cannot demonstrate that a competent person weighed the evidence, and one that automates nothing cannot demonstrate timely response. The repository's operational-resilience and operational-support content owns the framework these controls sit inside; what is DCGM-specific here is the *conditions* (which `DCGM_FR_*` codes, which XID recovery actions, which thresholds) and the principles above.

**Operational cautions on the module itself.** `--import` "replaces the host engine's complete field-policy registry from YAML rather than merging entries", and — stated as a hazard — "**DCGM 4.7 can replace the registry with an empty or partially valid import**, so validate the file and review the resulting list" ✅; a pipeline that imports generated policy files without validating can silently delete every policy on the host engine, so validate and then verify with `--list`. Policy notifications are **"Not supported"** on non-datacenter GPUs, so a GeForce-based lab will not exercise the production policy path. `--watch` gives you a listener while the *retained* violation is what survives a client disconnect — prefer the retained form for monitoring integrations. And the per-entity rate limit exists because a policy on an oscillating field will otherwise notify on every crossing; set it deliberately rather than discovering the cost during an incident.

---
## 9. Kubernetes, Prometheus and the Exporter

### 9.1 The exporter's role, and what it is not

**DCGM Exporter** is a separate NVIDIA project — its own repository (`github.com/NVIDIA/dcgm-exporter`, **Apache-2.0**, verified from the repository `LICENSE` file and the README licence badge on 19 Sep 2026 ✅), its own container image, release notes and Helm chart. It "exposes NVIDIA GPU telemetry from NVIDIA Data Center GPU Manager (DCGM) in the Prometheus text format" and "converts selected DCGM telemetry fields to the Prometheus exposition format"; it is written in Go and is "a tool based on the Go APIs to NVIDIA DCGM". ✅

The architectural detail that determines deployment shape: **by default the exporter initialises an embedded DCGM host engine inside its own process through `libdcgm`**, and can instead "connect to a separately managed DCGM host engine over TCP, a Unix socket, or VSOCK". ✅

| Mode | What happens | When it bites |
|---|---|---|
| **Embedded** (default) | The exporter's process owns an in-process host engine; watches, samples and module state live in the pod | Clean and self-contained — but if the node already runs `nvidia-dcgm.service` there are now **two** host engines collecting overlapping data. NVIDIA flags it: "Because DCGM Exporter starts nv-hostengine as an embedded process… appropriate configuration options should be used if DCGM Exporter is run on systems (such as NVIDIA DGX) that have DCGM (or rather nv-hostengine) running" |
| **Remote to a host engine** (`-r host:port`) | The exporter's `libdcgm` client talks to a host engine elsewhere | Version compatibility is on you: the client library "must be greater than or equal to (but not less than) the version of DCGM running on the host" |
| **Remote to a DCGM standalone container** | The same, with the host engine in its own container publishing `5555` | Lets host `dcgmi` and the exporter share one engine — and another place version skew appears |

**Version pairing is an explicit support condition, not a nicety.** "Run DCGM Exporter with the DCGM version paired with that exporter release. Mismatched combinations might function, but they are not tested or supported." Container image tags encode the tested pair as `<dcgm-version>-<dcgm-exporter-version>-<image-variant>`, and `dcgm-exporter --version` reports both embedded versions. ✅ This is the mechanism behind a nasty fleet failure mode: a node whose driver and DCGM package were upgraded while the exporter image was not — producing blank metrics rather than an error (§14).

**The exporter's controls** (each has a flag and an environment variable):

| Environment variable | Flag | Purpose | Documented default |
|---|---|---|---|
| `$DCGM_EXPORTER_COLLECTORS` | `-f` | Path to the file containing DCGM fields to collect | `/etc/dcgm-exporter/default-counters.csv` |
| `$DCGM_EXPORTER_LISTEN` | `-a` | HTTP listen address | `:9400` |
| `$DCGM_EXPORTER_INTERVAL` | `-c` | Collection interval, **milliseconds** | `30000` |
| `$DCGM_EXPORTER_KUBERNETES` | `-k` | Enable Kubernetes mapping of metrics to pods | `false` |
| `$DCGM_EXPORTER_CONFIGMAP_DATA` | `-m` | ConfigMap namespace and name containing DCGM fields | `none` |
| `$DCGM_REMOTE_HOSTENGINE_INFO` | `-r` | Connect to a remote host engine `Host:Port`; otherwise embedded mode | none |
| `$DCGM_EXPORTER_DEVICES_STR` | `-d` | Which devices to monitor | All GPU instances in MIG mode; all GPUs if MIG disabled |

**Device-string syntax** — `[f] | [g[:id1[,-id2]]] | [i[:id1[,-id2]]]`: `f` = all GPUs if MIG is disabled or all GPU instances if MIG is enabled; `g` = all GPUs; `i` = all GPU instances; `g:0,1` = GPUs 0 and 1; `i:0,2-4` = GPU instances 0, 2, 3 and 4. Three documented warnings: `-i` cannot be specified unless MIG mode is enabled; any indices specified must exist on the system; and **in MIG mode only `f` or `i` with a range can be specified, because "GPUs are not assigned to pods and therefore reporting must occur at the GPU instance level"** ✅ — the technical reason a MIG-shared platform's monitoring must key on the *instance*, not the card.

### 9.2 The metric naming contract, and why it breaks dashboards

The exporter emits **Prometheus metric families named after the configured DCGM field**, with `# HELP`/`# TYPE` derived from the collector CSV. NVIDIA's own published example output shows the shape:

```
# HELP DCGM_FI_DEV_SM_CLOCK SM clock frequency (in MHz).
# TYPE DCGM_FI_DEV_SM_CLOCK gauge
DCGM_FI_DEV_SM_CLOCK{gpu="0", UUID="GPU-604ac76c-d9cf-fef3-62e9-d92044ab6e52"} 139

DCGM_FI_DEV_GPU_TEMP{gpu="0",UUID="GPU-…",device="nvidia0",Hostname="…"} 61
DCGM_FI_DEV_FB_USED{gpu="0",UUID="GPU-…"} 4845
DCGM_FI_DEV_XID_ERRORS{gpu="0",UUID="GPU-…"} 0
DCGM_FI_PROF_GR_ENGINE_ACTIVE{gpu="0",UUID="GPU-…"} 0.995630
```

Labels observed in the vendor's examples: `gpu` (index), `UUID`, `device`, `Hostname`, and for MIG `GPU_I_PROFILE` and `GPU_I_ID`. ✅ Two rules follow from the configured name *being* the metric name:

1. **The name you query is the name in the CSV, not the canonical field name.** The exporter's reference documents the divergence: configured `DCGM_FI_DEV_GPU_UTIL` → canonical `DCGM_FI_DEV_GPU_UTIL_RATIO`; `DCGM_FI_DEV_MEMORY_TEMP` → `DCGM_FI_DEV_MEMORY_TEMP_CELSIUS`; `DCGM_FI_DEV_XID_ERRORS` → `DCGM_FI_DEV_XID_ERROR`; `DCGM_FI_DEV_PCIE_REPLAY_COUNTER` → `DCGM_FI_DEV_PCIE_REPLAY_TOTAL`; `DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL` → `DCGM_FI_DEV_NVLINK_THROUGHPUT_TOTAL`. A runbook line naming "the DCGM metric for framebuffer used" and a dashboard querying `DCGM_FI_DEV_FB_USED` are both right *and* not the same string as the field's canonical documentation heading.
2. **Naming a field in a collector does not guarantee emission.** "Selecting a row does not guarantee runtime emission. DCGM, the GPU and other entities, the driver, permissions, and the deployment configuration determine whether a field is available."

The exporter also "does not infer or semantically validate" the Prometheus type you write against the field definition — "the configured Prometheus type is used as written" ✅ — so a `counter` written as a `gauge` produces `rate()` nonsense that looks like real data.

### 9.3 Wiring Prometheus: static targets, ServiceMonitor, and the three clocks

```yaml
scrape_configs:
  - job_name: dcgm-exporter
    scrape_interval: 30s
    scrape_timeout: 25s
    static_configs:
      - targets:
          - gpu-node-1:9400
          - gpu-node-2:9400
```

Verify first with `curl --fail http://gpu-node-1:9400/metrics`, then confirm the target is `UP` and smoke-test with `up{job="dcgm-exporter"}` and a DCGM metric such as `DCGM_FI_DEV_GPU_UTIL`. ✅

**ServiceMonitor wiring in Kubernetes.** The exporter's Helm chart "enables a `ServiceMonitor` by default" and the cluster must have the Prometheus Operator CRDs. The wiring has three independent gates that teams commonly satisfy only partially: the chart's resource selects the chart-managed Service; the Prometheus resource's `serviceMonitorSelector`/`serviceMonitorNamespaceSelector` must admit that `ServiceMonitor` **and its namespace**; and if Prometheus selects by label you must set `serviceMonitor.additionalLabels`. ✅ Two further documented facts matter operationally. **The chart-managed `ServiceMonitor` assumes unauthenticated HTTP** — "if the exporter uses TLS or basic authentication, disable that resource and provide a Prometheus scrape configuration with matching credentials and TLS settings", so a regulated estate requiring TLS on scrape endpoints must turn the vendor convenience off and wire it deliberately. And **a scrape does not force a hardware query**, so `scrape_interval` faster than the DCGM collection cadence returns duplicates: keep `scrape_timeout` below both `scrape_interval` and the exporter's `--web-write-timeout`, with documented chart values of `service.webReadTimeout: 10s`, `service.webWriteTimeout: 30s`, `serviceMonitor.interval: 30s`, `serviceMonitor.scrapeTimeout: 25s`. ✅ Verify the resource rather than assuming it:

```bash
kubectl get servicemonitor --namespace gpu-monitoring dcgm-exporter
```

### 9.4 How the GPU Operator deploys it

The **NVIDIA GPU Operator** "uses the operator framework within Kubernetes to automate the management of all NVIDIA software components needed to provision GPU. These components include the NVIDIA drivers (to enable CUDA), Kubernetes device plugin for GPUs, the NVIDIA Container Toolkit, automatic node labeling using GFD, **DCGM based monitoring** and others." ✅ (GPU Operator docs, version 26.7.) A default installation "deploys the NVIDIA GPU driver, NVIDIA Container Toolkit, NVIDIA Device Plugin, **DCGM Exporter**, and MIG Manager as pods on every GPU worker node", alongside GPU Feature Discovery and the operator/validator pods — `gpu-feature-discovery`, `nvidia-container-toolkit-daemonset`, `nvidia-cuda-validator`, `nvidia-device-plugin-daemonset`, `nvidia-driver-daemonset`, `nvidia-operator-validator`, `nvidia-dcgm-exporter`. ✅ Verification is `kubectl get pods -n gpu-operator` plus `kubectl get clusterpolicy` showing `ready`.

| Practical point | Detail |
|---|---|
| Find the exporter pods | `kubectl get pods --all-namespaces --selector app=nvidia-dcgm-exporter` |
| Change the metric set | "Customize custom fields through the Operator-managed metrics ConfigMap. **Do not edit the Operator-owned DaemonSet directly**" — the Operator reconciles your edit away |
| Namespace security | The Operator namespace needs `pod-security.kubernetes.io/enforce=privileged` where Pod Security Admission is in use — a governance review point, not just a technical one |
| Supply chain | The chart is available as an OCI artefact and NVIDIA documents verifying it with `cosign verify --key …/public-key --insecure-ignore-tlog nvcr.io/nvidia/cloud-native-charts/gpu-operator:v26.7.0`. Mirror-and-verify is the regulated-estate pattern |
| Standalone chart alternative | Repository `https://nvidia.github.io/dcgm-exporter/helm-charts`, namespace `gpu-monitoring`, DaemonSet `dcgm-exporter`, selector `app.kubernetes.io/name=dcgm-exporter`; install without the ServiceMonitor via `--set serviceMonitor.enabled=false` when the Prometheus Operator CRDs are absent |
| Package alternative | `datacenter-gpu-manager-exporter` installs `/usr/bin/dcgm-exporter`, `/etc/dcgm-exporter/default-counters.csv` and `/lib/systemd/system/nvidia-dcgm-exporter.service`. Do not edit the vendor unit under `/lib` (a package upgrade replaces it) — use `systemctl edit` drop-ins, remembering that a replaced `ExecStart` **does not inherit vendor command changes on upgrade** |

### 9.5 The dashboard ecosystem: vendor artefact versus community artefact

| Dashboard / integration | Provenance | Where it lives |
|---|---|---|
| **DCGM Exporter dashboard for Grafana** | Published by NVIDIA; linked from the exporter's own README | `grafana.com/grafana/dashboards/12239/` |
| **Checked-in Grafana dashboard** | NVIDIA; shipped in the repository | `NVIDIA/dcgm-exporter` → `grafana/dcgm-exporter-dashboard.json` |
| **OpenObserve dashboard + integration guide** | **Community**; listed by the exporter README under "Community" | `github.com/openobserve/dashboards` (NVIDIA GPU Monitoring) and the OpenObserve blog |
| Third-party dashboards, exporter forks, Helm forks | **Community**, unless published by NVIDIA | Various; verify provenance before adopting |

The working rule: **anything linked from NVIDIA's README or repository is a vendor artefact; anything the README labels "Community" is not; anything else is community by default until traced to an NVIDIA-published source.** A community dashboard is perfectly usable — it just cannot be the basis of a vendor-support conversation, and it must be reviewed like any third-party artefact.

### 9.6 The multi-tenant question

| Question | Tenant's view | Platform team's view |
|---|---|---|
| How busy is *my* allocation, and how much memory is it using? | Yes, scoped to it | Yes |
| How hot is the card that hosts me? | Useful, but only for *my* instance | Yes, and for the whole device |
| Which other tenants are on this card, and how busy are they? | Should not be visible | Yes |
| Is the card degrading (ECC, row remap, NVLink errors, XID history)? | Not their problem, and arguably not their business | Yes — the health signal that drives drain decisions |
| Which node should this workload run on? | Not their decision | Yes |

**The mechanism, and its limit.** In MIG mode, samples carry `GPU_I_PROFILE` and `GPU_I_ID` labels, so per-instance series exist alongside whole-GPU series, and `DCGM_EXPORTER_KUBERNETES`/`-k` enables "Kubernetes mapping of metrics to kubernetes pods", which is what attaches a pod identity to a series. But NVIDIA is explicit: "Kubernetes, DRA, vGPU, runtime-container, and HPC mapping add labels; **they do not make an unsupported DCGM field available**." ✅ The tenant-visible metric set is therefore bounded by what DCGM supports at the *instance* entity level, not by what a dashboard's queries can be filtered to. The honest position for a multi-tenant platform is fourfold: **tenant dashboards must be built on a separate data path, not a filtered view of the platform's** (label filtering in one Grafana is a configuration guarantee, not a technical one, and it fails open on the first misconfigured query — use a separate Prometheus or a tenant-facing aggregation layer scoped to tenant-relevant series); **health telemetry is platform-visible by default and should stay that way** (tenants need their resource envelope, not device-level failure archaeology that reveals platforms, topology and co-tenancy); **the defensible tenant rule** is *resource-envelope metrics scoped to their allocation* (utilisation, framebuffer used, per-instance profiling ratios) plus *availability*, and nothing about device health, peer links, node identity or co-tenancy; and **whole-GPU metrics on a MIG-shared card leak aggregate activity** — revealing them to the platform team while hiding them from tenants is itself a disclosure decision that belongs in the platform's data-classification review. The sharing mechanisms are owned by [hami_gpu_sharing_guide.md](hami_gpu_sharing_guide.md); the Kubernetes platform layer by [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md) and [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md); this guide supplies the GPU telemetry plane those stacks consume.

---
## 10. GPU Fleet Observability in Practice

### 10.1 Alert on health signals, page on failure signals

The single most useful split in GPU alerting is between metrics that **predict degradation** and metrics that **describe load**. A dashboard needs both; a pager needs only the first.

| Signal | Type | Why it belongs (or not) in an alert |
|---|---|---|
| `DCGM_FI_DEV_ECC_SBE_VOL_TOTAL` / `…_SBE_AGG_TOTAL` rising | **Health, leading** | Correctable errors are corrected — the *rate of change* predicts degradation. Correlates with XID 92 and precedes row remapping (XID 63/64) |
| `DCGM_FI_DEV_ECC_DBE_VOL_TOTAL` / `…_DBE_AGG_TOTAL` non-zero | **Health, failure-class** | Uncorrectable errors. Correlates with XID 48/140; a GPU reset or node reboot is needed to clear XID 48 |
| `DCGM_FI_DEV_ROW_REMAP_FAILED` = 1 | **Health, failure** | The vendor's own flag that remapping has failed. Pair with XID 64 |
| `DCGM_FI_DEV_UNCORRECTABLE_REMAPPED_ROWS` / `…_CORRECTABLE_REMAPPED_ROWS` climbing | **Health, leading** | Remapping capacity is finite; consumption is a countdown |
| `DCGM_FI_DEV_XID_ERRORS`, `DCGM_EXP_XID_ERRORS_COUNT/TOTAL` | **Health, event** | The gauge is last-value only (§7.4) — use the windowed/counted families for alerting, and route the code to the vendor's resolution bucket |
| `DCGM_FI_DEV_THERMAL_VIOLATION`, `DCGM_FI_DEV_GPU_THERMAL_THROTTLE_SECONDS_TOTAL` | **Health, performance-affecting** | Non-zero growth means the device is being constrained. A capacity and a reliability signal at once |
| `DCGM_FI_DEV_POWER_VIOLATION`, `…_GPU_POWER_THROTTLE_SECONDS_TOTAL` | **Health, performance-affecting** | Growth indicates power capping; distinguish board limit from thermal |
| `DCGM_FI_DEV_PCIE_REPLAY_COUNTER` growth rate | **Health, leading** | DCGM's own health rule fires above **8** replays per minute (`DCGM_FR_PCI_REPLAY_RATE`); XID 32 is the correlated failure |
| NVLink CRC/replay/recovery counters, `mnnvbandwidth` results | **Health, leading** | DCGM's health rules fire above **100** CRC errors/second (`DCGM_FR_NVLINK_CRC_ERROR_THRESHOLD`) |
| `DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL`, `DCGM_FI_PROF_PCIE_*_BYTES` | **Load** | Describes traffic. Useful for topology and bottleneck work, wrong as a page |
| `DCGM_FI_DEV_GPU_UTIL`, `DCGM_FI_DEV_FB_USED` | **Load** | Describes allocation and occupancy — see §11 for what it does *not* mean |
| `DCGM_FI_DEV_GPU_TEMP`, `DCGM_FI_DEV_MEMORY_TEMP`, `DCGM_FI_DEV_POWER_USAGE` | **Load with a health threshold** | Absolute values are context; sustained excursion above a device-specific ceiling is a health event |
| `DCGM_EXP_GPU_HEALTH_STATUS` | **Health, aggregate** | The exporter's DCGM-reported health status — the closest thing to a single "is this device well" series |

Two rules that make the difference between a useful page and alert fatigue. **Alert on rates and changes, not on absolute values**, for cumulative counters: a device with 50 historically-accumulated correctable errors is not the same as one that accumulated 50 in ten minutes. And **alert per device, then aggregate for humans**: a node-level "3 of 8 GPUs degraded" is a dashboard tile, not a page; the page should name the GPU and the condition.

### 10.2 The difference between a capacity metric and a health metric

A **capacity metric** answers "how much is left": framebuffer free, remapping capacity remaining, thermal headroom, power headroom, NVLink bandwidth headroom. A **health metric** answers "is this trending toward failure": error rates, throttle-time accumulation, ECC growth, row-remap consumption. The two are frequently conflated because both are "how much", which is why fleets end up alerting on "GPU memory > 90 %" — a *load* condition that is completely normal for a training job — while ignoring a steady climb in correctable ECC errors that predicts the failure. Write the alert list so that each entry names which of the three it is: **load**, **capacity**, or **health**.

### 10.3 Failure patterns a fleet operator actually sees over time

- **The slowly degrading card.** Correctable ECC errors climb for weeks, then a row-remap event (XID 63), then a remap *failure* (XID 64) or an SRAM DBE threshold breach, then XID 48. The only way to see this coming is a persisted baseline of ECC and remap counters — which is precisely what the default exporter CSV does *not* enable (§4.3).
- **The thermal case.** One node in a rack runs hotter than its identical peers because of airflow, dust or a neighbouring workload. It manifests as throttle-time accumulation and a lower clocks-per-work curve, never as a hard failure. It is detectable only comparatively — across the fleet, not on the device.
- **The interconnect problem.** An NVLink error count on one GPU (XID 74) that is really the *remote* end of the link being unwell (§7.3). Teams replace the wrong card when they treat the reporting GPU as the faulty one.
- **The silent regression.** A driver upgrade, a container library change or a scheduler change reduces throughput with no health incident at all — the case the chaos-engineering and capacity guides own, and the reason a performance baseline belongs in the observability design ([chaos_engineering_guide.md](chaos_engineering_guide.md), [capacity_sizing_guide.md](capacity_sizing_guide.md)).
- **The application fault in hardware clothing.** XIDs 13, 31 and 43 are application faults by the vendor's own trigger text, and 45 is usually just a Ctrl-C. A fleet that pages on every XID will page for user errors, then suppress the alert channel, then miss the real ones.

### 10.4 Distinguishing an application problem from a device problem

| Evidence | Points to application | Points to device |
|---|---|---|
| XID class | 13, 31, 43, 45 (vendor trigger text says user application / not a bug) | 48, 63/64, 79, 94, 95, 140 |
| Scope | One job or one pod, repeatedly, on different devices | The same device across different jobs and users |
| Diagnostic outcome | Diagnostics `Pass` on the affected device | Health incident and/or diagnostic `Fail` (`DCGM_ST_NVVS_ISOLATE_ERROR`) |
| Health state | `Healthy` throughout | `Warning`/`Failure`, or a historical incident in retention |
| Device counters | Flat ECC, throttle and remap counters | Rising ECC/remap; throttle-time growth for no new load |
| Reproducibility | Fails at a particular input or step | Fails at a random point, or hangs |

The deciding test in practice is **scope**: a device problem follows the *device*; an application problem follows the *job*. DCGM's per-entity health model plus the scheduler's placement history is what lets you tell them apart, and it is why `GPU_I_ID`/`UUID` labels matter more than node names.

### 10.5 The interaction with scheduling and drain decisions

Health and XID evidence is only actionable if it reaches the scheduler. The chain is: **DCGM condition → exported metric or event → alerting rule → scheduler action (cordon, drain, exclude) → workload resumption**. The DCGM-side inputs that justify intervening are the ones whose vendor action vocabulary says so: XID 154's recovery action (`Drain P2P`, `Drain and Reset`, `GPU Reset Required`, `Node Reboot Required`), XID 95 (uncontained, "the affected GPU must be reset before applications can restart"), XID 79 (`RESTART_BM`), and `DCGM_FR_FALLEN_OFF_BUS`. Everything else is an investigate-first signal. Note the asymmetry that trips teams up on MIG and shared platforms: **draining a node is a node-level action, but the fault is device-level.** On a node with one bad GPU out of eight, the cost of a full drain is high and the cost of leaving the bad device schedulable is higher; that trade-off is a platform policy decision, and it is why per-device cordon semantics (via the device plugin or DRA) are worth designing for rather than discovering during an incident.

### 10.6 The honest problem: most fleets discover their observability gaps during an incident

The pattern is near-universal: a fleet runs "GPU monitoring" that is really utilisation and temperature, a card degrades, an incident occurs, and the post-incident review reveals that the ECC counters were never exported, the XID gauge only ever showed the last code, the health watches were never enabled, and the baseline that would have shown the change never existed. Three countermeasures, all cheap before the incident and expensive after:

1. **Enable the health-relevant metric rows now, not later.** The default exporter set is a telemetry set (§4.3). Adding ECC, throttle-time, remap and NVLink error rows is a configuration change, not a project.
2. **Capture a baseline on day one.** A tag or snapshot of representative per-device counters per node, per driver version, per workload class. Without it, "is this normal?" is unanswerable — and every fleet eventually needs that answer.
3. **Test the failure path deliberately.** Inject a known condition (`dcgmi test` supports synthetic field injection for exercising monitoring, health and policy responses in an isolated environment ✅) and confirm that the alert fires, the runbook is followed, and the drain behaves. This is ordinary chaos engineering applied to the GPU layer; the discipline belongs to [chaos_engineering_guide.md](chaos_engineering_guide.md).

---
## 11. The Economics and the Capacity Angle

### 11.1 What the common utilisation metric does and does not mean

`DCGM_FI_DEV_GPU_UTIL` (field 203, integer percent) — and its ratio twin `DCGM_FI_DEV_GPU_UTIL_RATIO` (field 1613) — is the number most GPU dashboards are built on, and it is the most over-interpreted metric in the stack. NVIDIA's profiling documentation says the exporter's utilisation family "samples over a period that varies depending on the product" and, for the profiling metrics that are its better-posed cousins, warns that "a value of 0.8 or greater is necessary, but not sufficient, for effective use of the GPU. A value less than 0.5 likely indicates ineffective GPU usage." ✅ The consequences:

- **A GPU can be fully allocated and barely working.** 100 % `DCGM_FI_DEV_GPU_UTIL` means the device was busy, not that it was doing useful arithmetic. The vendor's own worked example shows why the distinction is real: GRACT of `0.845` after computing `2×42/98` for a MIG `3g.20gb` instance shows how a headline number is derived from geometry, not from useful work.
- **It is insensitive to *how* the work was done.** `DCGM_FI_PROF_SM_UTIL_RATIO`'s definition is explicit: "the fraction of time at least one warp was active on a multiprocessor" where "'active' does not necessarily mean a warp is actively computing. For instance, warps waiting on memory requests are considered active."
- **A low number is more informative than a high one.** The vendor's threshold language says a value below 0.5 "likely indicates ineffective GPU usage"; above 0.8 is necessary but *not* sufficient. So the metric's real value is as a **waste detector**, not as a work meter.
- **For shared and MIG fleets, per-card numbers mislead by construction.** The vendor's own example shows a GPU at 84.5 % while each of its two MIG instances reports ~98.6 % — the whole-GPU figure is the instance figure scaled by the compute fraction, and reading it without the geometry produces a wrong story.

### 11.2 Metrics that better reflect actual work

| Question | Better metric(s) | Why |
|---|---|---|
| Is the compute pipe doing work? | `DCGM_FI_PROF_SM_UTIL_RATIO` (1002), `DCGM_FI_PROF_SM_OCCUPANCY_RATIO` (1003) | Distinguishes "SMs busy" from "SMs occupied and spinning on memory" |
| Is the tensor pipe doing work? | `DCGM_FI_PROF_TENSOR_UTIL_RATIO` (1004) | The nearest thing to "is the AI work actually happening" |
| Is memory the constraint? | `DCGM_FI_PROF_DRAM_UTIL_RATIO`, `DCGM_FI_DEV_MEM_COPY_UTIL` (204) | Memory-bandwidth-bound workloads show here while SM activity stays high |
| Is the device being held back? | Throttle-time counters (`DCGM_FI_DEV_THERMAL_VIOLATION`, `…_POWER_VIOLATION`, and their seconds-valued twins) | Throttling is lost capacity you already paid for |
| Is the interconnect the constraint? | `DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL` / `…_THROUGHPUT_BYTES_PER_SECOND`, `DCGM_FI_PROF_PCIE_TX_BYTES`/`RX_BYTES` | Multi-GPU training is frequently non-GPU-bound |
| What did the useful work cost? | `DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION` (156) / `DCGM_FI_DEV_GPU_ENERGY_JOULES_TOTAL` (1611) differenced over an interval | The denominator for efficiency, and an input to sustainability reporting |
| Is the allocation *used*? | `DCGM_FI_DEV_FB_USED` (252) against `DCGM_FI_DEV_FB_TOTAL` (250), per allocation | The classic "reserved but unused" waste in shared platforms |

### 11.3 Using fleet data for capacity and cost decisions

Three defensible uses, each with a caveat:

1. **Right-sizing allocations.** Framebuffer-used versus framebuffer-reserved per workload, over weeks, tells you what the allocation *should* be. Caveat: `DCGM_FI_DEV_FB_RESERVED` exists precisely because reserved ≠ used, and reading `FB_USED` alone overstates consumption.
2. **Idle-capacity recovery.** Devices with sustained low SM activity and low framebuffer use are the recovery target — the "alert on idle GPU pools" case the platform guides name ([openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md)). Caveat: low utilisation during a *legitimate* phase (data loading, checkpointing, evaluation) is not waste, and a rule that cannot tell them apart will generate false positives on every training pipeline.
3. **Cost allocation and chargeback.** Per-namespace or per-tenant allocation and usage, aggregated over time, is the raw material for chargeback and for the FinOps conversation. Caveat: DCGM measures the *device*, not the business value; attribution to a cost centre comes from the scheduler and the platform's metadata, not from DCGM.

The honest caveat on all vendor-reported percentages: **a utilisation percentage excludes whatever the vendor's definition excludes**, and both `DCGM_FI_DEV_GPU_UTIL`'s sampling period and `DCGM_FI_PROF_*`'s interval-average semantics are definitional choices. A capacity decision built on a single headline percentage, without the accompanying profiling ratios and the allocation geometry, is a decision built on a number that was never designed to carry it. The sizing arithmetic itself belongs to [capacity_sizing_guide.md](capacity_sizing_guide.md); what DCGM supplies is the *measured* input, and the discipline is to state which field, over which interval, at which cadence, when quoting it.

---
## 12. The Regulated-Enterprise Angle

### 12.1 The evidence a GPU estate produces

The audit value of a DCGM-instrumented fleet is not "we have dashboards"; it is that the estate emits a machine-generated, timestamped record of three things:

| Evidence class | What DCGM contributes | Limits to state honestly |
|---|---|---|
| **Availability** | Per-device health state over time (`Healthy`/`Warning`/`Failure`), plus the incident codes that caused each transition, retained and exportable | Health is passive and rule-bound: a `Healthy` result means no enabled rule found an incident in retained data (§5.3), not that the device was proven good |
| **Fault history** | XID codes with timestamps, `DCGM_FR_*` incidents, remap and ECC counter history, diagnostic results with pass/fail per test and entity | XIDs alone are "a starting point for further investigation" (§7.1); diagnostic results are *online* and "do not… determine RMA eligibility" (§6.5) |
| **Capacity and utilisation** | Per-device, per-instance, per-tenant allocation and usage over time | Utilisation is definitional (§11.1); attribution to a cost centre comes from the scheduler, not from DCGM |

The design implication is that the *record* must be built deliberately: a health state queried on demand leaves no record, whereas a health state scraped continuously into a retention-controlled time series does. That is the difference between "we could have known" and "we can show what we knew, and when".

### 12.2 Feeding operational-resilience and vendor-management obligations

- **Operational resilience.** An important-business-service impact tolerance has to be supported by evidence that the underlying platform was monitored, that failures were detected, and that the response was defined. DCGM supplies the *detection* half (health incidents with severities; XID recovery actions; diagnostics) and the *response* half only once a human writes it down (§8.2). The framework, impact tolerances and incident-governance obligations are owned by the repository's operational-resilience content ([../../banking/operational_resilience_framework_guide.md](../../banking/operational_resilience_framework_guide.md)) — a GPU guide should not re-derive them.
- **Vendor management.** Monitoring capability becomes a contractual and service-management topic the moment the GPU estate is procured as a service or under support: what telemetry the supplier exposes, at what cadence, with what retention, and what their diagnostic and replacement obligations are. The DCGM-specific items to put in that conversation are concrete: the **DCGM and exporter version pairing** the supplier supports, the **diagnostic levels** they will run and when, the **XID resolution-bucket** workflow they follow, and whether **full offline diagnostics** (which DCGM explicitly does not replace) are part of their fault-determination process.
- **Capacity and resilience testing.** Diagnostic campaigns are, in effect, scheduled resilience testing of the accelerator layer. Keeping their results — node, date, level, tests run, pass/fail/skip, error codes — produces an evidence trail for both the resilience and the vendor-management narratives, and it costs nothing beyond storing JSON output.

### 12.3 The audit question: can you prove a device was healthy when it ran a workload that mattered?

This is the sharpest question a GPU estate will be asked, and the honest answer has three parts.

**What DCGM can support.** A defensible reconstruction: the device's health state before, during and after the workload window; any health incidents and their codes; XID codes with timestamps joined on GPU GUID; the diagnostic result nearest in time; ECC/remap/throttle counter deltas across the window; and the driver and DCGM/exporter versions in force. With continuous export and retention aligned to the workload's retention obligation, that is a real evidentiary record.

**What DCGM cannot support.** It cannot prove the workload was *correct*; it cannot prove the device was *capable* at that moment unless a diagnostic ran then; it cannot substitute for offline field diagnostics or for RMA determination; and it cannot attribute a device fault to a specific job without the scheduler's placement data. **The join is the hard part**: GPU GUID → node → pod → job → tenant is a chain across three systems (DCGM, the scheduler, the platform metadata), and a fleet that cannot perform that join cannot answer the audit question at all, no matter how many metrics it collects.

**The design consequences.** Store the GUID as a first-class identifier everywhere (not the index, not the PCI address); retain health state long enough to cover the obligations it will be asked about; record version metadata (driver, DCGM, exporter, catalogue revision) alongside the evidence, because XID catalogue `Applies to` columns and field semantics are version-scoped; and keep diagnostic results as structured output (`--json`) rather than console logs, since the exit statuses and codes are the machine-readable part.

### 12.4 Segregation between tenant-visible and platform-visible telemetry

On a shared platform, telemetry is tenant data as much as it is platform data, and it should be governed accordingly: tenant-facing series are the tenant's resource envelope and availability; platform-only series are device health, topology, peer links, node identity and co-tenancy (§9.6). Two controls are worth naming explicitly. **Enforce by data path, not by dashboard filter** — separate scrape/aggregation for tenant-facing series, because label filtering fails open. **Classify whole-GPU metrics on shared cards as potentially disclosing**, since aggregate temperature, power or utilisation on a MIG-partitioned card reveals activity that is not solely the tenant's own. Where tenants are, or include, supervised counterparties, this is a data-sharing question rather than a monitoring-preference question, and it belongs in a data-classification review and a tenancy agreement, not only in a Grafana folder permission.

### 12.5 Vendor-concentration risk of a vendor-specific monitoring layer

DCGM is NVIDIA-specific by construction, and that concentration deserves to be named rather than assumed away.

| Concentration | The exposure | The proportionate mitigation |
|---|---|---|
| **Instrumentation** | Every GPU health signal, alert rule, dashboard and runbook is written against NVIDIA's field identifiers and XID codes. A second accelerator vendor means a second, parallel observability stack | Keep platform-level SLOs vendor-neutral (availability, queue time, throughput, cost per unit of work) and let DCGM supply the device-level detail beneath them |
| **Semantics** | Metric definitions, XID catalogues and diagnostic behaviour are vendor-controlled and version-scoped; they can change between releases | Pin versions, re-verify field and catalogue semantics on upgrade, and never make a metric definition the sole basis of a customer commitment |
| **Licence and lifecycle** | DCGM source is Apache-2.0 but the shipped images are under NVIDIA's product terms (§15), and DCGM's own end-of-life and support policy is the vendor's to set | Track the vendor's support/EOL documentation as a vendor-management item rather than discovering it during an incident |
| **Evidence** | Device-level fault determination runs through the vendor's own offline process | Keep the *platform-side* evidence (health history, diagnostics, versions) in your own systems, so an audit does not depend on vendor tooling still being available |

None of these argue against DCGM — it is the only layer that scales to a GPU fleet of any size, and the alternatives are weaker for NVIDIA hardware. They argue for keeping the *service* commitments vendor-neutral and the *device* detail vendor-specific, which is a design decision made once and revisited at every accelerator-platform change.

---
## 13. Worked Example: The Cymbal Bank Shared GPU Platform

⚠️ **Everything in this section is fictional and illustrative.** The fleet, the figures, the quotations and the cost numbers are invented for Cymbal Bank, a fictional institution. They are shaped to be arithmetically consistent so the *reasoning* can be followed, not to represent any real vendor pricing, market rate or observed fleet.

### 13.1 The situation

Cymbal Bank is standing up a shared GPU platform for model training and inference: three clusters, two regions, serving a model-risk team (training and fine-tuning), a document-intelligence team (batch inference), and a real-time assistant team (online inference). The platform team has to decide **what to monitor, who sees it, what to alert on, and how to handle a device that fails a diagnostic** — and has to be able to defend those choices to internal audit and to the bank's operational-resilience function.

### 13.2 The fleet inventory (illustrative)

| Cluster | Nodes | GPUs per node | Interconnect | Tenancy | Workloads |
|---|---|---|---|---|---|
| `train-a` (metal, region A) | 16 | 8 | NVLink/NVSwitch fabric | Dedicated, non-shared | Training and fine-tuning jobs (multi-day) |
| `serve-a` (region A) | 12 | 4 | PCIe | MIG-shared (4 instances per GPU) | Batch document inference, offline scoring |
| `serve-b` (region B) | 8 | 4 | PCIe | Time-sliced | Real-time assistant inference |

Total: 36 nodes, 208 devices, plus 48 MIG instances on `serve-a`. At the vendor's documented cadence defaults, monitoring is node-local service plus one exporter pod per node; the estate is small enough that a single Prometheus pair can hold it, which matters for the cardinality arithmetic below.

### 13.3 Choosing what to monitor — and the reasoning

The team starts from the requirement, not from the metric list: *the platform must be able to detect device degradation before it causes a workload failure, prove device health when a workload failed, and report capacity honestly.* That requirement resolves into four metric groups, and the team explicitly rejects a fifth.

| Group | Selected metrics (exporter configured names) | Why |
|---|---|---|
| **Device health** | `DCGM_FI_DEV_ECC_SBE_VOL_TOTAL`, `DCGM_FI_DEV_ECC_DBE_VOL_TOTAL`, `DCGM_FI_DEV_ECC_SBE_AGG_TOTAL`, `DCGM_FI_DEV_ECC_DBE_AGG_TOTAL`, `DCGM_FI_DEV_UNCORRECTABLE_REMAPPED_ROWS`, `DCGM_FI_DEV_CORRECTABLE_REMAPPED_ROWS`, `DCGM_FI_DEV_ROW_REMAP_FAILURE`, `DCGM_FI_DEV_XID_ERRORS`, plus exporter-owned `DCGM_EXP_XID_ERRORS_TOTAL` and `DCGM_EXP_GPU_HEALTH_STATUS` | These are the rows shipped **commented out** by default (§4.3). The team enables them explicitly rather than discovering later that the default set has no health signal in it |
| **Constraint and throttle** | `DCGM_FI_DEV_THERMAL_VIOLATION`, `DCGM_FI_DEV_POWER_VIOLATION`, `DCGM_FI_DEV_PCIE_REPLAY_COUNTER` | Distinguishes "slow" from "held back", and catches the PCIe quality class (vendor health rule: >8 replays/min) including its correlated XID 32 |
| **Capacity and use** | `DCGM_FI_DEV_FB_USED`, `DCGM_FI_DEV_FB_FREE`, `DCGM_FI_DEV_FB_RESERVED`, `DCGM_FI_DEV_GPU_UTIL`, `DCGM_FI_PROF_SM_UTIL_RATIO`, `DCGM_FI_PROF_TENSOR_UTIL_RATIO`, `DCGM_FI_PROF_DRAM_UTIL_RATIO`, `DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION` | Framebuffer reserved-vs-used is the waste signal on shared nodes; the profiling ratios bring in the profiler requirement |
| **Fabric (train cluster only)** | `DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL`, `DCGM_FI_DEV_SM_CLOCK`, `DCGM_FI_DEV_MEM_CLOCK`, `DCGM_FI_DEV_GPU_TEMP`, `DCGM_FI_DEV_MEMORY_TEMP`, `DCGM_FI_DEV_POWER_USAGE`, `DCGM_FI_DRIVER_VERSION` (label) | NVSwitch fabric is where multi-day training dies; clocks, temperatures and power are the context that explains why a job slowed |
| **Profiling counters on `serve-b`** | *not enabled* | Rejected — see below |

**What the team deliberately chose NOT to monitor, and why:**

- **Per-lane NVLink fields (18 lanes × 128 GPUs on `train-a`).** The team enables the aggregate NVLink bandwidth field, not the L0–L17 per-lane set. The rationale is the cardinality arithmetic in §4.6: per-lane series would add `18 × 128 = 2,304` time series for a signal whose actionable form is an aggregate plus the health watch's own rate rules. If per-lane diagnosis is needed, it is run on demand with `dcgmi nvlink` / `dcgmi dmon` on the suspect node rather than stored fleet-wide.
- **Per-process and per-job statistics fleet-wide.** Workload attribution comes from the scheduler's metadata joined on GPU UUID, and pulling per-process statistics from every device on every node is a cost with no corresponding alert.
- **Profiling counters on the real-time path.** The team declines `DCGM_FI_PROF_*` on `serve-b` for two reasons: the counters are a shared resource that conflicts with developer profilers, and pausing or resuming profiling is **host-engine-wide** (§4.5) — a profiling collector on the low-latency cluster is a control surface the on-call team does not want to hold. `serve-b` keeps utilisation and framebuffer metrics only.
- **Temperature as a page.** Temperature is collected and charted, but not paged. The page is on throttle-time *accumulation* and on health state, because a hot-but-unthrottled card is not an incident and a paging rule on absolute temperature generates noise in every warm rack.

### 13.4 The alert set — with the reasoning

| Alert | Condition | Severity | Reasoning |
|---|---|---|---|
| `GpuHealthDegraded` | `DCGM_EXP_GPU_HEALTH_STATUS` indicates degraded, for 5 min | **Page** | The vendor's own health judgement, sustained |
| `GpuXidFailureClass` | `DCGM_EXP_XID_ERRORS_TOTAL` increase, and the code's catalogue bucket is a `RESET_GPU`/`RESTART_BM` class (48, 62, 64, 79, 94, 95, 119, 120, 140) | **Page** | Bucket-first triage (§7.4). The rule does not page on XID 13/31/43/45, whose vendor trigger text calls them application or user/system events |
| `GpuRecoveryActionChanged` | XID 154 reported with a non-`None` recovery action | **Page + cordon** | XID 154 *is* the recovery-action summary; a `Node Reboot Required` state makes the node unschedulable by construction |
| `GpuUncorrectableEcc` | Any increase in `DCGM_FI_DEV_ECC_DBE_VOL_TOTAL` or `DCGM_FI_DEV_ECC_DBE_AGG_TOTAL` | **Page** | Uncorrectable errors; correlates with XID 48 |
| `GpuRowRemapFailed` | `DCGM_FI_DEV_ROW_REMAP_FAILURE` = 1 | **Page** | The vendor's own flag; correlates with XID 64 |
| `GpuEccCorrectedRateRising` | `rate(DCGM_FI_DEV_ECC_SBE_VOL_TOTAL[1h])` above the fleet baseline for that card model | **Ticket** | Leading indicator, not a failure — but the one that predicts the page three weeks later |
| `GpuRemapCapacityConsuming` | `increase(DCGM_FI_DEV_UNCORRECTABLE_REMAPPED_ROWS[24h]) > 0` | **Ticket** | Finite capacity being consumed |
| `GpuThrottleTimeGrowing` | `increase(DCGM_FI_DEV_THERMAL_VIOLATION[1h])` > threshold, or power equivalent | **Ticket** | Lost capacity that was paid for |
| `GpuPcieReplayRate` | `rate(DCGM_FI_DEV_PCIE_REPLAY_COUNTER[1m])` above the vendor health rule's 8/min | **Ticket** | DCGM's own health threshold; the correlated failure is XID 32 |
| `NodeGpuScrapeDown` | `up{job="dcgm-exporter"} == 0` for 5 min | **Page** | An unmonitored GPU node is a silent risk; this alert protects the observability plane itself |
| `TenantAllocationReservedUnused` | `DCGM_FI_DEV_FB_RESERVED` high against `DCGM_FI_DEV_FB_USED` per MIG instance, for 7 days | **Review** | Recovery of over-sized allocations; a monthly review item, never a page |
| `GpuUtilisationSustainedHigh` | `DCGM_FI_DEV_GPU_UTIL` > 95 % for a training window | **None — dashboard only** | Deliberately not an alert: high utilisation is the platform working. This is the alert most teams add and later remove |

### 13.5 The tenancy and visibility model

Three tiers, enforced by data path rather than dashboard configuration (§9.6):

| Tier | Audience | Series | Control |
|---|---|---|---|
| **Tenant** | Model-risk, document-intelligence and assistant teams | Per-allocation utilisation, framebuffer used, and `GPU_I_PROFILE`/`GPU_I_ID`-scoped series plus availability | A separate scrape/aggregation path scoped to tenant-relevant series, keyed on the scheduler's namespace/pod mapping |
| **Platform** | GPU platform team and SRE | Everything above, plus device health, ECC and remap counters, throttle time, NVLink aggregates, XID codes, node identity and co-tenancy | The primary Prometheus, restricted by normal platform access controls |
| **Restricted** | Internal audit, risk, and the vendor-management function | Frozen extracts: health-state history, XID history with catalogue bucket and version, diagnostic results, version metadata | Extract-and-store, not live dashboards — because the audit question (§12.3) is about a window that has already passed |

The design decision the team writes down explicitly: **whole-GPU metrics on `serve-a`'s MIG-shared cards are platform-tier only**, because aggregate utilisation or power on a partitioned card discloses activity beyond the requesting tenant's own share. That decision is recorded in the platform's data-classification review, not left to a Grafana folder permission.

### 13.6 The failure-handling runbook

Walked through for the case that matters: a device that fails a diagnostic.

1. **Detect.** `GpuHealthDegraded` or `GpuRecoveryActionChanged` pages the on-call SRE. The alert carries node, GPU UUID and the `DCGM_FR_*` code or XID code.
2. **Assess (never diagnose from the code alone).** Read the XID's catalogue row: description, `Applies to` for the GPU generation, and the **immediate action bucket**. This single step prevents the two classic errors — treating an application fault (13/31/43/45) as hardware, and treating an interconnect report as the reporting GPU's fault (XID 74).
3. **Cordon first.** Any `Drain P2P`, `Drain and Reset`, `GPU Reset Required` or `Node Reboot Required` recovery action (XID 154) cordons the node automatically: reversible, logged, observable, per §8.2. Where the fault is one GPU on a multi-GPU node, the cordon is scoped to the device via the device plugin/DRA where the platform supports it.
4. **Preserve evidence.** Capture the health report (`dcgmi health --check --json`), the kernel-log lines matching `NVRM: Xid` for that GUID, and `nvidia-bug-report.sh` output if a vendor escalation is likely. Retention matters: the health watch's own retention window is finite (§5.4), so export before clearing.
5. **Diagnose — only on a drained node, only with an explicit level.** Post-incident and with the node out of service, the runbook prescribes a **medium (`2`) suite** as the first active test, because the vendor's own epilogue guidance places a medium suite after a failed workload at ~2 minutes. Escalation to level `3` requires a maintenance window; level `4` (`memtest`, `pulse_test`) requires the platform lead's approval and a named window, because it is the intrusive level.
6. **Interpret correctly.** `Skip` is treated as `Unknown`, never `Pass`. Exit status `205` (`DCGM_ST_NVVS_ISOLATE_ERROR`) isolates immediately without waiting for a human interpretation — DCGM has already said the device should be out of service.
7. **Decide.** Diagnostic `Fail` plus a health incident → the device stays out of service and the hardware vendor is engaged, with the DCGM evidence as input. Diagnostic `Fail` with a `healthy` health history → the diagnostic is the stronger evidence about current capability (§6.4), so the device stays out until the failing test is investigated. Diagnostic `Pass` with a health `Warning` → the incident is **not** closed on the pass alone; the device returns to service under observation with watches enabled, and the incident closes only after a clean observation window.
8. **Return to service.** `dcgmi diag --run 1` as the prologue gate, watches re-enabled, baseline re-captured, and the incident record closed with the evidence attached.

### 13.7 The diagnostic policy

| Occasion | Level | Window | Approval |
|---|---|---|---|
| Node acceptance into the pool | `1`, then `3` once per node | Maintenance window | Platform engineering |
| Before a multi-day training job (prologue) | `1` | None needed (seconds) | Automated |
| After a failed workload, node idle (epilogue) | `2` | None needed beyond the drain | On-call |
| Incident triage with a specific suspicion | Named test (`pcie`, `nvbandwidth`, `memtest`, `pulse_test`) | Drained node | On-call, platform lead for `memtest`/`pulse_test` |
| Rack fabric validation | `dcgmi mndiag` (`nvloom`, `mnnvbandwidth`) | Quarterly maintenance window | Platform lead + network team |

The policy states what diagnostics are **not**: they are online tests run in the installed environment, they do not repair faults, they do not replace NVIDIA's offline field diagnostics, and they do not determine RMA eligibility ✅. The bank's asset-disposal and vendor-engagement controls sit outside this runbook, and the runbook says so — which is what makes it defensible to an auditor.

### 13.8 A cost comparison — clearly illustrative

⚠️ **Fictional figures, invented for this example.** They are chosen to demonstrate the *shape* of the trade-off, not to price anything.

| Approach | What you get | Illustrative annual effort/cost | What breaks |
|---|---|---|---|
| **A. No DCGM: `nvidia-smi` scripts over SSH** | Point-in-time values, per node | ~0.5 FTE of script maintenance; no licence cost | No history, no health model, no fleet view. A degradation that takes weeks to manifest is invisible until it fails a job. This is the baseline the bank is leaving |
| **B. DCGM + exporter + Prometheus/Grafana, defaults only** | The 26 shipped rows: clocks, temperatures, power, utilisation, framebuffer, XID last-value, three remap rows, three profiling ratios | ~0.3 FTE operation; existing monitoring platform absorbs the series | **No ECC, no throttle time, no NVLink errors** — the health signal is absent (§4.3). Produces dashboards that look complete and cannot answer a health question |
| **C. DCGM + exporter with the health rows enabled + alerts + runbook + diagnostic policy** (recommended) | B, plus ECC and remap counters, throttle time, PCIe replay rate, NVLink aggregates, exporter-owned health and XID-count families, an alert set, a documented runbook and a diagnostic schedule | ~0.6 FTE operation — roughly 0.3 FTE more than B, spent on alert tuning, baseline capture and the quarterly diagnostic campaign | Nothing about this approach is free: alert thresholds must be tuned per card model, and the diagnostic campaign needs windows |
| **D. C plus per-lane NVLink, per-process statistics and profiling on every cluster** | Everything in C, at the finest available granularity | ~1.2 FTE operation plus a materially larger time-series footprint (the per-lane arithmetic in §4.6) | Cardinality and cost grow faster than insight. The extra series are diagnostic, not alerting — better obtained on demand |

**The recommendation: C.** B is a false economy that the bank would discover during its first incident (§10.6). D is a research team's configuration imposed on an operations team: it triples the observability cost for signals that are better fetched on demand from the suspect node. C is the point where the platform can answer all three of the questions it is asked — *is this device degrading*, *was this device healthy when the job failed*, and *what capacity do we actually have* — at a cost the platform can staff.

### 13.9 The recommendation, restated in one paragraph

Cymbal Bank's shared GPU platform runs the NVIDIA GPU Operator on every GPU node, with DCGM and DCGM Exporter Operator-managed and pinned as a matching version pair; the exporter's collector configuration is a reviewable artefact that enables the health rows the defaults omit; one Prometheus carries the platform tier and a separate, tenant-scoped path serves team dashboards; health and XID evidence is joined to the scheduler on GPU UUID so a device fault can be traced to a job and a tenant; alerting is bucket-first on XIDs and rate-based on counters, with health state and throttle accumulation as the two paging signals and utilisation deliberately excluded from paging; automated response is limited to reversible cordoning on explicit recovery actions, with every action emitting an event; diagnostics follow a written policy with named levels, windows and approvals, and their JSON results are retained; and every fact in this guide that could change — field semantics, diagnostic behaviour, the XID catalogue, version pairings and licence terms — is re-verified at each driver and platform upgrade rather than assumed to have stayed still.

---
## 14. Gotchas and Anti-Patterns

| Symptom | Cause | Guardrail |
|---|---|---|
| Dashboards silently changed meaning after an upgrade | Alert rules on metrics whose **definition or field ID changed**. DCGM 4.7 added base-unit field IDs 1600–1650 and canonical aliases for fields whose names did not identify their unit, value kind or status semantics; the vendor's warning is explicit that "code that changes IDs must also change its value-type and scaling assumptions", and its `dmon` guidance says "do not infer units or counter behavior from a short `dmon` column name" | Pin the producing release; re-verify field semantics and units at every upgrade; express thresholds in units the field's documented contract guarantees; treat field-ID changes as a breaking change with a test |
| "The GPUs are at 100 % utilisation" but throughput is poor | Alerts on a metric that reports **allocation rather than work**. `DCGM_FI_DEV_GPU_UTIL` reports integer percent; the vendor notes the sample period "varies depending on the product", and for the profiling cousins that "active" does not mean "actively computing", that ≥0.8 is "necessary, but not sufficient", and that <0.5 "likely indicates ineffective GPU usage" | Alert on health and throttle signals, not utilisation; pair `DCGM_FI_DEV_GPU_UTIL` with `DCGM_FI_PROF_SM_UTIL_RATIO`, `…_TENSOR_UTIL_RATIO` and `…_DRAM_UTIL_RATIO`; on MIG, read instance values with the geometry |
| A node that was serving tenants suddenly lost state | **Diagnostics run on nodes in service.** NVIDIA: diagnostics "can consume GPU compute, memory, PCIe, NVLink, CPU, power, and cooling capacity. Longer suites can reset state or conflict with running processes", and the pre-flight requires draining applications and peer users from the selected entities | Diagnose only on drained nodes; enforce level/window/approval in a written diagnostic policy (§6.5, §13.7); automate invasive levels never, and cordon-only actions only when reversible |
| Prometheus is timing out, queries are slow, memory is climbing | **Scraper cardinality explosion.** Per-lane NVLink fields, per-MIG-instance samples and per-process statistics multiply series; the documented cost levers are more entity-field combinations, shorter intervals, longer retention, more clients and larger result sets | Do the series arithmetic before enabling (§4.6); keep `scrape_timeout` below both `scrape_interval` and the exporter's `--web-write-timeout`; choose the slowest cadence that meets the response requirement; use a separate, scoped path for tenant series |
| A tenant dashboard shows another tenant's GPU activity | **A filtered view of one shared data source.** Label filtering in a single Grafana is a configuration guarantee, not a technical one, and it fails open. NVIDIA also notes that mapping mechanisms "add labels; they do not make an unsupported DCGM field available" | Serve tenants from a separate scrape/aggregation path scoped to their allocation; classify whole-GPU metrics on shared cards as disclosing; record the decision in a data-classification review (§9.6, §12.4) |
| The team pages on every XID, then mutes the channel | **XIDs triaged without the vendor's reference.** XIDs 13, 31 and 43 are application faults by the vendor's own trigger text; 45 is usually a Ctrl-C or sigkill; 63 is informational. Meanwhile the codes with `RESET_GPU`/`RESTART_BM` immediate buckets are the ones that need action | Triage bucket-first, description second; page only on failure-class buckets; keep the catalogue pinned and versioned; never interpret a code without its source (§7.3–7.4) |
| Metrics that used to appear are now blank or missing, with no error | **The exporter left un-upgraded against a newer driver or DCGM.** "Run DCGM Exporter with the DCGM version paired with that exporter release. Mismatched combinations might function, but they are not tested or supported." Also: "selecting a row does not guarantee runtime emission", and fields are hardware-, driver- and permission-dependent | Treat the DCGM/exporter pair as a versioned artefact upgraded together (§9.1); alert on exporter scrape health; verify the effective `/metrics` output rather than the configured CSV; keep the health-relevant rows enabled so a silent loss is visible |
| Nobody could tell when behaviour changed — because nobody recorded what it was before | **No baseline.** Health detects conditions, not regressions (§5.3); a fleet with no captured baseline cannot answer "is this normal", and most fleets discover the gap during an incident (§10.6) | Capture per-node, per-card-model, per-driver baselines at bring-up and at every upgrade; persist counter history (ECC, remap, throttle time) rather than reading it on demand; test the failure path deliberately with synthetic injection and the discipline in [chaos_engineering_guide.md](chaos_engineering_guide.md) |
| Health checks keep returning `Healthy` during an incident | Retention and cadence misunderstood: watches need "approximately 60 seconds of samples" before a first meaningful check, `--max-keep-age` defaults to 600 s, and a short retention "can discard the evidence for an earlier incident" | Export health state continuously instead of querying on demand; set retention to cover the consumer's read latency and the analysis window; re-create watches after any host-engine restart (they are not durable) |
| Every policy vanished from a host engine after a deployment | **`dcgmi policy --import` replaces the whole registry rather than merging**, and DCGM 4.7 "can replace the registry with an empty or partially valid import" | Validate generated policy files, import, then verify with `--list`; treat the policy registry as state that needs a backup and a post-deploy assertion |
| Profiling metrics went blank fleet-wide and stayed blank | A developer profiler took the counters and the host-engine-wide `dcgmi profile --pause` was never followed by `--resume` | Make pause/resume a paired operation in the profiling workflow, with a monitoring check for paused state; remember the pause is host-engine-wide, not per GPU |

---
## 15. The Claims Audit

Every high-risk class — metric identifiers, diagnostic levels, XID codes, versions, licences — audited against the source read on **19 September 2026**. Status key: **Verified** (read from the vendor's own material during this research), **Flagged** (documented but version-, hardware- or context-dependent, or an inference this guide labels as such), **Rejected** (not asserted in this guide).

| # | Claim | Status | Source | Source date / version | Quality note |
|---|---|---|---|---|---|
| 1 | `DCGM_FI_DEV_GPU_TEMP_CELSIUS` is field **150** | Verified | DCGM *Field Identifiers* | doc set `latest`, release notes open at DCGM 4.7.0; checked 19 Sep 2026 | The vendor uses this very field as its worked example |
| 2 | `DCGM_FI_DEV_SM_CLOCK`/`_MEM_CLOCK`/`_VIDEO_CLOCK` are **100/101/102**; hertz variants **1603/1604/1605** | Verified | DCGM *Field Identifiers* (base-unit tables) | 4.7.0; 19 Sep 2026 | Base-unit fields are "distinct field IDs, not aliases" |
| 3 | Field **203** remains integer-percent `DCGM_FI_DEV_GPU_UTIL`; field **1613** is double `DCGM_FI_DEV_GPU_UTIL_RATIO` | Verified | DCGM 4.7.0 release notes + *Field Identifiers* | 4.7.0; 19 Sep 2026 | Two independent pages agree |
| 4 | `DCGM_FI_DEV_FB_TOTAL/_FREE/_USED/_RESERVED` = **250/251/252/253**; byte variants **1625–1628** | Verified | DCGM *Field Identifiers* | 4.7.0; 19 Sep 2026 | `FB_RESERVED` exists to distinguish usage from reservation |
| 5 | ECC totals `…_SBE_VOL_TOTAL`/`…_DBE_VOL_TOTAL`/`…_SBE_AGG_TOTAL`/`…_DBE_AGG_TOTAL` = **310/311/312/313** | Verified | DCGM *Field Identifiers* | 4.7.0; 19 Sep 2026 | — |
| 6 | `DCGM_FI_DEV_ROW_REMAP_UNCORRECTABLE_TOTAL`/`…_CORRECTABLE_TOTAL`/`…_FAILED`/`…_PENDING` = **393/394/395/396** | Verified | DCGM *Field Identifiers* | 4.7.0; 19 Sep 2026 | `…_FAILED` is the vendor's remap-failure flag |
| 7 | Throttle-time seconds fields are **1617–1624**, paired with legacy nanosecond fields **240–247** | Verified | DCGM *Field Identifiers* | 4.7.0; 19 Sep 2026 | Conversion is ÷1,000,000,000 from the legacy counter |
| 8 | `DCGM_FI_DEV_XID_ERROR` is field **230** and documented as the **last** XID encountered | Verified | `NVIDIA/dcgm-exporter` `etc/default-counters.csv` + DCGM *DCGM Exporter Metrics* | 19 Sep 2026 | Gauge-of-last-value semantics make it unsuitable as a primary XID alert |
| 9 | Exporter's shipped default collector CSV has **26 active rows** (25 metric families + 1 label) | Verified | DCGM *DCGM Exporter Metrics* reference **and** raw `etc/default-counters.csv` | 19 Sep 2026 | Two independent sources; the count is of active (uncommented) rows |
| 10 | Profiling IDs `DCGM_FI_PROF_GR_ENGINE_UTIL_RATIO` **1001**, `…_SM_UTIL_RATIO` **1002**, `…_SM_OCCUPANCY_RATIO` **1003**, `…_TENSOR_UTIL_RATIO` **1004**, `…_FP64_UTIL_RATIO` **1006**, `…_FP32_UTIL_RATIO` **1007** | Verified | DCGM *Profiling* learn page (device-level metrics table) | 4.7.0; 19 Sep 2026 | — |
| 11 | Exporter configured names differ from canonical field names (`DCGM_FI_DEV_GPU_UTIL`→`…_UTIL_RATIO`, `…_XID_ERRORS`→`…_XID_ERROR`, `…_PCIE_REPLAY_COUNTER`→`…_REPLAY_TOTAL`, `…_NVLINK_BANDWIDTH_TOTAL`→`…_THROUGHPUT_TOTAL`, `DCGM_FI_PROF_GR_ENGINE_ACTIVE`→`…_GR_ENGINE_UTIL_RATIO`) | Verified | DCGM *DCGM Exporter Metrics* ("Configured name" → "Canonical DCGM field" table) | 19 Sep 2026 | The single most likely source of broken dashboards |
| 12 | Diagnostic suites: `1` seconds, `2` ≈ 2 min, `3` ≈ 15 min, `4` longer-running; aliases `quick`/`short`/`medium`/`long`/`xlong`; `production_testing` is not a numbered level and does **not** add level-4 `memtest`/`pulse_test` | Verified | `dcgmi diag` command reference | 4.7.0; 19 Sep 2026 | Runtimes are the vendor's "normally lasting" characterisations, not SLAs — flagged as such in §6.1 |
| 13 | Diagnostic plugins include `software`, `context_create`, `memory`, `pcie`, `diagnostic`, `memory_bandwidth`, `targeted_stress`, `targeted_power`, `nvbandwidth`, `memtest`, `pulse_test` | Verified | DCGM *Diagnostics* learn page (selection table) | 4.7.0; 19 Sep 2026 | Names are canonical test selectors |
| 14 | Diagnostic exit statuses `226`/`217`/`215`/`205`/`204`/`203`/`198` map to the documented `DCGM_ST_*` names | Verified | `dcgmi diag` reference (EXIT STATUS) | 4.7.0; 19 Sep 2026 | `205` = "requires isolation" is the strongest vendor signal |
| 15 | Health `--set` selectors are `p`/`m`/`i`/`t`/`n`/`d`/`e`/`x`/`a`; defaults `--update-interval` 30 s, `--max-keep-age` 600 s | Verified | `dcgmi health` reference + Health Monitoring learn page | 4.7.0; 19 Sep 2026 | `a` must be used alone; SM is displayable but not settable via `--set` |
| 16 | Health incidents include `DCGM_FR_FALLEN_OFF_BUS`, `DCGM_FR_XID_ERROR`, `DCGM_FR_FAULTY_MEMORY`, `DCGM_FR_PCI_REPLAY_RATE` (>8 replays/min), `DCGM_FR_NVLINK_CRC_ERROR_THRESHOLD` (>100 CRC/s), `DCGM_FR_NVLINK_ERROR_CRITICAL`, the `DCGM_FR_GPU_RECOVERY_*` family, `DCGM_FR_NVSWITCH_*`, `DCGM_FR_FIELD_VIOLATION`, `DCGM_FR_IMEX_UNHEALTHY` | Verified | Health Monitoring learn page (Failure Conditions) + DCGM 4.7.0 release notes for the IMEX additions | 4.7.0; 19 Sep 2026 | `DCGM_FR_*` is also used by the diagnostic error catalogue — a different catalogue |
| 17 | Health results are `Healthy`/`Warning`/`Failure`, and `Healthy` "does not mean that every subsystem was watched or that an active stress test passed" | Verified | Health Monitoring learn page + `dcgmi health` reference | 4.7.0; 19 Sep 2026 | The single most important limitation to quote |
| 18 | XIDs are NVIDIA-driver error reports indicating a hardware, NVIDIA-software or user-application problem; meanings are consistent across driver versions; they are debugging guides, not diagnoses | Verified | XID Errors *Introduction* | `docs.nvidia.com/deploy/xid-errors/`, 19 Sep 2026 | Basis for this guide's refusal to over-interpret codes |
| 19 | XID codes discussed: **8, 11, 13, 14, 25, 31, 32, 37, 38, 39–41, 43, 45, 46, 48, 62, 63, 64, 70–72, 74, 75–77, 78, 79, 80, 85, 92, 94, 95, 119, 120, 140, 144, 154** — with mnemonics, descriptions, `Applies to` columns and immediate-action buckets | Verified | XID Errors, *Analyzing Xid Errors with the Xid Catalog* | catalogue page + `Xid-Catalog.xlsx`, checked 19 Sep 2026 | A **selection**, explicitly not the catalogue; rows are hardware-scoped and version-linked |
| 20 | XID 154 summarises the recovery action; expected values **None, Drain P2P, Drain and Reset, GPU Reset Required, Node Reboot Required** | Verified | XID catalog, row 154 | 19 Sep 2026 | The most operationally useful row; drives cordon decisions |
| 21 | XID 79 = "GPU has fallen off the bus", immediate bucket `RESTART_BM`; XID 48 needs a GPU reset or node reboot to clear; XID 74 may indicate a fault at the *remote* end of the link | Verified | XID catalog rows 79, 48, 74 | 19 Sep 2026 | Practical consequence: do not replace the reporting GPU reflexively |
| 22 | SXid messages apply only to Hopper and earlier; SXids are documented in the Fabric Manager User Guide | Verified | XID Errors *Introduction* | 19 Sep 2026 | — |
| 23 | Policy is evaluated against numeric fields with operators `>`, `>=`, `<`, `<=`, `==`, `!=`, `changed`; targets all GPUs or a list; channels console/file/callback with per-entity rate limiting; legacy interface watches seven fixed conditions | Verified | DCGM *Policy* learn page + `dcgmi policy` reference | 4.7.0; 19 Sep 2026 | `--import` replaces the whole registry (a flagged hazard) |
| 24 | The vendor's guidance places response logic (drain, reset, diagnose, page, correlate) in an **external controller**, not in DCGM | Verified | DCGM *Policy* learn page | 4.7.0; 19 Sep 2026 | Directly supports §8.2's recommendation |
| 25 | Legacy policy action/validation values "do not reset a GPU or run a diagnostic" | Verified | DCGM *Policy* learn page (migrating from legacy group policies) | 4.7.0; 19 Sep 2026 | A common misreading of legacy policy config |
| 26 | Exporter serves `/metrics` on port **9400** by default; collection interval default **30000 ms**; device-string syntax `[f] \| [g[:id…]] \| [i[:id…]]` | Verified | NVIDIA GPU Telemetry *DCGM Exporter* page + DCGM *Install DCGM Exporter* | GPU Telemetry doc set 1.0.0; DCGM 4.7.0; 19 Sep 2026 | The GPU Telemetry doc set is the older of the two; the DCGM doc set is authoritative where they differ |
| 27 | DCGM Exporter defaults to an **embedded host engine** and can instead connect over TCP, Unix socket or VSOCK; the exporter's `libdcgm` client version must be ≥ the host engine version | Verified | DCGM *Install DCGM Exporter* + GPU Telemetry *DCGM Exporter* | 19 Sep 2026 | Directly explains the "blank metrics after a partial upgrade" failure |
| 28 | The Helm chart enables a `ServiceMonitor` by default; it assumes unauthenticated HTTP; `serviceMonitor.enabled=false` is the documented workaround | Verified | DCGM *Configure Prometheus for DCGM Exporter* + *Install DCGM Exporter* | 4.7.0; 19 Sep 2026 | Three independent selector gates must all pass |
| 29 | GPU Operator 26.7 deploys driver, container toolkit, device plugin, **DCGM Exporter** and MIG Manager as pods; verification is `kubectl get pods -n gpu-operator` and `clusterpolicy` `ready` | Verified | GPU Operator 26.7 *Installing the NVIDIA GPU Operator* | 26.7 (patch v26.7.0); 19 Sep 2026 | Includes `nvidia-dcgm-exporter` in the documented pod list |
| 30 | DCGM **source** in `NVIDIA/DCGM` is Apache-2.0 | Verified | `NVIDIA/DCGM` README "License" section + repository licence listing | 19 Sep 2026 | Applies to the repository source, not to the packaged binaries |
| 31 | DCGM Exporter **source** is Apache-2.0 | Verified | `NVIDIA/dcgm-exporter` `LICENSE` (Apache License 2.0 full text) + README licence badge | 19 Sep 2026 | Read the raw licence file, not the badge alone |
| 32 | NVIDIA's **DCGM** and **DCGM Exporter** *images* are licensed under **Product-Specific Terms for NVIDIA AI Products**, while the **GPU Operator Helm chart** is Apache 2.0 and the GPU Operator *image* is under Product-Specific Terms | Verified | GPU Operator 26.7 documentation, "Licenses and Contributing" table | 26.7; 19 Sep 2026 | The source-vs-image licence split is the fact most often mis-stated |
| 33 | A subset of DCGM functionality is **not** in the open-source project and ships as `datacenter-gpu-manager-4-proprietary*`, installed as a recommendation | Verified | DCGM *Install DCGM* (package relationships) | 4.7.0; 19 Sep 2026 | Relevant to source builds and air-gapped mirrors |
| 34 | Release cadence / current release | **Flagged** | DCGM *Release Notes* (opens at **4.7.0**) + GitHub tags pattern `v<version>` | 19 Sep 2026 | The doc set's top release-note entry is 4.7.0, but no release **date** was read and no assertion is made about cadence or the newest tag — see §16 |
| 35 | DCGM does not replace NVML, `nvidia-smi`, or Fabric Manager; "drivers and platform services still control the hardware" | Verified | DCGM *Learn* landing page ("How DCGM Fits") | 4.7.0; 19 Sep 2026 | Underpins the layering conclusion in §2 |
| 36 | Diagnostics "do not repair faults, replace NVIDIA offline field diagnostics, or determine RMA eligibility" | Verified | DCGM *Diagnostics* learn page (Beyond Scope) | 4.7.0; 19 Sep 2026 | Quoted directly in §6.5 and §13.7 |
| 37 | The three failure-class actions (application → sanitizers/debuggers; hardware → contact the hardware vendor; driver → file a bug with `nvidia-bug-report.sh`) | Verified | XID Errors *Working with Xid Errors* | 19 Sep 2026 | Makes the RMA boundary explicit |
| 38 | "vGPU" appears in DCGM's metric catalogue (e.g. `DCGM_FI_DEV_VGPU_LICENSE_STATUS`, `DCGM_FI_DEV_VGPU_MEMORY_USAGE`, entity type `vgpu`) as *host-side* inventory and telemetry | Verified | DCGM *Field Identifiers*, *Modules* and `dcgmi` entity-selector table | 4.7.0; 19 Sep 2026 | Per pet hazard (b), no claim is made about vGPU licensing terms or about DCGM's role in vGPU provisioning |
| 39 | DCGM observes NVIDIA accelerators only; AMD/Intel or other accelerators are out of scope | **Flagged** | Inference from DCGM's documented scope (NVIDIA datacenter GPUs, NVSwitch, ConnectX, NVIDIA CPUs) | 4.7.0; 19 Sep 2026 | The vendor does not discuss alternatives; the statement is this guide's inference, labelled as such in §2.3 and §12.5 |
| 40 | Sustained-condition logic ("for more than N minutes") is not expressible in a field-based policy | **Flagged** | Inference from the documented condition model (field, operator, threshold, `changed`) | 4.7.0; 19 Sep 2026 | Labelled as inferred, not vendor-stated, in §8.1 |
| 41 | Page-retirement fields `DCGM_FI_DEV_PAGE_RETIRED_SBE_TOTAL`/`_DBE_TOTAL`/`_PENDING` = **390/391/392**; `DCGM_FI_DEV_PCIE_REPLAY_TOTAL` = **202**; `DCGM_FI_DEV_NVLINK_THROUGHPUT_TOTAL` = **449**; `DCGM_FI_DEV_NVLINK_CRC_ERROR_TOTAL` = **497** | Verified | DCGM *Field Identifiers* reference (field catalogue) | 4.7.0; 19 Sep 2026 | Read directly from the catalogue during verification; these four rows were the ones left open in the first pass |
| 42 | `DCGM_FI_DEV_NVLINK_RECOVERY_TOTAL` and `DCGM_FI_DEV_NVLINK_REPLAY_TOTAL` IDs | **Rejected** | Not read from the catalogue | 4.7.0; 19 Sep 2026 | No ID is stated anywhere in this guide for these two; the exporter CSV names them without IDs |

---
## 16. What Could Not Be Verified

Everything in this section is a research task, not a fact. Nothing here is asserted elsewhere in the guide as verified.

**Field identifiers and metadata not read.** The page-retirement fields were resolved during verification and are quoted in §4.2 under their **canonical** names — `DCGM_FI_DEV_PAGE_RETIRED_SBE_TOTAL` (390), `DCGM_FI_DEV_PAGE_RETIRED_DBE_TOTAL` (391) and `DCGM_FI_DEV_PAGE_RETIRED_PENDING` (392) — read from the *Field Identifiers* catalogue; the exporter's `default-counters.csv` ships the same three rows renamed `DCGM_FI_DEV_RETIRED_SBE` / `…_DBE` / `…_PENDING` and **commented out**, which is why the initial pass could not derive them. What remains unread: the IDs for `DCGM_FI_DEV_NVLINK_RECOVERY_TOTAL` and `DCGM_FI_DEV_NVLINK_REPLAY_TOTAL` (named in the exporter CSV without IDs, and not resolved during verification), any field referenced in this guide only through the exporter CSV or the exporter metrics page without an accompanying ID, and fields mentioned in passing (for example the link-level field 1200 used in the vendor's `dcgmi dmon` example, whose symbolic name was not read). **Rule: an ID is quoted in this guide only where the vendor stated it.**

**Release dates and cadence.** The DCGM documentation set's release-notes page opens at **DCGM 4.7.0**, and the repository README documents that releases are tagged as `v<version>` once finalised. No release **date** for 4.7.0 (or any other version) was read, and no statement is made about the release cadence, the newest git tag, or end-of-life dates. The DCGM repository's `docs/support_EOL.md` (referenced from the README) was not read. Anything version-specific in this guide is therefore anchored to "the documentation set whose release notes open at 4.7.0, checked 19 September 2026", not to a date.

**XID coverage.** The Xid catalog page is large, and this guide quotes a **selection** of rows with their mnemonics, descriptions, `Applies to` columns, resolution buckets and trigger text. Rows not quoted — including many `Unused` rows and any row whose text was not captured — are not summarised here. The catalogue itself (with its downloadable spreadsheet) is the authoritative artefact and should be read directly and pinned. No XID code is interpreted in this guide without quoting the vendor's own description or trigger text for it.

**The diag-skus configuration schema.** The diagnostic configuration file is documented as `dcgm-diag-v1` YAML with SKU overlays, GPU matching, recursive merging and source precedence; this guide reproduces only the vendor's example overlay and the precedence order. The full schema, every configurable field, and the per-plugin parameter constraints were not read.

**Exporter command-reference flags beyond the documented set.** The seven environment-variable/flag pairs, the device-string syntax and the defaults in §9.1 were read from the exporter documentation. The full command reference — including exact startup syntax for TLS/basic-auth, logging, debug dumps, pprof and any flags not surfaced in the pages read — was not read in full. Where the guide gives an option, it is one the vendor's own text named.

**GPU Operator metrics-ConfigMap specifics.** The Operator documentation states that custom fields are configured "through the Operator-managed metrics ConfigMap" and that the Operator-owned DaemonSet must not be edited directly. The ConfigMap's exact name, key structure and reconciliation behaviour across Operator versions were not read; teams should confirm them against the Operator version they deploy.

**Concurrent profiling metric-group limits.** The exporter metrics page refers to "hardware support, concurrent metric-group limits, and blank values" for profiling metrics, and the profiling page describes `dcgmi profile` as discovering "which profiling fields the selected GPU supports and how older hardware groups counters". The **specific** limits — how many profiling metric groups can be collected concurrently on a given GPU generation, and the interaction rules — were not read, so no numbers are given here. §4.5 therefore describes the pause/resume discipline without asserting a concurrency ceiling.

**Community artefacts.** Beyond the OpenObserve dashboard named in §9.5, no inventory of community dashboards, exporters, forks or integrations was attempted, and no community artefact is recommended. Vendor-versus-community provenance was established only for the artefacts the exporter's own README names.

**Regulatory and licensing interpretation.** §12 describes a design discipline, not legal advice. Nothing here interprets how a specific regulator's operational-resilience, outsourcing or vendor-management rules apply to GPU telemetry, and the licence statements in §15 are statements of what the vendor's own files say — not of what those licences permit in any particular deployment. Lineage and consumer-duty-style rules are deliberately not named, because the repository's compliance content owns them and this guide has no primary source for them.

**Tool limitations during this research.** All research was performed with `web_search` and `web_extract` against the primary URLs named in §15 and the reference list in §17. No search returned empty and no extraction failed outright; two URLs returned NVIDIA's "Page Not Found" notice — `…/learn/modules/health.html` (superseded by `…/learn/modules/health-monitoring.html`) and `…/cloud-native/gpu-operator/latest/dcgm-exporter.html` (the exporter is documented under the DCGM documentation set and under the GPU Telemetry doc set instead) — and the corrected pages were used. One documentation set, **NVIDIA GPU Telemetry 1.0.0**, describes the exporter in terms of earlier versions (its pages reference DCGM 3.3 material), so where it and the current DCGM documentation set differ, the DCGM set was treated as authoritative and the divergence is flagged in §15 row 26.

---
## 17. Glossary, Cross-References and Closing Summary

### 17.1 Glossary

| Term | Definition |
|---|---|
| **DCGM** | NVIDIA Data Center GPU Manager — a host-level framework for inventory, telemetry, health, policy, configuration, diagnostics, profiling and workload accounting on NVIDIA data-center hardware. ✅ Vendor definition |
| **Host engine** | The persistent DCGM process (`nv-hostengine`) that discovers entities, collects fields, caches samples and owns module state. Shared management plane for one host. |
| **`dcgmi`** | The DCGM command-line client. Connects to a host engine over localhost, TCP, a Unix socket or VSOCK. |
| **DCGM Exporter** | A separate Go program (Apache-2.0 source) that converts selected DCGM telemetry fields to Prometheus format at `/metrics`, port 9400 by default. |
| **Entity** | A managed object: GPU, GPU instance, compute instance, vGPU, NVSwitch, link, CPU, CPU core, ConnectX. Families are distinct types. |
| **Field** | One item of telemetry or state with a numeric ID and metadata (entity level, scope, value type, semantics). `DCGM_FI_*` is the C header symbol. |
| **Field group** | A named, reusable list of field IDs in the host engine — selects *columns*. Does not start collection. |
| **Entity group** | A named, reusable set of entities — selects *rows*. Does not imply any field is watched. |
| **Watch** | The operation combining entities, fields, an update interval and a retention limit — the operation that costs resources. Samples are shared host-engine state. |
| **Module** | DCGM's internal feature boundary. Core Services is module ID 0 (always loaded); ten further modules are loadable and denylistable. |
| **Health check** | Passive assessment of retained telemetry against enabled health rules, producing `Healthy`/`Warning`/`Failure` and `DCGM_FR_*` incidents. Applies no workload. |
| **Diagnostic level** | A numbered active-test suite: `1` quick (seconds), `2` medium (~2 min), `3` long (~15 min), `4` extended. Applies a real workload. |
| **XID** | An error report from the NVIDIA driver to the OS kernel or event log. Vendor-defined, stable across driver versions, and explicitly "a starting point for further investigation". |
| **SXid** | The NVSwitch equivalent of an XID, documented in the Fabric Manager User Guide. Applies to Hopper and earlier GPUs. ✅ |
| **NVML** | NVIDIA Management Library — the low-level per-device management and monitoring API that ships with the driver. |
| **`nvidia-smi`** | The driver's command-line tool for point-in-time, local inspection and administration. |
| **GPU Operator** | The Kubernetes operator that automates the NVIDIA node software stack, including DCGM-based monitoring and DCGM Exporter. |
| **MIG** | Multi-Instance GPU — hardware partitioning of one GPU into GPU instances and compute instances. Configuration is a driver concern; DCGM observes the instances as entities. |
| **ServiceMonitor** | The Prometheus Operator custom resource the exporter's Helm chart creates by default to wire the scrape. Assumes unauthenticated HTTP. |

### 17.2 Cross-references

**Vendor primary sources cited throughout:**

- DCGM documentation set — `docs.nvidia.com/datacenter/dcgm/latest/` (index, *Learn*, *Install DCGM*, *Install DCGM Exporter*, *Command Line*, *Field Identifiers*, *Diagnostics*, *DCGM Exporter Metrics*, *Configure Prometheus for DCGM Exporter*, release notes opening at 4.7.0)
- XID Errors — `docs.nvidia.com/deploy/xid-errors/` (*Introduction*, *Working with Xid Errors*, *Analyzing Xid Errors with the Xid Catalog*, and the downloadable `Xid-Catalog.xlsx`)
- NVML API reference — `docs.nvidia.com/deploy/nvml-api/`
- `nvidia-smi` documentation — `docs.nvidia.com/deploy/nvidia-smi/`
- GPU Operator documentation — `docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/` (26.7, patch v26.7.0)
- NVIDIA GPU Telemetry documentation set — `docs.nvidia.com/datacenter/cloud-native/gpu-telemetry/latest/dcgm-exporter.html` (1.0.0; describes the exporter in terms of earlier versions — see §16)
- `github.com/NVIDIA/DCGM` (README, `LICENSE`, release-tag convention) and `github.com/NVIDIA/dcgm-exporter` (README, `LICENSE`, `etc/default-counters.csv`, `grafana/dcgm-exporter-dashboard.json`)
- NVIDIA DCGM Exporter dashboard for Grafana — `grafana.com/grafana/dashboards/12239/`

**Companion guides in this repository:**

- GPU *sharing and virtualization* — the layer that produces the MIG instances DCGM observes: [hami_gpu_sharing_guide.md](hami_gpu_sharing_guide.md)
- GPU *performance engineering* — the application-side complement to DCGM's profiling fields: [gpu_optimization_guide.md](gpu_optimization_guide.md)
- Platform stacks that consume DCGM as a component: [openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md), [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md), [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md), [ai_llm/scalable_ai_deployment.md](ai_llm/scalable_ai_deployment.md), [ai_llm/enterprise_ai_platforms_guide.md](ai_llm/enterprise_ai_platforms_guide.md), [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md)
- Deliberate fault injection against the alert and runbook paths designed here: [chaos_engineering_guide.md](chaos_engineering_guide.md)
- The sizing arithmetic that consumes DCGM's measured utilisation: [capacity_sizing_guide.md](capacity_sizing_guide.md)
- The IT-operations discipline and framework that automated-response controls sit inside: [operational_support_frameworks_guide.md](operational_support_frameworks_guide.md), [../../banking/operational_resilience_framework_guide.md](../../banking/operational_resilience_framework_guide.md)
- GPU fundamentals and career-path context for the platform team: [gain_nvidia_experience.md](gain_nvidia_experience.md)

### 17.3 Keeping this guide current

Four clocks move under this material, and each one invalidates a different set of facts. **The DCGM release train** (the documentation set read here opens at 4.7.0; 4.7.0 is where base-unit field IDs 1600–1650, the canonical aliases, persistent field-based policies and the multi-node `nvloom`/`mnnvbandwidth` tests arrived) — re-verify field IDs, aliases, units and policy behaviour at each upgrade, because both `dcgmi` JSON output and the field catalogue are release-specific. **The driver and CUDA train** — several XID catalogue rows are explicitly linked to driver and CUDA versions ("CUDA 12.7; GPU driver R565"), the health model gained IMEX watches, and diagnostics gained MIG support on CUDA 12.8+; re-download and re-pin the XID catalogue whenever the driver major version changes. **The exporter/Kubernetes train** — the DCGM/exporter version pair is a supported combination, not two independent components, and the GPU Operator's metrics ConfigMap and chart defaults move with each Operator release. And **the hardware train** — profiling fields, health rules, NVLink counter families and even which XID codes apply are hardware-generation-scoped, so a fleet that adds a new GPU generation should expect to re-verify rather than assume. Everything flagged ⚠️ or ❓ in this guide is a re-verification candidate by design; everything marked ✅ is anchored to a page that was read on 19 September 2026 and should be re-read, not remembered.

### 17.4 The bottom line

DCGM is not a dashboard product and not a replacement for `nvidia-smi`; it is the stateful layer that makes a GPU fleet measurable. It holds what happened (watches with cadence and retention), it judges what that means (health rules with severities and incident codes), it proves what a device can do (numbered diagnostic suites with pass/fail and an isolation status), and it speaks the vendor's own failure vocabulary (XID codes with resolution buckets and a recovery action). Its limits are as important as its capabilities: `Healthy` is not proof, a diagnostic pass is not an RMA clearance, a utilisation percentage does not measure work, an XID code is a starting point rather than a diagnosis, and none of it remediates anything by itself. The platform that gets value from DCGM is the one that enables the health metrics the defaults omit, persists them, alerts on health and throttle signals rather than on utilisation, triages XIDs by the vendor's resolution bucket, keeps the DCGM/exporter pair versioned as one artefact, scopes tenant telemetry by data path rather than by dashboard filter, and writes the diagnostic policy before it needs it. Do that, and the fleet stops being a collection of machines that occasionally fail and becomes a system whose behaviour you can see, prove and explain — the signal before the failure.