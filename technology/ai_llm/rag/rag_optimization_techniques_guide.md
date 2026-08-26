# RAG Optimization Techniques — The Practical Playbook

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG vs HyDE](rag_vs_hyde_guide.md) · [Query Rewriting in RAG](query_rewriting_rag_guide.md) · [LLM Latency Optimization](../llm_latency_optimization_guide.md) · [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md)
> **Last Updated:** July 2026

---

## Table of Contents

1. [Introduction: Optimization Is a Process, Not a Tweak](#1-introduction-optimization-is-a-process-not-a-tweak)
2. [The RAG Optimization Mindset](#2-the-rag-optimization-mindset)
3. [RAG Failure Taxonomy — Where Things Go Wrong](#3-rag-failure-taxonomy--where-things-go-wrong)
4. [The RAG Optimization Roadmap — Five Phases](#4-the-rag-optimization-roadmap--five-phases)
5. [Indexing Optimization](#5-indexing-optimization)
6. [Retrieval Optimization](#6-retrieval-optimization)
7. [Generation Optimization](#7-generation-optimization)
8. [Latency and Cost Optimization](#8-latency-and-cost-optimization)
9. [Observability and Monitoring](#9-observability-and-monitoring)
10. [Evaluation](#10-evaluation)
11. [The Optimization Playbook by Symptom](#11-the-optimization-playbook-by-symptom)
12. [Banking-Specific RAG Optimization](#12-banking-specific-rag-optimization)
13. [The 80/20 Optimization List](#13-the-8020-optimization-list)
14. [Conclusion](#14-conclusion)

---

## 1. Introduction: Optimization Is a Process, Not a Tweak

A RAG system that works *sometimes* is easy to build. A RAG system that works *reliably* — correct, grounded, fast, and cheap, across a changing corpus and a changing query mix — is hard. The gap between the two is not closed by a single clever trick. It is closed by a **systematic optimization process**: measuring where the pipeline fails, forming a hypothesis, changing one variable, and re-measuring until the failure is gone or the cost of further work exceeds the benefit.

This guide is the practical playbook for that process. It walks the entire pipeline — **indexing → retrieval → generation → evaluation** — and for each stage covers:

- **What can go wrong** and how to recognize it from symptoms.
- **What to change**, in order of leverage, with concrete parameters and tooling.
- **How to measure** whether a change actually helped.

It is deliberately practical: configuration values, tool names, code sketches, and decision rules rather than theory. It assumes you are already running a RAG pipeline (built on LangChain, LlamaIndex, Haystack, or a managed service — see the [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) for stack selection) and now need to make it *good*.

**Scope and companion guides.** This playbook covers the full optimization cycle end-to-end. Three sibling guides go deeper on specific levers and are referenced where relevant rather than duplicated: [Query Rewriting in RAG](query_rewriting_rag_guide.md) (pre-retrieval query transformations), [RAG vs HyDE](rag_vs_hyde_guide.md) (hypothetical-document embeddings), [LLM Latency Optimization](../llm_latency_optimization_guide.md) (model- and serving-level latency techniques), and [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) (judge models, evaluation harnesses, CI integration).

### 1.1 The core truth: RAG quality is a chain, and chains break at the weakest link

Every RAG failure has a *location* in the pipeline. The single most important habit in RAG optimization is **attributing the failure correctly before touching anything**:

| Failure location | Example symptom | Typical root cause |
|---|---|---|
| Parsing | Answer drops table numbers | PDF table extracted as garbage text |
| Chunking | Context is a fragment of a regulation | Fixed-size splitter cut mid-clause |
| Embedding | Wrong domain docs retrieved | Generic embedding model |
| Retrieval | Correct docs ranked at position 12 | No reranker, bad top-k |
| Prompt/context | Model ignores a relevant chunk | Relevant chunk buried in the middle |
| Generation | Answer not grounded in any chunk | Weak grounding instructions, high temperature |
| Evaluation | "It works" with no evidence | No golden dataset, no metrics |

Optimize in **pipeline order** — index quality first, then retrieval, then generation — because each stage's ceiling is set by the stage before it. A perfect reranker cannot rescue chunks destroyed at parse time; a perfect prompt cannot rescue a retrieval stage that returns the wrong documents.

---

## 2. The RAG Optimization Mindset

Optimization fails more often from bad process than from bad technique. Five disciplines separate teams that steadily improve their RAG systems from teams that flail:

### 2.1 Establish a baseline first — measure before you change anything

You cannot know whether a change helped unless you know the numbers before it. Before the first optimization, run the pipeline as-is against your evaluation set and record:

- **Retrieval quality:** recall@k, precision@k, MRR, nDCG@k, hit rate (see §9.2 for definitions).
- **Answer quality:** faithfulness, answer relevance, context precision/recall (RAGAS — see §10.2).
- **Latency:** p50/p95 end-to-end, and the per-stage breakdown (embedding, search, rerank, generation).
- **Cost:** tokens per query (input + output), embeddings cost, LLM calls per query.

Write these numbers in a baseline report — a file in the repo, a dashboard, a spreadsheet. Every subsequent optimization records its *before* and *after* against this baseline. Without it, you are doing vibes-based engineering.

### 2.2 Change one variable at a time

The pipeline has dozens of knobs: chunk size, overlap, embedding model, top-k, similarity threshold, reranker, prompt template, temperature, context ordering. Change **one** per experiment cycle.

Why: if you change chunk size *and* the reranker in the same deploy, and quality improves, you do not know which change (or interaction) caused it — and if quality degrades, you cannot roll back intelligently. One-variable-at-a-time is slower per cycle but converges far faster overall, because every result is interpretable. The exception is bundled "known-good" upgrades (e.g., upgrading the embedding model and re-indexing — those are inseparable).

**Practical rhythm:** one change → re-index if needed → run the golden dataset → compare against baseline → record result → keep or revert → next change. A change is *kept* only if it improves the metrics you care about without regressing others (a reranker that improves faithfulness but doubles latency may still be a win — if latency is inside budget).

### 2.3 Use a golden dataset for evaluation

All optimization decisions rest on measurement, and measurement rests on a **golden dataset**: a curated set of realistic queries with known-good answers and the document IDs that contain the ground truth. Details in §10.1. The essential rules:

- **Curate, don't scrape:** 50–200 hand-validated question–answer pairs that represent how users actually ask (plus deliberate edge cases).
- **Include the hard cases:** multi-hop questions, questions whose answer is a table, negations ("which products are NOT covered"), acronyms, regulatory codes.
- **Version it** like code; **update it** quarterly as the corpus and query mix evolve.
- **Run it on every change** — automated, ideally in CI, so optimizations are continuously regression-tested.

Without a golden dataset, every optimization decision is a guess and every regression goes unnoticed until users complain.

### 2.4 Track the whole pipeline, not just the answer

A RAG system has four independent quality dimensions, and optimizing one can silently destroy another:

1. **Retrieval quality** — did we find the right chunks?
2. **Generation quality** — is the answer correct, grounded, and useful?
3. **Latency** — how long did the user wait?
4. **Cost** — what did it cost per query?

Track all four in every experiment. A common trap: improving retrieval recall by raising top-k to 50 dilutes the context, degrades faithfulness, blows the token budget, and triples cost — while "retrieval metrics" look great. The golden dataset must therefore be scored on retrieval *and* generation, and you must record latency and cost per run.

### 2.5 Treat the corpus as a living input

Optimization is not a one-time project. The corpus changes (new regulations, updated policies, removed documents), the query mix drifts (a new product launch changes what people ask), and the models underneath get upgraded. Schedule optimization as a **recurring loop**: quarterly golden-dataset refresh, continuous monitoring (§9), and a standing backlog of hypothesis-driven experiments. Systems that are not periodically re-measured quietly rot.

**Checklist before any optimization cycle:**

- [ ] Golden dataset exists, is versioned, and is current.
- [ ] Baseline metrics (retrieval + generation + latency + cost) recorded.
- [ ] Observability is capturing per-query logs (§9.1) so failures can be traced.
- [ ] One change per experiment, with before/after recorded.
- [ ] Rollback path known (config versioned, index rebuildable).

---

## 3. RAG Failure Taxonomy — Where Things Go Wrong

Before optimizing, learn to *diagnose*. Nearly every RAG failure falls into one of seven categories. The playbook in §11 maps symptoms to these root causes, so internalize the taxonomy now.

### 3.1 Poor chunking → context fragmentation

The splitting strategy destroys the semantic units of the document: a sentence split across two chunks, a regulation clause cut mid-list, a table torn in half. Retrieval then can never return the *complete* unit of meaning, so the LLM gets fragments and produces a wrong or incomplete answer.

- **Signature:** the correct document is retrieved, but the chunks are partial or incoherent; answers miss the second half of a clause; citation points at a chunk that contains only half the claim.
- **Where it bites:** fixed-size character splitters on legal prose, dense paragraphs, or tables.

### 3.2 Wrong chunks retrieved (retrieval failure)

The retriever returns documents that are topically related but do not contain the answer. Causes: weak embedding model for the domain, query phrasing that does not match document vocabulary, pure dense search missing exact terms, low top-k, or a vector store misconfigured (wrong distance metric, bad HNSW params).

- **Signature:** retrieved chunks "look related" but the answer is not in them; the model hallucinates or says "not found" when the answer *is* in the corpus.
- **Where it bites:** domain jargon, product codes, error messages, regulatory identifiers.

### 3.3 Irrelevant context in the prompt (noise)

The prompt contains chunks that are irrelevant to the query. Noise hurts in three ways: it crowds out relevant chunks (token budget), it distracts the model (LLMs do use irrelevant context, especially when it contradicts the answer), and it inflates cost.

- **Signature:** answer is *mostly* right but includes details pulled from an off-topic chunk; or the model hedges because contradictory chunks are present.
- **Where it bites:** broad top-k, no reranking, no similarity threshold, no metadata filtering.

### 3.4 Missing context (recall failure)

The information needed to answer simply never made it into the context. The chunk is in the index but not retrieved (query phrasing mismatch, embedding failure), or it was never indexed (parse failure, dedup removed it, the document is stale).

- **Signature:** the model says "I don't have that information" or answers with outdated info, and manual search of the corpus finds the answer in a chunk the retriever did not return.
- **Where it bites:** synonyms, paraphrased queries, exact-code queries against a dense-only index, recently added documents not yet indexed.

### 3.5 Answer not grounded in context (hallucination)

The model generates content not supported by the retrieved context. Causes: weak grounding instructions, high temperature, prompt overrides ("ignore the context"), or — critically — **the context genuinely lacks the answer**, in which case the model prefers to guess over admitting ignorance.

- **Signature:** confident, fluent answer containing facts, figures, or citations not traceable to any retrieved chunk.
- **Where it bites:** factual queries, numbers, regulatory requirements — the highest-stakes failure mode in banking.

### 3.6 Context overload (token limits, dilution)

Too much context: the prompt exceeds the model's context window (truncation errors, dropped chunks) or contains so many chunks that relevant information is diluted and the model's attention is spread thin (the *lost-in-the-middle* effect, §7.2).

- **Signature:** errors about exceeding context length; answers that ignore the key chunk; quality drops when top-k is raised even though recall improved.
- **Where it bites:** high top-k, huge chunks, no token budgeting.

### 3.7 Citation errors

Citations point at the wrong document, the wrong section, or a chunk that does not support the claim. Causes: chunk metadata lost at parse/index time (no page numbers, no section path), citation markers generated by the LLM from context order rather than actual source, or dedup/merging that broke the chunk→source mapping.

- **Signature:** user clicks a citation and lands on the wrong page; an auditor cannot trace a claim to its source; sources cited that were not retrieved.
- **Where it bites:** compliance-sensitive use cases where traceability is mandatory.

### 3.8 How to localize a failure fast

When an answer is bad, **ask where it broke** in this order:

1. **Is the answer in the retrieved context?** (Inspect the actual prompt sent to the LLM.) No → retrieval or indexing failure (§3.2–3.4). Yes → generation failure (§3.5–3.7).
2. **Is the chunk in the index?** Query the vector store directly for the document/ID. No → indexing failure (parse, chunk, ingest).
3. **Would a human find the chunk for this query?** Search with BM25 and with the embedding. Only one finds it → fix the other half of hybrid search.
4. **Did the LLM use the chunk correctly?** Yes but answer is still wrong → prompt/generation issue.

This four-step localization, done on real failing queries, tells you which section of this guide to read.

---

## 4. The RAG Optimization Roadmap — Five Phases

Do not optimize randomly. Run the roadmap in order; each phase hands the next a solid foundation.

### Phase 0 — Baseline and observability

**Goal:** know your current numbers and be able to see failures.

- Instrument the pipeline: log every query end-to-end — raw query, rewritten query (if any), retrieved chunk IDs + scores, rerank scores, assembled context (token count), final answer, latency per stage, token usage, cost (§9.1).
- Add tracing (LangSmith, Langfuse, Arize Phoenix, OpenLLMetry) so any individual query can be replayed and inspected.
- Build the golden dataset (50–200 pairs) and a repeatable evaluation script.
- Record the baseline: retrieval metrics, generation metrics, latency p50/p95, cost/query.
- **Exit criteria:** you can answer "what is the recall@5, faithfulness, p95 latency, and cost per query of the current system?" from a dashboard or report.

### Phase 1 — Indexing optimization

**Goal:** make the index contain clean, complete, well-structured units.

- Fix document parsing: PDF quality, tables, OCR for scans, boilerplate removal, dedup (§5.1).
- Choose a chunking strategy that respects document structure; tune size/overlap per corpus (§5.2).
- Enrich metadata (source, date, doc type, section path, page) and enable filtering (§5.3).
- Select and benchmark the embedding model; consider multilingual and dimension trade-offs (§5.4).
- **Exit criteria:** a human spot-checking retrieved chunks for golden queries finds them complete and coherent; recall@k improves measurably over the Phase 0 baseline.

### Phase 2 — Retrieval optimization

**Goal:** find the right chunks reliably.

- Move from dense-only to **hybrid retrieval** (dense + BM25 + RRF) — the single biggest retrieval win for most corpora (§6.1).
- Add query transformation where queries are complex or paraphrase-heavy: rewriting, HyDE, multi-query, step-back, decomposition (§6.2).
- Add a **reranker** between retrieval and context assembly to lift precision (§6.3).
- Tune retrieval parameters: top-k (recall-oriented for rerank, precision-oriented for generation), similarity threshold, MMR, HNSW/BM25 internals (§6.4).
- **Exit criteria:** for golden queries, the answer-bearing chunk is in the top 1–3 of the final context; retrieval metrics improved and stayed improved across the query-type breakdown.

### Phase 3 — Generation optimization

**Goal:** turn good context into good answers.

- Engineer the prompt: grounding instructions, context formatting with delimiters and numbered sources, few-shot examples, citation requirements, uncertainty handling (§7.1).
- Manage the context: selection, ordering (most relevant first), dedup, compression, token budgeting, adaptive k (§7.2).
- Enforce output structure: JSON schema / function calling, citation markers, length and tone constraints (§7.3).
- Where appropriate, adopt RAG-specific generation patterns: LLM re-ranking of retrieved chunks, self-reflection/self-RAG, corrective RAG, agentic retrieval (§7.4).
- **Exit criteria:** faithfulness and answer relevance scores on the golden set reach your target; hallucination-type failures are rare; citations trace to real chunks.

### Phase 4 — Evaluation and iteration

**Goal:** institutionalize measurement so the system keeps improving.

- Automate golden-dataset regression runs on every pipeline change (§10.3).
- Run A/B tests for material changes (new embedding model, new reranker) against real traffic with holdout users (§10.4).
- Monitor online: user feedback, retrieval-failure rate, hallucination proxies, latency p95, cost (§9.3–9.5).
- Refresh the golden dataset quarterly; keep a prioritized backlog of experiments.
- **Exit criteria:** the loop is running — changes are proposed, measured, and either promoted or reverted, with an audit trail (especially relevant under banking model-risk governance, §12.4).

---

## 5. Indexing Optimization

Indexing quality is the ceiling for everything downstream. Retrieval can only return what the index contains, in the units the index stored. Most "retrieval is bad" problems are actually indexing problems. Work through this section top to bottom: parse, chunk, metadata, embed.

### 5.1 Document parsing and cleaning

Garbage in, garbage out — and in RAG, "garbage" usually means: text that lost its structure (tables flattened, columns merged), OCR noise in scanned documents, boilerplate polluting every chunk, and duplicates diluting the index.

#### PDF parsing quality

Not all PDF extractors are equal. The choice of extractor is a first-class optimization decision, especially for banking corpora (prospectuses, regulatory notices, research PDFs):

| Tool | Strength | Weakness | Use when |
|---|---|---|---|
| **PyMuPDF (fitz)** | Very fast, good text and basic layout, low dependency weight | Weak on complex layouts and tables; no OCR | Clean digital PDFs, high throughput, simple layouts |
| **pdfplumber** | Excellent for tables and coordinates; precise layout analysis | Slower; needs per-document tuning for complex layouts | Tabular regulatory/financial documents |
| **marker-pdf** | Converts PDF → clean Markdown; strong table handling; vision-assisted; high-quality output | Heavier (ML models), slower, GPU helps | Mixed-layout documents, Markdown downstream, high quality bar |
| **LlamaParse** (managed) | API-based; handles tables, charts, and messy layouts; returns structured Markdown | Cost per page; data leaves your infra (assess for banking) | Complex heterogenous documents, quick wins without local tuning |
| **unstructured** | Library of parsers for many formats (PDF, HTML, DOCX, PPT, email); partitioning into elements | Quality varies by backend; configuration surface is large | Heterogeneous ingestion pipelines, quick connectors |

**Practical rule:** start with PyMuPDF for digital PDFs; if tables or two-column layouts come out garbled, try pdfplumber for tables and marker-pdf (or LlamaParse) for the rest. Always spot-check parse quality on 10–20 representative documents — parse errors are silent killers.

#### Table extraction

Tables are the hardest content type for RAG, and financial documents are full of them. Two failure modes: the extractor destroys the table (columns run together, values land in the wrong cells), or the chunker destroys it later (a table split across chunks).

- **Complex tables → Markdown or HTML.** Convert tables to a structured text representation an LLM can read — Markdown tables or HTML `<table>` — rather than raw text dumps. Keep the header row with the body (repeat headers if a table spans pages).
- **Vision models for complex layouts.** Multi-row headers, merged cells, rotated text, and chart-embedded numbers defeat rule-based extractors. Layout-aware / vision models (marker-pdf, LlamaParse, PaddleOCR's PP-Structure) handle these; for critical documents, render the page and have a VLM transcribe the table, validating against a sample.
- **Decide the chunk unit:** a table should live in one chunk (or be retrievable as a unit — see parent-document retrieval in §5.2). Never let a chunk boundary cut through a table mid-row.

#### OCR for scanned documents

Scanned PDFs have no text layer; extraction without OCR returns nothing. Options:

- **Tesseract** — free, mature, works everywhere; fine for clean scans; weaker on dense or degraded layouts; produces text without layout structure.
- **PaddleOCR** — strong accuracy, especially for dense/mixed-language documents; PP-Structure adds table and layout recovery; heavier to deploy.
- **DocTR** (mindee) — deep-learning OCR with decent layout analysis; good accuracy/latency balance.
- **OCR + layout + vision pipelines** (marker-pdf, LlamaParse) — best end-to-end quality for messy scans, at higher compute/cost.

**OCR checklist:** detect scanned pages first (no text layer → route to OCR); keep page order and add page numbers to metadata; run a quality gate (e.g., word-error-rate estimate on a sample); weigh OCR cost against retrieval value for archival documents.

#### Removing boilerplate

Headers, footers, page numbers, running titles, navigation menus (in HTML), disclaimers, and repeated boilerplate pollute chunks: they waste tokens, create near-duplicate chunks, and get retrieved for the wrong reasons. Clean at parse time:

- Strip page furniture (headers/footers/page numbers) before or during chunking — PyMuPDF and pdfplumber expose text blocks so you can drop header/footer regions.
- For HTML, remove nav, footer, and script/style elements with the parser (BeautifulSoup/trafilatura) before chunking.
- Remove repeated disclaimers/boilerplate paragraphs (detect strings that recur across many documents).
- Keep *meaningful* headers (section titles) — they are structure, not furniture.

#### Deduplication (exact and near-duplicate)

Duplicate documents silently halve retrieval precision: the same content appears in multiple chunks, crowding out distinct information and producing redundant context. Dedup at ingest:

- **Exact duplicates:** hash document content (SHA-256) and drop repeats. Cheap, do it always.
- **Near-duplicates:** versioned policies, "final_v2.pdf" vs. "FINAL_final.pdf", the same notice on two intranets. Use MinHash/LSH on shingled text to cluster near-duplicates, or embedding similarity with a threshold (~0.95+ for near-copies). Keep the canonical version (newest, most complete, or from the authoritative source) and record the alias in metadata.

#### Language detection and translation

- **Detect language per document** (fastText lid.176, langdetect) and store it in metadata so retrieval can filter and so you know which embedding model to use (§5.4 multilingual).
- **Translation policy:** decide deliberately whether to translate non-English docs at ingest. Translating *before* indexing lets one embedding model and one LLM serve a multilingual corpus; the cost is translation quality loss and latency/cost at ingest. For banking, regulatory text should be indexed in its original language (legal precision), with translation only as a retrieval aid — or use a multilingual embedding model (§5.4) and let the LLM answer in the query's language.

### 5.2 Chunking strategies

Chunking determines the *units* retrieval searches over and the *units* the LLM reads. There is no universal best chunker — only the best chunker for your corpus and query types. Test on your golden dataset (§10.1) and let recall@k decide.

#### Fixed-size chunking

Split on a fixed token count. Simple, fast, deterministic — but it **cuts through sentences, paragraphs, and semantic units**, fragmenting meaning (§3.1). Use it only as a baseline or for homogeneous content where boundaries don't matter (e.g., log data).

#### Recursive character splitting

The workhorse: split on a list of separators in priority order (`\n\n` → `\n` → `. ` → ` `), so chunk boundaries fall at paragraph or sentence breaks when possible. LangChain's `RecursiveCharacterTextSplitter` (chunk_size, chunk_overlap) is the most common production choice. It respects text structure better than fixed-size while staying cheap and deterministic.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=512,          # tokens (or characters depending on backend)
    chunk_overlap=64,        # ~10-20% overlap
    separators=["\n\n", "\n", ". ", " ", ""],
)
chunks = splitter.split_text(document_text)
```

#### Semantic chunking

Embedding-based: compute embeddings for candidate break points and split where the similarity between adjacent segments drops below a threshold — chunks become "topically coherent" units. Better coherence than recursive splitting, at the cost of an extra embedding pass over every document at ingest.

- Example: `semantic-text-splitter` (Rust/Python), LangChain's `SemanticChunker`, LlamaIndex's `SemanticSplitterNodeParser`.
- **Trade-off:** more compute at ingest; boundaries can be inconsistent between runs if you use a stochastic embedding provider; requires tuning the breakpoint threshold per corpus.
- **When it wins:** dense prose without explicit structure — research papers, legal prose, analyst notes.

#### Structure-aware chunking

Split by the document's own structure: headings, sections, subsections, tables. Markdown headers (`#`, `##`), HTML headings, LaTeX `\section{}` boundaries, and PDF outline entries are natural chunk boundaries. Structure-aware chunking preserves the document's logical units — the single most important chunking improvement for well-structured corpora (regulations, policies, manuals, docs sites).

- LangChain: `MarkdownHeaderTextSplitter`, `HTMLHeaderTextSplitter`, `RecursiveCharacterTextSplitter.from_language(...)`.
- LlamaIndex: `MarkdownNodeParser`, `HTMLNodeParser`, `JSONNodeParser`.
- **Keep the heading in the chunk:** prepend the section path (`Regulation X > Article 5 > 5.3`) to the chunk text or metadata so each chunk is self-describing.

#### Sentence-window chunking

Store small chunks (1–2 sentences) for retrieval precision, but expand each retrieved chunk with the surrounding sentences (a window of ±N sentences) before sending to the LLM. Improves precision at retrieval (small units match tightly) while giving the model enough context to answer. Implement with a sentence splitter plus a window expansion step, or LlamaIndex's `SentenceWindowNodeParser`.

#### Parent-document retrieval

The extreme version of the above: retrieve with **small child chunks** (e.g., 128–256 tokens) but return the **full parent document** (a section or the whole document) as context. LangChain's `ParentDocumentRetriever`, LlamaIndex's `HierarchicalNodeParser` + `AutoMergingRetriever`.

- **Why:** small chunks match the query precisely; the parent gives the model complete context — especially important for multi-hop reasoning across nearby paragraphs.
- **Cost:** much larger prompts; manage with token budgeting (§7.2). Great for documents where answers span paragraphs (regulatory clauses, policy questions).

#### Contextual chunking (Anthropic's contextual retrieval)

Before indexing, have an LLM **prepend a short contextual explanation to each chunk** — where this chunk sits in the document and what it's about — so the chunk is self-contained for retrieval and generation. Example transformation:

> *Original chunk:* "The bank must maintain a minimum capital ratio of 8%..."
> *Contextualized chunk:* "This is from Section 5 (Capital Requirements) of MAS Notice 637, issued June 2024, applying to banks licensed in Singapore. The bank must maintain a minimum capital ratio of 8%..."

Anthropic reported large retrieval gains (up to ~49% in their benchmark vs. naive chunking, with reranking adding more). **Cost:** one LLM call per chunk at ingest (batch it; use a cheap model; cache). **When it shines:** corpora with dense, context-dependent prose where chunks are ambiguous in isolation — regulations, contracts, technical docs. This is one of the highest-leverage chunking upgrades available; pair it with hybrid search + reranking for compounding gains.

#### Chunk size and overlap tuning

- **Typical range:** 256–1024 tokens. Small chunks (128–256) improve retrieval precision and work with parent-document retrieval; large chunks (512–1024) give the model more context per hit but dilute relevance and blow the token budget.
- **Overlap:** 10–20% of chunk size, so context that straddles a boundary is not lost. More overlap costs tokens and creates near-duplicate chunks; less risks losing boundary context.
- **Test per corpus:** chunk size interacts with document type, query granularity, and embedding model. Run a small grid (e.g., 256/512/1024 tokens × 10%/20% overlap) on the golden set and pick by recall@k + faithfulness — do not copy a number from a blog post.

#### Table/PDF-specific chunking

- **Keep tables whole:** a table (or a logical table block) should be one chunk; if a table is too large, chunk by row-groups with the header repeated. Structure-aware splitters that understand Markdown/HTML keep tables intact.
- **Two-column PDFs:** extract in reading order (left column then right) — naive extractors interleave columns. Layout-aware tools (pdfplumber with explicit column detection, marker-pdf) handle this.
- **Code and JSON:** split on function/class boundaries or top-level keys, not token counts.

**Chunking decision guide:**

| Corpus | Recommended starting point |
|---|---|
| Regulations / policies (structured) | Structure-aware by section + contextual chunking; parent-document for long clauses |
| Research papers / dense prose | Semantic or recursive; sentence-window for precision |
| FAQs / support articles | Sentence-window or small recursive chunks |
| Mixed intranet (HTML/Markdown) | Structure-aware (headers) + recursive fallback |
| Financial tables | Table-aware extraction → Markdown; keep tables whole |

### 5.3 Metadata and indexing

Metadata is what turns a vector search into a *filterable, citable, governable* search. Underinvesting in metadata is one of the most common production mistakes.

#### Rich metadata

Attach to every chunk at least: `source` (file path/URL), `doc_id`, `date` (publication/effective date), `author`, `doc_type` (regulation, policy, report, email), `section_path` (e.g., "MAS Notice 637 > Part II > 5.2"), `page_number`, and `title`. This enables:

- **Filtering** before vector search (§ below).
- **Citations** — the LLM can cite "MAS Notice 637, Part II §5.2, p. 12" and the UI can deep-link (§7.3).
- **Auditability** — trace any answer to its source documents (mandatory in banking, §12.4).

#### Hierarchical metadata

Model the corpus hierarchy: **document → section → chunk**. Store the doc-level attributes (owner, business unit, classification) on every chunk (denormalized) so filters and access control can be applied at chunk level without joins, and the section path for citations. If you use parent-document retrieval, keep the parent/child IDs in metadata.

#### Metadata filtering

Filter *before* vector search, not after: `WHERE doc_type = 'regulation' AND date >= '2023-01-01' AND business_unit = 'GMD'` reduces the search space, removes noise, and enforces policy (e.g., only current-year documents). Implement as:

- **Pre-filter:** vector stores that support metadata predicates natively (Qdrant filter, Pinecone metadata filter, pgvector WHERE, Elasticsearch post-filter/filter context) — the query vector is only compared against the filtered subset.
- **Self-query / semantic filters:** let the LLM translate the natural-language query into metadata filters (LangChain `SelfQueryRetriever`) — "the 2024 MAS notices" → `doc_type=notice AND year=2024`.

#### Hybrid index (dense + sparse/BM25 in the same index)

Plan for hybrid retrieval (§6.1) at indexing time: maintain **both** a dense vector index and a sparse (BM25) index over the same chunks. Some stores do this natively (Elasticsearch/OpenSearch with dense_vector + BM25, Weaviate hybrid, Qdrant sparse vectors, Milvus hybrid search, pgvector + tsvector); otherwise run two stores and fuse in the app layer. Keep chunk IDs consistent across both so fusion and reranking can reconcile results.

### 5.4 Embedding model selection

The embedding model determines what "similar" means. A generic model on a legal corpus is like a translator who only speaks everyday English. Selection is a real optimization lever — and one of the few changes that requires re-indexing the whole corpus, so choose deliberately.

#### Embedding model comparison

| Model | Typical dims | Notes |
|---|---|---|
| **OpenAI text-embedding-3-large** | 3072 (Matryoshka-reducible) | Strong general quality, multilingual; API cost per token |
| **OpenAI text-embedding-3-small** | 1536 | ~5x cheaper than large, near-large quality for many tasks |
| **Cohere embed-v3** | 1024 | Good multilingual; embed-english/embed-multilingual variants; retrieval-focused training |
| **BGE-M3** (BAAI) | 1024 | Dense + sparse + multi-vector from one model; strong multilingual; open weights |
| **E5 / multilingual-e5-large** | 1024 | Strong open-source generalist; prefix "query:" / "passage:" convention |
| **Nomic Embed v1.5** | 768 (Matryoshka) | Open weights (Apache-2.0), good quality/dimension ratio, long context (8192) |
| **jina-embeddings-v3** | 1024 | Task-type tokens (retrieval.query vs retrieval.passage); multilingual; long context |
| **Voyage (voyage-3 / voyage-3-large)** | 1024/3072 | API; strong retrieval benchmarks; domain variants (voyage-law, voyage-finance) |
| **GTE (Alibaba)** | 768/1024 | Strong Chinese + multilingual; open weights |

#### Model size vs. quality

Small open models (100–300 MB, e.g., BGE-small, MiniLM, GTE-small) are fast and cheap to self-host but measurably weaker on domain jargon and nuanced similarity. Large models (1–2 GB, e.g., bge-large, E5-large, or API models) cost more in latency and hosting but usually pay for themselves in retrieval quality. **Rule of thumb:** start with a mid-size multilingual model; if golden-set recall@k is clearly below target, upgrade and re-index — measure the delta before committing to the cost of re-indexing.

#### Multilingual support

If the corpus or users span languages (Singapore banking: English + Chinese regulatory text), use a multilingual embedding model (BGE-M3, multilingual-e5, Cohere embed-v3 multilingual, jina-v3, voyage-multilingual). Monolingual English models collapse non-English chunks to near-random vectors. Store `language` in metadata (§5.1) to diagnose cross-language retrieval failures.

#### Dimension reduction (Matryoshka embeddings)

Matryoshka models (OpenAI text-embedding-3, Nomic v1.5, GTE-Qwen2, BGE-M3) are trained so that *prefixes* of the vector remain meaningful — you can truncate 3072 → 1024 (or 768, 512) with graceful quality loss. Benefits: 3x less storage, faster search, cheaper queries; cost: some accuracy. **Do not truncate non-Matryoshka models** — that is just throwing away dimensions. Trade-off: a 1024-dim Matryoshka embedding from a 3072 model often beats a native 1024 model; test on your golden set.

#### Domain-specific fine-tuning

Fine-tune the embedding model on your domain (e.g., contrastive training on (query, relevant chunk) pairs mined from your logs) — expensive (GPU hours, labeled or mined data, eval rig) but potentially the **biggest quality gain available** for a specialized corpus, often beating model upgrades. Considerations:

- Mine training pairs from click-through/feedback logs or generate synthetic pairs with an LLM.
- Use a solid base (bge, E5, GTE); fine-tune with sentence-transformers `MultipleNegativesRankingLoss` / `MatryoshkaLoss`.
- Guard against overfitting: hold out a chunk of the golden dataset; re-run standard benchmarks to catch catastrophic forgetting.
- **Cost/benefit:** only pursue after cheaper levers (hybrid, reranking, chunking) are exhausted — typically the last resort for retrieval quality, not the first.

#### Embedding cache

Cache query embeddings by query text (hash → embedding). Repeated/near-repeated queries (common in internal search and support) skip the embedding call: ~10–50 ms saved per hit plus provider cost. Cache document embeddings implicitly by building the index once at ingest (§8.4 precomputation).

#### Query vs. document embeddings (asymmetric vs. symmetric)

- **Symmetric:** query and document are the same kind of text (FAQ retrieval, paraphrase matching). One embedding model for both.
- **Asymmetric:** short query vs. long document (most RAG). Models trained with a query/passage prefix convention (E5 "query:" / "passage:", BGE `encode_queries`/`encode_corpus`, jina task-type tokens) embed both sides correctly — **always use the right prefix/instruction for each side**; mixing them up silently destroys retrieval quality.
- Some models (BGE-M3) support both; for others, using a symmetric model asymmetrically is a common silent failure — verify with a recall@k sanity check.

---

## 6. Retrieval Optimization

Retrieval is where most RAG quality is won or lost. The order of operations matters: **fix the index (§5), then make retrieval hybrid, then transform queries, then rerank, then tune parameters.**

### 6.1 Hybrid retrieval — dense + sparse

**This is the single biggest retrieval improvement for most corpora.** Pure dense retrieval (vector similarity) is excellent at semantic matching and terrible at exact terms; pure sparse (BM25) is the opposite. Real queries need both.

- **Dense catches semantic:** synonyms, paraphrases, conceptual matches — "liquidity coverage" finds "LCR" docs.
- **BM25 catches exact terms:** identifiers, codes, error messages, names — "MAS Notice 637", "BCBS 239", "IBAN SG73...", product SKUs. Regulatory text, product codes, and error-message lookups *depend* on exact-term matching that dense embeddings routinely miss (and which a query rewrite can accidentally destroy).
- **Hybrid = both, merged.** Retrieve the top-k from each and fuse the ranked lists.

#### Reciprocal Rank Fusion (RRF)

The standard merge: for each document, sum its reciprocal ranks across the result lists:

```
RRF(d) = Σ_lists 1 / (k + rank_l(d))      # k = 60 is the typical default
```

RRF needs only *rankings*, not scores, so it works even when dense and sparse scores are on incomparable scales. k=60 is the common starting point; lower k weights top ranks more heavily. Simple to implement:

```python
def rrf(*ranked_lists, k=60):
    scores = {}
    for lst in ranked_lists:
        for rank, doc_id in enumerate(lst, start=1):
            scores[doc_id] = scores.get(doc_id, 0.0) + 1.0 / (k + rank)
    return sorted(scores, key=scores.get, reverse=True)
```

#### Weighted hybrid

Alternatively combine normalized scores: `final = α · dense_score + (1 − α) · sparse_score`, α typically 0.5–0.8. Requires score normalization (min-max or z-score per retriever). More tunable than RRF but more fragile; RRF is the safer default. When a corpus is dominated by exact-code lookups, skew toward sparse (lower α); for semantic-heavy corpora, skew toward dense.

#### Implementation

- **One store:** Elasticsearch/OpenSearch (dense_vector + BM25 + RRF out of the box), Weaviate, Qdrant (sparse vectors), Milvus, or pgvector + tsvector with app-level RRF.
- **Two stores:** vector DB + separate BM25 index (e.g., rank-bm25, Elasticsearch) fused in the app — more moving parts but full control.
- **Framework support:** LangChain `EnsembleRetriever` (RRF), LlamaIndex `QueryFusionRetriever` / `HybridRetriever` (dense + BM25).

**Why this matters so much:** for many corpora, hybrid search alone recovers more retrieval failures than every other retrieval technique combined, because the most common failure mode (§3.2) is exact-term queries against a dense-only index.

### 6.2 Query transformation

Pre-retrieval transformations reshape the query before it hits the index. The full catalog with trade-offs lives in [Query Rewriting in RAG](query_rewriting_rag_guide.md) and [RAG vs HyDE](rag_vs_hyde_guide.md) — here is the decision summary:

| Technique | What it does | Best for | Cost/risk |
|---|---|---|---|
| **Query rewriting** | LLM paraphrases/expands the query to match document vocabulary | Conversational queries, typos, vague phrasing | +1 LLM call; can drift off intent if the rewrite prompt is weak |
| **HyDE** | Generate a hypothetical answer document, embed *that* to retrieve | Questions where query words don't overlap document words (concepts) | +1 LLM call + extra embedding; can hurt on exact/factual queries — see [RAG vs HyDE](rag_vs_hyde_guide.md) |
| **Multi-query** | Generate N query variants, retrieve for each, merge results | Queries with several plausible phrasings or intents | N× retrieval cost; dedup needed |
| **Step-back prompting** | Abstract the question first ("what general principle applies?"), retrieve on the abstraction, then answer concretely | Broad conceptual questions, reasoning chains | Can over-abstract factual queries |
| **Query decomposition** | Split a complex query into sub-questions, retrieve per sub-question, combine evidence | Multi-hop questions spanning documents | Multiplies retrieval calls; needs answer synthesis |

**Operational rules:** route transformations by query type (simple lookups skip them; complex/ambiguous queries get them); cache the transformed query alongside the original; log both so you can diagnose when a rewrite degrades retrieval.

### 6.3 Reranking

Retrieval top-k is **recall-oriented**; generation needs **precision**. The standard pattern: retrieve 20–100 candidates cheaply, then rerank to the top 5–10 for the prompt. Reranking is the biggest precision lever after hybrid search.

#### Why rerank

- Initial retrieval (vector or hybrid) optimizes recall: it returns *likely* relevant chunks, with order that is good but not great.
- A reranker scores each candidate *against the query* with a much stronger model, fixing ordering errors and cutting noise (§3.3).
- Net effect: the context contains fewer irrelevant chunks, so the LLM answers more accurately and citations are more reliable.

#### Reranker types

| Type | Examples | Quality | Speed | Notes |
|---|---|---|---|---|
| **Cross-encoder** | Cohere Rerank (API), bge-reranker-v2-m3, ms-marco-MiniLM (e.g., cross-encoder/ms-marco-MiniLM-L-6-v2), Jina Reranker | Best | Slowest (full attention over query+doc pairs) | The quality standard; ~100–500 ms for 50 docs on GPU |
| **Late-interaction** | ColBERT / RAGatouille | Near-cross-encoder | Fast at scale (precomputed token embeddings) | Token-level matching; strong precision/recall trade-off |
| **Bi-encoder** | Embedding-model-based "rerank" (cosine on a fine-tuned model) | Weaker | Fastest | Often little better than the retriever itself; use only for high-throughput |

**Cross-encoders are the default choice** when quality matters (API: Cohere Rerank; open: bge-reranker-v2-m3, ms-marco-MiniLM family). ColBERT (via RAGatouille) is the strong self-hosted alternative for large candidate sets.

#### Reranker placement

**After fusion, before context assembly:** retrieve top-k (20–100) → fuse (hybrid) → rerank → take top 5–10 → build prompt. Do not rerank before fusion (you'd rerank two partial lists), and do not skip reranking because retrieval "looks fine" — rerankers routinely fix mid-list errors you can't see in a spot check.

#### Reranking cost and latency

Cross-encoder scoring of 50 documents is roughly 100–500 ms on GPU — significant but usually worth it. Control the cost:

- **Batch it:** score all candidates in one forward pass rather than per-document calls.
- **Right-size the candidate set:** 20–50 candidates is usually enough; 100+ multiplies rerank time for marginal recall gains.
- **FlashRank** (ONNX-optimized cross-encoders, e.g., FlashRank BGE/ms-marco variants) runs **30–100× faster on CPU** — often 5–20 ms for dozens of docs — making reranking viable without GPU. Start with FlashRank if you have CPU-only infrastructure; upgrade to full cross-encoders if quality demands.
- **Cache rerank results** for repeated queries (§8.4).

### 6.4 Retrieval parameter tuning

The cheap knobs — tune these before changing models:

- **top-k:** 5–10 for generation (context budget and noise), 20–100 when a reranker follows (recall-oriented candidate set). Tune per query type; support-asistants often want smaller k, research assistants larger.
- **Similarity threshold (cutoff):** drop candidates below a minimum score to keep noise out (§3.3). Calibrate per embedding model — cosine thresholds are model-specific (e.g., 0.3–0.5 for many OpenAI/bge models, but verify). Too aggressive a cutoff causes recall failures (§3.4) — validate against golden-set queries that legitimately have low similarity.
- **MMR (Maximum Marginal Relevance):** trades relevance against diversity: `score = λ · sim(query, doc) − (1 − λ) · max(sim(doc, already_selected))`. λ 0.5–0.8 typical; lower λ = more diversity. Use when a query has several distinct answer aspects (multi-topic questions) and the top-k are redundant copies of one aspect.
- **Context window budget:** decide how many tokens of context fit the prompt (2k–8k is typical for most production setups), then derive top-k and chunk size from it: `top-k ≈ budget / avg_chunk_tokens`. This links retrieval tuning to generation constraints (§7.2).
- **Hybrid weights:** α (dense weight) and RRF k — sweep small ranges on the golden set.
- **BM25 internals:** k1 (term-frequency saturation, default 1.2–2.0) and b (length normalization, default 0.75). For corpora with very uneven document lengths (long regulatory texts vs. short memos), lower b reduces the length penalty.
- **HNSW internals** (most vector stores): `ef_search` (higher = more accurate recall, slower; e.g., 100–400 for production) and `M` (graph connectivity, set at index build; 16–64). Raise ef_search first if recall looks low; M trades memory/ingest time for recall.

**Parameter tuning discipline:** one knob per run (§2.2), judged on the golden set, recorded in the experiment log.

---

## 7. Generation Optimization

Once retrieval is solid, generation quality is decided by three things: what goes into the prompt (context management), how the model is instructed (prompt engineering), and what the output must look like (output constraints).

### 7.1 Prompt engineering

#### Context formatting

How the context is presented materially affects grounding and citation quality. Best practices:

- **Clear delimiters:** wrap each chunk in distinct markers so the model can tell context apart from instructions — `<context>`, `[Document 1] ... [/Document 1]`, XML tags, or a numbered list.
- **Numbered sources:** label each chunk `[1]`, `[2]`, … and instruct the model to cite by number. Numbering *inside the prompt* (not the chunk ID) keeps citations stable and lets you map back to chunk IDs in post-processing.
- **Consistent structure:** same delimiter scheme across all prompts; a parser-friendly format (e.g., `---\n[1] <source metadata>\n<text>\n---`).

Example skeleton:

```
You are a compliance assistant for a bank. Answer the question using ONLY the
provided documents. Cite your sources as [1], [2], etc. If the documents do not
contain the answer, say "The provided documents do not contain this information."
Do not use outside knowledge.

Documents:
[1] Source: MAS Notice 637, Part II §5.2, p.12
<text of chunk 1>
[2] Source: MAS Notice 637, Part II §5.3, p.13
<text of chunk 2>

Question: <user query>
Answer:
```

#### Instruction clarity

- **"Answer from context only"** — explicit grounding instruction; without it, models lean on parametric memory.
- **"Say 'not in context'"** — give the model an escape hatch for recall failures (§3.4) instead of letting it guess; this converts hallucinations into honest "not found" answers (which you then fix at retrieval, §11).
- **"Do not use outside knowledge"** — for compliance and factual use cases; relax only when the task is creative or conversational.
- **Role/system prompt** — set the persona and constraints once ("You are a compliance analyst..."), keep it stable, version it (§12.4).
- **Few-shot examples** — 1–3 in-domain examples of (question, context, ideal answer) dramatically improve formatting compliance and tone, especially for citation format and "not in context" handling. Keep them short; long few-shots eat the token budget.

#### Anti-hallucination instructions

- Require every factual claim to be traceable to a cited chunk.
- Ask for **uncertainty acknowledgment** ("state confidence / flag missing information") rather than confident guessing.
- For numeric or regulatory content, instruct the model to quote rather than paraphrase.
- Remember: instructions reduce but do not eliminate hallucination; if the context lacks the answer, no prompt fixes it (§3.5). The prompt is the *last* line of defense, not the first.

#### Citation format requirements

State the format explicitly: `[n]` markers inline, a Sources section listing `[n] = source metadata`, and the rule that every cited number must correspond to a retrieved chunk. Post-process to validate: extract citation numbers, map to chunk IDs, drop or flag unsupported citations (§7.3, §11 citations-broken row).

### 7.2 Context management

#### Context selection

**Don't stuff everything.** Only chunks that survive filtering (similarity threshold, reranking, metadata filters) belong in the prompt. Each extra chunk costs tokens, dilutes attention, and invites the model to use irrelevant material (§3.3). Prefer 3–8 high-quality chunks over 15 mediocre ones.

#### Context ordering

Order matters more than most teams realize. **Lost-in-the-middle:** LLMs attend best to the beginning and end of long contexts, and worst to the middle. So:

- Put the **most relevant chunk first** (it is read most carefully and sets the frame).
- Put the second-most important **last**.
- Bury the rest in the middle — or better, keep the context short enough that ordering matters less (a 2–4k-token context rarely exhibits lost-in-the-middle; a 30k-token one does).
- Restructure alternatives: two-part prompts ("Summary of key evidence first, full details after"), or retrieval that feeds only the top-k most relevant with the rest compressed (§ below).

#### Context deduplication

Near-duplicate chunks (from document dedup misses, overlap, or multi-query retrieval) waste tokens and can confuse the model with slightly conflicting wording. Dedup by chunk ID after fusion, and by embedding similarity (cosine > ~0.95) for near-copies, keeping the highest-ranked representative.

#### Context compression

When candidates are only partially relevant, **compress instead of dropping**: keep the relevant sentences, summarize the rest. LangChain's `ContextualCompressionRetriever` (with an LLMChainExtractor or LLMChainFilter) does exactly this — useful when chunks are large and heterogeneous. Cost: one LLM call per query (or a cheaper extractor model); latency added before generation. Use when token budget is tight or when large parent documents are retrieved (§5.2).

#### Token budgeting

- Count tokens per chunk (the tokenizer for your generation model, not the embedding model).
- Set a hard budget for context (e.g., 4k tokens) derived from the model's window minus system prompt, few-shots, and answer headroom.
- Fill greedily in relevance order; **trim the tail** when over budget (the tail is the least-attended region anyway).
- Log context token counts per query (§9.1) to catch budget drift.

#### Adaptive context

Vary top-k by query complexity: simple lookups get 3 chunks; multi-hop or broad questions get 8 + compression. Route by heuristics (query length, entity count) or by a cheap classifier. Improves both quality (right-sized context) and cost (fewer tokens for easy queries).

### 7.3 Output constraints

#### Structured output

When answers feed downstream systems (workflow engines, forms, analytics), enforce structure:

- **JSON schema / function calling:** declare the schema; use the provider's structured-output mode or function-calling API so the model emits valid JSON.
- **Constrained decoding:** grammar/schema-constrained decoding (Outlines, Guidance, llama.cpp grammars, vLLM guided decoding) guarantees syntactic validity at generation time — see the constrained-decoding guide in this series.
- **Retry on validation:** schema-validate; on failure, retry with the validation error appended (1 retry usually suffices).

#### Citation generation

- Enforce `[n]` markers tied to prompt numbering (§7.1).
- **Grounding requirement:** every claim cites a chunk; answers with zero citations are flagged.
- Post-process: extract citation numbers, verify each maps to a retrieved chunk, and drop or annotate unsupported ones. This catches the "citations broken" failure (§3.7) automatically.

#### Answer format enforcement

Length, tone, and structure constraints belong in the prompt *and* should be validated: max word count, bullet vs. paragraph, "reply in plain language for a non-expert", "no legal advice disclaimer required". For tone consistency (customer-facing bots), combine the system prompt with 2–3 few-shot exemplars of the desired voice.

### 7.4 RAG-specific generation patterns

Beyond the single-pass retrieve-then-read pipeline, several patterns move quality up at the cost of extra LLM calls. Adopt deliberately, not by default.

| Pattern | Mechanism | When it pays | Cost |
|---|---|---|---|
| **LLM re-ranking in generation** | The LLM re-sorts or filters the retrieved chunks before answering (e.g., "rank these excerpts by usefulness for the question, use the top 3") | When reranker quality is the bottleneck and candidates are few | 1 extra LLM call (small output) |
| **Self-reflection / self-RAG** | Generate, then have the model critique its own answer for grounding and gaps; regenerate or patch | High-stakes factual answers; hallucination-prone domains | 2–3× generation calls |
| **Retrieve-then-read vs. generate-then-retrieve** | Classic RAG retrieves first; generate-then-retrieve generates hypotheses/claims first, then retrieves evidence to verify or fill gaps (e.g., Self-RAG-style claim verification, or RAG-checking) | Fact verification, hallucination auditing | Extra calls; requires a verification step design |
| **Corrective RAG (CRAG)** | A lightweight evaluator scores retrieval quality; if weak, correct via web search or knowledge refinement before generating | Noisy corpora; when retrieval confidence varies wildly | Evaluator call + possible search |
| **Agentic RAG** | An agent decides *whether* to retrieve, *what* to query, iterates (multi-hop), and can use tools | Multi-hop questions, heterogeneous sources, evolving information needs | Unbounded calls; needs guardrails, budgets, and tracing (§9) |

**Selection rule:** single-pass RAG first; add self-reflection or CRAG when faithfulness targets are unmet and you can afford the latency; go agentic only when queries are genuinely multi-step (agent loops are expensive to run *and* to operate — see [Hybrid & Multi-Agent Systems](../hybrid_multi_agent_systems_guide.md) for architecture guidance).

---

## 8. Latency and Cost Optimization

A RAG system that is correct but slow and expensive will not survive production. This section is about the pipeline-level levers; model- and serving-level techniques (speculative decoding, quantization, KV-cache tricks) are covered in [LLM Latency Optimization](../llm_latency_optimization_guide.md).

### 8.1 Where the time goes — the latency budget

Typical per-stage budgets for a standard RAG request:

| Stage | Typical latency | Notes |
|---|---|---|
| Query embedding | 10–50 ms | One API call or local model forward pass |
| Retrieval (vector search) | 20–200 ms | Scales with index size, ef_search, filters |
| Reranking (cross-encoder) | 100–500 ms | ~50 candidates; FlashRank: 5–20 ms on CPU |
| **Generation** | **1–10 s (dominant)** | First-token latency + generation time; grows with output length and context size |

**Generation dominates.** Optimizing 50 ms of retrieval is pointless if generation takes 5 s. Attack in this order: generation → rerank (if slow) → retrieval → embedding.

### 8.2 Optimize generation

- **Smaller/faster model for generation:** if quality holds, a smaller model (or a distilled variant) cuts both latency and cost — measure faithfulness before/after; many RAG tasks are context-bound, so a small model with good context often matches a large one.
- **Speculative decoding** (draft model + target verification) — see the latency guide; can yield 2–3× generation speedup with the same model quality.
- **Shorter outputs:** instruct length limits ("answer in ≤ 3 sentences") — output tokens are the most expensive latency and cost driver per query.
- **Streaming:** first-token-latency hides total latency from users (token-by-token output) — near-mandatory for chat UIs; pair with "typing" affordances.
- **Reduce context size:** fewer tokens in = faster prefill; context compression (§7.2) helps here too.

### 8.3 Parallelize retrieval

The retrieval path is embarrassingly parallel: run **dense search, BM25 search, and reranking as concurrent calls** where dependencies allow (rerank waits for both searches, but the two searches run in parallel). In Python, run them in threads/asyncio (I/O-bound API calls) or processes (CPU-bound local rerankers). Typical win: 30–50% off the retrieval-stage wall-clock. Beware of slowest-call variance — set timeouts and fallbacks.

### 8.4 Caching

Caching is the highest-leverage cost/latency lever for workloads with repeated or similar queries (support bots, internal search, dashboards).

- **Query embedding cache:** hash normalized query text → embedding. Saves 10–50 ms and API cost on repeats.
- **Retrieval result cache:** normalized query → ranked chunk IDs. Saves the whole search stage; invalidate on index updates (version the index, key the cache by index version).
- **Semantic cache (answer cache):** embed the query, find a *similar* previously-answered query (cosine above threshold, e.g., 0.9+), and return the cached answer. Tools: GPTCache, Redis-based custom, LangChain `SemanticCache`. This is the biggest cost saver of all — it eliminates the LLM call entirely for near-duplicate questions. Tune the threshold carefully: too low returns wrong answers; too high misses hits. Log cache hits/misses and monitor answer validity over time (cached answers go stale).
- **Rerank cache:** query → reranked list, same invalidation rules as retrieval cache.
- **Precomputation:** document embeddings are computed once at ingest, not per query — always; also precompute BM25 stats at ingest (that's what BM25 indexing is).

### 8.5 Batching

- **Batch embedding at ingest:** embed documents in batches (100s–1000s per call for APIs) — cheaper and faster than one-by-one.
- **Batch reranking:** one cross-encoder forward pass for all candidates (§6.3).
- **Server-side request batching:** if the workload is offline/batch (report generation, nightly summarization), batch queries through vLLM/other servers for throughput; for interactive traffic, batching trades latency for throughput — measure first.

### 8.6 Hardware

- **Reranking:** cross-encoders on GPU (100–500 ms for 50 docs) vs. CPU (seconds) vs. FlashRank ONNX on CPU (5–20 ms). For CPU-only infra, FlashRank is the default.
- **Embedding:** local small models run fine on CPU for moderate QPS; large models and high QPS want GPU or a managed API.
- **Vector search:** HNSW on RAM; consider quantization (product quantization, scalar) to shrink memory 4–32× at small recall cost for very large corpora.

### 8.7 Cost optimization

Cost per query ≈ embedding + retrieval + rerank + LLM in/out tokens. Attack each:

- **Model routing:** cheap/fast model for easy queries, expensive model for complex ones (classify by query difficulty — length, entities, past success rate). Typical savings: 30–70% of LLM cost. Route conservatively; misrouting a hard question to a weak model degrades quality.
- **Token optimization:** fewer chunks (context selection §7.2), compressed context, shorter outputs (§8.2), dedup. Input tokens dominate for long-context RAG — cutting context from 8k to 3k tokens cuts ~60%+ of input cost.
- **Embedding cost:** batch at ingest (§8.5), cache queries (§8.4), consider Matryoshka truncation (§5.4) — fewer dimensions = cheaper storage and cheaper query embedding for some providers.
- **Evaluation cost:** use cheaper judge models for RAGAS metrics (a small judge correlates reasonably with a large one for ranking — validate on a sample first); batch evaluations; reuse cached evals by config hash.
- **Semantic caching** (§8.4) eliminates whole LLM calls for repeated queries — usually the single biggest cost lever after routing.

**Latency/cost tuning checklist:** measure stage latencies first (observability §9) → optimize generation → parallelize retrieval → cache aggressively → route models → then revisit.

---

## 9. Observability and Monitoring

You cannot optimize what you cannot see. Observability in RAG means capturing what happened on every query, computing quality metrics continuously, and alerting when the system degrades.

### 9.1 What to log (the RAG trace)

Log one structured record per query, with:

- **Input:** raw query, rewritten/transformed query, query type classification, user/tenant ID, timestamp.
- **Retrieval:** retrieved chunk IDs, dense scores, BM25 scores, fused/rerank scores, filters applied, top-k used, which retriever contributed.
- **Context:** number of chunks in the final prompt, context token count, chunk order, compression applied.
- **Generation:** model name, temperature, prompt template version, full answer, citations emitted.
- **Performance:** latency per stage (embed, search, rerank, generate, total), first-token latency, token counts (input/output), cost estimate.
- **Outcome:** user feedback (thumbs up/down), downstream actions.

This is the raw material for every diagnosis in §11. Without chunk IDs in logs, you cannot tell whether a bad answer was a retrieval failure or a generation failure — the single most common observability gap.

### 9.2 Retrieval quality metrics

Computed offline on the golden set (§10) and online on sampled traffic:

- **Recall@k** — fraction of relevant documents found in the top-k. The primary recall metric: how often the answer-bearing chunk made it into the context.
- **Precision@k** — fraction of top-k that are relevant. Measures noise (§3.3).
- **nDCG@k** — graded relevance, position-weighted. Best single ranking metric when relevance is graded.
- **MRR (Mean Reciprocal Rank)** — how high the *first* relevant hit ranks. 1.0 = always first; most RAG systems live in the 0.6–0.95 range on good corpora.
- **Hit rate** — fraction of queries where at least one relevant chunk is retrieved. The simplest "did we find it at all" metric; pairs with recall for diagnosis.

### 9.3 Generation quality metrics

- **Faithfulness / groundedness** — every claim in the answer is supported by the context (RAGAS `faithfulness`). The #1 metric for banking RAG.
- **Answer relevance** — the answer actually addresses the question (RAGAS `answer_relevance`).
- **Context precision / context recall** — whether the retrieved context was relevant and complete (RAGAS, §10.2).
- **Hallucination rate proxy** — share of answers where a claim fails a grounding check (LLM-as-judge or entailment model on sampled traffic; also detected via citation validation §7.3).
- **User feedback** — thumbs up/down, helpfulness ratings, "wrong answer" reports. Sparse but the truest signal; correlate it with logged features (which chunks, which model) to find systemic issues.

### 9.4 Drift detection

Quality degrades over time even with no code changes:

- **Query distribution drift:** new topics appear (new product launch, new regulation), old ones fade. Track query clusters / embedding centroid shifts over time; refresh the golden set (§10.1) to match.
- **Retrieval performance drift:** recall@k on sampled traffic declining → corpus or query drift, or a silently broken index (failed incremental update, embedding model swapped upstream).
- **Index staleness:** documents updated in the source system but not re-indexed → answers go stale (§11 stale row). Monitor index age per document and re-ingest lag.
- **Model drift:** the upstream LLM or embedding provider changes behavior (version bumps, provider-side updates) — re-run the golden set after any model change, even "minor" ones.

### 9.5 Alerting

Alert on leading indicators, not just user complaints:

- **Retrieval failure rate** — share of queries where no chunk passes the similarity threshold (or hit rate on sampled traffic drops). Sudden rise = index/embedding problem.
- **Hallucination rate proxy** — grounding-check failures on sampled traffic above threshold.
- **Latency p95 breach** — end-to-end p95 above budget; also alert per-stage (generation p95 spiking → model/serving issue; retrieval p95 spiking → index/HNSW issue).
- **Cost per query** — moving average crossing budget.
- **Cache hit rate drop** — semantic cache invalidation storms or query-drift.
- **Empty/error responses** — the canary: 5xx from the LLM provider, vector store timeouts.

### 9.6 Tracing

Trace the full query → answer path in a tracing platform so any individual failure can be replayed and inspected:

- **LangSmith** — LangChain-native tracing with run trees, eval hooks, and dataset integration.
- **Langfuse** — open-source; traces, scores, and prompt versioning; strong for regulated environments (self-hostable).
- **Arize Phoenix** — open-source; embeddings + LLM tracing, drift and retrieval-quality views.
- **OpenLLMetry** (Traceloop) — OpenTelemetry-based instrumentation that works with many frameworks, feeding existing observability stacks (Grafana, Datadog, etc.).

**For banking:** prefer self-hostable tracing (Langfuse, Phoenix, OpenTelemetry-based) so query/answer data stays inside the perimeter; ensure traces redact PII (§12.2) and are retained per records-management policy.

### 9.7 Dashboards

Slice metrics by dimensions that expose problems:

- **Per corpus** — which document collection underperforms (retrieval quality varies wildly by source quality).
- **Per query type** — simple lookups vs. multi-hop vs. numeric; failures cluster in one type → targeted fix.
- **Per embedding model / per LLM** — A/B comparisons in production; catch regressions from provider model updates.
- **Per tenant/business unit** — in banking, per-department filtering and access rules can change retrieval behavior.
- **Per stage** — latency and error breakdown across embed → search → rerank → generate.

---

## 10. Evaluation

Evaluation is the engine of the whole optimization loop: it tells you whether a change helped, and it catches regressions before users do.

### 10.1 The golden dataset

The foundation of everything. Curate 50–200 question–answer pairs with, for each: the query, the reference answer, and the **document/chunk IDs that contain the ground truth** (needed for retrieval metrics).

- **Multi-format:** include simple lookups, multi-hop questions, "abstract" questions (answer implied across several chunks), and edge cases (negations, acronyms, numeric ranges, "which is NOT", cross-lingual queries).
- **Sourced from reality:** mine real user queries (redacted) from logs; don't invent idealized phrasing — real queries are messier and expose real failures.
- **Version it:** store the dataset in the repo (or a data store) with a version; every evaluation run records the dataset version. Changes to the dataset are code-reviewable changes.
- **Update quarterly** (or on corpus/query-mix shifts): retire stale pairs, add new ones. Versioning + quarterly refresh keeps the evaluation honest as the world moves.

### 10.2 Offline evaluation metrics

#### RAGAS metrics (the standard set)

| Metric | What it measures | Failure it catches |
|---|---|---|
| **Faithfulness** | Answer claims are supported by the retrieved context | Hallucination (§3.5) |
| **Answer relevance** | Answer addresses the question | Off-topic answers |
| **Context precision** | Retrieved context is relevant (noise-free) | Noise (§3.3) |
| **Context recall** | Retrieved context contains what's needed | Recall failure (§3.4) |
| **Context entity recall** | Key entities in the reference answer appear in the context | Entity-level recall failures |

RAGAS is LLM-judge based — you can run it on any pipeline regardless of framework (it's metric code, not a framework). See [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) for how it compares with TruLens and DeepEval, and [LLM Evaluation Frameworks](../llm_evaluation_frameworks_guide.md) for judge-model selection and calibration.

#### Retrieval-only metrics

recall@k, precision@k, nDCG@k, MRR, hit rate (§9.2) — computed from the golden set's ground-truth document IDs, no LLM judge needed. Fast and cheap; run these on every experiment.

#### Regression testing

**Run the golden set on every pipeline change** — a config change, a prompt edit, an embedding upgrade, a chunking tweak — automatically (CI job or scheduled). Compare against the stored baseline and flag regressions. This is what makes "change one variable at a time" (§2.2) actually safe: regressions are caught in minutes, not by users.

### 10.3 A/B testing (online)

Offline metrics don't capture everything (real query mix, user interaction, latency under load). For material changes — new embedding model, new reranker, new prompt — run an online A/B test:

- Split traffic (holdout users or query hash bucketing) between the current pipeline and the candidate.
- Compare: user feedback rates, answer quality on sampled traffic (judges), latency p95, cost per query, error rates.
- **Holdout users** (e.g., 5–10%) let you compare on *identical* query distributions and roll back instantly; hash-bucketing is simpler but users may see both variants.
- Run long enough to cover the query-mix cycle (at least 1–2 weeks for support/assistant workloads), and pre-register the decision metric to avoid cherry-picking.

### 10.4 Online monitoring (continuous)

The online half of evaluation: feedback loops and alerting (§9.3–9.5) running permanently — user feedback collection, sampled-judge evaluation of production answers, drift monitors, latency/cost alerting. Offline eval says "the change is good in the lab"; online monitoring says "the system stays good in production."

### 10.5 Evaluation frameworks

- **RAGAS** — the de-facto RAG metric library (faithfulness, answer/context metrics); framework-agnostic.
- **TruLens** — feedback functions + dashboards; RAG triad (context relevance, groundedness, answer relevance); good for structured feedback-function authoring.
- **DeepEval** — pytest-style test-driven LLM evaluation; metric classes with thresholds; strong for CI integration.
- **LangSmith** — datasets + evaluators + tracing in one place; the natural choice if you're on LangChain.
- See [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) for the full comparison and selection guidance.

**Evaluation stack recommendation:** golden dataset (versioned) → RAGAS for offline metrics → CI regression gate → LangSmith/Langfuse for tracing + sampled online eval → dashboards with drift/alerting (§9.7). Run retrieval metrics on every change (cheap), full RAGAS on release candidates, and sampled judge evaluation continuously.

---

## 11. The Optimization Playbook by Symptom

The fast path to a fix: match the symptom to the diagnosis, then apply the fixes in the order listed (cheapest/highest-leverage first). Each row links back to the relevant section.

### 11.1 Answers wrong or not grounded (hallucination-like)

- **Diagnosis:** retrieval failure (answer not in context) or generation failure (answer in context but model ignored/misused it). **First check which** by inspecting the actual prompt (§3.8).
- **Fixes:**
  1. Check context relevance for failing queries — is the answer-bearing chunk present? If not, treat as retrieval failure (§11.2).
  2. Raise top-k / improve recall (hybrid search, better chunking, multi-query) if context lacks the answer.
  3. Improve chunking so the answer arrives complete (§5.2).
  4. Add reranking to put the right chunk first (§6.3).
  5. Strengthen grounding instructions: "answer from context only", "say not in context", cite every claim (§7.1).
  6. Lower temperature; validate citations post-hoc (§7.3).

### 11.2 Answers missing key information (recall failure)

- **Diagnosis:** the info exists in the corpus but never reached the model (§3.4).
- **Fixes:**
  1. Verify the chunk is in the index (parse/ingest failure — §5.1); check dedup didn't drop it.
  2. **Hybrid search** — exact terms and codes are the classic dense-only miss (§6.1).
  3. Better chunking — structure-aware/contextual so the answer-bearing unit survives (§5.2).
  4. Multi-query / query rewriting for phrasing mismatch (§6.2).
  5. Increase top-k (and rerank to keep precision) (§6.3–6.4).

### 11.3 Answers include irrelevant information (noise)

- **Diagnosis:** off-topic chunks in the prompt (§3.3).
- **Fixes:**
  1. Add a reranker and tighten the final top-k (§6.3).
  2. Enforce a similarity threshold cutoff (§6.4).
  3. Use MMR for diversity when a query spans multiple aspects (§6.4).
  4. Metadata filtering to exclude out-of-scope doc types/dates (§5.3).
  5. Context dedup and compression (§7.2).

### 11.4 Answers are stale

- **Diagnosis:** index refresh issue (§3.4/§9.4) — the corpus changed but the index didn't.
- **Fixes:**
  1. Re-ingest the affected documents; verify the ingestion pipeline runs (CDC or scheduled sync).
  2. Add a **CDC pipeline** (change data capture from source systems: documents, policies, regulation feeds) or scheduled re-index with staleness monitoring (§9.4).
  3. Version the index; on rebuild, invalidate caches (§8.4).
  4. Store effective dates in metadata and filter `date <= today` so superseded docs don't surface (§5.3).

### 11.5 Slow answers

- **Diagnosis:** latency breach — usually generation, sometimes rerank (§8.1).
- **Fixes:**
  1. Measure per-stage latency from logs (§9.1) — fix the dominant stage.
  2. Stream output; cap answer length (§8.2).
  3. Smaller/faster generation model if faithfulness holds (§8.2).
  4. Cache aggressively (semantic cache, retrieval cache) (§8.4).
  5. Parallelize dense + sparse + rerank (§8.3).
  6. FlashRank if reranking is the bottleneck (§6.3).

### 11.6 Too expensive

- **Diagnosis:** cost per query over budget.
- **Fixes:**
  1. Model routing — cheap model for easy queries (§8.7).
  2. Semantic caching for repeated/similar queries (§8.4).
  3. Token reduction — fewer chunks, compressed context, shorter outputs (§7.2, §8.7).
  4. Cheaper judge models for evaluation; batch evals (§8.7).
  5. Embedding cost: cache queries, batch ingest, Matryoshka truncation (§5.4, §8.5).

### 11.7 Citations broken

- **Diagnosis:** metadata/parsing issue — chunk→source mapping lost (§3.7).
- **Fixes:**
  1. Fix chunking metadata: preserve source, section path, page number through parse→chunk→index (§5.3).
  2. Number chunks in the prompt (`[1]…[n]`) and map back to chunk IDs in post-processing (§7.1, §7.3).
  3. Validate citations at generation: drop/flag markers that don't map to retrieved chunks (§7.3).
  4. Re-parse documents whose text lost structure (tables, columns) — the citation points at a chunk that doesn't contain the claim (§5.1).

### 11.8 Inconsistent answers

- **Diagnosis:** generation variance — same question, different answers across runs.
- **Fixes:**
  1. Lower temperature (0.0–0.3 for factual tasks) (§7.1).
  2. Deterministic decoding / fixed seed where supported (§7.1).
  3. Sharpen prompts — fewer ambiguous instructions, explicit answer format (§7.1).
  4. Stabilize the context — dedup, consistent ordering, fixed top-k (§7.2).
  5. Check the upstream model/provider for silent version drift (§9.4).

### 11.9 Quick-reference table

| Symptom | Primary diagnosis | First fixes (in order) |
|---|---|---|
| Wrong / ungrounded answers | Retrieval failure or weak grounding | Check context §3.8 → hybrid §6.1 → chunking §5.2 → rerank §6.3 → grounding prompt §7.1 |
| Missing info | Recall failure | Index check §5.1 → hybrid §6.1 → chunking §5.2 → multi-query §6.2 → raise k §6.4 |
| Irrelevant info | Noise | Rerank §6.3 → threshold §6.4 → MMR §6.4 → filters §5.3 → compression §7.2 |
| Stale answers | Index refresh | Re-ingest → CDC pipeline → index versioning §8.4 → date filters §5.3 |
| Slow answers | Generation dominant | Stage profiling §9.1 → streaming/length §8.2 → smaller model §8.2 → caching §8.4 → parallelize §8.3 |
| Expensive | Token/call cost | Routing §8.7 → semantic cache §8.4 → token reduction §7.2 → eval cost §8.7 |
| Broken citations | Metadata loss | Chunk metadata §5.3 → prompt numbering §7.1 → citation validation §7.3 → re-parse §5.1 |
| Inconsistent answers | Generation variance | Temperature ↓ §7.1 → deterministic decoding §7.1 → stable context §7.2 → drift check §9.4 |

---

## 12. Banking-Specific RAG Optimization

Banking RAG (and Singapore-regulated financial services specifically) adds constraints that change *which* optimizations matter and how they are governed. Jack Liu Shurui is a Solution Architect at Cymbal Bank; the patterns below reflect production realities in that world: regulatory corpora, strict latency budgets for customer-facing channels, PII, compliance, and model-risk governance.

### 12.1 Regulatory text (MAS, BCBS, HKMA, EU regulations)

- **Hybrid retrieval is essential.** Regulatory references are exact strings: "MAS Notice 637", "BCBS 239", "SFA Section 12", "MAS Notice 644". Dense embeddings routinely fail on these; **BM25 catches them exactly** (§6.1). For a regulatory corpus, dense-only is not an option — hybrid (with a sparse skew for identifier-heavy queries) is the baseline.
- **Chunk by regulatory structure** — regulation → part → section → subsection — not by token count. Sections are the legal unit; a clause split across chunks is a compliance incident waiting to happen (§5.2 structure-aware).
- **Preserve cross-references.** Regulations reference each other ("as defined in MAS Notice 645, paragraph 8"). Keep these references *in* the chunk text and record them in metadata (regulation name, section number) so retrieval and citation can follow the reference graph.
- **Semantic chunking on dense legal prose** — for regulations written as long unbroken paragraphs, semantic chunking (or contextual chunking) keeps clauses coherent where structure-aware splitting can't (§5.2).
- **Versioned/effective-dated content:** index by effective date; filter `date <= today`; keep superseded versions retrievable *on request* for audits but out of default results (§5.3, §11.4).
- **Grounding is non-negotiable:** faithfulness metrics are the release gate; "not in context" answers must outrank hallucinated ones (§7.1, §10.2).

### 12.2 Customer-facing assistants

- **Strict latency budget:** streaming, small fast model for the common path, semantic cache for the top-100 questions (which usually absorb most traffic) (§8.2–8.4).
- **Tone consistency:** fixed system prompt + 2–3 few-shot exemplars of the approved voice; A/B any tone change through the golden set first (§7.1).
- **PII handling:** redact PII **before indexing** (names, account numbers, NRIC) — the index must never contain it; filter in retrieval so PII-bearing chunks are excluded from prompts by default; redact in logs and traces (§9.6). PII queries (account lookups) should route to *transactional systems*, not RAG.
- **Compliance:** maintain an **audit trail of retrieval + generation** — for every answer: query, chunk IDs, scores, prompt version, model version, timestamp (this is §9.1 made mandatory). Define **human escalation paths**: if confidence is low, the answer says "not in context", or the query is high-stakes (product terms, eligibility), route to a human rather than answering.
- **Disclaimer hygiene:** regulated-advice boundaries ("not financial advice", "check current rates") must be in the system prompt and versioned with legal review.

### 12.3 Internal knowledge search (intranet, policies, procedures)

- **Metadata filtering is the backbone:** department, business unit, date, doc type, confidentiality classification — filter before search so a GMD user doesn't surface a Retail memo (§5.3).
- **Access control (per-role retrieval):** filter chunks by authorization at query time — the retriever must never return chunks the user cannot see. Implement via metadata filters populated from the identity provider, *not* by post-hoc filtering in the UI (which leaks data through scores and snippets). Test with cross-role probes.
- **Freshness:** prioritize recent docs — effective-date metadata + recency weighting in fusion; stale-index alerting (§9.4).

### 12.4 Evaluation and governance in banking

- **Golden dataset from real queries (redacted):** mine production queries, strip PII, label ground truth from the actual source documents (§10.1).
- **Compliance-aware metrics:** groundedness/faithfulness is *mandatory* — a release cannot ship below threshold. Add citation validity (every citation maps to a real chunk) as a first-class metric (§7.3, §10.2).
- **Human review for high-stakes:** sample high-stakes answers (regulatory interpretation, client-facing content) for manual review; feed findings back into the golden set.
- **Regulator-ready documentation (SR 11-7 / MAS model-risk mindset):** treat the RAG pipeline as a model under risk governance:
  - **Document optimization changes** — every change (chunking config, embedding model, reranker, prompt) recorded with rationale, before/after metrics, and sign-off.
  - **Versioned prompts and chunking config** — prompts, splitters, and embedding model versions stored in the repo with the code (§9.6).
  - **Reproducibility** — the index must be rebuildable from a pinned config, and evaluations reproducible from the versioned golden set.
  - **Audit trail** — retrieval + generation logs retained per records-management policy.
- This turns §2's optimization discipline into a *compliance requirement*: one-variable-at-a-time, measured, documented changes are exactly what model-risk reviewers expect to see.

---

## 13. The 80/20 Optimization List

If you do nothing else, do these ten — in this order. They deliver most of the achievable quality at a fraction of the effort of the long tail.

1. **Hybrid retrieval (BM25 + dense + RRF).** The biggest single win for most corpora — recovers the most common failure mode (exact-term queries against dense-only indexes) in an afternoon of work (§6.1).
2. **Better chunking** — structure-aware, semantic, or contextual (Anthropic-style). Chunk quality outranks retrieval tuning: no retriever can fix a destroyed unit of meaning (§5.2).
3. **Reranking** — cross-encoder (or FlashRank on CPU). Big precision gains; fixes ordering and cuts noise for ~100–500 ms (§6.3).
4. **Metadata filtering** — pre-filter by doc type, date, business unit. Removes noise, enforces policy, and in banking implements access control (§5.3).
5. **Prompt engineering** — grounding instructions, "say not in context", numbered citations, few-shot examples. Cheap, immediate, measurable (§7.1).
6. **Golden dataset + RAGAS evaluation.** Measure before you optimize; regression-test every change. This is what makes all the other items safe (§10).
7. **Query transformation** — rewriting, HyDE, multi-query for complex/ambiguous queries. Target the query types where retrieval fails, not all queries (§6.2).
8. **Context management** — compression, ordering, token budgeting, dedup. Improves faithfulness and cuts cost simultaneously (§7.2).
9. **Caching** — semantic cache for repeated queries; retrieval/embedding caches. The biggest cost/latency lever for real workloads (§8.4).
10. **Embedding model upgrade** — larger/better model, or fine-tuned on domain data. Highest-effort item; do it last, and only when cheaper levers are exhausted (§5.4).

**The trap to avoid:** skipping 1–6 to chase 10 (or worse, chasing exotic techniques like agentic RAG) before the fundamentals. In practice, items 1–6 fix 80%+ of real RAG failures, and items 7–10 cover the rest.

---

## 14. Conclusion

RAG optimization is a systematic process, not a bag of tricks. The playbook in one paragraph:

**Measure first** — golden dataset, baseline metrics, per-query traces (§2, §9, §10). **Fix the index** — parse cleanly, chunk with structure, enrich metadata, choose the embedding deliberately (§5). **Fix retrieval** — hybrid search, transform queries, rerank, tune k (§6). **Fix generation** — ground the prompt, manage the context, constrain the output (§7). **Watch latency and cost** — generation dominates, cache aggressively, route models (§8). **Monitor continuously** — drift, alerts, dashboards, quarterly golden-set refresh (§9). **And document it all** — in banking, the optimization log *is* the model-risk evidence (§12.4).

Work the pipeline in order, change one variable at a time, and let the golden dataset arbitrate. The system that follows this loop — measured, incremental, evidence-driven — is the system that keeps getting better while the corpus, the queries, and the models keep changing.
