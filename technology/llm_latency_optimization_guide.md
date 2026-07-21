# LLM Latency Optimization Techniques

> A comprehensive guide to understanding, measuring, and reducing the time Large Language Models take to generate responses in production. Critical for real-time AI applications where user expectations demand sub-second or near-real-time interactions.

---

## Table of Contents

1. [Why Latency Matters](#1-why-latency-matters)
2. [Measuring Latency](#2-measuring-latency)
3. [Model-Level Optimization](#3-model-level-optimization)
4. [Serving Infrastructure Optimization](#4-serving-infrastructure-optimization)
5. [Inference Engine Comparison](#5-inference-engine-comparison)
6. [Serving Architecture Patterns](#6-serving-architecture-patterns)
7. [Prompt Engineering for Latency](#7-prompt-engineering-for-latency)
8. [Network and Infrastructure Optimization](#8-network-and-infrastructure-optimization)
9. [Caching Strategies](#9-caching-strategies)
10. [Latency vs Throughput Trade-off](#10-latency-vs-throughput-trade-off)
11. [Measurement and Monitoring](#11-measurement-and-monitoring)
12. [LLM Latency by Model Size](#12-llm-latency-by-model-size)
13. [Cost-Latency Optimization Framework](#13-cost-latency-optimization-framework)
14. [Tools and Techniques Comparison](#14-tools-and-techniques-comparison)
15. [Decision Framework](#15-decision-framework)

---

## 1. Why Latency Matters

Latency is the single most visible performance metric for LLM-powered applications. Every millisecond a user waits erodes trust, reduces engagement, and in business contexts, costs money.

### 1.1 User Experience Expectations

| Interaction Type | Target Latency | User Perception |
|---|---|---|
| Autocomplete / Code completion | < 500 ms | Must feel instantaneous |
| Chat / Conversational AI | < 2 s | Feels natural; >3 s causes impatience |
| Real-time translation | < 1 s | Preserves conversation flow |
| Search / RAG queries | < 1 s | Competitive with traditional search |
| Complex reasoning / Analysis | < 3–5 s | Acceptable with progress indicators |
| Batch processing / Async | 5–30 s | Acceptable with progress bars |

**Key insight:** Perceived responsiveness is dominated by **time-to-first-token (TTFT)** — users judge speed from the moment the first character appears.

### 1.2 Cost Correlation

Faster inference directly reduces operational costs:
- **Higher throughput per GPU:** A model serving 100 req/s on one GPU costs 10× less per request than one serving 10 req/s.
- **Reduced GPU fleet size:** If each request takes half the time, you need half the GPUs for the same load.
- **Better utilization:** Sub-second TTFT allows tighter batching, increasing utilization from ~30% to 70%+.

The relationship is non-linear: a 2× latency improvement can yield 3–5× cost reduction at scale due to tighter GPU packing.

### 1.3 Banking Context (Crédit Agricole CIB)

| Use Case | Latency Budget | Rationale |
|---|---|---|
| Real-time fraud detection | < 100 ms | Transaction must be approved before next action |
| Customer service chatbot | < 2 s | SLA-bound; banking customers expect faster responses |
| Compliance checks (AML/KYC) | < 5 s | Transaction screening across multiple models |
| Trade summarization | < 3 s | Trader needs context before next decision |
| Document analysis | < 10 s | Legal/contract review |
| Report generation | < 30 s | Overnight batch jobs |

Latency optimization directly impacts regulatory compliance SLAs, customer satisfaction, and cost per transaction.

### 1.4 TTFT vs TPS — The Two Key Metrics

**Phase 1 — Prefill (TTFT):** The input prompt is processed as a batch in one forward pass. Compute-bound (GEMM). GPU utilization >80%. Scales with input length.

**Phase 2 — Decode (TPS):** Tokens generated one at a time (autoregressive). Memory-bandwidth-bound — needs to load full model weights from HBM per step. GPU utilization 5–20%. Scales with output length.

**Optimization implications:** Prefill-focused techniques (FlashAttention, prefix caching) improve TTFT. Decode-focused techniques (quantization, KV cache optimization, speculative decoding) improve TPS. A balanced strategy targets both phases.

---

## 2. Measuring Latency

You cannot optimize what you cannot measure.

### 2.1 Key Metrics

| Metric | Definition | Typical Range | Critical For |
|---|---|---|---|
| **TTFT** | Time from request receipt to first output token | 50 ms – 5 s | Perceived responsiveness |
| **TPS** | Output tokens per second after first token | 10–500+ tok/s | Response completion time |
| **End-to-End Latency** | Total request time to complete response | 100 ms – 30 s | SLA compliance |
| **Queue Time** | Wait time before processing begins | 0–10+ s | System saturation detection |
| **Prefill Time** | Time to process the input prompt | 10 ms – 3 s | TTFT component |
| **Decode Time** | Time to generate output tokens | 50 ms – 25 s | TPS computation |

### 2.2 Tail Latency

Averages hide problems. Always track distributions:
- **p50 (median):** The "typical" experience. If high, the system is fundamentally slow.
- **p95:** 1 in 20 requests experience this. Industry target: p95 < 2× p50.
- **p99:** 1 in 100 requests. Drives SLA violations and user complaints.
- **p999:** 1 in 1000. Indicates GC pauses, cold starts, or network hiccups.

**Rule of thumb:** A system with p50 TTFT of 200 ms and p99 TTFT of 2 s has a tail-latency problem. Fix the tail before optimizing the median further.

### 2.3 Benchmarking Tools

| Tool | Description | Key Features |
|---|---|---|
| **LLMPerf** | Open-source LLM latency benchmarking suite | TTFT, TPS, end-to-end; p50/p95/p99 reports; supports OpenAI API, vLLM, TGI |
| **vLLM benchmark** | Built-in vLLM benchmarks | Dataset-driven; throughput and latency under varying load |
| **TensorRT-LLM benchmark** | NVIDIA's official suite | Multi-GPU, multi-node; FP8/INT4 profiling; per-layer breakdown |
| **MLPerf Inference** | Industry-standard ML benchmark | Standardized LLM workloads; vendor-comparable |
| **GenAI-Perf** | NVIDIA's generative AI perf tool | Throughput, latency, memory; Triton Server reports |
| **Custom load testing** | Locust, k6, Artillery | Realistic user patterns; ramp-up to find saturation |

### 2.4 Recommended Measurement Stack

```
Prometheus ← vLLM/TensorRT-LLM metrics endpoint
    ↓
Grafana (TTFT p50/p95/p99, TPS, queue depth, GPU util)
    ↓
Alerting (p99 TTFT > 2× target for 5 min → page)
    ↓
Cost monitoring (cost per 1M tokens, GPU-hour per request)
```

---

## 3. Model-Level Optimization

The most impactful latency optimizations start at the model level. Changes here compound across all serving infrastructure.

### 3.1 Model Quantization

Quantization reduces weight precision, shrinking memory footprint and increasing throughput.

| Precision | Bits/Weight | Memory vs FP16 | Latency Impact | Quality Impact |
|---|---|---|---|---|
| FP16 (baseline) | 16 | 1× | 1× | Reference |
| FP8 (E5M2/E4M3) | 8 | ~2× | 1.5–2× faster | Negligible on H100 |
| INT8 | 8 | ~2× | 1.5–2× faster | Minor on most tasks |
| INT4 | 4 | ~4× | 2–3× faster | Noticeable on complex reasoning |
| NF4 | 4 | ~4× | 2–3× faster | Better than INT4 |

**GPU quantization methods:**
- **AWQ (Activation-Aware Weight Quantization):** Per-channel scaling based on activation distributions. Best quality for INT4 GPU inference. Natively supported in vLLM and TensorRT-LLM.
- **GPTQ (Post-Training Quantization):** One-shot quantization using approximate second-order optimization. Good INT4/INT3 quality.
- **bitsandbytes:** HuggingFace-integrated 8-bit/4-bit quantize-on-load. Easy but not production-optimized.
- **FP8 (Transformer Engine):** Native on H100/H200. No calibration needed. Best quality preservation.

**CPU quantization:**
- **GGUF (GPT-Generated Unified Format):** Standard for CPU inference. Q2_K through Q8_0 levels. Q4_K_M is the recommended sweet spot. Used by llama.cpp, Ollama, LM Studio.

**Recommendation:** For production GPU serving, start with AWQ INT4 or FP8. For edge/CPU, use GGUF Q4_K_M.

### 3.2 Smaller Models

The simplest latency optimization: use the smallest model meeting quality requirements.

| Model | Parameters | Relative Speed (vs 70B) | Use Case |
|---|---|---|---|
| Phi-3-mini | 3.8B | 20–30× faster | Simple Q&A, classification |
| Llama 3.2 3B | 3B | 15–25× faster | Edge, mobile, real-time |
| Mistral 7B | 7B | 8–12× faster | General-purpose, coding |
| Llama 3 8B | 8B | 6–10× faster | Production chat, moderate reasoning |
| Mixtral 8x7B | 47B active | 2–3× faster | Strong reasoning, code |
| Llama 3 70B | 70B | 1× (baseline) | Complex reasoning |

**Model router pattern:** Deploy multiple sizes; route simple queries to small model, complex ones to large. Serves 70%+ of requests at 8B latency while maintaining 70B quality for hard cases.

### 3.3 Model Distillation

Knowledge distillation trains a smaller "student" to mimic a larger "teacher."

| Model | Size | Teacher | Quality vs Teacher | Speedup |
|---|---|---|---|---|
| Phi-3-mini | 3.8B | GPT-4 class | ~85–90% | 10–20× |
| TinyLlama | 1.1B | Llama 2 7B | ~80% | 6–8× |
| DistilBERT | 67M | BERT-base | ~95% | 2× |

**Practical guidance:** Distilled models excel in high-throughput model-router "fast paths." Re-evaluate quality monthly on production data.

### 3.4 Pruning

Removes less important weights from the model.

| Technique | Method | Sparsity | HW Support |
|---|---|---|---|
| SparseGPT | Hessian-based one-shot pruning | 50–60% | Limited (needs sparse HW) |
| Wanda | Weight × activation product | 50% | Limited |
| Structured pruning | Remove entire neurons/heads | 20–30% | Good (standard GEMM) |
| 4:2 structured sparsity | NVIDIA sparse tensor cores | 50% | Excellent on A100/H100 |

**Guidance:** Unstructured sparsity is hard to accelerate on current GPUs. Structured pruning gives immediate speedup. NVIDIA's 4:2 structured sparsity is the most production-practical.

### 3.5 Architecture Optimization

| Architecture | Quality | Latency | KV Cache Size | Used In |
|---|---|---|---|---|
| Multi-Head Attention (MHA) | Best | 1× baseline | Most | Original Transformer |
| Multi-Query Attention (MQA) | Slightly worse | 1.2–1.5× faster | ~1/8 | Falcon, PaLM |
| Grouped-Query Attention (GQA) | Near MHA | 1.1–1.3× faster | ~1/4–1/2 | Llama 2/3, Mistral |
| Sliding Window Attention (SWA) | Good local | 1.3–1.8× faster | O(window) | Mistral, Gemma |

- **GQA:** Llama 2 uses 8 KV heads with 32 query heads (4:1 ratio). Llama 3 uses 8 KV heads. 4× KV cache reduction vs MHA while maintaining quality. De facto standard for 2024+ models.
- **FlashAttention-2:** Fused attention kernel. 2× speedup over PyTorch attention on A100.
- **FlashAttention-3:** H100-native with asynchronous WGMMA + FP8 Tensor Cores. 75% H100 FLOP utilization (up from 35% for FA2). 1.5–2× faster than FA2 on H100.
- **MLA (Multi-head Latent Attention):** DeepSeek's approach compressing KV cache into latent space. Dramatically reduces memory for larger batch sizes.

---

## 4. Serving Infrastructure Optimization

The serving layer translates model-level optimizations into production latency.

### 4.1 KV Cache Optimization

The KV cache stores attention keys/values during generation. Its management is the single largest serving-level optimization.

| Technique | Description | Latency Benefit | Memory Benefit |
|---|---|---|---|
| **PagedAttention (vLLM)** | Non-contiguous KV cache allocation like OS pages | Eliminates fragmentation, enables larger batches | 3–5× effective memory |
| **Continuous batching** | Dynamically add/remove sequences from batch | Higher GPU utilization, lower queue time | More efficient memory |
| **Prefix caching** | Reuse KV cache for shared prefixes | 30–60% TTFT reduction | Significant on cache hit |
| **KV cache quantization** | Store KV cache in INT8/FP8 | Minimal overhead | 2× memory reduction |
| **KV cache eviction** | LRU, LFU, H2O policies | Maintains hit rate | Bounded cache size |

**Prefix caching deep dive:** vLLM's Automatic Prefix Caching (APC) hashes KV blocks and reuses them across requests with identical prefixes. Especially powerful for system prompts shared across requests, multi-turn conversations, and RAG pipelines with repeated retrieval prefixes. Enable with `--enable-prefix-caching`. Typical TTFT reduction: 30–60% for workloads with shared system prompts.

### 4.2 GPU Memory Management

| Strategy | Implementation | How It Works | Latency Impact | Throughput Impact |
|---|---|---|---|---|
| Dynamic batching | TGI, Triton | Accumulate N ms, batch together | Adds fixed delay | 2–5× throughput |
| Continuous batching | vLLM, TRT-LLM, TGI | Sequences enter/leave dynamically | Minimal overhead | 3–10× vs static |
| Inflight batching | TensorRT-LLM | Interleave prefill and decode | Better GPU utilization | Higher than continuous |
| Split-fuse (SARATHI) | vLLM (experimental) | Chunk long prefills, interleave with decode | Reduces tail TTFT | More uniform latency |
| Disaggregated prefill/decode | Separate services | Prefill on dedicated GPU, decode on another | Better TTFT | Independent scaling |

**Continuous batching explained:** Traditional static batching waits for all sequences to finish. Continuous batching adds/removes sequences mid-batch — finished sequences release memory immediately, new sequences join without waiting. Dramatically reduces "empty slots."

**Inflight batching (TensorRT-LLM):** Interleaves prefill and decode within the same batch iteration. Keeps GPU units busy during the memory-bandwidth-bound decode phase.

**Split-fuse / SARATHI:** Long prefills (e.g., RAG contexts) block short requests. Split-fuse decomposes long prefills into chunks and interleaves with decode. Short requests see dramatically lower TTFT even under long-context workloads.

### 4.3 Speculative Decoding

The most impactful single technique for decode-phase latency. Uses a fast "draft" model to generate candidate tokens, then verifies in parallel with the target model.

```
Standard:      T₁ → T₂ → T₃ → T₄ → T₅ (sequential, 5 steps)
Speculative:   Draft: [D₁,D₂,D₃,D₄] → Verify: [V₁,V₂,✓,✓] → accept 2 tokens (3 steps)
               One parallel verification of 4 candidates replaces 2 decode steps
```

| Technique | Draft Source | Speedup | Training | Key Feature |
|---|---|---|---|---|
| Standard Spec. Decoding | Separate small model | 2–3× | No | Works with any model |
| Medusa | Multiple draft heads | 2.2–3.6× | Yes (1–2 days) | No separate model needed |
| EAGLE (1/2/3) | Lightweight prediction module | 2–4× (EAGLE-3: 3.2×) | Yes | Fastest verified; exact output distribution |
| Lookahead Decoding | n-gram candidates from past output | 1.5–2× | No (training-free) | Simple, no extra model |
| Self-Speculative | Early exit layers | 1.5–2.5× | Yes | Single model, no draft |
| Sequoia | Tree-structured speculation | 2–4× | Yes | Optimized for long generations |

**Production status (2025/2026):** vLLM integrates EAGLE-3 and Medusa. SGLang supports SpecForge (EAGLE-3). TensorRT-LLM supports Medusa. Typical production speedup: 2.0–3.2× throughput with bit-exact outputs.

**Best for:** Long output sequences (100+ tokens), where draft-to-target speed ratio is high (>10×), and GPU memory can accommodate both models.

### 4.4 Streaming

Return tokens as generated rather than waiting for the full response.

| Approach | Perceived Latency | User Experience |
|---|---|---|
| No streaming | = end-to-end | User waits in silence |
| Token-by-token | ~TTFT + 1 inter-token delay | First chars appear almost instantly |
| Chunked (every N tokens) | ~TTFT + N × inter-token delay | Good balance for UI rendering |

**Implementation:** Most engines support SSE (Server-Sent Events). Enable `stream=True`. Streaming adds ~1–5 ms overhead per chunk. Does not reduce total generation time, but dramatically improves **perceived latency**.

### 4.5 Batching vs Latency Trade-off

| Batch Size | Throughput | p50 Latency | p99 Latency | GPU Utilization |
|---|---|---|---|---|
| 1 | 1× | 1× (baseline) | 1× (baseline) | ~5–15% |
| 4 | 3–4× | 1.1–1.3× | 1.5–2× | ~25–40% |
| 8 | 5–8× | 1.3–1.8× | 2–3× | ~40–60% |
| 16 | 8–14× | 1.8–3× | 4–6× | ~55–75% |
| 32 | 12–20× | 3–5× | 8–12× | ~65–85% |

**Practical guidance:**
- Real-time (<100 ms): Batch size 1, dedicated GPU. Accept high cost.
- Chat (<2 s): Batch size 4–16, continuous batching. Balance cost and latency.
- Batch processing: Batch size 32–128. Maximize throughput.

### 4.6 Request Batching Strategies

| Strategy | Implementation | Latency Impact | Throughput | Complexity |
|---|---|---|---|---|
| Static batching | Wait for N, process together | Adds fixed delay | Moderate | Low |
| Dynamic batching | max_batch or max_delay_ms | Bounded delay | Good | Medium |
| Continuous batching | Sequences enter/leave dynamically | Minimal per-seq delay | Excellent | High |
| Inflight batching | Interleave prefill/decode | Best for mixed workloads | Best | Very high |

---

## 5. Inference Engine Comparison

The inference engine manages model loading, tokenization, batching, and GPU execution. Choosing the right engine is one of the highest-leverage decisions.

### 5.1 Feature Comparison

| Feature | vLLM | TensorRT-LLM | TGI | Ollama | llama.cpp | DeepSpeed-MII | CTranslate2 | MLC-LLM |
|---|---|---|---|---|---|---|---|---|
| PagedAttention | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Continuous batching | ✅ | ✅ (inflight) | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| Prefix caching | ✅ (auto) | ✅ (KV reuse) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Speculative decoding | ✅ (EAGLE, M.) | ✅ (Medusa) | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| FP8 quantization | ✅ (H100) | ✅ (native) | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |
| INT4 AWQ/GPTQ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| GGUF support | ❌ | ❌ | ❌ | ✅ (core) | ✅ (core) | ❌ | ❌ | ❌ |
| Tensor parallelism | ✅ | ✅ (multi-node) | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| OpenAI API compat | ✅ | ❌ (needs Triton) | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| HuggingFace integration | ✅ (direct) | ❌ (convert) | ✅ (native) | ❌ | ❌ | ✅ | ❌ | ❌ |
| CPU inference | ❌ | ❌ | ❌ | ✅ | ✅ (primary) | ❌ | ✅ | ✅ |
| Apple Silicon | ❌ (CUDA) | ❌ | ❌ | ✅ | ✅ (Metal) | ❌ | ❌ | ✅ (Metal) |
| Mobile/WebGPU | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ (Vulkan/WGPU) |

### 5.2 Engine Profiles

**vLLM — The Industry Standard**
- **Best for:** Cloud GPU, moderate-to-high throughput, rapid iteration.
- **Strengths:** PagedAttention eliminates KV cache fragmentation; automatic prefix caching; broadest model support; OpenAI-compatible API; largest community.
- **Weaknesses:** CUDA only; no CPU fallback; no multi-node pipeline parallelism.
- **Latency profile:** Excellent TTFT via PagedAttention + prefix caching; TPS competitive but not absolute fastest.

**TensorRT-LLM — Maximum Performance on NVIDIA**
- **Best for:** Dedicated NVIDIA GPU clusters, latency-critical production, on-prem.
- **Strengths:** Inflight batching; native FP8/INT4/INT8; multi-node multi-GPU; C++ runtime; best absolute TPS on H100/H200.
- **Weaknesses:** Requires model conversion; NVIDIA-only; steeper learning curve.
- **Latency profile:** Best raw TPS; inflight batching keeps tail latency low under mixed loads.

**TGI — HuggingFace Ecosystem**
- **Best for:** Teams in HuggingFace ecosystem, moderate throughput.
- **Strengths:** Native HF integration; message passing for multi-node; watermarking, safety filters.
- **Weaknesses:** No PagedAttention; no prefix caching; lower throughput than vLLM/TRT-LLM.
- **Latency profile:** Good single-request latency; throughput degrades under high concurrency.

**Ollama — Developer/Edge Simplicity**
- **Best for:** Local dev, edge devices, prototyping.
- **Strengths:** One-command setup; GGUF support; simple API; multi-platform.
- **Weaknesses:** No continuous batching; limited production throughput.
- **Latency profile:** Acceptable for single-user local; unsuitable for production scaling.

**llama.cpp — CPU-First, Universal**
- **Best for:** CPU inference, edge/embedded, Apple Silicon.
- **Strengths:** GGUF with extensive quantization; Metal optimization; works on almost any hardware; single-binary deployment.
- **Weaknesses:** Limited GPU support; no distributed inference; basic HTTP server.
- **Latency profile:** Best CPU TPS (5–40 tok/s depending on model/CPU).

**MLC-LLM — Universal Deployment**
- **Best for:** Edge deployment across platforms (CUDA, Metal, Vulkan, WebGPU).
- **Strengths:** TVM-compiled for any GPU backend; WebGPU browser inference; mobile Android/iOS.
- **Weaknesses:** Complex compilation pipeline; smaller ecosystem.
- **Latency profile:** Competitive per-platform; excellent for cross-platform edge.

### 5.3 Engine Decision Tree

```
CPU/edge inference?
├── Apple Silicon? → llama.cpp or MLC-LLM
├── CPU-only? → llama.cpp (GGUF)
├── Browser/mobile? → MLC-LLM (WebGPU/Vulkan)
└── No → Continue

NVIDIA GPUs?
├── Need max TPS? → TensorRT-LLM
├── Need rapid iteration/API compat? → vLLM
├── Heavy HF ecosystem? → TGI or vLLM
└── Not sure? → Start with vLLM

Local dev simplicity? → Ollama
```

---

## 6. Serving Architecture Patterns

### 6.1 Single-Model Endpoint
```
Client → LB → [One Model on One GPU/Instance]
```
Simplest; predictable latency; easy monitoring. Best for dedicated use cases.

### 6.2 Multi-Model Serving
```
Client → LB → [Model A, B, C on shared GPU pool]
```
Higher utilization across models; shared cost. Risk: resource contention raises p99 latency.

**Isolation approaches:** MIG (hardware isolation on A100/H100), time-slicing, separate vLLM instances.

### 6.3 Model Router
```
Client → Router → [Small Model (8B) — 70% of requests]
                → [Large Model (70B) — 30% of requests]
```
Route requests based on complexity, intent, or user tier. Routing latency: 5–20 ms.

**Pros:** 70%+ of requests at small-model latency; cost-efficient. **Cons:** Routing errors send complex queries to small model. **Implementation:** LiteLLM, FastAPI proxy, Envoy custom filters.

### 6.4 Pre-Warmed Endpoints

Cold start for 70B model: 30–120 seconds. Keep models loaded with:
- Dummy health-check prompts every 30–60 s.
- Serverless providers (Modal, Banana) offer automatic keep-warm.
- K8s liveness probes prevent scaling-to-zero for latency-critical paths.

### 6.5 GPU Auto-Scaling

- **Metrics:** Queue depth, p95 TTFT, GPU utilization.
- **Scale up:** When p95 TTFT > 2× target. **Scale down:** When GPU utilization < 30% for 5+ minutes.
- **K8s:** Karpenter or Cluster Autoscaler + GPU node pools.

### 6.6 Edge Deployment

Run 1B–8B models on edge devices. Zero network latency; offline; private.

| Framework | Hardware | Latency (3B model) |
|---|---|---|
| llama.cpp | Apple M3 | 30–60 ms TTFT, 65–80 tok/s |
| MLC-LLM | Snapdragon 8 Gen 3 | 80–150 ms TTFT, 25–40 tok/s |
| llama.cpp | NVIDIA Jetson Orin | 60–120 ms TTFT, 30–50 tok/s |

### 6.7 Hybrid Edge-Cloud

```
Client → Edge Model (3B) [fast: return] OR → Cloud Model (70B) [complex: return]
```
90% queries at <50 ms (edge); 10% at 1–5 s (cloud). Average perceived latency: ~100–200 ms.

---

## 7. Prompt Engineering for Latency

### 7.1 Shorter Prompts

Prefill time scales linearly with input token count. Halving input tokens roughly halves TTFT.

| Input Tokens | Approx TTFT (A100, 8B) | Notes |
|---|---|---|
| 512 | 60–100 ms | Minimal context |
| 2048 | 200–350 ms | Typical RAG |
| 4096 | 400–700 ms | Long context |
| 8192 | 800–1400 ms | Very long |

**Technique:** Prune unnecessary context, remove redundant instructions, limit conversation turns.

### 7.2 Prompt Caching

Pre-compute KV cache for shared system prompts. vLLM APC handles this transparently. **Impact:** 30–60% TTFT reduction for workloads with shared system prompts. Provider-level caching: OpenAI, Anthropic, Google Cloud offer as billable feature.

### 7.3 System Prompt Compression

A 2000-token system prompt can often be reduced to 200 tokens without quality loss:
- Remove redundant explanations.
- Use shorter action verbs ("Summarize:" vs "Please provide a summary of...").
- Move instructions to compact format (JSON schema vs natural language).

### 7.4 Context Window Management

- Limit conversation history to last N turns (e.g., 10).
- Use summarized history ("Previous summary: ...") instead of full raw text.
- Reduce context from 32K to 4K → 5–8× faster prefill.

### 7.5 Early Stopping

Stop generation as soon as the answer is sufficient. Use `stop` tokens, confidence-based exiting, or task-aware stopping. **Impact:** 30–50% average output length reduction.

### 7.6 Constrained Generation

| Technique | Description | Latency Impact |
|---|---|---|
| Grammar constraints | Force SQL, JSON adherence | Reduces invalid tokens |
| JSON schema (Outlines, LMQL) | Match structured output schema | Prevents long invalid generations |
| Logit bias | Boost/penalize specific tokens | <5% overhead |

Use JSON mode for programmatic consumption. Libraries: Outlines, instructor, LMQL, vLLM guided decoding.

### 7.7 Output Length Control

- Set `max_tokens` to the minimum acceptable value.
- Use `min_tokens` + `max_tokens` together.
- Halving output (1024 → 512) approximately halves decode latency.

---

## 8. Network and Infrastructure Optimization

### 8.1 Model Placement

- Co-locate model with application. Every network hop adds latency.
- **Cross-region:** USA→Europe ~100–150 ms; USA→Asia ~150–250 ms.
- **Same-region:** <1 ms within AZ, 5–10 ms cross-AZ.
- **For Crédit Agricole CIB (APAC):** Deploy GPUs in ap-southeast-1 (Singapore) rather than eu-west-1. Saves 150–250 ms round-trip.

### 8.2 GPU Instance Selection

For LLM inference, **memory bandwidth** matters more than raw compute.

| GPU | Memory BW | VRAM | Relative TPS (8B) | Relative TPS (70B) |
|---|---|---|---|---|
| H200 SXM | 4.8 TB/s | 141 GB | 1.2–1.4× | 1.3–1.5× |
| H100 SXM | 3.35 TB/s | 80 GB | 1.0× (baseline) | 1.0× (baseline) |
| A100 80GB | 2.0 TB/s | 80 GB | 0.5–0.7× | 0.4–0.6× |
| L40S | 864 GB/s | 48 GB | 0.25–0.35× | Not viable |
| L4 | 300 GB/s | 24 GB | 0.10–0.15× | Not viable |

**Key insight:** TPS is directly proportional to memory bandwidth for decode-heavy workloads. H100's 3.35 TB/s vs A100's 2.0 TB/s → ~1.7× TPS for decode.

### 8.3 Multi-GPU Parallelism

| Technique | Description | Latency Impact | Use Case |
|---|---|---|---|
| Tensor Parallelism (TP) | Split layers across GPUs | Reduces per-token latency | Model too large for one GPU |
| Pipeline Parallelism (PP) | Layers in pipeline stages | Slightly higher per-request | Batch processing |
| Data Parallelism (DP) | Full model per GPU, diff requests | Same per-request | High throughput |
| Expert Parallelism (EP) | Distribute MoE experts | Variable | Mixtral, Llama 4 |

**TP guidance:** Required when model doesn't fit one GPU. 8-way TP adds 10–30% communication overhead. NVLink (600 GB/s) is critical for efficiency. For models < 8B, data parallelism gives better latency.

### 8.4 RDMA Networking

- Multi-node inference (405B, etc.): InfiniBand HDR/NDR or RoCE.
- RDMA: ~5 µs inter-node latency vs ~50 µs for TCP/IP.
- Without RDMA, cross-node TP overhead can exceed benefits.

### 8.5 Quantization-Aware Network

INT8/FP8 KV cache reduces data transfer between GPUs during tensor parallel attention. 2× less data movement = proportionally lower communication latency. TensorRT-LLM supports FP8 KV cache natively; vLLM supports on H100.

---

## 9. Caching Strategies

Caching avoids inference entirely — the ultimate latency reduction.

### 9.1 Semantic Caching

```
Request → Embedding → Similarity Search → [Hit: return cached response]
                                            [Miss: infer → cache]
```
- **Hit latency:** 5–20 ms (embedding + lookup) vs 500–5000 ms for inference.
- **Tech:** Redis + pgvector, Chroma, Milvus, GPTCache.
- **Best for:** FAQ, customer service (same questions repeated), code completion patterns.
- **Caveat:** Lossy — cached response may not perfectly match query intent.

### 9.2 Response Caching

- Cache exact request-response pairs. Hit latency: <1 ms.
- **Tech:** Redis, Memcached, CDN.
- **Cache key:** (model + prompt + temperature + max_tokens + stop_sequences).
- **Invalidation:** On model update.

### 9.3 KV Cache Caching

- Pre-compute KV cache for system prompts, fixed instruction prefixes, common RAG prefixes.
- **Storage tiers:** GPU memory (fastest), CPU RAM (larger), NVMe (cold).
- **Impact:** 30–60% TTFT reduction for cached prefixes.

### 9.4 Longest Prefix Matching (Multi-Turn)

- Cache KV blocks for each conversation prefix.
- Turn N only computes KV for turn N's tokens — not all N turns.
- vLLM APC: hash-based block matching. SGLang RadixAttention: tree-structured prefix matching.

### 9.5 Cache Strategy Summary

| Cache Type | Hit Latency | Storage | Risk |
|---|---|---|---|
| Response cache | <1 ms | Redis, RAM | Stale responses |
| Semantic cache | 5–20 ms | Vector DB | Semantic drift |
| KV cache (GPU) | Skips prefill | GPU VRAM | Memory pressure |
| KV cache (CPU) | Skips prefill + PCIe | CPU RAM | PCIe bandwidth |
| Prefix cache | Skips shared prefix | GPU VRAM | Miss for unique prefixes |

---

## 10. Latency vs Throughput Trade-off

### 10.1 Optimizing for Latency

| Configuration | Setting | Rationale |
|---|---|---|
| Batch size | 1 | Full GPU attention per request |
| GPU | Over-provisioned (H100 for 8B) | Memory bandwidth headroom |
| Model | Smallest adequate (8B) | Less compute per token |
| Quantization | INT4/FP8 | Less data per weight read |
| Speculative decoding | Enabled | Fewer sequential decode steps |
| Hardware | Dedicated, no sharing | No resource contention |

**Cost:** Highest per-request. GPU utilization 5–20%. 2–5× cost premium.

### 10.2 Optimizing for Throughput

| Configuration | Setting | Rationale |
|---|---|---|
| Batch size | Max (memory-limited) | Max GPU utilization |
| GPU | Right-sized (L40S for 8B) | Lower cost, higher utilization |
| Quantization | FP16/INT8 (preserve quality) | Enables larger batches |
| Continuous batching | Maximized | Keep GPU busy |

**Cost:** Lowest per-token. 70–90% GPU utilization. 2–5× better tokens-per-dollar.

### 10.3 Finding the Sweet Spot

| Requirement | Strategy | Expected Latency | Relative Cost |
|---|---|---|---|
| Real-time (<100 ms) | Over-provision GPU, batch=1, speculative, edge | 50–100 ms | 5–10× |
| Interactive chat (<2 s) | Batch=4–16, continuous batching, KV cache, streaming | 500–2000 ms | 1.5–3× |
| Response time (<5 s) | Batch=16–64, model router, aggressive caching | 2–5 s | 1–1.5× |
| Batch processing | Max batch, pipeline parallelism, FP16 | 10–120 s | 1× (most efficient) |

### 10.4 Defining Latency SLOs

| Component | Example SLO | Alert Threshold |
|---|---|---|
| TTFT | p95 < 300 ms for chat | >600 ms for 5 min |
| TPS | Average > 50 tok/s | <25 tok/s for 5 min |
| End-to-end | p99 < 5 s for complex tasks | >10 s for 1 min |
| Queue time | p95 < 50 ms | >200 ms for 5 min |

**Error budget:** For 99.9% availability with p99 TTFT < 500 ms → ~43 seconds of violations per month.

---

## 11. Measurement and Monitoring

### 11.1 Key Metrics

| Category | Metric | Alert On |
|---|---|---|
| Latency | TTFT p50/p95/p99 | p95 > 2× target for 5 min |
| Latency | TPS p50/p95/p99 | TPS < 50% target |
| Latency | End-to-end latency | p99 > SLO |
| Throughput | Requests per second | Sudden drop |
| Throughput | Prefix cache hit rate | <30% |
| Cost | Cost per 1M tokens | >2× baseline |
| Reliability | Request failure rate | >1% |
| Reliability | OOM errors | >0 (investigate immediately) |

### 11.2 Recommended Stack

```
App → Inference Engine → Prometheus → Grafana Dashboards
                                         ├── Latency (TTFT, TPS, queue, E2E)
                                         ├── GPU (util, memory, power, temp)
                                         ├── Throughput (RPS, tok/s, cache hit)
                                         ├── Cost (cost/1M tok, cost/req)
                                         └── SLO (error budget burn rate)
                                      → Alertmanager (page on SLO violation)
```

### 11.3 Cost Per Query Tracking

```
cost_per_query = (gpu_hourly_rate / (req_per_hour × batch_size)) +
                 (input_tokens × input_cost) + (output_tokens × output_cost)
```

**GPU rates (2025 cloud, approximate):** H100: $3–5/hr, A100: $1.5–3/hr, L40S: $1–2/hr, L4: $0.5–1/hr. **Target:** < $0.001 per chat query.

---

## 12. LLM Latency by Model Size

Approximate benchmarks for common model-GPU combinations. Actual numbers vary with batch size, quantization, prompt length, and engine config.

### 12.1 Latency Benchmarks (Llama 3)

| Model | GPU | Quant | BS | TTFT (p50) | TPS (p50) | E2E (256 tok) |
|---|---|---|---|---|---|---|
| **8B** | L4 | FP16 | 1 | 250–400 ms | 25–35 | 7.5–10 s |
| **8B** | L4 | INT4 | 1 | 80–150 ms | 50–70 | 3.5–5 s |
| **8B** | A100 | FP16 | 1 | 60–120 ms | 80–120 | 2–3.5 s |
| **8B** | A100 | INT4 | 1 | 25–50 ms | 150–220 | 1–1.5 s |
| **8B** | H100 | FP8 | 1 | 20–40 ms | 200–300 | 0.8–1.3 s |
| **8B** | H100 | INT4 | 1 | 15–30 ms | 300–450 | 0.5–0.9 s |
| **8B** | A100 | INT4 | 8 | 150–300 ms | 400–600* | 0.6–1.2 s* |
| **8B** | H100 | FP8 | 8 | 60–120 ms | 800–1200* | 0.3–0.5 s* |
| **70B** | 2× A100 | INT4 | 1 | 200–400 ms | 25–40 | 6–10 s |
| **70B** | 2× A100 | FP16 | 1 | 400–700 ms | 12–20 | 12–20 s |
| **70B** | H100 (1×) | FP8 | 1 | 100–200 ms | 45–70 | 3.5–5 s |
| **70B** | H100 (1×) | INT4 | 1 | 60–150 ms | 80–120 | 2–3.5 s |
| **70B** | 2× H100 | FP8 | 1 | 60–120 ms | 80–130 | 2–4 s |
| **405B** | 8× H100 | FP8 | 1 | 500–1200 ms | 15–30 | 8–18 s |
| **405B** | 8× H100 | INT4 | 1 | 300–800 ms | 30–50 | 5–10 s |
| **405B** | 16× H100 | FP8 | 4 | 600–1500 ms | 100–180* | 2–4 s* |

*\*Batch-level throughput; individual request sees higher variability.*

### 12.2 Edge Device Benchmarks

| Model | Device | Quant | TPS | TTFT |
|---|---|---|---|---|
| Phi-3-mini 3.8B | Apple M3 (16 GB) | GGUF Q4_K_M | 40–60 | 50–100 ms |
| Llama 3.2 3B | Apple M3 (16 GB) | GGUF Q4_K_M | 65–80 | 30–60 ms |
| Gemma 2 2B | Apple M3 (16 GB) | GGUF Q4_K_M | 80–120 | 20–40 ms |
| Llama 3.2 3B | Snapdragon 8 Gen 3 | QNN | 25–40 | 80–150 ms |
| Phi-3-mini 3.8B | NVIDIA Jetson Orin | FP16 | 30–50 | 60–120 ms |

### 12.3 Key Takeaways

1. **Model size dominates latency.** 8B on a modest GPU (L4) can match 70B on H100 in raw TPS.
2. **Quantization is the highest-leverage optimization.** INT4 AWQ gives 3–5× TPS improvement over FP16.
3. **H100 vs A100 is ~2× for decode** due to memory bandwidth difference (3.35 vs 2.0 TB/s).
4. **Small models on GPU are extremely fast.** Llama 3 8B on H100 at INT4: >400 tok/s = sub-second 256-token responses.
5. **Edge models are viable.** Phi-3-mini on Apple Silicon at 40–60 tok/s is real-time capable.

---

## 13. Cost-Latency Optimization Framework

### 13.1 The Optimization Pyramid

```
                    ┌─────────────────────────────┐
                    │ 1. Define Latency SLOs       │ ← Start here
                    ├─────────────────────────────┤
                    │ 2. Benchmark HW + Models     │ ← Test on actual hardware
                    ├─────────────────────────────┤
                    │ 3. Select Smallest Model     │ ← Meeting quality bar
                    ├─────────────────────────────┤
                    │ 4. Quantize (FP8/INT4/GGUF)  │ ← 2–4× improvement
                    ├─────────────────────────────┤
                    │ 5. Choose Engine (vLLM/TRT)  │ ← 2–5× improvement
                    ├─────────────────────────────┤
                    │ 6. KV Cache + Prefix Caching │ ← 30–60% TTFT reduction
                    ├─────────────────────────────┤
                    │ 7. Speculative Decoding      │ ← 2–3× TPS for long gens
                    ├─────────────────────────────┤
                    │ 8. Cache (Response/Semantic) │ ← Eliminate inference entirely
                    ├─────────────────────────────┤
                    │ 9. Right-Size GPU            │ ← Memory bandwidth is key
                    └─────────────────────────────┘
                        10. Model Router (optional)
```

### 13.2 Step-by-Step Framework

**Step 1 — Define Latency SLOs:**
```
Example: TTFT p50 < 300 ms, p95 < 800 ms; TPS > 50; E2E (256 tok) p95 < 5 s
```

**Step 2 — Benchmark models on target hardware.** Use LLMPerf with representative prompts. Measure TTFT, TPS, E2E at varying batch sizes.

**Step 3 — Select smallest model meeting quality.** A fine-tuned 8B often outperforms a generic 70B on specific tasks. Model router: 8B for 70% of queries, 70B for 30%.

**Step 4 — Apply quantization:**
| Hardware | Recommended | Rationale |
|---|---|---|
| H100/H200 | FP8 | Native, best quality, no calibration |
| A100/A10G/L4 | INT4 AWQ | Best quality-for-speed |
| CPU/Apple Silicon | GGUF Q4_K_M | Best CPU speed-quality |

**Step 5 — Use optimized inference engine:**
| Deployment | Engine |
|---|---|
| Cloud GPU | vLLM |
| On-prem NVIDIA | TensorRT-LLM |
| Edge/CPU | llama.cpp or Ollama |

**Step 6 — KV cache optimization:** Enable PagedAttention (vLLM default). Enable prefix caching (`--enable-prefix-caching`). Consider FP8 KV cache on H100.

**Step 7 — Speculative decoding:** EAGLE-3 or Medusa for latency-critical paths. Lookahead Decoding for training-free setup.

**Step 8 — Cache aggressively:** Response cache (Redis) for exact matches. Semantic cache (vector DB) for similar queries. KV cache for shared prefixes.

**Step 9 — Choose right GPU:** H100 > A100 > L40S > A10G. Memory bandwidth determines decode TPS. Don't over-provision (waste) or under-provision (slow).

**Step 10 — Model router (optional):** Deploy 8B + 70B. Classify requests. 40–70% cost savings vs all-large.

### 13.3 Cost Projection (Approximate)

| Config | GPUs | Max Throughput | Cost/1M tok | E2E Latency (256 tok) |
|---|---|---|---|---|
| 8B, L4, FP16 | 10 | 500 req/s | $0.80 | 9 s |
| 8B, L4, INT4 | 10 | 1200 req/s | $0.35 | 4 s |
| 8B, A100, INT4 | 5 | 2000 req/s | $0.25 | 1.2 s |
| 8B, H100, FP8 | 3 | 2500 req/s | $0.30 | 0.9 s |
| 70B, 2× A100, INT4 | 10 | 800 req/s | $2.50 | 4 s |
| 70B, 2× H100, FP8 | 5 | 1000 req/s | $2.00 | 3 s |

---

## 14. Tools and Techniques Comparison

### 14.1 Comprehensive Matrix

| Technique | Latency Reduction | Throughput Impact | Complexity | Cost Impact | Best For |
|---|---|---|---|---|---|
| Quantization (INT4) | 2–4× TTFT & TPS | 3–5× | Low | Low | All production |
| Quantization (FP8) | 1.5–2× | 2–3× | Low | Low | H100/H200 |
| Smaller model | 5–20× | 5–20× | Low | Low | Model router, simple tasks |
| Model distillation | 3–10× | 3–10× | Medium | Medium (training) | High-volume serving |
| Structured pruning | 1.2–1.5× | 1.3–1.8× | Medium | Low | Memory constrained |
| FlashAttention-2/3 | 1.3–2× (attention) | 1.3–2× | Low (built-in) | None | All transformer models |
| PagedAttention | 1.2–2× | 3–5× | Low (vLLM) | Low | High-throughput serving |
| Continuous batching | 1.1–1.3× (queue) | 3–10× | Medium | Low | All production serving |
| Prefix caching | 1.3–1.6× TTFT | 1.5–3× | Low (flag) | Low | Shared system prompts |
| Speculative decoding | 2–3× TPS | 2–3× | Medium | Low–Medium | Long generations |
| Streaming | Perceived 2–10× | None | Low | None | All user-facing apps |
| Semantic caching | 50–100× (hit) | 2–10× effective | Medium | Low (on hit) | FAQ, customer service |
| Response caching | 100–1000× (hit) | 10–100× effective | Low | Very low | Identical requests |
| Model router | 5–10× avg | 3–5× cost-eff. | Medium | Medium | Variable complexity |
| Disaggregated prefill/decode | 1.5–3× TTFT | 1.5–2× | High | Medium | Mixed long/short |
| Prompt compression | 1.3–3× TTFT | 1.3–3× | Low | None | Long-context systems |
| Early stopping | 1.5–2× E2E | 1.5–2× | Low | None | All generations |

### 14.2 Highest-Leverage Techniques (Ranked)

1. **Model quantization:** 2–4× improvement, low effort. Do this first.
2. **Smallest adequate model:** 5–20× improvement, zero cost. Model selection is everything.
3. **Right inference engine (vLLM/TRT-LLM):** 2–5× improvement over naive deployment.
4. **Prefix caching:** 30–60% TTFT reduction with a single flag. Free speed.
5. **Streaming:** Perceived latency improvement. Free UX win.
6. **Speculative decoding:** 2–3× TPS for long generations. Worth integration effort.
7. **Continuous batching:** 3–10× GPU throughput. Essential for production.
8. **Model router:** 40–70% cost reduction; 5–10× avg latency improvement.
9. **Semantic caching:** Eliminates inference for common queries.
10. **Right-size GPU:** Don't over-provision or under-provision.

---

## 15. Decision Framework

### 15.1 By Latency Requirement

| Requirement | Strategy | Expected Latency | Engine | Hardware |
|---|---|---|---|---|
| **< 100 ms** (Real-time) | Spec decode + quant + small model + edge | 50–100 ms | TRT-LLM or llama.cpp | H100 or Apple Silicon |
| **< 500 ms** (Autocomplete) | vLLM/TRT-LLM + KV cache + streaming + FP8/INT4 | 100–500 ms | vLLM or TRT-LLM | H100 or A100 |
| **< 2 s** (Chat) | Continuous batching + prefix caching + streaming + router | 500 ms – 2 s | vLLM (recommended) | A100 or L40S |
| **< 5 s** (Complex tasks) | Model router + caching + right-sized GPU | 2–5 s | vLLM or TGI | A100 or L40S |
| **Batch processing** | Max throughput + large batches + FP16 | 10–120 s | vLLM or TRT-LLM | Any GPU, max batch |

### 15.2 By Deployment Environment

| Environment | Primary Engine | Quantization | Key Optimization |
|---|---|---|---|
| On-prem NVIDIA GPU | TensorRT-LLM | FP8 (H100) / INT4 (A100) | Inflight batching, multi-node TP |
| Cloud GPU | vLLM | INT4 AWQ | PagedAttention, prefix caching, auto-scaling |
| Edge device | llama.cpp | GGUF Q4_K_M | Zero network latency, offline |
| CPU-only | llama.cpp | GGUF Q4_K_M | Aggressive quantization |
| Mobile | MLC-LLM | INT4 | On-device NPU/GPU, 1–3B models |
| Web browser | MLC-LLM (WebGPU) | INT4 | WebGPU, < 3B model |
| Apple Silicon | llama.cpp (Metal) | GGUF Q4_K_M | Metal GPU acceleration |

### 15.3 By Budget

| Budget | Primary Strategy | Cost/1M tok | Latency |
|---|---|---|---|
| **Low** | Model selection + prompt compression + response caching | $0.05–0.20 | 2–10 s |
| **Moderate** | Quantization (INT4) + vLLM + prefix caching + streaming | $0.20–0.80 | 500 ms – 3 s |
| **High** | Spec decode + H100 GPUs + TRT-LLM + model router | $0.80–3.00 | 100 ms – 1.5 s |

### 15.4 Quick-Start Checklist

- [ ] **Define SLOs** (p50/p95 TTFT, TPS, E2E) — document them.
- [ ] **Select smallest model** meeting quality on your task.
- [ ] **Quantize to INT4** (AWQ for GPU, GGUF Q4_K_M for CPU).
- [ ] **Choose inference engine** — vLLM unless NVIDIA-only infra (TensorRT-LLM).
- [ ] **Enable continuous batching** in the engine.
- [ ] **Enable prefix caching** (vLLM `--enable-prefix-caching`).
- [ ] **Enable streaming** (`stream=True` in API).
- [ ] **Set up monitoring** — Prometheus + Grafana for TTFT, TPS, queue depth.
- [ ] **Load test** — find saturation point with representative prompts.
- [ ] **Add response cache** (Redis) for redundant requests.
- [ ] **Add semantic cache** (vector DB) for similar queries.
- [ ] **Implement model router** if quality needs vary.
- [ ] **Add speculative decoding** for long-generation paths (>100 tokens).
- [ ] **Review GPU utilization** — if < 50%, reduce GPU count or increase batch.
- [ ] **Document cost per query** and track trends.

---

> **Author:** Jack Liu Shurui (Solution Architect, Crédit Agricole CIB)
> **Repository:** https://github.com/jackliusr/research
> **Classification:** Technology / LLM Deployment Optimization
> **Version:** 1.0 — July 2025
