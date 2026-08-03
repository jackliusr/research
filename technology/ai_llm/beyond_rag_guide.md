# Beyond RAG — The Next-Generation Paradigms Beyond Retrieval-Augmented Generation

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides — the capstone of the RAG series
> **Companion Guides:** [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) · [RAG Optimization Techniques](rag_optimization_techniques_guide.md) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [RAG with Data Streaming](rag_with_data_streaming_guide.md) · [Knowledge Distillation](../knowledge_distillation_guide.md) · [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) · [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md) · [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) · [Closed-Loop Data Engineering](closed_loop_data_engineering_guide.md) · [LLM Development Risks & Security](../llm_development_risks_security_guide.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Introduction: The End of the Retrieve-Then-Generate Era](#1-introduction-the-end-of-the-retrieve-then-generate-era)
2. [Why Beyond RAG — The Limits of Classical RAG](#2-why-beyond-rag--the-limits-of-classical-rag)
3. [Paradigm 1 — Test-Time Compute & Search-Based Reasoning](#3-paradigm-1--test-time-compute--search-based-reasoning)
4. [Paradigm 2 — Long-Context-Native Architectures](#4-paradigm-2--long-context-native-architectures)
5. [Paradigm 3 — Memory Systems](#5-paradigm-3--memory-systems)
6. [Paradigm 4 — Agentic Retrieval & Tool Use](#6-paradigm-4--agentic-retrieval--tool-use)
7. [Paradigm 5 — Retrieval-as-Training / Knowledge into Weights](#7-paradigm-5--retrieval-as-training--knowledge-into-weights)
8. [Paradigm 6 — Context Engineering](#8-paradigm-6--context-engineering)
9. [Paradigm 7 — Neuro-Symbolic & Knowledge-Graph-Grounded Generation](#9-paradigm-7--neuro-symbolic--knowledge-graph-grounded-generation)
10. [Paradigm 8 — Retrieval-Free / Parametric-Only Approaches](#10-paradigm-8--retrieval-free--parametric-only-approaches)
11. [Paradigm 9 — Self-Improving / Learning Systems](#11-paradigm-9--self-improving--learning-systems)
12. [The Synthesis — A Beyond-RAG Reference Architecture](#12-the-synthesis--a-beyond-rag-reference-architecture)
13. [Practical Guidance — What to Adopt, and When](#13-practical-guidance--what-to-adopt-and-when)
14. [The Future Trajectory (2026–2028)](#14-the-future-trajectory-2026-2028)
15. [Conclusion](#15-conclusion)
16. [References and Further Reading](#16-references-and-further-reading)

---

## 1. Introduction: The End of the Retrieve-Then-Generate Era

For three years, the canonical architecture for grounded LLM applications has been RAG: *embed the corpus, embed the query, retrieve the top-k chunks, stuff them into the prompt, generate.* It is the pattern that made LLMs usable in the enterprise — it grounds answers in documents, keeps knowledge fresh, and gives regulators citations. It is also, by 2026, increasingly recognized as **one point in a much larger design space** — and for a growing class of workloads, not the best one.

This guide is the capstone of the RAG series. It maps what comes after classical RAG: the **nine paradigms** that extend or transcend retrieve-then-generate, from test-time-compute reasoning models to long-context-native architectures, memory systems, agentic retrieval, retrieval-as-training, context engineering, neuro-symbolic generation, parametric-only models, and self-improving systems. For each paradigm it explains the mechanism, the provenance, the trade-offs, and — critically — **what is adoptable today versus what is still emerging research**, with a consistent banking lens throughout.

### 1.1 What this guide covers

- **Why beyond RAG** — the five structural limits of classical RAG (retrieval bottleneck, static corpus, shallow grounding, pipeline complexity, and the two workarounds that became their own disciplines: context-window stuffing and agentic complexity).
- **The nine paradigms** — each treated consistently: what it is, provenance, mechanism, trade-offs, implications for RAG, and adoption status.
- **The synthesis** — a beyond-RAG reference architecture that routes queries across parametric, reasoning, retrieval, long-context, memory, and graph paths, with a component maturity matrix.
- **Practical guidance** — where to start, what to pilot, how to evaluate across paradigms, what to avoid, and the banking-specific regulatory picture (SR 11-7, MAS TRM, PII, audit, guardrails).
- **The trajectory** — informed predictions for 2026–2028 and what they mean for a bank's roadmap.

### 1.2 The one-paragraph summary (for busy architects)

If you read nothing else: **classical RAG is not obsolete — it is a special case.** The retrieve-then-generate pipeline was the first successful answer to "how do we ground an LLM in knowledge," and for moderate corpora with well-specified questions it remains a fine one. But the 2025–2026 research frontier has converged on a different thesis: **retrieval, reasoning, memory, and generation are blending into a single learned capability, and the winning systems decide per-query how much of each to spend.** Reasoning models decide *whether* to retrieve; long-context models make whole-corpora-in-context viable; memory layers add persistent state; verifiers check groundedness; and feedback loops improve the whole system over time. The adoptable-today core for most enterprises: **agentic RAG + a reasoning model + a memory layer, evaluated with a unified golden set, with RAG-then-distill for repeated queries.** The rest of this guide is the map behind that sentence.

### 1.3 Relationship to the rest of the series

This guide is the **forward-looking capstone**. Each sibling guide owns a deep dive; this guide owns the horizon — it references their deep dives rather than duplicating them:

| Sibling guide | What it owns | How this guide uses it |
|---|---|---|
| [Advanced RAG Techniques](advanced_rag_techniques_guide.md) | GraphRAG, agentic RAG, Self-RAG, CRAG, adaptive routing, FLIP | The "RAG-side" baseline every beyond-RAG paradigm extends; agentic RAG is the on-ramp to Paradigm 4 |
| [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) | The context-window arms race, effective context, hybrid RAG+LC | The evidence base for Paradigm 2 and for §2.5's "arms race" critique |
| [RAG Optimization Techniques](rag_optimization_techniques_guide.md) | Chunking, hybrid retrieval, reranking, evaluation playbook | The retrieval baseline that Paradigms 3–9 assume; evaluation foundations for §11 and §13 |
| [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) | LangChain/LlamaIndex/Haystack/DSPy orchestration | Framework substrate for agentic retrieval and context engineering |
| [RAG vs HyDE](rag_vs_hyde_guide.md) / [RAG with Data Streaming](rag_with_data_streaming_guide.md) | Query-side levers / freshness pipelines | Query rewriting feeds agentic planning; streaming feeds the freshness half of Paradigm 2 |
| [Knowledge Distillation](../knowledge_distillation_guide.md) | KD fundamentals, reasoning distillation, banking governance | The training-side machinery for Paradigm 5 |
| [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md) / [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) | Multi-agent orchestration patterns | The orchestration substrate for Paradigm 4 |
| [Closed-Loop Data Engineering](closed_loop_data_engineering_guide.md) | Feedback pipelines, continuous improvement | The data-side machinery for Paradigm 9 |
| [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) | Structured output, schema-enforced generation | The output discipline for Paradigm 6 and verifier loops |
| [LLM Development Risks & Security](../llm_development_risks_security_guide.md) | OWASP LLM Top 10, guardrails, red teaming | The guardrail layer every agentic/memory system needs (§13.5) |

---

## 2. Why Beyond RAG — The Limits of Classical RAG

Classical RAG is not failing; it is **saturating**. It was designed for a world of 4k–128k context windows, static document corpora, and single-shot question answering. Each of those design assumptions has eroded. This section lays out the five structural limits — the reasons the field moved beyond, not the reasons to abandon RAG.

### 2.1 The retrieval bottleneck — recall failures propagate

RAG is a **bottleneck architecture**: the retriever is a lossy filter between the corpus and the model, and every retrieval failure is final. If the right evidence is not in the top-k, the generator cannot see it, no matter how capable it is — *garbage in, gospel out*. The failure modes compound:

- **Vocabulary mismatch** — query words that never appear in the target document defeat dense and sparse retrievers alike (the problem HyDE and query rewriting patch; see [RAG vs HyDE](rag_vs_hyde_guide.md)).
- **Chunking artifacts** — the answer straddles a chunk boundary, or the chunk context is too thin to disambiguate (see [RAG Optimization Techniques](rag_optimization_techniques_guide.md) for the chunking deep dive).
- **Semantic not syntactic** — the relevant passage requires inference ("the deal signed in Q3" vs. "the acquisition closed in September") that embeddings capture imperfectly.
- **Reranker ceiling** — rerankers can only reorder what the first stage recalled; they cannot recover missed documents.

The deeper point: **retrieval quality is a ceiling, not a floor.** Every downstream improvement (better prompts, better models) multiplies against that ceiling. Beyond-RAG paradigms attack this from multiple directions — reasoning models that retrieve iteratively (Paradigm 1/4), long contexts that bypass the top-k filter entirely (Paradigm 2), and retrieval trained end-to-end with the generator (Paradigm 5).

### 2.2 The static corpus assumption — knowledge frozen at index time

Classical RAG treats the corpus as a **snapshot**. The index reflects the world as of the last indexing job; anything that changes afterward is invisible until the next re-index. The consequences in a bank are concrete:

- A sanctions list updated this morning is not in a vector index rebuilt nightly.
- A counterparty restatement, a credit-rating change, a new legal judgment — all invisible mid-cycle.
- The [RAG with Data Streaming](rag_with_data_streaming_guide.md) guide exists precisely because this problem became urgent: CDC pipelines, event-driven indexing, and freshness SLAs are *patches* on a fundamentally snapshot-based architecture.

Beyond-RAG systems treat knowledge as **live**: long-context models can ingest today's data at query time, memory systems accumulate state across interactions, and retrieval-as-training can be re-run on demand. Freshness stops being an indexing problem and becomes a routing decision.

### 2.3 Shallow grounding — chunk-level, not reasoning-level

Classical RAG grounds the model in **chunks of text**; it does not ground the model's **reasoning**. The evidence is present in the context, but the model must still perform multi-hop inference, aggregation, contradiction detection, and entity resolution on top of unordered text fragments. The grounding is shallow in three ways:

- **No structure** — the relationships between facts (who owns whom, which clause modifies which covenant) are implicit in prose and must be re-derived by the model on every query.
- **No provenance graph** — the chain from claim → chunk → document → source is linear; there is no persistent representation of how facts relate, so follow-up questions re-pay the full inference cost.
- **No verification** — classical RAG trusts the generator to use the chunks faithfully; nothing checks whether the answer actually *follows from* the evidence. Self-RAG-style reflection ([Advanced RAG Techniques](advanced_rag_techniques_guide.md)) is a step, but it uses the same model to check itself.

Beyond-RAG systems replace shallow grounding with **structured grounding**: knowledge graphs encode relations explicitly (Paradigm 7), verifiers check entailment (Paradigm 1), and memory systems preserve the accumulation of past reasoning (Paradigm 3).

### 2.4 Pipeline complexity — many failure points, one error budget

The classical RAG pipeline — *ingest → chunk → embed → index → query → embed → retrieve → rerank → prompt → generate* — is a chain of ten-plus stages, each with its own failure modes (duplicate chunks, bad metadata, embedding drift, index corruption, reranker bias, prompt bloat). In reliability terms, it is a **serial system**: availability and correctness multiply across stages, and the error budget is consumed by the weakest link. Operationally it means:

- Every stage needs monitoring, versioning, and its own golden set.
- Diagnosing a bad answer requires tracing which stage degraded — often impossible after the fact.
- The pipeline is *rebuilt* for every new corpus (chunking tuned per document type, embeddings per language, reranker per domain).

The beyond-RAG thesis inverts this: **reduce the number of fixed stages and push decisions into the model.** A reasoning agent that calls a retrieval tool when needed has fewer hard-wired stages than a ten-stage pipeline — and its behavior is debuggable through its tool-call trace.

### 2.5 The context-window arms race as workaround

The first major workaround for RAG's limits was not architectural — it was **capacity**: build context windows so large that retrieval becomes unnecessary. The [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) guide covers this battle in full; the summary is that 1M–4M-token windows (MiniMax-Text-01 at 4M, Kimi K3 at 1M, Gemini 2.5 at 1M–2M, Claude's 1M mainstream and 4M beta) genuinely eliminate retrieval for moderate corpora, but do not eliminate it for huge corpora, and introduce their own costs (prefill latency, quadratic attention, lost-in-the-middle degradation, per-token cost). The arms race is a workaround because **it buys capacity, not capability** — a bigger window does not tell the model *what* to attend to or *whether* the answer is grounded. It also cannot scale forever: the 2026 frontier is explicit that efficiency (sparse/linear attention, KV-cache compression — Paradigm 2) matters as much as raw window size.

### 2.6 Agentic complexity — the second workaround

The second workaround — covered in [Advanced RAG Techniques](advanced_rag_techniques_guide.md) — was to make the pipeline **adaptive**: agentic RAG, where the model decides whether to retrieve, rewrites queries, iterates, and fuses multiple sources. Agentic RAG fixes the retrieval bottleneck (iterative retrieval recovers recall failures) but at the cost of **new complexity**: unbounded loops, tool-selection errors, context accumulation, and a larger attack surface. It is, in the taxonomy of this guide, the *bridge* paradigm — the first step from "retrieval as pipeline stage" to "retrieval as learned capability," and the foundation for Paradigms 1 and 4.

### 2.7 The beyond-RAG thesis

The synthesis of §2.1–§2.6 is a one-sentence thesis that frames the rest of this guide:

> **Instead of retrieving-then-generating, next-generation systems blend retrieval, reasoning, memory, and generation — retrieval becomes a learned capability, not a pipeline stage.**

Concretely, this means four shifts:

| Shift | From (classical RAG) | To (beyond RAG) |
|---|---|---|
| Retrieval | A fixed pipeline stage executed once | An action the model decides to take — when, what, how many rounds |
| Knowledge | A static snapshot in an index | Live sources, weights, memory, and graphs — selected per query |
| Grounding | Chunks in the prompt | Verified reasoning over structured evidence |
| Improvement | Manual pipeline tuning | Feedback loops that retrain retriever, generator, and corpus |

The nine paradigms that follow are the concrete instantiations of this thesis. Each is presented as a self-contained capability; the reference architecture in §12 shows how they compose.

---

*[Continue to Paradigm 1 — Test-Time Compute & Search-Based Reasoning](#3-paradigm-1--test-time-compute--search-based-reasoning)*

## 3. Paradigm 1 — Test-Time Compute & Search-Based Reasoning

**What it is.** The shift from *one forward pass* to *spend as much compute at inference as the question deserves*: models that think before answering, search over alternative reasoning paths, verify their own steps, and — critically for this guide — **decide when and what to retrieve as part of that thinking.**

**Provenance.** The lineage runs: chain-of-thought prompting (Wei et al., 2022) → self-consistency (Wang et al., 2022 — sample many CoTs, majority-vote) → tree-of-thoughts (Yao et al., 2023 — search over reasoning branches) → reasoning-optimized models (OpenAI o1, 2024) → open reasoning models (DeepSeek-R1, 2025) → the 2025–2026 reasoning-model generation (o3, Kimi K2/K3 thinking mode, Qwen QwQ, Claude's extended thinking) and inference-time search (MCTS-style search over thoughts, process reward models, verifier loops).

### 3.1 The escalation ladder: CoT → self-consistency → ToT → search

The progression is one of *increasing search over decreasing granularity of steps*:

| Stage | Mechanism | Compute cost | When it helps |
|---|---|---|---|
| Chain-of-thought | One trace of intermediate steps | ~2–5x | Multi-step math, logic, planning |
| Self-consistency | N independent CoT samples + vote | ~5–50x | Answer distributions are bimodal; reduces variance |
| Tree-of-thoughts | Branch, evaluate, backtrack | ~10–100x | Tasks with local optima — planning, puzzles |
| Inference-time search (MCTS/beam) | Value-guided expansion of reasoning states | ~100–1000x | Long-horizon tasks where step quality is checkable |
| Verification-first generation | Generate → verify → regenerate | 2–10x (but better answers) | Every task where a verifier exists — including grounded QA |

The 2026 insight: **test-time compute is a dial, not a switch.** Budgeted reasoning — scale the compute to the query's difficulty (an o1-style "reasoning effort" knob, DeepSeek-R1's controllable thinking tokens) — makes the paradigm economically viable in production. For a bank, a KYC question gets 10 seconds of thinking; a complex cross-border trade-finance dispute gets 5 minutes.

### 3.2 Reasoning models

By mid-2026, reasoning is a *standard feature*, not a premium tier: OpenAI o1/o3 with adjustable effort, DeepSeek-R1 (open weights, MIT, distilled variants), Kimi K2/K3 with native thinking mode (K3: 2.8T parameters, 1M context), Qwen QwQ, and Claude's extended thinking across Sonnet/Opus tiers. Two properties matter architecturally:

1. **They spend tokens thinking** — hidden reasoning traces that expand effective compute per query by 5–50x, which changes cost models: reasoning tokens are *input-priced but output-generated*, and prompt caching applies to the visible prefix.
2. **They plan tool calls** — reasoning models interleave deliberation with actions, which is the enabler for retrieval-as-an-action (§3.5).

### 3.3 Inference-time search: MCTS and beam search over thoughts

Beyond sampling, the frontier is **structured search over reasoning states**:

- **MCTS for LLMs** — AlphaZero-style: expand a reasoning state with candidate next thoughts, evaluate each with a value model (or rollouts), backpropagate, and search deeper where value is promising. DeepSeek-R1's lineage (and the "R1-Zero" experiments) demonstrated that pure RL with outcome rewards can produce emergent search-like behavior; 2025–2026 systems add explicit MCTS on top of RL-trained policies for long-horizon tasks.
- **Beam search over thoughts** — keep the top-k partial reasoning traces, extend each, prune by score. Cheaper than full MCTS, and a natural fit when a step-scorer exists.
- **Self-consistency as the cheap baseline** — for most production workloads, N samples + a verifier beats elaborate search at a fraction of the engineering cost. Search wins when step-level credit assignment matters (multi-hour tasks, code, formal proofs).

The retrieval angle: in these systems, **a retrieval call is a reasoning step** — the search tree branches over "what to look up next," not just "what to think next." That is agentic RAG generalized.

### 3.4 Process reward models (PRM) — step-level verification

Outcome rewards (was the final answer right?) are too sparse to guide long reasoning; **process reward models** score each step. PRMs are trained on step-level labels (human, or increasingly AI-generated via search/consensus — the R1-style RL pipeline, and the *math-shepherd*/PRM800K line of work) and used either to prune search (MCTS) or to verify finished traces. The 2025–2026 shift: **PRMs are becoming general-purpose verifiers**, not just math checkers — verifying groundedness, constraint satisfaction, and tool-call correctness.

### 3.5 Verification-first generation: generate → verify → regenerate

The most adoptable-today pattern in this paradigm is not fancy search — it is a **verification loop**:

```
draft = generate(context, question)              # pass 1
checks = verifier(draft, context, question)      # groundedness, facts, constraints
if checks.fail:
    feedback = synthesize(checks)                # what to fix, and why
    draft = generate(context, question, feedback) # pass 2 (regenerate with critique)
answer = draft + citations
```

Verifiers can be: a smaller cross-encoder-style model checking entailment between claims and chunks; a PRM scoring step quality; a constrained-decoding schema check ([Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md)); or a second LLM pass with a rubric. This is strictly stronger than Self-RAG's self-reflection ([Advanced RAG Techniques](advanced_rag_techniques_guide.md)) because the verifier is a *separate* model — it does not share the generator's blind spots.

### 3.6 Implications for RAG

Three concrete consequences for retrieval systems:

1. **Retrieval as an action inside reasoning.** Reasoning models decide *whether, what, and when* to retrieve — they do not assume retrieval. This is the mechanism behind agentic RAG's query planning and iterative retrieval, and it upgrades RAG from pipeline to policy. A query about a known product manual may get zero retrieval calls; a due-diligence question gets five, interleaved with reasoning.
2. **Verifiers check groundedness.** Beyond Self-RAG's reflection, a separate verifier can check *"does this claim follow from this chunk?"* per sentence, reject ungrounded drafts, and trigger regeneration — turning hallucination control from prompt instruction into enforced loop.
3. **Search over retrieval strategies at inference time.** The search tree can branch over *retrieval strategies* (dense vs. hybrid vs. graph vs. SQL tool) and *top-k budgets* — the model/verifier learns which strategy works for which query, per query, without a router trained offline. This is "adaptive RAG" ([Advanced RAG Techniques](advanced_rag_techniques_guide.md)) generalized to a learned policy.

### 3.7 Adoptable today

| Capability | Status | Effort |
|---|---|---|
| Reasoning model + retrieval tools (o3/R1/K2/Claude thinking + retriever tool) | **Adoptable today** | Low — API-level; orchestrate with LangGraph/LlamaIndex agents |
| Verification loop (generate → verify → regenerate) with a separate verifier | **Adoptable today** | Medium — needs a verifier model + rubric golden set |
| Budgeted test-time compute (effort knobs, query difficulty routing) | **Adoptable today** | Low–Medium — cost governance is the real work |
| MCTS/beam search over thoughts | Emerging (research → select production niches) | High — infra heavy; only for long-horizon, high-value tasks |
| General-purpose PRMs as groundedness verifiers | Emerging | Medium — PRM quality on non-math domains is uneven |

**Banking angle.** Reasoning models change the model-risk calculus: they are *less transparent* (hidden traces) but *more auditable* when tool calls and citations are surfaced. §13.5 covers the SR 11-7 implications — the short version is: document the reasoning model as a model component, keep the visible trace, and pair reasoning with verification, not faith.

---

## 4. Paradigm 2 — Long-Context-Native Architectures

**What it is.** Building systems that assume the model can *see* far more than a retrieval window: 1M–10M-token contexts, efficient attention that makes long context affordable, and input/KV-compression that keeps it fast. The full RAG-vs-long-context battle is the subject of the [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) guide — this section covers the *architecture* layer: how long context got cheap, and what that does to RAG's role.

### 4.1 The context window landscape (2025–2026)

| Model | Context window | Architecture highlight |
|---|---|---|
| MiniMax-Text-01 | 4M | Lightning attention (linear-attention hybrid) |
| Kimi K3 (Moonshot, Jul 2026) | 1M | Kimi Delta Attention + attention residuals, 2.8T params |
| Gemini 2.5 Pro | 1M (2M available) | Sparse MoE, long-context tuned |
| Claude Sonnet/Opus 4.x | 1M mainstream, 4M beta | Sliding-window + global-token hybrid |
| GPT-4.1 / o-series | 1M | Long-context trained with reinforcement-tuned attention |

The marketing number and the *effective* number still differ — lost-in-the-middle, dilution, and length-generalization gaps mean models use a fraction of the window reliably ([RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) §6). But the trend is unambiguous: by 2028, 10M-token windows at near-linear cost are plausible (§14).

### 4.2 Sparse attention — sliding windows + global tokens

The workhorse of long-context efficiency. Instead of every token attending to every token:

- **Sliding window** — attend to the last W tokens; cost scales linearly, not quadratically.
- **Global tokens** — a small set of "anchor" tokens (task instructions, document IDs, section headers, special CLS-like tokens) attend to everything and are attended-to by everything, giving the model a way to bind distant information (this is how many 1M-token production models work under the hood).
- **DeepSeek Sparse Attention (DSA)** — the 2025 frontier: *learned* sparsity where the model predicts which tokens each query needs to attend to (lightning indexer + top-k selection), cutting attention FLOPs dramatically on 1M-token contexts (DeepSeek-V3.2). DSA is the template for "retrieval *inside* the attention mechanism" — a pointed convergence with this guide's thesis.

### 4.3 Linear attention and state-space models

The quadratic-attention alternative family: **linear attention** (attention rewritten as a recurrent form — cost linear in sequence length), **state-space models** (SSMs: Mamba, RWKV) where the model compresses history into a fixed-size hidden state, and **hybrids** (Jamba — Mamba + attention layers; Hymba — parallel hybrid heads; MiniMax's Lightning Attention). Trade-offs: constant-memory inference and linear scaling, but weaker long-range *content addressing* than full attention — which is exactly why hybrids dominate the frontier (keep attention where binding matters, SSM where compression suffices).

### 4.4 Context compression at input

Make the input smaller before the model reads it:

- **LLMLingua** and its descendants — prompt compression: an alignment-tuned small LM prunes tokens (and later, structured prompts) that contribute little, shrinking context 5–20x with modest quality loss. Deep-dived in [Advanced RAG Techniques](advanced_rag_techniques_guide.md).
- **Context distillation** — train a smaller model (or a summary layer) to compress a document set into a dense representation that preserves answerability. Related to the distillation machinery in the [Knowledge Distillation](../knowledge_distillation_guide.md) guide.
- **Hierarchical summarization** — map-reduce: corpus → section summaries → document summaries → query-relevant digest. The "read-wide cheaply" pattern from [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) §7.

### 4.5 KV-cache compression

The context window is only usable if the key-value cache fits in memory. The 2025–2026 toolkit:

- **KV quantization** — 8-bit/4-bit caches (and MXFP4-style formats, as in Kimi K3's serving stack) with minimal quality loss.
- **Eviction policies** — H2O (Heavy-Hitter Oracle: keep the tokens with the highest accumulated attention scores, evict the rest), SnapKV (keep attention peaks per head), and streaming-LLM-style "attention sinks + recent window" maintenance.
- **Offloading/hierarchical memory** — KV blocks paged across GPU/CPU/NVMe (vLLM-style paged attention) so a 4M-token context fits on one node.

The architectural consequence: **long context is now an infrastructure property (cache + memory), not a model property.** You buy a big window by managing the cache, which is why prompt caching (§8.4) and KV management are becoming first-class platform capabilities in 2026.

### 4.6 Infinite context via external memory

The logical endpoint: don't fit everything in one forward pass at all.

- **MemGPT / Letta** — an OS-inspired virtual context manager: a *main context* (fast, working set) and *external context* (slow, paged), with the model issuing paging operations (summarize, evict, recall) like a process managing virtual memory. This is simultaneously a long-context architecture *and* a memory system — it is the bridge to Paradigm 3.
- **Infini-attention** (Google, 2024) — compress past context into a recurrent memory matrix inside the attention block itself; bounded-memory "infinite" context with compressive memory (the attention-side cousin of SSM states).
- **Ring attention / sequence parallelism** — distribute one huge sequence across devices so a single forward pass can cover millions of tokens without compression.

### 4.7 The implication: RAG's role shrinks — but does not vanish

With 1M–10M windows and near-linear attention, **whole-corpora-in-context becomes viable for moderate corpora** (a few hundred thousand tokens of manuals, contracts, or policy documents). For that class, RAG's chunking/embedding/retrieval machinery is pure overhead — the long-context reader is simpler, avoids chunk-boundary artifacts, and does cross-document reasoning natively ([RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) is definitive here). Retrieval remains mandatory for:

1. **Corpora > 10M tokens** (the enterprise reality: regulatory text, data rooms, historical archives) — no window will fit them; retrieval is the only way to *narrow*.
2. **Access control** — retrieval is a security boundary: per-user permissions filter what can be seen *before* generation. Stuffing a whole corpus into a context bypasses the boundary; banks cannot do that.
3. **Freshness** — live data (streaming, [RAG with Data Streaming](rag_with_data_streaming_guide.md)) must be selected at query time, not cached in a window.
4. **Cost/latency** — for high-volume interactive queries, retrieving 10 chunks still beats prefill of 1M tokens by orders of magnitude.

The 2026 pattern is therefore **hierarchical**: route small/medium stable corpora to long-context reading; route huge, permissioned, or live corpora to retrieval; and use long context as the *reader* on top of retrieved evidence (retrieve-broad, read-long).

### 4.8 Adoptable today

| Capability | Status | Notes |
|---|---|---|
| Long-context models (1M mainstream APIs) | **Adoptable today** | Pick by *effective* context, not marketing (§1.3 of the LC guide) |
| Prompt caching for static prefixes/corpora | **Adoptable today** | 10–90% cost cuts; the first thing to do |
| Hierarchical context management (summaries → full text) | **Adoptable today** | Map-reduce patterns, all frameworks |
| KV-cache quantization/eviction (self-hosted) | **Adoptable today** | vLLM/SGLang support H2O/SnapKV-style policies |
| Learned sparse attention (DSA-style) | Emerging (frontier models) | Consume via APIs; self-hosting the techniques is bleeding-edge |
| Infini-attention / infinite-memory blocks | Research | Monitor; MemGPT-style paging is the practical stand-in |

**Banking angle.** Long context changes *reading*, not *control*. The compliance perimeter (who can see what, what was used to answer) still requires retrieval-side filtering and full traceability — banks should adopt long-context readers *behind* the access-control layer, never instead of it.

---

*[Continue to Paradigm 3 — Memory Systems](#5-paradigm-3--memory-systems)*

## 5. Paradigm 3 — Memory Systems

**What it is.** The move from **stateless to stateful** generation. RAG is stateless: query → retrieve → answer, and nothing persists between turns. Memory systems add persistent state — what the system has seen, done, decided, and learned about its users — and make that state a first-class retrieval source. This is the paradigm that turns an LLM application into something with *continuity*: a relationship, not a transaction.

### 5.1 From stateless to stateful

| Property | Stateless (RAG) | Stateful (memory) |
|---|---|---|
| Input | Query + retrieved chunks | Query + retrieved chunks + accumulated state |
| Between sessions | Nothing persists | Episodic + semantic memory persists |
| Personalization | None (same answer for all users) | Preference-aware, history-aware |
| Learning | Corpus updates only | System improves from its own experience |
| Failure recovery | No continuity | Can resume prior context |

The statefulness spectrum: **short-term** (within-conversation working memory — what the chat libraries call conversation memory), **long-term/episodic** (past interactions with this user or this task), **semantic** (knowledge accumulated across interactions — user preferences, domain facts learned, decisions made), and **procedural** (how to do things — tool-usage patterns, workflows learned).

### 5.2 MemGPT / Letta — virtual context management

MemGPT (Packer et al., 2023; productized as **Letta**) is the OS-inspired design that made agent memory concrete: the model operates a **main context** (fast, working set — like RAM) and an **external context** (slow, archival — like disk), and issues *paging operations* — `heartbeat`, `recall`, `sleep`, `archival` — to move information between them, exactly as a process pages memory. Architecturally it is the bridge between Paradigm 2 (infinite context) and this paradigm (persistent state): the *same* mechanism gives you long context and memory, because both are "what belongs in the working set right now."

### 5.3 The research lineage

- **Generative Agents** (Park et al., 2023) — the Smallville town: agents with a **memory stream** (timestamped observations), **reflection** (periodic higher-level synthesis of memories into insights), and **planning** that reads from memory. The template for every memory-augmented agent since.
- **MemoryBank** (2023) — long-term memory with **forgetting curves** (Ebbinghaus-style): memories decay unless reinforced, mirroring human memory — the first principled *forget* mechanism.
- **A-MEM** (AGI Research, NeurIPS 2025) — **agentic memory**: Zettelkasten-style linked notes where each memory is a note that *grows and re-links* when related information arrives; the note graph itself is produced by the agent, and multi-hop recall emerges from the link structure. The strongest current evidence that memory *organization* matters, not just storage.
- **Mem0's paper** (2025) — the first systematic study of agent memory in production-ish settings: which operations (add/update/delete) and which storage structures matter, popularizing the "memory as an API" framing.

### 5.4 Production memory layers (2026 landscape)

| Layer | Type | Positioning |
|---|---|---|
| **Mem0** | Open-source SDK + hosted | The most widely adopted "memory as API": extract → store → retrieve memories with relevance/recency/importance scoring; works across any LLM |
| **Letta** | Open-source framework | MemGPT's product: stateful agents with self-editing memory, server for production |
| **Zep / Graphiti** | Service | Temporal knowledge-graph memory: entities + relationships with time-awareness (bi-temporal edges), built for enterprise continuity |
| **Cognee** | Open-source | Memory + knowledge graph: ECL (Extract-Cognify-Load) pipelines that turn conversations and documents into a queryable graph memory |
| **LangMem / hyperscaler memory** | SDK / platform | LangChain's memory SDK; AWS Bedrock AgentCore, Microsoft Foundry user-scoped memory, Anthropic Claude managed memory — memory is becoming a platform primitive in 2026 |

### 5.5 Memory vs RAG — the distinction that matters

RAG retrieves from a **static corpus**; memory retrieves from **experience**. The same retrieval machinery (embeddings, ranking, top-k) is used, but the source and the stakes differ:

- RAG answers "what do the documents say?" — public/enterprise knowledge, shared across users.
- Memory answers "what do we know about *this* user/task/decision?" — private, personal, cumulative.
- RAG's corpus is curated; memory is **written by the system itself** — which is why write quality and hygiene (PII, decay, consent) dominate memory engineering, and why §13.5's audit requirements bite harder here than anywhere else in this guide.

### 5.6 Memory architecture: write, store, retrieve, forget

A complete memory system is four operations, not one database:

1. **Write** — decide *what* to store: salience/importance scoring (would this matter to a future query?), deduplication (update existing memory vs. create new), and privacy filtering (strip PII before storage, or store with consent flags). Poor write policies are the #1 cause of memory bloat and recall failure.
2. **Store** — structured organization: episodic (timestamped event records), semantic (facts and preferences), procedural (workflows); increasingly *graph-structured* (Zep/Cognee/A-MEM) so recall can traverse relationships, not just similarity.
3. **Retrieve** — weighted scoring: **relevance** (semantic similarity to the query) × **recency** (recency decay, Ebbinghaus curves) × **importance** (salience at write time) — the Mem0-style three-factor formula; plus graph traversal for linked memories.
4. **Forget** — decay (unreinforced memories fade), consolidation (episodic → semantic summarization, the generative-agents reflection step), and deletion (GDPR erasure, retention policy). **Forgetting is a compliance feature, not a bug** — see §13.5.

### 5.7 Memory + RAG — the unified knowledge plane

The three composition patterns, in increasing sophistication:

1. **Memory as another retrieval source** — unified retrieval over corpus + memory: query → embed → retrieve from (a) document index and (b) memory store → fuse and rerank. Cheap to build on any RAG stack; the memory store is just a second retriever.
2. **Memory-guided retrieval** — memory *shapes* corpus retrieval: stored user preferences and past interactions rewrite the query (the user always asks about *income* when they say "money"), bias the ranking (a risk-averse user gets conservative interpretations), or filter the corpus (their jurisdiction, their product line). Personalization without custom models.
3. **Memory for agent state** — agents remember context across sessions: an onboarding agent recalls what documents a client already submitted, an AML agent remembers the investigation state across weeks of case work. This is the difference between a demo agent and a production agent.

### 5.8 Adoptable today

| Capability | Status | Effort |
|---|---|---|
| Conversation memory (short-term) | **Adoptable today** | Trivial — framework built-ins |
| Persistent memory layer (Mem0/Zep/Cognee) | **Adoptable today** | Low–Medium — SDK integration; the write-policy design is the work |
| Memory-enriched RAG (unified retrieval) | **Adoptable today** | Medium — second retriever + fusion |
| Memory-guided retrieval (preference-aware) | **Adoptable today** | Medium — needs user-profile schema + consent |
| Memory for agent state across sessions | **Adoptable today** | Medium — Letta/Mem0 agent frameworks |
| Forgetting/consolidation policies | Emerging in practice | Medium — decay curves are research-grade; retention rules are production-grade |
| Self-organizing memory (A-MEM-style note graphs) | Emerging | High — promising research; evaluate before betting |

**Banking angle.** Memory is where the PII risk concentrates: interactions, preferences, and decisions are stored and retrieved — consent, retention limits, access control, and audit trails are non-negotiable (MAS TRM and PDPA/GDPR territory; see §13.5 and the [LLM Development Risks & Security](../llm_development_risks_security_guide.md) guide). Design write-policy PII filtering and a hard delete path *before* launch, not after.

---

## 6. Paradigm 4 — Agentic Retrieval & Tool Use

**What it is.** Retrieval demoted from *the* architecture to **one tool among many**, inside an agent that plans, calls tools, reasons over results, and iterates. The agent decides which knowledge source fits the query — vector index, SQL database, web search, internal API, code execution, graph database — and how many rounds of evidence-gathering the question deserves. This is agentic RAG ([Advanced RAG Techniques](advanced_rag_techniques_guide.md) §8) generalized: *retrieval is what you do when a tool call happens to be a retriever.*

### 6.1 Retrieval as one tool among many

A 2026 knowledge agent's toolbelt looks like:

| Tool type | Example | When the agent picks it |
|---|---|---|
| Vector retrieval | Internal document index | Unstructured prose, manuals, contracts |
| Web/search | Enterprise search, web search API | Fresh external info, counterparty news |
| Database | SQL over transaction/risk systems | Structured facts, aggregations, exact values |
| Graph | Neo4j counterparty/org graph | Relationship queries, ownership chains |
| API | Internal REST services | Live operational data, workflow actions |
| Code execution | Python sandbox | Computation, what-if analysis over retrieved data |
| Memory (§5) | User/task memory store | Personalization, continuity |

The router is the model itself (with a tool-description schema), not a trained classifier — which is exactly why reasoning models (Paradigm 1) and agentic retrieval co-evolved: **tool choice is a reasoning step.**

### 6.2 Retrieval orchestration patterns

Four orchestration moves separate agentic retrieval from classical RAG:

1. **Query planning** — decompose the question into sub-queries and assign each to a tool: "What is the exposure to entity X?" → sub-queries for (a) corporate structure (graph), (b) positions (SQL), (c) news (web), (d) internal memos (vector).
2. **Iterative retrieval** — retrieve → reason → identify gaps → retrieve more: the *multi-hop* loop that fixes the recall bottleneck (§2.1). The agent can recover from a failed first retrieval by reformulating (HyDE-style) or switching tools.
3. **Tool selection (routing)** — per sub-query, choose the source; the tool descriptions themselves are the routing knowledge.
4. **Result synthesis** — merge heterogeneous evidence (a SQL aggregate + a graph path + three chunks + a news item) into one grounded answer, with per-claim provenance.

### 6.3 Frameworks

All the 2025–2026 orchestration stacks support this natively; the [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) guide compares them for retrieval workloads, and the [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md) / [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) guides cover the multi-agent layer above:

- **LangGraph** — graph-defined agents; the strongest for controllable tool loops, checkpointing, and human-in-the-loop.
- **LlamaIndex AgentWorkflow** — agent over the full LlamaIndex retrieval/query toolbox.
- **OpenAI Agents SDK** — lightweight handoffs, built-in guardrails, tracing.
- **Claude Agent SDK** — tool use with thinking models, sub-agents, and the MCP tool ecosystem.
- **AutoGen / AG2** — multi-agent conversation patterns for delegating retrieval subtasks.
- **MCP** — the tool-unification layer: one protocol for retrievers, databases, and APIs, so the agent's toolbelt is pluggable.

### 6.4 Agentic vs classical RAG — the honest comparison

| Dimension | Classical RAG | Agentic retrieval |
|---|---|---|
| Retrieval | Fixed, one round, top-k | Decided per query, multi-round, multi-tool |
| Query | Used as-is (or rewritten once) | Planned, decomposed, refined mid-flight |
| Knowledge sources | One index | Heterogeneous tools |
| Failure handling | None (garbage in, gospel out) | Re-plan, re-retrieve, escalate |
| Predictability | High (deterministic pipeline) | Lower (agent behavior varies) |
| Cost/latency | Low, stable | Higher, variable — needs budgets |
| Audit | Simple trace | Tool-call trace (but complex) |

The rule: **agentic retrieval buys recall and flexibility at the price of predictability.** For high-volume, well-scoped queries, classical RAG (or long-context reading) is still the right cost profile; agentic retrieval earns its cost on complex, multi-source, low-volume questions — exactly the mix a bank's risk desk produces.

### 6.5 Banking example — the AML investigator agent

The canonical 2026 use case: a sanctions-screening / AML investigation agent that replaces a dozen point queries:

1. **Query planning** — "Is counterparty ACME Trading connected to any sanctioned entity?" decomposes into: corporate ownership (graph tool), transaction flows (SQL over payment systems), sanctions list matching (vector over watchlists), recent news (web tool), internal case history (memory).
2. **Iterative retrieval** — the graph tool returns a shell-company chain; the agent re-queries SQL for transactions along that chain, then the web tool for the shell's directors, then loops back to the graph with the new names (expanding ripple).
3. **Synthesis** — a structured case file: findings per source, confidence per link, missing information flagged, recommended next actions — every claim carrying provenance for the case record.
4. **Human-in-the-loop** — the agent drafts the STR (suspicious transaction report) and hands off to the analyst for sign-off; the investigation state persists in memory across sessions (Paradigm 3) so a case that spans weeks does not restart each day.

This is not hypothetical architecture: every major bank's 2026 AML roadmap has a variant of this agent, because it converts a *pipeline* (fixed queries against fixed sources) into a *case* (adaptive investigation with provenance).

### 6.6 Adoptable today — with guardrails

Agentic retrieval is **adoptable today** — it is the most mature beyond-RAG paradigm, because it grew out of agentic RAG. The adoption conditions, from the [LLM Development Risks & Security](../llm_development_risks_security_guide.md) guide:

- **Loop bounds** — max tool calls per query, max rounds per sub-query, escalation after N failures (never let an agent spin).
- **Tool allow-lists** — agents only see the tools their role permits; in a bank, the SQL tool is read-only, scoped, and row-limited.
- **Permission enforcement at the tool, not the prompt** — the vector index filters by ACL *inside* the retriever; the SQL tool runs under a least-privilege service account.
- **Full tool-call tracing** — every call, input, output, and timestamp logged for audit (this is also what makes agentic systems SR 11-7-reviewable).
- **Human-in-the-loop for consequential actions** — investigations draft, analysts decide.

---

## 7. Paradigm 5 — Retrieval-as-Training / Knowledge into Weights

**What it is.** Instead of retrieving at inference time, **train the knowledge into the model** — fine-tune on the domain corpus so the answers live in the weights. The retriever disappears; the corpus becomes training data. This is the paradigm that reframes RAG's core trade (context vs. weights) and it is the least "glamorous" and most underrated paradigm in this guide — because for high-volume repeated queries it is often the cheapest, fastest, and most accurate option.

### 7.1 The RAG vs fine-tuning spectrum

| Dimension | RAG (retrieve at inference) | Fine-tuning (memorize in weights) |
|---|---|---|
| Freshness | Live — corpus can change any time | Frozen at training time (until re-trained) |
| Upfront cost | Indexing only | Training runs, GPU, data curation |
| Per-query cost | Retrieval + larger context | Nothing extra — same tokens |
| Latency | Retrieval hop + big prompt | Minimal (short prompts) |
| Accuracy on corpus specifics | High when retrieval succeeds | High when knowledge is stable and frequent |
| Hallucination risk | Grounded, but retrieval failures | Memorization failures on tails |
| Citations/audit | Natural (chunks are citable) | Hard — knowledge is implicit in weights |
| Access control | Per-user at retrieval time | Impossible per-user (one set of weights) |
| Security | Prompt-injection surface via context | Fine-tuning data poisoning risk |

The honest synthesis (matching the [Knowledge Distillation](../knowledge_distillation_guide.md) guide's governance framing): **RAG for what changes and what must be auditable; fine-tuning for what is stable, high-volume, and performance-critical.** They are complements, not competitors — and the 2026 frontier is explicit that the best systems use both (see §7.4's RAG-then-distill).

### 7.2 Knowledge injection with LoRA adapters

The operational breakthrough that made "knowledge into weights" practical: **parameter-efficient fine-tuning (PEFT)**. LoRA adapters are small (tens of MB), swappable at runtime, and per-domain:

- A bank runs the *same* base model with a **credit-risk adapter**, a **trade-finance adapter**, and a **regulatory-QA adapter**, swapped per request (or per agent role).
- Adapters are cheaper to train, version, and A/B test than full fine-tunes; each domain's knowledge is isolated in its own adapter (a containment property that full fine-tuning lacks).
- Serving stacks (vLLM, SGLang) support multi-LoRA serving, so one GPU pool serves many domain adapters.

Adapters are the "context-free" knowledge mechanism: no retrieval, no prompt bloat — the knowledge is *always on*, and the latency profile is identical to the base model.

### 7.3 Retrieval-guided training

Use retrieval to select *what to train on*: instead of fine-tuning on an entire corpus (expensive, noisy, unbalanced), **retrieve the passages most relevant to the target tasks** and fine-tune on those. Concretely: mine real query logs → retrieve top passages per query → build a curated training set of (query, passage, answer) triples. This concentrates the model's capacity where it pays, and it is the natural meeting point of RAG infrastructure (the retriever already exists) and training pipelines.

### 7.4 RAG-then-distill — the killer pattern for repeated queries

The highest-value adoptable pattern in this paradigm, and the one the [Knowledge Distillation](../knowledge_distillation_guide.md) guide's machinery enables:

```
1. Generate:  run RAG on the high-volume query set (golden queries × corpus)
2. Verify:    keep only grounded, verified (query → answer) pairs   # Paradigm 1 verifier
3. Distill:   fine-tune (or LoRA) the model on the verified pairs
4. Serve:     the distilled model answers the repeated queries without retrieval
5. Monitor:   re-run steps 1-4 when the corpus or query mix drifts   # Paradigm 9 loop
```

Why it wins: the **long tail of queries** (every question asked once) stays on RAG; the **head of repeated queries** (the 5% of questions that are 60% of traffic — "what is the credit approval limit for X category?", "what does clause 12.3 cover?") moves into weights, cutting latency and cost by an order of magnitude and *removing retrieval as a failure point* for the highest-traffic path. The verification step is what keeps distillation honest: you only memorize answers you can prove were grounded.

### 7.5 The hybrid: parametric + non-parametric memory in one model

The research frontier here is older than the current wave but prophetic — models with **both** weight knowledge and retrieval built into the architecture:

- **RETRO** (DeepMind, 2022) — retrieval-augmented transformer trained *with* retrieval: chunks are retrieved during training **and** inference from a frozen neighbor index, and the model is trained end-to-end to use them. Parametric + non-parametric memory as a single architecture, not a pipeline.
- **kNN-LM** (Khandelwal et al., 2020) — interpolate the LM's next-token distribution with a kNN over a datastore of training examples: retrieval at the *token* level, generalizing the model's own memory at training time.
- **Memorizing Transformer** (Wu et al., 2022) — a knn-attention layer that retrieves from past keys during training, letting the model memorize rare patterns explicitly.

These lines of work — largely dormant during the RAG boom — are re-emerging in 2025–2026 as **retrieval-trained models** (models where retrieval is a learned internal capability, not an external pipeline): the strongest architectural expression of this guide's thesis. Expect them in frontier serving stacks by 2027–2028 (§14), with RETRO-style frozen-index hybrids the most plausible first production form.

### 7.6 Adoptable today

| Capability | Status | Effort |
|---|---|---|
| Domain LoRA adapters (per-domain knowledge) | **Adoptable today** | Medium — data curation + training infra |
| RAG-then-distill for repeated queries | **Adoptable today** | Medium — verification step is the crux |
| Retrieval-guided training data selection | **Adoptable today** | Low–Medium — reuses existing retrievers |
| Full domain fine-tuning | **Adoptable today** (with governance) | High — SR 11-7 model-validation scope |
| kNN-LM / Memorizing-Transformer-style serving | Research | High — not productized |
| RETRO-style retrieval-trained foundation models | Research (frontier labs) | Consume when available |

**Banking angle.** Distillation has a governance catch: a fine-tuned model is a **new model**, in SR 11-7 terms — it needs validation, documentation, and monitoring like any model component (the [Knowledge Distillation](../knowledge_distillation_guide.md) guide has the full banking section). The compliant pattern: keep distilled models narrow (one domain, one task), keep the verified RAG pairs as the audit trail, and re-validate on every retrain.

---

*[Continue to Paradigm 6 — Context Engineering](#8-paradigm-6--context-engineering)*

## 8. Paradigm 6 — Context Engineering

**What it is.** The discipline of **designing what goes into the context window** — selection, ordering, compression, structure, and caching — treated with the same rigor as prompt engineering was in 2023. The thesis (Karpathy, repeatedly): **"the context window is the new database."** If context is where knowledge lives, then engineering the context is the new data engineering. This paradigm is the *craft layer* that makes Paradigms 1–5 affordable: a reasoning model with a badly built context is expensive and wrong; a long-context reader with a well-built context is cheap and right.

### 8.1 Context construction — select, order, compress, structure

Every context you hand a model is a design decision, whether or not you make it consciously:

- **Select** — what goes in and what stays out: retrieved chunks, memory, tool results, instructions. Selection is a *filtering* decision (relevance, dedup, ACL).
- **Order** — ordering is an accuracy lever, not cosmetics: models attend best to the beginning and end of context (lost-in-the-middle, [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) §6). Put the instructions and the most critical evidence at the ends; bury nothing important mid-context. There is now strong evidence that *randomized* chunk order beats ranked order for some retrieval workloads (it decorrelates position bias) — a reminder that order is an empirical variable, not a stylistic one.
- **Compress** — remove redundancy before it costs tokens (dedup near-duplicate chunks, drop boilerplate, LLMLingua-style pruning).
- **Structure** — encode the context in a machine-readable schema (XML tags, JSON blocks, delimited sections) so the model can address it precisely: `<document id=.. source=..>` blocks with headers, per-chunk metadata, explicit instructions for which sections are citable.

### 8.2 Context compression

The compression toolkit (deep-dived in [Advanced RAG Techniques](advanced_rag_techniques_guide.md)): **LLMLingua**-family prompt compression (prune low-information tokens, 5–20x shrinkage), **hierarchical summarization** (map-reduce digests), **query-aware compression** (keep what the query needs, drop the rest), and **learned compression** (a small model trained to emit a condensed context — the input-side cousin of Paradigm 5's distillation). Compression is the *cheapest* performance lever in this guide: it cuts cost and latency linearly while often *improving* accuracy by removing noise — the classic "less context, better answers" effect.

### 8.3 Context distillation

Train a smaller model to **summarize/compress** a document set into a dense context that preserves answerability — a specialized application of knowledge distillation ([Knowledge Distillation](../knowledge_distillation_guide.md)). Two forms:

- **Answer-preserving distillation** — distill "given this corpus, produce the minimal context that still answers the golden question set"; the compressed context replaces the corpus in production.
- **Long-context teacher → short-context student** — a 1M-window model reads the full corpus and produces structured digests; the production model reads only the digest. This is how banks get long-context *quality* at short-context *cost*.

### 8.4 Prompt caching — the economics enabler

The quiet revolution of 2025–2026: **cache the static prefix, pay only for the delta.** Anthropic/OpenAI/Gemini cache-control blocks, and self-hosted engines (vLLM prefix caching) make repeated-context workloads 10–90% cheaper. The architectural implications:

- **Corpus-as-prefix** — put the stable corpus (or its digest) in the cached prefix and vary only the query: long-context reading at RAG-ish costs ([RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) §8 has the math).
- **System-prompt-as-prefix** — move all static instructions, tool schemas, and policy text into the cached prefix; per-query tokens shrink to the actual question.
- **Cache-aware context design** — structure contexts so the *stable* parts come first and the *variable* parts come last; every token moved from the variable tail to the cached head is money saved on every query.

Prompt caching is the reason Paradigm 2 (long context) became economically viable in production — the 2026 cost curves for 1M-token reading assume a warm cache.

### 8.5 Structured context and constrained output

Context engineering has an output side: **structured context** (XML/schema-encoded inputs, Claude-style document blocks) and **constrained decoding** (schema-enforced outputs, JSON mode, grammar constraints — the [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) guide). Together they give the model an *addressable* world: the context is a database, the answer is a typed record, and both are machine-checkable. For banks this is not a convenience — it is how you get answers that flow into downstream systems (case files, STORs, risk systems) without a parsing layer that can break.

### 8.6 Context as product — the shift in mindset

The 2026 mindset shift: **context engineering is the new prompt engineering.** Prompt engineering optimizes *instructions*; context engineering optimizes *knowledge assembly* — what the model can know, in what order, at what cost, under what cache regime. Teams that treat the context window as a database they design (schema, indexes-as-retrieval, caching, compression, lifecycle) outperform teams that treat it as a text field they fill. The concrete organizational consequences:

- Context design reviews become part of the release process (like schema reviews).
- Context *versions* are managed (cache invalidation, corpus versioning) like data migrations.
- Every prompt template is evaluated for cache-hit rate and token efficiency, not just accuracy.
- Tool outputs entering the context are *normalized* (schema-checked) before insertion — the context is an API boundary, not a dumping ground.

### 8.7 Adoptable today

| Capability | Status | Effort |
|---|---|---|
| Systematic context construction (select/order/compress/structure) | **Adoptable today** | Low — pure engineering discipline |
| Prompt caching (prefix design, cache-aware ordering) | **Adoptable today** | Low — provider SDKs + cache-control blocks |
| LLMLingua-style compression | **Adoptable today** | Low–Medium — library integration |
| Context distillation (teacher digests) | **Adoptable today** | Medium — needs a training step |
| Structured context + constrained decoding | **Adoptable today** | Medium — schema governance |
| Cache-aware corpus lifecycle (versioning, invalidation) | Emerging in practice | Medium — needs platform support |

**Banking angle.** Context engineering is the cheapest compliance investment in this guide: citation-structured contexts, schema-enforced outputs, and cached, versioned, auditable context assemblies are what make grounded answers *reviewable*. It is also the discipline that makes every other paradigm's cost numbers real.

---

## 9. Paradigm 7 — Neuro-Symbolic & Knowledge-Graph-Grounded Generation

**What it is.** Grounding generation in **explicit structure** — knowledge graphs, ontologies, and symbolic constraints — instead of (or in addition to) flat text chunks. Where RAG retrieves *prose*, graph-grounded systems retrieve *relations*: entities, edges, and paths that encode how the world fits together. This is the paradigm that answers the "shallow grounding" critique (§2.3) most directly.

### 9.1 GraphRAG — the on-ramp

Microsoft's **GraphRAG** (2024) is the reference implementation: build a knowledge graph from the corpus (entity/relation extraction), then answer *global* questions ("what are the main themes?") by community detection over the graph, and *local* questions by retrieving entity neighborhoods. **LightRAG** and the 2025 wave (GraphRAG 2.0-style, hierarchical community summarization, incremental graph updates) made it practical. The full treatment lives in [Advanced RAG Techniques](advanced_rag_techniques_guide.md) — here we note the strategic point: **GraphRAG is where retrieval stopped being "find similar text" and became "navigate structured knowledge"** — the bridge into this paradigm.

### 9.2 Knowledge graphs as structured memory

A knowledge graph is a **persistent structured memory**: entities (counterparties, instruments, clauses, people), relations (owns, guarantees, defaults, signs), and attributes (ratings, limits, dates). As a retrieval source it offers what vector similarity cannot:

- **Exactness** — "who owns whom, through how many hops?" is a traversal, not a similarity search.
- **Multi-hop determinism** — graph queries return *paths*, each hop auditable.
- **Aggregation** — counts, paths, closures, and subgraph extraction over the whole structure.
- **Schema discipline** — the ontology *is* the data contract (see §9.5).

### 9.3 Neuro-symbolic retrieval — constraints + neural similarity

The hybrid pattern: **symbolic constraints prune, neural retrieval ranks.** A query like "exposure to European shipping companies with a rating below BBB" becomes: symbolic filter (jurisdiction = EU, industry = shipping, rating < BBB) over the graph/relational store → neural similarity or LLM reasoning *within* the pruned set. This kills the classic vector-search failure where semantics dominate constraints ("BBB" similarity matches are noise). It is the pattern that makes enterprise RAG work on *structured* questions — the majority of a bank's real queries.

### 9.4 Logical reasoning over retrieved facts

Beyond retrieving structure, use it: **rule-based verification over retrieved facts.** If the graph says X owns Y and Y owns Z, and the query asks about X's exposure to Z, the system can *derive* the answer by traversal and verify the derivation chain — no generation involved until synthesis. In regulated contexts this is a feature regulators actually trust: the reasoning is a *traceable computation*, not a model's internal inference. (This is also the logic-programming lineage — Datalog/rule engines — meeting LLMs; the [Drools rule engine guide](../drools_rule_engine_alternatives_guide.md) in this repo covers the rule-engine side for readers who want that lineage.)

### 9.5 Ontology-grounded RAG

An **ontology** (the schema of the domain: entity types, relation types, constraints) turns RAG from "retrieve against a pile of text" into "retrieve against a model of the domain":

- Extraction is guided by the ontology (extract only what the schema cares about — higher precision, less noise).
- Queries are typed (the LLM maps the question onto the ontology's relations before retrieving — less ambiguity).
- Answers are validated against the schema (a "counterparty" answer that lacks an identifier is rejected before delivery).
- Cross-corpus consistency emerges (the same entity in two documents resolves to one node).

For a bank, the ontology is typically the **data model itself** — the counterparty master, the product taxonomy, the legal-entity hierarchy — which is why graph/ontology grounding integrates with existing enterprise data governance instead of fighting it.

### 9.6 Reasoning over retrieved knowledge: the KG + LLM loop

The canonical architecture (fully worked in [Neo4j Complete Guide](../neo4j_complete_guide.md) in this repo for the graph side):

```
question
  → LLM: translate question → graph query (Cypher/SPARQL) + optional vector retrieval
  → execute: graph traversal + vector search (neuro-symbolic, §9.3)
  → retrieve: paths, subgraphs, chunks
  → LLM: synthesize answer from the retrieved structure, citing paths
  → verifier: check answer against the retrieved paths (§3.5) — each claim must trace to a path
```

The 2026 refinement: **text-to-Cypher quality is now good enough to trust with guardrails** — constrained decoding (§8.5) enforces query syntax, read-only accounts enforce safety, and verification catches wrong queries. The failure mode to design against is *silent schema mismatch* (the LLM misuses a relation type and gets a correct-looking wrong answer) — hence schema validation before execution.

### 9.7 Concept-based retrieval — retrieve concepts, not chunks

The 2025 research frontier: **CORE** ("concept-based RAG" — retrieval above the token level, where the unit of retrieval is an extracted *concept* with its context, and queries retrieve concept chains rather than text spans) and **CONCORDIA** (concept-based retrieval for long-horizon planning — retrieve the *knowledge* needed to plan, not the passages that mention the words). The thesis: chunks are an accident of text segmentation; concepts are what reasoning actually consumes. This is research-grade today (§9.9) but it is the clearest statement of where retrieval is heading: **the unit of retrieval is converging with the unit of reasoning.**

### 9.8 Structured extraction pipelines — documents → graph

The enabling infrastructure: **documents → entities/relations → graph → query**. The 2026 pipeline: LLM extraction (ontology-guided, schema-validated), entity resolution/deduplication (same counterparty across documents merges), relation typing with confidence, incremental updates (new documents add edges; [RAG with Data Streaming](rag_with_data_streaming_guide.md) patterns apply to the graph too), and quality gates (precision audits on extraction — a 95%-precision entity graph beats a 99%-precision one with perfect recall). Extraction quality *is* graph quality; every downstream query inherits it.

### 9.9 Adoptable today

| Capability | Status | Effort |
|---|---|---|
| GraphRAG/LightRAG on entity-heavy corpora | **Adoptable today** | Medium — graph infra + extraction pipeline |
| Neuro-symbolic retrieval (constraint filter + neural rank) | **Adoptable today** | Low–Medium — composite retrievers in all frameworks |
| KG + LLM query loop with schema validation | **Adoptable today** | Medium — text-to-graph-query quality is the risk |
| Ontology-grounded extraction and QA | **Adoptable today** (with data-governance buy-in) | High — ontology design is the real work |
| Rule-based verification over retrieved facts | **Adoptable today** | Medium — rule engine + graph |
| Concept-based retrieval (CORE/CONCORDIA-style) | Research | High — promising, not productionized |

**Banking angle.** The KG+LLM hybrid is the *best fit* of any paradigm for bank data, because banking already runs on structured models: **counterparty graphs** (ownership, control, exposure chains — the AML investigator's graph tool from §6.5), **org structures** (hierarchies, mandates, delegation), **product/fee/sanction taxonomies**, and the regulatory entity models. Regulators understand graphs (they are how supervisors model interconnectedness); a graph-grounded answer with an explicit path is the most *explainable* artifact in this guide — the natural partner to the verification loop (§3.5) for SR 11-7 narratives.

---

## 10. Paradigm 8 — Retrieval-Free / Parametric-Only Approaches

**What it is.** The quiet workhorse: **answer from the weights alone.** No retrieval, no context stuffing — the model's parametric knowledge *is* the knowledge source. This paradigm is often treated as the "before RAG" default, but in 2026 it is a deliberate, routed choice with a real economic and quality case.

### 10.1 Answering from weights — what the big models actually know

Frontier models by 2026 carry substantial world knowledge: high-frequency facts, canonical domain knowledge, common reasoning patterns, and — thanks to training on enormous corpora — a surprising amount of stable enterprise-adjacent knowledge (regulatory frameworks, financial products, standard contracts). "Big models memorize at scale" is not a failure mode; it is a *feature* of scale that this paradigm exploits deliberately. The [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) guide's framing applies: for common knowledge, the parametric answer is often as accurate as the retrieved answer — at a fraction of the latency and cost, and with zero retrieval-failure risk.

### 10.2 When parametric-only works

The four conditions:

1. **Stable knowledge** — facts that do not change (financial mathematics, standard definitions, well-established law, product mechanics). No freshness requirement.
2. **High-frequency facts** — the knowledge is in the training distribution's head: common enough that memorization is reliable.
3. **Latency-critical** — interactive surfaces where the retrieval hop + large context is unacceptable (real-time chat, embedded assistants, high-volume call-center deflection).
4. **Low-stakes grounding** — the answer does not *need* a citation trail; it is advisory, explanatory, or a first draft.

### 10.3 The limits

Equally clear:

- **Stale knowledge** — anything time-varying (sanctions, rates, ratings, personnel, regulations) is wrong by construction without retrieval.
- **Niche hallucinations** — low-frequency enterprise specifics (your bank's internal policies, this quarter's deals) are exactly where parametric models confabulate most confidently.
- **No citations** — the answer cannot point at evidence; auditors and regulators will not accept "the model knows."
- **No access control** — one set of weights serves everyone; you cannot filter what the weights know.

### 10.4 The knowledge-mechanism spectrum — choose per query

The strategic move of this paradigm is *routing across the whole spectrum*:

```
Parametric-only  ←→  RAG  ←→  Long-context  ←→  Memory
(weights)        (retrieved   (corpus in     (experience,
                  evidence)    window)        state)
```

| Mechanism | Knowledge source | Freshness | Audit | Cost/latency | Best for |
|---|---|---|---|---|---|
| Parametric-only | Weights | Frozen | None | Lowest | Common, stable, latency-critical |
| RAG | Index | Index-cycle | Citations | Low–Med | Enterprise specifics, large corpora |
| Long-context | Window | Query-time | Full context trace | Med–High | Medium corpora, deep reading |
| Memory | Experience | Continuous | Interaction log | Med | Personalization, continuity |

A router (Paradigm 9 gives it a feedback loop; §12 makes it the center of the architecture) classifies each query: *common knowledge → parametric; enterprise-specific → RAG; deep-read/aggregate → long-context; user-relative → memory.* This is the "no-retrieval-when-not-needed" policy — the inverse of classical RAG's "always retrieve."

### 10.5 Adoptable today

| Capability | Status | Effort |
|---|---|---|
| Parametric-first routing (answer from weights, retrieve on demand) | **Adoptable today** | Low — classifier or LLM router |
| Confidence-gated fallback (parametric → retrieve when uncertain) | **Adoptable today** | Low–Medium — needs calibrated uncertainty |
| Model routing for knowledge type (small fast model for common knowledge, RAG path for specifics) | **Adoptable today** | Medium — per-tier model selection |
| Learned "when to retrieve" policies (train routers on retrieval-benefit labels) | Emerging | Medium — needs benefit-labeled data |

**Banking angle.** The compliant version of parametric-only is narrow: **it is fine for non-citable, non-consequential content** (explanatory text, product education, first-draft generation) and *never* for anything decision-relevant. Banks should route aggressively to parametric for cost, but wrap it: no-citation content is labeled as such, and anything consequential is forced through retrieval + verification regardless of what the router thinks.

---

*[Continue to Paradigm 9 — Self-Improving / Learning Systems](#11-paradigm-9--self-improving--learning-systems)*

## 11. Paradigm 9 — Self-Improving / Learning Systems

**What it is.** Systems that **improve from feedback** — retrieval, generation, and corpus all become tunable by the system's own performance. This is the paradigm that closes the loop: beyond-RAG systems are not static architectures; they are *learning* architectures, and the feedback that drives them is the by-product of every production query. The data-side machinery lives in the [Closed-Loop Data Engineering](closed_loop_data_engineering_guide.md) guide; here we focus on the LLM-side loop.

### 11.1 The feedback sources

| Signal | Where it comes from | What it can tune |
|---|---|---|
| Explicit ratings | User thumbs up/down, analyst corrections | Generator, router, corpus curation |
| Implicit behavior | Accepted vs. rewritten answers, time-on-answer, follow-up questions | Retriever, reranker, router |
| Verifier scores | Groundedness/factuality checks (§3.5) | Retriever (recall), generator, corpus |
| Retrieval outcomes | Did the answer need the retrieved chunk? Was a re-retrieval required? | Retriever, chunking, embeddings |
| Golden-set deltas | Regression runs against labeled questions | Everything — the umbrella signal |
| Business outcomes | Decision quality, error flags downstream (e.g., STR rejections) | The whole pipeline's priorities |

### 11.2 RL on retrieval feedback — reward grounded answers

The training-side frontier: **reinforcement learning where the reward is groundedness** — the model is rewarded for answers that (a) are entailed by retrieved evidence and (b) are correct on the golden set, and penalized for ungrounded but fluent answers. This is the direct descendant of RLHF/RLAIF, with the reward function pointed at *grounding* instead of human preference. Two consequences:

- Generators learn to *use* evidence rather than decorate answers with it — the model internalizes "cite or don't say."
- The reward signal can come from a verifier (PRM, §3.4) — making the verification loop itself the training signal: **verified answers become training data** (this is also the engine behind RAG-then-distill, §7.4).

### 11.3 Preference tuning for retrieval

Retrievers are trainable too: **FLIP** (fine-grained preference tuning for retrieval, 2025 — deep-dived in [Advanced RAG Techniques](advanced_rag_techniques_guide.md)) trains retrievers on *preference pairs* — "for this query, document A is better than document B" — mined from user feedback and golden sets, closing the gap between embedding similarity and what users actually want. The 2026 practice: reranker/retriever preference tuning is a *routine* quarterly maintenance job, not a research project.

### 11.4 Active learning — retrieval failures become training data

The systematic version of "learn from mistakes":

1. **Detect** — verifier flags a low-groundedness answer, or a user corrects one, or a query exceeds N re-retrieval rounds.
2. **Collect** — log the query, the retrieved set, the answer, and the failure signal.
3. **Label** — route to a lightweight labeling workflow (analyst confirmation, or high-confidence auto-labeling by the verifier).
4. **Retrain** — the labeled failures become: preference pairs for the retriever/reranker (§11.3), few-shot/rubric examples for the generator, or curated pairs for distillation (§7.4).
5. **Verify** — the retrained component must pass the regression gate (§11.6) before promotion.

The economic insight: **every production failure is a free training example.** Systems that log and route failures capture a training signal that no benchmark can provide — their own users' standards.

### 11.5 Feedback-driven corpus curation

The corpus is not sacred either. Feedback tunes the *index*:

- **Deduplication** — near-duplicate chunks that split attention and votes are merged; user "wrong" flags on specific chunks mark them suspect.
- **Promotion/demotion** — chunks that consistently produce verified answers get boosted metadata; chunks that consistently produce ungrounded answers get demoted or quarantined for human review.
- **Coverage gaps** — queries with persistent retrieval failure *are* the roadmap: they identify missing documents, stale content, or missing data sources (and feed the [RAG with Data Streaming](rag_with_data_streaming_guide.md) ingestion backlog).
- **Chunking retuning** — failure clusters at chunk boundaries trigger chunk-size/split-point experiments (the [RAG Optimization Techniques](rag_optimization_techniques_guide.md) playbook, now automated).

### 11.6 Continuous evaluation — golden sets and regression gates

None of this is safe without measurement. The 2026 standard (from [RAG Optimization Techniques](rag_optimization_techniques_guide.md)):

- **Golden sets** — a labeled query set per workload (typically 200–2,000 queries with reference answers and required evidence), versioned like code.
- **Regression gates in CI** — every retriever, reranker, model, prompt, or corpus change runs against the golden set before promotion; a recall/faithfulness drop fails the build.
- **Unified metrics across paradigms** — because beyond-RAG systems route across mechanisms (§10.4), the evaluation must score *the routed outcome*, not the components: end-to-end faithfulness, citation precision, latency, and cost per resolved query (see §13.3).
- **Drift monitoring** — golden sets decay; scheduled refresh (quarterly re-labeling, new-query mining from logs) keeps the gate honest.

### 11.7 The closed loop

```
queries → router (§12) → retrieval/reasoning/memory/long-context paths
       → answers + verifier scores + user signals
       → feedback store (logged, PII-filtered)
       → improvement jobs:
            retriever/reranker preference tuning   (§11.3)
            generator RL/verifier training         (§11.2)
            RAG-then-distill refreshes             (§7.4)
            corpus curation                        (§11.5)
       → regression gate (golden set in CI)        (§11.6)
       → promote → back to production
```

The [Closed-Loop Data Engineering](closed_loop_data_engineering_guide.md) guide covers the data-pipeline mechanics (scheduling, idempotency, backfill); this loop is its LLM-side consumer.

### 11.8 Adoptable today

| Capability | Status | Effort |
|---|---|---|
| Feedback collection (signals, logging, PII filtering) | **Adoptable today** | Low–Medium — the foundation; do this first |
| Golden sets + regression gates in CI | **Adoptable today** | Medium — the discipline that makes everything else safe |
| Reranker/retriever preference tuning (FLIP-style) | **Adoptable today** | Medium — training infra exists; data is the work |
| Active-learning failure routing | Emerging in practice | Medium — needs labeled-failure workflow |
| Generator RL on groundedness rewards | Emerging | High — frontier technique, specialized teams |
| Fully automated corpus curation | Emerging | Medium — start with flag-and-review, not auto-edit |

**Banking angle.** The closed loop is also the *model-risk* loop: every retrained retriever or distilled model is a model change under SR 11-7 and needs the validation-and-approval workflow (§13.5). Design the feedback store from day one as an audit artifact — what changed, why, on what evidence, approved by whom. A self-improving system without a change log is a compliance incident waiting to happen.

---

## 12. The Synthesis — A Beyond-RAG Reference Architecture

This section assembles the nine paradigms into one reference architecture: the **2026 knowledge system**. Not every organization needs every component — the maturity matrix (§12.3) is the shopping list, and §13 is the adoption order.

### 12.1 The architecture

```
                          ┌─────────────────────────────┐
   query ────────────────▶│  ROUTER (query typing)      │
                          │  parametric / reasoning /   │
                          │  retrieval / memory / graph │  ← trained router + rules
                          └──────┬──────────────────────┘
                                 │
              ┌──────────────────┼──────────────────────┬───────────────┐
              ▼                  ▼                      ▼               ▼
   ┌──────────────────┐ ┌─────────────────┐ ┌──────────────────┐ ┌────────────┐
   │ PARAMETRIC path  │ │ REASONING path  │ │ RETRIEVAL path   │ │ MEMORY     │
   │ weights only     │ │ reasoning model │ │ (agentic, §6)    │ │ layer (§5) │
   │ (§10)            │ │ + tools:        │ │  - vector corpus │ │ episodic   │
   │                  │ │  retriever      │ │  - SQL           │ │ semantic   │
   │                  │ │  SQL  web  graph│ │  - web           │ │ preferences│
   │                  │ │  memory  code   │ │  - graph (KG §9) │ │            │
   └──────────────────┘ └────────┬────────┘ └────────┬─────────┘ └─────┬──────┘
                                 │                   │                 │
                                 ▼                   ▼                 ▼
                          ┌──────────────────────────────────────────────┐
                          │        UNIFIED CONTEXT ASSEMBLY (§8)         │
                          │  select · order · structure · compress ·     │
                          │  cache-aware · access-controlled · cited     │
                          └──────────────────────┬───────────────────────┘
                                                 ▼
                          ┌──────────────────────────────────────────────┐
                          │        GENERATION + VERIFICATION (§3.5)      │
                          │  generate → verifier (groundedness, schema,  │
                          │  entailment) → regenerate on failure →       │
                          │  structured, cited answer                     │
                          └──────────────────────┬───────────────────────┘
                                                 ▼
                          ┌──────────────────────────────────────────────┐
                          │        FEEDBACK LOOP (§11)                   │
                          │  verifier scores · user signals · golden-set │
                          │  deltas → retrain retriever/generator/corpus │
                          │  → regression gate in CI → promote           │
                          └──────────────────────────────────────────────┘
```

### 12.2 How the paths compose — the routing policy

The router is the brain. Its decision policy (rules + a trained classifier, with the trained part learning from §11 feedback):

1. **User-relative or continuity-required?** → memory path (personalization, state).
2. **Common, stable, latency-critical, non-consequential?** → parametric path (fast, cheap, labeled no-citation).
3. **Multi-hop, multi-source, investigative?** → reasoning path (agentic retrieval, budgeted test-time compute).
4. **Entity/relationship heavy, or requires exact structure?** → graph path (KG + LLM loop, neuro-symbolic).
5. **Deep reading, aggregation, medium corpus?** → long-context path (cache the corpus, read wide).
6. **Everything else — enterprise specifics, large or permissioned corpora** → retrieval path (classical/hybrid RAG as the floor).

Crucially, the paths are **not exclusive**: the reasoning path *contains* the retrieval path (retrieval as a tool), the long-context path *reads* retrieval output (retrieve-broad, read-long), the memory path *augments* any other path's context, and the graph path *feeds* the retrieval path (graph-pruned candidates). The 2026 system is a **layered capability**, not a switch.

### 12.3 Component maturity matrix

| Component | Paradigm | Status | Maturity | Banking applicability |
|---|---|---|---|---|
| Hybrid retrieval + reranking baseline | (RAG floor) | **Adoptable today** | Production-proven | Everywhere — the compliance floor |
| Reasoning model + tool use (retrieval as action) | 1, 4 | **Adoptable today** | Production (2025–2026) | Complex investigations, due diligence, deal analysis |
| Verification loop (generate → verify → regenerate) | 1 | **Adoptable today** | Production patterns | Mandatory for decision-adjacent answers |
| Long-context reader + prompt caching | 2, 6 | **Adoptable today** | Production | Contract review, regulatory reading, deep-dive QA |
| Persistent memory layer (Mem0/Zep/Letta/Cognee) | 3 | **Adoptable today** | Production (fast-moving) | Client onboarding continuity, case state, personalization |
| Agentic retrieval with guardrails | 4 | **Adoptable today** | Production (with governance) | AML investigation, research desks, ops automation |
| Domain LoRA adapters | 5 | **Adoptable today** | Production | Per-domain knowledge (credit, trade, regulatory) |
| RAG-then-distill | 5 | **Adoptable today** | Production patterns | High-volume repeated queries (call-center, FAQ) |
| Context engineering discipline (structure/cache/compress) | 6 | **Adoptable today** | Production | The cost + compliance layer for all of the above |
| GraphRAG / KG+LLM loop | 7 | **Adoptable today** | Production (entity-heavy) | Counterparty graphs, org structures, exposure chains |
| Neuro-symbolic retrieval (constraint + neural) | 7 | **Adoptable today** | Production | Structured queries over risk/financial data |
| Parametric-first routing | 8 | **Adoptable today** | Production (routing is old; learned routing is newer) | Non-consequential, latency-critical surfaces |
| Golden sets + regression gates | 9 | **Adoptable today** | Production | SR 11-7 validation backbone |
| Retriever preference tuning (FLIP-style) | 9 | Emerging | Pilot-grade | Retrieval quality on bank-specific queries |
| Budgeted test-time compute (effort routing) | 1 | Emerging | Pilot-grade | Cost governance for reasoning workloads |
| MCTS/beam search over thoughts | 1 | Emerging | Research → niche pilots | Long-horizon, high-value (complex structuring) |
| Learned sparse attention / DSA-style serving | 2 | Emerging | Frontier labs | Consume via API; not self-hosted |
| Concept-based retrieval (CORE/CONCORDIA) | 7 | Research | Paper-grade | Watch; unit-of-retrieval convergence |
| RETRO-style retrieval-trained foundation models | 5 | Research | Frontier labs | Watch; consume when available |
| Self-organizing memory (A-MEM-style) | 3 | Research | Paper-grade | Watch; evaluate after Mem0/Zep |
| Generator RL on groundedness | 9 | Research | Frontier labs | Not yet bank-ready; monitor |
| Infini-attention / infinite-memory blocks | 2 | Research | Paper-grade | MemGPT-style paging is the practical stand-in |

### 12.4 A worked example — the routed lifecycle of one query

*"What is our total exposure to the Greenline Shipping group, and has it changed since the last sanctions update?"*

1. **Router** — multi-hop, entity-heavy, freshness-required → reasoning path, graph-augmented, retrieval-backed.
2. **Reasoning path** — the agent plans: (a) graph tool: resolve Greenline Shipping group structure (subsidiaries, holdings); (b) SQL tool: aggregate positions per legal entity (read-only, scoped); (c) retrieval tool: sanctions-list delta since last update (vector over watchlists); (d) memory: prior exposure analyses of this group (case continuity).
3. **Iterative retrieval** — graph traversal reveals a 60%-owned subsidiary not in the original query; the agent re-runs SQL for it; a news tool call surfaces a fleet transfer the group announced yesterday.
4. **Context assembly** — structured context: entity table (graph), exposure table (SQL), watchlist delta (retrieval), news digest, prior-memo excerpt (memory) — ordered with the most critical evidence at the ends, cache-aware (static group profile cached), ACL-filtered (analyst's permission scope).
5. **Generation + verification** — the model synthesizes with per-claim citations to table rows and document IDs; the verifier checks each numeric claim against the SQL results and each entity claim against the graph paths; one aggregate is flagged (summed a subsidiary twice) → regenerate → pass.
6. **Feedback** — the verifier notes the duplicate-aggregation failure (a retriever/query-planning lesson); the analyst accepts with an edit (a preference signal); both flow to the improvement jobs (§11).

The same query routed through classical RAG would have returned a plausible-sounding answer with no structure, no graph, no freshness, and no verification. The gap is the entire point of this guide.

---

*[Continue to Practical Guidance — What to Adopt, and When](#13-practical-guidance--what-to-adopt-and-when)*

## 13. Practical Guidance — What to Adopt, and When

The nine paradigms are a map, not a mandate. This section is the adoption playbook: where to start, what to pilot, how to evaluate, what to avoid, and the banking-specific regulatory frame. It assumes the RAG baseline from the sibling guides is already solid — [RAG Optimization Techniques](rag_optimization_techniques_guide.md) first, then [Advanced RAG Techniques](advanced_rag_techniques_guide.md) where the baseline measurably fails.

### 13.1 Start with the adoptable-today core

The highest ROI bundle, in order:

1. **Agentic RAG with guardrails (Paradigm 4)** — the most mature beyond-RAG paradigm and the natural evolution of the agentic RAG most teams already have. Fixes the retrieval bottleneck (§2.1) with iterative retrieval and multi-source synthesis, and every framework supports it. Guardrails from day one (§6.6): loop bounds, tool allow-lists, permission enforcement at the tool, full tracing.
2. **A reasoning model on the retrieval path (Paradigm 1)** — swap the generator for a reasoning model (o3/R1/K2/Claude thinking) on the complex-query path only, with budgeted effort (§3.1). Measure the accuracy delta on the golden set before rolling out; the cost delta is real.
3. **Verification loop (Paradigm 1, §3.5)** — a separate verifier checking groundedness, with regenerate-on-failure. This is the single highest-quality-per-effort upgrade in this guide: it converts hallucination control from a prompt instruction into an enforced loop.
4. **A memory layer (Paradigm 3)** — Mem0/Zep/Letta for persistent state on continuity-heavy use cases (onboarding, case management, personalization). Design the write policy (PII filtering, consent flags) and the delete path before launch (§13.5).
5. **Context engineering discipline (Paradigm 6)** — structured, cache-aware, compressed contexts on every path. This is not a project; it is a standard.

### 13.2 Pilot next

After the core is proven on one or two use cases, pilot:

- **RAG-then-distill (Paradigm 5)** — on the high-volume repeated query set (the 5%-of-queries/60%-of-traffic head). Expected outcome: order-of-magnitude latency/cost reduction on that path, with the verified RAG pairs as the audit trail. This is the paradigm with the fastest *economic* payback and the least architectural disruption.
- **GraphRAG / KG+LLM (Paradigm 7)** — on an entity-heavy corpus (counterparty data, org structures) where relationship questions dominate. Expected outcome: multi-hop and aggregation questions that vector RAG answers poorly.
- **Long-context reading (Paradigm 2)** — on medium corpora and deep-reading workloads (contract review, regulatory Q&A), behind the access-control layer, with prompt caching to control cost.
- **Parametric-first routing (Paradigm 8)** — on non-consequential, latency-critical surfaces (deflection, explanations, first drafts), with confidence-gated fallback to retrieval.

### 13.3 Evaluate across paradigms — one yardstick

Because beyond-RAG systems route across mechanisms, per-component metrics are not enough. The unified evaluation (foundations in [RAG Optimization Techniques](rag_optimization_techniques_guide.md)):

| Metric | What it catches | Tooling |
|---|---|---|
| Faithfulness / groundedness (claims entailed by evidence) | Hallucination on every path | RAGAS-style metrics + verifier entailment |
| Answer relevance | Did we answer the question asked | RAGAS, human eval |
| Citation precision/recall | Are citations real and sufficient | Structured citation checks (§8.5) |
| Retrieval recall (per path) | The bottleneck (§2.1) — still the ceiling | Golden-set recall@k |
| End-to-end latency (p50/p95) | The cost of reasoning/agentic paths | APM + LLM gateway |
| Cost per resolved query | The economics of test-time compute | Token accounting per path |
| Regression deltas | Every change's effect | Golden sets in CI (§11.6) |

Two disciplines make this work: **one golden set per workload** (labeled queries + reference answers + required evidence, versioned) and **budget ceilings** (max tokens, max tool calls, max cost per query class — enforced, not aspirational). Every paradigm adoption must show a *net* improvement on the unified yardstick; a paradigm that improves faithfulness but triples latency for no business gain is not adopted.

### 13.4 What to avoid

- **Chasing every paradigm.** The frontier publishes weekly; your roadmap should not. Adopt what the *unified yardstick* says your workloads need, not what is newest. The most expensive mistake in 2026 is a team that has prototyped all nine paradigms and productionized none.
- **Reasoning models everywhere.** Hidden thinking tokens on every query is cost without benefit for simple questions — that is what routing (§12.2) is for.
- **Memory without a forget policy.** An ever-growing memory store is a PII liability and a retrieval-quality disaster (stale memories outrank fresh ones). Design decay/consolidation/deletion before storage.
- **Agentic retrieval without guardrails.** Unbounded loops, overly broad tools, and prompt-level permissions are how agentic systems become incidents (see the [LLM Development Risks & Security](../llm_development_risks_security_guide.md) guide's OWASP LLM Top 10 mapping — prompt injection, excessive agency, and insecure output handling are the top three relevant here).
- **Distillation without verification.** Memorizing ungrounded RAG outputs bakes hallucinations into weights, where they are *harder* to detect than in a context. Only distill verified pairs (§7.4).
- **Paradigm adoption without golden-set gates.** Every retrained retriever, new model, or new path is a regression risk; the gate is the only safety net.

### 13.5 Banking-specific: the regulatory frame

The beyond-RAG stack lands in regulated territory. The essentials, mapped to the frameworks a Crédit Agricole CIB architect will face:

**SR 11-7 (model risk management — the umbrella).** US supervisory guidance on model risk applies to *models*, and the 2025–2026 supervisory consensus extends it to AI/ML components:

- **Document paradigm choices** — the routing policy (§12.2), the choice of reasoning vs. retrieval vs. parametric per query class, and the *rationale* must be documented as part of model development. "We route investigative queries to a reasoning agent" is a model-design decision; write it down with validation evidence.
- **Validate reasoning models as model components** — a reasoning model on the decision path is a model component: it needs development documentation, independent validation, ongoing monitoring, and an approved use limit. The hidden reasoning trace complicates this — validate *outcomes* on golden sets, monitor the visible trace (tool calls, citations), and keep a human-verifiable summary. Pair reasoning with verification (§3.5) so the component's *output* is checkable even if its internals are not.
- **Explainability trade-offs** — reasoning models are less transparent than classical RAG (whose citations are natural explanations). The compliant balance: **verifiers + citations + structured traces** (what was retrieved, what was checked, what changed between drafts) as the explanation artifact. For decisions that require *reasoning* transparency (e.g., adverse-action explanations), prefer the graph path (§9.4) whose derivations are explicit.
- **Change control** — every retrained retriever, distilled model, or LoRA adapter is a model change: validation, approval, and post-change monitoring per the bank's model-risk policy (§11.8).

**MAS TRM (Technology Risk Management).** Singapore's supervisory framework (the home regulator for this repo's author) covers the operational side: system availability, data governance, and — since the 2023 AI/GenAI additions — *AI governance*: accountability for AI outcomes, human oversight of consequential decisions, and robustness against adversarial inputs. The beyond-RAG implications: agentic systems need human-in-the-loop for consequential actions (§6.6), memory systems need data-governance compliance (below), and the whole stack needs monitoring for prompt injection and tool abuse.

**Audit trails and data governance (the memory problem).** Memory systems (§5) store *interactions* — which means PII, and which means:

- **Consent** — storing user/analyst interactions requires consent capture and honoring (PDPA in Singapore, GDPR in the EU).
- **Retention** — memories must respect retention limits; forgetting (§5.6) is a compliance feature. Hard delete paths (not soft-marking) are required for erasure requests.
- **Access control** — memories are per-user/per-role; retrieval from the memory store must enforce the same ACLs as the rest of the stack.
- **Auditability** — every memory write/read/forget must be logged (who, what, when, why). The memory store is an audit artifact, not a cache.

**Guardrails for agentic systems.** The [LLM Development Risks & Security](../llm_development_risks_security_guide.md) guide is the full treatment; the non-negotiables for agentic retrieval: loop bounds, least-privilege tools enforced at the tool layer (not the prompt), read-only default for data tools, input/output filtering for injection, full tool-call tracing, and human sign-off on consequential actions. In a bank, an agent that can query transaction systems is a *control* — it must be monitored like one.

**The regulatory bottom line.** None of the nine paradigms is inherently non-compliant; all of them are non-compliant if adopted without documentation, validation, and control. The adoption order in §13.1–13.2 is also the *governance* order: each step builds the evidence base (golden sets, traces, change logs) that the next step's validation will need.

---

## 14. The Future Trajectory (2026–2028)

### 14.1 Predictions

Informed extrapolation from the 2025–2026 frontier — not guarantees, but the bets a sensible roadmap should be positioned for:

| Horizon | Prediction | Confidence | What it means for you |
|---|---|---|---|
| 2026–2027 | Context windows reach 10M+ tokens at near-linear cost; learned sparse attention becomes the default serving architecture | High | Whole-corpora-in-context for most mid-size corpora; RAG's role shifts further toward access control, freshness, and huge corpora |
| 2026–2027 | Reasoning models become the *default* generator, not a premium tier; effort is budgeted per query automatically | High | The verification loop becomes mandatory (reasoning without verification is untrustworthy); cost governance via routing |
| 2026–2027 | Memory becomes a default platform primitive (hyperscaler memory services, SDKs in every framework) | High | "Stateless" LLM apps start to look legacy; design PII/consent/retention now, before it is everywhere |
| 2027 | Retrieval-trained models (RETRO-lineage) reach frontier serving stacks — retrieval as a learned internal capability | Medium | The strongest expression of this guide's thesis; consume via API when available |
| 2027 | Verification becomes standard practice: verifiers ship in every major serving stack | Medium-High | Build golden sets and verifier rubrics now; they are the moat, not the model |
| 2027–2028 | Retrieval shrinks to enterprise-specific corpora, access-control enforcement, and freshness — its classical "everything" role recedes | Medium-High | Route aggressively (§10.4); keep the retrieval floor for what only retrieval can do |
| 2027–2028 | Neuro-symbolic renaissance in regulated industries: graph-grounded generation with rule verification as the *compliance-preferred* pattern | Medium | Banks lead this; explicit paths and derivations are what regulators trust |
| 2027–2028 | Agentic everything with governance: agents as standard workloads, with standardized guardrail/observability layers | High | Invest in agent governance (tracing, evaluation, human-in-the-loop) as platform, not per-project |

### 14.2 The strategic reading for a bank

The trajectory has one through-line: **knowledge mechanisms are converging, and the differentiator is moving from the model to the system** — routing, verification, memory governance, and feedback loops. A bank's 2026–2028 position should be:

1. **Now (2026):** strengthen the RAG baseline; add reasoning + verification on complex queries; add memory on continuity-heavy cases; adopt context engineering as a standard. (All adoptable today.)
2. **Next (2027):** pilot RAG-then-distill and graph grounding; standardize the unified evaluation and the closed feedback loop; build the agent-governance platform.
3. **Later (2028):** consume retrieval-trained models and 10M-token infrastructure as they mature; re-route the knowledge landscape per measured results.

The banks that win this transition are not the ones with the most paradigms — they are the ones with the *measurement and governance* to adopt each paradigm at the moment it earns its place.

---

## 15. Conclusion

Beyond RAG is not one technique, and it is not the death of RAG. It is the recognition — forced by three years of production experience and the 2025–2026 research frontier — that **retrieval, reasoning, memory, and generation are converging into a single learned capability**, and that the architecture that wins is the one that blends them *per query*: retrieving when knowledge is needed, reasoning when the question is hard, remembering when continuity matters, verifying before answering, and learning from every answer.

The nine paradigms form a coherent whole, not a menu:

- **Test-time compute** (1) and **agentic retrieval** (4) make the system *adaptive* — the model decides how much to think and what to fetch.
- **Long-context-native** (2) and **context engineering** (6) make the system *capacious and affordable* — the window is a database, and the database is designed.
- **Memory** (3) makes the system *continuous* — stateful, personalized, and experience-driven.
- **Retrieval-as-training** (5) and **parametric-only** (8) make the system *economical* — knowledge moves into weights where it pays, and stays out where it must be fresh and auditable.
- **Neuro-symbolic** (7) makes the system *explainable* — structure and rules where regulators demand them.
- **Self-improvement** (9) makes the system *alive* — every production answer feeds the next one.

And the synthesis (§12) is the practical form of all of it: a router, layered knowledge paths, unified context assembly, verification, and a feedback loop — with most of the value available **today** (§13), not in some distant research future.

The adoption guidance, one final time: **strengthen your RAG baseline, add reasoning + verification, add memory, add distillation, keep evaluating** — in that order, against one golden set, with budgets and guardrails, and with the documentation that makes every step regulator-ready. Classical RAG taught us how to ground models in knowledge. Beyond RAG is what we do now that we know grounding is not a pipeline — it is a capability, and it is ours to design.

---

## 16. References and Further Reading

**RAG series (this repo):**
- [Advanced RAG Techniques](advanced_rag_techniques_guide.md) — GraphRAG, agentic RAG, Self-RAG, CRAG, adaptive routing, FLIP
- [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) — the context-window battle, effective context, hybrids
- [RAG Optimization Techniques](rag_optimization_techniques_guide.md) — the retrieval baseline and evaluation playbook
- [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) — orchestration stack selection
- [RAG vs HyDE](rag_vs_hyde_guide.md) / [RAG with Data Streaming](rag_with_data_streaming_guide.md) — query-side levers and freshness

**Supporting guides (this repo):**
- [Knowledge Distillation](../knowledge_distillation_guide.md) — the training machinery behind Paradigm 5
- [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) — structured output for Paradigm 6
- [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md) / [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) — orchestration above Paradigm 4
- [Closed-Loop Data Engineering](closed_loop_data_engineering_guide.md) — the data pipeline behind Paradigm 9
- [LLM Development Risks & Security](../llm_development_risks_security_guide.md) — the guardrail layer for §13.5

**Primary sources cited in this guide:**
- Wei et al. (2022) — Chain-of-Thought Prompting; Wang et al. (2022) — Self-Consistency; Yao et al. (2023) — Tree of Thoughts
- OpenAI (2024–2025) — o1/o3 reasoning models; DeepSeek-AI (2025) — DeepSeek-R1, DeepSeek Sparse Attention (V3.2); Moonshot AI (2025–2026) — Kimi K2/K3; Anthropic — extended thinking; Qwen — QwQ
- Lightman et al. (2023) — PRM800K / process reward models; Uesato et al. — process supervision
- Packer et al. (2023) — MemGPT/Letta; Park et al. (2023) — Generative Agents; Zhong et al. (2023) — MemoryBank; A-MEM (2025, NeurIPS); Mem0 (2025); Zep/Graphiti; Cognee
- Borgeaud et al. (2022) — RETRO; Khandelwal et al. (2020) — kNN-LM; Wu et al. (2022) — Memorizing Transformer
- Microsoft Research (2024) — GraphRAG; LightRAG (2025); CORE & CONCORDIA (2025) — concept-based retrieval
- Jiang et al. (2023) — LLMLingua; Zhang et al. (2023) — H2O; SnapKV (2024); Google (2024) — Infini-attention
- Gu & Dao (2023) — Mamba; Peng et al. — RWKV; AI21 (2024) — Jamba; NVIDIA (2024) — Hymba
- FLIP (2025) — preference tuning for retrieval; RAGAS (2023) — RAG evaluation metrics
- Karpathy — "the context window is the new database" (talks, 2024–2025)

---

*End of guide. Companion material: [Advanced RAG Techniques](advanced_rag_techniques_guide.md) (the RAG-side frontier) · [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) (the context-window battle) · [RAG Optimization Techniques](rag_optimization_techniques_guide.md) (the baseline and eval playbook) · [Knowledge Distillation](../knowledge_distillation_guide.md) (Paradigm 5's machinery) · [Closed-Loop Data Engineering](closed_loop_data_engineering_guide.md) (Paradigm 9's data side) · [Hierarchical Multi-Agent Frameworks](hierarchical_multi_agent_frameworks_guide.md) · [Hybrid Multi-Agent Systems](hybrid_multi_agent_systems_guide.md) (Paradigm 4's orchestration) · [Constrained Decoding Frameworks](../constrained_decoding_frameworks_guide.md) (Paradigm 6's output discipline) · [LLM Development Risks & Security](../llm_development_risks_security_guide.md) (the guardrails for §13.5).*
