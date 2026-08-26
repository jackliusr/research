# Ollama vs Xinference vs LocalAI: Local LLM Serving Platforms — Comprehensive Comparison Guide

> **Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** LLM/AI Engineering Guides
> **Companion Guides:** [LLM Application Frameworks](llm_frameworks_comparison_guide.md) · [RAG Frameworks](rag/rag_frameworks_comparison_guide.md) · [Enterprise AI Gateway](../enterprise_ai_gateway_guide.md) · [LLM Optimization](llm_optimization_complete_guide.md) · [Scalable AI Deployment](scalable_ai_deployment.md) · [Fine-Tuning Frameworks](fine_tuning_frameworks_comparison_guide.md)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Introduction: Why Local LLM Serving](#1-introduction-why-local-llm-serving)
2. [The Landscape: Three Platforms at a Glance](#2-the-landscape-three-platforms-at-a-glance)
3. [Ollama — The Most Popular Local LLM Runner](#3-ollama--the-most-popular-local-llm-runner)
4. [Xinference — The Distributed Inference Framework](#4-xinference--the-distributed-inference-framework)
5. [LocalAI — The OpenAI API Drop-In](#5-localai--the-openai-api-drop-in)
6. [Head-to-Head Comparison](#6-head-to-head-comparison)
7. [Detailed Features Comparison](#7-detailed-features-comparison)
8. [Deployment Scenarios](#8-deployment-scenarios)
9. [Banking-Specific Deployment](#9-banking-specific-deployment)
10. [Selection Guide](#10-selection-guide)
11. [Alternatives: The Broader Serving Landscape](#11-alternatives-the-broader-serving-landscape)
12. [Conclusion](#12-conclusion)

---

## 1. Introduction: Why Local LLM Serving

Running open-source LLMs on infrastructure you control — your own servers, GPUs, or even a laptop — has moved from hobbyist curiosity to a mainstream architectural option for enterprises. For a bank, the question is rarely "can we call an API instead?" but "can we afford *not* to control where our data goes and how our models behave?"

### 1.1 Data sovereignty

**When prompts and responses never leave your premises, the data-sovereignty problem disappears by construction.** Cloud LLM APIs require sending text to a third-party data center — in many cases across borders, subject to foreign legal regimes. For a bank handling client financial data, trade information, and internal strategy, that transfer itself is the risk. Local serving means the model, the inference engine, and the data all stay inside the corporate network boundary, behind the bank's own firewall. This is the single most cited reason financial institutions adopt local LLM serving.

### 1.2 Privacy and compliance

- **GDPR (EU)** — personal data processed by an LLM API is a "transfer" to a processor; local inference removes the transfer and the associated Article 28 processor agreements, DPA chains, and Schrems II complications.
- **PDPA (Singapore)** — the Personal Data Protection Act requires organisations to protect personal data in their possession and to obtain consent for purpose; keeping inference on-prem makes the accountability chain short and auditable.
- **Internal policies** — banks typically restrict which tools employees may paste data into; a self-hosted model is an approved internal system, not an unvetted external service.

### 1.3 Cost at scale

**Per-token API pricing does not amortise.** At low volume, GPT-class APIs are cheap; at millions of tokens per day across thousands of employees, the per-token bill becomes a line item that finance questions. Local inference has high fixed cost (GPUs, power, cooling, engineering) and near-zero marginal cost. Break-even analysis for a mid-size bank usually lands between a few hundred thousand and a few million tokens per day, depending on model size and hardware — after which on-prem is cheaper per token by an order of magnitude.

An illustrative comparison for a sustained 1,000-seat assistant workload:

| Cost item | Cloud API (per-token) | On-prem (fixed + marginal) |
|---|---|---|
| LLM inference | ~$0.5–3 per 1M tokens (7B–70B class) | ~$0 after hardware |
| Embeddings | ~$0.02–0.13 per 1M tokens | ~$0 after hardware |
| Hardware (24 GB GPU server, 3-yr TCO) | — | ~$20–40k all-in |
| Engineering (deploy, monitor, govern) | ~$0 | ~0.5–1 FTE initially, then light ops |
| Compliance overhead (DPA, transfer) | Non-trivial | ~$0 |
| Marginal cost per extra token | Linear | ~$0 |

The crossover point is workload-specific — model it with your own token volumes before committing, but the direction is clear at sustained scale.

### 1.4 Offline and air-gapped operation

Banks run environments with no outbound internet at all — segregated networks for production, DR sites, and regulated workloads. A local serving platform runs entirely inside that boundary: models are pre-downloaded (or pre-staged on media), container images are mirrored into an internal registry, and the inference API never needs an external call. This also removes the availability dependency on a third-party SaaS. The operational discipline for air-gapped deployment:

- **Stage everything in advance** — download model weights (GGUF), container images, and platform binaries on a connected machine; scan and transfer via approved media or a one-way data diode.
- **Mirror registries** — run an internal container registry (Harbor/Nexus) and point installers at it; `OLLAMA_MODELS`, Xinference's registry config, and LocalAI's gallery config all support internal mirrors.
- **No phone-home** — verify at the firewall and in configuration that no telemetry, update checks, or license pings leave the network.
- **Test the install path** in a pre-production replica before the real air-gapped cutover — the first deployment in a closed network is not the place to discover a missing dependency.

### 1.5 Latency control

Cloud APIs add network round-trips, rate-limit queues, and provider-side congestion. Local inference on a GPU delivers first-token latency in tens of milliseconds and stable tail latencies (p95) that you can measure and budget for — critical for interactive copilots, real-time assistants, and trader-facing tools where seconds matter.

### 1.6 Customisation freedom and no vendor lock-in

Open weights mean you can quantise, fine-tune (LoRA/QLoRA), prune, or distill the model; open serving platforms mean you can patch the engine, change the scheduler, or swap models at will. There is no deprecation notice, no pricing change, no model-retirement deadline imposed by a vendor. The stack is yours to evolve.

### 1.7 Regulatory requirements (MAS TRM and AI governance)

Singapore's MAS Technology Risk Management (TRM) guidelines expect financial institutions to manage technology risk across the full lifecycle — including AI systems. Local serving fits this posture:

- **Asset and data governance** — model weights and data stay inventoried on premises; no unmanaged external data flows.
- **Change and release management** — model versions are controlled artifacts deployed through the bank's change process.
- **Audit trail** — inference logs, model versions, and system access are under the bank's own logging and SIEM tooling.
- **AI governance frameworks** (MAS Fairness, Ethics, Accountability and Transparency, and internal AI policies) — a self-hosted model with controlled deployment, monitoring, and human oversight is materially easier to evidence compliance for than an opaque external API.

In TRM terms, the serving platform becomes a managed technology asset: it gets a system-inventory entry, change management through the normal pipeline, defined data flows (no undocumented egress), and operational-resilience requirements (backup, DR, capacity) like any other critical system. That is precisely the posture regulators expect to see documented.

The three platforms in this guide — Ollama, Xinference, and LocalAI — are the most popular open-source tools for that local serving layer. Each takes a different trade-off between simplicity, breadth, and production readiness.

---

## 2. The Landscape: Three Platforms at a Glance

| | **Ollama** | **Xinference** | **LocalAI** |
|---|---|---|---|
| **One-liner** | Simplest way to run local LLMs | Distributed inference for every model type | OpenAI API drop-in, container-native |
| **Language** | Go | Python | Go (+ Python backends) |
| **Created by** | Jeffrey Morgan (jjm), 2023 | Xorbits team (open source), 2023 | Ettore Di Giacinto (mudler), 2023 |
| **Primary engine** | llama.cpp | vLLM / GGML / transformers | llama.cpp (+ vLLM, whisper, diffusers…) |
| **Default port** | 11434 | 9997 | 8080 |
| **Distributed** | No | Yes (multi-node cluster) | Limited (container/K8s scaling) |
| **Model types** | LLM, vision, embeddings | LLM, embeddings, rerank, image, audio, video | LLM, embeddings, image, audio, TTS/STT |
| **Community** | Largest (90k+ GitHub stars, mid-2026, approximate) | Smaller but active | Smaller but active |
| **Best for** | Desktop, prototyping, quick start | Enterprise multi-model, distributed serving | OpenAI migration, containers, ARM/edge |

All three expose an OpenAI-compatible API, all three run on CPU and GPU, and all three are Docker-deployable. The differences are in how much they abstract, how far they scale, and how broad their model coverage is.

---

## 3. Ollama — The Most Popular Local LLM Runner

**Ollama** ([github.com/ollama/ollama](https://github.com/ollama/ollama), [ollama.com](https://ollama.com)) is the default answer to "how do I run an LLM on my machine?" Created by Jeffrey Morgan (jjm) in 2023, written in Go on top of llama.cpp, it turned local LLM inference from a compile-and-configure exercise into a two-word command.

### 3.1 Architecture

Ollama is a client/server design: a lightweight Go daemon manages models and runs inference (via llama.cpp), while a CLI and REST API talk to it. Models are stored in a content-addressed layer store (similar in spirit to Docker's image layers) under `~/.ollama/models` (or `OLLAMA_MODELS`). Because the runtime is Go with a single native binary, installation is a one-command script on macOS, Linux, and Windows — no Python environment, no dependency hell.

### 3.2 The UX that made it famous

```bash
ollama run llama3.2        # pull if needed, then interactive chat
ollama pull qwen2.5:7b     # one-command model download
ollama list                # installed models
ollama show llama3.2       # details: params, context, system
ollama cp llama3.2 my-copy # duplicate a model under a new tag
ollama rm my-copy          # delete
```

The `ollama run` experience (interactive REPL with streaming, `/bye`, `/set` commands) is what converted a generation of developers. Underneath, every command maps to a simple REST API, so the CLI is a thin wrapper over the same interface applications use.

### 3.3 Model library and the Modelfile

- **Library** — [ollama.com/library](https://ollama.com/library) hosts thousands of models in GGUF format (Llama, Qwen, Mistral, DeepSeek, Gemma, Phi, and by mid-2026 also Kimi-K2.6, GLM-5.2, gpt-oss and more), each with tagged variants. Tags encode quantization: `llama3.2:3b-q4_K_M`, `qwen2.5:7b-instruct-q8_0`, and so on.
- **Modelfile** — a Dockerfile-for-models concept that lets you build custom models declaratively:

```dockerfile
FROM llama3.2:3b
PARAMETER temperature 0.7
PARAMETER num_ctx 8192
SYSTEM "You are a compliance assistant for a European investment bank. Answer only from provided context."
TEMPLATE """{{ .Prompt }}"""

# build: ollama create my-bank-assistant -f Modelfile
```

Modelfile supports `FROM`, `PARAMETER` (temperature, top_p, num_ctx, stop…), `SYSTEM` (system prompt), `TEMPLATE` (chat template), `ADAPTER` (LoRA adapters), and license metadata. It is the standard way to bake a system prompt and inference parameters into a versioned, shareable artifact.

### 3.4 API compatibility

Ollama serves an OpenAI-compatible API at `http://localhost:11434/v1` (`/v1/chat/completions`, `/v1/embeddings`, `/v1/models`) plus its native API (`/api/generate`, `/api/chat`, `/api/embed`, `/api/ps`, `/api/show`). Both support streaming. The native API adds `keep_alive` — how long a loaded model stays resident in memory between requests — which is the main lever for interactive latency. Any OpenAI SDK works by pointing `base_url` at `http://<host>:11434/v1`:

```bash
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.2",
    "messages": [{"role": "user", "content": "Explain MAS TRM in one paragraph."}],
    "stream": false
  }'
```

On the native API, `keep_alive` controls how long a model stays resident between requests — set it high for interactive latency, low for memory-constrained multi-model hosts:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Summarise this document.",
  "stream": false,
  "keep_alive": "30m"
}'
```

### 3.5 Multimodal, embeddings, quantization

- **Vision** — LLaVA, Llama 3.2 Vision, Qwen-VL and others run through the same `ollama run` flow, accepting image paths in chat.
- **Embeddings** — `nomic-embed-text`, `mxbai-embed-large`, `bge-m3` etc. via `/api/embed` or the OpenAI-compatible `/v1/embeddings` — a favourite for local RAG stacks.
- **Quantization** — GGUF quants from `q2_K` to `q8_0`, plus `fp16`; `ollama pull` automatically downloads the layers you need and deduplicates shared layers across models.

### 3.6 Configuration and deployment

- **Environment variables** — the most-used knobs:

| Variable | Purpose |
|---|---|
| `OLLAMA_HOST` | Bind address/port (default `127.0.0.1:11434`) |
| `OLLAMA_MODELS` | Model storage directory |
| `OLLAMA_NUM_PARALLEL` | Concurrent requests per loaded model |
| `OLLAMA_KEEP_ALIVE` | Default model residency time |
| `OLLAMA_MAX_LOADED_MODELS` | How many models stay loaded in VRAM |
| `OLLAMA_CONTEXT_LENGTH` | Default context window |
| `OLLAMA_LOAD_TIMEOUT` | Model load timeout |

- **Docker** — official image (`ollama/ollama`) with GPU passthrough via NVIDIA Container Toolkit, typically paired with Open WebUI:

```yaml
# docker-compose.yml
services:
  ollama:
    image: ollama/ollama
    ports: ["11434:11434"]
    volumes: ["./ollama-models:/root/.ollama"]
    environment:
      OLLAMA_HOST: "0.0.0.0:11434"
      OLLAMA_NUM_PARALLEL: "4"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    ports: ["3000:8080"]
    environment:
      OLLAMA_BASE_URL: "http://ollama:11434"
```
- **Open WebUI** — the most popular UI for local LLMs pairs with Ollama out of the box (also works with OpenAI-compatible endpoints), giving chat, RAG, and admin features.
- **Ollama Cloud** — a recent addition offering hosted models, with local-first sync and a web playground; for banking it matters mainly as a dev convenience — production stays local.

### 3.7 Strengths and weaknesses

**Strengths:** easiest setup in the category; huge model library with quantized variants; excellent macOS/Windows support (Metal acceleration); built-in OpenAI-compatible API; Modelfile customisation; enormous community and ecosystem (Open WebUI, Continue, Cline, LangChain/LlamaIndex integrations all treat Ollama as a first-class target).

**Weaknesses:** single-node only (no distributed inference); the server is tuned for interactive, single-user-ish workloads, not high-concurrency production serving; no native model registry/versioning beyond tags; no built-in fine-tuning; less suited to serving embedding + rerank + image models side by side at scale.

---

## 4. Xinference — The Distributed Inference Framework

**Xinference** ([github.com/xorbitsai/inference](https://github.com/xorbitsai/inference), [xorbits.io](https://xorbits.io)) — Xorbits Inference — is a Python-based distributed inference framework. Its pitch: "Swap GPT for any LLM by changing a single line of code" — one unified, production-ready inference API across every model family, on cloud, on-prem, or laptop.

### 4.1 Architecture

Xinference is built around a **supervisor + worker** model. A supervisor coordinates; workers (on one or many machines) hold models in memory and serve requests. You can start with `xinference-local` on a single box, then scale to a multi-node cluster without changing the client API. Because it is Python-native, it sits naturally in a data-science/ML team's stack and integrates with the Hugging Face ecosystem directly.

```bash
pip install "xinference[all]"
xinference-local --host 0.0.0.0 --port 9997   # single node
# or register workers across machines:
xinference-supervisor --host 0.0.0.0 --port 9997
xinference-worker --supervisor http://supervisor:9997 --host 10.0.0.5
```

### 4.2 Model type coverage — the broadest in this comparison

| Category | Examples |
|---|---|
| LLM | Llama 3.1 8B/70B, Qwen 2.5 7B/72B, DeepSeek, Mistral, GLM, gpt-oss |
| Embedding | bge-m3, text2vec, nomic-embed-text |
| Rerank | bge-reranker, jina-reranker |
| Text-to-image | Stable Diffusion, SDXL, FLUX |
| Text-to-audio / TTS | ChatTTS, CosyVoice, MeloTTS |
| Text-to-video | text2video models |
| Multimodal (vision) | Qwen-VL, LLaVA, InternVL |
| Audio (STT) | Whisper (large-v3 etc.) |

No other platform in this guide serves rerank, image, audio, and video models from the same API. That breadth is the main reason teams choose Xinference.

### 4.3 Distributed deployment

The killer feature: **one cluster, many machines.** Model scheduling, placement, and load distribution are handled by the supervisor; a model can be sharded across GPUs (tensor parallelism via vLLM) or served from whichever worker has capacity. Heterogeneous hardware is supported — a worker with an A100 and a worker with CPU-only can coexist in the same cluster, with models placed on the hardware they need:

```bash
# shard a 70B model across 4 GPUs on the cluster using vLLM
xinference launch --model-name llama-3.1-70b-instruct \
  --engine vLLM \
  --tensor-parallel-size 4 \
  --quantization AWQ

# CPU-only embedding model placed on a worker without GPUs
xinference launch --model-name bge-m3 --model-type embedding --device cpu
```

The supervisor also handles failure and re-placement: if a worker dies, its models are re-scheduled onto surviving workers (subject to configured placement rules), and the client API stays unchanged — an availability property the single-node tools cannot offer.

### 4.4 Built-in model hub and management

- **Model hub** — a curated built-in registry (`xinference list`) covering every category; one command installs and launches a model with sensible defaults.
- **Versioning** — model registry tracks versions, so rollbacks and approval workflows are possible.
- **Lifecycle** — download, cache, launch, terminate, and monitor through CLI, UI, or API.

```bash
xinference launch --model-name llama-3.1-8b-instruct --size-in-billions 8 --quantization q4_K_M
xinference list                 # running models
xinference stop --model-uid my-llm
```

### 4.5 Engines and quantization

Engine selection is per-model: **auto** (framework picks), **vLLM** (high-throughput, continuous batching, PagedAttention), **GGML** (llama.cpp-based GGUF), or **transformers** (HF reference). Quantization options span GGUF (q2–q8), GPTQ, AWQ, FP16, and INT8. LoRA fine-tuning (SFT) is supported through the framework — you can train and serve adapters without leaving the platform.

### 4.6 API and SDKs

OpenAI-compatible endpoints on `localhost:9997/v1` cover chat completions, embeddings, rerank, image generation, audio transcription, and audio speech — plus a native REST API and first-party Python and JavaScript SDKs. Streaming and async are supported throughout.

```bash
curl http://localhost:9997/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "llama-3.1-8b-instruct",
       "messages": [{"role": "user", "content": "Hello"}]}'

# rerank — an endpoint the other two platforms do not expose
curl http://localhost:9997/v1/rerank \
  -H "Content-Type: application/json" \
  -d '{"model": "bge-reranker-v2-m3",
       "query": "KYC requirements",
       "documents": ["Doc about KYC", "Doc about pricing"]}'
```

```python
from xinference.client import Client
client = Client("http://localhost:9997")
model = client.get_model("my-llm")
for chunk in model.chat(messages=[{"role": "user", "content": "Hello"}]):
    print(chunk["choices"][0]["delta"].get("content", ""), end="")
```

### 4.7 Strengths and weaknesses

**Strengths:** broadest model-type coverage of the three; true distributed inference; vLLM backend for production throughput (continuous batching, tensor parallelism); built-in model registry with versioning; heterogeneous hardware; LoRA SFT; solid SDKs.

**Weaknesses:** more complex setup (Python environment, supervisor/worker topology); smaller community than Ollama; Python-heavy runtime is heavier to containerise and operate than a single Go binary; less polished UX; documentation has gaps, especially around advanced distributed configuration; ARM support is limited.

---

## 5. LocalAI — The OpenAI API Drop-In

**LocalAI** ([github.com/mudler/LocalAI](https://github.com/mudler/LocalAI), [localai.io](https://localai.io)) is a community-driven local AI platform created by Ettore Di Giacinto (mudler) in 2023. Go-based (with Python backends for some model types), its founding promise: **a drop-in replacement for the OpenAI API** — same endpoints, same payloads, run locally. As of March 2026 it is at 4.0, adding agentic orchestration (Agenthub), a rewritten React UI with Canvas mode, MCP support, and MLX-distributed inference.

### 5.1 Architecture

LocalAI is a single Go binary that proxies to pluggable **backends** — llama.cpp (GGUF, and GPTQ/EXL2/AWQ via its forks), whisper.cpp, diffusers, vLLM (experimental), and more. Configuration is YAML-driven per model: you define a `model.yaml` declaring the backend, quantization, GPU layers, and template, and LocalAI wires the rest. "No GPU required" is a core promise — CPU inference is a first-class citizen.

### 5.2 OpenAI API compatibility — the best in class

LocalAI mirrors the OpenAI surface more completely than the others: `/v1/chat/completions`, `/v1/completions`, `/v1/embeddings`, `/v1/images/generations`, `/v1/audio/transcriptions`, `/v1/audio/speech`, `/v1/moderations`, plus function calling (tool use) and a native API. Migration is a `base_url` change: existing OpenAI SDK code, LangChain/LlamaIndex configs, and tools keep working.

```bash
# function calling / tool use — same payload shape as OpenAI
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama-3.1-8b-instruct",
    "messages": [{"role": "user", "content": "What is the FX rate EUR/USD?"}],
    "tools": [{
      "type": "function",
      "function": {
        "name": "get_fx_rate",
        "parameters": {"type": "object", "properties": {"pair": {"type": "string"}}}
      }
    }]
  }'
```

Applications that relied on legacy OpenAI features (`/v1/completions`, moderation) keep working unchanged — the main reason teams migrating from OpenAI pick LocalAI.

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama-3.1-8b-instruct",
    "messages": [{"role": "user", "content": "Summarise the RAG framework options."}]
  }'
```

### 5.3 Model gallery and P2P sharing

- **AI Gallery** — curated model manifests make installs curl-driven and scriptable; galleries are configurable to point at internal registries — valuable for air-gapped environments:

```bash
local-ai gallery install llama-3.1-8b-instruct
local-ai gallery list
# point the gallery at an internal mirror for air-gapped fleets:
local-ai gallery add my-bank-gallery https://gallery.internal.bank/models.yaml
```

A model's runtime behaviour is declared in `model.yaml` — backend, quantization, GPU offload, template:

```yaml
name: llama-3.1-8b-instruct
backend: llama.cpp
parameters:
  model: llama-3.1-8b-instruct.Q4_K_M.gguf
  context_size: 8192
  gpu_layers: 99
  temperature: 0.2
template:
  chat: chatml
```
- **P2P model sharing** — decentralized model distribution over IPFS-based transport, so fleets can fetch models from peers instead of a single registry.

### 5.4 Backends and model types

LLMs (GGUF, GPTQ, EXL2, AWQ), embeddings, image generation (Stable Diffusion, FLUX), audio transcription (Whisper), text-to-speech (Piper, Coqui), vision models, and function calling — all behind one API. Fine-tuning is supported via llama.cpp and other backends. Vector database integrations and an included WebUI round out the feature set.

### 5.5 Container-first and broad hardware support

LocalAI ships container images for all architectures — x86, ARM, Apple Silicon — and runs on Raspberry Pi. It is the natural fit for Kubernetes: stateless containers, YAML model configs as ConfigMaps, horizontal scaling at the container level.

```bash
docker run -p 8080:8080 -v $PWD/models:/models \
  -v $PWD/conf:/conf \
  --gpus all \
  quay.io/go-skynet/local-ai:latest
```

### 5.6 Strengths and weaknesses

**Strengths:** best OpenAI API compatibility (broadest endpoint surface, function calling); broadest backend support; container-native with K8s scaling; best architecture coverage (ARM, Apple Silicon, x86, Raspberry Pi); model gallery + P2P; includes WebUI and TTS/STT out of the box.

**Weaknesses:** slower release cadence; smaller community; YAML-heavy configuration that is powerful but steep; performance varies by backend and needs tuning; documentation is fragmented across wiki, GitHub, and the site.

---

## 6. Head-to-Head Comparison

| Dimension | **Ollama** | **Xinference** | **LocalAI** |
|---|---|---|---|
| Language | Go | Python | Go + Python backends |
| Inference engine | llama.cpp | vLLM / GGML / transformers | llama.cpp / vLLM / diffusers / whisper |
| Model formats | GGUF | GGUF, GPTQ, AWQ, FP16 | GGUF, GPTQ, EXL2, AWQ |
| Model types | LLM, vision, embeddings | LLM, embeddings, rerank, image, audio, video | LLM, embeddings, image, audio, TTS |
| API compatibility | OpenAI-compatible | OpenAI-compatible | **Best** OpenAI-compatible (drop-in) |
| Distributed inference | No | **Yes** (multi-node) | Limited (container/K8s scale-out) |
| GPU / CPU | Both (CUDA, ROCm, Metal) | Both (CUDA, CPU) | Both (CUDA, CPU, Metal) |
| Ease of use | **Easiest** | Moderate | Moderate |
| Docker | Yes | Yes | Yes (container-first) |
| UI | Open WebUI integration | Built-in web UI + chat UI | Built-in WebUI |
| Model management | `ollama pull` / library | Model hub + registry (versioned) | AI Gallery (manifests) |
| Fine-tuning | No | LoRA / SFT | Via backends (llama.cpp etc.) |
| Community | **Largest** (90k+ stars) | Smaller | Smaller |
| Best for | Personal / desktop / quick start | Enterprise multi-model / distributed | OpenAI drop-in / container / K8s |

**Reading the table:** Ollama wins on simplicity and ecosystem; Xinference wins on breadth and scale; LocalAI wins on API fidelity and portability. None is strictly better — the right choice depends on the deployment scenario (§8).

---

## 7. Detailed Features Comparison

### 7.1 Model management

**Ollama — the `ollama` command family.** `ollama pull llama3.2` fetches from the library (tags select quantization: `q4_K_M`, `q8_0`, `fp16`); `ollama list` / `rm` / `cp` / `show` handle inventory; layer downloads are automatic and deduplicated across models (shared weights are stored once). Custom models come from a **Modelfile** (`FROM` / `PARAMETER` / `SYSTEM` / `TEMPLATE` / `ADAPTER`) built with `ollama create`, and can be pushed to a registry (`ollama push`) for internal distribution — an Ollama-flavoured analogue of a Docker registry.

**Xinference — the built-in model hub.** The hub is categorised (LLM, embedding, rerank, image, audio, video) with one-command launch: `xinference launch --model-name llama-3.1-8b-instruct`. Models are **versioned** in the registry — a genuine governance advantage (rollback, pinning, approval tracking). Registry endpoints are configurable (point at an internal mirror for air-gapped use), and downloads are cached locally.

**LocalAI — the AI Gallery.** Manifests define models declaratively (`local-ai gallery install <model>`); `model.yaml` per model declares backend, quantization, GPU offload, and templates; GGUF repos from Hugging Face are supported directly. **P2P sharing** lets nodes fetch models from peers — handy for fleets without a central registry.

### 7.2 API compatibility

| Endpoint | Ollama | Xinference | LocalAI |
|---|---|---|---|
| `/v1/chat/completions` | ✅ | ✅ | ✅ |
| `/v1/completions` | — | — | ✅ |
| `/v1/embeddings` | ✅ | ✅ | ✅ |
| `/v1/rerank` | — | ✅ | — |
| `/v1/images/generations` | — | ✅ | ✅ |
| `/v1/audio/transcriptions` | — | ✅ | ✅ |
| `/v1/audio/speech` | — | ✅ | ✅ |
| `/v1/moderations` | — | — | ✅ |
| Function calling | Partial (tool use) | ✅ | ✅ |
| Native API | `/api/*` (generate, chat, embed, ps) | REST + Python/JS SDKs | Native API |
| Streaming | ✅ | ✅ (async too) | ✅ |

Ollama additionally exposes `keep_alive` (model residency between requests) on its native API — an interactive-latency lever the others handle differently. LocalAI's extra endpoints (`/v1/completions`, `/v1/moderations`) matter when migrating apps that used legacy OpenAI features.

### 7.3 Hardware and performance

**Ollama** — GPU acceleration on CUDA, ROCm, and Metal; CPU inference; **automatic partial offload** (model split across GPU and CPU when VRAM is short); NVLink multi-GPU; `OLLAMA_NUM_PARALLEL` for per-model concurrency; `num_ctx` for context size; `keep_alive` / idle-unload for memory management; quantization for VRAM reduction. Performance profile: **single-user/desktop-focused, great interactive latency, not designed for high-concurrency serving** — parallel requests queue behind one loaded model.

**Xinference** — CUDA and CPU; the **vLLM engine** brings continuous batching, PagedAttention, and tensor parallelism for multi-GPU; distributed across machines with model scheduling on heterogeneous hardware; quantizations GGUF/GPTQ/AWQ/FP16/INT8; LoRA support. Performance profile: **designed for production serving** — higher concurrency, throughput-oriented, distributed scale-out.

**LocalAI** — CUDA, CPU, Metal; llama.cpp backends (GGUF, EXL2, AWQ, GPTQ); vLLM experimental; multi-GPU via backend options; performance tuning through YAML (`gpu_layers`, `mmap`, threads). Performance profile: **good single-node, less distributed**; scale-out is container-level (K8s replicas behind a load balancer) rather than model-level sharding.

**Workload-to-platform fit for performance:**

| Workload | Best engine | Platform fit |
|---|---|---|
| Interactive chat, 1–10 users | llama.cpp (Metal/CUDA) | Ollama (easiest), LocalAI |
| RAG pipeline (LLM + embed + rerank) | llama.cpp / transformers | Xinference (one API), LocalAI |
| High-concurrency production chat | vLLM (continuous batching) | Xinference; LocalAI (experimental); vLLM directly |
| 70B+ on multi-GPU | vLLM tensor parallel | Xinference; LocalAI (backends); Ollama (NVLink, limited) |
| Batch summarisation, offline | CPU llama.cpp | Any (Ollama/LocalAI on CPU) |

---

## 8. Deployment Scenarios

### Scenario 1 — Personal desktop (Mac/Windows)

**Winner: Ollama + Open WebUI.** Install the app, `ollama run llama3.2`, done. LocalAI with Docker works but adds YAML ceremony for no benefit; Xinference is overkill. Ollama's Metal support makes it the best Mac experience, and the ecosystem (Open WebUI, Continue, Cline) is designed around it.

### Scenario 2 — Single on-prem server (Linux, one GPU)

All three work. **Ollama** is the simplest path to a chat/embeddings endpoint. **Xinference** pays off when the server must serve multiple model types — LLM + embedding + rerank for RAG — because all three live behind one API and one management plane. **LocalAI** is the pick when the consuming applications are OpenAI-SDK-based and you want zero code change.

### Scenario 3 — Production multi-model serving (enterprise)

**Xinference** (distributed, vLLM throughput, model registry with versions) or **LocalAI** (K8s-native, container replicas, gallery manifests as GitOps config). Ollama is limited here: no distributed inference, no registry, interactive-first scheduling.

### Scenario 4 — Air-gapped / offline (banking)

All three work, with pre-staging as the key discipline: download models and container images once, mirror into internal registries, then deploy with no external calls. **Ollama** is the simplest offline story (one binary, `OLLAMA_MODELS` directory, no Python). **LocalAI** fits offline container fleets (gallery pointed at an internal registry; images in a private mirror). **Xinference** supports offline mode for its model hub but the Python stack is more moving parts to stage.

### Scenario 5 — OpenAI API migration (drop-in replacement)

**LocalAI** first (broadest endpoint surface incl. `/v1/completions`, `/v1/moderations`, function calling). **Ollama** is good enough when the workload is chat + embeddings only, and its `:11434/v1` endpoint accepts standard OpenAI SDKs. Xinference also exposes `/v1`, but LocalAI's fidelity is the point of the platform.

### Scenario 6 — Multi-GPU, high throughput

**Xinference** (vLLM + tensor parallelism, distributed) is the throughput leader. **LocalAI** supports multi-GPU via backend config. **Ollama** handles NVLink multi-GPU but is limited by its interactive-first scheduler and lack of distributed mode.

### Scenario 7 — Edge / Raspberry Pi / ARM

**LocalAI** is the clear edge winner: first-class ARM support, runs on Raspberry Pi, container images for every architecture. **Ollama** ships ARM64 builds (Apple Silicon is its home turf) but Pi-class devices are not its focus. **Xinference** has limited ARM support.

**Scenario summary:**

| Scenario | Primary pick | Runner-up | Notes |
|---|---|---|---|
| 1. Personal desktop | Ollama | LM Studio | Simplest possible start |
| 2. Single on-prem server, one GPU | Ollama | Xinference / LocalAI | Xinference if multi-model-type RAG |
| 3. Production multi-model (enterprise) | Xinference | LocalAI | Registry + vLLM throughput |
| 4. Air-gapped / offline | Ollama (simplest) | LocalAI | Pre-staging discipline is key |
| 5. OpenAI drop-in migration | LocalAI | Ollama | LocalAI for legacy endpoints |
| 6. Multi-GPU high throughput | Xinference | LocalAI | vLLM tensor parallelism |
| 7. Edge / Raspberry Pi / ARM | LocalAI | Ollama (ARM64) | Xinference limited |

---

## 9. Banking-Specific Deployment

### 9.1 Reference stack

A pragmatic on-prem LLM stack for a bank looks like:

```
┌────────────────────────────────────────────────────────┐
│  Clients: Open WebUI / custom UI / existing apps       │
├────────────────────────────────────────────────────────┤
│  Gateway: enterprise API gateway (authN/authZ, SSO,    │
│  rate limits, audit) — see enterprise_ai_gateway_guide │
├────────────────────────────────────────────────────────┤
│  Serving: Ollama OR Xinference OR LocalAI (+ vLLM)     │
│  Models: GGUF-quantized Llama 3.1 8B/70B, Qwen 2.5     │
│  7B/72B, Mistral, DeepSeek                             │
├────────────────────────────────────────────────────────┤
│  RAG layer: frameworks (see rag_frameworks_comparison  │
│  _guide) + vector DB: pgvector or Qdrant               │
├────────────────────────────────────────────────────────┤
│  Observability: Prometheus + Grafana (token throughput,│
│  latency percentiles, GPU utilization, error rates)    │
└────────────────────────────────────────────────────────┘
```

### 9.2 Data sovereignty and security

- **On-prem, no cloud calls** — the serving platform must never phone home; verify no telemetry endpoints escape the network (block egress at the firewall as a control, not a hope).
- **Internal network** — models and clients on the corporate network; inference nodes in a dedicated VLAN with restricted access.
- **API auth** — **none of the three platforms has enterprise-grade authentication built in.** Ollama's server, Xinference's supervisor, and LocalAI's HTTP API all expect to sit behind a gateway or reverse proxy that provides SSO (OIDC/SAML), API keys, mTLS, and rate limiting. Treat the serving tier as an internal-only service; never expose it directly.

Security checklist for the serving tier:

- [ ] Inference nodes in a dedicated VLAN; no inbound from the internet.
- [ ] Egress firewall: deny by default; allow only approved registries/mirrors.
- [ ] All API traffic via the gateway: SSO, API keys, mTLS, rate limits, per-team quotas.
- [ ] Model artifacts signed/hash-pinned; downloads verified before load.
- [ ] Container images from the internal mirror only; image scanning in the pipeline.
- [ ] Secrets (API keys, certs) in the bank's secret manager, never in model config.
- [ ] Inference logs classified and access-controlled like client-data systems.

### 9.3 Model governance

Model registry and versioning (Xinference's native registry; Ollama tags + internal registry; LocalAI gallery manifests in Git), approval workflows before promotion to production, and audit trails (who deployed which model version, when, from what artifact). Pair with the bank's AI governance framework: model cards, risk classification, and sign-off per use case.

Governance checklist:

- [ ] Model catalog with owner, risk class, and approved use cases per model.
- [ ] Version pinning in production; no floating tags in prod configs.
- [ ] Promotion workflow: dev → QA → prod with evidence (eval results, license check).
- [ ] License compliance — verify each model's license permits internal commercial use.
- [ ] Periodic re-evaluation — re-benchmark on model upgrades; retire obsolete versions.
- [ ] Audit log of deployments, config changes, and access to the serving tier.

### 9.4 GPU capacity planning

Budget VRAM for **weights + KV cache + activations + overhead**, scaled by concurrency:

| Model (GGUF) | Weights (approx.) | KV cache (8k ctx, approx.) | Practical footprint |
|---|---|---|---|
| 7–8B q4_K_M | ~4.5 GB | ~1–2 GB | fits 16 GB GPU, comfortably 24 GB |
| 8B q8_0 / fp16 | ~8–16 GB | ~2–4 GB | 24 GB GPU, one model at a time |
| 70B q4_K_M | ~40 GB | ~8–16 GB | multi-GPU (2×80 GB or 4×48 GB) or CPU offload |
| 72B (Qwen) q4 | ~42 GB | ~10–20 GB | multi-GPU with tensor parallelism |

Rule of thumb: 7–8B models on a single 24 GB GPU serve interactive copilots; 70B+ requires multi-GPU (Xinference tensor parallel, LocalAI multi-GPU backends, or Ollama NVLink). Concurrency multiplies KV-cache demand — plan for peak, not average.

### 9.5 Model selection

- **Quantization vs quality** — q4_K_M is the sweet spot for most banking tasks; q8/fp16 for tasks where fidelity matters (contract extraction, regulatory text); 2-bit quants are for demos only.
- **7B–8B for CPU/edge** (CPU-only inference is viable but slow — acceptable for batch, not interactive).
- **70B+ for multi-GPU** where quality is non-negotiable (e.g., document-heavy analysis).
- **Prefer instruct-tuned, and prefer models with strong function-calling** if agentic workflows are planned. Fine-tune (LoRA via Xinference, or via backends) on bank-specific style and domain data where generic models underperform.

### 9.6 Latency and monitoring

Define SLOs per use case (interactive copilot: p95 first-token < 1s; batch summarisation: throughput over latency). Monitor token throughput, latency percentiles, GPU utilization, memory pressure, and error rates via Prometheus/Grafana (Ollama exposes `/api/ps` and metrics; LocalAI has a metrics endpoint; Xinference has built-in monitoring). Watch for KV-cache exhaustion — it degrades silently as context grows.

| Metric | What it catches | Example alert threshold |
|---|---|---|
| Tokens/sec (throughput) | Saturation, batching issues | < 50% of baseline for 10 min |
| p50/p95/p99 TTFT (time-to-first-token) | Interactive-latency regressions | p95 > 2s for copilot SLO |
| p95 inter-token latency | Generation stalls | > 100 ms/token sustained |
| GPU utilization | Under/over-provisioning | < 30% average for 24h → resize |
| GPU VRAM / host memory | KV-cache exhaustion, OOM risk | > 90% for 5 min |
| Error rate (4xx/5xx, timeouts) | Model crashes, overload | > 1% over 5 min |
| Queue depth / model evictions | Concurrency mismatch | Sustained queue > 10 |

### 9.7 Fallback strategy

Design for the on-prem tier failing: a gateway-level **fallback to a cloud API** (with data-loss prevention — only non-sensitive traffic may egress) or a degraded mode (queue jobs, serve from cache). Document the failover path in the runbook; test it quarterly. The reverse also matters: cloud-first apps should be able to fail over *to* on-prem.

### 9.8 Cost and TCO

Compare hardware + power + cooling + engineering vs API per-token pricing over 3–5 years. A single 24 GB GPU server (~$20–40k all-in with support) serving 7B models can replace six figures of annual API spend at sustained volume. Include intangibles: procurement lead times, GPU scarcity, and the cost of model-version churn (retraining/requantizing on each release).

### 9.9 Compliance

- **MAS TRM** — technology risk management across the model lifecycle; local serving puts assets, changes, and operations inside the bank's control framework.
- **Internal AI policy** — approval gates, use-case inventory, human oversight for high-impact outputs.
- **Audit trail** — log prompts/responses (with PII redaction policies), model versions, and access; retain per retention schedules.
- **PII handling in logs** — configure logging to redact or omit personal data; ensure inference logs are classified and access-controlled like any other client-data system.

---

## 10. Selection Guide

### 10.1 Choose Ollama when…

- Simplicity matters most — you want value in minutes, not days.
- Use case is desktop/personal, quick prototyping, or dev environments.
- Single user or low concurrency; interactive chat and embeddings only.
- Team is on macOS/Windows and wants native installers + Metal.
- You want the biggest community, best docs, and the richest ecosystem (Open WebUI, IDE tools, framework integrations).
- You don't need distributed inference, a model registry, or fine-tuning.

### 10.2 Choose Xinference when…

- You need multiple model types behind one API: LLM + embedding + rerank + image + audio.
- Distributed inference across machines is required (multi-node cluster, heterogeneous hardware).
- Production multi-model serving with high concurrency (vLLM continuous batching, tensor parallelism).
- Model versioning/registry and governance matter (rollback, pinning, approval).
- Your team is Python-native and comfortable with a heavier stack.

### 10.3 Choose LocalAI when…

- OpenAI API drop-in compatibility is critical (legacy endpoints, `/v1/completions`, `/v1/moderations`, function calling).
- Deployment is container/K8s-centric and GitOps-driven (YAML model configs as ConfigMaps).
- Hardware breadth matters: ARM, Raspberry Pi, Apple Silicon, x86.
- You need image/audio/TTS in addition to LLM from one service.
- Model gallery manifests + P2P distribution fit your fleet model.
- You're migrating from OpenAI with minimal code change.

### 10.4 Hybrid approach

The pragmatic enterprise pattern: **Ollama for dev/desktop and prototyping** (fastest feedback loop for data scientists and engineers) + **Xinference or LocalAI for production** (registry, throughput, governance). The OpenAI-compatible API means the same application code runs against both tiers — dev on Ollama, prod on Xinference/LocalAI, with the gateway routing by environment.

### 10.5 Decision table

| Requirement | Ollama | Xinference | LocalAI |
|---|---|---|---|
| Ease of setup | ⭐⭐⭐ | ⭐ | ⭐⭐ |
| Model types covered | LLM, vision, emb. | **All** (LLM→video) | LLM, image, audio, TTS |
| Distributed inference | ✗ | **Yes** | Limited (K8s) |
| OpenAI compatibility | Good | Good | **Best** |
| Production readiness | Low–medium | **High** | Medium–high |
| Community/docs | **Largest** | Smaller | Smaller |
| Governance (registry/versioning) | Basic (tags) | **Built-in** | Via gallery/GitOps |
| Best for | Desktop, quick start | Enterprise, distributed, multi-model | OpenAI drop-in, edge, K8s |

---

## 11. Alternatives: The Broader Serving Landscape

### 11.1 llama.cpp — the engine under the hood

The C/C++ inference engine (by Georgi Gerganov) that Ollama and LocalAI build on: maximum control, minimum abstraction. You compile it, quantise models with its tooling, and run `llama-server` or the CLI directly. Best when you need to squeeze performance out of unusual hardware (Apple Silicon, low-end GPUs) or want to understand every knob. Least turnkey. **When to choose:** you need fine-grained control (custom sampling, exotic quantisation, niche hardware) and your team can own the engineering.

### 11.2 vLLM — production high-throughput serving

Python serving engine with PagedAttention, continuous batching, tensor/pipeline parallelism, and an OpenAI-compatible server. The standard for GPU clusters serving large models at scale (it is what Xinference uses as its fast backend). Not for laptops; it wants CUDA GPUs. See the dedicated `serving-llms-vllm` guide in this series. **When to choose:** a dedicated GPU cluster with an ML platform team serving high-concurrency production traffic — this is the throughput ceiling of the local-serving spectrum.

### 11.3 LM Studio

Polished desktop GUI for local models — extremely popular on Mac/Windows. Browse/download models, chat, run a local OpenAI-compatible server, all point-and-click. Like Ollama with a GUI-first philosophy; great for non-engineers and evaluation, less suited to automation and server deployment. **When to choose:** business users or analysts evaluating models on their own laptops without touching a terminal.

### 11.4 Jan

Offline-first desktop app (Electron) positioning itself as a private ChatGPT alternative: chat UI, model management, local API. Good for individual privacy-conscious users; similar positioning to LM Studio with a smaller ecosystem. **When to choose:** a fully offline personal assistant with a polished chat experience and no cloud dependency at all.

### 11.5 GPT4All

Local LLM desktop app by Nomic AI, bundled with a curated model zoo; famous for its simple installer and CPU-friendly models. Historically paired with the Nomic embedding models; now part of the broader local-AI desktop wave. **When to choose:** the simplest possible CPU-only chat experience for non-technical users.

### 11.6 Text Generation WebUI (oobabooga)

The classic feature-rich browser UI for local LLMs: chat, training (LoRA), extensions, and deep llama.cpp/exllamav2 integration. Powerful but fiddly; the power user's choice before Ollama's rise. **When to choose:** you want a single UI that also does LoRA training and deep backend tuning, and you accept the setup effort.

### 11.7 llama-server

llama.cpp's own built-in HTTP server (`llama-server -m model.gguf`). No abstraction at all — you get the OpenAI-compatible endpoint directly from the engine. Ideal when you want llama.cpp performance with no wrapper layer. **When to choose:** you are already building on llama.cpp and want one model exposed as an API without adding a platform.

### 11.8 Llamafile (Mozilla)

Single-file LLM deployment: model + runtime in one executable (~4–5 GB) that runs on multiple OSes without installation. Brilliant for sharing and demos; less suited to multi-model production serving. **When to choose:** distributing a fixed model to many machines (or colleagues) with zero-install friction.

### 11.9 IPEX-LLM

Intel's optimised LLM library (formerly BigDL-LLM): runs PyTorch/transformers models fast on Intel CPUs, GPUs, and Arc — including 4-bit low-precision inference. The choice when your hardware fleet is Intel and you want maximum utilisation of it. **When to choose:** an Intel-only estate (Xeon servers, Arc GPUs) where NVIDIA CUDA tooling is not an option.

### 11.10 TGI (Text Generation Inference)

Hugging Face's production serving solution for GPU clusters: continuous batching, quantization, tensor parallelism, OpenAI-compatible API (recent versions). The HF-ecosystem answer to vLLM; strong on multimodal and tool-calling support, well integrated with HF Hub. **When to choose:** your model lifecycle already runs through the Hugging Face Hub and you want a managed, HF-native serving tier.

### 11.11 SGLang

High-performance serving framework with RadixAttention (prefix caching across requests) and structured output control; often the top of throughput benchmarks for serving large models. More research-flavoured and younger; a serious contender when raw performance on big GPUs is the priority. **When to choose:** maximum throughput and shared-prefix workloads (agents, RAG with long common context) on a dedicated GPU fleet, with a team that can absorb a younger codebase.

### 11.12 Alternatives at a glance

| Tool | Type | Best for | Trade-off |
|---|---|---|---|
| llama.cpp | Engine + server | Max control, any hardware | Most setup, least abstraction |
| vLLM | Serving engine | GPU-cluster production throughput | CUDA-only, not desktop |
| LM Studio | Desktop GUI | Mac/Windows evaluation, non-devs | GUI-first, thin automation |
| Jan | Desktop app | Offline-first personal ChatGPT | Small ecosystem |
| GPT4All | Desktop app | Simple local chat on CPU | Limited to curated models |
| Text Generation WebUI | Web UI | Power-user control, LoRA training | Fiddly setup |
| llama-server | Engine server | Direct llama.cpp API | No management layer |
| Llamafile | Single-file exe | Portable demos/sharing | One model per file |
| IPEX-LLM | Intel library | Intel CPU/GPU fleets | Intel-only |
| TGI | HF serving | HF-ecosystem production | HF-centric |
| SGLang | Serving framework | Max throughput on big GPUs | Younger, less mature |

**Where the big three sit:** Ollama is the abstraction layer *over* llama.cpp; Xinference is a multi-model distributed framework that can use vLLM; LocalAI is a multi-backend OpenAI-compatible gateway. vLLM/TGI/SGLang are the high-end production engines — if your bank has a dedicated GPU cluster and an ML platform team, one of those (with Xinference or LocalAI as the management layer) is the enterprise-grade endpoint of this spectrum.

---

## 12. Conclusion

Local LLM serving is not a niche anymore — for a bank it is a governance decision with architectural teeth. Ollama, Xinference, and LocalAI cover the spectrum from "run a model in five minutes" to "serve every model family across a distributed cluster behind one OpenAI-compatible API".

**The short version:** start with Ollama to build momentum and evaluate models; scale to Xinference when you need breadth (embedding + rerank + LLM + image/audio) and distributed throughput; pick LocalAI when OpenAI API fidelity, containers, and edge/ARM support dominate the requirements. Whatever you choose, put it behind an enterprise gateway, plan the GPU capacity, wire in monitoring, and treat the model catalog as governed artifacts — the platform is the easy part; the governance is the deliverable.

---

*Part of the LLM/AI Engineering Guides series. Related guides: [LLM Application Frameworks](llm_frameworks_comparison_guide.md), [RAG Frameworks](rag/rag_frameworks_comparison_guide.md), [Enterprise AI Gateway](../enterprise_ai_gateway_guide.md), [LLM Optimization](llm_optimization_complete_guide.md), [Scalable AI Deployment](scalable_ai_deployment.md).*
