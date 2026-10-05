# Information Retrieval System Architecture — An Index Is a Bet About the Queries You Will Be Asked

*A deep-research guide to information retrieval as a system: the inverted index and its postings, the analysis and query pipeline, the ranking stack, the serving and write paths, the evaluation loop, the entitlement and deletion problems, and the one structural bet every retrieval design makes before it sees a single real query. Written for architects and engineers who must own retrieval as a system rather than as a feature of one product.*

> **Author:** Jack Liu Shurui, Solution Architect
> **Repository:** github.com/jackliusr/research · **Category:** Technology Series · **Date:** October 2026

**Verification posture for this guide.** Every algorithm, paper attribution, metric definition and product fact below is labelled by provenance. The labels used are: **[IR-BOOK]** = stated in Manning, Raghavan and Schütze, *Introduction to Information Retrieval* (Cambridge University Press, 2008; free HTML edition by the authors at nlp.stanford.edu/IR-book); **[PAPER]** = stated in the named primary paper, with author, title, venue and year; **[LUCENE]** = stated in Apache Lucene's own documentation; **[ES]** = stated in Elasticsearch's own documentation; **[OS]** = stated in OpenSearch's own documentation; **[WEB]** = stated on a named web page. Figures or attributions that could not be pinned to a primary source this pass are marked ⚠ and listed in §15 and §16. **No recall, precision, latency, throughput or quality number appears in this guide unless it is quoted from a dated source; this guide prints no worked metric value as if it were a benchmark.** Where a number is used it is a *worked illustration*, and it is labelled as one in the same paragraph.

---

### Table of Contents

1. The Overview, the Thesis and the Decoder
2. The Stack Map
3. The Index
4. The Query Pipeline
5. The Ranking Stack
6. The Serving Tier
7. The Write Path
8. Evaluation
9. The Classical and the Modern
10. The Platform Mapping
11. The Failure Modes
12. Entitlement and Deletion
13. The Cymbal Bank Worked Example
14. The Anti-Patterns
15. The Claims Audit
16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

---

## 1. The Overview, the Thesis and the Decoder

**The thesis, in one line: an index is a bet about the queries you will be asked.**

Everything else in this guide is commentary on that sentence. An index is not a copy of the collection. It is a *reorganisation of the collection around a guess about how it will be interrogated* — which terms matter, whether order matters, whether the query is a handful of keywords or a paragraph, whether a user must be shown only what they are entitled to see. Every one of those guesses is expensive to change after the fact, and some of them cannot be changed at all without rebuilding the index. The rest of this guide is about making the guess consciously, and about knowing what you have given up.

### 1.1 The decoder — twelve terms, defined precisely

Read these once; the whole guide leans on them.

| Term | Definition as used in this guide | Provenance |
|---|---|---|
| **The collection** | The fixed set of documents a retrieval system is built to serve. "Documents" here are the unit of retrieval, which may be a file, a web page, an email, a message, or a passage — the choice of unit is a design decision, not a given. | **[IR-BOOK]** ch. 2 (choosing a document unit) |
| **The document** | One retrievable unit, identified by a document ID; the system indexes and returns documents, not "answers". | **[IR-BOOK]** ch. 1 |
| **The term** | A token that survives the analysis chain and is actually indexed — the vocabulary item, not the raw word. "Term" is what the index stores; "word" is what the user typed. | **[IR-BOOK]** ch. 2 |
| **The posting** | One entry in a postings list: minimally a document ID, optionally a term frequency, and optionally the positions at which the term occurs. The IR book defines a posting as a docID in a postings list. | **[IR-BOOK]** ch. 5 |
| **The inverted index** | The core structure: it maps each term to the list of documents (the postings list) that contain it. Built from a collection of documents by tokenising, preprocessing and writing out the terms each document contains. | **[IR-BOOK]** ch. 1, ch. 2 |
| **The forward index** | The mirror image: it maps each document to the terms/values it contains. Used to answer "what is in this document?", and — in Lucene-family stacks — realised as **doc values**, an on-disk columnar structure built at index time for sorting and aggregation. | **[IR-BOOK]** ch. 1 (contrast); **[ES]** doc_values |
| **The segment** | The immutable unit a shard is built from. A Lucene index is "a collection of segments plus a commit point"; a shard is a Lucene index broken into immutable segments, which are periodically merged into larger segments. | **[ES]** near-real-time; index-modules-merge |
| **The query** | The system's representation of an information need after analysis and rewriting — not the string the user typed. The user has an *information need*; the query is its formal proxy. | **[IR-BOOK]** ch. 8, ch. 11 |
| **The ranking** | The order in which matching documents are returned, produced by a scoring function that is only a *proxy* for perceived relevance. | **[IR-BOOK]** ch. 7 |
| **The relevance judgment** | A human decision that a document is (or is not, or how strongly it is) relevant to an information need. Retrieval is "a highly empirical discipline"; judgments are the ground truth that evaluation is built on, and they are expensive and imperfect. | **[IR-BOOK]** ch. 8 |
| **Recall and precision** | Precision = fraction of retrieved documents that are relevant = `tp/(tp+fp)`. Recall = fraction of relevant documents that are retrieved = `tp/(tp+fn)`. They trade off: retrieving all documents gives recall 1 at poor precision. | **[IR-BOOK]** ch. 8 |
| **The index refresh** | The act of making documents written since the last refresh visible to search — in Elasticsearch, "writing and opening a new segment", done periodically (default every second) so that a document is searchable "in near real-time — within 1 second". Refresh is **not** the same as a durable commit. | **[ES]** near-real-time; docs-refresh |

### 1.2 What this guide owns, and what it does not

This guide owns **information retrieval as a system**: the index and its data structures, the analysis and query pipeline, the execution model, the ranking stack, the serving and write paths, evaluation, entitlement and deletion, and the architecture that connects them.

It deliberately does **not** re-derive, and cites instead:

- **Search-platform operations and topology** — owned by [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md). That guide owns fork history, the sizing method, shard strategy, node roles, heap and JVM, storage tiers, ingest capacity, scaling, cost and the monitoring loop. Cite it for capacity and sizing; this guide does not restate its configuration guidance.
- **Mapping, text analysis and indexing strategy for one platform** — owned by [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md). That guide owns mappings, analysis-as-configured, data-modelling patterns, aggregations-friendly schema, reindexing and schema evolution. Cite it for how analysis is configured; this guide treats analysis as a pipeline concept (§4), not as product settings.
- **Vector and approximate-nearest-neighbour retrieval** — owned by [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md): kNN, HNSW, IVF, product quantisation, distance metrics, filtering, hybrid search and the vendor landscape. The vector index appears here as a **second index type** (§3.6), not as a replacement for the inverted index.
- **The BM25 and FAISS/SCANN research notes** — owned by [ai_llm/rag/bm25_faiss_scann_research.md](ai_llm/rag/bm25_faiss_scann_research.md). Cross-referenced in §5, not restated.
- **The search-versus-RAG architectural comparison** — owned by [ai_llm/agentic_search_vs_rag_guide.md](ai_llm/agentic_search_vs_rag_guide.md). Cross-referenced in §9.
- **The regulatory frame** for §12 — owned by [data/data_compliance_frameworks.md](data/data_compliance_frameworks.md), [data/data_governance_framework.md](data/data_governance_framework.md) and [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md). This guide cross-references the legal frame and asserts no jurisdiction's rule.
- **The inverted index as an interview topic** — owned by the repo's system-design-interview guides; this guide owns it as *architecture*.

### 1.3 How to read the sections

§2 maps the tiers. §3 is the section this guide exists for — the index. §4 is the zero-coverage section — the query pipeline. §5 is ranking, §6 serving, §7 the write path, §8 evaluation. §9 says what RAG changed and did not. §10 maps the architecture onto real implementations. §11 lists failure modes; §12 is the section the guide earns its place with — entitlement and deletion; §13 works one full example; §14 lists anti-patterns; §15 audits the claims; §16 states what could not be verified. **Every section ends with a reference or reference table, not a trailing paragraph.**

| Section | Owns | Primary provenance |
|---|---|---|
| §2 | The tier map and where cost sits | [IR-BOOK], [ES] |
| §3 | The index and its data structures | [IR-BOOK], [PAPER], [ES] |
| §4 | Analysis, rewriting, execution model | [IR-BOOK], [PAPER] |
| §5 | Scoring, learning-to-rank, re-ranking, fusion | [IR-BOOK], [PAPER], [LUCENE] |
| §6 | Latency budget, sharding, caches | [ES], cross-ref platform guide |
| §7 | Segments, merges, refresh, translog | [ES], [LUCENE] |
| §8 | Judgments and metrics | [IR-BOOK], [PAPER] |
| §9 | What RAG changed and did not | cross-ref RAG guides |
| §10 | Platform mapping | [ES], [OS] |
| §11 | Failure modes | synthesis |
| §12 | Entitlement and deletion | architecture; compliance guides cross-ref |
| §13 | Cymbal Bank worked example | fictional illustration |
| §14 | Anti-patterns | synthesis |
| §15–§16 | Audit and provenance | all of the above |

## 2. The Stack Map

A retrieval system is a pipeline of tiers, each with a single owner and a characteristic cost. The tiers exist because the raw collection is expensive to interrogate directly, and the index is the bet that makes direct interrogation unnecessary. This section fixes the responsibilities and locates the cost; the rest of the guide develops the tiers that matter architecturally.

### 2.1 The tiers

| Tier | What it owns | What it produces | Where the cost sits |
|---|---|---|---|
| **Acquisition** | Getting the collection in: crawling, connectors, file shares, message stores, database change streams. | Raw documents plus a stable document identity. | Breadth and change-rate; re-crawl cost; identity churn. |
| **Extraction and parsing** | Turning a byte stream into text plus structure: PDF/Office/HTML/markup decoding, layout, headings, tables. | Text and fields. | Per-document CPU; format fragility; the quality ceiling of everything downstream. |
| **Enrichment** | Adding fields and derived terms: metadata, classification labels, entities, entitlement attributes, language detection. | An enriched document. | Throughput of the enrichment service; correctness of labels that later become filters. |
| **Indexing** | Analysis (§4), postings construction, term dictionary, doc values, segment writing. | Immutable segments; a searchable index. | Index-time CPU and I/O; merge amplification (§7). |
| **The query pipeline** | Parsing, analysis, rewriting, expansion, the execution model, early termination. | A scored, pruned candidate set. | Query-time CPU; the latency tail. |
| **Ranking** | Scoring, learning-to-rank features, re-ranking, fusion of signals. | An ordered result list. | Model inference; feature extraction; the top-k merge. |
| **Serving** | Sharding, fan-out, merge of per-shard top-k, caches, the fetch phase. | The rendered response. | Network hops; per-shard overhead; cache misses (§6). |
| **Feedback / evaluation** | Judgments, metrics, offline and online comparison, logging. | A verdict on whether the index was a good bet. | Human judgment time; the cost of measuring the wrong thing (§8). |

### 2.2 Two structural facts the map hides

1. **The tiers are not symmetric in reversibility.** Extraction and enrichment can be re-run and re-indexed at a cost bounded by the pipeline. The *index structure itself* — whether it is positional, what the term dictionary compresses, how entitlement is enforced — is the part that is hard or impossible to change in place (thesis, §1).
2. **The cost of a tier is a rate, not a total.** Acquisition, extraction and indexing pay per document change; the query pipeline and serving pay per query; evaluation pays per judgment. A design that is cheap per document and expensive per query is a *different* design from the reverse, and the bet in §1 is essentially a wager on which rate grows faster.

### 2.3 The read path versus the write path

The map above is the *conceptual* stack. Physically it collapses into two paths that share the same index and contend for the same resources: the **write path** (acquisition → extraction → enrichment → indexing → segments → merges, §7) and the **read path** (query → analysis → execution → ranking → serving, §4–§6). The platform guides own the capacity arithmetic of that contention; this guide owns the architecture of why it exists — an inverted index is built to be read, and every write mutates the structure the reads depend on.

| Boundary | This guide | Cited sibling |
|---|---|---|
| Sizing, topology, shard strategy, heap, tiers, cost | Not restated | [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md) |
| Mapping, analysis-as-configured, schema evolution | Not restated | [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md) |
| Vector/ANN retrieval and hybrid search | Second index type only (§3.6) | [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) |
| Search vs. RAG architecture | Cross-referenced (§9) | [ai_llm/agentic_search_vs_rag_guide.md](ai_llm/agentic_search_vs_rag_guide.md) |
| Regulatory frame for entitlement/deletion | Cross-referenced, not asserted (§12) | [data/data_compliance_frameworks.md](data/data_compliance_frameworks.md), [data/data_governance_framework.md](data/data_governance_framework.md) |

