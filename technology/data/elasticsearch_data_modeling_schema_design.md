# Elasticsearch Data Modeling & Schema Design — A Comprehensive Guide

> Mastering mappings, analysis, indexing strategy, and schema patterns for production Elasticsearch deployments.

---

## 1. Introduction

Elasticsearch is a distributed, JSON-based search and analytics engine built on Apache Lucene. Unlike a relational database, it is **schema-on-write** — every field is indexed per its **mapping** before becoming searchable. Getting the mapping right is the single most impactful decision for search relevance, query performance, and storage efficiency. This guide covers mapping fundamentals, text analysis, indexing strategy, data modeling patterns (nested vs join vs flattened), performance tuning, aggregations-friendly design, schema evolution, pitfalls, best practices, and a complete e-commerce example.

---

## 2. Mapping Fundamentals

### 2.1 Dynamic vs Explicit Mapping

**Dynamic mapping** infers field types from the first document seen. Convenient for prototyping but dangerous in production — a single malformed document can lock in the wrong type, and unknown fields can silently flood the mapping.

| `dynamic` Value | Behaviour |
|-----------------|-----------|
| `true` (default) | New fields auto-added via type inference |
| `runtime` | New fields as runtime fields — queryable, not indexed |
| `false` | New fields silently ignored |
| `strict` | Reject documents with unmapped fields |

**Explicit mapping** — define every field's type, analyzer, and sub-fields upfront. Use for all production indices. The setting goes on the root of `mappings`:

```json
{ "mappings": { "dynamic": "strict", "properties": { ... } } }
```

### 2.2 Major Data Types

**Core:** `text` (full-text, analyzed, no `doc_values` by default), `keyword` (exact match, `doc_values: true` by default), `long`/`integer`/`short`/`byte` (signed integers — choose the narrowest), `double`/`float`/`half_float`/`scaled_float` (floating point — `scaled_float` with `scaling_factor` is most storage-efficient for prices), `boolean`, `date` (ISO 8601 or epoch millis with `format`), `binary` (base64, stored only), `range` (`integer_range`, `float_range`, `date_range`, `ip_range` for continuous intervals).

**Complex:** `object` (default — sub-docs flattened into prefix fields, no cross-field querying within arrays), `nested` (each array object indexed as separate hidden Lucene doc, supports cross-field queries within elements), `join` (parent-child in same index with `has_child`/`has_parent` queries), `flattened` (entire JSON as opaque keyword leaf blob — no full-text, supports term queries and basic aggs).

**Specialised:** `ip` (CIDR queries), `version` (semver ordering), `percolator` (stored queries for alerting/reverse-search), `search_as_you_type` (auto-generates n-gram sub-fields for as-you-type completion), `completion` (prefix autocomplete via in-memory FST), `token_count` (token length for scoring/filtering), `dense_vector`/`sparse_vector` (semantic/kNN search), `rank_feature` (numeric feature for query-time boosting without reindexing), `murmur3` (hash for `cardinality` aggs), `annotated-text` (text with entity markers).

### 2.3 Key Mapping Parameters

| Parameter | Purpose | Applies To |
|-----------|---------|------------|
| `analyzer` | Analysis chain | `text` |
| `doc_values` | On-disk columnar store for aggs/sorting | Most types (default `true` except `text`) |
| `norms` | Scoring length normalisation | `text`, `keyword` — disable for filter-only fields |
| `index` | Whether field is searchable | All types |
| `index_options` | Scoring info stored: `docs`, `freqs`, `positions`, `offsets` | `text` |
| `eager_global_ordinals` | Pre-load ordinals for fast aggs | `keyword` |
| `coerce` | Auto-convert strings to numbers | Numeric types |
| `ignore_malformed` | Skip docs with bad field values | All |
| `fields` | Multi-fields — one value, multiple representations | Any type |
| `copy_to` | Aggregate field values into a virtual catch-all | All |
| `store` | Return field value independent of `_source` | All (default `false`) |

---

## 3. Text Analysis

### 3.1 Analyzer Pipeline

An analyzer is a three-stage pipeline:

