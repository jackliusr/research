# LLM Agent Use Cases: A Comprehensive Landscape

> Last updated: July 2026
> A structured survey of how LLM-powered agents are deployed across industries — with real companies, products, metrics, and evidence.

---

## 1. What Are LLM Agents?

### Definition
LLM agents are AI systems that use large language models as their core reasoning engine to **perceive** their environment, **plan** sequences of actions, **use tools** (APIs, databases, files, GUIs), **maintain memory** across turns, and **execute multi-step workflows** autonomously or semi-autonomously toward a goal.

### Core Capabilities
| Capability | Description | Techniques |
|---|---|---|
| **Planning** | Decompose goals into sub-tasks, reason about order and dependencies | ReAct, Chain-of-Thought, Tree-of-Thought, Plan-and-Solve |
| **Tool Use** | Call external functions, APIs, databases, and services | Function calling, MCP tools, code execution |
| **Memory** | Retain context across interactions, retrieve relevant past knowledge | Short-term (context window), long-term (vector DB), RAG |
| **Multi-step Execution** | Iterate, self-correct, verify results, and adapt to feedback | Reflection, loop-until-solved, eval-driven |
| **Multi-agent Coordination** | Specialized agents collaborate, delegate, and hand off | Agent teams, orchestrator-workers, debate/consensus |

### Distinction: Agents vs. Chatbots vs. RAG
- **Chatbot**: Answers questions from a static knowledge base. No tool use, no action.
- **RAG System**: Retrieves documents and synthesizes answers. Reads but does not act.
- **LLM Agent**: Takes actions — writes files, calls APIs, creates PRs, books meetings, executes code — not just answers questions. An agent has agency.

### Key Frameworks (2025–2026)
| Framework | Creator | Paradigm | Stars (GitHub) |
|---|---|---|---|
| **LangChain / LangGraph** | LangChain | Graph-based agent orchestration, tool calling, rich ecosystem | ~100k+ |
| **CrewAI** | CrewAI | Multi-agent role-based delegation, lightweight | ~25k+ |
| **AutoGen** | Microsoft Research | Conversational multi-agent, enterprise-grade | ~40k+ |
| **OpenAI Agents SDK** | OpenAI | Lightweight, Python-native, built-in guardrails | ~15k+ |
| **Anthropic MCP** | Anthropic | Protocol: tools/context for agents (Linux Foundation) | Open standard |
| **Google ADK / GenKit** | Google | Full-stack agent dev + A2A protocol | Growing |
| **Semantic Kernel** | Microsoft | Enterprise .NET/Python agents with Azure AI | ~25k+ |
| **Dify / RAGFlow** | Community | Visual agent builder, RAG-first | ~50k+ / ~25k+ |

---

## 2. Software Engineering & Development

### 2.1 Code Generation & Review

LLM coding agents are the most mature agent category, with massive adoption and measurable productivity gains.

**Key Products & Evidence:**

- **Cursor**: AI-first IDE with agentic code editing. Supports multi-file refactoring, diff-aware suggestions, and inline code generation. Used by hundreds of thousands of developers. In 2025, Cursor was valued at ~$10B and reached $100M+ ARR. Its agent mode can plan and execute changes across multiple files from a single prompt.

- **GitHub Copilot / Copilot Workspace**: Copilot agent mode (2025) extends beyond autocomplete — it can fix issues, generate PR descriptions, and run terminal commands. Copilot Workspace (GA 2025) ingests an entire issue description, produces a plan, generates code across multiple files, and opens a PR. GitHub reports that Copilot accelerates developers by up to 55% on repetitive tasks.

- **Claude Code (Anthropic)**: Terminal-based agent that reads/writes files, runs shells, creates git commits, and manages PRs. Used extensively at companies like Canva, GitLab, and Bloomberg. Key feature: can self-correct after running tests or linters.

- **Codex CLI (OpenAI)**: Terminal agent with sandboxed code execution, persistent sessions, and a built-in evaluation loop. Supports multi-file edits, git integration, and iterative debugging.

- **Devin (Cognition Labs)**: First fully autonomous AI software engineer. Raised $1B+ at a $26B valuation (June 2026), hit $492M ARR with 1,230% YoY growth. Independently resolves GitHub issues, deploys apps, fixes bugs, and writes documentation. Used by thousands of companies including some of the largest enterprises.

- **SWE-bench Results**: SWE-bench Verified (500 human-filtered GitHub issues) is the standard. By mid-2026, top scores reach ~50-70% on the verified subset (Claude Fable 5 leading at 95%). SWE-bench Multilingual covers 300 tasks across 9 languages. The rapid improvement from ~14% (Devin early 2024) to ~70%+ shows exponential agent capability growth.

**Use Cases**: Bug fixing, feature implementation, code review, test generation, refactoring, documentation generation, legacy migration, dependency upgrades.

### 2.2 DevOps & Infrastructure

DevOps agents monitor, diagnose, and remediate infrastructure issues autonomously.

**Key Products & Evidence:**

- **Kubiya**: DevOps agent that translates natural language requests into Terraform, Kubernetes, and CI/CD actions. Used by enterprises to self-serve infrastructure provisioning. Integrates with PagerDuty, Jira, Slack. Customers report 60-80% reduction in ops ticket handling time.

- **o11y (observability agent)**: AI agent that ingests logs, metrics, and traces, triages alerts, identifies root causes, and suggests or applies fixes. Built on OpenTelemetry data. Agents can correlate across services, detect anomalies, and auto-remediate common issues.

- **FireHydrant**: Incident response platform with an AI agent that triages alerts, assembles the right responders, creates war rooms, and generates post-mortems. Integrates with PagerDuty, Opsgenie, and status pages.

