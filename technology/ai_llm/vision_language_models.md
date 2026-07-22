# Vision-Language Models (VLMs): A Comprehensive Guide

> **Jack Liu Shurui** — Research repo: [github.com/jackliusr/research](https://github.com/jackliusr/research)
>
> Last updated: July 2026

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Historical Evolution](#2-historical-evolution)
3. [Architecture Patterns](#3-architecture-patterns)
4. [Training Stages & Methodologies](#4-training-stages--methodologies)
5. [Top Models Comparison](#5-top-models-comparison)
6. [Benchmarks & Evaluation Metrics](#6-benchmarks--evaluation-metrics)
7. [Key Applications](#7-key-applications)
8. [Efficiency: Compression & Quantization](#8-efficiency-compression--quantization)
9. [Multilingual VLMs](#9-multilingual-vlms)
10. [Future Directions](#10-future-directions)
11. [References & Further Reading](#11-references--further-reading)

---

## 1. Introduction

Vision-Language Models (VLMs) are a class of multimodal AI systems that jointly understand visual content (images, videos, documents) and natural language. Unlike earlier computer vision systems that output fixed-label predictions, VLMs enable open-ended visual reasoning, captioning, question answering, and dialogue grounded in visual inputs. They represent one of the most transformative shifts in AI since the LLM revolution, combining the representational power of vision encoders with the generative and reasoning capabilities of large language models.

The core challenge in VLMs is **modality alignment** — learning a shared representation space where the semantic content of an image and its associated text are mapped close together, enabling cross-modal understanding and generation. Modern VLMs solve this through various architectural connectors (MLP projectors, Q-Former, cross-attention layers) trained on massive-scale image-text data.

By mid-2026, VLMs have matured from research curiosities into production-grade systems powering document processing pipelines, medical diagnostics, robotics control, video analytics, and consumer AI products. Open-weight models (LLaVA, Qwen-VL, InternVL) now compete within 5–10 points of proprietary frontier models (GPT-4o, Gemini 2.5 Pro) on standard benchmarks.

---

## 2. Historical Evolution

### 2.1 The Pre-CLIP Era (2015–2020)

The roots of vision-language modeling trace back to image captioning and visual question answering (VQA). Early task-specific architectures included **VSE++** (triplet-loss visual-semantic embeddings), **FiLM** (feature-wise linear modulation for cross-modal conditioning), **ImageBERT** (masked language modeling on image-text pairs), **ViLBERT** (two-stream co-attentional transformer), and **LXMERT** (transformer with three pretraining objectives). These models demonstrated the promise of multimodal pretraining but were limited by small datasets and task-specific designs.

### 2.2 CLIP and the Contrastive Revolution (2021)

**CLIP (Contrastive Language-Image Pretraining)** by OpenAI, January 2021, fundamentally changed the landscape. Key innovations:

- **Scale**: Trained on 400 million image-text pairs scraped from the internet (the WebImageText dataset).
- **Contrastive Objective**: A symmetric InfoNCE loss that pulls paired image-text embeddings together and pushes unpaired pairs apart in a shared 512/768-dimensional space.
- **Zero-Shot Transfer**: CLIP could be applied to any classification task by comparing an image's embedding against text prompts describing each class — no dataset-specific fine-tuning needed.
- **Architecture**: Dual-encoder design — a Vision Transformer (ViT) or ResNet for images, a standard Transformer for text, with both encoders trained jointly.

CLIP demonstrated that web-scale contrastive pretraining could produce remarkably robust visual representations. It became the backbone vision encoder for virtually all subsequent VLMs.

### 2.3 BLIP, BLIP-2, and the Q-Former Bridge (2022–2023)

**BLIP (Bootstrapping Language-Image Pretraining)** by Salesforce, 2022, introduced a unified framework combining understanding (contrastive learning, image-text matching) and generation (language modeling) in a single model. It also pioneered **caption filtering** — using a trained model to filter noisy web data for higher-quality training pairs.

**BLIP-2 (2023)** introduced the **Q-Former (Query Transformer)** — a lightweight, 188M-parameter trainable bridge connecting a frozen vision encoder (ViT) and a frozen LLM (OPT / FlanT5). The Q-Former uses 32 learnable query tokens that cross-attend to frozen visual features, compressing them into a fixed-length representation consumable by the LLM. This decoupled design meant BLIP-2 didn't need to train the massive backbone models end-to-end, making it dramatically more efficient.

### 2.4 Flamingo and the Cross-Attention Path (2022)

**Flamingo** (DeepMind, 2022) pioneered a different approach: inserting **gated cross-attention layers** between pre-trained, frozen LLM layers. A Perceiver Resampler compresses variable-length visual features into a fixed number of tokens, which are then injected into the frozen LM via cross-attention. Flamingo achieved impressive few-shot multimodal learning with as few as 4–8 examples, setting a new bar for in-context multimodal learning.

### 2.5 LLaVA and Simple Connectors (2023–2024)

**LLaVA (Large Language and Vision Assistant)** by UW-Madison / Microsoft, released April 2023, embraced simplicity. Rather than cross-attention or Q-Former:

- **Single linear MLP projection layer** maps CLIP visual features directly into the LLM embedding space.
- The LLM backbone (Vicuna, based on LLaMA) processes the projected visual tokens interleaved with text tokens — **all via standard self-attention**.
- Visual instruction tuning data was generated by prompting GPT-4 with image captions to create diverse question-answer pairs.

LLaVA showed that a single trainable projection layer is sufficient for strong multimodal performance when (a) the vision encoder is already contrastively aligned (CLIP) and (b) sufficient instruction-tuning data is used. This simplicity made it the most widely adopted VLM framework.

**LLaVA 1.5 (Oct 2023)** upgraded the MLP to a 2-layer GELU-activated network, added high-resolution image support (anyres), and expanded training data. **LLaVA 1.6 (Jan 2024)** further improved with dynamic high-resolution processing, better OCR, multilingual support, and extended visual input capabilities.

### 2.6 The GPT-4V / Proprietary Frontier (2023–2024)

**GPT-4V (OpenAI, September 2023)** was the first major proprietary VLM from a frontier AI lab, demonstrating that multimodal understanding at near-human level on complex visual reasoning tasks was achievable. It could interpret charts, diagrams, handwritten notes, photos, and screenshots, and engage in nuanced visual dialogue. While its architecture and training details remain undisclosed, GPT-4V is believed to use a massive-scale vision encoder integrated with GPT-4's LLM backbone through learned cross-modal connectors.

**Gemini (Google DeepMind, December 2023)** launched as a natively multimodal model trained from the ground up across text, images, audio, video, and code — unlike GPT-4V which appeared to bolt vision onto an existing text model. Gemini's architecture was designed for cross-modal reasoning from the start, with unified attention over multimodal token sequences.

**Claude 3 Vision (Anthropic, March 2024)** added vision capabilities to Claude 3, with strong performance on document understanding, data visualizations, and scientific diagrams.

**GPT-4o (May 2024)** unified text, vision, and audio processing in a single end-to-end multimodal model with dramatically reduced latency and improved performance across all modalities.

**Gemini 2.5 Pro (March 2025)** extended the trend with a 1M+ token context window, native video understanding, and deep reasoning capabilities, achieving state-of-the-art on MMMU, MathVista, and other key benchmarks.

### 2.7 Open-Source Renaissance: Qwen-VL, InternVL, CogVLM (2024–2025)

The open-source VLM ecosystem exploded in 2024–2025:

- **Qwen-VL (Alibaba, 2024)**: A series of VLMs built on Qwen LLM backbones, with bilingual (English/Chinese) capability, visual grounding (bounding box output), and strong OCR. Qwen-VL uses a ViT-based vision encoder with a cross-attention connector.

- **InternVL (OpenGVLab/Shanghai AI Lab, 2024)**: Scaled the vision encoder to 6B parameters — an order of magnitude larger than CLIP's ViT-L/14 (307M). This massive vision encoder, combined with a Q-Former connector, reduced data dependency significantly. InternVL 2.5 (Dec 2024) achieved near-GPT-4o performance with only 1/10th the training tokens of comparable models.

- **CogVLM (THUDM/Tsinghua, 2024)**: Introduced a deep-fusion architecture with a trainable visual expert module inserted at every LLM layer, going beyond shallow connector-based approaches.

- **Qwen2.5-VL (Jan 2025)**: Added native dynamic resolution processing, improved video understanding, GUI agent capabilities, and significantly stronger OCR. **Qwen3-VL (May 2025)** further improved with MoE architecture (235B total / 22B active), native dynamic frame sampling for video, and thinking mode for deep reasoning.

- **InternVL3 (mid-2025)**: Achieved GPT-4o-class performance on most benchmarks, demonstrating that open models could match proprietary frontier systems.

### 2.8 Evolution Timeline Summary

| Year | Milestone | Significance |
|------|-----------|-------------|
| 2017 | FiLM | Conditioning visual features on language |
| 2019 | ViLBERT, LXMERT | BERT-based multimodal pretraining |
| 2021 | CLIP | Web-scale contrastive vision-language alignment |
| 2021 | ALIGN | Scaling to 1.8B image-text pairs |
| 2022 | BLIP | Unified understanding + generation + caption filtering |
| 2022 | Flamingo | Gated cross-attention for few-shot multimodal learning |
| 2023 | BLIP-2 | Q-Former bridge between frozen ViT and LLM |
| 2023 | LLaVA | Simple MLP connector paradigm |
| 2023 | GPT-4V | First frontier proprietary VLM |
| 2023 | Gemini | Natively multimodal architecture |
| 2024 | LLaVA 1.6 | Dynamic high-res, SOTA open VLM |
| 2024 | GPT-4o | Unified text+vision+audio |
| 2024 | InternVL | 6B vision encoder, data efficiency breakthrough |
| 2025 | Gemini 2.5 Pro | 1M+ context, native video, SOTA benchmarks |
| 2025 | Qwen2.5-VL, InternVL2.5 | Open models approach GPT-4o |
| 2025 | Qwen3-VL, InternVL3 | Open models match proprietary frontier performance |

---

## 3. Architecture Patterns

Modern VLM architectures can be categorized by how they fuse visual and textual information. The key components are:

### 3.1 Vision Encoders

The vision encoder converts raw pixel data into feature representations. Common choices:

- **CLIP ViT**: Vision Transformer variants (ViT-B/32, ViT-B/16, ViT-L/14) pretrained via contrastive learning on 400M image-text pairs. The default backbone for most VLMs. Patch size (14, 16, 32) determines the resolution/compute trade-off.
- **SigLIP**: Google's Sigmoid-Loss variant of CLIP, replacing softmax-based contrastive loss with a sigmoid pairwise loss. This enables training with smaller batch sizes and achieves better performance than CLIP at equivalent compute. **SigLIP 2 (Feb 2025)** adds multilingual support, improved localization, and dense prediction features.
- **EVA-CLIP**: A CLIP variant trained with masked image modeling as an auxiliary task, achieving better visual representations than vanilla CLIP.
- **InternViT**: Scaled to 6B parameters, significantly outperforming smaller encoders on fine-grained visual understanding. Particularly impactful for document and OCR-heavy tasks.
- **DINOv2**: Self-supervised vision encoder (Meta, 2023), strong on semantic segmentation and visual correspondence, increasingly used as a complementary encoder alongside CLIP variants.

### 3.2 Language Models (Backbones)

The LLM backbone processes both text tokens and projected visual tokens:

- **Small**: Phi-3 (3.8B), Qwen2.5 (0.5–3B), Gemma (2–7B)
- **Medium**: LLaMA 3 (8B), Vicuna (7B/13B), Qwen2.5 (7–14B), Mistral (7B)
- **Large**: LLaMA 3 (70B), Qwen2.5 (72B), DeepSeek-V2 (67B)
- **Frontier**: GPT-4, Gemini Pro, Claude 3 Opus (proprietary)

The LLM can be frozen (BLIP-2 style), partially trained (LoRA adapters), or fully fine-tuned alongside the vision connector.

### 3.3 Connector / Fusion Layers

The connector bridges visual features from the encoder into the LLM's representation space. This is the key architectural differentiator:

#### (A) MLP Projector (LLaVA-style)
- **Mechanism**: Simple learned projection mapping visual patch tokens to LLM-compatible embeddings.
- **Pros**: Minimal added parameters, fast inference, easy to train.
- **Cons**: No cross-attention; visual tokens treated as "image words" in self-attention.
- **Used in**: LLaVA 1.5/1.6, LLaVA-NeXT.

#### (B) Q-Former (BLIP-2-style)
- **Mechanism**: BERT-initialized transformer with 32 learnable query tokens. Each query attends to visual features via alternating self-attention and cross-attention. Output query vectors slot into the LLM input stream.
- **Pros**: Fixed small token count (32), parameter-efficient (188M bridges frozen ViT + LLM), strong bottleneck regularization.
- **Cons**: Information bottleneck loses fine-grained details needed for OCR/dense captions.
- **Used in**: BLIP-2, InstructBLIP, VideoChat, some InternVL variants.

#### (C) Cross-Attention Layers (Flamingo-style)
- **Mechanism**: Perceiver Resampler compresses visual features → gated cross-attention layers inserted between frozen LLM layers.
- **Pros**: Visual info injected at multiple depths, frozen LLM preserves language capabilities.
- **Cons**: Computationally expensive, harder to scale, complex training.
- **Used in**: Flamingo, OpenFlamingo.

#### (D) Deep Fusion / Visual Experts (CogVLM-style)
- **Mechanism**: Each LLM layer gets a trainable visual expert module with separate QKV parameters for visual tokens. Base LLM weights frozen.
- **Pros**: Deeper visual-language interaction than shallow connectors, strong on detailed reasoning.
- **Cons**: Significant parameter overhead (38.8B additional for CogVLM-17B).
- **Used in**: CogVLM, CogAgent, GLM-4V.

#### (E) Perceiver Resampler
- **Mechanism**: Perceiver architecture uses latent queries to cross-attend to the full spatial feature map, producing compact output tokens.
- **Used in**: Flamingo, Kosmos-2, multimodal PaLM.

### 3.4 Dual-Encoder vs. Fusion Encoder vs. Unified Architecture

Three high-level paradigms:

| Paradigm | How It Works | Example Models | Use Case |
|----------|-------------|----------------|----------|
| **Dual Encoder (Two-Tower)** | Separate ViT + Text encoder, contrastive alignment in shared embedding space | CLIP, SigLIP, ALIGN | Retrieval, zero-shot classification, feature extraction |
| **Fusion Encoder** | Single multimodal encoder with cross-attention between visual and text streams | ViLBERT, LXMERT, FLAVA | VQA, entailment, cross-modal understanding |
| **Unified Generative** | Visual tokens projected into LLM, processed via self-attention with text tokens | LLaVA, Qwen-VL, InternVL, GPT-4V | Open-ended visual dialogue, captioning, reasoning |

Most production systems now use the **Unified Generative** paradigm, as it combines the generative fluency of LLMs with visual grounding.

---

## 4. Training Stages & Methodologies

VLM training typically proceeds through multiple stages. The standard recipe (inspired by LLaVA, Qwen-VL, InternVL pipelines) involves 3–4 stages:

### 4.1 Stage 1: Vision Encoder Pretraining

The vision encoder is pre-trained on large-scale image-text data, usually via contrastive learning. This stage produces an encoder whose output features are semantically aligned with language.

- **Objective**: Contrastive loss (InfoNCE for CLIP, sigmoid pairwise loss for SigLIP).
- **Data**: 400M–5B image-text pairs (WebImageText, LAION-400M/5B, DataComp-1B).
- **Typical duration**: Weeks on hundreds of GPUs.
- **Output**: A ViT whose [CLS] or patch embeddings can be directly compared with text embeddings via cosine similarity.

**Key insight**: Recent work (InternVL, EVA-CLIP) shows that larger vision encoders (1B–6B parameters) provide substantially better downstream performance and dramatically reduce the amount of training data needed for subsequent stages.

### 4.2 Stage 2: Multimodal Pretraining (Connector Training)

The connector/bridge is trained to map frozen vision encoder features into the LLM's embedding space. The LLM may be frozen or partially unfrozen.

Two sub-stages are common:

#### Stage 2a: Alignment Pretraining

- **Trainable components**: Connector only (MLP, Q-Former, or cross-attention layers). Vision encoder and LLM **frozen**.
- **Objectives**:
  - **Image-Text Contrastive (ITC)**: Align query output embeddings with text embeddings (Q-Former only).
  - **Image-Text Matching (ITM)**: Binary classification — does an image-text pair match?
  - **Image-Text Generation (ITG)**: Language modeling loss on text tokens conditioned on visual features.
  - **Language Modeling (LM)**: Standard next-token prediction on caption text, with visual tokens prepended.
- **Data**: Large-scale image-caption pairs (COCO, CC3M/12M, SBU, LAION subsets).
- **Typical size**: 10M–100M image-text pairs.

#### Stage 2b: Multi-Task Pretraining (Optional)

Additional objectives: masked image modeling (BEiT-style), OCR-aware text reading/grounding, and interleaved image-text data from natural documents (MMC4, OBELICS).

### 4.3 Stage 3: Multimodal Instruction Tuning (Supervised Fine-Tuning)

Adapts the VLM for conversational AI and instruction-following. Trainable: connector + LLM (full fine-tune) or connector + LoRA adapters.

- **Data**: Visual instruction-following datasets — LLaVA-Instruct-150K, LLaVA-Mix-665K, LRV-Instruction, SVIT, ShareGPT4V (1.2M GPT-4V-annotated pairs), MIMIC-IT. Format: multi-turn `(image, question, answer)` triplets.
- **Key hyperparameters**: LR 2e-5, cosine schedule, 1–3 epochs, batch size 128.
- **Key insight**: Data quality > quantity. GPT-4V-generated instructions outperform human-written or template-based data.

### 4.4 Stage 4: RLHF / Preference Optimization

Aligns VLM outputs with human preferences for helpfulness, accuracy, and safety.

- **Approaches**: RLHF (reward model + PPO), DPO (direct preference optimization — simpler, increasingly preferred), RLHF-V (dense rewards with hallucination penalties), and DPO with visual preference pairs (e.g., LLaVA-RLHF, Silkie).
- **Data**: Human preference pairs comparing VLM responses to visual queries (VLFeedback, LLaVA-RLHF dataset).
- **Challenges**: Visual hallucination (object presence, spatial relationships, counting), factual accuracy, culturally appropriate responses.

### 4.5 Training Data Scaling Laws

Empirical findings from recent VLM scaling work:

- Vision encoder size matters: 6B encoder + 7B LLM can match 300M encoder + 70B LLM on visual understanding tasks.
- Image-text pretraining data quality > quantity: filtered/high-quality subsets (e.g., LAION-aesthetic, CC12M-filtered) outperform raw web data at the same scale.
- Instruction data diversity is critical: mixing OCR, grounding, region-level, conversation, and reasoning data outperforms uniform caption-based instruction data.
- Dynamic high-resolution strategies (native resolution input, tiling for large images) produce substantial gains on document and chart understanding without losing efficiency on standard-sized images.

---

## 5. Top Models Comparison

### 5.1 Contrastive Vision-Language Models

| Model | Release | Encoder | Data | Architecture | Key Features |
|-------|---------|---------|------|-------------|--------------|
| **CLIP** (OpenAI) | Jan 2021 | ViT-L/14 (307M) | 400M pairs | Dual-encoder, contrastive (InfoNCE) | Zero-shot classification, retrieval, foundation for most VLMs |
| **OpenCLIP** (LAION) | 2022 | ViT variants | LAION-400M/5B | Same as CLIP, open-source | Reproduced CLIP with larger data, community-supported |
| **SigLIP** (Google) | 2023 | ViT-So400m | WebLI (5B pairs) | Dual-encoder, sigmoid pairwise loss | Better at small batch sizes, multilingual |
| **SigLIP 2** (Google) | Feb 2025 | ViT variants | WebLI expanded | SigLIP + localization + dense features | Multilingual, improved localization, dense prediction |
| **EVA-CLIP** (BAAI) | 2023 | EVA-ViT | LAION-400M | CLIP + masked image modeling auxiliary task | Better visual representations than vanilla CLIP |
| **ALIGN** (Google) | 2021 | EfficientNet | 1.8B noisy pairs | Dual-encoder, contrastive | Demonstrated massive noisy data works at scale |

### 5.2 Generative / Instruction-Tuned VLMs

| Model | Release | Encoder | LLM Backbone | Connector | Key Features |
|-------|---------|---------|-------------|-----------|-------------|
| **BLIP-2** (Salesforce) | Jan 2023 | EVA-ViT-G | OPT 2.7B / FlanT5-XL | Q-Former (188M) | Frozen encoder+LLM, efficient bridging, strong captioning/VQA |
| **InstructBLIP** (Salesforce) | May 2023 | EVA-ViT-G | Vicuna-7B/13B | Q-Former (+ instruction-aware queries) | First instruction-tuned Q-Former; SOTA on 13 benchmarks at release |
| **Flamingo** (DeepMind) | Apr 2022 | NFNet-F6 | Chinchilla (70B) | Perceiver + gated cross-attention | Few-shot multimodal learning, interleaved visual-text processing |
| **LLaVA 1.5** (UW-Madison) | Oct 2023 | CLIP ViT-L/14 | Vicuna 7B/13B | 2-layer MLP (GELU) | Simple, strong, widely adopted; 665K instruction-tuning data |
| **LLaVA 1.6** (UW-Madison) | Jan 2024 | CLIP ViT-L/14 | Vicuna 13B / LLaMA-3 8B | 2-layer MLP | Dynamic high-res, better OCR, multilingual, SOTA open VLM at release |
| **Qwen-VL** (Alibaba) | Jan 2024 | ViT-bigG | Qwen-7B / Qwen-14B | Cross-attention | Bilingual (EN/ZH), object grounding, bounding box output |
| **Qwen2.5-VL** (Alibaba) | Jan 2025 | ViT-dynamic | Qwen2.5 3B/7B/72B | Cross-attention | Dynamic resolution, video understanding, GUI agent, strong OCR |
| **Qwen3-VL** (Alibaba) | May 2025 | ViT-dynamic | Qwen3 (MoE, 235B/22B active) | Cross-attention | MoE efficiency, thinking mode, dynamic video frame sampling |
| **InternVL** (Shanghai AI Lab) | Dec 2023 | InternViT-6B | Vicuna-7B/13B | Q-Former | Massive 6B vision encoder, approaches GPT-4V |
| **InternVL2** (Shanghai AI Lab) | Jul 2024 | InternViT-6B | InternLM2-7B/20B/76B | MLP + dynamic high-res | Near GPT-4V performance, diverse model sizes |
| **InternVL2.5** (Shanghai AI Lab) | Dec 2024 | InternViT-6B | InternLM3-78B | MLP + dynamic high-res | 10x data efficiency, near GPT-4o performance |
| **InternVL3** (Shanghai AI Lab) | mid-2025 | InternViT-6B | InternLM3 | Advanced dynamic res | Matching GPT-4o on most benchmarks |
| **CogVLM** (Tsinghua) | Feb 2024 | EVA-CLIP ViT | Vicuna-7B | Deep visual expert per layer | 38.8B trainable visual experts, strong on detailed visual reasoning |
| **GPT-4V** (OpenAI) | Sep 2023 | Proprietary | GPT-4 | Proprietary | First frontier VLM, strong across all tasks, no architectural disclosure |
| **GPT-4o** (OpenAI) | May 2024 | Proprietary | GPT-4o (unified) | End-to-end multimodal | Unified vision+text+audio, low latency, SOTA on release |
| **Gemini Pro Vision** (Google) | Dec 2023 | Proprietary | Gemini Pro | Natively multimodal | Trained from scratch on multimodal data, strong reasoning |
| **Gemini 2.5 Pro** (Google) | Mar 2025 | Proprietary | Gemini 2.5 | Natively multimodal | 1M+ token context, native video understanding, deep reasoning |
| **Claude 3 Opus** (Anthropic) | Mar 2024 | Proprietary | Claude 3 | Proprietary | Strong at document analysis, data visualization, scientific diagrams |

### 5.3 Comparative Performance Overview

Approximate benchmark ranges (open-source / open-weight models, 2025–2026 data):

| Benchmark | CLIP | BLIP-2 | LLaVA 1.6 | Qwen2.5-VL | InternVL2.5 | GPT-4o | Gemini 2.5 Pro |
|-----------|------|--------|-----------|------------|-------------|--------|----------------|
| **MMBench** | — | 65–70 | 73–78 | 80–85 | 82–87 | 83–88 | 84–88 |
| **MMMU** (val) | — | 42–46 | 48–54 | 58–64 | 60–66 | 62–69 | 65–72 |
| **MathVista** | — | 38–42 | 48–52 | 58–65 | 60–66 | 63–70 | 64–71 |
| **DocVQA** | — | 55–60 | 72–78 | 88–92 | 90–94 | 92–96 | 91–95 |
| **ChartQA** | — | 52–58 | 68–74 | 80–86 | 82–88 | 85–90 | 84–89 |
| **SEED-Bench** | 55–60 | 60–65 | 68–72 | 72–78 | 74–80 | 75–81 | 76–82 |

*Values approximate; exact numbers depend on specific model variant, resolution, and evaluation protocol. Proprietary models cited from published leaderboard figures.*

**Key takeaways**:
- Open-weight models have closed the gap from 10–20 points behind GPT-4V (2023) to within 3–8 points of GPT-4o (2025–2026).
- Document/OCR-heavy tasks show the largest improvement from dynamic high-resolution processing.
- Math and scientific reasoning remain the hardest benchmarks, with the largest gap between open and proprietary models.
- Vision encoder scaling (InternViT-6B) has been the single most impactful open-source innovation for closing the gap.

---

## 6. Benchmarks & Evaluation Metrics

### 6.1 Comprehensive Benchmarks

#### MMBench
- **Purpose**: Evaluate VLMs across 20 fine-grained ability dimensions (perception, reasoning, OCR, spatial understanding, knowledge, etc.).
- **Format**: Multiple-choice questions with images, bilingual (English + Chinese).
- **Subsets**: MMBench-Test (circa-perfect labeling), MMBench-Dev, CCBench (cultural context).
- **Metrics**: Accuracy per dimension + overall average.
- **Key insight**: MMBench's dimensional breakdown reveals specific VLM weaknesses — e.g., many VLMs struggle with "attribute comparison" and "counterfactual reasoning" while excelling at "object recognition" and "color perception."

#### MMMU (Massive Multi-discipline Multimodal Understanding)
- **Purpose**: College-level multimodal understanding across 30 subjects (art, engineering, medicine, social sciences, etc.).
- **Format**: 11.5K questions requiring domain expertise and deliberate reasoning. Questions often contain images, diagrams, or charts.
- **Metrics**: Accuracy per subject, overall accuracy.
- **Key insight**: MMMU is the hardest standard benchmark — even frontier models (GPT-4o, Gemini 2.5) score 65–72%, far below human expert performance (~90%). Subject-specific analysis (e.g., medicine vs. art history) reveals domain biases.

#### MathVista
- **Purpose**: Mathematical reasoning in visual contexts — charts, geometry diagrams, plots, word problems.
- **Format**: Multiple-choice + free-form numerical answers across 28 task categories.
- **Metrics**: Accuracy (exact match for numerical answers), fine-grained per-task breakdown.
- **Key insight**: MathVista is especially discriminating for multimodal reasoning. Many VLMs that perform well on MMBench drop 15–20 points here, indicating that visual math reasoning remains a core weakness.

#### SEED-Bench
- **Purpose**: Generative comprehension evaluation across 12 dimensions (scene understanding, temporal comprehension, instance attributes, etc.).
- **Format**: Human-annotated multiple-choice questions covering image and video understanding.
- **Key insight**: SEED-Bench is the primary benchmark for video VLM evaluation, with substantial gap between models that process video naively (frame averaging) and those with temporal modeling.

### 6.2 Specialized Benchmarks

| Benchmark | Domain | Description |
|-----------|--------|-------------|
| **VQAv2** | General VQA | 265K questions on COCO images, the classic benchmark |
| **TextVQA** | OCR + VQA | Questions requiring reading text in images |
| **DocVQA** | Document VQA | Visual QA on scanned/PDF documents |
| **ChartQA** | Chart comprehension | QA on bar/line/pie charts |
| **POPE** | Hallucination | Probing Object Hallucination — binary "is there an X?" queries |
| **MM-Vet** | Integrated capabilites | Complex tasks combining OCR, spatial reasoning, knowledge |
| **OCRBench** | OCR quality | Text recognition in natural and document scenes |
| **GQA** | Spatial reasoning | Compositional QA requiring relational visual reasoning |
| **VSR** | Visual spatial reasoning | Binary classification of spatial relations |
| **CMMMU** | Multilingual | Chinese MMMU for multilingual evaluation |
| **NaturalBench** | Vision-centric | Simple but challenging visual questions (NeurIPS'24) |
| **MMMU-Pro** | Hard MMMU | Harder subset for differentiating frontier models |

### 6.3 Evaluation Metrics

| Metric | Description | Used In |
|--------|-------------|---------|
| **Acc.** | Exact match or correct answer rate | MMBench, MMMU, MathVista |
| **PPL** | Cross-entropy loss on generated text | Captioning, generative eval |
| **CIDEr** | n-gram consensus-based caption quality | COCO Captioning |
| **BLEU/ROUGE-L/METEOR** | n-gram precision / recall / synonym matching | Captioning, summarization |
| **F1 Score** | Precision-recall harmonic mean | VQA, classification |
| **CHAIR** | Caption hallucination rate | Captioning, instruction eval |
| **POPE Accuracy** | Precision/recall/F1 on object existence queries | Hallucination detection |
| **Human Evaluation** | Expert rating of response quality | Instruction following |

### 6.4 Evaluation Frameworks

- **VLMEvalKit** (OpenCompass): 220+ models, 80+ benchmarks — the standard open-source VLM eval toolkit.
- **LMMs-Eval**: Unified harness inspired by lm-eval-harness.
- **EvalScope** (ModelScope): Standard benchmarks with multi-backend flexibility.

### 6.5 Common Failure Modes

1. **Object Hallucination**: Describing objects not in the image.
2. **Spatial Misunderstanding**: Misjudging relative positions.
3. **Counting Errors**: Hard with >5 objects or occlusion.
4. **Attribute Confusion**: Misattributing color/texture between objects.

---

## 7. Key Applications

### 7.1 Document Understanding

VLMs have revolutionized document processing by treating documents as images end-to-end, replacing separate OCR → NLP → extraction pipelines. Capabilities include form understanding, invoice extraction, receipt parsing, contract analysis, and scientific paper reading. Top performers: Qwen2.5-VL (88–92% on DocVQA), InternVL2.5 (90–94%), GPT-4o (92–96%). Key challenge: document-quality variation (handwriting, poor scans, dense tables).

### 7.2 Chart & Data Visualization Analysis

- **Capabilities**: Extract data from bar/line/pie charts, detect trends, answer data questions, convert charts to tabular data. Top performers: GPT-4o, Gemini 2.5 Pro, Qwen2.5-VL (80–86% on ChartQA). Emerging: multi-chart reasoning across multiple visualizations.

### 7.3 Medical Imaging, Robotics & Video Understanding

**Medical**: Radiology report generation (X-ray/CT/MRI), pathology analysis, dermatological diagnosis. Specialized models include Med-PaLM M, LLaVA-Med, RadFM, BioMedCLIP. Used as clinical decision support (not autonomous diagnosis) due to privacy and accuracy requirements.

**Robotics**: Visual navigation, zero-shot object grasping, task planning from demonstration videos. Key models: RT-2 (VLM→robot actions), VoxPoser, SayCan, Octo. Major challenge: VLMs lack physical world understanding (weight, friction, material properties), leading to physically implausible plans.

**Video Understanding**: Video captioning, temporal grounding, activity recognition, video QA. Architectures use frame-level sampling + temporal aggregation (VideoLLaMA, VideoChat) or 3D spatiotemporal encoders. Top performers: Gemini 2.5 Pro (native video), Qwen2.5-VL (dynamic frame sampling), InternVideo2. Key challenge: processing hour-long video at native resolution remains computationally prohibitive.

### 7.6 GUI Agents & Automation

- **Capabilities**: Screen understanding, UI element identification, automated web navigation, form filling, mobile app control.
- **Key models**: CogAgent (Tsinghua, based on CogVLM), Qwen2.5-VL (GUI agent mode), ScreenAI (Google).
- **Format**: Take screenshot → identify interactable elements by bounding box → plan sequence of actions → execute via accessibility API or coordinate-based click.
- **Current state**: Simple multi-step tasks (booking flights, filling forms) work well. Complex reasoning chains and cross-application workflows remain challenging.

### 7.7 Other Applications

| Application | Description | Key Models |
|-------------|-------------|------------|
| **Visual Search & Retrieval** | Search by image content, not metadata | CLIP, SigLIP (as embedding models) |
| **Image Editing & Generation** | Instruct image editing via natural language | InstructPix2Pix (CLIP + diffusion) |
| **Content Moderation** | Analyze images + text for policy violations | GPT-4V, proprietary classifiers |
| **E-commerce** | Product tagging, visual search, automated descriptions | CLIP, custom VLMs |
| **Education** | Diagram explanation, math problem solving, science tutoring | GPT-4o, Khanmigo |
| **Accessibility** | Scene description, reading text for visually impaired | GPT-4o, Claude Vision |

---

## 8. Efficiency: Compression & Quantization

### 8.1 Why VLM Efficiency Matters

A typical 7B VLM + 300M vision encoder requires ~16GB GPU memory in FP16; production models use 70B+ LLMs. Quantization and compression reduce costs, improve latency for real-time use (robotics, GUI agents), enable edge deployment (<4GB), and reduce energy consumption.

### 8.2 Quantization Strategies

| Technique | Precision | Compression Ratio | Quality Impact | Notes |
|-----------|-----------|-------------------|---------------|-------|
| **FP16** | 16-bit | 2× vs FP32 | Negligible | Standard inference format |
| **INT8** | 8-bit | 4× vs FP32 | Minimal (≤1%) | Widespread support (GPTQ, bitsandbytes) |
| **INT4** | 4-bit | 8× vs FP32 | Small (1–3%) | AWQ, GPTQ, GGUF formats |
| **NF4** | NormalFloat4 | 8× | Small | QLoRa-style, normal distribution optimized |
| **INT3/INT2** | 3-bit / 2-bit | 10–16× | Moderate–Heavy | Research stage, significant quality degradation |
| **Sparse Quantization** | Mixed | Variable | Task-dependent | Retains important weights at higher precision |

### 8.3 VLM-Specific Quantization Challenges

Unlike text-only LLMs, VLMs present unique quantization challenges: vision encoders are generally more quantization-tolerant than LLM layers; connector layers (Q-Former, cross-attention) are often the most quantization-sensitive components; dynamic high-resolution processing makes memory management variable; and cross-modal attention noise can amplify hallucination.

**Empirical findings**: Quantizing only the vision encoder to INT8 preserves 98–99% of benchmark performance. Full-model INT4 causes 3–8% degradation on MMBench and 5–10% on MathVista. Cross-attention connectors (Q-Former) are more quantization-robust than MLP projectors.

### 8.4 Model Compression Techniques

| Technique | Description | Effectiveness |
|-----------|-------------|---------------|
| **Pruning** | Remove redundant weights (magnitude-based, SparseGPT) | 20–40% sparsity with minimal loss |
| **Distillation** | Train a smaller student VLM from a larger teacher VLM | Significant compression (3×–10×) with task-dependent loss |
| **Low-Rank Adaptation (LoRA)** | Fine-tune with low-rank weight updates; can be merged for full-rank inference | Not compression per se, but enables efficient fine-tuning |
| **Architecture-Level Efficiency** | Smaller vision encoders, efficient attention variants | Depends on design |
| **Token Reduction** | Reduce the number of visual tokens (average pooling, Perceiver, Q-Former) | Direct latency improvement |
| **Dynamic Resolution** | Process images at variable compute depending on content complexity | Adaptive efficiency |

### 8.5 Efficient VLM Architectures

Several model families specifically prioritize efficiency:

- **SmolVLM (Hugging Face, 2025)**: 2B total parameters, optimized for edge deployment with strong performance on standard benchmarks.
- **Moondream**: 0.5B–9B variants, with a 0.5B model designed for constrained edge hardware.
- **Phi-3-Vision (Microsoft, 2024)**: 4.2B parameters, strong accuracy:efficiency ratio, suitable for on-device deployment.
- **PaliGemma (Google, 2024)**: 3B parameter VLM based on SigLIP + Gemma, optimized for fine-tuning efficiency.
- **Qwen-VL 2B**: Compact model achieving reasonable performance for its size class.

### 8.6 Practical Deployment Guidelines

| Use Case | Recommended Quantization | Hardware |
|----------|------------------------|----------|
| **Cloud API serving** | FP16 or INT8 (for throughput) | A100, H100, L40S |
| **On-premise server** | INT4 (AWQ/GPTQ) | A10, RTX 4090, L4 |
| **Laptop/Desktop** | INT4 (GGUF) | Apple Silicon, RTX 3060+ |
| **Mobile/Edge** | INT4 + token reduction + small backbone | Qualcomm, MediaTek, Apple Neural Engine |

---

## 9. Multilingual VLMs

### 9.1 The Challenge

Most VLM research has focused on English. Extending VLMs to other languages faces:
- **Data scarcity**: Image-text pairs in non-English languages are orders of magnitude less available than English.
- **Script diversity**: Arabic, Chinese, Devanagari, Cyrillic, and other scripts require compatible text encoders.
- **Cultural context**: Visual concepts vary across cultures (food, clothing, landmarks, symbols).

### 9.2 Approaches & Notable Models

| Approach | Method | Example |
|----------|--------|---------|
| **Bilingual Pretraining** | Train on EN + target language from scratch | Qwen-VL (EN+ZH) |
| **Cross-lingual Transfer** | Pretrain on EN, fine-tune on machine-translated data | LLaVA variants |
| **Multilingual Encoder** | mT5, XLM-RoBERTa, or NLLB as LLM backbone | Various research models |
| **Culture-aware Curation** | Region-specific image-text data | MMBench-CC, CMMMU |

| Model | Languages | Performance |
|-------|-----------|-------------|
| **Qwen-VL / Qwen2.5-VL** | EN + Chinese | Strong on both CMMMU and MMBench |
| **InternVL** | EN + Chinese | Comparable to Qwen-VL on Chinese |
| **PaliGemma** | 100+ languages | Good via multilingual PaLI decoder |
| **GPT-4o / Gemini** | ~50 languages | Strong on high-resource languages |

**Key findings**: English-centric VLMs lose 10–25 points on non-English benchmarks. Bilingual models (EN+ZH) close this gap to 95%+ of English performance. Low-resource languages remain a major open challenge with few models supporting African, SE Asian, or indigenous languages.

---

## 10. Future Directions

### 10.1 Embodied Agents

VLMs are increasingly viewed as the "brain" for embodied agents — robots and virtual agents that perceive, reason, and act in physical environments.

- **Current capability**: VLMs can generate high-level action plans ("pick up the red mug, then place it on the blue table") but struggle with low-level motor control and physical dynamics.
- **Research directions**:
  - **RT-2-style direct action tokenization**: Treat robot actions as additional output tokens, enabling end-to-end VLM-to-control.
  - **Embodied pretraining**: Train VLMs on egocentric video data (e.g., Ego4D) to build physical world understanding.
  - **3D-aware VLMs**: Extend VLMs to process 3D scene representations (point clouds, neural radiance fields) rather than 2D images.
  - **Online adaptation**: VLMs that learn from physical interaction feedback in real-time.

### 10.2 Video-Native Models

Most current VLMs process video by sampling frames independently — missing temporal dynamics. The next generation will be **video-native**:

- **Spacetime encoders**: Video ViT that processes 3D spatiotemporal patches (VideoMAE, ViViT).
- **Native video tokenization**: End-to-end video processing without frame sampling (Gemini Omni direction).
- **Long-form video understanding**: Processing hour-long videos with temporal grounding and narrative comprehension.
- **Key challenge**: Computational cost of native spatiotemporal attention scales quadratically with both spatial resolution and temporal duration.

### 10.3 Merging with Speech / Audio

GPT-4o already demonstrated unified vision+text+audio processing. Audio-VLMs (AudioGPT, Qwen-Audio) jointly understand images, speech, and text. Key challenge: aligning modalities with different temporal resolutions (ms for audio, frame-level for video, token-level for text).

### 10.4 Efficiency Scaling

- **MoE**: Qwen3-VL (235B/22B active) shows sparse activation works for VLMs.
- **Infinite-resolution / learned token reduction**: Adaptive compute allocation per image region.
- **Hardware co-design**: ASICs optimized for dynamic VLM operations.

### 10.5 Safety, Hallucination & Alignment

Key areas: multimodal hallucination detection, Constitutional AI for vision, bias mitigation across cultures/demographics, adversarial robustness against visually-embedded prompt injection, and systematic VLM red teaming.

### 10.6 Long-Context Multimodal Reasoning

Gemini 2.5 Pro's 1M+ token context enables multi-image and long-video reasoning. Challenges: consistency across many visual inputs, attention scaling for long sequences.

### 10.7 Expanding Modalities

VLM paradigm expanding from image+text → text to include input from video, audio, 3D, tactile, and IR sensors; output to speech, images, video, robot actions, and code — the unified "Omni" agent paradigm.

### 10.8 Personalization & Few-Shot Adaptation

VLMs adapting to new visual concepts from few examples, learning user preferences, and expanding in-context learning for vision beyond Flamingo's original demonstration.

---

## 11. References & Further Reading

### Survey Papers
- "A Comprehensive Survey and Guide to Multimodal LLMs in Vision-Language Tasks" — arXiv:2411.06284
- "A Survey of State of the Art Large Vision Language Models" — arXiv:2501.02189
- "A Comprehensive Survey of Vision-Language Models" — Information Fusion, 2025
- "Vision Language Models: A Survey of 26K Papers" — ResearchGate, 2025

### Key Papers
- **CLIP**: Radford et al., 2021
- **SigLIP**: Zhai et al., 2023; SigLIP 2 — 2025
- **BLIP/BLIP-2**: Li et al., 2022; Li et al., 2023
- **Flamingo**: Alayrac et al., 2022
- **LLaVA / LLaVA 1.5**: Liu et al., 2023
- **InstructBLIP**: Dai et al., 2023
- **Qwen-VL**: Bai et al., 2023
- **InternVL**: Chen et al., 2023
- **CogVLM**: Wang et al., 2024
- **MMMU**: Yue et al., 2023
- **MMBench**: Liu et al., 2024

### Evaluation & Ecosystems
- **VLMEvalKit**: github.com/open-compass/VLMEvalKit
- **Open VLM Leaderboard**: opencompass-open-vlm-leaderboard.hf.space
- **Awesome VLM Architectures**: github.com/gokayfem/awesome-vlm-architectures

---

*This guide is maintained as part of the research repository at [github.com/jackliusr/research](https://github.com/jackliusr/research). Contributions, corrections, and updates are welcome.*
