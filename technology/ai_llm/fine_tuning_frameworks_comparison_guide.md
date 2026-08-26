# LLM Fine-Tuning Frameworks — Comprehensive Comparison Guide

A detailed technical comparison of the tools, platforms, and techniques for fine-tuning Large Language Models.

**Author:** Jack Liu Shurui  
**Series:** LLM/AI Technical Guides  
**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)

---

## 1. What Is Fine-Tuning?

Fine-tuning is the process of adapting a pre-trained Large Language Model to a specific task, domain, or behaviour by continuing training on task-specific data. A pre-trained LLM has learned general language patterns and broad knowledge from massive text corpora; fine-tuning specialises this foundation for a particular use case.

### Fine-Tuning vs RAG (Retrieval-Augmented Generation)

| Aspect | Fine-Tuning | RAG |
|--------|-------------|-----|
| **What changes** | Model weights updated | Model weights frozen |
| **Knowledge added** | Learned into parameters during training | Injected into context window at inference |
| **Persistence** | Permanent (in weights) | Per-query |
| **Latency** | Same as base model inference | Higher (retrieval step) |
| **Freshness** | Stale until re-trained | Live databases |
| **Offline** | Works fully offline | Requires retrieval backend |
| **Transparency** | Black box | Can cite sources |
| **Cost** | High upfront, low per-query | Low upfront, moderate per-query |

**They are complementary.** The most common production pattern is **fine-tune for behaviour + RAG for knowledge**: fine-tune the model to follow instructions and format outputs, then augment with retrieved context at inference for up-to-date information.

### What Fine-Tuning Is NOT
- **Not prompt engineering** — that crafts better inputs without changing the model.
- **Not RAG** — that adds context at inference; fine-tuning changes weights.
- **Not pre-training** — that learns language from scratch on vast unlabelled data.
- **Not continued pre-training** — domain-adaptive pre-training on unlabelled domain text before task-specific fine-tuning (a separate step).

---

## 2. Why Fine-Tuning Matters

### 2.1 Task-Specific Performance
Fine-tuned models outperform base models on domain tasks by **10–40%**. A fine-tuned 7B model can match or exceed a base 13B or 70B on the target task.

### 2.2 Domain Adaptation
General-purpose LLMs lack specialised knowledge in **legal** (case law, contract clauses), **medical** (clinical terminology, diagnostic criteria, ICD-10 coding), **financial** (earnings calls, 10-K filings, ratios), **technical** (proprietary APIs, internal tooling), and **scientific** (field-specific notation, protocols) domains.

### 2.3 Instruction Following
Dramatically improves adherence to output formats (JSON, XML, markdown), multi-step instruction reliability, constraint compliance, and chain-of-thought reasoning quality.

### 2.4 Style and Tone Adaptation
Adapts the model voice to brand guidelines, company communication standards, role-specific personas, and regional language varieties.

### 2.5 Knowledge Distillation
Use a large teacher model (GPT-4, Claude) to generate synthetic training data, then fine-tune a smaller student model (Mistral 7B, Phi-3) on it. The student approaches teacher quality on the target task at a fraction of inference cost.

### 2.6 Up-to-Date Knowledge Injection
Inject recent events, product specs, updated regulations, internal knowledge, or proprietary research beyond the model's pre-training cut-off.

### 2.7 Bias Mitigation
Reduce gender, racial, or cultural stereotypes; improve performance across demographic groups; align outputs with ethical guidelines; reduce toxicity.

### 2.8 Hallucination Reduction
Train models to say "I don't know" for out-of-domain queries, stay within known boundaries, express uncertainty appropriately, and cite sources when confident.

### 2.9 Cost Optimisation
A smaller fine-tuned model can replace a much larger base model on specific tasks, reducing inference cost by 10–100× and improving latency.

---

## 3. Full Fine-Tuning vs Parameter-Efficient Fine-Tuning (PEFT)

### Full Fine-Tuning
Updates **all model parameters**. Highest potential quality but massive memory requirements.

**Memory formula (full FT, FP16):** ~4× model size in GB (weights + gradients + optimizer states + activations). For a 7B model: ~28 GB minimum. For 70B: over 1 TB. Requires multi-GPU for models >7B. Produces a separate full model copy per task. Risk of catastrophic forgetting.

### Parameter-Efficient Fine-Tuning (PEFT)
Updates only a **small subset of parameters** (0.01–1%) while keeping most weights frozen.

**Advantages:** 70–99% less memory, can fine-tune 70B on a single GPU, near-full quality for many tasks, multiple tasks share same base model with different adapters, faster training cycles, lower forgetting risk.

### When to Choose Each

| Use Full FT When | Use PEFT When |
|-----------------|---------------|
| Maximum quality is non-negotiable | Compute is constrained (single GPU) |
| You have sufficient compute budget | Fine-tuning multiple tasks |
| Single task, long-term deployment | Rapid iteration matters |
| Models under 7B | Models >7B |
| Need significant behavioural change | Avoid catastrophic forgetting |

---

## 4. PEFT Methods Deep Dive

