# Custom Checkpoint Systems: A Comprehensive Guide to Checkpoint Engineering for ML Training at Scale

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore  
> **Context:** AI Infrastructure / MLOps — Distributed Training, Fault Tolerance, Storage Engineering, LLM Training   
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** August 2026

---

> A comprehensive guide to the engineering discipline of checkpointing — saving and loading training state — for machine learning at scale. Covers why checkpointing matters (fault tolerance, resumability, the economics of lost GPU-hours), what a checkpoint actually contains (weights, optimizer state, RNG, dataloader, scheduler), the serialization formats (PyTorch pickle, safetensors, Orbax, custom schemas), the save strategies (periodic, async, sharded, distributed, crash-consistent), the orchestration layer (managers, retention, stores, registries, atomicity), the scale challenges (checkpoint bandwidth, the checkpoint tax, spot-instance preemption, auto-resume, restore testing), the LLM-era reality (100GB+ checkpoints, LoRA fine-tuning, released model weights), the tools landscape (PyTorch DCP, Lightning, Transformers Trainer, DeepSpeed ZeRO, Megatron, Orbax, W&B/MLflow), and a full build-your-own design with a worked 70B fine-tuning example.

**Audience**: Solution architects, ML platform engineers, MLOps leads, and quant/infra developers who run training workloads — from a single GPU to a 1000-GPU cluster — in an enterprise or banking context. Assumes working Python/PyTorch knowledge and basic systems concepts (filesystems, object storage, processes, networks). The banking context (Crédit Agricole CIB, Singapore) frames the business value: training time is a budget line item, GPU hours are money, and a checkpoint is insurance against both hardware failure and wasted spend.

---

## Table of Contents

