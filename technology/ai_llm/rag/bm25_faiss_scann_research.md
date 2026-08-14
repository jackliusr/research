# BM25, FAISS, and ScaNN: A Comprehensive Research Report

> **Researched for:** Jack Liu Shurui (Singapore)
> **Repository:** https://github.com/jackliusr/research
> **Last Updated:** July 2026

---

## Table of Contents

1. [BM25 (Okapi BM25)](#bm25-okapi-bm25)
2. [FAISS (Facebook AI Similarity Search)](#faiss-facebook-ai-similarity-search)
3. [ScaNN (Scalable Nearest Neighbors)](#scann-scalable-nearest-neighbors)
4. [Comparison Table](#comparison-table)
5. [Synergies and Modern Pipelines](#synergies-and-modern-pipelines)
6. [Recommendations for Different Scenarios](#recommendations-for-different-scenarios)
7. [Practical Notes](#practical-notes)
8. [References and Data Sources](#references-and-data-sources)

---

## BM25 (Okapi BM25)

### Origins

BM25 (Best Match 25) emerged from the probabilistic retrieval framework developed by **Stephen Robertson**, **Karen Spärck Jones**, and colleagues at City University London and Cambridge:

- **1970s–1980s:** Probability Ranking Principle (PRP) and binary independence model laid the theoretical foundation.
- **1994:** The Okapi project produced the first practical BM25 implementation (Okapi Basic Search System).
- **2009:** Robertson & Zaragoza published the definitive survey — *"The Probabilistic Relevance Framework: BM25 and Beyond"* (FnTIR, Vol. 3, No. 4, pp. 333–389).

### Core Formula

```
BM25(D, Q) = Σⁿᵢ₌₁ IDF(qᵢ) · (tf(qᵢ, D) · (k₁ + 1)) / (tf(qᵢ, D) + k₁ · (1 − b + b · |D| / avgdl))
```

- **IDF(qᵢ)** = Inverse Document Frequency (`log((N − n(qᵢ) + 0.5) / (n(qᵢ) + 0.5) + 1)`)
- **tf(qᵢ, D)** = Raw term frequency in document D
- **|D|** = Document length (words/tokens)
- **avgdl** = Average document length across the corpus
- **k₁** = Term saturation parameter (controls how quickly TF impact plateaus) — Typical: 1.2–2.0
- **b** = Length normalization parameter (0 = none, 1 = full) — Typical: 0.6–0.9

### How It Works

BM25 is an evolution of TF-IDF with two critical improvements:

1. **Term Frequency Saturation:** Unlike raw TF-IDF where repeating a term 100× scores 100× more, BM25 saturates via `tf / (tf + k₁)`. This asymptotically approaches 1, preventing a single document from dominating by repeating a query term.

2. **Document Length Normalization:** The `b` parameter penalizes long documents that contain query terms simply because they contain more words total. Shorter, concentrated documents are prioritized.

### Variants

| Variant | Key Innovation | Source |
|---------|---------------|--------|
| **BM25F** | Field-aware weighting (title, body, anchor get distinct weights) | Robertson et al., 2004 |
| **BM25+** | Lower-bound TF via constant δ to avoid zero-scoring near-matches | Lv & Zhai, 2011 |
| **BM25L** | Log-based TF normalization for long documents | Lv & Zhai, 2011 |
| **BM25-adpt** | Learns k₁ per term automatically from collection statistics | Lv & Zhai, 2009 |

### Implementation Libraries

| Library | Language | Notes | Stars |
|---------|----------|-------|-------|
| **Elasticsearch / Lucene** | Java | Default similarity since Lucene 4.0; production standard | ~76k (ES) |
| **Apache Solr** | Java | Enterprise search on Lucene | ~2k |
| **rank-bm25** | Python | De facto standard Python impl; 5 variants (BM25, BM25L, BM25+, BM25-adpt) | ~1.3k |
| **bm25s** | Python | NumPy/Numba = 10–100× faster than rank-bm25 | ~2.2k |
| **Whoosh** | Python | Pure Python search; BM25F support | ~500 |
| **Pyserini** | Python | Anserini (Lucene) Python bindings | ~1.5k |
| **Terrier** | Java | IR platform from U. Glasgow | ~100 |

### Strengths

- **No training required** — works out-of-the-box with zero labelled data
- **Interpretable** — scores decompose per-term; easy to explain rankings
- **Fast** — sparse inverted index scores only docs containing query terms; sub-10ms on millions of docs
- **Memory efficient** — inverted index is 10–30% of raw corpus size
- **Robust** — handles noisy text, rare terms, and domain-specific vocabulary well
- **Production proven** — dominates web search, legal discovery, academic search

### Weaknesses

- **No semantic understanding** — "car" ≠ "automobile"; exact vocabulary match required
- **Bag-of-words** — word order, phrase structure, proximity ignored
- **Zero-shot lexical gap** — "canine nutrition" misses "dog food"
- **Static** — requires re-indexing for new documents

### Use Cases

- Web search engines (dominant production algorithm), legal e-discovery, patent search, QA retrieval stage, academic literature search (arXiv, PubMed), enterprise document search

### BM25 vs. Neural Dense Retrieval

BM25 still beats dense models in several regimes:

| Regime | Why BM25 Wins |
|--------|---------------|
| **Out-of-domain** | Dense models degrade on distribution shift; BM25 is immune |
| **Low-resource languages** | No training data needed; exact match works across all languages |
| **High vocabulary overlap** | Legal/medical/technical queries — BM25's exact match > embedding similarity |
| **Rare/entity terms** | "CRISPR-Cas9", patent numbers — embeddings may conflate; BM25 pinpoints |
| **Cold-start** | No embedding model or vector index needed |

On the **BEIR benchmark** (Thakur et al., 2021), BM25 achieves competitive or better nDCG@10 than many dense retrievers on 6 of 19 datasets — particularly on legal (Legal-TREC), biomedical (BioASQ), and news (TREC-COVID).

### BM25 in Hybrid/Modern Pipelines

BM25 is rarely used alone in SOTA systems — it serves as the **sparse leg** in hybrid retrieval:

- **ColBERT / ColBERTv2 / PLAID:** BM25 for first-stage candidate generation → late-interaction re-ranking
- **SPLADE:** Neural sparse retrieval that explicitly mimics BM25's weighting within a learned model
- **RRF (Reciprocal Rank Fusion):** Fuses BM25 + dense rankings with `score = Σ 1 / (k + rank)`, no training needed
- **HyDE:** BM25 on hypothetical generated documents supplements dense retrieval

---

## FAISS (Facebook AI Similarity Search)

### Origins

Developed by **Facebook AI Research (FAIR)** — primarily **Matthijs Douze**, **Jeff Johnson**, **Hervé Jégou**, and **Leon Bottou** — and **open-sourced February 2017**.

**Key Papers:**

| Paper | Year | Citation |
|-------|------|----------|
| *Billion-scale similarity search with GPUs* | 2019 | Johnson, Douze, Jégou (arXiv:1702.08734) |
| *The Faiss library* | 2024 | Douze et al. (arXiv:2401.08281) |

**GitHub Stats (as of July 2026):** ~39.1k stars, ~4.2k forks, MIT license, v1.14.3 (June 2026), ~180 contributors.

### Core Purpose

FAISS is a library for **efficient similarity search** and **clustering** of dense vectors at **billion-scale**. It provides 15+ index types, GPU acceleration, vector compression (PQ, scalar quantization), and auto-tuning utilities.

### Key Index Types

#### 1. IndexFlat — Brute Force (Exact)

`IndexFlatL2` (Euclidean) / `IndexFlatIP` (Inner Product)

- Compares query against every vector: O(N·D)
- Use for: datasets < 100K vectors, ground-truth benchmarks
- Memory: 4 bytes × D × N (float32)

#### 2. IVF (Inverted File) — Clustering-Based

`IndexIVFFlat(quantizer, d, nlist, nprobe)`

- K-means clusters the database (`nlist` clusters). At search time, probes nearest `nprobe` clusters.
- **nprobe** controls recall/speed tradeoff; **nlist** = number of Voronoi cells (typical: 10×√N to 4√N)
- Training required: yes (k-means on representative data)
- Use for: 100K–10M vectors, moderate memory, good recall

#### 3. HNSW — Graph-Based

`IndexHNSWFlat(d, M)`

- Multi-layer navigable graph: higher layers = sparse (long jumps), lower = dense (fine search)
- **M** = bi-directional links per node (16–64); higher M = better recall, more memory
- Best recall/speed among approximate methods — often Pareto-optimal on ANN benchmarks
- Memory intensive: ~3–5× raw data size
- Use for: <10M vectors, max performance, memory available

#### 4. PQ (Product Quantization) — Compression

`IndexIVFPQ(quantizer, d, nlist, M, nbits)`

- Splits D-dim vector into M sub-vectors, each quantized independently to `2^nbits` centroids
- Compression ratio: up to 64× (e.g., 1024-D float32 = 4096 bytes → 64 bytes)
- **M** = sub-quantizers, **nbits** = bits per code (typically 8)
- Use for: >10M vectors, memory-constrained

#### 5. IVF+PQ+OPQ — Production Combo

`OPQMatrix` + `IndexIVFPQ`: OPQ rotates vectors before PQ to align dimensions with the quantization grid, giving significantly better recall at the same compression ratio. The standard choice for 10M–1B+ vectors.

#### Index Family Summary

| Index | Exact? | Memory | Speed | Recall | Training | Best For |
|-------|--------|--------|-------|--------|----------|----------|
| IndexFlat | Yes | High | Slow | 1.0 | No | Ground truth, <100K |
| IndexIVFFlat | No | High | Fast | 0.8–0.99 | Yes | Mid-scale, fast |
| IndexHNSWFlat | No | Very High | Very Fast | 0.95–0.999 | No | <10M, max perf |
| IndexIVFPQ | No | Low | Medium | 0.7–0.95 | Yes | Large-scale, tight mem |
| IndexIVFPQ+OPQ | No | Low | Medium | 0.8–0.98 | Yes | Best comp/resolution |
| IndexScalarQuantizer | No | Low | Fast | 0.85–0.97 | No | Simple compression |
| IndexBinaryFlat | Yes (hamming) | Very Low | Very Fast | Good | No | Binary embeddings |

### GPU Support

FAISS provides CUDA-accelerated indexes (IndexFlat, IndexIVFFlat, GPU k-means). Multi-GPU sharding transparently distributes index across GPUs. Milestone: search 1B vectors on a single 4-GPU server in ~1ms/query.

### Python API (Minimal Example)

```python
import faiss, numpy as np
d = 768
xb = np.random.random((100000, d)).astype('float32')
index = faiss.IndexFlatL2(d); index.add(xb)
# IVF version
nlist = 100
index = faiss.IndexIVFFlat(faiss.IndexFlatL2(d), d, nlist, faiss.METRIC_L2)
index.train(xb); index.add(xb); index.nprobe = 10
D, I = index.search(np.random.random((10, d)).astype('float32'), k=5)
# GPU
gpu_index = faiss.index_cpu_to_gpu(faiss.StandardGpuResources(), 0, index)
```

### Newer Features (v1.14+)

- **MatrixID** — arbitrary 64-bit IDs; **Range search** — find vectors within a distance threshold
- **IDMap** — arbitrary ID ↔ internal ID mapping; **Prefetch** — async prefetch for lower batch latency
- **FD I/O** — read/write indexes via file descriptors; **Auto-tuning** — automatic nprobe/nlist sweeps

### Limitations

- HNSW memory: 3–5× raw vector size in RAM
- IVF/PQ/OPQ all require training data + k-means convergence
- Parameter tuning: nprobe, nlist, M, m, nbits all affect the recall/speed/memory triangle
- Static index: adding vectors post-indexing requires rebuilding IVF/PQ
- GPU/CPU API surfaces differ; not all CPU indexes have GPU equivalents

### Use Cases

- Semantic search, recommendation systems, RAG vector stores, large-scale clustering (GPU k-means), duplicate/near-duplicate detection, backing Chroma/Weaviate/Qdrant vector DBs

---

## ScaNN (Scalable Nearest Neighbors)

### Origins

ScaNN was developed by **Google Research** and **open-sourced in 2020**. It originated from Google's need for high-throughput Maximum Inner Product Search (MIPS) in production systems.

**Key Paper:**

| Paper | Year | Venue | Citation |
|-------|------|-------|----------|
| *Accelerating Large-Scale Inference with Anisotropic Vector Quantization* | 2020 | ICML 2020 | Guo, Sun, Lindgren et al. (arXiv:1908.10396) |
| *SOAR: Improved Indexing for Approximate Nearest Neighbor Search* | 2023 | NeurIPS 2023 | Sun, Simcha, Dopson, Guo, Kumar (arXiv:2404.00774) |

**GitHub Stats (as of July 2026):** ~37k stars (google-research monorepo), Apache 2.0, PyPI `scann`, Python 3.9–3.13, Linux only.

### Core Approach

ScaNN's key innovation is **anisotropic vector quantization** — a fundamentally different quantization loss compared to traditional PQ.

#### Traditional (Isotropic) PQ

Minimizes reconstruction error: `Loss = E[||x − x̂||²]`. Treats all residual directions equally — optimizes for average squared error, not ranking quality.

#### ScaNN's Anisotropic Quantization

Optimizes a **score-aware loss** designed for MIPS:

```
Loss = E[||⟨q, x⟩ − ⟨q, x̂⟩||²]   (for query distribution q)
```

This penalizes the **parallel component** (the residual direction aligned with the data vector) more heavily than the orthogonal component — because the parallel component directly affects inner product scores. The quantizer allocates more bits to preserving score-relevant directions.

### Tree-AH Hybrid Architecture

#### Stage 1: Tree Partitioning (Pruning)

- KD-tree / random projection tree partitions the space
- **num_leaves** — number of leaf partitions (typically √N to N/1000)
- Query probes a subset of leaves, pruning ~90–99% of candidates

#### Stage 2: Asymmetric Hashing (Scoring)

- Database vectors quantized with anisotropic quantizer
- Query vectors kept full-precision (asymmetric)
- **dimensions_per_block** — quantization granularity (2–8)
- Precomputed lookup tables for fast inner product scoring
- **reorder_top_k** — top candidates re-scored with exact vectors

### Key Parameters

| Parameter | Typical Range | Description |
|-----------|--------------|-------------|
| **num_leaves** | 1,000–100,000 | Tree leaf count; finer = better pruning, more memory |
| **reorder_top_k** | 100–10,000 | Exact re-scoring budget; higher = better recall |
| **dimensions_per_block** | 2–8 | Quantization granularity; lower = finer, more accurate |
| **distance** | "dot_product" / "squared_l2" | Search distance function |

### Performance

ScaNN claims **~2× faster than FAISS** at equal recall on Google's internal benchmarks for MIPS. On public ANN-Benchmarks, ScaNN is consistently on or near the Pareto frontier for glove-100-angular (dot-product) and competitive on sift-128-euclidean.

| Metric | ScaNN (Tree-AH) | FAISS (IVF+PQ) | FAISS (HNSW) |
|--------|-----------------|----------------|--------------|
| QPS (glove-100, R@0.9) | ~10,000+ | ~5,000 | ~8,000 |
| Index build | Moderate (tree+training) | Moderate (k-means+PQ) | Fast (graph) |
| Memory efficiency | Good | Good | Poor |

### Python API

```python
import scann, numpy as np
dataset = np.random.random((100000, 768)).astype('float32')
searcher = scann.scann_ops.builder(dataset, 10, "dot_product") \
    .tree(num_leaves=2000, num_leaves_to_search=100) \
    .score_ah(2, anisotropic_quantization_threshold=0.2) \
    .reorder(100) \
    .build()
neighbors, distances = searcher.search(np.random.random(768).astype('float32'))
```

### Strengths

- Optimized for MIPS (the loss function is directly designed for dot-product search)
- Anisotropic quantization preserves score ordering, not just vector reconstruction
- Very high QPS on single CPU; optimized for production throughput
- Clean `reorder_top_k` mechanism for recall/speed tradeoff
- Production proven at Google (YouTube recommendations, Ads, document retrieval)

### Limitations

- Smaller community than FAISS; fewer index types (Tree-AH only, no HNSW/IVF binaries)
- Limited GPU support (primary optimization is CPU)
- Linux only (pre-built wheels)
- Training required for anisotropic quantizer
- Primarily optimized for cosine/MIPS — L2 supported but not the primary strength

### Use Cases

- Google YouTube recommendations, Google Ads, high-throughput MIPS serving (10K+ QPS), document retrieval in RAG pipelines, cross-modal retrieval

---

## Comparison Table

| Feature | BM25 | FAISS | ScaNN |
|---------|------|-------|-------|
| **Type** | Sparse (term-based) | Dense (vector) | Dense (vector) |
| **Search paradigm** | Exact term matching | Approximate nearest neighbor | Approximate nearest neighbor |
| **Semantic understanding** | No | Yes (with good embeddings) | Yes (with good embeddings) |
| **Training required** | No | Yes (IVF/PQ/quantization) | Yes (tree + quantizer) |
| **Speed (relative)** | Very fast (sparse) | Varies by index | Very fast (Tree-AH) |
| **Query latency (typical)** | <5ms (millions of docs) | 1–50ms | 1–20ms |
| **Index building** | None (inverted index) | Required (clustering/graph) | Required (tree + AH) |
| **Memory efficiency** | Excellent (10–30% corpus) | Poor–Moderate | Good |
| **Parameter tuning** | k₁, b | nprobe, nlist, M, m, nbits | num_leaves, reorder_top_k |
| **Open source license** | Various (BSD-like) | MIT | Apache 2.0 |
| **GitHub stars** | ~3.5k (Python libs) | ~39.1k | ~37k (google-research) |
| **Community size** | Massive (Elasticsearch ecosystem) | Very large (180+ contributors) | Growing (Google-backed) |
| **GPU support** | N/A | Full (CUDA) | Limited (CPU primary) |
| **Exact search possible** | Yes (by construction) | Yes (IndexFlat) | No |
| **Best distance metric** | N/A (bag-of-words) | L2, IP, Cosine | MIPS / Cosine |
| **Best for** | Exact term matching | General vector search | High-throughput MIPS |
| **Year open-sourced** | ~1994 (Okapi) / in Lucene ~2005 | 2017 | 2020 |

---

## Synergies and Modern Pipelines

### Hybrid Search (BM25 + Dense)

The dominant modern architecture fuses BM25's lexical precision with dense retrieval's semantic coverage:

```
Query → BM25 (sparse candidates) ──┐
     → FAISS/ScaNN (dense candidates) ─┤ → RRF Fusion → Re-ranker → LLM → Answer
```

**RRF (Reciprocal Rank Fusion):** `score(d) = Σ 1 / (k + rankᵢ(d))` — no training needed, parameter k typically 60 (Cormack et al., 2009).

### Two-Stage Retrieval (Fast + Slow)

| Stage | Model | Index | Recall | Latency |
|-------|-------|-------|--------|---------|
| **Stage 1** (Candidate gen) | BM25 / FAISS-IVF / ScaNN | Inverted / IVF / Tree-AH | 0.80–0.95 | 1–10ms |
| **Stage 2** (Re-ranking) | Cross-encoder / LLM | None (exact scoring) | 0.95–0.99+ | 10–200ms |

Spending compute re-ranking 100 candidates with a cross-encoder is far cheaper than scoring 1M with one.

### ColBERTv2 / PLAID — Late Interaction

Uses **residual compression** conceptually similar to FAISS's PQ: each token's embedding is compressed via clustered centroids + residual codebooks, enabling scalable token-level late interaction that combines BM25-style granularity with dense semantic understanding.

### SPLADE — Neural Sparse Retrieval

Explicitly bridges BM25 and dense retrieval: learns sparse term-weight vectors via BERT, output is a bag of weighted vocabulary terms directly indexable by Lucene. Achieves BM25's interpretability + lexical precision with neural generalization.

---

## Recommendations for Different Scenarios

| Scenario | Recommended Approach | Rationale |
|----------|---------------------|-----------|
| **Production search with exact term matching** | BM25 + Elasticsearch | Mature ecosystem, robust, interpretable |
| **Pure semantic search (<10M vectors)** | FAISS IVF+PQ or HNSW | Rich API, GPU support, flexible indexes |
| **Pure semantic search (10M–1B vectors)** | FAISS IVF+PQ+OPQ or ScaNN | Memory efficient, billion-scale |
| **High-throughput MIPS (100M+)** | ScaNN | 2× FAISS speed for inner product |
| **RAG pipeline** | FAISS (dense) + BM25 (sparse) + RRF | Best of both worlds, no training |
| **Low-latency serving** | ScaNN or FAISS HNSW (small nprobe) | Sub-10ms query latency |
| **Batch clustering (large-scale)** | FAISS GPU k-means | Unique GPU-accelerated clustering |
| **Resource-constrained (small RAM)** | BM25 or FAISS SQ/PQ | BM25 wins for text; PQ for dense |
| **Cold-start / no labeled data** | BM25 | Zero training, works immediately |

---

## Practical Notes

### Ecosystem Health

| Technology | Maintained? | Release Frequency | Community |
|------------|-------------|-------------------|-----------|
| **BM25 (Lucene/ES)** | ✅ Active (Lucene core) | Quarterly (ES) | Massive — Elasticsearch ubiquitous in enterprise |
| **rank-bm25** | ⚠️ Low activity | Rare (~1.3k stars) | Niche Python |
| **bm25s** | ✅ Active | Regular (~2.2k stars) | Growing — fast Python alternative |
| **FAISS** | ✅ Very active | Monthly (~39k stars) | 180+ contributors, prolific |
| **ScaNN** | ✅ Active | Stable releases | Google-backed, smaller |

### Key Takeaways

1. **BM25 is not obsolete.** It remains the most widely deployed retrieval algorithm in production — almost every search engine uses it. Dense retrieval complements, not replaces, BM25.

2. **FAISS is the Swiss Army knife** of vector search. It covers the widest range of index types, has GPU support, and dominates the vector database ecosystem. Choose FAISS for flexibility.

3. **ScaNN's anisotropic quantization is a genuine innovation.** Don't blindly use FAISS PQ for every MIPS problem — ScaNN's score-aware quantization often yields better recall at the same speed for inner product search.

4. **Hybrid BM25 + dense is modern best practice**, not either/or. The BEIR benchmark consistently shows hybrid beating either leg alone. RRF fusion is simple, effective, and requires no training.

5. **Learn the index design space** — there is no single "best" index. The right choice depends on: dataset size, dimensionality, required recall, latency budget, memory budget, GPU availability, and throughput requirements.

6. **FAISS integrates with PyTorch** — tensors can share CUDA memory, making it trivial to build end-to-end deep learning + retrieval pipelines.

7. **ScaNN is worth evaluating for high-throughput production** — especially if you serve 1000+ QPS on a single machine using cosine/inner product similarity.

---

## References and Data Sources

### BM25

1. **Robertson, S. & Zaragoza, H.** (2009). *"The Probabilistic Relevance Framework: BM25 and Beyond."* FnTIR, 3(4), 333–389. DOI: 10.1561/1500000019. [PDF](https://www.staff.city.ac.uk/~sbrp622/papers/foundations_bm25_review.pdf)
2. **Robertson, S., Zaragoza, H., & Taylor, M.** (2004). *"Simple BM25 extension to multiple weighted fields."* CIKM 2004. (BM25F)
3. **Lv, Y. & Zhai, C.** (2011). *"When documents are very long, BM25 fails!"* SIGIR 2011. (BM25+/BM25L)
4. **Thakur, N. et al.** (2021). *"BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models."* NeurIPS 2021 Datasets & Benchmarks.

### FAISS

5. **Johnson, J., Douze, M., & Jégou, H.** (2019). *"Billion-scale similarity search with GPUs."* IEEE TBD, 7(3), 535–547. arXiv:1702.08734.
6. **Douze, M. et al.** (2024). *"The Faiss library."* arXiv:2401.08281.
7. **Jégou, H., Douze, M., & Schmid, C.** (2011). *"Product Quantization for Nearest Neighbor Search."* IEEE TPAMI, 33(1), 117–128.
8. **FAISS GitHub:** [facebookresearch/faiss](https://github.com/facebookresearch/faiss) — 39.1k stars, MIT, v1.14.3 (Jun 2026).

### ScaNN

9. **Guo, R., Sun, P., Lindgren, E., Geng, Q., Simcha, D., Chern, F., & Kumar, S.** (2020). *"Accelerating Large-Scale Inference with Anisotropic Vector Quantization."* ICML 2020. arXiv:1908.10396.
10. **Sun, P. et al.** (2023). *"SOAR: Improved Indexing for Approximate Nearest Neighbor Search."* NeurIPS 2023. arXiv:2404.00774.
11. **ScaNN (google-research):** [google-research/scann](https://github.com/google-research/google-research/tree/master/scann) — Apache 2.0, PyPI `scann`.

### Benchmarks

12. **ANN-Benchmarks:** [ann-benchmarks.com](https://ann-benchmarks.com/) — Standard ANN algorithm benchmark across SIFT, GLOVE, DEEP datasets.
13. **BEIR Benchmark:** [beir-cellar.github.io](https://beir-cellar.github.io/) — Zero-shot IR evaluation across 19 datasets.
14. **big-ann-benchmarks:** [big-ann-benchmarks.com](http://big-ann-benchmarks.com/) — Billion-scale ANN benchmark extension.

---

*Research compiled for Jack Liu Shurui. For questions or corrections, please open an issue or PR at https://github.com/jackliusr/research.*
