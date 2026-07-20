# LLM Guard Models: A Comprehensive Guide to AI Safety and Content Moderation

**Author:** Jack Liu Shurui | **Role:** Solution Architect, Crédit Agricole CIB | **Series:** ML/AI Technology Guides | **Date:** July 2026

---

## 1. What Are Guard Models?

Guard models are specialized LLMs fine-tuned to classify content safety. Unlike general-purpose LLMs that generate text, guard models act as **discriminative classifiers** that determine whether content violates a defined safety policy. They sit at interception points in the LLM serving pipeline:

```
User Input → [Guard Model] → LLM → [Guard Model] → User Output
```

Guard models are one component of a broader AI safety stack:

| Layer | Examples | Function |
|-------|----------|----------|
| **Guard Models** | Llama Guard 3, ShieldGemma | ML-based content safety classification |
| **Prompt Guards** | Jailbreak detectors | Detect adversarial input patterns |
| **Content Filters** | Keyword/Regex blocklists | Rule-based content blocking |
| **Moderation APIs** | OpenAI Moderation, Perspective | Cloud-hosted safety classification |
| **Guardrails** | NeMo Guardrails, Guardrails AI | Logic-based policy enforcement |

Guard models are distinct because they are themselves LLMs — they understand context, nuance, and intent rather than relying on static rules alone. This makes them significantly more effective at catching sophisticated jailbreaks, implicit harmful content, and context-dependent violations. Guard model categories include: **input guards** (classify prompts before LLM), **output guards** (classify LLM responses), **dual guards** (both sides), **multimodal guards** (text + image), and **agentic guards** (tool calls, plans, intermediate reasoning).

---

## 2. Why Guard Models Matter

### 2.1 Preventing Harmful Content

Guard models prevent LLMs from generating hate speech, illegal instructions, sexually explicit material, self-harm encouragement, violence, defamatory statements, and privacy violations — content that would otherwise pass through unfiltered.

### 2.2 Regulatory Compliance

| Regulation | Requirements | Impact on Guard Models |
|------------|-------------|----------------------|
| **EU AI Act** (2024–2027) | High-risk AI systems need safety filters; systemic-risk GPAIs require evaluations; penalties up to 7% of global revenue or €35M | Mandates documented safety controls; drives adoption of guard models with audit trails |
| **Singapore AI Verify** | Testing framework for transparency and safety; companion AI Verify Foundation open-sources toolkits | Guard model testing required for AI Verify certification |
| **MAS FEAT** | Fairness, Ethics, Accountability, Transparency principles for financial AI | Financial institutions must demonstrate AI safety controls for MAS review |
| **China GenAI Regulations** | Mandatory content safety reviews, banned content lists | Guard models with Chinese language support required |
| **Canada AIDA** | Safety requirements for high-impact AI systems | Guard models as compliance evidence |

### 2.3 Enterprise Risk Management

| Risk | Guard Model Mitigation |
|------|----------------------|
| Brand damage from offensive outputs | Output guard catches harmful responses before publication |
| Data leakage via prompt injection | Input guard detects jailbreak attempts |
| Regulatory fines | Documented safety controls demonstrate compliance |
| IP infringement | Guard models with IP/copyright categories |
| Customer harm | Multi-layer safety reduces probability of harmful output |

### 2.4 Jailbreak and Prompt Injection Protection

Guard models detect: direct injection ("Ignore all previous instructions..."), role-play jailbreaks ("You are DAN..."), payload splitting, context manipulation, multi-language attacks, and encoding-based attacks (Base64, ROT13). They generalize to novel attack variants that rule-based filters miss.

### 2.5 Multi-Language Support

| Tool | Language Coverage |
|------|------------------|
| OpenAI Moderation | Broad multi-language (dozens of languages) |
| Llama Guard 3 | 8 languages (EN, FR, DE, HI, IT, PT, ES, TH) |
| ShieldGemma | Primarily English |
| Perspective API | 20+ languages (toxicity only) |
| Azure Content Safety | Multi-language with varying quality |

---

## 3. Llama Guard 3 (Meta)

### 3.1 Overview

Meta's content safety model, part of the Llama 3.1 family. Fine-tuned for safety classification — not generation. Based on Llama 3.1-8B, aligned to the **MLCommons standardized hazards taxonomy**, optimized for on-device deployment.

