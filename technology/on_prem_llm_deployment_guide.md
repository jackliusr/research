# On-Prem LLM Deployment — The End-to-End Playbook for Enterprise Data Centers

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides · **Topic:** LLM Infrastructure / Deployment
> **Focus:** Banking & regulated industries (Singapore, EU, global)
> **Companion Guides:** [Enterprise AI Gateway](enterprise_ai_gateway_guide.md) · [Ollama vs Xinference vs LocalAI](ai_llm/ollama_xinference_localai_guide.md) · [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) · [OpenShift AI Alternatives](openshift_ai_alternatives_guide.md) · [LLM Development Risks & Security](llm_development_risks_security_guide.md) · [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md) · [Knowledge Distillation](knowledge_distillation_guide.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Why On-Prem: The Case for Self-Hosted LLMs](#1-why-on-prem-the-case-for-self-hosted-llms)
2. [The Decision Framework: On-Prem vs Cloud API vs Hybrid](#2-the-decision-framework-on-prem-vs-cloud-api-vs-hybrid)
3. [Hardware and Infrastructure: The GPU Landscape 2025–2026](#3-hardware-and-infrastructure-the-gpu-landscape-20252026)
4. [GPU Selection for LLM Inference: Memory Is King](#4-gpu-selection-for-llm-inference-memory-is-king)
5. [Memory Bandwidth: The Real Inference Bottleneck](#5-memory-bandwidth-the-real-inference-bottleneck)
6. [CPU-Only Inference and Edge Options](#6-cpu-only-inference-and-edge-options)
7. [Interconnect, Server Form Factors, Storage, Network, Power](#7-interconnect-server-form-factors-storage-network-power)
8. [Model Selection for On-Prem](#8-model-selection-for-on-prem)
9. [MoE vs Dense Architectures](#9-moe-vs-dense-architectures)
10. [Model Licensing for Banking](#10-model-licensing-for-banking)
11. [Quantization for On-Prem](#11-quantization-for-on-prem)
12. [Fine-Tuning for On-Prem](#12-fine-tuning-for-on-prem)
13. [The Serving Stack Architecture](#13-the-serving-stack-architecture)
14. [Inference Serving Patterns](#14-inference-serving-patterns)
15. [The Gateway Layer](#15-the-gateway-layer)
16. [Model Registry and Versioning](#16-model-registry-and-versioning)
17. [Environment Design: Dev / Test / Prod / DR](#17-environment-design-dev--test--prod--dr)
18. [Air-Gapped Deployment](#18-air-gapped-deployment)
19. [Security and Networking](#19-security-and-networking)
20. [Operations: Monitoring, Alerting, Logging, Capacity](#20-operations-monitoring-alerting-logging-capacity)
21. [Testing and Validation](#21-testing-and-validation)
22. [Cost Analysis: On-Prem TCO vs API](#22-cost-analysis-on-prem-tco-vs-api)
23. [Governance and Compliance](#23-governance-and-compliance)
24. [The Deployment Playbook: Step by Step](#24-the-deployment-playbook-step-by-step)
25. [Common Pitfalls](#25-common-pitfalls)
26. [The Future: On-Prem LLM 2026 and Beyond](#26-the-future-on-prem-llm-2026-and-beyond)
27. [Conclusion](#27-conclusion)

---

## 1. Why On-Prem: The Case for Self-Hosted LLMs

Deploying LLMs inside your own data center — rather than calling a vendor's cloud API — is not a technology preference. For a bank it is increasingly a **compliance, risk, and economic decision** with hard regulatory consequences. This section lays out the seven pillars of the on-prem argument, each mapped to a concrete banking driver.

### 1.1 Data Sovereignty

**The core proposition: when prompts, contexts, and responses never leave your premises, the data-sovereignty problem disappears by construction.**

- **Customer data / PII** — client names, account numbers, transaction narratives, KYC documents, and trade details fed into an LLM via a third-party API become data shared with an external processor, often in a foreign jurisdiction.
- **Regulatory data** — regulatory reporting inputs, audit findings, risk models' inner workings: any leakage is a reportable event.
- **No cross-border transfer** — with on-prem inference there is no "transfer" at all, which sidesteps GDPR Chapter V transfer mechanisms, Schrems II considerations, and PDPA transfer obligations entirely.
- **Sub-processor chains** — a cloud API call may transit model provider → hosting cloud → observability/logging vendor; each hop is a sub-processor that must be vetted, contracted, and audited. On-prem collapses the chain to systems you already control.

> **Banking reality:** For Cymbal Bank, client-relationship data (names, mandates, facility terms) is the crown jewel. The question "does this prompt contain client data?" is unanswerable at scale for free-text workloads — so the safe default is that it *might*, and the architecture must therefore keep inference local.

### 1.2 Compliance

On-prem is not automatically compliant — you still have to operate it correctly — but it makes compliance *achievable* where cloud APIs make it *conditional*:

| Regulation | Why on-prem helps |
|---|---|
| **MAS TRM** (Technology Risk Management Notice) | Availability, security, and change management of critical systems stay inside the bank's own operational risk framework; no dependency on a third party's operational resilience. |
| **MAS FEAT** (Fairness, Ethics, Accountability, Transparency) | Accountability for model behavior is unambiguous when you own the full stack — model, data, serving, logging. |
| **PDPA (Singapore)** | Personal data stays within the organisation; consent and purpose limitations are enforceable on infrastructure you control. |
| **GDPR (EU)** | No data transfer to a processor; Article 28 DPA chains and Article 32 security obligations are materially simpler. |
| **BCBS 239** (Risk Data Aggregation) | Data lineage is fully traceable when the data never leaves your platform. |
| **Internal AI policies** | Bank AI acceptable-use policies typically restrict which external tools employees may feed data into; a self-hosted model is an *approved internal system*, not an unvetted external service. |
| **EU AI Act** | For high-risk systems, the deployer retains full control over the AI system lifecycle, monitoring, and logging — far easier on-prem. |

### 1.3 Security

- **No data sent to third-party APIs** — the attack surface shrinks from "internet + vendor + vendor's sub-processors" to your own network boundary.
- **Supply chain control** — you choose the model weights (open weights), the serving binary, the container images, and the dependency versions; you can scan, sign, and pin all of them (see [§18 Air-Gapped Deployment](#18-air-gapped-deployment) and [§19 Security](#19-security-and-networking)).
- **Inference is a private compute** — no prompt injection exfil channel via the model provider's own logging/analytics pipeline.
- **Audit and forensic readiness** — every request can be logged, redacted, and retained under your own retention policy, which is frequently impossible with vendor APIs (see [LLM Development Risks & Security](llm_development_risks_security_guide.md)).

### 1.4 Latency

- Cloud API latency = network RTT to the provider + provider's queueing under shared load + provider's prefill scheduling. The RTT alone from Singapore to a US/EU cloud region is 70–200 ms; under congestion it jitters far beyond that.
- On-prem latency = internal network (sub-millisecond) + your own GPU queue. Deterministic, measurable, and under your control.
- For **streaming chat** and **interactive agents**, the difference is time-to-first-token (TTFT) stability; for **trading-floor copilots** and **real-time decision support**, jitter matters more than the mean. See [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) for the full latency engineering treatment.

### 1.5 Cost at Scale

- Per-token API pricing does not amortise: every additional token costs the same, forever, and volume growth is a linear OpEx line.
- On-prem is a **capex-heavy, marginal-cost-near-zero** model: once the GPUs are bought, incremental tokens cost electricity.
- At sustained high volume (millions of tokens/day) on-prem per-token cost is typically **5–20× cheaper** than API per-token pricing for the same model class — see [§22 Cost Analysis](#22-cost-analysis-on-prem-tco-vs-api) for the full break-even model.

### 1.6 Customization

- **Fine-tuned models** — domain adapters (credit analysis, trade documentation, regulatory Q&A) can only be deployed if you own the weights and the serving stack. Most API providers do not serve *your* fine-tunes with full control over inference parameters.
- **Full control** — sampling parameters, context windows, stop sequences, structured output, tool calling, and prompt formatting are all yours to tune (see [Constrained Decoding Frameworks](constrained_decoding_frameworks_guide.md)).
- **No provider policy interference** — no content-policy middle layer rewriting or refusing bank-internal requests (subject to your own governance, of course).

### 1.7 Resilience

- **No vendor API outages** — an API provider's incident (and they happen: rate-limit storms, model retirements, pricing changes, deprecations) takes down your production workload with zero recourse.
- **No version surprise** — the API provider can silently upgrade the model behind an alias; on-prem the model version is pinned, tested, and immutable.
- **Operational control** — you decide when to patch, when to upgrade, when to fail over to a standby pool. This aligns with MAS TRM expectations on availability of critical systems.

> **Caveat (intellectual honesty):** on-prem is not automatically better. You inherit the full operational burden — GPU hardware failures, driver/kernel compatibility, capacity planning, and a platform team. The decision framework in the next section exists precisely to prevent on-prem-for-its-own-sake.

---

## 2. The Decision Framework: On-Prem vs Cloud API vs Hybrid

### 2.1 The Decision Factors

| Factor | On-prem | Cloud API | Notes |
|---|---|---|---|
| **Volume** | Wins at sustained high volume | Wins at low/spiky volume | Break-even typically 10–50M tokens/day sustained — see §22 |
| **Data sensitivity** | Required for PII/regulatory data | Only for non-sensitive data | This single factor often decides the architecture regardless of cost |
| **GPU investment** | Needs capex budget + lead time (GPU lead times can be months) | Zero capex | Finance appetite for a multi-million-dollar GPU pool is a real constraint |
| **Team skills** | Needs GPU/MLOps platform engineers | Needs only application developers | Skills gap is the #1 underestimated on-prem cost |
| **Model freshness** | You control upgrades (weeks-months lag) | Instant access to newest models | If you must have the frontier model the day it ships, API wins |
| **Vendor risk tolerance** | Full independence | Vendor lock-in, pricing risk, policy risk | Regulatory pressure increasingly pushes banks toward independence |
| **Time to market** | PoC → prod in months | PoC in days | Hybrid lets you start on API and migrate workloads to on-prem as they stabilise |
| **Compliance** | Achievable by construction | Conditional on vendor commitments | DPA, data residency, audit rights |
| **Latency SLO** | Deterministic, internal | Subject to network + provider load | For sub-500ms p95 TTFT on streaming, on-prem is more predictable |

### 2.2 The Three Options

1. **On-prem only** — every workload self-hosted. Justified when *any* workload may touch sensitive data, when regulatory pressure is maximal (EU AI Act high-risk, sovereign-AI mandates), or when total volume is high enough that the whole portfolio amortises the GPU pool.
2. **Cloud API only** — no self-hosted infrastructure. Justified for prototyping, low-volume non-sensitive workloads, and teams without platform skills. Not defensible for a bank's client-data workloads.
3. **Hybrid (the banking norm)** — a **policy-routed** split:
   - **On-prem:** workloads touching client data, PII, trade data, regulatory reporting, internal strategy — anything where data classification is "confidential" or above.
   - **Cloud API:** public-data workloads, code assistance on non-proprietary code, generic summarisation of sanitised content, benchmark/exploration, burst overflow when on-prem pools are saturated.
   - The **enterprise AI gateway** becomes the enforcement point: it routes each request by data classification + model policy (see [Enterprise AI Gateway](enterprise_ai_gateway_guide.md) and §15).

### 2.3 Typical Banking Pattern

```
                 ┌────────────────────────────────────────────┐
                 │           Enterprise AI Gateway           │
                 │   authN/Z · policy routing · quotas · log  │
                 └───────┬──────────────────────┬────────────┘
                         │ data class =         │ data class =
                         │ confidential          │ public / sanitised
                 ┌───────▼────────┐      ┌───────▼────────┐
                 │  ON-PREM pool  │      │   CLOUD API    │
                 │ vLLM/SGLang    │      │ vendor models  │
                 │ Llama/Granite/ │      │ (frontier,     │
                 │ Qwen/DeepSeek  │      │  non-sensitive)│
                 └────────────────┘      └────────────────┘
```

**Key design principle:** the *default route* is on-prem; cloud API is an explicit, policy-gated exception — not the other way around. This inverts the risk posture of most early AI adoptions.

### 2.4 A Pragmatic Scoring Sheet

Score each candidate workload 1–5 on: sensitivity (5 = client data), volume (5 = >50M tok/day), latency SLO strictness, model-freshness need, and skills availability. Then:

- **Sensitivity ≥ 4** → on-prem, no further discussion.
- **Volume ≥ 4 and sensitivity ≥ 2** → strong on-prem case, run break-even model.
- **Everything else** → API or hybrid-burst, revisit quarterly.

---

## 3. Hardware and Infrastructure: The GPU Landscape 2025–2026

### 3.1 The GPU Landscape at a Glance

| GPU | Memory | Bandwidth | Generation / Notes | On-prem fit |
|---|---|---|---|---|
| **NVIDIA A100 80GB** | 80GB HBM2e | ~2.0 TB/s | Ampere (2020), previous gen, still very common in enterprise fleets | 70B INT4, 8B FP16 many-replica; cheap second-hand |
| **NVIDIA H100 SXM** | 80GB HBM3 | 3.35 TB/s | Hopper — the workhorse of 2024–2026 fleets; FP8 native | 70B FP16 (2× TP), 405B INT8 (4–8× TP) |
| **NVIDIA H200** | 141GB HBM3e | 4.8 TB/s | Hopper refresh — big-memory variant | Single-GPU 70B FP16; 405B INT4 in fewer nodes |
| **NVIDIA B200 / GB200** | 192GB HBM3e (per GPU) | ~8 TB/s | Blackwell — 2× Hopper perf; GB200 NVL72 is the rack-scale form | Frontier on-prem; heavy power/cooling (liquid) |
| **NVIDIA L40S** | 48GB GDDR6 | ~864 GB/s | Ada inference-optimised; no NVLink | 70B INT4 single GPU, 8B serving farms |
| **NVIDIA RTX 6000 Ada** | 48GB GDDR6 | ~960 GB/s | Workstation class, no NVLink, air-cooled | Dev/CI/test boxes, small pilots |
| **AMD MI300X** | 192GB HBM3 | 5.3 TB/s | Strong price/performance; ROCm software stack (mature for vLLM/SGLang) | 70B FP16 single GPU; cost-effective dense pools |
| **Intel Gaudi 3** | 128GB HBM2e | ~3.7 TB/s | AI accelerator, open SDK, strong TCO for inference | Emerging alternative; ecosystem smaller |
| **Groq LPU** | SRAM-based (~230MB/chip) | Ultra-low latency | Inference-speed champion for small/medium models; memory-limited for large models | Specialised low-latency serving; not general-purpose |
| **AWS Trainium2** | 32GB/chip (pools via NeuronLink) | — | Cloud-only (Trn2 instances) | Not applicable to on-prem DCs |

### 3.2 NVIDIA vs AMD vs Alternatives — Selection Notes

- **NVIDIA** is the safe default: best software maturity (CUDA, TensorRT-LLM, vLLM first-class), best resale value, but highest cost and longest lead times.
- **AMD MI300X** is the value play: 192GB at a fraction of B200 cost, and vLLM/SGLang ROCm support is now production-grade. The risk is the long tail of ecosystem issues (custom kernels, quantization kernels, observability tools).
- **Intel Gaudi 3** and **Groq** are credible for *inference-specific* fleets where you control the model zoo; they are not drop-in replacements for a general-purpose ML platform.
- **Practical bank stance:** standardise on NVIDIA for the main pool (operational familiarity, MIG, NVLink), keep an AMD evaluation node for TCO benchmarking, and treat alternative silicon as a procurement lever.

---

## 4. GPU Selection for LLM Inference: Memory Is King

### 4.1 The Memory Rule

For autoregressive LLM inference, **VRAM capacity decides which models you can serve; memory bandwidth decides how fast; compute is rarely the binding constraint.** Everything starts with fitting the model.

**The sizing formula:**

```
total_VRAM ≈ model_params × bytes_per_param × 1.2 + KV_cache
```

Where:

- `model_params` — parameter count (e.g. 70B = 70 × 10⁹)
- `bytes_per_param` — FP16/BF16 = 2, INT8 = 1, INT4 = 0.5, FP8 = 1
- `1.2` — overhead factor: activations, CUDA context, fragmentation, framework buffers
- `KV_cache` — key-value cache for the context window: roughly `2 (K+V) × num_layers × hidden_size × num_tokens × bytes_per_elem`. Rule of thumb: **~1–2 GB per 1K tokens of context for a 70B-class model**, far less for 7–8B models (tens of MB per 1K tokens).

### 4.2 Worked Examples

| Model | Precision | Weights | + Overhead | KV cache (typical) | GPU fit |
|---|---|---|---|---|---|
| **70B** | FP16 | 140 GB | ~168 GB | 2–8 GB @ 4–16K ctx | **2× H100 (TP2)** or 1× H200 / 1× MI300X |
| **70B** | INT4 | 35 GB | ~42 GB | 2–8 GB | **1× L40S / RTX 6000 Ada (48GB)** |
| **8B** | INT4 | 4 GB | ~5 GB | <1 GB | 1× RTX 4090 / consumer card |
| **8B** | FP16 | 16 GB | ~19 GB | ~1 GB | 1× A100 40GB / L40S |
| **405B** | INT8 | 405 GB | ~486 GB | 10–30 GB | **4–8× H100 (TP4/8)** with NVLink |
| **671B MoE (DeepSeek V3)** | FP8 | ~134 GB (active ~37B) | ~160 GB | 5–20 GB | 1–2× H200 / MI300X for 128K ctx |
| **32B** | INT4 | 16 GB | ~19 GB | 1–3 GB | 1× L40S / RTX 6000 Ada |

### 4.3 GPU Selection Decision Tree

1. **Pick the largest model class you must serve** (from §8 model selection).
2. **Choose precision** (FP16 baseline; INT4/FP8 if you need to fit or cut cost — §11).
3. **Compute weights × bytes × 1.2** → minimum VRAM.
4. **Add KV cache** for the *maximum* context length you will actually serve (not the model's max).
5. **If one GPU fits → prefer 1 GPU** (no TP overhead, simpler ops). If not → split across the *smallest* number of GPUs with NVLink (tensor parallelism; §7.1).
6. **Add headroom**: never run a model that uses >90% of VRAM at peak — batching needs KV-cache headroom (see pitfalls P1/P2 in §25).

---

## 5. Memory Bandwidth: The Real Inference Bottleneck

### 5.1 Why Bandwidth Dominates

Decoding is memory-bound: for every output token, the model must stream *all* weights from HBM to compute once. Therefore:

```
token_generation_speed ≈ memory_bandwidth / model_weights_bytes
```

| GPU | Bandwidth | 70B FP16 (140GB) | 70B INT4 (35GB) | 8B FP16 (16GB) |
|---|---|---|---|---|
| H100 | 3.35 TB/s | ~24 tok/s | ~95 tok/s | ~200 tok/s |
| H200 | 4.8 TB/s | ~34 tok/s | ~135 tok/s | ~300 tok/s |
| MI300X | 5.3 TB/s | ~38 tok/s | ~150 tok/s | ~330 tok/s |
| L40S | 0.86 TB/s | ~6 tok/s (fits only INT4) | ~24 tok/s | ~54 tok/s |
| A100 80GB | 2.0 TB/s | ~14 tok/s | ~57 tok/s | ~125 tok/s |

These are *theoretical single-sequence ceilings*; real serving gets close with continuous batching because decode tokens from many requests are interleaved, keeping the memory pipe busy. **The practical takeaways:**

- **Quantization is a speed multiplier**, not just a memory saver: INT4 roughly 4× decode speed vs FP16 on the same GPU (because bandwidth, not FLOPs, is the constraint).
- **Context length hurts decode indirectly** — longer prompts mean more prefill work and bigger KV cache, but per-token decode speed is constant per model+precision.
- **Do not buy compute you cannot feed**: a 700W compute-dense GPU is wasted if the model is small and latency-bound; conversely a 70B FP16 on a workstation GPU (L40S) will feel slow (~6 tok/s) — that is a bandwidth problem, not a "GPU too weak" problem.

### 5.2 Prefill vs Decode

- **Prefill** (processing the prompt) is compute-bound and parallel — it benefits from FLOPs and batch size.
- **Decode** (generating tokens) is bandwidth-bound and sequential — it benefits from bandwidth and batch interleaving.
- This asymmetry drives serving architecture: chunked prefill, prefix caching, and continuous batching (see [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) and §14).

---

## 6. CPU-Only Inference and Edge Options

### 6.1 When CPU-Only Makes Sense

- **Small models (1–8B)** with low QPS requirements: internal tools, dev/test environments, document classification, batch summarisation of short texts.
- **llama.cpp / GGUF** is the standard CPU stack — see [Ollama vs Xinference vs LocalAI](ai_llm/ollama_xinference_localai_guide.md) for the tool comparison and setup.
- **Apple Silicon** (M-series) is genuinely good at this: unified memory gives the GPU access to the full RAM (e.g. M2 Ultra: 192GB unified, ~800 GB/s), making 70B INT4 laptops workable for prototyping.
- **The hard truth about memory bandwidth:**

| Platform | Memory bandwidth | Relative to H100 |
|---|---|---|
| H100 GPU | 3.35 TB/s | 1× (reference) |
| M2 Ultra (unified) | ~800 GB/s | ~4× slower |
| Server DDR5 (8-channel) | ~400–500 GB/s | ~7–8× slower |
| Laptop DDR5 (2-channel) | ~50–100 GB/s | **~35–60× slower** |

An 8B FP16 model (~16GB) on a laptop streams at ~50 GB/s → **~3 tok/s**. Usable for experimentation, not for production. **CPU-only production inference is viable only for 1–4B models at modest concurrency**, or as a cold-standby tier.

### 6.2 CPU/GPU Hybrid Patterns

- **CPU fallback tier**: route low-priority, small-model workloads (e.g. embeddings, classification) to CPU nodes so GPU pools stay reserved for generation. This is a common capacity lever.
- **Edge inference**: for branch-office / low-latency local workloads, deploy 1–8B INT4 GGUF on edge servers with llama.cpp or ONNX Runtime — see the on-prem + edge hybrid in §26.

---

## 7. Interconnect, Server Form Factors, Storage, Network, Power

### 7.1 NVLink / NVSwitch vs PCIe

- **NVLink** (GPU-to-GPU): H100 ~900 GB/s per direction; B200 ~1.8 TB/s. Needed for tensor parallelism — the weight shards exchange activations every layer, so interconnect bandwidth directly caps TP scaling.
- **NVSwitch** (fully-connected switch fabric within a node, e.g. 8×GPU DGX): all-to-all bandwidth, low latency — the reason an 8-GPU box behaves like one big GPU.
- **PCIe Gen5** (~64 GB/s per direction, shared): adequate for *pipeline* parallelism and data-plane traffic, inadequate for tight TP on large models. **Rule: TP only across NVLink-connected GPUs; never TP across PCIe-only GPUs for >7B models** (it can be 2–5× slower than NVLink TP).

### 7.2 Server Form Factors

| Form factor | GPUs | Typical use | Notes |
|---|---|---|---|
| 8×GPU (DGX-like, e.g. DGX H100, GB200 NVL72 rack) | 8 × H100/B200 | Production dense pool, large models, TP8 | ~10 kW+/node; NVSwitch; liquid cooling for Blackwell |
| 4×GPU (e.g. HGX baseboard, Supermicro 4U) | 4 × H100/L40S | Mid-size pool, 70B FP16 TP4, multi-model nodes | ~4–7 kW |
| 2×GPU workstation (e.g. Precision, ThinkStation with RTX 6000 Ada) | 2 × 48GB Ada | Dev, CI, PoC, pilots | Air-cooled, 1–2 kW, office-safe |

**Procurement reality:** 8×GPU nodes are the cheapest *per GPU* (shared chassis, power, networking) but concentrate blast radius and need dedicated power/cooling; 4×GPU nodes give better failure isolation and fit standard racks; workstations accelerate dev velocity at the cost of higher per-GPU price.

### 7.3 Storage

- **NVMe (local or shared)** for model weights: a 140GB FP16 model loads in ~10s from a 15GB/s NVMe vs minutes from spinning disk; fast load = fast restart = faster failover and rolling updates. Weights can also be pinned in page cache / GPU memory across restarts.
- **Model registry storage**: object storage (MinIO/Ceph/S3-compatible) or NFS for versioned artifacts — see §16.
- **Checkpoint storage**: fine-tuning checkpoints (adapter weights are small — MBs; full fine-tune checkpoints can be hundreds of GB) with retention policy.
- **Anecdote from practice:** a 70B FP16 model on 2× H100 loads from local NVMe in ~30–60s including tokenizer/engine init; from a slow NFS mount it can take 5–10 minutes. That difference decides whether a rolling update is 1 minute or a maintenance window.

### 7.4 Network

- **Within node**: NVLink (above).
- **Between nodes**: 25/100/400 GbE. For *multi-node* tensor/pipeline parallelism (rare; only for 405B+ or MoE giants), RDMA/RoCEv2 or InfiniBand is required — TCP loses to RDMA at these message sizes.
- **Cluster fabric**: spine-leaf topology, 100GbE+ to each GPU node, jumbo frames, lossless QoS for RoCE.
- **Management plane**: separate out-of-band network (iLO/iDRAC/Redfish) for hardware management — keep it off the data fabric.
- **Practical stance:** most on-prem LLM deployments are *single-node* per model replica (TP within node via NVLink). Multi-node parallelism is an exception, not the default — design for it, but don't build for it initially.

### 7.5 Power and Cooling

- **Power envelope per server:**
  - 2×GPU workstation: 1–2 kW
  - 4×GPU node (H100): 4–7 kW
  - 8×GPU node (H100): 10–13 kW
  - GB200 NVL72 rack: 120+ kW (liquid-cooled)
- **Liquid cooling** becomes mandatory for Blackwell density; H100 8×GPU nodes can still run air with careful rack layout, but high ambient Singapore DCs (26–32°C) push you toward liquid-assisted or rear-door heat exchangers.
- **Capacity planning:** a 10-GPU-node pool at ~10kW average = ~100 kW IT load ≈ 120–150 kW facility (with cooling overhead) — a material chunk of a bank DC's capacity. **Do the power study before signing the PO**, not after.

---

## 8. Model Selection for On-Prem

### 8.1 Model Families Suitable for On-Prem

The on-prem model zoo is defined by **open weights** — models you can download, host, and fine-tune yourself. As of mid-2026:

| Family | Sizes | License | Strengths / On-prem fit |
|---|---|---|---|
| **Llama 3.x (Meta)** | 8B, 70B | Llama Community License (acceptable-use) | The default enterprise open-weight family; enormous ecosystem, tooling, quantization support. 70B is the on-prem quality ceiling reference. |
| **Qwen 2.5 (Alibaba)** | 0.5B–72B | Apache 2.0 | Strong multilingual incl. **Chinese** — relevant for Asia desks; excellent code and math; 72B rivals Llama-70B. |
| **DeepSeek V3 / R1** | 671B MoE; distilled 7B/32B/70B | MIT (model license permissive) | Frontier-class reasoning at open weights; **distilled 7B/32B are the practical on-prem targets** — 671B needs a serious cluster. |
| **Mistral** | 7B, 8x22B MoE | Apache 2.0 | Efficient, strong Europe/FR language coverage (relevant for Cymbal Bank); 8x22B MoE gives 70B-class quality at lower active-param cost. |
| **Gemma 3 (Google)** | 2B–27B | Gemma license (acceptable-use, no 100M+ user restriction) | Small efficient models, strong for edge/dev; 27B is a good mid-tier. |
| **Phi-4 (Microsoft)** | 14B | MIT | Small-but-strong reasoning for its size; cheap to serve; good for constrained budgets. |
| **Granite (IBM)** | 3B–34B | Apache 2.0 (truly open) | **Banking-friendly**: trained with enterprise/regulated-domain emphasis, permissive license, Red Hat ecosystem support (works with the OpenShift AI discussion in [OpenShift AI Alternatives](openshift_ai_alternatives_guide.md)). |
| **Command-R / Command-R+ (Cohere)** | 35B / 104B | CC-BY-NC (non-commercial!) | Excellent RAG and tool use, multilingual; **license restricts commercial use — evaluate carefully** (§10). |
| **OLMo (AI2)** | 1B–7B | Apache 2.0 (fully open, incl. data) | Full transparency (training data released) — useful for the most demanding model-governance reviews. |
| **Aya (Cohere)** | 8B–23B | Apache 2.0 | Multilingual coverage (100+ languages) — good for regional compliance/translation workloads. |

### 8.2 Selection Criteria for a Bank

1. **License** first (§10) — a technically great model with a non-commercial license is disqualified.
2. **Quality on your domain** — run your own evaluation set (§21); vendor benchmarks are marketing. See [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md).
3. **Serving economics** — a 70B FP16 needs 2× H100; the same budget serves 4× replicas of a fine-tuned 32B INT4 with better latency. Model size is a fleet decision.
4. **Ecosystem maturity** — vLLM/SGLang support, quantization availability (GPTQ/AWQ/FP8), fine-tuning recipes, community fixes. Llama and Qwen win here; smaller families can lag.
5. **Future-proofing** — the family's roadmap: will there be a 3.x/4.x that reuses your fine-tuning and evaluation investment?
6. **Multilingual needs** — if your book of business includes Chinese, Arabic, or French clients, weight Qwen/Aya/Mistral accordingly.

### 8.3 The "Model Zoo" Strategy

Do not standardise on one model. Banks operate a **zoo**: a small cheap model for classification/routing, a mid model for summarisation, a large model for complex reasoning, an embedding model for RAG, and a reranker. The gateway (§15) routes by task. This is both an economic and a resilience strategy — no single-model dependency.

---

## 9. MoE vs Dense Architectures

### 9.1 The Basics

- **Dense** (Llama 70B, Qwen 72B): every parameter participates in every token. Simple, predictable memory and latency; VRAM = full parameter count.
- **Mixture of Experts** (DeepSeek V3 671B, Mistral 8x22B, Qwen3-MoE): the model has many expert sub-networks; a **router** activates only a few per token. All parameters are *loaded* in memory, but only the active ones are *computed*.

### 9.2 Implications for On-Prem

| Dimension | Dense | MoE |
|---|---|---|
| **Memory** | All params in VRAM (the whole model) | **All params in VRAM** (the whole model!) — no saving vs dense at same total size |
| **Compute per token** | All params | Only active experts (e.g. 37B active of 671B total) |
| **Throughput** | Proportional to size | **Much higher per token** — 671B MoE can decode like a ~40B dense |
| **Serving complexity** | Simple | Complex: expert parallelism, load balancing, higher prefill cost, KV-cache handling |
| **Quality** | — | Can reach frontier quality with far less compute per token (DeepSeek-V3/R1) |
| **On-prem fit** | Predictable; standard | **Best cost/quality on-prem** if you can handle ops complexity |

**The on-prem sweet spot for MoE:** DeepSeek-R1-Distill (7B/32B dense) for reasoning, and — if you have H200/MI300X-class 192GB GPUs — a genuine MoE like DeepSeek-V3 or Qwen3-235B served with vLLM, which supports MoE expert parallelism. See [Knowledge Distillation](knowledge_distillation_guide.md) for why distilled reasoning models are the practical path.

---

## 10. Model Licensing for Banking

### 10.1 Open Weights ≠ Open Source

All "open" LLMs are **open weights**: the trained parameters are downloadable, but the license governs use. Few are OSI open source. The distinction matters enormously in a bank's legal review:

| Model | License | Key terms a bank must check |
|---|---|---|
| **Llama 3.x** | Llama Community License | Acceptable-use policy (no unlawful/harmful use), no restrictions on commercial use for <700M MAU; **attribution not required but redistribution terms apply** |
| **Qwen 2.5** | Apache 2.0 | Permissive: commercial use, modification, redistribution allowed; patent grant; no copyleft |
| **Mistral (7B/8x22B)** | Apache 2.0 | Same as Qwen — permissive |
| **Gemma 3** | Gemma Terms of Use | Commercial use allowed; prohibited uses list (similar to Llama); no 100M+ user restriction (differs from Gemma 1) |
| **Granite** | Apache 2.0 | **Truly open** — the cleanest choice for compliance-sensitive orgs |
| **Phi-4** | MIT | Very permissive, minimal conditions |
| **Command-R family** | CC-BY-NC | **Non-commercial** — disqualifies most banking use unless you license from Cohere commercially |
| **OLMo** | Apache 2.0 | Permissive + training data released |
| **DeepSeek** | MIT | Permissive |

### 10.2 License Review Checklist for Banking

1. **Commercial use** explicitly permitted? (CC-BY-NC models fail here.)
2. **Acceptable-use policy** — does any clause (e.g. "no unlawful use", "no high-risk domains") conflict with your deployment (e.g. credit decisions, advice to clients)? Interpretations differ; get legal sign-off.
3. **Attribution / redistribution** obligations — do they leak into your fine-tuned derivatives? (Apache/MIT derivatives can be relicensed; Llama derivatives keep Llama terms.)
4. **Indemnification** — open weights carry none. If the model outputs cause a loss, the vendor doesn't cover it. This is a *model risk* issue, not just legal — see §23.
5. **Export controls / sanctions** — verify the model origin and the license doesn't create export-compliance issues for your entity's jurisdictions.
6. **Fine-tuned derivatives** — confirm the license covers fine-tuning (training) and serving of the adapter+base combination, and whether your adapters must carry the same license (MIT/Apache: no; Llama: yes for the combined work in some interpretations).
7. **Audit trail** — record license versions and review dates in the model registry (§16); regulators will ask.

> **Bottom line:** for a bank, **Granite (Apache 2.0), Qwen (Apache 2.0), Mistral (Apache 2.0), Phi (MIT), DeepSeek (MIT), OLMo (Apache 2.0)** are the clean set. Llama and Gemma are usable with acceptable-use review. Command-R requires a commercial license from Cohere.

---

## 11. Quantization for On-Prem

### 11.1 The Precision Ladder

| Format | Bytes/param | Quality impact | Notes |
|---|---|---|---|
| **FP16 / BF16** | 2 | Baseline | The reference. BF16 preferred for training; FP16 for inference. |
| **FP8 (E4M3)** | 1 | Minimal (≈FP16) | **Native on H100/H200/B200**; the best quality-per-byte on Hopper+; vLLM `--kv-cache-dtype fp8` too. |
| **INT8 (W8A8)** | 1 | Minimal (~0.5–1%) | SmoothQuant-style weight+activation quantization; broad support. |
| **INT4 (GPTQ / AWQ)** | 0.5 | ~1–2% on 7B–70B | **2× memory savings vs INT8**; AWQ generally more robust than GPTQ at low bit; the workhorse for fitting 70B on 48GB. |
| **GGUF (llama.cpp format)** | 1–6 bits per-weight | Varies by quant level | The llama.cpp/Ollama format (Q4_K_M, Q5_K_M, etc.) — see [Ollama vs Xinference vs LocalAI](ai_llm/ollama_xinference_localai_guide.md). CPU/edge friendly, also served by vLLM. |

### 11.2 Quantized Model Quality — What Actually Happens

- **INT4 is good for 7B–70B**: benchmark deltas of 1–2% on standard evals, often zero on real banking tasks (classification, extraction, summarisation) where the task headroom is large.
- **Smaller models lose more**: a 1–3B INT4 can visibly degrade (it has less redundancy to absorb the noise). For ≤3B, prefer FP8/INT8 or a slightly larger model at INT4.
- **Reasoning models are sensitive**: DeepSeek-R1-distill at INT4 can lose chain-of-thought robustness — test per model, don't assume.
- **Rule: always A/B your own eval set** (the same golden set from §8.2) at FP16 vs quantized before production. See [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md) for the harness; many quantizations ship with published evals but your domain is what matters.

### 11.3 Quantization + Serving Stack Notes

- vLLM supports GPTQ, AWQ, FP8, GGUF, and bitsandbytes; SGLang similar; TensorRT-LLM does FP8/INT8/INT4 with compile-time optimization.
- **KV cache quantization** (FP8 KV) is a separate lever that shrinks the KV-cache footprint without touching weights — worth enabling on H100+.
- **Don't quantize embeddings/rerankers** unless necessary; small models and retrieval quality suffer first.

---

## 12. Fine-Tuning for On-Prem

### 12.1 LoRA / QLoRA vs Full Fine-Tune

| Approach | Cost | Data needed | Best for |
|---|---|---|---|
| **LoRA / QLoRA** | Cheap (1 GPU-class node, hours) | 100–10K examples | **Per-domain adapters** — the default for banking: credit memo style, regulatory Q&A, trade-narrative formatting, tone. QLoRA = LoRA on a 4-bit base → fine-tune 70B on a single 48GB GPU. |
| **Full fine-tune** | Expensive (multi-GPU, days, checkpoint management) | 10K+ high-quality examples | Only when you need to change *behavior* the adapter can't (new knowledge, major format shift). Rarely justified in banking vs RAG + adapter. |
| **Distillation** | Data generation + training | Teacher outputs | Shrinking a frontier teacher into a small deployable student — see [Knowledge Distillation](knowledge_distillation_guide.md). |

**Key economics:** fine-tuning a 7B with QLoRA on one L40S-class GPU costs ~$50–200 in power/amortization per run — trivial vs API fine-tuning fees and *retained control* of the resulting weights.

### 12.2 The On-Prem Fine-Tuning Stack

- **Axolotl** — YAML-driven, the de-facto standard for Llama/Mistral/Qwen fine-tunes; good multi-GPU (FSDP/DeepSpeed) support.
- **Unsloth** — 2–5× faster LoRA/QLoRA with less VRAM; excellent for 7–70B on modest hardware.
- **TRL (HuggingFace)** — `SFTTrainer`/`DPOTrainer`, tight with the HF ecosystem; good for RLHF-style preference tuning if you go there.
- **NeMo (NVIDIA)** — enterprise-grade multi-node training on NVIDIA clusters; heavier but has checkpointing, eval, and model-export tooling.

### 12.3 Adapter Serving (Multi-LoRA)

- vLLM's **multi-LoRA** support serves one base model with many adapters from a single GPU: 1× 70B base + 20 domain adapters (credit, compliance, trade, ...) on one node, switching adapters per request with zero reload.
- This collapses the "one model per business unit" cost problem: **one base, many adapters** is the standard banking pattern for per-domain customisation.
- Governance angle: adapters are *model versions* too — they go through the same registry, approval, and audit as base models (§16, §23).

### 12.4 Fine-Tuning vs RAG — the banking division of labour

- **RAG** for facts that change (policies, rates, client data) — retrieval, not weights.
- **Fine-tuning** for style/behavior/tone that must be deterministic (regulatory response format, internal communication norms).
- Fine-tuning cannot fix hallucinated facts; RAG cannot fix consistently wrong format. Use both — see the RAG series (`advanced_rag_techniques_guide.md`, `rag_optimization_techniques_guide.md`).

---

## 13. The Serving Stack Architecture

### 13.1 Reference Architecture

```
┌────────────────────────────────────────────────────────────────┐
│  CLIENTS  (trading desk apps, branch apps, portals, agents)   │
└──────────────────────────────┬─────────────────────────────────┘
                               │ HTTPS (TLS 1.3, API keys / SSO)
┌──────────────────────────────▼─────────────────────────────────┐
│  APPLICATION LAYER  (RAG pipelines, agent orchestrators,       │
│  copilots — see RAG series: advanced_rag_techniques_guide.md)  │
└──────────────────────────────┬─────────────────────────────────┘
                               │ OpenAI-compatible / REST / gRPC
┌──────────────────────────────▼─────────────────────────────────┐
│  ENTERPRISE AI GATEWAY  (authN/Z, policy routing, rate limits, │
│  quotas, observability, semantic cache, guardrails)            │
└──────────────┬───────────────────────────────┬─────────────────┘
               │ (routing: model per policy)   │
┌──────────────▼───────────────────────────────▼─────────────────┐
│  MODEL ROUTER / LOAD BALANCER  (K8s Service / Nginx / Envoy,   │
│  model-aware routing, canary weights, health checks)           │
└──────────────┬───────────────────────────────┬─────────────────┘
       ┌───────▼────────┐              ┌───────▼────────┐
       │  GPU NODE 1    │              │  GPU NODE N    │
       │  vLLM /        │              │  vLLM /        │
       │  SGLang /      │              │  SGLang /      │
       │  TRT-LLM       │              │  TRT-LLM       │
       │  model:A v1    │              │  model:A v1    │
       │  model:B v3    │              │  model:C v1    │
       └───────┬────────┘              └───────┬────────┘
               │  NVLink TP (in-node)          │
       ┌───────▼───────────────────────────────▼────────┐
       │  INFRA: model registry (object storage),       │
       │  metrics (Prometheus), logs (EFK/ELK),         │
       │  secrets (Vault), K8s control plane            │
       └────────────────────────────────────────────────┘
```

**Key architectural principles:**

1. **The gateway is the only entry point** — applications never talk to GPU nodes directly. This is what makes policy routing, tenant isolation, and audit possible.
2. **OpenAI-compatible API everywhere** — every model server exposes the same `/v1/chat/completions` contract, so the application layer is model-agnostic and the gateway can swap models behind the same endpoint.
3. **Stateless GPU nodes** — nodes pull weights from the registry at startup; no node holds unique state. This is what makes autoscaling, rolling updates, and failover trivial.
4. **One serving engine per node type** — standardise on vLLM for the main pool; keep SGLang/TensorRT-LLM as specialised options (below).

### 13.2 Model Server Components Compared

| Server | Strengths | Weaknesses | On-prem role |
|---|---|---|---|
| **vLLM** | **De-facto standard**: PagedAttention (KV-cache efficiency), continuous batching, OpenAI-compatible API, multi-LoRA, GPTQ/AWQ/FP8/GGUF, prefix caching, speculative decoding, ROCm support. Massive community. | Needs tuning for peak perf; Python-based (fine for prod); some exotic models lag support | **Default engine for the main pool** — see the [serving-llms-vllm skill](mlops/inference/vllm) and [Scalable AI Deployment](ai_llm/scalable_ai_deployment.md) |
| **SGLang** | RadixAttention (prefix reuse across requests), high throughput, fast structured output (constrained decoding), multi-modal | Younger ecosystem; fewer production war stories | Best for **RAG/agent workloads with heavy shared prefixes** |
| **TensorRT-LLM** | NVIDIA's max-performance engine; compile-time optimization; FP8 native; inflight batching | Build/compile per model+GPU+precision (slow iteration); NVIDIA-only; more ops complexity | **Latency-critical flagship workloads** where you accept the compile pipeline |
| **llama.cpp** | CPU/edge, GGUF, zero GPU needed, tiny footprint | Single-node, lower throughput | CPU tier, edge, dev boxes (see [Ollama vs Xinference vs LocalAI](ai_llm/ollama_xinference_localai_guide.md)) |
| **TGI (HF)** | Good HF integration, message streaming, serverless-ish batching | Fewer optimization levers than vLLM/SGLang now | Legacy/ecosystem-fit cases |
| **Ollama / LocalAI / Xinference** | One-command UX, great for dev/pilots | Not built for multi-tenant HA serving at scale | **Dev, PoC, pilot phases only** in banking; migrate to vLLM for prod (see the tool comparison guide) |
| **NVIDIA Triton** | Multi-framework, ensembles, model repository | More generic; LLM perf needs the vLLM/TRT-LLM backends | Heterogeneous fleets (CV+LLM+tabular) |

**Selection rule of thumb:** start everything on vLLM; adopt SGLang where prefix reuse dominates (RAG agents); adopt TensorRT-LLM only for the 2–3 most latency-critical models; keep llama.cpp for CPU/edge; never run Ollama in production serving paths.

### 13.3 Serving Config Example (vLLM)

```bash
# 70B FP16 on 2× H100 (TP2), production settings
vllm serve /models/llama-3-70b-instruct \
  --tensor-parallel-size 2 \
  --gpu-memory-utilization 0.90 \
  --max-model-len 16384 \
  --enable-prefix-caching \
  --enable-chunked-prefill \
  --max-num-seqs 256 \
  --port 8000 \
  --host 0.0.0.0 \
  --served-model-name "llama-70b" \
  --enable-metrics --metrics-port 9090

# 8B INT4 for high-QPS assistant, 1× L40S
vllm serve /models/qwen2.5-8b-instruct-awq \
  --quantization awq \
  --gpu-memory-utilization 0.92 \
  --max-num-seqs 1024 \
  --enable-prefix-caching
```

---

## 14. Inference Serving Patterns

| Pattern | What | When to use |
|---|---|---|
| **Single model per node** | One model, one replica per GPU/node | Simplest ops, predictable perf; default for the flagship model |
| **Multi-model per node** | Several small models share one GPU | Save cost on small-model workloads (7–14B); **isolation caveats** — a batch of one model can starve others; use per-model max-num-seqs and monitoring |
| **Model pools** | Multiple replicas of the same model behind a load balancer | Scale-out: the workhorse for high QPS; pairs with autoscaling (§20.5) |
| **Model mesh / router** | Routing layer selects the right model per request (by task, cost, tenant, or quality policy) | The gateway pattern (§15); the norm in multi-model banks |
| **Multi-LoRA** | One base + many adapters on one GPU, switched per request | Per-domain customisation without per-domain GPUs (§12.3) |
| **Continuous batching** | Requests join/leave the batch dynamically | **Always on** in vLLM — the throughput engine; see [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) |
| **Prefix caching** | Reuse KV blocks for shared prompt prefixes | RAG/agents with long system prompts; can cut prefill cost 30–70%; see the latency guide |
| **Speculative decoding** | Small draft model proposes, target model verifies | 1.5–2.5× decode speedup when latency-bound; see [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md) |
| **Chunked prefill** | Split long prompts into chunks interleaved with decodes | Reduces TTFT variance under batch load |

**Deployment notes for banking:**

- **Pin model versions in the router** — routing by "model name" must resolve to an immutable version (see §16); never route to "latest".
- **Per-tenant isolation** — a rogue batch job must not degrade the trading copilot's latency: give critical tenants dedicated pools (node affinity / taints) and quotas (§15, §19).
- **Degradation ladder** — when the pool saturates: shed low-priority batch work first, then degrade to smaller models, then queue — never silently drop SLO-bound workloads.

---

## 15. The Gateway Layer

The **enterprise AI gateway** is the control plane of the whole deployment — see the dedicated [Enterprise AI Gateway](enterprise_ai_gateway_guide.md) guide. Summary of its on-prem roles:

| Capability | On-prem implementation |
|---|---|
| **Unified API** | One OpenAI-compatible endpoint for *all* models (on-prem + cloud) — apps see a single "AI" |
| **Routing / model selection** | Policy-based: data classification → model tier (small/cheap vs large/quality), tenant, cost budget, capability needs; canary/A-B weights for model rollouts |
| **AuthN/Z** | SSO (OIDC/SAML) for humans, API keys for services, **per-tenant keys** with scopes (which models a tenant may call) |
| **Rate limiting & quotas** | Per-tenant token budgets, QPS limits, hard caps on costly models; protects the GPU pool from one tenant's runaway job |
| **Observability** | Tokens in/out, latency percentiles, **cost per tenant per model**, error rates — the basis for chargeback (§20, §22) |
| **Caching** | Exact-match and **semantic cache** for repeated queries (frequent compliance questions, standard clauses) — big cost lever |
| **Guardrails integration** | Prompt/response filtering, PII detection, jailbreak defense hooked at the gateway — see [LLM Development Risks & Security](llm_development_risks_security_guide.md) |
| **Audit** | Request metadata (tenant, model version, token counts, timestamps) forwarded to the audit store — payload redaction happens here too (§19.5) |

**Open-source gateway options:** LiteLLM (proxy, routing, budgets), Kong AI Gateway, Envoy AI Gateway, Higress (cloud-native AI gateway), plus commercial (Portkey, Kong Konnect, Azure APIM with AI). For a bank: pick one that supports **OpenAI-compatible passthrough, per-tenant keys, semantic caching, and guardrail webhooks**, and that can run fully on-prem.

---

## 16. Model Registry and Versioning

### 16.1 Model Storage

- **Object storage** (MinIO / Ceph / S3-compatible on-prem) or NFS for weight artifacts — **immutable, signed, checksummed**.
- **Signed artifacts**: every model package (weights + tokenizer + config + license + model card) is signed (cosign/sigstore) and checksum-verified at pull time — the same discipline as container images (§18).
- **Serving configs**: per-model YAML (engine, precision, TP size, max context, env vars) versioned *with* the model, so a deployment is reproducible: `model.tar.gz` + `serving.yaml` + `model_card.md` = one deployable unit.

### 16.2 Versioning and Promotion

- **Semantic versioning, immutable**: `granite-34b-instruct:1.2.0` is written once, never overwritten. A "re-quantized" artifact is a new version.
- **Promotion pipeline**: dev → test → prod registries with **approval gates** (quality gates from §21, security scan, license re-check, model-risk sign-off from §23).
- **Model cards**: documentation as a first-class artifact — intended use, training data, eval results, known limitations, license, owner. Regulators increasingly expect this (EU AI Act model documentation, MAS FEAT transparency).

### 16.3 Deployment Strategies

| Strategy | Mechanics | Use |
|---|---|---|
| **Canary** | Route 5% → 10% → 50% → 100% of traffic to new model version via the gateway | Default for any model upgrade |
| **A/B testing** | Two versions serve concurrently, responses compared on business metrics | Choosing between model families/candidates |
| **Rollback** | Registry holds N-1; instant switch at the gateway; GPU nodes pre-warm the previous weights | Any regression (quality, latency, security) |

**Rollback readiness is an architectural requirement, not a nice-to-have**: keep the previous version's weights on local NVMe (disk is cheap; re-download from the registry is slow under incident pressure) and keep a warm standby replica for the flagship model.

---

## 17. Environment Design: Dev / Test / Prod / DR

| Environment | Hardware | Data | Purpose / Rules |
|---|---|---|---|
| **Dev** | CPU or small GPU (workstations, 1× L40S) | Synthetic / masked only | Rapid iteration by data scientists & engineers; no production data; no SLOs |
| **Test** | Representative GPU (same arch as prod, smaller pool) | Synthetic + masked prod-shaped data | Quality gates (§21), integration, load tests at reduced scale, release validation |
| **Prod** | Full GPU pool, HA (≥2 nodes per critical model), real data | Production data with full controls | The governed deployment: autoscaling, monitoring, security, audit |
| **Staging / DR** | Standby GPU pool (could be powered-off capacity or a second DC) | Replicated weights + config, no live traffic | Failover drills, disaster recovery; **weights replicated** to the DR site via registry sync; DR runbook tested quarterly |

**Key rules:**

1. **Data flow is one-way**: prod data never flows to dev/test. Masking at ingestion, not at export.
2. **Model promotion is the same path everywhere**: dev-model → test gates → prod. No "just deploy it from the laptop".
3. **DR is a contract, not a feature**: define RTO/RPO for the LLM service (e.g. RTO 4h, RPO 15min of *config* — weights are immutable artifacts, so RPO is about logs/config), and *drill* it. MAS TRM expects business-impact analysis and testing for critical systems.
4. **Cross-region or secondary DC**: Singapore-based banks often use a second local DC (low latency, data stays in-country) rather than a far-away region — data-residency rules constrain DR topology.

## 18. Air-Gapped Deployment

Many bank environments — especially segregated/compliance zones — have **no internet access at all**. Every artifact must be staged in advance. This is a discipline, not an afterthought: models, containers, and dependencies are all software supply chain (§10 of [LLM Development Risks & Security](llm_development_risks_security_guide.md) covers the threat model).

### 18.1 What Must Be Pre-Staged

| Artifact | How | Tools |
|---|---|---|
| **Container images** | Pull on a connected staging host → push to an **internal registry mirror** (Quay / Harbor / Nexus / Artifactory) → cluster pulls from internal mirror only | skopeo, crane, Harbor replication, Quay mirroring |
| **Model weights** | Download from Hugging Face on connected host → verify checksums → transfer via **secure transfer or sneaker-net** (encrypted media for truly isolated networks) → store in internal model registry | `huggingface-cli download`, `hf_transfer`, checksum + cosign verification, registry replication |
| **Python / system deps** | Internal **PyPI / npm / apt mirrors** (Devpi, Nexus, Artifactory, Pulp) | pip `--index-url`, Nexus proxy repos |
| **Vulnerability DBs** | Scan with **offline CVE databases** (Trivy/Grype with local DB, updated on a schedule via media or a one-way sync) | trivy, grype, `--offline-scan` |
| **SBOM** | Generate SBOMs at build time, store beside images; required for supply-chain attestation | syft, cosign attest, SPDX/CycloneDX |
| **Signing keys / certs** | Internal CA, cosign keys — provisioned out-of-band | cosign, Vault PKI |
| **Model updates** | New weights arrive via the same secure transfer path, validated (checksum + signature) before promotion | registry promotion pipeline |

### 18.2 The Air-Gap Operating Rhythm

1. **One-way sync**: a single connected "staging" zone with strict egress allowlist; everything moves *into* the environment, nothing phones home.
2. **Verify at every boundary**: checksum + signature verification at ingest, at registry push, and at node pull. Trust-on-first-use is not acceptable for weights.
3. **Offline license compliance**: the license texts, OSI/Apache terms, and your legal review notes are stored *inside* the model registry as artifacts — legal review happens on the connected side, the *result* travels in.
4. **Patch process**: OS/container patches come via media or one-way sync on a cadence; the vulnerability scan runs against the updated local DB and produces an exception report for any unpatched items (documented for audit).
5. **Model update cadence**: new weights are a *change* (see §23 change management) — transfer, validate, promote through dev/test gates, then prod. Expect days, not hours, per cycle. Plan model freshness expectations accordingly (§2.1).

> **Reference:** the air-gap section of [OpenShift AI Alternatives](openshift_ai_alternatives_guide.md) covers the platform-level air-gap pattern (registry mirroring, Operator catalogs, disconnected installs); this guide assumes that foundation and focuses on the LLM-specific artifacts above.

---

## 19. Security and Networking

### 19.1 Network Zones (Defense in Depth)

```
        ┌──────────┐     ┌──────────────┐     ┌──────────────────┐
        │   DMZ    │ ──► │  APP ZONE    │ ──► │    GPU ZONE      │
        │ (edge,   │     │ (gateway,    │     │ (model servers,  │
        │ WAF,     │     │  apps, RAG   │     │  registry pulls) │
        │ ingress) │     │  services)   │     │                  │
        └──────────┘     └──────────────┘     └──────────────────┘
                              │  ▲                  │    ▲
                              │  │ egress: allowlist │    │ egress: NONE
                              └──┘ (to GPU zone only)┘    └ (no internet)
```

- **GPU nodes have NO direct internet** — egress is blocked or strictly allowlisted (at most: monitoring, internal registry). This is the single most important rule; it is what makes "no data leaves premises" mechanically true.
- **App zone → GPU zone only**: applications cannot reach GPU nodes directly; they go through the gateway, which is the only ingress to the GPU zone.
- **Registry/control-plane zone**: model registry, K8s API, Vault, monitoring — separated from both app and GPU zones; admin access via bastion + MFA.

### 19.2 Transport Security

- **TLS everywhere** (1.2+ minimum, 1.3 preferred) — ingress, inter-service, and node-to-registry.
- **mTLS for service-to-service** within the mesh (Istio/Linkerd/Cilium) — both sides authenticate; prevents lateral movement from a compromised app pod.
- **API gateway authentication** at the edge: OIDC for humans, API keys for machines, short-lived tokens; keys stored in Vault, rotated on a schedule.

### 19.3 Secrets Management

- **HashiCorp Vault** (or equivalent) for: model access tokens (Hugging Face tokens if any), registry credentials, DB creds, signing keys.
- Secrets injected at runtime (K8s CSI/Vault agent), never baked into images or serving configs.
- **Rotation**: model access tokens, gateway API keys rotated on schedule; emergency rotation on personnel change.

### 19.4 Model Access Control

- **Who can query which model** — RBAC at the gateway: tenant → allowed models → token budgets (§15).
- **Data classification check** — the gateway can refuse requests lacking the required classification header for confidential-data-only models.
- **Sensitive-model tier** — models fine-tuned on client data get stricter access (fewer principals, shorter token budgets, extra logging) than public-model tier.

### 19.5 Prompt / Data Logging and Audit Trail

- **Log metadata, not payloads** by default: tenant, model version, token counts, latency, timestamps, decision IDs.
- **Payload logging is opt-in and redacted**: when required (compliance cases, incident forensics), PII detection/redaction runs **before** the payload touches the log store; redaction is a pipeline (NER + regex + allowlist), not a single regex.
- **Retention**: audit logs follow the bank's records schedule (often 5–7 years for regulated records); operational logs 30–90 days. Deletion is *also* governed (PDPA/GDPR right to erasure doesn't stop at AI).

### 19.6 Inference Isolation and GPU Security

- **Tenant isolation**: separate model pools per tenant tier (critical vs batch), per-tenant quotas, no cross-tenant batching in multi-tenant nodes (use dedicated nodes or MIG partitions).
- **MIG partitioning** (NVIDIA Multi-Instance GPU): slices one physical GPU into isolated instances with dedicated compute/memory — good for small-model multi-tenancy on A100/H100; note MIG has **no NVLink** and fixed slice shapes. See the [OpenShift AI Alternatives](openshift_ai_alternatives_guide.md) coverage of GPU partitioning.
- **Side-channel considerations**: multi-tenant GPU inference on a *shared* GPU exposes timing/memory side channels between tenants. For regulated or adversarial-multitenant workloads, prefer **dedicated GPUs, MIG, or vGPU with strong isolation** over shared-address-space serving. The practical banking stance: internal tenants are trusted-ish; external/third-party tenants get dedicated resources.
- **GPU node hardening**: driver/kernel locked versions, no SSH from the app zone, node images signed, runtime scanning.

---

## 20. Operations: Monitoring, Alerting, Logging, Capacity

### 20.1 Monitoring

Two planes: **infrastructure** (host, GPU, network) and **model service** (SLO metrics).

| Category | Metrics |
|---|---|
| **GPU health** | Utilization %, VRAM used/free, HBM temperature, power draw, ECC errors, NVLink errors (nvidia-smi / DCGM exporter / Prometheus) |
| **Throughput** | Tokens/sec (in/out), requests/sec per model, **batch size** (continuous batching fills) |
| **Latency** | **TTFT** (time to first token), **TPOT** (time per output token), inter-token latency, p50/p95/p99, per model and per tenant |
| **Queue** | Queue depth, queue wait time, request rejection rate — the earliest warning of saturation |
| **Serving engine** | KV-cache usage %, prefill vs decode time split, prefix-cache hit rate, max-num-seqs utilization (vLLM Prometheus metrics) |
| **Errors** | 4xx/5xx rates, timeouts, aborted streams, model load failures |
| **Business** | Cost per query (tokens × model rate), cost per tenant, successful-completion rate |
| **Power** | Per-node power draw (kW), efficiency (tokens/kWh) — capex review fodder (§22) |

### 20.2 Alerting

| Alert | Trigger | Response |
|---|---|---|
| **GPU OOM** | VRAM usage > 90% or OOM errors | Reduce max-num-seqs, shed batch load, scale out, check KV-cache sizing |
| **Latency SLO breach** | p95 TTFT/TPOT above budget for N minutes | Scale out, enable/verify prefix caching, shed low-priority traffic, check batch starvation |
| **Queue buildup** | Queue depth > threshold for N minutes | Autoscale, shed batch work, notify on-call |
| **Model health** | Periodic **test prompts** with expected outputs (canary prompts per model) fail or drift | Page platform team; gateway fails traffic to standby/older version |
| **Drift detection** | Response-quality drift vs golden set (embedding similarity / eval score) on sampled traffic | Trigger re-evaluation (§21 regression) |
| **Node loss** | GPU node down / ECC errors / NVLink degradation | Failover via router; incident process |
| **Power anomaly** | Node power outside envelope | Check cooling, throttle if thermal |

### 20.3 Logging

- **Request logs** (redacted) with: tenant, **model version** (immutable ID — mandatory for any dispute/audit), prompt length, output length, latency breakdown, error codes.
- **Serving logs** (engine-level) rotated and retained short-term; **audit logs** long-term (see §19.5).
- **Correlation**: one request ID through gateway → router → engine → RAG, so a compliance question ("what did this model output for this client on this date?") is answerable end-to-end.

### 20.4 Capacity Planning

**The capacity formula:**

```
GPUs ≈ (peak_QPS × avg_output_tokens × 2 × quantization_factor) / (GPU_tok_per_s)
```

- `peak_QPS × avg_output_tokens` → tokens/sec of *generation* demand.
- `× 2` — prefill+decode multiplier: prefill work (prompt tokens + compute) is roughly as costly as decode at the system level, and the multiplier also absorbs batching overhead and headroom. Tune this factor with your own measurements.
- `quantization_factor` — INT4 ≈ 0.5× vs FP16 weights for memory; the formula's tok/s already reflects bandwidth, so keep this factor for memory-driven constraints (or set to 1 and rely on GPU tok/s from §5).
- `GPU_tok_per_s` — *sustained, batched* throughput from load tests (§21), not theoretical single-stream tok/s.

**Process:**

1. Model per-workload demand (QPS × tokens) from business forecasts; sum to peak (not average).
2. Apply the formula → baseline GPU count.
3. **Headroom 20–30%** above peak for burst, rollouts, and tenant isolation.
4. **Growth planning**: token volume grows ~2–5× per year as adoption spreads; buy capacity in cohorts with 12–18 month lookahead, and design for **GPU generational refresh** (see §22.3 hidden costs).
5. **Peak vs average**: the bank's peak (month-end reporting, market-open) can be 3–5× average; decide whether to size for peak (costly) or ride it with autoscaling + degradation ladder (§14).

### 20.5 Autoscaling

- **KEDA** with **queue-based scalers** (request queue depth, or vLLM metrics via Prometheus) is the standard on K8s: scale replicas on queue buildup, scale to zero for batch tiers.
- **GPU-aware scaling**: scale on *tokens/sec demand* and KV-cache utilization, not just QPS — a long-context request consumes far more than a short one.
- **Node-level**: cluster-autoscaler-style node provisioning for GPU pools (with careful lead-time and power planning), or pre-provisioned standby nodes for critical tiers (HA trumps elasticity for the flagship model).
- **Don't autoscale the flagship**: trading copilots want pre-warmed replicas, not cold starts. Autoscale the batch/small-model tiers aggressively instead.

### 20.6 Rolling Updates and Zero-Downtime Model Swaps

1. New model version passes gates (§21) and is loaded to the registry.
2. **Pre-warm** a replica of the new version on a node (weights pulled from registry/local NVMe).
3. Gateway **canary** 5% → 50% → 100% (§16.3), watching SLO alerts.
4. Old replicas drain (stop new requests, finish in-flight) and are torn down.
5. **Rollback** = flip gateway back to previous version (kept warm for 72h).

### 20.7 Backup / Recovery

- **Model registry backup** (weights + configs + model cards) to a second store/DC — immutable, versioned, DR-replicated.
- **Config backup** — gateway config, serving configs, quotas: treat as code (GitOps), so recovery is `git checkout` + apply.
- **DR runbook** — documented, quarterly-tested: failover to standby pool / DR site, RTO/RPO validated, post-drill report to risk (MAS TRM alignment).

## 21. Testing and Validation

LLMs fail in ways traditional software doesn't: a model can pass unit tests and still produce policy-violating output. The test pyramid for on-prem LLM deployment:

### 21.1 Model Acceptance Testing (Quality)

- **Domain golden set**: 200–1000 curated prompt/expected-output pairs *from your actual banking use cases* (credit memos, compliance Q&A, trade extraction, client comms). Vendor benchmarks are not your acceptance criteria.
- **Automated scoring**: LLM-as-judge + exact-match + embedding-similarity + task-specific metrics (extraction F1, classification accuracy). See [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md) for the harness.
- **Human review pass** on a stratified sample (edge cases, sensitive topics) before prod promotion.
- **Gate**: candidate model must beat (or match within tolerance) the incumbent on the golden set; document the delta in the model card.

### 21.2 Latency Testing

- **Realistic concurrency, not curl loops**: k6, Locust, or Gatling against the *gateway* (full path), with realistic prompt-length distributions and streaming clients.
- **Measure**: TTFT p50/p95/p99, TPOT, tokens/sec under load, queue wait.
- **Test at 30/60/80/100% of expected peak** to map the latency-vs-load curve — the shape of this curve is your autoscaling trigger design.

### 21.3 Capacity Testing

- **Find the knee**: ramp concurrency until SLO breach; record max sustainable QPS and tokens/sec per model per GPU config.
- **Use the output to calibrate the §20.4 formula's `GPU_tok_per_s`** — never size a fleet on theory alone.
- **Long-context soak**: sustained 4h+ runs to catch KV-cache leaks and memory fragmentation.

### 21.4 Failover Testing

- **Node loss**: kill a GPU node mid-traffic → verify router failover, in-flight request handling, recovery time.
- **Model crash**: kill a serving pod → verify gateway health-check → restart with pre-warmed weights.
- **DR drill**: full failover to standby pool/DR site quarterly (§17).
- **Degradation ladder drill**: saturate the pool → verify low-priority shedding behaves as designed.

### 21.5 Security Testing

- **Prompt injection**: adversarial prompts against the deployed stack (system-prompt extraction, indirect injection via RAG documents) — see [LLM Development Risks & Security](llm_development_risks_security_guide.md) and the [Prompt Injection](ai_llm/prompt_injection_guide.md) guide.
- **Red teaming** rounds before major releases and on a schedule for the flagship model.
- **PII leak checks**: verify redaction pipeline (prompts containing synthetic PII must not reach logs).
- **Access control tests**: tenant A must not reach tenant B's models; revoked keys rejected; rate limits enforced.

### 21.6 Regression Testing

- **The golden set is versioned** and runs on *every* candidate model version before promotion — the same way a test suite runs on every commit.
- **Per-model regression**: quantization change, context-length change, fine-tune refresh, and *base-model upgrade* (a new Llama release re-qualifies all adapters built on the old base).
- **Drift monitoring in prod** (§20.2) feeds back into the regression suite.

---

## 22. Cost Analysis: On-Prem TCO vs API

### 22.1 The On-Prem Cost Stack

| Component | Typical cost | Notes |
|---|---|---|
| **GPU servers** | **$100k–500k per 8×GPU node** (A100 80GB: ~$150–250k; H100: ~$250–400k; B200-era: $400k+) | Depreciation 3–5 years; second-hand A100s are a legitimate entry point |
| **Storage + networking** | $20–60k per node-equivalent (NVMe, 100GbE fabric, switches) | |
| **Power** | ~5–10 kW/server × $0.10–0.20/kWh × 24×365 ≈ **$5–17k/year per node** | Singapore industrial tariffs are at the higher end; cooling adds ~30–50% |
| **Cooling + space** | Rack, cooling (air or liquid), DC floor space | Liquid cooling capex for dense nodes |
| **People** | **0.5–1.5 FTE platform engineer per ~8–16 GPUs** + ML engineers; ~$100–200k/yr loaded cost each | The line most TCO models forget — see §22.3 |
| **Software** | Serving stack (vLLM/SGLang/TRT-LLM) is **open source**; enterprise support optional ($10–50k/yr) | No per-token license |
| **Maintenance** | Hardware refresh, GPU failures (HBM failures are not rare), driver/kernel upkeep | Budget 3–5%/yr of capex |

### 22.2 API Cost Reference Points (mid-2026 list prices)

| Model class | Input | Output | Notes |
|---|---|---|---|
| Frontier (GPT-4o-class / Claude-class) | **~$2.5–5/M tokens** | **~$10–15/M tokens** | Output is 3–4× input — generation-heavy workloads pay dearly |
| Mid (GPT-4o-mini-class) | ~$0.15–1/M | ~$0.6–4/M | |
| Hosted open-weights (Llama-70B-class) | **~$0.5–1/M** | ~$0.5–2/M | "API margin" on open weights: the same model you can run yourself |

**Example — a 100-seat assistant, 2,000 tokens out per conversation, 50 conversations/seat/day = 10M output tokens/day:**

- Frontier API output-only: 10M × $12.5/M ≈ **$125k/month** (+ input).
- On-prem (4× 8×H100 nodes ≈ $1.2M capex, $60k/yr power, 1 FTE): ≈ $30–35k/month all-in first year, dropping to ~$10–15k/month afterwards.
- Hosted Llama-70B API: 10M × $1/M ≈ $10k/month + input — cheaper than on-prem *until* volume grows 5–10×, *and* you still have the data-residency problem.

### 22.3 Break-Even Analysis — When Each Side Wins

**On-prem wins when (any of):**

- Sustained volume **> 10–50M tokens/day** (the range depends on model class, utilization, and your power/labor rates — model it with your own numbers).
- **Sensitive data premium**: the compliance cost of API use (DPA review, transfer assessments, vendor audits, incident risk) is real money — often $50–200k/yr of legal/risk effort that on-prem removes.
- **Predictable workloads** (steady internal assistants, batch document processing) that keep utilization high.
- You need **fine-tuned models** served at scale (API fine-tuning + inference margin compounds).

**API wins when:**

- **Variable/spiky workloads** — paying for idle GPUs is worse than paying per token.
- **Small volume** (< a few M tokens/day) — capex never amortises.
- **Fastest time-to-market** — a pilot in days, zero infrastructure.
- **Model freshness** — you must be on the newest frontier model the day it ships (on-prem model cadence is weeks-months, §18.5).

**The hybrid answer:** use API for spike absorption and frontier access; run on-prem for the steady-state sensitive load. The gateway's policy routing (§2.3, §15) makes this operational rather than architectural.

### 22.4 The Hidden Costs (The Ones That Bite)

1. **GPU utilization** — idle GPUs are wasted capital. A pool averaging **<40% utilization** may be *more expensive per token* than an API. Target **50–70% average utilization**; use batch workloads to fill troughs; autoscale-to-zero batch tiers; and be honest that "we have GPUs, so use them" is not a cost model.
2. **Model churn** — every new model family is re-evaluation (§8.2 golden set), re-quantization, re-fine-tune, re-certification (§23). Budget 1–2 weeks of platform team per model refresh, several per year.
3. **Upgrade cycles** — GPU generations leap (H100 → B200 ≈ 2× perf and memory); your 3-year-old fleet is *functionally* fine but *economically* obsolete vs new SKUs. Plan refresh cohorts and resale, don't run GPUs to failure.
4. **Staffing** — GPU admin + ML platform + MLOps is a scarce skill set; recruiting/retaining it in Singapore is expensive and slow. If you can't staff it, you can't run on-prem — this alone has killed more bank AI programs than hardware cost.
5. **Power cost escalation** — DC power prices and cooling constraints are worsening; a 100kW+ GPU footprint is a multi-year commitment.

---

## 23. Governance and Compliance

### 23.1 Model Governance

- **Model inventory** — every model (base and adapter) in the registry with owner, business use, data lineage, license, and status. If it's not in the registry, it doesn't exist (shadow models are a §25 pitfall).
- **Approval workflow** — new model / new version / new use case requires sign-off: business owner + model risk + security + legal (license). Use-case-level approval, not just model-level.
- **Model risk assessment (SR 11-7 lens)** — treat the LLM as a *model component* within a broader system: the OCC/Fed guidance expects validation, documentation, ongoing monitoring, and **effective challenge**. For an LLM: documented eval methodology, known limitations, human-oversight controls, and periodic revalidation. MAS FEAT maps the same expectations locally.
- **Validation** — independent (not the builder) review of the eval methodology and results before prod; periodic revalidation on a schedule and on material change (new base, new fine-tune data).

### 23.2 Data Governance

- **What data goes into prompts** — data-classification policy enforced at the gateway: confidential data → on-prem models only; PII minimization (prompt only what the task needs); no client data in dev/test.
- **Retention & deletion** — prompt/response logs: retention schedule, deletion process, and erasure requests handled like any PII store (PDPA/GDPR).
- **Training-data governance** — fine-tuning datasets: provenance, consent, masking, and approval before training. A fine-tune on unmasked client data is a data breach in waiting.

### 23.3 Audit Trail

- Full request logs: tenant, model **version**, tokens, latency, decision IDs (§20.3).
- **Decisions** (model approvals, promotion approvals, change records) logged in the same audit store.
- Regulators will ask: *"which model answered this client query in March?"* — the answer must be one query away.

### 23.4 Change Management

- Model updates go through the **formal change process** (RFC/change ticket, risk assessment, test evidence, approval, rollback plan) — same as any production change in a bank. No "hotfix model drops".
- Emergency rollback is pre-approved and rehearsed (canary/rollback drill §20.6).

### 23.5 Vendor Governance

- **NVIDIA/AMD relationships**: support contracts, driver/kernel maintenance windows, hardware **EOL planning** (NVIDIA GPU EOL cycles are real — plan fleet refresh 12–18 months ahead).
- **Software vendors**: if you buy enterprise support (vLLM enterprise, SGLang support), contract SLAs and escalation paths.
- **Open-source governance**: SBOMs, license compliance, CVE monitoring for the serving stack itself — you now *own* what the API provider used to own.

### 23.6 Regulatory Mapping Summary

| Regulation | Scope | On-prem LLM response |
|---|---|---|
| **MAS TRM** | Technology risk management: availability, security, change management | HA pools, DR drills, patching cadence, change process, BIA for LLM services |
| **MAS FEAT** | AI governance: fairness, ethics, accountability, transparency | Model cards, human oversight, bias evaluation on golden sets, documented accountability |
| **PDPA (SG)** | Personal data protection | Data stays on-prem, access control, retention/deletion, breach response |
| **GDPR (EU)** | Data protection + transfer | No cross-border transfer; DPIA where relevant; erasure support |
| **BCBS 239** | Risk data aggregation & reporting | Data lineage: prompts → models → outputs → reports traceable end-to-end |
| **EU AI Act** | AI system lifecycle (high-risk) | Full deployer control, monitoring, logging, human oversight, documentation — all native to on-prem |
| **SR 11-7 (US, but the industry benchmark)** | Model risk management | Validation, documentation, ongoing monitoring, effective challenge |

> **The honest framing:** on-prem makes compliance *controllable*, not automatic. Every control above still has to be implemented, evidenced, and audited. The difference is that the evidence is in your hands rather than a vendor's attestation.

## 24. The Deployment Playbook: Step by Step

The seven-stage path from zero to a governed on-prem LLM platform. Typical duration for a bank: **6–12 months** from kickoff to production for the first workloads.

### Step 1 — Requirements Assessment (weeks 1–4)

- **Workloads**: enumerate candidate use cases (assistant, summarisation, extraction, classification, RAG copilots) with owners and business value.
- **Volume**: forecast tokens/day per workload (from business activity, not IT guesses) → peak vs average.
- **Latency**: SLO per workload (interactive <2s p95 TTFT, batch minutes).
- **Budget**: capex ceiling, opex run-rate, depreciation policy, power budget.
- **Compliance**: data classification per workload, regulatory mapping (§23.6), internal AI policy check.
- **Skills**: honestly assess platform engineering bench (GPU ops, K8s, MLOps). If absent, this step also produces the hiring/training plan.
- **Decision gate**: run the §2 scoring sheet per workload → on-prem / hybrid / API split.

### Step 2 — Model Selection (weeks 3–6)

- Shortlist families by license (§10), quality on a mini golden set (§8.2), multilingual needs, ecosystem maturity.
- Choose sizes + precision per workload tier (§8.3, §11).
- Build the initial **golden evaluation set** from real (masked) use-case data.
- **Legal review** of licenses before any download.

### Step 3 — Hardware Sizing (weeks 4–8)

- Apply the VRAM formula (§4) and capacity formula (§20.4) with *measured* tok/s from reference benchmarks.
- Choose GPU SKUs + node form factors (§7.2); validate with vendor benchmark data and your own micro-benchmarks on rented/borrowed hardware if possible.
- Size storage (registry + NVMe), network fabric, power/cooling (§7.5) — and get the **power study signed off** before PO.
- Plan the procurement timeline (GPU lead times are the critical path).

### Step 4 — PoC (weeks 8–14)

- **One node, one model, vLLM** (or the dev-tier tooling from [Ollama vs Xinference vs LocalAI](ai_llm/ollama_xinference_localai_guide.md) if the team is new).
- Validate **quality** (golden set vs expected) and **latency** (TTFT/TPOT under realistic load) — this is where the sizing assumptions get corrected.
- Validate the *data path*: registry → node → API contract → one pilot app.
- **Exit criteria**: quality acceptable, latency within SLO at target concurrency, ops team can run it (documented runbook v1).

### Step 5 — Pilot (months 3–6)

- **2–4 models**, the gateway (§15), basic monitoring (§20.1), one real business use case in production-like conditions.
- Stand up dev/test/prod separation (§17) and the model registry with promotion gates (§16).
- Run the pilot with a real business team; capture feedback on quality *and* on the ops experience.
- Begin the change-management and governance artifacts (§23) — do not defer them.
- **Exit criteria**: one workload live, SLOs met for 30 days, cost model validated against forecast, governance artifacts approved.

### Step 6 — Production (months 6–10)

- **HA**: ≥2 replicas per critical model, rollback readiness (§20.6), DR standby (§17).
- **Autoscaling** for batch/small-model tiers; degradation ladder (§14).
- **Full monitoring/alerting** (§20.1–20.2), audit logging with redaction (§19.5), secrets via Vault.
- **Security hardening**: network zones, mTLS, egress lockdown for GPU nodes (§19).
- **Capacity review process** established (monthly, §20.4).
- **Governance live**: model approvals, change management, incident process, quarterly model risk review.

### Step 7 — Operate (continuous)

- Monthly: capacity vs forecast, utilization review, cost-per-token review, alert tuning.
- Per model release: re-evaluation → re-qualification → canary rollout (§20.6).
- Quarterly: DR drill, red team round, license/regulatory review, platform backlog.
- Annual: TCO refresh vs API (§22), hardware refresh planning, strategy review (on-prem vs hybrid mix).

---

## 25. Common Pitfalls

| # | Pitfall | Symptom | Prevention |
|---|---|---|---|
| **P1** | **Undersized GPU** | OOM at peak, model won't load with target context | VRAM formula (§4) + headroom; test max context, not nominal |
| **P2** | **No KV-cache headroom** | Latency collapse under batch load, OOM at 60% concurrency | Account for KV cache at *max concurrent sequences × max context*; monitor `gpu_cache_usage_perc` |
| **P3** | **Single point of failure** | One node dies → service down | ≥2 replicas per critical model, router health checks, warm standby |
| **P4** | **No autoscaling** | Idle GPUs at night, overloaded at 9am | KEDA queue-based scaling; autoscale batch tiers aggressively |
| **P5** | **Skipping PoC quality validation** | Production model underperforms on real data | Golden set from day one (§24 step 2); gate every promotion |
| **P6** | **License violations** | Legal discovers non-commercial model in prod | License review before download (§10.2); registry stores license + approval |
| **P7** | **No air-gap plan** | Model can't be loaded into segregated zone; weeks of delay | Stage registry/deps/mirrors in step 3–4 (§18), not at deployment time |
| **P8** | **No model governance (shadow models)** | Unsanctioned models serving client data | Registry + approval workflow + "no registry, no prod" rule (§23.1) |
| **P9** | **Ignoring quantization loss** | Quality regression after INT4 adoption, blamed on "the model" | A/B golden set FP16 vs quantized before prod (§11.2) |
| **P10** | **No load testing** | SLO breach on day one of production | Capacity tests in step 4–5, calibrate formula (§21.3) |
| **P11** | **Over-buying** | Pool at 15% utilization; CFO asks why | Buy in cohorts, autoscale-to-zero, batch fill, utilization targets (§22.4) |
| **P12** | **Model churn without plan** | Every new Llama release becomes a fire-drill re-certification | Fixed cadence + re-qualification pipeline (§21.6); don't chase every release |
| **P13** | **Security gaps (no egress control)** | GPU node can reach the internet; data path unproven | Egress lockdown from day one (§19.1); verify with egress tests |
| **P14** | **TP across PCIe** | 70B "split across 2 cheap GPUs" is 3× slower than 1 big GPU | TP only over NVLink (§7.1); prefer fewer/bigger GPUs |
| **P15** | **Staffing gap** | GPUs idle because nobody can operate them | Skills plan in step 1; managed services / vendor support as bridge |

---

## 26. The Future: On-Prem LLM 2026 and Beyond

1. **Smaller efficient models** — distillation keeps closing the gap to frontier (see [Knowledge Distillation](knowledge_distillation_guide.md)): 7–32B models doing what 70B did two years ago. This is *the* on-prem enabler — it lowers the hardware bar every quarter.
2. **MoE inference optimizations** — expert-parallel serving (vLLM/SGLang) makes 671B-class MoE practical on modest clusters; expect MoE to dominate the on-prem flagship tier.
3. **Open-weight models approaching frontier quality** — the gap to closed frontier APIs narrows; for banking domains (where the marginal quality difference is small vs the data-control benefit), open weights increasingly win the decision framework.
4. **On-prem RAG + agents become standard** — retrieval and agent orchestration are already on-prem native (see the RAG series); expect agent platforms to assume local inference as the default backend.
5. **Hardware evolution** — Blackwell Ultra, AMD MI400, and specialized inference chips (Groq LPU successors, custom ASICs) push the tok/$ and tok/W curves; the 192GB+ single-GPU class keeps making large models single-node feasible.
6. **Liquid cooling becomes standard** for dense GPU zones — power density forces the issue; plan DC retrofits now.
7. **Sovereign AI initiatives** — EU and Singapore data-residency mandates are driving *national* on-prem/private-cloud AI capacity; banks are anchor tenants. "Sovereign AI" is now a procurement category, not a slogan.
8. **On-prem + edge hybrid** — branch-level and desk-level edge inference (1–8B INT4 on small servers/endpoints, via llama.cpp/ONNX) for low-latency, offline-tolerant workloads, with the data center pool as the authority tier. See [Ollama vs Xinference vs LocalAI](ai_llm/ollama_xinference_localai_guide.md) for the edge tooling.
9. **Inference cost collapse** — hardware + efficiency gains are pushing on-prem tok/$ down ~2× per generation; the break-even point keeps moving in on-prem's favor for steady-state loads.

---

## 27. Conclusion

On-prem LLM deployment in a bank is not "hosting a model" — it is **building a governed, observable, resilient inference platform** that happens to speak OpenAI-compatible APIs. The seven pillars (sovereignty, compliance, security, latency, cost, customization, resilience) make the *direction* clear for regulated institutions; the decision framework keeps it honest workload-by-workload; and the hybrid pattern with a policy-routing gateway is the realistic operating model — sensitive workloads on-prem, non-sensitive bursts via API.

The disciplines that separate success from failure are unglamorous: **sizing math done before the PO, a golden evaluation set owned by the business, an air-gap plan that predates the segregated zone, a model registry that makes shadow models impossible, and a capacity review that treats idle GPUs as the cost they are.** None of it is exotic — it is classical enterprise engineering applied to a new workload, with the added discipline that the "software" can be wrong in ways that don't crash.

**The one-line summary:** buy GPUs for the workloads that must never leave the building, run them with vLLM behind an enterprise AI gateway, govern them like models (not services), and let the hybrid split absorb everything else.

---

*End of guide — see the companion guides for the adjacent topics: [Enterprise AI Gateway](enterprise_ai_gateway_guide.md), [LLM Latency Optimization](ai_llm/llm_latency_optimization_guide.md), [Ollama vs Xinference vs LocalAI](ai_llm/ollama_xinference_localai_guide.md), [OpenShift AI Alternatives](openshift_ai_alternatives_guide.md), [LLM Development Risks & Security](llm_development_risks_security_guide.md), [LLM Evaluation Frameworks](ai_llm/llm_evaluation_frameworks_guide.md), and [Knowledge Distillation](knowledge_distillation_guide.md).*