- **Incident Response Pattern**: Agent ingests alert → searches logs/metrics → identifies contributing change (git blame, deployment history) → proposes rollback/fix → notifies on-call → drafts post-mortem.

- **CI/CD Agents**: Agents optimize pipeline speed by parallelizing test execution, caching dependencies, retrying flaky tests intelligently, and suggesting concurrency and resource tuning. Tools like Buildkite and CircleCI embed agentic optimization.

### 2.3 Database & Data Engineering

**Key Products & Evidence:**

- **TextQL**: "AI data analyst" that lets business users ask natural language questions and get SQL queries, visualizations, and explanations. Agents plan multi-step queries, join across tables, and handle edge cases (NULL handling, type mismatches).

- **dbt AI (dbt Labs)**: Agentic features in dbt Cloud include natural-language-to-dbt-model generation, automated data testing, lineage explanation, and documentation generation. Agents help data engineers debug failing models by examining upstream dependencies.

- **Databricks AI / Genie**: Databricks' AI assistant (Genie) enables conversational analytics — users ask business questions and the agent generates SQL, runs it against the lakehouse, and visualizes results. Databricks reports that Genie handles ~40% of ad-hoc analytics queries autonomously.

- **Text-to-SQL Agents**: LLM agents with self-correction loops (generate SQL → execute → analyze error → retry) achieve 75-85% accuracy on benchmarks like Bird and Spider. Enterprise deployments add schema linking, business glossary RAG, and human-in-the-loop for production queries.

**Use Cases**: Ad-hoc analytics, schema migration validation, data quality monitoring, pipeline debugging, report generation, data cataloging.

---

## 3. Customer Service & Support

### 3.1 Customer Support Agents

The most widely deployed enterprise agent category — over 50% of large enterprises have piloted or deployed AI support agents as of 2026.

**Key Products & Evidence:**

- **Zendesk AI**: Handles Tier 1-2 support across email, chat, and social. Zendesk reports 30-50% ticket deflection rates, with AI agents resolving common issues (password resets, order status, billing inquiries) in under 30 seconds. Their intelligent triage routes complex issues to human agents with full context synthesized.

- **Intercom Fin**: Fin 2 automates ~51% of conversations end-to-end (industry-reported). Resolution rates vary by complexity — simple FAQ can hit 80%+. The agent uses RAG over the company's knowledge base, tool calling (CRM, order system), and human handoff when confidence is low. Integrates with Shopify, Salesforce, and HubSpot.

- **Ada.cx**: Ada's AI agent platform is used by companies like Telus, Verizon, and Shopify. Reports up to 80% automation rates for Tier 1 queries in telecom. Ada's agents handle multi-channel support (web, SMS, WhatsApp, Messenger) with context persistence across channels.

- **Forethought**: AI agent for support with "Solve" (automated resolution), "Triage" (intent classification + routing), and "Assist" (agent copilot). Customers include Instacart, Medium, and Upwork. Forethought reports 35-50% auto-resolution rates and 3-5× improvement in agent productivity.

**Impact Metrics (Aggregate)**:
- 30-50% ticket deflection/reduction
- 2-5× faster resolution times
- 50-70% cost reduction per interaction
- 24/7 availability with sub-minute first response
- 80-90% CSAT scores (parity or better vs. human agents for Tier 1)

**Implementation Pattern**: RAG over knowledge base → tool calling (order system, CRM, password reset API) → confidence check → human handoff with full context → feedback loop for continuous improvement.

### 3.2 Sales & Marketing Agents

**Key Products & Evidence:**

- **11x.ai**: Fully autonomous AI sales development representatives (SDRs). Agents handle outbound prospecting — researching leads, writing personalized emails, handling replies, scheduling meetings. Reports 2-5× more qualified meetings booked per human SDR. Raised significant funding at a $500M+ valuation in 2025.

- **Outreach.io AI**: Agentic features for sales engagement — automated sequence optimization, lead scoring, reply intent detection, and follow-up personalization. Customers report 20-30% increase in reply rates from AI-optimized sequences.

- **Gong AI**: Revenue intelligence platform with agentic capabilities — automatically logs calls, generates summaries, extracts insights, and recommends next actions. The Gong agent can draft follow-up emails, update CRM fields, and surface competitive intelligence from call transcripts.

- **Salesforce Einstein**: Einstein AI agents handle lead qualification, opportunity scoring, and workflow automation. The Einstein Trust Layer includes data masking, toxicity detection, and audit trails. Salesforce reports that customers using Einstein see 15-30% improvement in sales productivity.

- **Apollo.io**: AI-powered sales engagement with agentic lead enrichment — agents research company news, tech stack changes, funding events, and personnel moves to keep CRM data fresh.

**Impact Metrics**:
- 2-5× more leads handled per SDR
- 10-20% increase in conversion rates
- 40-60% reduction in manual CRM data entry
- 3-5× personalized outreach sequences generated

---

## 4. Healthcare & Life Sciences

### 4.1 Clinical Decision Support

**Key Products & Evidence:**

- **Viz.ai**: AI-powered care coordination platform for stroke, pulmonary embolism, and other time-sensitive conditions. Viz analyzes medical imaging, flags critical findings, and autonomously alerts the specialist team. FDA-cleared (510(k)), deployed in 2,000+ hospitals. The agent triages scans in minutes vs. hours manually.

- **Ada Health**: AI symptom assessment and care navigation agent. Used by 13M+ users globally. The agent conducts a conversational symptom analysis using a medical knowledge graph (not just LLM), generates differential diagnoses, and recommends appropriate care pathways. CE-marked as a Class I/IIa medical device.

