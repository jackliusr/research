# Comprehensive Practical Guide to LLM Model Pruning

> **Author:** Jack Liu Shurui | **Date:** July 2026 | **Context:** Solution Architecture — Crédit Agricole CIB, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research) | **Series:** LLM Optimization

---

## 1. What Is LLM Pruning

LLM pruning is the process of removing unnecessary weights, neurons, attention heads, layers, or other components from a neural network to create a smaller, faster model while preserving accuracy. The core insight: large language models are significantly overparameterized — many weights contribute little to the model's output and can be removed with minimal impact on quality.

**Distinction from quantization:** Quantization reduces the *precision* of weights (e.g., FP16→INT4, where each weight uses fewer bits). Pruning removes weights entirely from the computation graph. They are complementary:

| Technique | What It Does | Compression Ratio | Precision Loss |
|---|---|---|---|
| Quantization | Fewer bits per weight | 2–8× | Yes (lower precision) |
| Pruning | Fewer weights total | 1.5–4× | No (remaining weights are full precision) |
| Both combined | Fewer weights × fewer bits | Up to 16× | Yes (quantization noise on fewer weights) |

**Golden rule:** Prune first, then quantize. This order consistently yields better final accuracy than quantizing first and pruning second, because pruning importance metrics work best on clean (unquantized) weights.

### Brief History
- **2015–2019:** Deep Compression (Han et al.) established pruning + quantization + Huffman coding for vision models
- **2020:** Lottery Ticket Hypothesis showed pruning can discover trainable subnetworks that match full network performance
- **2022–2023:** SparseGPT and Wanda demonstrated LLMs can be pruned to 50–70% sparsity in one shot with no fine-tuning
- **2024–2025:** Tools like SparseML, LLM-Pruner, and SliceGPT made pruning practical for production LLM deployments
- **2026:** Pruning + quantization pipelines are standard practice, with 2:4 semi-structured sparsity supported natively on NVIDIA Hopper/Ampere GPUs

### How Pruning Works
1. Compute an importance score for each weight (or group of weights)
2. Remove the lowest-scoring weights (set to zero)
3. Optionally fine-tune to recover accuracy
4. (Optional) Quantize remaining non-zero weights
5. Deploy using hardware/software that exploits sparsity for speedup

## 2. Why Prune LLMs

**Model size reduction:** 50% sparsity = ~50% less memory. A 70B FP16 model (~140GB) at 50% sparsity needs ~70GB — fits on one A100 80GB instead of two.

**Inference speedup:** Fewer parameters = fewer FLOPs. CPUs with DeepSparse: 1.5–3× at 50–90% unstructured sparsity. NVIDIA Ampere+ with 2:4 structured sparsity: ~1.5–2× via cuSPARSELt. Standard hardware: minimal unstructured speedup (need structured pruning).

**Edge deployment:** Consumer GPUs (RTX 4090) running 13B–30B models. CPU-only servers. Apple Silicon with limited unified memory.

**Reduced energy:** ~40–50% less energy at 50% sparsity at inference scale.

**Enabling larger models within same budget:** 70B at 50% sparsity fits in dense 35B memory.

**Complementary to quantization:** 50% sparse + INT4 = ~8× compression vs FP16 dense — current SotA for max compression.

## 3. Types of Pruning

### 3.1 Weight Pruning (Unstructured)
Removes individual weights with the smallest importance scores. Resulting weight matrices have zeros scattered arbitrarily throughout.

**Characteristics:**
- Highest compression ratio for a given sparsity target — maximum flexibility in which weights to remove
- Requires sparse matrix support for actual inference speedup
- Hardware support: NVIDIA GPUs do not accelerate random unstructured sparsity (only 2:4 pattern); CPUs can via DeepSparse engine, x86 SPARSE library, or MKL sparsity
- When no sparse hardware support exists, unstructured pruning still saves memory bandwidth but does not reduce FLOPs

**When to use:** CPU deployment via DeepSparse, or when maximum compression is needed and memory reduction alone (not speedup) is the goal.

### 3.2 Structured Pruning
Removes entire structures (heads, neurons, channels). Dense matrices maintained = hardware-friendly. Immediate speedup on any hardware. Less flexible than unstructured — lower compression at same quality.

### 3.3 Semi-Structured Pruning (2:4)
NVIDIA 2:4 pattern — exactly 2 non-zero out of every 4 consecutive weights. Best balance for GPU. Supported on Ampere+ GPUs via cuSPARSELt. ~1.5–2× speedup on linear layers. Requires training/fine-tuning with sparsity constraint.