### 3.2 Variants

| Variant | Parameters | Modality | Use Case |
|---------|-----------|----------|----------|
| Llama Guard 3-8B | 8B | Text-only | Standard, highest accuracy |
| Llama Guard 3-1B | 1B | Text-only | Lightweight, edge/mobile/on-device |
| Llama Guard 3-Vision | 11B | Text + Image | Multi-modal (Llama 3.2 11B vision backbone) |
| Llama Guard 3-1B-INT4 | 1B quantized | Text-only | Extremely low-resource environments |

### 3.3 The 13 Safety Risk Categories

| ID | Category | Description |
|----|----------|-------------|
| S1 | Violent Crimes | Murder, assault, terrorism, kidnapping |
| S2 | Non-Violent Crimes | Fraud, theft, drug trafficking |
| S3 | Sex-Related Crimes | Human trafficking, sexual assault |
| S4 | Child Sexual Exploitation | CSAM-related content |
| S5 | Defamation | Content defaming individuals/organizations |
| S6 | Specialized Advice | Untrusted medical, legal, financial advice |
| S7 | Privacy | PII exposure, privacy violations |
| S8 | Intellectual Property | Copyright/trademark infringement |
| S9 | Indiscriminate Weapons | WMD instructions and content |
| S10 | Hate | Hate speech targeting protected characteristics |
| S11 | Self-Harm | Suicide, eating disorders, self-injury |
| S12 | Sexual Content | Sexually explicit content (not S3/S4) |
| S13 | Elections | Electoral interference, political misinformation |

### 3.4 Token Format and Output

Uses special delimiters to distinguish prompt from response:

```
Input classification: [[BEGIN USER INPUT]] <text> [[END USER INPUT]]
Output classification: [[BEGIN LLM OUTPUT]] <text> [[END LLM OUTPUT]]
Combined: [[BEGIN USER INPUT]] <prompt> [[END USER INPUT]] [[BEGIN LLM OUTPUT]] <response> [[END LLM OUTPUT]]
```

**Output:** `safe` or `unsafe\nS<category_number>` (e.g., `unsafe\nS10` for hate speech).

### 3.5 Supported Languages

English, French, German, Hindi, Italian, Portuguese, Spanish, Thai (8 languages).

### 3.6 Strengths and Weaknesses

**Strengths:** Open-source (Apache 2.0), self-hostable and fine-tunable, comprehensive 13-category taxonomy, multiple model sizes, vision variant available, on-device optimization.

**Weaknesses:** 1B variant has reduced accuracy on edge cases, vision variant (11B) higher latency, requires GPU for 8B in production, superseded by newer models on some benchmarks (Qwen3Guard, Llama Guard 4).

### 3.7 Llama Guard 4 (Evolution)

Released in 2025–2026: 12B parameters, 14 hazard categories (added S14: Code Interpreter Abuse), multi-image support, improved adversarial robustness.

---

## 4. ShieldGemma (Google)

### 4.1 Overview

Google's safety classifier suite built on Gemma 2. Uses a **fine-tuned classifier** approach (classification head + confidence scores) rather than raw LLM prompting. Designed for policy-based customization — organizations define their own thresholds.

### 4.2 Variants

| Variant | Parameters | Modality | Notes |
|---------|-----------|----------|-------|
| ShieldGemma 2B | 2B | Text | Fastest, lowest latency |
| ShieldGemma 9B | 9B | Text | Balanced accuracy and speed |
| ShieldGemma 27B | 27B | Text | Highest accuracy, higher latency |
| ShieldGemma 2 (4B) | 4B | Image | Built on Gemma 3, synthetic + natural images |

### 4.3 Safety Categories

**ShieldGemma 1 (text):** Hate Speech, Sexually Explicit, Dangerous Content, Harassment (4 categories).

**ShieldGemma 2 (image):** Sexually Explicit, Violence & Gore, Dangerous Content (for both synthetic and natural images).

### 4.4 Policy Customization

Key differentiator: organizations configure **confidence thresholds per category** and define **custom safety policies** as text instructions or structured configurations. Supports **multi-policy** deployments (different thresholds for different user segments).

