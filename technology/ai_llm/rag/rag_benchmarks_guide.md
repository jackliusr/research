# RAG Benchmarks — The Public Benchmark and Leaderboard Landscape, and How to Read It

> **Author:** Jack Liu Shurui · **Role:** Solution Architect
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [RAG Evaluation Methodology](rag_evaluation_methodology_guide.md) (every metric definition and the golden-dataset method — this guide is the *dataset landscape*, that guide is the *method*) · [RAG Evaluation Tools Comparison](rag_evaluation_tools_comparison_guide.md) · [Ragas](ragas_guide.md) · [TruLens](trulens_guide.md) · [DeepEval](deepeval_guide.md) · [Vector Databases](vector_databases_guide.md) · [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG vs Long-Context LLMs](rag_vs_long_context_llms_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [Agentic Search vs RAG](../agentic_search_vs_rag_guide.md) · [LLM Evaluation vs Validation](../llm_evaluation_vs_validation_guide.md) · [AI Governance Frameworks](../ai_governance_framework_guide.md)
> **Last Updated:** September 2026

---

## Table of Contents

1. [The Overview, the Honest Framing and the Decoder](#1-the-overview-the-honest-framing-and-the-decoder)
2. [Why Benchmarks Exist and What They Structurally Cannot Do](#2-why-benchmarks-exist-and-what-they-structurally-cannot-do)
3. [The Retrieval Benchmarks](#3-the-retrieval-benchmarks)
4. [The RAG-Specific Benchmarks](#4-the-rag-specific-benchmarks)
5. [The Long-Context and QA Datasets Used as Proxies](#5-the-long-context-and-qa-datasets-used-as-proxies)
6. [The Leaderboards and How They Behave](#6-the-leaderboards-and-how-they-behave)
7. [Contamination and Saturation](#7-contamination-and-saturation)
8. [Why a Retrieval Score Is Not a RAG Score](#8-why-a-retrieval-score-is-not-a-rag-score)
9. [How to Read a Benchmark Claim](#9-how-to-read-a-benchmark-claim)
10. [How to Use a Public Benchmark Responsibly](#10-how-to-use-a-public-benchmark-responsibly)
11. [Build versus Borrow](#11-build-versus-borrow)
12. [The Regulated-Institution Angle](#12-the-regulated-institution-angle)
13. [Worked Example — Three Vendor Pitches at Cymbal Bank](#13-worked-example--three-vendor-pitches-at-cymbal-bank)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, Glossary, Cross-References and Closing Summary](#16-what-could-not-be-verified-glossary-cross-references-and-closing-summary)

---

## 1. The Overview, the Honest Framing and the Decoder

### 1.1 The thesis in one line

**A public benchmark measures a benchmark-shaped task on a fixed corpus, so it can rank approaches and cannot validate your system.**

That is not a complaint about benchmarks. It is a statement about what they are. A benchmark is a *yardstick*, and a yardstick is useful precisely because it does not change while you measure. Your corpus changes. Your users change. Your regulator does not care what the yardstick said. The value of the public landscape is that it lets you *narrow a field cheaply* before you spend the money that actually decides the question — your own goldens, on your own corpus, with your own failure costs attached.

### 1.2 Why the landscape is worth knowing at all

If the answer is always "your own goldens", why learn twenty benchmark names?

- **Because you will be sold to.** An embedding vendor, a reranker vendor and a vector-database vendor will each arrive with a leaderboard placement. You cannot apply the discipline in [§9](#9-how-to-read-a-benchmark-claim) to a claim you cannot read.
- **Because a shortlist has to start somewhere.** Running a bake-off between twenty embedding models on your own data is not affordable. Running it between three that a public benchmark says are in the same band is.
- **Because "the number went up" is the most common false signal in RAG engineering.** Knowing which benchmark produces which number tells you when a vendor's metric is measuring a different object than your product.
- **Because the frontier moves and the vocabulary leaks.** Researchers, analysts and internal stakeholders will say "BEIR", "MTEB", "RGB" and "FRAMES" in the same meeting and mean four different things.

### 1.3 The decoder — the vocabulary you need before the rest of this guide lands

| Term | What it really means | Why the sloppy version misleads |
|------|----------------------|---------------------------------|
| **The benchmark** | A named *evaluation programme*: a task definition + one or more datasets + a scoring rule + (usually) a leaderboard. "BEIR" is a benchmark. | People say "the BEIR corpus" for the 18 heterogeneous datasets inside BEIR. There is no single BEIR corpus. |
| **The dataset** | A concrete collection of instances: queries + documents + relevance labels, or questions + gold answers + supporting evidence. | A dataset is an *artifact with a version*. Two papers on "MS MARCO passages" may not have used the same artifact — see [§8.4](#84-two-corpora-one-name). |
| **The corpus** | The document collection a retrieval task searches. | It is frozen at publication. Your corpus is not, and it is not that one. |
| **The split** | The partition: train / dev / test. | A dev-split number reported without the word "dev" is a red flag. Test splits are held back precisely so this matters. |
| **The track** | The rule-set inside a benchmark: zero-shot vs fine-tuned; retrieval-only vs end-to-end; oracle-context vs open-corpus. | A fine-tuned score and a zero-shot score are not the same number, and a leaderboard may sort them in the same column. |
| **The metric** | The scoring function — nDCG@10, Recall@k, MRR@10, exact match, accuracy, a model-judged faithfulness score. | Metrics are defined in the methodology guide; here it matters only that *different metrics rank systems differently*. |
| **The leaderboard** | A public table where submissions accumulate. | It is a *snapshot service*, not a fact. A placement is true of one day and one submission protocol. |
| **Contamination** | Overlap between benchmark instances and the data a system was trained on. | "Contaminated" is used for everything from a proven train/test overlap to a general suspicion. [§7](#7-contamination-and-saturation) keeps these apart. |
| **Saturation** | The state in which a benchmark no longer separates the systems on it. | Saturation is why new benchmarks keep appearing, and why a benchmark's number stops being informative even though it is still published. |
| **The proxy benchmark** | A benchmark that measures something *adjacent* to RAG — long-context understanding, multi-hop QA, fact verification — and gets borrowed as evidence about RAG. | A QA dataset is a retrieval-and-reasoning task. It is not a RAG benchmark. [§5](#5-the-long-context-and-qa-datasets-used-as-proxies) is explicit about this. |

### 1.4 The boundary this guide declares

This guide owns **the dataset x landscape dimension only** — which public benchmarks exist, what each actually measures, how each was built, what is documented as wrong with them, and how to read a claim built on one.

- Every **metric definition** (retrieval metrics, generation metrics, end-to-end metrics, the three evaluation layers, the failure-mode taxonomy) lives in [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) — §1 Evaluation Dimensions, §2 Retrieval Metrics, §3 Generation Metrics, §4 End-to-End Metrics. This guide cross-references those by name and does **not** restate a single formula.
- The **golden-dataset method** — how to write, size, version, own and re-review your own test set — is that guide's §6, and it is the reason this guide can say "a public benchmark is not your release gate" without arguing about it. This guide owns the *public* dataset landscape; that guide owns the *private* one.
- **Tooling** is owned by [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md), with the deep-dives in [ragas_guide.md](ragas_guide.md), [trulens_guide.md](trulens_guide.md) and [deepeval_guide.md](deepeval_guide.md). Those tools *run* metrics over *your* test set. None of them ships a public benchmark, and none of them is a benchmark.
- **Embedding and framework selection** is owned by [vector_databases_guide.md](vector_databases_guide.md) and [rag_frameworks_comparison_guide.md](rag_frameworks_comparison_guide.md). Where BEIR and MTEB appear there as *selection evidence*, this guide cites them rather than re-deriving their construction.
- **The long-context comparison** is owned by [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md) — including LongBench, RULER, HELMET and InfiniteBench. This guide covers them only as *proxy benchmarks* ([§5](#5-the-long-context-and-qa-datasets-used-as-proxies)).
- **The techniques** that benchmarks measure are owned by [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) and [agentic_search_vs_rag_guide.md](../agentic_search_vs_rag_guide.md). Where those mention HotpotQA or MultiHop-RAG as an evaluation vehicle, this guide is the reference for what those datasets actually contain.

The gap this fills is real and was checked, not assumed: the repo's evaluation guides teach you to build your own test set and never catalogue the public ones. This guide is the catalogue, with the caveats attached to each entry.

---

## 2. Why Benchmarks Exist and What They Structurally Cannot Do

### 2.1 What a benchmark is for

Three legitimate purposes, all of which are worth defending:

1. **Comparable numbers across systems.** Without a shared task and a shared corpus, "our retriever is better" is an unfalsifiable sentence. A benchmark makes a claim checkable by a stranger.
2. **A moving research frontier.** Progress needs a scoreboard that stays still long enough for the next result to be measured against it. That is what BEIR and MTEB are for embeddings and retrieval, and it is why their maintainers publish engineering papers about keeping them reproducible at all (MTEB's maintainers describe continuous-integration pipelines that validate dataset integrity and assess result generalisability — [arXiv:2506.21182](https://arxiv.org/abs/2506.21182), Chung et al., 26 June 2025).
3. **An adoption signal.** A leaderboard placement is not a proof, but it is a *filter*. Practitioners use it to decide which five of fifty embedding models deserve their own time. [§6](#6-the-leaderboards-and-how-they-behave) treats this dynamic explicitly, because it is the mechanism by which benchmark numbers turn into procurement decisions.

### 2.2 The four structural limits

Every critique later in this guide is a corollary of four facts about how benchmarks are built.

| Limit | What is fixed | Consequence for you |
|-------|---------------|---------------------|
| **The task is fixed** | One task definition, chosen by the benchmark's authors, scoring one thing (retrieve, or answer, or judge). | Your workload is a *mixture* of tasks nobody specified. A benchmark cannot tell you how a system behaves on the mixture. |
| **The corpus is fixed** | One document collection, frozen at publication, with relevance labels produced once. | Your corpus has a different vocabulary, a different update cadence, a different messiness — and no labels. |
| **The user is absent** | There is no query log, no reformulation, no "close enough", no user who asked the wrong question. | Benchmarks score *queries as written*. Production systems fail on queries as *meant*. |
| **The deployment is absent** | No latency budget, no cost ceiling, no index-refresh window, no failure consequence, no fallback path. | A benchmark cannot express that an irrelevant answer is cheap in a sandbox and expensive in a regulated process. |

### 2.3 The corollary that governs the whole guide

Because all four are fixed, a benchmark number is a statement about **a system in a controlled condition**, and every deployment question is a question about **your conditions**. That is why the correct use of a public benchmark is to *narrow* and *sanity-check*, never to *accept*. The rest of this guide is that sentence, applied.

---

## 3. The Retrieval Benchmarks

### 3.1 The map before the detail

| Benchmark | Paper (verified at source) | Year | What it scores | Who it is for |
|-----------|---------------------------|------|----------------|---------------|
| **MS MARCO** | *MS MARCO: A Human Generated MAchine Reading COmprehension Dataset* — Bajaj et al., [arXiv:1611.09268](https://arxiv.org/abs/1611.09268) | 2016 | Passage/document ranking and reading comprehension over real Bing queries | Retrieval models trained on large data; the substrate of the TREC DL track |
| **TREC Deep Learning track** | *Overview of the TREC 2020 deep learning track* — Craswell et al., [arXiv:2102.07662](https://arxiv.org/abs/2102.07662) | 2021 (track runs 2019–) | Ad-hoc document and passage ranking in the large-data regime, with deep relevance judgments | The evaluation-track counterpart to MS MARCO's leaderboards |
| **BEIR** | *BEIR: A Heterogenous Benchmark for Zero-shot Evaluation of Information Retrieval Models* — Thakur et al., [arXiv:2104.08663](https://arxiv.org/abs/2104.08663) | 2021 | Zero-shot, out-of-distribution retrieval across heterogeneous domains and query types | Choosing a retriever you cannot fine-tune on the target domain |
| **MTEB** | *MTEB: Massive Text Embedding Benchmark* — Muennighoff et al., [arXiv:2210.07316](https://arxiv.org/abs/2210.07316) | 2022 | Embedding quality across a task taxonomy, not retrieval alone | Choosing an embedding model for a mixed workload |
| **BRIGHT** | *BRIGHT: A Realistic and Challenging Benchmark for Reasoning-Intensive Retrieval* — Su et al., [arXiv:2407.12883](https://arxiv.org/abs/2407.12883) | 2024 | Retrieval where the query needs reasoning, not lexical or semantic overlap | Testing whether a retriever generalises past relevance-style queries |
| **TREC RAG track** | project site [trec-rag.github.io](https://trec-rag.github.io/) (2024 page: [trec-rag.github.io/trec24](https://trec-rag.github.io/trec24/)) | 2024– | End-to-end retrieval **and** grounded generation with attribution, assessed by human judges | Teams that need the full pipeline measured, not a component |

Read the table as a taxonomy, not a ranking. These six do not compete for the same slot: the first three score a *retriever*, MTEB scores an *embedding model across tasks*, BRIGHT stress-tests *query difficulty*, and the TREC RAG track scores a *pipeline plus its citations*.

### 3.2 MS MARCO — the substrate

The abstract states the construction facts plainly: **1,010,916 anonymised questions sampled from Bing's search query logs**, each with a human-generated answer and **182,669** human-rewritten answers, over **8,841,823 passages** extracted from **3,563,535 web documents** retrieved by Bing. The paper proposes three tasks: predict answerability and extract/synthesise an answer, generate a well-formed answer, and **rank a set of retrieved passages**. Because the questions are derived from real user search queries, they are not the clean, well-formed questions that later QA datasets use.

The project's own page (read 25 September 2026) adds the operational detail that matters for reading leaderboards:

- The **passage ranking** task is scored as *re-ranking* (a candidate top-1000 from BM25) or *full ranking* (produce a candidate top-1000 from the 8.8M-passage corpus). The **document ranking** task has the same two modes over 3.2M documents.
- Relevance labels are **derived from which passage was marked as containing the answer** in the question-answering dataset — an important construction detail: the labels are inherited from an answer-annotation process, not produced by a dedicated relevance-assessment exercise.
- The page records that both leaderboards were subsequently **retired** (passage retrieval: retired, 26 October 2018 – 1 January 2023; document retrieval: retired, 11 August 2020 – 1 January 2023), and that the earlier NLGEN and QnA leaderboards closed on 23 October 2020. **This is a first-class fact about leaderboards in general:** they retire, and a claim citing a retired leaderboard can no longer be checked against a live table.
- The submission instructions ask for an eval file, a dev file, team and institution names, model code (recommended), model information and paper information, and state that the maintainers discourage too many submissions from the same group in a short period "to avoid P-hacking". That is the shape of a self-reported leaderboard: the entrant supplies the artifact, the maintainer supplies the scoring.

### 3.3 The TREC Deep Learning track — evaluation with judges

TREC DL studies ad-hoc search *in the large data regime*: the training set is enormous, so the track's contribution is the **test side** — a small number of test queries with far more thorough relevance judging than a leaderboard's inherited labels, scored TREC-style as a single-shot run. The 2020 overview (Craswell et al., [arXiv:2102.07662](https://arxiv.org/abs/2102.07662), first posted 15 February 2021) reports document and passage tasks with hundreds of thousands of human-labelled *training* queries and continues to find BERT-style rankers ahead in the large-data regime.

Two companion papers are unusually useful for reading *any* retrieval number:

- *TREC Deep Learning Track: Reusable Test Collections in the Large Data Regime* ([arXiv:2104.09399](https://arxiv.org/abs/2104.09399), Craswell et al., 19 April 2021) documents the collections in one place and — critically — names **iteration and selection bias** as a risk when a dataset is reused, and prescribes best practices for writing a paper on TREC DL data without overfitting to it.
- *MS MARCO: Benchmarking Ranking Models in the Large-Data Regime* ([arXiv:2105.04021](https://arxiv.org/abs/2105.04021), Craswell et al., 9 May 2021) argues that the goal of an evaluation effort is not to identify which run scores highest but to move the field, and shows that **the design of the evaluation effort itself can encourage or discourage certain outcomes**, raising questions of internal and external validity. If you read one paper from this section, read this one: it is benchmark maintainers auditing their own benchmark in public.

### 3.4 BEIR — zero-shot heterogeneity

BEIR's claim is narrow and important: existing neural IR results were measured in *homogeneous, narrow* settings, so their out-of-distribution generalisation was unknown. BEIR selects **18 publicly available datasets** from diverse tasks and domains and evaluates **10** retrieval systems spanning lexical, sparse, dense, late-interaction and re-ranking architectures. Both construction facts are in the abstract.

The reported findings are the part vendors quote — and the part they quote selectively:

- **BM25 is a robust baseline.** The sparse lexical method that predates neural retrieval remains the thing everything must beat.
- **Re-ranking and late-interaction models achieve the best average zero-shot performance, at high computational cost.**
- **Dense and sparse-retrieval models are cheaper but often underperform**, which the authors read as "considerable room for improvement in their generalization capabilities".

The known criticisms, each from its own source:

- **Provenance and comparability.** *Resources for Brewing BEIR* ([arXiv:2306.07471](https://arxiv.org/abs/2306.07471), Kamalloo et al., 13 June 2023) states two problems directly: the sophistication of modern neural methods and the software infrastructure create barriers to entry, and **there was no single authoritative nexus for reporting results**, which caused difficulty comparing methods. Their remedy was reproducible reference implementations for the two main approach classes and an **official self-service BEIR leaderboard**. A benchmark without a maintained leaderboard accumulates incomparable numbers — that is a general lesson, not a BEIR-only one.
- **Judgment quality on at least one subset.** *Systematic Evaluation of Neural Retrieval Models on the Touché 2020 Argument Retrieval Subset of BEIR* ([arXiv:2407.07790](https://arxiv.org/abs/2407.07790), Thakur et al., 10 July 2024) reproduces the famous "neural models lose to BM25 on argument retrieval" result and digs in. It finds an **inherent bias of neural models toward retrieving short passages** from that data, and that **quite a few of the neural models' results are unjudged** — missing relevance judgments that complicate fair comparison. After denoising (excluding passages under 20 words) and adding post-hoc judgments, the comparison changes. The lesson generalises: *an average over 18 datasets can hide a subset where the labels, not the model, drive the difference.*
- **Language coverage.** BEIR is English. The derivatives are explicit about the cost: BEIR-NL ([arXiv:2412.08329](https://arxiv.org/abs/2412.08329), Banar et al., 11 December 2024) reports that BM25 remains competitive after translation and that **back-translating a subset to English produces a performance drop for both dense and lexical methods**, "indicating the limitations of translation for creating benchmarks". Hindi-BEIR ([arXiv:2408.09437](https://arxiv.org/abs/2408.09437), Acharya et al., 18 August 2024) and BEIR-PL ([arXiv:2305.19840](https://arxiv.org/abs/2305.19840), Wojtasik et al., 31 May 2023) exist for the same reason. **If your corpus is not English, the English BEIR number is a weaker signal than you think** — and translation is not a free substitute.

### 3.5 MTEB — the multi-task embedding benchmark

MTEB's stated motivation is that text embeddings were being evaluated on a small set of datasets from a single task, leaving it unclear whether a model strong on semantic textual similarity would be strong on clustering or reranking. MTEB addresses this with a **task taxonomy**: the abstract reports **8 embedding tasks** covering **58 datasets** and **112 languages**, with **33 models** benchmarked. Two findings matter for how you read any embedding leaderboard:

- **No particular text embedding method dominates across all tasks** — the authors' own conclusion is that the field has not converged on a universal embedding method. A single "best embedding model" headline is therefore task-conditional by construction.
- The benchmark ships a **public leaderboard**, and its maintainers later published on the engineering needed to keep it honest: *Maintaining MTEB* ([arXiv:2506.21182](https://arxiv.org/abs/2506.21182), Chung et al., 26 June 2025) describes validation of dataset integrity, automated test execution, assessment of whether results generalise, and the handling of community contributions and new tasks. **A leaderboard that keeps growing its task set is a leaderboard whose scores are not comparable across time.**

`⚠` The MTEB leaderboard itself is a dynamic web application (the Hugging Face Space at `huggingface.co/spaces/mteb/leaderboard`). When read for this guide on 25 September 2026 it returned no static table content to extract. This guide therefore describes how the leaderboard behaves and reports **no current placement from it** (`❌` no table read).

### 3.6 BRIGHT — the reasoning-intensive retriever test

BRIGHT exists because "existing retrieval benchmarks primarily consist of information-seeking queries where keyword or semantic-based retrieval is usually sufficient". Its construction: **1,384 real-world queries** spanning economics, psychology, mathematics and coding, "drawn from naturally occurring and carefully curated human data". The finding that makes it useful here is a documented **non-transfer of benchmark rank**: the paper reports that the model then leading the MTEB leaderboard on retrieval scored **59.0 nDCG@10 on its home turf and 18.3 nDCG@10 on BRIGHT** (figures as reported in the paper, July 2024). The authors also report that explicit reasoning about the query improves retrieval by up to 12.2 points. Keep the date on that comparison: it is a 2024 snapshot from one paper, and it is included here as evidence about *benchmark non-transfer*, not as a current ranking.

### 3.7 The TREC RAG track — the academic end-to-end track

The TREC RAG track is the closest thing to a *peer-reviewed, human-judged* end-to-end RAG benchmark, and its structure is worth stating because it shows what proper end-to-end assessment costs.

**TREC 2024** (structure read at [trec-rag.github.io/trec24](https://trec-rag.github.io/trec24/) on 25 September 2026) ran **three tasks** over the MS MARCO Segment v2.1 collection:

1. **(R) Retrieval** — rank and retrieve the most relevant segments for the given topics.
2. **(AG) Augmented Generation** — generate RAG answers with attributions, using the top-k segments from the baseline retrieval system.
3. **(RAG) Retrieval-Augmented** — generate RAG answers with attributions using the participant's own retriever and chunking, provided chunks map back to MS MARCO Segment v2.1 for reproducibility and evaluation.

The 2024 evaluation methodology was published as a pipeline: gather answers → **evaluate whether the citations support the sentences** → pool and *nuggetise* the retrieved sentences → assign nuggets to each answer sentence → aggregate with linguistic features such as fluency and coherence. **301 test topics** were released; the organizer list was Ronak Pradeep, Nandan Thakur and Jimmy Lin (University of Waterloo) and Nick Craswell (Microsoft). **Note the author concentration:** the same group that maintains BEIR and TREC DL also ran this track — that is a neutral fact about who builds retrieval benchmarks, worth knowing when you read a chain of mutually-citing results.

**The 2026 iteration** (project site read 25 September 2026) shows how a track evolves and how a *fixed corpus* is not actually permanent: two tasks remain — **Retrieval** and **Retrieval-Augmented Generation** over a *narrative* input — while the collection changed from MS MARCO v2.1 to **NVIDIA's ClimbMix-400b**. Runs are submitted through NIST's Evalbase (a 2026 submission deadline was stated as 8 August). The materials now include RAG25 nuggets, UMBRELA qrels, ResearchRubrics, and an automated end-to-end evaluation toolkit, **RAGDoll** (hosted under the castorini organisation).

`⚠` The track overview papers for TREC RAG 2024 and 2025 were not verified by this guide (an arXiv full-text query for the track returned no matching entry). The structure above is cited to the track's own site, read 25 September 2026, and is labelled a snapshot of the track as it stands — not of its results. `❌` No TREC RAG ranking or per-team system result is reported anywhere in this guide.

---

## 4. The RAG-Specific Benchmarks

A retrieval benchmark scores a component. The benchmarks in this section attempt to score a *RAG-shaped thing* — and each one defines "RAG-shaped" differently. The differences are the content of this section.

| Benchmark | Paper (verified at source) | Year | What it measures | How instances were produced |
|-----------|---------------------------|------|------------------|-----------------------------|
| **RGB** | *Benchmarking Large Language Models in Retrieval-Augmented Generation* — Chen et al., [arXiv:2309.01431](https://arxiv.org/abs/2309.01431) | 2023 | Four RAG abilities: noise robustness, negative rejection, information integration, counterfactual robustness | A purpose-built corpus in English and Chinese, split into four testbeds by ability |
| **MultiHop-RAG** | *MultiHop-RAG: Benchmarking Retrieval-Augmented Generation for Multi-Hop Queries* — Tang et al., [arXiv:2401.15391](https://arxiv.org/abs/2401.15391) | 2024 | Retrieval **and** reasoning over multiple pieces of evidence | Knowledge base built from an English news-article dataset; multi-hop queries with gold answers and supporting evidence |
| **CRUD-RAG** | *CRUD-RAG: A Comprehensive Chinese Benchmark for Retrieval-Augmented Generation of Large Language Models* — Lyu et al., [arXiv:2401.17043](https://arxiv.org/abs/2401.17043) | 2024 | All RAG components across four application types: **C**reate, **R**ead, **U**pdate, **D**elete | Chinese-language corpus and tasks spanning content generation and knowledge-intensive QA |
| **RAGBench** | *RAGBench: Explainable Benchmark for Retrieval-Augmented Generation Systems* — Friel et al., [arXiv:2407.11005](https://arxiv.org/abs/2407.11005) | 2024 | Five industry domains and multiple RAG task types, plus the **TRACe** explainable evaluation framework | **100k examples** sourced from industry corpora such as user manuals |
| **FRAMES** | *Fact, Fetch, and Reason: A Unified Evaluation of Retrieval-Augmented Generation* — Krishna et al., [arXiv:2409.12941](https://arxiv.org/abs/2409.12941) | 2024 | Factuality, retrieval and reasoning **in one end-to-end number** | Challenging multi-hop questions requiring integration of information from multiple sources |
| **ARES** | *ARES: An Automated Evaluation Framework for Retrieval-Augmented Generation Systems* — Saad-Falcon et al., [arXiv:2311.09476](https://arxiv.org/abs/2311.09476) | 2023 | **An evaluator, not a dataset:** scores RAG systems on context relevance, answer faithfulness and answer relevance | Synthetic training data plus a small human-annotated set for prediction-powered inference |

### 4.1 RGB — the diagnostic benchmark

RGB is the benchmark most often cited in RAG slide decks, and its abstract is unusually specific about what it does and does not establish. It analyses LLM performance on **four fundamental abilities required for RAG**:

1. **Noise robustness** — can the model still answer when irrelevant documents are retrieved?
2. **Negative rejection** — can the model decline to answer when no retrieved document supports an answer?
3. **Information integration** — can the model combine evidence from multiple documents?
4. **Counterfactual robustness** — can the model resist *false* information in the retrieved context?

The construction is a **new corpus for RAG evaluation in both English and Chinese**, divided into **four separate testbeds**, one per ability. Six representative LLMs were evaluated. The headline finding: models "exhibit a certain degree of noise robustness" but **struggle significantly on negative rejection, information integration, and false information**.

`⚠` The abstract does not state the number of instances, the corpus size, or the document count, and this guide did not read the full paper — so **no counts are given here**. That is deliberate: an invented corpus size is worse than an absent one.
`❌` No RGB score for any model is reported in this guide.

Note the shape of what RGB measures: it varies the *context* while holding a task fixed. That makes it a diagnostic instrument for one part of the pipeline — see [§8](#8-why-a-retrieval-score-is-not-a-rag-score) for why a diagnostic result is not a system result.

### 4.2 MultiHop-RAG — retrieval plus reasoning over an evidence set

MultiHop-RAG's motivation is a genuine gap: at publication, the authors state that **no existing RAG benchmarking dataset focused on multi-hop queries**. Its construction, per the abstract: a **knowledge base** plus a large collection of **multi-hop queries**, their **ground-truth answers**, and the **associated supporting evidence**, built using an **English news article dataset** as the underlying RAG knowledge base. Two experiments demonstrate its use: one comparing embedding models for evidence retrieval on multi-hop queries, and one evaluating GPT-4, PaLM and Llama2-70B on reasoning and answering given the evidence. Both found existing RAG methods "perform unsatisfactorily" at retrieving and answering multi-hop queries.

The distinction that matters: MultiHop-RAG supplies **supporting evidence** for each query, so it can score retrieval and generation separately and jointly. A benchmark without evidence labels can only score the final answer.

`⚠` The abstract states no query count, no knowledge-base size and no article count; none is asserted here.

### 4.3 CRUD-RAG — the non-QA RAG benchmark

CRUD-RAG is worth including precisely because its criticism of the field is the same one this guide makes. Its abstract states the problem directly: existing benchmarks **predominantly assess question-answering applications**, overlooking the broader range of situations where RAG helps; and **they only evaluate the LLM component of the RAG pipeline**, neglecting the retrieval component and the external knowledge database. Its remedy is a **Chinese-language** benchmark organised by application type:

- **Create** — generating original, varied content.
- **Read** — answering intricate questions in knowledge-intensive situations.
- **Update** — revising or correcting inaccuracies and inconsistencies in pre-existing text.
- **Delete** — the fourth application type in the benchmark's taxonomy. `⚠` The abstract read for this guide truncates mid-sentence at "Delete refers to…", so **this guide does not paraphrase what it did not read**; the naming comes from the paper's own CRUD framing (Create, Read, Update, Delete).

It then evaluates **all components** of a RAG system across these scenarios, which is the design choice that distinguishes it from QA-only benchmarks.

`⚠` The abstract gives no instance counts or corpus size for CRUD-RAG; none is asserted here. The Chinese-language focus means a CRUD-RAG number is evidence about Chinese-language behaviour, not a general one.

### 4.4 RAGBench and TRACe — the industry-corpus benchmark

RAGBench is the most "enterprise-shaped" of the current benchmarks, and its construction facts are stated in the abstract: it is described as the **first comprehensive, large-scale RAG benchmark dataset of 100k examples**, covering **five unique industry-specific domains** and various RAG task types, with examples **sourced from industry corpora such as user manuals** and released publicly on Hugging Face (the `rungalileo/ragbench` dataset). Alongside the data it formalises **TRACe**, described as a set of **explainable and actionable** evaluation metrics applicable across RAG domains — the point being that a score should tell you *which* of the four TRACe dimensions failed, not just that something did.

Its headline experimental finding is a direct caution about evaluator choice: the authors report that **LLM-based RAG evaluation methods struggle to compete with a fine-tuned evaluator** on their labels. In other words, the *judge* is a variable, not a constant. That is why [§9](#9-how-to-read-a-benchmark-claim) insists that a claim state which evaluator produced the number.

The "user manuals" detail is the reason RAGBench appears in this guide's enterprise discussion: a manual corpus is closer to a bank's procedure documents than Wikipedia or English news is — and it is still not your corpus.

### 4.5 FRAMES — end-to-end factuality with a single number

FRAMES expands to **Factuality, Retrieval, And reasoning MEasurement Set**, and its abstract states the design intent explicitly: previous work provided datasets and benchmarks that evaluate these abilities **in isolation**, whereas FRAMES offers a **unified framework** giving a clearer picture of LLM performance in **end-to-end RAG scenarios**. Instances are **challenging multi-hop questions requiring the integration of information from multiple sources**.

The reported baselines are the useful part for reading claims, because they quantify the gap between "model alone" and "model plus retrieval": state-of-the-art LLMs achieve **0.40 accuracy with no retrieval**, rising to **0.66 with the authors' proposed multi-step retrieval pipeline** — a stated improvement of more than 50%. Two things follow: the retrieval pipeline is doing a lot of work, and **a FRAMES number is jointly a claim about a model and a pipeline**, not about either alone.

`⚠` The abstract states no question count, no corpus size and no number of sources for FRAMES; none is asserted here. `⚠` The 0.40 and 0.66 figures are the paper's own reported baselines from 2024, not current state-of-the-art, and are labelled as the paper's results.

### 4.6 ARES — an evaluator, kept firmly on the evaluator side of the line

ARES is in this section only because it is so often mistaken for a dataset benchmark. It is not. Its abstract says what it is: **an Automated RAG Evaluation System** that evaluates RAG systems along three dimensions — **context relevance, answer faithfulness, answer relevance**. Its mechanism:

- It **creates its own synthetic training data** and **fine-tunes lightweight LM judges** to assess the quality of individual RAG components.
- To mitigate prediction errors it uses **prediction-powered inference (PPI)** with a **small set of human-annotated datapoints**.
- It is validated across **eight different knowledge-intensive tasks in KILT, SuperGLUE and AIS**, using only a few hundred human annotations during evaluation.
- Its judges remain effective across **domain shifts** — accurate even after changing the type of queries and/or documents in the evaluated systems.

KILT itself (*KILT: a Benchmark for Knowledge Intensive Language Tasks*, Petroni et al., [arXiv:2009.02252](https://arxiv.org/abs/2009.02252), 4 September 2020) is the dataset family ARES evaluates against; its design point is that **all tasks are grounded in the same snapshot of Wikipedia** to reduce engineering turnaround and enable task-agnostic memory architectures. That is a *corpus-fixing* choice, with exactly the limitation [§2](#2-why-benchmarks-exist-and-what-they-structurally-cannot-do) describes.

**The line, stated once:** a dataset benchmark gives you *instances to score against*; an evaluator framework gives you *a way to score*. ARES is the second. Choosing ARES is a tooling decision owned by [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md); choosing what to score it against is this guide's subject.

### 4.7 The long tail — newer RAG benchmarks, each verified at its own abstract

The RAG benchmark population is still growing, and the newer entries are explicit about what the older ones missed. Three verified examples:

- **RAGTruth** (*A Hallucination Corpus for Developing Trustworthy Retrieval-Augmented Language Models*, Niu et al., [arXiv:2401.00396](https://arxiv.org/abs/2401.00396), 31 December 2023) — **nearly 18,000 naturally generated responses** from diverse LLMs using RAG, **manually annotated at the case and word level** with hallucination intensity, across domains and tasks. It is a hallucination-annotation corpus: it scores *unsupported claims*, and its authors report that a fine-tuned relatively small LLM can reach competitive hallucination-detection performance against prompt-based GPT-4 approaches. This is the annotation layer that generation-side metrics need and that most RAG benchmarks lack.
- **T²-RAGBench** (*Text-and-Table Benchmark for Evaluating Retrieval-Augmented Generation*, Strich et al., [arXiv:2506.12071](https://arxiv.org/abs/2506.12071), 4 June 2025) — **23,088 question-context-answer triples** over real-world text-and-table data, built by transforming datasets into a **context-independent** format (validated by experts as 91.3% context-independent) so that answers do not depend on which context was retrieved. Its finding is directly relevant to component-vs-pipeline reading: it reports **Hybrid BM25** — dense plus sparse — as the most effective approach for text-and-table data, and notes that results remain challenging for state-of-the-art LLMs and RAG methods.
- **LIT-RAGBench** (*Benchmarking Generator Capabilities of Large Language Models in Retrieval-Augmented Generation*, Itai et al., [arXiv:2603.06198](https://arxiv.org/abs/2603.06198), 6 March 2026) — a **generator-focused** benchmark defining five categories (Integration, Reasoning, Logic, Table, Abstention) over **114 human-constructed Japanese questions** plus a machine-translated, human-curated English version, using **fictional entities and scenarios** and LLM-as-a-judge scoring. Two construction choices are instructive: deliberately fictional entities reduce the chance of the answers existing in model weights — a partial contamination defence — and abstention is a scored category, which is what RGB's "negative rejection" argued for two years earlier.

Read this subsection as evidence of [§7](#7-contamination-and-saturation)'s saturation argument: each of these benchmarks justifies itself by naming a gap in an earlier one. That is the field's normal mechanism for producing new benchmarks, and it is why a benchmark's existence is never evidence that the earlier benchmark was wrong.

---

## 5. The Long-Context and QA Datasets Used as Proxies

### 5.1 The honest framing first

**A QA dataset is a retrieval-and-reasoning task, not a RAG benchmark.** When a paper reports "we evaluated on HotpotQA", it is reporting a score on a fixed question set with a fixed, usually small, supplied context set — not on a deployed retrieval-augmented system with your corpus, your chunking, your index and your refusal policy. When a long-context benchmark reports "our model handles 128K tokens", it is reporting a *context-window* result, not a retrieval result.

This does not make the numbers worthless — it makes them **proxy evidence**, and the honest question is always *proxy for what*. The four long-context and multi-hop families below are the standard proxies, and the differences between them are precisely the differences that matter:

- A **supplied-context** benchmark (HotpotQA given its paragraphs; LongBench with the document packed into the prompt) measures the **reader**.
- An **open-corpus** benchmark (BEIR; TREC RAG) measures the **retriever**, and requires an index.
- A **mixed** benchmark (FRAMES; the TREC RAG track) measures both and lets you attribute the failure.

The long-context comparison itself — whether to put the whole corpus in a 1M-token window instead of retrieving — is owned by [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md), which discusses LongBench, RULER and HELMET in that framing. This section covers them only as *things people cite as RAG evidence*.

### 5.2 LongBench and LongBench v2

**LongBench** (Bai et al., [arXiv:2308.14508](https://arxiv.org/abs/2308.14508), 28 August 2023) is described as the **first bilingual, multi-task benchmark for long-context understanding**: **21 datasets** across **6 task categories** in English and Chinese, with an average length of **6,711 words** (English) and **13,386 characters** (Chinese). The categories are single-doc QA, multi-doc QA, summarisation, few-shot learning, synthetic tasks and code completion, all normalised to a unified format for automatic evaluation — a practical design choice that is why LongBench is widely used as a smoke test.

**LongBench v2** (Bai et al., [arXiv:2412.15204](https://arxiv.org/abs/2412.15204), 19 December 2024) is the response to v1's weaknesses: **503 challenging multiple-choice questions** with **contexts from 8k to 2M words**, across six major task categories (single-document QA, multi-document QA, long in-context learning, long-dialogue history understanding, code repository understanding, long structured data understanding). Construction facts worth noting: data were collected **from nearly 100 highly educated individuals** with diverse professional backgrounds, with both automated and manual review processes to maintain quality and difficulty. The reported difficulty calibration is the key figure: **human experts achieved only 53.7% accuracy under a 15-minute time constraint**, and the best model answering directly achieved **50.1%**.

`⚠` The 53.7% / 50.1% figures, and the paper's statement that a reasoning-mode model (o1-preview) reached 57.7%, are the **paper's December 2024 snapshot**, quoted as the paper's own results. They are not a current leaderboard state and are not presented as one.

The reason LongBench matters for a RAG practitioner: **long-context scores say nothing about retrieval quality.** A model that scores well on a multi-document QA category given all documents in the prompt may still fail with a retrieval frontier that returns the wrong five chunks. LongBench measures the reader, not the retriever.

### 5.3 RULER and HELMET — the synthetic-task critique

Both of these exist because the most popular long-context test in the industry is a weak one, and both say so in their abstracts.

**RULER** (Hsieh et al., [arXiv:2404.06654](https://arxiv.org/abs/2404.06654), 9 April 2024) opens by naming the problem: the needle-in-a-haystack test "is indicative of only a superficial form of long-context understanding". RULER is a **synthetic benchmark with flexible configurations for customised sequence length and task complexity**, expanding the vanilla NIAH test to cover **diverse types and quantities of needles** and adding **multi-hop tracing** and **aggregation** categories. It evaluates **17 long-context LMs** across **13 representative tasks**. The finding is the one to remember: **despite nearly perfect accuracy on vanilla NIAH, almost all models show large performance drops as context length increases**, and while the models claim context sizes of 32K tokens or more, **only about half maintained satisfactory performance at 32K**.

**HELMET** (Yen et al., [arXiv:2410.02694](https://arxiv.org/abs/2410.02694), 3 October 2024) investigates *why* long-context evaluation is noisy and names four causes: **limited coverage of applications, insufficient context lengths, unreliable metrics, and incompatibility with base models**. Its remedy is **seven application-centric categories**, controllable lengths **up to 128K tokens**, **model-based evaluation** for reliable metrics, and few-shot prompting so that base models can be evaluated robustly. Across a study of **59 long-context LMs** it reports three findings that a benchmark reader should internalise generally:

1. **Synthetic tasks like NIAH do not reliably predict downstream performance** — the headline result, and the reason "our model aces needle-in-a-haystack" is not evidence of anything useful.
2. **The diverse categories exhibit distinct trends and low correlations with each other** — a single aggregate long-context score is an average over things that do not move together.
3. Most LCLMs reach perfect NIAH scores (the abstract's third finding continues beyond the excerpt read here; the NIAH-saturation part is explicit).

Point 2 is the long-context instance of MTEB's "no method dominates across all tasks" and of RGB's four-abilities split. **Aggregate scores hide task-conditionality.** It is the same finding in three different benchmark families, which is about as close to a consensus as this literature gets.

### 5.4 The classic QA datasets RAG papers borrow

| Dataset | Paper (verified at source) | Year | Construction facts stated in the abstract |
|---------|---------------------------|------|-------------------------------------------|
| **Natural Questions** | *Natural Questions: A Benchmark for Question Answering Research* — Kwiatkowski et al., TACL 7:452–466, [ACL Anthology Q19-1026](https://aclanthology.org/Q19-1026/) | 2019 | Questions are **real anonymised aggregated queries issued to the Google search engine**; an annotator sees the question plus a Wikipedia page from the top-5 search results and annotates a **long answer** (typically a paragraph) and a **short answer** if present, or marks null. Public release: **307,373** training examples with single annotations, **7,830** with 5-way annotations for development, and **7,842** 5-way-annotated sequestered test examples. |
| **TriviaQA** | *TriviaQA: A Large Scale Distantly Supervised Challenge Dataset for Reading Comprehension* — Joshi et al., [arXiv:1705.03551](https://arxiv.org/abs/1705.03551) | 2017 | Over **650K question-answer-evidence triples**; **95K** question-answer pairs authored by trivia enthusiasts, with evidence documents gathered **independently**, **six per question on average**. Reported as having relatively complex compositional questions, high lexical variability, and more cross-sentence reasoning than comparable sets; baselines of 23% and 40% against an 80% human figure. |
| **HotpotQA** | *HotpotQA: A Dataset for Diverse, Explainable Multi-hop Question Answering* — Yang et al., [arXiv:1809.09600](https://arxiv.org/abs/1809.09600) | 2018 | **113k Wikipedia-based question-answer pairs** requiring reasoning over **multiple supporting documents**, with **sentence-level supporting facts** for supervision and explanation, plus a new type of **factoid comparison** question. |
| **MuSiQue** | *MuSiQue: Multihop Questions via Single-hop Question Composition* — Trivedi et al., [arXiv:2108.00573](https://arxiv.org/abs/2108.00573) | 2021 | Built **bottom-up** by selecting composable pairs of single-hop questions where one reasoning step critically relies on another, with filters targeting connected reasoning. **MuSiQue-Ans** has **25K 2–4 hop questions**; reported as **3x more difficult** than existing datasets by human-machine gap, with a **30-point F1 drop** for a single-hop model. **MuSiQue-Full** adds **unanswerable contrast questions**. |
| **2WikiMultiHopQA** | *Constructing A Multi-hop QA Dataset for Comprehensive Evaluation of Reasoning Steps* — Ho et al., [arXiv:2011.01060](https://arxiv.org/abs/2011.01060) | 2020 | Combines **structured and unstructured** data; adds **evidence information containing a reasoning path** for multi-hop questions, so predictions can be explained and reasoning skills evaluated. Questions are generated with a carefully designed pipeline and templates that **guarantee the multi-hop steps**, plus **Wikidata logical rules** to create natural questions that still require multi-hop reasoning. |
| **FEVER** | *FEVER: a large-scale dataset for Fact Extraction and VERification* — Thorne et al., [arXiv:1803.05355](https://arxiv.org/abs/1803.05355) | 2018 | **185,445 claims** generated by **altering sentences extracted from Wikipedia**, verified by annotators **without knowledge of the sentence they were derived from**; labels of Supported / Refuted / NotEnoughInfo at inter-annotator agreement of **0.6841 Fleiss κ**; for the first two classes, annotators recorded the **sentences forming the necessary evidence**. Reported baselines: best accuracy **31.87%** with correct evidence and **50.91%** ignoring evidence. |

### 5.5 What changes when you borrow a QA dataset as RAG evidence

Four differences, each with a consequence:

- **Context is usually supplied, not retrieved.** HotpotQA with its gold paragraphs in the prompt measures the reader. If you run it *open-corpus*, you are running a different benchmark with the same name — and both numbers will appear in vendor decks.
- **Refusal is usually unscored.** FEVER's NotEnoughInfo and MuSiQue's unanswerable contrasts are the exceptions; most QA sets assume an answer exists, so a system's ability to *decline* — RGB's "negative rejection" — is invisible. This is exactly the ability [§4.1](#41-rgb--the-diagnostic-benchmark) was built to expose.
- **Labels are inherited from a construction process, not from a relevance assessment.** MS MARCO's relevance labels derive from answer-annotated passages; FEVER's claims derive from altered Wikipedia sentences. Both are reasonable; neither is the same thing as a judge asking "is this document relevant to this query?".
- **The corpus is small enough to be memorised, and old enough to be in the weights.** Every dataset in §5.4 predates the model generation being evaluated on it — HotpotQA 2018, TriviaQA 2017, FEVER 2018, NQ 2019, MuSiQue 2021. That is the setup for [§7](#7-contamination-and-saturation), and it is the single strongest reason a QA dataset score should never be the *only* evidence for a RAG claim.

**Cross-reference:** the long-context-versus-retrieval design trade-off, including the cost, freshness and attribution arguments for each, is owned by [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md) and is not re-derived here.

---

## 6. The Leaderboards and How They Behave

### 6.1 What a leaderboard is made of

A leaderboard is a **submission pipeline with a public results table**. Its anatomy, in order:

1. **A frozen dataset** with a hidden or held-back test split (the dev split is public and the test split is not, which is why "please evaluate on the test set" is a *submission*, not a score you can produce yourself).
2. **A submission format** — a run file, a prediction file, or a container.
3. **A scoring harness** run by the maintainer, not the entrant.
4. **A public table** ordering submissions by a stated metric.
5. **A policy** about who may submit, how often, and what must be disclosed.

The MS MARCO submission requirements are a good concrete instance of item 5: an eval file, a dev file, team and institution names, model information, optional paper information, recommended model code — and an explicit discouragement of too many submissions from the same group in a short period "to avoid P-hacking" (MS MARCO project page, read 25 September 2026). **Read that as a design acknowledgement:** leaderboard maintainers know that repeated submission is a way to win by trying often, and they build policy to blunt it.

### 6.2 How rankings move

Rankings move for reasons that have nothing to do with model quality. The verified mechanisms:

| Mechanism | Evidence |
|-----------|----------|
| **The task set changes** | MTEB's maintainers describe extending the benchmark with new tasks and datasets as an ongoing maintenance activity, alongside CI that validates dataset integrity and assesses whether results generalise ([arXiv:2506.21182](https://arxiv.org/abs/2506.21182)). A score computed over 56 datasets is not comparable to one computed over a larger set. |
| **The leaderboard is retired or closed** | MS MARCO's passage and document retrieval leaderboards are recorded on the project page as **retired** (passage: retired 26 October 2018 – 1 January 2023; document: retired 11 August 2020 – 1 January 2023), and the earlier NLGEN and QnA leaderboards **closed on 23 October 2020** (page read 25 September 2026). |
| **The corpus is replaced** | The TREC RAG track moved from MS MARCO v2.1 to NVIDIA's ClimbMix-400b for its 2026 iteration (track site, read 25 September 2026). A run from the earlier corpus is not a run on the new one. |
| **The evaluation harness or judgments are revised** | The Touché 2020 subset of BEIR was denoised and augmented with post-hoc judgments in a reproduction study ([arXiv:2407.07790](https://arxiv.org/abs/2407.07790)), and the BEIR ecosystem had no single authoritative nexus for reporting until an official self-service leaderboard was built ([arXiv:2306.07471](https://arxiv.org/abs/2306.07471)). Re-scoring moves tables. |
| **A new benchmark takes over the question** | BRIGHT was created because relevance-style queries were considered too easy; the paper reports that the then-leading MTEB retrieval model scored 59.0 nDCG@10 there and **18.3 nDCG@10 on BRIGHT** (as reported, July 2024). When a benchmark stops separating systems, attention migrates and the old table becomes a historical record. |

### 6.3 The adoption dynamic — why a rank is worth money

A leaderboard placement is a **purchasing signal**, and it is rational for it to be one: a buyer with no labelled data of their own has almost nothing else to go on. The clearest evidence that it functions this way is that model authors explicitly cite their placement, with a date, in their own technical reporting. Two verified examples, each a **dated, self-reported snapshot**:

- An embedding technical report published **4 December 2024** states a **rank-1 placement among all models for retrieval tasks on the MTEB leaderboard, as of 29 May 2024**, alongside an average of 68.2 across 56 datasets (Linq-Embed-Mistral, [arXiv:2412.03223](https://arxiv.org/abs/2412.03223), abstract read 25 September 2026).
- A distillation technical report published **26 December 2024** states a **No. 3 position on the MTEB leaderboard, as of 24 December 2024**, with an average of 71.54 across 56 datasets (Jasper and Stella, [arXiv:2412.19048](https://arxiv.org/abs/2412.19048), abstract read 25 September 2026).

Both are historical. Neither is a current placement, and **this guide asserts no current ranking, top model or score on any leaderboard.** Their value here is as evidence of the mechanism: *leaderboard position is reported as a product claim, which is why it must be read as one.*

### 6.4 The honest limits of any leaderboard

1. **A rank is a snapshot.** It has a date, whether or not the claim carries one. A placement without a date is not a fact about the model; it is a fact about the model at an unknown time.
2. **A submission is self-reported at the input.** The maintainer scores the artifact, but the entrant chooses the configuration, the prompt, the retriever, the number of attempts, and often whether to report the best run.
3. **The leaderboard's corpus is nobody's corpus.** Every benchmark in [§3](#3-the-retrieval-benchmarks) uses a frozen collection. Yours is different, changing, and unlabelled.
4. **The benchmark's *choice of task* is doing work in the result.** The *Benchmark Lottery* paper (Dehghani et al., [arXiv:2107.07002](https://arxiv.org/abs/2107.07002), 14 July 2021) shows on multiple prevalent benchmark setups that **relative performance of algorithms can be altered significantly simply by choosing different benchmark tasks**, calls the overall fragility of the benchmarking process "a benchmark lottery", and argues that because every benchmark makes a statement about what it perceives to be important, this can produce **biased progress**. That paper is the theoretical basis for [§9](#9-how-to-read-a-benchmark-claim)'s insistence that a claim name *which* benchmark, not just *that* a benchmark was won.
5. **A leaderboard measures a submission, not a system in your environment.** Not the latency, not the index-refresh path, not the cost per query, not the refusal behaviour, not the fallback.

---

## 7. Contamination and Saturation

### 7.1 The mechanism, stated plainly

Public benchmark datasets live on the open internet. Pre-training corpora are scraped from the open internet. Benchmarks are therefore *inside* the training distribution of models evaluated on them, unless someone deliberately removes them. The contamination literature exists because that overlap inflates measured performance in a way that is invisible in the score.

The second mechanism is subtler and applies even to models with clean pre-training data: a benchmark's **test-set-derived artifacts** (gold answers, explanations, judging rubrics) can enter **fine-tuning** or evaluation-training data, and a benchmark's **dev split is public by design** — which is how iteration and selection bias creeps in. TREC DL's own reusable-collections paper names **iteration and selection bias when reusing a dataset** as a known risk and prescribes best practices for avoiding it ([arXiv:2104.09399](https://arxiv.org/abs/2104.09399)).

### 7.2 Demonstrated findings, with source and date

| Finding | Source | Date | Status |
|---------|--------|------|--------|
| **69% of the Robust04 test queries have near-duplicate queries in the MS MARCO/ORCAS training data**; leakage can **improve effectiveness and change the ranking of systems**, but these effects diminish as leakage becomes more realistic | Fröbe et al., *How Train-Test Leakage Affects Zero-shot Retrieval*, [arXiv:2206.14759](https://arxiv.org/abs/2206.14759) | 29 June 2022 | **DEMONSTRATED** by controlled experiment. This is the single most directly relevant contamination result for retrieval: it shows a *ranking* can be corrupted by leakage, not just an absolute score. |
| **Contamination affects models in measurable ways, and the prevailing n-gram definitions of contamination are inadequate.** Text contamination and ground-truth contamination have different effects, and repeating contamination matters | Jiang et al., *Investigating Data Contamination for Pre-training Language Models*, [arXiv:2401.06059](https://arxiv.org/abs/2401.06059) | 11 January 2024 | **DEMONSTRATED** in a controlled setting (a series of GPT-2-scale models pre-trained from scratch by the authors). The paper explicitly **criticises n-gram-based contamination definitions** used in LLM reports. |
| **Accuracy drops of up to 8% when models evaluated on a benchmark are re-evaluated on a twin benchmark built to the same specification**, with several model families showing evidence of **systematic overfitting across almost all model sizes**; a positive relationship (Spearman r² = 0.36) between a model's probability of generating a GSM8k example and its GSM8k-vs-GSM1k performance gap suggests some models may have **partially memorised** GSM8k | Zhang et al., *A Careful Examination of Large Language Model Performance on Grade School Arithmetic* (GSM1k), [arXiv:2405.00332](https://arxiv.org/abs/2405.00332) | 1 May 2024 | **DEMONSTRATED, AND PARTIAL.** The paper also reports that **many models, especially those on the frontier, show minimal signs of overfitting**, and all models broadly generalised to problems guaranteed not to be in their training data. It is evidence about *some* models, not all. |
| **A masked-option guessing protocol (TS-Guessing) finds that certain commercial LLMs can guess the missing option in multiple-choice items**, with ChatGPT and GPT-4 reported at **52% and 57% exact-match rates respectively** on MMLU; TruthfulQA performance improved when benchmark metadata was supplied | Deng et al., *Investigating Data Contamination in Modern Benchmarks for Large Language Models*, [arXiv:2311.09783](https://arxiv.org/abs/2311.09783) | 16 November 2023 | **DEMONSTRATED AS A DETECTION SIGNAL, SUSPECTED AS A CAUSAL CLAIM.** The paper *finds* that models can guess missing options; that is the authors' demonstrated observation. It is **not** a demonstration that any named model was trained on that benchmark — the paper's own framing is about inflated benchmark scores and lack of training-data transparency. Do not upgrade it. |

### 7.3 General concerns and methods, with source and date

| Claim | Source | Date | Status |
|-------|--------|------|--------|
| Data contamination — "the unintended overlap between training and test datasets" — can artificially inflate performance and overestimate true generalisation; contamination-free evaluation strategies fall into **data updating, data rewriting, and prevention**, and detection methods are categorised as **white-box, grey-box and black-box** | Cheng et al., *A Survey on Data Contamination for Large Language Models*, [arXiv:2502.14425](https://arxiv.org/abs/2502.14425) | 20 February 2025 | **SUSPECTED / GENERAL CONCERN, SURVEYED.** This is a survey of the concern and of measurement methods, not a finding about a specific benchmark or model. |
| A **Kernel Divergence Score** can measure dataset contamination by comparing the kernel similarity structure of sample embeddings before and after fine-tuning on a benchmark, with near-perfect correlation to contamination levels in **controlled** scenarios | Choi et al., *How Contaminated Is Your Benchmark?*, [arXiv:2502.00678](https://arxiv.org/abs/2502.00678) | 2 February 2025 | **METHOD, VALIDATED IN CONTROLLED SETTINGS.** Its validation is against injected contamination, not against real-world models. |
| Data leakage is a widespread cause of reproducibility failure in ML-based science: **17 fields**, collectively affecting **329 papers**, with a taxonomy of **8 types of leakage** | Kapoor & Narayanan, *Leakage and the Reproducibility Crisis in ML-based Science*, [arXiv:2207.07048](https://arxiv.org/abs/2207.07048) | 14 July 2022 | **DEMONSTRATED for pre-LLM ML science generally.** Included because it establishes the norm that leakage must be actively checked, not assumed away. It is **not** an LLM benchmark-contamination result — do not cite it as one. |

### 7.4 Where the field has no consensus

State these as open, because they are:

- **Whether n-gram overlap is sufficient to call something "contaminated".** One of the demonstrated studies above explicitly argues the prevailing n-gram definitions are inadequate and that text contamination and ground-truth contamination behave differently. A paper's "we checked for 13-gram overlap and found none" is therefore a **weak** contamination defence, not a clean bill of health.
- **How large the effect is on current frontier models.** The GSM1k result shows both overfitting *and* minimal overfitting, in different model families from the same evaluation. A blanket claim in either direction is unsupported.
- **Whether leaderboard movement is contamination-driven.** No source read for this guide establishes that a specific leaderboard transition was caused by contamination. Treat it as a hypothesis, not a finding.
- **Whether contamination even helps on every task.** Some of the demonstrated work distinguishes contamination types by effect, which implies the answer is task-dependent.

### 7.5 Saturation

Saturation is the state in which a benchmark is close enough to its ceiling that the remaining differences between systems are noise, measurement error, or luck. Three verified observations of it, in three different families:

- **MMLU is saturated by measurement error, not by model capability.** *Are We Done with MMLU?* (Gema et al., [arXiv:2406.04127](https://arxiv.org/abs/2406.04127), 6 June 2024) identifies ground-truth errors in the benchmark, reports that **57% of the analysed questions in the Virology subset contain errors**, and estimates that **6.49% of MMLU questions overall contain errors** using a 5,700-question manually re-annotated subset (MMLU-Redux) across all 57 subjects. Its comparison against the re-annotated subset **disagrees with originally reported model performance metrics**. A saturated score is not only uninformative — it may be measuring a partly wrong answer key.
- **Needle-in-a-haystack is saturated, and the benchmark that saturates reveals the one that does not.** RULER reports that models achieve **nearly perfect accuracy on vanilla NIAH** while showing large drops as context grows and on harder task categories ([arXiv:2404.06654](https://arxiv.org/abs/2404.06654)). HELMET finds most LCLMs reach perfect NIAH scores while **synthetic tasks do not reliably predict downstream performance** ([arXiv:2410.02694](https://arxiv.org/abs/2410.02694)).
- **Benchmark choice itself produces apparent progress.** The Benchmark Lottery ([arXiv:2107.07002](https://arxiv.org/abs/2107.07002)) shows relative algorithm performance can be altered significantly by choosing a different benchmark task. When benchmarks saturate, the visible "progress" can be a property of the yardstick.

**And the observable consequence:** new benchmarks keep appearing, each justified by a gap in an earlier one. The verified timeline in this guide alone is FEVER 2018 → NQ 2019 → 2WikiMultiHopQA 2020 → MuSiQue 2021 → BEIR 2021 → MTEB 2022 → RGB 2023 → RAGTruth 2023 → MultiHop-RAG 2024 → CRUD-RAG 2024 → RAGBench 2024 → RULER 2024 → HELMET 2024 → MultiHop-RAG-era derivatives → BRIGHT 2024 → FRAMES 2024 → T²-RAGBench 2025 → LIT-RAGBench 2026. That is what a working frontier looks like, and it is also why **a five-year-old benchmark number is a historical artifact, not a current measurement.**

`⚠` **Flagged:** this guide found no single canonical paper on *benchmark saturation* as a phenomenon (arXiv full-text queries for saturation-related titles returned no matching entries). The saturation argument above is assembled from the papers that *observe* saturation in specific benchmarks — MMLU, NIAH, and benchmark-choice effects — and is labelled as assembled reasoning rather than cited to one source. See [§16](#16-what-could-not-be-verified-glossary-cross-references-and-closing-summary).

---

## 8. Why a Retrieval Score Is Not a RAG Score

This is the analytical centre of the guide. Everything before it describes instruments; this section is about the gap between an instrument and a system.

### 8.1 The one-sentence version

**A benchmark scores a component or a pipeline on a fixed corpus with a fixed task; a deployment is a different corpus, different queries, a changing index, a real failure cost, and an end-to-end answer nobody in the benchmark ever had to produce.**

### 8.2 The five axes of divergence

| Axis | What the benchmark fixes | What your deployment has | What breaks in the translation |
|------|--------------------------|--------------------------|-------------------------------|
| **The corpus** | One collection, published once, with relevance labels produced for that collection. | Your documents: different vocabulary, different structure, different density of near-duplicates, and **no labels**. | A retriever can be strong on a 1M-passage web corpus and weak on 40,000 internal policy PDFs with tables and headers. Learned lexical overlap statistics do not transfer across those distributions for free. |
| **The queries** | Queries written or collected for the benchmark, including the benchmark's *style* (keyword, natural-language question, narrative). | Your users' queries: malformed, mixed-language, containing product codes, referring to a previous turn. | The TREC RAG track had to change its input from *topics* to *narratives* between iterations — the input distribution is itself a design variable. The colour of your queries is not the benchmark's colour. |
| **The freshness** | A frozen snapshot. MS MARCO's corpus has not changed; BEIR's datasets are versioned artifacts. | A corpus that changes: new products, revised fee schedules, superseded procedures, deleted pages. | A retrieval metric computed at t=0 says nothing about the t+6-months system, which is what actually runs in production. Freshness failures appear as answers grounded in superseded documents — a *correct retrieval of a stale document*. |
| **The failure cost** | An irrelevant passage in the top-10 costs a fraction of a metric point. | An irrelevant answer in a regulated process costs money, a complaint, or a remediation. | Metrics are symmetric in a way consequences are not. A false negative (missing a document) and a false positive (adding a distracting one) may be weighted equally in nDCG and weightlessly differently in your process. |
| **The end-to-end answer** | Most retrieval benchmarks never produce an answer. | The user receives an answer, with an attribution, or a refusal. | This is the largest gap, and it is additive: even a perfect retriever can be followed by a misreading, a fabricated attribution, or an unjustified refusal. |

### 8.3 The metric layers are owned elsewhere — cross-referenced, not restated

The scoring definitions for all three layers live in [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) and are **not** repeated here:

- **Retrieval metrics** (the retrieval layer) — that guide's **§2 Retrieval Metrics**.
- **Generation metrics** (the generation layer) — that guide's **§3 Generation Metrics**.
- **End-to-end metrics** (the user-visible layer) — that guide's **§4 End-to-End Metrics**, including the reference-based-versus-reference-free distinction and the role of goldens.
- **The three evaluation layers themselves** — that guide's **§1 Evaluation Dimensions**, which maps each layer to the question it answers and the object it measures.
- **The failure-mode taxonomy** that turns a bad score into a diagnosis — that guide's **§9 RAG Failure Modes**.

The point of the cross-reference is structural, not decorative. **Every benchmark in this guide sits at exactly one layer.** Mapping them is the fastest way to see why their numbers do not compose:

| Benchmark | Layer it scores | What its number therefore is *not* |
|-----------|-----------------|-----------------------------------|
| BEIR, MTEB, MS MARCO, TREC DL, BRIGHT | **Retrieval** (and, for MTEB, embedding quality across tasks) | Not an answer-quality number. A retriever has no answer. |
| RGB, CRUD-RAG, MultiHop-RAG | **Retrieval + generation, diagnostically** | Not a deployment number — the corpus is the benchmark's, and the abilities are scored in isolation by testbed. |
| RAGBench, FRAMES, TREC RAG 2024/2026 | **End-to-end, with attribution or explainability** | Not a component number — you cannot attribute a drop to the retriever or the reader from the aggregate alone. |
| LongBench, LongBench v2, RULER, HELMET | **Reader under supplied context** (long-context) | Not a retrieval number at all. |
| ARES, RAGTruth | **Evaluation method / annotation layer** | Not a benchmark dataset — these are how you score, and how the labels are made. |

### 8.4 Two corpora, one name

The sharpest concrete demonstration that a retrieval score is not portable is a dataset-version problem, and it is documented in the MS MARCO family itself.

*The tale of two MS MARCO — and their unfair comparisons* (Lassance et al., [arXiv:2304.12904](https://arxiv.org/abs/2304.12904), 25 April 2023) shows that **two different corpora of MS MARCO-passage are used in the literature** — the official one, and a second in which passages were augmented with titles (largely because of one code base's default). The addition of titles **leaks relevance information** and breaks the original dataset's guidelines; the authors demonstrate empirically that the two versions **make a significant difference when evaluating a new method**, and that **if a paper does not properly report which version is used, reproducing its results fairly is basically impossible**.

A second MS MARCO construction issue compounds it: *On Survivorship Bias in MS MARCO* (Gupta et al., [arXiv:2204.12852](https://arxiv.org/abs/2204.12852), 27 April 2022) observes that annotators could not find answers to **38–45% of the queries**, which were consequently discarded from training and evaluation; many of the discarded queries were valid questions that could have been answered with more complete annotation (about two thirds of them, using modern ranking techniques). The paper finds this **distorts the natural distribution of queries by the type of information needed** and **likely yields a significant distortion of the absolute performance scores** — and that models trained on more survivorship-biased subsets score up to **9.9% worse** on more completely annotated versions and up to **3.5% worse** at zero-shot transfer.

Read those two papers together and you have the whole argument of §8 in miniature: **the same benchmark name can denote two different artifacts, and the absolute score depends on annotation choices made years before your evaluation.**

### 8.5 The amplification argument

Retrieval and generation failures do not simply add; they interact.

- **Recall limits the ceiling.** If the evidence needed to answer is not in the retrieved context, no reader can produce a grounded answer — so a retrieval metric is an *upper bound* on the end-to-end score for answerable queries. But a high retrieval recall is not a *lower bound*: the reader can still misread, over-conclude, or attach a wrong citation.
- **Noise consumes attention.** RGB's noise-robustness testbed and the information-integration testbed exist precisely because adding documents helps and hurts at the same time. A retrieval metric that only rewards including the right document does not capture the *harm* of the extra documents a system retrieved to be safe.
- **Refusal is a policy, not a metric.** "Negative rejection" (RGB) and Abstention (LIT-RAGBench) are scored in benchmark-specific ways. In a deployment, the refusal threshold is an operational choice with a cost curve, and no retrieval benchmark contains it.
- **A ranked list has no answer.** The final answer, its citations, and its fluency are scored by generation and end-to-end metrics in the methodology guide — never by a retrieval score.

### 8.6 The three questions a retrieval score cannot answer

Put these on the wall next to any RAG scorecard:

1. **Which corpus was this measured on, and is mine shaped like it?**
2. **Which layer did this measure — the retriever, the reader, the evaluator, or the pipeline — and were the other layers held constant or assumed perfect?**
3. **What does this number license me to do?** A retriever comparison licenses "shortlist these three for my own evaluation". It never licenses "accept this retriever". That distinction is [§10](#10-how-to-use-a-public-benchmark-responsibly) and [§12](#12-the-regulated-institution-angle).

---

## 9. How to Read a Benchmark Claim

### 9.1 The eleven questions

A benchmark claim is a sentence of the form *"our system is better, because of a number."* Every one of the following questions must have an answer in the claim itself — not in a call you have to make, not in a paper you have to find — before the number can be admitted as evidence. Nine of them can be answered by a competent vendor in an email; the absence of any one of them is the finding.

1. **Which benchmark — and does it exist?** A named, verifiable benchmark with a real paper or project page, traceable to its own source. A benchmark you cannot find is not a benchmark. (See [§15](#15-the-claims-audit) for a real example of a name that looks like a benchmark and is a corporate entity.)
2. **Which version or release?** Benchmarks are versioned artifacts. MS MARCO's two passage corpora are the canonical demonstration ([§8.4](#84-two-corpora-one-name)). "BEIR" without a release is an incomplete citation.
3. **Which split?** Test, dev, or "we evaluated on the dev set". The third is a legitimate engineering step and an illegitimate headline.
4. **Which track?** Zero-shot or fine-tuned? Fine-tuned *on what* — the benchmark's own training split, the benchmark's training data, or data derived from the benchmark's domain? A fine-tuned number and a zero-shot number are different products.
5. **Which metric, at which cutoff?** nDCG@10 and nDCG@100 can rank systems differently; Recall@k and MRR@k answer different questions. The definitions are the methodology guide's §2; the requirement here is that the claim *name* them.
6. **Which baseline, and was it tuned?** "Beats the previous state of the art" and "beats BM25" are very different sentences. BEIR's abstract states plainly that BM25 is a robust baseline — a claim that omits a BM25 comparison on the same corpus and metric has omitted the cheapest available control.
7. **Read on which date?** A leaderboard placement expires in weeks. Without a date, the claim cannot be checked and cannot be falsified.
8. **Which evaluator produced the number?** Where the metric is model-judged, the judge is a variable — RAGBench's authors report that LLM-based evaluation methods struggled to compete with a fine-tuned evaluator on their labels ([arXiv:2407.11005](https://arxiv.org/abs/2407.11005)). A judged number without the judge named is a number with an unnamed author.
9. **At what cost?** BEIR's finding that the best zero-shot average performers are re-ranking and late-interaction models **at high computational cost** is the classic statement of this: quality claims that silently ignore compute and latency are not comparable claims.
10. **Whose hand wrote the numbers — and does the author also produce a model in the same benchmark?** That a lab both builds a benchmark and is evaluated on it is a **neutral fact worth recording** ([§9.4](#94-a-benchmark-is-a-task-and-a-task-has-an-author)). It does not imply misconduct. It does mean the claim should be read with the reproduction path in hand.
11. **On whose hardware and with whose index?** A retrieval score is a property of a retriever *plus* an index *plus* a chunking choice. The TREC RAG track had to mandate that participants map their chunks back to a common collection "for reproducibility and ease of evaluation" (track site, read 25 September 2026) precisely because this variable is otherwise uncontrolled.

### 9.2 The vendor-language decoder

| What the claim says | What it establishes | What it does not |
|---------------------|---------------------|------------------|
| "We are #1 on the leaderboard." | One submission ranked first on one table, on one day, against the submissions that happened to be there. | Anything about your corpus, your queries, or your failure cost. |
| "We scored 0.9 accuracy on RAGBench." | A pipeline scored 0.9 on one of five industry domains' task types, under RAGBench's TRACe metrics and its evaluator. | That your domain is among the five that matter to you, or that 0.9 on that metric is 0.9 on your process. |
| "We top the retrieval leaderboard." | A **retriever** ranking result. | Anything about answers. A retriever produces no answer ([§8.3](#83-the-metric-layers-are-owned-elsewhere--cross-referenced-not-restated)). |
| "We beat GPT-class models on multi-hop QA." | A result on a named QA dataset, probably with supplied context. | That retrieval is in the loop at all ([§5.5](#55-what-changes-when-you-borrow-a-qa-dataset-as-rag-evidence)). |
| "We handle 1M tokens." | A context-window capability claim, in the family RULER and HELMET both show is poorly predictive of downstream performance. | Retrieval quality, or quality at that length on a hard task. |
| "Our model was trained on the same data as the top leaderboard entries, so we are strictly better." | Nothing. | This is an unverifiable training-data assertion, offered as a quality argument. It asserts contamination as a feature — and it is a red flag, not evidence. |

### 9.3 The checklist, for copy-pasting into a vendor evaluation file

> **A benchmark claim is admissible as evidence only when it answers all of:**
> `[ ]` Named benchmark, traceable to its own paper or project page.
> `[ ]` Named version/release of the dataset used.
> `[ ]` Named split (test, not dev).
> `[ ]` Named track (zero-shot or tuned, and tuned on what).
> `[ ]` Named metric with its cutoff.
> `[ ]` Named baseline, ideally including BM25 on the same corpus and metric.
> `[ ]` The date the placement was read.
> `[ ]` The evaluator/judge named where the metric is judged.
> `[ ]` The cost/latency context stated.
> `[ ]` The publication or report that carries the claim, so it can be reproduced.
> `[ ]` A statement of what the claim does *not* cover.
>
> **A claim missing any line is not evidence. It is a lead.**

### 9.4 A benchmark is a task, and a task has an author

Every benchmark in this guide was built by people, usually a small group, usually from one or two institutions, and usually alongside their own systems.

- BEIR, MTEB, and the BEIR reference/leaderboard work share authorship across a small number of groups; the TREC DL and TREC RAG tracks were organised by the same research community (TREC RAG 2024 organisers: Pradeep, Thakur, Lin at Waterloo and Craswell at Microsoft; TREC RAG 2026 organisers add Jedidi, Gu, Sadeghi, Upadhyay, Campos and retain Thakur, Craswell, Pradeep and Lin — track sites read 25 September 2026).
- ARES's abstract records that it was validated on **eight knowledge-intensive tasks in KILT, SuperGLUE and AIS**, and KILT's own authors are a laboratory that also builds retrieval models.
- MS MARCO, TREC DL and BEIR are all, in different degrees, **search-engine industry artifacts** — built by the organisations that have the query logs and the relevance-assessment capacity.

**Record that as a neutral fact, exactly as this repo's house rule requires.** Concretely, three implications that are about method rather than motive:

1. **A benchmark's task definition encodes its authors' view of what matters.** The Benchmark Lottery paper makes this argument explicitly: every benchmark "makes a statement about what it perceives to be important", which can lead to biased progress ([arXiv:2107.07002](https://arxiv.org/abs/2107.07002)).
2. **A benchmark that scores a component will favour whatever the authors' component pipeline is good at.** MTEB's task taxonomy, RGB's four abilities and HELMET's seven categories are all *choices* of what to measure.
3. **Self-evaluation is normal, disclosed, and requires the reader to demand a reproduction path.** When a laboratory's model is evaluated on a benchmark that laboratory also built or maintains, the claim is legitimate and the natural independent check is another benchmark — which is precisely the BRIGHT result ([§3.6](#36-bright--the-reasoning-intensive-retriever-test)) where a leader's home-turf score did not transfer.

This is why [§12](#12-the-regulated-institution-angle) separates *evidence* from *vendor evidence*, and why the checklist in [§9.3](#93-the-checklist-for-copy-pasting-into-a-vendor-evaluation-file) requires the publication that carries the claim.

---

## 10. How to Use a Public Benchmark Responsibly

### 10.1 The two legitimate uses

**Use 1 — the sanity floor.** A public benchmark tells you whether a candidate is *plausibly* competent. If a vendor's embedding model is materially behind BM25 on a heterogeneous zero-shot retrieval benchmark — the baseline BEIR's own authors describe as robust — you have learned something cheap and important, and you have learned it without building anything. That is a *floor*: it disqualifies, it does not qualify.

**Use 2 — the shortlist generator.** A public benchmark reduces a field of fifty to a field of three or four, all within a band that the benchmark cannot statistically separate. At that point the benchmark has done its job and has nothing more to say, because the differences inside the band are the ones only your corpus can adjudicate.

### 10.2 The five rules

1. **Record the version and the read date.** Every benchmark number you keep in a decision file gets a dataset version and a date, in the same way a valuation gets a timestamp. A number without both is not a data point; it is a memory.
2. **Never let a rank override your own goldens.** The release decision belongs to a test set you own, whose items you can defend, whose coverage you designed, and whose reviewers are named — the method in [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §6, and specifically §6.1 (curated Q&A with verified answers) and §6.5 (ownership, versioning and re-review).
3. **Use subsets that match your task shape.** A benchmark is a family of tasks, and the families are heterogeneous by construction: BEIR spans 18 domain/task combinations; MTEB spans a task taxonomy; LongBench spans six categories whose trends are distinct and only weakly correlated. Reporting one aggregate number over a heterogeneous family is averaging away the only part you needed.
4. **Keep the release gate internal, always.** Public benchmark instances are public; your gate items must not be. If a public benchmark instance becomes part of your release gate, then anyone who trains on that benchmark has trained on your gate — and you will not know. The permanent answer is a gate set that is *yours* and is not published.
5. **Treat every borrowed number as a hypothesis to be falsified on your data.** The cheapest falsification is usually a small, targeted run: 50–100 queries from your own logs, with relevance labels for the documents your retriever should have found.

### 10.3 What a public benchmark is good for, and what it is not

| Good for | Not good for |
|----------|-------------|
| Disqualifying a candidate that is plainly behind a strong baseline | Accepting any candidate |
| Producing a defensible shortlist cheaply | Producing a release decision |
| Establishing that the field has a frontier worth tracking | Establishing that you are on it |
| Calibrating your expectations of what a retriever or embedder *can* do | Calibrating what your system *does* do |
| Giving an auditor a public, dated reference point for the *approach class* | Giving an auditor evidence of fitness for a regulated use case |
| Cheaper external context in a vendor comparison | Replacing the vendor comparison on your corpus |

### 10.4 The failure mode to avoid: benchmark-driven development

The subtle version of misuse is not dishonesty — it is **optimising the yardstick instead of the product**. It looks like this: the team adopts a public benchmark as the internal metric because it is free and reproducible; every stack change is measured against it; the score climbs; the product does not improve, because the benchmark's corpus and queries are not the product's.

The guardrail is the discipline the methodology guide calls eval-driven development against a test set **you** own, run on **every** change ([rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §8). A public benchmark can live in the same CI as a *guard* — if a change tanks a well-known benchmark, something changed — but it cannot be the number that gates a release.

---

## 11. Build versus Borrow

Kept deliberately short, because the method is owned elsewhere.

| Dimension | Public benchmark | Your own goldens |
|-----------|------------------|------------------|
| **What it proves** | Comparable to the field, on the field's task and corpus | Fit for *your* purpose, on your population and corpus |
| **Cost to obtain** | Near zero (download and run) | High upfront — a bank's SME time is the real cost |
| **Freshness** | Frozen at publication; ages with the field | You re-review on your own cadence |
| **Coverage of your corpus** | None; a different collection entirely | Total, by construction |
| **Defensibility to a model-risk function** | Useful as external context for the *approach class* | The only artifact that can carry a release decision |
| **Contamination risk** | Public by definition; a trainable target | Containable, if the gate set is never published |
| **Maintenance** | Someone else's problem — and someone else's closure decision (MS MARCO's leaderboards were retired) | Yours: owners, versions, re-review triggers |

**The method** — how to write, size, version, own, and re-review a golden set, including the coverage dimensions to sample across — is [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §6, with the reference-free-versus-reference-based trade-off in §5 and the pipeline that runs it in §8. **The tooling** that executes it is [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md) and its deep-dives. Nothing about either is re-derived here.

The one addition this guide makes is the *sequencing* rule:

> **Borrow to shorten the list. Build to make the decision.** A public benchmark is a filter with a wide aperture; your goldens are the gate. Use the filter first because it is cheap, and never let the aperture decide.

---

## 12. The Regulated-Institution Angle

### 12.1 The claim that must be refused

**A leaderboard placement is not model validation, not a control, and not evidence of fitness for a regulated use case.**

Spell out the failure: a model-risk function that accepts *"our embedding model ranks in the top handful on a public retrieval leaderboard"* as the evidentiary basis for deploying a customer-facing retrieval system has accepted (a) a number measured on someone else's corpus, (b) by someone else's evaluator, (c) on someone else's task, (d) at an unstated date, (e) without any defined population, threshold, monitoring plan, or limitation statement. That is not a thin file. It is not a file.

### 12.2 The repo's governance content, cited by verified name

The framing that a benchmark number cannot be a control belongs to guides that already own it in this repo — read those rather than re-deriving:

- **[ai_governance_framework_guide.md](../ai_governance_framework_guide.md)** — *AI Governance Frameworks and the Enterprise Operating Model*: the operating model within which any evaluation artifact has to sit, including who owns the model, who approves it, and what the evidence chain looks like.
- **[llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md)** — *LLM Evaluation vs Validation*: the distinction this section depends on. Evaluation is the engineering activity; **validation is the independent confirmation that the system is fit for its intended use**, and it is validation — not a benchmark score — that a model-risk function signs.
- **[ai_governance_bias_redteaming_guide.md](../ai_governance_bias_redteaming_guide.md)** — *AI Governance, Bias Mitigation & Red-Teaming Practices*: the adversarial side of assurance. Note that no public retrieval benchmark in [§3](#3-the-retrieval-benchmarks) contains an adversarial or red-team component; poisoning and prompt-injection robustness are separate evidence requirements.
- **[llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md)** — *DeepEval and LLM Evaluation Frameworks*: the tooling layer that runs your metrics — the layer at which your own evaluation becomes automatable and auditable.
- **[implementing-responsible-ai.md](../implementing-responsible-ai.md)** — the responsible-AI practice guide: the broader set of obligations a regulated deployment carries beyond accuracy.
- **[rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md)** — §6.5 on goldens as a **controlled asset with owners and versioning**, and §7 on offline-versus-online evaluation: the artifact and the rhythm that a validation file actually needs.

### 12.3 What a model-risk function requires instead

Not a benchmark placement. Concretely:

| Requirement | Why a public benchmark cannot satisfy it |
|-------------|------------------------------------------|
| **A defined intended use and population** | A benchmark has no population. Its "users" are the queries someone wrote. |
| **A test set representative of that population, with a version and an owner** | Yours is unlabelled; the benchmark's is someone else's. |
| **Metrics tied to the decision being made, with thresholds agreed in advance** | A benchmark's metric was chosen to separate research systems, not to price a false-answer risk. |
| **Documented limitations, including known failure modes** | The benchmark paper's limitations are about *the benchmark*, not about your deployment. |
| **Reproducibility on demand, by an independent party** | A vendor's benchmark claim is normally not reproducible by the institution without the vendor's artifact and configuration. |
| **Ongoing monitoring, drift detection and re-validation triggers** | A benchmark is scored once. A deployment is scored forever. |
| **Human oversight, escalation and rollback** | Not representable in a metric at all. |
| **An audit trail from requirement to evidence** | A leaderboard URL is not an audit trail. |

### 12.4 The honest value of a public benchmark inside a regulated institution

It is real, and it is narrow:

1. **It narrows a shortlist cheaply.** Ten candidate embedding models cannot each be evaluated on your corpus by your SMEs; three can.
2. **It provides external, dated context for the approach class.** "Retrieval-augmented approaches of this type are competitive on heterogeneous zero-shot retrieval, and BM25 remains a strong baseline" is a legitimate, citable statement — and the second half of it is *useful*, because it forces the "why not BM25 first?" conversation that a governance function should be having anyway.
3. **It supplies a falsification tool for vendor claims.** The checklist in [§9.3](#93-the-checklist-for-copy-pasting-into-a-vendor-evaluation-file) turns an unverifiable marketing sentence into a specific, answerable question. That is a real control — an administrative one, applied to the claim rather than to the model.

And the boundary, stated once: **after the shortlist, the institution's own evaluation governs.** The benchmark's role ends where the institution's evidence begins, and the two must never be filed in the same folder.

---

## 13. Worked Example — Three Vendor Pitches at Cymbal Bank

> **All figures in this section are ILLUSTRATIVE. Cymbal Bank is a fictional institution used throughout this repo as its only bank persona; no real institution, vendor or model is described, and no real benchmark placement is asserted. Any resemblance to a real pitch is the point — these pitches are composites of claim shapes that the checklist in [§9](#9-how-to-read-a-benchmark-claim) is built to handle.**

### 13.1 The situation

Cymbal Bank is standing up a RAG service over its internal product, fee-schedule and trade-finance procedure documents — a corpus of a few tens of thousands of documents, refreshed weekly, with answers that must carry a citation and must never invent a fee. The service needs **an embedding model and a reranker**. Three vendors arrive. Each cites a different benchmark and a different placement.

### 13.2 The three pitches, as received

**Pitch A — the leaderboard rank with no date.**

> "Our reranker holds the top position on the whole-corpus benchmark that everyone in retrieval uses. We are the best reranker available."

**Pitch B — the component score presented as a system score.**

> "Our embedding model scores 0.92 on the leading RAG benchmark's end-to-end accuracy, and leads the multilingual embedding leaderboard on retrieval."

**Pitch C — the collapse.**

> "Our reranker holds the top slot on the leaderboard everyone trained on — the passage ranking leaderboard over that famous search corpus. It has been the reference point for a decade, and we are still on top."

### 13.3 Applying the §9.3 checklist to each

| Checklist line | Pitch A | Pitch B | Pitch C |
|----------------|---------|---------|---------|
| Named benchmark, traceable to a paper | ⚠ "the whole-corpus benchmark that everyone uses" — no name | ⚠ "the leading RAG benchmark" / "the multilingual embedding leaderboard" — no name | ✅ A real, named leaderboard family exists (the MS MARCO passage-ranking leaderboard) |
| Named version/release | ❌ | ❌ | ❌ |
| Named split (test, not dev) | ❌ | ❌ | ❌ |
| Named track (zero-shot or tuned) | ❌ | ❌ | ❌ |
| Named metric with cutoff | ❌ "top position" | ✅ a number, but unattributed to a metric definition | ❌ |
| Named baseline, incl. BM25 | ❌ | ❌ | ❌ |
| **The date the placement was read** | ❌ | ❌ | ❌ |
| Named evaluator where judged | ❌ | ❌ | n/a |
| Cost/latency context | ❌ | ❌ | ❌ |
| A publication carrying the claim | ❌ | ❌ | ❌ |
| What the claim does **not** cover | ❌ | ❌ | ❌ |
| **Checklist result** | **Inadmissible as evidence; admissible as a lead** | **Inadmissible; and category-mixed** | **Rejected** |

### 13.4 The collapse, explained

Pitch C fails twice, and the second failure is the one worth teaching.

**Failure one — category confusion that is also a currency problem.** The claim names a *passage re-ranking* leaderboard. Re-ranking is a component task scored over a corpus that is not Cymbal's, and — per the MS MARCO project page read 25 September 2026 — the passage-retrieval leaderboard itself is recorded as **retired (26 October 2018 – 1 January 2023)**, with the document-retrieval leaderboard retired 11 August 2020 – 1 January 2023 as well. So the claim cites a **retired** leaderboard with **no date**, which means the placement cannot be checked against a live table and cannot be dated at all. "We are still on top" of a table that stopped accepting submissions is a statement about a historical record, not about a product.

**Failure two — the sourcing claim.** Pitch C's framing ("the leaderboard everyone trained on") asserts that the field trained on the benchmark. That is precisely the kind of contamination assertion [§7](#7-contamination-and-saturation) forbids: the documented literature contains **demonstrated** leakage effects in controlled and retrieval settings, and **no** source read for this guide establishes that a specific vendor trained on a specific benchmark. Here it does not matter, because the claim is the vendor's own about itself — but the right response is still not "we accept that you trained on it", it is: *a training-data assertion is unverifiable, and an unverifiable assertion cannot be a quality argument.* If anything, contamination is the reason a benchmark number is worth *less*, not more.

**Why C collapses while A and B survive.** A and B are incomplete and category-mixed, but they are *answerable*: Cymbal can send back the checklist and get a named benchmark, a version, a date and a report. C's claim is not merely incomplete — its provenance is retired and undated, so there is no live artifact against which any follow-up question can be resolved.

### 13.5 The shortlist the benchmarks generate

| Candidate | Shortlist outcome | Condition attached |
|-----------|-------------------|--------------------|
| **Reranker A** | **Shortlisted** | Provide the named benchmark, version, metric and read date; then a bake-off on Cymbal's own retrieval labels |
| **Embedder B** | **Shortlisted, with a corrected comparison** | Re-submit the claim as a *component* result: an embedding/retrieval score, not "end-to-end RAG accuracy" — then the same bake-off |
| **Reranker C** | **Rejected** | A retired, undated leaderboard placement is not checkable evidence. Re-pitch with a live artifact or not at all |

Note what the benchmarks did here: they did **not** select the winner. They eliminated one candidate and shortened the list to two, on the strength of a claim's *provenance* rather than its *magnitude*. That is the sanity-floor and shortlist-generator use of [§10](#10-how-to-use-a-public-benchmark-responsibly), applied.

### 13.6 The sanity floor the benchmarks provide

Two floors, both cheap:

1. **The BM25 floor.** BEIR's authors report BM25 as a robust baseline that dense and sparse models often underperform zero-shot ([arXiv:2104.08663](https://arxiv.org/abs/2104.08663)). So the service's retrieval baseline is **BM25 first** — the cheap, explainable, no-embedding baseline goes into the bake-off as a control. If a candidate reranker cannot beat that baseline on Cymbal's own queries, the candidate has failed a test the public literature told Cymbal to run.
2. **The task-shape floor.** MTEB's finding that **no particular text embedding method dominates across all tasks** ([arXiv:2210.07316](https://arxiv.org/abs/2210.07316)), and HELMET's finding that its seven categories **exhibit distinct trends and low correlations with each other** ([arXiv:2410.02694](https://arxiv.org/abs/2410.02694)), together mean a single aggregate embedding or long-context number cannot be Cymbal's selection criterion. The floor is: *which task in the taxonomy is the one Cymbal's workload actually resembles*, and evaluate that subset.

Both floors cost an afternoon. Neither requires Cymbal's own labelled data.

### 13.7 The goldens Cymbal builds instead — the release gate

`⚠` **All numbers below are ILLUSTRATIVE.**

- **150 release-gate items.** Written by the product and trade-finance SMEs, curated by the evaluation owner, over the corpus the service actually serves, distributed across the coverage dimensions of [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §6.3 — including the cases a benchmark would never contain: a fee that changed last quarter, a question whose correct answer is "there is no such product", and a question where two documents appear to conflict and one is superseded. (The methodology guide's own guidance is a 100–200+ item range for a single domain's release-gate goldens; 150 sits inside it.)
- **Retrieval labels for the same items.** Per item, the chunk set that *must* be retrieved for a grounded answer to be possible — so retrieval and generation can be scored separately, in the manner of RGB's testbeds and MultiHop-RAG's supporting-evidence annotations, but on Cymbal's corpus.
- **A refusal set.** Questions with no supporting document, scored as the deployment scores them — the operational version of RGB's "negative rejection", with the threshold treated as a business decision rather than a metric default.
- **A versioned, owned artifact.** Versioned, owned, and re-reviewed quarterly against the current corpus, because fee schedules and products change — the guard against the "correct retrieval of a stale document" failure in [§8.2](#82-the-five-axes-of-divergence).
- **Never published.** The gate set is internal. It is not shown to vendors, not used in marketing, and not uploaded anywhere — which is Cymbal's only real defence against the gate itself becoming a training target.

Every metric computed over those artifacts — precision/recall/nDCG at the retrieval layer, faithfulness at the generation layer, correctness on goldens end-to-end — is defined in the methodology guide and tooled per the evaluation-tools comparison. Cymbal writes no new metric definitions; it applies those.

### 13.8 The recommendation

> **Adopt BM25 as the control, shortlist Reranker A and Embedder B, reject Reranker C, and let the 150-item golden set with retrieval labels decide.** Keep the vendors' benchmark claims in the decision file as *dated external context for the approach class*, with the checklist attached and the gaps marked. Require the winning configuration to be re-scored after every corpus refresh, not once at selection.

The public benchmark landscape was worth Cymbal's attention for exactly one afternoon and exactly two jobs: eliminating a candidate, and telling Cymbal which baseline to include. The rest is Cymbal's own evidence. That ratio — one afternoon of borrowed evidence against a permanent internal gate — is the honest shape of the trade.

---

## 14. The Anti-Patterns

| Symptom | Cause | Guardrail |
|---------|-------|-----------|
| **"We're top of the leaderboard"** appears in a slide with no date | Leaderboard claims are repeated as evergreen; the table they came from is a snapshot | Every rank carries leaderboard + task + read-date + the word *snapshot*. A rank without a date is not admitted to the decision file ([§9.1](#91-the-eleven-questions), line 7) |
| A retrieval score is presented as a RAG quality score | Layer confusion: the retrieval layer, the generation layer and the end-to-end layer are different objects | Map every number to exactly one layer before it is quoted; the benchmark-to-layer map is [§8.3](#83-the-metric-layers-are-owned-elsewhere--cross-referenced-not-restated) |
| A benchmark number is quoted from a vendor blog, an analyst slide or a news article | Secondary sources drop the version, split, metric and date that make a number legible | Cite the paper or the project's own page. If it can only be sourced to a blog, it is a lead, not a fact |
| A public benchmark is wired into the release gate | It is free, reproducible and already automated — and it is somebody else's corpus | The release gate is a versioned, owned, unpublished internal set. Benchmarks may sit in CI as *guards*, never as the gate ([§10.2](#102-the-five-rules)) |
| "It's fine, our corpus is just like Wikipedia / news / web search" | Assumption that retrieval generalises across distributions for free | Assume the opposite until measured: a 50–100 query pilot on your own logs is the cheapest test. BEIR's translation findings show even *linguistic* transfer is not free |
| A contamination claim is repeated as a proven fact about a named model | The literature mixes controlled demonstrations with suspicion and general concern | Label every claim DEMONSTRATED or SUSPECTED, with source and date, and never upgrade one to the other ([§7.2](#72-demonstrated-findings-with-source-and-date), [§7.3](#73-general-concerns-and-methods-with-source-and-date)) |
| A benchmark is cited that does not exist — or the name is a homonym of a company, a colour space, a DID method or an ordinary word | Names collide across domains; naive text search cannot tell them apart | Verify at the benchmark's own paper before using the name. Real examples of these collisions are recorded as rejected items in [§15.4](#154-the-four-names-that-look-like-coverage-and-are-not) |
| A five-year-old benchmark number is used as a current measurement | Benchmarks saturate and are superseded; the field's answers move | Date every number, and prefer benchmarks whose task still resembles yours. FEVER (2018) and MMLU-era numbers are historical artifacts |
| A vendor's "we trained on the benchmark data" is treated as a quality argument | Contamination is being reframed as capability | Unverifiable training-data assertions are not evidence; they are reasons to discount a benchmark number, not to trust it ([§13.4](#134-the-collapse-explained)) |
| A leaderboard placement is presented to an auditor as validation | Rank is a purchase signal, and the model-risk file needs test evidence, thresholds, monitoring and limitations | Keep the benchmark as external *context for the approach class* and file it separately from the institution's own evaluation evidence ([§12.4](#124-the-honest-value-of-a-public-benchmark-inside-a-regulated-institution)) |
| An aggregate score over a heterogeneous benchmark family is treated as *the* score | Aggregates average across tasks that do not move together | Report the task subset that matches your workload. MTEB, BEIR, HELMET and LongBench are all families, not single tasks |

---

## 15. The Claims Audit

Every benchmark entry in this guide is traced to its **own** paper or the project's **own** page. Nothing in this guide is sourced to a blog, a README marketing line, a secondary summary, or recall.

### 15.1 Verified — traced to the source's own abstract or page

| Item | Source | Date | Read on / quality |
|------|--------|------|-------------------|
| BEIR — 18 datasets, 10 systems, BM25 robust baseline; re-ranking/late-interaction best zero-shot at high cost | Thakur et al., [arXiv:2104.08663](https://arxiv.org/abs/2104.08663) | 17 April 2021 | ✅ Verbatim abstract, arXiv API |
| BEIR official self-service leaderboard + no single authoritative reporting nexus before it | Kamalloo et al., [arXiv:2306.07471](https://arxiv.org/abs/2306.07471) | 13 June 2023 | ✅ Verbatim abstract |
| BEIR Touché 2020 — unjudged results, short-passage bias, denoising changes the comparison | Thakur et al., [arXiv:2407.07790](https://arxiv.org/abs/2407.07790) | 10 July 2024 | ✅ Verbatim abstract |
| BEIR-NL / Hindi-BEIR / BEIR-PL — translation limits, language coverage | Banar et al. [arXiv:2412.08329](https://arxiv.org/abs/2412.08329); Acharya et al. [arXiv:2408.09437](https://arxiv.org/abs/2408.09437); Wojtasik et al. [arXiv:2305.19840](https://arxiv.org/abs/2305.19840) | 11 Dec 2024 / 18 Aug 2024 / 31 May 2023 | ✅ Verbatim abstracts |
| MTEB — 8 tasks, 58 datasets, 112 languages, 33 models; no method dominates across tasks | Muennighoff et al., [arXiv:2210.07316](https://arxiv.org/abs/2210.07316) | 13 October 2022 | ✅ Verbatim abstract |
| MTEB maintenance — CI, dataset integrity, result generalisability, task-set growth | Chung et al., [arXiv:2506.21182](https://arxiv.org/abs/2506.21182) | 26 June 2025 | ✅ Verbatim abstract |
| MS MARCO — 1,010,916 questions, 182,669 rewritten answers, 8,841,823 passages, 3,563,535 documents, three tasks | Bajaj et al., [arXiv:1611.09268](https://arxiv.org/abs/1611.09268) | 28 November 2016 | ✅ Verbatim abstract |
| MS MARCO leaderboards — task modes, inherited labels, submission requirements, anti-P-hacking policy, **retirement/closing dates** | MS MARCO project page, [microsoft.github.io/msmarco](https://microsoft.github.io/msmarco/) | page read 25 September 2026 | ✅ Direct page read |
| MS MARCO survivorship bias — 38–45% unanswerable, score distortion, up to 9.9% / 3.5% penalties | Gupta et al., [arXiv:2204.12852](https://arxiv.org/abs/2204.12852) | 27 April 2022 | ✅ Verbatim abstract |
| Two MS MARCO corpora — title augmentation leaks relevance; unreportable version makes reproduction impossible | Lassance et al., [arXiv:2304.12904](https://arxiv.org/abs/2304.12904) | 25 April 2023 | ✅ Verbatim abstract |
| TREC DL 2020 — document + passage ad-hoc ranking in the large-data regime | Craswell et al., [arXiv:2102.07662](https://arxiv.org/abs/2102.07662) | 15 February 2021 | ✅ Verbatim abstract |
| TREC DL reusable collections — iteration/selection bias, best practices for reuse | Craswell et al., [arXiv:2104.09399](https://arxiv.org/abs/2104.09399) | 19 April 2021 | ✅ Verbatim abstract |
| MS MARCO / TREC DL evaluation design can encourage or discourage outcomes; internal/external validity | Craswell et al., [arXiv:2105.04021](https://arxiv.org/abs/2105.04021) | 9 May 2021 | ✅ Verbatim abstract |
| BRIGHT — 1,384 queries; MTEB retrieval leader at 59.0 nDCG@10 vs 18.3 on BRIGHT (as reported) | Su et al., [arXiv:2407.12883](https://arxiv.org/abs/2407.12883) | 16 July 2024 | ✅ Verbatim abstract; the comparison is the **paper's July 2024 snapshot** |
| TREC RAG 2024 — three tasks (R/AG/RAG), MS MARCO Segment v2.1, 301 topics, nuggetisation pipeline, organisers | Track site, [trec-rag.github.io/trec24](https://trec-rag.github.io/trec24/) | page read 25 September 2026 | ✅ Direct page read. `⚠` Track overview *paper* not verified |
| TREC RAG 2026 — two tasks, narratives, ClimbMix-400b corpus, Evalbase, RAG25 nuggets / UMBRELA qrels / ResearchRubrics / RAGDoll, organisers | Track site, [trec-rag.github.io](https://trec-rag.github.io/) | page read 25 September 2026 | ✅ Direct page read |
| RGB — four abilities, EN+ZH corpus, four testbeds, six LLMs; struggles on negative rejection, integration, false info | Chen et al., [arXiv:2309.01431](https://arxiv.org/abs/2309.01431) | 4 September 2023 | ✅ Verbatim abstract. `⚠` No counts stated in the abstract, so none are asserted |
| MultiHop-RAG — knowledge base from an English news dataset; queries + gold answers + supporting evidence; two experiments | Tang et al., [arXiv:2401.15391](https://arxiv.org/abs/2401.15391) | 27 January 2024 | ✅ Verbatim abstract. `⚠` No counts in the abstract |
| CRUD-RAG — Chinese; Create/Read/Update/Delete; criticises QA-only benchmarks and LLM-only evaluation; evaluates all components | Lyu et al., [arXiv:2401.17043](https://arxiv.org/abs/2401.17043) | 30 January 2024 | ✅ Verbatim abstract (truncated mid-"Delete"; nothing paraphrased beyond the read point). `⚠` No counts |
| RAGBench — 100k examples, five industry domains, industry corpora (e.g. user manuals), TRACe; fine-tuned evaluators beat LLM-based evaluation | Friel et al., [arXiv:2407.11005](https://arxiv.org/abs/2407.11005) | 25 June 2024 | ✅ Verbatim abstract |
| FRAMES — Factuality, Retrieval, And reasoning MEasurement Set; multi-hop; 0.40 no retrieval → 0.66 with multi-step retrieval | Krishna et al., [arXiv:2409.12941](https://arxiv.org/abs/2409.12941) | 19 September 2024 | ✅ Verbatim abstract; figures are the **paper's own 2024 baselines** |
| ARES — **evaluator framework**; context relevance/answer faithfulness/answer relevance; synthetic training data; PPI; eight tasks in KILT, SuperGLUE, AIS | Saad-Falcon et al., [arXiv:2311.09476](https://arxiv.org/abs/2311.09476) | 16 November 2023 | ✅ Verbatim abstract |
| KILT — knowledge-intensive tasks grounded in one Wikipedia snapshot | Petroni et al., [arXiv:2009.02252](https://arxiv.org/abs/2009.02252) | 4 September 2020 | ✅ Verbatim abstract |
| RAGTruth — nearly 18,000 RAG responses, case- and word-level hallucination annotation | Niu et al., [arXiv:2401.00396](https://arxiv.org/abs/2401.00396) | 31 December 2023 | ✅ Verbatim abstract |
| T²-RAGBench — 23,088 triples, text-and-table, context-independent (91.3% expert-validated), Hybrid BM25 best | Strich et al., [arXiv:2506.12071](https://arxiv.org/abs/2506.12071) | 4 June 2025 | ✅ Verbatim abstract |
| LIT-RAGBench — five generator categories, 114 Japanese questions + curated English version, fictional entities, abstention scored | Itai et al., [arXiv:2603.06198](https://arxiv.org/abs/2603.06198) | 6 March 2026 | ✅ Verbatim abstract |
| LongBench — 21 datasets, 6 categories, 6,711 words EN / 13,386 chars ZH average | Bai et al., [arXiv:2308.14508](https://arxiv.org/abs/2308.14508) | 28 August 2023 | ✅ Verbatim abstract |
| LongBench v2 — 503 MCQs, 8k–2M words, six categories, ~100 contributors, human 53.7%, best direct 50.1% | Bai et al., [arXiv:2412.15204](https://arxiv.org/abs/2412.15204) | 19 December 2024 | ✅ Verbatim abstract; figures are the **paper's December 2024 snapshot** |
| RULER — 17 models, 13 tasks; nearly perfect vanilla NIAH yet large drops with length; ~half fine at 32K | Hsieh et al., [arXiv:2404.06654](https://arxiv.org/abs/2404.06654) | 9 April 2024 | ✅ Verbatim abstract |
| HELMET — seven categories, up to 128K, model-based evaluation, 59 LCLMs; NIAH does not predict downstream; low inter-category correlation | Yen et al., [arXiv:2410.02694](https://arxiv.org/abs/2410.02694) | 3 October 2024 | ✅ Verbatim abstract (third finding truncated in the excerpt; only the NIAH-saturation part is asserted) |
| Natural Questions — real Google queries; 307,373 train / 7,830 dev (5-way) / 7,842 test (5-way); long + short answers | Kwiatkowski et al., TACL 7:452–466, [ACL Anthology Q19-1026](https://aclanthology.org/Q19-1026/) | 2019 | ✅ Direct page read 25 September 2026. **Not an arXiv paper**, so it was verified at the ACL Anthology |
| TriviaQA — 650K+ triples, 95K pairs, six evidence documents per question on average | Joshi et al., [arXiv:1705.03551](https://arxiv.org/abs/1705.03551) | 9 May 2017 | ✅ Verbatim abstract |
| HotpotQA — 113k Wikipedia-based pairs, sentence-level supporting facts, factoid comparison questions | Yang et al., [arXiv:1809.09600](https://arxiv.org/abs/1809.09600) | 25 September 2018 | ✅ Verbatim abstract |
| MuSiQue — bottom-up single-hop composition, 25K 2–4 hop questions, 3x human-machine gap, 30-point F1 drop for single-hop, MuSiQue-Full contrasts | Trivedi et al., [arXiv:2108.00573](https://arxiv.org/abs/2108.00573) | 2 August 2021 | ✅ Verbatim abstract |
| 2WikiMultiHopQA — structured + unstructured, evidence reasoning paths, template pipeline + Wikidata logical rules | Ho et al., [arXiv:2011.01060](https://arxiv.org/abs/2011.01060) | 2 November 2020 | ✅ Verbatim abstract |
| FEVER — 185,445 claims from altered Wikipedia sentences, three labels, Fleiss κ 0.6841, evidence sentences, 31.87% / 50.91% baselines | Thorne et al., [arXiv:1803.05355](https://arxiv.org/abs/1803.05355) | 14 March 2018 | ✅ Verbatim abstract |
| Benchmark Lottery — relative performance altered significantly by benchmark task choice | Dehghani et al., [arXiv:2107.07002](https://arxiv.org/abs/2107.07002) | 14 July 2021 | ✅ Verbatim abstract |
| MMLU-Redux — 6.49% of MMLU questions contain errors; 57% of the analysed Virology subset; metrics disagree with originally reported performance | Gema et al., [arXiv:2406.04127](https://arxiv.org/abs/2406.04127) | 6 June 2024 | ✅ Verbatim abstract |
| GSM1k — up to 8% accuracy drops; Spearman r² = 0.36; systematic overfitting in several families; frontier models show minimal signs | Zhang et al., [arXiv:2405.00332](https://arxiv.org/abs/2405.00332) | 1 May 2024 | ✅ Verbatim abstract |
| Retrieval train-test leakage — 69% of Robust04 queries near-duplicated; leakage can change system ranking | Fröbe et al., [arXiv:2206.14759](https://arxiv.org/abs/2206.14759) | 29 June 2022 | ✅ Verbatim abstract |
| Contamination effects in controlled pre-training; n-gram definitions inadequate | Jiang et al., [arXiv:2401.06059](https://arxiv.org/abs/2401.06059) | 11 January 2024 | ✅ Verbatim abstract |
| TS-Guessing — ChatGPT 52% / GPT-4 57% exact-match on masked MMLU options | Deng et al., [arXiv:2311.09783](https://arxiv.org/abs/2311.09783) | 16 November 2023 | ✅ Verbatim abstract |
| Data-contamination survey — definitions, effects, contamination-free evaluation strategies, white/grey/black-box detection | Cheng et al., [arXiv:2502.14425](https://arxiv.org/abs/2502.14425) | 20 February 2025 | ✅ Verbatim abstract |
| Kernel Divergence Score — near-perfect correlation with contamination levels in controlled scenarios | Choi et al., [arXiv:2502.00678](https://arxiv.org/abs/2502.00678) | 2 February 2025 | ✅ Verbatim abstract |
| Leakage in ML-based science — 17 fields, 329 papers, 8 leakage types | Kapoor & Narayanan, [arXiv:2207.07048](https://arxiv.org/abs/2207.07048) | 14 July 2022 | ✅ Verbatim abstract; **pre-LLM ML science**, used only as the norm that leakage must be checked |

### 15.2 Flagged — included with an explicit caveat

| Item | Flag |
|------|------|
| TREC RAG track overview papers (2024, 2025) | `⚠` Not verified. An arXiv full-text query for the track returned no matching entry, so the track's structure is cited to the **track's own site**, read 25 September 2026 |
| Benchmark *saturation* as a named phenomenon | `⚠` No canonical paper found. The saturation argument is assembled from sources that observe saturation in specific benchmarks (MMLU-Redux, RULER, HELMET, the Benchmark Lottery) and is labelled as assembled |
| Instance counts for RGB, MultiHop-RAG, CRUD-RAG and FRAMES | `⚠` Not stated in the abstracts read, and the full papers were not read. **No counts are asserted** |
| CRUD-RAG's "Delete" branch description | `⚠` The abstract truncates mid-sentence; nothing beyond the read point is paraphrased |
| MTEB leaderboard table | `⚠`/`❌` The leaderboard is a dynamic Hugging Face Space; reading it on 25 September 2026 returned no static table content. **No placement is reported from it** |
| A RAG-specific leaderboard Space hosted on Hugging Face | `⚠` A search result indicated a third-party RAG-leaderboard Space exists, but the page could not be read (it returns only a loading message). **Not described further, and nothing is asserted about it** |
| BRIGHT's 59.0 → 18.3 comparison | `⚠` Quoted as the **paper's own July 2024** reported figures. It is a historical snapshot, not a current ranking |
| LongBench v2's model/human figures; FRAMES's 0.40/0.66; MS MARCO's retirement dates | `⚠` All quoted with the source's own date. None is presented as current state |

### 15.3 Rejected — claims deliberately not made

| Rejected claim | Why |
|----------------|-----|
| Any **current** leaderboard ranking, top model or score | Rankings expire in weeks. This guide describes how leaderboards behave and cites only dated, self-reported, historical snapshots as evidence of the *mechanism* |
| That any named model was trained on any named benchmark | No source read for this guide asserts it. The literature distinguishes demonstrated leakage from suspicion, and this guide keeps that boundary |
| That benchmark contamination explains any specific leaderboard transition | Not established by any source read |
| That "n-gram overlap found none" constitutes a clean contamination check | Contradicted by a demonstrated source ([arXiv:2401.06059](https://arxiv.org/abs/2401.06059)) |
| Any dataset size, corpus size, question count or metric score not read at the source | Inventing a plausible figure is worse than an absent one |
| That a public benchmark can be a release gate, a control, or a validation artifact | The central argument of [§8](#8-why-a-retrieval-score-is-not-a-rag-score), [§10](#10-how-to-use-a-public-benchmark-responsibly) and [§12](#12-the-regulated-institution-angle) |

### 15.4 The four names that look like coverage and are not

A coverage grep for this guide's subject matter over this repository returns hits that naively look like pre-existing benchmark coverage. Each was checked in context. **All four are false positives of the same class:** a benchmark name that collides with a company, a colour space, a decentralised-identifier method, and an ordinary English word respectively. (This repository has a standing pattern of exactly these collisions — earlier examples include rust/trust, CAMS/scams, iLO/pilot, ISIN/-ising, and Antithesis the rhetorical term.)

| Trap | Hits | What every hit actually is | The benchmark it is **not** |
|------|------|----------------------------|-----------------------------|
| `\bARES\b` (case-insensitive word boundary) | **4 files** — `banking/asset_backed_trading_guide.md`, `banking/sun_venture_guide.md`, `banking/singapore_private_markets_guide.md`, `banking/asset_management_alternatives_guide.md` | The **asset manager Ares Management** — verified in context (a CLO-manager list reads "Ares, KKR, BlackRock, CIFC, Carlyle, Seix…" and another file reads "later acquired by Ares Management"). Case-sensitively, `\bARES\b` matches **0** files | The RAG evaluation framework ARES ([arXiv:2311.09476](https://arxiv.org/abs/2311.09476)) |
| `\bRGB\b` | **3 files** — `technology/video_editor_software_ubuntu.md`, `technology/remote_sensing_technologies_guide.md`, `technology/physical_ai_guide.md` | The **RGB colour space / colour model and RGB camera bands** — colour grading, remote-sensing band composition, and camera sensors | The RAG benchmark RGB ([arXiv:2309.01431](https://arxiv.org/abs/2309.01431)) |
| `\bKILT\b` (case-insensitive word boundary) | **1 file** — `technology/w3c_dids_guide.md` | The **`did:kilt` DID method of the KILT Protocol** — a decentralised identifier method, not a dataset | The knowledge-intensive benchmark KILT ([arXiv:2009.02252](https://arxiv.org/abs/2009.02252)) |
| `FRAMES` | **203 files** case-insensitive *substring* across all file types; **154** case-insensitive word-boundary in `*.md`; **2** case-sensitive `\bFRAMES\b` in `*.md` | The **ordinary English word "frames"** — a table column heading, "time frames", "value frames", and similar | The FRAMES RAG benchmark ([arXiv:2409.12941](https://arxiv.org/abs/2409.12941)) |

**Counts with and without the traps.** The four patterns together produce **207 distinct file hits** (union, de-duplicated) across a repository of **622 markdown files** — of which **zero** are the benchmark they appear to name. Excluding the traps, i.e. counting only genuine public-benchmark coverage, the count is **0 files**. That null result is the verified gap this guide fills, and it is why [§1.4](#14-the-boundary-this-guide-declares) can state the boundary as a fact rather than a claim.

`⚠` **Discrepancy recorded.** The dispatcher's pre-check reported the FRAMES pattern at "203 files" and this audit reproduces 203 for a case-insensitive substring match across all file types, but 154 for a case-insensitive word-boundary match restricted to markdown. Both are reported above rather than reconciled to one number. `⚠` The dispatcher recorded `\bARES\b` as 4 files without noting that this requires case-insensitive matching; case-sensitively it is 0. Both readings are recorded.

### 15.5 Tooling note for this audit

**`web_search` limitation.** The dispatcher reported that `web_search` was returning empty result sets on this host, and instructed that this be recorded as a **tool limitation, not evidence of absence**. In this session, the single `web_search` call attempted returned non-empty results — so the reported behaviour did not reproduce. It makes no difference to the audit either way: **no benchmark fact in this guide was sourced from a search snippet.** Every benchmark fact is traced to its own arXiv abstract via the **arXiv API over HTTPS** (`https://export.arxiv.org/api/query`, always requiring `<feed>` entries to be confirmed before a result was accepted) or to the project's/leaderboard's **own page read directly** (MS MARCO, the TREC RAG track site, the ACL Anthology page for Natural Questions). The two pages that could not be read at all — the MTEB leaderboard Space and a third-party RAG-leaderboard Space — returned only a loading message and are recorded as `⚠`/`❌` above rather than filled from memory.

---

## 16. What Could Not Be Verified, Glossary, Cross-References and Closing Summary

### 16.1 What Could Not Be Verified

Grouped by cause, so that the shape of the gap is legible.

**Could not be verified because the source does not state it (and no figure was invented):**

- Instance counts, corpus sizes and document counts for **RGB**, **MultiHop-RAG**, **CRUD-RAG** and **FRAMES** — the abstracts do not state them and the full papers were not read. `❌`
- The full text of the third finding in **HELMET**'s abstract (truncated in the excerpt read). Only the NIAH-saturation part is asserted. `⚠`
- The complete "Delete" branch description in **CRUD-RAG**'s abstract (truncated mid-sentence). Nothing beyond the read point is paraphrased. `⚠`
- Whether the **TREC RAG** track has published formal overview papers for its 2024 and 2025 iterations — an arXiv full-text query returned no matching entry, so the track is cited to its own site. `⚠`/`❌`

**Could not be read because the tool could not render the page:**

- The **MTEB leaderboard** (Hugging Face Space `spaces/mteb/leaderboard`) — returns a JS application shell with no extractable table content. No placement is reported from it. `❌`
- A third-party **RAG-leaderboard Space** surfaced by search — same rendering issue. Not described, nothing asserted. `⚠`
- The **BEIR official leaderboard** page was not independently read; its existence and purpose are cited to the paper that announced it ([arXiv:2306.07471](https://arxiv.org/abs/2306.07471)). `⚠`

**Could not be verified as a coherent literature:**

- **Benchmark saturation** as a named, canonical phenomenon. No single paper was found; the argument in [§7.5](#75-saturation) is assembled from sources that observe saturation in specific benchmarks and is labelled as such. `⚠`
- Any **claim that a specific model was trained on a specific benchmark**. No source read asserts it; the literature read divides into demonstrated leakage effects, detection methods, and general concern. `❌`

**Tool limitations recorded:**

- **`web_search`**: reported by the dispatcher as returning empty result sets on this host. In this session a single call returned non-empty results, so the reported behaviour did not reproduce. **It was not used as a source for any benchmark fact** — the substitute was the **arXiv API over HTTPS plus direct page reads**, which is also the more reliable route because it reaches the primary source. Note the operational detail worth carrying forward: the arXiv API returns **HTTP 406** to a non-browser User-Agent header, so a user agent must be supplied or the call fails in a way that can look like an empty result.
- **`web_extract`**: worked on real pages (MS MARCO project page, TREC RAG track sites, the ACL Anthology NQ page). It **failed to render the two JavaScript-only Hugging Face Spaces** listed above, and that failure is recorded rather than papered over.
- **Plain HTTP** to the arXiv API fails silently; every query in this audit used HTTPS and confirmed a `<feed>` with entries before any result was accepted.

**Not attempted / out of scope for this guide:**

- Reproducing any benchmark result. This guide reads benchmarks; it does not run them.
- Any current leaderboard state, for any leaderboard. Deliberately excluded — see [§6.4](#64-the-honest-limits-of-any-leaderboard).

### 16.2 Glossary

| Term | Definition |
|------|------------|
| **Benchmark** | A named evaluation programme: task definition + dataset(s) + scoring rule + usually a leaderboard. BEIR, MTEB, RGB and FRAMES are benchmarks. |
| **Dataset** | A concrete collection of instances with labels or gold answers, versioned as an artifact. |
| **Corpus** | The frozen document collection a retrieval task searches. Not your corpus. |
| **Split** | Train / dev / test partition. Test splits are withheld; dev splits are public by design and are a legitimate engineering target. |
| **Track** | The rule-set within a benchmark — zero-shot vs fine-tuned, retrieval-only vs end-to-end, supplied-context vs open-corpus. |
| **Leaderboard** | The public results table for a benchmark. A snapshot service, not a fact. |
| **Contamination** | Overlap between benchmark instances and a system's training data. Demonstrated cases exist; general suspicion is not a demonstration. |
| **Saturation** | The state where a benchmark no longer separates systems, because of a ceiling, measurement error, or the yardstick's own construction. |
| **Proxy benchmark** | A benchmark measuring something adjacent to RAG (long-context, multi-hop QA, fact verification) that gets borrowed as RAG evidence. |
| **Zero-shot track** | Evaluation of a model on a task it was not trained on — the setting BEIR is built for. |
| **nDCG@k / Recall@k / MRR@k** | Retrieval metrics and their cutoffs. **Definitions: [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §2.** |
| **TRACe** | RAGBench's explainable, actionable RAG evaluation framework. |
| **PPI (prediction-powered inference)** | ARES's technique for combining a small human-annotated set with model predictions to reduce prediction error. |
| **Nuggetisation** | The TREC RAG track's process of decomposing pooled retrieved content into "nuggets" that can be assigned to answer sentences. |
| **UMBRELA / ResearchRubrics / RAGDoll** | TREC RAG 2026 track materials: an assessment approach, a rubric resource, and an automated end-to-end evaluation toolkit. |
| **Benchmark lottery** | The finding that relative algorithm performance can be altered significantly by choosing different benchmark tasks ([arXiv:2107.07002](https://arxiv.org/abs/2107.07002)). |
| **MPPI / sanity floor / shortlist generator** | This guide's terms for the two legitimate uses of a public benchmark: disqualify cheaply, and reduce fifty candidates to three. |

### 16.3 Cross-references

**Owned by this guide:** the public benchmark and leaderboard landscape for retrieval and RAG — which benchmarks exist, what each measures, how each was built, what is documented as wrong with them, how to read a leaderboard claim, and how to use public benchmarks responsibly.

**Owned elsewhere in this repository — cite, do not re-derive:**

| Subject | Owned by |
|---------|----------|
| Metric definitions; the three evaluation layers; reference-based vs reference-free; test-set design and the golden-dataset method; offline vs online; the evaluation pipeline; RAG failure modes | [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md) §1–§9 |
| The RAG evaluation tool landscape (RAGAS, TruLens, DeepEval, LangSmith, Arize Phoenix) | [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md) |
| Tool deep-dives | [ragas_guide.md](ragas_guide.md) · [trulens_guide.md](trulens_guide.md) · [deepeval_guide.md](deepeval_guide.md) |
| Embedding and vector-database selection (where BEIR/MTEB appear as selection evidence) | [vector_databases_guide.md](vector_databases_guide.md) |
| Framework selection | [rag_frameworks_comparison_guide.md](rag_frameworks_comparison_guide.md) |
| Long-context vs retrieval (LongBench, RULER, HELMET, InfiniteBench in that framing) | [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md) |
| The retrieval techniques benchmarks measure (and HotpotQA / MultiHop-RAG as evaluation vehicles) | [advanced_rag_techniques_guide.md](advanced_rag_techniques_guide.md) · [agentic_search_vs_rag_guide.md](../agentic_search_vs_rag_guide.md) |
| Model risk, validation versus evaluation, governance operating model, red-teaming, responsible AI | [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) · [ai_governance_framework_guide.md](../ai_governance_framework_guide.md) · [ai_governance_bias_redteaming_guide.md](../ai_governance_bias_redteaming_guide.md) · [llm_evaluation_frameworks_guide.md](../llm_evaluation_frameworks_guide.md) · [implementing-responsible-ai.md](../implementing-responsible-ai.md) |

### 16.4 Closing summary

- A **public benchmark measures a benchmark-shaped task on a fixed corpus.** It can rank approaches. It cannot validate a system.
- **Use benchmarks for two jobs:** disqualify candidates that are plainly behind a strong baseline, and shrink a field of fifty to a field of three. Then stop.
- **Read every claim through the eleven questions** ([§9.1](#91-the-eleven-questions)) — which benchmark, which version, which split, which track, which metric, which baseline, read when, judged by whom, at what cost, published where, and what it does not cover. A claim missing a line is a lead, not evidence.
- **Never confuse the layers.** BEIR and MTEB score retrieval. RGB, CRUD-RAG and MultiHop-RAG score diagnostically. RAGBench, FRAMES and the TREC RAG track score end-to-end. ARES and RAGTruth are how you score and how labels are made. A number from one layer is not a number about another.
- **Keep contamination honest.** Demonstrated findings get a source and a date; suspicions get the word *suspected*. Never assert that a named model trained on a named benchmark.
- **Know that names collide.** ARES is also an asset manager, RGB is also a colour space, KILT is also a DID method, and frames are also what tables have.
- **In a regulated institution, a leaderboard placement is external context, never evidence.** The release decision belongs to a versioned, owned, unpublished golden set evaluated against your corpus, your queries and your failure costs — and that is the one asset no public benchmark can supply.
- And the rule that governs all of it: a benchmark is a yardstick, a deployment is a process, and the two are not interchangeable.

*Series note: this guide is the public benchmark and leaderboard landscape — the dataset side of RAG evaluation. The metric definitions, the evaluation layers and the golden-dataset method live in [rag_evaluation_methodology_guide.md](rag_evaluation_methodology_guide.md); the tools that run them live in [rag_evaluation_tools_comparison_guide.md](rag_evaluation_tools_comparison_guide.md) and the tool deep-dives; the embedding-selection use of BEIR and MTEB lives in [vector_databases_guide.md](vector_databases_guide.md); the long-context comparison lives in [rag_vs_long_context_llms_guide.md](rag_vs_long_context_llms_guide.md); and the assurance framing that a benchmark placement cannot satisfy lives in [llm_evaluation_vs_validation_guide.md](../llm_evaluation_vs_validation_guide.md) and [ai_governance_framework_guide.md](../ai_governance_framework_guide.md).*

**Closing rule — a benchmark can rank a model; only your own evaluation can certify a system:** a benchmark is not your use case.