### 4.1 LoRA (Low-Rank Adaptation)
Injects trainable low-rank decomposition matrices into attention layers. For weight matrix W ∈ ℝ^(d×k), learns A ∈ ℝ^(d×r) and B ∈ ℝ^(r×k): W' = W + BA.

**Key hyperparameters:** `r` (rank, 8–128), `alpha` (scaling, typically 2× rank), `target_modules` (q_proj, v_proj for basic; full linear for comprehensive adaptation), `dropout` (0–0.1).

**Target modules by architecture:**
| Architecture | Common Targets |
|-------------|----------------|
| Llama / Mistral | q_proj, v_proj (or all linear) |
| GPT / BLOOM | query_key_value, dense |
| Falcon | query_key_value, dense |
| Gemma | q_proj, v_proj, k_proj, o_proj |
| Qwen | q_proj, v_proj, k_proj, o_proj |

**Pitfalls:** Too high rank overfits small datasets; too low rank misses task complexity; rank >64 rarely helps.

### 4.2 QLoRA (Quantized LoRA)
Combines **4-bit NF4 base model** + LoRA adapters. Key innovations: NF4 data type, Double Quantization (quantizes quantization constants), Paged Optimizers (GPU→CPU paging). Enables 65B fine-tuning on single 48GB GPU with 93–98% of full FT quality.

### 4.3 DoRA (Weight-Decomposed LoRA)
Decomposes weights into **magnitude + direction**, learns directional updates with LoRA. Consistently outperforms LoRA at the same rank. Same inference cost as LoRA (magnitude can be merged). Closer to full FT quality (96–100%).

### 4.4 AdaLoRA (Adaptive Budget Allocation)
Dynamically allocates parameter budget across layers based on importance. Uses SVD-based parameterisation; less important layers get lower rank or are pruned. Better quality at same total budget as uniform LoRA.

### 4.5 (IA)³ (Infused Adapter)
Learns scaling vectors for key, value, and feed-forward activations. Only 0.01–0.1% of parameters. Very low memory but lower peak quality than LoRA.

### 4.6 Prompt Tuning / Prefix Tuning
**Prompt Tuning:** Learns soft prompt tokens (5–100) prepended to input. Smallest parameter count. **Prefix Tuning:** Learns continuous prefix vectors at each transformer layer. More params than prompt tuning, higher quality for generation.

### 4.7 PEFT Method Comparison

| Method | Params Updated | Base Precision | Memory vs Full FT | Quality vs Full FT | Training Speed | Best For |
|--------|---------------|----------------|-------------------|--------------------|---------------|----------|
| LoRA | 0.1–1% | FP16/BF16 | 70–90% less | 95–99% | Fast | General PEFT |
| QLoRA | 0.1–1% | 4-bit NF4 | 95–98% less | 93–98% | Fastest | Single GPU training |
| DoRA | 0.1–1% | FP16/BF16 | 70–90% less | 96–100% | Fast | Higher quality than LoRA |
| AdaLoRA | 0.1–1% (adaptive) | FP16/BF16 | 70–90% less | 96–100% | Moderate | Adaptive budget |
| (IA)³ | 0.01–0.1% | FP16/BF16 | 95–99% less | 85–95% | Fastest | Lightweight tasks |
| Prompt Tuning | 0.001–0.01% | FP16/BF16 | 99% less | 70–90% | Fastest | Soft prompts |
| Prefix Tuning | 0.1–0.5% | FP16/BF16 | 90–95% less | 85–95% | Fast | Layer-level prompts |

---

## 5. Fine-Tuning Frameworks Comparison

### 5.1 Hugging Face Transformers + PEFT + TRL
**The industry-standard open-source stack.** Transformers for model loading/training, PEFT for LoRA/QLoRA/DoRA/AdaLoRA, TRL for SFT, DPO, PPO, ORPO. Accelerate for distributed training, BitsAndBytes for quantization.

**Pros:** Maximum flexibility, 200+ model architectures, extensive docs, largest ecosystem.  
**Cons:** More boilerplate than config-driven frameworks, steeper learning curve.

### 5.2 Unsloth
**Optimised LoRA/QLoRA training.** 2× faster training, 50% less memory usage vs standard HF. Supports Llama, Mistral, Gemma, Qwen, Phi. Zero quality loss. Custom Triton kernels.

**Memory example (QLoRA):** Llama 3.2 8B: ~8 GB vs ~16 GB standard. Llama 70B: ~28 GB vs ~48 GB standard.  
**Pros:** Fastest training, lowest memory, easy drop-in replacement for HF.  
**Cons:** Fewer model architectures, newer/smaller community.

### 5.3 Axolotl
**Config-driven fine-tuning (YAML).** Supports full + LoRA + QLoRA + DeepSpeed + FSDP. MLflow/W&B logging built in. Dataset mixture, prompt templating, gradient checkpointing, flash attention. Popular in open-source community.  
**Pros:** Reproducible configs, great for experiments, S3/GCS checkpointing.  
**Cons:** Less flexible than raw HF+TRL, YAML complexity for advanced use.