```python
policies = {
    "education": {"sexual": 0.95, "hate": 0.7, "violence": 0.8},
    "support_forum": {"self_harm": 0.5, "hate": 0.7},
    "general": {"hate": 0.7, "sexual": 0.8, "violence": 0.75},
}
```

### 4.5 Integrations

Vertex AI Model Registry, Gemini API safety settings, Cloud Armor, AutoML fine-tuning pipelines.

### 4.6 Strengths and Weaknesses

**Strengths:** High accuracy benchmarks, multiple model sizes, policy customization, calibrated confidence scores, Google Cloud integration, fine-tuning support.

**Weaknesses:** Primarily English-only, Google ecosystem alignment (less portable), fewer categories (4) than Llama Guard 3 (13), threshold tuning requires operational expertise.

---

## 5. Guardrails vs Guard Models

### 5.1 Conceptual Distinction

| Dimension | Guard Models | Guardrails |
|-----------|-------------|------------|
| **Core Technology** | ML-based classifiers (fine-tuned LLMs) | Rule-based/logic-based systems |
| **How They Work** | Neural network classifies content by learned patterns | Predefined rules, DSL scripts, validation logic |
| **What They Check** | Content safety, harmfulness, toxicity | Structural rules, format, data validation |
| **Output** | Safe/Unsafe + category with confidence | Pass/Fail, transformed output |
| **Context Understanding** | High — understands nuance | Low — operates on explicit patterns |
| **Adaptability** | Fine-tuning or retraining | Declarative rule changes |
| **Latency** | Variable (2B–27B) | Typically <10ms per check |
| **Examples** | Llama Guard 3, ShieldGemma | NeMo Guardrails, Guardrails AI |

### 5.2 Typical Architecture: Both Together

```
User Input → [Guardrails: structural rules, format, PII] → [Guard Model: content safety]
    → LLM → [Guard Model: output safety] → [Guardrails: output validation] → User
```

### 5.3 Comparison: NeMo Guardrails vs Guardrails AI

| Feature | Guardrails AI | NeMo Guardrails |
|---------|--------------|-----------------|
| **Language** | Python-based RAIL spec (XML) | Colang DSL |
| **Strengths** | Output validation, structured data | Dialogue management, conversational flow |
| **Pipeline Stages** | Input/Output validation | 5 stages: input, dialog, retrieval, execution, output |
| **Best For** | Validating structured LLM outputs (JSON, code) | Conversational safety, multi-turn policies |

### 5.4 Rule-Based vs ML-Based Safety

| Criteria | Rule-Based | ML-Based (Guard Models) |
|----------|-----------|------------------------|
| Novel attack detection | Poor | Good |
| False positives | Low | Higher |
| Context understanding | None | Strong |
| Maintenance | Manual rule updates | Retraining/fine-tuning |
| Interpretability | High | Low |
| Cost | Low | Higher (GPU) |
| Best for | Known-bad patterns, formatting | Nuanced content safety, novel threats |

---

## 6. Other Guard Models and Safety Classifiers

### 6.1 OpenAI Moderation API

Free cloud-based content safety endpoint. **Categories:** Hate (incl. threatening), Harassment, Self-Harm (incl. intent/instruction), Sexual (incl. minors), Violence (incl. graphic), Illicit. **Strengths:** Always free, multi-language, well-maintained, low latency (~300ms). **Weaknesses:** API-only (no self-hosting), no customization, data leaves your infrastructure.

### 6.2 Perspective API (Jigsaw/Google)

Toxicity detection for publishers. **Attributes:** Toxicity, severe toxicity, identity attack, insult, profanity, threat, sexually explicit. **Scale:** 500M+ requests daily from 1,000+ partners (NYT, WSJ, Reddit). **Languages:** 20+ (Charformer architecture). **Strengths:** Proven at scale, free tier. **Weaknesses:** Toxicity only, API-only, evadable.

### 6.3 Azure AI Content Safety

**Categories (severity 0–7):** Hate, Sexual, Violence, Self-Harm. **Features:** Text + image moderation, jailbreak detection (prompt shields), groundedness detection, custom severity thresholds, container deployment option (on-prem). Integrates with Azure OpenAI Service.

### 6.4 AWS Bedrock Guardrails

