# Comprehensive Guide to LLM Compression Tools & Alternatives to Neural Magic's LLMCompressor

> **Author:** Jack Liu Shurui  
> **Date:** July 2026  
> **Context:** Solution Architecture — Crédit Agricole CIB, Singapore  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)

---

## Table of Contents

1. [What Is LLM Compression and Why It Matters](#1-what-is-llm-compression-and-why-it-matters)
2. [Neural Magic LLMCompressor Overview](#2-neural-magic-llmcompressor-overview)
3. [Quantization Frameworks](#3-quantization-frameworks)
   - 3.1 [bitsandbytes](#31-bitsandbytes)
   - 3.2 [GPTQ](#32-gptq)
   - 3.3 [AWQ](#33-awq)
   - 3.4 [GGUF / GGML (llama.cpp)](#34-gguf--ggml-llamacpp)
   - 3.5 [AQLM](#35-aqlm)
   - 3.6 [HQQ](#36-hqq)
   - 3.7 [QuIP#](#37-quip)
   - 3.8 [NVIDIA TensorRT-LLM](#38-nvidia-tensorrt-llm)
   - 3.9 [vLLM](#39-vllm)
   - 3.10 [CTranslate2](#310-ctranslate2)
4. [Pruning and Sparsity Frameworks](#4-pruning-and-sparsity-frameworks)
   - 4.1 [SparseGPT](#41-sparsegpt)
   - 4.2 [Wanda](#42-wanda)
   - 4.3 [SliceGPT](#43-slicegpt)
   - 4.4 [LLM-Pruner](#44-llm-pruner)
   - 4.5 [DistilBERT-Style Distillation for LLMs](#45-distilbert-style-distillation-for-llms)
5. [Knowledge Distillation Approaches](#5-knowledge-distillation-approaches)
6. [Model Merging Techniques](#6-model-merging-techniques)
7. [Full-Stack Optimization Frameworks](#7-full-stack-optimization-frameworks)
   - 7.1 [DeepSpeed](#71-deepspeed)
   - 7.2 [Hugging Face Optimum](#72-hugging-face-optimum)
   - 7.3 [ONNX Runtime](#73-onnx-runtime)
   - 7.4 [Apache TVM](#74-apache-tvm)
   - 7.5 [OpenVINO](#75-openvino)
   - 7.6 [TensorFlow Lite / MediaPipe](#76-tensorflow-lite--mediapipe)
   - 7.7 [ExecuTorch (PyTorch Edge)](#77-executorch-pytorch-edge)
8. [Quantization Technique Comparison](#8-quantization-technique-comparison)
9. [Model Format and Deployment Considerations](#9-model-format-and-deployment-considerations)
10. [Tool Selection Guide](#10-tool-selection-guide)
11. [Benchmarks and Comparisons](#11-benchmarks-and-comparisons)
12. [Emerging Trends and Future Directions](#12-emerging-trends-and-future-directions)
13. [Quick-Reference Selection Matrix](#13-quick-reference-selection-matrix)

---

## 1. What Is LLM Compression and Why It Matters

Large Language Models (LLMs) have grown from millions to hundreds of billions of parameters. A single Llama 3 70B model in FP16 requires ~140 GB of GPU memory — more than a single NVIDIA A100-80GB can hold. This creates a fundamental deployment challenge.

LLM compression refers to a family of techniques that reduce the memory footprint, computational cost, and energy consumption of LLMs while preserving as much model quality as possible. The four primary compression levers are:

| Technique | What It Does | Typical Compression Ratio |
|---|---|---|
| **Quantization** | Reduces numerical precision of weights/activations | 2×–8× |
| **Pruning / Sparsity** | Removes redundant weights or neurons | 1.5×–4× |
| **Knowledge Distillation** | Trains a smaller "student" model to mimic a larger "teacher" | 2×–10× |
| **Model Merging** | Combines multiple models without retraining | N/A (orthogonal) |

### Why Compression Matters in Production

1. **Reduced Memory Requirements** — A 70B model compressed to 4 bits needs only ~35 GB, fitting on a single A100 or two RTX 4090s. This dramatically lowers hardware barrier to entry.

2. **Faster Inference** — Lower precision enables faster arithmetic (INT8 adds are ~2× faster than FP16 on modern GPUs) and reduces memory bandwidth pressure, which is the dominant bottleneck in LLM inference (arithmetic intensity is low — the model is memory-bound).

3. **Lower Cost** — Running on fewer or cheaper GPUs directly reduces cloud inference bills. CPU inference via quantization opens the door to using existing commodity servers instead of expensive GPU instances.

4. **Edge and Device Deployment** — Compression is the enabler for running LLMs on smartphones, laptops, IoT devices, and vehicles where GPU memory is limited or unavailable.

5. **Higher Throughput** — Quantized models occupy less memory per request, allowing larger batch sizes and higher throughput in serving infrastructure.

6. **Energy Efficiency** — Lower precision arithmetic consumes less energy per operation, critical for both datacenter sustainability and battery-powered devices.

---

## 2. Neural Magic LLMCompressor Overview

[Neural Magic's LLM Compressor](https://github.com/neuralmagic/llm-compressor) is an open-source library for compressing LLMs, introduced in early 2025. It unifies quantization, pruning, and knowledge distillation in a single framework and integrates tightly with vLLM for deployment.

### Key Capabilities

- **Quantization Formats Supported:**
  - **W4A16** (4-bit weights, 16-bit activations) — best accuracy-efficiency trade-off
  - **W8A16** (8-bit weights, 16-bit activations) — minimal accuracy loss
  - **W8A8** (8-bit weights, 8-bit activations) — maximum throughput on INT8 hardware
  - GPTQ integration for GPU-optimized weight-only quantization

- **Pruning / Sparsity:**
  - Leverages **SparseGPT** for unstructured pruning (up to 50% sparsity with minimal accuracy degradation)
  - Structured sparsity patterns for hardware-efficient inference
  - Supports N:M sparsity (e.g., 2:4 pattern for NVIDIA Ampere+ GPUs)

- **Knowledge Distillation:**
  - Built-in distillation loss during compression to recover accuracy
  - Supports teacher-student training paradigms

- **Framework Integration:**
  - Built on top of **SparseML** (Neural Magic's optimization toolkit)
  - Seamless **vLLM** integration — compressed models load directly
  - HuggingFace Transformers compatible
  - **ONNX export** for cross-platform deployment

- **Hardware Targets:**
  - **CPU:** DeepSparse engine — leverages AVX-512, AVX2, VNNI instructions for CPU-optimized sparse/quantized inference. Achieves GPU-comparable throughput on commodity server CPUs.
  - **GPU:** CUDA-accelerated inference via vLLM backend. Supports NVIDIA Ampere, Hopper, and Blackwell architectures.

### When to Use LLM Compressor

LLM Compressor is an excellent choice when you want a single, integrated workflow from compression to deployment, especially if you are already in the vLLM ecosystem or targeting CPU-based inference with DeepSparse.

### Limitations

- Smaller community and fewer pre-quantized model uploads compared to GGUF or GPTQ ecosystems
- DeepSparse CPU performance, while impressive, still lags behind GPU for latency-sensitive interactive applications
- Primarily focused on Neural Magic's own inference engine; integration with TensorRT-LLM is not native

---

## 3. Quantization Frameworks

### 3.1 bitsandbytes

[bitsandbytes](https://github.com/TimDettmers/bitsandbytes) by Tim Dettmers is the most widely adopted library for low-precision LLM training and inference. It is the default quantization backend for HuggingFace Transformers and PEFT (LoRA/QLoRA).

**Key Features:**
- **4-bit NormalFloat (NF4):** Information-theoretically optimal quantization for normally distributed weights. Better than uniform INT4 for LLM weights.
- **4-bit FP4:** Float-point 4-bit format, useful for weights with wider dynamic range.
- **8-bit INT8:** Per-channel and per-token quantization for inference and training.
- **Double Quantization:** Quantizes the quantization constants themselves, saving additional memory.
- **QLoRA Integration:** Enables fine-tuning 65B models on a single 24 GB GPU via 4-bit base model + LoRA adapters.

**Use Cases:**
- Training and fine-tuning large models on consumer GPUs
- Quick drop-in quantization for existing HuggingFace pipelines
- Research prototyping where ease of use matters most

**Strengths:** Extremely easy to use (load_in_4bit=True), tight HF integration, QLoRA ecosystem.
**Weaknesses:** Not optimized for production serving throughput; GPU-only; no CPU support.

### 3.2 GPTQ

[GPTQ](https://github.com/IST-DASLab/gptq) (Generative Pre-trained Transformer Quantization) by IST-DASLab is a post-training quantization method specifically designed for generative LLMs. It performs one-shot weight quantization using an approximate second-order (Hessian-based) optimization.

**Key Features:**
- **Weight-Only Quantization:** Only weights are quantized; activations remain in FP16
- **Bit Widths:** 2-bit, 3-bit, 4-bit, 8-bit (4-bit is the most common and practical sweet spot)
- **Group Size:** Typically 128 or 32 — smaller groups increase accuracy at the cost of larger lookup tables
- **Calibration Data:** Requires a small calibration dataset (128–256 samples) from typical training data
- **ExLlama V2 Integration:** Highly optimized CUDA kernel for GPTQ models, providing the fastest GPTQ inference on consumer GPUs

**Ecosystem:**
- **AutoGPTQ:** The most popular library for GPTQ quantization and inference
- **ExLlama V2:** The fastest GPTQ inference engine, supports 4-bit and 3-bit
- **HuggingFace Transformers:** Native GPTQ support via `quantization_config` with `GPTQConfig`

**Strengths:** Excellent GPU inference speed with ExLlama V2, large community of pre-quantized models on HuggingFace Hub, mature tooling.
**Weaknesses:** Does not quantize activations (FP16 matmuls hurt throughput on compute-bound scenarios); calibration data required; CPU inference is not a primary use case.

### 3.3 AWQ

[AWQ](https://github.com/mit-han-lab/llm-awq) (Activation-aware Weight Quantization) from MIT HAN Lab is an improved quantization method that identifies "salient" weights (those corresponding to large activations) and protects them by scaling them up before quantization.

**Key Features:**
- **Activation-Aware:** Identifies 1% of weights that are disproportionately important based on activation distributions
- **Per-Group Quantization:** Similar group-size mechanism as GPTQ
- **Bit Widths:** 4-bit (primary), also supports 3-bit and 2-bit
- **Better Accuracy than GPTQ:** At equivalent bit widths, AWQ typically shows 0.5–1 perplexity point improvement over GPTQ on Llama-family models
- **No Backpropagation:** Fast quantization — no gradient computation, just a calibration pass

**Ecosystem:**
- **AutoAWQ:** Primary library for AWQ quantization and inference
- **vLLM:** Native AWQ support with optimized CUDA kernels
- **TGI (Text Generation Inference):** HuggingFace's inference server supports AWQ
- **TensorRT-LLM:** Supports AWQ quantized models

**Strengths:** Superior accuracy-to-compression ratio vs. GPTQ, fast quantization process, good GPU throughput.
**Weaknesses:** Slightly more complex than simple round-to-nearest quantization; smaller pre-quantized model ecosystem than GPTQ/GGUF (but growing fast).

### 3.4 GGUF / GGML (llama.cpp)

[GGUF](https://github.com/ggml-ai/llama.cpp) (GPT-Generated Unified Format) is the quantization format and inference engine behind the llama.cpp project by Georgi Gerganov. It is the de facto standard for on-device and CPU-based LLM inference.

**Key Features:**
- **CPU-First Design:** Highly optimized for CPU inference using SIMD (AVX2, AVX-512, NEON on ARM), no GPU required
- **K-Quant Methods:** Novel quantization types that vary precision per weight-group based on importance:
  - **Q2_K, Q3_K_S/M/L, Q4_K_S/M, Q5_K_S/M, Q6_K, Q8_0**
  - **Q4_K_M:** The most popular "sweet spot" — good accuracy with 4.5 bits per weight effective
  - **Q5_K_M:** Higher quality at ~5.1 bits per weight
  - **Q8_0:** Near-lossless 8-bit quantization
- **Metal Backend:** GPU acceleration on Apple Silicon via Metal
- **CUDA / Vulkan / SYCL:** Growing GPU support across platforms
- **Wide Community Support:** Thousands of pre-quantized models on HuggingFace Hub

**Strengths:** Best CPU inference engine available, massive model ecosystem, runs on virtually any hardware (including Raspberry Pi for small models), active development community.
**Weaknesses:** GPU performance lags behind vLLM/TensorRT-LLM for datacenter deployment; not designed for high-throughput serving with continuous batching (though server backend exists).

### 3.5 AQLM

[AQLM](https://github.com/Vahe1994/AQLM) (Additive Quantization of Language Models) pushes extreme compression to 2 bits per weight while maintaining surprisingly good quality. It was published at ICML 2024 by Yandex Research / HSE University.

**Key Features:**
- **Additive Quantization:** Each weight is represented as a sum of multiple codebook entries (typically 2–8 codebooks at 8–16 bits each)
- **Extreme Bit Widths:** 2-bit compression is the headline capability; also supports 3-bit and 4-bit
- **Input-Adaptive:** Codebooks are optimized with an input-dependent loss (calibration data)
- **Joint Block Optimization:** Optimizes quantization parameters across entire transformer blocks, not per-layer

**Strengths:** Best-in-class quality at extreme 2-bit compression (~8× compression over FP16); pushes the boundary of what's possible at ultra-low precision.
**Weaknesses:** Slow quantization process (can take hours for a 70B model); limited deployment ecosystem (no vLLM or TensorRT-LLM support yet); specialized CUDA kernels required for inference speed.

### 3.6 HQQ

[HQQ](https://github.com/mobiusml/hqq) (Half-Quadratic Quantization) by Mobius Labs / Dropbox is a fast, data-free quantization method that uses half-quadratic splitting to solve the quantization problem.

**Key Features:**
- **No Calibration Data Required:** Quantizes from the weight statistics alone — a unique advantage
- **Extremely Fast:** Quantizes a 70B model in minutes on a single GPU
- **Bit Widths:** Supports 8, 4, 3, 2, and even 1 bit
- **Per-Group Optimization:** Uses half-quadratic optimization to find per-group zero-points and scaling factors
- **Hybrid Packing:** Efficient memory layout for extreme bit widths

**Strengths:** The fastest quantization method available; no need to curate calibration datasets; works on any model architecture.
**Weaknesses:** Quality at 2-bit and 3-bit lags behind AQLM and QuIP#; smaller community and fewer pre-quantized models; inference kernels are less mature than GPTQ or GGUF.

### 3.7 QuIP#

[QuIP#](https://github.com/Cornell-RelaxML/quip-sharp) (Quantization with Incoherence Processing and Lattice Codebooks) from Cornell, published at ICML 2024, achieves state-of-the-art quality at 2–4 bit compression.

**Key Features:**
- **Incoherence Processing:** Applies random orthogonal transforms (Hadamard matrices) to "whiten" the weight distribution, making weights uniformly distributed and easier to quantize
- **Lattice Codebooks (E₈):** Uses the highly symmetric E₈ lattice for vector quantization — the densest sphere packing in 8 dimensions, provably optimal for quantization of whitened Gaussian data
- **Vector Quantization:** Quantizes groups of weights together rather than independently
- **State-of-the-Art Quality:** Typically outperforms GPTQ, AWQ, and AQLM at the same bit width

**Strengths:** Best reported quality at 2-bit quantization; theoretically grounded approach combining incoherence and lattice quantization.
**Weaknesses:** Slower quantization than simple methods; the E₈ codebook requires specialized CUDA kernels; project is no longer under active development (successor is QTIP).

### 3.8 NVIDIA TensorRT-LLM

[TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) is NVIDIA's official toolkit for optimizing and deploying LLMs on NVIDIA GPUs. It is the gold standard for production GPU inference in enterprise datacenters.

**Key Features:**
- **Quantization Support:** FP8 (Transformer Engine on H100+), INT4 (AWQ/GPTQ), INT8 (SmoothQuant, per-channel, per-tensor)
- **FP8 Inference:** Leverages H100/H200/B200 Transformer Engine for native FP8 compute — 2× throughput vs. FP16 with minimal quality loss
- **In-Flight Batching:** Dynamically adds/removes requests from a running batch without padding — maximizes GPU utilization
- **PagedAttention:** Efficient KV cache management inspired by vLLM, built into the TensorRT engine
- **Multi-GPU Multi-Node:** Tensor parallelism, pipeline parallelism, and expert parallelism for MoE models
- **Plugin Architecture:** Custom kernels for attention (FlashAttention-2, MLA), MoE (FasterMoE), and quantization
- **Weight-Stripping:** Removes optimizer states and unnecessary metadata for minimal memory footprint

**Strengths:** Maximum throughput on NVIDIA hardware; production-proven at scale (used by OpenAI, Meta, Google); broad quantization support; enterprise-grade features (health checks, metrics, dynamic batching).
**Weaknesses:** NVIDIA GPU only; complex build pipeline (requires TRT engine compilation); slow iteration on new model architectures; limited community pre-built models vs. HF Transformers.

### 3.9 vLLM

[vLLM](https://github.com/vllm-project/vllm) is a high-throughput LLM serving system developed at UC Berkeley. It has rapidly become the most popular open-source LLM inference server.

**Key Features:**
- **PagedAttention:** Efficient KV cache management that eliminates fragmentation and enables near-100% cache utilization
- **Continuous Batching:** Dynamically schedules requests within and across iteration boundaries
- **Quantization Support:**
  - **AWQ** — native, with hand-tuned CUDA kernels
  - **GPTQ** — via Marlin kernel (fast GPTQ inference)
  - **FP8** — via Transformer Engine on H100
  - **SqueezeLLM** — non-uniform quantization
  - **bitsandbytes** — for on-the-fly quantization
- **Prefix Caching:** Reuses KV cache across requests with shared prefixes (prompt engineering, system prompts)
- **Automatic Prefix Caching (APC):** Redis-based or in-memory KV cache sharing
- **Speculative Decoding:** Draft model + target model for 2–3× latency improvement
- **Multi-LoRA:** Serve multiple fine-tuned LoRA adapters from a single base model
- **OpenAI-Compliant API:** Drop-in replacement for OpenAI chat/completions endpoints

**Strengths:** Best-in-class throughput for open-source serving; broadest quantization support of any serving framework; massive community (100k+ GitHub stars); fast to iterate (new model architectures supported quickly).
**Weaknesses:** GPU-focused (CPU backend is secondary); PagedAttention adds VRAM overhead for KV cache management; less mature than TensorRT-LLM for mission-critical enterprise deployments (though rapidly closing the gap).

### 3.10 CTranslate2

[CTranslate2](https://github.com/OpenNMT/CTranslate2) is an optimized inference engine for transformer models, developed by OpenNMT. While less known than vLLM, it offers exceptional CPU performance.

**Key Features:**
- **INT8 and FP16 Quantization:** Via weight quantization and INT8 gemm
- **CPU-First Optimization:** Uses Intel MKL, oneDNN, and ARM Compute Library for SIMD-optimized inference
- **GPU Support:** CUDA backend with INT8 tensor cores
- **Memory Efficiency:** Automatic weight quantization on model loading
- **Cross-Platform Works:** Linux, macOS, Windows, and mobile (ARM)
- **Client Bindings:** Python, C++, Rust, and .NET

**Strengths:** Excellent CPU inference throughput; lightweight (no heavy dependencies); supports many transformer architectures (not just LLMs, but also BERT, T5, Whisper).
**Weaknesses:** Smaller community; fewer pre-quantized models; llama.cpp/GGUF has largely overtaken it for LLM CPU inference.

---

## 4. Pruning and Sparsity Frameworks

Pruning removes parameters from the model to reduce size and computation. Unlike quantization (which changes precision), pruning changes the structure of the model.

### 4.1 SparseGPT

[SparseGPT](https://github.com/IST-DASLab/sparsegpt) (2023) is a one-shot pruning method that can remove up to 50% of parameters from an LLM without any fine-tuning, with minimal accuracy loss.

**How It Works:**
- Uses an approximate second-order (Hessian-based) optimization to determine which weights to prune
- Prunes weights column-by-column using a closed-form update that compensates for removed weights
- No gradient computation or training loops needed — just a single forward pass for calibration

**Key Results:**
- 50% sparsity on OPT-175B with only 1–2 perplexity point degradation
- 30% sparsity is essentially lossless for most LLMs
- Works on models up to 175B parameters

**Strengths:** True one-shot pruning, no retraining, excellent accuracy retention.
**Weaknesses:** Unstructured sparsity (irregular sparsity pattern) is poorly supported by GPU hardware accelerators; structured sparsity versions exist but are less mature.

### 4.2 Wanda

[Wanda](https://github.com/locuslab/wanda) (Pruning by Weights and Activation Norms) from Princeton (2023) offers a dramatically simpler pruning criterion that achieves comparable results to SparseGPT.

**How It Works:**
- Pruning score = |weight| × ‖activation‖ (per-output)
- Weights with small magnitude AND small input activations are removed first
- No Hessian computation, no iterative weight updates

**Key Results:**
- Comparable to SparseGPT at all sparsity levels (30–60%)
- Significantly faster than SparseGPT (minutes vs. hours for large models)
- Works on the same one-shot paradigm — no fine-tuning

**Strengths:** Simplicity and speed; easy to implement; comparable quality to SparseGPT.
**Weaknesses:** Same unstructured sparsity problem as SparseGPT; slightly lower accuracy at very high sparsity levels (>60%).

### 4.3 SliceGPT

[SliceGPT](https://github.com/microsoft/TransformerCompression) (Microsoft, 2024) takes a fundamentally different approach — instead of zeroing individual weights, it reduces the embedding dimension of the entire model.

**How It Works:**
- Leverages the insight of **computational invariance**: applying an orthogonal transformation to consecutive weight matrices does not change the model's output
- Uses PCA over calibration activations to find a basis where the residual stream has low-variance dimensions
- "Slices" (removes) those low-variance dimensions, reducing every weight matrix's shape
- Result is a smaller **dense** model, not a sparse one

**Key Results:**
- Removes up to 25% of parameters from Llama-2 70B while retaining 99% of zero-shot performance
- Actual speedup on GPUs because the model is genuinely smaller (dense matmuls), not just sparse
- Works on any transformer architecture

**Strengths:** Produces smaller dense models that actually run faster on GPUs; no sparsity hardware support needed.
**Weaknesses:** Compression ratio limited to ~25% before quality degrades significantly; requires recovery fine-tuning (RFT) to retain accuracy at higher ratios.

### 4.4 LLM-Pruner

[LLM-Pruner](https://github.com/horseee/LLM-Pruner) (2023) performs structural pruning of LLMs — removing entire neurons, attention heads, and layers rather than individual weights.

**How It Works:**
- Uses gradient-based importance estimation to identify which structures are redundant
- Prunes coupled groups (neurons + their corresponding weights in downstream layers) to maintain model structure
- Followed by a brief recovery fine-tuning phase

**Key Results:**
- Can remove 20% of parameters with minimal quality loss
- Produces models that are genuinely faster on any hardware (no sparsity support needed)

**Strengths:** Structural pruning yields actual speedup on any hardware; produces smaller model files.
**Weaknesses:** Requires recovery fine-tuning (not one-shot); accuracy drops more sharply than unstructured pruning at high compression ratios.

### 4.5 DistilBERT-Style Distillation for LLMs

Applying the DistilBERT paradigm (knowledge distillation during pre-training or fine-tuning) to LLMs has produced several important model families:

**DeepSpeed Inference:** Microsoft's DeepSpeed ZeRO-Inference integrates structured pruning with INT8 quantization for LLM inference optimizations.

**Knowledge Distillation via Teacher-Student:**
- **Orca (Microsoft, 2023):** Distilled GPT-4's reasoning capabilities into a 13B model via explanation tuning
- **Phi Series (Microsoft, 2023–2025):** Small models (1.3B–14B) trained on textbook-quality synthetic data distilled from larger models. Phi-3-mini (3.8B) competes with Llama-3-8B.
- **Mistral 7B → Various:** Distilled variants like Zephyr (fine-tuned from Mistral 7B using distilled data from larger models)
- **DistilBERT → DistilGPT2 / DistilLLaMA:** Smaller, faster versions of GPT-2 and LLaMA trained via distillation

**Key Advantage:** These models are permanently smaller — the compression is "baked in" during training, requiring no runtime optimization. The trade-off is the high cost of the distillation training run.

---

## 5. Knowledge Distillation Approaches

Knowledge distillation (KD) compresses models by transferring knowledge from a large "teacher" model to a smaller "student" model. This is orthogonal to quantization and pruning — it produces a fundamentally smaller model rather than compressing an existing one.

### Teacher-Student Distillation Frameworks

| Approach | Teacher | Student | Key Technique |
|---|---|---|---|
| **Orca** | GPT-4 (175B+) | 13B | Explanation tuning (chain-of-thought + rationale) |
| **Phi-1/2/3** | GPT-3.5/4 | 1.3B–14B | Textbook-quality synthetic data filtering |
| **Mistral → Zephyr** | GPT-4 | 7B | Direct preference optimization (DPO) from distilled data |
| **DeepSeek-R1 Distill** | DeepSeek-R1 (671B) | 1.5B–70B | Reasoning chain distillation |
| **DistilBERT / DistilGPT2** | BERT/GPT-2 | 40–60% smaller | Logit-level KL divergence + triple loss |

### Dataset Distillation

A complementary approach where the training data itself is compressed:
- **Dataset Condensation:** Generate a small synthetic dataset (e.g., 1% of original) that captures essential information for training
- **Instruction Tuning From Larger Models:** Distilling a large model's output distribution on diverse prompts into a smaller model's training data
- **DSP (Distilled Self-Play):** Student generates outputs, teacher critiques them, student improves from feedback

### Practical Considerations

- **Compute Cost:** Distillation requires running the teacher on massive datasets — often more expensive than PTQ/pruning but pays off in permanently smaller models
- **Architecture Freedom:** Student can be a completely different architecture (e.g., Mamba state-space model distilled from a transformer)
- **Domain Specificity:** Distillation can be targeted — e.g., distill a code-specialized LLM from a general-purpose teacher

---

## 6. Model Merging Techniques

Model merging combines multiple fine-tuned models into a single model without additional training. While not traditional compression, it can reduce deployment complexity by consolidating multiple models.

### Model Soups (Wortsman et al., 2022)
- Simple averaging of weights from models fine-tuned with different hyperparameters
- Improves accuracy without increasing inference cost
- Limitation: Models must share the same base architecture and initialization

### TIES-Merging (Yadav et al., 2023)
- Task-Specific Intervention of Error Signals
- Resolves parameter interference when merging models fine-tuned on different tasks
- Steps: (1) Trim redundant parameters, (2) Elect sign consensus, (3) Disjoint merge
- Enables merging models specialized in different domains (e.g., coding + math + creative writing)

### DARE (Yu et al., 2023)
- Drop And REscale — randomly drops a large fraction (90–99%) of delta parameters before merging
- Rescales remaining deltas to maintain expected output
- Enables merging 10+ models simultaneously

### Evolutionary Model Merging (Sakana AI, 2024)
- Uses evolutionary algorithms to find optimal merge coefficients
- Merges models across architectures and tasks using gradient-free optimization
- Produced the first "merged model" that outperformed individual constituent models across all benchmarks
- Automates the search for merge configurations (layer-wise, non-uniform mixing)

### Relevance to Compression
Model merging is an **orthogonal** technique — it doesn't reduce a single model's size but can replace multiple specialized models with one merged model, reducing total deployed footprint. It is often combined with quantization (quantize the merged model) for maximum efficiency.

---

## 7. Full-Stack Optimization Frameworks

### 7.1 DeepSpeed

[DeepSpeed](https://github.com/microsoft/DeepSpeed) by Microsoft is a comprehensive deep learning optimization library that spans training, inference, and compression.

**Key Features:**
- **ZeRO Optimization (ZeRO-1/2/3):** Shards optimizer states, gradients, and parameters across GPUs. Enables training models with trillions of parameters.
- **ZeRO-Offload:** Offloads optimizer states and parameters to CPU/NVMe memory, enabling fine-tuning of 175B+ models on a single GPU.
- **ZeRO-Inference:** Inference-time memory optimization with offloading and INT8 quantization.
- **Mixture of Experts (MoE):** DeepSpeed-MoE trains and serves MoE models efficiently.
- **Inference Optimization:** Kernel fusion, INT8 quantization, and custom CUDA kernels.
- **Automatic Tensor Parallelism:** No need to manually shard models.

**Strengths:** Unmatched for large-model training; ZeRO-Offload enables fine-tuning beyond GPU memory limits; MoE support.
**Weaknesses:** Complex configuration; primarily a training library with inference as a secondary feature; less mature serving stack than vLLM/TensorRT-LLM.

### 7.2 Hugging Face Optimum

[Optimum](https://github.com/huggingface/optimum) is HuggingFace's extension for hardware-specific optimizations. It provides a unified API over multiple optimization backends.

**Key Features:**
- **Intel NNCF:** Neural Network Compression Framework for quantization, pruning, and distillation on Intel hardware
- **ONNX Runtime:** Graph optimization, INT8/FP16 quantization, and CPU/GPU execution
- **GPTQ & AWQ:** Integrates with AutoGPTQ and AutoAWQ for weight-only quantization
- **Habana Gaudi:** Optimizations for Intel Habana AI accelerators
- **AMD ROCm:** GPU acceleration for AMD hardware
- **BetterTransformer:** Fast path execution using PyTorch native scaled_dot_product_attention

**Strengths:** Unified API across hardware vendors; works seamlessly with HuggingFace ecosystem; continuous updates.
**Weaknesses:** Abstraction layer means less control than vendor-specific toolkits; some features are backend-specific and not fully portable.

### 7.3 ONNX Runtime

[ONNX Runtime](https://github.com/microsoft/onnxruntime) is Microsoft's cross-platform inference engine supporting the Open Neural Network Exchange (ONNX) format.

**Key Features:**
- **Graph Optimization:** Operator fusion, constant folding, layout optimization
- **Quantization:** INT8 dynamic/static quantization, FP16, INT4 support via `onnxruntime.quantization`
- **CPU Backend:** Optimized with MLAS (Microsoft Linear Algebra Subroutines)
- **GPU Backend:** CUDA, TensorRT, and DirectML execution providers
- **Transformer Optimization:** Specialized attention fusion and rotary embedding optimization

**Strengths:** Cross-platform (Windows, Linux, macOS, mobile); wide hardware support; mature and battle-tested.
**Weaknesses:** ONNX conversion of LLMs is non-trivial (large model size, dynamic shapes); limited support for latest LLM architectures (Mamba, RWKV); slower iteration than HF Transformers.

### 7.4 Apache TVM

[Apache TVM](https://github.com/apache/tvm) is an open-source ML compiler framework for optimizing models on diverse hardware targets.

**Key Features:**
- **Auto-Tuning:** Uses ML-based search (AutoTVM, Ansor) to find optimal kernel configurations for target hardware
- **Quantization:** Automatic INT8/FP16 quantization with calibration
- **Hardware Targets:** CPU (x86, ARM), GPU (CUDA, ROCm, Metal, Vulkan), FPGA, and custom accelerators
- **Relay IR:** Graph-level optimizations before code generation
- **BYOC (Bring Your Own Codegen):** Extensible to new hardware backends

**Strengths:** Hardware-agnostic with excellent auto-tuning; can optimize for non-standard hardware; generates efficient code for edge devices.
**Weaknesses:** High learning curve; compilation time can be long for large models; LLM-specific optimizations (PagedAttention, FlashAttention) require custom implementation.

### 7.5 OpenVINO

[OpenVINO](https://github.com/openvinotoolkit/openvino) is Intel's toolkit for optimizing and deploying deep learning models on Intel hardware.

**Key Features:**
- **Model Compression:** INT8/FP16 quantization with accuracy-aware algorithms, pruning, and filter pruning
- **Post-Training Optimization:** Uses Intel NNCF (Neural Network Compression Framework)
- **Hardware Targets:** CPU (Intel Xeon, Core), GPU (Intel Iris, Arc), VPU (Myriad), FPGA
- **Heterogeneous Execution:** Automatically distributes workloads across available compute devices
- **ONNX / TensorFlow / PyTorch Import:** Supports conversion from major frameworks
- **OpenVINO Tokenizers:** Text preprocessing and tokenizer support for LLM pipelines

**Strengths:** Best inference performance on Intel hardware; extensive optimization tooling; good for CPU-only deployments.
**Weaknesses:** Intel-centric — no first-class support for NVIDIA or AMD GPUs; model conversion can be brittle; smaller community than ONNX Runtime.

### 7.6 TensorFlow Lite / MediaPipe

[TensorFlow Lite](https://www.tensorflow.org/lite) and [MediaPipe](https://mediapipe.dev) are Google's frameworks for on-device machine learning.

**Key Features:**
- **TFLite Quantization:** INT8 dynamic/static quantization, FP16, and weight quantization
- **MediaPipe Tasks:** Pre-built LLM inference pipeline for Android/iOS
- **GPU Delegate:** OpenGL ES and Vulkan acceleration on mobile
- **XNNPACK Delegate:** CPU-optimized for ARM (mobile CPUs)
- **On-Device Fine-Tuning:** Limited support for on-device adaptation

**Strengths:** Battle-tested on billions of mobile devices; hardware acceleration on mobile GPUs/NPUs; good for small LLMs (up to 3–7B on high-end devices).
**Weaknesses:** LLM support is limited compared to dedicated frameworks; constrained to smaller models; int8 dynamic range quantization loses accuracy vs. more advanced methods.

### 7.7 ExecuTorch (PyTorch Edge)

[ExecuTorch](https://github.com/pytorch/executorch) is PyTorch's official solution for on-device inference across mobile, embedded, and edge devices.

**Key Features:**
- **Native PyTorch Integration:** Export models directly from PyTorch using `torch.export()` with full-graph capture
- **Quantization:** INT8 per-channel quantization, INT4/INT8 weight-only quantization
- **Memory Budgeting:** Runs with as little as 1 MB RAM for the runtime
- **Backend Delegation:** Qualcomm Hexagon DSP, Arm Ethos NPU, Apple Neural Engine, MediaTek APU
- **Operator Coverage:** Supports LLM-specific ops (rotary embedding, RMSNorm, SiLU, GQA, KV cache)
- **Cross-Platform:** Android, iOS, embedded Linux, and microcontrollers

**Strengths:** Modern, PyTorch-native approach; LLM-specific optimizations built-in; active development with broad industry adoption (Apple, Qualcomm, Meta).
**Weaknesses:** Still maturing (1.x releases); smaller community than TFLite; limited pre-built model zoo for LLMs.

---

## 8. Quantization Technique Comparison

### 8.1 Weight-Only vs. Weight+Activation Quantization

| Aspect | Weight-Only | Weight+Activation |
|---|---|---|
| Quantizes | Weights only | Weights + activations (KV cache, intermediate) |
| Typical Bits | W4A16, W3A16, W2A16 | W8A8, W4A8, INT8, FP8 |
| Speedup | ~2–4× (memory-bound ops) | ~2–4× plus compute-bound speedup |
| Accuracy Impact | Lower impact | Higher impact (activations have outliers) |
| Hardware Support | Universal (just dequantize) | Requires INT8/FP8 hardware support |
| Examples | GPTQ, AWQ, GGUF, AQLM | SmoothQuant, TensorRT-LLM FP8 |
| Best For | Memory-bound inference | Compute-bound inference (large batch) |

### 8.2 Per-Tensor vs. Per-Channel vs. Per-Group Quantization

| Method | Granularity | Overhead | Accuracy | Examples |
|---|---|---|---|---|
| Per-Tensor | Single scale/zero-point for entire tensor | Minimal | Lowest | Simple INT8 |
| Per-Channel | Scale/zero-point per output channel | Low | Good | bitsandbytes INT8 |
| Per-Group | Scale/zero-point per group (e.g., 128 weights) | Moderate | Better | GPTQ, AWQ, GGUF K-quant |
| Per-Vector | Sub-channel groups (e.g., 32 or 64) | Higher | Best | AQLM codebooks, QuIP# |

**Rule of Thumb:** Smaller group size = better accuracy + more storage overhead for quantization parameters. Group size 128 is the most common practical trade-off.

### 8.3 Symmetric vs. Asymmetric Quantization

| Type | Range | Complexity | When to Use |
|---|---|---|---|
| Symmetric | [-127, 127] for INT8 | Simpler (no zero-point) | Weights (symmetric around 0) |
| Asymmetric | [0, 255] or [min, max] | Requires zero-point | Activations (often skewed) |

- **Symmetric quantization** is standard for weights since pretrained weights are roughly symmetric around zero.
- **Asymmetric quantization** is preferred for activations which can have skewed distributions (especially in the presence of outliers).

### 8.4 Dynamic vs. Static Quantization

| Aspect | Dynamic Quantization | Static Quantization |
|---|---|---|
| Calibration | On-the-fly (per input) | Pre-computed with calibration data |
| Scale/ZP | Computed at runtime | Fixed from calibration |
| Accuracy | Better (per-input adaptation) | Slightly worse |
| Latency | Higher (compute scales each time) | Lower (scales are baked in) |
| Use Case | Variable input distributions | Production serving (fixed) |

### 8.5 NF4 vs. FP4 vs. INT4 vs. INT8

| Format | Bits | Distribution Fit | Accuracy | Hardware Support |
|---|---|---|---|---|
| INT8 | 8 | Uniform | Near-lossless | Universal (GPU, CPU, NPU) |
| INT4 | 4 | Uniform | Good | Limited (TensorRT-LLM, GGUF) |
| NF4 | 4 | Normal (information-theoretic) | Best for weights | CUDA (bitsandbytes) |
| FP4 | 4 | Floating point (wide range) | Better for outliers | H100 FP8 (no native FP4) |
| FP8 (E4M3/E5M2) | 8 | Floating point | Lossless | H100/H200/B200 native |

**NF4** (NormalFloat4) is theoretically optimal for normally distributed data — LLM weights closely follow a normal distribution, making NF4 the best 4-bit format for weights.

### 8.6 Calibration Dataset Impact

| Factor | Effect |
|---|---|
| **Dataset Size** | 128–512 samples is typically sufficient. More data rarely helps beyond this threshold. |
| **Dataset Quality** | In-domain data (training data distribution) is best. GPTQ/AWQ use Wikipedia, C4, or custom curated data. |
| **Sequence Length** | 2048 tokens is standard. Shorter sequences may miss long-range weight interactions. |
| **Random vs. Selected** | Random samples from training data work as well as carefully curated sets for most methods. |

**Key Insight:** Most quantization methods are robust to calibration data choice. Using 128 random samples from the pre-training corpus is nearly as good as 10,000 carefully selected samples.

### 8.7 Accuracy vs. Compression Trade-off (Approximate)

```
Compression Ratio
     8×  │                    AQLM (2-bit)
     6×  │              QuIP# (2-bit)  AQLM (3-bit)
     5×  │         AWQ (3-bit)
     4×  │    GGUF Q4_K_M        GPTQ (4-bit)
     3×  │    AWQ (4-bit)
     2×  │ GGUF Q8_0  INT8
     1×  │ FP16 (baseline)
         └─────────────────────────────────────→ Perplexity Delta (↑ = worse)
              0     +0.5    +1.0    +2.0    +4.0
```

*Approximate relationships. Actual results vary by model family, bit width, and evaluation benchmark.*

---

## 9. Model Format and Deployment Considerations

### Model Format Overview

| Format | Primary Framework | Typical Use Case | Compression |
|---|---|---|---|
| **HF Transformers (safetensors)** | HuggingFace Transformers | Research, training, flexible deployment | None (FP16/FP32) |
| **GGUF** | llama.cpp | CPU inference, on-device, Apple Silicon | K-quant (Q4, Q5, Q8) |
| **ONNX** | ONNX Runtime | Cross-platform production, enterprise | INT8, FP16 |
| **TensorRT Engine** | TensorRT-LLM | NVIDIA GPU datacenter | INT4, INT8, FP8 |
| **OpenVINO IR** | OpenVINO | Intel CPU/GPU deployment | INT8, FP16 |
| **CoreML** | Apple CoreML | Apple Silicon (iOS/macOS) | INT8, FP16 |
| **MLX** | MLX Framework | Apple Silicon (research) | FP16, 4-bit |
| **ExecuTorch (.pte)** | ExecuTorch | Edge, mobile, embedded | INT8, INT4 |

### Format Conversion Tools

| From → To | Tool | Notes |
|---|---|---|
| HF → GGUF | `convert.py` in llama.cpp | Requires HF model download |
| HF → ONNX | `optimum-cli export onnx` | May need `--trust-remote-code` |
| HF → TensorRT | `trtllm-build` | Complex, requires build script |
| HF → OpenVINO | `optimum-cli export openvino` | HuggingFace Optimum integration |
| Any → CoreML | `coremltools` | Requires PyTorch → CoreML conversion |
| Any → ExecuTorch | `executorch/examples/models/` scripts | Model-specific export scripts |

### Deployability Comparison

| Factor | HF Transformers | GGUF | ONNX | TensorRT | OpenVINO |
|---|---|---|---|---|---|
| **Ease of Setup** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| **GPU Performance** | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **CPU Performance** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | N/A | ⭐⭐⭐⭐⭐ |
| **Community Support** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Model Availability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| **Enterprise Features** | ⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## 10. Tool Selection Guide

### By Hardware Target

| Hardware | Recommended Tools | Why |
|---|---|---|
| **NVIDIA GPU (Datacenter: A100/H100/B200)** | TensorRT-LLM + FP8/INT4 | Maximum throughput, native hardware support |
| **NVIDIA GPU (Consumer: RTX 3090/4090/5090)** | vLLM + AWQ/GPTQ | Balanced performance, easy setup, large ecosystem |
| **NVIDIA GPU (Low VRAM: RTX 3060/4060)** | GGUF + llama.cpp + CUDA offload | Flexible VRAM management, partial GPU offload |
| **CPU (Server: Xeon, EPYC)** | GGUF + llama.cpp OR DeepSparse | Best CPU inference; DeepSparse for high-throughput batch, GGUF for general use |
| **CPU (Consumer/Laptop)** | GGUF + llama.cpp | Most compatible, excellent SIMD optimization |
| **Apple Silicon (M1/M2/M3/M4)** | MLX OR GGUF + Metal | MLX is Apple-native; GGUF with Metal backend is more mature |
| **Mobile (Android/iOS)** | ExecuTorch OR TFLite | ExecuTorch is PyTorch-native; TFLite for wider device support |
| **Intel GPU (Arc, Iris)** | OpenVINO | Intel's optimized stack for their hardware |
| **AMD GPU** | vLLM (ROCm) OR ONNX Runtime (ROCm) | vLLM has best AMD support; or use HuggingFace + ROCm |

### By Deployment Scenario

| Scenario | Recommended Stack | Rationale |
|---|---|---|
| **Cloud GPU Serving (production)** | vLLM + AWQ/GPTQ (cost-sensitive) or TensorRT-LLM (performance-max) | vLLM for flexibility + cost; TRT-LLM for maximum throughput |
| **Self-Hosted API (single GPU)** | vLLM + AWQ (4-bit) | Fits 70B on one A100-80GB |
| **On-Device / Desktop** | GGUF + llama.cpp (Q4_K_M) | Runs anywhere, minimal setup |
| **Enterprise GPU Cluster** | NVIDIA TensorRT-LLM + FP8 | Production-proven at scale, support contracts available |
| **Cost-Sensitive CPU Inference** | DeepSparse (W8A8) or GGUF (Q5_K_M) | Avoids GPU costs entirely |
| **Edge / IoT** | ExecuTorch (INT8) or TFLite | Minimal memory, ARM optimization |
| **Research Prototyping** | bitsandbytes (NF4) or AutoGPTQ | Fast iteration, HF ecosystem |
| **Mobile App** | ExecuTorch (LLM delegate) | On-device inference, no cloud round-trip |

### By Optimization Goal

| Goal | Best Approach | Second Choice |
|---|---|---|
| **Maximum Compression, Minimal Accuracy Loss** | AQLM (2-bit) or QuIP# (2-bit) | AWQ (3-bit) |
| **Easy Integration (existing pipelines)** | bitsandbytes (load_in_4bit=True) | AutoGPTQ / AutoAWQ |
| **Best CPU Performance** | GGUF + llama.cpp (Q4_K_M or Q5_K_M) | CTranslate2 (INT8) |
| **Best GPU Throughput** | TensorRT-LLM (FP8) | vLLM (AWQ) |
| **Lowest Latency (interactive)** | TensorRT-LLM (FP8, single batch) | vLLM (AWQ + chunked prefill) |
| **Smallest Model File Size** | GGUF (Q2_K) or AQLM (2-bit) | HQQ (2-bit) |
| **Quickest Compression Setup** | HQQ (no calibration) | bitsandbytes (HF native) |

---

## 11. Benchmarks and Comparisons

### Perplexity vs. Model Size (Approximate, Llama-2 7B)

*Note: Exact numbers vary by model and evaluation dataset. These are representative ranges from published papers.*

| Method | Bits/Weight | Wiki PPL (↓) | Model Size | Δ vs. FP16 |
|---|---|---|---|---|
| FP16 (baseline) | 16 | 5.47 | 13.5 GB | — |
| INT8 | 8 | 5.48 | 7.0 GB | +0.01 |
| GGUF Q8_0 | 8 | 5.48 | 7.0 GB | +0.01 |
| GGUF Q5_K_M | ~5.1 | 5.51 | 4.8 GB | +0.04 |
| GGUF Q4_K_M | ~4.5 | 5.56 | 4.2 GB | +0.09 |
| GPTQ (g128) | 4 | 5.53 | 4.1 GB | +0.06 |
| AWQ (g128) | 4 | 5.50 | 4.1 GB | +0.03 |
| NF4 | 4 | 5.54 | 4.0 GB | +0.07 |
| GGUF Q3_K_M | ~3.5 | 5.92 | 3.5 GB | +0.45 |
| GPTQ (g128, 3-bit) | 3 | 5.88 | 3.1 GB | +0.41 |
| AQLM (2-bit) | 2 | 6.31 | 1.8 GB | +0.84 |
| QuIP# (2-bit) | 2 | 6.20 | 1.8 GB | +0.73 |

### Inference Speed (Tokens/Second)

| Method | Hardware | Batch=1 | Batch=16 | Batch=64 |
|---|---|---|---|---|
| TensorRT-LLM FP16 | A100-80GB | 120 t/s | 800 t/s | 1,800 t/s |
| TensorRT-LLM FP8 | H100-80GB | 200 t/s | 1,500 t/s | 3,500 t/s |
| vLLM + AWQ (4-bit) | A100-80GB | 95 t/s | 650 t/s | 1,400 t/s |
| vLLM + GPTQ (4-bit) | A100-80GB | 85 t/s | 550 t/s | 1,200 t/s |
| vLLM FP16 | A100-80GB | 70 t/s | 400 t/s | 900 t/s |
| GGUF Q4_K_M + llama.cpp | Intel Xeon (56c) | 25 t/s | — | — |
| GGUF Q4_K_M + Metal | M3 Max | 40 t/s | — | — |
| DeepSparse W8A8 | Intel Xeon (56c) | 35 t/s | — | — |

*Note: Batch>1 is typically used for throughput optimization in production; batch=1 represents interactive latency.*

### Memory Usage (Llama-3 70B)

| Method | GPU Memory Required | Notes |
|---|---|---|
| FP16 (no optimization) | ~140 GB | Requires 2× A100-80GB |
| FP8 (TensorRT-LLM) | ~75 GB | Fits on 1× A100-80GB or H100 |
| INT4 AWQ/GPTQ (4-bit) | ~40 GB | Fits on 1× A100-80GB with room for KV cache |
| GGUF Q4_K_M | ~38 GB | CPU RAM, not GPU |
| AQLM (2-bit) | ~22 GB | 2-bit extreme compression |
| QuIP# (2-bit) | ~22 GB | Comparable to AQLM |

### Calibration and Runtime Overhead

| Method | Calibration Time (7B) | Calibration Data | Runtime Overhead |
|---|---|---|---|
| bitsandbytes NF4 | Instant (no calibration) | None | Per-token dequantization |
| GPTQ | 10–30 min | 128 samples | Minimal (dequant matmul kernel) |
| AWQ | 5–15 min | 128 samples | Minimal (scale reparameterization) |
| GGUF (K-quant) | 10–60 min (conversion) | 256 samples | None (native format) |
| AQLM | 2–8 hours | 1024+ samples | Specialized kernel required |
| HQQ | 1–5 min | None | Fast dequant kernel |
| QuIP# | 1–4 hours | 256 samples | E₈ lattice kernel required |
| TensorRT-LLM (FP8) | 10–30 min (build) | Calibration required | None (native FP8) |

---

## 12. Emerging Trends and Future Directions

### 12.1 FP8 Training and Inference Standardization

- **H100/H200/B200 native FP8 support** (Transformer Engine) makes FP8 the new standard for high-performance LLM inference
- FP8 offers near-lossless quality (~FP16 parity) with 2× throughput improvement
- All major frameworks (TensorRT-LLM, vLLM, PyTorch) now support FP8
- **E4M3** (4 exponent, 3 mantissa) for weights and **E5M2** (5 exponent, 2 mantissa) for gradients/activations
- Expected to become the default precision for cloud GPU serving by 2027

### 12.2 Mixture of Experts (MoE) Compression

- MoE models (Mixtral 8×7B, DeepSeek-V2/V3, Qwen2.5-MoE) route tokens to a subset of experts per forward pass
- Compression techniques for MoE:
  - **Expert Pruning:** Removing less-used experts entirely
  - **Expert Quantization:** Applying lower precision to less important experts
  - **Expert Merging:** Merging similar experts to reduce total expert count
  - **Load-Balancing-Aware Quantization:** Quantizing experts based on their routing frequency
- DeepSeek-V3 (671B total, 37B active) demonstrates the efficiency of the MoE approach
- Future MoE models will likely ship pre-quantized from the factory

### 12.3 1.58-Bit Ternary Quantization (BitNet)

- **BitNet b1.58** (Microsoft, 2024): Weights constrained to {−1, 0, +1} — only 1.58 bits per weight (log₂3)
- Eliminates floating-point multipliers entirely — inference uses only integer addition
- Custom accelerators could skip zero-weight multiplications entirely
- **BitLinear** replaces `nn.Linear` in transformer architectures
- Comparable performance to FP16 LLaMA at 2.7B scale in initial results
- **Limitations:** Requires training from scratch (not post-training quantization); quality gap at larger scales still being investigated
- If successful, this represents a paradigm shift — LLMs become adder networks, not multiply-accumulate networks

### 12.4 On-Device LLM Optimization Growth

- Apple Intelligence, Google AI Core, Samsung Galaxy AI driving massive investment in on-device LLMs
- Models shrinking to 1–3B while maintaining useful capability
- Key optimizations specific to mobile:
  - **NPU Delegation:** Offload to dedicated neural processing units (Apple Neural Engine, Qualcomm Hexagon, Google TPU)
  - **Memory-Constrained Inference:** Shared memory between CPU/GPU/NPU, page-based swapping
  - **Pre-Fill on Server, Decode on Device:** Hybrid cloud-edge inference
  - **Speculative Decoding on Device:** Small on-device model verifies/speculates for larger cloud model
- ExecuTorch and MediaPipe are the leading deployment frameworks for this trend

### 12.5 Quantization-Aware Training (QAT) for LLMs

- Traditional QAT (simulating quantization during training) is compute-intensive but yields best accuracy
- **LLM-QAT** (Liu et al., 2024): Data-free distillation + QAT for LLMs — uses the LLM itself to generate training data
- **QuaRot** (2024): Rotates weights before quantization during training for better spherical distribution
- Emerging approach: **QAT as fine-tuning** — start from a PTQ checkpoint and do a brief QAT fine-tuning pass to recover accuracy
- Expected to become standard practice for production-quantized models

### 12.6 QLoRA and Fine-Tuning with Quantization

- QLoRA demonstrated that 4-bit base models + LoRA adapters can match full fine-tuning quality
- Maturation of the QLoRA ecosystem:
  - **Unsloth:** 2× faster QLoRA training with manual kernel optimization
  - **PEFT + bitsandbytes:** Default stack for parameter-efficient fine-tuning
  - **GPTQ-LoRA:** Fine-tuning with GPTQ quantization
  - **AWQ-LoRA:** LoRA adapters for AWQ-quantized base models
- Enables fine-tuning entire 70B models on a single RTX 4090 (24 GB)
- Production pattern: Quantize base model → fine-tune with LoRA → merge/export for deployment

### 12.7 Model Merging as Compression

- Model merging has emerged as an orthogonal compression technique with zero inference cost overhead
- **Evolutionary merging** (Sakana AI) can discover optimal weight combinations automatically
- Merging + quantization pipeline:
  1. Fine-tune multiple small specialized models
  2. Merge into one multi-skill model (no size increase)
  3. Quantize the merged model
- This pipeline produces a single compact model that outperforms individual specialized models
- Active research area: **task arithmetic** for composing and decomposing skills

### 12.8 Other Notable Directions

- **NVIDIA TensorRT-Model-Optimizer:** Automated quantization calibration and optimization pipeline
- **Pruna AI:** Commercial compression-as-a-service combining multiple techniques automatically
- **Apache TVM Relax:** Next-generation IR for LLM optimization with dynamic shape support
- **Paged Quantization:** Combining PagedAttention-style memory management with per-page quantization
- **Lossy vs. Lossless Compression:** Lightweight entropy coding (arithmetic coding, LZ-style) on quantized weights for storage savings (10–15% additional compression with no accuracy loss)

---

## 13. Quick-Reference Selection Matrix

| Tool / Method | Bits Supported | Hardware Target | Key Strength | Best Use Case |
|---|---|---|---|---|
| **Neural Magic LLM Compressor** | W4A16, W8A16, W8A8 | CPU (DeepSparse), GPU (vLLM) | Integrated quantize+prune+distill pipeline | End-to-end compression with vLLM deployment |
| **bitsandbytes (NF4)** | 4, 8 | NVIDIA GPU | Drop-in HF integration, QLoRA ecosystem | Research, fine-tuning, quick eval |
| **GPTQ (AutoGPTQ)** | 2, 3, 4, 8 | NVIDIA GPU (ExLlama V2) | Fast GPU inference, largest model zoo | Production inference on consumer GPUs |
| **AWQ (AutoAWQ)** | 2, 3, 4 | NVIDIA GPU | Best accuracy at 4-bit, vLLM native | Production serving with vLLM |
| **GGUF (llama.cpp)** | Q2–Q8, K-quant | CPU, Apple Silicon, GPU partial | Best CPU inference, massive ecosystem | On-device, desktop, CPU-only deployments |
| **AQLM** | 2, 3, 4 | GPU (specialized) | Best quality at 2-bit | Extreme compression (8×) |
| **HQQ** | 1, 2, 3, 4, 8 | GPU | No calibration needed, fast | Quick compression without data |
| **QuIP# / QTIP** | 2, 3, 4 | GPU (specialized) | SOTA 2-bit quality via lattice codes | Highest possible quality-per-bit |
| **TensorRT-LLM** | INT4, INT8, FP8 | NVIDIA GPU (A100/H100/B200) | Maximum GPU throughput | Enterprise datacenter GPU serving |
| **vLLM** | AWQ, GPTQ, FP8 | NVIDIA, AMD GPU | Best throughput + flexibility | Open-source production serving |
| **CTranslate2** | INT8, FP16 | CPU, GPU | Fast CPU transformer inference | Lightweight CPU deployment |
| **DeepSpeed** | INT8, FP16 | NVIDIA GPU (multi-node) | ZeRO optimization, MoE | Large-model training + inference |
| **ONNX Runtime** | INT8, FP16 | CPU, GPU, mobile | Cross-platform standard | Enterprise multi-hardware deployment |
| **OpenVINO** | INT8, FP16 | Intel CPU, GPU, VPU | Best Intel inference | Intel-only hardware stacks |
| **Apache TVM** | INT8, FP16 | CPU, GPU, edge, custom | Auto-tuning for any hardware | Non-standard hardware targets |
| **ExecuTorch** | INT8, INT4 | Mobile, edge, embedded | PyTorch-native edge inference | On-device LLM (Android/iOS) |
| **TFLite / MediaPipe** | INT8, FP16 | Mobile, embedded | Google ecosystem, billions of devices | Mobile apps, Google ecosystem |
| **MLX** | 4, 8, FP16 | Apple Silicon | Apple-native framework | Apple Silicon ML research |
| **SparseGPT** | N/A (sparsity) | GPU (unstructured) | 50% sparsity, no retraining | Research, storage compression |
| **Wanda** | N/A (sparsity) | GPU (unstructured) | Simpler than SparseGPT | Fast sparsity exploration |
| **SliceGPT** | N/A (structural) | GPU (dense speedup) | Smaller dense models | Actual GPU speedup from pruning |
| **Model Merging (TIES/DARE/Sakana)** | N/A (merging) | Any | Combine skills, zero inference cost | Consolidating multiple models |

---

## Appendix: Key Libraries and Commands

### Installation Quick Reference

```bash
# Neural Magic LLM Compressor
pip install llmcompressor

# bitsandbytes (for QLoRA)
pip install bitsandbytes

# AutoGPTQ
pip install auto-gptq

# AutoAWQ
pip install autoawq

# llama.cpp (GGUF) — build from source for best performance
git clone https://github.com/ggml-ai/llama.cpp
cd llama.cpp && make -j

# vLLM
pip install vllm

# TensorRT-LLM — NVIDIA container recommended
# See https://github.com/NVIDIA/TensorRT-LLM

# HQQ
pip install hqq

# AQLM
pip install aqlm

# HuggingFace Optimum
pip install optimum[onnxruntime]
```

### Quick Quantization Commands

```python
# bitsandbytes — load any model in 4-bit
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
quant_config = BitsAndBytesConfig(load_in_4bit=True, bnb_4bit_quant_type="nf4")
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.1-8B", quantization_config=quant_config)

# AutoGPTQ — quantize a model
from auto_gptq import AutoGPTQForCausalLM
model = AutoGPTQForCausalLM.from_quantized("model-path", use_marlin=True)

# AutoAWQ — quantize a model
from awq import AutoAWQForCausalLM
model = AutoAWQForCausalLM.from_quantized("model-path", fuse_layers=True)

# HQQ — quantize without calibration
from hqq.models.hf import HQQModelForCausalLM
model = HQQModelForCausalLM.from_pretrained("meta-llama/Llama-3.1-8B")
model.quantize_model(quant_config={"nbits": 4, "group_size": 64})
```

---

## References

1. Frantar et al., "GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers" (2023)
2. Lin et al., "AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration" (2024)
3. Gerganov et al., "llama.cpp" — https://github.com/ggml-ai/llama.cpp
4. Dettmers et al., "QLoRA: Efficient Finetuning of Quantized Language Models" (2023)
5. Egiazarian et al., "Extreme Compression of Large Language Models via Additive Quantization" (ICML 2024)
6. Tseng et al., "QuIP#: Even Better LLM Quantization with Hadamard Incoherence and Lattice Codebooks" (ICML 2024)
7. Frantar and Alistarh, "SparseGPT: Massive Language Models Can Be Accurately Pruned in One-Shot" (2023)
8. Sun et al., "A Simple and Effective Pruning Approach for Large Language Models" (Wanda, 2023)
9. Ashkboos et al., "SliceGPT: Compress Large Language Models by Deleting Rows and Columns" (2024)
10. Ma et al., "BitNet: Scaling 1-bit Transformers for Large Language Models" (2024)
11. Wang et al., "BitNet b1.58: 1.58-bit LLM" (Microsoft, 2024)
12. NVIDIA, "TensorRT-LLM" — https://github.com/NVIDIA/TensorRT-LLM
13. Kwon et al., "Efficient Memory Management for Large Language Model Serving with PagedAttention" (vLLM, 2023)
14. Rajbhandari et al., "ZeRO: Memory Optimizations Toward Training Trillion Parameter Models" (DeepSpeed, 2020)
15. Badri and Shaji, "HQQ: Half-Quadratic Quantization of Large Machine Learning Models" (2023)
16. Wortsman et al., "Model Soups: Averaging Weights of Multiple Fine-tuned Models Improves Accuracy" (2022)
17. Yadav et al., "TIES-Merging: Resolving Interference When Merging Models" (2023)
18. Yu et al., "DARE: Large-Scale Model Merging via Drop And REscale" (2023)
19. Sakana AI, "Evolutionary Model Merging" (2024)
20. PyTorch Team, "ExecuTorch" — https://github.com/pytorch/executorch

---

*This guide is maintained as part of the [research repository](https://github.com/jackliusr/research) by Jack Liu Shurui.  
Last updated: July 2026.*