### 3.4 Layer Pruning
Removes entire transformer layers (or blocks). Most aggressive compression approach. Can be surprisingly effective for very deep models (30+ layers) where middle layers may be redundant. Model architecture changes (layer count), so the pruned model is effectively a new architecture. Often combined with knowledge distillation to recover quality.

**How to choose which layers to remove:**
- Measure layer importance via gradient-based or activation-based metrics
- Common heuristic: remove 2–3 of the least important layers, evaluate perplexity, repeat
- Some research shows removing every other layer in very deep models can preserve surprising quality — the model appears to have built-in redundancy across depth
- Avoid removing the first 2 or last 2 layers (embedding-proximal and LM-head-proximal) — these tend to be the most critical

**When to use:** Extreme compression scenarios, or when the model is over-deep for the target task. Best combined with distillation.

### 3.5 Attention Head Pruning
Removes specific attention heads. Targets redundancy in multi-head attention. Well-studied in BERT/RoBERTa era. Less impactful than weight/layer pruning for modern LLMs.

### 3.6 FFN Neuron Pruning
Removes intermediate neurons in feed-forward layers. FFN layers are ~2/3 of total parameters — most effective single structured target. Can remove 25–50% of FFN intermediate dimensions.

## 4. Pruning Techniques

### 4.1 Magnitude Pruning
Remove weights with smallest absolute values. Simplest approach. Surprisingly effective at 50–70% sparsity. No calibration data needed. Quality degrades rapidly above 70%.

```python
def magnitude_prune(weight, sparsity):
    threshold = weight.abs().quantile(sparsity)
    return torch.where(weight.abs() > threshold, weight, 0.0)
```

### 4.2 SparseGPT
One-shot pruning to 50–70% sparsity without accuracy loss, no fine-tuning. Uses approximate second-order (Hessian) information — solves per-layer reconstruction (Optimal Brain Surgeon framework). Best post-training accuracy at high sparsity.

**Reported:** OPT-175B at 50% sparsity → perplexity increase < 0.1. LLaMA-65B at 50% → < 0.5.

### 4.3 Wanda (Weight-Aware Decomposition for Activations)
Simpler than SparseGPT. Importance = |weight| × ‖activation_row‖₂ — considers both weight magnitude and activation magnitude. Input-dependent. Faster than SparseGPT, comparable at 50% sparsity, slightly worse at 70%.

### 4.4 SparseGPT vs Wanda

| Dimension | SparseGPT | Wanda |
|---|---|---|
| Mechanism | Hessian-based weight update | Weight mag × activation norm |
| Complexity | Medium | Low |
| Accuracy at 50% | Excellent (Δperp ~0 on large) | Excellent (Δperp ~0.3–1.5) |
| Accuracy at 70% | Good | Moderate |
| Speed | Slower | Faster |
| Calibration | 128 samples | 128 samples |
| Retraining | No | No |

**Practical advice:** Try Wanda first (fast). If accuracy insufficient, try SparseGPT. At 50% sparsity both perform similarly for most models.

### 4.5 Iterative Pruning
Prune gradually over training via prune→retrain→repeat cycles. Each iteration removes a small percentage (5–10%) of remaining weights followed by recovery fine-tuning. Better final quality than one-shot at high sparsity (70%+). Higher training cost — typically 10–20 iterations. The model adapts to the sparsity structure gradually, unlike one-shot methods where all pruning happens simultaneously.

### 4.6 Gradual Magnitude Pruning
A specific iterative approach where sparsity increases slowly during training using a cubic schedule:

```python
sparsity_t = final_sparsity × (1 − (1 − t/T)³)
```

Where t is current training step and T is total pruning steps. The cubic schedule starts pruning slowly, accelerates in the middle, and plateaus near the end — giving the model time to adapt. Predictable, widely implemented in SparseML, PyTorch, and DeepSpeed.

### 4.7 Lottery Ticket Hypothesis
Training discovers subnetworks ("winning tickets") matching full network performance. Requires training from scratch with weight rewinding. Less practical for LLMs than vision.

### 4.8 Distillation + Pruning
Train smaller student to mimic larger teacher, then prune student further. Highest potential compression (5–10×). Highest compute cost.

