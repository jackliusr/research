# AI Agent Frameworks from China: A Comprehensive Analysis

**Date:** July 2026  
**Author:** Hermes Agent Research  

---

## Table of Contents

1. [Overview of the Chinese AI Agent Landscape](#1-overview-of-the-chinese-ai-agent-landscape)
2. [Framework-by-Framework Deep Dive](#2-framework-by-framework-deep-dive)
3. [Comparison Matrix](#3-comparison-matrix)
4. [Unique Chinese Characteristics](#4-unique-chinese-characteristics)
5. [Model Ecosystem](#5-model-ecosystem)
6. [Ecosystem Comparison: China vs West](#6-ecosystem-comparison-china-vs-west)
7. [Regulatory & Compliance](#7-regulatory--compliance)
8. [Recommendations](#8-recommendations)
9. [References](#9-references)

---

## 1. Overview of the Chinese AI Agent Landscape

### Why China Developed Its Own Agent Frameworks

China's emergence as a parallel AI ecosystem to the West is not accidental — it is the result of deliberate geopolitical, economic, and technological forces converging over the past five years.

#### US Export Controls and the Chip Barrier

The cornerstone of China's AI independence drive is the US export control regime. Beginning with the October 2022 Biden administration rules restricting export of advanced semiconductors (NVIDIA A100/H100) to China, and continuing through successive rounds of tightening, China's AI industry has been forced to operate under significant hardware constraints. The A800 and H800 "chip down" variants were also banned in October 2023, and subsequent regulations have targeted lithography equipment and chip design tools. This has created two consequences: (1) Chinese AI companies must maximize efficiency from available hardware (Huawei Ascend 910/910B, Cambricon, etc.), and (2) they have strong incentives to build software stacks optimized for domestic hardware. Agent frameworks are part of this stack — they must run efficiently on domestic GPUs and integrate with domestic cloud platforms.

#### The Domestic LLM Market

China has developed a rich ecosystem of large language models that, while generally not matching GPT-4-class performance until recently, have achieved remarkable parity with DeepSeek-V3/R1 and Qwen2.5-72B now competitive with top Western models. The landscape includes:

- **Alibaba's Qwen series** (Qwen2.5, Qwen3) — open-weight models reaching 72B+ parameters
- **Baidu's ERNIE Bot** — closed API, deeply integrated with Baidu's search and cloud
- **Zhipu AI's GLM series** — open-weight, strong tool-use capabilities
- **DeepSeek** — open-weight, world-class reasoning capabilities (DeepSeek-R1, 91k+ GitHub stars)
- **ByteDance's Doubao** — closed API, massive consumer reach through Douyin
- **01.AI's Yi series** — open-weight, strong multilingual performance
- **Baichuan AI** — open-weight, strong Chinese language understanding
- **iFlytek's Spark/Xinghuo** — closed API, speech-first interface
- **Minimax** — closed API, strong creative writing
- **Moonshot AI's Kimi** — long-context specialist (up to 2M tokens)

These models need agent frameworks to be useful for anything beyond simple chat. Chinese tech giants, recognizing this, have rushed to build the "application layer" while simultaneously developing their foundation models. Unlike the West where LangChain emerged as a community project before being adopted by enterprises, China's agent framework landscape is characterized by both top-down (big tech platforms) and bottom-up (open-source academic) development.

#### The Regulatory Environment

China's AI regulatory framework is perhaps the most comprehensive in the world and directly shapes agent framework design:

1. **Interim Measures for the Management of Generative AI Services** (August 2023, effective August 15, 2023) — The primary regulation governing generative AI. Requires content filtering, "socialist core values" alignment, and prohibits generation of illegal or harmful content.

2. **Algorithm Registration Requirements** — All AI-powered algorithms must be registered with the Cyberspace Administration of China (CAC). Agent frameworks that use algorithms must ensure their underlying models and orchestration logic comply.

3. **Deep Synthesis Regulation** — Covers AI-generated content that could be confused with real content. Agent frameworks that generate images, video, or audio content must implement watermarking and traceability.

4. **Data Sovereignty** — For many regulated industries (finance, healthcare, government), LLMs must be hosted on domestic servers. This drives demand for self-hosted agent frameworks like Dify and FastGPT.

These regulations create unique compliance requirements baked into Chinese frameworks — content filtering pipelines, audit logging, and model-level alignment are often integrated features, not afterthoughts.

#### Government Support and "New Quality Productive Forces"

The Chinese government has thrown its weight behind AI as a strategic priority. President Xi Jinping's concept of **"New Quality Productive Forces"** (新质生产力) — announced in 2023 and elevated to central policy in 2024 — positions AI as a transformative force that can drive productivity growth through technological innovation. This narrative directly supports:

- State funding for AI research (thousands of AI-related projects through NSFC and MoST)
- Preferential procurement policies favoring domestic AI solutions in government and state-owned enterprises
- Tax incentives for AI R&D
- Support for AI talent development (universities expanding AI programs, talent recruitment programs)

The "New Quality Productive Forces" framing means AI adoption is not just commercially driven but also a matter of national industrial policy. Chinese enterprises are incentivized to adopt domestic agent frameworks, contributing to their rapid growth and feature development.

### Key Players

| Player | Core AI Product | Agent Framework(s) | Cloud Platform |
|--------|----------------|-------------------|----------------|
| Alibaba Group | Qwen (Tongyi Qianwen) | AgentFabric, Qwen-Agent, ModelScope Agents | Alibaba Cloud (Aliyun) |
| Baidu | ERNIE Bot | AgentBuilder, Agents (Baidu) | Baidu AI Cloud |
| ByteDance | Doubao | Coze | Volcano Engine |
| Tencent | Hunyuan | Tencent Cloud AI Agent | Tencent Cloud |
| Tsinghua University | GLM (via Zhipu AI) | MetaGPT, AgentBench, Owl | — |
| Zhipu AI | GLM-4 | Agent development tools | Zhipu API |
| Shanghai AI Lab | InternLM | XAgent | — |
| DeepWisdom | — | MetaGPT | — |
| LangGenius | — | Dify | — |
| Labring | — | FastGPT | — |

### Landscape Overview

The Chinese agent framework landscape can be categorized into three tiers:

**Tier 1: Enterprise Platforms (Cloud-Backed)**
- Dify (147k GitHub stars), Coze (ByteDance), Baidu AgentBuilder, Alibaba Cloud Agents — these are full-stack platforms with visual builders, model hosting, and enterprise features. They compete directly with Western platforms like Azure AI Studio and OpenAI's GPTs ecosystem.

**Tier 2: Open-Source Developer Frameworks**
- Qwen-Agent, AgentScope, XAgent, Owl, Agently — code-first frameworks targeting developers building custom agent applications. Analogous to LangChain/LlamaIndex in the West.

**Tier 3: Specialized / Niche Frameworks**
- MetaGPT (multi-agent simulation), DB-GPT (data/DB agents), FastGPT (RAG/knowledge management), BISHENG (LLMOps), AnythingLLM (local-first) — frameworks optimized for specific use cases.

---

## 2. Framework-by-Framework Deep Dive

### 2a. AgentFabric / ModelScope Agents (Alibaba Cloud)

| Metadata | Value |
|----------|-------|
| **Developer** | Alibaba Cloud (Alibaba Group) |
| **Primary Language** | Python |
| **GitHub Stars** | ~8,994 (ModelScope); AgentFabric is a component of ModelScope |
| **License** | Apache-2.0 |
| **Created** | July 2022 (ModelScope) |
| **Last Updated** | June 2026 (active) |

**Architecture & Key Features:**

AgentFabric is Alibaba's agent framework built on top of **ModelScope** (魔搭社区), Alibaba's model-as-a-service platform akin to Hugging Face. ModelScope hosts thousands of models (NLP, CV, speech, multimodal) and provides a unified inference API. AgentFabric extends this with:

- **Multi-agent orchestration**: Define multiple agents that can collaborate on complex tasks
- **Tool/API integration**: Native connectors to Alibaba Cloud services (OSS, TableStore, MaxCompute, DataWorks, etc.)
- **Qwen model optimization**: Deep integration with Qwen models (Qwen2.5, Qwen3) including optimized inference
- **Code Interpreter**: Sandboxed Python execution environment for data analysis
- **Function Calling**: OpenAI-compatible function calling API for Qwen models
- **RAG Pipeline**: Built-in document parsing, chunking, embedding, and retrieval

**Model Support:** Primarily Qwen series (Qwen3, Qwen2.5), but also supports fine-tuned models hosted on ModelScope. Limited Western model support.

**Ecosystem:** Part of Alibaba Cloud's broader AI ecosystem including:
- Alibaba Cloud ModelStudio (visual model training)
- Alibaba Cloud PAI (Platform for AI)
- Elasticsearch for RAG
- MaxCompute for large-scale data

**Comparison to Western Equivalent:** AgentFabric maps most closely to **LangChain** in terms of agent orchestration and tool integration, but with much tighter cloud platform lock-in. It is less flexible than LangChain (which supports many model providers) but offers deeper integration with Chinese infrastructure.

**Documentation:** Available in Chinese and English, moderate quality. Alibaba Cloud documentation is generally adequate but less polished than Western equivalents.

---

### 2b. MetaGPT (DeepWisdom / Tsinghua University)

| Metadata | Value |
|----------|-------|
| **Developer** | DeepWisdom (originally Geekan, now associated with Tsinghua) |
| **Primary Language** | Python |
| **GitHub Stars** | 69,129 |
| **Forks** | 8,812 |
| **License** | MIT |
| **Created** | June 30, 2023 |
| **Last Updated** | January 2026 |
| **Topics** | agent, gpt, llm, metagpt, multi-agent |

**Architecture & Key Features:**

MetaGPT is arguably the most innovative open-source agent framework to emerge from China. Its core insight is that software development is a multi-person collaborative process, and this can be simulated by assigning distinct roles to LLM-powered agents:

- **Product Manager (PM)**: Analyzes requirements, writes PRDs
- **Architect**: Designs system architecture and component interactions
- **Engineer**: Writes actual code
- **QA Engineer**: Writes and runs tests

The agents communicate through a structured message bus, with each role bringing a different system prompt, toolset, and response format. This "software company in a box" approach produces remarkably coherent output — the framework can take a one-line requirement and produce a complete codebase with tests, documentation, and deployment scripts.

**Key Capabilities:**
- Automated SOPs (Standard Operating Procedures) generation
- Code generation from natural language specifications
- Requirements documentation with design diagrams
- Automated code review
- Multi-turn task decomposition and execution
- Integration with GPT-4, Claude, Qwen, GLM, DeepSeek

**Model Support:** Excellent — supports OpenAI GPT-4/GPT-4o, Anthropic Claude, Google Gemini, Qwen, GLM, DeepSeek, and many open-source models via API. One of the most model-agnostic Chinese frameworks.

**Release History:** Very active in 2023-2024 with major version releases (v0.5, v0.7, v0.8). Development pace has slowed in 2025-2026 but the project remains actively maintained.

**Documentation:** Good English and Chinese documentation. Extensive README with examples, research papers published about the methodology.

**Comparison to Western Equivalent:** MetaGPT maps to **CrewAI + AutoGen** in the West. However, MetaGPT's role-based architecture is more structured and opinionated than CrewAI's flexible agent definitions. Where CrewAI lets you define any agent with any tools, MetaGPT prescribes specific software development roles — it's less flexible but more productive for its target use case.

**Notable:** The "First AI Software Company" framing has been highly influential. Many subsequent multi-agent frameworks (including Western ones like AutoGen's team feature) have adopted similar role-playing patterns.

---

### 2c. AutoGPT (Significant Gravitas) — Western Context

| Metadata | Value |
|----------|-------|
| **Developer** | Significant Gravitas (Western, US-based) |
| **GitHub Stars** | 185,220 |
| **Forks** | 46,117 |
| **Created** | March 16, 2023 |
| **Language** | Python |

While not a Chinese framework, AutoGPT warrants mention because:

1. It is extremely popular in China's developer community (many Chinese-language tutorials, forks, and adaptations)
2. It inspired Chinese spin-offs like **AgentGPT** (reworkd/AgentGPT, 36,234 stars) — a browser-based fork that became independently popular in China
3. Many Chinese frameworks (XAgent, Owl) explicitly cite AutoGPT as inspiration

**AgentGPT (Chinese Fork):** Reworkd's AgentGPT (36k+ stars) is a web-based implementation of the AutoGPT concept, allowing users to configure and deploy autonomous agents through a browser UI. It has a significant Chinese user base.

---

### 2d. Agents (Baidu) / Baidu AgentBuilder

| Metadata | Value |
|----------|-------|
| **Developer** | Baidu |
| **Primary Language** | Python (SDK), Visual (Builder) |
| **GitHub** | Closed-source (proprietary) |
| **Created** | 2023 |
| **Platform** | Baidu AI Cloud |

**Architecture & Key Features:**

Baidu's agent framework exists in two forms:

1. **AgentBuilder (千帆智能体)** — A visual, no-code/low-code platform for building agents on Baidu AI Cloud. Users drag-and-drop components to create conversational agents, RAG bots, and workflow automations.

2. **Agents SDK** — A Python/API-based SDK for programmatic agent development, integrated with Baidu's ERNIE Bot.

**Key Features:**
- **ERNIE Bot integration**: Deep, proprietary integration with Baidu's flagship LLM
- **Search-enhanced agents**: Leverages Baidu's search index for RAG — arguably the strongest web-search capabilities of any Chinese framework
- **Knowledge Base**: Built-in document management with Baidu's embedding models
- **Workflow engine**: Visual DAG-based workflow definition
- **Enterprise features**: SSO, auditing, role-based access control
- **Chinese language optimization**: ERNIE is specifically trained on Chinese internet data, giving it an edge in Chinese language tasks

**Model Support:** Primarily ERNIE Bot. Limited support for other models.

**Comparison to Western Equivalent:** AgentBuilder is most analogous to **Azure AI Studio** — a managed, enterprise-focused platform with deep cloud integration, visual tools, and robust compliance features. It is less flexible than open-source alternatives but offers the smoothest experience for existing Baidu Cloud customers.

---

### 2e. XAgent (Shanghai AI Lab / OpenXLab)

| Metadata | Value |
|----------|-------|
| **Developer** | Shanghai Artificial Intelligence Laboratory (OpenXLab) |
| **Primary Language** | Python |
| **GitHub** | Repository appears to have been removed or migrated (404 at original OpenXLab/XAgent) |
| **Created** | July 2023 (original release) |
| **Status** | Archive/maintenance mode? |

**Note:** As of July 2026, the original XAgent repository at github.com/OpenXLab/XAgent returns 404. The project may have been archived, renamed, or its organization restructured. This section is based on its historical documentation and community presence.

**Architecture & Key Features (Historical):**

XAgent was designed as a "general-purpose agent for complex tasks" — a direct Chinese counterpart to AutoGPT. Its architecture featured:

- **Dual-Loop Task Decomposition**: An outer planning loop (high-level task decomposition) and an inner execution loop (step-by-step tool use)
- **Tool Server**: A separate process managing tool execution and output
- **Function Calling**: Strong integration with function-calling capable models
- **Memory System**: Short-term and long-term memory with vector store integration
- **Multi-Turn Conversation**: Stateful interaction management

**Model Support:** Supported multiple LLMs including GPT-4, Qwen, GLM, and internLM (Shanghai AI Lab's own model).

**Comparison to Western Equivalent:** XAgent was a direct analogue to **AutoGPT** with similar autonomous task-completion capabilities. Its dual-loop architecture was more sophisticated than early AutoGPT but less mature than later versions.

---

### 2f. CAMEL (KAUST / CAMEL-AI) — Popular in China

| Metadata | Value |
|----------|-------|
| **Developer** | KAUST (Saudi Arabia) / CAMEL-AI |
| **Primary Language** | Python |
| **GitHub Stars** | 17,305 |
| **Forks** | 1,974 |
| **License** | Apache-2.0 |
| **Created** | March 17, 2023 |
| **Last Updated** | June 2026 (very active) |

**Architecture & Key Features:**

CAMEL (Communicative Agents for "Mind" Exploration of Large Language Model Society) pioneered the role-playing multi-agent paradigm that later influenced MetaGPT and others. Its architecture:

- **Role-Playing Framework**: Two or more agents with assigned roles engage in structured conversation
- **Task-Oriented Communication**: Agents communicate via a structured message protocol to complete tasks
- **Inception Prompting**: A technique for generating complex multi-agent conversations from simple prompts
- **Extensible Agent Types**: AI Society, Code, and more specialized agent configurations

While CAMEL itself is not Chinese-developed (it originated from KAUST in Saudi Arabia and is now developed by the global CAMEL-AI team), it has achieved extraordinary popularity in China's AI community:

- Widely cited in Chinese AI research papers
- Used as a foundation for many Chinese multi-agent projects
- The Owl project (2g) is built on CAMEL
- Extensively documented in Chinese AI blogs and tutorials

**Comparison to Western Equivalent:** CAMEL is the direct predecessor of **CrewAI's** multi-agent orchestration but predates it by several months. It is more research-focused and less production-ready than CrewAI.

---

### 2g. DB-GPT (csunny/dbgpt)

| Metadata | Value |
|----------|-------|
| **Developer** | csunny / DB-GPT Team |
| **Primary Language** | Python |
| **GitHub Stars** | 19,309 |
| **Forks** | 2,784 |
| **License** | MIT |
| **Created** | April 13, 2023 |
| **Last Updated** | June 2026 (very active) |
| **Topics** | agents, bgi, database, deepseek, gpt, gpt-4, hacktoberfest, llm, private, rag |

**Architecture & Key Features:**

DB-GPT distinguishes itself by focusing on the intersection of databases and LLMs — it is an "agentic AI data assistant" that understands SQL, databases, and data pipelines natively.

**Key Features:**
- **Natural Language to SQL**: Conversational database querying across multiple SQL dialects
- **Multi-Model Support**: Plug in any LLM (OpenAI, Qwen, DeepSeek, GLM, etc.)
- **RAG Pipeline**: Integrated document parsing, chunking, embedding, vector storage
- **Agent Framework**: Multi-agent orchestration for complex data workflows
- **Data Visualization**: Automatic chart and dashboard generation from query results
- **Reporting**: Automated report generation from data analysis
- **Fine-tuning**: Built-in support forfine-tuning models on enterprise data
- **AWEL (Agentic Workflow Expression Language)**: A workflow language for defining agent orchestration logic

**Model Support:** Excellent multi-model support including DeepSeek, Qwen, GLM, Yi, Baichuan, alongside OpenAI, Claude, Gemini. DB-GPT is one of the most model-agnostic Chinese frameworks.

**Ecosystem:** Integrates with LangChain, LlamaIndex, and has its own plugin system. Supports multiple vector databases (Milvus, Chroma, FAISS, Qdrant).

**Documentation:** Comprehensive Chinese documentation, growing English documentation. Active community on Chinese developer platforms.

**Comparison to Western Equivalent:** DB-GPT combines features from **LangChain** (agent orchestration) and **LlamaIndex** (data indexing/retrieval) with a database-focused twist. It is more specialized than either but offers superior database integration capabilities.

---

### 2h. Agently (Maplemx)

| Metadata | Value |
|----------|-------|
| **Developer** | Maplemx |
| **Primary Language** | Python, JavaScript |
| **GitHub Stars** | 3 |
| **License** | Apache-2.0 |
| **Created** | January 2026 |
| **Last Updated** | April 2026 |

**Architecture & Key Features:**

Agently is a very new framework (released early 2026) that describes itself as a "GenAI Application Development Framework." Key architectural features:

- **Event-Driven Flow**: Agent interactions are defined as event-driven flows
- **Chain-of-Thought Support**: Built-in CoT prompting patterns
- **Multi-Provider**: Supports OpenAI, Azure, and Chinese LLM providers
- **Lightweight**: Minimal dependencies, easy to integrate

**Model Support:** OpenAI, Azure OpenAI, plus Chinese models via API.

**Current State:** With only 3 GitHub stars as of July 2026, Agently has minimal traction. It is too new and unproven to recommend for any use case. Listed here for completeness as the user requested coverage of "all" frameworks mentioned.

---

### 2i. Qwen-Agent (Alibaba)

| Metadata | Value |
|----------|-------|
| **Developer** | Alibaba Group (Qwen Team) |
| **Primary Language** | Python |
| **GitHub Stars** | 16,636 |
| **Forks** | 1,656 |
| **License** | Apache-2.0 |
| **Created** | September 22, 2023 |
| **Last Updated** | March 2026 |
| **Open Issues** | 520 (significant backlog) |

**Architecture & Key Features:**

Qwen-Agent is Alibaba's official agent framework for the Qwen model family. It is a code-first developer framework (contrast with AgentFabric/ModelScope which targets enterprise users).

**Key Features:**
- **Function Calling**: Native OpenAI-compatible function calling for Qwen models
- **MCP (Model Context Protocol) Support**: Implementation of the emerging MCP standard
- **Code Interpreter**: Sandboxed Python execution for code generation/analysis
- **RAG Integration**: Document chunking, embedding, and retrieval pipelines
- **Chrome Extension**: The "Qwen Browser Assistant" enables web-based agent interaction
- **Multi-Turn Conversation Management**: State tracking across sessions
- **Tool Registration**: Simple decorator-based API for adding custom tools
- **Streaming**: Real-time response streaming
- **Batch Processing**: Efficient batch inference support

**Architecture:**
```
Qwen-Agent Core
├── Agent (Base agent with memory, planning, tool use)
│   ├── ChatAgent (Conversation-focused)
│   ├── AssistantAgent (Task-completion focused)
│   └── CodeAgent (Code generation/execution)
├── Tool Registry
│   ├── Built-in tools (search, weather, calc, etc.)
│   └── Custom tool API
├── RAG Engine
│   ├── Document loader
│   ├── Text splitter
│   └── Vector store integration
├── MCP Client
└── Web UI (Streamlit-based playground)
```

**Model Support:** Primarily Qwen (Qwen3, Qwen2.5). Limited support for other models. This is the most Qwen-optimized framework available.

**Documentation:** Good English and Chinese documentation. Active community on GitHub Discussions.

**Comparison to Western Equivalent:** Qwen-Agent maps most closely to **OpenAI's Assistants API** — both provide a code-first agent framework tightly coupled to a specific model family. Qwen-Agent is more open (Apache-2.0) and self-hostable than Assistants API (which is cloud-only), but less feature-rich in terms of managed infrastructure.

**Notable Issue:** 520 open issues suggests significant maintenance burden or rapid feature development creating a documentation/quality gap.

---

### 2j. Baidu AgentBuilder (千帆智能体)

| Metadata | Value |
|----------|-------|
| **Developer** | Baidu |
| **Primary Language** | Visual (no-code/low-code), Python SDK |
| **GitHub** | Closed-source (proprietary SaaS) |
| **Created** | 2023 |
| **Platform** | Baidu Qianfan AI Platform |
| **Target Audience** | Enterprise |

**Architecture & Key Features:**

Baidu AgentBuilder (part of the Qianfan platform) is a full-lifecycle agent development and deployment platform:

- **Visual Agent Builder**: Drag-and-drop interface for defining agent behavior, knowledge bases, and tools
- **Prompt Optimization Tools**: A/B testing, prompt templates, version management
- **Knowledge Base Management**: Upload documents, auto-chunking, embedding with ERNIE
- **Plugin System**: Pre-built plugins for Baidu Maps, Baidu Search, Baidu Translate, and third-party services
- **Multi-Channel Deployment**: Deploy agents to Baidu Search, Baidu APP, WeChat mini-programs, and custom websites
- **Monitoring & Analytics**: Usage tracking, conversation logs, quality scoring
- **Compliance**: Built-in content filtering aligned with Chinese regulations

**Model Support:** Exclusive to Baidu's ERNIE Bot. No support for Western models.

**Pricing:** Usage-based pricing with free tier. Enterprise plans with custom pricing.

**Comparison to Western Equivalent:** AgentBuilder is most similar to **Azure AI Studio** with some overlap with **OpenAI's GPTs** ecosystem. It is more enterprise-focused than OpenAI GPTs but less open than Azure AI Studio.

---

### 2k. Coze (ByteDance)

| Metadata | Value |
|----------|-------|
| **Developer** | ByteDance |
| **Primary Language** | Visual (no-code/low-code) |
| **GitHub** | Closed-source (proprietary SaaS) |
| **Created** | 2023 (public release 2024) |
| **Platform** | Coze.cn (China), Coze.com (International) |
| **Target Audience** | General users, creators, enterprises |

**Architecture & Key Features:**

Coze (扣子) is ByteDance's ambitious entry into the agent platform space. It is the most consumer-friendly Chinese agent platform and has gained significant traction.

**Key Features:**
- **Visual Workflow Builder**: Intuitive drag-and-drop for creating complex multi-step agent workflows
- **Plugin Ecosystem**: Extensive marketplace of plugins (news, productivity, entertainment, e-commerce)
- **Knowledge Base**: Upload documents, websites, or structured data with automatic processing
- **Bot Publishing**: Deploy agents to Douyin (TikTok China), WeChat (via mini-programs), Feishu (Lark), WhatsApp, and custom websites
- **Advanced Prompt Engineering**: Built-in tools for prompt testing, versioning, and optimization
- **Variable System**: Define and manipulate variables within workflows
- **Code Nodes**: Execute custom JavaScript/Python snippets within workflows
- **Multi-Model Support**: Supports Doubao (ByteDance), Qwen, GLM, ChatGPT, Claude, Gemini
- **Analytics Dashboard**: Engagement metrics, user satisfaction scores, conversation logs
- **Free Tier**: Generous free usage quota with paid tiers for higher limits
- **Template Gallery**: Hundreds of pre-built bot templates

**International Version:** Coze.com (international) supports more Western models and platforms; Coze.cn (China) is optimized for the domestic ecosystem.

**Model Support:** Excellent — supports Doubao, Qwen, GLM, DeepSeek, ChatGPT, Claude, Gemini. One of the most model-diverse Chinese platforms.

**Comparison to Western Equivalent:** Coze is best understood as **OpenAI GPTs + Assistants API + plugin ecosystem + multi-channel publishing**. It offers a superset of what OpenAI provides in GPTs, with the critical advantage of multi-platform deployment (especially Douyin and WeChat). Its plugin marketplace is more curated than OpenAI's.

**Pricing:** Free tier (limited usage). Creator plan (~$15-30/month). Enterprise (custom). Coze is significantly more affordable than comparable Western platforms for Chinese users.

---

### 2l. Dify (LangGenius)

| Metadata | Value |
|----------|-------|
| **Developer** | LangGenius |
| **Primary Language** | TypeScript (frontend), Python (backend) |
| **GitHub Stars** | 147,171 |
| **Forks** | 23,188 |
| **License** | NOASSERTION (commercial-friendly license) |
| **Created** | April 12, 2023 |
| **Last Updated** | July 2026 (extremely active) |
| **Topics** | agent, agentic-ai, agentic-framework, agentic-workflow, ai, automation, gemini, genai, gpt |

**Architecture & Key Features:**

Dify is the crown jewel of the Chinese open-source agent framework ecosystem. With 147,171 GitHub stars, it is the most popular AI framework to come out of China and one of the most popular AI tools globally.

**Key Features:**
- **Visual Workflow Builder**: DAG-based workflow editor for creating complex AI applications
- **RAG Pipeline**: End-to-end document ingestion, chunking, embedding, and retrieval
- **Agent Capabilities**: Function calling, tool use, multi-agent orchestration
- **Model Management**: Unified API for 100+ LLM providers including Chinese and Western models
- **Prompt Engineering**: Version control, A/B testing, template library
- **Dataset Management**: Upload and manage training/evaluation datasets
- **App Templates**: Pre-built application templates (chatbot, RAG QA, agent, etc.)
- **API Publishing**: RESTful API endpoints for deployed apps
- **Self-Hosted**: Docker Compose and Kubernetes deployment
- **Cloud Version**: Dify Cloud (managed hosting)
- **Conversation Logs**: Full interaction history with analytics
- **Annotation**: Human-in-the-loop feedback for model improvement
- **Plugin System**: Extensible via plugins

**Architecture:**
```
Dify Platform
├── Workflow Engine (DAG-based)
│   ├── LLM Nodes
│   ├── Knowledge Retrieval Nodes
│   ├── Code Nodes
│   ├── HTTP Request Nodes
│   └── Logic Nodes (condition, loop, variable)
├── RAG Engine
│   ├── Document Loaders
│   ├── Pre-processing Pipelines
│   ├── Embedding Models
│   └── Vector Stores (Weaviate, Qdrant, Milvus, etc.)
├── Agent Framework
│   ├── ReAct Agent
│   ├── Function Calling Agent
│   └── Multi-Agent Orchestrator
├── Model Provider Interface
│   ├── OpenAI / Azure
│   ├── Anthropic Claude
│   ├── Google Gemini
│   ├── Qwen
│   ├── DeepSeek
│   ├── GLM
│   └── 50+ others
├── API Layer
│   └── REST API + WebSocket
└── Frontend (React/Next.js)
```

**Model Support:** Excellent — supports OpenAI, Anthropic, Google, AWS Bedrock, Azure, Hugging Face, Ollama, Xinference, and all major Chinese models (Qwen, DeepSeek, GLM, Yi, Baichuan, Spark, Minimax).

**Release History:** Extremely active development with weekly releases. Major versions: v0.3 (RAG), v0.4 (workflow), v0.6 (multi-agent), v0.8 (enterprise features), v1.0 (2025).

**Documentation:** Excellent — comprehensive English and Chinese documentation, video tutorials, example projects, active Discord community (30k+ members).

**Enterprise Edition:** Paid enterprise version with SSO, audit logs, RBAC, priority support.

**Comparison to Western Equivalent:** Dify is best understood as **LangChain + Flowise + LlamaIndex all in one platform**. The comparison:
- **LangChain** provides the underlying orchestration primitives → Dify provides the visual/API layer on top
- **Flowise** provides drag-and-drop workflow → Dify provides a more polished version with broader model support
- **LlamaIndex** provides data indexing → Dify's RAG pipeline matches or exceeds it

Dify's killer advantage is **model diversity** — it is equally comfortable connecting to OpenAI and DeepSeek, Claude and Qwen. This makes it the default choice for teams that need to support both Chinese and Western AI infrastructure.

---

### 2m. FastGPT (Labring)

| Metadata | Value |
|----------|-------|
| **Developer** | Labring (Sealos Cloud) |
| **Primary Language** | TypeScript (frontend), Go (backend), Python |
| **GitHub Stars** | 28,744 |
| **Forks** | 7,182 |
| **License** | NOASSERTION |
| **Created** | February 23, 2023 |
| **Last Updated** | July 2026 (very active) |
| **Topics** | agent, claude, deepseek, llm, mcp, nextjs, openai, qwen, rag, workflow |

**Architecture & Key Features:**

FastGPT is a knowledge-based platform built on LLMs, specializing in RAG (Retrieval-Augmented Generation) and workflow automation.

**Key Features:**
- **Knowledge Base Management**: Upload documents (PDF, Word, Excel, Markdown, images), auto-chunking and indexing
- **RAG Pipeline**: Highly optimized retrieval with re-ranking, hybrid search (keyword + vector), and multi-path retrieval
- **Visual Workflow Automation**: Drag-and-drop workflow editor for complex LLM applications
- **Team Collaboration**: Multi-user support, role-based access, shared knowledge bases
- **Self-Hosted**: Easy deployment via Docker, Sealos cloud, or Kubernetes
- **API Access**: REST API for integration with external applications
- **MCP Support**: Model Context Protocol integration
- **Multi-Model**: Supports OpenAI, Claude, DeepSeek, Qwen, GLM, and others
- **OpenAPI**: Extensive API surface for custom development

**Model Support:** Excellent — supports OpenAI, Anthropic Claude, Google Gemini, DeepSeek, Qwen, GLM, Yi, Baichuan, and more.

**Ecosystem:** Tightly integrated with Labring's Sealos cloud platform (a Kubernetes distribution optimized for AI workloads). FastGPT can be deployed on Sealos with one click.

**Documentation:** Good Chinese documentation, growing English documentation.

**Comparison to Western Equivalent:** FastGPT is most similar to **RAGFlow** (an open-source RAG engine) and **AnythingLLM** (local-first document QA). Among Chinese frameworks, it is the strongest pure RAG platform alongside DB-GPT.

---

### 2n. Owl (CAMEL-AI / Tsinghua)

| Metadata | Value |
|----------|-------|
| **Developer** | CAMEL-AI (associated with Tsinghua University / KAUST) |
| **Primary Language** | Python |
| **GitHub Stars** | 19,906 |
| **Forks** | 2,293 |
| **License** | N/A |
| **Created** | March 3, 2025 |
| **Last Updated** | June 2026 (very active) |
| **Topics** | agent, artificial-intelligence, multi-agent-systems, task-automation, web-interaction |

**Architecture & Key Features:**

Owl (Optimized Workforce Learning) is a newer multi-agent framework focused on real-world task automation, built on top of the CAMEL project.

**Key Features:**
- **Dynamic Agent Team Formation**: Agents are dynamically assigned to teams based on task requirements
- **Web Interaction**: Built-in browser automation for web-based tasks
- **Task Decomposition**: Advanced task planning and decomposition algorithms
- **Multi-Agent Collaboration**: Agents communicate and coordinate to complete complex workflows
- **Human-in-the-Loop**: Optional human oversight and intervention
- **Real-World Task Automation**: Designed for practical tasks (not just research demos)

**Model Support:** Supports multiple LLMs through CAMEL's provider interface.

**Comparison to Western Equivalent:** Owl is conceptually closest to **CrewAI** (multi-agent task automation) with elements of **Browser-Use** (web interaction). It is more research-oriented than production-ready at this stage.

**Notable:** At 19,906 stars and having been created only in March 2025 (about 16 months ago), Owl has achieved remarkable growth, suggesting strong interest in multi-agent collaboration for practical tasks.

---

### 2o. BISHENG (Dataelem / Bisheng)

| Metadata | Value |
|----------|-------|
| **Developer** | Dataelem |
| **Primary Language** | TypeScript (frontend), Python (backend) |
| **GitHub Stars** | 11,480 |
| **Forks** | 1,876 |
| **License** | Apache-2.0 |
| **Created** | August 28, 2023 |
| **Last Updated** | July 2026 (very active) |
| **Topics** | agent, ai, chatbot, enterprise, finetune, genai, gpt, langchian, llama, llm |

**Architecture & Key Features:**

BISHENG is an open-source LLM operations (LLMOps) platform that covers the entire lifecycle of enterprise AI applications, from model hosting to agent management.

**Key Features:**
- **GenAI Workflow**: Visual workflow builder for complex AI pipelines
- **RAG Pipeline**: Full document processing and retrieval pipeline
- **Agent Management**: Deploy, monitor, and manage AI agents
- **Model Hosting**: Unified inference API for multiple models
- **Fine-tuning**: Built-in fine-tuning support for popular base models
- **Evaluation**: Model quality evaluation and A/B testing
- **Prompt Engineering**: Prompt management, versioning, and optimization
- **Enterprise DevOps**: CI/CD for AI applications, monitoring, alerting
- **Unified Model Management**: Single API for 50+ models

**Model Support:** Excellent — supports OpenAI, Claude, Gemini, Qwen, GLM, DeepSeek, Yi, Baichuan, Llama, and many open-source models.

**Comparison to Western Equivalent:** BISHENG combines features from **MLflow** (model lifecycle management), **LangFuse** (LLM observability), and **Kubeflow** (ML pipelines) into a single platform tailored for LLM/agent applications. It is more comprehensive than any single Western tool but also more complex.

---

### 2p. AnythingLLM (Mintplex Labs)

| Metadata | Value |
|----------|-------|
| **Developer** | Mintplex Labs |
| **Primary Language** | JavaScript (Node.js) |
| **GitHub Stars** | 62,358 |
| **Forks** | 6,810 |
| **License** | MIT |
| **Created** | June 4, 2023 |
| **Last Updated** | July 2026 (very active) |
| **Topics** | agent-harness, agentic-ai, ai-agents, hermes-agent, llm, local-ai, localai, multimodal, no-code, open-claw |

**Architecture & Key Features:**

AnythingLLM is a local-first, open-source document management and RAG platform with agent capabilities. Its philosophy is "stop renting your intelligence — own it."

**Key Features:**
- **Local-First**: All processing can run locally with no cloud dependency
- **Multi-Model**: Supports Ollama, OpenAI, Anthropic, Azure, Google, Hugging Face, and Chinese models
- **Document Management**: Upload, parse, and index documents (PDF, DOCX, TXT, code, websites)
- **RAG Pipeline**: Conversational QA over uploaded documents
- **Agent Capabilities**: Web search, code execution, custom tool integration
- **Multi-User**: Team workspaces with shared documents
- **Custom Embeddings**: Multiple embedding model options
- **Appearance Customization**: White-label options
- **API Access**: REST API for automation and integration

**Model Support:** Excellent — supports Ollama (any model), OpenAI, Claude, Gemini, Qwen, DeepSeek, GLM, and more. One of the most model-diverse local-first platforms.

**Comparison to Western Equivalent:** AnythingLLM is the most popular open-source **local RAG platform** in the West. It competes with **RAGFlow** and **LocalAI**. Its Chinese model support makes it uniquely useful for the Chinese market as well.

---

### Bonus: AgentScope (Alibaba / DAMO Academy)

| Metadata | Value |
|----------|-------|
| **Developer** | Alibaba DAMO Academy / ModelScope |
| **Primary Language** | Python (also Java variant) |
| **GitHub Stars** | ~27,343 (agentscope-ai/agentscope) |
| **License** | Apache-2.0 |
| **Created** | 2024 |
| **Last Updated** | Active |

**Architecture & Key Features:**

AgentScope is a multi-agent platform from Alibaba's DAMO Academy (Discovery, Adventure, Momentum, and Outlook — the research institute). It provides:

- **Agent-Oriented Programming**: Define agents as programmable entities with roles, capabilities, and communication protocols
- **Distributed Deployment**: Agents can run on different machines/machines
- **Service-as-Agent**: Expose any service as an agent
- **Multi-Modal Agents**: Support for text, image, audio, and video-capable agents
- **Java SDK**: One of the few Chinese agent frameworks with a full Java SDK

**Note:** There is also a Java variant (agentscope-ai/agentscope-java, ~4,180 stars).

---

## 3. Comparison Matrix

| Framework | Language | Open Source | GitHub Stars | Multi-Agent? | Visual Builder? | RAG Support? | Chinese Models | Western Models | Self-Hostable? | Cloud-Only? | Enterprise Focus? | Best Use Case |
|-----------|----------|-------------|-------------|-------------|----------------|-------------|---------------|---------------|--------------|------------|-----------------|--------------|
| **Dify** | TS/Python | ✅ | 147,171 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | Both | ✅ | Full-stack LLM app platform |
| **MetaGPT** | Python | ✅ | 69,129 | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | Multi-agent code generation |
| **AnythingLLM** | JS | ✅ | 62,358 | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | Both | ❌ | Local-first RAG |
| **AgentGPT** | TS | ✅ | ~36,234 | ❌ | ✅ | ❌ | ❌ | ✅ | ✅ | Both | ❌ | Browser-based AutoGPT |
| **FastGPT** | TS/Go | ✅ | 28,744 | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | Both | ✅ | Enterprise RAG/knowledge mgmt |
| **AgentScope** | Python | ✅ | ~27,343 | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | Multi-agent research |
| **Qwen-Agent** | Python | ✅ | 16,636 | ✅ | ❌ | ✅ | ✅ | ⚠️ Limited | ✅ | ❌ | ❌ | Qwen-powered agents |
| **DB-GPT** | Python | ✅ | 19,309 | ✅ | ⚠️ Partial | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | Database + AI agents |
| **Owl** | Python | ✅ | 19,906 | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | Multi-agent task automation |
| **CAMEL** | Python | ✅ | 17,305 | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | Multi-agent research |
| **BISHENG** | TS/Python | ✅ | 11,480 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | LLMOps for enterprises |
| **ModelScope** | Python | ✅ | 8,994 | ❌ | ❌ | ❌ | ✅ | ⚠️ Limited | ✅ | ❌ | ❌ | Model-as-a-service |
| **Coze** | Visual | ❌ | N/A | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ Cloud | ❌ | No-code agent building |
| **Baidu AgentBuilder** | Visual | ❌ | N/A | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ Cloud | ✅ | Enterprise ERNIE agents |
| **Agently** | Python/JS | ✅ | 3 | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | (Too new to recommend) |
| **XAgent** | Python | ✅ | (Removed) | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | Complex task automation (historical) |

**Notes:**
- Star counts are from July 2026
- "Both" in Cloud-Only means the framework offers both cloud and self-hosted options
- ⚠️ = partial or limited support
- Coze and Baidu AgentBuilder are proprietary SaaS — no GitHub repo

---

## 4. Unique Chinese Characteristics

### 4.1 Deep Integration with Chinese LLMs

The most defining characteristic of Chinese agent frameworks is their deep, native integration with China's domestic LLM ecosystem. Unlike Western frameworks that treat model providers as interchangeable API endpoints, Chinese frameworks often have:

- **Optimized prompt templates** for Chinese models' instruction-following patterns
- **Chinese tokenizer awareness** — some frameworks can split/deal with Chinese tokens more efficiently
- **Model-specific function calling formats** — Qwen's function calling API differs from OpenAI's in subtle ways; Chinese frameworks handle these differences natively
- **Fallback chains** — automatically switch between domestic models when one is unavailable (important given API reliability concerns with some Chinese providers)

### 4.2 Platform Integration (WeChat, Douyin, Feishu)

Chinese agent frameworks offer deployment options that simply don't exist in the West:

- **WeChat Ecosystem**: Mini-programs, Official Accounts, and WeChat Work integration. A Chinese enterprise agent often lives inside WeChat.
- **Douyin (TikTok China)**: ByteDance's Coze enables agents to be published as Douyin bots, reaching China's 700M+ Douyin users.
- **Feishu (Lark)**: ByteDance's enterprise collaboration platform, popular among tech companies.
- **Alibaba DingTalk**: Enterprise messaging/ collaboration platform integration.
- **Baidu Search / Baidu APP**: Baidu AgentBuilder agents can be embedded in search results.

This multi-channel publishing is a capability that Western frameworks are only beginning to explore.

### 4.3 Chinese Language Optimization

Chinese agent frameworks handle several linguistic challenges that Western frameworks face difficulties with:

- **Chinese tokenization**: Chinese text lacks spaces between words; effective chunking for RAG requires Chinese-aware tokenizers
- **Instruction following**: Chinese LLMs are optimized for Chinese instructions, which have different patterns than English (e.g., more implicit context, topic-prominent structure)
- **CJK character handling**: Some Western frameworks struggle with CJK character widths, rendering, and encoding

### 4.4 Government and Compliance Features

Framework-level compliance features are increasingly common in Chinese agent platforms:

- **Content filtering**: Built-in censorship/filtering pipelines using keyword matching + model-based detection
- **Audit logs**: All agent interactions logged for regulatory review
- **Model alignment**: Frameworks provide tools to align agent outputs with regulatory requirements
- **Data localization**: Self-hosted frameworks confidently handle data sovereignty requirements
- **Algorithm registration support**: Some frameworks provide metadata formats compatible with CAC registration

### 4.5 Cloud Ecosystem Lock-In

Chinese cloud vendors (Alibaba Cloud, Baidu Cloud, Tencent Cloud, Volcano Engine) actively incentivize use of their proprietary agent frameworks through:

- **Free tier credits** for using their agent platform
- **Optimized inference pricing** for domestic models on their cloud
- **Integrated billing** — agent usage and model inference billed together
- **Lower latency** for same-cloud inference
- **Compliance guarantees** — cloud vendors certify their platforms meet regulatory requirements

This creates a "walled garden" dynamic where enterprises are heavily incentivized to stay within a single cloud's ecosystem.

### 4.6 Mobile-First Agent Deployment

China's mobile-first internet culture means agents are primarily deployed as mobile experiences:

- **WeChat mini-programs** are the dominant distribution channel
- **Douyin bots** reach users in their entertainment context
- **Mobile app SDKs** for embedding agents

Western agent platforms (OpenAI GPTs, Assistants API) are predominantly web-based; Chinese platforms are mobile-first by necessity.

### 4.7 Visual/Low-Code Builder Popularity

China has a lower density of experienced AI/ML developers than the West, leading to higher demand for:

- **No-code agent builders** (Coze, Baidu AgentBuilder)
- **Visual workflow editors** (Dify, FastGPT)
- **Template galleries** (ready-to-deploy agent templates)

These tools allow non-technical users (product managers, operations staff, content creators) to build and deploy AI agents without writing code. This trend is more pronounced in China than in the West, where developer-oriented frameworks (LangChain, AutoGPT) dominate.

### 4.8 Pricing Models

| Model | Typical Approach |
|-------|-----------------|
| **Coze** | Freemium — free tier with limited usage, paid plans $15-30/month |
| **Dify** | Open-source (free self-hosted), Cloud free tier, Enterprise (custom) |
| **FastGPT** | Open-source (free self-hosted), Sealos cloud (pay-per-use) |
| **Baidu AgentBuilder** | Usage-based pricing on ERNIE tokens + platform fee |
| **Qwen-Agent** | Free open-source (bring your own model inference) |
| **BISHENG** | Open-source community edition, Enterprise (paid) |
| **AnythingLLM** | Open-source (free self-hosted), Cloud (paid $20-50/month) |

The open-source self-hosted model is dominant in China, likely driven by data sovereignty concerns. Cloud-only platforms (Coze, AgentBuilder) coexist with self-hosted options, serving different segments.

---

## 5. Model Ecosystem

Chinese agent frameworks are powered by a rich ecosystem of domestic LLMs, each with distinct strengths and weaknesses:

### 5.1 Qwen (Alibaba) — Qwen2.5 / Qwen3
| Attribute | Value |
|-----------|-------|
| **Stars** | 27,348 (Qwen2.5 repo) |
| **Weights** | Open (Apache-2.0) |
| **Parameters** | Up to 72B (dense), Mixture of Experts |
| **Strengths** | Excellent multilingual, strong tool use, long context |
| **Agent Use** | Primary target for Qwen-Agent, supported by Dify, FastGPT, DB-GPT |

### 5.2 ERNIE (Baidu)
| Attribute | Value |
|-----------|-------|
| **Weights** | Closed (API only) |
| **Strengths** | Deep Baidu search integration, Chinese web knowledge |
| **Agent Use** | Primary for AgentBuilder, limited support elsewhere |

### 5.3 GLM (Zhipu AI) — GLM-4
| Attribute | Value |
|-----------|-------|
| **Stars** | 7,070 (GLM-4 repo) |
| **Weight** | Open |
| **Strengths** | Strong function calling, bilingual, open research |
| **Agent Use** | Supported by Dify, MetaGPT, DB-GPT |

### 5.4 DeepSeek (DeepSeek AI) — DeepSeek-V3 / DeepSeek-R1
| Attribute | Value |
|-----------|-------|
| **Stars** | 103,846 (V3), 91,981 (R1) |
| **Weights** | Open (MIT) |
| **Strengths** | World-class reasoning (R1), strong math/code, very cost-effective |
| **Agent Use** | Widely supported — Dify, DB-GPT, FastGPT, AnythingLLM |
| **Note** | DeepSeek-R1's chain-of-thought reasoning makes it excellent for agent planning tasks |

### 5.5 Yi (01.AI) — Yi-34B
| Attribute | Value |
|-----------|-------|
| **Stars** | 7,822 |
| **Weights** | Open (Apache-2.0) |
| **Strengths** | Multilingual, strong on Chinese, efficient inference |
| **Agent Use** | Supported by Dify, DB-GPT |

### 5.6 Baichuan (Baichuan AI)
| Attribute | Value |
|-----------|-------|
| **Weights** | Open (earlier versions), closed for newer |
| **Strengths** | Strong Chinese language, good for content creation |
| **Agent Use** | Supported by Dify |

### 5.7 Moonshot (Kimi)
| Attribute | Value |
|-----------|-------|
| **Weights** | Closed (API only) |
| **Strengths** | Up to 2M token context window — market-leading long context |
| **Agent Use** | Best for frameworks needing long-document processing |

### 5.8 Doubao (ByteDance)
| Attribute | Value |
|-----------|-------|
| **Weights** | Closed (API only) |
| **Strengths** | Integrated with Coze, strong on creative tasks |
| **Agent Use** | Native to Coze, limited support elsewhere |

### 5.9 Minimax
| Attribute | Value |
|-----------|-------|
| **Weights** | Closed (API only) |
| **Strengths** | Strong creative writing, character-based narratives |

### 5.10 Spark/Xinghuo (iFlytek)
| Attribute | Value |
|-----------|-------|
| **Weights** | Closed (API only) |
| **Strengths** | Speech-first interface, education vertical |
| **Agent Use** | Supported by Dify |

### Model Selection Guide for Agents

| Agent Task | Best Chinese Model | Best Western Equivalent |
|-----------|-------------------|----------------------|
| Multi-step reasoning / planning | DeepSeek-R1 | OpenAI o3 / Claude Opus |
| Tool use / function calling | Qwen-2.5-72B | GPT-4o |
| Chinese-language RAG | Qwen-2.5 / GLM-4 | (Western models weaker here) |
| Code generation | DeepSeek-V3 / Qwen-2.5-Coder | Claude Opus |
| Long document processing | Kimi (2M context) | Gemini 2.0 Pro |
| Cost-effective general purpose | DeepSeek-V3 (cheapest $/token) | GPT-4o-mini |

---

## 6. Ecosystem Comparison: China vs West

The Chinese and Western agent framework landscapes have developed along parallel but distinct paths. Below is a detailed mapping:

### Core Agent Frameworks

| Function | China | West | Difference |
|----------|-------|------|------------|
| **Full-stack LLM app platform** | Dify (147k ⭐) | LangChain + Flowise | Dify is more opinionated and integrated; LangChain is more modular |
| **No-code agent builder** | Coze / Baidu AgentBuilder | OpenAI GPTs / Assistants API | Coze offers multi-platform publishing (WeChat, Douyin); GPTs are OpenAI-only |
| **Multi-agent simulation** | MetaGPT (69k ⭐) | CrewAI / AutoGen | MetaGPT is role-prescriptive (PM, Engineer, QA); CrewAI is flexible role definitions |
| **Database-focused agents** | DB-GPT (19k ⭐) | LangChain + LlamaIndex | DB-GPT is database-specific; LangChain/LlamaIndex are general purpose |
| **Model-specific agent SDK** | Qwen-Agent (17k ⭐) | OpenAI Assistants API / Anthropic MCP | Qwen-Agent is open-source and self-hostable; Assistants API is cloud-only |
| **Autonomous task completion** | XAgent (historical) | AutoGPT (185k ⭐) | AutoGPT is more mature and broadly used in both ecosystems |
| **LLMOps platform** | BISHENG (11k ⭐) | MLflow + LangFuse + Kubeflow | BISHENG is more LLM-specific; Western tools evolved from ML frameworks |
| **RAG/knowledge management** | FastGPT (29k ⭐) | RAGFlow / AnythingLLM (62k ⭐) | FastGPT has stronger workflow automation; AnythingLLM has broader model support |
| **Multi-agent collaboration** | Owl (20k ⭐) | CrewAI / AutoGen | Owl is newer but growing fast; both target real-world task automation |
| **Model hub + agents** | ModelScope (9k ⭐) + AgentScope (27k ⭐) | Hugging Face + smolagents | Hugging Face is larger ecosystem; ModelScope is more China-focused |

### Characteristic Differences

| Dimension | Chinese Frameworks | Western Frameworks |
|-----------|-------------------|-------------------|
| **Primary target users** | Enterprise + non-technical creators | Developers + researchers |
| **Deployment model** | Self-hosted dominant | Cloud + API dominant |
| **UI paradigm** | Visual workflows very common | Code-first predominant |
| **Model diversity** | Both Chinese and Western models | Primarily Western models (OpenAI, Anthropic, Google) |
| **Open source rate** | Very high (most frameworks are OSS) | High (LangChain, CrewAI, AutoGen are OSS) |
| **Platform integration** | WeChat, Douyin, Feishu, DingTalk | Slack, Discord, web |
| **Regulatory awareness** | Built-in compliance features | Minimal (assumes Western regulation) |
| **Avg. GitHub stars** | ~30-50k (Dify skews this higher) | ~40-70k (LangChain skews this higher) |
| **Documentation language** | Chinese > English | English > everything else |
| **Pricing model** | Freemium + OSS self-host | SaaS + OSS self-host |

### Who's Winning?

By GitHub stars alone, **Dify** (147k) is the most popular AI agent framework from any country, surpassing LangChain (~100k stars) in the West. However, this metric is imperfect — Chinese developers tend to star repositories more readily, and Dify's massive star count partly reflects its positioning as an "app platform" rather than a pure agent framework.

In terms of **production deployment**, Western frameworks (LangChain + ecosystem) likely power more real-world agent applications globally. But within China, Dify is the dominant platform for enterprise AI applications.

---

## 7. Regulatory & Compliance

China's AI regulatory environment is the most comprehensive globally and directly impacts how agent frameworks are designed, deployed, and operated.

### 7.1 Algorithm Registration

**Requirement:** All AI-powered algorithms must be registered with the Cyberspace Administration of China (CAC).

**Impact on Agent Frameworks:**
- Enterprise deployments must track which algorithms are used by their agent applications
- Frameworks like Dify and BISHENG provide audit trails that facilitate registration
- The registration process requires documentation of algorithm purpose, training data, and safety measures
- Non-compliance can result in fines up to ¥100,000 and service suspension

### 7.2 Deep Synthesis Regulation

**Requirement:** AI-generated content that could be confused with real content (text, images, audio, video) must be clearly labeled.

**Impact on Agent Frameworks:**
- Agent frameworks that generate content must implement watermarking
- Platforms like Coze include automatic content labeling
- Failure to label synthetic content can result in penalties under the Personal Information Protection Law

### 7.3 Content Filtering Obligations

**Requirement:** Generative AI services must ensure output content complies with "socialist core values" and does not contain prohibited content.

**Impact on Agent Frameworks:**
- Enterprise-focused frameworks (Dify, BISHENG, FastGPT) include configurable content filtering pipelines
- Cloud platforms (Coze, AgentBuilder) apply mandatory filtering that cannot be disabled
- Self-hosted frameworks allow custom filtering rules but operators bear compliance responsibility
- Prohibited content categories include: threats to national security, social instability, ethnic hatred, illegal activities

### 7.4 Data Sovereignty

**Requirement:** For regulated industries (finance, healthcare, government), AI data must remain within China's borders.

**Impact on Agent Frameworks:**
- Drives demand for self-hosted frameworks (Dify, FastGPT, DB-GPT, BISHENG)
- Cloud platforms must offer China-region hosting (Alibaba Cloud, Baidu Cloud, Tencent Cloud, Volcano Engine)
- International versions of Chinese platforms (e.g., Coze.com) separate Chinese user data into China region
- Western frameworks that don't offer China hosting are effectively blocked from regulated industries

### 7.5 "Socialist Core Values" Alignment

**Requirement:** AI outputs must align with "socialist core values" — a broad requirement covering prosperity, democracy, civility, harmony, freedom, equality, justice, rule of law, patriotism, dedication, integrity, and friendship.

**Impact on Agent Frameworks:**
- Model-level alignment is expected — models used with agent frameworks should be fine-tuned for value alignment
- Agent behavior should not generate content that contradicts these values
- Most Chinese agent frameworks rely on the underlying LLM's built-in alignment rather than adding framework-level value enforcement
- Some enterprise frameworks offer customizable "guardrails" for specific compliance needs

### 7.6 Impact on International Usage

Chinese agent frameworks face several barriers to international adoption:

| Barrier | Description | Affected Frameworks |
|---------|-------------|-------------------|
| **Documentation in Chinese** | Most frameworks have limited English docs | Dify (good English), all others (limited) |
| **Cultural assumptions** | WeChat/Feishu integration useless outside China | Coze, AgentBuilder |
| **Model availability** | ERNIE/GLM/Doubao APIs may not be accessible overseas | AgentBuilder, Coze (China version) |
| **Privacy concerns** | Chinese government data access concerns | All Chinese cloud-hosted platforms |
| **Regulatory compatibility** | GDPR vs Chinese regulations may conflict | Platforms handling EU user data |
| **Features designed for Chinese market** | Chinese language optimizations less useful globally | Most frameworks |

**Notable Exception:** Dify has successfully expanded internationally with strong English documentation, a global community (30k+ Discord), and cloud hosting in multiple regions. Its deep Western model support makes it useful globally.

---

## 8. Recommendations

### By Use Case

| Use Case | Recommended Framework(s) | Rationale |
|----------|------------------------|-----------|
| **Chinese enterprise full-stack AI** | **Dify** (self-hosted) | 147k stars, comprehensive features, best model diversity, enterprise edition with SSO |
| **Developers building on Chinese LLMs** | **Qwen-Agent** (for Qwen) / **ModelScope Agents** (for Alibaba Cloud) | Deepest integration; 16.6k stars; open-source; full tool-use stack |
| **Multi-agent code generation** | **MetaGPT** | 69k stars; unique role-based architecture; excellent for software development simulation |
| **No-code/low-code agent building** | **Coze** (for consumer/social) / **Baidu AgentBuilder** (for enterprise) | Coze: best publishing channels, Douyin/WeChat integration. AgentBuilder: best ERNIE integration |
| **RAG / knowledge management** | **FastGPT** (enterprise) / **AnythingLLM** (personal/local) | FastGPT: 29k stars, strong workflow. AnythingLLM: 62k stars, local-first |
| **Global use with Chinese models** | **Dify** | Best multi-model support, international cloud hosting, English docs, active global community |
| **Research / academia** | **MetaGPT** (multi-agent) / **Owl** (task automation) / **AgentScope** (distributed agents) | Strong academic provenance, published papers, active research communities |
| **Personal productivity** | **Coze** (cloud) / **AnythingLLM** (local) | Coze: free tier, templates, easy. AnythingLLM: local, no API costs |
| **Database / data agent** | **DB-GPT** | 19k stars; best SQL/database integration; supports enterprise data workflows |
| **LLMOps / MLOps for LLMs** | **BISHENG** | 11k stars; comprehensive lifecycle management; fine-tuning, evaluation, monitoring |

### By Framework Maturity & When to Adopt

| Tier | Framework | Maturity Level | When to Adopt |
|------|-----------|---------------|---------------|
| **Production-ready** | Dify, FastGPT, DB-GPT, AnythingLLM, BISHENG | High — active development, large communities, commercial backing | Now |
| **Production-ready (cloud)** | Coze, Baidu AgentBuilder | High — major company backing, stable APIs | Now |
| **Developer-ready** | Qwen-Agent, MetaGPT, AgentScope | Medium-High — active but smaller communities | For custom development |
| **Research / experimental** | Owl, CAMEL, XAgent | Low-Medium — fast-moving, may break | For research projects |
| **Too early** | Agently (3 stars) | Very low | Not recommended |

### Decision Tree

```
What's your primary need?
├── Enterprise AI application
│   ├── Need self-hosted → Dify (best overall) or FastGPT (RAG-focused)
│   ├── Need cloud → Coze (multi-model) or AgentBuilder (Baidu ecosystem)
│   └── Need LLMOps → BISHENG
├── Developer building agents
│   ├── Using Qwen → Qwen-Agent
│   ├── Need multi-model → Dify SDK or DB-GPT
│   └── Building multi-agent sims → MetaGPT
├── No-code agent creation
│   ├── For social/consumer → Coze (Douyin, WeChat publishing)
│   └── For enterprise → Baidu AgentBuilder
├── Knowledge management / RAG
│   ├── Enterprise → FastGPT (workflow automation)
│   ├── Local/personal → AnythingLLM
│   └── Data-intensive → DB-GPT
└── Research / experimentation
    ├── Multi-agent theory → CAMEL
    ├── Task automation → Owl
    └── Code generation → MetaGPT
```

---

## 9. References

### Official Documentation & Websites

| Framework | URL |
|-----------|-----|
| Dify | https://dify.ai |
| Coze | https://www.coze.com |
| Coze (China) | https://www.coze.cn |
| MetaGPT | https://github.com/geekan/MetaGPT |
| Qwen-Agent | https://github.com/QwenLM/Qwen-Agent |
| FastGPT | https://github.com/labring/FastGPT |
| DB-GPT | https://github.com/csunny/DB-GPT |
| AnythingLLM | https://github.com/Mintplex-Labs/anything-llm |
| BISHENG | https://github.com/dataelement/bisheng |
| CAMEL | https://github.com/camel-ai/camel |
| Owl | https://github.com/camel-ai/owl |
| AgentScope | https://github.com/modelscope/agentscope |
| ModelScope | https://modelscope.cn |
| AgentGPT | https://github.com/reworkd/AgentGPT |
| AutoGPT | https://github.com/Significant-Gravitas/AutoGPT |
| Baidu AgentBuilder | https://cloud.baidu.com/product/agentbuilder.html |
| Qwen Models | https://github.com/QwenLM/Qwen |
| DeepSeek-V3 | https://github.com/deepseek-ai/DeepSeek-V3 |
| DeepSeek-R1 | https://github.com/deepseek-ai/DeepSeek-R1 |
| ChatGLM-6B | https://github.com/THUDM/ChatGLM-6B |
| GLM-4 | https://github.com/THUDM/GLM-4 |
| AgentBench | https://github.com/THUDM/AgentBench |

### Key GitHub Statistics (as of July 2026)

| Rank | Framework | Stars | Language | License |
|------|-----------|-------|----------|---------|
| 1 | AutoGPT (Western) | 185,220 | Python | — |
| 2 | Dify | 147,171 | TypeScript | — |
| 3 | DeepSeek-V3 (Model) | 103,846 | Python | — |
| 4 | DeepSeek-R1 (Model) | 91,981 | — | — |
| 5 | MetaGPT | 69,129 | Python | MIT |
| 6 | AnythingLLM | 62,358 | JavaScript | MIT |
| 7 | ChatGLM-6B (Model) | 41,033 | Python | — |
| 8 | AgentGPT | ~36,234 | TypeScript | — |
| 9 | AgentScope | ~27,343 | Python | Apache-2.0 |
| 10 | Qwen2.5 (Model) | 27,348 | Python | — |
| 11 | MiniCPM-o (Model) | 25,759 | Python | — |
| 12 | FastGPT | 28,744 | TypeScript | — |
| 13 | Qwen (Model) | 21,367 | Python | — |
| 14 | DB-GPT | 19,309 | Python | MIT |
| 15 | Owl | 19,906 | Python | — |
| 16 | CAMEL | 17,305 | Python | Apache-2.0 |
| 17 | Qwen-Agent | 16,636 | Python | Apache-2.0 |
| 18 | ChatGLM3 (Model) | 13,678 | Python | — |
| 19 | BISHENG | 11,480 | TypeScript | Apache-2.0 |
| 20 | MiniCPM (Model) | 9,530 | Python | — |
| 21 | ModelScope | 8,994 | Python | Apache-2.0 |
| 22 | Yi (Model) | 7,822 | Python | Apache-2.0 |
| 23 | GLM-130B (Model) | 7,660 | Python | — |
| 24 | GLM-4 (Model) | 7,070 | Python | — |
| 25 | AgentBench | 3,535 | Python | — |

### Regulatory References

- Cyberspace Administration of China. "Interim Measures for the Management of Generative AI Services." August 2023. https://www.cac.gov.cn
- State Council. "New Generation Artificial Intelligence Development Plan." July 2017. https://www.gov.cn
- CAC. "Deep Synthesis Internet Information Service Management Regulations." January 2023.
- CAC. "Internet Information Service Algorithm Recommendation Management Regulations." March 2022.

---

## Appendix A: Quick Reference — Best Framework for Each Scenario

| Scenario | Framework |
|----------|-----------|
| "I need to build a customer service chatbot for my Chinese enterprise" | **Dify** (self-hosted, full RAG) |
| "I want to create a WeChat bot that answers questions about my products" | **Coze** (publish to WeChat mini-program) |
| "I'm a developer building an agent using Qwen" | **Qwen-Agent** |
| "I need to simulate a software team to generate code" | **MetaGPT** |
| "I want to query my company databases using natural language" | **DB-GPT** |
| "I need a knowledge base QA system for internal documents" | **FastGPT** |
| "I want a local-first RAG system with no cloud dependency" | **AnythingLLM** |
| "I need to fine-tune and deploy LLMs for my enterprise" | **BISHENG** |
| "I'm researching multi-agent collaboration" | **Owl** or **CAMEL** |
| "I want a no-code tool to create social media bots" | **Coze** |
| "I'm a Baidu Cloud customer building ERNIE agents" | **Baidu AgentBuilder** |
| "I need a platform that works with both OpenAI and DeepSeek" | **Dify** |
| "I want to build agents programmatically with Python" | **Qwen-Agent** or **AgentScope** |

---

## Appendix B: Ecosystem Timeline

| Date | Event |
|------|-------|
| July 2017 | China's State Council publishes "New Generation AI Development Plan" |
| March 2022 | Algorithm registration requirements take effect |
| March 2023 | AutoGPT released; CAMEL released |
| April 2023 | Dify created; DB-GPT created |
| June 2023 | MetaGPT released; AnythingLLM created |
| July 2023 | AgentBench released (THUDM) |
| August 2023 | BISHENG created; Qwen-Agent created |
| August 2023 | China's Generative AI regulations take effect |
| September 2023 | Qwen-Agent released |
| February 2024 | Coze gains widespread adoption |
| January 2025 | DeepSeek-R1 released — global recognition |
| March 2025 | Owl released (CAMEL-AI) |
| July 2026 | Dify reaches 147k stars; Chinese agent ecosystem fully mature |

---

*This analysis was compiled in July 2026. GitHub star counts and framework statuses are current as of that date. The landscape evolves rapidly — verify specifics before making deployment decisions.*
