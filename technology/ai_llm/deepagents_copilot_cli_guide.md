# Running DeepAgents Using GitHub Copilot CLI — The Practical Runbook

> **A comprehensive how-to guide on running LangChain's DeepAgents from GitHub Copilot CLI — the practical runbook — covering the prerequisites, the Copilot CLI setup, the DeepAgents install, the integration pattern, the configuration, a bank-domain worked example, troubleshooting, and the one-page summary.** Every command in this guide was verified against the primary sources (the `langchain-ai/deepagents` repository and PyPI metadata, `docs.github.com` Copilot CLI documentation, and LangChain's Deep Agents launch material) at the time of writing; anything that could not be verified is flagged explicitly rather than guessed.

**Author:** Jack Liu Shurui · **Role:** Solution Architect, Crédit Agricole CIB
**Series:** LLM/AI · Agent Engineering · **Domain:** AI Engineering · Developer Tooling
**Reading time:** ~40 minutes
**Companion guides:** [autonomous_agents_guide.md](autonomous_agents_guide.md) (the umbrella — where agent taxonomies live), [agent_scaffolding_guide.md](agent_scaffolding_guide.md) (the harness anatomy), [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) (the tool-execution containment angle), [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md) (the production bar), [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) (the failure taxonomy), [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md) (the boundary), [agentops_guide.md](agentops_guide.md) (the observability), [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md) (the platform), [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) (the MCP tools an agent may use), [../durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md) (the durable workflows), [../agentic_workflows_guide.md](../agentic_workflows_guide.md) (the workflow patterns), [../on_prem_llm_deployment_guide.md](../on_prem_llm_deployment_guide.md) (the self-hosted model option)

---

## Table of Contents