**Policies:** Content filters (configurable thresholds), denied topics, word filters, PII redaction, contextual grounding, automated reasoning. Works with all Bedrock models (Claude, Llama, Mistral). Image content filters available from 2025.

### 6.5 Amazon Comprehend Toxicity

Labels: toxic_content, profanity, hate_speech, harassment, violence, sexual. Simple API, part of broader Comprehend suite. Pay-per-use.

### 6.6 Hugging Face Safety Models

| Model | Size | Language | Focus |
|-------|------|----------|-------|
| toxic-bert (Unitary) | 110M | English | Toxicity, multi-head |
| Detoxify | 110M | EN + Multilingual | 6 toxicity attributes |
| Hate-speech models | Varies | Multiple | Specialized classifiers |

**Advantages:** Fully self-hosted, free, CPU-runnable, fine-tunable. **Disadvantages:** Lower accuracy than dedicated guard models, limited context (BERT vs LLM), narrower coverage.

### 6.7 Emerging Models (2025–2026)

| Model | Developer | Size | Highlight |
|-------|-----------|------|-----------|
| Qwen3Guard | Alibaba | 0.5B–8B | 6-model family, beats Llama Guard 3 by 10+ F1, Apache 2.0 |
| Granite Guardian | IBM | 2B–13B | Enterprise-focused, domain-specific |
| WildGuard | Community | 7B | Open safety classifier |
| Nemotron Safety | NVIDIA | 4B | NeMo ecosystem integration |
| Llama Guard 4 | Meta | 12B | 14 categories, multi-image |

---

## 7. Architectural Patterns

### 7.1 Input Filter

```
User → [Guard Model] → Safe? → Yes → [LLM] → User
                         No → Block
```

**Pros:** Prevents harmful inputs reaching LLM, saves inference cost. **Cons:** Misses LLM-generated harmful content, false positives block legitimate use.

### 7.2 Output Filter

```
User → [LLM] → [Guard Model] → Safe? → Yes → User
                                  No → Block/Fallback
```

**Pros:** Catches model-generated harmful content, can regenerate on false positive. **Cons:** Harmful input still reaches LLM (wasted compute), no injection protection.

### 7.3 Dual Guard (Input + Output)

Maximum security: guard models on both sides. Comprehensive coverage, detailed audit logs, separately tunable policies. **Cons:** Highest latency (2 calls), higher cost, dual false-positive risk.

### 7.4 Fallback Pattern

When guard blocks, redirect to: static safe response ("I can't answer that"), pre-approved response library, human escalation, or response regeneration (output guard only).

### 7.5 Multi-Layer Safety

```
Layer 1: Network/Infrastructure (IP blocking, WAF)
Layer 2: Rate Limiting & Authentication (API keys, OAuth)
Layer 3: Guardrails (format validation, PII, topic denial)
Layer 4: Guard Models (input + output content safety)
Layer 5: Monitoring & Human Review (audit, sampling, red-teaming)
```

No single mechanism is foolproof — each layer catches what others miss, and regulatory auditors expect defense-in-depth.

### 7.6 Agentic Guard Pattern

For AI agents, guard models must inspect **tool call arguments** (SQL injection via params), **agent plans** (entire multi-step workflow), **intermediate reasoning** (chain-of-thought safety), and **data exfiltration** (unauthorized data transfer). Each tool invocation, plan step, and reasoning trace is classified before execution continues.

---

## 8. Integration Patterns

### 8.1 API Gateway Integration

Guard model as middleware in API Gateway (AWS Lambda@Edge, Azure API Management, GCP Apigee, Kong/Envoy). **Pros:** Centralized enforcement, no app code changes, consistent policy. **Cons:** Single point of failure, adds latency, needs scaling to match throughput.

### 8.2 Proxy/Sidecar Pattern

Guard model runs as sidecar in the same pod (K8s sidecar, Docker Compose, Envoy ext_authz). **Pros:** Low latency (localhost), independent scaling, language-agnostic. **Cons:** Resource competition with LLM (GPU memory), complex orchestration.

### 8.3 SDK Integration

Guard model called directly from application code via Hugging Face transformers, vLLM, or Llama.cpp. **Pros:** Full control, caching, fine-grained error handling. **Cons:** Code complexity, tight coupling, language-dependent.