- **Corti**: AI agent that listens to patient-clinician conversations in real-time and provides decision support. Used in emergency rooms and primary care. Corti's agent suggests differential diagnoses, flags red flags, and generates clinical notes automatically. Handles 10M+ encounters per month. Customers report 30% reduction in documentation time.

- **Babylon Health (eMed)**: AI triage agent that collects patient symptoms and history conversationally, then recommends appropriate care. Used by the UK NHS as part of Babylon's GP at Hand service.

**Regulatory Context**: Many clinical decision support agents require FDA 510(k) clearance or CE marking as medical devices. HIPAA compliance (US) and GDPR (EU) are mandatory. LLM-specific concerns include hallucination in clinical contexts and the need for transparent reasoning.

### 4.2 Medical Coding & Billing

**Key Products & Evidence:**

- **SmarterDx**: AI agent that reviews inpatient charts for missed diagnoses and coding errors. Scans clinical documentation against ICD-10 coding guidelines, identifies discrepancies, and generates compliant queries for physicians. Deployed at 350+ hospitals. Reports 5-7% revenue uplift from captured diagnoses.

- **Notable Health**: AI agent platform for prior authorization, billing, and clinical workflows. The agent automates the prior authorization process — extracts medical necessity from clinical notes, matches against payer criteria, fills out forms, and submits. Notable reports 70-80% reduction in manual prior auth work.

- **CodaMetrix**: AI-powered medical coding agent that reads clinical notes and generates ICD-10, CPT, and HCPCS codes automatically. Trained on 100M+ annotated medical records. Customers report 95%+ coding accuracy and 3-5× throughput improvement.

### 4.3 Drug Discovery

**Key Products & Evidence:**

- **Insilico Medicine**: AI drug discovery platform with generative chemistry agents. Insilico's AI agents design novel molecules, predict ADMET properties, optimize synthesis routes, and propose clinical trial designs. Lead candidate for idiopathic pulmonary fibrosis (INS018_055) completed Phase II trials — the first AI-discovered drug to reach this stage.

- **Recursion Pharmaceuticals**: AI platform that combines cellular imaging, genomics, and LLM-based literature agents for drug-target discovery. Recursion's agents mine scientific literature, match compounds to disease phenotypes, and prioritize targets. The company has 40+ programs in pipeline, 5+ in clinical trials.

- **BenevolentAI**: AI drug discovery agent that builds knowledge graphs from biomedical literature, then reasons over them to find novel drug-target-disease relationships. Discovered a candidate for ALS that progressed to Phase II. Benevolent's agents can generate novel hypotheses unsuggested by existing research.

- **Clinical Trial Agents**: AI agents optimize trial protocols — predict enrollment timelines, identify suitable trial sites, draft patient-facing documents, and monitor adverse events from medical literature. Microsoft's Project Genome applies LLM agents to clinical trial optimization.

---

## 5. Finance & Banking

### 5.1 Financial Analysis

**Key Products & Evidence:**

- **Bloomberg Terminal AI (ASKB)**: Bloomberg launched an agentic AI interface (ASKB) in early 2026, directly embedded in the Terminal. Agents can analyze earnings transcripts, generate comp tables, flag key metrics changes, and summarize market-moving events. Bloomberg's agents use proprietary financial data + LLM reasoning for analyst-grade outputs.

- **AlphaSense**: AI market intelligence platform with agentic search and summarization. Agents search 10,000+ sources (SEC filings, earnings calls, expert transcripts, news, broker research), synthesize findings, and generate reports. Used by 4,000+ enterprise clients including 75% of the S&P 100.

- **Koyfin**: AI-powered financial analytics platform with an agent interface for portfolio analysis, sector screening, and risk decomposition. Agents can run custom financial models, generate visualizations, and alert on portfolio concentration risks.

- **BloombergGPT / FinBERT**: Domain-specific financial LLMs fine-tuned on regulatory filings, earnings calls, and financial news. These underpin agentic workflows for sentiment analysis, disclosure matching, and risk factor extraction.

### 5.2 Compliance & Fraud Detection

**Key Products & Evidence:**

- **ComplyAdvantage**: AI agent platform for AML/CTF compliance. Agents monitor transactions in real-time, generate suspicious activity reports (SARs), reduce false positives by 70%+, and screen customers against sanctions and PEP lists. Their AI agents learn from investigator feedback to improve accuracy.

- **Chainalysis**: Blockchain analytics with AI agents that trace crypto transactions, identify suspicious wallet clusters, and generate investigative reports. Agents correlate on-chain activity with off-chain identity data. Used by 1,000+ government agencies and financial institutions.

- **Ayasdi (SymphonyAI)**: AI for financial crime detection. Agents apply geometric machine learning to transaction data, identifying anomalous patterns that rules-based systems miss. Reports 50-80% reduction in false positives for AML alerts.

- **Regulatory Filing Agents**: Agents prepare regulatory filings (SEC 10-K/Q, 8-K, FERC, etc.) by extracting data from internal systems, generating narrative sections (MD&A, risk factors), and flagging compliance gaps. Law firms and in-house teams use agents like Harvey for regulatory gap analysis.

### 5.3 Personalized Banking

**Key Products & Evidence:**

- **Cleo**: AI financial assistant used by 4M+ users in the UK and US. The agent analyzes spending patterns, flags waste (subscriptions, fees), negotiates bills, and provides budgeting advice. Cleo's agent uses LLM reasoning to have personality-driven, motivating conversations — users interact with it like a friend, not a dashboard.

