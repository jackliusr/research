# RAG Evaluation Tools — A Head-to-Head Comparison Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) · [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) · [Ragas](ragas_guide.md) · [TruLens](trulens_guide.md) · [LLM Evaluation vs Validation](../llm_evaluation_vs_validation_guide.md) · [AI Agent Drift](../ai_agent_drift_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [Beyond RAG](beyond_rag_guide.md) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG Optimization Techniques](rag_optimization_techniques_guide.md) · [Vector Databases](vector_databases_guide.md) · [Autonomous Agents](../autonomous_agents_guide.md) · [Responsible AI](../implementing-responsible-ai.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [The Tool Landscape](#1-the-tool-landscape)
2. [RAGAS — The Metrics-First Framework](#2-ragas--the-metrics-first-framework)
3. [TruLens — The RAG Triad and Dashboard](#3-trulens--the-rag-triad-and-dashboard)
4. [DeepEval — The Test-Driven Framework](#4-deepeval--the-test-driven-framework)
5. [LangSmith — The LangChain Platform](#5-langsmith--the-langchain-platform)
6. [Arize Phoenix — The Open-Source Observability Platform](#6-arize-phoenix--the-open-source-observability-platform)
7. [The Others — Weave, MLflow, Langfuse, G-Eval](#7-the-others--weave-mlflow-langfuse-g-eval)
8. [The Head-to-Head Comparison](#8-the-head-to-head-comparison)
9. [Tool Selection Guidance](#9-tool-selection-guidance)
10. [Worked Example — Selecting Tools for a Banking RAG](#10-worked-example--selecting-tools-for-a-banking-rag)
11. [Summary — RAG Eval Tools in One Page](#11-summary--rag-eval-tools-in-one-page)
12. [Glossary](#12-glossary)

---

## 1. The Tool Landscape

### 1.1 What This Guide Is

This guide is the **head-to-head comparison of the RAG-specific evaluation tools**: the tools whose primary job is to measure whether a retrieval-augmented generation (RAG) system is working — did it retrieve the right context, and did it answer from that context rather than from thin air.

It sits inside a small family of evaluation guides in this series:

| Guide | Scope |
|-------|-------|
| **This guide** | The **tool head-to-head**: RAGAS vs TruLens vs DeepEval vs LangSmith vs Phoenix vs Weave vs MLflow vs Langfuse vs G-Eval — capabilities, comparison dimensions, selection guidance, worked selection example |
| [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) | The **tool-agnostic methodology playbook**: how to design an evaluation for a RAG system (two-layer eval, datasets, thresholds, gates) — the "what to measure and how" that any of these tools implements |
| [ragas_guide.md](ragas_guide.md) | The full deep-dive on RAGAS (metrics, judge setup, testset generation) |
| [trulens_guide.md](trulens_guide.md) | The full deep-dive on TruLens (RAG triad, feedback functions, dashboard) |
| [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) | The **master tooling comparison** covering all the generic LLM-eval tooling (DeepEval §5, LangSmith §6, MLflow §7, RAGAS §8, Arize §9, W&B §10, G-Eval §11) |
| [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) | The two tracks: continuous *evaluation* vs formal *validation* — and how tooling maps onto each |

**The division of labour:** the master [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) covers evaluation tooling broadly; this guide goes **deeper on the RAG-specific evaluation angle** — the RAG metrics each tool ships, the RAG-focused workflows (retrieval+generation scoring, testset generation, RAG tracing), and the RAG-specific selection logic. Where a topic is already covered in depth elsewhere, this guide says so and cross-references rather than duplicating.

> **How to use this guide:** read §1–§7 to understand each tool's RAG capabilities, §8 for the master comparison, §9 for selection guidance, and §10 for a worked selection example end-to-end. If you only have five minutes, read §1.5 (the landscape table), §8.1 (the master comparison table), and §11 (the one-page summary).

### 1.2 The RAG-Specific Evaluation Tools

Nine tools matter for RAG evaluation in 2026. They fall into a spectrum from *pure metrics libraries* (measure, give you numbers) to *full observability platforms* (trace every call, attach scores, monitor production):

1. **RAGAS** — the open-source, metrics-first framework; the deepest catalogue of RAG-specific metrics (see [ragas_guide.md](ragas_guide.md)).
2. **TruLens** — the open-source evaluate+trace library that popularised the **RAG triad**; ships its own dashboard (see [trulens_guide.md](trulens_guide.md)).
3. **DeepEval** — the open-source, **pytest-style test-driven** framework with RAG metrics and native RAGAS integration.
4. **LangSmith** — LangChain's commercial platform: tracing, datasets, experiments, **online evaluators**, production monitoring.
5. **Arize Phoenix** — the open-source LLM observability platform (tracing + evals + embedding analysis); Arize AX is its managed cloud sibling.
6. **W&B Weave** — Weights & Biases' LLM toolkit: tracing, the **Weave evals** framework (datasets + scorers), experiment tracking.
7. **MLflow** — the OSS MLOps platform; `mlflow.evaluate()` gained LLM/RAG evaluation over the generic ML tracking core.
8. **Langfuse** — the MIT-licensed tracing/eval platform; self-hostable, with LLM-as-a-judge scoring, datasets, and experiments.
9. **G-Eval** — not a tool but a *technique*: the chain-of-thought LLM-as-a-judge framework (arXiv:2303.16634) that other tools implement (DeepEval's `GEval` metric being the canonical implementation).

### 1.3 The Categories

Four categories help structure the comparison:

| Category | What it is | Tools in this guide |
|----------|-----------|---------------------|
| **Metrics frameworks** | Libraries that compute evaluation scores (usually LLM-as-a-judge) over datasets you provide; no tracing, no dashboard | RAGAS, G-Eval (as a metric technique), partly DeepEval |
| **Observability platforms** | Trace every LLM call/retrieval/tool step, store it, and attach evaluation scores to records; dashboard + production monitoring | Arize Phoenix, Langfuse, LangSmith, W&B Weave, (TruLens — the "evaluate + trace" hybrid) |
| **Test-driven frameworks** | Turn evaluation into assertions that fail a build/CI run; thresholds + pass/fail contracts, pytest-style | DeepEval (canonical), plus eval-gate patterns in RAGAS/TruLens workflows |
| **Tracing backends** | Capture and visualise execution traces (spans, latency, cost); evaluation is layered on top | Langfuse, LangSmith, Phoenix, Weave |

The categories overlap deliberately: LangSmith, Phoenix, Langfuse, and Weave are all "observability platforms with evaluation on top", while RAGAS and DeepEval are "evaluation first" and bolt onto a tracing backend when you need one. **A common architecture is to pair a metrics framework (RAGAS or DeepEval) with a tracing platform (Langfuse or Phoenix)** — the metrics for depth, the platform for records and monitoring. §9 and §10 develop this pattern.

### 1.4 Why RAG-Specific Evaluation Tools Exist

Generic LLM evaluation asks: *is this output good?* RAG evaluation asks two separable questions, because a RAG system has two distinct stages that fail in distinct ways:

1. **The retrieval layer** — did we fetch the right documents (precision: relevant chunks ranked high) and all of them (recall: nothing relevant missing)?
2. **The generation layer** — is the answer faithful to the retrieved context (no hallucination) and relevant to the user's question?

This is the **two-layer evaluation** framework developed in [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md): score retrieval and generation *separately*, because the fixes are different — a reranker fixes precision, chunking/index changes fix recall, prompt/grounding changes fix faithfulness. Generic evaluation tools (e.g., plain ROUGE/BLEU on the answer) collapse these layers into one number and tell you nothing about *where* the RAG pipeline broke.

The RAG-specific tools in this guide exist precisely to make the two layers measurable:

- **Reference-free by design** — most RAG metrics score retrieval quality and grounding without needing a human-written gold answer per question (you rarely have one for open-ended questions).
- **RAG-native metrics** — faithfulness, context precision/recall, contextual relevancy, answer relevancy — each mapped to a specific RAG failure mode.
- **RAG-aware tracing** — retrieval spans (query, top-k chunks, scores) are first-class citizens, so a bad score can be traced to the exact retrieved context that caused it.

> **Cross-reference:** the methodology guide ([rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md)) defines the two-layer evaluation design, dataset construction, thresholds, and eval-gate mechanics that this guide's tools implement. Read it first if you want the "what to measure" before the "which tool".

### 1.5 The Landscape Table — Tools at a Glance

| Tool | Category | Open-source / Hosted | RAG focus |
|------|----------|----------------------|-----------|
| **RAGAS** | Metrics framework | OSS (Apache 2.0) — runs in your process | Deepest RAG metric catalogue (retrieval + generation) + testset generation |
| **TruLens** | Metrics + tracing + dashboard | OSS (MIT) — self-hosted dashboard | The RAG triad (context relevance, groundedness, answer relevance) + per-record traces |
| **DeepEval** | Test-driven metrics framework | OSS (Apache 2.0); Confident AI cloud optional | RAG metrics (faithfulness, answer/contextual relevancy, contextual precision/recall) as pytest-style assertions |
| **LangSmith** | Observability + eval platform | Hosted SaaS (closed source); free tier | Dataset-based RAG eval (answer relevance, accuracy, retrieval quality) + online evaluators on live traffic |
| **Arize Phoenix** | Observability platform | OSS (Apache 2.0) self-host; Arize AX cloud | RAG evals (hallucination, QA correctness, context relevance) attached to OpenTelemetry traces |
| **W&B Weave** | Observability + eval platform | Hosted SaaS (closed source); free tier | Weave evals (datasets + scorers) with RAGAS integration; trace + score RAG chains |
| **MLflow** | MLOps platform + LLM eval | OSS (Apache 2.0) self-host; Databricks | `mlflow.evaluate()` with question-answering/RAG task support, LLM-as-judge, ROUGE/BLEU |
| **Langfuse** | Tracing + eval platform | OSS (MIT) self-host *or* Langfuse Cloud | Trace retrieval spans, score with LLM-as-a-judge, datasets + experiments for RAG regressions |
| **G-Eval** | Metric technique (LLM-judge) | OSS technique; implemented in DeepEval etc. | Chain-of-thought judge for custom criteria (e.g., banking-specific answer quality) |

> **Reading the table:** "OSS" means the tool runs in your own environment (data stays in-house — often decisive for banks, see §9.3); "Hosted SaaS" means your traces/eval data live on the vendor's platform. RAGAS, TruLens, DeepEval, MLflow, Phoenix, and Langfuse are all self-hostable; LangSmith and Weave are not.

---

## 2. RAGAS — The Metrics-First Framework

### 2.1 Overview

**RAGAS** (Retrieval Augmented Generation Assessment) is the open-source (Apache 2.0) Python framework that is the **reference implementation of metrics-first RAG evaluation**: one purpose-built metric per RAG failure mode, research-grounded in the paper *"Ragas: Automated Evaluation of Retrieval Augmented Generation"* (arXiv:2309.15217, EACL 2024), built by the Vibrant Labs / explodinggradients team (repo `vibrantlabsai/ragas`, docs at docs.ragas.io).

RAGAS is deliberately **narrower than an observability platform and deeper than a generic metric library**: it measures, it does not trace or dashboard. Its full deep-dive lives in [ragas_guide.md](ragas_guide.md) — this section summarises the RAG capabilities that matter for the head-to-head.

### 2.2 The RAG Metrics

RAGAS splits its catalogue along the **two-layer evaluation** line of §1.4 (see [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §2 for the framework):

**Retrieval layer (component-level):**

| Metric | What it measures | Reference-based? |
|--------|------------------|------------------|
| `context_precision` | Are the *relevant* chunks ranked high in the retrieved window? (ranking quality) | No (needs retrieved context + question) |
| `context_recall` | Did we retrieve *all* the relevant context? (completeness) | Yes — needs ground truth context |
| `context_entity_recall` | Entity-level retrieval completeness (cheap, deterministic) | Yes |

**Generation layer (end-to-end):**

| Metric | What it measures | Reference-based? |
|--------|------------------|------------------|
| `faithfulness` | Is the answer *grounded* in the retrieved context (no hallucination)? | No |
| `answer_relevancy` (renamed `response_relevancy` in the 0.2.x docs) | Does the answer address the question (not off-topic)? | No |
| `answer_correctness` | Is the answer factually correct vs the reference answer? | Yes |
| `answer_similarity` | Semantic similarity to the reference (embedding-based, cheap) | Yes |

Plus, on the 0.2.x line: **aspect critiques** (harmfulness, coherence, conciseness via natural-language aspects), robustness metrics (e.g., `noise_sensitivity`), agentic metrics (tool-call faithfulness, action correctness), and multimodal metrics. The faithfulness ↔ groundedness mapping and the retrieval precision-vs-recall split are the two design decisions that make RAGAS the natural "deep metrics" choice (see §8.2).

> **Verification note:** metric names and semantics verified against [ragas_guide.md](ragas_guide.md) §2–§3 (the dedicated deep-dive) — including the 0.2.x rename of `answer_relevancy` → `response_relevancy` (old imports kept as aliases).

### 2.3 Testset Generation

RAGAS ships **`TestsetGenerator`** — synthetic evaluation data from *your own documents*, so you can evaluate a RAG system before any human has written a single golden Q&A:

- **0.1.x style:** evolution-based generation — simple / reasoning / multi-context / conditional questions derived from document chunks.
- **0.2.x style:** rebuilt `ragas.testset` module — document-parser/node-based generation with a knowledge-graph step; better coverage control and fewer artifacts.

Practical pattern (from [ragas_guide.md](ragas_guide.md) §9): ~150 human-reviewed goldens for release gates + ~300 synthetic questions for coverage and dev iteration, with a 10–20% human-review sample of the synthetic set to catch generator artifacts.

### 2.4 Strengths

1. **Metric depth** — the deepest, most RAG-specific metric catalogue in OSS; every metric maps to a concrete failure mode and a concrete fix (rerank vs chunk vs prompt).
2. **Research-grounded** — published, peer-reviewed metric definitions (EACL 2024); defensible methodology when risk/audit asks *"how do you know it doesn't hallucinate fees?"* (see [implementing-responsible-ai.md](../implementing-responsible-ai.md)).
3. **Reference-free by default** — usable when no gold answers exist.
4. **Testset generation** — `TestsetGenerator` turns your corpus into an eval dataset with no manual labelling.
5. **Integrations** — runs with LangChain, LlamaIndex, and can send results to LangSmith / Langfuse for monitoring (§9.4's combo pattern).

### 2.5 Weaknesses

1. **LLM cost** — most metrics are LLM-as-a-judge: 1,000 questions × 5 metrics ≈ 5,000+ judge calls per run; budgeting and judge caching matter (see [ragas_guide.md](ragas_guide.md) §4).
2. **Complexity** — judge model/prompt setup, metric choice, and threshold calibration require methodology knowledge; not a zero-config tool.
3. **No tracing/dashboard/CI runner built in** — it is a library; you pair it with Langfuse/Phoenix/LangSmith for records and with your own CI job for gates (unlike DeepEval's pytest runner, §4.3).
4. **RAG-only** — not useful for non-RAG apps (chatbots without retrieval, content generation).

### 2.6 RAGAS at a Glance

| Dimension | RAGAS |
|-----------|-------|
| Category | Metrics framework |
| License / hosting | Apache 2.0, OSS, runs in-process |
| RAG metrics | Faithfulness, context precision/recall (+entity recall), answer relevancy/relevancy, answer correctness/similarity, aspect critiques, robustness, agent metrics |
| Reference-based metrics | Yes (correctness, recall, similarity) |
| LLM-as-a-judge | Yes (most metrics); deterministic variants for similarity/entity recall |
| Testset generation | ✅ `TestsetGenerator` (evolution-based / node-based in 0.2) |
| Tracing / dashboard | ❌ (integrate with LangSmith, Langfuse, Phoenix) |
| Cost | Free library; pay for judge-LLM calls |
| Best for | Deep retrieval + generation metrics, RAG architecture comparison, synthetic test data |

---

## 3. TruLens — The RAG Triad and Dashboard

### 3.1 Overview

**TruLens** (truera/trulens, MIT license, stewarded by Snowflake since TruEra's 2025 acquisition) is the open-source library that popularised the **RAG triad** and combines **evaluation + tracing + a dashboard** in one package: instrument any app with a decorator, score every step with feedback functions, and inspect everything in a local Streamlit dashboard. Its full deep-dive is [trulens_guide.md](trulens_guide.md); this section covers the RAG capabilities for the head-to-head.

TruLens' positioning — "evaluate + trace" — means every evaluation is attached to a concrete, inspectable record of what the app actually did, which distinguishes it from pure metrics libraries (RAGAS) and pure tracing backends (Langfuse).

### 3.2 The RAG Triad

The **RAG triad** is TruLens' core contribution to RAG evaluation: three feedback functions that cover the two-layer evaluation of §1.4 with exactly one metric per failure mode:

| Triad leg | Feedback function | Layer | Failure mode caught |
|-----------|-------------------|-------|---------------------|
| **Context relevance** | `context_relevance` | Retrieval | Irrelevant context retrieved |
| **Groundedness** | `groundedness` | Generation | Answer not supported by the context (hallucination) |
| **Answer relevance** | `answer_relevance` | Generation | Answer doesn't address the question |

The triad maps directly onto the RAGAS catalogue (faithfulness ↔ groundedness, context precision/recall ↔ context relevance split into two, answer relevancy ↔ answer relevance) — the mapping is worked through in [ragas_guide.md](ragas_guide.md) §9.3 and [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md). TruLens collapses retrieval into one "context relevance" score; RAGAS splits it into precision vs recall. That is the single most useful difference between the two when choosing (§8.2).

### 3.3 Feedback Functions

Feedback functions are the building blocks: programmatic (deterministic) or **LLM-as-a-judge** functions that score each record. Beyond the triad, TruLens ships feedback for correctness, relevance, toxicity, moderation, and lets you define custom functions (including calling any model — OpenAI, Bedrock, local). Feedback functions can run **inline** (during app execution) or **offline** (over stored records), which is what makes TruLens usable both in dev notebooks and as a batch evaluation pass.

### 3.4 The Dashboard

TruLens' Streamlit dashboard provides: an app **leaderboard** (compare app versions by triad scores), **per-record traces** (click a low score → see the retrieved chunks and prompt), **score time series**, and cost/latency per step. It runs locally — data never leaves your environment except judge-API calls — which matters for banking data residency (see [trulens_guide.md](trulens_guide.md) §1.3).

### 3.5 Strengths

1. **Triad simplicity** — three well-explained metrics cover the essential RAG failure modes; fastest on-ramp to a meaningful RAG evaluation (§9.3's "evaluate quickly" scenario).
2. **Open source (MIT)** — permissive, self-hosted dashboard, no vendor lock-in; OpenTelemetry-native since the 1.x line (export traces to any OTLP backend).
3. **Evaluate + trace integrated** — scores are attached to inspectable records, so a bad groundedness score leads you to the exact context that caused it.
4. **Works with any framework** — decorator-based instrumentation for LangChain, LlamaIndex, and raw Python.

### 3.6 Weaknesses

1. **Scale** — the local dashboard and in-process storage are dev-oriented; heavy production traffic needs the managed tier or another backend for long-term storage.
2. **Maturity** — the project went through the TruEra → Snowflake transition; API evolution (e.g., the `trulens_eval` → `trulens` 1.x rewrite) means older tutorials are stale.
3. **Metric depth** — the triad is shallow compared with RAGAS' retrieval split and aspect critiques; deep retrieval debugging (precision vs recall) needs RAGAS (§8.2).
4. **LLM-judge cost** — triad feedback functions are LLM calls per record.

### 3.7 TruLens at a Glance

| Dimension | TruLens |
|-----------|---------|
| Category | Metrics + tracing + dashboard (evaluate + trace hybrid) |
| License / hosting | MIT, OSS, self-hosted dashboard |
| RAG metrics | RAG triad: context relevance, groundedness, answer relevance (+ custom feedback functions) |
| Reference-based metrics | Yes (correctness-style feedback, custom) |
| LLM-as-a-judge | Yes (most feedback functions) |
| Testset generation | ❌ (bring your own dataset) |
| Tracing / dashboard | ✅ Built-in: OpenTelemetry tracing + Streamlit dashboard |
| Cost | Free library; pay for judge-LLM calls |
| Best for | Quick triad-based evaluation with per-record traceability, small-to-mid teams, data-residency-sensitive setups |

---

## 4. DeepEval — The Test-Driven Framework

### 4.1 Overview

**DeepEval** is the open-source (Apache 2.0, Python) LLM evaluation framework by **Confident AI** — the one that brought **pytest-style, test-driven development to LLM applications**: metrics return scores *and* reasoning, and `assert_test()` raises on threshold breach with the same contract as a normal pytest assertion. It is the reference implementation of the "test-driven" category from §1.3.

- **Repo:** `confident-ai/deepeval` · **Docs:** deepeval.com · **Cloud layer (optional):** Confident AI (dashboards, collaboration, production monitoring).
- It is also covered at platform level in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §5; this section focuses on the RAG-specific angle.

### 4.2 The RAG Metrics

DeepEval ships a comprehensive RAG metric family (verified against the current docs and [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §5.2):

| Metric class | Metrics | Layer |
|--------------|---------|-------|
| **Answer relevancy** | `AnswerRelevancyMetric` | Generation — does the answer address the question? |
| **Faithfulness** | `FaithfulnessMetric` | Generation — is the answer grounded in the retrieved context? |
| **Contextual relevancy** | `ContextualRelevancyMetric` | Retrieval — is the retrieved context relevant to the question? |
| **Contextual precision / recall** | `ContextualPrecisionMetric`, `ContextualRecallMetric` | Retrieval — ranking quality / completeness |
| **Hallucination** | `HallucinationMetric` | Generation — claims unsupported by context |
| **RAGAS score** | `RagasScore` (native RAGAS integration) | All layers — run RAGAS metrics inside DeepEval |

The RAG metric set deliberately mirrors the two-layer framework of §1.4 (see [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md)): retrieval metrics (contextual relevancy/precision/recall) and generation metrics (answer relevancy, faithfulness, hallucination). The native **RagasScore** integration means teams that want RAGAS' metric definitions with DeepEval's test-runner plumbing can have both without running two frameworks.

> **Verification note:** the metric list (including `ContextualRelevancyMetric` and the `RagasScore` wrapper) is verified against DeepEval's RAG evaluation quickstart and the master guide §5.2. DeepEval moved off the historic 0.x line — check the current PyPI version before formal use; the API surface described here (`assert_test`, metric classes) is stable across recent releases.

### 4.3 The Test-Driven Workflow

The core contract is a pytest assertion:

```python
from deepeval import assert_test
from deepeval.metrics import FaithfulnessMetric
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input="What are the eligibility criteria for a standby LC?",
    actual_output="Eligibility requires an approved credit line...",
    retrieval_context=["Standby LC: credit-line approval required"])
assert_test(test_case, [FaithfulnessMetric(threshold=0.7)])
```

- Deterministic pass/fail with configurable **thresholds** and clear failure messages.
- **Parametrised test cases, fixtures**, and native pytest integration (`pytest test_llm.py` or `deepeval test run test_llm.py`).
- **CI-friendly by construction** — the exit code is the eval gate: a failing assertion fails the build (§9.3's "CI test-driven" scenario).
- **`Synthesizer`** generates test cases (input, expected output, context) from your documents for dataset creation.

### 4.4 Strengths

1. **Test-driven** — evaluation as assertions with thresholds; the natural fit for eval-gate/CI workflows (see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) §2 for the eval-track framing).
2. **CI-friendly** — pytest-native, CLI test runner, GitHub-Actions-ready; gates on every PR.
3. **Comprehensive RAG metrics** — faithfulness, answer relevancy, contextual relevancy/precision/recall, hallucination, plus native RAGAS scores.
4. **Model-agnostic judge** — any LLM as judge (OpenAI, Anthropic, local via Ollama/vLLM).
5. **50+ metrics overall** — quality (G-Eval, summarisation), safety (toxicity, bias, jailbreak), conversation/agent metrics beyond RAG.

### 4.5 Weaknesses

1. **Smaller ecosystem** — younger community than LangSmith/MLflow; fewer tutorials, less Stack Overflow coverage; docs can lag features.
2. **Python-only** — no TypeScript/JS SDK.
3. **LLM-judge cost** — advanced RAG metrics are judge calls per assertion.
4. **Production monitoring needs the paid layer** — OSS handles offline test-driven eval; dashboards/collaboration/monitoring live in Confident AI cloud.

### 4.6 DeepEval at a Glance

| Dimension | DeepEval |
|-----------|----------|
| Category | Test-driven metrics framework |
| License / hosting | Apache 2.0, OSS; Confident AI cloud optional |
| RAG metrics | Answer relevancy, faithfulness, contextual relevancy, contextual precision/recall, hallucination, RagasScore |
| Reference-based metrics | Yes (correctness-style, custom) |
| LLM-as-a-judge | Yes (+ G-Eval CoT metric, §7.4) |
| Testset generation | ✅ `Synthesizer` |
| Tracing / dashboard | ❌ in OSS (Confident AI cloud provides dashboards/monitoring) |
| Cost | Free library; pay for judge-LLM calls; paid cloud optional |
| Best for | CI eval gates, pytest-native teams, evaluation-driven development, RAG regressions on every PR |

## 5. LangSmith — The LangChain Platform

### 5.1 Overview

**LangSmith** is LangChain's commercial **LLM evaluation and observability platform** (SaaS, Python + TypeScript): tracing, datasets, experiments, evaluators, human annotation, and production monitoring in one hosted product. It is the reference implementation of the "integrated tracing" category — evaluation is attached to the full execution trace, so a low score is one click away from the exact retrieval step that produced it.

It is covered at platform level in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §6; this section focuses on its RAG-specific evaluation surface.

### 5.2 RAG Evaluators

LangSmith's RAG evaluation workflow is dataset-based (per the official RAG evaluation tutorial): **create a dataset of questions + expected answers → run the RAG app on those questions → score results with evaluators** for:

- **Answer relevance** — does the answer address the question?
- **Answer accuracy / correctness** — does the answer match the expected answer (LLM-as-judge or reference-based metrics like ROUGE/BLEU/embedding similarity)?
- **Retrieval quality** — are the retrieved documents relevant to the question?

Evaluators come in several flavours: **LLM-as-judge** (rubric-scored by GPT-4, Claude, or any model), **code** (Python/TS custom functions), **pairwise** (A/B comparison), and **reference-based** (ROUGE, BLEU, embedding similarity). Experiments compare runs across models/prompts/configs with side-by-side tables and statistical significance testing. LangSmith can also run **RAGAS metrics** on your datasets, and RAGAS documents LangSmith as its monitoring integration — a natural bridge between the metrics-first and platform worlds.

### 5.3 Online Evaluators (Production)

Since 2025 LangSmith has shipped **online evaluators**: run-level, LLM-as-judge evaluators that score **live production traces** as they happen. Because production traffic has no reference answers, these are **reference-free** evaluations — the judge grades each trace against a rubric (internal consistency, clarity, completeness, policy adherence). Online evaluators plug directly into monitoring dashboards with alerting, drift detection, and cost/latency tracking — this is LangSmith's answer to the "production monitoring" requirement (§9.3).

### 5.4 Strengths

1. **Integrated tracing + eval** — evaluation attached to full execution traces; root-cause debugging from score to span.
2. **LangChain-native** — automatic instrumentation of LangChain chains; evaluators understand the LangChain data model.
3. **Best-in-class experiment/compare views** — A/B tests of models, prompts, retrievers with significance testing.
4. **Production monitoring** — online evaluators, drift detection, alerting, human-feedback widgets (thumbs up/down linked to traces).
5. **Multi-language** — Python + TypeScript SDKs.

### 5.5 Weaknesses

1. **LangChain-centric** — deepest integration only within the LangChain ecosystem; less natural for raw Python or other frameworks.
2. **Hosted only** — no self-hosted option; traces and eval data live in LangSmith's cloud (data-residency blocker for some banks — see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) §3 on validation evidence location).
3. **Cost at scale** — usage-based pricing (see §5.6); the bill grows with trace volume.
4. **Vendor lock-in** — evaluation data accumulates in the platform; fewer built-in metric definitions than RAGAS/DeepEval (you configure or write evaluators).

### 5.6 LangSmith at a Glance

| Dimension | LangSmith |
|-----------|-----------|
| Category | Observability + eval platform (SaaS) |
| License / hosting | Closed source, hosted SaaS (no self-host) |
| RAG metrics | Answer relevance/accuracy, retrieval quality (via evaluators); RAGAS metrics supported on datasets |
| Reference-based metrics | Yes (ROUGE, BLEU, embedding similarity, correctness) |
| LLM-as-a-judge | Yes (evaluators + online evaluators) |
| Testset generation | ❌ (datasets from traces/CSV/UI; no document-to-question generator) |
| Tracing / dashboard | ✅ Full tracing, experiments, monitoring dashboards |
| Cost | Free Developer tier; Plus ≈ US$39/user/month + usage-based trace metering (LCU/LSU); Enterprise custom — **approximate, verify live pricing** |
| Best for | LangChain stacks needing tracing + eval + production monitoring in one hosted product |

---

## 6. Arize Phoenix — The Open-Source Observability Platform

### 6.1 Overview

**Arize Phoenix** is the open-source (Apache 2.0) **LLM observability platform** from Arize AI: tracing (OpenTelemetry-native), evaluation, embedding analysis, and drift monitoring in one self-hostable tool. **Arize AX** is the managed cloud sibling (free tier + usage-based) for team collaboration and large-scale monitoring. Phoenix is covered at platform level in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §9; this section covers its RAG-specific evaluation surface.

### 6.2 RAG Evals

Phoenix ships **built-in RAG evaluators** that plug into its tracing pipeline (verified against the Phoenix RAG evaluation docs and cookbooks):

| Evaluator | What it scores | Layer |
|-----------|----------------|-------|
| **Hallucination** | Does the answer contain claims unsupported by the retrieved context? | Generation |
| **QA correctness** | Is the answer correct against a reference answer? | Generation |
| **Relevance / context relevance** | Is the retrieved context relevant to the question? | Retrieval |
| **Toxicity** | Safety of the output | Safety |

They run via the `run_evals()` API over traced data (e.g., `run_evals(dataframe=..., evaluators=[hallucination_evaluator, qa_correctness_evaluator])`), with `provide_explanation=True` giving judge reasoning, and results are logged back into Phoenix attached to the traces. Eval-as-code (custom Python evaluators) and pre-built judge templates round out the set. Because evaluation results link to full execution traces, a hallucination flag opens the exact retrieval + prompt that produced it.

### 6.3 Strengths

1. **Production observability** — real-time dashboards, alerting, cost/latency tracking; mature drift detection inherited from Arize's ML-observability background (see [ai_agent_drift_guide.md](../ai_agent_drift_guide.md)).
2. **Tracing + eval integrated** — OpenTelemetry-based; scores attached to traces for root-cause analysis.
3. **Embedding analysis** — UMAP projections, clustering, outlier detection on inputs/outputs — useful for spotting retrieval-distribution shifts.
4. **Open source + self-hostable** — data stays in-house; standards-based (OTel).

### 6.4 Weaknesses

1. **Setup complexity** — self-hosting requires the Phoenix service, a database, and an OpenTelemetry collector; heavier than a pip-install metrics library.
2. **Newer LLM-specific features** — RAG/LLM evals are younger and less mature than the ML-monitoring core; fewer built-in RAG metrics than RAGAS/DeepEval (no pre-built contextual precision/recall, for instance).
3. **Advanced features in the cloud** — collaboration and large-scale hosted monitoring push you toward Arize AX.

### 6.5 Phoenix at a Glance

| Dimension | Arize Phoenix |
|-----------|---------------|
| Category | Observability platform (tracing + evals + monitoring) |
| License / hosting | Apache 2.0, OSS self-host; Arize AX cloud optional |
| RAG metrics | Hallucination, QA correctness, relevance/context relevance, toxicity (+ custom eval-as-code) |
| Reference-based metrics | Yes (QA correctness) |
| LLM-as-a-judge | Yes (pre-built templates + custom) |
| Testset generation | ❌ (bring your own eval dataset) |
| Tracing / dashboard | ✅ Full OTel tracing, dashboards, drift + embedding analysis |
| Cost | Free OSS; Arize AX free tier + usage-based — **approximate, verify live pricing** |
| Best for | Production RAG monitoring with trace-level root cause, drift detection, self-hosted data |

---

## 7. The Others — Weave, MLflow, Langfuse, G-Eval

### 7.1 W&B Weave — Evals Inside the W&B Ecosystem

**Weave** is Weights & Biases' LLM toolkit (hosted SaaS, Python + TypeScript): tracing, **Weave evals**, experiment tracking, dataset versioning, and dashboards — the natural extension for teams already on W&B. Covered at platform level in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §10.

**The Weave evals framework:** evaluation is a first-class object — you define a **dataset** (e.g., golden Q&A samples with ground-truth contexts) and **scorers** (LLM-as-judge prompts, deterministic functions, or RAGAS metrics), then call `Evaluation.evaluate(model, dataset, scorers)`; every run is logged to W&B with traces and scores. The RAG++ course materials demonstrate the canonical pattern: download a golden eval dataset, run the RAG chain through Weave tracing, score with RAGAS/LLM judges, and review in the W&B dashboard.

**Strengths:** W&B ecosystem integration (artifacts, reports, sweeps); best-in-class dataset versioning (immutable snapshots); human-annotation UI; RAGAS integration gives real RAG metrics without leaving W&B.
**Weaknesses:** hosted SaaS only (no self-host — data leaves your infrastructure); pricing scales with team size; fewer built-in RAG metrics than RAGAS/DeepEval (you bring scorers or RAGAS).

| Dimension | W&B Weave |
|-----------|-----------|
| Category | Observability + eval platform (SaaS) |
| License / hosting | Closed source, hosted SaaS |
| RAG metrics | Via scorers / RAGAS integration (no large built-in RAG set) |
| Testset generation | ❌ (dataset versioning yes; generation via RAGAS/Synthesizer) |
| Tracing / dashboard | ✅ Weave tracing + W&B dashboards |
| Cost | Free tier; Team/Enterprise paid — **approximate, verify live pricing** |
| Best for | Teams already on W&B wanting trace + eval + annotation in one place |

### 7.2 MLflow — LLM Evaluation on the MLOps Platform

**MLflow** (Apache 2.0, Python) is the OSS MLOps platform whose `mlflow.evaluate()` was extended to LLMs — **LLM evaluation as an extension of experiment tracking** rather than a dedicated product. Covered in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §7.

For RAG: `mlflow.evaluate(model=..., data=eval_df, model_type="question-answering", evaluators="default")` with column mapping for question/ground-truth; built-in metrics include ROUGE-1/2/L, BLEU, answer/sentence similarity, toxicity, and readability; **LLM-as-judge** via custom prompt-defined criteria with any model; the model-comparison UI shows side-by-side aggregates.

**Strengths:** seamless for teams already on MLflow; open source, self-hostable (data stays in-house); Databricks integration; one platform for traditional ML + LLM eval.
**Weaknesses:** **fewer RAG-specific metrics** — no built-in faithfulness, contextual precision/recall, or hallucination (you must build them as custom LLM-judge criteria); offline evaluation only, no production monitoring; LLM eval features still maturing.

| Dimension | MLflow |
|-----------|--------|
| Category | MLOps platform + LLM eval component |
| License / hosting | Apache 2.0, OSS self-host; Databricks managed |
| RAG metrics | None built-in RAG-specific (ROUGE/BLEU/similarity + custom LLM-judge criteria) |
| Testset generation | ❌ |
| Tracing / dashboard | Partial (experiment tracking + model-comparison UI; no span-level tracing) |
| Cost | Free OSS |
| Best for | MLflow-centric teams wanting basic RAG eval alongside model tracking, self-hosted |

### 7.3 Langfuse — Open-Source Tracing + Evals

**Langfuse** (MIT license) is the open-source LLM engineering platform: **tracing** (spans, latency, cost), **scores**, **datasets**, **LLM-as-a-judge evaluations**, prompt management, and experiments — self-hostable for free (Docker Compose/Helm) or used as Langfuse Cloud (subscription + usage-based events). It is the reference implementation of the "open-source tracing" category.

For RAG: every retrieval step is traced (query, top-k chunks, scores), and **LLM-as-a-judge** with rubric-guided prompts scores traces for context relevance, groundedness/faithfulness, or any custom criterion; **datasets + experiments** give a regression workflow (re-run a golden RAG dataset after a chunking/reranker change, compare scores); online evaluation scores live production traces. Because it is MIT-licensed and self-hostable, it is the standard choice when a bank wants tracing + eval with full data residency (see §9.3 and the [ai_agent_drift_guide.md](../ai_agent_drift_guide.md) production-monitoring discussion).

**Strengths:** open source + self-host (data sovereignty); framework-agnostic (LangChain, LlamaIndex, raw Python, TS); cheaper at scale than hosted rivals; good eval surface (judge prompts, datasets, experiments).
**Weaknesses:** metric depth is bring-your-own — Langfuse provides the scoring mechanism, not a deep RAG metric catalogue (pair it with RAGAS, §9.4); self-hosting adds operational load (ClickHouse-backed stack).

| Dimension | Langfuse |
|-----------|----------|
| Category | Tracing + eval platform |
| License / hosting | MIT, OSS self-host *or* Langfuse Cloud |
| RAG metrics | Mechanism, not catalogue: LLM-as-a-judge rubrics, code scorers, RAGAS integration |
| Testset generation | ❌ (dataset management yes; generation via RAGAS) |
| Tracing / dashboard | ✅ Full tracing, dashboards, prompt management |
| Cost | Self-host free; Cloud subscription + usage-based events — **approximate, verify live pricing** |
| Best for | Open-source tracing + evals with self-hosting/data residency, production monitoring |

### 7.4 G-Eval — The LLM-as-a-Judge Technique

**G-Eval** (arXiv:2303.16634) is not a platform but the **framework for LLM-as-a-judge evaluation with chain-of-thought reasoning**: the judge LLM (1) receives evaluation criteria in natural language, (2) generates step-by-step reasoning about the output's quality, then (3) produces a score (1–5 or 1–10). CoT-before-scoring correlates better with human judgment than direct scoring. Covered in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §11.

In RAG evaluation, G-Eval appears as the **custom-criteria judge** — e.g., DeepEval's `GEval` metric:

```python
from deepeval.metrics import GEval
metric = GEval(name="Banking Accuracy",
    criteria="Determine if output contains accurate financial figures and correctly cites regulations",
    evaluation_params=[LLMTestCaseParams.INPUT, LLMTestCaseParams.ACTUAL_OUTPUT])
```

**Strengths:** any custom criteria in natural language (domain-specific quality, compliance, tone); research-backed; flexible score range.
**Weaknesses:** LLM-judge only — the most expensive evaluation method (full CoT call per sample); judge-model dependent (smaller models degrade); not a framework (no datasets/CI/monitoring) — use it *through* DeepEval or another tool.

| Dimension | G-Eval |
|-----------|--------|
| Category | Metric technique (LLM-as-a-judge with CoT) |
| License / hosting | OSS technique; implemented in DeepEval (`GEval`), Phoenix, Langfuse judges |
| RAG metrics | None built-in — any custom criterion you define |
| Testset generation | ❌ |
| Tracing / dashboard | ❌ (use a host tool) |
| Cost | Judge-LLM calls only |
| Best for | Custom, domain-specific quality criteria where human correlation matters (e.g., compliance phrasing) |

---

## 8. The Head-to-Head Comparison

### 8.1 The Master Comparison Table

> **Pricing note:** all prices below are **approximate snapshots (mid-2026) and change frequently** — treat them as order-of-magnitude guidance and verify live pricing before any budget commitment. OSS tools are free as libraries; their real cost is judge-LLM API calls and self-hosting operations.

| Tool | RAG metrics coverage | Reference-based | LLM-judges | Testset generation | Tracing | Dashboard | Open-source | Hosted | Pricing (approx.) | Best for |
|------|---------------------|-----------------|------------|--------------------|---------|-----------|-------------|--------|-------------------|----------|
| **RAGAS** | Deepest — retrieval (context precision/recall, entity recall) + generation (faithfulness, answer relevancy/correctness/similarity), aspect critiques, robustness, agent metrics | Yes (correctness, recall, similarity) | Yes (most metrics) | ✅ TestsetGenerator | ❌ | ❌ | ✅ Apache 2.0 | ❌ | Free lib + judge calls | Deep retrieval/generation metrics, RAG architecture comparison, synthetic datasets |
| **TruLens** | RAG triad — context relevance, groundedness, answer relevance (+ custom feedback) | Yes (custom) | Yes | ❌ | ✅ OTel | ✅ Local Streamlit | ✅ MIT | ❌ | Free lib + judge calls | Quick triad evaluation with per-record traceability, data residency |
| **DeepEval** | Broad — answer relevancy, faithfulness, contextual relevancy/precision/recall, hallucination, RagasScore | Yes (custom) | Yes (+ G-Eval) | ✅ Synthesizer | ❌ | ❌ (Confident AI cloud) | ✅ Apache 2.0 | Optional cloud | Free lib + judge calls; paid cloud | CI eval gates, pytest-style test-driven RAG testing |
| **LangSmith** | Via evaluators — answer relevance/accuracy, retrieval quality; RAGAS metrics supported | Yes (ROUGE/BLEU/sim) | Yes (incl. online) | ❌ | ✅ Full | ✅ | ❌ closed | ✅ SaaS | Free tier; ≈$39/user/mo + usage — flag | LangChain stacks, integrated tracing + eval + production monitoring |
| **Arize Phoenix** | Built-in — hallucination, QA correctness, relevance, toxicity | Yes (QA correctness) | Yes (templates + custom) | ❌ | ✅ OTel | ✅ | ✅ Apache 2.0 | Arize AX cloud | Free OSS; AX usage-based — flag | Production RAG observability, drift, embedding analysis, self-hosted |
| **W&B Weave** | Via scorers/RAGAS — no large built-in set | Yes (custom) | Yes (scorers) | ❌ | ✅ | ✅ W&B | ❌ closed | ✅ SaaS | Free tier; paid team plans — flag | W&B teams wanting trace + eval + annotation |
| **MLflow** | Minimal RAG-specific — ROUGE/BLEU/similarity + custom LLM-judge criteria | Yes | Yes (custom prompts) | ❌ | Partial (experiments) | Partial (compare UI) | ✅ Apache 2.0 | Databricks | Free OSS | MLflow-centric teams, basic RAG eval alongside ML tracking |
| **Langfuse** | Mechanism not catalogue — LLM-as-judge rubrics, code scorers, RAGAS integration | Via scorers | Yes (judges + online) | ❌ | ✅ Full | ✅ | ✅ MIT | Langfuse Cloud | Self-host free; Cloud subscription + events — flag | Open-source tracing + evals, production monitoring, data residency |
| **G-Eval** | None built-in — any custom criterion (CoT judge) | No (judge-only) | Yes (CoT) | ❌ | ❌ | ❌ | Technique | Via host tools | Judge calls only | Custom domain criteria, small high-quality eval sets |

### 8.2 Dimension Analysis — Who Wins Each Dimension

**Metrics depth → RAGAS.** No other tool ships a comparable RAG-specific catalogue: the retrieval split (context precision vs recall) that TruLens collapses into one score, plus aspect critiques, robustness, and (in 0.2) agentic/multimodal metrics. If the question is "how deep can I measure my RAG", RAGAS is the answer — see [ragas_guide.md](ragas_guide.md) §2–§3.

**Observability (traces + dashboards + production) → Langfuse / Phoenix.** For attached-to-traces evaluation, live monitoring, and drift: Phoenix (OSS, self-hosted, embedding analysis) and Langfuse (MIT, self-hostable, datasets/experiments) lead the open-source side; LangSmith leads the hosted side with the smoothest LangChain integration and online evaluators. W&B Weave wins only if you are already committed to the W&B ecosystem.

**Test-driven / CI → DeepEval.** The only tool whose native contract is an assertion that fails a build: `assert_test` + thresholds + pytest runner. RAGAS/TruLens eval gates are things you *build* around `evaluate()`/`TruSession`; DeepEval's gate *is* the tool (see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) §2 for where gates fit).

**Testset generation → RAGAS.** `TestsetGenerator` (evolution/node-based) is the most mature document-to-questions generator; DeepEval's `Synthesizer` is the alternative. LangSmith/Langfuse/Phoenix manage datasets but do not generate them.

**Cost → RAGAS / TruLens / DeepEval / MLflow / Phoenix / Langfuse (OSS).** All free as software; the variable cost is judge-LLM API calls (every LLM-judge metric × sample). Hosted tools (LangSmith, Weave, Langfuse Cloud, Arize AX) add subscription + usage costs that scale with trace volume — budget carefully for production traffic.

**Ease of first value → TruLens.** Three feedback functions, a decorator, a local dashboard: fastest path from "RAG exists" to "RAG triad scores on real records". RAGAS is nearly as quick for pure numbers; Phoenix/Langfuse need more setup but pay off for production.

### 8.3 Strengths / Weaknesses Summary

| Tool | Key strengths | Key weaknesses |
|------|---------------|----------------|
| **RAGAS** | Metric depth; research-grounded; reference-free; testset generation | LLM cost; no tracing/dashboard/CI runner; RAG-only |
| **TruLens** | Triad simplicity; MIT OSS; evaluate+traces+dashboard; data residency | Scale limits; post-acquisition maturity wobble; shallow retrieval split |
| **DeepEval** | Test-driven/CI-native; broad RAG metrics + RagasScore; Synthesizer | Smaller ecosystem; Python-only; monitoring needs paid cloud |
| **LangSmith** | Integrated tracing; LangChain-native; experiments/A-B; online evaluators | LangChain-centric; hosted-only; cost at scale; lock-in |
| **Arize Phoenix** | Production observability; drift + embedding analysis; self-host; OTel | Setup complexity; younger LLM-eval features; AX for advanced |
| **W&B Weave** | W&B ecosystem; dataset versioning; annotation UI; RAGAS integration | SaaS-only; team-priced; thin built-in RAG metrics |
| **MLflow** | MLflow synergy; OSS self-host; familiar API; Databricks | No RAG-specific metrics; offline only; maturing |
| **Langfuse** | MIT self-host; framework-agnostic; datasets/experiments; online evals | BYO metric depth; self-hosting ops load |
| **G-Eval** | Custom criteria with CoT; research-backed; human-correlated | Judge-only; expensive; not a framework |

## 9. Tool Selection Guidance

### 9.1 Which Tool for Which Need

The selection logic follows from §8.2 — each tool wins a specific dimension, so the mapping is mostly one-to-one:

| If your dominant need is… | Choose | Why |
|---------------------------|--------|-----|
| **Metrics depth** (deep retrieval + generation scoring, RAG architecture comparison) | **RAGAS** | The deepest RAG-specific catalogue; precision-vs-recall split; research-grounded definitions |
| **RAG triad + a dashboard, fast** | **TruLens** | Three feedback functions, a decorator, a local Streamlit dashboard — first meaningful scores in hours |
| **Integrated tracing + evals in one hosted product (LangChain stack)** | **LangSmith** | Evaluation attached to traces, experiments/A-B, online evaluators for production |
| **Open-source tracing + evals with self-hosting / data residency** | **Langfuse** | MIT license, self-hostable, datasets/experiments, LLM-as-a-judge on traces |
| **Test-driven evaluation / CI gates** | **DeepEval** | Assertions that fail the build; pytest-native; the eval gate *is* the tool |
| **Production observability with drift and embedding analysis** | **Arize Phoenix** | Real-time dashboards, drift detection, trace-level root cause, self-hostable |
| **Stay inside an existing ecosystem (W&B / MLflow)** | **Weave / MLflow** | Best synergy when the team already lives in that platform |
| **Custom compliance/quality criteria, human-correlated scoring** | **G-Eval** (via DeepEval) | Chain-of-thought judge for criteria like "accurate financial figures and correct regulatory citations" |

### 9.2 The Scenario Table

| Scenario | Recommended tool | Runner-up / complement |
|----------|------------------|------------------------|
| "I need to evaluate my RAG **quickly**" | **TruLens** — triad scores + traces + dashboard in an afternoon | RAGAS for pure metric numbers |
| "I need **deep retrieval metrics** (precision vs recall, entity recall)" | **RAGAS** — context precision/recall, faithfulness, answer relevancy | DeepEval (RagasScore) if you also want CI assertions |
| "I need **production monitoring** of a live RAG service" | **Phoenix** (self-hosted) or **Langfuse** (self-hosted/cloud) | LangSmith if LangChain + hosted is acceptable; add drift detection via Phoenix |
| "I need **CI test-driven** RAG regression on every PR" | **DeepEval** — `assert_test` + thresholds in the pipeline | RAGAS wrapped in a custom CI job |
| "I need the **combined pattern** (dev metrics + production records)" | **RAGAS + Langfuse** — RAGAS for dev-time metric depth, Langfuse for tracing/evals/monitoring | RAGAS + LangSmith (hosted), or DeepEval + Phoenix |

### 9.3 The Selection Framework

A repeatable four-step framework (the same shape as the methodology guide's evaluation design — see [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §6):

**Step 1 — Requirements.** Write down your non-negotiables, not your wishlist:
- *Eval surface:* which layers must be scored (retrieval only? generation? both?) and which metrics are mandatory (faithfulness for a bank is non-negotiable; context precision matters only if you plan retrieval tuning).
- *Workflow:* dev-time batch eval, CI gates, production monitoring — or all three? (Each track may use a different tool — see §10.)
- *Constraints:* open-source preference / data residency (banking), judge-model availability (Bedrock/on-prem), team language (Python/TS), existing platform commitments (MLflow, W&B, LangChain).
- *Budget:* judge-LLM call volume for OSS; subscription + usage for hosted.

**Step 2 — Candidates.** Shortlist from §8.1's table using the constraints. Typical shortlists:
- OSS-only + data residency → RAGAS, TruLens, DeepEval, Phoenix, Langfuse (drop LangSmith, Weave).
- LangChain stack + no residency constraint → LangSmith (+ RAGAS metrics).
- CI-first → DeepEval (+ a tracing backend for records).

**Step 3 — Pilot.** Run the top 1–2 candidates on a **real slice of your eval dataset** (the methodology guide's golden set) against the *same* RAG pipeline for one week. Score the pilot on: metric usefulness (does a low score point at a fixable failure mode?), integration effort, judge cost per sample, and dashboard/trace inspectability. Two tools can legitimately pass the pilot for different tracks.

**Step 4 — Decide.** Choose per-track, document the decision (tool, version, metric list, judge model, thresholds) in the eval specification — this becomes part of the validation evidence file if you are in a regulated context (see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) §3 and [implementing-responsible-ai.md](../implementing-responsible-ai.md)). Revisit quarterly: this market moves fast.

> **Anti-pattern to avoid:** picking one tool before writing down the requirements. The tools are complementary, not interchangeable — "best tool" does not exist; "best tool *for this track*" does.

---

## 10. Worked Example — Selecting Tools for a Banking RAG

### 10.1 The Scenario

Cymbal Bank's trade-finance desk wants a **product-support RAG assistant**: front-office users ask about trade-finance products (standby LCs, documentary credits, guarantees, FX forwards, commodity finance), and the assistant answers from the **product documentation corpus** (term sheets, operational handbooks, regulatory summaries). Answers must be **accurate, grounded, and relevant** — a hallucinated fee or eligibility criterion is a compliance incident, not a bug.

This is the **same scenario** used across the series' evaluation guides — [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) (methodology), [trulens_guide.md](trulens_guide.md) §9 (TruLens deep-dive), and [ragas_guide.md](ragas_guide.md) §9 (RAGAS deep-dive) — so the tool choice below is the piece those guides deliberately leave open.

### 10.2 The Requirements

| # | Requirement | Implication for tool choice |
|---|-------------|-----------------------------|
| 1 | **Deep retrieval metrics** — the team plans to tune chunking and add a reranker, and needs precision vs recall separately (the retrieval fix differs: *rerank* fixes precision, *chunking/index* fixes recall) | Needs RAGAS-level metric depth (context precision *and* recall) — TruLens' single context-relevance score is not enough |
| 2 | **Production monitoring** — once live, the assistant must be watched: groundedness of real traffic, latency/cost, alerts when quality drops | Needs a tracing + eval platform with online evaluation, not a metrics library |
| 3 | **Open-source preference + data residency** — bank policy: evaluation data and traces must stay in-house; no hosted SaaS for this workload | Rules out LangSmith and W&B Weave (hosted-only); favours self-hostable OSS (RAGAS, TruLens, DeepEval, Phoenix, Langfuse) |
| 4 | **CI eval gate** — every retrieval/chunking change runs the golden set before merge (see [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §7 on eval gates) | Nice-to-have for DeepEval; can be built with RAGAS in a CI job |
| 5 | **Validation evidence** — audit needs documented eval specs and per-sample scores (MAS FEAT accountability; [implementing-responsible-ai.md](../implementing-responsible-ai.md)) | Any tool works if the spec + scores are exported and versioned |

### 10.3 The Selection — RAGAS + Langfuse

Applying §9.3: requirements 1 and 3 immediately shortlist **RAGAS** (deep metrics, OSS) and rule out LangSmith/Weave (hosted). Requirement 2 then decides the second tool: a tracing/eval platform. Between **Phoenix** and **Langfuse** (both self-hostable OSS), **Langfuse** wins here because the bank already runs the RAG stack on LangChain/LlamaIndex-agnostic Python and wants datasets/experiments for regression runs; Phoenix remains the runner-up (better if embedding-drift analysis were the priority).

**The chosen combo — RAGAS for dev-time metrics + Langfuse for production tracing/evals:**

| Track | Tool | Role |
|-------|------|------|
| **Dev-time evaluation** | **RAGAS** | Deep two-layer metrics on the golden + synthetic sets; testset generation from term sheets; architecture comparisons (chunking × reranker × query strategy) |
| **CI eval gate** | **RAGAS in a CI job** | Run `evaluate()` on the 150-question golden set; fail the pipeline if faithfulness < 0.9 or context precision < 0.7 (thresholds per methodology guide) |
| **Production tracing + evals** | **Langfuse (self-hosted)** | Trace every retrieval/generation step; LLM-as-a-judge groundedness + context-relevance scores on live traffic; datasets/experiments for release regressions; alerting on score drops |

**Why not the alternatives:** TruLens (great for a fast triad start, but requirement 1 needs the retrieval split); DeepEval (excellent CI story, but RAGAS + a CI job covers it and keeps one metric framework — *or* DeepEval + RAGAS metrics if the team prefers pytest); LangSmith/Weave (blocked by requirement 3); Phoenix alone (would cover monitoring, but metrics depth still needs RAGAS); MLflow (no RAG-specific metrics).

### 10.4 The Setup — Integration Sketch

```python
# --- Track 1+2: RAGAS dev-time metrics + CI gate (in the repo's eval job) ---
from datasets import Dataset
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall

eval_dataset = Dataset.from_dict({
    "question": golden_questions,      # 150 human-reviewed goldens (term sheets, handbooks)
    "answer": rag_answers,             # produced by the assistant under test
    "contexts": retrieved_chunks,      # list[list[str]], rank-ordered
    "ground_truth": golden_answers,
})
result = evaluate(dataset=eval_dataset,
                  metrics=[faithfulness, answer_relevancy, context_precision, context_recall])
# CI gate: fail if faithfulness < 0.9 or context_precision < 0.7
assert result["faithfulness"] >= 0.9, f"faithfulness too low: {result['faithfulness']}"
assert result["context_precision"] >= 0.7, f"context precision too low: {result['context_precision']}"
```

```python
# --- Track 3: Langfuse tracing + production evals (in the serving app) ---
from langfuse import Langfuse
from langfuse.decorators import observe

langfuse = Langfuse()  # self-hosted endpoint; traces stay in-house

@observe()
def answer_product_question(question: str) -> str:
    chunks = retrieve(question)          # traced: query, top-k chunks, scores
    answer = generate(question, chunks)  # traced: prompt, completion, tokens
    return answer

# Online judge (LLM-as-a-judge, rubric-guided) scores each live trace for
# groundedness and context relevance; scores land on the trace for dashboards/alerts.
```

The two tracks share the **same golden set and thresholds** (defined once in the methodology guide's eval spec), and the judge-model choice (Bedrock-hosted LLM, per bank policy) is identical for RAGAS metrics and Langfuse judges — see [ragas_guide.md](ragas_guide.md) §4 for judge setup and calibration.

### 10.5 The Lesson — Two Tools for the Two Tracks

The selection worked because it separated the **two tracks** of [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md): the **evaluation track** (dev-time measurement, iteration, gates — RAGAS) and the **validation/monitoring track** (production evidence, drift, alerts — Langfuse). No single tool was the best fit for both:

- RAGAS alone would leave production unmonitored.
- Langfuse alone would leave retrieval depth shallow (its metrics are BYO).
- Together, each does what it is best at, sharing one eval spec — and the whole setup is open source and in-house, satisfying the data-residency requirement.

This "metrics framework + observability platform" pairing is the **most common production pattern** in the RAG-eval landscape (the same shape as RAGAS+LangSmith, DeepEval+Phoenix, or TruLens standalone for smaller scopes). For the formal sign-off side — who approves the model, with which evidence — see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) §3 and the governance overlay in [implementing-responsible-ai.md](../implementing-responsible-ai.md).

---

## 11. Summary — RAG Eval Tools in One Page

**The landscape.** RAG evaluation needs two layers measured separately — retrieval (precision/recall) and generation (faithfulness, relevancy) — per [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md). Nine tools serve this: metrics frameworks (RAGAS, DeepEval, G-Eval), observability platforms (Arize Phoenix, Langfuse, LangSmith, W&B Weave, TruLens as the hybrid), and test-driven frameworks (DeepEval).

**The winners by dimension:**
- **Metrics depth → RAGAS** (retrieval split, faithfulness, testset generation; see [ragas_guide.md](ragas_guide.md))
- **Triad + dashboard, fastest first value → TruLens** (see [trulens_guide.md](trulens_guide.md))
- **Test-driven CI gates → DeepEval** (`assert_test`, pytest-native)
- **Integrated tracing + eval (hosted, LangChain) → LangSmith** (online evaluators)
- **Open-source tracing + evals (self-hosted) → Langfuse**
- **Production observability + drift → Arize Phoenix**
- **Ecosystem synergy → W&B Weave / MLflow** (only if you already live there)
- **Custom compliance criteria → G-Eval** (via DeepEval's `GEval`)

**Selection in one line:** write requirements → shortlist from the master table (§8.1) → pilot on your real golden set → decide **per track**, because the common winning architecture is *a metrics framework for dev-time depth + an observability platform for production records* — RAGAS + Langfuse being the canonical open-source pairing (worked example in §10).

**The final word.** RAG evaluation tools are complementary, not interchangeable. The bank's lesson applies to everyone: measure retrieval and generation separately, gate releases on real thresholds, keep production monitored — and choose tools per track, not one tool for everything. The tooling changes fast (pricing, versions, and features all moved within the last year); the two-layer evaluation discipline is what lasts.

---

## 12. Glossary

| Term | Definition |
|------|-----------|
| **RAGAS** | Open-source (Apache 2.0) Python framework for RAG evaluation; the deepest catalogue of RAG-specific metrics (faithfulness, context precision/recall, answer relevancy/correctness) plus testset generation. See [ragas_guide.md](ragas_guide.md). |
| **TruLens** | Open-source (MIT) evaluate + trace library (TruEra/Snowflake) built around the RAG triad, with feedback functions and a local Streamlit dashboard. See [trulens_guide.md](trulens_guide.md). |
| **DeepEval** | Open-source (Apache 2.0) LLM evaluation framework by Confident AI; pytest-style, test-driven (`assert_test`), with RAG metrics (answer relevancy, faithfulness, contextual relevancy/precision/recall, hallucination) and native RagasScore integration. |
| **LangSmith** | LangChain's commercial (SaaS) LLM evaluation and observability platform: tracing, datasets, experiments, evaluators, online evaluators, production monitoring. |
| **Arize / Phoenix** | Arize AI's open-source LLM observability platform: OpenTelemetry tracing, built-in RAG evals (hallucination, QA correctness, relevance), embedding/drift analysis; Arize AX is the managed cloud sibling. |
| **W&B / Weave** | Weights & Biases' hosted LLM toolkit: Weave tracing, Weave evals (datasets + scorers), experiment tracking, dataset versioning; RAGAS integration for RAG metrics. |
| **MLflow** | Open-source MLOps platform whose `mlflow.evaluate()` supports LLM/RAG evaluation (question-answering task, ROUGE/BLEU/similarity, custom LLM-judge criteria); Databricks integration. |
| **Langfuse** | Open-source (MIT) LLM engineering platform: tracing, scores, datasets, LLM-as-a-judge evals, experiments, prompt management; self-hostable or Langfuse Cloud. |
| **G-Eval** | LLM-as-a-judge framework using chain-of-thought before scoring (arXiv:2303.16634); implemented as DeepEval's `GEval` metric; best for custom natural-language criteria. |
| **Metrics framework** | A library that computes evaluation scores over provided data (usually LLM-as-a-judge); no tracing or dashboard built in (RAGAS, DeepEval). |
| **Observability platform** | A system that traces LLM app executions (spans, latency, cost), stores them, and attaches evaluation scores; provides dashboards and production monitoring (Phoenix, Langfuse, LangSmith, Weave, TruLens). |
| **Test-driven** | Evaluation as assertions with thresholds that fail a build/CI run; the eval gate is the tool's native contract (DeepEval). |
| **Tracing** | Recording each execution step of an LLM app (retrieval, generation, tool calls) as structured spans, enabling per-record debugging and score attachment. |
| **RAG metrics** | Metrics specific to RAG systems, split across the retrieval layer (context precision/recall, contextual relevancy) and the generation layer (faithfulness, answer relevancy/correctness). |
| **Faithfulness** | Whether the answer is grounded in the retrieved context — the hallucination gate; RAGAS `faithfulness`, TruLens `groundedness`, DeepEval `FaithfulnessMetric`. |
| **Context precision** | Retrieval ranking quality — are relevant chunks ranked high? RAGAS `context_precision`; DeepEval `ContextualPrecisionMetric`. |
| **Answer relevancy** | Whether the answer addresses the question (off-topic detection); RAGAS `answer_relevancy`/`response_relevancy`, DeepEval `AnswerRelevancyMetric`, TruLens `answer_relevance`. |
| **Contextual relevancy** | Whether the retrieved context is relevant to the question (retrieval layer); DeepEval `ContextualRelevancyMetric`. |
| **LLM-as-a-judge** | Using a separate LLM to score outputs against criteria (with or without chain-of-thought, as in G-Eval); the engine behind most RAG metrics; cost scales with samples × metrics. |
| **Testset generation** | Synthesising evaluation questions/answers from your own documents (RAGAS `TestsetGenerator`, DeepEval `Synthesizer`) — eval data before human goldens exist. |
| **Dashboard** | A UI for browsing scores, traces, and trends (TruLens Streamlit, Langfuse, Phoenix, LangSmith, W&B). |
| **Open-source / hosted** | Open-source = runs in your environment (data in-house); hosted = SaaS where traces/eval data live on the vendor's platform (LangSmith, Weave). |
| **Pricing** | For OSS tools: free software + judge-LLM API costs; for hosted: subscription + usage-based metering. All figures in this guide are approximate and must be verified. |
| **Eval gate** | A threshold-based checkpoint (CI or release) that fails when metrics drop below targets; DeepEval assertions or a CI job around RAGAS `evaluate()`. |
| **CI** | Continuous integration — running the eval suite on every pull request so RAG regressions fail fast. |
| **Production monitoring** | Continuous evaluation of live traffic: online LLM-judge scoring, drift detection, alerts (LangSmith online evaluators, Langfuse online evals, Phoenix drift). |
| **Online evaluator** | An evaluator that scores live production traces as they happen, reference-free, against a rubric (LangSmith online evaluators; Langfuse online evaluation). |

---

*End of guide. Next in the series: [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) (the tool-agnostic methodology) · [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) (master tooling comparison) · [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) (evaluation vs validation).*