### 8.4 Streaming Guard

For streaming responses, accumulate content to sentence boundaries, then classify. **Strategies:** Sentence-bounded, time-bounded, token-bounded, or hybrid flushing. **Challenge:** partial content may be flagged unsafe when full context would be safe; retroactive revocation of streamed content is complex.

---

## 9. Customizing Guard Models

### 9.1 Fine-Tuning Llama Guard 3

**Process:** (1) Curate domain dataset (banking fraud, healthcare privacy, legal privilege). (2) Map/customize categories (e.g., banking: financial misinformation, insider trading). (3) Train with LoRA or full fine-tuning. (4) Evaluate on held-out domain data. (5) A/B test against baseline.

**Best Practices:** Include adversarial examples, balance positive/negative, include borderline cases, add production false positives, refresh data regularly.

### 9.2 Policy Customization in ShieldGemma

Configure confidence thresholds per category without full fine-tuning. Support for **multi-policy** (different thresholds per context: education vs. general vs. support forum).

### 9.3 Creating Custom Categories

Define clear category with inclusion/exclusion criteria, collect 500–1000+ examples, augment training dataset, fine-tune, validate on edge cases, monitor in production.

**Example custom categories for banking:** Financial misinformation, insider trading references, unauthorized transactions, regulatory violation content.

### 9.4 Multi-Language Adaptation

Options: (1) Translate to English then classify. (2) Fine-tune Llama Guard 3 on non-English data (already supports 8 languages). (3) Language-specific ensemble per region. **Considerations:** Accuracy varies significantly by language; false positive rates higher for lower-resource languages; cultural context affects harm definitions; data privacy laws may restrict cross-border classification.

---

## 10. Deployment Considerations

### 10.1 Model Size vs Latency

| Model | Params | GPU Latency | CPU Latency | Memory |
|-------|--------|-------------|-------------|--------|
| Llama Guard 3-1B | 1B | 15–30ms | 200–500ms | ~2GB |
| Llama Guard 3-8B | 8B | 50–150ms | 2–5s | ~16GB |
| Llama Guard 3-Vision | 11B | 100–300ms | N/A | ~22GB |
| ShieldGemma 2B | 2B | 20–40ms | 300–800ms | ~4GB |
| ShieldGemma 9B | 9B | 60–180ms | 3–8s | ~18GB |
| ShieldGemma 27B | 27B | 150–500ms | N/A | ~54GB |
| Qwen3Guard-0.5B | 0.5B | 5–15ms | 50–150ms | ~1GB |

### 10.2 GPU vs CPU

| Factor | GPU | CPU |
|--------|-----|-----|
| Latency | 15–500ms | 50ms–8s |
| Throughput | 10–1000 req/s | 1–50 req/s |
| Cost/hr | $0.50–$2.00 | $0.05–$0.30 |
| Max model | All models | <3B quantized |

INT8 quantization: ~2x speedup, <0.5% accuracy loss. INT4: ~3–4x speedup, 1–3% loss.

### 10.3 Batch vs Real-Time

| | Real-Time | Batch |
|--|-----------|-------|
| Latency target | <500ms per call | Hours for millions |
| Use case | Online LLM inference | Offline content review |
| Hardware | GPU (low batch) | GPU (high batch) or CPU |

### 10.4 Caching Strategies

Cache classification results for identical or near-identical inputs using embedding similarity (cosine distance). LRU/TTL-based expiration. Pre-compute common patterns.

### 10.5 Cost Per 100K Classifications (USD)

| Model | GPU/Infra Cost | API Cost | Total |
|-------|---------------|----------|-------|
| Llama Guard 3-1B (self-hosted) | $1–$3 | $0 | $1–$3 |
| Llama Guard 3-8B (self-hosted) | $5–$15 | $0 | $5–$15 |
| ShieldGemma 27B (self-hosted) | $15–$45 | $0 | $15–$45 |
| OpenAI Moderation | $0 | $0* | $0 |
| Azure Content Safety | $0 | ~$1 | ~$1 |
| AWS Bedrock Guardrails | $0 | ~$2 | ~$2 |

*OpenAI Moderation API remains free as of 2026.

### 10.6 Model Versioning