### 4.9 Pruning at Initialization
Prune before training based on weight importance at initialization (SNIP, GraSP, SynFlow). Less effective than post-training for LLMs. Not recommended for production.

## 5. Practical Pruning Workflow (9 Steps)

### Step 1: Choose Pruning Target
| Goal | Target |
|---|---|
| Half model size | 50% unstructured |
| Fit on smaller GPU | Match memory budget |
| 2× latency reduction | 2:4 structured + quantization |
| CPU deployment | 70–90% unstructured (DeepSparse) |
| Max compression | 50% SparseGPT + INT4 |

**Formula:** `target_sparsity = 1 − (available_memory − activation_overhead) / model_weight_memory`

### Step 2: Select Pruning Type
```
Target NVIDIA Ampere+ GPU?
├── Yes → 2:4 semi-structured or structured
└── No → CPU with DeepSparse?
    ├── Yes → Unstructured weight pruning (70–90%)
    └── No → Structured (FFN + head)
```

### Step 3: Choose Pruning Technique
| Sparsity | Technique | Retraining Needed |
|---|---|---|
| 0–50% | Wanda or Magnitude | No |
| 50–60% | Wanda or SparseGPT | No (fine-tuning helps) |
| 60–70% | SparseGPT | Recommended |
| 70–90% | Iterative pruning | Yes |
| 90%+ | Lottery Ticket / distillation | Yes |

### Step 4: Calibrate on Representative Data
- **Size:** 128–1024 samples (128 sufficient)
- **Source:** Same domain as deployment
- **Common choices:** C4, The Pile, Wikipedia, or domain corpus
- **Avoid:** Data contamination — never use eval data for calibration
- **Format:** Tokenized at max context length or ~2048 tokens

### Step 5: Prune the Model
- **One-shot (Wanda/SparseGPT):** Single pass at target sparsity
- **Iterative:** 10–20 iterations, each removing 5–10% of remaining weights
- **Layer-wise vs global:** Layer-wise (same sparsity each layer) or global (pool all weights). Global can be better when redundancy varies per layer.
- **Exclude:** Embedding layers and LM head from pruning

### Step 6: Evaluate Pruned Model
**Minimum:** Perplexity on held-out data (quick sanity check). Task benchmarks (MMLU, HellaSwag, GSM8K). Inference speed on target hardware.

**Pass/fail:** 50% sparsity Δperp < 1 → OK. Δperp 1–3 → fine-tune. Δperp > 3 → reduce sparsity.

### Step 7: Fine-Tune If Needed
- **Recovery fine-tuning:** 1–10 epochs, LR 1e-5 to 5e-5, on task data
- **Knowledge distillation:** KL-divergence loss against original model
- **Combined:** Recovery + distillation

### Step 8: Quantize Pruned Model
Apply after pruning (never before): INT8 (2×), INT4 (4×), NF4. Same calibration data can be reused.

### Step 9: Deploy and Benchmark
Measure: tokens/sec, P50/P95/P99 latency, peak memory, model size, task accuracy.

## 6. Tools and Libraries

| Tool | Pruning Methods | Key Feature | Best For |
|---|---|---|---|
| **Neural Magic SparseML** | SparseGPT, Wanda, magnitude, gradual | Integrated pruning+quant+distill; DeepSparse CPU engine | End-to-end pipeline; CPU deployment |
| **Hugging Face Optimum** | Via SparseML integration | Simple API | HF ecosystem users |
| **SparseGPT (standalone)** | SparseGPT | Reference implementation | Research, any GPT model |
| **Wanda (standalone)** | Wanda | Simple, clean code | Fast experiments |
| **PyTorch** | Magnitude (torch.prune), 2:4 (torch.ao.sparsity) | Built-in, no dependencies | Quick prototyping |
| **NVIDIA TensorRT-LLM** | 2:4 structured | Native cuSPARSELt GEMM | NVIDIA GPU production |
| **LLM-Pruner** | Structural (heads, FFN, layers) | Group-wise structural pruning | When you need dense speedup |
| **SliceGPT** | PCA-style dimension removal | Dense smaller output model | Max compression without sparse support |
| **DeepSpeed** | Pruning during training | ZeRO + sparsity | Large-scale distributed training |

## 7. Code Examples

### 7.1 SparseGPT with SparseML

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from datasets import load_dataset
from sparseml.optim import SparseGPTModifier

model_name = "meta-llama/Llama-2-7b-hf"
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.float16, device_map="auto")
tokenizer = AutoTokenizer.from_pretrained(model_name)

