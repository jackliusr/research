# DeepEval — The Test-Driven LLM Evaluation Framework — A Comprehensive Guide
> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) (master tooling comparison — DeepEval §5) · [LLM Evaluation vs Validation](../llm_evaluation_vs_validation_guide.md) · [TruLens](trulens_guide.md) · [Ragas](ragas_guide.md) · [RAG Evaluation Tools Comparison](rag_evaluation_tools_comparison_guide.md) · [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [AI Agent Drift](../ai_agent_drift_guide.md) · [Autonomous Agents](../autonomous_agents_guide.md) (§5 agent evals) · [Implementing Responsible AI](../implementing-responsible-ai.md)
> **Last Updated:** August 2026

---

## Table of Contents
1. [DeepEval Overview](#1-deepeval-overview)
2. [The Metrics Catalog](#2-the-metrics-catalog)
3. [Metric Deep-Dives](#3-metric-deep-dives)
4. [Test-Driven Evaluation](#4-test-driven-evaluation)
5. [Usage](#5-usage)
6. [CI/CD and the Dashboard](#6-cicd-and-the-dashboard)
7. [Ecosystem and Integrations](#7-ecosystem-and-integrations)
8. [Comparison with Alternatives](#8-comparison-with-alternatives)
9. [Worked Example — Evaluating a Banking RAG Assistant](#9-worked-example--evaluating-a-banking-rag-assistant)
10. [Summary — DeepEval in One Page](#10-summary--deepeval-in-one-page)
11. [Glossary](#11-glossary)

---

## 1. DeepEval Overview
### 1.1 What DeepEval Is
**DeepEval** is an open-source **LLM evaluation framework** built and maintained by **Confident AI** — "the LLM evaluation framework" per its own README — designed around one idea: **unit tests for LLM applications**. It is similar to **pytest**, but specialised for testing LLM apps: you write test cases, attach metrics with pass/fail thresholds, and get deterministic green/red verdicts instead of vague "vibe checks".
- **Repo:** [github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval) — the canonical open-source project (Python; a TypeScript SDK was also open-sourced in 2026, see §7).
- **Docs:** [deepeval.com/docs](https://deepeval.com/docs) (the primary documentation site — see §7.5 for a verification note on `docs.confident-ai.com`).
- **Cloud layer:** [Confident AI](https://www.confident-ai.com/) — the hosted platform on top of DeepEval (dashboards, dataset management, traces, production monitoring — see §6.3).
- **Language/runtime:** Python ≥ 3.9; runs evaluations **locally** on your machine by default (judge LLM calls go to whatever provider you configure).

DeepEval evaluates outputs with **research-backed metrics** — G-Eval, faithfulness, answer relevancy, hallucination, bias, toxicity, contextual precision/recall, and more — powered by **LLM-as-a-judge** (any model: OpenAI, Anthropic, Gemini, Azure OpenAI, Ollama, vLLM, custom), by **statistical/NLP methods** (BLEU, ROUGE, BERTScore), or by **embedding-based** methods. The current catalog is **50+ metrics** covering RAG pipelines, chatbots, agents, safety, and summarisation.

### 1.2 Origin and History — Confident AI
- **Confident AI** is the company behind DeepEval (originally branded around "Confident AI / confident-ai"), with the project created in 2023 (the repository's star history begins **August 2023**).
- DeepEval grew from a metrics library into a **full test-driven evaluation framework** with a CLI test runner (`deepeval test run`), a data synthesizer, and deep framework integrations (LangChain, LlamaIndex, OpenAI Agents, CrewAI, Hugging Face).
- **Confident AI** (the product/platform) is the commercial layer: a hosted platform for centralised test-run storage, dashboards, LLM traces, dataset management, and production/continuous evaluation. The open-source library remains the engine; the platform is optional for local use but required for team-scale dashboards and production monitoring.

> **Verification note:** the "Confident AI is the cloud platform built on top of DeepEval" split (OSS framework vs paid platform) is confirmed by both the GitHub README and Confident AI's own framework page. The company also publishes a **TypeScript version of DeepEval** (announced as fully open-source on deepeval.com in 2026).

### 1.3 Open Source and License
- **License: Apache 2.0** — verified from the `confident-ai/deepeval` repository (star-history and GitHub both list Apache-2.0). Permissive: free to use, modify, and embed commercially; no copyleft obligations. This matters for banks — Apache 2.0 is one of the most enterprise-friendly licences in the LLM-eval space (TruLens is MIT, Ragas is Apache 2.0, LangSmith is closed-source).
- **Cost:** the framework, CLI, metrics, and local evaluation are entirely free. You pay only for **judge-model calls** when using LLM-as-a-judge metrics (e.g., OpenAI or a self-hosted vLLM serving a judge model), plus whatever inference your app under test already consumes.
- **No lock-in for local use:** evaluations run locally without any account; the Confident AI platform (login, dashboards) is optional. Cloud features activate only when you run `deepeval login` and push results.

### 1.4 Positioning — "Test-Driven LLM Evaluation"
DeepEval's design philosophy is explicit: **treat your LLM application the way you treat regular software — with tests.** Concretely:

1. **Write a test case** — an `LLMTestCase` (input, actual output, optional retrieval context, optional expected output) plus one or more **metrics** with **thresholds**.
2. **Assert** — `assert_test(test_case, metrics=[...])` evaluates the metrics and raises if any score is below its threshold. A test either **passes** (green) or **fails** (red) — no ambiguity.
3. **Run in CI** — `deepeval test run` executes your test file (pytest under the hood) and the **exit code gates the pipeline**: regressions block merges.

This is the **test-driven development (TDD) loop applied to LLM apps**: write the test → run → fail → fix (prompt, retriever, model) → pass. DeepEval's own README calls the workflow "similar to Pytest but specialized for unit testing LLM applications", and the docs describe `assert_test()` as the unit-testing primitive that plugs into pytest.

### 1.5 DeepEval vs the Alternatives (Overview)
| Tool | Type | One-line positioning |
|------|------|----------------------|
| **DeepEval** | OSS library (Apache 2.0) | Pytest-native unit tests for LLM apps; 50+ metrics; `deepeval test run` CLI |
| **TruLens** | OSS library (MIT) | The RAG triad + tracing/instrumentation + local dashboard — see [trulens_guide.md](trulens_guide.md) |
| **Ragas** | OSS library (Apache 2.0) | RAG-specific research metrics + synthetic testset generation — see [ragas_guide.md](ragas_guide.md) |
| **LangSmith** | Commercial (SaaS) | Integrated observability + eval for the LangChain ecosystem |
| **Langfuse** | OSS + cloud (MIT) | Tracing/LLM-ops platform with eval on top — see [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md) |

The full comparison — including the master table in the series' evaluation-tooling umbrella, [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) (where DeepEval is covered in §5 and compared against LangSmith, MLflow, RAGAS, Phoenix, W&B, G-Eval) — is in §8 of this guide.

### 1.6 When to Use DeepEval
| Use case | Why DeepEval fits |
|----------|-------------------|
| **CI/CD eval gates** | `deepeval test run` returns a process exit code; pytest integration; GitHub Actions-ready (§6) |
| **Test-driven development for LLM apps** | `assert_test()` + thresholds give deterministic red/green; if you know pytest you know DeepEval |
| **Regression testing** | Re-run the same golden test set on every prompt/model/retriever change; failures localise regressions (§4, §6) |
| **RAG evaluation** | Faithfulness, contextual precision/recall/relevancy, hallucination, plus native RagasScore |
| **Agent/chatbot evaluation** | Multi-turn `ConversationalTestCase`, task completeness, role adherence, tool-call metrics |
| **Safety screening** | Toxicity, bias, jailbreak detection, non-advice — run as gates on outputs |
| **On-prem / bank-friendly eval** | Apache 2.0, runs locally, any judge model (incl. self-hosted via Ollama/vLLM) |

**When it is a weaker fit:** deep observability/tracing in production (TruLens or Langfuse are stronger), research-grade RAG metric depth alone (Ragas), or managed team-scale tracing with vendor support (LangSmith). Details in §8.4.

### 1.7 The DeepEval Niche
- **The only pytest-native eval framework** — unit-test culture, parametrised test cases, fixtures, exit-code gating. TruLens scores records, Ragas scores datasets; DeepEval *asserts*.
- **50+ plug-and-play metrics** — the widest built-in catalog in OSS, spanning RAG, quality, safety, conversation, agents, and deterministic NLP metrics.
- **CI-first by design** — CLI runner, JSON reports, governance gate (`deepeval gate`), no SaaS required for the gate itself.
- **Model-agnostic judging** — any LLM as judge, including local models, which keeps evaluation data inside the bank if needed.
- **Dual stack (OSS + platform)** — free local unit tests, optional Confident AI for dashboards and production monitoring.

### 1.8 The Overview Table
| Aspect | Description |
|--------|-------------|
| **What it is** | Open-source LLM evaluation framework — unit tests for LLM applications |
| **Maker** | Confident AI (company); repo `confident-ai/deepeval` |
| **License** | Apache 2.0 (verified) |
| **First released** | 2023 (repo created Aug 2023) |
| **Current version (mid-2026)** | 4.x line (e.g., 4.0.7 on PyPI, June 2026; 58 releases, latest July 2026) — see §7.7 |
| **Language** | Python ≥ 3.9; TypeScript SDK open-sourced in 2026 |
| **Positioning** | Test-driven LLM evaluation: `assert_test()` + pytest + `deepeval test run` CLI |
| **Metrics** | 50+ built-in: G-Eval, faithfulness, answer relevancy, contextual precision/recall/relevancy, hallucination, bias, toxicity, summarisation, RAGAS, deterministic NLP |
| **Judge models** | Any LLM-as-a-judge (OpenAI, Anthropic, Gemini, Azure, Ollama, vLLM, custom) |
| **CI/CD** | `deepeval test run` with exit-code gating; GitHub Actions examples; `deepeval gate` |
| **Dashboard** | Optional hosted **Confident AI** platform; local CLI output, JSON reports, `deepeval inspect` TUI |
| **Integrations** | LangChain, LlamaIndex, OpenAI Agents, CrewAI, Hugging Face, RagasScore |
| **Community** | ~17.5k GitHub stars (Aug 2026, approximate — see §7.4) |

---

## 2. The Metrics Catalog
### 2.1 Design Principle — One Metric per Failure Mode
Like Ragas ([ragas_guide.md](ragas_guide.md) §2), DeepEval's metric philosophy is **one metric per failure mode**: retrieval quality, generation faithfulness, relevance, safety, and conversational behaviour each get a dedicated, independently thresholdable metric. This is what makes `assert_test()` useful — you can gate a RAG app on *"faithfulness ≥ 0.85"* while separately watching *"contextual precision ≥ 0.70"*, and a failure tells you **which stage** of the pipeline regressed.

Metrics fall into three method families:

| Family | How it scores | Examples | Cost |
|--------|---------------|----------|------|
| **LLM-as-a-judge** | An LLM evaluates the output (often with explanations/CoT) | G-Eval, faithfulness, answer relevancy, contextual relevancy, hallucination, bias, toxicity, summarisation | Judge-model tokens per call |
| **Embedding-based** | Cosine similarity over embeddings | Answer relevancy (legacy variant), BERTScore | Embedding calls |
| **Statistical/NLP** | Deterministic string/NLP algorithms | BLEU, ROUGE, METEOR, SacreBLEU | Free, local |

### 2.2 The Core Metrics
- **G-Eval (`GEval`)** — the flagship *custom* metric: LLM-as-a-judge with **chain-of-thought (CoT)**, based on the G-Eval paper (arXiv:2303.16634, Microsoft). You define ANY evaluation criterion in natural language (e.g., "the answer is written in plain language a relationship manager can act on"), optionally give explicit evaluation steps, and the judge scores 1–5 with a probability-weighted expectation. See §3.1. Verified: DeepEval's README lists G-Eval first among its metrics.
- **Answer relevancy (`AnswerRelevancyMetric`)** — does the answer actually address the user's question? Catches off-topic or evasive answers. See §3.3.
- **Faithfulness (`FaithfulnessMetric`)** — is the answer **grounded in** the retrieved context? Statement-by-statement verdicts; the hallucination detector. See §3.2.
- **Contextual relevancy (`ContextualRelevancyMetric`)** — is the *retrieved context* relevant to the question (regardless of the final answer)? See §3.4.
- **Hallucination (`HallucinationMetric`)** — the dual of faithfulness: fraction of answer statements that **cannot be verified** against the context. Verified as a first-class metric in the catalog.
- **Bias (`BiasMetric`)** — safety metric: does the output contain gender, racial, or political bias? LLM-as-a-judge with explanations.
- **Toxicity (`ToxicityMetric`)** — safety metric: does the output contain hate speech, insults, or toxic language?
- **Summarisation (`SummarizationMetric`)** — quality of a summary vs the source text (alignment + coverage via LLM judge).
- **Jailbreak detection (`JailbreakDetectionMetric`)** — did the model refuse/fall for a jailbreak attempt in the input?
- **Non-advice (`NonAdviceMetric`) / Misuse** — safety metrics for regulated domains (e.g., a financial chatbot must not dispense personalised financial advice) — particularly relevant for banking use cases.

> **Verification note:** the exact catalog evolves each release. The above reflects the current docs (deepeval.com/docs/metrics-introduction) and README as of August 2026; multi-turn safety metrics (bias, toxicity, non-advice, misuse) are documented as `ConversationalTestCase` metrics.

### 2.3 The RAG Metrics
The RAG family (all verified in the current catalog) decomposes retrieval and generation quality so a RAG failure can be localised:

- **Contextual precision (`ContextualPrecisionMetric`)** — are the *relevant* retrieved chunks ranked at the top of the retrieval window? Punishes noise-before-signal.
- **Contextual recall (`ContextualRecallMetric`)** — does the retrieved context cover everything the ground-truth answer needs? (Requires an `expected_output`/ground-truth reference.)
- **Contextual relevancy** — is the retrieved context on-topic (signal vs noise in the window)?
- **Faithfulness / hallucination** — is the generation grounded in whatever was retrieved?
- **RagasScore** — a **native RAGAS integration**: DeepEval can run Ragas' own metrics (e.g., `context_precision`, `context_recall` from the RAGAS library) inside `assert_test()`, so teams get Ragas' research metric definitions with DeepEval's test runner. Verified via the `deepeval.metrics.ragas` module.

### 2.4 Conversation and Agent Metrics
For multi-turn chatbots and agents (using `ConversationalTestCase`), the catalog includes (verified via the README and docs):

- **Conversation relevancy / completeness** — does the conversation stay on-topic and answer the user's goal?
- **Role adherence (`RoleAdherenceMetric`)** — does the agent stay in its assigned role (e.g., a bank's support agent must not act as a salesperson)?
- **Task completion (`TaskCompletionMetric`)** — did the agent finish the requested task?
- **Knowledge retention (`KnowledgeRetentionMetric`)** — does the agent remember facts from earlier in the conversation?
- **Tool-call correctness** — agentic metrics checking that the agent called the right tools with the right arguments (agent evaluation coverage also in [autonomous_agents_guide.md](../autonomous_agents_guide.md) §5).

### 2.5 Deterministic (Non-LLM) Metrics
For tasks with exact references — translation, summarisation baselines, keyword checks — DeepEval ships classic NLP metrics that run **locally with zero judge cost**: **BLEU**, **ROUGE**, **METEOR**, **SacreBLEU**, **BERTScore**, and **cosine similarity**. These are reference-based and fully deterministic — useful as cheap complements to LLM judges.

### 2.6 The Metric Catalog Table
| Metric | Type | What it measures | Use case |
|--------|------|------------------|----------|
| **G-Eval** | Reference-free, LLM judge + CoT | Any custom criterion, scored 1–5 with reasoning | Custom quality rubrics, style/format checks |
| **Answer relevancy** | Reference-free, LLM judge | Does the answer address the question? | Off-topic/evasive answers |
| **Faithfulness** | Reference-free, LLM judge | Is the answer supported by the retrieval context? | Hallucination in RAG answers |
| **Hallucination** | Reference-free, LLM judge | Fraction of answer statements unverifiable in context | Hallucination (dual of faithfulness) |
| **Contextual relevancy** | Reference-free, LLM judge | Is the retrieved context relevant to the question? | Retrieval noise |
| **Contextual precision** | Reference-free, LLM judge | Are relevant chunks ranked first? | Retrieval ranking quality |
| **Contextual recall** | Reference-based, LLM judge | Does the context cover the ground truth? | Retrieval completeness (missing chunks) |
| **Bias** | Reference-free, LLM judge | Gender/racial/political bias in output | Safety gates |
| **Toxicity** | Reference-free, LLM judge | Hate speech, insults, toxicity | Safety gates |
| **Jailbreak detection** | Reference-free, LLM judge | Input is a jailbreak attempt | Guardrails |
| **Non-advice / misuse** | Reference-free, LLM judge | Output dispenses advice/abuse for regulated domains | Financial/medical chatbots |
| **Summarisation** | Reference-free, LLM judge | Summary alignment + coverage vs source | Meeting/email summarisers |
| **Task completion / role adherence / knowledge retention** | Reference-free, LLM judge | Agent behaviour in multi-turn conversations | Chatbots, agents |
| **RagasScore** | Mixed (RAGAS metrics) | Ragas' research metrics inside DeepEval | RAG eval using Ragas definitions |
| **BLEU / ROUGE / METEOR / SacreBLEU / BERTScore / cosine** | Reference-based, deterministic | N-gram/semantic overlap with a reference | Translation, keyword, exact-match checks |
| **CustomMetric** | User-defined | Any rubric you define with a scoring prompt | Organisation-specific criteria |
| **Composite metrics** | Aggregated | Combine several metrics into one score | One-number release gate |

---
## 3. Metric Deep-Dives
### 3.1 G-Eval — LLM-as-a-Judge with Chain-of-Thought
G-Eval is the evaluation method from the paper *"G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment"* (Liu et al., **arXiv:2303.16634**, Microsoft, 2023) — one of the most-cited LLM-as-a-judge papers. DeepEval implements it as the `GEval` metric, and the same CoT-judging machinery underpins several other DeepEval metrics. The method (per the paper and DeepEval docs):

1. **Define the evaluation task and criteria** — you specify the criterion in natural language. In DeepEval: `criteria="The answer must be factually consistent with the provided context"`, or a custom rubric.
2. **Generate chain-of-thought evaluation steps** — the judge LLM (or you, via `evaluation_steps=[...]`) produces a numbered list of steps to reason through before scoring. CoT makes the score **interpretable**: the judge explains its reasoning, which you can inspect.
3. **Score with a probability-weighted expectation** — the judge outputs a score 1–5 **and** a probability for each score. The final score is the **weighted sum**: `score = Σᵢ sᵢ · P(sᵢ)`. For a binary pass/fail, DeepEval applies a threshold over the resulting 0–1 scale.

```python
from deepeval.metrics import GEval
from deepeval.test_case import LLMTestCase

geval = GEval(
    name="Regulatory Tone",
    criteria="The answer uses plain, professional language suitable for a "
             "relationship manager; it must not give legal or investment advice.",
    evaluation_steps=[
        "1. Check the answer is in plain professional language.",
        "2. Check the answer does not give personalised financial advice.",
        "3. Score 1-5 and give probabilities for each score.",
    ],
    threshold=0.6,
)
test_case = LLMTestCase(input="Are guarantee fees capped?", actual_output="...")
geval.measure(test_case)
print(geval.score, geval.reason)   # e.g. 0.83, "The answer is professional..."
```

**Strengths:** any criterion expressible in natural language; CoT reasons are inspectable evidence. **Weaknesses:** the most expensive metric family (a full CoT judge call per evaluation); judge-model dependent — GPT-4-class judges align with humans well, smaller local judges degrade (see [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §11 for the cross-ref on G-Eval as a standalone framework). In a bank, pair G-Eval with judge calibration against human labels before trusting its numbers.

### 3.2 Faithfulness — The Answer → Context Groundedness Check
**Faithfulness** answers: *is every claim in the answer supported by the retrieved context?* It is the DeepEval counterpart of TruLens' **groundedness** ([trulens_guide.md](trulens_guide.md) §2) and Ragas' `faithfulness` ([ragas_guide.md](ragas_guide.md) §3.1). The procedure:

1. The judge LLM extracts the **atomic statements** from the answer.
2. Each statement is checked against the **retrieval context** (the chunks actually fed to the LLM): supported / not supported.
3. **Score = (number of supported statements) / (total statements in the answer).**

```python
from deepeval.metrics import FaithfulnessMetric
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input="What is the confirmation period for a standby LC?",
    actual_output="The confirmation period is 5 business days and fees are capped at 1.2%.",
    retrieval_context=[
        "Standby LCs: confirmation within 5 business days.",
        "Trade finance fee schedule: confirmation fee 1.2% max.",
    ],
)
metric = FaithfulnessMetric(threshold=0.8)
metric.measure(test_case)
print(metric.score, metric.verdicts)   # 1.0 here — both statements supported
```

**Why it matters for a bank:** a RAG assistant that invents a fee cap or a regulatory date is a compliance incident waiting to happen. Faithfulness catches *groundedness* failures — the answer is wrong *relative to what was retrieved* — which is the correct first gate. Its honest limit (shared with TruLens' groundedness): it cannot catch a *wrong-but-grounded* answer, i.e., a stale or incorrect document faithfully repeated — that is content-level validation, not evaluation (see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md)).

### 3.3 Answer Relevancy — The Question–Answer Alignment
**Answer relevancy** answers: *does the answer actually address the user's question?* The judge checks the answer against the input question and scores alignment (0–1), with a reason. Historically DeepEval used an embedding variant — generate N questions *from the answer* and average the cosine similarity between each generated question and the original question — and the current default is an LLM judge with explanations.

```python
from deepeval.metrics import AnswerRelevancyMetric

metric = AnswerRelevancyMetric(threshold=0.7)
metric.measure(LLMTestCase(
    input="What documents are needed for a documentary credit amendment?",
    actual_output="Our trade finance desk is open 9am-6pm SGT.",
))
print(metric.score)   # low — the answer never mentions documents
```

**Failure mode it catches:** the answer is fluent and even grounded, but irrelevant — e.g., the model answered a *different* question or gave boilerplate. Answer relevancy is the third leg of the RAG triad (context relevance → groundedness → answer relevance); a strong answer-relevancy score with strong faithfulness usually means a trustworthy answer for the user.

### 3.4 Contextual Relevancy — The Retrieved-Context Quality
**Contextual relevancy** answers: *is the retrieved context relevant to the question at all?* It evaluates **retrieval, not generation**:

1. The judge splits the retrieved context into statements.
2. Each statement is classified as relevant or irrelevant to the question.
3. **Score = (relevant statements) / (total statements in the context).**

```python
from deepeval.metrics import ContextualRelevancyMetric

metric = ContextualRelevancyMetric(threshold=0.7)
metric.measure(LLMTestCase(
    input="Are guarantee fees capped?",
    actual_output="Yes, capped at 0.5%.",
    retrieval_context=[
        "Guarantee fee schedule: cap 0.5% per annum.",       # relevant
        "Marketing brochure: our guarantees protect your cash flow.",  # noise
        "Staff onboarding handbook: how to use the CRM.",     # noise
    ],
))
print(metric.score)   # 1/3 ≈ 0.33 — retrieval pulled in noise
```

**Why it matters:** a low contextual-relevancy score with high faithfulness means the retriever is feeding the LLM noise, which degrades user experience and *invites* future hallucination even when today's answer is correct. It is the signal-vs-noise metric; fix it with rerankers, chunking, and hybrid search (§9.5).

### 3.5 Contextual Precision and Contextual Recall — The Retrieval Split
Where contextual relevancy measures *how much of the window is relevant*, the two RAG-specific metrics measure *where* and *whether*:

- **Contextual precision** — are the relevant chunks ranked **at the top**? For each retrieved chunk (in rank order), compute precision@k; the metric rewards relevant chunks appearing early and punishes irrelevant chunks before relevant ones. **Reference-free.**
- **Contextual recall** — does the retrieved context **cover** everything the ground-truth answer needs? The judge checks how much of `expected_output` (ground truth) is attributable to the retrieved context. **Reference-based** — requires `expected_output`.

Together they decompose retrieval quality the way Ragas does ([ragas_guide.md](ragas_guide.md) §3.2–3.3): **precision → reranker/ranking fixes; recall → chunking/index/coverage fixes.**

### 3.6 Metric Formulas — Scoring Summary
| Metric | Formula (conceptual) | Range | Reference needed? |
|--------|----------------------|-------|-------------------|
| **G-Eval** | `Σᵢ sᵢ · P(sᵢ)` over scores 1–5 (normalised); CoT first | 0–1 | No |
| **Faithfulness** | `supported statements / total statements in answer` | 0–1 | No |
| **Hallucination** | `1 − (verifiable statements / total statements)` (unverifiable share) | 0–1 | No |
| **Answer relevancy** | Judge alignment score (or mean cosine of generated questions vs input) | 0–1 | No |
| **Contextual relevancy** | `relevant context statements / total context statements` | 0–1 | No |
| **Contextual precision** | Rank-weighted precision over relevant chunks (avg of precision@k at relevant ranks / total relevant) | 0–1 | No |
| **Contextual recall** | `ground-truth statements attributable to context / total ground-truth statements` | 0–1 | Yes (expected_output) |
| **Bias / toxicity / jailbreak / non-advice** | Judge verdict (violations → low score) | 0–1 | No |
| **BLEU / ROUGE / BERTScore** | Standard NLP overlap/similarity | 0–1 | Yes (reference) |

> All LLM-judge metrics return a score in 0–1 plus `reason`/`verdicts` for auditability; each metric takes a `threshold` used by `assert_test()` for pass/fail.

### 3.7 Metric Examples — Worked Calculations

**Example A — Faithfulness (statement verdicts).** Answer: *"The confirmation period is 5 business days (1). Fees are capped at 1.2% (2). Regulatory approval is required within 24 hours (3)."* Context supports (1) and (2) but not (3). Score = 2/3 ≈ **0.67** → below a 0.8 threshold → test fails. The verdicts tell the developer *exactly which sentence* is ungrounded — the fix is either retrieval (find the regulation page) or prompt (don't extrapolate).
**Example B — Contextual relevancy (window noise).** Context has 5 statements; the judge marks 3 relevant. Score = 3/5 = **0.60**.
**Example C — Contextual precision (ranked retrieval).** Retrieved 3 chunks; relevant ones sit at ranks 2 and 3. Precision@2 = 1/2, precision@3 = 2/3; metric ≈ (1/2 + 2/3)/2 ≈ **0.58** — the relevant info exists but is buried, so ranking needs work (reranker).
**Example D — G-Eval (probability-weighted score).** Judge outputs: score 4 with P=0.7, score 5 with P=0.2, score 3 with P=0.1 → expectation = 4·0.7 + 5·0.2 + 3·0.1 = 2.8 + 1.0 + 0.3 = **4.1/5 → 0.82**.


## 4. Test-Driven Evaluation
### 4.1 `assert_test` — Unit Tests for LLM Apps
The heart of DeepEval is **`assert_test()`** — a pytest-style assertion that turns evaluation into a **deterministic pass/fail**:

```python
from deepeval import assert_test
from deepeval.metrics import FaithfulnessMetric
from deepeval.test_case import LLMTestCase

test_case = LLMTestCase(
    input="What is Crédit Agricole's approach to sustainable finance?",
    actual_output="CA-CIB arranges green and sustainability-linked loans and publishes "
                  "an annual sustainable-finance report.",
    retrieval_context=[
        "CA-CIB: Green and Sustainable Finance — annual report.",
        "CA-CIB sustainability: loans, bonds, advisory.",
    ],
)

def test_sustainable_finance_answer():
    assert_test(test_case, [FaithfulnessMetric(threshold=0.7)])
```

Verified semantics: `assert_test(test_case, metrics)` runs every metric against the test case, and **raises `AssertionError` on any score below its threshold** — exactly like a unit-test assertion, but for LLM behaviour. Because it is built on pytest, you get parametrisation, fixtures, marks, and `pytest test_llm.py` for free.

### 4.2 The Test Cases — `LLMTestCase` and `ConversationalTestCase`
- **`LLMTestCase`** (single-turn) — fields: `input` (user query), `actual_output` (the app's answer), `retrieval_context` (list of retrieved chunks, for RAG metrics), `expected_output` (ground truth, for reference-based metrics), plus optional `context`/`tools_called` for agent tests.
- **`ConversationalTestCase`** (multi-turn) — a conversation history + final output, used by conversation/agent metrics (bias, toxicity, task completion, role adherence…).
- **`EvaluationDataset`** — a collection of test cases (or goldens) that can be parametrised into pytest tests, pushed/pulled from Confident AI, or evaluated in bulk with `evaluate()`.

### 4.3 The Test Run — `deepeval test run`
```bash
deepeval test run tests/test_llm.py
deepeval test run tests/test_llm.py --verbose          # per-metric details
deepeval test run tests/ -n 4                           # parallel
```

Verified: `deepeval test run` is the CLI test runner, built **on top of pytest** (pytest is the underlying engine), with DeepEval orchestration on top (metric caching, result persistence, tracing). A passing run exits 0; **any failed assertion exits non-zero** — the property that makes it a CI gate. The CLI also offers:

- `deepeval login` / `deepeval logout` — connect the Confident AI platform;
- `deepeval dataset push/pull <alias>` — manage evaluation datasets;
- `deepeval report` — export results (e.g., JSON) for downstream gates;
- `deepeval gate` — check a project against a governance policy file;
- `deepeval inspect` — a TUI for browsing test results;
- `deepeval diagnose` — environment/model/settings diagnostics.

### 4.4 The CI Integration — Eval Gates
Because the runner is pytest + exit codes, CI integration is a two-step job (GitHub Actions example in §6.2):

```yaml
- run: pip install -U deepeval
- run: deepeval test run tests/test_llm.py
  env: { OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }} }
```

The pipeline step **fails when a metric regresses below its threshold** — an eval gate. The same test file you run locally runs on every push/PR, so prompt, model, or retriever changes get regression-tested automatically.

### 4.5 The Test-Driven Workflow — Write → Run → Fail → Fix → Pass
DeepEval's documentation frames the loop explicitly as unit-testing:

1. **Write the test** — encode the requirement as an `LLMTestCase` + metric + threshold (e.g., "the support bot must never contradict the fee schedule": faithfulness ≥ 0.85).
2. **Run** — `deepeval test run`; expect **red** on first run (that is the point — the test encodes an unmet requirement).
3. **Fail and diagnose** — the metric `reason`/`verdicts` say *why*: an ungrounded sentence, irrelevant chunks, off-topic answer.
4. **Fix** — improve the prompt, retriever (rerank/hybrid), chunking, or model.
5. **Pass and lock in** — green locally, then the same test gates CI, so the requirement stays enforced.

This is "eval-driven development": the TDD red-green-refactor cycle translated into probabilistic terms, with the eval suite as living documentation of what *good* means for your app.

### 4.6 Code Sketch — The Full Loop
```python
# tests/test_tf_support.py
import pytest
from deepeval import assert_test
from deepeval.metrics import FaithfulnessMetric, AnswerRelevancyMetric, ContextualRelevancyMetric
from deepeval.test_case import LLMTestCase
from deepeval.dataset import EvaluationDataset

dataset = EvaluationDataset(alias="TF_Support_Goldens", test_cases=[...])  # golden set

@pytest.mark.parametrize("test_case", dataset.test_cases)
def test_tf_assistant(test_case):
    assert_test(test_case, metrics=[
        FaithfulnessMetric(threshold=0.85),
        AnswerRelevancyMetric(threshold=0.80),
        ContextualRelevancyMetric(threshold=0.75),
    ])
```

```bash
deepeval test run tests/test_tf_support.py --verbose
# 42 passed in 3m12s   (or: 1 failed — metric FaithfulnessMetric: 0.67 < 0.85)
```

---
## 5. Usage
### 5.1 Install
```bash
pip install -U deepeval        # Python >= 3.9; verified — pip install deepeval
deepeval set-openai --model gpt-4o   # set default judge model (or set OPENAI_API_KEY env var)
```

DeepEval runs evaluations **locally** on your environment. Judge models can be OpenAI, Azure OpenAI, Anthropic, Gemini, Ollama (local), vLLM, or any custom LLM. For local/on-prem judging (a real requirement in banking), point a metric at a self-hosted endpoint — e.g., an Ollama or vLLM-served model — so evaluation data never leaves the environment.

### 5.2 `evaluate()` — Dataset, Metrics, Results
For bulk evaluation **without** pytest (notebooks, batch scoring), DeepEval provides `evaluate()`:

```python
from deepeval import evaluate
from deepeval.metrics import FaithfulnessMetric, AnswerRelevancyMetric
from deepeval.test_case import LLMTestCase

results = evaluate(
    test_cases=[test_case_1, test_case_2, ...],          # or an EvaluationDataset
    metrics=[FaithfulnessMetric(), AnswerRelevancyMetric()],
)
```

`evaluate()` runs every metric against every test case and returns a `TestResult` object with **per-test-case, per-metric scores** and aggregate statistics (mean per metric, pass/fail counts against thresholds). This is the same scoring engine `assert_test()` uses — `assert_test()` adds the pytest assertion semantics on top.

### 5.3 The Dataset — Format and Goldens
- **`EvaluationDataset(alias="...", test_cases=[...])`** — a named collection of `LLMTestCase` objects. The canonical structure: `input` + `actual_output` (+ `retrieval_context`, `expected_output` where the metric needs them).
- **Goldens** — the curated, human-reviewed subset used for release gates. In CI, you typically parametrise pytest over the dataset's goldens (see §4.6), so every golden is one test case.
- **`deepeval dataset push <alias>` / `dataset.pull(alias=...)`** — sync datasets with Confident AI so the whole team runs the same goldens.
- **Synthesizer** — when you lack labelled data, `Synthesizer().generate(context_documents=[...], num_cases=N)` produces synthetic `input`/`expected_output`/`context` test cases from your own documents (stratified question types). Treat synthetic sets as *coverage* for dev iteration; keep a human-reviewed golden subset for gates.

### 5.4 Results — What You Actually Get
For each metric, per test case: **`score` (0–1)**, **`threshold`**, **`reason`** (judge explanation), and **`verdicts`** (per-statement evidence for faithfulness-style metrics). Aggregates give mean score and pass-rate per metric. In CI, the JSON report (`deepeval report`) feeds downstream dashboards or custom gates; on the Confident AI platform, full test runs (scores + traces + reasons) are stored and diffable across commits.

### 5.5 Usage Example — Evaluate a RAG App (Complete Sketch)
```python
# eval_rag.py — run a RAG app over a dataset, score it, print results
from deepeval import evaluate
from deepeval.metrics import (
    FaithfulnessMetric, AnswerRelevancyMetric,
    ContextualPrecisionMetric, ContextualRecallMetric,
)
from deepeval.test_case import LLMTestCase

def run_rag(query: str):
    """Your RAG app: retrieve chunks, generate answer. Returns (answer, chunks)."""
    chunks = retriever.retrieve(query)          # e.g. hybrid search, top-k
    answer = generator.generate(query, chunks)  # your LLM call
    return answer, [c.text for c in chunks]

test_cases = []
for q, expected in golden_set:                  # question + ground-truth reference
    answer, chunks = run_rag(q)
    test_cases.append(LLMTestCase(
        input=q, actual_output=answer,
        retrieval_context=chunks, expected_output=expected,
    ))

results = evaluate(test_cases, metrics=[
    FaithfulnessMetric(threshold=0.85),          # hallucination gate
    AnswerRelevancyMetric(threshold=0.80),       # off-topic gate
    ContextualPrecisionMetric(threshold=0.70),   # ranking gate
    ContextualRecallMetric(threshold=0.80),      # coverage gate (needs expected_output)
])

for tc, scores in results.test_results.items():
    print(tc.input, {m: round(s.score, 2) for m, s in scores.items()})
print(results.get_aggregate_scores())            # {'faithfulness': 0.91, ...}
```

---

## 6. CI/CD and the Dashboard
### 6.1 Eval Gates — The Concept
An **eval gate** is a CI check that fails the build when a metric drops below a threshold. DeepEval makes this native: `deepeval test run` returns a non-zero exit code on any failed assertion, so a standard pipeline step *is* the gate. The gate runs the same goldens on every PR touching prompt/retriever/model — the regression-testing pattern from [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md). Verified in the docs: "deepeval plugs into pytest via `assert_test()` and the `deepeval test run` command, so every push (or every PR) runs the same evals you'd run locally."

### 6.2 `deepeval test run` in CI — GitHub Actions
```yaml
# .github/workflows/eval.yml
name: LLM Eval Gate
on:
  pull_request:
    paths: ['app/**', 'prompts/**', 'retriever/**', 'tests/**']   # trigger on LLM-relevant changes
jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -U deepeval
      - name: Run eval suite (golden set)
        run: deepeval test run tests/test_tf_support.py --verbose
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}   # or a self-hosted judge endpoint
      - name: Upload results to Confident AI (optional)
        run: deepeval dataset push TF_Support_Goldens && deepeval login && deepeval test run tests/ --report
        if: always()
```

Operational notes for real pipelines: set generous timeouts (LLM judge calls are slow — 10–15 min for medium suites), cache judge responses where safe, and run a **smoke subset** on every commit with the full golden suite on merges to a release branch (see [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) for staging strategies).

### 6.3 The Dashboard — Confident AI (Hosted Platform)
**Confident AI** is the production platform built on DeepEval. Verified capabilities (GitHub README + Confident AI docs):

1. **Centralised test-run storage** — upload test results with full metric details, reasons, and LLM traces; view them in dashboards instead of terminal output.
2. **Debugging via traces** — inspect the exact retrieval/LLM calls behind a failing test case.
3. **Hyperparameter comparison** — compare prompt templates, models, chunk sizes, etc., across eval runs and pick the winner by score.
4. **Dataset management** — create, version, and centralise evaluation datasets (`deepeval dataset push/pull`).
5. **Production monitoring** — track events from live traffic and continuously evaluate, augmenting the eval dataset over time.

Connection is opt-in: `deepeval login` (CLI) enables pushing runs; without it, everything still runs locally. The platform is the paid layer — the OSS framework is free. For a bank, this means: local/on-prem evaluation is fully supported; only if you want team dashboards and hosted trace storage do you involve the cloud (and the associated data-governance review).

### 6.4 The Local Experience — Logs and Reports
Without any platform, you still get: per-test console output (`--verbose` shows per-metric score, reason, verdicts), a `deepeval inspect` TUI to browse results, and **JSON reports** (`deepeval report --format json`) that downstream tooling can parse — e.g., a shell gate: `jq -e '.failures | length == 0'`. Local results can also be written to your own artifact storage, which is what an air-gapped bank would do.

### 6.5 The CI Table
| Stage | Command | Gate |
|-------|---------|------|
| **Install** | `pip install -U deepeval` | — |
| **Unit eval (per commit)** | `deepeval test run tests/smoke/ --verbose` | Exit code ≠ 0 on any threshold failure |
| **Full eval (PR / release branch)** | `deepeval test run tests/ -n 4` | Same gate over the golden set |
| **Report** | `deepeval report --format json` | Parse JSON; fail if failures > 0 |
| **Governance policy** | `deepeval gate` | Enforce project eval policy (required metrics/thresholds) |
| **Dataset sync** | `deepeval dataset push <alias>` | Version goldens alongside code |
| **Platform upload (optional)** | `deepeval login` + run with `--report` | Dashboards, traces, hyperparameter compare |

---

## 7. Ecosystem and Integrations
### 7.1 Framework Integrations — LangChain, LlamaIndex, OpenAI Agents, CrewAI
DeepEval plugs into the major LLM frameworks (verified from the README and docs):

- **LangChain** — a `CallbackHandler` integration (`deepeval.integrations.langchain`) captures traces from LangChain chains/agents, enabling evals with tracing on LangChain apps.
- **LlamaIndex** — `instrument_llama_index` registers a handler with LlamaIndex's instrumentation dispatcher so all spans/events are captured automatically; LlamaIndex's own docs show `deepeval.integrations.llama_index` wrappers (e.g., `FunctionAgent`) for tracing + evaluating LlamaIndex agents.
- **OpenAI Agents / CrewAI** — integration points for agent traces and evals (agent evaluation context: [autonomous_agents_guide.md](../autonomous_agents_guide.md) §5).
- **Hugging Face** — real-time evaluations during LLM fine-tuning.

### 7.2 DeepEval + Ragas — `RagasScore`
DeepEval ships a **native RAGAS integration** (`deepeval.metrics.ragas.RagasScore`): you can use Ragas' research metrics (e.g., `context_precision`, `context_recall` — the library from [ragas_guide.md](ragas_guide.md)) inside `assert_test()` and `evaluate()`, combining **Ragas' metric definitions with DeepEval's test runner and CI gating**. This is a genuinely useful bridge: metric depth from Ragas, unit-test ergonomics from DeepEval.

### 7.3 Synthesizer — Evaluation Data Generation
`deepeval.synthesizer.Synthesizer` generates synthetic test cases from your own documents (context documents → `input`/`expected_output`/`context`), closing the no-labelled-data gap for new apps. Combined with the golden-set discipline (§5.3), it supports the data-generation discussion in [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md).

### 7.4 Community — GitHub (Stars — Flag)
- **~17,500 GitHub stars as of August 2026** (star-history.com shows 17.5k, Global Rank ~#2600) — **approximate and moving fast**; vendor marketing cited 15.3k+ in May 2026 and "3M+ monthly PyPI downloads" (vendor claim, unverified independently).
- ~1,800+ merged PRs, ~460+ closed issues, 58 releases by July 2026 (repositorystats.com) — an active, release-cadence-driven project (~monthly releases).
- Smaller than LangSmith's ecosystem, but among the largest open-source LLM-eval communities (comparable to Ragas; larger than TruLens by stars).

### 7.5 Docs
- **Primary docs: [deepeval.com/docs](https://deepeval.com/docs)** (getting started, metrics introduction, unit-testing in CI/CD, CLI reference).
- **Confident AI platform docs: [confident-ai.com/docs](https://www.confident-ai.com/docs)**.
- **Verification note:** the task brief referenced `docs.confident-ai.com`; as of August 2026 the documentation is served from **deepeval.com/docs** with the platform docs on **confident-ai.com/docs** — `docs.confident-ai.com` may redirect or be legacy. Verify the URL before citing it in a formal deck.

### 7.6 License
**Apache 2.0** — verified from the repository metadata. Permissive, patent-grant clause, commercial-use friendly, no copyleft. One of the safest licences for embedding in a bank's internal tooling; the Confident AI platform itself is a separate commercial product.

### 7.7 Versions
- **Versioning: 4.x as of mid-2026** (e.g., `deepeval 4.0.7` on PyPI, June 2026 per piwheels; 58 releases, latest July 2026, "Flaky tests? Skip the failures!").
- **Flag:** earlier series material (e.g., [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §5, written when DeepEval was on 0.x) lists "pre-1.0, breaking changes possible" as a weakness. That is now **outdated**: the project moved past 1.0 to a 4.x line, and the 2026 release notes centre on UX concerns (flaky-test handling) rather than API churn — but pin your dependency (`deepeval==4.0.x`) and re-validate on upgrade, as with any fast-moving OSS.
- **TypeScript SDK** — DeepEval for TypeScript was announced as fully open-source in 2026 (`npm install -D deepeval`, `npx deepeval test run llm_app.test.ts`), which removes the "Python-only" limitation noted in earlier guides.
---
## 8. Comparison with Alternatives
### 8.1 The Comparison Table
| Tool | Type | Metrics | Test-driven | LLM judges | CI | Dashboard | Open-source | Best for |
|------|------|---------|-------------|------------|----|-----------|-------------|----------|
| **DeepEval** | OSS library (Apache 2.0) | 50+ (RAG, quality, safety, agents, deterministic) | ✅ `assert_test()` + pytest + `deepeval test run` | ✅ any model, incl. local | ✅ exit-code gates, GA-ready | ⚠️ via Confident AI (optional) | ✅ | Test-driven eval, CI gates, regression testing |
| **TruLens** | OSS library (MIT) | RAG triad + feedback functions | ⚠️ via scripts/asserts, not pytest-native | ✅ any provider | ⚠️ script gates | ✅ free local Streamlit | ✅ | Triad + traces + local dashboard — see [trulens_guide.md](trulens_guide.md) |
| **Ragas** | OSS library (Apache 2.0) | RAG research metrics + testset gen | ⚠️ via `evaluate()` + custom asserts | ✅ | ⚠️ script gates | ❌ | ✅ | RAG metric depth, synthetic testsets — see [ragas_guide.md](ragas_guide.md) |
| **LangSmith** | Commercial SaaS | Eval suite + tracing | ⚠️ via SDK | ✅ | ✅ native | ✅ managed | ❌ | Managed observability in LangChain shops |
| **Langfuse** | OSS + cloud (MIT) | Eval on traces | ⚠️ | ✅ | ⚠️ | ✅ self-host/cloud | ✅ | Tracing/LLM-ops platform + eval — see [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md) |

Cross-references: the master tooling table with MLflow, Arize Phoenix, W&B, Galileo and the DeepEval-vs-each comparison lives in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) (§5 DeepEval, §6 LangSmith, §8 RAGAS); the RAG-eval-specific comparison is in [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md).

### 8.2 DeepEval Strengths
1. **Test-driven by design** — `assert_test()` + thresholds = deterministic red/green. If you know pytest, you know DeepEval; no separate eval DSL to learn.
2. **CI-native** — `deepeval test run` with exit-code gating, JSON reports, governance gate (`deepeval gate`), GitHub Actions examples. The strongest CI story among OSS eval frameworks.
3. **50+ metrics, model-agnostic judges** — the widest built-in catalog in OSS, and any LLM can judge (OpenAI, Anthropic, Gemini, Ollama, vLLM) — evaluation can stay fully on-prem.
4. **RAG + agent + safety coverage** — faithfulness/hallucination, contextual precision/recall/relevancy, bias/toxicity/jailbreak, conversation and agent metrics — one framework for most gates.
5. **RAGAS bridge** — native `RagasScore` gives Ragas metric definitions under DeepEval's runner.
6. **Open and permissive** — Apache 2.0, local by default, optional cloud — bank-friendly licensing and deployment.
7. **Active project** — ~17.5k stars, ~monthly releases, TS SDK since 2026.

### 8.3 DeepEval Weaknesses (Honest)
1. **LLM-judge cost and latency** — advanced metrics (G-Eval, faithfulness, hallucination) make a full CoT judge call per test case; large golden sets get expensive and slow (mitigate: judge caching, smoke vs full suites, local judges).
2. **Judge variability** — scores depend on the judge model and prompt; without calibration against human labels (kappa-style agreement checks), thresholds are guesswork. DeepEval gives you the machinery, not the calibration discipline.
3. **Dashboard/monitoring is paywalled** — local eval is free and complete, but team dashboards, trace storage, and production monitoring require the **Confident AI** platform (cloud, paid, data-governance review in a bank).
4. **Maturity nuance (updated)** — no longer pre-1.0 (4.x since 2026, see §7.7), but still fast-moving: pin versions, re-validate on upgrade, and some advanced docs (composite metrics, custom rubric authoring) remain thinner than the core docs.
5. **Ecosystem smaller than LangSmith/MLflow** — fewer third-party tutorials and fewer managed integrations; Python-first culture (TS SDK is new).
6. **Metric semantics differ from Ragas/TruLens** — e.g., DeepEval's faithfulness vs Ragas' faithfulness vs TruLens' groundedness are *similar but not identical*; don't compare raw numbers across tools without understanding each definition.

### 8.4 When to Pick Which
| Need | Pick |
|------|------|
| **Unit-test culture, CI eval gates, regression testing** | **DeepEval** — pytest-native assertions and exit-code gates are unmatched in OSS |
| **RAG metric depth + synthetic testset generation** | **Ragas** — research-grounded per-failure-mode metrics; pair its metrics into DeepEval via RagasScore if you want both |
| **Triad + tracing + a free local dashboard** | **TruLens** — feedback functions, instrumentation, Streamlit dashboard out of the box |
| **Managed observability + eval in a LangChain-heavy team** | **LangSmith** — commercial but deepest LangChain integration |
| **Production tracing platform with eval on top, self-hostable** | **Langfuse** — LLM-ops/tracing first, eval second |
| **Banking default combo** | DeepEval for CI gates + Ragas for RAG metric depth (+ TruLens traces where per-record evidence is needed for validation files) |

---

## 9. Worked Example — Evaluating a Banking RAG Assistant
### 9.1 Scenario (Same as the TruLens and Ragas Guides)
Crédit Agricole CIB's trade-finance desk wants a **product-support RAG assistant**: front-office users ask about trade-finance products (standby LCs, documentary credits, guarantees, FX forwards, commodity finance) and the assistant answers from the **product documentation corpus** (term sheets, operational handbooks, regulatory summaries). Requirements: answers must be **accurate** (no invented fees/limits), **grounded** (traceable to a source document), and **relevant** (address the actual question). Same scenario as [trulens_guide.md](trulens_guide.md) §9 (triad + traces) and [ragas_guide.md](ragas_guide.md) §9 (metric depth) — here evaluated with DeepEval's **test-driven, CI-gated** workflow.

### 9.2 Setup — Metrics
Three metrics, each a gate with a calibrated threshold:

```python
# eval_metrics.py
from deepeval.metrics import FaithfulnessMetric, AnswerRelevancyMetric, ContextualRelevancyMetric

faithfulness   = FaithfulnessMetric(threshold=0.85)      # hallucination gate — critical for a bank
answer_relev   = AnswerRelevancyMetric(threshold=0.80)   # off-topic answers
ctx_relevancy  = ContextualRelevancyMetric(threshold=0.75)  # retrieval noise
```

Rationale (mirrors the RAG triad, [trulens_guide.md](trulens_guide.md) §2): faithfulness ↔ groundedness, contextual relevancy ↔ context relevance, answer relevancy ↔ answer relevance. Add `ContextualPrecisionMetric`/`ContextualRecallMetric` for the retrieval-split diagnosis (rank vs coverage) once the baseline gates are green.

### 9.3 Test Cases — The Golden Set as `assert_test`
```python
# tests/test_tf_support.py
import pytest
from deepeval import assert_test
from deepeval.test_case import LLMTestCase
from deepeval.dataset import EvaluationDataset
from eval_metrics import faithfulness, answer_relev, ctx_relevancy

# 150 human-reviewed goldens: input, expected_output, retrieval_context (versioned in git)
dataset = EvaluationDataset(alias="TF_Support_Goldens", test_cases=[
    LLMTestCase(
        input="What is the confirmation period for a standby LC?",
        actual_output="5 business days, unless the terms specify otherwise.",
        expected_output="Confirmation period is 5 business days.",
        retrieval_context=["Standby LC handbook: confirmation within 5 business days."],
    ),
    # ... 149 more, incl. edge cases: "Are guarantee fees capped?", 
    #     "What documents are needed for a documentary credit amendment?"
])

@pytest.mark.parametrize("tc", dataset.test_cases)
def test_tf_assistant(tc):
    assert_test(tc, metrics=[faithfulness, answer_relev, ctx_relevancy])
```

### 9.4 The Run — `deepeval test run`
```bash
deepeval test run tests/test_tf_support.py --verbose
```

Illustrative v1.0 result (naive vector search, coarse chunks):

```
tests/test_tf_support.py::test_tf_assistant[case_23] FAILED
  Metric ContextualRelevancyMetric: 0.58 < 0.75
  Reason: 4 of 7 retrieved statements are irrelevant to the question
  ("Are guarantee fees capped?"); retrieved marketing brochure and
  neighbouring product sheet instead of the fee schedule.
... 143 passed, 7 failed in 4m38s
```

The failed tests are **not** flaky — each failure carries a metric reason pointing at the failing stage: retrieval noise, not generation.

### 9.5 Analysis — The Failing Test and the Retrieval Fix
The failure signature is the classic one: **faithfulness and answer relevancy pass, contextual relevancy fails** (0.58). Diagnosis: retrieval is pulling in relevant chunks *and* noise — marketing material and adjacent product sheets bury the fee schedule. The LLM happens to answer correctly from the signal, but the window is dirty, which degrades UX and invites future hallucination.

Retrieval fixes (see [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) and [rag_optimization_techniques_guide.md](rag_optimization_techniques_guide.md)):

1. **Add a reranker** (cross-encoder) over the top-K candidates — directly lifts contextual relevancy/precision;
2. **Tune chunking** — table-aware, heading-aware splits so fee schedules and term sheets aren't split mid-row;
3. **Hybrid search** (BM25 + embeddings) with query rewriting ([query_rewriting_rag_guide.md](query_rewriting_rag_guide.md));
4. **Re-weight embedding choice** on the banking corpus.

Re-run the suite after each change: v1.1 (hybrid + reranker) lifts contextual relevancy to 0.83 and all 150 tests pass. The eval suite — identical tests, thresholds, and goldens — is what *proves* the fix worked and prevents regression when someone later touches the retriever.

### 9.6 The CI Gate — The Pipeline
```yaml
# .github/workflows/eval.yml — gate every PR touching prompts/retriever/model
on: { pull_request: { paths: ['app/**', 'prompts/**', 'retriever/**', 'tests/**'] } }
jobs:
  eval-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -U deepeval
      - run: deepeval test run tests/test_tf_support.py --verbose   # exit code = gate
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}   # or self-hosted judge endpoint
      - run: deepeval report --format json > eval_report.json
      - uses: actions/upload-artifact@v4
        with: { name: eval-report, path: eval_report.json }
```

A failed gate blocks the merge and links to the failing test + judge reason; the JSON report is archived as evidence. Version the golden set and thresholds with the code, and re-run the full suite on release branches (staging guidance: [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md)).

### 9.7 The Banking Context — Evaluation as Validation Evidence
For a bank, this eval suite is not just engineering hygiene — it is **validation evidence**:

- **Model validation (SR 11-7 / MAS FEAT accountability):** the eval specification — dataset version (150 goldens + review record), metric list, judge model + prompt versions, thresholds, per-sample scores and judge reasons — goes into the model-validation file as the "independent testing" artifact. The reproducibility of `deepeval test run` (same code, same goldens, same judge → same verdict) is exactly what a validation review wants to see.
- **Traceability:** faithfulness verdicts show each answer statement's support in a source document — the machine-readable version of "we can show where this answer came from".
- **Regression monitoring:** the same gates re-run on every change and periodically in production-sample mode; a drop in the 7-day rolling faithfulness triggers investigation (see [ai_agent_drift_guide.md](../ai_agent_drift_guide.md)).
- **Limits to state honestly:** eval scores measure *grounding in the retrieved context and relevance*, not *regulatory correctness* — a faithfully retrieved document can still be wrong or outdated. The eval-vs-validation distinction — what a score means vs what a sign-off requires — is the subject of [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md), with the governance overlay in [implementing-responsible-ai.md](../implementing-responsible-ai.md): automated DeepEval gates for continuous monitoring, human validation for formal sign-off.
---

## 10. Summary — DeepEval in One Page
**DeepEval** is the open-source (Apache 2.0), pytest-native LLM evaluation framework from **Confident AI** — its own description: *"similar to Pytest but specialized for unit testing LLM applications."* It turns LLM evaluation into the same discipline as software testing: write an `LLMTestCase` (input, output, retrieval context, ground truth), attach **50+ research-backed metrics** (G-Eval with chain-of-thought, faithfulness, answer relevancy, contextual relevancy, contextual precision/recall, hallucination, bias, toxicity, summarisation, RAGAS, deterministic NLP), assert with `assert_test()` against per-metric thresholds, and run everything with `deepeval test run` — whose exit code **gates CI**.
**Three things working together:**

1. **Metrics** — the widest built-in catalog in OSS, LLM-as-a-judge (any model, incl. local/on-prem) with explanations and per-statement verdicts, plus embedding-based and deterministic options.
2. **Test-driven evaluation** — `assert_test()` + pytest + parametrised goldens = deterministic red/green; the write → run → fail → fix → pass loop applied to prompts, retrievers, and models.
3. **CI/CD** — `deepeval test run` in GitHub Actions/GitLab/Jenkins with exit-code eval gates, JSON reports, `deepeval gate` governance, and the optional **Confident AI** platform for dashboards, traces, and production monitoring.

**Positioning:** DeepEval is the *test-driven, CI-friendly* option in the evaluation landscape — stronger than Ragas at gating (but Ragas' metric depth is available natively via `RagasScore`), more gate-native than TruLens (but TruLens has the free local dashboard and tracing), fully open-source unlike LangSmith, and bank-friendly (Apache 2.0, local execution, any judge model). Honest weaknesses: judge cost/variability (calibrate thresholds against human labels!), cloud-paywalled dashboards, and fast-moving versions (pin them).
**Final word:** if your team wants LLM applications tested the way code is tested — unit tests, red/green, CI gates, regression suites — DeepEval is the most direct open-source path. In a bank: gate CI on faithfulness/relevancy thresholds with a self-hosted or Bedrock judge, pin `deepeval==4.x`, file the eval spec (dataset version, metrics, judge, thresholds, per-sample scores) as validation evidence ([llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md)) — and "the assistant seems fine" becomes an auditable, reproducible claim.


## 11. Glossary
| Term | Definition |
|------|------------|
| **DeepEval** | Open-source (Apache 2.0) pytest-native LLM evaluation framework by Confident AI; 50+ metrics, `assert_test()`, `deepeval test run` CLI. |
| **Confident AI** | The company behind DeepEval; also the name of its hosted platform (dashboards, datasets, traces, production monitoring). |
| **G-Eval** | LLM-as-a-judge evaluation method (arXiv:2303.16634, Microsoft): natural-language criteria + chain-of-thought steps + probability-weighted score; implemented as `GEval`. |
| **LLM-as-a-judge** | Using an LLM to score LLM outputs; the engine behind most DeepEval metrics. |
| **Chain-of-thought (CoT)** | Step-by-step reasoning the judge performs before scoring; makes DeepEval scores interpretable (`reason`). |
| **Answer relevancy** | Metric: does the answer address the question (question–answer alignment)? |
| **Faithfulness** | Metric: fraction of the answer's statements supported by the retrieved context — the hallucination detector. |
| **Groundedness** | TruLens' term for the same concept as faithfulness: is the answer grounded in the retrieved context? |
| **Contextual relevancy** | Metric: fraction of retrieved-context statements relevant to the question (retrieval noise). |
| **Hallucination** | Metric (and failure mode): answer statements unverifiable against the context. |
| **Bias** | Safety metric: gender/racial/political bias in outputs. |
| **Toxicity** | Safety metric: hate speech, insults, toxic language in outputs. |
| **Contextual precision** | RAG metric: are relevant retrieved chunks ranked at the top (ranking quality)? |
| **Contextual recall** | RAG metric: does the retrieved context cover the ground truth (retrieval completeness; needs `expected_output`)? |
| **RAG metrics** | The retrieval/generation decomposition family: contextual precision, recall, relevancy, faithfulness, hallucination. |
| **assert_test** | DeepEval's unit-test primitive: `assert_test(test_case, metrics=[...])` — raises on any score below threshold. |
| **LLMTestCase** | The single-turn test-case object: `input`, `actual_output`, `retrieval_context`, `expected_output`, etc. |
| **ConversationalTestCase** | Multi-turn test case for chatbot/agent metrics. |
| **Test-driven** | Development loop applied to LLM apps: write the test → run → fail → fix → pass; eval suite as living requirements. |
| **pytest** | The Python test framework DeepEval builds on; `pytest test_llm.py` works directly. |
| **deepeval test run** | The CLI test runner (pytest under the hood); non-zero exit code on failures = CI gate. |
| **CLI** | Command-line interface: `deepeval test run`, `deepeval gate`, `deepeval report`, `deepeval dataset push`, `deepeval diagnose`, `deepeval inspect`. |
| **CI/CD** | Continuous integration/delivery; DeepEval gates run evals on every push/PR. |
| **Eval gate** | A CI check that fails the build when a metric score drops below its threshold. |
| **GitHub Actions** | CI platform; DeepEval gates are standard workflow steps (`pip install deepeval` + `deepeval test run`). |
| **Dashboard** | Result visualisation; DeepEval's is the hosted Confident AI platform (optional) — no local UI, unlike TruLens. |
| **Confident AI (platform)** | Hosted layer: centralised test runs, dashboards, traces, hyperparameter comparison, production monitoring. |
| **EvaluationDataset** | Named collection of test cases/goldens; push/pull to Confident AI; parametrised into pytest. |
| **Synthesizer** | `deepeval.synthesizer.Synthesizer`: generate synthetic test cases from your documents. |
| **RagasScore** | Native DeepEval integration running RAGAS metrics inside `assert_test()`/`evaluate()`. |
| **LlamaIndex** | Data/RAG framework; DeepEval integration (`instrument_llama_index`, wrappers) for tracing + evals. |
| **LangChain** | LLM-application framework; DeepEval `CallbackHandler` integration captures traces for evals. |
| **RAGAS** | The RAG metric library from Vibrant Labs — see [ragas_guide.md](ragas_guide.md). |
| **Apache 2.0** | Permissive open-source licence used by DeepEval (and Ragas/Phoenix); commercial-use friendly. |
| **TruLens** | OSS (MIT) eval + tracing + dashboard library; the RAG-triad tool — see [trulens_guide.md](trulens_guide.md). |
| **LangSmith** | LangChain's commercial observability/eval platform. |
| **Langfuse** | OSS + cloud tracing/LLM-ops platform with eval on top — see [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md). |
| **Validation** | The formal, human sign-off process that eval evidence feeds into — see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md). |
| **Regression** | A quality drop caused by a change; regression testing = re-running goldens on every change to catch it. |
| **Golden set** | Curated, human-reviewed test cases with verified references — the authoritative subset for gates/sign-off. |

---
*Companion guides: [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) (master tooling comparison — DeepEval §5) · [LLM Evaluation vs Validation](../llm_evaluation_vs_validation_guide.md) (eval as validation evidence) · [TruLens](trulens_guide.md) & [Ragas](ragas_guide.md) (sibling deep-dives) · [RAG Evaluation Tools Comparison](rag_evaluation_tools_comparison_guide.md) & [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) (the RAG-eval series) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) & [RAG Optimization Techniques](rag_optimization_techniques_guide.md) & [Query Rewriting RAG](query_rewriting_rag_guide.md) (retrieval fixes) · [AI Agent Drift](../ai_agent_drift_guide.md) (production monitoring) · [Autonomous Agents](../autonomous_agents_guide.md) (§5 agent evals) · [Implementing Responsible AI](../implementing-responsible-ai.md) (governance overlay) · [LLM Guard Models](../llm_guard_models_guide.md) (safety layers).*
