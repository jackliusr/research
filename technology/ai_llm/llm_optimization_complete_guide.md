# LLM Optimization: The Complete Guide

> **An umbrella reference covering the end-to-end practice of making Large Language Models smaller, faster, cheaper, and more efficient for production deployment. This guide ties together the detailed deep-dives in this repository and provides the overarching framework for understanding, planning, and executing LLM optimization.**
>
> **Author:** Jack Liu Shurui | **Date:** July 2026 | **Context:** Solution Architecture — Crédit Agricole CIB, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research) | **Series:** LLM Optimization — Umbrella Guide
> **Related Deep-Dives:** [LLM Model Pruning](../llm_model_pruning_guide.md) | [LLM Compression Alternatives](llm_compression_alternatives_guide.md) | [LLM Latency Optimization](llm_latency_optimization_guide.md)

---

## Table of Contents

1. [What Is LLM Optimization?](#1-what-is-llm-optimization)
2. [Why LLM Optimization Matters](#2-why-llm-optimization-matters)
3. [The LLM Optimization Stack](#3-the-llm-optimization-stack)
   - [Level 1 — Model Architecture](#31-level-1--model-architecture)
   - [Level 2 — Model Compression](#32-level-2--model-compression)
   - [Level 3 — Inference Optimization](#33-level-3--inference-optimization)
   - [Level 4 — Infrastructure Optimization](#34-level-4--infrastructure-optimization)
   - [Level 5 — Prompt and System Optimization](#35-level-5--prompt-and-system-optimization)
4. [Comparison of Optimization Techniques](#4-comparison-of-optimization-techniques)
5. [End-to-End Optimization Pipeline](#5-end-to-end-optimization-pipeline)
6. [Pipeline Decision Points](#6-pipeline-decision-points)
7. [Cost-Latency-Accuracy Trade-off Framework](#7-cost-latency-accuracy-trade-off-framework)
8. [Optimization Evaluation Metrics](#8-optimization-evaluation-metrics)
9. [LLM Optimization for Banking](#9-llm-optimization-for-banking)
10. [Tools and Platforms Comparison](#10-tools-and-platforms-comparison)
11. [Getting-Started Decision Tree](#11-getting-started-decision-tree)
12. [Further Reading and Deep Dives](#12-further-reading-and-deep-dives)

---

## 1. What Is LLM Optimization?

LLM optimization is the complete stack of techniques — spanning model architecture, compression, inference infrastructure, system architecture, and operational practices — that collectively reduce the size, latency, and cost of deploying Large Language Models while preserving acceptable accuracy. It is what transforms a research model into a production-grade service.

**LLM optimization encompasses:**

| Layer | What It Optimizes | Typical Gains |
|---|---|---|
| Model Architecture | Design choices before training | 1.2–3× efficiency |
| Model Compression | Size and precision after training | 2–16× compression |
| Inference Optimization | Serving engine and runtime | 2–5× speedup |
| Infrastructure Optimization | Hardware and deployment topology | 1.5–3× cost reduction |
| Prompt & System Optimization | Request-level efficiency | 1.2–2× cost reduction |

**Key insight:** Optimization is not a single technique — it is a pipeline. The best results come from applying techniques at multiple levels in the right order. A model that is pruned, quantized, served on an optimized engine with batch caching on efficiently scaled infrastructure can run **10–50× cheaper and faster** than the unoptimized baseline.

**The golden rule of optimization:** Measure before you optimize, and measure after every step. Without a baseline, you cannot know what works.

---

## 2. Why LLM Optimization Matters

### 2.1 Cost — The Primary Driver

GPT-4 class models cost **$10–100+ per hour per GPU** to run. Serving a single 70B model on an A100 80GB costs approximately $2–4/hour in cloud compute. At production scale serving millions of requests per day, these costs compound rapidly.

- **Without optimization:** A single 70B model serving 1M requests/day (2K tokens each) on A100-80G GPUs ≈ $50,000–$100,000/month.
- **With optimization (pruning + quantization + batching):** Same workload on optimized 7B model with INT4 compression ≈ $3,000–$10,000/month — a **5–20× reduction**.

### 2.2 Latency — User Expectations

Users expect sub-second responses. Every 100ms of additional latency measurably reduces user engagement and conversion rates.

- Unoptimized 70B model: 3–8 seconds per response
- Optimized (INT4 + speculative decoding + vLLM): 100–500ms — **2–50× speedup**

### 2.3 Scalability — Serving at Scale

Serving millions of concurrent users requires efficient infrastructure. The same GPU cluster can serve 5–20× more users with optimized models, directly reducing infrastructure footprint and capital expenditure.

### 2.4 Deployment Flexibility

Optimized models run on constrained hardware — laptops, edge devices, CPUs — enabling use cases that are impossible with full-precision, full-size models:

- **On-device AI:** Laptops, mobile devices, IoT
- **Edge deployments:** Retail, manufacturing, healthcare
- **Air-gapped systems:** Defense, classified environments

### 2.5 Energy Efficiency

Reducing compute reduces environmental impact. A single LLM inference call can consume 0.1–10 Wh depending on model size. At 1M requests/day, optimization can save **MWh per month**.

### 2.6 Enabling New Use Cases

Real-time applications (voice assistants, live translation), on-device AI (offline assistants, privacy-preserving inference), and cost-sensitive workloads (high-volume document processing, customer service triage) are only economically viable with aggressive optimization.

### 2.7 Banking Context — Special Considerations

For Crédit Agricole CIB and tier-1 financial institutions, LLM optimization addresses specific constraints:

- **Cost optimization for on-premise deployment:** Banking operates private GPU clusters. Every GPU not purchased is a direct capital saving.
- **Data residency:** Customer data and financial transactions cannot leave the bank's infrastructure. Local inference requires models that fit within on-premise GPU budgets.
- **Latency for real-time fraud detection:** <100ms response times are required for transaction screening. Optimized models can meet this on local hardware.
- **Regulatory compliance:** SR 11-7 (Model Risk Management) requires documented optimization processes. This guide, and the companion deep-dives, provide the documentation framework.

---

## 3. The LLM Optimization Stack

LLM optimization operates at five distinct levels. Each level targets different bottlenecks, and the techniques are complementary — applying techniques at multiple levels compounds the gains.

### 3.1 Level 1 — Model Architecture

Architecture-level optimization happens **before training**. The choices made here determine the baseline efficiency of the model. Retrofitting is possible but expensive — most architecture optimizations require pre-training or significant fine-tuning.

#### 3.1.1 Attention Mechanisms

The attention mechanism is the dominant compute cost in transformer models. Architecture choices fundamentally determine the cost-performance envelope.

| Architecture | Key-Value Head Sharing | Memory per Token | Compute per Token | Quality Impact |
|---|---|---|---|---|
| MHA (Multi-Head Attention) | None — each head has independent KV | High | High | Baseline |
| GQA (Grouped-Query Attention) | Groups of query heads share one KV head | Medium | Medium | Minimal (used in Llama 2/3) |
| MQA (Multi-Query Attention) | All query heads share one KV head | Low | Low | Small degradation (used in Falcon, PaLM) |

**Recommendation:** GQA provides the best accuracy-efficiency trade-off for most production use cases. MQA trades ~1–2% accuracy for ~30% faster inference at high batch sizes.

#### 3.1.2 Attention Optimizations

- **FlashAttention-2/3:** Fuses the attention computation to avoid materializing the full attention matrix in GPU HBM. Achieves 2–3× speedup over standard attention with **no accuracy loss**. Now standard in most training and inference frameworks.
- **Sparse Attention:** Only computes attention for a subset of token pairs (local windows, global tokens, random patterns). Useful for long-context models where full O(n²) attention is prohibitive.
- **Sliding Window Attention:** Each token only attends to N nearby tokens. Used in Mistral, yields O(n) attention cost. Excellent for streaming/text generation, less suitable for tasks requiring long-range bidirectional context.
- **ALiBi (Attention with Linear Biases):** Adds position-dependent bias directly to attention scores, enabling extrapolation to longer sequences than trained on. Faster than learned position embeddings.
- **RoPE (Rotary Position Embedding):** Encodes position through rotation of query/key vectors. Standard in Llama, Mistral, and most modern architectures. Supports relative position encoding without extra parameters.

#### 3.1.3 Activation Functions

- **SwiGLU / SiLU:** Used in Llama, PaLM. Higher quality than ReLU/GELU but adds ~2/3 extra parameters in FFN layers due to the gating mechanism. The quality gain usually outweighs the parameter cost.
- **ReLU / GELU:** Simpler, fewer parameters, lower quality. Rare in modern LLMs but efficient for edge deployment.
- **Squared ReLU:** Emerging alternative that matches SwiGLU quality with simpler computation.

#### 3.1.4 Normalization

- **RMSNorm:** Used in Llama, Mistral, Gemma. ~15–20% faster than LayerNorm with equivalent quality. Replaces the mean-centering step with root-mean-square scaling only.
- **LayerNorm:** Full normalization (mean and variance). Higher computational cost, used in older architectures.

#### 3.1.5 Vocabulary and Embedding Configuration

- **Vocabulary size:** Larger vocabularies (128K+ tokens) reduce the number of tokens needed to represent text (2–3× fewer tokens for some languages), reducing overall compute. Used in Llama 3 (128K), GPT-4. However, larger vocabularies increase embedding table size linearly.
- **Embedding dimension:** The embedding dimension determines the model width. Smaller dimensions reduce memory but may bottleneck representational capacity.
- **Tied embeddings:** Tying input and output embedding matrices saves billions of parameters in large models. Common practice in most modern LLMs.

#### 3.1.6 Parallelism Strategies

Parallelism is critical for training and serving large models. Different strategies address different bottlenecks:

| Strategy | Description | Bottleneck Solved | Best For |
|---|---|---|---|
| Tensor Parallelism (TP) | Splits individual tensor operations across GPUs | Compute & memory per layer | Single-node multi-GPU |
| Pipeline Parallelism (PP) | Layers distributed across GPUs | Memory (full model fit) | Multi-node large models |
| Sequence Parallelism (SP) | Splits sequence dimension across GPUs | Long-context memory | Very long sequences |
| Expert Parallelism (EP) | Distributes experts across GPUs | MoE model memory | Mixture-of-Experts |
| Data Parallelism (DP) | Replicates model, splits data batch | Throughput | Training at scale |

For inference serving, tensor parallelism is the most common pattern. Pipeline parallelism introduces inter-node communication overhead that often erases gains for latency-sensitive workloads.

---

### 3.2 Level 2 — Model Compression

Model compression techniques reduce model size after training. These are the most impactful optimization tools, often delivering 4–16× compression with minimal accuracy loss.

#### 3.2.1 Pruning

Pruning removes weights that contribute minimally to model output. The remaining weights stay at full precision, so the model quality is preserved across the sparse structure.

**Key techniques (detailed in the [LLM Model Pruning Guide](../llm_model_pruning_guide.md)):**

| Technique | Type | Sparsity Target | Key Feature | Accuracy at 50% |
|---|---|---|---|---|
| Magnitude Pruning | Unstructured | 30–60% | Simplest; removes smallest weights | Moderate drop |
| SparseGPT | Unstructured | 50–70% | One-shot, no fine-tuning; optimal brain surgeon | Minimal drop |
| Wanda | Unstructured | 50–60% | One-shot, weight × activation magnitude | Slight drop |
| 2:4 Structured Pruning | Semi-structured | 50% | NVIDIA Ampere/Hopper hardware speedup | Minimal drop |
| SliceGPT | Structured | 20–30% | Removes entire rows/columns; speeds up dense compute | Moderate drop |

**Golden rule:** Prune first, then quantize. The pruning importance metrics work best on clean (unquantized) weights, and quantization after pruning is more accurate than the reverse order.

**SparseGPT at 50% sparsity is the recommended starting point** for most production workloads — it requires no fine-tuning, preserves accuracy well, and provides a solid foundation for additional compression.

#### 3.2.2 Quantization

Quantization reduces the number of bits used to represent each weight (and sometimes activations). Instead of removing weights (pruning), it stores each weight in fewer bits.

**Key techniques (detailed in the [LLM Compression Alternatives Guide](llm_compression_alternatives_guide.md)):**

| Technique | Precision | Memory Saving | Best For | Accuracy Impact |
|---|---|---|---|---|
| GPTQ | INT4/INT3/INT2 | 4× (INT4) | GPU inference, batch serving | Minimal at INT4, moderate at INT3 |
| AWQ | INT4 | 4× | GPU inference, latency-sensitive | Minimal — best INT4 quality |
| GGUF (llama.cpp) | INT4/INT5/INT8/Q8_0 | 2–5× | CPU inference, edge, Apple Silicon | Varies by quantization type |
| bitsandbytes NF4 | 4-bit NormalFloat | 4× | Training-time quantization (QLoRA) | Good for training, not inference |
| FP8 (E4M3/E5M2) | FP8 | 2× | H100/H200 native, accuracy-critical | Near-lossless |
| INT8 | INT8 | 2× | Older GPUs, CPU | Near-lossless |

**AWQ INT4 is the recommended default for GPU serving** — it consistently achieves the best accuracy-quality trade-off at 4× compression for LLMs.

**FP8 is the recommended choice for accuracy-critical workloads** (banking, healthcare) where even minimal degradation is unacceptable.

#### 3.2.3 Knowledge Distillation

Knowledge distillation trains a smaller "student" model to mimic the outputs of a larger "teacher" model. Unlike pruning and quantization (which operate on an existing model), distillation produces a new, smaller model.

| Approach | Description | Student Size | Example Models |
|---|---|---|---|
| Logit distillation | Student learns from teacher's output probabilities | 1.5–3× smaller | DistilBERT (40% smaller, 97% BERT quality) |
| Feature distillation | Student learns from intermediate representations | 2–5× smaller | TinyBERT |
| Data distillation | Student trained on teacher-generated data | 3–10× smaller | Phi-1, Phi-1.5, Phi-2 |
| Response distillation | Student learns from teacher's chain-of-thought | 3–10× smaller | Orca (13B with GPT-4 quality) |

**The Phi family (Microsoft):** Phi-1 (1.3B), Phi-1.5 (1.3B), Phi-2 (2.7B), Phi-3 (3.8B / 7B / 14B) demonstrate that small models trained on high-quality distilled datasets (textbooks, synthetic data) can match or exceed models 5–10× larger. This is the frontier of efficient AI: **small models, high-quality data, big results.**

**The Orca approach (Microsoft):** Orca (13B) learns to explain its reasoning process (chain-of-thought) from GPT-4. Not just the answer — the reasoning path. This technique produces a small model that can tackle complex tasks typically requiring large models.

#### 3.2.4 Model Merging

Model merging combines multiple fine-tuned models into a single model without additional training. It is a form of "free" optimization — you get the capabilities of multiple specialized models in one.

| Technique | Description | Typical Gain | Complexity |
|---|---|---|---|
| Model Soups | Average weights of fine-tuned models | Improved robustness | Low |
| TIES-Merging | Trim redundant weights, Elect sign consensus, Disjoint merge | Better task performance | Medium |
| DARE (Drop And REscale) | Randomly drop delta weights, rescale remainder | Preserve diverse capabilities | Medium |
| Evolutionary Merging (Sakana AI) | Use evolutionary algorithms to discover optimal merges | State-of-the-art merged models | High |

Model merging is particularly valuable for creating multi-capability models (code + chat + reasoning) without training a new model from scratch.

---

### 3.3 Level 3 — Inference Optimization

Inference optimization techniques make the runtime execution of models faster and more efficient. These are the tools you use **after** model selection and compression.

#### 3.3.1 Serving Engines

Serving engines are the runtime infrastructure that runs LLM inference. The choice of engine determines how efficiently your compressed model performs.

| Engine | Key Innovation | GPU | CPU | Edge | Best For |
|---|---|---|---|---|---|
| **vLLM** | PagedAttention | ✓ | ✗ | ✗ | General GPU serving, ease of use |
| **TensorRT-LLM** | In-flight batching, FP8, kernel optimization | ✓ | ✗ | ✗ | Max NVIDIA GPU performance |
| **TGI (Hugging Face)** | Continuous batching | ✓ | ✓ | ✗ | HF integration, quick start |
| **llama.cpp** | GGUF, mmap, CPU optimizations | ✓ | ✓ | ✓ | CPU, Apple Silicon, edge |
| **DeepSpeed-MII** | ZeRO-Inference, hybrid engine | ✓ | ✗ | ✗ | Multi-GPU inference |
| **CTranslate2** | INT8/FP16, CPU/GPU | ✓ | ✓ | ✓ | Lightweight deployment |

**Detailed comparison available in the [LLM Latency Optimization Guide](llm_latency_optimization_guide.md).**

**Selection guidance:**
- **vLLM** is the best default for most GPU-serving workloads — easy to set up, excellent performance, supports most model architectures.
- **TensorRT-LLM** is the choice when maximum throughput on NVIDIA hardware is required (5–30% faster than vLLM, depending on workload).
- **llama.cpp** is the choice for CPU inference, Apple Silicon, edge devices, or when GGUF quants are the target format.

#### 3.3.2 Speculative Decoding

Speculative decoding uses a small "draft" model to generate candidate tokens, which the large "target" model then verifies in parallel. The intuition: the large model's expensive computation is parallelized, while the cheap draft model generates many tokens quickly.

| Technique | Draft Model | Speedup | Overhead |
|---|---|---|---|
| **Medusa** | Multiple draft heads on target model | 2–3× | Training needed for draft heads |
| **EAGLE/EAGLE-2** | Lightweight feature-level draft model | 2–4× | Small additional model needed |
| **Lookahead Decoding** | Generates n-grams speculatively | 1.5–2× | No training needed |
| **Self-Speculative** | Target model's own early layers as draft | 1.5–2× | No additional model |
| **Standard speculative decoding** | External small model as draft | 2–3× | Must serve two models |

**Speculative decoding is most valuable for latency-sensitive applications** where model size cannot be reduced further. It provides 2–3× speedup without changing the target model's accuracy.

#### 3.3.3 KV Cache Optimization

The KV (Key-Value) cache stores attention keys and values for previously generated tokens, growing linearly with sequence length. For large models and long contexts, the KV cache becomes the dominant memory consumer.

| Technique | Description | Memory Saving | Complexity |
|---|---|---|---|
| **PagedAttention (vLLM)** | Manages KV cache in non-contiguous pages, like OS virtual memory | 2–4× utilization | Built into vLLM |
| **Prefix Caching** | Reuse KV cache for common prefixes (system prompts) | Variable | Built into vLLM/SGLang |
| **KV Cache Quantization** | Store KV cache in INT4/INT8 instead of FP16 | 2–4× | Medium (KIVI, KVQuant) |
| **Memory KV Cache Compression (MInference)** | Compress KV cache using attention sparsity in long contexts | 10× at 128K | Medium |
| **KV Cache Offloading** | Offload KV cache to CPU memory | Variable | High |

**For production serving:** PagedAttention is a baseline requirement. Prefix caching provides immediate savings for workloads with fixed system prompts. KV cache quantization is the next frontier for long-context workloads.

#### 3.3.4 Batching Strategies

Batching increases throughput by processing multiple requests together, amortizing the model weight loading cost.

| Technique | Mechanism | Latency Impact | Throughput Gain |
|---|---|---|---|
| **Static Batching** | Wait for batch to fill, then process | High (increases latency) | 2–5× |
| **Dynamic Batching** | Process requests as they arrive, batch on the fly | Medium | 3–10× |
| **Continuous Batching** | Add new requests to running batch as others finish | Low | 5–20× |
| **In-flight Batching** | Batch different decode phases together (TensorRT-LLM) | Very low | 5–30× |

**Continuous batching (vLLM) and in-flight batching (TensorRT-LLM)** are the gold standard. They maximize GPU utilization without adding fixed batch-wait latency.

#### 3.3.5 Streaming

Streaming sends tokens to the client as they are generated, rather than waiting for the full response. This dramatically reduces **perceived** latency — the user sees the first token in milliseconds even if the full response takes seconds.

- **Time-to-first-token (TTFT):** The critical metric for streaming quality. Sub-100ms TTFT is the target.
- **Server-Sent Events (SSE):** The standard protocol for token streaming.
- **Chunked transfer encoding:** Used for HTTP streaming of LLM responses.

---

### 3.4 Level 4 — Infrastructure Optimization

Infrastructure optimization ensures the hardware and network topology are configured to maximize the efficiency gains from model and inference optimization.

#### 3.4.1 GPU Selection

Different GPUs have different price-performance characteristics. The right GPU for your workload depends on model size, latency requirements, and budget.

| GPU | Memory | Memory Bandwidth | FP8 Support | Best For | Est. Cloud Cost/hr |
|---|---|---|---|---|---|
| **H100 SXM** | 80 GB HBM3 | 3.35 TB/s | Yes | Maximum throughput, FP8, large models | $25–40 |
| **H200** | 141 GB HBM3e | 4.8 TB/s | Yes | Large models needing >80GB, memory-bound workloads | $30–45 |
| **A100 SXM** | 80 GB HBM2e | 2.0 TB/s | No | General-purpose, budget-friendly | $15–25 |
| **L40S** | 48 GB GDDR6 | 864 GB/s | Yes | Mid-range inference, smaller models | $5–10 |
| **L4** | 24 GB GDDR6 | 300 GB/s | Yes | Edge inference, cost-sensitive, small models | $2–4 |
| **TPU v5p** | 95 GB HBM2e | — | bfloat16 | Google Cloud, large-scale training | $30–50 |
| **TPU v4** | 32 GB HBM2 | — | bfloat16 | Google Cloud, efficient training | $15–25 |

**Key insight:** For inference, memory bandwidth is more important than raw compute. The H200's 4.8 TB/s bandwidth directly translates to faster token generation for memory-bound decode phases.

#### 3.4.2 GPU Memory Management

When a model doesn't fit in a single GPU's memory, distribution strategies are required.

| Strategy | How It Works | When to Use | Overhead |
|---|---|---|---|
| **Tensor Parallelism (TP)** | Split each layer across GPUs | Model fits in aggregate memory, need lower latency | Medium (all-reduce per layer) |
| **Pipeline Parallelism (PP)** | Split layers across GPUs | Model fits in aggregate, latency not critical | High (inter-node communication) |
| **ZeRO** | Partition optimizer states, gradients, parameters | Training or serving with weight updates | Medium |
| **Activation Checkpointing** | Don't store all activations, recompute on backward | Training only | 33% training slowdown, 2–4× memory saving |
| **Offloading** | Move weights/activations to CPU | When GPU memory is critically constrained | Very high (PCIe bandwidth bottleneck) |

**For inference serving:** Tensor parallelism (within a node) is the standard pattern. Pipeline parallelism across nodes adds too much latency for interactive workloads.

#### 3.4.3 Network Optimization

GPU-to-GPU communication is the primary bottleneck in distributed inference.

| Technology | Bandwidth | Latency | Use Case |
|---|---|---|---|
| **NVLink (H100)** | 900 GB/s | Sub-microsecond | Within-GPU communication |
| **InfiniBand NDR400** | 400 Gb/s | ~1μs | Inter-node GPU communication |
| **InfiniBand HDR200** | 200 Gb/s | ~1.3μs | Budget inter-node communication |
| **RoCE v2** | 200 Gb/s | ~3μs | Standard Ethernet alternative |
| **RDMA** | Enables direct GPU↔GPU memory access | | Required for TP/PP efficiency |

**For tensor parallelism:** NVLink within a node is essential. InfiniBand between nodes is required if TP must span nodes (avoid this if possible).

#### 3.4.4 Auto-Scaling

Scaling GPU infrastructure efficiently requires intelligent auto-scaling policies.

| Strategy | Description | Best For |
|---|---|---|
| **K8s GPU Node Autoscaling** | Add/remove GPU nodes based on queue depth | Inference with variable load |
| **Horizontal Scaling** | Add more replicas of inference endpoints | Throughput-bound workloads |
| **Vertical Scaling** | Use larger GPU instances | Memory-bound workloads |
| **Spot/Preemptible Instances** | Use discounted but interruptible VMs | Training, batch inference |
| **Inference Queue Management** | Prioritize and shape requests to maximize throughput | Peak load management |

**Recommendation:** Use K8s with GPU node autoscaling for inference workloads. Use spot instances for training and batch inference (with checkpointing for fault tolerance).

#### 3.4.5 Deployment Patterns

| Pattern | Description | Best For |
|---|---|---|
| **Single-Model Endpoint** | One model per endpoint/serving pod | Simple workloads, dedicated models |
| **Multi-Model Serving** | One endpoint serves multiple models from shared pool | Cost efficiency, mixed workloads |
| **Model Router** | Route requests to different models based on complexity | Cost-quality optimization |
| **Edge Deployment** | Models run on local devices | Offline, privacy, latency-critical |
| **Hybrid Edge-Cloud** | Simple tasks on edge, complex tasks sent to cloud | Best of both worlds |

---

### 3.5 Level 5 — Prompt and System Optimization

Even after the model and infrastructure are optimized, the way you interact with the system has a significant impact on cost and latency.

#### 3.5.1 Prompt Engineering for Efficiency

| Technique | Savings | Implementation |
|---|---|---|
| **Shorter prompts** | 10–50% cost reduction (linear with token count) | Trim unnecessary instructions, examples |
| **System prompt caching** | 50–80% on system tokens | Prefix caching in vLLM/SGLang |
| **Prefix caching** | Reuse KV for shared prompt prefixes | Built into vLLM, TGI, SGLang |
| **Context window management** | Avoid context window growth for long conversations | Set max context, truncate early, summarize history |

**Every token in the prompt costs money and time.** A prompt that is 50% shorter costs 50% less and responds 50% faster.

#### 3.5.2 Output Optimization

| Technique | Savings | Trade-off |
|---|---|---|
| **Max tokens limit** | Proportional to output length | Shorter outputs may be incomplete |
| **Early stopping** | Stop generation when confidence threshold met | May truncate reasoning |
| **Constrained generation (JSON, regex)** | Forces shorter, structured outputs | May affect output quality |
| **Minimal output style** | "Answer concisely" reduces output by 30–50% | Less verbose, same information |
| **Speculative decoding** | 2–3× speedup without output quality change | Requires draft model |

#### 3.5.3 Caching Strategies

| Cache Type | What It Caches | Hit Conditions | Impact |
|---|---|---|---|
| **Semantic caching** | Response for semantically similar queries | Embedding similarity ≥ threshold | 10–40% reduction for repetitive queries |
| **Response caching** | Exact response for exact query | Exact string match | 5–15% reduction |
| **KV cache caching** | KV cache for prompt prefixes | Prefix match (longest prefix wins) | 50–80% on shared prefixes |
| **Longest prefix match (LPM)** | Reuse KV for best-matching prefix | Partial prefix match | 30–60% on structured prompts |

**Semantic caching is the highest-ROI caching strategy** for customer service, chatbots, and any workload with repetitive query patterns.

#### 3.5.4 Request Batching

| Strategy | Mechanism | Best For |
|---|---|---|
| **Dynamic batching** | Group requests arriving within a time window | Variable throughput workloads |
| **Continuous batching** | Add to running batch as slots free up | Consistent throughput workloads |
| **In-flight batching** | Batch decode phases across requests | Maximum GPU utilization |

#### 3.5.5 Efficient Model Selection

| Strategy | Description | Savings |
|---|---|---|
| **Route simple queries to smaller models** | Use a classifier/router to direct queries | 50–80% cost reduction on easy queries |
| **Model router for cost-quality trade-off** | Multi-model endpoint with dynamic routing | Adaptive cost-quality balance |
| **Cascade models** | Start with small model, escalate to large if confidence low | 60–90% cost reduction on simple queries |
| **Task-specific fine-tuned models** | Small model specialized for task outperforms large general model | Better quality + lower cost |

---

## 4. Comparison of Optimization Techniques

| Technique | Optimization Level | Cost Reduction | Latency Reduction | Implementation Complexity | Accuracy Impact | Hardware Requirement | Best For |
|---|---|---|---|---|---|---|---|
| **MHA → GQA/MQA** | Architecture | 1.5–2× | 1.3–2× | High (requires pre-training) | Minimal (GQA) to small (MQA) | Any | New model training |
| **FlashAttention-2/3** | Architecture | 2–3× | 2–3× | Low | None | GPU (Ampere+) | Any LLM deployment |
| **SparseGPT (50%)** | Compression | 2× | 1.5–2× | Medium | Minimal | GPU with eval compute | Max compression path |
| **AWQ INT4** | Compression | 4× | 2–3× | Low | Minimal | GPU (CUDA) | Default GPU quantization |
| **GPTQ INT4** | Compression | 4× | 2–3× | Medium | Minimal–small | GPU (CUDA) | Alternative quantization |
| **GGUF Q4_K_M** | Compression | 4× | 2–4× | Low | Small | CPU, Apple Silicon | Edge/CPU deployment |
| **FP8 (H100)** | Compression | 2× | 1.5× | Low | Near-zero | H100/H200 | Accuracy-critical |
| **Knowledge Distillation** | Compression | 3–10× | 3–10× | Very high | Small–moderate | Training GPU cluster | New model creation |
| **Model Merging** | Compression | 0× (same size) | 0× | Low–medium | Variable | GPU for eval | Combined capabilities |
| **vLLM PagedAttention** | Inference | 2–4× | 1.5–3× | Low | None | GPU | General GPU serving |
| **TensorRT-LLM** | Inference | 2–5× | 2–4× | Medium | None | NVIDIA GPU | Max NVIDIA perf |
| **llama.cpp** | Inference | 3–6× (CPU) | 2–4× | Low | None | CPU, Apple | CPU/edge inference |
| **Speculative Decoding** | Inference | — | 2–3× | Medium | None | GPU | Latency-critical |
| **KV Cache Optimization** | Inference | 1.5–4× | 1.3–2× | Low–medium | None | GPU | Long-context serving |
| **Continuous Batching** | Inference | 3–10× | 1× (same) | Low | None | GPU | High throughput |
| **GPU Right-Sizing** | Infrastructure | 2–5× | 1× (same) | Low | None | Any | Cost optimization |
| **Auto-Scaling** | Infrastructure | 1.5–3× | 1× (same) | Medium | None | Cloud/K8s | Variable load |
| **Semantic Caching** | System | 1.5–5× | 5–50× (hit) | Medium | None (cached) | Any | Repetitive queries |
| **Model Router** | System | 2–5× | Variable | Medium | Variable | GPU cluster | Diverse query complexity |
| **Prompt Optimization** | System | 1.2–2× | 1.2–2× | Low | None | Any | All workloads |

---

## 5. End-to-End Optimization Pipeline

The following is the recommended pipeline for optimizing an LLM from baseline to production deployment. Each step builds on the previous one, and skipping steps may leave significant optimization gains on the table.

### Step 1 — Define Requirements

Before any optimization, establish measurable targets:

| Requirement | Question to Answer | Typical Values |
|---|---|---|
| **Latency SLO** | What is the maximum acceptable E2E response time? | <200ms (real-time), <2s (interactive), <5s (batch) |
| **Throughput** | How many requests per second/hour/day? | 10 req/s (small), 10K req/s (enterprise) |
| **Budget** | What is the maximum $/request or $/month? | $0.001–$0.10/req |
| **Accuracy Floor** | What is the minimum acceptable task accuracy? | 95% or X% drop from baseline |
| **Hardware Constraints** | What hardware is available or budgeted? | A100 80GB, H100, CPU only, edge device |

### Step 2 — Model Selection

Choose a base model appropriate for the task. Consider the size vs. capability trade-off.

- **Task complexity:** Simple classification → small model (1–7B). Complex reasoning → larger model (13–70B+).
- **Size-capability frontier:** Models of similar size have diverging capabilities. Compare benchmarks (MMLU, GSM8K, HumanEval) for your specific task domain.
- **Architecture support:** Ensure the model architecture is supported by your target serving engine (vLLM supports most, TensorRT-LLM requires specific format conversion).

### Step 3 — Compress the Model

Apply compression in the correct order:

1. **Prune first** — SparseGPT at 50% sparsity is the recommended starting point
2. **Measure accuracy** — Run task benchmarks and compare to baseline
3. **Quantize** — AWQ INT4 for GPU, GGUF Q4_K_M for CPU
4. **Measure accuracy again** — Validate that combined compression is within acceptable degradation
5. **Iterate** — If accuracy is below floor, reduce sparsity (e.g., 30%) or use higher precision (FP8)

### Step 4 — Deploy Optimized Model

Choose serving engine based on target hardware:

- **GPU serving:** vLLM (general) or TensorRT-LLM (max performance)
- **CPU/Edge serving:** llama.cpp (GGUF format)
- **Apple Silicon:** MLX or llama.cpp

### Step 5 — Optimize Serving Configuration

Configure the serving engine for maximum performance:

- Enable **KV cache optimization** (PagedAttention default in vLLM)
- Configure **streaming** for lower perceived latency
- Enable **prefix caching** if workloads share system prompts
- Enable **continuous batching** for throughput optimization
- Configure **model router** if diverse query complexity exists

### Step 6 — Scale Infrastructure

- **Right-size GPU instances** — Don't use H100 for 7B models where L40S suffices
- **Configure auto-scaling** — Use K8s GPU node autoscaling for variable load
- **Optimize networking** — Ensure NVLink within nodes, InfiniBand between nodes for distributed inference

### Step 7 — Monitor and Iterate

Monitor continuously and iterate when conditions change:

| Metric | What to Watch | When to Act |
|---|---|---|
| **Latency (p50/p95/p99)** | TTFT, TPS, E2E latency | Consistently above SLO → add resources or reduce model size |
| **Throughput** | Requests/sec, tokens/sec | Flat or dropping → check GPU utilization, batching |
| **Cost** | $/1M tokens, $/request | Above budget → further compress, switch model, add caching |
| **Accuracy** | Task benchmarks, drift | Degradation > tolerance → rollback or adjust compression |
| **GPU Utilization** | %, memory used | <50% → right-size or consolidate; >95% → scale out |

---

## 6. Pipeline Decision Points

### Prune or Not?

| Scenario | Decision | Rationale |
|---|---|---|
| Need max compression | **Yes, prune first (SparseGPT 50%)** | Pruning + quantization compounds savings |
| Compute for evaluation available | **Yes** | Evaluation lets you verify accuracy preservation |
| Accuracy is critical and cannot degrade | **No** | Use FP8 quantization only |
| Model already fits in available memory | **Maybe** | Pruning adds latency via sparse compute; only prune if you need additional headroom |
| Small model (<7B) | **Skip** | Pruning benefits diminish for small models; quantization alone is sufficient |

### Quantize to What?

| Constraint | Decision | Rationale |
|---|---|---|
| Max compression | **INT4 (AWQ or GPTQ)** | 4× compression, minimal quality loss |
| Max accuracy preservation | **FP8** | Near-lossless, native H100 support |
| GPU serving | **AWQ INT4 or FP8** | AWQ for 4×, FP8 for accuracy |
| CPU/Edge serving | **GGUF Q4_K_M** | Optimized for CPU inference |
| Apple Silicon | **GGUF Q4_K_M or MLX** | Native acceleration |
| Training with quantization (QLoRA) | **bitsandbytes NF4** | Designed for training-time quantization |

### Which Serving Engine?

| Scenario | Recommended Engine | Why |
|---|---|---|
| General GPU serving | **vLLM** | Easy setup, wide model support, PagedAttention, excellent performance |
| Max NVIDIA performance | **TensorRT-LLM** | NVIDIA-optimized kernels, in-flight batching, FP8 |
| CPU/Edge deployment | **llama.cpp** | GGUF format, CPU optimizations, mmap loading |
| Apple Silicon | **llama.cpp** or **MLX** | Native Metal/Apple GPU acceleration |
| Quick prototype | **TGI** | HF ecosystem integration, simple API |
| Multi-GPU inference | **DeepSpeed-MII** | ZeRO-Inference, hybrid engine |
| Lightweight deployment | **CTranslate2** | Small binary, fast CPU inference |

### Cache Strategy?

| Query Pattern | Recommended Strategy | Expected Hit Rate |
|---|---|---|
| High repeat rate (customer service) | **Semantic caching** | 30–60% for common questions |
| Shared system prompts (chatbots) | **KV prefix caching** | 50–80% on system tokens |
| Exact duplicate queries (API) | **Response caching** | 5–15% |
| Structured prompts (templates) | **Longest prefix match** | 30–60% |
| Unique queries (fraud detection) | **No caching** | 0% hit rate expected |

### Model Router?

| Query Diversity | Decision | Rationale |
|---|---|---|
| Simple + complex queries mixed | **Yes** | Route easy queries to small/cheap model (1–3B), hard queries to large model (13–70B+) |
| All queries similar difficulty | **No** | Routing adds latency and complexity with no benefit |
| Cost optimization priority | **Yes** | Cascade: start with small model, escalate to large if low confidence |
| Latency is the priority | **Maybe** | Router adds ~10–50ms overhead; only use if routing saves more time than it costs |

---

## 7. Cost-Latency-Accuracy Trade-off Framework

### The Fundamental Trade-off

Better accuracy requires larger models, which increases both cost and latency. This is the fundamental constraint in LLM deployment.

```
Accuracy
    ↑
    |                               70B
    |                          13B
    |                     7B
    |                3B
    |           1B
    |_________________________→ Cost / Latency
```

### How Optimization Shifts the Frontier

Optimization techniques shift the cost/latency/accuracy frontier **outward** — achieving better accuracy at the same cost, or lower cost at the same accuracy.

```
Accuracy
    ↑
    |            [Optimized 70B]          ← After pruning + quant
    |        [Optimized 13B]
    |   [Optimized 7B]
    |      [Unoptimized 13B]
    |  [Unoptimized 7B]
    |_________________________→ Cost / Latency
```

Each technique shifts the frontier:

| Technique | Shift Direction | Magnitude |
|---|---|---|
| Pruning (50%) | Left (lower cost) | ~2× |
| Quantization (INT4) | Left (lower cost) | ~4× |
| Knowledge Distillation | Up-left (better accuracy at lower cost) | 3–10× |
| Speculative Decoding | Left (lower latency) | 2–3× |
| Caching | Left (lower cost on hit) | 2–50× |
| Serving Engine | Left (lower cost and latency) | 2–5× |

### Choosing the Sweet Spot

1. **Define minimum acceptable accuracy** — This is your non-negotiable constraint. No optimization can violate this.
2. **Optimize for cost or latency** — Given the accuracy constraint, minimize cost or latency (whichever is the primary driver).
3. **Apply techniques in order of ROI** — Start with the highest-impact, lowest-risk techniques (quantization, caching, serving engine choice), then add more aggressive techniques (pruning, distillation, model merging).

### Example 1 — Real-Time Fraud Detection

- **Constraints:** <200ms latency, 95%+ accuracy
- **Primary driver:** Speed (within accuracy constraint)
- **Optimization stack:** 7B model + AWQ INT4 + vLLM + KV cache + streaming
- **Result:** ~150ms latency at ~$0.001/call
- **Rationale:** Smaller model meets accuracy floor; quantization adds speed; vLLM provides PagedAttention + continuous batching; streaming minimizes perceived latency

### Example 2 — Batch Document Processing

- **Constraints:** No strict latency requirement, minimize cost
- **Primary driver:** Cost
- **Optimization stack:** 70B → SparseGPT 50% → GPTQ INT4 → TensorRT-LLM batch
- **Result:** ~$0.0001/page
- **Rationale:** Large model for document accuracy; aggressive compression (50% prune + INT4 quant ≈ 8× total); batch processing maximizes throughput

### Example 3 — On-Premise Banking Deployment

- **Constraints:** Single A100 80GB, model must fit with margin
- **Primary driver:** Memory
- **Optimization stack:** 70B → SparseGPT 50% → INT4 quant = ~20GB
- **Result:** ~20GB model, fits on A100 with 60GB for KV cache and overhead
- **Rationale:** 70B in FP16 = 140GB (cannot fit). 50% prune = 70GB params in FP16. INT4 quant = ~17.5GB + overhead ≈ 20GB total. Fits with headroom.

---

## 8. Optimization Evaluation Metrics

### 8.1 Latency Metrics

| Metric | Definition | Target | Measurement |
|---|---|---|---|
| **TTFT (Time To First Token)** | Time from request receipt to first output token | <100ms (streaming), <500ms (non-streaming) | Engine latency logs |
| **TPS (Tokens Per Second)** | Output tokens generated per second | 50–200 TPS (depends on model size) | Engine throughput logs |
| **End-to-End Latency** | Total time from request to full response | <200ms (real-time), <2s (interactive) | Application-level timing |
| **p50/p95/p99 Latency** | Percentile distribution of response times | p99 < 2× p50 | Statsd/Prometheus aggregation |

### 8.2 Throughput Metrics

| Metric | Definition | Target |
|---|---|---|
| **Requests per Second (RPS)** | Number of completed inference requests per second | Varies by deployment (10–10K+) |
| **Tokens per Second per GPU** | Total tokens generated per second across all running requests | Maximize given latency SLO |
| **Concurrent Users Supported** | Maximum simultaneous users while maintaining latency SLO | Varies by deployment |

### 8.3 Cost Metrics

| Metric | Definition | Target |
|---|---|---|
| **Cost per 1M Tokens** | Total serving cost for 1M output tokens | $0.10–$1.00 (optimized), $1–$10 (unoptimized) |
| **Cost per Request** | Average cost of a single inference request | $0.0001–$0.01 |
| **Cost per Hour per GPU** | Infrastructure cost for one GPU-hour | $2 (L4) to $40 (H100) |
| **Total Monthly Inference Cost** | Aggregate cost for all inference traffic | Budget-dependent |

### 8.4 Accuracy Metrics

| Metric | What It Measures | Typical Target |
|---|---|---|
| **Perplexity** | Model confidence in correct predictions | Lower is better; compare to baseline |
| **MMLU** | Multitask language understanding | Within 1–5% of baseline |
| **HellaSwag** | Commonsense reasoning | Within 1–3% of baseline |
| **GSM8K** | Math reasoning | Within 1–5% of baseline |
| **HumanEval** | Code generation | Within 1–5% of baseline |
| **Domain-Specific Validation** | Custom eval for your task | Within defined tolerance |
| **Ablation vs Baseline** | Compare optimized model to original on all tasks | Report delta per technique |

### 8.5 Resource Utilization Metrics

| Metric | Target | Action if Below |
|---|---|---|
| **GPU Memory Utilization** | 80–95% | Right-size instance or increase batch size |
| **GPU Compute Utilization** | 70–95% | Increase batch size, add concurrent requests |
| **CPU Usage** | <50% for GPU serving (backup tasks only) | Consolidate, reduce overhead |
| **Memory Bandwidth Utilization** | >80% for memory-bound decode | Optimize KV cache, reduce model size |

### 8.6 Optimization Gain Metrics

| Metric | Formula | Target |
|---|---|---|
| **Compression Ratio** | baselinesize / optimizedsize | 4–16× |
| **Speedup Factor** | baselinelatency / optimizedlatency | 2–50× |
| **Cost Reduction Factor** | baseline$perreq / optimized$perreq | 5–20× |
| **Accuracy Preservation** | optimizedaccuracy / baselineaccuracy | >0.95–0.99 |

### 8.7 Optimization Cost Metrics

| Metric | What It Measures | Typical Values |
|---|---|---|
| **Pruning Compute** | GPU hours for pruning pass | 10–100 GPU-hours for 70B (SparseGPT) |
| **Quantization Compute** | GPU hours for calibration | 1–10 GPU-hours for 70B (AWQ) |
| **Eval Compute** | GPU hours for accuracy validation | 5–50 GPU-hours per eval run |
| **Engineering Time** | Person-hours to implement optimization pipeline | 20–200 hours for first deployment |

---

## 9. LLM Optimization for Banking

### 9.1 Regulatory Requirements

Financial institutions face specific regulatory constraints that impact LLM optimization:

**Model Risk Management (SR 11-7, OCC/Fed):**
- Requires a **documented optimization process** — every compression step must be recorded, justified, and validated
- Optimization must **not introduce bias or degrade explainability** — compressed models must be validated against the original on fairness metrics
- **Validation of compressed models against baseline** — separate validation (independent team) must confirm accuracy preservation
- **Periodic monitoring of optimized model quality** — ongoing drift detection, performance tracking, and re-validation triggers

**Documentation requirements for each optimization step:**
1. **Rationale** for applying each technique (why prune vs. not, why INT4 vs. FP8)
2. **Methodology** used (SparseGPT parameters, calibration data, quantization algorithm)
3. **Validation results** comparing compressed model to baseline on all relevant metrics
4. **Fairness and bias analysis** comparing behavior across demographic groups
5. **Drift monitoring plan** for ongoing performance tracking

### 9.2 Use Case Optimization Profiles

#### Customer Service Chatbot

| Parameter | Requirement | Optimization Profile |
|---|---|---|
| Latency | <2s | 7B model |
| Throughput | 100+ concurrent users | AWQ INT4 |
| Accuracy | 90%+ | vLLM + continuous batching |
| Budget | $0.002/call max | Semantic caching (40% hit rate expected) |
| **Stack** | **7B + AWQ INT4 + vLLM + semantic caching + model router** | Model router: simple FAQ → 3B, complex questions → 7B |

#### Fraud Detection

| Parameter | Requirement | Optimization Profile |
|---|---|---|
| Latency | <100ms | 13B model |
| Accuracy | 99%+ | FP8 quantization (zero degradation) |
| Uniqueness | Each transaction unique | No caching |
| **Stack** | **13B + FP8 + TensorRT-LLM + speculative decoding** | Maximum speed with accuracy preservation |

#### Document Processing

| Parameter | Requirement | Optimization Profile |
|---|---|---|
| Latency | Batch (no strict SLO) | 70B → SparseGPT 50% → GPTQ INT4 |
| Throughput | High volume | TensorRT-LLM batch inference |
| Cost | Minimize | Aggressive compression, batch processing |
| **Stack** | **70B → 50% prune → INT4 quant → TensorRT-LLM batch** | Maximum compression for cost efficiency |

#### Regulatory Compliance

| Parameter | Requirement | Optimization Profile |
|---|---|---|
| Accuracy | No degradation | FP8 quantization only (no pruning) |
| Explainability | Full model, traceable | Full model with staged deployment |
| Audit | Documented process | Full traceability of optimization |
| **Stack** | **Full model → FP8 only + staged deployment + A/B testing + continuous monitoring** | Accuracy-preserving, audit-ready |

### 9.3 Common Banking Requirements

| Requirement | Implication for Optimization | Mitigation Strategy |
|---|---|---|
| **On-premise deployment** | Memory and GPU constrained | Aggressive compression (prune + quant), right-size model to available hardware |
| **Data residency** | No cloud, local inference only | Optimize models to fit within on-prem GPU capacity; use distillation to create deployable models |
| **Compliance audit** | Documented optimization process and validation | Maintain optimization log with rationale, methodology, validation results per technique |
| **Explainability** | Compressed model must maintain explainability | Avoid pruning techniques that remove interpretable structure; document which components are compressed |
| **Stability** | Consistent behavior across optimization | Validate that compressed model produces consistent outputs for identical inputs; test edge cases |
| **Fairness and bias** | Optimization must not introduce bias | Evaluate fairness metrics (demographic parity, equal opportunity) pre- and post-optimization |
| **Vendor risk** | Third-party optimization tools must be vetted | Maintain in-house capability for critical optimization steps; document usage of external tools |

---

## 10. Tools and Platforms Comparison

| Tool / Platform | Optimization Level | Key Features | Supported Techniques | Hardware | Open Source | Pricing |
|---|---|---|---|---|---|---|
| **vLLM** | Inference | PagedAttention, continuous batching, prefix caching, LoRA, multi-LoRA | AWQ, GPTQ, FP8, INT4, INT8 | NVIDIA GPU, AMD GPU | Yes | Free |
| **TensorRT-LLM** | Inference | In-flight batching, FP8, INT4/INT8, multi-GPU, inflight batching | AWQ, GPTQ, FP8, SmoothQuant, INT4, INT8, 2:4 sparse | NVIDIA GPU (A100+, H100) | Yes | Free (NVIDIA) |
| **TGI (HF)** | Inference | Continuous batching, streaming, weight streaming | GPTQ, AWQ, INT8, FP8, bitsandbytes | NVIDIA GPU, AMD GPU | Yes | Free |
| **llama.cpp** | Inference | GGUF, mmap, CPU/GPU hybrid, Apple Silicon | All GGUF types (Q2–Q8), IQ quants | CPU, Apple, NVIDIA, AMD | Yes | Free |
| **DeepSpeed-MII** | Inference | ZeRO-Inference, hybrid engine, multi-GPU | INT4, INT8, FP16 | NVIDIA GPU | Yes | Free |
| **CTranslate2** | Inference | INT8/FP16, CPU SIMD, GPU, parallel decoding | INT8, FP16, INT16 | CPU, NVIDIA GPU | Yes | Free |
| **SGLang** | Inference | Structured generation, RadixAttention, prefix caching | AWQ, GPTQ, FP8 | NVIDIA GPU | Yes | Free |
| **OpenAI API** | Full stack | Managed service, no infra management | — | Cloud (OpenAI-managed) | No | Pay-per-token |
| **Anthropic API** | Full stack | Managed service, safety-focused | — | Cloud (Anthropic-managed) | No | Pay-per-token |
| **Together AI** | Full stack | Managed API, model routing, fine-tuning | AWQ, GPTQ, FP8, speculative decoding | Cloud-managed | No | Pay-per-token |
| **Fireworks AI** | Full stack | Fast inference, multi-model, function calling | AWQ, INT4, speculative decoding | Cloud-managed | No | Pay-per-token |
| **Modal** | Infrastructure | Serverless GPU, auto-scaling, Python SDK | Any (bring your own engine) | Cloud (NVIDIA GPU) | No | Pay-per-second |
| **RunPod** | Infrastructure | Serverless GPU, spot instances, auto-scaling | Any | Cloud (NVIDIA GPU) | No | Pay-per-hour |
| **K8s + GPU Operator** | Infrastructure | GPU scheduling, MIG, autoscaling, monitoring | Any | Cloud or on-prem | Yes | Free |
| **BentoML** | Full stack | Serving, batching, monitoring, multi-model | vLLM, TGI, llama.cpp, custom engines | GPU/CPU | Yes | Free (self-hosted) |
| **Ray Serve** | Inference | Distributed serving, autoscaling, multi-model | vLLM, TensorRT-LLM, custom | GPU/CPU cluster | Yes | Free |
| **OpenLLM** | Inference | vLLM/TGI/llama.cpp integration, Bento | AWQ, GPTQ, GGUF | GPU/CPU | Yes | Free |
| **Neural Magic** | Compression | LLMCompressor, SparseML, CPU optimizations | Pruning (SparseGPT, Wanda), Quantization | GPU (compression), CPU (inference) | Yes | Free (open core) |
| **AutoAWQ** | Compression | Fast AWQ quantization, cuBLAS kernels | AWQ INT4 | NVIDIA GPU | Yes | Free |
| **AutoGPTQ** | Compression | GPTQ quantization, exllama kernels | GPTQ INT4/INT3/INT2 | NVIDIA GPU | Yes | Free |
| **bitsandbytes** | Compression | NF4, QLoRA, LLM.int8() | NF4, INT8 | NVIDIA GPU | Yes | Free |
| **AQLM** | Compression | Additive quantization, extreme compression | 2-bit, 3-bit | GPU | Yes | Free |
| **LM Eval Harness** | Evaluation | Standard benchmarks, extensible | Accuracy evaluation | GPU | Yes | Free |
| **MLflow** | MLOps | Experiment tracking, model registry, deployment | Monitoring | Any | Yes | Free |
| **Weights & Biases** | MLOps | Experiment tracking, sweeps, model registry | Monitoring | Any | No | Free tier + paid |

---

## 11. Getting-Started Optimization Decision Tree

### Start Here: Define Your Primary Constraint

```
┌──────────────────────────────────────────────────┐
│         What is your PRIMARY constraint?          │
└──────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
   COST-BOUND        LATENCY-BOUND       MEMORY-BOUND
   ↓                  ↓                  ↓
   (Go to Path A)     (Go to Path B)     (Go to Path C)

        ┌──────────────────────────────────────────┐
        │              ACCURACY-CRITICAL            │
        └──────────────────────────────────────────┘
                              ↓
                     (Go to Path D)
```

### Path A — Cost Optimization

```
Primary constraint: COST
Goal: Minimize $/request

1. Model Router
   → Route simple queries to small/cheap model (1–3B)
   → Complex queries to larger model (13B+)
   → Expected savings: 50–80% on routed queries

2. Semantic Caching
   → Cache responses for semantically similar queries
   → Expected hit rate: 30–60% for repetitive workloads
   → Savings: 30–60% on cached requests

3. Response Caching
   → Exact match cache for repeated queries
   → Savings: 5–15% on exact duplicates

4. Batch Processing
   → Group requests for batch inference
   → Higher throughput, lower per-request cost

5. Spot/Preemptible Instances
   → Use for training and batch inference
   → Savings: 60–90% vs on-demand

6. Right-Size GPU Instance
   → 7B model → L4 or L40S
   → 13B model → L40S or A100
   → 70B+ model → A100 or H100

7. Quantization
   → AWQ INT4 or GGUF Q4_K_M
   → Savings: 4× cost reduction

Recommended stack: Model Router + Semantic Cache + AWQ INT4 + Right-sized GPU
Expected cost reduction: 10–50× vs unoptimized baseline
```

### Path B — Latency Optimization

```
Primary constraint: LATENCY
Goal: Minimize response time

1. Serving Engine
   → vLLM (general) or TensorRT-LLM (max NVIDIA perf)
   → Enable PagedAttention + continuous batching

2. KV Cache Optimization
   → PagedAttention (default in vLLM)
   → Prefix caching for shared prompts
   → KV cache quantization for long contexts

3. Streaming
   → SSE for immediate first token
   → Improves perceived latency 5–10×

4. Quantization
   → AWQ INT4 (fast GPU inference)
   → FP8 (native H100 speed, near-lossless)

5. Speculative Decoding
   → Medusa or EAGLE for 2–3× speedup
   → Best for latency-critical paths

6. Smaller Model
   → Use smallest model that meets accuracy floor
   → 7B instead of 13B saves 2× latency

7. Prompt Caching
   → Prefix caching for system prompts
   → Saves prompt processing time

8. Early Stopping
   → Stop generation when answer is confident
   → Reduces output length by 30–50%

Recommended stack: vLLM/TensorRT-LLM + AWQ INT4 + Speculative Decoding + Streaming + Prefix Caching
Expected latency reduction: 5–50× vs unoptimized baseline
```

### Path C — Memory Optimization

```
Primary constraint: MEMORY
Goal: Fit model within available hardware

1. Pruning
   → SparseGPT at 50% sparsity
   → Reduces model size by 2×
   → Apply before quantization

2. Quantization
   → AWQ INT4 (GPU) or GGUF Q4_K_M (CPU)
   → Reduces model size by 4×
   → Combined with pruning: 8× total compression

3. GGUF for CPU
   → llama.cpp with GGUF format
   → Runs on systems without GPUs
   → Quantization types: Q4_K_M (balanced), Q2_K (max compression)

4. Quantization-Aware Smaller Model
   → Choose a smaller architecture trained for quantization
   → Example: Llama 3.2 1B/3B or Phi-3 mini
   → Train with quantization-aware training (QAT)

5. KV Cache Offloading
   → Offload KV cache to CPU memory when GPU memory is full
   → Enables longer context on limited GPU memory
   → Increases latency (PCIe transfer bottleneck)

Recommended stack: SparseGPT 50% → AWQ INT4 → Smaller quant-aware model (if needed) → KV cache offloading
Expected compression: 4–16× vs unoptimized baseline
```

### Path D — Accuracy-Preserving Optimization

```
Primary constraint: ACCURACY
Goal: Optimize with NO accuracy degradation

1. Quantization (FP8 Only)
   → FP8 quantization (H100/H200 native)
   → Near-lossless, typically <0.5% degradation on benchmarks
   → No pruning (pruning always has some accuracy impact)

2. Same Architecture, Smaller Model
   → Same architecture family, smaller parameter count
   → Maintains behavior patterns and output quality
   → Example: Llama 3.1 8B instead of 70B for simpler tasks

3. Staged Deployment with A/B Testing
   → Roll out optimized model incrementally
   → A/B test against baseline in production
   → Roll back immediately if degradation detected

4. No Caching for Unique Queries
   → Caching serves stale or approximate responses
   → For accuracy-critical workloads: always compute fresh
   → Exception: exact-match caching with TTL < 1 second

5. Full Precision Fallback
   → Serve full-precision model for hard queries
   → Use router to send complex queries to baseline
   → Maintains accuracy while saving cost on easy queries

6. Continuous Monitoring
   → Track accuracy metrics in production
   → Automated drift detection and alerting
   → Rollback trigger on degradation exceeding threshold

Recommended stack: FP8 quantization only + staged deployment + A/B testing + full precision fallback
Expected cost reduction: 2× (from FP8) — will not match aggressive compression
Expected latency reduction: 1.5× (from FP8 kernel speed)
```

### Hybrid Decision Table

| Primary Constraint | Secondary | Recommended Hybrid Path |
|---|---|---|
| Cost | Latency matters (<2s) | Path A + streaming + vLLM (no speculative decoding — adds complexity) |
| Cost | Accuracy critical | Path D for accuracy tasks + Path A for cost savings on routine tasks |
| Latency | Cost matters | Path B + batch processing for non-latency-sensitive requests |
| Latency | Memory constrained | Path B + Path C pruning for model fit |
| Memory | Latency matters | Path C + vLLM + streaming (accept some latency increase from aggression) |
| Accuracy | Memory constrained | Path D + FP8 only + smaller architecture variant |

---

## 12. Further Reading and Deep Dives

This guide is the umbrella reference for LLM Optimization. Each major topic has a dedicated detailed guide in this repository:

### Core Optimization Deep Dives

| Guide | Focus | Link |
|---|---|---|
| **LLM Model Pruning** | SparseGPT, Wanda, SliceGPT, 2:4 structured pruning, pruning pipelines | [../llm_model_pruning_guide.md](../llm_model_pruning_guide.md) |
| **LLM Compression Alternatives** | Quantization frameworks: GPTQ, AWQ, GGUF, bitsandbytes, AQLM, HQQ, QuIP# — full comparison | [llm_compression_alternatives_guide.md](llm_compression_alternatives_guide.md) |
| **LLM Latency Optimization** | TTFT, TPS, serving engines, speculative decoding, caching, measurement | [llm_latency_optimization_guide.md](llm_latency_optimization_guide.md) |

### Related MLOps / LLMOps Guides

| Guide | Relevance to Optimization | Link |
|---|---|---|
| **MLOps Lifecycle Frameworks** | Model management, CI/CD for ML, deployment pipelines | [../mlops_lifecycle_frameworks_guide.md](../mlops_lifecycle_frameworks_guide.md) |
| **LLM Evaluation Frameworks** | Accuracy measurement, benchmark selection, validation methodology | [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) |
| **Scalable AI Deployment** | Kubernetes GPU serving, auto-scaling, monitoring | [scalable_ai_deployment.md](scalable_ai_deployment.md) |
| **AI Agent Drift** | Drift detection for LLM-based systems, monitoring | [ai_agent_drift_guide.md](ai_agent_drift_guide.md) |
| **Enterprise AI Platforms** | End-to-end platform comparisons | [enterprise_ai_platforms_guide.md](enterprise_ai_platforms_guide.md) |

### External References

- **FlashAttention-2/3:** [https://github.com/Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention)
- **vLLM:** [https://github.com/vllm-project/vllm](https://github.com/vllm-project/vllm)
- **TensorRT-LLM:** [https://github.com/NVIDIA/TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM)
- **llama.cpp:** [https://github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp)
- **Neural Magic LLMCompressor:** [https://github.com/neuralmagic/llmcompressor](https://github.com/neuralmagic/llmcompressor)
- **AutoAWQ:** [https://github.com/casper-hansen/AutoAWQ](https://github.com/casper-hansen/AutoAWQ)
- **SR 11-7 (OCC Model Risk Management):** [https://www.occ.gov/news-issuances/bulletins/2011/bulletin-2011-12.html](https://www.occ.gov/news-issuances/bulletins/2011/bulletin-2011-12.html)
- **Sakana AI Evolutionary Model Merging:** [https://github.com/SakanaAI/evolutionary-model-merge](https://github.com/SakanaAI/evolutionary-model-merge)

---

> **Next Steps:** Identify your primary constraint using the decision tree (Section 11), select the appropriate optimization path, and consult the relevant deep-dive guide for implementation details. For banking-specific deployments, review Section 9 for regulatory and compliance considerations before proceeding.
>
> **Contributing:** This guide is part of the [research](https://github.com/jackliusr/research) repository. Updates, corrections, and additions welcome via PR.
