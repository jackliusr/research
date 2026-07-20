# DeepEval and LLM Evaluation Frameworks — A Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [MLOps Lifecycle](mlops_lifecycle_frameworks_guide.md) · [LLM Compression](llm_compression_alternatives_guide.md) · [Guard Models](llm_guard_models_guide.md) · [Multi-Pass LLM Pipelines](multi_pass_llm_pipelines_guide.md)
> **Last Updated:** July 2026

---

## Table of Contents

1. [What Are LLM Evaluation Frameworks?](#1-what-are-llm-evaluation-frameworks)
2. [Why LLM Evaluation Matters](#2-why-llm-evaluation-matters)
3. [Evaluation Approaches](#3-evaluation-approaches)
4. [Evaluation Metric Categories](#4-evaluation-metric-categories)
5. [DeepEval Deep Dive](#5-deepeval-deep-dive)
6. [LangSmith Evaluation](#6-langsmith-evaluation)
7. [MLflow LLM Evaluation](#7-mlflow-llm-evaluation)
8. [RAGAS](#8-ragas)
9. [Arize AI / Phoenix](#9-arize-ai--phoenix)
10. [Weights & Biases Prompts](#10-weights--biases-prompts)
11. [G-Eval](#11-g-eval)
12. [Other Alternatives](#12-other-alternatives)
13. [Comparison Dimensions](#13-comparison-dimensions)
14. [Quick-Reference Comparison Matrix](#14-quick-reference-comparison-matrix)
15. [CI/CD Integration Patterns](#15-cicd-integration-patterns)
16. [Evaluation Data Strategies](#16-evaluation-data-strategies)
17. [Evaluation as Part of LLMOps](#17-evaluation-as-part-of-llmops)
18. [Tool Selection Guide](#18-tool-selection-guide)
19. [Conclusion and Recommendations](#19-conclusion-and-recommendations)

---

## 1. What Are LLM Evaluation Frameworks?

LLM evaluation frameworks are specialised tools for testing, evaluating, and monitoring LLM outputs — covering accuracy, relevance, hallucination, bias, safety, and custom metrics. They are the critical infrastructure that moves LLMs from prototype to production.

**What they provide:**
- **Metrics** — quantitative scores for quality, safety, hallucination, bias, and more
- **Evaluation pipelines** — automated workflows that run metrics against model outputs
- **Test suites** — structured datasets and assertions for regression testing
- **Production monitoring** — continuous evaluation of live LLM traffic
- **Comparison tools** — side-by-side evaluation of models, prompts, or configurations
- **Data synthesis** — generation of evaluation datasets when real data is scarce
- **CI/CD integration** — gating deployments on evaluation thresholds

### 1.1 The LLM Evaluation Challenge

| Aspect | Traditional ML | LLM Evaluation |
|--------|---------------|----------------|
| Output type | Single value / class label | Free-form text, code, structured data |
| Ground truth | Typically available | Often unavailable or subjective |
| Correctness | Binary or continuous metric | Multi-dimensional (factual, relevant, safe, coherent) |
| Subjectivity | Low | High — quality perception varies by user |
| Safety surface | Narrow (label-based) | Broad — toxicity, bias, hallucination, jailbreaks |
| Evaluation cost | Low (computation only) | High — LLM-as-judge calls, human annotation |
| Reproducibility | Deterministic | Non-deterministic (temperature, judge model variance) |
| Regulatory scrutiny | Growing | Intense — EU AI Act, MAS FEAT, Singapore AI Verify |

Without systematic evaluation, teams rely on anecdotal testing and "vibes-based" quality assessment — a recipe for deploying unreliable, unsafe, or non-compliant LLM applications.

---

## 2. Why LLM Evaluation Matters

### 2.1 Differences from Traditional ML Evaluation

1. **No single ground truth.** An LLM asked "Explain quantum computing to a 10-year-old" has multiple valid responses. Evaluating free-form text requires semantic understanding, not string matching.
2. **Open-ended outputs.** LLMs generate prose, code, analysis, creative content — each requiring different quality criteria (relevance, coherence, factual accuracy, style).
3. **Subjective quality dimensions.** Relevance, helpfulness, tone — these are human judgments. A BLEU score captures surface form, not usefulness.
4. **Safety and harm.** LLMs can produce toxic content, reinforce biases, hallucinate facts, or be jailbroken. These failure modes are absent in traditional ML classification.
5. **Compound systems.** Modern LLM apps chain retrieval, prompting, tool calls, and generation. Each component can fail independently or interact in unexpected ways.
6. **Non-deterministic outputs.** The same input with temperature > 0 produces different outputs. Evaluation must account for sampling variability.

### 2.2 Banking and Regulatory Context

**MAS FEAT (Fairness, Ethics, Accountability, Transparency):**

| Principle | LLM Evaluation Implication |
|-----------|---------------------------|
| **Fairness** | Must not discriminate against protected groups. Requires bias metrics cross-checked against demographic fairness. |
| **Ethics** | Must not generate harmful, manipulative, or deceptive content. Requires toxicity, harmful content, and jailbreak detection metrics. |
| **Accountability** | Human oversight must be possible. Requires traceability — every evaluation result linked to specific model version, prompt, and input. |
| **Transparency** | Customers must be informed when interacting with AI. Requires output watermarking and disclosure verification. |

For LLM-powered applications in banking — robo-advisory, KYC chatbots, credit analysis, compliance research — evaluation frameworks must demonstrate compliance across all four FEAT dimensions.

**EU AI Act (Regulation 2024/1689):**

| Risk Level | Examples | Evaluation Requirements |
|------------|----------|------------------------|
| **High-Risk** | Credit scoring, employment, access to services | Conformity assessment, risk management, human oversight, accuracy/robustness documentation |
| **Limited Risk** | Chatbots, AI-generated content | Transparency obligations (disclose AI interaction) |

LLMs in banking often fall into **High-Risk** or **Limited Risk** categories. Article 15 mandates "appropriate levels of accuracy, robustness, and cybersecurity" — directly requiring systematic, documented evaluation. Penalties up to 7% of global revenue or €35M.

**Singapore AI Verify:** Certification requires documented testing across fairness, explainability, robustness, and safety — all covered by LLM evaluation frameworks.

### 2.3 Model Risk Management for LLMs

Traditional MRM (OCC 2011-12, SR 11-7, MAS guidelines) requires: model identification, development, validation, monitoring, and governance. For LLMs:

| MRM Phase | LLM-Specific Challenge | Evaluation Role |
|-----------|----------------------|----------------|
| **Identification** | Every prompt template + model combination is a distinct "model" | Track evaluation datasets per use case |
| **Development** | Prompt engineering IS model development | Unit-test assertions, regression suites |
| **Validation** | No ground truth for open-ended outputs | Structured eval datasets, LLM-as-judge, human annotation |
| **Monitoring** | Outputs drift with model updates, retrieval changes | Production monitoring, drift detection |
| **Governance** | Audit trail: model → prompt → input → output → evaluation score | Trace-based evaluation, experiment tracking |

Without systematic evaluation, MRM for LLMs is impossible — you cannot manage what you cannot measure.

---

## 3. Evaluation Approaches

### 3.1 LLM-as-Judge

A separate LLM evaluates the target model's outputs against a rubric or criteria.

**How it works:** `Input → [Target LLM] → Output → [Judge LLM with Rubric] → Score + Reasoning`

**Strengths:** Highly flexible — evaluates any dimension expressible in natural language. Correlates well with human judgment when using capable judge models (GPT-4, Claude). Evaluates subjective qualities (relevance, coherence, tone). No ground truth required for many metrics. Supports custom evaluation criteria.

**Weaknesses:** Costly (1000 samples × 5 metrics = 5000+ LLM calls). **Judge bias** — judge LLMs favour outputs matching their own style. **Position bias** — first-presented output favoured in pairwise comparisons. **Self-enhancement bias** — judges rate own outputs higher. **Calibration** — scores inconsistent across different judge models. **Latency** — evaluation adds inference time.

**Judge Model Selection:**

| Judge Model | Reliability | Cost | Notes |
|-------------|-------------|------|-------|
| GPT-4 / GPT-4o | High | $$$ | Best human alignment |
| Claude 3.5 Sonnet | High | $$$ | Less position bias |
| Gemini 1.5 Pro | Medium-High | $$ | Good long-context |
| DeepSeek-V3 / R1 | Medium | $ | Cost-effective alternative |
| Llama 3 70B / 405B | Medium | $ | Self-hosted option |
| Local (7B–13B) | Low-Medium | Free | Unreliable scoring risk |

**Mitigation strategies:** Use multiple judge models and average scores. Randomise presentation order (pairwise). Calibrate judge scores against human annotations. Use chain-of-thought prompting for the judge. Provide explicit rubrics with concrete examples.

### 3.2 Reference-Based Metrics

Compare LLM outputs against a known correct reference.

| Metric | What It Measures | Best For |
|--------|-----------------|----------|
| BLEU | N-gram precision | Translation |
| ROUGE-L | Longest common subsequence | Summarisation |
| ROUGE-1/2 | Unigram/bigram recall | Summarisation, QA |
| METEOR | Unigram matching with synonym support | Translation, paraphrasing |
| BERTScore | Semantic similarity via BERT embeddings | Any text generation |
| Exact Match | Output equals reference exactly | Factual QA, extraction |
| F1 Score | Token-level precision + recall | QA, entity extraction |
| Cosine Similarity | Embedding-based semantic similarity | Semantic equivalence |

**Strengths:** Deterministic, reproducible, cheap (no LLM calls for n-gram metrics), well-understood with decades of research.

**Weaknesses:** Require ground truth references (expensive to create). N-gram metrics miss semantic equivalence ("car" vs "automobile" ≠ BLEU match). Poor correlation with human judgment for open-ended tasks. Useless for safety/bias evaluation.

### 3.3 Embedding-Based Metrics

Encode both generated output and reference into embedding vectors, compute cosine similarity.

**Embedding models:** OpenAI text-embedding-3-small/large, BAAI/bge-large-en-v1.5, intfloat/e5-mistral-7b-instruct, nomic-ai/nomic-embed-text-v1.5.

**Strengths:** Captures semantic meaning (not surface form), fast and cheap (embedding inference cheaper than LLM calls), language-agnostic for multilingual evaluation.

**Weaknesses:** Less precise than LLM-as-judge for nuanced criteria. Cannot evaluate safety, bias, or factual correctness directly. Embedding quality varies by domain and model.

### 3.4 Code-Based Metrics

Programmatic checks for structured outputs: JSON Schema validation, regex pattern matching, Pydantic deserialisation, code compilation/execution tests, length/constraint checks, business rule assertions.

**Strengths:** Deterministic, fast, cheap, CI/CD-friendly, precise failure localisation.
**Weaknesses:** Only works for structured/semi-structured outputs. Cannot evaluate quality, relevance, or safety. Brittle — small format changes break strict checks.

### 3.5 Human Evaluation

Gold standard: Likert scales (rate 1–5), pairwise comparison (which is better?), best-worst scaling, error annotation (mark specific failures).

**Strengths:** Most reliable quality assessment. Catches subtle issues models miss (cultural nuance, domain error). Ground truth for calibrating automated metrics. Required for High-Risk EU AI Act compliance.

**Weaknesses:** Expensive ($5–$50 per 100 annotations). Slow (days to weeks for large campaigns). Inconsistent (inter-annotator agreement target κ > 0.7). Not reproducible. Hard to scale.

**When necessary:** EU AI Act conformity assessment, MAS FEAT bias/fairness evaluation, subject-matter expert validation (legal, medical, financial), calibrating LLM-as-judge metrics, production release gates for customer-facing applications.

### 3.6 Ensemble / Combination Approaches

Best practice combines multiple approaches: LLM-as-judge (quality/safety) + reference-based (factual accuracy) + embedding (semantic similarity) + code-based (schema validation) + human spot-checks (calibration). Most comprehensive, most expensive, and satisfies regulatory requirements for human oversight of automated evaluation.

---

## 4. Evaluation Metric Categories

### 4.1 Quality Metrics

| Metric | What It Measures | Typical Use |
|--------|-----------------|-------------|
| Relevance | Output addresses the input question | All LLM applications |
| Coherence | Ideas flow logically, connect well | Narrative, explanations |
| Fluency | Grammatically correct, natural-sounding | All text generation |
| Consistency | No self-contradictions | Factual QA, reporting |
| Correctness | Factually accurate | Knowledge-based tasks |
| Completeness | Covers all required aspects | Summarisation, instruction following |
| Conciseness | Appropriately brief | Assistant chatbots |
| Helpfulness | Genuinely assists user's goal | Open-domain assistants |
| Instruction Following | Adheres to system/user instructions | Structured tasks |

### 4.2 Safety Metrics

| Metric | What It Measures | Regulatory Relevance |
|--------|-----------------|---------------------|
| Toxicity | Hate speech, harassment, profanity | EU AI Act, MAS FEAT |
| Bias | Harmful stereotypes, discrimination | MAS FEAT Fairness, EU AI Act |
| Harmful Content | Violence, self-harm, illegal instructions | EU AI Act, Online Safety |
| Jailbreak Detection | System prompt leakage, guardrail bypass | Security, MAS FEAT |
| Sexually Explicit | Pornographic or sexually violent content | Safety moderation |
| PII Leakage | Personally identifiable information | GDPR, PDPA |
| Copyright Violation | Copyrighted text reproduced verbatim | Legal compliance |

### 4.3 Hallucination Metrics

| Metric | What It Measures | Evaluation Approach |
|--------|-----------------|-------------------|
| Faithfulness | Grounded in provided context, doesn't fabricate | LLM-as-judge |
| Factuality | Statements are true in the real world | LLM-as-judge + knowledge base |
| Context Adherence | Doesn't exceed or contradict given context | LLM-as-judge |
| Entity Accuracy | Named entities (people, places, dates) correct | Reference + entity extraction |
| Claim Verification | Individual claims in output are verifiable | Decomposition + verification |

### 4.4 Retrieval Metrics (RAG)

Context Precision (proportion of retrieved documents that are relevant), Context Recall (proportion of relevant documents that were retrieved), Context Relevancy, Context Entity Recall.

### 4.5 Task-Specific Metrics

| Task | Metrics |
|------|---------|
| Summarisation | Compression ratio, coverage, redundancy, faithfulness, ROUGE |
| Translation | BLEU, COMET, chrF, adequacy, fluency, terminology consistency |
| Coding | Execution success rate, HumanEval pass rate, functional correctness |
| Question Answering | Exact Match, F1, semantic equivalence |
| Agents | Tool call accuracy, plan completeness, goal achievement rate |
| Classification | Accuracy, precision, recall, F1, calibration |

### 4.6 Custom / Domain-Specific Metrics (Banking)

Regulatory citation accuracy, risk disclosure completeness, product suitability, KYC/AML compliance, numerical precision, jurisdiction awareness.

**Implementation:** DeepEval `CustomMetric` with scoring rubric, LangSmith `CustomEvaluator`, RAGAS custom `Metric` class, Phoenix Python callable eval functions.

---

## 5. DeepEval Deep Dive

### 5.1 Overview

**DeepEval** is an open-source LLM evaluation framework by **Confident AI** (Apache 2.0, Python). It provides a **pytest-like, unit-testing approach** with 50+ research-backed metrics covering RAG, quality, safety, hallucination, bias, and conversation.

- **Repo:** [github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval) | **Website:** [deepeval.com](https://deepeval.com/)
- **Cloud layer:** [Confident AI](https://www.confident-ai.com/) (dashboards, collaboration, production monitoring)

### 5.2 Built-In Metrics

**RAG Metrics:** FaithfulnessMetric, AnswerRelevancyMetric, ContextualPrecisionMetric, ContextualRecallMetric, ContextualRelevancyMetric, HallucinationMetric, RagasScore (native RAGAS integration).

**Quality Metrics:** GEval (G-Eval framework with chain-of-thought for ANY custom criteria), SummarizationMetric.

**Safety Metrics:** ToxicityMetric, BiasMetric, JailbreakDetectionMetric.

**Conversation/Agent Metrics:** ConversationRelevancyMetric, ConversationCompletenessMetric, RoleAdherenceMetric.

**Non-LLM Metrics:** BLEU, ROUGE, BERTScore, METEOR, SacreBLEU, cosine similarity.

**Custom:** `CustomMetric` with scoring rubric. Composite metrics combining multiple metrics into a single score.

### 5.3 Test-Driven Development for LLMs

```python
from deepeval import assert_test
from deepeval.metrics import FaithfulnessMetric
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input="What is Crédit Agricole's approach to sustainable finance?",
    actual_output="Crédit Agricole CIB provides green bonds, sustainability-linked loans...",
    retrieval_context=["Crédit Agricole CIB: Green and sustainable bonds"])
assert_test(test_case, [FaithfulnessMetric(threshold=0.7)])
```

Provides deterministic pass/fail with configurable thresholds, clear failure messages, parametrised test cases, fixtures, and native pytest integration (`pytest test_llm.py`).

### 5.4 Data Synthesis

```python
from deepeval.synthesizer import Synthesizer
dataset = Synthesizer().generate(
    context_documents=["sustainability_report_2025.pdf"],
    num_cases=50)
# Generates input, expected_output, context per case
```

Creates realistic test cases from documents with stratified question types.

### 5.5 CI/CD Integration

```yaml
- run: pip install deepeval
- run: deepeval test run test_llm.py
  env: { OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }} }
```

CLI: `deepeval test run test_llm.py --verbose`, `deepeval dataset push my_dataset`.

### 5.6 Strengths

1. **Pythonic, unit-test style** — if you know pytest, you know DeepEval. `assert_test()` integrates naturally with existing Python test suites.
2. **50+ built-in metrics** — most comprehensive metric library among open-source frameworks.
3. **RAG-focused evaluation** — faithfulness, hallucination, contextual precision/recall, native RAGAS integration.
4. **Built-in data synthesis** — generate evaluation datasets from documents without manual labelling.
5. **CI/CD native** — CLI test runner, pytest integration, GitHub Actions ready.
6. **Model-agnostic judge** — any LLM as judge (OpenAI, Anthropic, local via Ollama/vLLM).
7. **Modular composable metrics** — mix and match metrics per test case, create custom metrics with rubrics.
8. **Active community** — regular releases (~monthly), responsive maintainers, growing ecosystem.

### 5.7 Weaknesses

1. **Still maturing** — pre-1.0 versions (historically 0.x), breaking changes possible, documentation can lag features.
2. **LLM-as-judge cost** — advanced metrics (faithfulness, G-Eval, hallucination) use LLM-as-judge. Large datasets get expensive.
3. **Documentation gaps** — advanced custom metric writing and composite configuration lack worked examples.
4. **Smaller community than LangSmith/MLflow** — fewer third-party tutorials, fewer Stack Overflow answers.
5. **Python-only** — no TypeScript/JS SDK. Node.js teams need Python shims.
6. **Production monitoring requires Confident AI** — OSS handles offline eval; monitoring/dashboards/collaboration need paid layer.

### 5.8 Best For

Teams wanting pytest-style LLM testing, comprehensive RAG evaluation, CI/CD integration, evaluation-driven development, and open-source budget-friendly evaluation.

---

## 6. LangSmith Evaluation

LangChain's LLM evaluation and observability platform (SaaS, Python + TypeScript). Provides tracing, evaluation, datasets, human annotation, experiment management, and production monitoring. [langchain.com/langsmith](https://www.langchain.com/langsmith)

**Key Features:**
- **Datasets:** Versioned test examples, automatic construction from production traces, API/SDK/UI management.
- **Evaluators:** Manual (human rating via UI), LLM-as-Judge (GPT-4, Claude, any), Code (Python/TS custom functions), Pairwise (A/B comparison), Reference-Based (ROUGE, BLEU, embedding similarity).
- **Trace-Based Evaluation:** Every LLM call, tool invocation, and retrieval step is traced. Evaluations attach to full execution traces for root-cause debugging.
- **Experiment Tracking:** Compare runs across models/prompts/configs. Side-by-side tables with statistical significance testing.
- **Production Monitoring:** Real-time evaluation of live traffic, automated feedback collection, drift detection, alerting, latency/cost tracking.
- **Human Feedback:** Embedded widgets (thumbs up/down, star ratings, free-text comments) linked to traces.
- **Comparison Views:** A/B testing, model comparison (GPT-4 vs Claude vs Llama), prompt comparison, historical regression.

**Strengths:** Deep LangChain integration (automatic tracing, evaluators understand LangChain data model). Trace-based evaluation connected to execution traces. Production monitoring with drift + alerting. Best-in-class A/B comparison views. Human-in-the-loop annotation workflows. Multi-language (Python + TS). Cloud-managed (zero infrastructure).

**Weaknesses:** LangChain-centric (deepest integration only with LangChain). Cloud-only (no self-hosted — data residency issues for banking). Expensive at scale (usage-based pricing). Vendor lock-in (evaluation data lives in LangSmith). Fewer built-in metrics than DeepEval (relies on configuring/writing evaluators).

**Best For:** LangChain-heavy stacks, teams wanting continuous production monitoring + evaluation, human annotation workflows, and A/B testing models/prompts.

---

## 7. MLflow LLM Evaluation

LLM evaluation component within MLflow (Apache 2.0, Python). `mlflow.evaluate()` extended to LLMs — familiar API for existing MLflow users. [mlflow.org/docs/latest/llms/llm-evaluate](https://mlflow.org/docs/latest/llms/llm-evaluate/)

**Key Features:**
- **Built-In Metrics:** Toxicity, Flesch Kincaid readability, sentence similarity (embedding), answer similarity, ROUGE-1/2/L, BLEU.
- **LLM-as-Judge:** Evaluate with custom prompts, define criteria as natural language rubrics, any model as judge.
- **Datasets:** Pandas DataFrames or MLflow Dataset objects with versioning.
- **Model Comparison UI:** Side-by-side across models with aggregated metrics and distribution visualisation.
- **Supported Tasks:** QA, summarisation, text classification, RAG, custom tasks.

```python
with mlflow.start_run():
    results = mlflow.evaluate(model="openai:/gpt-4", data=eval_df,
        model_type="question-answering", evaluators="default",
        evaluator_config={"col_mapping": {"inputs": "question",
                           "ground_truth": "expected_answer"}})
```

**Strengths:** Integrated with MLflow ecosystem (seamless if already using MLflow). Open source (Apache 2.0). Model comparison across traditional ML + LLM metrics. Supports many model types (OpenAI, Anthropic, local, MLflow registered). Familiar API. Databricks integration.

**Weaknesses:** Newer than core MLflow (LLM features added 2024, still maturing). Fewer LLM-specific metrics than DeepEval (no built-in faithfulness, contextual precision/recall, hallucination, bias). Less mature LLM evaluation UI. Python-only. No production monitoring (offline only).

**Best For:** Teams already on MLflow, wanting basic LLM eval alongside traditional ML tracking, open-source self-hosted evaluation, and Databricks users.

---

## 8. RAGAS

**RAGAS** (RAG Assessment) is an open-source framework (Apache 2.0, Python) purpose-built for evaluating RAG pipelines — retrieval-level and generation-level metrics specifically for RAG systems. [github.com/vibrantlabsai/ragas](https://github.com/vibrantlabsai/ragas) | [docs.ragas.io](https://docs.ragas.io/)

**Metrics — Retrieval (Component-Level):** Context Precision (are retrieved docs relevant?), Context Recall (all relevant docs retrieved?), Context Relevancy, Context Entity Recall.

**Metrics — Generation (End-to-End):** Faithfulness (grounded in context, not hallucinating?), Answer Relevancy (relevant to query?), Answer Correctness (factually correct vs ground truth), Answer Similarity (semantic similarity), Answer Semantic Similarity (embedding-based).

**Aspect Critique:** DiscreteMetric/ContinuousMetric for harmfulness, maliciousness, coherence, correctness, conciseness. Custom aspects via natural language description.

**Components:**
- **TestsetGenerator:** Creates simple/reasoning/multi-context/conditional questions from documents. `generator.generate_with_langchain_docs(documents, test_size=50)`.
- **Pipeline Evaluation:** `evaluate(dataset=test_dataset, metrics=[faithfulness, answer_relevancy])` returns per-sample + aggregated scores as Pandas DataFrame.
- **Monitoring:** Integrates with LangSmith for production monitoring.

**Strengths:** Purpose-built for RAG — every metric targets a specific RAG failure mode. Well-defined, research-backed methodology (ACL 2023). Open source (Apache 2.0). Active development with rapid feature additions. Component-level + end-to-end metrics give complete RAG health picture. Built-in test set generation from documents.

**Weaknesses:** RAG-only focus (not useful for non-RAG LLM apps — chatbots, content generation, agents). LLM-as-judge cost (most metrics use LLM-as-judge internally — 1000 samples × 5 metrics = 5000+ LLM calls). No built-in CI/CD integration (library only, must write CI layer yourself). Python-only. No production monitoring (needs LangSmith or custom pipeline). No safety metrics (toxicity/bias outside scope).

**Best For:** RAG pipeline evaluation exclusively — retrieval quality assessment, RAG architecture comparison, synthetic evaluation dataset generation.

---

## 9. Arize AI / Phoenix

Open-source LLM observability platform (Apache 2.0, Python) combining tracing, evaluation, and monitoring in one tool. Evaluation results connect to full execution traces for root-cause analysis. [github.com/Arize-ai/phoenix](https://github.com/Arize-ai/phoenix) | [arize.com/phoenix](https://arize.com/phoenix/)

**Key Features:**
- **Tracing (OpenTelemetry):** Auto-instrumentation for LangChain, LlamaIndex, OpenAI. Span-level latency and token cost tracking. Visualise execution DAGs.
- **Evaluation:** Response correctness, relevance, toxicity (LLM-as-judge). Eval as Code (Python callables). Pre-built judge templates. Human rating via UI.
- **Drift Monitoring:** Embedding drift (input/output distribution changes). Metric drift (score trends). Data quality monitoring.
- **Embedding Analysis:** UMAP projections, clustering, outlier detection, similarity search.
- **Production Monitoring:** Real-time dashboard with metrics. Alerts on degradation. Cost and latency tracking. Arize AI cloud for team collaboration.

**Strengths:** Open source + self-hostable (Apache 2.0). Mature drift detection from Arize's ML observability background. Tracing + evaluation integrated (connect scores to execution traces for root cause analysis). Embedding-level analysis (visualise outputs in embedding space). OpenTelemetry compatible (standards-based).

**Weaknesses:** Newer LLM-specific evaluation features (less mature than ML observability). Fewer built-in LLM metrics than DeepEval (no pre-built hallucination, bias, contextual precision/recall). Complex self-hosted setup (requires database, OpenTelemetry collector, Phoenix service). Documentation originally for ML monitoring. Advanced features require Arize AI cloud.

**Best For:** Teams wanting tracing + evaluation in one tool, LLM observability with drift detection, embedding-level analysis, and debugging complex chains/agents.

---

## 10. Weights & Biases Prompts

LLM evaluation and prompt management within the W&B platform (SaaS, Python + TypeScript). [wandb.ai/site/prompts](https://wandb.ai/site/prompts/)

**Key Features:** LLM-as-judge evaluation (GPT-4, Claude, custom endpoints). Human annotation UI (label-based, free-text comments, queue management, inter-annotator agreement metrics). Versioned datasets with diffing. Prompt versioning (track, compare, rollback). Comparison tables. Chain tracing. W&B ecosystem integration (artifacts, reports, dashboards, sweeps).

**Strengths:** W&B ecosystem integration (natural extension if already using W&B). Best-in-class dataset versioning (immutable, auditable snapshots). Accessible human annotation UI for non-technical team members. Tight prompt version → evaluation linkage. Rich comparison views.

**Weaknesses:** Requires W&B account (no self-hosted — W&B Dedicated Cloud is still managed). SaaS-only (evaluation data leaves your infrastructure). Expensive for large teams (pricing scales with team size). Fewer built-in LLM metrics than DeepEval (relies on LLM-as-judge and custom functions).

**Best For:** Teams already using W&B, needing prompt management + evaluation + human annotation, and comparison-driven A/B testing.

---

## 11. G-Eval

Framework for LLM evaluation using LLM-as-judge with chain-of-thought (CoT) reasoning (arXiv:2303.16634). Not a full platform — implemented by DeepEval as the `GEval` metric.

**Process:** Define evaluation criteria → LLM generates CoT reasoning about output quality → LLM produces score (1–5 or 1–10).

**Metrics (via DeepEval):** Coherence, consistency, fluency, relevance, and ANY custom criteria defined in natural language.

```python
from deepeval.metrics import GEval
metric = GEval(name="Financial Accuracy",
    criteria="Determine if output contains accurate financial figures and correctly cites regulations",
    evaluation_params=[LLMTestCaseParams.INPUT, LLMTestCaseParams.ACTUAL_OUTPUT],
    model="gpt-4")
```

**Strengths:** CoT-based evaluation correlates better with human judgment than direct LLM scoring. Supports ANY custom evaluation criteria — domain-specific quality, tone, style, compliance. Research-backed (peer-reviewed). Flexible scoring range (1–5 or 1–10).

**Weaknesses:** LLM-as-judge only (always the most expensive method — every evaluation requires a full CoT LLM call). Not a full evaluation framework (no built-in datasets, CI/CD, monitoring — use through DeepEval). Judge model dependent (GPT-4 shows good results; smaller models degrade). No safety metrics (toxicity/bias — must define custom criteria).

**Best For:** Research teams wanting CoT-based evaluation, custom domain-specific criteria, and small-scale high-quality evaluation where human correlation matters most.

---

## 12. Other Alternatives

### 12.1 Continuous Eval

Open-source library (PyPI) for data-driven LLM evaluation. Features modular metric design, data synthesis, RAG metrics (faithfulness, answer relevancy, context precision/recall), non-RAG metrics (hallucination, bias, coherence), custom metric support.
**Best for:** Teams wanting lightweight modular open-source evaluation without full framework overhead.

### 12.2 Galileo

Managed enterprise GenAI evaluation platform [galileo.ai](https://galileo.ai/). Features chain-level evaluation (assess every step of a chain), proprietary Hallucination Index (benchmarks LLM hallucination propensity), Context Sufficiency/Adherence metrics, managed pipelines, model comparison. Most expensive option.
**Best for:** Enterprises wanting managed evaluation with proprietary hallucination metrics.

### 12.3 Vertex AI Evaluation (Rapid Evaluators)

GCP-native service [cloud.google.com/vertex-ai](https://cloud.google.com/vertex-ai). Features AutoSxS (automated side-by-side pairwise comparison), safety evaluation (toxicity, harmfulness, bias), Rapid Evaluators (pre-built for summarisation/QA/classification), LLM Comparator interactive UI, custom evaluators. Pay per evaluation.
**Best for:** Teams already on GCP/Vertex AI.

### 12.4 Azure AI Evaluation

Microsoft Azure AI Studio. Features quality metrics (coherence, fluency, relevance, groundedness), safety metrics (toxicity, hate speech, self-harm, sexual content, violence), AI-assisted safety evaluations, custom evaluators via Python SDK, CI/CD with Azure DevOps, human feedback collection.
**Best for:** Teams on Azure/Microsoft ecosystem with strong safety evaluation requirements.

### 12.5 Helix (Nomic)

Open-source research project for LLM agent evaluation. Agent-specific metrics (tool call accuracy, plan completion, goal achievement), multi-step trajectory evaluation, embedding-based agent behaviour analysis.
**Best for:** Teams building and evaluating LLM agents.

### 12.6 LangFuse

Open-source (MIT) LLM engineering platform [github.com/langfuse/langfuse](https://github.com/langfuse/langfuse). Features OpenTelemetry tracing, evaluation (manual, LLM-as-judge, automated), prompt management, datasets + experiments, cost tracking. Self-hosted or cloud. Python + TypeScript SDKs.
**Best for:** Teams wanting open-source alternative to LangSmith with tracing + evaluation + prompt management. Self-hostable for banking data residency requirements.

### 12.7 Comet Opik

Open-source core with Comet cloud for advanced features. LLM-as-judge evaluation, human annotation, dataset/experiment management, prompt versioning, chain tracing, comparison views. Self-hosted or cloud.
**Best for:** Teams evaluating Opik as an alternative to LangSmith or MLflow for LLM-specific workflows.

---

## 13. Comparison Dimensions

### 13.1 By Evaluation Approach

| Approach | Frameworks Supporting It |
|----------|-------------------------|
| LLM-as-Judge | DeepEval, LangSmith, G-Eval, W&B, Galileo, Vertex AI, Azure AI, LangFuse, Opik |
| Reference-Based | MLflow, RAGAS, DeepEval (BLEU/ROUGE/BERTScore) |
| Embedding-Based | MLflow, Phoenix, RAGAS, DeepEval |
| Code-Based | DeepEval (assertions), LangSmith (custom evaluators), Phoenix (Eval as Code) |
| Human Annotation | LangSmith, W&B, Galileo, LangFuse |

### 13.2 RAG Metrics vs Non-RAG Metrics Support

| Metric | DeepEval | RAGAS | LangSmith | MLflow | Phoenix |
|--------|----------|-------|-----------|--------|---------|
| Faithfulness | ✅ Native | ✅ Native | ✅ LLM-as-judge | ⚠️ Custom | ⚠️ Custom |
| Context Precision | ✅ Native | ✅ Native | ✅ LLM-as-judge | ❌ | ⚠️ Custom |
| Context Recall | ✅ Native | ✅ Native | ✅ LLM-as-judge | ❌ | ⚠️ Custom |
| Answer Relevancy | ✅ Native | ✅ Native | ✅ LLM-as-judge | ⚠️ Embedding | ⚠️ Custom |
| Hallucination | ✅ Native | ⚠️ Faithfulness | ✅ LLM-as-judge | ❌ | ⚠️ Custom |
| Toxicity | ✅ Native | ❌ | ✅ LLM-as-judge | ✅ Native | ✅ Native |
| Bias | ✅ Native | ❌ | ✅ LLM-as-judge | ❌ | ❌ |
| Jailbreak Detection | ✅ Native | ❌ | ⚠️ Custom | ❌ | ❌ |
| Coherence | ✅ via GEval | ❌ | ✅ LLM-as-judge | ❌ | ⚠️ Custom |

### 13.3 CI/CD, Monitoring, Data Synthesis

| Feature | DeepEval | LangSmith | MLflow | RAGAS | Phoenix | LangFuse |
|---------|----------|-----------|--------|-------|---------|----------|
| pytest runner | ✅ Native | ❌ | ❌ | ❌ | ❌ | ❌ |
| CLI test runner | ✅ `deepeval test run` | ❌ | ❌ | ❌ | ❌ | ❌ |
| GitHub Actions | ✅ | ✅ | ✅ | ⚠️ Manual | ⚠️ Manual | ✅ |
| Production monitoring | ⚠️ Confident AI | ✅ Real-time | ❌ | ❌ | ✅ Real-time | ✅ Real-time |
| Drift detection | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ |
| Alerting | ⚠️ Confident AI | ✅ | ❌ | ❌ | ✅ | ✅ |
| Dashboard | ⚠️ Confident AI | ✅ | ✅ (MLflow UI) | ❌ | ✅ | ✅ |
| Data synthesis | ✅ Built-in | ❌ | ❌ | ✅ Built-in | ❌ | ❌ |

### 13.4 Cost Model

| Framework | Open Source | SaaS Pricing | Self-Hostable |
|-----------|-------------|-------------|---------------|
| DeepEval | ✅ Apache 2.0 | Confident AI (paid) | ✅ |
| LangSmith | ❌ | Usage-based (pay as you go) | ❌ |
| MLflow LLM Eval | ✅ Apache 2.0 | Databricks (optional) | ✅ |
| RAGAS | ✅ Apache 2.0 | ❌ (library only) | ✅ |
| Phoenix (Arize) | ✅ Apache 2.0 | Arize AI (paid) | ✅ |
| W&B Prompts | ❌ | Free tier + paid plans | ❌ (cloud only) |
| LangFuse | ✅ MIT | Cloud (free + paid) | ✅ |
| G-Eval | ✅ (via DeepEval) | ❌ | ✅ |
| Galileo | ❌ | Enterprise pricing | ❌ (cloud only) |
| Vertex AI Eval | ❌ | Pay per evaluation | ❌ (GCP only) |
| Azure AI Eval | ❌ | Pay per evaluation | ❌ (Azure only) |
| Continuous Eval | ✅ Open source | ❌ | ✅ |

---

## 14. Quick-Reference Comparison Matrix

| Tool | Open Source | Primary Focus | RAG Metrics | LLM-as-Judge | CI/CD | Production Monitor | Pricing |
|------|-------------|---------------|-------------|--------------|-------|-------------------|---------|
| **DeepEval** | ✅ Apache 2.0 | General LLM + RAG eval | ✅ 7 native | ✅ Any model, G-Eval | ✅ pytest CLI, GA | ⚠️ Confident AI | Free OSS; paid cloud |
| **LangSmith** | ❌ | Observability + eval | ⚠️ LLM-as-judge | ✅ | ✅ API/SDK | ✅ Real-time | Usage-based |
| **MLflow LLM Eval** | ✅ Apache 2.0 | MLflow-integrated eval | ⚠️ Limited | ✅ | ✅ CI | ❌ | Free OSS |
| **RAGAS** | ✅ Apache 2.0 | RAG pipeline eval | ✅ 6 native | ✅ Internal | ⚠️ Manual | ❌ | Free OSS |
| **Phoenix (Arize)** | ✅ Apache 2.0 | Observability + eval | ⚠️ Custom | ✅ | ⚠️ Manual | ✅ Real-time | Free OSS; paid cloud |
| **W&B Prompts** | ❌ | Prompt mgmt + eval | ⚠️ Custom | ✅ | ✅ SDK | ✅ Real-time | Free tier + paid |
| **LangFuse** | ✅ MIT | Observability + eval | ⚠️ Custom | ✅ | ✅ API/SDK | ✅ Real-time | OSS free; paid cloud |
| **Galileo** | ❌ | Enterprise GenAI eval | ✅ Context metrics | ✅ | ✅ Native | ✅ | Enterprise |
| **G-Eval** | ✅ (via DeepEval) | CoT-based LLM-as-judge | ❌ | ✅ CoT-based | ❌ | ❌ | Free (judge call cost) |
| **Vertex AI Eval** | ❌ | GCP-native eval | ⚠️ Limited | ✅ AutoSxS | ✅ GCP CI/CD | ✅ | Pay per eval |
| **Azure AI Eval** | ❌ | Azure-native eval | ⚠️ Limited | ✅ | ✅ Azure DevOps | ✅ | Pay per eval |
| **Continuous Eval** | ✅ Open source | Modular LLM eval | ✅ | ✅ | ⚠️ Manual | ❌ | Free OSS |

---

## 15. CI/CD Integration Patterns

### 15.1 PR Check Pattern

Evaluate LLM outputs against a baseline on every PR. Block merge if metrics below threshold.

```yaml
- run: pip install deepeval && deepeval test run tests/llm_regression.py
  env: { OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }} }
- name: Check thresholds
  run: deepeval test report --format json | jq -e '.failures | length > 0' && exit 1 || exit 0
```

### 15.2 Regression Detection Pattern

Run evaluation on current branch, compare against baseline from main branch. `deepeval compare current_results.json baseline_results.json`.

### 15.3 Safety Gate Pattern

Block deployment if safety metrics below threshold (e.g., toxicity score < 0.8 → block). Extract score from JSON output, compare against configured threshold.

### 15.4 Automatic Evaluation Pipeline

CI triggers: generate eval dataset → run metrics → check thresholds → report to PR → push to monitoring dashboard.

### 15.5 Integration by Framework

| Framework | PR Check | Regression | Safety Gate | Dashboard |
|-----------|----------|------------|-------------|-----------|
| DeepEval | ✅ Native CLI | ✅ `deepeval compare` | ✅ Threshold assertions | ✅ via Confident AI |
| LangSmith | ✅ API | ✅ Experiment comparison | ✅ Custom evaluator | ✅ Native |
| MLflow | ✅ Python | ✅ `mlflow.evaluate()` compare | ⚠️ Custom | ✅ MLflow UI |
| RAGAS | ⚠️ Script wrapper | ⚠️ Manual | ❌ | ❌ |
| Phoenix | ⚠️ Script wrapper | ⚠️ Manual | ⚠️ Custom | ✅ Native |
| LangFuse | ✅ API | ✅ Experiment comparison | ✅ | ✅ Native |

---

## 16. Evaluation Data Strategies

### 16.1 Human-Annotated Golden Dataset

100–500 curated examples with human-validated ground truth. Use for benchmarking, regression testing, and calibrating LLM-as-judge metrics. Refresh quarterly as model capabilities and business requirements evolve.

### 16.2 Synthetic Data Generation

Programmatic generation using DeepEval `Synthesizer` or RAGAS `TestsetGenerator`. Scalable, covers edge cases. Weakness: may not reflect real user behaviour or domain-specific nuance.

**DeepEval:** `Synthesizer().generate(context_documents=[...], num_cases=200)`
**RAGAS:** `TestsetGenerator.with_openai().generate_with_langchain_docs(documents, test_size=200, distributions={simple: 0.5, reasoning: 0.25, multi_context: 0.25})`

### 16.3 Production Data Sampling

Stratified sampling of real traffic for offline evaluation — ensures evaluation datasets reflect actual usage patterns. Anonymise/PII-scrub before adding to evaluation dataset.

### 16.4 Adversarial Test Set

Deliberately crafted edge cases: jailbreak prompts, empty/long/multilingual inputs, boundary inputs (e.g., "What if a client wants to launder money?"), conflicting context, adversarial retrieval queries.

### 16.5 Stratified Sampling

Ensure coverage across: input complexity (simple QA vs multi-step reasoning), domains (KYC, AML, trade finance, credit analysis), languages, demographics, time periods, and known failure modes.

---

## 17. Evaluation as Part of LLMOps

### 17.1 The Evaluate → Monitor → Improve Loop

```
Development [Unit Tests, per commit, DeepEval/pytest]
    → Staging [Regression Detection, per PR/release, DeepEval/LangSmith]
    → Production [Monitoring, continuous, LangSmith/Phoenix/LangFuse]
    → Pre-release [Safety Gate, per release]
    → Quarterly [Full re-evaluation, all tools + human annotators]
    → Improve (feedback→prompts, models, retrieval)
```

### 17.2 Connecting Evaluation to Improvement

- Metric regression → rollback prompt/model (automated)
- Safety threshold breach → block deployment
- Drift detected → re-evaluate model
- Annotation patterns → prompt refinement
- Cost tracking → switch to cheaper model if quality metrics hold

### 17.3 Link to MLOps Lifecycle

| MLOps Phase | LLM Evaluation Integration |
|-------------|---------------------------|
| Data Management | Evaluation datasets are versioned artifacts (MLflow, W&B) |
| Model Development | Metrics guide prompt/model selection |
| Model Validation | Independent evaluation against golden dataset |
| Deployment | CI/CD gates based on evaluation thresholds |
| Monitoring | Production evaluation feeds drift detection and alerts |
| Governance | Evaluation results serve as audit trail for compliance |

### 17.4 Link to Guard Models

LLM evaluation and guard models are complementary:

| Aspect | LLM Evaluation | Guard Models |
|--------|---------------|-------------|
| Applied | Periodically (offline or sampled) | Real-time, per-request |
| Purpose | Measure and improve quality | Block harmful content |
| Output | Metrics, scores, regression alerts | Block/allow decisions |
| Cost | Periodic | Per-request |
| Regulatory role | Compliance evidence | Safety enforcement |

**Combined workflow:**
```
Input → [Guard Model: Block/Pass] → [LLM] → [Guard Model: Block/Pass] → Output
                                                                          ↓
                                                              [Evaluation Sampling]
                                                                (DeepEval, LangSmith)
```

Evaluation frameworks assess whether guard models are working correctly and whether the combined guard+LLM system meets quality and safety requirements.

---

## 18. Tool Selection Guide

### 18.1 By Evaluation Focus

| Focus | Primary Choice | Why |
|-------|---------------|-----|
| **RAG evaluation** | RAGAS + DeepEval | Retrieval metrics (RAGAS) + end-to-end + safety (DeepEval) |
| **General LLM quality** | DeepEval or LangSmith | DeepEval for built-in metrics; LangSmith for production monitoring |
| **Production monitoring** | LangSmith, Phoenix, LangFuse | Real-time eval, drift detection, alerting |
| **Safety/bias** | DeepEval + Azure AI/Vertex AI | DeepEval OSS safety metrics; cloud platforms for regulatory-grade |
| **Agent evaluation** | LangSmith or Helix | Trace-based multi-step evaluation |
| **Lightweight offline** | DeepEval | pytest-style, no platform dependency |
| **Academic research** | G-Eval (via DeepEval) | Best human alignment via CoT |

### 18.2 By Ecosystem

| Ecosystem | Choice | Why |
|-----------|--------|-----|
| LangChain | LangSmith | Deepest integration, automatic tracing |
| MLflow | MLflow LLM Evaluation | Same API/UI/artifacts |
| GCP / Vertex AI | Vertex AI Evaluation | Native GCP, AutoSxS |
| Azure / Microsoft | Azure AI Evaluation | AI Studio, safety evals, Azure DevOps |
| W&B | W&B Prompts | Dataset versioning, prompt management |
| None | DeepEval | Open source, platform-agnostic |

### 18.3 By Evaluation Approach

| Preferred approach | Choice | Why |
|-------------------|--------|-----|
| LLM-as-Judge | DeepEval (G-Eval integration) | Best G-Eval support, any model as judge, custom rubrics |
| Reference-Based | MLflow or RAGAS | Built-in BLEU/ROUGE/embedding metrics |
| Human Annotation | LangSmith or W&B Prompts | Best human annotation UIs, queue management |
| Code-Based | DeepEval | pytest-native assertions, assert_test() syntax |

### 18.4 By Deployment

| Need | Choice | Why |
|------|--------|-----|
| Self-hosted (on-prem, data residency) | DeepEval, RAGAS, Phoenix, LangFuse | Apache 2.0/MIT, fully self-hostable |
| Self-hosted with UI | Phoenix or LangFuse | Open-source dashboards + monitoring |
| SaaS (zero infrastructure) | LangSmith or W&B Prompts | Fully managed |
| Cloud platform native | Vertex AI or Azure AI | Single provider, integrated billing |
| Air-gapped / no internet | DeepEval + local judge (Ollama/vLLM) | OSS framework + local LLM |

### 18.5 Banking-Specific Recommendations

| Use Case | Recommended Stack | Rationale |
|----------|------------------|-----------|
| Internal RAG QA system | DeepEval + RAGAS | Faithfulness, contextual precision/recall, hallucination |
| Customer-facing chatbot | DeepEval (unit tests) + LangFuse self-hosted (monitoring) | Dev testing + data-residency-compliant production monitoring |
| Credit analysis assistant | DeepEval + Human annotation | Automated metrics + domain expert validation |
| Compliance/NLP system | DeepEval (bias/safety) + MLflow (tracking) | Bias metrics + experiment tracking for audit trails |
| Regulatory reporting | DeepEval custom metrics + Human annotation | Domain-specific metrics + expert review |
| Production monitoring | LangFuse (self-hosted) | Trace-based evaluation with data residency control |

**Key banking considerations:**
- **Data residency:** Prefer self-hosted frameworks (DeepEval, LangFuse, Phoenix) over SaaS
- **Regulatory compliance:** Need audit trails linking model version → prompt → output → evaluation score
- **Human oversight:** High-risk applications require human-in-the-loop validation, not just automated metrics
- **Custom metrics:** Banking-specific dimensions (regulatory accuracy, numerical precision, jurisdiction awareness)

---

## 19. Conclusion and Recommendations

### 19.1 Summary

LLM evaluation frameworks are critical infrastructure for moving from prototype to production. Without systematic evaluation, LLM applications risk hallucination, safety failures, regulatory non-compliance, quality degradation, and regression.

### 19.2 Framework Selection Summary

| Need | Simplest Path |
|------|--------------|
| Single framework covering most needs | **DeepEval** — 50+ metrics, pytest-style, free |
| Production monitoring + evaluation | **LangSmith** or **LangFuse** |
| RAG evaluation | **RAGAS** + **DeepEval** |
| MLflow ecosystem | **MLflow LLM Evaluation** |
| Self-hosted / air-gapped | **DeepEval** + **Phoenix** or **LangFuse** |
| Cloud-platform native | **Vertex AI** (GCP) or **Azure AI** (Microsoft) |
| Best human-aligned evaluation | **G-Eval** (via DeepEval) |
| LLM agent evaluation | **LangSmith** (traces) or **Helix** |

### 19.3 No Single Best Framework

There is no single "best" LLM evaluation framework. The right choice depends on what you're evaluating (RAG, chatbot, agent), where you are (development, staging, production), what ecosystem you're in, compliance requirements, budget, and scale. **Most mature teams use 2–3 frameworks in combination:**

- **DeepEval** (or RAGAS) for offline evaluation and CI/CD gates
- **LangSmith** (or Phoenix or LangFuse) for production monitoring and tracing
- **Human annotation** workflows for high-risk edge cases

### 19.4 Getting Started

1. **Start small** — evaluate 50–100 representative samples with 3–5 core metrics
2. **Build a golden dataset** — 100–200 human-annotated examples as ground truth
3. **Calibrate LLM-as-judge** — compare judge scores against human annotations
4. **Add CI/CD gates** — block PRs/metrics that fall below thresholds
5. **Monitor in production** — add trace-based evaluation for live traffic
6. **Iterate** — use evaluation results to improve prompts, models, and retrieval
7. **Scale** — expand to full regression suites, synthetic data, adversarial testing

### 19.5 Related Guides

- [Full MLOps Lifecycle Frameworks Guide](mlops_lifecycle_frameworks_guide.md) — MLOps lifecycle, stages, tools, best practices
- [LLM Compression Tools Guide](llm_compression_alternatives_guide.md) — Quantization, pruning, distillation, model merging
- [LLM Guard Models Guide](llm_guard_models_guide.md) — Llama Guard, ShieldGemma, safety classification
- [Multi-Pass LLM Pipelines Guide](multi_pass_llm_pipelines_guide.md) — Chaining multiple LLM passes for reliability
- [Enterprise AI Platforms Guide](enterprise_ai_platforms_guide.md) — End-to-end AI platforms for banking

---

*This guide is maintained as part of the [github.com/jackliusr/research](https://github.com/jackliusr/research) repository. For corrections or additions, please open an issue or PR.*