## 3. The Index

This is the section the guide exists for. Everything else assumes an index exists; this section is what an index *is*.

### 3.1 The inverted index and the postings list

The central data structure of information retrieval is the **inverted index**: for each term, the list of documents that contain it. The IR book builds it from a term-document incidence matrix — a matrix with a row per term and a column per document, whose entries are 1 (present) or 0 (absent) — and then inverts it so that lookups go *from term to documents* rather than from document to terms **[IR-BOOK]** ch. 1. The list stored against a term is its **postings list**; the IR book defines a *posting* as a docID in a postings list, so the postings list `(6; 20, 45, 100)` means termID 6 occurs in documents 20, 45 and 100 **[IR-BOOK]** ch. 5.

The reason the structure is inverted at all is the shape of the query: users supply terms and want documents. A forward structure answers the opposite question far more naturally and far less usefully for search.

### 3.2 The term dictionary

The **term dictionary** (also called the vocabulary) is the set of all terms, with a pointer from each term to its postings list. The dictionary is what makes term lookup sub-linear; the IR book treats dictionary search structures, wildcard matching and the k-gram index for wildcard queries as its own subject **[IR-BOOK]** ch. 3. Two structural facts follow: the dictionary is small relative to the postings (a collection has many more postings than distinct terms), and the dictionary is the part a query touches *first*, so its representation is latency-critical.

### 3.3 Positional versus non-positional indexes

A **non-positional** index stores, per term per document, at most a frequency. A **positional** index additionally stores the positions at which the term occurs. Positions are what make **phrase queries** and proximity queries possible: the IR book introduces biword indexes (indexing adjacent word pairs) as a poor precursor and positional indexes as the general solution, and notes the positional index's cost — it "is usually substantially larger than a non-positional index" **[IR-BOOK]** ch. 2 (positional indexes; combination schemes). This is a first instance of the thesis: a positional index is a bet that the queries you will be asked include phrases, and you pay for that bet in index size on every query that did not need it.

### 3.4 The forward index and its role in scoring

The **forward index** goes the other way: document → its terms or field values. It is the structure Lucene-family stacks call **doc values**, described by Elasticsearch as "an on-disk data structure that is built at document index time", storing "the same values as `_source`, but in a columnar format that is more efficient for sorting and aggregation" **[ES]** doc_values. The distinction matters architecturally: the inverted index answers "which documents match this term?" and is the right structure for *matching*; the forward index answers "what are this document's values?" and is the right structure for *scoring, sorting and aggregation* once a candidate set is known. A retrieval system needs both, and they are built at the same moment from the same document.

### 3.5 Compression schemes, by name

Postings lists are long and are compressed, and the compression schemes have names the architect should be able to say. All of the following are named in the IR textbook's index-compression chapter unless marked otherwise:

| Scheme | What it does | Provenance |
|---|---|---|
| **Gap (delta) encoding** | Stores the difference between consecutive docIDs rather than the docIDs, because the gaps are small and the docIDs are not. | **[IR-BOOK]** ch. 5 |
| **Variable-byte (VB) codes / VByte** | Encodes each gap in an integral number of bytes; the first bit of each byte is a continuation bit (1 on the last byte). "For most IR systems variable byte codes offer an excellent tradeoff between time and space." | **[IR-BOOK]** ch. 5 (Variable byte codes) |
| **Unary codes** | `n` ones followed by a zero; inefficient alone, used as the length part of gamma codes. | **[IR-BOOK]** ch. 5 (Gamma codes) |
| **Elias gamma (γ) codes** | Variable-length *bit*-level code splitting a gap into length (unary) and offset (binary with the leading 1 removed); "universal" — within a factor of the optimal for any distribution. The δ (delta) code is its sibling, given as an exercise. | **[IR-BOOK]** ch. 5 (Gamma codes); original: P. Elias, "Universal codeword sets and representations of the integers", *IEEE Trans. Information Theory*, 1975 **[PAPER]** |
| **Elias delta (δ) codes** | A refinement of gamma coding for large integers, encoding the length part with gamma coding. ⚠ The IR book presents δ only as an exercise and the Elias 1975 paper is the primary source; the specific "delta" form is attributed here to Elias. | **[IR-BOOK]** ch. 5; **[PAPER]** Elias 1975 (⚠ see §15) |
| **Bit-packing** | Packing a block of small integers into a fixed number of bits per value. | **[IR-BOOK]** ch. 5 (blocked storage, dictionary-as-a-string discuss the block model) |
| **Simple-9** | A word-aligned binary code that packs a variable number of small integers into a 32-bit word by reserving bits for the pattern. ⚠ Attributed to Anh & Moffat but **not verified at primary this pass**; see §15/§16. | ⚠ |
| **PForDelta ("PFD")** | A word-aligned code optimised for fast decoding of postings; Elasticsearch-adjacent and search-engine literature refer to a "New-PFD" variant. Primary source: M. Zukowski, S. Heman, N. Nes, P. Boncz, "Super-Scalar RAM-CPU Cache Compression", *ICDE 2006*, p. 59. | **[PAPER]** Zukowski et al. 2006 |
| **Block-based skip lists / skip pointers** | Auxiliary pointers that let the intersection of two postings lists skip large runs instead of advancing one posting at a time. | **[IR-BOOK]** ch. 2 (Faster postings list intersection via skip pointers) |

The compression chapter's own framing is worth carrying: the point of compression is not only disk — it is that "increased use of caching" and faster disk-to-memory transfer often make the *decompressed* structure faster, not just smaller **[IR-BOOK]** ch. 5. Compression is a latency decision disguised as a storage decision.

### 3.6 The vector index as a second index type

A **vector index** stores dense embeddings and answers nearest-neighbour queries (kNN) rather than term-match queries. It is a *second* index type over the same collection, not a replacement for the inverted index: it is built from the same documents, at the same time, for a different query model. Its structures — HNSW, IVF, product quantisation, the distance metrics, filtering and hybrid search — are owned by [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) and are **not** re-derived here. What this guide owns about it is the architectural contrast: the inverted index is an *exact* structure over discrete terms with an interpretable postings model; the vector index is an *approximate* structure over continuous space whose recall is a tunable parameter, and — as §12 develops — whose deletion semantics are materially worse.

| Index type | Query model | Exact or approximate | Deletion property (§12) |
|---|---|---|---|
| Inverted index | Term match / Boolean / ranked | Exact match, ranked by scoring proxy | Immutable segments; soft delete until merge |
| Doc values (forward index) | Sort / aggregate / score | Exact | Columnar, rebuilt on reindex |
| Vector index (ANN) | kNN in embedding space | Approximate (recall is a parameter) | Graph/quantised structures do not delete cleanly |

**References for §3.** Manning, Raghavan & Schütze, *Introduction to Information Retrieval*, CUP 2008, chs. 1–5 (incidence matrix, inverted index, postings, dictionary, positional indexes, skip pointers, compression) **[IR-BOOK]**; Elias, *IEEE Trans. Inf. Theory*, 1975 **[PAPER]**; Zukowski, Heman, Nes & Boncz, *ICDE 2006* **[PAPER]**; Elasticsearch, `doc_values` and near-real-time pages **[ES]**; Lucene 9.0.0 `org.apache.lucene.search` package summary **[LUCENE]**.

## 4. The Query Pipeline

The query pipeline is the zero-coverage section: no other guide in the repo owns "query processing" end to end. It has two halves — **analysis and rewriting** (turning an information need into a query) and the **execution model** (turning a query into a scored top-k). Both are places where the index's bet is paid or broken.

### 4.1 The analysis chain

Analysis is the chain applied identically to documents at index time and to queries at query time. Apply it asymmetrically and recall silently collapses, because the term the user typed no longer matches the term that was stored.

| Step | What it does | Named provenance |
|---|---|---|
| **Tokenisation** | Chops a character stream into tokens. "Tokenization is the process of chopping character streams into tokens, while linguistic preprocessing then deals with building equivalence classes of tokens which are the set of terms that are indexed." | **[IR-BOOK]** ch. 2 |
| **Normalisation (case folding, accent folding)** | Maps tokens into equivalence classes so that `USA`/`usa` and `naïve`/`naive` match. The IR book treats capitalisation/case-folding and accents/diacritics as the two canonical normalisation decisions. | **[IR-BOOK]** ch. 2 (Normalization; Accents and diacritics; Capitalization/case-folding) |
| **Stop-word handling** | Drops very common terms that carry little discriminating signal. The trade-off is explicit: dropping them saves postings and speeds queries but can remove the terms a phrase or a title needs. | **[IR-BOOK]** ch. 2 (Dropping common terms: stop words) |
| **Stemming / lemmatisation** | Reduces inflected and derivationally related forms to a common base. *Stemming* is "a crude heuristic process that chops off the ends of words"; *lemmatisation* does "full morphological analysis" to a dictionary form. | **[IR-BOOK]** ch. 2 (Stemming and lemmatization) |

Stemmers have names and dates, and they belong to the people who wrote them:

| Stemmer | Attribution | Provenance |
|---|---|---|
| **Porter stemmer** | Porter, 1980 — "the most common algorithm for stemming English, and one that has repeatedly been shown to be empirically very effective". Five phases of word reductions applied sequentially. | **[IR-BOOK]** ch. 2 (citing Porter, 1980) |
| **Lovins stemmer** | The "older, one-pass Lovins stemmer" — Lovins, 1968. | **[IR-BOOK]** ch. 2 |
| **Paice/Husk stemmer** | Paice, 1990. | **[IR-BOOK]** ch. 2 |
| **Krovetz stemmer** | A dictionary/linguistically motivated English stemmer. ⚠ Named in this guide but **not verified at a primary source this pass**; see §16. | ⚠ |
| **Snowball** | "A small string processing language for creating stemming algorithms for use in Information Retrieval", "originally designed and built by Martin Porter"; the English Snowball stemmer maps *connection, connections, connective, connected, connecting* to *connect*. | **[WEB]** snowballstem.org |

The IR book's own verdict on normalisation is worth quoting for its honesty: either form of normalisation "tends not to improve English information retrieval performance in aggregate — at least not by very much", and "stemming increases recall while harming precision" **[IR-BOOK]** ch. 2. The choice is a bet about queries, and the textbook says so.

### 4.2 Classical query rewriting

Before execution, a query is often rewritten. The classical techniques, all owned by the IR textbook's chapters 3 and 9:

| Technique | What it does | Provenance |
|---|---|---|
| **Spelling correction** | Detects and corrects query-term errors, using edit distance and/or k-gram indexes; requires a source of spelling variants. | **[IR-BOOK]** ch. 3 (Spelling correction; Edit distance; k-gram indexes) |
| **Synonym expansion / query expansion** | Adds terms to the query from a controlled vocabulary or thesaurus so that documents phrased differently match. | **[IR-BOOK]** ch. 9 (Query expansion; Vocabulary tools for query reformulation) |
| **Query relaxation** | Loosens constraints (drops the least important term, removes a filter) when a query returns too few results — the opposite direction from expansion. | **[IR-BOOK]** ch. 9 (Global methods for query reformulation) |
| **Relevance feedback** | Uses the user's judgments of an initial result set to move the query towards the relevant documents. | **[IR-BOOK]** ch. 9 (Relevance feedback and pseudo relevance feedback) |
| **The Rocchio algorithm** | "The classic algorithm for implementing relevance feedback"; it models incorporating feedback into the vector-space model, moving the query vector towards relevant and away from non-relevant documents. | **[IR-BOOK]** ch. 9 (The Rocchio algorithm for relevance feedback), citing **Rocchio (1971)** **[PAPER]** |
| **Pseudo-relevance feedback** | Assumes the top-ranked documents are relevant and performs relevance feedback automatically, with no user in the loop. | **[IR-BOOK]** ch. 9 (Pseudo relevance feedback) |

**The RAG-era rewrites are deliberately out of scope.** HyDE, multi-query generation and LLM-based query rewriting are *consumers* of retrieval, and they belong to the sibling RAG guides — [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) and the RAG guides under `ai_llm/rag/`. They are named here only to fix the boundary: the classical machinery above is the *architecture* of query rewriting; the RAG-era variants are a *model class* applied to it, not a replacement for it.