1. **Character filters** — transform the raw character stream (strip HTML, replace patterns).
2. **Tokenizer** — split chars into tokens. Exactly one per analyzer.
3. **Token filters** — transform, add, or remove tokens (lowercase, stem, stop words, synonyms).

```
"Running foxes!" → [html_strip] → "Running foxes!"
                → [standard tokenizer] → ["Running", "foxes"]
                → [lowercase, stemmer] → ["run", "fox"]
```

### 3.2 Built-in Analyzers

`standard` (Unicode text segmentation + lowercase — general), `simple` (split on non-letters + lowercase), `whitespace` (whitespace split — machine tags), `stop` (simple + stop-word removal), `keyword` (no-op — exact match on text), `pattern` (regex-based), `language` analyzers (e.g., `english`, `french`, `arabic` — language-specific stemming + stop words), `fingerprint` (sort + deduplicate tokens — cluster detection).

### 3.3 Tokenizers

`standard` (Unicode Text Segmentation), `letter` (non-letter split), `lowercase` (letter + lowercase), `whitespace`, `uax_url_email` (preserves URLs/emails), `ngram`/`edge_ngram` (partial-word matching for autocomplete), `keyword` (no-op), `pattern` (regex), `path_hierarchy` (`/a/b/c` → `["/a", "/a/b", "/a/b/c"]`), `char_group` (custom character groups).

### 3.4 Token Filters

`lowercase`, `stop`, `stemmer`/`snowball` (language stemming), `ngram`/`edge_ngram` (partial tokens), `shingle` (word n-grams), `synonym`/`synonym_graph` (query expansion), `asciifolding` (accent normalization), `length` (min/max length), `truncate`, `elision` (`l'avion` → `avion`), `pattern_capture`/`pattern_replace` (regex rewriting), `remove_duplicates`, `unique`, `hunspell` (dictionary stemming).

### 3.5 Custom Analyzer Example

```json
{
  "settings": {
    "analysis": {
      "char_filter": { "html_strip_no_script": { "type": "html_strip" } },
      "filter": {
        "english_stop": { "type": "stop", "stopwords": "_english_" },
        "english_stemmer": { "type": "stemmer", "language": "light_english" },
        "autocomplete_ngram": { "type": "edge_ngram", "min_gram": 2, "max_gram": 10 }
      },
      "analyzer": {
        "my_fulltext": { "type": "custom", "char_filter": ["html_strip_no_script"],
          "tokenizer": "standard", "filter": ["lowercase", "english_stop", "english_stemmer"] },
        "my_autocomplete": { "type": "custom", "tokenizer": "standard",
          "filter": ["lowercase", "autocomplete_ngram"] }
      }
    }
  }
}
```

### 3.6 Search-Time vs Index-Time Analysis

Index-time analysis determines storage; search-time analysis (set via `search_analyzer`) processes the query. They must be compatible. Typical pattern — index with edge_ngram for autocomplete, search without:

```json
"title": { "type": "text", "analyzer": "my_autocomplete", "search_analyzer": "standard" }
```

### 3.7 Language-Specific Considerations

Never use the `standard` analyzer for non-English text. ES ships language analyzers (`english`, `french`, `german`, `arabic`, `chinese`, `japanese`, `korean`, etc.) with proper stemming and stop words. For CJK (Chinese/Japanese/Korean), use the `icu_analysis` plugin for correct Unicode segmentation.

---

## 4. Indexing Strategy

### 4.1 Index Templates

Templates apply mappings, settings, and aliases when indices are created, based on `index_patterns`:

```json
PUT _index_template/products_template
{
  "index_patterns": ["products-*"], "priority": 100,
  "template": {
    "settings": { "number_of_shards": 3, "number_of_replicas": 1 },
    "mappings": { "dynamic": "strict", "properties": {
      "title": { "type": "text", "analyzer": "english" },
      "price": { "type": "scaled_float", "scaling_factor": 100 },
      "created_at": { "type": "date" }
    } },
    "aliases": { "products": { "is_write_index": true } }
  }
}
```

