# TruLens — The Open-Source LLM Evaluation and Observability Library — A Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) · [LLM Evaluation vs Validation](../llm_evaluation_vs_validation_guide.md) · [AI Agent Drift](../ai_agent_drift_guide.md) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [Autonomous Agents](../autonomous_agents_guide.md) · [Responsible AI](../implementing-responsible-ai.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [TruLens Overview](#1-trulens-overview)
2. [The RAG Triad](#2-the-rag-triad)
3. [Feedback Functions](#3-feedback-functions)
4. [Instrumentation and Tracking](#4-instrumentation-and-tracking)
5. [The Dashboard and Evaluation](#5-the-dashboard-and-evaluation)
6. [Usage and Quickstart](#6-usage-and-quickstart)
7. [Ecosystem and Community](#7-ecosystem-and-community)
8. [Comparison with Alternatives](#8-comparison-with-alternatives)
9. [Worked Example — Evaluating a Banking RAG Assistant](#9-worked-example--evaluating-a-banking-rag-assistant)
10. [Summary — TruLens in One Page](#10-summary--trulens-in-one-page)
11. [Glossary](#11-glossary)

---

## 1. TruLens Overview

### 1.1 What TruLens Is

**TruLens** (from "Tru" + "lens") is an open-source library for the **evaluation and observability (tracing/tracking) of LLM applications** — RAG pipelines, agents, and any instrumented Python app that calls an LLM. It was originally created by the AI observability company **TruEra** and is today developed in the open under the `truera/trulens` GitHub repository, with documentation at [trulens.org](https://www.trulens.org).

Its own current tagline is a good summary of what it actually does:

> "TruLens finds where your agent fails and where you can cut cost without losing quality. Open source, OpenTelemetry-native. Instrument any app with a decorator, score every step with LLM judges that explain themselves, then compare versions and ship the one that earns it."

In practice, TruLens provides three things that work together:

| Capability | What it does |
|------------|--------------|
| **Feedback functions** | Programmatic and LLM-as-a-judge metrics (context relevance, groundedness, answer relevance, etc.) that score every step of an app run |
| **Instrumentation / tracing** | Automatic capture of every LLM call, retrieval, tool invocation, and function call as structured spans — OpenTelemetry-native since the 1.x line |
| **Dashboard** | A local, Streamlit-based UI to browse app leaderboards, per-record traces, feedback scores over time, and cost/latency |

The combination of *evaluate* (feedback functions) + *trace* (instrumentation) + *dashboard* is what distinguishes TruLens from a pure metrics library (like RAGAS) or a pure tracing backend (like Langfuse): **every evaluation is attached to a concrete, inspectable record of what the app actually did**.

### 1.2 Origin and History — TruEra

- **TruEra** was founded in **2021** as an AI observability company (led by CEO William Ussler, with co-founders from the UC Berkeley AI community — verify the exact co-founder list before quoting it in a formal deck). TruEra's products evolved from traditional ML model monitoring into **generative AI observability**: evaluating and monitoring LLM apps across the full lifecycle from development to production.
- **TruLens** was open-sourced as TruEra's self-service evaluation library, first published on PyPI in **2022** (originally as the `trulens_eval` package). It became one of the earliest widely known LLM evaluation tools, popularised together with the **RAG triad** concept (see §2).
- **Snowflake acquisition (2025):** Snowflake entered a definitive agreement to acquire **TruEra's AI Observability Platform** (the managed enterprise product), announced in 2025. Importantly, the acquisition covered the *managed platform*, and **TruLens itself remained open source** — per the project's own statement, "since TruEra's acquisition by Snowflake, Snowflake now actively oversees and supports the development of TruLens in open source." So the current ownership situation is: **open-source project stewarded under truera/trulens, backed by Snowflake**, with the commercial observability platform living in Snowflake's data cloud (see [enterprise_ai_platforms_guide.md](../enterprise_ai_platforms_guide.md)).

> **Verification note:** the acquisition was announced in 2025; check the [Snowflake announcement](https://www.snowflake.com/en/blog/snowflake-acquires-truera-to-bring-llm-ml-observability-to-data-cloud/) for the exact close date. The fact that TruLens stayed OSS is confirmed by the project's own site and by contemporaneous reporting (InfoWorld noted TruLens was "not likely to be part of the Snowflake deal").

### 1.3 Open Source and License

- **License: MIT** — verified from the `truera/trulens` repository (`LICENSE` file). MIT is permissive: free to use, modify, and embed commercially, no copyleft obligations. This matters for banks: it is one of the few genuinely permissive licences in the LLM-eval space (DeepEval/RAGAS/Arize Phoenix are Apache-2.0; LangSmith is closed-source; Langfuse is MIT).
- **Cost:** the core library, dashboard, and feedback functions are entirely free. You only pay for the **judge model calls** when using LLM-as-a-judge feedback (e.g., an OpenAI or Bedrock model scoring your answers) — or for whatever retrieval/embedding infrastructure your app already uses.
- **No telemetry lock-in:** because instrumentation is OpenTelemetry-native, traces can be exported to any OTLP-compatible backend (Jaeger, Grafana Tempo, Datadog) without being locked into TruLens' own storage.

### 1.4 Positioning — "Evaluate + Trace"

TruLens' design philosophy is that **evaluation without tracing is un-debuggable, and tracing without evaluation is un-actionable**:

1. **Instrument** the app (decorator or framework wrapper) → every LLM call, retrieval, and tool call becomes a structured record.
2. **Define feedback functions** → metrics that score inputs, intermediate steps, and outputs (the RAG triad being the canonical set).
3. **Run and evaluate** → each app record gets scored inline (or offline) and stored.
4. **Inspect in the dashboard** → leaderboard of app versions, per-record traces, score time series, cost and latency per step.

The result is a tight loop: when a score drops, you can click into the exact record, see the retrieved chunks and the prompt, and identify *which step* caused the failure — rather than staring at a number.

### 1.5 TruLens vs the Alternatives (Overview)

TruLens competes with a crowded field. The full comparison lives in §8 of this guide and in the series' master evaluation-tooling guide, [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) (which covers DeepEval, LangSmith, MLflow, RAGAS, Arize Phoenix, W&B, G-Eval — **TruLens is deliberately not covered there; this guide is the dedicated TruLens deep-dive that fills that gap**). Quick orientation:

| Tool | Type | One-line positioning |
|------|------|----------------------|
| **TruLens** | OSS eval + tracing + dashboard | The RAG triad, feedback functions, local Streamlit dashboard |
| **LangSmith** | Commercial (SaaS) | Integrated observability + eval for the LangChain ecosystem |
| **Langfuse** | OSS + cloud | Tracing/LLM-ops platform with eval on top |
| **DeepEval** | OSS library | Pytest-native LLM evaluation with 14+ built-in metrics |
| **RAGAS** | OSS library | RAG-specific reference metrics (faithfulness, context precision…) |
| **Arize Phoenix** | OSS + cloud | Open-source tracing + eval platform from Arize AI |

### 1.6 The TruLens Niche

- **The RAG triad as a first-class concept** — TruLens invented/popularised the triad framing (context relevance → groundedness → answer relevance) and makes it a two-minute setup.
- **Local and free** — a full eval + trace + dashboard stack runs entirely on your laptop or inside a bank's air-gapped environment; no SaaS account required (only the judge-model calls, which can be served by on-prem models too).
- **Inline/streaming evaluation** — feedback functions can run as the app executes, which makes continuous in-production evaluation natural (see [ai_agent_drift_guide.md](../ai_agent_drift_guide.md) for production monitoring patterns).
- **OpenTelemetry-native tracing** — portable traces to any observability backend.

### 1.7 When to Use TruLens

| Use case | Why TruLens fits |
|----------|------------------|
| **RAG evaluation** (offline test sets, dev iteration) | RAG triad feedback functions are built in; scores attach to retrievable records |
| **LLM app observability** (understand what an app did per request) | Decorator/wrapper instrumentation, per-step spans, cost & latency |
| **CI eval gates** | Run offline eval scripts in CI, pull leaderboard scores, assert thresholds (§6.5) |
| **Vendor-neutral / on-prem evaluation** | MIT licence, SQLite/Postgres storage, no mandatory cloud |
| **Comparing app versions** (model, prompt, retriever changes) | Versioned app records + leaderboard comparison view |

**When it is a weaker fit:** deep agentic-trace debugging at scale, tight LangChain-only workflows where LangSmith's integration is more convenient, or a pytest-native unit-testing culture where DeepEval's runner fits better (§8).

---

## 2. The RAG Triad

### 2.1 The Core Concept

The **RAG triad** is TruLens' signature evaluation framework: three feedback functions that, taken together, check whether a RAG (retrieval-augmented generation) system produced a trustworthy answer. Per the official docs: *"The RAG triad is made up of 3 evaluations: context relevance, groundedness and answer relevance. Satisfactory evaluations on each provides us confidence that our LLM app is free from hallucination."*

| Evaluation | Question it answers | Failure mode it catches |
|------------|---------------------|-------------------------|
| **Context relevance** | *Did we retrieve the right context for the question?* | Bad retrieval — irrelevant/missing chunks fed to the LLM |
| **Groundedness** | *Is the answer supported by (grounded in) the retrieved context?* | Hallucination — the LLM asserts facts not in the context |
| **Answer relevance** | *Does the answer actually address the user's question?* | Off-topic or unhelpful answers — the app answered something else |

### 2.2 Why the Triad — RAG Failure Modes

A RAG pipeline is a chain: **retrieve → augment → generate**. Each link can fail, and the failure modes are *distinct* — one score cannot catch all of them:

1. **Bad retrieval (context relevance fails).** The retriever returns the wrong documents, stale product sheets, or misses the relevant chunk (chunking too coarse, embedding drift, hybrid-search misconfig). The LLM then answers from garbage — and sounds confident while doing it. *RAG systems often sound correct even when they aren't.*
2. **Hallucination (groundedness fails).** Retrieval was fine, but the LLM embellishes: it adds numbers, names, or product features that are not in the retrieved context. This is the highest-risk failure in banking — a fabricated limit, fee, or regulatory detail.
3. **Off-topic / unhelpful answer (answer relevance fails).** The LLM understood something, retrieved something, and generated something — but not what the user asked (e.g., answering the nearest FAQ instead of the specific product question, or refusing to answer a question the knowledge base actually covers).

The triad covers all three in a deliberately simple triangle: **context relevance** checks retrieval quality, **groundedness** checks generation faithfulness to context, **answer relevance** checks the final answer against the query. Satisfactory scores on all three give you confidence the app is "free from hallucination" in the RAG sense. (Note: the triad is a *necessary* quality bar, not a *sufficient* one — it says nothing about toxicity, bias, regulatory compliance, or factual correctness beyond the retrieved context; combine it with the metrics in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) and [implementing-responsible-ai.md](../implementing-responsible-ai.md).)

### 2.3 The Triad in Code (Sketch)

The three feedback functions, expressed with TruLens' `Feedback` class and `Select` selectors (0.x-era API shown; the 1.x API is equivalent in shape — see §3 and §6):

```python
from trulens_eval import Feedback, Select
from trulens_eval.feedback.provider.openai import OpenAI

provider = OpenAI(model_engine="gpt-4o")

# 1. Context relevance — is the retrieved context relevant to the question?
f_context_relevance = (
    Feedback(provider.qs_relevance_with_cot_reasons, name="Context Relevance")
    .on(Select.RecordCalls.retrieve.args.query)     # the question
    .on(Select.RecordCalls.retrieve.rets.collect()) # the retrieved chunks
)

# 2. Groundedness — is the answer supported by the context?
from trulens_eval.feedback import Groundedness
grounded = Groundedness(groundedness_provider=provider)
f_groundedness = (
    Feedback(grounded.groundedness_measure_with_cot_reasons, name="Groundedness")
    .on(Select.RecordCalls.retrieve.rets.collect()) # the context
    .on_output()                                    # the answer
)

# 3. Answer relevance — does the answer address the question?
f_answer_relevance = (
    Feedback(provider.relevance_with_cot_reasons, name="Answer Relevance")
    .on(Select.RecordCalls.retrieve.args.query)     # the question
    .on_output()                                    # the answer
)
```

The pattern to notice: each feedback function declares **what it needs from the record** (the query argument, the retriever's return values, the final output) via `Select`, and TruLens wires those values in automatically when it evaluates the recorded run. Selector targets differ by app structure — the ones above assume a retriever method named `retrieve` (typical for LlamaIndex/LangChain RAG apps; see §4.3 for how selectors work).

### 2.4 Using Triad Scores

- **Dev iteration:** run a golden question set, watch the three scores per question, fix whichever leg is weakest (retrieval vs generation vs prompt).
- **Regression gates:** assert on triad means in CI (§6.5).
- **Production monitoring:** score live traffic with the same feedback functions; a groundedness dip signals retrieval drift or a bad deployment (§9; [ai_agent_drift_guide.md](../ai_agent_drift_guide.md)).

---

## 3. Feedback Functions

### 3.1 The `Feedback` Class

Everything evaluable in TruLens is a **feedback function** — a callable that takes parts of a record (inputs, intermediate steps, outputs) and returns a **score** (typically 0–1) plus, optionally, a **reason** (chain-of-thought explanation). Feedback functions are constructed with the `Feedback` class:

```python
Feedback(implementation_fn, name="My Metric", aggregate=mean)
```

Key elements:

| Element | Meaning |
|---------|---------|
| `implementation_fn` | The metric logic — a provider method (LLM judge), a programmatic function, or a custom callable |
| `name` | Shown in the dashboard/leaderboard |
| `.on(selector)` | Binds an input to the function from the app record |
| `.on_output()` | Binds the app's final output as an argument |
| `aggregate` | How per-step/per-chunk scores are combined into one record score (mean, median, min, max, …) |

### 3.2 Programmatic vs Model-Based Feedback

TruLens supports **two families** of feedback:

1. **Programmatic (deterministic):** no LLM involved. Examples: exact/lexical matching, embedding-distance-based similarity (e.g., `EmbeddingSimilarity` comparing answer to a golden answer), `GroundTruthAgreement` (compare against labelled golden answers), regex/safety checks. Fast, cheap, reproducible — but limited to what can be computed mechanically.
2. **Model-based (LLM-as-a-judge):** a provider LLM scores the app's behaviour with a prompt-based rubric, often with chain-of-thought reasoning (`..._with_cot_reasons`) that explains *why* a score was given. Expressive, matches human judgement better, but costs tokens and has judge-model bias/variance (see §3.4 and [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §3.1 on LLM-as-judge calibration).

The RAG triad functions are typically LLM-as-judge; `GroundTruthAgreement` and embedding similarity are typical programmatic ones.

### 3.3 Built-in Feedback

The `trulens-feedback` package ships a set of built-ins; the most commonly used:

| Feedback | Kind | What it scores |
|----------|------|----------------|
| `groundedness_measure_with_cot_reasons` (via `Groundedness` class) | LLM judge | Whether the answer's statements are supported by the context (fact-check vs context) |
| `qs_relevance_with_cot_reasons` | LLM judge | **Question–statement relevance** — relevance of each retrieved chunk to the query (context relevance) |
| `relevance_with_cot_reasons` | LLM judge | **Answer relevance** — relevance of the answer to the question |
| `GroundTruthAgreement` | Programmatic | Agreement of the answer with golden ground-truth answers |
| `EmbeddingSimilarity` | Programmatic | Cosine/embedding similarity between answer and reference |

The provider methods are the classic entry points: `provider.relevance_with_cot_reasons`, `provider.qs_relevance_with_cot_reasons`, and the `Groundedness` wrapper for groundedness — these are the three pieces of the RAG triad (§2.3). Additional evals cover helpfulness/harmlessness/honesty, toxicity, and more (the docs group them under "Honest, Harmless and Helpful Evals" and "Metrics").

### 3.4 LLM-as-a-Judge and Providers

Model-based feedback runs through **providers** — adapters that wrap a judge model. Verified/known provider packages in the 1.x line:

| Provider | Package | Notes |
|----------|---------|-------|
| OpenAI | `trulens-provider-openai` | `OpenAI` (e.g., `gpt-4o`, `gpt-4`), `AzureOpenAI` |
| AWS Bedrock | `trulens-provider-bedrock` | `Bedrock` — important for banks on AWS landing zones |
| Local/other models | via community and framework providers | Judge on self-hosted models keeps data on-prem (relevant for banking data residency) |

Judge-model choice matters: the judge should be at least as capable as the app's generator, and judge bias (favouritism toward verbose or self-consistent answers) must be calibrated — see the judge-model selection guidance in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md). For a bank, using a **Bedrock or self-hosted judge** (rather than a public API) keeps evaluation data inside the controlled environment — a real advantage over cloud-only eval tools.

### 3.5 Custom Feedback

Writing a custom feedback function is just wrapping any callable in `Feedback` and binding its arguments with selectors:

```python
def my_rule(input_text: str, output_text: str) -> float:
    # e.g., flag answers mentioning a blacklisted term
    return 0.0 if "guaranteed" in output_text.lower() else 1.0

f_custom = (
    Feedback(my_rule, name="No-Guarantee Language")
    .on(Select.RecordCalls.main.args.query)  # or whatever input you need
    .on_output()
)
```

Custom functions can also be LLM-based: write a prompt that returns a score (and optionally a reason), expose it as a Python function, and wrap it. This is how banks encode domain policies (e.g., "answer must not mention specific client names", "must hedge disclaimers", "must cite the source document") as automated feedback.

### 3.6 Feedback Aggregation

A single record can produce **many** raw scores — e.g., groundedness is computed per statement in the answer, context relevance per retrieved chunk. `aggregate` combines them into the record-level score shown in the leaderboard:

```python
Feedback(..., name="Groundedness", aggregate=min)   # worst statement wins — strict
Feedback(..., name="Context Relevance", aggregate=mean)  # average over chunks
```

- `mean` — typical default; smooth, hides outliers.
- `min` — strict: one bad statement/chunk sinks the score. Often right for groundedness in banking (one fabricated statement is unacceptable).
- `median` / `max` — robust or lenient variants.

Choose deliberately: `min`-aggregated groundedness is a stricter hallucination gate than `mean`, and the choice should be documented as part of the eval specification.

### 3.7 Feedback Examples (Complete Sketches)

Groundedness feedback (0.x-style, as most widely documented; shape carries to 1.x):

```python
from trulens_eval import Feedback, Select
from trulens_eval.feedback import Groundedness
from trulens_eval.feedback.provider.openai import OpenAI

provider = OpenAI(model_engine="gpt-4o")
grounded = Groundedness(groundedness_provider=provider)

f_groundedness = (
    Feedback(grounded.groundedness_measure_with_cot_reasons, name="Groundedness")
    .on(Select.RecordCalls.retrieve.rets.collect())
    .on_output()
    .aggregate(min)   # strict: worst statement decides
)
```

Relevance feedback:

```python
f_answer_relevance = (
    Feedback(provider.relevance_with_cot_reasons, name="Answer Relevance")
    .on(Select.RecordCalls.retrieve.args.query)
    .on_output()
)
```

---

## 4. Instrumentation and Tracking

### 4.1 The `Tru` / `TruSession` Class and the Database

The central object manages the **evaluation store** (where records, feedback scores, and app metadata live) and the **dashboard**:

- **0.x (`trulens_eval`):** `from trulens_eval import Tru` → `tru = Tru()`.
- **1.x (current):** `from trulens.core import TruSession` → `tru = TruSession()`. The 1.0 rewrite reorganised the library into `trulens-core`, `trulens-feedback`, `trulens-dashboard` plus optional provider/instrument packages, and renamed the main class — most pre-2024 tutorials show the old `Tru` API, so expect to translate them.

**Storage:**

| Backend | When to use | Notes |
|---------|-------------|-------|
| **SQLite** (default) | Local dev, single-user evaluation | Zero-config; database file under `~/.trulens/` (e.g., `trulens.db`) — easy to version or ship with a repo |
| **Postgres** | Shared team dashboards, CI, light production | Multi-user access to the same evaluation data; connection string configured on `TruSession` |
| Snowflake / other connectors (1.x) | Enterprise environments | The Snowflake connection fits the post-acquisition direction and existing bank data estates |

For a bank, the **SQLite → Postgres progression** is the natural one: SQLite for the analyst's laptop, Postgres for the shared model-validation workspace where risk and validation teams can query the same eval records.

### 4.2 App Tracking — Instrumentation

TruLens tracks an app in one of two ways:

1. **Framework wrappers** (0.x and 1.x): wrap a LangChain chain or LlamaIndex index in `TruChain` / `TruLlama` and every call inside it is recorded automatically (see §6.3).
2. **Decorator instrumentation** (the 1.x flagship pattern): annotate any function with the `@instrument` decorator and TruLens captures its calls — LLM generations, retrievals, tool invocations, and any nested instrumented function — as spans:

```python
from trulens.instrument import instrument

@instrument
def rag_answer(query: str) -> str:
    chunks = retriever.retrieve(query)   # captured
    prompt = build_prompt(query, chunks) # captured
    return llm.generate(prompt)          # captured
```

The decorator approach is what makes TruLens suitable for **custom and agentic apps** that don't use LangChain/LlamaIndex — "instrument any app with a decorator" is the current tagline. (There is no `@tru.app`-style magic wrapper in the standard API; the canonical mechanisms are the decorator and the framework wrappers.)

### 4.3 `Select` and `Select.RecordCalls`

**`Select`** is TruLens' selector DSL for pointing feedback functions at specific parts of an app's record. The most important entry point is **`Select.RecordCalls`** — "select into the *calls* made by the app":

```python
Select.RecordCalls.retrieve.args.query     # the `query` argument of the `retrieve` method
Select.RecordCalls.retrieve.rets.collect() # the return values of `retrieve` (flattened to a list)
Select.RecordCalls.llm.rets                 # the return of the `llm` call
```

- `args.<name>` — an argument of an instrumented call.
- `rets` — the return value(s) of a call.
- `.collect()` — gathers a collection of values (e.g., all retrieved chunks) into a single list argument for the feedback function.
- `.on_output()` — a shorthand binding the app's final output.

Selectors are **validated at runtime**: if a selector doesn't exist in the recorded app (e.g., you renamed `retrieve`), TruLens raises a clear error (the well-known *"Some selectors do not exist in the app or record"*), which forces eval definitions to stay in sync with app code.

### 4.4 Records, Spans, and Traces

- **Record** — one full run of the app (one user query end-to-end), with all inputs, outputs, intermediate values, and metadata.
- **Span** — a single instrumented operation within a record: one LLM call, one retrieval, one tool invocation, one nested function.
- **Trace** — the tree of spans for a record (which call led to which).

Since the 1.x line, instrumentation is **built on OpenTelemetry (OTEL)**: every function call, LLM generation, retrieval, and tool invocation is captured as a structured OTEL span, with **latency, inputs, outputs, tokens, and cost recorded per step**. This is a significant architectural choice:

- Traces are **portable** — export to Jaeger, Grafana Tempo, Datadog, or any OTLP-compatible backend; a "bad answer has a traceable cause rather than a vibe".
- The same spans feed the TruLens dashboard *and* the enterprise observability stack — no double instrumentation.
- For a bank, OTEL export means LLM evaluation traces can land in the same monitoring platform (e.g., Datadog/Splunk) the bank already runs, satisfying auditability expectations.

### 4.5 Streaming and Async

- **Streaming:** feedback functions can run **inline with app execution** (while the app streams), which is what makes continuous/in-production evaluation natural in TruLens compared with batch-only evaluators. In the dashboard, streaming records appear as they complete.
- **Async:** async apps are supported — `ainvoke`/`astream` on LangChain chains, async LlamaIndex queries, and async agent loops are traced and evaluated (community reports confirm async streaming works, with selector targets adjusted accordingly). When in doubt, verify async behaviour against the current docs for your exact app type — it has been a common support topic.

### 4.6 Instrumentation Example — Tracking a RAG App

Full skeleton (1.x-style):

```python
from trulens.core import TruSession
from trulens.instrument import instrument

tru = TruSession()  # SQLite store by default

@instrument
def rag(query: str) -> str:
    chunks = retriever.retrieve(query)
    context = "\n\n".join(c.text for c in chunks)
    prompt = f"Answer using ONLY the context.\n\nContext:\n{context}\n\nQuestion: {query}"
    return llm.generate(prompt)

tru.add_app(rag, app_name="BankingRAG", app_version="v1.0")

# ... add feedback functions (triad, §2.3/§3.7) ...

answer = rag("What is the confirmation period for a standby LC?")
tru.start_evaluator()   # async evaluator processes queued records
# scores land in the store; inspect via tru.get_leaderboard() or the dashboard
```

---

## 5. The Dashboard and Evaluation

### 5.1 The TruLens Dashboard — Streamlit-Based

The dashboard is a **Streamlit** web app that ships with TruLens (`trulens-dashboard` package). No separate frontend stack to run; it reads the evaluation store and renders:

- App **leaderboard** (scores per app/version)
- **Record inspection** (per-run traces with feedback scores)
- Feedback score **time series**
- **Cost and latency** per step
- (1.1+) a **comparison view** across app versions, multi-app support, metadata editing, and **virtual apps** (import evaluation data from a non-TruLens source, e.g., a benchmark run elsewhere).

### 5.2 The Leaderboard

The main page lists **apps and versions** with their evaluation scores (mean feedback scores), plus cost and latency. Workflow: give each iteration its own `app_version` (e.g., `v1.0` → `v1.1` after a reranker change), run the same golden set, and the leaderboard shows which version "earned" deployment — scores, latency, and cost side by side ("the cheapest version is often not the worst one").

### 5.3 Record Inspection

Clicking into a record shows the full trace: the query, retrieved chunks, prompt, final answer, and each feedback function's **score with its chain-of-thought reason** (e.g., groundedness's per-statement verdicts with explanations). This is the debugging superpower: a low groundedness score links directly to *which sentence was unsupported* and *what context was (or wasn't) available* — then you fix the retriever or the prompt, not the number.

### 5.4 Eval Results and Time Series

Feedback scores are stored per record with timestamps, so the dashboard plots **score evolution over time** — the core primitive for production monitoring and drift detection (see [ai_agent_drift_guide.md](../ai_agent_drift_guide.md)). Filter by app version, date range, or metadata (e.g., product area) to see whether a model update or corpus change degraded groundedness before users complain.

### 5.5 Cost and Latency

Every span records **latency, tokens, and cost** (when the provider exposes usage). The dashboard surfaces per-step cost — which is how teams find "where you can cut cost without losing quality": e.g., a step with high latency and low marginal score is a candidate for a cheaper model or fewer retrieved chunks.

### 5.6 Launching the Dashboard

```bash
trulens dashboard            # CLI (1.x)
```

or programmatically:

```python
tru.run_dashboard()          # serve the dashboard (Streamlit app on localhost)
tru.get_leaderboard(app_ids=["BankingRAG_v1.0"])  # pull scores headlessly
tru.stop_dashboard(force=True)
```

For CI and scripting, prefer **headless access** (`get_leaderboard`, direct DB queries) rather than the UI; for humans, `trulens dashboard` and browse.

---

## 6. Usage and Quickstart

### 6.1 Installation

```bash
pip install trulens                          # core + feedback + dashboard
pip install trulens-provider-openai          # OpenAI/AzureOpenAI judge provider
pip install trulens-provider-bedrock         # AWS Bedrock judge provider
pip install trulens-instrument-langchain     # TruChain (LangChain apps)
pip install trulens-instrument-llamaindex    # TruLlama (LlamaIndex apps)
```

(Extras-style installs — e.g., `trulens[langchain]`-like bundles — appear in some guides; the explicit packages above match the 1.x split. The legacy `trulens_eval` package remained available as a backwards-compatibility shim after the 1.0 split.) Python 3.9+ typically; verify against current metadata.

### 6.2 Basic Usage Flow

The canonical loop:

1. **Build the app** (any Python: LangChain, LlamaIndex, custom) and instrument it (wrapper or decorator).
2. **Create `TruSession`** (SQLite default).
3. **Define feedback functions** — triad and any domain metrics (§3).
4. **Register app + feedback**, run the app over your dataset (records + scores accumulate).
5. **Inspect** — dashboard or `get_leaderboard`; iterate; bump `app_version`.
6. **Gate** — assert on thresholds in CI (§6.5).

### 6.3 Framework Integrations

| Integration | Wrapper/package | Status |
|-------------|-----------------|--------|
| **LangChain** | `TruChain` (`trulens-instrument-langchain` / `trulens.apps.langchain`) | Mature — the classic integration; wraps chains (incl. LCEL) and records every step |
| **LlamaIndex** | `TruLlama` (`trulens-instrument-llamaindex` / `trulens.apps.llamaindex`) | Mature — the canonical pairing for RAG-triad tutorials |
| **Haystack** | `TruHaystack` (following the TruLlama/TruChain pattern) | Newer — auto-instrumentation of Haystack v2 pipeline components (retrievers, generators, rankers) as OTEL spans was still landing as of 2025/26 (see truera/trulens issue #2448); verify current status before relying on it |
| **Custom apps** | `@instrument` decorator | Fully supported — the 1.x flagship path (§4.2) |

Both `TruChain` and `TruLlama` support streaming and async variants, and both make the recorded selectors (`.retrieve.args.query`, `.rets.collect()`, etc.) available for feedback binding.

### 6.4 RAG Quickstart — Build, Evaluate with the Triad

A complete minimal example (0.x-style API, the most documented; translate to 1.x per §4.1):

```python
# 1. Your RAG app (LangChain here)
from langchain.chains import RetrievalQA
chain = RetrievalQA.from_chain_type(llm=llm, retriever=vectorstore.as_retriever())

# 2. Recorder + feedback
from trulens_eval import Feedback, Tru, Select
from trulens_eval.feedback.provider.openai import OpenAI
from trulens_eval.feedback import Groundedness
from trulens_eval.app.langchain import TruChain

tru = Tru()
provider = OpenAI(model_engine="gpt-4o")
grounded = Groundedness(groundedness_provider=provider)

f_context_relevance = (Feedback(provider.qs_relevance_with_cot_reasons, name="Context Relevance")
    .on(Select.RecordCalls.retrieve.args.query)
    .on(Select.RecordCalls.retrieve.rets.collect()))
f_groundedness = (Feedback(grounded.groundedness_measure_with_cot_reasons, name="Groundedness")
    .on(Select.RecordCalls.retrieve.rets.collect())
    .on_output().aggregate(min))
f_answer_relevance = (Feedback(provider.relevance_with_cot_reasons, name="Answer Relevance")
    .on(Select.RecordCalls.retrieve.args.query)
    .on_output())

truchain = TruChain(chain, app_id="BankingRAG_v1.0",
                    feedbacks=[f_context_relevance, f_groundedness, f_answer_relevance])

# 3. Run the golden set
with truchain as recording:
    for q in golden_questions:
        chain.invoke({"query": q})

# 4. Inspect
tru.run_dashboard()
```

Run this against a **golden question set** (30–100 representative queries with expected behaviours) and you have: per-question triad scores, retrievable traces for every low score, and a versioned leaderboard.

### 6.5 CI/CD — Eval Gates

TruLens fits CI as an **offline eval script + threshold assertion**:

```python
# eval_gate.py — run in CI after retriever/model changes
tru = Tru()
# ... record runs over the golden set ...
lb = tru.get_leaderboard(app_ids=["BankingRAG_v1.1"])
mean_grounded = lb[lb["record_id"] != "ALL" ... ]  # or query the store directly
assert mean_groundedness >= 0.80, f"groundedness {mean_groundedness} below gate"
assert mean_context_relevance >= 0.75
assert mean_answer_relevance >= 0.80
```

- Wire it as a GitHub Actions / GitLab CI job: on PRs touching prompts, retrievers, or the model, run the gate; failure blocks merge.
- **Thresholds must be calibrated** from a baseline run (pick gates from the current best version's scores minus a margin), not guessed.
- Keep the **golden set versioned** alongside the code; eval-set drift quietly invalidates gates.
- For the full CI/CD pattern catalogue (DeepEval's pytest runner vs LangSmith's API vs TruLens' script approach), see [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §15 — the honest take from that guide: *"RAGAS for metrics, DeepEval for CI ergonomics, LangSmith/TruLens for tracing"*; TruLens' CI role is the traceable gate, not the prettiest test runner.

### 6.6 Production — Ongoing Evaluation

In production, TruLens' inline/streaming feedback lets you **score live traffic continuously** (sampled, for cost control): the same triad functions run on real user queries, and the dashboard's time series becomes the drift/monitoring surface. Combine with:

- **Sampling** — judge every Nth request to bound judge-model cost.
- **Alerting on score dips** — e.g., groundedness dropping below threshold over a window signals retrieval drift, corpus staleness, or a bad model rollout.
- **Human review of low-scoring records** — export them for the validation team.

Production monitoring and drift concepts are covered in depth in [ai_agent_drift_guide.md](../ai_agent_drift_guide.md); the eval-vs-validation framing for what these scores *mean* in a regulated setting is in [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md).

---

## 7. Ecosystem and Community

### 7.1 Integrations

- **Frameworks:** LangChain (`TruChain`), LlamaIndex (`TruLlama`), Haystack (`TruHaystack`, newer), plus any custom Python via `@instrument`.
- **Judge providers:** OpenAI/AzureOpenAI (`trulens-provider-openai`), AWS Bedrock (`trulens-provider-bedrock`), and local-model options for on-prem judging.
- **Observability:** OpenTelemetry-native spans exportable to Jaeger, Grafana Tempo, Datadog, and any OTLP endpoint — plus the TruLens dashboard itself, and community/adopter builds such as Crest Data's TruLens app on the Datadog Marketplace (see `ADOPTERS.md` in the repo).
- **Storage:** SQLite, Postgres, Snowflake connector (1.x).

### 7.2 Community

- **GitHub:** `truera/trulens` — MIT-licensed; roughly **3.1k+ stars** as of mid-2026 (community trackers; verify live on the repo page). Active issue tracker and a maintained `ADOPTERS.md`.
- **Adopters/community:** active cookbook of examples (LangChain, LlamaIndex, agents, Snowflake tools), integrations with Streamlit-in-Snowflake discussions, and a steady stream of third-party tutorials — though the community is **smaller** than LangChain/LangSmith's and much of the third-party content still shows the pre-1.0 `trulens_eval` API (an honest ecosystem caveat, §8.3).

### 7.3 Docs

- **Official site:** [trulens.org](https://www.trulens.org) — getting-started, core concepts (RAG triad, metrics, evals), cookbook, and API reference, including a section on Snowflake's commitment to the OSS project.
- **Repo docs:** `docs/` in `truera/trulens` with the source of the guides.

### 7.4 License

**MIT** — permissive, commercial-friendly, no copyleft. Verified from the repository LICENSE. This is a clean licensing position for enterprise adoption (see also [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §13.4 for the cost-model table across tools).

### 7.5 Development Status

- **Versions:** the 1.x line is current — 1.0 (the big 2024 rewrite: package split, `TruSession`, decorator instrumentation), 1.1 (Oct 2024: dashboard comparison view, multi-app support, metadata editing, virtual apps), and subsequent 1.x releases through 2026 (PyPI shows ongoing updates; verify the exact latest version on [PyPI](https://pypi.org/project/trulens/) before citing).
- **Direction:** OpenTelemetry-native tracing, decorator-first instrumentation, agent evaluation (tool-call tracing), and Snowflake-backed stewardship are the defining features of the current era.
- **Caveat:** the 0.x → 1.x API break means older tutorials/code need translation; the project maintained `trulens_eval` as a compatibility shim.

---

## 8. Comparison with Alternatives

### 8.1 Comparison Table

TruLens vs the main alternatives. This table complements (and does not duplicate) the master matrix in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §14, which also covers MLflow, W&B, Galileo, Vertex/Azure eval, and Continuous Eval.

| Tool | Type | RAG triad | LLM-judges | Tracing | Dashboard | Open-source | Best for |
|------|------|-----------|------------|---------|-----------|-------------|----------|
| **TruLens** | Eval + tracing + dashboard library | ✅ First-class (context relevance, groundedness, answer relevance) | ✅ Providers (OpenAI, Bedrock, local) | ✅ OTEL-native, decorator + wrappers | ✅ Local Streamlit, free | ✅ MIT | Open-source RAG evals with debuggable traces, on-prem, CI gates |
| **LangSmith** | Commercial observability + eval platform | ⚠️ LLM-as-judge based, not triad-native | ✅ | ✅ Deep LangChain integration | ✅ Hosted | ❌ | LangChain apps wanting integrated tracing, datasets, online feedback |
| **Langfuse** | LLM-ops / tracing platform | ⚠️ Custom eval templates | ✅ | ✅ Core strength (spans, sessions) | ✅ Hosted or self-host | ✅ MIT | Production tracing + prompt management with eval on top |
| **DeepEval** | Evaluation library (pytest-native) | ⚠️ Metrics exist (contextual relevancy, faithfulness, answer relevancy) but no triad framing | ✅ Any model, G-Eval | ❌ (no tracing) | ⚠️ Confident AI cloud only | ✅ Apache-2.0 | CI unit-testing of LLM outputs (`assert_test`), metric breadth |
| **RAGAS** | RAG metrics library | ⚠️ Equivalent metrics (faithfulness, context precision/recall, answer relevancy) | ✅ Internal judge prompts | ❌ | ❌ | ✅ Apache-2.0 | Reference/benchmark RAG metrics on any pipeline (metric code, not a platform) |
| **Arize Phoenix** | OSS tracing + eval platform | ⚠️ Custom evals | ✅ | ✅ OpenInference spans | ✅ Local | ✅ Apache-2.0 | Open-source production tracing with eval, Arize cloud upgrade path |

Key differentiators in one line: **TruLens** = triad feedback functions + traces + local dashboard, all free; **LangSmith** = deepest LangChain integration but closed/SaaS; **Langfuse** = the tracing/ops platform with eval bolted on; **DeepEval** = the CI/pytest experience; **RAGAS** = the reference metric definitions. They are **complementary** — as [rag_frameworks_comparison_guide.md](rag_frameworks_comparison_guide.md) §11 notes, a common stack is "RAGAS for metrics, DeepEval for CI ergonomics, LangSmith/TruLens for tracing" — but if you want one free tool that does triad evals *and* tracing *and* a dashboard, TruLens is the only entry in that row.

### 8.2 TruLens Strengths

- **Genuinely open and free** — MIT licence, local SQLite/Postgres, local dashboard; no mandatory SaaS (unlike LangSmith; Langfuse is OSS but its core value is the hosted platform).
- **The RAG triad, done well** — the concept is first-class: three feedback functions, two-minute setup, per-statement/per-chunk scoring with chain-of-thought reasons.
- **Trace-linked evaluation** — every score is attached to an inspectable record; low scores are debugged, not just reported.
- **Local / on-prem friendly** — runs fully inside a bank's environment (including with Bedrock or self-hosted judges), which suits data-residency and air-gapped requirements.
- **OpenTelemetry-native (1.x)** — traces export to any OTLP backend; no lock-in, fits enterprise observability estates.
- **Version comparison** — leaderboard + comparison view make model/prompt/retriever experiments measurable ("ship the version that earns it").
- **Streaming/inline evaluation** — natural fit for continuous production scoring.

### 8.3 TruLens Weaknesses (Honest)

- **Smaller ecosystem and community** than LangSmith/LangChain-land: ~3k GitHub stars vs LangSmith's much larger adoption; fewer third-party integrations, cookbooks, and hire-able specialists.
- **0.x → 1.x API break** — a large share of online tutorials/code shows the old `trulens_eval` API; translating is friction, and docs churn has been a real pain point (the project does maintain a compatibility shim).
- **Scale limits** — the default SQLite store is single-user; Postgres helps, but TruLens is not built as a high-throughput distributed tracing backend — at large production scale, teams typically export OTEL spans to a dedicated backend.
- **Not a test runner** — no pytest-native assertion UX like DeepEval; CI gates are hand-rolled scripts (fine, but you build the harness).
- **Agent evaluation is younger** — agent tracing (multi-step tool calls, loops) works via instrumentation, but the agent-eval story (e.g., trajectory-level metrics) is less mature than the RAG story; see [autonomous_agents_guide.md](../autonomous_agents_guide.md) §5 for agent-eval guidance.
- **Judge-model cost and bias** — LLM-as-judge feedback costs tokens per run and inherits judge bias; needs sampling and calibration (true of every judge-based tool).

### 8.4 When to Pick What

| Situation | Recommendation |
|-----------|----------------|
| Open-source RAG evaluation with debuggable traces, local dashboard, CI gates | **TruLens** |
| LangChain-heavy team wanting zero-friction integrated tracing + datasets + online feedback, budget available | **LangSmith** |
| Production LLM-ops/tracing platform, prompt management, team workflows (self-hosted or cloud) | **Langfuse** |
| Pytest-native CI evaluation culture, broad metric library, minimal ceremony | **DeepEval** |
| Reference RAG metrics on an existing pipeline, benchmark reporting | **RAGAS** |
| OSS production tracing platform with eval, or an Arize cloud migration path | **Arize Phoenix** |

Selection detail and cost models for all of these live in [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) §14/§18 and [rag_frameworks_comparison_guide.md](rag_frameworks_comparison_guide.md) §11.

---

## 9. Worked Example — Evaluating a Banking RAG Assistant

### 9.1 Scenario

Crédit Agricole CIB's trade-finance desk wants a **product-support RAG assistant**: front-office users ask questions about trade-finance products (standby letters of credit, documentary credits, guarantees, FX forwards, commodity finance) and the assistant answers from the **product documentation corpus** (term sheets, operational handbooks, regulatory summaries). The requirements are strict: answers must be **accurate** (no invented fees/limits), **grounded** (traceable to a source document), and **compliant** (no advice-like language, no fabrications).

This is a textbook case for the **RAG triad + TruLens**: the team needs to prove retrieval quality, generation faithfulness, and answer relevance — and needs per-answer evidence for the model-validation file (see [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) on evaluation as *validation evidence*, and [implementing-responsible-ai.md](../implementing-responsible-ai.md) for the governance overlay).

### 9.2 Setup

```python
# eval_setup.py
from trulens_eval import Feedback, Select, Tru
from trulens_eval.feedback import Groundedness
from trulens_eval.feedback.provider.openai import OpenAI
from trulens_eval.app.langchain import TruChain

tru = Tru()                      # SQLite store, versioned with the repo
provider = OpenAI(model_engine="gpt-4o")   # or Bedrock provider for on-prem judging
grounded = Groundedness(groundedness_provider=provider)

f_context_relevance = (Feedback(provider.qs_relevance_with_cot_reasons, name="Context Relevance")
    .on(Select.RecordCalls.retrieve.args.query)
    .on(Select.RecordCalls.retrieve.rets.collect())
    .aggregate(mean))
f_groundedness = (Feedback(grounded.groundedness_measure_with_cot_reasons, name="Groundedness")
    .on(Select.RecordCalls.retrieve.rets.collect())
    .on_output()
    .aggregate(min))                       # strict: one unsupported sentence fails
f_answer_relevance = (Feedback(provider.relevance_with_cot_reasons, name="Answer Relevance")
    .on(Select.RecordCalls.retrieve.args.query)
    .on_output())

truchain = TruChain(rag_chain, app_id="TF_Support_RAG_v1.0",
                    feedbacks=[f_context_relevance, f_groundedness, f_answer_relevance])
```

**Golden set:** 40 questions across product families, including edge cases the business cares about (e.g., "What is the confirmation period for a standby LC?", "Are guarantee fees capped?", "What documents are needed for a documentary credit amendment?"). Golden answers are *not* required for the triad (it is reference-free), but a small labelled subset enables `GroundTruthAgreement` spot-checks.

### 9.3 Evaluation Run

```python
with truchain as recording:
    for q in golden_set:
        rag_chain.invoke({"query": q})

tru.run_dashboard()
```

Expected output shape (illustrative v1.0 results):

| Version | Context Relevance (mean) | Groundedness (min) | Answer Relevance (mean) | Avg latency | Avg cost/query |
|---------|--------------------------|--------------------|--------------------------|-------------|----------------|
| v1.0 (baseline: naive vector search) | 0.71 | **0.58** | 0.74 | 3.1 s | $0.021 |
| v1.1 (hybrid search + reranker) | 0.88 | **0.86** | 0.87 | 3.4 s | $0.023 |
| v1.2 (v1.1 + stricter prompt) | 0.88 | 0.87 | 0.91 | 3.4 s | $0.023 |

### 9.4 Analysis — Fixing Low Groundedness

The v1.0 run shows the classic failure signature: **groundedness (0.58) is the weakest leg**. Clicking the low-scoring records in the dashboard shows *why*:

- For "Are guarantee fees capped?", the retriever returned marketing material and a neighbouring product sheet, missing the actual fee schedule page → the answer asserted a made-up cap (groundedness per-statement verdict: "unsupported by context").
- Root cause: pure cosine similarity over coarse chunks loses exact-figure documents; chunking splits tables mid-row.

**Fix applied (v1.1):** hybrid retrieval (BM25 + embeddings) with a reranker, table-aware chunking, and a `top_k` of 5 with a relevance floor. Groundedness rises to 0.86. **v1.2** tightens the system prompt ("never state figures not present in the context") which lifts answer relevance to 0.91. Each version is a row in the leaderboard with its own traces — the whole improvement story is inspectable and auditable.

### 9.5 The Dashboard as Evidence

- **Leaderboard** shows the three versions with scores, latency, and cost — the material for the release decision.
- **Record view** provides per-answer evidence: retrieved chunks, prompt, answer, and the judge's chain-of-thought reasons. This is exactly the *traceable evaluation evidence* a model-validation review wants.
- **Time series** starts accumulating the moment the app serves traffic — the monitoring baseline for [ai_agent_drift_guide.md](../ai_agent_drift_guide.md) patterns (e.g., corpus refresh drops groundedness → alert before users notice).

### 9.6 Production — Eval Gates in CI

```python
# ci/eval_gate.py — runs on PRs touching prompt/retriever/model
MIN_GROUNDED, MIN_CTX, MIN_ANS = 0.80, 0.75, 0.80   # calibrated from v1.1 baseline
# ... record runs over the golden set (new app_version) ...
scores = get_scores_from_store("TF_Support_RAG_v1.2")
assert scores["Groundedness"] >= MIN_GROUNDED
assert scores["Context Relevance"] >= MIN_CTX
assert scores["Answer Relevance"] >= MIN_ANS
```

The gate is **versioned with the golden set** and runs in CI on every candidate change; a failed gate blocks merge and links to the offending records. In production, sample ~10% of live traffic through the same feedback functions; alert if the 7-day rolling groundedness falls below the gate.

### 9.7 Banking Context — Evaluation as Validation Evidence

For a bank, TruLens output is not just engineering telemetry — it is **validation evidence**:

- **Model validation (SR 11-7 / MAS FEAT accountability):** triad scores over a defined golden set, with per-record traces and judge reasons, document accuracy and reliability of the LLM app; the eval harness is the "independent testing" artifact.
- **Transparency:** groundedness + context relevance demonstrate that answers trace to source documents — the machine-readable version of "we can show where this answer came from".
- **Monitoring:** production score time series satisfy the ongoing-monitoring expectation; a groundedness dip triggers investigation and re-validation, not just a ticket.
- **Limits to state honestly:** the triad measures *grounding in the retrieved context*, not *regulatory correctness* — a document can be retrieved faithfully and still be wrong or outdated. Pair TruLens with content-level review, human-in-the-loop sign-off on critical answers, and the broader governance in [implementing-responsible-ai.md](../implementing-responsible-ai.md). The eval-vs-validation distinction — what a score means vs what a sign-off requires — is the subject of [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md).

---

## 10. Summary — TruLens in One Page

**Core:** TruLens is the open-source (MIT) LLM evaluation and observability library — **RAG triad feedback functions + automatic instrumentation/tracing + a free local Streamlit dashboard** — created by TruEra (now stewarded under Snowflake's oversight since the 2025 acquisition of TruEra's platform).

**The RAG triad** (context relevance → groundedness → answer relevance) is its signature: three feedback functions that catch bad retrieval, hallucination, and off-topic answers — the three ways RAG systems fail — and give confidence the app is "free from hallucination."

**Feedback functions** score every step: programmatic (ground truth agreement, embedding similarity) and LLM-as-a-judge (provider-based, with chain-of-thought reasons), wired to records via `Select`/`Select.RecordCalls`, aggregated per record (mean/min/median), and easily extended with custom domain rules.

**Instrumentation** (decorator or `TruChain`/`TruLlama` wrappers) captures every LLM call, retrieval, and tool invocation as OpenTelemetry spans with latency, tokens, and cost — stored in SQLite (default) or Postgres, inspectable in the dashboard, exportable to any OTLP backend.

**Usage:** `pip install trulens` → build/instrument the app → define triad feedback → run over a golden set → `trulens dashboard` (or `tru.run_dashboard()`) → iterate with `app_version` → gate CI on thresholds → sample live traffic for production monitoring.

**Positioning:** the open-source, local-first option in an eval field otherwise dominated by SaaS (LangSmith), tracing platforms (Langfuse), test runners (DeepEval), and metric libraries (RAGAS) — the natural pick when you need triad evals *and* traces *and* a dashboard without a cloud account, and a strong fit for regulated, on-prem environments (banks).

**Final word:** TruLens' enduring contribution is the *RAG triad* and the discipline of attaching every score to a traceable record. For any RAG system that must be provably grounded — and for banks that must evidence it — TruLens is the lowest-friction way to make evaluation inspectable, versioned, and continuous.

---

## 11. Glossary

| Term | Definition |
|------|------------|
| **TruLens** | Open-source (MIT) library for LLM evaluation and observability: feedback functions (evals), instrumentation (tracing), and a local Streamlit dashboard. Repo: `truera/trulens`; docs: trulens.org. |
| **TruEra** | The AI observability company (founded 2021) that created TruLens. Snowflake acquired TruEra's AI Observability Platform in 2025; TruLens remains open source, now overseen/supported by Snowflake. |
| **RAG triad** | TruLens' three core RAG evaluations: context relevance, groundedness, answer relevance. Satisfactory scores on all three give confidence the app is free from hallucination. |
| **Context relevance** | Feedback eval: is each retrieved chunk relevant to the user's question? Catches bad retrieval. |
| **Groundedness** | Feedback eval: is every statement in the answer supported by the retrieved context? Catches hallucination; often aggregated with `min` for strictness. |
| **Answer relevance** | Feedback eval: does the final answer address the user's question? Catches off-topic/unhelpful answers. |
| **Feedback function** | A metric wrapped by the `Feedback` class that scores a part of an app record (input, intermediate step, or output) with an optional chain-of-thought reason. |
| **LLM-as-a-judge** | Using an LLM (the judge) to score another LLM's output via a prompt rubric; the basis of model-based feedback functions. |
| **Provider** | Adapter wrapping a judge model (OpenAI, AzureOpenAI, Bedrock, local) that supplies model-based feedback methods. |
| **Tru** | The main class in TruLens 0.x (`trulens_eval.Tru`) managing the evaluation store and dashboard. |
| **TruSession** | The main class in TruLens 1.x (`trulens.core.TruSession`) — the 1.0-rewrite successor to `Tru`. |
| **Select** | TruLens' selector DSL binding feedback function arguments to parts of an app's record (e.g., `Select.RecordCalls.retrieve.args.query`). |
| **Select.RecordCalls** | The selector namespace for "the calls made by the app" — used to reach method arguments (`args`), return values (`rets`), and collections (`.collect()`) of instrumented calls. |
| **Record** | One full run of an app: inputs, outputs, intermediate values, spans, and feedback scores. |
| **Span** | A single instrumented operation in a trace (one LLM call, retrieval, or tool invocation); OpenTelemetry-native in 1.x. |
| **Trace** | The tree of spans produced by one record — the end-to-end story of a request. |
| **Instrumentation** | Automatic capture of app calls — via the `@instrument` decorator or framework wrappers (`TruChain`, `TruLlama`) — to build records/traces. |
| **Dashboard** | The Streamlit-based UI for browsing the leaderboard, records, scores, and cost/latency; launched via `trulens dashboard` or `tru.run_dashboard()`. |
| **Streamlit** | The Python web-app framework the TruLens dashboard is built on. |
| **Leaderboard** | The dashboard's app/version comparison view: mean feedback scores, latency, and cost per version. |
| **TruLlama** | The LlamaIndex integration wrapper: wrap a LlamaIndex app for automatic tracing and evaluation. |
| **TruChain** | The LangChain integration wrapper: wrap a LangChain chain for automatic tracing and evaluation. |
| **LangChain** | A popular LLM application framework; integrated with TruLens via `TruChain`. |
| **LlamaIndex** | A data/RAG-focused LLM framework; integrated with TruLens via `TruLlama`. |
| **Haystack** | The deepset LLM pipeline framework; newer TruLens integration (`TruHaystack`) following the TruLlama/TruChain pattern. |
| **SQLite** | TruLens' default zero-config evaluation store (local database file). |
| **Postgres** | Optional shared evaluation store for team dashboards, CI, and light production use. |
| **Eval gate** | A CI/CD check that fails the build when evaluation scores fall below thresholds — TruLens' pattern is an offline eval script + threshold assertion. |
| **CI/CD** | Continuous integration/deployment — where eval gates run on every candidate change. |
| **Threshold** | The minimum score a feedback metric must reach (calibrated from a baseline) for the gate to pass. |
| **LangSmith** | LangChain's commercial observability + evaluation platform (closed-source SaaS); TruLens' main integrated-eval competitor. |
| **Langfuse** | Open-source (MIT) LLM-ops/tracing platform with evaluation features; TruLens' main tracing-platform competitor. |
| **DeepEval** | Open-source (Apache-2.0) pytest-native LLM evaluation library with 14+ built-in metrics. |
| **RAGAS** | Open-source (Apache-2.0) RAG metrics library (faithfulness, context precision/recall, answer relevancy). |
| **Arize** | AI observability company; maker of Phoenix, the open-source tracing + eval platform (Apache-2.0). |
| **Hallucination** | The LLM asserting facts not present in its context; the failure mode groundedness catches. |
| **Retrieval** | The RAG step that fetches relevant chunks from a knowledge base; the failure mode context relevance catches. |
| **Grounding** | The property of an answer being supported by (traceable to) its retrieved context; measured by groundedness feedback. |
| **OpenTelemetry (OTEL)** | The open observability standard TruLens 1.x instrumentation is built on; traces export to any OTLP-compatible backend. |
| **Virtual app** | A dashboard feature (1.1+) to import evaluation data from non-TruLens sources into the leaderboard. |

---

*End of guide — cross-references: [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) · [Evaluation vs Validation](../llm_evaluation_vs_validation_guide.md) · [AI Agent Drift](../ai_agent_drift_guide.md) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [RAG Optimization](rag_optimization_techniques_guide.md) · [Autonomous Agents](../autonomous_agents_guide.md) · [Responsible AI](../implementing-responsible-ai.md)*


