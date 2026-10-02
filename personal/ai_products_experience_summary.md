# Jack Liu Shurui (jackliusr) — AI Product Experience Summary

> **Compiled:** 2 October 2026
> **Handle:** jackliusr
> **Full Name:** Jack Liu Shurui
> **Location:** Singapore
> **Scope:** AI products, frameworks and platforms evidenced by public artefacts only
> **Primary sources:** [github.com/jackliusr/agents](https://github.com/jackliusr/agents) (fetched live 2 October 2026) · `personal/jack_liu_profile.md` (compiled 3 July 2026) · `personal/jackliusr_digital_footprint.md` (compiled 8 July 2026) · GitHub REST API `users/jackliusr` and `repos/jackliusr/agents` (fetched 2 October 2026)
> **Method:** every figure in this record is read from one of those sources. Nothing is inferred from memory and no number is rounded up. Where the evidence does not support a claim, that is stated rather than omitted.

> **Note on naming:** this record describes the current employer by sector rather than by name. The employer is named in `jack_liu_profile.md` and `jackliusr_digital_footprint.md`; it is not repeated here.

---

## 1. The Short Read

A hands-on GenAI architect who **builds and operates rather than strategy-writes.** The behavioural evidence points consistently at one working method: pick a framework, stand it up locally, work through it in a notebook, note what it does and does not do, then move to the next one while keeping the reference.

Caution that belongs with the whole document: the artefacts are **practitioner learning and architecture work**, not shipped AI products at scale, and this summary says so in §9.

---

## 2. The Primary Artefact — the `agents` Repository

The strongest single body of evidence in the footprint is one public repository. All figures fetched live on 2 October 2026.

| Field | Value |
|:------|:------|
| URL | https://github.com/jackliusr/agents |
| Description (owner's own) | *"learning on agents related topics"* |
| Created | 19 April 2026 |
| Last pushed | 25 September 2026 |
| Commits | 116 |
| Language | Jupyter Notebook |
| Size | ~254 MB |
| Visibility | Public |
| Stars / forks | 0 / 0 |
| Topics | agent · crewai · daytona · deepagents · gemini · langchain · langsmith · ollama · qdrant · qwen · rag · slack · sub-agent · tavily |

**Roughly 21 project directories, one per framework or capability:** `AutoAgents/MyAgent` · `adk-rust/my_agent` · `adk` · `agent-framework` · `claude/quickstart` · `cookbook/tool-use-tool-search-with-embeddings` · `deepeval` · `dspy` · `foundry-local/chat` · `genkit/my-genkit-app` · `haystack` · `langchain` · `llama-server` · `llamaagents` · `llamaindex` · `llamaparse/parse/getting_started` · `neo4j` · `openai/quickstart` · `ragas` · `strandsagents/quickstart` · `vllm/optimize-model-llm-compressor`.

**The repository's own README carries the analysis, and three things in it are worth more than the file list:**

- **The model posture, in the owner's own words:** *"Ollama:qwen3.6:35b-a3b-q4_K_M is used in those projects. Other models and APIs are used if there is no other options."* ✅ — a **local-first, quantised, mixture-of-experts default**, with hosted APIs as the fallback rather than the norm.
- **A checked capability list:** agent ✅ · **skills** ✅ · tools ✅ · **deepagents** ✅ · **MCP** ✅ — with **A2A ✗ · Agent Protocol ✗ · A2UI ✗** explicitly still open. That is an evaluated roadmap rather than a wish list.
- **Frameworks rejected with a stated reason:** agentsea (*"not mature, it has dependency issue of origin, can't run"*) ✗ · FastGPT ✗ · Coze ✗. Rejecting with a cause is a different signal from listing.
- **A named direction not yet started:** *"Coding Agent as SDK for harness as services"* — claude code, opencode, codex, copilot, all unchecked.
- **A China-market survey inside the README:** a table mapping Alibaba (Qwen / AgentFabric / Aliyun), Baidu (ERNIE / AgentBuilder), ByteDance (Doubao / Coze / Volcano Engine), Tencent (Hunyuan), Tsinghua and Zhipu (GLM / MetaGPT), Shanghai AI Lab (InternLM / XAgent), DeepWisdom, LangGenius (Dify) and Labring (FastGPT) by player, core model, agent framework and cloud platform. ✅ Market literacy, not tool familiarity.

**Non-obvious depth items:** `adk-rust/my_agent` — an agent built against Google's ADK **in Rust** · `foundry-local/chat` — a **local** Microsoft Foundry deployment · `vllm/optimize-model-llm-compressor` — model compression on the serving side · `cookbook/tool-use-tool-search-with-embeddings` — tool search rather than tool enumeration.

---

## 3. Frameworks, Platforms and Tooling Worked In

Consolidated from the repository tree and the checked README lists.

| Layer | Evidenced |
|:------|:----------|
| **Agent frameworks** | LangChain ✅ · LangGraph ✅ · CrewAI ✅ · Google ADK ✅ · Genkit ✅ · Microsoft Agent Framework ✅ · DSPy ✅ — plus Straws (AWS) · LlamaIndex · Haystack · LlamaAgents · AutoAgents in the tree |
| **Agent protocol / capability** | Model Context Protocol (MCP) ✅ · skills ✅ · tools ✅ · deepagents ✅ · sub-agents |
| **Inference and serving** | Ollama ✅ · llama.cpp / llama-server ✅ · vLLM ✅ · GGUF quantisation ✅ |
| **Local / edge model runtime** | Qwen3.6 35B-A3B Q4_K_M via Ollama ✅ · Microsoft Foundry Local ✅ |
| **Retrieval and RAG** | Qdrant · Neo4j · LlamaParse (document parsing) · a 14-item advanced-RAG checklist still open in the README (query transformation, fan-out, RRF, decomposition, HyDE, chunking, reranking, metadata, hybrid search, rewriting, autocut, context distillation, LLM fine-tuning, embedding fine-tuning) |
| **Evaluation** | DeepEval · RAGAS |
| **Optimisation** | Model compression via LLM Compressor · an open checklist for quantisation, knowledge distillation, parallelism and distributed serving |
| **Models and APIs** | Qwen ✅ · Gemini · OpenAI · Claude (quickstarts) |
| **Integrations** | Slack · Tavily |
| **Hardware** | AMD AI MAX 395 with the ROCm stack |

**The hardware detail is not incidental.** The profile records quantised Qwen3.6 GGUF models running on **AMD AI MAX 395 via llama.cpp and ROCm** — the non-NVIDIA path — which is consistent with a deliberate posture on cost, local control and reproducibility rather than benchmark-chasing.

---

## 4. What Has Been Built

Beyond the agents repository, from the profile's notable-repository list:

| Repository | Language | What it is |
|:-----------|:---------|:-----------|
| **hiring-agent** | Python | An AI agent that evaluates and scores résumés |
| **lca-reliable-agents** | — | Exploration of reliable agent architectures |
| **copilot-provider-4claude-code-router** | — | Claude Code and GitHub Copilot integration |
| **optimization-tech** | Python | Optimisation techniques, actively developed |
| **llm** · **notebooks** | Jupyter Notebook | LLM and data-science working repositories |
| **automatic-watermark-detection** | — | Watermark detection |
| **Polyglot-Multilingual-Learning-Assistant** | — | Multilingual learning tool |
| **superapp** | Dart/Flutter | Super-app sample with mini-programs (WeChat/AliPay model) |
| **research** | various | The research corpus this record sits in |

**Hugging Face:** account `jackliusr`, joined 6 May 2025; **one model published — `code-search-net-tokenizer`** (updated ~June 2026); the profile additionally records an earlier `phi3-mini-yoda-adapter` upload. ❌ No datasets published.

**Certification:** **Claude Certified Architect – Foundations** (Anthropic, 2026). ✅

---

## 5. Writing and the Positions That Differentiate

The personal site carries **193+ tech-tagged posts**; the Blogger archive carries **197+ posts** from ~2007 to 2024. The 2026 AI cluster is the relevant evidence:

| Post | Date | Subject |
|:-----|:-----|:--------|
| One GEPA Report of DSPy | June 2026 | Programmatic LLM optimisation with DSPy/GEPA |
| Running Qwen3.6 MTP GGUF on AMD AI MAX 395 with llama.cpp ROCm | June 2026 | Local LLM deployment on non-NVIDIA hardware |
| Reducing Architecture Drift in Spec-Driven Development with Coding Agents and LLMs | May 2026 | Agentic development and architecture governance |
| From Conventional Commits to LLM-Generated Release Notes | June 2026 | LLMs in the software-delivery pipeline |
| Fault-Oblivious Stateful Workflows: Durable Execution Matters More Than Orchestration | May 2026 | Durable execution, continuing pre-AI work |
| Working Around ROCm PyTorch Replacement Issues with uv and ComfyUI | May 2026 | AMD GPU tooling |

**The differentiating position is the third of those, and it is also the owner's own stated thesis.** The current LinkedIn headline argues that *AI coding agents can generate software at incredible speed, but speed without architectural governance creates architecture drift.* ⚠ The pairing is the notable part: very few practitioners running eight agent frameworks are simultaneously publishing on the governance failure mode that those frameworks create.

**Corroborating evidence for the AI focus:** the profile's 2024–2026 focus areas record DSPy for programmatic LLM optimisation, LLM evaluation frameworks, GEPA optimisation runs for financial NLP, quantised models on AMD hardware, and coding agents for spec-driven development.

---

## 6. Professional Frame

- **Current role** ⚠ per the digital-footprint record: **Solution Architect, GenAI** at a corporate and investment bank in Singapore — GenAI solutions architecture, architecture governance, DevOps, SRE and platform engineering, including AI coding-agent integration.
- **LinkedIn:** 563 followers, **246 posts**, 3 articles (as of the July 2026 compile). Posting pattern is certification milestones, technical learning and DevOps/cloud-native material.
- **Stack Overflow:** 590 reputation, 25 answers, ~157k reach, 13+ years. ⚠ **The topics are database, .NET and infrastructure — not AI.**
- **Stack Exchange network:** 9 accounts including Quantitative Finance, Server Fault, Unix & Linux and Super User.
- **Public repos:** 166 at the July 2026 compile; **173 as returned by the live API on 2 October 2026** — the profile has drifted and its figures should be read as of its compile date.
- **Career context:** team-lead and architecture roles across defence/SCADA and command-and-control (ST Electronics), flight simulation (Combuilder), logistics and warehouse management (Y3 Technologies), healthcare IT (a national HealthTech agency GitHub account), and the current banking role. The pre-AI domain breadth — SCADA, C2, WMS, ERP, e-commerce, payments, gaming, core banking, credit processing, VPN/VoIP, HIS/FHIR — is the context in which the AI work now sits.

---

## 7. What the Evidence Does Not Support

Stated as plainly as the rest, because a summary that only lists strengths is not a summary.

- ❌ **The repositories are learning repositories by the owner's own description** (*"learning on agents related topics"*), not shipped products. Stars and forks are light: **12 stars and 19 forks across all 173 public repositories**; the agents repository has 0 of each.
- ❌ **No Kaggle presence**, no AI patents, no academic publications, no ORCID, no Google Scholar record, and no conference talks or slide decks found — the footprint searched and recorded none.
- ❌ **No production-scale AI claim is evidenced anywhere in the public record.** The employer-facing role is architecture and governance, and that is what the record supports.
- ⚠ **Stack Overflow reputation is not AI-derived** (see §6).
- ⚠ **The two profile documents are point-in-time records** (3 and 8 July 2026) and every count in them has moved since.
- ⚠ **The framework/registry lists in the agents README are the owner's own checkboxes.** A checked box evidences work started; the repository tree evidences scaffolding alongside completed work. Neither is a delivered product.

---

## 8. Verdict

Unusually broad, genuinely hands-on AI-product range: **roughly twenty frameworks and platforms across agents, MCP, skills, retrieval, evaluation, serving and quantised local inference**, plus an original and currently under-populated governance position on architecture drift in agent-assisted development.

The honest qualifier is the one the evidence itself supplies: this reads as **deep practitioner learning and architecture work by someone who insists on standing each technology up himself**, rather than as a portfolio of shipped AI products.

---

## 9. Sources, Dates and Verification

| Source | How obtained | Date | What it established |
|:-------|:-------------|:-----|:--------------------|
| `github.com/jackliusr/agents` | Fetched live | 2 Oct 2026 | Description, 116 commits, created 19 Apr 2026, pushed 25 Sep 2026, Jupyter Notebook, ~254 MB, 14 topics, the full directory tree, and the README's model line, capability checklist, rejected frameworks, coding-agent direction and China survey |
| GitHub REST API `users/jackliusr` | Fetched live | 2 Oct 2026 | 173 public repos, 21 gists, 55 followers, 120 following, joined 4 Apr 2009, website `jackliusr.github.io`, location Singapore |
| `personal/jack_liu_profile.md` | Read in full | compiled 3 Jul 2026 | Current role and focus, previous roles, domains, skills inventory, notable repositories (hiring-agent, lca-reliable-agents, copilot-provider-4claude-code-router and others), blog post titles and dates, AMD AI MAX 395 / ROCm / DSPy / GEPA details, Stack Overflow and X figures, Claude certification |
| `personal/jackliusr_digital_footprint.md` | Read in full | compiled 8 Jul 2026 | LinkedIn figures (563 followers, 246 posts), Hugging Face model `code-search-net-tokenizer` and join date, Stack Exchange accounts, certification list, the negative findings in §7, and the repo/licence breakdown |

**Verification notes.** ✅ Figures marked from a live fetch were read from the API or page on 2 October 2026. ⚠ Figures from the two profile documents are as of their compile dates and have drifted. ❌ The absence findings in §7 are recorded as *not found in the sources searched*, which is what the footprint itself concluded — an absence of evidence in a public-platform search, not proof of non-existence.

**No credential was used to compile this record.** All sources were public pages or the unauthenticated public GitHub API.

---

*Compiled by Hermes Agent on 2 October 2026 for Jack Liu Shurui (jackliusr). Companion records: `personal/jack_liu_profile.md`, `personal/jackliusr_digital_footprint.md`.*