**Component templates** compose reusable building blocks, combined via `composed_of`:

```json
PUT _component_template/products-base
{ "template": { "settings": { "number_of_shards": 3, "number_of_replicas": 1 },
    "mappings": { "_source": { "enabled": true }, "dynamic": "strict" } } }

PUT _index_template/products
{ "index_patterns": ["products-*"], "composed_of": ["products-base", "products-mappings"] }
```

### 4.2 Index Lifecycle Management (ILM)

ILM automates transitions through phases: **Hot** (write + search) → **Warm** (less frequent search, reduce resources) → **Cold** (rare search, searchable snapshots) → **Delete** (purge). Each phase triggers rollover, shrink, force-merge, or delete.

```json
PUT _ilm/policy/products_policy
{
  "policy": {
    "phases": {
      "hot":  { "min_age": "0ms",  "actions": { "rollover": { "max_size": "50gb", "max_age": "30d" }, "set_priority": { "priority": 100 } } },
      "warm": { "min_age": "30d",  "actions": { "forcemerge": { "max_num_segments": 1 }, "shrink": { "number_of_shards": 1 }, "set_priority": { "priority": 50 } } },
      "cold": { "min_age": "90d",  "actions": { "searchable_snapshot": { "snapshot_repository": "my_backup" }, "set_priority": { "priority": 0 } } },
      "delete": { "min_age": "365d", "actions": { "delete": {} } }
    }
  }
}
```

### 4.3 Index Aliases

Aliases provide indirection for zero-downtime reindexing. `is_write_index` controls which backing index receives new documents:

```json
POST /_aliases
{ "actions": [
    { "add": { "index": "products-20240701", "alias": "products", "is_write_index": true } },
    { "add": { "index": "products-20240601", "alias": "products" } }
] }
```

**Filtered aliases** restrict which documents an alias exposes:

```json
{ "add": { "index": "products", "alias": "active_products",
           "filter": { "term": { "status": "active" } } } }
```

### 4.4 Data Streams

Append-only time-series indices managed by ILM with auto-generated backing indices. Enforce `@timestamp` on every document. Prefer data streams over manual time-based patterns for logs, metrics, events:

```
PUT _data_stream/my-logs
```

---

## 5. Data Modeling Patterns

### 5.1 Denormalization (Default)

Elasticsearch is not normalised like SQL. The default and most performant approach is denormalization — duplicating related data into each document:

```json
{ "order_id": 1001, "customer_name": "Alice",
  "items": [{ "sku": "ABC", "qty": 2, "price": 12.99 }] }
```

**Pros:** Fastest query performance (single index, no joins), simplest routing. **Cons:** Data duplication, update anomalies.

### 5.2 Object Type — Flat Sub-Documents

Default for nested JSON. Arrays are flattened into independent field lists — cross-field queries **do not work**:

```json
{ "variant": [{ "color": "red", "size": "L" }, { "color": "blue", "size": "S" }] }
```

Query `variant.color=red AND variant.size=S` — matches incorrectly (Lucene sees `color:[red,blue]`, `size:[L,S]`).

### 5.3 Nested Type — Proper Array of Objects

Each object indexed as a hidden Lucene doc, preserving intra-element field relationships. Query with `nested` query:

```json
{
  "mappings": { "properties": {
    "variants": { "type": "nested", "properties": {
      "color": { "type": "keyword" }, "size": { "type": "keyword" },
      "price": { "type": "scaled_float", "scaling_factor": 100 }
    } }
  } }
}
```

```json
{ "query": { "nested": { "path": "variants", "query": { "bool": { "must": [
  { "term": { "variants.color": "red" } }, { "term": { "variants.size": "L" } }
] } } } } }
```

**Performance:** Each nested object = one hidden Lucene doc. Expensive for arrays >100 items. Nested aggregations need the `nested` aggregation path.

### 5.4 Join Type — Parent-Child in Same Index

Models 1:many with separate documents sharing the same index:

```json
{ "mappings": { "properties": {
    "relations": { "type": "join", "relations": { "product": ["review", "price_change"] } }
} } }
```

