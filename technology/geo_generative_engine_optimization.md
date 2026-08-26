# Generative Engine Optimization (GEO): The Complete Guide

> **Author:** Jack Liu Shurui | **Last Updated:** July 2026 | **Version:** 1.0
> **Maintained at:** https://github.com/jackliusr/research

---

## Table of Contents

1. [What Is GEO and Why It Matters](#1-what-is-geo-and-why-it-matters)
2. [The Shift from 10 Blue Links to AI-Generated Answers](#2-the-shift-from-10-blue-links-to-ai-generated-answers)
3. [How Generative Engines Work](#3-how-generative-engines-work)
4. [GEO vs SEO: Comparison Table](#4-geo-vs-seo-comparison-table)
5. [How Different AI Engines Rank Sources](#5-how-different-ai-engines-rank-sources)
6. [Core GEO Optimization Strategies](#6-core-geo-optimization-strategies)
7. [GEO Content Formats That Win Citations](#7-geo-content-formats-that-win-citations)
8. [Measuring GEO Success](#8-measuring-geo-success)
9. [GEO Tools Landscape](#9-geo-tools-landscape)
10. [GEO for Different Content Types](#10-geo-for-different-content-types)
11. [GEO Pitfalls and Risks](#11-geo-pitfalls-and-risks)
12. [Future Trends in GEO](#12-future-trends-in-geo)
13. [GEO Content Checklist](#13-geo-content-checklist)
14. [References](#14-references)

---

## 1. What Is GEO and Why It Matters

Generative Engine Optimization (GEO) is the practice of optimizing content to be discovered, retrieved, cited, and recommended by AI-powered answer engines — ChatGPT, Perplexity, Google AI Overviews, Bing Copilot, Claude, Gemini, Grok, and DeepSeek. Unlike SEO (optimizing for a ranked list of blue links), GEO optimizes for being named, quoted, or cited inside an AI-generated answer. The unit of success shifts from a ranking position to a citation.

The term was formalized in "GEO: Generative Engine Optimization" (Aggarwal et al., 2024, KDD '24) by Princeton, Georgia Tech, and IIT Delhi researchers. They tested nine optimization strategies across 10,000 queries and found visibility lifts up to 40%.

**Why GEO matters now:**
- AI Overviews appear in ~47% of informational U.S. queries (Semrush, March 2026). Zero-click answers drain 30-60% of organic traffic.
- Only ~12-16% of URLs cited by AI engines rank in Google's top 10 (BrightEdge, 2026). Strong SEO does not guarantee AI visibility.
- ChatGPT alone has 300M+ weekly active users. Citation is the new ranking.
- Sources cited by AI engines experience 14-day citation volatility — continuous monitoring is essential.
- Overlap between engines is as low as 16-23% — each requires distinct optimization.

## 2. The Shift from 10 Blue Links to AI-Generated Answers

**The old model:** User queries → ranked blue links → user clicks through. SEO optimized for position. Traffic followed rankings.

**The new model:** User queries → AI synthesizes an answer from 3-12 sources → user gets the answer inline → click may never happen.

Key dynamics of the new paradigm:
- **Answer-first interfaces** present generated responses above links.
- **Multi-source synthesis** weaves information from multiple domains into coherent paragraphs.
- **Conversational follow-ups** let users drill deeper without visiting source pages.
- **Attribution varies** — Perplexity provides explicit inline citations; ChatGPT cites when web search is enabled; Claude and Gemini cite less consistently.

**Market snapshot (2026):** Google still commands ~85% of traditional search, but the zero-click AI answer layer grows rapidly. GEO addresses visibility in that layer — a market where traditional SEO offers diminishing returns.

## 3. How Generative Engines Work

### The RAG Pipeline

Most generative engines use Retrieval-Augmented Generation (RAG), a three-stage pipeline:

**Stage 1 — Retrieval:** The system retrieves candidate documents via vector search (semantic embeddings), hybrid keyword+semantic retrieval (BM25 + dense vectors), or live web search. Key AI crawlers: GPTBot / OAI-SearchBot (OpenAI), Google-Extended (Google), Claude-Web (Anthropic), PerplexityBot (Perplexity), CCBot (Common Crawl). Blocking these in robots.txt makes you invisible to that engine.

**Stage 2 — Re-ranking:** Retrieved candidates are re-ranked by: semantic relevance to the query, authority signals (domain reputation, link profile, citation frequency), freshness (recency of content), structured data presence, factual density (verifiable claims per paragraph), and information gain (does this source add unique value vs. repeating others).

**Stage 3 — Generation + Citation:** The LLM synthesizes an answer, deciding which sources to cite inline. Engines prefer sources with explicit, extractable claims (statistics, quotes, list items, table cells). Earlier-cited sources get higher weight. Models may hallucinate or misattribute — this is a known GEO risk.

### Freshness Windows by Engine

| Engine | Freshness Window | Updates |
|--------|-----------------|---------|
| Google AI Overviews | Near-real-time | Live Google Search index |
| Perplexity | ~30-day crawl | Regular re-crawl + freshness boost |
| ChatGPT (with search) | Real-time | Live web search per query |
| ChatGPT (base) | At model training | New model releases only |
| Bing Copilot | Near-real-time | Bing search index |
| Claude | Training cut-off | Periodic model updates |
| Gemini | Near-real-time | Google Search integration |

## 4. GEO vs SEO: Comparison Table

| Dimension | Traditional SEO | GEO |
|-----------|----------------|-----|
| **Goal** | Rank #1 in organic SERP | Get cited in AI-generated answer |
| **Unit of success** | SERP position (1-10) | Citation / brand mention |
| **Primary audience** | Google ranking algorithm | LLM + RAG pipeline |
| **Content target** | Keywords, backlinks, meta tags | Facts, quotes, statistics, entities |
| **Traffic model** | Click-through from SERP | Zero-click referral or direct brand exposure |
| **Optimization surface** | Your website | Your site + 3rd-party citations + entity ecosystem |
| **Key signals** | Links, keywords, UX, E-E-A-T | Semantic relevance, fact density, citation frequency, authority |
| **Measurement** | Rankings, organic traffic, CTR | Citation rate, share of voice, AI visibility score |
| **Competition** | Domains targeting same keywords | All sources AI could cite for a topic |
| **Update cadence** | Continuous crawl | Days to months, engine-dependent |
| **Overlap** | ~85% of searches go through Google | Only 12-16% overlap with Google top 10 |

**Where they overlap:** Technical SEO (speed, crawlability, structured data), content quality / E-E-A-T, backlinks (authority signals), schema markup.

**Where they diverge:** Keywords vs. entities; link building vs. citation building; click-through vs. zero-click; keyword density vs. fact density.

## 5. How Different AI Engines Rank Sources

Each generative engine retrieves and cites sources differently. Citation overlap between engines can be as low as 16-23%.

### ChatGPT (Web Search)
- **Search backbone:** Bing + OpenAI crawler (GPTBot, OAI-SearchBot)
- **Citation style:** Numbered inline citations with sources at bottom when web search enabled
- **Preferred sources:** Brand websites (44% of citations), news/media (25%), Wikipedia, academic
- **Format preference:** Listicles (25.37%), comparison tables, data analysis, FAQ, how-to guides
- **Freshness:** High for temporal queries; real-time web search
- **Volatility:** 14-day citation churn — sources appear/disappear with model updates
- **Overlap with AI Overviews:** ~23%

### Perplexity
- **Search backbone:** Own crawl + multiple APIs; PerplexityBot
- **Citation style:** Explicit numbered inline citations on specific claims — highest transparency
- **Preferred sources:** News media, academic papers, authoritative blogs, Wikipedia, docs
- **Format preference:** Direct answers, data-backed claims, research reports, comparisons
- **Freshness:** ~30-day crawl with freshness boost
- **Re-ranking:** Multiple layers (relevance, authority, novelty)
- **Strategy:** Harder for niche brand content to break in; favors established media

### Google AI Overviews
- **Search backbone:** Google's web index (largest globally)
- **Citation style:** Inline links in generated text, source card carousel
- **Preferred sources:** E-E-A-T strong sites, .gov, .edu, health authorities
- **Format preference:** Structured content, clear headings, lists, tables, schema
- **Freshness:** Near-real-time via Google's live index
- **Unique behavior:** AI Overviews and AI Mode (both Gemini 3) cite different sources — URL overlap ~10.7%
- **Impact:** Can cut organic click-through 30-60% for informational queries

### Bing Copilot
- **Search backbone:** Bing index + OpenAI models
- **Preferred sources:** Microsoft-ecosystem content, authoritative publications
- **User base:** ~100M DAU through Edge sidebar and Copilot key
- **Citation style:** Inline citations with footnotes

### Claude (Anthropic)
- **Search:** No native web search; limited API tool use. Relies on training data.
- **Citation style:** Minimal inline citations; conversational paraphrasing
- **Strategy:** Optimize for inclusion in high-quality training corpora (Wikipedia, academic datasets, widely-cited publications)

### Gemini (Google)
- **Search backbone:** Google Search integration (near-real-time)
- **Preferred sources:** Google-indexed web, YouTube, Google Business Profiles
- **Unique:** Can cite Google Business Profile data for local results

**Cross-engine strategy:** Ensure crawlability (don't block AI crawlers). Build broad authority (Wikipedia, academic, .gov/.edu). Structure everything (schema, headings, tables). Monitor per engine. Prioritize by audience demographic.

## 6. Core GEO Optimization Strategies

Based on the Princeton GEO paper (KDD 2024) and industry validation (2025-2026), ranked by impact:

### 6.1 Citation Optimization (Highest Impact)
Including authoritative citations boosted visibility by over 40% in the Princeton study.
- Cite .gov, .edu, academic papers, and major publications
- Include internal citations to your own research
- Format citations cleanly (numbered references, hyperlinked text)
- **Why it works:** RAG re-rankers score documents higher when they reference authoritative sources

### 6.2 Quote-Worthy Snippets
- Write short, standalone sentences expressing definitive claims
- Use blockquote formatting for key insights
- Attribute quotes to named experts with credentials
- Include contrarian or novel takes for information gain

### 6.3 Statistical and Data-Backed Claims
- Use specific numbers ("74% of enterprises" not "most enterprises")
- Attribute every statistic to its source
- Original research (surveys, proprietary data) is gold — highest information gain
- **Why it works:** Factual density — verifiable claims per paragraph — correlates strongly with citation probability

### 6.4 Entity-Based Content
- Define entities explicitly on first mention
- Use consistent entity names (don't vary terminology)
- Implement Thing, Organization, Person, Product schema markup
- **Why it works:** RAG retrieval is semantic — well-defined entities increase retrieval probability

### 6.5 Authoritative Sourcing and E-E-A-T
- Every article needs a named author with credentials and bio
- Content on your own domain > guest posts or syndication
- Build topical authority clusters (multiple pieces on same topic)
- Earn external citations from authoritative publications (creates citation feedback loop)
- Maintain freshness — add "Last updated" dates, review annually

### 6.6 FAQ and Q&A Structures
- Add FAQ sections with FAQPage schema markup
- Frame headings as questions ("What is GEO?" not "Introduction")
- Provide answer immediately below the question
- Map directly to how users phrase conversational queries

### 6.7 Structured Data and Markdown Formatting
- Article, FAQPage, HowTo, Product, Organization, Person schemas
- Clean heading hierarchy (H1 > H2 > H3, no skips)
- Bullet points, numbered lists, and data tables for easy RAG extraction
- Write in Markdown-compatible structures — AI models parse clean hierarchy better than nested divs
- Use `<blockquote>` and `<cite>` semantically

### 6.8 Freshness Signals
- Visible "Last updated" dates on every page
- dateModified schema markup
- Content updates timed to AI crawl schedules
- Publish new data periodically — updated statistics beat static content

### 6.9 Information Gain and Novelty
- Do not rephrase the top-ranking page. AI de-duplicates — same info = zero gain.
- Add unique data, proprietary research, or expert interviews
- Present novel frameworks, taxonomies, or perspectives
- Synthesis of multiple sources into new insight > summary

## 7. GEO Content Formats That Win Citations

### 7.1 Listicles with Citations (Highest Citation Rate)
- **ChatGPT citation rate:** 25.37% of all citations go to listicle formats
- Each list item should be a self-contained extractable claim with a statistic
- Use numbered lists (not bullets) for easier AI extraction
- Cite authoritative sources within each item

### 7.2 Authoritative How-To Guides
- Use HowTo schema markup
- Numbered steps with clear outcomes
- Include time, tools, and difficulty level
- Add TL;DR summary at top

### 7.3 Statistical Reports and Original Research
- Open with key statistics summary (these become quotable snippets)
- Use data tables (AI-extractable)
- Include methodology section (builds trust for re-rankers)
- Update regularly for freshness signals
- **Why it works:** Original data = information gain

### 7.4 Q&A and FAQ Content
- FAQPage schema, one question per H2, answer in 1-3 paragraphs
- Include a data point or citation per answer
- Best for informational queries and conceptual explanations

### 7.5 Wikipedia-Style Structured Articles
- Concise definition in first paragraph
- Infobox-style structured data, table of contents, "See also" sections
- Every claim cited with numbered reference
- **Why it works:** Wikipedia is the single most-cited source across all engines. Structure like Wikipedia = disproportionate citation rate.

### 7.6 Comparison Tables
- Table schema markup; summary sentence above and below table
- Each cell backed by citation or data point
- Best for product comparisons, framework comparisons, decision guides

### 7.7 Expert Roundups
- Each quote attributed to named expert with credentials and bio
- Include expert headshot alt text
- Link to expert LinkedIn / profile pages

## 8. Measuring GEO Success

### 8.1 Core Metrics

**Citation Rate:** % of relevant AI-generated queries where your brand is cited. Formula: (Queries citing brand / total tracked queries) × 100. Benchmark: Median B2B SaaS ~6.2% (Profound Q1 2025). Target: 15-25% for competitive categories; 40%+ for established brands.

**AI Share of Voice (AI-SOV):** Your brand's citation frequency vs. competitors. Formula: (Your citations / total category citations) × 100. Target: 25-40%.

**AI Visibility Score:** Composite (vendor-dependent) combining citation rate, position, and sentiment. Profound/Otterly/BrightEdge each have proprietary 0-100 scales.

**Position Within Answer:** First mention (highest value) > middle > list item > marginal citation.

**Citation Sentiment:** Positive (recommended) > neutral > negative (criticized). Tracked by Profound, Georion.

**Source Link Inclusion:** Whether the AI response includes a clickable link. Perplexity always includes links; ChatGPT with search includes links; Claude rarely does.

**AI Referral Traffic:** Traffic from AI platform clicks. Use UTM parameters — ChatGPT traffic often appears as direct/unknown in analytics without them.

### 8.2 Monitoring Cadence

| Frequency | Activity |
|-----------|----------|
| Daily | Automated citation scans on 3-5 key queries |
| Weekly | Brand mention report across all engines |
| Bi-weekly | Competitor citation benchmarking |
| Monthly | Share of voice and sentiment analysis |
| Quarterly | Full GEO audit: strategies, engines, content gaps |

### 8.3 Baseline Setup
1. Define 20-50 target queries (informational, commercial, branded)
2. Run each query across ChatGPT, Perplexity, Gemini, AI Overviews
3. Record: cited (Y/N), position, sentiment, competitors, link included
4. Calculate baseline citation rate, AI-SOV, visibility score
5. Track weekly — expect 14-day+ citation volatility

## 9. GEO Tools Landscape

### Dedicated GEO Analytics

| Tool | Primary Function | Price Range |
|------|----------------|-------------|
| **Profound** | AI citation tracking across 5+ engines; 0-100 visibility score | $200-500/mo |
| **OtterlyAI** | Multi-engine visibility monitoring; citation studies | $150-400/mo |
| **Georion** | 7+ engine tracking; proprietary GEO scoring; competitor dashboards | $200-600/mo |
| **BrightEdge** | Enterprise SEO + GEO Visibility Score | $500-2,000+/mo |
| **Peec AI** | Cross-engine tracking; content gap analysis | $100-300/mo |
| **Athena AI** | AI search visibility; keyword-to-citation mapping | Custom |
| **GEOReport AI** | Automated GEO audit reports | $100-500/mo |

### SEO Platforms Adding GEO
- **Semrush** — AI Overviews tracker, citation mentions (rolling out)
- **Ahrefs** — Limited AI citation data, web crawler updates
- **SE Ranking** — AI visibility monitoring module
- **Similarweb** — AI traffic attribution estimates
- **Conductor** — Enterprise AI search visibility tracking

### Free / Low-Cost
- Manual prompt testing (free, labor-intensive)
- Google Search Console (limited AI Overviews click data)
- OtterlyAI / Georion free tiers
- Custom Python scripts using OpenAI and Perplexity APIs

### Selection Criteria
Engine coverage, citation depth (which claim was cited vs. just brand mention), competitor tracking, sentiment analysis, integration with existing SEO stack, update frequency.

## 10. GEO for Different Content Types

### 10.1 B2B (SaaS, Enterprise)
- **Goal:** Cited for industry-specific queries, solution comparisons, thought leadership
- **Best formats:** Original research reports, benchmarks, whitepapers, comparison guides, expert roundups
- **Key strategies:** Publish proprietary survey data; maintain Wikipedia pages; get cited in analyst reports (Gartner, Forrester) which trickle into AI; build topical authority clusters
- **Benchmark:** Median B2B SaaS citation rate ~6.2% (Profound Q1 2025)
- **Metrics:** Citation rate in industry queries, AI-SOV vs. competitors

### 10.2 Local Business
- **Goal:** Appear for "near me" and city-specific queries
- **Best formats:** Google Business Profile, local guides, neighborhood content
- **Key strategies:** Optimize GBP (cited by Gemini/AI Overviews for local); LocalBusiness schema; local directory citations; customer reviews (AI models factor review count/rating)
- **Challenge:** Local content has lower information gain — differentiate with unique local data

### 10.3 E-Commerce
- **Goal:** Products cited in "best X" queries, comparisons, shopping recommendations
- **Best formats:** Product comparisons, buying guides, "best of" lists with pricing, product Q&A
- **Key strategies:** Product schema (price, availability, reviews, specs); "vs." comparison content; UGC Q&A on product pages; review snippets
- **Challenge:** AI overviews cannibalize e-commerce traffic by answering "best X" without linking

### 10.4 News and Publishing
- **Goal:** Cited as primary source for current events
- **Best formats:** Breaking news (timeliness critical), data journalism, investigative reports, explainers
- **Key strategies:** Publish fast (freshness window is hours for news); NewsArticle schema; original reporting (primary sources preferred); authoritative bylines; get cited by other outlets
- **Challenge:** 30-60% traffic loss on informational news queries from AI overviews

## 11. GEO Pitfalls and Risks

### 11.1 AI Hallucinations and Misattribution
**Risk:** AI fabricates quotes, misattributes claims, cites your content inaccurately.
**Mitigation:** Monitor citation accuracy weekly. Use literal, unambiguous language. Contact platform feedback channels for false attributions (limited recourse). Some brands add "AI citation policy" pages with canonical versions of key claims.

### 11.2 Over-Optimization
**Risk:** Content aggressively optimized for AI extraction reads like robot text, harming engagement and trust, and may trigger anti-spam filters.
**Signals:** Every sentence is a quotable stat; unnatural entity name repetition; content structured only for extraction, not readability; excessive schema markup.
**Mitigation:** Write for humans first, AI extraction second. Follow 80/20 rule: 80% human-readable content, 20% AI-optimized structure.

### 11.3 Brand Safety in AI Context
**Risk:** AI cites your brand alongside controversial topics, misinformation, or low-quality content.
**Mitigation:** Monitor AI brand mentions for negative sentiment/context. Define brand-safe AI visibility policies. Consider excluding sensitive-topic pages from AI crawlers via robots.txt.

### 11.4 Content Cannibalization (Zero-Click)
**Risk:** AI summarizes your best content in its answer, satisfying the user without a click. Your top-ranking content drives brand awareness but zero referral traffic.
**Mitigation:** Balance ~60% citation-worthy content with ~40% click-optimized commercial content (Georion). Use UTM tracking. Build conversion paths within cited content (CTAs in data reports, gated research). Monitor assisted conversions.

### 11.5 Crawler Blocking
**Risk:** Blocking AI crawlers makes you invisible to those engines.
**Common causes:** robots.txt disallows GPTBot / Google-Extended / PerplexityBot / Claude-Web; Cloudflare security rules block AI crawlers; paywalls without exceptions.
**Mitigation:** Audit robots.txt monthly. Allow known AI crawlers while protecting sensitive content. Test each engine for your content.

### 11.6 Data Privacy and Compliance
**Risk:** GDPR, CCPA impact AI crawling; PII in content can be cited by AI.
**Mitigation:** Consult legal counsel before aggressive GEO in regulated industries (healthcare, finance, legal). Consider "not for AI training" terms of service clauses (limited enforceability).

## 12. Future Trends in GEO

### 12.1 Multi-Modal GEO
AI engines increasingly cite images, videos, and audio. GEO will expand to optimizing visual assets with schema markup (ImageObject, VideoObject), accessible captions, transcripts, and data tables embedded in chart alt text.

### 12.2 Personalized AI Answers
Models move toward personalized responses based on user history, location, and context. ChatGPT may cite different sources for beginners vs. experts. Gemini cites local sources differently by location. **GEO action:** Build content for multiple persona levels and local variants.

### 12.3 Agent-Sourced Content
AI agents (AutoGPT, Claude agents, ChatGPT tasks) autonomously browse and synthesize information. Agents favor unambiguous structure, clean data, and machine-readable formats. **GEO action:** Publish JSON-LD data feeds, structured APIs, and content designed for autonomous consumption.

### 12.4 Real-Time Data Optimization
Freshness windows shrink to hours. Live data feeds (stock prices, scores, availability) and first-to-publish breaking news get cited fastest. **GEO action:** Implement last-modified signals, automated content refreshes, and scheduled updates synced to AI crawl patterns.

### 12.5 GEO for Voice and Conversational AI
Voice assistants (Siri, Alexa, Google Assistant) increasingly use generative AI. Voice-optimized content uses concise, declarative, single-claim sentences with the answer front-loaded. **GEO action:** Lead with the conclusion in short, declarative sentences.

### 12.6 Standardized GEO Metrics
The industry is converging on standardized KPIs (ISOM consortium expected late 2026-2027). Expect GEO-specific crawling APIs from engines, formal certification programs, and integration into major analytics platforms.

### 12.7 Regulatory Oversight
EU AI Act compliance will require citation transparency. Publishers may gain rights to opt out of AI training/citation. Attribution requirements in AI-generated answers are likely. **GEO action:** Stay informed via IAB, DMCA, and press association groups.

## 13. GEO Content Checklist

### Pre-Publishing

**Foundation**
- [ ] Content addresses a specific, research-backed user query (not just a keyword)
- [ ] Topic has sufficient AI engine search volume (verified via GEO tool)
- [ ] Content provides information gain — does not merely rephrase existing sources
- [ ] Target query identified for citation tracking; baseline established
- [ ] Content format selected for AI extractability (listicle, guide, report, FAQ, comparison)

**Structure**
- [ ] Title clearly states the topic
- [ ] First paragraph contains a direct, concise answer to the core query
- [ ] H2 headings mirror natural language questions ("What is X?", "How does X work?")
- [ ] Clean heading hierarchy (H1 > H2 > H3, no skipped levels)
- [ ] Bullet points and numbered lists for extractable claims
- [ ] Data tables for comparative or statistical information
- [ ] FAQ section included (minimum 3-5 Q&A pairs)
- [ ] Table of contents for long-form content
- [ ] TL;DR or executive summary at top

**Citation and Authority**
- [ ] At least 3 authoritative external citations (.gov, .edu, academic, industry reports)
- [ ] Every statistic attributed to its source
- [ ] Named expert quotes with credentials and attribution
- [ ] Internal links to related authoritative content on your domain
- [ ] Author byline present with credentials and bio
- [ ] "Last updated" date visible and accurate
- [ ] Content reviewed by subject matter expert (if applicable)

**Data and Statistics**
- [ ] Key statistics highlighted (bolded or pulled into separate paragraph)
- [ ] Statistics from verifiable, citable sources
- [ ] Original data or proprietary research included where possible
- [ ] Data visualizations have alt text and accessible data tables

**Technical SEO / Structured Data**
- [ ] Article schema markup implemented (Article, NewsArticle, BlogPosting)
- [ ] FAQPage schema (if FAQ section exists)
- [ ] HowTo schema (if guide/tutorial content)
- [ ] Organization and Person schema on author bio
- [ ] BreadcrumbList schema implemented
- [ ] datePublished and dateModified schema properties set
- [ ] Robots.txt allows GPTBot, Google-Extended, PerplexityBot, Claude-Web
- [ ] Page loads in under 3 seconds; mobile-responsive
- [ ] Clean URL structure with descriptive slugs

**Entity Optimization**
- [ ] Primary entity clearly defined in first paragraph
- [ ] Consistent entity naming throughout
- [ ] Related entities linked (internal or authoritative external definitions)
- [ ] Named entities identified with schema

**Freshness**
- [ ] Content includes recent data/statistics (within 12 months)
- [ ] "Last reviewed" date shown
- [ ] Content scheduled for regular review (quarterly for evergreen)
- [ ] News/temporal content published within 24-48 hours of event

### Post-Publishing

- [ ] Baseline citation check performed (query across ChatGPT, Perplexity, Gemini, AI Overviews)
- [ ] Week 1 citation rate recorded
- [ ] Week 2 citation volatility check
- [ ] Competitor citation analysis (who is cited instead of you?)
- [ ] AI referral traffic monitored (UTM on cited URLs)
- [ ] Citation sentiment reviewed
- [ ] Broken external links identified and fixed
- [ ] Content updated if citation rate not improved by Week 4

### Ongoing Maintenance

- [ ] Weekly citation rate monitoring (20-50 key queries)
- [ ] Bi-weekly competitor GEO benchmarking
- [ ] Monthly content freshness audit
- [ ] Monthly robots.txt audit (AI crawlers not blocked)
- [ ] Quarterly full GEO audit (all strategies, all engines)
- [ ] Content pruning: consolidate/redirect pages competing for same citations
- [ ] New content gap analysis (queries where competitors are cited but not you)
- [ ] AI platform changes monitored (new engines, citation behavior changes)
- [ ] GEO tools evaluated for relevance and accuracy
- [ ] Brand safety review (content being cited in undesirable contexts?)

## 14. References

1. Aggarwal, P., Murahari, V., et al. (2024). "GEO: Generative Engine Optimization." *KDD '24*, Barcelona. arXiv:2311.09735.
2. Lewis, P., et al. (2020). "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." *NeurIPS 2020*.
3. Semrush. (2026). "State of Search 2026" — AI Overviews in ~47% of U.S. informational queries.
4. BrightEdge. (2026). "AI Citation Source Analysis" — ~12-16% overlap AI-cited URLs vs. Google top 10.
5. Profound. (2025). "Q1 2025 GEO Benchmark Dataset" — Median B2B SaaS citation rate ~6.2%.
6. OtterlyAI. (2025-2026). "AI Citation Studies" — 44% ChatGPT citations from brand websites.
7. Georion. (2026). "Cross-Engine Citation Analysis" — ~23% overlap ChatGPT vs. AI Overviews.
8. Google. "AI Overviews in Search." https://support.google.com/websearch
9. OpenAI. "ChatGPT Web Search." https://openai.com/chatgpt
10. Perplexity. "How Perplexity Works." https://perplexity.ai
11. Anthropic. "Claude Web Crawling." https://anthropic.com
12. Microsoft. "Copilot in Bing." https://copilot.microsoft.com

---

*This guide is maintained at https://github.com/jackliusr/research. Contributions welcome. Last updated: July 2026.*
