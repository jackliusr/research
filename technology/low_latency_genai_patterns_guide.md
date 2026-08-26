# Low-Latency GenAI Application Patterns — The GenAI Latency Discipline, Deep-Dive

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — AI/LLM engineering series; the **dedicated deep-dive on low-latency generative-AI application patterns**: the GenAI latency discipline — latency anatomy (TTFT/TPOT/end-to-end/perceived), the inference levers (KV cache, speculative decoding, continuous batching, quantization, parallelism), the serving frameworks (vLLM, TGI, TensorRT-LLM, SGLang), the application patterns (token streaming, prompt/prefix caching, semantic caching), RAG latency, concurrency, the banking context, and a worked low-latency copilot
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** NVIDIA NIM LLM benchmarking metrics documentation (TTFT/TPOT/ITL/e2e definitions); ClickHouse engineering — "LLM inference latency: TTFT, tokens per second, and what to measure" (Jul 2026); arXiv primary papers — PagedAttention/vLLM (2309.06180, SOSP 2023), speculative decoding (2211.17192, ICML 2023), Medusa (2401.10774), SGLang/RadixAttention (2312.07104), HNSW (1603.09320), GPTQ (2210.17323, ICLR 2023), AWQ (2306.00978, MLSys 2024); Hugging Face Text Generation Inference documentation; NVIDIA TensorRT-LLM repository; OpenTelemetry GenAI semantic conventions; press coverage of bank GenAI rollouts (CNBC, JPMorganChase.com)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Overview — The Latency Discipline](#1-overview--the-latency-discipline)
   - 1.1 What GenAI Latency Is
   - 1.2 Why GenAI Latency Is Different
   - 1.3 The Overview Table
2. [The Latency Anatomy — TTFT, TPOT, End-to-End, Perceived](#2-the-latency-anatomy--ttft-tpot-end-to-end-perceived)
   - 2.1 Time-to-First-Token (TTFT)
   - 2.2 Time-Per-Output-Token (TPOT)
   - 2.3 End-to-End and Perceived Latency
   - 2.4 The Anatomy Table
   - 2.5 A Sample Latency Timeline
   - 2.6 The Observability Contract
3. [The Inference Levers](#3-the-inference-levers)
   - 3.1 The KV Cache
   - 3.2 Speculative Decoding — Draft Models and Medusa
   - 3.3 Continuous Batching — Orca and vLLM's PagedAttention
   - 3.4 Quantization — INT8/INT4, GPTQ and AWQ
   - 3.5 Tensor and Pipeline Parallelism
   - 3.6 The Levers Table
   - 3.7 Choosing the Lever — Symptom to Lever Map
4. [The Serving Frameworks](#4-the-serving-frameworks)
   - 4.1 vLLM
   - 4.2 Hugging Face Text Generation Inference (TGI)
   - 4.3 NVIDIA TensorRT-LLM
   - 4.4 SGLang
   - 4.5 The Framework Table
   - 4.6 The Selection Matrix
   - 4.7 The Hardware Backdrop — H100, B200, TPU
5. [The Application Patterns](#5-the-application-patterns)
   - 5.1 Token Streaming — SSE
   - 5.2 Prompt and Prefix Caching
   - 5.3 Semantic Caching
   - 5.4 The Patterns Table
   - 5.5 The Caching Hierarchy — Where the Patterns Sit
6. [The RAG Latency](#6-the-rag-latency)
   - 6.1 HNSW Vector Indexes
   - 6.2 Hybrid Search
   - 6.3 Reranking
   - 6.4 The RAG Table
   - 6.5 The RAG Latency Budget
7. [The Concurrency](#7-the-concurrency)
   - 7.1 Async
   - 7.2 Queueing
   - 7.3 The Concurrency Table
   - 7.4 Load Testing and Capacity Planning
   - 7.5 The Gateway's Role
8. [The Banking Context](#8-the-banking-context)
   - 8.1 Trading Assistants
   - 8.2 Client-Facing Copilots
   - 8.3 The Latency Budgets
   - 8.4 The Banking Table
   - 8.5 The Singapore Angle
9. [The Worked Example — A Low-Latency Cymbal Bank RAG Copilot](#9-the-worked-example--a-low-latency-cymbal-bank-rag-copilot)
   - 9.1 The Scenario
   - 9.2 The Latency-Budget Design
   - 9.3 The Levers Actually Applied
   - 9.4 The Lessons
   - 9.5 The Risk Register
   - 9.6 What If the Levers Stop Working?
10. [Summary — One Page](#10-summary--one-page)
11. [Glossary](#11-glossary)
12. [Claims Status, References and Disclaimer](#12-claims-status-references-and-disclaimer)

### How to Read This Guide

This is the **dedicated deep-dive on low-latency GenAI application patterns** — the latency discipline for generative-AI systems, from the silicon up to the user's first rendered token. It is the latency companion to the broader AI/LLM engineering cluster in this repository:

- **The latency sibling** — [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md) surveys latency-optimisation *techniques* (metrics, model-level and serving-level tricks, prompt engineering, caching). This guide is the *discipline*: the anatomy of where latency comes from, the levers that move it, the frameworks that operationalise them, and the application patterns that convert raw speed into perceived speed. Cross-reference heavily — where a technique is listed there, its latency *mechanics* are explained here.
- **The caching angle** — [ai_llm/agent_runtime_cache_design_guide.md](ai_llm/agent_runtime_cache_design_guide.md) is the repository's canonical caching design (L1–L5 hierarchy, cache keys, invalidation, what to cache and what never to cache, in banking context). §5 of this guide cross-references it for semantic/prefix caching depth.
- **The serving-framework angle** — [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) (just added) covers the model-hub layer and documents CSGHub's orchestration of vLLM, SGLang, TGI, llama.cpp, KTransformers and MindIE. This guide's §4 covers the same engines from the *latency* perspective — what each framework actually does to your TTFT and TPOT.
- **The gateway and production angles** — [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) is the enterprise AI gateway (routing, load, guardrails) — §7 (concurrency) and §9 (worked example) assume the gateway sits in front of the inference tier; [ai_llm/llm_agents_failures_production_guide.md](ai_llm/llm_agents_failures_production_guide.md) covers the production-failure half; [ai_llm/llm_evaluation_frameworks_guide.md](ai_llm/llm_evaluation_frameworks_guide.md) and the `ai_llm/rag/` evaluation family cover the latency-observability angle (percentiles, evals under load).
- **The trust/availability trade-offs** — [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) and [ai_verify_guide.md](ai_verify_guide.md) (just added) cross-ref lightly: latency optimisation must never silently degrade the verifiability of a bank's AI output; [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (just added) covers the availability/latency relationship (failover cost is latency at the tail).
- **The banking cluster** — [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) is the trading-latency backdrop ("latency here is revenue" — microsecond/millisecond regimes, co-location); the bank series ([../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md), [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md)) and [../banking/singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md) supply the Singapore context; [../management/business_case_development_guide.md](../management/business_case_development_guide.md) supplies the hardware-investment angle for §9.
- **The RAG family** — the 18-guide `ai_llm/rag/` family covers retrieval depth: [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) (indexes), [ai_llm/rag/rag_optimization_techniques_guide.md](ai_llm/rag/rag_optimization_techniques_guide.md) (the latency levers), [ai_llm/rag/advanced_rag_techniques_guide.md](ai_llm/rag/advanced_rag_techniques_guide.md) (reranking, hybrid), [ai_llm/rag/bm25_faiss_scann_research.md](ai_llm/rag/bm25_faiss_scann_research.md) (BM25/FAISS/SCANN benchmarks), [ai_llm/rag/rag_vs_long_context_llms_guide.md](ai_llm/rag/rag_vs_long_context_llms_guide.md) (context-length vs retrieval trade-off — a latency decision).

**Note on verification.** This guide was researched in August 2026. Claims are marked **Verified** (confirmed against primary sources during research — arXiv abstracts, vendor documentation, benchmarking documentation), **Reported** (widely reported but not independently confirmed this pass), or **flagged** inline where specifics are approximate or could not be pinned down. The consolidated claims-status table is in §12. Where this guide inherits a fact from a sibling guide, it cites the sibling rather than re-verifying.

**Reader paths.** *Architect designing a low-latency copilot today* — read §1, §2, then §9, and lift the tables from §3–§7. *ML/inference engineer* — read §2–§4 in full. *Application developer* — read §5–§7. *Banking stakeholder* — read §8, §9 and §10. *The one-paragraph answer* — the §10 summary, which is deliberately self-contained.

---

## 1. Overview — The Latency Discipline

### 1.1 What GenAI Latency Is

**GenAI latency is the time a generative-AI system takes to produce output after receiving a request** — but unlike classical service latency, it is not one number. It is a *profile*: how long until the first token arrives (TTFT), how fast the tokens then flow (TPOT), and how long the whole response takes (end-to-end). Verified — NVIDIA NIM benchmarking documentation defines TTFT as "the time from query submission to the first received token," end-to-end request latency as "how long it takes from submitting a query to receiving the full response," and inter-token latency (ITL, a.k.a. TPOT) as "the average time between consecutive tokens"; ClickHouse's engineering guide (Jul 2026) names the same four core metrics — TTFT, TPOT (also called inter-token latency), tokens per second, and end-to-end latency — and notes that OpenTelemetry's GenAI semantic conventions standardise the first two as the histogram metrics `gen_ai.server.time_to_first_token` and `gen_ai.server.time_per_output_token`.

The discipline matters because a generative response is **produced, not retrieved**: the model computes every token, and the compute has a two-phase structure that classical web services do not have (see §2). The same request can feel instant (fast first token, streaming) or glacial (slow first token, or a long monologue), and the same server can report great average latency and terrible p99 (the tail). The GenAI latency discipline is: **measure the phases separately, attack each phase with the lever that moves it, and engineer the application so the user perceives the fastest phase.**

### 1.2 Why GenAI Latency Is Different

Four structural differences separate GenAI latency from classical request/response latency:

- **Two compute phases with opposite bottlenecks.** Prefill processes the whole prompt in one pass and builds the KV cache (compute-bound — matrix multiplication, high GPU utilisation); decode then emits tokens one at a time, each step re-reading the model weights and the KV cache from high-bandwidth memory (memory-bandwidth-bound, low GPU utilisation). Verified — ClickHouse: "Prefill processes every prompt token in parallel and builds the KV cache… it is compute-bound and sets TTFT. Decode then generates one token per step, reading the whole cache each time; it is memory-bandwidth-bound and sets TPOT."
- **Latency and throughput are the same resource.** A GPU serving a batch interleaves many requests; one request's latency (queueing + decode) is another request's throughput. Continuous batching and KV-cache management exist precisely because batch size and latency fight over GPU memory. Verified — the vLLM PagedAttention paper (SOSP 2023) opens on this: "High throughput serving of LLMs requires batching sufficiently many requests at a time. However, existing systems struggle because the KV cache memory for each request is huge and grows and shrinks dynamically."
- **Output length is a first-class latency term.** End-to-end latency ≈ TTFT + (output tokens − 1) × TPOT (Verified — NVIDIA NIM docs give e2e = TTFT + generation time; ClickHouse gives the token-level decomposition). A "fast" model that writes 500 tokens at 30 ms/token still takes 15 seconds end-to-end. Classical services don't have a term that grows with response *content*.
- **Perceived latency ≠ measured latency.** Streaming converts one long wait into a short wait plus readable progress — ClickHouse cites a 10–30× difference in time-to-first-content between streamed and non-streamed responses. Perceived responsiveness is dominated by TTFT (Verified — ClickHouse: "TTFT dominates perceived responsiveness because streaming UIs hide generation time behind reading time"; adults read ~5–6 tokens per second, so a stream sustaining 20 tokens/s outpaces nearly every reader).

The definition this guide will use, stated once: **GenAI latency is the end-to-end time from request submission to completed, usable response, decomposed into TTFT (time to first token), TPOT (time per output token), and the non-model segments (queueing, retrieval, orchestration, network) — engineered against a latency budget rather than a single number.**

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **Definition** | Time from request submission to usable output; decomposed into TTFT, TPOT, and non-model segments (Verified — NVIDIA NIM, ClickHouse) |
| **Why it matters** | Perceived responsiveness is dominated by TTFT; streaming hides generation behind reading speed; the tail (p95/p99) is where users suffer (Verified — ClickHouse; Dean & Barroso, "The Tail at Scale," CACM 2013) |
| **The two phases** | Prefill (compute-bound, builds KV cache, sets TTFT) and decode (memory-bandwidth-bound, one token per step, sets TPOT) (Verified — ClickHouse) |
| **The identity** | e2e ≈ TTFT + (output tokens − 1) × TPOT; long outputs dominate regardless of component health (Verified — NVIDIA NIM, ClickHouse) |
| **The levers** | KV cache, speculative decoding, continuous batching, quantization, parallelism (§3); serving frameworks operationalise them (§4) |
| **The application layer** | Streaming, prefix caching, semantic caching (§5) convert raw speed into perceived speed and cut repeated work |
| **The retrieval layer** | HNSW indexes, hybrid search, reranking (§6) shrink the pre-model segment for RAG |
| **The concurrency layer** | Async and queueing (§7) determine how latency behaves under load — and whether TTFT grows linearly with queue depth |
| **The budget** | A per-use-case latency budget (e.g. trading assistant < 1–3 s TTFT; client copilot < 2 s) that the design is measured against (§8, §9) |
| **In one line** | Measure the phases, move each with the right lever, and make the user see the first token fast — the first token wins (§10) |

---

## 2. The Latency Anatomy — TTFT, TPOT, End-to-End, Perceived

### 2.1 Time-to-First-Token (TTFT)

**Definition (Verified — NVIDIA NIM):** TTFT measures how long you wait before seeing the model's output — the time from query submission to the first received token (if the response is non-empty). NVIDIA's benchmark tooling notes that TTFT is meaningless when the first response contains no token, and that TTFT "generally includes request queuing time, prefill time, and network latency."

**What moves it (Verified — NVIDIA NIM, ClickHouse):**
- **Prompt length (prefill).** The attention mechanism uses the full input sequence to build the KV cache before generation begins; longer prompts increase TTFT. A 500-token question and a 100,000-token document produce very different first-token waits on the same model (ClickHouse).
- **Queueing.** Under load, requests wait for a batch slot; TTFT grows with queue depth (ClickHouse's triage: "High TTFT under load, fine when idle → queueing").
- **Cold starts.** First request after a cold start waits for model weights to load onto the GPU — "tens of seconds" (ClickHouse).
- **Network and orchestration.** Client-to-server transit, proxies, and pre-model steps (retrieval, guardrails) all sit inside the TTFT window.

**Why it dominates perception (Verified — ClickHouse):** streaming UIs hide generation behind reading time; the only wait the user consciously experiences is the silence before the first token. Nielsen's response-time limits put the boundary for an uninterrupted flow of thought at about 1 second; typical targets for interactive chat with short prompts are under ~1 s TTFT, while multi-second TTFT is normal for very long contexts. The metric is exposed as a first-class histogram: vLLM records `vllm:time_to_first_token_seconds` with buckets from 1 ms upward (ClickHouse), and OpenTelemetry's GenAI conventions define `gen_ai.server.time_to_first_token`.

**The lever map:** prefill optimisations (FlashAttention, prefix caching) and queueing control (continuous batching, admission control) attack TTFT; so do retrieval-layer cuts that shrink the pre-model segment (§6). A TTFT regression often turns out to be a prompt that grew (ClickHouse) — measure TTFT bucketed by input-token count.

### 2.2 Time-Per-Output-Token (TPOT)

**Definition (Verified — NVIDIA NIM):** Inter-token latency (ITL) is the average time between consecutive tokens, also known as time per output token (TPOT). NVIDIA's AIPerf defines it as (e2e latency − TTFT) / (total output tokens − 1) — excluding the first token so the metric characterises only the decoding part of request processing. Tools differ on whether TTFT is included in the average; AIPerf excludes it.

**What moves it (Verified — ClickHouse):** model size, quantization, batch load, and memory bandwidth. Decode is memory-bandwidth-bound: each token step must load the model weights (and read the growing KV cache) from HBM, do a small amount of arithmetic, and discard them. The practical target band is **10–50 ms per token (20–100 tokens/s per request)** (ClickHouse). Consistent inter-token latencies indicate efficient memory management, memory bandwidth and attention computation (NVIDIA).

**The subtlety:** TPOT degrades as batch size grows — batching trades per-request TPOT against server-wide throughput. This is the fundamental tension the serving frameworks in §4 manage, and the reason TPOT must be reported per request (p50/p95/p99), never as a server average. The inverse metric, tokens per second (TPS), asymptotically approaches 1/ITL per request as output length grows (NVIDIA) — 20 tokens/s outpaces a reader, so beyond that, cutting TPOT is imperceptible in a chat UI (ClickHouse).

**The lever map:** quantization (fewer bits per weight → less HBM traffic per decode step), speculative decoding (verify several draft tokens per step → fewer serial steps), KV-cache management (more requests fit per GPU → smaller effective batch per request), and model size itself. For non-interactive consumers — agents that block on full completions — TPOT across long outputs dominates end-to-end latency, and TTFT barely matters (ClickHouse); that inversion drives different designs for copilots vs agent backends.

### 2.3 End-to-End and Perceived Latency

**End-to-end (Verified — NVIDIA NIM, ClickHouse):** the time from query submission to the final token received, including queueing, batching, and network latency. The identity is e2e ≈ TTFT + (output tokens − 1) × TPOT. A response with 300 ms TTFT and 30 ms TPOT still takes over 15 seconds if the model writes 500 tokens — which is why output-length control (`max_tokens`), prompt discipline, and retrieval quality (getting the answer in fewer tokens) matter as much as any engine lever (ClickHouse). The four core metrics decompose any slow response into "slow to start," "slow to stream," or "simply long" — the triage order in ClickHouse's guide: high TTFT + long prompt → prefill (trim context, prefix-cache the static system prompt); high TTFT first-request-only → cold start (keep instances warm); high TTFT under load → queueing (replicas or admission control); TTFT fine + total high → output length (tighten `max_tokens`); model-side timings fine + client slow → network/proxy/orchestration — "only a trace across the whole request reveals this."

**Perceived (Verified — ClickHouse):** perceived latency is what the user experiences, and it is governed by three effects:
1. **Streaming.** The first token arrives at TTFT instead of the full response at e2e — often a 10–30× improvement in time-to-first-content; generation beyond reading speed is hidden entirely. The trade-off: perceived quality now hinges almost wholly on TTFT.
2. **The tail.** Users suffer at the tail, not the mean — Dean and Barroso's "The Tail at Scale" (CACM 2013) applies directly to LLM serving. Dashboards should track p50/p95/p99, segmented by model, prompt version, and input-token bucket, not means (ClickHouse).
3. **Progress signals.** Where a full response is unavoidable (agent steps, batch jobs), progress indicators and partial results set the perceived pace — the banking copilot in §9 uses staged rendering for exactly this reason.

**The observability pattern (Verified — ClickHouse, NVIDIA):** wrap each model call in an OpenTelemetry span; record the first-token moment as a span event; compute TTFT/TPOT percentiles over raw request events (Langfuse-style traces over a columnar store allow slicing "p95 TTFT for prompt version B on requests over 8,000 input tokens" — a pre-bucketed histogram cannot answer that). vLLM and TGI expose Prometheus histograms natively (`vllm:time_to_first_token_seconds`, `vllm:time_per_output_token_seconds`; TGI ships OpenTelemetry tracing and Prometheus metrics — Verified — TGI docs).

### 2.4 The Anatomy Table

| Segment | Definition | What moves it | Typical target | Primary levers |
|---|---|---|---|---|
| **TTFT — time-to-first-token** | Query submitted → first token received (incl. queueing, prefill, network) (Verified — NVIDIA) | Prompt length (prefill), queueing, cold starts, network | < ~1 s interactive (short prompts); multi-second normal for long contexts (Verified — ClickHouse) | Prefix caching, FlashAttention, continuous batching, admission control, retrieval cuts (§6) |
| **TPOT — time-per-output-token** (a.k.a. ITL) | Average gap between tokens after the first: (e2e − TTFT) / (tokens − 1) (Verified — NVIDIA) | Model size, quantization, batch load, memory bandwidth (Verified — ClickHouse) | 10–50 ms (20–100 tok/s per request) (Verified — ClickHouse) | Quantization, speculative decoding, KV-cache management, model selection |
| **End-to-end** | Query submitted → final token received (Verified — NVIDIA) | TTFT + output length × TPOT (Verified — ClickHouse) | Bounded by output length; cap `max_tokens` (Verified — ClickHouse) | All of the above + output-length control, retrieval quality |
| **Perceived** | What the user experiences: TTFT dominates; streaming hides generation behind reading speed; the tail hurts (Verified — ClickHouse) | Streaming UX, TTFT, p95/p99 tail, progress signals | First token within ~1 s; stream ≥ reading speed (~5–6 tok/s) (Verified — ClickHouse, Nielsen) | Token streaming (§5), percentile observability, staged rendering |
| **Non-model segments** | Queueing, network, retrieval, orchestration, guardrails — inside TTFT or before the model call (Verified — NVIDIA, ClickHouse) | Queue depth, index latency, agent loop count, gateway hops | Millisecond-scale each; they stack | Async/queueing (§7), HNSW/hybrid/rerank (§6), gateway routing (cross-ref enterprise_ai_gateway_guide.md) |

### 2.5 A Sample Latency Timeline

One numeric walkthrough makes the anatomy concrete. The trace below adapts the worked timeline from the ClickHouse engineering guide (their example: a 3,200-token prompt, 410 output tokens) to a RAG copilot request:

```
t = 0 ms      client sends request
t = 0–50 ms   network + server queue (waiting for a batch slot)      ← queueing segment
t = 50 ms     prefill: all 3,200 prompt tokens processed, KV cache built
t = 400 ms    ── first token streamed                                 ← TTFT = 400 ms
t = 400 ms+   decode: one token per step, ~30 ms each                 ← TPOT ≈ 30 ms
t = 12.7 s    final token (410 tokens out)                            ← end-to-end ≈ 12.7 s
```

Reading this trace with the discipline of §2: the user *sees* the first token at 400 ms — that is the perceived latency in a streaming UI, and it is why the §9 copilot's TTFT budget is ~500 ms, not 12.7 s. Now apply each lever to the trace and watch which segment moves:

- **Prefix caching** (§5.2): the 3,200-token prompt is mostly static system prompt + tool schemas. With a cache hit, prefill shrinks to the ~200-token per-request tail → TTFT drops from 400 ms toward ~150–200 ms.
- **Continuous batching / admission control** (§3.3, §7.2): the 0–50 ms queue segment is the first thing to inflate under load; bounded queues and a scheduler that admits at token granularity keep it flat.
- **Quantization / speculative decoding** (§3.4, §3.2): TPOT drops from ~30 ms toward ~15–20 ms. The e2e impact is large (410 tokens × 10 ms saved ≈ 4 s), the *perceived* impact is small once the stream outpaces reading speed — the right lever depends on who the consumer is (§2.3's interactive-vs-agent inversion).
- **`max_tokens` and retrieval quality** (§6): capping output at 150 tokens with better-grounded context cuts the decode span from ~12 s to ~4.5 s — the biggest e2e win available without touching the engine.
- **Semantic caching** (§5.3): for a repeated query, the *entire* trace is replaced by a ~10–100 ms vector lookup — the only lever that makes the whole timeline disappear.

The discipline in one line: **every lever has an owner segment in the timeline** — choose levers by which segment your SLA actually cares about.

### 2.6 The Observability Contract

A latency discipline without instrumentation is an opinion. The contract below is the minimal metric set the §9 copilot ships with — every line item in the §9.2 budget has a dashboard row, and every dashboard row has an owner. Metric names are the standardised ones (Verified — OpenTelemetry GenAI semantic conventions and the vLLM Prometheus exports, as documented by ClickHouse; TGI ships OpenTelemetry tracing and Prometheus metrics — Verified — TGI docs):

| Metric | What it measures | Where it comes from | What to alert on |
|---|---|---|---|
| `gen_ai.server.time_to_first_token` / `vllm:time_to_first_token_seconds` | TTFT histogram, 1 ms upward (Verified) | Engine Prometheus export / OTel span event | p95 TTFT > budget (e.g. 500 ms desk, 1 s client) for 5 min |
| `gen_ai.server.time_per_output_token` / `vllm:time_per_output_token_seconds` | TPOT histogram (Verified) | Engine Prometheus export | p95 TPOT > 50 ms (decodes are drifting — check batch load/quantization) |
| End-to-end request latency | Full request duration incl. queueing (Verified — NVIDIA) | Gateway span | p95 e2e > budget for the class |
| Queue depth / in-flight requests | Backlog at engine and gateway (§7.2) | Engine/gateway gauge | Sustained queue > 0 — admission cap is being hit |
| Prefix-cache hit rate | Share of prefill avoided (§5.2) | Engine metric | Drop below baseline → prompt drift or eviction churn |
| Semantic-cache hit rate | Share of calls skipped (§5.3) | Application metric | Below target → threshold tuning or key-design regression |
| Retrieval segment latency (embed/HNSW/BM25/rerank) | The §6.5 line items | Pipeline traces | Any stage > its budget line for 5 min |
| Guardrail/verification latency | The §8.2 trust stage | Gateway/policy service | > budget line — verification is eating TTFT |
| Tokens per second (per request, per lane) | The §2.2 inverse of TPOT | Engine + application | Per-request TPS below reading speed (~5–6 tok/s) in the streaming lane |

Two rules govern the contract. **Percentiles, segmented** — p50/p95/p99 per lane, per prompt version, per input-token bucket (a TTFT regression is usually a prompt that grew; only segmentation shows it — Verified — ClickHouse). **Traces, not just histograms** — the evaluation and observability cluster in this repository ([ai_llm/agentops_guide.md](ai_llm/agentops_guide.md), the `ai_llm/rag/` eval family, and the eval guides) is where the *quality* half of the contract lives: latency percentiles say *how slow*, retrieval/faithfulness evals say *how wrong* — a copilot that answers fast but wrong has optimised the wrong number, and the trust trade-offs of [ai_verify_guide.md](ai_verify_guide.md) close the loop.

---

## 3. The Inference Levers

The five levers below are the canonical toolkit for moving TTFT and TPOT at the model layer. Each is verified against its primary source; the mechanics column in the §3.6 table summarises how each one acts on the latency anatomy.

### 3.1 The KV Cache

**What it is (Verified — the KV-cache literature and the PagedAttention paper, arXiv 2309.06180):** the key-value (KV) cache is a foundational optimisation in Transformer-based LLMs: it stores the attention keys and values computed for already-processed tokens, eliminating redundant recomputation of past token representations during autoregressive generation. Prefill builds it; every decode step reads it.

**Why it is a latency lever:** because its footprint scales linearly with context length (and with batch size), the KV cache is the resource that decides how many requests fit on a GPU — and therefore the effective batch size each request waits in and decodes with. The PagedAttention paper's motivating observation is exactly this: the KV cache for each request is huge and grows and shrinks dynamically, and when managed inefficiently it is wasted by fragmentation and redundant duplication, limiting the batch size. A recent survey of KV-cache optimisation strategies (arXiv 2603.20397) frames it as a "foundational optimization" whose memory footprint "scales linearly with context length, imposing critical bottlenecks on GPU memory capacity, memory bandwidth, and inference throughput as production LLMs push context windows from thousands to millions of tokens."

**The levers inside the lever:** block-based paged management (PagedAttention — §3.3), prefix caching (reuse cache for shared prompt prefixes — §5.2), multi-query/grouped-query attention (GQA — fewer KV heads, less cache per token; a standard architecture choice in modern models), and KV-cache quantization/offload (e.g. NVIDIA's CPU–GPU memory-sharing work on Grace Hopper/Blackwell for KV-cache offload — Verified as documented by NVIDIA developer blog). For the architect: the KV cache is why "context is memory" — every token you add to a prompt costs GPU memory per concurrent request, and every token you *avoid* re-processing (prefix caching) is TTFT you do not pay.

### 3.2 Speculative Decoding — Draft Models and Medusa

**The original technique (Verified — arXiv 2211.17192, Leviathan, Kalman & Matias, submitted Nov 2022, ICML 2023 Oral):** decoding K tokens from an autoregressive model takes K serial model runs. Speculative decoding breaks the serial chain: a small, fast **draft model** proposes a sequence of tokens; the large target model then verifies the draft tokens **in parallel** in a single forward pass, accepting the tokens that match its own distribution and rejecting the first mismatch (then continuing from there). Because verification is parallel, several tokens can be generated per target-model step. The authors prove the method is **lossless** — it samples from exactly the same distribution as the large model alone — and demonstrate 2×–3× acceleration on T5-XXL with identical outputs. It works on off-the-shelf models without retraining or architecture changes (Verified — abstract).

**Medusa (Verified — arXiv 2401.10774, Cai et al., Jan 2024):** removes the need for a separate draft model, which the authors identify as the main adoption barrier ("their implementation is impeded by the challenges associated with acquiring and maintaining a separate draft model"). Medusa adds **extra decoding heads** to the backbone model that predict multiple subsequent tokens in parallel, and uses a **tree-based attention** mechanism to construct and verify multiple candidate continuations simultaneously. Two recipes: **Medusa-1** fine-tunes the heads on a frozen backbone — lossless acceleration of over 2.2×; **Medusa-2** fine-tunes heads and backbone together — 2.3–3.6× speedup at the cost of a special training recipe. The family has since widened (EAGLE, lookahead decoding, and the self-drafting variants — Reported, flag: named as the broader speculative-decoding landscape in survey/tutorial sources; not re-verified against each primary paper this pass).

**Adoption note (flagged):** speculative decoding is a mainline option in vLLM and TensorRT-LLM for on-prem deployments; the speedup is largest at low batch sizes (Reported — tutorial sources state gains shrink as batch fills, because the target model's parallel-verification slots are also serving other requests). It is a TPOT lever first and foremost — it attacks the decode phase — and it is a *free* lever only when the draft/head cost is amortised.

### 3.3 Continuous Batching — Orca and vLLM's PagedAttention

**Continuous (iteration-level) batching (Verified — ClickHouse, citing the primary source):** classic request batching waits for a whole batch to finish before admitting new requests, wasting decode slots. **Continuous batching** admits and evicts requests at **token granularity** — each iteration, the scheduler packs the active requests into the batch — so a finished request's slot is immediately reused. ClickHouse attributes the introduction of the technique to the **Orca system at OSDI 2022** (Yu et al., "Orca: A Distributed Serving System for Transformer-Based Generative Models"), noting it "cuts queue time." This is the single biggest throughput lever of the modern serving stack, and because queue time is a component of TTFT, it is a latency lever too.

**PagedAttention and vLLM (Verified — arXiv 2309.06180, submitted Sep 2023, published at SOSP 2023):** vLLM's core insight is that the KV cache can be managed like virtual memory in an operating system. **PagedAttention** stores the KV cache in fixed-size **blocks** (pages) rather than one contiguous buffer per request, so memory is allocated on demand, fragmentation is eliminated, and — critically — **identical prefix blocks can be shared across requests** (the seed of prefix caching, §5.2). The paper reports 2–4× throughput improvement over the state of the art (FasterTransformer, Orca) and up to 24× vs the naive baseline; the vLLM team's own blog headline claims "23x throughput" vs the pre-continuous-batching baseline (Reported as the project's own headline figure — Verified as *their* claim via vLLM docs referencing "How continuous batching enables 23x throughput in LLM inference while reducing p50 latency"; the exact comparison baseline is theirs, not independently re-run this pass). vLLM is the reference implementation of the lever: PagedAttention (memory) + continuous batching (scheduling) + prefix caching (reuse).

### 3.4 Quantization — INT8/INT4, GPTQ and AWQ

**The principle (Verified — GPTQ and AWQ abstracts; survey sources):** quantizing weights from BF16/FP16 to INT8 or INT4 shrinks model size and, because decode is memory-bandwidth-bound, directly cuts the bytes moved per token — lowering TPOT and fitting more of the model (and more requests) on a GPU. A 70B model that needs ~140 GB of VRAM at BF16 runs in ~40 GB at INT4 (Reported — survey sources give this worked example consistently; flagged as illustrative). Quantization is the decode lever *par excellence*: easing the memory-bandwidth ceiling on decode is precisely how it lowers TPOT (ClickHouse).

**GPTQ (Verified — arXiv 2210.17323, Frantar et al., Oct 2022, ICLR 2023):** a **one-shot** post-training quantization method based on approximate **second-order information** (the Hessian), quantizing weights column-block by column-block with error compensation. It quantizes models down to **3–4 bits per weight** with negligible accuracy degradation, and for the first time ran a 175B-parameter model for generative inference on a single GPU; the paper reports end-to-end inference speedups over FP16 of ~3.25× on an A100 and ~4.5× on an A6000.

**AWQ (Verified — arXiv 2306.00978, Lin et al., MIT Han Lab, Jun 2023, MLSys 2024 Best Paper Award):** activation-aware weight quantization. AWQ's key finding: **not all weights are equally important — protecting only ~1% of salient weights** (identified by the *activation* distribution, not the weights) greatly reduces quantization error. Rather than hardware-inefficient mixed precision, AWQ derives an equivalent transformation that **scales up the salient channels** to protect them, with scales collected offline from activation statistics; no backpropagation or reconstruction, so it generalises beyond the calibration set. The companion **TinyChat** inference framework reports more than 3× speedup over the Hugging Face FP16 implementation on desktop and mobile GPUs, and the paper demonstrates running 70B Llama-2 on mobile GPUs.

**Practical band (Verified surface):** INT8 is the robust baseline with wide kernel support; INT4 (GPTQ/AWQ) pushes compression further with increasingly sophisticated error preservation (survey sources; the trade-off between accuracy and bits is model- and task-dependent — flag: exact per-model quality deltas should be re-measured on the target task, which is exactly the sort of evaluation the `ai_llm/` eval guides cover). Note the separation: GPTQ/AWQ are weight-only post-training methods; dynamic-range quantized *activations* and KV-cache quantization are separate axes that modern serving stacks increasingly expose.

### 3.5 Tensor and Pipeline Parallelism

**Tensor parallelism (TP) (structural/industry knowledge, presented as such):** splits a single layer's weight matrices across multiple GPUs; each GPU computes a shard of every layer, and results are combined with all-reduce communication at each layer boundary. It is the standard way to fit and speed up large models on multi-GPU nodes, and it is what "tensor parallelism" means in every serving framework in §4. Communication cost scales with hidden dimension per layer — it is best inside a node (NVLink), which is why TP across nodes is rarely used.

**Pipeline parallelism (PP):** splits the *layers* across GPUs (each GPU owns a contiguous stage of the model) and streams micro-batches through the stages. It reduces the per-step communication of TP at the cost of pipeline bubbles; with enough micro-batches the bubble is amortised and throughput rises. Modern practice combines both (e.g. 4-way TP × 2-way PP for an 8-GPU deployment), and the frameworks in §4 expose this as a launch flag.

**The latency relationship (structural):** parallelism is a *capacity* lever first and a latency lever second — it lets a model that does not fit on one GPU serve at all, and it shortens prefill (TTFT) for very large models by using more compute in parallel. TP/PP do not fix a memory-bandwidth-bound decode by themselves (the weights still cross HBM per step); quantization and speculative decoding attack that. For an on-prem bank fleet, parallelism is the hardware-investment decision — see [../management/business_case_development_guide.md](../management/business_case_development_guide.md) for the cost side.

### 3.6 The Levers Table

| Lever | Mechanics | Notes |
|---|---|---|
| **KV cache** | Stores attention keys/values for processed tokens; eliminates recomputation; footprint scales linearly with context length (Verified — arXiv 2603.20397; PagedAttention 2309.06180) | The resource that decides batch size and therefore latency-under-load; context is memory. Variants: GQA, KV quantization/offload, prefix sharing |
| **Speculative decoding** | Small draft model proposes tokens; target verifies them in one parallel pass; lossless (Verified — arXiv 2211.17192, ICML 2023 Oral; 2–3× on T5-XXL) | Medusa removes the draft model with extra decoding heads + tree attention: Medusa-1 2.2×, Medusa-2 2.3–3.6× (Verified — arXiv 2401.10774). TPOT lever; gains largest at low batch (flagged) |
| **Continuous batching** | Admit/evict requests at token granularity per iteration; reuse finished slots (Verified — introduced by Orca, OSDI 2022) | Cuts queue time (a TTFT component) and raises GPU utilisation; the modern serving default |
| **PagedAttention** | KV cache in fixed-size blocks, demand-paged, prefix-shared, fragmentation-free (Verified — arXiv 2309.06180, SOSP 2023) | vLLM's core; 2–4× vs prior systems per paper; "23x throughput" is the project's own blog headline (Reported) |
| **Quantization** | Fewer bits per weight (INT8/INT4) → less HBM traffic per decode step → lower TPOT (Verified — GPTQ, AWQ; ClickHouse) | GPTQ: one-shot, 2nd-order info, 3–4 bits, 175B on one GPU, ~3.25× A100 (Verified — ICLR 2023). AWQ: protect ~1% salient weights via activation-aware scaling, 3× TinyChat (Verified — MLSys 2024 BP). Accuracy deltas must be re-measured per task (flagged) |
| **Tensor / pipeline parallelism** | TP: shard each layer across GPUs + all-reduce. PP: stage layers across GPUs, micro-batch stream (structural knowledge) | Capacity lever first; shortens prefill for big models; does not fix bandwidth-bound decode; hardware-investment decision (cross-ref business_case_development_guide.md) |

### 3.7 Choosing the Lever — Symptom to Lever Map

The five levers are not interchangeable; each owns a segment of the anatomy. The mapping below starts from the symptom (which the §2 observability stack makes visible) and lands on the lever — the same triage logic the ClickHouse guide applies to diagnosis, extended with the lever that fixes each cause:

| Symptom you observe | Likely cause | Lever that owns it |
|---|---|---|
| High TTFT, prompt is long | Prefill cost on a big prompt | Prefix caching for the static part; trim/compress context; retrieval cuts (§5.2, §6) |
| High TTFT on first request only | Cold start — weights loading to GPU | Warm replicas, pre-loaded weights, keep-alive (availability-as-latency, §8.2) |
| High TTFT under load, fine when idle | Queueing for batch capacity | Continuous batching, bounded queues, admission control (§3.3, §7.2) |
| TTFT fine, stream feels slow (high TPOT) | Memory-bandwidth-bound decode | Quantization (INT4/FP8), speculative decoding, smaller model (§3.2, §3.4) |
| TTFT fine, TPOT fine, e2e too long | Output too long | `max_tokens` cap, better retrieval → shorter answers, prompt discipline (§6.3) |
| GPU memory pressure / small batches | KV cache crowding out batch slots | PagedAttention-style block management, KV-cache quantization, GQA models (§3.1, §3.3) |
| Cost per request too high, repeated queries | Full pipeline re-run on near-duplicates | Semantic caching + prefix caching (§5.2, §5.3) |
| Model doesn't fit / too slow at any size | Capacity wall on one GPU | Quantization first, then tensor/pipeline parallelism (§3.4, §3.5) |

Two rules govern the map. **First, measure before moving:** the symptom table is only as good as the percentiles behind it — a TTFT regression that turns out to be a prompt that grew is fixed by context discipline, not by buying GPUs (§2.3). **Second, levers compose multiplicatively but not freely:** quantization + speculative decoding + prefix caching stack (each attacks a different segment), but every lever has a cost — accuracy risk (quantization), memory trade (prefix caching), or engineering ownership (TensorRT-LLM builds) — so the §9 worked example picks the subset that owns *its* budget lines, not all of them.

---

## 4. The Serving Frameworks

Four open-source engines dominate self-hosted LLM serving. The model-hub layer that distributes the weights these engines serve is covered in [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) — its §3.4/§6 show CSGHub orchestrating exactly these engines (vLLM, SGLang, TGI, llama.cpp, KTransformers, MindIE — Verified there). This section is the *latency* view of the same four.

### 4.1 vLLM

**What it is (Verified — arXiv 2309.06180, SOSP 2023; vLLM docs):** an open-source high-throughput LLM serving engine from UC Berkeley, built around **PagedAttention** (§3.3), continuous batching, and **prefix caching** (the docs list "Prefix caching support" among headline features). It exposes an OpenAI-compatible API, native Prometheus metrics (`vllm:time_to_first_token_seconds`, `vllm:time_per_output_token_seconds` — Verified via ClickHouse's documentation of the metric names), and is the engine TGI's own maintainers now recommend as the successor path (Verified — HF TGI docs, Aug 2026). Supports speculative decoding, quantization (GPTQ, AWQ, FP8, INT8), tensor/pipeline parallelism, and multi-LoRA serving.

**Latency character:** the default choice for high-throughput self-hosted serving; its continuous batching keeps TTFT flat under load (that is what the "23x throughput while reducing p50 latency" headline means), and its prefix caching collapses TTFT for repeated system prompts — the single most useful latency feature for banking copilots (§5.2). Watch-out: defaults favour throughput; TTFT-sensitive deployments should tune max-num-seqs and enable prefix caching explicitly.

### 4.2 Hugging Face Text Generation Inference (TGI)

**What it is (Verified — huggingface.co/docs/text-generation-inference):** Hugging Face's toolkit for deploying and serving LLMs (Llama, Falcon, StarCoder, BLOOM, GPT-NeoX, T5), production-ready with OpenTelemetry distributed tracing and Prometheus metrics. Its documented feature set is a checklist of this guide's levers: **token streaming using Server-Sent Events (SSE)**, **continuous batching** of incoming requests, tensor parallelism, Flash Attention and **Paged Attention** kernels, quantization with **bitsandbytes and GPT-Q**, safetensors weight loading, guidance for structured outputs (function calling), logits warpers, and stop sequences.

**Latency character:** the most turnkey of the four for HF-ecosystem models; its SSE streaming and guidance were early differentiators. **Important current fact (Verified — HF docs, Aug 2026): TGI is now in maintenance mode** — HF states it has "initiated the movement for optimized inference engines to rely on transformers model architectures," and recommends vLLM, SGLang, and local engines (llama.cpp, MLX) going forward. Architects should treat TGI as a stable incumbent for existing deployments, not the default for new ones.

### 4.3 NVIDIA TensorRT-LLM

**What it is (Verified — github.com/NVIDIA/TensorRT-LLM):** NVIDIA's open-source serving stack (14.5k stars at research time): a Python API to define LLMs plus Python and C++ runtimes that "orchestrate the inference execution in a performant way," with state-of-the-art optimisations on NVIDIA GPUs — the repo's own topics are Blackwell, CUDA, LLM serving, MoE, PyTorch — plus a Triton Inference Server backend and an AutoDeploy beta for PyTorch models. TensorRT-LLM is the engine beneath NVIDIA's commercial NIM offering (Reported — NVIDIA's positioning; the NIM benchmarking docs used throughout §2 are the same performance vocabulary).

**Latency character:** the low-level performance ceiling on NVIDIA hardware — in-flight (continuous) batching, paged KV cache, FP8/INT4 quantization, CUDA graphs, tensor/pipeline parallelism, and speculative decoding are all in its toolkit (structural — the optimisations are documented across NVIDIA's TensorRT-LLM and NIM materials; flagged where specific kernel claims were not re-benchmarked this pass). Cost: engine compilation and model-specific build steps make it the least turnkey of the four — the right choice when you need maximum tokens/s per H100/B200 and have the engineering to own the build pipeline. The hardware backdrop (H100/B200/TPU economics) is the [../management/business_case_development_guide.md](../management/business_case_development_guide.md) and [tco_modeling_guide.md](tco_modeling_guide.md) conversation.

### 4.4 SGLang

**What it is (Verified — arXiv 2312.07104, Zheng et al., UC Berkeley, Dec 2023):** a system for efficient execution of *structured language model programs* — a frontend language (primitives for generation and parallelism control) plus a runtime. Its headline optimisation is **RadixAttention**: automatic **KV-cache reuse across requests** by organising the cache as a radix tree over prompt prefixes (the generalisation of prefix caching — §5.2), plus **compressed finite-state machines for faster structured output decoding** (JSON schemas, etc.). The paper reports **up to 6.4× higher throughput** than state-of-the-art inference systems across agent control, reasoning, few-shot, JSON decoding, RAG pipelines, and multi-turn chat.

**Latency character:** the strongest prefix-caching story in the stack — for banking copilots with long shared system prompts, compliance preambles, and tool schemas, RadixAttention turns repeated prefill work into cache hits. Its structured-output FSM compression is directly relevant to the *latency budget of tool-calling agents* (structured output is where agents burn TTFT). SGLang is the second of the two engines HF's TGI maintainers now recommend (Verified — TGI docs).

### 4.5 The Framework Table

| Framework | Core mechanism | Latency signature | Best fit / watch-out |
|---|---|---|---|
| **vLLM** | PagedAttention KV blocks + continuous batching + prefix caching (Verified — SOSP 2023, docs) | Flat TTFT under load; prefix hits collapse prefill; p50 latency reduced at high throughput (23× headline, Reported) | Default for self-hosted serving; enable prefix caching; tune max-num-seqs for TTFT-sensitive workloads |
| **TGI** | SSE streaming, continuous batching, TP, Flash/Paged attention, bitsandbytes+GPTQ (Verified — HF docs) | Turnkey latency levers out of the box; good tracing/metrics | Stable incumbent; **maintenance mode as of Aug 2026** — HF recommends vLLM/SGLang for new builds (Verified) |
| **TensorRT-LLM** | Compiled engines, in-flight batching, paged KV, FP8/INT4, CUDA graphs; C++ runtime (Verified — repo; kernel-level claims flagged) | Highest raw tokens/s on NVIDIA silicon; lowest per-request TPOT at the cost of build complexity | Max-performance NVIDIA fleets (H100/B200); engine builds need owning; Triton backend for serving |
| **SGLang** | RadixAttention radix-tree KV reuse + compressed FSM structured output (Verified — arXiv 2312.07104) | Prefix reuse across requests; faster structured/tool output — both TTFT levers | Agent/copilot workloads with shared prefixes and JSON tool calls; up to 6.4× throughput claim (Verified — paper) |

### 4.6 The Selection Matrix

The four frameworks overlap heavily; the selection question is which *latency signature* your workload needs. The matrix below maps deployment classes to a primary and a runner-up, with the deciding property:

| Deployment class | Primary choice | Runner-up | Deciding property |
|---|---|---|---|
| **Small copilot, one GPU, low ops budget** | vLLM | TGI (only if already deployed — maintenance mode) | Turnkey OpenAI-compatible API, continuous batching, prefix caching out of the box (Verified — vLLM docs) |
| **High-throughput chat / many models behind one gateway** | vLLM | SGLang | Flat TTFT under load; gateway integration maturity ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)); multi-model support |
| **Agent-heavy workloads: tool calls, JSON schemas, shared prefixes everywhere** | SGLang | vLLM | RadixAttention prefix reuse + compressed-FSM structured output — both TTFT levers for agents (Verified — arXiv 2312.07104) |
| **Maximum tokens/s on an NVIDIA fleet (H100/B200), team owns the build** | TensorRT-LLM | vLLM (FP8 path) | Compiled engines, in-flight batching, FP8/INT4 kernels; lowest TPOT at the cost of build ownership (Verified — repo; kernel claims flagged) |
| **Structured-output-heavy, HF-ecosystem models, existing investment** | TGI (incumbent only) | SGLang | Guidance + SSE were early differentiators; maintenance mode makes it a hold, not a buy (Verified — HF docs, Aug 2026) |

Two cross-cutting selection rules. **The hub decides part of this:** the model registry you choose ([huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md)) ships pre-built integration with vLLM/SGLang/TGI — a CSGHub-style private registry makes the engine choice partly an orchestration compatibility question. **Never select on throughput alone:** a framework's headline tokens/s says nothing about your p99 TTFT under *your* prompt mix — the §7.4 load-test discipline is the actual selection test, and the framework table should be re-scored against its output.

### 4.7 The Hardware Backdrop — H100, B200, TPU

The frameworks run on silicon, and the silicon generation sets the latency floor. The landscape as of this guide's research pass (structural/industry knowledge, flagged — vendor spec sheets and analyst coverage, not re-benchmarked here): the **H100** (Hopper, 2022–2023) became the default production inference GPU for open-weight serving — its HBM3 bandwidth (~3.35 TB/s class) is precisely the resource decode is bound by (§2.2), which is why the same INT4 model serves noticeably faster TPOT on H100 than on the A100 generation; the **B200** (Blackwell, shipping through 2025–2026) roughly doubles the HBM capacity/bandwidth envelope and adds FP4/FP8 throughput — for latency, more KV-cache capacity per GPU means bigger effective batches and flatter TTFT at the same replica count (§3.1); **TPUs** remain the hyperscaler-alternative path (the Google TPU line is documented in [ai_llm/llm_optimization_complete_guide.md](ai_llm/llm_optimization_complete_guide.md) and [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md)) — relevant mainly when the inference tier rides a hyperscaler rather than the on-prem estate.

Three architectural rules follow. **Bandwidth, not flops, is the decode metric** — when comparing hardware for a chat/copilot workload, compare HBM bandwidth and KV-cache capacity, not peak FLOPS; prefill is flops-bound, decode is bandwidth-bound (Verified — ClickHouse's phase analysis applies to the hardware comparison directly). **The GPU generation shifts the lever mix** — on Blackwell-era hardware, FP8/FP4 quantization and larger KV blocks change the §3.7 trade-offs (less need for aggressive INT4, more headroom for prefix caching), so the lever table is re-scored per hardware generation. **The investment decision is a business case** — the hardware-investment angle is [../management/business_case_development_guide.md](../management/business_case_development_guide.md) and the TCO discipline of [tco_modeling_guide.md](tco_modeling_guide.md): a latency improvement of 2× on paper rarely justifies a fleet upgrade unless it moves a revenue or SLA line (§8.3's budgets are the translation layer between latency and business value).

---

## 5. The Application Patterns

The inference layer (§3–§4) sets the ceiling; the application layer decides what the user actually experiences. Three patterns dominate the low-latency GenAI application playbook.

### 5.1 Token Streaming — SSE

**What it is (Verified — TGI docs: "Token streaming using Server-Sent Events (SSE)"; W3C SSE is a standard mechanism):** the model's tokens are pushed to the client incrementally over a persistent HTTP connection. Server-Sent Events is the W3C mechanism (a long-lived `text/event-stream` response carrying `data:` frames); OpenAI-compatible APIs and all four frameworks in §4 expose a `stream: true` mode. In the OpenAI-compatible protocol, the server emits a `data:` frame per token (a chunk carrying the delta) and a final `[DONE]` frame — the `[done]` signal is what NVIDIA's benchmark tooling explicitly excludes from e2e latency (Verified — NVIDIA NIM docs).

**The latency effect (Verified — ClickHouse):** streaming converts one long wait into a short wait plus readable progress — often a **10–30× improvement in time-to-first-content** — and generation beyond reading speed (~5–6 tokens/s, Brysbaert 2019 meta-analysis cited by ClickHouse) is hidden entirely. The user's perceived latency becomes TTFT + first-streaming-delay, not e2e. The trade-offs: perceived quality now hinges almost wholly on TTFT (so a slow first token is *more* visible, not less), and streaming changes the trust surface — tokens appear before the full answer is known, which matters in banking (see §8 and the [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) trade-off discussion). Implementations must also handle mid-stream cancellation (a trader interrupting a long answer should stop token generation, not the server's decode).

**The pattern in practice:** stream tokens over SSE from the serving engine through the gateway ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)) to the client; render tokens as they arrive; show a progress/typing affordance before the first token; keep the first-hop network inside the same region/DC (the Singapore data-centre reality — [singapore_data_centres_guide.md](singapore_data_centres_guide.md)); and record the first-token timestamp as a span event for the §2 observability stack.

### 5.2 Prompt and Prefix Caching

**What it is (Verified — vLLM docs list prefix caching; SGLang's RadixAttention, arXiv 2312.07104, is the generalised form; ClickHouse: "Prefix caching reuses the KV cache for shared prompt prefixes, collapsing prefill cost for repeated system prompts"):** when many requests share a prompt prefix — the system prompt, compliance preamble, tool schemas, few-shot examples — the KV cache computed for that prefix is **reused instead of recomputed**. The prefill segment of TTFT for the shared prefix drops to a cache lookup. This is the single most bank-relevant latency pattern: a Cymbal Bank copilot's system prompt (role, compliance, tool definitions, formatting rules) can be thousands of tokens, and it is *identical* on every request — without prefix caching, every user request re-prefills it; with it, only the per-request tail (the question + retrieved context) is prefilled.

**The mechanics (Verified surface):** vLLM implements prefix caching over its PagedAttention blocks (identical prefix blocks are shared — the paper's "redundant duplication" elimination); SGLang generalises it with a radix tree that caches and evicts *all* common prefixes (multi-turn chat histories share prefixes across turns); TGI's paged-attention integration provides the same primitive. Cache hits require **exact token-level prefix match** — any change to the system prompt (even a timestamp) invalidates the prefix, which is why "keep the static prefix truly static" is a design rule (this is the caching-key discipline of [ai_llm/agent_runtime_cache_design_guide.md](ai_llm/agent_runtime_cache_design_guide.md) applied at the KV level).

**Cost side:** prefix caching trades GPU memory (cached blocks) for prefill compute — the classic caching economics, covered in depth by the runtime-cache guide (which also covers what *never* to cache in banking: client-specific, secret-bearing, or regulated content must not leak into shared cache blocks; tenant isolation of cache state is a compliance requirement).

### 5.3 Semantic Caching

**What it is (Verified — Azure Cosmos DB semantic-cache documentation; arXiv 2411.05276 "GPT Semantic Cache"):** an application-level cache that answers a **similar** (not identical) prompt with a stored response: the incoming prompt is embedded, a **vector similarity search** finds past prompts above a similarity threshold, and the cached completion is returned **without calling the LLM at all**. Microsoft's docs: "A semantic cache provides a way for you to use prior user prompts and LLM completions to address similar user prompts by using vector similarity search. A semantic cache can reduce latency and save costs in your generative AI applications since making calls to LLMs is often the most costly and highest-latency service." The GPT Semantic Cache paper (arXiv 2411.05276) reports high hit rates on predictable query categories (e.g. order/shipping queries) with embedding + in-memory cache.

**The latency effect:** a cache hit turns a 2–5 s LLM+RAG call into a 10–100 ms vector lookup — the order-of-magnitude claim the repository's own caching guide makes ([ai_llm/agent_runtime_cache_design_guide.md](ai_llm/agent_runtime_cache_design_guide.md) §1.2: full agent call 5–30 s → 10–100 ms on hit). It is the *application-layer* complement to prefix caching: prefix caching saves prefill; semantic caching saves the whole generation.

**The discipline (cross-ref the caching guide heavily):** threshold tuning (too low → wrong answers; too high → misses), cache-key design (embedding model versioning), TTL and eviction, hit-rate measurement, and the **never-cache list** — anything personalised, time-sensitive, regulated, or secret-bearing (a bank's client-specific answers must not be served from a shared semantic cache; at minimum, cache only *static-knowledge* answers and tag cache entries with freshness/regulatory metadata). Semantic caching and RAG interact: cache the *retrieved-and-answered* pair, not just the answer, and invalidate on document updates.

### 5.4 The Patterns Table

| Pattern | Mechanism | Latency effect | Key notes |
|---|---|---|---|
| **Token streaming (SSE)** | Push tokens over `text/event-stream`; OpenAI-compatible `stream: true` + `[DONE]` (Verified — TGI docs, NVIDIA) | 10–30× faster time-to-first-content; perceived latency ≈ TTFT (Verified — ClickHouse) | Perceived quality hinges on TTFT; support mid-stream cancellation; banking trust surface (tokens before full answer) |
| **Prompt / prefix caching** | Reuse KV cache for exact shared prefixes (system prompt, tools, preamble) (Verified — vLLM, SGLang RadixAttention, ClickHouse) | Collapses prefill → TTFT for the shared prefix drops to a cache hit | Requires exact token match; keep static prefixes static; tenant-isolate cache blocks in banking (cross-ref agent_runtime_cache_design_guide.md) |
| **Semantic caching** | Embed prompt → vector similarity lookup → return stored completion, no LLM call (Verified — Azure docs, arXiv 2411.05276) | Hit: 2–5 s → 10–100 ms (order-of-magnitude, cross-ref caching guide) | Threshold/TTL/eviction discipline; never cache personalised, regulated, or secret-bearing answers; invalidate on doc updates |

### 5.5 The Caching Hierarchy — Where the Patterns Sit

The repository's caching design ([ai_llm/agent_runtime_cache_design_guide.md](ai_llm/agent_runtime_cache_design_guide.md)) distinguishes four *inference-layer* cache mechanisms — prompt/prefix caching, KV-cache reuse, semantic caching, exact-match result caching (its §1.3, in the original) — from the *agent-runtime* L1–L5 hierarchy that caches whole executions (tool results, retrieval, planning). The three patterns in this guide map onto that taxonomy, and the mapping explains what each one saves:

| Pattern (this guide) | Caching-guide mechanism | What it saves | Where it lives |
|---|---|---|---|
| **Prefix caching** (§5.2) | Prompt/prefix caching + KV-cache reuse (Anthropic-style context caching; RadixAttention/PagedAttention prefix sharing) | The **prefill** segment — TTFT on every request with a shared static prefix | The serving engine (vLLM/SGLang/TGI) — application-invisible, but the application must keep the prefix static |
| **Semantic caching** (§5.3) | Semantic caching (the caching guide's L1 core — GPTCache-style embedding lookup) | The **entire generation** — the LLM call itself is skipped on a hit | The application/gateway layer, in front of the engine |
| **Exact-match result caching** (companion) | Result caching (identical prompt + params → stored response) | The entire call, with zero mis-hit risk | The application layer; the narrowest but safest cache — right for regulated static content |

Two design consequences, both inherited from the caching guide's discipline. **The layers compose:** a request that misses the semantic cache still hits the prefix cache at the engine (prefill saved), and a request that hits neither still benefits from retrieval caching (the RAG-level cache in §6.5). **The never-cache list applies at every layer:** client-specific, regulated, or secret-bearing content must not land in shared cache state — prefix blocks are shared across requests by design, so tenant isolation of the cache is a compliance requirement, not a performance choice (cross-ref the caching guide's "绝不缓存" list, which this guide's §5.3 and §9.3 enforce in the cache key).

---

## 6. The RAG Latency

Retrieval-augmented generation adds a pre-model segment to every request: embed the query, search the index, optionally rerank, then prefill the model with prompt + retrieved context. Each step is a latency line item inside TTFT. The repository's RAG family covers the retrieval *quality* discipline ([ai_llm/rag/advanced_rag_techniques_guide.md](ai_llm/rag/advanced_rag_techniques_guide.md), [ai_llm/rag/rag_optimization_techniques_guide.md](ai_llm/rag/rag_optimization_techniques_guide.md), [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md)); this section is the *latency* view of the same three levers.

### 6.1 HNSW Vector Indexes

**What it is (Verified — arXiv 1603.09320, Malkov & Yashunin, 2016):** Hierarchical Navigable Small World graphs are the dominant approximate-nearest-neighbour (ANN) index in production vector stores. HNSW builds a **multi-layer graph**: elements appear in nested layers with an exponentially decaying probability, forming a hierarchy of proximity graphs over nested subsets of the data; search starts at the top (coarse, long-range) layer and descends, using the scale separation to reach logarithmic complexity scaling — the authors describe the structure as similar to a skip list, which also makes it amenable to balanced distributed implementation.

**Why it is the latency lever:** ANN search over millions of vectors returns in **milliseconds** (typically 1–50 ms depending on index size, recall target and hardware — structural/industry knowledge, flagged: exact numbers are benchmark-dependent; the repository's own [ai_llm/rag/bm25_faiss_scann_research.md](ai_llm/rag/bm25_faiss_scann_research.md) documents FAISS/SCANN/BM25 benchmark shapes) — versus the seconds a full scan would cost. The lever inside the lever is the **recall/latency dial**: HNSW's `efSearch`/`efConstruction` parameters trade a few milliseconds against recall, and the `M` (neighbour count) trades memory against accuracy. For a copilot, the discipline is: set the index to *just enough* recall (top-k quality targets from the eval guides), because every extra millisecond in retrieval is TTFT.

**The second lever inside the index: embedding cost.** The query embedding itself is a model call (a small transformer) — often 2–10 ms, but it happens *before* the index search and it is the fixed cost every RAG request pays. Batch-embed, cache query embeddings for repeated intents (the semantic-cache interaction, §5.3), and keep the embedding model on the same GPU tier or a cheap CPU/GPU inference lane.

### 6.2 Hybrid Search

**What it is (structural/industry standard, cross-ref the RAG family):** hybrid search runs **two retrievers in parallel** — a sparse lexical retriever (BM25-style keyword matching) and the dense vector retriever — and fuses their result lists (classic fusion: reciprocal-rank fusion, RRF). The point is coverage: lexical search nails exact identifiers, product codes, ISINs, regulation references and client names; dense search captures semantic paraphrase. Neither alone is enough in a banking corpus where "the exact clause number" matters as much as "the thing about repurchase agreements."

**The latency effect:** the parallel retrieval itself adds little wall-clock time (both retrievers run in tens of milliseconds; the fusion is a sort over two small lists), but it *reduces end-to-end latency indirectly* in two ways: (1) better top-k precision means the prompt carries fewer, better context chunks — and since prefill cost scales with prompt length, fewer tokens of context is TTFT saved; (2) fewer retrieval misses means fewer "I don't know" regeneration loops in agentic flows. The [query_rewriting_rag_guide.md](ai_llm/rag/query_rewriting_rag_guide.md) and [rag_vs_hyde_guide.md](ai_llm/rag/rag_vs_hyde_guide.md) family cover the query-side variants. Watch-out: hybrid search doubles the retrieval tier's p99 unless both backends are co-located and monitored as one path — a slow BM25 shard can add a full retrieval latency budget by itself.

### 6.3 Reranking

**What it is (structural/industry standard; the RAG family covers the technique):** after fast first-stage retrieval (ANN top-50 or top-100), a **cross-encoder reranker** scores each candidate *with the query jointly encoded* — far more accurate than the bi-encoder similarity used in ANN, but too slow to run over the whole corpus. The pattern is **coarse-to-fine**: retrieve 50 cheaply, rerank 50, keep 3–5. The reranker pass typically costs 10–100 ms for a few dozen candidates (flagged — model- and hardware-dependent), which is a deliberate, budgeted spend that buys *prompt quality*.

**The latency relationship:** reranking is the one RAG stage where you *add* latency to *save* much more of it: five high-precision chunks generate a shorter, more direct answer (fewer output tokens → less TPOT and e2e), avoid hallucinated filler, and reduce the odds of a follow-up question that restarts the whole pipeline. It also pairs with §5.3: reranked answers are better semantic-cache entries. The discipline from the eval guides applies — measure retrieval quality (hit rate, MRR, faithfulness) jointly with latency, because reranking that doesn't lift answer quality is pure added TTFT.

### 6.4 The RAG Table

| Lever | Mechanism | Latency effect | Notes |
|---|---|---|---|
| **HNSW vector index** | Multi-layer navigable-small-world graph; search top-down across layers, logarithmic scaling (Verified — arXiv 1603.09320) | 1–50 ms ANN over millions of vectors (flagged — hardware-dependent); recall/latency dial via efSearch | Set recall to *just enough*; watch the embedding-model call before the search; batch/cache query embeddings |
| **Hybrid search** | Parallel BM25 (sparse) + vector (dense), fused via RRF (structural standard) | Both retrievers in tens of ms; better top-k → shorter prompts → less prefill | Coverage for identifiers/ISINs/clause numbers; monitor both backends' p99; cross-ref bm25_faiss_scann_research.md |
| **Reranking** | Cross-encoder scores top-50 candidates; keep top-3–5 (coarse-to-fine) | Adds 10–100 ms budgeted spend (flagged); saves output tokens and regeneration loops | Only worth it if answer quality measurably improves (eval guides); reranked answers feed the semantic cache |

### 6.5 The RAG Latency Budget

The retrieval segment is a *budgeted pipeline* — each stage has a line item, and the sum must fit inside the pre-model share of TTFT. A representative budget for the §9 copilot (order-of-magnitude figures, flagged as hardware-dependent):

| Stage | Budget | Notes |
|---|---|---|
| Query embedding | ~5–10 ms | Small embedding model; cached embeddings for repeated intents (§6.1) |
| Dense HNSW search (top-50) | ~10–30 ms | efSearch tuned to *just-enough* recall |
| Sparse BM25 search (top-50) | ~5–20 ms | Runs in parallel with the dense search — wall-clock ≈ max, not sum (§6.2) |
| Fusion (RRF) + filtering | ~1–5 ms | Sort over two small lists; permission/tenant filtering here |
| Rerank top-50 → top-5 | ~20–60 ms | Cross-encoder; the largest single line item, deliberately spent (§6.3) |
| Context assembly + tokenization | ~5–10 ms | Chunk ordering, citation metadata, prompt template |
| **Retrieval segment total** | **~50–130 ms** | vs ~200–400 ms prefill — retrieval must stay *smaller* than prefill or it owns TTFT |

Two decisions dominate this budget. **First, the RAG-vs-long-context decision** — the repository's [ai_llm/rag/rag_vs_long_context_llms_guide.md](ai_llm/rag/rag_vs_long_context_llms_guide.md) frames it as a latency trade: a 100K-token in-context corpus turns every request into a multi-second prefill (TTFT disaster) even before the KV-cache memory cost limits concurrency (§3.1); RAG keeps the prompt small and the prefill fast at the price of the retrieval segment above. For a latency-budgeted copilot, RAG wins unless the corpus is tiny or the latency budget is loose. **Second, the streaming-retrieval interaction** — [ai_llm/rag/rag_with_data_streaming_guide.md](ai_llm/rag/rag_with_data_streaming_guide.md) covers incremental ingestion; from the latency angle, index freshness updates must never block query paths (read-through indexes, async refresh), or the p99 of retrieval silently becomes the p99 of indexing.

---

## 7. The Concurrency

Latency under load is a concurrency problem. The inference tier is a shared, stateful resource (GPU memory, batch slots, KV cache), and how requests are admitted, awaited and queued determines whether TTFT stays flat or grows linearly with traffic.

### 7.1 Async

**What it is (structural/industry standard):** asynchronous I/O — in Python, `async`/`await` on asyncio, Starlette/FastAPI, and async HTTP clients — lets a service hold thousands of in-flight requests while using a handful of threads. For GenAI services this is not optional polish: a single LLM request occupies its connection for seconds (streaming), so a synchronous thread-per-request model exhausts threads at a few hundred concurrent users. The async pattern is: await the upstream streaming call without blocking, pump tokens through the SSE response as they arrive (§5.1), and let the event loop multiplex.

**The latency effect (structural, but note the trap):** async eliminates *thread-switch and connection-exhaustion* latency, not model latency — the GPU is still the bottleneck. The trap is the **blocking call hidden inside the async handler**: a synchronous vector-database client, a CPU-bound reranker, or a `requests`-style call inside an async route stalls the whole event loop, turning one slow call into p99 latency for everyone. Discipline: async clients end-to-end (HTTP, Redis, vector store, DB), CPU-bound stages (embedding, reranking) offloaded to worker pools or separate services, and timeouts/backpressure on every upstream await. The gateway layer ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)) is where the routing-side async discipline belongs in the enterprise pattern.

**The second trap: request fan-out.** A copilot call may await *several* upstreams (retrieval, guardrail, market-data lookup, model). Sequential awaits sum their latencies into TTFT; parallel awaits (`asyncio.gather` for independent calls) collapse them to the slowest. Every independent pre-model call should be parallel — retrieval and market-data lookup do not depend on each other, so they should not be serialised.

### 7.2 Queueing

**What it is (Verified — NVIDIA NIM: TTFT "generally includes request queuing time"; ClickHouse triage: "High TTFT under load, fine when idle → queueing"):** requests wait for a batch slot before prefill begins; queue time is a first-class component of TTFT. The serving engine's scheduler (continuous batching, §3.3) is the first queue; the gateway/ingress queue is the second; the client-side retry pile is the third, and each adds its own latency distribution to the tail.

**The discipline:** (1) **bounded queues** — an unbounded queue converts overload into ever-growing TTFT; a bounded queue converts it into fast rejections (503 + retry-after), which is the honest signal for autoscaling; (2) **admission control** — cap in-flight requests per engine replica to protect the p99; the capacity knob is max-num-seqs (vLLM) or the equivalent, and it is the *single most important TTFT stabiliser* under load; (3) **Little's law as budget math** — concurrency = arrival rate × service time; a 2 s average response at 50 req/s needs 100 in-flight slots of capacity, and the queue depth is the difference between offered concurrency and that number; (4) **priority lanes** — a trading assistant and a client copilot sharing one engine should not share one queue: priority scheduling (or separate replicas) keeps the latency-critical class out of the batch behind the bulk class; (5) **shedding** — when overloaded, drop the least valuable work first (batch jobs, pre-computable summaries) before shedding interactive traffic.

**The queueing table lives in §7.3 with the async row** — the two are the concurrency story together: async keeps the front door open, queueing keeps the back end stable.

### 7.3 The Concurrency Table

| Pattern | Mechanism | Latency effect | Discipline / notes |
|---|---|---|---|
| **Async I/O** | Non-blocking awaits end-to-end; event-loop multiplexing; streaming pumps tokens (structural) | Eliminates thread/connection-exhaustion latency; parallel `gather` for independent pre-model calls collapses sequential waits | No blocking calls in async handlers; CPU-bound stages (embed/rerank) to worker pools; timeouts on every await; gateway routing (cross-ref enterprise_ai_gateway_guide.md) |
| **Queueing** | Requests wait for batch slots; queue time is inside TTFT (Verified — NVIDIA, ClickHouse) | Under load, TTFT grows with queue depth; bounded queues convert overload into fast rejection | Bounded queues + admission control (max in-flight) protect p99; priority lanes for latency-critical classes; shedding lowest-value work first; Little's law for capacity math |

### 7.4 Load Testing and Capacity Planning

Latency under load is *measured*, not assumed — the framework selection (§4.6) and the budget (§9.2) are only as good as the load test that validates them. The discipline for GenAI services:

- **Sweep load, not just one point.** Run synthetic traffic at 25/50/75/100/125% of the target RPS and plot TTFT/TPOT percentiles (p50/p95/p99) at each level. The useful output is the **knee** — the load beyond which TTFT departs from flat — because that is where the admission cap belongs (§7.2). Averages are worthless here: the Tail-at-Scale lesson (§2.3) applies in full.
- **Test the queue, not just the engine.** The serving engine's batch scheduler, the gateway's queue, and the client retry pile are three queues in series (§7.2); load-test through the gateway with realistic request shapes (prompt-length mix, streaming vs non-streaming, the §9 two-lane priority scheme) or you are testing the wrong system.
- **Capacity math with Little's law.** For the §9 copilot: target 50 req/s at ~2.5 s average service time → ~125 in-flight requests needed (concurrency = rate × time). Two replicas at 64 max-num-seqs give 128 slots — capacity is exactly at target, so *any* growth lands in the queue; three replicas (192 slots) buy the headroom that keeps TTFT flat through announcement spikes. The queue depth on the dashboard is the difference between offered concurrency and slot capacity.
- **Soak for the KV-cache effects.** Short load tests miss the slow effects: KV-cache fragmentation patterns, prefix-cache eviction churn, and slow memory growth on long-lived deployments (§3.1). A multi-hour soak at 70% target load exposes them; a 5-minute burst proves nothing.
- **Failover is a latency test.** A cold replica joining the pool mid-incident is a tens-of-seconds TTFT event (§8.2); the failover drill should measure TTFT from the moment the primary degrades — which is why [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) belongs in the same workstream as the load test.

### 7.5 The Gateway's Role

The enterprise AI gateway ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)) is the concurrency layer's front door — routing, load, guardrails, and cost control between the application tier and the inference tier. From the latency discipline's viewpoint the gateway has four obligations: **route on latency reality** (model/engine selection per request class — the §4.6 matrix as routing policy; a degraded engine replica is taken out of rotation by health signals, not by hope); **own the admission boundary** (bounded queues, priority lanes for the §9 two-class scheme, and fast rejection — §7.2 — live here, *before* the engine's own scheduler); **preserve streaming** (the gateway must be a streaming proxy — token frames pass through untouched, with no buffering that converts SSE into a single delayed response, §5.1); and **be the observability aggregation point** (the §2.6 contract is enforced at the gateway span, which is the one place the full request path — client, retrieval, guardrail, engine — joins). A gateway that buffers, re-queues or retries indiscriminately is a latency multiplier: the discipline applies to it as strictly as to the engine, and the §7.4 load test must run *through* it.

---

## 8. The Banking Context

Banking is where GenAI latency stops being a UX nicety and becomes a revenue and risk property. The capital-markets reality — documented in [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) — is that "latency here is revenue": the front office runs in microsecond-to-millisecond regimes, and the estate's architecture is judged on hand-offs and timing. GenAI enters that estate in two distinct latency classes: trading assistants and client-facing copilots. (The market-data, co-location and event-driven backdrop is the capital-markets guide's §2/§9; this section adds the GenAI layer on top.)

### 8.1 Trading Assistants

**The use case (structural + Reported industry pattern):** desk-side copilots that summarise positions and P&L, answer risk questions against the bank's data, draft client commentary, and assist with pre-trade checks — JPMorgan's LLM Suite is the most publicised example (Verified as Reported — CNBC, Aug 2024, rolled out broadly across consumer, investment banking and asset/wealth management; Reported scale figures: 200K–250K employees with access, "83% faster portfolio research" per third-party analyses — flagged as vendor-adjacent claims not independently audited this pass; Morgan Stanley's AI @ Work assistant is the same pattern, Reported — widely covered but not re-verified this pass). For Cymbal Bank's trading floor, the assistant sits inside the [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) front office: it must coexist with, not block, the desk's real-time systems.

**The latency reality:** a trader's assistant is *interactive* but not *microsecond-critical* — the trade execution path stays on the dedicated low-latency stack; the GenAI assistant is a decision-support layer that must feel instant relative to a human's reading and reasoning. The practical budget: **TTFT well under 1 s** (the response must begin while the trader is still looking at the screen), **streaming at or above reading speed** (§2.3), and **end-to-end bounded to a few seconds** even for multi-step answers — because the trader's next decision waits on it. The one hard boundary: the assistant must never insert itself into the execution path — GenAI latency and execution latency are different regimes, and the architecture must keep them separate (cross-ref the capital-markets guide's two-speed design).

**What that means technically:** small-to-mid models (fast TPOT) or quantized large models; prefix caching for the desk's static context (position schemas, book definitions, compliance preamble); semantic caching for the repeated queries of a trading day ("what's my P&L today" variants); RAG against market-data/position documents with the §6 levers; and co-location of the inference tier with the desk data (Singapore DC reality — [singapore_data_centres_guide.md](singapore_data_centres_guide.md)) to keep the network segment out of the budget.

### 8.2 Client-Facing Copilots

**The use case (structural):** client-facing copilots — relationship-manager assistants, client self-service chat, deal- and product-explanation bots — are the public face of the bank's GenAI estate. Their latency budget is set by *customer expectation and SLA*: the repository's latency guide ([ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md) §1.3) tabulates the banking budgets — customer-service chatbot < 2 s, compliance checks < 5 s, trade summarisation < 3 s, document analysis < 10 s — and interactive chat degrades user trust beyond ~1–2 s of first-token silence (Nielsen's 1 s flow threshold, §2.3).

**What that means technically:** the same stack as the trading assistant, plus two banking-specific disciplines: (1) **the trust/latency trade-off** — streaming reveals tokens before the full answer is verified; client-facing answers need the guardrail and grounding checks from [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) and [ai_verify_guide.md](ai_verify_guide.md), and those checks must be *in the latency budget* (a guardrail call that adds 500 ms is 25% of a 2 s budget — budget it explicitly, or run cheap pre-checks in parallel with retrieval); (2) **availability is latency** — a failover or cold-start adds tens of seconds of TTFT, which is why [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (just added) matters here: warm replicas, pre-loaded weights, and active-active gateways are latency features, not just availability features.

### 8.3 The Latency Budgets

A latency budget is the contract between the business and the design — the numbers the §2 observability stack is measured against. The banking budgets this guide uses (drawn from the repository's latency guide and the capital-markets context):

| Class | Budget | Rationale |
|---|---|---|
| **Trading assistant — first token** | < 1 s TTFT (target ~300–500 ms) | Decision-support must start while the trader is still looking at the screen |
| **Trading assistant — end-to-end** | < 3–5 s for multi-step answers | The trader's next decision waits on the answer |
| **Client copilot — first token** | < 1–2 s TTFT | Nielsen flow threshold; chatbot SLA (cross-ref llm_latency_optimization_guide.md) |
| **Client copilot — end-to-end** | < 2–5 s typical; < 10 s document-heavy | Customer expectation; document analysis budget is 10 s (cross-ref) |
| **Guardrail/verification stage** | Explicit line item, not hidden | Trust checks are inside the budget (cross-ref ai_verify_guide.md) |
| **Non-interactive (reports, batch)** | 5–30 s | Progress bars acceptable; agents block on full completion (§2.3) |

### 8.4 The Banking Table

| Dimension | Trading assistant | Client-facing copilot |
|---|---|---|
| **Latency regime** | Interactive decision-support: TTFT < 1 s, e2e < 3–5 s (budget per §8.3) | Customer-facing: TTFT < 1–2 s, e2e < 2–5 s (budget per §8.3) |
| **Never in the path of** | Trade execution (microsecond regime — cross-ref capital_markets_architecture_guide.md) | Payment/onboarding decisions (the bank series: cross-ref singapore_fintech_payments_guide.md) |
| **Dominant levers** | Prefix caching (desk context), semantic caching (repeated day queries), fast TPOT models, co-located inference | Same stack + explicit guardrail budget, availability-as-latency (zero_downtime cross-ref), grounded-citation discipline |
| **Observability** | TTFT/TPOT p50/p95/p99 per desk, segmented by prompt version and input size (§2.3) | TTFT/TPOT per journey + guardrail latency + customer-facing SLA reporting |
| **Trust interaction** | Answer-before-verification risk for streaming (ai_trust_assessments cross-ref) | Verification must fit inside the budget, not after it (ai_verify cross-ref) |
| **Scale pattern** | Small user base, bursty desk hours; priority lane over shared engine (§7.2) | High concurrency, peak-hour traffic; bounded queues + autoscaling (§7.2) |

### 8.5 The Singapore Angle

Singapore is the regional operating reality for this guide's worked example, and it shapes the latency design in three ways (cross-ref the Singapore cluster: [../banking/banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md), [../banking/singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md), [singapore_data_centres_guide.md](singapore_data_centres_guide.md)):

- **Data residency is the latency architecture.** A Cymbal Bank-SG copilot serving regulated content must keep model, corpus, and cache inside Singapore-resident infrastructure — which is precisely what makes the inference tier *co-locatable* with the desk data (§9.2's ~10–20 ms network line item). The same constraint rules out convenient offshore inference for the client-facing class; the private-hub/on-prem patterns of [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) are the enabling layer.
- **Regional latency geography.** Singapore's DC ecosystem ([singapore_data_centres_guide.md](singapore_data_centres_guide.md)) gives the copilot tier the same co-location economics the trading estate already exploits (cross-ref [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) §2/§9): the GenAI tier sits in the same facilities, so the "first-hop network" line item is a DC-internal hop, not a cross-border one.
- **Verification is a local compliance conversation.** IMDA's AI Verify framework (covered in [ai_verify_guide.md](ai_verify_guide.md)) is the Singapore governance reference; for a latency guide, the point is the §8.2 one: verification and guardrail checks must fit inside the latency budget as a designed, parallel stage — the AI-governance regime and the performance budget are negotiated together, not sequentially. The MAS fintech context ([../banking/singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md)) adds the same note for anything touching payments-adjacent flows: the copilot stays out of the decision path (§8.1's hard boundary) and the latency budget reflects that separation.

---

## 9. The Worked Example — A Low-Latency Cymbal Bank RAG Copilot

### 9.1 The Scenario

Cymbal Bank Singapore is standing up a **structured-products RAG copilot** for the rates desk and client-facing relationship managers: it answers questions about the bank's structured-products documentation (term sheets, ISDA definitions, internal product policies, regulatory guidance), grounded in the bank's own corpus. Familiar context — this is the canonical Cymbal Bank copilot this repository's guides keep returning to, and it exercises every section of this guide. The business requirements, stated as latency:

- **Desk users (trading assistant class, §8.1):** first token within ~500 ms; streaming; end-to-end < 3–5 s for a grounded answer with citations.
- **Client-facing users (copilot class, §8.2):** first token within ~1 s; end-to-end < 2–5 s; the answer must be verifiable (citations to the corpus), and the verification must be *inside* the budget.
- **Load:** peak ~50 req/s across both classes, bursty around market hours and after regulatory announcements.
- **Hard boundary:** the copilot never touches the execution path; it is decision support only (cross-ref [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md)).

### 9.2 The Latency-Budget Design

The budget decomposes the ~500 ms desk TTFT target into line items, in the spirit of §2.3's identity (e2e ≈ TTFT + tokens × TPOT):

| Segment | Budget (desk) | How it is met |
|---|---|---|
| Client → gateway (Singapore DC, co-located) | ~10–20 ms | Same-region network; gateway as the single ingress ([enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md)); no cross-region hops |
| Guardrail pre-check (parallel) | ~20–50 ms (parallel with retrieval) | Cheap classifier runs *concurrently* with retrieval, not before it (§8.2 discipline) |
| Query embedding | ~5–10 ms | Small embedding model on the inference lane; cached embeddings for repeated intents (§6.1) |
| Hybrid retrieval (HNSW + BM25, RRF) | ~30–80 ms | HNSW at *just-enough* recall (efSearch tuned); BM25 shard co-located; both monitored as one path (§6.2) |
| Rerank top-50 → top-5 | ~30–60 ms | Budgeted cross-encoder pass; buys shorter, grounded prompts (§6.3) |
| Prefill (prompt = prefix-cached system prompt + question + 5 chunks) | ~100–200 ms | Prefix caching makes the ~2K-token static prefix a cache hit; only the per-request tail is prefilled (§5.2) |
| **TTFT total** | **~200–400 ms** | Under the 500 ms desk target; the queueing headroom is the difference |
| Decode (TPOT ~25–40 ms; answer capped at ~150 tokens) | ~3.5–6 s e2e at full length | Streaming shows the first tokens at TTFT; `max_tokens` cap keeps worst case inside the 3–5 s e2e budget for typical answers |
| Semantic cache in front | Hit: ~10–100 ms end-to-end | Repeated day-queries ("what's today's product list?") never reach the engine (§5.3) |

The client-facing class gets the same design with a ~1 s TTFT budget: the extra 600 ms absorbs the stricter guardrail/verification pass (grounded-citation checks, PII scan) that desk users waive, and the answer-quality bar is enforced by the eval discipline of the `ai_llm/rag/` family rather than by latency headroom.

### 9.3 The Levers Actually Applied

- **Inference stack:** vLLM or SGLang on one NVIDIA node (H100-class), INT4 quantized mid-size model (GPTQ/AWQ — quality delta re-measured per the eval guides before sign-off, §3.4), prefix caching enabled, speculative decoding on for the low-batch desk lane (flagged: gains largest at low batch, §3.2). TensorRT-LLM is the candidate if the per-token TPOT must go lower still and the team owns the build pipeline (§4.3). TGI remains viable only as an incumbent, not a new build (maintenance mode — Verified, §4.2).
- **Two lanes, one engine, two queues:** the desk lane and the client lane share the engine but not the queue — priority scheduling protects desk TTFT during client peak hours (§7.2). Bounded queues and admission control cap in-flight requests; the autoscaler reads queue depth, not CPU.
- **The caching triad:** prefix caching (static system prompt + tool schemas) at the KV layer; semantic caching at the application layer for repeated questions (never caching client-specific, regulated or secret-bearing answers — the never-cache list from [ai_llm/agent_runtime_cache_design_guide.md](ai_llm/agent_runtime_cache_design_guide.md) is enforced in the cache key); RAG-level caching of retrieved-and-answered pairs, invalidated on document updates (§5.3).
- **Observability as a contract:** TTFT/TPOT p50/p95/p99 per lane, segmented by prompt version and input-token bucket; first-token recorded as an OpenTelemetry span event; the §9.2 table is the live SLA dashboard, and a TTFT regression is triaged by the §2.3 playbook (prefill? cold start? queueing? output length? network?).
- **The agent boundary:** where the copilot evolves into multi-step agent flows (summarise → draft → send-for-approval), the *interactive* steps stay in the streaming lane, and the *blocking* steps move to the non-interactive 5–30 s lane with progress rendering (§2.3's inversion: agents block on full completions, so TTFT barely matters there).

### 9.4 The Lessons

1. **Budget first, optimise second.** The 500 ms TTFT budget was decomposed before a single knob was turned; every lever in §3–§7 was chosen because it owns a line item in §9.2. Without the budget, "optimise latency" degenerates into random tuning.
2. **The free wins are application-layer, not engine-layer.** Prefix caching, semantic caching, parallel fan-out and streaming delivered more perceived latency than any kernel flag — and they are the ones the application team actually controls (§5–§7).
3. **The tail is the SLA.** Average TTFT is a vanity metric; the p95/p99 of the queueing segment is what a trader feels on a bad day, and bounded queues + admission control are what protect it (§7.2, §2.3 — Tail at Scale).
4. **Trust fits inside the budget or it fails.** A client-facing copilot whose guardrail pass adds 800 ms misses the 1 s TTFT target; verification must be designed as a parallel, budgeted stage, not bolted on after the fact (§8.2 — cross-ref [ai_verify_guide.md](ai_verify_guide.md)).
5. **Latency and availability are the same conversation.** Warm replicas and active-active gateways are the difference between a 400 ms TTFT and a 30-second cold start during an incident — [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) is a latency reference as much as an availability one.
6. **GenAI latency is not execution latency.** The copilot's regime ends where the microsecond execution stack begins; keeping them separate is a hard architectural boundary, not a performance aspiration (cross-ref [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md)).

### 9.5 The Risk Register

The latency design is also a risk register — every lever trades something, and the worked example's mitigations are explicit:

| Risk | Latency impact | Mitigation in the design |
|---|---|---|
| Prefix-cache misses from prompt drift | TTFT reverts to full prefill (~+200 ms) | Versioned system prompts; cache-key discipline from [ai_llm/agent_runtime_cache_design_guide.md](ai_llm/agent_runtime_cache_design_guide.md); monitor prefix-hit rate as a first-class metric (§5.2) |
| Semantic cache serving stale answers after a document update | Correctness risk, not just latency | Cache the retrieved-and-answered pair; invalidate on ingestion; freshness TTL for regulated content (§5.3) |
| Queue saturation during announcement spikes | TTFT grows linearly with queue depth | Bounded queues + admission control; three replicas for headroom (§7.2, §7.4); shedding non-interactive work first |
| Cold start after deploy / failover | Tens-of-seconds TTFT | Warm replicas, pre-loaded weights, active-active gateway (§8.2, §7.4) |
| Guardrail latency creep | Eats the client-class TTFT budget | Guardrails as a parallel, budgeted stage; per-journey guardrail latency on the SLA dashboard (§8.2) |
| Quantization quality regression | Trust risk (wrong answers) — perceived as "slow to trust" | Quality delta re-measured per task before sign-off (eval guides); INT8 fallback lane for the highest-stakes class (§3.4) |
| Engine upgrade (e.g. TGI → vLLM/SGLang migration) | Transient TTFT regressions | Re-run the §7.4 load sweep against the new engine before cutover; the §4.5 table is the migration map |

### 9.6 What If the Levers Stop Working?

The discipline assumes the levers hold; the fallback playbook covers when they stop. **Token growth:** if the corpus or prompts grow, prefill re-owns TTFT despite prefix caching — the response is context discipline (trim retrieved chunks, compress tool schemas) before buying GPUs (§3.7's first row). **Load doubling:** if peak traffic doubles, the Little's-law math (§7.4) says slots must roughly double too — and the cheapest slots are the caching ones (semantic cache hit-rate improvements) before new replicas. **Model upgrade:** a larger, better model silently changes both TTFT and TPOT; the budget table (§9.2) is re-measured, not re-argued, and the §3.7 symptom map decides which new lever (if any) is needed. **The honest answer when nothing holds:** renegotiate the budget with the business — the §8.3 budgets are contracts, and contracts are re-negotiated with data, not silently broken. That is the final discipline: the latency budget is a living document, and the observability stack (§2.3) is what keeps it honest.

---

## 10. Summary — One Page

**The discipline in one paragraph.** GenAI latency is not one number but a profile — TTFT (time to first token), TPOT (time per output token), and the non-model segments — decomposed by the identity e2e ≈ TTFT + (output tokens − 1) × TPOT (Verified — NVIDIA NIM, ClickHouse). Prefill (compute-bound) sets TTFT; decode (memory-bandwidth-bound) sets TPOT; streaming hides generation behind reading speed, so perceived latency is dominated by TTFT (Verified — ClickHouse). The levers are verified and stable: the KV cache (context is memory), speculative decoding and Medusa (parallel verification, lossless — arXiv 2211.17192, 2401.10774), continuous batching and PagedAttention (Orca OSDI 2022; vLLM SOSP 2023), quantization (GPTQ ICLR 2023; AWQ MLSys 2024), and parallelism (capacity first). The frameworks operationalise them — vLLM and SGLang are today's defaults (TGI is in maintenance mode as of Aug 2026 — Verified); TensorRT-LLM is the NVIDIA performance ceiling. The application layer converts raw speed into perceived speed: token streaming (10–30× faster time-to-first-content), prefix caching (collapsed prefill for repeated system prompts), semantic caching (whole generations skipped). RAG latency is a budgeted pipeline — HNSW, hybrid search, reranking — and concurrency is the discipline that keeps TTFT flat under load: async end-to-end, bounded queues, admission control, priority lanes. In banking, the regimes split: trading assistants (TTFT < 1 s, decision-support, never in the execution path) and client-facing copilots (TTFT < 1–2 s, verification inside the budget) — cross-referencing the capital-markets and trust/verify guides. The worked example (§9) shows the whole playbook on a Cymbal Bank copilot, and the lessons are: budget first; the free wins are application-layer; the tail is the SLA; trust fits inside the budget; availability is latency; GenAI latency is not execution latency.

**The final word: the first token wins.** Every optimisation in this guide ultimately serves one moment — the instant the user sees the first token. Streaming makes that moment early; prefix caching makes the wait before it short; semantic caching makes it instant for repeated work; queueing keeps it honest under load; and observability keeps it under contract. The user does not feel the p50 of your decode phase, the elegance of your radix tree, or the bits of your quantization — they feel the silence before the first token, and then the pace at which the answer unfolds. Engineer the silence first. **The first token wins.**

**The ten rules, stated once.** (1) Measure TTFT and TPOT separately — a slow response is one of four different diseases (§2). (2) Budget before optimising — every lever owns a line item, or it is not applied (§9.2). (3) The first token is the user experience; the rest is reading speed (§2.3). (4) Prefix cache the static, semantic cache the repeated, and never cache the regulated (§5). (5) Keep retrieval smaller than prefill — HNSW, hybrid, rerank, all budgeted (§6.5). (6) Async end-to-end, but never a blocking call inside the event loop (§7.1). (7) Bounded queues and admission control are TTFT features, not ops afterthoughts (§7.2). (8) The tail is the SLA — p95/p99, segmented, always (§2.3, §2.6). (9) Verification fits inside the budget or the copilot does not ship (§8.2). (10) GenAI latency is not execution latency — keep the copilot out of the trading path (§8.1).

---

## 11. Glossary

| Term | Definition |
|---|---|
| **Latency** | The time a system takes to produce output after receiving a request; for GenAI, decomposed into TTFT, TPOT and non-model segments (§1, §2) |
| **TTFT** | Time-to-first-token: query submitted → first output token received, including queueing, prefill and network (Verified — NVIDIA NIM) |
| **Time-to-first-token** | Full name of TTFT — the dominant driver of perceived responsiveness in streamed GenAI (Verified — ClickHouse) |
| **TPOT** | Time-per-output-token: average gap between consecutive tokens after the first; a.k.a. inter-token latency (ITL); (e2e − TTFT) / (tokens − 1) (Verified — NVIDIA NIM) |
| **Time-per-output-token** | Full name of TPOT — sets the streaming pace and, with output length, the end-to-end time (Verified — ClickHouse) |
| **KV-cache** | Stored attention keys/values for processed tokens, eliminating recomputation; footprint scales with context length and batch — the resource that decides GPU batch capacity (Verified — arXiv 2603.20397, 2309.06180) |
| **Speculative decoding** | Draft model proposes tokens; target model verifies them in one parallel pass; lossless (Verified — arXiv 2211.17192, ICML 2023 Oral) |
| **Draft model** | The small, fast model that proposes token sequences for a large target model to verify (§3.2) |
| **Medusa** | Draft-model-free acceleration: extra decoding heads predict multiple tokens in parallel with tree-based attention; Medusa-1 2.2×, Medusa-2 2.3–3.6× (Verified — arXiv 2401.10774) |
| **Continuous batching** | Iteration-level scheduling that admits/evicts requests at token granularity; introduced by Orca (OSDI 2022); cuts queue time and raises utilisation (Verified — ClickHouse) |
| **PagedAttention** | vLLM's block-based KV-cache management (virtual-memory style); eliminates fragmentation, enables prefix sharing (Verified — arXiv 2309.06180, SOSP 2023) |
| **vLLM** | UC Berkeley's high-throughput serving engine: PagedAttention + continuous batching + prefix caching; OpenAI-compatible API (Verified — SOSP 2023, docs) |
| **TGI** | Hugging Face Text Generation Inference: SSE streaming, continuous batching, TP, Flash/Paged attention, bitsandbytes+GPTQ; in maintenance mode as of Aug 2026 (Verified — HF docs) |
| **TensorRT-LLM** | NVIDIA's serving stack: Python API + Python/C++ runtimes, compiled engines, in-flight batching, FP8/INT4, Triton backend (Verified — NVIDIA repo) |
| **SGLang** | Berkeley's structured-program execution engine: RadixAttention (radix-tree KV reuse) + compressed FSM structured output; up to 6.4× throughput (Verified — arXiv 2312.07104) |
| **Quantization** | Representing weights in fewer bits (INT8/INT4) to cut memory traffic per decode step and fit more on a GPU (§3.4) |
| **INT8 / INT4** | 8-bit / 4-bit integer weight formats; INT8 is the robust baseline, INT4 the aggressive compression band (survey sources) |
| **GPTQ** | One-shot post-training quantization using approximate second-order information; 3–4 bits, 175B model on one GPU, ~3.25× A100 speedup (Verified — arXiv 2210.17323, ICLR 2023) |
| **AWQ** | Activation-aware weight quantization: protects ~1% salient channels via scaling derived from activation statistics; TinyChat 3× (Verified — arXiv 2306.00978, MLSys 2024 Best Paper) |
| **Parallelism** | Splitting model computation across GPUs: tensor parallelism (shard layers) and pipeline parallelism (stage layers) (§3.5) |
| **Tensor parallelism** | Sharding each layer's weights across GPUs with all-reduce at layer boundaries; best inside a node (NVLink) (§3.5) |
| **Pipeline parallelism** | Partitioning layers into stages across GPUs, streaming micro-batches; amortises communication at the cost of pipeline bubbles (§3.5) |
| **Streaming** | Emitting tokens incrementally to the client as they are generated; converts one long wait into a short wait + readable progress (§5.1) |
| **SSE** | Server-Sent Events — the W3C mechanism (long-lived `text/event-stream`) for pushing token deltas over HTTP; used by TGI and the OpenAI-compatible protocol (Verified — TGI docs) |
| **Prompt caching** | Reusing computed KV state for the static prompt prefix across requests; collapses prefill → TTFT for shared prefixes (Verified — vLLM, ClickHouse) |
| **Prefix caching** | Generalisation of prompt caching over shared prefixes (system prompts, tool schemas, chat histories); SGLang's RadixAttention is the radix-tree form (Verified — arXiv 2312.07104) |
| **Semantic caching** | Application cache that answers *similar* prompts via embedding similarity without calling the LLM; hit turns seconds into milliseconds (Verified — Azure docs, arXiv 2411.05276) |
| **HNSW** | Hierarchical Navigable Small World graphs — the dominant ANN index: multi-layer proximity graph, logarithmic search scaling (Verified — arXiv 1603.09320) |
| **Hybrid search** | Parallel sparse (BM25) + dense (vector) retrieval fused by RRF; coverage for exact identifiers + semantic paraphrase (§6.2) |
| **Reranking** | Cross-encoder second pass over first-stage top-k candidates; coarse-to-fine: retrieve 50 cheaply, rerank, keep 3–5 (§6.3) |
| **Async** | Non-blocking I/O (async/await, event loops) that multiplexes thousands of in-flight streaming requests over few threads (§7.1) |
| **Queueing** | Requests waiting for batch slots; queue time is inside TTFT; bounded queues and admission control protect the tail (§7.2) |
| **Copilot** | Interactive GenAI assistant embedded in a user's workflow — desk copilots (trading assistants) and client-facing copilots are the banking classes (§8) |
| **Latency budget** | The per-use-case contract decomposing target latency into line items (TTFT, TPOT, retrieval, guardrails, queueing) that the design and its SLA dashboard are measured against (§8.3, §9.2) |

---

## 12. Claims Status, References and Disclaimer

**Verification convention.** Claims in this guide are marked **Verified** (confirmed against primary sources during the August 2026 research pass), **Reported** (widely reported but not independently confirmed this pass), or **flagged** (approximate, vendor-claimed, or not re-verified). Where a fact was inherited from a sibling guide in this repository, the sibling is cited rather than re-verified. No technique attribution or paper date in this guide is invented; every named paper was checked against its arXiv record.

| Claim cluster | Status | Source |
|---|---|---|
| TTFT/TPOT/ITL/e2e definitions and the e2e = TTFT + generation-time identity | Verified | NVIDIA NIM LLMs Benchmarking metrics documentation (docs.nvidia.com/nim/benchmarking/llm/latest/metrics.html) |
| Four-metric framing; OTel GenAI metric names (`gen_ai.server.time_to_first_token` / `time_per_output_token`); vLLM metric names; typical targets (TTFT < ~1 s; TPOT 10–50 ms); prefill/decode bottlenecks; streaming 10–30×; reading speed; percentiles-not-averages; prefix caching; Orca/continuous-batching attribution; TGI maintenance-mode recommendation | Verified | ClickHouse engineering — "LLM inference latency: TTFT, tokens per second, and what to measure" (Jul 2026); OpenTelemetry GenAI semantic conventions |
| PagedAttention/vLLM mechanics; KV-cache fragmentation motivation; 2–4× vs prior systems; SOSP 2023 | Verified | arXiv:2309.06180 |
| "23x throughput" headline | Reported (project's own claim) | vLLM announcing blog / docs ("How continuous batching enables 23x throughput…") |
| Speculative decoding (lossless, 2–3× T5-XXL; ICML 2023 Oral) | Verified | arXiv:2211.17192 |
| Medusa (heads, tree attention, Medusa-1 2.2× / Medusa-2 2.3–3.6×; Jan 2024) | Verified | arXiv:2401.10774 |
| EAGLE / lookahead / self-drafting landscape | Reported / flagged | Survey & tutorial sources; primary papers not all re-read this pass |
| KV-cache linear scaling with context; bottlenecks | Verified | arXiv:2603.20397 (survey); PagedAttention paper |
| GPTQ (one-shot, 2nd-order, 3–4 bits, 175B single GPU, ~3.25× A100; ICLR 2023) | Verified | arXiv:2210.17323 |
| AWQ (activation-aware, ~1% salient weights, TinyChat 3×; MLSys 2024 Best Paper) | Verified | arXiv:2306.00978 |
| HNSW (multi-layer NSW graph, logarithmic scaling, 2016) | Verified | arXiv:1603.09320 |
| SGLang / RadixAttention / compressed FSM (6.4×; Dec 2023) | Verified | arXiv:2312.07104 |
| TGI feature set (SSE, continuous batching, TP, Flash/Paged attention, bitsandbytes+GPTQ) and maintenance-mode status (Aug 2026) | Verified | huggingface.co/docs/text-generation-inference |
| TensorRT-LLM (Python API, C++ runtimes, Blackwell/CUDA/MoE, Triton backend, AutoDeploy) | Verified (repo surface); kernel-level performance claims flagged | github.com/NVIDIA/TensorRT-LLM |
| Semantic caching definition and latency/cost rationale | Verified | Microsoft Azure Cosmos DB semantic-cache docs; arXiv:2411.05276 |
| Streaming protocol shape (`stream: true`, `[DONE]` exclusion) | Verified | NVIDIA NIM docs; OpenAI-compatible API convention |
| 70B @ BF16 ≈ 140 GB → INT4 ≈ 40 GB illustration | Reported / flagged illustrative | Survey sources (consistent across sources) |
| HNSW search times 1–50 ms; rerank 10–100 ms; embedding 2–10 ms; batch-slot figures | Flagged (hardware/benchmark-dependent) | Industry knowledge; repository's bm25_faiss_scann_research.md |
| Tensor/pipeline parallelism mechanics; async/queueing patterns; hybrid search/RRF; coarse-to-fine reranking | Structural/industry knowledge, presented as such | Standard systems/IR practice |
| JPMorgan LLM Suite rollout and scale; "83% faster portfolio research"; Morgan Stanley AI @ Work | Reported (vendor-adjacent claims flagged) | CNBC (Aug 2024), JPMorganChase.com, third-party analyses |
| Banking latency budgets (chatbot < 2 s, compliance < 5 s, trade summarisation < 3 s, document < 10 s) | Inherited from sibling | ai_llm/llm_latency_optimization_guide.md §1.3 |
| Trading-floor latency regimes (microsecond/millisecond; latency is revenue) | Inherited from sibling | ../banking/capital_markets_architecture_guide.md |
| Cache-hit latency and never-cache discipline | Inherited from sibling | ai_llm/agent_runtime_cache_design_guide.md |
| Serving-engine orchestration (vLLM/SGLang/TGI/llama.cpp/KTransformers/MindIE) | Inherited from sibling | huggingface_vs_csghub_guide.md |

**Known flags / open items.** (1) The "23x throughput" figure is vLLM's own headline, not an independently re-run benchmark this pass. (2) Speculative-decoding speedup under batch load and exact per-model quantization quality deltas should be measured on the target workload before sign-off — the `ai_llm/` evaluation guides are the method for that. (3) HNSW/rerank/embedding millisecond figures are order-of-magnitude and hardware-dependent. (4) Bank GenAI scale figures are vendor-adjacent press claims, flagged as Reported. (5) TGI's maintenance-mode status and the recommendation of vLLM/SGLang are as documented by HF in August 2026 — re-check before locking a roadmap.

**Cross-reference map.** Siblings and clusters this guide leans on: [ai_llm/llm_latency_optimization_guide.md](ai_llm/llm_latency_optimization_guide.md) (techniques), [ai_llm/agent_runtime_cache_design_guide.md](ai_llm/agent_runtime_cache_design_guide.md) (caching design), [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) (model hub + serving orchestration), [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) (routing/load), [ai_trust_assessments_guide.md](ai_trust_assessments_guide.md) and [ai_verify_guide.md](ai_verify_guide.md) (trust/verification), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (availability), [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) (trading latency), [../banking/singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md) (Singapore payments context), [../management/business_case_development_guide.md](../management/business_case_development_guide.md) (hardware investment), the `ai_llm/rag/` family (retrieval depth).

**Disclaimer.** This guide is technology research for architecture planning, written from public sources as of August 2026. It is not investment advice, not a vendor endorsement, and not a regulatory interpretation. Latency figures labelled as typical or flagged are order-of-magnitude engineering guidance — measure your own workload. All product names are the property of their owners.