### 4.3 The execution model, and early termination by name

How a query is evaluated against the postings is a first-class architectural choice, because it decides what the system can prune.

**Two execution strategies, named.** The two basic techniques for traversing the index are **Document-At-A-Time (DAAT)** and **Term-At-A-Time (TAAT)**; "for conjunctive queries, DAAT is often preferred, while many optimized approaches for disjunctive queries use TAAT" **[PAPER]** Ding & Suel 2011, citing Turtle & Flood 1995. The distinction and the optimisation analysis originate in H. Turtle and J. Flood, "Query evaluation: strategies and optimizations", *Information Processing & Management*, 31(6):831–850, 1995 **[PAPER]**. In TAAT the system walks one term's postings at a time and accumulates partial scores in an accumulator array; in DAAT the system walks all terms' postings in lock-step by document, so it can score and discard a document as soon as it is complete.

**Early termination, named.** "Early termination" is the optimisation that avoids scoring documents that cannot enter the top-k ; "the current state-of-the-art methods … the WAND algorithm by Broder et al. [and the approach of Strohman and Croft] achieve great benefits" **[PAPER]** Ding & Suel 2011. The names and dates:

| Technique | Attribution | What it does | Provenance |
|---|---|---|---|
| **WAND** | A. Z. Broder, D. Carmel, M. Herscovici, A. Soffer, J. Y. Zien, "Efficient query evaluation using a two-level retrieval process", *CIKM 2003*, pp. 426–434 | A "two-level" process: iterate in parallel over query-term postings, identify candidate documents by approximate evaluation, then fully score only candidates — pruning documents that cannot reach the top-k. Named "WAND" in the later literature. | **[PAPER]** Broder et al. 2003; named in **[PAPER]** Ding & Suel 2011 |
| **Block-max WAND (BMW)** | S. Ding and T. Suel, "Faster top-k document retrieval using block-max indexes", *SIGIR 2011* | Augments each compressed block of an inverted list with its maximum impact score (a **block-max index**), enabling WAND to skip whole blocks; performs DAAT traversal. | **[PAPER]** Ding & Suel 2011 |
| **MaxScore** | Often credited to Turtle & Flood, 1995 | Orders processing so that terms with the highest potential contribution are scored first, allowing safe pruning once the top-k threshold exceeds the remaining terms' maximum possible contribution. ⚠ The specific *name* "MaxScore" was **not verified at primary this pass**; see §15/§16. | ⚠ |

**The platform's execution model, named.** Lucene's own search-algorithm appendix describes the iteration as a `DocIdSetIterator.nextDoc()` loop: a `Scorer` "return[s] iterators over matches", and for a multi-term query the scorer advances by document. This is a **document-at-a-time** iteration model, and Lucene exposes the pruning machinery as `BlockMaxDISI` ("skips non-competitive docs by checking the max score of the provided Scorer for the current block") and `ImpactsDISI` ("skips non-competitive docs thanks to the indexed impacts") **[LUCENE]** `org.apache.lucene.search` package summary. The architectural point: the textbook names appear in the platform as concrete iterators, and the bet — early termination assumes the top-k is small relative to the collection — is baked into the platform's data structures.

**References for §4.** Manning, Raghavan & Schütze, *Introduction to Information Retrieval*, CUP 2008, chs. 2, 3, 7, 9 **[IR-BOOK]**; Turtle & Flood, *Inf. Process. Manage.* 31(6):831–850, 1995 **[PAPER]**; Broder, Carmel, Herscovici, Soffer & Zien, *CIKM 2003*, pp. 426–434 **[PAPER]**; Ding & Suel, *SIGIR 2011* **[PAPER]**; Rocchio, 1971, as cited in **[IR-BOOK]**; Snowball, snowballstem.org **[WEB]**; Lucene 9.0.0 `org.apache.lucene.search` **[LUCENE]**.

## 5. The Ranking Stack

Matching is binary; ranking is a model. A document either matches the query or it does not, and then a **scoring function** assigns it a real number that is used to order the matched set. The textbook is explicit that this number is only a proxy: cosine similarity "is only a proxy for the user's perceived relevance" **[IR-BOOK]** ch. 7. The ranking stack is the family of proxies.

### 5.1 The lexical scoring family, with original citations

| Model | Definition | Original provenance |
|---|---|---|
| **TF-IDF** | The product of term frequency and inverse document frequency: `tf-idf(t,d) = tf(t,d) × idf(t)`. It is highest when a term occurs many times in a few documents, lower when it occurs in many, and lowest when it occurs in almost all. | **[IR-BOOK]** ch. 6 (Tf-idf weighting) |
| **The vector-space model (VSM)** | Documents and queries are represented as vectors in a common term space; the score is a similarity between the query vector and each document vector. | **[IR-BOOK]** ch. 6 (The vector space model for scoring) |
| **Okapi BM25** | A probabilistic model sensitive to term frequency and document length, "often called Okapi weighting, after the system in which it was first implemented". Score sums over query terms an idf factor times a saturated tf factor, with parameters `k1` (tf saturation) and `b` (length normalisation); the textbook's reasonable defaults are `k1` between 1.2 and 2 and `b = 0.75`. | **[IR-BOOK]** ch. 11 (Okapi BM25: a non-binary model), citing Spärck Jones et al. 2000; original: Robertson, Walker, Jones, Hancock-Beaulieu & Gatford, "Okapi at TREC-3", *TREC-3*, 1994 **[PAPER]** |

**The BM25 research notes are owned elsewhere.** The repo's [ai_llm/rag/bm25_faiss_scann_research.md](ai_llm/rag/bm25_faiss_scann_research.md) owns the BM25 and FAISS/SCANN research; this section cites it and does not restate the derivations.

**Lucene's practical BM25, by name.** Lucene implements BM25 as `BM25Similarity`, whose Javadoc records the origin — "Introduced in Stephen E. Robertson, Steve Walker, Susan Jones, Micheline Hancock-Beaulieu, and Mike Gatford. Okapi at TREC-3. In Proceedings of the Third Text REtrieval Conference (TREC 1994)" — and whose defaults are `k1 = 1.2`, `b = 0.75`, `discountOverlaps = true`, with `idf` "implemented as `log(1 + (docCount - docFreq + 0.5)/(docFreq + 0.5))`" **[LUCENE]** `BM25Similarity`. The IDF variant Lucene uses is the smoothed form; the textbook's alternative `log[(N − df + 0.5)/(df + 0.5)]` can go negative when a term occurs in over half the documents, which is one reason the smoothed form exists **[IR-BOOK]** ch. 11.

### 5.2 Learning to rank, with named formulations

Learning-to-rank (LTR) replaces a hand-written scoring function with a learned one. The formulations are named, and so are their papers:

| Formulation | Idea | Named reference |
|---|---|---|
| **Pointwise** | Treat each document as an independent example with a relevance label; learn a regression/classification function. | Described as a family in the LTR literature; formalised contrast in **[PAPER]** Cao et al. 2007 |
| **Pairwise** | Treat ordered document *pairs* as the examples; learn to order the pair correctly. | **RankNet**: C. Burges, T. Shaked, E. Renshaw, A. Lazier, M. Deeds, N. Hamilton, G. Hullender, "Learning to Rank using Gradient Descent", *ICML 2005* **[PAPER]**. **RankSVM**: T. Joachims, "Optimizing search engines using clickthrough data", *KDD 2002* **[PAPER]** |
| **Listwise** | Treat the whole ranked *list* as the example; optimise a list-level objective. | **ListNet**: Z. Cao, T. Qin, T.-Y. Liu, M.-F. Tsai, H. Li, "Learning to Rank: From Pairwise Approach to Listwise Approach", *ICML 2007* **[PAPER]** |
| **LambdaMART** | Boosted-tree ranking built on LambdaRank, which is built on RankNet; optimises a ranking metric through the "lambda" gradient. | C. Burges, "From RankNet to LambdaRank to LambdaMART: An Overview", Microsoft Research Technical Report **MSR-TR-2010-82**, 2010 **[PAPER]** |

The IR book frames the same idea from the other side: the weights of a scoring function should be set "to optimize performance on a development test collection … either manually or with optimization methods such as grid search or something more advanced" **[IR-BOOK]** ch. 11. LTR is that sentence industrialised.

### 5.3 Re-ranking as a separate model class

Re-ranking is a **second-stage** model applied to the top-k of a first-stage retriever, and it is a *separate model class*, not a bigger lexical score. The distinction that matters architecturally is **bi-encoder versus cross-encoder**: a bi-encoder embeds query and document independently (so documents can be pre-computed and indexed, enabling ANN retrieval), whereas a cross-encoder scores the query and document *together* (so it cannot be pre-computed, and can only be run on a small candidate set). This is the structural reason the two-stage pattern exists: retrieval must be cheap and indexable; re-ranking may be expensive because it sees few candidates. The RAG-oriented treatment of re-rankers is owned by the sibling RAG guides; the architecture point — a cascade whose first stage is recall-oriented and whose later stages are precision-oriented — belongs here.

### 5.4 Score fusion across signals

When several ranked lists or several signals must be combined, the fusion rule is an architectural choice with named options:

| Fusion method | Rule | Provenance |
|---|---|---|
| **Linear combination** | A weighted sum of normalised scores; the weights must be calibrated or the scores drown one another. Lucene exposes this pattern directly ("combine the similarity score and the feature score using a linear combination"). | **[LUCENE]** package summary (Integrating field values into the score) |
| **Reciprocal Rank Fusion (RRF)** | `RRFscore(d) = Σ over rankings r of 1/(k + r(d))`, where `r(d)` is the rank of document `d` in ranking `r` and `k = 60` "was fixed during a pilot investigation and not altered during subsequent validation". It combines *ranks*, not scores, so no calibration is needed. | **[PAPER]** G. V. Cormack, C. L. A. Clarke, S. Büttcher, "Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods", *SIGIR 2009*, pp. 758–759 |

The architectural property of RRF — rank-based, calibration-free, streaming — is precisely why it survives as a default for hybrid lexical/vector fusion; the vector-DB guide owns the hybrid-search configuration, and this section owns the fusion *rule*.

**References for §5.** Manning, Raghavan & Schütze, *IR*, CUP 2008, chs. 6, 7, 11 **[IR-BOOK]**; Robertson et al., *TREC-3*, 1994 **[PAPER]**; Spärck Jones et al., 2000 (as cited in **[IR-BOOK]**); Burges et al., *ICML 2005* **[PAPER]**; Joachims, *KDD 2002* **[PAPER]**; Cao et al., *ICML 2007* **[PAPER]**; Burges, MSR-TR-2010-82 **[PAPER]**; Cormack, Clarke & Büttcher, *SIGIR 2009* **[PAPER]**; Lucene `BM25Similarity` and `org.apache.lucene.search` **[LUCENE]**.

## 6. The Serving Tier

The serving tier turns a scored candidate set into a response, across shards and replicas. It is where the architecture's read path becomes latency.

### 6.1 Where the latency budget is spent

A single query's wall-clock time decomposes into a small number of stages, each of which some tier owns:

| Stage | What happens | Owner |
|---|---|---|
| **Query parsing and analysis** | The query string is parsed and analysed (§4.1). | The coordinating/serving node, or the application. |
| **Per-shard fan-out** | The query is broadcast to one copy of every shard; the cost is the *slowest* shard, not the average. | Coordination. |
| **Per-shard scoring** | Each shard scores its local documents (execution model, §4.3). | Data nodes. |
| **Merge of the top-k** | Per-shard top-k lists are merged into the global top-k; k must be gathered from every shard. | Coordination. |
| **Fetch phase** | The stored fields (`_source`) of the winning documents are retrieved and returned. | Data nodes, then coordination. |

Two structural consequences follow. First, latency is governed by the **tail**: because a query waits for every shard, one slow shard sets the floor, which is why shard count multiplies fan-out cost and why the capacity guide owns the sizing arithmetic. Second, the **fetch phase is separate from the match phase** for a reason — matching only needs the index (postings and doc values); returning a document needs its stored fields, so the system deliberately defers the expensive retrieval of full documents until the final k is known.

### 6.2 Sharding and replication as architecture

Sharding partitions the index so that no single node holds the whole collection; replication copies each shard so that reads scale and failures do not lose data. This guide treats both as **architecture** — the reasons they exist and the invariant they impose (a query must see every shard) — and does **not** restate their configuration. The sizing method, shard strategy, replica arithmetic, node roles and the operational consequences are owned by [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md).

