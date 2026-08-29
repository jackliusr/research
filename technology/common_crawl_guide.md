# Common Crawl — the Open Web's Memory

Jack Liu Shurui, Solution Architect

> **Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank
> **Topic:** Common Crawl — the non-profit foundation, the corpus, the data formats (WARC/WAT/WET), the URL indexes, the free-access hosting model, the LLM-training lineage from C4 to RedPajama and RefinedWeb, the quality and limitations, the alternatives and ecosystem, the condensed legal and ethical posture, and the Cymbal Bank LLM/RAG corpus-strategy worked example
> **Audience:** Solution Architects, Data Platform Engineers, Data Engineering Leads, AI/LLM Platform Teams, Research Engineers, Compliance and Legal Technology Teams, Banking IT Decision-Makers
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Version:** 1.0 — August 2026

---

# Table of Contents

1. [Executive Summary — The Open Web's Memory](#1-executive-summary--the-open-webs-memory)
2. [The Organization — the Common Crawl Foundation](#2-the-organization--the-common-crawl-foundation)
3. [The Corpus — Scale, Cadence, and History](#3-the-corpus--scale-cadence-and-history)
4. [The Data Formats — WARC, WAT, WET, and Friends](#4-the-data-formats--warc-wat-wet-and-friends)
5. [The Indexes, Access, and Hosting — Querying the Free Corpus](#5-the-indexes-access-and-hosting--querying-the-free-corpus)
6. [LLM Training Usage — the C4 Lineage and Beyond](#6-llm-training-usage--the-c4-lineage-and-beyond)
7. [Academic Research and Web Analytics](#7-academic-research-and-web-analytics)
8. [Quality and Limitations — What the Crawl Is Not](#8-quality-and-limitations--what-the-crawl-is-not)
9. [The Alternatives and Ecosystem](#9-the-alternatives-and-ecosystem)
10. [Legal and Ethics — Condensed](#10-legal-and-ethics--condensed)
11. [Cymbal Bank Worked Example — an LLM/RAG Corpus Strategy](#11-cymbal-bank-worked-example--an-llmrag-corpus-strategy)
12. [Claims Audit — Verified, Flagged, Unverified](#12-claims-audit--verified-flagged-unverified)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)

---

## 1. Executive Summary — The Open Web's Memory

### Purpose of This Guide

This guide is the repository's **platform deep-dive on Common Crawl**: the California non-profit that has, since 2007, been quietly archiving the open web and giving the result away. Where the sibling [Large-Scale Web Data Acquisition guide](large_scale_web_data_acquisition_guide.md) covers the *engineering* of crawling, scraping and fetching at scale — the URL frontier, politeness, deduplication, tooling, and the legal boundaries — this guide covers the *archive itself*: the organization behind it, the corpus and its formats, the indexes that make it queryable, the free-access hosting model, the LLM-training lineage built on top of it, its quality and limitations, the alternatives, and a regulated-bank worked example for using it as an LLM/RAG corpus source.

The guide answers six questions an architect actually asks:

1. **Who runs it, and why can we trust it?** — the Common Crawl Foundation's 501(c)(3) status, founding, mission, governance and funding model (Section 2).
2. **What is in the corpus, and how is it structured?** — scale, crawl cadence, the `CC-MAIN-YYYY-WW` naming, and the WARC/WAT/WET formats and indexes (Sections 3–5).
3. **How do I get at the data?** — the S3 open-data hosting, the CDXJ and URL-index query layers, Athena/Spark access (Section 5).
4. **Why does every LLM lab use it?** — the C4 lineage and the open LLM corpora built on Common Crawl (Section 6).
5. **What are its limits?** — coverage, noise, deduplication, biases, freshness, and the controversies (Section 8).
6. **What should a regulated bank do with it?** — the Cymbal Bank LLM/RAG corpus-strategy worked example (Section 11).

### The One-Paragraph Answer

Common Crawl is a 501(c)(3) non-profit, founded in 2007 by Gil Elbaz, that maintains a free, open repository of web crawl data collected since 2008 — today the homepage headline reads **over 300 billion pages spanning 19 years, with 3–5 billion new pages added each month** ✅ (commoncrawl.org, retrieved August 2026). Crawls are published roughly monthly under names like `CC-MAIN-2026-34` (the August 2026 crawl alone holds 2.14 billion pages and 84.78 TiB of compressed WARC data ✅ — data.commoncrawl.org). The raw bytes are stored as **WARC** (the ISO 28500 web-archive standard), with derived **WAT** metadata and **WET** plain-text extracts alongside, and the whole corpus is queryable through a **CDXJ URL index** and a columnar **URL Index** (formerly the Columnar Index) usable from Amazon Athena and Spark. Access is free — the data lives in AWS's Public Data Sets program (bucket `s3://commoncrawl/`) plus academic clouds, and since April 2026 also a Hugging Face storage bucket. Google built its **C4** corpus from a Common Crawl snapshot for the T5 model; RedPajama, Falcon's RefinedWeb, The Pile and most open LLM corpora followed; ChatGPT-era training runs leaned on it heavily. The trade-offs are real: it is a *sample* of the web, not the web; it is noisy (spam, boilerplate, machine-generated text); it is English-heavy and developed-world-skewed; and its robots.txt/paywall posture has been publicly challenged. For a bank, that means: use the corpus for background and regulatory-text enrichment, filter and deduplicate aggressively, keep a provenance register, and never treat it as a licensed data source.

### Integrity Convention

Following repository convention, every verifiable claim is marked: ✅ = verified this pass against a primary or cited source (URL in Section 12); ⚠ = flagged, approximate, dated, or well-documented industry knowledge not re-verified live; ❌ = a claim checked and found incorrect, with the correction stated. No dates, citations or statistics were invented; anything that could not be confirmed is flagged honestly in Section 13. The guide was researched in August 2026 against commoncrawl.org, data.commoncrawl.org, index.commoncrawl.org, ISO, Wikipedia, TensorFlow Datasets, Hugging Face dataset cards, archive.org, and the sibling acquisition guide.

---

## 2. The Organization — the Common Crawl Foundation

### 2.1 Non-Profit Status and Founding

- **501(c)(3) non-profit.** Common Crawl states it plainly on its own homepage: "Common Crawl is a 501(c)(3) non-profit founded in 2007." ✅ *Sources: commoncrawl.org; commoncrawl.org/about; Wikipedia (Common Crawl); ProPublica Nonprofit Explorer lists the foundation under EIN 26-1635908, with IRS Form 990 filings publicly available.*
- **Founded 2007 by Gil Elbaz.** The foundation was created in 2007 in San Francisco by Gil Elbaz, the entrepreneur who co-founded Applied Semantics (the company behind AdSense, acquired by Google in 2003 for US$102 million) and later founded the open-data company Factual (merged into Foursquare in 2020). ✅ *Sources: commoncrawl.org/about; Wikipedia (Gil Elbaz; Common Crawl).*
- **Timeline nuance.** The foundation began archiving websites in 2007 and maintains an open repository "collected since 2008" (about page); per Wikipedia it *began publishing its crawls in 2011*, and the index server still lists the earliest collections as `CC-MAIN-2008-2009` and `CC-MAIN-2009-2010` ARC files. So: founded 2007, collecting since 2008, publicly distributed since ~2011. ✅ *Sources: Wikipedia (Common Crawl); index.commoncrawl.org.* ⚠ The brief for this guide mentioned a 2023 death of Gil Elbaz; **no obituary or primary confirmation could be found this pass** — see Section 13, item 1. The claim is NOT asserted as fact here.

### 2.2 The Mission

The foundation's mission is to **democratize access to web data**: "dedicated to providing a copy of the Internet to Internet researchers, companies and individuals at no cost for the purpose of research and analysis" (FAQ), and "we make wholesale extraction, transformation and analysis of open web data accessible to researchers" (homepage). In the founder's words (Los Angeles Times, 2012, quoted in Wikipedia): the foundation "seeks to make a copy of the Web accessible to a data scientist, or to a start-up or to a researcher or to an analyst that just wants to improve the world." ✅ *Sources: commoncrawl.org; Wikipedia.*

The founding motivation is historical: in the early 2010s the only comparable web-scale datasets belonged to search engines (Google above all), and researchers had no neutral, bulk-accessible alternative. Common Crawl was designed — per MIT Technology Review (2013) — to have more and fresher data, more efficiently analyzable, than the Wayback Machine's on-demand archives. ✅ *Sources: Wikipedia; MIT Technology Review, 23 January 2013.*

### 2.3 Governance and People

- **Executive director:** Rich Skrenta (Wikipedia infobox says "managing director"; the article body says "executive director"). ✅ *Source: Wikipedia (Common Crawl).*
- **Advisory board:** Peter Norvig and Joi Ito have served on the advisory board. ✅ *Source: Wikipedia, citing MIT Technology Review (2013).*
- **Board of directors:** Carl Malamud (of public.resource.org) and Nova Spivack are listed alongside Elbaz as board members in the Gil Elbaz Wikipedia article — that passage carries a citation-needed tag, so treat membership details as ⚠-level.
- **Team:** a small engineering team publishes crawl announcements, statistics and research blog posts (e.g., Senior Research Engineer Michael Paris, co-author of the 2026 paper "What the crawler keeps, and what it loses" featured on the homepage). ✅ *Source: commoncrawl.org homepage.*

### 2.4 Funding and Sponsorship — Verified Facts and Honest Gaps

The funding model is one of the least transparent parts of the organization, and this guide flags rather than fabricates:

- **Elbaz Family Foundation Trust.** Per Wikipedia (citing The Atlantic's November 2025 investigation), Common Crawl "received funding almost exclusively from the Elbaz Family Foundation Trust until 2023." ✅-via-secondary-source.
- **AI-industry donations from 2023.** In 2023 the foundation "began receiving significant financial support from AI companies, including Anthropic and OpenAI, each of which donated $250,000." ✅-via-secondary-source (Wikipedia, citing The Atlantic).
- **AWS in-kind support.** Wikipedia (citing *American Scientist*, 2015) reports AWS provided some services free, "allowing computing costs to average $2–4000/month" — a **2015** figure, flagged ⚠-dated, not a current cost statement.
- **Public filings exist.** The foundation files IRS Form 990s, publicly available via ProPublica's Nonprofit Explorer (EIN 26-1635908). ✅ *This pass did not open the 990s; exact current revenue, sponsor names and amounts are therefore listed in Section 13, item 2.* ⚠
- **Sponsor list.** Beyond the named 2023 donations, the foundation does not publish a full, current sponsor roster on its public pages as of this pass — ⚠ not verified; see Section 13.

The takeaway for a downstream consumer: the *corpus* is open and its license posture permissive (Terms of Use), but the *organization's* finances are not fully public; the 2023–2025 era saw both AI-lab sponsorship and, per The Atlantic (November 2025), accusations that the foundation's paywall and removal claims did not match its actual data. Both matter to a bank's provenance and reputational review (Sections 8 and 11).

---

## 3. The Corpus — Scale, Cadence, and History

### 3.1 Scale — Current Published Figures

The figures below are what the organization itself publishes, with dates, because they move:

| Figure | Value | Source & status |
|---|---|---|
| Pages in the open repository, all time | **Over 300 billion pages spanning 19 years** | commoncrawl.org homepage, retrieved August 2026 ✅ |
| New pages added each month | **3–5 billion** | commoncrawl.org homepage, retrieved August 2026 ✅ |
| Research papers citing the corpus | **Over 10,000** | commoncrawl.org homepage; Wikipedia (as of 2024) ✅ |
| Corpus size (about page, older wording) | "More than 10 petabytes… each crawl typically containing more than two billion web pages" | commoncrawl.org/about ⚠-dated wording; the sibling guide's §7.7 cites the same |
| August 2026 crawl (`CC-MAIN-2026-34`) | **2.14 billion pages**; 84.78 TiB compressed WARC | data.commoncrawl.org/crawl-data/CC-MAIN-2026-34 ✅ primary |
| April 2023 crawl era | ~3.1 billion webpages captured | Wikipedia, citing a 2023 study ⚠-secondary |
| 2015 corpus | 1.8 billion webpages | Wikipedia, citing *American Scientist* (2015) ⚠-dated |
| 2012 corpus | 210 TB | cc-crawl-statistics page, referencing the 2012 corpus analysis ✅ |

A reconciliation note the architect should keep in mind: the homepage's "3–5 billion new pages per month" is a *corpus-wide* growth figure; an individual monthly crawl typically contains ~2–3 billion pages (2.14B for August 2026). The two numbers are not contradictory — the corpus includes the monthly crawls plus the daily CC-NEWS stream and other collections — but citing "3–5 billion per crawl" would be wrong. ❌-check: see Section 12.3.

### 3.2 Crawl Schedule and the CC-MAIN Naming

- **Monthly cadence.** Crawls are published approximately once a month. The index server's collection list shows the modern era at roughly one crawl per calendar month (2015: 12 crawls; 2016: 12; 2017: 12; …; 2024: 11; 2025: 12; 2026 through August: 8). ✅ *Source: index.commoncrawl.org collection list.*
- **Naming.** Modern crawls are named **`CC-MAIN-YYYY-WW`**, where `YYYY` is the year and `WW` is the ISO week number of the crawl's start (e.g., `CC-MAIN-2026-34` for the August 2026 crawl). ✅ *Source: data.commoncrawl.org crawl archive listings.*
- **Early era (2008–2013) — the ARC years.** The collection list retains the ancestors: `CC-MAIN-2008-2009`, `CC-MAIN-2009-2010`, `CC-MAIN-2012` (the 210 TB corpus), `CC-MAIN-2013-20` (Summer 2013), `CC-MAIN-2013-48` (Winter 2013). These were ARC-format crawls — pre-WARC — and are still downloadable. ✅ *Source: index.commoncrawl.org.*
- **2014–2015 — the transition.** 2014 saw ~11 crawls and 2015 saw the naming settle into the week-number scheme; Nutch was adopted as the crawl engine in February 2014 (see the sibling guide §7.6). ✅ *Sources: index.commoncrawl.org; sibling guide verified record.*
- **Announcement cadence.** Each crawl gets a blog announcement and a listing page (e.g., "Common Crawl August 2026 Crawl Archive (CC-MAIN-2026-34)"). ✅ *Source: data.commoncrawl.org.*

### 3.3 What a Crawl Contains

Each crawl archive ships a set of path-listing files (`*.paths.gz`) pointing at the actual objects. For `CC-MAIN-2026-34` (August 2026), the listing is: ✅ *Source: data.commoncrawl.org/crawl-data/CC-MAIN-2026-34/index.html — primary, retrieved August 2026.*

| Collection | Files | Compressed size |
|---|---|---|
| Segments | 100 segment manifests | — |
| WARC (raw crawl payloads) | 100,000 files | 84.78 TiB |
| WAT (metadata/derivatives) | 100,000 files | 13.92 TiB |
| WET (plain-text extractions) | 100,000 files | 5.84 TiB |
| robots.txt snapshots | 100,000 files | 0.14 TiB |
| Non-200 responses | 100,000 files | 2.63 TiB |
| URL index files (CDXJ) | 302 files | 0.22 TiB |
| Columnar URL index files | 900 files | 0.20 TiB |

Two practical implications: (1) the WARC layer dominates cost — WET is ~7% of WARC size, which is why text-only consumers start from WET; (2) the robots.txt and non-200 collections are first-class deliverables, which matters for compliance auditing (Section 10).

### 3.4 Sampling, Not Completeness

The FAQ is explicit: "Common Crawl's dataset is a sample of the web, and we do not generally archive any entire website but a randomly selected subset of it." The crawler follows links and honors sitemaps, but the corpus is a *sample* — approximately 2–3 billion pages per month out of a web far larger. ✅ *Source: commoncrawl.org FAQ.* This single fact drives most quality expectations (Section 8): coverage is broad but not complete, and per-site depth is limited. The 2026 homepage-featured paper "What the crawler keeps, and what it loses" (Paris, Celikkanat, Foppiano — arXiv:2607.13636) is the organization's own longitudinal study of exactly this persistence/coverage question. ✅ *Source: commoncrawl.org homepage (full arXiv URL shown).*

### 3.5 Reading the Crawl Statistics

The `cc-crawl-statistics` project publishes, for every crawl, a consistent set of metrics computed from the URL index: ✅ *Source: commoncrawl.github.io/cc-crawl-statistics.*

- **Crawl size** — number of pages, unique URLs, hosts, domains and top-level domains (public suffixes), plus cumulative corpus growth over time;
- **Top-level domain distribution** — which TLDs dominate (the classic picture: `.com` first by a wide margin, then `.org`, `.net`, country codes);
- **Top-500 registered domains** — the long-tail concentration of the crawl;
- **Crawler metrics** — fetch status codes, response sizes, and the robots.txt outcomes;
- **Crawl overlap** — how much of each month's crawl repeats the previous month's URLs (the input to cross-crawl deduplication, §8.3);
- **Content distribution** — media types (MIME), character encodings, and languages per crawl.

The project itself acknowledges its lineage: it is "inspired by Sebastian Spiegler's *Statistics of the Common Crawl Corpus 2012*," which analyzed the **210 TB 2012 web corpus** — a useful historical baseline for how much the corpus has grown since. ✅ *Source: commoncrawl.github.io/cc-crawl-statistics.* An architect planning a multi-crawl study should start here, not with ad-hoc queries: the language and overlap pages answer "which crawls do I actually need?" before any WARC is downloaded.

### 3.6 A Corpus Timeline

| Year | Milestone | Status |
|---|---|---|
| 2007 | Foundation established (San Francisco) | ✅ commoncrawl.org; Wikipedia |
| 2008 | Archiving begins; earliest collections (`CC-MAIN-2008-2009`) | ✅ commoncrawl.org/about; index.commoncrawl.org |
| 2009–2010 | Second ARC-era collection | ✅ index.commoncrawl.org |
| 2011 | Public crawl publication begins | ✅ Wikipedia |
| 2012 | 210 TB corpus analyzed | ✅ cc-crawl-statistics reference |
| 2013 | Two crawls (Summer `2013-20`, Winter `2013-48`); MIT Tech Review coverage | ✅ index.commoncrawl.org; Wikipedia |
| 2014 | Nutch adopted (February); ~11 crawls; the monthly cadence takes shape | ✅ sibling guide §7.6; index.commoncrawl.org |
| 2015 | Monthly cadence established; URL index announced (April); 1.8B pages in the corpus | ✅ index.commoncrawl.org; Wikipedia |
| 2016 | Full year of 12 monthly crawls | ✅ index.commoncrawl.org |
| 2019 | T5/C4 built from a Common Crawl snapshot (October) | ✅ arXiv:1910.10683 |
| 2023 | AI-lab donations (Anthropic, OpenAI); CCBot blocking accelerates; C4 content controversy | ✅/⚠ Wikipedia |
| 2024 | 10,000+ citations; publishers target Common Crawl (Wired) | ✅/⚠ Wikipedia |
| 2025 | The Atlantic investigation (November) | ⚠ Wikipedia |
| 2026 | Homepage headline: 300B+ pages / 19 years; Hugging Face mirror (April); web-graph releases | ✅ commoncrawl.org; Wikipedia |

## 4. The Data Formats — WARC, WAT, WET, and Friends

### 4.1 WARC — the Web ARChive Format (ISO 28500)

**WARC (Web ARChive)** is the archival format at the heart of the corpus: one file, many concatenated records, each record carrying a set of simple text headers plus an arbitrary data block. The ISO standard states its purpose precisely: "to store both the payload content and control information from mainstream Internet application layer protocols, such as the HTTP, DNS, and FTP; to store arbitrary metadata linked to other stored data…; to support data compression and maintain data record integrity." ✅ *Source: ISO 28500:2017, iso.org/standard/68004.html.*

- **Standard number:** ISO 28500, *"Information and documentation — WARC file format."* First edition 2009; the current edition is **ISO 28500:2017**. ✅ *Source: ISO catalogue (iso.org).* The sibling acquisition guide's §6.5 flagged the 2017 edition as ⚠-knowledge; this pass verified it at the ISO catalogue.
- **Lineage.** WARC is an extension of the **ARC** format "that has traditionally been used to store web crawls as sequences of content blocks harvested from the World Wide Web" (ISO OBP). ARC is the format of the 2008–2013 Common Crawl collections and of the Internet Archive's early crawls. ✅ *Source: ISO Online Browsing Platform.*
- **Why records matter.** Each WARC record stores the response *plus* control metadata: the URI, the fetch timestamp, the IP address, the HTTP status, and a **digest** (payload checksum) that enables deduplication and integrity verification across the corpus — the mechanism the sibling guide's §6.5 uses for content-dedup and replay. ✅-knowledge, consistent with ISO's record-integrity goal.

### 4.2 WAT — Web Archive Transformation (metadata)

**WAT** files are derived, per-record metadata: the computed derivatives of each crawl record — response headers, link extractions, text metrics, and other machine-readable summaries — without the full payload. The overview page calls the corpus "raw web page data, metadata extracts, and text extracts," and the crawl archive ships WAT as its own collection (13.92 TiB for August 2026, vs 84.78 TiB of WARC). ✅ *Sources: commoncrawl.org/overview; data.commoncrawl.org.* A consumer doing link-graph or header analysis uses WAT; a consumer doing full-text work uses WET.

### 4.3 WET — plain-text extraction

**WET** files are the plain-text extractions of the crawl: the boilerplate-stripped text content of each page, ready for language modeling, indexing, or RAG ingestion without parsing HTML. At 5.84 TiB for the August 2026 crawl, WET is roughly **7% of the WARC volume** — the cheapest entry point for text-only pipelines. ✅ *Sources: commoncrawl.org/overview; data.commoncrawl.org.*

### 4.4 The Supporting Collections

- **Robots.txt snapshots** — every robots.txt the crawler read, archived per crawl (0.14 TiB for August 2026). This is a compliance gift: a downstream user can reconstruct exactly what CCBot was permitted to fetch at crawl time. ✅ *Source: data.commoncrawl.org.*
- **Non-200 responses** — the redirects, 404s, and errors encountered (2.63 TiB), useful for link-rot and web-health research. ✅ *Source: data.commoncrawl.org.*
- **Segments** — the corpus is partitioned into ~100 segments per crawl (shards for parallel processing); every file path includes its segment (`crawl-data/CC-MAIN-2026-34/segments/<segment-id>/warc/...`). ✅ *Source: data.commoncrawl.org layout.*

### 4.5 Why the Format Stack Matters Architecturally

The WARC/WAT/WET split is the corpus's original design gift to downstream engineering:

- **WARC** = the system of record — replayable, digest-verifiable, court-of-truth bytes (the sibling guide's "archive everything" rule, §6.5).
- **WAT** = the metadata layer — cheap analytics over structure and links without touching payloads.
- **WET** = the text layer — the input to LLM corpora, RAG pipelines, and search indexes.
- **robots.txt + non-200** = the compliance and web-health layers.

One format, three projections, all keyed by the same URL + timestamp + digest identifiers. This is the pattern a bank's own acquisition platform should copy (Section 11; sibling guide §6.5).

---

## 5. The Indexes, Access, and Hosting — Querying the Free Corpus

### 5.1 The Two Query Layers: CDXJ and the URL Index

Common Crawl provides **two indexes** over the corpus: ✅ *Sources: commoncrawl.org/cdxj-index; commoncrawl.org FAQ.*

1. **The CDXJ index** — the per-crawl lookup index, served by the URL index server at `index.commoncrawl.org` (one collection per crawl, e.g., `CC-MAIN-2026-34-index`). It answers "has this URL been crawled, when, and where in the WARC files?" The format is **CDXJ** — CDX with JSON metadata fields; "in other documentation, this index is often referred to as the CDX index for legacy reasons." ✅ *Source: commoncrawl.org/cdxj-index.* The server implements the PyWB CDX server API. ✅ *Source: index.commoncrawl.org.* The index was announced in April 2015. ✅ *Source: index.commoncrawl.org ("More about the URL index in the original announcement").*
2. **The URL Index (formerly the "Columnar Index")** — a corpus-wide, cross-crawl index of URL-level metadata in **Apache Parquet** columnar form (the `cc-index-table` project). It is built from the CDXJ files with Spark, and is queryable with SparkSQL, **Amazon Athena**, Apache Hive, and other big-data engines — designed for broad filtering and aggregation across crawls ("a better fit for bulk filtering and aggregation" than the CDX server). ✅ *Sources: github.com/commoncrawl/cc-index-table; index.commoncrawl.org; commoncrawl.org FAQ.*

Practical guidance from the FAQ itself: use the CDXJ server for targeted URL lookups, but "use our URL Index… via Amazon Athena or Apache Spark if your query involves broad or large-scale filtering." ✅ *Source: commoncrawl.org FAQ.*

### 5.2 Access Model — Free Data, Pay-Your-Own-Compute

- **The data is free.** "Access to the corpus hosted by Amazon is free. You may use Amazon's cloud platform to run analysis jobs directly against it or you can download it, whole or in part." ✅ *Source: commoncrawl.org/overview.* "All data and index files are free to download." ✅ *Source: index.commoncrawl.org.*
- **Hosting.** The corpus lives in **Amazon Web Services' Public Data Sets** program (bucket `s3://commoncrawl/`, region us-east-1), and — per the overview page — "on multiple academic cloud platforms across the world." ✅ *Sources: commoncrawl.org/overview; index.commoncrawl.org; Wikipedia.*
- **Who pays for what.** Downloading to your own infrastructure outside AWS incurs normal S3 data-transfer charges; running Athena/Spark *inside* AWS against the bucket avoids egress. The overview's "free" refers to the data itself. ⚠ The precise bucket settings (e.g., requester-pays flags) were not re-verified this pass — see Section 13, item 4. Wikipedia's formulation — "Access to the data is free on Amazon Web Services, but users may incur storage and compute costs" — is the accurate mental model. ✅ *Source: Wikipedia (Common Crawl).*
- **Hugging Face mirror (2026).** In April 2026 Common Crawl experimentally began distributing its data through a **Hugging Face storage bucket** in addition to S3. ✅ *Source: Wikipedia, citing commoncrawl.org blog "April 2026 Crawl Archive Now Available in a Hugging Face Storage Bucket."*
- **Official tooling.** `cc-downloader` (official, "robust and polite" bulk downloader) and `cdx-toolkit` (PyPI) are the reference clients; the FAQ explicitly asks consumers to be polite to the index server — it is "frequently abused and therefore heavily rate limited," HTTPS only, sleep between calls, one thread per IP, no proxy networks, and 503 means slow down. ✅ *Source: commoncrawl.org FAQ.* The sibling guide's §4 politeness discipline applies verbatim to consuming this API.

### 5.3 Query Patterns an Architect Should Know

- **URL lookup:** `https://index.commoncrawl.org/CC-MAIN-2026-34-index?url=example.com&output=json` — returns CDXJ records with WARC filename, offset, length, digest, MIME, status. ✅ *Source: commoncrawl.org/cdxj-index (pattern).*
- **Bulk filtering:** Athena over the `cc-index-table` Parquet (`ccindex` table name in the public examples) — e.g., "all pages from `*.gov.sg` in 2025 crawls" is a SQL query over the columnar index, not a CDX crawl. ✅ *Source: github.com/commoncrawl/cc-index-table; commoncrawl.org FAQ.*
- **WARC fetch by coordinates:** given (filename, offset, length) from the index, `Range`-GET the exact record from `https://data.commoncrawl.org/<path>` — the standard pattern used by `cc-downloader` and `cdx-toolkit`. ✅ *Source: commoncrawl GitHub examples (cc-examples); index.commoncrawl.org.*
- **Rate discipline:** the CDX server is a shared, rate-limited public service; bulk jobs must use the columnar index or the download manifests (`warc.paths.gz`), not hammer the API. ✅ *Source: commoncrawl.org FAQ.*

### 5.4 A Concrete Athena Pattern

The `cc-index-table` project documents the canonical usage: build (or reuse) the index table from the URL index files with Spark, register it in the AWS Glue catalog, and query from Athena. A representative query shape for the Cymbal Bank use case (Section 11) — all `.gov.sg` pages seen in the 2025 crawls:

```sql
SELECT url, fetch_time, status, mime, warc_filename, warc_offset, warc_length
FROM ccindex
WHERE crawl IN ('CC-MAIN-2025-05','CC-MAIN-2025-08','CC-MAIN-2025-13',
                'CC-MAIN-2025-18','CC-MAIN-2025-21','CC-MAIN-2025-26',
                'CC-MAIN-2025-30','CC-MAIN-2025-33','CC-MAIN-2025-38',
                'CC-MAIN-2025-43','CC-MAIN-2025-47','CC-MAIN-2025-51')
  AND url_host_registered_domain IN ('gov.sg')
  AND status = 200
```

The result set gives every WARC coordinate needed for a ranged GET of the payloads — the exact (crawl, filename, offset, length) provenance tuple Section 11.6 requires. ✅ *Pattern per github.com/commoncrawl/cc-index-table and the commoncrawl public Athena examples; the column names (`url_host_registered_domain`, `warc_filename`, `warc_offset`, `warc_length`, `fetch_time`, `status`, `mime`) follow the documented `ccindex` schema.* ⚠ The exact column set should be confirmed against the current `cc-index-table` schema at build time; the schema has evolved across versions.

### 5.5 The Tooling Ecosystem

The Common Crawl GitHub organization (`github.com/commoncrawl`) hosts the official stack: ✅ *Sources: commoncrawl.org FAQ; index.commoncrawl.org; github.com/commoncrawl.*

- **`cc-downloader`** — the official, "robust and polite" bulk download client;
- **`cdx-toolkit`** — the reference Python client for the CDXJ server (PyPI), explicitly endorsed as "an example of good practices and politeness";
- **`cc-index-table`** — the columnar index builder/query layer (Spark, Athena, Hive);
- **`cc-pyspark`** — PySpark examples for processing the tabular URL index at scale;
- **`cc-notebooks`** — Jupyter notebooks for exploration;
- **`cc-crawl-statistics`** and **`cc-webgraph-statistics`** — the per-crawl metrics and web-graph metrics projects;
- **`cc-citations`** — the curated BibTeX list of papers using the corpus.

Community tooling referenced by the FAQ includes `cc_net` (the pipeline RedPajama and others use to filter Common Crawl into training corpora) and the sibling guide's §7 tooling landscape. ✅ *Source: HF RedPajama card (cc_net); sibling guide §7.*

## 6. LLM Training Usage — the C4 Lineage and Beyond

### 6.1 Why Common Crawl Became the Default LLM Corpus

A pre-training corpus must be enormous, open, and cheap to obtain. Common Crawl is the only public dataset that is all three at once — which is why "the data had mostly been primarily used by researchers and some startups until the 2020s, when AI companies started training large language models using the data" (Wikipedia). ✅ *Source: Wikipedia (Common Crawl).* The result is a family tree: raw monthly crawls → cleaned/derived corpora (C4, mC4, The Pile's Pile-CC, RedPajama's CommonCrawl slice, Falcon RefinedWeb, OSCAR, FineWeb, DCLM) → models (T5, GPT-3, LLaMA-family reproductions, Falcon, and, per 2023 reporting, ChatGPT and Gemini). ⚠ The ChatGPT/Gemini training-data claims rest on secondary reporting (Wikipedia cites Dark Reading, 2023); the labs do not publish ingredient lists — flagged, not asserted as fact.

### 6.2 C4 — the Colossal Clean Crawled Corpus (Google, 2019)

C4 is the canonical Common Crawl derivative and the anchor of this section:

- **Origin.** C4 was introduced with Google's T5 paper, *"Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer"* (Raffel et al.), submitted to arXiv on **23 October 2019** (arXiv:1910.10683). ✅ *Source: arXiv; sibling guide §10.4 verified record.*
- **Definition.** TensorFlow Datasets calls it "A colossal, cleaned version of Common Crawl's web crawl corpus," generated with the cleaning pipeline distributed with the T5 codebase. ✅ *Source: tensorflow.org/datasets/catalog/c4.*
- **Size (verified this pass).** The TFDS catalog lists `c4/en` (default config) at **806.87 GiB** with **364,613,570 training examples**; the uncleaned `c4/en.noclean` config is **6.21 TiB** with 1,063,805,169 examples — a dramatic demonstration of how much cleaning removes (~87% of examples). ✅ *Source: tensorflow.org/datasets/catalog/c4.* The commonly cited "~750 GB / 156B tokens" figures come from the T5 paper itself; ⚠ the paper's tables were not re-read this pass (see Section 13, item 3). The sibling guide's §10.4 and §12.2 flag the ~800 GB figure similarly.
- **Cleaning recipe.** The T5 pipeline removes code and boilerplate, drops near-duplicate lines, filters placeholder text and pages whose text is mostly punctuation/numbers, and applies a bad-words filter. ⚠-knowledge per the T5 paper's §2.1 as summarized in the sibling guide; not re-derived here.
- **The controversies are data-quality facts.** In April 2023, reporting by The Register (and The Guardian) noted C4's filtering was imperfect — the corpus drew from sources including 4chan and contained racist content; the same reporting noted concerns over copyrighted text. ✅-via-secondary-source (Wikipedia citing The Register, 20 April 2023). A 2024 study (as reported by the New York Times, via Wikipedia) found **45% of C4 content was explicitly restricted by websites' terms of service** for uses like AI training by for-profit companies. ⚠ Single-study figure, secondary sourcing — flagged. These facts drive the Cymbal Bank compliance posture in Section 11.

### 6.3 mC4 and the C4 Variants

- **mC4** (multilingual C4): generated from **86 Common Crawl dumps**, covering **101 languages**, at **38.49 TiB** (TFDS `c4/multilingual`). ✅ *Source: tensorflow.org/datasets/catalog/c4.* The per-language example counts are themselves a bias dataset: English leads with ~3.9B examples, `und` (unidentified) ~3.65B, Russian ~1.01B, German ~546M, Spanish ~591M, while scores of languages have under a million — quantitative evidence for the English-heavy / long-tail-thin skew discussed in Section 8.4.
- **Other C4 derivatives** include `c4/realnewslike` (36.91 GiB, restricted to RealNews domains) and `c4/webtextlike` (17.93 GiB). ✅ *Source: TFDS catalog.*

### 6.4 The Open LLM Corpora Built on Common Crawl

| Corpus | Builder | Common Crawl relationship | Verified scale |
|---|---|---|---|
| **C4** | Google (2019) | Cleaned snapshot | 806.87 GiB / 364.6M docs (TFDS ✅) |
| **mC4** | Google (2020) | 86 dumps, 101 languages | 38.49 TiB (TFDS ✅) |
| **The Pile (Pile-CC)** | EleutherAI (2020) | 22-subset corpus incl. a Common Crawl subset | 825 GiB total (arXiv:2101.00027 ✅, sibling guide) |
| **RedPajama-Data-1T** | Together Computer (April 2023) | "Clean-room implementation of the LLaMA dataset"; CommonCrawl slice = **878B tokens** of 1.2T total; C4 slice = 175B | HF dataset card ✅ |
| **Falcon RefinedWeb** | TII (June 2023) | "Based on CommonCrawl," deduplicated + filtered, web data only | 968M documents / 1.68 TB (HF ✅; paper arXiv:2306.01116) |
| **FineWeb, DCLM** | HF/DataComp (2024) | Refined Common Crawl derivatives | ⚠ not re-verified this pass |
| **OSCAR** | OpenSuperlarge (2019–) | Filtered Common Crawl by language | ⚠ not re-verified this pass |
| **GPT-3's largest bucket** | OpenAI (2020) | *Filtered* Common Crawl | ⚠ per Brown et al. 2020, not re-verified this pass (sibling guide §10.4) |

Verified details worth keeping: **RedPajama** runs five Common Crawl dumps through the `cc_net` pipeline, deduplicates at paragraph level, and filters with a Wikipedia-reference-vs-random classifier; **RefinedWeb** (the Falcon dataset) demonstrated that "outperforming curated corpora with web data, and web data only" is possible once deduplication and filtering are done properly — the key citation for the argument that corpus *quality engineering* matters more than corpus *origin*. ✅ *Sources: HF dataset cards (togethercomputer/RedPajama-Data-1T; tiiuae/falcon-refinedweb).*

### 6.5 The C4-Lineage Lesson for Architects

The lineage teaches three things:

1. **Raw Common Crawl is not a training corpus.** Every serious derivative exists because the raw WET text is too noisy (Section 8.2). Plan for a cleaning stage, not a download stage.
2. **Cleaning has a massive yield curve.** C4's clean/unclean split (364M vs 1.06B examples) and the whole RefinedWeb paper's premise are the evidence.
3. **Legal surface grows with scale.** The 45%-ToS-restricted finding and the 2023 C4 content controversy mean downstream users — especially regulated firms — must own their filtering and provenance, not inherit Common Crawl's (Section 11).

### 6.6 From Corpus to Model — the Data-Centric AI Field

The C4 lineage turned "web data curation" into its own research discipline. The arc runs: T5's C4 (2019) showed a *cleaned* web corpus could beat curated corpora; GPT-3 (2020) showed a *filtered* web bucket at massive scale was enough for a foundation model (⚠ per Brown et al., sibling guide §10.4); RefinedWeb (2023) proved "web data, and web data only" could match curated corpora once dedup and filtering were done right; and the 2024-era refinements (FineWeb, DCLM — ⚠ not re-verified this pass) pushed the state of the art in *how to filter*, with ablations over quality classifiers, dedup granularity and PII removal. Alongside them, the open-source toolchain — `cc_net`, `datatrove`, and the RedPajama scripts — made the Common Crawl → corpus → model path reproducible by anyone. ✅ *Sources: HF dataset cards; arXiv:2306.01116; the sibling guide's §10.4.* The practical consequence for an enterprise: the pipeline stages of §11.4 are not bespoke science — they are a documented, field-standard playbook with open reference implementations, and a bank should adopt the *pattern* while owning the *policy* (allow-lists, PII, provenance).

---

## 7. Academic Research and Web Analytics

### 7.1 The Citation Record

Common Crawl is one of the most-cited research infrastructure projects on the web: "Cited in over 10,000 research papers" (homepage, 2026); Wikipedia records "more than 10,000 academic studies" as of 2024. ✅ *Sources: commoncrawl.org; Wikipedia.* The foundation maintains a curated BibTeX list (`commoncrawl/cc-citations` on GitHub) and features recent papers on its homepage — including "What the crawler keeps, and what it loses" (2026, on crawl persistence), "CommonLID" (2026, web language identification), and "Web Crawl Refusals: Insights From Common Crawl" (a study of crawlers facing inconsistent blocking). ✅ *Source: commoncrawl.org homepage.*

### 7.2 Web Data Commons — the Structured-Data Extraction Layer

The **Web Data Commons (WDC)** project is the single most important academic consumer of Common Crawl: it extracts *structured data* from the crawls and republishes it for research and industry. ✅ *Source: webdatacommons.org.*

- Started in **2012** by researchers from Freie Universität Berlin and the Karlsruhe Institute of Technology; now mainly maintained by the Data and Web Science group at the **University of Mannheim**, coordinated by **Christian Bizer**. ✅ *Source: webdatacommons.org.*
- Outputs include: RDFa/Microdata/Microformat/embedded-JSON-LD extractions per crawl (schema.org data), the **WDC Web Table Corpus** (233 million web tables from the July 2015 crawl), product-data corpora and entity-matching gold standards, an "IsA" hypernymy database, and the **hyperlink graph** — the basis of the 2015 *Journal of Web Science* paper "The Graph Structure in the Web — Analyzed on Different Aggregation Levels." ✅ *Source: webdatacommons.org.*
- Relevance: WDC is the bridge between "raw crawl" and "structured web data" — the layer a bank would use for entity/product/marketplace analytics without re-deriving extractors. It also funds itself partly through an AWS education grant and DFG support. ✅ *Source: webdatacommons.org.*

### 7.3 Hyperlink Graphs and Web-Graph Research

Two graph layers matter:

1. **The WDC hyperlink graphs** (host-level and domain-level), built from Common Crawl, used for link analysis, PageRank-style studies, and web-structure research. ✅ *Source: webdatacommons.org.*
2. **Common Crawl's own web-graph releases** — the foundation publishes host- and domain-level web graphs per crawl quarter; the June–August 2026 release covers **235.0 million nodes and 3.1 billion edges at host level, and 119.7 million nodes and 2.5 billion edges at domain level**. ✅ *Source: commoncrawl.org blog (June/July/August 2026 web graphs).* Statistics and plots for every crawl (pages, unique URLs, hosts, domains, TLDs, MIME types, charsets, languages, crawl overlaps) are published via the `cc-crawl-statistics` project. ✅ *Source: commoncrawl.github.io/cc-crawl-statistics.*

### 7.4 Web Analytics — Cross-Reference to the Sibling Guide

Common Crawl is the substrate for a family of analytics use cases that the sibling [Large-Scale Web Data Acquisition guide](large_scale_web_data_acquisition_guide.md) covers in depth in its Section 10 (search, price monitoring, news aggregation, LLM corpora): price and product analytics over WDC product corpora (e-shop offers), market-share and technology-adoption studies (the homepage-featured "Diffusion of Artificial Intelligence Across Firms: Evidence from Europe," built on firm websites in Common Crawl), news timelines via CC-NEWS, and link-based reputation research. ✅ *Sources: commoncrawl.org homepage featured papers; sibling guide §10.* This guide does not re-derive that material; the sibling guide's §7.7 also contains the acquisition-side profile of Common Crawl as a tool.

### 7.5 Featured Research on the Homepage (2026)

The foundation curates recent research on its homepage; the following were listed with full citation URLs in the August 2026 extraction: ✅ *Source: commoncrawl.org homepage.*

- **"What the crawler keeps, and what it loses"** — Michael Paris, Hande Celikkanat, Luca Foppiano (Common Crawl's own team): discovery curves, core persistence and shell dynamics in longitudinal web crawls (arXiv:2607.13636) — the persistence/coverage study cited in §3.4 and §8.4;
- **"The Diffusion of Artificial Intelligence Across Firms: Evidence from Europe"** — Julio Garbers, Terry Gregory (IZA discussion paper): tracking AI adoption from firm websites in Common Crawl — a live example of §7.4's web-analytics class;
- **"CommonLID: Re-evaluating State-of-the-Art Language Identification Performance on Web Data"** — Ortiz Suarez, Burchell, Arnett et al. (arXiv:2601.18026): a web-language-identification benchmark built on Common Crawl — relevant to §11.4's language filter;
- **"Web Crawl Refusals: Insights From Common Crawl"** — Ansar, Sperotto, Holz: a study of crawlers facing inconsistent, poorly signalled blocking — the academic mirror of §8.1's rising-blockage problem;
- **"Banned Books: Analysis of Censorship on Amazon.com"** — Citizen Lab (2024): a free-expression study using Common Crawl data.

The pattern is worth noting: the homepage doubles as a *public research log* — the organization's own team and outside academics both publish through it, which gives the corpus an unusual degree of self-documentation (and self-criticism, as the persistence and refusal papers show).

## 8. Quality and Limitations — What the Crawl Is Not

### 8.1 Coverage — a Sample of the Surface Web

- **Surface web only, sampled.** The corpus is "a sample of the web" (FAQ, §3.4): no login-walled content, no JavaScript-rendered content (CCBot "does not execute JavaScript and does not use cookies"), no app-internal data. ✅ *Source: commoncrawl.org FAQ.* Anything behind authentication, client-side rendering, or bot-challenges is structurally absent — a critical limitation for banking use cases where the interesting pages are behind portals.
- **Blocking is rising.** CCBot's share of blocked traffic grew through the 2020s: by 2025 reporting (The Atlantic, via Wikipedia), CCBot was "the most widely-blocked bot by the top 1000 websites." ⚠ Secondary-source figure — flagged. Academic work on this exact phenomenon is featured by the foundation itself ("Web Crawl Refusals: Insights From Common Crawl," 2025/2026). ✅ *Source: Wikipedia; commoncrawl.org homepage.* The practical effect: coverage skews toward sites that *don't* block bots, and the skew grows over time.
- **Per-site depth is thin.** A randomly sampled subset per site means the corpus is broad-but-shallow; deep archives of specific properties require the Internet Archive or a targeted crawl (Section 9.1).

### 8.2 Noise — Why C4-Style Cleaning Exists

Raw WET text is famously noisy. The 2023-era sample rows in the Falcon RefinedWeb dataset card are a perfect illustration: streaming-piracy spam ("Watch Survivor Redemption Island Season 22 Episode 11... Free Stream Megavideo"), scraped tag-soup pages, and comment-spam blog pages sitting next to legitimate content in the same dump. ✅ *Source: HF tiiuae/falcon-refinedweb dataset viewer (CC-MAIN-2013-20 records).* The noise classes an architect must filter:

- HTML boilerplate, navigation and template text (WET strips markup but not nav-chrome);
- spam, SEO farms, machine-generated and scraped-content sites;
- placeholder text, cookie banners, "consent" walls, and short/punctuation-dominated pages;
- duplicate and near-duplicate content across domains and crawls (mirrors, syndication, reposts).

This is precisely why the C4 pipeline, RedPajama's paragraph-level dedup + classifier, and RefinedWeb's dedup/filter stages exist (§6). The sibling guide's Section 5 (URL normalisation, hashing, SimHash) is the engineering deep-dive for the deduplication half.

### 8.3 Deduplication Practices

Three levels of dedup matter when building on Common Crawl:

1. **At the corpus level** — the WARC digests and the CDXJ index give you exact-duplicate detection for free (same content hash across URLs/crawls); the sibling guide's §5 machinery (URL canonicalisation, SimHash) extends this to near-duplicates.
2. **At the derivative level** — C4 (line-level dedup), RedPajama (paragraph-level), RefinedWeb (MinHash-style document dedup) all report large yield losses from dedup alone; treat dedup as a mandatory stage, not an optimization.
3. **Across crawls** — the `cc-crawl-statistics` project publishes per-crawl overlap analyses; month-over-month overlap is substantial, so multi-crawl corpora must deduplicate *across* dumps or you silently multiply identical content. ✅ *Source: commoncrawl.github.io/cc-crawl-statistics (crawl overlap plots).*

### 8.4 Biases — English-Heavy, Developed-World-Skewed, Temporally Gappy

- **Language.** The corpus is English-heavy. mC4's own per-language counts (§6.3) show English at ~3.9B examples vs. single-digit-millions for most of the world's languages, plus a ~3.65B "und" (unidentified) bucket. ✅ *Source: TFDS catalog.* Common Crawl publishes per-crawl language statistics via cc-crawl-statistics. ✅
- **Geography and wealth.** Web presence correlates with developed-world infrastructure, and bot-blocking (8.1) amplifies the skew; specific share figures for geographic skew were not re-verified this pass ⚠ — the direction is well documented, the magnitude is not pinned here (Section 13, item 8).
- **Temporal.** Each crawl is a monthly snapshot; a page's *history* within the corpus is only as good as its presence in successive crawls. The foundation's own 2026 paper "What the crawler keeps, and what it loses" studies exactly this: core pages persist across crawls, but a long tail of pages appears once and disappears ("discovery curves, core persistence, and shell dynamics"). ✅ *Source: arXiv (homepage-featured), 2026.* For time-series or "what did the web say in month X" questions, pick the right crawl(s) deliberately (§11.4).

### 8.5 The Controversies — Paywalls, Removals, and the 2025 Atlantic Investigation

A November 2025 investigation by Alex Reisner in *The Atlantic* alleged that Common Crawl **misled publishers**: it claimed to respect paywalls in its scraping, and claimed to honor publisher removal requests, while (per the investigation) paywalled content remained in the data and some removed sites were still present in the crawls consumed by AI companies; the investigation also reported the public search function on commoncrawl.org showed no entries for sites that had requested removal while the underlying data still contained them. ⚠ *Single-source investigation, reported via Wikipedia; Common Crawl's public response was not reviewed this pass — see Section 13, item 6.* Related reporting: Wired (June 2024) documented publishers targeting Common Crawl in the fight over AI training data. ⚠-via-Wikipedia.

**What this means downstream:** the foundation's *stated* posture is robots.txt-honoring and opt-out-able (Section 10), but the 2025 allegations put the *operational* enforcement of paywall and removal claims in dispute. A regulated firm cannot rely on Common Crawl's representations alone — it must run its own domain allow/deny lists, PII screening, and provenance register (Section 11).

### 8.6 Freshness and Overlap in Practice

Two operational facts shape every time-sensitive use of the corpus:

- **Publication lag.** Each crawl's WARC/WAT/WET sets and indexes are released together, announced on the blog, and listed at `data.commoncrawl.org/crawl-data/CC-MAIN-YYYY-WW/`; the index server's collection list is the authoritative "what is available now" view (August 2026's crawl `CC-MAIN-2026-34` was listed by the time of this pass). ✅ *Source: index.commoncrawl.org.* Typical freshness: content fetched in month M becomes queryable roughly at the month boundary — fine for monthly analytics, not for anything sub-weekly (that is CC-NEWS's job, §9.2).
- **Overlap is your enemy and your friend.** The cc-crawl-statistics overlap plots document substantial URL repetition between consecutive crawls ✅ — an enemy because naive multi-crawl ingestion multiplies identical documents (dedup across dumps is mandatory, §8.3), and a friend because it gives multi-snapshot time series: a page present in 24 consecutive crawls is a stable, monitorable entity, while a page appearing in exactly one crawl is likely ephemeral or newly discovered (the "core vs shell" distinction of the 2026 persistence paper, §3.4). For the bank's RAG corpus, the practical rule is: **ingest by crawl window, deduplicate across the window, and tag every document with its crawl id** — provenance before embeddings.

### 8.7 The Bottom Line on Quality

Raw Common Crawl is best understood as *the web's raw material, not the web's finished product*: it is sampled (not complete), shallow per site (not deep), noisy (not curated), English-heavy and developed-world-skewed (not representative), and monthly (not fresh). Every one of those limitations is addressable by a designed pipeline — §11.4's filter chain, the sibling guide's dedup machinery, per-crawl selection, and provenance tagging — which is exactly why the C4/RedPajama/RefinedWeb lineage and this guide's worked example exist.

---

## 9. The Alternatives and Ecosystem

### 9.1 Internet Archive — the Other 1996-Era Giant

The **Internet Archive** is the non-profit digital library founded in **1996 by Brewster Kahle**; its **Wayback Machine** launched as a public web-page search engine in **2001** (its first crawl dates to 1996), and by 2021 it had captured ~588 billion pages working with 800+ partners. ✅ *Sources: en.wikipedia.org/wiki/Internet_Archive; archive.org/details/wayback-machine-1996.* The relationship with Common Crawl:

| Dimension | Common Crawl | Internet Archive / Wayback Machine |
|---|---|---|
| Founded | 2007 (Gil Elbaz) ✅ | 1996 (Brewster Kahle) ✅ |
| Model | Bulk monthly crawls for analysis | On-demand archiving + replay ("time machine") |
| Primary output | WARC/WAT/WET corpora + indexes | Page replay service + collections |
| Access | Free bulk download / Athena | Free replay; bulk via APIs/collections ⚠ |
| Best for | LLM corpora, analytics, research at scale | Historical page reconstruction, citation, link rot |

The two were designed for different jobs: MIT Technology Review (2013) quotes the design intent that Common Crawl offers "more and fresher data that was more efficient to analyze" than the Wayback Machine's archives. ✅ *Source: Wikipedia, citing MIT Technology Review.* For a bank: Wayback is the audit-trail tool (what did a page say on date X), Common Crawl is the corpus tool (all the text, at scale, now).

### 9.2 CC-NEWS — the News-Specific Stream

**CC-NEWS** is Common Crawl's continuous crawl of news articles from news sites worldwide, published as WARC files on a **daily** basis in `crawl-data/CC-NEWS/` on the `commoncrawl` S3 bucket, with filenames keyed by year and month. ✅ *Source: commoncrawl.org blog "News Dataset Available"; data.commoncrawl.org.* ⚠ The exact launch year was not re-verified this pass (Section 13, item 5). For news-timeline and market-event analytics, CC-NEWS is the freshest Common Crawl product; the sibling guide's §10.3 news-aggregation case and the geolocated German-news study featured on the homepage both build on it. ✅ *Source: commoncrawl.org homepage; sibling guide §10.3.*

### 9.3 The Derivative-Corpus Ecosystem

The ecosystem around Common Crawl now includes: the C4 family (C4, mC4, realnewslike, webtextlike — §6.2/6.3), The Pile (Pile-CC subset), RedPajama (and RedPajama-V2, ⚠ not re-verified), Falcon RefinedWeb (and RefinedWeb-Edition-2), OSCAR (language-filtered Common Crawl), FineWeb and DCLM (2024-era "best-practice" refinements), plus hosted mirrors on Hugging Face. ✅/⚠ per the §6.4 table. The ecosystem lesson: **nobody consumes raw Common Crawl** — the value chain is crawl → filter → dedup → refine → train/RAG, and the refinements have become a research field of their own (the "data-centric AI" angle cross-referenced in the sibling guide's §10.4 and the repository's AI-governance guide).

### 9.4 The Commercial Alternatives

The big AI labs' proprietary crawls (Google's internal web index, OpenAI's GPTBot-based crawl, Anthropic's ClaudeBot-based crawl) are orders of magnitude larger but completely closed — no public corpus, no formats, no audit trail ⚠ (their crawler user-agents and robots.txt posture are publicly documented; their corpora are not). Commercial web-data providers (Bright Data, Zyte — the company behind Scrapy, Oxylabs, and similar) sell *targeted, structured* extraction and proxy infrastructure rather than open archives; they are the sibling guide's §7.1/§8 territory and are alternatives to *doing your own crawl*, not to Common Crawl. For corpus-scale open data, the honest comparison is Common Crawl vs Internet Archive vs the derivative corpora above.

### 9.5 Comparison Table

| Corpus / organization | Founding / era | Scale | Formats | Access / cost | Best for |
|---|---|---|---|---|---|
| **Common Crawl** | 2007 (Elbaz) ✅ | 300B+ pages all-time; 2–3B pages/crawl; ~85 TiB WARC/crawl ✅ | WARC, WAT, WET, CDXJ, Parquet URL index | Free (S3 open data + HF mirror); pay your own compute ✅ | LLM corpora, web-scale analytics, research |
| **Internet Archive** | 1996 (Kahle) ✅ | ~588B pages (2021) ⚠-dated | WARC, replay service | Free replay; bulk access ⚠ | Historical replay, link-rot, audit trails |
| **CC-NEWS** | 2016-era ⚠ | Daily news stream | WARC (daily) | Free (S3) ✅ | News timelines, event studies |
| **C4 / mC4** | 2019/2020 (Google) ✅ | 807 GiB / 38.5 TiB ✅ | TFRecords/JSON (HF) | Free (HF/TFDS) ✅ | English/multilingual pre-training |
| **RedPajama-1T** | 2023 (Together) ✅ | 1.2T tokens ✅ | JSONL (HF) | Free ✅ | LLaMA-style pre-training reproduction |
| **Falcon RefinedWeb** | 2023 (TII) ✅ | 968M docs / 1.68 TB ✅ | Parquet/JSONL (HF) | Free ✅ | Web-only pre-training |
| **OSCAR** | 2019– ⚠ | Multi-hundred-GB per language ⚠ | JSONL (HF) ⚠ | Free ⚠ | Language-specific corpora |
| **Proprietary lab crawls** | 2020s ⚠ | Not disclosed | Not disclosed | Closed | (Not usable externally) |
| **Bright Data / Zyte et al.** | 2010s ⚠ | Commercial | API/datasets | Paid | Targeted structured extraction |

### 9.6 Choosing Between Them — a Decision Guide

| If your need is… | Use… | Why |
|---|---|---|
| A bulk corpus for LLM/RAG/analytics, free and at web scale | **Common Crawl** (+ a derivative like C4/RefinedWeb if you want pre-cleaned text) | Only open, corpus-scale, index-queryable option (§3–§6) |
| "What did this specific page look like on 14 March 2019?" | **Internet Archive / Wayback Machine** | Replay-by-date is its core competency (§9.1) |
| Fresh news text, daily | **CC-NEWS** | Daily WARC stream (§9.2) |
| Cleaned, deduplicated pre-training text without building a pipeline | **C4 / mC4 / RedPajama / RefinedWeb** | The cleaning already happened (§6.4) |
| Structured web data (schema.org, tables, product offers) | **Web Data Commons** | Extraction layer over Common Crawl (§7.2) |
| Targeted, live, structured extraction from specific sites | **Commercial providers (Bright Data, Zyte) or your own crawler** | Common Crawl is a sample, not a live targeted tool (§9.4; sibling guide §7–§8) |
| Regulated, authoritative documents | **Official channels, not any crawl** | MAS/SGX/BIS/IMF portals are the system of record (§11.1) |

The decision rule in one line: **Common Crawl for breadth and history, official sources for authority, Wayback for reconstruction, CC-NEWS for freshness, derivatives for convenience, and commercial or self-built crawls for precision.**

---

## 10. Legal and Ethics — Condensed

This section is deliberately **condensed**: the sibling [Large-Scale Web Data Acquisition guide](large_scale_web_data_acquisition_guide.md), Section 9, is the repository's deep dive on the legal boundaries (CFAA and *hiQ v. LinkedIn*, GDPR, Singapore's PDPA, copyright and contracts, robots.txt as RFC 9309), and its Section 8 covers the anti-bot arms race. Nothing here re-derives that analysis; this section states only Common Crawl's own posture and the obligations it creates for downstream users.

### 10.1 CCBot's Documented Posture (Verified from the FAQ)

| Aspect | CCBot behavior (FAQ, verified ✅) |
|---|---|
| Engine | Nutch-based crawler using Apache Hadoop; Map-Reduce candidate extraction |
| User agent | `CCBot/2.0 (https://commoncrawl.org/faq/)` (older: `CCBot/1.0`) |
| robots.txt | Checks robots.txt first; fetches only if allowed; honors `Crawl-delay`; follows RFC 9309 redirect rules for robots.txt fetches |
| Blocking | `User-agent: CCBot / Disallow: /` stops crawling; the crawler periodically re-checks robots.txt |
| Politeness | Adaptive back-off on HTTP 429/5xx; seconds between requests to the same host by default; honors `nofollow` for link discovery |
| Fidelity | No JavaScript execution, no cookies; conditional GET; gzip/Brotli/ZStandard; HTTP/1.1 + HTTP/2 over TLS; IPv4/IPv6 |
| Verification | Dedicated IP ranges with reverse DNS (`*.crawl.commoncrawl.org`) so webmasters can verify traffic |
| Sitemaps | Supports the Sitemap Protocol, including sitemaps announced in robots.txt |
| Opt-out | robots.txt exclusion plus a foundation **opt-out registry** (blog: "Common Crawl Foundation Opt-Out Registry") |

*Sources: commoncrawl.org FAQ; commoncrawl.org/ccbot.* The robots.txt discipline matches the RFC 9309 framework the sibling guide's §9.1 documents — Common Crawl was, for years, the exemplary polite crawler, which is also why CCBot's later blocking rate (8.1) is notable.

### 10.2 The Archive's Stance, and the Gaps

- **Open-data stance.** "As strong believers in Open Data, we apply as few restrictions as possible to the dataset"; the Terms of Use add restrictions "primarily in an effort to prevent abusive or illegal usage." ✅ *Source: commoncrawl.org FAQ (terms link).* The Terms of Use are the license surface every downstream consumer (including Cymbal Bank) must accept; derivative corpora (C4, RedPajama, RefinedWeb) each re-state CC ToU compliance in their cards. ✅ *Source: HF dataset cards.*
- **The disputed gap.** The November 2025 Atlantic investigation (§8.5) alleges paywall and removal claims did not match practice. ⚠ The foundation's own FAQ posture (robots.txt honored, opt-out registry) is verified; the *operational enforcement* of paywalls/removals is disputed. A bank's compliance overlay cannot rely on it (Section 11.6).
- **Copyright and personal data.** Common Crawl is a *copy* of the web: it contains copyrighted text and personal data. The C4 2023 controversy (racist/copyrighted content) and the 2024 45%-ToS-restricted finding (§6.2) are the concrete evidence that downstream users must filter and document their own use. The sibling guide's §9.5–9.7 (personal-data screen under GDPR/PDPA, ToS review, audit trail) applies directly to consuming this corpus; Cymbal Bank's PDPA obligations (Singapore) are unchanged by the corpus being "open."

### 10.3 The Obligation Matrix for Downstream Users

| Obligation | Why | Cross-ref |
|---|---|---|
| Accept CC Terms of Use | It is the corpus license | commoncrawl.org/terms-of-use |
| Maintain your own allow/deny lists | The corpus is not curated for your jurisdiction or domain | §11.4 |
| PII screening before any use | The corpus contains personal data | Sibling guide §9.5 (PDPA/GDPR) |
| Provenance register per document | Crawl id, WARC coordinates, fetch timestamp, robots snapshot | §11.6 |
| Respect source-site ToS in your *use* | 45%-ToS-restricted finding (§6.2) | Sibling guide §9.6 |
| Re-verify opt-outs for domains you care about | 2025 Atlantic allegations (§8.5) | §11.4, §13 item 6 |
## 11. Cymbal Bank Worked Example — an LLM/RAG Corpus Strategy

### 11.1 The Scenario and the Boundary

**Cymbal Bank** — the repository's Singapore-headquartered bank persona — is building an **LLM/RAG knowledge base** for its research, compliance and credit teams: a retrieval corpus that answers questions about monetary policy, financial regulation, market structure and the bank's own public footprint, grounded in citations. The design decision under examination: **how much of that corpus comes from Common Crawl, and how?**

The boundary is deliberately strict, following the sibling acquisition guide's Section 11 conventions:

- Common Crawl supplies **background and regulatory-text enrichment only** — the open-web layer (regulator websites, central-bank publications, exchange pages, reputable financial media, industry bodies).
- **Regulated, authoritative documents** (MAS notices, SGX rules, BIS/IMF publications) are sourced *directly from the official channels* (sibling guide §11.2) and are the *system of record*; Common Crawl is used for corroboration, historical context, and coverage of what official portals no longer publish.
- **Nothing from Common Crawl is redistributed** outside the bank, and no Common Crawl text enters customer-facing generation without citation and human review. Internal research and RAG use only.

### 11.2 Why Common Crawl at All, for a Bank?

- **Historical depth no API gives you:** the corpus reaches back to 2008–2013 ARC-era crawls — e.g., a 2013 MAS consultation paper or a defunct SGX page that the official portal has since removed is often recoverable from the crawls. ✅ *Source: index.commoncrawl.org collection list (2008–2013 ARC collections).*
- **Breadth at zero marginal data cost:** hundreds of thousands of Singapore- and Asia-relevant domains (`.sg`, `.com.sg`, central-bank and regulator sites, regional media) are already in the monthly crawls; the bank's own acquisition platform (sibling guide §11) cannot afford to crawl the same universe.
- **The corpus is free and reproducible:** any downstream artifact can be rebuilt from WARC coordinates — the provenance story regulators like (Section 11.6).

### 11.3 Selection — Which Crawls, Which Index Filtering

| Decision | Design | Rationale / source |
|---|---|---|
| Crawl window | **Trailing 24 months of monthly crawls** (e.g., `CC-MAIN-2024-33` … `CC-MAIN-2026-34`) + **selected historical crawls** (2013, 2016, 2020–2022) for the archival layer | Freshness vs coverage; monthly cadence ✅ (Section 3.2) |
| Selection mechanism | **URL Index (columnar) via Athena** for bulk domain filtering; **CDXJ server** only for targeted URL lookups | FAQ guidance: bulk filtering belongs on the columnar index ✅ (Section 5.1) |
| Domain allow-list (primary) | `mas.gov.sg`, `sgx.com`, `sgxgroup.com`, `bis.org`, `imf.org`, `worldbank.org`, `abs.org.sg`, `sgs.gov.sg`, `data.gov.sg`, `mof.gov.sg`, `parliament.gov.sg`, plus major financial media (e.g., Bloomberg terminal-adjacent public pages ⚠ licence-dependent — flag for legal review) | Regulatory and market-data relevance; sibling guide §11.2 source portfolio |
| Domain deny-list | Pirated-content, gambling, adult, malware, and known spam/SEO domains; **any domain that has opted out of Common Crawl** | 2025 Atlantic allegations make opt-out verification mandatory (§8.5, §10.3) |
| Record-level filter | WARC `content-type` in HTML/text; HTTP 200 only; language filter (English primary; Chinese/Malay for Singapore context ⚠ — verify per-crawl language stats via cc-crawl-statistics) | WET/robots/non-200 collections ✅ (Section 4.4) |

### 11.4 Filtering — the Cleaning Pipeline (Mandatory, Not Optional)

The pipeline is the C4 lesson applied (Section 6.5), sized to the bank's needs:

1. **Extraction:** WET text (or WARC→WET for pages missing WET), keyed by (crawl, WARC filename, offset, length, digest).
2. **Quality filter:** boilerplate/nav removal, sentence-perplexity or classifier-based quality scoring (the RedPajama Wikipedia-reference classifier pattern, §6.4), short-document and placeholder-text drop.
3. **Deduplication:** exact via WARC digests, then SimHash near-duplicate collapse across crawls and domains (sibling guide §5.3); cross-crawl overlap is material (§8.3).
4. **PII redaction:** automated screening (names+identifiers, NRIC-like patterns, emails, phone numbers) with quarantine-and-review routing to the data-protection officer — the sibling guide's §9.5 personal-data screen, applied to a corpus known to contain personal data (§10.2).
5. **Licensing and policy filter:** drop documents from domains whose terms-of-service prohibit the intended use (the 45%-ToS-restricted finding, §6.2); legal sign-off on the allow-list itself.
6. **Language and locality:** language identification (CommonLID-class tools ⚠-mention) and Singapore/ASEAN relevance scoring; keep only what the use case needs.

### 11.5 How It Feeds the RAG Knowledge Base

- **Chunking and embedding:** filtered documents → section-aware chunking → embeddings → the repository's RAG vector-store guides ([RAG Optimization Techniques](ai_llm/rag/rag_optimization_techniques_guide.md), [Advanced RAG Techniques](ai_llm/rag/advanced_rag_techniques_guide.md), [Vector Databases](ai_llm/rag/vector_databases_guide.md) — cited, not re-derived).
- **Refresh cadence:** monthly, aligned to each crawl release (announcement → index availability → pipeline run); CC-NEWS (daily) feeds a separate *news* index for market-event awareness with a 24-hour freshness SLA ⚠ (design target, not measured).
- **Groundedness:** every retrieved chunk carries its provenance block (Section 11.6) so the RAG layer can cite `crawl CC-MAIN-2026-34, WARC 134..., offset ..., fetched 2026-08-XX, robots.txt honored` — the retrieval equivalent of a citation.
- **Evaluation:** retrieval quality is measured with the repository's RAG evaluation tooling ([RAG Evaluation Methodology](ai_llm/rag/rag_evaluation_methodology_guide.md), [Ragas](ai_llm/rag/ragas_guide.md)) against a bank-built golden set of regulatory Q&A.

### 11.6 The Compliance Overlay

- **Provenance register:** per document — crawl id, WARC filename/offset/length, digest, fetch timestamp, source URL, robots.txt snapshot status, allow-list version, filter version. The register is the audit artifact MAS-aligned governance expects (cross-ref the sibling [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) guide).
- **Licensing posture:** Common Crawl Terms of Use accepted and recorded; source-site ToS reviewed per allow-listed domain class; no redistribution; internal use only.
- **Retention:** derived corpus retained with the register; raw WARC not re-hosted — the bank stores pointers + digests, not copies, of the open corpus (space and licensing discipline).
- **AI governance:** the corpus feeds only *approved* use cases under the bank's AI-governance framework — cross-ref [AI Governance, Bias and Red-Teaming](../technology/ai_llm/ai_governance_bias_redteaming_guide.md); the corpus's known biases (Section 8.4) are documented in the model card the RAG service ships.
- **The disputed-enforcement risk:** because the 2025 Atlantic findings (§8.5) put paywall/removal enforcement in dispute, the bank re-verifies opt-out status per allow-listed domain at each refresh and keeps denial records. This is a *bank-owned* control, not a Common Crawl one.

### 11.7 What This Design Does NOT Do

It does not train or fine-tune foundation models on Common Crawl (that would raise the ToS-restriction and copyright surface to an unacceptable level for a regulated firm under current legal uncertainty — the sibling guide's §9.6 copyright analysis); it does not use Common Crawl for any customer personal data; it does not redistribute; and it does not treat the corpus as an authoritative source for any fact it can source officially. The open web's memory is a *background* for the bank's knowledge base, never its spine.

### 11.8 The Pipeline Shape

The monthly refresh maps onto the sibling guide's Section 6 architecture, with Common Crawl standing in for the bank's own crawler:

```text
crawl announcement (blog) ──► index availability (index.commoncrawl.org)
        │
        ▼
[1] SELECT    URL Index (Athena) — domain allow/deny lists, HTTP 200,
              content-type filter           ──► ccindex query (§5.4)
        │
        ▼
[2] FETCH     ranged GETs of WARC/WET records from data.commoncrawl.org
              via cc-downloader, keyed by (crawl, warc, offset, length)
        │
        ▼
[3] CLEAN     boilerplate strip ─► quality/classifier filter ─►
              language ID ─► PII screen (quarantine ─► DPO) ─►
              ToS/policy filter (allow-list legal sign-off)
        │
        ▼
[4] DEDUP     exact (WARC digests) ─► SimHash near-dup collapse
              across crawls in window (sibling guide §5)
        │
        ▼
[5] REGISTER  provenance record per doc: crawl id, warc coords, digest,
              fetch ts, robots snapshot, allow-list/filter versions
        │
        ▼
[6] LOAD      chunk ─► embed ─► vector store (RAG layer, §11.5)
              + monthly snapshot to the lake (sibling guide §11.5 pattern)
```

Each stage is idempotent and replayable: a failed run re-reads from the WARC coordinates instead of re-fetching the web (politeness and reproducibility in one rule — the sibling guide's §6.4/§11.5 discipline). The whole pipeline runs as a monthly Airflow DAG (the sibling [Apache Airflow Guide](apache_airflow_guide.md) is the reference orchestrator), with the CC-NEWS daily stream feeding a separate, lighter news DAG.

### 11.9 The Operating Model

- **Runbooks per corpus stage** — including the blocked/removed-domain register: any allow-listed domain that appears in Common Crawl's opt-out registry or changes its robots.txt posture is flagged at refresh time, not silently re-ingested (§10.3).
- **Source-health scorecards** — per crawl window: pages retrieved vs indexed, filter yield per stage, PII-quarantine counts, dedup collapse ratios, reviewed monthly.
- **Quarterly compliance review** — with legal: Common Crawl Terms of Use changes, allow-list sign-offs, the Atlantic-era opt-out re-verification results, and any new regulator guidance on AI training data.
- **Annual re-verification** — of every source class and of the corpus itself: the web changes, the corpus's posture changes, and the compliance overlay must be re-derived from it (the sibling guide's §11.7 convention, applied to an external corpus instead of an internal crawler).

---

## 12. Claims Audit — Verified, Flagged, Unverified

Following repository convention, every verifiable claim is marked in the text; this section consolidates them. ✅ = verified this pass against a primary or cited source; ⚠ = flagged, dated, or well-documented knowledge not re-verified live; ❌ = checked and found incorrect, with the correction.

### 12.1 The Verified Claims (✅)

| Claim | Source |
|---|---|
| Common Crawl is a 501(c)(3) non-profit founded in 2007 by Gil Elbaz | commoncrawl.org; commoncrawl.org/about; Wikipedia; ProPublica EIN 26-1635908 |
| Open repository collected since 2008; crawls publicly published from ~2011; earliest collections `CC-MAIN-2008-2009`, `CC-MAIN-2009-2010` (ARC) | commoncrawl.org/about; Wikipedia; index.commoncrawl.org |
| Homepage scale: "over 300 billion pages spanning 19 years"; "3–5 billion new pages added each month"; cited in 10,000+ papers | commoncrawl.org homepage, retrieved August 2026 |
| Crawls published ~monthly, named `CC-MAIN-YYYY-WW` (ISO week); modern cadence visible in the collection list | index.commoncrawl.org; data.commoncrawl.org |
| August 2026 crawl `CC-MAIN-2026-34`: 2.14 billion pages; WARC 84.78 TiB (100,000 files); WAT 13.92 TiB; WET 5.84 TiB; robots.txt 0.14 TiB; non-200 2.63 TiB | data.commoncrawl.org/crawl-data/CC-MAIN-2026-34/index.html |
| WARC is the Web ARChive format; ISO 28500:2017 "Information and documentation — WARC file format" (first edition 2009); WARC extends ARC | iso.org/standard/68004.html; ISO Online Browsing Platform |
| Two indexes: CDXJ (per-crawl, "CDX for legacy reasons") and the URL Index (formerly "Columnar Index"), Parquet via `cc-index-table`, queryable with Athena/Spark/Hive | commoncrawl.org/cdxj-index; commoncrawl.org FAQ; github.com/commoncrawl/cc-index-table |
| Access: data free, hosted on AWS Public Data Sets (s3://commoncrawl/) + academic clouds; all data and index files free to download | commoncrawl.org/overview; index.commoncrawl.org |
| April 2026: distribution via a Hugging Face storage bucket added | Wikipedia, citing commoncrawl.org blog |
| CCBot: Nutch-based, Hadoop/Map-Reduce; UA `CCBot/2.0 (https://commoncrawl.org/faq/)`; robots.txt-first; honors Crawl-delay and nofollow; adaptive back-off on 429/5xx; no JS/cookies; dedicated IPs with reverse DNS; sitemaps supported; opt-out registry exists | commoncrawl.org FAQ; commoncrawl.org/ccbot |
| C4: "colossal, cleaned version of Common Crawl," introduced with the T5 paper (Raffel et al., arXiv:1910.10683, 23 October 2019); `c4/en` = 806.87 GiB / 364,613,570 train examples; `en.noclean` = 6.21 TiB / 1.06B examples; mC4 = 101 languages, 86 dumps, 38.49 TiB | tensorflow.org/datasets/catalog/c4; arxiv.org/abs/1910.10683 (date via sibling guide) |
| RedPajama-Data-1T: clean-room LLaMA-dataset reproduction, 1.2T tokens, CommonCrawl slice 878B tokens, C4 slice 175B, April 2023 | HF togethercomputer/RedPajama-Data-1T card |
| Falcon RefinedWeb: based on Common Crawl, 968M documents / 1.68 TB, paper arXiv:2306.01116 (June 2023) | HF tiiuae/falcon-refinedweb card |
| Internet Archive founded 1996 by Brewster Kahle; Wayback Machine launched publicly 2001; ~588B pages by 2021 | Wikipedia (Internet Archive); archive.org/details/wayback-machine-1996 |
| CC-NEWS: news-article WARC stream, released daily, at crawl-data/CC-NEWS/ | commoncrawl.org blog "News Dataset Available" |
| Web Data Commons: started 2012 (FU Berlin/KIT), now University of Mannheim (Bizer); extracts structured data (RDFa/Microdata/Microformat/JSON-LD), web tables, product corpora, hyperlink graphs from Common Crawl | webdatacommons.org |
| Common Crawl web graphs (June–Aug 2026): 235.0M nodes / 3.1B edges host-level; 119.7M / 2.5B domain-level; cc-crawl-statistics project | commoncrawl.org blog; commoncrawl.github.io/cc-crawl-statistics |
| Nutch adopted by Common Crawl February 2014 | Sibling guide §7.6 verified record |
| The corpus is "a sample of the web," not a full archive | commoncrawl.org FAQ |

### 12.2 The Flagged Claims (⚠)

| Claim | Flag |
|---|---|
| Gil Elbaz died in 2023 | No obituary or primary/secondary confirmation found this pass; Wikipedia and commoncrawl.org team page give no death date — Section 13, item 1 |
| Exact current funding amounts and full sponsor roster | 990s (ProPublica EIN 26-1635908) not opened this pass — Section 13, item 2 |
| C4 "~750 GB / 156B tokens" | T5-paper tables not re-read this pass; TFDS gives 806.87 GiB ✅ — Section 13, item 3 |
| About-page "10+ petabytes; crawls of 2B+ pages" wording | Dated relative to the 2026 homepage headline figures (§3.1) |
| ~3.1B pages captured (April 2023); 1.8B pages (2015); AWS costs "$2–4000/month" (2015) | Secondary/dated figures, via Wikipedia |
| November 2025 Atlantic investigation: paywall and removal claims disputed; CCBot most-blocked bot by top-1000 sites (2025) | Single investigation, secondary sourcing — Section 13, item 6 |
| 45% of C4 content restricted by source-site ToS for for-profit AI training (2024 study) | Single study, via NYT/Wikipedia — Section 13, item 7 |
| GPT-3's largest training bucket was a filtered Common Crawl | Brown et al. 2020, not re-verified this pass (sibling guide §10.4) |
| ChatGPT/Gemini trained largely on Common Crawl (2023 reporting) | Secondary reporting only |
| OSCAR, FineWeb, DCLM current sizes and access details | Not re-verified this pass |
| CC-NEWS launch year | Not re-verified — Section 13, item 5 |
| Requester-pays / exact S3 bucket settings | Not re-verified — Section 13, item 4 |
| Internet Archive current page count | 588B is the 2021 figure — dated |
| Geographic/wealth skew magnitude | Direction documented; magnitude not pinned — Section 13, item 8 |
| Cymbal Bank freshness SLA (24h news index) | Design target, not measured |

### 12.3 The Rejected Claims (❌)

| Claim | Verdict | Basis |
|---|---|---|
| "Common Crawl began publishing crawls in 2008" | ❌ — collecting since 2008, but public crawl publication began ~2011; the earliest *public* collections are the 2008–2009 and 2009–2010 ARC files, with the monthly CC-MAIN series starting in earnest 2013–2015 | Wikipedia (Common Crawl history); index.commoncrawl.org collection list |
| "Each monthly crawl contains 3–5 billion pages" | ❌ — the homepage's "3–5 billion new pages per month" is corpus-wide growth; individual crawls contain ~2–3 billion pages (August 2026: 2.14B) | commoncrawl.org homepage vs data.commoncrawl.org crawl listing |
---

## 13. What Could Not Be Verified

This section collects everything this pass could not confirm against a primary or reliable secondary source, so the reader can separate verified fact from honest uncertainty:

1. **Gil Elbaz's death.** The task brief stated he died in 2023; this pass found **no obituary, no primary confirmation, and no secondary source** stating a death date. Wikipedia's article (retrieved August 2026) is written in the present tense ("He is the founder and Chairman…"), and commoncrawl.org still lists him on the team page. The claim is therefore **not asserted as fact** in this guide; it is flagged ⚠ in Section 2.1 and listed here for correction.
2. **Exact current funding amounts and the full sponsor roster.** Wikipedia (via The Atlantic) documents the Elbaz Family Foundation Trust era and the 2023 Anthropic/OpenAI $250,000 donations; IRS Form 990s exist publicly (ProPublica Nonprofit Explorer, EIN 26-1635908) but were not opened this pass, so current revenue and additional sponsors are unverified.
3. **C4's exact byte size and token count in the T5 paper.** TFDS's 806.87 GiB / 364.6M-example figures are verified (Section 6.2); the paper's own "~750 GB" and "156B tokens" numbers were not re-read from arXiv this pass.
4. **Exact S3 bucket configuration** (requester-pays flags, region details) for `s3://commoncrawl/`. The free-access and AWS Public Data Sets facts are verified; the bucket's precise billing settings are not.
5. **CC-NEWS launch year.** The daily news WARC stream and its location are verified; the specific release year was not re-verified this pass (the §9.5 table marks it "2016-era ⚠").
6. **The November 2025 Atlantic investigation's specific allegations** (paywall bypass, unhonored removals, misleading search results). Reported via Wikipedia only; Common Crawl's public response was not reviewed this pass, and no second source was checked.
7. **The 45%-ToS-restricted finding for C4** (2024). Reported via the New York Times/Wikipedia; the underlying study was not read this pass.
8. **Magnitude of geographic/wealth skew** in the corpus. Direction is well documented (Section 8.4); specific share figures were not re-verified.
9. **OpenAI/Anthropic/Google proprietary corpus composition and scale.** These are not publicly documented in verifiable detail; the §9.4 characterization is qualitative.
10. **Current total petabytes of the corpus.** No aggregate petabyte figure newer than the about page's "10+ petabytes" wording was found this pass; the 2026 homepage headline is expressed in pages (300B+), not bytes.
11. **Internet Archive's current page count.** 588B is the 2021 figure; a 2026 number was not verified.
12. **The exact 2012-corpus size** (210 TB) is cited from the cc-crawl-statistics page's reference to the 2012 corpus analysis, not re-measured.
13. **The featured 2026 homepage papers** ("What the crawler keeps…" arXiv:2607.13636; CommonLID arXiv:2601.18026) were verified only to the level of the commoncrawl.org homepage listing (full arXiv URLs shown there, cited in §7.5); the papers' contents were not read this pass.

---

## 14. Glossary

| Term | Definition |
|---|---|
| Common Crawl | The 501(c)(3) non-profit foundation (founded 2007 by Gil Elbaz) that publishes free, open web-crawl archives; also the name of the corpus itself |
| CC-MAIN-YYYY-WW | The naming convention for monthly crawls: year + ISO week number of the crawl's start (e.g., `CC-MAIN-2026-34`) |
| ARC | The pre-WARC web-archive format (Internet Archive lineage) used by Common Crawl's 2008–2013 collections |
| WARC | Web ARChive — the ISO 28500:2017 archival format of concatenated records with headers, payloads and digests; the corpus's system of record |
| WAT | Web Archive Transformation — per-record metadata/derivatives (headers, links, metrics) |
| WET | Web Archive Transformation — plain-text extractions of page content |
| CDX / CDXJ | Index formats for URL lookups; CDXJ adds JSON metadata fields; "CDX" is used for legacy reasons |
| URL Index | The corpus-wide columnar index (Parquet, formerly "Columnar Index"), built by the `cc-index-table` project, queryable via Athena/Spark/Hive |
| cc-index-table | The Common Crawl project that builds and serves the columnar URL index |
| CCBot | Common Crawl's Nutch/Hadoop crawler (UA `CCBot/2.0`), robots.txt-honoring, with verifiable IP ranges |
| robots.txt | The Robots Exclusion Protocol (RFC 9309, 2022) — the file CCBot checks before fetching |
| Crawl-delay | robots.txt directive for minimum seconds between requests; CCBot honors it |
| Opt-out registry | Common Crawl's removal list for publishers who want their domains excluded |
| CC-NEWS | Common Crawl's daily news-article WARC stream under `crawl-data/CC-NEWS/` |
| C4 | Colossal Clean Crawled Corpus — Google's cleaned Common Crawl derivative for T5 (2019) |
| mC4 | Multilingual C4 — 101 languages from 86 dumps |
| RedPajama | Together Computer's open LLaMA-dataset reproduction (1.2T tokens; Common Crawl slice 878B) |
| Falcon RefinedWeb | TII's deduplicated, filtered Common Crawl corpus for the Falcon LLM |
| The Pile | EleutherAI's 825 GiB, 22-subset corpus including the Common Crawl-based Pile-CC |
| OSCAR | Language-filtered Common Crawl derivative corpus |
| WDC | Web Data Commons — University of Mannheim project extracting structured data (schema.org, tables, hyperlink graphs) from Common Crawl |
| Wayback Machine | The Internet Archive's public web-page replay service (launched 2001) |
| Internet Archive | The non-profit digital library founded 1996 by Brewster Kahle |
| 501(c)(3) | The US tax-code section for charitable non-profits; Common Crawl's status |
| AWS Public Data Sets | Amazon's program hosting large open datasets (Common Crawl's bucket) at no data cost |
| RFC 9309 | The IETF standard for the Robots Exclusion Protocol (September 2022) |
| SimHash | Charikar's locality-sensitive hashing for near-duplicate detection (sibling guide §5.3) |
| RAG | Retrieval-Augmented Generation — the architecture the Cymbal Bank knowledge base uses |
| MAS | Monetary Authority of Singapore — Singapore's central bank and financial regulator |
| PDPA | Singapore's Personal Data Protection Act 2012 — the data-protection regime governing the bank's corpus use |
| Cymbal Bank | The repository's Singapore-headquartered bank persona — the only bank persona in this guide |

---

## 15. Cross-References and Further Reading

**Repository guides (technology — sibling plain filenames):**
- [Large-Scale Web Data Acquisition — the Harvest at Scale](large_scale_web_data_acquisition_guide.md) — the sibling platform guide: crawling/scraping/fetching engineering, the URL frontier, politeness (its §4 applies to consuming `index.commoncrawl.org`), deduplication (its §5 is the deep-dive behind §8.3), pipeline and storage design (its §6.5 WARC-and-Parquet pattern), tooling (its §7.7 is the acquisition-side profile of Common Crawl), the legal deep-dive (its §9: CFAA, *hiQ v. LinkedIn*, GDPR, PDPA, copyright), large-scale use cases (its §10), and the Cymbal Bank market-data worked example (its §11)
- [AI Governance, Bias and Red-Teaming](ai_llm/ai_governance_bias_redteaming_guide.md) — the AI-governance angle on training-data acquisition and corpus bias (§11.6)
- [LLM Optimization: The Complete Guide](ai_llm/llm_optimization_complete_guide.md) — the training/inference side of LLM pipelines (§6)
- [RAG Optimization Techniques](ai_llm/rag/rag_optimization_techniques_guide.md), [Advanced RAG Techniques](ai_llm/rag/advanced_rag_techniques_guide.md), [Vector Databases](ai_llm/rag/vector_databases_guide.md) — the retrieval layer of the Section 11 knowledge base
- [RAG Evaluation Methodology](ai_llm/rag/rag_evaluation_methodology_guide.md), [Ragas](ai_llm/rag/ragas_guide.md) — retrieval-quality evaluation of the bank's RAG service (§11.5)

**Repository guides (banking — prefix `../banking/`):**
- [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) — the Singapore regulatory overlay and the Cymbal Bank persona conventions (§11.6)

**Primary sources used this pass:**
- commoncrawl.org — homepage (300B+ pages, 3–5B/month, 10,000+ citations), /about, /overview ("Access to the corpus hosted by Amazon is free"), /faq (CCBot posture, index guidance, sampling), /ccbot, /cdxj-index, /terms-of-use
- data.commoncrawl.org/crawl-data/CC-MAIN-2026-34/index.html — the August 2026 crawl listing (2.14B pages; WARC 84.78 TiB; WAT 13.92; WET 5.84; robots 0.14; non-200 2.63; index sizes)
- index.commoncrawl.org — the CDX server and the full collection list (2008–2009 ARC … CC-MAIN-2026-34)
- commoncrawl.github.io/cc-crawl-statistics — per-crawl statistics (sizes, TLDs, languages, overlaps); github.com/commoncrawl/cc-index-table — the columnar index project
- iso.org/standard/68004.html and ISO Online Browsing Platform — ISO 28500:2017, WARC file format
- tensorflow.org/datasets/catalog/c4 — C4/mC4 sizes and configs; arxiv.org/abs/1910.10683 — the T5 paper; arxiv.org/abs/2101.00027 — The Pile (via sibling guide)
- huggingface.co/datasets/togethercomputer/RedPajama-Data-1T and tiiuae/falcon-refinedweb — dataset cards; arxiv.org/abs/2306.01116 — the RefinedWeb paper
- en.wikipedia.org/wiki/Common_Crawl; en.wikipedia.org/wiki/Gil_Elbaz; en.wikipedia.org/wiki/Internet_Archive — cross-checking and citation chains into primary sources (including The Atlantic's November 2025 investigation and the April 2026 Hugging Face announcement)
- archive.org/details/wayback-machine-1996 — the Wayback Machine's first crawl (1996) and 2001 public launch
- webdatacommons.org — WDC project history, outputs and papers
- projects.propublica.org/nonprofits/organizations/261635908 — Common Crawl Foundation 990 filings

### Closing

Common Crawl is the closest thing the world has to a public copy of the web: a 501(c)(3) founded in 2007 by Gil Elbaz, collecting since 2008, publishing monthly `CC-MAIN` crawls of 2–3 billion pages each, in WARC (ISO 28500) with WAT and WET projections and CDXJ and columnar indexes on top — all free, on AWS open data and, since April 2026, a Hugging Face mirror. It is the substrate of the LLM era — C4, mC4, The Pile, RedPajama, Falcon RefinedWeb all grew from it — and of a research ecosystem from Web Data Commons to the foundation's own web graphs. It is also a sample, not the web: noisy, English-heavy, blocked-and-gappy, with real controversies over paywalls and removals, which is why every serious consumer — and certainly a regulated bank — must filter, deduplicate, provenance, and govern rather than merely download. Used that way, with the sibling acquisition guide's engineering and legal machinery behind it, Common Crawl remains what it set out to be: the open web's memory.