# Load 128 calibration samples from C4
dataset = load_dataset("allenai/c4", "en", split="train", streaming=True)
calib = [next(iter(dataset))["text"] for _ in range(128)]
calib_enc = tokenizer(calib, return_tensors="pt", padding=True, truncation=True, max_length=2048)

# Apply 50% SparseGPT pruning
modifier = SparseGPTModifier(sparsity=0.5, mask_structure="unstructured", sequential_update=True)
model = modifier.apply(model, calib_enc)

# Export for DeepSparse (CPU inference)
from sparseml.export import export_model
export_model(model, output_dir="./pruned_llama_7b", tokenizer=tokenizer, opset=17)
```

### 7.2 Wanda Pruning on LLaMA

```python
import torch, torch.nn as nn
from transformers import AutoModelForCausalLM, AutoTokenizer
from datasets import load_dataset

model_name = "meta-llama/Llama-2-7b-hf"
sparsity_ratio = 0.5
nsamples = 128

model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype=torch.float16, device_map="auto")
tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token

# Load calibration data
traindata = load_dataset("allenai/c4", "en", split="train", streaming=True)
calib_texts = [next(iter(traindata))["text"] for _ in range(nsamples)]

# Wanda pruning
model.eval()
inputs = tokenizer(calib_texts, return_tensors="pt", padding=True, truncation=True, max_length=2048).input_ids.cuda()

act_scales = {}
def gather(name):
    def hook(mod, inp, out):
        act = inp[0].detach()
        if len(act.shape) == 3:
            act = act.permute(0,2,1).reshape(-1, act.shape[2])
        act_scales[name] = torch.norm(act, p=2, dim=0)
    return hook

handles = []
for n, m in model.named_modules():
    if isinstance(m, nn.Linear):
        handles.append(m.register_forward_hook(gather(n)))

with torch.no_grad():
    model(inputs)
for h in handles:
    h.remove()

for n, m in model.named_modules():
    if not isinstance(m, nn.Linear) or n not in act_scales:
        continue
    w = m.weight.data
    importance = w.abs() * act_scales[n].to(w.device).unsqueeze(0)
    k = int(w.shape[1] * sparsity_ratio)
    thresh = importance.kthvalue(k+1, dim=1, keepdim=True).values
    m.weight.data.mul_((importance >= thresh).float())

# Evaluate perplexity
test = load_dataset("wikitext", "wikitext-2-raw-v1", split="test")
test_text = "\n\n".join(test["text"])
enc = tokenizer(test_text, return_tensors="pt", truncation=True, max_length=4096).input_ids.cuda()
with torch.no_grad():
    loss = model(enc, labels=enc).loss
print(f"Perplexity: {torch.exp(loss).item():.2f}")
```

### 7.3 2:4 Structured Pruning with PyTorch

```python
import torch
from torch.ao.sparsity import apply_dynamic_sparsity, WeightNormSparsifier

# Apply to model
apply_dynamic_sparsity(model)  # Post-training (may need fine-tuning)

# Or with training (recommended)
sparsifier = WeightNormSparsifier(sparsity_level=0.5, sparsity_block_size=4, zeros_per_block=2)
for n, m in model.named_modules():
    if isinstance(m, torch.nn.Linear):
        sparsifier.prepare(m, ["weight"])

for epoch in range(3):
    for batch in dataloader:
        optimizer.zero_grad()
        loss = criterion(model(batch.input_ids), batch.labels)
        loss.backward()
        sparsifier.step()
        sparsifier.squash_mask()
        optimizer.step()
```

### 7.4 Magnitude Pruning with PyTorch

```python
import torch.nn.utils.prune as prune

params = [(m, "weight") for n, m in model.named_modules() if isinstance(m, torch.nn.Linear)]
prune.global_unstructured(params, pruning_method=prune.L1Unstructured, amount=0.5)
for m, _ in params:
    prune.remove(m, "weight")
