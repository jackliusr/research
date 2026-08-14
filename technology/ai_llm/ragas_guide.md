# Ragas — The RAG Evaluation Framework — A Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) · [TruLens](trulens_guide.md) · [LLM Evaluation vs Validation](llm_evaluation_vs_validation_guide.md) · [AI Agent Drift](ai_agent_drift_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [Beyond RAG](beyond_rag_guide.md) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [Vector Databases](vector_databases_guide.md) · [Responsible AI](implementing-responsible-ai.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Ragas Overview](#1-ragas-overview)
2. [Core Metrics](#2-core-metrics)
3. [Metric Details](#3-metric-details)
4. [LLM-as-a-Judge](#4-llm-as-a-judge)
5. [Testset Generation](#5-testset-generation)
6. [Usage](#6-usage)
7. [Integrations and Ecosystem](#7-integrations-and-ecosystem)
8. [Comparison with Alternatives](#8-comparison-with-alternatives)
9. [Worked Example — Evaluating a Banking RAG Assistant](#9-worked-example--evaluating-a-banking-rag-assistant)
10. [Summary — Ragas in One Page](#10-summary--ragas-in-one-page)
11. [Glossary](#11-glossary)

---

## 1. Ragas Overview

### 1.1 What Ragas Is

**Ragas** (pronounced like "ra-gas"; the name expands to **RAG Assessment** — more precisely **Retrieval Augmented Generation Assessment**) is an **open-source Python framework for the evaluation of RAG (retrieval-augmented generation) pipelines**. It is the evaluation library that asks a deceptively simple set of questions about every answer your RAG system produces:

- *Did we retrieve the right context?* (retrieval quality) · *Is the answer actually grounded in that context?* (faithfulness / hallucination)
- *Does the answer address the question?* (relevancy) · *Is the answer factually correct?* (correctness — when ground truth exists)

Ragas' own tagline captures its purpose: it helps teams **"move from 'vibe checks' to systematic evaluation loops"** — you define metrics, run them over a test set, and get reproducible per-sample and aggregate scores instead of judging answers by feel.

| Fact | Detail | Verification status |
|------|--------|---------------------|
| **What** | Open-source Python library for evaluating RAG pipelines (retrieval + generation) | ✅ Verified — repo & docs |
| **Name** | RAGAS = Retrieval Augmented Generation Assessment | ✅ Verified — arXiv paper & docs |
| **Repo** | `explodinggradients/ragas` (org later renamed `vibrantlabsai/ragas`; the old URL redirects) | ✅ Verified — GitHub |
| **Docs** | [docs.ragas.io](https://docs.ragas.io/) | ✅ Verified |
| **License** | Apache 2.0 | ✅ Verified — repo LICENSE |
| **Paper** | arXiv:2309.15217, *"Ragas: Automated Evaluation of Retrieval Augmented Generation"* (Sept 2023, EACL 2024) | ✅ Verified — arXiv |

> **Verification note:** the sibling guide [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) §8 references `github.com/vibrantlabsai/ragas` — this is correct today: the project's GitHub org was renamed from **explodinggradients** to **vibrantlabsai** (the team's company is Vibrant Labs). Both URLs resolve. This guide uses `vibrantlabsai/ragas` as the canonical repo and notes the original `explodinggradients` identity in §7.

### 1.2 Origin and History

- **The paper (2023):** introduced in September 2023 in *"Ragas: Automated Evaluation of Retrieval Augmented Generation"* (arXiv:2309.15217) by **Shahul Es, Jithin James, Luis Espinosa-Anke, Steven Schockaert**, presented at **EACL 2024**. Central claim: evaluate RAG pipelines with **reference-free** metrics — no human-written reference answer needed per question (though some Ragas metrics still use ground truth when available; see §2.5).
- **The team (explodinggradients):** the framework was built by the team behind the **explodinggradients** GitHub org (now **vibrantlabsai**), associated with the startup **Vibrant Labs** (Y Combinator-backed). The org also publishes fine-tuned evaluation/critic models on Hugging Face (e.g., `vibrantlabsai/Ragas-critic-llm-Qwen1.5-GPTQ` — fine-tuned judge LLMs, Apache-2.0) — see §4.
- **Version history:**
  - **0.1.x (2023–2024):** the original metric library — `faithfulness`, `answer_relevancy`, `context_precision`, `context_recall`, plus `answer_correctness` / `answer_similarity`; evaluation via `ragas.evaluate()` over a Hugging Face Dataset; the `TestsetGenerator` with evolution-based question generation (simple / reasoning / multi-context / conditional) appeared in this line.
  - **0.2.x (Oct 2024 → present):** a major re-architecture. Added **multi-turn / conversational evaluation**, **agentic evaluation metrics** (tool-call faithfulness, action correctness, agent goal accuracy), a rebuilt `ragas.testset` module (document-parser/node-based synthetic data generation), **multimodal metrics** (image+text faithfulness/relevance), and renamed some metrics (notably `answer_relevancy` → **`response_relevancy`** in the docs; the old import names remain as aliases). As of the latest PyPI release (Jan 2026) Ragas is on the **0.2.x** line.
- **Community:** ~5,000+ GitHub stars (mid-2025 snapshot), 80+ contributors, an active Discord, monthly releases. Star counts move fast — quote "≈5k+" and verify the live number before formal use.

### 1.3 Open Source and License

- **License: Apache 2.0** — verified from the repo LICENSE. Permissive, commercial-use friendly (important for banks: no copyleft obligations, safe to embed in internal tooling and to fork for air-gapped environments).
- **Cost model:** the library itself is free. The real cost is the **judge-model calls**: most Ragas metrics are implemented as LLM-as-a-judge evaluations (see §4), so each metric × each sample = at least one LLM call. A 1,000-question test set × 5 LLM metrics ≈ 5,000+ judge calls per eval run.
- **Data residency:** evaluation runs in your own Python process — the only data leaving your environment is what you send to the judge API. A Bedrock or self-hosted judge keeps everything on-prem (§4.2), the same argument as for [TruLens](trulens_guide.md) §1.3.

### 1.4 Positioning — "Metrics-First" RAG Evaluation

Ragas is deliberately **narrower than an observability platform and deeper than a generic metric library**:

| Capability | Ragas | Why it matters |
|------------|-------|----------------|
| **RAG-specific metrics** | ✅ Native, purpose-built | Every metric targets a specific RAG failure mode (bad retrieval, hallucination, off-topic answers) |
| **Reference-free metrics** | ✅ Core design principle (from the paper) | Evaluate without human-written gold answers; critical when no canonical answer exists |
| **Reference-based metrics** | ✅ Where useful (correctness, recall) | Factual checking against ground truth when you have goldens |
| **Synthetic test data** | ✅ `TestsetGenerator` | Generate evaluation datasets from your own documents — no manual labelling |
| **Tracing / dashboard** | ❌ Not built in | Ragas is a metrics library; for per-record traces and a dashboard pair it with LangSmith, Langfuse, or [TruLens](trulens_guide.md) |
| **CI/CD runner** | ⚠️ Library only | No built-in test runner (unlike DeepEval's pytest integration); you wrap `evaluate()` in your own CI job (§6.5) |

Design philosophy: **RAG systems fail in identifiable, separable ways** — retrieval, augmentation, generation — and each failure mode needs its own measurement. TruLens popularised the same idea as the **RAG triad** (see [trulens_guide.md](trulens_guide.md) §2); Ragas is its "metric-depth" expression: a larger, research-grounded catalogue (8+ RAG metrics, aspect critiques, agent metrics) with published formulas and LLM-judge-free variants of several metrics.

### 1.5 Ragas vs the Alternatives (Overview)

Full comparison in §8 of this guide and in the series' master evaluation-tooling guide, [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) (whose ~24-line RAGAS section **this guide expands to full depth**). Quick orientation:

| Tool | Type | One-line positioning |
|------|------|----------------------|
| **Ragas** | OSS library (Apache 2.0) | RAG-specific reference-free metrics + synthetic testset generation |
| **TruLens** | OSS eval + tracing + dashboard (MIT) | The RAG triad, feedback functions, local Streamlit dashboard — see [trulens_guide.md](trulens_guide.md) |
| **DeepEval** | OSS library (Apache 2.0) | Pytest-native LLM evaluation, 50+ metrics, includes native RagasScore integration |
| **LangSmith** | Commercial SaaS | Integrated observability + eval + datasets for the LangChain ecosystem (RAGAS available as a built-in evaluator) — see [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) §6 |
| **Arize Phoenix** | OSS + cloud | Tracing + evaluation platform with drift detection |
| **Langfuse** | OSS + cloud (MIT) | LLM-ops/tracing platform with evaluation on top |

### 1.6 When to Use Ragas

| Use case | Why Ragas fits |
|----------|----------------|
| **RAG evaluation** (offline, dev iteration) | The core purpose: score a test set against retrieval + generation metrics in one `evaluate()` call |
| **Retrieval quality assessment** | `context_precision` / `context_recall` / `context_entity_recall` isolate the retrieval stage; compare retrievers, chunking, embedding models |
| **Generation quality assessment** | `faithfulness` catches hallucination; `answer_relevancy` catches off-topic answers; `answer_correctness` checks facts against goldens |
| **CI eval gates** | Run `evaluate()` in a CI job and assert thresholds (e.g., faithfulness ≥ 0.85) before merge (§6.5) |
| **RAG architecture comparison** | Benchmark retrieval strategy A vs B (hybrid search, re-ranking, different chunk sizes) on the same synthetic test set |
| **Evaluation data generation** | When you have documents but no labelled Q&A pairs — `TestsetGenerator` synthesises them (§5) |

**When it is a weaker fit:** non-RAG LLM apps (Ragas' catalogue is RAG-centric); deep production observability (no built-in tracing — pair with [ai_agent_drift_guide.md](ai_agent_drift_guide.md) patterns); pytest-native cultures where DeepEval's runner fits better (§8).

### 1.7 The Ragas Niche

- **Metric depth for RAG** — the deepest RAG-specific catalogue in OSS: retrieval (precision, recall, entity recall), generation (faithfulness, relevancy, correctness, similarity), robustness (`noise_sensitivity`), plus aspect critiques and, since 0.2, agent + multimodal metrics.
- **Research-grounded** — published at EACL 2024 with formulas and human-correlation studies in the paper; the metric definitions are not ad-hoc prompt engineering, which makes them easier to defend in a model-validation file.
- **Reference-free by design** — the original paper's core contribution: RAG evaluation without hand-written goldens, which scales to large corpora and fast-moving product docs.
- **Synthetic testset generation** — turns documents into evaluation datasets, closing the "no labelled data" gap that blocks most eval programmes (see [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) for why labelled eval data is the usual bottleneck).
- **Pure library, composable** — no SaaS account, no vendor lock-in; scores are plain Python objects/DataFrames that plug into your existing CI, dashboards, and model-risk tooling.

---

## 2. Core Metrics

### 2.1 Design Principle — One Metric per Failure Mode

A RAG pipeline is a chain: **retrieve → augment → generate**; each link fails differently, so a single score cannot catch all failure modes. Ragas mirrors the RAG triad ([trulens_guide.md](trulens_guide.md) §2) but expands it into a full catalogue:

| Pipeline stage | Failure mode | Ragas metric |
|----------------|--------------|--------------|
| Retrieval | Retrieved chunks are irrelevant / noise | `context_precision` (are the *relevant* chunks ranked high?) |
| Retrieval | Relevant chunks are missing | `context_recall` (does the context cover the ground truth?) |
| Retrieval | Named entities (clients, products, numbers) missing | `context_entity_recall` |
| Generation | Answer asserts facts not in context | `faithfulness` |
| Generation | Answer doesn't address the question | `answer_relevancy` / `response_relevancy` |
| Generation | Answer is factually wrong vs ground truth | `answer_correctness` |
| Generation | Answer diverges semantically from the reference | `answer_similarity` |
| Robustness | Irrelevant context degrades answer quality | `noise_sensitivity` |

### 2.2 Retrieval Metrics (Component-Level)

- **Context Precision** (`context_precision`) — **Are the relevant chunks ranked high?** Given a question and the ordered list of retrieved contexts, it measures whether the chunks that actually help answer the question appear early in the ranking. It is the RAG analogue of *precision@k*: signal vs noise in the retrieval window. **Reference-free** (needs `question`, `answer`, `contexts`).
- **Context Recall** (`context_recall`) — **Does the retrieved context cover the ground truth?** Given a question, the ground-truth answer, and the retrieved contexts, it measures what fraction of the ground truth can be inferred from the retrieved context. **Reference-based** (needs `ground_truth`).
- **Context Entity Recall** (`context_entity_recall`) — **Are the key entities present?** Extracts named entities (organisations, products, amounts, dates) from the ground truth and measures the fraction that also appear in the retrieved context. Useful when entity-level coverage matters (e.g., legal or financial documents). **Reference-based** (needs `ground_truth`).

### 2.3 Generation Metrics (End-to-End)

- **Faithfulness** (`faithfulness`) — **Is the answer grounded in the context?** Decomposes the answer into statements and verifies each against the retrieved context. The canonical hallucination detector: high score = the answer says only what the context supports. **Reference-free** (needs `answer`, `contexts`).
- **Answer Relevancy** (`answer_relevancy`, renamed **`response_relevancy`** in the 0.2 docs) — **Does the answer address the question?** Measures whether the answer is on-topic and complete relative to the user's question (the RAG-triad "answer relevance"). **Reference-free** (needs `question`, `answer`).
- **Answer Correctness** (`answer_correctness`) — **Is the answer factually correct against the ground truth?** Compares the answer to a reference answer: factual overlap (true positives / false positives / false negatives) blended with semantic similarity. **Reference-based** (needs `ground_truth`).
- **Answer Similarity** (`answer_similarity`) — **How semantically similar is the answer to the reference?** An embedding-based (cosine) similarity between answer and ground truth — a soft, cheap proxy for correctness when the judge LLM cost is a concern. **Reference-based** (needs `ground_truth`).
- **Noise Sensitivity** (`noise_sensitivity`) — **How badly does irrelevant context hurt the answer?** Measures the degradation in answer quality when noise (irrelevant chunks) is present in the context — a robustness check for the whole pipeline. **Reference-based** in its original form (needs `ground_truth`).

### 2.4 Additional Catalogues (0.2+)

- **Aspect Critique** — judge-based critiques with fixed rubrics (`harmfulness`, `maliciousness`, `coherence`, `correctness`, `conciseness`) plus **custom aspects** defined in natural language (handy for bank policies, e.g., "must include a disclaimer").
- **Agent metrics (0.2)** — for agentic RAG/agents: `tool_call_correctness`, `tool_call_error`, `tool_call_faithfulness`, `action_correctness`, `agent_goal_accuracy`. See [autonomous_agents_guide.md](autonomous_agents_guide.md) §5 for agent evaluation context.
- **Multimodal metrics (0.2)** — faithfulness and relevance for image+text inputs/outputs.

### 2.5 Metric Catalog (Reference Table)

| Metric | Type | What it measures | Inputs | Use case |
|--------|------|------------------|--------|----------|
| `faithfulness` | Reference-free (LLM judge) | Answer statements supported by context | `answer`, `contexts` | Hallucination gate; generation quality |
| `answer_relevancy` (a.k.a. `response_relevancy`) | Reference-free (LLM + embeddings) | Answer addresses the question | `question`, `answer` | Off-topic / unhelpful answers |
| `context_precision` | Reference-free (LLM judge) | Relevant chunks ranked high | `question`, `answer`, `contexts` | Retrieval ranking quality |
| `context_recall` | Reference-based (LLM judge) | Context covers ground truth | `question`, `ground_truth`, `contexts` | Retrieval completeness |
| `context_entity_recall` | Reference-based (NER, no judge LLM) | Ground-truth entities present in context | `ground_truth`, `contexts` | Entity-level retrieval coverage |
| `answer_correctness` | Reference-based (LLM judge + embeddings) | Factual agreement with ground truth | `question`, `answer`, `ground_truth` | Factual accuracy vs goldens |
| `answer_similarity` | Reference-based (embeddings only) | Semantic similarity to ground truth | `answer`, `ground_truth` | Cheap correctness proxy |
| `noise_sensitivity` | Reference-based (LLM judge) | Robustness to irrelevant context | `question`, `answer`, `contexts`, `ground_truth` | Pipeline robustness |
| Aspect critiques (`harmfulness`, etc.) | Reference-free (LLM judge) | Rubric-based quality/safety aspects | `question`, `answer` | Policy/safety checks, custom aspects |

> **Verification note:** metric names follow the current docs.ragas.io catalogue. The 0.2 docs renamed `answer_relevancy` → **`response_relevancy`**; both names appear in the wild and the old import alias still works in the 0.2 line. `context_relevancy` (an older metric) was deprecated/removed in favour of `context_precision` — if you see `context_relevancy` in older posts, treat it as legacy. Full per-metric mechanics are in §3.

## 3. Metric Details

### 3.1 Faithfulness — The Hallucination Detector

**Question it answers:** *Is every claim in the answer supported by the retrieved context?*

**Mechanism (two LLM steps):**
1. **Decompose:** an LLM splits the answer into a list of **statements** (atomic claims). Example answer: *"Our standby letter of credit fees start at 0.5% per annum, and issuance takes up to 3 business days."* → statements: S1 = "standby letter of credit fees start at 0.5% per annum"; S2 = "issuance takes up to 3 business days".
2. **Verify:** for each statement, an LLM (or NLI model) checks whether it is **entailed by / attributable to** the retrieved context. Each statement is scored 1 (supported) or 0 (not supported).

**Formula:**

```
faithfulness = (number of statements supported by the context) / (total number of statements in the answer)
```

Range [0, 1]; higher = more grounded. If the answer has zero statements (degenerate), the metric is undefined — guard against empty answers in your eval pipeline.

**What it catches:** hallucination — the LLM inventing fees, limits, client names, or regulatory details not present in the retrieved documents. In banking this is the highest-severity failure mode (see the triad discussion in [trulens_guide.md](trulens_guide.md) §2.2): a fabricated fee or eligibility rule is worse than a missing answer.

**Caveats:** faithfulness only checks the *retrieved* context — if retrieval was bad, a "faithful" answer can still be factually wrong, so pair it with `context_recall`/`context_precision`. Also, *verbose* answers have more statements and get punished harder than terse ones; compare answers of similar verbosity.

### 3.2 Context Precision — Signal vs Noise in the Retrieval Window

**Question it answers:** *Are the chunks that actually help answer the question ranked at the top?*

**Mechanism:** for each retrieved chunk (in ranked order), an LLM judge labels whether the chunk is **relevant** to the question (optionally conditioned on the answer). The metric then computes a precision-style score that rewards relevant chunks appearing early in the ranking — the RAG analogue of precision@k averaged over the relevant chunks.

**Formula (average precision over the relevant contexts):**

```
context_precision = (1 / |relevant contexts|) × Σ_k [ precision@k × rel_k ]

where rel_k = 1 if the k-th ranked chunk is relevant (else 0),
      precision@k = (number of relevant chunks in the top k) / k,
      and the sum runs over all ranked positions k.
```

Range [0, 1]. A perfect retriever that puts all relevant chunks first scores 1.0; a retriever that buries the one useful chunk under four irrelevant ones scores low.

**What it catches:** retrieval noise — the embedding/hybrid search returning broadly "similar" but useless chunks, over-broad chunking that dilutes the signal, or a re-ranker mis-ordering results. Low precision + high recall = the info is in the window but buried in noise — which also drags down faithfulness, as the LLM latches onto the wrong chunk.

### 3.3 Context Recall — Ground-Truth Coverage

**Question it answers:** *Could the retrieved context, in principle, support the correct answer?*

**Mechanism:** take the **ground-truth answer** (reference), decompose it into statements (same statement-extraction step as faithfulness), then check each ground-truth statement against the *retrieved contexts* — is it attributable to at least one retrieved chunk?

**Formula:**

```
context_recall = (number of ground-truth statements attributable to the context) / (total ground-truth statements)
```

Range [0, 1]. **Reference-based**: it needs `ground_truth`, which makes it the one core metric that cannot run in a purely reference-free eval loop.

**What it catches:** retrieval incompleteness — the retriever returning *relevant-ish* chunks that miss the specific facts needed (e.g., the term sheet retrieved but not the fee schedule; the FAQ page retrieved but not the operational handbook). Low context recall is the classic "the LLM couldn't have known" failure: no metric on the generation side can fix facts that were never retrieved. Debugging this leads upstream to chunking, embedding, and index strategy — see [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) and [vector_databases_guide.md](vector_databases_guide.md).

### 3.4 Answer Relevancy — Question–Answer Alignment

**Question it answers:** *Does the answer actually respond to the question asked?*

**Mechanism (LLM + embeddings):**
1. An LLM generates **a set of plausible questions** from the answer (typically 3) — i.e., "what questions would this answer be a good answer to?"
2. Each generated question is embedded and compared (cosine similarity) with the embedding of the **original user question**; the score is the **mean cosine similarity** across the generated questions.

**Formula:**

```
answer_relevancy = mean( cos_sim(embed(original_question), embed(generated_question_i)) )  for i = 1..n generated questions
```

Range [0, 1]. **Reference-free** — needs only `question` and `answer`.

**What it catches:** off-topic or evasive answers — the assistant answering the nearest FAQ instead of the specific question, answering a different product question, or refusing with boilerplate when the knowledge base actually covers the topic. Note the mechanism: an answer that is verbose and self-contained tends to generate many question variants, inflating the score even when the answer only partially addresses the user's question — a known bias, so treat this metric as a *relevancy floor*, not a ceiling. (This is also the metric TruLens' triad calls "answer relevance" — same idea, different mechanism; see [trulens_guide.md](trulens_guide.md) §2.)

### 3.5 Answer Correctness — Factual Comparison with Ground Truth

**Question it answers:** *Is the answer factually correct compared to a reference answer?*

**Mechanism:** the answer and the ground truth are decomposed into statements; an LLM judge computes factual overlap:

- **TP (true positive):** statements present in both answer and ground truth;
- **FP (false positive):** statements in the answer but not in the ground truth (fabricated/extra claims);
- **FN (false negative):** statements in the ground truth but missing from the answer (omissions).

**Formula (the classic Ragas formulation):**

```
factual_correctness = (TP) / (TP + FP + FN)
answer_correctness  = 0.75 × factual_correctness + 0.25 × semantic_similarity(answer, ground_truth)
```

The blend rewards both *factual precision* (TP-dominated) and *semantic likeness* to the reference (via `answer_similarity`-style embedding similarity). The 0.75/0.25 weights are the historical defaults and are configurable.

**What it catches:** factual errors that faithfulness cannot see — when the context was fine but the model still said the wrong number, or when the answer is grounded in an *outdated* document and contradicts the current ground truth. This is the metric to run when you have **goldens** (human-reviewed reference answers), e.g., in a regulated environment where a validation team curates a golden set (see [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) on goldens as validation evidence).

### 3.6 Metric Formulas — Summary

| Metric | Formula (conceptual) | Deterministic? |
|--------|----------------------|----------------|
| Faithfulness | supported statements / total statements | No (LLM judge for support labels) |
| Context precision | Σ_k (precision@k × rel_k) / \|relevant chunks\| | Partially (relevance labels from LLM) |
| Context recall | attributable GT statements / total GT statements | No (LLM judge) |
| Answer relevancy | mean cosine(original Q, generated Qs) | No (LLM generates questions; embeddings deterministic) |
| Answer correctness | 0.75 × TP/(TP+FP+FN) + 0.25 × sim(answer, GT) | No (LLM for TP/FP/FN) |
| Answer similarity | cosine(embed(answer), embed(GT)) | **Yes** — no LLM judge |
| Context entity recall | matching entities / total GT entities | **Yes** — NER-based, no judge LLM |
| Noise sensitivity | correctness(noisy context) vs correctness(clean context) | No (LLM judge) |

**Two deterministic metrics are worth knowing:** `answer_similarity` (embedding cosine) and `context_entity_recall` (entity matching) involve **no judge LLM at all** — they are cheap, reproducible, and useful as smoke tests or as building blocks in CI when judge-model cost is a concern.

### 3.7 Metric Examples — Worked Calculations

All examples below are worked by hand to show the mechanics; a real run would have the LLM judge produce the labels.

**Example A — Faithfulness (banking Q&A):**

- Question: *"What are the fees for standby letters of credit?"*
- Context (retrieved): *"Standby letter of credit issuance fees are 0.5% per annum, subject to a minimum of USD 250. Issuance typically completes within 3 business days."*
- Answer: *"Standby LC issuance fees are 0.5% per annum with a USD 250 minimum, and issuance takes about 3 business days."*
- Statements: S1 "fees are 0.5% per annum", S2 "minimum of USD 250", S3 "issuance takes about 3 business days".
- Verification: S1 ✅ supported, S2 ✅ supported, S3 ✅ supported.
- **Faithfulness = 3/3 = 1.0** — fully grounded.

Variant: if the answer instead said *"…with a USD 500 minimum"*, S2 would fail (0/1) → **faithfulness = 2/3 ≈ 0.67** — a hallucinated number caught.

**Example B — Context Precision (ranked relevance):**

- Question: *"Does our documentary credit product cover pre-shipment finance?"*
- Retrieved chunks in rank order: [C1: credit insurance FAQ, C2: documentary credit term sheet, C3: shipping docs checklist].
- Relevance labels (LLM judge): C1 = irrelevant, C2 = relevant, C3 = irrelevant.
- |relevant| = 1. precision@1 = 0/1 = 0; precision@2 = 1/2 = 0.5; precision@3 = 1/3 ≈ 0.333.
- **Context precision = (0×0 + 0.5×1 + 0.333×0) / 1 = 0.5** — the one relevant chunk is buried at rank 2; the retriever deserves a retrieval fix (§9).

**Example C — Context Recall (ground-truth coverage):**

- Ground truth: *"Documentary credits provide payment against compliant documents. Pre-shipment finance is available under the export credit line."*
- GT statements: G1 "documentary credits provide payment against compliant documents"; G2 "pre-shipment finance is available under the export credit line".
- Retrieved contexts contain only the documentary-credit terms; nothing about pre-shipment finance.
- G1 ✅ attributable, G2 ❌ missing.
- **Context recall = 1/2 = 0.5** — retrieval is missing the finance products; generation-side metrics cannot fix this.

**Example D — Answer Correctness (TP/FP/FN):**

- Ground truth: *"The minimum issuance fee is USD 250."* → statements: G1 "minimum issuance fee is USD 250".
- Answer: *"The minimum issuance fee is USD 250, and guarantees are always unsecured."* → statements: A1 "minimum issuance fee is USD 250" (TP), A2 "guarantees are always unsecured" (FP — not in GT).
- FN = 0 (G1 covered). factual_correctness = TP/(TP+FP+FN) = 1/(1+1+0) = 0.5. If semantic similarity ≈ 0.8:
- **Answer correctness = 0.75×0.5 + 0.25×0.8 = 0.575** — the fabricated "always unsecured" claim halves the factual component.

**Example E — Answer Relevancy (sketch):**

- Question: *"What is the process for cancelling a standby LC?"*
- Answer: *"Standby LC issuance fees are 0.5% per annum."*
- Generated questions from the answer: "What are standby LC fees?", "How much does a standby LC cost?" — neither matches the cancellation question. Mean cosine ≈ low (e.g., 0.2–0.4).
- **Answer relevancy ≈ low** — the answer is grounded (faithfulness could be 1.0) but irrelevant: a classic *"faithful but useless"* case that only shows up when you run both metrics.

---

## 4. LLM-as-a-Judge

### 4.1 Ragas Runs on LLM Judges

Most Ragas metrics are implemented as **LLM-as-a-judge** evaluations: the metric is a prompt template plus a parsing/aggregation step, executed against an underlying **evaluation (critic) LLM**. The judge model does the statement extraction, the relevance labelling, the support verification, and the correctness comparison that the formulas in §3.1–§3.5 consume. This is the same architecture family as G-Eval and TruLens' feedback functions (see [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) §11 and [trulens_guide.md](trulens_guide.md) §3.4), with two differences:

- Ragas ships **ready-made, research-validated prompt templates** for each metric (the `faithfulness` prompt, the `context_precision` relevance prompt, etc.) rather than asking you to write your own rubrics;
- Ragas' judge outputs are **structured** (statement lists, per-item verdicts), which the library aggregates into the scalar metrics — you rarely touch raw prompts.

### 4.2 Configurable Judge Models

Ragas is model-agnostic: any LangChain-compatible chat model can act as the judge, which matters for banks on restricted infrastructure:

| Judge provider | How | Notes |
|----------------|-----|-------|
| **OpenAI** | default when using `evaluate()` with `OPENAI_API_KEY`; or pass an explicit `ChatOpenAI` | The zero-config path; also Azure OpenAI |
| **AWS Bedrock** | pass a `ChatBedrock` model (Claude, etc.) | Important for banks on AWS landing zones — data stays inside the VPC |
| **Local / self-hosted** | pass any LangChain LLM (Ollama, vLLM, llama.cpp) | Keeps all evaluation data on-prem; slower and weaker judges reduce metric reliability |
| **Ragas critic LLMs** | the team releases fine-tuned judge models on Hugging Face (e.g., `vibrantlabsai/Ragas-critic-llm-Qwen1.5-GPTQ`, Apache-2.0) | Fine-tuned on GPT-4 judge outputs; a cheaper local-judge option |

The pattern is the same everywhere: construct the judge model, attach it to the metrics (or let Ragas default to OpenAI), run `evaluate()` (§6.3). Ragas also supports **different models per metric** — a strong judge for `answer_correctness`, a cheap local one for `context_precision`.

> **Banking note:** judge-model choice is an evaluation-data-governance decision as much as a quality decision. With a self-hosted or Bedrock judge, question/answer/context text never leaves your controlled environment — the same residency argument made for [TruLens](trulens_guide.md) §1.3 and the validation framing in [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md).

### 4.3 Prompt-Based, Not Hard-Coded

Each metric's prompt template is a first-class, inspectable object in the library (the `faithfulness` prompt, the relevance-criteria prompts, etc.). You can:

- **Inspect** them (useful when a metric behaves unexpectedly);
- **Customise** the criteria sections (e.g., make the `context_precision` relevance definition domain-aware);
- **Localise** them (prompts are language-sensitive — for non-English corpora, check the prompt language vs your corpus).

Because the prompts are the actual evaluation instruments, version them alongside your app code — a prompt tweak shifts scores, and the model-validation file should record which judge + prompt version produced each eval run.

### 4.4 Cost — The Real Budget Line

Each LLM-judged metric consumes **at least one judge call per sample**; the statement-based metrics (faithfulness, context recall, answer correctness) consume **several** (extraction + one verification call per statement):

| Metric | Judge calls per sample (typical) |
|--------|-----------------------------------|
| `answer_similarity` | 0 (embeddings only) |
| `context_entity_recall` | 0 (NER only) |
| `context_precision` | ~1 per sample (per-chunk relevance labelling; can be batched) |
| `answer_relevancy` | 1 (question generation) + embeddings |
| `faithfulness` | 1 (statement extraction) + 1 per statement (verification) |
| `context_recall` | 1 + 1 per GT statement |
| `answer_correctness` | 1 + 1 per statement pair |

Rough rule: **5 metrics × 1,000 samples ≈ 5,000–10,000+ judge calls** per full run — affordable in dev, compounding in CI (every merge runs the gate) and regression suites. Levers: the deterministic metrics (§3.6) for smoke tests, a 200-question sample for daily gates with the full set weekly, and aggressive sampling (cost models in [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) §14).

### 4.5 Judge Quality — The Honest View

Ragas' scores are only as trustworthy as the judge LLM, and the failure modes are real:

- **Judge capability:** the judge should be *at least as capable* as the generator it evaluates. A weak local judge labelling relevance for answers produced by a frontier model will under-detect errors — your faithfulness scores will look better than reality.
- **Calibration and bias:** LLM judges exhibit well-documented biases — favouring verbose/self-consistent answers, positional bias, and self-preference (judges score their own family's outputs higher). The paper and follow-up work studied judge agreement; in practice you should **calibrate**: spot-check a sample of judge verdicts against human labels and compute agreement (Cohen's kappa or similar) before trusting thresholds. See the judge-model guidance in [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) and the human-eval discussion in [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md).
- **Variance:** LLM judges are non-deterministic (temperature, sampling). Run each metric **more than once** on small samples to estimate score variance; if variance is high relative to your threshold, either fix the judge (stronger model, lower temperature) or widen the gate margin. Ragas lets you set the judge's temperature; lower is generally better for evaluation.
- **The honest bottom line:** Ragas metrics are *strong heuristics, not proofs* — they rank pipelines (A vs B, v1 vs v2) and catch gross failures, but do not replace human review of edge cases. Treat scores as **monitoring instruments**; back formal sign-off with the human-in-the-loop validation in [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md).

---

## 5. Testset Generation

### 5.1 The Problem Ragas Solves

Evaluation needs test data: questions, (optionally) reference answers, and the expectation that the app should handle them. Hand-labelling a few hundred banking questions is expensive, slow, and quickly stale as products change. Ragas' answer is the **`TestsetGenerator`**: a synthetic data generator that turns **your own documents** into an evaluation dataset, so the test set tracks your actual knowledge base rather than a stale hand-written fixture.

### 5.2 How Generation Works

High-level pipeline (0.2 architecture; the 0.1 flow was similar in spirit):

1. **Load documents** — ingest your corpus (LangChain `Document`s via `generate_with_langchain_docs`, or LlamaIndex documents via `generate_with_llamaindex_docs`).
2. **Chunk / structure** — the generator splits documents into nodes/chunks (configurable parser, e.g., LangChain text splitters) that will act as the "context" for generated questions.
3. **Evolve questions** — starting from seed questions extracted from the chunks, an LLM **evolves** them into harder variants using evolution operators (see §5.3), each designed to stress a different RAG capability.
4. **Synthesise answers** — for each (question, contexts) pair the generator writes a reference answer, optionally with a *scenario* describing the intended difficulty. The result is a Hugging Face `Dataset` (`question`, `answer`, `contexts`, …) directly consumable by `evaluate()` (§6).

Generation is LLM-driven (question evolution, answer synthesis) — **same cost model as evaluation**: one pass over a few hundred chunks costs real judge tokens.

### 5.3 Testset Types — The Evolution Operators

The `distributions` parameter of `generate_with_langchain_docs` lets you mix question types. The canonical set (0.1, still the mental model for 0.2):

| Evolution | Question style | What it stresses |
|-----------|----------------|------------------|
| `simple` | Direct factoid questions: *"What is the minimum issuance fee?"* | Basic retrieval + faithful answering |
| `reasoning` | Multi-hop questions: *"Given the fee schedule and the eligibility criteria, would a USD 2m standby LC for a new client qualify for the reduced rate?"* | Reasoning over retrieved facts; answer composition |
| `multi_context` | Questions whose answer requires **multiple chunks**: *"Compare the fee structures of documentary credits and standby LCs."* | Multi-source retrieval; context stitching |
| `conditional` | Questions with conditions/constraints: *"If the beneficiary is in a sanctions-affected country, what additional documents apply?"* | Conditional logic; edge-case retrieval |

You specify the mix, e.g. `distributions={simple: 0.5, reasoning: 0.25, multi_context: 0.25}`. The 0.2 `ragas.testset` rework generalises this with an evolution-based architecture (LLM-based evolution, customisable node parser, scenario filtering) while keeping the same intent: generate questions that are *harder than trivial* so the eval suite has signal.

> **Verification note:** the four evolution names (`simple`, `reasoning`, `multi_context`, `conditional`) are verified against the 0.1 docs and numerous 0.1-era examples; the 0.2 `ragas.testset` module re-architected generation (document parsers, `TestsetGenerator` with `LLMTestsetGenerator`/evolution configs) — check the current docs.ragas.io testset-generation page for the exact 0.2 API surface before writing code against it.

### 5.4 Testset Quality — Garbage In, Garbage Out

Synthetic generation is fast but not free of defects. Real-world issues to plan for:

- **Surface questions:** generated questions sometimes paraphrase the document so closely that retrieval is trivially easy — inflating scores. The evolution operators mitigate this; also sample the generated set and prune trivial items.
- **LLM artifacts:** synthetic reference answers can contain the generator's own errors; a wrong "ground truth" poisons `context_recall`/`answer_correctness`. **Human review of a sample** (10–20%) of the generated set is the standard mitigation — and in a bank, the model-validation file should record that the golden subset was human-reviewed (see [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md)).
- **Coverage:** generation coverage follows document coverage. Documents that never get indexed can never produce questions; if a product line is under-represented in the corpus, it will be under-represented in the eval set — audit the generated set's topic distribution.
- **Drift:** a synthetic set generated once goes stale as the corpus changes; regenerate (or at least refresh) on a schedule, not just once.

### 5.5 Testset Usage — From Synthetic to Golden

The generated dataset is the **evaluation dataset** for `evaluate()`. Two operating modes: 1. **Fully synthetic (fast, cheap, broad)** — run the whole generated set as-is for dev iteration, architecture comparison, and CI smoke gates. 2. **Golden subset (slow, expensive, authoritative)** — curate a smaller, human-reviewed, version-controlled subset with verified reference answers for release gates and regulatory sign-off. Ragas gives you the generation machinery; the human review step is yours (where evaluation meets validation — see [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md)).

### 5.6 Generation Example (Code Sketch)

```python
from ragas.testset.generator import TestsetGenerator
from ragas.testset.evolutions import simple, reasoning, multi_context
from langchain_openai import ChatOpenAI, OpenAIEmbeddings

# 0.1-style API (most documented); the 0.2 ragas.testset module has a similar flow — check current docs.
generator = TestsetGenerator.with_openai()
# or: TestsetGenerator(llm=ChatOpenAI(model="gpt-4o"),
#                      embedding_model=OpenAIEmbeddings())

documents = [...]  # list of LangChain Documents from your corpus

testset = generator.generate_with_langchain_docs(
    documents,
    test_size=50,
    distributions={simple: 0.5, reasoning: 0.25, multi_context: 0.25},
)

dataset = testset.to_dataset()   # Hugging Face Dataset: question, answer, contexts, ...
print(dataset)
# Dataset({ features: ['question', 'answer', 'contexts', ...], num_rows: 50 })
```

The generated `dataset` feeds straight into `evaluate()` (§6.3).

## 6. Usage

### 6.1 Install

```bash
pip install ragas
```

- Optional extras: `"ragas[langchain]"`, `"ragas[llamaindex]"`, `"ragas[all]"`. Pure Python library, no SaaS/server component; it pulls in `datasets` and LangChain-era deps — vendor the wheels and install offline for air-gapped bank environments.

### 6.2 The Dataset Format

`evaluate()` consumes a Hugging Face `Dataset` (from `datasets`) with these columns:

| Column | Type | Required for | Description |
|--------|------|--------------|-------------|
| `question` | `str` | all metrics | The user's query |
| `answer` | `str` | generation metrics | The RAG system's generated answer |
| `contexts` | `list[str]` | retrieval + faithfulness metrics | The retrieved chunks, in ranked order |
| `ground_truth` | `str` | reference-based metrics | The reference/correct answer (needed for `context_recall`, `answer_correctness`, `answer_similarity`, `context_entity_recall`, `noise_sensitivity`) |

Notes:

- `contexts` must be the **actual retrieved chunks** from your pipeline (in rank order) — not the whole knowledge base. This is what makes the metrics component-level: they evaluate *your* retriever's output, not an ideal one.
- `ground_truth` is optional at the dataset level, but **required by** the reference-based metrics. Run reference-free metrics (`faithfulness`, `answer_relevancy`, `context_precision`) on any dataset; add `ground_truth` when you want recall/correctness.
- Non-standard column names map via `evaluate(column_map=...)` (e.g., `{"contexts": "contexts_v1"}`); multi-turn/agentic eval (0.2) extends the schema with conversation/tool-call columns — check current docs.

### 6.3 `evaluate()` — The Core API

```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_precision,
    context_recall,
)

result = evaluate(
    dataset=eval_dataset,                    # HF Dataset: question, answer, contexts, ground_truth
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall],
)

result.ravel()   # per-sample scores, indexed by sample
result           # aggregated scores, e.g.:
# {'faithfulness': 0.89, 'answer_relevancy': 0.78, 'context_precision': 0.71, 'context_recall': 0.83}
```

- **Metrics selection:** import the metric objects you want; pass them as a list. `evaluate()` runs each metric over the whole dataset and returns a `Result` with **per-sample scores** (`result.ravel()` / `result.to_pandas()`) and **aggregated scores** (the `Result` object prints the mean per metric; the full distribution is available per sample).
- **Judge model:** by default metrics use OpenAI (`OPENAI_API_KEY`). For explicit control, attach models to metrics before evaluating (see §4.2), e.g.:

```python
from langchain_openai import ChatOpenAI
from ragas.metrics import faithfulness

faithfulness.llm = ChatOpenAI(model="gpt-4o", temperature=0)   # judge for this metric
# ...repeat per metric; embeddings configurable via metric.embeddings
```

- **Async/performance:** `evaluate()` is parallelised (a thread pool) so a 200-sample × 4-metric run completes in a few minutes against a typical judge API; the wall-clock cost is dominated by judge-model latency (§4.4).

### 6.4 Results — What You Actually Get

- **Per-sample scores:** the row-level values that show *which* questions fail (export via `result.to_pandas()`, slice the worst rows — these become your debugging backlog and later your regression fixtures).
- **Aggregated scores:** the mean per metric — headline numbers for dashboards and CI gates; report them **with sample count and judge-model + prompt versions** (a mean without context is noise — see [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md)). **Score semantics:** all core metrics are [0, 1], higher = better, and thresholds must be calibrated against your own goldens (e.g., faithfulness 0.85, context recall 0.75) before encoding them as gates.

### 6.5 CI/CD — Eval Gates

Ragas is a library, so the CI layer is yours to write — which is simple and fully controllable:

```yaml
# .github/workflows/rag-eval.yml (sketch): install ragas -> run RAG over the test set -> build HF Dataset -> evaluate -> assert thresholds
```

```python
# eval_gate.py (sketch)
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

result = evaluate(dataset=eval_dataset,
                  metrics=[faithfulness, answer_relevancy, context_precision, context_recall])
scores = result.ravel()

thresholds = {"faithfulness": 0.85, "answer_relevancy": 0.75,
              "context_precision": 0.70, "context_recall": 0.80}

fails = {m: s for m, s in scores.items() if s < thresholds[m]}
if fails:
    print("EVAL GATE FAILED:", fails)   # -> non-zero exit code blocks the merge
    raise SystemExit(1)
```

CI gate design points:

- **Sample the test set for speed:** run a 100–200-question sample on every PR, the full set nightly/weekly. Judge cost per merge is the constraint (§4.4).
- **Track, don't just assert:** persist scores (CSV/DataFrame, or push to your observability stack) so you can see *trends* — a metric degrading slowly across two weeks is how drift shows up before a hard failure (see [ai_agent_drift_guide.md](ai_agent_drift_guide.md)).
- **Version the eval spec:** the dataset, metric list, judge model, and prompt versions together form the eval specification; record it in the model-validation file when the gate is used for sign-off.

### 6.6 Usage Example — Evaluate a RAG (Complete Sketch)

```python
from datasets import Dataset
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

# 1. Run your RAG over the test questions and capture retrieval + generation output
questions = ["What are the fees for standby letters of credit?"]
answers = ["Issuance fees are 0.5% per annum with a USD 250 minimum."]
contexts = [["Standby LC issuance fees are 0.5% p.a., min USD 250. Issuance takes 3 business days."]]
ground_truths = ["Standby letter of credit issuance fees are 0.5% per annum, minimum USD 250."]

eval_dataset = Dataset.from_dict({
    "question": questions,
    "answer": answers,
    "contexts": contexts,
    "ground_truth": ground_truths,
})

# 2. Evaluate
result = evaluate(dataset=eval_dataset,
                  metrics=[faithfulness, answer_relevancy, context_precision, context_recall])

# 3. Read results
print(result)                # aggregated scores per metric
df = result.to_pandas()      # per-sample scores
```

---

## 7. Integrations and Ecosystem

### 7.1 Framework Integrations

Ragas is deliberately **framework-agnostic on the data side** (it evaluates whatever `question`/`answer`/`contexts` you hand it) but ships conveniences for the two dominant RAG frameworks:

| Integration | What it gives you | Notes |
|-------------|-------------------|-------|
| **LangChain** | `TestsetGenerator.generate_with_langchain_docs()`; judge models via any LangChain chat model (`ChatOpenAI`, `ChatBedrock`, `Ollama`, …) | The most common pairing; LangChain documents feed generation, LangChain LLMs feed judging |
| **LlamaIndex** | `generate_with_llamaindex_docs()`; judge via LlamaIndex LLMs | Same generation flow for LlamaIndex-based RAG apps |
| **Hugging Face** | `datasets` is the native dataset type for `evaluate()`; critic/judge LLMs published on the HF Hub (`vibrantlabsai/...`) | Your eval set is a HF Dataset — versionable, shareable, CI-friendly |
| **LangSmith** | RAGAS is available as a **built-in evaluator** in LangSmith; scores can be logged into LangSmith runs | The managed route: Ragas metrics + LangSmith tracing/datasets (see below) |

### 7.2 Ragas Inside LangSmith

LangSmith (the commercial LLM-ops platform in the LangChain ecosystem) ships a **RAGAS evaluator**: select it, point it at your dataset, and LangSmith runs the Ragas metrics server-side, attaching scores to traced runs — the "RAGAS-in-LangSmith" pattern in [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md) §6/§8. **Ragas supplies the metric mathematics; LangSmith supplies datasets, tracing, dashboards.** The *managed* option (data leaves your environment unless you self-host); running `ragas` directly keeps everything in-process — choose per data-governance requirements.

### 7.3 Community

- **GitHub:** `vibrantlabsai/ragas` (originally `explodinggradients/ragas`) — **Apache 2.0**, Python. ~5,000+ stars, 80+ contributors (mid-2025); verify the live count before formal use.
- **Docs:** [docs.ragas.io](https://docs.ragas.io/) — the canonical reference for the current metric catalogue, `evaluate()` API, and the 0.2 `ragas.testset` module. The docs also host the metric pages with formulas and examples.
- **Community channels:** Discord server, GitHub Discussions, and a large ecosystem of third-party tutorials (the RAGAS eval pattern is one of the most-documented OSS evaluation workflows).
- **Hugging Face:** the team publishes fine-tuned critic LLMs (e.g., `vibrantlabsai/Ragas-critic-llm-Qwen1.5-GPTQ`) and dataset artefacts under the same org.

### 7.4 License

**Apache 2.0** — verified from the repo LICENSE file. Permissive: use, modify, embed commercially, and fork for air-gapped deployment without copyleft obligations. (Compare: TruLens is MIT, DeepEval is Apache-2.0, LangSmith is closed-source — see [trulens_guide.md](trulens_guide.md) §1.3 and §8 of this guide.)

### 7.5 Development and Versions

- **0.1.x** — the classic metric library + `TestsetGenerator` with evolutions; the API most tutorials show. **0.2.x** — the current line (PyPI releases through at least Jan 2026): multi-turn/agentic metrics, rebuilt `ragas.testset`, multimodal metrics, `response_relevancy` naming. Releases are roughly monthly — pin your version in CI so score baselines don't shift, and read the changelog before upgrading (metric prompt changes can move scores by a few points).

---

## 8. Comparison with Alternatives

### 8.1 The Comparison Table

The series' master comparison lives in [llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md); this table focuses on the RAG-evaluation-relevant axes (TruLens detail in [trulens_guide.md](trulens_guide.md), DeepEval detail in the master guide §5, LangSmith in §6):

| Axis | **Ragas** | **TruLens** | **DeepEval** | **LangSmith (RAGAS-in-LangSmith)** |
|------|-----------|-------------|--------------|-------------------------------------|
| **Type** | OSS library (Apache 2.0) | OSS eval + tracing + dashboard (MIT) | OSS library (Apache 2.0) | Commercial SaaS platform |
| **Core metrics** | 8+ RAG-specific (faithfulness, context precision/recall, entity recall, noise sensitivity…) | RAG triad + HHH + programmatic | 50+ general metrics incl. RAG suite + RagasScore | Ragas metrics as a built-in evaluator + LangSmith metrics |
| **Reference-free** | ✅ Core design (paper) | ✅ (triad) | ✅ | ✅ (via RAGAS evaluator) |
| **LLM-as-a-judge** | ✅ configurable (OpenAI/Bedrock/local/critic LLMs) | ✅ via providers | ✅ model-agnostic | ✅ managed |
| **Testset generation** | ✅ `TestsetGenerator` (evolutions) | ❌ (bring your own data) | ✅ data synthesis (Evol) | ✅ datasets + synthetic data tooling |
| **Tracing / observability** | ❌ (library only) | ✅ OTEL-native tracing + dashboard | ❌ (library; pytest runner) | ✅ platform-grade tracing |
| **CI/CD** | ⚠️ library — you write the gate | ⚠️ offline eval scripts | ✅ pytest/CLI native | ✅ platform pipelines |
| **Open-source** | ✅ Apache 2.0 | ✅ MIT | ✅ Apache 2.0 | ❌ closed |
| **Best for** | RAG metric depth + synthetic test sets | Triad + per-record debugging + local dashboard | Pytest-native eval culture, broad metric library | Teams already in LangSmith wanting managed eval |

### 8.2 Ragas Strengths

- **Metric depth for RAG:** the deepest RAG-specific catalogue in OSS — retrieval-stage (precision, recall, entity recall) *and* generation-stage (faithfulness, relevancy, correctness, similarity) *and* robustness (`noise_sensitivity`), plus aspect critiques and agent/multimodal metrics in 0.2.
- **Research-grounded:** published formulas and human-correlation studies (EACL 2024 paper) — defensible in a model-validation file in a way that ad-hoc rubric prompts are not.
- **Synthetic testset generation built in:** documents → eval dataset, solving the "no labelled data" bottleneck; the other tools either lack it (TruLens) or implement it differently (DeepEval).
- **Reference-free by default:** eval without goldens when you don't have them; reference-based metrics when you do.
- **Open and composable:** Apache 2.0, plain Python objects/DataFrames out, no platform lock-in; deterministic sub-metrics (`answer_similarity`, `context_entity_recall`) for cheap CI smoke tests.

### 8.3 Ragas Weaknesses (Honest)

- **LLM-judge cost:** most metrics burn multiple judge calls per sample; full eval runs are the most expensive OSS option per metric-batch after you add statement-level checks (§4.4).
- **Judge variability:** scores depend on judge model, temperature, and prompt version; non-determinism means you must calibrate and repeat runs before trusting thresholds (§4.5).
- **RAG-only focus:** little value for non-RAG apps (plain chatbots, codegen, summarisation) — the general-purpose metrics are thin compared to DeepEval's 50+.
- **No built-in tracing/dashboard:** scores without traces are hard to debug; you need LangSmith/Langfuse/TruLens or your own observability to see *why* a score dropped.
- **No built-in CI runner** (you own the gate code, §6.5 — DeepEval's pytest flow is more turnkey) **and API churn between 0.1 and 0.2** (0.1 tutorials like `TestsetGenerator.with_openai()` no longer match the 0.2 `ragas.testset` surface — check docs for your installed version).

### 8.4 When to Pick Which

| Situation | Pick |
|-----------|------|
| RAG metric depth, benchmark reporting, synthetic eval sets, library-only footprint | **Ragas** |
| RAG triad + per-record traces + local dashboard, vendor-neutral on-prem eval | **TruLens** ([trulens_guide.md](trulens_guide.md)) |
| Pytest-native CI culture, broad general-purpose metric library, minimal ceremony | **DeepEval** |
| Already on LangSmith; want managed datasets + tracing + Ragas metrics in one place | **LangSmith with the RAGAS evaluator** |
| Production drift monitoring of an LLM app | pair any of the above with [ai_agent_drift_guide.md](ai_agent_drift_guide.md) patterns |

A common bank pattern: **Ragas for metric depth and synthetic test sets + LangSmith (or Langfuse) for tracing**, with the eval spec recorded for validation evidence (see [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md) and [implementing-responsible-ai.md](implementing-responsible-ai.md)).

---

## 9. Worked Example — Evaluating a Banking RAG Assistant

### 9.1 Scenario (Same as the TruLens Guide)

Crédit Agricole CIB's trade-finance desk wants a **product-support RAG assistant**: front-office users ask about trade-finance products (standby LCs, documentary credits, guarantees, FX forwards, commodity finance) and the assistant answers from the **product documentation corpus** (term sheets, operational handbooks, regulatory summaries), with answers that must be **accurate**, **grounded**, and **relevant**. Same scenario as [trulens_guide.md](trulens_guide.md) §9 — here with Ragas, complementing the TruLens triad+traces view with deeper metrics and synthetic test data.

### 9.2 Setup — Testset

Two sources of evaluation data:

1. **Goldens (primary, for release gates):** ~150 human-reviewed Q&A pairs covering the product lines (fees, limits, eligibility, documentation, regulatory notes) with verified reference answers, version-controlled.
2. **Synthetic (coverage + dev iteration):** `TestsetGenerator` (§5) adds ~300 questions from term sheets and handbooks, weighted toward `reasoning`/`multi_context`; a 10–20% human-review sample catches generator artifacts (§5.4).

### 9.3 Metrics

```python
from ragas.metrics import (
    faithfulness,          # hallucination gate — critical for a bank
    context_precision,     # retrieval ranking quality
    context_recall,        # retrieval completeness (needs ground_truth)
    answer_relevancy,      # off-topic answers
    # answer_correctness   # add for golden-set sign-off runs
)
```

The metric rationale mirrors the RAG triad ([trulens_guide.md](trulens_guide.md) §2): `faithfulness` ↔ groundedness (hallucination), `context_precision`/`context_recall` ↔ context relevance (retrieval), `answer_relevancy` ↔ answer relevance. Ragas adds the retrieval split (precision vs recall) that the triad collapses — useful here because retrieval fixes differ: *rerank* fixes precision; *chunking/index* fixes recall.

### 9.4 Evaluation

```python
from datasets import Dataset
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

# For each golden: run the assistant; capture answer + retrieved contexts (in rank order — the component-level contract)
eval_dataset = Dataset.from_dict({
    "question": questions,          # 150 goldens
    "answer": assistant_answers,    # produced by the RAG app under test
    "contexts": retrieved_chunks,   # list[list[str]]
    "ground_truth": goldens,        # human-reviewed references
})

result = evaluate(dataset=eval_dataset,
                  metrics=[faithfulness, answer_relevancy, context_precision, context_recall])
print(result)
# e.g. {'faithfulness': 0.91, 'answer_relevancy': 0.82, 'context_precision': 0.64, 'context_recall': 0.88}
```

### 9.5 Analysis — The Retrieval Fix

The scores tell a classic story: **faithfulness 0.91 (good), context_recall 0.88 (good), context_precision 0.64 (poor)**. Diagnosis: retrieval is *complete* (the right info is in the window) but *noisy* (relevant chunks buried under irrelevant ones — the average-precision structure of §3.2 punishes this). The LLM is forced to wade through noise, which risks future hallucination and degrades the user experience even when the final answer happens to be right.

Retrieval fixes to try (see [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) and [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md)):

1. **Add a reranker** (cross-encoder or LLM reranker) over the top-K candidates — directly targets ranking quality;
2. **Tune chunking** (smaller, semantic chunks; heading-aware splits for term sheets) so signal isn't diluted;
3. **Improve the query** (hybrid search with BM25 + embeddings, query rewriting — see [query_rewriting_rag_guide.md](query_rewriting_rag_guide.md));
4. **Re-weight the embedding** model choice on the banking corpus.

Re-run the eval after each change and compare `context_precision` (and re-check `faithfulness` — it must not regress when the window contents change). Gate on all four metrics, not one (§6.5).

### 9.6 The Banking Context — Evaluation as Validation Evidence

For a bank, this eval run is not just a dev nicety — it is **validation evidence**. The eval specification (dataset version, metric list, judge model + prompt versions, thresholds, and the per-sample scores) should be recorded in the model-validation file, alongside the human-review record of the golden set and the judge-calibration results (kappa vs human labels, §4.5). This is exactly the "evaluation as validation" framing in [llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md), with the governance overlay from [implementing-responsible-ai.md](implementing-responsible-ai.md): automated Ragas gates for continuous monitoring, human validation for formal sign-off. Ragas' research-grounded metric definitions (EACL 2024 paper) make the methodology easier to defend than ad-hoc rubric prompts — a real advantage when a risk or audit function asks *"how do you know the assistant doesn't hallucinate fees?"*

---

## 10. Summary — Ragas in One Page

**Ragas (RAG Assessment)** is the open-source, Apache-2.0 Python framework for evaluating RAG pipelines — introduced by the paper *"Ragas: Automated Evaluation of Retrieval Augmented Generation"* (arXiv:2309.15217, EACL 2024) and built by the explodinggradients/Vibrant Labs team (repo now `vibrantlabsai/ragas`, docs at docs.ragas.io). Its purpose: replace "vibe checks" with **systematic, reproducible evaluation loops** — and its design philosophy is **metric-first**: one metric per RAG failure mode, research-grounded and (per the paper) reference-free by default.

**The core is three things working together:**

1. **Metrics** — the deepest RAG-specific catalogue in OSS: retrieval (`context_precision`, `context_recall`, `context_entity_recall`), generation (`faithfulness`, `answer_relevancy`/`response_relevancy`, `answer_correctness`, `answer_similarity`), robustness (`noise_sensitivity`), plus aspect critiques and, in 0.2, agent and multimodal metrics. Most are LLM-as-a-judge; two (`answer_similarity`, `context_entity_recall`) are deterministic and cheap.
2. **Testset generation** — `TestsetGenerator` turns your documents into evaluation datasets (simple / reasoning / multi-context / conditional evolutions), closing the no-labelled-data gap.
3. **`evaluate()`** — one call over a HF Dataset (`question`, `answer`, `contexts`, `ground_truth`) with your metric list yields per-sample and aggregate scores, ready for CI eval gates.

**Positioning:** Ragas is the *metric-depth* option in the evaluation landscape — narrower than an observability platform (no tracing/dashboard built in), deeper than generic metric libraries. Pair it with TruLens for the triad + traces ([trulens_guide.md](trulens_guide.md)), LangSmith for managed datasets/tracing, or DeepEval for pytest-native culture ([llm_evaluation_frameworks_guide.md](llm_evaluation_frameworks_guide.md)). Its honest weaknesses: judge-LLM cost, judge variability (calibrate!), and RAG-only scope.

**Final word:** if you are building a RAG system and want to know — *quantitatively, per failure mode* — whether retrieval, grounding, and relevance are actually good, Ragas is the most direct open-source path. In a bank, run it with a Bedrock or self-hosted judge, gate CI on thresholds, and file the eval spec as validation evidence ([llm_evaluation_vs_validation_guide.md](llm_evaluation_vs_validation_guide.md)) — and you have turned "the assistant seems fine" into an auditable, reproducible claim.

---

## 11. Glossary

| Term | Definition |
|------|------------|
| **Ragas** | Open-source (Apache 2.0) Python framework for evaluating RAG pipelines; metrics + synthetic testset generation + `evaluate()`. |
| **RAG Assessment** | The expansion of the RAGAS acronym — Retrieval Augmented Generation Assessment. |
| **explodinggradients** | The original GitHub org/team that created Ragas (the team behind Vibrant Labs); the org was later renamed `vibrantlabsai`. |
| **Faithfulness** | Metric: fraction of the answer's statements supported by the retrieved context — the hallucination detector. |
| **Answer relevancy** | Metric: how well the answer addresses the question (mean cosine between the question and questions generated from the answer); renamed `response_relevancy` in 0.2 docs. |
| **Answer correctness** | Metric: factual agreement with a ground-truth answer — TP/(TP+FP+FN) blended with semantic similarity. |
| **Answer similarity** | Metric: embedding-based cosine similarity between answer and ground truth; deterministic, no judge LLM. |
| **Context precision** | Metric: whether the relevant retrieved chunks are ranked high (average precision over relevant chunks). |
| **Context recall** | Metric: fraction of the ground truth attributable to the retrieved context (retrieval completeness). |
| **Context entity recall** | Metric: fraction of ground-truth named entities present in the retrieved context (NER-based, deterministic). |
| **Noise sensitivity** | Metric: how much irrelevant context degrades answer quality (robustness). |
| **Reference-based** | Metrics that require a `ground_truth` reference answer (e.g., `context_recall`, `answer_correctness`). |
| **Reference-free** | Metrics that need no gold answer (e.g., `faithfulness`, `answer_relevancy`, `context_precision`) — Ragas' core design principle. |
| **LLM-as-a-judge** | Using an LLM as the scorer of an evaluation (statement extraction, relevance labelling, correctness comparison). |
| **Critic** | The judge LLM used by evaluation metrics; Ragas also publishes fine-tuned critic LLMs on Hugging Face. |
| **TestsetGenerator** | Ragas' synthetic evaluation-data generator: documents → questions (+reference answers) via evolution operators. |
| **Synthetic testset** | An evaluation dataset generated from documents rather than hand-labelled. |
| **evaluate()** | Ragas' core API: `evaluate(dataset, metrics=[...])` → per-sample + aggregated scores. |
| **Dataset** | The Hugging Face `Dataset` consumed by `evaluate()` with columns `question`, `answer`, `contexts`, `ground_truth`. |
| **Ground truth** | The human/reference answer used by reference-based metrics; also the "correct answer" column. |
| **Goldens** | A curated, human-reviewed evaluation set with verified reference answers — the authoritative subset for gates/sign-off. |
| **Eval gate** | A CI check that fails the build when a metric drops below a threshold. |
| **Threshold** | The score cut-off that defines a passing evaluation (calibrated per corpus, e.g., faithfulness ≥ 0.85). |
| **LangChain** | LLM-application framework; Ragas integrates with it for document loading and judge models. |
| **LlamaIndex** | Data/RAG framework; Ragas integrates with it for testset generation and judges. |
| **HuggingFace** | Hub + `datasets` library; Ragas' dataset type and its published critic models live here. |
| **LangSmith** | LangChain's commercial observability/eval platform; ships RAGAS as a built-in evaluator. |
| **TruLens** | OSS (MIT) eval + tracing + dashboard library; the RAG-triad tool — see [trulens_guide.md](trulens_guide.md). |
| **DeepEval** | OSS (Apache 2.0) pytest-native LLM evaluation library with 50+ metrics and a native RagasScore integration. |
| **Apache 2.0** | Permissive open-source licence used by Ragas (and DeepEval/Phoenix); commercial-use friendly. |
| **Retrieval** | The RAG stage that fetches relevant chunks from the knowledge base (index/embedding/hybrid search). |
| **Generation** | The RAG stage where the LLM produces the answer from the retrieved context. |
| **Hallucination** | The generation failure where the answer asserts facts not supported by the context. |
| **Prompt** | The instruction template; in Ragas, each metric has an inspectable prompt that drives the LLM judge. |
| **Calibration** | The practice of checking judge scores against human labels (agreement stats) and adjusting thresholds/judges before trusting eval numbers. |

---

*Companion guides: [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) (master tooling comparison — RAGAS §8, DeepEval §5, LangSmith §6) · [TruLens](trulens_guide.md) (the triad + dashboard sibling deep-dive) · [LLM Evaluation vs Validation](llm_evaluation_vs_validation_guide.md) (eval as validation evidence) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) & [RAG Optimization Techniques](rag_optimization_techniques_guide.md) & [Beyond RAG](beyond_rag_guide.md) & [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) (the RAG series) · [Vector Databases](vector_databases_guide.md) (the retrieval layer) · [AI Agent Drift](ai_agent_drift_guide.md) (production monitoring) · [Autonomous Agents](autonomous_agents_guide.md) (§5 agent evals) · [Implementing Responsible AI](implementing-responsible-ai.md) (governance overlay) · [LLM Guard Models](llm_guard_models_guide.md) (safety layers).*