- **Plum**: AI savings and investment agent popular in Europe and Asia. The agent analyzes income/expenses, auto-saves optimal amounts, invests in curated funds, and provides market insights conversationally. 10M+ users.

- **Klarna AI**: Klarna's AI shopping and finance agent handles buying, payments, and refunds for customers via conversational interface. After layoffs / attrition, Klarna's AI agent handles the work of 700+ customer service agents. The AI resolves customer issues in 1-2 minutes vs. 11 minutes for human agents.

- **JPMorgan LOXM**: AI agent for equity trading execution. LOXM trades optimally across venues to minimize market impact — learns from past trades to improve execution. Deployed on JPMorgan's electronic trading desk, handling millions of shares daily. JPMorgan has over 400 AI use cases in production across the bank.

**Use Cases**: Financial advisory, spending analysis, budgeting, bill negotiation, investment recommendations, loan pre-qualification.

---

## 6. Legal

### 6.1 Document Review & Analysis

**Key Products & Evidence:**

- **Harvey**: The leading generative AI platform for law firms. Built on OpenAI, fine-tuned on legal data. Harvey agents handle contract review, due diligence, compliance analysis, litigation strategy, and regulatory research. By early 2026, Harvey surpassed $50M ARR, 500+ customers, 42% of Am Law 100 firms, and a $5B valuation. Customers report 40-70% time savings on document review tasks.

- **Ironclad AI**: AI agent for contract lifecycle management. Automates contract review, redlining, clause extraction, and obligation tracking. The agent learns each company's playbooks and approval workflows. Used by 1,800+ legal teams. Ironclad reports 300%+ ROI for enterprise customers within 6 months.

- **Evisort (Walmart-owned)**: AI agent for contract intelligence. Extracts key terms, obligations, dates, and risks from contracts. Evisort's agents monitor contract compliance (auto-renewals, expirations, SLA tracking). Used by Walmart, Microsoft, and Google.

- **LawGeex**: AI contract review agent that benchmarks against expert human lawyers. LawGeex proved AI can match or exceed human accuracy on standard contract review tasks (non-disclosure agreements, procurement contracts). Their agent highlights deviations from playbook, suggests alternative language, and flags risk.

### 6.2 Legal Research & Drafting

**Key Products & Evidence:**

- **Casetext (CoCounsel, acquired by Thomson Reuters)**: AI legal assistant that searches case law, summarizes rulings, and drafts memos. CoCounsel was the first generative AI tool certified by the American Bar Association for legal use. Used by 10,000+ law firms. Agents can analyze deposition transcripts, recommend questions, and generate chronologies.

- **LexisNexis AI (Lexis+ AI)**: Conversational legal research agent with cites-to-authority. The agent searches 200M+ documents (cases, statutes, regulations, law reviews), generates memoranda, and provides validated citations. Used by 80% of Am Law 100 firms.

- **Thomson Reuters Westlaw AI (Westlaw Precision + CoCounsel)**: AI agents for research, drafting, and analysis. The Ask Practical Law agent answers complex legal questions with references to Practical Law resources. Westlaw's AI agents incorporate editorial analysis with generative drafting.

- **Patent Prior Art Search**: Specialized agents from Anaqua, PatSnap, and Google Patents use LLMs to understand patent claims and search prior art across millions of patents and non-patent literature. Reducing search time from days to minutes.

### 6.3 E-Discovery

**Key Products & Evidence:**

- **E-discovery Agents**: Major platforms — Relativity, Everlaw, Disco, Reveal — embed AI agents for document classification (responsive, privileged, relevant), email thread analysis, key document identification, timeline reconstruction, and privilege log generation.

- **Impact**: AI-powered e-discovery reduces document review costs by 40-70% (from ~$1-5/page to fractions of a cent). Agents can classify millions of documents in hours vs. weeks for human review teams. Accuracy on standard TAR (technology-assisted review) workflows exceeds 90% recall at 95%+ precision.

- **Pattern**: Agent clusters documents by topic → identifies key custodians → reconstructs communication timelines → surfaces documents responsive to discovery requests → flags privilege → generates privilege logs → produces in native format.

---

## 7. Education & E-Learning

### 7.1 Personalized Tutoring

**Key Products & Evidence:**

