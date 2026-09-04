# Context Engineering — Designing, Managing, and Optimizing the LLM Context Window

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides — the *cross-cutting umbrella* over the RAG cluster ([rag/](rag/)) and the agent cluster ([agent_scaffolding_guide.md](agent_scaffolding_guide.md))
> **Companion Guides:** [RAG vs Long-Context LLMs](rag/rag_vs_long_context_llms_guide.md) (window-size debate) · [Advanced RAG Techniques](rag/advanced_rag_techniques_guide.md) · [RAG Optimization Techniques](rag/rag_optimization_techniques_guide.md) · [Production-Grade Agentic RAG](rag/production_grade_agentic_rag_guide.md) · [RAG with Data Streaming](rag/rag_with_data_streaming_guide.md) · [RAG Evaluation Methodology](rag/rag_evaluation_methodology_guide.md) · [RAG Evaluation Tools Comparison](rag/rag_evaluation_tools_comparison_guide.md) · [Agent Scaffolding](agent_scaffolding_guide.md) · [Fine-Tuning Frameworks Comparison](fine_tuning_frameworks_comparison_guide.md) · [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) · [LLM Compression Alternatives](llm_compression_alternatives_guide.md)
> **Last Updated:** September 2026

---

## Table of Contents

1. [The Discipline — What Context Engineering Is](#1-the-discipline--what-context-engineering-is)
2. [The Term's Rise — A 2025 Coinage?](#2-the-terms-rise--a-2025-coinage)
3. [Context Engineering vs. Prompt Engineering, RAG, and Fine-Tuning](#3-context-engineering-vs-prompt-engineering-rag-and-fine-tuning)
4. [Context-Window Economics — the Finite Resource](#4-context-window-economics--the-finite-resource)
5. [Structure and Position — Roles, Ordering, Lost in the Middle](#5-structure-and-position--roles-ordering-lost-in-the-middle)
6. [Context Budgets — Allocating the Window](#6-context-budgets--allocating-the-window)
7. [Freshness and Recency — Stale Context, Eviction, Refresh](#7-freshness-and-recency--stale-context-eviction-refresh)
8. [Context Compression — Summaries, Selection, and Learned Compression](#8-context-compression--summaries-selection-and-learned-compression)
9. [Retrieval-Fed Context Assembly](#9-retrieval-fed-context-assembly)
10. [Agentic and Long-Running Context](#10-agentic-and-long-running-context)
11. [Evaluating Context Quality](#11-evaluating-context-quality)
12. [Worked Example — A Cymbal Bank Retail-Banking Context Design](#12-worked-example--a-cymbal-bank-retail-banking-context-design)
13. [What Could Not Be Verified](#what-could-not-be-verified)
14. [Glossary](#glossary)
15. [Claims Audit](#claims-audit)

---

## 1. The Discipline — What Context Engineering Is

Prompt engineering taught the industry how to *write*. Context engineering is the discipline of deciding *what the model gets to see at all* — and in what form, in what order, and for how long. It is the deliberate design of everything that enters the context window: its **selection** (which documents, tool results, memories, and instructions), its **structure** (roles, delimiters, ordering), and its **lifecycle** (how content is added, refreshed, summarized, evicted, and rotated across a session).

### 1.1 A working definition, anchored to primary sources

The most precise vendor definition comes from Anthropic's engineering blog, in the post widely treated as the discipline's popularization point:

> **Context** refers to the set of tokens included when sampling from a large-language model (LLM). The **engineering** problem at hand is optimizing the utility of those tokens against the inherent constraints of LLMs in order to consistently achieve a desired outcome. — *Anthropic, "Effective context engineering for AI agents," September 29, 2025*

The same post defines the practice in contrast to prompt writing:

> **Context engineering** refers to the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference, including all the other information that may land there outside of the prompts. — *ibid.*

That "other information" is the expansion that makes context engineering a discipline of its own: system instructions, tool definitions, Model Context Protocol (MCP) resources, retrieved external data, message history, and intermediate scratch outputs — everything an agent loop accumulates between one inference call and the next, which must be "cyclically refined" because it is finite.

The academic literature codified the same idea in mid-2025. *A Survey of Context Engineering for Large Language Models* (Mei et al., arXiv:2507.13334, July 2025) introduces context engineering as "a formal discipline that transcends simple prompt design to encompass the systematic optimization of information payloads for LLMs," decomposed into foundational components — **context retrieval and generation, context processing, and context management** — and system-level implementations that integrate them: **RAG, memory systems with tool-integrated reasoning, and multi-agent systems**.

That taxonomy matters for this repo: this guide sits above the RAG cluster and the agent cluster not by editorial fiat but because the primary literature frames RAG and agentic memory as *instances* of context engineering. Retrieval-augmented generation is one way to construct context; memory architectures are another; both are context-construction mechanisms under the umbrella.

### 1.2 The core stance: context is a finite resource

Every school of context engineering converges on one operational stance, best stated by Anthropic's guiding principle: find "the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome." The reason is empirical, not aesthetic: models demonstrably lose precision as the window fills, an effect discussed in §5 (position bias) and §4.2 (context rot). Anthropic's engineering post is explicit that this is architectural — the transformer lets every token attend to every other token, producing n² pairwise relationships for n tokens, and longer training sequences are rarer than short ones, so models have less specialized capacity for context-wide dependencies. Context, in this view, is not a bucket to fill; it is an **attention budget** to spend.

### 1.3 What this guide covers (and where it stops)

- **The discipline and its lineage** (§1–§3) — definition, term history, and boundaries against prompt engineering, RAG, and fine-tuning.
- **The economics** (§4) — window sizes, the quadratic-attention legacy, KV-cache growth, and input-price scaling.
- **Design principles** (§5–§8) — structure and position, budgets, freshness, and compression.
- **Engineering patterns** (§9–§11) — retrieval-fed assembly, agentic/long-running context, and evaluation — each cross-referencing the sibling guide that owns the deep mechanics.
- **A worked production design** (§12) for a Cymbal Bank retail-banking assistant.

This guide does **not** re-derive retrieval mechanics (the RAG cluster owns those), agent-loop mechanics (the agent cluster owns those), or the RAG-vs-long-context strategic debate (one cross-referenced paragraph in §4.5). It is the umbrella: the design logic that decides what context those systems assemble in the first place.

> **Honesty note.** Where a claim is verified against a primary source (paper, vendor engineering post, official model documentation), it is stated as fact and itemized in the [Claims Audit](#claims-audit). Where the record is fuzzy — most notably the *coinage* of the term itself — the guide flags it with ⚠ and says so in [What Could Not Be Verified](#what-could-not-be-verified). Specs that vendors change quarterly are dated at retrieval time and marked accordingly.

### 1.4 Who does context engineering? — a role map

Because the discipline is young, its ownership is usually scattered across roles that predate it. The pragmatic mapping used in production teams:

| Existing role | Context-engineering responsibilities they already carry |
|---|---|
| Prompt/instruction owner | System-prompt design, output contracts, few-shot curation — the *static* layer (§3.1) |
| RAG engineer | Retrieval, chunking, reranking — the *evidence-construction* layer (see the [RAG cluster](rag/)) |
| Agent engineer | Tool design, loop state, memory, compaction — the *lifecycle* layer (see [Agent Scaffolding](agent_scaffolding_guide.md)) |
| LLMOps/platform engineer | Cache geometry, token budgets, fill-rate and cost monitoring, model swaps (§6, §9.1) |
| Evaluator | Context sufficiency/precision/groundedness suites (§11) |
| Compliance/risk | Eviction logs, provenance, citation enforcement, audit manifests (§12.4) |

The context-engineering mindset is what connects these six seats into one design conversation: they are all deciding, at different layers and timescales, *which tokens deserve the window*. The worked example in §12 shows the artifact that makes that conversation concrete — a single budget and lifecycle table owned by one architect.

### 1.5 The artifacts of the discipline

A discipline is only as real as its deliverables. Teams that practice context engineering produce and maintain a small set of artifacts, each owned, versioned, and wired into the pipeline that enforces it:

| Artifact | Contents | Owner | Governs |
|---|---|---|---|
| **Context budget** | Token allocation table per component, with shares, positions, and headroom (§6) | Architect | Assembly config; alert thresholds on overruns |
| **Ordering spec** | Position rules: premium slots, evidence ordering, prefix/suffix split (§5, §9.1) | Architect + RAG engineer | Serialization code and prompt layout |
| **Lifecycle / refresh policy** | Per-content-class TTL, eviction order, compaction triggers, cache invalidation (§7, §12.4) | LLMOps + session manager | Cache TTLs, compaction cron, invalidation hooks |
| **Provenance & eviction manifest** | Per-request record of what was included, sourced, and excluded — and why (§12.4) | Compliance/risk | Audit store, regulator responses |
| **Quality gate suite** | Golden-set sufficiency/precision/groundedness runs bound to CI (§11) | Evaluator | Merge and rollout of any context change |
| **Model-swap checklist** | Re-verification steps for effective window, position sensitivity, cache geometry (§12.6) | LLMOps | Every vendor model upgrade |

The tell-tale sign that a team has crossed from prompt tinkering into context engineering is not vocabulary — it is that these artifacts exist, have owners, and fail loudly in review. Where they are missing, the six roles in §1.4 are still doing context engineering; they are just doing it implicitly, inconsistently, and without a paper trail.

---

## 2. The Term's Rise — A 2025 Coinage?

### 2.1 What the primary sources say

The term "context engineering" entered the applied-LLM mainstream in **2025**. The two anchor points verified for this guide:

1. **arXiv:2507.13334** (July 17, 2025) — *A Survey of Context Engineering for Large Language Models* (Mei et al., 15 authors, 166 pages, ~1,400 citations surveyed). This is the earliest *formal/academic* treatment this research pass could verify, and it already treats the term as established enough to survey ("This survey introduces Context Engineering, a formal discipline…").
2. **Anthropic, "Effective context engineering for AI agents"** (September 29, 2025) — the vendor post that popularized the framing for builders: "After a few years of prompt engineering being the focus of attention in applied AI, a new term has come to prominence: **context engineering**," and "At Anthropic, we view context engineering as the natural progression of prompt engineering." Anthropic's Applied AI team (Rajasekaran, Dixon, Ryan, Hadfield) is credited as authors.

Anthropic continued the series into 2026: a follow-up engineering post, *The new rules of context engineering for Claude 5 generation models*, reports that Anthropic "removed over 80% of Claude Code's system prompt for more advanced models" — an existence-confirmed post whose headline claim is repeated here as reported, not independently re-measured (⚠). Anthropic also published a developer cookbook titled *Context engineering: memory, compaction, and tool clearing* that compares lifecycle strategies for long-running agents, and shipped a file-based **memory tool** (announced alongside Sonnet 4.5) that stores and consults information outside the window.

### 2.2 Who coined it? — ⚠ contested and ultimately unverifiable

The honest answer: **the earliest usage cannot be pinned down from primary sources.** Three things can be said with confidence:

- The term has a long, unrelated prehistory in other engineering fields (e.g., human-computer interaction and knowledge engineering used "context engineering" decades before LLMs), so any claim of a single LLM-era "first use" is inherently fuzzy.
- Within the LLM era, mid-2025 is the earliest verifiable cluster: the July 2025 survey and contemporaneous practitioner writing. Anthropic's own post treats the term as already "come to prominence" by September 2025 rather than claiming to coin it.
- Practitioner lore attributes early popularization to Andrej Karpathy — Anthropic's post links to a Karpathy post characterizing context engineering as "the art and science of curating what will go into the limited context window" — and some secondary write-ups date the popularization to June 2025 via Karpathy and Shopify's Tobi Lütke. Neither the exact date nor the coinage priority of that thread could be verified at a primary source for this guide (⚠).

**Bottom line for the series:** treat the term as a *2025-era* consolidation of practices that predate the name — context design has been implicit in RAG pipelines since 2020 and in agent design since 2023 (see the [RAG evolution timeline](rag/rag_evolution_timeline_guide.md)). What is genuinely new is the *unifying frame*: one discipline that owns retrieval, memory, compression, and prompt structure as sub-problems of a single budget problem.

### 2.3 Distinguish the term from its neighbors

Three near-synonyms cause confusion in vendor literature and should be kept straight:

- **Context management** (as in Anthropic's *memory and context management* cookbook and the memory-tool announcement) is the *operational* subset of context engineering: the mechanics of storing, summarizing, clearing, and restoring context across a session. Context engineering is the wider discipline that decides *what* those mechanics should do — management is the how, engineering is the why-and-what.
- **Context windows / long context** describe a *model property* (how many tokens the architecture accepts), not a practice. Vendors market windows; engineers engineer context. Conflating the two is how "1M tokens will fix it" arguments happen (§4.5).
- **Contextual retrieval** (a specific 2024-era technique for making chunks self-contained before embedding) is one retrieval-side trick inside the assembly layer, not the discipline. The repo's [RAG Optimization Techniques guide](rag/rag_optimization_techniques_guide.md) covers it where it belongs.

---

## 3. Context Engineering vs. Prompt Engineering, RAG, and Fine-Tuning

The fastest way to internalize a new discipline is to draw its borders against the three practices it overlaps.

### 3.1 Prompt engineering: the static instruction — context engineering: the whole window

Prompt engineering optimizes the *instruction text*: wording, structure, few-shot examples, output formatting. Context engineering includes that activity and then supersedes it, because in any modern system the instruction is a minority of what the model reads. Anthropic's boundary statement:

> Prompt engineering refers to methods for writing and organizing LLM instructions for optimal outcomes… Context engineering refers to the set of strategies for curating and maintaining the optimal set of tokens during LLM inference, including all the other information that may land there outside of the prompts. — *Anthropic, September 2025*

The practical boundary for an architect: **a prompt is static text you author; context is the dynamic state you assemble per request** — retrieved documents, tool outputs, conversation history, user identity and entitlements, cached corpus prefixes, and scratchpad content. Prompt engineering asks "what should the instructions say?" Context engineering asks the harder questions: "which 20,000 of the available 20 million tokens should accompany those instructions, in what order, in what structure, and what happens to them after this turn?" A system prompt is a context-engineering artifact, but it is one artifact among many.

### 3.2 RAG: retrieval is a context-construction mechanism

The survey's taxonomy is unambiguous: RAG is classified as a *system implementation* of context engineering — one architecture for assembling the information payload. This reframing is liberating: **RAG is not the goal; it is one mechanism for building context.** Its siblings under the umbrella are memory systems, tool-integrated "just-in-time" retrieval, and long-context stuffing, all of which solve the same sub-problem — deciding which information deserves the window — through different machinery:

- **Pre-inference retrieval** (classic RAG): embeddings select chunks before generation; context is constructed in advance.
- **Just-in-time retrieval**: agents hold lightweight references (file paths, queries, URLs) and pull content into the window at runtime with tools — the pattern Anthropic describes as the shift "from pre-processing all relevant data up front" toward dynamic loading, mirroring human use of file systems and bookmarks rather than memorized corpora.
- **Hybrid**: some context is pre-loaded for speed (e.g., a standing corpus prefix), the rest explored on demand.

The retrieval mechanics themselves — chunking, hybrid search, reranking, indexing — live in the sibling guides: [Advanced RAG Techniques](rag/advanced_rag_techniques_guide.md), [RAG Optimization Techniques](rag/rag_optimization_techniques_guide.md), and [Production-Grade Agentic RAG](rag/production_grade_agentic_rag_guide.md). This guide's contribution is the *assembly logic*: given a retriever's output, how should that output be structured, budgeted, ordered, and refreshed inside the window (§5–§9).

### 3.3 Fine-tuning: what you do when context design is insufficient

Context engineering and fine-tuning modify different storage: **context engineering modifies the non-parametric state presented at inference; fine-tuning modifies the parametric weights.** The decision rule used across this repo's series (derived in depth in the [Fine-Tuning Frameworks Comparison guide](fine_tuning_frameworks_comparison_guide.md), which this section deliberately does not duplicate):

- **Context first.** If the knowledge or behavior can be supplied per-request — policies, product data, recent events, user-specific facts, formatting rules — context engineering is cheaper, faster to change, auditable, and reversible. It is the default move for any *information* problem.
- **Fine-tune only for what context cannot carry.** Persistent behavior that must fire without being told (tone, domain vocabulary, tool-use style), skills that must generalize beyond any retrievable corpus, latency/cost constraints that make shipping the context unaffordable, or knowledge too large or too sensitive to stuff or retrieve. Fine-tuning changes what the model *is*; context engineering changes what it *sees*. When the desired behavior survives neither better prompts nor better context, weights are the remaining lever — and the fine-tuning guide's frameworks comparison (LoRA/QLoRA, full FT, distillation) is the reference for executing it.

One nuance worth keeping: the two interact. A fine-tuned model still needs engineered context to stay grounded and current; an engineered context cannot teach the model a capability it lacks in weights. Production systems in this repo's banking persona (§12) treat them as a stack — weights for behavior, context for knowledge — never as substitutes.

### 3.4 The four levers on one axis

| Lever | Object of change | Best when | Costs and limits |
|---|---|---|---|
| Prompt engineering | The instruction text | Behavior is mostly specified by what you say | Zero marginal cost, but cannot supply knowledge or adapt to state |
| Context engineering | Everything fed at inference (structure, selection, lifecycle) | Knowledge, state, or grounding must vary per request or over time | Per-token cost and latency; degradation at length (§4.2); needs lifecycle machinery |
| Retrieval augmentation | Where the context comes from | Corpus too large/live/permissioned for the window | Retrieval recall is the ceiling; adds pipeline latency |
| Fine-tuning | The model weights | Persistent behavior/skills that must fire untold, or context too costly to ship | Expensive, slow to change, needs eval governance |

The decision heuristic used throughout this series: **prompt first, then context, then retrieval (as a context source), then weights** — escalating only when the cheaper lever provably fails on your golden set. Context engineering is the middle of that escalation and the connective tissue of the other three.

---

## 4. Context-Window Economics — the Finite Resource

### 4.1 The window-size table at retrieval time (September 2026)

Context-window specs are the fastest-moving numbers in this guide. The table below reflects official documentation retrieved for this edition; treat any cell as a point-in-time reading and re-verify quarterly.

| Vendor family | Model(s) | Marketed context window | Max output | Verified source (retrieved Sept 2026) |
|---|---|---|---|---|
| Anthropic Claude | Opus 5, Sonnet 5, Fable 5.1 | 1M tokens (input) | 128K | platform.claude.com models overview |
| Anthropic Claude | Haiku 4.5 | 200K tokens | 64K | platform.claude.com models overview |
| OpenAI | GPT-4.1 (Apr 2025) | 1M tokens | 32K | openai.com GPT-4.1 announcement |
| OpenAI | GPT-5 (Aug 2025) | 272K class ⚠ | — | third-party trackers; OpenAI API docs |
| OpenAI | GPT-5.4 / GPT-5.6 | 1.05M tokens | 128K | developers.openai.com API docs |
| Google Gemini | Gemini 2.5 Pro | 1,048,576 input / 65,536 output | 65,536 | ai.google.dev model page (updated Jun 2026) |
| Google Gemini | Gemini 1.5 Pro | up to 1M (public), millions in research | 8K | arXiv:2403.05530 technical report |
| Meta Llama | Llama 4 Scout (17B×16E) | 10M tokens | — | official MODEL_CARD (github.com/meta-llama/llama-models) |
| Meta Llama | Llama 4 Maverick (17B×128E) | 1M tokens | — | official MODEL_CARD |
| Meta Llama | Llama 3.1 405B | up to 128K | — | arXiv:2407.21783 (Llama 3 Herd) |
| Mistral | Mistral 7B (open) | 8K trained, SWA for arbitrary length | — | arXiv:2310.06825 |

Notes and caveats on the table:

- **Marketed ≠ effective.** The single most replicated finding in long-context research is that usable length trails advertised length. RULER (Hsieh et al., arXiv:2404.06654, COLM 2024) evaluated 17 long-context models claiming 32K+ windows and found "almost all models exhibit large performance drops as the context length increases," with only about half "maintain[ing] satisfactory performance at the length of 32K." Budget your context at a fraction of the marketed window until you measure your own workload.
- **Llama 4's 10M number is a research-scale claim in a shipping card.** Scout's model card lists a 10M context length; real deployments typically operate far below it, and Meta's own evaluation guidance in the card is narrower than the headline (⚠ on any production use near the limit).
- **Fast churn.** Between mid-2025 and mid-2026, Anthropic's flagship input window went from 200K-class (Sonnet 4.5) to 1M (Sonnet 5 / Opus 5); OpenAI's went from 400K-class GPT-5 to 1.05M GPT-5.4/5.6. Architect for window size as a *parameter*, not a constant — the context budget design in §12 is written to survive a model swap.

### 4.2 The degradation curve: context rot, dilution, and position bias

Three well-documented phenomena bound the practically usable window:

- **Context rot** (the umbrella term): Chroma's study *Context Rot: How Increasing Input Tokens Impacts LLM Performance* (Hong, Troynikov, Huber; July 2025) evaluated **18 LLMs** — closed and open — on semantic needle-in-a-haystack variants, LongMemEval conversational QA, and synthetic repeated-word tasks, holding task complexity constant and varying only input length. Result: performance degrades as input grows, "often in surprising and non-uniform ways," across essentially all models tested. Anthropic's context-engineering post cites exactly this line of work: "as the number of tokens in the context window increases, the model's ability to accurately recall information from that context decreases… this characteristic emerges across all models."
- **Dilution/lost-in-the-middle**: positional degradation in long contexts — the subject of §5.3 (Liu et al., arXiv:2307.03172).
- **Length-vs-ability gradient**: models remain capable at long context but with "reduced precision for information retrieval and long-range reasoning compared to… shorter contexts" (Anthropic). Treat window size as a capacity curve, not a cliff — and not a uniform plane.

### 4.3 The compute legacy: O(n²) attention and its mitigations

The reason context is expensive is architectural. Self-attention's time and memory complexity are **quadratic in sequence length** — the original *Attention Is All You Need* architecture (Vaswani et al., arXiv:1706.03762) made every token attend to every token, which FlashAttention's authors restate bluntly: "Transformers are slow and memory-hungry on long sequences, since the time and memory complexity of self-attention are quadratic in sequence length." Doubling the context roughly quadruples the attention work and the memory for attention matrices — the economic engine behind every RAG-vs-long-context cost comparison.

The mitigation stack, roughly in deployment order:

| Mitigation | What it does | Verified anchor |
|---|---|---|
| **FlashAttention / FlashAttention-2** | IO-aware tiling of exact attention; reduces HBM traffic, memory linear instead of quadratic; 2–4× speedup vs. optimized baselines (FA), ~2× more (FA-2); no approximation | arXiv:2205.14135; arXiv:2307.08691 |
| **Grouped-query attention (GQA)** | Shares KV heads across query heads, cutting KV-cache size and inference cost | arXiv:2310.06825 (Mistral 7B) |
| **Sliding-window attention (SWA)** | Each token attends only to a local window — "handle sequences of arbitrary length with a reduced inference cost" at some cost in long-range access | arXiv:2310.06825 |
| **Sparse/linear-attention research** | Longformer/BigBird-class sparsity and linear-time formulations trade exactness for O(n) scaling — surveyed broadly in the long-context literature; ⚠ no single canonical deployment winner as of this writing | ⚠ see survey literature |
| **Distributed attention (sequence/ring parallelism)** | Shards the KV state and attention computation across devices so context can exceed single-device memory | ⚠ qualitative; no primary source re-verified in this pass |

The upshot for practitioners is not to master each kernel but to internalize the budget math: **KV-cache memory grows with sequence length and batch**, which is why long-context serving is memory-bound and why providers price long inputs the way they do (§4.4). FlashAttention-class kernels made 1M-token inference *possible*; they did not make it *cheap*.

The arithmetic is worth doing once, by hand, because it explains every pricing page in §4.4. For a decoder-only transformer with grouped-query attention, the KV cache stores two tensors (keys and values) per layer, per KV head, per token:

> **KV bytes per token ≈ 2 × (number of layers) × (KV heads per layer) × (head dimension) × (bytes per element)**

A concrete worked example from a verified architecture: the Llama 3.1 405B dense model (arXiv:2407.21783) uses **126 layers, 8 KV heads, and a head dimension of 128**. At FP16 (2 bytes per element), that is 2 × 126 × 8 × 128 × 2 = **516,096 bytes ≈ 0.5 MB of KV state per token** — per sequence, per batch slot. Scale it:

| Sequence length | KV-cache bytes (one sequence, FP16) | Note |
|---|---|---|
| 8K | ~4 GB | Fits comfortably on one accelerator |
| 128K | ~66 GB | Exceeds the HBM of a single device — this is why 128K-class serving needs multi-device sharding or aggressive batching trade-offs |
| 1M | ~0.5 TB | Only feasible sharded across many devices, or with cache-reuse tricks such as prefix caching across requests |

Two consequences follow directly. First, **batch is a multiplier**: serving B concurrent 128K sequences costs ~B × 66 GB of KV state — the reason providers cap long-context throughput and bill it at a premium. Second, **the cache is why prefixes are cheap**: because KV state for a shared prefix can be *reused* across requests (prompt caching, §9.4), a provider can serve a cached-prefix request at a fraction of the prefill work — which is exactly what the 0.1× cache-hit price in §4.4 reflects. The numbers above are generic architecture arithmetic; actual per-model constants differ, but the shape of the curve does not.

### 4.4 Price-per-token input scaling (verified at Anthropic + OpenAI, Sept 2026)

Input tokens are not billed like output tokens, and long contexts are billed like a scarce resource:

- **Anthropic** (platform.claude.com pricing): Claude Opus 5 at **$5 per MTok input** / $25 output; Sonnet 5 at $2/$10; Haiku 4.5 at $1/$5; Fable 5.1 at $10/$50. **Prompt caching** is the lever that rescues repeated long prefixes: cache *hits* are billed at **0.1× the base input price** (0.025× for Fable 5.1-class models), while cache *writes* cost 1.25×–2× base depending on retention (5-minute vs 1-hour). A 200K-token static prefix that costs $1.00 fresh costs ~$0.10 on a cache hit — caching is a context-engineering feature as much as a billing one (§9.4).
- **OpenAI** (API docs): GPT-5.4-class models with a 1.05M window impose a **2× input / 1.5× output surcharge for the whole session once a prompt exceeds 272K input tokens** — a pricing cliff explicitly designed to price long-context sessions. GPT-4.1's 1M window was itself the 2025 proof point that "paste everything" is billable even when it works.
- **Rough arithmetic**: one 1M-token request at Opus 5 pricing is **$5.00 of input alone**; the same content delivered as 10 retrieved 2K-token chunks (~20K tokens, ~90% cache hit) costs on the order of $0.02–0.10 and answers in a fraction of the prefill time. That ratio — 50–250× — is the permanent economic argument for context curation over context stuffing, independent of the accuracy arguments.

### 4.5 One cross-reference: the RAG-vs-long-context debate

The strategic question this economics raises — "if windows are 1M+, why retrieve at all?" — is owned by the sibling [RAG vs Long-Context LLMs guide](rag/rag_vs_long_context_llms_guide.md), which maps the accuracy, cost, latency, and compliance evidence in depth. Its conclusion is adopted here without re-derivation: window size is a *capacity*, not a capability; retrieval and long context are complementary context-construction mechanisms (retrieval to narrow, long context to read broadly); and the winning architectures route per-query between them. Everything below assumes that conclusion and focuses on the design layer both mechanisms share: how to build the context itself.

### 4.6 Two worked arithmetic sketches (illustrative, from §4.4 prices)

All prices below are computed from the verified September 2026 list prices; the shapes matter more than the cents.

**Sketch 1 — stuffing vs. curated reading, per request.** A 800K-token "paste the corpus" request on Claude Opus 5 costs **$4.00 of input** ($5/MTok); on Sonnet 5, **$1.60** ($2/MTok). The same workload as a cache-shaped session — 750K stable prefix + 50K fresh tail — costs $0.375 (Opus, prefix at 0.1× cache-hit) + $0.25 (fresh tail) = **$0.625 on Opus**, and $0.15 + $0.10 = **$0.25 on Sonnet**. Caching alone turns a ~6.4× input-cost difference per repeated request; the RAG alternative (25K tokens/request) is another 10× below that, at $0.05 on Sonnet — which is precisely the 50–250× spread cited in §4.4 and explored end-to-end in the sibling cost sections of the [RAG vs Long-Context LLMs guide](rag/rag_vs_long_context_llms_guide.md).

**Sketch 2 — the over-272K cliff.** On OpenAI's GPT-5.4-class models (1.05M window), a 300K-token request crosses the 272K threshold and prices the *entire session* at 2× input. A 300K request that would be ~$X at base rates becomes ~2×$X, and every subsequent turn in that session inherits the multiplier. The rational response is context engineering, not heroics: keep any single request under the cliff by routing, retrieval, or compaction — or budget the 2× deliberately for the rare deep-read request. (Base dollar figures for GPT-5.4 are not reproduced here because the current per-MTok list price was not re-verified this pass; the multiplier is the verified fact.)

### 4.7 A verified milestone timeline (why the economics changed so fast)

| When | Milestone | Source |
|---|---|---|
| Early 2024 | Gemini 1.5 Pro introduces a 1M-token context window; near-perfect recall reported on long-context retrieval | Gemini 1.5 report (arXiv:2403.05530); Chroma context-rot report |
| Apr 2025 | OpenAI GPT-4.1 ships a 1M context window; long-context coding/agent workloads go mainstream in the API | openai.com GPT-4.1 announcement |
| Apr 5, 2025 | Llama 4 Scout (10M context) and Maverick (1M) released as open weights | Official Llama 4 model card |
| Jul 2025 | Context-rot study: 18 LLMs degrade non-uniformly with input length | Chroma research report |
| Sep 29, 2025 | Anthropic publishes "Effective context engineering for AI agents" — the discipline's popularization anchor | Anthropic engineering blog |
| 2026 | Claude Opus 5 / Sonnet 5 / Fable 5.1 standardize on 1M windows (128K output); GPT-5.4/5.6 reach 1.05M | platform.claude.com docs; OpenAI API docs |

The pattern the timeline exposes: *windows outgrew reliable use faster than research could map the gap*. Every jump on this table was followed by a paper or report showing that effective context trails marketed context (§4.2) — which is exactly why the design discipline in the second half of this guide matters more, not less, as windows grow.

---

## 5. Structure and Position — Roles, Ordering, Lost in the Middle

Context is not a bag of tokens; it is a *sequence*, and sequence position is a first-class design variable. Three verified findings make position matter:

1. **Models use the ends of the window better than the middle.** Liu et al., *Lost in the Middle: How Language Models Use Long Contexts* (arXiv:2307.03172, TACL 2023), tested multi-document question answering and key-value retrieval across long contexts and found performance "often highest when relevant information occurs at the beginning or end of the input context, and significantly degrades when models must access relevant information in the middle of long contexts, even for explicitly long-context models."
2. **The effect survives in follow-up benchmarks.** RULER (arXiv:2404.06654, COLM 2024) extended needle-in-a-haystack (NIAH) testing with multi-needle, multi-hop, and aggregation tasks across 17 models and confirmed large, near-universal performance drops as context grows — i.e., positional failure is not an artifact of one benchmark. Chroma's 18-model context-rot study (July 2025) reached the same conclusion on semantic-retrieval and conversational tasks.
3. **The gradient is architectural, not incidental.** Full attention spreads a finite "attention budget" over n² pairwise relationships (Anthropic, 2025); anything placed mid-context competes with more neighbors than anything placed at the extremes.

### 5.1 Design consequence: put your most load-bearing content in premium positions

The verified "U-shaped" (or "U-ish") attention profile translates into an ordering discipline. Position value is roughly: **very start > very end > everything else**, with the middle reserved for content the model must *hold* rather than *act on precisely* — and even then, budgeted. Anthropic's own guidance is to keep the system prompt minimal and sectioned (XML tags or Markdown headers such as `<background_information>`, `<instructions>`, `## Tool guidance`, `## Output description`) rather than long and flat, and the repo's long-context sibling ([RAG vs Long-Context LLMs](rag/rag_vs_long_context_llms_guide.md), §15.1) distills the same evidence into operational rules: relevant documents first, instructions short and late, structural delimiters throughout.

### 5.2 Role salience: system, user, tool

Modern APIs give context three structural homes — system, user/turn history, and tool messages — and engineers should assume they are *not* attended to identically:

- **System prompt.** Highest-leverage position for durable instructions and identity. Because it is constant across turns it is also the natural cache prefix (§9.4). Verified guidance: keep it at the "right altitude" — specific enough to shape behavior, general enough not to hardcode brittle logic; the failure modes Anthropic names are over-engineering (prompts that hardcode agentic control flow) and under-specification (vague guidance that "falsely assumes shared context").
- **User turns / conversation history.** The model's ground truth for *what the user wants now*; recency within the history matters (see §7). Historical turns are the primary target for compaction (§8.4, §10.3).
- **Tool messages.** Machine-generated observations. They are the most voluminous and least durable content class — results are dead once consumed. Anthropic's engineering guidance names **tool-result clearing** as the "safest lightest touch form of compaction": once a tool call deep in history has been acted on, the raw result rarely needs to be re-read.

Heuristic (not a cited research claim — treat as engineering guidance): *instructions at the top, evidence in the middle ordered by relevance, the current question and output contract at the bottom, tool noise pruned aggressively.* Role tags are cheap structure that survives serialization; sibling guides in this repo use the same `<document id=… source=…>` conventions ([RAG Optimization Techniques](rag/rag_optimization_techniques_guide.md)).

### 5.3 Ordering retrieved content

For retrieval-fed contexts the ordering rule follows directly from §5.1: **order documents by expected relevance, best first** — and be aware that "best first" and "most recent first" conflict; resolve by task. When a document set is too big for premium positions, only the top documents get full text while the rest are represented by summaries (a map-then-territory pattern whose mechanics live in the [Advanced RAG Techniques guide](rag/advanced_rag_techniques_guide.md) and the [RAG vs Long-Context LLMs guide](rag/rag_vs_long_context_llms_guide.md)).

### 5.4 Test position bias on your own workload

Vendor benchmarks cannot tell you where *your* facts sit in *your* context. The standard protocol, assembled from the verified methodologies of §5 (Liu et al.'s position sweeps, RULER's task families, Chroma's semantic-needle variant):

1. Take 50–200 golden questions that require a specific fact from a specific document.
2. For each, build the context three ways — target fact placed early (~5% depth), middle (~50%), late (~95%) — holding everything else constant.
3. Measure answer accuracy per position, plus a distractor-heavy variant (relevance of other documents shuffled) to expose dilution.
4. Plot accuracy vs. position; if your curve is flat, positional ordering matters less for your workload; if it is U-shaped (the common case per the verified literature), enforce the §5.3 ordering rule and add a test to CI that guards the position of the highest-ranked chunk.
5. Re-run on every model upgrade and every major prompt reorder — position sensitivity is model-specific and changes between generations.

### 5.5 Follow-ups: the effect is real but benchmark-shaped

Two verified extensions matter when designing against lost-in-the-middle:

- **RULER showed the effect hides behind easy benchmarks.** Models scored near-perfect on the vanilla single-needle test while degrading sharply on RULER's harder task families (multiple needles, multi-hop tracing, aggregation) as context grew — a warning that *your* eval must include your hardest retrieval shape, not the flattering one.
- **Chroma showed the same hiding in plain sight with semantics.** Its semantic-needle variants and distractor manipulations degraded where lexical NIAH looked clean, and degradation was "non-uniform" — some models lost precision on position tasks, others on word-count/aggregation tasks. "Context rot" is not one curve; it is a family of curves that differ per model and per task.

Design consequence: ordering rules (§5.3) and effective-window budgeting (§6.2) are *necessary* but not *sufficient* — the residual risk is task-specific, which is why §5.4's own-workload test and §11's evaluation gates are part of the discipline rather than optional hygiene.

---

## 6. Context Budgets — Allocating the Window

### 6.1 The budget as a first-class artifact

Because context is finite and positional, production teams should write an explicit **context budget** — a token allocation table, owned by an architect, reviewed when models or features change — before they write prompts. A budget forces the hard trade-offs into the open: how much window does the *identity and policy* deserve versus the *evidence* versus the *conversation*? The practice is directly implied by Anthropic's principle of "the smallest possible set of high-signal tokens" and by the degradation evidence of §4.2: every token in the window is a marginal drain on every other token's attention.

### 6.2 A generic allocation template

| Component | Typical share | Why this share | Lifecycle |
|---|---|---|---|
| System prompt (identity, policy, output contract) | 2–8% | Highest salience per token; keep minimal and sectioned (§5.2) | Static; cache prefix; change rarely |
| Standing corpus / policy prefix | 0–40% | Reused evidence; cheap only with prompt caching | Static between refreshes; cache-aware ordering |
| Per-query retrieved evidence | 20–50% | The answer's raw material; size = f(retriever precision) | Per-turn; re-retrieved, never trusted from memory |
| Conversation history (compacted) | 15–35% | Task continuity; the first thing to compress when tight | Rolled up by compaction (§8.4) |
| Scratchpad / intermediate state | 0–10% | Agent working notes; keep outside window when possible (§10) | Ephemeral; tool/note-backed |
| Current user request + output format | 2–5% | Must always fit in a premium position | Per-turn |

Percentages are starting points, not laws — the discipline is *having* the budget and measuring against it. Two operating rules make budgets work:

- **Budget against *effective* context, not marketed context.** Given §4.2, size the working window at 50–80% of the claimed window (and less on degraded workloads), leaving headroom for output tokens and for the model's own reasoning.
- **Make overruns loud.** Log per-request token counts by component; alert when retrieved evidence silently grows past its allocation — the classic failure mode is a retriever whose top-k drifts upward as the corpus changes, starving history and instructions (this repo's [RAG Evaluation Methodology guide](rag/rag_evaluation_methodology_guide.md) and [RAG Optimization Techniques guide](rag/rag_optimization_techniques_guide.md) cover detecting that drift).

### 6.3 Priority ordering under pressure

When a request cannot fit the budget, the eviction order should be pre-agreed, roughly: **scratchpad → verbose tool results → older history → lowest-ranked retrieved chunks → (only if desperate) system-prompt trimming.** Never evict from the current request, the output contract, or the top-ranked evidence. This ordering mirrors what compaction systems actually do (§8.4, §10.3) and gives the budget teeth: allocation is only meaningful if de-allocation is also specified.

### 6.4 The budget review checklist

A quarterly (or model-swap-triggered) review should answer, in order:

1. Has any component exceeded its allocation in the last 90 days, and was the alert loud (§6.2)?
2. Is the *effective*-window assumption (§4.2) still right for the current model generation?
3. What is the measured cache-hit rate on the stable prefix, and did any prompt reorder break cache geometry (§9.1)?
4. Are scratchpad/notes still outside the window (§10.2), or did they silently migrate in?
5. Did the retriever's top-k or chunk sizes drift, quietly reallocating the evidence share (§11, lifecycle health)?
6. Does the eviction order (§6.3) still match the newest failure postmortems?
7. Is the budget document itself current — owned, reviewed, and wired to the config that enforces it?

If the answer to 7 is "no," the other six questions are moot: an unwritten budget is just an undocumented overflow.

### 6.5 Budget anti-patterns

Experience across this repo's deployments keeps surfacing the same six mistakes. Each is a design smell with a known fix:

| Anti-pattern | Symptom | Fix |
|---|---|---|
| **The 100%-allocated budget** | Every component's share is fixed and the headroom row is empty; any overrun evicts something load-bearing | Reserve explicit headroom (the §12.2 design keeps 30%) and treat it as an allocation, not slack |
| **Allocation by model name** | Shares written as "50K on Claude, 80K on GPT" instead of percentages of an *effective* window | Express shares as % of effective context (§6.2); convert per model at swap time (§12.6) |
| **The polite overflow** | Overruns are absorbed silently because no alert exists; the retriever's top-k drifts and history starves over weeks | Per-component counters with loud thresholds (§6.2) and lifecycle-health monitoring (§11.1) |
| **Budget in the doc, not the config** | The table lives in a design review; the assembly code hardcodes its own numbers | Wire the budget document to the config that enforces it, and diff them in review (the §1.5 artifact list) |
| **Eviction by mood** | No pre-agreed eviction order; whatever the last developer to touch the code thought was expendable gets dropped | Adopt and freeze the §6.3 ordering; change it only in review, with the failure postmortem in hand |
| **Budgeting the input, forgetting the output** | The window is fully allocated with no room for the output tokens and the model's own reasoning that the API counts against the same window | Reserve output budget explicitly (§12.1 reserves 4K) and include the provider's output allowance in the effective-window math |

The common thread: every anti-pattern is a *governance* failure dressed up as a *sizing* problem. Budgets fail when they are documents; they work when they are enforced, measured, and reviewed — the three properties the review checklist in §6.4 audits.

---

## 7. Freshness and Recency — Stale Context, Eviction, Refresh

### 7.1 The staleness problem

Context has a shelf life, and stale context is worse than no context: it is confidently wrong. Three failure classes recur in production:

- **Corpus staleness** — a cached policy prefix or indexed document set that no longer reflects the current state (a product term sheet updated last week, a regulatory notice published this morning).
- **Conversational staleness** — older user statements that the model weights equally with recent ones, so a superseded request ("actually, cancel that") competes with the original.
- **Rot within long sessions** — the §4.2 degradation curve means the *same* information recalled later in a session is recalled less reliably; Anthropic's engineering post describes the needed discipline as cyclically refining the accumulating data of an agent loop.

### 7.2 Eviction and refresh policies

Practical policies, each with a clear owner:

- **Recency-weighted history.** Keep recent turns verbatim, summarize older ones (rolling window + compaction, §8.4). Anthropic's Claude Code implementation is a concrete verified pattern: when nearing the window limit, the model summarizes the conversation — preserving architectural decisions, unresolved bugs, and implementation details while discarding redundant tool output — and continues with "this compressed context plus the five most recently accessed files."
- **Timestamped provenance on every context block.** Store `as_of`/`source_updated` metadata with retrieved content and cache prefixes; refresh on a schedule matched to the content's volatility (static corpus: on publish events; live data: per request — see the [RAG with Data Streaming guide](rag/rag_with_data_streaming_guide.md) for the streaming variant of this problem).
- **Explicit invalidation on change.** A corpus change must invalidate dependent caches and pre-computed summaries — the sibling [RAG vs Long-Context LLMs guide](rag/rag_vs_long_context_llms_guide.md) calls the missed invalidation "the most common silent-cost bug in long-context deployments."
- **Eviction with a paper trail.** For regulated domains, log what was evicted and why; auditability of *what the model did not see* is often as important as what it saw (§12).

### 7.3 Freshness policy by content class

Different content classes have different half-lives; one refresh policy for all of them is either wasteful or dangerous. The Cymbal Bank design in §12 applies exactly this table:

| Content class | Typical volatility | Refresh policy | Dominant failure mode if stale |
|---|---|---|---|
| System prompt / conduct rules | Very low | Versioned releases; change-controlled | Drift between approved and served policy |
| Product/policy corpus | Low–medium | Publish-event invalidation + nightly rebuild | Answering from withdrawn terms |
| Customer entitlements/profile | Medium | Daily re-fetch, event push on change | Serving privileges the customer no longer has |
| Live account data (balances, cards, transactions) | High | Per-request fetch, TTL ≤ minutes | Confidently reporting a stale balance |
| Conversation history | Grows continuously | Rolling window + compaction (§8.4) | Superseded requests competing with current ones |
| Retrieved evidence chunks | Per-query | Never reused across sessions unless re-validated | Citing a document version that changed mid-session |

The rule under the table: **refresh cadence = content half-life, never the pipeline's convenience.** A cache that outlives its content's half-life is not a cache; it is a liability.

### 7.4 Detecting staleness before it misleads

Eviction and refresh policies (§7.2) only work if staleness is *visible*. Four cheap detection heuristics catch most stale-context incidents at serving time, before a user is misled (engineering guidance, not cited research):

- **Age-vs-half-life alarms.** Every context block carries an `as_of` timestamp (§7.2). Compare block age against its content class's refresh cadence from the §7.3 table; log and alert on any block served past its class TTL — a transaction record older than its 5-minute live-data TTL should be impossible to serve, and if it happens, that is an incident, not a log line.
- **Re-fetch diffs.** When a block *is* refreshed, diff old vs. new content. A changed balance, a withdrawn product term, or a revoked entitlement is exactly the delta a downstream answer may already have relied on — the diff feed is what drives dependent-cache invalidation and summary regeneration (§7.2).
- **Conversational contradiction monitors.** In long sessions, a user's latest statement supersedes earlier ones ("actually, cancel that"). A lightweight check that flags when the current request contradicts a retained history turn lets the session manager compact the superseded turn *before* the model has to resolve the contradiction — cheaper than relying on the model to notice (§10.3).
- **Citation re-validation.** In grounded pipelines, every cited `source_id` (§12.4) resolves to a block that carries an `as_of`. Before serving, verify the cited block is not older than the current version of its source; a citation to a superseded document version is the audit-trail equivalent of a stale read.

None of these requires model judgment — they are deterministic metadata checks, which is exactly why they belong in the pipeline rather than the prompt. The model cannot be asked to notice that its context is stale; the context machinery has to refuse to be stale in the first place.

---

## 8. Context Compression — Summaries, Selection, and Learned Compression

Compression is the pressure valve of context engineering: the family of techniques that fit more *meaning* into fewer *tokens*. This section is about **context-window compression** — shrinking what is fed to the model at inference. It is a different problem from model-level weight/KV-compression (quantization, pruning, KV-cache eviction inside the serving engine), which the sibling [LLM Compression Alternatives guide](llm_compression_alternatives_guide.md) owns; the two meet only in that both fight the same memory and cost curves.

### 8.1 Hard vs. soft compression

- **Hard compression** — content is *removed* before it ever enters the window: truncation, top-k selection, eviction, tool-result clearing. Lossless for what remains, but lossy for what is dropped; the risk is dropping the token whose importance only becomes clear later (Anthropic warns that "overly aggressive compaction can result in the loss of subtle but critical context").
- **Soft compression** — content is *rewritten* into fewer tokens: extractive or abstractive summarization, condensation of history into bullet notes, distillation of a sub-agent's exploration into a 1,000–2,000-token report (§10.4). Keeps more semantics per token but can introduce summary error and loses verbatim retrievability.

Production systems use both: soft compression for what must persist (history, sub-agent results), hard compression for what must not (consumed tool output, superseded chunks).

### 8.2 Learned (model-based) compression — the LLMLingua family

A verified research line treats prompt compression itself as a learned task:

- **LLMLingua** (Jiang et al., arXiv:2310.05736, EMNLP 2023) — coarse-to-fine prompt compression with a *budget controller* to preserve semantic integrity at high ratios and token-level iterative compression; reports up to **20× compression with little performance loss** across GSM8K, BBH, ShareGPT, and arXiv QA.
- **LLMLingua-2** (arXiv:2403.12968, ACL 2024) — reformulates prompt compression as **token classification** ("keep or drop") using a distilled small model, targeting *faithful, task-agnostic* compression so the compressed prompt stays true to the original.

The family's own framing — compress before inference to cut cost and latency — is a direct alternative to retrieval for the "trim the corpus" problem, with the caveat that compression quality must be measured per workload (see §8.3). LLMLingua-2's faithfulness emphasis is the right instinct for regulated domains: compressed context must be provably consistent with source context.

### 8.3 Measuring the quality impact of compression

Compression always trades tokens for fidelity, and the trade must be measured, not assumed. The repo's evaluation cluster owns the measurement playbooks: [RAG Evaluation Methodology](rag/rag_evaluation_methodology_guide.md) (context precision/recall and answer-groundedness protocols) and [RAG Evaluation Tools Comparison](rag/rag_evaluation_tools_comparison_guide.md) (Ragas/TruLens tooling) apply directly to compressed pipelines — the standard move is to run the golden set on uncompressed vs. compressed context and record the delta. One short cross-reference note only: treat "compression hurt answer groundedness" the same way you would treat "retrieval recall dropped" — as a context-quality regression to be fixed before shipping.

### 8.4 Compaction: compression with continuity

**Compaction** is the operational form of soft compression for conversations: "taking a conversation nearing the context window limit, summarizing its contents, and reinitiating a new context window with the summary" (Anthropic). It is distinguished from naive summarization by intent — it exists to preserve *continuity of task state* (decisions, open problems, constraints) rather than to produce a readable digest — and by its tuning: Anthropic recommends tuning the compaction prompt on real agent traces, first maximizing recall (capture everything relevant), then improving precision (drop the superfluous). Compaction is the workhorse of long-running sessions and is covered as a pattern in §10.3.

### 8.5 A compression cascade in practice

When a session approaches its effective-window limit, apply compression in ascending order of destructiveness — each stage is reversible or cheap until the last one:

1. **Tool-result clearing** (§5.2): drop consumed raw tool payloads to one-line summaries. Biggest win, smallest risk; always on.
2. **Truncate the evidence tail**: drop the lowest-ranked retrieved chunks first (they contributed least; the eviction order in §6.3 already encodes this).
3. **Compact history** (§8.4): summarize turns older than the recent window into the session-state block, tuned for high recall first.
4. **Learned compression on what remains**: run an LLMLingua-2-class compressor over the surviving evidence if the quality gate (§8.3) passes on your golden set.
5. **Sub-agent or deferred processing**: move the token-hungry remainder out of the window entirely — a sub-agent digests it and returns 1,000–2,000 tokens (§10.2).

Each stage has a measurement: after stages 1–4, re-run the groundedness spot-check; after stage 5, verify the summary preserved the facts the current question needs. The cascade exists because the cheapest stage is not always sufficient — and the most destructive stage is almost never necessary.

---

## 9. Retrieval-Fed Context Assembly

This section covers the *assembly logic* that turns retriever output into a well-formed window. The retrieval mechanics themselves — chunking, indexing, hybrid search, reranking — are owned by the RAG cluster and are not re-derived here; the four sibling guides are referenced at the point each one's mechanics become relevant.

### 9.1 Assembly rules that survive any retrieval stack

- **Serialize with structure, not as flat text.** Wrap each retrieved unit in typed delimiters (`<document id="…" source="…" as_of="…">`), order by relevance (§5.3), and keep a short corpus summary at the top of the evidence block as a map. Structure survives serialization; sibling guides in this repo use identical conventions ([RAG Optimization Techniques](rag/rag_optimization_techniques_guide.md) is the canonical reference for chunking/serialization trade-offs).
- **Two-tier evidence: summaries for breadth, full text for depth.** When the relevant set exceeds the premium positions, feed per-document summaries plus the full text of the top documents only. (The repo's deep treatment: [Advanced RAG Techniques](rag/advanced_rag_techniques_guide.md) — hierarchical/map-reduce retrieval patterns.)
- **Design for cache geometry.** Because providers bill cache hits at a fraction of fresh input (Anthropic: 0.1× on standard models; OpenAI: 2× surcharge on sessions crossing 272K — §4.4), the context should be ordered as **[stable prefix][per-query suffix]** so the prefix hits cache across requests. The stable prefix is the system prompt plus the standing corpus section; everything user- or query-specific goes after it. Cache invalidation on corpus change is a pipeline responsibility (see §7.2).
- **Route before you assemble.** Not every request needs retrieval, and not every request needs the standing prefix. Adaptive routing (RAG vs. long-context vs. hybrid per query) is owned by the [Advanced RAG Techniques guide](rag/advanced_rag_techniques_guide.md) and the strategic case is in the [RAG vs Long-Context LLMs guide](rag/rag_vs_long_context_llms_guide.md) — from a context-engineering view, routing is just *budget allocation at request time* (§6).

### 9.2 Just-in-time context vs. pre-assembled context

Anthropic's 2025 framing distinguishes two assembly paradigms, both of which are context engineering:

- **Pre-inference retrieval** — "embedding-based pre-inference time retrieval to surface important context for the agent to reason over": assemble everything before the call. Fast, deterministic, but pays for content that may go unused and can go stale between indexing and serving.
- **Just-in-time retrieval** — the agent holds lightweight references (file paths, stored queries, URLs) and loads content into context at runtime with tools: "rather than pre-processing all relevant data up front… dynamically load data into context at runtime." Claude Code is the verified exemplar: standing instructions dropped in up front, `glob`/`grep`-style primitives pulling files in on demand — explicitly framed by Anthropic as "bypassing the issues of stale indexing."
- **Progressive disclosure** — each tool interaction yields context that informs the next decision; agents "assemble understanding layer by layer, maintaining only what's necessary in working memory" (Anthropic). This is the mechanism by which an agent's context *grows a design* instead of *dumping a corpus*.

The streaming variant of freshness — where context arrives incrementally from live sources rather than being fetched per query — is the subject of the sibling [RAG with Data Streaming guide](rag/rag_with_data_streaming_guide.md), which owns the incremental-ingestion and window-update mechanics.

### 9.3 When assembly must be agentic

If retrieval decisions themselves must be iterative — the query is vague, the evidence is spread across systems, or verification loops are required — assembly moves from a linear pipeline into an agent loop. That pattern (agentic RAG: retrieval as a skill inside a reasoning loop, with self-reflection and correction) is owned by [Production-Grade Agentic RAG](rag/production_grade_agentic_rag_guide.md). Its context-engineering contribution is the *loop discipline*: each retrieval round appends to the window, so the assembly must budget for accumulation (§6) and prune consumed evidence between rounds (§5.2, tool-result clearing) or the loop drowns in its own output — the single most common agentic-RAG failure mode, and one the agent cluster's [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) documents from the failure side.

### 9.4 The cache-prefix engineering checklist

Since cache geometry (§9.1) is where context engineering directly meets the bill, treat the prefix as an engineered artifact with its own review:

1. **One stable prefix, one volatile tail.** System prompt + standing corpus = prefix; anything request-specific (user, query, evidence, history tail) = suffix. No interleaving that moves prefix bytes between requests.
2. **Order the prefix by frequency of use across requests**, not by topic: the most-shared corpus block first after the system prompt, so even partial prefix matches hit.
3. **Keep the prefix byte-identical across turns.** Cosmetic regeneration (timestamps, reordered bullets) silently defeats the cache — the sibling long-context guide calls this family of bugs the most common silent-cost failure in long-context deployments.
4. **Invalidate on content change, not on a timer** (§7.2) — publish events for the corpus, re-fetch events for entitlements.
5. **Measure the hit rate** per endpoint (provider consoles expose it) and alert on drops; a cache-hit regression is usually a prompt-engineering change in disguise.
6. **Mind provider cliffs** (the >272K session multiplier on OpenAI-class pricing, §4.6): prefix size is not free even when cached — the first write of a longer prefix is 1.25–2× base on Anthropic pricing (§4.4).

---

## 10. Agentic and Long-Running Context

Agents are the reason context engineering became its own discipline: a prompt is static, but an agent loop *generates context continuously* — tool calls, observations, partial results, sub-agent reports — and every new turn re-decides what of that accumulation deserves the window. Anthropic's post is explicit: "An agent running in a loop generates more and more data that *could* be relevant for the next turn of inference, and this information must be cyclically refined."

### 10.1 Virtual context management — MemGPT and the OS analogy

The foundational research framing for context that outlives the window is **MemGPT** (Packer, Wooders, Lin, Fang, Patil, Stoica, Gonzalez — arXiv:2310.08560, October 2023): "virtual context management, a technique drawing inspiration from hierarchical memory systems in traditional operating systems that provide the appearance of large memory resources through data movement between fast and slow memory." MemGPT manages memory tiers (in-window main context vs. out-of-window external context) and uses interrupts for control flow, evaluated on document analysis and multi-session chat where the model "remember[s], reflect[s], and evolve[s] dynamically." The OS analogy is the correct mental model for every long-running agent: the context window is RAM, and everything else — vector stores, notes files, memory tools, session stores — is disk that must be *paged in* deliberately.

### 10.2 The three verified long-horizon techniques

Anthropic's engineering post names the techniques it uses for tasks spanning tens of minutes to hours (codebase migrations, research projects):

1. **Compaction** (§8.4) — summarize-and-restart when the window fills; preserves conversational flow for tasks requiring extended back-and-forth.
2. **Structured note-taking (agentic memory)** — the agent periodically writes notes *outside* the window (a to-do list, a `NOTES.md`, a memory-tool file) and pulls them back in later. The verified exemplar is striking: Claude playing Pokémon maintained precise tallies, region maps, and strategy notes across thousands of game steps and multiple context resets — "after context resets, the agent reads its own notes and continues" — coherence across summarization steps that would be impossible in a single window. Anthropic shipped this as a file-based **memory tool** (public beta alongside Sonnet 4.5) so agents "build up knowledge bases over time, maintain project state across sessions, and reference previous work without keeping everything in context."
3. **Sub-agent architectures** — specialized sub-agents do deep, token-hungry work in *their own* clean windows and return only distilled summaries: "each subagent might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000–2,000 tokens)." The lead agent's context stays small and synthesizing; Anthropic reports "substantial improvement over single-agent systems on complex research tasks" from this pattern (multi-agent research system).

Technique choice is task-shaped: compaction for conversational flow, note-taking for milestone-driven development, sub-agents for parallel exploration. The agent-mechanics layer — how the loop, tool registry, and prompt scaffold are built — is owned by the sibling [Agent Scaffolding guide](agent_scaffolding_guide.md), and the viability economics of retrieval loops by [Production-Grade Agentic RAG](rag/production_grade_agentic_rag_guide.md).

### 10.3 Session management: rotation, rollover, episodic summarization

Long-running production sessions need explicit lifecycle machinery, which the verified techniques above compose into a policy:

- **Context rotation/rollover.** Define a fill threshold (e.g., 70–80% of the *effective* window, §6.2). At threshold, trigger compaction: summarize the session so far into a **session state block** (decisions, open items, user constraints, key facts), start a fresh window, and re-inject: [stable prefix] + [session state block] + [recent N turns verbatim]. Anthropic's Claude Code pattern — compressed context plus the five most recently accessed files — is the canonical shape.
- **Episodic summarization between turns** (not only at the limit). For sessions expected to span hours or days, write the state block after every milestone, not just at rollover — this is structured note-taking applied to sessions, and it is what lets a rolled-over agent resume with full task state rather than a vague recap.
- **External memory for what must outlive the session.** Anything the business needs across sessions (customer context, project state, learned preferences) belongs in an explicit memory tier (memory tool, vector store, database) — never only in a rolled-over summary. The memory-architecture trade-offs (short-term vs. episodic vs. semantic memory, and their storage) are part of the agent cluster's scaffolding/memory guides; from the context-engineering view the rule is simple: *the window holds working state; durable state lives outside and is paged in on purpose* (MemGPT's core lesson, §10.1).
- **Guardrails survive rollover.** Instructions that must never be lost — safety policy, output contracts, escalation rules — must be re-injected from the stable prefix on every rollover, never left to the summary's mercy.

### 10.4 The context lifecycle state machine

Long-running agents benefit from making the context lifecycle explicit — a small state machine every loop iteration consults:

| State | What happens to the window | Trigger | Guardrail |
|---|---|---|---|
| **Assemble** | Stable prefix + session state + evidence + history tail assembled per §9 | Start of turn | Budget check (§6); cache geometry intact (§9.1) |
| **Reason** | Model generates against the assembled window | Assembly done | Output contract enforced at the tail (§12.2, block 8) |
| **Act** | Tool calls execute; raw results land in a staging area, *not* directly in history | Model requests a tool | Result payloads capped; one-line summaries only (§5.2) |
| **Prune** | Consumed tool results cleared; evidence re-ranked for the next question | After every tool round-trip | Eviction order per §6.3; eviction log written (§7.2) |
| **Milestone** | Session-state block rewritten from notes (decisions, open items, facts) | Task milestones; every N turns | High-recall compaction prompt (§8.4) |
| **Rollover** | Compaction summary + last turns verbatim + fresh window | Fill ≥ 75–80% of effective window | Guardrail block re-injected from stable prefix (§10.3) |
| **Persist** | Durable state written to memory tool / session store | Session end, or on interrupt | Audit manifest completed (§12.4) |

The state machine is deliberately boring — its value is that *every* transition has an owner and a trigger, so "the window slowly filled with garbage" stops being a mysterious incident and becomes a missed transition at a known state.

### 10.5 The memory-tier map: what lives where

The OS analogy (§10.1) becomes operational policy once the tiers are named and each is assigned a retention rule and a page-in trigger. The map used across this repo's long-running agent designs:

| Tier | Contents | Backing store | Page-in trigger | Retention rule |
|---|---|---|---|---|
| **Working memory (in window)** | Current question, top evidence, recent verbatim turns, session-state block (§12.2) | The context window itself | Every turn, per the §6 budget | Evicted per §6.3; never the only copy of anything durable |
| **Notes / scratch (near-line)** | Agent working notes, to-do list, per-milestone state | Memory tool file, `NOTES.md` (§10.2) | Read on rollover and when the model signals it needs them | Survives the session; the raw material for episodic summaries |
| **Session store (off-line)** | Full turn logs, tool payloads, assembled-context manifests | Durable session storage | Audit and replay (§12.4); debugging | Regulatory window, append-only |
| **Episodic memory** | Cross-session summaries: what happened, what was decided, what the customer was told | Memory tool / vector store | When a new session touches the same customer or project | Long-lived; refreshed on contradiction (§7.4) |
| **Semantic / knowledge base** | Product, policy, and procedural knowledge | The standing corpus + retrieval stack (§9) | Per-request retrieval, cache-shaped as the prefix | Content half-life per §7.3 |

Two rules keep the map honest. First, **nothing is written to only one tier**: anything the business needs later exists in at least two (e.g., the session-state block is mirrored in the notes tier before rollover). Second, **page-in is always deliberate**: the model asks for a note or the assembler fetches a retrieval — context never appears in the window by ambient accumulation, which is the failure mode the state machine's **Act → Prune** transition exists to prevent (§10.4). With the tiers named, MemGPT's "the window is RAM" stops being a metaphor and becomes a storage architecture with explicit owners — the same move §1.5 makes for the discipline's artifacts.

---

## 11. Evaluating Context Quality

Context quality is a first-class property only if it is measured. The evaluation *tooling and protocols* belong to the sibling guides — [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md) (the general eval methodology), [RAG Evaluation Methodology](rag/rag_evaluation_methodology_guide.md), and [RAG Evaluation Tools Comparison](rag/rag_evaluation_tools_comparison_guide.md) (Ragas/TruLens-class tooling) — so this section only defines the *context-specific* metric families and how they map to design decisions:

- **Sufficiency (coverage).** Did the assembled context contain everything needed to answer? Operationally: can a strong model answer the golden question from the context alone (retriever/assembler recall), and does answer quality collapse when the context is removed (an ablation that isolates *context contribution* from parametric knowledge)? This is the family RAG evaluation calls context recall / answer relevance — protocols in the methodology guide.
- **Precision (economy).** How much of the context was irrelevant or redundant? Low precision is not just a cost problem; per §4.2 and §5, distractor tokens actively degrade recall of the relevant ones (verified: Chroma's context-rot experiments vary "question-answer similarity" and distractor content and show non-uniform degradation). Context precision metrics catch assembly bloat that token budgets (§6) miss.
- **Groundedness (faithfulness).** Are the model's claims traceable to the supplied context — citation coverage, no invented facts? This is the non-negotiable family for regulated deployments, and it is where compression (§8) does its damage: a compressed context that loses the fact behind a cited sentence converts a good pipeline into a hallucinating one while *looking* identical. Always re-run groundedness evals after any change to compression or compaction prompts.
- **Lifecycle health.** Beyond per-answer metrics: eviction logs complete? Cache hit rates where expected? Fill-rate histograms per component within budget (§6)? These operational signals catch context-design regressions (a retriever whose top-k drifted, a compaction prompt gone lossy) weeks before users complain.

### 11.1 Context-quality metrics at a glance

| Metric family | Question it answers | Typical implementation | Failure it catches |
|---|---|---|---|
| Sufficiency / context recall | Did the window contain what was needed? | Golden-set answerability from context alone; ablation vs. no-context | Retriever/assembler gaps; over-aggressive eviction |
| Precision | How much of the window was noise? | Share of chunks cited or judged relevant per answer | Assembly bloat; distractor-driven degradation (§4.2) |
| Groundedness / faithfulness | Are claims traceable to context? | Citation coverage; claim-level verification against sources | Hallucination from compressed or stale context (§8.3) |
| Contribution | Did the context change the answer? | Answer-with-context vs. answer-without-context delta on golden set | Parametric-knowledge crutch hiding a useless pipeline |
| Lifecycle health | Is the machinery still within spec? | Fill rates per component; cache-hit rates; eviction-log completeness | Drift: top-k creep, compaction lossiness, cache breakage |

The metrics map one-to-one onto the design sections: sufficiency audits §5/§9, precision audits §6, groundedness audits §7/§8, lifecycle health audits §6.4/§9.4. If a team only runs one of these, groundedness is the one — it is the only family that directly measures the *answer the customer received* rather than the machinery that produced it.

### 11.2 The honest summary

The evidence accumulated across this repo's evaluation guides points one way: most "model quality" regressions in production RAG and agent systems trace back to context-assembly changes — a reranker swap, a compression ratio bump, a cache-prefix reorder — not to the model. Evaluating the context is evaluating the system.

### 11.3 Where context evals live in this repo

| Need | Sibling guide |
|---|---|
| Protocol design (golden sets, judge setup, ablation methodology) | [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md), [RAG Evaluation Methodology](rag/rag_evaluation_methodology_guide.md) |
| Tooling (Ragas/TruLens-class metric automation) | [RAG Evaluation Tools Comparison](rag/rag_evaluation_tools_comparison_guide.md) |
| When context quality fails despite good retrieval | [RAG vs Long-Context LLMs](rag/rag_vs_long_context_llms_guide.md) (effective-context measurement) |
| Model-level compression evals (quantization/KV) | [LLM Compression Alternatives](llm_compression_alternatives_guide.md) |

The division of labor mirrors the rest of this guide: the mechanics live in the sibling guides; the *context-specific metric families* — sufficiency, precision, groundedness, contribution, lifecycle health (§11.1) — are this guide's contribution to the shared evaluation vocabulary.

---

## 12. Worked Example — A Cymbal Bank Retail-Banking Context Design

This section applies the full framework to a concrete production design: the **Cymbal Bank retail-banking assistant** — the persona used across this repo's series — for a session in which a customer asks why a card payment was declined and whether a fee can be reversed, then follows up with a product question about a rewards program. The design below is the *context architecture* only: the retrieval pipeline that feeds it follows the repo's [RAG Optimization Techniques guide](rag/rag_optimization_techniques_guide.md), and the agent loop follows the [Agent Scaffolding guide](agent_scaffolding_guide.md).

### 12.1 Design constraints

- **Model:** a 1M-token-window frontier model (per §4.1, e.g., the Claude Opus 5 / Sonnet 5 class). **Effective working budget: 200K tokens** — 20% of the marketed window — per the §4.2 degradation evidence and the §6.2 rule to budget against effective, not marketed, context. Output budget: 4K reserved separately.
- **Content classes present in every session:** identity and entitlements; the customer's accounts and risk posture (live, permissioned); product/policy knowledge (a large, mostly static corpus); live transaction and fee data; the conversation itself; compliance and escalation rules.
- **Regulatory posture:** every answer must be traceable to retrieved source or live account data; nothing may be answered from parametric memory alone; eviction and refresh must be auditable; a human handoff path must exist for disputes and fraud.
- **Cross-cluster note:** the Cymbal Bank assistant in this repo's other guides ([multi_agent_banking_guide.md](multi_agent_banking_guide.md), [llm_agent_use_cases_finance_guide.md](llm_agent_use_cases_finance_guide.md)) adds agentic orchestration on top of this same context substrate; this design is the substrate.

### 12.2 The context budget (per session, mid-conversation snapshot)

| # | Component | Token allocation | % of 200K | Position | Cache status |
|---|---|---|---|---|---|
| 1 | System prompt: identity, conduct rules, output contract (JSON + citations), escalation triggers | 3,000 | 1.5% | Very start (premium) | Stable prefix; cache hit |
| 2 | Customer envelope: entitlements, KYC/risk flags, language, channel | 2,000 | 1% | Immediately after system | Daily refresh; cache hit |
| 3 | Standing product & policy prefix: top-tier terms, fee schedules, dispute policy, rewards terms (retrieved summaries + full text of top ~40 docs) | 40,000 | 20% | Early middle | Corpus cache; refreshed on publish, invalidated explicitly |
| 4 | Session state block: customer goal, decisions made, open items, facts established this session, pending disclosures | 4,000 | 2% | Before live evidence | Rewritten at every milestone (§10.3); survives rollover |
| 5 | Live evidence (per-request retrieval): transactions (30 days), card status, fee records, dispute history, current product terms diff | 60,000 | 30% | Middle, ordered by relevance to the open question | Fresh per request; short-TTL cache; never reused across sessions |
| 6 | Conversation history: last 6 turns verbatim + older turns compacted | 30,000 | 15% | Late middle | Compaction-triggered; see §12.4 |
| 7 | Tool scratchpad | 0 | 0% | — | Banned from the window; notes tool + external session store instead (§10.2) |
| 8 | Current user request + required output schema | 1,000 | 0.5% | Very end (premium) | Per turn |
| — | **Headroom (unallocated)** | **60,000** | **30%** | — | Absorbs retrieval overruns, long documents, multi-hop follow-ups |

### 12.3 Ordering rationale (why each block sits where it sits)

The ordering implements §5's premium-position logic and §9.1's cache geometry:

- **Blocks 1–3 form the stable cache prefix** (45K tokens, ~90% cache-hit economics per §4.4). Putting identity and conduct rules at the very start exploits the strongest attention position for the content whose violation is costliest — consistent with the verified finding that the *start* of the context gets the most reliable use (§5.1). They change rarely, so the cache prefix is stable across turns and across customers sharing the same product prefix.
- **Block 4 (session state) sits between the stable prefix and the live evidence** because it must be attended to whenever the model reasons about *what has already been established* — it is the highest-value non-static block and earns the best non-prefix position. It is rewritten, not appended, so it never grows (§10.3).
- **Block 5 (live evidence) is the middle**, ordered inside itself by relevance to the current open question (best-first, §5.3): for the declined-card query, the card-status record and the specific decline event outrank the fee schedule; for the rewards follow-up, the rewards terms move to the front of the evidence block. Middle placement is acceptable because the model must *hold* this material rather than *obey* it precisely — but its 60K allocation is capped so it cannot push history or instructions into attention starvation (§6.2).
- **Block 6 (history) is placed after the evidence, not before it**, so that stale conversational context does not compete with fresher evidence for early attention; recency is preserved by keeping the last turns verbatim at the *tail of the block* — the second premium position — immediately before the current request.
- **Block 8 (current request + output contract) anchors the very end**, the other verified premium position. The output schema is repeated here (not only in the system prompt) so that format adherence is reinforced at the point of generation.
- **Tool scratchpad is banned from the window by policy** (block 7): working notes live in the memory tool / session store and are paged in only when the model signals it needs them (MemGPT-style virtual context management, §10.1) — this single rule keeps agentic turns from degrading into self-conversation.

### 12.4 Compression and retention policy

| Policy | Rule | Trigger / cadence | Owner |
|---|---|---|---|
| History compaction | Turns older than 6 verbatim turns are summarized into block 4; decisions, unresolved items, and stated customer constraints are preserved with high recall first (§8.4) | Fill ≥ 75% of the 200K budget, or every 10 turns, whichever first | Session manager |
| Tool-result clearing | Tool payloads (API responses, retrieval dumps) are dropped from the window once consumed; only a one-line `result_summary` remains | After every tool round-trip | Agent loop |
| Evidence eviction order | scratchpad → consumed tool results → lowest-ranked retrieved chunks → older history → (never) system prompt, session state, current request | Continuous; pre-agreed per §6.3 | Session manager |
| Cache invalidation | Product/policy prefix invalidated on any publish event; entitlements re-fetched daily; live evidence TTL ≤ 5 minutes | Event- and schedule-driven (§7.2) | Pipeline |
| Retention & audit | Full request/response + assembled-context manifest (component token counts, sources, eviction log) retained for the regulatory window; summaries stored with the session | Every turn, append-only | Audit service |
| Compression quality gate | Golden-set groundedness suite re-run on any compaction-prompt or compression-ratio change; regression blocks rollout (§8.3, §11) | On change | Evaluation team |

Two Cymbal Bank specifics worth calling out because they are banking-shaped, not generic:

1. **The audit trail is part of the context design, not an afterthought.** The manifest records what the model *did not see* (evicted chunks, expired cache entries) with reasons — for a regulated assistant, "why was this document excluded" is a first-class question a regulator can ask, and the eviction log (§7.2) is the answer.
2. **Citation grounding is enforced by the output contract, not by hope.** The block-8 schema requires every claim to carry a `source_id` that must resolve to a block-5/block-3 document or a block-2 entitlement — and the evaluation suite (§11) checks citation coverage per answer. Compression is the quiet risk here: if a compaction summary drops the fee-waiver eligibility fact mid-session, the model will cite a source it no longer contains — which is exactly the failure the groundedness gate in the policy table exists to catch.

The design composes the whole framework: a cache-shaped stable prefix (§9.1), premium-position ordering from the lost-in-the-middle evidence (§5), an explicit budget with pre-agreed eviction (§6), freshness policies with auditable invalidation (§7), compaction and notes-based compression (§8, §10), and a measurement gate on every change (§11) — while deliberately owning *none* of the retrieval or agent mechanics, which stay in their sibling guides.

### 12.5 Request archetypes and their context recipes

The same budget table composes differently per request type. Three archetypes from the Cymbal Bank assistant's traffic:

| Archetype | Example | Evidence fetched (block 5) | Notes on composition |
|---|---|---|---|
| **Account state query** | "Why was my card declined?" | Card status, decline event, 30-day transactions, fee records — narrow, fresh, high-precision | Small evidence slice (~10–15K); history matters (context of the attempt); answers must cite the decline record; risk flags consulted but not quoted |
| **Product/policy question** | "Do rewards points expire?" | Rewards terms from the standing prefix (§12.2, block 3) + eligibility from the customer envelope | Evidence mostly *already in the prefix* — the query may need no fresh retrieval at all; cache geometry does the work; the policy version cited must be the current one (§7.3) |
| **Service workflow** | "Reverse this late fee" | Fee policy, account history, dispute/waiver rules, prior waivers granted | Longest-lived turn: multiple tool round-trips; the session-state block must carry the workflow stage across turns (§10.4); waiver decisions require the audit trail and a human-handoff path by policy |

The recipes differ in *volume, freshness, and persistence* but share the same skeleton — which is the point of a budget document: the skeleton is stable, the slices vary.

### 12.6 Surviving the next model swap

The design above is written to survive the window arms race (§4.1). When Cymbal Bank swaps models:

- The budget is defined in **tokens and shares, not model names** — a new model changes the *effective* window assumption (§6.4, item 2), and the headroom column absorbs the difference.
- Position ordering and the eviction order are **re-validated, not assumed**, because position sensitivity is model-specific (§5.4).
- Cache geometry survives only if the prefix **stays byte-identical** across the migration — a model swap is the classic moment someone "tidies" the system prompt and silently orphans the cache.
- Groundedness and compression-quality gates (§12.4) **re-run before rollout**, not after.

The context budget is the contract that makes model churn a config change instead of a redesign — the single highest-ROI artifact in this guide for any team on a fast-moving vendor cadence.

### 12.7 The budget arithmetic on two concrete turns

To show the budget doing real work, here is the declined-card turn and the fee-reversal follow-up, token by token. All arithmetic uses the §12.2 allocations and the §4.4 Opus 5 list prices ($5/MTok input, 0.1× cache-hit, 1.25× first cache write at 5-minute retention; output reserved at 4K, §12.1).

**Turn 1 — "Why was my card declined?" (a card-payment dispute opener).** The stable prefix (blocks 1–3) is already cached from earlier turns: 3,000 + 2,000 + 40,000 = **45,000 tokens on a cache hit**. The volatile tail is assembled fresh: block 4 session state (4,000) + the block-5 evidence slice for this query — card-status record (~3K), 30-day transactions (~25K), the specific decline event (~2K), fee records (~3K) — ≈ 33K in this example, well inside its 60K allocation + block 6 history (30,000) + block 8 request (1,000). Total window ≈ **113K of the 200K budget** (56.5%), leaving ~87K headroom. Cost of the turn: 45K × $0.50/MTok (hit) + 68K × $5/MTok (fresh tail) ≈ **$0.02 + $0.34 ≈ $0.36 of input**, plus a few cents of output. The same turn with an uncached 113K prefix would be ~$0.57 of input — the cache saves ~$0.21 (about a third) per turn on this shape.

**Turn 6 — "…and can you reverse the fee?" (a service workflow).** Now the session has run several tool round-trips: a balance check, a dispute-policy read, and an eligibility check each landed payloads in the window and were cleared to one-line summaries (§12.4). The evidence slice has rotated toward fee policy, waiver rules, and prior waivers granted (~40K), history has grown past 6 turns and been compacted once: block 6 stays at 30,000 by construction (§12.4) while block 4 grows from 4,000 to 6,000 to carry the workflow stage, disclosures, and the eligibility verdict. Window ≈ 45K (cached prefix) + 6K + 40K + 30K + 1K = **122K (61%)** — still under the **150K rollover trigger** (75% of the 200K effective budget), so no compaction is forced; the session manager's *Milestone* transition (§10.4) has already rewritten block 4 after each tool round.

**The rollover, when it comes.** A long dispute session eventually crosses 150K. Per §12.4, history and the older evidence slices are compacted: block 4 absorbs the decisions and unresolved items (6K → 8K), block 6 resets to the last 6 verbatim turns (~10K), and the stale evidence is evicted with a manifest entry. The fresh window starts at ≈ 45K + 8K + 10K + new evidence (~30K) + request (1K) ≈ **94K (47%)** — a full session of headroom reclaimed, with continuity preserved by the state block rather than by an ever-growing window.

The point of walking the arithmetic is that none of it is exotic: it is addition against a published table, and it turns every design question ("will this fit?", "when do we compact?", "what does this cost?") into a lookup instead of a guess. That is what an owned, enforced budget (§6.5) buys — and it is the difference between context engineering and context hoping.

---

## What Could Not Be Verified

This section records, honestly, the claims this guide could *not* verify at a primary source during the September 2026 research pass, plus the fast-moving numbers that are inherently fragile:

1. **The coinage and earliest usage of "context engineering" in the LLM sense.** The earliest verifiable anchor points found were the July 2025 survey (arXiv:2507.13334) and Anthropic's September 2025 engineering post — but neither is a "first use," and secondary write-ups claiming popularization in June 2025 via Andrej Karpathy and Tobi Lütke (Shopify) could not be confirmed at a primary source. The term's pre-LLM history in other engineering fields makes any "first use" claim inherently contestable. Marked ⚠ throughout.
2. **OpenAI's "A Practical Guide to Building Agents."** Multiple searches during this pass returned no verifiable result for this widely-referenced document, so it is deliberately not cited here; treat any OpenAI agent-guidance claims in older sibling guides against their own verification notes. Marked ❌ for this pass.
3. **The headline claim of Anthropic's follow-up post "The new rules of context engineering for Claude 5 generation models"** — that Anthropic removed over 80% of Claude Code's system prompt for advanced models. The post's existence is confirmed and the claim is quoted from its own summary, but it was not independently re-measured or verified against Claude Code internals. Marked ⚠.
4. **GPT-5's (August 2025) exact default context window.** The official model page was not retrievable in this pass; the 272K figure comes from third-party trackers and is marked ⚠ in §4.1. GPT-5.4/5.6's 1.05M windows are verified from OpenAI API documentation snippets.
5. **Llama 4 Scout's practically usable context length near its 10M claim.** The 10M figure is verified from Meta's official model card; no independent evaluation of reliable performance at that length was verified in this pass. Marked ⚠.
6. **Current Mistral API model context windows (Mistral Large-class).** Only the Mistral 7B open-model paper (SWA/GQA) was verified; the vendor's current hosted-model specs were not re-checked this pass. Marked ⚠ where referenced.
7. **Context windows of the immediately previous Claude generation (Sonnet 4.6 / Opus 4.6-class models),** which appear in this repo's older sibling guides. Anthropic's current documentation lists Opus 5 / Sonnet 5 / Fable 5.1 at 1M and prices the older Sonnet 4.6 tier, but this pass did not re-verify the previous generation's window sizes; §4.1 therefore reports only current-generation figures. Where older sibling numbers disagree with the table above, treat the documentation retrieved for this guide as the newer reading.

---

## Glossary

| Term | Definition |
|---|---|
| **Context engineering** | The discipline of curating and maintaining the optimal set of tokens presented to an LLM at inference — selection, structure, ordering, and lifecycle of everything in the window (system prompts, retrieved data, tool results, history, memory). |
| **Context window** | The maximum number of input tokens a model accepts per request; a *capacity*, not a guarantee of reliable use at that length. |
| **Effective context** | The portion of the marketed window over which a model performs acceptably on real workloads; measured, not assumed (§4.2). |
| **Episodic summarization** | Writing a session-state summary after milestones rather than only at the window limit, so a rolled-over session resumes with full task state (§10.3). |
| **Attention budget** | The finite attention capacity a model spreads over the tokens in its window; the conceptual basis for treating every added token as a marginal drain on the rest. |
| **Context rot** | Empirically observed degradation of model performance as input length increases, holding task complexity constant (Chroma, July 2025; §4.2). |
| **Lost in the middle** | Positional bias in which models use information at the start and end of long contexts far more reliably than information in the middle (Liu et al., 2023; §5). |
| **Prompt engineering** | Writing and structuring instruction text; the predecessor discipline that context engineering subsumes (§3.1). |
| **KV cache** | Cached key/value activations from the attention computation over already-seen tokens; grows with sequence length and batch and dominates long-context serving memory. |
| **Prompt caching** | Provider-side reuse of a stable context prefix across requests, billed at a fraction of fresh input cost (§4.4, §9.1). |
| **Premium position** | A window location (very start, very end) over which models use information most reliably per the lost-in-the-middle evidence; reserved for the most load-bearing content (§5). |
| **Context budget** | An explicit token-allocation plan across context components, with pre-agreed eviction order (§6). |
| **Compaction** | Summarizing a conversation near the window limit and restarting with the summary, preserving task continuity (§8.4). |
| **Tool-result clearing** | Dropping consumed tool outputs from history, keeping at most a one-line summary (§5.2, §10.3). |
| **Virtual context management** | OS-inspired paging of information between in-window memory and external storage (MemGPT; §10.1). |
| **Just-in-time context** | Loading content into the window at runtime via tools rather than pre-assembling everything before inference (§9.2). |
| **Scratchpad** | Intermediate working notes; kept outside the window and paged in deliberately in production designs (§12). |
| **Session rollover** | The lifecycle event of compacting a window near its fill threshold and restarting with the summary, state block, and recent turns re-injected (§10.3, §12.7). |
| **Groundedness / faithfulness** | The degree to which a model's output is traceable to the supplied context; the core quality family for regulated deployments (§11). |
| **Compression (context-level)** | Fitting more meaning into fewer tokens before/around inference — distinct from model-level weight/KV compression (§8). |
| **Context management** | The operational subset of context engineering: storing, summarizing, clearing, and restoring context across a session (§2.3). |
| **Cache geometry** | Ordering a request as stable-prefix + volatile-tail so provider prompt caches hit across requests (§9.1, §9.4). |

---

## Claims Audit

Status legend: ✅ verified at primary source this pass · ⚠ partially verified / contested / fast-changing · ❌ could not verify.

| # | Claim in this guide | Status | Source (verification) |
|---|---|---|---|
| 1 | Anthropic defines context engineering as curating the optimal token set during inference, "the natural progression of prompt engineering" | ✅ | anthropic.com/engineering/effective-context-engineering-for-ai-agents (published Sep 29, 2025; page extracted) |
| 2 | Anthropic's guiding principle: smallest possible set of high-signal tokens | ✅ | Same post, conclusion section |
| 3 | Context rot: recall decreases as context tokens increase, across all models | ✅ | Same post + Chroma tech report (research.trychroma.com/context-rot, Hong/Troynikov/Huber, July 2025, 18 LLMs) |
| 4 | Transformer attention creates n² pairwise token relationships | ✅ | Same post (architecture claim) |
| 5 | Lost in the Middle: performance highest at start/end, degrades in the middle, even for long-context models | ✅ | arXiv:2307.03172 abstract (Liu et al., TACL 2023) |
| 6 | RULER: 17 models; near-universal drops as context grows; only ~half maintain satisfactory performance at 32K | ✅ | arXiv:2404.06654 abstract (Hsieh et al., COLM 2024) |
| 7 | Context-engineering survey exists (July 2025); taxonomy: retrieval/generation, processing, management → RAG, memory, multi-agent | ✅ | arXiv:2507.13334 abstract (Mei et al., submitted Jul 17, 2025) |
| 8 | FlashAttention: self-attention time/memory quadratic in sequence length; IO-aware; 2–4× speedups; linear-memory exact attention | ✅ | arXiv:2205.14135 abstract |
| 9 | FlashAttention-2: ~2× over FlashAttention, up to 225 TFLOPs/s per A100 | ✅ | arXiv:2307.08691 abstract |
| 10 | Mistral 7B uses GQA + sliding-window attention for arbitrary-length sequences at reduced inference cost | ✅ | arXiv:2310.06825 abstract |
| 11 | MemGPT: virtual context management, OS-inspired memory tiers, interrupts | ✅ | arXiv:2310.08560 abstract (Packer et al., Oct 2023) |
| 12 | LLMLingua: up to 20× prompt compression with little loss (EMNLP 2023); LLMLingua-2: token-classification compression (ACL 2024) | ✅ | arXiv:2310.05736; arXiv:2403.12968 abstracts |
| 13 | Gemini 1.5: reasoning over millions of tokens; near-perfect long-context recall | ✅ | arXiv:2403.05530 (Gemini 1.5 report) |
| 14 | Gemini 2.5 Pro: 1,048,576 input / 65,536 output token limits | ✅ | ai.google.dev model page (updated Jun 2026) |
| 15 | Claude Opus 5 / Sonnet 5 / Fable 5.1: 1M context, 128K output; Haiku 4.5: 200K / 64K | ✅ | platform.claude.com models overview (retrieved Sept 2026) |
| 16 | Anthropic pricing: Opus 5 $5/$25 per MTok; cache hits 0.1× base (0.025× on Fable 5.1-class); cache writes 1.25–2× | ✅ | platform.claude.com pricing page (retrieved Sept 2026) |
| 17 | GPT-4.1: 1M context window (April 2025) | ✅ | openai.com/index/gpt-4-1 (announcement) |
| 18 | GPT-5.4/5.6: 1.05M context, 128K output; >272K input priced at 2× input / 1.5× output | ✅ | developers.openai.com API docs (search-verified snippets) |
| 19 | GPT-5 (Aug 2025) default context ≈ 272K | ⚠ | Third-party trackers only; official page not retrieved |
| 20 | Llama 4 Scout: 17B×16E, 10M context; Maverick: 17B×128E, 1M context; released Apr 5, 2025 | ✅ | Official model card, github.com/meta-llama/llama-models (extracted) |
| 21 | Llama 3.1: up to 128K context (405B dense) | ✅ | arXiv:2407.21783 (Llama 3 Herd) |
| 22 | Term is a 2025-era consolidation; coinage contested/unverifiable | ⚠ | Anchors verified (survey + Anthropic post); no first-use found |
| 23 | June 2025 Karpathy/Lütke popularization claim | ⚠ | Secondary sources only (e.g., labo-llm.fr); primary not found |
| 24 | Anthropic removed >80% of Claude Code's system prompt for Claude 5-generation models | ⚠ | Post title/summary only; not independently verified |
| 25 | Claude Code compaction pattern: compressed context + five most recently accessed files | ✅ | Anthropic context-engineering post (Claude Code section) |
| 26 | Memory tool in public beta since Sonnet 4.5 launch | ✅ | Anthropic post (context-management announcement link) |
| 27 | OpenAI "A Practical Guide to Building Agents" exists | ❌ | Could not verify this pass; not cited in this guide |
| 28 | Claude Code / agents hybrid: standing files up front + glob/grep just-in-time retrieval | ✅ | Anthropic context-engineering post (Claude Code section) |
| 29 | Gemini long-context documentation: 1M-token standard windows with near-perfect retrieval (>99%) | ✅ | Google Cloud / ai.google.dev long-context docs (search-verified snippets) |
| 30 | Anthropic "Building Effective Agents" published Dec 19, 2024; workflows vs. agents framing | ✅ | anthropic.com/engineering/building-effective-agents (search-verified) |
| 31 | Context-engineering survey scope: 166 pages, ~1,400 citations analyzed | ✅ | arXiv:2507.13334 (comments field) |
| 32 | Gemini 2.5 Pro knowledge cutoff January 2025; page updated June 2026 | ✅ | ai.google.dev model page |
| 33 | Chroma context-rot report dated July 2025; codebase open-sourced | ✅ | research.trychroma.com (citation block + GitHub link) |
| 34 | Anthropic pricing FAQ: batch and prompt-caching discounts combine; 1 token ≈ 0.75 English words (≈ 4 chars) | ✅ | platform.claude.com pricing page FAQ (extracted) |
| 35 | Mistral 7B outperforms Llama 2 13B across all evaluated benchmarks (and Llama 1 34B on reasoning/math/code) | ✅ | arXiv:2310.06825 abstract |
| 36 | Chroma context-rot methodology held task complexity constant, varying only input length | ✅ | research.trychroma.com (extracted) |
| 37 | Anthropic recommends sectioned system prompts (XML tags / Markdown headers) over long flat instructions | ✅ | Anthropic context-engineering post |
| 38 | Anthropic sub-agent pattern: explorers return distilled summaries of ~1,000–2,000 tokens | ✅ | Anthropic context-engineering post |
| 39 | Llama 3.1 405B dense architecture: 126 layers, 8 KV heads, head dimension 128 (basis of the §4.3 KV-cache arithmetic) | ✅ | arXiv:2407.21783 (Llama 3 Herd, model architecture table) |

**Audit note.** Every ✅ row above was checked against a page extracted or search-verified during the September 2026 research pass for this guide; quotes in the body text are drawn verbatim from those pages. ⚠ rows are flagged inline where they appear. ❌ rows are recorded rather than papered over. Specs change quarterly — re-run this audit before relying on §4.1 in a design review.

**Method note.** Verification used web search plus direct extraction of primary pages only (arXiv abstracts, vendor engineering blogs, official model documentation and pricing pages, the official Llama 4 model card). No claim was carried into this guide from memory or from secondary aggregators without a primary check; where only secondary sources existed, the claim is marked ⚠ or ❌ and listed in [What Could Not Be Verified](#what-could-not-be-verified). Citation-style links in the body resolve to the pages checked.

The through-line of this guide, and of the sibling clusters it spans, is that model behavior in production is decided before a single token is generated — by the deliberate, budgeted, measurable composition of everything that enters the window. Retrieval, memory, compression, and prompting are not competing architectures; they are instruments for composing the engineered context.