### 6.3 Cache layers, and what each caches

Caching is where a retrieval system spends memory to buy latency. The layers are distinct, and confusing them is a common source of mis-tuned clusters:

| Cache | What it holds | Provenance |
|---|---|---|
| **Filter / query cache** | The document IDs matching a previously seen filter clause, so the same filter is not recomputed. Lucene exposes an LRU query cache and a usage-tracking policy deciding "which filters should be cached". | **[LUCENE]** `LRUQueryCache`, `UsageTrackingQueryCachingPolicy`, `QueryCachingPolicy` |
| **Request cache** | The full response to a previously seen (and still-valid) search request — a larger, coarser win that is invalidated by any refresh. | **[ES]** (product-level; cross-ref platform guide for settings) |
| **OS page cache** | The operating system's cache of index files read from disk; often the single largest and most effective cache, and the reason the heap is not the only memory that matters. | **[ES]** near-real-time (filesystem cache); cross-ref platform guide |
| **Doc-values / field-data cache** | Field values loaded for sorting and aggregation; the forward-index payload (§3.4). | **[ES]** doc_values; cross-ref platform guide |

The architectural rule: **each cache buys a different thing, and they are invalidated by different events.** Segment-level caches are invalidated by merges and refreshes; request-level caches by any change to the index; the page cache by nothing the application controls. A cache that is invalidated on every write is not a cache on a write-heavy index.

### 6.4 The read-versus-write asymmetry

An inverted index is built to be read many times for each time it is written. That asymmetry is the reason the optimisations in §4.3 (early termination) and §6.3 (caching) are worth so much, and the reason the write path (§7) is so much more expensive per document than the read path is per query. The platform guides price the asymmetry; this guide names it as the structural cause.

| Cache / stage | Invalidated by | Cost when it misses |
|---|---|---|
| Filter cache | Segment change (merge/refresh) | Recompute matching doc IDs |
| Request cache | Any refresh of the index | Re-run the whole query |
| Page cache | Eviction under memory pressure | Disk read of index files |
| Doc-values cache | Eviction / segment change | Re-read field values |

**References for §6.** Lucene 9.0.0 `org.apache.lucene.search` **[LUCENE]**; Elasticsearch near-real-time and `doc_values` pages **[ES]**; sharding/sizing configuration is cross-referenced to [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md) and not restated.

## 7. The Write Path

The write path is what makes the index a *live* structure rather than a batch artefact, and it is where the platform's segment model does the real work. The model is not a generic "database writes a row"; it is an append-and-merge model, and the appendix is the index's bet about queries being paid for at write time.

### 7.1 Ingest: batch and streaming

Ingest arrives in two shapes with different costs. **Batch** ingest amortises analysis and merge over large documents-per-second and pays in latency (the collection is stale between batches). **Streaming** ingest pays continuously and buys freshness. Both feed the same analysis chain (§4.1) and the same segment writer; the difference is the rate and the acceptable staleness. The capacity guide owns ingest throughput planning; this section owns what ingest *does* to the index.

### 7.2 The segment model, as the platform documents it

The platform documents the model in its own words. Elasticsearch states:

> "A shard in Elasticsearch is a Lucene index, and a Lucene index is broken down into segments. Segments are internal storage elements in the index where the index data is stored, and are immutable. Smaller segments are periodically merged into larger segments to keep the index size at bay and to expunge deletes." **[ES]** index-modules-merge

Three properties follow directly from that sentence, and they are the whole of the write path's architecture:

1. **Segments are immutable once written.** Nothing is edited in place; a write creates a new segment, and a delete marks documents deleted rather than rewriting them.
2. **Merges are how space and correctness are reclaimed.** "To expunge deletes" is the crucial phrase: a deleted document is only physically gone once a merge rewrites the segment without it.
3. **Merges are scheduled and throttled.** The merge scheduler "controls the execution of merge operations when they are needed"; merges run on a dedicated `merge` thread pool; "smaller merges are prioritized over larger ones"; and merges are disk-I/O throttled so that "bursts … are smoothed out in order to not impact indexing throughput". Available disk space is monitored so that no new merge is scheduled when space is low, with a disk watermark that defaults to `95%` **[ES]** index-modules-merge. Elasticsearch describes the overall mechanism as "auto-throttling to balance the use of hardware resources between merging and other activities like search" **[ES]**.

The architectural reading: the index is a **log-structured** structure in disguise. Writes append segments; background merges compact them; deletes are logical until compaction. Every one of those properties has a consequence for entitlement and erasure in §12.

### 7.3 Refresh, flush and commit — the near-real-time cost

Three operations are routinely confused, and the confusion is architectural:

| Operation | What it does | Cost | Provenance |
|---|---|---|---|
| **Refresh** | Writes the in-memory buffer to a new segment and opens it for search; makes documents visible "in near real-time — within 1 second". Default `index.refresh_interval` is `1s` in the Elastic Stack. | Cheap-ish, but "creates less efficient index constructs (tiny segments) that must later be merged". | **[ES]** near-real-time; docs-refresh |
| **Flush** | Performs a Lucene **commit** and starts a new translog generation; "performed automatically in the background … to make sure the translog does not grow too large". | Expensive (durable commit). | **[ES]** index-modules-translog |
| **Commit** | Persists changes to disk so they survive a crash; the durable operation behind a flush. | Expensive; "cannot be performed after every index or delete operation". | **[ES]** index-modules-translog |

The **translog** sits between refresh and commit: because Lucene commits are too expensive to run per operation, "each shard copy also writes operations into its transaction log known as the translog. All index and delete operations are written to the translog after being processed by the internal Lucene index but before they are acknowledged", and are recovered from it after a crash **[ES]** index-modules-translog. The default `index.translog.durability` is `request`, meaning success is reported only after the translog is `fsync`ed and committed on the primary and every replica **[ES]**.

The bet emerges clearly here: **trading durability and freshness for throughput**. `refresh=true` (or a short `refresh_interval`) buys immediacy and pays in tiny segments and merge pressure; `refresh=false` (the default for writes) buys throughput and accepts that a document is not searchable "immediately" **[ES]** docs-refresh. `translog.durability=async` buys write throughput and accepts losing acknowledged writes since the last background commit on failure **[ES]**.

### 7.4 What makes an index write-expensive

| Cost driver | Why | Provenance |
|---|---|---|
| **Analysis** | Every document is tokenised, normalised, stop-worded and stemmed; enrichment may run ML models. | **[IR-BOOK]** ch. 2; **[ES]** ingest pipelines |
| **Postings construction** | Building the inverted lists and the term dictionary for the new documents. | **[IR-BOOK]** ch. 4 (index construction) |
| **Term dictionary growth** | New terms expand the dictionary, which is latency-critical and must be searched. | **[IR-BOOK]** ch. 3, ch. 5 |
| **Merge amplification** | Merges rewrite data repeatedly as segments are compacted, so a document may be written to disk several times over its life. Merge is I/O-heavy and contends with search. | **[ES]** index-modules-merge |
| **Deletes as tombstones / soft deletes** | Deleting marks documents deleted; the space and the data persist until a merge "expunges" them. | **[ES]** index-modules-merge |
| **Doc values** | The forward index is "built at document index time", so every field that must be sortable or aggregatable adds index-time work and disk. | **[ES]** doc_values |

**References for §7.** Elasticsearch `index-modules-merge`, `docs-refresh`, `near-real-time`, `index-modules-translog` and `doc_values` pages **[ES]**; Manning, Raghavan & Schütze, *IR*, CUP 2008, chs. 2–5 **[IR-BOOK]**; ingest throughput planning is cross-referenced to [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md).

## 8. Evaluation

Evaluation is how a retrieval system learns whether the index was a good bet. It rests on relevance judgments, which are human, expensive and imperfect — and on metrics, which are exact only when defined exactly.

### 8.1 How relevance judgments are actually obtained

For tiny collections, exhaustive judgments of every (query, document) pair were obtained; for modern collections that is impossible, and "it is usual for relevance to be assessed only for a subset of the documents for each query" **[IR-BOOK]** ch. 8 (Assessing relevance). The standard method is **pooling**:

> "The most standard approach is *pooling*, where relevance is assessed over a subset of the collection that is formed from the top *k* documents returned by a number of different IR systems (usually the ones to be evaluated), and perhaps other sources such as the results of Boolean keyword searches or documents found by expert searchers." **[IR-BOOK]** ch. 8

Two honest consequences follow, and they are architectural, not academic. First, **pooled judgments inherit the bias of the systems that produced them**: if no system in the pool retrieved a relevant document, that document is not judged, and a *new* system that would have retrieved it is scored as if it had retrieved a non-relevant one. The pool defines the ceiling of what any evaluation can see. Second, **human judgments are idiosyncratic**: the IR book notes that "a human is not a device that reliably reports a gold standard judgment of relevance", and that inter-judge agreement on binary relevance "normally falls in the range of 'fair' (0.67–0.8)" by the kappa statistic, where kappa above 0.8 is "good agreement" **[IR-BOOK]** ch. 8. `kappa = (P(A) − P(E))/(1 − P(E))`, correcting observed agreement `P(A)` for chance agreement `P(E)` **[IR-BOOK]** ch. 8.

### 8.2 Each metric, defined precisely

The definition matters. A metric named without its cutoff, its normalisation and its undefined cases is not a measurement.

**Precision, recall, F-measure (set-based, unranked).**

- **Precision** `P = tp/(tp+fp)` = "the fraction of retrieved documents that are relevant" **[IR-BOOK]** ch. 8.
- **Recall** `R = tp/(tp+fn)` = "the fraction of relevant documents that are retrieved" **[IR-BOOK]** ch. 8.
- **F-measure**: `F = 1/(α·(1/P) + (1−α)·(1/R))`, the weighted harmonic mean; with `α = 1/2` the balanced **F₁ = 2PR/(P+R)** **[IR-BOOK]** ch. 8. The harmonic mean (not the arithmetic) is used because returning all documents trivially yields recall 1 and would give an arithmetic-mean floor that is easy to game **[IR-BOOK]** ch. 8. **Undefined when:** `P+R = 0` (the harmonic mean is 0/0); and the set of relevant documents is unknown, which is precisely what recall requires.

**Ranked metrics.**

