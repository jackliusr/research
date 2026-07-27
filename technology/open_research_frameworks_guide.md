# Open Research Frameworks: A Comprehensive Guide

> **A deep dive into the emerging ecosystem of open-source frameworks and platforms for autonomous and semi-autonomous AI-driven research — covering tools for deep research, literature analysis, scientific discovery, and knowledge synthesis.**

**Author:** Jack Liu Shurui — **Series:** LLM/AI Technical Guides — **Date:** July 2026

---

## 1. What Are Open Research Frameworks?

### 1.1 Definition

Open research frameworks are publicly available software frameworks, platforms, and toolkits that enable AI agents to autonomously or semi-autonomously conduct research tasks — literature review, hypothesis generation, experimental design, data analysis, paper writing, and knowledge synthesis. Unlike closed/commercial tools (OpenAI Deep Research, Perplexity Deep Research, Google Deep Research), open frameworks are publicly available (MIT/Apache licensed), customizable (swap models, tools, agents, prompts), self-hostable (run locally with full data privacy), community-driven, and auditable (every decision traceable).

### 1.2 Contrast with Closed Research Tools

| Dimension | Closed/Commercial | Open Research Frameworks |
|-----------|------------------|--------------------------|
| **Examples** | OpenAI Deep Research, Perplexity DR, Google DR | ROMA, LangChain ODR, HF smolagents, OpenManus, DeepSearcher |
| **Source code** | Proprietary, hidden behind APIs | Public on GitHub, permissive licenses |
| **Customization** | Limited to UI/API parameters | Full control over every component |
| **Data privacy** | Processed on vendor servers | Run 100% locally on your infrastructure |
| **Cost model** | Per-query or subscription fees | Free software; pay only for LLM/compute |
| **Search backends** | Vendor-chosen, opaque, non-swappable | Swappable (DuckDuckGo, Tavily, Bing, custom) |
| **Execution visibility** | Black box — no insight into reasoning | Full traces, logs, generated code visible |
| **Extensibility** | API-only (add endpoints, not internals) | Add tools, agents, models, MCP servers |

### 1.3 Why They Matter

Open research frameworks democratize AI-powered research. Previously only well-funded organizations could leverage AI agents for deep, multi-step research. Now any researcher, student, or enterprise can deploy a research agent, customize it to their domain, verify every step, and maintain data sovereignty. This mirrors earlier open-source movements — Linux (OS), React/Django (web frameworks), PyTorch (ML) — each accelerated innovation by an order of magnitude once barriers fell.

### 1.4 Timeline

- **Feb 2025:** Hugging Face builds Open Deep Research in 24 hours using smolagents
- **Feb 2025:** LangChain releases Open Deep Research (Supervisor-Researcher on LangGraph)
- **Mar 2025:** MetaGPT team launches OpenManus; Zilliz releases DeepSearcher
- **Sep 2025:** Sentient AGI releases ROMA beta v0.1.0 (recursive hierarchical agents)
- **Jan 2026:** OPPO PersonalAI releases O-Researcher (multi-agent distillation + agentic RL)
- **2025-2026:** AgentRxiv and aiXiv establish platforms for AI-generated research dissemination

---

## 2. Categories of Open Research Frameworks

### 2.1 Deep Research Agents
Autonomous agents that conduct web research and generate structured reports. Direct open-source counterparts to OpenAI's Deep Research. **Core capabilities:** accept research question → plan strategy → search web/academic sources → extract and synthesize → produce cited report. **Examples:** LangChain Open Deep Research, HF smolagents ODR, DeepSearcher.

### 2.2 Scientific Discovery Platforms
Go beyond literature review to support hypothesis generation, experimental design, and validation. **Core capabilities:** generate testable hypotheses from literature, design experiments, execute analysis code, draw conclusions. **Examples:** AgentLaboratory.

### 2.3 AI Research Scientists
Systems aiming to automate the full research lifecycle from idea to publication. **Core capabilities:** literature review → hypothesis generation → experimental design → execution → analysis → paper writing with citations. **Examples:** AgentLaboratory, ROMA (configured for full workflows).

