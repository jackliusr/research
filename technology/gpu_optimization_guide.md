# GPU Optimization: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** Systems / Performance Engineering — GPU & Parallel Computing, CUDA, AI Infrastructure, Banking Technology  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---
> A comprehensive guide to the engineering discipline of making GPU workloads fast — covering the hardware fundamentals (SMs, warps, the memory hierarchy, tensor cores), the CUDA programming model, the optimization techniques (coalescing, occupancy, tiling, fusion, streams), the profiling tooling (Nsight Compute/Systems), framework-level optimization (torch.compile, CUDA graphs, mixed precision, TensorRT), LLM-specific optimization (FlashAttention, quantization, vLLM, tensor parallelism), and the practical profile → identify → optimize → verify workflow — with a worked inference example and a 2026+ outlook.

**Audience**: Solution architects, ML platform engineers, quantitative developers, and infrastructure leads who run GPU workloads — training, inference, and HPC — in a banking or enterprise context. Assumes working knowledge of Python/PyTorch and basic systems concepts; no prior CUDA experience required. The banking context (Crédit Agricole CIB, Singapore) frames the business value: GPU cost is a measurable line item, and every 2x throughput gain is a real infrastructure saving.

---

## Table of Contents

1. [GPU Hardware Fundamentals](#1-gpu-hardware-fundamentals)
2. [The CUDA Programming Model](#2-the-cuda-programming-model)
3. [Optimization Techniques](#3-optimization-techniques)
4. [Profiling and Tooling](#4-profiling-and-tooling)
5. [Frameworks-Level Optimization](#5-frameworks-level-optimization)
6. [LLM-Specific Optimization](#6-llm-specific-optimization)
7. [The Practical Optimization Workflow](#7-the-practical-optimization-workflow)
8. [Worked Example: Optimizing PyTorch Transformer Inference](#8-worked-example-optimizing-pytorch-transformer-inference)
9. [The Future: 2026 and Beyond](#9-the-future-2026-and-beyond)
10. [Glossary](#10-glossary)
11. [References and Companion Guides](#11-references-and-companion-guides)

---

### How to Read This Guide

**Relationship to sibling guides.** This is the dedicated deep-dive on *GPU performance engineering* in the systems/performance series. It sits alongside the LLM/AI engineering series and cross-references it rather than duplicating it: GPU *sizing and deployment* live in [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) (§3 hardware landscape, §4 GPU selection, §5 memory bandwidth), LLM *latency engineering* lives in [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md), and model *compression* (quantization/pruning) lives in [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md). The *measurement mindset* (profile first, benchmark before/after, distrust intuition) is the same discipline as the CPU-side guide [low_latency_cpp_development_guide.md](low_latency_cpp_development_guide.md) — this guide is its parallel-computing sibling. Cloud GPU procurement is covered in [cloud_providers_guide.md](cloud_providers_guide.md), ML platforms on GPUs in [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md), and hands-on NVIDIA skill-building in [gain_nvidia_experience.md](gain_nvidia_experience.md).

**Suggested reading paths.** New to GPUs: §1 → §2 → §3 → §7. PyTorch user (no CUDA): §5 → §7 → §8. LLM serving engineer: §6 → §5 → §8. CUDA kernel developer: §2 → §3 → §4 → §7. Architect comparing GPUs: §1.6, §1.7, §9.

**Note on verification.** Facts were researched in August 2026. Claims are marked inline: ✅ **Verified** (confirmed against primary sources or multiple consistent sources during research), ⚠️ **Approximate** (widely cited but architecture-dependent or rounded), ❓ **Reported** (vendor marketing or single-source numbers, treat with care). A consolidated claims-status table follows the TOC. Anything that could not be verified is flagged honestly rather than asserted.

### Claims Status at a Glance

| Claim | Status | Note |
|---|---|---|
| Warp = 32 threads; SIMT execution | ✅ Verified | Fixed across all NVIDIA architectures since 2006 |
| A100: 80 GB HBM2e, ~2.0 TB/s, 312 TFLOPS FP16/BF16 dense | ✅ Verified | NVIDIA datasheet; 2039 GB/s measured |
| H100 SXM: 80 GB HBM3, 3.35 TB/s, FP8 via Transformer Engine | ✅ Verified | NVIDIA datasheet; FP8 = 2× FP16 throughput |
| B200: 192 GB HBM3e, 8 TB/s, FP4 support, 2024 | ✅ Verified | NVIDIA Blackwell launch; ~9 PFLOPS dense FP4 (⚠️ per-GPU TFLOPS vary by source) |
| Tensor cores debuted on Volta V100 (2017) | ✅ Verified | 1st-gen; Ampere A100 = 3rd-gen; Hopper H100 = 4th-gen; Blackwell = 5th-gen |
| FlashAttention (Dao et al., 2022), 2–4× speedup, IO-aware | ✅ Verified | arXiv 2205.14135, NeurIPS 2022; FA2 = arXiv 2307.08691 (2023) |
| PagedAttention / vLLM (Kwon et al., SOSP 2023) | ✅ Verified | arXiv 2309.06180; ~2–4× serving throughput vs baselines |
| GPTQ (2022), AWQ (2023), LLM.int8() (2022) | ✅ Verified | arXiv 2210.17323, 2306.00978, 2208.07339 |
| Speculative decoding (Leviathan et al.) | ✅ Verified | arXiv 2211.17192 (Nov 2022); ICML 2023; Chen et al. 2302.01318 |
| torch.compile / PyTorch 2.0 | ✅ Verified | Announced Dec 2022, stable March 2023 |
| nvprof deprecated, Nsight Compute/Systems replace it | ✅ Verified | Removed from CUDA 13.0; unsupported on modern architectures |
| Memory latencies (shared ~20–30 cyc, global ~400–800 cyc) | ⚠️ Approximate | Architecture- and load-dependent; commonly cited ranges |
| GB200 NVL72 ≈ 1.4 exaFLOPS FP4, 130 TB/s NVLink; B300 (2025) 288 GB class, ~14 PF dense FP4 | ✅ / ❓ | Rack specs ✅ (NVIDIA official); per-GPU dense FP4 varies by source, some cite 270 GB ❓ |
| Roofline model (Williams, Waterman, Patterson 2009) | ✅ Verified | Seminal paper, UC Berkeley |

---

## 1. GPU Hardware Fundamentals

GPU optimization is the art of working *with* a radically different piece of hardware than a CPU. Every technique in this guide — coalescing, occupancy, tiling, tensor cores, mixed precision — is a response to one of the hardware facts in this section. If you internalize nothing else, internalize these: **the GPU is a throughput machine, not a latency machine**, and **for most real workloads the memory system, not the arithmetic units, is the bottleneck**.

### 1.1 GPU vs CPU: Throughput vs Latency

A CPU is built to execute one complex stream of instructions as fast as possible: a few dozen powerful cores (16–128 threads in a server), each with deep out-of-order execution, branch prediction, and speculative execution, backed by a large private cache hierarchy. Its design goal is **latency** — minimize the time from instruction issue to result, for a single thread. A CPU core can go from stalled to productive in ~10–20 cycles because the hardware aggressively hides dependencies.

A GPU is built to execute an enormous number of *simple, independent* operations in parallel. An H100 has 132 streaming multiprocessors (SMs) × 128 FP32 cores ≈ **16,896 CUDA cores**, each of which is a small, in-order ALU with no branch prediction and no out-of-order execution. The GPU tolerates latency not by hiding it in hardware per-thread, but by **massive thread-level parallelism**: when one warp of threads stalls on a memory access, the scheduler switches to another ready warp for free. Its design goal is **throughput** — maximize floating-point operations and bytes moved per second, aggregated across thousands of threads, even if any single thread is slow.

| Property | CPU | GPU |
|---|---|---|
| Design goal | Latency (fast single thread) | Throughput (many parallel threads) |
| Typical core count | 8–128 (server) | 4,000–18,000+ ALUs |
| Threads in flight | 1–2 per core | Up to 2,048 threads per SM |
| Cache / latency hiding | Large per-core caches, HW latency hiding | Small caches; latency hidden by warp switching |
| Best at | Sequential, branchy, low-parallelism code | Dense, data-parallel, massively parallel code |
| Sweet spot | OS, databases, control logic | Matrix math, image/signal processing, simulation |

The practical consequence: **workloads with high parallelism and regular memory access fly on GPUs; small, serial, data-dependent workloads often go *slower*** than on a mid-range CPU (kernel launch overhead alone is microseconds). Part of optimization is knowing when *not* to use the GPU (see §7.3).

### 1.2 The NVIDIA GPU Architecture: SMs, CUDA Cores, Warps

An NVIDIA data-center GPU is a hierarchy of parallel execution units:

- **SM (Streaming Multiprocessor)** — the fundamental compute unit: 128 FP32 **CUDA cores**, 4 tensor cores, 4 warp schedulers, a 64K-register file, and a shared memory/L1 unit. An H100 has 132 SMs; an A100 has 108.
- **CUDA core** — the basic ALU: one FP32 multiply-add per clock. The "16,896 CUDA cores" marketing number is just SMs × 128 (the SM also has separate INT32 pipes).
- **Warp** — the execution group: **exactly 32 threads** in lockstep, one instruction at a time (SIMT, §2.5). The warp is the atomic unit of execution *and* of memory access. Each of the SM's 4 warp schedulers issues one warp-instruction per cycle; when a warp stalls on a load, the scheduler issues another ready warp — the latency-hiding trick.
- **Thread block** — the user-visible unit of work: 32–1024 threads (up to 1024 on modern GPUs) that run on a single SM and can cooperate through shared memory. Blocks are scheduled onto SMs by the hardware; an SM can host several blocks concurrently (limited by threads, registers, and shared memory).

> ✅ **Verified:** warp width = 32 threads has been constant across every NVIDIA GPU architecture since 2006. This is why "the warp is the unit" appears everywhere in CUDA optimization.

### 1.3 The Memory Hierarchy

The GPU memory hierarchy is the single most important thing to understand for optimization, because **memory bandwidth is the scarcest resource in most workloads**:

| Level | Size (A100/H100 per GPU) | Latency (approx.) | Bandwidth | Notes |
|---|---|---|---|---|
| Registers | 64K × 32-bit per SM | ~1 cycle | ~20 TB/s aggregate (per-SM) | Private to a thread; fastest storage; allocation is static per kernel |
| Shared memory | up to 164 KB (A100) / 228 KB (H100) per SM | ~20–30 cycles ⚠️ | ~10+ TB/s per SM | On-chip SRAM, shared by a block; explicit, user-managed cache |
| L1 cache / shared memory unit | same physical SRAM | ~30–40 cycles | | L1 and shared memory share the same on-chip SRAM (configurable split) |
| L2 cache | 40 MB (A100) / 50 MB (H100) | ~200 cycles | ~5–7 TB/s | Shared across all SMs; the last stop before HBM |
| Global memory (HBM) | 80 GB (A100/H100) | ~400–800 cycles ⚠️ | 2.0 / 3.35 TB/s | The main memory (VRAM); all threads can access it |

Key mental model: **registers and shared memory are on-chip and fast; global memory (HBM) is off-chip, slow, and bandwidth-finite.** The GPU version of cache-vs-DRAM is stark: L1/shared is ~100× lower latency than HBM. Every optimization that keeps data on-chip (tiling, fusion, register reuse) is a direct attack on HBM traffic.

### 1.4 Memory Bandwidth: The Currency of GPU Performance

HBM (High Bandwidth Memory) is the stacked-DRAM technology that gives GPUs their enormous bandwidth:

- **A100 80 GB**: HBM2e, **2.0 TB/s** (2039 GB/s measured) ✅
- **H100 SXM 80 GB**: HBM3, **3.35 TB/s** ✅
- **H200** (Hopper refresh, 2024): HBM3e, **4.8 TB/s**, 141 GB ✅
- **B200**: HBM3e, **8 TB/s**, 192 GB ✅

Why this number dominates: an LLM generates one token by reading *all* of its weights from HBM once per token. A 70B-parameter model in FP16 = 140 GB; at 3.35 TB/s the hard floor is 3,350 / 140 ≈ **24 tokens/s per H100 — no matter how many TFLOPS are idle**. That arithmetic (weights cross the bus once per token) is why LLM inference is bandwidth-bound and every LLM optimization (quantization §6.4, KV cache §6.5, batching §8) is about *moving fewer bytes*. The same holds for any streaming kernel: read inputs once, little arithmetic per byte → bandwidth-bound; the only fix is fewer bytes or more bandwidth.

### 1.5 Tensor Cores: The Matrix-Multiply Engines

A **tensor core** is a specialized matrix-multiply unit inside the SM: a dense matrix multiply-accumulate per clock, trading precision flexibility for massive throughput. Tensor cores are why deep learning got ~10× faster per generation even as FP32 CUDA-core throughput grew modestly.

- **1st gen — Volta (V100, 2017)**: the first tensor cores. FP16 inputs, FP32 accumulate. ✅
- **2nd gen — Turing (2018)**: adds INT8/INT4 tensor ops (for inference).
- **3rd gen — Ampere (A100, 2020)**: adds TF32 (FP32-range training with reduced mantissa) and **sparsity** support (2× throughput on 2:4 structured-sparse weights). ✅
- **4th gen — Hopper (H100, 2022)**: adds **FP8** (E4M3/E5M2) with the **Transformer Engine** — a software/hardware stack that automatically switches precision per layer (FP16/BF16/FP8) during training. FP8 doubles tensor throughput again. ✅
- **5th gen — Blackwell (B200, 2024)**: adds **FP4**, doubling peak AI throughput over Hopper. ✅

Tensor cores matter to optimization in three ways: (1) use them — FP16/BF16/FP8 math is 8–16× faster than FP32 on modern GPUs (§5.3); (2) structure data for them — tiled GEMMs with the right memory layout (channels_last for convs, padded K dimensions for GEMMs); (3) the entire software stack (cuBLAS, cuDNN, FlashAttention, vLLM) is written against them, so *framework-level* optimization is mostly "get the kernels that use tensor cores".

### 1.6 The Current Data-Center GPU Lineup

| Spec | A100 80GB SXM | H100 SXM | H200 | B200 | B300 (Ultra) |
|---|---|---|---|---|---|
| Architecture | Ampere (2020) | Hopper (2022) | Hopper | Blackwell (2024) | Blackwell Ultra (2025) |
| FP16/BF16 TFLOPS (dense) | 312 | ~990 | ~990 | ~2,250 (≈2.25 PF) | ~3,100 ❓ |
| FP8 TFLOPS (dense) | — | ~1,979 | ~1,979 | ~4,500 (≈4.5 PF) ❓ | ~6,200 ❓ |
| FP4 TFLOPS (dense) | — | — | — | ~9,000 (≈9 PF) ❓ | ~14,000 (≈14 PF) ❓ |
| Memory | 80 GB HBM2e | 80 GB HBM3 | 141 GB HBM3e | 192 GB HBM3e | 288 GB HBM3e ❓ |
| Bandwidth | 2.0 TB/s | 3.35 TB/s | 4.8 TB/s | 8 TB/s | 8 TB/s |
| NVLink (per GPU) | 600 GB/s | 900 GB/s | 900 GB/s | 1.8 TB/s | 1.8 TB/s |
| TDP | 400 W | 700 W | 700 W | ~1,000 W ❓ | ~1,400 W ❓ |
| Tensor cores | 3rd gen | 4th gen + Transformer Engine | 4th gen | 5th gen | 5th gen |

✅ Verified: A100 (2.0 TB/s, 312 TFLOPS FP16 dense), H100 (3.35 TB/s, FP8 ≈ 2× FP16), H200 (141 GB / 4.8 TB/s), B200 (192 GB / 8 TB/s, FP4), B300 (288 GB class). ⚠️/❓: Blackwell per-GPU dense TFLOPS vary by source and are often quoted *with sparsity* (2× dense); the FP16→FP8→FP4 ≈2× ratios are reliable. Treat vendor "up to" numbers as ceilings, not sustained performance ([on_prem_llm_deployment_guide.md §3](on_prem_llm_deployment_guide.md)).

### 1.7 GPU Generations: A Brief Timeline

| Generation | Year | Flagship | What it introduced |
|---|---|---|---|
| Kepler | 2012 | K80 | SMX design, GPUdirect (predecessor), good FP64 |
| Pascal | 2016 | P100 | HBM2 (first!), unified memory, NVLink 1 |
| Volta | 2017 | V100 | **Tensor cores**, independent thread scheduling, NVLink 2 |
| Turing | 2018 | T4 / RTX 20 | INT8/INT4 tensor ops, ray tracing, TensorRT era |
| Ampere | 2020 | A100 | **3rd-gen tensor cores** (TF32, sparsity), MIG partitioning, NVLink 3 |
| Hopper | 2022 | H100 | **Transformer Engine (FP8)**, NVLink 4 (900 GB/s), DPX instructions |
| Blackwell | 2024 | B200 | **FP4**, 2-die design, NVLink 5 (1.8 TB/s), 8 TB/s HBM3e |
| Blackwell Ultra | 2025 | B300 | 288 GB HBM3e, ~14 PF dense FP4, GB300 NVL72 |

Two through-lines to notice: (1) **memory bandwidth and capacity roughly double per generation** — that is the binding constraint for AI inference; (2) **each generation adds a lower-precision tensor format** (FP16 → TF32/INT8 → FP8 → FP4) — the "quantization everywhere" trend of §9.6 is literally the hardware roadmap.

---

## 2. The CUDA Programming Model

CUDA (**Compute Unified Device Architecture**, NVIDIA's parallel-computing platform and programming model, launched 2007) lets you write C++-style functions that run on the GPU and orchestrate them from the host CPU. Understanding CUDA at the *model* level is essential even if you never write a kernel, because every framework-level knob (torch.compile, CUDA graphs, mixed precision, TensorRT) is manipulating CUDA concepts underneath.

### 2.1 Kernels and the Grid/Block/Thread Hierarchy

A **kernel** is a device function — a function that runs on the GPU — launched from host code. The launch defines a three-level hierarchy:

```
Grid (all threads) → Blocks (each scheduled onto one SM) → Threads (32-thread warps execute in lockstep)
```

- **Thread** — the smallest unit; executes the kernel body with its own registers (within a warp, execution is lockstep).
- **Block** — 1–1024 threads on the *same SM*, able to `__syncthreads()` and share shared memory. The unit of hardware scheduling — and of *scalability*: blocks are distributed automatically, so a kernel runs unchanged on 10 or 132 SMs.
- **Grid** — the full set of blocks in one launch (1D/2D/3D).

### 2.2 Launch Configuration: `<<<grid, block>>>`

```cpp
kernel<<<numBlocks, threadsPerBlock>>>(args...);   // 1D launch
kernel<<<gridDim, blockDim>>>(args...);            // 2D/3D launch
```

- `threadsPerBlock` (blockDim) — typically 128–256, up to 1024. **Must be a multiple of 32** (warp size) for full efficiency — a block of 100 threads wastes 28 lanes.
- `numBlocks` (gridDim) — enough to cover the data (N/blockSize) and ideally to fill all SMs several times over (occupancy, §2.7). The grid-stride loop (§2.9) decouples grid size from data size.

Inside the kernel, three built-in variables give each thread its identity:

- `threadIdx` — index of the thread within its block (e.g., `threadIdx.x`, 0..blockDim.x-1).
- `blockIdx` — index of the block within the grid.
- `blockDim` (and `gridDim`) — dimensions of the block and grid.

The canonical global index for a 1D launch: `int i = blockIdx.x * blockDim.x + threadIdx.x;` — every CUDA program starts here.

### 2.3 A Minimal CUDA Kernel: Vector Add

```cpp
// device function: runs on the GPU, once per thread
__global__ void vecAdd(const float* a, const float* b, float* c, int n) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;   // global index
    if (i < n) c[i] = a[i] + b[i];                   // guard: grid may exceed n
}

// host: launch enough threads to cover n elements
int threads = 256;
int blocks  = (n + threads - 1) / threads;
vecAdd<<<blocks, threads>>>(d_a, d_b, d_c, n);
```

This is the "hello world" of CUDA and already contains the two ideas that recur throughout optimization: **index math** (each thread owns a slice of the output) and **the guard** (grid sizes rarely divide data sizes evenly). Note `cudaMalloc`/`cudaMemcpy` (host↔device) and `cudaDeviceSynchronize()` are omitted for brevity; memory management is covered in §2.4.

### 2.4 The CUDA Memory Model

| Memory | Scope | Lifetime | Speed | Typical use |
|---|---|---|---|---|
| Registers | per thread | kernel | fastest | loop variables, partial sums |
| Shared memory | per block | block | ~20–30 cycles ⚠️ | tiles, reductions, communication between threads of a block |
| Local memory | per thread (spills) | kernel | *slow* (lives in global!) | arrays indexed dynamically that don't fit registers — avoid |
| Global memory | all threads (device-wide) | application | ~400–800 cycles ⚠️ | the main dataset, input/output buffers |
| Constant memory | all threads | application | cached, broadcast-optimized | kernel parameters, lookup tables |
| Texture memory | all threads | application | cached, 2D-locality-optimized | image-like data (mostly legacy in the AI era) |

Rules of thumb that fall out of this table:

- **Global memory is the DRAM** (HBM) — big, slow, the thing you minimize traffic to.
- **Shared memory is the user-managed cache** — the #1 bandwidth tool (tiling, §3.3).
- **Local memory is a trap**: per-thread storage that physically lives in global memory; register "spills" into it (visible in NCU as local-memory traffic) collapse performance.
- **Constant memory** shines for broadcast reads (all threads, same address) — e.g., shared model parameters.

Host ↔ device transfers (`cudaMemcpy`) cross the PCIe bus (or NVLink) at far below HBM bandwidth and are a classic hidden bottleneck; pinned memory (§5.9) and async copies (§3.8) exist to tame them.

### 2.5 SIMT and Warp Execution

GPUs execute in **SIMT — Single Instruction, Multiple Threads**. A warp's 32 threads run the *same instruction* on *different data*, in lockstep, on 32 ALUs. Key consequences:

- The warp is the scheduling unit: the SM's warp schedulers issue one warp-instruction per cycle; 4 schedulers → up to 4 warp-instructions/cycle/SM.
- **One memory instruction = 32 threads' worth of memory transactions.** This is why coalescing (§2.8) is a *hardware-level* requirement, not a style preference.
- Threads in a warp can be conceptually independent (each has its own registers), but they progress in lockstep; the hardware only diverges them when branches demand it (§2.6).

### 2.6 Warp Divergence

```cpp
if (threadIdx.x % 2 == 0) { heavyPath(); } else { lightPath(); }
```

Because a warp executes one instruction at a time, an `if/else` that splits the warp **serializes**: the hardware executes the `if` body for the threads that took it (masking out the others), then the `else` body for the rest. Both paths run for the whole warp — worst case, **2× slowdown for a perfectly balanced branch**, and worse for nested branches. Divergence is measured in "reconvergence points" and shows up in Nsight Compute's `SchedulerStats` / `Divergence` metrics.

Mitigations: (1) sort data so branches become warp-uniform; (2) branch on block/tile-level conditions, not per-thread ones; (3) restructure the algorithm so warps rarely split. Predication (evaluate both sides, mask results) is sometimes cheaper than divergence for short branches.

### 2.7 Occupancy: Hiding Latency with Warps

**Occupancy = the number of active warps per SM, as a fraction of the maximum (64 warps = 2,048 threads on A100/H100).** The entire point of occupancy is latency hiding: when one warp stalls on a 600-cycle HBM load, the scheduler needs *other* ready warps to issue. Low occupancy (e.g., 25%) means the SM often sits idle waiting on memory; high occupancy is not always better (see §3.5 on register pressure) but is the default goal for memory-bound kernels.

Occupancy is limited by the most constrained of four resources, per SM:

| Resource | A100/H100 limit | Consumed by |
|---|---|---|
| Threads | 2,048 | block size × blocks resident |
| Blocks | 32 (A100) / 32 (H100) | launch config |
| Registers | 65,536 × 32-bit | registers/thread × threads (e.g., 255 regs/thread → 256 threads max → 12.5% occupancy!) |
| Shared memory | 164 / 228 KB | shared bytes/block × blocks |

The **occupancy calculator** (in Nsight Compute) computes the theoretical maximum for a config. In practice: memory-bound kernels want high occupancy; tensor-core kernels care more about keeping the tensor pipe fed.

### 2.8 Memory Coalescing

**Coalescing is the rule that consecutive threads should access consecutive addresses.** When warp threads 0–31 read 32 consecutive 4-byte floats (a 128-byte contiguous segment), the memory system satisfies the whole warp with **one 128-byte transaction**. When threads read strided/scattered addresses, the warp needs multiple transactions — 32 separate 32-byte sectors in the worst case — and effective bandwidth collapses by up to 32× (measured in Nsight Compute's `MemoryWorkloadAnalysis` as uncoalesced access).

```cpp
// COALESCED: thread i reads a[i]   → 32 consecutive floats = 1 transaction
float x = a[i];
// UNCOALESCED: thread i reads a[i * 32]  → stride-32 pattern = 32 transactions
float y = a[i * 32];
// UNCOALESCED: transpose-style access (column stride)
float z = mat[col * N + row];
```

The fix is *layout*: make the innermost (fastest-varying) index map to `threadIdx.x` — array-of-structs → struct-of-arrays; tile matrices so inner loops walk rows. Coalescing is the #1 memory optimization and the first check for memory-bound kernels (§3.2).

### 2.9 Grid-Stride Loops

The **grid-stride loop** decouples the grid from the data size and keeps the grid *persistent* across kernel launches:

```cpp
__global__ void saxpy(float* y, const float* x, float a, int n) {
    for (int i = blockIdx.x * blockDim.x + threadIdx.x; i < n; i += gridDim.x * blockDim.x) {
        y[i] = a * x[i] + y[i];
    }
}
// launch exactly enough blocks to fill the GPU, not enough to cover n
saxpy<<<numSMs * blocksPerSM, 256>>>(y, x, a, n);
```

Benefits (✅ well-established idiom — NVIDIA "CUDA Pro Tip: Grid-Stride Loops", 2013): grid size decouples from data size (no recompilation per N); enables **persistent kernels** (§3.9) that keep state in registers/shared memory across iterations; replaces many small launches with one; handles non-divisible `n` naturally. Cost: a loop-carried dependency — usually negligible.

---

## 3. Optimization Techniques

This section is the toolkit. Every technique below is a lever on one of the fundamental resources from §1: HBM bandwidth, on-chip SRAM (shared memory), registers, warp slots, or instruction issue. The discipline is to *know which resource is the bottleneck before pulling levers* (§7).

### 3.1 The Roofline Model: Compute-Bound vs Memory-Bound

The **roofline model** (Williams, Waterman & Patterson, 2009 ✅) is a single graph that tells you what a kernel *can* achieve. Plot attainable performance (FLOPS) vs **arithmetic intensity** — the ratio of FLOPs performed per byte of data moved from memory:

- **Arithmetic intensity (AI) = FLOPs / bytes moved.** A kernel that does 1 FLOP per byte has AI = 1; a GEMM that reuses data in registers has AI in the hundreds.
- Two roofs: the **memory roof** (peak HBM bandwidth × AI — the diagonal) and the **compute roof** (peak FLOPS — the flat top), meeting at the **ridge point** `peak_FLOPS / peak_BW` (≈ 300 FLOP/byte on an H100 at FP16).
- Left of the ridge, the kernel is **memory-bound** (bandwidth-limited): doubling FLOPS does nothing; cutting bytes moved does everything. Right of the ridge, the kernel is **compute-bound**: the opposite.

**This is the most important decision tool in GPU optimization.** It tells you where to spend effort: memory-bound kernels get tiling/fusion/precision-reduction (fewer bytes); compute-bound kernels get tensor cores/ILP/algorithmic improvements (fewer FLOPs or faster FLOPs). Nsight Compute's **Speed-of-Light (SOL)** analysis implements exactly this: it reports "Memory [x%] / Compute [y%] of peak" per kernel, and the lower percentage is your current ceiling.

### 3.2 Memory Optimization: Coalescing, Reordering, Padding

For memory-bound kernels, the objective is *fewer, wider, contiguous transactions*:

1. **Coalesce (§2.8)**: innermost index = `threadIdx.x`; SoA over AoS; avoid transpose-style access.
2. **Reorder**: transpose matrices on the host (or with a transpose kernel) once, instead of striding in the hot loop.
3. **Pad**: shared-memory tiles and row-major matrices benefit from padding (see bank conflicts §3.4); global-memory arrays benefit from alignment to 128-byte boundaries (use `cudaMalloc` — it guarantees 256-byte alignment) and from making row lengths multiples of 128 bytes.
4. **Vectorize**: load/store 4 floats at once as `float4` (§3.6) — one 16-byte transaction per thread instead of four 4-byte ones; this is standard practice in production kernels and often gives 1.5–2× on bandwidth-bound loops.
5. **Read-only data → `__ldg()` / const restrict** to route through the read-only cache and avoid L1 pollution.

### 3.3 Shared Memory and Tiling

**Tiling** is the canonical shared-memory pattern: divide the output into tiles; load each tile's inputs from global memory *once* into shared memory; have the whole block compute from the fast on-chip copy; write the tile out. For a matrix multiply C = A×B, naive kernels read A and B from HBM once per output element; tiled kernels read each input element once per tile — cutting HBM traffic by the tile size. The pattern: (1) cooperative load — a 32×32 tile = 1,024 elements loaded by 256 threads in parallel; (2) `__syncthreads()` to make the tile visible to the whole block; (3) compute, reusing shared data across many output elements; (4) repeat for the next tile. This is exactly how cuBLAS GEMMs and FlashAttention (§6.2) are structured — FlashAttention is *tiling applied to attention*.

**Shared memory is also the only way threads in a block communicate** (plus atomics on global memory). Typical uses: reductions (partial sums per block), stencils (halo cells), histograms, and as a software pipeline buffer.

### 3.4 Bank Conflicts

Shared memory is physically organized as **32 banks of 4 bytes each** (✅ standard architecture fact). In one cycle, a warp can access 32 *different* banks in parallel. When two or more threads in a warp hit the **same bank**, the access serializes: a **2-way conflict = 2 cycles**, 4-way = 4 cycles, worst case 32-way = 32 cycles (as bad as a global-memory round trip).

- Conflict-free: thread `i` reads `s[i]` — 32 consecutive words hit 32 banks. ✅
- 2-way conflict: `s[i + 16]` with a 32-float array — threads i and i+16 map to the same bank (mod 32).
- Classic fix: **pad the tile by one element** — `s[TILE][TILE+1]` — shifting row starts so columns land on different banks. This single-character-class fix (adding `+1` to the stride) is one of the highest-ROI edits in CUDA.
- Broadcast: when all threads read the *same* address, shared memory broadcasts it conflict-free — useful for constants.

Nsight Compute reports bank conflicts explicitly (the `shared` section of `MemoryWorkloadAnalysis`).

### 3.5 Register Pressure and the Occupancy Tradeoff

Registers are the fastest storage, and the compiler *wants* to use them — but each SM has a fixed 64K registers, so **registers-per-thread directly caps occupancy**: 255 regs/thread → max 256 threads/SM (12.5% occupancy on a 2,048-thread SM). The tradeoff:

- **Too few registers** → spills to local memory (which lives in HBM!) — catastrophic for bandwidth-bound kernels.
- **Too many registers** → occupancy drops → latency hiding starves → memory-bound kernels slow down.
- **Sweet spot**: for memory-bound kernels, ~32–64 regs/thread is typical; for compute-bound tensor-core kernels, higher register use is fine because occupancy is less important.
- Controls: `__launch_bounds__(256, 8)` caps registers; `-maxrregcount=N` is the blunt version; NCU's `LaunchStats` shows achieved occupancy and register counts. **Let the profiler, not intuition, drive this knob** — forcing 100% occupancy on a compute-bound kernel can *hurt*.

### 3.6 Compute Optimization: ILP, Vectorization, Fast Math

When a kernel is compute-bound (right of the roofline ridge):

1. **ILP (Instruction-Level Parallelism)**: within a thread, independent operations can overlap. Instead of one accumulator, use 4 (e.g., 4 independent partial sums in a reduction, then combine) so the compiler/scheduler has multiple independent instruction chains. Warp-level parallelism covers latency across threads; ILP covers it within a thread. Gains of 1.5–2× on serial-looking loops are common.
2. **Vectorization**: `float4` loads/stores (16 bytes per instruction) reduce instruction count 4× and improve memory throughput; also `int4`, `double2`. Combined with tiling, this is the standard GEMM structure.
3. **Fast math**: `__expf()`, `__logf()`, `__sinf()`, `-use_fast_math` swap precise IEEE math for faster approximations (roughly 2× on transcendentals; accuracy loss of ~1–2 ULP). Use only where the numerical tolerance allows — e.g., softmax denominators, not pricing models where precision is regulated.
4. **Reduce FLOPs**: strength reduction, hoisting invariants, avoiding `pow()` in loops. Also: 2:4 structured sparsity (Ampere+) gives 2× tensor-core throughput when weights have ~50% zeros by construction (pruning → see [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md)).

### 3.7 Kernel Fusion

**Fusion = merging multiple kernels (or framework ops) into one, eliminating intermediate global-memory round trips.** A chain like `z = sigmoid(a*x + b)` is three kernels in naive code: each writes a full array to HBM and reads it back. Fused, the intermediate `a*x+b` lives in registers/shared memory and never touches HBM. For element-wise chains the speedup is often 2–5× purely from bandwidth.

- The **memory-traffic math**: three-kernel chain moves 4 array-size passes (write+read × 2 intermediates); one fused kernel moves 1 (read inputs, write output).
- Framework level: torch.compile (§5.1), TensorRT, and XLA all fuse automatically — this is the main source of their speedups.
- Hand-written CUDA: fusion is why production kernels (FlashAttention, layer-norm+linear+activation, softmax) are monolithic.
- Caution: fusion that *increases* register pressure or serializes independent work can backfire — measure (§4).

### 3.8 Streams, Async Memcpy, Events

**CUDA streams** are independent queues of work. Kernels in *different* streams can run concurrently (if resources allow); kernels in the *same* stream execute in order. Streams enable the classic overlap pattern:

```
H2D copy (stream 1) ──▶ kernel (stream 1) ──▶ D2H copy (stream 1)
H2D copy (stream 2) ──▶ kernel (stream 2) ──▶ D2H copy (stream 2)
        ──────────────── timeline ────────────────▶
```

With multiple streams and chunked data, copy engines (DMA) transfer chunk *i+1* while the GPU computes chunk *i* — hiding PCIe/H2D latency behind compute. Key APIs:

- `cudaStreamCreate`, `kernel<<<grid, block, 0, stream>>>`, `cudaMemcpyAsync` (needs **pinned** host memory, §5.9) — async copies are executed by dedicated copy engines and don't occupy SMs.
- **CUDA events** (`cudaEventRecord`) timestamp the stream: wrap a kernel with events to measure its GPU time precisely, or use events to synchronize streams. Event-based timing is the correct way to benchmark kernels (vs `cudaDeviceSynchronize` around wall clock, which includes launch gaps).
- `cudaStreamSynchronize`/`cudaDeviceSynchronize` — sync points; excessive sync kills overlap. In framework code, `.cuda()`/`.item()` are notorious implicit syncs.

### 3.9 Persistent Kernels

A **persistent kernel** launches once with a device-sized grid and loops internally (grid-stride §2.9), keeping data in registers/shared memory across "iterations" and avoiding relaunch overhead. Use cases: repeated small workloads where launch overhead dominates (3–10 µs each); software pipelining; serving loops holding weights in shared memory. Costs: complexity and manual work scheduling. Modern practice prefers **CUDA graphs** (§5.2) for launch overhead, reserving persistent kernels for cross-iteration state.

### 3.10 Technique Summary

| Technique | What it fixes | Effort | Impact |
|---|---|---|---|
| Coalescing (§2.8/§3.2) | Wasted bandwidth from scattered access | Low | Up to 32× on pathological access; 2–4× typical |
| Tiling with shared memory (§3.3) | HBM traffic in data-reuse kernels | Medium | 2–10× on GEMM-like kernels |
| Bank-conflict padding (§3.4) | Shared-memory serialization | Trivial (a `+1`) | 2–32× on affected kernels |
| Register/occupancy tuning (§3.5) | Latency hiding / spills | Low–Medium | 1.5–3× on memory-bound kernels |
| Vectorization / ILP (§3.6) | Instruction issue + bandwidth | Low | 1.5–2× typical |
| Fast math (§3.6) | Transcendental cost | Trivial | ~2× on math-heavy kernels |
| Kernel fusion (§3.7) | Intermediate memory traffic, launch count | Medium | 2–5× on op chains |
| Streams/async (§3.8) | PCIe copy stalls, serialized copy+compute | Medium | 1.5–2× on pipeline workloads |
| Persistent kernels (§3.9) | Launch overhead, cross-launch state | High | 1.2–3× on small repeated work |
| Precision reduction / tensor cores (§5.3) | FLOP ceiling | Low (framework) | 2–8× vs FP32 |
| CUDA graphs (§5.2) | Launch overhead (µs-scale gaps) | Low (framework) | 1.2–2× on small-step workloads |

Rule: **the order of magnitude comes from the top rows; the last 20% comes from the bottom rows.** Always profile first (§4) — the technique list is a menu, not a checklist.
---

## 4. Profiling and Tooling

You cannot optimize what you cannot measure. The NVIDIA profiling stack has three layers, and each answers a different question:

| Tool | Level | Answers |
|---|---|---|
| **Nsight Systems** (`nsys`) | System / application | *Where is the time going* — kernel gaps, CPU-GPU overlap, PCIe transfers, launch overhead |
| **Nsight Compute** (`ncu`) | Single kernel | *Why is this kernel slow* — occupancy, memory pattern, compute utilization, speed-of-light |
| **Nsight Graphics** | Graphics / Vulkan / D3D | *Why is the frame slow* — game/graphics pipelines (brief, §4.5) |

The legacy tool `nvprof` (and its GUI, nvvp / Visual Profiler) is **deprecated and removed from CUDA 13.0**; NVIDIA's official guidance is to use Nsight Compute + Nsight Systems (✅ verified — NVIDIA developer forum announcement; also unsupported on Volta+ architectures). Treat any 2020s-era tutorial mentioning nvprof as outdated on the tooling, though the concepts carry over.

### 4.1 Nsight Systems (NSYS): The System-Level Profiler

`nsys` profiles the *whole application* — CPU threads, GPU kernels, memory copies, CUDA API calls, and their timeline interleaving. It answers: **is the GPU actually busy?** Typical findings: 60% of wall time is CPU-side gaps between kernel launches; copies and kernels are serialized instead of overlapped; a `torch.Tensor.item()` call is synchronizing the pipeline every iteration.

```bash
nsys profile --trace=cuda,nvtx,osrt -o app_profile ./my_app
nsys stats app_profile.nsys-rep          # summary tables
nsys stats --report cuda_gpu_kern_sum app_profile.nsys-rep   # kernel time table
```

The **NVIDIA Tools Extension (NVTX)** lets you annotate ranges (`nvtxRangePush("attention")`) so the timeline shows *your* components, not opaque kernels — PyTorch emits NVTX ranges automatically via `torch.profiler`. The classic NSYS workflow: run a few inference iterations, look for (1) GPU idle gaps (launch overhead, syncs), (2) H2D/D2H copy time, (3) dominant kernels — then zoom into the winner with NCU.

### 4.2 Nsight Compute (NCU): The Kernel-Level Profiler

`ncu` profiles a *single kernel* in depth and answers the "why". It replays the kernel with hardware counters and reports, per kernel:

- **Speed of Light (SOL) analysis**: "Compute (SM) [42%] · Memory [78%] of peak" — your roofline position (§3.1). The lower of the two is the practical ceiling.
- **Memory Workload Analysis**: achieved bandwidth vs peak, coalescing quality (uncoalesced access %), shared-memory bank conflicts, local-memory (spill) traffic.
- **Compute Workload Analysis**: pipe utilization (FMA, tensor, ALU, SFU), instruction mix, warp stall reasons.
- **SchedulerStats / WarpStateStats**: occupancy achieved vs theoretical, stall reasons (long scoreboard = memory waits, barrier, etc.).
- **LaunchStats**: grid/block config, registers/thread, shared memory/block, achieved occupancy.

```bash
ncu --set full -o kernel_profile kernel_launcher        # full metric set
ncu --section SpeedOfLight --section MemoryWorkloadAnalysis ./app
ncu --metrics sm__throughput.avg.pct_of_peak_sustained_elapsed,gpu__time_duration ./app
```

`--set full` gathers everything (slow); `--set basic` is the daily driver; `--section` selects analyses. **Workflow: NSYS finds the slow kernel → NCU explains it → fix → re-profile.** NCU also hosts the occupancy calculator and register counts (§3.5).

### 4.3 The Legacy: nvprof

`nvprof` was the CUDA profiler for a decade (`nvprof --print-gpu-trace ./app`). It was deprecated (CUDA 10, 2018), **removed from CUDA 13.0**, and does not support Volta-and-newer GPUs — so it cannot profile anything you'd optimize today. Superseded: kernel metrics → NCU; timeline → NSYS. `nsys` ships an `nvprof` compatibility mode for legacy scripts, but new work targets NSYS/NCU directly.

### 4.4 Nsight Graphics (Brief)

Nsight Graphics is the *graphics* profiler — frame-level capture of Vulkan/DirectX/OpenGL pipelines, shader performance, and ray-tracing debug. It is for rendering engineers, not compute (frame = your unit of work, like kernel = your unit of work). Listed only to disambiguate the Nsight brand; irrelevant to CUDA-AI.

### 4.5 The Profiling Workflow: Profile → Analyze → Fix → Re-Profile

1. **NSYS the application** (short run, 10–30 s): find GPU idle gaps, copy serialization, dominant kernels. *Question: is the GPU busy?*
2. **NCU the dominant kernel**: SOL analysis, memory vs compute bound, occupancy, coalescing, bank conflicts, spills. *Question: which resource is the wall?*
3. **Map the finding to a technique** (§3 menu): memory-bound → coalescing/tiling/precision; compute-bound → tensor cores/ILP/fast-math; latency-bound → CUDA graphs/streams; occupancy-bound → register/block tuning.
4. **Fix, benchmark, re-profile.** GPU kernel time and wall time differ — a 2× kernel win with a CPU-bound pipeline shows ~0 wall improvement (§7.6).

Discipline rules: **one change at a time** (techniques interact — fusion changes occupancy), and **re-benchmark on the target hardware** (A100-tuned kernels can regress on H100 and vice versa).

### 4.6 Tools Comparison

| Tool | Level | Primary use case | Key output | Command |
|---|---|---|---|---|
| Nsight Systems | System | Find idle GPU, launch gaps, copy serialization, CPU bottlenecks | Timeline (GPU/CPU/API) | `nsys profile` |
| Nsight Compute | Kernel | Diagnose why a kernel is slow (SOL, occupancy, memory) | Per-kernel metrics + SOL | `ncu --set full` |
| nvprof (legacy) | System/kernel | *Deprecated* — migration reference only | — | removed in CUDA 13 |
| Nsight Graphics | Frame | Graphics/shader debugging | Frame capture | standalone GUI |
| `torch.profiler` | Framework | PyTorch op-level timing, kernel names, memory | Table + trace | `torch.profiler.profile` |
| `nvidia-smi` | Device | Live utilization, memory, power, temperature | Dashboard | `nvidia-smi -l 1` |

---

## 5. Frameworks-Level Optimization

Most GPU optimization in 2026 happens *inside frameworks*, not in hand-written CUDA. The good news: the framework layer automates most of §3. The discipline is knowing which knobs exist, what they do under the hood, and when they pay off. This section covers PyTorch (the dominant framework in banking/AI), TensorRT (inference), ONNX Runtime (portability), and JAX (XLA), then the data pipeline.

### 5.1 PyTorch: torch.compile

`torch.compile` (PyTorch 2.0, announced Dec 2022, stable March 2023 ✅) replaces eager per-op kernel launches with **graph compilation**: it traces the model (TorchDynamo), decomposes ops, **fuses** element-wise chains, and generates Triton/CUDA kernels. Typical speedups: 1.2–2× on transformer inference/training, up to 5× on eager-heavy models. Modes:

```python
model = torch.compile(model)                       # default: max fusion
model = torch.compile(model, mode="reduce-overhead")  # + CUDA graph capture
model = torch.compile(model, mode="max-autotune")     # + best kernel configs
```

Under the hood it does exactly §3.7 (fusion), §3.6 (vectorization), §5.2 (CUDA graphs in reduce-overhead). Practical notes: (1) the first call pays compilation cost (seconds–minutes) — compile once, warm up, then benchmark steady state; (2) dynamic shapes hurt — fix them where possible; (3) `mode="reduce-overhead"` is the usual inference default; (4) custom Python control flow defeats the tracer. Context: [ai_llm/deep_learning_frameworks_comparison_guide.md §3](ai_llm/deep_learning_frameworks_comparison_guide.md).

### 5.2 CUDA Graphs

**CUDA graph capture** records a sequence of kernel launches into a graph object, then replays it with *one* launch call. This eliminates per-launch CPU overhead — each kernel launch costs ~3–10 µs of CPU time, and a 100-kernel transformer forward pass pays ~0.3–1 ms of pure launch overhead per iteration, on top of gaps where the CPU can't keep the GPU fed. Graphs collapse this to microseconds.

```cpp
cudaStreamBeginCapture(stream);   // capture: kernels launch normally
forward_pass(...);                // recorded, not executed
cudaStreamEndCapture(stream, &graph);
cudaGraphInstantiate(&exec, graph, 0);   // build executable graph
cudaGraphLaunch(exec, stream);           // replay with ONE launch
```

```python
g = torch.cuda.CUDAGraph(); s = torch.cuda.Stream()
with torch.cuda.stream(s):
    for _ in range(3): dummy = model(dummy)   # warmup
    torch.cuda.synchronize()
    with torch.cuda.graph(g): out = model(input_batch)  # capture
g.replay()                                     # steady state: one-call replay
```

Requirements: **static memory addresses** — input/output buffers must be pre-allocated and copied in-place each iteration (that's why vLLM and serving stacks pre-allocate request buffers); dynamic shapes/allocations are not capturable. Payoff: 1.2–2× on small-step workloads, and — critically — *stabler* latencies (removes CPU-jitter from the critical path). This is the same mechanism `torch.compile(mode="reduce-overhead")` and TensorRT use internally.

### 5.3 Automatic Mixed Precision (AMP) and FP16/BF16

**Mixed precision** runs the compute-heavy ops in a reduced precision (FP16 or BF16) while keeping a full-precision (FP32) copy of critical state (master weights during training, or FP32 accumulation in tensor-core ops). Tensor cores run FP16/BF16 at **2× the FP32 rate on A100 (312 vs 156 TFLOPS) and ~8× at FP8 on H100** — precision is a hardware-level throughput lever, not just a memory trick.

```python
# PyTorch AMP (modern API; torch.cuda.amp is the legacy name)
from torch.amp import autocast, GradScaler
with autocast("cuda", dtype=torch.float16):   # or bfloat16
    loss = model(x)
scaler = GradScaler("cuda")                    # FP16 needs loss scaling; BF16 does not
scaler.scale(loss).backward()
```

Key facts (✅): AMP/autocast arrived in PyTorch 1.6 (2020); FP16 halves memory *and* doubles tensor throughput; **BF16 (bfloat16)** — 8-bit exponent, 7-bit mantissa, FP32 range — avoids FP16's overflow/underflow issues and needs **no loss scaling**, with identical memory savings (both are 2 bytes). Rule of thumb: **training → BF16 (A100+); inference → FP16 or BF16; FP8 → H100+ only** (§6.4 covers weight quantization, a different axis). Validate output tolerance on *your* workloads before production (§7.6) — the banking discipline.

### 5.4 Memory Formats: `channels_last`

`channels_last` (NHWC) is the memory layout where the channel dimension is contiguous. cuDNN's tensor-core convolution kernels are optimized for NHWC, and PyTorch makes it one line:

```python
model = model.to(memory_format=torch.channels_last)
x = x.to(memory_format=torch.channels_last)
```

For convolution-heavy models (vision, and the conv layers inside multimodal models) this gives meaningful speedups (often 1.2–1.5× on the conv portion, more on tensor-core-constrained GPUs). Transformers are mostly GEMMs (unaffected), so `channels_last` matters for CNN/vision workloads — e.g., document-processing models. (The GEMM analogue — K-dimension padding/alignment to tensor-core tiles — is done automatically by cuBLAS/TensorRT.)

### 5.5 TensorRT

**NVIDIA TensorRT** is NVIDIA's inference engine: it takes a trained model (ONNX, or natively via PyTorch/TensorFlow) and produces an optimized **plan file** — a serialized, device-specific execution graph. What it does under the hood: layer fusion (including cross-layer fusions like conv+BN+ReLU), kernel autotuning, precision reduction (**FP16/INT8/FP8 with calibration**), and CUDA-graph-style execution. Typical results: 1.5–3× over eager FP32 PyTorch on the same GPU, with lower latency variance.

```bash
trtexec --onnx=model.onnx --fp16 --saveEngine=model.plan     # build
trtexec --loadEngine=model.plan --shapes=input:1x3x224x224   # benchmark
```

Key facts: (1) plans are **device- and TensorRT-version-specific** — a plan built for H100 doesn't run on A100; rebuild per fleet (CI artifact). (2) INT8 needs a **calibration dataset** (a representative sample set to compute activation ranges); FP8 on H100+ similarly. (3) For LLMs, the modern path is **TensorRT-LLM** (§6.7), which is TensorRT's engine rebuilt around LLM serving (paged KV cache, in-flight batching, FP8). TensorRT is the standard answer for "we have a fixed model, fixed shapes, and need the last drop of latency" — the classic bank use case: a document-intelligence model behind an SLA.

### 5.6 ONNX Runtime

**ONNX Runtime (ORT)** executes ONNX models across backends — including CUDA and TensorRT **execution providers (EPs)** — for cross-framework portability: train in PyTorch, export to ONNX, run in .NET/Java/C++/Python without PyTorch in production. Speed is comparable to PyTorch eager (better with graph optimizations); the real value is **operational** — no Python runtime, smaller attack surface, standard ops — and the ONNX file as an auditable model artifact. For maximum speed, ORT's TensorRT EP delegates to §5.5.

### 5.7 JAX / XLA

**JAX** (Google, 2018) brings NumPy-style programming with **XLA** compilation: `jax.jit` traces functions into fused, GPU-optimized programs (the same fusion/CUDA-graph family as torch.compile), and `jax.pmap`/`jax.shard_map` express data/tensor parallelism declaratively. Strengths: aggressive fusion and autotuning (often the fastest pure-Python path to a custom training loop), first-class FP8, and a functional model that makes compiler optimization easier. Weaknesses: ecosystem (PyTorch dominates LLM land) and harder debugging of traced programs. Full comparison: [ai_llm/deep_learning_frameworks_comparison_guide.md §4](ai_llm/deep_learning_frameworks_comparison_guide.md). The pattern worth stealing even in PyTorch: **think in fused graphs, not eager ops** (§5.1).

### 5.8 Framework Optimization Matrix

| Technique | PyTorch | TensorRT | ONNX RT | JAX | Effort | Typical gain |
|---|---|---|---|---|---|---|
| Graph compilation / fusion | torch.compile | built-in | built-in (graph opt) | XLA (implicit) | Low | 1.2–2× |
| CUDA graphs | `torch.cuda.graph` | built-in | via TRT EP | built-in | Low | 1.2–2× (small steps) |
| FP16/BF16 AMP | autocast | `--fp16` | `--enable-fp16`? (ORT flag) | `jax.numpy` float16 | Low | 1.5–2× |
| FP8 / INT8 | limited | calibration | via TRT EP | native | Medium | 2–4× |
| Kernel autotuning | `max-autotune` | `trtexec` autotune | — | XLA autotune | Low | 1.2–2× |
| channels_last | `.to(memory_format=...)` | built-in | built-in | via XLA layout | Low | 1.2–1.5× (conv) |
| Custom CUDA kernels | custom autograd | plugin | custom op | custom calls | High | 2–10× (hot spots) |

### 5.9 The Data Pipeline: DataLoader, Pinned Memory, Prefetch

The GPU is fast; starving it from the CPU side is embarrassingly common — 5 ms compute per step with 20 ms data loading yields 25% GPU utilization regardless of kernel quality.

- **`num_workers`**: load/transform data in worker processes (rule of thumb: 4–8× the number of GPUs, tuned empirically) so the main process never blocks on disk/decoding.
- **Pinned (page-locked) memory**: `pin_memory=True` in the DataLoader makes host buffers page-locked, enabling **async H2D copies** (`cudaMemcpyAsync`, §3.8) that overlap with compute instead of blocking.
- **Prefetching**: `prefetch_factor` keeps multiple batches in flight; on top of workers, the *next* batch transfers while the *current* one computes.
- **GPU-side data**: keep tabular/repeated datasets resident on the GPU (80–288 GB fits a lot) and skip the PCIe round trip entirely.
- **GPU decode**: batch decode images/audio on the GPU with DALI or `torchdata` where CPU decode is the bottleneck.

Measure it: `nsys` (or `nvidia-smi` GPU-util sampling) showing the GPU idle between steps is the signature of a CPU-bound pipeline — the fix is the data path, never a kernel change.

---

## 6. LLM-Specific Optimization

LLM inference is the highest-value GPU workload in banking (document analysis, RAG, code generation, trading-assistant copilots), and it has its own optimization physics: **decoding is memory-bandwidth-bound** (every token reads all weights from HBM — §1.4), and the **KV cache** (the per-request memory that stores every token's key/value activations) dominates memory at scale. This section covers the techniques that make LLM serving fast; the companion guides cover deployment ([on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md)) and latency engineering ([ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md)) in depth.

### 6.1 Why Attention Is the Battleground

The attention layer computes Q·Kᵀ scores (O(seq²) FLOPs) and reads/writes the whole sequence's activations from HBM. Naive attention (1) materializes the full seq×seq score matrix in HBM (quadratic memory), (2) round-trips it through softmax, (3) multiplies by V. For long sequences this is the #1 cost center — and it is *IO-bound*: the FLOPs are cheap, moving intermediates is not. Every modern attention implementation attacks that IO.

### 6.2 FlashAttention and FlashAttention-2

**FlashAttention** (Dao, Fu, Ermon, Rudra, Ré — arXiv May 2022, NeurIPS 2022 ✅) is an **IO-aware exact attention** algorithm: it tiles the Q/K/V matrices and processes attention in blocks entirely in on-chip SRAM (shared memory), recomputing softmax statistics on the fly (online softmax) instead of storing the full score matrix. Result: HBM traffic for attention drops from O(seq²) to O(seq), giving **2–4× speedup vs optimized baselines and near-linear (vs quadratic) memory** (✅ paper claims). **FlashAttention-2** (July 2023 ✅) improves parallelism and work partitioning — roughly **2× faster than FA1** and 5–9× vs standard attention at long sequences. Today FA ships inside PyTorch (`scaled_dot_product_attention` dispatches to it), vLLM, and TensorRT-LLM — you get it by default. It matters because (1) it is the canonical *tiling + fusion applied to a real algorithm* (§3.3/§3.7), and (2) it proves "attention is slow" is usually a memory problem, not a math problem.

### 6.3 PagedAttention and vLLM

The **KV cache** is per-request state: for every generated token, each layer stores a key and value vector. At 2 bytes/element (FP16), a 70B-class dense model with 80 layers and hidden 8192 needs ~2.6 MB per token — 32K-token contexts cost ~80+ GB. Naive serving **pre-allocates contiguous KV buffers per request**, wasting 60–80% of KV memory to fragmentation and worst-case reservations. **PagedAttention** (Kwon et al., SOSP 2023 ✅) applies OS-style *paging*: the KV cache lives in fixed-size **blocks** (e.g., 16 tokens) mapped via a block table, so requests occupy only the blocks they use, non-contiguously. On top of it, **vLLM** (the open-source serving engine) adds continuous batching. Result: **~2–4× serving throughput vs state-of-the-art baselines** (✅ paper), near-zero KV waste, and KV sharing across requests. vLLM is now the default open-source LLM server, and its design is copied by TensorRT-LLM and others ([ai_llm/llm_latency_optimization_guide.md §5](ai_llm/llm_latency_optimization_guide.md)).

### 6.4 Quantization: GPTQ, AWQ, FP8, LLM.int8()

Quantization shrinks weights (and sometimes activations) to fewer bits — the direct attack on the *bandwidth-bound* physics of §1.4: **4-bit weights move 4× fewer bytes per token than FP16**. The major methods (✅ dates verified):

- **LLM.int8()** (Dettmers et al., Aug 2022): 8-bit matrix multiplication with **outlier separation** — the ~0.1% of activation channels with extreme magnitudes stay in FP16, the rest run INT8. First practical lossless 8-bit inference for 175B-scale models; shipped in **bitsandbytes** (`load_in_8bit`). 
- **GPTQ** (Frantar et al., Oct 2022): **post-training quantization (PTQ)** to 3–4 bits using approximate second-order (Hessian) information to correct quantization error layer by layer. Opened the 4-bit era; `AutoGPTQ`/`gptqmodel`; 4-bit GPTQ is the standard compact-deployment format.
- **AWQ** (Lin et al., June 2023): **activation-aware weight quantization** — observes that ~1% of weights are "salient" (protecting them preserves quality) and scales them before quantizing; protects salient channels instead of doing error correction. Comparable quality to GPTQ at 4-bit with a simpler, hardware-friendly flow.
- **FP8** (H100+, 2023): the hardware-native 8-bit format (E4M3 for forward, E5M2 for backward) — 2× tensor throughput *and* half the bytes, no calibration dataset needed (unlike INT8). The default for H100-class training and high-throughput serving.

Practical guidance: 4-bit (GPTQ/AWQ) is the sweet spot on A100-class hardware (2.5–4× memory cut, minimal quality loss on most tasks); FP8 is best on H100+; INT8 is largely superseded by FP8 on new hardware. **Always validate on your own tasks** (§7.6) — banking text (legal language, product names, IDs) is outlier-heavy, exactly the regime where quantization degrades. Full treatment: [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md), [on_prem_llm_deployment_guide.md §11](on_prem_llm_deployment_guide.md).

### 6.5 KV Cache Optimization

Beyond paging (§6.3), KV-cache engineering includes:

- **GQA/MQA** (grouped/multi-query attention): shared KV heads cut KV size by the group factor (Llama-3 70B's GQA cut KV ~8× vs Llama-2) — an *architecture* decision at training time.
- **KV quantization**: store the cache in FP8/INT8/4-bit (vLLM, TensorRT-LLM support this) — 2–4× less KV memory, small quality cost; increasingly the default for long contexts.
- **Context management**: sliding-window attention (retain a window of recent tokens), chunked prefill, and context pruning/compaction at the serving layer.
- **Prefix caching** (vLLM, SGLang): shared system prompts reuse the same KV blocks across requests — big win for RAG/chat workloads with common prefixes.

### 6.6 Parallelism: TP, PP, DP

When a model doesn't fit one GPU, or one GPU isn't fast enough, the standard parallelisms are:

| Strategy | Split | Communication per step | Best when |
|---|---|---|---|
| **Data parallelism (DP)** | Replicate the model; shard the batch | Gradients all-reduce (training) / none (inference) | Small models, high batch demand |
| **Tensor parallelism (TP)** | Shard each layer's weight matrices across GPUs | All-reduce *per layer* (NVLink-critical) | Models too big for one GPU; latency-sensitive |
| **Pipeline parallelism (PP)** | Layer ranges per GPU | Activations between stages (bubbles) | Very large models; throughput over latency |
| Combined (3D parallel) | DP × TP × PP | All of the above | Frontier-scale training (e.g., Llama-3-405B) |

For inference, **TP is the workhorse**: shard weights across 2–8 GPUs; latency grows slowly and NVLink makes the per-layer all-reduces cheap (H100 900 GB/s, B200 1.8 TB/s) — hence vLLM/TensorRT-LLM default to TP for big models. DP helps inference throughput when KV-cache capacity (not weight size) is the limit; PP is mostly a training technique. The serving question is "TP=1, 2, or 4 given model size, memory, and latency budget" — NVLink topology matters more than GPU count (§9.3).

### 6.7 Inference Engines: vLLM and TensorRT-LLM

- **vLLM** (open source, 2023): PagedAttention + continuous batching + prefix caching + quantization support (GPTQ/AWQ/FP8) + TP/PP. The default choice for self-hosted open-weight serving; huge ecosystem, OpenAI-compatible API, Python-first. For most banking use cases (RAG copilots, document Q&A) this is where you start.
- **TensorRT-LLM** (NVIDIA, 2023): TensorRT rebuilt for LLMs — FP8/INT4 kernels, paged KV cache, in-flight batching, CUDA graphs, C++ runtime. Typically the fastest on NVIDIA hardware (esp. H100/B200 with FP8), at the cost of more engineering (build engines per model/GPU, calibration, version-pinned stacks). 
- Others: TGI (HF), SGLang (fast prefix caching + radix attention), llama.cpp (CPU/edge) — full comparison in [ai_llm/llm_latency_optimization_guide.md §5](ai_llm/llm_latency_optimization_guide.md).

The engine choice is an *operational* decision as much as a performance one: vLLM's velocity and ecosystem vs TensorRT-LLM's ceiling. Both implement the same core ideas (§6.2–6.5), and both are the "GPU optimization" of the LLM world — you rarely touch CUDA here.

### 6.8 Speculative Decoding

**Speculative decoding** (Leviathan et al., arXiv Nov 2022 / ICML 2023 ✅; independently Chen et al., Feb 2023) attacks the *serial* nature of decoding: a small **draft model** proposes K tokens in one cheap pass; the **target model** verifies all K in parallel; accepted tokens are kept — **with a mathematically identical output distribution** (lossless!). Typical speedup: **2–3×**, larger when the target is big and the draft is good. Variants: self-drafting n-grams (no second model), Medusa, Eagle. Built into vLLM, TensorRT-LLM, TGI.

### 6.9 LLM Techniques Summary

| Technique | What it fixes | Typical speedup | Complexity | Cross-ref |
|---|---|---|---|---|
| FlashAttention(-2) (§6.2) | Attention HBM traffic, quadratic memory | 2–4× (vs baseline); FA2 ≈ 2× FA1 | Built-in | — |
| PagedAttention / vLLM (§6.3) | KV-cache fragmentation/waste | 2–4× serving throughput | Built-in (vLLM) | [llm_latency_optimization_guide.md §4](ai_llm/llm_latency_optimization_guide.md) |
| 4-bit quantization (GPTQ/AWQ) (§6.4) | Weight bytes per token | 2.5–4× memory, ~2–3× tokens/s | Low–Medium (tooling) | [llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md) |
| FP8 (§6.4) | Weight bytes + FLOP ceiling | ~2× vs FP16 on H100 | Medium | [on_prem_llm_deployment_guide.md §11](on_prem_llm_deployment_guide.md) |
| KV quantization / GQA (§6.5) | KV-cache memory | 2–4× context capacity | Low (config) | — |
| Tensor parallelism (§6.6) | Model > one GPU | scales with GPUs (sublinear) | Medium | [on_prem_llm_deployment_guide.md §4](on_prem_llm_deployment_guide.md) |
| Continuous batching (§6.3/§8) | GPU idle on short requests | 3–10× throughput | Built-in (engines) | [llm_latency_optimization_guide.md §4](ai_llm/llm_latency_optimization_guide.md) |
| Speculative decoding (§6.8) | Serial decode latency | 2–3× tokens/s | Medium | — |
| CUDA graphs (§5.2) | Launch overhead in decode steps | 1.2–1.5× small-step | Built-in (engines) | — |
| Prefix caching (§6.5) | Repeated context compute | 2–10× on shared-prefix workloads | Built-in (vLLM/SGLang) | — |

The ordering is deliberate: the top rows deliver LLM serving's orders of magnitude, and all are *memory-system* fixes — consistent with the §1.4 physics. Full playbooks: [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md) (latency), [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) (sizing), [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md) and [knowledge_distillation_guide.md](knowledge_distillation_guide.md) (compression).
---

## 7. The Practical Optimization Workflow

Techniques are a menu; the workflow is the recipe. The disciplined loop — **profile → identify → optimize → verify** — is the difference between "we tried things" and "we engineered a speedup." It is the same loop the CPU-side guide teaches for latency ([low_latency_cpp_development_guide.md §15](low_latency_cpp_development_guide.md)); the GPU version just has different instruments (§4).

### 7.1 The Methodology: Profile → Identify → Optimize → Verify

```
Profile (measure) → Identify (root cause) → Optimize (one fix) → Verify (re-measure) → repeat
```

Rules that make this loop work:

1. **Measure before you touch code.** Baseline: end-to-end latency/throughput *and* a profile (§4). "It feels slow" is not a baseline.
2. **One change at a time.** Techniques interact (fusion changes occupancy; precision changes both memory and compute). Parallel experiments corrupt attribution.
3. **Every optimization is a hypothesis** with a falsifiable prediction: "this kernel is memory-bound (SOL 80% memory / 20% compute) → tiling should improve it; if not, the hypothesis was wrong."
4. **Verify both correctness and performance** (§7.6). A 3× speedup that breaks outputs is a defect.
5. **Write it down** — what was tried, what the profile said, what changed. This builds institutional GPU knowledge (and justifies GPU spend to finance).

### 7.2 Roofline First

Before any optimization, determine the bound class (§3.1). The decision tree:

- **Compute (SM) utilization ≫ memory utilization** (e.g., SOL compute 90%, memory 30%) → compute-bound: reduce FLOPs (fast math, sparsity, algorithmic), increase FLOP rate (tensor cores, ILP), or raise the compute roof (higher-clocked part, more GPUs).
- **Memory utilization ≫ compute** → memory-bound: reduce bytes (fusion, tiling, precision, quantization), improve transaction efficiency (coalescing, vectorization).
- **Both low** (e.g., 40%/40%) → the kernel is *latency/occupancy-bound*: not enough warps to hide latency; fix occupancy (registers, block size) or launch overhead (graphs).
- **Both high** → you're near the roofline; the remaining gains are algorithmic (fewer ops) or architectural (better GPU).

This single classification — done in minutes with `ncu --set basic` — tells you *which chapter of §3 to read*. Skipping it is how teams spend weeks tuning compute on a memory-bound kernel.

### 7.3 Bottleneck Categories

| Category | Signature in profile | Typical causes | Primary fixes |
|---|---|---|---|
| Memory-bound | SOL memory ~100%, low SM busy | Uncoalesced access, no tiling, too many bytes (FP32), intermediate round trips | Coalescing, tiling, fusion, precision, quantization |
| Compute-bound | SOL compute ~100%, memory idle | FP32 math, no tensor cores, transcendental-heavy | Tensor cores, FP16/BF16/FP8, fast math, ILP, sparsity |
| Latency-bound (launch overhead) | GPU idle gaps in NSYS timeline, short kernels | Many tiny kernels, eager framework ops, syncs | CUDA graphs, fusion, torch.compile, batching |
| Occupancy-bound | Low achieved occupancy, stalls on memory | Register pressure, small blocks, too much shared memory/block | Register caps, block sizing, occupancy calculator |
| Transfer-bound (host side) | H2D/D2H copies dominate NSYS | Unpinned memory, per-step copies, CPU data pipeline | Pinned memory, async copies, on-GPU data, DataLoader tuning (§5.9) |
| Not-a-GPU-problem | GPU idle, CPU busy | Wrong tool: workload too small/serial for GPU | Stay on CPU; or hybrid (see §1.1, and [low_latency_cpp_development_guide.md](low_latency_cpp_development_guide.md)) |

The last row is the most underrated: many "GPU slow" tickets are workloads that should never have been on a GPU (tiny matrices, serial logic). Saying "no" (or "hybrid") is part of the expertise (§7.7).

### 7.4 Quick Wins (Cheap, Safe, High-ROI)

In rough order of effort:

1. **Compile flags**: `-O3` (nvcc), `-use_fast_math` where tolerance allows, `--ptxas-options=-v` to see register/spill usage.
2. **Mixed precision** (§5.3): autocast FP16/BF16 — one or two lines, 1.5–2× on most models.
3. **torch.compile** (§5.1): one line; 1.2–2× on transformer workloads; remember warmup.
4. **Pinned memory + async copies** (§5.9): `pin_memory=True`, `non_blocking=True` on `.to(device)`.
5. **Batch size / num_workers tuning** (§5.9): the cheapest throughput lever; measure the knee.
6. **CUDA graphs** (§5.2) for small-step/inference loops: `torch.compile(mode="reduce-overhead")` or explicit capture.
7. **Serving-level (LLMs)**: continuous batching + a PagedAttention engine (vLLM) instead of ad-hoc single-request serving — usually a 3–10× throughput jump before any kernel work; plus KV/context hygiene (prefix caching, GQA models, KV quantization, §6.5).

If none of these move the needle, the workload is probably in the "deep" bucket.

### 7.5 Deep Optimization

When quick wins are exhausted and the profile says a specific kernel is the wall:

1. **Kernel fusion** (§3.7) — at the framework level first (torch.compile/TensorRT), then custom fused kernels for the hot spot (e.g., a fused attention/softmax/linear).
2. **Custom CUDA kernels** (or Triton) for the dominant kernel: tiling (§3.3), vectorization (§3.6), bank-conflict padding (§3.4), register tuning (§3.5) — driven *by ncu metrics*, not vibes.
3. **TensorRT / TensorRT-LLM** (§5.5/§6.7) for inference: engine build + FP8/INT8 + autotuning. This is usually where the last 2× for production inference lives.
4. **Algorithmic change**: FlashAttention-style IO-awareness (§6.2) — restructure *what* is computed, not just how; speculative decoding (§6.8); sparsity.
5. **Distributed**: TP for big-model inference (§6.6), streams for pipeline overlap (§3.8).

Deep optimization is where the 5–50× cumulative gains live, and where the profile-driven discipline (§7.1) is non-negotiable — every step here can regress on a different architecture or shape.

### 7.6 Verification: Correctness and Performance

**Correctness.** Precision/quantization changes require numerical verification:

- Unit-test outputs against the FP32 reference with a tolerance (e.g., `torch.allclose(rtol=1e-2, atol=1e-3)` for FP16; looser for INT4).
- Validate *task-level* quality on a golden set: for an LLM, run your banking eval set (NER on legal text, extraction on statements, RAG answer correctness) before/after quantization — perplexity or loss deltas don't capture domain-specific regressions.
- For regulated outputs, keep an auditable record: model version + precision + calibration set + eval results.

**Performance.** Benchmarks must be:

- **Before/after on the same hardware** (A100-tuned kernels can regress on H100 and vice versa — kernels and cuDNN/cuBLAS heuristics differ).
- **Steady-state, not first-call**: warm up (compilation, cudnn autotune, caches), then measure N iterations; report median/p99, not min.
- **End-to-end and kernel-level**: a 2× kernel win with a CPU-bound pipeline shows ~0 wall-time gain — measure both.
- **Throughput *and* latency** (LLMs trade one for the other via batching) and **memory** (peak RSS/VRAM — quantization that saves memory but OOMs at peak is not a win).

### 7.7 When to Stop: Diminishing Returns and Amdahl

**Amdahl's law applies twice over.** First: if the kernel you're optimizing is 30% of runtime, a perfect 10× there is only ~2.5× end-to-end — re-profile and pick the next bottleneck. Second: within a kernel, the first tiling/fusion pass buys 2–5×; the fifth tuning pass buys 5%. Practical stopping rules:

- The kernel is within ~80–90% of its roofline ceiling (SOL) → the remaining gains are architectural.
- The next candidate bottleneck is <10% of runtime → the effort-to-reward ratio has flipped; move on (or move the workload).
- The optimization's *operational* cost (custom kernels to maintain, version-pinned engines, calibration pipelines) exceeds the hardware cost it saves — for a bank, maintenance risk and auditability are real line items.
- The bottleneck has moved to a place you can't affect from the GPU (network, CPU, data prep) → fix that instead, or redesign the split.

The professional pattern is **iterate the loop until the dominant cost moves somewhere cheaper**, then stop. Knowing when to stop is as much of the skill as knowing how to start.

### 7.8 The GPU Optimization Checklist

**Before (baseline):** [ ] latency/throughput/memory baseline (warm, steady-state) · [ ] NSYS profile (GPU busy %? dominant kernels? copy/gap time?) · [ ] roofline classification (memory / compute / latency / occupancy / transfer-bound?)

**Quick wins (each measured):** [ ] `-O3` + fast-math where safe (check `--ptxas-options=-v` for spills) · [ ] FP16/BF16 autocast + tolerance check · [ ] `torch.compile` (reduce-overhead for inference) with warmup · [ ] DataLoader (`num_workers`, `pin_memory`, `non_blocking`, prefetch) · [ ] batch-size/concurrency knee · [ ] vLLM-class engine with continuous batching (LLMs)

**Deep (profile-driven, one at a time):** [ ] NCU on the dominant kernel (SOL, occupancy, coalescing, bank conflicts, spills) · [ ] fusion → tiling → vectorization → register tuning · [ ] CUDA graphs for launch-overhead-bound loops · [ ] TensorRT(-LLM) with FP8/INT8 for production inference · [ ] for LLMs: KV quantization, prefix caching, speculative decoding, TP sizing

**After (verify):** [ ] correctness (numerical tolerance + task-level golden set) · [ ] performance re-measured (same hardware, warm, median/p99, before/after) · [ ] profile re-run — did the bound class change? · [ ] documented (what, why, delta, operational cost)

---

## 8. Worked Example: Optimizing PyTorch Transformer Inference

To make the workflow concrete, here is a full pass over a realistic scenario: serving a 7B-parameter open-weight transformer for a document-Q&A copilot on a single A100 80GB, with a target of maximizing throughput at a p95 latency under 500 ms. **Note: numbers below are illustrative typical ranges from public benchmarks, not a measured run on a specific stack** — the structure is the lesson, the digits are directional.

### 8.1 Baseline

- Serving: naive PyTorch, batch size 1 (one request at a time), FP32, eager mode, `DataLoader` with `num_workers=0`.
- Measured (warm, median): **~12 tokens/s**, p95 latency **~1,800 ms** for a 512-token prompt + 128-token answer, GPU utilization ~35% (spiky), 18 GB VRAM peak.

### 8.2 Profile (NSYS + NCU)

The NSYS timeline showed three things immediately:

1. **Large CPU-side gaps between kernels** — the eager launch pipeline couldn't keep the GPU fed (each decode step = hundreds of tiny kernels; launch overhead + Python dispatch). GPU idle ~40% of the time. → *latency-bound component.*
2. **H2D/D2H copies per step** for token I/O with unpinned buffers — small but serialized. → *transfer-bound component.*
3. **The GEMMs were running at ~30% of tensor-core peak** — the model was in FP32, so cuBLAS couldn't use tensor cores at all. NCU SOL on the attention matmuls: compute 28% / memory 55% — memory-leaning because FP32 doubled the bytes. → *precision + bandwidth component.*

Classification: **mixed — launch-overhead-bound and precision/memory-bound**, not compute-bound. (Per §7.2, "both low" → latency/occupancy fixes first; memory utilization above compute → byte reductions next.)

### 8.3 Optimizations (one at a time, each measured)

| Step | Change | Rationale |
|---|---|---|
| 1 | `torch.compile(model, mode="reduce-overhead")` | Fusion + CUDA-graph capture collapse hundreds of launches per step |
| 2 | BF16 autocast (weights cast once, `torch.set_default_dtype`) | Halves GEMM bytes; enables tensor cores (BF16 needs no loss scaling for inference) |
| 3 | Pinned, pre-allocated I/O buffers + CUDA graph replay | Static addresses enable graph capture; removes per-step copies |
| 4 | Switch serving to vLLM-style continuous batching (batch 8–32) | Amortizes per-request cost; fills GPU during decode; PagedAttention KV management |
| 5 | KV cache in FP8 (engine option) | 2× KV capacity for long contexts; enables bigger effective batches |

Note what was *not* done: no custom CUDA kernels, no TensorRT yet — the profile said the wall was orchestration and precision, and steps 1–3 addressed exactly that. TensorRT-LLM with FP8 was listed as the "if we need the last 2×" follow-up.

### 8.4 Results (illustrative)

| Metric | Baseline | After steps 1–3 | After step 4 (+batching) | After step 5 |
|---|---|---|---|---|
| Tokens/s (throughput) | ~12 | ~28 | ~95 (batch 16) | ~120 |
| p95 latency (single request) | 1,800 ms | 700 ms | 850 ms (batched) | 820 ms |
| GPU utilization | ~35% | ~70% | ~90% | ~92% |
| VRAM peak | 18 GB | 10 GB | 22 GB (larger batches) | 20 GB (KV FP8) |

The three phases map to the three bottleneck classes: launch-overhead (1.4× latency), precision/memory (2.3× throughput), batching (3.4× throughput at a controlled latency cost) — **~10× throughput and ~2.2× better p95 latency without writing a single CUDA kernel**. A hand-tuned TensorRT-LLM + FP8 + speculative-decoding stack buys another ~1.5–2× at much higher engineering cost — the §7.7 "when to stop" decision in action.

### 8.5 The Lesson

**Profile first, optimize second — and optimize in the order the profile dictates.** The intuitive move ("the model is slow → get a faster GPU / write CUDA") would have missed that the GPU was 65% idle: the fastest kernel in the world can't help a pipeline that never launches it. The three cheap, framework-level changes (compile, precision, batching) delivered the order-of-magnitude; kernel surgery would have been the *last* step, not the first. The same sequence applies to any framework workload: NSYS for the orchestration, NCU for the kernels, roofline for the priority order, and benchmark discipline for the verdict.

---

## 9. The Future: 2026 and Beyond

### 9.1 Blackwell and Blackwell Ultra

Blackwell (2024) and Blackwell Ultra (B300, 2025) define the current data-center generation: **192–288 GB HBM3e, 8 TB/s bandwidth, FP4 native support, and the 2-die design** (two reticle-limited dies per package connected by high-speed links). The 2026 fleet reality: A100 is the *legacy* tier (still everywhere, now cheap), H100 the *workhorse* tier, B200/B300 the *frontier* tier. Two procurement implications for a bank: (1) the B-series' memory and bandwidth are what unlock long-context and big-batch LLM workloads (see [on_prem_llm_deployment_guide.md §3](on_prem_llm_deployment_guide.md) for the sizing framework); (2) vendor "up to" TFLOPS quotes are sparsity-inflated — compare dense numbers and real workloads ([cloud_providers_guide.md](cloud_providers_guide.md) has the commercial side).

### 9.2 FP4 and the Precision Ladder

Blackwell's headline feature is **FP4** — the next rung after FP8, roughly 2× the FLOPs of FP8 and 4× of FP16 at the same clock. The trend is unmistakable: every generation adds a lower-precision tensor format (FP16 → TF32 → FP8 → FP4), because AI quality keeps holding up at lower precision (with ever-smarter quantization, §6.4). Expect: FP4 weight formats in the quantization toolchains (GPTQ/AWQ-style methods are already targeting it), FP4 KV caches, and "mixed-precision-by-default" serving stacks that choose FP8/FP4 per layer from calibration — the hardware has already moved; the software is catching up through 2026. The honest caveat: FP4's quality cliff is workload-dependent, and *validation* (§7.6) becomes the gating process — the same discipline banks already apply to model risk.

### 9.3 NVLink and Multi-GPU Fabric

**NVLink** (NVIDIA's GPU-to-GPU interconnect, launched 2016 with Pascal) has scaled relentlessly: 600 GB/s (A100) → 900 GB/s (H100) → **1.8 TB/s per GPU (B200)**. At the rack scale, **GB200 NVL72** binds 72 Blackwell GPUs with **130 TB/s** of NVLink fabric — effectively one giant GPU from a programming perspective. Consequences: (1) tensor parallelism across 8–72 GPUs becomes cheap enough that "one big model on many GPUs" is the default for frontier serving; (2) the programming model shifts from "GPU + PCIe" to "fabric of GPUs" — frameworks (vLLM, TensorRT-LLM, JAX) already abstract this; (3) for a bank, NVLink-topology-aware scheduling matters more than GPU count (a TP=8 job on 8 NVLink-connected GPUs vs 8 PCIe-isolated ones is a different order of performance — ask your cloud provider which you're renting).

### 9.4 CXL and Memory Pooling

**CXL (Compute Express Link)** — the cache-coherent interconnect for memory expansion and pooling — is the main non-NVIDIA trend. In 2026 it is mature on the *CPU* side (memory expansion, tiered memory, pooling across servers) and emerging on the *GPU* side. The honest read: CXL gives more VRAM-class *capacity* at DRAM-class speed — valuable for model loading and infrequent paths, useless for the per-token weight read that drives decode. Watch for CXL 3.x pools in 2026–2027 enterprise GPU servers; capacity lever, not a bandwidth fix (❓ evolving).

### 9.5 AI Factories: The GB200 NVL72 Rack

NVIDIA's 2026 positioning is the **"AI factory"**: the **GB200 NVL72** — one rack with 72 Blackwell GPUs + 36 Grace CPUs, ~1.4 exaFLOPS FP4, 130 TB/s NVLink, liquid-cooled (✅ verified). Procurement shifts from "how many GPUs" to "how many racks," and *power/cooling* becomes the binding constraint (~120+ kW per rack — most data centers need retrofits; see [on_prem_llm_deployment_guide.md §7](on_prem_llm_deployment_guide.md)). The 2026 question is not which GPU but which *power envelope* you can provision.

### 9.6 Quantization Everywhere

The trajectory is unambiguous: FP8 for H100-class training, FP4/INT4 for inference, KV caches in FP8, calibration-driven mixed precision as the default. Combined with §9.2's precision ladder: **the model you serve is not the model you trained** — a quantized, pruned, distilled variant ([ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md), [knowledge_distillation_guide.md](knowledge_distillation_guide.md)) deployed at the precision the workload tolerates. The problem shifts from "make kernels fast" to "choose the cheapest representation that passes validation" — *bytes, not FLOPs, are the currency* (§1.4).

### 9.7 GPU Alternatives: AMD, Intel, Custom

NVIDIA's dominance is real but not a monopoly, and 2026's alternatives are credible:

- **AMD Instinct MI300X** (CDNA3, 2023): 192 GB HBM3, 5.3 TB/s, ~1.3 PFLOPS FP16 — the memory-capacity/bandwidth leader per dollar; MI350/MI355X (CDNA4, 2025) keep closing the ROCm software gap (❓ reported).
- **Intel Gaudi 3** (2024): 128 GB HBM2e, 3.7 TB/s — value tier; software ecosystem is the constraint.
- **Google TPU** (custom): v7 Ironwood (2025) — 192 GB HBM3e, ~7.4 TB/s, native FP8; only via Google Cloud; the *system* (pod-scale, 9k+ chips) is the product.
- **Others**: AWS Trainium/Inferentia (price-optimized, tied to Bedrock/SageMaker), Groq LPU and Cerebras (inference-latency specialists).

The strategic read: NVIDIA's moat is the *software stack* (CUDA, cuDNN, TensorRT, the Nsight tooling of §4 — see [gain_nvidia_experience.md §8](gain_nvidia_experience.md) for the skills angle), not just silicon. AMD's hardware is increasingly competitive, but the frameworks-optimization layer (§5) is built CUDA-first. Multi-vendor strategies (NVIDIA for training, AMD/TPU for inference at scale) are viable in 2026, with ROCm/JAX/ONNX as the portability layer.

### 9.8 Trends Summary

| Trend | Status 2026 | Impact on optimization practice |
|---|---|---|
| Blackwell/B300 deployment | Mainstreaming | Dense-number comparison discipline; FP8/FP4 default |
| Precision ladder (FP4) | Early adoption | Quantization toolchains + validation become core workflow |
| NVLink fabrics (NVL72) | Ramp | TP-across-fabric programming; topology-aware scheduling |
| CXL memory pooling | CPU-side mature, GPU-side emerging | Capacity lever only; not a bandwidth fix |
| AI factories / power | Binding constraint | Workload consolidation, power-aware capacity planning |
| Quantization everywhere | Default | Bytes-are-currency mindset; calibration pipelines |
| AMD/TPU alternatives | Credible | CUDA-first vs portability trade-offs; multi-vendor |

---

## 10. Glossary

**GPU (Graphics Processing Unit)** — a massively parallel processor originally for graphics, now the standard accelerator for AI and HPC; NVIDIA/AMD/Intel data-center parts are the enterprise norm.

**SM (Streaming Multiprocessor)** — the GPU's fundamental compute unit: 128 FP32 CUDA cores, tensor cores, warp schedulers, registers, and shared memory; the unit onto which thread blocks are scheduled.

**CUDA core** — the basic 32-bit ALU in an SM; "CUDA cores" (SMs × 128) is the marketing headline for compute width.

**Warp** — 32 threads that execute one instruction at a time in lockstep; the hardware unit of scheduling and of memory transactions.

**SIMT (Single Instruction, Multiple Threads)** — the GPU execution model: one instruction drives 32 data lanes; divergence serializes branch paths.

**Kernel** — a device function (CUDA: `__global__`) launched to run on the GPU across many threads.

**Grid** — the full set of thread blocks launched by one kernel call; 1D/2D/3D.

**Block (thread block)** — a group of up to 1,024 threads running on one SM, able to share shared memory and synchronize.

**Thread** — the smallest execution unit; one lane of a warp with its own registers.

**threadIdx / blockIdx / blockDim** — built-in variables giving a thread's index within its block, its block's index within the grid, and the block dimensions; the basis of global indexing (`blockIdx.x * blockDim.x + threadIdx.x`).

**Shared memory** — on-chip SRAM per SM, shared by a block, ~20–30 cycles; the user-managed cache central to tiling.

**Global memory** — the GPU's main memory (HBM/VRAM), device-wide, ~400–800 cycles; the bandwidth bottleneck for most workloads.

**HBM (High Bandwidth Memory)** — stacked-DRAM technology behind GPU memory: A100 2.0 TB/s, H100 3.35 TB/s, B200 8 TB/s.

**Register** — per-thread, on-chip storage (~1 cycle); the fastest memory; register count caps occupancy.

**L1/L2 cache** — on-chip caches: L1 shares SRAM with shared memory per SM; L2 (40–50 MB) is shared device-wide.

**Coalescing** — the requirement that consecutive threads access consecutive addresses so a warp's access costs one 128-byte transaction.

**Occupancy** — active warps per SM as a fraction of the maximum (2,048 threads); the measure of latency-hiding capacity.

**Bank conflict** — two threads of a warp hitting the same of 32 shared-memory banks in one access, serializing it; fixed with padding.

**Tile** — a block of data staged in shared memory so it can be reused across many computations (the basis of tiled GEMMs and FlashAttention).

**Grid-stride loop** — a loop where threads advance by the whole grid size each iteration, decoupling grid size from data size; the persistent-kernel idiom.

**Kernel fusion** — merging multiple kernels/ops into one to eliminate intermediate global-memory round trips.

**CUDA stream** — an ordered queue of work; kernels in different streams run concurrently; the basis of copy/compute overlap.

**Async memcpy** — a copy (`cudaMemcpyAsync`) executed by a DMA copy engine without occupying SMs; requires pinned host memory.

**CUDA event** — a timestamp marker in a stream, used for precise GPU timing and stream synchronization.

**CUDA graph** — a captured, replayable sequence of kernel launches executed with one launch, eliminating per-kernel CPU overhead.

**Nsight Compute (ncu)** — NVIDIA's kernel-level profiler: SOL analysis, occupancy, memory-workload, stall reasons.

**Nsight Systems (nsys)** — NVIDIA's system-level profiler: full application timeline, CPU/GPU overlap, copies, launch gaps.

**nvprof** — the legacy CUDA profiler; deprecated and removed from CUDA 13.0; superseded by Nsight tools.

**ncu / nsys** — the command-line interfaces of Nsight Compute / Nsight Systems.

**Roofline model** — the performance model plotting attainable FLOPS vs arithmetic intensity; classifies kernels as compute- or memory-bound.

**Arithmetic intensity** — FLOPs performed per byte moved from memory; the roofline x-axis.

**torch.compile** — PyTorch 2.0's graph compiler: fuses ops and generates fast kernels; 1.2–2× typical.

**AMP (Automatic Mixed Precision)** — running ops in FP16/BF16 with FP32 master weights/accumulation (PyTorch: autocast + GradScaler).

**Mixed precision** — using multiple precisions in one model to trade accuracy for speed/memory.

**bfloat16 (BF16)** — 8-bit-exponent/7-bit-mantissa format with FP32 range; no loss scaling needed; A100+.

**TensorRT** — NVIDIA's inference engine: builds device-specific optimized plan files with fusion, autotuning, INT8/FP8.

**FlashAttention** — IO-aware tiled attention (Dao et al., 2022) that keeps attention in SRAM; 2–4×; FA2 (2023) ~2× faster.

**PagedAttention** — KV-cache paging inspired by virtual memory (Kwon et al., SOSP 2023); the core of vLLM.

**vLLM** — the open-source high-throughput LLM serving engine built on PagedAttention + continuous batching.

**KV cache** — per-request stored key/value activations; the dominant memory cost in long-context serving.

**GPTQ** — 4-bit post-training weight quantization using second-order error correction (Frantar et al., 2022).

**AWQ** — activation-aware weight quantization protecting ~1% salient weights (Lin et al., 2023).

**FP8** — 8-bit float (E4M3/E5M2), hardware-native on H100+; 2× tensor throughput vs FP16.

**Tensor parallelism (TP)** — sharding each layer's weights across GPUs with per-layer all-reduce; needs NVLink.

**Pipeline parallelism (PP)** — assigning layer ranges to different GPUs; activations flow between stages.

**Data parallelism (DP)** — replicating the model and sharding the batch; gradients all-reduced (training).

**Speculative decoding** — a draft model proposes tokens the target verifies in parallel; lossless 2–3× decode speedup.

**NVLink** — NVIDIA's GPU-to-GPU interconnect: 900 GB/s (H100), 1.8 TB/s (B200), 130 TB/s rack fabric (NVL72).

**GB200 / NVL72** — Blackwell's flagship rack: 72 B200 GPUs + 36 Grace CPUs, ~1.4 exaFLOPS FP4, one NVLink domain.

**B200 / H100 / A100** — the current NVIDIA data-center GPU generations (Blackwell 2024, Hopper 2022, Ampere 2020).

**Tensor core** — the SM's matrix-multiply unit; 1st gen Volta 2017; 3rd gen Ampere; 4th gen Hopper (FP8); 5th gen Blackwell (FP4).

---

## 11. References and Companion Guides

**Companion guides in this repository** (cross-referenced throughout):

- [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) — GPU sizing, memory-bandwidth math, deployment (§3–§5, §11)
- [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md) — LLM latency engineering, serving engines, cost-latency (§3–§5, §13)
- [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md) — pruning + quantization pipeline (§10)
- [knowledge_distillation_guide.md](knowledge_distillation_guide.md) — distillation as model compression (§3)
- [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md) — PyTorch/JAX/TensorFlow comparison (§3–§4, §6)
- [low_latency_cpp_development_guide.md](low_latency_cpp_development_guide.md) — CPU-side performance mindset, profiling, benchmarking (§12–§15)
- [cloud_providers_guide.md](cloud_providers_guide.md) — GPU instances and cloud procurement
- [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) — ML platforms running on GPU clusters
- [gain_nvidia_experience.md](gain_nvidia_experience.md) — hands-on NVIDIA/CUDA skill-building (§8)

**Primary sources (verified August 2026):**

- NVIDIA datasheets: A100 (2 TB/s HBM2e, 312 TFLOPS FP16 dense, 3rd-gen TC), H100/H200 (3.35 TB/s HBM3, FP8 Transformer Engine; 141 GB / 4.8 TB/s H200), GB200 NVL72 (72 GPUs, ~1.4 exaFLOPS FP4, 130 TB/s NVLink) — nvidia.com; NVIDIA developer forum: nvprof/nvvp deprecated, removed in CUDA 13.0
- Dao et al., *FlashAttention* arXiv 2205.14135 (2022); *FlashAttention-2* arXiv 2307.08691 (2023); Kwon et al., *PagedAttention*, SOSP 2023 (arXiv 2309.06180)
- Frantar et al., *GPTQ* arXiv 2210.17323 (2022); Lin et al., *AWQ* arXiv 2306.00978 (2023); Dettmers et al., *LLM.int8()* arXiv 2208.07339 (2022)
- Leviathan et al., *Fast Inference from Transformers via Speculative Decoding* arXiv 2211.17192 (2022, ICML 2023); Chen et al. arXiv 2302.01318 (2023)
- Williams, Waterman & Patterson, *Roofline*, CACM 2009; PyTorch 2.0 / torch.compile release notes (Dec 2022 → March 2023 stable)
- GPU spec cross-checks: Pantheon, VESSL AI, Verda, TechPowerUp (dense-vs-sparse TFLOPS for Blackwell parts)

> *Companion performance-series guide: [low_latency_cpp_development_guide.md](low_latency_cpp_development_guide.md) · LLM engineering series: [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md) · [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md) · deployment: [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md)*