1. [The Prerequisites](#1-the-prerequisites)
2. [The Copilot CLI Setup](#2-the-copilot-cli-setup)
3. [The DeepAgents Install](#3-the-deepagents-install)
4. [The Integration](#4-the-integration)
5. [The Configuration](#5-the-configuration)
6. [The Worked Example](#6-the-worked-example--a-bank-domain-research-task)
7. [The Troubleshooting](#7-the-troubleshooting)
8. [The One-Page Summary](#8-the-one-page-summary)
9. [The Glossary](#9-the-glossary)
10. [Appendix — Sources and Verification Notes](#10-appendix--sources-and-verification-notes)
11. [Appendix A — The Quick Command Cheat Sheet](#appendix-a--the-quick-command-cheat-sheet)

---

## 1. The Prerequisites

### 1.1 The Big Picture: Two Agents, One Terminal

This runbook wires together **two independent agent tools that both live in your terminal**:

- **GitHub Copilot CLI** — GitHub's terminal-native AI agent, distributed as the npm package `@github/copilot`. It answers questions, suggests and explains shell commands, and (in its agentic mode) plans and executes coding work directly in the terminal. It authenticates against **your GitHub account's Copilot subscription** — it is the *GitHub-side* agent.
- **DeepAgents** — LangChain's open-source "batteries-included" agent harness (and its sibling, the **Deep Agents CLI**), built on LangGraph. It brings planning, a filesystem, context management, and **sub-agent delegation** (the hub-and-spoke pattern) out of the box. It is model-agnostic and runs on **your own model API keys** (OpenAI, Anthropic, Google, or self-hosted) — it is the *LangChain-side* agent.

The two have **complementary strengths**: Copilot CLI is a fast, conversational coding agent tied to GitHub's infrastructure; DeepAgents is a long-horizon, delegation-heavy research/engineering harness. The integration pattern in §4 composes them at the **process level** — the Copilot CLI's terminal agent *spawns* the DeepAgents CLI as a subprocess, and the two share a working directory, a repository, and a set of files. There is **no official GitHub plugin** that embeds DeepAgents inside Copilot CLI (flagged — nothing of the sort existed in the official docs at the time of writing); the integration is shell composition, which is exactly the kind of thing a terminal agent is good at.

Before any of that works, the machine, the accounts, and the runtimes need to be in place. This section is the checklist.

### 1.2 The Requirements

**1. Python 3.11+ (verified — and note the correction).** DeepAgents is a Python library/CLI. The task brief said "Python 3.10+", but the **PyPI metadata for `deepagents` (version 0.7.6, checked at the time of writing) declares `Requires-Python: <4.0,>=3.11`** — so current releases need **Python 3.11 or newer**. *Flag: earlier releases (mid-2025 vintage) supported older interpreters; if you pin an old `deepagents` version, check that release's own metadata with `pip index versions deepagents` / `pip show deepagents`.* Check your interpreter with:

```bash
python3 --version   # need 3.11+
```

**2. Node.js 22 LTS+ and npm (verified).** Copilot CLI is a Node.js application distributed via npm, so it needs a current Node.js LTS release with npm. GitHub's installation docs point at npm as the primary path; community walkthroughs cross-checked against the docs cite **Node.js 22+** as the practical minimum for the current CLI. *Flag: the exact minimum Node version has moved upward across CLI releases — "current LTS" is the safe rule.* Verify:

```bash
node --version     # Node 22 LTS or newer
npm --version      # npm ships with Node
```

**3. A GitHub account with an active Copilot subscription (verified — plan-flagged).** The Copilot CLI authenticates as *you* and bills usage to your Copilot plan. As of the 2026 pricing pages: **Copilot Free includes Copilot CLI** (with quotas — ~2,000 code completions and ~50 chat/agent-mode requests per month, per `docs.github.com` plan pages), and **Pro/Pro+**, **Business**, and **Enterprise** plans include it with higher limits. *Flags: (a) the Free-tier quotas are the classic "works until it doesn't" gotcha — a long DeepAgents delegation can burn a month's request quota in one session; (b) plan terms, names, and quotas change — verify your plan's current terms on the GitHub plans page and the usage page (`https://github.com/settings/copilot`) before assuming anything; (c) **organization-managed accounts** — if your GitHub account is governed by a Crédit Agricole CIB org policy, an admin may need to enable Copilot CLI / agent-mode access, and org policies can block terminal agents outright.* Verify your entitlement with `copilot auth status` once the CLI is installed (§2.3).

**4. A model API key for DeepAgents (verified — and a common misunderstanding).** DeepAgents does **not** use your Copilot subscription — it is model-agnostic and calls the model provider of your choice with **your own key**. For the CLI that means one of `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or `GOOGLE_API_KEY` exported in the environment (the CLI's documented default model is **Anthropic Claude Sonnet 4** — flagged, confirm at install time). The harness library can additionally run on open-weight models via Baseten/Fireworks, or self-hosted via Ollama/vLLM/llama.cpp (see [../on_prem_llm_deployment_guide.md](../on_prem_llm_deployment_guide.md) for the self-hosted track).

**5. A terminal.** macOS/Linux: bash or zsh. Windows: PowerShell 6+ or Windows Terminal (Copilot CLI supports Windows; some command-suggestion UX differs). WSL2 on Windows is a solid middle path for the Python side.

**6. Optional but recommended: a Tavily API key.** The Deep Agents CLI's `web_search` tool needs `TAVILY_API_KEY` (verified — the CLI docs list Tavily as the web-search backend). Without it, the agent researches from its training knowledge and whatever files/URLs you give it — fine for coding tasks, weak for a research brief.

### 1.3 The Prerequisites Table

| Requirement | Version | Notes |
|---|---|---|
| Python | **3.11+** | Verified from PyPI `Requires-Python: <4.0,>=3.11` (deepagents 0.7.6). Task brief said 3.10+ — **corrected**; pin older releases only with old-version metadata checked. |
| Node.js | **22 LTS+** (flag: minimum has crept up across CLI releases) | Required for the Copilot CLI (npm distribution). `node --version` to check. |
| npm | Ships with Node | The Copilot CLI's package manager: `npm install -g @github/copilot`. |
| GitHub account | Any, with active Copilot plan | **Plan-flagged**: Free includes CLI with ~2,000 completions + ~50 chat/agent requests/month (2026 docs); Pro/Pro+/Business/Enterprise higher. Org-managed accounts may need admin enablement. |
| Model API key (DeepAgents) | One of OpenAI / Anthropic / Google | DeepAgents bills to **your** key, not Copilot. `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or `GOOGLE_API_KEY`. CLI default model: Claude Sonnet 4 (flag — verify). |
| Terminal | bash/zsh (macOS/Linux); PowerShell 6+/WSL2 (Windows) | Both CLIs are interactive-first; a real TTY matters for approval prompts. |
| Tavily API key (optional) | `TAVILY_API_KEY` | Powers DeepAgents CLI's `web_search` tool (verified in CLI docs). |
| Disk + network | ~2 GB free, outbound HTTPS to npm, PyPI, GitHub, and your model provider | Corporate proxies/firewalls (common at banks) must allow these hosts. |

### 1.4 The Verification Sweep — Five Minutes Before You Start

Run all of these before the first delegation; they confirm the entire chain in under five minutes:

```bash
# 1. Python (needs 3.11+ — verified PyPI requirement)
python3 --version

# 2. Node + npm (needs current LTS — flag: minimum has crept up)
node --version && npm --version

# 3. Copilot CLI present?
copilot --version

# 4. Copilot CLI authenticated + entitled?
copilot auth status

# 5. DeepAgents CLI reachable (pulls the package on first run)?
uvx deepagents-cli --help

# 6. Model key set? (pick your provider)
[ -n "$ANTHROPIC_API_KEY" ] && echo "Anthropic key: OK" || echo "Anthropic key: MISSING"
[ -n "$TAVILY_API_KEY" ] && echo "Tavily key: OK" || echo "Tavily key: MISSING (web_search will fail)"
```

Expected outcome: six lines, all green except the optional Tavily line. Anything red maps to a row in the prerequisites table above and, if it is a *runtime* failure rather than a *setup* failure, to the troubleshooting table in §7.

---

## 2. The Copilot CLI Setup

### 2.1 What Copilot CLI Is

**GitHub Copilot CLI** is the command-line interface for GitHub Copilot: "use Copilot directly from your terminal to answer questions, write and debug code, and interact with GitHub" (docs.github.com). It started life as the `gh copilot` extension and matured into a standalone `copilot` binary distributed as the npm package **`@github/copilot`** (verified — this is the package name in GitHub's installation docs; some blog posts show the older/incorrect `@github/copilot-cli`, ignore those). It is built on Node.js, stores its configuration and session data under `~/.copilot`, and offers three modes you will actually use:

- **`copilot explore`** — an *interactive* session for understanding code: ask questions about a repository, get file-grounded answers, keep a threaded conversation. Think "chat with your repo" in the terminal.
- **`copilot suggest`** — a *non-interactive* one-shot: describe what you want to do in plain English and it returns a shell command (e.g. `copilot suggest "find all TODO comments in this repo"`). The companion `copilot explain` does the reverse — it explains a command you paste in. (Historically `gh copilot explain`; the standalone CLI carries the equivalent.)
- **`copilot agents`** — the *agentic mode*: a full agent loop in your terminal that plans, reads and edits files, runs commands, and iterates until the task is done. Interactive by default; run one-shot with `--prompt "..."` (verified pattern in GitHub's CLI docs and the 2026 GitHub blog "Copilot CLI 101"). This is the mode the integration in §4 builds on.

### 2.2 The Install (verified)

Install the CLI globally with npm:

```bash
npm install -g @github/copilot
```

Then confirm the binary is on your PATH:

```bash
copilot --version
```

*Verified against docs.github.com's "Installing GitHub Copilot CLI". Flags: (a) on Windows, the npm global bin directory (`%APPDATA%\npm`) must be on PATH; (b) if you cannot install globally (no admin rights on a bank laptop), run it without installing via `npx @github/copilot` — but note that interactive agent sessions work far better from a real global install; (c) some environments also offer Homebrew/WinGet/installer-script variants (flagged — npm is the canonical, verified path).*

### 2.3 The Authentication (verified)

The CLI authenticates against GitHub — it does not take a model key:

```bash
copilot auth login
```

This launches a browser-based OAuth flow (device-code flow in headless/SSH contexts): you approve the Copilot CLI app on github.com, and the CLI stores the token. Session checks:

```bash
copilot auth status     # shows the signed-in account and entitlement
```

*Verified against docs.github.com's "Authenticating GitHub Copilot CLI". Flags: (a) authentication is per-user and per-machine; (b) if your org enforces SSO, the OAuth approval must include your org's SSO authorization or the CLI will fail at request time with a 403/401; (c) in CI-style environments you can fall back to a GitHub token with the right scopes (`GH_TOKEN`) — flagged as an advanced path, prefer interactive OAuth for a workstation.*

### 2.4 The CLI Basics (verified)

Three commands cover 90% of daily use:

```bash
# 1. Explore — interactive, repo-grounded Q&A
cd /path/to/your/repo
copilot explore

# 2. Suggest — one-shot shell command generation
copilot suggest "compress the PNGs in assets/ to webp"

# 3. Agents — the agentic loop (interactive session)
copilot agents

# 3b. Agents — one-shot, non-interactive
copilot agents --prompt "Add unit tests for the payment service and run them"
```

Inside `copilot agents` you get an interactive loop: the agent proposes file edits, runs commands, and asks before destructive actions; `--prompt` runs the same loop non-interactively and prints a result. `Ctrl+C` interrupts, `/exit` (or `Ctrl+D`) ends a session, and `/help` lists session commands.

*Flags: (a) the exact subcommand surface (`explore`, `suggest`, `explain`, `agents`) has evolved across CLI releases and some commands were originally namespaced under `gh copilot`; run `copilot --help` on your installed version for the authoritative list; (b) the `--prompt` one-shot flag is verified in GitHub's CLI docs for the agents/agentic command family — exact spelling per version; (c) agent-mode usage counts against your Copilot plan's request quota (see §7, rate limits).*

### 2.5 The Session Basics (verified-flagged)

`copilot agents` is where the integration happens, so learn its session grammar early:

- **Prompt freely, confirm deliberately.** The agent proposes commands and file edits and asks before destructive actions — treat those confirmations as real gates, the same habit [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) drills for tool execution.
- **Session controls.** `Ctrl+C` interrupts the current agent turn; `/exit` (or `Ctrl+D`) ends the session; `/help` lists in-session commands (flag: the exact slash-command set varies by CLI release — `/help` is authoritative).
- **Context is per-session.** Copilot CLI sessions are conversational but bounded; long multi-turn sessions hit plan quotas (see §7) and diminishing context returns. For *long-horizon* work, that is precisely when you delegate to DeepAgents rather than grinding in the chat.
- **Where things live.** Configuration, session data, and customizations sit under `~/.copilot/` (verified — docs.github.com "Copilot CLI reference"). `copilot config` exposes preferences in newer releases (flag: version-dependent).

### 2.6 The Setup Table

| Step | Command | Notes |
|---|---|---|
| 1. Install the CLI | `npm install -g @github/copilot` | Verified package name; global install puts `copilot` on PATH. |
| 2. Verify the binary | `copilot --version` | If "command not found", fix the npm global bin PATH. |
| 3. Authenticate | `copilot auth login` | Browser OAuth; device-code flow on headless boxes. |
| 4. Check entitlement | `copilot auth status` | Confirms account + active Copilot plan; org SSO must be authorized. |
| 5. Smoke-test | `copilot suggest "show git status in one line"` | First call confirms end-to-end auth + model routing. |
| 6. Try the agent | `copilot agents --prompt "What files are in this repo?"` | One-shot agent run; watch it plan and answer. |
| 7. (Optional) Customize | `copilot config` | Model/keybinding/session preferences — flag: options vary by CLI version. |

---

## 3. The DeepAgents Install

### 3.1 What DeepAgents Is (verified)

**DeepAgents** is LangChain's open-source "batteries-included agent harness" (the `langchain-ai/deepagents` repository, MIT-licensed; launched June 2025 — verified via the LangChain blog "Deep Agents" and repo history). PyPI's own summary: *"Production-ready, extensible agent harness with a built-in filesystem and context management, sub-agent delegation, skills, and long-term memory."* Its verified feature set:

- **Sub-agents** — the **hub-and-spoke** pattern: a supervisor agent delegates chunks of work to subagents with **isolated context windows**, then assembles results. This is what makes it a *deep* agent for long-horizon tasks.
- **Filesystem** — read/write/edit/search over pluggable local, sandboxed, or remote backends.
- **Context management** — summarizes long threads and offloads tool outputs to disk so context stays small.
- **Shell access** — runs commands in your sandbox of choice (the tool-execution containment angle is covered in [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md)).
- **Persistent memory**, **human-in-the-loop** approvals, **skills**, and **tools** (bring your own functions or any MCP server — see [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md)).
- **Model-agnostic** — any LLM with tool calling: frontier APIs, open-weight via Baseten/Fireworks, or self-hosted via Ollama/vLLM/llama.cpp.

There are **two install surfaces**, and the runbook uses both:

1. **The harness library** — `pip install deepagents` (or `uv add deepagents`), imported in Python as `from deepagents import create_deep_agent`. This is the *framework*.
2. **The Deep Agents CLI** — LangChain's terminal agent (announced October 2025, "Introducing Deep Agents CLI" on langchain.com): an interactive CLI with persistent memory across sessions, built-in tools (`ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`, `shell`, `execute`, `web_search`, `fetch_url`, `task`, `write_todos`), approval gates on sensitive tools, and remote-sandbox execution (Modal, Daytona, Runloop). Installed via `uvx deepagents-cli` or `pip install deepagents-cli`. This is the *runner*.

### 3.2 The Harness: `pip install deepagents` (verified)

```bash
# Option A — pip (any Python 3.11+ environment)
pip install deepagents

# Option B — uv (faster, modern)
uv add deepagents            # in a project
uv pip install deepagents    # into the current environment
```

Then use it from Python:

```python
from deepagents import create_deep_agent

agent = create_deep_agent(
    model="openai:gpt-5.5",          # any tool-calling LangChain chat model
    tools=[],                        # your functions, or MCP servers
    system_prompt="You are a research assistant.",
)
result = agent.invoke({"messages": "Research LangGraph and write a summary"})
```

*Verified against the `langchain-ai/deepagents` README (which shows `uv add deepagents` + the `create_deep_agent` quickstart) and PyPI. Flags: (a) the model string format is `provider:model` and depends on the provider package you have installed (e.g. `langchain-anthropic`, `langchain-openai`) — install the matching `langchain-*` provider package; (b) `gpt-5.5` in the README example is the README's own example string — use the model your key can actually reach; (c) for the harness's default model when unspecified — flag: check the version's defaults rather than assuming.*

### 3.3 The CLI: `uvx deepagents-cli` (verified)

The Deep Agents CLI runs without a global install via `uvx`, or installs persistently via `uv tool` / pip:

```bash
# Option A — run directly (no global install) — verified in the CLI docs
uvx deepagents-cli

# Option B — persistent global install via uv — verified in the CLI docs
uv tool install deepagents-cli
deepagents

# Option C — pip — verified in the CLI docs
pip install deepagents-cli
deepagents
```

*Verified against the Deep Agents CLI documentation (docs.langchain.com, "Use the CLI") which shows `uv tool install deepagents-cli` → `deepagents`, `uvx deepagents-cli`, `pip install deepagents-cli`, and `uv add deepagents-cli`. Flag: `uvx deepagents-cli` is the canonical no-install path, but every invocation re-resolves the package — for repeated use, `uv tool install deepagents-cli` is faster.*

### 3.4 The DeepAgents Basics: `deepagents run` and the CLI (verified)

The CLI is interactive by default — start it, set a task, and it plans, delegates, and edits with approval gates:

```bash
# Set the model provider key first (CLI default model is Anthropic Claude Sonnet 4 — flag: confirm at install)
export ANTHROPIC_API_KEY="sk-ant-..."
export TAVILY_API_KEY="tvly-..."        # optional, enables web_search

# Interactive session
deepagents
> Create a Python script that prints "Hello, World!"

# One-shot / headless run — verified pattern in LangChain CLI material
deepagents run "Research X and write a markdown briefing" --headless

# Named agent with persistent memory
deepagents --agent research
```

Key CLI surface (verified against the CLI docs):

- **Commands:** `deepagents run "<task>"` (execute a task), `deepagents list` (list agents/memories), `deepagents help`, `deepagents reset --agent NAME` (clear memory), `deepagents reset --agent NAME --target SOURCE` (copy memory from another agent).
- **Flags:** `--agent NAME` (named memory), `--auto-approve` (skip approval prompts; toggle mid-session with `Ctrl+T`), `--sandbox TYPE` (`modal`, `daytona`, or `runloop`), `--sandbox-id ID` (reuse an existing sandbox), `--sandbox-setup PATH` (provisioning script).
- **In-session:** `/tokens` (token usage), `/clear` (clear conversation), `/exit`; prefix a line with `!` to run a raw shell command (`!git status`); `Ctrl+T` toggles auto-approve; `Alt+Enter` newline.
- **Memory:** agents persist markdown memories under `~/.deepagents/<AGENT_NAME>/memories/` — teach an agent your conventions once and it applies them in later sessions.

*Flags: (a) the exact `deepagents run` one-shot invocation is corroborated in LangChain CLI material and community write-ups (e.g. `deepagents run "<task>" --headless`) — if your installed version differs, `deepagents help` is authoritative; (b) the CLI's default model is reported as Claude Sonnet 4 in the CLI docs — verify against your installed version's help output; (c) the older blog-era `deepagents` demo CLI (pre-October-2025) is not the same surface — use the current `deepagents-cli` package.*

### 3.5 The Install Table

| Step | Command | Notes |
|---|---|---|
| 1. Python 3.11+ | `python3 --version` | Required by deepagents (PyPI metadata, verified). |
| 2. Install the harness | `pip install deepagents` (or `uv add deepagents`) | Verified README/PyPI. Brings `create_deep_agent`. |
| 3. Install provider packages | e.g. `pip install langchain-anthropic` | One per model provider you will use (verified pattern; package names per LangChain). |
| 4. Install the CLI | `uvx deepagents-cli` (no-install) or `uv tool install deepagents-cli` / `pip install deepagents-cli` | Verified CLI docs. Persistent install → `deepagents` command. |
| 5. Set keys | `export ANTHROPIC_API_KEY=...` (+ `TAVILY_API_KEY`) | CLI default model Claude Sonnet 4 (flag — verify). Keys also loadable from `.env`. |
| 6. Smoke-test | `deepagents run "List the files in this directory" --headless` | First run exercises plan → tool → answer loop. |
| 7. Try delegation | `deepagents` → `> Summarize the repo, delegating per-module` | Watch the `task` tool spawn subagents (hub-and-spoke). |

### 3.6 Which Install Surface When (decision guidance)

Three entry points exist in the DeepAgents family; picking wrong is the most common install confusion:

| Surface | Install | Use it when | Skip it when |
|---|---|---|---|
| **Harness library** | `pip install deepagents` | You are *building* — wiring DeepAgents into Python code, custom tools, MCP servers, LangGraph graphs, evaluation. | You just want a research assistant in the terminal. |
| **Deep Agents CLI** | `uvx deepagents-cli` (or `uv tool install deepagents-cli`) | You are *operating* — interactive research/coding tasks with memory, approvals, subagent delegation, sandboxes. **This is the surface the Copilot CLI delegates to in §4.** | You need programmatic control or custom middleware (harness instead). |
| **Deep Agents Code** | `curl -LsSf https://langch.in/dcode \| bash` (per repo README note) | You want a pre-built, Claude-Code-style coding agent without assembling the harness yourself. | You need the harness's full extensibility or the CLI's memory model. |

*The README's "Deep Agents Code" one-liner is quoted from the repo (verified); flag: it is a separate product line with its own docs — this runbook's integration uses the CLI surface, not Deep Agents Code.*

---

## 4. The Integration

### 4.1 The Pattern: Running DeepAgents from the Copilot CLI (verified — pattern-flagged)

There is **no official "Copilot CLI ↔ DeepAgents" integration** — no plugin, no native subagent bridge, no documented handshake in either project (flagged: neither the GitHub docs nor the DeepAgents repo documented one at the time of writing). The integration that *actually works* — and the one this runbook verifies — is **process composition**: the Copilot CLI's agentic mode is itself a shell-native agent that can plan, execute, and supervise shell commands, so you simply have it **run the DeepAgents CLI as a subprocess**. The two agents then cooperate across three shared surfaces:

1. **The working directory** — both agents operate on the same repo, so files DeepAgents writes are files Copilot CLI can read, review, and commit.
2. **The task boundary** — Copilot CLI owns the interactive session, the approvals, and the conversation with *you*; DeepAgents owns the long-horizon, delegation-heavy work (research, multi-file analysis, briefings).
3. **The observability boundary** — everything DeepAgents prints (plans, tool calls, subagent spawns) streams into the Copilot CLI session, so the terminal agent can summarize, verify, and follow up on it.

This mirrors the "orchestrator + specialist" split covered in [autonomous_agents_guide.md](autonomous_agents_guide.md) and [agent_scaffolding_guide.md](agent_scaffolding_guide.md): **Copilot CLI is the interactive orchestrator, DeepAgents is the delegated specialist.** The pattern is also the natural extension of what GitHub's own "Copilot CLI 101" blog describes — a terminal agent that "runs commands and iterates" — and it requires **no special flags on either side**: just a prompt that instructs the terminal agent to invoke `uvx deepagents-cli` (or `deepagents`) with a task.

### 4.2 The Workflow Steps (verified — practical)

**Step 1 — Start the Copilot CLI agent.** In the repo you want to work in:

```bash
cd ~/research   # or any repo
copilot agents
```

**Step 2 — Delegate the deep task.** Prompt the terminal agent to run DeepAgents as a subprocess. The prompt carries three things: *the tool* (`uvx deepagents-cli`), *the agent identity* (`--agent research` for persistent memory), and *the task* (exact, outcome-oriented). Example:

```
Run the DeepAgents CLI as a subprocess with this task:
uvx deepagents-cli --agent research \
  "Research the state of ISO 20022 adoption in corporate banking and write
   a structured markdown briefing to iso20022_briefing.md with sources."
Do not stop at the first error — if a dependency is missing, install it,
then rerun. When the run finishes, summarize the briefing for me.
```

**Step 3 — Supervise through the session.** The Copilot CLI streams DeepAgents' output. You keep the human-in-the-loop gates: DeepAgents will prompt for approvals before writing files or running shell commands (unless `--auto-approve`, which this runbook advises against for bank work — see [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) §10 for why approvals are the last line of defense). Interrupt with `Ctrl+C` if the delegation goes sideways.

**Step 4 — Verify and follow up.** When the DeepAgents run ends, keep the session going:

```
Read iso20022_briefing.md, check the claims for internal consistency,
then show me a 10-line executive summary and the list of sources.
```

The terminal agent reads the produced file (the shared-filesystem surface), critiques it, and hands you the summary — a review loop that DeepAgents alone wouldn't do.

**Step 5 — Optionally commit.** Let the Copilot CLI stage and commit the produced artifacts with a descriptive message — or do it yourself if your bank's commit policy is strict.

**Variants worth knowing (verified-flagged):**

- **One-shot delegation** — `copilot agents --prompt "Run: uvx deepagents-cli --agent research '<task>'"` runs the whole loop non-interactively. Good for CI-style research kicks, but you lose the mid-flight steering of an interactive session.
- **Side-by-side (no delegation)** — run `deepagents` in one terminal pane and `copilot agents` in another, both in the same repo. DeepAgents writes the research artifacts; Copilot CLI does the code surgery. Simpler mental model, no prompt engineering, but no supervision of the DeepAgents run.
- **DeepAgents-first** — start `deepagents`, do the research, then `!copilot agents --prompt "..."` from inside the DeepAgents session (the `!` shell escape) to invoke Copilot CLI. Reverses the supervision direction; useful when the research task is the primary and coding is secondary.

### 4.3 The Integration Table

| Step | Command / Prompt | Notes |
|---|---|---|
| 1. Enter the orchestrator | `cd <repo>` then `copilot agents` | Interactive agentic session in the target repo. |
| 2. Delegate | Prompt: *"Run `uvx deepagents-cli --agent research '<task>'`, fix errors, rerun, then summarize."* | The terminal agent spawns DeepAgents as a subprocess; shared working directory. |
| 3. Approve gates | Watch for DeepAgents approval prompts (file writes, shell, web_search) | Keep HITL on for bank work; `--auto-approve` exists but expands blast radius (see sandboxing guide). |
| 4. Verify output | Prompt: *"Read <file>, check for inconsistencies, summarize."* | Uses the shared filesystem — the key integration surface. |
| 5. Wrap up | Prompt: *"Commit the artifacts with a descriptive message."* — or commit manually | Respect repo/org commit policies. |
| One-shot variant | `copilot agents --prompt "Run: uvx deepagents-cli '<task>'"` | Non-interactive; no mid-flight steering. |
| Side-by-side variant | `deepagents` in pane A, `copilot agents` in pane B | No delegation; two agents, one repo. |

### 4.4 Security Considerations for the Delegation Pattern (flagged — think before you delegate)

Composing two agents in one repo multiplies the trust surface, so the runbook's bank-environment advice up front:

- **Secrets in prompts.** The delegation prompt travels through two agents' context. Never put API keys, tokens, or passwords in the *task text* — put them in the environment (`.env` / exported vars), which DeepAgents reads natively (verified: keys as env vars or `.env`). A key pasted into a prompt ends up in transcripts, session logs, and `~/.copilot` session data.
- **Prompt injection via researched content.** DeepAgents' `web_search`/`fetch_url` pull untrusted content into context; an injected page can steer the agent's tool calls. Your defenses: approval gates on file writes and shell (keep them on), a sandboxed or least-privilege execution context (see [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) and [prompt_injection_guide.md](prompt_injection_guide.md)), and — for bank work — never pointing the agent's *shell* tool at anything but a disposable workspace.
- **Blast radius of `--auto-approve`.** It exists for trusted, sandboxed loops. In an unsandboxed local run it converts every approval gate into a free pass — the exact trade the sandboxing guide's §1.3 blast-radius framing warns about.
- **Least privilege by construction.** Run the delegation in a working directory that contains only what the task needs; the shared-filesystem surface means the two agents collectively see everything in that repo. A dedicated research repo (or subdirectory) beats running agents in your personal dotfiles checkout.
- **Audit trail.** Both CLIs print full traces; for regulated work, keep the session transcripts (Copilot CLI stores session data under `~/.copilot/`; DeepAgents memories under `~/.deepagents/`) and treat them as evidence.

---

## 5. The Configuration

### 5.1 Model Selection: DeepAgents (verified)

DeepAgents is **model-agnostic** — the model is a configuration choice, not a hard dependency:

- **CLI (deepagents-cli):** the documented default is **Anthropic Claude Sonnet 4** (flag — confirm against your installed version's help; CLI releases change defaults). You select the provider by exporting its key: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or `GOOGLE_API_KEY` (verified — CLI docs list exactly these). Keys can live in the environment or a `.env` file in the working directory. The CLI docs also document `DEEPAGENTS_MODEL`-style environment configuration in some builds (flagged — the env-var surface varies by version; `deepagents help` is authoritative).
- **Harness (create_deep_agent):** pass `model="provider:model"` — e.g. `openai:gpt-5.5` (README example), `anthropic:claude-sonnet-4`, `google:gemini-2.5-pro`, or any tool-calling LangChain chat model including self-hosted (Ollama/vLLM/llama.cpp — see [../on_prem_llm_deployment_guide.md](../on_prem_llm_deployment_guide.md)). Requires the matching `langchain-*` provider package (`langchain-openai`, `langchain-anthropic`, `langchain-google-genai`, ...).
- **Tool-calling is mandatory** — DeepAgents is a tool-calling agent; a model without reliable tool calling will fail the smoke test. Frontier models and current open-weight models qualify (PyPI FAQ, verified).

*Rule of thumb for this environment: **Copilot CLI = the GitHub-side models on your Copilot plan; DeepAgents = whatever your API key pays for.** Never assume the Copilot subscription covers DeepAgents' tokens — it does not (verified pattern-flag: the two billing planes are entirely separate).*

### 5.2 The Copilot CLI Configuration (verified-flagged)

The Copilot CLI is comparatively configuration-light:

- `copilot auth status` — account and entitlement check (verified).
- `copilot config` — preferences (model selection, keybindings, session options) in newer CLI versions; **flag: the options and even the command's existence vary by release** — run `copilot config --help` on your version.
- `~/.copilot/` — where the CLI keeps configuration, session data, and customizations (verified — docs.github.com "Copilot CLI reference"). Session transcripts and agent settings live here.
- Model choice inside `copilot agents`/`explore` is governed by your Copilot plan's model lineup (Free tier exposes a limited model set, per the 2026 plan pages — flagged, plan-dependent) and, on managed accounts, by org policy.

### 5.3 The Config Table

| Setting | Where | Verified? | Notes |
|---|---|---|---|
| DeepAgents model (CLI) | Env: `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `GOOGLE_API_KEY` | ✅ CLI docs | Selecting the key selects the provider; default model Claude Sonnet 4 (flag — verify per version). |
| DeepAgents model (harness) | `create_deep_agent(model="provider:model", ...)` | ✅ README | Needs matching `langchain-*` provider package. |
| DeepAgents web search | `TAVILY_API_KEY` | ✅ CLI docs | Required for `web_search` tool. |
| DeepAgents memory | `--agent NAME`; stores under `~/.deepagents/<NAME>/memories/` | ✅ CLI docs | Named agents persist conventions across sessions. |
| DeepAgents approvals | `--auto-approve` (or `Ctrl+T` in-session) | ✅ CLI docs | Off by default; keep off for bank work. |
| DeepAgents sandbox | `--sandbox modal\|daytona\|runloop` + provider key | ✅ CLI docs | Remote execution; see sandboxing guide. |
| Copilot CLI model | `copilot config` / plan lineup / org policy | ⚠️ Flagged | Version- and plan-dependent; `copilot config --help` on your build. |
| Copilot CLI data dir | `~/.copilot/` | ✅ docs.github.com | Config, sessions, customizations. |
| Copilot CLI auth | `copilot auth login` / `auth status` | ✅ docs.github.com | Per-user OAuth; org SSO must be authorized. |

---

## 6. The Worked Example — A Bank-Domain Research Task

### 6.1 The Scenario (familiar context)

You are a Solution Architect at **Crédit Agricole CIB** with a treasury-products research question. The payments team wants a briefing on **ISO 20022 migration in corporate banking** — adoption status, vendor readiness, and what it means for the bank's corporate clients. This is a *long-horizon research task*: many sources, cross-cutting topics, a deliverable document. It is exactly what DeepAgents' hub-and-spoke subagent design is for — and exactly the kind of task you do **not** want to type into an interactive chat in one sitting.

The working directory is your research repo (`~/research`). The DeepAgents CLI will write the briefing as a file; the Copilot CLI will supervise, critique, and commit it. (The repo's AI/LLM guides live under `technology/ai_llm/` — for this example, write the briefing to the repo root area of your choice.)

### 6.2 The Exact Commands (the run)

**Phase 0 — environment (once per machine):**

```bash
# Python 3.11+ (verified requirement)
python3 --version

# DeepAgents CLI — no-install path
uvx deepagents-cli --help          # sanity check; pulls the package

# Keys (CLI default model: Claude Sonnet 4 — flag: verify)
export ANTHROPIC_API_KEY="sk-ant-..."
export TAVILY_API_KEY="tvly-..."   # enables web_search for the briefing
```

**Phase 1 — enter the Copilot CLI agent in the repo:**

```bash
cd ~/research
copilot agents
```

**Phase 2 — delegate the research task.** Paste this prompt into the `copilot agents` session:

```
Run the DeepAgents CLI as a subprocess. If uvx is missing, install it first.
Command:
uvx deepagents-cli --agent treasury-research \
  "Research ISO 20022 migration in corporate banking as of 2026.
   Cover: (1) adoption status by region and by bank tier, (2) the impact
   on corporate clients' payment formats and treasury operations,
   (3) vendor and core-banking readiness, (4) the implications for a
   global investment bank's transaction banking offering.
   Use web_search for current sources, cite them inline, and write the
   final deliverable as a structured markdown briefing at
   iso20022_corporate_briefing.md with an executive summary, findings,
   risks, and a source list."
Approve the file-write and web-search prompts it asks for. If the run
errors, read the error, fix it, and rerun. When it finishes, report the
briefing's executive summary to me.
```

**Phase 3 — what you should see.** The DeepAgents CLI plans the task, writes todos (`write_todos`), spawns **subagents** via the `task` tool (e.g. one per coverage area — the hub-and-spoke pattern), calls `web_search` (Tavily) per area, asks for approval before writing `iso20022_corporate_briefing.md`, and prints a plan/step trace throughout. The Copilot CLI session streams all of it to your terminal. Token usage is visible in-session with `/tokens`.

**Phase 4 — verify and wrap up** (still inside `copilot agents`):

```
Read iso20022_corporate_briefing.md. Check: are the inline citations
consistent with the source list? Are any claims unsupported? Then give me
a 10-line executive summary and suggest a commit message.
```

Then, if the repo policy allows: `Commit the briefing.`

**Phase 5 — reuse (the memory payoff).** Next quarter, the same delegation runs with the same `--agent treasury-research` identity and *remembers* the conventions from this run — source format, section structure, tone — because memories persist under `~/.deepagents/treasury-research/memories/` (verified CLI feature). Teach it once, delegate forever after.

### 6.3 The Lessons — CLI Agents in Practice

1. **Delegation beats conversation for long tasks.** A 20-minute research task with web search, files, and subagents is miserable in an interactive chat and natural as a delegated CLI run. Let the orchestrator orchestrate and the specialist specialize — the exact division of labor in [agentic_workflows_guide.md](../agentic_workflows_guide.md).
2. **Prompts to terminal agents are contracts.** The delegation prompt above is explicit about the *tool*, the *agent identity*, the *deliverable path*, and the *error-recovery instruction*. Vague prompts ("do some research") produce vague artifacts — the same failure mode catalogued in [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md).
3. **Keep the human-in-the-loop gates closed.** DeepAgents asked before writing the file and before each web search. For bank work, approvals are cheap insurance; `--auto-approve` is a convenience for trusted, sandboxed runs only (see [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md)).
4. **Verify the artifact, not the process.** The Copilot CLI's review pass over the produced briefing (citations vs. source list) is where quality is actually decided. Agents produce confident prose; humans (and second agents) verify claims. Grounded-citation hygiene matters — see the RAG-series guides under `rag/` (e.g. [rag/rag_evaluation_methodology_guide.md](rag/rag_evaluation_methodology_guide.md)).
5. **Watch the quotas and the keys.** One long delegation can eat a month of a Copilot Free plan's agent requests (flag — plan quotas), and DeepAgents tokens bill to *your* API key, not Copilot. Both meters are visible; both surprise people who never look.
6. **Memory is the compounding asset.** `--agent treasury-research` made the second run faster and more consistent than the first. Persistent memory is one of the CLI's verified headline features — and one of the most underused.
7. **Terminal agents are observable by construction.** Every plan step, tool call, and subagent spawn streams to stdout. That trace is your audit trail — the observability angle is covered in [agentops_guide.md](agentops_guide.md), and the failure modes to watch for in [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md).

### 6.4 Alternative Worked Tasks (same pattern, different payloads)

The ISO 20022 briefing is one instance of the pattern; the same delegation shell covers a whole class of bank-domain tasks. Swap the task text, keep the shape — *tool, agent identity, deliverable path, error-recovery clause*:

| Task | Delegation prompt (task text) | Deliverable | Notes |
|---|---|---|---|
| Regulatory watch | "Research the 2026 regulatory agenda for AI in financial services (EU AI Act, MAS, HKMA) and write a gap-analysis memo for the AI governance committee." | `ai_regulatory_memo.md` | Subagents per regulator; needs `TAVILY_API_KEY`. |
| Vendor comparison | "Compare three transaction-monitoring vendors for onboarding, pricing, and integration effort; write a scoring matrix." | `tm_vendor_comparison.md` | Good `task`-tool workload — one subagent per vendor. |
| Internal tech review | "Analyze the repository's agent code for failure modes (context drift, tool misuse, missing checkpoints) and propose mitigations." | `agent_code_review.md` | Filesystem tools instead of web; no Tavily needed. Cross-ref [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md). |
| Client briefing prep | "Draft a client-facing explainer on ISO 20022 for corporate treasurers, plain-language, with a glossary." | `client_explainer.md` | Same research, different register — reuse the `--agent treasury-research` memory. |

*Each row is one `copilot agents` session, one delegation prompt, one file to verify — the §6.2 Phase-2 template with a different task string.*

---

## 7. The Troubleshooting

### 7.1 The Common Issues (verified — where possible)

Most failures in this stack land in one of four buckets: **authentication** (the Copilot CLI is not talking to GitHub), **quotas** (the plan or the API key is out of budget), **environment** (wrong Python/Node, missing keys, missing packages), and **agent behaviour** (approvals, sandboxes, memory). The table below is the practical triage; anything not directly verifiable from the primary sources is flagged.

**Auth, first and always.** If `copilot agents` or `copilot suggest` fails with a 401/403 or "not authenticated", the fix is `copilot auth login` again, then `copilot auth status` to confirm the account and entitlement (verified commands). Two subtle variants: org **SSO** (the OAuth approval must include your org, or requests fail at runtime even though login "succeeded") and **managed-account policy** (an org admin has disabled Copilot CLI / agent mode — a policy problem, not a CLI problem).

**Rate limits — flagged, plan-dependent.** The Copilot Free tier (2026 plan pages) caps usage at roughly **2,000 code completions and ~50 chat/agent-mode requests per month**; Business/Enterprise meters are higher but still finite, and GitHub enforces per-hour request throttling across all plans. A single long `copilot agents` session can consume a large share of a Free plan's monthly agent quota — the worked example in §6 is exactly the kind of session that does it. The fix is triage: check the Copilot usage page (`https://github.com/settings/copilot`), wait out the window, or upgrade. DeepAgents has its **own** rate limit to watch — your model provider's API (OpenAI/Anthropic/Google) throttles per-minute token throughput, which surfaces as retryable 429s inside the DeepAgents run. *Flag: exact quota numbers change; the docs pages at the time of writing are the source.*

**Environment.** Python < 3.11 breaks `pip install deepagents` at resolution time (verified — `Requires-Python: >=3.11`). An old Node breaks the Copilot CLI install or produces cryptic npm errors. Missing `TAVILY_API_KEY` makes `web_search` fail inside DeepAgents. Missing provider packages make `create_deep_agent` fail at model-construction time. All of these are one-line fixes (below).

**Agent behaviour.** DeepAgents' approval prompts ("approve file write?") look like hangs if you don't realize the session is waiting — they are *not* hangs (verified: sensitive tools require approval by design). In headless/`--headless` contexts there is no TTY to answer them, so either pass `--auto-approve` (with the blast-radius caveat) or avoid headless mode for file-writing tasks. Sandbox flags fail without the matching provider key (`RUNLOOP_API_KEY`, `DAYTONA_API_KEY`, or `modal setup` — verified CLI docs). Memory problems (agent "forgets" or behaves oddly) are usually fixable with `deepagents reset --agent NAME`.

### 7.2 The Troubleshooting Table

| Issue | Fix | Notes |
|---|---|---|
| `copilot: command not found` | `npm install -g @github/copilot`; ensure npm's global bin is on PATH (`%APPDATA%\npm` on Windows) | Verified package name. `npx @github/copilot` as a no-install fallback. |
| `copilot auth login` fails / browser won't open | Use the device-code flow shown in the terminal; on headless boxes copy the code to any browser | Verified docs behaviour. |
| 401/403 at request time | `copilot auth login` again; `copilot auth status`; check org **SSO** authorization | Auth is per-user/per-machine; org SSO is the classic silent killer. |
| "No active Copilot subscription" | Verify plan on `https://github.com/settings/copilot`; Copilot Free includes CLI (flag: quotas); org-managed accounts need admin enablement | Plan-flagged — terms change. |
| **Rate limit / quota exceeded (Copilot)** | Check usage page; wait for reset; upgrade plan; shorten sessions | Flagged: Free ≈ 2,000 completions + ~50 agent/chat requests/month (2026 docs). One §6-style session can consume the agent quota. |
| **Rate limit (DeepAgents / provider)** | Retry on 429s; raise provider tier; reduce parallelism; use cheaper models | DeepAgents bills your API key — separate meter from Copilot (verified pattern-flag). |
| `pip install deepagents` fails resolution | Upgrade to **Python 3.11+** (verified `Requires-Python: >=3.11`) or pin an older deepagents release that matches your interpreter | Verified from PyPI metadata. |
| `deepagents` / `uvx deepagents-cli` errors "no API key" | `export OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `GOOGLE_API_KEY` (or `.env` in cwd) | Verified CLI docs; key selects provider. |
| `web_search` tool fails in DeepAgents | `export TAVILY_API_KEY=...` | Verified CLI docs — Tavily is the web-search backend. |
| `create_deep_agent` fails at model setup | `pip install langchain-<provider>` (e.g. `langchain-anthropic`, `langchain-openai`) | Verified pattern; model string must match provider package. |
| DeepAgents session appears frozen | It is waiting on an **approval prompt** — check the prompt and approve, or start with `--auto-approve` (caveat: blast radius) | Verified: sensitive tools require approval by default. |
| `--sandbox` errors | Set provider key: `RUNLOOP_API_KEY`, `DAYTONA_API_KEY`, or run `modal setup` | Verified CLI docs (providers: modal, daytona, runloop). |
| Agent "forgets" / misbehaves | `deepagents reset --agent NAME` (clears memory); `--target SOURCE` copies memory from another agent | Verified CLI commands. |
| Copilot CLI blocked by policy | Contact GitHub org admin — agent-mode access can be disabled org-wide | Org-policy issue, not a CLI defect (flagged). |
| Weird CLI behaviour after upgrade | Check `copilot --version`; consult `copilot --help`; config lives in `~/.copilot/` | Verified: command surface evolves across releases (flag). |

---

## 8. The One-Page Summary

**What this guide is.** A practical runbook for running LangChain's **DeepAgents** (the batteries-included, hub-and-spoke agent harness + its terminal CLI) from **GitHub Copilot CLI** (GitHub's terminal-native agent) — two independent agent tools composed at the process level, sharing one repo and one terminal.

**The verified stack, in one breath.** Copilot CLI installs via `npm install -g @github/copilot`, authenticates with `copilot auth login` against your GitHub Copilot plan (Free includes it — with quotas; plan terms flagged), and gives you three surfaces: `copilot explore` (repo Q&A), `copilot suggest` (one-shot shell commands), and `copilot agents` (the agentic loop). DeepAgents installs via `pip install deepagents` (harness; needs Python 3.11+ — verified, and note the correction from 3.10+) and `uvx deepagents-cli` (the CLI; default model Claude Sonnet 4 — flagged), runs on **your own model API key**, and brings planning, a filesystem, approvals, persistent memory, and sub-agent delegation. **There is no official plugin** (flagged) — the integration is a prompt: inside `copilot agents`, tell the terminal agent to run `uvx deepagents-cli --agent <name> "<task>"`, supervise the streamed output, then have it read, critique, and commit the artifacts DeepAgents wrote.

**The ten takeaways.**

1. Two agents, two billing planes — Copilot CLI runs on your Copilot subscription; DeepAgents runs on your model API key. Never conflate them.
2. Python **3.11+** for deepagents (PyPI-verified; the 3.10+ assumption is outdated).
3. Copilot Free includes the CLI, but a long agent session can eat a month's agent quota — plan-flagged, check the usage page.
4. The integration is process composition: `copilot agents` → `uvx deepagents-cli "<task>"` → verify the produced file → commit.
5. DeepAgents' superpower is the hub-and-spoke `task` tool: subagents with isolated contexts do the deep work; the supervisor assembles it.
6. Keep approvals on. `--auto-approve` and `Ctrl+T` exist; for bank work they are the last line of defense (see [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md)).
7. Named agents (`--agent treasury-research`) persist memory under `~/.deepagents/` — the compounding asset for recurring research.
8. Prompts to terminal agents are contracts: tool, agent identity, deliverable path, and error-recovery instructions, spelled out.
9. Verify artifacts, not vibes: have the second agent critique the first agent's file (citations vs. source list) before you trust it.
10. Everything is observable: plan steps, tool calls, and subagent spawns stream to stdout — that trace is your audit trail.

**Where this fits in the series.** The umbrella taxonomy lives in [autonomous_agents_guide.md](autonomous_agents_guide.md); the harness anatomy in [agent_scaffolding_guide.md](agent_scaffolding_guide.md); the failure modes in [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) and [agents_work_fall_apart_guide.md](agents_work_fall_apart_guide.md); the production bar in [production_ready_llm_agents_guide.md](production_ready_llm_agents_guide.md); the containment in [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md); the observability in [agentops_guide.md](agentops_guide.md); the platform context in [enterprise_agentic_platform_architecture_guide.md](enterprise_agentic_platform_architecture_guide.md); the MCP tool surface in [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md); durable execution in [../durable_ai_agent_workflows_guide.md](../durable_ai_agent_workflows_guide.md); workflow patterns in [../agentic_workflows_guide.md](../agentic_workflows_guide.md).

**The final word.** Running DeepAgents from the Copilot CLI is not exotic integration — it is the oldest trick in the terminal: one agent calling another as a subprocess, with a shared directory as the handoff point. Set up both CLIs, delegate with a precise prompt, keep the approvals on, and let the deep agent go deep while the terminal agent watches the door. It works today, with verified commands, and it costs nothing beyond the two plans you already have.

---

## 9. The Glossary

- **DeepAgents** — LangChain's open-source "batteries-included agent harness" (`langchain-ai/deepagents`, June 2025 launch): an opinionated agent built on LangGraph with planning, a filesystem, context management, sub-agent delegation, skills, tools, and persistent memory. Installed via `pip install deepagents`; used via `create_deep_agent` or the Deep Agents CLI.
- **Deep Agents CLI (deepagents-cli)** — LangChain's terminal agent (October 2025): interactive CLI with persistent memory (`~/.deepagents/<agent>/memories/`), built-in tools (`ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`, `shell`, `execute`, `web_search`, `fetch_url`, `task`, `write_todos`), approval gates, and remote sandboxes. Installed via `uvx deepagents-cli`, `uv tool install deepagents-cli`, or `pip install deepagents-cli`.
- **GitHub Copilot CLI** — GitHub's terminal-native AI agent, distributed as the npm package `@github/copilot`; uses your GitHub account's Copilot subscription to answer questions, suggest/explain commands, and run agentic coding sessions in the terminal.
- **copilot** — The CLI's binary/entry command (`copilot explore`, `copilot suggest`, `copilot agents`, `copilot auth ...`, `copilot --version`).
- **copilot auth** — The CLI's authentication command group: `copilot auth login` (browser OAuth / device-code flow) and `copilot auth status` (account + entitlement check).
- **copilot explore** — Interactive, repo-grounded Q&A mode of the Copilot CLI ("chat with your repo").
- **copilot suggest** — Non-interactive one-shot mode: describe a task in English, get a shell command; companion `copilot explain` explains a command you provide.
- **copilot agents** — The agentic mode of the Copilot CLI: a full plan → edit → run → iterate loop in the terminal; interactive by default, one-shot with `--prompt "..."`.
- **npm** — The Node.js package manager; ships with Node; used to install the Copilot CLI globally (`npm install -g @github/copilot`).
- **Node** — The JavaScript runtime that Copilot CLI is built on; current LTS (22+) recommended (flag: minimum has crept up across releases).
- **Python** — The language DeepAgents is written in; **3.11+ required** by current `deepagents` releases (PyPI-verified `Requires-Python: >=3.11`).
- **pip** — Python's package installer; `pip install deepagents` installs the harness, `pip install deepagents-cli` the CLI.
- **uv** — The fast, modern Python package/project manager (Astral); `uv add deepagents`, `uv tool install deepagents-cli`.
- **uvx** — uv's runner for tools without a global install; `uvx deepagents-cli` is the canonical no-install way to run the Deep Agents CLI.
- **deepagents run** — The Deep Agents CLI's task-execution invocation (`deepagents run "<task>"`, with `--headless` for non-interactive runs); corroborated in LangChain CLI material (flag: confirm exact flags via `deepagents help` on your version).
- **model** — The LLM driving an agent. Copilot CLI models come from your Copilot plan; DeepAgents models come from your own API key (`openai:gpt-5.5`, `anthropic:claude-sonnet-4`, ...) and must support tool calling.
- **LangChain** — The framework ecosystem (models, tools, integrations) that DeepAgents is built on; provider packages like `langchain-openai` / `langchain-anthropic` connect models to the harness.
- **LangGraph** — The graph runtime underneath DeepAgents (streaming, persistence, checkpointing); DeepAgents is an opinionated harness *on top of* it; any LangGraph graph can be dropped in as a subagent.
- **hub-and-spoke** — DeepAgents' delegation architecture: a supervisor (hub) delegates to subagents (spokes) with isolated context windows, then assembles their results — the `task` tool in the CLI.
- **subagent** — A delegated, context-isolated agent spawned by the supervisor; the unit of parallelism and context hygiene in hub-and-spoke designs.
- **tool** — A function the agent can call (filesystem ops, shell, web search, HTTP, MCP servers); tools are the agent's interface to the world and the primary attack/containment surface (see [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md)).
- **CLI** — Command-Line Interface: text-based program operated from a terminal, as opposed to a GUI.
- **terminal** — The text interface (bash/zsh/PowerShell) where both CLIs run; interactive sessions and approval prompts need a real TTY.
- **authentication** — Proving identity: `copilot auth login` proves your GitHub identity for Copilot; your model API keys prove identity to DeepAgents' model providers. Two separate authentication domains in this stack.
- **rate limit** — A cap on usage: Copilot plans cap monthly completions/agent requests (flag: Free ≈ 2,000 completions + ~50 agent/chat requests per month, 2026 docs); model providers cap tokens per minute. Both appear as errors at the worst moments.

---

## 10. Appendix — Sources and Verification Notes

**Primary sources (verified against these):**

- `langchain-ai/deepagents` GitHub repository README — fetched directly at the time of writing. Verified: `uv add deepagents` quickstart, `create_deep_agent` signature and model-string format, feature set (sub-agents, filesystem, context management, shell access, persistent memory, HITL, skills, tools/MCP), model-agnostic FAQ, MIT license, LangGraph base, "Deep Agents Code" note.
- PyPI metadata for `deepagents` (v0.7.6) — fetched directly. Verified: `Requires-Python: <4.0,>=3.11` (hence the **Python 3.11+ correction**), package summary ("Production-ready, extensible agent harness with built-in filesystem and context management, sub-agent delegation, skills, and long-term memory").
- Deep Agents CLI documentation ("Use the CLI", mirrored at docs.langchain.com) — fetched directly. Verified: `uv tool install deepagents-cli` / `uvx deepagents-cli` / `pip install deepagents-cli` / `uv add deepagents-cli`; default model Claude Sonnet 4 (flag — verify per version); `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `GOOGLE_API_KEY` / `TAVILY_API_KEY`; built-in tool list and approval requirements; `--agent`, `--auto-approve`, `--sandbox modal|daytona|runloop`, `--sandbox-id`, `--sandbox-setup`; CLI commands (`list`, `reset`, `help`); slash commands (`/tokens`, `/clear`, `/exit`); `!` shell escapes; memory directory layout.
- `docs.github.com` Copilot CLI documentation (how-tos: install, authenticate, use; CLI reference) — via targeted search of the official docs. Verified: npm package `@github/copilot`; `copilot auth login` / `copilot auth status`; `explore` / `suggest` / `agents` command family; `~/.copilot` data directory; browser OAuth + device-code flow.
- LangChain blog — "Deep Agents" (June 2025 launch) and "Introducing Deep Agents CLI" (October 2025) — verified via search snippets and the docs mirror.
- GitHub blog — "GitHub Copilot CLI 101" (November 2025) — verified via search snippet (terminal agent framing, starter prompts).

**Flagged / not directly verifiable (stated honestly in the guide):**

- Exact Copilot Free-tier quotas (≈2,000 completions + ≈50 chat/agent requests/month) and plan lineup — from 2026 docs/pricing pages; **plan terms change — verify at setup**.
- Copilot CLI minimum Node version (22+ per community cross-checks; "current LTS" is the safe rule).
- The CLI's default model being Claude Sonnet 4 — per CLI docs mirror; **verify via `deepagents help` on your installed version**.
- The exact `deepagents run` flag surface (`--headless` etc.) — corroborated in LangChain CLI material and community write-ups; **`deepagents help` is authoritative for your version**.
- `copilot config` / model-selection options — version-dependent; run `copilot config --help`.
- The existence of an official Copilot CLI ↔ DeepAgents plugin — **none found in either project's docs at the time of writing**; the integration pattern in §4 is process composition, presented as such.
- `DEEPAGENTS_MODEL`-style environment configuration — documented in some CLI builds; version-dependent.

**Verification method note:** `web_extract` was degraded (search-only backend) during this guide's production, so primary sources were verified via (a) direct `curl` fetches of the DeepAgents repo README and PyPI JSON metadata, (b) a direct fetch of the Deep Agents CLI docs mirror, and (c) targeted `web_search` queries restricted to the official domains (github.com/langchain-ai, docs.github.com, langchain.com, pypi.org). Anything that could not be confirmed against those is explicitly flagged in the text.

---

## Appendix A — The Quick Command Cheat Sheet

Everything in one place (all commands verified as discussed in the body; flags marked ⚠️ are version/plan-dependent — confirm with `--help` / docs at install time):

```bash
# ---------- Copilot CLI ----------
npm install -g @github/copilot          # install (verified)
copilot --version                       # verify binary
copilot auth login                      # browser OAuth / device code (verified)
copilot auth status                     # account + entitlement (verified)
copilot explore                         # interactive repo Q&A
copilot suggest "..."                   # one-shot shell command (verified)
copilot agents                          # interactive agentic loop (verified)
copilot agents --prompt "..."           # one-shot agent run (verified)
copilot config                          # preferences ⚠️ version-dependent
# session: Ctrl+C interrupt · /exit or Ctrl+D quit · /help · ~/.copilot/ data dir

# ---------- DeepAgents harness ----------
python3 --version                       # needs 3.11+ (verified PyPI requirement)
pip install deepagents                  # harness (verified README/PyPI)
uv add deepagents                       # uv equivalent (verified README)
pip install langchain-anthropic         # provider package (verified pattern)
from deepagents import create_deep_agent
agent = create_deep_agent(model="openai:gpt-5.5", tools=[...])   # README pattern

# ---------- Deep Agents CLI ----------
uvx deepagents-cli                      # run without installing (verified docs)
uv tool install deepagents-cli          # persistent install → `deepagents` (verified)
pip install deepagents-cli              # pip path (verified docs)
export ANTHROPIC_API_KEY="sk-ant-..."   # provider key (verified; default model Claude Sonnet 4 ⚠️)
export TAVILY_API_KEY="tvly-..."        # web_search backend (verified)
deepagents run "<task>" --headless      # one-shot task run ⚠️ flags per version
deepagents --agent NAME                 # named agent + persistent memory (verified)
deepagents list / help                  # CLI commands (verified)
deepagents reset --agent NAME           # clear memory (verified)
# flags: --auto-approve · --sandbox modal|daytona|runloop · --sandbox-id ID · --sandbox-setup PATH
# in-session: /tokens · /clear · /exit · !<shell cmd> · Ctrl+T auto-approve toggle

# ---------- The delegation (the whole integration in one line) ----------
copilot agents --prompt "Run: uvx deepagents-cli --agent research \
  '<task: outcome, sources, deliverable path>' — fix errors, rerun, then summarize."
```

---

*End of guide — verified as of the writing date; re-verify the flagged items (plan quotas, Node minimum, default models, CLI flag surfaces) against the primary sources above before production use.*