```

## 8. Evaluation After Pruning

### Perplexity
Quickest indicator. Use Wikitext-2, C4 validation, or The Pile test set. Measure before and after.

| Sparsity | Typical Δperp (LLaMA-7B) | Assessment |
|---|---|---|
| 50% | 0.5–2.0 | Acceptable for most uses |
| 60% | 1.5–4.0 | May need fine-tuning |
| 70% | 3.0–8.0 | Fine-tuning strongly recommended |

### Task Accuracy
MMLU, HellaSwag, ARC, GSM8K. Rule: < 2% drop on MMLU or < 5% on GSM8K acceptable.

### Inference Speedup
Tokens/sec on target hardware. Report P50/P95/P99 latency.

### Sparsity Verification
```python
zeros = sum((p == 0).sum().item() for p in model.parameters() if p.dim() >= 2)
total = sum(p.numel() for p in model.parameters() if p.dim() >= 2)
print(f"Sparsity: {zeros/total:.1%}")
```

### Best Practices
1. Measure perplexity first (fast screening)
2. Measure task accuracy (expensive but necessary)
3. Benchmark on target hardware (speedup varies)
4. Compare to baselines: quantization-only, smaller dense model
5. Ablation: test 30%, 50%, 70% to find sweet spot

## 9. When to Prune vs Use a Smaller Model

### Pruning is better when:
- You have a large trained model and want a smaller version
- You need the same architecture (reuse deployment infra)
- You want to combine with quantization
- You have compute for recovery fine-tuning
- A small model of the desired size doesn't exist

### Smaller model is better when:
- A good small model already exists (LLaMA-3-8B may beat pruned LLaMA-2-13B)
- No compute for pruning + eval + fine-tuning
- Guaranteed quality needed (small model is known quantity)
- Simpler pipeline desired

**Rule of thumb:** If you have a 70B and need 7B performance → use a 7B model. If you need 13B performance at 7B size → try pruning 13B.

## 10. Pruning + Quantization Pipeline

**Always prune first, then quantize.** Quantized weights have noise; pruning metrics on noisy weights make worse decisions.

### Compression calculation
```
FP16 dense:      16 bits/weight
50% sparse:       8 effective bits/weight  (2×)
50% sparse + INT4: 2 effective bits/weight (8×)
```

**Example:** LLaMA-2-13B (26GB FP16) → 50% SparseGPT pruning → INT4 GPTQ = ~3.3GB

**Tools:** SparseML (pruning) + GPTQ (quantization) or NNCF for both. Same calibration data (128 samples) can be reused for both steps.

## 11. Banking Context

### Why pruning matters for banking

Banks face unique constraints that make pruning particularly relevant:

1. **On-premise LLM deployment with limited GPU:** Data residency regulations (MAS guidelines in Singapore, GDPR in Europe) often require keeping customer data on-premise. Banks may only have 1–2 A100 80GB GPUs per data center. Pruning enables fitting 70B-class models on this hardware — a 70B model at 50% sparsity + INT4 quantization fits in ~3.5GB of weight memory.

2. **Cost optimization at inference scale:** Large banks process millions of documents and customer interactions daily. Even a 10% reduction in per-query GPU cost translates to significant annual savings. Pruning directly reduces GPU hours and energy consumption.

3. **Compliance and audit requirements:** Model Risk Management frameworks (SR 11-7 in the US, MAS guidelines in Singapore) require that model changes be documented, reproducible, and validated. Pruning produces a deterministic, documented transformation — unlike distillation which changes architecture and requires full retraining.

### Typical banking scenarios
- **Internal document processing (trade finance, KYC):** 70B→50% sparse→INT4 on single A100. Batch processing, throughput > 100 pages/min.
- **Customer-facing assistant:** 8B→2:4 structured sparsity for < 500ms P99 latency on L40S GPU.
- **Regulatory reporting (air-gapped, CPU-only):** 7B→50% Wanda→INT4 via DeepSparse on CPU. Maximum compression needed for compliance.

### Evaluation for banking
- Domain-specific accuracy (regulatory compliance, document extraction), not just MMLU
- Latency SLAs: real-time < 1s, near-real-time < 10s, batch > 100 pages/min
- Documented pruning process required for audit

### Audit record template
```yaml
Base Model: meta-llama/Llama-3-70b-hf
Method: SparseGPT | Target: 50% | Achieved: 49.8%
Calibration: Internal banking corpus (256 samples)
Domain Eval: NER Δ=-0.3%, Compliance Q&A Δ=-1.1%
MMLU: 0.684→0.672 (Δ=-1.8%)
Post-pruning FT: 1 epoch | Quant: INT4 GPTQ g128
Engine: TensorRT-LLM
```

## 12. Pruning Technique Selection Guide

| Technique | Max Sparsity | Retrain? | Accuracy | Complexity | Hardware | Best For |
|---|---|---|---|---|---|---|
| **Magnitude** | 50–70% | No | Moderate at low sparsity | Very low | CPU (DeepSparse) | Quick experiments |
| **SparseGPT** | 50–70% | No | Excellent at 50%; good at 60–70% | Medium | CPU/GPU | Production, max quality |
| **Wanda** | 50–60% | No | Very good at 50%; moderate at 60% | Low | Any GPU | Fast iteration |
| **Iterative** | 70–90% | Yes | Good–excellent | High | Any framework | Max sparsity |
| **Gradual Magnitude** | 70–90% | Yes | Good | Medium | SparseML, PyTorch | Train-from-scratch |
| **Lottery Ticket** | 80–95% | Yes | Good at high sparsity | High | Any framework | Research |
| **Distillation+Pruning** | 80–95% | Yes | Best at extreme compression | Very high | Any framework | Max compression |
| **2:4 Semi-Structured** | 50% (fixed) | Recommended | Very good | Medium | NVIDIA Ampere+ | GPU production serving |
| **FFN Neuron** | 25–50% of dims | Recommended | Good | Low | Any hardware | Structured compression |
| **Layer Pruning** | 10–30% of layers | Recommended | Moderate | Low–medium | Any hardware | Extreme compression |
| **SliceGPT** | 20–30% of dims | No | Good | Medium | Any hardware | Smaller dense model |
| **Pruning at Init** | 50–70% | Yes | Poor vs post-training | Medium | Any framework | Not recommended |

### Decision matrix
| Use Case | Recommended | Secondary |
|---|---|---|
| Quickest compression | Wanda 50% | SparseGPT 50% |
| Max quality at 50% | SparseGPT 50% | Wanda + fine-tune |
| Max compression (size) | SparseGPT 70% + INT4 | Iterative 80% + INT4 |
| GPU speedup | 2:4 structured + TRT-LLM | FFN pruning + quant |
| CPU speedup | SparseGPT 70% + DeepSparse | Magnitude + DeepSparse |
| No FT compute | SparseGPT or Wanda | Magnitude |
| Unlimited compute | Distillation + pruning + quant | Iterative + distillation |
| Banking (audit) | SparseGPT 50% + documented pipeline | Wanda + documented |
| Consumer GPU | SparseGPT 50% + INT4 | Layer pruning + INT4 |

## 13. Common Pitfalls

1. **Using test data for calibration** — inflated accuracy that doesn't generalize.
2. **Pruning too aggressively** — start at 30%, evaluate, increase gradually.
3. **Ignoring activation memory** — weight memory shrinks but activation memory doesn't.
4. **Assuming unstructured speedup** — no speedup without sparse HW support.
5. **Quantizing before pruning** — always prune first.
6. **Skipping fine-tuning when needed** — Δperp > 2? Invest in recovery FT.
7. **Calibration data mismatch** — domain-specific calibration matters for banking.
8. **Validating only perplexity** — always check task benchmarks too.
9. **Not removing pruning masks** — `prune.remove()` before export.
10. **Pruning embedding / LM head** — exclude these critical layers.

## 14. References

### Papers
- SparseGPT: Frantar & Alistarh (2023). ICML 2023. arXiv:2301.00774
- Wanda: Sun et al. (2023). ICLR 2024. arXiv:2306.11695
- LLM-Pruner: Ma et al. (2023). arXiv:2305.11627
- SliceGPT: Ashkboos et al. (2024). ICLR 2024. arXiv:2401.15024
- Deep Compression: Han et al. (2016). ICLR 2016
- Lottery Ticket: Frankle & Carbin (2019). ICLR 2019
- Sparsity in DL: Hoefler et al. (2021). JMLR 2021

### Tools
- SparseML: github.com/neuralmagic/sparseml
- Wanda: github.com/locuslab/wanda
- SparseGPT: github.com/IST-DASLab/sparsegpt
- LLM-Pruner: github.com/horseee/LLM-Pruner
- SliceGPT: github.com/microsoft/TransformerCompression
- TensorRT-LLM: github.com/NVIDIA/TensorRT-LLM
- Optimum: github.com/huggingface/optimum

### Related guides
- [LLM Compression Alternatives Guide](llm_compression_alternatives_guide.md)
- [LLM Latency Optimization Guide](llm_latency_optimization_guide.md) (forthcoming)

---

> **Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB, Singapore
> **Last Updated:** July 2026 | **License:** Internal use — Crédit Agricole CIB
