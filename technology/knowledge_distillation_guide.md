# Knowledge Distillation — A Comprehensive Guide

> **Last updated:** July 2026
>
> A comprehensive reference on knowledge distillation (KD) — the model compression technique where a smaller "student" model is trained to replicate the behavior of a larger "teacher" model. Covers the fundamentals (dark knowledge, soft labels, temperature), the distillation objective, every major variant (offline/online/self, feature-based, attention-based, relational, multi-teacher, data-free, quantization-aware), distillation for large language models (Alpaca/Vicuna-style response distillation, chain-of-thought and reasoning distillation à la DeepSeek-R1-Distill), a step-by-step practical implementation with PyTorch code, tools and frameworks, real-world case studies (DistilBERT, TinyBERT, DistilWhisper, and more), and a dedicated banking section (credit scoring, fraud detection, AML, SR 11-7 governance) written with the Cymbal Bank lens in mind.

---

## Table of Contents

1. [What Is Knowledge Distillation?](#1-what-is-knowledge-distillation)
2. [Why Distill? Motivation and Use Cases](#2-why-distill-motivation-and-use-cases)
3. [Distillation vs. Other Model Compression Techniques](#3-distillation-vs-other-model-compression-techniques)
4. [Types of Knowledge in Distillation](#4-types-of-knowledge-in-distillation)
5. [How Knowledge Distillation Works](#5-how-knowledge-distillation-works)
6. [Distillation Variants](#6-distillation-variants)
7. [Knowledge Distillation for LLMs](#7-knowledge-distillation-for-llms)
8. [Practical Implementation](#8-practical-implementation)
9. [Tools and Frameworks](#9-tools-and-frameworks)
10. [Case Studies](#10-case-studies)
11. [Distillation for Banking](#11-distillation-for-banking)
12. [Summary and Further Reading](#12-summary-and-further-reading)

---

## 1. What Is Knowledge Distillation?

### 1.1 Definition

**Knowledge distillation (KD)** is a model compression technique in which a smaller model — the **student** — is trained to mimic the outputs and behavior of a larger, more capable model — the **teacher**. It was introduced in the landmark paper *"Distilling the Knowledge in a Neural Network"* by **Hinton, Vinyals, and Dean (2015)**, which remains the canonical reference for the field.

The intuition is simple: instead of training the small model from scratch on hard labels alone, we let it learn *from the teacher's predictions*, which are far more informative.

### 1.2 The Core Idea: Dark Knowledge

A classifier trained with cross-entropy on one-hot labels is optimized to produce a high probability for the *correct* class — but a well-trained network's output distribution contains much more than that. Consider a model trained to recognize vehicles: given an image of a truck, the predicted distribution might be `{truck: 0.75, car: 0.12, bus: 0.08, ...}`. The small probabilities assigned to *wrong* classes encode real information:

- **Similarity between classes** — the model "knows" a truck is closer to a car than to a cat, because both car and truck got non-trivial probability mass.
- **Confidence levels** — how sure the model is about a given input.
- **Internal representations** — features and abstractions learned in intermediate layers.

Hinton et al. called this the **"dark knowledge"** — knowledge that hard labels alone cannot convey. One-hot labels say only "this is a truck." Soft outputs say "this is a truck, but it resembles a car more than a bus." The student can absorb this richer signal and converge to better solutions than it could from labels alone — often with a fraction of the parameters.

### 1.3 The Training Setup

The student is trained on a **combination of two signals**:

1. **Teacher soft labels** — the teacher's softened output probabilities over the training data (the distillation signal).
2. **Ground-truth hard labels** — the original one-hot labels (the task signal).

By balancing these two, the student learns the task while inheriting the teacher's generalization structure.

### 1.4 What Distillation Achieves

- **Transfers knowledge** from a large model to a small model while **preserving most of the accuracy**.
- Typical outcomes: **3-10x smaller models** retaining **90-99% of teacher accuracy** — sometimes the distilled student even *outperforms* the teacher (small students trained on a great teacher's soft labels can generalize better than the teacher itself, a phenomenon observed in Born-Again Networks and self-distillation literature).
- Unlike quantization or pruning, which operate on an *existing* model, distillation is a **training-time** technique: the compressed model is trained directly into its smaller form.

### 1.5 Where Distillation Sits in the ML Lifecycle

| Aspect | Detail |
|---|---|
| **When it happens** | Training time (the compressed artifact is the trained student) |
| **What it needs** | A trained teacher (or a teacher trained jointly) + training data |
| **Output** | A smaller, independent model — no runtime dependency on the teacher |
| **Complementarity** | Orthogonal to quantization, pruning, and parameter sharing — all can be stacked |

---

### 1.6 A Brief History of Knowledge Distillation

KD has evolved from a niche ensemble-compression trick into the standard tool for building deployable models:

| Year | Milestone |
|---|---|
| 2015 | **Hinton et al.** formalize KD with temperature-scaled soft targets (*Distilling the Knowledge in a Neural Network*); **FitNets** introduce hint-based (feature) distillation |
| 2017 | **Attention transfer** (Zagoruyko & Komodakis) distills attention maps for CNNs |
| 2018 | **Born-Again Networks** show self-distillation improves generalization |
| 2019 | **DistilBERT** and **TinyBERT** make transformer distillation mainstream; **Relational KD** adds structure-aware distillation; first data-free KD methods appear |
| 2021-2023 | Distillation becomes central to **LLM alignment pipelines** (Alpaca, Vicuna, Zephyr via dDPO); **DistilWhisper** extends KD to speech recognition |
| 2024-2025 | **Reasoning distillation** (DeepSeek-R1-Distill) transfers chain-of-thought capability from a 671B MoE teacher into small dense students; distillation + RL (dDPO/GRPO) pipelines mature |

The trajectory is consistent: every time a larger model class becomes state-of-the-art, distillation is the mechanism that makes a *deployable* version of that capability available.

---

## 2. Why Distill? Motivation and Use Cases

The driving question is always: *"I have a model that performs great — why can't I just deploy it?"* Distillation exists because model quality and deployability frequently conflict.

### 2.1 Production Inference Budget

Large models are expensive at inference time. Every prediction consumes **latency, memory, and compute cost**. In production, inference cost scales with traffic: a model serving millions of predictions per day has a very real budget. Distillation produces a model that fits the budget while keeping most of the quality.

### 2.2 Edge Deployment (Mobile, IoT, Embedded)

Phones, IoT gateways, ATMs, and embedded sensors have strict limits on memory (hundreds of MB at most, often much less), battery, and compute. A 500M-parameter model cannot run on-device; a 50M-parameter distilled student can. Edge deployment also removes network round-trips and enables offline operation.

### 2.3 On-Device Inference

On-device inference has privacy and UX benefits: data never leaves the device, and predictions are instant. Distilled models are the standard route to making on-device NLP, vision, and speech features feasible (e.g., keyboard autocomplete, on-device translation, biometric fraud checks).

### 2.4 Model Efficiency at Scale

Serving millions of predictions per day means the *marginal cost per prediction* matters enormously. A model that is 60% faster and 40% smaller cuts infrastructure spend proportionally — same traffic, fewer GPUs, lower cloud bills.

### 2.5 Energy Efficiency

Smaller models consume less energy per inference. For organizations with ESG commitments or large batch workloads (daily AML scoring across millions of accounts), the energy savings of a distilled model are measurable and reportable.

### 2.6 Regulatory and Compliance Constraints

In regulated industries — banking especially — models may be required to run **on-premise or in air-gapped environments** where cloud-hosted large models cannot be used (customer data must not leave the bank). A distilled student that runs on bank-owned hardware satisfies the constraint; a 70B-parameter cloud model does not.

### 2.7 The Economics of Distillation: A Worked Example

Concrete numbers make the motivation tangible. Suppose a bank scores **10 million transactions per day** with a fraud model:

| Metric | Teacher (e.g., 350M-param neural ensemble) | Distilled student (e.g., 40M params, INT8) |
|---|---|---|
| Inference latency (p99) | ~45 ms on GPU | ~6 ms on CPU |
| Throughput (single node) | ~8K predictions/s | ~90K predictions/s |
| Serving nodes for 10M/day | 3 GPU nodes | 1 CPU node |
| Memory footprint | ~1.4 GB | ~45 MB |
| Energy per 1M predictions | ~2.1 kWh | ~0.2 kWh |

At cloud pricing this is the difference between a six-figure annual inference bill and a four-figure one — before counting the latency headroom that enables real-time (sub-100ms) fraud decisions on CPU infrastructure, with no GPU dependence.

### 2.8 Summary of Motivations

| Motivation | Typical setting |
|---|---|
| Inference budget (latency/memory/cost) | High-traffic production APIs |
| Edge deployment | Mobile, IoT, embedded devices |
| On-device inference | Privacy-sensitive or offline features |
| Efficiency at scale | Millions of predictions/day |
| Energy efficiency | ESG reporting, batch workloads |
| Regulatory/on-premise | Banking, healthcare, government |

---

## 3. Distillation vs. Other Model Compression Techniques

Distillation is one of four major model compression families. They attack the problem from different angles and are **complementary** — the best results come from combining them.

| Technique | Core idea | Typical compression | Runtime dependency | Notes |
|---|---|---|---|---|
| **Distillation** | Train a small model to mimic a large one | 3-10x | None (student is standalone) | Training-time; needs teacher + data; often combined with others |
| **Quantization** | Reduce numeric precision of weights/activations (FP32 → FP16, INT8, INT4) | 2-4x | None | Can be post-training or quantization-aware training (QAT); near-lossless at INT8 for many models |
| **Pruning** | Remove unimportant weights/neurons/channels (structured or unstructured) | 2-10x | None | Structured pruning (removing heads/channels) needed for real speedups |
| **Sharing / Factorization** | Share parameters across layers; factorize large matrices into low-rank products | 2-10x | None | e.g., ALBERT's cross-layer parameter sharing, low-rank adapters |

### 3.1 Why They Are Complementary

- **Distillation + quantization** — distill a large teacher into a small student, then quantize the student to INT8. This is *quantization-aware distillation* (QKD) and routinely yields sub-100MB models that run on CPUs.
- **Distillation + pruning** — distill, then prune the student further.
- **Distillation + factorization** — distill into a student architecture that already uses shared/low-rank parameters (e.g., TinyBERT's 4-layer design with shared layers).

A practical rule of thumb for production: **distill for architecture, quantize for hardware**. Distillation decides *what* the model looks like; quantization decides *how* it runs on the target hardware (GPU tensor cores like INT8/FP16, CPU SIMD, NPU accelerators).

---

## 4. Types of Knowledge in Distillation

Distillation research distinguishes three levels of knowledge that can be transferred from teacher to student. The choice of knowledge type defines the loss function and the variant of distillation used.

### 4.1 Response-Based Knowledge

- **What it is:** The teacher's final output — logits or softened probabilities (soft labels).
- **Most common form:** Hinton's classic KD, which matches the student's softened output distribution to the teacher's via KL divergence.
- **Pros:** Simplest to implement; requires no architectural alignment between teacher and student; works for any model that outputs logits (neural networks, and with some care, tree ensembles).
- **Cons:** Only captures the teacher's *final* decision; the teacher's intermediate reasoning is invisible.

### 4.2 Feature-Based Knowledge

- **What it is:** Intermediate layer representations — feature maps, hidden states, attention maps, activation patterns.
- **Representative work:** **FitNets** (Romero et al., 2015), which adds intermediate losses at multiple layers so the student learns to match the teacher's intermediate representations; attention-transfer methods that align attention maps.
- **Pros:** Richer signal — the student learns *how* the teacher thinks, not just what it concludes; enables thinner and deeper student architectures that response-based KD cannot train effectively.
- **Cons:** Requires architectural alignment or projection layers (e.g., a regressor that maps student features to teacher feature dimensions); more complex to tune (layer matching, loss weights per layer).

### 4.3 Relation-Based Knowledge

- **What it is:** Relationships *between* data samples rather than properties of individual samples — pairwise distances in feature space, triplet relationships, similarity graphs.
- **Representative work:** **Relational Knowledge Distillation (RKD)** (Park et al., 2019), which preserves pairwise and triplet relationships between data points.
- **Pros:** Captures structural knowledge — how the teacher organizes the data manifold; useful when sample relationships matter (retrieval, metric learning, graph tasks).
- **Cons:** Higher computational cost (pairwise computations over batches); less mature tooling.

### 4.4 Which Knowledge Type to Choose

| Knowledge type | Complexity | Signal richness | Best when |
|---|---|---|---|
| Response-based | Lowest | Final decisions | Fast baseline; heterogeneous teacher/student architectures |
| Feature-based | Medium | Intermediate reasoning | Thinner/deeper students; vision transformers; LLM distillation |
| Relation-based | Highest | Structural/relational | Metric learning, retrieval, few-shot tasks |

**General rule:** response-based is the simplest starting point; feature-based captures richer knowledge and is the workhorse for transformers; relation-based adds structural knowledge where sample relationships matter. **Deeper and larger teachers hold more knowledge to transfer** — a bigger teacher is not just more accurate, it is a richer distillation source, which is why LLM distillation from frontier models works so well.

---

## 5. How Knowledge Distillation Works

### 5.1 The Three Ingredients

1. **Teacher model** — large, pre-trained, high accuracy on the target task. Frozen during student training (in offline distillation).
2. **Student model** — smaller architecture (fewer layers, fewer parameters, lower hidden dimension). It is *not* a pruned copy of the teacher — it is an independently designed smaller network.
3. **Training data** — the same (or a subset of the) data used to train the teacher.

### 5.2 The Loss Function

The student optimizes a **weighted combination of two losses**:

```
distillation_loss = α × (student vs. teacher soft labels)
                 + (1 - α) × (student vs. ground truth)
```

- **Distillation loss** — KL divergence between the student's softened output distribution and the teacher's softened output distribution.
- **Student loss (task loss)** — cross-entropy between the student's logits and the one-hot ground-truth labels.

### 5.3 Soft Labels and Temperature

Hard labels are one-hot. Soft labels are probability distributions. To expose the dark knowledge in the teacher's logits, we scale them by a **temperature T** before applying softmax:

```
softmax(q_i / T)
```

- **T = 1** — standard softmax, the normal probability distribution.
- **T > 1** — softer distribution: probabilities flatten, and the relative ordering of the *small* probabilities (the dark knowledge) becomes visible to gradient-based training.
- **Higher T → softer distribution → more information about class similarities** (but eventually a uniform distribution that carries no information).

**Temperature scaling is the single most important trick in KD.** Hinton's insight: when the student is trained on soft targets, the gradient with respect to each student logit is proportional to `(teacher_probability − student_probability) / T`, and the overall gradient magnitude scales as `1/T²`. This is why the distillation loss is multiplied by **T²** — to keep gradient magnitudes comparable to the hard-label loss when using larger temperatures.

### 5.4 The Distillation Objective

The complete objective, using teacher logits `z_t` and student logits `z_s`:

```
L = α · T² · KL( softmax(z_t / T) ‖ softmax(z_s / T) ) + (1 − α) · CE(z_s, y_true)
```

Where:

- `KL` — Kullback-Leibler divergence between the softened teacher and student distributions.
- `CE` — cross-entropy with the ground-truth hard labels `y_true`.
- `T` — temperature (typically 2-8; 4 is a common default).
- `α` — weight balancing the two terms (typically 0.5-0.9; 0.7 is a common default).
- `T²` — scaling factor that compensates for the `1/T²` gradient shrinkage of the KL term.

### 5.5 Temperature and Alpha Tuning

| Setting | Effect of going too far |
|---|---|
| **T too high** | Distribution becomes near-uniform; too much smoothing; loss of discriminative information |
| **T too low** | Approaches hard labels; minimal distillation benefit |
| **α too high** | Student overfits the teacher's errors (the teacher is not perfect) |
| **α too low** | Student ignores the teacher and learns from hard labels only — no distillation |

**Good starting points: T = 4, α = 0.7.** Typical practice: anneal T down as training progresses (start soft, finish sharp), and validate on a holdout set — the student's own accuracy, not the KD loss, is the metric that matters.

### 5.6 Why Soft Targets Help: A Worked Intuition

On a vehicle classification task, hard labels tell the student "this is a truck." The teacher's soft distribution additionally says "this is a truck, somewhat car-like, slightly bus-like, nothing like a cat." Early in training, the student's gradient from the soft target pushes it to reproduce the *relative* structure — the "truck vs. car" boundary gets learned early because even before the student can classify trucks, it can learn that trucks are closer to cars than to bicycles. This smooth, structured training signal is what lets small students reach accuracy levels that training from hard labels alone cannot achieve.

---

### 5.7 Temperature in Practice: A Worked Example

Consider teacher logits `[2.0, 1.0, 0.1, 0.05]` over four classes, softened at different temperatures:

| T | Class 0 | Class 1 | Class 2 | Class 3 | Character |
|---|---|---|---|---|---|
| 1 (standard softmax) | 0.60 | 0.22 | 0.09 | 0.09 | Near one-hot; dark knowledge invisible |
| 2 | 0.42 | 0.26 | 0.16 | 0.16 | Inter-class structure emerging |
| 5 | 0.31 | 0.26 | 0.22 | 0.21 | Smooth; similarity signal dominant |
| 10 | 0.28 | 0.25 | 0.23 | 0.23 | Approaching uniform; discriminative info fading |

At T=1 the student sees mostly "class 0 wins." At T=5 it sees "class 0 is clearly best, class 1 is a decent second, classes 2-3 trail" — exactly the structure Hinton called dark knowledge. At T=10 the distribution carries so little contrast that the student learns almost nothing from it. This is why T is tuned in the 2-8 range: soft enough to reveal structure, sharp enough to keep it informative.

---

## 6. Distillation Variants

The KD family has grown far beyond Hinton's original formulation. Variants differ along three axes: **when the teacher is trained** (offline/online/self), **what knowledge is transferred** (response/feature/relation), and **what the teacher is** (single/ensemble/cross-modal/generative).

### 6.1 Offline Distillation (Classic)

- **Setup:** Teacher is pre-trained and frozen; student is then trained separately against the frozen teacher's outputs.
- **Properties:** Simple, stable, the classic approach; the teacher's soft labels can be pre-computed once and reused (huge compute saving — one forward pass per sample, amortized over all student training runs).
- **When to use:** Almost always, unless you lack a pre-trained teacher.

### 6.2 Online Distillation (Mutual Learning)

- **Setup:** Teacher and student are trained **simultaneously**; both networks learn from each other (deep mutual learning, DML). No pre-trained teacher is needed — the "teacher" can be a peer of the same size, or a small ensemble.
- **Properties:** Useful when no good teacher exists yet; the ensemble of peer models often outperforms any single member; more complex to schedule (who teaches whom, when?).
- **When to use:** Cold-start scenarios, or when the "teacher" is really a committee of peers.

### 6.3 Self-Distillation

- **Setup:** A model distills **itself** — the teacher is the same architecture at an earlier training stage (or a deeper branch of the same network).
- **Properties:** Improves generalization **without any size reduction**; related to Born-Again Networks (Furlanello et al., 2018), where a network retrains on its own previous predictions and improves.
- **When to use:** When you cannot shrink the model but want a free accuracy boost; also used as a regularizer.

### 6.4 Feature-Based Distillation (FitNets)

- **Setup:** The student is guided to match the teacher's **intermediate feature maps**, with intermediate losses at multiple layers. Because feature dimensions rarely match, the student adds a **regressor layer** that projects its features to the teacher's dimension (hint-based training).
- **Properties:** Enables **thinner and deeper** students that response-based KD cannot train (a thin student has too few parameters to absorb all knowledge through the final layer alone); the standard approach for modern transformer distillation (TinyBERT, DistilBERT's hidden-state loss).
- **When to use:** Aggressive compression (e.g., 110M → 14M parameters), or when response-based KD plateaus.

### 6.5 Attention-Based Distillation

- **Setup:** Instead of (or in addition to) features, the student matches the teacher's **attention maps** — which tokens attend to which.
- **Representative work:** Attention Transfer (Zagoruyko & Komodakis, 2017).
- **Properties:** Works particularly well for vision transformers and BERT-style models, where attention maps are cheap to extract and carry rich structural information; attention loss is often combined with hidden-state loss in BERT distillation.

### 6.6 Relational Distillation (RKD)

- **Setup:** The student preserves the teacher's **pairwise and triplet relationships** between data samples — if the teacher embeds samples A and B close together, the student should too.
- **Properties:** Captures structural knowledge of the data manifold; useful when sample relationships matter (retrieval, metric learning, similarity search).
- **When to use:** Embedding/similarity tasks; tasks where relative structure matters more than absolute scores.

### 6.7 Logit / Response Distillation

The classic Hinton formulation — the teacher's final logits are the only knowledge transferred. Already covered in Section 5. Still the default baseline for most projects because it is architecture-agnostic and trivially implemented.

### 6.8 Multi-Teacher Distillation

- **Setup:** An **ensemble of teachers** supervises one student. Teacher outputs can be averaged, or each teacher contributes its own loss term (possibly weighted).
- **Properties:** Each teacher contributes different knowledge (different architectures, different training data, different strengths); students trained on ensembles typically beat students trained on any single teacher.
- **When to use:** When you have several good models (e.g., a gradient-boosting model, a neural model, and a fine-tuned transformer) and want one deployable model.

**In practice** the teachers' softened outputs are often averaged (logit averaging) before computing the KL term, which acts as a cheap ensemble distillation; alternatively each teacher gets its own KL term with a per-teacher weight reflecting its validation performance. Multi-teacher KD is the standard way to collapse a heterogeneous model zoo — e.g., a bank's fraud stack of GBM + neural net + transformer — into a single deployable student.

### 6.9 Cross-Modal Distillation

- **Setup:** The teacher and student operate on **different modalities** — e.g., a vision teacher distills into a language student, or a multimodal teacher into a unimodal student.
- **Representative work:** CLIP-style distillation, where image and text encoders are aligned; audio-visual distillation for speech recognition.
- **When to use:** Building a lightweight unimodal model from a heavy multimodal teacher; modality transfer (e.g., teaching a text model what a vision model knows about an image).

### 6.10 Progressive Distillation

- **Setup:** Iterative step-by-step compression: distill a large model into a medium model, then distill the medium model into a small model.
- **Properties:** Each step only needs to preserve quality across a modest size reduction, so the final student retains more accuracy than a single aggressive jump (e.g., 70B → 8B directly).
- **When to use:** Extreme compression ratios; also used in diffusion-model acceleration (progressive distillation of samplers) and in speech recognition (Whisper large → small via an intermediate step).

### 6.11 Generative / Data-Free Distillation

- **Setup:** A **generator** (or the teacher itself) creates synthetic training data for the student — the student learns from teacher-generated samples, with the generator optimized so that the student's predictions diverge from the teacher's (GAN-style adversarial loop).
- **Properties:** Distillation **without the original training data**; essential when the original data is unavailable, proprietary, or too sensitive to reuse.
- **When to use:** Privacy-sensitive domains — banking customer data, healthcare records — where training data cannot be redistributed even internally.

**Notable techniques:** DeepInversion (inverts the teacher's batch-norm statistics into realistic synthetic images), zero-shot KD with adversarial generators (the generator is trained to maximize the discrepancy between student and teacher, forcing the teacher to teach the student its blind spots), and LLM self-play data generation (the teacher LLM generates its own instruction-response pairs). In banking, data-free distillation is often the difference between being allowed to distill and not: customer transaction data can be used to train a teacher once, but may not be redistributed to every downstream experiment.

### 6.12 Data-Free Distillation

Often used interchangeably with generative distillation: distillation without the original training set, using synthetic data or teacher-generated data. Two flavors:

- **Generator-based:** a GAN or diffusion model synthesizes inputs that maximally activate the teacher (data-free KD, Chen et al. 2019; DeepInversion).
- **Teacher-as-generator:** the teacher itself produces outputs (e.g., LLM-generated instruction data — this is exactly how Alpaca/Vicuna-style datasets are built, Section 7).

### 6.13 Quantization-Aware Distillation (QKD)

- **Setup:** Combine distillation with quantization: the teacher is full-precision, the student is a **quantized** (INT8/FP16) version of a smaller architecture, and distillation compensates for the accuracy lost to quantization.
- **Properties:** QKD-trained quantized students routinely beat post-training-quantized models of the same size; this is the standard recipe for deploying sub-100MB NLP models on CPUs.
- **When to use:** Whenever the deployment target is quantized hardware (CPU SIMD, NPU, TensorRT INT8).

### 6.14 Variant Selection Cheat-Sheet

| Scenario | Recommended variant |
|---|---|
| Standard compression, teacher available | Offline logit KD (T≈4, α≈0.7) |
| No teacher available | Online/mutual learning, or self-distillation |
| Aggressive compression (10x+) | FitNets-style feature KD, progressive distillation |
| Transformer/LLM compression | Hidden-state + attention + logit KD (TinyBERT recipe) |
| Embedding/retrieval tasks | Relational KD |
| Sensitive/unavailable training data | Data-free / generative distillation |
| Quantized deployment target | Quantization-aware distillation (QKD) |
| Several good models, one deployable | Multi-teacher distillation |

---

## 7. Knowledge Distillation for LLMs

Knowledge distillation is the dominant strategy for creating **small, capable large language models**. The typical pattern: distill a frontier-class model (GPT-4/Claude-class, or a large open model like Llama-3-70B or DeepSeek-R1) into a smaller open model (1B-32B) that can be self-hosted, fine-tuned further, and deployed cheaply.

### 7.1 Response Distillation (Instruction Distillation)

The student is **fine-tuned on the teacher's responses** to a large set of prompts. This is how the "mini me" family of models was created:

- **Alpaca (2023)** — distilled from **GPT-3.5**: 52K instruction-response pairs generated by the teacher, used to fine-tune LLaMA-7B.
- **Vicuna (2023)** — distilled from **ChatGPT**: ~70K conversations from ShareGPT, fine-tuned onto LLaMA-13B.
- The recipe is now standard: prompt collection → teacher generates responses → quality filtering → supervised fine-tuning (SFT) of the student.

### 7.2 Logit Distillation for LLMs

The student matches the teacher's **probability distribution over the vocabulary** (temperature-scaled, exactly as in Hinton KD). Used in decoder-only distillation to transfer the teacher's token-level confidence. Challenges: vocabulary mismatch between teacher and student, and the computational cost of full-vocabulary softmax matching. Often paired with hidden-state distillation.

### 7.3 Hidden-State Distillation

The student matches the teacher's **intermediate hidden states** at matched layers (with projection layers when hidden dimensions differ). This is the standard recipe for BERT-class distillation (DistilBERT, TinyBERT) and is increasingly applied to decoder LLMs (MiniLLM, LaMini-LM, distill-llm pipelines). Layer-matching strategies: uniform (every k-th layer), attention-based matching, or learned mapping.

### 7.4 Chain-of-Thought (CoT) Distillation

The student learns the teacher's **reasoning traces** — the step-by-step chain of thought the teacher produces before the final answer. Students trained on CoT traces exhibit the teacher's reasoning behavior (multi-step arithmetic, planning, structured deduction) even though CoT was never in their original training data. This transfers *process*, not just *outcome*.

### 7.5 Distillation of Reasoning Capabilities

Building on CoT distillation: the student learns to **generate rationales** — explanations and step-by-step thinking — which improves generalization to unseen problems. The 2024-2025 wave of reasoning-distilled models (DeepSeek-R1-Distill, and similar OpenAI-inspired reasoning distillations) showed that a small model fine-tuned on a large reasoning model's traces can acquire a large share of its reasoning ability.

### 7.6 Distillation from Large Teacher LLMs

| Teacher | Student examples | Notes |
|---|---|---|
| Llama-3-70B | Llama-3-8B (instruct variants) | Common open-to-open distillation path |
| DeepSeek-R1 (671B MoE) | **DeepSeek-R1-Distill-Qwen-32B / Qwen-14B / Qwen-7B / Qwen-1.5B, DeepSeek-R1-Distill-Llama-70B / Llama-8B** | Released January 2025; students fine-tuned on R1-generated reasoning samples; distilled Qwen-32B **outperforms OpenAI o1-mini** on multiple benchmarks — a landmark result proving small dense models can carry frontier reasoning |
| GPT-4 / Claude-class | Alpaca, Vicuna, Zephyr (via dDPO) | API-as-teacher; distillation data generated via API calls |

### 7.7 Practical Considerations for LLM Distillation

| Concern | Guidance |
|---|---|
| **Data collection** | Teacher-generated responses; **quality filtering** (dedupe, length/format checks, human or model-based review); ensure **diversity** of prompts (domains, difficulty, languages, formats) |
| **Data volume** | 10K-1M examples typical; more is not always better — 50K high-quality diverse samples often beat 500K noisy ones; quality-filtered subsets (e.g., Alpaca-GPT4, Orca) are the norm |
| **Evaluation** | Benchmark scores (MMLU, GSM8K, HumanEval, GPQA, domain benchmarks) + **human evaluation**; evaluate the student *independently* — teacher accuracy does not automatically transfer |
| **Compute cost** | Teacher inference for data generation can dominate (millions of forward passes) — amortize by generating data once; student training is standard SFT (or SFT + preference optimization) |
| **Safety** | Teacher **bias propagates** to the student; **hallucinations transfer**; alignment must be re-checked — distilled students often need their own safety tuning (RLHF/dDPO) after distillation |
| **Temperature** | Low T (deterministic, greedy) for response distillation of factual tasks; **T=0.2-1.0** for sampling-based generation of diverse training data; reasoning traces usually generated with moderate sampling temperature + temperature 0 for the final answer |

### 7.8 Distillation vs. RAG vs. Fine-Tuning for LLMs

| Approach | What it does | Best for | Trade-offs |
|---|---|---|---|
| **Distillation** | Train a smaller model to imitate a larger one | Smaller model, faster and cheaper inference, offline/on-premise deployment | Requires teacher access + data generation; capability ceiling set by teacher |
| **Fine-tuning** | Continue training a model on domain/task data | Domain adaptation, task-specific behavior, style/tone control | Needs labeled data; model size unchanged; can degrade general capabilities if overdone |
| **RAG** | Retrieve external knowledge at inference and feed it to the prompt | External/up-to-date knowledge, no retraining, answer grounding | No capability change; needs retrieval infra; latency and cost per query |

**In practice they compose:** distill for size, fine-tune for domain, RAG for fresh knowledge — a distilled 8B model fine-tuned on bank documents with RAG over policy manuals is a realistic production stack.

---

### 7.9 A Practical Mini-LLM Distillation Recipe

The end-to-end flow used for most open small-LLM projects (Alpaca-style through R1-Distill-style):

1. **Select the teacher** — the best model you can access: a frontier API (GPT-4/Claude-class), a large open model (Llama-3-70B), or a reasoning model (DeepSeek-R1).
2. **Curate the prompt set** — 10K-1M prompts spanning domains, difficulty levels, and formats; seed with existing public sets (Alpaca, Dolly, OpenOrca) and add domain-specific prompts (e.g., banking Q&A, document summarization).
3. **Generate responses** — batch inference with the teacher; use sampling temperature **0.2-1.0** for diversity and temperature 0 for final answers in reasoning traces; filter for quality (dedupe, length checks, refusal rate, hallucination screening).
4. **Fine-tune the student** — SFT on the teacher's responses with standard causal LM loss; for reasoning models, keep the full chain-of-thought in the training text.
5. **Preference-optimize** (optional but recommended) — dDPO/DPO on teacher-generated preference pairs to sharpen instruction-following and safety.
6. **Evaluate independently** — benchmarks (MMLU, GSM8K, HumanEval, domain evals) + human eval + safety/red-team testing; compare student vs. teacher and vs. baselines.
7. **Deploy and monitor** — serve with vLLM/TensorRT-LLM; track drift, hallucination rates, and safety incidents.

The pipeline is compute-dominated by step 3 (teacher inference), so generate data once, quality-filter hard, and reuse the dataset across student iterations.

---

## 8. Practical Implementation

A complete distillation project in seven steps, with PyTorch code.

### Step 1 — Prepare the Teacher Model

Use a large, pre-trained model **fine-tuned on the target task**. The teacher must be the best model you have — distillation cannot create knowledge, only transfer it. The teacher is **frozen** during student training.

```python
import torch
from transformers import AutoModelForSequenceClassification

teacher = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2)
teacher.load_state_dict(torch.load("teacher_finetuned.pt"))
teacher.eval()          # frozen — no gradient flows into the teacher
```

### Step 2 — Generate Teacher Soft Labels

Run a forward pass of the teacher over the training data and **save the logits** (or compute them on-the-fly each epoch). Pre-computing once and storing for reuse is dramatically cheaper — the teacher pass is amortized over all student training epochs and future student experiments.

```python
with torch.no_grad():
    for i, (inputs, _) in enumerate(train_loader):
        logits = teacher(**inputs).logits
        torch.save(logits.cpu(), f"teacher_logits/batch_{i:06d}.pt")
```

### Step 3 — Design the Student Architecture

The student is a **smaller architecture** chosen for the deployment target: fewer layers, smaller hidden dimension, fewer attention heads. Consider the **width vs. depth trade-off**: wider-and-shallower is often easier to train; deeper-and-thinner captures more hierarchy but is harder to distill into (FitNets exist precisely to solve this).

```python
from transformers import AutoConfig, AutoModelForSequenceClassification

config = AutoConfig.from_pretrained("bert-base-uncased", num_labels=2,
                                    hidden_size=384, num_hidden_layers=6,
                                    num_attention_heads=6, intermediate_size=1536)
student = AutoModelForSequenceClassification.from_config(config)
```

### Step 4 — Define the Loss

The combined loss: **KL divergence on softened outputs** (distillation term) + **cross-entropy on hard labels** (task term), with the T² scaling factor.

```python
import torch.nn.functional as F

def distillation_loss(student_logits, teacher_logits, labels, T=4.0, alpha=0.7):
    # Softened student log-probs vs. softened teacher probs (KL), scaled by T^2
    s_log_soft = F.log_softmax(student_logits / T, dim=-1)
    t_soft = F.softmax(teacher_logits / T, dim=-1)
    kd = F.kl_div(s_log_soft, t_soft, reduction="batchmean") * (T ** 2)
    # Hard-label task loss
    ce = F.cross_entropy(student_logits, labels)
    return alpha * kd + (1 - alpha) * ce
```

If using feature-based distillation (FitNets/TinyBERT style), add an MSE term on matched intermediate representations:

```python
def feature_loss(student_feats, teacher_feats, projector, T=4.0, beta=0.1):
    # projector maps student features to teacher feature dimensions
    return beta * F.mse_loss(projector(student_feats), teacher_feats)
```

### Step 5 — Train the Student

Optimize the combined loss, **tune T and α**, and monitor the student's own performance against the teacher on a holdout set.

```python
optimizer = torch.optim.AdamW(student.parameters(), lr=5e-5)

for epoch in range(epochs):
    for i, (inputs, labels) in enumerate(train_loader):
        with torch.no_grad():
            t_logits = torch.load(f"teacher_logits/batch_{i:06d}.pt")  # or teacher(**inputs).logits
        s_logits = student(**inputs).logits
        loss = distillation_loss(s_logits, t_logits, labels, T=4.0, alpha=0.7)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
    # Track the metric that matters: student accuracy vs. teacher accuracy on eval
    print(f"epoch {epoch}: student {eval_student():.4f} | teacher {eval_teacher():.4f}")
```

**Tuning tips:** grid-search T ∈ {2, 4, 6, 8} × α ∈ {0.5, 0.7, 0.9} on a small validation slice; if the student overfits the teacher, lower α; if the student ignores the teacher, raise α or T.

**Alternative: Hugging Face Trainer.** For transformer students, subclass `Trainer` and override `compute_loss` — this keeps the whole HF training stack (mixed precision, gradient accumulation, checkpointing):

```python
from transformers import Trainer

class DistillationTrainer(Trainer):
    def __init__(self, teacher, T=4.0, alpha=0.7, **kwargs):
        super().__init__(**kwargs)
        self.teacher, self.T, self.alpha = teacher, T, alpha

    def compute_loss(self, model, inputs, return_outputs=False):
        labels = inputs.pop("labels")
        s_logits = model(**inputs).logits
        with torch.no_grad():
            t_logits = self.teacher(**inputs).logits
        loss = distillation_loss(s_logits, t_logits, labels, self.T, self.alpha)
        return (loss, s_logits) if return_outputs else loss
```

This is the pattern used to reproduce DistilBERT-class results: initialize the student from the teacher's first k layers, then run the distillation Trainer.

### Step 6 — Evaluate

Compare the student against the teacher and against the pre-distillation baseline on **four axes**:

| Metric | What it measures |
|---|---|
| Accuracy (or task metric) vs. teacher | Quality preservation (target: 90-99% of teacher) |
| Parameter count / model size | Compression ratio |
| Inference latency (p50/p99) & throughput | Speedup on target hardware |
| Memory footprint, energy per inference | Deployment fit |

Also validate on slices: minority classes, edge cases, and adversarial inputs — distillation can silently degrade rare-class performance.

A minimal evaluation harness:

```python
def evaluate(model, loader):
    model.eval(); correct = total = 0
    with torch.no_grad():
        for inputs, labels in loader:
            preds = model(**inputs).logits.argmax(-1)
            correct += (preds == labels).sum().item(); total += len(labels)
    return correct / total

print(f"student: {evaluate(student, eval_loader):.4f}  teacher: {evaluate(teacher, eval_loader):.4f}")
```

Run the same harness for latency (time 1,000 inferences on the target hardware) and size (size of the exported artifact) to fill the Step 6 table.

### Step 7 — Deploy

Export the student to the deployment format and serve it. The student is a **standalone model** — no teacher needed at runtime.

```python
# ONNX export
dummy_input = torch.randint(0, 30000, (1, 128))
torch.onnx.export(student, dummy_input, "student.onnx",
                  input_names=["input_ids"], output_names=["logits"],
                  dynamic_axes={"input_ids": {0: "batch"}, "logits": {0: "batch"}})
```

Typical deployment paths: **ONNX Runtime** (CPU/GPU), **TensorRT** (NVIDIA GPUs, often INT8-quantized), **TFLite/CoreML** (mobile/edge), or a containerized model server (Triton, TorchServe, vLLM for LLMs). If the target is quantized hardware, apply quantization-aware distillation (Section 6.13) before export.

---

## 9. Tools and Frameworks

| Tool | Language/Platform | Distillation support | Notes |
|---|---|---|---|
| **PyTorch** | Python | Manual implementation | `torch.nn.KLDivLoss`, `F.kl_div`, `torch.distributions`; full control over loss, temperature, and schedule |
| **TensorFlow / Keras** | Python | `tf.keras.losses.KLDivergence` | Plus TensorFlow Model Optimization Toolkit for quantization-aware training; Keras custom training loops make KD straightforward |
| **Hugging Face Transformers** | Python | Trainer with custom loss; DistilBERT is the reference example | `Trainer` subclass overriding `compute_loss`; `DistilBertForSequenceClassification` ships pre-distilled |
| **OpenMMLab (MMClassification / MMDetection)** | Python | Built-in distillation configs | `kd` config entries for logit/feature/attention distillation in vision models |
| **NNI (Microsoft Neural Network Intelligence)** | Python | Distillation + compression modules | `nni.compression` covers KD, pruning, and quantization with a unified API |
| **TextBrewer** | Python | NLP-focused distillation toolkit | Specialized for transformer/LLM distillation (logit, hidden-state, attention losses) |
| **Intel Neural Compressor** | Python | Distillation + quantization | Combines KD with INT8 quantization for CPU deployment; also the home of Intel's Distiller-style workflows |
| **NVIDIA TensorRT** | C++/Python | Deployment of distilled models | Runtime/compiler for exported students; INT8/FP16 kernels |
| **Ultralytics YOLO** | Python | Distillation recipes | `yolo train` with teacher-student configs for object detection |
| **scikit-learn** | Python | Baseline distillation | Train a simple model on teacher soft labels via `log_loss`/`predict_proba` — a useful sanity baseline |
| **LightGBM / XGBoost** | Python/C++ | Tree distillation with teacher logits | Gradient boosting with teacher soft labels as the target; practical for tabular banking models |

**Implementation note:** for most projects, a manual PyTorch implementation (Section 8) is ~50 lines and gives full control. Use a framework when you need the extras — NNI/Intel Neural Compressor for compression pipelines, TextBrewer for transformer-specific losses, OpenMMLab for vision configs.

---

## 10. Case Studies

### 10.1 Results Summary

| Model | Teacher → Student | Size / speed | Quality retained | Year |
|---|---|---|---|---|
| **DistilBERT** | BERT-base (110M) → 66M | 40% smaller, 60% faster | 97% of BERT (GLUE) | 2019 |
| **TinyBERT** | BERT-base (110M) → 4-layer (14.5M) | 7.5x smaller, 9.4x faster | 96.8% of BERT (GLUE) | 2019 |
| **DistilGPT2** | GPT-2 (124M) → 82M | 36% smaller | ~98% of GPT-2 perplexity | 2019 |
| **DistilWhisper** | Whisper large-v2 → 6-layer English student | ~2x faster | 99% WER parity (within 1%) | 2023 |
| **DeepSeek-R1-Distill** | DeepSeek-R1 (671B MoE) → Qwen-32B/14B/7B/1.5B, Llama-70B/8B | Massive (MoE → dense) | Qwen-32B outperforms OpenAI o1-mini | 2025 |
| **Zephyr-7B** | Ensemble of large models → Mistral-7B | 7B | Top-1 open 7B on MT-Bench at release (dDPO) | 2023 |

### 10.2 DistilBERT (Hugging Face, 2019)

The canonical NLP distillation result. BERT-base (110M parameters, 12 layers) distilled into a 6-layer, 66M-parameter student using a **triple loss**: soft-label KL loss + hard-label cross-entropy + cosine-distance loss on hidden states. The student uses **no token-type embeddings** and shares the teacher's vocabulary. Result: **40% smaller, 60% faster, 97% of BERT's accuracy** on GLUE. It proved that transformer distillation works at scale and became the template for all subsequent BERT-class distillation.

Notable findings from the paper: removing the token-type embeddings costs nothing on GLUE; the cosine hidden-state loss is worth roughly +1 point over logit-only KD; and a student initialized from the teacher's first 6 layers converges far faster than a randomly initialized one.

### 10.3 TinyBERT (2019)

A 4-layer student (14.5M parameters, ~28% of the teacher's parameters and ~31% of its inference time) trained with a **two-stage** process — general-domain distillation then task-specific distillation — transferring **embedding, hidden-state, attention, and prediction-layer** knowledge. Result: **96.8% of BERT's GLUE performance at 7.5x smaller and 9.4x faster** — one of the most aggressive transformer compression results that still holds up.

The two-stage design matters: the general-domain stage transfers linguistic knowledge (embedding and attention structure); the task-specific stage re-distills on the task's own data. Skipping either stage costs several GLUE points.

### 10.4 DistilGPT2 (2019)

GPT-2 (124M) distilled to 82M parameters — **36% smaller, ~98% of the original's perplexity**. Demonstrated that decoder-only autoregressive models can be distilled by matching output distributions over the vocabulary.

### 10.5 ALBERT (2019)

Not strictly distillation, but the related compression family: **parameter sharing across layers** plus factorized embeddings. ALBERT shares all 12 layers' parameters, cutting BERT-base's 110M to ~12M unique parameters while keeping a 12-layer depth — a pure *sharing/factorization* approach (Section 3) that is often combined with distillation in later work.

### 10.6 MobileNet and Mobile Vision Distillation

The original Hinton paper's vision experiments distilled large ensembles into small students for **mobile deployment**. This line continued through MobileNet-class models, where distillation (often combined with quantization) is the standard recipe for shipping image models on phones — object detection, segmentation, and OCR models for mobile banking apps are typically distilled.

Concrete examples: MobileNetV2 and YOLO-nano students distilled from ResNet-class teachers for on-device object detection; and the Hinton paper's own motivating experiment — a large ensemble of CNNs distilled into a single small network that matched the ensemble's accuracy, which is where the whole field started.

### 10.7 DeepSeek-R1-Distill (2025)

The landmark LLM reasoning-distillation result. DeepSeek released six dense models (Qwen-32B/14B/7B/1.5B and Llama-70B/8B) fine-tuned on **800K reasoning samples generated by DeepSeek-R1** (a 671B MoE reasoning model). The distilled **Qwen-32B outperforms OpenAI's o1-mini** on multiple benchmarks, and even the 1.5B and 7B students show strong reasoning — proving that a large reasoning model's traces can transfer a large share of its capability into small dense models. (Note: the R1-Distill students are SFT on R1 traces; they did not inherit R1's RL pipeline — RL on the distilled models remains open work.)

### 10.8 DistilWhisper (2023)

Speech recognition distillation: Whisper **large-v2** distilled into a **6-layer English-only student** (Distil-Whisper). Result: **~2x faster** inference with **99% WER parity** (within 1% of the teacher's word error rate). Progressive/iterative training plus a carefully curated 21,000-hour distillation dataset. The reference result for distilling speech models.

### 10.9 The Small-LLM Wave (2023-2025)

| Model | Size | How it got small | Notes |
|---|---|---|---|
| **TinyLlama** | 1.1B | Trained from scratch on 3T tokens with Llama architecture | Often grouped with distilled small models, but is a from-scratch training run, not distillation; frequently used as a distillation *student* base |
| **Phi-1 / Phi-2** | 1.3B / 2.7B | "Textbook-quality" synthetic training data | Microsoft's small models trained on curated, textbook-style data; shows data quality can rival distillation |
| **Gemma** | 2B / 7B / 9B / 27B | Google's small-model family | Designed for on-device deployment; open weights with permissive license; the 2B is a common on-device LLM choice |
| **Llama-3-8B-Instruct** | 8B | Distilled-style SFT on Llama-3-70B-generated data | Strong small instruct model; the de-facto student base for 2024-2025 distillation work |
| **Zephyr-7B** | 7B | **dDPO** — distilled Direct Preference Optimization | Distilled from an ensemble of larger models (GPT-4/Claude-generated preferences); top open 7B on MT-Bench at release |

The common thread: **small models are built, not born** — whether via distillation, synthetic data, or preference distillation, every strong small model is trained against a larger source of knowledge.

---

### 10.10 Distillation Beyond Language and Vision

| Domain | Example |
|---|---|
| **Speech** | DistilWhisper (Section 10.8); student acoustic models for on-device voice assistants |
| **Tabular / banking models** | Distill gradient-boosting ensembles into compact neural or tree students trained on teacher logits (e.g., LightGBM's `softprob` objective with teacher probabilities as labels) |
| **Diffusion models** | Progressive distillation reduces the number of denoising steps (1000 → 4-8) — a *temporal* distillation of the sampling trajectory |
| **GANs** | Distill a large generator into a small one for real-time synthesis (e.g., StyleGAN → mobile) |
| **Recommender systems** | Distill two-tower/ranking ensembles into single lightweight towers for real-time personalization |

The pattern is domain-independent: wherever a heavy, high-quality model exists and a lightweight deployable version is required, distillation is the transfer mechanism.

---

## 11. Distillation for Banking

Distillation is a first-class technique for regulated, high-volume banking ML. The banking context is defined by three forces: **real-time decision latency**, **data privacy (data never leaves the bank)**, and **model governance (SR 11-7 / MAS-style validation)**.

### 11.1 Banking Use Cases

| Use case | Distillation pattern | Payoff |
|---|---|---|
| **Credit scoring** | Distill complex gradient-boosting or neural models into interpretable smaller models | Faster real-time scoring; on-premise deployment; simpler explainability |
| **Fraud detection** | Distill ensemble fraud models (GBM + NN + rules) into one compact real-time model | Sub-100ms transaction scoring; edge deployment at terminals/ATMs; single model to maintain |
| **AML transaction monitoring** | Distill complex models to manageable size for daily batch scoring of millions of transactions | Tractable batch windows; model explainability for suspicious-activity reports |
| **Customer analytics** | Distill recommendation/segmentation models | Real-time personalization in banking apps; cheap re-scoring |
| **Regulatory reporting** | Distill models for on-premise / air-gapped environments | Compliance where cloud models cannot run |

### 11.2 Benefits in Banking

- **Cost reduction** — smaller models = lower inference cost at scale (millions of daily scoring calls).
- **Latency** — faster scoring for real-time decisions (fraud, credit authorization).
- **Data privacy** — on-premise deployment means **data never leaves the bank**; distilled students run on bank-owned hardware.
- **Regulatory compliance** — smaller models are **easier to explain** (SHAP/LIME on a small model is tractable); **SR 11-7 model risk management favors simpler models that are easier to validate**.
- **Energy efficiency** — supports ESG commitments for batch workloads.
- **Edge deployment** — ATMs, branch terminals, and on-device fraud checks.

### 11.3 Challenges in Banking

| Challenge | Detail |
|---|---|
| **Data sensitivity** | Training data cannot leave the bank (or even be reused freely internally) — **data-free distillation** (Section 6.12) is often required |
| **Model governance (SR 11-7)** | Distilled models are new models, not copies: document the distillation process, validate the student **independently** (challenger tests, backtesting, benchmarking against the teacher) |
| **Bias propagation** | Teacher bias transfers to the student — fairness testing must be re-run on the student, not inherited from teacher sign-off |
| **Accuracy monitoring** | Student drift over time (data drift hits the smaller model differently); continuous monitoring required |
| **Regulatory audit** | Document distillation methodology, hyperparameters (T, α), data used, and teacher-vs-student performance comparison for auditors |
| **Interpretability vs. accuracy trade-off** | Distilled models may lose some accuracy; the governance conversation must set the acceptable loss threshold up front (e.g., ≤2% AUC drop) |

### 11.4 Mapping Distillation to the Model Risk Management Lifecycle (SR 11-7)

SR 11-7 (and its international analogues — MAS guidelines, EBA) treats a distilled student as a **new model**, not a copy of the teacher. The standard validation activities map as follows:

| SR 11-7 validation activity | What it means for a distilled model |
|---|---|
| Model development documentation | Record teacher version, distillation data, T, α, loss terms, training config |
| Data quality assessment | Verify the distillation dataset (teacher outputs + labels) is representative and unbiased |
| Testing / benchmarking | Challenger tests: student vs. teacher vs. baseline on holdout, backtest, and adversarial slices |
| Outcome analysis | Monitor student predictions vs. teacher over time; threshold drift and performance decay |
| Ongoing monitoring | Re-distillation triggers when the teacher improves or data drift is detected |
| Independent validation | Second-line review of the *student*, not inherited teacher sign-off |

**The governance rule of thumb:** if the audit trail cannot reproduce the student from the teacher + data + hyperparameters, the model does not pass validation.

### 11.5 Banking Best Practices

1. **Validate the student model independently** — never assume teacher accuracy transfers; run the student through the full model-validation lifecycle.
2. **Document the distillation process for audit** — teacher version, data, temperature, alpha, loss terms, training logs, evaluation results.
3. **Monitor student performance in production** — drift detection, performance decay, and alerting on the student, not just the teacher.
4. **Consider data-free distillation for sensitive data** — synthetic/generator-based approaches avoid re-exposing customer data.
5. **Combine with quantization for maximum compression** — distilled INT8 models are the standard CPU-deployment recipe.
6. **Re-distill regularly when the teacher improves** — a better teacher yields a better student; schedule re-distillation with model refreshes.
7. **Keep human oversight for critical decisions** — distillation is not a reason to remove human review from high-impact decisions (large credit exposures, SAR filings).
8. **Version both models in the model registry** — teacher and student are distinct artifacts with distinct lifecycles.
9. **Use clear naming** — `credit_model_v3_teacher` vs. `credit_model_v3_student_int8`; auditors must never confuse them.
10. **Set performance baselines up front** — record the teacher baseline and the student target (e.g., ≥95% of teacher AUC, ≤100ms p99) before training, and hold the student to them.

---

## 12. Summary and Further Reading

### 12.1 Key Takeaways

- **Knowledge distillation** trains a smaller student to mimic a larger teacher, transferring the teacher's **dark knowledge** — the inter-class similarity structure hidden in soft outputs — that hard labels cannot convey.
- The objective is a weighted blend: `α·T²·KL(softmax(z_t/T) ‖ softmax(z_s/T)) + (1-α)·CE(z_s, y)`, with **T≈4 and α≈0.7** as sensible starting points.
- Knowledge comes in three flavors — **response, feature, relation** — and the variants (offline, online, self, FitNets, attention transfer, RKD, multi-teacher, data-free, QKD) are choices along the axes of *when the teacher is trained*, *what knowledge is transferred*, and *what the teacher is*.
- Distillation is **complementary** to quantization, pruning, and parameter sharing — production-grade compression stacks them.
- For LLMs, distillation is the engine of the small-model ecosystem (Alpaca, Vicuna, DistilBERT, Zephyr, DeepSeek-R1-Distill): response distillation for instruction following, CoT/reasoning distillation for reasoning, hidden-state distillation for transformer compression.
- In banking, distillation delivers **cost, latency, privacy, and regulatory** wins — provided the student is validated, documented, monitored, and versioned as its own model.

### 12.2 Quick Reference Card

| Parameter | Default | Tuning guidance |
|---|---|---|
| Temperature T | 4 | 2-8 typical; raise for more similarity signal, lower if distribution goes uniform |
| Alpha α | 0.7 | 0.5-0.9; raise if student underfits teacher, lower if it overfits teacher errors |
| T² factor | Always apply | Compensates for 1/T² gradient shrinkage of the KL term |
| Teacher | Frozen, best available | Fine-tuned on the target task before distillation |
| Student | 3-10x smaller | Fewer layers/hidden units; width-vs-depth per deployment target |
| LLM distillation data | 10K-1M examples | Quality-filtered, diverse; sampling T=0.2-1.0 |
| Success bar | 90-99% of teacher accuracy | Measured on the student's own eval, not the KD loss |

### 12.3 Further Reading

- Hinton, Vinyals & Dean (2015) — *Distilling the Knowledge in a Neural Network* (the founding paper)
- Romero et al. (2015) — *FitNets: Hints for Thin Deep Nets*
- Zagoruyko & Komodakis (2017) — *Paying More Attention to Attention: Improving the Performance of Convolutional Neural Networks via Attention Transfer*
- Furlanello et al. (2018) — *Born-Again Neural Networks*
- Park et al. (2019) — *Relational Knowledge Distillation*
- Sanh et al. (2019) — *DistilBERT, a distilled version of BERT*
- Jiao et al. (2019) — *TinyBERT: Distilling BERT for Natural Language Understanding*
- Chen et al. (2019) — *Distilling Knowledge via Knowledge Matching* (data-free KD)
- Gou et al. (2021) — *Knowledge Distillation: A Survey*
- Taori et al. (2023) — *Stanford Alpaca*
- Tunstall et al. (2023) — *Zephyr: Direct Distillation of LM Alignment* (dDPO)
- Bain et al. (2023) — *Distil-Whisper: Robust Knowledge Distillation via Large-Scale Pseudo Labelling*
- DeepSeek-AI (2025) — *DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning* (and the R1-Distill model series)

---

*Guide written for the research repository of Jack Liu Shurui, Solution Architect at Cymbal Bank. Part of the LLM/AI optimization series alongside LLM optimization, model pruning, compression alternatives, and fine-tuning frameworks guides.*