**Critical rules:** Children MUST use same `routing` as parent. Max 1 `join` field per index. Query via `has_child`/`has_parent`. Use only when child docs are independently queried and updated. For simpler 1:many, `nested` or denormalization is faster.

### 5.5 Flattened Type — Schema-Free JSON Blob

Indexes entire object as keyword leaves. Supports `term` queries and basic `terms`/`cardinality` aggs. No full-text search or `exists` queries:

```json
"attributes": { "type": "flattened" }
```

Ideal for custom product attributes, EAV patterns, or highly variable user-defined fields.

### 5.6 Decision Matrix

| Pattern | Use When | Avoid When |
|---------|----------|------------|
| Denormalization | Read performance critical, few updates | Write-heavy, shared data changes often |
| `nested` | Cross-field queries in arrays <100 items | Large arrays, complex updates |
| `join` | Children independently queried/updated | Simple 1:many that can be nested |
| `flattened` | Unknown/ever-changing schema | Full-text or range queries needed |

---

## 6. Performance Optimization

### 6.1 Shard Sizing

- **Target:** 10–50 GB per shard. Below 1 GB → coordinator overhead. Above 100 GB → merge overhead.
- **Count:** `total_data / target_shard_size`. Max ~20 shards per GB of heap.
- **Heap:** 50% system RAM (max 32 GB). Rest for OS page cache.
- **Cluster guard:** Configure `cluster.max_shards_per_node`.

### 6.2 Routing

Default: `shard = hash(_id) % num_shards`. Custom routing co-locates related docs on the same shard:

```json
PUT /products/_doc/123?routing=category_shoes { "name": "Running Shoes" }
```

**Pros:** Queries with routing hit fewer shards; join relationships require routing co-location. **Cons:** Skewed routing causes hot shards; must be specified on every request. `number_of_shards` is immutable after creation.

### 6.3 doc_values

On-disk columnar store for sorting, aggregations, scripting. Enabled by default for all types except `text`. **Disable on fields never used for aggs/sorting** to save disk. Immutable after data is indexed — set before first document.

### 6.4 Index Sorting & Compression

Sort documents within segments for faster range queries and better compression:

```json
PUT /logs
{ "settings": { "index.sort.field": "@timestamp", "index.sort.order": "desc" } }
```

Use `best_compression` codec for cold indices: `"index.codec": "best_compression"`.

### 6.5 Fielddata

In-heap structure for sorting/aggregations on `text`. **Expensive** — loads all unique tokens into heap. Never use on high-cardinality text fields. Use `.keyword` multi-fields instead.

### 6.6 Force-Merge

Reduces segment count to 1 on static (read-only) indices:

```json
POST /products-2024/_forcemerge?max_num_segments=1
```

---

## 7. Aggregations-Friendly Schema Design

### 7.1 Keyword Sub-Fields

Every `text` field should have a `.keyword` sub-field for aggs and sorting:

```json
"title": { "type": "text",
  "fields": { "keyword": { "type": "keyword", "ignore_above": 256 } } }
```

`ignore_above` prevents indexing values exceeding Lucene's limit (~32766 bytes).

### 7.2 Eager Global Ordinals

Global ordinals map terms to integers for fast aggregations. By default loaded lazily — causing a latency spike on the first agg request. Enable for high-cardinality fields:

```json
"category": { "type": "keyword", "eager_global_ordinals": true }
```

### 7.3 Histogram Type

Pre-bucketed data stored as paired `values` and `counts` arrays — avoids storing every data point:

```json
"response_times": { "type": "histogram", "values": [25, 50, 100], "counts": [100, 300, 200] }
```

Supports `percentiles` and `histogram` aggs directly.

### 7.4 Aggregation Performance Tips

- Use `composite` aggs instead of `terms` for deep pagination over high-cardinality buckets.
- `cardinality` with `precision_threshold` (default 3000, max 40000) — tune accuracy vs cost.
- `date_histogram` with `fixed_interval` for predictable width; `auto_date_histogram` for adaptive granularity.
- Filter aggressively before aggregating to reduce the doc set.
- `execution_hint`: `map` for <10k cardinality, `global_ordinals` for larger.