### 2.4 Literature Review Tools
Focused on searching, filtering, summarizing, and synthesizing academic literature. **Core capabilities:** academic DB search (arXiv, PubMed, Semantic Scholar), semantic similarity search, citation graph traversal, trend analysis. **Examples:** DeepSearcher (with academic DB integration), RAG-based literature tools.

### 2.5 Collaborative Research Platforms
Platforms where multiple agents (and humans) collaborate, share findings, and build on each other's work. **Core capabilities:** agent-to-agent communication, shared knowledge bases, iterative improvement cycles, preprint serving for AI-generated research. **Examples:** AgentRxiv, aiXiv, ROMA (multi-agent mode).

---

## 3. The Research Agent Architecture Pattern

### 3.1 Common Components

| Component | Role | Typical Implementation |
|-----------|------|----------------------|
| **Planner Agent** | Decomposes question into sub-questions; creates research plan | LLM with CoT/ReAct prompting; variations: one-shot plan (LangChain), dynamic plan (DeepSearcher), recursive plan (ROMA) |
| **Search/Retrieval Agent** | Executes searches across configured sources | Web APIs (Tavily, Bing, DuckDuckGo), academic APIs (arXiv, PubMed), vector DBs (Milvus, Chroma), MCP servers |
| **Extraction/Synthesis Agent** | Visits each source, extracts key info, produces summaries | Web crawler, PDF parser (pymupdf), table extractor + LLM summarizer |
| **Aggregation/Report Agent** | Combines findings across sources, formats output with citations | LLM with structured output prompts; cross-referencing, fact-checking, citation management |

### 3.2 Typical Workflow

Research Question → **PLAN** (break into sub-topics, plan search strategy) → **SEARCH** (web, arXiv, PubMed, vector DB, local docs) → **EXTRACT** (read and summarize each source) → **SYNTHESIZE** (combine findings across sources) → **VALIDATE** (cross-check facts, verify citations) → **WRITE** (produce structured report with citations)

### 3.3 Key Architectural Differences

**Single-Agent vs. Multi-Agent:** Single-agent (HF smolagents) is simpler, easier to debug, but limited by one model's context window and reasoning depth. Multi-agent (ROMA, LangChain ODR) enables parallelism, specialized models per task, and better context management but requires orchestration infrastructure.

**Linear vs. Recursive vs. Iterative Planning:** Linear plans (LangChain initial plan) are fast and predictable but cannot adapt mid-research. Recursive decomposition (ROMA) handles arbitrary complexity with minimal per-agent context but adds overhead. Iterative deepening (DeepSearcher) is thorough and self-correcting but slower and more expensive.

**Fixed vs. Dynamic Search:** Fixed search strategies (pre-planned queries) work for well-understood topics but miss unexpected connections. Dynamic strategies (search evolves based on results) are better for exploratory research but risk infinite loops without termination criteria.

**Local-First vs. Cloud-Dependent:** Local-first (HF smolagents with DuckDuckGo + Ollama) enables fully private, zero-API-cost research. Cloud-dependent (LangChain ODR with Tavily) provides higher search quality but requires API keys and internet connectivity.

---

## 4. Detailed Framework Profiles

### 4.1 ROMA (Recursive Open Meta-Agent)

**Developer:** Sentient AGI team — **Release:** September 2025 (beta v0.1.0)
**Repository:** github.com/gogelabs/roma — **Paper:** arXiv 2602.01848

**Architecture:** Recursive hierarchical structure — parent nodes decompose tasks into parallelizable atomic subtasks for child nodes; results aggregated back up the hierarchy. Each parent agent receives a high-level task, decomposes it, dispatches subtasks to child agents (which may themselves decompose further), then aggregates child outputs. Supports heterogeneous agents (different LLMs, tools, strategies per subtask).

