# Vertex AI vs Gemini — Disambiguating the Two Names, the Four Different Things They Refer To, and How to Choose Between Google's Two Access Paths to the Gemini Models

> **Author:** Jack Liu Shurui  
> **Role:** Solution Architect, Cymbal Bank  
> **Date:** September 2026  
> **Version:** 1.0  
> **Repository:** github.com/jackliusr/research  


## Table of Contents

1. [The Disambiguation](#1-the-disambiguation)
2. [The Model Family](#2-the-model-family)
3. [The Two Access Paths Explained](#3-the-two-access-paths-explained)
4. [The Differences That Decide It](#4-the-differences-that-decide-it)
5. [The Enterprise Controls That Only Exist on the Platform Path](#5-the-enterprise-controls-that-only-exist-on-the-platform-path)
6. [Beyond the Raw Models](#6-beyond-the-raw-models)
7. [Getting From One to the Other](#7-getting-from-one-to-the-other)
8. [The Decision](#8-the-decision)
9. [The Alternatives Frame](#9-the-alternatives-frame)
10. [The Regulated-Institution Angle](#10-the-regulated-institution-angle)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [Closing Summary](#16-closing-summary)


## 1. The Disambiguation

"Vertex AI vs Gemini" is not a comparison between two products. It is a comparison between four different things that share two words, and the confusion is not the reader's fault — Google has renamed almost every part of this stack within the last two years, and the live documentation still carries the old and new names side by side.

**(a) Gemini the model family.** The models themselves — the weights, the tiers, the modalities, the endpoints. `gemini-3.8-flash`, `gemini-3.1-pro-preview`, `gemini-3.5-flash-lite`, the Nano Banana image models, the Veo video models, the Lyria music models, the embedding models. This is a *thing*, not a place. It is what you call, and it is callable from more than one place.

**(b) Gemini the consumer and business products.** The Gemini app at [gemini.google.com](https://gemini.google.com/), the Gemini features embedded in Gmail, Docs, Slides, Sheets and Meet, the Google AI Plus / Pro / Ultra subscription plans sold through Google One, and **Gemini Enterprise app** — the enterprise-facing assistant and agent product sold separately from the developer platform. These are end-user products. They are what your colleagues mean when they say "we already have Gemini." They have nothing to do with your API integration.

**(c) Vertex AI the platform — now renamed Gemini Enterprise Agent Platform.** The Google Cloud service. It *serves* the Gemini models, alongside Google's other first-party models and — critically — alongside third-party and open-weight models it does not own. Around the models it provides the machinery: IAM, VPC Service Controls, CMEK, audit logging, regional endpoints, data residency, quota management, tuning, evaluation, vector search, RAG, and agent infrastructure. **The rename is the single most important naming fact in this guide.**

**(d) The developer-facing API and studio — the direct developer path.** The **Gemini Developer API** at [ai.google.dev](https://ai.google.dev/gemini-api/docs), with **Google AI Studio** at [aistudio.google.com](https://aistudio.google.com/) as its playground. An API key, a quota tier, an endpoint — no Google Cloud project required, no IAM, no VPC. It serves Google's own models only. It is deliberately frictionless, and the friction it removes includes most of the enterprise controls.

### 1.1 What sits where

```text
                    GOOGLE'S GEMINI-RELATED SURFACE

  (a) THE MODEL FAMILY
      Gemini 3.8 Flash / 3.1 Pro / Flash-Lite / Live / TTS
      Nano Banana / Veo 3.1 / Lyria 3.5 / Gemini Embedding 2
      -> the same models are served on BOTH paths (mostly)

        served directly |                    | served via the platform
                        v                    v
  (d) DIRECT DEVELOPER PATH        (c) PLATFORM PATH
      Gemini Developer API             Vertex AI, now renamed
      ai.google.dev                    "Gemini Enterprise Agent Platform"
      Google AI Studio                 cloud.google.com/gemini-enterprise-...
      aistudio.google.com              console.cloud.google.com/agent-platform
      - API key auth                   - ADC / IAM service accounts (or API key)
      - Free / Paid / Enterprise       - Cloud billing, per project, per region
      - NO VPC-SC, CMEK, audit, IAM    - VPC-SC, CMEK, audit logs, residency
      - Google models ONLY             - Google + partner + open models

  (b) CONSUMER / BUSINESS PRODUCTS  (not developer surfaces at all)
      Gemini app (gemini.google.com) - Gemini in Workspace - Gemini Notebook
      Google AI Plus / Pro / Ultra plans - Gemini Enterprise app
      = what your colleagues mean by "we already have Gemini"
```

The model family (a) sits in the middle and is reachable by two doors: the direct developer door (d) and the platform door (c). The consumer products (b) are a separate building with a similar logo on the front.

### 1.2 The rename, stated plainly

**"Vertex AI" is now branded "Gemini Enterprise Agent Platform."** Verified from live material on 2026-09-16:

| Evidence | What it shows |
| --- | --- |
| Docs home at `cloud.google.com/gemini-enterprise-agent-platform` and `docs.cloud.google.com/gemini-enterprise-agent-platform` | Platform documentation now lives under the new name |
| Console section at `console.cloud.google.com/agent-platform/...` | The console path moved too, to `/agent-platform/` |
| `cloud.google.com/vertex-ai/generative-ai/pricing` still serves a live page, titled "Agent Platform Pricing" | Old URLs persist while content is renamed |
| That page refers throughout to "Generative AI on Agent Platform" and "Gemini Enterprise Agent Platform" | The rename is in body text, not just titles |
| The platform model page is "Google models" under "Gemini Enterprise Agent Platform" | Model documentation moved with the platform |
| The SLA index `cloud.google.com/terms/sla` lists "Agent Search API on Gemini Enterprise Agent Platform" alongside a legacy "Vertex AI Platform" entry | Both names are live in one legal index |
| `cloud.google.com/vertex-ai/generative-ai/docs/migrate/migrate-from-google-ai` returns a Google Cloud 404 | Many old deep paths are dead |
| `cloud.google.com/terms/data-residency` lists "Gemini Enterprise Agent Platform (formerly Vertex AI Platform)" and "Generative AI on Gemini Enterprise Agent Platform (formerly Generative AI on Vertex AI)" | Google itself states the "formerly" relationship in a legal terms document |

⚠ **Flagged:** I could not find a single canonical, dated Google announcement page describing the rename; a search pass for one returned empty results. The rename is established from live product URLs, page titles, a console path change, and a legal-terms page saying "formerly Vertex AI Platform" — not from a press release. That is strong evidence of the *state*, and no evidence of the *date*. See section 13.

Practical rule for reading old material: **where you see "Vertex AI" in any document written before this guide, read "Gemini Enterprise Agent Platform." Where you see "Vertex AI Studio," read "Agent Studio."** Model names did not change this way — a Gemini model is still a Gemini model — which is precisely why people conflate the two.

### 1.3 The naming history — decode older material with this table

| Older name you will find | Current name (verified September 2026) | Notes |
| --- | --- | --- |
| MakerSuite | **Google AI Studio** | The oldest developer playground name; abandoned years ago but still in blog posts and old tutorials |
| Google AI Studio | **Google AI Studio** (unchanged) | `aistudio.google.com` — direct-path playground and storefront |
| Gemini API | **Gemini Developer API** | The pricing page H1 is literally "Gemini Developer API pricing"; the docs nav still says "Gemini API" in places |
| Gemini API Additional Terms of Service | Same title, still in use | Effective March 23, 2026; page last updated 2026-04-28 |
| Vertex AI | **Gemini Enterprise Agent Platform** | The platform rename |
| Generative AI on Vertex AI | **Generative AI on Gemini Enterprise Agent Platform** | Stated as "formerly" on the data-residency terms page |
| Vertex AI Studio | **Agent Studio** | `console.cloud.google.com/agent-platform/studio` |
| Generative AI Studio / Gen AI Studio | **Agent Studio** (the platform's design canvas) | The platform's model-playground role folded into the current console experience |
| Google AI Premium (consumer plan) | **Google AI Plus** | Google One FAQ: "The Google AI Premium plan has a new name: Google AI Plus" |
| Gemini Enterprise | **Gemini Enterprise app** | The enterprise assistant/agent product — NOT the same as Gemini Enterprise Agent Platform |

The **Gemini Enterprise** versus **Gemini Enterprise Agent Platform** collision deserves emphasis. They are distinct products from the same family, and they appear as separate entries in Google's own legal index: the SLA index lists "Gemini Enterprise" (`cloud.google.com/terms/gemini-enterprise/sla`) separately from "Gemini on Vertex" (`cloud.google.com/vertex-ai/generative-ai/sla`), and the data-residency page lists them as two separate services. **Gemini Enterprise app** is the seat-based business assistant and agent product, sold in Business, Standard, Plus, Pay-as-you-go and Frontline editions; **Gemini Enterprise Agent Platform** is the developer platform that used to be called Vertex AI. If your procurement conversation contains both names, you have two products in the room.

### 1.4 "Which one do I actually want?" — one sentence each

| You are... | You want... | Because |
| --- | --- | --- |
| An individual developer or small team prototyping | **The Gemini Developer API** with Google AI Studio | An API key, minutes to first token, no Cloud project, a free tier |
| A team shipping a small production workload with no regulatory constraints | **The Gemini Developer API**, Paid tier | Same models, same list prices, batch and context caching, no Cloud overhead |
| A regulated institution putting a workload in front of customers or over sensitive data | **The platform path — Gemini Enterprise Agent Platform** | Data terms, residency, audit logging, VPC-SC, CMEK and IAM exist only there |
| A team needing Claude, Grok, Llama, Qwen, DeepSeek or gpt-oss alongside Gemini | **The platform path** | The developer path serves Google's models only — there is no equivalent |
| A team needing managed tuning, evaluation, vector search, RAG and agent deployment | **The platform path** | Those live on the platform |
| Someone whose colleague said "we already have Gemini" | Probably **a consumer plan or a Gemini Enterprise app subscription** | Establish which of the four things they actually bought before scoping anything |
| Someone who wants the shortest wire latency and least config | **The Gemini Developer API** | One global surface, one key, no project/location routing |


## 2. The Model Family

**Date stamp: the models page at `ai.google.dev/gemini-api/docs/models` states "Last updated 2026-09-15 UTC."** ⚠ **This section moves faster than any other part of this guide.** The lineup has churned roughly every six to ten weeks through 2025–2026. Treat it as a snapshot with a date stamp, not a stable catalogue, and re-verify at the source before hard-coding a model ID.

### 2.1 The tier vocabulary in use today

The tiers are **Pro**, **Flash**, **Flash-Lite**, plus the variants **Live**, **TTS**, **Transcribe**, **Omni**, and the image models marketed as **Nano Banana**. **There is no "Ultra" tier in the current model documentation.** If your 2025-era strategy document refers to a Gemini Ultra tier, that reference is stale. "Google AI Ultra" is a *consumer subscription plan*, not a model tier. Same word, different thing — exactly the collision this guide exists to resolve.

### 2.2 The stable lineup (as of 2026-09-15)

| Model | Endpoint | The docs' own description |
| --- | --- | --- |
| Gemini 3.8 Flash | `gemini-3.8-flash` | "Our most intelligent Flash model, engineered for long-horizon software engineering, autonomous agents, and complex enterprise workflows" |
| Gemini 3.8 Live | `gemini-3.8-live` | Default Live API model for low-latency voice agents without reasoning delays |
| Gemini 3.7 Flash | `gemini-3.7-flash` | Previous-generation Flash for complex coding and multi-step execution |
| Gemini 3.6 Flash | `gemini-3.6-flash` | Previous-generation Flash, speed/multimodal balance |
| Gemini 3.5 Flash | `gemini-3.5-flash` | "Legacy Flash model," baseline speed for routine high-throughput workloads |
| Gemini 3.5 Flash-Lite | `gemini-3.5-flash-lite` | Fastest, most cost-effective 3.5 model for high-throughput execution |
| Gemini 3.1 Flash-Lite | `gemini-3.1-flash-lite` | Frontier-class performance at a fraction of the cost |
| Nano Banana 2 | `gemini-3.1-flash-image` | High-efficiency image generation and editing |
| Nano Banana Pro | `gemini-3-pro-image` | State-of-the-art image generation and editing |
| Gemini 3.5 Transcribe | `gemini-3.5-transcribe` | Speech-to-text with language detection, diarization, word timestamps |

### 2.3 The preview lineup (as of 2026-09-15)

| Model | Endpoint | Note |
| --- | --- | --- |
| Gemini 3.1 Pro | `gemini-3.1-pro-preview` | The flagship Pro-tier model — **and it is a Preview model** |
| Gemini 3 Flash | `gemini-3-flash-preview` | Frontier-class performance at Flash cost |
| Gemini 3.5 Live Translate | `gemini-3.5-live-translate-preview` | Real-time speech-to-speech, 70+ languages |
| Gemini Omni Flash | `gemini-omni-1.1-flash` | Fast video generation, editing, keyframe interpolation |

### 2.4 The Pro/Flash divergence — the detail most people miss

Version numbers in the Gemini family have **diverged between tiers**. The stable flagship is a Flash model (Gemini 3.8 Flash). The flagship Pro position — Gemini 3.1 Pro — is still a **Preview** model. The Pro line sits at 3.1 while the Flash line sits at 3.8. This is not an oversight; the two lines iterate on their own cadence. But it has a concrete consequence: **if the capability you need only exists in the Pro tier, you are building on a Preview model**, with the shorter deprecation notice, the more restrictive rate limits, and the SLA consequence described in section 4.3. The stable, enterprise-friendly recommendation today points at a Flash model, not a Pro model — the opposite of the 2025 default assumption.

**Third-party trackers disagreed with each other about this lineup** during research — some described it as "3.1 Pro, 3 Flash, 3.1 Flash-Lite," others as "Flash at 3.8, Pro stalled at 3.1." Neither third-party summary was complete. The primary source (models page, dated 2026-09-15) is what this guide asserts. That the trackers disagreed at all is the evidence for how fast this decays.

### 2.5 The 2.5 family is still live — but not indefinitely

Gemini 2.5 Pro, 2.5 Flash, 2.5 Flash-Lite, 2.5 Flash Image (Nano Banana), 2.5 Flash/Pro TTS and 2.5 Flash Live all remain available on the developer path as of 2026-09-15, not deprecated. **On the platform side they have retirement dates:** the model-versions page gives `gemini-2.5-pro`, `gemini-2.5-flash` and `gemini-2.5-flash-lite` a retirement date of **October 20, 2026**, with `gemini-2.5-flash-image` retiring **October 2, 2026**. If you are on 2.5 today you have a migration project with a deadline, and it is sooner than the tier chart implies.

Gemini 2.0 Flash and 2.0 Flash-Lite, Gemini 3.1 Flash-Lite Preview and Gemini 3 Pro Preview are shut down on the developer path; the platform's retired list is longer, including `gemini-1.5-*` and `gemini-1.0-*` and the PaLM-era `text-bison` / `chat-bison` / `code-gecko` generation (retired April 21, 2025). The two lifecycle lists are not the same list, and the retirement dates are not the same dates — compare both explicitly if model lifecycle is part of your path decision.

### 2.6 Model-ID and version conventions (verbatim from the models page)

Gemini models are available in **stable**, **preview**, **latest** or **experimental** versions.

| Channel | What the docs say | Example |
| --- | --- | --- |
| Stable | "Points to a specific stable model. Stable models usually don't change. Most production apps should use a specific stable model." | `gemini-3.6-flash` |
| Preview | "Points to a preview model which may be used for production… will typically have billing enabled, might come with more restrictive rate limits and will be deprecated with at least 2 weeks notice." | `gemini-2.5-flash-preview-09-2025` |
| Latest | "Points to the latest release for a specific model variation… This alias will get hot-swapped with every new release… a **2-week notice** will be provided through email before the version behind latest is changed." | `gemini-flash-latest` |
| Experimental | "…typically be not be suitable for production use and come with more restrictive rate limits… Experimental models are not stable and availability of model endpoints is subject to change." | — |

Two things to take from that table. First, preview IDs carry a `YYYY-MM` date suffix (`-09-2025`), so a preview ID tells you roughly when the build was cut. Second, the `latest` alias is explicitly hot-swapped and the notice channel is **email**. An email is not a change-management control. In a regulated environment, pin a specific stable model ID and treat `latest` as a prototyping convenience.

Deprecations are published at `ai.google.dev/gemini-api/docs/deprecations` and, on the platform side, on the model-versions and lifecycle page.

### 2.7 Context windows and modalities — what I verified, and what I did not

⚠ **Flagged / partially captured.** I did not capture the per-model context-window and modality table in full. The models page groups models into Gemini 3, Gemini 2.5 Flash, Gemini 2.5 Flash-Lite, Gemini 2.5 Pro, Audio models, Generative media models, Music generation models, Tool and agent models, Specialized task models, and Previous models — but the per-model context-window figures were not in the captured text.

What *is* verified: the platform models page describes Gemini 2.5 Pro as having "a 1 million token context." A long-context pricing threshold at **200K input tokens** is visible in both pricing pages' tables (section 4.1). I will not convert either into a per-model context-window figure. **Do not quote a context window from this guide** — read it from the individual model page before sizing a design around it. The gap is recorded in section 13.

### 2.8 The platform serves far more than Gemini — the load-bearing proof

The cleanest single argument that "the platform" and "the model family" are different things: **the platform sells models Google does not own.** Verified from the platform documentation and the platform pricing page (both 2026-09-15):

| Category | Models offered on the platform path |
| --- | --- |
| Google first-party | Gemini, Gemma, Veo, Lyria, Imagen family |
| **Partner models (managed APIs)** | **Anthropic Claude, xAI Grok, Mistral AI** |
| **Open-weight models (managed APIs)** | **Llama (Meta), DeepSeek, Mistral, Qwen (Alibaba)**, plus MiniMax M2, Moonshot Kimi-K2-Thinking, Zhipu GLM (GLM-4.7 / GLM-5 / GLM-5.2), and OpenAI's open-weight **gpt-oss-120b / gpt-oss-20b** |

The two access modes are described as **Model as a Service (MaaS)** — "use managed APIs for partner and open models without managing infrastructure" — and **Model Garden** — "discover, test, customize, and deploy" first-party, partner and open-source models, at `console.cloud.google.com/agent-platform/model-garden`.

**Illustrative prices from the platform pricing page** (per 1M input/output tokens, USD), to make the roster concrete:

| Model | Input / Output | Vendor |
| --- | --- | --- |
| gpt-oss-120b | $0.09 / $0.36 | OpenAI (open weights) |
| Llama 4 Scout | $0.25 / $0.70 | Meta |
| Qwen3-Next-80B | $0.15 / $1.20 | Alibaba |
| MiniMax-M2 | $0.30 / $1.20 | MiniMax |
| GLM-4.7 | $0.60 / $2.20 | Zhipu |
| DeepSeek-V3.2 | $0.56 / $1.68 | DeepSeek |
| Grok 4.6 | $2.00 / $6.00 (cache hit $0.50) | xAI |
| Claude Sonnet 5 | $2.00 / $10.00 (batch input $1.00) | Anthropic |
| Claude Opus 4.8 | $5.00 / $25.00 | Anthropic |

**Nothing equivalent exists on the Gemini Developer API path.** The developer path serves Google's models only. That asymmetry — not latency, not price, not SDK ergonomics — is the strongest single reason a team with a multi-model strategy, or a multi-model *mandate*, ends up on the platform. A bank that must demonstrate model diversity for concentration-risk reasons cannot satisfy that requirement on the direct developer path at all, regardless of how much it likes the API key.


## 3. The Two Access Paths Explained

Two products, both from Google, both able to call a Gemini model. The vendor frames it this way on `ai.google.dev/gemini-api/docs/migrate-to-cloud` (last updated 2026-09-02): *"When developing generative AI solutions with Gemini, Google offers two API products: the Gemini Developer API and the Gemini Enterprise Agent Platform API."* And: *"The Gemini Developer API provides the fastest path to build, productionize, and scale Gemini powered applications. Most developers should use the Gemini Developer API unless there is a need for specific enterprise controls."*

Read the second sentence carefully: it is the vendor telling you, in plain language, that the differentiator is **enterprise controls** — not capability, not model access, not speed. That clause is the spine of sections 5, 8 and 10.

### 3.1 Side-by-side comparison

| Dimension | Direct developer path | Platform path |
| --- | --- | --- |
| Product name | **Gemini Developer API** | **Gemini Enterprise Agent Platform** (formerly Vertex AI) |
| Console / studio | **Google AI Studio** — `aistudio.google.com` | **Agent Studio** — `console.cloud.google.com/agent-platform/studio`; plus the Cloud console |
| Authentication | API key | **ADC (recommended)** or API key, with IAM roles |
| Identity model | A key, scoped to a project | Service accounts, IAM roles, resource hierarchy (org/folder/project) |
| Endpoint shape | A single global surface keyed by API key | Regional and multi-region endpoints plus a global endpoint |
| SDK | Google GenAI SDK | **The same** Google GenAI SDK, constructed differently |
| Billing | Free / Paid / Enterprise tiers on Cloud Billing | Cloud billing, per project, region-differentiated, with consumption modes |
| On-ramp designed for | Fast individual and team prototyping, then small production | Enterprise deployment inside an existing Google Cloud estate |
| Model catalogue | Google models only | Google + partner + open-weight models |
| Free tier | Yes — with data-use consequences (section 4.4) | No equivalent free tier |

### 3.2 Authentication

**Direct developer path: an API key.** Created at `aistudio.google.com/apikey`, attached to a Google Cloud project under the hood, used as a credential on the wire. The rate-limits page states that "rate limits are applied per project, not per API key" — so the key is a credential, not a quota boundary. Nothing about it expresses *who* is calling or *what* they may do beyond "this key can call these models."

**Platform path: ADC recommended, API keys accepted.** The get-started page at `cloud.google.com/gemini-enterprise-agent-platform/models/start` (last updated 2026-09-15) states: *"You can authenticate to Gemini Enterprise Agent Platform by using Application Default Credentials (ADC) or by using an API key. ADC is the recommended method."*

So **API keys now work on the platform path too** — a genuine change from the older Vertex AI model where a service account was effectively mandatory. The documented requirements for non-express-mode access: `roles/aiplatform.user` ("Agent Platform User") on the project; the Agent Platform API (`aiplatform.googleapis.com`) enabled; billing enabled; and optionally `gcloud auth application-default login` for local ADC.

⚠ **Partly investigated, with an open question.** The get-started page distinguishes a "standard API key" from an "express mode API key" and says express-mode users can skip the IAM role step. I could **not** establish from primary sources what express-mode API keys explicitly do *not* grant — whether they bypass IAM but also bypass audit logging, or are limited to a subset of models or features. **Do not assume an API key on the platform path gives the same governance posture as ADC.** For a regulated workload use ADC + service account + IAM and treat the API-key option as a quick-start convenience until its semantics are verified with Google. Recorded in section 13.

### 3.3 Endpoints and addressing

The locations page at `cloud.google.com/vertex-ai/generative-ai/docs/learn/locations` (last updated 2026-09-15) documents three addressing modes:

1. **Regional endpoints** — `https://LOCATION-aiplatform.googleapis.com`, e.g. `us-central1`.
2. **Multi-region endpoints** — `https://aiplatform.us.rep.googleapis.com` and `https://aiplatform.eu.rep.googleapis.com`. The docs state the purpose explicitly: "Multi-region endpoints allow you to ensure that machine learning processing of Customer Data by the service stays within a specific jurisdictional boundary, such as the United States or the European Union."
3. **The global endpoint** — with a warning: "Don't use the global endpoint if you have ML processing requirements, because you can't control or know which region your ML processing requests are sent to when a request is made."

That warning is the most operationally important sentence on the page for a European regulated buyer, and note the tension it creates with the SLA (section 4.3), which measures its latency objective **only** on the global endpoint. There is also a networking wrinkle: "Private Google Access isn't supported for multi-region endpoints… To establish private connectivity to multi-region endpoints, you must configure Private Service Connect endpoints for regional Google APIs."

The path shape is a resource address:

```text
POST https://LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/MODEL_ID:generateContent
Authorization: Bearer TOKEN
```

Note what the URL contains: a project, a location, a publisher, a model. Every one of those is an addressable, IAM-controllable resource in Google Cloud. **That is the platform path's value proposition expressed as a URL.** The direct developer path has no equivalent project/location addressing — one global surface, keyed — which is why it is fast to start with and why it cannot express a residency or per-project isolation requirement.

### 3.4 The SDK story — the shared library, constructed differently

This is the "what does *not* change" half of the migration story. The libraries page (`ai.google.dev/gemini-api/docs/libraries`, last updated 2026-06-22) states the recommended library is the **Google GenAI SDK**: "These are the official, production-ready libraries… They are in General Availability and used in all our official documentation and examples." Five languages, GA since May 2025:

| Language | Package | Repository |
| --- | --- | --- |
| Python | `google-genai` | `github.com/googleapis/python-genai` |
| JavaScript / TypeScript | `@google/genai` | `github.com/googleapis/js-genai` |
| Go | `google.golang.org/genai` | `github.com/googleapis/go-genai` |
| Java | `com.google.genai:google-genai` | `github.com/googleapis/java-genai` |
| C# | `Google.GenAI` | `github.io/dotnet-genai` |

The **platform side points to the same Google GenAI Libraries** — the platform get-started page links "Google GenAI libraries" as its own installation guide. The library is shared; **what differs is how the client is constructed.**

Developer path (Python) — credential resolved from the environment:

```python
from google import genai

client = genai.Client()

response = client.models.generate_content(
    model="gemini-3.8-flash", contents="Explain how AI works in a few words"
)
print(response.text)
```

Platform path (Python) — per the vendor's `migrate-to-cloud` page:

```python
from google import genai

client = genai.Client(
    vertexai=True, project="your-project-id", location="us-central1"
)

response = client.models.generate_content(
    model="gemini-3.8-flash", contents="Explain how AI works in a few words"
)
print(response.text)
```

⚠ **Flagged inconsistency.** The `migrate-to-cloud` page uses `vertexai=True`. The platform's own get-started and locations pages instead show the environment variable `GOOGLE_GENAI_USE_ENTERPRISE=True` and a client built as `genai.Client(enterprise=True, project=..., location=...)`. The Go example uses `Backend: genai.BackendVertexAI`; the Java example uses `.vertexAI(true)`. So **both a `vertexai`/`BackendVertexAI` switch and an `enterprise` switch are documented in live material**, and the naming of the platform-mode flag is in transition alongside the product rename. Check the SDK version you have pinned and use the flag that version recognises — do not assume a snippet copied from a 2025 blog post still compiles.

**Legacy libraries** — `google-generativeai` (Python), `@google/generativeai` (JS), `google.golang.org/generative-ai` (Go), `google_generative_ai` (Dart/Flutter), `generative-ai-swift`, `generative-ai-android` — are marked **"Not actively maintained,"** deprecated as of **November 30th, 2025**, and lack recent features such as the Live API and Veo. Replacements for Dart/Swift/Android are Genkit Dart and Firebase AI Logic. There was never a legacy Google Java SDK, so Java carries no migration burden. The vendor's legacy-to-new migration guide is `ai.google.dev/gemini-api/docs/migrate` (last updated 2026-09-02).

### 3.5 Console and studio experiences — three surfaces, easily conflated

| Surface | Where | What it is |
| --- | --- | --- |
| **Google AI Studio** | `aistudio.google.com` | Direct-path playground. Prompt experiments, model comparison, API-key management (`/apikey`), rate-limit dashboard (`/rate-limit`), projects (`/projects`), and a "Build" mode for vibe-coding apps |
| **Agent Studio** | `console.cloud.google.com/agent-platform/studio` | The *platform's* low-code canvas for designing multi-agent reasoning loops. Not the same product as Google AI Studio |
| **The Google Cloud console** | `console.cloud.google.com/agent-platform/...` | Model Garden, IAM, quotas, audit logs, billing, VPC-SC, CMEK — the enterprise control plane |

"AI Studio" means the developer playground; "Agent Studio" means the agent-design canvas inside the renamed platform. Different doors, different buildings.

## 4. The Differences That Decide It

### 4.1 Pricing — the list rates are the same; the structure around them is not

The headline numbers for shared models are **identical on both paths**. Gemini 3.8 Flash, for example, is **$0.75 input / $3.75 output per 1M tokens through December 31, 2026**, then **$1.50 / $7.50 starting January 1, 2027** — and that identical promotional-then-standard structure appears on both the developer-API pricing page and the platform pricing page.

| Price element (Gemini 3.8 Flash) | Through Dec 31, 2026 | From Jan 1, 2027 |
| --- | --- | --- |
| Input (standard) | $0.75 / 1M | $1.50 / 1M |
| Output (standard, incl. thinking tokens) | $3.75 / 1M | $7.50 / 1M |
| Batch / Flex input | $0.375 / 1M | $0.75 / 1M |
| Batch / Flex output | $1.875 / 1M | $3.75 / 1M |
| Context caching (cached input) | $0.075 / 1M | $0.15 / 1M |
| Context caching **storage** | $0.50 / 1M tokens per hour | $1.00 / 1M tokens per hour |
| Priority input (platform) | $1.35 / 1M | $2.70 / 1M |
| Non-global input (platform) | $0.825 / 1M | — |

**Who should care:** everyone doing a multi-year cost model. A go-live business case built on the promotional rate understates steady-state run cost by 100% from January 1, 2027. That date is contractual, not aspirational.

**Context caching has a storage charge people forget** — the per-hour storage line accrues whether or not you read the cache, and it doubles in 2027. A large cached context left in place for a month is a real line item.

**Grounding with Google Search:** 5,000 free search requests per month (shared across all Gemini 3.x models), then **$14 per 1,000 requests**; the developer-API page shows this as "Not available" on the Free tier. **Grounding with Google Maps:** 5,000 prompts per month free, then $14 per 1,000. A customer-submitted request may generate one *or more* search queries, and "you will be charged for each individual search query performed." **Who should care:** anyone assuming grounding is free at the margin — a grounded assistant handling 500,000 conversations a month at one query each is roughly $7,000/month in search alone, often more than the token cost.

**What differs structurally:**

| Structural difference | Direct developer path | Platform path |
| --- | --- | --- |
| Tier structure | Free / Paid / Enterprise | Pay-as-you-go with region and consumption-mode dimensions |
| Region pricing | Single global list | **Global vs Non-global columns**, non-global roughly 10% higher |
| Consumption modes | Standard, Batch (50% off), Priority | **Standard, Priority (~1.8x), Flex/Batch (half)** |
| Failed-request billing | — | "You're charged only for requests that return a 200 response code. Requests returning any other response codes, such as 4xx and 5xx codes, aren't charged for the input or output." |
| Long-context rule | Verify on the developer page before asserting | "If a query input context is longer than 200K tokens, all tokens are charged at long context rates" |

The **Global vs Non-global** split is the commercial shadow of the residency mechanism in section 4.5. Choosing a specific region or multi-region for data-residency reasons moves you off the cheapest column onto a rate roughly 10% higher — a real, budgetable cost of compliance worth naming in a business case rather than discovering in an invoice. The **200-response-code rule** means capacity estimates should model billed requests, not attempted requests, and that retry storms do not silently double a bill.

⚠ **Flagged:** I did not confirm whether the developer-API pricing page states the same "all tokens charged at long context rates" rule for contexts above 200K tokens. The platform table states it explicitly. Verify before relying on it for a developer-path cost model.

### 4.2 Rate limits and quotas — two different philosophies

The developer path's regime is documented at `ai.google.dev/gemini-api/docs/rate-limits` (last updated 2026-09-02). Its mechanics: limits are measured on **RPM** (requests/minute), **TPM** (input tokens/minute) and **RPD** (requests/day); image models also carry **IPM**; some models carry **TPD**. "Rate limits are applied per project, not per API key," and RPD quotas "reset at midnight Pacific time." "Your usage is evaluated against each limit, and exceeding any of them will trigger a rate limit error." "Rate limits are more restricted for experimental and preview models."

| Tier | Qualification | Billing tier cap | Spend rate limit / 10 min |
| --- | --- | --- | --- |
| Free | Active project or free trial | N/A | N/A |
| Tier 1 | Set up and link an active billing account | $250 | $10 |
| Tier 2 | Paid $100 + 3 days from first successful payment | $2,000 | $50 |
| Tier 3 | Paid $1,000 + 30 days from first successful payment | $20,000 – $100,000+ | $200 |

Qualification for Tiers 2 and 3 is based on **cumulative spending on Google Cloud services** on the linked billing account — "including, but not limited to, the Gemini API" — not on Gemini API spend alone. Hitting a spend-based limit returns `429 RESOURCE_EXHAUSTED`. Upgrades from Free to Tier 1 "typically take effect instantly," later upgrades "within 10 minutes," and "in rare cases an upgrade request may be denied based on other factors identified during the review process."

**Priority inference** "holds its own rate limits even though consumption is counted towards overall interactive traffic rate limits. **Default rate limits are: 0.3x the standard rate limit for each model and tier.**" Priority buys a much smaller dedicated lane at roughly 1.8x the price — a poor trade unless you specifically need isolation from best-effort traffic.

**Batch API limits:** 100 concurrent batch requests, 2GB input file size, 20GB file storage, plus per-model "batch enqueued tokens" caps that vary by tier. Tier 1 examples: Gemini 3.8 / 3.7 / 3.6 / 3.5 Flash at 3,000,000 enqueued tokens; Gemini 3.1 Pro Preview at 5,000,000; Flash-Lite tiers at 10,000,000; Gemini Embedding at 500,000. At higher tiers these rise to 400,000,000–1,000,000,000 for text-out models.

**The single sharpest line on the page for a regulated reader:** *"Specified rate limits are not guaranteed and actual capacity may vary."* And on increase requests: "We offer no guarantees about increasing your rate limit, but we'll do our best to review your request." **Who should care:** anyone whose SLA, RTO or capacity plan depends on a throughput number. A guarantee-shaped word ("tier," "limit") is attached to a non-guarantee-shaped sentence.

**The platform path's philosophy differs in kind, not just in number.** There, quota is a **Google Cloud resource**: per-project, per-region, visible and adjustable in the Cloud console, raisable through Cloud support, with provisioned throughput available on the committed/Enterprise tier and an SLA attached to that consumption model (section 4.3). The developer path's quota is a tier ladder you climb by spending. **Even where the RPM numbers look alike, the two are not the same instrument** — one is a platform resource with a support path behind it; the other is a rolling tier assignment with an explicitly disclaimed guarantee.

### 4.3 The SLA — the asymmetry is stark, and it lands where it matters

**Platform path — verified.** There IS a published SLA for Gemini inference on the platform. The document at `cloud.google.com/vertex-ai/generative-ai/sla` is titled **"Gemini Online Inference API on Gemini Enterprise Agent Platform Service Level Agreement (SLA)"** (last modified June 30, 2026):

The covered service is the `generateContent` and `streamGenerateContent` methods of the Gemini Online Inference API on Gemini Enterprise Agent Platform, committed at **99.5% Monthly Uptime Percentage** — or **95%**: *"When used with the models designated for shorter availability listed in the table described in the applicable Documentation, the SLO's Uptime Percentage will be 95%."*

Separately, and only under the **Provisioned Throughput** consumption model, there is a latency objective: a **99% Monthly Latency Target Attainment Percentage** for `streamGenerateContent` for a Covered Model from a Covered Endpoint, where Covered Models = Gemini 2.5 Pro, 2.5 Flash and 2.5 Flash-Lite, and Covered Endpoint = the **Global Endpoint**. Latency Targets are 60 TPS (2.5 Pro), 80 TPS (2.5 Flash) and 110 TPS (2.5 Flash-Lite), "excluding Long Context," where **Long Context means any context above 200K**.

**Mechanism and remedy:** **Downtime** means more than a 5% server-side Error Rate; a **Downtime Period** is five or more consecutive minutes of Downtime; Valid Requests are those conforming to the documentation that would normally return a non-error response. Financial Credits scale with the miss: 99.0–<99.5% → **10%** of the monthly bill for the covered service; 95.0–<99.0% → **25%**; <95.0% → **50%**. **Maximum credit is 50%** of the amount due for the covered service in the month. "Customer must **notify Google technical support within 30 days**" and "must also provide Google with log files showing Downtime Periods… If Customer does not comply with these requirements, Customer will forfeit its right to receive a Financial Credit." And: "This SLA states Customer's **sole and exclusive remedy** for any failure by Google to meet the SLO."

**What is explicitly excluded** — verbatim: *"The SLA does not apply to any (a) features or services designated pre-general availability (unless otherwise set forth in the associated Documentation); (b) features or services excluded from the SLA (in the associated Documentation), **including requests made using Grounding with Google Search**; or (c) errors (i) caused by factors outside of Google's reasonable control; (ii) that resulted from Customer's software or hardware or third party software or hardware; (iii) that resulted from Customer's setting a deadline shorter than the current server default (i.e., 'deadline_exceeded' errors); (iv) that resulted from abuses or other behaviors that violate the Agreement; or (v) that resulted from **quotas applied by the system** or listed in the Documentation or Admin Console."*

Read those exclusions with four other verified facts and the picture sharpens:

1. **Preview models are pre-general-availability and therefore outside the SLA.** The platform listing shows Gemini 3 Flash, Gemini 3 Pro Image and the Omni models as Preview.
2. **Gemini 3.1 Pro — the flagship Pro model — is a Preview model** (section 2.4).
3. **The newest Flash models are on "shorter availability."** The platform model-versions page (2026-09-15) places `gemini-3.8-flash`, `gemini-3.7-flash` and `gemini-3.6-flash` on the **"Models available for shorter availability periods"** list, retiring 45 days after a replacement ships. The SLA footnote reduces the commitment to 95% for exactly those models.
4. **Quota-driven failures are excluded**, so a 429 caused by your own rate limit is not an SLA event on either path. Also note that the Service Specific Terms state that **Pre-GA Offerings "are not covered by any SLA or Google indemnity,"** that the Data Location section does not apply to them, and that liability for Pre-GA Offerings is capped at the lesser of the agreement's limit or **$25,000**.

The layering to internalise: **the SLA is 99.5% for models on the long-availability list, 95% for models on the short-availability list, and absent for preview models and for grounded requests.** The generative-AI SLA is also, on its own terms, about the `generateContent`/`streamGenerateContent` methods of the Gemini Online Inference API — not about Agent Engine, ADK, Memory Bank, RAG Engine or the other agent surfaces.

**There is also a separate, older Vertex AI Platform SLA** at `cloud.google.com/vertex-ai/sla` (last modified February 12, 2026) covering Training, Deployment and Batch Prediction (≥99.9%), AutoML online prediction (≥99.9%), Custom Model Online Prediction (≥99.5%), Vertex Pipelines (≥99.5%) and the training cluster control plane API (≥99%). **That SLA does not cover generative-AI inference.** If someone waves "the Vertex AI SLA" at a Gemini latency question, they are holding the wrong document.

**Direct developer path — no uptime commitment found.** I searched `ai.google.dev` for an SLA statement covering the Gemini Developer API. The **Gemini API Additional Terms of Service** (effective March 23, 2026; page last updated 2026-04-28) is a terms document — eligibility, use restrictions, data use, payment terms, grounding restrictions, disclaimers — and contains **no uptime commitment**. The closest thing to a service commitment on the developer path is that single sentence on the rate-limits page: *"Specified rate limits are not guaranteed and actual capacity may vary."*

❌ **Unverified as an absence, not as a fact.** I cannot prove a negative from a search pass. But there is no SLA in the developer-API terms, no SLA linked from the developer pricing page, and no developer-path Gemini API entry in the Google Cloud SLA index. **Working assumption: the direct developer path offers no uptime commitment and no service credits, and capacity is explicitly disclaimed.** Treat that as the position until Google publishes something to the contrary. Recorded in section 13.

**Who should care:** any workload with a contractual uptime obligation, an operational risk register, or a business owner who will ask "what do we get if it goes down?" The platform path gives a number (99.5%, or 95% for the newest models) with a capped credit remedy and a 30-day claim window. The developer path gives no number at all.

### 4.4 Data use — the free-tier question, answered precisely

This is the most consequential difference between the paths and the most often misunderstood. The governing document is the **Gemini API Additional Terms of Service** (`ai.google.dev/gemini-api/terms`), **effective March 23, 2026, page last updated 2026-04-28**. Everything quoted in this subsection is from that document.

**The definitional trap: "free" is not about paying this month.** Two definitions carry the whole regime:

> "Any Services that are offered free of charge like direct interactions with Google AI Studio or unpaid quota in Gemini API are unpaid Services (the **'Unpaid Services'**)."

> "Your access to Google AI Studio is a 'Paid Service' **even when it is offered free of charge**, as long as the account you are using to access Google AI Studio has access to a Cloud Project with an associated and active Cloud Billing account or is a Workspace enterprise account. Your access to Gemini API is a 'Paid Service' **only when accessing the API through a Cloud Project associated with an active billing account**."

**Read that twice.** Which data regime you are in depends on your **billing linkage** — whether the project behind the key, or the studio account, has an active Cloud Billing account attached — **not** on whether you happen to be paying anything this month. A project with billing linked but zero traffic is a Paid Service for data purposes. A Workspace enterprise account using AI Studio with no Cloud Billing project is *also* a Paid Service for data purposes. A developer's personal account with no billing linkage is Unpaid. This is exactly the nuance that gets lost when someone forwards a screenshot of the pricing page.

**What Google does with Unpaid Services data — verbatim:**

> "When you use Unpaid Services, including, for example, Google AI Studio and the unpaid quota on Gemini API, Google uses the content you submit to the Services and any generated responses to provide, improve, and develop Google products and services and machine learning technologies, including Google's enterprise features, products, and services, consistent with our Privacy Policy."

> "To help with quality and improve our products, human reviewers may read, annotate, and process your API input and output. Google takes steps to protect your privacy as part of this process. This includes disconnecting this data from your Google Account, API key, and Cloud project before reviewers see or annotate it. **Do not submit sensitive, confidential, or personal information to the Unpaid Services.**"

The licence grant is extended too: the "Submission of Content" licence in the API Terms "extends, to the extent required under applicable law for our use, to any content (e.g., prompts, including associated system instructions, cached content, and files such as images, videos, or documents) you submit to the Services and to any generated responses." **In plain terms: on the free tier, your prompts and outputs train Google's products, and human reviewers may read them.** The terms say it in bold, addressed directly to the developer.

**What Google does with Paid Services data — verbatim:**

> "When you use Paid Services, including, for example, the paid quota of the Gemini API, Google doesn't use your prompts (including associated system instructions, cached content, and files such as images, videos, or documents) or responses to improve our products, and will process your prompts and responses in accordance with the Data Processing Addendum for Products Where Google is a Data Processor. For Paid Services, Google logs prompts and responses for a limited period of time, solely for detecting and preventing violations of the Prohibited Use Policy to maintain the safety and security of the Services, and any required legal or regulatory disclosures. This data may be stored transiently or cached in any country in which Google or its agents maintain facilities."

Note the last clause — "in any country in which Google or its agents maintain facilities." Even on the Paid tier, under these developer terms, there is **no residency commitment** for safety-logging data. Other data (account information and settings, billing history, usage details, token counts, operational status, safety-filter triggers, error reports, authentication details, IP addresses) remains under the Google Controller-Controller Data Protection Terms and the Google Privacy Policy.

**The jurisdictional carve-out — it cuts the other way:**

> "If you're in the European Economic Area, Switzerland, or the United Kingdom, the terms under 'How Google uses Your Data' in 'Paid Services' apply to all Services, including Google AI Studio and unpaid quota in the Gemini API, even though they are offered free of charge."

And separately, in Use Restrictions:

> "**You may use only Paid Services when making API Clients available to users in the European Economic Area, Switzerland, or the United Kingdom.**"

So for an EEA/CH/UK institution: the free-tier *data-use* exposure is removed by the first clause, **but building an end-user-facing application on the free tier is prohibited outright** by the second. There is no free-tier loophole — it is closed from the other direction. A paid tier (billing-linked project) is not merely advisable for an EEA/CH/UK institution; it is a licence condition for any application your customers touch.

**Other verified terms worth one line each:**

- **Tuning:** "Google only uses content that you import or upload to our model tuning feature for that express purpose… When you delete a tuned model, the related tuning content is also deleted."
- **No competing models:** you "may not use the Services to develop models that compete with the Services," and may not "attempt to reverse engineer, extract or replicate any component of the Services, including the underlying data or models (e.g., parameter weights)."
- **Professional use only:** "Use of Google AI Studio and Gemini API is for developers building with Google AI models for professional or business purposes, not for consumer use."
- **Age and medical use:** 18+ to use the APIs, no API Clients "directed towards or is likely to be accessed by" under-18s, and no use "in clinical practice, to provide medical advice, or in any manner that is overseen by or requires clearance or approval from a medical device regulatory agency."

- **No clinical/medical use:** you "may not use the Services in clinical practice, to provide medical advice, or in any manner that is overseen by or requires clearance or approval from a medical device regulatory agency."
- **Agentic services:** "When using agentic services, including the Computer Use API, you are solely responsible for the actions and tasks performed by the service… You will not automatically bypass any requests for human confirmation."
- **Robotics:** Robotics Models "have not been tested with all makes and models of robotics hardware" and must not be used for safety-critical applications (healthcare, transportation, or "other areas where safety protocols are vital").
- **Grounding with Google Search** carries its own restrictions: 30-day storage of prompts and outputs for grounding and system debugging, no caching/syndication/training on Grounded Results, strict display rules, and a narrow 2-year retention allowance for Grounded Result text. Read that section before designing a caching layer for a grounded assistant.
- **Disclaimers:** "The Services include experimental technology and may sometimes provide inaccurate or offensive content that doesn't represent Google's views."

**The platform path's data position.** Verified from the **Abuse monitoring** page (`cloud.google.com/vertex-ai/generative-ai/docs/learn/abuse-monitoring`, last updated 2026-09-15):

- Automated safety classifiers detect potential abuse. If suspicious activity requires investigation, Google may log customer prompts **solely** to determine whether a violation occurred, and "**This data won't be used to train or fine-tune any AI/ML models.**"
- Prompt logs are "stored securely for up to 90 days in the same region or multi-region selected by the customer for their project and adheres to Google Cloud assurances, such as Data Residency, Access Transparency and VPC Service Controls."
- "**Prompt logs for the purposes of abuse monitoring are not encrypted by Customer-managed encryption keys (CMEK).**"
- Customers can request an **opt-out** from abuse logging.
- **"Customers whose use of Google Cloud is governed by the Google Cloud Platform Terms of Service… customers with a Google Cloud Master Agreement are exempt from prompt logging for this abuse monitoring by default."**
- Models designated **"Advanced AI"** fall under the **Advanced AI Safety Addendum**: all prompts *and responses* are logged and stored for up to 30 days for abuse monitoring, again not used to train models, again not CMEK-encrypted, and "It may not be possible to opt-out of prompt-response logging when using some Advanced AI features." The Advanced AI list currently includes Claude Mythos, Claude Fable, and Claude Opus ≥4.7 / Sonnet ≥5 under Anthropic's Cyber Verification Program.
- **Partner-model terms differ by vendor.** Anthropic-model use on the platform is governed by Anthropic's Commercial Terms; for Claude Fable 5 on Google Cloud and Mythos 5, "prompts and responses are retained for up to 30 days" and you "must enable sharing this data with Anthropic for abuse monitoring."

**The free-tier answer, stated for the record:** **Free is not free. On the developer path the free tier trades your data for the service — prompts and outputs are used to improve Google's products, and human reviewers may read them. On the platform path there is no equivalent free tier at all.**

**Who should care:** every institution with a data classification policy, and every developer who has ever pasted production data into a playground. The specific trap is a prototyping team on an unlinked personal or sandbox account whose *free* work is, by the terms, training data. If your data-handling policy says customer data may not leave the perimeter, a free-tier developer key is a policy breach waiting to be found in an audit — and it will be found, because the terms say in bold that reviewers may read it.

### 4.5 Regions and data residency

**The mechanism on the platform path.** Verified from the Google Cloud **Service Specific Terms** §1 (Data Location) at `cloud.google.com/terms/service-terms`: for any Service listed at `cloud.google.com/terms/data-residency`, "Customer may select a specific Region or Multi-Region as detailed in the Cloud Locations Page, and Google will store Customer Data for that Service at rest only within the selected Region or Multi-Region. Google may replicate that Customer Data within any other Region located within the country of the selected Region or within the country or countries of the selected Multi-Region (as applicable) for backup, reliability, debugging, support, maintenance, or security purposes."

Note exactly what that promises and what it does not. It is an **at-rest storage commitment** with a **replication allowance inside the country (or the countries of the multi-region)**. It is not a promise that no byte transits anywhere else.

Three separate lists matter, and the generative-AI entries appear on two of them:

- **Data-residency list (General):** includes "Generative AI on Gemini Enterprise Agent Platform (formerly Generative AI on Vertex AI)" — **"except for Grounding with Google Search, Grounding with Google Maps, and RAG Engine."**
- **AI/ML Data Location list:** includes "Gemini Enterprise Agent Platform (formerly Vertex AI Platform)" — **"except Agent Platform Feature Store, Agent Runtime, Memory Bank, Sessions, Sandboxes, RAG Engine, and Agent Evaluations"** — and "Generative AI on Gemini Enterprise Agent Platform (only for models listed in the 'ML processing' section of the documentation and excluding Grounding with Google Search, Web Grounding for Enterprise, Grounding with Google Maps, and RAG Engine)."
- **Additional services supporting data residency without location configuration** — i.e. services that "do not store Customer Data at-rest or process Customer Data in use," which include **Access Approval, Access Transparency, Organization Policy Service, IAM, Resource Manager, Virtual Private Cloud and VPC Service Controls.**

The page also carries the critical carve-out that **Pre-GA Offerings are not covered by the Data Location provisions** (Service Specific Terms §5). So: a preview model on the platform path is outside both the SLA *and* the data-residency commitment.

**The contrast with the developer path is total.** There is no residency selector on the direct developer path. The Paid Services data terms explicitly permit safety-log data to be "stored transiently or cached in any country in which Google or its agents maintain facilities." A residency requirement is therefore not a configuration choice on the developer path — it is a disqualifier, unless the workload's data classification permits global processing.

**The commercial shadow:** the platform pricing page's **Global vs Non-global** columns (section 4.1), with non-global roughly 10% higher. Residency has a price, and it is on the platform path only — which is another way of saying the developer path does not sell the thing at all. **Who should care:** every institution with a regulator, a data-protection officer, or a contract that names a jurisdiction. **The practical rule: if your data cannot be processed outside a named country or region, the developer path is not a candidate, and the platform path must be configured region-by-region — including checking that the specific model you want is available in the region you need, and remembering that grounded requests and RAG Engine are excluded from the residency commitment even there.**

## 5. The Enterprise Controls That Only Exist on the Platform Path

This is the section that settles the regulated-enterprise decision, and it is short because the answer is short: the direct developer path has no equivalent for any of the following. An API key against `ai.google.dev` does not give an institution a perimeter, a key it controls, an audit trail, an identity model, a private network path, or a governance hierarchy. The platform path does.

| Control | What it buys | Verified where | Equivalent on the direct developer path? |
| --- | --- | --- | --- |
| **VPC Service Controls (VPC-SC)** | A service perimeter around the API, so data cannot be exfiltrated to a project outside the perimeter. Listed as a supported control for Generative AI features | `cloud.google.com/vertex-ai/generative-ai/docs/security-controls` (2026-09-15); listed on the data-residency page as a service that stores no customer data at rest | **No** |
| **CMEK (customer-managed encryption keys)** | Customer-controlled encryption keys for at-rest data, so key revocation is a customer capability | Same security-controls table, feature by feature; `cloud.google.com/gemini-enterprise-agent-platform/machine-learning/general/cmek` | **No** |
| **Cloud Audit Logs** | An audit trail of who called what, in Cloud Logging, retainable and exportable — the artefact an internal audit function asks for | The platform's IAM and logging are the standard Google Cloud services; the security-controls page cross-references the Cloud control set | **No** — the developer path has no Admin Activity / Data Access log for model calls |
| **IAM and the resource hierarchy** | Least privilege, service accounts, org/folder/project separation, predefined and custom roles (e.g. `roles/aiplatform.user`, `roles/aiplatform.admin`), and the ability to *withdraw* access when someone leaves | Platform get-started page (2026-09-15); the Advanced AI consent governance example uses `aiplatform.consents.update` restricted by least-privilege IAM | **No** — a key is a bearer token with no identity |
| **Private networking / Private Service Connect** | Keeping model traffic off the public internet; PSC endpoints for regional Google APIs for multi-region endpoints | Locations page (2026-09-15) | **No** |
| **Quota management as a resource** | Per-project, per-region quotas visible and adjustable in the console, raisable via support, with provisioned throughput on the committed tier | Platform pricing and quota documentation | **No** — the developer path offers a tier ladder and a form with no guarantee |
| **Data residency selection** | Region or Multi-Region selection with an at-rest storage commitment | Service Specific Terms §1 and the data-residency page | **No** |
| **Abuse-logging opt-out** | The ability to request that Google store no prompts for abuse monitoring; default exemption for Master Agreement customers | Abuse monitoring page (2026-09-15) | **No** — on the developer path, free-tier prompts are used by design |

Two honest caveats to hold alongside that table.

First, **several of these controls do not apply uniformly to every model or feature.** The security-controls page is a matrix: the supported controls differ per model, per feature (online prediction, batch inference, tuning, context caching), and the page notes that prompt logging for Partner and Open Models (MaaS) is turned off by default and adheres to the Agent Platform Zero Data Retention policy. Do not read the table above as "everything is covered everywhere."

Second, and separately, **there are documented gaps even where a control exists.** Abuse-monitoring prompt logs are explicitly **not** encrypted with CMEK. Grounding with Google Search, Grounding with Google Maps and RAG Engine are excluded from the data-residency commitment. Agent Platform Feature Store, Agent Runtime, Memory Bank, Sessions, Sandboxes, RAG Engine and Agent Evaluations are excluded from the AI/ML Data Location list. Pre-GA (preview) offerings sit outside the SLA, the Data Location provisions, the warranty and the indemnity, with liability capped at $25,000. **A regulated institution that selects the platform path has to select it feature by feature, not path by path** — and that nuance is where most enterprise architecture reviews go wrong.

**Who should care:** the CISO, the internal audit function, the data protection officer, and whoever signs the outsourcing register entry. For these readers the platform path is not "the more expensive option" — it is the only option, because the controls they will be asked to evidence at an audit do not exist on the other path.


## 6. Beyond the Raw Models

The platform path sells more than model calls. This section stays bounded: it covers only what distinguishes the paths, and points at sibling guides for depth.

**Customisation.**

| Capability | What it is |
| --- | --- |
| **Gemini Supervised Tuning** | Fine-tuning on your own labelled examples |
| **Gemini Preference Tuning** | "Align model behavior using human feedback or preference data" |
| **Tune Embeddings** | Domain-adapted embedding models |

The pricing page also carries supervised fine-tuning rates for open models (e.g. Gemma 3 27B IT at $6.83 per 1M count; Llama 3.3 70B at $6.72; Qwen 3 32B at $6.57), and notes that for model inference starting from Gemini 3, a tuned model's endpoint prediction price is **1.5x the base model**. If tuning is on your roadmap, that 1.5x multiplier belongs in the cost model.

**Retrieval and grounding.** **RAG Engine** and **Vector Search** — the latter described as "an AI-native search engine that combines storage and retrieval into one scalable, auto-tuned platform." Remember from section 4.5 that both RAG Engine and the grounding features are **excluded** from the data-residency commitments.

**Evaluation.** The **Gen AI evaluation service**, including "adaptive rubrics," runnable from the console or the Python SDK.

**Batch and pipeline capability.** Batch inference with per-model enqueued-token caps (section 4.2), plus the wider Google Cloud data and pipeline estate — see `ml_platforms_comparison_guide.md` §6 for MLOps depth rather than rebuilding it here.

**Agent tooling, under its current names** (verified from the platform documentation home, last updated 2026-09-15):

| Stage | Current names |
| --- | --- |
| Build | **Agent Development Kit (ADK)** ("a modular, model-agnostic framework designed to make building and deploying complex AI agents feel just like standard software development"); **Agent Garden** ("Google's library of pre-built agents"); **Agent Studio** ("Low-code canvas for designing complex multi-agent reasoning loops"); **Skill Registry** ("a secure, private, and low-latency repository for managing and discovering agent skills"); **RAG Engine**; **Vector Search**; **Managed Agents API** ("A config-driven, REST-first API for building autonomous agents inside a fully managed sandbox") |
| Scale | **Agent Engine** (deploy agents from a Python SDK, source files, or container images); agent lifecycle management; **Agent Platform Sessions** (conversational state); **Agent Platform Memory Bank** (long-term memory) |
| Govern | **Agent Gateway** ("centralized gateway for routing, securing, and monitoring agent traffic"); semantic governance; content security; Identity & IAM; agent sharing across projects and orgs |
| Optimise | Agent evaluation; observability (health, latency, resource usage); traces; topology; offline, simulated and online evaluation; prompt optimisation |

Two points of genuine relevance to the path decision. First, **the agent surfaces carry their own residency and SLA exclusions** — Memory Bank, Sessions, Sandboxes and RAG Engine are named exclusions on the AI/ML Data Location list, so a stateful agent architecture needs its own residency conversation. Second, **agent architecture is a discipline in its own right and is owned elsewhere in this repository**: see `agentic_engineering_guide.md` (the discipline-level synthesis that cross-links the cluster), `agent_harness_engineering_guide.md`, `agent_scaffolding_guide.md`, `agentops_guide.md`, `autonomous_agents_guide.md`, `enterprise_agentic_platform_architecture_guide.md`, `ai_agent_platform_selection_guide.md` and `multi_agent_banking_guide.md`. Do not rebuild that here.


## 7. Getting From One to the Other

The good news is that the vendor *does* publish a migration guide, and it is more helpful than the old 404 suggests. The page at `ai.google.dev/gemini-api/docs/migrate-to-cloud` (last updated 2026-09-02) is titled **"Gemini Developer API vs. Gemini Enterprise Agent Platform"** and contains side-by-side code for Python, JavaScript/TypeScript and Go. (The old path `cloud.google.com/vertex-ai/generative-ai/docs/migrate/migrate-from-google-ai` now returns a 404 — do not cite it.)

### 7.1 What changes — developer path to platform path

| What changes | Detail |
| --- | --- |
| **Authentication** | "You'll need to use Google Cloud service accounts to authenticate." An API key may work, but ADC + service account + IAM is the recommended and auditable route |
| **Client construction** | Add the platform-mode switch and project/location: `genai.Client(vertexai=True, project=..., location=...)` per the migration page, or `genai.Client(enterprise=True, ...)` / `GOOGLE_GENAI_USE_ENTERPRISE=True` per the platform get-started page. Verify which flag your pinned SDK version uses |
| **Project and location** | The client now carries a project ID and a location (`us-central1`, `europe-west1`, `us`, `eu`, or `global`). "You can use your existing Google Cloud project (the same one you used to generate your API key) or you can create a new Google Cloud project" |
| **Model naming** | Model IDs are largely shared for shared models, but the platform has its own GA/preview designation per model and its own lifecycle dates (section 2). "Supported regions may differ between the Gemini Developer API and the Gemini Enterprise Agent Platform API" |
| **IAM and roles** | Grant `roles/aiplatform.user` (and whatever else the workload needs), enable `aiplatform.googleapis.com`, enable billing |
| **Tuned models** | "Any models you created in Google AI Studio need to be retrained in Gemini Enterprise Agent Platform." This is a hard false assumption to catch late — tuning does not port |
| **Compliance surface** | From nothing to VPC-SC, CMEK, audit logs, residency, Org Policy — which is the whole point, but it is configuration work, not a code change |
| **API-key hygiene** | "If you no longer need to use your Gemini API key for the Gemini Developer API, then follow security best practices and delete it" — via the Cloud console API Credentials page; deletion takes a few minutes to propagate, after which traffic using it is rejected |

### 7.2 What does NOT change

- **The client library.** The **Google GenAI SDK** is shared across both paths, and the platform's own documentation links the same "Google GenAI libraries" installation guide. Migration does not mean adopting a new SDK.
- **The request/response shape is close** — `client.models.generate_content(model=..., contents=...)` is recognisably the same call on both paths. That is what makes the migration look trivial, and it *is* trivial at the level of the first call.
- **The model IDs for shared models** are largely the same strings.
- **The pricing list rates for shared models** are the same numbers (before region and consumption-mode structure).

⚠ **Do not overstate the shared-SDK point.** The request/response shapes are close, not identical; the client construction differs; some features exist on one path only; and the platform adds request-level concepts (project, location, service-account identity, consumption mode) that the developer path has no notion of. **The migration is small in code and large in configuration and governance.** Teams that budget only for the code change routinely underestimate the landing-zone, IAM, networking, residency and approval work by an order of magnitude.

### 7.3 The common mistakes

1. **Assuming the tuning artefacts travel.** They do not — AI Studio-tuned models must be retrained on the platform.
2. **Copying a 2025 client snippet.** The platform-mode flag is in transition between `vertexai`/`BackendVertexAI` and `enterprise`. Check your pinned version.
3. **Migrating the model call and forgetting the identity.** Going from a key to IAM means someone now has to own role assignments, service-account key rotation and offboarding.
4. **Assuming `global` is fine.** It is cheaper, higher-availability, and unusable if you have ML processing residency requirements.
5. **Assuming residency covers everything on the platform.** Grounding, RAG Engine, Memory Bank, Sessions, Sandboxes and Agent Evaluations are named exclusions.
6. **Leaving the developer API key alive after migration.** Delete it; it is a standing credential with no identity attached.
7. **Building the business case on the promotional price.** The January 1, 2027 step-up is a 100% increase on input and output for the Flash tier.
8. **Treating the Enterprise tier on the developer pricing page as a third path.** It is described as "powered by Gemini Enterprise Agent Platform" — it is the platform path with a different front door.

### 7.4 The reverse direction — platform to developer path

Going the other way is legitimate and common: platform for production, developer path for prototyping and experimentation. It is a reasonable pattern, and it is what the tools are shaped for — Google AI Studio is a better place to try a prompt than the Cloud console.

**But the caveats do not follow you back.** Three specifically:

- **The data terms do not follow you.** Production data governed by the platform's abuse-monitoring regime (regionally stored, no training use, opt-out available, Master Agreement exemption) becomes, on an unlinked developer account, free-tier data used to improve Google's products. **Do not rehearse production prompts on the free tier.** Use synthetic or masked data, or link billing.
- **The controls do not follow you.** VPC-SC, CMEK, audit logs, residency and Org Policy stop at the path boundary. A prototyping environment exempt from those controls must be labelled and time-boxed as such in your risk register.
- **The SLA does not follow you.** The 99.5% commitment is on the platform path. Prototyping against the developer path is fine *because* it is prototyping; do not let the prototype drift into serving real traffic.
- **And the EEA/CH/UK rule bites hardest here:** if you are an EEA/CH/UK institution, making API Clients available to users requires Paid Services. A prototype with an unlinked free key that is reachable by users in those regions is outside the licence, not merely outside policy.


## 8. The Decision

**This section is the guide's own analysis, not vendor guidance.** It applies the verified facts from sections 1–7 to five reader situations. Where the vendor has an explicit position, it is labelled as such.

### 8.1 Prototyping and evaluation

**Recommended: the direct developer path, with a billing-linked project.** An API key, Google AI Studio, and three lines of Python. Rate-limit and quota questions are an inference-time discovery, not a procurement project. The vendor's own position agrees: "Most developers should use the Gemini Developer API unless there is a need for specific enterprise controls."

**Trade-off accepted:** no controls, no SLA, and — if the project is on an unlinked account — free-tier data terms. **Make the billing linkage explicit from day one** so that your data regime is the paid one, and never rehearse on real customer data regardless.

### 8.2 A small production workload

**Recommended: the direct developer path, Paid tier** — unless one of the disqualifiers below applies. Same models, same list prices, batch at 50% off, context caching, no Cloud estate required. The developer path's "Paid" tier is a genuinely capable production home for a bounded workload.

**Trade-off accepted:** no uptime commitment ("Specified rate limits are not guaranteed and actual capacity may vary"), no audit trail for model calls, no residency, and an incident escalation path that is a community forum unless you buy Enterprise — in which case you should seriously consider just taking the platform path.

**The disqualifiers that flip the recommendation even at small scale:** any resident personal data; any workload a regulator or auditor will ask about; any requirement that a named jurisdiction process the data; any requirement that a specific human be answerable for access; or any workload that will be part of a customer-facing service in the EEA, Switzerland or the UK where the paid-tiers-only rule applies.

### 8.3 A regulated or enterprise production workload

**Recommended: the platform path — Gemini Enterprise Agent Platform.** Not a close call. Data terms, residency, audit logging, VPC-SC, CMEK, IAM, private networking, quota as a resource, Org Policy, Access Transparency and Access Approval all exist there and nowhere else. The SLA gives 99.5% (or 95% for the newest Flash models) with a credit remedy; the developer path gives no number.

**Trade-off accepted:** materially more configuration and governance work (landing zone, IAM design, networking, residency selection, approval cycles); a region premium of roughly 10% on non-global endpoints; and the requirement to verify coverage **feature by feature**, because preview models, grounded requests, RAG Engine and the agent stateful services each sit outside some commitments.

**And a second recommendation specific to this reader:** pin stable models. The stable flagship today is a Flash model (Gemini 3.8 Flash) — but it is also on the platform's "shorter availability" list, which is the one place where the stable/GA designation and the longest SLA do not coincide. Read the lifecycle page before you commit an SLA-backed workload to a model.

### 8.4 A workload needing non-Gemini models or full MLOps

**Recommended: the platform path, unambiguously.** Claude, Grok, Mistral as managed partner APIs; Llama, DeepSeek, Qwen, GLM, MiniMax, Kimi and gpt-oss as managed open-weight models; one governance perimeter, one billing relationship and one audit surface across all of them. The developer path serves Google's models only, so a genuine multi-model strategy cannot be expressed there at all.

**Trade-off accepted:** vendor concentration on Google Cloud even where the *models* are diverse (see section 10 on concentration risk — multi-model is not the same as multi-vendor), a heavier operational surface, and per-vendor terms that differ (Anthropic's cyber-verification programme and 30-day retention for some models is the clearest example).

### 8.5 A workload needing the lowest latency and least friction

**Recommended: the direct developer path**, or the platform path's **global endpoint** with its availability and 429-reduction benefit. The developer path removes a project/location hop and a whole configuration surface. If your constraint is time-to-first-token and your compliance constraints are light, this is the right answer, and choosing it deliberately is not a governance failure.

**Trade-off accepted:** no residency guarantee — and note the direct tension: the global endpoint's availability benefit and the global endpoint's residency impossibility are the *same property* viewed from two directions. The platform's SLA latency objective (99% latency target attainment, 60–110 TPS depending on model) is available **only** under Provisioned Throughput on the global endpoint. So the fastest, most-committed option is also the one that cannot say where your ML processing happens. Every low-latency conversation is really a conversation about that trade.

A note on the pattern behind all five: the recommendation flips on governance, not on capability, and it flips at the point where the workload stops being *yours alone* — production, customer-facing, personal data, or audit-relevant. Everything above that line belongs on the platform path; everything below it is a legitimate developer-path workload.

## 9. The Alternatives Frame

Brief, and cross-referenced rather than rebuilt. The point of this table is to place Google's two paths against the other things a buyer considers — not to score them.

| Option | Type | Serves non-first-party models? | Enterprise controls of the VPC-SC/CMEK/audit class? | Uptime commitment | Notes |
| --- | --- | --- | --- | --- | --- |
| **Gemini Developer API** | Vendor direct API | No — Google only | No | None found | The frictionless door |
| **Gemini Enterprise Agent Platform** (formerly Vertex AI) | Hyperscaler platform | Yes — Claude, Grok, Mistral, Llama, DeepSeek, Qwen, GLM, MiniMax, Kimi, gpt-oss | Yes | 99.5% / 95% on the Gemini inference SLA | The governed door |
| **Gemini Enterprise app** | Business SaaS assistant | — | Yes (Standard/Plus editions: VPC-SC, CMEK, Access Transparency, residency) | Separate SLA from the platform | A different product with a similar name |
| **Microsoft Azure / Azure AI Foundry** | Hyperscaler platform | Yes — multi-model marketplace | Yes | Vendor-published SLAs | See the sibling platform guide |
| **AWS / Amazon Bedrock** | Hyperscaler platform | Yes — multi-model, incl. Anthropic | Yes | Vendor-published SLAs | See the sibling platform guide |
| **Anthropic direct API** | Vendor direct API | Own models only | Limited | Vendor terms | The Claude path without a cloud platform |
| **Databricks** | Data/AI platform | Yes | Yes | Vendor-published SLAs | Strongest when the data estate is already there |
| **火山方舟 / ModelArk (ByteDance, via Volcengine / BytePlus)** | Hyperscaler-adjacent platform | Yes — many Chinese and open models | Region-dependent | Vendor terms | The non-Western counterpart; see `modelark_guide.md` |

For the full apparatus — how to compare platform SLAs, residency regimes, model catalogues, governance controls and exit costs across hyperscalers — use **`enterprise_ai_platforms_guide.md`** rather than rebuilding that machinery here. For the non-Western counterpart, its naming puzzle and what a regulated buyer can establish about it, use **`modelark_guide.md`**. Both are richer than anything this section could say in a paragraph, and neither should be duplicated.

Since the question "both doors lead to the same models, so what's the strategic difference?" recurs in every architecture review: it is the **catalogue plus the controls**. Google's *models* are available from several places. Google's *platform* provides a multi-model catalogue and a Cloud control plane that the direct API does not, and every competitor's equivalent platform makes the same trade in roughly the same shape.


## 10. The Regulated-Institution Angle

A bank, an insurer or a hospital does not choose between two API products. It chooses between two evidence sets. This section states the banking view plainly, then says when the direct path is still legitimate.

### 10.1 Why a regulated institution ends up on the platform path

| Driver | The specific fact that drives it |
| --- | --- |
| **Data use** | On the platform path, prompt logging for abuse monitoring is "not used to train or fine-tune any AI/ML models," is regionally stored, is opt-out-able, and Master Agreement customers are exempt by default. On the developer path's free tier, prompts are used to improve Google's products by design |
| **Residency** | The platform path sells a Region or Multi-Region selection with an at-rest storage commitment. The developer path sells nothing equivalent and explicitly permits paid-tier safety-log data to be cached "in any country in which Google or its agents maintain facilities" |
| **Audit** | Cloud Audit Logs, Access Transparency and Access Approval are platform-path capabilities. Without an audit trail for model calls, an internal audit function cannot reconstruct who asked what, which is usually a hard requirement rather than a preference |
| **Access control** | IAM with service accounts, least privilege, role withdrawal and resource-hierarchy separation versus a bearer key with no identity. Staff-joiner-mover-leaver controls cannot be evidenced against a key |
| **Perimeter** | VPC-SC is what stops generated content being exfiltrated to an unapproved project, and CMEK is what makes key revocation a customer capability. Neither exists on the direct path |
| **Outsourcing / third-party risk** | A Cloud DPA, service terms, a published SLA with a credit remedy, named support and an indemnity position (with the Pre-GA carve-out) constitute an evidence pack. The developer-API terms plus a community forum do not |
| **Model-risk governance** | A published deprecation policy, versioned model IDs, pinned stable models and a defined lifecycle are what a model-risk framework needs to function. The `latest` alias — hot-swapped on two weeks' email notice — is the anti-pattern |
| **Concentration risk** | Note the honest counterpoint: multi-model does **not** mean multi-vendor if all the models are served through one Google Cloud platform. Model diversity and infrastructure diversity are different risk axes, and a platform-path decision should record which one it is addressing |

### 10.2 What the institution must document

**What it must document:** the data classification per workload and per environment; the residency selection and the models available in it; the feature-by-feature coverage map (which controls apply to which model and which feature); the SLA chosen and the models it actually covers; the abuse-logging position (opt-out requested or Master Agreement exemption claimed); the deprecation policy and the pinned model IDs; the exit plan and the portability of prompts, tuned models and grounded content; the sub-processor list; and the EEA/CH/UK paid-tiers-only rule as a control, not a footnote.

For the governance method behind those artefacts, use this repository's existing material rather than inventing a parallel framework: **`technology/ai_llm/ai_governance_framework_guide.md`** for the framework, **`technology/ai_llm/ai_governance_bias_redteaming_guide.md`** for the testing and assurance side, **`banking/ai_genai_banking_compliance_guide.md`** for the regulatory mapping, **`banking/enterprise_risk_management_guide.md`** for the risk register treatment, and **`banking/risk_management_models_guide.md`** for the model-risk lifecycle. Those five are condensed here to one line each deliberately; the depth lives there.

### 10.3 When the direct path is still legitimate

The platform path is not mandatory for everything, and treating it as mandatory wastes money and slows teams. Conditions under which a regulated institution can legitimately use the direct developer path:

1. **Non-production, non-sensitive data only** — synthetic, public, masked or already-published content, with the classification recorded.
2. **A billing-linked project** so the paid data terms apply, never an unlinked free-tier account.
3. **No user-facing API Client in the EEA, Switzerland or the UK** — the terms permit only Paid Services there, and a prototype reachable by users is an API Client.
4. **A defined expiry** — a time-boxed prototyping exception that closes or migrates, not an open-ended second estate.
5. **Documented as exempt from the controls the platform would have provided**, with the exemption named on the risk register rather than implied.
6. **No tuned artefacts that matter** — remembering that AI Studio-tuned models must be retrained on migration.
7. **No production data rehearsal** — ever, on any path or tier, at any price.

The failure mode to avoid is not choosing the developer path. It is choosing it implicitly, on a personal or sandbox account, and discovering three years later that a team's early experiments were training data under the terms.


## 11. The Cymbal Bank Worked Example

> **Every number in this section is illustrative and fictional.** Cymbal Bank is a fictional institution used throughout this repository as the worked-example persona. Prices are derived transparently from the public list rates verified in section 4 and reproduced as arithmetic, not as a quotation or a commitment. Nothing here is a forecast, a quote, or advice for any real institution.

### 11.1 The setup

Cymbal Bank is building an AI capability. Two teams are already moving, in opposite directions.

- **The pilots team** has a Gemini Developer API key, a Google AI Studio account, and a Python script using `google-genai`. It works. It took a morning. It is running on a sandbox project with a personal Google account and no Cloud Billing linkage. The team wants to promote it.
- **The platform team** argues for the platform path — Vertex AI, now Gemini Enterprise Agent Platform — on the grounds that the bank's other workloads already live in Google Cloud and that the AI workload must sit inside the same controls.

Both teams are right about something. The bank's job is to work out which thing each of them is right about.

### 11.2 The requirements, and what each path does with them

| Requirement | Detail | Direct developer path | Platform path |
| --- | --- | --- | --- |
| **Workloads** | (A) an internal analyst assistant over public and internal non-sensitive documents; (B) a customer-facing assistant touching account data | (A) plausible; (B) disqualified | Both supported |
| **Data classification** | (A) internal; (B) confidential + personal data | (A) acceptable once billing-linked; (B) fails — free-tier terms, and no residency | Both acceptable; abuse-logging position recorded |
| **Residency** | EEA processing for (B); the bank's policy names an EU region | Not available at any price | Region or EU multi-region selection; requires an EU region where the chosen model is available |
| **Audit** | Reconstruct who called which model, when, with what authorisation, for (B) | No audit trail for model calls | Cloud Audit Logs plus IAM identity per caller |
| **Access control** | Least privilege, service accounts, quarterly access review, joiner-mover-leaver evidence | A bearer key with no identity; access review not evidenced | IAM roles, withdrawable, reviewable |
| **Perimeter and encryption** | No exfiltration to unapproved projects; customer-controlled keys | Neither exists | VPC-SC and CMEK available (with the documented CMEK gap on abuse-monitoring logs) |
| **SLA the business needs** | 99.5% for (B)'s service hours; a documented remedy | No commitment found; capacity explicitly disclaimed | 99.5% for long-availability models; **95% for the newest Flash models**; credits capped at 50% with a 30-day claim window |
| **Cost at projected volume** | (B) 40M input + 8M output tokens/month; (A) 10M + 2M | Same list rates | Same list rates, plus ~10% non-global where residency is selected; Priority and FSPs available |
| **Team skills** | Two Python developers, one Cloud-certified architect, no IAM/networking specialist | Adequate | Needs a landing-zone and IAM design effort; the architect can lead it |

### 11.3 The cost arithmetic — illustrative, derived from section 4.1

Using the verified list rate for Gemini 3.8 Flash ($0.75 input / $3.75 output per 1M tokens through December 31, 2026; $1.50 / $7.50 from January 1, 2027):

| Scenario | Input cost | Output cost | Monthly total |
| --- | --- | --- | --- |
| Workload (B), promo rate, global endpoint | 40M × $0.75/1M = $30.00 | 8M × $3.75/1M = $30.00 | **$60.00** |
| Workload (B), promo rate, non-global (residency) at $0.825/$4.125 | 40M × $0.825/1M = $33.00 | 8M × $4.125/1M = $33.00 | **$66.00** |
| Workload (B), standard rate from Jan 1 2027, non-global (doubled) | $66.00 | $66.00 | **$132.00** |
| Workload (A) at promo, global | 10M × $0.75/1M = $7.50 | 2M × $3.75/1M = $3.75 | **$11.25** |
| Both, from Jan 1 2027, non-global | ~$148.50 | ~$74.25 | **≈$222.75** |

The token cost is trivial at that volume — which is the first honest finding: **the commercial decision is not made by the token line.** It is made by the residency premium, the ~10% non-global uplift, grounding charges if used ($14 per 1,000 search queries after 5,000 free per month per model family), context-caching storage ($0.50 per 1M tokens per hour through 2026, $1.00 thereafter, charged whether or not the cache is read), and — far larger than any of those — the **effort cost of the enterprise prerequisites**. In an illustrative Cymbal Bank estimate, the platform path's one-off engineering and governance effort (landing zone, IAM model, networking, residency design, control mapping, evidence pack) dominates several years of the token spend for a workload of this size. That is an argument for doing the work once, properly, at the organisation level, rather than per pilot.

### 11.4 The migration the bank actually has to do

1. **Auth and identity.** Replace the API key with a service account and ADC; grant `roles/aiplatform.user` (and least-privilege variants) on the workload projects; enable `aiplatform.googleapis.com`; enable billing.
2. **Client construction.** Add the platform mode to the `genai.Client(...)` construction — verifying whether the pinned SDK version takes `vertexai=True` or `enterprise=True` — and set project and location. The three-line script becomes a config-driven one.
3. **Endpoint and residency.** Choose the EU region or the `eu` multi-region hostname; confirm the chosen model is available there; accept that private connectivity to multi-region endpoints requires PSC endpoints for regional Google APIs.
4. **Model selection and pinning.** Pin a stable model ID rather than an alias; check the lifecycle page's retirement dates; note that the newest Flash models are on the short-availability list with the 95% SLA.
5. **Retrain tuned artefacts.** Any AI Studio-tuned model must be retrained on the platform.
6. **Controls.** Configure VPC-SC, CMEK (recording the abuse-log documentation gap), audit logging, Org Policy, and the abuse-logging opt-out request or Master Agreement exemption.
7. **Delete the developer key.** On the Cloud console API Credentials page; propagation takes a few minutes.
8. **Evidence.** Produce the control-coverage map, the data-flow diagram, the residency statement, the SLA selection and the deprecation-pinning record for internal audit and the regulator.

### 11.5 Governance approvals

Data protection sign-off on the residency and the abuse-logging position; information security on the perimeter, keys and audit trail; model risk on the pinned model IDs, lifecycle and validation evidence; procurement on the Cloud terms, DPA and sub-processors; and the outsourcing register entry. For the worked-example institution the binding constraint is usually **model risk and procurement, not engineering** — which is the reverse of the pilots team's intuition.

### 11.6 The recommendation, and what must be true for it to hold

**Recommendation for Cymbal Bank: put workload (B) — customer-facing, confidential, personal data, EEA residency — on the platform path. Allow workload (A) — internal, non-sensitive — to remain on the direct developer path only for the duration of its pilot, on a billing-linked project with synthetic or already-published content, with a stated expiry and a named exemption. Migrate (A) to the platform when either its data classification changes or its expiry arrives.**

This recommendation holds **only if** all of the following are true:

1. The measured demand and SLA obligations are as stated — if (B) turns out to need only best-effort availability, the calculus changes materially.
2. An EU region (or the `eu` multi-region) is available for the specific model (B) needs, and the bank's residency policy accepts at-rest storage with in-country replication.
3. The bank actually operates the controls it buys — a VPC-SC perimeter that exists on paper and is not enforced is worse than none, because it creates false assurance.
4. The bank accepts the ~10% non-global premium and has modelled the January 1, 2027 price step-up.
5. The bank records that the newest Flash models carry a 95% (not 99.5%) SLA and that preview models carry none, and either avoids them for (B) or accepts the lower commitment explicitly.
6. The bank treats multi-model availability as model diversity only, and does not book it as infrastructure-level concentration-risk mitigation.
7. The AI Studio-tuned models are either trivial or are retrained, and nobody has assumed they port.
8. The bank's EEA paid-tiers-only rule is implemented as a control — someone can demonstrate no free-tier API Client is exposed to users.

If any of the first three fails, the recommendation degrades: the workload moves to a different platform or a different provider, not to the direct developer path with a residency requirement pretended away.


## 12. The Claims Audit

Model names, prices and policies go stale fastest, so the dates in this table matter more than in most guides. ✅ verified at a primary source · ⚠ flagged (secondary, partial, or inconsistent) · ❌ unverified.

| # | Claim | Status | Source | Source quality | Date |
| --- | --- | --- | --- | --- | --- |
| 1 | Vertex AI is now branded Gemini Enterprise Agent Platform; both names are live | ⚠ | Live URLs, page titles, console path, and a terms page saying "formerly" | Multiple primary sources, non-contemporaneous; **no dated announcement found** | 2026-09-16 access |
| 2 | "Gemini Enterprise Agent Platform (formerly Vertex AI Platform)"; same for "Generative AI on…" | ✅ | `cloud.google.com/terms/data-residency` | Legal terms — strongest available form | Page last modified 2026-09-08 |
| 3 | Developer path is branded "Gemini Developer API" | ✅ | Pricing page H1 | Primary | 2026-09-15 |
| 4 | Model lineup (stable/preview/retired), endpoints, and the absence of an Ultra tier; Pro at 3.1 (preview), Flash at 3.8 (stable) | ✅ | `ai.google.dev/gemini-api/docs/models`; platform model page | Primary | 2026-09-15 |
| 5 | Third-party trackers disagreed on the lineup | ⚠ | Search results during research | Secondary; used only as evidence of churn | 2026-09-16 |
| 6 | Per-model context windows and the modality table | ⚠ | Not captured; only 2.5 Pro "1 million token context" and the 200K long-context threshold were seen | Primary but partial | 2026-09-15 |
| 7 | Gemini 3.8 Flash $0.75/$3.75 through 2026-12-31, $1.50/$7.50 from 2027-01-01, identical on both paths; caching, batch and grounding prices | ✅ | Both pricing pages | Primary | 2026-09-15 |
| 8 | Non-global region pricing ≈10% higher; Standard/Priority/Flex modes; FSPs; 200-response-code billing rule | ✅ | Platform pricing page ("Agent Platform Pricing") | Primary | 2026-09-15 |
| 9 | "All tokens charged at long context rates" above 200K | ⚠ | Platform pricing table only; developer page not confirmed | Primary, single path | 2026-09-15 |
| 10 | Rate-limit tiers, spend limits, 0.3x Priority rule, batch caps, "Specified rate limits are not guaranteed" | ✅ | `ai.google.dev/gemini-api/docs/rate-limits` | Primary | 2026-09-02 |
| 11 | Gemini inference SLA: 99.5%, or 95% for short-availability models; Provisioned Throughput latency SLO on the global endpoint; exclusions and 30-day claim window | ✅ | `cloud.google.com/vertex-ai/generative-ai/sla` | Legal SLA document | Last modified 2026-06-30 |
| 12 | There is a separate Vertex AI Platform SLA that does **not** cover generative-AI inference | ✅ | `cloud.google.com/vertex-ai/sla` | Legal SLA document | Last modified 2026-02-12 |
| 13 | Pre-GA offerings: no SLA, no indemnity, Data Location does not apply, $25,000 liability cap | ✅ | Service Specific Terms §5 | Legal terms | Accessed 2026-09-16 |
| 14 | The developer path offers no uptime commitment | ❌ | Absence of any SLA in the developer terms or the Cloud SLA index | Negative finding from a search pass | 2026-09-16 |
| 15 | Free-tier prompts/outputs are used to improve Google's products and human reviewers may read them; Paid/Unpaid is set by billing linkage; EEA/CH/UK carve-outs | ✅ | Gemini API Additional Terms, "Unpaid Services", "Paid Services", "Use Restrictions" | Legal terms, verbatim | Effective 2026-03-23; page updated 2026-04-28 |
| 16 | Platform abuse logging: ≤90 days, regionally stored, not used for training, not CMEK-encrypted, opt-out available, Master Agreement exemption by default | ✅ | Abuse monitoring page | Primary | 2026-09-15 |
| 17 | Advanced AI Safety Addendum: 30-day prompt **and** response logging; partner-specific retention for some Claude models | ✅ | Abuse monitoring page | Primary | 2026-09-15 |
| 18 | Data Location mechanism (Region/Multi-Region, at rest, in-country replication) and the residency lists with named exclusions (grounding, RAG Engine, Memory Bank, Sessions, Sandboxes, Agent Evaluations, Feature Store, Agent Runtime) | ✅ | Service Specific Terms §1; `cloud.google.com/terms/data-residency` | Legal terms | Page last modified 2026-09-08 |
| 19 | Security controls matrix (residency, CMEK, VPC-SC, Access Transparency) varies per model and feature | ✅ | Security controls page | Primary | 2026-09-15 |
| 20 | Platform serves partner (Claude, Grok, Mistral) and open-weight (Llama, DeepSeek, Qwen, GLM, MiniMax, Kimi, gpt-oss) models, with the illustrative price points quoted in section 2.8 | ✅ | Platform model pages, pricing page, locations page | Primary | 2026-09-15 |
| 21 | Google GenAI SDK is shared across both paths; five languages; GA since May 2025; legacy libraries deprecated 2025-11-30 and not actively maintained | ✅ | Libraries page; platform get-started page | Primary | 2026-06-22 / 2026-09-15 |
| 22 | Client construction differs: `vertexai=True` vs `enterprise=True` / `GOOGLE_GENAI_USE_ENTERPRISE` | ⚠ | `migrate-to-cloud` vs platform get-started and locations pages | Primary sources that disagree — flagged, not resolved | 2026-09-02 / 2026-09-15 |
| 23 | Platform accepts API keys as well as ADC; standard vs express-mode key distinction | ✅ (existence) / ❌ (semantics) | Platform get-started page | Primary for existence; semantics not established | 2026-09-15 |
| 24 | Migration guide exists ("Gemini Developer API vs. Gemini Enterprise Agent Platform"); AI Studio-tuned models must be retrained on the platform; the old migrate-from-google-ai URL returns 404 | ✅ | `ai.google.dev/gemini-api/docs/migrate-to-cloud`; direct fetch of the old URL | Primary | 2026-09-02 / 2026-09-16 |
| 25 | Platform lifecycle: 2.5 Pro/Flash/Flash-Lite retire 2026-10-20; 3.8/3.7/3.6 Flash sit on the short-availability list | ✅ | Platform model-versions page | Primary | 2026-09-15 |
| 26 | Consumer plans are Google AI Plus / Pro / Ultra; "AI Premium" renamed "AI Plus"; Gemini app at gemini.google.com; Gemini Enterprise app is distinct from Agent Platform | ✅ | `one.google.com/about/google-ai-plans/`; `cloud.google.com/gemini-enterprise`; SLA index; data-residency page | Primary, multiple | Accessed 2026-09-16 |
| 27 | Platform agent surface names (ADK, Agent Garden, Agent Studio, Agent Engine, Memory Bank, Agent Gateway, Skill Registry, Managed Agents API) | ✅ | Platform documentation home | Primary | 2026-09-15 |
| 28 | Third-party SLA commentary and consumer price points from aggregator sites | ❌ | Search results only | Excluded from this guide entirely — not used as a source | 2026-09-16 |


## 13. What Could Not Be Verified

1. **A canonical dated announcement of the Vertex AI → Gemini Enterprise Agent Platform rename.** Search passes returned no such page. The rename is established from live URLs, page titles, a console path and a legal "formerly Vertex AI Platform" clause — which evidences the state, not the date.
2. **Per-model context windows and the full modality table.** The models page text captured does not carry them. The only verified figures are Gemini 2.5 Pro's "1 million token context" and the 200K long-context pricing threshold. Do not infer a context window from this guide.
3. **Whether the developer-API pricing page applies the same "all tokens at long context rates above 200K" rule** as the platform pricing table states.
4. **The exact semantics of API keys on the platform path** — specifically what an "express mode" API key does and does not grant relative to ADC, whether it is subject to the same audit logging, and whether it is limited in model or feature scope.
5. **An SLA for the Gemini Developer API.** The absence is recorded as ❌ rather than asserted as a fact: no SLA exists in the developer terms, none is linked from the developer pricing page, and no such entry appears in the Cloud SLA index. A negative finding from a search pass is still a negative finding.
6. **The relationship between the consumer/business Gemini surfaces and the developer platform, stated authoritatively by Google.** "Gemini Enterprise app" and "Gemini Enterprise Agent Platform" appear as separate entries in the SLA index and the data-residency page, which is why this guide treats them as distinct — but I could not find a single page that lays out the whole family tree in one diagram.
7. **Which specific models are available in which specific regions** for the residency scenario. The locations page carries per-model, per-region tables, and the captured text was partial. Verify per model.
8. **The complete set of differences between the two paths' request/response shapes.** The vendor's migration page shows the shared call pattern; a systematic field-level diff was not performed. Related: whether any consumer Google AI plan affects data-use terms for direct API usage — the developer plans page says plan benefits apply to the AI Studio interface while direct API use "is billed and managed separately", but the interaction with the data regime was not fully traced.
9. **Any Cymbal Bank figure.** Everything in section 11 is illustrative and fictional by construction and is not a claim about any real price, volume or institution.

## 14. Glossary

| Term | Meaning |
| --- | --- |
| **Vertex AI** | The old name for the Google Cloud AI platform. Now branded **Gemini Enterprise Agent Platform**. Still appears in live URLs, SLA entries and most existing enterprise material |
| **Gemini Enterprise Agent Platform** | The current name for the platform formerly called Vertex AI. Serves Gemini plus Google's other models plus partner and open-weight models, with the Google Cloud control plane |
| **Gemini** | Context-dependent. Either the model family, or the consumer/business products, or a shorthand for whichever door the speaker uses. Establish which before answering |
| **Gemini Developer API** | The direct developer path to the Gemini models, at `ai.google.dev`. API-key auth, Free/Paid/Enterprise tiers, Google models only |
| **Google AI Studio** | The developer-path playground at `aistudio.google.com`. Not to be confused with Agent Studio |
| **Agent Studio** | The platform's low-code agent-design canvas at `console.cloud.google.com/agent-platform/studio`. Formerly the Vertex AI Studio role |
| **Gemini Enterprise app** | The seat-based enterprise assistant and agent product (Business / Standard / Plus / Pay-as-you-go / Frontline editions). Distinct from the Agent Platform |
| **MaaS (Model as a Service)** | Managed APIs for partner and open models on the platform, without managing infrastructure |
| **Model Garden** | The platform's catalogue for discovering, testing, customising and deploying first-party, partner and open-source models |
| **ADK (Agent Development Kit)** | The platform's modular, model-agnostic agent-building framework |
| **Provisioned Throughput** | The committed capacity consumption model on the platform; the only mode with a latency SLO attached |
| **Consumption modes** | Standard, Priority (~1.8x) and Flex/Batch (half) on the platform path; **FSP (Flexible Savings Plan)** is a spend-based committed-use discount across eligible Google Cloud products |
| **Grounding** | Calling Google Search or Google Maps to ground a response. Priced at $14/1,000 after 5,000 free per month; excluded from the platform SLA and from residency |
| **Unpaid / Paid Services** | The terms' classification of the developer services. Determined by billing linkage, not by whether a charge was incurred |
| **Pre-GA Offering** | A pre-general-availability feature or model — outside the SLA, the Data Location provisions, the warranty and the indemnity; $25,000 liability cap |
| **Stable / Preview / Latest / Experimental** | The four model-ID channels. `latest` is hot-swapped with two weeks' email notice |
| **VPC-SC / CMEK / ADC** | VPC Service Controls (a perimeter to prevent data exfiltration); customer-managed encryption keys (customer-controlled encryption at rest); Application Default Credentials (the recommended platform-path authentication method) |
| **RPM / TPM / RPD / IPM / TPD** | Requests per minute, input tokens per minute, requests per day, images per minute, tokens per day |
| **Naming decode** | MakerSuite → Google AI Studio; Gemini API → Gemini Developer API; Vertex AI → Gemini Enterprise Agent Platform; Vertex AI Studio → Agent Studio; Generative AI Studio → Agent Studio; Google AI Premium → Google AI Plus |


## 15. Cross-References and Further Reading

### 15.1 Sibling guides in this repository

- `technology/ai_llm/agentic_engineering_guide.md` — the discipline-level synthesis that cross-links the agent cluster
- `technology/ai_llm/agent_harness_engineering_guide.md`, `agent_scaffolding_guide.md`, `agentops_guide.md`, `autonomous_agents_guide.md` — agent engineering, harness and operations
- `technology/ai_llm/enterprise_agentic_platform_architecture_guide.md`, `ai_agent_platform_selection_guide.md`, `multi_agent_banking_guide.md` — platform architecture, selection and banking-specific multi-agent patterns
- `technology/ai_llm/agent_harness_engineering_guide.md`, `agent_scaffolding_guide.md`, `agentops_guide.md`, `autonomous_agents_guide.md` — agent engineering, harness and operations
- `technology/ai_llm/enterprise_ai_platforms_guide.md` — the platform-comparison machinery (use this for cross-hyperscaler SLA, residency and governance comparison)
- `technology/ai_llm/ml_platforms_comparison_guide.md` §6 — MLOps depth
- `technology/ai_llm/modelark_guide.md` — the non-Western counterpart (火山方舟 / ModelArk) and its own naming puzzle
- `technology/ai_llm/ai_governance_framework_guide.md`, `ai_governance_bias_redteaming_guide.md` — governance framework and assurance testing
- `banking/ai_genai_banking_compliance_guide.md`, `banking/enterprise_risk_management_guide.md`, `banking/risk_management_models_guide.md` — banking compliance, enterprise risk and model risk

### 15.2 Primary sources consulted, with access dates

| Source | URL | Page date | Accessed |
| --- | --- | --- | --- |
| Gemini models | `ai.google.dev/gemini-api/docs/models` | Last updated 2026-09-15 | 2026-09-16 |
| Gemini Developer API pricing | `ai.google.dev/gemini-api/docs/pricing` | Last updated 2026-09-15 | 2026-09-16 |
| Agent Platform Pricing (served at the old Vertex AI generative-AI pricing URL) | `cloud.google.com/vertex-ai/generative-ai/pricing` | — | 2026-09-16 |
| Gemini API Additional Terms of Service | `ai.google.dev/gemini-api/terms` | Effective 2026-03-23; updated 2026-04-28 | 2026-09-16 |
| Rate limits | `ai.google.dev/gemini-api/docs/rate-limits` | Last updated 2026-09-02 | 2026-09-16 |
| Google models on Gemini Enterprise Agent Platform | `cloud.google.com/gemini-enterprise-agent-platform/models/google-models` | Last updated 2026-09-15 | 2026-09-16 |
| Service Specific Terms | `cloud.google.com/terms/service-terms` | — | 2026-09-16 |
| Gemini Online Inference API SLA | `cloud.google.com/vertex-ai/generative-ai/sla` | Last modified 2026-06-30 | 2026-09-16 |
| Data residency services list | `cloud.google.com/terms/data-residency` | Last modified 2026-09-08 | 2026-09-16 |
| Abuse monitoring | `cloud.google.com/vertex-ai/generative-ai/docs/learn/abuse-monitoring` | Last updated 2026-09-15 | 2026-09-16 |
| Deployments and endpoints (locations) | `cloud.google.com/vertex-ai/generative-ai/docs/learn/locations` | Last updated 2026-09-15 | 2026-09-16 |
| Get started with Gemini Enterprise Agent Platform | `cloud.google.com/gemini-enterprise-agent-platform/models/start` | Last updated 2026-09-15 | 2026-09-16 |
| Gemini Developer API vs. Gemini Enterprise Agent Platform | `ai.google.dev/gemini-api/docs/migrate-to-cloud` | Last updated 2026-09-02 | 2026-09-16 |
| Google AI plans (consumer) | `one.google.com/about/google-ai-plans/` | — | 2026-09-16 |
| Gemini Enterprise app | `cloud.google.com/gemini-enterprise` | — | 2026-09-16 |

Where a URL above was seen only as a reference inside another page, it is not listed here — this table contains only pages actually fetched.


## 16. Closing Summary

"Vertex AI" and "Gemini" are two words for four different things: a model family, a set of consumer and business products, a Google Cloud platform that has been renamed **Gemini Enterprise Agent Platform**, and a direct developer API that is now branded the **Gemini Developer API**. The first of those is reachable by two doors. The consumer products are a different building entirely, and the similar names — Gemini Enterprise app versus Gemini Enterprise Agent Platform — are the trap this guide exists to disarm.

Between the two doors, the models and the list prices are largely the same, and the SDK is literally the same library. What is not the same is everything around them: the free tier's data terms, the residency regime, the audit trail, the identity model, the perimeter, the key custody, the quota instrument, the support path, and — most starkly — whether any uptime is committed at all. The vendor says it plainly: most developers should use the developer path **unless there is a need for specific enterprise controls**. For a regulated institution, that conditional is not a hedge; it is the whole decision.

The practical rule, if you keep one thing from this guide: **choose the model family first, choose the door second, and choose the door on governance rather than on capability** — because on capability the two doors look almost identical, and on governance they have almost nothing in common. Both doors lead to the same models, but only one of them can be walked through in front of an auditor. Choose deliberately, date-stamp your evidence, and re-verify the model line-up before you build on it — and remember that the choice is genuinely between the two doors.