Semantic versioning + A/B testing + shadow deployment (log only, no blocking) + canary rollout (5–10% traffic) + rollback plan. Monitor: false positive/negative rate change, latency regression, category-level accuracy, multi-language performance.

---

## 11. Benchmarks and Evaluation

### 11.1 Standard Safety Benchmarks

| Benchmark | Description | Categories | Languages |
|-----------|-------------|-----------|-----------|
| MLCommons Safety | Standardized safety eval | 13 hazards | English |
| HarmBench | Red-teaming eval | 7 categories | English |
| SafetyBench | Chinese + English | 6 categories | Chinese, English |
| SALAD-Bench | Jailbreak evaluation | Comprehensive | English |

### 11.2 Approximate Accuracy (F1 on MLCommons)

| Model | Overall | Hate | Violence | Sexual | Self-Harm |
|-------|---------|------|----------|--------|-----------|
| Llama Guard 3-8B | 0.89–0.94 | 0.91 | 0.93 | 0.90 | 0.92 |
| Llama Guard 3-1B | 0.82–0.88 | 0.84 | 0.87 | 0.83 | 0.85 |
| ShieldGemma 27B | 0.91–0.95 | 0.93 | 0.94 | 0.92 | 0.93 |
| ShieldGemma 9B | 0.88–0.92 | 0.90 | 0.91 | 0.89 | 0.91 |
| ShieldGemma 2B | 0.83–0.89 | 0.85 | 0.88 | 0.84 | 0.86 |
| OpenAI Moderation | 0.87–0.93 | 0.89 | 0.91 | 0.88 | 0.90 |
| Qwen3Guard-8B | 0.92–0.96 | 0.94 | 0.95 | 0.93 | 0.94 |

### 11.3 False Positive Rate Impact

| FPR | Impact |
|-----|--------|
| <1% | Excellent — users rarely see false blocks |
| 1–3% | Good — minor annoyance |
| 3–5% | Noticeable — tuning needed |
| 5–10% | Problematic — erodes trust |
| >10% | Unacceptable — unusable |

**Reduction strategies:** Lower thresholds (trade: more false negatives), output-only guard (regenerate on false positive), fallback responses instead of hard blocks, user appeals, context-aware strictness.

### 11.4 Multi-Language Accuracy (Approximate F1)

| Language | Llama Guard 3-8B | OpenAI Moderation | Perspective |
|----------|-----------------|-------------------|-------------|
| English | 0.91–0.94 | 0.87–0.93 | 0.82–0.88 |
| French | 0.85–0.90 | 0.82–0.88 | 0.78–0.84 |
| German | 0.84–0.89 | 0.81–0.87 | 0.77–0.83 |
| Hindi | 0.78–0.85 | 0.75–0.82 | 0.65–0.75 |
| Spanish | 0.84–0.89 | 0.82–0.88 | 0.78–0.84 |
| Thai | 0.72–0.80 | 0.70–0.78 | 0.60–0.70 |
| Chinese | Not supported | 0.78–0.85 | 0.70–0.78 |

---

## 12. Tool Selection Guide

### By Deployment Model

| Deployment | Recommended | Rationale |
|------------|-------------|-----------|
| On-Prem / Self-Hosted | Llama Guard 3, ShieldGemma, Qwen3Guard | Data sovereignty, no API dependency |
| Cloud API | OpenAI Moderation, Azure, Perspective | Zero infra, managed, always updated |
| Edge / Mobile | Llama Guard 3-1B, Qwen3Guard-0.5B | Small size, offline |
| Air-Gapped | Llama Guard 3, Hugging Face models | No external network required |

### By Language Coverage

| Need | Recommended |
|------|-------------|
| Best multi-language | OpenAI Moderation API |
| 8 languages (self-hosted) | Llama Guard 3 |
| English-only, highest accuracy | ShieldGemma 27B |
| English-only, lowest latency | ShieldGemma 2B, Llama Guard 3-1B |
| Chinese safety | Qwen3Guard, OpenAI Moderation |

### By Customization Need