**Key Innovation — GEPA+:** Multi-component prompt optimization achieving 1.75× efficiency and stability improvement over earlier approaches (OSPA single-component, GEPA basic group). Instead of optimizing prompts one component at a time (risking conflicting updates), GEPA+ optimizes across components simultaneously.

**Benchmarks:** SOTA on SEAL-0 structured reasoning (+15-20% over industry deep research), top-tier on FRAMES multi-hop QA, strong on EQ-Bench. Outperforms commercial deep research platforms on tasks requiring systematic decomposition.

**Strengths:** Handles complex long-horizon problems single-agent systems cannot; transparent hierarchical execution traces enable debugging and attribution; context-engineering friendly (each agent sees only relevant context); model-agnostic; parallel subtask execution.

**Limitations:** Complex setup requires understanding recursive agent design; overkill for simple questions; cost accumulates with tree depth; newer ecosystem with smaller community.

**Best for:** Complex multi-step research requiring hierarchical decomposition, long-horizon reasoning (10+ reasoning steps), deep multi-agent collaboration, regulated environments requiring execution transparency.

### 4.2 OML (Open, Monetizable, and Loyal AI)

**Developer:** Sentient research team — **Release:** 2024 — **Paper:** arxiv.org/abs/2411.03887

**Important:** OML is **not a functional research agent framework.** It is a decentralized AI governance and monetization platform addressing AI monopolization through cryptography. Included because ownership/attribution questions become critical as autonomous research agents proliferate.

**Three Pillars:** (1) **Open** — transparent, community-driven development with public audits. (2) **Monetizable** — creators earn from contributions via fair value distribution; token-based incentives. (3) **Loyal** — cryptographic model fingerprinting (LWE-based digital signatures) ensures attribution, tracks usage, prevents unauthorized use.

**Architecture:** Blockchain-based model registry with unique cryptographic IDs, LWE-based fingerprinting for proof-of-ownership, decentralized contribution tracking, token-based governance.

**Strengths:** Addresses fundamental AI ownership/attribution problems; enables decentralized alternative to current oligopoly; community-governed.

**Limitations:** Conceptual/early-stage (paper, not product); blockchain dependency (tx costs, latency); computational overhead; chicken-and-egg adoption problem.

**Best for:** Understanding decentralized AI governance models, projects exploring open AI monetization, AI×crypto intersection research.

### 4.3 LangChain Open Deep Research

**Developer:** LangChain — **Release:** February 2025
**Repository:** github.com/langchain-ai/open_deep_research — **PyPI:** open-deep-research
**Benchmark:** #6 on Deep Research Bench Leaderboard (score 0.4344, August 2025)

**Architecture:** Supervisor-Researcher multi-agent system built on LangGraph. The Supervisor/Planner agent generates a complete report outline with sections. Each section is then researched and written **in parallel** by independent Search Agent + Writer Agent pairs. Final assembly combines sections into a Markdown report.

**Workflow:** Research question → Supervisor generates outline (sections, search strategies, sources) → For each section concurrently: Search Agent queries Tavily/Bing → Writer Agent reads sources and synthesizes → Final assembly with table of contents and formatted citations.

**Key Features:** Configurable (models, search APIs, writing styles, MCP servers), LangGraph state management (persistence, error recovery, checkpointing), LangSmith observability (full agent traces), parallel section generation, MCP server support for custom tool access.