---

## 8. Reindexing & Schema Evolution

### 8.1 Reindex API

Primary mechanism for schema changes requiring data migration:

```json
POST /_reindex
{ "source": { "index": "products-v1" }, "dest": { "index": "products-v2" } }
```

**Zero-downtime pattern:** Create v2 → reindex → dual-write during catch-up → swap alias → drop v1.

### 8.2 Ingest Pipelines During Reindex

```json
PUT _ingest/pipeline/normalize
{ "processors": [
    { "convert": { "field": "price", "type": "float" } },
    { "remove": { "field": "legacy_price" } }
] }
POST /_reindex
{ "source": { "index": "products-v1" },
  "dest": { "index": "products-v2", "pipeline": "normalize" } }
```

### 8.3 Update By Query

In-place changes without full reindex — add fields, update values:

```json
POST /products/_update_by_query
{ "script": { "source": "if (ctx._source.tags == null) ctx._source.tags = ['new']" } }
```

### 8.4 Changes NOT Requiring Reindexing

Add a new field or multi-field, disable `doc_values` on a field with no data, change `ignore_above`/`null_value`/`coerce`.

### 8.5 Changes REQUIRING Reindexing

Change field type, change `analyzer` on indexed `text`, enable/disable `doc_values` with data present, change `norms`/`index_options` on a field with data.

### 8.6 Schema Versioning

Track schema versions in `_meta` for lifecycle management:

```json
PUT /products-v2/_mapping
{ "_meta": { "schema_version": 2, "last_migration": "2024-07-01" },
  "properties": { ... } }
```

### 8.7 Clone, Split, Shrink

- `_clone`: same shard count (backup/rebuild).
- `_split`: increase shards (requires `index.number_of_routing_shards`).
- `_shrink`: decrease shards (saves memory on cold indices).

---

## 9. Common Pitfalls

- **Mapping Explosion:** Dynamic mapping with unique field names creates hundreds of thousands of fields. **Fix:** `dynamic: "strict"` or `"false"`, use `flattened` for key-values.
- **Wide Documents:** Hundreds of fields degrade `_source` reconstruction. **Fix:** Keep <100 fields, use `_source.includes/excludes`, split by access pattern.
- **Oversharding / Undersharding:** 1 GB shards → coordinator overhead; 500 GB → merge overhead. **Fix:** 10–50 GB per shard, ILM rollover.
- **Join Overuse:** `has_child`/`has_parent` scans are expensive. **Fix:** Denormalise for 1:1/simple 1:many. Use `join` only for independently queried/updated children.
- **Nested Array Blowup:** 10,000 nested objects = 10,000 hidden Lucene docs. **Fix:** Cap nested arrays at 100–200; index larger arrays as separate docs with a parent ID.
- **Fielddata on Text:** Loads all tokens into heap — OOM risk. **Fix:** Use `.keyword` sub-fields.
- **Single Growing Index:** All data in one unbounded index degrades performance. **Fix:** Data streams + ILM rollover.
- **`_source` Bloat:** Storing large JSON inflates disk. **Fix:** `_source.includes/excludes`; disable `_source` for analytics-only indices.
- **No Scale Testing:** Schema working at 100 docs fails at 10M. **Fix:** Test with production-volume data.
- **Overusing `copy_to`:** Duplicates data on disk. **Fix:** Prefer `multi_match` queries with per-field boosting.

---

## 10. Best Practices

- **Design for queries first** — define top 5 query patterns before mappings. Unsearched? `"index": false`. Unaggregated? `"doc_values": false`.
- **Use explicit mappings** — `"dynamic": "strict"` in production. Version schemas in code.
- **Prefer narrow types** — `integer` > `long`, `short` > `integer`, `scaled_float` > `double`.
- **Use `ignore_above`** — `"tags": { "type": "keyword", "ignore_above": 256 }` prevents indexing values exceeding Lucene's max.
- **Disable norms on filter-only fields** — `"status": { "type": "keyword", "norms": false, "doc_values": true }`.
- **Normalize vs denormalize deliberately** — Denormalize: read-heavy, few updates, 1:few. Normalize: write-heavy, high-cardinality 1:many, independent lifecycles.
- **Monitor field count** — set `index.mapping.total_fields.limit` (default 1000, consider 200–500).
- **Use index sorting** — sort on `@timestamp` or `customer_id` for early termination. Immutable after creation.
- **Version your indices** — `_meta.schema_version`; use dates or semver in index names.
- **Benchmark before production** — test throughput, latency (p50/p95/p99), disk, heap at production scale.

