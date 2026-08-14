# Vector Databases: Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG Optimization Techniques](rag_optimization_techniques_guide.md) · [Beyond RAG — Agent Memory](beyond_rag_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [BM25 / FAISS / ScaNN Research](bm25_faiss_scann_research.md) · [Agent Runtime Cache Design](agent_runtime_cache_design_guide.md) · [On-Prem LLM Deployment](../on_prem_llm_deployment_guide.md) · [LLM Development Risks & Security](../llm_development_risks_security_guide.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Introduction: What Vector Databases Are](#1-introduction-what-vector-databases-are)
2. [The Core Operation: kNN and ANN Search](#2-the-core-operation-knn-and-ann-search)
3. [Use Cases — Why Vector Search Matters](#3-use-cases--why-vector-search-matters)
4. [The Rise with LLMs: Boom, Hype, and Consolidation](#4-the-rise-with-llms-boom-hype-and-consolidation)
5. [How Vector Search Works: Embeddings](#5-how-vector-search-works-embeddings)
6. [ANN Index Algorithms: HNSW, IVF, PQ, and Quantization](#6-ann-index-algorithms-hnsw-ivf-pq-and-quantization)
7. [Distance Metrics: Cosine, L2, Dot Product](#7-distance-metrics-cosine-l2-dot-product)
8. [Filtering and Hybrid Search](#8-filtering-and-hybrid-search)
9. [Vector Database Internals](#9-vector-database-internals)
10. [The Technology Landscape: Four Categories](#10-the-technology-landscape-four-categories)
11. [Dedicated Vector Databases — Vendor Profiles](#11-dedicated-vector-databases--vendor-profiles)
12. [Vector Search Libraries, Relational Extensions, and Managed Services](#12-vector-search-libraries-relational-extensions-and-managed-services)
13. [The Landscape Comparison Table](#13-the-landscape-comparison-table)
14. [Selection Framework: Choosing the Right Tool](#14-selection-framework-choosing-the-right-tool)
15. ["Do You Even Need a Vector Database?"](#15-do-you-even-need-a-vector-database)
16. [Operations: Deploying and Running Vector Databases](#16-operations-deploying-and-running-vector-databases)
17. [The Vector Database in a Banking Context](#17-the-vector-database-in-a-banking-context)
18. [Evaluation and Benchmarking](#18-evaluation-and-benchmarking)
19. [RAG Integration Recap](#19-rag-integration-recap)
20. [The Future of Vector Search (2026 and Beyond)](#20-the-future-of-vector-search-2026-and-beyond)
21. [Glossary](#21-glossary)

---

## 1. Introduction: What Vector Databases Are

**A vector database is a database purpose-built to store and search high-dimensional vectors — embeddings — by similarity.** Where a relational database is organized around rows, keys, and exact-value predicates, a vector database is organized around *positions in high-dimensional space*: every record is a point (a vector of floats), and the fundamental query is "give me the points nearest to this one."

The difference from a conventional database is not cosmetic. Traditional databases are built for **exact matches** (SQL `WHERE`, key lookups, range scans); vector databases are built for **proximity**. That single shift changes the storage format, the index structures, the query planner, the write path, and the failure modes — which is why the category exists at all rather than being a feature bolted onto SQL.

### 1.1 What an embedding actually is

An **embedding** is the output of a machine-learning model that maps an object — text, image, audio, user, transaction — into a fixed-length vector of real numbers, typically 384 to 3072 dimensions. The defining property: **objects that are semantically similar map to vectors that are close together** (small distance), while dissimilar objects land far apart. Cosine similarity between two vectors is the de facto semantic-proximity measure: two sentences that mean the same thing in different words get high cosine similarity, even with zero lexical overlap.

This is the entire premise of semantic search: instead of matching keywords, you match *meaning*. The database's job is to make "find the k vectors nearest to this query vector" fast at scale — a problem that gets hard surprisingly quickly (see §2).

### 1.2 Vector database vs. everything else in the neighborhood

The term "vector database" is overloaded. Three related things are often conflated:

| Thing | What it is | Examples |
|---|---|---|
| **Vector database** | A full database: storage, indexing, query engine, transactions/durability, API — with similarity search as a first-class operation | Qdrant, Milvus, Weaviate, Pinecone, Chroma, Vespa |
| **Vector search library** | An embedded library that indexes vectors in-process and answers kNN queries; no server, no durability, no metadata management | FAISS, ScaNN, Annoy, HNSWlib, USearch |
| **Vector extension / feature** | Vector support added to an existing database (relational, search, cache, warehouse) | pgvector (Postgres), SQLite-vec, Elasticsearch kNN, Redis, MongoDB Atlas Vector Search, Oracle 23ai, Snowflake |

The library-vs-database distinction matters most in practice: FAISS is not a database — it is a blazingly fast in-memory index you embed in your application process, and you own persistence, concurrency, and ops around it. The extension category is why many teams conclude they never need a dedicated product at all (see §15 — the honest take).

### 1.3 The RAG connection (brief)

Vector databases became mainstream because of retrieval-augmented generation: chunk documents, embed the chunks, index the vectors, and at query time retrieve the most relevant chunks to stuff into the LLM prompt. The vector database is one component of that pipeline — the retrieval store — and the full pipeline (chunking, embedding, retrieval, reranking, fusion) is covered in the [RAG Optimization Techniques guide](rag_optimization_techniques_guide.md) and the [RAG Frameworks Comparison guide](rag_frameworks_comparison_guide.md) (§8). This guide is the deep dive on the database technology itself: what happens inside the index, how the products differ, how to pick one, and how to run it.

---

## 2. The Core Operation: kNN and ANN Search

### 2.1 kNN — the exact answer, and why it does not scale

The core operation of a vector database: given a query vector **q**, return the **k nearest neighbors** by a distance metric — the k vectors in the collection with the smallest distance to q.

The naive way is a **brute-force linear scan**: compute the distance from q to every stored vector, keep the k smallest. This is exact, simple, trivially parallelizable, and surprisingly viable for small collections (tens of thousands of vectors, or a few hundred thousand with SIMD/GPU acceleration). But it is **O(N × D)** per query — for 100 million 768-dimensional vectors, that is ~77 billion multiply-adds per query. At any serious scale, exact kNN is too slow for interactive latency, so real systems make a trade:

### 2.2 ANN — approximate nearest neighbor

**Approximate Nearest Neighbor (ANN) search** trades a small, bounded loss of recall for orders-of-magnitude speedup. Instead of scanning everything, an ANN index organizes vectors so that only a small, promising subset is examined. With a well-tuned index you can get 95–99% of the exact-top-k results at 10–100× lower latency and cost.

The trade is quantified as **recall@k**: the fraction of the *true* top-k (from brute force) that the ANN index actually returns. Recall is a tunable dial — most indexes expose parameters (efSearch, nprobe, etc.) that slide along a latency–recall curve. The art of operating a vector database is finding the operating point on that curve that meets your application's quality bar at your QPS target.

**Why ANN is acceptable:** for retrieval in RAG, a 97% recall index that returns results in 10 ms beats a 100% recall scan that takes 200 ms — the LLM downstream dominates quality anyway, and retrieval quality is evaluated end-to-end (see §18). For applications that genuinely need exact results (some compliance searches), keep brute force with GPU acceleration as an option; FAISS offers exact search that still exploits SIMD.

### 2.3 The ANN taxonomy: four index families

Every ANN algorithm belongs to one of four families, and knowing the family explains the trade-offs (§6 goes algorithm-by-algorithm):

| Family | Idea | Representative | Strength | Weakness |
|---|---|---|---|---|
| **Graph-based** | Nodes = vectors, edges = proximity; greedy walk from entry point | HNSW, NGT, DiskANN | Best quality/speed; incremental inserts | Memory-heavy; graph maintenance on deletes |
| **Cluster/partition-based** | Quantize space into regions; search promising regions only | IVF, ScaNN's tree, FAISS IVF | Disk-friendly; simple; fast builds | Recall ceiling set by region choice (nprobe) |
| **Quantization/hash-based** | Compress or hash vectors so distances are computed on cheap codes | PQ, LSH, binary codes | Tiny memory; very fast distance | Lossy; needs rescoring for quality |
| **Tree-based** | Recursively split space (kd-tree / random projection trees) | Annoy, KD-trees | Fast build, simple | Read-only-ish; lower recall at high dim |

Production indexes are *combinations* — HNSW over PQ codes, IVF over PQ codes, binary-filter-then-rescore — because each family alone is either too memory-hungry (graph) or too lossy (quantization). The "index type" dropdown in every product is really a menu of these combinations.

### 2.4 The distance metrics in play

kNN is defined relative to a metric. The four in common use:

- **Cosine similarity** — the cosine of the angle between vectors; the standard for text embeddings (semantic similarity regardless of magnitude). Most "cosine" implementations actually store vectors L2-normalized and use dot product, which is mathematically equivalent and faster.
- **Euclidean / L2 distance** — straight-line distance; natural for raw (unnormalized) vectors, image embeddings, and geometric data.
- **Dot product** — unnormalized projection; the right choice when magnitude carries meaning (e.g., for some recommendation embeddings), and the fastest to compute.
- **Manhattan / L1** — sum of absolute differences; rare, occasionally used for sparse or quantized data.

The rule: **the metric must match what the embedding model was trained with** — using cosine on a model trained for dot-product ranking silently degrades results (§7).

---

## 3. Use Cases — Why Vector Search Matters

| Use case | What it looks like | Notes |
|---|---|---|
| **Semantic search / RAG** | Embed query, retrieve relevant chunks, feed to LLM | The reason the category exploded; see the RAG series |
| **Recommendation** | "Items similar to this item" — embed item catalogs, query with the user's recent item | Dot-product or cosine; can be folded into collaborative filtering |
| **Deduplication** | Find near-duplicate documents, code, images, or records by embedding proximity + threshold | Cheap win in data pipelines; thresholds need tuning |
| **Anomaly detection** | Embed normal behavior; flag vectors far from all clusters / from the query | Useful for fraud-adjacent analytics; see banking notes §17 |
| **Multimodal search** | Image, audio, video, and text in one vector space (CLIP-style models) | One index, cross-modal queries: "find images matching this sentence" |
| **Agent memory** | The agent's episodic store: past interactions, tool results, decisions retrieved by similarity | See [Beyond RAG — Agent Memory](beyond_rag_guide.md) |
| **Personalization** | User embeddings + content embeddings; nearest-neighbor as the retrieval stage | Often combined with metadata filtering (user id, region, entitlements) |
| **Entity resolution / graph enrichment** | Match entities across systems by embedding similarity rather than exact keys | See [Advanced RAG Techniques](advanced_rag_techniques_guide.md) on graph + vector |

Two properties make these use cases share infrastructure: they all query by *proximity* rather than equality, and they all tolerate approximation — a 97%-recall index is fine for all of them. That tolerance is what makes ANN-based products viable as a category.

---

## 4. The Rise with LLMs: Boom, Hype, and Consolidation

### 4.1 The 2023 vector database boom

Vector databases existed before LLMs — FAISS (2017), Annoy (2015), Milvus (2019), Weaviate (2019) — but remained a niche for ML teams. The ChatGPT moment in late 2022 changed that overnight: RAG needed a retrieval store, every tutorial used a vector database, and within months a gold rush was underway. By mid-2023 there were **dozens of vector database startups** — Pinecone, Qdrant, Weaviate, Chroma, Milvus/Zilliz, LanceDB, pgvecto.rs, Vald, Marqo, Typesense Vector, Vearch, and more — each raising venture funding on the "every LLM app needs a vector DB" thesis. Gartner's 2023 hype cycle duly placed vector databases near the **Peak of Inflated Expectations**. Analyst coverage, conference talks, and LinkedIn content all treated "vector database" as a mandatory component of any AI architecture.

### 4.2 The honest assessment: hype met gravity

With 2025+ hindsight, three things happened:

1. **The incumbents added vectors.** Postgres (pgvector), Elasticsearch, OpenSearch, Redis, MongoDB, SQL Server, Oracle (23ai AI Vector Search), Snowflake, ClickHouse, DuckDB — every major database shipped vector support within 18 months. The "you need a *new* database for vectors" pitch died; the default question became "does my existing database already do this?" — and for most teams the answer was yes, well enough.
2. **Consolidation.** The startup field thinned. Some were acquired (e.g., Zilliz remains independent; several smaller players were absorbed or folded), some repositioned, and the surviving dedicated vendors (Qdrant, Milvus/Zilliz, Weaviate, Pinecone) consolidated around differentiated features: advanced filtering, hybrid search, managed simplicity, GPU acceleration. The landscape is thinner and healthier than in 2023 (see §10–§13 for who is actually left and what each is best at).
3. **The "do you need it at all?" question got asked.** As §15 details, a large fraction of RAG workloads are served perfectly well by pgvector or even in-memory FAISS. Dedicated vector databases earn their keep at scale, with hard filtering requirements, hybrid search, high QPS, or when the team wants managed operations.

### 4.3 Where the market landed (2026 snapshot)

- **pgvector is the most-used vector index** by deployment count — not because it is fastest, but because it is Postgres, and Postgres is already everywhere.
- **Pinecone** remains the reference managed SaaS — easiest path, serverless pricing, closed source.
- **Qdrant** is the open-source darling for performance + filtering; **Milvus/Zilliz** leads at truly large scale with GPU acceleration; **Weaviate** owns the hybrid-search/modular niche; **Chroma/LanceDB** own the embedded, Python-first developer niche.
- **Managed RAG stacks** (AWS Bedrock Knowledge Bases, Azure AI Search, Google Vertex AI Vector Search) let teams skip running a vector DB entirely.

The practical consequence for architects: **vector capability is now table stakes across databases; the interesting decisions are about scale, filtering, hybrid search, and ops burden** — which is what the rest of this guide is organized around.

---

## 5. How Vector Search Works: Embeddings

### 5.1 Where embeddings come from

Embeddings are produced by dedicated embedding models. The vector database stores whatever the model emits; the database itself does not know or care what the dimensions "mean." The mainstream options:

| Model family | Typical dims | Notes |
|---|---|---|
| OpenAI `text-embedding-3-small` | 1536 (down-projectable to 512/256) | Cheap, good general quality; API-only |
| OpenAI `text-embedding-3-large` | 3072 (down-projectable) | Top general quality; 8× the storage of small |
| BGE (BAAI) | 768 / 1024 | Strong open-source Chinese+English model; bge-m3 does dense + sparse + multi-vector |
| E5 (Microsoft) | 384 / 768 / 1024 | Open source, strong for retrieval (BEIR leaderboard) |
| sentence-transformers (SBERT) | 384–1024 | The open-source ecosystem; thousands of fine-tuned variants, self-hostable |
| Cohere Embed | 384 / 1024 / 4096 (v3) | Managed API; multilingual; also offers rerank |
| Gemini embedding | 768 / 3072 | Google managed API |
| CLIP (images/text) | 512 / 768 | Multimodal: images and text in one space |
| Voyage | 1024 | RAG-focused managed API |

### 5.2 The dimension trade-off

**Dimension is a quality-vs-resource knob:**

- **Higher dimensions** → more expressive space → better retrieval quality (up to a point), but **more storage, more RAM, slower distance computation, and larger index builds** — all costs scale linearly (or worse) with dimension.
- **Lower dimensions** → cheaper everything, at some recall cost. The field's finding: for many corpora, 256–768 dims captures most of the value; 3000+ dims buys a few points of recall at 4–10× the memory.

The memory math is unforgiving: **1M vectors × 3072 dims × 4 bytes = ~12 GB** raw, before any index overhead (§16 has the full sizing formula). This is why quantization (below) and dimension reduction are standard techniques: many providers let you *down-project* embeddings (OpenAI supports Matryoshka-style truncation natively), and Matryoshka Representation Learning (MRL) models are trained so that truncating dimensions degrades gracefully.

**Practical rule:** choose the smallest dimension that meets your retrieval-quality bar on *your* data (measured, not assumed — see §18), then size infrastructure against it. A 384-dim model that hits 95% of a 3072-dim model's quality at 1/8 the memory is usually the right production choice.

### 5.3 The embedding pipeline (what the database sees)

```
documents → chunk → embed → [vector, payload/metadata] → insert into vector DB
query     → embed (same model!) → [query vector]      → ANN search → top-k
```

Two operational commandments:

1. **The same embedding model must be used at ingest and query time** — mixing models (or silently upgrading the model) breaks retrieval, because the vector spaces are different. This is the root of the re-indexing problem (§16.5).
2. **Embedding is upstream of the database.** The database's quality ceiling is set by the embedding model + chunking strategy, which is why the [RAG Optimization Techniques guide](rag_optimization_techniques_guide.md) treats embedding selection and chunking as retrieval-stage levers. A perfect vector index cannot rescue bad embeddings.

### 5.4 Practical embedding operations

- **Normalization.** Normalize vectors (L2 unit length) at ingest *and* query; this makes cosine ≡ dot product and lets engines use the fast path. Store the convention in the schema docs — mixing normalized and raw vectors in one index silently corrupts ranking.
- **Batching.** Embedding APIs and local models are far more efficient on batches (100–1000 texts per call) than one-at-a-time; the ingest pipeline should batch, checkpoint, and retry. Embedding cost/latency is often the ingest bottleneck, not the database.
- **Model and version pinning.** Record `model_name`, `model_version`, `dimension`, `normalization` as index metadata (and per-vector payload when migrating). This is what makes the §16.5 re-embedding playbook executable instead of a leap of faith.
- **Query-time embedding.** The query must be embedded with the *same* model and normalization as the corpus — the most common silent bug in production RAG is a pipeline where ingest and query paths drifted apart (different model, different SDK version, one normalizes and the other does not).
- **Dimension reduction (MRL).** Matryoshka-style models allow truncating dimensions (e.g., 3072 → 768) with graceful quality decay; benchmark the truncated embedding on your golden set (§18) — the memory savings (4×) often justify a point or two of recall loss.

---

## 6. ANN Index Algorithms: HNSW, IVF, PQ, and Quantization

The index is the heart of a vector database — the data structure that makes ANN search fast. Four families dominate, and every product's "index type" is a combination of them.

### 6.1 HNSW — Hierarchical Navigable Small World (the default)

**What it is:** a multi-layer graph. Every vector is a node; edges connect near neighbors. The graph is layered: a few nodes on the top layers (long "small-world" jumps), denser connectivity on lower layers down to the base layer containing all nodes. Search starts at a fixed entry point on the top layer and greedily walks toward the query, descending a layer whenever a local minimum is reached; the base layer then yields the final neighbors.

**Why it won:** best quality/speed trade-off across the board — high recall at low latency, no training phase (vectors can be added incrementally), and excellent behavior under moderate concurrency. It is the default index in Qdrant, Weaviate, Milvus, Pinecone, pgvector, Elasticsearch, and FAISS's `IndexHNSW`.

**Parameters (the three you will tune):**

- **M** — max number of connections per node. Higher M = better recall, more memory (each edge costs 8–16 bytes), slower builds. Typical 16–64.
- **efConstruction** — build-time search width; higher = better graph quality, slower builds. Typical 100–200.
- **efSearch** — query-time search width; the main latency/recall dial. Higher efSearch = higher recall, slower queries. Typical 100–400.

**The cost:** HNSW is **memory-hungry** — the graph lives in RAM for performance, and edges add ~1.5–2× overhead over raw vectors (M=16). Disk-based variants exist (e.g., Qdrant's on-disk HNSW) but trade latency for memory. For a "set it and forget it" default with strong quality, HNSW is it.

### 6.2 IVF — Inverted File (cluster-based)

**What it is:** partition vectors into **nlist** clusters (k-means/Voronoi) at build time; each cluster gets an inverted list. At query time, examine only the **nprobe** clusters whose centroids are nearest to the query — a classic two-stage "coarse search then refine" design.

**Variants:**

- **IVF-FLAT** — full-precision vectors in the lists; exact distances within the probed clusters. Simple, disk-friendly, decent speed.
- **IVF-PQ** — vectors inside lists are product-quantized (below) for massive compression; distances computed approximately against compressed codes. The classic disk-based big-scale design (this is what FAISS's "big" pipelines and old Milvus used).

**Parameters:** **nlist** (number of clusters — more clusters = finer partition, but each query must choose among them), **nprobe** (clusters examined per query — the recall/latency dial, like efSearch for HNSW).

**The trade:** IVF needs a build/training phase, has a coarse-filtering recall ceiling (missed clusters are missed results), and is generally out-performed by HNSW on quality at equal latency — but it is **far more memory- and disk-friendly**, which is why it survives for very large corpora and cost-constrained deployments.

### 6.3 PQ — Product Quantization (compression)

**What it is:** split each vector into **m subvectors** (e.g., a 768-dim vector into 8 chunks of 96 dims), and quantize each subvector independently against a small codebook (e.g., 256 centroids = 8 bits). The vector becomes **m bytes** instead of 4×dim. A 768-dim float32 vector (3072 bytes) → 8–16 bytes: **up to ~96% memory reduction**, at the cost of lossy distances (distances are computed against quantized representatives via lookup tables).

**Position in the stack:** PQ is not a search algorithm by itself — it is a *compression layer* that combines with a search structure: **IVF+PQ** (FAISS's classic), **HNSW+PQ** (Qdrant, Milvus support PQ over HNSW), or PQ as a "rescoring" stage (coarse search on compressed codes, refine on raw vectors). Always lossy; the loss is usually acceptable because it trades memory for recall at a favorable rate — and you can recover precision by rescoring the top candidates against raw vectors.

### 6.4 Scalar quantization (int8)

**What it is:** map each float32 value to an int8 range per dimension (min-max scaling, optionally per-vector or per-dimension). 4 bytes → 1 byte per value, **4× compression**, with minimal quality loss for most real embedding distributions (which are roughly Gaussian and cluster near zero).

**Where it sits:** the cheap default compression in Qdrant, Milvus, Weaviate, and FAISS. Far less lossy than PQ for the same 4× factor; PQ only wins at higher compression ratios. In practice: **int8 scalar quantization is usually the first memory lever to pull** — 4× RAM savings for a few tenths of recall points.

### 6.5 Binary quantization (1 bit)

**What it is:** threshold each dimension to a single bit (sign-based, or learned thresholds). 32× compression from float32. Distances become XOR/popcount — extremely fast, extremely cheap.

**Why it works at all:** for **very high-dimensional** embeddings (768+), binary codes still retain surprising ranking signal — and modern practice uses binary as a *filter stage*: search the binary index cheaply, then rescore the top candidates with full-precision vectors. This "binary + rescore" pattern is why Milvus and Qdrant ship binary options and why there was a wave of papers (and provider features) on binary quantization for embeddings in 2024–2026.

**Caveat:** for low-dim embeddings (384), binary alone degrades recall badly. Use it for high-dim data, or with rescoring, and always measure.

### 6.6 Hybrid indexes

Real products combine the above: **HNSW+PQ** (graph navigation over compressed vectors — Qdrant, Milvus, Vespa), **IVF+PQ** (FAISS, classic), **binary + full-precision rescore** (Milvus), HNSW with scalar quantization (everyone). The combination is chosen along the memory/latency/recall frontier:

| Combination | Memory | Latency | Recall | Typical use |
|---|---|---|---|---|
| HNSW (float32, RAM) | 1.5–2× raw | Lowest | Highest | Default; <10M vectors, money-is-no-object latency |
| HNSW + int8 | ~0.5× raw | Low | Very high | The cost/quality sweet spot |
| HNSW + PQ | ~0.05–0.1× raw | Low–med | High (rescoring helps) | Large RAM-constrained deployments |
| IVF + PQ | ~0.05× raw | Med | Medium–high | Very large corpora, disk-based, batch-ish workloads |
| Binary + rescore | ~0.03× raw | Lowest | High (with rescore) | Very high dim, extreme scale |

### 6.7 The algorithm trade-offs table

| Algorithm | Recall | Latency | Memory | Build time | Best for |
|---|---|---|---|---|---|
| **Brute force / flat** | 100% (exact) | High (O(N·D)) | Low (raw only) | None | Small collections, exactness requirements, GPU-accelerated |
| **HNSW** | Highest | Lowest | High (RAM-resident graph) | Medium (no training) | Default quality/speed; incremental inserts; <10–50M vectors |
| **IVF** | Medium–high (cluster-bound) | Medium | Low–medium (disk-friendly) | Medium (k-means training) | Very large corpora, memory-constrained, batch builds |
| **PQ (with HNSW/IVF)** | Medium–high (lossy) | Low–medium | Very low (up to 96% off) | Higher (codebook training) | Compression at scale; RAM-constrained; rescoring available |
| **Scalar int8** | High (slightly lossy) | Low | 4× off raw | Low | First memory lever; safe default compression |
| **Binary** | Medium alone, high + rescore | Lowest | 32× off raw | Low | Very high dim; filter-first search |

**The selection heuristic:** start with HNSW (float32 or int8) — it is the default for good reason. Move to IVF/PQ variants only when the corpus or RAM budget forces it. Add rescoring whenever you quantize. Verify every choice on your own data (§18).

### 6.8 DiskANN and disk-resident indexes

The memory wall (§16.2) spawned a fourth-generation design: **DiskANN** (Microsoft, 2019) — a graph index built like HNSW but stored on SSD, with a compressed copy (PQ) kept in RAM for candidate generation and full-precision vectors fetched from disk only for rescoring. It delivers near-HNSW recall at a fraction of the RAM (index on disk, small RAM cache), at the cost of higher p99 latency (disk reads) and careful SSD sizing. Milvus ships DiskANN; Vespa has similar disk-resident graphs; Qdrant offers on-disk HNSW. **When to reach for it:** corpora where RAM-resident HNSW is unaffordable (hundreds of millions of vectors) but IVF's recall ceiling is unacceptable. When *not*: any workload whose latency SLO rules out disk reads in the hot path.

### 6.9 The anatomy of a real ANN query (HNSW walkthrough)

What actually happens when a query hits an HNSW index (the mental model that explains every tuning knob):

1. **Entry.** Start at the fixed entry point on the top (sparsest) layer.
2. **Greedy descent.** Repeatedly move to the neighbor closest to the query, until no neighbor is closer (local minimum); then drop to the next layer down and repeat — each layer refines the region, and the base layer (all nodes) yields the final candidates.
3. **Beam search on the base layer.** Maintain a candidate list of size **efSearch**; expand the best candidates' neighbors, keeping the efSearch closest — the classic best-first traversal. Larger efSearch = wider exploration = higher recall, slower queries.
4. **Return top-k** (k ≤ efSearch), typically after a final exact re-ranking of the candidates and any metadata-filter application (filtered ANN interleaves the predicate into step 3; post-filtering applies it here — and silently shrinks k, per §8.1).
5. **Rescore (optional).** If the index stores compressed vectors (PQ/binary), recompute exact distances on the raw vectors for the final top candidates.

Every parameter in the products maps to this walk: efSearch and nprobe widen exploration; M and efConstruction determine graph quality at build time; quantization determines what "distance" is computed on. This is why "tune the index" means "trade exploration width against latency" — and why recall@k is the scoreboard (§18).

---

## 7. Distance Metrics: Cosine, L2, Dot Product

### 7.1 The three metrics and when each

| Metric | Formula intuition | Use when | Notes |
|---|---|---|---|
| **Cosine** | angle between vectors, ignores magnitude | Text embeddings, semantic similarity | Standard for RAG; equivalent to dot product on L2-normalized vectors |
| **L2 / Euclidean** | straight-line distance | Raw vectors, image/geometric embeddings | Sensitive to magnitude; natural for clustering |
| **Dot product** | projection, magnitude-sensitive | Recommendation-style embeddings trained for it, some ranking models | Fastest to compute; no normalization step |
| **Manhattan / L1** | sum of absolute diffs | Rare; sparse/quantized data | Few products support it |

### 7.2 The rules that matter

1. **Normalized embeddings → cosine ≈ dot product.** Most embedding models (OpenAI, BGE, E5, sentence-transformers in retrieval configs) are used with normalized vectors; then cosine similarity and dot product rank identically, and engines implement the fast path (dot product) while exposing "cosine."
2. **The metric must match the model's training.** Models are trained with a specific similarity objective; using the wrong metric silently degrades ranking. If you do not know, **cosine on normalized vectors is the safe default** for text embeddings; dot product for models explicitly trained with it.
3. **L2 vs cosine on raw vectors:** L2 measures absolute distance, so a "long" vector and a "short" vector pointing the same way are far in L2 but identical in cosine. Choose based on whether magnitude is meaningful (rarely is, for text).
4. **Switching metrics means re-indexing** (or at least re-normalizing) — it is a schema decision, not a query parameter. Decide before production.

---

## 8. Filtering and Hybrid Search

### 8.1 Metadata filtering — the performance trap

Real applications almost never search the whole collection: they search *within* a tenant, a date range, an entitlement scope, a document type. That means the vector search must combine with **metadata predicates** (payload fields: `tenant_id = X AND doc_type = 'trade_confirm'`). There are three implementation strategies, with very different quality characteristics:

| Strategy | How it works | The trap |
|---|---|---|
| **Pre-filter** | Apply metadata predicates *first*, search only the filtered subset | If the subset is small (e.g., one tenant's 10k docs out of 50M), the ANN index may not be searchable within it — you fall back to slow scans or an unindexed mess. Also: filtered subsets can be too small for the graph to navigate well. |
| **Post-filter** | ANN-search top-k (say 100), then drop results failing the predicate | **The classic trap:** if only 3 of the 100 pass the filter, you return 3 results instead of the true filtered top-k. Recall collapses silently — you miss the exact documents the user needed. Post-filtering after too-small k is the single most common production bug in filtered vector search. |
| **Filtered ANN** (native) | The index itself is filter-aware: graph traversal/pruning respects the predicate (e.g., Qdrant's filtered HNSW, pgvector 0.8+ filtered HNSW, Weaviate's filtered search) | Requires product support; may need per-filter-index trade-offs. This is the correct approach when filters are a first-class part of the query mix. |

**Operational consequences:** if filters are core to your workload, treat them as a *selection criterion for the database itself* — Qdrant, Weaviate, Milvus, Elasticsearch, and pgvector 0.8+ have real filtered-ANN support; some embedded options (raw FAISS) do not, and you end up hand-rolling pre-filtering (FAISS's `IDFilter` helps but with the same subset-search caveats). Test with your own filter selectivity (§18) — a filter that matches 1% of the corpus behaves completely differently from one that matches 50%.

### 8.2 Hybrid search — vector + keyword

**Hybrid search** combines dense (vector) and sparse (keyword/BM25) retrieval and fuses the results. Why it exists: embeddings are great at *semantics* but systematically bad at exact terms — IDs, product codes, regulation numbers, names, boilerplate phrases — while BM25 is exact but blind to meaning. RAG corpora (especially financial ones) are full of both.

The standard implementations:

- **Same-engine hybrid** — the database stores both dense vectors and a sparse/BM25 index and fuses natively: Weaviate (BM25F + vector), Qdrant (sparse vectors + dense, Query API with fusion), Elasticsearch/OpenSearch (dense + BM25 with RRF), Milvus (sparse/dense in one collection).
- **Two-engine hybrid** — a separate BM25 index (Elasticsearch, or in-process BM25s) plus the vector DB, fused in the application or via RRF. More moving parts but lets you pick best-of-breed per side.
- **Learned sparse models** — instead of BM25, a model emits sparse vectors (SPLADE, or bge-m3's sparse head), which lets the vector database itself handle both channels.

Fusion algorithms (reciprocal rank fusion, weighted sums, rank-based ensembles) and when hybrid beats pure dense are covered in the [RAG Optimization Techniques guide](rag_optimization_techniques_guide.md) — the database-level takeaway: **hybrid support is now a standard feature, not an exotic one**, and if your queries mix exact terms with semantics, prefer a product with native sparse + dense fusion (Weaviate, Qdrant, Elasticsearch/OpenSearch, Milvus).

---
## 9. Vector Database Internals

A vector database is a distributed database with a specialized query engine. Understanding the internals matters because the failure modes and tuning knobs differ from relational databases, and because "it's just a search index" is how production incidents start.

### 9.1 The three-layer architecture

| Layer | What it does | Typical implementation |
|---|---|---|
| **Storage engine** | Persists vectors + metadata/payload; owns durability | Log-structured storage (WAL + segment files, like LSM), or columnar files (Lance), or Postgres rows (pgvector) |
| **Index** | The ANN structure (§6) that makes search fast | HNSW (RAM-resident), IVF/PQ (disk), hybrid combos; usually memory-mapped or fully in RAM |
| **Query engine** | Executes ANN search + metadata filters + scoring/fusion/reranking; the query planner that decides index traversal | Per-node search → merge (scatter-gather) in distributed setups; supports query-time parameters (efSearch, nprobe) |

Around these sit the API layer (gRPC/REST/GraphQL), the distributed coordination layer (Raft, gossip, or cloud control plane), and the clients/SDKs.

### 9.2 The write path: insert → index build

- **Insert:** vector + payload arrives → validated → appended to the write-ahead log → acknowledged → asynchronously flushed to the storage engine → **index update**.
- **Incremental vs batch index build:**
  - **Incremental** — HNSW supports point inserts (add a node, connect edges). Simple, real-time visibility, but each insert costs graph traversal and can degrade graph quality over many updates; products batch small inserts into buffers (Qdrant's "optimizer," Milvus's "segments") to amortize cost.
  - **Batch** — IVF/PQ require a *training phase* (k-means over a sample to build codebooks/clusters) before vectors can be added; batch-loaded corpora (the usual RAG ingest) build far more efficiently in bulk. Expect: bulk load 1M vectors in minutes; point-by-point in hours.
- **The tension:** every insert touches the index, and index maintenance (edge updates, segment compaction) is the main background CPU/IO cost. Plan ingest as batch jobs with batching (1k–10k vectors per request), not per-document inserts at RAG-query time.

### 9.3 The delete/update path: tombstones and the append-only reality

Vector indexes are **fundamentally append-oriented**: graph edges and cluster assignments are expensive to rewire, so most products treat deletes and updates as:

- **Soft delete / tombstone** — mark the vector dead, exclude it from search, physically reclaim it later during compaction/optimization. Cheap, immediate, and the default in Qdrant, Milvus, Weaviate, and Pinecone.
- **Hard delete** — actually remove from storage and index. Expensive: HNSW node removal requires re-linking neighbors; products either rebuild affected structures lazily or defer to compaction.
- **Update = delete + insert** — replace the vector, mark the old one tombstoned. Note the *semantic* hazard: updating a *payload* (metadata) without the vector is cheap and indexed; updating the *vector itself* changes its position in the index.

This mirrors the cache-design lessons in the [Agent Runtime Cache Design guide](agent_runtime_cache_design_guide.md) (tombstones, invalidation, deferred reclamation) — the same patterns, in a database. **Consequence for architects:** if your workload is delete-heavy (e.g., documents expire, tenants churn), either accept compaction costs, use partitioning by lifecycle (drop whole partitions — the cheapest delete of all), or choose a product with efficient deletion. "Delete 5% of a 50M-vector HNSW daily" is a design problem, not an afterthought.

### 9.4 Persistence: WAL + snapshots

Durability follows the standard recipe, adapted:

- **WAL (write-ahead log)** — every acknowledged write hits the log first; the log is replayed on crash recovery. Vector payloads make WALs large; products batch and compress.
- **Snapshots / segments** — periodic point-in-time images (or immutable LSM-style segments) that bound replay time and enable backup.
- **Index durability** — the RAM-resident HNSW graph itself is *rebuilt* from the persisted vectors on restart (fast for millions of vectors, minutes for hundreds of millions) or persisted as an index snapshot (Qdrant, Milvus persist index files; FAISS-persisted indexes work too). Know which model your product uses — it determines restart time and whether index parameters survive restarts.

### 9.5 Replication: primary-secondary and Raft

- **Primary-secondary** — one node owns writes; replicas serve reads (or standby). Simplest; a primary failure needs failover logic. Pinecone serverless and most managed offerings abstract this away entirely.
- **Raft consensus** — the distributed-consensus protocol (leader election, log replication, majority commits) used by Qdrant, Milvus, and others for consistent metadata/coordination. Vector data replication often uses the same log or async segment shipping; read-your-writes vs. eventual visibility is a config choice.

**The consistency spectrum:** strong consistency (write acknowledged only after majority replication — slower writes, no lost updates) vs. eventual (fast, cheap, but readers may briefly miss recent writes). For RAG, the sweet spot is usually **read-your-writes for the ingest pipeline** (so a just-ingested document is searchable) and eventual read replicas for scale — but the exact contract differs per product; test it.

### 9.6 Sharding: scatter-gather

At scale, the collection is **sharded** — split across nodes, typically by hashing the vector ID (or tenant), sometimes by vector-space partitioning. A query becomes **scatter-gather**: broadcast to all shards, each runs ANN locally (top-k × shard), a coordinator merges and re-ranks. Consequences:

- Recall at the cluster level is bounded by per-shard recall (each shard returns its local top-k; the global top-k may straddle shards — hence "oversearch" factors in production).
- Shard count, per-shard size, and replication factor are the capacity-planning levers (§16.4).
- Filtered queries interact badly with naive sharding: if you shard by tenant, single-tenant queries hit one shard (great); if you shard globally, every filtered query touches every shard (expensive). **Shard key choice is an architectural decision, not an ops detail.**

### 9.7 The CRUD vs. append-only tension — the design philosophy

Conventional databases are built around *mutation*: rows are updated in place, indexes are B-trees designed for arbitrary point changes. Vector databases are built around **append + search**:

- Appends are cheap and streaming-friendly; the index absorbs them incrementally.
- **Updates and deletes are the expensive, second-class operations** — tombstoned, deferred, compacted.
- Transactionality is typically *per-document* (an insert is atomic), not multi-document: no joins, no ACID transactions spanning vectors, no `UPDATE ... WHERE` across the index. (pgvector inherits full Postgres ACID — one of its structural advantages; the "just use Postgres" argument in §15 leans heavily on this.)

**The architect's framing:** a vector database is a *retrieval store with database hygiene*, not a general-purpose database. If your data needs real relational modeling, joins, or transactional updates, that is a signal to keep the vector store a *sidecar* to your system of record (Postgres/Oracle) rather than the system of record — a point that matters double in banking (§17).

### 9.8 Segment-based storage and compaction (the LSM inheritance)

Most dedicated vector DBs (Qdrant, Milvus, Weaviate, Elasticsearch) borrow the **log-structured merge (LSM)** playbook from modern key-value stores:

- Incoming writes land in a small **in-memory buffer / memtable**, which flushes to immutable **segments** on disk (each segment = a self-contained chunk of vectors + payload + its own index slice).
- Reads search the buffer plus all relevant segments; per-segment indexes keep this fast.
- Background **compaction** merges small segments into larger ones, reclaiming tombstoned vectors (§9.3) and rebuilding index slices — this is the "optimizer" job in Qdrant, segment merging in Milvus, Lucene merges in Elasticsearch.

**Why the architecture matters operationally:**

- **Write throughput** is high (buffered, batched, sequential) — but *searchable latency* depends on flush cadence: a vector is not queryable until its segment is flushed/sealed. "I inserted it, why can't I find it?" is usually a flush/refresh-interval question, not a bug.
- **Compaction is the hidden cost.** It consumes CPU/IO in bursts; if it competes with query traffic, p99 latency spiking during compaction windows is expected unless you schedule quiet hours or throttle merges (Qdrant exposes optimizer limits; Milvus has compaction tuning).
- **Delete efficiency is a segment-count property.** Dropping a whole partition/segment is O(1)-ish; deleting scattered vectors is deferred to compaction. Lifecycle design (partition per month/tenant) turns expensive deletes into cheap segment drops (§9.3).
- **Consistency between buffer and segments** is where read-your-writes guarantees live and die: products differ on whether a just-inserted vector is immediately searchable (Qdrant: yes with default settings; Elasticsearch: after refresh; Milvus: after segment flush). Know your product's contract — it determines whether the ingest pipeline needs its own read-your-writes handling.

---
## 10. The Technology Landscape: Four Categories

The "vector database market" is really four overlapping markets. Almost every guide conflates them; keeping them separate is the key to sane selection.

| Category | What it is | Open source | Who owns ops | Representative products |
|---|---|---|---|---|
| **1. Dedicated vector databases** | Standalone systems built for similarity search as the primary workload | Mostly | You (self-host) or vendor (cloud) | Qdrant, Milvus, Weaviate, Pinecone, Chroma, Vespa, Vald, LanceDB |
| **2. Vector search libraries** | Embedded in-process index code — not a database | Yes | You (in your app) | FAISS, ScaNN, Annoy, HNSWlib, USearch, NGT |
| **3. Relational/search DB vector extensions** | Vector capability in an existing database | Mostly | Your existing DB team | pgvector, SQLite-vec, Elasticsearch/OpenSearch kNN, Redis, MongoDB Atlas, Oracle 23ai, SQL Server |
| **4. Managed/cloud offerings** | Fully hosted vector search or RAG components | Mixed (managed open core or proprietary) | Vendor | Pinecone, Zilliz Cloud, Qdrant Cloud, Weaviate Cloud, AWS Bedrock KB / OpenSearch Serverless, Azure AI Search, Vertex AI Vector Search, Supabase, Neon |

The four categories answer different questions: *do I want a server or a library?* (1/4 vs 2), *do I want a new system or extend what I have?* (1/4 vs 3), *do I want to run it or rent it?* (1/3 vs 4). §14 turns these into a decision procedure.

---

## 11. Dedicated Vector Databases — Vendor Profiles

### 11.1 Qdrant — the open-source performance/filtering favorite

- **Stack:** Rust; gRPC + REST APIs; open source (Apache 2.0) with Qdrant Cloud (managed) and Hybrid Cloud.
- **Strengths:** best-in-class **metadata filtering** (filtered HNSW is genuinely good, with payload indexes); **quantization** (scalar, PQ, binary) built in; **sparse vectors + dense** for native hybrid; excellent single-node performance; predictable on-prem deployment (single binary, no external dependencies).
- **Trade-offs:** distributed mode (sharding/replication) is a paid/cloud-tier feature in parts of the lifecycle; smaller ecosystem than Postgres; younger than some peers.
- **Best for:** production teams that need real filtering + performance, self-hosted or managed; the current default recommendation for "serious open-source vector search."

### 11.2 Milvus — distributed scale and GPU acceleration

- **Stack:** Go/C++; LF AI & Data Foundation project; Apache 2.0; Zilliz Cloud (managed, founded by Milvus creators). Decouples storage (object store), index nodes, query nodes, and meta store (etcd).
- **Strengths:** designed for **very large scale** (billions of vectors) from day one; **GPU-accelerated indexing** (CAGRA etc.) — the only mainstream OSS vector DB with serious GPU story; rich index menu (HNSW, IVF, DiskANN, scalar/binary quantization); sparse + dense hybrid.
- **Trade-offs:** heavier architecture (more components to operate) — overkill for small deployments; complexity of tuning the distributed parts.
- **Best for:** large corpora (>100M vectors), GPU-accelerated pipelines, teams that need distributed open source without vendor lock.

### 11.3 Weaviate — the modular/hybrid veteran

- **Stack:** Go; open source (BSD) + Weaviate Cloud; GraphQL *and* REST APIs; "modules" for vectorizers, rerankers, generative models.
- **Strengths:** long-standing **hybrid search** (BM25F + vector fusion); GraphQL is a genuine differentiator for complex queries; modules make it easy to embed at ingest and rerank at query; good multi-tenancy story.
- **Trade-offs:** some features have drifted toward the cloud tier; performance historically a step behind Qdrant on pure ANN benchmarks (competitive in hybrid workloads).
- **Best for:** teams that want hybrid search, GraphQL, and an all-in-one vector + keyword + rerank system.

### 11.4 Pinecone — the managed SaaS reference

- **Stack:** proprietary, fully managed; serverless architecture (indexes scale to zero); SDKs for Python/Node/Go; no self-host option (there is a "Pinecone Hybrid Cloud" for regulated environments).
- **Strengths:** the **easiest on-ramp** in the industry — create an index, upsert, query; serverless pricing removes capacity planning; solid filtering (serverless supports metadata filtering); namespaces for tenancy.
- **Trade-offs:** closed source — no code audit, no self-host (a dealbreaker for some regulated environments); cost at scale can exceed self-hosted open source by an order of magnitude; vendor lock-in (data migration out is a project).
- **Best for:** teams that want zero ops and fast time-to-value; prototypes that grow into production without an infra team.

### 11.5 Chroma — the embedded Python-native dev favorite

- **Stack:** Python (core) + Rust; Apache 2.0; runs embedded in-process or as a client/server; "the ChatGPT-retrieval default" in tutorials.
- **Strengths:** **zero-config developer experience**; Python-native API that feels like a dict; filters, collections, and now hybrid (BM25) support; ideal for notebooks, prototypes, and small production apps.
- **Trade-offs:** not built for heavy production scale or high QPS; limited distributed story; durability/ops maturity behind the server-class peers.
- **Best for:** Python developers, prototyping, embedded/desktop apps, small-to-mid collections (<a few million vectors).

### 11.6 Vespa — the full-featured search engine (Yahoo)

- **Stack:** Java/C++; open source (Apache 2.0); from the team that runs Yahoo search. **Not** a vector-only system: vectors + text (BM25) + structured data + ranking expressions in one engine.
- **Strengths:** the most **complete search platform** in this list — ANN + exact + filters + ranking functions (including learned ranking) + real-time indexing at web scale; used for recommendation systems at scale.
- **Trade-offs:** steep learning curve; its own config/ranking language; smaller community than the RAG-era products.
- **Best for:** high-performance search/recommendation platforms that need vector + text + structured search fused with custom ranking — overkill for simple RAG.

### 11.7 LanceDB — embedded, columnar, multimodal

- **Stack:** Rust; embedded (no server) with a Lance columnar format; Apache 2.0; serverless/multimodal positioning (images, video, point clouds).
- **Strengths:** **embedded** like SQLite/FAISS but with database features (versioning, filtering, full-text); columnar storage makes large multimodal payloads cheap; Python/JS/TS APIs; plays well with DuckDB/Polars ecosystems.
- **Trade-offs:** younger project; embedded model means you own concurrency/HA; the columnar design favors analytical access patterns.
- **Best for:** multimodal datasets, data-science/Python-centric pipelines, edge/embedded deployments.

### 11.8 Also in the dedicated category

- **Vald** — Kubernetes-native distributed vector search (Japan-origin, Yahoo Japan); NGT-based; niche but solid for K8s shops.
- **pgvecto.rs** — a Postgres extension written in Rust (alternative to pgvector) with better filter/quantization support; straddles categories 1/3; pgvector remains the default, pgvecto.rs the performance alternative.
- **Marqo / Typesense Vector / txtai** — smaller entrants; Marqo (tensor search over a text-first API), Typesense (fast typo-tolerant search + vectors), txtai (embedded semantic database). Worth knowing, rarely the default pick.

### 11.9 Search incumbents with strong vector support

- **Elasticsearch** — the search incumbent: dense vector fields, HNSW (and int8/BBQ quantization), kNN search, and the strongest **BM25 + dense hybrid with RRF** in the industry; mature ops, security, and observability. If you already run Elasticsearch, its kNN is usually "good enough" — see §14.
- **OpenSearch** — the open-source fork: kNN (HNSW, IVF, and now FAISS-backed), hybrid with RRF; AWS-hosted (OpenSearch Serverless) — a common AWS-default.
- **Redis** — Redisearch vector support: fast HNSW in a cache-grade system; great for *hot* vector lookups, less so as the system of record (memory cost, durability model). The "cache/DB hybrid" — vector caching patterns are in the [Agent Runtime Cache Design guide](agent_runtime_cache_design_guide.md).
- **MongoDB Atlas Vector Search** — vectors as first-class BSON data with an integrated ANN index; attractive when the app is already MongoDB.
- **Azure AI Search** — managed search + vectors + hybrid + semantic reranking; the natural choice in Azure shops.
- **Google Vertex AI Vector Search** (formerly Matching Engine) — managed ANN at Google scale (ScaNN-based); paired with Gemini embeddings.
- **AWS OpenSearch Serverless / Bedrock Knowledge Bases** — Bedrock KB abstracts the whole RAG store (OpenSearch, Aurora, or vector-index backends behind a KB API); the "managed RAG, don't touch a database" path.
- **Oracle 23ai AI Vector Search** — Oracle's answer: `VECTOR` column type + HNSW/IVF indexes inside the converged database, aimed squarely at enterprises that will never adopt a new database.
- **SQL Server** — `VECTOR` type with kNN support (2025 preview/general availability track); Azure SQL vector search.
- **Snowflake** — vector type + `VECTOR_CS`/`VECTOR_IP` functions and ANN search (2024–2025 releases); warehouse-native.

---
## 12. Vector Search Libraries, Relational Extensions, and Managed Services

### 12.1 Vector search libraries (embedded — not databases)

Libraries give you the ANN algorithm with nothing else: no server, no persistence contract, no metadata engine, no multi-tenancy, no HA. You embed them in your process and own everything around them. They remain the right tool for a surprisingly large share of workloads (see §15).

| Library | Origin | Algorithms | Notes |
|---|---|---|---|
| **FAISS** | Meta (2017) | IVF-FLAT/PQ, HNSW, scalar/binary quantization, GPU (cuVS) | **The industry standard**; exhaustive exact search with SIMD; the reference implementation every benchmark compares against. Library, not DB. |
| **ScaNN** | Google | Anisotropic vector quantization (AVQ), tree+AQ | Famous for the ANN-Benchmarks records; internal Google search use; less general-purpose than FAISS |
| **Annoy** | Spotify | Tree-based (random projection forests) | Read-only after build; used for music recsys; simple, but superseded by HNSW for most new work |
| **HNSWlib** | nmslib | HNSW | The reference HNSW implementation; the algorithm's canonical home |
| **USearch** | Unum | HNSW, SIMD-optimized, smaller memory | Fast, single-header-ish; good for embedded/edge |
| **NGT** | Yahoo Japan | Graph + quantized variants | Used by Vald; strong on accuracy |

A dedicated research deep-dive on FAISS vs. ScaNN vs. BM25 already exists in this repo — see the [BM25 / FAISS / ScaNN Research note](bm25_faiss_scann_research.md). The database-vs-library decision is covered in §15.

### 12.2 Relational DB vector extensions — the "just use Postgres" world

**pgvector** is the most important vector extension in existence — not because it is the fastest (it is not), but because Postgres is everywhere and pgvector makes vectors *just another column*:

- `VECTOR(n)` column type; HNSW and IVFFlat indexes; exact search via operators; distance ops (`<->` L2, `<=>` cosine, `<#>` negative inner product).
- **The killer feature: SQL.** `SELECT ... FROM docs WHERE tenant=$1 ORDER BY embedding <=> $2 LIMIT 5` — joins, transactions, ACID, backups, roles, and the whole Postgres ecosystem on the same data as the vectors.
- **Version trajectory:** 0.4 (2023) → HNSW; 0.5 (2024) → iterative index scans (no more re-scanning on concurrent inserts); 0.7 (2024) → faster HNSW (half the build time, better recall at equal params); **0.8 (2025) → filtered HNSW** (metadata predicates inside index traversal — fixes the §8.1 trap), parallel index builds, and better delete handling. The gap to dedicated products narrows every release.
- **When it breaks:** at scale — HNSW memory in Postgres shared buffers, write amplification on ingest, and per-query planning overhead mean the "good enough" zone is roughly **<1M vectors comfortably, up to ~10M with care** (filters, batching, hardware). Beyond that, dedicated engines pull ahead (see §15 for the honest thresholds).

**SQLite-vec** — the embedded equivalent: vector search inside SQLite (the "SQLite of vector search"), zero-server, single-file; ideal for local/edge/desktop apps and demos; no distributed story by design. **MySQL** has no first-class vector type as of 2026 (third-party hacks only) — teams on MySQL typically add a sidecar vector store instead. **DuckDB** added vector search (2024–2025) for analytical workloads.

### 12.3 Managed / cloud offerings (map)

| Provider | Offering | Underlying tech | Best for |
|---|---|---|---|
| **Pinecone** | Serverless vector DB | Proprietary (HNSW-based) | Zero-ops managed vector DB |
| **Zilliz Cloud** | Managed Milvus | Milvus | Managed distributed at scale; GPU tiers |
| **Qdrant Cloud** | Managed Qdrant (+ Hybrid Cloud for on-prem) | Qdrant | Managed with self-host option for data residency |
| **Weaviate Cloud** | Managed Weaviate | Weaviate | Managed hybrid/modular |
| **AWS** | OpenSearch Serverless; **Bedrock Knowledge Bases**; Aurora/Neptune pgvector (RDS pgvector) | OpenSearch / managed RAG / Postgres | AWS-native teams; KB = no-database RAG |
| **Azure** | **AI Search** (vector + hybrid + semantic ranking); Azure SQL vector; Cosmos DB vector search | Proprietary + open | Azure-native enterprise search |
| **GCP** | **Vertex AI Vector Search** (ex-Matching Engine, ScaNN-based) | ScaNN | Google cloud + Gemini embeddings |
| **MongoDB Atlas** | Atlas Vector Search | Integrated ANN in Atlas | Already-on-MongoDB apps |
| **Supabase / Neon** | Managed Postgres with pgvector enabled | pgvector | Serverless Postgres + vectors; the indie-hacker default |
| **Oracle 23ai / SQL Server** | In-database vector search | Oracle/SQL Server native | Banks & enterprises that will only use the incumbent DB |

**Pattern:** every cloud's story is "use the managed service on the database you already have" (Atlas, AI Search, OpenSearch Serverless) or "use our managed RAG wrapper" (Bedrock KB). The standalone managed vector DB (Pinecone/Zilliz/Qdrant Cloud) is now one option among several rather than *the* option — another sign of category maturation.

---

## 13. The Landscape Comparison Table

Legend — **Type:** D = dedicated, L = library, E = extension/feature, M = managed. **OSS:** ✓ = open source, ✗ = proprietary, ◐ = open core (some features paid).

| Product | Type | OSS | Stack | Index algorithms | Filtering | Hybrid | Scale ceiling | Cloud | Best for |
|---|---|---|---|---|---|---|---|---|---|
| **Qdrant** | D (+M) | ✓ (◐ distro) | Rust | HNSW, int8/PQ/binary quant | ★★★ (filtered HNSW, payload indexes) | ✓ (sparse+dense) | 100M+ | Qdrant Cloud | Production filtering + performance, self-host |
| **Milvus** | D (+M) | ✓ | Go/C++ | HNSW, IVF, DiskANN, GPU (CAGRA), quant | ★★★ | ✓ | Billions | Zilliz Cloud | Very large scale, GPU acceleration |
| **Weaviate** | D (+M) | ✓ (◐) | Go | HNSW, quant | ★★☆ | ✓ (BM25F + vector) | 100M+ | Weaviate Cloud | Hybrid search, GraphQL, modules |
| **Pinecone** | D, M | ✗ | Proprietary | HNSW (managed) | ★★☆ (serverless filters) | ✓ (hybrid indexes) | 100M+ | Pinecone | Zero-ops managed, fastest time-to-value |
| **Chroma** | D | ✓ | Python/Rust | HNSW | ★☆☆ (basic) | ✓ (BM25) | ~M | n/a (embedded) | Python dev, prototyping, embedded |
| **Vespa** | D | ✓ | Java/C++ | HNSW, exact, custom | ★★★ | ✓ | Billions | Vespa Cloud | Full search/recommendation platform, custom ranking |
| **LanceDB** | D | ✓ | Rust | IVF-PQ, HNSW, disk | ★★☆ | ✓ (full-text) | 100M+ | n/a (embedded) | Multimodal, embedded, data-science pipelines |
| **Vald** | D | ✓ | Go | NGT | ★★☆ | limited | 100M+ | K8s-native | Kubernetes-native vector search |
| **FAISS** | L | ✓ | C++/Python | IVF, HNSW, PQ, quant, GPU | ✗ (manual) | ✗ | RAM-bound | n/a | In-process ANN, exact/GPU search, research |
| **ScaNN** | L | ✓ | C++ | AVQ, tree | ✗ | ✗ | RAM-bound | n/a | Max single-node recall/speed |
| **Annoy / HNSWlib / USearch / NGT** | L | ✓ | C++/Rust | tree / HNSW / HNSW | ✗ | ✗ | RAM-bound | n/a | Specialized embedded ANN |
| **pgvector** | E | ✓ | C (Postgres) | HNSW, IVFFlat | ★★☆ (0.8 filtered HNSW) | via extensions (tsvector) | 1M–10M practical | Every cloud | Already-on-Postgres teams, SQL + ACID |
| **SQLite-vec** | E | ✓ | C | HNSW (brute-force + HNSW) | ★☆☆ | limited | <1M | n/a | Local/edge/desktop embedded |
| **Elasticsearch** | E (+M) | ✓ (◐) | Java | HNSW, int8/BBQ quant | ★★★ | ✓ (BM25 + RRF, best-in-class) | 100M+ | Elastic Cloud / AWS | Search incumbent; hybrid enterprise search |
| **OpenSearch** | E (+M) | ✓ | Java | HNSW, IVF, FAISS | ★★★ | ✓ (RRF) | 100M+ | AWS OpenSearch Serverless | AWS default, hybrid, open search |
| **Redis** | E | ✓ (◐) | C | HNSW (RediSearch) | ★★☆ | via plugins | 10M+ (RAM) | Redis Cloud | Hot vector cache / low-latency lookups |
| **MongoDB Atlas** | E, M | ◐ | C++ | HNSW | ★★☆ | ✓ (Atlas Search) | 100M+ | Atlas | Already-on-MongoDB apps |
| **Oracle 23ai** | E | ✗ | C | HNSW, IVF | ★★☆ | via Oracle Text | 100M+ | OCI | Banks on Oracle; converged DB mandate |
| **SQL Server** | E | ✗ | C++ | HNSW | ★★☆ | via full-text | 10M+ | Azure SQL | Microsoft shops |
| **Snowflake** | E | ✗ | Proprietary | ANN (vector functions) | ★★☆ | via search | 100M+ | Snowflake | Warehouse-native semantic search |
| **Azure AI Search** | M | ✗ | Proprietary | HNSW + hybrid + semantic ranker | ★★★ | ✓ | 100M+ | Azure | Azure enterprise search/RAG |
| **Vertex AI Vector Search** | M | ✗ | ScaNN | ScaNN | ★★☆ | via Vertex Search | Billions | GCP | Google/Gemini shops, huge scale |
| **Bedrock KB** | M | ✗ | OpenSearch/Aurora | Managed | ★★☆ | ✓ | Managed | AWS | Managed RAG with zero DB ops |

**Reading the table:** no row wins everything. The interesting axes for a bank are filtering maturity, hybrid support, self-hostability (data residency), and the ops model — all covered in §14–§17.

---
## 14. Selection Framework: Choosing the Right Tool

Selection is a decision procedure, not a popularity contest. Walk it in order: **scale → existing stack → requirements → latency/cost → ops model**. At every step, prefer the option that removes infrastructure rather than adds it.

### 14.1 Step 1 — Scale (the first filter)

| Corpus size (vectors) | Honest guidance |
|---|---|
| **< 100k** | **Anything works.** Brute-force FAISS in memory, SQLite-vec, Chroma, pgvector. Pick for developer experience. A dedicated server-class vector DB is wasted here. |
| **< 1M** | **pgvector / SQLite-vec / Chroma / FAISS** all fine. Postgres gives you SQL + ACID for free; FAISS gives you speed in-process. |
| **1M – 10M** | **pgvector (careful)** — batch your ingest, mind the RAM, use HNSW with int8 if memory bites — **or move to a dedicated DB** (Qdrant/Weaviate) which stops you from fighting the index inside a general-purpose engine. This is the decision zone. |
| **> 10M** | **Dedicated/distributed** (Qdrant, Milvus, Vespa, or managed Pinecone/Zilliz) or Elasticsearch/OpenSearch at scale. HNSW-in-Postgres gets uncomfortable; sharding and compaction become the product's job, not yours. |

Sizing is about *vectors × dimension × overhead*, not raw document count — 10M documents × 3 chunks each × 1536 dims is 30M vectors and a very different beast than 10M × 384 dims. Do the memory math first (§16.4), then apply the table.

### 14.2 Step 2 — Existing stack (the path of least resistance)

- **Already on Postgres?** → pgvector first. It is one `CREATE EXTENSION` away, and the SQL/ACID/backup story is unbeatable for most workloads. Move off only when measured pain demands it.
- **Already on Elasticsearch/OpenSearch?** → use its kNN. The hybrid (BM25 + RRF) story there is *better* than most dedicated vector DBs.
- **Already on MongoDB / Oracle / SQL Server / Snowflake?** → same logic: use the native vector feature before adding a system.
- **Python-only team, no infra appetite?** → Chroma or LanceDB embedded; pgvector via Supabase/Neon if you want managed Postgres.
- **Production managed, no ops team?** → Pinecone, Zilliz Cloud, Qdrant Cloud, or the cloud-native path (Bedrock KB / AI Search / Vertex).

**The rule:** the marginal cost of a *new* database is far higher than the marginal cost of a *feature* in the database you already run. Add infrastructure only when the incumbent measurably fails.

### 14.3 Step 3 — Requirements (the differentiators)

| Requirement | Preferred products | Why |
|---|---|---|
| **Filters + vector (tenants, entitlements, dates)** | Qdrant, Weaviate, Elasticsearch/OpenSearch, pgvector 0.8+, Milvus | Real filtered ANN (native index-aware predicates). Avoid post-filter-only paths for low-selectivity filters (§8.1). |
| **Hybrid search (dense + BM25)** | Weaviate, Elasticsearch/OpenSearch (RRF), Qdrant (sparse vectors), Milvus | Native fusion; don't hand-roll two engines unless you must. |
| **Multimodal (image/audio/video + text)** | LanceDB, Weaviate, Milvus, Qdrant | Cross-modal embeddings (CLIP etc.) are just vectors — any DB works; LanceDB's columnar/multimodal payloads and Weaviate's multimodal modules add value. |
| **GPU acceleration** | Milvus (CAGRA), FAISS (cuVS) | GPU ANN at ingest and query; Milvus is the only mainstream OSS DB with a real GPU story. |
| **Simplicity / time-to-value** | Pinecone, Chroma, pgvector (Supabase), Bedrock KB | Fewest concepts to learn; fastest first query. |
| **SQL + ACID + joins with vectors** | pgvector, Oracle 23ai, SQL Server | Vector + relational in one transactional store. |
| **Data residency / self-host mandate** | Qdrant, Milvus, Weaviate, Vespa, pgvector, Elasticsearch (all self-hostable OSS); Pinecone Hybrid Cloud as managed-but-on-prem | See §17; this is often the *deciding* factor in banking. |
| **Extreme scale (billion+)** | Milvus, Vespa, Vertex AI Vector Search, managed Zilliz/Pinecone | Distributed, sharded, oversearch-aware architectures. |

### 14.4 Step 4 — Latency/throughput and cost

- **Latency:** HNSW-in-RAM gives single-digit-ms p99 at high recall; disk-based (IVF, on-disk HNSW) trades latency for memory. If your SLO is "retrieve in <20 ms at p99 under load," budget RAM accordingly and choose a product whose index is RAM-resident by default (Qdrant, Weaviate, Pinecone, pgvector-with-enough-memory).
- **Throughput:** QPS scales with shards × replicas; read replicas scale reads, sharding scales capacity. Oversearch factor (searching more per shard than the global k) multiplies cost — tune it.
- **Cost:** self-hosted = hardware + your engineers' time (HNSW RAM is the line item); managed = predictable subscription that includes HA/backups/upgrades. For sporadic workloads, serverless (Pinecone, Zilliz Serverless) beats paying for idle RAM. Rule of thumb: at <1M vectors, managed ≈ self-host in cost; at 100M+, managed is a premium you buy for ops-free operation.

### 14.5 A worked example (typical bank RAG)

A trade-surveillance RAG: 5M docs → 12M chunks → 768-dim embeddings (~37 GB raw) → ~60 GB with HNSW. Requirements: tenant filtering by desk/region, hybrid search for trade IDs + semantics, on-prem for residency, p99 < 50 ms, ~200 QPS.

Walk: scale >10M → dedicated; stack: greenfield (no incumbent) → self-hosted OSS; requirements: filters + hybrid + residency → **Qdrant (self-hosted)** or **Weaviate**; Milvus if GPU or billion-scale later. Verdict: Qdrant with scalar quantization (drops RAM to ~20 GB), filtered HNSW, sparse vectors for the ID terms. pgvector would be the answer if the bank's Postgres estate could host it and filters stayed simple.

### 14.6 Selection anti-patterns (what to avoid)

- **"Everyone uses X"** — popularity is a prior, not a requirement. pgvector being default does not make it wrong for a 100M-vector multi-tenant workload; Qdrant being popular does not justify adding a database to a 200k-vector app.
- **Benchmark-driven picks without your data** — choosing on vendor recall/QPS charts (§18.2) selects the vendor's workload, not yours.
- **Ignoring the filter mix** — a product that aces unfiltered ANN but fumbles low-selectivity filters will fail your tenancy model (§8.1). Filter selectivity is a first-class requirement.
- **Deferring the embedding-model decision** — the store is chosen before the embedding model, then the model choice forces re-indexing. Decide model + store together (§5, §16.5).
- **The "one store for everything" stretch** — a vector DB is a retrieval store; using it as the system of record for relational data (or vice versa) creates the worst of both worlds. Keep the sidecar pattern (§9.7).
- **No exit plan** — every managed/closed option needs a documented data-export and re-indexing path before go-live, not after the vendor relationship sours.

---

## 15. "Do You Even Need a Vector Database?"

### 15.1 The honest take

The 2023 answer was "yes, absolutely, here's a new database." The 2026 answer is: **most RAG applications do not need a dedicated vector database.**

- **At moderate scale (<1M–5M vectors), pgvector or in-memory FAISS delivers the same retrieval quality at a fraction of the operational cost.** Retrieval quality is dominated by embeddings, chunking, and reranking — not by which ANN engine answers the query. A slower-but-correct pgvector query with a good reranker beats a blazing-fast index with bad embeddings every time.
- **The dedicated product earns its keep only under specific conditions:** scale that breaks the incumbent (>10M vectors), hard metadata filtering (filtered ANN that pgvector can't yet do well), native hybrid search, sustained high QPS, GPU acceleration, or managed simplicity when the team has no infra appetite.
- **The corollary:** many teams *add* a vector DB without first measuring whether their existing stack suffices. The cheapest optimization is deleting that migration from the roadmap. Run the decision table, measure on real data (§18), and let the evidence decide.

### 15.2 The decision table

| Your situation | Recommendation |
|---|---|
| Prototype / POC / notebook | Chroma or FAISS in-process. Zero setup. |
| Small production app, already on Postgres | pgvector. SQL, ACID, backups for free. |
| Small production app, no database yet | Supabase/Neon (managed pgvector) or SQLite-vec for embedded. |
| 1–10M vectors, simple filters, one tenant | pgvector with HNSW + batching; revisit if p99 slips. |
| 1–10M vectors, complex filters / multi-tenant | Qdrant or Weaviate (filtered ANN). |
| >10M vectors | Dedicated distributed (Qdrant, Milvus, Vespa) or managed (Pinecone, Zilliz). |
| Hybrid search required (IDs + semantics) | Elasticsearch/OpenSearch or Weaviate/Qdrant — not pgvector alone. |
| Exactness / compliance-grade retrieval | Brute-force FAISS (GPU) or exact search in pgvector; document recall@k in the runbook. |
| No ops team, budget available | Pinecone / Zilliz Cloud / Bedrock KB — buy the ops. |
| Regulated, on-prem, audit-driven | Self-hosted OSS (Qdrant, Milvus, pgvector) with full control — §17. |

### 15.3 The two genuine risks of "just use pgvector"

1. **Scale cliff.** pgvector's HNSW is a Postgres index, subject to Postgres's memory model (shared buffers, per-query overhead, WAL amplification on ingest). Teams that ignored the 1M–10M guidance have hit multi-minute ingest times and p99 blowups — the fix is usually moving to a dedicated engine, which is why the scale filter (§14.1) is step one, not step three.
2. **Feature lag.** pgvector 0.8's filtered HNSW (2025) closed the biggest gap, but advanced quantization, sparse vectors, native hybrid fusion, and distributed sharding remain dedicated-DB territory. If your roadmap needs those, the migration cost is lower *before* the data grows — pick deliberately early.

**Bottom line for architects:** choose the *simplest thing that meets the measured requirements*, prefer extending what you already run, and treat a dedicated vector database as a deliberate, evidence-backed addition — not an automatic part of the AI stack.

---
## 16. Operations: Deploying and Running Vector Databases

### 16.1 Self-host vs. managed — the ops burden

| Dimension | Self-hosted (Qdrant, Milvus, Weaviate, pgvector) | Managed (Pinecone, Zilliz Cloud, Qdrant Cloud, Bedrock KB) |
|---|---|---|
| **Setup** | You: install, configure, cluster, monitor | Vendor: index in minutes |
| **Capacity planning** | You own RAM/disk sizing (§16.4) — the #1 failure mode | Vendor abstracts (serverless) or guides (dedicated tiers) |
| **Index builds / compaction** | Your background jobs, your tuning | Vendor-managed (you set parameters) |
| **HA / failover** | Your replicas, your failover tests | SLA-backed |
| **Backup / restore** | Your scripts (snapshots + WAL), restore drills | Vendor features; verify restore SLAs |
| **Upgrades** | Your maintenance windows, version pinning, migration testing | Vendor pushes; you track breaking changes |
| **Monitoring** | Your stack (Prometheus/Grafana) | Vendor dashboards + API metrics |
| **Data residency / audit** | Total control — the banking decider (§17) | Depends: cloud region, Hybrid Cloud options |
| **Cost** | Hardware + engineering time | Subscription; scales with usage |

**The honest rule:** a self-hosted vector DB is a real database in your estate — it needs the same discipline as Postgres or Elasticsearch (backups, DR, upgrades, monitoring, runbooks). Teams that underestimate this end up with an unbacked-up, unpatchable index. If the org cannot staff that, buy managed and spend the saved time on the application.

### 16.2 HNSW memory sizing

HNSW is RAM-resident; memory is the binding constraint. The formula:

```
raw vectors        = N × D × 4 bytes        (float32)
HNSW overhead      = raw × (M-dependent factor, ≈1.5–2× at M=16, more at higher M)
graph edges        = N × M × ~8–16 bytes    (neighbor pointers/ids, both directions)
working memory     = raw + overhead + query scratch (efSearch buffers, rescoring)
```

**Worked example (the classic):** 1M vectors × 768 dims × 4 bytes = **~3 GB** raw. With HNSW at M=16, overhead ≈1.5–2× → **~5–6 GB** total resident. Scale linearly: 10M × 768 = ~30 GB raw → ~55–60 GB with HNSW. Reduce by: **int8 scalar quantization** (÷4, → ~1.5 GB per 1M×768), PQ (÷20–30), binary (÷32), or dimension reduction at the model level (§5.2).

**Sizing formula cheat sheet:** `RAM ≈ N × D × 4 × (1.5 to 2.0) bytes` for float32 HNSW; `≈ N × D × 1.0` with int8; plus 20–30% headroom for index growth, WAL buffers, and compaction peaks. **Rule of thumb: double the raw-vector size, then double again for comfort.** Underestimate RAM and the index falls out of cache — p99 latency collapses.

### 16.3 Capacity planning checklist

1. **Vector count** — documents × chunks per doc × (versions retained? deleted-but-uncompacted?).
2. **Dimension** — from the embedding model; fixed at schema time.
3. **Replication factor** — reads per query, HA needs; each replica multiplies RAM.
4. **Index parameters** — M, efConstruction/efSearch, quantization choice — all change the memory constant.
5. **Payload/metadata size** — filters live on payloads; payload indexes (Qdrant, Elasticsearch) add their own memory.
6. **Growth curve** — corpus grows; plan for +12–24 months, and for re-indexing events (§16.5).
7. **Shard layout** — shard count × per-shard RAM vs. node RAM; keep per-shard HNSW in RAM.

### 16.4 High availability

- **Replicas** — primary-secondary or Raft-based (Qdrant, Milvus use Raft for coordination; managed tiers abstract it). Read replicas scale QPS; write replicas provide failover.
- **Failover** — leader election (Raft) or DNS/load-balancer cutover; test it: kill a node in a drill and measure recovery time (index reload from disk is the slow part — see §9.4).
- **The failure mode to plan for:** a node with 60 GB of HNSW takes minutes to rebuild/load. Design for *deliberate* unavailability windows, replica placement across AZs, and index-snapshot persistence so restarts don't mean full rebuilds.

### 16.5 Data lifecycle: the re-indexing problem

**Embedding model upgrades break everything silently.** A new model produces a different vector space; old and new vectors are not comparable, and queries embedded with the new model will not find old documents. The standard playbook (also in the [RAG Optimization Techniques guide](rag_optimization_techniques_guide.md)):

1. **Version your embeddings** — store `model_name + version` as payload on every vector; filter queries by it during migration.
2. **Dual-run during migration** — embed new data with the new model while old data is re-embedded in batches; serve both indexes (or both collections) behind a router until the cutover completes.
3. **Re-embed + re-index as a batch job** — backfill scripts with checkpointing; verify recall@k on the golden set before/after (§18).
4. **Compaction / deletion** — tombstoned vectors accumulate; schedule compaction windows (products differ: Qdrant optimizers, Milvus segment compaction) and use partition-per-lifecycle (drop old partitions wholesale) where the data model allows.
5. **TTL** — some products/clouds support time-to-live on vectors (session/agent-memory use cases — see [Beyond RAG](beyond_rag_guide.md)); otherwise expire via scheduled delete + compaction.

### 16.6 Security

Vector databases inherit standard database security, with AI-specific additions:

- **Encryption at rest** (volume/segment encryption) and **in transit** (TLS on gRPC/REST).
- **Access control** — API keys/roles per client; namespace/collection-level tenancy; for banks, integration with the corporate IAM (OIDC/LDAP) rather than per-product users.
- **Network isolation** — private subnets/VPC, no public endpoints; egress control for any managed service (data-residency implications — see [On-Prem LLM Deployment guide](../on_prem_llm_deployment_guide.md) for the same security analogies applied to LLM serving).
- **Audit logging** — query and admin activity logs, retained per policy.
- **The vector-specific surface:** the *retrieved content* flows into LLM prompts, so the retrieval layer is part of the prompt-injection attack surface; authorization must be enforced at retrieval time, not assumed from document-level ACLs (see §17.3 and the [LLM Development Risks & Security guide](../llm_development_risks_security_guide.md)).

### 16.7 Monitoring: the five metrics that matter

A vector database in production needs its own dashboard — the standard database metrics (CPU, memory, disk, connections) are necessary but not sufficient. Track:

1. **Recall@k drift** — sample production queries (or replay the golden set, §18.3) and measure recall against brute force periodically. Recall drift is the early-warning signal for: bad index parameters, degraded graph quality after heavy update churn, quantization loss, or a *silent embedding-model change* upstream (§16.5). Nothing else on this list catches that.
2. **Latency p99 (query)** — per-collection, under load; watch the *spread* between p50 and p99 (compaction interference shows up here, §9.8) and the correlation with RAM/disk pressure.
3. **QPS and concurrency** — per collection and per node/shard; trending tells you when to add replicas (read-bound) vs. shards (capacity-bound) (§9.6).
4. **Index size and memory** — raw vectors vs. index overhead vs. payload indexes; the §16.2 formula as a live number. Memory creep without vector growth = tombstones/compaction backlog.
5. **Ingest pipeline health** — flush latency (time from insert-ack to searchable, §9.8), compaction backlog, segment count, and re-index job progress. Ingest is where vector-DB incidents start.

Alert thresholds come from your SLOs (e.g., p99 > 50 ms for 10 min, recall drop > 2 points on the golden set, compaction backlog older than X). The non-obvious one to instrument from day one is **recall drift** — it is the metric unique to vector systems, and the one most teams discover they cannot measure only after an incident.

---

## 17. The Vector Database in a Banking Context

### 17.1 Data residency — self-host for sensitive data

Bank data (client information, trades, surveillance outputs) has residency and sovereignty constraints that most SaaS vector offerings cannot meet out of the box. The pattern:

- **Classify the corpus first.** Public/internal-document RAG (policies, product info) may be fine on a managed service in-region; client-data RAG almost always must be **self-hosted** (Qdrant/Milvus/pgvector on-prem or in the bank's private cloud region), or on a vendor's dedicated "Hybrid Cloud"/private-region offering with contractual data-handling terms.
- **The managed middle path:** vendor Hybrid Cloud (Pinecone Hybrid Cloud, Qdrant Hybrid Cloud) runs the vendor's engine inside your VPC/region — managed operations, your data boundary. Verify what the vendor can see, and get it in writing.
- The full self-hosting security playbook (network isolation, encryption, patching, IAM integration) mirrors the [On-Prem LLM Deployment guide](../on_prem_llm_deployment_guide.md) — the vector DB should sit inside the same hardened perimeter as the models.

### 17.2 PII in embeddings — the leak surface

**Embeddings can leak training/reference data.** Embedding models are trained on text; with the right techniques (inversion attacks, nearest-neighbor probing), vectors can be reverse-engineered toward the original text, and retrieval itself *reveals* which documents exist. For a bank, embeddings derived from client names, account references, or trade details are sensitive-by-derivation:

- **Redact/hash before embedding** — strip identifiers (names, account numbers, email addresses) or replace with placeholders *before* the embedding step; store the mapping outside the vector store. The [Agent Runtime Cache Design guide](agent_runtime_cache_design_guide.md) covers the hashing/redaction patterns in the cache context — the same discipline applies at the embedding boundary.
- **Treat the vector DB as a sensitive store** — access control, encryption, audit (§16.6) apply to the *vectors themselves*, not just the payloads.
- **Document the derivation** for GDPR/data-protection records: "embeddings are personal data when derived from personal data" is the conservative, defensible position.
- See the [LLM Development Risks & Security guide](../llm_development_risks_security_guide.md) for the fuller risk analysis of embeddings and retrieval.

### 17.3 Access control on retrieval

The retrieval layer is where per-user authorization must actually be enforced, because anything retrieved gets into the prompt:

- **Enforce at query time, not document time:** tenant/entitlement predicates must be part of every vector query (the §8.1 filtering problem, with a compliance edge). A post-filtered top-k that drops a restricted document is a *control failure*, not a recall statistic.
- **Patterns:** per-tenant collections/namespaces (simplest isolation), or global collection + mandatory `entitlement` filter (Qdrant/pgvector filtered ANN, Elasticsearch with security filters). Verify with red-team-style tests: user A must never retrieve user B's documents even under adversarial query crafting.
- **Keep the vector store's ACLs aligned with the source system of record** — the vector index is a derived store; its ACLs are a control that must be reviewed like any other (audit, quarterly review, automated drift checks).

### 17.4 Audit, backup/DR, and the regulated-data angle

- **Audit:** every retrieve operation, admin action, and schema change logged and retained per policy; model/embedding-version changes logged (they change retrieval behavior = model-risk-relevant).
- **Backup/DR:** snapshot + WAL backups on schedule; restore drills (including index rebuild time measurement — the recovery-time budget must fit the RTO); cross-region replication where the residency rules allow; *documented* RPO/RTO per vector store, like any other data store.
- **Model risk angle:** retrieval quality is part of the AI system's behavior — the bank's model-risk framework should cover the embedding model *and* the retrieval configuration (recall@k targets, versioning, change control). The [LLM Development Risks & Security guide](../llm_development_risks_security_guide.md) and [RAG Frameworks Comparison guide](rag_frameworks_comparison_guide.md) (§15) treat this in the RAG context; the vector store is where the retrieval configuration lives, so it inherits the governance.
- **The regulated-data bottom line:** a vector database holding bank data is a bank database. It gets the same residency, security, audit, backup, and change-control treatment as any other — the "it's just a search index" framing does not survive an audit.

---
## 18. Evaluation and Benchmarking

### 18.1 The benchmark dimensions

Whatever you evaluate, measure all five — they trade against each other:

| Dimension | Definition | Why it matters |
|---|---|---|
| **Recall@k** | Fraction of the true (brute-force) top-k the ANN index returns | The quality dial; end-to-end retrieval quality is what the business feels |
| **QPS / throughput** | Queries per second at a given concurrency | Capacity planning; the serverless/managed pricing driver |
| **Latency (p50/p99)** | Query latency under load | SLO compliance; RAG UX |
| **Index build time** | Time to build/load the index for N vectors | Ingest windows, re-index events (§16.5) |
| **Memory footprint** | RAM per vector (raw + index + overhead) | The cost line item; the thing that kills budgets first |

Plus, for production candidates: **filtered-query recall** (recall when a metadata predicate is applied — often much worse than unfiltered recall), **write throughput** (ingest speed), and **operational recovery time** (restart, rebuild, failover).

### 18.2 The standard benchmarks and datasets

- **ANN-Benchmarks** (github.com/erikbern/ann-benchmarks) — *the* standard ANN shootout: datasets (SIFT, GLOVE, Deep1M/10M, GIST, etc.) and the recall-vs-QPS plots every vendor cites. The caveats: it benchmarks *libraries and algorithms*, not full databases; datasets are old and generic (not your corpus, not your filters); results are single-node and often memory-only.
- **BEIR** — retrieval *quality* benchmark (zero-shot on 18 datasets): measures whether your embedding+retriever finds the right documents, independent of engine speed. The right tool for comparing embedding models and retrieval configs; recall@k on BEIR correlates with real RAG quality better than ANN speed plots do.
- **MTEB** — embedding *model* quality (70+ tasks): pick the embedding model here; the database inherits its ceiling. MTEB and BEIR results are the evidence for the §5.2 dimension/quality trade-off on generic data — but **generic leaderboards are a starting point, not a verdict** for your corpus.
- **Vendor benchmarks** — Qdrant, Weaviate, Milvus, Pinecone, and pgvector each publish benchmark posts; they are informative for architecture but **vendor benchmarks are biased** (their hardware, their parameters, their workload definitions, cherry-picked operating points). Never make a purchase decision on a vendor's chart.

### 18.3 The evaluation methodology: your data, your queries, your filters

The only benchmark that matters is the one that resembles production:

1. **Build a golden dataset** from *your* corpus: sample 100–1000 real queries (log them from day one), with human- or LLM-judged relevance labels for the top 5–20 documents each. This is the same golden-dataset discipline as RAG evaluation — see the [RAG Optimization Techniques guide](rag_optimization_techniques_guide.md).
2. **Generate query patterns that match production** — including the *filter mix*: tenant scoping, date ranges, entitlement predicates at realistic selectivity. Unfiltered benchmarks hide the post-filtering trap (§8.1).
3. **Run each candidate** through the same harness: embed with the same model, index with same parameters, measure recall@k, p99 latency, QPS, build time, and RAM at your target vector count. Include the *incumbent* (pgvector or FAISS, as applicable) — "new DB beats nothing" is not a comparison.
4. **Slide the recall dial** (efSearch/nprobe) and plot the latency-recall curve — compare candidates at *equal recall*, not equal default settings.
5. **Load-test** with production-shaped concurrency (mixed read/write, ingest during queries, a compaction cycle) — a benchmark that never writes while it reads misses the write-amplification reality.
6. **Repeat after the embedding-model choice** — the model and the engine interact; evaluate the stack, not the component.

**Practical test:** if the candidate cannot beat the incumbent (pgvector/FAISS) by a *material* margin on your golden set at your scale, the §15 logic applies: you do not need it yet. The evaluation's job is to produce that evidence, honestly, before the migration starts.

### 18.4 A minimal benchmark harness (sketch)

You do not need a vendor benchmark platform to make the call — a scripted harness over your golden set suffices:

```
for candidate in [pgvector, qdrant, weaviate, faiss]:      # incl. the incumbent
    deploy candidate (docker-compose or embedded)
    bulk-load N vectors from your corpus (batched, same embedder)
    build index with candidate defaults → record build time + peak RAM
    for recall_target in [0.90, 0.95, 0.98]:
        tune dial (efSearch / nprobe / oversearch) to hit recall_target on golden set
        measure p50/p99 latency, QPS at C concurrency levels (1, 10, 50)
    rerun top-50 golden queries with production-shaped filters (tenant/date/type)
        → record filtered recall@k  (the §8.1 trap detector)
    mixed workload: ingest 10% new vectors while queries run → record p99 shift
    restart/reload index → record recovery time (for §16.4 RTO planning)
report: table of (candidate, recall, latency p99, QPS, RAM, build, recovery, filtered-recall)
```

Keep the harness in the repo and re-run it on every significant change (embedding model upgrade, corpus milestone, candidate version bump) — evaluation is a practice, not a project. Two warnings: load-test on hardware comparable to production (RAM is the variable that flips results), and record every candidate's *parameters*, not just its defaults — "candidate X was slow" usually means "candidate X's default dial was conservative."

---

## 19. RAG Integration Recap

### 19.1 The vector DB in the RAG pipeline

```
ingest:  documents → parse → chunk → embed → index (batch, versioned)
query:   query → rewrite (optional) → embed → retrieve (ANN + filters + hybrid)
         → rerank → prompt → LLM → answer
```

The vector database occupies exactly one slot — the retrieval store — and its quality ceiling is set upstream (chunking, embeddings) while its *output* is refined downstream (reranking, fusion). The full optimization of both sides is in the [RAG Optimization Techniques guide](rag_optimization_techniques_guide.md); the framework-level comparison (which RAG stack wires which vector stores) is in the [RAG Frameworks Comparison guide](rag_frameworks_comparison_guide.md) (§8). This guide's contribution is the middle: *which retrieval store, and how to run it* — selection (§14), operations (§16), and evaluation (§18).

### 19.2 Hybrid retrieval in the pipeline

Where vector-only retrieval underperforms (exact terms: IDs, codes, names), hybrid vector + BM25 with fusion is the fix — implemented natively in Weaviate/Qdrant/Elasticsearch/OpenSearch or as a two-engine ensemble. Fusion choice (RRF vs weighted) and when hybrid actually helps are covered in the [RAG Frameworks Comparison guide](rag_frameworks_comparison_guide.md) and the optimization playbook; the database-level requirement is simply: **if your query mix needs hybrid, pick a store that supports it natively** (§14.3).

### 19.3 The vector DB as one component — not the system

RAG quality failures are usually *not* the vector database's fault (chunking, embeddings, reranking dominate), and performance failures usually are (index parameters, RAM, sharding). Keep that attribution discipline when diagnosing: profile the pipeline before blaming the store, and apply the §18 golden-set methodology to isolate where the loss is.

---

## 20. The Future of Vector Search (2026 and Beyond)

- **Vector search in everything.** Every database has shipped or is shipping vector support (the §4 consolidation). By 2026 the question "which database has vectors?" is obsolete; the question is "which *doesn't*, and does anyone care?" Expect vector to become as unremarkable as full-text search.
- **SQL-vector convergence.** pgvector, SQLite-vec, DuckDB, MySQL-adjacent efforts (and Oracle/SQL Server native vectors) are making "SELECT ... ORDER BY embedding" idiomatic SQL. The direction of travel: vector as a data type, ANN as a query-plan step, hybrid as a SQL operator (ES/SQL bridges, pgvector hybrid via tsvector).
- **Graph + vector (GraphRAG).** Entity graphs and embeddings combine — graph traversal for multi-hop relationships, vectors for semantic retrieval (see [Advanced RAG Techniques guide](advanced_rag_techniques_guide.md)). Expect graph databases to add vector indexes and vector DBs to add graph-ish capabilities; the convergence is real but young.
- **Learned / ML indexes.** Learned index structures (neural routing, learned partitioning) are showing up in research and some products (e.g., learned IVF in Milvus experiments); expect incremental adoption where build-time cost is amortized over many queries.
- **Sparse-dense hybrids everywhere (ColBERT-style).** Multi-vector late-interaction models (ColBERT) and learned sparse (SPLADE) blur the dense/sparse line; stores are adding multi-vector and late-interaction support (Vespa, Weaviate, Milvus) — see [Advanced RAG Techniques guide](advanced_rag_techniques_guide.md).
- **Consolidation continues.** The startup field has thinned since 2023; expect more acquisition/merger/feature-folding as incumbent databases absorb the remaining differentiators. Vendor risk is a real selection criterion: prefer OSS with a healthy community or a cloud with a credible roadmap.
- **Vector search for agents.** Agent memory (episodic stores, tool-result recall, long-horizon state) is a growing workload — see [Beyond RAG — Agent Memory](beyond_rag_guide.md); expect vector stores optimized for high-write, TTL'd, session-scoped patterns (Pinecone serverless namespaces, Qdrant TTL, Chroma/LanceDB embedded).
- **"Embedding as the new primary key."** As everything gets embedded (docs, users, transactions, events), similarity becomes a join operator and embeddings become identity — a trend that pushes vector search from "feature" toward "primitive," with implications for data modeling and governance (§17).
- **Cost and efficiency pressure.** Quantization, binary search, dimension reduction, and disk-based indexes (DiskANN-style) keep winning because memory is the cost driver; expect default-on compression and auto-tuning rather than manual parameter fiddling.

---

## 21. Glossary

| Term | Definition |
|---|---|
| **Embedding** | Fixed-length vector of floats produced by a model; semantically similar objects map to nearby vectors |
| **Vector** | The list of floats itself; the stored unit of data in a vector DB |
| **Dimension (D)** | Length of the vector; 384–3072 typical; drives memory, compute, and quality |
| **kNN** | k-nearest-neighbors: exact search for the k closest vectors to a query |
| **ANN** | Approximate nearest neighbor: trades small recall loss for orders-of-magnitude speedup |
| **HNSW** | Hierarchical Navigable Small World: multi-layer graph index; the default quality/speed choice |
| **M / efConstruction / efSearch** | HNSW parameters: connections per node; build-time search width; query-time search width |
| **IVF** | Inverted File: cluster-based index (nlist clusters, nprobe probed per query) |
| **PQ** | Product Quantization: lossy compression splitting vectors into quantized subvectors (up to ~96% memory reduction) |
| **Scalar / binary quantization** | int8 (4× compression) / 1-bit (32×) vector compression |
| **Cosine similarity** | Angle-based similarity; the standard for text embeddings |
| **L2 / Euclidean** | Straight-line distance; for raw/unnormalized vectors |
| **Dot product** | Magnitude-sensitive projection; fastest metric |
| **Recall@k** | Fraction of the true top-k returned by the ANN index |
| **QPS** | Queries per second; throughput measure |
| **p99** | 99th-percentile latency; the tail-latency SLO metric |
| **Metadata filter** | Predicate on payload fields (tenant, date, type) combined with vector search |
| **Filtered ANN** | Index-aware filtering (predicates inside index traversal) vs. pre/post-filtering |
| **Hybrid search** | Dense (vector) + sparse (BM25) retrieval with result fusion |
| **RRF** | Reciprocal rank fusion: rank-based hybrid result merge |
| **Re-embedding** | Rebuilding the corpus under a new embedding model (breaks comparability with old vectors) |
| **WAL** | Write-ahead log: durability mechanism; writes logged before apply |
| **Snapshot** | Point-in-time persisted state used for recovery/backup |
| **Raft** | Consensus protocol (leader election + log replication) used for distributed coordination |
| **Scatter-gather** | Distributed query pattern: broadcast to shards, merge partial results |
| **Tombstone** | Soft-delete marker; excluded from search until compaction reclaims it |
| **Golden dataset** | Labeled query-relevance set from your own corpus used for evaluation |
| **Oversearch** | Searching more than k per shard so the merged global top-k stays accurate |
| **ANN-Benchmarks / BEIR / MTEB** | Standard benchmarks: index speed/recall; retrieval quality; embedding quality |

---

*This guide is part of the LLM/AI Engineering series in the [research repo](https://github.com/jackliusr/research). Related: [RAG Frameworks Comparison](rag_frameworks_comparison_guide.md) · [RAG Optimization Techniques](rag_optimization_techniques_guide.md) · [Beyond RAG — Agent Memory](beyond_rag_guide.md) · [Advanced RAG Techniques](advanced_rag_techniques_guide.md) · [BM25 / FAISS / ScaNN Research](bm25_faiss_scann_research.md) · [On-Prem LLM Deployment](../on_prem_llm_deployment_guide.md) · [LLM Development Risks & Security](../llm_development_risks_security_guide.md).*
