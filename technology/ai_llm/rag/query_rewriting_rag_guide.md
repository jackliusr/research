# Query Rewriting in RAG — A Comprehensive Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [MLOps Lifecycle](mlops_lifecycle_frameworks_guide.md) · [LLM Guard Models](llm_guard_models_guide.md) · [Multi-Pass LLM Pipelines](multi_pass_llm_pipelines_guide.md) · [LLM Evaluation Frameworks](llm_evaluation_frameworks_guide.md)
> **Last Updated:** July 2026

---

## Table of Contents

1. [What Is Query Rewriting in RAG?](#1-what-is-query-rewriting-in-rag)
2. [The Query Rewriting Problem](#2-the-query-rewriting-problem)
3. [Query Decomposition](#3-query-decomposition)
4. [Query Expansion](#4-query-expansion)
5. [Hypothetical Document Embeddings (HyDE)](#5-hypothetical-document-embeddings-hyde)
6. [LLM-Based Query Rewriting](#6-llm-based-query-rewriting)
7. [Step-Back Prompting](#7-step-back-prompting)
8. [Multi-Query Retrieval](#8-multi-query-retrieval)
9. [Query Transformation for Conversational RAG](#9-query-transformation-for-conversational-rag)
10. [Format Conversion and Clarification](#10-format-conversion-and-clarification)
11. [Embedding-Based Approaches](#11-embedding-based-approaches)
12. [LLM-Based Rewriting — Strategies](#12-llm-based-rewriting--strategies)
13. [Conversational RAG — Deep Dive](#13-conversational-rag--deep-dive)
14. [Multi-Modal Query Rewriting](#14-multi-modal-query-rewriting)
15. [Rewriting vs Routing vs Decomposition](#15-rewriting-vs-routing-vs-decomposition)
16. [Implementation with Frameworks](#16-implementation-with-frameworks)
17. [Evaluation](#17-evaluation)
18. [Rewriting vs Other RAG Optimizations](#18-rewriting-vs-other-rag-optimizations)
19. [Enterprise / Banking](#19-enterprise--banking)
20. [Best Practices](#20-best-practices)
21. [Techniques Comparison Table](#21-techniques-comparison-table)
22. [Quick-Reference Selection Guide](#22-quick-reference-selection-guide)
23. [Conclusion](#23-conclusion)

---

## 1. What Is Query Rewriting in RAG?

Query rewriting transforms a user's raw query into one or more optimised queries before retrieval. The goal: get better documents back so the LLM generates better answers.

**Why it matters:** Users write natural language — short, ambiguous, conversational, and context-dependent. Retrieval systems (vector stores, BM25, hybrid) work best with precise, self-contained, information-dense queries. Rewriting bridges this gap.

**Core insight:** Queries are interrogative, short, and abstract. Documents are declarative, long, and concrete. They occupy different regions in embedding space. A well-rewritten query moves closer to document representations, dramatically improving semantic similarity.

---

## 2. The Query Rewriting Problem

**Query length mismatch:** User queries average 2–8 words versus document chunks of 200–1000+ words. A query like "capital requirements" must match docs discussing "Basel III CET1 requirements, capital conservation buffers, countercyclical buffers, and leverage ratios."

**Ambiguity categories:**

| Type | Example | Problem |
|------|---------|--------|
| Pronoun references | "What was its Q3 revenue?" | Needs entity resolution |
| Vague terms | "How are they handling risk?" | Underspecified subject & object |
| Multi-intent | "Capital requirements and liquidity ratios?" | Two questions bundled |
| Domain jargon | "BCBS 239 DQ requirements?" | Assumes domain knowledge |
| Misspellings | "capitol adequacy ratio" | Embedding degradation |
| Overly broad | "Tell me about AI" | No retrieval focus |
| Conversational elision | "And the competition?" | No context without history |

**The NLP-to-search gap:**
```
User: "What about their numbers last quarter?"
Needs: "Apple Inc. Q3 2024 financial results revenue net income"
Index: "Apple reports Q3 2024 revenue of $85.8B, net income of $21.4B"
```

**Cascading cost of bad retrieval:** hallucinated answers → vague responses → user retries → trust erosion. Fixing the query at input prevents all downstream failures.

---

## 3. Query Decomposition

Breaks complex, multi-faceted queries into simpler sub-queries, each targeting one concept.

**Example:**
```
Original: "What are the capital requirements and liquidity ratios for Basel III?"
Sub-queries: ["What are Basel III capital requirements?", "What are Basel III liquidity ratios?"]
```

**Pattern:** LLM receives query + decomposition instruction → outputs N sub-queries (2–5) → parallel retrieve top-K per sub-query → merge (deduplicate, RRF) → pass merged context + original query to LLM.

**When to use:** Queries with "and"/"as well as"/"also"; comparative queries; multi-hop queries; enterprise queries bundling regulatory + operational + financial aspects.

**Trade-offs:** +Captures all facets, parallelisable. −N× retrieval calls, potential sub-query overlap.

---

## 4. Query Expansion

Augments the query with synonyms, related concepts, and domain terms to increase recall.

**Example:** `"loan default"` → `"loan default credit risk non-performing loan delinquency NPL mortgage default"`

**Expansion sources:**
- **Thesaurus:** "default" → "delinquency, non-payment" — broad, low cost
- **Domain ontologies:** "loan" → "credit facility, financing" — high precision
- **LLM-generated:** prompt-based generation — contextually relevant
- **Query logs:** past queries retrieving same docs — evidence-based

**Methods:** String expansion (`" ".join(synonyms[t] for t in query.split())`), LLM expansion (prompt-based), weighted expansion (lower BM25 weight for expanded terms).

**When to use:** Short queries (1–3 words), non-standard vocabulary, legal/medical/regulatory domains. Complements BM25 especially well.

**Trade-offs:** +High recall boost, low cost. −May introduce noise, needs quality synonym lists.

---

## 5. Hypothetical Document Embeddings (HyDE)

Uses an LLM to generate a hypothetical "ideal document" that would answer the query, embeds *that* document, and uses its embedding for retrieval — not the original query's. The generated document (declarative) sits closer to real documents in embedding space than the query (interrogative).

**How it works:**
```
Query → "Write a paragraph answering: {query}" → Embed that paragraph → Retrieve with that embedding
```

**Why it works:** Bridges query-document style mismatch. Captures implicit entities. Zero-shot, no training needed.

**Implementation:**
```python
def hyde_retrieve(query, llm, embedder, vector_store, k=5):
    hypo_doc = llm.generate(f"Write a paragraph answering: {query}")
    hypo_emb = embedder.embed(hypo_doc)
    return vector_store.similarity_search_by_vector(hypo_emb, k=k)
```

**Limitations:** Hallucinated facts (ok — only used for retrieval). Adds LLM + embedding call per query. Quality depends on LLM domain knowledge.

**When to use:** Short or poorly specified queries; high query-document style mismatch; fallback when zero-shot retrieval is poor.

---

## 6. LLM-Based Query Rewriting

The most direct approach: ask an LLM to rewrite the query for better retrieval.

**Example rewrites:**

| Original | Rewritten |
|----------|-----------|
| "How do they compare?" | "Comparative analysis of [A] vs [B] on financial performance metrics" |
| "BCBS 239 stuff" | "BCBS 239 compliance: risk data aggregation and reporting principles" |
| "tell me about risk" | "Overview of credit, market, and operational risk in banking" |

**Prompt design:** Include role instruction, task specification, constraints (max length, preserve intent), output format ("Return only rewritten query"), and optionally few-shot examples.

**Templates:**

*General:*
```
Rewrite for better document retrieval. Be specific, self-contained. Preserve intent.
Query: {query}
Rewritten:
```
*Conversational:* `History: {history}\nCurrent: {query}\nRewrite as standalone query.`
*Banking domain:* `Rewrite this banking/regulatory query using standard terminology.`

**Trade-offs:** +Flexible, domain-adaptable, easy to iterate. −Adds 200–500ms latency, needs careful prompt engineering, scales cost with volume.

---

## 7. Step-Back Prompting

Generates an abstract version of the query to retrieve broader context, then uses the original query for specific retrieval. Two-stage: big picture + details.

**How it works:**
```
Query: "What was model X's error rate on dataset Y?"
Step-back: "What are the benchmark results for model X?" → general docs
Original query: → specific dataset Y results
Combine both contexts
```

**Implementation:**
```python
def step_back(query, llm, vector_store):
    step_q = llm.generate(f"General high-level question for context: {query}")
    return merge_docs(vector_store.search(step_q, k=3),
                      vector_store.search(query, k=3))
```

**Banking example:** `"Pillar 2 capital for OCBC in 2025?"` → step-back: `"How are Pillar 2 requirements determined under MAS?"` → retrieves both framework + specific figure.

**When to use:** Overly specific queries needing broader context; technical queries; multi-step reasoning.

---

## 8. Multi-Query Retrieval

Generates several phrasings of the same query, retrieves for each, and merges via Reciprocal Rank Fusion (RRF). Hedges against vocabulary mismatch.

**Example:**
```
Original: "What are Basel III liquidity requirements?"
Generated: ["Basel III LCR requirements", "NSFR requirements Basel III",
            "Basel III liquidity risk rules", "Short/long-term liquidity ratios Basel III"]
```

**RRF merge:**
```python
def rrf(results_per_query, k=60):
    scores = {}
    for q_results in results_per_query:
        for rank, doc in enumerate(q_results):
            scores[doc.id] = scores.get(doc.id, 0) + 1 / (k + rank + 1)
    return sorted(scores.items(), key=lambda x: x[1], reverse=True)
```

**When to use:** High-stakes queries, ambiguous phrasing, known embedding model gaps. N× retrieval cost but maximises recall.

---

## 9. Query Transformation for Conversational RAG

Resolves pronouns, references, and omitted context from conversation history into standalone queries.

| Conversational | Standalone |
|----------------|-----------|
| "What was its profit margin?" | "Apple's Q3 2024 profit margin" |
| "What about the next quarter?" | "Apple's Q4 2024 revenue" |
| "Who was responsible?" | "Executive responsible for Project Phoenix breach" |
| "And DBS?" | "DBS Bank Q3 2024 financial results" |

**Implementation:**
```python
def conv_rewrite(query, history, llm):
    return llm.generate(f"History: {history}\nCurrent: {query}\nStandalone query:")
```

---

## 10. Format Conversion and Clarification

**Query-to-question:** Ensures well-formed question format for QA-oriented retrievers. `"Basel III capital requirements"` → `"What are Basel III capital requirements?"`

**Query-to-keywords:** Extracts key terms for BM25. `"What are differences between IFRS 9 and CECL?"` → `"IFRS 9 CECL expected credit loss comparison differences"`

**Query clarification:** When genuinely ambiguous, ask the user before rewriting.
```python
def needs_clarification(q):
    return len(q.split()) < 3 or any(t in q.lower() for t in ["it","they","that","this"])
```

| Situation | Approach |
|-----------|----------|
| Single entity, vague property | Rewrite with domain expansion |
| Multiple entities, no relation | Clarify which entity |
| Single pronoun, no referent | Clarify referent |
| Multiple interpretations | Clarify before rewriting |
| Well-formed query | No rewriting |

---

## 11. Embedding-Based Approaches

**HyDE** — covered in §5. Generate hypothetical document, embed it, use that embedding.

**Query2Doc** — extends HyDE: generates a full document snippet blending the original query with expanded context, used for both embedding and keyword retrieval.
```
"Basel III capital requirements" → "Basel III capital requirements CET1 Tier 1 Tier 2
capital conservation buffer countercyclical buffer leverage ratio RWA CAR"
```

**REPLUG** — adds retrieval-in-the-loop: first-pass retrieval with original → docs inform LLM rewrite → second retrieval with rewritten query.
```
Original → First retrieve (top-5) → Docs + Query → LLM rewrites → Second retrieve → Answer
```
The first pass shows the LLM corpus vocabulary, improving rewrite quality.

**Query-aware embedding fine-tuning:** Instead of rewriting text, train the query encoder to map poor queries closer to relevant documents. Approaches: cross-encoder fine-tuning on (bad_query, good_doc) pairs, adapter modules, learned instruction tokens.

**Instruction-based embedding:** Prepend task instructions at embedding time (no explicit rewrite). Models: Instructor, E5, BGE.
```
Without: "What is the capital conservation buffer?"
With: "Represent financial regulation query for doc retrieval: What is the capital conservation buffer?"
```
Lightweight but limited — cannot add missing terms or resolve pronouns.

---

## 12. LLM-Based Rewriting — Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Few-shot** | Provide 2–3 poor→good examples | Domain jargon, recurring patterns |
| **Zero-shot** | Task description only | Default starting point |
| **Task-specific** | Tailor rewrite per backend (vector/BM25/hybrid/SQL) | Multi-backend retrieval |
| **Chain-of-thought** | LLM reasons about intent before rewriting | Ambiguous queries (higher token cost) |
| **Context-aware** | Incorporate user profile, domain context, session state | Personalised retrieval |
| **Multi-step** | Broad rewrite → retrieve → refine → retrieve again | Complex information needs |
| **Cost-aware** | Route simple rewrites to cheap models (GPT-4o-mini, Llama 3B), complex to large models | Production cost optimisation |

**Task-specific examples:**
- **Vector:** Generate semantically rich, fully expanded query
- **BM25:** Extract distinctive terms, remove stop words, normalise
- **Hybrid:** Generate both semantic and keyword versions
- **SQL:** Convert natural language to query parameters

**Cost-aware model routing:**
| Query Complexity | Model | Relative Cost |
|-----------------|-------|---------------|
| Minor rephrasing | GPT-4o-mini / Claude Haiku / Llama 3B | 0.1× |
| Standard rewrite | GPT-4o / Claude Sonnet / Llama 8B | 1× |
| Complex reasoning | GPT-4o / Claude Opus / Llama 70B | 5× |

---

## 13. Conversational RAG — Deep Dive

**Reference types to resolve:**
| Type | Example | Resolution |
|------|---------|-----------|
| Pronoun | "What about *it*?" | Last mentioned entity |
| Demonstrative | "*That* approach" | Last mentioned approach |
| Ellipsis | "And the *deadlines*?" | Carry forward subject |
| Comparative | "How does *this compare*?" | Both comparison targets |
| Implicit | "*Also*, Basel IV?" | Continuation marker |

**Context window strategies:**
| Strategy | Description | Best For |
|----------|-------------|----------|
| Last N turns | Include last 2–5 turns | Short Q&A |
| Sliding window | Last N + always include first turn | Multi-step analysis |
| Summary-based | Running conversation summary | Long sessions |
| Entity tracking | Track entities + attributes | Entity-centric convos |
| Topic segmentation | Only current topic | Topic-switching sessions |

**Self-contained query generation** transforms context-dependent queries into standalone questions. History: "OCBC Q3 2024 results?" → "OCBC net profit SGD 1.98B". User: "What was their NIM?" → Rewrite: "DBS Bank net interest margin Q3 2024"

**Re-rewriting:** As conversation evolves, update earlier rewrites when user clarifies. `"Show me XYZ Corp"` → user: "I meant the Singapore office" → re-rewrite all affected queries with "Singapore office"

**Identity resolution:** Handle aliases ("the bank", "CACIB"), abbreviations ("Crédit Agricole CIB" ↔ "CACIB"), group membership ("the Big 4 banks"), role-based references ("the CEO"). Requires per-session entity registry.

---

## 14. Multi-Modal Query Rewriting

**Text-to-image:** Rewrite visual descriptors for image database retrieval. `"Org structure of credit risk dept"` → `"Organisational chart credit risk hierarchy structure diagram"`

**Image caption to text:** Generate caption from user-provided image, rewrite for doc retrieval. `[Risk dashboard screenshot]` → caption: "Risk dashboard VaR stress testing" → rewrite: "Risk management dashboard metrics VaR stress testing limit monitoring"

**Cross-modal:** Speech-to-text → rewrite for search; table queries → extract column names + values; diagram queries → extract entity names and relationships.

---

## 15. Rewriting vs Routing vs Decomposition

| Dimension | Rewriting | Routing | Decomposition |
|-----------|-----------|---------|---------------|
| **Action** | Transforms query text | Decides *where* to send | Splits into sub-queries |
| **Output** | Rewritten query(s) | Destination label | Multiple sub-queries |
| **When needed** | Query text suboptimal | Multiple data sources | Multiple intents |
| **Cost** | Low–medium (1 LLM call) | Low (classifier) | Medium (N calls) |

**Combined pipeline (most powerful):**
```
Query → Route (regulatory DB + compliance DB)
  → Per route: Decompose (sub-queries)
    → Per sub-query: Rewrite for target backend
      → Parallel retrieve → Source-weighted merge → Generate
```

---

## 16. Implementation with Frameworks

### LangChain
```python
# Multi-Query
retriever = MultiQueryRetriever.from_llm(
    retriever=base_retriever, llm=ChatOpenAI(model="gpt-4o-mini"))

# Ensemble
ensemble = EnsembleRetriever(
    retrievers=[vector_retriever, keyword_retriever], weights=[0.5, 0.5])

# Custom
chain = LLMChain(llm=llm, prompt=PromptTemplate(
    template="Rewrite for better retrieval: {query}", input_variables=["query"]))
```

### LlamaIndex
```python
# HyDE
hyde = HyDEQueryTransform(llm=llm, include_original=True)
query_bundle = hyde(user_query)

# Step decomposition
query_engine = MultiStepQueryEngine(
    query_engine=base_engine,
    query_transform=StepDecomposeQueryTransform(llm=llm))

# Custom transform
class BankingQueryTransform(BaseQueryTransform):
    def _run(self, qb, metadata):
        rewritten = llm.complete(f"Rewrite for regulatory retrieval: {qb.query_str}")
        return QueryBundle(rewritten.text)
```

### Cohere (Command-R)
Command-R / Command-R+ include built-in query rewriting optimised for RAG, multi-step tool use, and automatic citation generation.
```python
response = co.chat(
    message="What are BCBS 239 requirements?",
    connectors=[{"id": "web-search"}])
```

### DSPy
Programmatic optimisation of rewriting as a pipeline module, with automatic prompt optimisation:
```python
class RAGWithRewrite(dspy.Module):
    def __init__(self):
        self.rewriter = dspy.ChainOfThought("query -> rewritten_query")
        self.retrieve = dspy.Retrieve(k=3)
    def forward(self, query):
        rewritten = self.rewriter(query=query).rewritten_query
        return self.retrieve(rewritten)

optimizer = dspy.BootstrapFewShot(metric=validate_answer)
optimized = optimizer.compile(RAGWithRewrite(), trainset=trainset)
```

### Custom Python
```python
class QueryRewriter:
    def __init__(self, client, model="gpt-4o-mini"):
        self.client, self.model, self.cache = client, model, {}

    def rewrite(self, query, context="", strategy="default"):
        key = hashlib.md5(f"{query}|{context}".encode()).hexdigest()
        if key in self.cache: return self.cache[key]
        prompts = {
            "default": f"Rewrite for better retrieval: {query}",
            "conv": f"History: {context}\nRewrite standalone: {query}",
            "keywords": f"Extract keywords: {query}",
        }
        resp = self.client.chat.completions.create(
            model=self.model, temperature=0.3,
            messages=[{"role": "user", "content": prompts[strategy]}])
        rewritten = resp.choices[0].message.content.strip()
        self.cache[key] = rewritten
        return rewritten

    def multi_query(self, query, n=3):
        resp = self.client.chat.completions.create(
            model=self.model, temperature=0.7,
            messages=[{"role": "user", "content": f"Generate {n} queries for: {query}"}])
        return [q.strip("- ").strip() for q in
                resp.choices[0].message.content.split("\n") if q.strip()]
```

---

## 17. Evaluation

### Retrieval Quality Metrics

| Metric | What It Measures | Direction |
|--------|-----------------|-----------|
| Recall@K | Fraction of relevant docs in top-K | ↑ |
| Precision@K | Fraction of retrieved docs relevant | ↑ (balanced) |
| MRR | Rank of first relevant result | ↑ |
| NDCG@K | Position-weighted ranking quality | ↑ |

**Protocol:** Golden dataset of (query, relevant_docs) → baseline with original query → candidate with rewritten → compare ΔRecall, ΔPrecision, ΔMRR, ΔNDCG.

### Downstream Quality

| Metric | What It Measures | Tool |
|--------|-----------------|------|
| Answer accuracy | Factual correctness | RAGAS, DeepEval |
| Faithfulness | Grounded in retrieved docs | RAGAS |
| Completeness | All aspects addressed | RAGAS answer relevancy |
| User preference | A/B test pipelines | Online eval |

### Cost & Latency

| Strategy | Extra Calls | Latency | Recall Uplift |
|----------|------------|---------|---------------|
| No rewrite | 0 | 0ms | Baseline |
| Zero-shot | 1 LLM | ~200ms | +10–30% |
| Multi-query (3×) | 3 LLM + 3 retrieve | ~600ms | +20–40% |
| HyDE | 1 LLM + 1 embed | ~500ms | +15–35% |
| Conversational | 1 LLM | ~300ms | +30–50% |
| REPLUG | 2 LLM + 2 retrieve | ~1000ms | +25–45% |

### Production Monitoring
```python
def check_rewrite(original, rewritten, docs):
    orig_r = recall(original, docs)
    new_r = recall(rewritten, docs)
    if new_r < orig_r:
        log.warning(f"Rewrite regressed: {original} → {rewritten}")
        return original  # fallback to original
    return rewritten
```

---

## 18. Rewriting vs Other RAG Optimizations

| Technique | What It Fixes | Cost | Complementary? |
|-----------|--------------|------|---------------|
| **Chunking strategy** | Poor doc segmentation | Low | Yes — rewriting handles query side |
| **Embedding model** | Poor semantic representations | High | Yes — better embeddings amplify rewriting |
| **Reranking** | Poor ordering of retrieved docs | Medium | Yes — rewriting boosts recall, reranking boosts precision |
| **Hybrid search** | Single-backend limits | Low | **Highly complementary** — rewriting feeds both backends |
| **Metadata filtering** | Wrong doc subset | Low | Yes — rewriting can extract filter conditions |

**Investment priority (impact/effort):**
1. Hybrid search — highest ROI
2. Zero-shot rewriting — one LLM call, immediate gain
3. Reranking — cross-encoder precision
4. Chunking optimisation — semantic chunking
5. Multi-query — high-recall scenarios
6. Conversational rewrite — chat support
7. Fine-tuned embeddings — domain adaptation

**The optimal stack:**
```
Query → [Query Rewriting] → [Hybrid Search] → [Reranking] → [LLM Generation]
```
Each layer addresses a different failure mode: query-document gap → vocabulary mismatch → noise in top-K → final answer quality.

---

## 19. Enterprise / Banking

### Regulatory Query Rewriting
Regulations reference many related standards — naive expansion misses connections.

| Original | Enterprise Rewrite |
|----------|-------------------|
| "BCBS 239" | BCBS 239 + MAS 610 + risk data aggregation + risk reporting |
| "IFRS 9" | IFRS 9 + expected credit loss + ECL + impairment + staging |
| "GDPR" | GDPR + PDPA (Singapore) + data protection + cross-border data transfer |

### Multi-Entity Queries
`"Exposures to XYZ Corp"` → `"Credit exposures to XYZ Corporation including subsidiaries XYZ Capital Pte Ltd, XYZ International Ltd, ticker XYZ, industry: Financial Services - Banking"`
**Entity resolution pipeline:** Extract mentions → lookup canonical name in knowledge graph → include subsidiaries, tickers, LEI identifiers.

### Compliance-Sensitive Rewriting
- **PII stripping:** Remove customer names, NRIC numbers, account IDs
- **Access boundaries:** Stay within user's authorisation scope
- **Audit trail:** Log original + rewritten query + model + latency

### Rewriting for Different Sources
| Source | Goal |
|--------|------|
| Internal policies | Use internal doc coding: "Policy BCBS-239" |
| Regulatory databases | Use regulatory IDs: "Basel Committee Pub D303" |
| Knowledge base | FAQ-style: "Internal process for BCBS 239 gap analysis" |
| Market data | Include instrument IDs: "OCBC (SGX:O39)" |

### Audit Trail Schema
```json
{
  "session_id": "conv_7a3f2b", "user_id": "jsmith",
  "original_query": "What are BCBS 239 requirements?",
  "rewritten_query": "BCBS 239 risk data aggregation principles",
  "rewrite_strategy": "regulatory_expansion_v3",
  "llm_model": "gpt-4o-mini",
  "latency_ms": 340
}
```

---

## 20. Best Practices

### Do
- **Log everything** — original + rewritten pairs for debugging, eval, audit
- **Evaluate before deploying** — A/B test retrieval metrics
- **Start simple** — zero-shot before few-shot before HyDE before multi-query
- **Use cheap models** — GPT-4o-mini, Claude Haiku, Llama 3B suffice
- **Cache rewrites** — in-memory or Redis for identical/similar queries
- **Combine with multi-query** for high-stakes queries
- **Test edge cases** — 1-word, full paragraphs, pronouns, jargon, misspellings
- **Implement fallback** — revert to original if rewritten query underperforms
- **Monitor degradation** — track rate of rewrites performing worse than originals
- **Respect well-formed queries** — skip rewriting entirely

### Don't
- Don't rewrite every query — well-formed queries need no transformation
- Don't use expensive models for simple rewrites
- Don't rewrite in a black box — surface rewritten query for debugging
- Don't ignore token budgets — rewriting adds tokens to every invocation
- Don't hardcode prompts — version them like any other prompt
- Don't assume rewriting fixes all — it fixes query-side, not document-side issues

### Production Checklist
| # | Item |
|---|------|
| 1 | Rewriting prompt versioned and tracked |
| 2 | A/B test with/without rewriting |
| 3 | Fallback to original when rewrite doesn't improve |
| 4 | Cache for common queries |
| 5 | Monitoring: rewrite quality degradation alerts |
| 6 | Audit log: original + rewritten + model |
| 7 | Edge case test suite |
| 8 | Cost tracking: LLM calls and tokens |
| 9 | Model selection: small for simple, large for complex |
| 10 | Re-rewriting as conversation evolves |

---

## 21. Techniques Comparison Table

| Technique | What It Does | When to Use | Cost | Quality Impact | Complexity |
|-----------|-------------|-------------|------|---------------|------------|
| **Zero-shot LLM rewrite** | LLM rewrites from task description | Default starting point | Low (1 call) | +10–30% recall | Very low |
| **Few-shot LLM rewrite** | LLM rewrites from examples | Domain jargon, patterns | Low (1 call) | +15–35% | Low |
| **Query expansion (lexical)** | Add synonyms/terms | Short queries, BM25 | Negligible | +10–25% | Low |
| **Query expansion (LLM)** | LLM generates related terms | Unknown domain synonyms | Low (1 call) | +15–30% | Low |
| **HyDE** | Generate hypothetical doc, embed it | Short queries, style mismatch | Moderate | +15–35% | Medium |
| **Query decomposition** | Break into sub-queries | Multi-intent queries | Medium (N calls) | High for complex | Medium |
| **Multi-query retrieval** | N phrasings, merge via RRF | High-recall requirements | Med–High | +20–40% recall | Medium |
| **Step-back prompting** | Abstract query for broad context | Overly specific queries | Moderate (2 calls) | High for technical | Medium |
| **Conversational rewrite** | Resolve pronouns/history | Chat/conversation RAG | Low–Med | Critical for conv | Medium |
| **REPLUG** | Retrieve→rewrite→retrieve | Weak first-pass retrieval | High (2+2) | +25–45% | High |
| **CoT rewrite** | LLM reasons before rewriting | Ambiguous queries | Medium | High for edge cases | Low |
| **Query clarification** | Ask clarifying questions | Truly ambiguous | Low | Prevents wrong guesses | Medium |
| **Query-aware embedding** | Fine-tune query encoder | Production scale | Very high | Persistent improvement | Very high |
| **Instruction embedding** | Prepend task instruction | Multi-task retrieval | Negligible | Moderate | Low |

### Framework Comparison

| Framework | Key Modules | Best For |
|-----------|-------------|----------|
| LangChain | MultiQueryRetriever, EnsembleRetriever, LLMChain | General RAG, flexible |
| LlamaIndex | HyDEQueryTransform, StepDecomposeQueryTransform, MultiStepQueryEngine | Document-centric RAG |
| Cohere | Command-R built-in rewriter, tool-use | Enterprise, out-of-box |
| DSPy | ChainOfThought rewrite module, optimisers | Optimised pipelines |
| Custom | Any LLM + any vector store | Production, special requirements |

---

## 22. Quick-Reference Selection Guide

### By Query Type
| Type | Recommended Technique |
|------|---------------------|
| Very short (1–3 words) | HyDE + query expansion |
| Moderate (4–10 words) | Zero-shot LLM rewrite |
| Ambiguous / vague | Decomposition + clarification |
| Conversational / follow-up | Context-aware conversational rewrite |
| Domain-specific / jargon | Few-shot rewrite + domain expansion |
| Multi-intent ("and") | Query decomposition |
| Overly specific | Step-back prompting |
| Misspelled / noisy | LLM rewrite (built-in spell-check) |
| Multi-lingual | LLM rewrite in target language |

### By Retrieval Goal
| Goal | Primary | Secondary |
|------|---------|-----------|
| Improve recall | Multi-query retrieval | Query expansion |
| Improve precision | Task-specific rewriting | Query decomposition |
| Improve both | HyDE | Step-back prompting |
| Reduce latency | Zero-shot (cheap model) | Cache rewrites |
| Reduce cost | No rewrite (well-formed) | Lexical expansion only |
| Max accuracy | REPLUG + Multi-query + Reranking | HyDE + Step-back |

### By Budget
| Tier | Techniques | Est. Extra Cost/Query |
|------|-----------|----------------------|
| Free | No rewrite + hybrid search | $0 |
| Low | Zero-shot (GPT-4o-mini) + lexical expansion | ~$0.001 |
| Moderate | HyDE + multi-query (3×) | ~$0.005 |
| High | Multi-step + REPLUG + CoT | ~$0.02 |

### By Maturity Level
| Level | Techniques | Infrastructure |
|-------|-----------|---------------|
| MVP | Zero-shot rewrite | Single LLM call |
| V1 | + Fallback + cache | Redis cache |
| V2 | + Few-shot + multi-query + HyDE | Prompt registry, A/B testing |
| V3 | + Routing + decomposition + per-source rewrite | Full observability |
| V4 | All techniques + continuous optimisation | Automated eval, feedback loop |

---

## 23. Conclusion

### Key Takeaways
1. **Query rewriting is the highest-leverage RAG optimisation** — it fixes problems at input, before any retrieval happens.
2. **Start simple.** Zero-shot LLM rewriting with GPT-4o-mini/Claude Haiku/Llama 3B consistently improves retrieval by 10–30% with minimal latency and cost.
3. **Combine techniques.** Best results come from layering: rewrite → hybrid search → reranking. Each layer addresses a different failure mode.
4. **Evaluate everything.** Measure recall@K, MRR, NDCG before and after. Monitor for regression — prompts drift over time.
5. **Design for your domain.** Banking/enterprise needs regulatory expansion, entity resolution, and compliance-aware rewriting.
6. **Conversational RAG demands rewriting.** Without context-aware rewrite, multi-turn conversations degrade rapidly.
7. **Cache aggressively.** In enterprise, identical queries are common. Redis/in-memory caching eliminates most rewriting costs.

### Decision Framework
```
Is query well-formed and self-contained?
 ├── Yes → No rewriting needed
 └── No  →
      Ambiguous or multi-intent?
       ├── Ambiguous → Clarification or decomposition
       └── Multi-intent → Decomposition
      Conversational?
       ├── Yes → Context-aware conversational rewrite
       └── No  →
            Very short (1–3 words)?
             ├── Yes → HyDE + expansion
             └── No  → Zero-shot LLM rewrite
            Domain jargon?
             ├── Yes → Add few-shot examples
             └── No  → Deploy with caching
Monitor: Rewrites improving retrieval?
 ├── Yes → Iterate and optimise
 └── No  → Fallback to original, investigate
```

### Recommended Reading
- **HyDE:** "Precise Zero-Shot Dense Retrieval without Relevance Labels" (Gao et al., 2022)
- **Query2Doc:** "Query Expansion via Query-Document Generation" (Wang et al., 2023)
- **REPLUG:** "REPLUG: Retrieval-Augmented Black-Box Language Models" (Shi et al., 2023)
- **Step-back:** "Step-Back Prompting Enables Reasoning via Abstraction" (Zhou et al., 2023)
- **RAG-Fusion:** "RAG-Fusion: A New Take on Retrieval-Augmented Generation" (Rackspace, 2024)
- **RaFe:** "Ranking Feedback Improves Query Rewriting for RAG" (EMNLP 2024)
- **DSPy:** "DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines" (Khattab et al., 2023)