1. [Checkpointing Fundamentals](#1-checkpointing-fundamentals)
2. [Checkpoint Formats](#2-checkpoint-formats)
3. [Checkpointing Strategies](#3-checkpointing-strategies)
4. [Checkpoint Orchestration](#4-checkpoint-orchestration)
5. [Checkpointing at Scale](#5-checkpointing-at-scale)
6. [LLM-Era Checkpointing](#6-llm-era-checkpointing)
7. [Tools and Frameworks](#7-tools-and-frameworks)
8. [Build Your Own: A Custom Checkpoint System](#8-build-your-own-a-custom-checkpoint-system)
9. [Worked Example: A Checkpoint System for a 70B LLM Fine-Tune](#9-worked-example-a-checkpoint-system-for-a-70b-llm-fine-tune)
10. [The Future: 2026 and Beyond](#10-the-future-2026-and-beyond)
11. [Glossary](#11-glossary)
12. [References and Companion Guides](#12-references-and-companion-guides)

---

### How to Read This Guide

**Relationship to sibling guides.** This is the dedicated deep-dive on *checkpoint engineering* in the ML infrastructure series, and it deliberately cross-references rather than duplicates its siblings:

- **GPU hardware and memory** (why optimizer state costs what it costs, NVMe/CXL staging) → [gpu_optimization_guide.md](gpu_optimization_guide.md)
- **PyTorch/JAX/TensorFlow framework internals** (state dicts, device placement) → [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md)
- **LoRA/QLoRA fine-tuning** (tiny adapter checkpoints vs full fine-tune checkpoints) → [ai_llm/fine_tuning_frameworks_comparison_guide.md](ai_llm/fine_tuning_frameworks_comparison_guide.md)
- **The storage layer for checkpoints** — NFS, Lustre, BeeGFS, CephFS, object storage → [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md)
- **Checkpoints as governed data** (sensitivity, retention, access control) → [data_governance_guide.md](data_governance_guide.md)
- **Turning a checkpoint into a deployed model** (sharded inference, vLLM) → [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) and [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md)

**Suggested reading paths.** New to checkpointing: §1 → §2 → §3 → §4. PyTorch distributed user: §3.5–3.6 → §5 → §7.1. Platform engineer building the system: §4 → §5 → §8 → §9. LLM fine-tuning lead: §6 → §7 → §9. Storage engineer: §2 → §4.3 → §5.2.

**Note on verification.** Facts were researched in August 2026. Claims are marked inline: ✅ **Verified** (confirmed against primary sources or multiple consistent sources during research), ⚠️ **Approximate** (widely cited but configuration-, architecture-, or workload-dependent), ❓ **Reported** (vendor marketing or single-source numbers; treat with care). A consolidated claims-status table follows the TOC. Anything that could not be verified is flagged honestly rather than asserted — unverifiable claims are explicitly marked **⚠️ Not fully verifiable**.

### Claims Status at a Glance

| Claim | Status | Note |
|---|---|---|
| Adam optimizer state ≈ 2× model size (two moments, each the size of the model) | ✅ Verified | PyTorch docs: Adam state O = 2P; moments conventionally FP32 (4 B/param) |
| 70B model weights in BF16/FP16 ≈ 140 GB (70B × 2 B) | ✅ Verified | Standard arithmetic; multiple independent sources |
| 70B full mixed-precision checkpoint (weights + FP32 Adam moments + FP32 master weights) ≈ 1 TB | ✅ Verified | 140 GB + 560 GB + 280 GB; sources differ on whether master weights are persisted |
| "~280 GB optimizer state" (2 × weights at BF16 size) holds only if moments are stored in BF16/FP16 | ✅ Verified | FP32 moments = 560 GB; BF16 moments = 280 GB — the widely-cited 280 GB assumes reduced precision moments |
| 175B model weights ≈ 350 GB in FP16 | ✅ Verified | 175B × 2 B arithmetic (GPT-3-class) |
| `torch.save` uses pickle; arbitrary code execution risk on untrusted files | ✅ Verified | PyTorch docs; CVE-adjacent advisories; HF blog |
| safetensors: no pickle, JSON header + raw tensor bytes, zero-copy fast | ✅ Verified | safetensors repo/docs; HF "Risk of Pickle" blog |
| HuggingFace `save_pretrained` writes `pytorch_model.bin` (pickle) or `model.safetensors` | ✅ Verified | transformers docs; safetensors is default when installed |
| PyTorch DCP (`torch.distributed.checkpoint`): distributed, sharded save/load; mature by PyTorch 2.x | ✅ Verified | PyTorch docs/recipes; "mature support" cited for 2.9 |
| Orbax: JAX checkpointing library with async + multihost distributed checkpointing | ✅ Verified | google/orbax repo; Orbax docs (2023+) |
| DeepSpeed ZeRO checkpoints are partitioned per-rank; Universal Checkpointing converts across configs | ✅ Verified | DeepSpeed docs/tutorials |
| Megatron-LM `dist_checkpointing`: sharded, reshardable across TP/PP/DP configs, async save | ✅ Verified | Megatron-LM docs/DeepWiki |
| Lightning `ModelCheckpoint`: monitor/mode/save_top_k/save_last → `last.ckpt`/`best.ckpt` | ✅ Verified | Lightning docs |
| HF `Trainer`: `save_strategy`, `save_steps`, `save_total_limit`, `load_best_model_at_end` | ✅ Verified | transformers `TrainingArguments` docs |
| `torchrun` elastic: `--nnodes=1:4`, `--max_restarts`, rendezvous; restarts all procs, each loads last snapshot | ✅ Verified | PyTorch elastic docs + fault-tolerance tutorial |
| Checkpoint-free training is active research (CheckFree, arXiv 2506.15461, June 2025; PyTorch "Fault Tolerant Llama" blog) | ✅ Verified | arXiv + pytorch.org blog (June 2025) — not "impossible", but early |
| Typical checkpoint cadence: every 1,000–5,000 steps or every 1–2 hours | ⚠️ Approximate | Common practice per multiple training guides; varies widely |
| "Checkpoint tax" of 2–10% of training time with naive synchronous saves | ⚠️ Approximate | Workload-dependent; async + sharding push it toward 0–2% |
| SIGTERM handler → graceful checkpoint on preemption | ✅ Verified | Standard POSIX signal handling; used by SLURM/cloud preemption flows |
| Temp-then-rename gives single-file atomicity; multi-file needs manifest/commit ordering | ✅ Verified | Standard filesystem semantics (POSIX rename) |
| 4-bit / FP8 checkpoint quantization is lossy but used for storage reduction | ⚠️ Approximate | Reported in practice (e.g., QLoRA-style storage); precision loss is quantization-dependent |

---

## 1. Checkpointing Fundamentals

### 1.1 Why Checkpointing Matters

Checkpointing is the practice of periodically persisting the complete state of a training run — model weights, optimizer state, and everything else needed to resume — to durable storage. It is the single most important reliability mechanism in ML training, and it exists for one overriding reason: **training runs are long, and failures are inevitable.**

**Training runs are long.** A large-model training run lasts weeks to months. A 70B-parameter LLM pretraining run at frontier scale consumes millions of GPU-hours; even a "small" enterprise fine-tune runs for hours to days. At that timescale, the probability that *something* fails approaches 1. The relevant question is never "will this run fail?" but "when, and how much progress will we lose?"

**Failures are inevitable.** The failure modes are diverse and all of them are common at scale:

- **Node failures** — hardware faults (GPU hangs, ECC errors, DIMM failures, NIC drops, disk failures). On a 1000-GPU cluster with an annualized node failure rate of even 2–5%, a multi-week run is virtually guaranteed to see at least one failure. NVIDIA and hyperscaler experience reports MTBF (mean time between failures) figures that make multi-month uninterrupted runs statistically impossible.
- **OOM (out-of-memory)** — a slightly larger batch, a longer sequence, a memory leak in the dataloader, or a bad configuration change can kill a process mid-run.
- **Preemption** — on cloud spot/preemptible instances (抢占式实例), the hypervisor can reclaim the machine with a few minutes' (or seconds') warning. Spot pricing is 60–90% cheaper than on-demand, which makes preemption a *planned* event in cost-conscious training — and checkpointing the enabler of that cost strategy (see §5.5).
- **Software faults** — bugs, deadlocks, NCCL timeouts, data pipeline crashes, filesystem stalls (an NFS hang can freeze an entire job).
- **Operator error** — wrong hyperparameters, killed jobs, cluster maintenance windows, power events.

**Resumability.** A checkpoint converts a catastrophic failure into a minor delay. Without checkpoints, a failure at day 20 of a 30-day run means restarting from scratch: 20 days of GPU-hours (hundreds of thousands of dollars) are lost, *and* the training clock restarts. With a checkpoint taken every hour, the same failure costs at most one hour of recomputation plus the time to restore. Checkpointing is the difference between "the run failed" and "the run was interrupted."

**The cost of lost training — the checkpoint as insurance.** The economics are brutal and simple:

```
Lost GPU-hours ≈ (time since last checkpoint) × (number of GPUs) × (hourly GPU cost)
```

For a 512-GPU run of H100-class hardware at ~$2–4/GPU-hour (on-demand cloud, 2026 pricing), *one hour* of lost training costs roughly **$1,000–2,000**. A lost day costs $25,000–50,000. Checkpoint storage, by contrast, costs a few dollars per TB-month on object storage. Checkpointing is the cheapest insurance policy in ML infrastructure: it trades a few percent of training throughput (the "checkpoint tax", §5.4) and a modest storage bill for the elimination of catastrophic loss. The ROI calculation is unambiguous — and it is why every serious training framework ships checkpointing built in, and why mature ML platforms treat it as a first-class engineering discipline rather than an afterthought.

There is a second, quieter reason checkpointing matters: **checkpoints are the only durable artifact of a run.** They are what you keep when the cluster is torn down, what you fine-tune from, what you release, and what you audit. A training run without checkpoints is not just fragile — it is ephemeral.

### 1.2 What a Checkpoint Contains

A checkpoint is a snapshot of the **training state** — everything needed to continue training *exactly* (or near-exactly) from the saved point. The standard components are:

**1. Model weights (the parameters).** The model's parameters — the tensors that the optimizer updates. For a 7B model in FP16/BF16 this is ~14 GB; for 70B, ~140 GB. With parameter-efficient fine-tuning (LoRA/QLoRA, see [ai_llm/fine_tuning_frameworks_comparison_guide.md](ai_llm/fine_tuning_frameworks_comparison_guide.md)) the *trainable* weights are tiny adapters (~0.1–1% of the model), which is exactly why LoRA checkpoints are cheap.

**2. Optimizer state.** The optimizer's internal bookkeeping, without which the next optimizer step would be wrong. For **Adam** (the default for LLM training), this is the famous **2× model size** rule: Adam maintains two moments — the first moment (mean of gradients, the "momentum" term) and the second moment (mean of squared gradients, the "variance" term) — each the size of the model. ✅ **Verified** (PyTorch docs: Adam state O = 2P). Two subtleties:

- **Precision.** Moments are conventionally stored in FP32 even when weights are FP16/BF16. At 4 bytes/parameter each, two FP32 moments cost **8 bytes/parameter** — for a 70B model that is **560 GB** of optimizer state alone. ⚠️ If moments are stored in BF16/FP16 (an increasingly common memory-saving choice), the optimizer state halves to ~280 GB. The widely-cited figure "optimizer state ≈ 2× the model size" is exact in the *same-precision* case (2 × FP32 model size = 8 B/param for FP16-trained models).
- **Master weights.** Mixed-precision training keeps an FP32 copy of the weights ("master weights") for the optimizer step, since FP16/BF16 has too little precision for the small updates near convergence. That is another 4 B/param (~280 GB at 70B). ✅ Whether master weights are *persisted* in the checkpoint is implementation-dependent (some runs reconstruct them from the BF16 weights on resume — lossy; others persist them).

The practical consequence: for dense Adam mixed-precision training, the **checkpoint is dominated by optimizer state, not weights**. A 70B checkpoint is ~140 GB of weights but ~700 GB–1 TB total. This single fact drives almost every design decision in this guide — sharding (§3.5), async saves (§3.4), compression (§8.4), and storage costs (§9.3).

**3. RNG state.** The state of every random number generator used in training: the PyTorch/NumPy global generators, per-device CUDA generators (important for dropout), and any custom generators. Without them, resume is not reproducible — dropout masks, data shuffling, and initialization would differ from what would have happened in the un-interrupted run. ✅ This matters more than most engineers assume: the difference between an exact resume and a "close enough" resume is precisely the RNG state.

**4. Dataloader state.** The position in the data stream: the sampler state, current epoch, batch/step index, and (for sharded/streaming datasets) per-shard positions. Without it, resume either re-reads data (wasted work) or mis-orders the epoch (silent data-distribution drift). For iterable datasets and sharded data (e.g., WebDataset/tar shards), the per-rank shard offsets must be saved too. ⚠️ Dataloader state is the most frequently *omitted* component in hand-rolled checkpoints — and the most common cause of "resume works but results drift."

**5. Scheduler state.** The learning-rate schedule's position: current LR, step counter, warmup state, any cyclic/cosine phase bookkeeping. Adam's moments are tuned to a specific LR trajectory; resuming with a wrong LR (e.g., restarting the schedule from zero after a restart) is a classic source of training divergence after resume. ✅

**6. Metadata.** Step number, epoch, wall-clock timestamp, best metric values, and (in distributed runs) the parallel configuration (world size, parallelism layout — see §3.5). Metadata is small but essential: it is what the registry (§4.4), retention policies (§4.2), and restore logic (§4.7) run on.

### 1.3 The State Dict

PyTorch's `state_dict` is the canonical container for this state: a plain Python `dict` mapping names → tensors. `model.state_dict()` yields `{"layers.0.self_attn.q_proj.weight": tensor(...), ...}` — every parameter and registered buffer, keyed by module path. ✅ `optimizer.state_dict()` similarly maps parameter indices → per-parameter optimizer tensors (for Adam: the two moments, plus the step counter). The `state_dict` is the universal interchange unit: `torch.save(model.state_dict(), ...)` on save, `model.load_state_dict(...)` on load. Everything else in this guide — formats, sharding, distributed checkpointing — is an elaboration of "how do we serialize and distribute *these dicts* efficiently and safely."

### 1.4 Checkpoint Anatomy

```
┌──────────────────────────── CHECKPOINT (training state) ───────────────────────────┐
│                                                                                    │
│  ┌───────────────┐  ┌────────────────────────────┐  ┌──────────────────────────┐  │
│  │ MODEL WEIGHTS │  │ OPTIMIZER STATE (Adam)     │  │ SCHEDULER STATE          │  │
│  │ 70B × 2 B     │  │ 1st moment: 70B × 4 B FP32 │  │ current LR, step count,  │  │
│  │ = 140 GB BF16 │  │ 2nd moment: 70B × 4 B FP32 │  │ warmup phase, schedule   │  │
│  │ (per-rank     │  │ = 560 GB (8 B/param)       │  │ params (tiny)            │  │
│  │  shards)      │  │ (+ master weights 280 GB)  │  └──────────────────────────┘  │
│  └───────────────┘  └────────────────────────────┘  ┌──────────────────────────┐  │
│  ┌───────────────┐  ┌────────────────────────────┐  │ DATALOADER STATE         │  │
│  │ RNG STATE     │  │ STEP / EPOCH METADATA      │  │ sampler state, epoch,    │  │
│  │ torch, cuda,  │  │ step, epoch, timestamp,    │  │ batch index, shard       │  │
│  │ numpy RNGs    │  │ best metric, world size    │  │ offsets, seed            │  │
│  └───────────────┘  └────────────────────────────┘  └──────────────────────────┘  │
│                                                                                    │
│  Serialization: pickle (torch.save) │ safetensors │ custom schema │ sharded files  │
└────────────────────────────────────────────────────────────────────────────────────┘
```

The relative sizes matter: for a dense 70B Adam fine-tune, weights are ~15% of the checkpoint, optimizer state ~60–85%, and everything else (RNG, dataloader, scheduler, metadata) is kilobytes. "Checkpoint size" conversations are really "optimizer state size" conversations.

---

## 2. Checkpoint Formats

### 2.1 Framework-Native Formats

**PyTorch: `torch.save` / `.pt` / `.pth`.** The default PyTorch serialization is `torch.save(obj, f)` → pickle-based serialization of arbitrary Python objects (with tensor payloads). ✅ It is trivially easy (`torch.save(model.state_dict(), "ckpt.pt")`), handles arbitrary Python structures, and is the lingua franca of PyTorch. Its problems are well documented:

- **Security: pickle executes arbitrary code on load.** Loading an untrusted `.pt`/`.pth`/`.bin` file can run any Python code the attacker embedded in it. ✅ Verified — this is the "risk of pickle" (HF blog) and a real supply-chain vector: model weights are downloaded from hubs, shared in teams, and pulled from the internet. A checkpoint is an executable, not just data.
- **Performance: single-file, CPU-serialized, memory-hungry.** `torch.save` pickles through Python, doesn't use the fast paths for pure-tensor data, and deserialization first reconstructs the object graph in memory. For a 1 TB checkpoint this is slow, and on distributed runs you must gather state to rank 0 first (see §3.5) — the "save bottleneck" of naive distributed checkpointing.
- **Fragility across versions.** Pickle schemas drift across PyTorch versions; old checkpoints occasionally fail to load after upgrades.

**safetensors (HuggingFace).** A safe, fast tensor serialization format: a small JSON header (tensor names, shapes, dtypes, offsets) followed by the raw tensor bytes, with an explicit length prefix that makes loading safe and **zero-copy** (memory-map the file, point tensors at it). ✅ **Verified** — no pickle, no code execution; "impossible to embed executable code" (AI Wiki; safetensors repo). It is also fast: byte-for-byte identical layout means loading is essentially a memmap, and parallel loading across files is trivial. Since transformers v4.30+ (2023), `save_pretrained` writes `model.safetensors` by default when the `safetensors` package is installed; ✅ otherwise it falls back to `pytorch_model.bin` (pickle). ✅ **Verified.** Limitation: safetensors serializes *tensors* only — no arbitrary Python objects — so optimizer state, RNG, and scheduler state need a companion format (e.g., a small pickle/JSON sidecar or a custom handler). For model weights, safetensors is now the de-facto standard and the basis of HF Hub distribution (§6.4).

**HuggingFace `save_pretrained` checkpoints.** The transformers convention: a checkpoint is a *directory* containing `model.safetensors` (or `pytorch_model.bin`), `config.json`, tokenizer files (`tokenizer.json`, `tokenizer_config.json`), and optionally `optimizer.pt` / `scheduler.pt` / `trainer_state.json` when saved by the `Trainer` (§7.3). ✅ The directory-as-checkpoint convention (config + weights + metadata) is itself a design worth copying — it is self-describing and versionable.

**JAX: Orbax.** Orbax (`orbax.checkpoint`) is the JAX-native checkpointing library, developed by Google (2023+). ✅ **Verified.** It provides `CheckpointManager` (save/restore with retention), **asynchronous checkpointing** (overlap save with training), multihost/multislice distributed checkpointing, PyTree-based state serialization, and pluggable storage backends. ✅ Orbax's async mode is the reference implementation of the async pattern (§3.4), and its design (checkpoint manager + async + retention + storage abstraction) is effectively the template for the custom system in §8. JAX training state is a *PyTree* (nested dicts/tuples/lists of arrays), which serializes naturally; there is no "state_dict" ceremony.

**TensorFlow: `.ckpt` and SavedModel.** TF's classic checkpoint format is the `checkpoint` + `index` + `data-00000-of-00001` file trio (the "v2" format) written by `tf.train.Checkpoint`/`ModelCheckpoint` — sharded by design, with an index file mapping tensor names to byte offsets. ✅ TensorFlow also exports `SavedModel` (a directory with the model graph + variables + assets) for serving. The v2 checkpoint format's design (index + sharded data files, atomic via `checkpoint` pointer file) is worth studying: TF solved "sharded checkpoint with a manifest" years ago, and the pattern recurs in DCP, Megatron, and custom systems (§4.5).

### 2.2 Custom Formats

**Custom checkpoint schemas.** Once you outgrow framework defaults — or need to store non-framework state (rank-specific data, pipeline stage bookkeeping, cluster metadata) — you design your own schema: a directory layout (`step-12345/` containing `model-*.safetensors`, `optimizer-*.safetensors`, `rng.json`, `dataloader.json`, `scheduler.json`, `metadata.json`), a manifest file listing every file with its checksum and size, and a version field. The version field is non-negotiable:

**Checkpoint versioning / schema evolution.** Checkpoints outlive code. A checkpoint written by training code v1.0 must load in v1.3, and ideally in v2.0. Practices: (1) a top-level `format_version` (and per-component versions); (2) a migration registry — `upgrade(v1→v2)` functions applied on load; (3) forward-compatible fields — never remove a field, only add optional ones; (4) never repurpose a field's meaning. ✅ This is standard software-engineering versioning applied to data files, and its absence is the #1 cause of "we can't resume, the checkpoint is from last month's code."

**Streaming formats.** For very large checkpoints, write files *in order* so that a partially-written checkpoint is still loadable (you can read the first N tensors even if the write was interrupted), and so that readers can begin loading while the writer is still writing (pipelined restore). Streaming formats trade a small header/offset table for partial-load capability. ⚠️ The tradeoff: a streaming checkpoint without a valid trailer is *not* crash-consistent — you must decide whether a partial checkpoint is acceptable (it usually isn't for the *primary* copy, but it's great for staging copies, §4.6).

### 2.3 Format Comparison

| Format | Framework | Pros | Cons | Security | When to use |
|---|---|---|---|---|---|
| `torch.save` (.pt/.pth) | PyTorch | Trivial, arbitrary objects, universal | Pickle RCE risk, slow, single-file, no sharding | ❌ Unsafe for untrusted files | Quick experiments, small models, trusted local only |
| safetensors | Any (HF standard) | Safe, fast, zero-copy, parallel-loadable, hub-native | Tensors only (no Python objects), needs sidecar for optimizer/RNG | ✅ Safe (no code exec) | Model weights everywhere; the default for release checkpoints |
| transformers dir (`save_pretrained`) | PyTorch/JAX/TF | Self-describing (config + weights + metadata), hub-ready | Trainer-version-specific state files | Depends: safetensors ✅ / .bin ❌ | HF ecosystem fine-tuning, hub releases |
| Orbax | JAX | Async, multihost, manager + retention built in | JAX-only, PyTree-oriented | ✅ (custom serializers; no pickle by default) | JAX training (TPU/GPU) |
| TF v2 checkpoint (.ckpt trio) | TensorFlow | Index + sharded data, atomic pointer | TF-only, not interoperable | ⚠️ no code exec in data files, but TF-specific | TF training, TF Serving |
| Custom schema | Any | Exactly your needs; versioning, manifest, partial-load | You own correctness, migration, tooling | You decide (safetensors payloads → ✅) | Distributed LLM training, regulated environments (§8) |

**The 2026 default answer:** weights in safetensors, optimizer/RNG/scheduler state in a versioned custom sidecar, managed by a checkpoint manager — regardless of framework. The rest of this guide builds toward that answer.

---
## 3. Checkpointing Strategies

### 3.1 Periodic Checkpoints (Every-N-Steps)

The baseline strategy: every N steps (or every T minutes), synchronously save the full training state. Simple, predictable, and the default of every framework (`save_steps=500` in HF Trainer, `every_n_train_steps` in Lightning, `--save-interval` in Megatron). ✅ The two tuning knobs:

- **Every-N-steps vs every-N-minutes.** Steps are the natural unit (training progress), but wall-clock is what actually bounds data loss — on a slow node, 1000 steps might take 3 hours. Robust systems save whichever fires first, and always on termination (§3.3). ⚠️
- **The interval.** Common practice: every 1,000–5,000 steps or every 1–2 hours for LLM training. ⚠️ **Approximate** — the "right" interval balances the checkpoint tax (§5.4) against the cost of lost progress (§1.1). The math: if a checkpoint costs C seconds of training time and you checkpoint every T seconds, the tax is C/T. If losing L seconds of training costs the same per-second as checkpointing, the optimum is roughly T ≈ √(C·run_length). For a 1 TB save at 10 GB/s (C ≈ 100 s) on a 30-day run, that suggests hourly saves — which is why "every 1–2 hours" is what practice converged on.

### 3.2 Adaptive Frequency

Early training is where the model changes fastest — loss improves rapidly, and a checkpoint from step 100 is *valuable* (it might be a great init for another experiment), while a checkpoint from step 900,000 is nearly identical to step 899,000. Adaptive policies: frequent early (every 100–500 steps for the first few thousand steps), sparse later (every 2,000–10,000 steps); or metric-driven (save when the validation metric improves — the "keep-best" pattern of §4.2). ⚠️ This is standard practice in fine-tuning workflows (early checkpoints double as "warm-start" candidates, §6.3) but has no single canonical formulation — tune to your loss curve and storage budget.

### 3.3 Checkpoint-on-Failure (Crash-Consistent / Save-on-Signal)

A checkpoint taken at a *known-good* moment (end of a step, optimizer state consistent) is **crash-consistent**: the saved state exactly matches a state the training actually passed through. The strongest reliability play is to also save when you're told to die:

- **SIGTERM handler.** On POSIX systems, schedulers and clouds send SIGTERM before killing a process (SLURM's `--signal`, Kubernetes preStop, AWS EC2 Spot termination notice ~2 min before reclaim — sent as a Rebalance Recommendation / instance-termination event; GCP preemption gives ~30 s warning). ✅ A SIGTERM handler that (a) stops training at the next step boundary, (b) flushes an async save, or (c) performs a fast synchronous save of the *current* state turns "preempted, lose an hour" into "preempted, lose seconds." PyTorch's `torch.distributed.elastic` and most frameworks register exactly this. ✅ **Verified pattern.**
- **Crash-consistency caveat.** Saving *mid-step* (weights updated, optimizer not yet) produces a corrupt state. Rule: only snapshot at a step boundary, or save a consistent pair (weights + optimizer) captured atomically from the same instant. Distributed runs add a second requirement: all ranks must save the *same logical step* — hence barriers before save (see §5.2).

### 3.4 Async Checkpointing

The core optimization: **overlap the save with training.** A synchronous 1 TB save at 10 GB/s stalls training for ~100 seconds. Async checkpointing hides that:

1. At step N, the training process *copies* the state to a **staging buffer** (GPU memory → pinned host memory → NVMe/local disk) — a fast memcpy that takes seconds, not minutes.
2. Training resumes immediately on step N+1.
3. A background thread/process streams the staged copy to the durable store (NFS/object storage) while training continues.

✅ **Verified** — this is the pattern implemented by DeepSpeed (`async_save`), Orbax (async checkpointing), NVIDIA NeMo (async save), and PyTorch Lightning (`enable_checkpointing` + async). Two correctness subtleties:

- The staged copy must be a *snapshot* (copy-on-write or explicit copy), not a reference — training mutates the tensors on the next step.
- The *logical* checkpoint is complete only when the background upload finishes; a crash between staging and upload means the last durable checkpoint is the previous one. The registry (§4.4) must track "staged" vs "durable" states.

At scale, async checkpointing turns a multi-minute stall into a ~1–2% throughput hit (§5.4) — it is the single highest-ROI strategy in this guide, and the reason "save latency" is rarely a bottleneck on well-run clusters.

### 3.5 Checkpoint Sharding

For a model that doesn't fit on one rank, the full state is *already* partitioned across ranks (ZeRO-3, FSDP, tensor/pipeline parallelism — see [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md)). Two ways to save it:

- **Naive (gather)**: each rank sends its shard to rank 0, which assembles the full state dict and saves it. Simple, but rank 0 becomes a bandwidth and memory bottleneck, and the save time grows linearly with model size — a 1 TB checkpoint gathered through one NIC is painfully slow.
- **Sharded (partitioned)**: **each rank saves its own shard directly** — rank i writes `rank-{i}-of-{N}.safetensors` (plus a metadata file mapping logical tensor names → shard files). ✅ This is exactly what DeepSpeed ZeRO checkpoints do (per-rank optimizer/parameter shards), what PyTorch DCP does, and what Megatron-LM's `dist_checkpointing` does. Save time becomes roughly constant in model size (each rank writes its local share in parallel), bounded by the slowest rank's local write + the filesystem's aggregate bandwidth.

ZeRO-3 in particular makes checkpointing *cheaper than the full model*: since parameters and optimizer states are partitioned across ranks, each rank persists only its 1/N slice — total bytes written = full state, but no rank ever materializes the whole model, and no gather happens at all. ✅ **Verified.**

### 3.6 Distributed Checkpointing (PyTorch DCP)

**PyTorch Distributed Checkpoint (DCP)** — `torch.distributed.checkpoint` — is the framework's answer to distributed save/load. ✅ **Verified** (mature in PyTorch 2.x; cited as "mature support" by PyTorch 2.9-era material). Key properties:

- **Distributed save/load**: `save()` and `load()` are called by *all* ranks collectively; each rank writes/reads its local shards in parallel — no gather to rank 0.
- **Sharded state dicts**: `torch.distributed.checkpoint.state_dict.get_model_state_dict()` / `get_optimizer_state_dict()` produce state dicts where each tensor is a `ShardedTensor`/`DTensor` describing its global layout — so DCP can **reshard**: a checkpoint saved with 64 ranks can be loaded with 32, because the metadata encodes the logical global tensor, not the physical rank assignment. ✅ This is the "checkpoint is rank-agnostic" property that makes DCP the foundation for elastic resume (§5.6).
- **Pluggable storage**: the `StorageWriter`/`StorageReader` abstraction targets local FS, NFS, or object stores (S3/GCS) — the same interface pattern as Orbax.

For new PyTorch distributed code, DCP (or its higher-level cousin in FSDP2 / torchtitan) is the recommended path over `torch.save(model.state_dict())` for anything beyond single-GPU scale — the official guidance in PyTorch's own tutorials. ✅

### 3.7 Frequency Tradeoffs and Strategy Selection

| Strategy | Save cost | Data-loss window | Complexity | Best for |
|---|---|---|---|---|
| Periodic sync (every-N) | High (stall per save) | Interval × throughput | None | Small models, single GPU, simple stacks |
| Periodic + async | Low (~1–2%) | Interval × throughput | Low | Most distributed training |
| Adaptive frequency | Medium | Bounded by interval | Medium | Fine-tuning, early-stage experimentation |
| Checkpoint-on-failure (SIGTERM) | Near-zero | Seconds | Low | Spot/preemptible instances, SLURM |
| Sharded (per-rank) | Low, scales flat | Interval × throughput | Medium | ZeRO/FSDP/TP/PP runs, >10B params |
| Distributed (DCP/Orbax/Megatron) | Low, scales flat | Interval × throughput | Medium–High | Multi-node LLM training, elastic clusters |

**Selection rule of thumb.** Small models (<1B, single node): full synchronous checkpoints, pickle or safetensors, keep it simple. Mid-size (1–10B, one node or two): async + keep-last-N. Large (10B+, multi-node, ZeRO/FSDP): sharded + async + DCP/Orbax-style manager + object-store staging + retention + restore testing. The design in §8 implements exactly the "large" column.

---

## 4. Checkpoint Orchestration

A checkpoint system is more than a save call — it is a lifecycle: **save → retain → prune → restore**, plus the metadata and storage plumbing underneath. This is the layer where "custom checkpoint systems" earn their name.

### 4.1 The Checkpoint Manager

The manager is the component that owns the lifecycle: it decides *when* to save (callback on step boundaries, on signal, on metric improvement), *where* to save (store abstraction, §4.3), *what* to keep (retention, §4.2), and *how* to restore (given a step or a tag, find and load the right checkpoint). Every serious framework has one — Lightning's `ModelCheckpoint`, Orbax's `CheckpointManager`, HF's `TrainerCallback`-driven save logic, Megatron's `CheckpointManager` — and the build-your-own version (§8.3) is a clean-room implementation of the same idea. The manager is also the natural home for: naming conventions (`step-12345/`), locking (one writer per checkpoint directory), and the write path ordering (metadata last — see §4.5).

### 4.2 Retention Policies

Storage is finite and checkpoints are big; someone must delete old ones. The standard policies:

- **Keep-last-N (rotation).** Keep the newest N checkpoints, delete the rest. Bounded storage, simple, and sufficient when the only goal is resume-from-latest. This is HF's `save_total_limit`, Lightning's `save_top_k`-with-`every_n` combo, and the classic "checkpoint rotation" pattern.
- **Keep-best.** Keep the checkpoint with the best validation metric (and the best few: `save_top_k=3`). ✅ Lightning's `ModelCheckpoint(monitor="val_loss", mode="min", save_top_k=3)` — monitors any logged metric, saves top-k by it, and the top-1 is `best.ckpt`. ✅ **Verified.** HF Trainer's `load_best_model_at_end` + `metric_for_best_model` implements the same idea.
- **Time-based.** Keep at least one checkpoint per hour/day/week regardless of count — protects against "the last 5 checkpoints are all from a broken hyperparameter change."
- **Hybrid (production-grade).** Keep-last-N *plus* keep-best *plus* keep-one-per-X-hours, e.g., "last 3 + best-by-metric + hourly for the last day, daily for the last week." The registry (§4.4) makes hybrid policies trivial to express: retention is just a query + delete against the metadata table.

Retention runs as a **prune** step after each successful save (and on startup, to clean up after crashes). Pruning must respect the same atomicity rules as saving: delete the manifest-listed files of the victim checkpoint only after confirming the checkpoint is complete and unregistered-as-current.

### 4.3 Checkpoint Stores

The storage abstraction — where checkpoints physically live. The spectrum (detailed storage engineering in [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md)):

- **Local disk (NVMe).** Fastest, but node-local: survives process death, not node death. Used as a *staging* tier (write fast locally, upload to durable storage asynchronously — §4.6) and for single-node training.
- **NFS.** The classic shared filesystem for training clusters: every rank writes to the same mounted path, POSIX semantics, no upload step. Single-writer-per-file + rename atomicity work naturally (§4.5). The catch: NFS is a latency-sensitive shared service — a 1 TB save at 10 GB/s aggregate needs a beefy NFS/Lustre backend, and NFS outages are a notorious training-killer. ⚠️
- **Parallel filesystems (Lustre, BeeGFS, CephFS, Weka, FSx for Lustre).** HPC-grade shared storage engineered for exactly this bandwidth pattern: many clients, huge files, high aggregate throughput. Lustre/BeeGFS/FSx can sustain 100s of GB/s aggregate — the difference between a 100-second and a 5-second 1 TB save. ✅ See [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) for the full comparison; this guide assumes the reader can choose there and focuses on the checkpoint logic that runs on top.
- **Object storage (S3, GCS, Azure Blob).** The durable, cheap, geo-redundant tier. Not POSIX (no rename/atomicity — S3 is read-after-write consistent per object, but no cross-object transactions), latency is high (100ms+ per request), so direct synchronous saves from ranks are painful — hence the standard architecture: fast local/shared FS for live saves, background upload to object storage for durability and retention. ✅ Object storage is also the natural *distribution* tier: HF Hub, W&B artifacts, and release checkpoints (§6.4) all live on object storage.

### 4.4 The Checkpoint Registry

A checkpoint is only useful if you can *find* it. The registry is the metadata catalog — a small database (SQLite for a single cluster, Postgres for a fleet) with one row per checkpoint:

| Column | Example | Purpose |
|---|---|---|
| `id` | `ckpt-00012345` | Unique name / directory name |
| `run_id` | `ft-70b-lora-v3` | Which training run it belongs to |
| `step` | 12345 | Training progress (sort key) |
| `epoch` | 2.4 | Epoch position |
| `metric` | `val_loss=1.87` | Best-metric value (for keep-best) |
| `path` | `s3://ml-checkpoints/ft-70b-lora-v3/step-12345/` | Where it lives |
| `size_bytes` | 290,000,000,000 | For cost accounting and prune decisions |
| `checksum` | `sha256:ab12…` | Integrity verification on restore |
| `state` | `staged / durable / verified / failed` | Lifecycle state |
| `created_at` | 2026-08-13T03:22:11Z | Time-based retention |

The registry enables: retention as SQL, restore-by-tag (`restore run=ft-70b-lora-v3 best`), cost reports (§9.3), and audit trails (who/what/when — relevant under [data_governance_guide.md](data_governance_guide.md) when checkpoints contain regulated data). In the build-your-own section (§8.3) the schema above is the design.

### 4.5 Atomicity and Consistency

**Single-file atomicity: temp-then-rename.** Writing `ckpt.pt.tmp` and `os.rename()`-ing it to `ckpt.pt` gives atomic visibility: any reader sees either the old file or the new one, never a partial write. ✅ POSIX rename semantics; works on NFS (with care) and local FS. This is why the *last* thing a save does is write a small pointer/`metadata.json` file and rename it — the rename *commits* the checkpoint.

**Multi-file consistency: the manifest.** A checkpoint is many files (weight shards, optimizer shards, sidecars). Rename-atomicity per file does not make the *set* atomic: a crash mid-save leaves half-new, half-old shards. The fix is a **manifest** — a single file listing every member file with its checksum — written *last* and renamed into place. A checkpoint directory without a valid manifest is incomplete by definition; restore logic (and prune logic!) checks the manifest first. ✅ This ordering — "write data files, fsync, write manifest, rename manifest" — is the standard commit protocol, used by TF's checkpoint pointer, DCP's metadata file, and every serious custom system. (⚠️ Full multi-file transactionality — atomic *sets* across files — is not available on NFS/object stores; the manifest convention is the industry-standard substitute.)

### 4.6 The Upload Pipeline (Local → Object Store)

The canonical durable path for cloud training: ranks save shards to local NVMe / shared FS (fast, async, §3.4), and a **background uploader** streams completed checkpoints to S3/GCS:

- **Multipart upload** for large objects (S3 multipart: 5 MB+ parts, parallel, resumable). ✅ Standard S3 API practice.
- Upload is per-checkpoint and idempotent; the registry flips `staged → durable` only after all parts complete and the object-store-side manifest is in place.
- **Ordering**: never delete the local copy until the durable copy is verified (checksum match). Local disk becomes a small, fast buffer (a few checkpoints) and object storage the long-term home — this two-tier pattern is what makes "checkpoint every hour" affordable (hourly uploads at TB-scale are ~cents-to-dollars of storage, §9.3).

### 4.7 The Restore Path

Restore is the half of checkpointing that gets tested least and matters most. The full path: (1) **resolve** — registry query → checkpoint location (latest step, or best metric, or explicit ID); (2) **download/prepare** — object store → local, in parallel, with checksum verification; (3) **load** — read manifest, reconstruct state dicts, load weights + optimizer + RNG + dataloader + scheduler; (4) **verify** — confirm the loaded state is sane (tensor shapes/dtypes match the current model config, optimizer keys match, step number matches metadata) before stepping; (5) **resume**. Every step is a failure point: missing shards, checksum mismatches, config drift, version skew. The discipline that makes restore reliable is *testing it* — §5.8's restore-test, automated and periodic, because the worst checkpoint system failure mode is discovering at 3 AM after a node dies that the last three checkpoints don't load.

---

## 5. Checkpointing at Scale

### 5.1 Multi-Node Training: The Overhead Problem

At 1000 GPUs (say 125 nodes × 8), a checkpoint must be a *coordinated, collective* operation: all ranks must save the same logical step, or the checkpoint is internally inconsistent (rank 3 at step 5000, rank 7 at step 4999 — resume is corrupt). Hence the classic pattern: barrier → snapshot state → (async) write → barrier → resume. ✅ Every rank participates; the save completes only when all ranks have written their shards. The orchestration cost — two all-rank barriers per checkpoint — is small; what dominates is **bandwidth** (next).

### 5.2 Checkpoint Bandwidth: The Arithmetic

The fundamental constraint is storage bandwidth, not CPU or GPU. The math:

```
save_time ≈ checkpoint_size / effective_write_bandwidth
```

Examples (⚠️ approximate; real numbers depend on filesystem, NICs, and sharding):

| Checkpoint size | 10 GB/s (NFS-class) | 100 GB/s (Lustre/BeeGFS-class) |
|---|---|---|
| 7B fine-tune (~30 GB) | 3 s | 0.3 s |
| 70B full (~300 GB–1 TB) | 30–100 s | 3–10 s |
| Frontier-class (~10 TB) | 17 min | 100 s |

Two consequences: (1) a *synchronous* save on NFS-class storage of a frontier checkpoint is a multi-minute training stall — this is why async (§3.4) and fast parallel filesystems (§4.3) are inseparable from LLM training; (2) sharding makes the per-rank write small (1/N), so the *node-local* write is fast and the bottleneck moves to the shared store's aggregate bandwidth — exactly what Lustre/BeeGFS/FSx are built for. ⚠️ Also note: GPU memory → CPU staging copies run over PCIe/NVLink at tens of GB/s per rank — for very large models, the copy itself can take seconds, another argument for overlapping (pipeline the copy with the next steps).

### 5.3 Async at Scale: NCCL and All-Gather

A subtlety at scale: to save a *full* (unsharded) checkpoint — e.g., for release or for a different parallelism config — ranks must first **all-gather** the state over NCCL, and only then can rank 0 (or all ranks, redundantly) write the full model. ✅ NCCL all-gather of a 1 TB model at ~100–400 GB/s NVLink/NVSwitch speeds takes seconds, but it (a) needs the full model in memory (a real constraint at 70B+ on 8×H100 — it doesn't fit, so full checkpoints are written from sharded state or via CPU offload), and (b) stalls training unless overlapped. The standard resolution: **keep the sharded checkpoint as the live one** (it is what resume needs), and produce the full/resharded copy only at milestones (or on demand) in the background — the "conversion" step that DeepSpeed's Universal Checkpointing and Megatron's resharding automate. ✅ DeepSpeed Universal Checkpointing (2023) saves the standard ZeRO checkpoint and converts it to any target parallelism (dense, AutoTP, AutoEP) offline. ✅ **Verified.**

### 5.4 The Checkpoint Tax

The "checkpoint tax" is the fraction of total training time spent saving. A naive synchronous save every hour with a 100-second save time is ~2.7% overhead; every 500 steps with a 2-minute save on a 3-second-per-step run is ~7%. ⚠️ **Approximate** — reported ranges in practice are ~2–10% for synchronous naive saves at scale, dropping to ~0–2% with async + sharded saves (the copy is overlapped; only the staging memcpy hits the critical path). The tax is the price of insurance; the goal of the engineering in §3–§4 is to push it toward zero without weakening durability. (Research on *zero*-overhead checkpointing exists — e.g., Checkmate, which hides saves in communication gaps — but production practice today is async + sharded.)

### 5.5 Spot Instances and Preemption

Spot/preemptible instances (抢占式实例) are 60–90% cheaper and are the economic backbone of cost-conscious training — but they are reclaimed, often with only ~2 minutes of notice (AWS EC2 Spot: instance-rebalance recommendation; GCP: preemption notice ~30 s; ✅ both documented). Checkpoint-friendly training on spot is a three-part recipe:

1. **Frequent, cheap checkpoints** (async + sharded, hourly or better).
2. **Save-on-signal** (§3.3) so a termination notice triggers a final checkpoint — the SIGTERM handler is the difference between "lost since last hour" and "lost since last second."
3. **Auto-resume** (§5.6) so the job comes back without a human.

With all three, spot instances become a *reliability strategy* rather than a cost gamble: preemption degrades to "a pause with a fresh checkpoint." This is a first-class pattern in cloud LLM training and the reason "checkpoint-friendly training" is a listed feature of training platforms.

### 5.6 Auto-Resume: SLURM and Kubernetes

**Auto-resume** is the orchestration glue: when a job dies (failure or preemption), the scheduler restarts it and the training entry point loads the latest checkpoint automatically. ✅ Two standard implementations:

- **SLURM**: the job's entry script wraps training in a retry loop — on non-zero exit, `scontrol requeue` (or a job-array retry) relaunches; the training code detects the latest checkpoint (registry query) and resumes. SLURM's `--signal=B:TERM@300` gives jobs a termination warning window for the graceful-save handler.
- **Kubernetes**: a StatefulSet/Job with a restart policy, plus a training entry point that loads the latest durable checkpoint on startup. TorchElastic's rendezvous (`--rdzv_backend=c10d`, `--rdzv_endpoint=...`) handles worker discovery across restarts. ✅

The pattern that makes auto-resume *safe* is **idempotent startup**: on every start, resolve latest → load → resume. The checkpoint manager (§4.1) and registry (§4.4) make this a two-line entry point (`restore(latest(run_id))`).

### 5.7 Elastic Training

Elastic training tolerates *membership changes*: nodes can join and leave mid-run, and training continues with a resized world. `torchrun` elastic (`--nnodes=1:4`, `--max_restarts=3`, `--rdzv_backend=c10d`) is the standard launcher; on a failure it terminates all processes and restarts them, each loading the last saved snapshot. ✅ **Verified** (PyTorch docs: "If a failure occurs, torchrun will terminate all the processes and restart them. Each process entry point first loads and initializes the last saved snapshot"). Note what this implies: **torchrun's fault tolerance is checkpoint-driven** — the restart is fast *because* checkpoints exist and are rank-agnostic (DCP sharded checkpoints, §3.6, load cleanly at a different world size). Elastic training without reshardable checkpoints is just "restart with the same world size." True elastic *resizing* (world size changes mid-run) additionally needs checkpoint resharding — which DCP/Orbax/Megatron provide and which the naive gather-to-rank-0 format cannot.

### 5.8 Checkpoint Verification: The Restore Test

The reliability law of checkpointing: **a checkpoint that has never been loaded is a checkpoint that might not load.** ⚠️ The industry-standard mitigation is the **restore test** — periodically (daily for long runs, or on every Nth checkpoint) take the latest checkpoint, load it in a scratch environment, run a few steps, and verify (a) it loads without error, (b) the loss continues from approximately the saved value, and (c) the checksums match. This is backup validation applied to training — the same discipline as database restore drills. ✅ Organizations running multi-week LLM training treat restore-testing as non-negotiable (the worked example schedules it weekly, §9.2); the cost is one spare node for a few minutes. Related practice: **verify-on-save** — checksum + load-check the manifest before the registry flips the checkpoint to `durable`, catching bad writes immediately rather than at the next restore.

---

## 6. LLM-Era Checkpointing

### 6.1 The Sizes

LLM checkpoints are measured in hundreds of GB to TBs. The arithmetic (✅ verified sizes; ⚠️ totals depend on optimizer precision and master-weight persistence):

| Model | Weights (BF16/FP16) | Adam optimizer state (FP32 moments) | Full mixed-precision checkpoint* |
|---|---|---|---|
| 7B | ~14 GB | ~56 GB | ~70–100 GB |
| 13B | ~26 GB | ~104 GB | ~130–190 GB |
| 70B | ~140 GB | ~560 GB | ~700 GB–1 TB |
| 175B (GPT-3-class) | ~350 GB | ~1.4 TB | ~1.75–2.5 TB |

\* includes FP32 master weights when persisted; "~280 GB optimizer state" figures in the wild assume BF16/FP16 moments (halved). Every number above is the *minimum* — activations for memory, RNG/dataloader/scheduler for the checkpoint, are negligible; but tensor-parallel replicas and pipeline stages can *duplicate* state (TP replicates parameters across the TP group; PP replicates optimizer state across data-parallel replicas of each stage), inflating checkpoint size beyond the single-replica math. ⚠️

These sizes drive everything: a 70B checkpoint cannot be `torch.save`'d to one file by one rank (rank 0 would need ~1 TB RAM); it must be sharded (§3.5), saved asynchronously (§3.4), and stored on a tier that can hold TB-scale objects (§4.3). "Checkpointing at LLM scale" is the reason every strategy in §3–§5 exists.

### 6.2 LLM Training Runs

A frontier LLM pretraining run lasts months and checkpoints are existential: the run *will* be interrupted (hardware, preemption, cluster maintenance, bad data), and the checkpoint is the only thing that survives. Practices at that scale: hourly saves of the sharded state (async), a "latest" pointer that always points at the newest *verified* checkpoint, keep-last-N plus milestone saves (a checkpoint every ~10k steps that is never pruned — these are the *artifacts* of the run), and restore-testing on every significant save. The frontier runs also produce **checkpoint families**: the same weights at multiple precisions (BF16 training checkpoint, FP32 master checkpoint, quantized release checkpoint — §8.4), which is where compression and conversion tooling (§5.3) earn their keep.

### 6.3 Fine-Tuning Checkpoints (LoRA/QLoRA)

Fine-tuning changes the checkpoint economics completely (cross-ref [ai_llm/fine_tuning_frameworks_comparison_guide.md](ai_llm/fine_tuning_frameworks_comparison_guide.md)): with **LoRA/QLoRA**, the trainable parameters are small adapters (typically 0.1–1% of the model), so the *trainable* checkpoint is megabytes, not GB — you save the adapter weights plus the frozen base-model reference (by ID/URL, not by copying 140 GB!). ✅ This makes fine-tuning checkpoints cheap to store, version, and ship (a LoRA checkpoint is a ~10–100 MB artifact). The full fine-tune (all weights trainable) is the opposite extreme: it re-persists the full model every save, and the checkpoint is the full state (§6.1) — which is exactly why parameter-efficient fine-tuning is the default for 70B-class work, and why the worked example (§9) fine-tunes a 70B with *full* weights but saves *sharded*. Note the terminology trap: a "LoRA checkpoint" contains adapters; a "full checkpoint" contains the entire model; a "merged checkpoint" is base + adapters folded into one set of weights (needed for deployment, §6.5).

### 6.4 Released Model Checkpoints

"Checkpoint" also means the *distributed artifact*: open models (Llama, Qwen, Mistral, DeepSeek, etc.) are released as checkpoints — weight files on the HF Hub, downloaded millions of times. ✅ Release checkpoints are: safetensors format (the hub standard — the `model.safetensors` vs `pytorch_model.bin` split is visible on every open-model repo), sharded into ~10 GB index files for resumable downloads, accompanied by config + tokenizer + license, and versioned by tag. The engineering of release checkpoints is a subset of this guide's concerns: distribution (CDN + object storage), integrity (checksums), format safety (safetensors over pickle — a supply-chain decision), and **license/governance** — model weights are IP: weights trained on proprietary data, licenses (Llama community license, Qwen, etc.), and export-control considerations make checkpoint *governance* a real compliance topic, covered in [data_governance_guide.md](data_governance_guide.md). ✅ For an enterprise, "downloading a checkpoint" is a procurement-and-risk action, not a curl.

### 6.5 Inference Checkpoints

The same checkpoint file that training writes is what inference serves — but the *formats differ*: training checkpoints carry optimizer state and sharded layouts; inference wants the merged weights in a serving-friendly layout (tensor-parallel shards for the inference engine, quantized — see [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) and [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md)). The pipeline: training checkpoint → **conversion/export** (drop optimizer state, merge LoRA adapters if any, re-shard for the serving TP size, quantize to FP8/INT4 for the inference engine) → model repository → serving. ✅ This is why "checkpoint" tooling always includes an export step, and why model registries (MLflow, W&B, HF Hub) treat the *deployable artifact* as distinct from the *training checkpoint* (§7.7).

---
## 7. Tools and Frameworks

### 7.1 PyTorch: `torch.save`/`load` and DCP

- **`torch.save`/`torch.load`**: the single-device baseline — pickle-based, trivial, unsafe for untrusted files (§2.1).
- **`torch.distributed.checkpoint` (DCP)**: the distributed save/load API — collective `save()`/`load()` with sharded state dicts, resharding, pluggable storage (§3.6). ✅ Mature in PyTorch 2.x. Also worth naming: `torch.distributed.checkpoint.state_dict` helpers (get sharded model/optimizer state dicts), and the FSDP2-era higher-level integration in torchtitan. For anything multi-GPU, DCP is the recommended path over gather-and-save. ✅
- **`torch.distributed.elastic` / `torchrun`**: fault-tolerant launching — restart-on-failure with checkpoint-driven resume (§5.7). ✅

### 7.2 PyTorch Lightning

`ModelCheckpoint` callback: ✅ **Verified** — `ModelCheckpoint(dirpath=..., monitor="val_loss", mode="min", save_top_k=3, save_last=True, filename="step-{step}")`. Saves `last.ckpt` (latest) and `best.ckpt` (top-1 by monitor), plus top-k named files; `every_n_train_steps`/`every_n_epochs` control cadence; `save_on_train_epoch_end`; auto-prunes to `save_top_k`. Lightning checkpoints are zip archives (pickle-based) containing model state, optimizer state, LR schedulers, and trainer state — the full §1.2 anatomy in one file. ✅ `Trainer(resume_from_checkpoint=...)` restores everything. Lightning is the right tool when you want framework-managed checkpointing on 1–8 GPUs without building your own; at LLM scale you typically move to DCP/DeepSpeed/Megatron underneath.

### 7.3 HuggingFace Transformers

`Trainer` + `TrainingArguments`: ✅ **Verified** — `save_strategy` (`"no" | "steps" | "epoch"`), `save_steps` (interval; historically defaulted to ~500), `save_total_limit` (keep-last-N, oldest deleted), `load_best_model_at_end` + `metric_for_best_model` + `greater_is_better` (keep-best), `resume_from_checkpoint`. ✅ `save_pretrained` writes the directory convention: `model.safetensors`/`pytorch_model.bin`, `config.json`, tokenizer files, plus `optimizer.pt`, `scheduler.pt`, `trainer_state.json`, `rng_state.pth` (the framework's own §1.2 anatomy — and a good reference implementation of a complete training-state checkpoint). ✅ Also note `push_to_hub` — checkpoints to the Hub for distribution (§6.4).

### 7.4 DeepSpeed

DeepSpeed's checkpointing is ZeRO-aware: with ZeRO-1/2/3, `save_checkpoint` writes **partitioned** state (each rank saves its optimizer shard / parameter shard), and `load_checkpoint` restores it; ✅ DeepSpeed can also extract FP32 weights from a ZeRO checkpoint (for release/export) and supports `async_save` (⚠️ the config options `"async_save": true` in the checkpoint section — the underlying mechanism is an offload of the save to a background thread; treat exact option names as version-dependent). ✅ **Universal Checkpointing** (2023) converts a ZeRO checkpoint to any target parallel configuration — dense, AutoTP, AutoEP — offline, which is the resharding story for Elastic/heterogeneous setups (§5.3). ✅ **Verified.** DeepSpeed is the de-facto checkpoint layer for many LLM training stacks precisely because it makes ZeRO-3 checkpointing correct-by-default.

### 7.5 Megatron-LM

Megatron's `dist_checkpointing` (Megatron Core) is a full distributed checkpointing system: sharded checkpoint data (torch_dist format, plus fsdp_dtensor and legacy torch formats), async saving, and — its signature feature — **checkpoints that can be loaded under a different parallel configuration** (tensor/pipeline/data parallelism changed between save and load). ✅ **Verified** (Megatron docs: "a checkpoint saved under one parallel configuration can be loaded under a different parallel configuration"). That property is what makes Megatron checkpoints the interchange format for TP/PP/DP-resized runs and the basis for its NeMo integration.

### 7.6 JAX / Orbax

Orbax (`orbax.checkpoint`): ✅ **Verified** — `CheckpointManager` with `save`/`restore`/`best`/`all_steps`, retention (`CheckpointManagerOptions`), **asynchronous checkpointing**, multihost/multislice distributed checkpointing, PyTree state, and pluggable storage (GCS, local, TFDS-style). For JAX/TPU training it is the standard, and its manager API is the cleanest public reference for the §8 design.

### 7.7 Experiment Tracking: W&B and MLflow

- **Weights & Biases Artifacts**: log checkpoints as artifacts (`run.log_artifact("ckpt/step-12345", type="model")`) — W&B stores them on object storage, dedupes, versions them, and links them to the run's metrics, so "the checkpoint that achieved val_loss 1.87" is queryable. ✅ W&B is widely used for exactly this checkpoint-as-artifact workflow.
- **MLflow**: `mlflow.log_artifact()` / `mlflow.pytorch.log_model()` / the **MLflow Model Registry** — versioned, stage-tracked (staging/production) model artifacts with lineage to the training run; Lightning's `MLFlowLogger` can auto-log `ModelCheckpoint` outputs. ✅ **Verified** (MLflow docs; Lightning MLflowLogger `log_model` option). MLflow's registry is the natural home for the *deployable* artifact (§6.5), distinct from raw training checkpoints.

The division of labor in mature platforms: raw training checkpoints in your own store (S3/FS) managed by your manager; *artifacts of record* (best models, release candidates) promoted into W&B/MLflow with lineage and governance.

### 7.8 Tools Comparison

| Tool | Framework | Key checkpoint features | Best for |
|---|---|---|---|
| `torch.save`/`load` | PyTorch | Simple, pickle, single-file | Single-GPU, prototyping |
| PyTorch DCP | PyTorch | Distributed save/load, reshardable, pluggable storage | Multi-GPU/multi-node PyTorch training |
| Lightning ModelCheckpoint | PyTorch | monitor/top-k/last-best, auto-prune, full trainer state | 1–8 GPU research/fine-tuning |
| HF Trainer | PyTorch (multiframework) | save_steps/epoch, save_total_limit, best-at-end, hub push | Transformers fine-tuning |
| DeepSpeed ZeRO ckpt | PyTorch | ZeRO-partitioned, async_save, universal conversion | ZeRO-3 LLM training |
| Megatron dist_checkpointing | PyTorch | Sharded, reshardable across TP/PP/DP, async | Frontier-scale TP/PP runs |
| Orbax | JAX | Manager, async, multihost, GCS-native | JAX/TPU training |
| W&B Artifacts | Any | Versioned artifact store, lineage | Experiment tracking + checkpoint catalog |
| MLflow (registry) | Any | Model registry, stages, lineage | Deployable model governance |

---

## 8. Build Your Own: A Custom Checkpoint System

When do you build your own? When you need: (a) a specific save/restore contract across frameworks, (b) regulated retention/audit (banking context!), (c) store portability (on-prem FS + cloud object store), (d) behavior no framework offers (custom compression, multi-tier policy, chaos-tested restore). The design below is the minimal production shape — every component exists in some form in DCP/Orbax/Megatron (§7), and rolling your own is mostly *integrating those patterns* behind one API.

### 8.1 Requirements

| Requirement | Typical target | Driven by |
|---|---|---|
| Save latency (training impact) | <2% throughput tax | Async + sharded saves (§3.4–3.5) |
| Restore reliability | Restore succeeds 100% of the time, verified | Manifest + checksums + restore-test (§4.5, §5.8) |
| Data-loss window | ≤ 1 checkpoint interval | Cadence + save-on-signal (§3.1, §3.3) |
| Storage cost | Bounded by retention policy | Rotation/keep-best/compression (§4.2, §8.4) |
| Security | Encrypted at rest, access-controlled, safe format | KMS + IAM + safetensors (§8.5) |
| Auditability | Every checkpoint traceable | Registry + metadata (§4.4) |

### 8.2 Architecture

```
                     ┌──────────────────────────────────────────────┐
                     │            CHECKPOINT COORDINATOR            │
                     │  (in-process lib + small control service)    │
                     │  step callback · signal handler · policies   │
                     └───────┬──────────────┬──────────────┬────────┘
                             │              │              │
                 ┌───────────▼───┐  ┌───────▼───────┐  ┌───▼────────────┐
                 │ CHECKPOINT    │  │  REGISTRY     │  │  VERIFIER      │
                 │ STORE         │  │  (metadata DB)│  │  (restore-test)│
                 │ local NVMe →  │  │  step, metric,│  │  load + N-step │
                 │ shared FS →   │  │  path, size,  │  │  + checksum    │
                 │ object store  │  │  checksum     │  │                │
                 └───────────────┘  └───────────────┘  └────────────────┘
```

- **Checkpoint coordinator** — the save/restore API the training code calls; owns the lifecycle, the staging buffer, the async writer thread, and the signal handler.
- **Checkpoint store** — a storage abstraction (local NVMe / shared FS / S3/GCS) with the two-tier pattern of §4.3–4.6; the coordinator writes to the fast tier, the uploader promotes to the durable tier.
- **Registry** — the metadata catalog (§4.4); the source of truth for restore resolution, retention, and audit.
- **Verifier** — the restore-test worker (§5.8): periodically pulls the latest durable checkpoint, loads it in a scratch pod, steps it, reports pass/fail.

### 8.3 Implementation Sketch (Python)

The heart of the system — a manager with save/restore, atomic commit via manifest, and registry integration (simplified but real):

```python
import json, os, shutil, hashlib, threading, time
from pathlib import Path

class CheckpointManager:
    """Async checkpoint manager: save -> local staging -> durable store -> registry."""

    def __init__(self, run_id, local_dir, durable_uri, registry, saver, loader,
                 keep_last=3, keep_best=1, metric_name="val_loss", mode="min"):
        self.run_id = run_id
        self.local_dir = Path(local_dir)          # fast NVMe staging tier
        self.durable_uri = durable_uri            # e.g. s3://ml-ckpt/<run_id>/
        self.registry = registry                  # metadata DB (schema in 8.6)
        self.saver = saver                        # callable(state, path) -> writes shards
        self.loader = loader                      # callable(path) -> state
        self.keep_last, self.keep_best = keep_last, keep_best
        self.metric_name, self.mode = metric_name, mode
        self._lock = threading.Lock()

    def save(self, state, step, metric=None):
        """Called at step boundaries (and from the SIGTERM handler)."""
        ckpt_dir = self.local_dir / f"step-{step:09d}"
        staging = ckpt_dir.with_name(ckpt_dir.name + ".tmp")
        shutil.rmtree(staging, ignore_errors=True); staging.mkdir(parents=True)

        # 1. write data files (weight/optimizer shards, sidecars) to staging
        self.saver(state, staging)

        # 2. write the manifest LAST, then rename -> atomic commit of the set
        manifest = {
            "format_version": 2, "run_id": self.run_id, "step": step,
            "metric": metric, "files": {
                p.name: hashlib.sha256(p.read_bytes()).hexdigest()
                for p in staging.iterdir() if p.is_file()
            },
            "created_at": time.time(),
        }
        (staging / "manifest.json").write_text(json.dumps(manifest, indent=2))
        os.rename(staging, ckpt_dir)              # atomic on local FS

        # 3. async upload to the durable tier; registry row flips staged -> durable
        threading.Thread(target=self._promote, args=(ckpt_dir, manifest), daemon=True).start()
        self.registry.upsert(run_id=self.run_id, step=step, metric=metric,
                             path=str(ckpt_dir), state="staged",
                             size_bytes=sum(p.stat().st_size for p in ckpt_dir.rglob("*")))
        self._prune()                             # retention policy (8.6)

    def _promote(self, ckpt_dir, manifest):
        """Background upload with multipart semantics; idempotent."""
        durable_path = f"{self.durable_uri}/{ckpt_dir.name}"
        # upload each file (S3 multipart / gsutil cp / rsync), verify checksums
        upload_all(ckpt_dir, durable_path, manifest["files"])
        self.registry.update_state(run_id=self.run_id, step=manifest["step"], state="durable")

    def restore(self, tag="latest"):
        """tag: 'latest' | 'best' | explicit step. Returns a loaded state."""
        row = self.registry.resolve(self.run_id, tag)     # SQL query (8.6)
        local = self.local_dir / row["path"].rsplit("/", 1)[-1]
        if not local.exists():                            # pull from durable tier
            download_all(row["path"], local)
        manifest = json.loads((local / "manifest.json").read_text())
        for name, sha in manifest["files"].items():       # integrity check
            assert hashlib.sha256((local / name).read_bytes()).hexdigest() == sha
        return self.loader(local), manifest                # -> training loop resumes

    def _prune(self):
        with self._lock:
            rows = self.registry.list(self.run_id)
            keep = set(self._select(rows, self.keep_last, None)) | \
                   set(self._select(rows, self.keep_best, self.metric_name))
            for row in rows:
                if row["step"] not in keep:
                    delete_all(row["path"]); self.registry.delete(row["id"])
```

Key decisions embedded here: **staging + rename commit** (atomicity, §4.5), **async promote with registry state machine** (`staged → durable`, §4.6), **checksum-verified restore** (§4.7), **retention as a policy function** (§4.2), and a single `restore(tag)` entry point for auto-resume (§5.6). The `saver`/`loader` callables let the same manager wrap any format (§2): safetensors for weights, JSON/pickle sidecars for RNG/dataloader/scheduler.

### 8.4 Checkpoint Compression

Checkpoints compress better than you'd think, because neural weights are *not* random bytes:

- **Generic codecs**: zstd/lz4 on shard files. ✅ zstd is the practical default (better ratio than lz4 at similar speed); LLM weights typically compress ~1.5–2× losslessly ⚠️ (entropy varies by layer/step). Cheap, safe, transparent.
- **Precision conversion (lossy)**: store weights/moments in BF16/FP16 instead of FP32 (halves optimizer state), or **FP8** (another 2×). ⚠️ **Lossy** — resume from a lossy-precision checkpoint is a *warm restart* (§11), not an exact resume; acceptable at milestones, dangerous as the live resume chain. The safe pattern: keep the exact-precision checkpoint as the live one; produce converted copies only for release/export (§6.5).
- **Quantization (lossy)**: INT8/4-bit weight checkpoints (GPTQ/AWQ/QLoRA-style, cross-ref [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md)). ⚠️ For *storage/distribution*, 4-bit release checkpoints are standard; for *resume*, quantized optimizer state is not viable for Adam (moments need full precision to stay stable). Rule: **compress the weights you ship, never the state you resume.**
- **Skip-zero / sparse tricks**: optimizer moments contain near-zero entries early in training; structured sparsity can shave bytes, but adds CPU cost and complexity — rarely worth it versus zstd. ❓

### 8.5 Checkpoint Security

- **Format safety**: safetensors (or your own header+bytes schema) for anything that crosses trust boundaries; never pickle-untrusted checkpoints (§2.1). ✅
- **Encryption at rest**: checkpoints on S3/GCS/NFS encrypted with **KMS-managed keys** (SSE-KMS for S3/GCS; LUKS/TLS for NFS). ✅ Standard cloud practice; for regulated data, key rotation and per-run keys are the norm. Also encrypt the registry DB.
- **Access control**: IAM roles / bucket policies / FS permissions scoped to training and serving identities; checkpoints are model IP — restrict read to the team, write to the training service account, and *delete* access to the retention job only. ✅
- **Checkpoints as sensitive data**: a fine-tuned checkpoint may encode proprietary or personal data (training-data leakage is a documented LLM property — see [data_governance_guide.md](data_governance_guide.md)); in banking, checkpoint *lifecycle* (retention periods, deletion, audit logs, DPIA for models trained on personal data) is a governance obligation, not an engineering nicety. This is one of the strongest arguments for the registry + retention design of §4.

### 8.6 Registry Schema and Testing

**Registry schema** (SQLite/Postgres) — the table from §4.4, concretized:

```sql
CREATE TABLE checkpoints (
  id          TEXT PRIMARY KEY,          -- 'ckpt-00012345'
  run_id      TEXT NOT NULL,             -- training run
  step        INTEGER NOT NULL,
  epoch       REAL,
  metric      REAL,                      -- best monitored metric value
  path        TEXT NOT NULL,             -- local or s3:// URI
  size_bytes  INTEGER,
  checksum    TEXT,                      -- sha256 of manifest
  state       TEXT DEFAULT 'staged',     -- staged|durable|verified|failed
  created_at  TIMESTAMP DEFAULT now()
);
CREATE INDEX idx_run_step ON checkpoints(run_id, step DESC);
```

**Testing the system** — the two tests that matter:

1. **Restore-test automation** (CI + cron): every N checkpoints (or nightly), restore the latest durable checkpoint into a scratch environment, run 5 training steps, assert loss ≈ saved loss ± ε and checksums match. This is the §5.8 discipline, automated.
2. **Chaos / kill-the-node test**: in staging, kill a training pod mid-run (SIGKILL — no graceful signal!), restart the job, assert it resumes from the *previous durable* checkpoint and training continues. Run variants: kill during save, kill during upload, kill during restore. ⚠️ The "kill during save" variant is where naive systems die — the manifest protocol (§4.5) is what survives it.

---

## 9. Worked Example: A Checkpoint System for a 70B LLM Fine-Tune

### 9.1 Scenario

Fine-tuning a 70B-parameter LLM (e.g., Llama-3-class) on an **8× H100 node** (single node, 8 GPUs × 80 GB HBM — memory sizing per [gpu_optimization_guide.md](gpu_optimization_guide.md)), 4 weeks of training, on cloud spot instances to control cost. Requirements: no more than 2% checkpoint tax, resume within 15 minutes of any failure, storage bill predictable, and the best checkpoint available at the end.

### 9.2 Design

| Concern | Choice | Why |
|---|---|---|
| Format | safetensors weights + JSON sidecars (RNG/dataloader/scheduler) + manifest | Safe, fast, versioned (§2, §4.5) |
| Sharding | ZeRO-3, per-rank shards (8 shards/checkpoint) via DeepSpeed save | No gather, each rank writes ~1/8 of ~300 GB (⚠️ BF16 moments assumed) |
| Async | DeepSpeed `async_save` + local NVMe staging, background upload to S3 | Save off the critical path (§3.4) |
| Durable store | S3 bucket `ml-ckpt/ft-70b-v3/`, SSE-KMS, lifecycle rule | Durable, cheap, encrypted (§4.3, §8.5) |
| Cadence | Every 2,000 steps (~1 h) + SIGTERM handler save | ≈1 h data-loss window; preemption-safe (§3.3) |
| Retention | Keep-last-3 + keep-best (val_loss) + weekly milestone kept forever | Bounded storage, best model preserved (§4.2) |
| Registry | Postgres table (§8.6) | Restore resolution, audit, cost reporting |
| Restore test | Weekly restore-test job on the latest durable checkpoint | §5.8 discipline |
| Auto-resume | SLURM retry loop (`scontrol requeue` on spot reclaim) + `restore("latest")` entry point | §5.6 |

### 9.3 Checkpoint Economics

Checkpoint size: ~300 GB per checkpoint (140 GB BF16 weights + ~140 GB optimizer state at BF16 moments + sidecars; ⚠️ with FP32 moments it would be ~700 GB–1 TB — the design deliberately uses BF16 moments, a memory-and-storage-saving choice that trades exact resume fidelity). Storage over the run:

| Item | Math | Cost (approx., 2026 on-demand pricing) |
|---|---|---|
| Live checkpoints (keep-last-3) | 3 × 300 GB = 0.9 TB on S3 | ~$20/mo (S3 standard ≈ $0.023/GB-mo) |
| Milestones (4 weeks → 4 weekly) | 4 × 300 GB = 1.2 TB | ~$28/mo, retained for the model's life |
| Best + converted release (FP8, §8.4) | ~150 GB | ~$4/mo |
| **Total storage** | **~2.25 TB** | **~$50/mo** — the price of the insurance |
| S3 PUT/upload bandwidth | ~24 GB/day uploads + multipart | cents/day |
| Restore-test compute | 1 h/week of a spare node | ~$15/wk on spot |
| **Checkpoint tax** | async save ≈ 1–2% of training | ≈ 2–3 GPU-days over 4 weeks ≈ **$1.5–2.5k** ⚠️ |

Compare with the alternative: *one* uncheckpointed day lost to a node failure costs ~8 GPUs × 24 h × $3 ≈ **$576** — and a spot preemption without save-on-signal loses up to an hour of 8 GPUs ≈ $24 *per event*, with preemptions happening multiple times a week. The whole checkpoint system pays for itself with the first avoided multi-hour loss. ✅ The "280 GB × 4 = ~1.1 TB" style estimate (weights + optimizer at BF16 ≈ 280 GB per checkpoint, times the kept set) lands in the same ballpark as the table above — the exact number depends on precision choices and retention.

### 9.4 Failure Scenario

**The event**: 11 days into the run, a GPU ECC error takes down the node. SLURM requeues the job; spot reclaim may also have fired, and the SIGTERM handler saved a final checkpoint at step 154,302 (~1 min of training lost, not an hour).

**The recovery timeline**:

| Step | Time |
|---|---|
| SLURM/K8s detects failure, requeues | ~1–2 min |
| Entry point calls `restore("latest")` → registry → S3 → download (300 GB @ ~5 GB/s object-store-class bandwidth) | ~2–5 min |
| Checksum verify + load shards + rebuild ZeRO state | ~2–3 min |
| Warmup steps (optional LR re-anchor) | ~1 min |
| **Training resumes** | **~10–12 min total** — within the 15-minute requirement |

**The lesson**: "Checkpoints are insurance — but only if you *test the restore.*" Every number in that table is a *designed, tested* number: the restore-test (§9.2) ran weekly, the manifest was verified, and the first time the run actually died, the 11-day-old run came back in 12 minutes because the restore path had been exercised before. The teams that skip restore-testing discover at 3 AM that checkpoint #47 doesn't load; the teams that test it treat failures as a 12-minute interruption.

---

## 10. The Future: 2026 and Beyond

**Checkpoint-less training (research, not fantasy).** The idea of training *without* checkpoints — recovering state from surviving replicas or gradient history instead of persisted snapshots — moved from "impossible?" to active research in 2025: **CheckFree** (Gensyn, arXiv 2506.15461, June 2025) recovers from failures without checkpointing or redundant computation, and PyTorch's "Fault Tolerant Llama" work (June 2025) trained Llama on Crusoe L40S clusters through **2,000 synthetic failures every 15 seconds with no checkpoints** using replication-based recovery. ✅ **Verified — both exist.** This is genuinely promising for decentralized/spot-heavy training, but as of 2026 it is research: replication costs memory/bandwidth, recovery is approximate (not exact-resume), and it does not produce the *artifacts* (release checkpoints, milestone states) that make checkpoints valuable beyond fault tolerance. Expect checkpoint-less recovery to complement, not replace, checkpoints — and to raise the bar for what "checkpoint" means (state *reconstruction* vs state *persistence*). ⚠️ Flag honestly: production-grade checkpoint-less training at frontier scale is not yet demonstrated.

**Memory-based checkpoints.** Fast NVMe staging is already standard (§4.6); the frontier is moving checkpoints *into* the memory hierarchy — NVMe-over-fabric, and CXL memory pooling (see [gpu_optimization_guide.md](gpu_optimization_guide.md) for the memory-hierarchy context). The pattern: save to pooled CXL/NVMe memory in microseconds, promote to disk/object storage asynchronously — effectively making "checkpoint" a memory copy with near-zero tax. ⚠️ CXL-based checkpointing is early-2026 experimental; the NVMe tier is production.

**AI-driven checkpoint management.** Retention and cadence are policy problems with measurable objectives (storage cost, recovery time, data-loss window). Expect (and build) managers that tune cadence from loss dynamics (§3.2), predict preemption from spot-price signals, and auto-promote checkpoints based on metric deltas — the registry (§4.4) is the substrate; the policy is a function you can already express.

**Checkpoint standards convergence.** safetensors has effectively won the *weight* format (every open model ships it, §6.4); the open frontier is a standard for the *training state* (optimizer + RNG + dataloader + scheduler) — DCP's sharded state-dict metadata, Megatron's torch_dist format, and Orbax's PyTree serialization are converging on the same ideas (manifest + sharded files + rank-agnostic metadata). Expect the "checkpoint directory" convention (manifest + shards + sidecars, §2.2) to become the interchange standard across frameworks.

**Trends summary**: checkpoint tax → near-zero (async + NVMe/CXL staging); checkpoint size → managed by precision/quantization for storage while exact-precision stays for resume; restore reliability → first-class (verified, chaos-tested); checkpoint-less recovery → a complement for spot/decentralized training; checkpoints → governed data (encrypted, retained, audited) in regulated industries. The direction of travel is unambiguous: **checkpointing is becoming a platform service with the same reliability bar as the training itself.**

---

## 11. Glossary

- **Checkpoint** — a persisted snapshot of training state enabling resume (§1.2); also, loosely, any released model-weight artifact.
- **State dict** — PyTorch's `dict` of name → tensor; `model.state_dict()` / `optimizer.state_dict()`; the serialization unit (§1.3).
- **Model weights / parameters** — the trainable tensors; the "model" part of a checkpoint.
- **Optimizer state** — optimizer bookkeeping (for Adam: first + second moments; the 2× model-size rule, §1.2).
- **Adam moments** — the mean-of-gradients (m) and mean-of-squared-gradients (v) accumulators maintained by Adam.
- **RNG state** — random-generator state (torch/cuda/numpy) for reproducible resume (§1.2).
- **Dataloader state** — sampler/epoch/batch-index/shard-offset position in the data stream (§1.2).
- **Scheduler state** — LR-schedule position (current LR, step, warmup phase) (§1.2).
- **torch.save / .pt / .pth** — PyTorch's pickle-based serialization (§2.1).
- **safetensors** — HF's safe (no-pickle), fast, zero-copy tensor format (§2.1).
- **Pickle** — Python's object serialization; unsafe for untrusted data (arbitrary code execution).
- **pytorch_model.bin / model.safetensors** — the two weight files HF `save_pretrained` writes (pickle vs safetensors) (§2.1).
- **Orbax** — JAX checkpointing library: manager, async, multihost (§7.6).
- **PyTorch Lightning / ModelCheckpoint** — training framework; its checkpoint callback (monitor/mode/save_top_k).
- **last.ckpt / best.ckpt** — Lightning's latest / best-by-monitor checkpoints (§7.2).
- **HuggingFace Trainer / save_strategy / save_total_limit** — transformers training loop and its checkpoint controls (§7.3).
- **DeepSpeed / ZeRO** — distributed optimizer; ZeRO-1/2/3 partition optimizer/grad/param state; checkpoints are per-rank shards (§7.4).
- **Megatron-LM** — NVIDIA's large-model training framework; `dist_checkpointing` with reshardable shards (§7.5).
- **Distributed checkpoint / DCP** — `torch.distributed.checkpoint`: collective sharded save/load with resharding (§3.6).
- **Async checkpoint** — save overlapped with training via staged copy + background write (§3.4).
- **Checkpoint sharding / shard** — splitting checkpoint files across ranks; one shard per rank's slice of state (§3.5).
- **Checkpoint manager** — the component owning save/retain/prune/restore lifecycle (§4.1).
- **Retention / rotation** — policies for which checkpoints to keep (keep-last-N, keep-best, time-based) (§4.2).
- **Checkpoint store** — the storage abstraction/tier (NVMe, NFS, Lustre/BeeGFS, S3/GCS) (§4.3).
- **Object storage / S3 / GCS** — durable cloud blob storage; the long-term checkpoint tier (§4.3).
- **NFS / Lustre** — network/parallel filesystems used as shared checkpoint targets (§4.3; see cephfs_alternatives_guide.md).
- **Atomic write / temp-then-rename** — write `.tmp`, rename into place; single-file atomic commit (§4.5).
- **Manifest** — file listing a checkpoint's member files + checksums; written last to commit the set (§4.5).
- **Registry / metadata** — the checkpoint catalog DB (step, metric, path, size, checksum, state) (§4.4, §8.6).
- **Restore / restore-test** — the load-and-resume path; periodically testing it (backup validation) (§4.7, §5.8).
- **Auto-resume** — scheduler-driven restart that loads the latest checkpoint automatically (§5.6).
- **Elastic training** — training tolerant of membership changes; torchrun `--nnodes=1:4`, `--max_restarts` (§5.7).
- **Preemption / spot instance** — cloud reclaim of cheaper instances; checkpoint-friendly training turns it into a pause (§5.5).
- **Checkpoint tax** — the % of training time spent saving (§5.4).
- **Compression / zstd / quantization** — shrinking checkpoints: lossless codecs vs lossy precision/quantization (§8.4).
- **Encryption / KMS** — at-rest checkpoint encryption with managed keys (§8.5).
- **Checksum** — integrity hash (sha256) per file/manifest (§4.5).
- **W&B Artifacts / MLflow** — experiment-tracking artifact/registry stores (§7.7).
- **LoRA / QLoRA** — parameter-efficient fine-tuning; tiny adapter checkpoints (§6.3).
- **Fine-tuning / transfer learning** — continued training from a pretrained checkpoint (the most common "warm start").
- **Warm restart** — resuming from a checkpoint with a fresh schedule/state (vs exact resume) (§6.3, §8.4).

---

## 12. References and Companion Guides

**Companion guides in this repository** (cross-referenced throughout):

- [gpu_optimization_guide.md](gpu_optimization_guide.md) — GPU memory hierarchy, NVMe/CXL staging context, throughput engineering
- [cephfs_alternatives_guide.md](cephfs_alternatives_guide.md) — NFS, Lustre, BeeGFS, CephFS, object storage: the checkpoint storage layer
- [data_governance_guide.md](data_governance_guide.md) — checkpoints as governed data: sensitivity, retention, audit
- [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md) — PyTorch/JAX/TF internals: state dicts, PyTrees, device placement
- [ai_llm/fine_tuning_frameworks_comparison_guide.md](ai_llm/fine_tuning_frameworks_comparison_guide.md) — LoRA/QLoRA fine-tuning and its checkpoint economics
- [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) — checkpoint → sharded inference serving
- [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md) — inference-side performance; quantized model artifacts
- [ai_llm/llm_model_pruning_guide.md](ai_llm/llm_model_pruning_guide.md) — quantization (GPTQ/AWQ) for compressed release checkpoints

**Primary sources consulted for verification (August 2026):**

- PyTorch docs — Distributed Checkpoint (DCP) recipes, `torch.distributed.elastic` / `torchrun` docs, fault-tolerant training tutorial, optimizer-memory tutorial (Adam O = 2P)
- safetensors repo + HuggingFace "The Risk of Pickle" blog
- google/orbax repo + Orbax docs (async checkpointing, multihost)
- DeepSpeed docs — Model Checkpointing, ZeRO, Universal Checkpointing tutorial
- NVIDIA Megatron-LM docs — `dist_checkpointing`, checkpoint formats/backends, NeMo Megatron Bridge
- PyTorch Lightning docs — `ModelCheckpoint` API
- HuggingFace transformers — `TrainingArguments` (save_strategy, save_steps, save_total_limit)
- MLflow docs + Lightning `MLFlowLogger` docs
- CheckFree (arXiv 2506.15461, June 2025) and PyTorch "Fault Tolerant Llama" blog (June 2025) — checkpoint-free training research
- Multiple LLM training guides — checkpoint cadence (every 1,000–5,000 steps / 1–2 hours), checkpoint tax ranges, 70B/175B checkpoint size arithmetic

> **Disclaimer.** All sizes, costs, and timings are order-of-magnitude estimates for planning, not quotes. Verified claims carry the ✅ marker inline; ⚠️ marks configuration-dependent numbers; ❓ marks vendor-reported figures. Always validate against your own hardware, storage, and pricing before committing budgets.