| Need | Recommended | Method |
|------|-------------|--------|
| Full fine-tuning | Llama Guard 3, Qwen3Guard | LoRA/full fine-tuning |
| Threshold config | ShieldGemma, Azure Content Safety | Per-category thresholds |
| Policy/topic rules | NeMo Guardrails, AWS Bedrock Guardrails | DSL/topic lists |
| Structured output validation | Guardrails AI | RAIL spec |
| API-only, no customization | OpenAI Moderation | Fixed model |

### By Latency Requirement

| Target | Recommended |
|--------|-------------|
| <50ms | Llama Guard 3-1B (INT4), Qwen3Guard-0.5B |
| 50–100ms | ShieldGemma 2B, Llama Guard 3-1B (FP16) |
| 100–300ms | Llama Guard 3-8B, ShieldGemma 9B, OpenAI API |
| 300–500ms | ShieldGemma 27B |

### By Platform Ecosystem

| Ecosystem | Recommended |
|-----------|-------------|
| AWS | Bedrock Guardrails + self-hosted on SageMaker |
| Azure | Azure AI Content Safety + self-hosted |
| GCP | ShieldGemma + Perspective API + Vertex AI |
| Meta/Open-Source | Llama Guard 3, Qwen3Guard (most portable) |
| Multi-Cloud | Llama Guard 3 (any cloud, no vendor lock-in) |

---

## 13. Emerging Trends

### 13.1 On-Device Guard Models

Llama Guard 3-1B, Qwen3Guard-0.5B, and INT4 quantization enable mobile/edge deployment. **Benefits:** Zero latency, offline, privacy (data stays on device), no API costs, consistent safety when offline. Frameworks: ONNX Runtime, MLX (Apple Silicon).

### 13.2 Multimodal Guard Models

Llama Guard 3-Vision (11B) and ShieldGemma 2 (4B) classify text + image safety. Future direction: audio moderation, video frame-by-frame safety, document screening, code safety, 3D/VR content.

### 13.3 Adaptive Safety

Guard models adjusting strictness by context: education (lower — academic discussion), medical (moderate — precision needed), children's app (maximum — zero tolerance), creative writing (lower — artistic expression), safety research (minimum — need to study patterns).

### 13.4 Regulatory Pressure

EU AI Act enforcement (2025–2027) drives adoption. Singapore AI Verify + MAS FEAT for financial AI. US Executive Order on AI (safety testing for powerful models). China GenAI regulations. Brazil AI Bill 2338/2023.

### 13.5 Data Sovereignty

Self-hosted guard models (Llama Guard 3, ShieldGemma, Qwen3Guard) for in-region, on-prem moderation. Azure Content Safety container for air-gapped deployments. Increasing requirement for financial and government sectors.

### 13.6 Guard Models for Agentic AI

Agent-specific guard capabilities: **tool call classification** (validate parameters before execution), **plan validation** (evaluate multi-step workflow safety), **intermediate reasoning checks** (flag dangerous chain-of-thought), **memory safety** (prevent accumulation of unsafe patterns in agent context).

---

## 14. Quick-Reference Selection Matrix

| Model / Tool | Type | Params | Languages | Customization | Deployment | Best For |
|---|---|---|---|---|---|---|
| **Llama Guard 3-8B** | Guard Model (LLM) | 8B | 8 languages | Fine-tuning | Self-hosted GPU | General safety, self-hosted |
| **Llama Guard 3-1B** | Guard Model (LLM) | 1B | 8 languages | Fine-tuning | GPU/CPU/Edge | Low-latency, on-device |
| **Llama Guard 3-Vision** | Guard (Multimodal) | 11B | 8 + images | Fine-tuning | Self-hosted GPU | Text + image safety |
| **ShieldGemma 27B** | Guard (Classifier) | 27B | English | Threshold + FT | Self-hosted GPU | Highest accuracy English |
| **ShieldGemma 9B** | Guard (Classifier) | 9B | English | Threshold + FT | Self-hosted GPU | Balanced EN safety |
| **ShieldGemma 2B** | Guard (Classifier) | 2B | English | Threshold + FT | GPU/CPU | Low-latency English |
| **ShieldGemma 2 (4B)** | Guard (Image) | 4B | Images | Threshold + FT | Self-hosted GPU | Image moderation |
| **Qwen3Guard-8B** | Guard Model (LLM) | 8B | CN + EN + | Fine-tuning | Self-hosted GPU | Chinese/English, F1 leader |
| **Qwen3Guard-0.5B** | Guard Model (LLM) | 0.5B | CN + EN + | Fine-tuning | GPU/CPU/Edge | Ultra-low latency edge |
| **OpenAI Moderation** | Moderation API | API | Multi (broad) | None | API-only | Free, multi-language |
| **Perspective API** | Toxicity API | API | 20+ | Threshold | API-only | Toxicity, publishers |
| **Azure Content Safety** | Safety API | API+Container | Multi | Threshold + custom | API + container | Azure ecosystem |
| **AWS Bedrock Guardrails** | Guardrails System | Managed | Multi | Filters + topics + PII | API-only (Bedrock) | AWS ecosystem |
| **NeMo Guardrails** | Guardrails Framewk | N/A | N/A | Colang DSL | Self-hosted | Conversational guardrails |
| **Guardrails AI** | Guardrails Framewk | N/A | N/A | RAIL spec | Library | Structured output |
| **Detoxify** | Toxicity Model | 110M | EN + multi | Fine-tuning | CPU/GPU | Lightweight toxicity |
| **Granite Guardian** | Guard Model (LLM) | 2B–13B | English | Fine-tuning | Self-hosted GPU | Enterprise, IBM ecosystem |