| Metric | Exact definition | Where it is undefined |
|---|---|---|
| **Precision@k (P@k)** | Precision computed over the top *k* retrieved documents. The IR book calls this "Precision at *k*", "the least stable of the commonly used evaluation measures", and notes it "does not average well" because the number of relevant documents per query strongly influences it. | *k* must be fixed; undefined if *k* exceeds the result-set size; depends on how many relevant documents exist. |
| **Interpolated precision / 11-point interpolated average precision** | `p_interp(r) = max_{r' ≥ r} p(r')` — "the highest precision found for any recall level *r' ≥ r*". The 11-point measure samples the interpolated precision at recall levels 0.0, 0.1, …, 1.0 and averages per level. | Undefined without a chosen set of recall levels; 11-point was "used for instance in the first 8 TREC Ad Hoc evaluations". |
| **Average Precision (AP) / MAP** | For one information need, AP is "the average of the precision value obtained for the set of top *k* documents existing after each relevant document is retrieved", and MAP is the arithmetic mean of AP over information needs: `MAP(Q) = (1/|Q|)·Σ_j (1/m_j)·Σ_{k=1..m_j} Precision(R_{jk})`. "When a relevant document is not retrieved at all, the precision value in the above equation is taken to be 0." | Undefined if `m_j = 0` (a query with no relevant documents) — such queries are conventionally excluded. Unjudged documents count as non-relevant, so MAP inherits the pooling bias. MAP weights each information need equally regardless of how many documents are relevant. |
| **Mean Reciprocal Rank (MRR)** | The mean over queries of the reciprocal of the rank of the first relevant (correct) answer, `(1/|Q|)·Σ 1/rank_i`. Introduced in the TREC-8 Question Answering track. | Undefined when no correct answer is retrieved (that query's reciprocal rank is 0, conventionally included). Only measures the *first* correct hit; blind to recall of later ones. |
| **DCG (Discounted Cumulative Gain)** | `DCG[i] = CG[i]` for `i < b`, else `DCG[i] = DCG[i−1] + CG[i]/log_b(i)` where `CG[i] = CG[i−1] + G[i]`. No discount is applied at rank 1 (because `log_b(1) = 0`) or below the log base. Original source defines it for graded relevance scores 0–3 and default base `b = 2`. | Undefined without a chosen gain scale and log base; the "gain" values are a design choice. |
| **nDCG (normalised DCG)** | The DCG divided by the **ideal** DCG — the DCG of the theoretically best ordering, built by placing the highest-graded documents in the top positions: "fill the vector positions 1, …, m by the values 3, then … 2, then … 1, and finally the remaining positions by the values 0." This is the "relative-to-the-ideal performance". | Undefined if the ideal DCG is 0 (no relevant documents at all). |

The DCG/nDCG definitions are from K. Järvelin and J. Kekäläinen, "Cumulated Gain-based Evaluation of IR Techniques", *ACM Transactions on Information Systems*, 20(4):422–446, October 2002 **[PAPER]**. The paper's own framing: graded judgments and cumulated gain exist because binary judgments give "equal credit … for retrieving highly and marginally relevant documents" **[PAPER]** Järvelin & Kekäläinen 2002.

### 8.3 Offline versus online evaluation

Offline evaluation measures against pooled judgments and is cheap, repeatable and biased by its pool (§8.1). Online evaluation measures real user behaviour and is faithful but expensive and hard to attribute. **Interleaving** is the online technique that compares two systems by mixing their results into one list and observing which system's results the user engages with; it is the online counterpart to offline A/B comparison and it removes the need for absolute relevance judgments. ⚠ The canonical interleaving attribution was **not verified at a primary source this pass**; see §16. Joachims's clickthrough-optimisation paper (verified, *KDD 2002*) established that clickthrough data can train a ranking function and is the anchor for the online-evaluation family **[PAPER]** Joachims 2002.

### 8.4 The honest statement

**Pooled judgments inherit the bias of the systems that produced them.** A metric computed over a pool is a measurement of *relative* behaviour within the pool, not an absolute measure of retrieval quality. Two systems compared on a pool that neither could escape are being compared on a narrow surface. This is why "the evaluation that measures the wrong thing" is listed as a failure mode in §11, and why §15 keeps the metrics and their sources separate from any claim about a particular system's quality.

| Metric | Counts | Normalisation | Undefined / caveat |
|---|---|---|---|
| Precision | Relevant ∩ retrieved / retrieved | None | Undefined set of relevant docs |
| Recall | Relevant ∩ retrieved / relevant | None | Needs known relevant set |
| F₁ | Harmonic mean of P, R | α = 1/2 | Undefined if P+R = 0 |
| P@k | Relevant in top k / k | fixed k | Unstable; k dependent |
| 11-pt interpolated AP | Interpolated precision at 11 recall levels | Per recall level | Recall-level grid is a choice |
| MAP | Mean of per-query average precision | Per query, equal weight | Queries with no relevant docs excluded |
| MRR | Mean 1/rank of first correct | Per query | Only first hit; 0 if none |
| DCG | Graded gain, log-discounted | Log base b | Gain scale and base are choices |
| nDCG | DCG / ideal DCG | By ideal ordering | Undefined if ideal DCG = 0 |

**References for §8.** Manning, Raghavan & Schütze, *IR*, CUP 2008, ch. 8 (evaluation; pooling; precision/recall/F; P@k; interpolated precision; MAP; kappa) **[IR-BOOK]**; Järvelin & Kekäläinen, *ACM TOIS* 20(4):422–446, 2002 **[PAPER]**; Voorhees, "The TREC-8 Question Answering Track Report", *TREC-8*, 1999 **[PAPER]**, for MRR; Joachims, *KDD 2002* **[PAPER]**, for clickthrough/online evaluation.

## 9. The Classical and the Modern

Retrieval did not change its foundations when RAG arrived; it gained a new front end and a new consumer. Sorting what changed from what did not is the difference between extending the architecture and replacing it.

### 9.1 What RAG changed about retrieval

| Change | What actually happened | Where it is owned |
|---|---|---|
| **Dense and hybrid retrieval** | A vector index (§3.6) became a first-class retrieval structure alongside the inverted index, and hybrid search combined the two. | [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) |
| **Re-ranking as a standard stage** | The two-stage cascade (§5.3) stopped being an optional optimisation and became the default shape: retrieve broadly, re-rank with a cross-encoder. | RAG guides; architecture here (§5.3) |
| **The query-rewriting revival** | HyDE and multi-query generation re-popularised the idea that the user's string is not the query — a classical §4.2 idea with a model-class implementation. | RAG guides |

### 9.2 What RAG did *not* change

The index still exists. The write path still exists — segments, merges, refresh, translog (§7). The evaluation problem still exists and is *worse*, because a generated answer adds a second thing to evaluate (groundedness) on top of the retrieval that fed it. And the entitlement problem still exists, unchanged, because a paragraph of retrieved text handed to a model is the same disclosure as a document returned to a user if the user was not entitled to it (§12). RAG is a **consumer** of retrieval, not a replacement for its architecture.

### 9.3 The boundary, stated

The search-versus-RAG *architectural comparison* is owned by [ai_llm/agentic_search_vs_rag_guide.md](ai_llm/agentic_search_vs_rag_guide.md). This guide's claim is narrower and structural: the modern retrieval stack is the classical retrieval stack with a vector index bolted on and a re-ranker bolted after — and the classical stack's bets (positional or not, entitlement at query or index time, what the pool can see) are still the bets that decide the system.

| Concern | Changed by RAG? | Still owned by this guide |
|---|---|---|
| Matching model | Partly (dense added as a second index) | §3.6 |
| Query pipeline | Front end extended (rewrites) | §4 |
| Ranking | Re-ranking promoted to a standard stage | §5 |
| Write path | No | §7 |
| Evaluation | Harder, not different in kind | §8 |
| Entitlement / deletion | No | §12 |

**References for §9.** [ai_llm/agentic_search_vs_rag_guide.md](ai_llm/agentic_search_vs_rag_guide.md) (architectural comparison); [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) (vector retrieval); this guide §4, §5, §12 for the architecture that survives.

## 10. The Platform Mapping

This guide is not a platform guide, so this section is deliberately structural and brief. It shows how the architecture above lands on a real implementation, and defers all configuration to the sibling guides.

### 10.1 Lucene-based stacks

Elasticsearch and OpenSearch are the dominant Lucene-based stacks, and the mapping of this guide's concepts onto them is direct:

| This guide | Elasticsearch / OpenSearch realisation | Provenance |
|---|---|---|
| Inverted index | Per-shard Lucene index; postings per field | **[ES]** index-modules-merge |
| Segment | Lucene segment; immutable; merged periodically | **[ES]** |
| Forward index / doc values | `doc_values` columnar on-disk structure | **[ES]** doc_values |
| Query pipeline | Analysis chain + query DSL rewriting to Lucene queries | **[LUCENE]**, `Query.rewrite` |
| Execution model | Document-at-a-time `DocIdSetIterator` iteration, with block-max/impacts pruning | **[LUCENE]** package summary |
| Ranking | `Similarity` (e.g. `BM25Similarity`) + re-scoring stages | **[LUCENE]** |
| Write path | refresh/flush/commit + translog + merge scheduler | **[ES]** |
| Serving | Shards, replicas, coordinating-node fan-out and merge | **[OS]**, cross-ref platform guide |

### 10.2 A contrast

The Lucene model is not the only implementation of the architecture. Columnar and specialised engines, and cloud-native retrieval services, realise the same tiers differently — some with different segment or partition models, some with managed inference for re-ranking. The architectural invariants are what matter: every implementation has an index with a *bet* about queries (positional or not), a *write model* with a delete-visibility cost, and a *fan-out* with a tail-latency floor. The two platform guides own the platform-specific configuration; this guide owns the invariants that survive the platform.

| Aspect | Where configured (sibling) | Where architected (here) |
|---|---|---|
| Sharding, replicas, node roles, heap, tiers, cost | [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md) | §6.2 (why fan-out exists) |
| Mappings, analysis plugins, reindex strategy | [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md) | §4.1 (analysis as a pipeline) |
| Vector/hybrid configuration | [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) | §3.6 (second index type) |

**References for §10.** Elasticsearch `index-modules-merge`, `doc_values` pages **[ES]**; OpenSearch node-role documentation (as cited in the platform guide) **[OS]**; Lucene 9.0.0 `org.apache.lucene.search` **[LUCENE]**; configuration is cross-referenced to [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md) and [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md) and not restated.

## 11. The Failure Modes

Six failure modes account for most retrieval incidents. Each is stated as symptom / cause / guardrail.

### 11.1 The stale index

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | A document that was just updated is not in results; users see the old version; "I updated it and it still doesn't show". |
| **Cause** | The write path is asynchronous: refresh makes documents visible only periodically (default every second) and only on indices that have been searched recently; a flush/commit is slower still; merges and translog behaviour add further delay (§7.3). |
| **Guardrail** | Make freshness an explicit SLO, not an implicit assumption. If a workflow truly needs read-after-write, use `refresh=wait_for` rather than `refresh=true` (which "creates less efficient index constructs (tiny segments)"), and budget the merge cost **[ES]** docs-refresh. |

### 11.2 The query the index cannot serve

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | A query type the business considers basic returns nothing or ranks poorly, and no amount of scoring tuning fixes it. |
| **Cause** | The index is a bet about query *shape*, and the bet was wrong: a non-positional index cannot do exact phrase queries (§3.3); a mapping that folded a field into tokens cannot do exact-value filtering; an index built on one document unit cannot answer a query about another. |
| **Guardrail** | Derive the index's structure from the *query shapes the business actually needs*, not from the corpus alone. The query set is the requirement; the index is the design. |

### 11.3 The relevance regression after an innocent change

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | After a mapping tweak, an analyser change, or an upgrade, ranking quality drops and nobody can say by how much. |
| **Cause** | Analysis runs at *both* index and query time (§4.1); changing one side without the other silently breaks term matching; scoring parameters (`k1`, `b`) and boosts interact; re-ranking weights drift. |
| **Guardrail** | Keep a frozen query set with judgments (§8.1) and run it in CI on every change to analysis, mapping or scoring. A change without a metric is a change without evidence. |

### 11.4 The skew

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | A few shards or nodes are hot; latency is spiky; one coordinator does the work of ten. |
| **Cause** | Shard-to-node allocation, uneven term distribution, routing that concentrates a tenant or a term on one shard, and coordination concentration. The capacity guide owns the sizing arithmetic. |
| **Guardrail** | Monitor per-shard load, not just cluster averages; the query waits for the **slowest** shard (§6.1), so the distribution of load matters more than the mean. |

### 11.5 The merge storm

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | Indexing throughput collapses, search latency spikes, disk fills — often after a bulk load. |
| **Cause** | Many small segments created by refresh or bulk writes trigger heavy background merging; merges are I/O-heavy and contend with search; the merge disk watermark (default `95%`) stops new merges, which then stalls indexing **[ES]** index-modules-merge. |
| **Guardrail** | Control segment production: batch ingest, avoid `refresh=true`, force-merge deliberately and off-peak, and watch the merge thread pool and disk watermark as first-class signals. |

### 11.6 The evaluation that measures the wrong thing

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | Metrics improve while users complain; the A/B test is flat while the offline benchmark is up; a "win" never reproduces. |
| **Cause** | Pooled judgments bias the offline measurement (§8.1); P@k is unstable and k-dependent; the offline query set does not resemble real queries; MRR ignores recall beyond the first hit; the relevance definition drifted from the user's. |
| **Guardrail** | Pair offline metrics with an online measure; state each metric's cutoff, normalisation and undefined cases (§8.2); refresh the judgment pool, because a stale pool is a biased one. |

**References for §11.** Elasticsearch `docs-refresh` and `index-modules-merge` **[ES]**; Manning, Raghavan & Schütze, *IR*, CUP 2008, ch. 8 **[IR-BOOK]**; sizing and skew arithmetic cross-referenced to [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md).

## 12. Entitlement and Deletion

This is where the guide earns its place, because both problems are under-owned everywhere and both are *architectural*, not administrative. The regulatory frame — which jurisdictions impose which obligations, and what the law requires — is owned by [data/data_compliance_frameworks.md](data/data_compliance_frameworks.md), [data/data_governance_framework.md](data/data_governance_framework.md) and [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md). **This section asserts no jurisdiction's rule; it works the retrieval-architecture mechanics those frames operate on.**

### 12.1 Entitlement enforcement inside retrieval

**An index that returns a document a user may not see is a disclosure.** The retrieval system is not a neutral library; it is a disclosure surface, and every result it returns is a statement that this user was allowed to see this document. The architecture question is *where the entitlement filter is applied*, because each placement has a different cost and a different failure mode.

| Placement | How it works | What it costs | Failure mode |
|---|---|---|---|
| **Query-time filter clause** | The query carries the user's entitlement attributes and constrains results, e.g. a Lucene `FILTER` clause — "a clause … required to occur in the result set but [that] should not contribute to the score" **[LUCENE]**. | Filter evaluation on every query; filter caches help only if the filter is stable (§6.3); poorly designed filters become the dominant latency. | A missing or wrong filter is an instant disclosure; the filter must be applied on *every* path, including suggestions, aggregations and (crucially) RAG context assembly. |
| **Index-time partitioning per entitlement** | Documents are partitioned into separate indices or shards by entitlement class, so a query only touches indices the user may see. | Index count and shard count multiply; cross-class queries fan out or are impossible; reclassification means moving documents between indices, i.e. re-indexing (§7). | Over-partitioning multiplies fan-out and cost; a mis-routed document is invisible to a user who *should* see it (a recall failure) rather than disclosed. |
| **Application / serving-layer filter** | The retrieval layer returns candidates and the application drops those the user may not see, before display. | Post-retrieval filtering wastes retrieval work; the top-k must be over-fetched because filtering removes items after ranking. | A bypass of the application layer (a direct index API, an export, a RAG pipeline) leaks the unfiltered set — the filter must be at the *source of disclosure*, not one caller. |

**What it costs in recall — the honest statement.** Entitlement filtering costs recall, and the cost is *architectural*, not incidental:

1. **Pre-filtering reduces the pool.** If entitlement is enforced inside the query, the matching set is a subset of the collection, so recall is computed *relative to the user*, not relative to the collection. A relevant document the user is not entitled to is, correctly, not retrieved — but any evaluation that measured recall against the whole collection will now show a "drop" that is not a quality failure, it is the design working (§8.1's pooling bias applies here in reverse).
2. **Post-filtering silently truncates the result list.** If candidates are retrieved and then filtered, the effective top-k shrinks per user: a user with few entitlements may receive far fewer than *k* results, and a user whose entitlements do not intersect the top-k receives nothing even though relevant, permitted documents exist deeper in the ranking. This is *recall loss disguised as relevance*.
3. **Partitioning trades recall across classes.** Index-per-entitlement makes within-class retrieval exact and cheap but makes cross-class retrieval either impossible or an expensive fan-out; a query that should span classes quietly returns only one class.

The architectural rule: **enforce entitlement at the single point where disclosure happens, make the filter's recall cost explicit, and measure recall per entitlement class, not against the whole collection.** The two failure directions are asymmetric — a false negative is a quality problem, a false positive is a disclosure — and the design must be biased toward the false negative while the evaluation is written to *see* it.

### 12.2 Deletion and retention in an index

A deletion request is an **architectural problem**, not a data-entry task, because the index is built to be hard to delete from.

**What deletion means for an immutable segment.** Because segments are immutable (§7.2), deleting a document does not remove it: it *marks* it deleted. Elasticsearch's own wording is exact — smaller segments are merged into larger ones "to keep the index size at bay and **to expunge deletes**" **[ES]** index-modules-merge. So the lifecycle of a delete is:

1. The delete is applied logically (a tombstone / soft delete); the document stops appearing in results but its bytes remain.
2. Space and data are reclaimed only when a **merge** rewrites the segment without the deleted document.
3. Until that merge, the document's *content* may persist on disk (and in snapshots and backups) even though search no longer returns it.

This is why "we deleted it" and "it is gone" are different statements in a log-structured index, and why a retention limit expressed in the business's language must be translated into *merge behaviour* to be true.

**What deletion means for a vector index.** An ANN structure (HNSW, IVF, or a quantised variant) is a *trained graph/cluster* structure, not a sorted list of documents. Removing a node from an HNSW graph disturbs the neighbour links that make the graph navigable; removing an entry from an IVF list disturbs the centroids. Consequently vector indexes **do not delete cleanly**: the common practice is to mark entries deleted and filter them at query time, and to achieve real removal only through a rebuild or a periodic re-merge of the structure. The vector-DB guide owns the mechanisms; the architectural point is that the *delete visibility* of a vector index is at least as bad as the inverted index's, and rebuilding the whole ANN structure to honour one erasure can be enormously more expensive than one segment merge. A system that stores the same document in both an inverted index and a vector index inherits **both** delete latencies and must reconcile them.

**Why it is architectural, not administrative.** A deletion request touches every representation of a document at once:

| Representation | Delete cost | True removal requires |
|---|---|---|
| Inverted index (segments) | Soft delete; search hides it immediately | Merge (or force-merge) to expunge |
| Doc values / forward index | Rebuilt with the segment | Merge; reindex if columnar-only |
| Vector index (ANN) | Mark-deleted and filtered | Rebuild / re-merge of the graph |
| Caches (§6.3) | Stale entries until invalidated | Cache invalidation / expiry |
| Snapshots / backups | Copies outside the live index | Backup retention policy (owned by compliance guides) |
| RAG context / generated text | Content may already be in a prompt/log | Governance of downstream stores |

The last two rows are the ones retrieval architects most often miss: the index is one of *several* copies of a document, and honouring erasure means the retrieval layer's delete must be *coordinated* with the layers that copied its output. The regulatory *frame* — which obligations apply, to which data, with which exceptions — is owned by the compliance, data-governance and banking-compliance guides; the retrieval architecture's job is to make deletion and retention *mechanically real* in the index and to surface the places where they are not.

### 12.3 The architectural stance

1. **Entitlement is a filter with a recall cost; state the cost.** Choose where the filter lives, accept the recall consequence, and measure per class.
2. **Deletion has a visibility lag; state the lag.** "Deleted" means "hidden now, expunged on merge"; for vector indexes it means "hidden now, maybe expunged on rebuild".
3. **Reconciliation is the hard part.** Inverted index, doc values, vector index, caches, snapshots and downstream copies must be deleted *together*; the retrieval layer owns only some of them.

| Problem | The bet | The cost when the bet is wrong |
|---|---|---|
| Entitlement at query time | Queries carry correct entitlement attributes | One unfiltered path is a disclosure |
| Entitlement by partitioning | Entitlement classes are stable | Reclassification = reindex |
| Delete via soft delete + merge | Merges run promptly | Erasure is late; data persists on disk |
| Delete in a vector index | Rebuild is affordable | Rebuild cost per erasure |

**References for §12.** Lucene `BooleanClause.Occur.FILTER` semantics **[LUCENE]**; Elasticsearch `index-modules-merge` ("expunge deletes") **[ES]**; vector-index deletion mechanics are owned by [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md); the regulatory frame is cross-referenced to [data/data_compliance_frameworks.md](data/data_compliance_frameworks.md), [data/data_governance_framework.md](data/data_governance_framework.md) and [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md) and is **not** asserted here.

## 13. The Cymbal Bank Worked Example

**This example is fictional and explicitly illustrative.** Cymbal Bank is a fictional institution; the figures below are worked illustrations, not observed results, and no number is a benchmark. Cymbal Bank is the only institution in this example.

### 13.1 The corpus and the requirement

Cymbal Bank builds a **document search** over a mixed corpus: policy documents (PDF, versioned), product documents (structured, marketing and terms), and customer correspondence (email and message exports, mixed formats and mixed classifications). Documents carry a **classification** (e.g. public / internal / confidential / restricted) and are associated with **entitlement attributes** (which teams or roles may see them). The requirement is ordinary and exacting: *users must find documents they are entitled to see, and must not be shown documents they are not.*

Two facts about this corpus decide the architecture, and both are bets:

1. **The corpus is mixed-format and mixed-classification**, so analysis must be robust across formats and entitlement must be enforced across all of them.
2. **Some queries are exact phrases** ("force majeure clause"), so the index must be **positional** (§3.3) — a bet that costs index size on every non-phrase query.

### 13.2 The entitlement filter, worked end to end

Cymbal chooses **query-time filtering with an entitlement filter clause**, and partitions only the most sensitive class.

- **Enrichment** attaches the classification and entitlement attributes to each document as fields.
- **Indexing** stores them as filterable fields (doc values for filtering/aggregation, §3.4).
- **Query** carries the user's entitlement attributes and adds a `FILTER` clause — a clause required to match but that "should not contribute to the score" **[LUCENE]** — so entitlement never inflates a relevance score.
- **Serving** returns only permitted documents, and **over-fetches** candidates so that after filtering, a user still receives up to *k* results.

**What the design gives up in recall — stated plainly.** With query-time filtering, Cymbal's per-user recall is measured against *the documents that user may see*, not against the whole collection. Concretely:

- A user with **few entitlements** receives fewer results than a user with many, even for the same query. This is not a ranking failure; it is the design working. But an evaluation that measured recall against the whole collection would report a *drop* for the low-entitlement user that is pure artefact — so Cymbal evaluates **recall per entitlement class**.
- If Cymbal instead filtered *after* retrieval (application-layer), the top-k would silently truncate per user, and a user whose permitted documents sit below rank *k* would receive nothing despite relevant, permitted documents existing. Cymbal rejects post-filtering for exactly this reason.
- For the **restricted** class, Cymbal partitions into a separate index, accepting that cross-class queries must fan out to both, and accepting that a query that should span classes will not unless it is written to.

### 13.3 A deletion request, worked end to end

A data-subject erasure request arrives for a set of correspondence documents. The retrieval architecture must make deletion *mechanically real*, not merely nominal.

| Step | Action | Architectural consequence |
|---|---|---|
| 1. Identify | Every document ID for the subject across the corpus is resolved. | Identity must be stable across formats (§2). |
| 2. Delete in the inverted index | Issue deletes; documents stop appearing in search. | Soft delete: bytes remain until a merge "expunges" them **[ES]**. |
| 3. Force merge | A merge (or force-merge) rewrites segments without the deleted documents. | Merge is I/O-heavy; scheduled off-peak, not on the hot path (§7.2). |
| 4. Delete in the vector index | Mark-deleted and filtered; schedule a graph rebuild. | ANN structures "do not delete cleanly"; erasure latency is the rebuild time (§12.2). |
| 5. Invalidate caches | Filter cache, request cache and any cached context are invalidated. | A stale cache is a disclosure (§6.3). |
| 6. Handle snapshots and downstream copies | Coordinate with backup retention and any RAG store that copied the content. | The index is one of several copies; erasure spans them (§12.2); the retention policy is owned by the compliance guides. |

**The architectural truth this exposes:** the deletion request was not a data-entry task. It required a *merge* in the inverted index, a *rebuild* in the vector index, *cache invalidation* across layers, and *coordination* with stores the retrieval layer does not own — and each of those is a property of the index's structure chosen in §3, not of the deletion request.

### 13.4 What Cymbal bought, and what it paid

Cymbal's index is a bet about the queries it expects: exact phrases (so positional), robust across formats (so careful analysis), and filtered by entitlement (so filterable fields and query-time filters). What Cymbal paid: index size for positions, per-query filter cost, per-class recall accounting, and a deletion path that is as slow as its merges and its graph rebuilds. The design is coherent only because Cymbal measures each of those costs explicitly rather than discovering them in an incident.

**The thesis, one last time: an index is a bet about the queries you will be asked.**

| Cymbal decision | The bet | Recall / cost consequence |
|---|---|---|
| Positional index | Queries include phrases | Larger index; slower writes |
| Query-time entitlement filter | Entitlement attributes are query-available | Per-class recall; filter latency |
| Partition the restricted class | Restricted data rarely co-queried | Cross-class fan-out |
| Delete via merge + vector rebuild | Erasure can wait for compaction | Erasure lag = merge/rebuild time |

**References for §13.** Illustrative only; the mechanisms cited are Lucene `FILTER` semantics **[LUCENE]** and Elasticsearch `index-modules-merge` **[ES]**; vector deletion mechanics are owned by [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md); the retention/erasure frame is owned by the compliance, data-governance and banking-compliance guides.

## 14. The Anti-Patterns

Each anti-pattern is stated as symptom / cause / guardrail.

### 14.1 Index for the corpus, not the queries

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | A technically excellent index that the business finds useless. |
| **Cause** | The index was designed from the documents (what is in them) rather than from the queries (what will be asked of them). |
| **Guardrail** | Start from the query set. Every structural choice — positional or not, which fields are filterable, which are doc-values — is a statement about queries. |

### 14.2 Filter entitlement in the application, everywhere, inconsistently

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | A disclosure incident traced to "an export path that didn't apply the filter", or a suggestion API that leaked a title. |
| **Cause** | Entitlement applied at the display layer rather than at the disclosure surface; each caller reimplements it. |
| **Guardrail** | Enforce at the single point where disclosure happens; make unfiltered access structurally impossible rather than conventionally avoided (§12.1). |

### 14.3 Treat deletion as a data-entry task

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | "The record was deleted" — but a search, a snapshot, or a cached context still returns it. |
| **Cause** | Soft deletes are hidden but not expunged until a merge; vector structures and caches and backups hold further copies (§12.2). |
| **Guardrail** | Model deletion as an architectural operation with a visibility lag across every copy; test it end to end. |

### 14.4 Measure with a stale pool

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | Offline metrics are green; users are unhappy; the pool predates the current corpus. |
| **Cause** | The judgment pool inherits the bias of the systems that produced it (§8.1); as the corpus and the queries change, the pool drifts. |
| **Guardrail** | Refresh the pool; pair offline metrics with an online measure; state each metric's cutoff and undefined cases. |

### 14.5 Let merges run on the hot path

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | Latency collapses during bulk ingest or a force-merge. |
| **Cause** | Merges contend with search for I/O and CPU; refresh-heavy writes create tiny segments that must be merged (§7.2, §11.5). |
| **Guardrail** | Batch ingest; avoid `refresh=true`; schedule force-merges off-peak; alert on merge thread-pool and disk-watermark signals. |

### 14.6 Rebuild the whole index to change the bet

| Symptom / cause / guardrail | Detail |
|---|---|
| **Symptom** | A "small" requirement (phrase search, a new filter, per-tenant isolation) turns into a full reindex. |
| **Cause** | The bet was structural (§3) and was decided without knowing it was structural. |
| **Guardrail** | Identify the irreversible choices before indexing: positional or not, entitlement placement, document unit, field filterability. These are the ones to get right first. |

| Anti-pattern | Root cause | Guardrail |
|---|---|---|
| Index for the corpus | Wrong requirement source | Design from the query set |
| Inconsistent entitlement filtering | Filter at display, not disclosure | Single enforcement point |
| Deletion as data entry | Ignores segment/ANN/cache copies | Architectural delete with visibility lag |
| Stale evaluation pool | Inherited pool bias | Refresh pool; add online metric |
| Merges on the hot path | Refresh/bulk segment churn | Batch and schedule merges |
| Full reindex for a small change | Structural bet made blindly | Name the irreversible choices first |

**References for §14.** Synthesis of §3, §6, §7, §8, §12; Elasticsearch `index-modules-merge` and `docs-refresh` **[ES]**.

## 15. The Claims Audit

Every high-risk claim in this guide, sorted into **Verified**, **Flagged** ⚠, and **Rejected**. Dates are the retrieval/publication dates of the sources in this work (October 2026 research pass; the IR textbook is the 2008 edition). "Kind" is primary paper / textbook / vendor doc / secondary.

### 15.1 The index and its data structures (§3)

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| Inverted index maps term → documents; built from the term-document incidence matrix | **Verified** | Manning, Raghavan & Schütze, *Introduction to Information Retrieval*, ch. 1 | 2008 | Textbook **[IR-BOOK]** |
| A *posting* is a docID in a postings list | **Verified** | *IR*, ch. 5 | 2008 | Textbook **[IR-BOOK]** |
| Positional indexes support phrase queries and are substantially larger than non-positional | **Verified** | *IR*, ch. 2 | 2008 | Textbook **[IR-BOOK]** |
| Skip pointers speed postings-list intersection | **Verified** | *IR*, ch. 2 | 2008 | Textbook **[IR-BOOK]** |
| Variable-byte codes use a continuation bit; "excellent tradeoff between time and space" | **Verified** | *IR*, ch. 5 (Variable byte codes) | 2008 | Textbook **[IR-BOOK]** |
| Elias gamma codes split a gap into unary length + binary offset; "universal" | **Verified** | *IR*, ch. 5 (Gamma codes) | 2008 | Textbook **[IR-BOOK]** |
| Gamma/delta codeword sets originate with P. Elias | **Verified** | Elias, "Universal codeword sets and representations of the integers", *IEEE Trans. Inf. Theory*, 1975 (DOI 10.1109/TIT.1975.1055349) | 1975 | Primary paper **[PAPER]** |
| PForDelta-style compression originates in Zukowski et al., "Super-Scalar RAM-CPU Cache Compression" | **Verified** | Zukowski, Heman, Nes & Boncz, *ICDE 2006*, p. 59 | 2006 | Primary paper **[PAPER]** |
| Doc values are "an on-disk data structure … built at document index time", columnar, for sorting/aggregation | **Verified** | Elasticsearch `doc_values` page | Oct 2026 | Vendor doc **[ES]** |
| Simple-9 is attributed to Anh & Moffat, *Information Retrieval* journal | **Flagged** ⚠ | not verified at primary this pass | — | — |

### 15.2 The query pipeline and ranking (§4–§5)

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| Two execution strategies are TAAT and DAAT; DAAT usually preferred for conjunctive queries | **Verified** | Ding & Suel, *SIGIR 2011*, citing Turtle & Flood 1995 | 2011 / 1995 | Primary paper **[PAPER]** |
| Turtle & Flood, "Query evaluation: strategies and optimizations", *IP&M* 31(6):831–850, defines the TAAT/DAAT strategies | **Verified** | ERIC record EJ516508 (abstract) and Ding & Suel reference [32] | 1995 | Primary paper **[PAPER]** |
| WAND originates in Broder et al., "Efficient query evaluation using a two-level retrieval process", *CIKM 2003*, pp. 426–434 | **Verified** | Google Research and IBM publication records; named "WAND" in Ding & Suel 2011 | 2003 | Primary paper **[PAPER]** |
| Block-max WAND originates in Ding & Suel, "Faster top-k document retrieval using block-max indexes", *SIGIR 2011* | **Verified** | Author PDF (NYU); ACM DOI 10.1145/2009916.2010048 | 2011 | Primary paper **[PAPER]** |
| "MaxScore" is the name of a pruning method credited to Turtle & Flood 1995 | **Flagged** ⚠ | not verified at primary this pass | — | — |
| Porter stemming algorithm: Porter, 1980 | **Verified** | *IR*, ch. 2 (citing Porter 1980) | 2008 / 1980 | Textbook / primary **[IR-BOOK]** |
| Lovins stemmer (1968) and Paice/Husk stemmer (1990) | **Verified** | *IR*, ch. 2 | 2008 | Textbook **[IR-BOOK]** |
| Snowball is a stemming language designed by Martin Porter | **Verified** | snowballstem.org landing page | Oct 2026 | Vendor/community doc **[WEB]** |
| Krovetz stemmer (1993) | **Flagged** ⚠ | not verified at primary this pass | — | — |
| Rocchio relevance feedback algorithm: Rocchio (1971), as "the classic algorithm" | **Verified** (attribution to the 1971 reference as stated by the textbook) | *IR*, ch. 9 (The Rocchio algorithm), citing Rocchio 1971 | 2008 / 1971 | Textbook **[IR-BOOK]** |
| TF-IDF `= tf × idf`; vector-space model | **Verified** | *IR*, ch. 6 | 2008 | Textbook **[IR-BOOK]** |
| Okapi BM25 originates in Robertson, Walker, Jones, Hancock-Beaulieu & Gatford, "Okapi at TREC-3", TREC-3, 1994 | **Verified** | Lucene `BM25Similarity` Javadoc (quotes the full citation) | 1994 | Primary paper via **[LUCENE]** |
| Lucene BM25 defaults `k1=1.2`, `b=0.75`; idf = `log(1 + (docCount − docFreq + 0.5)/(docFreq + 0.5))` | **Verified** | Lucene 9.0.0 `BM25Similarity` | 2021 | Vendor doc **[LUCENE]** |
| RankNet: Burges et al., "Learning to Rank using Gradient Descent", *ICML 2005* | **Verified** | ICML 2005 proceedings PDF; ACM 10.1145/1102351.1102363 | 2005 | Primary paper **[PAPER]** |
| LambdaMART: Burges, "From RankNet to LambdaRank to LambdaMART: An Overview", MSR-TR-2010-82 | **Verified** | Microsoft Research technical report | 2010 | Primary paper **[PAPER]** |
| RankSVM / clickthrough optimisation: Joachims, "Optimizing search engines using clickthrough data", *KDD 2002* | **Verified** | ACM 10.1145/775047.775067; author PDF (Cornell) | 2002 | Primary paper **[PAPER]** |
| ListNet: Cao, Qin, Liu, Tsai & Li, "Learning to Rank: From Pairwise Approach to Listwise Approach", *ICML 2007* | **Verified** | ACM 10.1145/1273496.1273513; MSR PDF | 2007 | Primary paper **[PAPER]** |
| RRF: Cormack, Clarke & Büttcher, *SIGIR 2009*, pp. 758–759; `RRFscore(d)=Σ 1/(k+r(d))`, `k=60` | **Verified** | Author PDF; Google Research record | 2009 | Primary paper **[PAPER]** |
| Lucene search iterates document-at-a-time via `DocIdSetIterator.nextDoc()`; `BlockMaxDISI`/`ImpactsDISI` skip non-competitive docs | **Verified** | Lucene 9.0.0 `org.apache.lucene.search` package summary | 2021 | Vendor doc **[LUCENE]** |

### 15.3 The write path, serving and platform (§6–§7)

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| "A shard in Elasticsearch is a Lucene index, and a Lucene index is broken down into segments … immutable. Smaller segments are periodically merged … to expunge deletes." | **Verified** (quotation) | Elasticsearch `index-modules-merge` | Oct 2026 | Vendor doc **[ES]** |
| Merge scheduler prioritises smaller merges, is disk-I/O throttled, and stops scheduling when disk is low; watermark default `95%` | **Verified** | Elasticsearch `index-modules-merge` | Oct 2026 | Vendor doc **[ES]** |
| Refresh writes the in-memory buffer to a new searchable segment; document searchable "within 1 second"; default `index.refresh_interval` = `1s` | **Verified** | Elasticsearch `near-real-time` and `docs-refresh` | Oct 2026 | Vendor doc **[ES]** |
| `refresh=true` "creates less efficient index constructs (tiny segments)"; `refresh=wait_for` waits for a scheduled refresh | **Verified** | Elasticsearch `docs-refresh` | Oct 2026 | Vendor doc **[ES]** |
| Flush = Lucene commit + new translog generation; translog holds acknowledged-but-uncommitted ops; default `durability=request` | **Verified** | Elasticsearch `index-modules-translog` | Oct 2026 | Vendor doc **[ES]** |
| Lucene `FILTER` clause is required to match but "should not contribute to the score" | **Verified** | Lucene 9.0.0 `org.apache.lucene.search` (BooleanClause.Occur) | 2021 | Vendor doc **[LUCENE]** |
| LRU query cache and usage-tracking caching policy exist | **Verified** | Lucene 9.0.0 `org.apache.lucene.search` (class summary) | 2021 | Vendor doc **[LUCENE]** |

### 15.4 Evaluation (§8)

| Claim | Verdict | Source | Date | Kind |
|---|---|---|---|---|
| Precision `= tp/(tp+fp)`; recall `= tp/(tp+fn)` | **Verified** | *IR*, ch. 8 | 2008 | Textbook **[IR-BOOK]** |
| F-measure is the weighted harmonic mean; `F₁ = 2PR/(P+R)` | **Verified** | *IR*, ch. 8 | 2008 | Textbook **[IR-BOOK]** |
| Interpolated precision `p_interp(r) = max_{r' ≥ r} p(r')`; 11-point measure used in early TREC | **Verified** | *IR*, ch. 8 | 2008 | Textbook **[IR-BOOK]** |
| MAP formula `MAP(Q) = (1/|Q|)·Σ_j (1/m_j)·Σ Precision(R_jk)`; unretrieved relevant docs count as precision 0 | **Verified** | *IR*, ch. 8 | 2008 | Textbook **[IR-BOOK]** |
| Pooling: judgments taken from the top-k of the systems under test | **Verified** | *IR*, ch. 8 (Assessing relevance) | 2008 | Textbook **[IR-BOOK]** |
| Kappa statistic formula; fair agreement 0.67–0.8, good > 0.8 | **Verified** | *IR*, ch. 8 | 2008 | Textbook **[IR-BOOK]** |
| DCG/nDCG: Järvelin & Kekäläinen, *ACM TOIS* 20(4):422–446, 2002; DCG discount by `log_b(rank)`; nDCG = DCG/ideal DCG | **Verified** | ACM 10.1145/582415.582418; author PDF | 2002 | Primary paper **[PAPER]** |
| MRR introduced in the TREC-8 Question Answering track | **Verified** (track/paper); ⚠ the exact first use of the term "MRR" is not quoted | Voorhees, "The TREC-8 Question Answering Track Report", NIST SP 500-246 | 1999/2000 | Primary report **[PAPER]** |

### 15.5 Rejected claims — things this guide refuses to assert

| Rejected claim | Why it is rejected |
|---|---|
| "An index is a copy of the collection." | An index is a *reorganisation* of the collection around expected queries; that is the thesis. |
| Any specific recall/precision/latency figure for a real system. | No benchmark number was traced to a dated source in this pass; the guide prints none. |
| "Positional indexes are free." | They are substantially larger than non-positional indexes; the trade-off is the point (**[IR-BOOK]** ch. 2). |
| "Deleted means gone." | In a segment-based index, deletion is logical until a merge expunges it; in a vector index it persists until a rebuild (**[ES]**, §12.2). |
| "RAG replaced the inverted index." | RAG added a second index type and a re-ranking stage; the index, write path, evaluation and entitlement problems remain (§9). |
| "Entitlement filtering is free." | It costs recall, and the cost is architectural, not incidental (§12.1). |
| "Map the IDF variant of your choice without saying which." | Lucene uses a specific smoothed IDF; the textbook's alternative can go negative (**[LUCENE]**, **[IR-BOOK]** ch. 11). |

**References for §15.** As listed per row: **[IR-BOOK]** = Manning, Raghavan & Schütze, *Introduction to Information Retrieval*, CUP 2008; **[PAPER]** = the named primary paper/report; **[LUCENE]** = Apache Lucene 9.0.0 documentation; **[ES]** = Elasticsearch reference documentation (retrieved October 2026); **[WEB]** = the named web page.

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What could not be verified in this pass

**A tool note, recorded honestly.** `web_search` returned **empty result sets for several queries** during this work — repeatedly, for queries about Simple-9, the Krovetz stemmer, interleaving and the "MaxScore" name — while identical-class queries about other papers returned results. This is recorded as a **tool limitation, not evidence of absence**. All other claims were verified by direct extraction of primary URLs (the IR textbook's free HTML edition, Lucene and Elasticsearch documentation, and author/publisher PDFs of the named papers) or via `web_search` when it did return results.

| Item | Status | What would resolve it |
|---|---|---|
| **Simple-9** compression code attribution (Anh & Moffat, *Information Retrieval* journal) | ⚠ not verified | A direct extraction of the Anh & Moffat 2005 paper, or a citable index |
| **Krovetz stemmer** (1993) attribution | ⚠ not verified | The Krovetz 1993 SIGIR paper |
| **"MaxScore"** as the canonical name for Turtle & Flood's maximum-score pruning | ⚠ not verified | A primary source that names the algorithm "MaxScore" |
| **Interleaving** canonical attribution (e.g. Radlinski et al. 2008) | ⚠ not verified | The interleaving paper; Joachims *KDD 2002* is verified only as the clickthrough anchor |
| **Elias delta (δ)** code as distinct from gamma in the Elias 1975 paper | ⚠ partially verified | The Elias 1975 full text (the abstract and the IR book's exercise are what were reached) |
| **Robertson & Zaragoza 2009** ("The Probabilistic Relevance Framework: BM25 and Beyond") as the BM25 framework paper | ⚠ not re-fetched | A direct extraction of that survey |
| Specific **first-use of "MRR"** wording | ⚠ not quoted | The TREC-8 QA track report full text |
| Any **benchmark number** for BM25/BM25+ variants, or for block-max WAND speedups | Deliberately excluded | Would require a dated, scoped source; none is asserted |

The thesis does not depend on any flagged item. The index-as-bet argument rests on the inverted-index structure, the segment model and the entitlement/deletion mechanics, all of which are verified above.

### 16.2 Glossary

| Term | Definition as used here |
|---|---|
| **Analysis** | The chain (tokenise → normalise → stop-word → stem/lemmatise) applied to both documents and queries; asymmetric application breaks matching **[IR-BOOK]** ch. 2. |
| **Block-max index** | An index augmentation storing the maximum impact score per compressed block of a postings list, enabling whole-block skipping in WAND **[PAPER]** Ding & Suel 2011. |
| **BM25 / Okapi weighting** | A probabilistic scoring function sensitive to term frequency and document length, with parameters `k1` and `b` **[IR-BOOK]** ch. 11; **[PAPER]** Robertson et al., TREC-3, 1994. |
| **DAAT / TAAT** | Document-At-A-Time and Term-At-A-Time — the two strategies for traversing the index during query evaluation **[PAPER]** Turtle & Flood 1995. |
| **DCG / nDCG** | Discounted Cumulative Gain and its normalisation by the ideal ordering, for graded relevance **[PAPER]** Järvelin & Kekäläinen 2002. |
| **Doc values** | The forward-index structure: an on-disk columnar structure built at index time for sorting and aggregation **[ES]**. |
| **Early termination** | Query optimisation that avoids scoring documents that cannot enter the top-k **[PAPER]** Ding & Suel 2011. |
| **Forward index** | Document → its terms/values; the mirror of the inverted index. |
| **Inverted index** | Term → postings list (the documents containing it); the core retrieval structure **[IR-BOOK]** ch. 1. |
| **MAP** | Mean Average Precision — the mean over queries of average precision **[IR-BOOK]** ch. 8. |
| **Merge** | The background process that combines smaller immutable segments into larger ones "to keep the index size at bay and to expunge deletes" **[ES]**. |
| **MRR** | Mean Reciprocal Rank — the mean of `1/rank` of the first correct answer **[PAPER]** TREC-8 QA. |
| **PForDelta** | A word-aligned postings compression code optimised for fast decoding **[PAPER]** Zukowski et al. 2006. |
| **Postings list** | The list of documents (each a posting) that contain a term **[IR-BOOK]** ch. 1, ch. 5. |
| **Precision / recall** | Fraction of retrieved documents that are relevant / fraction of relevant documents retrieved **[IR-BOOK]** ch. 8. |
| **Refresh** | Making buffered writes visible to search by writing and opening a new segment; near-real-time, not a durable commit **[ES]**. |
| **Re-ranking** | A second-stage model (often a cross-encoder) scoring the top-k of a first-stage retriever. |
| **RRF** | Reciprocal Rank Fusion — combining rankings by rank, `Σ 1/(k+r(d))`, `k=60` **[PAPER]** Cormack et al. 2009. |
| **Segment** | The immutable unit a shard is built from; created by refresh and combined by merges **[ES]**. |
| **Term dictionary** | The vocabulary of indexed terms with pointers to their postings lists **[IR-BOOK]** ch. 3. |
| **Translog** | The per-shard transaction log holding acknowledged-but-uncommitted operations, replayed on recovery **[ES]**. |
| **WAND** | A two-level query-evaluation process that prunes candidate documents that cannot reach the top-k **[PAPER]** Broder et al. 2003. |

### 16.3 Cross-references

**Cited, never re-derived:**

- [opensearch_capacity_planning_guide.md](opensearch_capacity_planning_guide.md) — owns search-platform operations: fork history, sizing method, shard strategy, node roles, heap/JVM, storage tiers, ingest capacity, scaling, cost, monitoring. Cited in §6 and §11 for the fan-out/tail-latency and skew arithmetic; **not restated**.
- [data/elasticsearch_data_modeling_schema_design.md](data/elasticsearch_data_modeling_schema_design.md) — owns mapping, text analysis, indexing strategy, aggregations-friendly schema and reindexing for one platform. Cited in §4 and §10 for analysis-as-configured; **not restated**.
- [ai_llm/rag/vector_databases_guide.md](ai_llm/rag/vector_databases_guide.md) — owns vector/ANN retrieval: kNN, HNSW, IVF, product quantisation, distance metrics, filtering, hybrid search and the vendor landscape. The vector index is a **second index type** here (§3.6, §12.2); its mechanisms are **not restated**.
- [ai_llm/rag/bm25_faiss_scann_research.md](ai_llm/rag/bm25_faiss_scann_research.md) — owns the BM25 and FAISS/SCANN research notes. Cross-referenced in §5.
- [ai_llm/agentic_search_vs_rag_guide.md](ai_llm/agentic_search_vs_rag_guide.md) — owns the search-versus-RAG architectural comparison. Cross-referenced in §9.
- [data/data_compliance_frameworks.md](data/data_compliance_frameworks.md), [data/data_governance_framework.md](data/data_governance_framework.md) and [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md) — own the regulatory and data-governance frame for §12. Cross-referenced for the legal frame; **this guide asserts no jurisdiction's rule**.
- The repo's system-design-interview guides — own the inverted index as an interview topic; this guide owns it as architecture.

**Primary sources, retrieved October 2026:** Manning, Raghavan & Schütze, *Introduction to Information Retrieval*, CUP 2008 (free HTML edition, nlp.stanford.edu/IR-book: boolean-retrieval; the-term-vocabulary-and-postings-lists; index-construction; index-compression; scoring-term-weighting-and-the-vector-space-model; probabilistic-information-retrieval; evaluation-in-information-retrieval); Apache Lucene 9.0.0 `org.apache.lucene.search` and `similarities.BM25Similarity` documentation; Elasticsearch reference documentation (`index-modules-merge`, `docs-refresh`, `near-real-time`, `index-modules-translog`, `doc_values`); Elias, *IEEE Trans. Inf. Theory*, 1975; Zukowski, Heman, Nes & Boncz, *ICDE 2006*; Broder, Carmel, Herscovici, Soffer & Zien, *CIKM 2003*; Ding & Suel, *SIGIR 2011*; Turtle & Flood, *IP&M* 31(6):831–850, 1995; Robertson, Walker, Jones, Hancock-Beaulieu & Gatford, *TREC-3*, 1994; Burges et al., *ICML 2005*; Burges, MSR-TR-2010-82, 2010; Joachims, *KDD 2002*; Cao, Qin, Liu, Tsai & Li, *ICML 2007*; Cormack, Clarke & Büttcher, *SIGIR 2009*; Järvelin & Kekäläinen, *ACM TOIS* 20(4):422–446, 2002; Voorhees, *TREC-8* QA track report, 1999; snowballstem.org.

### 16.4 Closing summary

**The thesis, restated because everything above is commentary on it:** an index is a bet about the queries you will be asked. The inverted index is a reorganisation of the collection around a guess — which terms matter, whether order matters, whether a user's query is keywords or a paragraph, whether the answer must be filtered to what the user may see. That guess is expressed in *structures*: positional or non-positional, a term dictionary of a particular shape, an entitlement filter at one of three places, a segment model that hides deletes until a merge expunges them, a score that is only a proxy for relevance.

**The conclusions to carry into a design review:**

1. **Design from the query set, not the corpus.** The index is the answer to a question about queries; state the question first.
2. **Name the irreversible choices early.** Positional or not, entitlement placement, document unit and field filterability are the ones a full reindex cannot cheaply undo.
3. **Analysis is a contract between index time and query time.** Change one side only and matching breaks silently.
4. **Early termination is a bet that the top-k is small.** TAAT/DAAT, WAND and block-max WAND are the names, and they assume you rarely need the whole matched set.
5. **The write path is log-structured in disguise.** Immutable segments, merges, refresh/flush/commit and the translog are one design, and each has a latency cost.
6. **Evaluation is only as good as its pool.** Pooled judgments inherit the bias of the systems that produced them; state each metric's cutoff, normalisation and undefined cases.
7. **RAG added an index type and a stage; it did not remove the architecture.** The write path, the evaluation problem and the entitlement problem are unchanged.
8. **Entitlement is a filter with a recall cost — make it explicit and measure per class.**
9. **Deletion is architectural, not administrative.** "Hidden now, expunged on merge" is the real semantics of an index delete, and a vector index is worse.

**The final word:** the index is a decision made before the first real query arrives, and it is paid for every time a query is answered. Choose it from the queries you expect, keep the evidence that you chose well, and respect the parts you cannot change without rebuilding.

**an index is a bet about the queries you will be asked.**