---

## 11. Practical Example: E-Commerce Product Search Schema

### 11.1 Index Template with Full Mapping

```json
PUT _index_template/ecommerce-products
{
  "index_patterns": ["products-*"], "priority": 200,
  "template": {
    "settings": {
      "number_of_shards": 3, "number_of_replicas": 1, "refresh_interval": "30s",
      "index.sort.field": ["created_at"], "index.sort.order": ["desc"],
      "analysis": {
        "filter": {
          "autocomplete_edge":   { "type": "edge_ngram", "min_gram": 2, "max_gram": 20 },
          "english_stop":        { "type": "stop", "stopwords": "_english_" },
          "english_stemmer":     { "type": "stemmer", "language": "light_english" },
          "product_synonyms":    { "type": "synonym_graph", "synonyms": ["shoes, sneakers, trainers", "tv, television", "laptop, notebook"] }
        },
        "analyzer": {
          "product_search": { "type": "custom", "tokenizer": "standard", "filter": ["lowercase","english_stop","english_stemmer","product_synonyms"] },
          "product_autocomplete": { "type": "custom", "tokenizer": "standard", "filter": ["lowercase","autocomplete_edge"] }
        },
        "normalizer": { "lowercase_normalizer": { "type": "custom", "filter": ["lowercase","asciifolding"] } }
      }
    },
    "mappings": {
      "dynamic": "strict",
      "_meta": { "schema_version": 2, "last_updated": "2024-07-01" },
      "properties": {
        "product_id": { "type": "keyword", "norms": false },
        "sku": { "type": "keyword", "norms": false, "ignore_above": 64 },
        "title": { "type": "text", "analyzer": "product_search", "search_analyzer": "product_search",
          "fields": { "keyword": { "type": "keyword", "ignore_above": 256, "norms": false, "eager_global_ordinals": true },
                      "autocomplete": { "type": "text", "analyzer": "product_autocomplete", "search_analyzer": "standard" } } },
        "description": { "type": "text", "analyzer": "product_search", "copy_to": ["search_all"] },
        "category": { "type": "keyword", "norms": false, "eager_global_ordinals": true },
        "category_path": { "type": "keyword", "norms": false, "fields": { "tree": { "type": "text", "analyzer": "path_hierarchy" } } },
        "brand": { "type": "keyword", "norms": false, "eager_global_ordinals": true },
        "price": { "type": "scaled_float", "scaling_factor": 100 },
        "compare_at_price": { "type": "scaled_float", "scaling_factor": 100 },
        "price_range": { "type": "integer_range" },
        "currency": { "type": "keyword", "norms": false },
        "in_stock": { "type": "boolean", "norms": false },
        "stock_count": { "type": "integer", "norms": false },
        "rating": { "type": "float" },
        "review_count": { "type": "integer" },
        "tags": { "type": "keyword", "norms": false, "ignore_above": 100 },
        "status": { "type": "keyword", "norms": false },
        "created_at": { "type": "date", "format": "yyyy-MM-dd'T'HH:mm:ss.SSSZ||epoch_millis" },
        "updated_at": { "type": "date", "format": "yyyy-MM-dd'T'HH:mm:ss.SSSZ||epoch_millis" },
        "variants": { "type": "nested", "properties": {
          "variant_id": { "type": "keyword" },
          "color": { "type": "keyword", "norms": false, "fields": { "normalized": { "type": "keyword", "normalizer": "lowercase_normalizer" } } },
          "size": { "type": "keyword", "norms": false },
          "price": { "type": "scaled_float", "scaling_factor": 100 },
          "in_stock": { "type": "boolean" }
        } },
        "attributes": { "type": "flattened" },
        "search_all": { "type": "text", "analyzer": "product_search" },
        "suggest": { "type": "completion", "analyzer": "product_search", "search_analyzer": "standard",
          "contexts": [{ "name": "category", "type": "category", "path": "category" }] },
        "popularity": { "type": "rank_feature" },
        "sales_velocity": { "type": "rank_feature", "positive_score_impact": false },
        "embeddings": { "type": "dense_vector", "dims": 384, "similarity": "cosine",
          "index": true, "index_options": { "type": "hnsw", "m": 16, "ef_construction": 100 } }
      }
    }
  }
}
```

