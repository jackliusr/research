# Research Agents: A Comprehensive Guide

**AI Agents That Autonomously Plan, Search, Read, Synthesize, and Cite — the Discipline of Agentic Research**

> Part of the LLM/AI Architecture Series — alongside Open Research Frameworks (framework catalog), Hierarchical Multi-Agent Frameworks, Hybrid Multi-Agent Systems, Advanced RAG Techniques, and Beyond RAG guides.
> Target Audience: Solution Architects, AI Engineers, Technical Decision-Makers
> Version: 1.0 — August 2026

---

## Table of Contents

1. [What Are Research Agents?](#1-what-are-research-agents)
2. [The Research Agent Loop](#2-the-research-agent-loop)
3. [Phase 1 — Planning](#3-phase-1--planning)
4. [Phase 2 — Searching](#4-phase-2--searching)
5. [Phase 3 — Reading and Extraction](#5-phase-3--reading-and-extraction)
6. [Phase 4 — Synthesis](#6-phase-4--synthesis)
7. [Phase 5 — Citation and Grounding](#7-phase-5--citation-and-grounding)
8. [Phase 6 — Iteration and Quality Control](#8-phase-6--iteration-and-quality-control)
9. [Architecture Patterns](#9-architecture-patterns)
10. [The STORM Pattern](#10-the-storm-pattern)
11. [The Deep Research Pattern](#11-the-deep-research-pattern)
12. [The Orchestrator Role](#12-the-orchestrator-role)
13. [Practical Workflow: Building a Research Agent](#13-practical-workflow-building-a-research-agent)
14. [Tools and Libraries](#14-tools-and-libraries)
15. [Evaluation of Research Agents](#15-evaluation-of-research-agents)
16. [Production Considerations](#16-production-considerations)
17. [Banking-Specific Applications](#17-banking-specific-applications)
18. [The Future: Research Agents 2026+](#18-the-future-research-agents-2026)
19. [Conclusion and Decision Guidance](#19-conclusion-and-decision-guidance)

---

## 1. What Are Research Agents?

### 1.1 Definition

A **research agent** is an AI agent that autonomously performs end-to-end research: given a question or topic, it plans an investigation, searches multiple sources (web, internal corpora, APIs, databases), reads and extracts relevant information, synthesizes findings, and produces a cited, structured report. Unlike a chatbot that answers from parametric memory, or a RAG pipeline that retrieves once and generates, a research agent *drives its own research process* — deciding what to look for, in what order, how deeply, and when it has enough evidence to write.

The canonical pipeline:

```
Question/Topic
    │
    ▼
PLAN ────────► decompose into sub-questions, scope, depth, budget
    │
    ▼
SEARCH ──────► web, academic, news, internal corpora, APIs, databases
    │
    ▼
READ & EXTRACT ► parse pages/PDFs, pull claims, stats, quotes, entities
    │
    ▼
SYNTHESIZE ───► aggregate evidence, resolve conflicts, structure report
    │
    ▼
CITE ─────────► every claim → source (URL, title, date, access date)
    │
    ▼
ITERATE ◄───── gaps found → more targeted searches
    │
    ▼
CITED, STRUCTURED REPORT
```

The distinguishing properties of a research agent versus earlier systems:

| Property | Chatbot | RAG pipeline | Research agent |
|----------|---------|--------------|----------------|
| **Source of knowledge** | Parametric memory | Retrieved chunks (single pass) | Multi-round, self-directed retrieval |
| **Planning** | None | Implicit (query → retrieve) | Explicit plan of sub-questions & scope |
| **Tool use** | None or minimal | Retrieval only | Search APIs, browsers, parsers, databases, APIs |
| **Iteration** | None | Usually one-shot | Gap-driven re-search loops |
| **Output** | Answer text | Answer + retrieved context | Full report with inline citations & references |
| **Control** | User steers | Fixed pipeline | Agent decides next action |

### 1.2 Evolution: Q&A → RAG → Agentic Research

Research agents are the third stage of a documented progression (see the RAG series: `advanced_rag_techniques_guide.md`, `beyond_rag_guide.md` in `ai_llm/`):

1. **Single-shot Q&A (2020-2022).** LLMs answer from parametric knowledge. Fast, fluent, but static — knowledge cutoff, no sources, hallucination-prone on anything outside training data. Fine for general trivia; useless for current events, proprietary data, or anything requiring verification.

2. **RAG (2022-2024).** Retrieve-then-generate: embed a corpus, retrieve top-k chunks, stuff them into the prompt, generate. Grounded in sources, updatable, citable. But classic RAG is a *single retrieval pass* — the pipeline cannot notice it missed something and go look again, cannot decompose a complex question, cannot follow a lead. Limitations and the next-generation paradigms (agentic retrieval, memory systems, test-time compute) are cataloged in `ai_llm/beyond_rag_guide.md`. The agentic-RAG patterns — ReAct, plan-and-execute, tool-selecting agents, multi-agent RAG — are covered in `ai_llm/advanced_rag_techniques_guide.md`.

3. **Agentic research (2024-2026).** The LLM becomes the *controller of a research process*: it plans, calls search tools, reads what it finds, extracts evidence, decides what is still missing, searches again, and finally writes a cited report. The model's job is no longer "answer" but "run a research project." This shift from retrieval-as-pipeline to retrieval-as-tool-use is the essence of the research agent.

The step change matters: RAG answers "what does my corpus say about X?"; a research agent answers "what is the truth about X, given everything findable?" — and shows its work.

### 1.3 The Research Agent Boom (2024-2026)

Between late 2024 and 2026, research agents went from research prototype to mainstream product category. Key milestones:

- **STORM (Stanford, 2024)** — the academic origin of the modern pattern. Multi-perspective question asking + parallel research + outline generation + article writing with citations. Open-sourced; became the template for many later agents.
- **gpt-researcher (2024)** — early open-source autonomous research agent (plan → search → scrape → write), widely forked.
- **OpenAI Deep Research (Feb 2025)** — the product that defined the category for the general public: given a prompt, runs 20-100+ searches across the web, reads PDFs and pages, iterates, and produces a comprehensive cited report in 5-30 minutes. Built on a reasoning model fine-tuned for browsing + Python tool use.
- **Gemini Deep Research (Google, 2024-2025)** — integrated into Gemini; multi-step search with an interactive research plan the user can edit before execution; ~1.5M token context for synthesis.
- **Perplexity Deep Research (2025)** — Perplexity's take; explicit sub-question plans, parallel searches, and a "sources" panel with inline citation markers; also the Sonar API family for programmatic research.
- **Manus (2025)** — generalist agent that runs research *and* executes (browse, compute, produce deliverables like reports and spreadsheets); asynchronous cloud execution.
- **LangChain Open Deep Research (Feb 2025)** — open-source supervisor-researcher pattern on LangGraph: a supervisor plans, spawns researcher sub-agents per sub-question, and synthesizes.
- **Hugging Face smolagents / Open Deep Research (Feb 2025)** — built ODR in ~24 hours on top of smolagents (code-first agents); demonstrated how fast the pattern could be replicated.
- **Auto-Researcher (GitHub)** — open-source framework supporting single-turn, multi-turn, and team-based research modes with web search + optional web scraping.
- **OpenDeepResearch / O-Researcher / DeepSearcher / OpenManus** — the 2025-2026 open-source wave; full catalog in `open_research_frameworks_guide.md`.
- **Agentic RAG formalized** — self-RAG, CRAG, Adaptive-RAG, ITER-RETGEN, and multi-agent RAG gave the underlying control-loop vocabulary; see `ai_llm/advanced_rag_techniques_guide.md`.

Why the boom: (1) long-context models (1M+ tokens) made "read 50 sources, then write" feasible; (2) tool-use training made models reliable at search/extract calls; (3) search APIs optimized for agents (Tavily, Exa, Perplexity Sonar, Firecrawl) removed scraping friction; (4) reasoning models made planning and synthesis dramatically better; (5) enterprise demand for grounded, cited, audit-ready research — especially in banking, consulting, and legal.

### 1.4 Relationship to Sibling Guides

- `open_research_frameworks_guide.md` — the *catalog* of open-source research frameworks (ROMA, OML, LangChain ODR, smolagents, OpenManus, deep-searcher, O-Researcher). This guide covers the *discipline* — workflow, architecture, quality — and references the catalog rather than duplicating it.
- `ai_llm/hierarchical_multi_agent_frameworks_guide.md` and `ai_llm/hybrid_multi_agent_systems_guide.md` — the multi-agent architectures research agents use (supervisor-worker, parallel researchers, model routing).
- `ai_llm/advanced_rag_techniques_guide.md` / `ai_llm/beyond_rag_guide.md` — retrieval foundations and agentic RAG.
- `ai_llm/query_rewriting_rag_guide.md` — query formulation/rewriting techniques reused in the search phase.
- `ai_llm/rag_vs_long_context_llms_guide.md` — context budget trade-offs during extraction/synthesis.
- `constrained_decoding_frameworks_guide.md` — structured/JSON extraction.
- `llm_development_risks_security_guide.md` — hallucination, prompt injection, and safe tool use.

---

## 2. The Research Agent Loop

### 2.1 The Core Loop

Every research agent, whatever its architecture, implements the same core loop:

```
plan → search → read → extract → synthesize → (gaps? → search again) → produce report
```

1. **Plan** — interpret the question, decompose it into sub-questions, set scope/depth, estimate budget.
2. **Search** — issue queries to search tools; collect candidate sources (URLs, docs, API results).
3. **Read** — fetch and parse the promising sources (HTML, PDF, JSON).
4. **Extract** — pull claims, facts, statistics, quotes, and citations into an evidence store.
5. **Synthesize** — aggregate evidence by theme, resolve contradictions, structure the report.
6. **Iterate** — identify gaps in coverage; run targeted searches to fill them.
7. **Produce** — write the final cited, structured report.

The loop is *not* strictly linear: iteration is the norm. A good research agent treats phase 6 as continuous — after every synthesis pass it checks "do I have evidence for every sub-question? are the contradictions resolved? is anything thin?" and re-enters the loop with more precise queries.

### 2.2 Phases, Tools, and Deliverables

| Phase | Core activity | Typical tools | Intermediate output |
|-------|---------------|---------------|---------------------|
| Plan | Decompose question, scope, budget | LLM + prompt templates | Research plan (sub-questions, outline, budget) |
| Search | Query & collect sources | Tavily/Serper/Exa, Playwright, arXiv API, SQL | Candidate source list with metadata |
| Read | Fetch & parse | web_extract-style extractors, PyMuPDF, Jina Reader | Cleaned full text per source |
| Extract | Pull evidence | LLM + JSON schema, entity extractors | Evidence ledger (claim → source → quote) |
| Synthesize | Aggregate & structure | LLM (map-reduce, outline-driven) | Draft report sections |
| Iterate | Gap-driven re-search | Same as search + gap analysis prompt | Refined evidence set |
| Produce | Cite & format | Citation formatter, markdown/JSON renderer | Final cited report |

### 2.3 Loop-Level Design Decisions

- **When to stop searching?** Budget exhaustion (max rounds/tool calls), coverage satisfaction (all sub-questions have ≥N quality sources), or diminishing returns (last K searches added nothing). Stopping criteria should be explicit in the plan.
- **What triggers a new round?** Missing sub-question coverage, contradictory evidence needing more sources, a promising lead from snowballing, or reviewer feedback.
- **Who remembers what?** Short-term: the working context (recent pages, current extraction). Long-term: the evidence ledger and research log — persisted outside the context window so the agent doesn't lose earlier findings.
- **How much parallelism?** Independent sub-questions can be searched in parallel; dependent ones must be sequenced. Parallelism trades cost/latency for coverage; see Section 16 (concurrency limits).

### 2.4 A Worked Example: The Loop in Action

Following a concrete task through the loop makes the phases concrete. Task: *"Research how MAS-regulated banks in Singapore are approaching generative AI adoption for internal knowledge management, and recommend a pragmatic adoption path."*

1. **Plan**: decompose → (a) current MAS guidance on GenAI (FEAT principles, notices, speeches), (b) observed bank deployments (DBS, OCBC, UOB, foreign banks in Singapore), (c) common use cases and guardrails, (d) risks and governance expectations (SR 11-7 analogies), (e) adoption path synthesis. Scope: overview-to-deep-dive. Budget: 3 rounds × 6 queries, ~30 fetches.
2. **Search (round 1)**: breadth-first — queries per sub-question (MAS GenAI guidance; DBS GenAI knowledge management; Singapore bank AI governance; etc.) via Tavily; collect ~25 candidate sources; add arXiv/industry-report sources for governance frameworks.
3. **Read/extract**: fetch the top ~12 (MAS pages, bank announcements, industry surveys); extract claims into the ledger — e.g., claim "MAS published FEAT principles in 2018, updated 2024" ← source (mas.gov.sg URL, quote, date).
4. **Synthesize (draft 1)**: outline-driven draft. Gap found: sub-question (c) "common use cases" is thin — only vendor blogs; missing bank-published evidence.
5. **Iterate**: round 2 targeted queries ("DBS AI use cases employee productivity", "OCBC generative AI pilot", "MAS digital transformation reports"); snowball from a bank's own press release into its annual report PDF.
6. **Synthesize (draft 2)**: fill the gap, note the contradiction (one bank claims "1,000+ use cases", another is cautious about rollout) and present both.
7. **Review**: reviewer agent flags weak sourcing on the governance section → round 3 searches MAS speeches/consultation papers; re-verify URLs.
8. **Produce**: final report — executive summary, 5 sections, inline citations, limitations (excludes non-English sources, unverified vendor claims), methodology note.

The whole run: ~25-40 minutes, ~$2-5 in tokens/search costs, 30+ distinct sources, every claim traceable.

---

## 3. Phase 1 — Planning

Planning is the most underrated phase. Research agents fail as often from bad plans (vague scope, unbounded budget, missing sub-questions) as from bad searching. The plan is the contract between the agent and the question.

### 3.1 Query Analysis

The agent first analyzes the research question to understand what is actually being asked:

- **Intent**: What kind of answer is wanted — a definition, a comparison, a causal analysis, a forecast, a decision memo?
- **Entities**: What people, organizations, products, regulations, dates are named? These become search terms and extraction targets.
- **Assumptions embedded in the question**: e.g., "Why is X better than Y?" assumes X is better — the agent should be able to challenge this and look for counter-evidence.
- **Ambiguity**: "AI risk" could mean model risk, security risk, or regulatory risk. The plan should disambiguate or explicitly cover multiple readings.
- **Currency requirement**: Does the answer need current data (news, prices, regulations) or is historical/static knowledge fine?

### 3.2 Decompose into Sub-Questions (The Research Plan)

The core planning act: break the question into a small set of sub-questions whose answers, combined, answer the main question. This is the same decomposition used by query-decomposition RAG (`ai_llm/query_rewriting_rag_guide.md`), scaled up to a whole research project.

Example — "Assess the feasibility of a MAS-regulated digital asset custody service for a Singapore bank":

1. What are MAS's current requirements for digital asset custody (PSA, DPT licensing, custody standards)?
2. What technology stacks are used by licensed custodians (cold storage, HSMs, MPC, settlement)?
3. Who are the existing licensed providers in Singapore and what are their models?
4. What are the key risks (operational, cyber, legal) and mitigations?
5. What are the cost and timeline drivers for implementation?
6. What are the competitive/market dynamics in 2025-2026?

Each sub-question gets its own search strategy, sources, and target depth. The plan also records *knowledge gaps* — what the agent already knows (or suspects) versus what must be found. This gap list drives the first search round and later gap-driven re-search (Section 8).

### 3.3 Scope and Depth

The agent must decide (or be told) the depth of research:

| Depth | Definition | Typical use | Budget implication |
|-------|-----------|-------------|--------------------|
| **Overview** | Top-line facts, key sources, executive-level answer | Quick briefings, scoping conversations | 5-15 searches, 10-20 min |
| **Deep-dive** | Sub-question coverage, multiple sources per claim, contradictions noted | Decision support, market scans | 20-60 searches, 20-60 min |
| **Exhaustive** | Systematic coverage, all major sources, methodology, limitations, audit trail | Regulatory analysis, due diligence, research reports | 60-200+ searches, hours |

Depth should be *chosen, not defaulted*. The plan states it explicitly so the agent knows when to stop: an overview does not need 40 sources per sub-question; an exhaustive review does.

### 3.4 Budget Estimation

Before executing, the plan fixes the iteration budget:

- **Search rounds**: max number of search cycles (e.g., 3 rounds × 8 queries).
- **Tool calls**: cap total tool invocations (searches + fetches + parses) to bound cost and runtime.
- **Token budget**: estimate read+extract+generate tokens; reserve the largest share for synthesis. See `ai_llm/rag_vs_long_context_llms_guide.md` for long-context economics.
- **Time budget**: wall-clock target (deep research products quote 5-30 minutes).

The budget is a *soft* cap: dynamic replanning (3.6) can raise it when the question turns out harder than expected, but only deliberately and with justification recorded.

### 3.5 Report Structure Planning

The plan should sketch the target report structure: sections, depth per section, and the citation style. A good plan already anticipates the outline (this is the "outline-driven" synthesis pattern in Section 6). For the custody example:

```
1. Executive summary
2. MAS regulatory landscape (deep) [sub-questions 1]
3. Technology options (deep) [2]
4. Competitive landscape (medium) [3]
5. Risk analysis (medium) [4]
6. Cost/timeline drivers (medium) [5]
7. Market outlook (light) [6]
8. Limitations & methodology
```

### 3.6 Planning Techniques

- **Single-shot plan**: write one complete plan, execute it, done. Simple, predictable, cheap — but brittle: if early findings contradict the plan's assumptions, the agent continues down a wrong path. Appropriate for well-understood question types and bounded overviews.
- **Iterative planning**: re-plan as findings arrive — after each research round, update the sub-question list, add/remove sections, revise budget. More robust; the norm for deep research products.
- **Hierarchical planning**: plan → sub-plans per section. The top plan allocates sub-questions and budgets; each section gets its own mini-plan (sources, depth). This is the supervisor-worker decomposition (Section 9); see `ai_llm/hierarchical_multi_agent_frameworks_guide.md`.
- **Dynamic replanning**: the agent monitors execution for dead ends and redirects — e.g., a sub-question with zero quality sources is re-scoped or dropped with a note; a surprising finding triggers a new sub-question; budget is reallocated from a saturated topic to a thin one. This is what separates agentic research from a fixed pipeline.

**Planning pitfalls**: over-planning (spending budget on the plan itself), under-planning (vague scope → aimless searching), planning around the agent's assumptions instead of the evidence, and forgetting stopping criteria.

### 3.7 Planning Output: A Research Plan Schema

A plan should be a structured artifact, not prose. A JSON plan schema makes the plan machine-checkable, reviewable by humans, and auditable:

```json
{
  "question": "How are MAS-regulated banks approaching GenAI for knowledge management?",
  "depth": "deep-dive",
  "sub_questions": [
    {"id": "SQ1", "text": "What does MAS currently require/guide on GenAI?",
     "sources": ["mas.gov.sg", "academic", "consultancy"], "depth": "deep"},
    {"id": "SQ2", "text": "Which Singapore banks have deployed GenAI KM, and how?",
     "sources": ["bank announcements", "news", "annual reports"], "depth": "deep"},
    {"id": "SQ3", "text": "What use cases and guardrails are emerging?",
     "sources": ["industry surveys", "vendor blogs"], "depth": "medium"}
  ],
  "budget": {"max_rounds": 3, "max_queries_per_round": 6, "max_fetches": 30,
             "token_budget": {"search": "10%", "extract": "40%", "synthesize": "40%", "review": "10%"}},
  "outline": ["exec_summary", "MAS_guidance", "bank_deployments",
              "use_cases_and_guardrails", "risks_and_governance", "adoption_path",
              "limitations", "methodology"],
  "citation_style": "numbered",
  "stopping_criteria": ["budget_exhausted", "all_sqs_have_2_plus_quality_sources",
                        "diminishing_returns_last_round"]
}
```

The plan is a living document: iterative planning (3.6) updates it between rounds; the final version is archived with the report as part of the audit trail.

---

## 4. Phase 2 — Searching

### 4.1 Search Tools

The search phase gathers candidate sources. The tool mix depends on the domain:

**Web search APIs** (programmatic, agent-friendly):
- **Tavily** — purpose-built for agents; returns clean content snippets alongside results; supports search depth and topic filters.
- **Serper** — Google SERP API; cheap, fast, standard Google results as JSON.
- **Exa** — "search API for AI": embeddings-based neural search over the web, contents endpoint returns full text; native MCP support (Section 14).
- **Perplexity Sonar** — LLM-backed search API; returns cited answers that can seed the source list.
- **Google/Bing official APIs** — reliable but more expensive and quota-bound.
- **SearXNG** — self-hosted metasearch aggregator; useful for privacy and for avoiding per-vendor quotas; results quality varies by upstream.

**Browser-based search** (when APIs are insufficient):
- **Playwright / Puppeteer** — drive a real browser: execute JS-heavy pages, handle logins/cookies, crawl paginated results, extract rendered DOM. Slower and more fragile than APIs, but necessary for sites that block bots or render client-side.

**Academic search**:
- **arXiv API** — preprints; full-text PDFs; the `arxiv` skill covers query/search workflows.
- **Semantic Scholar API** — papers, citations, abstracts; the citation graph is excellent for snowballing (4.3).
- **Google Scholar** — best coverage but no official API; scraping is against ToS — prefer Crossref/OpenAlex/Semantic Scholar programmatic routes.
- **PubMed** — biomedical literature (E-utilities API).
- **OpenAlex / Crossref** — open scholarly metadata and citation graphs.

**News**:
- **GDELT** — global news graph, free, huge historical coverage; good for event timelines and sentiment shifts.
- **NewsAPI** — mainstream news headlines/articles; quota-limited free tier.
- Vendor news endpoints (e.g., Bloomberg, Reuters APIs) — for banking-grade feeds.

**Internal corpora & databases**:
- **Vector search** over internal document stores (policy docs, previous research, legal opinions) — standard RAG tooling; see the RAG series.
- **SQL databases** — structured queries for numbers (balances, limits, exposures).
- **APIs** — internal or third-party data services (market data, reference data, registry lookups).
- **Document stores** — SharePoint/Confluence/email archives via connectors.

### 4.2 Search Techniques

- **Query formulation**: never rely on one query. Generate multi-query variants (synonyms, phrasings, languages), and use query rewriting to convert a vague need into precise search terms — the full technique catalog is in `ai_llm/query_rewriting_rag_guide.md`. Include domain vocabulary: for banking research, "custody" should also be searched as "safekeeping", "digital asset custody", "PSA license", etc.
- **Iterative deepening**: search → read → refine queries. First-round results reveal vocabulary and sub-topics that sharpen second-round queries ("search → read → refine" is the loop's engine).
- **Breadth-first vs depth-first**: breadth-first covers all sub-questions shallowly first (good for building the map, catching surprises); depth-first follows the most promising lead exhaustively before moving on (good when one sub-question dominates). Deep research agents typically do breadth-first rounds interleaved with depth-first follow-ups on promising leads.
- **Snowballing (citation chaining)**: take the references of a good paper/report and search *them*; take who cites a key paper (forward chaining via Semantic Scholar). This is how research agents find primary sources and the academic backbone of a topic.
- **Source diversity**: deliberately mix academic + industry + news + primary sources (regulators, filings, standards bodies). A report built only from blog posts is weak; one built only from papers misses practice.
- **Recency filtering**: prioritize recent sources for fast-moving topics (regulations, markets); date metadata is a first-class filter, not an afterthought.
- **Credibility filtering**: rank by domain reputation, peer-review status, official-source status (regulator/exchange/issuer domains), and authoritativeness. This pre-filtering feeds the evidence weighting in synthesis (Section 6).

### 4.3 Search Output

The search phase must produce a **candidate source list**, not raw results: each entry carries URL, title, publisher, date, snippet, credibility score, and which sub-question it addresses. This list is deduplicated, prioritized, and fed to the reading phase. Keeping this metadata is also the seed of the audit trail (Section 17).

### 4.4 Choosing Search Tools: A Selection Matrix

| Need | Recommended | Why |
|------|-------------|-----|
| Agent-native web search, clean snippets | Tavily, Exa | Results + content in one call; purpose-built for loops |
| Google-quality SERP results | Serper, Google API | Familiar ranking, broad coverage |
| LLM-answer + source seeds | Perplexity Sonar | Cited answer can bootstrap the source list |
| JS-heavy / bot-protected pages | Playwright/Puppeteer + Firecrawl | Real browser rendering, stealth handling |
| Academic literature | arXiv API, Semantic Scholar, OpenAlex | Full-text + citation graph for snowballing |
| News/events, timelines | GDELT, NewsAPI | Historical coverage, event graphs |
| Internal documents | Vector search over internal store | Private, permissioned retrieval |
| Structured facts | SQL, internal APIs | Exact numbers, registry lookups |
| Privacy / no vendor lock-in | SearXNG (self-hosted) | Metasearch, own infrastructure |

**Pragmatic default for a first build**: Tavily or Exa for web + arXiv/Semantic Scholar for academic + a local extraction library (Trafilatura). Add browser automation and internal vector search only when the sources demand them.

---

## 5. Phase 3 — Reading and Extraction

### 5.1 Reading: From URL to Clean Text

- **Full-page extraction**: web_extract-style tools convert HTML → clean markdown/text (strip nav, ads, boilerplate). Jina Reader (`r.jina.ai`) and Firecrawl do this as a service; Trafilatura and Newspaper3k do it locally.
- **PDF parsing**: reports, filings, and papers are PDFs. PyMuPDF for text + layout, marker-pdf for complex/OCR-heavy documents, LlamaParse for LLM-ready structured parsing of messy docs. See the `ocr-and-documents` skill for the full extraction workflow.
- **Chunking**: long documents are split for extraction (the same chunking trade-offs as RAG apply — see `ai_llm/advanced_rag_techniques_guide.md`).
- **Relevance filtering**: not everything fetched is worth reading. Score each page against the sub-question before extracting; skip irrelevant pages *after recording their URL in the research log* (what was read-and-discarded is part of the audit trail).

### 5.2 Extraction: What to Pull Out

For each relevant source, extract structured evidence:

- **Key claims** — the substantive assertions relevant to the research question.
- **Facts and statistics** — numbers, dates, figures (with their units and context).
- **Quotes** — verbatim passages that will be cited directly in the report.
- **Citations** — the source's own references (enables snowballing) and its identity metadata (title, author, publisher, date, URL).
- **Entities** — people, organizations, dates, product names, regulation identifiers (e.g., "MAS Notice 645").
- **Structured extraction (JSON)** — enforce a schema per source so evidence is uniform and machine-checkable. JSON-mode/constrained decoding makes extraction reliable; see `constrained_decoding_frameworks_guide.md`.

### 5.3 Note-Taking: The Evidence Ledger

The single most important artifact in a research agent is the **evidence ledger** — a persisted mapping of claim → source → quote. Every extracted claim is recorded with:

- The claim text (normalized)
- Source URL + title + publisher + publication date + access date
- The verbatim quote supporting it
- Which sub-question it addresses
- A confidence/credibility tag (source quality, recency)

The ledger is the foundation of citation quality (Section 7) and the audit trail (Section 17). If the agent can't produce a ledger, it can't produce trustworthy citations — no amount of clever writing fixes missing provenance.

### 5.4 Extraction Techniques

- **Extract-then-synthesize**: extract facts per source *during* reading; defer all synthesis to phase 4. Mixing extraction and synthesis in one pass produces summaries-of-summaries and loses the claim→source mapping.
- **Incremental summarization**: summarize as you go to avoid context overflow. A 50-source deep dive can exceed any context window; per-source extraction + rolling per-sub-question summaries keep the working set bounded. Budget guidance in `ai_llm/rag_vs_long_context_llms_guide.md`.
- **Evidence ledger discipline**: every claim linked to source + quote, *at extraction time* — retrofitting citations after writing is where hallucinated URLs sneak in.

### 5.5 Evidence Ledger: Schema and Example

A ledger entry is a row in a structured store (JSON lines, SQLite, or a table) with a fixed schema:

```json
{
  "entry_id": "E042",
  "sub_question": "SQ2",
  "claim": "DBS reported deploying GenAI across 1,000+ use cases by early 2026",
  "source": {
    "url": "https://www.dbs.com/newsroom/...",
    "title": "DBS Annual Report 2025",
    "publisher": "DBS Group Holdings",
    "published": "2026-03-14",
    "accessed": "2026-08-03"
  },
  "quote": "As of December 2025, over 1,000 GenAI use cases have been deployed across the Group.",
  "confidence": "high",
  "credibility": "primary_official",
  "conflicts_with": ["E058"],
  "notes": "Annual report PDF, page 42; figure includes internal + client-facing use cases"
}
```

Ledger practices that separate good agents from bad ones:

- **One entry per claim** — atomic rows, not paragraph blobs; enables dedup, conflict detection, and per-claim verification.
- **Normalize entities** — "DBS", "DBS Group", "DBS Bank" map to one entity id so claims can be grouped.
- **Conflict tagging** — when a new claim contradicts an existing one, link them (see E058 above) so synthesis surfaces the disagreement rather than hiding it.
- **Never edit, only append** — the ledger is append-only; corrections add a new entry with a reference to the superseded one. This preserves the audit trail.
- **Persist outside context** — the ledger lives in storage, not in the prompt; only working subsets are loaded into context at any time.

---

## 6. Phase 4 — Synthesis

Synthesis is where evidence becomes a report. It is also where research agents most often fail — by writing plausible prose that drifts from the evidence. The guardrails are: aggregate from the ledger, not from memory; structure before writing; cite while writing.

### 6.1 Aggregate Evidence

- **Group by theme/claim**: cluster ledger entries by sub-question and by claim. Contradictory claims about the same fact land in the same cluster — that is the point.
- **Resolve contradictions**: when sources disagree, do not silently pick a winner. Note the disagreement explicitly in the report ("Sources differ: MAS guidance states X; industry commentary suggests Y"), attribute each side, and where possible find a tiebreaker source (the regulator's own FAQ, the primary study, newer data).
- **Weigh evidence**: weight by source quality (official/peer-reviewed > reputable industry > blog/aggregator), recency (relevant for fast-moving topics), and sample size/representativeness (a survey of 12 banks vs 1,200). Weighting informs both the report's confidence statements and its structure (strong evidence → firm claims; weak evidence → hedged claims).
- **Identify gaps**: as clusters form, mark sub-questions with thin or missing evidence. These gaps trigger the iteration phase (Section 8) — targeted searches, not acceptance of a weak section.

### 6.2 Structure the Report

Work from the plan's outline (Section 3.5), adjusted by what the evidence supports: drop sections with no evidence (note the omission in Limitations), merge thin sections, split overloaded ones. The skeleton is: executive summary → body sections (each answering a sub-question) → limitations → methodology note → references.

### 6.3 Write

- **Abstract / executive summary**: the answer, the key findings, the confidence level, the most important sources — written last, from the finished body.
- **Body with evidence**: each section states its claims, supports them with ledger evidence, and marks confidence ("MAS Notice 645 requires X [1]" vs "Industry sources suggest Y [4],[7]").
- **Citations inline**: numbered markers [1], [2] bound to the reference list; every factual claim gets one.
- **Limitations section**: what the research could not cover, conflicting evidence left unresolved, source gaps, recency constraints. A limitations section is a sign of quality, not weakness — and is increasingly expected in banking-grade outputs.
- **Methodology note**: how the research was done — tools used, search strategy, number of sources, date of research, budget spent. This is what makes the output reproducible and auditable (SR 11-7 context, Section 17).

### 6.4 Synthesis Patterns

- **Map-reduce**: synthesize each section independently from its evidence cluster (map), then merge into the full report with a consistent executive summary and cross-references (reduce). Natural fit for parallel architectures (Section 9); each map step is a bounded context, avoiding long-context degradation (see `ai_llm/rag_vs_long_context_llms_guide.md`).
- **Outline-driven**: fill the plan's outline section by section, in order, from the evidence. Simple, deterministic, easy to review; the default for single-agent systems.
- **Claim-verification**: draft claims first (from the ledger, one per line), verify each against its source quote, then write prose around verified claims. This separates "what can we say" from "how do we say it" and is the strongest anti-hallucination synthesis pattern.
- **Contradiction resolution**: explicitly present both sides with sourcing, per cluster (6.1). The report should never hide disagreement — hiding it is how confident-sounding-but-wrong research gets published.

### 6.5 A Report Template

A reusable skeleton keeps output consistent across tasks and makes automated quality checks (Section 8.2) straightforward:

```markdown
# <Research Question>
**Prepared:** <date> · **Research agent:** <agent/version> · **Depth:** <overview|deep-dive|exhaustive>

## Executive Summary
<The answer in 3-5 sentences; key findings with citations; confidence level; biggest caveat>

## 1. <Section answering SQ1>
### 1.1 <Sub-theme>
<Claims with inline citations [1], [2]; contradictions presented with both sides sourced>

## 2. <Section answering SQ2>
...

## N. Limitations
<What could not be covered; unresolved conflicts; source gaps; recency constraints>

## Methodology
<Search tools, query strategy, rounds, number of sources, budget spent, dates>

## References
[1] Author/Org. Title. Publisher, date. URL (accessed <date>)
[2] ...
```

---

## 7. Phase 5 — Citation and Grounding

### 7.1 Citation Requirements

Every factual claim must trace to a source. The minimum record per citation:

- **URL** (and DOI/identifier where applicable)
- **Title**
- **Publication date** (and access date — the research date matters, see Section 17)
- **Publisher/source name**
- **Inline citation marker** in the text ([1], [2], or author-date)
- **Reference list** at the end: numbered or bibliographic (APA/Chicago), consistent throughout

### 7.2 Grounding Verification

Before the report ships, run a grounding check: walk every factual claim and confirm it has a ledger entry with a real source. Claims without a source are either dropped, marked as the agent's inference/analysis (clearly separated from sourced facts), or sent back for another search round. This is the hallucination check — the moment where unsupported prose is caught. See the hallucination and overreliance material in `llm_development_risks_security_guide.md`.

### 7.3 Citation Quality Hierarchy

Not all citations are equal. Prefer, in order:

1. **Primary > secondary**: the regulator's notice over a news article about it; the study over the blog summarizing it.
2. **Official > aggregated**: exchange/regulator/issuer statements over aggregator sites.
3. **Recent > dated**: for fast-moving topics (regulatory, market data), a 2026 source beats a 2023 one; note the date explicitly.
4. **Specific page/section > whole document**: cite the section you actually read ("MAS Notice 645, para 3.2") over a bare home-page URL.

### 7.4 Tools

- **Automatic citation capture during extraction**: the evidence ledger (Section 5.3) is the citation pipeline — capture URL/title/date/quote once, at extraction time, and citations fall out at writing time.
- **Citation formatting**: render the ledger into APA/Chicago/numbered references automatically; keep a single formatting layer so style is consistent.
- **Anti-hallucination rules**: (1) only cite what was actually read — never fabricate URLs, titles, or page numbers; (2) verify URLs exist (fetch or HTTP-check before finalizing; dead links are cited-but-unverifiable); (3) never inherit citations from an intermediate AI answer without checking the underlying source. Fabricated citations are the #1 credibility killer for research agents — see `llm_development_risks_security_guide.md` for the failure modes.

### 7.5 A Citation Example, Good and Bad

**Good (specific, primary, dated):**
> MAS Notice 645 (prevention of money laundering and countering the financing of terrorism) was last revised effective 1 April 2025 [1], and its companion guidelines explicitly extend to digital payment token service providers [2].
> [1] Monetary Authority of Singapore. *Notice 645 — Prevention of Money Laundering and Countering the Financing of Terrorism*. MAS, rev. 1 Apr 2025. https://www.mas.gov.sg/regulation/notices/notice-645 (accessed 3 Aug 2026)
> [2] MAS. *Guidelines on AML/CFT for Digital Payment Token Service Providers*, para 3.2. (accessed 3 Aug 2026)

**Bad (fabricated or vague):**
> According to recent reports, MAS has updated its AML rules for crypto firms [3]. — *[3] references a bare homepage URL, no title/date, and the claim cannot be located anywhere on it.*

The difference is entirely a function of the evidence ledger: good citations are ledger rows rendered; bad citations are prose inventions.

---

## 8. Phase 6 — Iteration and Quality Control

Research quality is iterative: the first synthesis draft is a hypothesis, not a deliverable.

### 8.1 Quality Loops

- **Gap-driven re-search**: the synthesis phase marks thin sub-questions (6.1); the agent runs targeted searches for exactly those gaps, then re-synthesizes the affected sections. This loop (search → synthesize → find gaps → search) is what makes deep research "deep."
- **Reviewer agent**: a second agent — with fresh context and no sunk cost in the draft — critiques it: missing topics, weak sourcing, contradictions, unsupported claims, structural problems. The reviewer's output feeds another iteration. This is the researcher-critic architecture (Section 9); the multi-agent review mechanics are in `ai_llm/hierarchical_multi_agent_frameworks_guide.md`.
- **Self-reflection**: the researching agent reviews its own output against its evidence ledger — checking every claim is grounded, every citation real, every sub-question answered. This mirrors self-RAG's reflection tokens applied to whole reports; see `ai_llm/advanced_rag_techniques_guide.md`.
- **Human-in-the-loop**: approval gates at three points — plan (is the scope right?), draft (is the direction right?), final (is it publishable?). For high-stakes outputs (investment decisions, regulatory submissions), the final gate is mandatory (Section 17).

### 8.2 Quality Metrics

| Metric | Definition | Why it matters |
|--------|-----------|----------------|
| **Source coverage** | Number of distinct quality sources per sub-question | Thin coverage = thin conclusions |
| **Citation density** | Claims per citation (lower is denser; every claim should have one) | Density of unsupported claims is the hallucination proxy |
| **Factual accuracy** | Human-evaluated correctness on a sample of claims | The ultimate quality measure |
| **Freshness** | Median source date vs research date | Stale sources undermine fast-moving topics |
| **Completeness** | Fraction of sub-questions answered with evidence | Did we actually answer the question? |
| **Balance** | Both sides represented on contested points | One-sided reports mislead decision-makers |

Track these per research task and across tasks; they become the regression suite for agent changes (Section 15).

### 8.3 A Pre-Release Quality Checklist

Before a report is released, run this checklist — automated where possible, human-reviewed where it matters:

1. **Completeness**: every sub-question in the plan has an answered section with evidence; unanswered ones are declared in Limitations.
2. **Grounding**: every factual claim in the body maps to a ledger entry (automated: extract claims, join to ledger, flag orphans).
3. **Citation integrity**: every reference is real (URL check), every inline marker has a reference, no orphan markers (automated).
4. **Contradiction audit**: all ledger `conflicts_with` links are either resolved in the text or presented as open disagreements.
5. **Balance check**: for contested topics, both sides appear with sources (reviewer agent / human).
6. **Freshness**: source dates are within the task's recency window; research date recorded on the report.
7. **Limitations present**: the report states what it did not cover and why.
8. **Methodology present**: reproducible description of how the research was done.

Items 3-4 are cheap to automate and catch the most common failure modes; items 5-6 need a reviewer; item 7-8 are prompt-enforced.

---

## 9. Architecture Patterns

Six recurring architectures cover nearly all research agents. They compose — most production systems are a tool-augmented single agent with a reviewer, or a supervisor-worker tree with parallel researchers.

| Pattern | Structure | Strengths | Weaknesses | Best for |
|---------|-----------|-----------|------------|----------|
| **Single-agent** | One agent does everything | Simple, cheap, coherent | Limited parallelism, context-bound, one viewpoint | Overviews, bounded deep-dives |
| **Supervisor-worker** | Supervisor plans; workers research per sub-question; supervisor synthesizes | Parallelism, per-topic focus, STORM-style | Coordination overhead, synthesis bottleneck | Deep research, many sub-questions |
| **Researcher-critic** | Researcher drafts; critic reviews; iterate | Quality, catches errors, adversarial collaboration | 2x cost, iteration latency | High-stakes reports |
| **Parallel researchers** | N researchers on N topics in parallel; merge | Speed, coverage | Merge quality, duplicated effort | Broad multi-topic research |
| **Pipeline** | Sequential stages: planner → searcher → extractor → synthesizer → citer | Deterministic, testable per stage | Rigid, no feedback between stages | Productionized fixed-scope research |
| **Tool-augmented single agent** | One agent, many tools, self-directed loop | Most flexible, most common in production | Needs strong model, monitoring | General-purpose research agents |

- **Single-agent**: one model instance runs the whole loop. Simple and coherent, but serial (no parallel searches), context-bound, and single-viewpoint. Fine for overviews and small deep-dives.
- **Supervisor-worker**: the supervisor decomposes the question into sub-questions and spawns a worker per sub-question (or per section); workers research in parallel and return evidence; the supervisor synthesizes. This is the STORM-style pattern (Section 10) and the standard for serious deep research. Worker orchestration details in `ai_llm/hierarchical_multi_agent_frameworks_guide.md`.
- **Researcher-critic**: a researcher produces a draft; a critic — prompted to be adversarial, with the evidence ledger — finds weaknesses; the researcher revises; repeat until the critic passes it. Expensive, but the strongest quality architecture.
- **Parallel researchers**: N independent researchers on N topics, then a merge agent unifies into one report. Maximum speed and coverage; watch for conflicting claims across researchers and inconsistent citation styles at merge time. See `ai_llm/hybrid_multi_agent_systems_guide.md`.
- **Pipeline**: fixed sequential stages, each a specialized component (planner → searcher → extractor → synthesizer → citer). Deterministic, testable, cheap to operate — but cannot adapt mid-run. Good for standardized, repeatable research products.
- **Tool-augmented single agent**: one agent with a rich tool belt (search API, extractor, PDF parser, vector store, SQL, citation formatter) driving the loop itself. The most common production pattern — flexible, observable, and the recommended starting point (Section 19).

### 9.1 Choosing an Architecture

A decision procedure, not a preference:

1. **Start single**: one tool-augmented agent, the full loop, an evidence ledger. If it answers the question class well enough, stop — this is most use cases.
2. **Add a critic** when output quality is the constraint: researcher-critic iteration (8.1). Quality, not speed, is the goal.
3. **Add supervisor-worker** when the question has 5+ independent sub-questions and wall-clock speed matters: parallel workers per sub-question.
4. **Add parallel researchers** when topics are genuinely independent (multi-topic scans) — but budget for merge effort.
5. **Use a pipeline** only for fixed, repeatable research products where adaptability is not needed — the plan/search/extract/synthesize stages never change.

**Composition is normal**: a supervisor-worker tree whose workers are tool-augmented single agents, with a researcher-critic loop at the top, is a common production shape. Each pattern is a layer, not a religion. The hierarchical mechanics (communication, aggregation, failure propagation) are detailed in `ai_llm/hierarchical_multi_agent_frameworks_guide.md` and `ai_llm/hybrid_multi_agent_systems_guide.md`.

---

## 10. The STORM Pattern

STORM (Stanford, 2024 — "Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking") is the academic origin of modern research agents, and its structure is reused by many later systems (including elements of LangChain ODR and various open-source agents).

**The four STORM stages:**

1. **Multi-perspective question asking**: instead of one research question, STORM simulates different expert perspectives (personas — e.g., a regulator, an economist, a technologist) and generates questions *each would ask* about the topic. This broadens coverage beyond a single naive decomposition — each persona surfaces different sub-questions, vocabulary, and sources.
2. **Parallel research per perspective**: for each perspective's questions, the system searches and reads sources independently (in parallel), building a per-perspective knowledge base.
3. **Outline generation**: from the collected evidence, STORM generates a topic outline — the section structure for the article, grounded in what was actually found.
4. **Article writing with citations**: the outline is filled in section by section, with inline citations to the collected sources.

**Why it matters**: the multi-perspective step solves the "you don't know what you don't know" problem — a single-perspective plan misses the questions another expert would ask. Any research agent can adopt just this idea (persona-based question generation in the planning phase) without adopting the full architecture. The supervisor-worker research pattern (Section 9) is the generalization: perspectives ≈ worker research streams, outline generation ≈ synthesis planning.

### 10.1 Implementing STORM-Style Multi-Perspective Planning

For a topic like "digital asset custody in Singapore," persona questions look like:

- **Regulator persona**: What are MAS's licensing and custody requirements? What AML/CFT obligations apply? What enforcement actions signal supervisory expectations?
- **Technology persona**: What custody architectures exist (cold storage, MPC, HSM)? What are the operational risks and failover requirements?
- **Economist/business persona**: What is the market size and growth? Who are the competitors and what are their pricing models?
- **Risk/legal persona**: What are the legal classification and insolvency-remoteness questions? Who bears custodian failure risk?

Each persona generates 5-10 questions; the union (deduplicated) becomes the sub-question list. This is strictly better than a single "expert" decomposition because each persona's vocabulary seeds different search queries, and the sources each persona would trust differ (a regulator's answers come from MAS.gov.sg; a technologist's from vendor white papers and standards bodies).

Implementation notes: personas are cheap (one LLM call per persona at plan time); they also directly support the multi-role team architectures of Section 9 — each persona can become a worker agent with its own search strategy.

---

## 11. The Deep Research Pattern

The "deep research" pattern — popularized by OpenAI Deep Research (Feb 2025) and since copied by Gemini, Perplexity, and the open-source ecosystem — is the production form of the research agent:

- **Extensive parallel searches**: dozens to hundreds of searches, run in parallel where independent, across web, PDFs, and specialized sources.
- **Multi-round refinement**: search → read → evaluate coverage → search again with refined queries; the agent reasons about what it still needs before each round.
- **Comprehensive final report with citations**: a long-form, structured, cited report — executive summary, sections, references — typically 5-30 minutes of wall-clock time.
- **Reasoning model at the core**: the planner/synthesizer is a reasoning model; search and extraction may run on cheaper models (Section 16, model routing).

**Deep research vs quick search** — route by question complexity:

| Dimension | Quick search | Deep research |
|-----------|-------------|---------------|
| **Question** | Simple fact: "What is MAS Notice 645?" | Complex: "How would MAS Notice 645 affect our digital custody product?" |
| **Sources** | 1-5, top results | 20-100+, diverse, snowballed |
| **Depth** | One answer | Sub-questions, contradictions, weighing |
| **Time** | Seconds to minutes | 5-30+ minutes |
| **Cost** | Cents | Dollars (token-hungry) |
| **Output** | Short answer + links | Long report + citations + limitations |

Rule of thumb: quick search when the answer is a fact; deep research when the answer is an *assessment* that must weigh many sources. A research agent should itself be able to make this routing decision — cheap first pass, escalate when the question resists a simple answer.

---

## 12. The Orchestrator Role

In practice, research agents are usually *dispatched* by an orchestrator — a parent agent (or human) that owns the overall workflow:

1. **Topic intake**: receive the research question, scope, depth, and output format from the user/parent agent.
2. **Dispatch**: launch the research agent (or agent team) with a complete task spec — question, sub-question expectations, budget, citation style. In agent platforms this is a `delegate_task`-style call: pass the spec, get back the report and the research log.
3. **Review results**: check the report against the spec (did it answer the question? is it well-sourced?), run quality checks, and reject/return for another iteration if deficient.
4. **Iterate**: refine the spec based on the first output (narrow scope, add constraints, request more depth on a section) and re-dispatch.
5. **Save/commit output**: store the final report, the evidence ledger, and the research log in the knowledge base / repository — the deliverable is the *package* (report + provenance), not just the prose.

The orchestrator is also the boundary for human-in-the-loop gates: it presents the plan and draft for approval and blocks final publication until sign-off (Section 8.1).

---

## 13. Practical Workflow: Building a Research Agent

A seven-step path from idea to production research agent.

### Step 1 — Define the Research Task Schema

The interface contract between the world and your agent:

- **Input**: question, scope (topics covered), depth (overview/deep-dive/exhaustive), output format (markdown/JSON/slides), citation style, budget limits, optional constraints (sources to include/exclude, language, recency window).
- **Output**: report structure (sections), citation style, evidence ledger (claim → source → quote), research log (what was searched/read/used), limitations.

Define this schema before any code — it drives the prompts, the tool set, and the evaluation set.

### Step 2 — Choose the Agent Framework

Options: LangGraph (graph-based control flow, supervisor-worker built in), LlamaIndex (workflow/agent abstractions, strong RAG integration), smolagents (code-first, lightweight), OpenAI Agents SDK / Claude Agent SDK (managed platforms), or a custom loop (a `while` loop over tool calls — fully transparent, more work). The full framework catalog and comparison is in `open_research_frameworks_guide.md`; `ai_llm/rag_frameworks_comparison_guide.md` covers the RAG-side infrastructure. Choose by: control-flow needs (do you need supervisors? parallelism?), model flexibility, and observability requirements.

### Step 3 — Build the Tool Set

- Search API (Tavily/Serper/Exa) + optional browser automation (Playwright) for blocked sites.
- Extractor (Jina Reader / Firecrawl / Trafilatura) and PDF parser (PyMuPDF / marker-pdf).
- Vector store for internal corpora; SQL/API connectors for structured data.
- Citation formatter + a persistence layer for the evidence ledger.

### Step 4 — Implement the Loop

Wire the phases: plan (prompt with task schema → research plan JSON) → search (tool calls per sub-question) → read (fetch + parse) → extract (JSON extraction into the ledger) → synthesize (outline-driven or map-reduce) → cite (ledger → inline markers + references) → iterate (gap analysis → re-search). The core is a state machine: plan state, source list, evidence ledger, draft, review notes.

```python
# Minimal tool-augmented loop (pseudocode)
state = {"task": task_schema, "plan": None, "sources": [], "ledger": [], "draft": None}

state["plan"] = planner.plan(task_schema)              # sub-questions, budget, outline
for round in range(plan["max_rounds"]):
    queries = searcher.formulate(state["plan"], state["ledger"])  # multi-query, gap-driven
    state["sources"] += search_api.run(queries)        # parallel where independent
    for src in prioritize(state["sources"]):           # credibility + relevance filter
        text = extractor.fetch(src.url)                # web_extract-style / PDF parser
        state["ledger"] += extractor.extract(text, src, schema)   # claims → sources → quotes
    gaps = gap_analyzer.analyze(state["ledger"], state["plan"])   # thin sub-questions
    if not gaps or round == plan["max_rounds"] - 1: break
    state["plan"] = replanner.update(state["plan"], gaps)         # dynamic replanning
state["draft"] = synthesizer.write(state["ledger"], state["plan"])  # outline-driven
state["draft"] = reviewer.revise(state["draft"], state["ledger"])   # critic loop
render_citations(state["draft"], state["ledger"], style=task_schema["citation_style"])
```

### Step 5 — Add Quality Controls

- Reviewer agent pass (critic loop) and grounding verification (every claim → ledger entry).
- Citation verification: URL existence checks, reference-list consistency.
- Gap detection before final: re-run the completeness check (all sub-questions answered?).
- Human gates at plan and final for high-stakes tasks.

### Step 6 — Evaluate

Test on a golden set of research tasks (Section 15): 20-50 diverse questions with rubrics; measure accuracy, citation quality, completeness, efficiency; use LLM-as-judge plus human sampling. Evaluate *before* scaling — see `ai_llm/rag_optimization_techniques_guide.md` for the evaluation methodology pattern.

### Step 7 — Productionize

Caching (search results, extractions), rate-limit handling, cost controls, observability (traces per step), retry/failure handling, concurrency limits, and the human review workflow (draft → review → approve → publish). Details in Section 16.

## 14. Tools and Libraries

### 14.1 Agent Frameworks

| Tool | Type | Notes |
|------|------|-------|
| **LangChain / LangGraph** | Orchestration framework | LangGraph gives graph-based control flow, supervisor-worker patterns, persistence, human-in-the-loop interrupts; the most common production choice |
| **LlamaIndex** | Orchestration + RAG | Strong retrieval/agent abstractions, workflows, and document pipelines; good when research sits on top of a document corpus |
| **smolagents (HF)** | Lightweight agent framework | Code-first agents (tools as Python functions); powers HF's Open Deep Research; very transparent |
| **OpenAI Agents SDK** | Managed agent platform | Handoffs, guardrails, tracing; easy path to production on OpenAI models |
| **Claude Agent SDK** | Managed agent platform | Anthropic's agent toolkit; good tool-use ergonomics, hooks for review steps |
| **Manus** | Generalist agent platform | Runs research + execution asynchronously in the cloud; produces reports, spreadsheets, apps |
| **gpt-researcher** | Open-source research agent | Mature plan→search→scrape→write agent; many integrations; good starting point to fork |
| **STORM (Stanford)** | Open-source research system | Multi-perspective + parallel research + outline + article writing; the academic reference implementation |
| **Auto-Researcher** | Open-source framework | Single-turn, multi-turn, and team research modes; web search + optional scraping |
| **OpenDeepResearch** | Open-source deep research | OpenAI-ODR-style open implementation; see the catalog in `open_research_frameworks_guide.md` |

### 14.2 Search and Extraction Services

- **Perplexity Sonar** — LLM-backed search API; returns cited answers usable as source seeds.
- **Exa** — neural/embedding web search designed for AI agents; contents endpoint returns full-page text; native **MCP support** (below).
- **Tavily** — agent-oriented search API: clean results, content snippets, search depth, domain filters.
- **Serper / SerpAPI** — Google SERP APIs: standard Google results as JSON; cheap and familiar.
- **Firecrawl** — crawl + extract: turns any URL (including JS-heavy sites) into clean markdown; API and MCP.
- **Jina Reader** — URL → markdown via `r.jina.ai/<url>`; simple, no-code extraction.
- **Trafilatura** — local Python extraction library: HTML → clean text with high precision on news/academic pages.
- **Newspaper3k** — local article extraction (title, text, authors, dates) for news sources.

### 14.3 PDF and Document Parsing

- **PyMuPDF** — fast text + layout extraction from PDFs; the workhorse for filings and papers.
- **marker-pdf** — deep-learning PDF → markdown; handles complex layouts, tables, OCR-heavy scans (see the `ocr-and-documents` skill).
- **LlamaParse** — LLM-based document parser: messy PDFs, slides, tables → structured markdown; good for heterogeneous document sets.

### 14.4 MCP (Model Context Protocol)

MCP standardizes how agents connect to tools — a research agent's entire tool set can be MCP servers: a search server (Tavily/Exa/Firecrawl all ship MCP), an extraction server, a vector-store server, a database server. Benefits: the agent framework doesn't hard-code tool integrations; tools are swappable and auditable; the same research agent can run against different search backends by swapping servers. The MCP tooling landscape (server SDKs, client support in LangChain/Claude/OpenAI stacks, governance) is covered in `ai_llm/mcp_framework_tools_guide.md`. For banking deployments, MCP also centralizes access control — the server, not the agent, enforces who can query what.

---

## 15. Evaluation of Research Agents

### 15.1 Benchmarks

- **GAIA** (Meta/HF) — general AI assistant benchmark with research-heavy tasks (multi-step, tool-requiring, web/PDF-dependent); a standard proxy for research-agent capability.
- **DeepResearch Bench** (OpenAI, 2025) — tasks designed for deep research: expert-level prompts over diverse topics with grader-checked rubrics; measures whether agents produce expert-quality, well-sourced reports.
- **SimpleQA** — short factual questions with known answers; a hallucination/accuracy floor check (research agents should score near-perfect here).
- **FRAMES** — multi-hop fact retrieval over Wikipedia; tests the multi-step retrieval at the heart of research.
- **BrowseComp** (OpenAI) — browse-and-compare tasks where the answer requires navigating multiple pages and synthesizing.

Use these for *capability* checks; build your own golden set for *your* domain (below).

### 15.2 Evaluation Dimensions

| Dimension | What it measures | How |
|-----------|-----------------|-----|
| **Answer accuracy** | Factual correctness vs ground truth | Golden answers, fact-check sample |
| **Citation quality** | Do citations support claims? Are they real? | Manual + automated URL/quote checks |
| **Completeness** | Coverage of required aspects | Rubric per sub-question |
| **Efficiency** | Time, cost, tool calls per task | Logged traces |
| **Robustness** | Ambiguous queries, dead links, contradictory sources | Adversarial test set |

### 15.3 Evaluation Methodology

- **Golden task set**: 20-50 diverse research tasks representative of real usage, each with a rubric (dimensions, score bands). Keep it stable — it is the regression suite.
- **LLM-as-judge**: score reports against the rubric with a judge LLM (separate model/prompt from the researcher; judge on rubric items, not vibes). See `ai_llm/llm_evaluation_frameworks_guide.md` for judge design and bias controls.
- **Human review**: sample 10-20% of outputs for human expert grading — especially citation quality and factual accuracy, where LLM judges are weakest.
- **A/B testing**: compare agent versions on the same task set; statistical comparison of rubric scores.
- **Regression tracking**: run the golden set on every agent change (prompt, model, tool, architecture); track scores over time. The per-task quality metrics from Section 8.2 feed this dashboard.

Evaluation pitfalls: golden tasks that leak into the model's training data (rotate/refresh), rubrics that reward prose over evidence, and judging without the evidence ledger (the report alone can hide fabrication — grade report + ledger together).

### 15.4 A Sample Rubric

Rubrics make scores reproducible. A 5-point rubric per dimension keeps LLM-judge variance low:

| Dimension | 1 (fail) | 3 (pass) | 5 (excellent) |
|-----------|----------|----------|---------------|
| **Accuracy** | Multiple factual errors vs ground truth | No errors on checked claims | All claims verified correct, edge cases handled |
| **Citation quality** | Fabricated/vague citations | Real citations, mostly support claims | Primary sources, specific sections, every claim cited |
| **Completeness** | Misses >1 sub-question | All sub-questions addressed | Nuanced coverage incl. contradictions and edge cases |
| **Efficiency** | Budget blown, redundant searches | Within budget | Minimal tool calls for coverage achieved |
| **Robustness** | Fails on ambiguity/dead links | Handles with retries/fallbacks | Explicitly notes dead ends and adapts |

Score per task, average per dimension across the golden set, and track deltas on every agent change.

---

## 16. Production Considerations

### 16.1 Cost Control

Research agents are token-hungry — a deep-dive can consume millions of tokens. Controls:

- **Token budget per research task**: set a hard budget in the task schema; the planner allocates it across phases (typically extraction and synthesis dominate).
- **Cap tool calls per round**: bound search fetches and extraction calls; a page that yields nothing is wasted spend.
- **Model routing**: cheap/fast models for extraction and relevance filtering; powerful reasoning models for planning, synthesis, and review. See the model-router patterns in `ai_llm/hybrid_multi_agent_systems_guide.md`.
- **Caching**: cache search results and extractions by URL+date — deep research re-fetches aggressively; a shared cache across tasks eliminates repeat spend.
- **Rate limits**: search API quotas and polite crawling (respect robots.txt, backoff on 429s). Budget quota per task, not just cost.

### 16.2 Observability

Trace every step: the plan, each search (query, results, which were used), each extraction (source, claims), each synthesis call, each review note. Log to LangSmith/Langfuse or equivalent — the trace is simultaneously the debugger, the cost report, and the audit trail (Section 17). See `ai_llm/rag_frameworks_comparison_guide.md` for tracing platform comparison. A research agent without traces is un-debuggable — when a report is wrong, you must be able to replay *why*.

### 16.3 Reliability

- **Retry/failure handling**: dead links, API errors, timeouts are normal — retry with backoff, fall back to alternate extractors (e.g., Jina Reader if direct fetch fails), and mark unreachable sources in the log rather than silently dropping them.
- **Concurrency**: parallel searches with limits (e.g., 4-8 concurrent) — parallelism cuts latency but multiplies rate-limit pressure and cost; cap per task.
- **Output formats**: markdown (default), structured JSON (for downstream systems), slides (for executives). Keep the report and the ledger as separate artifacts.
- **Human review workflow**: draft → review → approve → publish. Encode the gates; block publication until sign-off for high-stakes tasks (Section 17).

---

## 17. Banking-Specific Applications

Research agents map directly onto high-value banking workloads — and banking is where their citation and audit-trail strengths matter most.

### 17.1 Financial Research Agents

- **Company/industry deep dives**: earnings releases, filings (10-K/20-F/AR), analyst reports, news — a research agent can assemble a cited profile (business, financials, risks, outlook) in minutes instead of analyst-hours. Every figure tied to its filing page.
- **Regulatory research**: MAS notices/guidelines, Basel texts, IFRS standards — track changes across versions, summarize impact on the bank's products and controls. See `financial_risk_compliance_systems_guide.md` for the compliance-systems context. This is a natural *watch* workload (continuous research, Section 18): a regulator publishes, the agent produces a change-impact brief with citations.
- **Due diligence support**: company research (ownership, structure, litigation), beneficial ownership resolution (entity matching across registries), sanctions screening (name/entity matching against OFAC/EU/UN lists), adverse media monitoring. Research agents handle the search+extract+ledger mechanics; human analysts own the judgment.
- **Market intelligence**: competitor moves, product launches, pricing changes, talent moves — recurring scans with fresh sources each cycle.
- **Trade finance research**: country risk profiles, sanctions exposure, compliance requirements per jurisdiction for a transaction or a corridor.
- **Investment research support**: macro reviews, sector outlooks, credit research (issuer, sector, covenant analysis) — draft-level research with full citation trails for the desk to refine.

### 17.2 Banking Considerations

| Concern | What it means in practice |
|---------|--------------------------|
| **Source reliability** | Prefer official/regulated sources — regulators, exchanges, audited filings, central banks — over blogs and aggregators. A research agent for banking should have a whitelist tier for authoritative domains. |
| **Citation mandatory** | Every claim sourced — this is compliance, not style. The evidence ledger is the deliverable. |
| **Audit trail** | Full research log: what was searched, read, used — for SR 11-7 model-risk reviews and MAS expectations. Traces must be retained, replayable, and attributable (which agent version, which model, which date). |
| **Data privacy** | Research on internal data (client info, deal data) — access control, PII handling under PDPA, no exfiltration to external search/LLM endpoints without approval. MCP servers as the access-control boundary (Section 14.4). |
| **Cost control** | Research agents are token-hungry — budget per task, cache aggressively, route models (16.1). |
| **Human review for high-stakes outputs** | Investment decisions, regulatory submissions — human sign-off is mandatory; the agent produces a *draft with provenance*, not a decision. |
| **Verification** | Cross-check critical claims manually (regulator text, filings) — the agent's citation is a pointer to check, not a substitute for checking. |
| **Freshness** | Regulatory/financial data changes fast — research date matters; a MAS notice from January may be superseded by March. Date-stamp every output. |
| **Model governance** | Document the research agent as a model component (SR 11-7): model inventory, validation, versioning, change management, monitoring. An agent is a model *system* — prompts, tools, and architecture all fall under governance. |

### 17.3 Worked Example: A Regulatory Change-Impact Brief

Scenario: MAS publishes a consultation paper amending AML/CFT requirements for cross-border payment service providers. A bank's compliance function needs a same-day impact brief.

**Task schema**: question — "What changes if this consultation becomes final, and what must our Singapore entity do?"; depth — deep-dive; output — 2-page brief + change table; budget — tight (same-day).

**Agent run**:

1. **Plan**: sub-questions — (a) what does the consultation propose (requirements, scope, timeline)? (b) how does it differ from the current Notice/Guidelines? (c) which of our products/processes are affected? (d) what implementation effort is implied? Sources: MAS.gov.sg (primary), industry commentary (secondary), internal policy inventory (vector search).
2. **Search/read**: fetch the consultation paper PDF + current Notice from MAS.gov.sg; extract per-paragraph change candidates into the ledger with exact quotes and section references; search industry responses and law-firm client alerts for interpretive angles.
3. **Extract**: for each proposed change — requirement text, current position, affected product/process mapping, effective date proposal, quote + URL + section.
4. **Synthesize**: change-impact table (requirement → current state → proposed state → impact on our entity → effort estimate) with citations per row; note interpretive disagreements between the paper's text and industry commentary where they exist.
5. **Quality**: reviewer agent checks that every row is grounded (ledger join); compliance officer reviews the change table and signs off; the brief carries the research date prominently since the consultation may be superseded before finalization.
6. **Audit**: the research log (queries, sources read, ledger) is archived with the brief — ready for the SR 11-7 / internal audit file.

This pattern — regulator publishes, agent produces a cited change brief, human approves — is the single most reusable banking research-agent workflow, and it composes with the continuous-research future direction (Section 18).

---

## 18. The Future: Research Agents 2026+

- **Deeper reasoning integration**: research + reasoning models — the agent verifies claims with reasoning (e.g., checking internal consistency, arithmetic on extracted numbers, cross-source logical coherence) rather than only retrieving. Test-time compute shifts from "more searches" to "more verification."
- **Multimodal research**: images, charts, videos, audio become first-class sources — extracting from earnings-call audio, chart data in PDFs, video briefings. Vision-language extraction feeds the same evidence ledger.
- **Continuous research (watch agents)**: agents that monitor topics over time — a regulator's site, news feeds, competitor pages — and alert on changes, producing delta briefings instead of one-shot reports. Natural fit for regulatory monitoring (Section 17).
- **Personalized research**: agents learn the user's preferences — depth, style, favored sources, template structure — and adapt plans and outputs per user.
- **Collaborative research**: multi-agent research teams with specialized roles (planner, searcher, extractor, fact-checker, writer, reviewer) coordinating on shared ledgers — the architecture patterns of Section 9 generalize into full agent teams.
- **Regulation of research agents**: disclosure of AI-generated research (labeling), accountability (who owns the output), and quality standards for AI research in regulated industries — banks will need to disclose and govern agent-produced research like any other model output (Section 17, SR 11-7).
- **Human-agent division of labor stabilizes**: the pattern of "agent produces draft with full provenance; human owns judgment and sign-off" is becoming the regulated-industry default — research agents are adopted not as replacements for analysts but as force multipliers on the search-read-extract-cite mechanics, freeing humans for judgment, challenge, and decision.
- **Open-source parity accelerates**: the gap between frontier deep-research products and open implementations (LangChain ODR, smolagents, STORM, OpenDeepResearch, O-Researcher) is closing within months of each release — see `open_research_frameworks_guide.md` — making self-hosted, auditable research agents viable for banks that cannot send questions to external platforms.

---

## 19. Conclusion and Decision Guidance

### 19.1 When to Use a Research Agent — and When Not

**Use a research agent when:**

- The topic is complex and multi-source — an assessment, not a fact (Section 11 routing).
- Citations and grounding are required — any output that will be relied upon, reviewed, or audited.
- Research needs recur — same-shaped questions at scale (market scans, regulatory trackers, DD templates); the agent amortizes its setup cost.
- Scale exceeds human capacity — hundreds of sources, many sub-questions, tight deadlines.

**Do not use a research agent when:**

- It's a simple fact — just search; an agent adds minutes and dollars for nothing.
- The topic is highly sensitive — human-only research with no agent involvement in drafting (control-room, M&A, legal-privilege contexts).
- It's niche with no findable sources — an agent cannot synthesize what does not exist; a human domain expert may know where the unpublished truth lives.
- The environment can't support it — no reliable search/extract access, no budget, no audit capability (Section 16).

### 19.2 Adoption Path

1. **Start with a tool-augmented single agent** (Section 9) — one agent, the full loop, an evidence ledger, and a reviewer pass. This delivers most of the value with the least complexity.
2. **Add the evidence ledger and citation pipeline first** — they are non-negotiable for banking-grade outputs; everything else can be upgraded later.
3. **Add a reviewer agent** — the cheapest quality multiplier: fresh-context critique catches what the researcher's sunk cost hides.
4. **Add parallelism and supervisor patterns as complexity grows** — parallel searches, then supervisor-worker for many sub-questions, then researcher-critic and merge patterns for high-stakes work.
5. **Evaluate continuously** — a stable golden task set (Section 15) tracked over every change; quality metrics (8.2) on every task.

The discipline to internalize: **research agents are judged by their citations, their audit trails, and their limitations sections — not by their prose.** Build the loop, wire the ledger, gate the output, and let the agent's work be checkable claim by claim.

---

*Series: LLM/AI Technical Guides — Research Agents. Related guides: `open_research_frameworks_guide.md` (framework catalog), `ai_llm/hierarchical_multi_agent_frameworks_guide.md`, `ai_llm/hybrid_multi_agent_systems_guide.md`, `ai_llm/advanced_rag_techniques_guide.md`, `ai_llm/rag_vs_long_context_llms_guide.md`, `ai_llm/query_rewriting_rag_guide.md`, `ai_llm/rag_optimization_techniques_guide.md`, `ai_llm/rag_frameworks_comparison_guide.md`, `ai_llm/llm_evaluation_frameworks_guide.md`, `constrained_decoding_frameworks_guide.md`, `llm_development_risks_security_guide.md`, `financial_risk_compliance_systems_guide.md`, `ai_llm/mcp_framework_tools_guide.md`, `ai_llm/beyond_rag_guide.md`.*