**Strengths:** Production-deployable with reliability and monitoring; well-documented (README, blog, video); easy customization; large LangChain ecosystem; proven competitive (#6 DRB).

**Limitations:** Quality depends on search backend + LLM; web-focused (needs custom configuration for academic DBs); requires LangChain familiarity; cost scales with report sections.

**vs. HF ODR:** LangChain = Supervisor-Researcher (multi-agent, LangGraph, Tavily/Bing, more configurable). HF = CodeAgent (single-agent, smolagents, DuckDuckGo free, simpler). Both achieve similar GAIA scores (51-55%). Choose LangChain for production, HF for prototyping.

**Best for:** Teams in LangChain ecosystem, production deployments requiring reliability, deep customization of research behavior.

### 4.4 Hugging Face Open Deep Research (smolagents)

**Developer:** Hugging Face — **Release:** February 2025
**Blog:** huggingface.co/blog/open-deep-research — **Benchmark:** 55.15% GAIA validation set

**Architecture:** CodeAgent pattern — instead of generating JSON action sequences (traditional ReAct), the agent generates **Python code** for each step. Code is executed in a sandboxed environment. The insight: LLMs are trained on vast Python code and generate correct code more reliably than correct JSON action sequences. Code naturally provides loops, conditionals, data structures, and error handling without reinventing them as agent actions.

**Workflow:** Research question → Agent plans steps → For each step: generate Python code (search via DuckDuckGo, parse HTML, extract text, summarize) → Execute code → Feed results back → Continue or finalize → Final report.

**Key Features:** CodeAgent pattern, free DuckDuckGo search (no API key), lightweight dependencies, transparent execution (see every line of generated code), reproducible in ~24 hours.

**Strengths:** Simple architecture, quick setup (minutes), no API dependencies for search, budget-friendly (only LLM costs), educational value, transparent code generation.

**Limitations:** Less sophisticated than Supervisor-Researcher for complex research; DuckDuckGo quality < Tavily/Bing; web-focused; single-agent struggles with parallel work; no state persistence.

**Best for:** Simple free deep research, quick prototyping, education, budget-conscious setups (pair with Ollama for completely free research).

### 4.5 OpenManus

**Developer:** MetaGPT team — **Release:** 2025 — **Repository:** github.com/mannaandpoem/OpenManus

**Architecture:** Modular multi-agent framework with tool-use orientation. Agent Orchestrator routes queries to specialized agents: Browser Agent (Playwright-based browser automation), Code Agent (Python execution sandbox), Web Search Agent (configurable search APIs).

**Key Features:** General agent framework (not just research), browser automation for dynamic/JS-rendered content, code execution sandbox for data analysis, multi-agent collaboration.

**Strengths:** Versatile (research + web automation + data analysis + testing), Python-native, browser automation accesses content static scraping cannot reach (SPAs, login-walled content, interactive visualizations).

**Limitations:** Less specialized for research (lacks citation management, academic search integrations); newer/smaller community; broad scope = less depth in research features; browser overhead.

**Best for:** Teams wanting one framework for research + general automation, scenarios needing browser automation, custom research workflows combining web research with data analysis.

### 4.6 DeepSearcher (Zilliz)

**Developer:** Zilliz (creators of Milvus vector database) — **Release:** Early 2025
**Repository:** github.com/zilliztech/deep-searcher — **Stars:** ~8K

> **Note:** The well-known open-source DeepSearcher is by Zilliz, not Tencent. A separate smaller project (nicknochnack/deep-searcher) is unrelated.

**Architecture:** Iterative search-reflect-deepen. After each search→extract cycle, the agent **reflects** — explicitly identifies knowledge gaps, contradictions, and missing evidence — then searches deeper on identified gaps. This continues until sufficient information is gathered, then final synthesis.

**Workflow:** Initial query → Multi-source search (web + Milvus vector DB + local documents) → Content extraction and chunking → Reflection (what's missing? what needs deeper investigation? are there contradictions?) → Deeper search on identified gaps → Iterate → Final synthesis.

**Key Features:** Iterative depth search (not one-pass), reflection mechanism for knowledge gap identification, multi-source retrieval (web + vector DB + local files), Milvus integration for private data, supports many LLMs (OpenAI, DeepSeek, Grok, Claude, Llama, Qwen).

**Strengths:** Iterative deepening produces thorough research for complex multi-hop questions; reflection reduces hallucination by catching knowledge gaps; private data support via Milvus; transparent reasoning process.

**Limitations:** Slower (each iteration adds search + LLM calls); requires tuning `max_iter` parameter; smaller community than LangChain/HF; Milvus infrastructure overhead.

**Best for:** Complex research requiring deep iterative investigation, multi-hop queries, quality-focused research where speed is secondary, web + private data research combinations.

### 4.7 O-Researcher

**Developer:** OPPO PersonalAI — **Release:** January 2026
**Repository:** github.com/OPPO-PersonalAI/O-Researcher — **Paper:** arXiv 2601.03743

**Important:** This is a **trained model** (not a configurable framework) — research capability is baked into model weights via a training pipeline, rather than orchestrated at inference time.

**Approach:** (1) **Multi-agent distillation** — stronger models (GPT-4, Claude) generate diverse multi-turn research traces (search queries, page crawls, syntheses, citations) → distill into Qwen-2.5-72B-Instruct via supervised fine-tuning. (2) **Agentic RL** — model performs research tasks → rewarded on accuracy, citation quality, depth, efficiency → self-improves via PPO/GRPO. No proprietary training data used.

**Strengths:** SOTA open-source on deep research benchmarks; self-improving pipeline (gets better with more RL training); scalable to different model scales.

**Limitations:** Significant training compute (72B + RL); single model harder to customize than frameworks; newer with less adoption.

**Best for:** Research on improving research agents via RL, teams with GPU resources, understanding distillation+RL approach.

### 4.8 AgentLaboratory

**Developer:** Samuel Schmidgall et al. — **Release:** 2025 — **Repository:** github.com/SamuelSchmidgall/AgentLaboratory — **Stars:** ~5.7K

**Architecture:** Pipeline of specialized agents covering full research lifecycle: Human Idea → Literature Review Agent → Hypothesis Generation Agent → Experimental Design Agent (writes code) → Code Execution Agent → Analysis & Writing Agent → Research Paper + Code Repository. Each agent is optimized for its specific phase with domain-appropriate prompts and tools.

**Features:** End-to-end workflow, human-in-the-loop (designed to assist, not replace), code + paper output, AgentRxiv integration for publishing results.

**Strengths:** Covers full lifecycle (not just literature review); produces executable code and formatted papers; human-in-the-loop design.

**Limitations:** Quality depends heavily on domain and LLM; computationally expensive (many sequential agents); early-stage.

**Best for:** Full research lifecycle automation, prototyping research ideas quickly, educational settings.

### 4.9 AgentRxiv & aiXiv

**AgentRxiv** (Schmidgall et al., 2025): Preprint server for AI-generated research. Agents upload papers; other agents retrieve and build upon them. Demonstrates that agents can build on prior work to generate increasingly sophisticated research.

**aiXiv** (Zhang et al., 2025; arXiv 2508.15126): Next-gen open-access platform for human + AI scientists with iterative review pipelines and quality-controlled AI-generated research.

**Implications:** Towards autonomous 24/7/365 research communities; self-improving ecosystems where knowledge compounds across generations of agents; new publication models for AI-generated research.

**Challenges:** Quality control (AI-generated research varies wildly), novelty vs. noise distinction, originality assessment, citation integrity (AI citation hallucination), gaming the system (agents optimizing for publication count), human oversight role.

**Best for:** Understanding future of AI research dissemination, autonomous research communities, self-improving research ecosystems.

---

## 5. Framework Comparison Table

| Framework | Developer | Release | Architecture | Key Innovation | Search | Multi-Agent | Benchmark | Best For |
|-----------|-----------|---------|-------------|----------------|--------|-------------|-----------|----------|
| **ROMA** | Sentient AGI | Sep 2025 | Recursive hierarchical | GEPA+ prompt optimization; recursive decomposition | Any (configurable) | ✅ | SOTA SEAL-0, FRAMES | Complex hierarchical reasoning |
| **OML** | Sentient | 2024 | Decentralized governance | Crypto model fingerprinting (LWE) | N/A | N/A | N/A | AI governance & monetization |
| **LangChain ODR** | LangChain | Feb 2025 | Supervisor-Researcher (LangGraph) | Parallel sections; LangGraph state mgmt | Tavily, Bing, custom, MCP | ✅ | #6 Deep Research Bench | Production deployments |
| **HF ODR (smolagents)** | Hugging Face | Feb 2025 | CodeAgent (single-agent) | CodeAgent pattern (code > JSON) | DuckDuckGo (free) | ❌ | 55.15% GAIA | Quick prototyping, education |
| **OpenManus** | MetaGPT team | 2025 | Modular multi-agent | Browser automation + code execution | Configurable | ✅ | N/A | General-purpose + research |
| **DeepSearcher** | Zilliz | Early 2025 | Iterative search-reflect | Reflection mechanism + vector DB (Milvus) | Web + Milvus | ❌ | N/A | Deep iterative research |
| **O-Researcher** | OPPO | Jan 2026 | Trained model (RL) | Multi-agent distill + agentic RL | Built-in tool-integrated | 🟡 (training) | SOTA DRB | Research agent improvement |
| **AgentLaboratory** | Schmidgall et al. | 2025 | Pipeline of agents | End-to-end research lifecycle | Academic + code exec | ✅ | N/A | Full research lifecycle |
| **AgentRxiv** | Schmidgall et al. | 2025 | Collaborative platform | Agent preprint server | N/A (repository) | ✅ | N/A | Collaborative research |

---

## 6. Comparison Across Key Dimensions

### 6.1 Research Approach

| Approach | Framework | Best For |
|----------|-----------|----------|
| Recursive decomposition (hierarchical) | ROMA | Complex multi-step tasks, long-horizon reasoning |
| Supervisor-Researcher (parallel sections) | LangChain ODR | Structured multi-section reports |
| CodeAgent (single-agent code gen) | HF smolagents | Simple transparent research, minimal setup |
| General agent + tools (multi-purpose) | OpenManus | Flexible workflows combining research + automation |
| Iterative search-reflect (deepening) | DeepSearcher | Complex multi-hop questions needing depth |
| Trained model (distill + RL) | O-Researcher | High-performance single-model research |
| Pipeline of specialized agents | AgentLaboratory | End-to-end research lifecycle |

### 6.2 Use Case Fit

| Use Case | Best Framework(s) |
|----------|-------------------|
| Complex long-horizon research (10+ reasoning steps) | ROMA, DeepSearcher |
| Quick web research (minutes, 1-2 page reports) | HF smolagents ODR, LangChain ODR |
| Scientific discovery (hypothesis → experiment → paper) | AgentLaboratory |
| Private data research (web + internal documents) | DeepSearcher |
| Production research agent (reliability, monitoring) | LangChain ODR |
| Research agent improvement (RL, self-play) | O-Researcher |
| Custom multi-agent research system | ROMA, OpenManus |
| Educational / understand how agents work | HF smolagents ODR |
| AI governance / attribution study | OML |

### 6.3 Deployment & Cost

| Framework | Local | Cloud | API Dependencies | Cost/Report (GPT-4o class) |
|-----------|-------|-------|-----------------|---------------------------|
| ROMA | ✅ | ✅ | LLM API (optional local model) | $0.50-$20 |
| LangChain ODR | ✅ | ✅ | LLM API + search API (Tavily/Bing) | $0.50-$10 |
| HF smolagents | ✅ | ✅ | LLM API only (DuckDuckGo free) | $0.02-$5 |
| OpenManus | ✅ | ✅ | LLM API (optional local) | $0.05-$10 |
| DeepSearcher | ✅ | ✅ | LLM API + Milvus (optional local) | $0.10-$15 |
| O-Researcher | ✅ | ✅ | Model weights + GPU | $0.01-$1 (inference) |
| AgentLaboratory | ✅ | ✅ | LLM API | $2-$50 |

**Cost ranges:** Simple query=$0.02-0.05 (2-3 LLM calls); Standard report=$0.50-1.50 (15-25 calls); Deep research=$2-10 (30-100 calls); Complex hierarchical=$5-50+ (100-500 calls). **Savings:** use open-weight models locally via Ollama/vLLM, DuckDuckGo for free search, cache results, set hard limits on iterations.

### 6.4 Extensibility

| Framework | Customization | Add Tools | Change Models | Community |
|-----------|--------------|-----------|---------------|-----------|
| ROMA | Moderate | Easy (tool registry) | Easy | Growing |
| LangChain ODR | High | Easy (LangChain tools, MCP) | Easy | Very Large |
| HF smolagents | Moderate | Moderate (extend CodeAgent) | Easy | Large |
| OpenManus | High | Easy (modular tools) | Easy | Growing |
| DeepSearcher | Moderate | Moderate (provider config) | Easy | Small-Med |
| O-Researcher | Low (model) | Moderate | Requires retrain | Small |
| AgentLaboratory | Moderate | Moderate (add/remove agents) | Easy | Small-Med |

---

## 7. Ecosystem Trends

### 7.1 Convergence

- **Multi-agent architectures are standard** — every active framework is multi-agent or evolving that direction. Single-agent approaches (HF smolagents) valued for simplicity but acknowledged as limited for complex research.
- **Tool-use paradigm is universal** — the "augmented LLM" (LLM + tools) has become the default architecture. Research agents without tools are trivial; tools give them real-world power.
- **Search quality is the key differentiator** — as LLMs converge, frameworks compete on backend quality (Tavily > Bing > DDG), strategy (iterative vs. one-pass), and source diversity (web + academic + vector DB + code).
- **Vector databases become standard infrastructure** — Milvus, Chroma, Pinecone power private data retrieval across frameworks.

### 7.2 Divergence

| Axis | Left End | Right End |
|------|----------|-----------|
| Agent count | Single-agent ⟷ Multi-agent |
| Planning | Fixed one-shot ⟷ Dynamic iterative ⟷ Recursive |
| Search | Free (DDG) ⟷ Paid API (Tavily) |
| Scope | Research-specialized ⟷ General-purpose |
| Modularity | Monolithic trained model ⟷ Modular framework |
| Human role | Fully autonomous ⟷ Human-in-the-loop |

### 7.3 Emerging Capabilities

- **Reflection & self-correction:** Agents evaluate own output, identify gaps, correct errors (DeepSearcher reflection, ROMA hierarchical aggregation)
- **Tool creation:** Agents write Python functions on-the-fly for novel sub-problems (OpenManus, CodeAgent)
- **Cross-source validation:** Active cross-referencing to detect contradictions and verify facts
- **Cited output generation:** In-text citations with URL verification, source prioritization, deduplication
- **Multi-modal research:** Images (figures, charts), tables, code execution, equations, video/audio

### 7.4 Future Directions

1. **Self-improving research agents** through RL and self-play (O-Researcher trailblazing)
2. **Collaborative agent research communities at scale** (AgentRxiv-style platforms with thousands of agents)
3. **Domain-specialized agents** optimized for biology, physics, materials science, social sciences
4. **Integrated experimental platforms** combining research agents with lab automation, simulation environments
5. **Research evaluator frameworks** for automated hallucination detection, citation verification, novelty assessment

---

## 8. Challenges and Open Problems

### 8.1 Hallucination in Research
The **#1 unsolved problem.** Fabricated citations, incorrect facts embedded in plausible text, confidently wrong answers. Especially dangerous in research because output looks authoritative. Current mitigations (cross-source validation, reflection, citation verification, human review) are **insufficient** — all frameworks still struggle.

### 8.2 Citation Quality
Even real citations can be irrelevant (paper doesn't support claim), outdated (superseded), misinterpreted (claim misrepresents source), or cherry-picked (contradicting evidence ignored). Need verification of the claim-support relationship, not just citation existence.

### 8.3 Depth vs. Breadth Trade-off
Every agent must allocate resources: deep on few sources vs. broad on many. Current frameworks use static configuration parameters (iteration depth, section count, search count) rather than intelligent dynamic allocation based on question type and domain.

### 8.4 Cost Management
Research agents can be deceptively expensive at scale. Strategies: tiered models (small/cheap for extraction, large for reasoning), search caching, hard caps, local open-weight models, early stopping when sufficient information gathered.

### 8.5 Evaluation
Measuring research quality is itself an open research question. GAIA (200 real-world questions) and Deep Research Bench (100 PhD-level tasks) exist but neither fully captures research value. The field needs benchmarks for citation accuracy, depth of analysis, novelty, factual correctness on complex claims, and practical research utility.

---

## 9. Getting Started Guide

### 9.1 Quick Framework Selection

| Need | Start With |
|------|-----------|
| Production-ready customizable research agent | **LangChain Open Deep Research** |
| Simple free research agent (fastest to running) | **Hugging Face smolagents + ODR** |
| Hierarchical deep reasoning (complex problems) | **ROMA** |
| General-purpose agent that also researches | **OpenManus** |
| Deep iterative research (quality-first) | **DeepSearcher** |
| Decentralized AI governance (study) | **OML** (paper) |
| Research agent improvement via RL | **O-Researcher** |
| End-to-end research lifecycle | **AgentLaboratory** |

### 9.2 Recommended Learning Path

1. **HF smolagents ODR** — understand core loop (plan→search→extract→synthesize) in 15 minutes with free search
2. **LangChain ODR** — experience multi-agent Supervisor-Researcher; compare quality and customization
3. **DeepSearcher** — learn iterative research with reflection; understand depth/breadth trade-offs
4. **ROMA** — recursive decomposition for long-horizon tasks once you understand the research pattern
5. **Specialized:** AgentLaboratory (full lifecycle), O-Researcher (RL improvement), AgentRxiv (collaborative)

### 9.3 Quick Starts

**HF smolagents (simplest, free search):**
```bash
pip install smolagents
git clone https://github.com/huggingface/smolagents.git
cd smolagents/examples/open_deep_research
export OPENAI_API_KEY=your_key
python open_deep_research.py --question "Latest advances in LLM alignment?"
```

**LangChain ODR (production):**
```bash
git clone https://github.com/langchain-ai/open_deep_research.git
cd open_deep_research && uv venv && source .venv/bin/activate && uv sync
export ANTHROPIC_API_KEY=your_key && export TAVILY_API_KEY=your_key
python open_deep_research.py --query "Compare transformer architectures for long context"
```

**ROMA (hierarchical):**
```bash
git clone https://github.com/gogelabs/roma.git && cd roma && pip install -r requirements.txt
python run.py --task "Analyze current state of mechanistic interpretability"
```

**DeepSearcher (iterative):**
```bash
git clone https://github.com/zilliztech/deep-searcher.git && cd deep-searcher && pip install -e .
export OPENAI_API_KEY=your_key
python deep_searcher/main.py --query "Failure modes of chain-of-thought?" --iterations 3
```

---

## 10. References

**Papers:** arXiv 2602.01848 (ROMA), arXiv 2411.03887 (OML), arXiv 2601.03743 (O-Researcher), Schmidgall et al. 2025 (AgentRxiv), arXiv 2508.15126 (aiXiv), arXiv 2508.19113 (Hybrid Deep Searcher)

**Repositories:** github.com/gogelabs/roma, github.com/langchain-ai/open_deep_research, github.com/huggingface/smolagents, github.com/mannaandpoem/OpenManus, github.com/zilliztech/deep-searcher, github.com/OPPO-PersonalAI/O-Researcher, github.com/SamuelSchmidgall/AgentLaboratory, agentrxiv.github.io

**Benchmarks:** GAIA (huggingface.co/spaces/gaia-benchmark/leaderboard), Deep Research Bench (futuresearch.ai/deep-research-bench/)

**Key Blogs:** Hugging Face "Open-source DeepResearch" (Feb 2025), LangChain "Open Deep Research: Evolution" (Jul 2025), Zilliz "Introducing DeepSearcher" (Feb 2025)

---

*This guide is part of the LLM/AI Technical Series. Companion guides: Hierarchical Multi-Agent Frameworks, MCP, Fine-Tuning, Agent Drift.*

*Last updated: July 2026*
