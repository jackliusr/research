# Large-Scale Web Data Acquisition — the Harvest at Scale

Jack Liu Shurui, Solution Architect

> **Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank
> **Topic:** Large-Scale Web Data Acquisition — the engineering of web crawling, scraping and data collection at scale; the URL frontier, politeness and deduplication; the end-to-end pipeline architecture; the verified tooling landscape; the legal and ethical boundaries; and the Cymbal Bank market-data and regulatory-data worked example
> **Audience:** Solution Architects, Data Platform Engineers, Data Engineering Leads, Compliance and Legal Technology Teams, Banking IT Decision-Makers
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Version:** 1.0 — August 2026

---

# Table of Contents

1. [Executive Summary — The Harvest at Scale](#1-executive-summary--the-harvest-at-scale)
2. [Fundamentals — Crawling vs Scraping vs Fetching](#2-fundamentals--crawling-vs-scraping-vs-fetching)
3. [The URL Frontier — Prioritisation, Politeness, Freshness](#3-the-url-frontier--prioritisation-politeness-freshness)
4. [Politeness Policies — robots.txt, Rate Limits, Retries](#4-politeness-policies--robotstxt-rate-limits-retries)
5. [Deduplication — URL Normalisation, Hashing, Bloom Filters](#5-deduplication--url-normalisation-hashing-bloom-filters)
6. [Pipeline Architecture — From Fetch to Monitor](#6-pipeline-architecture--from-fetch-to-monitor)
7. [Tooling — The Verified Landscape](#7-tooling--the-verified-landscape)
8. [The Anti-Bot Arms Race — An Ethical Boundary](#8-the-anti-bot-arms-race--an-ethical-boundary)
9. [Legal and Ethical Boundaries](#9-legal-and-ethical-boundaries)
10. [Large-Scale Use Cases — Search, Prices, News, LLM Corpora](#10-large-scale-use-cases--search-prices-news-llm-corpora)
11. [Cymbal Bank Worked Example — Market and Regulatory Data](#11-cymbal-bank-worked-example--market-and-regulatory-data)
12. [Claims Audit — Verified, Flagged, Unverified](#12-claims-audit--verified-flagged-unverified)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)

---

## 1. Executive Summary — The Harvest at Scale

### Purpose of This Guide

This guide is the repository's **platform deep-dive on large-scale web data acquisition**: the engineering discipline of collecting data from the open web — crawling, scraping, fetching — at a volume and cadence that make it a production data platform rather than a collection of scripts. It covers the four layers a solution architect must design for:

- **The fundamentals** — what crawling, scraping and fetching are, how they differ, and the vocabulary of the domain (Section 2).
- **The core machinery** — the **URL frontier**, **politeness policies** and **deduplication**, the three subsystems that decide *what* to fetch, *how fast*, and *whether we have seen it before* (Sections 3–5).
- **The pipeline** — fetch → parse → extract → transform → store → schedule → monitor, with the queue at the centre, the orchestration layer, the storage layer and the monitoring layer (Section 6).
- **The boundaries** — the verified tooling landscape, the anti-bot arms race, the legal and ethical limits (CFAA, hiQ v. LinkedIn, GDPR, PDPA, copyright), and the large-scale use cases that made this discipline famous: search indexes, price monitoring, news aggregation and LLM training corpora (Sections 7–10).

Section 11 is the **Cymbal Bank worked example**: a market-data and regulatory-data acquisition pipeline for a Singapore-headquartered bank, sourcing public exchange data, Monetary Authority of Singapore (MAS) publications and open-data portals, with a politeness and compliance posture, freshness SLAs, and a storage design that feeds the firm's market-data distribution layer.

### The One-Paragraph Answer

Large-scale web data acquisition is the discipline of turning the web into a **governed data source**. A crawler discovers URLs, a politeness layer throttles requests per host, a deduplication layer makes sure nothing is fetched twice, a fetch/parse/extract pipeline turns HTML into records, and a storage layer keeps both the raw bytes (WARC) and the cleaned, columnar, queryable forms (Parquet). Around that core sit a queue that absorbs load spikes, an orchestrator that runs it on a calendar, and a monitoring stack that watches success rates, crawl health and politeness compliance. The whole machine is constrained by law and ethics: robots.txt norms (now an IETF standard, RFC 9309), terms of service, the CFAA and the case law built around it, data-protection regimes such as the GDPR and Singapore's PDPA, and copyright — a set of boundaries that every acquisition design must treat as first-class requirements, not afterthoughts.

### Integrity Convention

Following repository convention, this guide marks every verifiable claim: ✅ = verified this pass against a primary or cited source (URL in Section 12); ⚠ = flagged, approximate, or well-documented industry knowledge not re-verified live; ❌ = a claim checked and found incorrect, with the correction stated. No dates, citations or statistics were invented; anything that could not be confirmed is flagged honestly in Section 13.

---

## 2. Fundamentals — Crawling vs Scraping vs Fetching

### 2.1 The Three Verbs

The industry uses three verbs loosely, but they describe distinct layers of the stack. Getting them precise matters because each maps to a different engineering component and a different cost model.

| Term | Definition | Typical output | Primary engineering concern |
|---|---|---|---|
| **Fetching** | Retrieving a resource over HTTP(S) — a single GET that returns bytes (HTML, JSON, PDF, image). | Raw response bytes + headers + status | Throughput, retries, connection reuse, politeness |
| **Crawling** | *Discovering* and *traversing* the web: following links, maintaining a frontier of URLs, deciding what to visit and when. | A stream of URLs, in priority order, with visit schedules | Frontier design, deduplication, prioritisation, freshness |
| **Scraping** | *Extracting structured data* from fetched content — parsing HTML/DOM or JSON and mapping it to records. | Structured records (rows, JSON documents) | Selector robustness, parser maintenance, schema drift |

A useful mental model: **fetching gets bytes, crawling decides which bytes to get and in what order, scraping turns bytes into data.** A search engine is mostly a crawler (its "scraper" is the indexer's HTML-to-text extraction). A price monitor is mostly a scraper over a small, known URL set. An LLM training-data pipeline is both: crawl the web (or reuse Common Crawl), then scrape/extract text at massive scale.

### 2.2 Why the Distinction Matters Architecturally

- **Fetching is where politeness lives.** Rate limits, `Crawl-delay`, retries and exponential backoff are all enforced at the fetcher, because the fetcher is the only component that touches other people's servers.
- **Crawling is where scale lives.** The frontier and the deduplication layer are what let a pipeline talk about *billions* of URLs instead of thousands. The rest of the pipeline scales by adding workers; the frontier scales by being designed right.
- **Scraping is where maintenance lives.** HTML changes constantly. Selectors break. The scraper layer is the highest-change-rate code in the stack, which is why professional pipelines isolate it behind an extraction contract (a schema) and version it aggressively.

### 2.3 The Shape of the Problem at Scale

Order-of-magnitude numbers frame the engineering. A mid-size production acquisition platform might track:

- **10⁶–10⁷ URLs** in its frontier (dedup set size),
- **10³–10⁵ fetches per hour** across a few hundred hosts (politeness-bound, not bandwidth-bound),
- **10⁸–10¹² records** in the archive over time (WARC + Parquet),
- **minutes-to-days freshness SLAs** depending on the dataset class.

For comparison, the largest open crawls operate at far larger scale: **Common Crawl** (a non-profit founded 2007 by Gil Elbaz) publishes crawls roughly monthly, each typically containing **more than two billion web pages**, with an open repository of more than 10 petabytes of crawl data collected since 2008 — see Section 7. Search-engine crawlers are larger still and are not publicly documented in comparable detail.

The rest of this guide builds the machinery that makes those numbers tractable — starting with the component that decides what gets fetched at all: the URL frontier.
## 3. The URL Frontier — Prioritisation, Politeness, Freshness

### 3.1 What the Frontier Is

The **URL frontier** is the set of URLs the crawler knows about but has not yet fetched, together with the machinery that decides *which* URL to fetch *next*. It is the crawler's scheduling heart, and it is deliberately not a plain FIFO queue: a naive FIFO would let one deep site starve the rest of the web, would hammer a single host, and would re-fetch stale pages while fresh ones wait.

A production frontier is three things in one:

1. **A priority structure** — which URLs matter most (a queue per priority class).
2. **A politeness structure** — per-host ordering so that no host receives requests faster than its allowed rate (a queue per host, drained by a per-host scheduler).
3. **A freshness structure** — when each URL is due again (a revisit schedule merged into the priority calculation).

### 3.2 Prioritisation

Priorities are computed from signals such as:

- **Source class** — a regulatory filing feed outranks a blog.
- **Page quality / importance** — approximated by link popularity (how many other pages link here), page rank-style scores, or site-level reputation.
- **Freshness requirements** — a URL whose content changes hourly must be scheduled before a URL that changes monthly.
- **Depth and scope rules** — pages matching the crawl scope (allowed paths, allowed domains) outrank pages that merely look interesting.
- **Application priority** — a price-monitoring crawl of a competitor catalogue outranks a background discovery crawl.

A common implementation is **priority buckets with weighted round-robin**: N queues with decreasing weights (e.g., 8:4:2:1), so high-priority URLs are fetched more often without ever starving the low-priority queues completely.

### 3.3 Politeness-Aware Ordering

The classic frontier design (dating to the seminal "Mercator" crawler architecture, ⚠-knowledge: Najork & Heydon, 2001, Compaq SRC) separates the *global* priority queue from *per-host* queues:

```text
                    ┌─────────────┐
  discovered URLs → │  frontier   │ → per-host queues
                    │  (priority) │    host A: [u1 u2 u3 ...]
                    └─────────────┘    host B: [v1 v2 v3 ...]
                                       host C: [w1 w2 ...]
                         │
                         ▼
              per-host scheduler: pick the host whose
              next request is due (earliest next-fetch
              time wins), then pop its queue
```

The per-host scheduler enforces **one request per host per minimum interval**: a global `min_interval` (say 1 s) and a per-host `Crawl-delay` (from robots.txt, when present) combine into `next_allowed[host] = last_fetch[host] + max(min_interval, crawl_delay[host])`. The scheduler repeatedly selects the host with the earliest `next_allowed` that is due *now*; if all hosts are cooling down, it sleeps until the earliest due time. This yields politeness (no host is hammered) and throughput (many hosts proceed in parallel) at the same time.

### 3.4 Fresh vs Stale URLs

The frontier must also decide **when to revisit**. Two standard policies (from Cho & Garcia-Molina's crawling literature, ⚠-knowledge):

- **Uniform policy** — revisit every URL on a fixed interval. Simple, wasteful for slowly changing pages, insufficient for fast changers.
- **Proportional / adaptive policy** — estimate each page's change frequency from its change history (e.g., exponential moving average of observed change intervals) and schedule revisits proportionally. Frequently changing pages get revisited often; static pages are re-checked rarely, with a hard cap (never more than once per day, never less than once per quarter).

In practice, pipelines combine both: a **base frequency per source** (Section 11's SLA table is exactly this) refined by **change-rate feedback** — a page that has not changed in the last five visits gets demoted; a page that changed between visits gets promoted.

### 3.5 Frontier Data Structures

| Concern | Typical choice |
|---|---|
| Priority queues | Heap-based queues per priority class (in-memory) or Redis sorted sets (distributed) |
| Per-host queues | In-memory deque per host, or sharded by host-hash across workers |
| Due-time index | A min-heap keyed on `next_allowed[host]`; a time-wheel for revisit schedules |
| Persistence | Frontier snapshots to disk/object storage so a restart resumes where it left off |
| Dedup membership | Bloom filter + exact set (Section 5) |

A distributed frontier shards by host hash so that all URLs of one host are processed by the same worker — politeness is then trivially local, and the only shared state is the dedup set and the priority layer.

## 4. Politeness Policies — robots.txt, Rate Limits, Retries

### 4.1 The Origin: robots.txt (Verified)

The **Robots Exclusion Protocol** — the `/robots.txt` file — was defined by **Martijn Koster** in **1994**, and the canonical early text, *"A Standard for Robot Exclusion"*, records a **consensus reached on 30 June 1994** on the robots mailing list among robot authors. Koster wrote it in response to 1993–1994 incidents in which robots swamped servers with rapid-fire requests, re-fetched the same files repeatedly, or traversed cgi-scripts with side effects. ✅ *Sources: robotstxt.org/orig.html ("This document represents a consensus on 30 June 1994 ... between the majority of robot authors"); RFC 9309, whose abstract states the protocol was "originally defined by Martijn Koster in 1994".*

Three decades later the protocol became a formal standard: **RFC 9309, "Robots Exclusion Protocol"**, IETF **Standards Track**, published **September 2022**, authored by Koster with Google's Gary Illyes, Henner Zeller and Lizzi Sassman. ✅ *Source: rfc-editor.org/rfc/rfc9309.html.* A standard, note, is a formalisation of a *norm* — see Section 9 for the norm-versus-law debate.

### 4.2 The robots.txt Contract

```text
# Example robots.txt — the contract a polite crawler honours
User-agent: *
Disallow: /search?
Disallow: /account/
Allow: /public/

# Source-specific budget: the webmaster asks for 10 seconds between hits
User-agent: CymbalDataBot/1.0
Disallow:
Crawl-delay: 10
```

Rules of interpretation every acquisition platform must encode:

- `User-agent` lines scope rules to a named bot (or `*` for all bots); a crawler should match its own exact user-agent string first, then the `*` group.
- `Disallow` with an empty value means "nothing is disallowed" (all clear). `Disallow: /` means "stay out entirely".
- `Allow` (an extension) overrides `Disallow` for matching paths, with longest-prefix-match winning.
- `Crawl-delay` (an extension, not part of the original 1994 spec nor of RFC 9309's core, but widely honoured) asks for a minimum delay in seconds between requests from that bot.
- **Sitemaps** — a `Sitemap:` directive points at an XML sitemap; sitemaps are the polite, sanctioned way to learn a site's URL inventory and should be consumed before any discovery crawling.

### 4.3 Rate Limiting

Politeness is enforced with a token-bucket or fixed-window limiter per host:

```python
# Token-bucket rate limiter, per host
class HostBudget:
    def __init__(self, rate_per_minute, burst):
        self.rate = rate_per_minute / 60.0      # tokens per second
        self.burst = burst                      # bucket capacity
        self.tokens = burst
        self.updated = time.monotonic()

    def allow(self):
        now = time.monotonic()
        self.tokens = min(self.burst,
                          self.tokens + (now - self.updated) * self.rate)
        self.updated = now
        if self.tokens >= 1.0:
            self.tokens -= 1.0
            return True
        return False
```

Budgets are configured per source class: a regulator's website gets a conservative budget (e.g., 10 requests/minute), a public API gets its documented limit, a large media site gets a modest budget. The polite crawler *under*-uses its budget by design — headroom is what keeps a relationship with a source healthy.

### 4.4 Retries and Exponential Backoff

The web is unreliable; a crawler must distinguish *transient* failures (5xx, timeouts, connection resets) from *permanent* ones (404, 410, 403 policy blocks). The standard discipline:

- **Transient failures** → retry with **exponential backoff plus jitter**: `delay = base * 2^attempt + random(0, jitter)`, e.g. base 2 s → 2 s, 4 s, 8 s, 16 s, capped at 60–300 s, for at most 3–5 attempts.
- **HTTP 429 (Too Many Requests)** and **Retry-After** → honour the server's explicit instruction; back off at least the requested interval and *reduce* the host's budget.
- **Permanent failures** → record the outcome, do not retry, alert only if the URL class is important.
- **Redirects** → follow a bounded chain (≤ 5 hops); record the final URL and canonicalise (Section 5).
- **Idempotency** — fetches must be idempotent so retries are safe: the dedup layer (Section 5) and the archive (Section 6) must tolerate a page being fetched twice after a crash.

### 4.5 Politeness Compliance as a Design Goal

Politeness is not just kindness; it is operational risk management. A crawl that hammers a source gets IP-blocked, poisons its own success-rate telemetry, and can trigger the anti-bot systems described in Section 8. The monitoring layer (Section 6.6) therefore tracks **politeness compliance** — per-host request rates against configured budgets, retry ratios, 429 counts, and robots.txt fetch freshness — as first-class crawl-health metrics.

## 5. Deduplication — URL Normalisation, Hashing, Bloom Filters

### 5.1 Why Dedup Is a Core Subsystem

At web scale, the same logical page is reachable through an unbounded number of distinct URL strings: tracking parameters, session IDs, ordering permutations, case differences. Without aggressive deduplication, a crawler wastes a large fraction of its budget on the same content, inflates the archive, and — worst of all — appears to the target site to be scraping aggressively. Dedup operates at two levels: **URL-level** (never fetch the same URL twice) and **content-level** (detect near-duplicate pages that arrived via different URLs).

### 5.2 URL Normalisation (Canonicalisation)

Before hashing, a URL is canonicalised. Common rules (RFC 3986-based, with web-crawler conventions):

| Rule | Example |
|---|---|
| Lowercase scheme and host | `HTTP://Example.COM` → `http://example.com` |
| Remove default ports | `http://example.com:80/` → `http://example.com/` |
| Remove fragment | `#section` dropped (fragments are client-side) |
| Resolve dot-segments | `/a/./b/../c` → `/c` |
| Sort or strip tracking parameters | drop `utm_*`, `fbclid`, `gclid`, `sessionid`; sort remaining query params |
| Trim trailing slash on empty path | `http://example.com` ≡ `http://example.com/` |
| Percent-decode unreserved characters | `%7E` → `~` |

**Normalisation is a policy, not a fixed algorithm**: the parameter-stripping rules must be configured per source, because a parameter that is tracking noise on one site is a meaningful product variant on another (e.g., `?size=large` on a retailer).

### 5.3 Hashing URLs and Content

- **URL hashing**: after canonicalisation, hash the URL (SHA-1 or SHA-256) to a fixed-size key; the key is the dedup-set member. This keeps the dedup set small — a 20-byte SHA-1 per URL instead of the URL string.
- **Content hashing**: hash the fetched payload (or its extracted text) to detect *exact* duplicates — the same PDF served from two paths, or a syndicated article. SHA-1 of content is the classic choice; the WARC ecosystem (Section 6.5) records payload digests in exactly this way.
- **Near-duplicate detection**: exact hashing misses pages that differ in a few sentences. **SimHash** (Charikar's locality-sensitive hashing scheme, ⚠-knowledge: Moses Charikar, "Similarity Estimation Techniques from Rounding Algorithms", STOC 2002) is the standard technique: tokenise the text, weight tokens by frequency, and build a 64-bit fingerprint per document; documents whose fingerprints differ in **≤ 3–4 bits** are treated as near-duplicates. SimHash is how large corpora pipelines (including LLM training-data pipelines, Section 10.4) discard the thousands of near-identical copies of the same article across the web.

### 5.4 The Bloom Filter

A Bloom filter is a space-efficient probabilistic set: a bit array of size *m* with *k* hash functions. To insert a key, set the *k* bits; to test membership, check the *k* bits — all must be set. It can answer "definitely not seen" with certainty, but "seen" is only *probable*: **false positives** are possible, false negatives are not.

```python
# Minimal Bloom filter (m bits, k hashes) — frontier dedup sketch
class BloomFilter:
    def __init__(self, m, k, hashes):
        self.bits = bytearray(m // 8 + 1)
        self.m, self.k, self.hashes = m, k, hashes

    def _pos(self, h, i):          # i-th hash gives a bit position
        return (h[i] % self.m)

    def add(self, key):
        for h in self.hashes(key):
            self.bits[self._pos(h, 0) // 8] |= 1 << (self._pos(h, 0) % 8)

    def maybe_contains(self, key):
        return all(self.bits[self._pos(h, 0) // 8] & (1 << (self._pos(h, 0) % 8))
                   for h in self.hashes(key))
```

The false-positive rate is `p ≈ (1 − e^(−kn/m))^k`, minimised at `k = (m/n)·ln 2`, giving `p ≈ 0.6185^(m/n)`. Concretely: **10 bits per element and 7 hash functions → ~1% false-positive rate; 16 bits per element → ~0.05%.** ⚠-knowledge: the analysis is standard (Burton H. Bloom, "Space/Time Trade-offs in Hash Coding with Allowable Errors", CACM, 1970).

**Practical use in the frontier**: the Bloom filter is the *fast negative* filter for the dedup set. On discovering a URL: canonicalise → hash → ask the Bloom filter. If it says *no*, the URL is certainly new: insert and enqueue. If it says *maybe*, consult the exact set (the authoritative, slower store — a database or key-value store keyed by URL hash). This layering gives the exactness of a real set with the memory economy of a filter, which is why frontier implementations (including Nutch's URL filters and most distributed crawlers) use exactly this pattern.

**The false-positive trade-off, stated plainly**: a Bloom filter can only *skip* a URL that is actually new (rare, ~p of the time) — it can never fetch a URL twice incorrectly. The cost of a false positive is a missed page; the benefit is a dedup set that fits in memory at 10⁸–10⁹ URLs. Tune *m* and *k* to the value of completeness for the crawl class: 1% misses is acceptable for a discovery crawl, rarely acceptable for a regulatory-filing crawl. Because filters cannot be shrunk or have elements removed, production systems use **scalable/counting variants** or shard the filter by host so a hot site's churn does not force filter rebuilds.
## 6. Pipeline Architecture — From Fetch to Monitor

### 6.1 The Pipeline at a Glance

A production acquisition pipeline is a **seven-stage data flow** with a queue at its centre and monitoring wrapped around the whole:

```text
        ┌──────────────────────────────────────────────────────────┐
        │                     Orchestration (Airflow)              │
        │      schedules crawls, retries runs, manages backfills   │
        └──────────────────────────────────────────────────────────┘
                        │
   seed URLs / sitemaps ▼
 ┌──────────┐   ┌──────────────┐   ┌───────────┐   ┌────────────┐
 │ 1 FETCH  │ → │ 2 PARSE      │ → │ 3 EXTRACT │ → │ 4 TRANSFORM│
 │ HTTP(S)  │   │ HTML/JSON/   │   │ selectors │   │ clean,     │
 │ politeness│   │ PDF parsing  │   │ → records │   │ type,      │
 └──────────┘   └──────────────┘   └───────────┘   │ enrich     │
    │  ▲                                            └────────────┘
    │  │ new URLs (discovered links)                       │
    ▼  │                                                  ▼
 ┌──────────┐   ┌──────────────────────┐   ┌──────────────────────┐
 │ 5 STORE  │ ← │   THE QUEUE          │ ← │ 6 SCHEDULE (frontier │
 │ WARC +   │   │   (Kafka-style bus)  │   │    + revisit logic)  │
 │ Parquet  │   └──────────────────────┘   └──────────────────────┘
 └──────────┘            │
                         ▼
              ┌──────────────────────┐
              │ 7 MONITOR            │
              │ success rates,       │
              │ crawl health,        │
              │ politeness compliance│
              └──────────────────────┘
```

The stages are deliberately decoupled: each stage reads from and writes to the queue, so a slow parser cannot block the fetcher, and a fetcher outage does not lose already-extracted records.

### 6.2 The Stages

1. **Fetch** — the politeness-constrained HTTP layer (Sections 3–4). Produces raw response bytes, status, headers, timing, and the final canonical URL.
2. **Parse** — turns bytes into a document tree: HTML → DOM (BeautifulSoup, lxml, or a browser engine in headless mode), JSON → object, PDF → text. This is where malformed markup ("tag soup") is tamed.
3. **Extract** — the scraping proper: selectors (CSS/XPath) or trained extractors map document structure to typed records against a versioned **extraction contract** (a schema). Extraction failures — the selector no longer matches — are *data-quality incidents*, not code bugs: they must surface in monitoring and block downstream publication rather than silently producing empty records.
4. **Transform** — cleaning, type coercion, deduplication keying (Section 5), enrichment (join against reference data), PII screening (Section 9.5), and schema validation. The output is a **canonical record** independent of the source's markup.
5. **Store** — write-through to the archive: raw bytes to WARC/WAT (Section 6.5), canonical records to Parquet in the data lake (Section 6.5), and operational state (fetch outcomes, digests) to the crawl database.
6. **Schedule** — the frontier and revisit logic decide what is due next (Section 3.4); the orchestrator (Section 6.4) decides *when* pipeline runs execute.
7. **Monitor** — crawl health, success rates, freshness SLAs, and politeness compliance (Section 6.6).

### 6.3 The Queue at the Centre

The queue is the backbone, and it is a **messaging system**, not a database table. The repository already contains the deep treatments of these patterns — this guide does not re-derive them:

- **Producers and consumers, partitioning, and consumer groups** — see [banking/kafka_guide.md](../banking/kafka_guide.md), the repository's Kafka deep-dive.
- **Backpressure, exactly-once semantics, DLQs and the broker-vs-bus comparison** — see [technology/enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md).

What the acquisition context adds is *what* flows through the queue, as typed topics:

| Topic | Producer | Consumer(s) | Semantics |
|---|---|---|---|
| `urls.to_fetch` | Frontier / discovery | Fetcher workers | At-least-once; idempotent fetches make duplicates harmless |
| `fetched.pages` | Fetcher | Parser | Keyed by URL hash; ordering per host preserved by partition key |
| `parsed.docs` | Parser | Extractor | Keyed by document id |
| `extracted.records` | Extractor | Transformer → store | The canonical record stream |
| `crawl.events` | All stages | Monitor | Metrics/audit trail, not data |
| `dead.letters` | Any stage | Ops tooling | Poison messages — see below |

**Backpressure** in the acquisition context is the politeness layer itself: the fetcher is a *rate-limited consumer*, so if producers outpace it, the natural response is not to fetch faster (that would be impolite) but to let the queue absorb the burst and to **slow the discovery producer** (the frontier's crawl-rate governor). The queue's backlog length is therefore a first-class crawl-health metric: a growing `urls.to_fetch` backlog means discovery is outrunning the polite fetch rate, and the governor should throttle discovery, not the fetcher.

**Dead-letter queues (DLQs)** catch the poison messages: records that fail parsing or extraction repeatedly (schema drift, corrupted payloads, unexpected content types). DLQ handling in a crawl pipeline is a *feedback loop*: a DLQ entry for a selector failure should (a) alert the owning team, (b) suppress that extractor for the affected source until fixed, and (c) preserve the raw WARC so the failure is fully recoverable — the raw archive is the ultimate replay source.

### 6.4 Scheduling and Orchestration

The pipeline runs on a calendar, not ad hoc. **Apache Airflow** is the repository's reference orchestrator — see [technology/apache_airflow_guide.md](apache_airflow_guide.md) for the platform deep-dive (DAG authoring, scheduling semantics, retries and deadline alerts); this guide only sketches the acquisition-specific DAG shapes:

- **One DAG per source class**, parameterised by date/run id: `crawl_mas_daily`, `crawl_sgx_intraday`, `refresh_reference_data_weekly`.
- **DAG structure**: `check_robots_and_budget → crawl (dynamic fan-out over frontier shards) → parse → extract → validate → publish → alert_on_sla`.
- **SLA enforcement**: each DAG declares a freshness SLA (Section 11.4); Airflow's deadline/alerting machinery (Section 8 of the Airflow guide) pages the on-call when the daily regulatory crawl misses its 07:00 SGT completion target.
- **Backfills and replays**: a failed day's crawl must be re-runnable from the WARC archive without re-hitting the source — replay from archive is *always* preferred over re-fetch, both for politeness and for reproducibility.

### 6.5 The Storage Layer

**Raw archives — WARC/WAT.** The **WARC (Web ARChive) format** is the archival standard for crawl payloads: one file per batch of records, each record carrying the request URI, capture timestamp, HTTP headers, and the payload with a **SHA-1 digest** (the digest is what makes content-dedup and replay verification possible). **WAT (Web Archive Transformation)** files hold derived metadata — extracted text, outlinks, header summaries — in JSON, so analytics can run without touching the big WARC files. WARC is standardised as **ISO 28500** (⚠-knowledge: first edition 2009, current edition 2017). Common Crawl publishes its crawls in WARC/WAT/WET form — the WET files are the plain-text extractions used to build text corpora (Section 10.4).

**Columnar stores — Parquet.** The canonical records land as **Parquet** files, partitioned by source and date (and, for market data, by instrument/venue). Parquet's columnar layout gives the analytics layer (and the lakehouse engines in the repository's [technology/cloud_object_storage_lakehouse_guide.md](cloud_object_storage_lakehouse_guide.md) and [technology/data/delta_lake_vs_iceberg.md](data/delta_lake_vs_iceberg.md)) compression and scan performance; schema evolution (adding fields as extraction contracts grow) is handled by the lakehouse table format.

**The data lake.** The acquisition platform is a *producer* into the firm's data lake: `landing/` (WARC raw), `curated/` (Parquet canonical records), `consumption/` (joined, quality-checked datasets for downstream systems). The lakehouse pattern — object storage plus a table format plus SQL engines — is covered in the sibling guides cited above; the acquisition pipeline's contract with the lake is: raw is immutable, curated is versioned, consumption is quality-gated.

**Operational stores.** Beyond the lake: the crawl database (URLs, fetch outcomes, digests, revisit schedules — the persistent frontier state), the dedup set (Section 5), and the queue topics (Section 6.3).

### 6.6 Monitoring — Crawl Health, Success Rates, Politeness Compliance

Monitoring answers three questions continuously:

1. **Is the crawl healthy?** Fetch throughput, error rates by class (4xx/5xx/timeout), retry ratios, queue backlogs, WARC write rates, disk/object-storage growth.
2. **Are we meeting SLAs?** Freshness per dataset class (Section 11.4) — time since last successful fetch per URL, per source; publication lag; extraction-contract failure counts (the "silent empty record" detector).
3. **Are we polite?** Per-host request rate vs configured budget, 429/403 counts (a rising 429 rate means the budget is too high or the source changed its posture), robots.txt fetch freshness, and the *blocked-host register* — hosts that have escalated to anti-bot enforcement (Section 8) and require human review before any further automated access.

The politeness dashboards matter as much as the throughput ones: in a regulated environment (Section 11) the ability to *demonstrate* compliance — request logs, rate budgets, robots.txt honouring — is an audit artefact, not just an operational nicety.
## 7. Tooling — The Verified Landscape

Every tool in this section was verified against its project site, GitHub repository, or official documentation this pass; the audit table in Section 12 records the sources. Nothing here is from memory.

### 7.1 Scrapy — the Python Workhorse

**Scrapy** is the leading open-source Python web-crawling framework: asynchronous, extensible, BSD-licensed. Its lineage is well documented by its maintainer: Scrapy was created inside **Insophia**, a Montevideo-based Python consultancy founded by **Pablo Hoffman**, and was **released as open source under the BSD license in August 2008** — making 2008 the founding year. The commercial company around it, **Scrapinghub**, was **founded in 2010** by Hoffman (and Shane Evans); Scrapinghub **rebranded to Zyte** (⚠ — the rebrand is widely reported as 2019; the company's own history posts describe it as "10 years ago" relative to Scrapinghub's 2010 founding, and the earliest Wayback snapshot of the rebrand announcement post is February 2021, so the exact month could not be re-confirmed at a primary source this pass). Scrapy 1.0 shipped in 2015. ✅ *Sources: zyte.com/blog/the-rise-of-scrapy/; zyte.com/blog/history-of-zyte-formerly-scrapinghub/; scrapy.org; github.com/scrapy/scrapy.*

Architecturally, Scrapy is the reference implementation of this guide's Sections 3–6 in miniature: a scheduler (the frontier), a downloader (the fetcher, with built-in robots.txt and `DOWNLOAD_DELAY` support), spiders (the extractors), item pipelines (the transform), and middlewares (retries, proxies, user-agents).

### 7.2 Playwright — Microsoft's Browser Automation

**Playwright** is Microsoft's open-source browser-automation framework for Chromium, Firefox and WebKit, with a single API across languages. Its first stable release, **v1.0.0, was published on 6 May 2020** (GitHub release record) — 2020 is the initial release year. ✅ *Sources: github.com/microsoft/playwright; api.github.com/repos/microsoft/playwright/releases/tags/v1.0.0 (published_at 2020-05-06).* Where Scrapy is the HTTP crawler, Playwright is the *browser-level* fetcher: it renders JavaScript, so it reaches the large fraction of the modern web that is client-side rendered and invisible to plain HTTP fetches. It is the tool of choice when the *page* must execute before there is anything to scrape.

### 7.3 Puppeteer — Google's Headless Chrome Driver

**Puppeteer** is the Node.js API for controlling Chrome/Chromium over the DevTools Protocol, built by the **Google Chrome DevTools team**. It was **released in 2017**: the first npm package, v0.9.0, was published **16 August 2017**, and it was covered as a new Google tool in August 2017. ✅ *Sources: registry.npmjs.org/puppeteer (0.9.0, 2017-08-16); infoq.com/news/2017/08/google-puppeteer-headless-chrome/; github.com/puppeteer/puppeteer.* Playwright's browser automation is the spiritual successor to Puppeteer's approach, with cross-browser support and a stronger testing story.

### 7.4 BeautifulSoup — the Parser

**Beautiful Soup** is the Python HTML/XML parsing library that tames "tag soup" — malformed markup — into a navigable parse tree. It was **started in 2004 by Leonard Richardson**, who still maintains it; Beautiful Soup 3 was the official release line from May 2006 to March 2012, with Beautiful Soup 4 current. ✅ *Sources: crummy.com/software/BeautifulSoup/; en.wikipedia.org/wiki/Beautiful_Soup_(HTML_parser).* In a pipeline it sits in the **parse** stage (Section 6.2): it is not a fetcher and not a crawler — it turns fetched HTML into something extractors can query.

### 7.5 Colly — the Go Framework

**Colly** is the elegant scraper/crawler framework for **Go**: a lightweight, fast, single-binary crawler with a clean fluent API, distributed-crawling support, and pluggable storage. Its GitHub repository was **created on 29 September 2017**, placing its origin in **2017** (the brief's "2017/2018" estimate). ✅ *Source: api.github.com/repos/gocolly/colly (created_at 2017-09-29); github.com/gocolly/colly.* Colly is the common choice when the crawl is part of a Go service (e.g., a price monitor inside a Go backend) rather than a Python data platform.

### 7.6 Apache Nutch — the Original Open-Source Web-Scale Crawler

**Apache Nutch** is the open-source, Java web crawler whose history is intertwined with Hadoop itself. It **originated with Doug Cutting and Mike Cafarella** (Cutting is also the creator of Lucene and Hadoop); a successful 100-million-page demonstration system existed by **June 2003**, and the distributed filesystem and MapReduce machinery Nutch needed were spun out as the **Hadoop** subproject. Nutch **joined the Apache Incubator in January 2005 and graduated in June 2005 as a subproject of Lucene**; it became an **independent Apache top-level project in April 2010** (so the brief's "top-level 2005-ish" is approximately right only for the Lucene-subproject graduation; full top-level status came in 2010). In **February 2014 Common Crawl adopted Nutch** for its open large-scale crawl. ✅ *Sources: en.wikipedia.org/wiki/Apache_Nutch; nutch.apache.org; the Hadoop origin story (2002, Cutting & Cafarella) is corroborated in Apache Hadoop's own history documentation and widely reproduced.* Nutch matters today as the reference *batch* architecture — crawl cycles processed with MapReduce-style jobs — and as the ancestor of the modern Hadoop ecosystem.

### 7.7 Common Crawl — the Open Web Archive

**Common Crawl** is the non-profit **501(c)(3)** organisation that crawls the web and freely provides its archives and datasets on AWS. It was **founded in 2007 by Gil Elbaz**; the foundation maintains an open repository of crawl data **collected since 2008, totalling more than 10 petabytes**, publishing crawls roughly monthly, **each typically containing more than two billion web pages**. ✅ *Sources: commoncrawl.org/about; en.wikipedia.org/wiki/Common_Crawl; en.wikipedia.org/wiki/Gil_Elbaz.* For any organisation that needs web-scale text *without running its own crawl*, Common Crawl is the standard starting point — the raw WARC/WAT/WET corpus behind most open web-text research and, critically, the source material for the C4 dataset (Section 10.4).

### 7.8 The Tooling Decision Table

| Need | Tool | Why |
|---|---|---|
| High-volume HTTP crawling, Python platform | Scrapy | Mature, asynchronous, robots.txt + delay support built in |
| JavaScript-rendered pages | Playwright / Puppeteer | Real browser engines; Playwright for cross-browser + multi-language |
| Parse messy HTML in Python | BeautifulSoup (+ lxml) | The standard parser |
| Embedded Go crawler | Colly | Lightweight, fast, composable |
| Batch web-scale crawl, Hadoop stack | Apache Nutch | The reference distributed crawler |
| Web corpus without crawling | Common Crawl | 10+ PB open archive, monthly crawls |
| Regulated / licensed data | Official APIs and data vendors | The only fully compliant option — see Sections 9 and 11 |

## 8. The Anti-Bot Arms Race — An Ethical Boundary

### 8.1 The Landscape

A significant fraction of the commercial web now sits behind **bot-management systems**. The major vendors — **Cloudflare** (Bot Management, Turnstile), **Akamai** (Bot Manager), **DataDome**, and others — operate detection layers that classify every request as human, good bot, or bad bot using a stack of signals: TLS/JA3-style client fingerprints, HTTP header ordering and behaviour, JavaScript challenges, proof-of-work challenges, behavioural and mouse-movement analysis (on real browsers), IP and ASN reputation, and machine-learning classifiers trained on billions of requests.

This is an **arms race** in the technical sense: as detection improves, automated access attempts become more sophisticated; as access attempts become more sophisticated, detection vendors add new signals. The practical consequence for acquisition engineering is a **three-tier web**:

1. **Open tier** — sites that welcome or tolerate well-behaved crawlers (most government and regulator sites, many open-data portals, most documentation sites). robots.txt and sitemaps work here.
2. **Guarded tier** — commercial sites that permit some automated access (often via an official API, a partner programme, or a data licence) and use bot management to enforce those boundaries.
3. **Closed tier** — sites whose entire business model or legal posture requires preventing automated access (paywalled content, login-walled services, sites whose terms prohibit scraping).

### 8.2 The Ethics of Evasion — Where the Line Is

This guide deliberately **does not describe evasion techniques** — no challenge-solving, no fingerprint spoofing, no CAPTCHA circumvention. The reasons are both ethical and practical, and every acquisition architect should internalise them:

- **Access controls are boundaries.** When a site operates a technical gate (CAPTCHA, challenge, block), it is exercising the same right a locked door expresses. The CFAA jurisprudence (Section 9.3) has repeatedly distinguished *public data* from *data behind access controls* — circumventing a technical access control is precisely the conduct that converts a civil dispute into a computer-crime question (18 U.S.C. § 1030(a)(2)/(a)(5)(B), and the DMCA's anti-circumvention provisions for access controls). The law is unsettled in detail, but the *direction of risk* is unambiguous: evasion is the highest-risk activity in this entire discipline.
- **The reputational and relational cost.** Being identified as an evader burns the source relationship, gets the firm's IP ranges blacklisted, and can trigger litigation — hiQ v. LinkedIn (Section 9.4) shows how a scraping dispute can consume a company for years.
- **The professional norm.** The industry's legitimate players — including the vendors themselves — sell *compliant* access: official APIs, licensed data feeds, and rendering services operated within the source's terms. "We needed the data" is not a legal defence.

### 8.3 The Legitimate Playbook

When a target is guarded or closed, the professional sequence is:

1. **Look for the sanctioned path first**: an official API (even paid), an open-data mirror, a licensed data vendor, or the source's own bulk-download facility.
2. **Negotiate**: many operators will agree to a documented access arrangement — a named user-agent, a rate budget, a data-usage agreement — when asked respectfully and specifically.
3. **Respect the boundary when it is drawn**: a 403 or a challenge means stop, document, and escalate to the source owner or legal, not to the evasion toolkit.
4. **Record the decision**: for a regulated firm (Section 11), the audit trail of "source X blocked us; we use vendor Y's licensed feed instead" is exactly the governance evidence an internal or external reviewer wants to see.

The arms race, viewed correctly, is a **market for access**: the winning strategy is not a better evasion technique but a better *access strategy* — APIs, licences, and relationships. Section 9 now gives the legal framework that makes that strategy the only defensible one.
---

## 9. Legal and Ethical Boundaries

### 9.1 The Legal Map at a Glance

An acquisition platform operates inside several legal layers at once. The table below maps the layers this guide covers; each is expanded in the subsections that follow. Two layers were verified at primary sources this pass (the CFAA and the case law built around it, GDPR, PDPA — Section 12); the rest are flagged where the law is unsettled or was not re-verified live.

| Layer | Instrument | What it governs | Status this pass |
|---|---|---|---|
| robots.txt | The Robots Exclusion Protocol — origin 1994, IETF RFC 9309 (2022) | A *norm*, not a law: what a polite crawler honours | ✅ (verified in §4.1) |
| Terms of service | Contract law | What the site's operator permits, as a contract | ✅ (hiQ breach finding, §9.4) |
| CFAA | 18 U.S.C. § 1030 — the Computer Fraud and Abuse Act of 1986 | Unauthorised access to protected computers | ✅ (§9.3) |
| DMCA | 17 U.S.C. § 1201 (1998) | Circumvention of technical access controls; takedowns | ⚠ (§9.6) |
| GDPR | Regulation (EU) 2016/679 — adopted 2016, applicable 2018 | Processing of personal data of individuals in the EU | ✅ (§9.5) |
| PDPA | Singapore Personal Data Protection Act 2012 — main rules in force 2014 | Collection, use and disclosure of personal data in Singapore | ✅ (§9.5) |
| Copyright | National copyright laws + the Berne Convention | Reproduction of protected expression | ⚠ (§9.6) |

### 9.2 robots.txt — Norm, Not Law

The protocol's origin (Martijn Koster, 1994) and its formalisation as **RFC 9309** (September 2022) are verified and covered in §4.1; this subsection adds the legal analysis.

- **Compliance with robots.txt is a professional norm, not a statutory obligation.** In almost no jurisdiction is ignoring robots.txt itself a crime. But it is *operationally* binding: a crawler that ignores it gets blocked, blacklisted, and — in any later dispute — has handed the other side evidence of bad faith.
- **Standards do not create duties.** RFC 9309 formalised a syntax, not a legal obligation; contracts and statutes create obligations. The polite crawler honours robots.txt because the norm is enforced by the market (access), not by the courts.
- **The page-level analogue.** The HTML `meta robots` tag (`noindex`, `nofollow`) plays the same role at page level and is treated the same way by the same argument.

### 9.3 The CFAA — 1986 Origin and the "Without Authorization" Debate

The **Computer Fraud and Abuse Act** is the US federal computer-crime statute, codified at **18 U.S.C. § 1030**. Its history, verified this pass:

- Section 1030 first appeared in **1984** inside the Comprehensive Crime Control Act. The **Computer Fraud and Abuse Act of 1986 — Public Law 99-474, signed 16 October 1986** — substantially expanded it, which is why the statute is universally called "the CFAA of 1986". ✅ *Sources: uscode.house.gov/statutes/pl/99/474.pdf (Pub. L. 99-474, "OCT. 16, 1986"); en.wikipedia.org/wiki/Computer_Fraud_and_Abuse_Act.*
- The CFAA criminalises, among other things, intentionally accessing a **protected computer** without authorisation — or **exceeding authorised access** — to obtain information (18 U.S.C. § 1030(a)(2)); victims may sue civilly under § 1030(g).
- **The debate.** For two decades, litigants argued whether violating a website's terms of service turns otherwise permitted access into access "without authorisation". Courts split. The Supreme Court's **Van Buren v. United States** (593 U.S. ___, 141 S. Ct. 1648, decided **3 June 2021**) narrowed the "exceeds authorised access" clause to *gate-based* violations (accessing parts of a system to which access does not extend), not *policy-based* violations (using authorised access against the operator's rules). ✅ *Source: Wikipedia (Van Buren), cited in the hiQ record.*
- **The open question.** Whether a cease-and-desist letter converts access to *publicly available* data into "unauthorised" access under the CFAA was the precise question in hiQ v. LinkedIn — the next subsection, and the most important case history in this entire discipline.

### 9.4 hiQ v. LinkedIn — The Definitive Case History

The most consequential scraping case to date: **hiQ Labs, Inc. v. LinkedIn Corp.**, 938 F.3d 985 (9th Cir. 2019). Every date below was verified this pass against the case record, the opinions, or contemporaneous reporting (sources in Section 12).

| Date | Event | Status |
|---|---|---|
| 2012 | hiQ founded; built analytics products ("Keeper", "Skill Mapper") over *public* LinkedIn profiles | ✅ |
| 23 May / 24 June 2017 | LinkedIn sends hiQ cease-and-desist letters alleging violations of its User Agreement, the CFAA, the DMCA and trespass; deploys technical blocks | ✅ |
| June 2017 | hiQ sues in the Northern District of California (No. 17-cv-03301-EMC), seeking declaratory relief and an injunction | ✅ |
| 14 August 2017 | **Judge Edward Chen grants the preliminary injunction** — LinkedIn must withdraw the letters and stop blocking hiQ's access to public profiles (273 F. Supp. 3d 1099 (N.D. Cal. 2017)) | ✅ |
| 9 September 2019 | **Ninth Circuit affirms** (938 F.3d 985): accessing *publicly available* data, where no authentication is required, does not violate the CFAA's "without authorisation" — the CFAA is an anti-intrusion statute, not a misappropriation statute | ✅ |
| 3 June 2021 | *Van Buren v. United States* decided (141 S. Ct. 1648) | ✅ |
| 14 June 2021 | **U.S. Supreme Court grants certiorari, vacates and remands** hiQ for reconsideration in light of *Van Buren* (141 S. Ct. 2752) | ✅ |
| 18 April 2022 | **Ninth Circuit reaffirms** the preliminary injunction on remand (31 F.4th 1180 (9th Cir. 2022)) — *Van Buren* confirmed the analysis | ✅ |
| November 2022 | District court rules hiQ **breached LinkedIn's User Agreement**; the parties reach a **settlement** (reported ~December 2022; terms confidential) | ✅ |

**The lessons for acquisition architecture:**

1. **Public data ≠ free data.** hiQ won the CFAA question (public, unauthenticated data is not "unauthorised access"), but LinkedIn won the contract question (the User Agreement was a contract hiQ breached). The two theories are independent, and a scraper can lose on either.
2. **"Publicly available" is a fact-specific finding.** The court's test: the means of access are widely known, and the visitor encounters no warnings, encryption, password requests or other indicia of intended privacy. A login wall changes everything — see Section 8.2.
3. **Litigation is existential risk.** Five years of litigation, a GVR from the Supreme Court, and a settlement with confidential terms: this is the cost of fighting over data access. The Section 8.3 playbook (APIs, licences, negotiation) exists to keep a firm out of this exact story.

### 9.5 Data Protection — GDPR and PDPA

**GDPR (EU).** The **General Data Protection Regulation — Regulation (EU) 2016/679** — was **adopted on 27 April 2016** (OJ L 119, 4.5.2016) and became **applicable on 25 May 2018**. ✅ *Source: EUR-Lex, eur-lex.europa.eu/eli/reg/2016/679/oj/eng.* For acquisition engineering the consequences are concrete:

- Scraping personal data is **processing** under the GDPR (Art. 2, 4), and it requires a **lawful basis** (Art. 6). "Publicly available" is *not* a lawful basis by itself.
- **Enforcement example, verified:** the French DPA (**CNIL**) fined **Clearview AI €20 million** in **October 2022** (decision published 20 October 2022) for collecting images from the open web without a legal basis under the GDPR — the canonical "we scraped public data" penalty. ✅ *Sources: cnil.fr (press release); Hunton Privacy & Cybersecurity Law Blog.*
- Territorial reach (Art. 3) extends to processors outside the EU who target EU individuals — directly relevant to a Singapore pipeline that touches EU data subjects.

**PDPA (Singapore).** The **Personal Data Protection Act 2012** (Act 26 of 2012) governs collection, use and disclosure of personal data by organisations in Singapore; its **main data-protection obligations took effect on 2 July 2014**, enforced by the **Personal Data Protection Commission (PDPC)**. ✅ *Sources: sso.agc.gov.sg/Act/PDPA2012; pdpc.gov.sg.* For a Singapore-headquartered bank, the PDPA is the baseline; the MAS overlay on data governance and technology risk is covered in the sibling [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) guide.

**The acquisition translation:** the **personal-data screen** is a compliance control, not an option. Section 6.2's transform stage already names "PII screening" — in a regulated firm this step is mandatory, automated, and audited: public-web data is screened for personal data at ingest, and any dataset containing it is quarantined and routed to the data-protection officer rather than published (§11.3).

### 9.6 Copyright, Database Rights and Terms of Service

- **Copyright** protects original *expression*, not facts or data as such (the US *Feist v. Rural* line, 1991 ⚠-knowledge). But the *selection and arrangement* of a database can be protected, and the EU's **Database Directive (96/9/EC)** creates a *sui generis* right in substantial extractions — a real exposure for corpus builders (§10.4). ⚠-knowledge: not re-verified against the directive's text this pass.
- **Terms of service are contracts.** The hiQ case (§9.4) shows the mechanism: even where the CFAA does not apply, a breached User Agreement is a breach of contract. Every source onboarding should include a ToS review, not just a robots.txt read.
- **DMCA.** The anti-circumvention provisions (17 U.S.C. § 1201) protect *technical access controls* — another reason Section 8.2 draws the evasion line where it does. The DMCA's takedown machinery is a separate, content-focused regime.

### 9.7 The Compliance Checklist for Acquisition Design

Every acquisition pipeline in a regulated firm should be able to produce, per source:

1. robots.txt and sitemaps fetched, parsed, and *snapshotted* (evidence of what was seen when).
2. A named, documented user-agent with a contact URL.
3. A configured per-host rate budget (§4.3) and proof it was honoured (request logs).
4. A terms-of-service review and a legal sign-off per source class.
5. A personal-data screen at the transform stage (§6.2, §9.5).
6. A licence/redistribution determination: internal use vs redistribution, with the audit trail.
7. A retention and deletion policy matching the data class.
8. A blocked-host register and an escalation path (§6.6, §8.3).

Section 11 shows this checklist applied end-to-end.

---

## 10. Large-Scale Use Cases — Search, Prices, News, LLM Corpora

### 10.1 Search Indexes — The Canonical Crawler

The web-search index is the *original* large-scale web data acquisition use case — the one that created the discipline. The founding facts, verified: **BackRub**, the research project started in **1996** by Stanford PhD students **Larry Page and Sergey Brin**, explored the web's link structure as a graph; Page's crawler began exploring the web in **March 1996**, and the first version of Google appeared on the Stanford site in August 1996. **Google** was launched in **1998**. ✅ *Source: en.wikipedia.org/wiki/History_of_Google.* The PageRank insight — importance flows through backlinks — is the intellectual ancestor of every link-based prioritisation signal in Section 3.2.

Search engines are where Sections 3–5 scale to their limit: frontiers of tens of billions of URLs, politeness enforced across millions of hosts, dedup sets that cannot be wrong at scale. Their internal scale is not publicly documented in verifiable detail (flagged in Section 13), which is precisely why **Common Crawl** (§7.7) exists as the open, verifiable stand-in, and why **Apache Nutch** (§7.6) was built as the open-source reproduction of the architecture.

### 10.2 Price Monitoring — The Polite-Crawl Discipline

Price monitoring is the scraper-heavy use case: a small, known URL set (10³–10⁶ product pages), high revisit frequency (hours to days), and extreme sensitivity to extraction breakage — every retailer's template change is an incident (§2.2's "scraping is where maintenance lives").

- **The operational profile** is politeness-bound (§4.3): hundreds of retailers, each with its own budget, robots.txt and anti-bot posture (§8).
- **The legal profile** is the sharpest of the four use cases: ToS typically prohibit scraping, and the guarded/closed tiers (§8.1) are dense. The named example, **CamelCamelCamel** — the long-running Amazon price tracker — is ⚠-knowledge: founded in 2008 per press and industry sources, not re-verified at a primary source this pass.
- For a bank, price monitoring appears in research and credit contexts (inflation signals, retail sector analysis) — a lighter cousin of the Section 11 worked example.

### 10.3 News Aggregation — Near-Duplicate Detection at Scale

News aggregation is the *freshness and dedup* use case: thousands of publishers, publication cadence in minutes, and the same story syndicated across hundreds of outlets. **Google News launched in 2002** ✅ (*source: en.wikipedia.org/wiki/History_of_Google*), and the rise of RSS/Atom feeds made syndication machine-readable.

The engineering centre of gravity is **near-duplicate detection** — exactly Section 5.3's SimHash: fingerprints of articles across outlets, with ≤ 3–4 bit differences treated as the same story, so an aggregator can cluster, rank and de-duplicate without storing the web's worth of copies. Freshness SLAs are minutes; the commercial battleground is licensing (flagged ⚠ in Section 13: current Google News licensing arrangements were not re-verified this pass).

### 10.4 LLM Training Corpora — Crawl-Scale Text

The most recent and largest-scale use case: turning the open web into pre-training text. The verified facts:

- **C4 — the Colossal Clean Crawled Corpus** — was introduced with Google's T5 paper, *"Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer"* (Raffel et al.), submitted to arXiv on **23 October 2019** (arXiv:1910.10683). ✅ *Source: arxiv.org/abs/1910.10683.* C4 is English text extracted from a **Common Crawl** snapshot, aggressively cleaned and de-duplicated (code, boilerplate and placeholder text removed). Its released size is commonly cited at roughly **800 GB** — the figure comes from dataset cards, not from re-reading the paper's tables this pass, so it is marked ⚠.
- **The Pile** — *"The Pile: An 800GB Dataset of Diverse Text for Language Modeling"* (Gao et al.), submitted **31 December 2020** (arXiv:2101.00027): an **825 GiB** English corpus built from **22 diverse subsets**, including a Common Crawl-derived subset (**Pile-CC**). ✅ *Source: arxiv.org/abs/2101.00027.*
- **GPT-3** (Brown et al., 2020) trained on a *filtered* Common Crawl as its largest bucket — ⚠-knowledge, not re-verified against the paper this pass.

The engineering lessons transfer directly from this guide's Sections 3–6: corpus-scale deduplication is mandatory (Section 5 — near-duplicate articles inflate corpora by multiples), quality filtering is a pipeline stage of its own, and the legal exposure (§9.6 copyright, §9.5 personal data) is the highest of any use case — the reason the AI-governance angle lives in the sibling [AI Governance, Bias and Red-Teaming](ai_llm/ai_governance_bias_redteaming_guide.md) guide, with the training-side machinery in [LLM Optimization: The Complete Guide](ai_llm/llm_optimization_complete_guide.md).

### 10.5 The Common Requirements

| Use case | Volume | Freshness | Legal exposure | Core machinery |
|---|---|---|---|---|
| Search indexes | 10⁹+ URLs | Weeks–months | Low–moderate | Frontier, dedup, distributed fetch (§§3–5) |
| Price monitoring | 10³–10⁶ URLs | Hours–days | High (ToS, anti-bot) | Polite fetch, robust selectors, change detection |
| News aggregation | 10⁴–10⁷ URLs | Minutes–hours | Moderate (copyright, licensing) | SimHash dedup (§5.3), feed ingestion |
| LLM corpora | 10⁸–10¹² tokens | Snapshot-based | High (copyright, PII) | WET extraction, corpus dedup, filtering |

All four converge on the same architecture — Section 6's pipeline with Sections 3–5's machinery and Section 9's boundaries. Section 11 now shows that architecture operating inside a bank.

---

## 11. Cymbal Bank Worked Example — Market and Regulatory Data

### 11.1 The Scenario

**Cymbal Bank** — the repository's Singapore-headquartered bank persona — operates an acquisition platform that feeds two downstream estates: the **market-data distribution layer** (the DDS-based bus in the sibling [DDS Guide](dds_guide.md), whose §10 is the trading-floor worked example) and the **regulatory-intelligence workspace** used by research, compliance and credit teams. The platform's scope is deliberately constrained: **public web and open-data sources only** — exchange publications, regulator publications and open-data portals — with no redistribution of licensed data. When a needed dataset is not publicly available under acceptable terms, the answer is the Section 8.3 playbook (vendor licence or official API), not evasion.

### 11.2 The Source Portfolio

The reference portfolio below is the *design's* source set, not a verified inventory: source classes and access paths are real and current as described, but specific endpoints, licence texts and rate limits are flagged ⚠ and listed again in Section 13 where not re-verified this pass.

| Source class | Example sources | Cadence | Access path | Compliance posture |
|---|---|---|---|---|
| Exchange public data | SGX website: daily trading statistics, listed-company announcements (public pages), historical data files | Daily; announcements intraday | robots.txt + sitemaps; named UA | Internal-use consumption only; redistribution requires an exchange data licence ⚠ |
| Regulator publications | MAS press releases, speeches, consultation papers, notices and guidelines | Daily (press releases) | robots.txt + sitemaps; conservative budget | Public-sector data; attribution expected |
| Singapore open data | data.gov.sg datasets (economy, prices, demographics) | Monthly snapshots | Documented open-data API | Open-licence terms per dataset |
| International open data | IMF and World Bank open-data portals | Monthly | Official APIs | Open licences; attribution |
| Company-registry reference | ACRA public-register lookups | Weekly | Official portal, per-lookup | Portal terms prohibit bulk harvesting ⚠ — per-lookup only |

### 11.3 The Compliance Posture

The Section 9.7 checklist, applied:

- **Named, documented user-agent** — `CymbalDataBot/1.0` (the §4.2 example), with a contact URL, so source operators can reach the platform team.
- **robots.txt and sitemaps first** — sitemap-driven discovery (§4.2) before any link-following; robots.txt snapshotted per fetch for the audit trail.
- **Conservative per-source budgets** (§4.3) — a regulator's site gets ~10 requests/minute; open-data APIs run at their documented limits; headroom is by design.
- **The personal-data screen** (§9.5) — automated screening at the transform stage; any dataset containing personal data is quarantined and routed to the data-protection officer under the PDPA. The platform is designed for non-personal, public-sector and market data.
- **ToS and licence review** per source class before onboarding, with legal sign-off recorded.
- **The audit trail** — request logs, robots.txt snapshots, rate-budget configs and onboarding decisions are retained: the §6.6 politeness dashboards double as compliance evidence for internal audit and MAS-aligned governance (cross-ref the sibling [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) guide).

### 11.4 The Freshness SLAs

| Dataset class | SLA | Mechanism |
|---|---|---|
| SGX listed-company announcements | Detected within 15 minutes of publication (market hours) | 5-minute re-check of the announcement index, politeness permitting |
| SGX daily trading statistics | T+1 by 08:00 SGT | Nightly crawl DAG |
| MAS press releases | Within 2 hours of publication | 30-minute checks, 07:00–22:00 SGT |
| MAS consultation papers and notices | Within 24 hours | Daily crawl DAG |
| data.gov.sg snapshots | Monthly, first business day | Monthly refresh DAG |
| ACRA reference lookups | Weekly | Weekly refresh DAG |

SLA enforcement uses the Section 6.4 machinery: each DAG declares its freshness deadline, Airflow's alerting pages the on-call when `crawl_mas_daily` misses its 07:00 SGT completion target, and §6.6 monitoring tracks time-since-last-successful-fetch per source.

### 11.5 The Pipeline

The design maps one-to-one onto Section 6: seed URLs come from sitemaps; **one DAG per source class** (`crawl_mas_daily`, `crawl_sgx_announcements`, `refresh_open_data_monthly`); the Section 6.3 topics carry `urls.to_fetch → fetched.pages → parsed.docs → extracted.records`; storage is **WARC in `landing/`** (immutable, with SHA-1 digests for content dedup and replay — §6.5) and **Parquet in `curated/`**, partitioned by source and date. Failed runs are **replayed from the WARC archive** rather than re-fetched (§6.4) — politeness and reproducibility in one rule. Announcement cross-posting between sources is collapsed with SimHash near-duplicate detection (§5.3).

### 11.6 The Distribution Boundary — DDS Cross-Ref

The curated Parquet lands in the lake (the lakehouse pattern of the sibling [Cloud Object Storage and Lakehouse](cloud_object_storage_lakehouse_guide.md) and [Delta Lake vs Iceberg](data/delta_lake_vs_iceberg.md) guides); the *real-time* distribution layer is the DDS-based bus documented in the sibling [DDS Guide](dds_guide.md) §10. The boundary contract:

- **The acquisition platform** is a *batch producer at the lake edge*: at-least-once, replayable, politeness-bound.
- **The DDS bus** carries *derived real-time signals* — announcement-detected events, intraday price statistics — to trading and research consumers with DDS QoS guarantees (reliability, deadline, durability — see the DDS guide §5).
- The two never share a queue: the lake is the system of record; DDS is the delivery fabric for time-critical derivations.

### 11.7 The Operating Model

- **Runbooks per source** — including the blocked-host register (§6.6): a 403 or challenge escalates to the source owner, not to evasion (§8.3).
- **Source-health scorecards** — success rates, freshness-vs-SLA, politeness compliance per source, reviewed weekly.
- **Quarterly compliance review** with legal — ToS changes, licence renewals, new sources, and the personal-data screen's outcomes.
- **Annual re-verification** of every source's terms and access path — the web changes; the compliance posture must be re-derived from it.

---

## 12. Claims Audit — Verified, Flagged, Unverified

Following repository convention, every verifiable claim in this guide is marked in the text; this section consolidates them. The ✅ table records claims verified this pass against a primary or cited source (URL given); the ⚠ table records flagged or approximate claims; the ❌ table records claims checked and found incorrect. Everything this pass could *not* confirm is collected in Section 13.

### 12.1 The Verified Claims (✅)

| Claim | Source |
|---|---|
| robots.txt defined by Martijn Koster in 1994; consensus reached 30 June 1994 | robotstxt.org/orig.html; RFC 9309 abstract |
| RFC 9309 ("Robots Exclusion Protocol"), IETF Standards Track, published September 2022 | rfc-editor.org/rfc/rfc9309.html |
| CFAA: 18 U.S.C. § 1030 expanded by Pub. L. 99-474, signed 16 October 1986 (origin: 1984) | uscode.house.gov/statutes/pl/99/474.pdf; en.wikipedia.org/wiki/Computer_Fraud_and_Abuse_Act |
| *Van Buren v. United States*, 141 S. Ct. 1648, decided 3 June 2021 | Wikipedia (*Van Buren*), cited in the hiQ record |
| hiQ v. LinkedIn: cease-and-desist letters dated 23 May and 24 June 2017 | District-court record (Justia) |
| hiQ v. LinkedIn: preliminary injunction granted 14 August 2017 by Judge Edward Chen (273 F. Supp. 3d 1099) | PR Newswire; WSGR; PACER docket 17-cv-03301 |
| Ninth Circuit opinion 9 September 2019, 938 F.3d 985 — public, unauthenticated data is not "without authorisation" | law.justia.com (9th Cir. 17-16783); eff.org |
| Supreme Court GVR 14 June 2021 (141 S. Ct. 2752) in light of *Van Buren* | Wikipedia; Reuters, 14 June 2021 |
| Ninth Circuit reaffirms on remand 18 April 2022, 31 F.4th 1180 | cdn.ca9.uscourts.gov (opinion 17-16783); TechCrunch, 18 April 2022 |
| District court finds hiQ breached LinkedIn's User Agreement; settlement reached (reported ~December 2022, terms confidential) | The National Law Review; Wikipedia |
| GDPR: Regulation (EU) 2016/679 adopted 27 April 2016, applicable 25 May 2018 | eur-lex.europa.eu/eli/reg/2016/679/oj/eng |
| CNIL fined Clearview AI €20 million, decision published 20 October 2022 | cnil.fr; Hunton Privacy & Cybersecurity Law Blog |
| PDPA: Singapore Act 26 of 2012; main data-protection obligations in force 2 July 2014 | sso.agc.gov.sg/Act/PDPA2012; pdpc.gov.sg |
| C4 corpus: Raffel et al., arXiv:1910.10683, submitted 23 October 2019 | arxiv.org/abs/1910.10683 |
| The Pile: Gao et al., arXiv:2101.00027, submitted 31 December 2020; 825 GiB; 22 subsets | arxiv.org/abs/2101.00027 |
| BackRub started 1996 (Page & Brin, Stanford); Google launched 1998; Google News launched 2002 | en.wikipedia.org/wiki/History_of_Google |
| Scrapy created in Insophia, released open source August 2008; Scrapinghub founded 2010 | zyte.com history posts; scrapy.org (§7.1) |
| Playwright v1.0.0 published 6 May 2020 | api.github.com release record (§7.2) |
| Puppeteer first npm release v0.9.0, 16 August 2017 | registry.npmjs.org/puppeteer (§7.3) |
| Beautiful Soup started 2004 by Leonard Richardson | crummy.com (§7.4) |
| Colly GitHub repository created 29 September 2017 | api.github.com/repos/gocolly/colly (§7.5) |
| Nutch: 100M-page system by June 2003; Apache Incubator January 2005; Lucene subproject June 2005; independent TLP April 2010; adopted by Common Crawl February 2014 | en.wikipedia.org/wiki/Apache_Nutch; nutch.apache.org (§7.6) |
| Common Crawl: non-profit founded 2007 by Gil Elbaz; open archive since 2008, 10+ PB; monthly crawls of 2B+ pages | commoncrawl.org/about (§7.7) |

### 12.2 The Flagged Claims (⚠)

| Claim | Flag |
|---|---|
| Scrapinghub → Zyte rebrand year (2019 widely reported; exact month unconfirmed; earliest Wayback snapshot of the announcement post is February 2021) | §7.1 |
| WARC standardised as ISO 28500 (first edition 2009, current 2017) | §6.5 ⚠-knowledge |
| C4 released size ~800 GB | Dataset-card figure, not re-read from the paper this pass |
| CamelCamelCamel founded 2008 | Press/industry reports, not re-verified at a primary source |
| GPT-3's largest training bucket was a filtered Common Crawl | Brown et al. 2020, not re-verified this pass |
| Mercator crawler architecture (Najork & Heydon, 2001, Compaq SRC) | §3.3 ⚠-knowledge |
| Cho & Garcia-Molina uniform/proportional revisit policies | §3.4 ⚠-knowledge |
| SimHash (Moses Charikar, STOC 2002) | §5.3 ⚠-knowledge |
| Bloom filter analysis (Burton H. Bloom, CACM 1970) | §5.4 ⚠-knowledge |
| *Feist v. Rural* (1991); EU Database Directive 96/9/EC | §9.6 ⚠-knowledge |
| SGX market-data redistribution licence specifics; data.gov.sg rate limits; ACRA bulk-harvest prohibition | §11.2 — illustrative design assumptions |
| SGX announcement detection latency (15-minute SLA) | §11.4 — a target, not a measured figure |

### 12.3 The Rejected Claims (❌)

| Claim | Verdict | Basis |
|---|---|---|
| "The Ninth Circuit ruled for hiQ in 2017" | ❌ — 2017 saw only the district court's preliminary injunction (14 August 2017); the Ninth Circuit's operative decision is 9 September 2019 (938 F.3d 985) | Case record (§9.4) |
| "Nutch reached Apache top-level status in ~2005" | ❌ — it graduated from the Incubator in June 2005 as a Lucene *subproject*; independent top-level status came in April 2010 | Wikipedia/Nutch history (§7.6) |
| "hiQ won the case outright" | ❌ — hiQ won the CFAA/public-data question (2019, reaffirmed 2022), but the district court found in November 2022 that hiQ breached LinkedIn's User Agreement, and the parties settled without a merits ruling on the contract claim | §9.4 record |

---

## 13. What Could Not Be Verified

This section collects everything this pass could not confirm against a primary or reliable secondary source, so the reader can separate verified fact from honest uncertainty:

1. **The exact month of the Scrapinghub → Zyte rebrand.** The rebrand is widely reported as 2019; the company's own history describes it as "10 years ago" relative to the 2010 founding; the earliest Wayback snapshot of the rebrand announcement post is February 2021. Year-level ✅, month-level ⚠ (§7.1).
2. **C4's exact byte size.** The commonly cited ~800 GB figure comes from dataset cards; the paper's tables were not re-read this pass (§10.4).
3. **The hiQ v. LinkedIn settlement terms** — confidential, per the December 2022 reporting (§9.4).
4. **hiQ Labs' operational status at the time of settlement.** The company's business status after the litigation years was not re-verified this pass; nothing in this guide assumes it.
5. **SGX market-data redistribution licence specifics** — the worked example assumes internal-use consumption; the exact licence regime for redistribution was not re-verified (§11.2).
6. **data.gov.sg API rate limits and dataset terms** — the worked example's budget is illustrative (§11.2, §13).
7. **Search-engine crawler scale** — Google/Bing index sizes and crawl rates are not publicly documented in verifiable detail (§10.1).
8. **The measured latency of SGX announcement publication-to-web** — the 15-minute SLA is a design target, not a measured figure (§11.4).
9. **ISO 28500 current-edition year (2017)** — flagged ⚠-knowledge in §6.5; not re-verified against the ISO catalogue this pass.
10. **Google News's current news-content licensing arrangements** (§10.3).
11. **ACRA bulk-harvest prohibition specifics** — the per-lookup-only posture is the conservative reading; the portal's exact terms were not re-verified (§11.2).

---

## 14. Glossary

| Term | Definition |
|---|---|
| URL frontier | The set of known-but-unfetched URLs plus the scheduling machinery that decides what to fetch next (Section 3) |
| Politeness | The discipline of never requesting from a host faster than its allowed rate (Section 4) |
| robots.txt | The Robots Exclusion Protocol — `/robots.txt`, originated 1994 by Martijn Koster, standardised 2022 as RFC 9309 |
| Crawl-delay | The robots.txt extension asking for a minimum delay in seconds between requests from a named bot |
| Sitemap | An XML inventory of a site's URLs — the sanctioned way to learn a site's URL set |
| Canonical URL | The normalised form of a URL after canonicalisation rules are applied (Section 5.2) |
| Bloom filter | The space-efficient probabilistic set used as the fast-negative filter of the dedup layer (Section 5.4) |
| SimHash | Charikar's locality-sensitive hashing scheme for near-duplicate document detection (Section 5.3) |
| WARC | Web ARChive — the ISO 28500 archival format for crawl payloads with digests (Section 6.5) |
| WAT / WET | Web Archive Transformation files — derived metadata (WAT) and plain-text extractions (WET) |
| Parquet | The columnar file format used for the curated, queryable record layer (Section 6.5) |
| Extraction contract | The versioned schema that isolates scraping from the rest of the pipeline (Section 6.2) |
| Bot management | Commercial detection layers (Cloudflare, Akamai, DataDome) that classify requests as human/bot (Section 8) |
| CFAA | The Computer Fraud and Abuse Act of 1986, 18 U.S.C. § 1030 — the US computer-crime statute at the centre of scraping law |
| DMCA | The Digital Millennium Copyright Act (1998) — anti-circumvention and takedown regime |
| GDPR | Regulation (EU) 2016/679 — the EU's data-protection regulation, applicable from 25 May 2018 |
| PDPA | Singapore's Personal Data Protection Act 2012 — main rules in force 2 July 2014 |
| PDPC | Personal Data Protection Commission — Singapore's data-protection authority |
| Preliminary injunction | Interim court relief — the procedural device at the centre of hiQ v. LinkedIn |
| C4 | Colossal Clean Crawled Corpus — the Common Crawl-derived pre-training corpus of the T5 paper (2019) |
| The Pile | The 825 GiB, 22-subset English corpus of Gao et al. (2020), including a Common Crawl subset |
| Common Crawl | The non-profit (founded 2007 by Gil Elbaz) that publishes open web crawls of 2B+ pages |
| DDS | Data Distribution Service — the OMG publish-subscribe middleware; Cymbal Bank's market-data bus (see the sibling DDS guide) |
| MAS | Monetary Authority of Singapore — Singapore's central bank and financial regulator |
| Cymbal Bank | The repository's bank persona — the only bank persona in this guide |

---

## 15. Cross-References and Further Reading

**Repository guides (banking — prefix `../banking/`):**
- [Apache Kafka Guide](../banking/kafka_guide.md) — the messaging backbone behind the queue-at-the-centre design (§6.3)
- [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) — the Singapore regulatory overlay and the Cymbal Bank persona conventions (§9.5, §11.3)

**Repository guides (technology — sibling plain filenames):**
- [Apache Airflow Guide](apache_airflow_guide.md) — the reference orchestrator; DAG authoring, SLA and alerting (§6.4)
- [Enterprise Middleware Integration Platform](enterprise_middleware_integration_platform_guide.md) — backpressure, DLQs and the broker-vs-bus comparison (§6.3)
- [Cloud Object Storage and Lakehouse](cloud_object_storage_lakehouse_guide.md) — the lakehouse pattern the acquisition platform writes into (§6.5)
- [Delta Lake vs Iceberg](data/delta_lake_vs_iceberg.md) — the table formats for the curated layer (§6.5)
- [DDS Guide](dds_guide.md) — the data-centric bus; its §10 is the Cymbal Bank trading-floor market-data worked example that Section 11.6 feeds
- [AI Governance, Bias and Red-Teaming](ai_llm/ai_governance_bias_redteaming_guide.md) — the AI-governance angle on training-data acquisition (§10.4)
- [LLM Optimization: The Complete Guide](ai_llm/llm_optimization_complete_guide.md) — the training and inference side of LLM pipelines (§10.4)

**Primary sources used this pass (verified claims in Section 12):**
- robotstxt.org/orig.html — the 30 June 1994 consensus text; rfc-editor.org/rfc/rfc9309.html — RFC 9309 (September 2022)
- uscode.house.gov/statutes/pl/99/474.pdf — Pub. L. 99-474 (CFAA of 1986, 16 October 1986)
- law.justia.com and cdn.ca9.uscourts.gov — the 9th Circuit opinions in *hiQ v. LinkedIn* (938 F.3d 985; 31 F.4th 1180); PR Newswire and WSGR (14 August 2017 injunction); Reuters (14 June 2021 GVR); The National Law Review (December 2022 settlement)
- eur-lex.europa.eu/eli/reg/2016/679/oj/eng — GDPR; sso.agc.gov.sg/Act/PDPA2012 — Singapore PDPA; cnil.fr — the Clearview AI €20M fine (October 2022)
- arxiv.org/abs/1910.10683 — C4 / T5 paper (Raffel et al., 2019); arxiv.org/abs/2101.00027 — The Pile (Gao et al., 2020)
- en.wikipedia.org/wiki/History_of_Google — BackRub (1996), Google (1998), Google News (2002)
- Tooling sources per §7.1–§7.7: zyte.com (Scrapy/Scrapinghub history), scrapy.org, github.com/scrapy/scrapy, api.github.com (Playwright v1.0.0; Colly repo), registry.npmjs.org/puppeteer, infoq.com (Puppeteer), crummy.com (Beautiful Soup), nutch.apache.org, commoncrawl.org/about
- Wikipedia (hiQ Labs v. LinkedIn; Computer Fraud and Abuse Act; Apache Nutch; Common Crawl; Gil Elbaz; Beautiful Soup) — used for cross-checking and citation chains into primary sources

### Closing

Large-scale web data acquisition is a discipline of *constraints turned into architecture*: the URL frontier turns politeness into a scheduling problem; the dedup layer turns memory into completeness; the queue turns bursty discovery into a governed stream; the WARC-and-Parquet archive turns a crawl into a replayable asset; and the legal boundaries — the 1994 robots.txt norm now formalised as RFC 9309, the 1986 CFAA and the decade of hiQ v. LinkedIn litigation that defined how it applies to public data, the GDPR and Singapore's PDPA, and the copyright and contract layer — turn compliance into a design requirement rather than an afterthought. The verified tooling landscape gives the architect working, dated, proven components from Scrapy's 2008 open-source release to Common Crawl's two-billion-page monthly harvests; the large-scale cases show where the discipline came from — search indexes, price monitoring, news aggregation, LLM corpora — and the Cymbal Bank worked example shows how a regulated firm turns all of it into a governed market-data and regulatory-data platform. Crawl politely, deduplicate ruthlessly, archive everything, and keep the harvest at scale.