### 5.4 LlamaFactory
**Unified framework.** 100+ models, 5+ PEFT methods (LoRA, QLoRA, DoRA, AdaLoRA, (IA)³), RLHF (DPO, PPO, ORPO, KTO). Gradio web UI + CLI + Python API. Great for beginners.  
**Pros:** Easiest to use, web UI for non-engineers, broadest PEFT + RLHF support.  
**Cons:** Less flexible than HF+TRL, smaller community than Axolotl.

### 5.5 LitGPT (Lightning AI)
**Simple CLI:** `litgpt finetune lora`. Supports LoRA, QLoRA, full FT, quantization-aware training.  
**Pros:** Very easy CLI, Lightning ecosystem.  
**Cons:** Fewer architectures, less community adoption.

### 5.6 DeepSpeed (Microsoft)
**The dominant optimisation library for large-model training.** ZeRO stages (0/1/2/3) for memory sharding, ZeRO-Offload (CPU offload), ZeRO-Infinity (NVMe offload). Fused Adam optimizer.

| Stage | Memory per GPU vs DDP | Communication |
|-------|----------------------|---------------|
| ZeRO-1 | 4× reduction (optimizer) | Same as DDP |
| ZeRO-2 | 8× reduction (grad + optimizer) | Same as DDP |
| ZeRO-3 | Linear with #GPUs (ideal) | Increased |

**Pros:** Enables extreme-scale training, production-proven, HF integration.  
**Cons:** Complex config, ZeRO-3 can be slow (parameter gathering).

### 5.7 FSDP (Fully Sharded Data Parallel) — PyTorch
Native PyTorch distributed training. Shards params/gradients/optimizer states. CPU offloading, mixed precision, activation checkpointing.

**FSDP vs DeepSpeed:**
| Aspect | FSDP | DeepSpeed |
|--------|------|-----------|
| Native PyTorch | Yes | No |
| CPU offload | Yes | Yes |
| NVMe offload | No | Yes (ZeRO-Infinity) |
| Optimizer fusion | No | Yes |
| Complexity | Moderate | Complex |

### 5.8 NeMo (NVIDIA)
Enterprise framework. P-Tuning, LoRA, Adapter, full FT. NeMo Guardrails integration. Optimised for NVIDIA GPUs (A100, H100). Model parallelism (tensor + pipeline).  
**Pros:** Enterprise support, GPU optimisation, Guardrails.  
**Cons:** NVIDIA-dependent, complex setup.

### 5.9 Comprehensive Frameworks Comparison

| Feature | HF+TRL+PEFT | Unsloth | Axolotl | LlamaFactory | LitGPT | DeepSpeed | FSDP | NeMo |
|---------|-------------|---------|---------|-------------|--------|-----------|------|------|
| **Ease of Use** | Moderate | Easy | Easy | Very Easy | Very Easy | Complex | Moderate | Hard |
| **PEFT Methods** | All major | LoRA, QLoRA | LoRA, QLoRA | All major | LoRA, QLoRA | Via HF | Via HF | LoRA, Adapter, P-Tuning |
| **RLHF Support** | SFT, DPO, PPO, ORPO, KTO, CPO | No | SFT, DPO | SFT, DPO, PPO, ORPO, KTO | DPO | Via HF | Via HF | Via NeMo |
| **Memory Efficiency** | Moderate | Best (50% less) | Moderate | Good | Good | Best (Offload) | Good | Good (NVIDIA) |
| **Training Speed** | Baseline | 2× faster | Slightly slower | Baseline | Fast | Variable | Variable | Fast on NVIDIA |
| **Model Support** | 200+ | 25+ | 50+ | 100+ | 15+ | Depends | Depends | NVIDIA models |
| **Distributed Training** | Accel + DS + FSDP | Manual | DS + FSDP | DS + FSDP | Lightning | Native | Native | TP + PP |
| **Quantization** | 4/8-bit, AWQ, GPTQ | Native 4-bit | BitsAndBytes | BitsAndBytes | Native | N/A | N/A | INT8, FP8 |
| **Logging** | W&B, MLflow, TB | W&B | W&B, MLflow | W&B, MLflow | MLflow | W&B | TB | W&B, NeMo |
| **Web UI** | No | No | No | Yes (Gradio) | No | No | No | Yes |
| **Community** | Largest | Growing | Large | Medium | Small | Large | Large | Small |
| **License** | Apache 2.0 | Apache 2.0 | Apache 2.0 | Apache 2.0 | Apache 2.0 | Apache 2.0 | BSD | NVIDIA EULA |

---

## 6. RLHF and Alignment Frameworks

### 6.1 TRL (Hugging Face)
TRL provides trainers for each RLHF stage:
- **SFTTrainer** — supervised fine-tuning on demonstrations (cross-entropy loss on response tokens)
- **RewardTrainer** — trains reward model on (chosen, rejected) preference pairs
- **PPOTrainer** — full PPO RL loop with reward model, KL penalty, value model
- **DPOTrainer** — Direct Preference Optimization: no reward model, single-stage training from preference pairs
- **ORPOTrainer** — Odds Ratio PO: combines SFT + alignment in single stage, no reference model needed
- **CPOTrainer** — Contrastive PO: extension of DPO with contrastive learning
- **KTO** — Kahneman-Tversky Optimisation: only requires binary (good/bad) feedback