- **Khanmigo (Khan Academy)**: AI tutor powered by GPT-4, designed to teach through Socratic questioning (guides, doesn't give answers). By 2025-2026, Khanmigo reached 1.5M students across 130 countries. After one academic year, randomized controlled trials showed 12% improvement in math scores and saved teachers ~37 minutes per week on administrative tasks. 50M+ student interactions logged. The agent adapts to each student's level, provides hints, and flags misconceptions to teachers.

- **Duolingo Max**: Subscription tier with AI-powered features: Roleplay (conversational practice with AI characters), Explain My Answer (agent analyzes user's response and provides targeted grammar/vocabulary feedback), and Adventure (interactive story quests). Duolingo's Birdbrain AI models track student knowledge state to optimize review scheduling.

- **Tutor.ai / Synthesis**: AI tutoring agents with Socratic method, step-by-step guidance, and multi-modal explanations (text, diagrams, math notation). Synthesis (from Khan Academy + Tesla/Alpha School co-founders) runs virtual group tutoring sessions led by AI agents.

- **Pattern**: Assess student knowledge → identify gaps → adapt difficulty → provide scaffolded hints → check for understanding → spiral back to reinforce.

### 7.2 Content Generation

- **Curriculum Design**: Agents generate course outlines, learning objectives, lesson plans aligned to standards (Common Core, IB, etc.), and weekly pacing guides. Platforms like TeachFx, Eduaide.Ai, and MagicSchool.ai serve 1M+ teachers with AI lesson planning.

- **Assessment Generation**: Agents write multiple-choice questions, coding challenges, essay prompts, and rubrics. They can generate distractors (wrong answers) for MCQs, create variations for anti-cheating, and align difficulty to Bloom's Taxonomy.

- **Automated Grading & Feedback**: LLM grading agents evaluate essays, code submissions, and math solutions. While not as reliable as expert humans for nuanced grading, they handle first-pass grading for standardized assessments. ETS (Educational Testing Service) uses AI agents for preliminary scoring on some exam sections.

- **Example Generation**: Agents generate worked examples, counterexamples, practice problems with solutions, and explanatory text for teacher-identified concepts students commonly struggle with.

---

## 8. Content Creation & Media

### 8.1 Writing & Editing

**Key Products & Evidence:**

- **Jasper**: AI content platform with agentic workflows for long-form content, ad copy, email sequences, and blog posts. Agents plan article structure, research topics via web search, draft sections, and optimize for SEO/Tone/Style. 100K+ paying customers. Enterprise features include brand voice customization, fact-checking agents, and editorial review workflows.

- **Copy.ai**: AI content agent platform used by 15M+ users. Named "Workflow" agents automate multi-step content creation: brief → outline → draft → edit → publish. Integrates with CMS platforms (WordPress, HubSpot, Webflow). Copy.ai reports 3-5× content output with AI agent assistance.

- **Writer.com**: Enterprise AI writing platform with Palmyra models (Writer's own LLMs). Agents handle brand-aligned content, generate marketing copy, edit for consistency, and enforce style guides. Used by 500+ enterprises including Spotify, Intel, and L'Oréal. Writer's agentic graph engine supports multi-step content workflows.

- **SEO Agents**: Platforms like MarketMuse, Clearscope, and Frase use AI agents to research keywords, analyze top-ranking content, generate outlines, and optimize drafts for search intent.

### 8.2 Video & Multimedia

**Key Products & Evidence:**

- **Runway AI**: AI video generation with agentic features — Gen-3 Alpha and Gen-4 models enable text-to-video, image-to-video, video-to-video, and multi-motion brush. Agents can extend clips, remove objects, generate green-screen mattes, and restyle scenes. Used in professional film (Late Night with the Devil used Runway for AI-generated transitions).

- **Synthesia**: AI video generation platform for avatars. Agents generate scripts from text prompts, select avatars, add background music, produce voiceovers in 140+ languages, and publish. 60K+ business customers. The Synthesia agent can produce a studio-quality video from a script URL in minutes.

- **Descript**: AI-powered video/audio editing platform. Agents transcribe audio (Whisper-based), allow text-based editing (edit the transcript → edit the video), generate voice clones (overdub), remove filler words, and composite multi-track recordings. Descript's AI agent can repurpose a long podcast into clips, transcripts, show notes, and social posts.

- **Social Media Repurposing Agents**: Agents take long-form content (webinar, podcast, article) and generate 5-10 social posts optimized per platform (Twitter/X, LinkedIn, TikTok, Instagram). Opus Clip, Veed.io, and Repurpose.io build agents that identify highlight moments and cut clips automatically.

### 8.3 Research & Analysis

**Key Products & Evidence:**

- **Perplexity (Pro/Agent Mode)**: AI research assistant that iteratively searches the web, reads sources, cross-references, and generates comprehensive answers with inline citations. Pro Search agents can be directed to follow specific methodologies (e.g., "find conflicting studies on this topic"). 15M+ active users. The agent maintains a knowledge base across a session, refining its understanding.

- **Elicit**: AI research agent focused on academic literature. Asks research questions → searches 200M+ papers → extracts key data (sample sizes, findings, methodologies) → synthesizes into tables and summaries. Used by 2M+ researchers. Elicit agents can identify trends across papers, find contradictory findings, and suggest avenues for further study.

- **Scite**: AI agent that shows how papers cite each other (supporting, contradicting, mentioning). Scite's Smart Citations help researchers evaluate claim reliability. The Assistant agent answers research questions with citation evidence.

- **Competitive Analysis Agents**: Agents that monitor company filings, job postings, product launches, press releases, and customer reviews to produce competitive intelligence reports. Platforms like Crayon, Klue, and AlphaSense build agentic competitive monitoring.

---

## 9. E-commerce & Retail

### 9.1 Shopping Assistants

**Key Products & Evidence:**

- **Shopify Sidekick**: AI commerce agent embedded in Shopify. Merchants can ask Sidekick to analyze sales data, create discount campaigns, write product descriptions, generate email marketing, set up abandoned cart sequences, and build custom reports. Sidekick agents have tool access to Shopify's entire admin API. 2M+ merchants on Shopify.

- **Klarna AI**: Shopping agent that helps users search products, compare prices, get personalized recommendations, and complete purchases. Klarna's AI also handles customer service (returns, refunds, order tracking). The agent processed 2M+ conversations monthly by 2025, with 70%+ handled fully autonomously.

- **Pinterest AI**: AI-powered visual discovery and shopping agent. Pinterest Lens lets users photograph an object in the real world → agent finds similar products, style ideas, and purchase links. Pinterest's AI agent recommends products based on board analysis, past pins, and current trends. 500M+ monthly active users.

- **Agent Commerce Pattern**: Context-aware product search → personalized recommendations → price comparison → virtual try-on → purchase → order tracking → post-purchase support. All in a single conversational flow.

### 9.2 Inventory & Supply Chain

**Key Products & Evidence:**

- **Demand Forecasting Agents**: LLMs combined with time-series models (Prophet, DeepAR, N-BEATS) agents that ingest historical sales, promotions calendars, weather data, social media trends, and macroeconomic indicators. Agents explain *why* forecasts changed (e.g., "T-shirt demand up 22% due to viral TikTok trend"). Tools: Blue Yonder, o9 Solutions, ToolsGroup, RELEX.

- **Supply Chain Disruption Agents**: Agents monitor real-time signals — port congestion data, weather events, supplier financial health, geopolitical news, labor disputes — and alert teams with impact analysis. They can recommend alternative suppliers, reroute shipments, and pre-order constrained materials. Tools: Resilinc, Everstream Analytics, Altana AI.

- **Warehouse Optimization Agents**: AI agents optimize pick paths, bin assignments, and labor allocation. Agents in fulfillment centers (Amazon, Walmart, DHL) coordinate robot fleets, assign pickers dynamically, and predict bottlenecks. Amazon's AI agents manage inventory placement across its global fulfillment network, optimizing for next-day delivery.

- **McKinsey Estimate**: AI in supply chain management could generate $1.2-2T in annual value globally. Key drivers: demand forecasting accuracy (+20-50%), inventory reduction (10-40%), logistics cost savings (15-30%).

---

## 10. Scientific Research

**Key Products & Evidence:**

- **Microsoft MatterGen**: AI agent for materials science that generates novel material candidates with desired properties (e.g., high-strength alloys, efficient battery cathodes). The agent uses diffusion models conditioned on property targets, then validates candidates through physics simulation. Microsoft also has MatterSim for property prediction.

- **DeepMind AlphaFold**: While primarily a prediction model (protein folding), AlphaFold's latest versions incorporate agentic features — it can now model protein-small molecule interactions, protein-DNA complexes, and suggest mutations. The AlphaFold Server has been used by 2M+ researchers from 190+ countries.

- **Self-Driving Labs**: Fully autonomous laboratories where AI agents design experiments, execute them on robotic hardware, analyze results, and iterate. Examples: Emerald Cloud Lab, Aperture (Strateos), Artimus Robotics. The agent selects which experiments to run by reasoning about which data would be most informative (active learning/Bayesian optimization).

- **Literature Mining Agents**: Agents that read millions of papers (via PubMed, arXiv, Semantic Scholar), extract facts into knowledge graphs, and formulate novel hypotheses. Pathfinder from Google, BioGPT from Microsoft, and DrugGPT from Oxford are examples. These agents have re-discovered known drug-disease relationships and proposed novel ones.

- **Paper Drafting & Peer Review**: AI agents assist with paper writing (generate first drafts, create figures, check formatting) and peer review (evaluate methodology, check statistical correctness, identify missing citations). Tools: Paperpal, Scholarcy, Typeset (SciSpace).

---

## 11. Emerging & Cross-Domain Use Cases

### 11.1 Personal Productivity Agents

**Key Products & Evidence:**

- **Motion**: AI calendar agent that auto-schedules tasks, meetings, and deep-focus blocks. Motion's agent learns user preferences (energy patterns, meeting fatigue, project deadlines) and dynamically reschedules when things change. 100K+ paid users. Reports 20-30% time savings per week for knowledge workers.

- **x.ai (acquired by Kronan):** AI scheduling agent. Handle the "email ping-pong" of scheduling meetings across time zones. x.ai agents negotiate timing, send calendar invites, set up video calls, and send reminders. Used by 50K+ enterprise teams.

- **Clara Labs**: AI executive assistant agent that manages complex calendars, books travel, drafts emails, and handles expense reports. Clara's agents use escalation to human assistants when needed. Clara Premium service combines AI + human oversight.

- **Pattern**: Ingest email/calendar → identify scheduling intent → negotiate timing → send invites → handle cancellations/rescheduling → send reminders. Modern agents use email access APIs (Gmail, Outlook Graph) for autonomous operation.

### 11.2 Gaming & Virtual Worlds

**Key Products & Evidence:**

- **Inworld AI**: Character engine for dynamic, context-aware NPCs. Inworld's agents have backstories, personalities, emotions, and memory. They perceive the game world, react to player actions, generate dialogue in real-time, and advance narrative. Used in major titles including Ubisoft, NetEase, and Take-Two. Inworld characters can hold hour-long coherent conversations with players. Raised $500M+ at a $1.3B valuation.

- **NVIDIA ACE (Avatar Cloud Engine)**: Platform for generating interactive AI NPCs. ACE combines NVIDIA Riva (speech recognition), Nemotron LLM (dialogue), NeMo (guardrails + steering), and Audio2Face (lip-sync animation) into an agent pipeline. Demonstrated in game demos (Kairos, Cyberpunk mods) and used by developers in the NVIDIA Inworld partnership.

- **Game Testing Agents**: AI agents that play games to find bugs, test balance, and verify level completion. Microsoft's Game Intelligence agents for Xbox test games at scale, playing thousands of hours of gameplay to discover edge cases. Unity's Muse agent helps with code generation, asset tuning, and playtesting.

- **Procedural Content Generation Agents**: Agents that generate quests, dialogue trees, item descriptions, lore entries, and entire levels. AI agents can write branching narratives with internal consistency, remembering player choices. Tools: Latitude (AI Dungeon), Scenario, Layer.

### 11.3 Robotics & Physical World

**Key Products & Evidence:**

- **Google DeepMind RT-2**: Vision-Language-Action (VLA) model that translates web knowledge into robot control. RT-2 can follow novel natural language commands ("pick up the extinct animal" → picks up a dinosaur toy) without explicit training. Demonstrated emergent semantic understanding by connecting web image knowledge to physical actions. RT-2 zero-shot generalizes to unseen objects and environments.

- **Covariant (acquired by Amazon)**: AI agent for robot pick-and-place in warehouses. Covariant's RFM-1 (Robotics Foundation Model) enables robots to handle diverse objects (varying shape, texture, fragility) with human-like adaptability. The agent understands language commands ("grab the blue mug on the right") and can self-correct when grasping fails. Deployed in hundreds of warehouses globally.

- **Physical World Agent Pattern**: Natural language instruction → perceive environment (cameras, sensors) → plan action sequence → execute (motion planning, gripper control) → verify outcome → self-correct on failure. LLMs provide reasoning and planning; VLMs provide scene understanding; motion controllers execute.

- **Autonomous Drone Inspection**: Agents that fly drones to inspect infrastructure (power lines, pipelines, bridges, solar farms). The agent plans flight paths, detects anomalies via computer vision, flags defects, and generates inspection reports with remediation recommendations. Used by Skydio, Aerones, and DJI Enterprise.

- **Manufacturing Optimization**: AI agents monitor production lines (sensor data, vision feeds), detect quality defects, predict maintenance needs, and adjust process parameters to maximize yield. Siemens Industrial Copilot, GE Digital's agents, and Bright Machines apply LLM agents to factory operations.

---

## 12. Agent Archetypes

| Archetype | Description | Human Role | Examples |
|---|---|---|---|
| **Copilot** | Suggests actions and code, explains, assists | Decides which suggestions to accept | Cursor, GitHub Copilot, Gong AI |
| **Autonomous SWE** | Works independently on tickets, bugs, features | Reviews PRs, sets goals, merges | Devin, SWE-agent, Codex CLI |
| **Customer Support** | Handles Tier 1-2 tickets autonomously | Handles escalations, complex cases | Intercom Fin, Zendesk AI, Ada.cx |
| **Research Assistant** | Searches, synthesizes, and summarizes information | Refines questions, validates findings | Perplexity, Elicit, Harvey |
| **Data Analyst** | Queries databases, generates charts and reports | Reviews insights, asks follow-ups | TextQL, Databricks Genie, dbt AI |
| **DevOps Agent** | Monitors, diagnoses, and fixes infrastructure | Defines SLOs, reviews remediation | Kubiya, o11y, FireHydrant |
| **Sales Development** | Prospecting, outreach, qualification, scheduling | Closes deals, manages relationships | 11x.ai, Apollo, Outreach.ai |
| **Compliance Agent** | Document review, regulatory monitoring, gap analysis | Final sign-off, appeals | Harvey, ComplyAdvantage |
| **Tutoring Agent** | Adaptive instruction, Socratic questioning | Curriculum design, student support | Khanmigo, Duolingo Max, Tutor.ai |
| **Personal Assistant** | Calendar, email, travel, expense management | Final scheduling, preferences | Motion, Clara Labs, x.ai |
| **Robot Controller** | Natural language robot operations | Safety monitoring, edge cases | RT-2, Covariant RFM-1 |
| **NPC / Character** | Dynamic dialogue, game world interaction | Narrative design, tuning | Inworld AI, NVIDIA ACE |

---

## 13. Adoption & ROI Metrics

### Market Size (2026)
| Metric | Value | Source |
|---|---|---|
| Global AI agent market | $10.9-12.1B (44-46% CAGR through 2030) | Gartner (2026) |
| Enterprise apps embedding AI agents | 40% by end of 2026 (from <5% in 2025) | Gartner (2026) |
| GenAI contribution to global GDP | $2.6-4.4T annually by 2030 | McKinsey (2025) |
| Organizations with AI agent pilots/deployments | 70%+ | McKinsey / Deloitte (2026) |
| Top concern: security, privacy, compliance | 68% prioritized investment | InsightMark (2026) |

### Industry-Specific ROI

**Customer Service**
- 30-50% ticket deflection/automation
- 2-5× faster resolution time
- 50-70% cost reduction per interaction
- 24/7 coverage, sub-minute first response
- Sources: Zendesk, Intercom, Ada, Forethought

**Software Engineering**
- 20-50% faster feature development
- 30-40% reduction in bug-fix time
- 55% reported acceleration (GitHub)
- 1,230% YoY revenue growth (Devin)
- Sources: GitHub Copilot, Cognition Labs, Cursor

**Sales & Marketing**
- 2-5× more leads processed per SDR
- 10-20% increase in conversion rates
- 40-60% reduction in CRM data entry
- Sources: 11x.ai, Outreach.io, Salesforce

**Legal**
- 40-70% reduction in e-discovery costs
- 50% faster contract review
- 40-70% time savings on document review
- Sources: Harvey, Ironclad, Relativity

**Healthcare**
- 70-80% reduction in manual prior auth work
- 5-7% revenue uplift from improved coding accuracy
- 30% reduction in clinical documentation time
- Sources: Notable, SmarterDx, Corti

**Supply Chain**
- $1.2-2T annual value potential (McKinsey)
- 20-50% improvement in demand forecast accuracy
- 10-40% inventory reduction
- Sources: McKinsey, Blue Yonder, RELEX

### Key Challenges
| Challenge | Description | Mitigations |
|---|---|---|
| **Hallucination** | Agents invent facts, code, or actions | RAG, tool-grounded actions, self-verification loops |
| **Prompt Injection** | Malicious instructions embedded in inputs | Input sanitization, permission gating, least-privilege tools |
| **Cost at Scale** | LLM API costs at high-volume deployment | Caching, model routing (cheap/fast for simple tasks), distillation |
| **Integration** | Connecting agents to enterprise systems | MCP, API standardization, connector libraries |
| **Regulatory** | FDA, SEC, GDPR, HIPAA, AI Act compliance | Guardrails, audit trails, human-in-the-loop controls |
| **Evaluation** | Measuring agent quality across multi-step tasks | LangSmith, Weights & Biases Prompts, trajectory eval |

---

## 14. Future Trends (2026–2028)

### Multi-Agent Systems
Teams of specialized agents collaborating — a research agent hands off to a writing agent, which hands off to a publishing agent. Frameworks like **AutoGen** (Microsoft), **CrewAI**, and **LangGraph** enable agent orchestration. Google's **Agent2Agent (A2A) protocol** (April 2025) and Anthropic's **MCP** (Nov 2024, now Linux Foundation) provide interoperability standards. By 2026, both protocols co-exist under the same foundation, co-governed by OpenAI, Google, Microsoft, Anthropic, and AWS.

### Agentic RAG
Static RAG is evolving into dynamic retrieval strategies where the agent plans *what* to retrieve, *when*, and *from where* — deciding between vector search, web search, SQL queries, or API calls based on the question. Multi-hop retrieval agents decompose complex questions into sub-queries, retrieve intermediate answers, and synthesize final results.

### Computer Use Agents
Agents that control GUIs like a human — clicking buttons, filling forms, navigating menus. **Anthropic's computer use** (Claude 3.5+, 2024), **OpenAI's Operator/CUA** (Jan 2025), **Google's Project Mariner** (2025), and **Microsoft's Copilot Actions** (2025) all demonstrate GUI-level agentic autonomy. These handle tasks without API access — legacy system automation, browser-based workflows, and cross-application tasks.

### Agent Observability & Evaluation
As agents execute longer, more autonomous workflows, monitoring and evaluation become critical. Tools: **LangSmith** (traces, evaluations, datasets for agent chains), **Weights & Biases Prompts** (agent trajectory logging), **Arize AI** (LLM observability, hallucination detection), **Phoenix** (agent traces). Standardized evaluation frameworks like **AgentBench**, **GAIA**, and **SWE-bench** measure agent capabilities across domains.

### Agent Safety & Governance
- **Guardrails**: Guardrails, NeMo Guardrails, and Nvidia's NeMo ensure agents stay within policy boundaries.
- **Constitutional AI**: Fine-tuning agents to follow behavioral principles. Used by Anthropic and increasingly by enterprise deployments.
- **Human-in-the-loop**: Mandatory approval for high-stakes actions (financial transactions, medical decisions, contract signing). The agent stops and requests confirmation before proceeding.
- **Audit Trails**: Every agent action logged for compliance review. Mandatory in regulated industries (finance, healthcare, legal).

### Vertical-Specific Agents
General-purpose agents are giving way to domain-specialized agents with:
- **Fine-tuned LLMs** on domain data (legal, medical, financial)
- **Domain-specific tool libraries** (Westlaw API, Epic FHIR, Bloomberg Terminal)
- **Regulatory compliance baked in** (HIPAA workflows, AML reporting, GDPR data handling)
- **Specialized evaluation** (legal accuracy, medical safety, financial precision)

Examples: **Harvey** (legal), **Ada / SmarterDx** (healthcare), **Bloomberg ASKB** (finance), **ComplyAdvantage** (compliance), **Khanmigo** (education).

### Agent-to-Agent Economy
Agents negotiating, transacting, and collaborating on behalf of humans. Examples:
- AI travel agent books with AI hotel agent
- AI supply chain agent negotiates with AI supplier agent
- AI recruiter agent schedules interviews with AI candidate agent

Protocols like **MCP** (tools/context) and **A2A** (inter-agent communication) enable this economy. Google's A2A announcement explicitly envisions "agents as the new API endpoints."

---

## Data Sources & References

1. **McKinsey Global Institute** — "The Economic Potential of Generative AI" (2025)
2. **Gartner** — "Predicts 2026: AI Agents and Autonomous Systems"
3. **SWE-bench** — swebench.com (Verified leaderboard, 2024-2026)
4. **Cognition Labs** — Devin annual performance review (2025), Series D announcement (2026)
5. **Harvey** — "10 Legal AI Adoption Metrics" blog (2026), Harvard Business School case study (2025)
6. **Khan Academy** — Khanmigo classroom results 2025-2026 (annual report, blog post)
7. **Intercom** — Fin AI Agent reporting and performance metrics (2026)
8. **Bloomberg** — "Bloomberg embeds agentic AI into the Terminal" — The TRADE (Feb 2026)
9. **Google** — "Announcing the Agent2Agent Protocol (A2A)" — Google Developers Blog (April 2025)
10. **Anthropic** — "Introducing the Model Context Protocol" (Nov 2024), Computer Use (2024)
11. **OpenAI** — "Introducing Operator" (Jan 2025), Computer-Using Agent technical report
12. **Google DeepMind** — RT-2: Vision-Language-Action Models (July 2023)
13. **Master of Code** — "150+ AI Agent Statistics" (May 2026)
14. **Paul Okhrem** — "Enterprise AI Agents Adoption Statistics 2026"
15. **InsightMark Research** — "LLM Agent Statistics 2026"
16. **Kubiya, FireHydrant, o11y** — Product documentation and case studies
17. **Casetext, LexisNexis, Thomson Reuters** — AI product announcements and adoption data
18. **Notable Health, SmarterDx, CodaMetrix** — Product metrics and published outcomes
19. **ComplyAdvantage, Chainalysis** — Product capabilities and customer impact
20. **Inworld AI, NVIDIA ACE** — Gaming AI agent product documentation

---

*This document is maintained as part of a research repository. For corrections, additions, or updates, please open an issue or PR at https://github.com/jackliusr/research.*