### Decision Tree

```
Q1: Can you self-host?
  ├── YES → Q2: What latency?
  │          ├── <50ms → Llama Guard 3-1B, Qwen3Guard-0.5B, ShieldGemma 2B
  │          ├── 50–200ms → ShieldGemma 9B, Llama Guard 3-8B, Qwen3Guard-8B
  │          └── >200ms → ShieldGemma 27B, Llama Guard 3-Vision
  └── NO → Use API: OpenAI Moderation (free) or Perspective (free)
            → Use cloud: Azure Content Safety or AWS Bedrock Guardrails

Q2 (cont): What languages?
  ├── English only → ShieldGemma
  ├── 8 languages → Llama Guard 3
  ├── Chinese + English → Qwen3Guard
  └── Broad multi-language → OpenAI Moderation

Q3: Customization needed?
  ├── Full fine-tuning → Llama Guard 3, Qwen3Guard
  ├── Threshold/policy → ShieldGemma, Azure Content Safety
  └── None → OpenAI Moderation, Perspective API

Q4: Platform ecosystem?
  ├── AWS → Bedrock Guardrails + self-hosted on SageMaker
  ├── Azure → Azure Content Safety + self-hosted
  ├── GCP → ShieldGemma + Perspective + Vertex AI
  └── Multi-cloud / On-prem → Llama Guard 3 (most portable)
```

---

## References

1. Llama Guard 3 — Meta AI (2024). https://ai.meta.com/research/publications/llama-guard-3/
2. Llama Guard 3 Vision — Meta AI (2024). https://arxiv.org/abs/2411.10414
3. ShieldGemma — Google DeepMind (2024). https://ai.google.dev/responsible/docs/safeguards/shieldgemma
4. ShieldGemma 2 — Google (2025). https://arxiv.org/abs/2504.01081
5. Qwen3Guard — Alibaba (2025). https://github.com/QwenLM/Qwen3Guard
6. OpenAI Moderation API — https://platform.openai.com/docs/guides/moderation
7. Perspective API — Jigsaw/Google. https://perspectiveapi.com/
8. Azure AI Content Safety — Microsoft. https://learn.microsoft.com/en-us/azure/ai-services/content-safety/
9. AWS Bedrock Guardrails — Amazon. https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html
10. NeMo Guardrails — NVIDIA. https://github.com/NVIDIA/NeMo-Guardrails
11. Guardrails AI — https://github.com/guardrails-ai/guardrails
12. EU AI Act — European Commission. https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai
13. AI Verify — IMDA Singapore. https://www.imda.gov.sg/ai-verify
14. MLCommons AI Safety — https://mlcommons.org/benchmarks/ai-safety/
15. Benchmarking Open-Source Safety Guard Models (2026). https://arxiv.org/abs/2605.28830

---

> **Disclaimer:** This guide reflects the state of LLM guard models as of mid-2026. The AI safety landscape evolves rapidly — always verify model versions, benchmark results, and regulatory requirements against latest sources.