### 6.2 DPO (Direct Preference Optimization)
**Paper:** Rafailov et al., 2023. Directly optimises from preference pairs without a reward model. Uses closed-form mapping between reward and policy. Loss: L = -E[log σ(β(log πθ(y_w|x)/πref(y_w|x) - β(log πθ(y_l|x)/πref(y_l|x)))]. β controls deviation from reference model.

**Advantages:** Single-stage, stable, lower compute, most popular alignment method.  
**Disadvantages:** Sensitive to preference data quality, cannot incorporate complex reward signals.

### 6.3 PPO (Proximal Policy Optimization)
**Paper:** Schulman et al., 2017. Three-stage pipeline: SFT → Reward Model → PPO RL loop. Policy updates with clipped surrogate objective + KL penalty.

**Advantages:** Can optimise against any reward function, well-established, multi-turn task optimisation.  
**Disadvantages:** Complex pipeline, unstable training, expensive, sensitive to reward model quality.

### 6.4 ORPO (Odds Ratio Preference Optimization)
**Paper:** Hong et al., 2024. Combines SFT loss + preference loss in single forward pass. Uses odds ratio: OR(p_w, p_l) = (p_w/(1-p_w)) / (p_l/(1-p_l)). No reference model.

**Advantages:** Simpler than DPO, single-stage, competitive performance.  
**Disadvantages:** Newer method, less battle-tested.

### 6.5 GRPO (Group Relative Policy Optimization)
**Paper:** DeepSeek, 2025 (DeepSeek-R1). Generates multiple responses per prompt (group), computes group-relative advantages. No value network (saves ~50% memory vs PPO). Used to train DeepSeek-R1.

**Advantages:** No value network, stable group-based advantage estimation, proven at scale.  
**Disadvantages:** New, limited framework support outside DeepSeek.

### 6.6 Alignment Method Comparison

| Aspect | DPO | PPO | ORPO | GRPO | KTO |
|--------|-----|-----|------|------|-----|
| **Reward Model Needed** | No | Yes | No | No | No |
| **Training Stages** | 1 | 3 | 1 | 1 | 1 |
| **Reference Model** | Yes | Yes (KL) | No | No | Yes |
| **Complexity** | Low | High | Low | Medium | Low |
| **Stability** | High | Low-Med | High | Medium | High |
| **Data Required** | Preference pairs | Preference pairs | Preference pairs | Pairs + groups | Binary feedback |
| **Compute Cost** | Low | High | Low | Medium | Low |
| **Adoption** | Most popular | Well-established | Growing | Research stage | Growing |

---

## 7. Quantization-Aware Fine-Tuning

### 7.1 QLoRA
The dominant approach. Base model in **4-bit NF4** (optimal for normally distributed weights). **Double Quantization** quantizes quantization constants (FP32→FP8, saves ~0.5 bits/param). **Paged Optimizers** handle memory spikes.

**Memory breakdown (65B, QLoRA):**
```
Base model (4-bit):   ~32.5 GB
LoRA adapters (FP16): ~0.5 GB
Gradients + Opt:      ~1.5 GB
Activations (gc):     ~8 GB
Total:                ~42-45 GB (fits single 48GB GPU)
```

**Quality retention:** 93–98% of full FT quality. Gap shrinks for larger models.

### 7.2 GPTQ + Fine-Tuning
Quantise base model to 4-bit GPTQ using optimal brain quantisation, then fine-tune with LoRA. GPTQ uses calibration data + Hessian-based weight selection. Slightly lower quality than QLoRA for training.

### 7.3 AWQ + Fine-Tuning
Quantise to 4-bit AWQ (activation-aware weight quantisation), then LoRA fine-tune. Preserves critical weights based on activation distributions. Similar quality to GPTQ approach.

### 7.4 FP8 Fine-Tuning
Emerging approach on H100 GPUs. Forward pass in FP8 (tensor cores), master weights in FP16. 2× effective batch size. Requires H100+ hardware, limited framework support.

### 7.5 Best Practice Pipeline
```
Step 1: Fine-tune with QLoRA (4-bit base + FP16 adapters)
Step 2: Merge LoRA adapters into FP16 base model
Step 3: Quantise merged model for deployment (GPTQ / AWQ / GGUF)
Step 4: Deploy on inference server (vLLM, TGI, llama.cpp)
```

### 7.6 Inference Quantisation Comparison

| Method | Bits/Param | Quality Loss | Speed | Best For |
|--------|-----------|-------------|-------|----------|
| FP16 | 16 | None | Baseline | Highest quality |
| INT8 | 8 | Minimal | 1.5-2× | Good balance |
| NF4 (QLoRA) | 4 | 1-3% | 2-3× | QLoRA training |
| GPTQ | 4 | 2-5% | 2-3× | GPU inference |
| AWQ | 4 | 2-4% | 2-3× | Production GPU (vLLM) |
| GGUF Q4 | 4 | 2-5% | 2-3× | CPU/Edge |
| FP8 (H100) | 8 | Minimal | 2× | H100 deployment |

---

## 8. Data Preparation for Fine-Tuning

Data quality is the single most important factor — 1000 high-quality examples beat 100,000 noisy ones.

### 8.1 Dataset Formats

**Instruction (Alpaca/ShareGPT):**
```json
{"instruction": "Translate to French", "input": "Hello", "output": "Bonjour", "system": "You are a translator."}
```

**Multi-Turn Chat:**
```json
{"messages": [
  {"role": "system", "content": "..."},
  {"role": "user", "content": "..."},
  {"role": "assistant", "content": "..."}
]}
```

**Completion:** `{"prompt": "...", "completion": "..."}`

### 8.2 Data Quality Checklist
- Deduplication (exact + near-duplicate via MinHash / embeddings)
- Filter low-quality (gibberish, repetition, toxicity)
- Remove PII
- Balance classes
- Consistent formatting (same template for all examples)
- Length distribution normalisation
- Language consistency
- Annotation quality validation

### 8.3 Dataset Size Guidelines

| Use Case | Recommended Size |
|----------|-----------------|
| Instruction tuning | 100–1,000 |
| Domain adaptation | 1,000–10,000 |
| Knowledge injection | 10,000+ |
| Style/format tuning | 100–500 |
| Classification | 500–5,000 per class |
| Multi-turn chat | 5,000–50,000 turns |
| RLHF preferences | 10,000–100,000 pairs |

### 8.4 Prompt Formatting — Use the Model's Native Template

**ChatML (OpenAI):**
```
<|im_start|>system\nYou are...<|im_end|>\n<|im_start|>user\n...<|im_end|>\n<|im_start|>assistant\n...<|im_end|>
```

**Llama 3:**
```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>\n\n...<|eot_id|>\n<|start_header_id|>user<|end_header_id|>\n\n...<|eot_id|>
```

**Mistral:**
```
<s>[INST] ... [/INST] ... </s>
```

**Rule:** Always use the exact chat template the model was pre-trained with. Wrong format degrades quality significantly.

### 8.5 Train/Validation Split
- **90/10 or 95/5** — fine-tuning datasets are small; allocate enough for validation
- Stratified split to maintain class distribution
- Monitor for overfitting (validation loss increases while training loss decreases)

### 8.6 Data Augmentation
Paraphrasing via LLM, negative examples for rejection, difficulty mixing (easy + hard), template variation for robustness, multi-turn expansion from single-turn examples.

### 8.7 Dataset Tools

| Tool | Purpose |
|------|---------|
| Hugging Face Datasets | Loading, processing, sharing |
| Argilla / RLHF | Preference data collection |
| Llamaindex | Synthetic data from documents |
| Distilabel | LLM-based annotation/filtering |
| Cleanlab | Data quality assessment |
| LabelStudio | Human annotation |

---

## 9. Fine-Tuning Infrastructure

### 9.1 Single GPU Capabilities

| GPU | VRAM | QLoRA Max | LoRA Max | Full FT Max |
|-----|------|-----------|----------|-------------|
| RTX 3090/4090 | 24 GB | 13B | 7B | 3B |
| RTX 6000 Ada | 48 GB | 34B | 13B | 7B |
| A6000 | 48 GB | 34B | 13B | 7B |
| A100 (40GB) | 40 GB | 30B | 13B | 7B |
| A100 (80GB) | 80 GB | 70B | 34B | 8B |
| H100 (80GB) | 80 GB | 70B | 34B | 13B |

*Assumes 4-bit QLoRA, gradient checkpointing, and appropriate sequence lengths.*

### 9.2 Multi-GPU Strategies

| Strategy | How It Works | Best For |
|----------|-------------|----------|
| DDP | Full model on each GPU, split batch | Models fitting single GPU |
| FSDP (ZeRO-3) | Shard params/grad/opt across GPUs | Large models across GPUs |
| DeepSpeed ZeRO | Memory sharding + CPU offload | Very large models (70B+) |
| Tensor Parallel | Split layer weights across GPUs | 70B+ models |
| Pipeline Parallel | Split layers across GPUs | Extreme scale |

**Example commands:**
```bash
# QLoRA on single A100 80GB (70B model)
python train.py --model meta-llama/Llama-3.1-70B --quantize 4bit

# FSDP on 4× A100
accelerate launch --num_processes 4 train.py --model meta-llama/Llama-3.1-70B --use_fsdp

# DeepSpeed ZeRO-3 on 8× A100
deepspeed --num_gpus=8 train.py --model meta-llama/Llama-3.1-70B --deepspeed ds_config.json
```

### 9.3 Cloud GPU Options

| Provider | GPU Options | Best For | Notes |
|----------|-------------|----------|-------|
| AWS SageMaker | A100, H100, Trainium | Enterprise, managed | Full ML platform |
| AWS EC2 | All GPU types | Custom setups | p4d/p5 instances |
| GCP GKE | A100, H100, L4 | Kubernetes workflows | GPU node pools |
| Colab Pro+ | A100, V100 | Small experiments | Limited hours |
| Azure ML | ND A100/H100 v5 | Azure shops | DeepSpeed integration |
| Lambda Labs | A100, H100, A6000 | Research teams | Fast provisioning |
| RunPod | A100, A6000, 4090 | Budget experimentation | Cheapest option |
| Vast.ai | Community GPUs | Cheapest | Marketplace model |
| Modal | A100, H100 | Serverless | Per-second billing |
| Together AI | A100, H100 | Managed fine-tuning | API-based |

### 9.4 On-Prem Considerations
GPU cluster (min 4-8 GPUs/node), InfiniBand or NVLink interconnect, shared NFS/GPFS storage, checkpoint archiving (S3/GCS), cooling/power (300-700W per GPU), 25-100 Gbps networking.

### 9.5 Experiment Tracking

| Tool | Best For |
|------|----------|
| W&B | General tracking: curves, GPU use, dataset versioning |
| MLflow | Production: model registry, experiment tracking, deployment |
| TensorBoard | Quick visualisation |
| Aim | Open-source alternative |
| Comet | Team collaboration |

### 9.6 Infrastructure Decision Checklist
1. Model size → GPU count, quantisation strategy
2. Budget → cloud vs on-prem, spot vs on-demand
3. Latency requirements → model size choice
4. Data residency → cloud region
5. Team expertise → managed vs DIY
6. Re-training frequency → one-off vs continuous
7. Batch size requirements → GPU memory
8. Multi-node need → 70B+ full FT typically requires 2+ nodes

---

## 10. Evaluation After Fine-Tuning

### 10.1 Task-Specific Metrics

| Task | Metrics |
|------|---------|
| Text Classification | Accuracy, F1, Precision, Recall, AUC-ROC |
| Summarization | ROUGE-1/2/L, BERTScore, METEOR |
| Translation | BLEU, chrF, COMET, BERTScore |
| QA | Exact Match, F1 |
| Code Generation | Pass@k, HumanEval |
| Instruction Following | AlpacaEval, MT-Bench, Arena-Hard |
| Chat | LLM-as-Judge, human preference |

### 10.2 General LLM Benchmarks (Regression Testing)

| Benchmark | What It Measures | Typical (7B) |
|-----------|-----------------|-------------|
| MMLU | Multi-task knowledge (57 subjects) | 55-72% |
| HellaSwag | Commonsense reasoning | 70-85% |
| ARC-Challenge | Science reasoning | 55-70% |
| GSM8K | Grade-school math | 40-70% |
| HumanEval | Code generation | 30-60% |
| TruthfulQA | Truthfulness | 40-60% |

**Rule:** Compare fine-tuned model to base model. 2-5% drop is acceptable if task improves significantly. >10% drop = catastrophic forgetting.

### 10.3 Quality Evaluation Approaches

**LLM-as-Judge:** GPT-4/Claude rates outputs using rubric (1-5, pairwise, or criteria-based). Frameworks: AlpacaEval, MT-Bench, Chatbot Arena. **Pitfall:** LLM judges prefer longer outputs and their own style.

**Human Evaluation:** Gold standard for subjective quality (helpfulness, harmlessness, honesty). Pairwise A/B comparisons, inter-annotator agreement, 100-500 samples.

**A/B Testing:** Serve both models to users, track satisfaction/completion/engagement, statistical significance (p < 0.05).

### 10.4 Safety Evaluation

| Dimension | Tools |
|-----------|-------|
| Toxicity | Perspective API, Detoxify |
| Bias | WinoBias, BBQ, CrowS-Pairs |
| Hallucination | Factuality benchmarks, human fact-checking |
| Refusal rate | Safety benchmarks, red-teaming |
| Jailbreak resistance | Custom adversarial testing |
| Privacy | Membership inference, extraction tests |

### 10.5 Cost-Benefit Analysis

**Quantify improvement:** Task accuracy gain (%), error rate reduction (%), human review time saved (hrs/week).  
**Calculate costs:** Training compute ($), data preparation ($), evaluation ($), ongoing inference ($).  
**Compare alternatives:** Base model + prompting, base model + RAG, larger base model.  
**Decision:** Fine-tuning is worth it if (quality_gain × value_per_unit) > total_cost over model lifetime.

### 10.6 Baselines to Compare Against
1. **Base model** — is there improvement?
2. **RAG baseline** — is fine-tuning better than RAG for this task?
3. **Larger base model** — does fine-tuned 7B match base 70B?
4. **Closed-source API** — GPT-4/Claude quality and cost comparison
5. **Previous system** — rule-based or smaller model

---

## 11. When to Fine-Tune vs When to Use RAG or Prompting

### 11.1 Fine-Tune When
| Criterion | Reason |
|-----------|--------|
| Consistent output format required | Models format more reliably than prompting |
| Domain terminology is critical | Embed vocabulary into weights |
| Latency critical | Same latency as base (no retrieval overhead) |
| Offline capability needed | No external services required |
| Reduce hallucination on topics | Learn to say "I don't know" |
| Knowledge is static | Won't change frequently |
| Need specific style/voice | More natural than prompting |
| Deploying at scale | Fixed inference cost per query |
| Data privacy | Knowledge stays in model weights |

### 11.2 Use RAG When
| Criterion | Reason |
|-----------|--------|
| Knowledge changes frequently | Re-index, no retraining |
| Large knowledge base needed | Access millions of docs via retrieval |
| Source transparency needed | Cite exact sources |
| Risk of catastrophic forgetting | Fine-tuning can erase unrelated skills |
| Budget constraints | No training compute cost |
| Mix public + private knowledge | Blend retrieved docs with model knowledge |
| Rapid knowledge iteration | Hours (indexing) vs days (training) |
| Multiple domains | Same model, different vector stores |

### 11.3 Use Prompt Engineering When
| Criterion | Reason |
|-----------|--------|
| Task is simple | Well-crafted prompt + few-shots suffice |
| Small number of examples | In-context learning with 1-10 examples |
| Quick iteration needed | Change instantly, no training cycle |
| No domain adaptation needed | General knowledge is sufficient |
| Low query volume | Lower cost without training |
| Temporary/exploratory | Don't invest in fine-tuning for one-off tasks |
| Prototyping | Validate before committing |

### 11.4 Combined Approach: Fine-Tune for Behaviour + RAG for Knowledge

This is the **most common production pattern**:

```
Fine-tune model → instruction-following, output format, style, tool use, refusal behaviour
RAG → up-to-date knowledge at inference time
Result → model correctly processes retrieved context into well-formatted responses
```

**Architecture:**
```
User Query → Query Rewriter (fine-tuned) → Vector Search → Retrieved Documents
    ↓
Fine-tuned Generator (instruction-tuned + RAG-aware)
    ↓
Response + Citations
```

### 11.5 Decision Flowchart
```
Need new behaviour? → Simple → Try prompt engineering → Works? Done → Fail → Fine-tune
                  → Complex → Fine-tune
Need new knowledge? → Static → Fine-tune (inject into weights)
                   → Dynamic → RAG
Both? → Fine-tune for behaviour + RAG for knowledge
```

---

## 12. Fine-Tuning for Specific Tasks

### 12.1 Instruction Following
**Datasets:** OpenAssistant (OASST1) — 10K+ conversations; ShareGPT — 100K+ conversations; Dolly 15K; Self-Instruct (LLM-generated); LIMA — 1K high-quality examples.  
**Approach:** SFT on instruction-response pairs, ChatML template, mix instruction types, include edge cases.  
**Hyperparams:** LR 1e-5 to 2e-5 (LoRA), batch 8-128, epochs 1-3, 10% warmup.

### 12.2 Chat / Role-Playing
**Datasets:** Multi-turn conversation logs, character cards, RP-specific datasets (Pygmalion), LLM-generated persona conversations.  
**Approach:** Persona injection in system prompt, maintain context across turns, diverse scenarios.

### 12.3 Code Generation
**Datasets:** CodeAlpaca (20K), Magicoder-Evol-Instruct (110K), CommitPack, The Stack.  
**Approach:** Mix code + NL instructions, multiple languages, evaluate on functional correctness (HumanEval pass@1, MBPP, MultiPL-E).

### 12.4 Domain Adaptation
**Legal:** Contracts, case law, statutes. Tasks: contract analysis, legal research.  
**Medical:** Clinical notes, papers, drug databases. Tasks: clinical decision support, medical coding.  
**Financial:** Earnings calls, 10-K/Q filings, financial news. Tasks: financial analysis, risk assessment.  
**Approach:** (1) Optional domain continued pre-training (100M+ tokens), (2) task-specific SFT (1K-10K), (3) alignment with domain guidelines.

### 12.5 Summarization
**Datasets:** CNN/DailyMail, XSum, Multi-News, GovReport, custom doc-summary pairs.  
**Approach:** Long context (4K-128K), extractive + abstractive, length constraints, key point extraction. **Metrics:** ROUGE-L, BERTScore, human evaluation.

### 12.6 Classification
**Datasets:** Task-specific labelled data (500-5K per class), few-shot examples.  
**Approach:** Format output as label (+ reasoning), include label definitions in system prompt, balanced classes. Fine-tuned classification achieves 5-15% higher accuracy than prompting.

### 12.7 Tool Use / Function Calling
**Datasets:** Tool-use trajectories (call → observation → reasoning → action), function definitions in JSON schema, multi-step chains, error recovery.  
**Approach:** Format as function definitions → user query → tool call. Train on correct function + parameter selection, error handling, multi-step success. **Metrics:** Function call accuracy, parameter hallucination rate, multi-step success rate.

---

## 13. Fine-Tuning Framework Selection Guide

### 13.1 Decision Matrix

| If You Want... | Choose... | Why |
|----------------|-----------|-----|
| Maximum flexibility | HF + PEFT + TRL | Full training loop control, 200+ models |
| Fastest training, lowest memory | Unsloth | 2× speed, 50% less memory |
| Easy reproducible experiments | Axolotl | YAML config-driven, team-friendly |
| Quick start, web UI | LlamaFactory | Gradio UI, 100+ models |
| Simple CLI | LitGPT | `litgpt finetune` |
| Training 70B+ models | DeepSpeed | ZeRO-3, CPU/NVMe offload |
| PyTorch-native distributed | FSDP | No external deps |
| Enterprise NVIDIA | NeMo | GPU optimisation, Guardrails, support |
| RLHF/alignment | TRL (DPO/PPO/ORPO) | Most complete open-source RLHF library |
| Cost-effective iteration | Unsloth + QLoRA | Single consumer GPU training |

### 13.2 Selection by Team Profile

**Individual / hobbyist (consumer GPU):** Unsloth + QLoRA. **Individual (cloud GPU):** Axolotl or LlamaFactory.  
**Small team (rapid prototyping):** Axolotl (reproducible configs). **Small team (production):** HF Transformers + TRL. **Tracking:** W&B or MLflow.  
**Enterprise:** DeepSpeed + FSDP for training, NeMo for NVIDIA shops, SageMaker/Azure ML for managed infra. MLflow Model Registry or W&B Artifacts. vLLM/TGI for inference.  
**Startup (speed-focused):** Unsloth for iteration, HF Transformers for production customisation, Modal/RunPod/Lambda Labs for infra.

### 13.3 Starter Configuration Cards

**Single GPU QLoRA (7-13B, RTX 4090/A6000):**
```python
from unsloth import FastLanguageModel
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Llama-3.2-8B-bnb-4bit",
    max_seq_length=2048, dtype=None, load_in_4bit=True,
)
model = FastLanguageModel.get_peft_model(
    model, r=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", "gate_proj", "up_proj", "down_proj"],
    lora_alpha=16, lora_dropout=0, bias="none", use_gradient_checkpointing=True,
)
```

**Single GPU QLoRA (70B, A100 80GB):**
```python
from transformers import BitsAndBytesConfig
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True, bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4", bnb_4bit_compute_dtype=torch.bfloat16,
)
```

**Multi-GPU (4× A100):** `deepspeed --num_gpus=4 axolotl train config.yml`  
**Full FT (8B, 2-4× A100):** `accelerate launch --config_file fsdp_config.yaml train.py`

### 13.4 Common Pitfalls

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| Overfitting | Val loss ↑, train loss → 0 | Fewer epochs, higher rank, dropout, early stopping |
| Catastrophic forgetting | Benchmarks drop >10% | Mix 5-10% general data, lower LR, use PEFT |
| Training instability | Loss spikes, NaN | Lower LR, gradient clip (1.0), more warmup |
| Slow training | Low GPU utilisation | Increase batch size, flash attention, torch.compile |
| OOM | CUDA OOM | Smaller batch, gradient checkpointing, QLoRA, shorter sequences |
| No improvement | Same as base model | Check data quality, higher rank, try DoRA, verify chat template |
| Format failure | Ignores instructions | More format examples, consistent system prompt |

### 13.5 Recommended Learning Path
1. **Unsloth + QLoRA** on 7B model (consumer GPU) — fastest feedback loop
2. **Experiment with data quality** — 500 high-quality > 5000 noisy
3. **Try DoRA** — better than LoRA at same rank
4. **Move to Axolotl** for reproducible config-driven workflows
5. **Add RLHF/DPO** when SFT quality plateaus
6. **Scale to larger models** with DeepSpeed/FSDP on multi-GPU
7. **Productionise** with RAG integration + vLLM/TGI deployment

---

## References

1. Hu et al. (2021). "LoRA: Low-Rank Adaptation of Large Language Models." *arXiv:2106.09685*.
2. Dettmers et al. (2023). "QLoRA: Efficient Finetuning of Quantized Language Models." *arXiv:2305.14314*.
3. Liu et al. (2024). "DoRA: Weight-Decomposed Low-Rank Adaptation." *arXiv:2402.09353*.
4. Zhang et al. (2023). "AdaLoRA: Adaptive Budget Allocation for Parameter-Efficient Fine-Tuning." *arXiv:2303.10512*.
5. Rafailov et al. (2023). "Direct Preference Optimization: Your Language Model is Secretly a Reward Model." *arXiv:2305.18290*.
6. Hong et al. (2024). "ORPO: Monolithic Preference Optimization without Reference Model." *arXiv:2403.07691*.
7. DeepSeek (2025). "DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning." *arXiv:2501.12948*.
8. Schulman et al. (2017). "Proximal Policy Optimization Algorithms." *arXiv:1707.06347*.
9. Lester et al. (2021). "The Power of Scale for Parameter-Efficient Prompt Tuning." *arXiv:2104.08691*.
10. Li & Liang (2021). "Prefix-Tuning: Optimizing Continuous Prompts for Generation." *arXiv:2101.00190*.
11. Rajbhandari et al. (2020). "ZeRO: Memory Optimizations Toward Training Trillion Parameter Models." *SC20*.
12. Zhao et al. (2023). "LlamaFactory: Unified Efficient Fine-Tuning of 100+ Language Models." *arXiv:2403.13372*.

---

*This guide is part of the LLM/AI technical series in the [research](https://github.com/jackliusr/research) repository. See also: LLM Compression & Pruning Guide, LLM Quantization Guide, LLM Evaluation Guide.*