### 11.2 Design Decisions

Key choices: `scaled_float` for prices (10x+ storage reduction vs `double`), `nested` for variants (correct cross-field queries), `flattened` for attributes (variable schema, no full-text needed), `completion` with category context (typeahead scoped by category), `dense_vector` + HNSW (semantic search), `rank_feature` on popularity (query-time boosting without reindexing), `eager_global_ordinals` on category/brand (pre-loaded faceting ordinals), `index.sort.field: created_at` desc (recent-first early termination), and `copy_to: search_all` (catch-all keyword search).

### 11.3 Sample Faceted Search Query

```json
GET /products/_search
{
  "query": { "bool": { "must": [{ "match": { "search_all": "running shoes" } }],
    "filter": [
      { "term": { "category": "footwear" } },
      { "term": { "in_stock": true } },
      { "range": { "price": { "gte": 50, "lte": 200 } } },
      { "nested": { "path": "variants", "query": { "bool": { "must": [
        { "term": { "variants.color": "red" } }, { "term": { "variants.size": "L" } }
      ] } } } }
    ],
    "should": [
      { "rank_feature": { "field": "popularity" } },
      { "rank_feature": { "field": "sales_velocity" } }
    ]
  } },
  "aggs": {
    "by_category": { "terms": { "field": "category", "size": 20 } },
    "by_brand": { "terms": { "field": "brand", "size": 20 } },
    "price_histogram": { "histogram": { "field": "price", "interval": 25 } },
    "avg_rating": { "avg": { "field": "rating" } }
  },
  "suggest": { "product_suggest": { "prefix": "runni", "completion": {
    "field": "suggest", "size": 5, "contexts": { "category": ["footwear"] }
  } } },
  "knn": { "field": "embeddings", "k": 10, "num_candidates": 50,
    "query_vector": [0.1, 0.2, 0.3], "filter": { "term": { "category": "footwear" } } }
}
```

This blends lexical search (`match`), structured filters (`term`, `range`, `nested`), rank-feature boosting, term aggregations, histogram aggs, context-aware completion, and kNN vector search — all against the same schema.

---

## 12. Conclusion

Elasticsearch schema design is a deliberate, query-first discipline. Every choice — dynamic vs explicit, field types, analyzers, templates, ILM, nested vs join vs flattened — directly determines query latency, storage cost, and operational stability.

**Foundational rules:**
1. **Map explicitly** — disable dynamic mapping in production.
2. **Design for your top queries** — every mapping decision flows from query patterns.
3. **Choose the smallest viable type** — narrow types save disk, memory, and time.
4. **Separate text analysis from keyword exact-match** — use multi-fields.
5. **Plan for schema evolution** — version indices, use aliases, automate reindexing.
6. **Size shards deliberately** — 10-50 GB per shard, use ILM rollover.
7. **Test at scale** — a schema working at 100 docs usually breaks at 10M.

When applied — as demonstrated in the e-commerce schema — Elasticsearch delivers fast, relevant, and reliable search and analytics at any scale.

---

## References

- [Elasticsearch Mapping Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) · [Analysis Overview](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) · [Managing Relations](https://www.elastic.co/blog/managing-relations-in-elasticsearch) · [ILM Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) · [Tune for Speed](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html) · [Dense Vector](https://www.elastic.co/guide/en/elasticsearch/reference/current/dense-vector.html)
